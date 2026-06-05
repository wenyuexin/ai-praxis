# LangGraph Notes

本目录用于承接 LangGraph framework study 的研究辅助材料，服务于源码核验、证据整理、失败搜索与后续维护，不替代 `langgraph/` 根目录下的对象总览和机制专题正文。

## 使用边界

本目录适合放：

- 官方文档长引用
- 源码核验摘录
- 失败搜索与版本冲突
- Claim / Source / Status 对照表
- 尚未整理进正文的研究过程材料

本目录不负责：

- 替代 `overview.md`、`architecture.md`、`checkpoint-and-persistence.md` 等正文专题
- 直接给出未核验的主线定论
- 作为 LangGraph 的主要阅读入口

## 当前结构

```text
notes/
├── README.md
├── source.md
└── evidence.md
```

## 使用约定

- `source.md`：记录源码入口、官方文档长摘录、失败搜索和局部核验。
- `evidence.md`：整理已形成的 Claim / Status / Sources / Needs，对接正文中的 Evidence 段落。
- 如果后续某个机制专题（如 checkpoint、runtime、store）材料过多，再按机制单独拆分 `checkpoint.md`、`runtime.md` 等 notes 文件。

## 当前阶段

- 目前只建立最小辅助层，方便后续 LangGraph 官方 docs / 源码核验时有稳定落点。
- 在证据尚少之前，不主动继续扩 notes 文件数量。