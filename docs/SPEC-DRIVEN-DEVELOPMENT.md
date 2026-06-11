# Spec-Driven Development 指南：OpenSpec 与 Spec-Kit

> 本文档记录在 Cursor 中安装、配置和使用 **OpenSpec** 与 **GitHub Spec-Kit** 的完整流程，适用于 spec-driven development（规格驱动开发）工作流。

## 目录

- [OpenSpec 手把手使用指南（命令详解与实战示例）](./OPENSPEC-USER-GUIDE.md)
- [什么是 Spec-Driven Development](#什么是-spec-driven-development)
- [工具对比](#工具对比)
- [环境要求](#环境要求)
- [安装 OpenSpec](#安装-openspec)
- [安装 Spec-Kit](#安装-spec-kit)
- [在 Cursor 中使用](#在-cursor-中使用)
- [项目目录结构](#项目目录结构)
- [维护与升级](#维护与升级)
- [常见问题](#常见问题)
- [参考链接](#参考链接)

---

## 什么是 Spec-Driven Development

Spec-Driven Development（规格驱动开发，SDD）是一种在 AI 辅助编程场景下的工作方式：**在写代码之前，先与 AI 对齐「要做什么、为什么做、怎么做」**，将规格文档作为可执行的工件，而不是写完后即丢弃的临时说明。

核心价值：

- 减少「凭感觉写代码」带来的返工
- 让人与 AI 在实现前达成一致
- 规格文档沉淀在仓库中，便于追溯和协作

---

## 工具对比

| 维度 | OpenSpec | Spec-Kit (GitHub) |
|------|----------|-------------------|
| 维护方 | Fission AI | GitHub |
| 运行时 | Node.js 20.19+ | Python 3.11+ / uv |
| 安装方式 | `npm install -g` | `uv tool install` |
| 工作流风格 | 轻量、迭代、无严格阶段门 | 阶段化、规范流程 |
| 适用场景 | 存量项目、小功能、快速迭代 | 新功能、大规划、严格治理 |
| 状态目录 | `openspec/` | `.specify/` + `specs/` |
| Cursor 集成 | Slash 命令 + Skills | Skills |

> **建议**：两套工具可以同时安装，但日常开发选一套主流程即可，避免规范重复。

---

## 环境要求

| 依赖 | 版本要求 | 说明 |
|------|----------|------|
| Node.js | ≥ 20.19.0 | OpenSpec 必需 |
| npm | 随 Node 安装 | 用于安装 OpenSpec |
| uv | 最新版 | Spec-Kit 推荐包管理器 |
| Python | 3.11+ | Spec-Kit 运行时（uv 会自动管理） |
| Cursor | 最新版 | IDE 集成 |

### Windows 升级 Node.js（示例）

若 Node 版本低于 20.19.0，可通过 winget 升级：

```powershell
winget install OpenJS.NodeJS.20 --accept-source-agreements --accept-package-agreements --disable-interactivity
```

验证：

```bash
node --version   # 应 >= v20.19.0
npm --version
```

---

## 安装 OpenSpec

### 1. 全局安装 CLI

```bash
npm install -g @fission-ai/openspec@latest
```

验证：

```bash
openspec --version
```

### 2. 在项目中初始化（Cursor 集成）

进入项目根目录：

```bash
cd your-project
openspec init . --tools cursor --force
```

参数说明：

| 参数 | 作用 |
|------|------|
| `.` | 在当前目录初始化 |
| `--tools cursor` | 配置 Cursor 集成（非交互） |
| `--force` | 自动清理旧文件，跳过确认 |

初始化完成后会创建：

- `openspec/config.yaml` — 项目配置
- `.cursor/commands/opsx-*.md` — Slash 命令
- `.cursor/skills/openspec-*` — Agent Skills

---

## 安装 Spec-Kit

### 1. 安装 uv（Windows）

Spec-Kit 官方推荐使用 [uv](https://docs.astral.sh/uv/) 作为包管理器：

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

安装后 `uv` 位于 `%USERPROFILE%\.local\bin`，请确保该目录在系统 PATH 中。

验证：

```bash
uv --version
```

### 2. 全局安装 specify CLI

建议固定版本号（可在 [Releases](https://github.com/github/spec-kit/releases) 查看最新 tag）：

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v0.10.1
```

验证：

```bash
specify version
specify check
```

> **注意**：Spec Kit 官方包仅来自 [github/spec-kit](https://github.com/github/spec-kit)，PyPI 上同名包与官方无关。

### 3. 在项目中初始化（Cursor 集成）

进入项目根目录：

```bash
cd your-project
specify init --here --integration cursor-agent --force --ignore-agent-tools
```

参数说明：

| 参数 | 作用 |
|------|------|
| `--here` | 在当前目录初始化（不新建子目录） |
| `--integration cursor-agent` | Cursor 集成（注意不是 `cursor`） |
| `--force` | 非空目录时跳过确认 |
| `--ignore-agent-tools` | 跳过 Agent CLI 检测 |

初始化完成后会创建：

- `.specify/` — 模板、脚本、工作流、constitution
- `.cursor/skills/speckit-*` — Agent Skills
- `.cursor/rules/specify-rules.mdc` — Cursor 规则

---

## 在 Cursor 中使用

### 前置步骤

1. 用 Cursor 打开已初始化的项目根目录
2. **重启 Cursor**（Slash 命令和 Skills 需重启后生效）
3. 在 Agent 对话中使用 Slash 命令或 Skills

---

### OpenSpec 工作流

OpenSpec 采用 **OPSX** 工作流，适合快速提出变更并迭代实现。

```
/opsx:propose "功能描述"
        ↓
  生成 proposal / design / tasks
        ↓
    /opsx:apply
        ↓
   按 tasks 实现代码
        ↓
   /opsx:archive
        ↓
   归档变更，更新 specs
```

#### 可用命令

| 命令 | 说明 |
|------|------|
| `/opsx:propose` | 提出新变更，一次性生成 proposal、design、tasks |
| `/opsx:explore` | 探索需求、澄清方向 |
| `/opsx:apply` | 按 tasks 执行实现 |
| `/opsx:sync` | 同步 spec 与代码状态 |
| `/opsx:archive` | 归档已完成的变更 |

#### 使用示例

```
/opsx:propose 为 RAG 模块添加文档分块策略配置
```

AI 会在 `openspec/changes/<变更名>/` 下生成规格文档，确认后可执行：

```
/opsx:apply
```

实现完成后：

```
/opsx:archive
```

#### 可选配置

编辑 `openspec/config.yaml`，可添加项目上下文，帮助 AI 生成更贴合项目的规格：

```yaml
schema: spec-driven

context: |
  Tech stack: Java 17, Spring Boot, Spring AI Alibaba
  项目为 Java AI 应用学习仓库，包含 RAG、Agent、MCP 等模块
```

---

### Spec-Kit 工作流

Spec-Kit 采用 **分阶段流水线**，适合从零规划、需要完整 PRD 与技术方案的场景。

```
/speckit-constitution
        ↓  建立项目原则
/speckit-specify
        ↓  写功能规格
/speckit-plan
        ↓  写技术方案
/speckit-tasks
        ↓  拆任务清单
/speckit-implement
        ↓  执行实现
```

#### 核心命令

| 命令 | 说明 |
|------|------|
| `/speckit-constitution` | 创建或更新项目治理原则 |
| `/speckit-specify` | 从自然语言描述生成功能规格 |
| `/speckit-plan` | 生成技术实现方案 |
| `/speckit-tasks` | 将方案拆解为可执行任务 |
| `/speckit-implement` | 按任务列表执行实现 |

#### 可选增强命令

| 命令 | 建议时机 |
|------|----------|
| `/speckit-clarify` | `/speckit-plan` 之前，澄清模糊需求 |
| `/speckit-checklist` | `/speckit-plan` 之后，生成质量检查清单 |
| `/speckit-analyze` | `/speckit-tasks` 之后、`/speckit-implement` 之前，做一致性分析 |
| `/speckit-taskstoissues` | 将任务转为 GitHub Issues |

#### 使用示例

```
/speckit-constitution
```

```
/speckit-specify 实现基于 Spring AI 的多模型路由，支持按任务类型自动选择 Qwen 或 DeepSeek
```

```
/speckit-plan
```

```
/speckit-tasks
```

```
/speckit-implement
```

#### 产出位置

- 项目原则：`.specify/memory/constitution.md`
- 功能规格：`specs/<功能名>/spec.md`
- 技术方案：`specs/<功能名>/plan.md`
- 任务清单：`specs/<功能名>/tasks.md`

---

### 工作流选择建议

| 场景 | 推荐工具 |
|------|----------|
| 给现有模块加一个小功能 | OpenSpec |
| 快速修复 + 留档 | OpenSpec |
| 新模块从零设计 | Spec-Kit |
| 需要 PRD + 技术方案 + 任务拆解 | Spec-Kit |
| 团队有严格阶段评审 | Spec-Kit |

---

## 项目目录结构

初始化后，项目根目录大致如下（与业务代码并存）：

```
your-project/
├── .cursor/
│   ├── commands/           # OpenSpec Slash 命令
│   │   ├── opsx-propose.md
│   │   ├── opsx-apply.md
│   │   ├── opsx-archive.md
│   │   ├── opsx-explore.md
│   │   └── opsx-sync.md
│   ├── skills/             # 两套工具的 Skills
│   │   ├── openspec-*
│   │   └── speckit-*
│   └── rules/
│       └── specify-rules.mdc
├── openspec/               # OpenSpec 配置与变更
│   ├── config.yaml
│   ├── changes/            # 进行中的变更
│   └── specs/              # 归档后的规格
├── .specify/               # Spec-Kit 基础设施
│   ├── memory/
│   │   └── constitution.md
│   ├── templates/
│   ├── scripts/
│   └── workflows/
├── specs/                  # Spec-Kit 功能规格（按功能分目录）
└── ...                     # 原有业务代码
```

---

## 维护与升级

### OpenSpec

```bash
# 升级全局 CLI
npm install -g @fission-ai/openspec@latest

# 在项目中刷新 Cursor 集成
openspec update
```

### Spec-Kit

```bash
# 升级 CLI（替换为最新 tag）
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v0.10.1

# 检查是否有新版本
specify self check

# 刷新项目中的 Agent 集成
specify integration upgrade
```

---

## 常见问题

### Slash 命令在 Cursor 中不显示

- 确认已在项目根目录打开 Cursor
- 重启 Cursor IDE
- 检查 `.cursor/commands/` 和 `.cursor/skills/` 是否存在

### `specify init` 报错 Unknown integration: 'cursor'

Cursor 的集成名是 **`cursor-agent`**，不是 `cursor`：

```bash
specify init --here --integration cursor-agent --force
```

### `uv` 或 `specify` 命令找不到

将 `%USERPROFILE%\.local\bin` 加入系统 PATH，然后重新打开终端。

### Spec-Kit 安装时 GitHub 连接失败

网络不稳定时可能失败，重试即可：

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v0.10.1
```

### OpenSpec 报 Node 版本不满足

需要 Node.js ≥ 20.19.0：

```powershell
winget install OpenJS.NodeJS.20 --accept-source-agreements --accept-package-agreements
```

### 两套工具会冲突吗？

不会覆盖彼此的核心目录（`openspec/` 与 `.specify/` 独立），`.cursor/` 下会并存两套命令和 Skills。建议日常只选一套主流程。

---

## 参考链接

- [OpenSpec GitHub](https://github.com/Fission-AI/OpenSpec)
- [OpenSpec 文档](https://github.com/Fission-AI/OpenSpec/tree/main/docs)
- [GitHub Spec-Kit](https://github.com/github/spec-kit)
- [Spec-Kit 安装指南](https://github.com/github/spec-kit/blob/main/docs/installation.md)
- [uv 安装文档](https://docs.astral.sh/uv/getting-started/installation/)
- [Spec-Kit vs OpenSpec 对比](https://hiddedesmet.com/speckit-vs-openspec)

---

## 版本记录

| 日期 | 说明 |
|------|------|
| 2026-06-11 | 初版：OpenSpec 1.4.1 + Spec-Kit 0.10.1，Cursor 集成，Node.js 20.20.2 |
