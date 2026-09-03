---
name: research-experiment
description: Defines the complete lifecycle and record structure for autonomous experiments, including experiment naming, planning, branch association, light/heavy artifact routing, execution, CSV results, analysis, failure handling, and closure.
---

# Research Experiment

## Purpose

Manage each experiment as a first-class research object with:

- one unique Experiment ID;
- one Git branch association;
- one Git-tracked light record directory;
- one Git-ignored heavy artifact directory;
- a written plan before execution;
- a PRE-RUN commit before execution;
- CSV-based result persistence;
- a written post-run analysis;
- a POST-RUN commit after analysis.

## Experiment ID

Use:

```text
exp-YYYYMMDD-HHMMSS-<slug>
```

Timestamp is Beijing time.

Example:

```text
exp-20260903-151823-attn-head-dim
```

Collision fallback:

```text
exp-20260903-151823-attn-head-dim-02
```

The slug should be short, lowercase, hyphenated, and descriptive.

## Branch association

Preferred branch name:

```text
research/<experiment-id>
```

Record the branch in `EXPERIMENT.md`.

Do not assume every repository must create a new branch if repository policy explicitly requires a different strategy, but every experiment must record the exact branch and base commit under which it is executed.

## Directories

For Experiment ID `<id>`:

Light:

```text
docs/experiments/<id>/
```

Heavy:

```text
research_run/<id>/
```

Required light structure:

```text
docs/experiments/<id>/
├── EXPERIMENT.md
├── params/
├── logs/
├── results/
└── artifacts.csv
```

Recommended heavy structure:

```text
research_run/<id>/
├── checkpoints/
├── visualizations/
├── intermediates/
└── exports/
```

Create only heavy subdirectories that are useful.

## Light versus heavy routing

### Light system

Use for:

- experiment plan;
- metadata;
- parameter/configuration files;
- raw textual logs that are small enough for Git;
- CSV results;
- experiment analysis and conclusion;
- heavy-artifact index.

These belong under:

```text
docs/experiments/<id>/
```

### Heavy system

Use for:

- checkpoints;
- large binary outputs;
- large visualizations;
- large exported result bundles;
- intermediate tensors/data;
- bulky generated assets;
- other outputs unsuitable for Git.

These belong under:

```text
research_run/<id>/
```

Important heavy artifacts must be listed in the corresponding Git-tracked `artifacts.csv`.

## Experiment record

Create:

```text
docs/experiments/<id>/EXPERIMENT.md
```

Preferred template:

