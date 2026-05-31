# Codex 架构与源码分析

> **专题文档**。本文分析 Codex 开源仓库（Rust 48+ crate）的源码架构。适合技术研究者深入阅读。注意：部分内容为社区推断，不确定处已标注证据等级。



> 基于 Codex 开源仓库（codex31/codex，Rust，48+ crate）+ CSDN 架构拆解文章整理。
> 源码镜像：https://gitcode.com/GitHub_Trending/codex31/codex
> 注意：部分细节依赖社区二次解读，已在文中标注。

---

## 一、项目整体结构

### 1.1 48+ crate 组织

| 层级 | Crate | 职责 | 依赖方向 |
|------|-------|------|----------|
| **核心引擎层** | `codex-rs/core` | 任务解析、调度、沙箱、通信、聚合 — 5 大并发组件 | 被上层所有 crate 依赖 |
| **协议层** | `codex-rs/mcp-server` | MCP Server 实现、stdin/stdout/消息处理 | → core |
| **CLI 入口** | `codex-rs/cli` | 命令行交互、Computer Use 驱动 | → core, → mcp-server |
| **工具扩展** | `codex-rs/tools/` | 内置工具集（lint、security scan、format 等） | → core |
| **沙箱实现** | `codex-rs/core/src/sandboxing/` | `resource_isolation.rs` + `sandbox.rs` | 独立（条件编译） |
| **通信层** | `codex-rs/core/src/communication/` | `async_channel.rs` + Tokio mpsc 封装 | → core |
| **调度器** | `codex-rs/core/src/scheduler/` | `priority_scheduler.rs` | → core |

**核心边界**：`codex-rs/core/src/` 是内核，`cli`、`tools/`、`mcp-server` 是消费者。

### 1.2 Cargo 依赖关系（推断）

```
cli ──→ core ──→ scheduler
         ├──→ sandboxing
         ├──→ communication
         └──→ aggregator
mcp-server ──→ core
tools/* ──→ core
```

### 1.3 构建命令

| 命令 | 作用 |
|------|------|
| `just build` | 编译全部 crate |
| `just test` | 运行全部单元测试 |
| `just lint` | Clippy 静态检查 |
| `just format` | rustfmt 格式化 |
| `just check` | build + test + lint 一条龙 |

---

## 二、任务解析与拆分器

**文件**：`codex-rs/core/src/tasks/task_parser.rs`

### 2.1 核心流程

```rust
pub fn parse_task(request: &str) -> Result<TaskGraph> {
    let mut parser = TaskParser::new(request);
    let root_task = parser.parse()?;          // 递归下降解析
    let task_graph = split_into_concurrent_tasks(root_task); // DAG 拆分
    Ok(task_graph)
}
```

1. `TaskParser::new(request)` 创建解析器
2. `parser.parse()` 递归下降，将自然语言分解为 AST
3. `split_into_concurrent_tasks()` 基于依赖关系拆分为可并行子任务

### 2.2 TaskGraph 数据结构（推断）

```rust
pub struct TaskGraph {
    nodes: Vec<TaskNode>,      // 任务原子单元
    edges: Vec<TaskEdge>,      // 依赖关系（DAG）
}
pub struct TaskNode {
    id: TaskId,
    task_type: TaskType,       // IO密集 / CPU密集
    resource_req: ResourceReq, // CPU/内存需求
    priority: u8,
}
pub enum TaskEdge {
    DependsOn(TaskId),         // 必须先完成
    Produces(TaskId),          // 产出供下游消费
}
```

### 2.3 拆分策略

> "根据依赖关系和资源需求拆分任务。例如'分析代码并生成文档'拆分为'代码语法分析'、'代码结构提取'、'文档生成'三个可并行子任务" — CSDN

社区拆解认为这种基于语义理解的自动拆分是 Codex 的核心竞争力之一，比固定模板更灵活。

---

## 三、优先级调度器

