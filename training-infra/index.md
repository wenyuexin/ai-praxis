# Training Infrastructure Index

> 如果你已经知道自己要找的是哪类训练基础设施问题、子目录或工程入口，从这里开始。
> 边界：本文件负责目录结构展示与查找导航，不替代 `README.md` 的目录定位说明，也不替代 `landscape.md` 的结构化研究文。

## 目录结构

```text
training-infra/
├── 01-hardware-and-networking/         # 硬件与网络
├── 02-distributed-training/            # 分布式训练
├── 03-frameworks-and-tools/            # 训练框架与工具
├── 04-memory-and-storage/              # 内存与存储优化
├── 05-compilation-and-kernels/         # 编译与内核
├── 06-ml-operations/                   # ML 运维
└── 07-observability-and-debugging/     # 可观测性与调试
```

## 按子目录定位

- `01-hardware-and-networking/`：GPU、TPU、互联拓扑与训练硬件底座
- `02-distributed-training/`：数据并行、模型并行、流水线并行与混合并行策略
- `03-frameworks-and-tools/`：DeepSpeed、Megatron-LM、HF Accelerate 与训练编排工具
- `04-memory-and-storage/`：梯度检查点、混合精度、offloading、checkpoint 与恢复
- `05-compilation-and-kernels/`：Triton、自定义 kernel、`torch.compile`、XLA / JIT
- `06-ml-operations/`：实验管理、数据管线、ML CI/CD 等训练运维能力
- `07-observability-and-debugging/`：profiling、异常检测、成本管理与训练调试

## 按问题找入口

- 想看算力、加速器和互联如何影响训练：进入 `01-hardware-and-networking/`
- 想看模型装不下、训练跑不快时怎么拆任务：进入 `02-distributed-training/`
- 想看并行与优化策略如何被框架封装：进入 `03-frameworks-and-tools/`
- 想看显存不足、checkpoint、恢复与存储管理：进入 `04-memory-and-storage/`
- 想看算子与编译层的训练加速：进入 `05-compilation-and-kernels/`
- 想看实验管理、数据管线、CI/CD：进入 `06-ml-operations/`
- 想看训练异常诊断、性能观察与成本控制：进入 `07-observability-and-debugging/`

## 其他入口

- 想先判断本目录是否适合你的问题，转到 [`README.md`](./README.md)
- 想理解本目录为什么按当前结构组织、与相邻目录如何分工时，转到 [`landscape.md`](./landscape.md)
- 想按阶段推进学习时，转到 [`roadmap.md`](./roadmap.md)

---

*最后更新: 2026-06-16*
