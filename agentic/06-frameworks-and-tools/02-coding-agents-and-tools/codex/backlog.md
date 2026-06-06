# Codex 内容缺口 Backlog

本文记录 `codex/` 目录范围内已识别但尚未完成的内容缺口、待补证方向与后续研究入口。

- 它记录的是**问题缺口**，不是任务清单。
- 它不分配责任人、不设置 deadline，也不追踪完成状态。
- 若后续发现的是具体论文、仓库、页面或 benchmark 对象，应优先考虑沉淀到 `candidates.md`；若发现的是术语、事实或边界冲突，应优先进入 `conflict.md`。

---

## P0：已识别且高价值的补证缺口

### 1. `codex-architecture.md` 的源码级细节仍需逐项核验

- **关联文档**：`codex-architecture.md`
- **当前缺口**：文中仍保留了较多代码片段、文件路径、TaskGraph 数据结构、调度与沙箱参数等细节，但相当一部分只是公开线索与社区拆解的候选解释。
- **为何尚未覆盖**：当前目录已完成一轮 evidence 最低门槛治理，但仅完成了第一轮公开锚点核验；更细的逐文件、逐符号级源码复核仍未完成。
- **现有相关资源**：`https://github.com/openai/codex`、`codex-architecture.md`、`codex-multi-agent.md`
- **还缺什么**：下一步仍需要按“文件存在性 → 结构名 / 命令名 / 协议名 → 代码片段可追溯性”逐项核验；本轮已先确认了 `codex-rs/README.md` 中可见的 workspace / key crates / `codex sandbox` / `codex mcp-server` 等外显锚点，并据此收敛了文档表述。

### 2. `codex-multi-agent.md` 的边界已明显收紧，但仍缺更细粒度补证

- **关联文档**：`codex-multi-agent.md`
- **当前缺口**：本文已能用本地源码确认 `subagent`、子线程生成、父子线程关系、异步转发与会话级并发上限，但 worktree 与 automations 的更细实现边界仍未完全打通。
- **为何尚未覆盖**：本轮已回到本地官方仓库完成第一轮源码核验，并删除了 `Manager`、`TaskGraph`、`TaskChannel`、`Tokio mpsc` 主通道、`oneshot` 结果回传、冲突仲裁器等未证实说法；但 linked worktree 与产品层 Worktree 能力之间仍缺更直接连接。
- **现有相关资源**：`codex-multi-agent.md`、`codex-architecture.md`、`D:\github\codex\codex-rs\core\src\agent\control.rs`、`D:\github\codex\codex-rs\core\src\codex_delegate.rs`、`D:\github\codex\codex-rs\protocol\src\protocol.rs`
- **还缺什么**：若继续深挖，应补 `spawn_agent` 周边测试、线程上限配置的生效路径、以及 linked worktree 与官方产品 Worktree 叙述之间是否存在更直接的一手源码锚点。

### 3. Cloud / Local 主边界已对齐，但镜像 / 时效 / 留存规格仍待补证

- **关联文档**：`codex-cloud-sandbox.md`、`codex-agent-mechanisms.md`
- **当前缺口**：两篇文档现在都已用官方页面确认 Cloud 远程容器、本地执行边界、`setup → agent phase` 两阶段模型、Agent 默认离线、Secrets 仅在 Setup 可用，以及 Local / Worktree / Cloud 三种运行位置；但镜像规格、缓存寿命、环境销毁与数据留存等更细规格仍未完全打实。
- **为何尚未覆盖**：本轮已把社区体感数据与主线规格拆开，并同步收紧了 `codex-agent-mechanisms.md` 的治理层沙箱段；但某些更细环境页公开口径仍不完整或抓取不稳定。
- **现有相关资源**：`codex-cloud-sandbox.md`、`codex-agent-mechanisms.md`、`D:\github\codex\README.md`、`https://developers.openai.com/codex/app/features`、`https://developers.openai.com/codex/agent-approvals-security`、`https://developers.openai.com/codex/cloud/internet-access`、`https://developers.openai.com/codex/llms-full.txt`
- **还缺什么**：若继续深挖，应补 Cloud environment 页面对基础镜像、缓存寿命、环境销毁/持久化与数据留存的更稳定一手口径，并继续判断 App 本地执行边界是否还有更直接的一手规格页。

---

## P1：有价值但边界仍模糊的补充方向

### 4. `overview.md` 与主线文档之间的边界仍需继续收敛

