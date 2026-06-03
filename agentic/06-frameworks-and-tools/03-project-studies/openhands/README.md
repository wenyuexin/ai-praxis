# OpenHands

OpenHands 是一个开源 coding agent / software agent platform。本目录用于集中沉淀 OpenHands 的项目级研究，提供完整阅读体验；其他专题目录只按需要克制回填与其机制直接相关的证据。

## 研究边界

本目录优先回答：

- OpenHands 的项目定位、架构分层与 V0 / V1 演进。
- runtime、sandbox、workspace、conversation / session、agent loop 等核心对象之间的关系。
- 官方文档能直接支撑哪些结论，哪些问题必须进入源码阶段核验。
- OpenHands 对 agentic 工程系统的可迁移设计启发。

不在本目录泛化为所有 coding agent 的通用结论；跨项目比较应进入 `../05-comparisons/`，环境机制的主线归纳应回填到 `../../../../05-environments/` 下对应专题。

## 当前阶段

- **阶段一：官方资料建档**：已开始，先整理官方资料索引、官方口径下的 runtime / sandbox / workspace 关系和待源码核验问题。
- **阶段二：源码研究**：待本地下载 OpenHands 代码后进行，重点核验文档未明确说明的生命周期、持久化、恢复和 traceability 机制。

## 目录结构

```text
openhands/
├── README.md
└── official-docs.md
```

## 阅读入口

- `official-docs.md`：OpenHands 官方资料阶段调研整理，包含资料清单、可用结论、风险点和源码阶段核验清单。

## 与专题目录的关系

- `agentic/05-environments/code-execution-environments/workspace-lifecycle.md`：只回填 OpenHands 与 workspace / runtime / sandbox 生命周期直接相关的证据。
- `agentic/05-environments/candidates.md`：保留 OpenHands 作为环境层补证对象的状态和产出链接。
- `agentic/02-single-agent/`、`agentic/03-multi-agent/`：仅在后续研究确实涉及 agent loop、tool use 或多 agent 机制时做少量证据引用。
