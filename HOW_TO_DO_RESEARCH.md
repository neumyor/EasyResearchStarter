# 自主研究开展指南（Autoresearch）

本文档说明在本仓库内开展持续、自主、证据驱动的模型研究：如何启动、按什么流程跑通一轮
「假设 → 最小实现 → 对照实验 → bad case 分析 → 决策」，以及应遵守的研究方法学。

**执行纪律与记录格式以四个 skill 的 `SKILL.md` 为准**：`autoresearch-orchestrator`、
`research-environment`、`research-git-ledger`、`research-experiment`。本文档只补充
跨实验、与程序无关的方法学原则，不与其冲突。

---

## 1. 如何启动

- 进入仓库后先按 skill `research-environment` 读取 / 生成 Git-ignored 的 `ENV.md`
  （机器能力快照）。
- 用户提出自主科研、持续迭代、实验驱动优化等请求时，调用 `/autoresearch-orchestrator`
  （Claude Code）或 `$autoresearch-orchestrator`（Codex）。它会检查 git 与 `docs/LOG.md`
  状态、恢复未完成的实验，然后进入主循环。
- 单次明确请求需要完整对照实验与样本级错误分析时，先写 EXPERIMENT.md（含基线、目标、
  退出条件），走完 skill `research-experiment` 的生命周期，不要另起一套存储约定。

## 2. 实验纪律（要点）

- **Experiment ID**：`exp-YYYYMMDD-HHMMSS-<slug>`（北京时间到秒）。
- **轻量记录**：`docs/experiments/<id>/`，Git 跟踪，含 `EXPERIMENT.md`、`params/`、
  `logs/`、`results/`、`artifacts.csv`。
- **重量产物**：`research_run/<id>/`（Git-ignore），在 `artifacts.csv` 中索引。
- **结果必须落 CSV**：定量/结构化结果写入 `results/*.csv` 并入 Git；Markdown 只能总结。
- **PRE-RUN 冻结**：执行前先 PRE-RUN commit 冻结代码与参数；之后不得边改边称同一实验。
  失败/中止也是结果，记录原因并提交（见 skill `research-experiment`）。
- **时间戳**：研究日志一律北京时间（`UTC+08:00`），带秒。
- **秘密**：ENV.md、docs、commit message、实验记录中不得出现 token / 凭据。

## 3. 初始化与基线

在修改任何模型前先理解当前项目结构（入口、数据加载、模型定义、训练/评估、指标、
结果存储），再**分层建立基线**，不要一上来跑全量：

1. 静态检查：依赖、入口、数据路径、GPU/CPU 可用性（对照 `ENV.md`）。
2. smoke test：最小成本确认训练与评估链路可跑通。
3. 快速基线：小 epoch、少数 horizon、代表性数据集/子集，了解指标量级与资源开销。
4. 重点基线：仅在目标相关的数据集 / horizon / 场景上扩大。
5. 全量基线：仅在需要正式比较、资源允许、链路已证明稳定时运行。

小轮数 / 数据子集 / 复用 checkpoint 的结果只能用于诊断，不能作为最终效果结论。

## 4. 假设与方法学

- 每次改动对应一个明确假设；先建立基线再评价改进；每次失败都转化为下一轮可执行方向。
- **创新性筛选**：须提出清晰的机制 / 结构约束 / 归纳偏置 / 理论动机 / 直接针对 bad case
  的算法改进。仅「新增 residual 分支」「gate 初始化 0.999」「切 loss/lr/batch size」最多
  算消融或工程 baseline，不能包装成新方法。
- 避免把 dataset-aware / horizon-aware 的 guardrail 当作“最佳结果”的来源；同一数据集内
  跨 horizon 随意换方案须有机制解释与证据。以提出可泛化、可解释的统一机制为目标。
- 选择假设优先考虑：与用户目标契合、机制简单可归因、有清晰直觉、验证成本低、
  对当前 bad case 有直接解释力。

## 5. bad case 与审计

- 关键实验后从**落盘的预测/误差产物**（CSV / npz / 图）中程序化挑选 bad case，覆盖不同
  失败模式，数量可控。
- 结论用的 bad case 必须可审计：实验 ID 与 commit、数据集/split、样本索引/通道/时间窗口、
  预测值与真值、误差度量、图表与数据文件路径、错误模式与归因、下一步动作。
- 只把可测量内容写成观察（“指标高 0.083”“峰值在 step X”）；“没学会趋势”“机制导致不稳”
  等只能标为假设并附验证方法。趋势/周期/突变/峰值/低谷/长漂移/噪声/通道关系/训练稳定性
  等都是常见分析维度。

## 6. 成本、记录与披露

- 默认走最低成本验证路径；扩展实验需证据支持。把推理成本、显存纳入模型质量的考量。
- 允许依用户要求在 test 上调参，但必须：记录全部参与选择的配置与指标、在 EXPERIMENT.md /
  LOG 中写明 test-set selection，不得表述为盲测或无偏泛化估计。
- 关键实验的可复现信息（日期、commit/Research-Log-ID、数据集/horizon/split、命令、
  参数、指标、环境、产物路径、结论与失败原因）随 skill `research-git-ledger` /
  `research-experiment` 的记录规则落盘；失败实验同样记录。

## 7. 退出条件

用户显式退出条件优先。默认同时满足才算停止：

- **效果**：当前最佳方案在目标场景上优于基线或达成合理平衡；
- **证据**：关键结论有重复 seed / 多 horizon / 消融 / bad case 证据支撑并可溯源；
- **资源**：继续迭代的预期收益低于计算 / 时间 / 工程成本，或遇到明确资源/数据限制。

停止时总结：最终方案、相对基线的改进、采纳与淘汰的假设、关键 bad case 变化、残余风险
与后续方向。
