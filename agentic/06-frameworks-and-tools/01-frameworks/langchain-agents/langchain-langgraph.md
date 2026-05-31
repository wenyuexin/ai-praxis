# LangChain 与 LangGraph

LangChain 和 LangGraph 是构建 LLM 驱动应用的两个核心框架。

---

## 一、LangChain 概述

LangChain 是一个围绕 LLM 构建应用的框架，核心抽象包括：

### 核心组件

| 组件 | 作用 |
|------|------|
| **Model I/O** | 统一调用各类 LLM（OpenAI、Claude、本地模型），支持 Prompt 模板和输出解析 |
| **Chains** | 将多个调用串联为线性 Pipeline，前一步输出作为下一步输入 |
| **Memory** | 维护对话历史，支持滑动窗口、摘要压缩、向量存储等策略 |
| **Tools** | 封装外部能力（搜索、计算、数据库查询等），供 Agent 调用 |
| **Agents** | 根据用户意图自主选择 Tool 并决定调用顺序 |

### Chain 的局限性

```python
# LangChain Chain 是线性 DAG
chain = prompt | model | output_parser
result = chain.invoke({"input": "..."})
```

- **线性执行**：步骤 A → B → C，无法跳过或回退
- **无状态管理**：Chain 之间不共享状态，需要手动传递上下文
- **无条件分支**：所有输入走相同的执行路径
- **适合场景**：固定的预处理/后处理流程、简单的 QA Pipeline

---

## 二、LangGraph 概述

LangGraph 在 LangChain 基础上扩展了**图状编排**能力，核心是 `StateGraph`。

### 核心概念

```
                    ┌──────────┐
                    │   Entry   │
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │   Node A  │
                    └────┬─────┘
                         │ condition
                    ┌────┴────┐
                    ▼         ▼
              ┌─────────┐ ┌─────────┐
              │ Node B   │ │ Node C  │
              └────┬────┘ └────┬────┘
                   │           │
              ┌────▼───────────▼────┐
              │      Node D         │
              └────────┬───────────┘
                       │
                  ┌────▼─────┐
                  │   Exit    │
                  └──────────┘
```

#### 1. StateGraph（状态图）

整个执行流程是一个**有向图**，图中每个节点是一个 Agent 或处理单元，边代表执行顺序。核心是**共享状态**——所有节点读写同一个 State 对象。

```python
from langgraph.graph import StateGraph, END

graph = StateGraph(MyState)

# 添加节点
graph.add_node("diagnose", diagnose_agent)
graph.add_node("treat", treat_agent)
graph.add_node("code", coding_agent)

# 添加边：入口 → 诊断
graph.set_entry_point("diagnose")

# 条件边：诊断 → 治疗 或 结束
graph.add_conditional_edges(
    "diagnose",
    router_function,        # 根据 state 返回下一个节点名
    {"treat": "treat", "end": END}
)

# 普通边：治疗 → 编码
graph.add_edge("treat", "code")
graph.add_edge("code", END)

app = graph.compile()
```

#### 2. Node（节点）

每个节点是一个函数，接收当前 State，返回更新：

```python
def diagnose_agent(state: ClinicalState) -> dict:
    # 读取当前状态（只读）
    symptoms = state["chief_complaint"]
    # 执行推理
    diagnosis = llm.invoke(f"基于 {symptoms} 给出诊断")
    # 返回增量更新
    return {"diagnosis": diagnosis, "current_step": "diagnose"}
```

#### 3. Edge（边）

- **普通边**：无条件连接，A 执行完自动到 B
- **条件边（Conditional Edge）**：根据当前 State 的某个字段决定下一步走向

条件边是 LangGraph 解决 Chain 无法处理循环和条件分支的核心机制。Chain 一旦进入下一步就无法回头；而条件边可以根据当前状态值决定下一步——当诊断置信度低于阈值时，条件边可以回到诊断节点重新推理，而不是机械地进入下游处理节点。这允许：

- **循环**：同一个节点可以被多次访问，直到满足退出条件
- **分支**：不同状态走不同路径，可以跳过某些节点
- **回退**：当某步失败时，回退到前序节点重新执行

#### 4. Reducer（合并函数）

各节点返回的 update 如何合并到全局 State：

```python
from typing import Annotated, TypedDict
from operator import add

class ClinicalState(TypedDict):
    messages: Annotated[list, add]          # append 合并
    diagnosis: str                          # 直接覆盖
    visit_count: Annotated[int, add]        # 累加
```

Reducer 是 LangGraph 状态管理的核心机制。StateGraph 维护一个全局 State 对象（TypedDict），所有节点共享。每个节点函数接收当前 State 的只读快照，返回一个增量更新的字典。更新由 Reducer（合并函数）处理：

- `add` reducer 做列表追加（如 messages）
- 默认（不指定 Annotated）直接覆盖（如 diagnosis）
- 也支持自定义 reducer 实现复杂的合并逻辑

