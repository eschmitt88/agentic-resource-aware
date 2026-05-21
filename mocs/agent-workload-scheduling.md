---
kind: moc
name: "agent workload scheduling"
status: live
added: "2026-05-21"
member_concepts:
  - reactive-vs-proactive-agent-flows
  - heterogeneous-accelerator-coordination
  - mixed-criticality-preemption
  - flow-level-concurrency
  - priority-weighted-proportional-allocation
related_mocs:
  - autonomous-research-agent-architecture
tags:
  - scheduling
  - coordinator
  - multi-tenancy
---

# Agent workload scheduling

How multiple concurrent agentic workloads share one workstation
without starving each other. The "outside view" companion to
[[autonomous-research-agent-architecture]] (which is the inside view
of a single agent).

## Members

- [[reactive-vs-proactive-agent-flows]] — the primary workload
  taxonomy. Reactive = user-initiated, low-latency. Proactive =
  background, throughput.
- [[heterogeneous-accelerator-coordination]] — placement across
  qualitatively different accelerators (CPU/iGPU/dGPU/NPU, plus
  remote LLM APIs as a "virtual accelerator").
- [[mixed-criticality-preemption]] — reactive preempts proactive,
  but slack-aware piggybacking keeps proactive alive. The
  scheduling-policy layer that turns the workload taxonomy into
  actual behavior.
- [[flow-level-concurrency]] — the runtime abstraction. Sessions /
  flows are the unit, not individual LLM calls.
- [[priority-weighted-proportional-allocation]] — concrete O(N)
  allocation algorithm (`dᵢ = λᵢ · Rᵢ · Pᵢ`, proportional + minimum +
  normalize). The arithmetic underneath the policy.

## Primary sources

- [[wei2025agent]] — Agent.xpu (Peking 2025): introduces reactive/
  proactive taxonomy, flow-level abstraction, heterogeneous
  coordination, mixed-criticality preemption.
- [[zhang2025adaptive]] — Adaptive GPU Allocation (GWU 2025):
  concrete proportional-allocation algorithm with priority and
  minimum-guarantee terms.

## Open questions

- What's the right "flow" unit for a research workstation where a
  flow might span days, include dvc pipelines, tool use, and
  external API calls? Possibly "session" (one `/iterate` chain, one
  `/discover→fetch→ingest→propose` arc) is the right granularity.
- How are priorities set and how do they drift over time?
  `zhang2025adaptive` treats them as static input; for research
  workloads they probably need to respond to deadlines, attention
  signal, and progress.
- Is there a unified currency across GPU-seconds, RAM-GB-hours, and
  LLM tokens? Current literature treats them as separate budgets.
- What's the safe-checkpoint primitive for preempting different job
  types (dvc stage boundary, LLM-turn boundary, training-step
  boundary)?

## Why this MoC is here

This is the project's load-bearing MoC. The coordinator's policy
(`coordinator/policy.py`) is its tangible deliverable, and the five
concepts here are the policy's vocabulary. Experiments under
`experiments/` should be ablations and validations of policy
variants drawn from this MoC.
