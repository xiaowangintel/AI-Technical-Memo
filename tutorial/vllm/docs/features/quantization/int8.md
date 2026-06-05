# int8.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/int8.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document covers INT8 W8A8 quantization (8-bit weights and activations) using `llm-compressor`. It uses a combined SmoothQuant + GPTQ approach, requiring calibration data. Hardware support and a Blackwell limitation are explicitly noted.

**CN:** 本文档介绍使用 `llm-compressor` 进行 INT8 W8A8 量化（8 位权重和激活），采用 SmoothQuant + GPTQ 组合方法，需要校准数据，明确说明了硬件支持情况和 Blackwell 限制。

### Hardware Support and Limitation / 硬件支持与限制

**EN:** Supported on compute capability > 7.5 (Turing, Ampere, Ada, Hopper). **Not supported on Blackwell (≥ 10.0)** — use FP8 instead on RTX 6000 Blackwell.

**CN:** 支持算力 > 7.5 的 GPU（Turing、Ampere、Ada、Hopper）。**Blackwell（≥ 10.0）上不支持**——在 RTX 6000 Blackwell 上改用 FP8。

### Four-Step Workflow / 四步工作流

**EN:** (1) Load model. (2) Prepare calibration data — 512 samples from `ultrachat_200k`, max seq len 2048. (3) Apply recipe combining `SmoothQuantModifier(smoothing_strength=0.8)` + `GPTQModifier(scheme="W8A8")`; save. (4) Load in vLLM and evaluate with `lm_eval`.

**CN:** (1) 加载模型。(2) 准备校准数据——从 `ultrachat_200k` 取 512 条样本，最大序列长度 2048。(3) 应用结合 `SmoothQuantModifier(smoothing_strength=0.8)` + `GPTQModifier(scheme="W8A8")` 的方案并保存。(4) 在 vLLM 中加载，用 `lm_eval` 评估。

### SmoothQuant Role / SmoothQuant 作用

**EN:** SmoothQuant migrates quantization difficulty from activations to weights by scaling activation outliers. This makes activation quantization to INT8 more accurate. Applied before GPTQ in the recipe.

**CN:** SmoothQuant 通过缩放激活异常值，将量化难度从激活转移到权重，使激活量化为 INT8 更加精确，在方案中先于 GPTQ 应用。

### Best Practices / 最佳实践

**EN:** Same principles as INT4: 512 calibration samples minimum, 2048 max seq len, use the model's chat template, and add `add_bos_token=True` during evaluation.

**CN:** 与 INT4 相同的原则：最少 512 条校准样本，最大序列长度 2048，使用模型对话模板，评估时添加 `add_bos_token=True`。

## Key Concepts / 关键概念

- **EN:** W8A8 — both weights and activations quantized to 8-bit integers; enables hardware-accelerated INT8 GEMM on Turing/Ampere/Ada/Hopper. **CN:** W8A8——权重和激活均量化为 8 位整数，支持 Turing/Ampere/Ada/Hopper 上的硬件加速 INT8 GEMM。
- **EN:** SmoothQuant — per-channel scaling that moves quantization difficulty from activations (hard) to weights (easy). **CN:** SmoothQuant——逐通道缩放，将量化难度从激活（难）转移到权重（易）。
- **EN:** `smoothing_strength=0.8` — controls the degree to which activation outliers are shifted to weights; higher values shift more. **CN:** `smoothing_strength=0.8`——控制将激活异常值转移到权重的程度，值越高转移越多。
- **EN:** Blackwell incompatibility — INT8 is explicitly unsupported on compute capability ≥ 10.0; FP8 is the recommended alternative. **CN:** Blackwell 不兼容——算力 ≥ 10.0 的 GPU 明确不支持 INT8，FP8 是推荐替代方案。
- **EN:** Combined recipe — SmoothQuantModifier followed by GPTQModifier for best W8A8 accuracy. **CN:** 组合方案——SmoothQuantModifier 后接 GPTQModifier，获得最佳 W8A8 精度。
