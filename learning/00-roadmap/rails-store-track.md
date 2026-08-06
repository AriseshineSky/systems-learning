# Rails Store Track

**Audience:** 有 Rails 经验，想通过官方 Tutorials 深耕现代 Rails（8/8.1）推荐做法。

**Goal:** 完整跟完 [Rails Tutorials](https://rubyonrails.org/docs/tutorials)，得到可运行的 Store，并留下可复用笔记 — 不是抄完教程打勾。

**App code:** 另开一个应用仓库（建议 `rails-store`）；**进度与笔记只写本仓库** `26-rails-store/`。

---

## 与其它轨的关系（方案 A 期间）

| 轨 | 角色 | 说明 |
|----|------|------|
| **Rails Store** | 工作之余主攻 | `parallel-rails-store.md` |
| DB | 系统主轨 | 仍在 `now.md`；与 Store 分日或分时段 |
| OS | 副轨 A | 有余力 |
| Ruby R1–R4 | **暂停** | Store 完成后再恢复元编程轨 |
| LeetCode | 最低 | 不挡主线 |

**原则：** 深耕 Rails 的晚上只看 `parallel-rails-store.md`；一次只做一个子项目（P1-A → P4）。

---

## 子项目地图

| ID | Task | 产出 |
|----|------|------|
| P1-A | Bootstrap | 可 `bin/rails s` 的空 Store + Rails 8 默认差异笔记 |
| P1-B | Product CRUD | 完整 Product CRUD |
| P1-C | Associations | 第一个关联可跑通 |
| P1-D | Testing & Wrap | 测试绿 + Phase 1 小结 |
| P2 | Auth & Settings | 注册/登录/Session/设置 |
| P3 | Wishlists | Join model + 页面 + 权限 |
| P4 | Reviews | Nested resources + UGC + 测试 |

官方入口：https://rubyonrails.org/docs/tutorials

---

## 每次 Session 流程

1. Read 官方对应章节 → 明确问题  
2. Implement（先自己写，再对照）  
3. Understand（为什么这样设计 / Rails 自动做了什么）  
4. Test（手动 + 自动化）  
5. Commit（应用仓库：一个学习目标一次）  
6. Record（勾选 task + 写 notes + 更新 `parallel-rails-store.md`）

---

## 完成标准（整轨）

- [ ] 不看教程能独立 `rails new` 并讲清默认结构  
- [ ] 独立 model / migration / controller / view  
- [ ] Associations + Rails 8 Auth + Turbo 基本用法  
- [ ] 能解释与旧 Rails 的主要差异  
- [ ] 模式能迁到自己的业务项目  
