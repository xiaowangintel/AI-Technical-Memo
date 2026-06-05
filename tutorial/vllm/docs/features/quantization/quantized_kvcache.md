# quantized_kvcache.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/quantized_kvcache.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document explains FP8 KV cache quantization in vLLM — reducing KV cache memory footprint by storing keys and values in FP8 rather than BF16/FP16. It covers quantization schemes, scale calibration approaches, and concrete code examples.

**CN:** 本文档介绍 vLLM 中的 FP8 KV 缓存量化——通过以 FP8 而非 BF16/FP16 存储键值来减少 KV 缓存内存占用，内容涵盖量化方案、缩放校准方法及具体代码示例。

### Quantization Schemes / 量化方案

**EN:** Two strategies: **Per-tensor** — one scale per Q/K/V tensor (`q/k/v_scale = [1]`). **Per-attention-head** — one scale per head (`q_scale = [num_heads]`, `k/v_scale = [num_kv_heads]`). Per-head is only available with the Flash Attention backend via `llm-compressor`.

**CN:** 两种策略：**逐张量**——每个 Q/K/V 张量一个缩放值（`q/k/v_scale = [1]`）。**逐注意力头**——每个头一个缩放值（`q_scale = [num_heads]`，`k/v_scale = [num_kv_heads]`）。逐头量化仅可通过 `llm-compressor` 在 Flash Attention 后端中使用。

### Three Calibration Approaches / 三种校准方式

**EN:** (1) No calibration — all scales = 1.0; `kv_cache_dtype="fp8", calculate_kv_scales=False`. (2) Random token calibration — scales estimated from random tokens at warmup; `calculate_kv_scales=True`. (3) Dataset calibration (recommended) — use `llm-compressor` for most accurate scales.

**CN:** (1) 无校准——所有缩放值 = 1.0；`kv_cache_dtype="fp8", calculate_kv_scales=False`。(2) 随机 token 校准——从热身时的随机 token 估算缩放值；`calculate_kv_scales=True`。(3) 数据集校准（推荐）——使用 `llm-compressor` 获得最精确的缩放值。

### `kv_cache_dtype` Options / 选项

**EN:** `"auto"` — model default dtype. `"fp8"` / `"fp8_e4m3"` — supported on CUDA 11.8+ and ROCm. `"fp8_e5m2"` — CUDA 11.8+ only.

**CN:** `"auto"`——模型默认数据类型。`"fp8"` / `"fp8_e4m3"`——CUDA 11.8+ 和 ROCm 支持。`"fp8_e5m2"`——仅 CUDA 11.8+ 支持。

### FA3 Backend Note / FA3 后端说明

**EN:** When using Flash Attention 3 backend with FP8 KV cache, attention operations are performed in the quantized domain — queries are also quantized to FP8 during the attention call.

**CN:** 使用 Flash Attention 3 后端配合 FP8 KV 缓存时，注意力运算在量化域中执行——注意力调用期间查询也会被量化为 FP8。

## Key Concepts / 关键概念

- **EN:** KV cache quantization — storing attention K/V states in FP8 instead of BF16, reducing memory ≈2× and enabling longer contexts or higher throughput. **CN:** KV 缓存量化——以 FP8 而非 BF16 存储注意力 K/V 状态，内存约减少 2 倍，支持更长上下文或更高吞吐量。
- **EN:** `kv_cache_dtype="fp8"` — the primary parameter to enable FP8 KV cache in vLLM. **CN:** `kv_cache_dtype="fp8"`——在 vLLM 中启用 FP8 KV 缓存的主要参数。
- **EN:** `calculate_kv_scales=True` — auto-estimates scales from a warmup batch; a quick way to improve over unit scales without dataset calibration. **CN:** `calculate_kv_scales=True`——从热身批次自动估算缩放值，无需数据集校准即可快速改善单位缩放。
- **EN:** Per-attention-head quantization — finer granularity than per-tensor; requires Flash Attention backend and llm-compressor calibration. **CN:** 逐注意力头量化——比逐张量粒度更细，需要 Flash Attention 后端和 llm-compressor 校准。
- **EN:** Memory benefit — FP8 KV cache doubles the number of tokens that can be stored in the same GPU memory, directly improving throughput and max context length. **CN:** 内存收益——FP8 KV 缓存使相同 GPU 内存可存储的 token 数量翻倍，直接提升吞吐量和最大上下文长度。
