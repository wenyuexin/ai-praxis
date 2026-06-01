# 02 Single-Agent

> 适用范围：单智能体能力研究与知识组织
> 阶段状态：调研优先，暂不收敛实现方案

## 一句话定位

本目录用于沉淀单智能体在任务执行中的核心能力模块与机制边界，不绑定某个具体框架实现。

## 快速入口

- 从 `../01-foundations/` 获取基础概念与认知框架。
- 从 `tool-use/`、`planning/`、`memory/` 进入单体能力的核心主题。
- 如遇证据冲突或版本不一致，统一记录到 `../temp/conflict.md`。

## 分类依据

本目录按单智能体执行闭环中的关键能力划分：记忆、规划、推理与行动、工具使用、自我反思，以及典型工作流模式。

## 边界说明

- 放在这里：单智能体内部能力、执行循环、工具使用机制、记忆与反思策略。
- 不放在这里：多智能体协作问题（放 `../03-multi-agent/`）、执行环境与隔离问题（放 `../05-environments/`）、框架/项目案例本身（放 `../06-frameworks-and-tools/`）。

## 同目录导航

- 如后续需要整体综述，使用 `overview.md`。
- 如需要记录内容缺口，使用 `backlog.md`。
- 如需要学习或建设顺序，使用 `roadmap.md`。
- 如发现目录范围内冲突，使用 `conflict.md`。

## 目录结构

```text
02-single-agent/
├── README.md
├── memory/
│   ├── images/
│   ├── long-term-memory/
│   ├── retrieval-methods/
│   └── short-term-memory/
├── patterns/
│   ├── autogpt-pattern/
│   ├── images/
│   ├── ra-aid/
│   └── react/
├── planning/
│   ├── plan-and-execute/
│   ├── task-decomposition/
│   └── tree-of-thoughts/
├── reasoning-and-acting/
│   └── README.md
├── self-reflection/
│   ├── critique-models/
│   └── iterative-refinement/
└── tool-use/
    ├── api-calling/
    ├── code-interpreter/
    ├── mcp/
    └── web-browsing/
```

## 与其他目录的关系

- `../01-foundations/` 提供单智能体研究所需的基础概念与认知框架。
- `../03-multi-agent/` 研究多个智能体如何在单体能力之上分工协作。
- `../05-environments/` 研究执行环境如何约束单体能力边界。
- `../07-evaluation/` 研究如何验证单体机制的有效性与稳定性。
