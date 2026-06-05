# auto_awq.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/auto_awq.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document covers AWQ (Activation-aware Weight Quantization) 4-bit quantization using the AutoAWQ library. It includes a deprecation notice, quantization workflow, and instructions for running AWQ models in vLLM via the CLI and Python API.

**CN:** 本文档介绍使用 AutoAWQ 库进行 AWQ（激活感知权重量化）4 位量化，包含废弃通知、量化工作流，以及通过 CLI 和 Python API 在 vLLM 中运行 AWQ 模型的说明。

### Deprecation Notice / 废弃通知

**EN:** AutoAWQ is deprecated. Its functionality has been adopted by `llm-compressor`. New users should use the AWQ examples in `llm-compressor` instead.

**CN:** AutoAWQ 已废弃，其功能已被 `llm-compressor` 接管。新用户应改用 `llm-compressor` 中的 AWQ 示例。

### Quantization Workflow / 量化工作流

**EN:** Install `autoawq`, configure `quant_config` with `zero_point`, `q_group_size=128`, `w_bit=4`, and `version="GEMM"`. Load model, call `model.quantize(tokenizer, quant_config)`, then save. Over 6500 pre-quantized AWQ models are available on HuggingFace.

**CN:** 安装 `autoawq`，配置包含 `zero_point`、`q_group_size=128`、`w_bit=4`、`version="GEMM"` 的 `quant_config`，加载模型并调用 `model.quantize(tokenizer, quant_config)` 后保存。HuggingFace 上有超过 6500 个预量化 AWQ 模型可用。

### Running in vLLM / 在 vLLM 中运行

**EN:** Specify `--quantization awq` (CLI) or `quantization="AWQ"` (Python). AWQ models can also be loaded from HuggingFace directly without specifying quantization if the config file declares it.

**CN:** 通过 `--quantization awq`（CLI）或 `quantization="AWQ"`（Python）指定，若模型配置文件已声明量化方式，也可无需显式指定直接加载。

## Key Concepts / 关键概念

- **EN:** AWQ — quantizes weights to INT4 while preserving critical salient weights, using activation statistics to minimise accuracy loss. **CN:** AWQ——将权重量化为 INT4，同时利用激活统计保留关键显著权重以减少精度损失。
- **EN:** `w_bit=4` — target bit width; AWQ focuses on 4-bit weight-only quantization. **CN:** `w_bit=4`——目标位宽；AWQ 专注于 4 位权重量化。
- **EN:** GEMM version — the default kernel variant for AWQ; affects performance on different hardware. **CN:** GEMM 版本——AWQ 的默认内核变体，对不同硬件性能有影响。
- **EN:** Memory benefit — BF16/FP16 to INT4 reduces model memory footprint by approximately 4×. **CN:** 内存收益——从 BF16/FP16 到 INT4，模型内存占用约降低 4 倍。
- **EN:** `llm-compressor` replacement — the recommended path for new AWQ quantization workflows going forward. **CN:** `llm-compressor` 替代——未来新 AWQ 量化工作流的推荐路径。
