---
kind: proposal
slug: priority-weighted-admission-replay
date: 2026-05-21
status: proposed
hypothesis: "On a replayed 30 days of state.db job traffic, a priority-weighted-proportional admission policy reduces p95 reactive-job wait time by ≥30% vs the current naive policy, without dropping aggregate proactive throughput by more than 10%."
rationale: "The current coordinator admits jobs with a near-naive policy. The user's stated workload is exactly the reactive+proactive mix that [[wei2025agent]] and [[zhang2025adaptive]] address — interactive /propose, /discover, /run vs scheduled /digest, overnight /iterate. Zhang's O(N) priority-weighted-proportional algorithm is implementable in <150 lines, fits cleanly as the micro layer of the two-tier scheduling hierarchy [[du2025temporal]] already implicit in the project (budget.yaml = macro, /plan = micro), and is a near-zero-risk first experiment. Replay against the existing state.db log avoids any real-job preemption risk."
reads:
  - "[[literature/papers/zhang2025adaptive]]"
  - "[[literature/papers/wei2025agent]]"
  - "[[literature/papers/du2025temporal]]"
  - "[[concepts/priority-weighted-proportional-allocation]]"
  - "[[concepts/reactive-vs-proactive-agent-flows]]"
  - "[[concepts/two-tier-scheduling-hierarchy]]"
  - "[[mocs/agent-workload-scheduling]]"
expected_metric:
  name: p95_reactive_wait_ms
  target: 0.70  # ratio vs naive baseline; ≤ 0.70 means ≥30% reduction
  direction: lower-is-better
design_sketch:
  - "Snapshot 30 days of ~/.claude/state.db job log to a trace file (jobs, arrival times, durations, resource asks)."
  - "Tag each historical job as reactive or proactive based on the originating skill (interactive /propose, /discover, /run, /headroom → reactive; /digest, /iterate --chain, /implement subagent runs → proactive)."
  - "Implement Zhang's allocator (dᵢ = λᵢ · Rᵢ · Pᵢ, proportional with minimum-guarantee + capacity renormalization) as policy_priority_weighted.py alongside the existing policy module."
  - "Build a deterministic replay harness that feeds the trace into both policies sequentially under matched simulated capacity, recording per-job admit/defer decisions and synthesized wait times."
  - "Compare distributions: p50/p95/p99 reactive wait, aggregate proactive throughput, GPU/CPU idle, switching-cost proxy (consecutive flips between project contexts)."
  - "Iterate Pᵢ and Rᵢ assignments via grid search if the first cut doesn't hit the hypothesis."
risks:
  - "state.db may not log resource asks with enough fidelity for a real λᵢ / Rᵢ estimate — may need to back-compute from observed durations + a static per-skill resource template."
  - "Reactive vs proactive tag is derived, not native — coordinator doesn't currently emit it. Tag accuracy could limit the realism of replay."
  - "Replay-only eval ignores real contention dynamics (memory pressure, model warm-up); positive replay results don't fully guarantee live wins."
  - "Workload is small (single-user) — the algorithm's strengths over round-robin may not show up at this N, in which case the hypothesis fails for benign reasons."
related_prior: []
estimated_runtime: "3–5 h wall; coordinator+replay code; no GPU needed; ~100–300k Claude tokens"
---

# Priority-weighted admission, replay-evaluated

This is the smallest credible experiment that exercises the project's
vocabulary against a concrete algorithm. It tests whether
[[priority-weighted-proportional-allocation]] is actually worth wiring
into `coordinator/policy.py`, or whether the user's single-workstation
workload is too small for the algorithm to materially help.

The literature is unanimous on the *qualitative* point — reactive
flows starve under naive admission, and any priority-aware policy
beats round-robin. What's unclear at the user's scale (one
workstation, ≤4 concurrent jobs typical, sub-second admission
budget) is whether the *quantitative* gap is large enough to justify
the implementation cost. A replayed trace from `state.db` is the
cheapest way to find out — no risk to live jobs, deterministic to
re-run, and the dataset already exists.

A 30% p95 reduction is an aggressive target deliberately. If we
clear it, the algorithm is worth promoting to the default policy and
we can move on to the macro layer (deadline-aware priority drift,
temporal-aware warm-up). If we miss it, the result is informative
either way: either the workload is too small to benefit (settle for
the simpler current policy and put effort elsewhere), or the
algorithm is the wrong shape (probably means the
[[mixed-criticality-preemption]] piece is doing more of the work
than allocation alone, and we should rebudget).

The proactive-throughput guardrail (≤10% drop) is the
counter-hypothesis safety net: a policy that hits the p95 target by
silently starving background work is not actually a win. Both
numbers should move in the right direction together for the result
to count.

If approved, next move is `/implement` — which spawns a subagent to
scaffold the experiment via `/new-experiment` and write the replay
harness + allocator code, while this main context stays free for
discussion.
