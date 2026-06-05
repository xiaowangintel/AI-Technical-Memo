# modelopt.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/modelopt.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This document explains how vLLM works with NVIDIA Model Optimizer checkpoints. It starts with the library purpose and installation, then enumerates the checkpoint formats that vLLM can auto-detect from `hf_quant_config.json`.
- **CN:** 本文说明 vLLM 如何对接 NVIDIA Model Optimizer 检查点。内容先介绍库的用途与安装方式，再列出 vLLM 可通过 `hf_quant_config.json` 自动识别的量化格式。
- **EN:** The practical workflow is PTQ-oriented: quantize a Hugging Face model with a calibration loop, export it with `export_hf_checkpoint`, then load it in vLLM through either the Python API or `vllm serve`.
- **CN:** 实操流程主要围绕 PTQ：先通过校准循环量化 Hugging Face 模型，再用 `export_hf_checkpoint` 导出，最后通过 Python API 或 `vllm serve` 在 vLLM 中加载。
- **EN:** The page also includes operational details often missing from overview docs, such as the exact `quant_algo` values, the required `quantization="modelopt"` flag, and environment variables for local unit tests.
- **CN:** 这页还补充了很多概览文档里常缺失的操作细节，例如具体的 `quant_algo` 取值、必须使用的 `quantization="modelopt"` 参数，以及本地单元测试所需的环境变量。

## Key Concepts / 关键概念
- **EN:** Supported ModelOpt formats include `FP8`, `FP8_PER_CHANNEL_PER_TOKEN`, `FP8_PB_WO`, `NVFP4`, and `MXFP8`, each mapped to a specific vLLM loading mode.
  **CN:** 支持的 ModelOpt 格式包括 `FP8`、`FP8_PER_CHANNEL_PER_TOKEN`、`FP8_PB_WO`、`NVFP4` 和 `MXFP8`，并且都对应到特定的 vLLM 加载方式。
- **EN:** The quantization path uses ModelOpt PTQ APIs with a forward calibration loop, showing that export-ready checkpoints are produced after in-place quantized module replacement.
  **CN:** 量化路径使用 ModelOpt 的 PTQ API 和前向校准循环，说明在原地替换量化模块后即可生成可导出的检查点。
- **EN:** vLLM integration is straightforward but strict: Python loading uses `LLM(..., quantization="modelopt")`, and server mode uses `--quantization modelopt`.
  **CN:** vLLM 集成方式直接但要求明确：Python 侧使用 `LLM(..., quantization="modelopt")`，服务模式使用 `--quantization modelopt`。
- **EN:** The testing section signals that ModelOpt coverage depends on local checkpoint paths and is intentionally skipped in CI by default.
  **CN:** 测试部分表明 ModelOpt 相关用例依赖本地检查点路径，因此默认在 CI 中会被跳过。

