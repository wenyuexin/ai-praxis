# Mem0（对象研究）

> 适用范围：Mem0 记忆层的对象级研究
> 版本锚点：论文 `arXiv:2504.19413v1`；Python `v2.2.1`、TypeScript `ts-v3.3.1`，commit `94c3fe9f238f3dbf29c9ce98643bd71eb13077cd`

## 一句话定位

本目录研究对象 Mem0——一个跨会话记忆层，核心接口是 `add()` 与 `search()`，覆盖论文旧版、开源 SDK 与托管 Platform 三代机制。

## 快速入口

- 想先建立整体理解：读 [`overview.md`](./overview.md)。
- 想看机制细节：读 [`consistency-model.md`](./consistency-model.md)（论文、OSS、Platform 三层一致性如何分层）与 [`retrieval.md`](./retrieval.md)（一次 `search()` 如何构造候选、融合打分，以及 hybrid 为何是后端条件能力）。
- 需要来源、证据对照或研究过程：进 [`notes/`](./notes/)；这些是研究辅助材料，服务于 traceability 与后续维护，不是给普通读者的正文。

## 边界说明

- 放在这里：Mem0 的对象本体、源码与版本、公开契约、能力边界与已证实限制。
- 不放在这里：记忆层的共性分类与跨对象比较（放上层 [`../../README.md`](../../README.md) 与 [`../../memory-taxonomy-conflicts.md`](../../memory-taxonomy-conflicts.md)）；其他记忆框架的独立研究（各自建对象目录）。
- 只做只读研究：不调用真实 API、不运行 benchmark、不改上游仓库。
