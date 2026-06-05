# engine_args.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/configuration/engine_args.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This page frames engine arguments as the common configuration layer shared by offline inference (`LLM`) and online serving (`vllm serve`). It also points advanced readers to the underlying config classes as the source of truth for types, defaults, and docstrings.

**EN:** Functionally, the page is a bridge between high-level docs and generated argument tables for `EngineArgs` and `AsyncEngineArgs`.

**CN:** 这一页把 engine arguments 描述为离线推理（`LLM`）与在线服务（`vllm serve`）共享的一层核心配置，并提醒高级用户：真正的类型、默认值和 docstring 以底层配置类为准。

**CN:** 从功能上看，它是高层文档与 `EngineArgs` / `AsyncEngineArgs` 自动生成参数表之间的桥梁。

## Key Concepts / 关键概念
- **EN:** Shared engine config — Unifies configuration concepts across offline and online execution modes.
- **CN:** 共享引擎配置 — 统一离线与在线两种执行模式下的配置概念。
- **EN:** Source of truth — Directs developers to config classes when they need authoritative details.
- **CN:** 权威来源 — 提示开发者在需要权威细节时查看配置类定义。
- **EN:** Async engine args — Separately documents async-specific engine options.
- **CN:** 异步引擎参数 — 单独覆盖异步引擎相关选项。
