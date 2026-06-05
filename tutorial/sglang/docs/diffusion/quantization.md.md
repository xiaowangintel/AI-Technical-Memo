# quantization.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/quantization.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang-Diffusion supports quantized transformer checkpoints. In most cases, keep the base model and the quantized transformer override separate. / 该文档围绕 量化 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang-Diffusion supports quantized transformer checkpoints. In most cases, keep the base model and the quantized transformer override separate.
**CN:** 本节围绕 Overview 展开，概述了 quantized, transformer, SGLang-Diffusion, most 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quick Reference
**EN:** If a repo contains multiple candidate checkpoints, pass --transformer-weights-path` explicitly.
**CN:** 本节围绕 Quick Reference 展开，概述了 --model-path, quantized, --transformer-weights-path, model-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quant Families
**EN:** This section provides a comparison table for Quant Families, covering columns such as quant_family, checkpoint form, canonical CLI and examples such as fp8 / mxfp4 (online quantization), fp8 (offline quantization), modelopt-fp8, modelopt-nvfp4.
**CN:** 本节围绕 Quant Families 展开，概述了 fp8, Wan2.2, --transformer-path, --transformer-weights-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Online Quantization
**EN:** Online quantization applies quantization to unquantized models at load time. This is useful for when pre-quantized checkpoints are not available.
**CN:** 本节围绕 Online 量化 展开，概述了 Online, quantization, load, time 等要点，并说明相关配置、流程、示例或限制条件。

### Section: FP8 Online Quantization
**EN:** Apply FP8 quantization to any unquantized model: ``bash sglang generate \ --model-path Tongyi-MAI/Z-Image-Turbo \ --quantization fp8 \ --prompt "a beautiful sunset" \ --save-output ``
**CN:** 本节围绕 FP8 Online 量化 展开，概述了 quantization, FP8, Apply, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: MXFP4 Online Quantization
**EN:** MXFP4 provides aggressive 4-bit compression with online quantization.
**CN:** 本节围绕 MXFP4 Online 量化 展开，概述了 Note, MXFP4, Requires, GPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Skipping Layers
**EN:** By default, online quantization quantizes every linear layer in the transformer. layers.0.attention.to_q`).
**CN:** 本节围绕 Skipping Layers 展开，概述了 --quantization-ignored-layers, layer, quantization, quantization-ignored-layers 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Validated ModelOpt Checkpoints
**EN:** This section provides a comparison table for Validated ModelOpt Checkpoints, covering columns such as Quant Algo, Base Model, Preferred CLI and examples such as FP8, FP8, FP8, FP8.
**CN:** 本节围绕 Validated 模型Opt Checkpoints 展开，概述了 FP8, --transformer-path, NVFP4, BF16 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage Examples
**EN:** Converted ModelOpt FP8 checkpoints should be loaded as transformer component overrides. If the repo or local directory already contains config.json, use --transformer-path.
**CN:** 本节围绕 Usage Examples 展开，概述了 --transformer-path, transformer-path, prompt, --prompt 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Notes
**EN:** transformer-path is the canonical flag for converted ModelOpt FP8 transformer component repos or directories that already carry config.json.
**CN:** 本节围绕 Notes 展开，概述了 FP8, ModelOpt, BF16, layers 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage Examples
**EN:** This content focuses on Usage Examples and highlights --model-path, NVFP4, model-path, prompt.
**CN:** 本节围绕 Usage Examples 展开，概述了 --model-path, NVFP4, model-path, prompt 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Notes
**EN:** Use --transformer-path for mixed ModelOpt NVFP4 transformer repos or local directories that already include config.json. Use --transformer-weights-path for raw NVFP4 exports, individual safetensors files, or repo layouts that should be treated as weights first.
**CN:** 本节围绕 Notes 展开，概述了 NVFP4, SGLANG_DIFFUSION_FLASHINFER_FP4_GEMM_BACKEND, path, override 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Install
**EN:** Install the runtime dependency first: ``bash pip install nunchaku `` For platform-specific installation methods and troubleshooting, see the Nunchaku installation guide.
**CN:** 本节围绕 Install 展开，概述了 Install, Nunchaku, installation, pip 等要点，并说明相关配置、流程、示例或限制条件。

### Section: File Naming and Auto-Detection
**EN:** This section provides a comparison table for File Naming and Auto-Detection, covering columns such as checkpoint name fragment, inferred precision, inferred rank and examples such as svdq-int4_r32, svdq-int4_r128, svdq-fp4_r32, svdq-fp4_r128.
**CN:** 本节围绕 File Naming and Auto-Detection 展开，概述了 int4, nvfp4, 128, checkpoint 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage Examples
**EN:** This content focuses on Usage Examples and highlights Qwen, Manual, prompt, sunset.
**CN:** 本节围绕 Usage Examples 展开，概述了 Qwen, Manual, prompt, sunset 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Notes
**EN:** transformer-weights-path is the canonical flag for Nunchaku checkpoints. Older config names such as quantized_model_path are treated as compatibility aliases.
**CN:** 本节围绕 Notes 展开，概述了 Nunchaku, fp4, int4, SM8x 等要点，并说明相关配置、流程、示例或限制条件。

### Section: ModelSlim
**EN:** MindStudio-ModelSlim (msModelSlim) is a model offline quantization compression tool launched by MindStudio and optimized for Ascend hardware.
**CN:** 本节围绕 模型Slim 展开，概述了 quantization, Ascend, model, Wan2.2 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** FP8 / **CN:** FP8
- **EN:** --transformer-path / **CN:** --transformer-path
- **EN:** NVFP4 / **CN:** NVFP4
- **EN:** --model-path / **CN:** --模型-path
- **EN:** quantization / **CN:** 量化
- **EN:** --transformer-weights-path / **CN:** --transformer-weights-path
- **EN:** transformer / **CN:** transformer
- **EN:** model-path / **CN:** 模型-path

## Dependencies / 依赖关系
- `build_modelopt_nvfp4_transformer.py`
- `python/sglang/multimodal_gen/tools/wan_repack.py`
- `../platforms/ascend/ascend_npu_quantization.md`
- `//gitcode.com/Ascend/msmodelslim/blob/master/example/multimodal_sd/README.md`
- `wan_repack.py`