**文件**：`codex-rs/core/src/scheduler/priority_scheduler.rs`

### 3.1 多级反馈队列

```rust
async fn schedule_tasks(&mut self) {
    while let Some(task) = self.select_next_task() {
        let priority = self.calculate_priority(&task);
        self.worker_pool.assign_task(task, priority).await;
    }
}
```

**以下队列设计为推断**（基于 CSDN 描述 + MLFQ 标准实现，非源码直读）：

| 队列 | 优先级 | 任务类型 | 升降级规则 |
|------|--------|----------|-----------|
| Q0 | 最高 | 实时交互（UI 渲染、用户输入） | 用完时间片 → Q1 |
| Q1 | 高 | IO 密集型（文件搜索、网络请求） | 用完时间片 → Q2 |
| Q2 | 中 | CPU 密集型（代码分析、lint） | 长期未调度 → 提升到 Q1（防饥饿） |
| Q3 | 低 | 后台任务（缓存清理、日志归档） | — |

### 3.2 IO vs CPU 分离

> "文件搜索等 IO 密集型任务会被分配到独立的线程池，避免阻塞 CPU 密集型的代码分析任务" — CSDN

社区分析指出这是"分离 IO/CPU 线程池"模式的典型应用，目的是避免 IO 等待阻塞 CPU 任务。

### 3.3 系统负载感知

CSDN 提到"动态调整执行顺序"，但具体机制未公开（可能基于 `std::thread::available_parallelism()`）。

### 3.4 任务抢占

源码未显式实现抢占，但超时机制（见第 6 节）本质上是一种"软抢占"。

---

## 四、资源隔离沙箱

**文件**：`codex-rs/core/src/sandboxing/resource_isolation.rs` + `sandbox.rs`

### 4.1 沙箱创建

```rust
pub async fn run_in_sandbox<F, R>(task: F) -> Result<R>
where F: FnOnce() -> R + Send + 'static, R: Send + 'static {
    let sandbox = Sandbox::new()
        .with_cpu_limit(1)
        .with_memory_limit(512);   // CPU limit=1, Memory limit=512MB
    sandbox.execute(task).await
}
```

### 4.2 三平台实现

以下实现细节基于社区拆解（可靠度 B/C），非源码直读：

| 平台 | 实现方式（推断） |
|------|----------|
| **Linux** | `namespaces`（CLONE_NEWNS \| CLONE_NEWPID \| CLONE_NEWNET）+ `seccomp-bpf` + Landlock（Linux 5.13+） |
| **macOS** | `sandbox_init()` + Seatbelt 策略文件 |
| **Windows** | `CreateProcessW` + Job Objects + ACL |

三平台条件编译是 Rust 跨平台沙箱的标准做法，Landlock 是 Linux 最新、最安全的方案。

### 4.3 启动 Overhead（推测）

Linux namespaces 启动通常 10-50ms。

### 4.4 网络策略

社区分析认为默认禁网 + 白名单是企业级沙箱的标准做法。Cloud 沙箱中 Setup 阶段可访问网络，Agent 阶段默认关闭。

---

## 五、异步通信层

**文件**：`codex-rs/core/src/communication/async_channel.rs`

### 5.1 Tokio mpsc 封装

```rust
pub struct TaskChannel<T> {
    sender: mpsc::Sender<T>,
    receiver: mpsc::Receiver<T>,
}
impl<T: Send + 'static> TaskChannel<T> {
    pub fn new(capacity: usize) -> (Self, Self) {
        let (tx, rx) = mpsc::channel(capacity);
        (TaskChannel { sender: tx.clone(), receiver: rx },
         TaskChannel { sender: tx, receiver: rx })
    }
}
```

### 5.2 消息类型

| 消息类型 | 用途 |
|----------|------|
| `TaskRequest` | 调度器 → 执行器 |
| `TaskResult` | 执行器 → 聚合器 |
| `TaskCancel` | 超时/用户取消 |
| `TaskStatus` | 进度更新（running/done/failed） |

