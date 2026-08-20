---
kind: concept
name: "fixed-budget evaluation protocol"
status: seedling
added: "2026-08-20"
sources:
  - literature/papers/lupidi2026airs.md
  - literature/papers/shen2026empirical.md
  - literature/papers/yang2026toward.md
related_concepts:
  - hidden-consistent-evaluation
  - efficiency-metric-taxonomy
  - collaboration-topology-tradeoff
related_experiments: []
tags:
  - evaluation
  - compute-budget
  - benchmarking
  - reporting
---

# Fixed-budget evaluation protocol

## Definition

A comparison is only interpretable if the resource envelope is declared
and held uniform across the things being compared, and if the result is
reported in one of exactly two forms: **effectiveness at a fixed cost
budget**, or **cost at matched effectiveness**. The pair traces a Pareto
frontier; a number quoted without its envelope is not a result.

## Why it matters here

This project compares coordinator policies, not models. Every such
comparison is budget-sensitive by construction — a policy that admits
more jobs looks better on throughput and worse on latency depending
entirely on what was held fixed. AIRS-Bench ([[lupidi2026airs]])
demonstrates the discipline at benchmark scale (1×H200, 24 h, ≥10 seeds,
uniform across all 20 tasks, deliberately, even where a task would have
benefited from more) and publishes a comparative envelope table
(MLE-Bench ~1,800 GPU-h; RE-Bench $123/run; MLGym-Bench ~$1/run) that
local runs can be sized against. The efficiency survey
([[yang2026toward]]) supplies the two-sided framing and observes that the
field has no unified protocol — inconsistent metrics with unclear
pipeline boundaries.

The boundary question is the sharp edge. [[shen2026empirical]] budgets
per-candidate *training time* and explicitly **excludes LLM inference,
preflight checks, and debugging** — reasonable on their GPU-bound
testbed, misleading on this one, where tokens are the binding constraint
and deliberation is most of the spend. Declaring the envelope means
declaring what is inside it.

## Connections

- Complements [[hidden-consistent-evaluation]]: HCE governs *which split*
  the score comes from; this governs *under what budget* it was earned.
  AIRS-Bench does both — the agent sees the test set's structure but
  never its labels.
- Supplies the reporting contract for [[efficiency-metric-taxonomy]].
- Practical rule for `experiments/**` here: every `README.md` result line
  states the envelope (tokens, wall time, GPU-hours, seeds) and says
  which of the two comparison forms it is.
- Open: the cheapest seed count that still separates two admission
  policies on this box. AIRS-Bench uses ≥10 and does not study
  sensitivity; 10 seeds of anything nontrivial is a large fraction of a
  weekly token quota.
