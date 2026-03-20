# User Stories

> Who uses the system and what workflows they follow. For acceptance criteria details, see [PRD.md](PRD.md). For system design, see [living-github-account.md](living-github-account.md).

## Personas

### Human Operator (qte77)

The human who bootstraps the system, sets high-level goals, monitors health, and handles security escalations. After Phase 0, involvement is minimal and reactive.

### Agent System (liminal-flux)

The collection of workflow-based agents that autonomously plan, execute, review, reflect, supervise, and evolve. Each agent is a GitHub Actions workflow with a single responsibility.

## User Stories

### Human Operator

- **As the operator**, I want to seed and bootstrap the living account with minimal effort, so that agents can take over from a known-good state. See [Deployment Guide](deployment.md).

- **As the operator**, I want to send goals via `repository_dispatch`, so that I can steer the system without logging into the living account.

- **As the operator**, I want to receive status reports and escalations on my dashboard repo, so that I can monitor without polling.

- **As the operator**, I want escalations to pause all goal processing, so that nothing runs unsupervised when something is wrong.

- **As the operator**, I want a weekly summary report, so that I can track progress and costs at a glance.

- **As the operator**, I want hard cost limits enforced automatically, so that runaway agents cannot exceed budget.

### Agent System

- **As the heartbeat**, I want to check goals every 15 minutes and create task Issues, so that work is dispatched promptly.

- **As the heartbeat**, I want to discover useful work when the queue is empty, so that the system stays productive without human input.

- **As the worker**, I want to execute a task via `claude -p` and open a PR, so that all work is auditable and reviewable.

- **As the orchestrator**, I want to decompose goals into task Issues, so that complex goals become manageable units of work.

- **As the reviewer**, I want to post structured quality scores on PRs, so that merge decisions are objective and automated.

- **As the reflector**, I want to analyze 7 days of traces and create improvement Issues, so that the system learns from its own performance.

- **As the supervisor**, I want to detect stuck tasks, cost spikes, and failure streaks, so that problems are caught before they escalate.

- **As the evolver**, I want to modify prompts and configs within governance boundaries, so that the system improves itself safely.

## Workflow Summary

```
Operator sets goal ──> repository_dispatch ──> goal-intake workflow
                                                     │
Heartbeat (idle) ──> discovers goal ─────────────────┘
                                                     │
                                               goals.json
                                                     │
                                          Heartbeat creates Issue
                                                     │
                              ┌───────────────────────┤
                              v                       v
                         Worker executes         Orchestrator decomposes
                              │                  (complex goals only)
                              v                       │
                           Opens PR                   v
                              │                  Task Issues
                              v                       │
                         Reviewer scores              └──> Worker executes
                              │
                 ┌────────────┤
                 v            v
            Auto-merge    Block + notify
```

**Escalation path**: Supervisor detects anomaly -> `label:human-required` -> processing pauses -> dashboard notification -> operator resolves -> `label:human-resolved` -> processing resumes.
