---
title: "obsidian-wiki 研究"
category: source
source: Ar9av/obsidian-wiki
url: https://github.com/Ar9av/obsidian-wiki
tags:
  - llm-wiki
  - skill-framework
  - obsidian
  - agent
  - markdown
  - karpathy-pattern
created: 2026-05-03
updated: 2026-05-03
status: complete
---

# obsidian-wiki 研究

> 一个基于 Karpathy LLM Wiki pattern 的 Skill-based Agent 框架，用于构建和维护 Obsidian 知识库。

## 基本信息

| 属性 | 值 |
|------|-----|
| **作者** | Ar9av |
| **Stars** | 906 |
| **Forks** | 113 |
| **主要语言** | Python / Markdown（无代码依赖） |
| **许可证** | MIT |
| **安装** | `npx skills add Ar9av/obsidian-wiki` |

**一句话定位**：No scripts or dependencies — the agent IS the runtime. Skill-based 框架。

---

## 核心架构

```
Agent ──► 读取 .skills/<name>/SKILL.md ──► 执行指令 ──► 操作 Vault
              │                              │
              ▼                              ▼
        20+ 内置 Skill              读写 Markdown
        15+ Agent 兼容              更新 .manifest.json
        零依赖                      维护 index.md / log.md
```

### 核心理念

1. **Agent IS the runtime** — 没有可执行脚本，所有逻辑是 Markdown 指令
2. **Skill = Markdown 工作流** — 每个 Skill 定义触发词 + 执行步骤
3. **Vault 是产物，Agent 是维护者，Obsidian 是查看器**

---

## 安装与配置

### 快速安装

```bash
npx skills add Ar9av/obsidian-wiki
```

安装后打开 Agent 说 **"set up my wiki"** 即可初始化。

### 配置

配置读取优先级（第一个找到的生效）：

1. `~/.obsidian-wiki/config` — 全局配置
2. `.env` — 本地回退

唯一必需配置：`OBSIDIAN_VAULT_PATH`（wiki 存放路径）

可选配置：

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `OBSIDIAN_SOURCES_DIR` | 待摄入文档目录（逗号分隔） | *(空)* |
| `OBSIDIAN_CATEGORIES` | wiki 页面分类 | `concepts,entities,skills,references,synthesis,journal` |
| `OBSIDIAN_MAX_PAGES_PER_INGEST` | 每次摄入最大更新页数 | `15` |
| `OBSIDIAN_LINK_FORMAT` | 链接格式 | `wikilink`（可选 `markdown`） |
| `LINT_SCHEDULE` | 健康检查频率 | `weekly` |

---

## Vault 预设结构

```
$OBSIDIAN_VAULT_PATH/
├── index.md                # 主索引 — 列出所有页面，始终保持最新
├── log.md                  # 按时间顺序的活动日志（摄入、更新、lint）
├── hot.md                  # 会话热缓存 — ~500字近期活动语义快照
├── .manifest.json          # 追踪每个摄入来源：路径、时间戳、产物页面
├── _meta/
│   ├── taxonomy.md         # 受控标签词汇表
│   └── *.base              # Obsidian Bases 仪表盘定义
├── _insights.md            # 图谱分析输出（hubs, bridges, dead ends）
├── _raw/                   # 暂存区 — 放入粗略笔记，下次摄入提升
├── concepts/               # 抽象概念、模式、心智模型
├── entities/               # 具体事物 — 人、工具、库、公司
├── skills/                 # 操作指南、技术、流程
├── references/             # 事实性查阅 — 规范、API、配置
├── synthesis/              # 跨领域分析，连接多个概念
├── journal/                # 时间绑定条目 — 日志、会话笔记
└── projects/
    └── <project-name>.md  # 每个项目一页，通过 wiki-update 同步
```

### 每页必需 frontmatter

```yaml
---
title: "..."
category: "..."
tags: ["..."]
sources: ["..."]
created: "YYYY-MM-DD"
updated: "YYYY-MM-DD"
---
```

---

## Skill 体系

20+ 内置 Skill，位于 `.skills/<name>/SKILL.md`：

