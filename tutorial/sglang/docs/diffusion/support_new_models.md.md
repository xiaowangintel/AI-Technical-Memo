# support_new_models.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/support_new_models.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This document explains how to add support for new diffusion models in SGLang Diffusion. Architecture Overview SGLang Diffusion is engineered for both performance and flexibility, built upon a pipeline architecture. / 该文档围绕 How to Support New Diffusion 模型s 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This document explains how to add support for new diffusion models in SGLang Diffusion.
**CN:** 本节围绕 Overview 展开，概述了 add, new, models, support 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Architecture Overview
**EN:** SGLang Diffusion is engineered for both performance and flexibility, built upon a pipeline architecture. This design allows developers to construct pipelines for various diffusion models while keeping the core generation loop standardized for optimization.
**CN:** 本节围绕 Architecture Overview 展开，概述了 PipelineStage, process, diffusion, architecture 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Two Pipeline Styles
**EN:** This section provides a comparison table for Two Pipeline Styles, covering columns such as Stage, Ownership, Responsibility and examples such as BeforeDenoisingStage, DenoisingStage, DecodingStage, Situation.
**CN:** 本节围绕 Two Pipeline Styles 展开，概述了 Modular, Style, Model, Hybrid 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Key Components for Implementation
**EN:** To add support for a new diffusion model, you will need to define or configure the following components: 1.
**CN:** 本节围绕 Key Components for Implementation 展开，概述了 model, DecodingStage, DenoisingStage, See 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Core Stages (used by all pipelines)
**EN:** This section provides a comparison table for Core Stages (used by all pipelines), covering columns such as Stage Class, Description and examples such as DenoisingStage, DecodingStage, DmdDenoisingStage, CausalDMDDenoisingStage.
**CN:** 本节围绕 Core Stages (used by all pipelines) 展开，概述了 denoising, DecodingStage, DenoisingStage, DmdDenoisingStage 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Pre-processing Stages (for Modular Composition Style)
**EN:** This section provides a comparison table for Pre-processing Stages (for Modular Composition Style), covering columns such as Stage Class, Description and examples such as InputValidationStage, TextEncodingStage, ImageEncodingStage, ImageVAEEncodingStage.
**CN:** 本节围绕 Pre-processing Stages (for Modular Composition Style) 展开，概述了 Encodes, SamplingParams, TextEncodingStage, ImageEncodingStage 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 1: Obtain and Study the Reference Implementation
**EN:** Find the model's model_index.json to identify required modules.
**CN:** 本节围绕 Step 1: Obtain and Study the Reference Implementation 展开，概述了 model, Diffusers, pipeline, HuggingFace 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 2: Evaluate Reuse of Existing Pipelines and Stages
**EN:** Before creating any new files, check whether an existing pipeline or stage can be reused or extended. Only create new pipelines/stages when the existing ones would need substantial structural changes or when no architecturally similar implementation exists.
**CN:** 本节围绕 Step 2: Evaluate Reuse of Existing Pipelines and Stages 展开，概述了 existing, new, Check, AutoencoderKL 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 3: Implement Model Components
**EN:** **Tensor Parallel (TP) and Sequence Parallel (SP)**: For multi-GPU deployment, it is recommended to add TP/SP support to the DiT model. This can be done incrementally after the single-GPU implementation is verified.
**CN:** 本节围绕 Step 3: Implement 模型 Components 展开，概述了 Implement, model, DiT, USPAttention 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 4: Create Configs
**EN:** **DiT Config**: configs/models/dits/ .py - **VAE Config**: configs/models/vaes/ .py - **SamplingParams**: configs/sample/ .py
**CN:** 本节围绕 Step 4: Create Configs 展开，概述了 model_name, Config, SamplingParams, DiT 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 5: Create PipelineConfig
**EN:** This content focuses on Step 5: Create PipelineConfig and highlights def, self, Build, kwargs.
**CN:** 本节围绕 Step 5: Create PipelineConfig 展开，概述了 def, self, Build, kwargs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 6: Implement Pre-processing
**EN:** This section provides a comparison table for Step 6: Implement Pre-processing, covering columns such as Field, Type, Description and examples such as batch.latents, batch.timesteps, batch.num_inference_steps, batch.sigmas.
**CN:** 本节围绕 Step 6: Implement Pre-processing 展开，概述了 list, torch.Tensor, DenoisingStage, Req 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 7: Define the Pipeline Class
**EN:** Monolithic pre-processing (model-specific) self.add_stage( MyModelBeforeDenoisingStage( vae=self.get_module("vae"), text_encoder=self.get_module("text_encoder"), tokenizer=self.get_module("tokenizer"), transformer=self.get_module("transformer"), scheduler=self.get_module("scheduler"), ), ) # 2.
**CN:** 本节围绕 Step 7: Define the Pipeline Class 展开，概述了 MyModelPipeline, self.get_module, scheduler, transformer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 8: Register the Model
**EN:** This content focuses on Step 8: Register the Model and highlights Register, pipeline, EntryClass, registry.py.
**CN:** 本节围绕 Step 8: Register the 模型 展开，概述了 Register, pipeline, EntryClass, registry.py 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 9: Verify Output Quality
**EN:** After implementation, verify that the generated output is not noise. A noisy or garbled output is the most common sign of an incorrect implementation.
**CN:** 本节围绕 Step 9: Verify Output Quality 展开，概述了 After, Wrong, Debug, Common 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Hybrid Style
**EN:** This section provides a comparison table for Hybrid Style, covering columns such as Model, Pipeline, BeforeDenoisingStage and examples such as GLM-Image, Qwen-Image-Layered.
**CN:** 本节围绕 Hybrid Style 展开，概述了 Model, Pipeline, GLM-Image, PipelineConfig 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Modular Style
**EN:** This section provides a comparison table for Modular Style, covering columns such as Model, Pipeline, Notes and examples such as Qwen-Image (T2I), Qwen-Image-Edit, Flux, Wan.
**CN:** 本节围绕 Modular Style 展开，概述了 Uses, add_standard_t2i_stages, runtime/pipelines/qwen_image.py, T2I 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Checklist
**EN:** This content focuses on Checklist and highlights DiT, DenoisingStage, model_name, PipelineConfig.
**CN:** 本节围绕 Checklist 展开，概述了 DiT, DenoisingStage, model_name, PipelineConfig 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** model / **CN:** 模型
- **EN:** DenoisingStage / **CN:** DenoisingStage
- **EN:** Step / **CN:** Step
- **EN:** DecodingStage / **CN:** DecodingStage
- **EN:** DiT / **CN:** DiT
- **EN:** PipelineConfig / **CN:** PipelineConfig
- **EN:** Diffusers / **CN:** Diffusers
- **EN:** pipeline / **CN:** pipeline

## Dependencies / 依赖关系
- `pipeline_*.py`
- `runtime/models/dits/wanvideo.py`
- `runtime/models/dits/qwen_image.py`
- `configs/models/dits/{model_name}.py`
- `configs/models/vaes/{model_name}.py`
- `configs/sample/{model_name}.py`
- `runtime/pipelines/glm_image.py`
- `stages/model_specific_stages/glm_image.py`
- `configs/pipeline_configs/glm_image.py`
- `runtime/pipelines/qwen_image.py`
- `stages/model_specific_stages/qwen_image_layered.py`
- `configs/pipeline_configs/qwen_image.py`
