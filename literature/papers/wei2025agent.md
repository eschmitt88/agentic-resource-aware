---
kind: paper
title: "Agent.xpu: Efficient Scheduling of Agentic LLM Workloads on Heterogeneous SoC"
authors:
  - Xinming Wei
  - Jiahao Zhang
  - Haoran Li
  - Jiayu Chen
  - Haoning Guan
  - Rui Qu
  - Maoliang Li
  - Xiang Chen
  - Guojie Luo
year: 2025
venue: arXiv 2506.24045
url: https://arxiv.org/abs/2506.24045
source: "raw/papers/wei2025agent.pdf"
added: "2026-05-21"
relevance: 5
status: skimmed
related_experiments: []
related_concepts:
  - reactive-vs-proactive-agent-flows
  - heterogeneous-accelerator-coordination
  - mixed-criticality-preemption
  - flow-level-concurrency
tags:
  - scheduling
  - heterogeneous-compute
  - personal-agents
  - on-device-llm
  - mixed-criticality
---

# Agent.xpu: Efficient Scheduling of Agentic LLM Workloads on Heterogeneous SoC

## TL;DR

First LLM engine that schedules concurrent reactive (foreground,
low-latency) and proactive (background, throughput) agent flows
across heterogeneous SoC accelerators (CPU + iGPU + NPU), treating
flows as long-lived stateful units instead of one-shot inferences.
Reports 1.2–4.9× proactive throughput and ≥91% reactive-latency
reduction vs iGPU-only and serial NPU-iGPU baselines.

## Claims

- Personal LLM agents have a fundamental two-mode workload structure:
  **reactive** flows (user-initiated, bursty, latency-critical) and
  **proactive** flows (background monitoring, long-running,
  best-effort). Existing engines treat all inference as static and
  single-shot — a categorical mismatch.
- Three gaps in current heterogeneous SoC stacks: flexibility-
  efficiency trade-off (NPU is fast but static-shaped; iGPU is
  flexible but inefficient and contends with graphics), shared-memory
  DDR-bandwidth contention between concurrent kernels, and absence of
  flow-aware runtime abstractions.
- Three opportunities the system exploits: operator-accelerator
  affinity (different LLM operators prefer different accelerators in
  prefill vs decode), asymmetric DDR contention, and stage-divergent
  batching behavior.
- The mixed-criticality scheduling problem (reactive priority,
  proactive throughput, no starvation) is solvable with fine-grained
  preemption + slack-aware piggybacking.

## Methods

- **Heterogeneous Execution Graph (HEG)** — captures NPU/iGPU
  affinity per operator and supports elastic operator binding
  (pre-run static placement + runtime elastic rebind).
- **Flow-aware NPU-iGPU coordination with stage elasticity** —
  decouples prefill (compute-heavy) from decode (bandwidth-heavy)
  across accelerators to reduce DDR contention and enforce flow
  priorities.
- **Fine-grained preemption with slack-aware piggybacking** —
  reactive flows pre-empt proactive kernels but proactive work
  piggybacks during reactive slack windows so it doesn't starve.
- Implemented on commodity laptop-class SoC; baselines are OpenVINO
  iGPU serving, Llama.cpp CPU serving, and tuned serial NPU-iGPU
  inference. Workloads use Llama3-3B/8B as backbones with realistic
  reactive (Q&A) + proactive (event handling, function calling, RAG)
  flow mixes.

## Results

- Proactive-only: 1.2–2.4× over iGPU baseline, 1.4–4.9× over serial
  NPU-iGPU.
- Mixed reactive+proactive: reactive latency reduced 91–97%,
  proactive throughput improved 0.3–58% vs iGPU baseline. Speedups
  larger on 8B than 3B (better scaling with model size).
- Energy: 26.8% reduction vs iGPU serving. iGPU utilization: 32.5%
  reduction vs serial NPU-iGPU (less graphics interference).

## Critique / open questions

- The hardware is consumer laptop SoC with iGPU+NPU+CPU. How do the
  same flow-aware primitives translate to a workstation with a single
  discrete GPU (no NPU), or to a server with multiple discrete GPUs?
  The "accelerator affinity" concept presumably still applies but
  with very different ratios.
- Token cost / dollar cost is not in the picture — it's all on-device
  hardware utilization. For the resource-aware coordinator that
  budgets *both* GPU-hours and LLM tokens (when calling out to
  Claude/Gemini), this needs a separate framing.
- "Flow" is well-defined for personal agents (one LLM, multiple
  concurrent prompts). What's the equivalent unit for a research
  workstation where flows might span days and include dvc pipelines,
  tool use, and external API calls? Probably "session" or "experiment
  run."
- No public code release reference seen in the skim — would need to
  verify whether the system is reproducible or proprietary.

## Follow-up

- **Relevance:** 5 — seeds 4 load-bearing concepts (reactive/proactive,
  heterogeneous coordination, mixed-criticality preemption, flow-level
  concurrency) that the resource-aware coordinator needs natively. The
  closest published match to the user's single-workstation setup,
  though at a smaller hardware scale.
- Strong candidate for an `expand`-style derivation: take the
  flow-aware preemption model and ask what it looks like for a 1-GPU
  workstation hosting both a research agent (proactive) and an
  interactive coding session (reactive). This is essentially the
  user's problem.
- Cross-link [[asynchronous-multi-gpu-worker-pool]] — Agent.xpu and
  AIRA² are answering the same question (how to keep accelerators
  busy in an agent system) at different scales.
