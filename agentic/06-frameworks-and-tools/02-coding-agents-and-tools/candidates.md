# Coding Agents and Tools Candidates

> 本文件记录 `coding-agents-and-tools` 目录下一步值得研究的候选对象。它不是推荐榜单，也不把 star 数、社区评价或产品能力写成已验证事实；未核验信息仅作为选题线索。

## 使用边界

- **对象范围**：coding agent、IDE/CLI 编程助手、自主软件工程系统、PR/Issue 自动化工具，以及与 coding agent 直接相关的 DevOps agent。
- **记录目的**：帮助决定后续应研究哪些仓库、产品、官方文档、benchmark 或社区材料。
- **证据状态**：当前条目主要来自外部综合线索，默认 `Unverified`；进入正文或案例研究前必须核验官方仓库、文档、论文、release notes 或 benchmark 结果。
- **落位原则**：对象本身足够重要时进入项目/产品案例目录；只支撑某个机制判断时，作为专题文档的证据来源；仅代表长期方向时保留在本文件。

## P0：优先核验候选

### OpenHands

- Type: 开源自主软件工程系统
- Status: 待核验 / 已有初步案例材料
- Why relevant: 已在本仓库中被用于观察 confirmation policy、hooks、人工暂停与审查等 harness 机制，适合继续作为 coding agent runtime / control plane 案例。
- Evidence needed: 官方仓库、文档、关键配置、CLI 行为、hooks/confirmation policy 的版本基线。
- Placement: `06-frameworks-and-tools/03-project-studies/openhands/`

### Claude Code

- Type: 商业 CLI coding agent
- Status: 待核验 / 已有初步产品材料
- Why relevant: 适合作为 plan mode、permission mode、交互式 agent workflow 与 human-in-the-loop gate 的产品样本。
- Evidence needed: 官方文档、CLI 行为说明、permission mode / plan mode 的当前语义、最佳实践文档。
- Placement: `06-frameworks-and-tools/02-coding-agents-and-tools/claude-code/`

### OpenCode

- Type: 开源 CLI coding assistant / coding agent
- Status: 进行中；源码已下载到 `D:\github\opencode`
- Source: <https://github.com/anomalyco/opencode>
- Why relevant: 接下来重点研究对象。适合核验 CLI coding workflow、模型/工具接入、权限边界、上下文管理、任务执行循环，以及它与 Claude Code / Codex CLI / Aider 等 CLI agent 的差异。
- Evidence needed: 官方 README、文档、源码结构、CLI 入口、工具调用机制、配置与权限模型、任务状态/会话持久化机制。
- Placement: `06-frameworks-and-tools/02-coding-agents-and-tools/opencode/`

### Codex CLI

- Type: 开源 CLI coding agent
- Status: 待看
- Why relevant: OpenAI 维护的终端 coding agent，可用于比较任务状态、工具调用、sandbox/approval、patch workflow 与人类接管机制。
- Evidence needed: 官方仓库、README、CLI 文档、sandbox/approval 设计、任务执行与验证工作流。
- Placement: 待定；可作为独立案例或与 Claude Code / OpenHands / OpenCode 做横向机制比较。

### Aider

- Type: 命令行 AI 结对编程工具
- Status: 待看
- Why relevant: 早期且长期活跃的 Git/GitHub 集成式 AI 编程工具，适合研究 repo map、patch editing、commit workflow 与轻量人机协作边界。
- Evidence needed: 官方仓库、文档、repo map 机制、编辑策略、Git 工作流说明。
- Placement: 待定；适合进入 CLI coding assistant 案例池。

### SWE-agent

- Type: benchmark-oriented autonomous software engineering agent
- Status: 待看
- Why relevant: 与 SWE-bench 关系紧密，适合作为自动修复真实 issue、agent-computer interface、evaluation workflow 的研究对象。
- Evidence needed: 论文、官方仓库、SWE-bench 配置、agent-computer interface 说明、实验设置。
- Placement: 待定；可放入 benchmark / evaluation 相关专题或项目案例。

## P1：重要但需先确认范围

### Cline

- Type: IDE coding agent / VS Code extension
- Status: 待看
- Why relevant: 代表 IDE 内 agentic coding workflow，适合研究文件编辑、工具调用、用户确认、上下文选择和 MCP 生态。
- Evidence needed: 官方仓库、扩展文档、MCP / tool-use 说明、权限与确认机制。
- Placement: 待定；可作为 IDE agent 案例。

### Continue

- Type: 开源 IDE AI assistant
- Status: 待看
- Why relevant: 适合研究开源 IDE assistant 的模型接入、上下文管理、代码库检索和团队可配置能力。
- Evidence needed: 官方仓库、文档、配置模型、上下文 provider、检索机制。
- Placement: 待定；可作为 IDE assistant / context management 案例。

### Gemini CLI

- Type: CLI coding agent / AI terminal assistant
- Status: 待核验
- Why relevant: 如果其官方定位和功能边界成立，可作为大型模型厂商 CLI coding workflow 的横向对照。
- Evidence needed: 官方仓库或文档、功能边界、工具调用、权限/沙箱/验证机制；原始 star 数和语言信息不保留为事实。
- Placement: 待定；先核验是否适合作为 coding agent 对象。

### Goose

