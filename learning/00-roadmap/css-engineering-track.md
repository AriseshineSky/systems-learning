# CSS Engineering Track

**Audience:** Rails / TypeScript 后端工程师 — 已有 Rails、TS、Tailwind、shadcn/ui 背景，目标是用 HTML/CSS 独立做出漂亮、响应式的页面。

**Goal:** 不是背 CSS 属性，而是「看到页面就能想到布局，然后把它实现出来」。走工程师实战路线，不从色彩理论等设计师路径入门。

**资源原则：** 只用免费公开资料（MDN / web.dev / CSS-Tricks），不买付费课。

**Target ceiling:** 独立用 HTML/CSS 还原真实网站、理解组件背后的布局与视觉决策、能用 Tailwind / shadcn/ui 自建设计 token — 不是设计学专家。

---

## 与其它轨的关系（方案 A 期间）

| 轨 | 每周时间 | 角色 |
|----|----------|------|
| **Rails Store** | 2–3 h | 工作之余主攻 |
| **CSS** | **2–3 h** | **工作之余主攻（与 Store 交替分日）** |
| DB | 5–6 h | 系统主轨（不变） |
| OS | 3–4 h | 副轨 A |
| Ruby R1–R4 | 暂停 | Store 完成后再恢复元编程轨 |

**原则：** Store 与 CSS 分日（如隔天交替），同属「工作之余」时段；不挤占 DB/OS。

---

## 学前水平（你已具备）

- Rails、TypeScript、Tailwind、shadcn/ui 实战经验
- 已在用 Tailwind，但偏「背诵 class」而非理解 CSS

## 还缺什么

- Box Model 直觉（padding / margin 对盒子实际尺寸的影响）
- Flex / Grid 布局思维（vs 不断试数值）
- Responsive（@media / fluid / clamp）
- 视觉概念：spacing scale、type scale、color tokens、层次

---

## 免费核心资料

| 资源 | 用途 | 链接 |
|------|------|------|
| MDN Learn CSS | C1–C4 系统性入门 | https://developer.mozilla.org/en-US/docs/Learn/CSS |
| web.dev Learn CSS | 现代 CSS 全解 | https://web.dev/learn/css/ |
| Flexbox Froggy | Flexbox 小游戏 | https://flexboxfroggy.com/ |
| CSS Grid Garden | Grid 小游戏 | https://cssgridgarden.com/ |
| A Complete Guide to Flexbox | C2 速查 | https://css-tricks.com/snippets/css/a-guide-to-flexbox/ |
| A Complete Guide to Grid | C3 速查 | https://css-tricks.com/snippets/css/complete-guide-grid/ |
| Tailwind Docs | C7 | https://tailwindcss.com/docs |
| shadcn/ui | C7 | https://ui.shadcn.com |

---

## 阶段时间线

| Phase | 周数 | 产出 |
|-------|------|------|
| C1 HTML + CSS 基础 & Box Model | 2 周 | 语义化文章页 + 一个 Card |
| C2 Flexbox | 2 周 | Navbar / Sidebar / Card / Button group / Form / Pricing table / 页面骨架 |
| C3 Grid | 2 周 | App 布局（header + sidebar + main）+ Gallery |
| C4 Position + Responsive | 2 周 | 响应式 3→2→1 列 + sticky nav + modal |
| C5 视觉设计 | 2 周 | 用 token / scale 重构前面页面 |
| C6 仿做真实网站 | 3 周 | 还原 2–3 个页面 + 决策笔记 |
| C7 Tailwind + shadcn/ui | 2 周 | 组件 gallery |
| C8 完整 Dashboard / SaaS | 3 周 | 一个响应式 Dashboard 页面 |

**Total:** ~18 周 × 2–3 h/周（与 Store 交替分日，可弹性延后）

---

## 每次 Session 流程

1. Read 对应资料 → 明确问题
2. Implement（先在 `~/src/css-playground` 动手，再对照资料）
3. Understand（为什么这样布局 / 为什么这个值）
4. Compare（同一效果，Flex 和 Grid 哪个更合适，写一句理由）
5. Record（勾选 task + 写 notes + 更新 `parallel-css.md`）

---

## 动手环境

```bash
# 练习代码另开仓库（本仓库规则 7：App code ≠ this repo）
mkdir -p ~/src/css-playground && cd ~/src/css-playground
# C1–C6 纯 HTML/CSS 即可；C7–C8 用 Vite + Tailwind：
npm create vite@latest . -- --template vanilla
npm install tailwindcss @tailwindcss/vite
```

笔记目录：`learning/27-css/notes/`

---

## 完成标准（整轨）

- [ ] 不看文档能说出 Flex vs Grid 的选择依据
- [ ] 能独立把一个真实网站页面从视觉还原成 HTML/CSS
- [ ] 能解释 Tailwind 每个 class 对应的 CSS 在做什么
- [ ] 手上有自己的 spacing / type / color token
- [ ] 能「先想好布局再实现」，而不是试数值
