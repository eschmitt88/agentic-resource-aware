---
kind: candidates
topic: "resource-aware agentic systems for autonomous research"
discovered: 2026-05-21
source: discover
n_requested: 10
n_returned: 10
curated: 2026-08-20
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

## Curation

Curated 2026-08-20 under `agency: max` with the coordinator verdict at
GO/high (17% weekly quota used, 111 h to reset, CPU/RAM/GPU idle).
All ten arXiv IDs were verified live against the arXiv API before any
disposition — none were link-rot.

1. **AIRA²** (2603.26499) — already in graph → [[hambardzumyan2026aira]].
2. **Agent.xpu** (2506.24045) — already in graph → [[wei2025agent]].
3. **Adaptive GPU Resource Allocation** (2512.22149) — already in graph →
   [[zhang2025adaptive]].
4. **Temporal-Aware GPU Resource Allocation (TORTA)** (2507.10259) —
   already in graph → [[du2025temporal]].
5. **AIRS-Bench** (2602.06855) — ingested → [[lupidi2026airs]].
   Seeds [[fixed-budget-evaluation-protocol]]; Appendix D's cross-benchmark
   compute-envelope table is the reusable artifact.
6. **ML-Agent** (2505.23723) — ingested → [[liu2025mlagent]]. Kept for the
   cost axis rather than the RL method: <$0.01/trajectory at parity with
   GPT-5-backed agents bears directly on `budget.yaml` model roles.
   Seeds [[cost-per-trajectory-frontier]].
7. **MLR-Copilot** (2408.14033) — declined — superseded prior art: the
   2024 paper→proposal→experiment pipeline it describes is already
   represented in the graph by AIRA² ([[hambardzumyan2026aira]]) and
   instantiated by this repo's own skill chain, and it carries no
   resource-allocation content.
8. **Spend Less, Reason Better (BAVT)** (2603.12634) — ingested →
   [[li2026spend]]. Seeds [[budget-conditioned-exploration-exploitation]] —
   the within-job budget layer beneath the coordinator's across-job one.
9. **An Empirical Study of Multi-Agent Collaboration** (2603.29632) —
   ingested → [[shen2026empirical]]. Closest published match to this
   project's setup (one box, fixed budget, worktree isolation). Seeds
   [[collaboration-topology-tradeoff]] and
   [[worktree-isolated-parallel-search]].
10. **Awesome-Efficient-Agents** (github.com/yxf203/Awesome-Efficient-Agents)
    — ingested as its companion **survey** → [[yang2026toward]]
    (arXiv 2601.14192, "Toward Efficient Agents") rather than as the link
    list. The repo is the paper's reading list; the paper carries the
    analysis. Seeds [[efficiency-metric-taxonomy]]. The repo remains the
    seed for later `/discover` runs on memory compression and
    tool-selection cost.

**Totals: ingested 5, declined 1, already in graph 4.**
