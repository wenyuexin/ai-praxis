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

## 二、P0：高优先级缺口

### 2.1 Sandboxing Layers

- **关联目录**：`sandboxing-and-safety/`
- **为什么重要**：当前“sandbox”很容易被简化为 container / VM 选择，但 agent 环境中的 sandbox 至少涉及执行隔离、权限控制、恢复机制和自治约束四个维度。
- **现状**：`overview.md` 已提出三层框架，但还没有专题展开。
- **建议产物**：`sandbox-layers.md`

### 2.2 Permission Policy as First-Class Topic

- **关联目录**：`sandboxing-and-safety/`
- **为什么重要**：agent 环境安全不只是“在哪里执行”，还包括“允许做什么”。若不单列 permission policy，环境文档容易退化为纯 runtime 讨论。
- **现状**：在 overview 中已强调，但尚无独立承接位置。
- **建议产物**：`permission-policy.md`

### 2.3 Workspace / Checkpoint / Traceability 拆分

- **关联目录**：`code-execution-environments/`
- **为什么重要**：workspace 结构、恢复机制和审计能力是不同问题，混写会导致目录边界混乱。
- **现状**：主题已显现，但目录层尚未拆清。
- **建议产物**：`workspace-structure.md`、`workspace-checkpoint.md`、`workspace-traceability.md`

### 2.4 Headless Autonomy vs Interactive Safety

- **关联目录**：`sandboxing-and-safety/`
- **为什么重要**：这是 agent 环境与传统交互式安全模型之间的深层冲突，当前产品和研究界取向差异很大。
- **现状**：已被外部调研明确提出，但主干尚无专题位置。
- **建议产物**：`autonomy-vs-confirmation.md`

---

## 三、P1：值得持续跟踪的专题

### 3.1 Transactional Filesystem Snapshot

- **关联目录**：`code-execution-environments/`、`sandboxing-and-safety/`
- **为什么值得关注**：为 rollback / recovery 提供不同于传统 checkpoint 的路径。
- **当前状态**：有启发，但通用性和成熟度仍需观察。

### 3.2 Workspace Sharing vs Isolation

- **关联目录**：`code-execution-environments/`
- **为什么值得关注**：subtask 是否共享 workspace、是否只读共享、是否完全隔离，是长期会反复出现的环境设计问题。
- **当前状态**：缺少系统化总结。

### 3.3 Browser Safety Beyond Automation

- **关联目录**：`browser-environments/`
- **为什么值得关注**：浏览器环境的风险不只是自动化能力，还包括 prompt injection、会话污染、页面级权限与数据泄露。
- **当前状态**：适合在后续补专题，而非直接写成成熟共识。

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
