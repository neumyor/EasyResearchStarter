# EasyResearch Starter — 可移植的自主研究工作流

本仓库既是一个可直接开始研究的 Git 模板，也是可通过 GitHub 插件市场分发的、纯 prompt
研究工作流。它以单写入者锁、实验契约、Git ledger 与实验生命周期约束 Agent 的持续、
证据驱动迭代；默认不会改写任何其他工作区。

仓库实际行为以仓库内指引文档与 skill 的 `SKILL.md` 为准，本 README 只提供入口与布局概览。

---

## 🧭 规范框架（五个 skill）

研究执行纪律以五个 skill 为权威来源（`.claude/skills/` 供 Claude Code，
`.agents/skills/` 供 Codex；两套目录均含完整、等价的 skill）：

| Skill | 职责 | 入口 |
|---|---|---|
| `autoresearch-orchestrator` | 控制平面：决定下一步做什么，协调其余三个 | `/autoresearch-orchestrator`（Claude Code）<br>`$autoresearch-orchestrator`（Codex） |
| `research-environment` | 机器/资源发现，维护 Git-ignore 的 `ENV.md` 快照 | `/research-environment` |
| `research-git-ledger` | commit 协议：每个 agent 创建的 commit 恰好对应一条按最新优先排列的 `docs/LOG.md`，用 `Research-Log-ID` 关联 | `/research-git-ledger` |
| `research-experiment` | 实验生命周期：命名、light/heavy 记录、PRE-RUN/POST-RUN commit、CSV 结果、长实验后台监控、失败与收尾 | `/research-experiment` |
| `easyresearch-adopt-workspace` | 将工作流增量接入一个既有工作区；默认 plugin-only，不覆写已有内容 | `/easyresearch-adopt-workspace`（Claude Code）<br>`$easyresearch-adopt-workspace`（Codex） |

### 核心不变量

- `docs/` 是唯一 Git 跟踪的研究文档/交付根目录；`docs/LOG.md` 是研究 ledger。
- 每个 commit ↔ 一条 LOG 记录，共享 `Research-Log-ID`（`CMT-YYYYMMDD-HHMMSS-NN`）。
- LOG 新记录写在文件头部（说明后的首个分隔线下），保持最近到最早；读取从头部按需渐进
  展开，避免一次性载入无关历史。
- 同一工作区仅一个写入 agent：写入、暂存和提交前须持有 Git 元数据中的工作区锁；其他 agent
  仅能只读。
- 所有研究时间戳使用**北京时间（UTC+08:00）到秒**。
- 每个独立运行唯一 ID 为 `exp-YYYYMMDD-HHMMSS-<topic>-<variant>[-seed-<seed>]`；执行前
  **PRE-RUN commit** 冻结代码与参数；之后不边改边称同一实验。
- `results/metrics.csv` 是必须入库的指标摘要；重量原始结果可使用 Parquet、NPZ、JSONL 等
  合适格式放在 `research_run/<id>/`（Git-ignore），用 `artifacts.csv` 索引。
- 预计超过 1 分钟的实验优先在后台运行并保留可观察日志；运行超过 5 分钟后至少报告一次
  已运行时间、进度证据和按当前进度估算的 ETA（无法可靠估算时说明原因）。
- 生成的关键图表/图片如可被当前宿主直接预览，必须至少视觉检查一次，确认标签、图例、裁切、
  布局和数据表达正确后才能作为分析或交付依据。
- `ENV.md` 与 `research_run/` 不入库；secret/凭据绝不写入文档、LOG、commit message。

---

## 📄 指引文档

- **`MANAGE_RULES.md`** — Agent 通用行为规则与规范框架（优先级、git/ledger、环境、
  数据与实验规范、验证、沟通、禁止事项）。
- **`HOW_TO_DO_RESEARCH.md`** — 研究启动与流程要点、分层基线、假设与方法学、bad case
  审计、test-set 披露、退出条件。
- **`CLAUDE.md` / `AGENTS.md`** — 分别面向 Claude Code / Codex 的仓库级入口指令。

---

## 🗂 仓库布局（当前状态）

