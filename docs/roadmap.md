# Roadmap

> Big-picture vision and phase progression. For detailed acceptance criteria, see [PRD.md](PRD.md). For system design, see [living-github-account.md](living-github-account.md).

## Vision

The system bootstraps from 4 seed files to full autonomy across multiple repositories. Each phase proves the previous one works before adding complexity. The key mechanism is idle discovery — when the goal queue is empty, the heartbeat observes the environment and generates new work, so the system stays productive without human input.

## Phase Progression

8 phases from bootstrap to full autonomy. See [PRD.md](PRD.md) for detailed acceptance criteria, added files, and dependencies per phase.

| Phase | Name | Current Status |
|-------|------|----------------|
| 0 | Seed | Ready to deploy |
| 1 | Single-Agent Loop | Design |
| 2 | Multi-Agent | Design |
| 3 | Self-Reflection | Design |
| 4 | Self-Supervision | Design |
| 5 | Self-Evolution | Design |
| 6 | Cross-Repo | Design |
| 7 | Full Autonomy | Design |

Key design decisions are formalized as ADRs in [architecture.md](architecture.md).

## What's Next

### Phase 0: Seed (immediate target)

Design complete. Seed file stubs and deployment guide ready. Remaining work is execution.

- [x] System design ([living-github-account.md](living-github-account.md))
- [x] PRD with phased AC ([PRD.md](PRD.md))
- [x] ADRs ([architecture.md](architecture.md))
- [x] User stories ([UserStory.md](UserStory.md))
- [x] Seed file stubs ([stubs/](stubs/))
- [x] Deployment guide ([deployment.md](deployment.md))
- [ ] Create GitHub account + `living-core` repo
- [ ] Set secrets (`CLAUDE_API_KEY`, `LIVING_PAT`)
- [ ] Push seed files, verify heartbeat runs green
- [ ] Inject seed goal, verify loop completes

### Phase 1: Single-Agent Loop

For added/modified files and AC, see [PRD Phase 1](PRD.md).

- [ ] Add `goal-intake.yaml`, `agent-worker.yaml`, `performance-log.jsonl`
- [ ] Update heartbeat and orchestrator prompt with goal execution + discovery
- [ ] Verify PRD Phase 1 AC passes

### Phase 2+

Each phase depends on proving the previous. See [PRD](PRD.md) for files, AC, and dependencies.
