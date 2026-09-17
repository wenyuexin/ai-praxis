# Incident: 渐进式路由未能把新 AI 带到 overview/landscape 的纠正层

- 当前状态：Resolved
- 是否复发：已复发（维护者多次澄清；本次治理讨论中 AI 再次当场把二者压平）
- 记录日期：2026-09-16
- 触发类别：§1 直接观察（表面 one-liner 会误导）+ 复发

## 问题背景

在一次关于元信息体系的讨论中，AI 被要求对比 `overview.md` 与 `landscape.md`。AI 只读了表面规则文档 `metadata-files.md` 就作答，给出“overview 负责主线理解、landscape 负责结构板块”的压平版——正是 `intent/overview-landscape.md` §6 明确列为误读信号的说法。这个误读维护者此前已澄清多次，本次被再次当场复现。

## 冲突点

讲对这个区别的唯一文档是 `intent/overview-landscape.md`（landscape 的对象是“文档体系如何构建”，不是主题内容板块图）。但它在 opt-in 的 intent 层、距表面两跳（metadata-files → intent/metadata-files → overview-landscape），其读取触发写的是“修改条文 / 发现反复误读”，不覆盖“随口解释或对比”这个误读真正发生的时刻。与此同时，`metadata-files.md` 的 landscape 一句话以“这个主题内部有哪些板块”起手，本身就往错误方向暗示，且不带任何“先去读 intent”的陷阱标记。

结果：渐进式披露无法把初次上手的 AI 路由到纠正层——表面层给了一个自洽但错误的答案，AI 就停下了。

## 问题轨迹

1. 表面 `metadata-files.md` 的 landscape 摘要（“主题内部板块、研究地图”）诱导内容结构式误读。
2. 纠正版埋在两跳深的 opt-in intent 层，触发词不含“解释 / 对比”。
3. AI 读到表面即自认为懂，答出压平版，未加载 intent，也不自知在误读。
4. 与 `cases/capability-first-routing-placement.md` 同类：表面顺畅、深层被跳过；区别是此次结果实际错误。

## 解决方案

- `metadata-files.md`（§3 表格、§4.4）：重写 landscape 一句话，明确“对象是文档体系、不是主题内容板块图”，去掉“研究地图”等诱导词；新增“常见误读（高频）”条并直指 `intent/overview-landscape.md`；`overview.md`（§4.3）补“须可独立阅读”。
- `intent/overview-landscape.md`（§8）：把“解释或对比 overview 与 landscape”加入读取触发。
- 可迁移原则：**高频误读点必须在表面层就地设“陷阱标记 + 跳叶子指针”**，不能只把纠正埋在 opt-in 深层——纯渐进式披露对“不自知的误读”必然失效。

## 复发信号

- 再次出现“overview 负责主线、landscape 负责结构，谁先都行”或“landscape 就是 topic / research map”。
- AI 在解释或对比二者时，未先加载 `intent/overview-landscape.md`。

## 关联文件

- `docs/contributing/rules/metadata-files.md`（§3、§4.3、§4.4）
- `docs/contributing/intent/overview-landscape.md`（§6、§8）
- `docs/contributing/cases/capability-first-routing-placement.md`（同类：表面顺畅、深层被跳过）
