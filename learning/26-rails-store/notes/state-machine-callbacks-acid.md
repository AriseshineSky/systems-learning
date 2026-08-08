# 状态机 + AR 回调的 ACID 分析（以 Spree 为例）

> 承接 `spree-architecture-analysis.md`。问题：Spree 大量使用状态机和 AR 回调，哪些满足 ACID、哪些不满足、为什么？满足的是如何实现的？（前提：DB 事务满足 ACID。）

## 0. 一句话结论

- **只有"写数据库"的部分**才可能享受 ACID，而且前提是它们落在**同一个 DB 事务**里。
- AR 把 `save/create/update/destroy` **自动包在一个事务**里，所以**事务内回调**（`before_*` / `after_save|create|update|destroy`）的 DB 写是原子的、可回滚的 → 满足 A/C/I/D（DB 层）。
- **`after_commit` / `after_rollback` 在事务之外**运行，其 DB 写是**另一个事务**，**不随原事务回滚** → 不满足与原操作的原子性。
- **一切外部副作用**（邮件、Redis、ES、HTTP、发 job、支付网关）**天然不在 ACID 范围内**，无论放在哪个回调里都无法回滚。
- Spree 结账状态机设了 **`use_transactions: false`**，意味着**状态机自己不包事务**，跨多条记录的一次推进**不保证原子**——这是最需要警惕的 ACID 缺口。

## 1. 复习：AR 如何"借用"DB 的 ACID

DB 事务给 ACID：
- **A 原子性**：BEGIN…COMMIT 之间要么全成要么全滚（靠 undo/回滚段）。
- **C 一致性**：约束（FK/unique/not null/check）+ 应用不变量在事务结束时成立。
- **I 隔离性**：并发事务互不干扰（由隔离级别决定，PG/MySQL 默认 READ COMMITTED / RR）。
- **D 持久性**：COMMIT 返回后落盘（WAL + fsync）。

AR 的关键机制：**`save` / `destroy` 被自动包进一个事务**：

```ruby
# 概念等价
ActiveRecord::Base.transaction do
  run before_validation / validations / before_save / before_create
  INSERT/UPDATE ...
  autosave 关联(has_many/has_one) 的 INSERT/UPDATE   # 同一事务内
  run after_create / after_save
end   # COMMIT
run after_commit   # ← 在 COMMIT 之后，事务外
```

所以回调分成泾渭分明的两类，ACID 属性完全不同。

## 2. 两类回调 × ACID

### 2.1 事务内回调（满足 ACID —— 仅限 DB 写）

包含：`before_validation` / `after_validation` / `before_save` / `before_create` / `before_update` / `after_create` / `after_update` / `after_save` / `before_destroy` / `after_destroy`。

- **A ✅**：它们里面做的**数据库写**和主记录的 INSERT/UPDATE 在**同一事务**。任一步抛异常或 `throw :abort`（before 回调返回 false）→ 整个事务回滚，DB 层没有任何残留。
- **C ✅（部分）**：`validations` 在事务内跑，配合 DB 约束共同保证一致性。**但**跨记录/跨聚合的业务不变量（如"订单总价 = 明细之和"）靠回调重算维持，不是 DB 约束，能被 `update_column` / `save(validate:false)` / `insert_all` 绕过。
- **I ✅（DB 层）**：受 DB 隔离级别保护；回调里读到的是本事务未提交的写，别的事务读不到（READ COMMITTED）。**但** check-then-act 类逻辑（"库存>0 就减 1"）在并发下会丢更新，需显式加锁（见 §4）。
- **D ✅**：随事务 COMMIT 持久化。

> **实现方式**：完全"借"DB 事务——AR 用 `transaction do ... end` 包裹，异常/`:abort` 触发 `ROLLBACK`；嵌套事务用 **SAVEPOINT**（需 `requires_new: true`，否则内层异常会污染外层，即经典"被吞掉的回滚"坑）。

