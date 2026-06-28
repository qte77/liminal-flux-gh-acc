# liminal-flux-gh-acc

Design docs, seed file templates, and the deployment guide for Lim Sid -- a
self-evolving, agent-operated GitHub account. Agents plan, code, review, reflect,
supervise, and evolve their own infrastructure; humans set goals and handle
security escalations. The system progresses through 8 phases (seed to full
autonomy). Status: Phase 0 ready to deploy; no running infrastructure yet.

This repo is documentation-only. When working here, edit the design docs, seed
stubs, and guide -- do not add runtime code.

## Key documents

- docs/living-github-account.md -- system design (components, mechanisms, comms)
- docs/architecture.md -- architecture decision records
- docs/sprints/sprint1.md -- phased roadmap from bootstrap to full autonomy
- docs/roadmap.md -- vision, phase status, next steps
- docs/UserStory.md -- personas and workflows
- docs/deployment.md -- end-to-end bootstrap instructions
- docs/stubs/ -- seed templates for the living-core bootstrap (docs/stubs/AGENTS.md
  is the governance stub for the deployed account, not the governance for
  contributors to this repo)

## Conventions

- Agent config is AGENTS.md-only; CLAUDE.md is a symlink to this file.
- Claude Code plugins are configured in .claude/settings.json against the
  qte77-claude-code-plugins marketplace; shared rules live in .claude/rules/.
