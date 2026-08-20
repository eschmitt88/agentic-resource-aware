---
kind: concept
name: "collaboration topology trade-off"
status: seedling
added: "2026-08-20"
sources:
  - literature/papers/shen2026empirical.md
  - literature/papers/yang2026toward.md
related_concepts:
  - worktree-isolated-parallel-search
  - flow-level-concurrency
  - fixed-budget-evaluation-protocol
related_experiments: []
tags:
  - multi-agent
  - topology
  - deliberation-cost
  - routing
---

# Collaboration topology trade-off

## Definition

Under a fixed budget, how concurrent agents are organized is a
resource-allocation decision with a measured trade-off:

- **Subagents** (parallel exploration, post-hoc consolidation by a
  coordinator) — resilient, high-throughput, cheap in tokens; produce
  shallow, low-diversity, greedy single-dimension improvements.
- **Agent teams** (fixed-role experts handing off *before* execution) —
  fragile (multi-author code crashes), expensive in deliberation; produce
  the coupled, multi-part architectural changes subagents never
  conceptualize.
- **Single agent** — plateaus once the obvious knobs are exhausted.

## Why it matters here

Deliberation spent before execution subtracts directly from the number of
candidates evaluated inside the budget. [[shen2026empirical]] measures
this on a shared testbed: at 300 s per candidate over 50 rounds,
subagents landed 7 effective improvements to the teams' 3 — but the
subagents spent those rounds shrinking one MLP ratio from 4× to 0.75×,
while a single team patch simultaneously changed the attention window
pattern, the LR warmdown schedule, and the value-embedding vocab size.

The conclusion is **routing, not ranking**: subagents for broad shallow
sweeps, specialist teams for deep algorithmic change. Neither paper
implements the router, and the routing signal — predicting "complex
refactor" vs "broad sweep" *before* the work starts — is exactly the
admission-time question this project has to answer.

This also matches the user-level rule that subagents handle high-volume
narrow work while the main agent owns primary context and decisions —
independently arrived at, now with an empirical shape attached.

## Connections

- Depends on [[worktree-isolated-parallel-search]] for the subagent
  side's stability advantage.
- [[yang2026toward]] names the same gap from the survey side
  ("deployment-aware agentic design"): true multi-model deployments vs
  single-model role-play pipelines differ in orchestration overhead and
  should be compared under matched budgets — which nobody does.
- Caveat that matters on this box: the 7-vs-3 result budgets *training
  seconds* and excludes LLM inference. Under a token budget the subagent
  advantage should be larger, not smaller — see
  [[fixed-budget-evaluation-protocol]].
- Evidence base is one task, one GPU, one model family, no seed
  replication. Treat as a shape, not a constant.
