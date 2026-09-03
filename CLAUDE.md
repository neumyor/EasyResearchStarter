# Claude Code Repository Instructions

This repository is operated as an **autonomous research workspace**. Before any
work, read and follow `MANAGE_RULES.md` (agent behavior rules); for research
tasks also read `HOW_TO_DO_RESEARCH.md`.

## Research skills (canonical framework)

Research work is executed through the autoresearch skill system. The skills'
`SKILL.md` files are the source of truth for their procedures.

- `/autoresearch-orchestrator` — control plane: decides what happens next and
  coordinates the foundational skills. **Start here** for autonomous research.
- `/research-environment` — machine/resource discovery; maintains the
  Git-ignored `ENV.md` capability snapshot.
- `/research-git-ledger` — commit protocol: every agent-created commit maps to
  exactly one `docs/LOG.md` entry, joined by a shared `Research-Log-ID`.
- `/research-experiment` — experiment lifecycle: naming, light/heavy record
  layout, PRE-RUN/POST-RUN commits, CSV-based result persistence, failure and
  closure handling.

Do not invent experiment/commit conventions that conflict with these skills, and
never bypass the git-ledger protocol.
