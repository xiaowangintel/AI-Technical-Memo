# custom_logitsprocs.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/custom_logitsprocs.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This page explains how to implement user-defined logits processors in vLLM without recompiling the project, while warning that the API is still evolving.
  **CN:** 本文说明如何在不重新编译 vLLM 的情况下实现用户自定义 logits processor，同时明确提示该 API 仍在演进中。
- **EN:** The document is structured around the lifecycle of a processor: subclass `LogitsProcessor`, validate request parameters, track batch state with `update_state`, transform batched logits in `apply`, and declare argmax invariance for greedy-optimization behavior.
  **CN:** 文档围绕 processor 的完整生命周期展开：继承 `LogitsProcessor`、校验请求参数、用 `update_state` 跟踪 batch 状态、在 `apply` 中处理批量 logits，并通过 `is_argmax_invariant` 声明对贪心采样优化的影响。
- **EN:** A detailed section explains `BatchUpdate` semantics (add/remove/move ordering, condensing, swaps), which is critical because custom processors operate on persistent batched requests rather than isolated requests.
  **CN:** 文档详细解释了 `BatchUpdate` 语义（新增/删除/移动顺序、压缩、交换），这很关键，因为自定义 processor 面向的是持久化批处理请求，而不是单独请求。
- **EN:** It includes two implementation patterns: a native batch-level processor example using `extra_args["target_token"]`, and an adapter-based wrapper for legacy request-level callables.
  **CN:** 文档给出了两类实现模式：一是使用 `extra_args["target_token"]` 的原生批处理 processor 示例，二是面向旧式 request-level callable 的适配器封装方案。
- **EN:** Integration guidance covers three loading methods—FQCN at initialization, Python entry points, and offline-only class-object passing—and shows how requests enable behavior through `vllm_xargs` or `SamplingParams.extra_args`.
  **CN:** 集成部分涵盖三种加载方式：初始化时传 FQCN、通过 Python entry point 自动发现、以及仅离线模式下直接传类对象；请求侧则通过 `vllm_xargs` 或 `SamplingParams.extra_args` 启用功能。
- **EN:** The closing best-practices section emphasizes efficiency, sparse state when appropriate, early exits on empty updates, cleanup of finished requests, and avoiding slow wrapped implementations when batch-native logic is feasible.
  **CN:** 末尾最佳实践强调实现效率、在适用时使用稀疏状态、在无更新时尽早返回、及时清理已完成请求，并指出若能原生批处理实现，就应避免性能较差的包装方案。

## Key Concepts / 关键概念
- **EN:** `LogitsProcessor` subclass contract: `validate_params`, `__init__`, `update_state`, `apply`, `is_argmax_invariant`.  
  **CN:** `LogitsProcessor` 子类契约：`validate_params`、`__init__`、`update_state`、`apply`、`is_argmax_invariant`。
- **EN:** `BatchUpdate` ordering matters: process removes, then adds, then moves.  
  **CN:** `BatchUpdate` 的处理顺序很重要：先 remove，再 add，最后 move。
- **EN:** Custom request arguments flow through `SamplingParams.extra_args` / REST `vllm_xargs`.  
  **CN:** 自定义请求参数通过 `SamplingParams.extra_args` / REST `vllm_xargs` 传入。
- **EN:** Loading is initialization-time only; processors cannot be added on demand after engine startup.  
  **CN:** 加载只能发生在初始化阶段；引擎启动后不能按需新增 processor。
- **EN:** `AdapterLogitsProcessor` helps reuse v0-style per-request callables but may trade away performance.  
  **CN:** `AdapterLogitsProcessor` 可复用 v0 风格的逐请求 callable，但可能牺牲性能。
