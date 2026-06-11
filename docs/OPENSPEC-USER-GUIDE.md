# OpenSpec 手把手使用指南

> OpenSpec 实战教程：每个命令怎么用、什么时候用、完整示例。  
> 安装说明见 [SPEC-DRIVEN-DEVELOPMENT.md](./SPEC-DRIVEN-DEVELOPMENT.md)。

## 目录

- [OpenSpec 是什么](#openspec-是什么)
- [使用前准备](#使用前准备)
- [文件结构说明](#文件结构说明)
- [完整生命周期](#完整生命周期)
- [五个命令详解](#五个命令详解)
- [真实示例：完整走一遍](#真实示例完整走一遍)
- [完整闭环示例](#完整闭环示例)
- [终端 CLI 命令](#终端-cli-命令)
- [常见问题](#常见问题)
- [速查表](#速查表)

---

## OpenSpec 是什么

OpenSpec 是 **「先写规格、再写代码」** 的工作流，运行在 Cursor Agent 里。

| 传统方式 | OpenSpec 方式 |
|----------|---------------|
| 「帮我在某个服务里加个功能」→ AI 直接改代码 | 先对齐做什么、为什么、怎么做 → 审阅文档 → 再按 tasks 写代码 → 归档 |

**价值：** 少返工、需求可追溯、AI 不会乱改无关模块。

---

## 使用前准备

### 1. 打开项目根目录

```
<项目根目录>    # 含 openspec/ 与 .cursor/ 的仓库根路径
```

不要在子模块或子服务目录里打开（例如单独的 `backend-api/`、`module-a/`）。

### 2. 使用 Agent 模式

Slash 命令在 **Cursor Agent 对话**里输入 `/` 弹出。

### 3. 命令名是连字符，不是冒号

| 文档/common 写法 | Cursor 实际命令 |
|------------------|-----------------|
| `/opsx:propose` | **`/opsx-propose`** |
| `/opsx:apply` | **`/opsx-apply`** |
| `/opsx:explore` | **`/opsx-explore`** |
| `/opsx:archive` | **`/opsx-archive`** |
| `/opsx:sync` | **`/opsx-sync`** |

### 4. 重启 Cursor

安装 OpenSpec 或 `openspec update` 后需重启 IDE，Slash 命令才会生效。

---

## 文件结构说明

```
your-project/
├── openspec/
│   ├── config.yaml          ← 项目背景（AI 每次 propose/apply 都会读）
│   ├── specs/               ← 归档后的「正式规格」（长期积累）
│   │   └── <capability>/
│   │       └── spec.md
│   └── changes/             ← 进行中的变更（临时工作区）
│       └── archive/         ← 已归档的历史变更
│           └── YYYY-MM-DD-<change-name>/
└── .cursor/commands/        ← Slash 命令定义（opsx-*.md）
```

| 目录 | 含义 |
|------|------|
| `openspec/changes/<名>/` | 正在做的功能（草稿区） |
| `openspec/specs/` | 做完并 sync 后的正式规格（图书馆） |
| `openspec/config.yaml` | 整个项目的说明书 |
| `openspec/changes/archive/` | 结案后的变更历史 |

每个进行中的变更通常包含：

| 文件 | 内容 |
|------|------|
| `proposal.md` | 为什么做、改什么、影响范围 |
| `design.md` | 技术方案 |
| `specs/<capability>/spec.md` | 需求规格（WHEN/THEN 场景） |
| `tasks.md` | 可勾选的任务清单 |

---

## 完整生命周期

```
/opsx-explore     可选：了解代码、讨论方案（不写业务代码）
       ↓
/opsx-propose     创建变更 + 生成 proposal / design / specs / tasks
       ↓          ← 你审阅文档，不满意就继续说「改 proposal 里的 xxx」
/opsx-apply       按 tasks.md 逐步实现
       ↓
/opsx-sync        可选：把变更 specs 合并到 openspec/specs/
       ↓
/opsx-archive     归档变更，移入 archive/
```

---

## 五个命令详解

### 1. `/opsx-explore` — 探索模式

**什么时候用**

- 还不确定要做什么
- 想先了解现有代码
- 写规格前讨论几种方案
- **明确：只讨论，不要写代码**

**怎么输入**

```
/opsx-explore 梳理知识库服务的 RAG 查询完整链路
```

```
/opsx-explore 我想给报表 Agent 加导出 Excel，但不确定改哪个模块
```

**AI 会做什么**

- 搜索、阅读源码
- 画架构图
- 分析利弊
- **不会改业务代码**

**示例对话**

```
你：/opsx-explore 知识库服务里文档上传后是怎么索引的？

AI：上传 → 对象存储 → 异步分块 → Embedding → 向量库 ...

你：好，帮我 propose
你：/opsx-propose 为知识库服务添加 PDF 分页索引策略
```

---

### 2. `/opsx-propose` — 提出变更

**什么时候用**

- 已经知道要做什么
- 需要生成规格文档，而不是直接写代码

**怎么输入**

```
/opsx-propose 为 QueryController 增加按 metadata 过滤的 RAG 查询
```

**AI 会做什么**

1. `openspec new change "<名>"`
2. 在 `openspec/changes/<名>/` 创建目录
3. 依次生成 `proposal.md` → `specs/` → `design.md` → `tasks.md`
4. 完成后提示可以 `/opsx-apply`

**你要做什么**

1. 打开 `proposal.md` 看「为什么做」
2. 看 `design.md` 方案是否合理
3. 看 `tasks.md` 是否可执行

不满意直接说：

```
proposal 里把范围缩小，只改 knowledge-base 模块，不要动 demo-tutorial 模块
```

---

### 3. `/opsx-apply` — 执行实现

**什么时候用**

- propose 完成且审阅通过
- `tasks.md` 已生成

**怎么输入**

```
/opsx-apply
```

指定变更名（多个变更时）：

```
/opsx-apply add-metadata-filter
```

**AI 会做什么**

1. 读 proposal、design、specs、tasks
2. 读 `openspec/config.yaml`
3. 按 tasks 逐项改代码
4. 完成一项：`[ ]` → `[x]`

**你要做什么**

- 确认改在**正确的模块/服务**（生产服务 vs 教程 Demo 要分清）
- 本地验证：`cd <目标模块> && mvn compile`（或项目对应的构建命令）

**若 tasks 已全部 `[x]`**

AI 会提示 **Implementation Complete**，直接 `/opsx-archive`，不会重复劳动。

---

### 4. `/opsx-sync` — 同步规格

**什么时候用**

- 变更里写了 `specs/` 增量
- 想合并到 `openspec/specs/` 正式库
- 通常在 archive 之前

**怎么输入**

```
/opsx-sync add-metadata-filter
```

**做什么**

```
openspec/changes/xxx/specs/.../spec.md  →  openspec/specs/.../spec.md
         （变更草稿）                              （正式规格）
```

---

### 5. `/opsx-archive` — 归档

**什么时候用**

- 功能或文档变更完成
- tasks 全部勾选
- 想「结案」

**怎么输入**

```
/opsx-archive add-metadata-filter
```

**AI 会做什么**

1. 检查 artifacts 和 tasks
2. 若 specs 未 sync，询问是否 sync
3. 移动到 `openspec/changes/archive/YYYY-MM-DD-<名>/`
4. `openspec list` 不再显示该变更

**归档后保留**

- `openspec/config.yaml`、业务代码、`docs/` 等改动仍在原处
- 历史记录在 `archive/` 目录

---

## 真实示例：完整走一遍

### 场景：给知识库服务增加「按文档类型过滤查询」

**Step 1 — 探索（可选）**

```
/opsx-explore 查询时 metadata 存在哪、怎么过滤
```

**Step 2 — 提出变更**

```
/opsx-propose 为知识库服务增加按文档类型 metadata 过滤 RAG 查询
```

**Step 3 — 审阅**

打开 `openspec/changes/add-doc-type-filter/` 下的 proposal、design、tasks。

**Step 4 — 实现**

```
/opsx-apply add-doc-type-filter
```

**Step 5 — 验证**

```bash
cd <目标模块目录>
mvn spring-boot:run
```

**Step 6 — 同步 + 归档**

```
/opsx-sync add-doc-type-filter
/opsx-archive add-doc-type-filter
```

---

## 完整闭环示例

一次标准的 OpenSpec 全流程如下（变更名仅为示例）：

| 步骤 | 命令 | 说明 |
|------|------|------|
| 提出 | `/opsx-propose 为 openspec/config.yaml 补充项目背景说明` | 创建变更目录与规格文档 |
| 实现 | `/opsx-apply` | 按 tasks.md 修改文件 |
| 同步 | `/opsx-sync <变更名>` | 合并 specs 到 `openspec/specs/` |
| 归档 | `/opsx-archive <变更名>` | 移入 `changes/archive/` |

归档后可在以下位置查看历史：

```
openspec/changes/archive/YYYY-MM-DD-<change-name>/
├── proposal.md
├── design.md
├── tasks.md
└── specs/
```

---

## 终端 CLI 命令

Slash 命令背后 AI 会执行这些 CLI，你也可以手动查状态：

```bash
cd <项目根目录>

openspec list                              # 进行中的变更
openspec status --change "变更名"           # 文档与任务进度
openspec instructions apply --change "变更名" --json   # apply 详情
openspec update                            # 升级后刷新 Cursor 集成
```

---

## 常见问题

### 输入 `/` 看不到 opsx 命令？

1. 确认打开项目根目录  
2. 完全重启 Cursor  
3. 执行 `openspec update`

### AI 改错模块了？

propose 时明确范围，并确保 `openspec/config.yaml` 写清模块边界：

```
只改 knowledge-base 模块，不要动 demo-tutorial 模块
```

### propose 和 apply 能一句搞定吗？

可以但不推荐。标准流程：**propose → 审阅 → apply**。

### OpenSpec 和 Spec-Kit 怎么选？

| 情况 | 推荐 |
|------|------|
| 小功能、改现有代码 | OpenSpec |
| 大模块从零、完整 PRD | Spec-Kit（`/speckit-specify` 等） |

### config.yaml 有什么用？

AI 每次 propose/apply 都会读。应包含：模块清单、端口冲突、框架选型、编码约定。新增模块后记得更新。

### apply 显示 all_done 但没写代码？

说明 tasks 在上一步（often propose 阶段）已完成，直接进入 archive 即可。

---

## 速查表

| 命令 | 一句话 | 会写业务代码吗 |
|------|--------|----------------|
| `/opsx-explore` | 先聊清楚、看代码 | ❌ |
| `/opsx-propose` | 生成规格文档 | ❌（只写 md） |
| `/opsx-apply` | 按 tasks 写代码 | ✅ |
| `/opsx-sync` | 规格草稿 → 正式库 | ❌ |
| `/opsx-archive` | 结案归档 | ❌ |

---

## 相关文档

- [安装与 OpenSpec/Spec-Kit 对比](./SPEC-DRIVEN-DEVELOPMENT.md)
- [OpenSpec 官方文档](https://github.com/Fission-AI/OpenSpec/tree/main/docs)

---

| 日期 | 说明 |
|------|------|
| 2026-06-11 | 初版：五命令详解、通用示例、完整生命周期 |
| 2026-06-11 | 脱敏：移除具体项目/模块名，仅保留通用表述 |
