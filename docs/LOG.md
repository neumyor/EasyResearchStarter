# Research Log

Research ledger for this repository. Every agent-created Git commit MUST have
exactly one corresponding entry below, sharing the same `Research-Log-ID`
(format `CMT-YYYYMMDD-HHMMSS-NN`, Beijing time `UTC+08:00`, to the second).

The log records research activity across commit boundaries — code work,
experiments run, results and failures, conclusions, and artifact locations —
not a prose duplicate of `git diff`. See the `research-git-ledger` skill for
the full protocol and entry format.

---

## CMT-20260903-161527-01 — Initialize autoresearch framework repository

- Beijing time: 2026-09-03 16:15:27+08:00
- Branch: main
- Parent commit: none (initial commit)
- Commit type: SETUP
- Experiment IDs: none

### Purpose

Reset the repository from a prior PhaseFormer experiment history to a blank
workspace that runs under the autoresearch skill framework, and capture the
initial state as the git baseline.

### Since previous commit

#### Code / configuration

- Deleted all prior-experiment artifacts: model `src/`, `scripts/`, `tests/`,
  `config/`, `archive/`, `gift_eval/`, `log/`, `docs/` reports/figures, root
  experiment-plan files, and all `run_*.py` entry points.
- `pyproject.toml`: renamed project to neutral `cleanstart`; removed the stale
  `gift-eval` optional dependency group and the obsolete CUDA/RTX4090 comment;
  re-ran `uv lock` (lock pruned to 76 packages, root name synced).
- `.gitignore`: rewritten for the framework — ignores `/ENV.md`, `/research_run/`,
  `/resources/`, and local tooling config; blanket `*.csv` ignore removed so
  experiment results under `docs/` stay trackable.
- Installed the four autoresearch skills (`autoresearch-orchestrator`,
  `research-environment`, `research-git-ledger`, `research-experiment`) into
  both `.claude/skills/` and `.agents/skills/`; removed the superseded
  `experiment-and-error-analysis` skill.
- `git init` (branch `main`); created `docs/LOG.md` and `docs/experiments/`.

#### Documentation

- Rewrote `README.md`, `MANAGE_RULES.md`, `HOW_TO_DO_RESEARCH.md`, `CLAUDE.md`,
  and `AGENTS.md` to describe the autoresearch framework and its conventions
  (ENV.md, docs/LOG.md ledger + Research-Log-ID, PRE-/POST-RUN commits,
  docs/experiments light records + research_run/ heavy artifacts, Beijing time,
  no secrets in tracked docs).

### Results and conclusions

No experiments run. Environment lock verified (`uv lock --check` and a full
`uv lock` after dependency edits both succeeded). Repository is a clean,
git-initialized starting point for the first autonomous research round.

### Artifacts and deliverables

- Git-tracked: guidance docs, four research skills (both agent ecosystems),
  `pyproject.toml` / `uv.lock`, `.gitignore`, `docs/LOG.md`, `docs/experiments/`.
- Heavy/local: none. `ENV.md` to be generated on first `research-environment` run.

---

