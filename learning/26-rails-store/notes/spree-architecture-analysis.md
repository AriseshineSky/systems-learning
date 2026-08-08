# Spree 深度解析：订单流程、可取之处、升级方案

> 以 Spree（Rails 开源电商）为例，讲解订单/结账流程，分析可取之处、待升级点、升级方案与原因。承接 `architecture-usecase-services-aggregate.md` 的分层观点。
> 状态机 + AR 回调的 ACID 分析见 `state-machine-callbacks-acid.md`。
>
> 版本背景（2026-08）：Spree **5.6**，已全面 headless / API-first（Store API v3 + TypeScript SDK + Next.js storefront）；5.6 去掉 Redis 强依赖、默认 Solid Queue 单容器部署。**Spree 6 开发中**：React admin、开源多商户、大量结构性清理。

## 1. 全景：Engine 组合 + 状态机

Spree = 一组 Rails Engine gem：

| Gem | 职责 |
|-----|------|
| `spree_core` | 领域模型 + 状态机 + 业务逻辑（心脏） |
| `spree_api` | Store API v3 / Admin API |
| `spree_admin` | 后台（5.0 重写，6.0 转 React） |
| `spree_storefront` | 传统 Rails 店面（headless 后可不用） |
| TS SDK + Next.js storefront | 5.4+ 前端主推 |

主线：`spree_core` 是胖领域层；定制靠 **decorator（猴补丁）** 覆盖 core 的类。

## 2. 订单 / 结账流程

### 2.1 Order 既是购物车又是订单
- 无单独 Cart 模型；`Spree::Order` 始终同一条记录。
- `completed_at == nil` → 购物车（in-progress）；走完结账 → 订单。
- 两个字段：`state`（结账状态机） + `status`（`draft/placed/canceled` 生命周期）。
- 好处：模型简单、天然弃单分析。代价：一个模型承担加购→售后所有职责。

### 2.2 结账状态机（`state_machines-activerecord`）
```
cart → address → delivery → payment → confirm → complete
        另有 canceled / resumed / returned 等管理态
```
- 推进：`order.next!`（抛异常）/ `order.next`（返回 false）。
- Guard：每次 transition 跑 `delivery_required?` / `payment_required?` 决定能否/跳过。
- 定制 DSL：decorator 里 `checkout_flow` + `go_to_state` / `remove_transition`，重声明时状态机重建。

### 2.3 副作用（官方已进化的最佳实践）
- 校验/阻断 → state machine callback（guard）。
- 副作用（邮件/同步外部/积分）→ **Events subscribers**，别塞 transition callback（会拖慢结账、静默失败）。
- 即笔记的"领域事件驱动、跨聚合最终一致"。

### 2.4 订单对象关系（对应复杂订单）
- `Order` → `LineItem` → `Variant`(SKU/价格/库存) → `Product`
- `Variant` 承载变体、价格、`StockItem`/`StockLocation`
- `Order` → `Shipment`(状态机)、`Payment`(状态机)、`Adjustment`(促销/税)、`Address`
- **价格快照**：`LineItem` 存下单时 `price`，不实时引用 `variant.price` ✅

## 3. 可取之处
1. "Order 即购物车"极简建模，天然弃单分析。
2. 状态机把结账显式化；`Shipment`/`Payment` 各有状态机，职责内聚。
3. Engine + Decorator 可扩展性 → 支付/物流扩展生态繁荣。
4. Checkout DSL 把"改结账流程"收敛成声明式 API。
5. 向 API-first / 事件驱动演进（5.4 SDK/Next.js，5.6 去 Redis + Solid Queue，副作用走 Events）。
6. 快照定价 + `Adjustment` 体系，可组合、可审计。

## 4. 待升级点（对照分层讨论）

