---
kind: moc
name: "autonomous research agent architecture"
status: live
added: "2026-05-21"
member_concepts:
  - structural-bottlenecks-in-research-agents
  - asynchronous-multi-gpu-worker-pool
  - hidden-consistent-evaluation
  - react-agent-operator
  - evolutionary-search-over-solutions
related_mocs:
  - agent-workload-scheduling
tags:
  - architecture
  - autonomous-research
---

# Autonomous research agent architecture

The internal design of a single autonomous research agent that can run
unattended for hours-to-days against an ML benchmark. This MoC frames
the *inside* of one project's agent; the sibling MoC
[[agent-workload-scheduling]] frames the *outside* — how multiple such
agents share one workstation.

## Members

- [[structural-bottlenecks-in-research-agents]] — the three load-bearing
  limits (compute throughput, generalization gap, operator capability)
  that bind agent performance regardless of model scale. The taxonomy
  this whole MoC is organized around.
- [[asynchronous-multi-gpu-worker-pool]] — answer to throughput
  bottleneck: orchestrator + N workers in isolated dev containers,
  steady-state evolution dispatches work without barriers.
- [[hidden-consistent-evaluation]] — answer to generalization gap:
  separate search-signal split from final-scoring split; agent only
  ever sees the search signal. Already promoted to a framework-wide
  rule in `~/claude-system/claude/rules/evaluation.md`.
- [[react-agent-operator]] — answer to operator capability: replace
  single-turn prompts with reason→act→observe loops that dynamically
  scope compute to sub-problem difficulty.
- [[evolutionary-search-over-solutions]] — the orchestrator pattern
  that ties the others together: population of candidate solutions,
  rank-temperature parent selection, asynchronous mutation dispatch.

## Primary sources

- [[hambardzumyan2026aira]] — AIRA² (FAIR/UCL/Oxford 2026): names the
  three bottlenecks (inheriting from AIRA-dojo 2025) and gives one
  architectural answer per bottleneck.

## Open questions

- Does the multi-GPU worker pool gracefully degrade to 1–2 GPUs (the
  user's setup)? AIRA² is engineered for 8.
- How is "scope of comparable experiments" defined when one project
  hosts multiple tasks? `evaluation.md` already addresses this.
- What's the right per-task ReAct budget (max steps, max tool calls)
  and how does it compose with the global compute budget?

## Why this MoC is here

The point of `agentic-resource-aware` is to schedule and admit
autonomous research agents on shared hardware. Knowing what's inside
each agent — and what their throughput / evaluation / operator
constraints actually are — is what lets the coordinator make
informed admission decisions. This MoC is the "inside view"; the
coordinator works the "outside view."
