---
kind: design-doc
title: "System requirements — agentic resource-aware coordinator"
status: draft
added: "2026-05-22"
revisions:
  - "rev-1 2026-05-22: initial draft"
  - "rev-2 2026-05-22: autonomy-first vision; operating principles; R10–R14, N7–N11; taxonomy recast to blocking/background"
supersedes: []
superseded_by: []
related_mocs:
  - agent-workload-scheduling
  - autonomous-research-agent-architecture
---

# System requirements — agentic resource-aware coordinator

Draft. Open questions are marked `[Q-N]`. Literature gaps are
marked `[GAP]`.

## 1. Vision

**The user is present for a small fraction of the time. Autonomy is
the design center, not a feature.**

The system runs on one workstation hosting multiple research
projects. Each project has a stated goal. Most of the time the user
is away, and agents are advancing those goals using the available
tokens and compute — exploring, ingesting, proposing, implementing,
iterating — without supervision. When the user is present, their
attention is the scarcest resource in the system and the most
expensive to consume.

The coordinator's job is to make that mode work: keep the agents
fed with the resources they need, prevent collisions when several
goals want the same hardware, surface only the work-product the
user actually has to see, and stay out of the way otherwise.

Inside-view (what makes one agent good) lives in
[[autonomous-research-agent-architecture]]. This doc is the
outside-view contract.

## 2. Operating principles

The philosophical commitments that constrain every requirement
below. When two requirements conflict, principles decide.

- **P1. Autonomy by default.** Agents advance toward stated goals
  using available resources. They do not gate work on user input
  when exploration could disambiguate. They stop only on
  goal-shift, irreversible action, or genuine strategic forks.
- **P2. User attention is the scarcest resource.** Every question
  to the user, every dashboard interrupt, every summary must
  justify its cost in attention. Default action: don't ask.
- **P3. Principles over prescription.** Skills follow simple
  principles, not rigid scripts. Hard contracts apply only where
  the system has mechanical state to protect — the job database,
  the resource API, the audit log. Everywhere else, judgment.
- **P4. Token budget is a target, not a cap.** Default target
  ~75% utilization of both windows (5-hour rolling, weekly Monday
  reset). Headroom is for spikes, not a savings account. Burn
  budget toward goals; don't hoard.
- **P5. Single-tenant default; multi-tenant when needed.** One
  active high-priority job uses the box. Two or more concurrent
  high-priority jobs trigger active resource partitioning.
- **P6. Skills stay simple; the orchestrator carries complexity.**
  A skill is a short prompt with a few tools. The orchestrator has
  the full state — telemetry, history, queue, budget, project
  goals — and the judgment to use it.
- **P7. Summaries are the user interface.** Reports, `/wrap`
  entries, dashboard panels: concise, no jargon, no lingo, fit for
  someone who hasn't seen the work, readable in 30 seconds.

## 3. Functional requirements

What the system must DO. Each is anchored to a principle, a
literature concept, or both. `[MAY]` = optional.

- **R1. Workload classification.** Every session declares itself
  as **blocking** or **background** at start.
  - *Blocking*: user is actively waiting OR a hard deadline /
    external trigger applies. Highest priority when set. Expected
    to be rare under P1.
  - *Background*: autonomous goal-pursuit. The default. Includes
    sub-shapes (goal-driven exploration, scheduled maintenance,
    chained experiments) that the policy may treat differently
    but the orchestrator handles uniformly.

  Anchored in [[reactive-vs-proactive-agent-flows]] from
  [[wei2025agent]] (taxonomy recast: "reactive" assumed user
  presence; "blocking" is the more accurate label given P1).

- **R2. Resource declaration.** Every session declares its
  expected footprint at start: tokens, GPU minutes, vRAM, RAM,
  disk, wall-time. Estimates may revise mid-flight. Anchored in
  [[priority-weighted-proportional-allocation]] from
  [[zhang2025adaptive]].

