# Codex Cloud 执行与沙箱边界

> **专题文档**。本文只整理目前能从 OpenAI 官方 Codex 文档与本地开源仓库入口中保守确认的 Cloud / Local / Worktree 执行边界，重点回答：代码在哪里运行、网络何时开放、缓存如何生效、以及哪些说法仍不宜写成稳定规格。

> 截至当前，可以较明确确认的点包括：Codex CLI 本地运行；Codex Web 是 cloud-based agent；Codex Cloud 运行在 OpenAI 托管的隔离容器中；Cloud 采用 `setup → agent phase` 两阶段模型；Agent 阶段默认离线；Secrets 仅在 Setup 阶段可用；Worktree 与 Local 都在用户本机运行。连续执行时长、固定 CPU/内存规格、以及部分成本样本仍不应写成稳定一手口径。

---

## 1. 当前可直接确认的 Cloud / Local 边界

### 1.1 CLI、本地 App 与 Web 并不是同一种运行位置

根据本地仓库 `D:\github\codex\README.md` 与官方文档，可直接确认：

| 形态 | 当前可确认的边界 | 一手锚点 |
|------|------------------|----------|
| **Codex CLI** | `runs locally on your computer` | `D:\github\codex\README.md` |
| **Codex App / Local** | 线程直接在当前项目目录运行 | `https://developers.openai.com/codex/app/features` |
| **Codex App / Worktree** | 在 Git worktree 中隔离修改，但运行位置仍在本机 | `https://developers.openai.com/codex/app/features` |
| **Codex App / Cloud** | 在配置好的远程 cloud environment 中运行 | `https://developers.openai.com/codex/app/features` |
| **Codex Web** | `cloud-based agent` | `D:\github\codex\README.md` |

因此，当前最稳妥的说法是：**Local / Worktree 更接近“本机执行”，Cloud / Web 更接近“远程环境执行”。**

### 1.2 Cloud 容器与本地沙箱是两套不同模型

官方 `agent-approvals-security` 页面明确区分了两类执行边界：

- **Codex cloud**：运行在 `isolated OpenAI-managed containers` 中，避免访问宿主机系统或无关数据。
- **Codex CLI / IDE extension**：由 OS-level mechanisms 执行本地沙箱策略，默认无网络、默认仅可写活动 workspace。

这意味着：

- Cloud 的隔离核心是**远程托管容器**；
- CLI / IDE / App Local 的隔离核心是**本机受限执行边界**；
- 不能把本地沙箱术语直接等同于 Cloud 容器模型。

---

## 2. Cloud 两阶段运行模型

### 2.1 `setup → agent phase` 是当前最强的一手规格

官方 `agent-approvals-security` 与 `cloud/internet-access` 页面都明确提到：Codex Cloud 使用 two-phase runtime model。

| 阶段 | 当前可确认的行为 | 一手锚点 |
|------|------------------|----------|
| **Setup** | 在 agent phase 之前运行；可访问网络以安装指定依赖 | `https://developers.openai.com/codex/agent-approvals-security`、`https://developers.openai.com/codex/cloud/internet-access` |
| **Agent phase** | 默认离线；只有为该环境显式开启 internet access 时才可联网 | 同上 |
| **Secrets** | 仅在 Setup 阶段可用，agent phase 开始前移除 | `https://developers.openai.com/codex/agent-approvals-security` |

这比旧版“审批队列工作流图”更稳，因为它直接对应官方对运行时的描述。

### 2.2 Agent 网络不是全局开启，而是环境级配置

官方 `cloud/internet-access` 页面可直接确认：

- Agent 阶段默认阻断互联网访问；
- Internet access 是 **per-environment** 配置；
- 可通过 allowlist 控制域名；
- 可进一步限制允许的 HTTP 方法；
- 更保守的做法是只放开 `GET`、`HEAD`、`OPTIONS`。

因此，更稳妥的写法应是：**Cloud 并非“有网”或“没网”的单一模式，而是 Setup 默认可联网、Agent 默认离线、再按环境粒度放开特定网络能力。**

---

## 3. Worktree、Local 与 Cloud 的关系

### 3.1 Worktree 是本机执行模式，不等于 Cloud

官方 `app/features` 页面明确写到：

- `Local: work directly in your current project directory.`
- `Worktree: isolate changes in a Git worktree.`
- `Cloud: run remotely in a configured cloud environment.`
- `Both Local and Worktree threads will run on your computer.`

因此可以直接确认：

- Worktree 与 Local 一样，执行位置都在本机；
- Worktree 的核心价值是**修改隔离**，不是把运行位置切到云端；
- Cloud 则是单独的远程执行形态。

### 3.2 Automations 与 Worktree 的公开边界

官方 `app/features` 页面还明确写到：

- `Automations run in dedicated background worktrees for Git repositories`
- 对非版本控制项目，Automations 直接在项目目录运行。

