# AGENTS.md

本文件是 AI 协作者在本仓库工作的入口说明。它不替代 `CONTRIBUTING.md`，只提炼 AI 执行时必须优先遵守的规则。

## 0. 一句话原则

目录结构服务未来写作者的落位判断；未验证材料不写成主线定论。

## 1. 任务路由

开始修改前，先读本文件和 [`CONTRIBUTING.md`](./CONTRIBUTING.md) 了解仓库级强约束，再按当前操作选择最相关的规则文件；不要为了小任务一次性通读所有规则。

- 如果你在**研究论文、开源项目、产品、benchmark 或具体案例**：读 [`research-artifacts.md`](./docs/contributing/research-artifacts.md)
- 如果你在**新建/改名/迁移目录**：读 [`structure-refactoring-rules.md`](./docs/contributing/structure-refactoring-rules.md)
- 如果你在**修改 README 树或导航**：读 [`readme-rules.md`](./docs/contributing/readme-rules.md)
- 如果你在**判断 overview/backlog/candidates/conflict 落位**：读 [`metadata-files.md`](./docs/contributing/metadata-files.md)
- 如果你在**处理 temp / 外部调研 / 回流正文**：读 [`documentation-workflow.md`](./docs/contributing/documentation-workflow.md)
- 如果你在**讨论结构是否长期合理**：读 [`organization-principles.md`](./docs/contributing/organization-principles.md)
- 如果你在**标注 Evidence 或 Trace**：先读 [`evidence-and-traceability.md`](./docs/contributing/evidence-and-traceability.md)，需要细则时再读 [`evidence-rules.md`](./docs/contributing/evidence-rules.md) 和 [`traceability-rules.md`](./docs/contributing/traceability-rules.md)

## 2. 硬约束

- 知识正文放主题目录，治理规则放 `docs/contributing/`，模板放 `docs/templates/`，测试材料放 `docs/test/`；不要把仓库建设约束写进知识正文目录，也不要把具体领域知识写进 `docs/contributing/`。
- 处理新材料时依次判断：材料类型 → Evidence 状态 → 目标落位 → 是否需要 Trace；拿不准时先写元信息文件或保留在 `temp/`，不要抢写正文。细则见 [`documentation-workflow.md`](./docs/contributing/documentation-workflow.md)。
- 外部 AI 调研、网页搜索、博客摘录、issue/PR 讨论只是上游输入，不直接写成主线结论；回流时必须说明 Evidence 状态和 Trace；模型综合判断默认标为 `Inferred` 或 `Unverified`，不得写成已验证事实。细则见 [`evidence-and-traceability.md`](./docs/contributing/evidence-and-traceability.md)。
- README 只做定向和目录导航，不列出元信息文件，目录树只展示目录不展示内容文件。细则见 [`readme-rules.md`](./docs/contributing/readme-rules.md)。
- `candidates.md`、`roadmap.md` 按需创建；`conflict.md` 仅在发现冲突时创建。细则见 [`metadata-files.md`](./docs/contributing/metadata-files.md)。
- 只改与当前任务直接相关的文件；文档重组时先移动再小改；删除前确认内容已迁移或不再需要；不要无任务依据地保留临时反馈文件；不要主动创建新规则文档。细则见 [`structure-refactoring-rules.md`](./docs/contributing/structure-refactoring-rules.md)。
- 如果用户要求与仓库规则冲突，先说明冲突点，交付时说明偏离了哪条规则。

## 3. 自检

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
