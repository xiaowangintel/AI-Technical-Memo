# ascend_npu_quantization.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/ascend/ascend_npu_quantization.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: To load already quantized models, simply load the model weights and config. Again, if the model has been quantized offline, there's no need to add --quantization argument when starting the engine. / 该文档围绕 量化 on Ascend 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This section provides a comparison table for Overview, covering columns such as Quantization scheme, quant_type in JSON, Scheme class and examples such as W4A4 dynamic, W8A8 static, W8A8 dynamic, MXFP8.
**CN:** 本节围绕 Overview 展开，概述了 span, style, color, green 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Diffusion Model Quantization on Ascend NPU
**EN:** This section provides a comparison table for Diffusion Model Quantization on Ascend NPU, covering columns such as Quantization method, quant_type in JSON, Scheme class and examples such as MXFP8 (W8A8), MXFP8 (W8A8), W8A8 static, W8A8 dynamic.
**CN:** 本节围绕 Diffusion 模型 量化 on Ascend NPU 展开，概述了 span, W8A8, style, color 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Online MXFP8 Quantization
**EN:** Online quantization dynamically quantizes FP16/BF16 weights to MXFP8 at load time using npu_dynamic_mx_quant + npu_quant_matmul CANN kernels. Pass --quantization mxfp8 to override auto-detection.
**CN:** 本节围绕 Online MXFP8 量化 展开，概述了 quantization, MXFP8, --quantization, CANN 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Offline MXFP8 Quantization (ModelSlim)
**EN:** For offline quantization, pre-quantize the model with msModelSlim and load the resulting checkpoint. The quantization scheme is auto-detected from quant_model_description.json, so no extra --quantization flag is needed.
**CN:** 本节围绕 Offline MXFP8 量化 (模型Slim) 展开，概述了 Step, Diffusers, Convert, quantization 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** span / **CN:** span
- **EN:** style / **CN:** style
- **EN:** color / **CN:** color
- **EN:** green / **CN:** green
- **EN:** TBD / **CN:** TBD
- **EN:** Supported / **CN:** Supported
- **EN:** MoE / **CN:** MoE
- **EN:** yellow / **CN:** yellow

## Dependencies / 依赖关系
- `python/sglang/multimodal_gen/tools/wan_repack.py`