这可以支撑的最保守结论是：

- **Automations 与 worktree 的关系在产品层是明确存在的**；
- 对 Git 仓库，Automations 会使用专用后台 worktrees；
- 但这不自动等价于“所有 Cloud 任务”或“所有 subagent”都固定使用同一套 worktree 机制。

---

## 4. 缓存、Setup 脚本与环境复用

### 4.1 当前能直接确认的缓存行为

从 `https://developers.openai.com/codex/llms-full.txt` 当前可提取到的明确表述包括：

- 当环境首次缓存时：Codex 克隆仓库、checkout 默认分支、运行 setup script，并缓存 resulting container state。
- 当缓存容器恢复时：Codex checkout 任务指定分支，并运行 optional maintenance script。
- 当 setup script、maintenance script、environment variables 或 secrets 变化时，缓存会自动失效。

这说明：

- Cloud 缓存并不是“只缓存依赖目录”的含糊概念；
- 它更接近**环境容器状态复用**；
- `maintenance script` 的角色是处理“缓存来自旧 commit”时的增量更新。

### 4.2 旧版 `12 小时缓存` 口径不宜继续写死

当前官方可直接抓取到的公开文本里，我没有重新拿到“最长 12 小时”这一规格的稳定一手锚点。

因此此时更稳妥的处理是：

- 保留“存在缓存与恢复机制”这一事实；
- 不把 `12 小时` 写成当前稳定规格；
- 若后续再次获得明确官方页面，再补回精确时效。

---

## 5. 当前不宜写成稳定规格的说法

下列说法在当前一手材料下仍不宜继续写死：

| 不宜写死的说法 | 当前状态 |
|----------------|----------|
| Cloud 固定使用 `openai/codex-universal` 作为公开基础镜像规格 | 当前未重新取得稳定一手锚点 |
| Cloud 有公开固定 CPU / 内存配额 | 未找到 |
| Cloud 可稳定连续执行 `7h+` | 更像社区样本，不宜写成规格 |
| Cloud 缓存最长固定 `12 小时` | 当前未重新取得稳定一手锚点 |
| 所有 Cloud 任务都会在 worktree 中运行 | 未找到 |
| 本地沙箱与 Cloud 容器具有相同安全属性 | 不成立，属于不同模型 |
| 成本样本可直接外推到所有模型与任务 | 不成立，需看模型 / token / 场景口径 |

这意味着：**Cloud / Local 章节更适合围绕运行位置、阶段边界、网络策略、缓存机制与环境复用写事实，而不要混入过多社区体验数字。**

---

## 6. 当前可接受的 Cloud / Local 对比

| 维度 | Cloud | Local / Worktree |
|------|-------|------------------|
| **执行位置** | OpenAI 托管的隔离容器 | 用户本机 |
| **默认网络** | Setup 可联网；Agent 默认离线 | 本地沙箱默认无网络（CLI / IDE 文档口径） |
| **隔离核心** | 远程容器边界 | OS-level sandbox / workspace boundary |
| **修改隔离** | 取决于环境与任务形态，不应泛化 | Worktree 可提供 Git 级修改隔离 |
| **Secrets 使用时机** | 仅 Setup 阶段 | 取决于本地环境与配置，不同于 Cloud Secrets 机制 |

> 一句话说：**Cloud 更像“远程容器 + 两阶段执行 + 环境级网络控制”，Local / Worktree 更像“本机受限执行 + 目录或 Git worktree 隔离”。**

---

## Evidence

- Status: Mixed
- Sources: `D:\github\codex\README.md`；OpenAI 官方页面 `https://developers.openai.com/codex/app/features`、`https://developers.openai.com/codex/agent-approvals-security`、`https://developers.openai.com/codex/cloud/internet-access`、`https://developers.openai.com/codex/llms-full.txt`、`https://developers.openai.com/codex/app/local-environments`。
- Trace: 本文已从“规格表 + 社区样本混写”收缩为“官方可确认边界 + 未找到项 + 外推限制”结构；凡缺少当前稳定一手锚点的镜像、时长、缓存时效、成本数字，均不再写成主线定论。
- Needs: 若继续补证，可进一步核验 Cloud environment 页面对镜像、缓存寿命、数据留存与环境销毁是否存在更明确的公开规格；也可补充本地 CLI 沙箱在 Windows / macOS / Linux 的平台差异锚点。

## 参考来源

- `D:\github\codex\README.md`
- `https://developers.openai.com/codex`
- `https://developers.openai.com/codex/app/features`
- `https://developers.openai.com/codex/app/local-environments`
- `https://developers.openai.com/codex/agent-approvals-security`
- `https://developers.openai.com/codex/cloud/internet-access`
- `https://developers.openai.com/codex/llms-full.txt`

*最后更新: 2026-06-04*
