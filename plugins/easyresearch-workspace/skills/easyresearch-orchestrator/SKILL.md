---
name: easyresearch-orchestrator
description: Orchestrates autonomous research work in a Git repository by coordinating environment discovery, experiment lifecycle, commit logging, execution, analysis, and repository integrity.
---

# Autoresearch Orchestrator

## Role

You are the control plane for autonomous research in this repository.

You decide what should happen next and delegate procedural responsibilities to the appropriate research skill.

Do not duplicate detailed Git, environment, or experiment procedures when a foundational skill already defines them.

## Required companion skills

- `easyresearch-environment`
- `easyresearch-git-ledger`
- `easyresearch-experiment`

## Global invariants

MUST:

- Treat `docs/` as the only root for Git-tracked research documentation and deliverables.
- Ensure every agent-created Git commit has exactly one corresponding `docs/LOG.md` entry.
- Use Beijing time (`UTC+08:00`) with seconds in all research timestamps.
- Ensure every experiment has one unique Experiment ID.
- Ensure every experiment has a PRE-RUN commit before execution.
- Never allow result-affecting code or parameters to change after PRE-RUN while still treating the run as the same experiment.
- Ensure every experiment has a Git-tracked `results/metrics.csv` summary and
  indexes any large raw outputs under `research_run/<experiment-id>/`.
- Store large artifacts under `research_run/<experiment-id>/`.
- Ensure important heavy artifacts are indexed in the experiment's Git-tracked `artifacts.csv`.
- Visually inspect each material generated chart, figure, or image when the
  current host provides a direct image viewer or preview tool; do not treat a
  successful plotting command as sufficient evidence of a usable artifact.
- Keep `ENV.md` and `research_run/` out of Git.
- Preserve and record failed experiments.
- Never store secrets in research documentation, logs, commit messages, or ENV files.

## Single-writer workspace lock

Only one agent may write to a workspace at a time. This includes edits to code,
docs, experiment records, Git staging, commits, and lock state; read-only
inspection may proceed without the lock.

Before the first write, acquire the workspace-local lock through Git metadata:

```bash
git rev-parse --git-path easyresearch-write-lock
```

Create the returned lock directory atomically, record the agent/session
identifier, task, acquisition time, and last activity time in its `OWNER.md`,
then re-check that it belongs to the current agent. Do not write if the lock is
held by another agent. Refresh the activity time during a long-running task and
release the lock only after all writes are complete.

If a lock appears stale, first inspect its owner and last activity. Take it over
only after confirming the owner is no longer active and recording the recovery
reason. Never delete an active or ambiguous lock. In a non-Git workspace, use
the additive `.easyresearch/write-lock/` location instead; do not overwrite an
existing lock.

## Startup procedure

When entering a repository:

1. Inspect repository root and acquire the single-writer lock before creating
   or changing any file.
2. Confirm `docs/` exists; create it if missing.
3. Confirm `docs/LOG.md` exists; initialize it if missing.
4. Confirm `docs/experiments/` exists; create it if missing.
5. Confirm `.gitignore` ignores:
   - `/ENV.md`
   - `/research_run/`
6. Invoke the `easyresearch-environment` procedure.
7. Inspect current Git state:
   - current branch;
   - HEAD;
   - dirty/untracked files;
   - relevant existing research branches;
   - recent `docs/LOG.md` state.
8. Inspect existing experiments and determine whether any are:
   - planned;
   - prepared;
   - running or interrupted;
   - completed but not analyzed;
   - analyzed but not committed.
9. Resume the most appropriate incomplete research state before starting unrelated work.

## Main loop

Repeat:

1. Inspect current research state.
2. Decide whether to:
   - initialize environment;
   - continue an incomplete experiment;
   - create a new experiment;
   - analyze a completed experiment;
   - create a required commit;
   - deliver documentation/artifacts;
   - stop because the research objective is satisfied or blocked.
3. Use the appropriate foundational skill.
4. Verify resulting repository and experiment state.
5. Record any required commit through `easyresearch-git-ledger`.
6. Continue.

## Experiment entry protocol

Before running any experiment:

1. Use `easyresearch-experiment` to:
   - create or confirm the Experiment ID;
   - establish the experiment branch association;
   - create the light directory;
   - create the heavy directory;
   - write the experiment plan;
   - write or capture all relevant parameters;
   - prepare experiment code;
   - route light and heavy outputs correctly;
   - leave result/analysis/conclusion sections empty.
2. Inspect the planned run for reproducibility.
3. Use `easyresearch-git-ledger` to create a PRE-RUN commit.
4. Record the resulting commit identity in the experiment record where appropriate.
5. Only then start the experiment.

## Experiment completion protocol

After execution:

1. Preserve raw logs.
2. Persist the required metric summary as `results/metrics.csv` and route large
   raw structured outputs to `research_run/<experiment-id>/`.
3. Preserve heavy artifacts under `research_run/<experiment-id>/`.
4. Update `artifacts.csv`.
5. Visually inspect material generated charts/figures/images when a direct
   viewer is available, then analyze the experiment from the metric summary,
   indexed raw outputs, and visual review.
6. Complete the experiment record.
7. Use `easyresearch-git-ledger` to create a POST-RUN commit.
8. Decide the next research action.

## Mutation after PRE-RUN

If result-affecting code, configuration, parameters, prompts, datasets, or runtime settings must change after PRE-RUN:

- do not silently continue the same experiment;
- stop or mark the current experiment appropriately;
- explain the reason in the experiment record;
- create a new Experiment ID;
- repeat preparation;
- create a new PRE-RUN commit;
- run the new experiment.

## Failure handling

If an experiment fails:

- preserve its logs;
- persist failure information in CSV when practical;
- preserve useful heavy artifacts;
- update `EXPERIMENT.md`;
- state the failure cause and evidence;
- create the appropriate POST-RUN/failure commit;
- do not erase the failed experiment from research history.

## Final delivery

All deliverable research documents must be located under `docs/`.

Before declaring work complete:

1. verify required documents exist;
2. verify `results/metrics.csv` is present and large outputs are indexed;
3. verify important heavy artifacts are indexed;
4. verify material visual artifacts were directly inspected when a viewer was
   available, or document why visual inspection was unavailable;
5. verify `docs/LOG.md` covers all agent-created commits;
6. verify `ENV.md` and `research_run/` are not tracked;
7. verify repository state is intentional and explain any remaining uncommitted files.
8. release the single-writer lock after all write work is complete.
