# online.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/online.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document describes *online quantization* — quantizing a BF16/FP16 model's weights at load time without a pre-quantized checkpoint or calibration data. It covers supported schemes, advanced `quantization_config` configuration, separate dense/MoE quantization, activation overrides, and layer exclusion.

**CN:** 本文档介绍*在线量化*——在加载时对 BF16/FP16 模型权重进行量化，无需预量化检查点或校准数据，内容涵盖支持的方案、高级 `quantization_config` 配置、独立的稠密/MoE 量化、激活覆盖和层排除。

### Supported Schemes / 支持的方案

**EN:** `fp8_per_tensor` — per-tensor weight/activation scales (Ada/Hopper use per-token activation scaling). `fp8_per_block` — 128×128 block weight scales + 1×128 block activation scales. `mxfp8` — e8m0 per-1×32-block scales; requires SM 100+ for full W8A8, falls back to W8A16 on older GPUs.

**CN:** `fp8_per_tensor`——逐张量权重/激活缩放（Ada/Hopper 使用逐 token 激活缩放）。`fp8_per_block`——128×128 块权重缩放 + 1×128 块激活缩放。`mxfp8`——e8m0 逐 1×32 块缩放，SM 100+ 完整支持 W8A8，旧 GPU 回退到 W8A16。

### Advanced `quantization_config` / 高级配置

**EN:** Provides fine-grained control: `linear.weight`, `linear.activation`, `moe.weight`, `moe.activation`, and `ignore` (exact names or `re:` prefixed regex). Accepts JSON or dotted CLI args (`--quantization-config.moe.activation mxfp8`).

**CN:** 提供细粒度控制：`linear.weight`、`linear.activation`、`moe.weight`、`moe.activation` 和 `ignore`（精确名称或 `re:` 前缀正则）。接受 JSON 或点分 CLI 参数（`--quantization-config.moe.activation mxfp8`）。

### Activation Overrides on Pre-quantized Models / 预量化模型的激活覆盖

**EN:** For checkpoint-quantized models, `quantization_config` can override the activation format independently of baked-in weight quantization. Currently enabled for MXFP4 MoE checkpoints (e.g., `gpt-oss`).

**CN:** 对于检查点量化模型，`quantization_config` 可独立于内嵌权重量化来覆盖激活格式，目前已为 MXFP4 MoE 检查点（如 `gpt-oss`）启用。

### Layer Exclusion / 层排除

**EN:** `ignore` list accepts exact layer names or `re:`-prefixed regex patterns. For fused layers, patterns must match the unfused shard names (e.g., `q_proj`, `k_proj`, not `qkv_proj`).

**CN:** `ignore` 列表接受精确层名称或 `re:` 前缀正则模式。对于融合层，模式必须匹配未融合的分片名称（如 `q_proj`、`k_proj`，而非 `qkv_proj`）。

## Key Concepts / 关键概念

- **EN:** Online quantization — weights are quantized at load time from a standard BF16 checkpoint; no prior calibration step required. **CN:** 在线量化——权重在加载时从标准 BF16 检查点量化，无需事先校准。
- **EN:** `fp8_per_block` — block-level scaling trades higher memory overhead for better accuracy than per-tensor scaling. **CN:** `fp8_per_block`——块级缩放以更高的内存开销换取比逐张量缩放更好的精度。
- **EN:** `mxfp8` — microscaling FP8 with e8m0 exponent scales; requires Blackwell (SM 100+) for full W8A8 hardware acceleration. **CN:** `mxfp8`——带 e8m0 指数缩放的微缩放 FP8，完整 W8A8 硬件加速需要 Blackwell（SM 100+）。
- **EN:** Separate `linear` / `moe` config — allows dense layers and MoE expert layers to use different quantization schemes in the same model. **CN:** 独立 `linear` / `moe` 配置——允许同一模型中稠密层和 MoE 专家层使用不同的量化方案。
- **EN:** `ignore` with regex — `re:.*[qkv]_proj` pattern efficiently excludes all QKV projections from quantization. **CN:** 正则 `ignore`——`re:.*[qkv]_proj` 模式可高效排除所有 QKV 投影层不参与量化。
