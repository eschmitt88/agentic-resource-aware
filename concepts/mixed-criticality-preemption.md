---
kind: concept
name: "mixed-criticality preemption"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/wei2025agent.md
related_concepts:
  - reactive-vs-proactive-agent-flows
  - flow-level-concurrency
related_experiments: []
tags:
  - scheduling
  - preemption
  - real-time
---

# Mixed-criticality preemption

## Definition

A scheduling policy for workloads with mixed priority classes (e.g.
[[reactive-vs-proactive-agent-flows]]) that lets high-priority work
preempt low-priority work but uses **slack-aware piggybacking** to
keep low-priority work from starving — running proactive kernels
during reactive idle windows, at fine granularity (kernel-level or
token-step-level), so neither class loses its SLO.

Introduced in [[wei2025agent]] for LLM operator scheduling, but the
pattern is older — borrowed from real-time systems literature.

## Why it matters here

The user's resource-aware coordinator must handle exactly this mix:
an interactive `/propose` or `/discover` call should not wait behind
a long `/implement` subagent run, but the `/implement` run shouldn't
be killed either. Fine-grained preemption (suspend at safe
checkpoints, resume when interactive load drops) is the right
primitive — softer than process-level kill, harder than naive FIFO.

## Connections

- [[reactive-vs-proactive-agent-flows]] — the two priority classes
  this policy operates on.
- [[flow-level-concurrency]] — preemption happens at flow boundaries
  (between LLM steps), not mid-tensor-op.
- Open question: what's the safe-checkpoint primitive for the user's
  workloads? For a dvc pipeline it's per-stage; for an LLM session
  it's between turns; for a training run it's at the next gradient
  step. The coordinator needs a per-job declaration.
