---
kind: design-doc
title: "System requirements — agentic resource-aware coordinator"
status: draft
added: "2026-05-22"
supersedes: []
superseded_by: []
related_mocs:
  - agent-workload-scheduling
  - autonomous-research-agent-architecture
---

# System requirements — agentic resource-aware coordinator

This document is a **draft**. It exists to write down what the system
is supposed to do and what sessions promise it, *before* we evaluate
competing algorithms or policies. Open questions are marked
explicitly; gaps in the literature corpus are flagged with `[GAP]`.

## 1. Vision

One workstation hosts multiple concurrent research projects with an
unpredictable workload mix:

- **Interactive work**: user is present, expects sub-second feedback
  from skills like `/discover`, `/propose`, `/run`, `/headroom`.
- **Background work**: cron-scheduled `/digest`, overnight
  `/iterate --chain`, `/implement` subagent runs that can take hours.
- **Mixed-profile resource demand**: some sessions are token-heavy
  (paper-research, ideation), some GPU-heavy (training, eval), some
  CPU-heavy (data scraping), some idle (waiting on external APIs).

The coordinator's job is to make admission, throttling, and
preemption decisions so interactive sessions stay responsive and
background work makes steady progress, without the user manually
orchestrating which project gets the GPU when.

This is the *outside view* of multiple agents sharing one
workstation. The *inside view* of any one agent (its operator,
search, evaluation) is anchored in
[[autonomous-research-agent-architecture]] and not this document's
concern.

## 2. Functional requirements

What the system must DO. Each is anchored to an ingested concept or
paper. `[MAY]` means optional / nice-to-have, default is "must."

- **R1. Workload classification.** Every session must declare itself
  as **reactive** or **proactive** at start. Reactive = user-present,
  latency-critical. Proactive = background, throughput-critical.
  Anchored in [[reactive-vs-proactive-agent-flows]] from
  [[wei2025agent]].

- **R2. Resource declaration.** Every session must declare its
  expected resource footprint at start: tokens, GPU minutes, RAM,
  disk, wall-time. Estimates can be revised mid-flight. Anchored in
  [[priority-weighted-proportional-allocation]] (the algorithm needs
  `Rᵢ` to work) and [[zhang2025adaptive]].

- **R3. Admission decision.** The coordinator decides admit / defer /
  queue for each session based on declared footprint + priority +
  current resource availability + global budget. Decision is logged
  with a reason.

- **R4. Mixed-criticality preemption.** Reactive sessions may
  preempt proactive sessions at safe checkpoints; preempted proactive
  work resumes when reactive load drops, with slack-aware piggybacking
  to avoid starvation. Anchored in [[mixed-criticality-preemption]]
  from [[wei2025agent]].

- **R5. Auditability.** Every session and decision lands in a
  persistent log (`jobs`, `decisions` in `state.db`). Without this,
  no policy can be evaluated post-hoc and the framework is unfalsifiable.

- **R6. Pluggable policy.** The admission/preemption logic is a
  swappable component (`coordinator/policy.py`). Naive,
  priority-weighted, temporal-aware, learned — all sit behind the
  same interface. Anchored in
  [[priority-weighted-proportional-allocation]] and
  [[two-tier-scheduling-hierarchy]].

- **R7. Manual override.** The user can always force-admit or
  force-defer a session, with the decision logged as `verdict='override'`.

- **R8. `[MAY]` Temporal awareness.** The coordinator may pre-warm
  environments or defer expensive work based on historical demand
  patterns. Anchored in [[temporal-aware-scheduling]] from
  [[du2025temporal]]. Optional because it requires a non-trivial
  amount of history before any predictor is useful — bootstrap
  problem.

- **R9. Two-tier separation.** Slow strategic decisions (budgets,
  model roles, ceilings) live in declarative config; fast tactical
  decisions (per-session admission, preemption) live in code.
  Already partially implemented as `budget.yaml` + `/plan`. Anchored
  in [[two-tier-scheduling-hierarchy]] from [[du2025temporal]].

## 3. Non-functional requirements

How well, and under what constraints.

- **N1. Decision latency.** Admission decision must complete in
  < 100 ms. Interactive sessions must not stall on the coordinator.

- **N2. Footprint.** Single-process SQLite is sufficient. No
  distributed scheduler, no extra daemons beyond what already exists.

- **N3. Zero-config default.** A newly scaffolded project must work
  with the coordinator out of the box, with sensible defaults from
  `~/.claude/templates/project/budget.yaml`. No per-project tuning
  required to participate.

- **N4. Reversibility.** Every coordinator decision is logged with a
  human-readable reason. Nothing is destructive. Force-overrides are
  logged, not gated.

- **N5. Graceful non-compliance.** If a skill skips the declaration
  protocol, the system degrades gracefully — telemetry from
  `token_events` and `hardware_samples` is still collected, and the
  session can still run, but it doesn't participate in admission.
  This is how the system bootstraps: not every skill files jobs on
  day one.

- **N6. Cache-warmth.** The 5-min prompt cache (Anthropic API) is a
  real resource — coordinator decisions that cause cache misses for
  interactive sessions cost more than they save. Decisions that
  affect interactive sessions should prefer cache-warm continuations
  over cold restarts.

## 4. Out of scope (explicit non-goals)

- **Multi-machine scheduling.** One workstation. If we ever need
  more, that's a different project.
- **Adversarial scheduling.** Single trusted user; no QoS classes
  for untrusted tenants.
