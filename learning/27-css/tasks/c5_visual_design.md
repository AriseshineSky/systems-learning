# Task: 视觉设计（Spacing / Typography / Color）(C5)

**Status:** not started  
**Track:** css-engineering

## Goal

CSS 技术本身不会让页面变漂亮。建立 spacing scale、type scale、color tokens 三个视觉概念。

## Must understand

### 1. Spacing（间距刻度）

- [ ] 只用刻度：4 / 8 / 12 / 16 / 24 / 32 / 48 / 64
- [ ] 避免 13 / 17 / 21 / 29 / 37 到处乱用
- [ ] 间距节奏：相关元素间距小，区块间距大，层次从间距里来

### 2. Typography

- [ ] font-family / font-size / font-weight / line-height / letter-spacing
- [ ] 一套基础比例：

| 角色 | size | line-height | weight |
|------|------|-------------|--------|
| Heading | 32px | 1.2 | 700 |
| Body | 16px | 1.5 | 400 |
| Caption | 14px | 1.4 | 400 |

### 3. Color tokens

- [ ] 不写裸色。定义语义 token：

primary / secondary / background / surface / border / text / muted text / success / warning / error

- [ ] 正文 vs 背景对比度至少 WCAG AA

## Hands-on

- [ ] 把 C4 的页面抽成 CSS 变量（spacing / type / color 三组）
- [ ] 用 token 重写按钮的 normal / hover / active / disabled 四态
- [ ] 找一个你觉得「丑」的页面，只改 spacing + type 三处，看效果
- [ ] Optional：读一篇 design system 文章（web.dev Learn CSS 相关章节）

## Output

- [ ] `notes/c5-visual-design.md` — 自己的 token 清单 + 3 个 spacing / type 决策
- [ ] 重构后的页面，说得出「为什么这里用 24 不用 23」

## Skip

- 色彩理论 / 色轮（工程师路线不需要）
- 动画缓动

## Notes

<!-- Your notes here -->
