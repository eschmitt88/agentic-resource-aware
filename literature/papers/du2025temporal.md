---
kind: paper
title: "Temporal-Aware GPU Resource Allocation for Distributed LLM Inference via Reinforcement Learning"
authors:
  - Chengze Du
  - Zhiwei Yu
  - Heng Xu
  - Haojie Wang
  - Bo Liu
  - Jialong Li
year: 2025
venue: arXiv 2507.10259 (Shenzhen Univ. of Adv. Tech. + China Mobile Research)
url: https://arxiv.org/abs/2507.10259
source: "raw/papers/du2025temporal.pdf"
added: "2026-05-21"
relevance: 4
status: skimmed
related_experiments: []
related_concepts:
  - temporal-aware-scheduling
  - two-tier-scheduling-hierarchy
  - heterogeneous-accelerator-coordination
  - priority-weighted-proportional-allocation
tags:
  - scheduling
  - reinforcement-learning
  - optimal-transport
  - gpu-allocation
  - data-center
---

# Temporal-Aware GPU Resource Allocation via RL (TORTA)

## TL;DR

TORTA is a two-layer GPU scheduler for distributed LLM inference:
the **macro** layer uses RL + optimal transport to coordinate
inter-region task distribution against predicted demand, and the
**micro** layer assigns tasks to servers within a region while
minimizing switching costs. Reports up to 15% inference response
time reduction, 4–5% better load balance, and 10–20% lower
operational cost vs reactive baselines.

## Claims

- Reactive (instantaneous-state-only) GPU schedulers are
  fundamentally limited: they ignore predictable demand patterns
  and pay repeated switching costs. Adding temporal awareness is
  worth the order-of-magnitude solution-space inflation.
- Two timescales matter, and they want different machinery:
  short-term wants warm-up / cache affinity ("allocate similar
  tasks to the same models"); long-term wants stable utilization
  and minimal cross-cluster switching.
- A two-layer architecture (macro coordination + micro allocation)
  cleanly separates the two timescales. The macro layer learns
  offline; the micro layer runs online with the macro layer's plan
  as input.
- Reinforcement learning + optimal transport is a workable recipe
  for the macro layer when the demand pattern is partially
  predictable.

## Methods

- **Problem framing**: GPU scheduling as an MDP. State includes
  GPU utilization, queue depths, request load distribution. Reward
  combines latency, cost, and load balance.
- **Macro layer**: RL policy outputs an inter-region task-allocation
  plan; optimal transport solves the assignment subject to capacity
  and cost constraints. Trained offline on large samples.
- **Micro layer**: online server-selection within each region.
  Minimizes task completion time and switching cost (avoiding
  oscillation between servers as load shifts).
- **Evaluation**: multiple network topologies, simulation-driven,
  baselines are reactive single-layer schedulers.

## Results

- 15% reduction in average inference response time.
- 4–5% improvement in load balance.
- 10–20% reduction in total operational cost.

## Critique / open questions

- Data-center scale (inter-region, multiple clusters) is much
  larger than the user's single-workstation problem. The RL+OT
  machinery is overkill for ≤4 concurrent jobs on one box.
- However, the **two-layer separation** and **temporal awareness**
  are directly transferable. The user's existing system already
  has the bones of this (slow strategic = `budget.yaml`, fast
  tactical = `/plan` admission), just without the temporal model
  or offline learning.
- "Switching cost" framing is genuinely useful: re-loading a model
  or warming a venv is the user's analog of cross-cluster
  migration cost. Coordinator should track it.
- Demand prediction needs historical data. The coordinator has
  `state.db` already; the question is whether the log there is
  rich enough to learn a useful temporal model.

## Follow-up

- **Relevance:** 4 — seeds two new concepts that the project's MoC
  has been implicitly assuming but never named
  ([[temporal-aware-scheduling]], [[two-tier-scheduling-hierarchy]]),
  and strengthens existing concepts with a concrete RL+OT recipe.
  Not a 5 because the scale (data-center vs single workstation)
  means the algorithm itself doesn't port directly — only the
  framing.
- Possible derived experiment: log the user's existing coordinator
  decisions over a week, inspect for predictable demand patterns
  (interactive bursts, scheduled `/digest` runs, overnight
  `/iterate` chains), then evaluate whether a simple
  temporal-aware policy outperforms the current reactive admission.
