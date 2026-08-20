---
kind: paper
title: "An Empirical Study of Multi-Agent Collaboration for Automated Research"
authors:
  - Yang Shen
  - Zhenyi Yi
  - Ziyi Zhao
  - Lijun Sun
  - Dongyang Li
  - Chin-Teng Lin
  - Yuhui Shi
institutions:
  - "University of Technology Sydney"
  - "Southern University of Science and Technology"
  - "Shenzhen Technology University"
  - "Tongji University"
year: 2026
venue: arXiv 2603.29632v2 (LNCS-formatted preprint; UTS + SUSTech)
peer_reviewed: unknown
url: https://arxiv.org/abs/2603.29632
code_url: https://github.com/yshenfab/MAAR
citations: null
source: "raw/papers/shen2026empirical.pdf"
added: "2026-08-20"
relevance: 4
credibility: 3
status: skimmed
related_concepts:
  - collaboration-topology-tradeoff
  - worktree-isolated-parallel-search
  - fixed-budget-evaluation-protocol
  - evolutionary-search-over-solutions
related_experiments: []
tags:
  - multi-agent
  - topology
  - fixed-time-budget
  - worktree-isolation
  - autoresearch
---

# An Empirical Study of Multi-Agent Collaboration for Automated Research

## TL;DR

Head-to-head comparison of three autoresearch topologies — single agent,
**subagents** (parallel exploration, post-hoc consolidation by a
coordinator), and **agent teams** (fixed-role experts handing off before
execution) — on the same nanochat-style LM optimization task under strict
per-candidate training budgets (300 s / 600 s). Subagents are the
resilient high-throughput shallow searcher; teams are fragile but produce
the deep, coupled architectural changes. The recommendation is a *routed*
topology that switches by task complexity.

## Claims

- Topology is a **resource-allocation decision, not a style choice**:
  under a short budget, deliberation cost (context + API calls spent
  before any training run) directly subtracts from the number of
  candidates evaluated.
- Subagent mode is a stable, high-throughput empirical search engine —
  isolated workers mean individual failures don't contaminate the run —
  but its proposals lack diversity and collapse into greedy
  single-dimension hyperparameter squeezing.
- Agent teams achieve deeper alignment and genuinely coupled, multi-part
  patches, at the cost of higher operational fragility from multi-author
  code generation.
- The single-agent baseline plateaus fast: it improves on obvious knobs,
  then cannot escape local optima, and fails often on complex zero-shot
  edits.
- Future autoresearch systems should **dynamically route** — subagents
  for broad sweeps, specialist teams for deep algorithmic change.

## Methods

- **Testbed**: Karpathy's `autoresearch` loop, target = a single-file
  GPT-style `train.py`; metric = validation bits-per-byte on a fixed
  ClimbMix shard; **Git worktree isolation** per candidate, a
  deterministic Search/Replace patch contract, preflight static checks,
  and an explicit global memory shared across rounds.
- **Budgets**: per-candidate training time `T ∈ {300 s, 600 s}`, applied
  per training job (not per round). LLM inference, preflight checks and
  debugging are excluded from the budget — an important accounting
  caveat.
- **Hardware**: adapted from the original H100 setting down to a single
  RTX 3090 (24 GB) by cutting EVAL_TOKENS 4× and TOTAL_BATCH_SIZE 4×.
- **Agents**: glm-4.6v workers, glm-4.7 coordinator/engineer. Subagent
  mode = 3 workers + 1 coordinator proposal per round; team mode = 3
  specialists (architecture, optimizer/schedule, efficiency/memory) in a
  fixed six-turn relay, with an isolated Engineer agent as crash-repair
  fallback.
- **Instrumentation**: rounds *and* proposal counts, failure counts,
  cumulative training time; every proposal classified as proposal
  failure / preflight failure / training crash / training success.

## Results

- At T=300 s over 50 rounds: subagents land **7** effective
  improvements, teams only **3**.
- Subagent improvements are narrow — e.g. iteratively shrinking the MLP
  expansion ratio 4× → 0.75× across rounds, ignoring other dimensions.
- Team improvements are structurally richer — a single patch that
  simultaneously changed the attention window pattern (SSLL → SLSL), the
  LR warmdown schedule (0.50 → 0.30), and the value-embedding vocab size.
- Stability ordering: subagents most stable (isolated worktrees contain
  failures), single agent suffers heavily on complex zero-shot edits,
  teams pay a crash tax from multi-author patches.
- Summary table contrasts subagents (context returns to caller, report to
  main, manager coordination, simple tasks, lower token cost) against
  agent teams (fully independent context, agent-to-agent messaging,
  shared task list, complex work, higher token cost).

## Critique / open questions

- Single task, single dataset, single GPU, one model family (GLM), and
  the headline counts are 7-vs-3 out of 50 rounds — suggestive, not
  conclusive. No seed replication, no variance reported.
- **The budget excludes LLM inference, preflight, and debugging.** That
  is exactly the cost this project must account for: on a token-bound
  Max plan, deliberation *is* the scarce resource, so the paper's fixed
  training-time budget understates the team topology's true cost. Under
  a token budget the subagent advantage should be larger, not smaller.
- "Routed topology" is proposed but not implemented or evaluated — the
  routing signal (task complexity) is left undefined, which is precisely
  the admission-policy question this project cares about.
- The independent replication of this project's own runtime discipline
  (worktree isolation per candidate, global memory, deterministic patch
  contract) is the most reassuring part of the paper — it is also the
  part most likely to be convergent design rather than a validated
  finding.

## Trust signals

- **Credibility:** 3 — mid-tier university group, but the testbed and
  code are released (`yshenfab/MAAR`), the protocol is explicit enough to
  rerun, and the failure taxonomy is honestly reported. Held at 3 by the
  unknown peer-review status (LNCS-formatted preprint), the single-task
  scope, and the absence of seed replication.

## Follow-up

- **Relevance:** 4 — the closest published match to this project's own
  setup (one workstation, fixed budget, worktree isolation) and the
  direct empirical answer to the "coordinated agents vs independent
  agents on a scheduler" question. Seeds
  [[collaboration-topology-tradeoff]] and
  [[worktree-isolated-parallel-search]]. Not a 5 because the evidence
  base is one task on one GPU.
- Derived experiment worth running here: re-run the comparison with the
  budget denominated in **tokens** rather than training seconds, which is
  the binding constraint on this box, and see whether the 7-vs-3 gap
  widens.
- Feeds the routing question directly: if the coordinator must pick a
  topology at admission time, what observable predicts "complex
  architectural refactor" vs "broad shallow sweep" *before* the work
  starts?
