# hicache_design.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/hicache_design.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This document provides a comprehensive overview of SGLang HiCache, covering its system architecture, workflow and key components. / 该文档围绕 Hi缓存 System Design and Optimization 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This document provides a comprehensive overview of SGLang HiCache, covering its system architecture, workflow and key components.
**CN:** 本节围绕 Overview 展开，概述了 HiCache, LLM, key, tune 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Why and What is HiCache?
**EN:** In large language model inference, the prefill phase is often time-consuming: input sequences need to be first converted into Key-Value cache (KV cache) for subsequent decoding.
**CN:** 本节围绕 Why and What is Hi缓存? 展开，概述了 cache, HiCache, GPU, CPUs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overall Architecture
**EN:** In many modern CPU architectures, the small but fast L1 and L2 caches are private to each core, enabling rapid access to the hottest data, while the larger L3 cache is shared across all cores to significantly reduce redundancy within the cache.
**CN:** 本节围绕 Overall Architecture 展开，概述了 cache, CPU, caches, shared 等要点，并说明相关配置、流程、示例或限制条件。

### Section: HiRadixTree: Metadata Organization in HiCache
**EN:** For KV cache data organization, HiCache builds upon the RadixTree structure introduced in RadixAttention and proposes HiRadixTree. In RadixAttention, each node of the RadixTree corresponds to the KV cache of a consecutive span of tokens in GPU memory.
**CN:** 本节围绕 HiRadixTree: Metadata Organization in Hi缓存 展开，概述了 HiRadixTree, cache, GPU, RadixTree 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Overall Workflow
**EN:** The workflow of HiCache mainly involves three key operations: **local match**, **prefetch** and **write-back**. When the system receives a new request, it first searches the local L1 and L2 caches for matching KV caches.
**CN:** 本节围绕 Overall 工作流 展开，概述了 caches, HiCache, GPU, Once 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Local Match
**EN:** Local matching is the first step in HiCache's workflow, where incoming request tokens are matched against the HiRadixTree to locate cached KV data in local memory tiers (L1 GPU memory and L2 host memory).
**CN:** 本节围绕 Local Match 展开，概述了 HiRadixTree, node, memory, matching 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Prefetch from L3
**EN:** Data prefetching is one of HiCache’s core optimization techniques, designed to proactively load KV caches from L3 storage into local L2 memory, thereby reducing access latency during subsequent operations.
**CN:** 本节围绕 Prefetch from L3 展开，概述了 timeout, HiCache, After, prefetch 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Data Write-back
**EN:** The write-back mechanism is responsible for moving frequently accessed KV caches from L1 to L2 and L3, enabling larger and longer-term storage as well as cache sharing across instances.
**CN:** 本节围绕 Data Write-back 展开，概述了 Data, back, written, level 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-Rank Synchronization
**EN:** During multi-GPU parallel computation, such as tensor parallelism (TP), HiCache must ensure consistent states across different ranks. Therefore, critical computation steps require the use of all_reduce for state synchronization.
**CN:** 本节围绕 Multi-Rank Synchronization 展开，概述了 all_reduce, ranks, min, During 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Data Transfer Optimization
**EN:** **Zero-Copy Data Transfers**: Both prefetching and write-back involve substantial data movement. Minimizing the number of data copies can significantly improve system performance.
**CN:** 本节围绕 Data Transfer Optimization 展开，概述了 data, HiCache, GPU, layer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Integration with PD-Disaggregation Deployment Mode
**EN:** SGLang supports a PD (Prefill-Decode) disaggregation deployment mode through the Mooncake TransferEngine (for details, see this doc).
**CN:** 本节围绕 Integration with PD-Disaggregation 部署 Mode 展开，概述了 nodes, decode, mode, HiCache 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Unified Interfaces and Rich L3 Storage Backends
**EN:** HiCache encapsulates all read, write, and query operations on L3 backends within the class HiCacheStorage(ABC), exposing a set of simple and consistent interfaces.
**CN:** 本节围绕 Unified Interfaces and Rich L3 Storage Backends 展开，概述了 Try, NIXL, here, HF3FS 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Related Parameters
**EN:** **--enable-hierarchical-cache**: Enable hierarchical cache functionality. This is required to use HiCache.
**CN:** 本节围绕 Related Parameters 展开，概述了 cache, memory, See, size 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** HiCache / **CN:** Hi缓存
- **EN:** cache / **CN:** 缓存
- **EN:** GPU / **CN:** GPU
- **EN:** data / **CN:** data
- **EN:** memory / **CN:** memory
- **EN:** storage / **CN:** storage
- **EN:** HiRadixTree / **CN:** HiRadixTree
- **EN:** CPU / **CN:** CPU

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