- Type: 开源开发者 agent / CLI assistant 候选
- Status: 待核验
- Why relevant: 若其 Rust 实现、工具接入和开发者自动化能力成立，可作为高性能本地/CLI agent 对照。
- Evidence needed: 官方仓库、文档、模型/工具集成方式、安全与权限边界。
- Placement: 待定。

## P2：专题或长期观察候选

### PR-Agent

- Type: PR 自动化审查工具
- Status: 待看
- Why relevant: 与代码审查、PR 摘要、review action item 和 CI 集成有关，适合作为软件交付环节 agent 化的专项样本。
- Evidence needed: 官方仓库、支持平台、review 能力边界、配置方式、实际输出样例。
- Placement: 可进入 code review / PR automation 专题。

### Pullfrog

- Type: PR / Issue 自动化或 agent 编排候选
- Status: 待核验
- Why relevant: 原始材料称其关注代码审查、Issue 分类等生命周期管理，但项目指向和能力边界尚不明确。
- Evidence needed: 准确仓库或官网、功能说明、是否仍活跃、与 coding agent 的关系。
- Placement: 待定；先保留为低优先级线索。

### OpenCode MCP Server

- Type: MCP server / coding tool integration 候选
- Status: 待核验
- Why relevant: 如果确实提供标准化 MCP 接入，可用于研究 coding agent 能力如何通过协议外部化。
- Evidence needed: 准确仓库 URL、MCP 工具定义、与 OpenCode 的关系、调用边界。
- Placement: 可进入 MCP / tool integration 相关专题。

### GitHub Copilot Agent Mode

- Type: 商业 IDE / 平台级 coding agent
- Status: 待看
- Why relevant: 代表平台级 issue/PR 修复、IDE 内 agent mode 和 GitHub 工作流集成，适合做商业产品对照。
- Evidence needed: 官方文档、agent mode 功能边界、GitHub issue/PR 工作流、权限与验证机制。
- Placement: 待定；更适合产品机制比较，不宜与开源仓库 star 排名混列。

### Cursor

- Type: 商业 AI IDE
- Status: 待看
- Why relevant: 代表 IDE-native agentic coding 体验，可用于研究代码库上下文、composer/agent workflow、人类确认与编辑接管。
- Evidence needed: 官方文档、公开功能说明、agent workflow、隐私/权限/上下文机制。
- Placement: 待定；可作为商业 AI IDE 样本。

### AutoDev

- Type: 多平台 AI 编程工具候选
- Status: 待核验
- Why relevant: 原始材料称其覆盖需求、开发、测试、运维，但对象边界和当前活跃度需确认。
- Evidence needed: 官方仓库或官网、产品边界、支持平台、真实 workflow 示例。
- Placement: 待定。

### AutoSoftware

- Type: 自主软件工程平台候选
- Status: 待核验
- Why relevant: 原始材料称其从单一命令规划、编码并提交 PR；如果属实，可作为 end-to-end autonomous SWE 平台样本。
- Evidence needed: 准确项目来源、官方说明、演示或代码、是否具备可复现实例。
- Placement: 待定；暂不作为事实引用。

## 暂不纳入主候选池

### Hermes Agent

- Reason: 原始材料将其描述为运维 agent，但缺少准确项目指向；与 coding agent 主线关系不清。
- Action: 只有在能确认其与软件开发、代码变更、PR/Issue 或工程自动化直接相关时，才重新纳入。

### Nimbus

- Reason: 原始材料描述为 DevOps / cloud CLI agent，可能更适合 DevOps agent 或平台工程专题，而不是 coding agent 主候选池。
- Action: 先核验具体项目与能力边界，再决定是否移动到更合适目录。

### DevOps AI Toolkit (`@vfarcic/dot-ai`)

- Reason: 主要指向 Kubernetes / 平台工程自动化，和 coding agent 目录的直接关系较弱。
- Action: 如后续研究 DevOps agent，可作为候选对象；当前只保留为边界线索。

### `500-AI-Agents-Projects`

- Reason: 这是资源合集，不是单一研究对象；更适合作为发现候选的上游来源，而不是本目录案例对象。
- Action: 若使用其中条目，需要逐项回到原始项目核验。

## Evidence

- Status: `Unverified`
- Sources:
  - 原 `candidates.md` 中的外部综合候选清单。
  - 本目录已有的 Claude Code / OpenHands 相关研究材料。
  - <https://github.com/anomalyco/opencode>；本地源码路径 `D:\github\opencode`。
- Trace: 本文件从推荐榜单改造为 L5 候选研究对象队列；保留原始对象作为选题线索，但移除未核验 star 数、社区评价和排名语气，把候选对象按优先级、对象类型、研究理由、证据需求与落位重新组织。2026-06-09 用户指定接下来重点研究 `anomalyco/opencode`，因此将 OpenCode 移入 P0 并标记为进行中。
- Needs:
  - 为 OpenCode 建立案例研究入口，并从官方 README、文档和源码结构开始核验。
  - 为其他 P0 / P1 候选补准确官方 URL、版本时间点和基础证据。
  - 判断哪些对象应进入独立案例目录，哪些只作为机制专题证据。
  - 对 DevOps agent 与 coding agent 的边界建立更明确的目录落位判断。
