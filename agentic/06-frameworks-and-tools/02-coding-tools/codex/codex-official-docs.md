# Codex 官方文档入口图

> **专题文档**。本文只整理 `Codex` 官方资料的入口结构、页面分组和查阅顺序，帮助读者快速定位一手来源。它不是主线综述，也不负责承接术语冲突、产品边界裁决或实现细节结论。

> 以 `https://developers.openai.com/codex` 为主要入口；官方站点结构可能调整，使用时仍应以当前导航为准。

---

## 一、官方资料的几类主要入口

当前最值得优先关注的官方入口大致可分为几组：

- **Overview / Quickstart**：回答 `Codex` 是什么、如何开始用
- **App**：回答桌面端、功能页、Computer Use、Remote Connections 等产品能力
- **CLI**：回答命令行参考、配置方式、子命令与非交互调用
- **Guides**：回答 `AGENTS.md`、`MCP`、sandbox、profiles 等机制与实践问题
- **Open Source / GitHub**：回答哪些组件开源、仓库结构在哪里
- **产品公告与实践材料**：回答版本变化、官方实践、产品定位更新

---

## 二、官方文档主站的查阅路径

官方 hub 位于：`https://developers.openai.com/codex`

可先按下面的路径理解：

### 2.1 入门入口

- `https://developers.openai.com/codex`
- `https://developers.openai.com/codex/quickstart`

适合回答：

- `Codex` 是什么
- 首次安装和启动方式是什么
- CLI / App / Cloud 大致有哪些入口

### 2.2 App 与产品能力入口

- `https://developers.openai.com/codex/app/features`
- `https://developers.openai.com/codex/app/computer-use`
- `https://developers.openai.com/codex/remote-connections`
- `https://developers.openai.com/codex/app/troubleshooting`

适合回答：

- App 中有哪些能力对象
- Computer Use、Browser、Worktrees、Automations 等分别是什么
- Remote Connections 与 App 能力如何关联
- App 场景下的常见排障入口在哪里

### 2.3 CLI 与配置入口

- `https://developers.openai.com/codex/cli`
- `https://developers.openai.com/codex/cli/reference`
- `https://developers.openai.com/codex/cli/config`
- `https://developers.openai.com/codex/cli/features`

适合回答：

- CLI 子命令有哪些
- 非交互执行、配置文件、profiles、sandbox 相关入口在哪里
- CLI 用户应优先查哪几页

### 2.4 Guides 与机制入口

- `https://developers.openai.com/codex/guides/agents-md`
- `https://developers.openai.com/codex/guides/mcp`
- `https://developers.openai.com/codex/guides/sandboxing`
- `https://developers.openai.com/codex/guides/profiles`

适合回答：

- `AGENTS.md` 怎么写
- `MCP` 怎么接入外部工具
- sandbox / approval / profiles 等治理相关问题去哪里看

### 2.5 开源与仓库入口

- `https://developers.openai.com/codex/open-source`
- `https://github.com/openai/codex`
- `https://github.com/openai/skills`
- `https://github.com/openai/codex-universal`

适合回答：

- 哪些组件明确开源
- 官方仓库和相关开源对象在哪里
- 开源工具链与产品形态边界应从哪里找一手资料

### 2.6 官方公告与实践入口

- `https://openai.com/index/introducing-the-codex-app/`
- `https://openai.com/index/introducing-upgrades-to-codex/`
- `https://cdn.openai.com/pdf/6a2631dc-783e-479b-b1a4-af0cfbd38630/how-openai-uses-codex.pdf`

适合回答：

- 产品近期新增了什么能力
- 官方如何描述真实工程使用方式
- 功能上线节奏与产品口径如何变化

---

## 三、按问题类型查官方资料

### 3.1 想先知道 `Codex` 值不值得看

优先阅读：

- `https://developers.openai.com/codex`
- `https://developers.openai.com/codex/quickstart`
- `https://developers.openai.com/codex/app/features`

### 3.2 想落地到自己的项目

优先阅读：

- `https://developers.openai.com/codex/guides/agents-md`
- `https://developers.openai.com/codex/guides/mcp`
- `https://developers.openai.com/codex/cli/config`
- `https://developers.openai.com/codex/cli/reference`

### 3.3 想理解产品能力边界

优先阅读：

- `https://developers.openai.com/codex/app/features`
- `https://developers.openai.com/codex/app/computer-use`
- `https://developers.openai.com/codex/remote-connections`
- `https://developers.openai.com/codex/open-source`

### 3.4 想看源码或开源实现

优先阅读：

- `https://github.com/openai/codex`
- `https://developers.openai.com/codex/open-source`
- `https://github.com/openai/skills`
- `https://github.com/openai/codex-universal`

---

## 四、使用本文时的边界

本文适合做：

- 官方页面入口图
- 页面分组与查找路径
- 阅读顺序建议
- 一手来源定位

本文不适合做：

- 主线综述
- 冲突裁决
- 产品边界定论
- 源码实现分析
- 用社区材料替代官方页面下结论

若问题已经从“去哪找官方页”转向“这件事到底怎么理解”，应回到主线或相应专题：

- 领域综述：`overview.md`
- 主线认知：`codex-agent-overview.md`、`codex-agent-mechanisms.md`、`codex-agent-practice.md`
- 环境边界：`codex-cloud-sandbox.md`
- 多代理与工作域：`codex-multi-agent.md`
- 开源边界：`codex-open-source.md`
- 冲突与口径不一致：`conflict.md`

---

## Evidence

- Status: Mixed
- Sources: `developers.openai.com/codex` 及其相关子页面；OpenAI 官方产品公告；官方实践 PDF；官方 GitHub 仓库。
- Trace: 本文已从“导航 + 边界澄清混写”收紧为“官方资料入口图”；只保留页面分组、查阅路径和阅读顺序建议，不再承接冲突裁决与产品边界定论。
- Needs: 继续定期复核官方站点的栏目结构、路径命名和 App / CLI / Guides 的页面分工。

## 参考来源

- `https://developers.openai.com/codex`
- `https://developers.openai.com/codex/quickstart`
- `https://developers.openai.com/codex/app/features`
- `https://developers.openai.com/codex/app/computer-use`
- `https://developers.openai.com/codex/remote-connections`
- `https://developers.openai.com/codex/app/troubleshooting`
- `https://developers.openai.com/codex/cli`
- `https://developers.openai.com/codex/cli/reference`
- `https://developers.openai.com/codex/cli/config`
- `https://developers.openai.com/codex/guides/agents-md`
- `https://developers.openai.com/codex/guides/mcp`
- `https://developers.openai.com/codex/guides/sandboxing`
- `https://developers.openai.com/codex/open-source`
- `https://github.com/openai/codex`
- `https://openai.com/index/introducing-the-codex-app/`
- `https://openai.com/index/introducing-upgrades-to-codex/`

*最后更新: 2026-06-04*
