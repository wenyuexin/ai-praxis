# OpenCode Session and Server

> 本文解释 OpenCode 的 session、server API、message、permission、share、diff、revert 等机制如何构成可恢复、可审查、可协作的 agent execution substrate。

## 核心判断

OpenCode 的 session 不是简单对话缓存，而是贯穿 CLI、TUI、server、web、desktop 和 API 的核心对象。它可以被列举、继续、fork、分享、摘要、diff、revert，并通过 message、part、event、permission 等结构持久化。

这让 OpenCode 具备几个重要能力：

- 同一任务可以跨入口继续或连接。
- 工具执行和模型输出可以通过 message / part / event 被记录。
- 修改结果可以 diff、revert 或 unrevert。
- 会话可以 share 给团队或外部界面。
- permission request 可以通过 server API 暴露给客户端处理。

## API Surface

主 `packages/opencode` server 暴露 legacy instance API，并挂载 v2 `/api` server API。

Legacy session API 覆盖：

- list / get / children / messages / message。
- create / update / remove / fork。
- share / unshare。
- summarize。
- prompt / prompt_async / command / shell。
- revert / unrevert。
- message / part delete 和 update。

Permission API 覆盖 pending request 列表和 request reply。V2 API 则以 `packages/server` 的 groups / handlers 组织 session、message、permission、agent、skill、model、provider 等资源。

## Storage Model

OpenCode 当前同时存在 legacy projection 与 V2 事件/投影相关结构。

核心持久化结构包括：

- `session`：保存 session identity、project/workspace、parent/fork、title、share URL、summary、revert、permission、agent/model、时间戳等。
- `message` / `part`：保存 legacy conversation 结构。
- `session_message`：保存 V2 projected messages。
- `session_input`：保存 V2 durable admitted inputs。
- `session_context_epoch`：保存 V2 context epoch / compaction 相关状态。
- `event_sequence` / `event`：保存事件日志。
- `session_share`：保存分享状态。
- `permission`：保存 V2 permission rows。

这说明 OpenCode 正在从传统 session/message 存储向更明确的 durable input、projected message 和 event log 方向演进。

## Session Lifecycle

Legacy session 创建时会记录项目、目录、agent、model、permission、成本和时间戳，并发布 session created 事件。后续更新通过 patch 和 event projector 落到 read model。

V2 `SessionV2.prompt` 更强调 durable prompt admission：先把 prompt 写入 `session_input`，再唤醒 execution。这个设计把“接受用户输入”和“执行模型调用”分开，有利于重试、恢复和并发调度。

## Continue / Fork / Share

CLI 文档中的 `--continue`、`--session`、`--fork`、`--share` 与 server API 的 session routes 相互对应：

- Continue / session ID 支持恢复既有 session。
- Fork 支持从既有 session 分叉。
- Share / unshare 支持把 session 暴露为可分享对象。
- Attach 支持 TUI 连接已有 server，从而连接已有 session backend。

这组能力说明 OpenCode 把 session 视为长期对象，而不是一次终端会话的临时上下文。

## Diff / Revert

Session API 提供 diff、revert、unrevert。源码中 revert 相关逻辑会保存 session diff，并通过 storage/session state 更新回滚信息。

这对 coding agent 尤其重要：agent 的输出不是纯文本回答，而是文件系统变更。diff / revert 是把 agent 行为纳入人类审查和恢复机制的关键层。

## Permission as Session Interaction

Permission request 不只是本地 prompt。server API 暴露 permission 列表和 reply，说明客户端可以把权限请求作为 session interaction 处理。这让 TUI、web、desktop 或远程 attach 都有机会参与同一权限门控流程。

## 设计意义

OpenCode 的 session/server 设计把 coding agent 的关键 runtime 问题变成了 API 对象：

- prompt 如何进入 durable 队列。
- message 如何投影给客户端。
- permission 如何请求和回复。
- 文件变更如何 diff / revert。
- session 如何 fork / share / summarize。

这使它非常适合研究从“agent loop”到“agent execution substrate”的产品化路径。

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - <https://opencode.ai/docs/zh-cn/cli>
  - <https://opencode.ai/docs/zh-cn/server>
  - `packages/opencode/src/server/routes/instance/httpapi/groups/session.ts`
  - `packages/opencode/src/server/routes/instance/httpapi/handlers/session.ts`
  - `packages/opencode/src/server/routes/instance/httpapi/groups/permission.ts`
  - `packages/opencode/src/server/routes/instance/httpapi/handlers/permission.ts`
  - `packages/server/src/api.ts`
  - `packages/server/src/groups/session.ts`
  - `packages/server/src/groups/message.ts`
  - `packages/server/src/groups/permission.ts`
  - `packages/core/src/session/sql.ts`
  - `packages/core/src/session.ts`
  - `packages/core/src/session/projector.ts`
  - `packages/core/src/event/sql.ts`
  - `packages/core/src/share/sql.ts`
  - `packages/core/src/permission/sql.ts`
  - `packages/opencode/src/session/session.ts`
  - `packages/opencode/src/session/message-v2.ts`
  - `packages/opencode/src/session/revert.ts`
  - `packages/opencode/src/share/session.ts`
- Trace: 本文将 `backlog.md` 中“会话状态、diff、revert 与分享”缺口升级为机制专题。
- Needs: 后续可补一次完整运行观察，确认 legacy 与 v2 API 在默认用户路径中的具体分工。
