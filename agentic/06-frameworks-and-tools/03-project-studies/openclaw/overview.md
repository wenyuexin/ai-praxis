## OpenClaw：自托管 AI 自动化执行引擎

### 一、项目概述与定位

**正式名称**：OpenClaw（曾用名 Clawdbot / Moltbot）

OpenClaw 是一款遵循 MIT 许可的开源自托管 AI 代理引擎，通过 TypeScript 编写的 CLI 进程在本地设备上常驻运行，接入 WhatsApp、Telegram、Discord、iMessage、飞书、钉钉、Slack 等超过 20 个消息渠道，将自然语言指令转化为可执行任务——读取本地文件、管理日历、监控 GitHub 仓库、执行 Shell 命令等跨平台操作。

> 项目的核心差异化在于：它不只是对话式问答机器人，而是能**真正“动手执行任务”** 的自主代理。

项目由奥地利开发者 Peter Steinberger 于 2025 年底发起，早期采用“龙虾（lobster）”图标作为视觉标识，社区也常以“龙虾AI”代指该框架。2026 年 1 月首次开源发布后迅速走红，首周即获超 2 万星标，八周内突破 10 万星标，截至 2026 年 5 月已积累超过 200,000 GitHub Stars。项目创始人随后被 OpenAI 招募，项目代码库已启动向独立基金会转型。

### 二、与 Clawdbot / Moltbot 的关系

OpenClaw 曾使用 Clawdbot、Moltbot 等名称。项目的开源身份追溯至 Clawdbot，2026 年初因商标争议经历更名风波——从 Clawdbot 更名为 Moltbot，再正式定名 OpenClaw。“Open”指代免费开源属性，“Claw”突出其高效抓取和执行任务的能力。学术研究和社区讨论中，Clawdbot、Moltbot、OpenClaw 在实际应用场景中可能指代同一个核心技术栈。

### 三、核心能力

#### 3.1 真正的“做而不只是说”

OpenClaw 与传统聊天助手的本质区别在于执行能力。使用者通过日常使用的即时通讯应用向 Agent 发送指令，后台运行的 OpenClaw 实例通过大模型解析意图并执行具体操作——订航班、清理收件箱、触发部署、读写文件，全部通过对话完成。特斯拉前 AI 主管 Andrej Karpathy 曾公开评价该项目“重新定义了 AI 代理与物理系统的交互边界”。

#### 3.2 本地优先与私有化部署

OpenClaw 运行在用户自有硬件（个人电脑、VPS 或树莓派）上，并非依赖云服务商托管。电子邮件内容、日历条目、文件结构等都保留在用户本地基础设施内，除非用户明确指示 Agent 向外部发送信息。所有配置、记忆状态和对话历史均以纯文本 Markdown 文件存储在标准文件夹中，用户可随时使用文本编辑器直接阅读和修改。

#### 3.3 模型无关性

用户可自行提供 LLM API 密钥（Claude、GPT、DeepSeek、Llama via Ollama 等），也可选择本地模型运行。通过适配器模式实现模型热插拔，使得 LLM 成为可更换的模块，而框架围绕模型构建的“脚手架”决定了 Agent 的最终能力边界。

**支持渠道**包括 WhatsApp、Telegram、Signal、Discord、Slack、iMessage、飞书、钉钉、QQ、企业微信和自定义 WebChat，一个 Agent 实例可同时接入多个渠道。

### 四、架构设计

OpenClaw 采用**Hub-and-Spoke 架构**，以 Gateway 作为中心枢纽连接输入端渠道与 Agent 运行时。其总体架构可拆解为以下核心层次：

#### 4.1 Gateway 网关层

Gateway 是 OpenClaw 的入口控制面，负责三件事：接收来自各消息渠道的消息、为每个会话维护工作队列、将消息路由至 Agent 执行运行时。Gateway 本身持有多渠道适配器（即 Channel Plugins），使一套 Agent 能力可同时服务多个平台。Gateway 还可托管单个默认 Agent 或多个独立 Agent 并排运行。

#### 4.2 Agent 运行时层