- **R3. Admission decision.** Coordinator decides admit / defer /
  queue from declared footprint + priority + current availability
  + global budget shape (P4). Under P4, when budget is plentiful,
  default-admit background sessions; gate only when constrained.

- **R4. Mixed-criticality preemption.** Blocking sessions may
  preempt background sessions at safe checkpoints; preempted work
  resumes when load drops, with slack-aware piggybacking. Anchored
  in [[mixed-criticality-preemption]] from [[wei2025agent]].

- **R5. Auditability.** Every session and decision lands in a
  persistent log (`jobs`, `decisions`). Without this no policy can
  be evaluated and the framework is unfalsifiable.

- **R6. Pluggable policy.** Admission/preemption logic is a
  swappable module (`coordinator/policy.py`). Naive,
  priority-weighted, temporal-aware, learned — all sit behind the
  same interface. Anchored in
  [[priority-weighted-proportional-allocation]] and
  [[two-tier-scheduling-hierarchy]].

- **R7. Manual override.** User can always force-admit /
  force-defer. Logged as `verdict='override'`, never gated.

- **R8. `[MAY]` Temporal awareness.** Coordinator may pre-warm
  environments or defer expensive work based on historical demand
  patterns and budget-window position (5-h, weekly). Anchored in
  [[temporal-aware-scheduling]] from [[du2025temporal]]. Optional
  pending sufficient history.

- **R9. Two-tier separation.** Slow strategic decisions in
  declarative config; fast tactical decisions in code. Already
  shaped as `budget.yaml` (macro) + `/plan` (micro). Anchored in
  [[two-tier-scheduling-hierarchy]] from [[du2025temporal]].

- **R10. Goal declaration.** Each project declares one or more
  goals in a structured form the coordinator and skills can read.
  Goals are the input to autonomous exploration (R11). Without
  declared goals, "advance toward the user's goal" is undefined.
  See `[Q-9]` for goal-model shape.

- **R11. Autonomous exploration.** Under P1+P4, when budget
  allows and a session encounters uncertainty, it explores
  candidate paths against the project goal rather than stopping
  to ask the user. The exploration cost is bounded by remaining
  budget; the alternatives explored are journaled for review.

- **R12. Open-questions surface.** The system maintains, per
  project, a list of open questions the agent has queued for
  asynchronous user review. The user can dismiss, answer, or
  comment; agents read responses on next session. Surface is the
  dashboard (new template, see `[Q-10]`). Mirrors a project-local
  file (`docs/open_questions.md`) for grep-ability.

- **R13. Summary discipline.** Every user-facing artifact
  (`/wrap` entry, dashboard panel, work summary, end-of-chain
  report) obeys P7. The `/wrap` skill and dashboard templates
  enforce this by shape; agents enforce it in tone.

- **R14. Multi-tenant compute management.** When ≥2 high-priority
  workloads need CPU / GPU / RAM concurrently, coordinator
  actively partitions (cgroups, CUDA MPS / MIG, RAM ceilings).
  Single-tenant default — one active job takes the box. Anchored
  in [[heterogeneous-accelerator-coordination]] from
  [[wei2025agent]] + [[zhang2025adaptive]]. See `[Q-12]`.

## 4. Non-functional requirements

How well, under what constraints.

- **N1. Decision latency.** Admission decision < 100 ms.
- **N2. Footprint.** Single-process SQLite. No distributed
  scheduler. No daemons beyond what already runs.
- **N3. Zero-config default.** New project works with the
  coordinator out of the box from
  `~/.claude/templates/project/budget.yaml`. No per-project tuning
  required.
- **N4. Reversibility.** Every coordinator decision logged with a
  human-readable reason. Nothing destructive. Overrides logged,
  not gated.
- **N5. Graceful non-compliance.** A skill that skips the
  declaration protocol still runs; telemetry still flows. It just
  doesn't participate in admission. This is how the system
  bootstraps without big-bang skill rewrites.
