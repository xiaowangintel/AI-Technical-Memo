# modelopt.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/modelopt.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document describes NVIDIA Model Optimizer (modelopt) integration with vLLM. It covers supported checkpoint formats, post-training quantization (PTQ) workflow using the `mtq` API, exporting quantized checkpoints, and serving/testing with vLLM.

**CN:** 本文档介绍 NVIDIA Model Optimizer（modelopt）与 vLLM 的集成，内容涵盖支持的检查点格式、使用 `mtq` API 的训练后量化（PTQ）工作流、导出量化检查点，以及在 vLLM 中服务和测试。

### Supported Formats / 支持的格式

**EN:** vLLM detects modelopt checkpoints via `hf_quant_config.json`. Supported formats: `FP8`, `FP8_PER_CHANNEL_PER_TOKEN`, `FP8_PB_WO` (block-scaled FP8 weight-only), `NVFP4` (`quantization="modelopt_fp4"`), `MXFP8` (`quantization="modelopt_mxfp8"`).

**CN:** vLLM 通过 `hf_quant_config.json` 检测 modelopt 检查点。支持格式：`FP8`、`FP8_PER_CHANNEL_PER_TOKEN`、`FP8_PB_WO`（块缩放 FP8 仅权重）、`NVFP4`（`quantization="modelopt_fp4"`）、`MXFP8`（`quantization="modelopt_mxfp8"`）。

### PTQ Workflow / PTQ 工作流

**EN:** Load model, select config (e.g., `mtq.FP8_DEFAULT_CFG`), define a `forward_loop` over calibration data, call `mtq.quantize(model, config, forward_loop)`, then export with `export_hf_checkpoint(model, export_dir)`.

**CN:** 加载模型，选择配置（如 `mtq.FP8_DEFAULT_CFG`），定义校准数据上的 `forward_loop`，调用 `mtq.quantize(model, config, forward_loop)`，然后用 `export_hf_checkpoint(model, export_dir)` 导出。

### Deployment / 部署

**EN:** Load with `LLM(model=..., quantization="modelopt")` or serve with `vllm serve <path> --quantization modelopt`. Example: `nvidia/Llama-3.1-8B-Instruct-FP8` (available on HuggingFace).

**CN:** 使用 `LLM(model=..., quantization="modelopt")` 加载，或通过 `vllm serve <path> --quantization modelopt` 服务。示例：HuggingFace 上的 `nvidia/Llama-3.1-8B-Instruct-FP8`。

### Testing / 测试

**EN:** Modelopt unit tests require local checkpoint paths and are skipped in CI by default. Set environment variables `VLLM_TEST_MODELOPT_*_MODEL_PATH` to run them locally.

**CN:** Modelopt 单元测试需要本地检查点路径，CI 中默认跳过。设置环境变量 `VLLM_TEST_MODELOPT_*_MODEL_PATH` 可在本地运行。

## Key Concepts / 关键概念

- **EN:** NVIDIA Model Optimizer — a library for PTQ and QAT of LLMs, VLMs, and diffusion models targeting NVIDIA GPU deployment. **CN:** NVIDIA Model Optimizer——面向 NVIDIA GPU 部署的 LLM、VLM 和扩散模型的 PTQ 和 QAT 库。
- **EN:** `hf_quant_config.json` — the sentinel file that tells vLLM a model was quantized with modelopt. **CN:** `hf_quant_config.json`——告知 vLLM 模型是用 modelopt 量化的标志文件。
- **EN:** `quantization="modelopt"` — the vLLM flag required when loading most modelopt checkpoints. **CN:** `quantization="modelopt"`——加载大多数 modelopt 检查点时所需的 vLLM 标志。
- **EN:** NVFP4 / MXFP8 — advanced NVIDIA-specific quantization formats requiring dedicated quantization flags. **CN:** NVFP4 / MXFP8——需要专用量化标志的高级 NVIDIA 专用量化格式。
- **EN:** `export_hf_checkpoint` — modelopt API to save a quantized model in HuggingFace-compatible format for use with vLLM. **CN:** `export_hf_checkpoint`——将量化模型以 HuggingFace 兼容格式保存供 vLLM 使用的 modelopt API。