每个 Agent 是完整的“人格范围”（per-persona scope），拥有自己的工作区文件、认证配置、模型注册表和会话存储，独立存储路径为 `~/.openclaw/agents/<agentId>/`。此模式支持**多 Agent 隔离并行**，官方将 Main Agent 定位为“项目经理”，Architect Agent 负责架构设计，Writer Agent 专注执行落地，形成如专业团队的协作体系。

Agent 执行循环并非简单的一问一答，而是以 `while` 循环持续接收消息、调用 LLM、执行工具、处理反馈并迭代完成开放式复杂任务。

#### 4.3 Skills 技能层

OpenClaw 预置超过 200 个基础原子操作能力，涵盖文件管理、网络请求、系统监控等场景，遵循标准化 Skill 接口定义（包含名称、执行逻辑、输入验证等字段）。开发者可通过组合原子 Skill 构建复杂任务流程，所有 Skill 按 Agent 工作区加载，并由允许列表（allowlist）按需控制。项目同时支持**子 Agent 机制**——通过创建独立子 Agent 实现并行执行和上下文隔离，类似派出多个助手分头处理不同子任务。

#### 4.4 Memory 记忆层

OpenClaw 的四层记忆结构包含：短期记忆（会话内上下文变量）、长期记忆（用户偏好与历史操作记录持久化）、向量记忆（基于向量数据库的语义检索）、文档记忆（导入的外部文档片段）。记忆系统支持在配置中开启检索增强（RAG），可设置 `topK` 和 `scoreThreshold` 参数控制召回行为。

#### 4.5 AGENTS.md 结构化定义

AGENTS.md 是 OpenClaw 中无需编写代码即可完成 Agent 完整定义的声明式配置文件。其标准结构声明 Agent 的职责描述、能力目标、Skill 列表、工具调用权限、任务工作流以及输出格式约束，支持多 Agent 继承、能力隔离和权限约束。

### 五、功能与生态

#### 5.1 多 Agent 协作与编排

OpenClaw 支持在单 Gateway 内托管多个 Agent，每个 Agent 拥有独立的 workspace 目录和会话历史。通过 binding 配置，可将不同消息渠道（如 WhatsApp、Telegram 或 Slack 账号）映射至不同 Agent 进行处理。Agent 路由可实现跨渠道的消息分发、任务流转与结果汇总，构建起角色分工的协作体系。

#### 5.2 定时任务与后台常驻（Heartbeat）

OpenClaw 提供可选的心跳守护进程（heartbeat daemon），可在预设时间间隔自动唤醒 Agent，无需用户主动提问即可执行主动检测、定时报告、后台巡检等自动化流程。

#### 5.3 跨平台部署

支持 macOS / Windows / Linux / iOS / Android 全平台部署。提供轻量级（树莓派+64GB）、标准（Mac mini+M2）、企业级（X86 + NVIDIA A100）三套部署方案，通过单条 curl 命令完成自动化配置。

#### 5.4 MCP 协议支持

OpenClaw 具备对 Model Context Protocol（MCP）的原生集成能力，可作为 MCP Server 提供工具调用服务，或通过 MCP 协议向其他 Agent 暴露能力。社区已有插件通过 MCP 实现 OpenClaw 与 IDE 工具链的打通。

### 六、学术价值与风险认知

> **重要提示**：OpenClaw 作为一款赋予 LLM 自治系统执行能力的框架，**安全性设计高度依赖用户配置**，默认配置下存在被远程控制的风险。**在获得完整安全加固前，不建议在生产环境或高价值系统中直接运行 OpenClaw 默认配置。**

#### 6.1 学术背景与安全审计

OpenClaw 的系统架构和交互范式引发了学术界的广泛研究。2026 年 2 月，上海科技大学 ASPIRE 实验室联合上海人工智能实验室发布《基于轨迹的 Clawdbot（OpenClaw）安全审计》论文，指出在默认配置下攻击者最快 5 分钟即可完全接管运行 OpenClaw 的电脑。后续研究进一步构建了三维基准 CLAWSAFETY，涵盖了软件工程、财务、医疗、法律和运维等五个伤害领域，从模型、运行框架、信任通道与场景四个维度评估 Claw 类智能体的安全风险。

