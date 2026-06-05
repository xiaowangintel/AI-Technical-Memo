# hicache_best_practices.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/hicache_best_practices.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Why HiCache Matters SGLang HiCache extends the traditional RadixAttention with a three-tier hierarchical KV caching system that dramatically improves performance for long-context and multi-turn conversation scenarios. / 该文档围绕 SGLang Hi缓存 Best Practices 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Why HiCache Matters
**EN:** SGLang HiCache extends the traditional RadixAttention with a three-tier hierarchical KV caching system that dramatically improves performance for long-context and multi-turn conversation scenarios.
**CN:** 本节围绕 Why Hi缓存 Matters 展开，概述了 HiCache, GPU, memory, RadixAttention 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Core HiCache Parameters
**EN:** See Runtime Attach/Detach HiCache Storage Backend.
**CN:** 本节围绕 Core Hi缓存 Parameters 展开，概述了 GPU, HiCache, CPU, Host 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Memory Layout Optimization
**EN:** This content focuses on Memory Layout Optimization and highlights I/O, backend, Optimized, direct.
**CN:** 本节围绕 Memory Layout Optimization 展开，概述了 I/O, backend, Optimized, direct 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Heterogeneous TP Support (GQA/MHA models)
**EN:** HiCache storage supports cross-cluster KV reuse when different deployments use different TP sizes (for example, tp=4 and tp=8) and share the same storage backend namespace.
**CN:** 本节围绕 Heterogeneous TP Support (GQA/MHA 模型s) 展开，概述了 HiCache, tp_lcm_size, same, storage 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prefetch Policies
**EN:** ``bash # Best-effort: Terminate prefetch when needed --hicache-storage-prefetch-policy best_effort # Wait-complete: Ensure complete prefetch, higher cache reuse --hicache-storage-prefetch-policy wait_complete # Timeout: Balance between completion and best-effort --hicache-storage-prefetch-policy timeout ``
**CN:** 本节围绕 Prefetch Policies 展开，概述了 hicache-storage-prefetch-policy, --hicache-storage-prefetch-policy, Ensure, Timeout 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Integration with PD Disaggregation
**EN:** HiCache works seamlessly with PD Disaggregation. You can choose between two configurations: 1.
**CN:** 本节围绕 Integration with PD Disaggregation 展开，概述了 Prefill, HiCache, Decode, nodes 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Deployment with HF3FS
**EN:** Here is an example of deploying DeepSeek-R1 with HiCache-HF3FS. For more details, see the HF3FS Documentation.
**CN:** 本节围绕 部署 with HF3FS 展开，概述了 DeepSeek-R1, Here, see, --tp 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Deployment with Mooncake
**EN:** Here is an example of deploying Qwen3-235B-A22B-Instruct-2507 with Mooncake. For more details, see the Mooncake Documentation.
**CN:** 本节围绕 部署 with Mooncake 展开，概述了 export, Mooncake, Here, MODEL_PATH 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Custom Storage Backend Integration
**EN:** To integrate a new storage backend: 1. **Implement three core methods:** - get(key): Retrieve value by key - exists(key): Check key existence - set(key, value): Store key-value pair 2.
**CN:** 本节围绕 Custom Storage Backend Integration 展开，概述了 key, HiCache, backend, Add 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Dynamic Backend Loading
**EN:** This content focuses on Dynamic Backend Loading and highlights dynamic, Custom, backend, class_name.
**CN:** 本节围绕 Dynamic Backend Loading 展开，概述了 dynamic, Custom, backend, class_name 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Community and Support
**EN:** **GitHub Issues**: Report bugs and feature requests - **Slack Channel**: Join community discussions in #sgl-kv-cache-store - **Documentation**: Refer to storage backend-specific guides --- *This document will be continuously updated based on community feedback and new features.
**CN:** 本节围绕 Community and Support 展开，概述了 Join, Refer, Report, community 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** HiCache / **CN:** Hi缓存
- **EN:** backend / **CN:** backend
- **EN:** Prefill / **CN:** prefill 阶段
- **EN:** storage / **CN:** storage
- **EN:** GPU / **CN:** GPU
- **EN:** Mooncake / **CN:** Mooncake
- **EN:** hicache-storage-backend / **CN:** hi缓存-storage-backend
- **EN:** --hicache-storage-backend / **CN:** --hi缓存-storage-backend

## Dependencies / 依赖关系
- `hicache_storage_runtime_attach_detach.md`
- `../../python/sglang/srt/mem_cache/storage/hf3fs/docs/README.md`
- `../../python/sglang/srt/mem_cache/storage/mooncake_store/README.md`
- `../../python/sglang/srt/mem_cache/storage/backend_factory.py`
