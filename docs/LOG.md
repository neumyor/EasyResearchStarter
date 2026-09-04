# Research Log

Research ledger for this repository. Every agent-created Git commit MUST have
exactly one corresponding entry below, sharing the same `Research-Log-ID`
(format `CMT-YYYYMMDD-HHMMSS-NN`, Beijing time `UTC+08:00`, to the second).

The log records research activity across commit boundaries — code work,
experiments run, results and failures, conclusions, and artifact locations —
not a prose duplicate of `git diff`. See the `research-git-ledger` skill for
the full protocol and entry format.

---

## CMT-20260904-212308-01 — Monitor long-running experiments in background

- Beijing time: 2026-09-04 21:23:08+08:00
- Branch: main
- Parent commit: e254751
- Commit type: DOCS
- Experiment IDs: none

### Purpose

Add an explicit operating rule for experiments whose expected runtime makes
foreground waiting inefficient or opaque.

### Since previous commit

#### Code / configuration

- Required experiments not reasonably expected to finish within one minute to
  prefer a project-appropriate, observable background launch.
- Required background launches to preserve logs and record their launch method,
  task/process identifier, and expected duration.
- Required a progress report once runtime exceeds five minutes, including
  elapsed time, evidence-backed progress, and an ETA derived from observed
  progress or an explanation of why no defensible ETA exists.
- Added the same lifecycle rule to the repository-hosted Claude/Codex skills
  and portable namespaced plugin skills; bumped plugin version to `1.0.3`.

#### Documentation

- Updated workspace instructions, management rules, research guide, and README
  with the long-run background and reporting expectation.

#### Research activity

- No experiment was run; this is a prompt-only workflow change.

### Results and conclusions

Long experiments now have a clear, low-blocking execution mode and a concrete
five-minute reporting threshold. ETA must be grounded in observed work rather
than asserted without evidence.

### Artifacts and deliverables

- Git-tracked: synchronized skill instructions, portable plugin manifests,
  workspace documentation, and this ledger entry.
- Heavy/local: none.

---

## CMT-20260904-203739-01 — Make research ledger newest-first

- Beijing time: 2026-09-04 20:37:39+08:00
- Branch: main
- Parent commit: d6d0e86
- Commit type: DOCS
- Experiment IDs: none

### Purpose

Make research-log ordering deterministic and prevent routine ledger inspection
from consuming context on irrelevant historical records.

### Since previous commit

#### Code / configuration

- Added a mandatory newest-to-oldest ledger rule: each new `docs/LOG.md` entry
  is inserted immediately below the introductory separator rather than appended.
- Added progressive log access: agents begin at the file head and newest entry,
  use targeted lookup for known identifiers, and expand older entries only when
  necessary.
- Applied the same behavior to both repository-hosted skill trees and the
  portable namespaced Codex/Claude plugin skills.
- Bumped the portable plugin version from `1.0.1` to `1.0.2`.

#### Documentation

- Updated the workspace instructions, management rules, research guide, and
  README to state the write order and bounded-read expectation.

#### Research activity

- No experiment was run; this is a prompt-only workflow change.

### Results and conclusions

The ledger now exposes current research state at its head while preserving all
older records. Routine agents should retain less irrelevant history in context
without losing a path to progressively reconstruct it when required.

### Artifacts and deliverables

- Git-tracked: synchronized Claude/Codex skills, portable plugin manifests,
  workflow documentation, and this ledger entry.
- Heavy/local: none.

### Notes

- The existing ledger was already newest-first; this entry is inserted at the
  mandated location to establish the rule in both content and procedure.

---

## CMT-20260904-133324-01 — Require visual review of generated figures

- Beijing time: 2026-09-04 13:33:24+08:00
- Branch: main
- Parent commit: ff1727f
- Commit type: DOCS
- Experiment IDs: none

### Purpose

Require agents to visually inspect material generated charts and images whenever
the current host provides a direct image viewer or preview tool, so a successful
rendering command is not mistaken for a usable research artifact.

### Since previous commit

