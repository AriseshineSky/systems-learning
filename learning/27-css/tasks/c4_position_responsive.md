# Task: Position + Responsive (C4)

**Status:** not started  
**Track:** css-engineering

## Goal

让页面在 Desktop / Tablet / Mobile 下都成立。程序员最容易忽略、但实战和面试都考的一环。

## Must read

- [ ] MDN `position`：static / relative / absolute / fixed / sticky
- [ ] MDN `z-index` 与 stacking context
- [ ] MDN Media queries
- [ ] MDN 响应式设计（fluid / clamp）

## Must understand

- [ ] absolute 的定位参照是谁（最近的 positioned 祖先）
- [ ] sticky vs fixed 的区别、sticky 什么时候失效
- [ ] `@media (max-width:)` vs `(min-width:)` — 移动优先为什么常用 min-width
- [ ] `min()` / `max()` / `clamp()` 在 typography 和宽度上的用法
- [ ] container queries（概念级，Optional）

## Hands-on

- [ ] 把 C3 的 Gallery 改成响应式：

```
Desktop  ┌─────┬─────┬─────┐      Mobile  ┌─────┐
         ├─────┼─────┼─────┤              ├─────┤
         ├─────┼─────┼─────┘              ├─────┤
```

- [ ] sticky 导航栏 + 页面滚动观察
- [ ] modal 弹层（absolute 定位 + 遮罩）
- [ ] 内容区 `max-width: 1200px; margin-inline: auto;`
- [ ] 标题用 `clamp()` 做响应式字号

## Output

- [ ] `notes/c4-responsive.md` — 移动优先的心智模型 + clamp 例子
- [ ] 一个页面在 3 个宽度下都成立

## Skip

- 复杂断点体系（C8 再说）
- 动画

## Notes

<!-- Your notes here -->
