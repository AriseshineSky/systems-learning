# Current Task

**Task:** B+Tree & Indexes (Engineering E1)  
**Track:** engineering — Part 1 (~40% ROI)  
**Started:** 2026-06-06  
**Target:** Index design checklist + leftmost prefix fluency

## Goal

Understand B+Tree indexing for production — range scans, composite indexes, leftmost prefix rule. Skip buffer pool and page-split implementation.

## Output

- [ ] Notes: index design checklist
- [ ] 3+ EXPLAIN (ANALYZE, BUFFERS) examples saved
- [ ] Can explain why `WHERE b = 1` can't use `INDEX(a, b, c)`

## Materials

See: [tasks/e1_storage_and_indexes.md](../10-database/tasks/e1_storage_and_indexes.md)

## Today's focus

1. Watch CMU Lecture #08 (Indexes & Filters I)
2. Read textbook Ch 14.1–14.5 (B+Tree, composite indexes)
3. Skim Lecture #07 — why Hash ✗ for `BETWEEN` range queries
4. Run one query with `EXPLAIN (ANALYZE, BUFFERS)` — with and without an index

## Skip today

- Lectures #03–#05, #04 (buffer pool), #10 (latches)
- BusTub P1/P2 implementation
- Clock / LRU-K replacement policies

---

## Parallel tracks

- **Rails Store（工作之余主攻）:** [`parallel-rails-store.md`](./parallel-rails-store.md) — P1-A Bootstrap  
- OS parallel A: [`parallel.md`](./parallel.md) — O1 when time allows  
- Ruby R1–R4: [`parallel-ruby.md`](./parallel-ruby.md) — **paused** until Store done  
- Master schedule: [`master-schedule.md`](../00-roadmap/master-schedule.md)
