# Web Browsing

> 适用范围：单智能体中的网页浏览、页面观察与信息提取研究
> 阶段状态：占位入口，后续补充主干内容

## 一句话定位

本目录用于整理单智能体如何浏览网页、读取页面状态、提取结构化信息，并把浏览结果纳入后续执行循环。

## 快速入口

- 结合上层 `../README.md` 理解 web browsing 在 tool-use 体系中的位置。
- 如需先理解多工具协作，优先查看 `../tool-orchestration.md`。
- 如需先理解调用稳定性与失败恢复，优先查看 `../tool-invocation-reliability.md`。

## 边界说明

- 放在这里：页面浏览、DOM/页面状态观察、导航流程、信息提取。
- 不放在这里：一般 API 调用（放 `../api-calling/`）、统一工作流组织（放 `../tool-orchestration.md`）、浏览器运行环境隔离（放 `../../05-environments/`）。
