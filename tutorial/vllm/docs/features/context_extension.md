# context_extension.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/context_extension.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document explains how to extend the effective context length of a model in vLLM using RoPE (Rotary Position Embedding) scaling. It covers both offline and online (OpenAI-compatible API) usage, key parameters, and a migration note for users of older vLLM versions.

**CN:** 本文档介绍如何使用 RoPE（旋转位置嵌入）缩放在 vLLM 中扩展模型的有效上下文长度，涵盖离线和在线（OpenAI 兼容 API）两种使用方式、关键参数，以及针对旧版 vLLM 用户的迁移说明。

### Migration Note / 迁移说明

**EN:** The old `--rope-scaling` parameter is no longer supported. Use `--hf-overrides` with a `rope_parameters` dict instead.

**CN:** 旧版 `--rope-scaling` 参数已不再支持，请改用 `--hf-overrides` 传入 `rope_parameters` 字典。

### Offline Example / 离线示例

**EN:** Run `examples/features/context_extension/context_extension_offline.py`, which extends Qwen context via YARN and runs a chat example.

**CN:** 运行 `examples/features/context_extension/context_extension_offline.py`，该脚本通过 YARN 方法扩展 Qwen 模型上下文并执行对话示例。

### Online Usage / 在线使用

**EN:** Launch vLLM server with `--hf-overrides` specifying `rope_parameters` (e.g., `factor`, `rope_type: "yarn"`, `original_max_position_embeddings`) and `--max-model-len` for the new sequence limit. Clients use the standard OpenAI Python SDK.

**CN:** 通过 `--hf-overrides` 指定 `rope_parameters`（如 `factor`、`rope_type: "yarn"`、`original_max_position_embeddings`）和 `--max-model-len` 启动服务端，客户端使用标准 OpenAI Python SDK。

### Key Parameters / 关键参数

**EN:** `rope_type` — RoPE variant (`yarn`, `linear`, `dynamic`). `factor` — context extension multiplier. `original_max_position_embeddings` — baseline for the model. `max_model_len` — new maximum sequence length (original × factor), used for KV cache pre-allocation and request limit.

**CN:** `rope_type`——RoPE 变体（`yarn`、`linear`、`dynamic`）。`factor`——上下文扩展倍数。`original_max_position_embeddings`——模型原始基准值。`max_model_len`——新的最大序列长度（原始值 × 倍数），用于 KV 缓存预分配和请求限制。

## Key Concepts / 关键概念

- **EN:** RoPE scaling — modifies positional encoding to allow the model to generalise beyond its original training context length. **CN:** RoPE 缩放——通过修改位置编码使模型能够泛化到训练时的最大上下文长度之外。
- **EN:** YARN — a specific RoPE extension method that offers smoother extrapolation than plain linear scaling. **CN:** YARN——一种特定的 RoPE 扩展方法，相比简单线性缩放提供更平滑的外推效果。
- **EN:** `--hf-overrides` — vLLM mechanism to inject arbitrary Hugging Face model-config overrides at serve/load time. **CN:** `--hf-overrides`——vLLM 在服务/加载时注入任意 Hugging Face 模型配置覆盖的机制。
- **EN:** `max_model_len` — controls both KV cache allocation size and the maximum accepted request length. **CN:** `max_model_len`——同时控制 KV 缓存分配大小和最大请求长度。
- **EN:** HF Transformers RoPE docs — referenced as the authoritative source for all supported `rope_type` variants and their parameters. **CN:** HF Transformers RoPE 文档——作为所有支持的 `rope_type` 变体及其参数的权威参考。
