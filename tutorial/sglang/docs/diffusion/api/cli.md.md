# cli.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/api/cli.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Use the CLI for one-off generation with sglang generate or to start a persistent HTTP server with sglang serve. / 该文档围绕 SGLang Diffusion CLI 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Use the CLI for one-off generation with sglang generate or to start a persistent HTTP server with sglang serve.
**CN:** 本节围绕 Overview 展开，概述了 CLI, start, serve, server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overlay repos for non-diffusers models
**EN:** If --model-path points to a supported non-diffusers source repo, SGLang can resolve it through a self-hosted overlay repo. SGLang first checks a built-in overlay registry.
**CN:** 本节围绕 Overlay repos for non-diffusers 模型s 展开，概述了 repo, overlay, --model-path, SGLANG_DIFFUSION_MODEL_OVERLAY_REGISTRY 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Generate
**EN:** ``bash sglang generate \ --model-path Qwen/Qwen-Image \ --prompt "A beautiful sunset over the mountains" \ --save-output ``
**CN:** 本节围绕 Generate 展开，概述了 Qwen, prompt, sunset, --prompt 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Serve
**EN:** ``bash sglang serve \ --model-path Wan-AI/Wan2.1-T2V-1.3B-Diffusers \ --num-gpus 4 \ --ulysses-degree 2 \ --ring-degree 2 \ --port 30010 ` For request and response examples, see OpenAI-Compatible API.
**CN:** 本节围绕 Serve 展开，概述了 help, serve, --help, see 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Model and runtime
**EN:** This section provides a comparison table for Model and runtime, covering columns such as - `--lora-merge-mode {auto, merge, dynamic}: choose how LoRA is applied. auto` statically merges regular weights and uses dynamic LoRA for FSDP-sharded weights to avoid full-gather peaks. and examples such as - `--performance-mode {manual.
**CN:** 本节围绕 模型 and runtime 展开，概述了 auto, LoRA, parallelism, FSDP 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Sampling and output
**EN:** This content focuses on Sampling and output and highlights PATH, PROMPT, FPS, SEED.
**CN:** 本节围绕 Sampling and output 展开，概述了 PATH, PROMPT, FPS, SEED 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quantized transformers
**EN:** attention.to_) See Quantization for supported quantization families and examples.
**CN:** 本节围绕 Quantized transformers 展开，概述了 quantized, quantization, See, transformer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Configuration Files
**EN:** Use --config to load JSON or YAML configuration. Command-line flags override values from the config file.
**CN:** 本节围绕 配置 Files 展开，概述了 config, --config, Command-line, red 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Generate
**EN:** sglang generate runs a single generation job and exits when the job finishes. ` For diffusers pipelines, Cache-DiT can be enabled with SGLANG_CACHE_DIT_ENABLED=true or --cache-dit-config`.
**CN:** 本节围绕 Generate 展开，概述了 Cache-DiT, generate, See, job 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Layerwise Offload
**EN:** Use layerwise offload when a large component does not fit comfortably in GPU memory. By default, --dit-layerwise-offload only applies to legacy DiT components.
**CN:** 本节围绕 Layerwise Offload 展开，概述了 component, --dit-layerwise-offload, --layerwise-offload-components, GPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Serve
**EN:** sglang serve starts the HTTP server and keeps the model loaded for repeated requests.
**CN:** 本节围绕 Serve 展开，概述了 serve, port, keeps, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Cloud Storage
**EN:** SGLang Diffusion can upload generated images and videos to S3-compatible object storage after generation.
**CN:** 本节围绕 Cloud Storage 展开，概述了 export, See, storage, S3-compatible 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Component Path Overrides
**EN:** Override individual pipeline components such as vae, transformer, or text_encoder with --<component>-path.
**CN:** 本节围绕 Component Path Overrides 展开，概述了 component, vae, key, path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Component Attention Backend Overrides
**EN:** Use --component-attention-backends when one pipeline component needs a different native attention backend from the global --attention-backend.
**CN:** 本节围绕 Component 注意力后端 Overrides 展开，概述了 --attention-backend, --component-attention-backends, component, text_encoder 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Diffusers Backend
**EN:** Use --backend diffusers to force vanilla diffusers pipelines when no native SGLang implementation exists or when a model requires a custom pipeline class.
**CN:** 本节围绕 Diffusers Backend 展开，概述了 diffusers, force, model, class 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Key Options
**EN:** This section provides a comparison table for Key Options, covering columns such as Argument, Values, Description and examples such as --backend, --diffusers-attention-backend, --trust-remote-code, --vae-tiling and --vae-slicing.
**CN:** 本节围绕 Key Options 展开，概述了 diffusers, native, PATH, flag 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Example
**EN:** This content focuses on Example and highlights CLI, Japanese, diffusers_kwargs, pass.
**CN:** 本节围绕 Example 展开，概述了 CLI, Japanese, diffusers_kwargs, pass 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** --model-path / **CN:** --模型-path
- **EN:** model-path / **CN:** 模型-path
- **EN:** repo / **CN:** repo
- **EN:** generate / **CN:** generate
- **EN:** component / **CN:** component
- **EN:** PATH / **CN:** PATH
- **EN:** CLI / **CN:** CLI
- **EN:** diffusers / **CN:** diffusers

## Dependencies / 依赖关系
- `openai_api.md`
- `post_processing.md`
- `../quantization.md`
- `../performance/cache/cache_dit.md`
- `../environment_variables.md`
- `configs/wan_s2v.yaml`
- `config.yaml`
