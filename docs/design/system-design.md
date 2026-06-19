# Repository Knowledge Agent Design

本文从系统设计视角解释：这个仓库不只是知识内容的集合，也不只是规则文件的堆叠，而是一个由 **目录结构、`README.md` / `index.md`、元信息文件、规则文档，以及 AI 协作者** 共同组成的知识型 agent system。

它回答的不是具体某条规则怎么执行，而是：**这套系统提供什么能力、各部分为什么这样分层、以及哪些文档属于能力说明，哪些文档属于维护规则。**

## 1. 系统目标

这套系统至少服务四类核心目标：

- **积累知识**：把论文、开源项目、产品、benchmark、跨学科材料逐步沉淀成长期可维护的知识内容
- **管理知识**：让内容能稳定分层，避免未验证材料、候选对象、缺口问题和正式正文混在一起
- **搜索知识**：让人类读者与 AI 协作者都能从问题出发，沿着目录树快速定位到最可能的落位点
- **回答问题**：在已有知识基础上恢复上下文、解释边界、回答问题，并区分稳定结论、观察、推断与未验证材料

因此，这个仓库中的目录、入口文件和规则文件，不只是静态文档，也共同构成了一套运行中的知识系统。

## 2. 系统由什么组成

### 2.0 当前文档系统树

从 agent system 视角看，当前与知识协作直接相关的核心文件树大致如下：

```text
repo/
├── AGENTS.md                         # AI 协作者入口：最短硬约束 + 任务路由
├── CONTRIBUTING.md                   # 仓库级强约束与总入口
│
├── agentic/ llm/ training-infra/ ... # 知识树本体：主题目录与正文内容
│   ├── README.md                     # 语义定向接口
│   ├── index.md                      # 结构路由接口
│   ├── overview.md                   # 人类向主文（按需）
│   ├── landscape.md                  # 结构化研究文（按需）
│   ├── backlog.md                    # 内容缺口（按需）
│   ├── candidates.md                 # 候选对象队列（按需）
│   ├── roadmap.md                    # 路径指引（按需）
│   └── conflict.md                   # 冲突记录（按需）
│
└── docs/
    ├── README.md                     # docs/ 说明入口
    ├── index.md                      # docs/ 查找导航页
    ├── design/
    │   ├── README.md                 # 设计层入口
    │   ├── system-design.md          # 系统级设计说明
    │   ├── capability-design.md      # capability layer 专项设计与当前映射
    │   ├── research-ingestion-design.md # 研究型输入吸收链专项设计
    │   └── cases-layer-design.md     # case layer 长期边界专项设计
    │
    ├── contributing/
    │   ├── README.md                 # rules / methods 层入口
    │   ├── meta-rules.md             # 规则如何修改
    │   ├── documentation-workflow.md # 新材料处理与文档增改流程
    │   ├── metadata-files.md         # 元信息文件模型
    │   ├── readme-rules.md           # README 规则
    │   ├── organization-principles.md# 知识树组织原则
    │   ├── structure-refactoring-rules.md # 结构重构规则
    │   ├── research-artifacts.md     # 研究产物组织规则
    │   ├── evidence-and-traceability.md   # Evidence + Trace 组合入口
    │   ├── evidence-rules.md         # Evidence 规则
    │   ├── traceability-rules.md     # Traceability 规则
    │   ├── intent/                   # 设计意图与长解释（按需）
    │   └── cases/                    # 边界样本与误判复盘（按需）
    │
    ├── capabilities/
    │   ├── README.md                 # capability layer 入口
    │   ├── index.md                  # capability layer 查找导航页
    │   ├── meta.md                   # capability layer 自我约束文档
    │   ├── navigate.md
    │   ├── place.md
    │   ├── ingest.md
    │   ├── structure.md
    │   ├── answer.md
    │   └── maintain.md
    │
    ├── templates/                    # 文档模板
    └── test/                         # 渲染与兼容性测试材料
```

这棵树里，最重要的不是目录名本身，而是三类角色：

- **运行时接口**：`README.md`、`index.md`、各类元信息文件
- **行为约束与方法层**：`AGENTS.md`、`CONTRIBUTING.md`、`docs/contributing/`
- **知识承载层**：各主题目录下的正文、专题与研究材料

后文的系统分层，就是对这棵实际文件树的抽象，而不是脱离仓库现状凭空设计另一套结构。

### 2.1 知识树本体

仓库的主题目录（如 `llm/`、`agentic/`、`training-infra/`、`interdisciplinarity/`）构成了知识树本体。

它的职责是：

