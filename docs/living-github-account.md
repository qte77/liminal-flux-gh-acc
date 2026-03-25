# Architecture: Living GitHub Account

> System design for a self-evolving, agent-operated GitHub account.
> Every component passes three gates: Is it the simplest approach? Does it duplicate anything? Do we need it now?
>
> Related: [ADRs](architecture.md) | [User Stories](UserStory.md) | [PRD](PRD.md)

## Core Idea

GitHub Actions is the agent runtime. The repo is the agent's memory. Issues are the task queue. PRs are the audit trail. No external infrastructure.

## Two-Account Model

```
qte77 (Command Center)                    liminal-flux (Agent-Operated)
─────────────────────────                  ─────────────────────────────
Sets high-level goals             ──>      living-core/ (brain + state)
Monitors via GH API               <──      Status reports as Issues
polyforge: multi-repo orchestration        living-dev-*/ (Phase 6+)
Agents-eval: agent quality metrics
living-account-dashboard/: reports
```

**Goal flow**: qte77 sends goals via `repository_dispatch`. Living account processes them autonomously. Living account posts status back via cross-account Issues.

**Why two accounts**: Separation of concern. The command center observes and steers. The living account operates. Neither can accidentally break the other. The `DASHBOARD_PAT` has write access to exactly one qte77 repo.

## Runtime

| Choice | What | Why |
|--------|------|-----|
| GitHub Actions | Compute | Free tier, cron + event triggers, agents can self-modify workflows |
| `claude -p` | Agent backbone | Best coding agent, tool use built-in, runs in GHA runners |
| Repo files | Persistent state | Git-versioned, auditable, no external DB |
| GitHub Issues | Task queue | Native, searchable, labeled, assignable |
| Issue comments | Agent communication | Structured JSON in fenced blocks, human-readable |

## Repo Structure

Only files that exist from Phase 0 are listed as Phase 0. Everything else appears when its phase proves the need.

```
living-core/
  .github/workflows/
    heartbeat.yaml              [Phase 0] Cron */15min — system pulse
    goal-intake.yaml            [Phase 1] repository_dispatch — accepts goals
    agent-worker.yaml           [Phase 1] on issues[labeled:task] — executes work
    orchestrator.yaml           [Phase 2] Decomposes goals into stories
    reviewer.yaml               [Phase 2] Reviews PRs, posts quality scores
    self-reflect.yaml           [Phase 3] Daily — analyzes traces, creates improvements
    supervisor.yaml             [Phase 4] Hourly — anomaly detection, escalation
    self-evolve.yaml            [Phase 5] Applies approved self-modifications
  state/
    goals.json                  [Phase 0] Goal registry
    agent-memory.md             [Phase 3] Accumulated learnings (append-only)
    performance-log.jsonl       [Phase 1] Per-run trace metrics
    supervisor-config.json      [Phase 4] Anomaly thresholds (single source of truth)
    anomaly-log.jsonl           [Phase 4] Supervisor findings
    evolution-history.json      [Phase 5] Self-modification audit trail
  prompts/
    system-orchestrator.md      [Phase 0] The one prompt that runs Phase 0-1
    system-coder.md             [Phase 2] Worker prompt
    system-reviewer.md          [Phase 2] Reviewer prompt
    system-reflector.md         [Phase 3] Reflection prompt
    system-supervisor.md        [Phase 4] Supervision prompt
    system-evolver.md           [Phase 5] Evolution prompt
  scripts/
    evolution-preflight.sh      [Phase 5] Governance boundary enforcement
  ralph/                        [Phase 2] Git submodule — dev loop engine
  .claude/settings.json         [Phase 2] Plugin marketplace config
  AGENTS.md                     [Phase 0] Governance rules for all agents
```

**DRY principle**: Each concern has exactly one file. Thresholds live in `supervisor-config.json` only. Learnings live in `agent-memory.md` only. Governance rules live in `AGENTS.md` only.

## Agent Roles

Agents are workflows, not separate services. Each workflow is one agent with one job.