- **N6. Cache-warmth.** The 5-min prompt cache (Anthropic API) is
  a real resource. Decisions that cause cache misses on
  user-facing sessions cost more than they save.
- **N7. Token budget shape.** Target ~75% utilization of both
  windows (5-h rolling, weekly Monday reset). `ccusage.py` is the
  source of truth for current consumption. Coordinator tunes
  admission aggressiveness against window position: aggressive
  early in window, conservative late.
- **N8. Question economy.** Agent → user questions are
  rate-limited and weighted by importance. Cheap, frequent
  questions are a bug. Each question must answer "what about this
  cannot be resolved by exploration in the remaining budget?"
  before it is sent. Open questions accumulate on the dashboard
  (R12), not in chat.
- **N9. Skill simplicity.** Skill prompts stay under ~200 lines.
  Complexity belongs in the orchestrator, which has full state.
  Multi-step skill machinery should consolidate, not proliferate.
- **N10. Memory-aware ML scheduling.** For workloads that train
  or run inference, coordinator can declare per-job vRAM and RAM
  ceilings, and partition (MIG / MPS / cgroups) when concurrent
  ML jobs run. A solo ML job is allowed to grab everything; two
  concurrent jobs are not.
- **N11. Orchestrator data access.** The component making
  scheduling decisions has read access to all telemetry —
  `token_events`, `hardware_samples`, `ccusage` window state,
  per-project goal + open-questions list, recent job history.
  Decisions must be explainable from that data alone.

## 5. Out of scope (non-goals)

