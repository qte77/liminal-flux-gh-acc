# TODO

> Actionable next steps. For acceptance criteria details, see [living-account-prd.md](living-account-prd.md). For big picture, see [roadmap.md](roadmap.md).

## Phase 0: Seed

- [ ] Create GitHub account for liminal-flux
- [ ] `gh repo create living-core --public`
- [ ] Set secrets: `CLAUDE_API_KEY`, `LIVING_PAT`
- [ ] Write `.github/workflows/heartbeat.yaml` — cron */15min, install claude, run orchestrator prompt, commit state
- [ ] Write `state/goals.json` — `{"version":1,"goals":[],"last_heartbeat":null}`
- [ ] Write `prompts/system-orchestrator.md` — read goals, check Issues, execute or dispatch
- [ ] Write `AGENTS.md` — governance rules for all agents
- [ ] Push seed files, enable heartbeat workflow
- [ ] Verify: heartbeat runs green, `goals.json` committed on main, no files beyond these 4

## Phase 0/1 Boundary: Seed Goal

- [ ] Add seed goal to `goals.json`: "Generate and maintain your own README from repo contents"

## Phase 1: Single-Agent Loop

- [ ] Add `goal-intake.yaml` — `on: repository_dispatch[goal]`
- [ ] Add `agent-worker.yaml` — `on: issues[labeled:task]`, runs `claude -p`
- [ ] Add `state/performance-log.jsonl`
- [ ] Update heartbeat: goal scanning, Issue creation, idle discovery mode
- [ ] Update `system-orchestrator.md`: goal execution logic, discovery section
- [ ] Verify: manual goal -> Issue -> worker -> PR, end-to-end
- [ ] Verify: `repository_dispatch` from qte77 completes same flow
- [ ] Verify: idle discovery triggers within 15 minutes, goals have type + cost estimate

## Phase 2+

Phases 2-7 depend on proving previous phases. See [living-account-prd.md](living-account-prd.md) for full AC.

- [ ] Phase 2: Orchestrator + coder + reviewer, Ralph loop, auto-merge thresholds
- [ ] Phase 3: Daily reflector, agent memory, improvement feedback loop
- [ ] Phase 4: Hourly supervisor, anomaly thresholds, cross-account escalation
- [ ] Phase 5: Self-evolution with governance preflight, audit trail
- [ ] Phase 6: Multi-repo orchestration, polyforge integration
- [ ] Phase 7: Full autonomy — 14 days self-generated goals, weekly reports
