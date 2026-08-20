# Systems Learning System

Task-driven knowledge system for **Rails / Java / Go backend engineers**.

Repo: [`AriseshineSky/systems-learning`](https://github.com/AriseshineSky/systems-learning)  
（原名 `database-learning`，已改名以覆盖多轨。）

**Not a file dump.** Every material must be linked to a task.  
**No Google Calendar required** — schedule lives in Markdown (`now` / roadmap / weekly-review).

## Tracks（方案 A）

| Track | Role | Dashboard |
|-------|------|-----------|
| **Rails Store** | 工作之余主攻（与 CSS 交替） | [`parallel-rails-store.md`](learning/01-now/parallel-rails-store.md) |
| **CSS** | 工作之余主攻（与 Store 交替） | [`parallel-css.md`](learning/01-now/parallel-css.md) |
| **DB** | 系统知识主轨 | [`now.md`](learning/01-now/now.md) |
| **OS** | 副轨 A | [`parallel.md`](learning/01-now/parallel.md) |
| **Ruby R1–R4** | 暂停（Store 后再开） | [`parallel-ruby.md`](learning/01-now/parallel-ruby.md) |

Philosophies:

- [Rails Store track](learning/00-roadmap/rails-store-track.md)
- [CSS engineering track](learning/00-roadmap/css-engineering-track.md)
- [DB engineering track](learning/00-roadmap/engineering-track.md)
- [OS engineering track](learning/00-roadmap/os-engineering-track.md)
- [Ruby engineering track](learning/00-roadmap/ruby-engineering-track.md)（paused）

## Daily workflow

```bash
cd ~/src/systems-learning

# 工作之余深耕 Rails → 只看这个
cat learning/01-now/parallel-rails-store.md
cat learning/26-rails-store/tasks/<task>.md

# 工作之余 CSS 日（与 Rails 交替）
cat learning/01-now/parallel-css.md
cat learning/27-css/tasks/<task>.md

# 系统主轨（DB）
cat learning/01-now/now.md

# 有余力：OS
cat learning/01-now/parallel.md

# 全局队列 / 周回顾
cat learning/02-backlog/backlog.md
cat learning/00-roadmap/weekly-review.md
```

## Structure

```
learning/
├── 00-roadmap/
│   ├── rails-store-track.md / rails-store-roadmap.md
│   ├── engineering-track.md      # DB
│   ├── os-engineering-track.md
│   ├── ruby-engineering-track.md # paused
│   ├── master-schedule.md
│   └── weekly-review.md
├── 01-now/
│   ├── now.md                    # DB
│   ├── parallel-rails-store.md   # Rails Store ★
│   ├── parallel.md               # OS
│   └── parallel-ruby.md          # paused
├── 02-backlog/
├── 10-database/
├── 15-operating-systems/
├── 25-ruby/                      # R1–R4 paused
├── 26-rails-store/               # P1-A … P4
│   ├── tasks/
│   └── notes/
├── 27-css/                       # C1–C8
│   ├── tasks/
│   └── notes/
├── 20-architecture/
└── 30-leetcode/
```

## Rules

1. **One active sub-project per dashboard** — Store 看 `parallel-rails-store.md`，CSS 看 `parallel-css.md`，DB 看 `now.md`。
2. **Priority（方案 A）:** Rails Store ≈ CSS（工作之余，交替）≈ DB（系统日）> OS > Ruby(paused) > LeetCode。
3. **New material → must link to a task.**
4. **Store cards, not PDFs** — link + your understanding.
5. **Status:** `not started → reading → understanding → mastered`
6. **Weekly review** — `learning/00-roadmap/weekly-review.md`
7. **App code ≠ this repo** — Rails 应用另开仓库；进度与笔记只写这里。

## Course resources

### Rails Store (after-work focus)

| Resource | Link |
|----------|------|
| Rails Tutorials | https://rubyonrails.org/docs/tutorials |
| Rails Guides | https://guides.rubyonrails.org/ |

### CSS (after-work focus, alternates with Rails Store)

| Resource | Link |
|----------|------|
| MDN Learn CSS | https://developer.mozilla.org/en-US/docs/Learn/CSS |
| web.dev Learn CSS | https://web.dev/learn/css/ |
| Flexbox Froggy | https://flexboxfroggy.com/ |
| CSS Grid Garden | https://cssgridgarden.com/ |

### Database

| Resource | Link |
|----------|------|
| CMU 15-445 (Fall 2025) | https://15445.courses.cs.cmu.edu/fall2025/ |
| Lectures (YouTube) | https://www.youtube.com/playlist?list=PLSE8ODhjZXjYMAgsGH-GtY5rJYZ6zjsd5 |
| Hands-on | PostgreSQL |

### Operating Systems (parallel A)

| Resource | Link |
|----------|------|
| NJU 2026 OS (jyy) | https://www.bilibili.com/video/BV1eqEA6JEZA |
| Course wiki | https://jyywiki.cn |

### Ruby (paused)

见 [ruby-engineering-track.md](learning/00-roadmap/ruby-engineering-track.md) — Store 完成后再恢复。

## Optional kernel track

BusTub (P0–P4) tasks remain for DB implementation depth. Keep BusTub **code in a private repo** — CMU discourages public solutions.

This repo stores only learning notes, task plans, and material cards.
