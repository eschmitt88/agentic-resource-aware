---
kind: concept
name: "temporal-aware scheduling"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/du2025temporal.md
related_concepts:
  - two-tier-scheduling-hierarchy
  - heterogeneous-accelerator-coordination
  - mixed-criticality-preemption
related_experiments: []
tags:
  - scheduling
  - prediction
  - switching-cost
---

# Temporal-aware scheduling

## Definition

A scheduling discipline that uses *predicted future state* (demand,
arrival patterns, cache state) in addition to *current state* when
making allocation decisions. Distinguishes from "reactive" schedulers
that consider only instantaneous state. Two main levers: short-term
**warm-up / cache affinity** (route similar work to already-warm
units) and long-term **switching-cost minimization** (avoid bouncing
work across units as load shifts). Introduced for distributed LLM
inference in [[du2025temporal]] (TORTA).

## Why it matters here

The user's workload is highly periodic: `/digest` runs on cron,
`/iterate` chains run overnight, `/discover` and `/propose` cluster
in working hours. A reactive coordinator can't exploit this — it
admits each job as it arrives, on instantaneous state. A
temporal-aware policy would pre-warm environments before predicted
arrivals and avoid the swap thrash that comes from naive
chronological FIFO.

## Connections

- [[two-tier-scheduling-hierarchy]] — temporal predictions go in the
  macro (slow) layer; reactive arbitration goes in the micro (fast)
  layer.
- [[heterogeneous-accelerator-coordination]] — "warm-up affinity"
  means a project that ran on GPU last time should ideally land on
  the same GPU next time, all else equal.
- [[mixed-criticality-preemption]] — predicted reactive bursts give
  the preemption policy advance warning, letting it drain proactive
  work proactively instead of preempting mid-flight.
- Open question: is the user's existing `state.db` log dense enough
  to fit even a simple ARIMA / Markov-chain temporal model on the
  workload? If not, this concept is a longer-term bet.
