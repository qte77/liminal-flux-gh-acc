# TODO

> Actionable next steps. For acceptance criteria details, see [PRD.md](PRD.md). For big picture, see [roadmap.md](roadmap.md).

## Phase 0: Seed

Follow the [Deployment Guide](deployment.md) for step-by-step bootstrap commands. Verify against [PRD Phase 0 AC](PRD.md).

- [ ] Complete deployment guide Phase 0 steps
- [ ] Inject seed goal per deployment guide Phase 0/1 boundary
- [ ] Verify PRD Phase 0 AC passes

## Phase 1: Single-Agent Loop

Operational steps. For added/modified files and AC, see [PRD.md](PRD.md) Phase 1.

- [ ] Add `goal-intake.yaml`, `agent-worker.yaml`, `performance-log.jsonl`
- [ ] Update heartbeat and orchestrator prompt with goal execution + discovery
- [ ] Verify PRD Phase 1 AC passes (end-to-end goal flow, discovery, tracing)

## Phase 2+

Each phase depends on proving the previous. See [PRD.md](PRD.md) for files, AC, and dependencies.

- [ ] Phase 2: Multi-agent (orchestrator + coder + reviewer)
- [ ] Phase 3: Self-reflection
- [ ] Phase 4: Self-supervision
- [ ] Phase 5: Self-evolution
- [ ] Phase 6: Cross-repo
- [ ] Phase 7: Full autonomy
