# Design

本目录是 `docs/design/` 的入口，只回答：**这里有哪些设计侧文档，它们分别解决什么问题。**

当前设计层文件不多，所以这份 README 保持最小，不展开系统设计细节，也不替代具体设计文档本身。

## 当前文件

- `system-design.md`：系统级设计主文，解释仓库为什么被组织成一个 knowledge agent system。
- `capability-design.md`：capability layer 的专项设计文档，说明当前核心能力、承接文档与补强状态。
- `../capabilities/meta.md`：capability layer 自我约束文档，说明能力层的边界、增长方式与联动更新面。
- `contributing-design.md`：`docs/contributing/` 这一层的专项设计文档，解释 rules / methods layer 为什么这样分层。

## 阅读建议

- 想理解这个仓库为什么这样分层：读 [`system-design.md`](./system-design.md)
- 想横向看 capability layer 当前有哪些能力位、由什么承接：读 [`capability-design.md`](./capability-design.md)
- 想理解 capability layer 自己如何守边界、如何增长、变动后要同步哪些面：读 [`../capabilities/meta.md`](../capabilities/meta.md)
- 想理解为什么需要 `docs/contributing/` 这一层，以及它为什么继续拆成主规则、`intent/`、`cases/`：读 [`contributing-design.md`](./contributing-design.md)

## 边界说明

- 本 README 只做设计层入口，不重复系统设计正文。
- 若未来设计层继续增长，再按实际需要决定是否补 `index.md` 或更多专项设计文档。
