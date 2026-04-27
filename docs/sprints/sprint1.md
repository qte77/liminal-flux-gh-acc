# Living GitHub Account — PRD

> Phased roadmap from bootstrap to full autonomy.
> Each phase builds only what the previous phase proved necessary.
> Architecture reference: [living-github-account.md](../living-github-account.md)

## Principles

- **KISS**: Every phase is the minimum addition that unblocks the next capability.
- **DRY**: Each concern has one authoritative location. Reference, don't duplicate.
- **YAGNI**: If a phase doesn't need it, it doesn't exist yet. Phase 0 is 4 files.
- **Goal-oriented**: Every agent run answers "what goal am I advancing?"
- **Actionable**: Every acceptance criterion is directly testable.

---

## Phase 0: Seed

**Goal**: Create the minimum human-bootstrapped seed. After this, agents take over.

**Duration**: ~1 hour of human work, one-time.

**Human steps**:

1. Create GitHub account
2. `gh repo create living-core --public`
3. `gh secret set CLAUDE_API_KEY`, `gh secret set LIVING_PAT`
4. Push 4 seed files, enable heartbeat workflow

**Seed files**:

| File | Content |
|------|---------|
| `.github/workflows/heartbeat.yaml` | Cron */15min, install claude, run orchestrator prompt, commit state |
| `state/goals.json` | `{"version":1,"goals":[],"last_heartbeat":null}` |
| `prompts/system-orchestrator.md` | Read goals, check Issues, execute or dispatch |
| `AGENTS.md` | Governance: what agents can/cannot do, escalation triggers |

**AC**:

- [ ] Heartbeat workflow runs green
- [ ] `state/goals.json` committed on main
- [ ] No files exist beyond these 4

---

### Seed Goal

The first human-injected goal that proves the loop: **"Generate and maintain your own README from repo contents."** Added to `goals.json` as part of Phase 0 bootstrap. Each completed goal's observations naturally surface the next one.

---

## Phase 1: Single-Agent Loop

**Goal**: One agent reads goals, executes a task, logs a trace, commits state.

**Duration**: Weeks 1-2. **Depends on**: Phase 0 green heartbeat.

**Added**:

| File | Purpose |
|------|---------|
| `.github/workflows/goal-intake.yaml` | `on: repository_dispatch[goal]` — accepts goals from qte77 |
| `.github/workflows/agent-worker.yaml` | `on: issues[labeled:task]` — runs `claude -p`, opens PR or closes Issue |
| `state/performance-log.jsonl` | One line per run: run_id, tokens, cost, duration, outcome |

**Modified**:

| File | Change |
|------|--------|
| `heartbeat.yaml` | Add goal scanning, Issue creation |
| `system-orchestrator.md` | Add goal execution logic |

**Goal Type Taxonomy**:

| Type | Example | Cost | Gate |
|------|---------|------|------|
| **Self-maintenance** | "My README is stale — regenerate" | Low | None |
| **Self-build** | "I have no CI — add lint + test workflow" | Medium | None |
| **Serve qte77** | "qte77/polyforge has 3 stale PRs — review them" | Medium | Read-only first |
| **Build tooling** | "Create a reusable GHA for dependency checking" | Medium | Supervisor cost check |
| **Research** | "CC released new feature — evaluate for my workflows" | Low | None (read-only) |
| **Strategic** | "Propose managing qte77/CABIO-test" | High | Human approval |

**Cost gates**: Estimated > $1.00 requires supervisor validation. Estimated > $5.00 requires human approval. Enforced in heartbeat logic.

**AC**:

- [ ] Manual goal in `goals.json` -> heartbeat creates Issue within 20 min -> worker executes -> PR or closed Issue
- [ ] `repository_dispatch` goal from qte77 -> same flow completes
- [ ] `performance-log.jsonl` has entry with all fields
- [ ] `goals.json` status updated to `completed` or `failed`
- [ ] Simple goal ("add hello-world function") completes without human touch
- [ ] When `goals.json` has no pending goals, heartbeat runs discovery mode and creates a goal within 15 minutes
- [ ] Self-generated goal has type label and cost estimate

---

## Phase 2: Multi-Agent

**Goal**: Orchestrator + coder + reviewer. Communicate via Issue comments. Ralph loop integrated.

**Duration**: Weeks 3-6. **Depends on**: Phase 1 proven end-to-end.

**Added**:

| File | Purpose |
|------|---------|
| `.github/workflows/orchestrator.yaml` | `on: issues[labeled:goal-ready]` — decomposes into task Issues |
| `.github/workflows/reviewer.yaml` | `on: pull_request` — posts structured quality score |
| `prompts/system-coder.md` | Worker prompt for implementation |
| `prompts/system-reviewer.md` | Reviewer prompt for quality scoring |
| `ralph/` | Git submodule — ralph-loop dev engine |
| `.claude/settings.json` | `extraKnownMarketplaces: ["qte77/claude-code-plugins"]` |

**AC**:

- [ ] Code goal -> orchestrator creates stories -> worker runs `make ralph ITERATIONS=5` -> PR
- [ ] Reviewer posts score: `{correctness, test_coverage, security, code_style, composite}`
- [ ] Auto-merge when composite >= 7.0 AND security >= 8
- [ ] PR blocks when thresholds not met
- [ ] 3 tasks run concurrently without deadlock
- [ ] Simple goal cost < $0.50

---

## Phase 3: Self-Reflection

**Goal**: Agents analyze own performance. Outcomes feed back into improvements and accumulated memory.

**Duration**: Weeks 7-10. **Depends on**: Phase 2 produces enough trace data.

**Added**:

