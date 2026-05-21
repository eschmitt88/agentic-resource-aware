---
kind: concept
name: "ReAct agent operator"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/hambardzumyan2026aira.md
related_concepts:
  - structural-bottlenecks-in-research-agents
  - asynchronous-multi-gpu-worker-pool
related_experiments: []
tags:
  - operator
  - agent-architecture
---

# ReAct agent operator

## Definition

A worker that iteratively reasons → executes code → observes outputs
until a candidate solution is ready, replacing the single-turn LLM
"operator" (one prompt → one solution) used in earlier
autonomous-research systems. In AIRA² ([[hambardzumyan2026aira]]),
each of N=8 workers is a ReAct agent inside an isolated dev container,
dynamically scoping the compute it spends to sub-problem difficulty.

## Why it matters here

Addresses the third structural bottleneck: fixed single-turn
operators cap reasoning depth and waste compute on easy sub-problems
while starving hard ones. For the resource-aware coordinator, the
ReAct loop is the unit of work that needs to be budgeted — it's
where the variable per-job CPU / GPU / token cost actually lives.

## Connections

- Addresses bottleneck (3) of [[structural-bottlenecks-in-research-agents]].
- The worker inside [[asynchronous-multi-gpu-worker-pool]] is a ReAct
  agent — these two concepts compose, they don't substitute.
- Adjacent question: how does the per-task ReAct budget (max
  reasoning steps, max tool calls) interact with the orchestrator's
  global compute budget? The paper doesn't address this directly.
