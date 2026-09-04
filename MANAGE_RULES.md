# Repository Agent Management Rules

本文档描述负责维护本仓库的 agent 行为规范。本仓库以 **autoresearch skill 系统**为规范
框架运行（五个 skill 的 `SKILL.md` 是各自程序细节的权威来源）。本文档只保留跨 skill、
与具体程序无关的通用规则与约定。

## 规范框架与优先级

- 研究工作按 `autoresearch-orchestrator` 编排，具体操作遵循
  `research-environment` / `research-git-ledger` / `research-experiment`。
  orchestrator 决定「下一步做什么」与流程顺序；foundational skill 规定「某类操作必须
  怎么做」；两者不得互相冲突。任何自定义的实验或提交约定不得与 skill 冲突。
- 规则冲突时按以下优先级取舍：
  1. 保护用户数据与凭据；
  2. 保留未提交或未索引的研究工作；
  3. 保留实验溯源；
  4. 保留 Git / 仓库完整性；
  5. 保留可复现性；
  6. 满足文档约定；
  7. 便利或整洁。
- 不确定时优先**检查并记录**，而不是删除、重置、覆盖或静默修复。
- **单写入者**：同一工作区任一时刻只能有一个 agent 进行写入、暂存或提交。写入前通过
  `git rev-parse --git-path easyresearch-write-lock` 获取工作区锁；其他 agent 仅可只读检查，
  不得并发修改。锁疑似遗留时，先确认原 owner 已不活跃并记录接管原因。

## Git 与提交（git ledger）

- 仓库使用 git。Agent 创建的**每一次 commit 必须有且仅有一条 `docs/LOG.md` 记录**，
  并用同一 `Research-Log-ID`（格式 `CMT-YYYYMMDD-HHMMSS-NN`，**北京时间到秒**）作为
  commit message trailer 与 LOG 条目的 join key。详见 skill `research-git-ledger`。
- 实验执行前必须有 **PRE-RUN commit**，冻结将运行的代码与参数；PRE-RUN 之后任何可能
  影响结果的内容（代码、参数、数据、seed、评估配置、影响结果的环境选择）不得在仍称
  为同一实验的情况下改动。确需改动：停跑 → 记录原因 → 标记状态 → 新建 Experiment ID
  → 重新准备与 PRE-RUN。
- 破坏性命令（`git reset --hard`、`git checkout -- <file>`、强制删除文件）除非用户明确
  要求，否则不使用。发现未提交改动时先确认来源与影响，不擅自回滚或重置。
- 提交粒度清晰：代码、实验配置、文档尽量分开提交；提交前用 `git diff` / `git diff --stat`
  审阅范围。`ENV.md` 与 `research_run/` 绝不入库；大体积数据集、模型权重、临时日志、
  缓存目录不提交。

## 运行环境

- 进入仓库先按 skill `research-environment` 读取或生成 Git-ignored 的 `ENV.md`
  （机器能力快照：uv / conda / 计算资源 / git 与 GitHub 能力）。
- 运行训练或评估时记录所使用环境（conda 环境名或 uv）及关键版本，保证可复现。
- 参考（本机示例，以 `ENV.md` 为准）：
  - conda `py310`（`/home/niuyiming/.conda/envs/py310`，含 torch + PyTorch Lightning，
    依赖齐全）——直接以完整解释器路径运行：
    ```bash
    /home/niuyiming/.conda/envs/py310/bin/python <script>.py
    /home/niuyiming/.conda/envs/py310/bin/python -m pytest tests/ -q
    ```
  - 仅当 conda 不可用或缺少依赖时 fallback 到 uv：`uv sync` → `uv run python <script>.py`。
  - 不提交虚拟环境目录；不把 secret、token、私有路径、远程凭据写入 ENV.md、日志、
    commit message 或实验记录。

## 数据与实验规范

- 数据目录约定：`resources/all_datasets/<dataset>/`，原始数据不入库。
- 每个实验一个唯一 Experiment ID：`exp-YYYYMMDD-HHMMSS-<topic>-<variant>[-seed-<seed>]`
  （北京时间）。轻量记录在
  `docs/experiments/<id>/`（Git 跟踪：`EXPERIMENT.md`、`params/`、`logs/`、`results/`、
  `artifacts.csv`）；重量产物在 `research_run/<id>/`（Git-ignore，用 `artifacts.csv` 索引）。
- 实验命名为 `exp-YYYYMMDD-HHMMSS-<topic>-<variant>[-seed-<seed>]`：时间为北京时间，
  topic/variant 均为 2–24 字符的小写 kebab-case；每次独立运行（包括不同 seed）均使用新 ID。
- **指标摘要必须以 `results/metrics.csv` 入库**；Markdown 只能总结、不能作为指标唯一来源。
  大型预测、embedding、trace 等可置于 `research_run/<id>/`，使用合适格式并在
  `artifacts.csv` 索引。失败实验同样必须留下摘要与日志。
- 新增实验脚本或修改默认超参时，说明数据集与路径、关键超参、运行命令、输出目录，
  并记录修改前后值与原因。
- 生成对结论或交付有实质影响的图表、图片或 figure（包括 Matplotlib）后，当前宿主如提供
  可直接查看图片的工具，Agent 必须至少查看一次再据此分析或交付。检查内容包括数据对应性、
  标签/单位/图例、裁切、布局与可读性；无查看工具时如实记录限制，不得声称已视觉检查。

## 验证要求

- 能运行轻量验证时，不只停留在静态检查。优先级（环境见上）：
  语法/导入检查 → 单元测试（`pytest tests/ -q`）→ 训练脚本冒烟 → 完整训练（仅在用户
  要求、资源允许、数据齐备时）。
- 无法验证时说明原因（缺数据集 / 缺 GPU / 依赖未装 / 运行时间过长）。

## 沟通规范

- 先给结论，再列关键改动与验证情况；不确定事项说明假设、风险与建议，不把猜测写成事实。
- 需要用户决策时给出具体选项与影响，不做开放式提问；发现仓库约定与用户请求冲突时，
  先说明冲突点再给最小可行调整。

## 禁止事项

- 不在未确认的情况下删除数据、日志、checkpoint 或用户未提交代码。
- 不把临时实验结果伪装成正式 benchmark；不把 test-set 调参结果表述为盲测/无偏泛化。
- 不提交机器本地绝对路径、私有凭据或不可复现配置。
- 不在没有记录的情况下改变依赖版本、默认训练参数或指标口径。
- 不为完成单个任务引入重量级依赖或全局架构改造。
