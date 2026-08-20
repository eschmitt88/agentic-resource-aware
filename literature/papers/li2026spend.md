---
kind: paper
title: "Spend Less, Reason Better: Budget-Aware Value Tree Search for LLM Agents"
authors:
  - Yushu Li
  - Wenlong Deng
  - Jiajin Li
  - Xiaoxiao Li
institutions:
  - "University of British Columbia"
  - "Vector Institute"
year: 2026
venue: arXiv 2603.12634v1 (UBC + Vector Institute)
peer_reviewed: false
url: https://arxiv.org/abs/2603.12634
code_url: null
citations: null
source: "raw/papers/li2026spend.pdf"
added: "2026-08-20"
relevance: 4
credibility: 3
status: skimmed
related_concepts:
  - budget-conditioned-exploration-exploitation
  - two-tier-scheduling-hierarchy
  - evolutionary-search-over-solutions
related_experiments: []
tags:
  - budget-aware
  - tree-search
  - test-time-scaling
  - token-budget
  - exploration-exploitation
---

# Spend Less, Reason Better: Budget-Aware Value Tree (BAVT)

## TL;DR

A training-free inference-time framework that runs multi-hop agent
reasoning as a search tree and makes the **remaining budget ratio the
exploration/exploitation knob**: node-selection weights are
`V(n)^(1/r_t)` where `r_t` is the fraction of tool/token budget left, so
selection anneals from broad sampling to greedy exploitation as the
budget drains. Under a 5-tool-call budget it matches or beats parallel
sampling given 20 calls — a 4× compute saving.

## Claims

- Current agents "treat compute as an abundant resource" and burn budget
  on dead-end trajectories; the failure is not a lack of compute but a
  lack of **mid-execution** budget control.
- Trajectory-level budget awareness (e.g. putting the remaining budget in
  the prompt, as BATS does) is too coarse: it cannot abandon a failing
  trajectory in flight, so agents silently exhaust budget on unpromising
  directions.
- Standard UCB is the wrong exploration rule under a *depleting* budget —
  it assumes an unbounded horizon. Budget ratio as a scaling exponent is
  the parameter-free replacement.
- LLM self-evaluation is overconfident in absolute terms but usable in
  **relative** terms: scoring the residual value delta of a step
  (marginal progress) rather than absolute state quality is what makes
  pruning reliable.
- Intelligent budget management beats brute-force scaling: better
  allocation dominates more allocation.

## Methods

- Reasoning as a dynamic tree: nodes = intermediate states, edges =
  actions/tool calls; single LLM backbone serves as actor and critic.
- **Effective remaining budget** `r_t = min(b_tool,t/B_tool,
  b_token,t/B_token)` — the *tighter* of two independent resource
  ceilings governs the policy.
- **Dynamic exponent** `α_t = 1/r_t`; selection weight
  `w_i = V(n_i)^{α_t}`, normalized to a sampling distribution. Full
  budget ⇒ α≈1 ⇒ near-uniform exploration; near-exhausted ⇒ α large ⇒
  argmax-like exploitation.
- **Residual value critic** scores marginal progress per step, enabling
  pruning of redundant tool calls.
- **Budget-aware planning**: one root-level planning step injects a
  budget hint and an estimate of required tool calls into the shared
  context.
- Theorem 1 gives probabilistic convergence to a terminal answer
  (≥1−ε) under an explicit finite budget bound.
- Evaluated on HotpotQA, 2WikiMultihopQA, MuSiQue, Bamboogle with
  GPT-OSS-20B and Qwen3-30B-A3B, implemented in Inspect AI, against a
  budget-matched parallel-sampling + majority-vote baseline at Low (5
  calls) / Middle (10) / High (20) tiers.

## Results

- BAVT dominates the baseline's performance–efficiency frontier at every
  budget tier, on both model families.
- OSS-20B: **0.338 average EM at Low (5 tool calls)** vs the baseline's
  **0.334 EM at High (20 calls)** — parity-plus at ¼ the resources.
- Ablation: the tree structure alone is not enough; the step-level value
  and the budget-aware exponent each contribute, and removing α_t leaves
  the framework "constrained".

## Critique / open questions

- Budgets here are tiny and per-question (5–20 tool calls, 1k–8k tokens).
  This project's budgets are per-session and per-week (`max_tokens:
  5_000_000`, weekly quota). Whether `α = 1/r` anneals sensibly over a
  110-hour reset window rather than a 20-step trajectory is untested.
- `r_t = min(tool_ratio, token_ratio)` is exactly the shape the
  coordinator needs — it already tracks several independent ceilings
  (tokens, wall hours, disk, GPU) and currently reduces them with ad-hoc
  logic. `min` over normalized ratios is a defensible unification, but
  the paper gives no evidence it is better than, say, a weighted mean.
- No code released, and the value critic is the same LLM scoring its own
  steps — the residual-delta trick mitigates but does not remove the
  self-evaluation circularity.
- Convergence theorem assumes bounded values `v_min > 0` and a bounded
  exponent; as `r_t → 0`, `α_t → ∞`, so the bound is doing real work and
  its practical tightness is unclear.

## Trust signals

- **Credibility:** 3 — reputable group (UBC + Vector Institute), a
  theoretical convergence result, and evaluation across four benchmarks
  and two model families; but an unreviewed v1 preprint with **no
  released code**, and the headline 4× claim rests on a 0.338-vs-0.334
  margin that no variance is reported for.

## Follow-up

- **Relevance:** 4 — seeds [[budget-conditioned-exploration-exploitation]],
  the missing *within-job* layer of this project's budget story: the
  coordinator decides which jobs run, BAVT decides how a running job
  spends what it was given. Not a 5 because the mechanism is validated
  only at trajectory scale.
- Cheap derived experiment: apply `α = 1/r` to `/iterate` chain depth —
  broad candidate generation early in the weekly window, greedy
  exploitation as the reset nears — and compare against the current
  fixed-depth chain.
- Cross-check against the *opposite* pacing signal in the agency rule:
  `claude-coordinator-agency` currently spends *more* aggressively when
  behind pace near a reset. BAVT's exponent says explore less as budget
  drains. These are not in conflict (one is about total spend, the other
  about spend shape) but the interaction deserves an explicit note.
