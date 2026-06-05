# README.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/README.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This page is the hub for vLLM quantization documentation. It introduces quantization at a high level, recommends LLM Compressor as the starting point, and links out to the supported quantization method pages.
- **CN:** 本页是 vLLM 量化文档的总入口。它先从高层解释量化，再把 LLM Compressor 推荐为起点，并链接到各类受支持的量化方法页面。
- **EN:** A major section is the hardware compatibility matrix, which maps implementations to NVIDIA generations, AMD GPU, Intel GPU, and x86 CPU, then adds caveats for Turing/Marlin, Intel Gaudi migration, and TPU-specific documentation.
- **CN:** 重要主体是硬件兼容矩阵，它把不同实现映射到各代 NVIDIA GPU、AMD GPU、Intel GPU 与 x86 CPU，并补充了 Turing/Marlin、Intel Gaudi 迁移和 TPU 文档等说明。
- **EN:** The latter half shifts from user-facing formats to extension mechanics by explaining out-of-tree quantization plugins, required `QuantizationConfig` methods, and how custom Linear or MoE quantization methods plug into vLLM.
- **CN:** 后半部分则从用户视角的量化格式转向扩展机制，讲解树外量化插件、`QuantizationConfig` 必需方法，以及自定义 Linear/MoE 量化方法如何接入 vLLM。

## Key Concepts / 关键概念
- **EN:** The page separates discovery from implementation: first it catalogs built-in quantization options, then it explains how developers can register new ones.
  **CN:** 该页把“发现能力”和“实现扩展”分开：前半部分罗列内置量化方案，后半部分说明开发者如何注册新方案。
- **EN:** The compatibility table is an operational reference for hardware planning, not just a feature list, because it makes platform limits visible at a glance.
  **CN:** 兼容性表格不仅是功能清单，更是硬件规划参考，因为它能直观看出平台限制。
- **EN:** Custom quantization support relies on `@register_quantization_config`, a `QuantizationConfig` subclass, and layer-type dispatch through `get_quant_method`.
  **CN:** 自定义量化支持依赖 `@register_quantization_config`、`QuantizationConfig` 子类，以及通过 `get_quant_method` 做层类型分发。
- **EN:** The examples show that extension points cover both standard Linear layers and Fused MoE layers, making the plugin system relevant to modern expert models.
  **CN:** 示例表明扩展点同时覆盖普通 Linear 层和 Fused MoE 层，因此该插件系统对现代专家模型也同样适用。

