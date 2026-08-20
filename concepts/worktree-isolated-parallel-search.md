---
kind: concept
name: "worktree-isolated parallel search"
status: seedling
added: "2026-08-20"
sources:
  - literature/papers/shen2026empirical.md
related_concepts:
  - collaboration-topology-tradeoff
  - evolutionary-search-over-solutions
  - asynchronous-multi-gpu-worker-pool
related_experiments: []
tags:
  - isolation
  - git-worktree
  - failure-containment
  - autoresearch
---

# Worktree-isolated parallel search

## Definition

Give every candidate modification its own Git worktree, gate it behind a
deterministic patch contract and a static preflight check, and let a
shared global memory carry what was learned across rounds. Failures stay
inside the worktree that caused them; the main branch only advances on a
candidate that trained, evaluated, and improved the metric.

## Why it matters here

This is already the project's runtime discipline ("destructive runs
happen in Git worktrees, never against the primary checkout"), and
[[shen2026empirical]] supplies the first external measurement of *why* it
pays: isolated workers are the reason subagent mode is the stable,
high-throughput topology, while the agent-team topology — which patches a
single shared worktree from three authors — pays a visible crash tax.

The instrumentation is the part worth copying. Every proposal is
classified into four states: **proposal failure** (patch violates the
Search/Replace format), **preflight failure** (well-formed but rejected
by static checks), **training crash** (passed preflight, died at
runtime), **training success**. That taxonomy turns "the agent failed"
into a diagnosis with an obvious remedy per bucket, and it is cheap —
it costs a counter per state.

## Connections

- The stability leg of [[collaboration-topology-tradeoff]].
- Local analogue of the isolation in
  [[asynchronous-multi-gpu-worker-pool]]: there, workers are isolated so
  a slow job doesn't stall the pool; here, so a broken patch doesn't
  contaminate the main branch.
- Cheap adoption: emit the four-state proposal taxonomy into each
  experiment's `metrics.json`, so failure mode is tracked alongside
  score rather than buried in `log.md`.
- Costs to keep in view: a worktree per candidate is disk and setup time;
  the shared global memory is the one channel through which contamination
  can still cross worktrees.
