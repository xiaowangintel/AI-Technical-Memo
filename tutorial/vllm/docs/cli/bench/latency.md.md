# latency.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/cli/bench/latency.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This page is a thin command reference for `vllm bench latency`. It mainly delegates the detailed option list to generated argparse documentation and reuses the shared JSON argument note.

**EN:** Its role is to provide a stable documentation entry point without duplicating the canonical CLI argument definitions.

**CN:** 这一页是 `vllm bench latency` 的轻量级命令参考页。具体参数说明主要通过自动生成的 argparse 文档引入，并复用了共享的 JSON 参数说明。

**CN:** 它的价值在于提供稳定的文档入口，同时避免重复维护参数定义。

## Key Concepts / 关键概念
- **EN:** Generated argument reference — Pulls the authoritative argument list from generated docs.
- **CN:** 自动生成参数参考 — 从生成文档中引入权威参数列表。
- **EN:** JSON CLI notation — Reuses shared guidance for nested JSON-style arguments.
- **CN:** JSON 命令行写法 — 复用嵌套 JSON 参数的统一写法说明。