| File | Purpose |
|------|---------|
| `.github/workflows/self-reflect.yaml` | Cron daily 02:00 UTC |
| `prompts/system-reflector.md` | Analyze traces, propose improvements |
| `state/agent-memory.md` | Append-only learnings (Context/Problem/Solution/Evidence/References) |
| `.github/ISSUE_TEMPLATE/improvement.yaml` | Structured: pattern, evidence, proposed change, expected impact |

**Feedback loop**:

```
Outcomes -> performance-log -> Reflector -> Improvement Issues -> Orchestrator -> Worker -> Better outcomes
                                        -> agent-memory.md -> All agent prompts -> Smarter agents
```

**AC**:

- [ ] After 7 days of data: at least 1 `improvement` Issue with all required fields
- [ ] `agent-memory.md` contains at least 1 entry after first run
- [ ] Orchestrator processes improvement Issues normally
- [ ] Within 14 days: at least 1 prompt/config updated from reflector output
- [ ] The update measurably changes a metric

---

## Phase 4: Self-Supervision

**Goal**: Safety net. Catches stuck states, cost anomalies, quality drops. Escalates to human.

**Duration**: Weeks 11-16. **Depends on**: Phase 3 proven (enough activity to supervise).

**Added**:

| File | Purpose |
|------|---------|
| `.github/workflows/supervisor.yaml` | Cron hourly |
| `prompts/system-supervisor.md` | Monitor metrics, detect anomalies, escalate |
| `state/supervisor-config.json` | Single source of truth for all thresholds |
| `state/anomaly-log.jsonl` | Supervisor findings per run |
| `.github/ISSUE_TEMPLATE/escalation.yaml` | Structured escalation with severity and blocked goals |

**New repo in qte77**: `living-account-dashboard` — receives reports and escalations.

**AC**:

- [ ] Stuck task > 90min -> supervisor escalates with `label:human-required`
- [ ] Cost spike > 2x average -> escalation before daily limit
- [ ] Escalation blocks goal processing until `label:human-resolved`
- [ ] qte77 dashboard receives escalation within 5 minutes
- [ ] Meta-supervision detects if an anomaly type has 0 entries for 7+ days
- [ ] No 3 consecutive false positives on same metric

---

## Phase 5: Self-Evolution

**Goal**: Agents modify own prompts, configs, workflows within governance boundaries.

**Duration**: Weeks 17-22. **Depends on**: Phase 4 proven (supervision guards against bad changes).

**Added**:

| File | Purpose |
|------|---------|
| `.github/workflows/self-evolve.yaml` | `on: issues[labeled:evolution-approved]` |
| `prompts/system-evolver.md` | Apply modifications, run pre-flight, commit via PR |
| `scripts/evolution-preflight.sh` | Blocks diffs with forbidden patterns |
| `state/evolution-history.json` | Audit: what, why, which Issue, actual diff |
| `.github/ISSUE_TEMPLATE/evolution-proposal.yaml` | Proposed diff, rationale, expected impact |

**Governance enforcement** — `evolution-preflight.sh` blocks if diff contains:
`permissions:`, `secrets\.`, `gh secret`, `push --force`, `repo delete`, `GITHUB_TOKEN`

**AC**:

- [ ] Approved proposal -> evolver modifies prompt file -> PR merged
- [ ] Evolver adjusts workflow env var successfully
- [ ] Pre-flight blocks `permissions:` change (test explicitly)
- [ ] Pre-flight blocks `gh secret set` (test explicitly)
- [ ] Every evolution logged in `evolution-history.json`
- [ ] After 30 days: at least 3 entries

---

## Phase 6: Cross-Repo

**Goal**: Manage multiple project repos. Polyforge patterns internalized.

**Duration**: Weeks 23-30. **Depends on**: Phase 5 proven (evolver can create repos).

**Added**:

| File | Purpose |
|------|---------|
| `.github/workflows/multi-orchestrator.yaml` | GHA matrix across repos, `fail-fast: false` |
| `state/multi-repo-status.json` | Aggregated status across all repos |

**New repos**: `living-dev-*` created by evolver as goals require.

**AC**:

- [ ] Multi-orchestrator fans out goal to 2+ repos in parallel
- [ ] Results aggregated back to `living-core/state/`
- [ ] At least 2 project repos beyond `living-core`
- [ ] qte77 polyforge `cc-parallel.sh` works against living account repos

---

## Phase 7: Full Autonomy

**Goal**: Lim generates and executes real goals across all types without human injection. Human is escalation-only.

**Duration**: Month 8+. **Depends on**: Phase 6 proven (multi-repo without intervention).

**Added**:

| File | Purpose |
|------|---------|
| `.github/workflows/weekly-report.yaml` | Cron weekly Mon 08:00 UTC |

No `system-strategic-planner.md` — the orchestrator prompt's discovery mode (added in Phase 1) handles goal generation. The heartbeat's idle discovery scales naturally: as Lim gains more observation capabilities in later phases, discovery mode sees more and generates richer goals.

**AC**:

- [ ] 14 consecutive days of self-generated goals, covering at least 3 goal types from the taxonomy
- [ ] Weekly report to qte77 dashboard every Monday
- [ ] Monthly cost < $50
- [ ] 10+ entries in `evolution-history.json`
- [ ] No security escalation unresolved > 48 hours

---

## Risks

| Risk | Mitigation | Phase |
|------|------------|-------|
| Runaway costs | `daily_cost_usd` hard stop in supervisor-config | 4 |
| Infinite loops | GHA `timeout-minutes` + max turns per `claude -p` | 1 |
| Bad self-modifications | `evolution-preflight.sh` + all changes via PR | 5 |
| PAT leak | GH secrets only, never in code | 0 |
| Goal drift | Human-editable `goals.json` + weekly report | 7 |
| GH rate limits | Track API calls, back off at 80% | 1 |
| Supervisor blind spots | Meta-supervision daily check | 4 |
