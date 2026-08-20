---
kind: paper
title: "ML-Agent: Reinforcing LLM Agents for Autonomous Machine Learning Engineering"
authors:
  - Zexi Liu
  - Jingyi Chai
  - Xinyu Zhu
  - Shuo Tang
  - Rui Ye
  - Bo Zhang
  - Lei Bai
  - Siheng Chen
institutions:
  - "Shanghai Jiao Tong University"
  - "Shanghai AI Laboratory"
year: 2025
venue: arXiv 2505.23723v2 (SJTU + Shanghai AI Lab)
peer_reviewed: false
url: https://arxiv.org/abs/2505.23723
code_url: null
citations: null
source: "raw/papers/liu2025mlagent.pdf"
added: "2026-08-20"
relevance: 3
credibility: 3
status: skimmed
related_concepts:
  - cost-per-trajectory-frontier
  - react-agent-operator
  - structural-bottlenecks-in-research-agents
related_experiments: []
tags:
  - reinforcement-learning
  - agentic-ml
  - cost-efficiency
  - model-role-selection
  - small-models
---

# ML-Agent: Reinforcing LLM Agents for Autonomous ML Engineering

## TL;DR

Instead of scaffolding a frontier model, *train* a 7B agent (Qwen2.5-7B)
with online RL on interactive ML tasks. Three ingredients:
exploration-enriched fine-tuning, **step-wise** RL (train on a single
action step rather than a whole episode), and a reward module that
unifies heterogeneous ML feedback. Trained on 9 tasks, it generalizes to
10 held-out ones, matching GPT-5-backed agents at **<$0.01 per
trajectory** — >20× cheaper than the baselines it ties.

## Claims

- The prompt-based paradigm is stuck between two bad options: small
  models can't learn from execution trajectories, large proprietary
  models are too expensive to run at agentic volumes.
- Agentic ML can be learned rather than prompted — the first
  demonstration of online RL over real ML-experiment trajectories.
- **Slow experience collection is the binding constraint on training**:
  ML experiments take minutes to hours, so episode-wise RL starves the
  learner. Step-wise RL reformulates the objective per action step,
  cutting rollout cost dramatically.
- Capability per dollar, not capability per parameter, is the right axis:
  a small trained agent can occupy the top-left of the
  performance-vs-cost plane.

## Methods

- **Exploration-enriched fine-tuning** — SFT stage (2 epochs, A100s)
  tuned to produce diverse actions so subsequent RL exploration isn't
  degenerate.
- **Step-wise RL** — optimize on single action steps instead of full
  episodes, accelerating experience collection; contrasted directly
  against episode-wise PPO.
- **Agentic-ML reward module** — normalizes varied ML feedback signals
  (different metrics, different directions) into consistent rewards;
  performance gain measured as relative improvement over the initial
  script, sign-corrected per metric, averaged over 8 trajectories.
- **Evaluation** — 3 held-in + 10 held-out tasks, against MLAB, AIDE and
  ML-Master scaffolds backed by Qwen2.5-7B, Qwen3-235B, DeepSeek-R1,
  Gemini-2.5-Pro and GPT-5, with matched time limits and matched numbers
  of code modifications.

## Results

- Outperforms much larger open-source backbones (including 671B
  DeepSeek-R1) and stays competitive with GPT-5-backed agents.
- >15% average performance gain at **<$0.01/trajectory**; baselines with
  comparable or lower gain cost >20× more.
- Step-wise RL beats episode-wise PPO on both held-in and held-out tasks
  at equal GPU-hours; the ablation curve shows continuous improvement
  through training rather than early saturation.

## Critique / open questions

- The cost comparison omits the **training** cost (SFT + online RL on
  A100s) from the per-trajectory figure. Amortized over enough
  trajectories that's fair; for a single workstation running a handful of
  jobs a week it is not obviously fair, and the break-even point isn't
  computed.
- No code release, and the reward module — the piece most likely to be
  fiddly — is described but not published.
- "Comparable to GPT-5" is measured as relative gain over an initial
  script on 13 tasks; that is a narrow slice of what the implementer role
  does here (code editing, tool use, long-horizon planning).
- Directionally the opposite bet from AIRA²/AIRS-Bench: spend on training
  the operator vs spend on the scaffold around a fixed operator. Nobody
  has run the comparison under a matched total budget.

## Trust signals

- **Credibility:** 3 — strong group (Shanghai AI Laboratory + SJTU) with
  a broad baseline sweep and clean ablations, but a preprint with no
  released code or model weights, and the headline cost claim depends on
  an accounting choice (excluding training) that the paper does not
  defend.

## Follow-up

- **Relevance:** 3 — useful prior art on the axis this project's
  `budget.yaml` model roles sit on (`ideator: opus`, `implementer:
  opus`): it is the strongest published evidence that a cheap operator
  can hold the frontier on *narrow, repetitive* agentic work. It seeds
  [[cost-per-trajectory-frontier]] but doesn't shift the coordinator's
  architecture — hence 3, not 4.
- Practical read for this box: the case for a cheaper `implementer` is
  strongest exactly where ML-Agent was trained — repetitive,
  well-specified, feedback-rich edit loops. Ideation and curation, which
  is what this repo mostly runs, is the case *least* supported by this
  evidence. Do not downgrade the ideator on the strength of this paper.
- Local training is out of reach regardless (7B online RL ≫ one 16 GB
  RTX 5080); this stays an architectural datapoint, not a recipe.