#### Code / configuration

- Added visual-artifact QA to the orchestrator, experiment lifecycle, and Git
  ledger procedures for both repository-hosted skill trees.
- Added the same rule to the portable, namespaced Codex/Claude plugin skills.
- Bumped the portable plugin version from `1.0.0` to `1.0.1`.

#### Documentation

- Added a workspace-wide visual-output rule to AGENTS.md and CLAUDE.md.
- Updated the management rules, research guide, and README with the required
  direct-viewing check and the fallback requirement to disclose when no viewer
  is available.

#### Research activity

- Verified all skill frontmatter and JSON manifests; confirmed the Claude Code
  plugin manifest validates and the Codex/Claude repository skill trees remain
  identical.

### Results and conclusions

No model experiment was run. Material figures now require a visual check of
data correspondence, labels, units, legends, clipping, layout, and readability
when direct inspection is available. If it is not available, agents must state
that limitation rather than claim visual QA.

### Artifacts and deliverables

- Git-tracked: updated repository and plugin skills, host instructions,
  research documentation, and plugin manifests.
- Heavy/local: none.

---

## CMT-20260903-174518-01 — Package portable single-writer research workflow

- Beijing time: 2026-09-03 17:45:18+08:00
- Branch: main
- Parent commit: 2770cde
- Commit type: INTEGRATION
- Experiment IDs: none

### Purpose

Strengthen the prompt-based research workflow for single-writer operation and
reproducible experiment records, then package it for safe reuse in existing
Codex and Claude Code workspaces without overwriting their code or skills.

### Since previous commit

#### Code / configuration

- Added a workspace-local, Git-metadata single-writer lock protocol. It guards
  all Agent writes, staging, and commits while leaving read-only inspection
  available to other agents.
- Expanded the experiment record into a research contract covering immutable
  dataset identity, baseline, primary metric and threshold, seeds/replications,
  resource budget, and stop condition.
- Defined the experiment ID format
  `exp-YYYYMMDD-HHMMSS-<topic>-<variant>[-seed-<seed>]` and state transitions.
- Replaced the CSV-only output rule with a required tracked
  `results/metrics.csv` summary plus indexed, appropriately formatted heavy raw
  outputs (for example Parquet, NPZ, or JSONL).
- Added `easyresearch-adopt-workspace`, an explicit opt-in skill whose default
  plugin-only mode leaves a target workspace untouched.
- Added native Codex and Claude Code plugin manifests, marketplace catalogs,
  and a portable `easyresearch-workspace` package whose skills are namespaced
  `easyresearch-*` to avoid collisions with pre-existing user skills.

#### Documentation

- Updated README with direct template usage, non-destructive adoption modes,
  and Codex/Claude marketplace installation instructions.
- Updated both host entry files and research guidance to reflect the fifth
  skill, single-writer ownership, experiment contract, naming convention, and
  metric-summary rule.
- Updated environment discovery to scan currently available proxy listeners and
  run a read-only GitHub SSH-over-443 connectivity test before networked Git
  operations.

#### Research activity

- Reviewed the four original skills for enforcement gaps and selected a pure
  prompt-based approach, as requested; no project validation scripts were
  added.
- Validated JSON manifests and every skill frontmatter header. Confirmed the
  `.agents/skills/` and `.claude/skills/` trees are identical. Claude Code's
  plugin validator accepted the plugin manifest and marketplace (the installed
  version emits only a non-blocking marketplace-description warning).

### Results and conclusions

No model experiment was run. The repository now supports two safe modes:
direct template use and plugin-based reuse in an existing workspace. A GitHub
URL alone never authorizes silent mutation; installation adds capabilities to
the agent, while persistent workspace adoption remains explicit and additive.

### Artifacts and deliverables

- Git-tracked: updated dual-host skills and guidance, Codex marketplace
  `.agents/plugins/marketplace.json`, Claude marketplace
  `.claude-plugin/marketplace.json`, and the portable plugin under
  `plugins/easyresearch-workspace/`.
- Heavy/local: none.

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
