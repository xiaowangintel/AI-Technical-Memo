# online.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/online.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This page documents load-time online quantization: a BF16/FP16 model can be quantized while loading, without a pre-quantized checkpoint or offline calibration dataset.
- **CN:** 本页介绍加载时在线量化：BF16/FP16 模型在加载过程中即可完成量化，无需预量化检查点，也不需要离线校准数据集。
- **EN:** The document first gives quick-start examples for `fp8_per_tensor`, `fp8_per_block`, and `mxfp8` in both Python and CLI forms, then explains the supported scheme table and hardware caveats such as SM100+/Blackwell requirements for full MXFP8 support.
- **CN:** 文档先用 Python 与 CLI 示例展示 `fp8_per_tensor`、`fp8_per_block`、`mxfp8` 的快速用法，再通过表格解释各方案细节以及 MXFP8 对 SM100+/Blackwell 的硬件要求等限制。
- **EN:** Its advanced section is especially important: `quantization_config` can split dense linear and MoE policies, override activation handling for some pre-quantized checkpoints, and exclude layers by exact names or regex—with a caveat that fused layers must be matched by unfused shard names.
- **CN:** 高级部分尤其关键：`quantization_config` 可分别指定稠密层与 MoE 层策略、为部分已量化检查点覆写激活格式，并按精确名称或正则排除层；但对融合层，匹配对象必须是未融合的分片名称。

## Key Concepts / 关键概念
- **EN:** Online quantization converts weights during model loading and dynamically scales activations during forward passes, reducing preparation work for deployment.
  **CN:** 在线量化在模型加载时转换权重、在前向过程中动态缩放激活，从而减少部署前的准备步骤。
- **EN:** The built-in shorthands cover per-tensor FP8, per-block FP8, and MXFP8, with clear Python and CLI entry points.
  **CN:** 内置简写方案覆盖 per-tensor FP8、per-block FP8 与 MXFP8，并提供清晰的 Python 与 CLI 入口。
- **EN:** `quantization_config` supports fine-grained control over `linear`, `moe`, and `ignore`, so dense layers and expert layers can use different recipes.
  **CN:** `quantization_config` 支持对 `linear`、`moe` 和 `ignore` 做细粒度控制，因此稠密层与专家层可以采用不同方案。
- **EN:** For some checkpoint-quantized models, activations can be overridden independently of stored weights, which is a useful integration hook for MoE kernels and backend tuning.
  **CN:** 对部分已量化检查点，可在不改变权重格式的前提下单独覆写激活格式，这为 MoE 内核选择和后端调优提供了实用接口。

