# json_tip.inc.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/cli/json_tip.inc.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This shared include explains two equivalent ways to pass JSON-shaped CLI arguments: as a single JSON string or as dotted sub-arguments. It also documents the `+` suffix for appending list elements.

**EN:** Because many vLLM commands accept nested configuration, this snippet standardizes user-facing syntax and avoids repeating the explanation across many pages.

**CN:** 这个共享片段解释了两种等价的 JSON 风格 CLI 传参方式：直接传整个 JSON 字符串，或使用带点号的分层子参数；同时也说明了通过 `+` 追加列表元素的写法。

**CN:** 由于很多 vLLM 命令支持嵌套配置，这个片段统一了用户语法说明，避免在多处文档中重复解释。

## Key Concepts / 关键概念
- **EN:** Dotted JSON arguments — Allows nested structures to be expressed as flattened CLI flags.
- **CN:** 点号式 JSON 参数 — 允许把嵌套结构展平成命令行参数。
- **EN:** List append syntax — Uses `+` to add list items incrementally from the CLI.
- **CN:** 列表追加语法 — 使用 `+` 在命令行中逐项追加列表内容。
- **EN:** Reusable include — Acts as a shared snippet embedded in many CLI docs.
- **CN:** 可复用片段 — 作为共享片段被多处 CLI 文档复用。