这种方式保证了数据的**不可变性**和**可追溯性**，避免了节点间数据污染。每个节点的输出不会影响其他节点的输入，而是通过 reducer 有序地合并到全局状态中。

---

## 三、LangGraph 关键特性

### 1. 状态回溯与调试

LangGraph 自动保存每一步的 State 快照，支持：

```python
# 查看执行轨迹
for step in app.get_state_history(config):
    print(step.values["current_step"], step.values["diagnosis"])

# 从某一步恢复 / 人工干预
app.update_state(config, {"diagnosis": "修正后的诊断"})
```

状态回溯在排查问题时的典型流程：

1. 导出完整执行轨迹（每个节点的输入/输出/路由决策）
2. 比较正常执行和有问题的路径，定位首次出错的节点
3. 检查该节点的输入上下文（上游节点的输出）是否正确
4. 如果确认是 LLM 输出错误，将该 case 加入微调数据集；如果是状态传递错误，修正 reducer 逻辑
5. 用 `update_state` 在出错节点前注入正确的状态，继续执行验证修复效果

### 2. 并行执行（Send API）

当多个节点互相独立时，可以并行执行：

```python
from langgraph.graph import Send

def continue_to_nodes(state):
    tasks = []
    if state.get("need_coding"):
        tasks.append(Send("coding_agent", {"patient_id": state["patient_id"]}))
    if state.get("need_ddi_check"):
        tasks.append(Send("ddi_agent", {"prescription": state["prescription"]}))
    return tasks

graph.add_conditional_edges("treat", continue_to_nodes)
```

`Send` API 在条件边中将不同的数据分发到不同的节点同时执行。与普通并发（如 Python 的 `asyncio.gather`）不同，`Send` 是在图编排层面实现的——它根据当前状态动态创建并行分支，每个分支独立执行并在后续节点汇合。这种方式与图的执行语义一致，分支执行完成后自动回到图的流程中。

### 3. Interrupt（中断与人工介入）

在关键节点前暂停执行，等待人类确认或修正：

```python
graph.add_node("human_review", review_node)
# 该节点执行完成后暂停
graph.add_edge("diagnose", "human_review")
# 编译时设置中断点
app = graph.compile(interrupt_before=["human_review"])
```

Interrupt 机制在以下场景中尤为关键：

- **高风险决策**：AI 的分析结果需要专家审核确认后才能继续，如在审核节点前设置 interrupt，系统暂停等待，专家可以修正状态后恢复执行
- **人工监督**：多步骤流程中，某些步骤需要人类确认（如费用审批、内容发布）
- **异常处理**：当节点检测到无法自动处理的异常时，中断并等待人工介入

---

## 四、LangChain vs LangGraph 对比

| 维度 | LangChain Chain | LangGraph |
|------|----------------|-----------|
| **编排模型** | 线性 DAG | 有向图（支持循环、条件分支） |
| **状态管理** | 无内置状态，需手动传递 | 内置 StateGraph，自动管理共享状态 |
| **条件路由** | 不支持 | 条件边，根据状态动态选择 |
| **循环/回退** | 不支持 | 支持，可回退到前序节点重新执行 |
| **并行执行** | 不支持 | Send API 分发到多个节点并行 |
| **状态回溯** | 不支持 | 自动保存快照，支持回放和调试 |
| **人工介入** | 不支持 | Interrupt 机制 |
| **复杂度** | 低，上手快 | 较高，适合复杂流程 |
| **适用场景** | 固定流程、简单 Pipeline | 动态决策、多 Agent 协作、需要回退/重试的场景 |

---

## 五、LangGraph Agent Pipeline vs Skill 串流程

Agent Pipeline 与传统的 Skill 串流程在本质上属于不同的抽象层次。

### 对比

| 维度 | LangGraph Agent Pipeline | Skill 串流程 |
|------|--------------------------|-------------|
| **核心区别** | 一个**思考、决策与执行的闭环**，具备自主智能，能动态调整下一步行动 | 一个**线性的、预定义的步骤集**，是**确定性的自动化流程** |
| **决策方式** | **动态决策**。Agent 观察状态，自主判断并选择下一个 Action | **静态编排**。执行顺序由开发者预先硬编码，无法动态调整 |
| **流程结构** | **图结构**，原生支持灵活的**循环、分支和重试** | **链式结构**，通常为线性 DAG，数据和流程单向流动 |
| **代码表示** | `graph.add_node()`, `graph.add_conditional_edges()` | A 函数 → B 函数 → C 函数 |

### 示例对比

以**冲突检测**为例：

**Skill 串联流程**像一套流水线：预先写死 `A → B → C`。A 获取输入数据 → B 调用检测接口 → C 返回结果。无论如何都会调用检测接口，顺序和逻辑是硬编码的。

**LangGraph Agent Pipeline** 则像一个智能团队：
- A 检测输入 → 状态正常 → 结束
- 状态异常 → Agent B 分析和标记 → Agent C 生成处理建议并提醒人工审核
- Agent A 执行后，LangGraph **根据当前最新状态**判断是结束还是调用 B，逻辑是动态的而非线性的。

