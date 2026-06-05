# fp8_vit_attn.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/fp8_vit_attn.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Focuses on a specific multimodal bottleneck: ViT encoder attention for large-image understanding, especially when the text side is already quantized and attention becomes the dominant cost.
  **CN:** 该文档聚焦于一个特定多模态瓶颈：大图像理解场景下的 ViT 编码器注意力，尤其是文本侧已量化后，注意力计算会成为主要开销。
- **EN:** The page is strongly operational: it lists model/backend constraints, shows the enabling flags, then recommends a calibrate-once-and-reuse workflow that converts dynamic runtime scaling into reusable static scale files.
  **CN:** 文档具有很强的操作性：先列出模型与后端约束，再给出启用参数，随后推荐“一次校准、重复复用”的流程，把动态缩放结果沉淀为可复用的静态 scale 文件。
- **EN:** Performance and accuracy sections make the trade-off concrete: FP8 helps mainly at QHD/4K or multi-image settings, while measured accuracy stays close to BF16 within normal variance.
  **CN:** 性能与精度部分把权衡讲得很具体：FP8 主要在 QHD/4K 或多图请求下带来收益，而测得精度与 BF16 基本处于统计波动范围内。

## Key Concepts / 关键概念
- **EN:** Scope limitation: support is currently limited to Qwen3-VL family models and Qwen3-ViT-based variants.
  **CN:** 适用范围限制：目前仅支持 Qwen3-VL 系列及使用 Qwen3 ViT 的相关模型。
- **EN:** Backend requirement: FP8 ViT attention depends on the FlashInfer cuDNN backend with cuDNN >= 9.17.1.
  **CN:** 后端要求：FP8 ViT 注意力依赖 FlashInfer cuDNN 后端，且需要 cuDNN >= 9.17.1。
- **EN:** Dynamic vs static scales: dynamic scaling is default and calibration-free, while static scales remove runtime overhead after an offline calibration pass.
  **CN:** 动态与静态缩放：动态缩放默认开启且无需校准，静态缩放则通过离线校准换取运行时更低开销。
- **EN:** Scale file contract: saved JSON stores per-attention-module Q/K/V scales, and alias keys such as `q_scale` are accepted.
  **CN:** scale 文件约定：保存的 JSON 按注意力模块记录 Q/K/V 缩放值，并兼容 `q_scale` 等别名。
- **EN:** Performance crossover: smaller images may not benefit, but QHD-and-above workloads show the intended FP8 speedup.
  **CN:** 性能拐点：较小图像未必受益，但在 QHD 及以上分辨率中更能体现 FP8 加速效果。
