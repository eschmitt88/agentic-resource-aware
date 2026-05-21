---
kind: concept
name: "asynchronous multi-GPU worker pool"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/hambardzumyan2026aira.md
related_concepts:
  - structural-bottlenecks-in-research-agents
  - evolutionary-search-over-solutions
related_experiments: []
tags:
  - scheduling
  - throughput
  - compute-budgets
---

# Asynchronous multi-GPU worker pool

## Definition

A scheduling pattern where a Global Orchestrator maintains a
population of candidate solutions and dispatches mutation/crossover
tasks to N workers running in isolated dev containers, with no
synchronization barriers — whenever a worker finishes, the next task
is dispatched immediately. AIRA² ([[hambardzumyan2026aira]]) reports
~linear throughput scaling: 8 GPUs ≈ 8× experimental throughput.

## Why it matters here

This is the canonical architectural answer to the compute-throughput
bottleneck. For the resource-aware coordinator, the open question is
how the pattern degrades on a single-machine 1–2 GPU setup: at N=1,
the async pool collapses to sequential execution and the speedup
disappears. The minimum useful fan-out and the right policy for
job-mixing on a single GPU are both project-relevant.

## Connections

- Addresses bottleneck (1) of [[structural-bottlenecks-in-research-agents]].
- Closely coupled to [[evolutionary-search-over-solutions]] — the
  steady-state evolutionary loop is what makes the pool fully
  asynchronous (no generation barrier).
- Adjacent literature: Agent.xpu (arxiv 2506.24045) does the same
  thing on heterogeneous SoC accelerators; Adaptive GPU Resource
  Allocation (arxiv 2512.22149) generalizes to serverless multi-agent.
