# compatibility_matrix.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/compatibility_matrix.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: The table below shows every supported model and the optimizations supported for them. The symbols used have the following meanings: ✅ = Full compatibility ❌ = No compatibility. / 该文档围绕 Compatibility Matrix 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** The table below shows every supported model and the optimizations supported for them. The symbols used have the following meanings: - ✅ = Full compatibility - ❌ = No compatibility - ⭕ = Does not apply to this model
**CN:** 本节围绕 Overview 展开，概述了 Full, Does, model, supported 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Models x Optimization
**EN:** The HuggingFace Model ID can be passed directly to from_pretrained() methods, and sglang-diffusion will use the optimal default parameters when initializing and generating videos.
**CN:** 本节围绕 模型s x Optimization 展开，概述了 from_pretrained, Model, passed, videos 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Video Generation Models
**EN:** This section provides a comparison table for Video Generation Models, covering columns such as Model Name, Hugging Face Model ID, Resolutions and examples such as FastWan2.1 T2V 1.3B, FastWan2.2 TI2V 5B Full Attn, Wan2.2 TI2V 5B, Wan2.2 T2V A14B.
**CN:** 本节围绕 Video Generation 模型s 展开，概述了 T2V, TI2V, LTX-2, Wan-AI 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Image Generation Models
**EN:** This section provides a comparison table for Image Generation Models, covering columns such as Model Name, HuggingFace Model ID and examples such as FLUX.1-dev, FLUX.2-dev, FLUX.2-dev-NVFP4, FLUX.2-Klein-4B.
**CN:** 本节围绕 Image Generation 模型s 展开，概述了 Qwen, SANA, Image, Efficient-Large-Model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Components
**EN:** SGLang Diffusion supports overriding individual pipeline components with --<component>-path. The value can be either a Hugging Face repo ID or a local component directory.
**CN:** 本节围绕 Supported Components 展开，概述了 component, path, Face, repo 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Common Syntax
**EN:** This section provides a comparison table for Common Syntax, covering columns such as Component Type, Supported Keys, Notes and examples such as VAE, Transformer / DiT, Text / Preprocess, Auxiliary.
**CN:** 本节围绕 Common Syntax 展开，概述了 vae, transformer, CLI, DiT 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Known Component Repos
**EN:** This section provides a comparison table for Known Component Repos, covering columns such as Base Model, Override Key, Example Repo and examples such as black-forest-labs/FLUX.2-dev, black-forest-labs/FLUX.2-dev.
**CN:** 本节围绕 Known Component Repos 展开，概述了 vae, black-forest-labs/FLUX.2-dev, Notes, repos 等要点，并说明相关配置、流程、示例或限制条件。

### Section: VAE
**EN:** vae-path is the common image-generation override. video-vae-path and --audio-vae-path are only relevant for pipelines with separate video or audio VAEs.
**CN:** 本节围绕 VAE 展开，概述了 VAEs, --vae-path, --video-vae-path, --audio-vae-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Transformer / DiT
**EN:** transformer-path is the standard override for the main denoising transformer. For quantized transformers, prefer --transformer-path or --transformer-weights-path; see quantization.md.
**CN:** 本节围绕 Transformer / DiT 展开，概述了 --transformer-path, quantization.md, --video-dit-path, --audio-dit-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Text Encoders and Preprocessors
**EN:** text-encoder-path and --text-encoder-2-path override primary and secondary text encoders. tokenizer-path, --processor-path, and --image-processor-path are useful when the replacement encoder requires matching preprocessing assets.
**CN:** 本节围绕 Text Encoders and Preprocessors 展开，概述了 --tokenizer-path, --processor-path, --text-encoder-path, --text-encoder-2-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Auxiliary Components
**EN:** scheduler-path is only relevant when the pipeline exposes a scheduler component. spatial-upsampler-path is mainly for two-stage pipelines such as LTX2TwoStagePipeline.
**CN:** 本节围绕 Auxiliary Components 展开，概述了 LTX2TwoStagePipeline, pipelines, --vocoder-path, --scheduler-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Notes
**EN:** Component overrides are only valid when the target pipeline actually uses that component. The override key should match the component name in the pipeline's model_index.json or the native pipeline's registered module name.
**CN:** 本节围绕 Notes 展开，概述了 pipeline, name, Component, model_index.json 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Verified LoRA Examples
**EN:** This section lists example LoRAs that have been explicitly tested and verified with each base model in the **SGLang Diffusion** pipeline. > Important: > LoRAs that are not listed here are not necessarily incompatible.
**CN:** 本节围绕 Verified LoRA Examples 展开，概述了 LoRAs, been, SD-style, Important 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Verified LoRAs by Base Model
**EN:** This section provides a comparison table for Verified LoRAs by Base Model, covering columns such as Base Model, Supported LoRAs and examples such as Wan2.2, Wan2.1, Z-Image-Turbo, Qwen-Image.
**CN:** 本节围绕 Verified LoRAs by Base 模型 展开，概述了 Flux, Wan2.2, Wan2.1, XLabs-AI 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Sliding Tile Attention
**EN:** Currently, only Hopper GPUs (H100s) are supported.
**CN:** 本节围绕 Sliding Tile Attention 展开，概述了 H100s, Currently, GPUs, Hopper 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Qwen / **CN:** Qwen
- **EN:** SANA / **CN:** SANA
- **EN:** vae / **CN:** vae
- **EN:** T2V / **CN:** T2V
- **EN:** pipeline / **CN:** pipeline
- **EN:** component / **CN:** component
- **EN:** LoRAs / **CN:** LoRAs
- **EN:** Model / **CN:** 模型

## Dependencies / 依赖关系
- `quantization.md`