| Skill | 触发词 | 用途 |
|-------|--------|------|
| `wiki-setup` | "set up my wiki" / "initialize" | 初始化 vault 结构、创建索引/日志 |
| `wiki-ingest` | "ingest" / "add this to the wiki" | 将源文档提炼为 wiki 页面 |
| `ingest-url` | "/ingest-url <url>" / "add this URL" | 摄入单个 URL |
| `wiki-history-ingest` | "/wiki-history-ingest claude/codex" | 统一历史摄入路由 |
| `claude-history-ingest` | "import my Claude history" | 挖掘 `~/.claude` 对话 |
| `codex-history-ingest` | "import my Codex history" | 挖掘 `~/.codex` 会话 |
| `hermes-history-ingest` | "import my Hermes history" | 挖掘 `~/.hermes` |
| `openclaw-history-ingest` | "import my OpenClaw history" | 挖掘 `~/.openclaw` |
| `data-ingest` | "process this export" / logs | 处理任意文本数据 |
| `wiki-status` | "what's the status" / "show delta" | 审计摄入状态 |
| `wiki-query` | "what do I know about X" | 从 wiki 回答问题 |
| `wiki-lint` | "audit" / "find broken links" | 查找孤儿、断链、陈旧内容 |
| `wiki-rebuild` | "rebuild" / "archive" / "restore" | 归档/重建/恢复 |
| `cross-linker` | "link my pages" / "cross-reference" | 自动发现并插入缺失 wikilinks |
| `tag-taxonomy` | "fix my tags" / "normalize tags" | 规范化标签词汇 |
| `wiki-update` | "update wiki" / "sync to wiki" | 将当前项目知识同步到 vault |
| `wiki-export` | "export wiki" / "graphml" / "neo4j" | 导出图谱数据 |
| `graph-colorize` | "color my graph" / "color by tag" | 按标签/分类/可见性着色 |
| `wiki-capture` | "/wiki-capture" / "save this" | 保存当前对话到 wiki |
| `wiki-research` | "/wiki-research [topic]" | 研究主题并写入 wiki |
| `wiki-dashboard` | "create a dashboard" / "dynamic view" | 创建 vault 仪表盘 |
| `wiki-synthesize` | "/wiki-synthesize" / "find connections" | 发现概念间的关联模式 |
| `skill-creator` | "create a new skill" | 创建新 skill 扩展框架 |

### Skill 分发