- 承接知识正文
- 提供长期稳定的主题边界
- 通过层级结构表达主从关系与下钻路径

知识树首先是树，而不是图。这带来两个效果：

- 好处是结构直观、维护成本相对可控
- 代价是知识内部真实结构往往比树更复杂，因此需要额外的导航与映射机制

### 2.2 入口接口：`README.md` 与 `index.md`

在这套系统里，`README.md` 与 `index.md` 不只是普通文档，而是最重要的导航接口。

- `README.md`：语义定向接口。回答“这是什么地方、为什么存在、与相邻目录如何区分”。
- `index.md`：结构路由接口。回答“这个目录下面有什么、我已经知道想找什么时该去哪”。

对普通读者，这两个文件提供浏览入口；对 AI 协作者，它们同时也是定位落位点时的搜索接口。

也就是说：

- `README.md` 帮助系统判断“该不该进这个目录”
- `index.md` 帮助系统判断“进入后该继续往哪一层走”

### 2.3 元信息文件层

`overview.md`、`landscape.md`、`backlog.md`、`candidates.md`、`roadmap.md`、`conflict.md` 等元信息文件，不是知识正文，而是这套系统的中间层能力位。

它们负责把不同认知任务拆开承接，例如：

- 说明
- 导航
- 理解
- 结构化研究
- 缺口管理
- 对象队列
- 路径指引
- 冲突记录

没有这一层，系统就会不断把“尚未稳定的材料”和“已成形的正文”混在一起。

### 2.4 规则与方法层

`AGENTS.md`、`CONTRIBUTING.md` 与 `docs/contributing/` 下的规则文档，共同构成系统的规则与方法层。

这层的职责不是保存领域知识，而是约束系统如何稳定运行，例如：

- 规则入口与任务路由
- 文档治理与结构重构
- Evidence / Traceability
- 元信息文件职责
- README 规则
- 材料处理流程

因此，`docs/contributing/` 更像是这套 agent system 的 rules / methods layer，而不只是普通意义上的贡献规范目录。关于为什么这一层还要继续拆成主规则、`intent/`、`cases/`，以及它与 capability layer、design layer 的更细边界，另见 [`contributing-design.md`](./contributing-design.md)。

### 2.5 AI 协作者层

AI 协作者不是系统外部的使用者，而是系统的一部分。

它的职责包括：

- 读取入口文件与规则文件
- 借助知识树与入口接口定位落位点
- 根据元信息文件模型选择合适的承接文件
- 在 Evidence / Traceability 约束下写入、修改、迁移内容
- 在问答时利用现有知识结构恢复上下文，并避免把未验证材料写成定论

在系统设计层面，AI 协作者不是只读消费者，也是写入参与者：它与人类维护者共享同一套落位规则、元信息文件边界和 Evidence 约束；不会因为操作者是 AI，就降低验证标准或改变分流逻辑。

因此，AI 协作者与规则文档、入口文件共同形成一个运行时系统，而不是“AI 在外部调用一堆静态文档”。

## 3. 系统提供的能力

从能力设计角度看，这套系统至少包含以下能力域。

### 3.1 Ingest：输入与吸收

回答：拿到新材料后，先怎么分流、暂存与吸收，并在条件成熟后回流到元信息文件、对象目录 `notes/` 或正文，而不是直接写正文。

典型问题：

- 新材料是什么类型
- Evidence 状态如何
- 是正文候选、缺口、对象、冲突，还是临时输入
- 今天不写正文时先放哪里
- 什么时候应继续留在 `temp/`，什么时候应先进入元信息文件或对象目录 `notes/`

当前主要由以下文档承接：

- `docs/contributing/documentation-workflow.md`
- `docs/contributing/evidence-and-traceability.md`
- `docs/contributing/evidence-rules.md`
- `docs/contributing/traceability-rules.md`

### 3.2 Place：定位后的落位决策

回答：在已经找到或至少缩小到一片稳定区域之后，这个内容应停在哪一层、应落成哪类文件，以及是否继续下沉。

典型问题：

- 在几个候选目录之间，哪个才是最高稳定归属
- 应该停在当前层，还是继续下沉到更细的子目录
- 找到位置后，应该写正文、`backlog.md`、`candidates.md`、`landscape.md` 还是其他元信息文件
- 是先保留在元信息文件中，还是已经适合进入正文

Place 与 Navigate 会共享“沿着 `README.md` / `index.md` 缩小范围”的前置动作，但两者的输出不同：Navigate 的输出是最可能的目标位置，Place 的输出是具体的落位决策。

### 3.3 Structure：结构与导航