```markdown
# Experiment: <experiment-id>

## Status

PLANNED

## Metadata

- Experiment ID:
- Created at:
- Branch:
- Base commit:
- Pre-run commit:
- Post-run commit:
- Machine environment: `ENV.md`

## Purpose

Why this experiment is being performed.

## Hypothesis

What change is expected to cause what result, and why.

## Plan

What will be changed, how the experiment will be executed,
what baseline/comparator will be used, and what success/failure
criteria will be evaluated.

## Parameters

Parameter/configuration files:

- `params/...`

Important runtime parameters:

- ...

## Artifact Plan

### Light artifacts

- parameters:
- raw logs:
- CSV results:

### Heavy artifacts

- checkpoints:
- visualizations:
- intermediate outputs:
- other:

## Execution

Pending.

## Results

Pending.

All structured or quantitative results must be stored under:

`results/*.csv`

## Analysis

Pending.

## Conclusion

Pending.
```

## Pre-run preparation

Before any experiment process is launched:

1. create Experiment ID;
2. record creation timestamp in Beijing time;
3. determine branch and base commit;
4. create light directory;
5. create heavy directory;
6. create `EXPERIMENT.md`;
7. write Purpose;
8. write Hypothesis;
9. write Plan;
10. record parameters/configuration;
11. create `artifacts.csv`;
12. modify or write experiment code;
13. ensure experiment code routes:
    - light logs/results to the light directory;
    - heavy artifacts to the heavy directory;
14. verify that Results, Analysis, and Conclusion are still empty/pending;
15. use `research-git-ledger` to create PRE-RUN commit;
16. only then launch the experiment.

## PRE-RUN immutability rule

After PRE-RUN commit, do not change anything that can materially alter the experiment outcome while still calling the execution the same experiment.

This includes, as applicable:

- source code;
- model code;
- data-processing code;
- prompts;
- parameter files;
- hyperparameters;
- dataset selection;
- seed;
- evaluation configuration;
- environment choices that materially affect results.

If such a change is necessary:

- stop the current experiment;
- record the reason;
- mark status appropriately;
- create a new Experiment ID;
- prepare again;
- create another PRE-RUN commit.

## Experiment code output requirements

While writing experiment code, design output routing before execution.

The code should make it clear where each class of output goes.

Preferred conceptual pattern:

```python
light_root = repo_root / "docs" / "experiments" / experiment_id
heavy_root = repo_root / "research_run" / experiment_id

params_dir = light_root / "params"
logs_dir = light_root / "logs"
results_dir = light_root / "results"

checkpoint_dir = heavy_root / "checkpoints"
visualization_dir = heavy_root / "visualizations"
intermediate_dir = heavy_root / "intermediates"
```

Adapt to the project language and structure.

Do not require the project to copy this exact code if a better native implementation exists.

## Results must be CSV

Every experimental result must be persisted as one or more CSV files under:

```text
docs/experiments/<id>/results/
```

This includes failed or partial runs where structured result information exists.

Examples:

```csv
metric,value
validation_loss,1.932
accuracy,0.842
```

or:

```csv
step,train_loss,val_loss
100,2.11,2.08
200,1.98,1.94
```

or for a failure:

```csv
status,error_type,error_message
failed,CudaOutOfMemoryError,out of memory during evaluation
```

Markdown may summarize the values but is not the source of truth.

## Raw logs

Preserve raw textual logs when practical under:

```text
docs/experiments/<id>/logs/
```

If raw logs are too large for Git, they may be treated as heavy artifacts, but that choice must be documented and indexed.

## Heavy artifact index

Create:

```text
docs/experiments/<id>/artifacts.csv
```

Recommended schema:

```csv
artifact_type,path,description,created_at_bjt
```

Paths should normally be repository-relative.

Add artifacts that matter to interpretation, reproducibility, debugging, or later analysis.

## Execution phase

When starting execution:

1. confirm current code corresponds to PRE-RUN commit;
2. confirm parameters correspond to PRE-RUN state;
3. confirm output directories match the experiment ID;
4. record start time;
5. run experiment;
6. preserve raw logs;
7. preserve partial outputs if failure occurs;
8. record end time and exit status.

Do not modify `EXPERIMENT.md` analysis while the experiment is still producing results except for factual execution-state updates when necessary.

## Analysis phase

After execution has ended:

1. inspect raw logs;
2. inspect every relevant result CSV;
3. inspect indexed heavy artifacts as needed;
4. compare against the stated baseline or success criteria;
5. distinguish observations from interpretation;
6. identify anomalies, confounders, uncertainty, or invalidation;
7. update `EXPERIMENT.md`.

The analysis should be based on persisted outputs, not only remembered console text.

## Completing EXPERIMENT.md

After the run, fill:

### Execution

Include:

- start time;
- end time;
- command or launch method;
- runtime environment reference;
- success/failure status;
- material runtime events.

### Results

Summarize important values and point to exact CSV files.

### Analysis

Explain:

- what happened;
- whether the hypothesis was supported;
- relevant comparisons;
- surprising behavior;
- reliability/limitations;
- implications for the next experiment.

### Conclusion

State a short actionable conclusion such as:

- retain;
- reject;
- inconclusive;
- retry with changed design;
- investigate anomaly;
- promote as new baseline.

## Final statuses

Suggested statuses:

```text
PLANNED
PREPARED
PRE-RUN COMMITTED
RUNNING
COMPLETED
FAILED
ABORTED
ANALYZED
POST-RUN COMMITTED
```

Use a status that truthfully reflects reality.

## Post-run closure

Before closing the experiment:

1. verify result CSV files exist;
2. verify logs are preserved;
3. verify heavy artifacts are located correctly;
4. verify important heavy artifacts are indexed;
5. complete Analysis;
6. complete Conclusion;
7. update status;
8. invoke `research-git-ledger`;
9. create POST-RUN commit.

## Failed experiments

A failure is still research history.

For failure:

- preserve error logs;
- store failure status in CSV when practical;
- preserve partial outputs;
- explain cause if known;
- distinguish known cause from speculation;
- record whether a retry needs a new Experiment ID;
- commit the failed experiment record.

Do not delete failed experiment records merely because no positive metric was produced.