- Multi-machine scheduling. One workstation.
- Adversarial scheduling. Single trusted user.
- Replacing the user as final arbiter. Coordinator recommends; the
  user (or agents on the user's behalf, per P1) decides.
- LLM-API quota logic beyond `ccusage.py`.
- General-purpose job scheduling (Slurm, k8s, Ray). Opinionated
  for LLM-agent workloads.

## 6. Current state vs. requirements

Snapshot of `~/.claude/state.db` + coordinator + dashboard as of
2026-05-22.

| Req | Status | Evidence / gap |
|---|---|---|
| R1 (blocking/background) | **Missing** | No flow-class column on `jobs`; no skill emits a tag. Taxonomy recast in rev-2; original "reactive/proactive" not implemented either. |
| R2 (resource declaration) | **Partial — schema only** | `est_tokens / est_gpu_minutes / est_vram_gb` exist; only `/plan` and `/headroom` reference the coordinator. |
| R3 (admission decision) | **Partial — schema only** | `decisions` table exists; **0 rows after 4 weeks**. |
| R4 (preemption) | **Missing** | No mechanism; no safe-checkpoint primitive defined. |
| R5 (auditability) | **Partial** | `token_events` (242), `hardware_samples` (16,402) flowing. `jobs` / `decisions` empty. |
| R6 (pluggable policy) | **Shape exists** | `coordinator/policy.py` module present. Implementation depth not yet verified. |
| R7 (manual override) | **Unknown** | Read `policy.py` + `/plan` to confirm. |
| R8 (temporal awareness) | **Not started** | Optional. |
| R9 (two-tier) | **Implemented in shape** | `budget.yaml` + `/plan`. Macro hand-edited, not learned. |
| R10 (goal declaration) | **Missing** | No goal schema. Project `CLAUDE.md` has a "What this project is about" field but it's prose, not structured. |
| R11 (autonomous exploration) | **Missing** | Agents stop on uncertainty today. Pre-rev-2 the design assumed they should. |
| R12 (open-questions surface) | **Missing** | Dashboard templates exist for queue / candidates / concepts / literature / projects, none for open-questions or work-product comments. |
| R13 (summary discipline) | **Partial — by skill** | `/wrap` shape enforces Did/Findings/Next; dashboard summaries inconsistent. |
| R14 (multi-tenant compute) | **Missing** | No cgroups / MIG / MPS integration. |
| N1 (< 100 ms) | **Unknown** | No latency benchmark. |
| N2 (single SQLite) | **Met** | |
| N3 (zero-config) | **Met** | |
| N4 (reversibility) | **Met** | |
| N5 (graceful non-compliance) | **Met by accident** | Telemetry runs independent of declaration. |
| N6 (cache-warmth) | **Not considered** | |
| N7 (token budget shape) | **Partial — caps only** | `budget.yaml` caps `max_tokens`; no window-aware shaping; `ccusage.py` exists but not wired to admission. |
| N8 (question economy) | **Cultural — no mechanism** | |
| N9 (skill simplicity) | **Mostly met** | Most skills ≤150 lines. `/implement` is longest, but inherent. |
| N10 (memory-aware ML) | **Missing** | |
| N11 (orchestrator data access) | **Met for telemetry; missing for goals + open-questions** | Telemetry exists; goal + open-question schemas don't. |

**Headline gaps**:
1. **R1+R2+R3+R5 dead zone** — no session declares → no decision →
   empty audit → no policy can be evaluated. Bootstrap problem.
2. **R10+R11+R12 autonomy gap** — no goals declared, no autonomous
   exploration mode, no open-questions surface. The system isn't
   set up for the autonomy P1 calls for.
3. **N7 missing window awareness** — `ccusage.py` knows the
   windows; admission ignores them.

## 7. Open questions

- **Q-1.** Canonical *unit of work* — "session" vs "job" vs "flow"?
  Provisional: session = one Claude Code invocation (has session_id);
  job = a declared work-unit inside a session (may be multiple per
  session, e.g. `/iterate --chain` files N jobs).
- **Q-2.** How skills *declare* — explicit `declare_job` call,
  hook-driven inference, or YAML frontmatter? Tilt toward
  frontmatter + hook (skills stay simple per N9; orchestrator does
  the work per P6).
- **Q-3.** Safe-checkpoint primitive per job kind?
- **Q-4.** How priorities are set — static default, user flag,
  learned, deadline?
- **Q-5.** Unified currency across GPU-seconds and LLM tokens?
- **Q-6.** Macro-layer: hand-edited (today) or learned predictor?
- **Q-7.** Session ↔ job relationship: 1:1, 1:N, or independent?
- **Q-8.** `[GAP]` UX of admission deferrals + asynchronous
  open-questions review. Need ≥1 paper on interactive scheduling
  perception or async-feedback UX.
- **Q-9.** Goal model — per-project, per-session, hierarchical?
  Editable mid-flight by the agent?
- **Q-10.** Open-questions surface shape — dashboard template + a
  project-local `docs/open_questions.md` file? Both? Source of
  truth?
- **Q-11.** Question-economy enforcement — hard cap (≤N
  questions/window) or pure heuristic? Or a "question budget" that
  trades off against token budget?
- **Q-12.** Memory partitioning mechanism for concurrent ML —
  CUDA MPS, MIG, container cgroups, hard OOM? Trade-offs and
  fall-throughs not yet evaluated.

## 8. Anchored in literature

- [[hambardzumyan2026aira]] — structural bottlenecks; HCE; async
  multi-GPU pool; ReAct.
- [[wei2025agent]] — flow taxonomy (rev-2 recast as
  blocking/background); mixed-criticality preemption; flow-level
  concurrency; heterogeneous accelerator coordination.
- [[zhang2025adaptive]] — concrete O(N)
  priority-weighted-proportional allocation.
- [[du2025temporal]] — two-tier hierarchy; temporal-aware demand;
  switching-cost minimization.

## 9. What this doc is and is not

**Is**: the system's contract with itself. RFC-style; revised in
place as conventions stabilize and constraints surface.

**Is not**: a roadmap or task list. Conversion to actionable work
happens in `02-session-conventions.md` (the protocol) and in
`docs/decisions/NNNN-*.md` (lightweight ADRs).
