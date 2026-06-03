# 04 Human-Agent Interaction

> 适用范围：人与 Agent 之间的任务委托、交互控制与协作体验研究
> 阶段状态：目录骨架已建立，后续持续补充主干内容

## 一句话定位

本目录用于沉淀人在 Agent 系统中的角色、控制权、交互界面、信任建立与接管机制，关注 human-in-the-loop 与更广义的人机协作边界。

## 快速入口

- 从 `../02-single-agent/`、`../03-multi-agent/` 理解 Agent 能力与协作形态，再回看人类如何介入这些执行闭环。
- 从 `human-in-the-loop/`、`delegation-and-control/`、`agent-ui/`、`trust-and-alignment/` 进入核心子主题。
- 若后续出现术语冲突、边界重叠或口径不一致，可再补 `conflict.md`；当前以目录骨架和主题分工为主。

## 分类依据

本目录按人类参与 Agent 系统的关键切面划分：

- `human-in-the-loop/`：人类在规划、执行、审核、纠偏与接管中的介入模式
- `delegation-and-control/`：任务委托、权限约束、自动化程度与控制面设计
- `agent-ui/`：Agent 面向用户的界面、状态呈现与操作入口
- `trust-and-alignment/`：信任建立、解释边界、行为预期与偏好对齐

## 边界说明

- 放在这里：任务委托、人工确认、人工接管、交互体验、信任沟通、控制权边界。
- 不放在这里：单智能体内部能力机制（放 `../02-single-agent/`）、多智能体组织结构（放 `../03-multi-agent/`）、执行环境与隔离（放 `../05-environments/`）、具体框架或产品案例（放 `../06-frameworks-and-tools/`）。

## 同目录导航

- 核心主题入口：`human-in-the-loop/`、`delegation-and-control/`、`agent-ui/`、`trust-and-alignment/`。
- 当前以主题占位和边界拆分为主，后续正文可优先围绕接管模式、自治与确认张力、用户心智模型与交互反馈展开。

## 目录结构

```text
04-human-agent-interaction/
├── agent-ui/
├── delegation-and-control/
├── human-in-the-loop/
└── trust-and-alignment/
```

## 与其他目录的关系

- `../02-single-agent/` 提供人机交互所约束的单体能力背景。
- `../03-multi-agent/` 提供多人/多 Agent 分工协作时的人类控制与介入语境。
- `../05-environments/` 提供权限、沙箱与恢复能力等交互控制的运行时约束。
- `../06-frameworks-and-tools/` 提供真实产品如何落地这些交互模式的案例。
- `../07-evaluation/` 可承接信任、可解释性、人工评估与调试相关的验证问题。
