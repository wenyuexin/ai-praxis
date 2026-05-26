# Codex Automations & CI/CD 集成

> **专题文档**。本文聚焦 Automations 和 CI/CD 集成，适合评估工程化落地时查阅。不属于主线阅读路径。



> 基于 `developers.openai.com/codex` 官方文档 + 社区实测整理（2026.5 最新）。

---

## 1. Automations 触发器类型

| 触发器 | 说明 | 示例 | 证据等级 |
|--------|------|------|---------|
| **定时（Cron）** | 按固定时间触发 | 每天 9:00 跑 lint 检查、每周一生成周报 | A-官方 |
| **Webhook** | 接收外部 HTTP 请求触发 | GitHub PR 创建 → 触发 Code Review | A-官方 |
| **事件驱动** | 监听特定事件自动触发 | Issue 评论 → 自动分类、CI 失败 → 自动分析日志 | A-官方 |
| **手动触发** | 用户主动调用 | `/automation run <name>` | A-官方 |

---

## 2. 真实配置案例

### 案例 1：PR 自动代码审查（官方推荐）

```
触发条件：GitHub PR 创建（webhook）
执行动作：codex review → 生成评论 → 如无问题则自动 approve
配置位置：AGENTS.md + Automations 配置
效果：几秒钟启动，无需离开页面
```
证据等级：B-社区实测

### 案例 2：每日 Lint 监控 + 告警

```
触发条件：每天 8:00（cron）
执行动作：运行 lint → 如有问题 → 发送 Slack 告警
MCP 联动：Slack MCP Server 发送消息
效果：团队不再人工跑 lint
```
证据等级：B-什么值得买

### 案例 3：CI 失败自动分析

```
触发条件：GitHub Actions 失败（webhook）
执行动作：codex 分析日志 → 定位根因 → 创建 issue
MCP 联动：GitHub MCP Server 创建 issue
效果：80% 的 CI 失败可自动定位，无需人工介入
```
证据等级：B-社区

### 案例 4：Skill + Automation 组合

```
Skill："分析这批日志"（手动触发）
Automation：每天 23:00 自动运行该 Skill，生成日报
效果：一次性写 Skill，之后零边际成本自动运行
```
证据等级：B-什么值得买

---

## 3. CI/CD 集成深度

| CI/CD 工具 | 集成方式 | 深度 | 证据等级 |
|-----------|----------|------|---------|
| **GitHub Actions** | 官方 MCP Server + webhook 触发 | ⭐⭐⭐⭐ 完整 | A-官方 |
| **CircleCI** | 官方 MCP Server | ⭐⭐⭐⭐ 稳定 | A-官方 |
| **Jenkins** | 通过 MCP Server（社区维护） | ⭐⭐⭐ 可用 | B-GitHub |
| **GitLab** | 官方 MCP Server | ⭐⭐⭐⭐ 稳定 | A-官方 |
| **Jira** | 官方 MCP Server | ⭐⭐⭐⭐ 稳定 | A-官方 |
| **Slack** | 官方 MCP Server | ⭐⭐⭐⭐ 稳定 | A-官方 |

> 🔑 **集成模式**：`事件触发 → MCP Server 接收 → Codex 执行 → MCP Server 回调`

---

## 4. 与 MCP Server 联动模式

| 模式 | 说明 | 示例 |
|------|------|------|
| **MCP 触发 Automation** | MCP Server 收到外部事件 → 触发 Codex Automation | GitHub MCP 收到 PR → 触发 Code Review Automation |
| **Automation 调用 MCP** | Automation 定时运行 → 调用 MCP 获取数据 → 执行分析 | 每天 9:00 → Slack MCP 获取未读消息 → 生成摘要 |
| **Skill 编排 MCP** | Skill 定义流程 → 内部调用多个 MCP Server | "分析日志" Skill → 调用 GitHub MCP + Slack MCP |

证据等级：A-官方 + B-社区

---

## 5. 官方 MCP Server 清单（Automations 高频使用）

| MCP Server | 用途 | 稳定性 | 证据等级 |
|-----------|------|--------|---------|
| **GitHub** | PR/Issue 管理、触发 Automation | ⭐⭐⭐⭐⭐ | A-官方 |
| **CircleCI** | CI 状态查询、触发构建 | ⭐⭐⭐⭐ | A-官方 |
| **GitLab** | MR 管理、文件操作 | ⭐⭐⭐⭐ | A-官方 |
| **Slack** | 发送告警、通知 | ⭐⭐⭐⭐ | A-官方 |
| **Jira** | 创建/更新 issue | ⭐⭐⭐⭐ | A-官方 |
| **总计** | **90+ 插件** | — | A-官方 |

---

## 参考来源

- https://developers.openai.com/codex
- https://openai.com/index/introducing-upgrades-to-codex/

*最后更新: 2026-05-26*