| Agent | Trigger | Does One Thing |
|-------|---------|----------------|
| Heartbeat | cron */15min | Checks goals, emits task Issues; **if idle, observes environment and generates goals** |
| Worker | label:task | Executes a task via `claude -p`, commits results |
| Orchestrator | label:goal-ready | Decomposes a goal into task Issues |
| Reviewer | pull_request | Posts structured quality score |
| Reflector | cron daily | Analyzes traces, creates improvement Issues |
| Supervisor | cron hourly | Detects anomalies, escalates when needed |
| Evolver | label:evolution-approved | Applies approved self-modifications |

**KISS**: No agent does two things. No two agents do the same thing.

## Communication

All inter-agent messages use one format — structured JSON in Issue comments:

````
```agent-message
{
  "agent": "orchestrator",
  "run_id": "run-abc123",
  "msg_type": "task_assignment",
  "payload": {"task_id": "TASK-042"}
}
```
````

**Why not a separate messaging system**: YAGNI. Issues are persistent, searchable, and human-readable. Adding Slack/webhooks/queues creates infrastructure to maintain with no proven benefit.

**Cross-account communication**:
- qte77 -> living account: `gh api repos/.../dispatches -f event_type=goal -f client_payload='{...}'`
- living account -> qte77: `gh api repos/qte77/living-account-dashboard/issues -X POST -f title="..." -f body="..."`

## Self-Reflection

The reflector is the system's learning loop. It runs daily, not continuously — reflection needs accumulated data to be useful.

```
performance-log.jsonl (7 days) ──> Reflector agent ──> Improvement Issues
                                                   ──> agent-memory.md entries
```

**What it looks for**:
- Repeated failure patterns (same error across runs)
- Cost trends (rising cost per goal)
- Skill gaps (tasks that consistently need more iterations)
- Stale improvements (old improvement Issues still open)

**What it produces**:
- `improvement` Issues with: pattern, evidence (run IDs), proposed change, expected impact
- Memory entries in Context/Problem/Solution/Evidence/References format

**Feedback loop**: Improvement Issues enter the normal orchestrator queue. The reflector's output becomes the system's next input. Memory entries are injected into all agent prompts, making every agent smarter over time.

**YAGNI guard**: The reflector does NOT modify prompts or workflows directly. It only creates Issues. The orchestrator and evolver handle execution. One concern per agent.

## Idle Discovery

When the goal queue is empty, the heartbeat switches to discovery mode instead of idling. This is one `if/else` in the heartbeat — not a new agent or workflow.

```
Heartbeat runs:
  IF goals pending → dispatch them (existing behavior)
  IF no goals pending → run discovery mode:
    1. Observe own repo state (files, Issues, CI status)
    2. Observe qte77 repos via GH API (PRs, Issues, deps, CI)
    3. Read agent-memory.md for context
    4. Generate 1 goal, write it to goals.json
    5. Next heartbeat dispatches it
```

**What Lim can observe** (available tools in GHA):

| Observation | Tool | Available |
|-------------|------|-----------|
| Own repo files, structure, gaps | `ls`, `cat`, file analysis | Phase 0 |
| Own open/closed Issues, PRs | `gh issue list`, `gh pr list` | Phase 1 |
| Own CI status, workflow runs | `gh run list` | Phase 1 |
| qte77 repo list | `gh repo list qte77` | Phase 1 (`DASHBOARD_PAT` read scope) |
| qte77 repo Issues, PRs, deps | `gh api repos/qte77/REPO/...` | Phase 4+ |
| ai-agents-research dispatches | `repository_dispatch` events | Phase 4+ |
| Performance trends | `state/performance-log.jsonl` | Phase 3+ |

**Goal types and cost gates**: Defined once in [PRD.md](PRD.md) Phase 1.

**Why the heartbeat, not the reflector**: The reflector runs daily. Idle discovery should happen within 15 minutes of the queue emptying. The heartbeat already runs at that cadence. DRY — don't add a second cron for the same trigger ("nothing to do").

**KISS**: Discovery mode is a section in `system-orchestrator.md`, not a new prompt. The orchestrator already reads goals and dispatches. Discovery adds: "if nothing to do, look around and find something."

## Self-Supervision

The supervisor is the safety net. It catches what individual agents can't see about themselves.

**Distinct from reflection**:
- Reflector: "What patterns can we learn from?" (daily, improvement-oriented)
- Supervisor: "Is something wrong right now?" (hourly, safety-oriented)

**Anomaly thresholds** (externalized in `supervisor-config.json`):

