---
kind: concept
name: "flow-level concurrency"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/wei2025agent.md
related_concepts:
  - reactive-vs-proactive-agent-flows
  - heterogeneous-accelerator-coordination
  - mixed-criticality-preemption
related_experiments: []
tags:
  - scheduling
  - abstraction
  - agent-runtime
---

# Flow-level concurrency

## Definition

The runtime abstraction treats long-lived, stateful **flows**
(reactive or proactive agent sessions) as the unit of concurrency,
not individual single-shot LLM inferences. A flow interleaves prefill
and decode stages over its lifetime, holds persistent KV-cache /
session state, and has its own priority and SLO.

Introduced in [[wei2025agent]] as the missing abstraction in
on-device LLM engines, which assume static single-shot inference.

## Why it matters here

The coordinator's basic accounting unit shouldn't be "one LLM call"
or "one dvc stage" — it should be a *session* (one experiment, one
agent invocation, one /iterate chain). Sessions have lifecycle
events, persistent state (NOTES.md, /wrap, journal/), and natural
priority labels. Picking the right unit of work is what lets
admission, preemption, and budgeting all use the same vocabulary.

## Connections

- [[reactive-vs-proactive-agent-flows]] — the two priority classes
  of flows.
- [[heterogeneous-accelerator-coordination]] — coordination happens
  at flow granularity.
- [[mixed-criticality-preemption]] — preemption respects flow state
  (suspend at safe inter-step boundaries).
- The coordinator's existing `state.db` already tracks "jobs" — open
  question whether the right schema upgrade is "session = ordered
  collection of jobs" or "job = flow with multiple inner steps."
