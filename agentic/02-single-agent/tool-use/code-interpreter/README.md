# Code Interpreter

> 适用范围：单智能体中的代码执行、脚本运行与执行反馈研究
> 阶段状态：占位入口，后续补充主干内容

## 一句话定位

本目录用于整理单智能体如何通过代码执行器或脚本运行环境完成计算、验证与中间结果生成。

## 快速入口

- 结合上层 `../README.md` 理解 code interpreter 在 tool-use 体系中的位置。
- 如需先理解工具调用可靠性，优先查看 `../tool-invocation-reliability.md`。
- 如需先理解环境约束，结合 `../../05-environments/` 相关主题阅读。

## 边界说明

- 放在这里：代码执行器、脚本运行、执行反馈、运行时结果消费。
- 不放在这里：工具工作流整体组织（放 `../tool-orchestration.md`）、浏览器交互（放 `../web-browsing/`）、执行环境隔离（放 `../../05-environments/`）。