**致命例外**：如果事务内回调做了**外部副作用**（`after_create` 里发邮件 / 调 HTTP / `Redis.set` / 入队 job），这些**不是 DB**，无法回滚。若之后事务失败：DB 滚了，但邮件已发、job 已入队（且指向一条根本不存在的记录）→ **破坏原子性**。这就是 **dual-write 问题**。

### 2.2 事务后回调（不满足与原操作的原子性）

包含：`after_commit` / `after_create_commit` / `after_update_commit` / `after_destroy_commit` / `after_rollback`。

- **不在原事务里**。`after_commit` 在 COMMIT **之后**执行；此时数据已持久化。
- 它里面即使抛异常，**已提交的数据也不会回滚**（只是后续 after_commit 可能被跳过，且现代 Rails 会把异常抛出）。它自己的 DB 写是**新的独立事务**。
- **A ❌**：与原操作不是原子的。
- **D 顺序正确 ✅**：因为它在持久化之后才跑，所以"确认写成功后再触发外部副作用"是对的位置。

> **这就是为什么最佳实践是：外部副作用放 `after_commit`，而不是 `after_save`。** 放 after_save 会在"可能被回滚的写"之前就把邮件发出去；放 after_commit 至少保证"只有真的提交了才发"。但注意：after_commit **仍不是事务性的**——发送本身失败没有回滚可言，需要重试/幂等/outbox 兜底。

## 3. 状态机的 ACID（Spree 关键点）

Spree 用 `state_machines-activerecord`。两个事实：

1. 该 gem 集成 AR 时，**默认 `use_transactions: true`**：一次 `event`（transition + 持久化 + `around/after_transition` 回调）会被**包进一个事务**，任一步失败则整体回滚 → 这种情况下**状态迁移是原子的**。
2. **但 Spree 的结账状态机显式设了 `use_transactions: false`**（见 core `Order::Checkout`：`state_machine :state, use_transactions: false, action: :save_state`）。

`use_transactions: false` 的含义与后果：
- 状态机**不再自己开事务**。持久化只由 `action: :save_state`（一次 `save`）完成，**那次 save 仍是原子的**（它自带事务）。
- **但**：如果一次 `order.next!` 触发了**多条记录**的变更（改 order.state + `after_transition` 里建 `StateChange` + 创建 `Shipment` + 调整 `Payment`…），这些若分散在**不同的 save**里，就落在**不同事务**中 → **跨记录不原子**。中途失败可能留下"order 已到 payment 态，但 shipment 没建成"的不一致。
- 也就是说：**Spree 结账推进的原子性，不由状态机保证**，要么落在单条 save 的范围内，要么依赖调用方（use case / `Order.transaction { ... }`）自己包事务。

> 为什么 Spree 这么设？工程折中：状态机事务与 AR 嵌套事务/回调交互容易出问题，Spree 倾向"状态尽量先落库"并自行管理更大范围的事务。**代价就是把原子性责任推给了调用方**——这正是 `spree-architecture-analysis.md` 里"抽显式 use case 层、用 `Order.transaction` 包编排"升级方案要解决的。

**满足 ACID 的做法（若要让一次推进原子）**：把整段编排包进一个事务：

```ruby
Spree::Order.transaction do
  order.next!            # 状态迁移
  build_shipments!(order)
  order.recalculate!
end                      # 全成才 COMMIT，任一步异常整体回滚
```

外部副作用（发确认邮件、同步 ES、扣积分）**不要**放进这个事务，改用事务提交后的事件订阅（Spree 官方推荐 Events subscribers = 语义上的 after_commit）。

## 4. 隔离性专题：超卖 = 典型不满足

即使写落在同一事务，**并发**下仍可能违反业务一致性，这是**隔离级别 + check-then-act** 的问题，不是"包了事务就自动解决"：

```ruby
# 危险：两个请求同时读到 stock=1，各自减 1 → 卖出 2 件（丢更新）
if stock_item.count_on_hand > 0
  stock_item.update!(count_on_hand: stock_item.count_on_hand - 1)
end
```

