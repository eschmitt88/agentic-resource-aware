---
kind: moc
name: "budget-aware evaluation"
status: live
added: "2026-08-20"
member_concepts:
  - fixed-budget-evaluation-protocol
  - hidden-consistent-evaluation
  - efficiency-metric-taxonomy
  - cost-per-trajectory-frontier
  - budget-conditioned-exploration-exploitation
related_mocs:
  - agent-workload-scheduling
  - autonomous-research-agent-architecture
tags:
  - moc
  - evaluation
  - budget-aware
  - metrics
  - accounting
---

# Budget-aware evaluation

**What counts as a result on a budgeted box, and what did it cost?**

The other two MoCs describe machinery: what is inside one agent
([[autonomous-research-agent-architecture]]) and how several of them
share a workstation ([[agent-workload-scheduling]]). Neither says how
you would know a coordinator policy actually worked. That is this MoC.
It exists because on a single-workstation, quota-bound setup the
resource envelope is not a footnote to the result — it *is* half the
result. A policy that admits more jobs wins on throughput and loses on
latency depending entirely on what was held fixed, so an unqualified
number here carries no information at all.

The five concepts form a chain: declare the envelope, choose the signal,
count the right things, read the frontier, and let what is left of the
budget steer the spend.

## The comparison contract

What has to be true before a number is a result.

- [[fixed-budget-evaluation-protocol]] — declare the envelope and hold
  it uniform, then report in exactly one of two forms: effectiveness at
  fixed cost, or cost at matched effectiveness. The pair traces a Pareto
  frontier. The sharp edge is boundary declaration — [[shen2026empirical]]
  budgets per-candidate training time and *excludes* LLM inference,
  which is defensible on a GPU-bound testbed and misleading on this one,
  where deliberation is most of the spend.
- [[hidden-consistent-evaluation]] — the orthogonal half of the same
  contract: HCE governs *which split* the score came from, the protocol
  above governs *under what budget* it was earned. Cross-listed from
  [[autonomous-research-agent-architecture]], where it answers the
  generalization-gap bottleneck; it belongs here too because the two
  disciplines are only useful together. AIRS-Bench ([[lupidi2026airs]])
  runs both at once.

## What to count

The vocabulary the contract is reported in.

- [[efficiency-metric-taxonomy]] — four families (token & monetary,
  time & runtime, resource, interaction & search) and the warning that
  collapsing to one hides the others. Doubles as an audit checklist for
  `claude-coordinator-status`, which today covers token and resource,
  partially covers time, and misses interaction entirely — the cheapest
  gap to close, since calls-per-unit-work is already latent in session
  transcripts.
- [[cost-per-trajectory-frontier]] — family (1) taken seriously:
  capability per dollar, not per parameter. [[liu2025mlagent]] puts a 7B
  RL-trained implementer at <$0.01/trajectory tying agents costing 20×
  more — but earned on repetitive, feedback-rich edit loops, which is
  the regime this repo runs *least*. The scope condition is the finding.

## Spending what you measured

- [[budget-conditioned-exploration-exploitation]] — the policy side.
  Make the remaining-budget ratio the control parameter rather than a
  tuned constant: `r = min(usedᵢ/ceilingᵢ)`, exploration exponent
  `α = 1/r`, annealing from broad to greedy for free as the window
  drains ([[li2026spend]]). It closes the loop — the accounting above is
  what produces `r`.

## Primary sources

- [[yang2026toward]] — *Toward Efficient Agents* (Shanghai AI Lab + 8
  universities, 2026): the anchor. Supplies the four metric families,
  the two-sided comparison framing, and the field-level observation that
  there is no unified efficiency evaluation — inconsistent dimensions,
  usually collapsed to tokens, unclear pipeline boundaries.
- [[lupidi2026airs]] — AIRS-Bench (FAIR/Oxford/UCL 2026): the discipline
  demonstrated at benchmark scale (1×H200, 24 h, ≥10 seeds, uniform
  across 20 tasks) plus a comparative envelope table local runs can be
  sized against.
- [[li2026spend]] — BAVT (UBC + Vector 2026): budget ratio as annealing
  schedule.
- [[liu2025mlagent]] — ML-Agent (SJTU + Shanghai AI Lab 2025): the
  cheap-implementer datapoint on the cost frontier.
- [[hambardzumyan2026aira]] — AIRA² (FAIR/UCL/Oxford 2026): origin of
  HCE and the evidence that apparent "overfitting" in prior work was
  evaluation noise.

## Open threads

- **Where does the envelope end?** Tokens are the binding constraint on
  this box, so any protocol that excludes deliberation cost is measuring
  the wrong thing. Every `experiments/**` result line should state
  tokens, wall time, GPU-hours, and seeds — and say which of the two
  comparison forms it is.
- **Cheapest seed count that separates two admission policies here.**
  AIRS-Bench uses ≥10 and never studies sensitivity; 10 seeds of
  anything nontrivial is a large fraction of a weekly quota. This is
  the single most actionable unknown in the MoC.
- **Two budget instincts point opposite ways.** The agency rule spends
  *harder* when behind pace near a weekly reset (unused quota is
  wasted); `α = 1/r` explores *less* as budget drains. They govern
  different things — total spend vs the shape of the spend — but a
  policy using both must say so explicitly.
- **Family (3) is thin exactly where this project lives.** No metric in
  the taxonomy addresses multi-tenant contention — several agents
  competing for one GPU. That is the measurement gap
  [[agent-workload-scheduling]] will eventually need filled, and no
  surveyed source fills it.

## Why this MoC is here

[[agent-workload-scheduling]] is the deliverable; this is its
scoreboard. Any ablation of a coordinator policy variant is only
interpretable through the contract defined here, which makes this MoC a
precondition for the first experiment rather than a retrospective on it.
