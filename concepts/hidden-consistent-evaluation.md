---
kind: concept
name: "hidden consistent evaluation"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/hambardzumyan2026aira.md
related_concepts:
  - structural-bottlenecks-in-research-agents
related_experiments: []
tags:
  - evaluation
  - generalization-gap
  - search-discipline
---

# Hidden Consistent Evaluation (HCE)

## Definition

An evaluation protocol that partitions data into a search split
(D_search, visible to the agent and used for optimization) and a
final-scoring split (D_val, held out and used only at chain end), with
scoring run in a separate container so the agent sees only the score.
Introduced in AIRA² ([[hambardzumyan2026aira]]) as the answer to the
generalization-gap bottleneck.

## Why it matters here

Already promoted to a framework-wide soft rule in
`~/claude-system/claude/rules/evaluation.md`: `test/` is off-limits
during search; `metrics.json` (search signal) and `final_metrics.json`
(held-out) are kept separate; comparable experiments share splits.
The AIRA² paper provides the empirical anchor — its ablations show
"overfitting" in prior work was actually evaluation noise, and HCE
suppresses it.

## Connections

- Addresses bottleneck (2) of [[structural-bottlenecks-in-research-agents]].
- Anchored in the project framework via `evaluation.md` (the user's
  global rule file, listed under `respects:` by all experiment-touching
  skills).
- Inapplicable to projects without a held-out test set — open
  question: what's the equivalent discipline for literature-curation
  or data-scraping projects?
