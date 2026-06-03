# Mermaid 降级兼容性测试

针对 yarkdown 等旧版 Mermaid 渲染器的降级测试

---

## 测试 1: 最基础语法 (graph TD)
```mermaid
graph TD
    A-->B
```

## 测试 2: 基础带标签
```mermaid
graph TD
    A[Node A]-->B[Node B]
```

## 测试 3: 多条边
```mermaid
graph TD
    A[Start]-->B[Process]
    B-->C[End]
    A-->C
```

## 测试 4: 中文标签
```mermaid
graph TD
    A[开始]-->B[处理]
    B-->C[结束]
```

## 测试 5: 复杂连接
```mermaid
graph TD
    A[Input]-->B[Process 1]
    A-->C[Process 2]
    B-->D[Output]
    C-->D
```

## 测试 6: 带描述的边
```mermaid
graph TD
    A[Start]-->|Step 1|B[Process]
    B-->|Step 2|C[End]
```

## 测试 7: 不同方向
```mermaid
graph LR
    A[Left]-->B[Right]
```

## 测试 8: 简单架构图
```mermaid
graph TD
    Data[Data Source]-->Engine[Engine]
    Engine-->Storage[Storage]
    Engine-->LLM[LLM]
    Storage-->Output[Output]
    LLM-->Output
```

---

## 不兼容语法（避免使用）

以下语法在旧版 Mermaid 中可能报错：

```mermaid
%% 不要使用 flowchart 关键字
%% flowchart TD
%%     A-->B

%% 不要使用 subgraph
%% graph TD
%%     subgraph Group
%%         A-->B
%%     end

%% 不要使用特殊形状
%% graph TD
%%     A([Round])-->B[(DB)]
%%     B-->C{{Diamond}}

%% 不要使用复杂箭头
%% graph TD
%%     A==>B
%%     B-.->C
```

---

## 诊断记录

| 测试编号 | 结果 | 备注 |
|:---------|:-----|:-----|
| 1 | | |
| 2 | | |
| 3 | | |
| 4 | | |
| 5 | | |
| 6 | | |
| 7 | | |
| 8 | | |

