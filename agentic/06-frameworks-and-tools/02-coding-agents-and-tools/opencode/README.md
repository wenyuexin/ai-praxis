# OpenCode

> 本目录研究 `anomalyco/opencode` 作为开源 AI coding agent 的结构、机制与边界。

## 目录结构

```text
opencode/
└── notes/
```

## 阅读入口

- 读对象全貌：`overview.md`
- 读机制专题：`architecture.md`、`agents-and-permissions.md`、`session-and-server.md`、`context-and-configuration.md`、`model-and-provider-runtime.md`、`ecosystem-and-extensions.md`
- 读产品与工程：`product-surfaces.md`、`engineering-and-operations.md`
- 查辅助材料：`notes/`（含 `source.md`、`evidence.md`、`runtime-validation.md`）

## 研究边界

- 本目录优先研究官方仓库、官方文档和本地源码中可核验的机制，不沿用外部榜单中的 star 数、社区评价或类比标签。
- 与 Claude Code、Codex CLI、Aider 等工具的对比应基于可追溯证据，不先写成印象比较。
- `notes/` 不是正文目录；稳定结论应回流到对象级概览或机制专题，并保留 Evidence / Trace。

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - <https://opencode.ai/docs/zh-cn>
  - <https://github.com/anomalyco/opencode>
  - `notes/source.md`
  - `notes/evidence.md`
- Trace: 用户指定 OpenCode 为重点研究对象并提供本地源码路径后，本目录从候选对象升级为具体案例研究；当前已形成对象概览、机制专题、产品表面、工程运营、backlog 与 notes 辅助材料分层。
- Needs: 后续补运行实证和跨工具横向比较。
