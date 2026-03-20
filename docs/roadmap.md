# Roadmap

> Big-picture vision and phase progression. For detailed acceptance criteria, see [PRD.md](PRD.md). For system design, see [living-github-account.md](living-github-account.md).

## Vision

The system bootstraps from 4 seed files to full autonomy across multiple repositories. Each phase proves the previous one works before adding complexity. The key mechanism is idle discovery — when the goal queue is empty, the heartbeat observes the environment and generates new work, so the system stays productive without human input.

## Phase Progression

8 phases from bootstrap to full autonomy. See [PRD.md](PRD.md) for detailed acceptance criteria, added files, and dependencies per phase.

| Phase | Name | Current Status |
|-------|------|----------------|
| 0 | Seed | Design |
| 1 | Single-Agent Loop | Design |
| 2 | Multi-Agent | Design |
| 3 | Self-Reflection | Design |
| 4 | Self-Supervision | Design |
| 5 | Self-Evolution | Design |
| 6 | Cross-Repo | Design |
| 7 | Full Autonomy | Design |

Key design decisions are formalized as ADRs in [architecture.md](architecture.md).

## What's Next

Phase 0 is the immediate target: create the living-core repo, push 4 seed files, verify heartbeat runs green. Everything after that is agent-driven. See [TODO.md](TODO.md) for concrete steps.
