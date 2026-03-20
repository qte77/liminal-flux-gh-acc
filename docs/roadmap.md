# Roadmap

> Big-picture vision and phase progression. For detailed acceptance criteria, see [living-account-prd.md](living-account-prd.md). For system design, see [living-github-account.md](living-github-account.md).

## Vision

A GitHub account that operates itself. Agents plan, code, review, reflect, supervise, and evolve their own infrastructure. Humans set high-level goals and handle security escalations. The system bootstraps from 4 seed files to full autonomy across multiple repositories.

## Phase Timeline

| Phase | Name | What It Proves | Status |
|-------|------|----------------|--------|
| **0** | Seed | 4 files, heartbeat runs green, human bootstraps once | Design |
| **1** | Single-Agent Loop | One agent reads goals, executes tasks, logs traces, commits state. Idle discovery generates goals when queue is empty. | Design |
| **2** | Multi-Agent | Orchestrator + coder + reviewer communicate via Issue comments. Ralph loop integrated. | Design |
| **3** | Self-Reflection | Daily reflector analyzes traces, creates improvement Issues, builds agent memory. | Design |
| **4** | Self-Supervision | Hourly supervisor catches stuck states, cost anomalies, quality drops. Escalates to human. | Design |
| **5** | Self-Evolution | Agents modify own prompts, configs, workflows within governance boundaries. | Design |
| **6** | Cross-Repo | Manage multiple project repos. Polyforge patterns internalized. | Design |
| **7** | Full Autonomy | Self-generated goals across all types. Human is escalation-only. | Design |

## Key Design Decisions

- **GitHub Actions as runtime** — no external infrastructure
- **Two-account model** — command center (qte77) observes; living account operates
- **Issues as task queue** — native, searchable, human-readable
- **Idle discovery in heartbeat** — no separate planner agent (KISS)
- **Append-only state files** — git-versioned, auditable, no external DB

See [architecture.md](architecture.md) for formal ADRs.

## What's Next

Phase 0 is the immediate target: create the living-core repo, push 4 seed files, verify heartbeat runs green. Everything after that is agent-driven. See [TODO.md](TODO.md) for concrete steps.
