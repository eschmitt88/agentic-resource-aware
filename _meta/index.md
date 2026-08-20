---
name: index
description: Entry-point index for this project's knowledge graph.
---

# Index

Orientation for the project knowledge graph. Updated by `/wrap`, `/ingest`,
and `/new-experiment`.

## Maps of Content

(promote a cluster of ≥5 related concepts into `mocs/<theme>.md`)

- [[autonomous-research-agent-architecture]] — the inside view of a
  single autonomous research agent; 5 members, anchored on AIRA²
  ([[hambardzumyan2026aira]]).
- [[agent-workload-scheduling]] — the outside view of multiple
  concurrent agents sharing one workstation; 9 members, anchored on
  Agent.xpu ([[wei2025agent]]), Adaptive GPU Allocation
  ([[zhang2025adaptive]]), and TORTA ([[du2025temporal]]). This is
  the project's load-bearing MoC.
- [[budget-aware-evaluation]] — the measurement view: what counts as a
  result on a budgeted box; 5 members, anchored on *Toward Efficient
  Agents* ([[yang2026toward]]) and AIRS-Bench ([[lupidi2026airs]]).
  Promoted 2026-08-20 from the "agent efficiency accounting" candidate.

## Active experiments

(list of `experiments/YYYY-MM-DD-<slug>/` folders currently in flight)

## MoC candidates

(none ripe — the "agent efficiency accounting" candidate was promoted
2026-08-20 as [[budget-aware-evaluation]], with
[[hidden-consistent-evaluation]] cross-listed in as its fifth member.)

## Open questions

(anything you want to return to)

- What observable, available **at admission time**, predicts "deep
  architectural refactor" vs "broad shallow sweep"? That is the routing
  signal [[collaboration-topology-tradeoff]] needs and no paper supplies.
- How does budget-drain annealing (`α = 1/r`,
  [[budget-conditioned-exploration-exploitation]]) interact with the
  agency rule's opposite instinct — spend harder when behind pace near a
  weekly reset?
- Cheapest seed count that still separates two admission policies on this
  box. AIRS-Bench uses ≥10 and never studies sensitivity
  ([[fixed-budget-evaluation-protocol]]).
