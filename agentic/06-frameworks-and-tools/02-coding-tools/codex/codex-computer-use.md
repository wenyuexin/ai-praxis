# Codex Computer Use 与浏览器操作边界

> **专题文档**。本文只整理目前能从 OpenAI 官方 Codex 文档中保守确认的 Computer Use / browser use / in-app browser 相关边界，重点回答：它在产品层大致是什么、与普通页面预览有何不同、以及哪些 GUI 自动化叙述仍不宜写成稳定结论。

> 截至当前，可以较明确确认的点包括：Codex 文档体系存在独立的 `Computer Use` 入口；`In-app browser` 是单独能力；in-app browser 适合预览本地开发服务器、文件预览与无需登录的公开页面；in-app browser 不支持认证流程、已登录页面、常规浏览器 profile、cookies、extensions 或既有 tabs；当需要让 Codex 直接操作页面时，官方建议使用 browser use。至于 SSH Key 双重认证、稳定跨服务流程、GUI 点击/输入/截图的完整动作列表、以及“无法处理 CAPTCHA / 无法下载到本地磁盘”等更细能力矩阵，当前都不应继续写成稳定一手规格。

---

## 1. 当前可直接确认的产品层对象

### 1.1 `Computer Use` 是官方公开入口，但当前公开细节有限

从 `https://developers.openai.com/codex` 当前可直接看到：

- `Computer Use` 是 Codex App 文档导航中的独立入口；
- 相关能力还与 `In-app browser`、`Chrome extension` 并列出现。

这至少说明：

- **Computer Use 是官方公开产品对象**，不是纯社区叫法；
- 它与普通页面预览、浏览器扩展等能力并不是同一个入口。

但在当前可抓取的公开文本里，`Computer Use` 专页本身没有稳定拿到详细正文，因此更细能力边界暂时不宜写死。

### 1.2 `In-app browser` 与“直接操作页面”是两个层次

官方 `app/features` 页面当前可直接确认：

- `In-app browser` 用于 preview、review、comment on local development servers、file-backed previews 与无需登录的 public pages；
- `Use browser comments` 可以在页面特定元素或区域上标注反馈；
- `When you want Codex to operate the page directly, use browser use ...`

因此，当前最稳妥的理解是：

| 能力 | 当前可确认的边界 |
|------|------------------|
| **In-app browser** | 更偏页面预览、检查与评论 |
| **Browser use / Computer Use** | 更偏让 Codex 直接操作页面 |

这比把所有页面相关能力都统称为“远程操控 GUI”更稳。

---

## 2. 当前可直接确认的页面与登录态边界

### 2.1 In-app browser 明确不支持一批浏览器态能力

官方 `app/features` 页面可直接确认，`In-app browser` **doesn’t support**：

- authentication flows
- signed-in pages
- your regular browser profile
- cookies
- extensions
- existing tabs

这意味着：

- 至少在 **in-app browser** 这一层，登录态与常规浏览器环境不是默认继承的；
- 若任务依赖已登录会话、浏览器扩展、持久 cookie 或既有标签页，上述能力边界会成为第一层限制。

### 2.2 官方目前更明确支持的是“无需登录页面”与本地预览场景

官方 `app/features` 页面更明确支持的场景是：

- local development servers
- file-backed previews
- public pages that don’t require sign-in

因此，当前更稳妥的正文写法应是：**公开一手材料更清楚地支持“本地预览 / 无登录页面 / 页面评论”，而不是广泛支持任意网站、任意登录流和任意外部系统。**

---

## 3. Computer Use 与项目/权限边界的关系

### 3.1 项目边界与审批仍然成立

结合 `app/features` 当前可直接确认：

- approvals 决定 Codex 什么时候会暂停并请求许可；
- sandbox 决定它可以访问哪些目录与网络；
- 默认情况下，Codex 会把工作范围限制在当前 project；
- 如果任务跨多个目录或仓库，官方更建议 separate projects 或 worktrees，而不是让 Codex 任意越出 project root。

