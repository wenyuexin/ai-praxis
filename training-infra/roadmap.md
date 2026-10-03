# Training Infrastructure Roadmap

> 如果你不确定从哪开始，或想知道训练基础设施应按什么顺序学习，从这里开始。
> 边界：本文件回答“从哪开始、按什么路径推进”，不替代 `README.md` 的目录定位说明，也不替代 `index.md` 的结构展示与查找导航。

## 按读者背景推荐起点

- ML 工程师 / 训练平台开发者：从 `02-distributed-training/` 或 `03-frameworks-and-tools/` 开始，先建立系统工程视角
- 模型训练实践者：从 `04-memory-and-storage/` 开始，再回看 `02-distributed-training/`
- 基础设施 / 系统方向读者：从 `01-hardware-and-networking/` 与 `05-compilation-and-kernels/` 开始
- 运维 / 平台治理方向读者：从 `06-ml-operations/` 与 `07-observability-and-debugging/` 开始

## 按学习阶段推荐路径

### 基础阶段

- `01-hardware-and-networking/`：先理解训练系统的物理资源与互联约束
- `03-frameworks-and-tools/`：熟悉主流训练框架与工具抽象

### 进阶阶段

- `02-distributed-training/`：学习各种并行化策略与通信代价
- `05-compilation-and-kernels/`：理解编译与 kernel 层优化

### 深入阶段

- `04-memory-and-storage/`：系统化处理显存、offloading、checkpoint 与恢复问题
- `06-ml-operations/`：掌握实验管理、数据管线、CI/CD 等运维支撑能力
- `07-observability-and-debugging/`：建立训练诊断、profiling 与成本控制能力

## 按问题推进的简单顺序

1. 先问：瓶颈主要来自计算、通信、显存、编译还是运维？
2. 再进入对应子目录建立局部理解。
3. 如果发现问题横跨多个子目录，先回看 [`landscape.md`](./landscape.md) 的结构分工，再继续下钻。

---

*最后更新: 2026-06-16*
