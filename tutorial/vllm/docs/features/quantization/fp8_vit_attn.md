# fp8_vit_attn.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/fp8_vit_attn.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document describes optional FP8 quantization for the ViT (Vision Transformer) encoder attention in vLLM, targeting multimodal workloads with large images (QHD/4K). It uses the FlashInfer cuDNN backend to quantize Q/K/V on-the-fly before the attention kernel.

**CN:** 本文档介绍 vLLM 中 ViT（视觉 Transformer）编码器注意力的可选 FP8 量化，面向大分辨率图像（QHD/4K）的多模态工作负载，使用 FlashInfer cuDNN 后端在注意力内核调用前即时量化 Q/K/V。

### Use Case / 适用场景

**EN:** Beneficial when text model is already quantized (e.g., NVFP4) and ViT attention becomes the bottleneck. Gains are significant at QHD (1440×2560) and above; smaller images may see no speedup due to quantization kernel overhead.

**CN:** 当文本模型已量化（如 NVFP4）且 ViT 注意力成为瓶颈时效果显著。在 QHD（1440×2560）及更高分辨率时收益明显，较小图像因量化内核开销可能无加速。

### Requirements / 要求

**EN:** FlashInfer cuDNN backend with cuDNN ≥ 9.17.1. Currently supports Qwen3-VL family models only.

**CN:** 需要 FlashInfer cuDNN 后端且 cuDNN ≥ 9.17.1，目前仅支持 Qwen3-VL 系列模型。

### Dynamic vs Static Scaling / 动态与静态缩放

**EN:** **Dynamic** (default): a 16-entry circular buffer of observed Q/K/V amax values drives per-forward scale updates — no calibration needed but adds per-forward overhead. **Static**: calibrate once with `vllm bench mm-processor` to save a scale JSON file, then serve with `--mm-encoder-fp8-scale-path` for zero overhead.

**CN:** **动态**（默认）：16 条目循环缓冲区追踪观测到的 Q/K/V amax 值并按前向传播更新缩放比例——无需校准但每次前向传播有额外开销。**静态**：使用 `vllm bench mm-processor` 一次性校准并保存缩放 JSON 文件，通过 `--mm-encoder-fp8-scale-path` 以零开销提供服务。

### Performance Numbers / 性能数据

**EN:** Core cuDNN kernel speedup on GB300: 1.42× at seq_len=8192. End-to-end encoder speedup (Qwen3-VL-30B, 3 images): 1.08× at QHD, 1.18× at 4K. HD resolution shows a slight slowdown due to overhead.

**CN:** GB300 上核心 cuDNN 内核加速：seq_len=8192 时 1.42 倍。端到端编码器加速（Qwen3-VL-30B，3 张图像）：QHD 时 1.08 倍，4K 时 1.18 倍。HD 分辨率因开销略有减速。

### Scale File Format / 缩放文件格式

**EN:** JSON mapping layer names to `{"q": float, "k": float, "v": float}` scale values. Aliases `q_scale`, `k_scale`, `v_scale` are also accepted.

**CN:** JSON 格式，将层名称映射到 `{"q": float, "k": float, "v": float}` 缩放值，也接受别名 `q_scale`、`k_scale`、`v_scale`。

## Key Concepts / 关键概念

- **EN:** On-the-fly FP8 quantization — Q/K/V are quantized to FP8 just before the cuDNN attention kernel, then results are dequantized. **CN:** 即时 FP8 量化——Q/K/V 在 cuDNN 注意力内核调用前即时量化为 FP8，结果再反量化。
- **EN:** `--mm-encoder-attn-backend FLASHINFER --mm-encoder-attn-dtype fp8` — the two flags required to enable this feature. **CN:** `--mm-encoder-attn-backend FLASHINFER --mm-encoder-attn-dtype fp8`——启用该功能所需的两个标志。
- **EN:** Calibration scale margin — default 1.5× multiplier on saved scales to handle activation outliers not present in calibration set. **CN:** 校准缩放余量——对保存的缩放值默认乘以 1.5 倍，以应对校准集中未出现的激活异常值。
- **EN:** Resolution crossover — FP8 ViT attention is only beneficial at FullHD and above (roughly 1080p or higher). **CN:** 分辨率分界点——FP8 ViT 注意力仅在 FullHD 及以上（约 1080p 或更高）分辨率时有益。
- **EN:** Accuracy preservation — static scales calibrated on one dataset (VisionArena-Chat) generalise accurately to another (ChartQA). **CN:** 精度保持——在一个数据集（VisionArena-Chat）上校准的静态缩放可准确泛化到另一个数据集（ChartQA）。
