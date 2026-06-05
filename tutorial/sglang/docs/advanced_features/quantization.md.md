# quantization.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/quantization.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang supports various quantization methods, including offline quantization and online dynamic quantization. Offline quantization loads pre-quantized model weights directly during inference. / 该文档围绕 量化 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang supports various quantization methods, including offline quantization and online dynamic quantization. Offline quantization loads pre-quantized model weights directly during inference.
**CN:** 本节围绕 Overview 展开，概述了 quantization, online, weights, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Platform Compatibility
**EN:** This section provides a comparison table for Platform Compatibility, covering columns such as Method, NVIDIA GPUs, AMD GPUs (MI300X/MI325X/MI350X) and examples such as fp8, mxfp4, blockwise_int8, w8a8_int8.
**CN:** 本节围绕 Platform Compatibility 展开，概述了 Yes, AMD, Ascend, NVIDIA 等要点，并说明相关配置、流程、示例或限制条件。

### Section: GEMM Backends for FP4/FP8 Quantization
**EN:** ::: Backend selection is supported only for **blockwise FP8** and **NVFP4** GEMM. When running FP8 or FP4 quantized models, you can select the GEMM backend via --fp8-gemm-backend and --fp4-gemm-backend.
**CN:** 本节围绕 GEMM Backends for FP4/FP8 量化 展开，概述了 FP8, GEMM, FP4, NVFP4 等要点，并说明相关配置、流程、示例或限制条件。

### Section: --fp8-gemm-backend (Blockwise FP8 GEMM)
**EN:** This section provides a comparison table for --fp8-gemm-backend (Blockwise FP8 GEMM), covering columns such as Backend, Hardware, Description and examples such as auto, deep_gemm, flashinfer_trtllm, flashinfer_cutlass.
**CN:** 本节围绕 --fp8-gemm-backend (Blockwise FP8 GEMM) 展开，概述了 SM100, SM90, CUTLASS, FlashInfer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: --fp4-gemm-backend (NVFP4 GEMM)
**EN:** This section provides a comparison table for --fp4-gemm-backend (NVFP4 GEMM), covering columns such as Backend, Hardware, Description and examples such as auto, cutlass, flashinfer_cutlass, flashinfer_cudnn.
**CN:** 本节围绕 --fp4-gemm-backend (NVFP4 GEMM) 展开，概述了 SM100, FlashInfer, SM120, SM100/120 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Offline Quantization
**EN:** To load already quantized models, simply load the model weights and config. **Again, if the model has been quantized offline, there's no need to add --quantization argument when starting the engine.
**CN:** 本节围绕 Offline 量化 展开，概述了 quantization, --quantization, FP8, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Examples of Offline Model Quantization
**EN:** #### Using Unsloth We strongly suggest the use of Unsloth to quantize and load the model. Please refer to SGLang Deployment & Inference Guide with Unsloth.
**CN:** 本节围绕 Examples of Offline 模型 量化 展开，概述了 quantization, model, ModelOpt, import 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Online Quantization
**EN:** To enable online quantization, you can simply specify --quantization in the command line. SGLang will soon support methods including but not limited to ["awq", "gptq", "marlin", "gptq_marlin", "awq_marlin", "bitsandbytes", "gguf"]`.
**CN:** 本节围绕 Online 量化 展开，概述了 quantization, FP8, --quantization, meta-llama/Meta-Llama-3.1-8B-Instruct 等要点，并说明相关配置、流程、示例或限制条件。

### Section: torchao online quantization method
**EN:** SGLang also supports quantization methods based on torchao. You can simply specify --torchao-config in the command line to support this feature.
**CN:** 本节围绕 torchao online 量化 method 展开，概述了 int8dq, int4wo-128, --torchao-config, meta-llama/Meta-Llama-3.1-8B-Instruct 等要点，并说明相关配置、流程、示例或限制条件。

### Section: quark_int4fp8_moe online quantization method
**EN:** projections in the attention layers) have their weights quantized online to float8 directly.
**CN:** 本节围绕 quark_int4fp8_moe online 量化 method 展开，概述了 float8, layers, quantized, MoE 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Diffusion Model Quantization on Ascend NPU
**EN:** SGLang-Diffusion supports MXFP8 quantization for diffusion models (such as Wan2.2) on Ascend A5 NPUs, in both online and offline (ModelSlim) modes. This is separate from the LLM serving path and uses the sglang serve / sglang generate CLI.
**CN:** 本节围绕 Diffusion 模型 量化 on Ascend NPU 展开，概述了 Ascend, LLM, CLI, RC3 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Online MXFP8
**EN:** Pass --quantization mxfp8 to dynamically quantize FP16/BF16 transformer weights to MXFP8 at load time: ``bash sglang serve \ --model-path Wan-AI/Wan2.2-T2V-A14B-Diffusers \ --quantization mxfp8 \ --num-gpus 4 ``
**CN:** 本节围绕 Online MXFP8 展开，概述了 Pass, MXFP8, quantization, --quantization 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Offline MXFP8 (ModelSlim)
**EN:** This content focuses on Offline MXFP8 (ModelSlim) and highlights Ascend, Pre-quantize, quantization, quant_model_description.json.
**CN:** 本节围绕 Offline MXFP8 (模型Slim) 展开，概述了 Ascend, Pre-quantize, quantization, quant_model_description.json 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Reference
**EN:** GPTQModel - LLM Compressor - NVIDIA Model Optimizer (ModelOpt) - NVIDIA Model Optimizer LLM PTQ - Petit: NVFP4 on ROCm — LMSYS blog, AMD ROCm blog - Torchao: PyTorch Architecture Optimization - vLLM Quantization - auto-round - ModelSlim
**CN:** 本节围绕 Reference 展开，概述了 NVIDIA, ROCm, GPTQModel, LLM 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** quantization / **CN:** 量化
- **EN:** Yes / **CN:** Yes
- **EN:** model / **CN:** 模型
- **EN:** Ascend / **CN:** Ascend
- **EN:** NVIDIA / **CN:** NVIDIA
- **EN:** FP8 / **CN:** FP8
- **EN:** ModelOpt / **CN:** 模型Opt
- **EN:** AMD / **CN:** AMD

## Dependencies / 依赖关系
- `../platforms/amd_gpu.md`
- `../platforms/ascend/ascend_npu_quantization.md`
- `hf_ptq.py`
- `../diffusion/quantization.md`
- `modelopt_quantize_and_export.py`
- `//github.com/sgl-project/sglang/blob/main/examples/usage/modelopt_quantize_and_export.py`
- `examples/usage/modelopt_quantize_and_export.py`
- `//github.com/sgl-project/sglang/blob/main/python/sglang/srt/configs/load_config.py`
- `qwen3_5_moe_w8a8.yaml`
- `//gitcode.com/Ascend/msmodelslim/blob/master/example/README.md`
- `//github.com/sgl-project/sglang/blob/v0.4.8/python/sglang/srt/layers/moe/fused_moe_triton/layer.py`
