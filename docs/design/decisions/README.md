# Design Decisions

本目录存放**一次性设计决策记录**（ADR 式，Any Decision Record）：某个时点面临一个选择、比较了哪些备选、为什么择其一、依据是什么。

## 与 `design/` 顶层的区别

- **`design/` 顶层 = 系统架构文档**：描述“这套系统怎么运作”，随系统长期演进，没有“完成”态（system-design、navigation-design、evolution-design、runtime-development…）。
- **本目录 = 决策记录**：记录一次具体选择，有生命周期（开放 → 拍板 → 归档）；拍板后基本是历史，不随系统演进改写。

**判据（一篇该放哪）**：它是在**描述系统怎么运作**（→ `design/` 顶层），还是在**记录一次选择**（→ 这里）？

## 与治理记录家族的关系

`../../contributing/cases/`（复发模式）、`../../contributing/incidents/`（单次问题）是**治理记录**，服务 `meta-rules`；本目录是**设计记录**，记录设计 / 产品选择。两类记录各自贴近它服务的父概念，不混。

## 当前决策

- [`repo-naming/`](./repo-naming/)：为本仓库选名（当前 `ai-praxis`）——判断框架 + 候选短名单 + GitHub 命中量证据。

## 放什么 / 不放什么

- **放**：一次性、点时刻的设计 / 产品 / 命名 / 格式选择，带备选与理由。
- **不放**：系统架构论证（→ `design/` 顶层）；已确认复发的治理模式（→ `../../contributing/cases/`）；单次治理问题（→ `../../contributing/incidents/`）；纯发散过程稿（留各决策自己的 `reference/` 或 `temp/`）。
