# liminal-flux-gh-acc

[![License](https://img.shields.io/badge/license-MIT-58f4c2.svg)](LICENSE)
![Version](https://img.shields.io/badge/version-0.2.0-58f4c2.svg)

**Lim Sid** — a self-evolving, agent-operated GitHub account. Agents plan, code, review, reflect, supervise, and evolve their own infrastructure. Humans set goals and handle security escalations.

Lim Sid is an agent-operated GitHub account. The docs defined 7 phases from bootstrap to autonomy, but goal-setting was vague — Phase 7 said "agents set own sub-goals from metrics" with no concrete mechanism. We added idle discovery: the heartbeat generates goals when the queue is empty. No new agents, no new workflows.

## Documents

- [Architecture — System Design](docs/living-github-account.md) — components, mechanisms, communication
- [Architecture — ADRs](docs/architecture.md) — architecture decision records
- [PRD](docs/PRD.md) — phased roadmap from bootstrap to full autonomy
- [Roadmap](docs/roadmap.md) — big-picture vision and phase progression
- [TODO](docs/TODO.md) — actionable next steps
- [User Stories](docs/UserStory.md) — personas and workflows
- [Deployment Guide](docs/deployment.md) — end-to-end bootstrap instructions

## Status

Design phase. No running infrastructure yet.
