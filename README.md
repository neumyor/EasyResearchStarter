# CleanStart — 自主研究仓库

本仓库是一个**已清空历史实验产物**、以 **autoresearch skill 框架**运行的 git 研究仓库：
Agent 在 git-ledger 与实验生命周期的纪律约束下，对一个目标模型开展持续、自主、证据驱动的
迭代开发。上一轮的模型代码、实验脚本、测试、实验报告与旧存档均已清除。

仓库实际行为以仓库内指引文档与 skill 的 `SKILL.md` 为准，本 README 只提供入口与布局概览。

---

## 🧭 规范框架（四个 skill）

研究执行纪律以四个 skill 为权威来源（`.claude/skills/` 供 Claude Code，
`.agents/skills/` 供 Codex，两者内容一致）：

| Skill | 职责 | 入口 |
|---|---|---|
| `autoresearch-orchestrator` | 控制平面：决定下一步做什么，协调其余三个 | `/autoresearch-orchestrator`（Claude Code）<br>`$autoresearch-orchestrator`（Codex） |
| `research-environment` | 机器/资源发现，维护 Git-ignore 的 `ENV.md` 快照 | `/research-environment` |
| `research-git-ledger` | commit 协议：每个 agent 创建的 commit 恰好对应一条 `docs/LOG.md`，用 `Research-Log-ID` 关联 | `/research-git-ledger` |
| `research-experiment` | 实验生命周期：命名、light/heavy 记录、PRE-RUN/POST-RUN commit、CSV 结果、失败与收尾 | `/research-experiment` |

### 核心不变量

- `docs/` 是唯一 Git 跟踪的研究文档/交付根目录；`docs/LOG.md` 是研究 ledger。
- 每个 commit ↔ 一条 LOG 记录，共享 `Research-Log-ID`（`CMT-YYYYMMDD-HHMMSS-NN`）。
- 所有研究时间戳使用**北京时间（UTC+08:00）到秒**。
- 每个实验唯一 ID `exp-YYYYMMDD-HHMMSS-<slug>`；执行前 **PRE-RUN commit** 冻结代码与
  参数；之后不边改边称同一实验。
- 实验结果以 **CSV 落盘并随 `docs/experiments/<id>/` 入库**；重量产物放
  `research_run/<id>/`（Git-ignore），用 `artifacts.csv` 索引。
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
├── .claude/skills/…  .agents/skills/…   # 四个研究 skill
├── docs/
│   ├── LOG.md                     # 研究 ledger（初始化为空）
│   └── experiments/               # 轻量实验记录根（Git 跟踪）
├── research_run/                  # 重量实验产物根（Git-ignore，尚未建立）
└── ENV.md                         # 机器能力快照（Git-ignore，首次运行时生成）
```

不含任何模型/实验代码与历史实验数据 —— 均为待建。

---

## 🚀 开始研究

1. 仓库已 `git init`（分支 `main`），尚未有首个 commit。
2. 首次进入：按 `research-environment` 生成 `ENV.md`（发现 uv/conda/计算资源/git-GitHub）。
3. 用户提出自主科研请求时，从 `autoresearch-orchestrator` 进入：它会检查 git 与
   `docs/LOG.md`、恢复未完成实验，然后编排环境/实验/提交。
4. 具体实验按 `research-experiment` 生命周期执行；每次提交经 `research-git-ledger`
   记账（LOG 记录 + commit trailer 同一 `Research-Log-ID`）。

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
- 仓库当前 git 分支 `main` 无远程；需要推送到 GitHub 时遵循 `AGENTS.md` 的网络路由
  （本机 SSH-over-443）。
