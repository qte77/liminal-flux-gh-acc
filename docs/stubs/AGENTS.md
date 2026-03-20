# Agent Governance

> Stub: governance rules for living-core agents. See docs/deployment.md for usage.
> See PRD Phase 0 for spec: docs/sprints/sprint1.md

## Core Rules

1. **One agent, one job.** No agent does two things. No two agents do the same thing.
2. **All work via Issues and PRs.** Every task has an Issue. Every code change has a PR.
3. **Append-only state.** Never delete or overwrite `performance-log.jsonl` or `agent-memory.md` entries.
4. **Cost awareness.** Every goal must have a cost estimate before execution.

## What Agents CAN Do (no human gate)

- Read and write files in the repo
- Create and close Issues
- Open and merge PRs (when review score meets threshold)
- Modify prompt files in `prompts/`
- Modify workflow environment variables
- Modify anomaly thresholds in `state/supervisor-config.json`

## What Requires Human Escalation

- Workflow `permissions:` blocks
- Secret creation or rotation
- Destructive operations (`push --force`, `repo delete`)
- External service integrations (new API keys)
- New repository creation
- Any goal estimated > $5.00

## Escalation Process

1. Create Issue with `label:human-required`
2. All goal processing pauses
3. Post notification to qte77 dashboard
4. Wait for human to add `label:human-resolved`
5. Resume processing
