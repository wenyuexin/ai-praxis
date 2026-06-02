# Environments Backlog

> 适用范围：`05-environments/` 的内容缺口、前沿问题与待补专题
> 使用原则：本文件记录环境层中已经识别但尚未充分覆盖的高价值问题，不是实现路线图，也不代表相关结论已成为主干定论

## 一、收录规则

适合进入本文件的内容通常满足以下条件之一：

- 能补足执行环境、安全边界、workspace、恢复能力之间的关键桥梁问题
- 代表 agent 环境区别于传统软件 runtime 的重要观察视角
- 当前证据不足以直接写入主干定论，但值得持续跟踪
- 对 `permission layer / execution sandbox / observability` 三层框架有补充价值

不适合进入本文件的内容：

- 纯粹的底层基础设施选型清单
- 具体框架内部实现细节（应放 `../06-frameworks-and-tools/`）
- 只有单一博客或单一产品宣传支撑的结论

---

## 二、P0：当前高优先级缺口

最近一轮已经补上了一批环境层主干专题：

- `sandboxing-and-safety/sandbox-layers.md`
- `sandboxing-and-safety/permission-policy.md`
- `sandboxing-and-safety/permission-vs-execution-boundary.md`
- `sandboxing-and-safety/autonomy-vs-confirmation.md`
- `code-execution-environments/workspace-structure.md`
- `code-execution-environments/workspace-checkpoint.md`
- `code-execution-environments/workspace-traceability.md`
- `code-execution-environments/rollback-recovery-design-paths.md`

因此，backlog 的重点不再是“先把这些专题补出来”，而是收敛为仍缺正文或仍需继续补证的高价值问题。

### 2.1 Environment Boundary Stop-Line

- **关联目录**：`overview.md`、`sandboxing-and-safety/`、`code-execution-environments/`
- **为什么重要**：主干已经形成“permission / execution / observability-recovery / governance”的工作性结构，但 environment 的外延如果没有停线规则，后续很容易把 tool executor、policy engine、evaluation、交互层全部重新混写回来。
- **当前状态**：`conflict.md` 已把 `Environment 是否等于 Execution Container` 收敛为“已有稳定认识 + 剩余边界问题”，但还缺少一篇专门处理 environment 停线规则的正文或综述段落。
- **建议产物**：可先在 `overview.md` 扩一小节，或后续补专门的 boundary 主题。

### 2.2 Workspace Lifecycle Binding / Sharing Model

- **关联目录**：`code-execution-environments/`
- **为什么重要**：workspace 已被稳定定义为 task-specific working context，但它是否与 runtime 生命周期绑定、subtask 如何共享、overlay 是否应作为一等模型，仍是后续工程判断的关键。
- **当前状态**：`workspace-structure.md` 已把问题提出，但尚未单列后续专题系统展开。
- **建议产物**：可继续补 `workspace-lifecycle.md` 或 `workspace-sharing-models.md`。

### 2.3 Traceability Object Model

- **关联目录**：`code-execution-environments/`
- **为什么重要**：`workspace-traceability.md` 已建立 event-first、artifact-centric、task-centric、hybrid 四种视角，但“最小必要对象集是什么、对象之间如何关联”仍未收敛。
- **当前状态**：`conflict.md` 已把它提升为单独冲突条目，但正文还没有专门讨论 object model。
- **建议产物**：可继续补 `traceability-object-model.md` 或在 `workspace-traceability.md` 中扩出专节。

### 2.4 Safety Composition: Policy + Isolation + Recovery

- **关联目录**：`sandboxing-and-safety/`、`overview.md`
- **为什么重要**：现在已经能稳定说明“Docker sandbox 不足以单独定义 Agent 安全”，但不同安全维度最小应如何组合，仍缺一个更成体系的比较框架。
- **当前状态**：相关认识已分散在 `permission-policy.md`、`permission-vs-execution-boundary.md`、`sandbox-layers.md` 与 `conflict.md`。
- **建议产物**：可后续补 `safety-composition.md`，或在 `overview.md` 中增加“最小安全组合”总结段。

---

## 三、P1：值得持续跟踪的专题

### 3.1 Transactional Filesystem Snapshot / Overlay Revert

- **关联目录**：`code-execution-environments/`、`sandboxing-and-safety/`
- **为什么值得关注**：为 rollback / recovery 提供不同于传统 checkpoint restore 的路径，也和 shared baseline + isolated write layer 模式高度相关。
- **当前状态**：`rollback-recovery-design-paths.md` 已把它纳入主流恢复路径之一，但通用性和成熟度仍需继续观察。

### 3.2 Workspace Sharing vs Isolation

- **关联目录**：`code-execution-environments/`
- **为什么值得关注**：subtask 是否共享 workspace、是否只读共享、是否完全隔离，是长期会反复出现的环境设计问题。
- **当前状态**：`workspace-structure.md` 已提出 shared / isolated / hybrid 三类模型，但还缺更系统的长期维护成本比较。

### 3.3 Browser Safety Beyond Automation

- **关联目录**：`browser-environments/`
- **为什么值得关注**：浏览器环境的风险不只是自动化能力，还包括 prompt injection、会话污染、页面级权限与数据泄露。
- **当前状态**：适合在后续补专题，而非直接写成成熟共识。

### 3.4 Default-Safe to Staged Autonomy

- **关联目录**：`sandboxing-and-safety/`
- **为什么值得关注**：permission profile、confirmation gate、execution upgrade 与 rollback-backed autonomy 如何组合，决定系统能否在安全与流畅之间渐进放权。
- **当前状态**：`permission-policy.md` 与 `autonomy-vs-confirmation.md` 已提供基础框架，但还缺更偏策略演进的专题总结。

---

## 四、P2：保留观察线索

### 4.1 MicroVM vs Container 量化比较

- **关联目录**：`code-execution-environments/`
- **说明**：值得长期跟踪，但进入主干前应优先依赖官方文档、工程资料或高质量论文，而不是产品营销对比。

### 4.2 Agent-Specific Security Frameworks

- **关联目录**：`sandboxing-and-safety/`
- **说明**：如 capability control、anti-jailbreak sanitation、dynamic caps 等方向，值得观察其是否形成更稳定的理论与工程规范。

---

## 五、与现有材料的关系

- `overview.md` 已建立 permission / execution / observability 三层理解框架。
- `temp/web-search/2.md` 为本 backlog 提供了多条高价值观察线索。
- `temp/conflict.md` 已承接 Docker safety、workspace 定义、rollback 路径等关键冲突。
- 后续若某条线索得到更充分证据，应拆成专题文档，而不是长期停留在 backlog。