- **关联文档**：`overview.md`、`codex-agent-overview.md`、`codex-agent-mechanisms.md`、`codex-agent-practice.md`
- **当前缺口**：`overview.md` 已重写为面向读者的领域综述，但其中哪些整体判断应进一步吸收到主线、哪些应继续保留为综述层表达，仍需继续收敛。
- **为何尚未覆盖**：本轮已先完成 `overview.md` 的定位切换与结构重写，但还没有逐节核对它与主线三篇在结论粒度上的边界。
- **现有相关资源**：`overview.md`、各主线文档的 `Evidence` 小节
- **还缺什么**：需要判断哪些内容已经达到 `Verified / Observed / Inferred` 可进入主线的门槛，哪些更适合继续留在领域综述层。

### 5. Computer Use / Automations / Troubleshooting 已完成首轮主边界治理，但仍需补更细规格

- **关联文档**：`codex-computer-use.md`、`codex-automations-ci.md`、`codex-troubleshooting.md`
- **当前缺口**：三篇专题现在都已完成一轮主边界收紧：`automations` 可较稳确认 thread automation、project / standalone automation 与 Git 仓库中的 dedicated background worktrees；`troubleshooting` 可较稳确认官方 Troubleshooting 页面、日志 / session transcript / archived sessions 路径，以及 stuck state 的第一层恢复动作；`computer-use` 可较稳确认 `Computer Use` 与 `In-app browser` 的产品入口、无需登录页面支持范围、登录态限制与网站控制面。但三篇文档的更细语义仍存在未补证部分。
- **为何尚未覆盖**：本轮优先用官方产品页把产品层对象、入口边界、恢复动作与权限/网站控制面拆开；但更细入口页（如 `GitHub Action`、`Non-interactive Mode`、`Codex SDK`、`App Server`、`MCP Server`、`Computer Use` 独立正文）与更细恢复/审计语义仍没有逐页补证。
- **现有相关资源**：对应专题文档、`https://developers.openai.com/codex`、`https://developers.openai.com/codex/app/features`、`https://developers.openai.com/codex/app/troubleshooting`、`https://developers.openai.com/codex/cli/features`、`https://developers.openai.com/codex/agent-approvals-security`、`conflict.md`
- **还缺什么**：下一步更适合逐页深挖自动化相关各入口、会话恢复 / known issues / changelog，以及 `Computer Use` 独立页面或更细能力规格；同时判断哪些内容应拆到 `candidates.md` 或其他专题。

### 6. 开源边界主结论已收紧，但替代路径判断仍待对象级回填

- **关联文档**：`codex-open-source.md`、`candidates.md`
- **当前缺口**：本文现已能用官方页面确认 `CLI / SDK / App Server / Skills / codex-universal` 的开源范围，以及 `IDE extension / Codex web` 的非开源边界；但“本地模型接入是否足够替代官方能力”仍未完成对象级回填。
- **为何尚未覆盖**：本轮已把“整体开源/整体闭源”的粗粒度说法拆成官方组件清单、不开源项、运行形态边界与可替换后端边界；并已把 `Ollama`、`LM Studio`、`Bedrock`、`codex-plugin-cc` 等登记到 `candidates.md`，但具体对象研究尚未完成。
- **现有相关资源**：`codex-open-source.md`、`candidates.md`、`https://developers.openai.com/codex/open-source`、`D:\github\codex\README.md`、`D:\github\codex\codex-rs`
- **还缺什么**：继续按对象逐项核验接入方式、能力覆盖、限制条件与官方关系，再决定哪些内容可回写主线、哪些仍应保留为候选对象或观察结论。

---

## P2：长期方向

### 7. 为 `codex/` 目录建立“对象文档 ↔ 一手来源”轻量映射

- **关联范围**：`codex/` 全目录
- **当前缺口**：现在每篇文档都有最小 `Evidence`，但还没有形成跨文档的一手来源映射关系。
- **为何尚未覆盖**：当前仓库规则明确不要求机械建立 `evidence-registry`，而当前阶段也不需要过重结构。
- **现有相关资源**：各文档文末 `Evidence`
- **还缺什么**：如果未来 `codex/` 继续长期维护，可能需要按主题建立更稳定的来源索引，但应按需启用，不应现在机械创建。

### 8. `candidates.md` 已建立，但对象队列仍需持续整理

- **关联范围**：`codex/` 全目录
- **当前缺口**：`candidates.md` 已建立并收录开源替代路径与部分官方对象，但对象覆盖范围仍偏初始，尚未形成更完整的持续研究队列。
- **为何尚未覆盖**：本轮先完成了“问题缺口 → 对象队列”的元信息分流，尚未继续扩展到更多官方页面、仓库模块或 benchmark 级对象。
- **现有相关资源**：`backlog.md`、`candidates.md`、`conflict.md`
- **还缺什么**：后续若继续稳定跟踪具体对象，可继续把 cloud 细规格页面、automations 各入口、multi-agent / worktree 直接源码锚点等拆成更清晰的候选对象。
