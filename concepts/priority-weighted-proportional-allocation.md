---
kind: concept
name: "priority-weighted proportional allocation"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/zhang2025adaptive.md
related_concepts:
  - heterogeneous-accelerator-coordination
  - reactive-vs-proactive-agent-flows
  - mixed-criticality-preemption
related_experiments: []
tags:
  - algorithm
  - scheduling
  - resource-allocation
---

# Priority-weighted proportional allocation

## Definition

A concrete O(N) resource-allocation algorithm in which each consumer's
share is proportional to a demand score `dᵢ = λᵢ · Rᵢ · Pᵢ` (arrival
rate × min resource × priority), subject to a per-consumer minimum
guarantee and a global capacity cap. Allocations are renormalized
down when the minimum-guarantee phase pushes the total above
capacity. Introduced for GPU allocation in serverless multi-agent
systems by [[zhang2025adaptive]].

## Why it matters here

This is the simplest credible algorithm that solves the
resource-aware coordinator's core problem: divide a fixed GPU budget
across heterogeneous concurrent jobs with different priorities and
floor requirements, fast enough to re-evaluate on every job
admission. The minimum-guarantee clause is what keeps low-priority
proactive work from being starved by interactive bursts.

## Connections

- Implements [[heterogeneous-accelerator-coordination]] — this is the
  arithmetic underneath the coordination policy.
- Naturally pairs with [[reactive-vs-proactive-agent-flows]] —
  priority is the axis the two flow classes get compared on.
- Composes with [[mixed-criticality-preemption]] — allocation sets
  the steady-state share, preemption handles burst arrivals before
  reallocation kicks in.
- Open question: how to set `Pᵢ` for research projects whose priority
  changes with proximity to a milestone or deadline. The paper
  treats priority as static input; the coordinator probably needs it
  to drift.
