---
kind: concept
name: "structural bottlenecks in research agents"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/hambardzumyan2026aira.md
related_concepts:
  - asynchronous-multi-gpu-worker-pool
  - hidden-consistent-evaluation
  - react-agent-operator
related_experiments: []
tags:
  - taxonomy
  - autonomous-research-agents
---

# Structural bottlenecks in research agents

## Definition

Three load-bearing performance limits, originally formalized in
AIRA-dojo (Toledo et al. 2025) and inherited by AIRA²
([[hambardzumyan2026aira]]), that bind autonomous-research-agent
performance regardless of model scale:

1. **Compute throughput** — synchronous single-GPU execution starves
   the search loop of samples (≈1–20 candidates/day on compute-heavy
   tasks).
2. **Generalization gap** — divergence between the search-signal
   metric and the true held-out metric causes the loop to overfit to
   its own evaluation noise over long horizons.
3. **Operator capability** — fixed single-turn LLM operators can't
   dynamically allocate reasoning to sub-problem difficulty,
   capping search depth.

## Why it matters here

This taxonomy is the design constraint for the resource-aware
coordinator. Any admission / throttling / scheduling policy needs to
defend against all three — not just compute, which is the obvious one.

## Connections

- Each bottleneck has a corresponding architectural answer in AIRA²:
  [[asynchronous-multi-gpu-worker-pool]],
  [[hidden-consistent-evaluation]], [[react-agent-operator]].
- Bottleneck (2) is the rationale for the framework-wide HCE rule in
  `~/claude-system/claude/rules/evaluation.md`.
