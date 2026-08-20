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
  - temporal-aware-scheduling
  - two-tier-scheduling-hierarchy
  - collaboration-topology-tradeoff
  - worktree-isolated-parallel-search
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
- [[temporal-aware-scheduling]] — adds prediction: warm-up affinity
  on the short scale, switching-cost minimization on the long scale.
  Lifts the policy from reactive to anticipatory.
- [[two-tier-scheduling-hierarchy]] — splits slow strategic
  decisions from fast tactical ones. The architectural skeleton that
  carries all the above concepts; already implicit in the project as
  `budget.yaml` (macro) + `/plan` (micro).
- [[collaboration-topology-tradeoff]] — *how* the admitted agents are
  organized is itself an allocation decision: subagents buy throughput
  and resilience, expert teams buy depth at a deliberation cost that
  comes straight out of the budget. Turns "how many jobs" into "how many
  jobs, arranged how".
- [[worktree-isolated-parallel-search]] — the isolation primitive that
  makes parallel candidates safe to run at all, plus a four-state
  proposal taxonomy (proposal failure / preflight failure / training
  crash / success) that makes failure legible per candidate.

## Primary sources

- [[wei2025agent]] — Agent.xpu (Peking 2025): introduces reactive/
  proactive taxonomy, flow-level abstraction, heterogeneous
  coordination, mixed-criticality preemption.
- [[zhang2025adaptive]] — Adaptive GPU Allocation (GWU 2025):
  concrete proportional-allocation algorithm with priority and
  minimum-guarantee terms.
- [[du2025temporal]] — TORTA (Shenzhen UAT + China Mobile 2025):
  two-tier RL + optimal-transport scheduler with temporal awareness;
  data-center scale but two-tier framing transfers directly.
- [[shen2026empirical]] — Multi-agent collaboration for automated
  research (UTS + SUSTech 2026): the closest published match to this
  project's setup — one box, fixed per-candidate budget, worktree
  isolation — and the empirical answer to subagents-vs-teams.
- [[li2026spend]] — BAVT (UBC + Vector 2026): budget-conditioned
  exploration/exploitation *within* an admitted job, and a candidate
  answer to the unified-currency question below.

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
  LLM tokens? Current literature treats them as separate budgets —
  though [[li2026spend]] offers the cheapest plausible answer: don't
  convert, just reduce to the *tightest* normalized remaining ratio,
  `r = min(used_i / ceiling_i)`, and condition the policy on that one
  scalar. Untested above trajectory scale.
- What observable, available at **admission time**, predicts a deep
  architectural refactor vs a broad shallow sweep? That is the routing
  signal [[collaboration-topology-tradeoff]] needs, and neither
  [[shen2026empirical]] nor [[yang2026toward]] supplies it — both name
  routing as future work.
- What's the safe-checkpoint primitive for preempting different job
  types (dvc stage boundary, LLM-turn boundary, training-step
  boundary)?

## Why this MoC is here

This is the project's load-bearing MoC. The coordinator's policy
(`coordinator/policy.py`) is its tangible deliverable, and the nine
concepts here are the policy's vocabulary. Experiments under
`experiments/` should be ablations and validations of policy
variants drawn from this MoC.
