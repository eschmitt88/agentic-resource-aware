---
kind: concept
name: "efficiency metric taxonomy"
status: seedling
added: "2026-08-20"
sources:
  - literature/papers/yang2026toward.md
related_concepts:
  - fixed-budget-evaluation-protocol
  - cost-per-trajectory-frontier
related_experiments: []
tags:
  - metrics
  - accounting
  - reporting
  - coordinator
---

# Efficiency metric taxonomy

## Definition

Agent efficiency is reported in four metric families
([[yang2026toward]]), and collapsing to any one of them hides the others:

1. **Token & monetary cost** — token usage/savings, dollar cost,
   `cost-of-pass` (cost combined with success probability).
2. **Time & runtime overhead** — end-to-end latency, inference time,
   retrieval vs index latency kept separate.
3. **Resource cost** — GPU memory, cache/index size, system overhead.
   Catches the trap where token savings are bought with a bigger store.
4. **Interaction & search cost** — LLM calls per response, reasoning
   steps, search depth/breadth, retries, iterations to a valid solution.

## Why it matters here

This is a ready-made audit checklist for what the coordinator reports.
Against `claude-coordinator-status` today:

- token & monetary — **covered** (5h block + reset-anchored weekly,
  burn rate, projected end-of-block).
- resource — **covered** (CPU/RAM/GPU/VRAM/disk sampled every 30 s).
- time — **partial** (hardware is sampled over time; per-job wall time
  isn't a first-class metric).
- interaction — **absent**, and the cheapest to add: LLM calls per unit
  of work and steps-to-completion are already latent in the session
  transcripts.

The survey's own headline gap is that the field has no unified efficiency
evaluation — inconsistent dimensions, usually reduced to tokens or API
cost, with unclear pipeline boundaries for memory, tool-use, and planning
overhead. That gap is, restated, the argument for building this
accounting deliberately here rather than inheriting it.

## Connections

- Supplies the metric vocabulary; [[fixed-budget-evaluation-protocol]]
  supplies the comparison contract they must be reported under.
- [[cost-per-trajectory-frontier]] is family (1) taken seriously.
- Family (3) is where a single-workstation coordinator lives and is the
  thinnest section of the survey — the multi-tenant contention case
  (several agents competing for one GPU) is not addressed by any of the
  metrics above.