- 平台：[skills.sh/ar9av/obsidian-wiki](https://skills.sh/ar9av/obsidian-wiki)
- 安装：`npx skills add Ar9av/obsidian-wiki`
- 扩展：使用 `skill-creator` skill 创建自定义 workflow

---

## Agent 兼容性

支持 15+ 种 AI Agent：

Claude Code、Cursor、Windsurf、Gemini CLI、Google Antigravity、Codex、Hermes、OpenClaw、OpenCode、Aider、Factory Droid、Trae / Trae CN、Kiro、GitHub Copilot（CLI + VS Code Chat）

安装方式：`setup.sh` 为每个 Agent 创建：
- 项目本地 skill 符号链接（`.claude/skills/`、`.cursor/skills/` 等）
- 全局符号链接（`~/.claude/skills/`、`~/.gemini/skills/` 等）
- 常驻规则文件（`CLAUDE.md`、`GEMINI.md`、`AGENTS.md`、`.cursor/rules/` 等）

---

## 追踪与 Delta 机制

`.manifest.json` 追踪所有摄入来源，支持：

- **状态视图**："已摄入什么？新增了什么？什么变了？"
- **Delta 摄入**：只处理新增/修改来源，跳过已有内容
- **溯源**：哪个来源产出了哪个 wiki 页面
- **陈旧检测**：来源已变但 wiki 页面未更新

### 典型工作流

```
"What's the status?"     → wiki-status 计算 delta
"Ingest the new stuff"   → wiki-ingest 只处理 delta（append 模式）
"What's the status now?" → wiki-status 确认全部最新
```

### 重建与恢复

```
"Archive and rebuild"    → wiki-rebuild 归档当前 wiki 到 _archives/，清空，准备重新摄入
"Restore the old one"    → wiki-rebuild 从之前归档恢复
```

---

## 可选：QMD 语义搜索

- 基于 QMD（Query-Model-Document）的语义搜索
- 扫描标题、标签、`summary:` frontmatter（低成本首轮）
- 仅当索引轮无法回答时才打开页面正文
- 回答包含 `[[wikilink]]` 引用

---

## 图谱可视化

Obsidian 原生图谱视图 + 额外分析：

- `_insights.md`：图谱分析输出
  - **Hubs**：高度连接的节点
  - **Bridges**：连接不同社区的关键节点
  - **Dead ends**：无出链的孤立节点
- `graph-colorize` skill：按 tag / category / visibility 着色

---

## Visibility 标签（可选）

页面可携带 `visibility/` 标签标记预期传播范围：

| 标签 | 含义 |
|------|------|
| *(无标签)* | 等同于 `visibility/public` — 所有模式可见 |
| `visibility/public` | 显式公开 — 所有模式可见 |
| `visibility/internal` | 团队内部 — 过滤模式下排除 |
| `visibility/pii` | 敏感数据 — 过滤模式下排除 |

- **系统标签**：不计入 5 标签限制，与领域标签分开列出
- **过滤模式**：通过查询中的 "public only" 等短语触发，默认显示全部

---

## 跨项目使用

### wiki-update（写入 wiki）

从任何项目目录运行：
1. 读取 `~/.obsidian-wiki/config` 获取 `OBSIDIAN_VAULT_PATH`
2. 扫描当前项目：README、源码结构、git log、包元数据
3. 提炼值得记录的内容（架构决策、模式、权衡 — 非代码清单）
4. 写入 `$VAULT/projects/<project-name>.md`，交叉链接 concept/entity 页面
5. 更新 `.manifest.json`、`index.md`、`log.md`

重复运行时检查 `.manifest.json` 中的 `last_commit_synced`，只处理 delta：`git log <last_commit>..HEAD`

### wiki-query（读取 wiki）

1. 读取配置获取 vault 路径
2. 先扫描标题、标签、`summary:` frontmatter（低成本）
3. 仅当索引轮无法回答时才打开页面正文
4. 返回综合回答，附带 `[[wikilink]]` 引用

---

## 核心原则

1. **Compile, don't retrieve.** Wiki 是预编译知识。更新现有页面 — 不追加或重复。
2. **Track everything.** 摄入后更新 `.manifest.json`，任何写入后更新 `index.md`、`log.md`、`hot.md`。
3. **Connect with `[[wikilinks]]`.** 每页都应链接到相关页面。这是知识图谱，不是文件夹。
4. **Frontmatter is required.** 每页需要：`title`, `category`, `tags`, `sources`, `created`, `updated`。
5. **Single source of truth.** Visibility 标签只影响展示方式，不复制或分离内容。
6. **Keep context warm.** `hot.md` 是 ~500 字近期活动语义快照。每次写入 skill 更新它，使下次会话无需爬取整个 vault 即可接续。

---

## 在 Karpathy 模式上的增强

| 维度 | Karpathy 模式 | obsidian-wiki |
|------|--------------|---------------|
| **实现方式** | 描述性 pattern | Skill-based 框架 |
| **运行时** | 通用 LLM | Agent 自身执行 Markdown 指令 |
| **摄入来源** | 手动整理 | 20+ skills 覆盖 URL/历史/导出/数据 |
| **增量更新** | 手动跟踪 | `.manifest.json` delta 追踪 |
| **质量控制** | 人工审核 | `wiki-lint` skill |
| **多 Agent** | 单一 Agent | 15+ Agent 兼容 |
| **图谱分析** | 基础链接 | Hubs/bridges/dead ends 分析 |
| **可见性控制** | 无 | `visibility/` 标签 |
| **热缓存** | 无 | `hot.md` 会话连续 |
| **Skill 扩展** | 无 | `skill-creator` 自定义 workflow |

---

## 适用场景

- **个人知识管理**：Agent 作为持续维护者，日常更新个人知识库
- **多 Agent 用户**：同时使用 Claude/Cursor/Codex 等多种工具，统一知识库
- **零依赖偏好**：不想安装 Node.js/Python 依赖，Agent 直接操作文件
- **项目知识同步**：使用 `wiki-update` 将各项目知识自动汇入统一 vault
- **Obsidian 用户**：已有 Obsidian 使用习惯，希望 AI 辅助维护
- **Skill 生态探索**：通过 `skill-creator` 实验自定义 AI workflow

---

## 相关链接

- [GitHub 仓库](https://github.com/Ar9av/obsidian-wiki)
- [Skill 列表](https://skills.sh/ar9av/obsidian-wiki)
- [Karpathy's LLM Wiki pattern](https://github.com/karpathy/llm-wiki)
- [skills.sh](https://skills.sh) — Skill 分发平台

---

*研究基于 2026-05-03 仓库快照，main 分支最新版本。*
