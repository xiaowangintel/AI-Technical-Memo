# fp8.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/fp8.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document covers FP8 W8A8 (8-bit weight and activation) quantization in vLLM. It explains FP8 formats (E4M3 and E5M2), hardware requirements, the llm-compressor workflow for producing quantized models, accuracy evaluation, and online dynamic quantization as a quick-start option.

**CN:** 本文档介绍 vLLM 中的 FP8 W8A8（8 位权重和激活）量化，内容涵盖 FP8 格式（E4M3 和 E5M2）、硬件要求、使用 llm-compressor 生成量化模型的工作流、精度评估，以及作为快速入门选项的在线动态量化。

### FP8 Formats / FP8 格式

**EN:** E4M3 — 1 sign + 4 exponent + 3 mantissa bits; values up to ±448. E5M2 — 1 sign + 5 exponent + 2 mantissa bits; values up to ±57344. E4M3 is preferred for W8A8 due to higher precision; E5M2 has wider dynamic range.

**CN:** E4M3——1 位符号 + 4 位指数 + 3 位尾数，范围 ±448。E5M2——1 位符号 + 5 位指数 + 2 位尾数，范围 ±57344。E4M3 因精度更高而优选用于 W8A8；E5M2 动态范围更宽。

### Hardware Requirements / 硬件要求

**EN:** Full W8A8 FP8 computation requires compute capability ≥ 8.9 (Ada Lovelace, Hopper). On Turing+ (≥ 7.5) models run as W8A16 weight-only using FP8 Marlin kernels.

**CN:** 完整 W8A8 FP8 计算需要算力 ≥ 8.9（Ada Lovelace、Hopper）。在 Turing+（≥ 7.5）上模型以 W8A16 仅权重模式通过 FP8 Marlin 内核运行。

### llm-compressor Workflow / llm-compressor 工作流

**EN:** Three steps: (1) Load model with `transformers` AutoModel. (2) Apply `QuantizationModifier(targets="Linear", scheme="FP8_DYNAMIC")` — static per-channel weight scales, dynamic per-token activation scales; no calibration data needed. (3) Evaluate accuracy with `lm_eval`. Benefits: ~2× memory reduction, up to 1.6× throughput improvement.

**CN:** 三步流程：(1) 用 `transformers` AutoModel 加载模型。(2) 应用 `QuantizationModifier(scheme="FP8_DYNAMIC")`——静态逐通道权重缩放，动态逐 token 激活缩放，无需校准数据。(3) 用 `lm_eval` 评估精度。收益：内存约减少 2 倍，吞吐量最高提升 1.6 倍。

### Online Dynamic Quantization / 在线动态量化

**EN:** Pass `quantization="fp8"` to `LLM(...)` or `--quantization fp8` to `vllm serve`. All Linear layers (except `lm_head`) are quantized to FP8_E4M3 with per-tensor weight scales. Activations are dynamically scaled per forward pass. Latency improvement is limited in this mode.

**CN:** 向 `LLM(...)` 传入 `quantization="fp8"` 或向 `vllm serve` 传入 `--quantization fp8`。除 `lm_head` 外的所有 Linear 层均以逐张量权重缩放量化为 FP8_E4M3，激活在每次前向传播时动态缩放，此模式下延迟改善有限。

## Key Concepts / 关键概念

- **EN:** W8A8 — both weights and activations are quantized to 8-bit; enables hardware-accelerated matrix multiplication. **CN:** W8A8——权重和激活均量化为 8 位，支持硬件加速矩阵乘法。
- **EN:** FP8_DYNAMIC scheme — static per-channel weight quantization + dynamic per-token activation quantization; no calibration dataset required. **CN:** FP8_DYNAMIC 方案——静态逐通道权重量化 + 动态逐 token 激活量化，无需校准数据集。
- **EN:** `llm-compressor` — the recommended library for producing pre-quantized FP8 checkpoints for production use. **CN:** `llm-compressor`——用于生成生产级预量化 FP8 检查点的推荐库。
- **EN:** `add_bos_token=True` — important flag when evaluating quantized models with `lm_eval` to avoid accuracy degradation. **CN:** `add_bos_token=True`——使用 `lm_eval` 评估量化模型时的重要标志，避免精度下降。
- **EN:** HF collection — neuralmagic maintains a collection of ready-to-use FP8 checkpoints for popular LLMs. **CN:** HF 集合——neuralmagic 维护了一个包含主流 LLM 即用 FP8 检查点的 HuggingFace 集合。
