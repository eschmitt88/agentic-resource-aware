---
kind: candidates
topic: "resource-aware agentic systems for autonomous research"
discovered: 2026-05-21
source: discover
n_requested: 10
n_returned: 10
---

## 1. AIRA²: Overcoming Bottlenecks in AI Research Agents

- url: https://arxiv.org/abs/2603.26499
- type: paper
- summary: Identifies three structural bottlenecks in autonomous research agents (synchronous-execution throughput limits, evaluation noise accumulating over long search loops, and rigidity of fixed LM operators) and proposes asynchronous multi-GPU worker pools, improved evaluation protocols, and interactive ReAct operators.
- reason: This is the load-bearing paper for this entire project — already cited in `~/claude-system/claude/rules/evaluation.md` as the source of HCE rationale, and its three bottlenecks are the failure modes any resource-aware coordinator must defend against.

## 2. Agent.xpu: Efficient Scheduling of Agentic LLM Workloads on Heterogeneous SoC

- url: https://arxiv.org/abs/2506.24045
- type: paper
- summary: A scheduling engine that coordinates concurrent reactive (foreground) and proactive (background) LLM flows across heterogeneous CPU/iGPU/NPU accelerators on a single SoC, with adaptive batching and fine-grained preemption — reports 1.2–4.9× throughput and 91% latency reductions over baselines.
- reason: Mirrors the user's exact setup — one machine, mixed-priority concurrent agent workloads, heterogeneous accelerators — at a smaller hardware scale. The mixed-criticality preemption story is directly applicable to the coordinator's admission policy.

## 3. Adaptive GPU Resource Allocation for Multi-Agent Collaborative Reasoning in Serverless Environments

- url: https://arxiv.org/abs/2512.22149
- type: paper
- summary: Dynamic GPU allocation framework for multi-agent systems where lightweight coordinators and heavyweight specialists have heterogeneous demands; reports 85% latency reduction vs. round-robin by adapting to workload priorities.
- reason: Frames the exact "few research projects with very different resource profiles" problem the user described, with concrete numbers against naive scheduling baselines.

## 4. Temporal-Aware GPU Resource Allocation for Distributed LLM Inference via Reinforcement Learning

- url: https://arxiv.org/abs/2507.10259
- type: paper
- summary: Two-layer scheduler — macro-level RL + optimal-transport policy for long-term workload patterns, micro-level allocator for short-term task placement — cutting inference response time ~15% and operational cost 10–20%.
- reason: The two-layer architecture (slow strategic + fast tactical) is a clean template for the user's coordinator: budget.yaml as the macro layer, /plan admission as the micro layer.

## 5. AIRS-Bench: a Suite of Tasks for Frontier AI Research Science Agents

- url: https://arxiv.org/abs/2602.06855
- type: paper
- summary: 20-task benchmark for autonomous research agents spanning language modeling, bioinformatics, etc., with standardized 1×H200 / 24h-quota / ≥10-seed evaluation; agents currently surpass humans on only 4 of the 20 tasks.
- reason: The yardstick the user's resource-aware coordinator can be evaluated against — gives a published budget envelope (24 GPU-hours) the local system should track its own runs against.

## 6. ML-Agent: Reinforcing LLM Agents for Autonomous Machine Learning Engineering

- url: https://arxiv.org/abs/2505.23723
- type: paper
- summary: Trains LLM agents adaptively from task-solving trajectories via RL so they progressively refine decision-making across diverse ML strategies — counterpoint to prompt-only scaffolded agents like AIRA.
- reason: A different posture on the same problem — instead of a smarter scheduler around a fixed model, train the model itself to be a better scheduler. Worth keeping in view as a long-horizon alternative architecture.

## 7. MLR-Copilot: Autonomous Machine Learning Research based on Large Language Models Agents

- url: https://arxiv.org/abs/2408.14033
- type: paper
- summary: LLM-driven agent that ingests a research paper, extracts the problem, generates hypothesis and experiment plan, and executes — the canonical "paper → proposal → experiment" pipeline.
- reason: Predecessor / sibling to AIRA² that maps almost exactly onto the user's /ingest → /propose → /implement chain. Useful for grounding the existing skill design in prior literature.

## 8. Spend Less, Reason Better: Budget-Aware Value Tree Search for LLM Agents

- url: https://arxiv.org/abs/2603.12634
- type: paper
- summary: Tree-search inference algorithm that uses step-level value estimation and budget-aware expansion to shift from exploration to exploitation under strict compute budgets; surpasses baselines using 4× the budget.
- reason: The user's coordinator allocates budget across projects; this paper allocates budget within a single agent's reasoning step. Both layers benefit from the same exploration/exploitation framing.

## 9. An Empirical Study of Multi-Agent Collaboration for Automated Research

- url: https://arxiv.org/abs/2603.29632
- type: paper
- summary: Empirical study of multi-agent setups for autonomous research — division of labor, coordination overhead, when collaboration helps vs. hurts.
- reason: Directly informs whether the user's "few concurrent projects" should be coordinated agents sharing context or independent agents on a scheduler. Bears on the architecture choice.

## 10. Awesome-Efficient-Agents (curated reading list)

- url: https://github.com/yxf203/Awesome-Efficient-Agents
- type: repo
- summary: Curated survey of efficiency-focused LLM-agent work spanning memory, tool learning, and planning — entry point into adjacent literature not surfaced by individual searches.
- reason: The "what am I missing?" backstop. Worth a single pass to identify subtopics (memory compression, tool selection cost) that deserve their own /discover run later.
