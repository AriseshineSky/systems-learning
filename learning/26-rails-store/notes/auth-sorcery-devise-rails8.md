# 认证方案对比：Sorcery / Devise / Rails 8 原生

> 对应 P2 Auth & Settings。选型不是「哪个最强」，而是：**代码归谁、功能边界在哪、定制时你要打什么仗**。
> Rails 8 起官方推荐 `bin/rails generate authentication`；本轨跟官方 Tutorials，默认走原生。

## 1. 一句话定位

| 方案 | 本质 | 代码在哪 |
|------|------|----------|
| **Devise** | 功能齐全的 Engine + 模块化 DSL | gem 内（你覆盖 / 配置） |
| **Sorcery** | 轻量 mixin：给核心能力，不绑死流程 | gem 提供 API，流程你写 |
| **Rails 8 Auth Generator** | 官方脚手架：把安全默认写进你的 app | **全部在你仓库里**（无 auth gem） |

```
控制权：  Devise（封装多） ←————→ Sorcery（API 少） ←————→ Rails 8（代码全透明）
功能箱：  Devise（模块多） ←————→ Rails 8（登录+重置） ←————→ Sorcery（更裸）
```

## 2. 功能对照

| 能力 | Devise | Sorcery | Rails 8 Generator |
|------|--------|---------|-------------------|
| 邮箱/密码登录 | ✅ `:database_authenticatable` | ✅ | ✅ `has_secure_password` |
| 注册 | ✅ `:registerable` | 自己写 | **自己写**（故意不生成） |
| 登出 / Session | ✅ | ✅ | ✅ 独立 `Session` 模型 |
| 密码重置 | ✅ `:recoverable` | ✅（插件） | ✅ 生成 reset 流程 |
| Remember me | ✅ `:rememberable` | ✅ | 需自加 |
| 邮箱确认 | ✅ `:confirmable` | ✅（插件） | 需自加 |
| 锁定 / 暴力破解 | ✅ `:lockable` | 插件 / 自写 | 通常靠 `rack-attack`（偏 IP，非账户级） |
| 登录追踪 | ✅ `:trackable` | 自写 | Session 有 `ip_address` 等，追踪逻辑自写 |
| 超时登出 | ✅ `:timeoutable` | 自写 | 自写 |
| OmniAuth / 社交登录 | 成熟生态 | 可接 | 另接 OmniAuth |
| 代码可读性 | 低（Engine + Warden） | 中 | **高（就是普通 Rails 代码）** |
| 依赖 | `devise` + Warden | `sorcery` | **零 auth gem**（bcrypt 等已在 Rails） |

要点：

- Devise = **开箱即用功能箱**；代价是抽象层厚、定制要学 Devise 约定。
- Sorcery = **「给我锤子，房子自己盖」**；比 Devise 透明，但安全加固与高级功能几乎全 DIY。
- Rails 8 = **官方把「够用且安全的最小集」直接生成进 app**；哲学接近 Sorcery（透明、可控），但 Session 模型与安全默认更贴现代 Rails，且有官方文档/维护。

## 3. 架构差异（为什么定制体验差这么多）

### Devise：Engine + Warden + 模块开关

```
Request → Devise Engine / Controllers → Warden strategies → User (devise modules)
              ↑
         routes.rb 里 devise_for
         config/initializers/devise.rb（大量开关）
```

- `devise :database_authenticatable, :recoverable, ...` 一句话挂一堆行为。
- 改流程 = override controller / 改 view / 钩子；简单改很快，**非标准流程容易和 Engine 较劲**。
- 适合：确认信、锁定、OmniAuth、团队已熟 Devise 的产品。

### Sorcery：Concern / mixin + 你写的 Controller

```
Request → 你的 SessionsController → User.authenticate(...)  (Sorcery API)
                                      ↑
                              authenticates_with_sorcery!
```

- 不抢路由、不塞 Engine；登录/注册/重置的 **HTTP 流程是你的**。
- 功能靠子模块（remember_me、reset_password、activity_logging…）按需打开。
- 适合：要轻量 gem、又不想从零写 bcrypt/token 细节。

