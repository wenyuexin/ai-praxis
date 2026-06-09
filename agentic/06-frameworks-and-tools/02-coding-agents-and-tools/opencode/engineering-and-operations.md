# OpenCode Engineering and Operations

> 本文整理 OpenCode 作为长期维护开源项目的工程成熟度：测试、CI、benchmark、迁移、observability、release、开发工作流与运行维护。

## 核心判断

OpenCode 的研究价值不只在 agent runtime。它已经具备较完整的工程化和运营支撑：包级测试约束、Linux/Windows CI、E2E、数据库迁移、性能 benchmark、OpenTelemetry、Sentry/Honeycomb、stats pipeline、release signing、desktop matrix build 等。

这说明它不是“概念验证式 coding agent”，而是一个快速演进但已经有工程治理能力的开源产品。

## 测试体系

根目录 test 被有意阻止，要求从 package 目录运行测试。这和仓库指导一致，避免误跑全仓库或绕过 package-local 环境。

`packages/opencode/test/` 覆盖面很广，包括：

- server HTTP API。
- session / storage / migration。
- plugin / provider / config。
- CLI / TUI / ACP。
- LSP / tools / permissions。
- project / worktree。

测试辅助材料也有明确约束，例如临时目录、Effect-aware fixtures、避免 fixed sleeps、使用 readiness signals 等。这说明测试并非只覆盖 happy path，而是在处理异步、Effect runtime 和集成边界。

## CI 与类型检查

CI 覆盖 Linux 和 Windows，输出 JUnit test reports。App 层还有 Playwright E2E，覆盖 web/app product surface。

Typecheck 有独立 workflow；root scripts 通过 turbo 做 workspace orchestration。仓库明确要求不要从 root 直接运行测试，也要求 package-local typecheck。

这些规则降低了 monorepo 中“看似跑了测试，实际上下文不对”的风险。

## Benchmark 与性能治理

OpenCode 有 test-suite benchmark 文档和脚本，记录 test suite speed、primary metric、历史慢文件和优化决策。脚本会统计 median、mean、best、worst，并有 per-file profiler 帮助定位慢测试。

这类 benchmark 对 coding agent 项目很重要：agent runtime、server API、TUI 和 integration tests 很容易拖慢反馈循环，性能治理决定长期维护成本。

## 数据库与迁移

Core SQLite DB 初始化时设置 durability pragmas，并有自定义迁移 journal。迁移逻辑支持跳过已完成 migration、transaction 包裹、兼容 legacy Drizzle journal、检测未生成或 stale registry 的 migration。

同时，旧 JSON storage 也有迁移逻辑，覆盖 legacy project/session/message/part layouts、session diff extraction、marker 写入和锁机制。

这说明 OpenCode 在 session/storage 演进中不是一次性重写，而是保留迁移路径和兼容层。

## Observability 与 Telemetry

OpenTelemetry 是可配置的，支持 logs/traces，并带 service version、deployment channel、client、run ID、service instance ID 等资源属性。

LLM 调用路径接入 AI SDK telemetry，session ID 会进入 spans。Web/app 层有 Sentry。Infra 中还配置 Honeycomb alerting、Discord webhook、model/provider error 和 TPS 相关监控。

这说明 OpenCode 已经把模型调用、前端错误和服务运行状态纳入观测体系。

## Stats Pipeline

Stats package 有独立 Drizzle/Planetscale migration workflow 和 infra。Stats sync 作为单副本服务运行，定期从 Athena 聚合数据并 upsert model/provider/geo rows。

这部分显示 OpenCode 不只记录本地运行，还在构建产品级使用统计和分析链路。

## Release 与分发治理

Release pipeline 覆盖 CLI build、Windows CLI signing、signature verification、Electron matrix build、Electron signature verification 等。关键 workflows 有 concurrency control，部分 actions pin 到 SHA，提升供应链安全性。

结合 README 中多平台安装方式和 desktop assets，OpenCode 的开源分发已经进入产品化阶段。

## 开发工作流

Root dev scripts 覆盖 CLI/TUI、desktop、web、console、stats、storybook、lint、typecheck。CONTRIBUTING 文档说明本地 setup、server modes、debugger 和 SDK/API generation。

这对研究者也很重要：OpenCode 的源码不是只靠 README 理解，需要结合 package scripts、CI、AGENTS、CONTRIBUTING 和 specs 才能判断当前实现与迁移方向。

## 限制

- 工程成熟度并不均匀，重点集中在 `packages/opencode`、`packages/core`、`packages/app`、`packages/stats` 等核心包。
- 当前未实际运行测试或 CI，只基于源码、配置和 workflow 做静态核验。
- 快速演进项目的 workflow 和 migration 可能变化较快，需要结合具体 commit / release 重新核验。

## Evidence

- Status: `Observed / Inferred`
- Version Basis: <https://github.com/anomalyco/opencode>, local checkout observed on 2026-06-09.
- Sources:
  - `package.json`
  - `AGENTS.md`
  - `CONTRIBUTING.md`
  - `turbo.json`
  - `.github/workflows/test.yml`
  - `.github/workflows/typecheck.yml`
  - `.github/workflows/deploy.yml`
  - `.github/workflows/publish.yml`
  - `packages/opencode/package.json`
  - `packages/opencode/test/`
  - `packages/opencode/test/AGENTS.md`
  - `packages/opencode/test/EFFECT_TEST_MIGRATION.md`
  - `perf/test-suite.md`
  - `packages/opencode/script/bench-test-suite.ts`
  - `packages/opencode/script/profile-test-files.ts`
  - `packages/core/src/database/database.ts`
  - `packages/core/src/database/migration.ts`
  - `packages/core/script/migration.ts`
  - `packages/opencode/src/storage/storage.ts`
  - `packages/core/src/observability/otlp.ts`
  - `packages/opencode/src/session/llm.ts`
  - `packages/app/src/entry.tsx`
  - `infra/monitoring.ts`
  - `infra/stats.ts`
  - `packages/stats/core/src/stat-sync.ts`
- Trace: 本文补足前一版 OpenCode 案例中遗漏的工程成熟度层，避免只从 runtime 和产品功能解释该开源框架。
- Needs: 后续可实际运行 package-local tests、typecheck 和 benchmark，补充动态验证结果。
