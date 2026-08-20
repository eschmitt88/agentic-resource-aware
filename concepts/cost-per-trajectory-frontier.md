---
kind: concept
name: "cost-per-trajectory frontier"
status: seedling
added: "2026-08-20"
sources:
  - literature/papers/liu2025mlagent.md
  - literature/papers/yang2026toward.md
related_concepts:
  - efficiency-metric-taxonomy
  - fixed-budget-evaluation-protocol
related_experiments: []
tags:
  - cost-efficiency
  - model-roles
  - small-models
  - pareto
---

# Cost-per-trajectory frontier

## Definition

Plot agents on (average performance gain) × (cost per trajectory) and
read the top-left. Capability per dollar, not capability per parameter,
is the axis a budget-bound operator actually chooses along.

## Why it matters here

`budget.yaml` assigns model roles (`ideator`, `implementer`) and both are
currently `opus`. [[liu2025mlagent]] is the strongest published evidence
that the implementer role *can* be cheap: a 7B agent trained with
step-wise online RL sits at <$0.01/trajectory with >15% average gain,
tying agents backed by GPT-5 that cost >20× more.

The scope condition is the important part. That result was earned on
repetitive, well-specified, feedback-rich ML edit loops — the regime
where a trained small operator has the most room. Ideation, curation, and
long-horizon planning, which is most of what this repo runs, is the
regime the evidence says *least* about. The honest read: a cheaper
implementer is defensible for narrow mechanical work; downgrading the
ideator is not supported.

Two accounting caveats: the headline figure excludes SFT + online RL
training cost (fair only when amortized over many trajectories — the
break-even is not computed), and cost-of-pass style metrics
([[yang2026toward]]) argue the right denominator is cost *weighted by
success probability*, not raw cost.

## Connections

- One family within [[efficiency-metric-taxonomy]] (token & monetary
  cost), and a direct instance of the "cost at matched effectiveness"
  half of [[fixed-budget-evaluation-protocol]].
- Directionally opposite bet to the scaffold-centric line
  ([[lupidi2026airs]], [[hambardzumyan2026aira]]): train the operator vs
  scaffold a fixed one. No published comparison under a matched total
  budget — a real gap.
- Not reproducible locally: 7B online RL far exceeds one 16 GB RTX 5080.
  This stays an architectural datapoint, not a recipe.
