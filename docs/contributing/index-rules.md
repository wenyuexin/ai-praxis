# Index 规则

本文件定义仓库中 `index.md` 的职责、触发条件、结构模型和层级导航细则。

相关入口：[`CONTRIBUTING.md`](../../CONTRIBUTING.md) / [`docs/contributing/README.md`](./README.md) / [`docs/contributing/index.md`](./index.md) / [`元信息文件模型`](./metadata-files.md) / [`README 规则`](./readme-rules.md)。如需理解这些规则为什么这样设计、为什么 `README.md` / `index.md` / `roadmap.md` 不能重新混回去，读 [`intent/metadata-files.md`](./intent/metadata-files.md)。

## 1. 使用边界

本文件只回答四件事：

- `index.md` 解决什么问题
- 什么时候该创建 `index.md`
- `index.md` 至少应长成什么样
- `index.md` 与 README、子层 `index.md`、`roadmap.md` 如何分工

`index.md` 在元信息文件模型中的位置、与其他元信息文件的总边界，仍以 [`metadata-files.md`](./metadata-files.md) 为准。

## 2. `index.md` 解决什么问题

`index.md` 负责目录结构展示与查找导航，回答：**“这个目录下面有什么？我已经知道想找什么时该去哪？”**

它通常承接：

- 目录结构展示
- 直接子目录或稳定对象的一句话定位
- 按问题找入口
- 按阅读目标找入口
- 按子目录、对象类型或主题类型找入口

`index.md` 不负责：

- 目录存在理由
- 主题主线解释
- 板块切分治理
- 阅读顺序、学习阶段或推进路径

这些问题分别继续留给 `README.md`、`overview.md`、`landscape.md`、`roadmap.md`。

## 3. 什么时候该创建 `index.md`

默认触发基线是：目录下已有**两个及以上语义稳定、预期会持续保留的子目录**时，应优先考虑创建一个骨架型 `index.md`。

其中“语义稳定”指子目录表达的是长期主题分工，而不是：

- 临时容器
- 过渡迁移位
- 一次性占位目录

即使当前内容仍较薄，只要稳定子结构已经出现，也不要继续让 README 代行目录树。

以下现象通常提示应创建 `index.md`：

- 读者需要先看结构再决定往哪走
- 读者知道自己要找什么，但 README 不能直接定位
- 同一主题横跨多个子目录，难以只靠 README 稳定导航
- 需要按问题、阅读目标或对象类型，而不是只按层级浏览

## 4. `index.md` 的两档模型

### 4.1 骨架型 `index.md`

骨架型 `index.md` 至少包含：

- 目录结构或结构列表
- 直接子目录的一句话定位

当目录刚形成稳定子结构时，先补一个薄的骨架型 `index.md` 通常就足够。

### 4.2 展开型 `index.md`

当查找压力继续上升时，再在骨架型基础上补展开型入口，例如：

- 按问题找入口
- 按阅读目标找入口
- 按对象类型找入口
- 跨子目录的稳定公共锚点

不要为了形式完整，一开始就把每个 `index.md` 写成复杂导航表。

## 5. 父层与子层如何分工

`index.md` 默认遵循“父层粗定位、子层细定位”的分工：

- 父层 `index.md` 主要把读者稳定路由到直接子目录
- 子层 `index.md` 再继续承接更细的文件级、对象级或更深子目录定位
- 父层跨层直链到更深层文件，只应作为少量高频例外

如果父层 `index.md` 的某一节已经主要由孙级或更深层链接构成，通常说明这一层开始替代子层 `index.md`，应优先收回到子目录级路由。

## 6. 与 README 的边界

README 负责目录说明，`index.md` 负责结构展示与查找导航。

因此：

- 不要把 README 里的目录树当作 `index.md` 的默认替代
- 不要因为 README 天然显眼，就把本应由 `index.md` 承接的结构展示折叠回 README
- README 可以用一句入口说明把读者转交给 `index.md`，但不应直接替代 `index.md` 的导航职责

当目录下已有两个及以上语义稳定、预期会持续保留的子目录时，应优先把结构展示迁到骨架型 `index.md`。

## 7. 与 `roadmap.md` 的边界

`index.md` 解决“去哪”，`roadmap.md` 解决“从哪开始、怎么走”。

因此：

- 目录结构展示、查找导航留在 `index.md`
- 顺序、阶段、路径分流、前置依赖留在 `roadmap.md`
- 不要把阅读顺序、学习路径写回 `index.md`
- 也不要把目录结构展示反过来塞进 `roadmap.md`

## 8. 常见误写与 stop-line

- 不要把 `index.md` 写成第二篇 README
- 不要把 `index.md` 写成学习路径文档
- 不要在未达到触发条件时为了形式对称机械补 `index.md`
- 不要让父层 `index.md` 主要由深层文件链接构成
- 不要因为当前内容还薄，就继续让 README 长期代行目录树

## 9. 代表样本

- [`cases/interdisciplinarity-readme-index-roadmap.md`](./cases/interdisciplinarity-readme-index-roadmap.md)：复杂目录中 `README.md`、`index.md`、`roadmap.md` 的拆分样本
- [`intent/metadata-files.md`](./intent/metadata-files.md)：`index.md` 为什么要更早触发、为什么不能退回 README 的设计解释