| Metric | Default | Escalation |
|--------|---------|------------|
| Cost spike | > 2x 7-day avg | Notify + continue |
| Task stuck | > 90 minutes | Escalate |
| Failure rate | > 30% in 24h | Escalate |
| Consecutive failures | > 3 | Escalate |
| Daily cost | > $5.00 | Hard stop |

**Escalation flow**: Supervisor opens Issue with `label:human-required` -> all goal processing pauses -> cross-account notification to qte77 dashboard -> human adds `label:human-resolved` -> processing resumes.

**Meta-supervision**: A daily check (runs after reflector) that reviews the supervisor's own `anomaly-log.jsonl`. Detects blind spots: if an anomaly type has zero entries for 7+ days, the supervisor might be missing it. This is one extra `claude -p` call, not a separate workflow — KISS.

**Why hourly, not real-time**: YAGNI. GHA cron granularity is sufficient. Real-time monitoring needs webhooks and external infrastructure. If hourly proves insufficient, the evolver can tighten it.

## Self-Evolution

Agents can modify their own prompts, configs, and workflows — within hard boundaries.

**What agents CAN self-modify** (no human gate):
- Prompt files in `prompts/`
- Workflow environment variables (timeouts, iteration counts, model selection)
- Anomaly thresholds in `supervisor-config.json`
- New cron schedules (if cost impact < 10% of daily budget)

**What requires human escalation**:
- Workflow `permissions:` blocks
- Secret creation or rotation (`gh secret`)
- Destructive operations (`push --force`, `repo delete`)
- External service integrations (new API keys)
- New repository creation

**Enforcement**: `evolution-preflight.sh` greps proposed diffs for forbidden patterns before any commit. Simple, auditable, no false negatives.

**Audit trail**: Every modification logged in `evolution-history.json` with: what changed, why, which Issue approved it, the actual diff. Git history provides the second layer.

## qte77 Assets

Each asset has one integration point. No asset serves two purposes.

| Asset | Role | When |
|-------|------|------|
| **ralph-loop template** | Structural reference for `living-core` | Phase 0: empty repo; cherry-pick patterns as needed |
| **ralph-loop scripts** | Dev loop engine for workers | Phase 2: git submodule at `ralph/` |
| **claude-code-utils-plugin** | Skill registry for all agents | Phase 2: `extraKnownMarketplaces` in settings |
| **polyforge** | Multi-repo orchestration from qte77 | Phase 6: qte77 runs `cc-parallel.sh` externally |
| **ai-agents-research** | Platform intel feed | Phase 4+: posts `repository_dispatch` to living-core |
| **gha-ai-changelog** | Automated CHANGELOG | Phase 2+: reusable workflow post-merge |
| **gha-dirtree-to-readme** | Auto-sync README structure | Phase 2+: reusable workflow post-merge |
| **Agents-eval** | Agent performance evaluation | Phase 4+: qte77 runs evals on living account traces |

**DRY**: No asset capability is reimplemented inside the living account. If polyforge does multi-repo orchestration, the living account calls polyforge — it doesn't rebuild it.

## Security

**Three secrets, three scopes**:

| Secret | Scope | Can Do |
|--------|-------|--------|
| `CLAUDE_API_KEY` | Anthropic API | Call claude -p |
| `LIVING_PAT` | living-account repos | Read/write repos, Issues, PRs, workflows |
| `DASHBOARD_PAT` | qte77/living-account-dashboard only | Post status reports. Nothing else. |

**Minimum privilege**: `DASHBOARD_PAT` cannot read or write anything in the living account. A compromised report pathway cannot escalate to living account control.

**Workflow permissions** (every workflow):
```yaml
permissions:
  contents: write
  issues: write
  pull-requests: write
  actions: read
```
Never: `admin`, `organization`, `security_events`.

## Tracing

One log format, one file, append-only:

```json
{"run_id":"run-abc123","timestamp":"2026-03-19T10:05:33Z","goal_id":"GOAL-017","task_id":"TASK-042","agent":"worker-1","model":"claude-sonnet-4-6","tokens_in":12400,"tokens_out":3200,"duration_s":187,"outcome":"success","cost_usd":0.047}
```

**Consumers**: Reflector reads it for patterns. Supervisor reads it for anomalies. Weekly report aggregates it for humans. One source, three readers.