- **Replacing the user as final arbiter.** The coordinator
  *recommends*; the user (via `/plan` invocation, override flag, or
  ignoring it) decides.
- **LLM-API quota tracking beyond `ccusage.py`.** Anthropic's plan
  tier rules are owned by `ccusage.py`; the coordinator reads from
  it, doesn't duplicate the logic.
- **General-purpose job scheduling** (Slurm, k8s, Ray). The
  coordinator is opinionated about LLM-agent workloads specifically.

## 5. Current state vs. requirements

Snapshot of `~/.claude/state.db` and the coordinator code as of
2026-05-22:

| Requirement | Status | Evidence / gap |
|---|---|---|
| R1 (reactive/proactive tag) | **Missing** | No `flow_class` column on `jobs`; no skill emits the tag. |
| R2 (resource declaration) | **Partial — schema only** | `est_tokens / est_gpu_minutes / est_vram_gb` columns exist; only `/plan` and `/headroom` skills reference the coordinator. |
| R3 (admission decision) | **Partial — schema only** | `decisions` table exists; **0 rows after 4 weeks**. |
| R4 (preemption) | **Missing** | No preemption mechanism; no safe-checkpoint primitive defined. |
| R5 (auditability) | **Partial** | `token_events` (242 rows) and `hardware_samples` (16,402 rows) are flowing. `jobs` and `decisions` are not. |
| R6 (pluggable policy) | **Implemented** | `coordinator/policy.py` exists as a module. Verify on next pass. |
| R7 (manual override) | **Unknown** | Need to read `policy.py` and `/plan` skill. |
| R8 (temporal awareness) | **Not started** | Optional; no predictor or trace analysis exists. |
| R9 (two-tier separation) | **Implemented in shape** | `budget.yaml` (macro) + `/plan` (micro) exists. Macro is hand-edited, not learned. |
| N1 (< 100 ms) | **Unknown** | No latency measurement of `/plan` round-trip. |
| N2 (single SQLite) | **Met** | Already this. |
| N3 (zero-config) | **Met** | Template provides default `budget.yaml`. |
| N4 (reversibility) | **Met** | All writes via writers.py, none destructive. |
| N5 (graceful degradation) | **Met by accident** | Telemetry runs independent of job declaration. |
| N6 (cache-warmth) | **Not considered** | No mechanism today. |

The headline gap is **R1+R2+R3+R5 form a connected dead zone** — no
session declares itself, so no admission decision is made, so the
audit trail is empty, so no policy can be evaluated. Until sessions
start declaring (R1+R2), nothing downstream works. That's the
bootstrap problem.

## 6. Open questions

These are deliberately not answered here — they're the inputs to the
**conventions** doc (`02-session-conventions.md`) and to follow-up
research. Marked `[Q-N]` for cross-reference.

- **Q-1.** What's the canonical *unit of work* — "session," "job,"
  "flow," or some hybrid? `wei2025agent` says "flow"; the existing
  `state.db` says "job"; a `/iterate --chain` invocation contains
  many of either.

- **Q-2.** How do skills *declare* themselves to the coordinator?
  Explicit `declare_job` call at skill start? Hook-driven inference
  from tool calls? A YAML frontmatter field on every skill?

- **Q-3.** What's the safe-checkpoint primitive for preemption per
  job kind? DVC stage boundary, LLM turn boundary, training-step
  boundary — these are not interchangeable.

- **Q-4.** How are priorities set? Static per-skill default,
  user-flag override, learned from outcomes, deadline-driven?
  [[temporal-aware-scheduling]] notes priorities probably need to
  drift over time.

- **Q-5.** Is there a unified currency across GPU-seconds and LLM
  tokens? The literature treats them as separate budgets; the user's
  setup has both as real costs.

- **Q-6.** Macro-layer: hand-edited config (today's `budget.yaml`)
  or learned predictor? The trade-off is the bootstrap problem (a
  learned predictor needs history) vs. ongoing maintenance burden.

- **Q-7.** What's the relationship between a "session" (Claude Code
  session, has a `session_id`) and a "job" (coordinator's
  declared-resource unit)? 1:1, 1:N, or independent axes? The
  current `token_events` schema treats them as the session being
  primary; `jobs` treats them as separable.

- **Q-8.** `[GAP]` Literature corpus is heavy on infrastructure /
  systems papers and light on human-factors / UX work — how does a
  user *experience* admission deferrals? Need to fetch at least one
  paper on interactive scheduling perception or borrow from
  real-time-system UX literature.

## 7. Anchored in literature

- [[hambardzumyan2026aira]] — names the three structural bottlenecks
  that bind agent performance; HCE protocol; async multi-GPU pool.
- [[wei2025agent]] — reactive/proactive flow taxonomy;
  mixed-criticality preemption; flow-level concurrency as a runtime
  abstraction; heterogeneous accelerator coordination.
- [[zhang2025adaptive]] — concrete O(N)
  priority-weighted-proportional allocation algorithm
  (`dᵢ = λᵢ · Rᵢ · Pᵢ` + minimum guarantee + capacity normalize).
- [[du2025temporal]] — two-tier scheduling hierarchy (macro RL+OT,
  micro server-selection); temporal-aware demand prediction;
  switching-cost minimization.

## 8. What this document is and is not

It **is**: a draft of the system's contract with itself. It will be
revised — RFC-style — as conventions are worked out and constraints
are discovered.

It **is not**: a roadmap or a task list. Conversion to actionable
work happens in `02-session-conventions.md` (the protocol) and in
`docs/decisions/NNNN-*.md` (lightweight ADRs for specific calls).
