# agentic-resource-aware

**Planning, scheduling, and resource-allocating concurrent autonomous-research
agents on a single shared workstation.**

## What this is

Several autonomous research agents share one machine, and their workloads
look nothing alike: literature sweeps are token-heavy and idle on the GPU, model
training pins VRAM for hours, tool-using experiments burst CPU and RAM. Nothing
about the mix is predictable in advance. This project asks how to admit,
throttle, or defer that work against real-time resource state so the box stays
usable and no run is launched into a collision. The output is twofold: a
literature and concept graph on scheduling, admission control, and multi-tenant
resource management, and a working coordinator policy that reads live CPU /
RAM / GPU / token headroom and decides what runs when.

Personal research project. See `CLAUDE.md` for the agent-facing orientation
and `~/.claude/CLAUDE.md` for the framework's durable principles.

## Quick start

```sh
make env       # uv sync
make lint      # orphan/dead-link check
```

## Layout

- `raw/` — immutable sources (papers, repos, web captures).
- `literature/` — processed notes.
- `concepts/` / `mocs/` — knowledge graph.
- `experiments/` — runs, each dated and slugged.
- `docs/decisions/` — ADRs.
- `journal/` — per-session log.
- `_meta/` — index, log, templates.
