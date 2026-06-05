# Frameworks（智能体框架）

本目录用于沉淀通用 Agent 框架、SDK 与编排框架的资料，重点关注具体框架如何组织 Agent、工具、状态、编排与运行时。

## 目录结构

```text
01-frameworks/
├── autogen/
├── crewai/
├── langchain-agents/
├── langgraph/
└── README.md
```

## 收录范围

适合放入本目录的对象：

- 通用 Agent framework / SDK
- workflow / graph orchestration runtime
- multi-agent orchestration platform
- LM program / prompt optimization 框架中与 Agent 系统构建强相关的对象

当前代表对象：

- `autogen/`
- `crewai/`
- `langchain-agents/`
- `langgraph/`

后续可考虑补充：

- `semantic-kernel/`
- `microsoft-agent-framework/`
- `dspy/`

## 边界

- 框架如何实现 planning、memory、tool-use，可在框架文档中讲。
- planning、memory、tool-use 的通用理论，主归属仍是 `../../02-single-agent/`。
- 多 Agent 协作模式的理论分类，主归属仍是 `../../03-multi-agent/`。
- 本目录重点回答“这个框架如何把能力组织成可用工程系统”。

*最后更新: 2026-06-05*