```text
.
├── AGENTS.md / CLAUDE.md          # Agent 入口指令（Codex / Claude Code）
├── MANAGE_RULES.md / HOW_TO_DO_RESEARCH.md / README.md
├── pyproject.toml / uv.lock       # 环境依赖（uv 锁定）
├── .gitignore                     # /ENV.md、/research_run/ 等
├── .claude/skills/…  .agents/skills/…   # Claude Code / Codex 的等价五个研究 skill
├── .claude-plugin/marketplace.json       # Claude Code 插件市场清单
├── .agents/plugins/marketplace.json      # Codex 插件市场清单
├── plugins/easyresearch-workspace/       # 双宿主、命名空间化的可安装插件
├── docs/
│   ├── LOG.md                     # 研究 ledger（初始化为空）
│   └── experiments/               # 轻量实验记录根（Git 跟踪）
├── research_run/                  # 重量实验产物根（Git-ignore，尚未建立）
└── ENV.md                         # 机器能力快照（Git-ignore，首次运行时生成）
```

不含任何模型/实验代码与历史实验数据 —— 均为待建。

---

## 🚀 开始研究

1. 仓库已 Git 初始化，并可直接用作新的研究工作区。
2. 首次进入：按 `research-environment` 生成 `ENV.md`（发现 uv/conda/计算资源/git-GitHub，
   扫描可用代理端口并以只读方式验证 GitHub 连通性）。
3. 用户提出自主科研请求时，从 `autoresearch-orchestrator` 进入：它会检查 git 与
   `docs/LOG.md`、恢复未完成实验，然后编排环境/实验/提交。
4. 具体实验按 `research-experiment` 生命周期执行；每次提交经 `research-git-ledger`
  记账（LOG 记录 + commit trailer 同一 `Research-Log-ID`）。

---

## 📦 在既有工作区安全复用

这是推荐的成熟分发路径：**安装插件，而不是把本仓库的 `.agents/skills/` 或
`.claude/skills/` 直接复制到目标项目。** 插件中的技能以 `easyresearch-` 命名空间提供，
因此不会覆盖用户已有技能或修改其代码。

1. 将本仓库 GitHub 地址交给 Agent，或按宿主执行一次明确安装：

   ```bash
   # Codex CLI
   codex plugin marketplace add neumyor/EasyResearchStarter --ref main
   codex plugin add easyresearch-workspace@easyresearch-marketplace
   ```

   ```text
   # Claude Code
   /plugin marketplace add neumyor/EasyResearchStarter
   /plugin install easyresearch-workspace@easyresearch-marketplace
   ```

2. 开启新的 Agent 会话，并使用插件提供的 namespaced `easyresearch-*` skills。
3. 默认使用 plugin-only 模式，目标工作区不会改变。只有当用户明确要求持久化接入时，调用
   `easyresearch-adopt-workspace`；它先检查冲突、获取单写入锁，并只做增量添加。

Claude Code 支持以 `.claude-plugin/marketplace.json` 为市场入口；Codex 使用
`.agents/plugins/marketplace.json` 与原生 `.codex-plugin/plugin.json`。更新插件时应发布新的
版本号，用户随后从市场更新并开启新会话。

“仅提供 GitHub URL”本身不应触发对任意工作区的静默写入或安装：该操作必须由用户或其 agent
明确授权。安装后的 plugin-only 模式已把能力附加到 Agent，而不接触目标项目；这正是避免破坏
已有代码和技能的默认安全边界。

---

## 🛠 环境

Python ≥ 3.10（`pyproject.toml` 锁定 `<3.14`），核心依赖：`torch==2.7.1`、
`pytorch-lightning>=2.1,<2.7`、`numpy` / `pandas` / `scikit-learn` 等。

- **推荐（本机）**：conda 环境 `py310`（`/home/niuyiming/.conda/envs/py310`），用完整解释器
  路径运行，避免从零安装（详见 `MANAGE_RULES.md`；以 `ENV.md` 记录为准）。
- **可复现（uv fallback）**：
  ```bash
  uv sync                 # 按 uv.lock 安装
  uv run python <script>.py
  uv run python -m pytest tests/ -q
  ```

### 数据目录约定

原始数据集放 `resources/all_datasets/<dataset>/`，不入库。

---

## 🔑 其他说明

- 包名：`pyproject.toml` 使用中性名 `cleanstart`；确定正式项目名后改 `name` 并运行
  `uv lock` 同步即可。
- 需要访问 GitHub 时，先由 `research-environment` 扫描当前可用代理端口并验证
  SSH-over-443 路由；不要依赖固定端口。Codex 参照 `AGENTS.md`，Claude Code 参照
  `CLAUDE.md`。