---

## 六、LangGraph 的调度机制

LangGraph 的调度是一个**循环**，直到任务达成目标：

```
START → [执行节点 → 更新状态 → 条件路由判断] → 循环直到 END
```

1. **逐一执行节点**：从 START 触发，调度器依次执行每个节点
2. **动态更新状态**：每执行完一个节点，更新共享状态（全局信息板）
3. **条件路由与重循环**：根据条件边自动判断，要么 END，要么执行下一个节点，甚至返回到之前的节点形成循环

这种机制使 AI 应用不再是一条**单行道**，而是一个能够自我导航，并在需要时绕路、掉头、重试的智能体。

---

## 七、实战：5-Agent 决策 Pipeline

以 5-Agent Pipeline 为例：`START → 分类Agent → 分析Agent → 处理Agent → 编码Agent → 审计Agent → END`。

对于**"诊断置信度低"**的情况：

1. **决策**：`分析Agent` 处理后在共享状态写入 `{"confidence": 0.3}`
2. **动态路由**：LangGraph 通过条件边检查到低置信度状态，触发预设的动态路由规则
3. **自动循环**：调度器不会走向下一个处理 Agent，而是将流程重新定向回 `分析Agent` 节点
4. **状态回溯**：`分析Agent` 再次执行时，从共享状态读取上一次的分析结果，进行二次确认

此外，LangGraph 也支持同步并行（Send API）、异步外部触发以及人工介入审批（Interrupt）。

### 总结

LangGraph 的应用是对传统 AI 自动化方式的**范式跃迁**。它解锁了构建高复杂度、高自主性 AI 应用的能力，使得 AI 能像项目管理和决策团队一样灵活运转。

---

## 八、共享状态 vs Agent 消息传递

共享状态和消息传递是 Agent 间通信的两种不同范式，核心区别在**数据流向**和**耦合方式**。

### 共享状态（Shared State）

所有 Agent 读写同一个"全局黑板"，每个 Agent 独立发布自己的输出，其他 Agent 按需读取。

```
                   ┌──────────────────┐
                   │   Shared State   │
                   │  (全局黑板/记忆体) │
                   └──┬──┬──┬──┬──┬──┘
                      │  │  │  │  │
              ┌───────┘  │  │  │  └───────┐
              ▼          ▼  ▼  ▼          ▼
           Agent A    Agent B  ...     Agent N
           (写diagnosis)  (读diagnosis)
```

- **数据模型**：一个中心化的 TypedDict / 字典
- **通信方式**：隐式——Agent 写完 state 后，下游 Agent 自动看到新值
- **耦合度**：低耦合——Agent 不需要知道谁消费自己的输出
- **典型实现**：LangGraph StateGraph、Redux 模式
- **优点**：天然支持状态回溯、调试方便、不需要点对点连接
- **缺点**：并发写需要 reducer 解决冲突；state 结构变更影响所有 Agent

### Agent 消息传递（Message Passing）

Agent 之间通过显式的消息通道（异步队列、事件总线、点对点调用）直接发送和接收消息。

```
Agent A ──(send message)──→ Agent B ──(send message)──→ Agent C
                              ↑
                              │
Agent D ──(send message)──────┘
```

- **数据模型**：消息对象（包含 sender、receiver、payload、message_id 等字段）
- **通信方式**：显式——A 明确知道要发给谁（或通过主题订阅）
- **耦合度**：中等到高耦合——发送方可能需要知道接收方的身份或消息格式
- **典型实现**：AutoGen GroupChat、Kafka/RabbitMQ、RPC 调用
- **优点**：天然支持异步和并发、松耦合事件驱动架构、可扩展性强
- **缺点**：调用链追踪复杂、全局状态需要额外聚合、实现更重

### 对比总结

| 维度 | 共享状态 | Agent 消息传递 |
|------|---------|---------------|
| **数据位置** | 全局唯一 | 分散在各消息中 |
| **Agent 是否需感知对方** | 不需要 | 需要（或通过订阅机制） |
| **并发安全** | 需 Reducer 合并 | 天然隔离（异步队列） |
| **可观测性** | 只需 dump state | 需分布式追踪 |
| **状态回溯** | 天然支持（每次快照） | 需要额外实现事件溯源 |
| **扩展性** | 单点瓶颈风险 | 天然可水平扩展 |
| **何时选用** | 流程内多个 Agent 协作完成一个任务 | 跨系统、跨服务的事件驱动通知 |

### 一句话总结

- **共享状态** = 多个 Agent 在一个房间对着同一块白板写写画画——协作紧密，回溯简单
- **消息传递** = 每个 Agent 在独立隔间里工作，通过小纸条传递信息——松耦合，扩展方便

LangGraph 选共享状态，AutoGen GroupChat 选消息传递，两者都是合理的设计，取决于你需要的是**紧协作的工作流**还是**松耦合的事件系统**。

---

*最后更新: 2026-05-11*