### 5.3 流式传输 & 背压

> "代码分析任务的结果可以实时流式传输到文档生成任务，而无需等待整个分析过程完成" — CSDN

Tokio mpsc 的 channel 机制在满时 sender 会阻塞，这天然提供了背压能力。

---

## 六、结果聚合器

**文件**：`codex-rs/core/src/aggregator/result_aggregator.rs`

### 6.1 聚合逻辑

```rust
pub async fn aggregate_results(&mut self) -> Result<FinalResult> {
    let mut results = Vec::new();
    while let Some(result) = self.result_receiver.recv().await {
        results.push(result);
        if results.len() == self.expected_count {
            break;
        }
    }
    self.merge_results(results)
}
```

### 6.2 重试策略（推断）

| 参数 | 推断值 | 
|------|--------|
| 重试次数 | 3 次（Rust 生态标准） |
| 间隔 | 指数退避（100ms → 500ms → 2s） |
| 放弃条件 | 3 次全失败 OR 超过 total_timeout |

### 6.3 结果补偿

CSDN 提到"结果补偿策略"但未展开。推测含义：部分任务失败时用默认值/降级输出填充，而非整个任务失败。这种"尽力而为"聚合比"全有或全无"更适合并发场景。

---

## 七、工具扩展机制（MCP）

### 7.1 MCP Server 入口

```rust
#[tokio::main]
async fn main() -> Result<()> {
    let stdin_handle = tokio::spawn(stdin_processor.run());
    let processor_handle = tokio::spawn(message_processor.run());
    let stdout_handle = tokio::spawn(stdout_processor.run());
    tokio::join!(stdin_handle, processor_handle, stdout_handle);
    Ok(())
}
```

从公开代码看，MCP Server 同时运行三个异步任务：stdin 输入处理、消息处理、stdout 输出。

### 7.2 协议实现

MCP 协议基于 JSON-RPC 2.0，Codex 原生支持而非后装。自定义 Server 通过 `message_processor` 动态加载。

---

## 八、构建与发布

| 项目 | 说明 |
|------|------|
| CI/CD | `.github/workflows/` 存在（GitCode 确认） |
| 跨平台 | x86_64 + aarch64, Linux/macOS/Windows（Rust 默认支持） |
| 发布节奏 | 推测每月一次 minor + 按需 patch |

---

## 九、架构总评

| 维度 | 评分 | 一句话 |
|------|------|--------|
| **模块化** | ⭐⭐⭐⭐⭐ | core crate 是真正的内核，上层全是消费者，依赖方向清晰 |
| **并发模型** | ⭐⭐⭐⭐☆ | Tokio + MLFQ 调度 + 分离 IO/CPU 线程池。设计成熟，但部分细节为社区推断 |
| **安全性** | ⭐⭐⭐⭐☆ | Landlock+Seccomp 是 Linux 最佳实践，三平台条件编译覆盖广 |
| **可扩展性** | ⭐⭐⭐⭐⭐ | MCP 协议 + 插件式工具注册，扩展成本极低 |
| **源码文档** | ⭐⭐☆☆☆ | 源码注释偏少，社区二次解读是当前主要信息源 |

> **一句话总结**：从社区分析和公开代码看，Codex 的 Rust 架构大致可概括为 **"Tokio 运行时 + MLFQ 调度器 + Landlock 沙箱 + MCP 协议"** 的组合。每一层都采用了 Rust 生态中成熟的技术方案，但部分实现细节仍待源码进一步确认。

---

## 参考来源

- https://gitcode.com/GitHub_Trending/codex31/codex
- https://blog.csdn.net/gitblog_01166/article/details/157419484
- https://developers.openai.com/codex
- https://openai.com/index/introducing-upgrades-to-codex/

*最后更新: 2026-05-26*