回答：如何让目录树保持可导航、可维护、可长期扩展。

典型问题：

- README 和 index 的分工是什么
- overview 和 landscape 如何拆分
- 什么时候该出现 backlog / candidates / roadmap / conflict
- 父层 index 与子层 index 如何分工
- 横向映射时，目录级浅链与少量深链例外如何控制

当前主要由以下文档承接：

- `docs/contributing/readme-rules.md`
- `docs/contributing/metadata-files.md`
- `docs/contributing/organization-principles.md`
- `docs/contributing/structure-refactoring-rules.md`

### 3.4 Navigate：搜索与导航

回答：人类读者或 AI 如何从问题出发，在树结构仓库中快速找到最可能的入口或目标位置。

典型问题：

- 如果我只知道问题，不知道目录名，该先去哪里
- 如何利用 `README.md` 做语义归属判断
- 如何利用 `index.md` 做层级下钻
- 当知识内在结构更像图时，如何在树结构仓库中近似导航

Navigate 关注的是“如何找到位置”，而不是“找到位置以后该写什么文件”。它的输出是导航路径或目标位置，而不是落位决策。

这是系统对外能力的一部分，不应只被理解为维护者内部规则。

### 3.5 Answer：问答与解释

回答：系统如何基于已有知识回答问题，同时守住证据边界。

典型问题：

- 什么已经是稳定结论
- 什么只是观察或推断
- 什么还只是 backlog / candidates / conflict 中的输入
- 回答一个问题时，何时需要回到仓库重新查找

这项能力目前更多体现于 AI 协作者的行为，而不是单独成文的公共文档。

## 4. 为什么“搜索 / 定位”是独立能力

在这套系统里，“搜索 / 定位”不能简单视为“写文档前的一步”，原因有三点：

1. 它不只服务维护者，也服务普通读者与 AI 协作者。
2. 它处理的是“在知识树中找到位置”，而不是“找到位置以后写哪种文件”。
3. 它依赖的主要接口是 `README.md` 与 `index.md`，而不是某个单独的维护规则文件。

因此，搜索 / 定位应被看作一个相对独立的系统能力，而不是附着在 `documentation-workflow.md` 里的小步骤。

## 5. 文档层应如何分工

如果把这套系统当成一个 agent system 来设计，文档至少应分成三层。

### 5.1 Capability docs

回答：系统提供什么能力，读者或协作者如何使用这些能力。

这里的“capability docs”不是维护规则的另一种说法，而是把仓库长期依赖的底层能力单独显式化。它们关注的是：

- 系统能稳定完成什么任务
- 这些任务对人类读者、维护者与 AI 协作者分别意味着什么
- 不同能力之间如何衔接，而不是只告诉维护者“下一步按什么流程做”

因此，这一层长期上不应继续散落在 `docs/` 根下，也不应混回 `docs/contributing/`。更合理的方向是逐步形成与 rules / methods layer 并列的 `docs/capabilities/`。其中，能力说明、能力分流与能力层自我约束应分别由能力文档、`index.md` 与 `meta.md` 承接，而不是重新混回单篇总文。

### 5.2 Operational rules

回答：当你要维护仓库时，应该按什么顺序执行，如何减少返工与误写。

典型文档：

- `docs/contributing/documentation-workflow.md`
- `docs/contributing/metadata-files.md`
- `docs/contributing/readme-rules.md`
- `docs/contributing/research-artifacts.md`

### 5.3 Behavioral constraints

回答：系统绝不能跨越什么边界。

例如：

- 未验证材料不能直接写成主线定论
- README 不应吞并导航或元信息职责
- 不要机械补齐元信息文件
- 不要无依据新建规则文档

这类约束主要由：

- `AGENTS.md`
- `CONTRIBUTING.md`
- `docs/contributing/meta-rules.md`

共同承接。

## 6. 对当前文档系统的设计判断

基于上面的分层，可以得到几个设计判断。

### 6.1 `docs/contributing/` 更像 rules / methods layer

它负责系统如何稳定运行，而不是对所有读者暴露完整使用能力。

### 6.2 `documentation-workflow.md` 应继续只负责操作流程

它回答“如何动手”，而不应承担搜索 / 定位能力设计。

### 6.3 搜索 / 定位应单独建模

因为它既服务维护者，也服务普通读者与 AI 协作者；它不只是内部维护步骤。

### 6.4 `README.md` / `index.md` 是系统接口，不只是静态文档

后续围绕它们的设计，不应只从写作规则出发，也应从系统导航接口出发。

## 7. 设计文档层的组织方向