### Rails 8 Generator：生成物即实现

```
bin/rails generate authentication
  → User + Session models
  → SessionsController / PasswordsController
  → Authentication concern（Current / require_authentication）
  → 密码重置 mailer 等
```

典型形态（概念上）：

| 构件 | 职责 |
|------|------|
| `User` | `has_secure_password`；身份凭证 |
| `Session` | **服务端会话行**（可吊销、可列设备） |
| cookie | 存 session id（signed），不是「只塞 user_id」 |
| `Authentication` concern | `Current.session` / `Current.user`、强制登录 |
| 注册 | **不生成** → 逼你按产品写 Settings / Sign up |

与旧习惯对比：很多老项目 cookie 里直接 `user_id`；Rails 8 默认 **Session 表**，登出/踢设备/审计更干净。这是理解 P2「Session 生命周期」的关键。

## 4. 安全与「默认正确」

三者密码侧都可落到 **bcrypt**（Devise/Sorcery 可配；Rails 用 `has_secure_password`）。

| 维度 | Devise | Sorcery | Rails 8 |
|------|--------|---------|---------|
| 社区战斗时长 | 很长 | 中等 | 新，但进 Core 维护 |
| 默认功能面 | 宽（易开 confirm/lock） | 窄 | 窄但官方脚手架 |
| 账户级锁定 | 模块开箱 | DIY | DIY / rate limit |
| Session 可吊销 | 常见靠 cookie/remember | 看你怎么建 | **一等公民 Session 模型** |
| 定制时踩坑 | Engine/Warden 魔法 | 漏写安全边角 | 漏写产品功能（确认信等） |

威胁模型粗分：

- 普通 Store / SaaS MVP → Rails 8 +（必要时）rack-attack 通常够。
- 要账户锁定、确认信、一堆 OmniAuth → Devise（或原生 + 自己补，工期更长）。
- 极简、完全掌控 mixin → Sorcery；但 **2025+ 新项目更常直接选 Rails 8 生成器**，少一个 gem。

## 5. 怎么选（决策表）

| 场景 | 倾向 |
|------|------|
| 跟官方 Tutorials / 学现代 Rails 默认 | **Rails 8 Auth**（本轨 P2） |
| 新 app，登录+重置够用，想读自己的代码 | **Rails 8 Auth** |
| 必须 confirmable / lockable / 现成 OmniAuth 生态，且要快 | **Devise** |
| 已有 Devise 生产系统 | **继续 Devise**（别为换而换） |
| 想要比 Devise 薄的 gem，又不想用生成器 | Sorcery（或 Authentication Zero 等「增强版脚手架」） |
| 企业 SSO / SCIM / 托管登录 | 超出三者；看 WorkOS / Auth0 等 |

**对本仓库 Rails Store：** P2 目标就是「理解 Authentication Generator、Session、与 Authorization 的区别」→ 用原生，并在笔记里对照旧项目若用过 Devise/Sorcery 的差异。

## 6. 与 Authorization 的边界（三种都不管）

Authentication = **你是谁**（登录态）。  
Authorization = **你能做什么**（admin / policy）。

Devise / Sorcery / Rails 8 **都不替代** Pundit / Action Policy / 手写 `admin?`。P2 checklist 里的「权限/管理员」是另一层，别和登录 gem 绑死。

## 7. 本轨落地建议

1. `bin/rails generate authentication`，把生成文件当教材读一遍（尤其 `Session` + `Authentication` concern）。
2. 自己补 **Sign up** 与 **Settings**（官方刻意留白）。
3. 写 `notes/p2-auth.md` 时对照本节：若以前用 Devise，记 3 条「魔法消失后你多写了什么 / 多看懂了什么」。
4. 需要社交登录再加 OmniAuth；需要确认信/锁定再评估「自写 vs 上 Devise」，不要一上来装全套。

## 参考

- Rails Guides / Tutorials：Authentication（官方 Store 教程对应章）
- `bin/rails generate authentication` 生成树（以本机 Rails 8.x 为准）
- Devise modules 文档；Sorcery README（plugins 列表）
