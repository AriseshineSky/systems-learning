# Rails 分层架构：Use Case / Service / 外部交互 / 聚合根

> 起因：use case 放 services 吗？与 Redis/ES 等外部交互放哪？如何实现聚合根？use case 是不是很薄的一层，复杂逻辑放哪？以"组装订单"（商品/价格/库存/用户/地址/图片/规格/变体/付款方式）为例。

## 0. 先破误区：Rails 主流有两大流派

| 流派 | 代表 | 主张 |
|------|------|------|
| Vanilla Rails / "The Rails Way" | DHH / 37signals | Fat model + skinny controller + PORO + Concern，**不滥用 service object** |
| Service Object / 类 DDD 分层 | 大团队、复杂业务 | `app/services` 放 use case，引入 repository / value object / domain event |

- **Rails 没有框架内建的 use case 层**，`app/services` 是社区约定。
- 是否引入取决于**业务复杂度**。订单场景足够复杂，值得显式 use case 层。
- 简单 CRUD 直接 fat model 就够，别过度设计。

## 1. Use Case 是薄层（编排，不是计算）

职责边界：

- ✅ 编排：步骤顺序、事务边界、调用哪些领域对象、发什么事件、返回结果。
- ❌ 不写：定价公式、库存规则、变体校验等复杂领域逻辑。

> use case 负责"编排流程"，领域对象负责"业务规则"。复杂逻辑下沉，use case 保持薄。

```ruby
# app/use_cases/orders/place_order.rb
module Orders
  class PlaceOrder
    def initialize(user:, cart:, address:, payment_method:)
      @user, @cart, @address, @payment_method = user, cart, address, payment_method
    end

    def call
      ApplicationRecord.transaction do
        priced = Pricing::Calculator.new(@cart).call            # 复杂定价 → 领域对象
        Inventory::Reservation.new(priced.line_items).reserve!  # 复杂库存 → 领域对象
        order  = Order.place!(user: @user, items: priced.line_items, address: @address)
        Payments::Gateway.for(@payment_method).charge!(order)   # 外部交互 → adapter
        Result.success(order)
      end
    rescue Inventory::OutOfStock, Payments::Declined => e
      Result.failure(e)
    end
  end
end
```

每一行几乎都在"调用别人"，自己不写规则 → 这就是薄。

## 2. 复杂逻辑放哪：领域对象（重点）

| 复杂点 | 放哪 | 对象类型 |
|--------|------|---------|
| 定价（折扣/税/满减/币种） | `pricing/` | Policy / Calculator（PORO） |
| 库存扣减/预留/超卖 | `inventory/` | 领域服务（PORO 或 AR 方法） |
| 商品规格/变体（SKU） | `Product` / `Variant` 模型 | Entity + Value Object |
| 金额/币种 | `Money` / `Price` | Value Object（不可变 PORO） |
| 地址 | `Address` | Value Object / Entity |
| 付款方式差异 | 每种一个 gateway | Strategy + Adapter |

原则：**一段逻辑"复杂"或"会变"，就给它一个名字（一个类）。** 别散落在 use case / controller。

## 3. 聚合根（Aggregate Root）在 Rails 的落地

张力：AR 模型既是实体又是仓储（任意模型都能全局 `.where`），破坏"只能经聚合根访问内部实体"。Rails **不强制**，靠约定 + 封装逼近。

以 `Order` 为聚合根：

1. **单一入口**：外部只操作 `Order`，不直接 `LineItem.create`。

```ruby
class Order < ApplicationRecord
  has_many :line_items

  def self.place!(user:, items:, address:)
    order = new(user:, address:)
    items.each { |i| order.add_item(i) }  # 通过根维护子实体
    order.recalculate_total!
    order.save!
    order
  end

  def add_item(item)
    raise Order::Locked if placed?        # 不变量在根上强制
    line_items.build(item.to_attrs)
  end
end
```

2. **聚合间用 ID 引用**：`Order` 存 `product_id / user_id`，不嵌套持有整个 `Product / User`。`Product / User / Inventory` 各是独立聚合根。
3. **事务边界 = 聚合边界**：一次事务只强一致改一个聚合；跨聚合用**领域事件 / 最终一致**（`after_commit` 发事件 → job）。
4. **值对象**：`Money / Address / 规格组合` 用不可变 PORO 或 `composed_of`，随根存储。

> 分层良好，但边界靠团队纪律，不是框架保证。

## 4. 与外部系统交互（Redis / ES / 支付）

核心：**领域层不直接依赖 `Redis.new` / `Elasticsearch::Client`，而依赖自己封装的 adapter / gateway / repository。**

```
app/
  adapters/            # 或 gateways/ clients/
    payments/
      stripe_gateway.rb
      paypal_gateway.rb
    search/
      product_search.rb   # 包 ES，暴露领域语义查询
  repositories/
    product_search_repository.rb
```

| 外部系统 | 定位 | 放哪 | 说明 |
|---------|------|------|------|
| Redis（缓存） | 基础设施 | `Rails.cache` / Cache wrapper | 别到处 `Redis.get` |
| Redis（队列/锁/计数） | 基础设施 | adapter（如 `Inventory::StockLock`） | 领域只看到"加锁/扣减" |
| ES（搜索） | 独立读模型 | `app/searches/ProductSearch` 或 repository | 读侧与写侧聚合分开（CQRS 味道），写后用 `after_commit`/job 同步 |
| 支付网关 | 外部服务 | `app/adapters/payments/*` + Strategy | 每种付款方式统一接口 `charge!(order)` |

为什么包一层：可测（stub adapter）、可换实现、依赖倒置（领域 → 抽象接口 ← 实现 → SDK）。

## 5. 订单例子的完整分层

```
下单请求
  → Controller（薄：参数、鉴权）
    → Orders::PlaceOrder（use case，薄：编排 + 事务）
        ├─ Pricing::Calculator      (领域：定价规则)
        ├─ Inventory::Reservation   (领域：库存，内部用 Redis 锁 adapter)
        ├─ Order.place!             (聚合根：建单、维护 line_items、不变量)
        ├─ Payments::Gateway.for(m) (adapter：支付，Strategy 分方式)
        └─ 发 OrderPlaced 事件      (after_commit → job → 同步 ES/邮件/积分)
```

- **Product / Variant / 图片 / 规格**：属 `Product` 聚合；`PlaceOrder` 只按 `variant_id` 取快照价。
- **金额**：`Money` 值对象贯穿，避免 float。
- **下单快照**：把价格、名称快照进 `LineItem`（商品价会变，订单必须记录下单当时的值，别直接引用 `product.price`）。

## 6. 一句话总结

1. Use case（services）= 薄编排层，只管流程/事务/调度。
2. 复杂逻辑下沉到命名清晰的领域对象：Calculator / Policy / Value Object / 领域服务。
3. 聚合根是好抽象，Rails 不强制，靠"单一入口 + 聚合间 ID 引用 + 事务=聚合边界 + 领域事件"逼近。
4. 外部系统（Redis/ES/支付）一律包 adapter/gateway/repository，领域依赖抽象而非 SDK。
5. 别过度设计：简单场景 fat model 够，复杂订单才上 use case + 聚合。
