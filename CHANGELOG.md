<!-- markdownlint-disable MD024 no-duplicate-heading -->

# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

**Types of changes**: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`

## [Unreleased]

## [0.1.0] - 2026-03-20

### Added

- Project description paragraph in README
- `docs/roadmap.md` — phase timeline, vision, key design decisions
- `docs/TODO.md` — actionable Phase 0/1 checklist
- `docs/architecture.md` — 6 ADRs (GHA runtime, two-account model, Issues as queue, JSON comments, idle discovery, append-only state)
- Idle discovery mechanism: heartbeat generates goals when queue is empty
- Goal type taxonomy (6 types) with cost gates in PRD Phase 1
- Seed goal ("Generate README from repo contents") at Phase 0/1 boundary
- PR template with conventional commits, self-review, security checklist
- Issue templates: bug report (GHA/agent context), question (links to docs)
- `.gitmessage` conventional commits template
- `CHANGELOG.md`

### Changed

- Phase 7 rewritten: removed `system-strategic-planner.md`, discovery scales via heartbeat
- Heartbeat role updated to include idle observation and goal generation
- `README.md`: added Lim Sid nickname

## [0.0.1] - 2026-03-18

### Added

- Architecture doc: two-account model, agent roles, communication, self-reflection, self-supervision, self-evolution, security, tracing
- PRD: 8-phase roadmap from seed to full autonomy (Phase 0-7)
- `README.md` with project overview and doc links
- MIT license, `.gitignore`
