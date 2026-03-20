# TODO

> Actionable next steps. For acceptance criteria details, see [PRD.md](PRD.md). For big picture, see [roadmap.md](roadmap.md).

## Phase 0: Seed

Operational steps to bootstrap. For seed file contents and AC, see [PRD.md](PRD.md) Phase 0.

- [ ] Create GitHub account for liminal-flux
- [ ] `gh repo create living-core --public`
- [ ] Set secrets: `CLAUDE_API_KEY`, `LIVING_PAT`
- [ ] Write and push the 4 seed files (see PRD for specs)
- [ ] Enable heartbeat workflow
- [ ] Verify PRD Phase 0 AC passes

## Phase 0/1 Boundary: Seed Goal

- [ ] Add seed goal to `goals.json`: "Generate and maintain your own README from repo contents"

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
