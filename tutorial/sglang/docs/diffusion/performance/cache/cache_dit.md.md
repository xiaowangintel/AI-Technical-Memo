# cache_dit.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/performance/cache/cache_dit.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang integrates Cache-DiT, a caching acceleration engine for Diffusion Transformers (DiT), to achieve up to **1.69x inference speedup** with minimal quality loss. / 该文档围绕 缓存-DiT 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang integrates Cache-DiT, a caching acceleration engine for Diffusion Transformers (DiT), to achieve up to **1.69x inference speedup** with minimal quality loss.
**CN:** 本节围绕 Overview 展开，概述了 DiT, Cache-DiT, loss, engine 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overview
**EN:** This content focuses on Overview and highlights caching, SCM, DBCache, Cache-DiT.
**CN:** 本节围绕 Overview 展开，概述了 caching, SCM, DBCache, Cache-DiT 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Basic Usage
**EN:** Enable Cache-DiT by exporting the environment variable and using sglang generate or sglang serve : ``bash SGLANG_CACHE_DIT_ENABLED=true \ sglang generate --model-path Qwen/Qwen-Image \ --prompt "A beautiful sunset over the mountains" ``
**CN:** 本节围绕 Basic Usage 展开，概述了 generate, SGLANG_CACHE_DIT_ENABLED, Qwen, serve 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Diffusers Backend
**EN:** Cache-DiT supports loading acceleration configs from a custom YAML file. For diffusers pipelines (diffusers backend), pass the YAML/JSON path via --cache-dit-config.
**CN:** 本节围绕 Diffusers Backend 展开，概述了 diffusers, Cache-DiT, --cache-dit-config, cache_dit.load_configs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Single GPU inference
**EN:** The SCM will automatically # generate the steps computation mask based on the num_inference_steps.
**CN:** 本节围绕 Single GPU inference 展开，概述了 SCM, DBCache, TaylorSeer, Fn_compute_blocks 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Distributed inference
**EN:** 1D Parallelism Define a parallelism only config yaml parallel.yaml file that contains: ``yaml parallelism_config: ulysses_size: auto attention_backend: native ` Then, apply the distributed inference acceleration config from yaml.
**CN:** 本节围绕 Distributed inference 展开，概述了 config, parallelism, ulysses_size, Then 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Hybrid Cache and Parallelism
**EN:** `bash sglang generate \ --backend diffusers \ --num-gpus 4 \ --model-path Qwen/Qwen-Image \ --cache-dit-config hybrid.yaml \ --prompt "A beautiful sunset over the mountains" ``
**CN:** 本节围绕 Hybrid 缓存 and 并行策略 展开，概述了 hybrid.yaml, Then, cache, Define 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Attention Backend
**EN:** In some cases, users may want to only specify the attention backend without any other optimization configs. In this case, you can define a yaml file attention.yaml that only contains: ``yaml attention_backend: "flash" # '_flash_3' for Hopper ``
**CN:** 本节围绕 注意力后端 展开，概述了 Hopper, attention.yaml, may, want 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quantization
**EN:** You can also specify the quantization config in the yaml file, required torchao>=0.16.0.
**CN:** 本节围绕 量化 展开，概述了 quantization, layers, quantize.yaml, Then 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Combined Configs: Cache + Parallelism + Quantization
**EN:** Please also enable torch.compile for better performance if you are using quantization.
**CN:** 本节围绕 Combined Configs: 缓存 + 并行策略 + 量化 展开，概述了 Then, Please, quantization, combined.yaml 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DBCache Parameters
**EN:** This section provides a comparison table for DBCache Parameters, covering columns such as Parameter, Env Variable, Default and examples such as Fn, Bn, W, R.
**CN:** 本节围绕 DB缓存 Parameters 展开，概述了 Number, SGLANG_CACHE_DIT_FN, SGLANG_CACHE_DIT_BN, SGLANG_CACHE_DIT_MC 等要点，并说明相关配置、流程、示例或限制条件。

### Section: TaylorSeer Configuration
**EN:** This section provides a comparison table for TaylorSeer Configuration, covering columns such as Parameter, Env Variable, Default and examples such as Enable, Order.
**CN:** 本节围绕 TaylorSeer 配置 展开，概述了 Taylor, Enable, TaylorSeer, SGLANG_CACHE_DIT_TS_ORDER 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Combined Configuration Example
**EN:** This content focuses on Combined Configuration Example and highlights DBCache, TaylorSeer, SGLANG_CACHE_DIT_FN, SGLANG_CACHE_DIT_BN.
**CN:** 本节围绕 Combined 配置 Example 展开，概述了 DBCache, TaylorSeer, SGLANG_CACHE_DIT_FN, SGLANG_CACHE_DIT_BN 等要点，并说明相关配置、流程、示例或限制条件。

### Section: SCM (Step Computation Masking)
**EN:** This section provides a comparison table for SCM (Step Computation Masking), covering columns such as Preset, Compute Ratio, Speed and examples such as none, slow, medium, fast.
**CN:** 本节围绕 SCM (Step Computation Masking) 展开，概述了 SCM, SGLANG_CACHE_DIT_ENABLED, SGLANG_CACHE_DIT_SCM_POLICY, medium 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Environment Variables
**EN:** All Cache-DiT parameters can be configured via environment variables. See Environment Variables for the complete list.
**CN:** 本节围绕 Environment Variables 展开，概述了 See, via, list, complete 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Models
**EN:** This section provides a comparison table for Supported Models, covering columns such as Model Family, Example Models and examples such as Wan, Flux, Z-Image, Qwen.
**CN:** 本节围绕 Supported 模型s 展开，概述了 Wan, Flux, Qwen, Wan2.1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Performance Tips
**EN:** **Start with defaults**: The default parameters work well for most models 2. **Use TaylorSeer**: It typically improves both speed and quality 3.
**CN:** 本节围绕 性能 Tips 展开，概述了 SCM, Tune, Start, Lower 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Limitations
**EN:** **SGLang-native pipelines**: Distributed support (TP/SP) is not yet validated; Cache-DiT will be automatically disabled when world_size > 1.
**CN:** 本节围绕 Limitations 展开，概述了 SCM, Cache-DiT, Model, steps 等要点，并说明相关配置、流程、示例或限制条件。

### Section: SCM disabled for low step count
**EN:** For models with < 8 inference steps (e.g., DMD distilled models), SCM will be automatically disabled. DBCache acceleration still works.
**CN:** 本节围绕 SCM disabled for low step count 展开，概述了 DMD, SCM, models, DBCache 等要点，并说明相关配置、流程、示例或限制条件。

### Section: References
**EN:** Cache-DiT - SGLang Diffusion
**CN:** 本节围绕 References 展开，概述了 Cache-DiT, index.md, Diffusion, SGLang Diffusion 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** SCM / **CN:** SCM
- **EN:** config / **CN:** config
- **EN:** Then / **CN:** Then
- **EN:** Cache-DiT / **CN:** 缓存-DiT
- **EN:** TaylorSeer / **CN:** TaylorSeer
- **EN:** parallelism / **CN:** 并行策略
- **EN:** ulysses_size / **CN:** ulysses_size
- **EN:** DBCache / **CN:** DB缓存

## Dependencies / 依赖关系
- `cache.yaml`
- `parallel.yaml`
- `parallel_2d.yaml`
- `parallel_3d.yaml`
- `parallel_uaa.yaml`
- `parallel_fp8.yaml`
- `parallel_async.yaml`
- `parallel_extra.yaml`
- `hybrid.yaml`
- `attention.yaml`
- `quantize.yaml`
- `combined.yaml`
