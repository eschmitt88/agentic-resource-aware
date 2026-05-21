---
kind: concept
name: "reactive vs proactive agent flows"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/wei2025agent.md
related_concepts:
  - mixed-criticality-preemption
  - flow-level-concurrency
related_experiments: []
tags:
  - taxonomy
  - scheduling
  - workload-characterization
---

# Reactive vs proactive agent flows

## Definition

A two-class taxonomy for LLM-agent workloads, introduced in
[[wei2025agent]]:

- **Reactive flows** — user-initiated, foreground, bursty,
  latency-critical (time-to-first-token matters). Example: a
  conversational query.
- **Proactive flows** — system-initiated, background, long-running,
  best-effort (throughput matters; latency mostly doesn't).
  Example: continuous indexing of new files, scheduled paper digest,
  background experiment.

The classes have fundamentally different SLOs and benefit from
different scheduling policies.

## Why it matters here

This is the right primary taxonomy for the resource-aware
coordinator. The user's described workload — "few research projects
going on at once, some require CPU/RAM/GPU for different durations,
others mostly Claude tokens" — naturally decomposes into reactive
(interactive `/propose`, `/discover`, `/run`) and proactive
(scheduled `/digest`, long `/iterate --chain`, `/implement` subagent
runs). Admission policy and preemption rules should follow this cut.

## Connections

- [[mixed-criticality-preemption]] — the corresponding scheduling
  mechanism: reactive preempts proactive, but proactive doesn't get
  starved.
- [[flow-level-concurrency]] — both classes share the same primitive
  (a long-lived stateful flow), differing only in priority.
- Open question: are there workloads that don't fit either
  class — e.g. periodic but interruptible, or batch-with-deadline?
  These may need a third "deferred-reactive" class.
