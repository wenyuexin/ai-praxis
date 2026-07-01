# AGENTS.md

本文件是 AI 协作者在本仓库工作的入口说明。它不替代 `CONTRIBUTING.md`，只保留执行时必须优先遵守的规则。首次进入本仓库时，必须先读完本文件，再立即阅读 [`CONTRIBUTING.md`](./CONTRIBUTING.md)；未完成这两步前，不要响应任何仓库相关的分析、修改或结构建议。

## 1. 一句话原则

目录结构要让人一眼看出内容该放在哪里；未验证材料不要写成主线定论。

## 2. 任务路由

完成 `AGENTS.md` 与 [`CONTRIBUTING.md`](./CONTRIBUTING.md) 的阅读后，再按当前操作选择最相关的能力文档与规则文件；不要为了小任务一次性通读所有规则。如果你想先理解规则层整体分工，读 [`docs/contributing/README.md`](./docs/contributing/README.md)；如果已经进入规则层但拿不准下一步该读哪篇，读 [`docs/contributing/index.md`](./docs/contributing/index.md)。如果你想先理解这套系统提供哪些公共能力，读 [`docs/capabilities/README.md`](./docs/capabilities/README.md)；如果你要判断 capability layer 自己的边界、增长方式与联动更新面，读 [`docs/capabilities/meta.md`](./docs/capabilities/meta.md)；如果已经进入能力层但拿不准下一步该读哪篇，读 [`docs/capabilities/index.md`](./docs/capabilities/index.md)。

### 2.1 先判断能力类型

在进入具体规则文件前，先判断当前任务主要属于哪类公共能力：

- 如果你在**从问题、关键词或对象名出发，想找到最可能的目录入口、目标区域或候选支路**：先读 [`docs/capabilities/navigate.md`](./docs/capabilities/navigate.md)
- 如果你在**已经找到大致区域后，想判断应停在哪一层、应写正文还是哪类元信息文件**：先读 [`docs/capabilities/place.md`](./docs/capabilities/place.md)
- 如果你在**处理新材料、外部调研、博客摘录、网页搜索结果或临时笔记，想判断今天先落到哪里、能否进入正文**：先读 [`docs/capabilities/ingest.md`](./docs/capabilities/ingest.md)

能力文档负责回答“这套系统怎么找、怎么放、怎么吸收”；具体 stop-line、文件职责与执行细则仍回到 `docs/contributing/`。如果你不确定该先走哪条路由，默认先走本节的能力路由，再回到下面的规则路由。

- 如果你在**研究论文、开源项目、产品、benchmark 或具体案例**：读 [`research-artifacts.md`](./docs/contributing/research-artifacts.md)
- 如果你在**新建/改名/迁移目录**：读 [`structure-refactoring-rules.md`](./docs/contributing/structure-refactoring-rules.md)
- 如果你在**判断 `README.md` / `index.md` / `overview.md` / `landscape.md` / `backlog.md` / `candidates.md` / `conflict.md` 该如何分工或落位**：读 [`metadata-files.md`](./docs/contributing/metadata-files.md)
- 如果你在**修改 README 本身的写法、出现条件、入口说明或导航边界**：读 [`readme-rules.md`](./docs/contributing/readme-rules.md)
- 如果你在**修改 `index.md` 本身的结构、粒度、父子层导航方式或 stop-line**：读 [`index-rules.md`](./docs/contributing/index-rules.md)
- 如果你在**修改目录树、查找导航，或判断结构展示是否应从 README 迁到 `index.md`**：先读 [`metadata-files.md`](./docs/contributing/metadata-files.md)；涉及 `index.md` 的专项细则时继续读 [`index-rules.md`](./docs/contributing/index-rules.md)；README 写法问题再按需读 [`readme-rules.md`](./docs/contributing/readme-rules.md)
- 如果你在**处理 temp / 外部调研 / 回流正文**：读 [`documentation-workflow.md`](./docs/contributing/documentation-workflow.md)
- 如果你在**讨论结构是否长期合理**：读 [`organization-principles.md`](./docs/contributing/organization-principles.md)
- 如果你在**标注 Evidence 或 Trace**：先读 [`evidence-and-traceability.md`](./docs/contributing/evidence-and-traceability.md)；需要细则时，再读 [`evidence-rules.md`](./docs/contributing/evidence-rules.md) 和 [`traceability-rules.md`](./docs/contributing/traceability-rules.md)

## 3. 硬约束

- 知识正文放主题目录，治理规则放 `docs/contributing/`，模板放 `docs/templates/`，测试材料放 `docs/test/`；不要把仓库建设约束写进知识正文目录，也不要把具体领域知识写进 `docs/contributing/`。
- 处理新材料时，按这个顺序判断：材料类型 → Evidence 状态 → 目标落位 → 是否需要 Trace；拿不准时先写元信息文件或保留在 `temp/`，不要抢写正文。细则见 [`documentation-workflow.md`](./docs/contributing/documentation-workflow.md)。
- 外部 AI 调研、网页搜索、博客摘录、issue/PR 讨论都只是上游输入，不要直接写成主线结论；回流时必须说明 Evidence 状态和 Trace；模型综合判断默认标为 `Inferred` 或 `Unverified`，不要写成已验证事实。细则见 [`evidence-and-traceability.md`](./docs/contributing/evidence-and-traceability.md)。
- README 只做目录说明与最小入口分流，不列出元信息文件；目录结构与查找导航优先由 `index.md` 承接。细则见 [`readme-rules.md`](./docs/contributing/readme-rules.md)。
- `candidates.md`、`roadmap.md` 按需创建；`conflict.md` 仅在发现冲突时创建。细则见 [`metadata-files.md`](./docs/contributing/metadata-files.md)。
- 只改与当前任务直接相关的文件；文档重组时先移动，再小改；删除前先确认内容已迁移或确实不再需要；不要无任务依据地保留临时反馈文件；不要主动创建新规则文档。细则见 [`structure-refactoring-rules.md`](./docs/contributing/structure-refactoring-rules.md)。
- 如果用户要求和仓库规则冲突，先说明冲突点；交付时再说明偏离了哪条规则。

## 4. 自检

提交前至少检查：

- 链接是否仍然有效
- 是否残留旧路径或旧术语（如"证据强度"、`Unverified / Unreliable`、`主干正文`、`回流主干`）
- 新增 Markdown 文件是否使用 LF 行尾
- README 是否违反"不列出元信息文件"的规则
- 是否把未验证材料写成了主线定论

如果产出的内容符合以下任何特征，请暂停并修正：

- 没有来源的断言，读起来像教科书定论，但实际只是模型综合判断
- overview.md 中出现"目前暂无内容"或"当前内容较少"这类用空内容证明结构合理的表述
- 把 AI 搜索结果、博客摘录或 issue 讨论直接写成了主线结论
- README 目录树中 images/、papers/、notes/ 与知识主题并列展示
- 空目录下唯一的文件是 README，且没有写出"什么应放这里、什么不放这里"
