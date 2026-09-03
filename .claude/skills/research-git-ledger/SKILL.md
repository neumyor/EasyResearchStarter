---
name: research-git-ledger
description: Enforces the repository's Git commit protocol by pairing every agent-created commit with exactly one detailed docs/LOG.md entry using a shared Research-Log-ID.
---

# Research Git Ledger

## Purpose

Every Git commit created by the agent must correspond to exactly one research-ledger entry in `docs/LOG.md`.

`docs/LOG.md` records research activity across commit boundaries, not merely file diffs.

## Hard invariants

MUST:

- Never create a Git commit without first creating exactly one corresponding `docs/LOG.md` entry.
- Never create a `docs/LOG.md` entry for an agent-created commit without using the same `Research-Log-ID` in the commit message.
- Use Beijing time (`UTC+08:00`) with seconds.
- Describe activity since the previous commit broadly enough to include:
  - code work;
  - documentation work;
  - experiment activity;
  - commands/runs when materially relevant;
  - results;
  - failures;
  - conclusions;
  - delivered/generated artifacts and locations.
- Keep commit/log pairing one-to-one.
- Never place credentials or secrets in the log or commit message.
- Inspect repository state before commit.
- Preserve uncommitted work when its provenance is unclear.

## Research-Log-ID

Preferred format:

```text
CMT-YYYYMMDD-HHMMSS-NN
```

Example:

```text
CMT-20260903-153012-01
```

Use Beijing time.

If multiple commit records share the same second, increment the suffix.

## Why the log uses Research-Log-ID instead of the current commit hash

Do not attempt to write the new commit's own hash inside `docs/LOG.md` before creating that commit.

The commit hash depends on the exact committed contents, including `docs/LOG.md`, which would create a self-reference cycle.

Instead:

- put `Research-Log-ID` in `docs/LOG.md`;
- put the same ID in the commit message trailer;
- resolve the actual Git hash later from Git history.

Example commit message:

```text
exp: prepare attention head dimension experiment

Research-Log-ID: CMT-20260903-153012-01
```

## Pre-commit inspection

Before writing the log entry:

1. inspect current branch;
2. inspect HEAD and parent context;
3. inspect staged, unstaged, and untracked changes;
4. inspect recent Git history;
5. inspect the previous relevant `docs/LOG.md` entry;
6. identify all material research activity since the previous commit;
7. determine whether the planned commit boundary is coherent.

Typical inspection patterns:

```bash
git branch --show-current
git rev-parse HEAD
git status --short
git diff
git diff --cached
git log -n 5 --oneline
```

Use additional inspection where needed.

## LOG.md entry format

Preferred structure:

```markdown
## <Research-Log-ID> — <short title>

- Beijing time: <YYYY-MM-DD HH:MM:SS+08:00>
- Branch: <branch>
- Parent commit: <hash or context>
- Commit type: <type>
- Experiment IDs: <ids or none>

### Purpose

Why this commit exists as a research boundary.

### Since previous commit

#### Code / configuration

- ...

#### Research activity

- experiments prepared or executed;
- commands/runs that materially affected research state;
- failures or interruptions;
- validation/evaluation work.

#### Results and conclusions

- major results;
- negative results;
- uncertainty;
- conclusions reached so far.

#### Artifacts and deliverables

- Git-tracked documents/results and locations;
- heavy artifacts and indexed locations;
- other deliverables.

### Notes

- anything important for the next commit boundary.
```

Omit irrelevant subsections rather than inventing content.

## Commit types

Suggested types:

```text
SETUP
DOCS
PRE-RUN
POST-RUN
FAILURE
FIX
REFACTOR
INTEGRATION
DELIVERY
OTHER
```

These are semantic labels, not strict Git types.

## PRE-RUN commit requirements

Before creating a PRE-RUN commit, verify:

- unique Experiment ID exists;
- experiment branch association is recorded;
- `EXPERIMENT.md` exists;
- purpose, hypothesis, and plan are filled;
- parameters are captured;
- result/analysis/conclusion sections remain empty or pending;
- experiment code is prepared;
- output routing distinguishes light and heavy artifacts;
- `artifacts.csv` exists;
- no experiment execution has yet occurred for this experiment version.

The PRE-RUN commit is the frozen provenance boundary for the executed code and parameters.

## POST-RUN commit requirements

Before creating a POST-RUN commit, verify:

- execution has ended in success, failure, or explicit abort;
- raw logs are preserved when available;
- structured/quantitative results are stored as CSV;
- heavy artifacts are stored under `research_run/<experiment-id>/`;
- important heavy artifacts are indexed in `artifacts.csv`;
- `EXPERIMENT.md` contains analysis and conclusion;
- status is updated.

## Commit creation

After the log entry is complete and verified:

1. stage the intended files;
2. re-check the staged diff;
3. ensure `ENV.md` is not staged;
4. ensure `research_run/` is not staged;
5. create the commit;
6. include the matching `Research-Log-ID` trailer;
7. verify the commit exists and the working tree state is intentional.

Do not silently add unrelated files merely to make the tree clean.

## Recovery

If a commit was accidentally created without a corresponding log entry:

- do not hide the inconsistency;
- inspect whether amending is safe and appropriate;
- repair the one-to-one mapping while preserving user work;
- record what was repaired.

If repository history is shared or amending would rewrite published history, avoid unsafe rewriting unless explicitly appropriate.