学术界已围绕 OpenClaw 的安全生态形成一系列系统化研究：
- 2026 年 3 月《SafeClaw-R》提出多 Agent 个人助手的安全策略；
- 2026 年 4 月系统性安全评估覆盖 OpenClaw 及其六个衍生变体（AutoClaw、QClaw、KimiClaw、MaxClaw、ArkClaw）；
- 2026 年 5 月《Sleeper Channels and Provenance Gates》识别出后台常驻 AI Agent 特有的 sleeper channel 攻击面。

#### 6.2 安全防护与加固

OpenClaw 安全架构在持续迭代中逐渐形成了多层防护体系——基于单用户信任边界的设计哲学，包含身份层（从配置可选到强制执行）、能力过滤、沙箱隔离、操作审批与审计等机制。

**安全加固方向**包括：
- **操作隔离**：启用沙箱模式可限制工具执行范围，敏感操作需要用户审批；
- **权限隔离**：Skill 执行环境分配独立的操作账户，支持操作级审批机制；
- **数据脱敏**：敏感信息在传输过程中自动加密处理。

企业在 AWS 上部署类 OpenClaw 智能体时，安全团队基于 MAESTRO 框架进行了 7 层威胁建模，并从沙箱模式、Work Directory 作用域划分、多层级工具过滤等维度给出了系统化的安全加固建议。

### 七、社区发展与里程碑

#### 7.1 关键版本演进

| 阶段 | 时间 | 关键事件 |
|------|------|---------|
| Clawdbot 时期 | 2026 年 1 月初至 1 月下旬 | 首个自主购车演示案例，AI 代理通过比价、议价、支付等 17 个步骤完成整车购买 |
| Moltbot 时期 | 2026 年 1 月下旬 | 商标争议应对期，模型蒸馏技术将响应延迟降低 40% |
| OpenClaw 时期 | 2026 年 1 月底至今 | 正式定名，200K+ GitHub Stars，开源基金会化进程启动 |

#### 7.2 生态影响力

OpenClaw 迅速跃升为 GitHub 上增长最快的开源仓库之一，与 React、Linux 等在历史星标榜上比肩。创始人 Peter Steinberger 被 OpenAI 招募，成为 AI Agent 领域“从开源项目到前沿研究机构”的标志性人才流动案例。

在 2026 年开源 AI Agent 生态中，OpenClaw 稳坐头把交椅，GitHub 星标约 24.7 万。主要竞品 Hermes Agent 于 2026 年 5 月首次在 OpenRouter 推理榜上以 2240 亿 token 日处理量反超 OpenClaw 的 1860 亿，标志该赛道进入激烈的良性竞争阶段。

OpenClaw 与 Nous Research 的 Hermes 代表了两种不同的技术哲学——前者以“连接一切”为核心理念构建多渠道协同系统，后者聚焦“自主进化”打造持续学习能力引擎。

### 八、总结

OpenClaw 构建了一套本地优先、模型无关、多渠道协同的自主 AI 执行引擎架构，将大语言模型从“聊天系统”转变为能真正在真实环境中执行任务的自动化系统。通过 Gateway 路由、Agent 运行时隔离、多级记忆系统、声明式配置以及可扩展的 Skill 机制，它为开发者提供了一套可私有化部署的 Agent 基础设施。

然而，赋予 LLM 自治的系统访问权限本身伴随显著安全风险。围绕 OpenClaw 的安全研究揭示了 Claw 类 Agent 固有的多维度攻击面，其默认配置的脆弱性已在学术审计中得到证实。对于考虑使用 OpenClaw 的用户而言，必须充分理解其安全模型边界并主动实施加固措施。

> 本文基于 OpenClaw 官方文档、GitHub 项目仓库、arXiv 论文和社区技术博客等公开信源编写，不构成官方技术背书或安全保证。建议读者结合 OpenClaw 官方文档 [docs.openclaw.ai](https://docs.openclaw.ai) 获取最新部署指导和更新说明。