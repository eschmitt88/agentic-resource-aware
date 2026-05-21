---
kind: concept
name: "evolutionary search over candidate solutions"
status: seedling
added: "2026-05-21"
sources:
  - literature/papers/hambardzumyan2026aira.md
related_concepts:
  - asynchronous-multi-gpu-worker-pool
related_experiments: []
tags:
  - search
  - orchestration
---

# Evolutionary search over candidate solutions

## Definition

Frames autonomous research as a search over a population of candidate
solutions, with the orchestrator sampling parents (one or two) and
dispatching mutation/crossover tasks to workers. AIRA²
([[hambardzumyan2026aira]]) uses asynchronous steady-state evolution
(Syswerda 1991) with temperature-scaled rank-based parent selection —
p(i) ∝ (N − rᵢ + 1)^(1/T), T → 0 yielding greedy exploitation, T → ∞
uniform exploration.

## Why it matters here

The orchestrator pattern (population + scoring + parent selection +
async dispatch) is the natural shape for the resource-aware
coordinator at the *project* level: each project is a "candidate
solution," budget headroom is the fitness landscape, and admission is
parent selection. Whether this analogy holds operationally is one of
the project's open questions.

## Connections

- Tightly coupled to [[asynchronous-multi-gpu-worker-pool]] — the
  steady-state schedule is what removes synchronization barriers.
- Sibling work: MARS, MLEvolve, PiEvolve, FM-Agent 2.0, ML-Master 2.0
  all use evolutionary or tree-search variants (cited in AIRA² §2).
- Distinct from [[react-agent-operator]] — evolutionary search is the
  *outer* loop, ReAct is the *inner* loop. The two compose.
