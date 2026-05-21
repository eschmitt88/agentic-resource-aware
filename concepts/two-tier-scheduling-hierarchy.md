---
kind: concept
name: "two-tier scheduling hierarchy"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/du2025temporal.md
  - literature/papers/zhang2025adaptive.md
related_concepts:
  - temporal-aware-scheduling
  - priority-weighted-proportional-allocation
related_experiments: []
tags:
  - scheduling
  - architecture
  - separation-of-concerns
---

# Two-tier scheduling hierarchy

## Definition

A scheduling architecture that splits decisions across two timescales:

- **Macro / strategic** — slow, optionally learned, coordinates
  across coarse units (regions, clusters, projects). Outputs a plan
  or budget allocation.
- **Micro / tactical** — fast, online, dispatches individual tasks
  within the macro plan's constraints. Minimizes immediate latency
  and switching cost.

The macro layer can afford expensive optimization (RL, optimal
transport, MIP) because it runs infrequently. The micro layer must
be O(N) or cheaper because it runs on every job arrival. Formalized
in [[du2025temporal]]; implicit in [[zhang2025adaptive]]'s single-tier
allocator (which conflates the two and only works because the
problem is small).

## Why it matters here

The project already has the *shape* of this hierarchy without naming
it: `budget.yaml` is a static macro plan (hard ceilings on wall-time,
tokens, disk; model-role assignments) and `/plan` admission is the
tactical layer. The MoC question is whether to make the macro layer
*learned* (offline temporal model over `state.db` log) or keep it as
hand-edited config. The hierarchy itself is the right abstraction
either way.

## Connections

- [[temporal-aware-scheduling]] — temporal awareness lives at the
  macro layer (slow predictions); micro is reactive.
- [[priority-weighted-proportional-allocation]] — natural micro-layer
  algorithm; runs inside whatever macro plan is current.
- [[mixed-criticality-preemption]] — micro-layer mechanism;
  preemption decisions need to be sub-second.
- Open question: is the user's `state.db` schema currently a flat
  job log or already structured as a two-tier state? If flat, this
  concept implies a schema migration before the macro layer can
  learn anything.
