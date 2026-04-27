# Deployment Guide

> End-to-end guide to bootstrap the living account. For requirements and AC, see [PRD Phase 0](sprints/sprint1.md). For system design, see [living-github-account.md](living-github-account.md).

## Prerequisites

- A new GitHub account (e.g., `liminal-flux`) — separate from the command center (qte77)
- [GitHub CLI](https://cli.github.com/) (`gh`) installed and authenticated as the new account
- An [Anthropic API key](https://console.anthropic.com/) for `claude -p`
- A fine-grained PAT for the living account with repo + Issues + PRs + workflow permissions

## Phase 0: Bootstrap

After this, agents take over.

### 1. Create the repo

```bash
gh repo create living-core --public --clone
cd living-core
```bash

### 2. Set secrets

```bash
gh secret set CLAUDE_API_KEY    # Anthropic API key
gh secret set LIVING_PAT        # Living account PAT (repo + issues + PRs + workflows)
```

### 3. Copy seed files

The seed file stubs are in [`docs/stubs/`](stubs/). Copy them into the living-core repo:

```bash
# Workflow
mkdir -p .github/workflows
cp /path/to/liminal-flux-gh-acc/docs/stubs/heartbeat.yaml .github/workflows/

# State
mkdir -p state
cp /path/to/liminal-flux-gh-acc/docs/stubs/goals.json state/

# Orchestrator prompt
mkdir -p prompts
cp /path/to/liminal-flux-gh-acc/docs/stubs/system-orchestrator.md prompts/

# Governance
cp /path/to/liminal-flux-gh-acc/docs/stubs/AGENTS.md .
```bash

Review and customize each file before committing. The stubs are starting points — adapt the orchestrator prompt and governance rules to your needs.

### 4. Push and enable

```bash
git add .
git commit -m "feat: phase 0 seed files"
git push origin main
```

The heartbeat workflow starts automatically on the next cron tick (within 15 minutes). To trigger immediately:

```bash
gh workflow run heartbeat
```bash

### 5. Verify

Check the [PRD Phase 0 AC](sprints/sprint1.md) against the running system:

```bash
gh run list --workflow=heartbeat
```

## Phase 0/1 Boundary: Inject Seed Goal

Once the heartbeat runs green, inject the first goal:

```bash
# Edit goals.json to add the seed goal
cat > state/goals.json << 'EOF'
{
  "version": 1,
  "goals": [
    {
      "id": "GOAL-001",
      "type": "self-maintenance",
      "description": "Generate and maintain your own README from repo contents",
      "status": "pending",
      "cost_estimate_usd": 0.10,
      "created": "2026-03-20T00:00:00Z"
    }
  ],
  "last_heartbeat": null
}
EOF

git add state/goals.json
git commit -m "feat: inject seed goal"
git push
```bash

The next heartbeat picks up the goal, creates a task Issue, and the loop begins.

## Monitoring

Once running, monitor via:

| What | Where |
|------|-------|
| Workflow runs | `gh run list` on living-core |
| Task queue | `gh issue list --label task` on living-core |
| Goal status | `state/goals.json` on main branch |
| Traces | `state/performance-log.jsonl` (Phase 1+) |

Cross-account setup (dashboard repo, `DASHBOARD_PAT`, escalation routing) is Phase 4 scope. See [PRD Phase 4](sprints/sprint1.md).

## Seed File Reference

| Stub | Target in living-core | Description |
|------|----------------------|-------------|
| [`stubs/heartbeat.yaml`](stubs/heartbeat.yaml) | `.github/workflows/heartbeat.yaml` | Cron */15min, runs orchestrator prompt |
| [`stubs/goals.json`](stubs/goals.json) | `state/goals.json` | Empty goal registry |
| [`stubs/system-orchestrator.md`](stubs/system-orchestrator.md) | `prompts/system-orchestrator.md` | Orchestrator prompt with discovery mode |
| [`stubs/AGENTS.md`](stubs/AGENTS.md) | `AGENTS.md` | Governance rules and escalation triggers |
