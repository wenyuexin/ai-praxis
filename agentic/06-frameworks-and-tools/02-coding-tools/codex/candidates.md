# Codex Candidates

> 适用范围：`codex/` 中值得持续研究的官方页面、开源仓库模块、插件生态与替代模型后端对象
> 使用原则：本文件记录候选研究对象，不直接承接正文定论；对象研究完成后，再回填对应专题、`backlog.md` 或 `conflict.md`

## 一、收录规则

适合进入本文件的对象通常满足以下条件之一：

- 能直接支撑 `codex/` 当前仍待补证的关键专题，例如开源边界、替代路径、cloud 环境细节、automations 入口或 multi-agent / worktree 实现边界
- 是官方文档、开源仓库、产品组件、插件或模型后端中的稳定研究对象，适合作为 `Verified` / `Observed` 级结论的候选证据
- 已经形成持续跟踪需求，不适合继续塞进 `backlog.md` 的问题缺口条目中
- 能帮助校正“可接入”与“可等价替代”、“仓库存在性”与“产品承诺”之间的边界

不适合进入本文件的内容：

- 纯粹的问题缺口、抽象疑问或待写主题（应进入 `backlog.md`）
- 已经足够稳定、应直接回填正文的定论
- 只有零散社区说法支撑、尚未形成稳定研究对象的线索
- 与 `codex/` 当前 evidence 缺口无直接关系的泛产品清单

---

## 二、P0：开源替代路径对象

### 2.1 Ollama

- **对象类型**：本地模型运行时 / 模型提供商后端
- **关联问题**：`codex-rs/ollama/` 的接入边界；本地模型后端是否只解决推理接入，还是足以覆盖官方工作流能力
- **为什么值得研究**：`codex-open-source.md` 已能确认仓库中存在 `ollama/` 相关对象，但“存在接入层”不等于“可形成完全本地替代”；它是验证该边界的首要对象
- **当前状态**：候选 / 待研究
- **Evidence need / 证据优先级**：本地源码 `D:\github\codex\codex-rs\ollama`、相关 provider 接口、官方 README / docs；社区体验仅作辅助观察
- **预期产出**：补强 `codex-open-source.md` 中“可替换模型后端 ≠ 完全开源替代”的对象级证据；必要时形成单独对象研究文档
- **Source / Decision / Placement / Gap**：来源于 `codex-open-source.md` 的 `Needs` 与 `backlog.md` 当前开源替代缺口；作为对象级研究入口放入本文件；服务本地替代路径判断；仍缺接入方式、能力覆盖与限制条件的源码级核验

### 2.2 LM Studio

- **对象类型**：本地模型运行时 / 模型提供商后端
- **关联问题**：`codex-rs/lmstudio/` 的接入边界；本地桌面模型服务与 Codex 开源工具链之间的关系
- **为什么值得研究**：它与 `Ollama` 一样位于可见仓库结构中，但产品形态、调用方式与可替代边界可能不同，适合作为并列对照对象
- **当前状态**：候选 / 待研究
- **Evidence need / 证据优先级**：本地源码 `D:\github\codex\codex-rs\lmstudio`、provider 适配层、官方文档；社区教程需降级处理
- **预期产出**：为 `codex-open-source.md` 提供本地 provider 并列样本，判断它更像“可选后端”还是“可替代工作站路径”的一部分
- **Source / Decision / Placement / Gap**：来源于 `codex-open-source.md` 的 `Needs` 与本地仓库目录结构；作为本地 provider 候选对象登记；服务本地替代路径对照；仍缺协议、配置与能力边界核验

### 2.3 Bedrock

- **对象类型**：云模型提供商后端 / 认证与 provider 路径
- **关联问题**：`aws-auth/` 与 model provider 相关对象如何支撑非 OpenAI 模型后端；云后端接入与“本地开源替代”之间的边界
- **为什么值得研究**：它能帮助区分“多 provider 支持”与“纯本地替代”这两个常被混写的方向，避免把任意 provider 接入都写成离线或开源替代能力
- **当前状态**：候选 / 待研究
- **Evidence need / 证据优先级**：本地源码 `D:\github\codex\codex-rs\aws-auth`、provider 相关实现、官方仓库说明
- **预期产出**：回写 `codex-open-source.md`，明确云 provider 接入在替代路径中的位置，并收紧“非 OpenAI provider = 本地替代”的表述
- **Source / Decision / Placement / Gap**：来源于 `codex-open-source.md` 中可见 provider 结构与当前替代路径歧义；作为边界校正对象登记；服务区分云 provider 与本地 provider；仍缺具体接入链路和适用边界证据

