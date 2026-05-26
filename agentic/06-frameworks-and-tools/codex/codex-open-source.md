# Codex 开源状态

> **专题文档**。本文总结 Codex 的开源状态——CLI 工具开源（Apache 2.0）、模型权重闭源、以及本地替代方案。适合评估本地部署可行性时查阅。



> 基于官方公告 + 开源仓库 + 社区报道整理（2026.5）。

---

## 开源的部分（Apache 2.0）

| 内容 | 说明 |
|------|------|
| **Codex CLI** | 命令行工具完全开源，Rust 编写，2025.4.17 由 Greg Brockman 宣布开源 |
| **codex-rs 源码仓库**（48+ crate） | 全部开放，包含任务解析器、调度器、沙箱、MCP Server 等核心模块 |
| **Sandbox Runtime** | 沙箱运行时开源，已被社区项目 Zerobox 复用 |
| **codex-plugin-cc** | 2026.3.31 开源，允许 Claude Code 调用 Codex 能力 |
| **Skills / Plugins 系统** | 开源，可自定义工作流技能包 |

**仓库地址**：
- GitHub：`https://github.com/openai/codex`
- GitCode：`https://gitcode.com/GitHub_Trending/codex31/codex`

---

## 未开源的部分

| 内容 | 说明 |
|------|------|
| **模型权重**（GPT-5-Codex / GPT-5.2 等） | ❌ 未开源，无法本地部署模型本身 |
| **历史 API**（code-davinci-002） | ❌ 已于 2023.8 停用 |

---

## 灰色地带：本地模型驱动开源 CLI

虽然模型权重闭源，但 Codex CLI 支持接入非 OpenAI 模型：

| 模型供应商 | 是否需要 API Key | 说明 |
|-----------|----------------|------|
| OpenAI（默认） | ✅ 需要 | GPT-5-Codex / GPT-5.4 等 |
| **Ollama** | ❌ 不需要 | 本地模型，完全离线可用 |
| **LM Studio** | ❌ 不需要 | 本地模型，完全离线可用 |
| Amazon Bedrock | ✅ 需要 AWS 账号 | 云端托管模型 |

> 💡 **实际效果**：工具是开源的，大脑可以换成你自己的。Codex CLI 支持通过 MCP 协议接入 Ollama/LM Studio 等本地模型，可以搭建"全开源 AI 编程工作站"。

---

## 一张图总结

```
Codex = CLI工具(开源) + 沙箱(开源) + MCP协议(开源) + 模型权重(❌闭源)
        │                    │                    │
        ✅ 你可以改           ✅ 你可以改           ❌ 你不能改
        ✅ 你可以 Fork        ✅ 社区已有 Zerobox   ✅ 但可以换成 Ollama
                              等二次开发项目
```

> **Codex CLI 是真开源（Apache 2.0），模型权重是真闭源。但因为 CLI 支持 Ollama/LM Studio，可以用纯本地开源模型 + 开源工具搭建全开源编程工作站。**

---

## 参考来源

- https://github.com/openai/codex
- https://gitcode.com/GitHub_Trending/codex31/codex
- https://openai.com/index/introducing-upgrades-to-codex/

*最后更新: 2026-05-26*