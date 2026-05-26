# Codex 多智能体系统详解

> **专题文档**。本文详解 Manager-Subagent 多智能体系统，包括通信协议、Worktree 隔离和冲突仲裁。适合技术研究者。不属于主线阅读路径。



> 基于 OpenAI 官方文档 + CSDN 架构拆解 + 源码追踪整理（2026.5 最新）。注意：部分 Worktree 和冲突仲裁的细节为社区推断，非源码直读。

---

## 一、Manager-Subagent 架构

### 1.1 角色定义

| 角色 | 职责 |
|------|------|
| **Manager（指挥官）** | ① 将用户请求拆分为子任务（调用 `task_parser.rs` 的 `split_into_concurrent_tasks`） ② 监控所有 Subagent 状态 ③ 冲突仲裁与重调度 ④ 结果聚合 |
| **Subagent（执行者）** | ① 领取单一原子子任务（如"代码语法分析""安全扫描"） ② 在独立沙箱中执行 ③ 将结果通过通道回传 ④ 执行范围受沙箱限制，具体能否自主决定修改范围取决于 Manager 的调度策略 |

从架构设计角度看，Manager 更像一个**调度器+仲裁器**角色，而不是一个"更聪明的 Agent"。这种设计可以避免多 Agent 系统中 Manager 自身产生幻觉的问题。

### 1.2 通信协议

```
┌──────────┐    async_channel (Tokio mpsc)    ┌──────────────┐
│  Manager  │ ◄─────────────────────────────► │  Subagent N  │
│(scheduler)│   TaskChannel<Task>                │(sandboxed)   │
└────┬─────┘                                     └──────────────┘
     │
     │  MCP 协议（跨节点/跨进程时）
     ▼
┌──────────────┐
│  MCP Server  │  (codex-rs/mcp-server/src/lib.rs)
└──────────────┘
```

| 通信场景 | 协议 | 源码位置 |
|----------|------|----------|
| **同一进程内 Manager → Subagent** | `TaskChannel<T>`（Tokio mpsc） | `codex-rs/core/src/communication/async_channel.rs` |
| **跨进程/跨节点** | MCP | `codex-rs/mcp-server/src/mcp_connection_manager.rs` |
| **结果回传** | `oneshot::Sender<Response>` + `Arc<Mutex<HashMap>>` | `codex-rs/mcp-server/src/outgoing_message.rs` |

同一进程内推测使用 mpsc（零拷贝、低延迟），跨节点场景使用 MCP 协议，分层设计比较清晰。

### 1.3 典型工作流 Trace

以用户输入 **"分析这个项目的代码并生成安全报告"** 为例：

```
[User Input]
    │
    ▼
[Manager::parse_task] ──► TaskGraph（3 个子任务）
    │
    ├──► [TaskChannel] ──► Subagent_A: "静态代码分析" (lint)
    │       │                    │
    │       │                    ▼
    │       │              [sandbox::run_in_sandbox]
    │       │                    │
    │       │                    ▼
    │       └── [oneshot] ◄──────┘  Result_A
    │
    ├──► [TaskChannel] ──► Subagent_B: "依赖安全扫描" (security)
    │       │                    │
    │       │                    ▼
    │       │              [sandbox::run_in_sandbox]
    │       │                    │
    │       │                    ▼
    │       └── [oneshot] ◄──────┘  Result_B
    │
    └──► [TaskChannel] ──► Subagent_C: "生成文档" (docgen)
            │                    │
            │                    ▼
            │              [sandbox::run_in_sandbox]
            │                    │
            │                    ▼
            └── [oneshot] ◄──────┘  Result_C
    │
    ▼
[Manager::aggregate_results] ──► merge_results([A, B, C]) ──► FinalReport
```

工作流对应 `task_parser.rs` → `priority_scheduler.rs` → `resource_isolation.rs` → `result_aggregator.rs` 的完整调用链。

---

## 二、Worktree 物理隔离

### 2.1 实现方式

- 每个 Subagent 通过 `git worktree` 创建独立工作目录
- 可绑定不同 git 分支
- 共享 `.git/objects`（不是完整 clone），磁盘开销低
- 就是标准 `git worktree` 命令封装，无自定义魔改



---

## 三、DAG 拓扑排序与分发

### 3.1 TaskGraph → Subagent 映射

```
TaskGraph (DAG)
    │
    ├── Node_0 (root: "分析代码并生成文档")
    │     │
    │     ├── Node_1: "代码语法分析"      ──► Subagent_1
    │     ├── Node_2: "代码结构提取"      ──► Subagent_2
    │     └── Node_3: "文档生成"          ──► Subagent_3
    │           │
    │           └── [依赖 Node_1, Node_2] ──► 等 Node_1+2 完成后触发
```


---

## 四、冲突仲裁

### 4.1 冲突类型

社区报告提到的主要冲突类型包括：文件修改冲突（多个 Subagent 改同一文件）、资源竞争（抢 CPU/内存）。具体的仲裁策略细节官方未公开。

### 4.2 冲突预防

从公开信息看，任务拆分会考虑依赖关系，以减少互斥资源被并发调度的概率。

### 4.3 社区案例

| 来源 | 案例 | 结果 |
|------|------|------|
| Reddit | 3 个 Subagent 同时修改 `package.json`，导致 merge conflict | Manager 检测到冲突 → 暂停 2 个 → 逐个合并 → 成功 |
| CSDN | 电商团队代码检查，lint + security + format 并行 | 效率提升 75%，无冲突 |

从社区分析看，其处理思路更接近"预防—检测—仲裁"的保守策略，但具体实现细节官方未公开。

---

## 五、实际表现

### 5.1 性能数据

社区实测的几组数据表明并行能带来显著加速（如代码质量检查从 4 分半降至 1 分多），但具体加速比因任务类型和资源条件而异。



---

## 六、最佳实践：何时使用多 Agent

| 最适合的任务 | 不适合的任务（应关掉多 Agent） |
|-------------|-------------------------------|
| ✅ **多个独立子任务可并行**（lint + security + format） | ❌ **强顺序依赖**（无并行空间） |
| ✅ **IO 密集型混合 CPU 密集型** | ❌ **单任务已足够快**（调度开销 > 并行收益） |
| ✅ **需要多视角验证**（安全扫描 + 风格检查互为备份） | ❌ **资源极度受限**（内存 < 1GB 会 OOM） |
| ✅ **大规模测试/批量处理**（100 个文件并行检查） | ❌ **需要强一致性事务**（冲突概率极高） |

> **一句话总结**：从架构角度看，Codex 的多智能体设计侧重于"**让并发更安全**"而非"让 AI 更聪明"——Manager 类似于编译器调度，Subagent 类似于执行线程，worktree 提供隔离环境，MCP 处理跨节点通信。

---

## 参考来源

- https://developers.openai.com/codex/app/features
- https://developers.openai.com/codex
- https://blog.csdn.net/gitblog_01166/article/details/157419484
- https://openai.com/index/introducing-upgrades-to-codex/

*最后更新: 2026-05-26*