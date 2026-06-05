# custom_arguments.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/custom_arguments.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document describes vLLM's *custom arguments* mechanism, which allows users to pass arbitrary key-value arguments into vLLM requests without modifying or recompiling the vLLM source code. It covers both offline and online (REST API / OpenAI SDK) usage.

**CN:** 本文档介绍 vLLM 的*自定义参数*机制，允许用户在不修改或重新编译 vLLM 源码的情况下，向 vLLM 请求中传入任意键值参数，涵盖离线和在线（REST API / OpenAI SDK）两种用法。

### Offline Usage / 离线用法

**EN:** Pass a dictionary to `SamplingParams(extra_args={...})`. Any code that holds the `SamplingParams` object can read these values, making them available to custom logits processors and other downstream components.

**CN:** 将字典传入 `SamplingParams(extra_args={...})`。任何持有 `SamplingParams` 对象的代码均可读取这些值，从而使自定义 logits 处理器等下游组件可以访问它们。

### Online Usage / 在线用法

**EN:** Add a `vllm_xargs` key to the REST API JSON body. OpenAI SDK users access this via the `extra_body` parameter. Under the hood, `vllm_xargs` is mapped to `SamplingParams.extra_args`, so both paths are compatible.

**CN:** 在 REST API 的 JSON 请求体中添加 `vllm_xargs` 键。OpenAI SDK 用户通过 `extra_body` 参数访问该功能。在底层，`vllm_xargs` 被映射到 `SamplingParams.extra_args`，因此两种路径兼容互通。

### Integration with Custom Logits Processors / 与自定义 Logits 处理器的集成

**EN:** Custom arguments are especially useful when combined with custom logits processors. The processor should implement `validate_params` to catch invalid argument values early and prevent undefined behaviour.

**CN:** 自定义参数在与自定义 logits 处理器结合使用时尤为有用。处理器应实现 `validate_params` 方法以尽早捕获无效参数值，防止未定义行为。

## Key Concepts / 关键概念

- **EN:** `SamplingParams.extra_args` — the dict field that carries custom arguments in both offline and online paths. **CN:** `SamplingParams.extra_args`——在离线和在线路径中传递自定义参数的字典字段。
- **EN:** `vllm_xargs` — the REST API key that vLLM maps to `extra_args`; used in raw curl calls and via the OpenAI `extra_body` parameter. **CN:** `vllm_xargs`——REST API 中映射到 `extra_args` 的键名，可在原始 curl 请求或 OpenAI `extra_body` 中使用。
- **EN:** No recompile needed — arguments are passed as a plain dictionary, avoiding any code changes to vLLM itself. **CN:** 无需重新编译——参数以普通字典形式传入，不需要改动 vLLM 自身代码。
- **EN:** `validate_params` — recommended guard method on custom logits processors to reject invalid custom arguments at request time. **CN:** `validate_params`——推荐在自定义 logits 处理器上实现的守卫方法，在请求阶段拒绝无效的自定义参数。
- **EN:** Symmetry — `extra_args` works identically whether the request comes from the offline `LLM` API or the online REST server. **CN:** 对称性——无论请求来自离线 `LLM` API 还是在线 REST 服务，`extra_args` 的行为完全一致。
