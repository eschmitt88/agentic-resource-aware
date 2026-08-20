---
kind: paper
title: "AIRS-Bench: a Suite of Tasks for Frontier AI Research Science Agents"
authors:
  - Alisia Lupidi
  - Bhavul Gauri
  - Thomas Simon Foster
  - Bassel Al Omari
  - Despoina Magka
  - Karen Hambardzumyan
  - Roberta Raileanu
  - Jakob Foerster
  - Yoram Bachrach
institutions:
  - "FAIR at Meta"
  - "University of Oxford"
  - "University College London"
year: 2026
venue: arXiv 2602.06855v3 (FAIR at Meta + Oxford + UCL)
peer_reviewed: false
url: https://arxiv.org/abs/2602.06855
code_url: https://github.com/facebookresearch/airs-bench
citations: null
source: "raw/papers/lupidi2026airs.pdf"
added: "2026-08-20"
relevance: 4
credibility: 4
status: skimmed
related_experiments: []
related_concepts:
  - fixed-budget-evaluation-protocol
  - hidden-consistent-evaluation
  - evolutionary-search-over-solutions
  - two-tier-scheduling-hierarchy
tags:
  - benchmark
  - evaluation
  - compute-budget
  - research-agents
  - scaffolds
---

# AIRS-Bench: a Suite of Tasks for Frontier AI Research Science Agents

## TL;DR

A 20-task benchmark for autonomous research agents (language modeling,
maths, bioinformatics, time-series), scored against human SOTA from the
source papers, and — most useful here — evaluated under a **uniform,
explicitly declared resource envelope**: 1×H200, 24 h wall clock, ≥10
seeds per task per agent. 14 agents (LLM × scaffold) were run; they beat
human SOTA on 4 of 20 tasks and stay far from the ceiling on the rest.

## Claims

- Agentic research capability must be measured as an **(LLM, scaffold)
  pair**, not as a model property. The same base model moves
  substantially with the scaffold (Greedy tree search ≫ One-Shot for both
  CWM and GPT-4o), and ReAct's benefit is model-dependent.
- **Test-time search substitutes for model size**: Greedy gpt-oss-20b is
  on par with Greedy gpt-oss-120b. Scaffold spend buys what parameters
  would otherwise buy.
- Reported aggregate scores are meaningless without a stated budget. The
  paper holds constraints uniform across tasks *deliberately*, noting
  some tasks would have benefited from more compute — the budget is part
  of the benchmark definition, not an implementation detail.
- The benchmark is far from saturated: a large Elo gap remains between
  the best agent and the "SOTA player".

## Methods

- **Task format**: {problem, dataset, metric} triplet plus a
  `metadata.yaml`; no baseline code is given, so the agent must do the
  full research lifecycle (idea → implementation → refinement).
- **Two harnesses**: MLGym (sequential/ReAct) and AIRA-dojo
  (tree-search/Greedy, One-Shot), run under matched constraints to
  isolate scaffold effects from harness effects.
- **Resource protocol**: each run = 24 h on 1×H200; ≥10 seeds per
  (agent, task); pre-cached HF checkpoints (nothing newer than 2021) to
  dodge rate limits; agents are never told the SOTA method or score.
- **Score aggregation**: valid submission rate; a normalized score using
  a "march of 9s" log transform toward the task optimum; and Elo from a
  Bradley–Terry fit over pairwise task outcomes, with human SOTA entered
  as an extra player.

## Results

- Agents exceed human SOTA on **4 of 20** tasks; on none do they reach
  the theoretical ceiling.
- Reasoning models (o3-mini, gpt-oss-120b) lead in both One-Shot and
  Greedy; tree search lifts both open and closed models.
- Appendix D tabulates the compute envelope of comparable benchmarks —
  the single most reusable artifact here for this project:
  AIRS-Bench 1×H200 × 24 h/task (20 tasks, 10–20 runs each);
  MLE-Bench 1×A10 × 24 h/competition (~1,800 GPU-h total, 75
  competitions); MLGym-Bench 0–2 GPUs, 2–4 h/task, ~$1/run (up to $9);
  RE-Bench 0–6 H100s, 8 h/run, **$123/run**; ML-Agent-Bench 0.5–2 h/task,
  ≤$60 total.

## Critique / open questions

- The envelope is a *data-center* envelope. This project's box is one
  RTX 5080 (16 GB) — a task budgeted at 24 h × H200 does not port. What
  ports is the **discipline of declaring the envelope** and holding it
  uniform across compared runs.
- 10+ seeds per (agent, task) is the noise-control answer to AIRA²'s
  evaluation bottleneck ([[hidden-consistent-evaluation]]), but it
  multiplies cost by 10×. For a single-workstation coordinator the
  interesting question is the cheapest seed count that still separates
  policies — the paper does not study seed-count sensitivity.
- Budget is uniform across tasks *by choice*, which the authors admit
  disadvantages some tasks. A resource-aware coordinator would instead
  allocate non-uniformly — this benchmark is precisely the setting where
  such a policy could be tested, and no such baseline is run here.
- Human SOTA is sourced from literature, so "beats SOTA" inherits every
  reporting bias of the source papers.

## Trust signals

- **Credibility:** 4 — FAIR at Meta with Oxford and UCL co-authors,
  task definitions and evaluation code open-sourced
  (`facebookresearch/airs-bench`), 10+ seeds and two independent
  harnesses. Not peer-reviewed (arXiv v3), which is the only thing
  keeping it off 5.

## Follow-up

- **Relevance:** 4 — does not change the coordinator's architecture, but
  it is the external yardstick this project's budget language should
  speak: it seeds [[fixed-budget-evaluation-protocol]] and supplies a
  published cost table to size local runs against. Not a 5 because the
  hardware scale is an order of magnitude off this workstation.
- Concrete use: when `budget.yaml` ceilings are next revised, express
  them in the same units as Appendix D (GPU-hours per task, $ per run,
  seeds per comparison) so local runs are comparable to published ones.
- Open thread: measure the seed count at which two admission policies
  become statistically separable on this box — the cheap-seeds question
  AIRS-Bench leaves open.
