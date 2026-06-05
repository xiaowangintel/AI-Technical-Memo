# gguf.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/gguf.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document describes GGUF model support in vLLM. GGUF is described as experimental and under-optimised, primarily useful for memory footprint reduction. It covers loading from HuggingFace, local files, tensor parallelism, tokenizer handling, and config overrides.

**CN:** 本文档介绍 vLLM 中的 GGUF 模型支持。GGUF 被描述为实验性且未充分优化，主要用于减少内存占用，内容涵盖从 HuggingFace 加载、本地文件加载、张量并行、分词器处理和配置覆盖。

### Loading Models / 加载模型

**EN:** Use `repo_id:quant_type` format (e.g., `unsloth/Qwen3-0.6B-GGUF:Q4_K_M`) on the command line. Only single-file GGUF models are supported; use `gguf-split` to merge multi-file models first. Always use the tokenizer from the base model (`--tokenizer`) to avoid slow/buggy GGUF tokenizer conversion.

**CN:** 在命令行使用 `repo_id:quant_type` 格式（如 `unsloth/Qwen3-0.6B-GGUF:Q4_K_M`）。仅支持单文件 GGUF 模型，多文件模型需先用 `gguf-split` 合并。始终使用基础模型的分词器（`--tokenizer`），避免 GGUF 分词器转换缓慢或出错。

### Tensor Parallelism / 张量并行

**EN:** Add `--tensor-parallel-size N` to enable multi-GPU inference with GGUF models.

**CN:** 添加 `--tensor-parallel-size N` 以启用 GGUF 模型的多 GPU 推理。

### Config Override / 配置覆盖

**EN:** If HuggingFace does not support the model's config conversion, manually provide a compatible config via `--hf-config-path`.

**CN:** 如果 HuggingFace 不支持该模型的配置转换，通过 `--hf-config-path` 手动提供兼容的配置。

### Python API / Python API

**EN:** Pass `model="repo:quant"` and `tokenizer="base_model"` to `LLM(...)`. Use `llm.chat(conversation, sampling_params)` for chat-formatted inputs.

**CN:** 向 `LLM(...)` 传入 `model="repo:quant"` 和 `tokenizer="base_model"`，使用 `llm.chat(conversation, sampling_params)` 处理对话格式输入。

## Key Concepts / 关键概念

- **EN:** GGUF — a quantization format from the llama.cpp ecosystem; supports types like Q4_K_M, Q8_0, etc. **CN:** GGUF——来自 llama.cpp 生态的量化格式，支持 Q4_K_M、Q8_0 等类型。
- **EN:** `repo_id:quant_type` format — vLLM's syntax for loading GGUF files directly from HuggingFace. **CN:** `repo_id:quant_type` 格式——vLLM 直接从 HuggingFace 加载 GGUF 文件的语法。
- **EN:** Single-file limitation — vLLM only supports one-file GGUF; multi-part models must be merged with `gguf-split` first. **CN:** 单文件限制——vLLM 仅支持单文件 GGUF，多分片模型需先用 `gguf-split` 合并。
- **EN:** Tokenizer mismatch — GGUF tokenizer conversion is slow and error-prone; always use `--tokenizer base_model`. **CN:** 分词器不匹配——GGUF 分词器转换缓慢且易出错，始终使用 `--tokenizer base_model`。
- **EN:** Experimental status — GGUF support in vLLM may conflict with other features; report issues to the vLLM team. **CN:** 实验性状态——vLLM 中的 GGUF 支持可能与其他功能冲突，遇到问题请向 vLLM 团队反馈。