从系统设计与长期组织原则看，设计信息不应只压在单篇系统设计文档里。随着 capability layer、入口层和规则层逐步稳定，`docs/design/` 更适合作为专项设计文档目录存在。

它与 `docs/capabilities/`、`docs/contributing/` 的关系应是并列，而不是包含：

- `docs/design/`：design layer，回答“系统为什么这样分层、当前整体设计如何演化、专项设计文档放在哪里”
- `docs/contributing/`：rules / methods layer，回答“维护仓库时应遵守什么规则、按什么流程执行”
- `docs/capabilities/`：capability layer，回答“这套系统提供什么能力、如何使用这些能力、能力之间如何衔接”

如果按长期建设来规划，一个更合理的目标树可先理解为：

```text
docs/
├── README.md
├── index.md
├── design/
│   ├── README.md
│   ├── system-design.md
│   ├── capability-design.md
│   ├── research-ingestion-design.md
│   └── cases-layer-design.md
├── capabilities/
│   ├── README.md
│   ├── index.md
│   ├── meta.md
│   ├── navigate.md
│   ├── place.md
│   ├── ingest.md
│   ├── structure.md
│   ├── answer.md
│   └── maintain.md
├── contributing/
│   ├── README.md
│   └── ...
├── templates/
└── test/
```

这里的关键不是要求设计目录立刻膨胀，而是承认：

- `docs/design/README.md` 承接设计层入口
- `docs/design/system-design.md` 承接系统级设计主文
- `docs/design/capability-design.md` 承接 capability layer 的专项设计文档与横向总表
- `docs/design/research-ingestion-design.md` 承接“从外部临时材料到稳定知识层”的研究型输入吸收链专项设计
- `docs/design/cases-layer-design.md` 承接 case layer 是否应从 contributing 附属物升级为独立层的长期结构判断
- `docs/design/contributing-design.md` 承接 `docs/contributing/` 这一层的专项设计说明
- `docs/capabilities/meta.md` 承接 capability layer 自身的边界、增长方式与联动更新面
- capability layer 本身不再承担“能力施工总表”的默认入口职责

## 8. 仍待补强的能力空缺

从系统设计视角看，当前最明显的能力空缺与待继续建设点有三个：

- **长期维护能力补强**：`docs/capabilities/maintain.md` 已建立，但仍需要更多对象级示例与生命周期判断样式
- **问答能力补强**：`docs/capabilities/answer.md` 已建立，但仍需要更多回答样式、边界表达与问题类型分层
- **结构能力补强**：`docs/capabilities/structure.md` 已建立，但仍需要更多结构退化样式、分工误判样例与层间协作样式

这三者都还没有完全成熟为对外稳定、足够完整的一组能力文档：

- `maintain` 已有首份公共文档，但仍在补强阶段
- `answer` 已有首份公共文档，但仍在补强阶段
- `structure` 已有首份公共文档，但仍在补强阶段

## 9. 本文的边界

本文不负责：

- 定义单个元信息文件的完整职责
- 规定 README、index、overview、landscape 的细节写法
- 展开 `docs/contributing/` 这一层的内部设计细分
- 代替材料处理流程或证据规则
- 给出具体目录的落位结论

这些问题应回到对应专题文件：

- `docs/contributing/` 的专项层设计：[`contributing-design.md`](./contributing-design.md)
- 元信息文件职责：`docs/contributing/metadata-files.md`
- README 规则：`docs/contributing/readme-rules.md`
- 材料处理流程：`docs/contributing/documentation-workflow.md`
- Evidence / Traceability：`docs/contributing/evidence-and-traceability.md`

## 10. 阅读建议

- 想理解这个仓库式 agent system 为何这样设计：读本文
- 想看 capability layer 的专项设计与当前映射状态：读 [`capability-design.md`](./capability-design.md)
- 想理解为什么“外部临时材料 → 稳定知识层”的处理更接近一条 skill-like workflow：读 [`research-ingestion-design.md`](./research-ingestion-design.md)
- 想判断复杂案例长期应属于 contributing 附属案例、capability 案例，还是独立 case layer：读 [`cases-layer-design.md`](./cases-layer-design.md)
- 想实际新增、迁移、修改文档：读 `docs/contributing/documentation-workflow.md`
- 想判断某种元信息文件该不该出现：读 `docs/contributing/metadata-files.md`
- 想理解 README / index / overview / landscape 如何分工：读 `docs/contributing/readme-rules.md` 与 `docs/contributing/metadata-files.md`
- 想看复杂边界样本与误判复盘：进入 `docs/contributing/cases/`