这意味着：

- 即使讨论 Computer Use / browser use，也不能脱离 approvals、sandbox 与 project scope；
- 页面操作能力并不自动等价于“可无边界访问本机或任意目录”。

### 3.2 Browser plugin 与网站 allow/block 清单是官方明示的控制点

官方 `app/features` 页面还能确认：

- 可以在 settings 里管理 Browser plugin；
- 可以配置 allowed websites 与 blocked websites。

因此，更稳妥的结论是：**页面操作能力至少带有网站级别的显式控制面，而不是完全开放。**

---

## 4. 当前不宜继续写死的说法

下列说法在当前一手材料下仍不宜继续写成正文定论：

| 不宜写死的说法 | 当前状态 |
|----------------|----------|
| Computer Use 已公开完整动作矩阵：点击、输入、滚动、切 tab、截图等 | 当前未重新取得稳定一手正文锚点 |
| Computer Use 一定能稳定跨 GitHub → Jira → Slack 完成整条 GUI 流程 | 社区样本，不宜写成规格 |
| 官方明确规定“不能处理 CAPTCHA / 不能下载文件到本地磁盘 / 不能编辑 DOM” | 当前未重新取得稳定一手锚点 |
| Computer Use 的认证模型就是 `SSH Key + API Key` 双重认证 | 当前未重新取得稳定一手锚点 |
| 每个点击 / 输入 / 截图都有统一公开审计语义 | 当前缺少稳定一手规格页支持 |
| Computer Use 本质上等同远程桌面 | 当前不宜写死；官方更明确的是 browser / page operation 语义 |
| 已确认存在稳定公开的 `developers.openai.com/codex/computer-use` 正文页面 | 当前抓取结果不稳定，不能据此写正文细节 |

这意味着：**Computer Use 专题更适合先写“产品入口、browser use 与 in-app browser 的关系、登录态限制、网站控制面与 project/sandbox 边界”，而不是先写详细 GUI 操作矩阵。**

---

## 5. 当前可接受的保守理解

### 5.1 页面预览与页面操作应分开理解

当前较稳的理解是：

- `In-app browser` 负责预览、检查、评论页面；
- `Browser use / Computer Use` 更接近让 Codex 直接操作页面；
- 两者相关，但不应混成同一能力。

### 5.2 登录态与常规浏览器环境不是默认前提

当前较稳的理解是：

- 无需登录的公开页面与本地预览场景更接近一手支持范围；
- 认证流程、已登录页、cookies、profile、extensions、existing tabs 在 in-app browser 层已明确受限；
- 更复杂的跨站登录流、长期会话与高稳定性 GUI 编排仍需额外补证。

---

## 6. 当前可接受的一句话总结

> 截至当前，更稳妥的说法是：Codex 已明确公开 `Computer Use` 与 `In-app browser` 相关产品入口，且官方更清楚支持的是本地预览、无需登录页面、页面评论以及在受控网站范围内的页面操作；但详细 GUI 动作矩阵、跨服务稳定性、安全实现细节与认证模型，仍不应写成已核实规格。

---

## Evidence

- Status: Mixed
- Sources: OpenAI 官方页面 `https://developers.openai.com/codex`、`https://developers.openai.com/codex/app/features`；既有社区案例材料。
- Trace: 本文已从“能力矩阵 + 社区案例混写”收缩为“官方可确认产品入口 + browser/in-app-browser 边界 + 未找到项 + 社区样本外推限制”结构；凡缺少当前稳定一手锚点的 GUI 动作清单、认证细节与跨服务能力，均不再写成主线定论。
- Needs: 若继续补证，应核验 `Computer Use` 独立页面是否存在稳定可抓取正文、`Chrome extension` 与 `In-app browser` 的职责边界、以及 browser use 的审计/权限模型是否有更明确公开说明。

## 参考来源

- `https://developers.openai.com/codex`
- `https://developers.openai.com/codex/app/features`

*最后更新: 2026-06-04*
