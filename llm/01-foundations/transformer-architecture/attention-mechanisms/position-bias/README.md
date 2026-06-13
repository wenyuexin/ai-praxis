# Position Bias (位置偏差)

LLM 注意力机制在长序列上的系统性位置偏差——首因效应与近因效应导致的 U 型性能曲线，从现象发现到注意力校准、理论解释与工程利用的完整研究线。

## 目录结构

```
position-bias/
├── README.md          ← 本文件：定位与导航
├── survey.md          ← 领域全景：10 篇论文的研究脉络、关键概念、论文关系图
├── backlog.md         ← 内容缺口与填补计划
└── papers/            ← 论文笔记
    ├── LostInTheMiddle_2307.03172.md                   # 现象发现 (Liu et al., 2024)
    ├── FoundInTheMiddle_2406.16008.md                  # 注意力校准 (Hsieh et al., 2024)
    ├── LostInTheMiddle_EmergentProperty_2510.10276.md  # 需求适应论 (Salvatore et al., 2025)
    ├── FromBiasToBenefit_8362.md                       # U型放置策略 (2025)
    ├── InitialSaliency_2512.13109.md                   # 初始显著性 (2025)
    ├── LostInTheMiddle_AtBirth_2603.10123.md            # 结构起源论 (Chowdhury, 2026)
    ├── GoldPanning_2510.09770.md                       # 偏差即信号 (Byerly & Khashabi, 2025)
    ├── WhatWorks_LostInTheMiddle_2511.13900.md         # 缓解方法基准 (Gupte et al., 2025)
    ├── ResidualAwareTheory_2602.16837.md               # 残差感知理论 (Herasimchyk, 2026)
    ├── EmergenceOfPositionBias_2502.01951.md           # 图论框架 (Wu, 2025 ICML)
    └── DistanceBias_2410.14641.md                      # 多信息间距偏差 (Tian et al., 2025 ACL)
```

## 同目录导航

- [survey.md](./survey.md) — 领域全景：10 篇论文的研究脉络、关键概念、论文关系图
- [backlog.md](./backlog.md) — 内容缺口与填补计划

## 快速入口

- **想了解领域全貌** → [survey.md](./survey.md)
- **从现象开始读** → [LostInTheMiddle_2307.03172.md](./papers/LostInTheMiddle_2307.03172.md)
- **从解决方案开始读** → [FoundInTheMiddle_2406.16008.md](./papers/FoundInTheMiddle_2406.16008.md)
- **想看还有哪些缺口** → [backlog.md](./backlog.md)

---

*最后更新: 2026-05-17*