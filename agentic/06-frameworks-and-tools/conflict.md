# Frameworks and Tools Conflict（冲突与待核验问题）

本文件记录 `06-frameworks-and-tools` 范围内的口径冲突、证据不足和待核验问题。冲突解决后，应同步更新相关主干文档，并移除或归档对应条目。

---

## 一、已解决的结构冲突

### 1.1 `06` 是否应按能力层重建目录

**冲突描述**：早期调研稿建议在 `06-frameworks-and-tools/` 下建立 `protocols/`、`capabilities/`、`coding-agents/` 等能力型目录。但该方案会与 `02-single-agent/`、`03-multi-agent/`、`05-environments/`、`07-evaluation/` 形成结构性重叠。

**当前结论**：已改为对象视角组织：

- `01-frameworks/`
- `02-coding-agents-and-tools/`
- `03-project-studies/`
- `04-skill-and-tool-systems/`
- `05-comparisons/`

**处理结果**：相关结论已写入 `README.md` 和 `overview.md`。旧调研稿 `temp/web1.md`、`temp/web2.md`、`temp/web3.md` 已删除。

---

## 二、待核验问题

### 2.1 框架成熟度与版本状态

| 对象 | 待核验点 | 建议来源 |
|------|----------|----------|
| `Microsoft Agent Framework` | GA / RC / LTS 状态、AutoGen 与 Semantic Kernel 的整合边界 | Microsoft 官方文档与发布公告 |
| `Semantic Kernel` | Agent Framework 相关功能的稳定状态与长期支持范围 | Microsoft Learn / GitHub release |
| `OpenAI Agents SDK / Responses API` | “七层架构”应继续视为外部归纳；当前主线只可稳定写到 handoffs、results/state、sessions、tracing、sandbox、tools/MCP 等公开能力面，不宜上推成统一 runtime 契约 | OpenAI 官方 docs / SDK 文档 / 对象目录正文 |
| `OpenAI Agents SDK / Responses API` | `session` 与 `conversation_id` / `previous_response_id` / `auto_previous_response_id` 的互斥边界已较清楚，但 `RunState`、server-managed continuation、sandbox `session_state` / `snapshot` 这几层恢复面如何组合仍待源码级核验 | `running_agents`、`sessions`、`results`、`human_in_the_loop`、`ref/sandbox`、`openai-agents-sdk/notes/evidence.md` |
| `OpenAI Agents SDK / Responses API` | `SandboxRunConfig.session_state`、`snapshot`、sandbox capabilities 的字段存在已可确认，但 snapshot 是否完整覆盖 workspace changes、是否跨 sandbox client / hosted provider 一致，公开资料仍不足 | `v0.14.0` release notes、`ref/sandbox`、sandbox client 源码、`openai-agents-sdk/notes/evidence.md` |
| `OpenAI Agents SDK / Responses API` | local MCP / hosted MCP integration 已可确认存在，但 cancellation、connect/cleanup、resume/reconnect 后连接状态、hosted MCP 恢复语义仍缺统一公开说明 | `tools`、`running_agents`、后续 `MCP servers / manager` reference 与源码、`openai-agents-sdk/notes/evidence.md` |
| `OpenAI Agents SDK / Responses API` | error handlers、`ToolTimeoutError`、partial output、max turns 与 retry/cancel/recovery 之间的边界未形成统一契约；当前只宜保留“存在结构化错误类型与有限 terminal fallback”这一层 | `ref/run`、`running_agents`、异常/runner 源码、`openai-agents-sdk/notes/evidence.md` |
| `CrewAI` | 企业使用比例、运行规模等第三方数据是否有官方证据 | CrewAI 官方博客、客户案例、release notes |
| `DSPy` | 是否应作为 Agent framework 主干对象，还是只作为 prompt / program optimization 对比对象 | Stanford DSPy 官方文档与论文 |

### 2.2 候选项目与热度数据

| 对象 | 待核验点 | 建议来源 |
|------|----------|----------|
| `OpenClaw` | GitHub stars、开源基金会化、创始人去向、安全审计论文等信息 | 官方 GitHub、官方文档、论文原文 |
| `Hermes Agent` | GitHub stars、推理榜 token 数据、与 OpenClaw 的对比结论 | 官方仓库、OpenRouter 官方数据 |
| `Ruflo` | 是否为正式项目、与 Claude Code 的关系、生态规模 | 官方仓库与文档 |
| `Agency` | 项目定位、成熟度、是否适合作为框架或项目案例 | 官方仓库与文档 |

### 2.3 前沿观察对象

| 对象 | 待核验点 | 建议来源 |
|------|----------|----------|
| `SAGE` | 是否开源、是否有后续引用与基准结果 | arXiv 论文、GitHub 链接 |
| `ATOM` | 是否开源、预算可控协作是否被后续系统采用 | arXiv 论文、作者主页 |
| `MANGO` | 代码发布状态与实际实验复现情况 | arXiv 论文、GitHub 链接 |
| `AgentBay` | 是否公开 SDK、ASP 协议是否被采用 | 论文与官方项目页 |
| `ceLLMate` | 浏览器扩展兼容性、真实浏览器环境集成状态 | 论文与项目仓库 |
| `AG-UI` | 是否有稳定规范、roadmap、多框架客户端生态 | 官方协议文档 |

---

## 三、维护规则

- 只有当某项存在事实冲突、证据不足或口径不一致时，才保留在本文件。
- 如果待核验问题已经转入 `backlog.md` 且不影响当前主干结论，可以从本文件移除。
- 如果核验后形成稳定结论，应同步更新 `overview.md`、相关子目录 README 或具体对象文档。

---

*最后更新: 2026-05-31*
