# Hermes Agent Study

> Reference-architecture comparison for the self-evolving agent loop. Hermes is not a dependency; it is a production implementation used to check liminal-flux's design before Phase 2-3 are built. For decisions, see [architecture.md](architecture.md). For phase scope, see [PRD](sprints/sprint1.md).

## Source and scope

- Project: [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent), MIT licence.
- Read at `main` commit `1f91da2` (2026-10-25). Latest release at that date: v0.22.5 (2026-10-23); the original study request referenced v0.8.0.
- Sources are the repo's own docs (`website/docs/`) and code. The hosted docs site could not be reached from the research session, so the repo copy of the docs was used. File paths below are relative to the repo root at that commit.

## 1. Skill creation loop (informs Phase 2-3)

**Hermes**: Skills are `SKILL.md` files (agentskills.io format, YAML frontmatter, optional `references/`, `templates/`, `scripts/`) in `~/.hermes/skills/`. A background review after a turn (`agent/background_review.py`) decides create vs reuse with a fixed order: (1) patch a skill already loaded this session, (2) patch an existing broad "umbrella" skill, (3) add a support file under an umbrella, (4) create a new class-level umbrella only if nothing fits. Names tied to one task, PR or error string are forbidden. The review must re-read a skill before patching it and may never delete one. An optional `skills.write_approval` stages writes for human approval (`website/docs/user-guide/features/skills.md`).

**For liminal-flux**: Adopt the "patch before create" order for the reflector's improvement Issues and for prompt changes. Reject improvements named after a single incident; they belong in `agent-memory.md`, not in a new prompt or skill. Hermes' optional write approval matches our human gate on evolution (Phase 5).

## 2. Cross-session memory and search

**Hermes**: All sessions go into SQLite (`~/.hermes/state.db`) with an FTS5 index. The `session_search` tool makes no LLM call and returns real messages. Ranking is BM25 by default; `sort=newest|oldest` puts time first. Sessions from cron jobs rank below the user's own sessions; subagent and tool sessions are hidden. Results default to 3 (max 10), and only the top hit gets full context (`website/docs/user-guide/sessions.md`, `tools/session_search_tool.py`). Persistent memory is separate: `MEMORY.md` is capped at 2,200 characters and `USER.md` at 1,375 (`website/docs/user-guide/features/memory.md`).

**For liminal-flux**: Recency vs relevance is a caller choice (a sort flag), not a blended score. That is the simplest workable model; `git grep` / `gh search` over state files plus a "newest first" option covers it without a database (keeps [ADR-006](architecture.md#adr-006-append-only-state-files)). Rank automated output (heartbeat traces) below human-authored goals and Issues so it cannot crowd them out. ADR-006 already notes that append-only `agent-memory.md` grows without bound; Hermes' answer is a hard size cap, see question 5.

## 3. Cron scheduler and failure handling

**Hermes**: A gateway ticks every 60 s; each due job runs in a fresh agent session, guarded by a lock file. Every attempt is logged (claimed, running, completed, failed, unknown); abandoned attempts become `unknown` and are never re-run automatically. Retries (after 5, 15 and 30 min) happen only when a run failed before reaching the model, such as network or DNS errors (`cron.retry_unreachable`, default true). One alert is sent per error signature, repeated every 6 h; after 3 consecutive failures (`failure_nudge_threshold`) it suggests fixing, pausing or removing the job. Agents started by cron cannot edit cron jobs unless `cron.allow_agent_scheduling: true` (default false) (`website/docs/user-guide/features/cron.md`).

**For liminal-flux**: GitHub Actions already provides the scheduler and run log ([ADR-001](architecture.md#adr-001-github-actions-as-agent-runtime)). Ideas to adopt:

- Retry only infrastructure failures, never model or task failures.
- Do not re-run abandoned runs automatically.
- Escalate once per error signature instead of once per failed run. This keeps the supervisor's "consecutive failures > 3" rule (Phase 4) from flooding the dashboard.
- Keep scheduled agents off their own schedules by default. This already matches the Phase 5 rule that workflow changes need approval.

## 4. Compute backends (informs Phase 4+)

**Hermes**: Six backends are built in: local, docker, ssh, modal, daytona and vercel_sandbox. Singularity/Apptainer is an official plugin, not core. The operator picks the backend in config (`terminal.backend` or `TERMINAL_BACKEND`). Each backend only implements `execute()` and `cleanup()`. Hermes provisions the sandbox lazily, using the operator's credentials (`website/docs/user-guide/configuration.md`, `website/docs/developer-guide/terminal-environment-plugin.md`). No model-facing tool for choosing or switching backends was found; this was not an exhaustive search.

**For liminal-flux**: In Hermes the human picks the backend and the agent only uses it. If Phase 4+ self-provisions compute, keep the same split. Adding a runtime or a credential stays a governance-gated change (secret creation is already a forbidden pattern for self-evolution). Any new runtime should sit behind the same two-call interface.

## 5. Reinforcement nudges and memory governance

**Hermes**: A "nudge" is a counter that starts a background review after the reply is sent:

- Memory review runs every 10 user turns (`memory.nudge_interval`).
- Skill review runs after 10 tool iterations without a `skill_manage` call (`skills.creation_nudge_interval`).
- Cron sessions are skipped (`hermes_cli/config_defaults.py`, `agent/turn_context.py`).

Drift is controlled by limits rather than by review:

- Fixed memory size caps. A write that would overflow fails, so the agent has to consolidate entries.
- Exact duplicates are rejected.
- Writes are scanned for injection and exfiltration patterns.
- `memory.write_approval` is optional.
- A curator runs every 7 days. It marks skills unused for 14 days as stale and archives (never deletes) those unused for 30 days. The LLM merge pass is off by default (`curator.consolidate: false`). Every change is logged and can be rolled back (`website/docs/user-guide/features/curator.md`).

**For liminal-flux**: The daily reflector (Phase 3) plays the role of the nudge. To prevent drift without a human in the loop, adopt the limits rather than more review:

- Set a size cap on `agent-memory.md`; a write that would overflow must consolidate entries.
- Reject duplicate entries.
- Screen memory writes for injection patterns before they reach agent prompts.
- Archive stale learnings instead of deleting them.

## Summary for Phase 2-3 design

| Area | Validate | Revise |
| ---- | -------- | ------ |
| Skill/prompt growth | Human gate on evolution | Add a "patch before create" order to the reflector prompt |
| Memory | Plain files, no DB | Cap `agent-memory.md` size, dedupe, archive stale entries |
| Scheduling | GHA cron as scheduler | Escalate once per error signature; retry only infra failures |
| Compute | Operator-chosen runtime | Keep self-provisioning behind governance |
| Drift | Daily reflector as review loop | Screen memory writes for injection |

These are proposals. They become decisions only when adopted as ADRs or PRD acceptance criteria.
