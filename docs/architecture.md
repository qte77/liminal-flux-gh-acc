# Architecture Decision Records

> Formalized decisions from the system design. For full context on each decision, see [living-github-account.md](living-github-account.md).
>
> Related: [PRD](PRD.md) | [User Stories](UserStory.md)

## ADR-001: GitHub Actions as Agent Runtime

- **Status**: Accepted
- **Context**: Agents need compute to run `claude -p`, cron triggers, and event-driven workflows. Options: external VMs, container orchestration, serverless functions, or GitHub Actions.
- **Decision**: Use GitHub Actions as the sole compute runtime.
- **Consequences**: Free tier sufficient for early phases. Agents can self-modify workflows. Cron granularity limited to minutes. No persistent processes — all work is event-driven. Vendor lock-in to GitHub, acceptable given the repo is already GitHub-native.

## ADR-002: Two-Account Model

- **Status**: Accepted
- **Context**: The system needs separation between human oversight (qte77) and autonomous agent operations. A single account risks accidental cross-contamination.
- **Decision**: Two GitHub accounts — qte77 as command center, liminal-flux as agent-operated. Communication via `repository_dispatch` (goals in) and cross-account Issues (status out).
- **Consequences**: Clear blast radius — a compromised living account cannot affect qte77 repos (beyond one dashboard repo). Adds `DASHBOARD_PAT` secret with minimal scope. Requires managing two accounts but each has a single responsibility.

## ADR-003: Issues as Task Queue

- **Status**: Accepted
- **Context**: Agents need a persistent, searchable task queue. Options: external message queue (SQS, Redis), database table, flat files, or GitHub Issues.
- **Decision**: Use GitHub Issues with labels as the task queue. Goals get `label:goal-ready`, tasks get `label:task`.
- **Consequences**: No external infrastructure to maintain. Human-readable and auditable. Searchable via `gh issue list`. Labels drive workflow triggers (`on: issues[labeled:task]`). Limited throughput compared to dedicated queues — acceptable at current scale.

## ADR-004: Structured JSON in Issue Comments for Agent Communication

- **Status**: Accepted
- **Context**: Agents need to pass structured data between workflows. Options: shared files, webhook payloads, external messaging (Slack/webhooks), or Issue comments.
- **Decision**: All inter-agent messages use structured JSON in fenced code blocks (`agent-message`) within Issue comments.
- **Consequences**: Persistent and human-readable. Searchable via GitHub API. No external messaging infrastructure (YAGNI). Couples communication to GitHub Issues — acceptable since Issues are already the task queue.

## ADR-005: Idle Discovery in Heartbeat

- **Status**: Accepted
- **Context**: When the goal queue is empty, the system should find useful work rather than idle. Options: separate planner agent/workflow, or extend the existing heartbeat.
- **Decision**: Add an `if/else` to the heartbeat — if no goals pending, run discovery mode (observe repos, read memory, generate 1 goal).
- **Consequences**: No new agent, no new workflow (KISS). Discovery cadence matches heartbeat (15 min). The orchestrator prompt grows by one section. Scales naturally — as observation capabilities expand in later phases, discovery sees more.

## ADR-006: Append-Only State Files

- **Status**: Accepted
- **Context**: Agents need persistent state for goals, traces, memory, and audit trails. Options: external database, SQLite in repo, or plain files.
- **Decision**: State lives in repo files — `goals.json`, `performance-log.jsonl` (append-only), `agent-memory.md` (append-only), `evolution-history.json`.
- **Consequences**: Git-versioned and auditable. No external DB to provision or maintain. Append-only files grow unbounded — will need rotation/archival if the system runs long enough. File conflicts possible with concurrent writes — mitigated by GHA's sequential job execution within a workflow.
