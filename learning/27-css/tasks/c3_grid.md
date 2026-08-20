# Task: Grid (C3)

**Status:** not started  
**Track:** css-engineering

## Goal

掌握二维布局。Flex 解决「一排」，Grid 解决「整块」。

## Must read

- [ ] [A Complete Guide to Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [ ] [CSS Grid Garden](https://cssgridgarden.com/) 全部通关

## Must understand

- [ ] `grid-template-columns` / `grid-template-rows`
- [ ] `gap`
- [ ] `grid-column` / `grid-row`（span / 起止）
- [ ] `grid-template-areas`（给区域命名）
- [ ] `fr` 单位 vs `%` vs `auto`

## Hands-on

- [ ] App 布局：

```
┌───────────────┐
│     Header    │
├──────┬────────┤
│      │        │
│ Side │ Main   │
│ bar  │        │
└──────┴────────┘
```

- [ ] Gallery：3×3 网格，第 1 张卡片横跨两列（`grid-column: span 2`）
- [ ] 卡片列表：`repeat(auto-fill, minmax(240px, 1fr))` 自适应列数

## 决策练习（每题写一句理由）

- [ ] 导航栏一排 → Flex or Grid？
- [ ] 整个页面骨架 → Flex or Grid？
- [ ] 一组等宽卡片 → Flex or Grid？
- [ ] 一行内「左标题 + 右按钮」→ Flex or Grid？

## Output

- [ ] `notes/c3-grid.md` — 用自己的话写 Flex vs Grid 选择依据 + minmax / auto-fill 用法
- [ ] App 布局 + Gallery 两个页面可运行

## Skip

- Subgrid（高级，之后再看）
- 动画 / 过渡

## Notes

<!-- Your notes here -->
