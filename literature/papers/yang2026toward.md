---
kind: paper
title: "Toward Efficient Agents: A Survey of Memory, Tool Use, and Planning"
authors:
  - Xiaofang Yang
  - Lijun Li
  - Heng Zhou
  - Tong Zhu
  - Xiaoye Qu
  - Yuchen Fan
  - Rui Ye
  - Siheng Chen
  - Jing Shao
institutions:
  - "Shanghai Artificial Intelligence Laboratory"
  - "Fudan University"
  - "University of Science and Technology of China"
  - "Shanghai Jiao Tong University"
  - "Institute of Automation, Chinese Academy of Sciences"
  - "The Chinese University of Hong Kong (Shenzhen)"
  - "Tsinghua University"
year: 2026
venue: arXiv 2601.14192v2 (Shanghai AI Lab + 8 universities)
peer_reviewed: false
url: https://arxiv.org/abs/2601.14192
code_url: https://github.com/yxf203/Awesome-Efficient-Agents
citations: null
source: "raw/papers/yang2026toward.pdf"
added: "2026-08-20"
relevance: 4
credibility: 4
status: skimmed
related_concepts:
  - efficiency-metric-taxonomy
  - fixed-budget-evaluation-protocol
  - budget-conditioned-exploration-exploitation
  - collaboration-topology-tradeoff
related_experiments: []
tags:
  - survey
  - efficiency
  - cost-metrics
  - memory
  - tool-use
  - planning
---

# Toward Efficient Agents: A Survey of Memory, Tool Use, and Planning

## TL;DR

The field's map of agent *efficiency*, organized by the three components
that spend resources — memory, tool use, planning. Its durable
contribution for this project is definitional: efficiency is only
meaningful **effectiveness-first**, measured two ways — effectiveness
under a fixed cost budget, or cost at matched effectiveness — i.e. a
Pareto frontier, plus a four-family taxonomy of the cost metrics people
actually report. This is the paper behind the `Awesome-Efficient-Agents`
reading list.

## Claims

- Agent effectiveness has advanced much faster than agent efficiency, and
  efficiency is what gates real deployment.
- Efficiency is **not** a standalone objective: an agent is efficient
  only if it preserves acceptable task quality. Hence the two-sided
  characterization and the Pareto framing.
- Superficially different methods converge on a few high-level
  principles: bound context via compression and management; shape RL
  rewards to minimize tool invocation; use controlled search to cut
  wasted steps.
- **There is no unified efficiency evaluation for agents.** Reported
  metrics are inconsistent, usually collapsed to tokens or API dollars,
  with unclear pipeline boundaries for runtime, latency, memory
  overhead, and planning overhead.
- Multi-agent designs realized as true multi-model deployments vs
  single-model role-play pipelines differ substantially in orchestration
  overhead, latency and reliability — and should be compared **under
  matched resource budgets**, which nobody does.

## Methods

- Survey structure: efficient memory (construction / management / access
  / procedural reuse via skills / multi-agent memory), efficient tool use
  (selection / calling / tool-integrated reasoning), efficient planning
  (single-agent / multi-agent collaborative), then benchmarks and
  measurements, then challenges.
- Section 6.2 groups reported efficiency metrics into four families:
  - **Token & monetary cost** — the default proxy; token savings, dollar
    cost, and `cost-of-pass` (cost combined with success probability).
  - **Time & runtime overhead** — end-to-end latency, inference time,
    retrieval/index latency (distinguished separately by MemoRAG).
  - **Resource cost** — GPU memory and system overhead; the family that
    catches the trap where token savings are bought with bigger caches
    and indexes.
  - **Interaction & search cost** — LLM calls per response, reasoning
    steps, search depth/breadth, trials, iterations to a valid solution.
- Notes which benchmarks build efficiency in directly (Evo-Memory step
  efficiency, MemBench read/write times, TPS-Bench tokens + end-to-end
  time + tool turns, CostBench explicit tool costs and Cost Gap).

## Results

- The consolidated finding: effectiveness and efficiency must be
  **reported jointly** — holistic benchmarks show whether real task
  completion improved, component benchmarks localize where, efficiency
  metrics say whether the same capability came cheaper.
- Future directions named: a unified efficiency evaluation framework;
  agentic latent reasoning; deployment-aware agentic design (compare
  multi-model vs role-play under matched budgets); efficiency for
  MLLM-based agents.

## Critique / open questions

- It is a survey: no new measurements, and coverage decisions are
  editorial. Value is taxonomic, not evidential.
- Framing is single-agent-workload-centric — cost of *one* agent doing
  its job. This project's problem is the layer above: several agents
  contending for one machine. The four metric families still apply, but
  nothing here addresses contention, admission, or preemption.
- The "resource cost" family is the thinnest section, and it is the one
  closest to this project's concerns (VRAM, disk, CPU contention).
- "Deployment-aware agentic design" is posed as an open direction, and
  [[shen2026empirical]] is a partial answer to it — worth tracking
  whether the survey's next revision picks that up.

## Trust signals

- **Credibility:** 4 — Shanghai AI Laboratory leading a nine-institution
  author list, a maintained companion repository
  (`yxf203/Awesome-Efficient-Agents`) and project page, and citation-dense
  coverage. Not peer-reviewed, and as a survey its claims are
  organizational rather than tested — so not a 5.

## Follow-up

- **Relevance:** 4 — supplies the vocabulary this project's coordinator
  should report in. Seeds [[efficiency-metric-taxonomy]] and
  [[fixed-budget-evaluation-protocol]]; the "no unified efficiency
  evaluation" gap is, restated, an argument for exactly the accounting
  this project is building. Not a 5 because it does not address
  multi-tenant contention at all.
- Concrete use: audit what `claude-coordinator-status` currently reports
  against the four metric families. It covers token/monetary and resource
  well; **interaction cost** (LLM calls per unit of work, steps to
  completion) is largely absent and is the cheapest addition.
- Adopt the two-sided framing in this project's own experiment READMEs:
  every coordinator-policy comparison should state whether it is
  "effectiveness at fixed budget" or "cost at matched effectiveness".
  Mixing the two is how scheduling results get oversold.
- The companion repo is the backstop for missed subtopics — memory
  compression and tool-selection cost are the two branches most likely to
  deserve their own `/discover` run.