### 2.4 codex-plugin-cc

- **对象类型**：生态插件 / 非官方扩展对象
- **关联问题**：生态插件应处于“Codex 开源边界主结论”还是“候选补充对象”；插件是否提供官方能力等价替代
- **为什么值得研究**：当前已明确它不适合直接压成主线结论，但它仍是判断生态扩展边界、插件能力与官方产品能力差异的典型对象
- **当前状态**：候选 / 待研究
- **Evidence need / 证据优先级**：插件仓库 README、源码、维护说明；若缺官方关系说明，应保持为 `Observed` 或候选对象
- **预期产出**：避免在 `codex-open-source.md` 中把生态插件误写成官方组件，同时为后续生态补充留出稳定落点
- **Source / Decision / Placement / Gap**：来源于 `codex-open-source.md` 中“不宜写死的说法”与当前对象级研究需求；作为生态插件候选登记；服务官方组件与生态扩展边界区分；仍缺插件定位、能力覆盖与官方关系的稳定证据

---

## 三、P1：待逐页深挖的官方对象

### 3.1 Cloud environment 细规格页面

- **对象类型**：官方文档页面 / 环境规格对象
- **关联问题**：基础镜像、缓存寿命、环境销毁、持久化与数据留存边界
- **为什么值得研究**：`codex-cloud-sandbox.md` 已对齐主边界，但更细规格仍缺稳定一手口径；若后续形成可稳定抓取的页面或章节，应作为对象持续跟踪
- **当前状态**：候选 / 待研究
- **Evidence need / 证据优先级**：OpenAI 官方环境页、`llms-full.txt`、相关 release notes 或产品文档
- **预期产出**：补强 `codex-cloud-sandbox.md` 与 `codex-agent-mechanisms.md` 的细粒度环境规格
- **Source / Decision / Placement / Gap**：来源于 `backlog.md` 中 cloud 细规格缺口；作为官方对象登记而非问题缺口本身；仍缺稳定可复用的一手页面锚点

### 3.2 Automations 各入口对象

- **对象类型**：官方文档入口组 / 产品能力对象
- **关联问题**：`GitHub Action`、`Non-interactive Mode`、`Codex SDK`、`App Server`、`MCP Server` 各自与 automations 的关系
- **为什么值得研究**：`codex-automations-ci.md` 已完成首轮收敛，但后续更适合按入口对象拆开逐页核验，而不是继续把所有细节堆在问题缺口里
- **当前状态**：候选 / 待研究
- **Evidence need / 证据优先级**：官方各入口页面、开源仓库 README、相关示例代码
- **预期产出**：补强 `codex-automations-ci.md` 的对象层支撑，必要时拆出更细专题
- **Source / Decision / Placement / Gap**：来源于 `backlog.md` 中 automations 细入口缺口；作为对象组登记；服务后续逐页补证；仍缺逐入口的稳定事实边界

---

## 四、对象与证据缺口映射

- `Ollama`：主要服务本地 provider 接入边界与“可接入 ≠ 完全替代”缺口
- `LM Studio`：主要服务本地 provider 并列对照与能力边界缺口
- `Bedrock`：主要服务云 provider 接入与本地替代路径区分缺口
- `codex-plugin-cc`：主要服务官方组件与生态插件边界缺口
- `Cloud environment 细规格页面`：主要服务 cloud 镜像 / 缓存 / 留存细节缺口
- `Automations 各入口对象`：主要服务 automations 逐入口补证缺口

---

## 五、与现有主干的关系

- `backlog.md` 继续记录 `codex/` 目录中的问题缺口；本文件只登记为这些缺口服务的候选研究对象
- `codex-open-source.md` 继续保留当前保守主结论；对象核验完成后，再按 Evidence 状态回写正文
- `codex-cloud-sandbox.md`、`codex-automations-ci.md` 等继续承接稳定结论；本文件不直接替代专题正文
- `conflict.md` 继续承接术语、事实、适用边界或结论力度冲突；本文件不新增冲突记录，只保留对象队列
- 当前不创建主题级 `evidence-registry`；证据治理仍以各专题文末 `Evidence` 与本文件的对象队列为主
