# custom_arguments.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/custom_arguments.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This page describes vLLM custom arguments, which let callers attach request-specific parameters outside the standard `SamplingParams` or REST schema. The design goal is extensibility without recompiling vLLM.

**CN:** 本页介绍 vLLM 的 custom arguments（自定义参数）：调用方可以在标准 `SamplingParams` 或 REST 规范之外附加请求级参数。其设计目标是在不重新编译 vLLM 的前提下实现扩展。

**EN:** The document presents both offline and online paths. Offline, custom data is sent through `SamplingParams.extra_args`; online, the REST API uses `vllm_xargs`, and OpenAI SDK users pass the same payload through `extra_body`.

**CN:** 文档同时给出离线与在线两种路径。离线时通过 `SamplingParams.extra_args` 传入；在线时 REST API 使用 `vllm_xargs`，而 OpenAI SDK 则通过 `extra_body` 发送同样的数据结构。

**EN:** A key integration detail is that `vllm_xargs` is mapped internally to `SamplingParams.extra_args`, so downstream code can consume one unified interface. The main caveat is validation: custom logits processors should implement `validate_params` to avoid silent misuse or unstable behavior.

**CN:** 一个关键集成点是：`vllm_xargs` 在内部会映射到 `SamplingParams.extra_args`，因此下游代码可以统一按一种接口读取。主要注意事项是参数校验：自定义 logits processor 应实现 `validate_params`，否则可能导致错误参数引发异常行为。

## Key Concepts / 关键概念
- **EN:** Extending requests without changing core schemas. **CN:** 无需改动核心 schema 即可扩展请求参数。
- **EN:** Offline entry point is `SamplingParams.extra_args`. **CN:** 离线入口是 `SamplingParams.extra_args`。
- **EN:** Online entry point is `vllm_xargs` / `extra_body`. **CN:** 在线入口是 `vllm_xargs` / `extra_body`。
- **EN:** Validation is important for custom logits processors. **CN:** 对自定义 logits processor 来说，参数校验非常重要。
