# attention_backends.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/performance/attention_backends.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This document describes the attention backends available in sglang diffusion (sglang.multimodal_gen) and how to select them. / 该文档围绕 注意力后端s 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This document describes the attention backends available in sglang diffusion (sglang.multimodal_gen) and how to select them.
**CN:** 本节围绕 Overview 展开，概述了 sglang.multimodal_gen, select, document, backends 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overview
**EN:** Attention backends are defined by AttentionBackendEnum (sglang.multimodal_gen.runtime.platforms.interface.AttentionBackendEnum) and selected via the CLI flag --attention-backend. Backend selection is performed by the shared attention layers (e.g.
**CN:** 本节围绕 Overview 展开，概述了 SDPA, uses, PyTorch, PyTorch SDPA 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Backend options
**EN:** This section provides a comparison table for Backend options, covering columns such as CLI value, Enum value, Notes and examples such as fa / fa3 / fa4, torch_sdpa, sliding_tile_attn, sage_attn.
**CN:** 本节围绕 Backend options 展开，概述了 Requires, aiter, Configure, --attention-backend-config 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Selection priority
**EN:** The selection order in runtime/layers/attention/selector.py is: 1. global_force_attn_backend(...) / global_force_attn_backend_context_manager(...) 2.
**CN:** 本节围绕 Selection priority 展开，概述了 ServerArgs.attention_backend, CLI, Auto, Component 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Configuration
**EN:** Some backends require additional configuration. You can pass these parameters via --attention-backend-config.
**CN:** 本节围绕 配置 展开，概述了 e.g, sparsity, Key-value, configuration 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Supported Configuration Parameters
**EN:** This section provides a comparison table for Supported Configuration Parameters, covering columns such as Parameter, Type, Description and examples such as mask_strategy_file_path, sta_mode, skip_time_steps, Parameter.
**CN:** 本节围绕 Supported 配置 Parameters 展开，概述了 int, Number, str, Type 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Platform support matrix
**EN:** This section provides a comparison table for Platform support matrix, covering columns such as Backend, CUDA, ROCm and examples such as fa, torch_sdpa, sliding_tile_attn, sage_attn.
**CN:** 本节围绕 Platform 支持矩阵 展开，概述了 Requires, CUDA-only, aiter, Configure 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Select a backend via CLI
**EN:** ``bash sglang generate \ --model-path <MODEL_PATH_OR_ID> \ --prompt "..." \ --attention-backend fa ` `bash sglang generate \ --model-path <MODEL_PATH_OR_ID> \ --prompt "..." \ --attention-backend torch_sdpa ``
**CN:** 本节围绕 Select a backend via CLI 展开，概述了 MODEL_PATH_OR_ID, prompt, --prompt, generate 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Override one component
**EN:** This content focuses on Override one component and highlights module, Component, transformer, text_encoder.
**CN:** 本节围绕 Override one component 展开，概述了 module, Component, transformer, text_encoder 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Using Sliding Tile Attention (STA)
**EN:** ``bash # Pass the mask strategy file path via config sglang generate \ --model-path <MODEL_PATH_OR_ID> \ --prompt "..." \ --attention-backend sliding_tile_attn \ --attention-backend-config "mask_strategy_file_path=/abs/path/to/mask_strategy.json" ``
**CN:** 本节围绕 Using Sliding Tile Attention (STA) 展开，概述了 Pass, MODEL_PATH_OR_ID, via, mask 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Notes for ROCm / MPS
**EN:** ROCm: use --attention-backend torch_sdpa or fa depending on what is available in your environment. MPS: the platform implementation always uses torch_sdpa.
**CN:** 本节围绕 Notes for ROCm / MPS 展开，概述了 torch_sdpa, MPS, ROCm, uses 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Requires / **CN:** Requires
- **EN:** int / **CN:** int
- **EN:** --attention-backend-config / **CN:** --attention-backend-config
- **EN:** aiter / **CN:** aiter
- **EN:** Number / **CN:** Number
- **EN:** Configure / **CN:** Configure
- **EN:** torch_sdpa / **CN:** torch_sdpa
- **EN:** SpargeAttn / **CN:** SpargeAttn

## Dependencies / 依赖关系
- `setup.py`
- `runtime/layers/attention/selector.py`
