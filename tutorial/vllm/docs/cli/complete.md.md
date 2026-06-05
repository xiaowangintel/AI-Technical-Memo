# complete.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/cli/complete.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This page is the parallel reference for `vllm complete`. Like `chat.md`, it is essentially a shell around generated argparse documentation.

**EN:** Its minimal content reflects a docs design where examples and positioning are centralized in broader guides, while option matrices are generated automatically.

**CN:** 该页是 `vllm complete` 的对应参考页。和 `chat.md` 一样，它本质上是对自动生成 argparse 文档的一个包装。

**CN:** 内容极简，体现出一种文档策略：示例和定位放在更大的指南里，参数矩阵则自动生成。

## Key Concepts / 关键概念
- **EN:** Completion reference — Represents the CLI entry for text completion requests.
- **CN:** 补全文本参考 — 对应文本补全请求的 CLI 文档入口。
- **EN:** Generated argparse include — Imports the real option list from generated docs.
- **CN:** 生成式 argparse 引入 — 从生成文档引入真实参数清单。