- READ COMMITTED 下两事务都读到 1，互不可见对方的未提交写 → 超卖。
- **正确实现（Spree 也用）**：
  - **悲观锁** `stock_item.with_lock { ... }`（`SELECT … FOR UPDATE`），串行化对该行的修改；
  - 或 **乐观锁** `lock_version`（AR 内置，版本不匹配抛 `StaleObjectError`）；
  - 或把"扣减"下推为**原子 SQL** `UPDATE … SET count = count - 1 WHERE count >= 1`，靠 DB 行锁保证。
- 结论：**A/D 靠事务，I 在并发热点上必须靠显式锁或原子写**，光"有事务"不够。

## 5. 外部副作用为什么永远不满足 ACID + 怎么办

DB 事务只能回滚 DB。邮件/HTTP/Redis/ES/发 job/支付网关都是"另一个系统"，无回滚概念：

- 放**事务内**：违反原子性（回滚了但副作用已发生），还可能在网络调用期间**长时间持有 DB 行锁**（如在事务里 charge 支付网关）。
- 放**after_commit / 事件订阅**：顺序对（提交后才触发），但**仍非事务**——发送失败不会回滚已提交数据。
- 工程解法：
  1. **入队 job 要在提交后**：Rails 7.2+ 的 `enqueue_after_transaction_commit`（避免"事务回滚了 job 却指向不存在记录"）。
  2. **幂等 + 重试**：外部调用做幂等键，失败重试。
  3. **Outbox 模式**：在**同一事务**里写一条 `outbox_events` 记录（这步是 ACID 的），提交后由 relay/job 读表再投递外部 → 把"不可靠外部调用"转成"可靠的本地事务写 + 异步投递"。这是分布式下最接近"事务性副作用"的方案。
- 支付：Spree 用 `Payment` 状态机 + Payment Sessions 把"外部 charge 结果"**记录成本地状态**并对账，而不是幻想让 charge 参与 DB 事务。

## 6. 速查表

| 机制 / 位置 | A | C | I | D | 说明 |
|-------------|---|---|---|---|------|
| 单个 `save` 的 DB 写 | ✅ | ✅* | ✅DB | ✅ | AR 自动包事务；*跨聚合不变量靠约束/锁 |
| 事务内回调的 **DB 写** | ✅ | ✅* | ✅DB | ✅ | 与主记录同一事务，一起回滚 |
| 事务内回调的 **外部副作用** | ❌ | ❌ | ❌ | ❌ | 不可回滚，dual-write |
| `after_commit` 的 DB 写 | ❌(与原操作) | 局部✅ | ✅DB | ✅ | 独立新事务，不随原事务回滚 |
| `after_commit` 的外部副作用 | ❌ | ❌ | ❌ | — | 位置对但非事务，需幂等/outbox |
| 状态机 `use_transactions: true`（默认） | ✅ | ✅* | ✅DB | ✅ | 一次 event 原子 |
| Spree 结账 `use_transactions: false` | ⚠️单 save 内✅ / 跨记录❌ | ⚠️ | ⚠️ | ✅ | 原子性责任交给调用方，需 `Order.transaction` 包 |
| 并发热点（库存等） | — | ❌若无锁 | ❌若无锁 | — | 必须悲观/乐观锁或原子 UPDATE |

## 7. 落到 Spree 的结论

1. **满足 ACID 的**：一次 `save`/`event(默认)` 内的所有 **DB 写**（含事务内回调、autosave 关联）——靠 AR/状态机自动包裹的 DB 事务实现（SAVEPOINT 处理嵌套）。
2. **不满足的**：
   - `after_commit` 及一切**外部副作用**（与原操作非原子）；
   - **Spree 结账推进跨多记录**（因 `use_transactions: false`，除非调用方自己包 `Order.transaction`）；
   - **并发下的业务不变量**（超卖等，需显式锁/原子 SQL）。
3. **升级方向**（呼应架构笔记）：编排收进 **use case + 显式事务**保证跨记录原子；外部副作用一律走**提交后事件 + 幂等 + outbox**；热点资源用**锁/原子写**补齐隔离性。
