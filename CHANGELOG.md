<!-- markdownlint-disable MD024 no-duplicate-heading -->

# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

**Types of changes**: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`

## [Unreleased]

### Added

- `docs/deployment.md` — end-to-end bootstrap guide
- `docs/stubs/` — 4 seed file templates (heartbeat.yaml, goals.json, system-orchestrator.md, AGENTS.md)

### Changed

- Merged `TODO.md` into `roadmap.md` "What's Next" section (DRY)
- README description rewritten as forward-looking project summary
- Roadmap Phase 0 status: Design -> Ready to deploy
- Fixed v0.1.0/v0.0.1 CHANGELOG attribution

### Removed

- `docs/TODO.md` — content absorbed into roadmap

## [0.2.0] - 2026-03-20

### Added

- `docs/UserStory.md` — personas (human operator, agent system) and workflow summary
- Version badge in README

### Changed

- PRD moved to `docs/sprints/sprint1.md`, symlinked at `docs/PRD.md`
- Doc hierarchy enforced: removed duplicate content from `roadmap.md` and `TODO.md` (reference PRD instead)
- Removed score threshold from `UserStory.md` workflow diagram (PRD authority)
- Added cross-reference links in `living-github-account.md` and `architecture.md`
- Updated all cross-references to new PRD path (README, PR template, Issue template, roadmap, TODO, system design)

## [0.1.0] - 2026-03-20

### Added

- Project description paragraph in README
- `docs/roadmap.md` — phase timeline, vision, key design decisions
- `docs/TODO.md` — actionable Phase 0/1 checklist
- `docs/architecture.md` — 6 ADRs (GHA runtime, two-account model, Issues as queue, JSON comments, idle discovery, append-only state)
- Version badge in README

## [0.0.1] - 2026-03-18

### Added

- Architecture doc: two-account model, agent roles, communication, self-reflection, self-supervision, self-evolution, security, tracing
- PRD: 8-phase roadmap from seed to full autonomy (Phase 0-7)
- Idle discovery mechanism: heartbeat generates goals when queue is empty
- Goal type taxonomy (6 types) with cost gates in PRD Phase 1
- Seed goal ("Generate README from repo contents") at Phase 0/1 boundary
- PR template with conventional commits, self-review, security checklist
- Issue templates: bug report (GHA/agent context), question (links to docs)
- `.gitmessage` conventional commits template
- `CHANGELOG.md`
- `README.md` with project overview, Lim Sid nickname, and doc links
- MIT license, `.gitignore`

### Changed

- Phase 7 rewritten: removed `system-strategic-planner.md`, discovery scales via heartbeat
- Heartbeat role updated to include idle observation and goal generation