| # | 问题 | 本质 |
|---|------|------|
| 1 | `Spree::Order` 上帝模型 | 结账编排+购物车+订单+状态机+海量业务方法全挤一个 AR 类；缺显式 use case 层 |
| 2 | 业务逻辑绑死 ActiveRecord | 定价/库存/结账规则长在 AR/回调，难测/难复用/难替换 |
| 3 | 聚合边界几乎为零 | 处处能 `LineItem.create`、直接改 `order.state`，不变量易绕过 |
| 4 | Decorator 猴补丁定制脆弱 | 覆盖 core 私有方法 → 升级易碎；扩展点是"改源码"而非实现接口 |
| 5 | 状态机承载副作用/编排包袱 | 老代码 transition callback 塞逻辑；状态机职责混淆 |
| 6 | gem 边界 ≠ 领域边界 | `spree_core` 巨 gem 内耦合，按技术分层非业务能力 |
| 7 | 搜索/读模型与写模型耦合 | 复杂查询压同一套 AR；CQRS 不彻底（虽有 Meilisearch） |

## 5. 升级方案与原因（渐进、向后兼容）

### A. 抽显式 Use Case / Command 层（最高优先）
- 把 `order.next!` 背后编排提到命令对象：`Checkout::AdvanceState`、`Orders::AddLineItem`、`Orders::Place`。
- Controller/API 只调命令；命令内做事务 + 调领域对象 + 发事件。
- 原因：`Order` 回归"聚合根 + 数据 + 不变量"，编排剥离；可测/可复用；与 Events 衔接。

```ruby
module Spree::Orders
  class Place
    def initialize(order:, gateway_factory: Payments::GatewayFactory)
      @order, @gateway_factory = order, gateway_factory
    end
    def call
      Spree::Order.transaction do
        Inventory::Reserve.new(@order).call
        @gateway_factory.for(@order.payment_method).charge!(@order)
        @order.finalize!
        Spree::Event.publish("order.placed", order: @order)
      end
    end
  end
end
```

### B. 定价/库存/促销下沉为领域服务 + 值对象
- `Pricing::Calculator` / `Inventory::Reservation` / `Promotions::RuleEngine`（5.1 已有规则引擎）；金额统一 `Money` 值对象。
- 原因：复杂易变规则要有名字、可测、可替换；解耦后 B2B Price Lists 等扩展更省。

### C. Adapter / Repository 收口外部依赖
- 支付（Payment Sessions，provider-agnostic）统一 `PaymentGateway` 接口；搜索走 `ProductSearchRepository`(Meilisearch/DB 双实现)；缓存/锁走 wrapper。
- 原因：领域依赖抽象非 SDK；搜索做独立读模型（CQRS），缓解 #7。5.6 去 Redis 印证依赖应可插拔。

### D. Packwerk 按业务能力切包
- 在 core / 宿主 app 用 packwerk 划 `checkout`/`pricing`/`inventory`/`payments`/`catalog`，声明依赖方向与 public API。
- 原因：技术分层 → 业务能力边界，逼近聚合/限界上下文，缓解 #6；渐进、零运行时成本，与 Spree 6 结构清理一致。

### E. 定制从 Decorator 迁移到 Hook / 接口
- 高频扩展点定义成显式接口/可注入策略（把 `checkout_flow` 式 DSL 扩到定价/履约），减少 `class_eval` 覆盖私有方法。
- 原因：猴补丁是升级碎裂之源（#4）；显式扩展点 = 依赖倒置 + 稳定契约。

### F.（可选，重）结账编排与状态机解耦
- 状态机只记录状态/校验；"下一步做什么"交命令对象；callback 只留纯校验，副作用走 Events。
- 原因：状态机专注合法状态与迁移；编排交 use case + 事件（#1/#5）。顺 Spree 官方引导方向。

## 6. 一句话总结
- 最值得学：Order=购物车极简建模、结账状态机显式化、Engine+DSL 可扩展、API-first + 事件驱动现代化。
- 最该升级：`Order` 上帝模型 + 逻辑绑死 AR + 聚合边界缺失 + decorator 脆弱定制。
- 升级主线：显式 use case/command 薄层 → 复杂逻辑下沉领域服务/值对象 → 外部依赖收口 adapter/repository → packwerk 切业务边界 → 定制转接口/hook → 状态机只管状态、编排交事件。全部可渐进，且与 Spree 5.x→6 演进方向一致。
