# API Calling

> 适用范围：单智能体中的 API 调用方式、参数组织与接口约束研究
> 阶段状态：占位入口，后续补充主干内容

## 一句话定位

本目录用于整理单智能体如何调用外部 API、组织请求参数、处理返回结构，并与更大的 tool-use 工作流衔接。

## 快速入口

- 结合上层 `../README.md` 理解 API calling 在 tool-use 体系中的位置。
- 如需先理解多工具工作流，优先查看 `../tool-orchestration.md`。
- 如需先理解调用稳定性，优先查看 `../tool-invocation-reliability.md`。

## 边界说明

- 放在这里：HTTP / RPC / function-style API 调用、请求参数、返回结构、接口约束。
- 不放在这里：浏览器环境交互（放 `../web-browsing/`）、代码执行环境（放 `../code-interpreter/`）、统一工具工作流组织（放 `../tool-orchestration.md`）。

## 目录结构

```text
api-calling/
└── README.md
```
