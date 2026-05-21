---
kind: concept
name: "heterogeneous accelerator coordination"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/wei2025agent.md
  - literature/papers/zhang2025adaptive.md
  - literature/papers/du2025temporal.md
related_concepts:
  - flow-level-concurrency
  - asynchronous-multi-gpu-worker-pool
  - priority-weighted-proportional-allocation
  - temporal-aware-scheduling
  - two-tier-scheduling-hierarchy
related_experiments: []
tags:
  - scheduling
  - heterogeneous-compute
---

# Heterogeneous accelerator coordination

## Definition

Scheduling LLM operators across qualitatively different accelerators
(CPU, iGPU, dGPU, NPU, TPU) that have asymmetric strengths — NPUs are
fast for static-shaped graphs but rigid; iGPUs are flexible but
energy-hungry and contend with graphics; CPUs absorb dynamic-shape
fallback. [[wei2025agent]] introduces operator-accelerator affinity
(per-operator placement preferences) and elastic operator binding
(runtime rebind based on contention) as the two key primitives.

## Why it matters here

The user's workstation has CPU + RAM + GPU and runs both local models
and remote LLM API calls. The coordinator should route work the way
Agent.xpu routes operators: training and high-throughput inference to
the GPU, light prompt routing to the CPU, tool-use sweeps to remote
APIs. The "right accelerator" for a job is the project-level analog
of operator-accelerator affinity.

## Connections

- [[flow-level-concurrency]] — coordination happens at flow
  granularity, not single inference.
- [[asynchronous-multi-gpu-worker-pool]] — AIRA²'s 8-GPU pool is the
  homogeneous-accelerator analog of this concept; the techniques
  diverge once accelerators differ qualitatively.
- Adjacent literature: Adaptive GPU Resource Allocation
  (arxiv 2512.22149) and Temporal-Aware GPU Allocation
  (arxiv 2507.10259) — both worth fetching next.
