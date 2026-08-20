# Knowledge Registry

Single source of truth. Every entry links to a task in `10-database/tasks/`, `15-operating-systems/tasks/`, or `25-ruby/tasks/`.

**Status:** `not started` | `reading` | `understanding` | `implemented` | `mastered`  
**Depth (engineering):** `deep` | `conceptual` | `skim` | `skip`

---

## Lectures

| ID | Title | Link | Task | Depth | Status |
|----|-------|------|------|-------|--------|
| L01 | Relational Model & Algebra | [CMU #01](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | foundation | deep | not started |
| L07 | Hash Tables | [CMU #07](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e1_storage_and_indexes | skim | not started |
| L08 | Indexes & Filters I | [CMU #08](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e1_storage_and_indexes | deep | not started |
| L11 | Sorting & Aggregations | [CMU #11](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e2_query_execution | deep | not started |
| L12 | Joins Algorithms | [CMU #12](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e2_query_execution | deep | not started |
| L13 | Query Execution I | [CMU #13](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e2_query_execution | deep | not started |
| L15 | Query Optimization I | [CMU #15](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e4_query_optimizer | deep | not started |
| L16 | Query Optimization II | [CMU #16](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e4_query_optimizer | deep | not started |
| L17 | Concurrency Control Theory | [CMU #17](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e3_transactions_and_isolation | skim | not started |
| L18 | Two-Phase Locking | [CMU #18](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e3_transactions_and_isolation | skip | not started |
| L20 | Multi-Version Concurrency Control | [CMU #20](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e3_transactions_and_isolation | deep | not started |
| L21 | Database Logging | [CMU #21](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e5_recovery_and_postgresql | conceptual | not started |
| L22 | Database Recovery (ARIES) | [CMU #22](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | — | skip | not started |
| L23 | Distributed OLTP | [CMU #23](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e5_architecture_and_scale | optional | not started |
| L24 | Distributed OLAP | [CMU #24](https://15445.courses.cs.cmu.edu/fall2025/schedule.html) | e5_architecture_and_scale | optional | not started |

**Skipped for engineering track:** L03–L05 (storage layout), L04 (buffer pool implementation), L10 (index latches)

Card files: `10-database/lectures/`

---

## Papers

| ID | Title | Card | Task | Depth | Status |
|----|-------|------|------|-------|--------|
| P04 | A Critique of ANSI SQL Isolation Levels | [isolation-levels.md](../10-database/papers/isolation-levels.md) | e3_transactions_and_isolation | **must read** | not started |
| P05 | Serializable Isolation for Snapshot Databases | [ssi-snapshot-databases.md](../10-database/papers/ssi-snapshot-databases.md) | e3_transactions_and_isolation | **must read** | not started |
| P06 | The Log-Structured Merge-Tree | [lsm-tree.md](../10-database/papers/lsm-tree.md) | e5_recovery_and_postgresql | **must read** | not started |
| P01 | Volcano — Extensible Query Evaluation | [volcano.md](../10-database/papers/volcano.md) | — | optional | not started |
| P02 | ARIES Recovery Algorithm | [aries.md](../10-database/papers/aries.md) | — | skip | not started |
| P03 | Architecture of a Database System | [architecture.md](../10-database/papers/architecture.md) | e5_architecture_and_scale | optional | not started |

---

## Textbook chapters

| Chapters | Topic | Task | Depth | Status |
|----------|-------|------|-------|--------|
| Ch 1–5 | Intro, SQL | foundation | deep | not started |
| Ch 14 | Indexing (B+Tree, composite) | e1_storage_and_indexes | deep | not started |
| Ch 15 | Query processing (joins) | e2_query_execution | deep | not started |
| Ch 16 | Query optimization | e4_query_optimizer | deep | not started |
| Ch 17–18 | Transactions, concurrency, MVCC | e3_transactions_and_isolation | deep | not started |
| Ch 19.1–19.4 | WAL basics | e5_recovery_and_postgresql | conceptual | not started |

Book: *Database System Concepts*, 7th ed. — **focus Ch 14–18** for engineering track.

---

## Blogs & external

| ID | Title | Card | Task | Depth | Status |
|----|-------|------|------|-------|--------|
| B01 | CMU 15-445 study guide (csdiy) | [csdiy-15445.md](../10-database/blogs/csdiy-15445.md) | foundation | skim | not started |
| B03 | PostgreSQL MVCC internals | [pg-mvcc.md](../10-database/blogs/pg-mvcc.md) | e3_transactions_and_isolation | deep | not started |

---

## Projects

### Engineering track (active)

| ID | Task | Card | Status |
|----|------|------|--------|
| E1 | B+Tree & Indexes | [e1_storage_and_indexes.md](../10-database/tasks/e1_storage_and_indexes.md) | in progress |
| E2 | Query Execution | [e2_query_execution.md](../10-database/tasks/e2_query_execution.md) | not started |
| E3 | Transactions & MVCC ★ | [e3_transactions_and_isolation.md](../10-database/tasks/e3_transactions_and_isolation.md) | not started |
| E4 | Query Optimizer | [e4_query_optimizer.md](../10-database/tasks/e4_query_optimizer.md) | not started |
| E5 | Recovery & PostgreSQL Capstone | [e5_recovery_and_postgresql.md](../10-database/tasks/e5_recovery_and_postgresql.md) | not started |

### Optional / kernel track

| ID | Task | Card | Status |
|----|------|------|--------|
| E5-opt | Architecture & Scale | [e5_architecture_and_scale.md](../10-database/tasks/e5_architecture_and_scale.md) | optional |
| BT0 | C++ Primer | [p0_cpp_primer.md](../10-database/tasks/p0_cpp_primer.md) | not started |
| BT1 | Buffer Pool Manager | [p1_buffer_pool.md](../10-database/tasks/p1_buffer_pool.md) | paused |
| BT2 | B+ Tree Index | [p2_b_plus_tree.md](../10-database/tasks/p2_b_plus_tree.md) | not started |
| BT3 | Query Execution | [p3_query_execution.md](../10-database/tasks/p3_query_execution.md) | not started |
| BT4 | Concurrency Control | [p4_concurrency.md](../10-database/tasks/p4_concurrency.md) | not started |

---

## OS track

See [os-roadmap.md](./os-roadmap.md) and `15-operating-systems/tasks/`.

---

## Ruby track

See [ruby-roadmap.md](./ruby-roadmap.md) for full index.

---

## CSS track

See [css-roadmap.md](./css-roadmap.md) for full index.
