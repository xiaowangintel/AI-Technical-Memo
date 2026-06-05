# vLLM Bilingual Technical Tutorial / vLLM 双语技术教程

**EN:** This tutorial is written for engineers who want to understand how vLLM works from the system level down to key implementation modules.
**CN:** 本教程面向希望从系统层到关键实现模块理解 vLLM 的工程师。

**EN:** It is based on the repository `vllm-project/vllm` at `/root/xw/vllm` and cites important docs and source paths from that tree.
**CN:** 本教程基于 `/root/xw/vllm` 中的 `vllm-project/vllm` 仓库编写，并引用其中的重要文档与源码路径。

**EN:** The goal is not to mirror every source file, but to explain the mental model you need to read, extend, debug, and tune vLLM effectively.
**CN:** 目标不是逐字复述每个源码文件，而是帮助你建立足够的心智模型，以便高效阅读、扩展、调试和调优 vLLM。

**EN:** The repository evolves quickly; treat this document as a guided map of the current codebase rather than a permanent specification.
**CN:** 该仓库演进很快；请把本文视为当前代码库的导览图，而不是永久不变的规格说明。

**EN:** Whenever the design docs and code disagree, prefer the code, then use the docs for intent and architectural rationale.
**CN:** 当设计文档与代码存在差异时，应优先以代码为准，再结合文档理解设计意图与架构动机。

**EN:** A few design notes in the repo explicitly label themselves as historical or work-in-progress; those sections are called out where relevant below.
**CN:** 仓库中的部分设计文档明确说明自己是“历史文档”或“进行中设计”；下文会在相关位置特别指出。

**EN:** You will see `**EN:**` and `**CN:**` throughout so the tutorial can be read line by line in both English and Chinese.
**CN:** 全文统一采用 `**EN:**` 与 `**CN:**` 的双语格式，方便逐行对照阅读。

**EN:** If you want to study the code after reading, start with the V1 engine, the scheduler, the KV cache manager, and the model registry.
**CN:** 如果你希望在阅读后继续深入源码，建议优先从 V1 引擎、调度器、KV 缓存管理器以及模型注册表开始。

**EN:** If you want to operate vLLM in production, focus on architecture, batching, KV cache, quantization, parallelism, and API server sections first.
**CN:** 如果你的目标是生产部署 vLLM，请优先阅读架构、批处理、KV 缓存、量化、并行化以及 API 服务器章节。

**EN:** If you want to contribute to performance work, pay special attention to PagedAttention, CUDA kernels, fused MoE, `torch.compile`, and distributed KV transfer.
**CN:** 如果你想参与性能优化工作，请重点关注 PagedAttention、CUDA 内核、融合 MoE、`torch.compile` 与分布式 KV 传输章节。

## Table of Contents / 目录

- **EN:** Introduction
- **CN:** 简介

- **EN:** Architecture Overview
- **CN:** 架构概览

- **EN:** PagedAttention
- **CN:** 分页注意力

- **EN:** KV Cache Management
- **CN:** KV 缓存管理

- **EN:** Continuous Batching
- **CN:** 连续批处理

- **EN:** V1 Engine Architecture
- **CN:** V1 引擎架构

- **EN:** Model Support
- **CN:** 模型支持

- **EN:** Tensor Parallelism
- **CN:** 张量并行

- **EN:** Pipeline Parallelism
- **CN:** 流水线并行

- **EN:** Speculative Decoding
- **CN:** 投机解码

- **EN:** Quantization
- **CN:** 量化

- **EN:** Fused MoE
- **CN:** 融合混合专家

- **EN:** LoRA Serving
- **CN:** LoRA 服务

- **EN:** Multimodal Models
- **CN:** 多模态模型

- **EN:** API Server
- **CN:** API 服务器

- **EN:** Structured Output
- **CN:** 结构化输出

- **EN:** CUDA Kernels
- **CN:** CUDA 内核

- **EN:** torch.compile Integration
- **CN:** torch.compile 集成

- **EN:** Distributed KV Transfer
- **CN:** 分布式 KV 传输

- **EN:** Performance Tuning
- **CN:** 性能调优

- **EN:** Tool Calling
- **CN:** 工具调用

- **EN:** Speech-to-Text
- **CN:** 语音转文字

- **EN:** Build System
- **CN:** 构建系统

- **EN:** Appendices
- **CN:** 附录

## 1. Introduction / 1. 简介

**EN:** vLLM is a high-throughput, memory-efficient library for LLM inference and serving.
**CN:** vLLM 是一个面向大语言模型推理与服务的高吞吐、内存高效框架。

**EN:** The docs front page describes it as “easy, fast, and cheap LLM serving for everyone,” which is a concise summary of the project mission.
**CN:** 文档首页将其概括为“面向所有人的简单、快速、低成本 LLM 服务”，这很好地总结了项目使命。

**EN:** At a high level, vLLM combines efficient memory management, continuous batching, optimized kernels, broad model support, and production APIs.
**CN:** 从整体上看，vLLM 将高效内存管理、连续批处理、优化内核、广泛模型支持与生产级 API 结合在一起。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The headline innovations called out in `docs/README.md` include PagedAttention, continuous batching, chunked prefill, and prefix caching.
- **CN:** `docs/README.md` 中强调的核心创新包括 PagedAttention、连续批处理、分块预填充和前缀缓存。

- **EN:** The same page also highlights quantization, speculative decoding, CUDA/HIP graphs, and distributed serving as first-class capabilities.
- **CN:** 同一页面还强调量化、投机解码、CUDA/HIP 图以及分布式服务都是一等能力。

- **EN:** vLLM aims to support both offline inference through Python APIs and online serving through HTTP and other network entrypoints.
- **CN:** vLLM 既支持通过 Python API 的离线推理，也支持通过 HTTP 等网络入口的在线服务。

- **EN:** The project integrates tightly with Hugging Face model ecosystems while keeping its own execution engine and kernel stack.
- **CN:** 该项目与 Hugging Face 模型生态紧密集成，同时保持自己的执行引擎与内核栈。

- **EN:** The practical value proposition is simple: use GPU memory more efficiently so the same hardware can serve more tokens at lower latency.
- **CN:** 它的实际价值主张很直接：更高效地利用 GPU 显存，让同样的硬件以更低延迟服务更多 token。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/README.md` is the best short summary of project scope, features, hardware, and supported model families.
- **CN:** `docs/README.md` 是理解项目范围、特性、硬件支持和模型家族的最佳简明入口。

- **EN:** `docs/design/arch_overview.md` connects the product view to the internal engine view.
- **CN:** `docs/design/arch_overview.md` 将产品层视角与内部引擎视角连接起来。

- **EN:** `vllm/entrypoints/llm.py` is the classic offline Python entrypoint for local generation workflows.
- **CN:** `vllm/entrypoints/llm.py` 是本地生成工作流的经典离线 Python 入口。

- **EN:** `vllm/entrypoints/openai/api_server.py` is the main online serving entrypoint used by `vllm serve`.
- **CN:** `vllm/entrypoints/openai/api_server.py` 是 `vllm serve` 所使用的主要在线服务入口。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** If you are new to vLLM, first decide whether you care more about serving architecture, model integration, or kernel-level performance.
- **CN:** 如果你刚接触 vLLM，首先要确定自己更关心服务架构、模型集成，还是内核级性能。

- **EN:** If you are migrating from `transformers.generate`, the most important conceptual shift is that vLLM is built around a serving engine, not a single-request loop.
- **CN:** 如果你是从 `transformers.generate` 迁移而来，最重要的概念变化是：vLLM 围绕服务引擎构建，而不是单请求循环。

- **EN:** If you are migrating from another serving stack, compare batching policy, KV cache layout, and quantization/kernel support first.
- **CN:** 如果你是从其他服务框架迁移而来，建议优先比较批处理策略、KV 缓存布局，以及量化/内核支持能力。

- **EN:** A good mental model is that vLLM is a specialized operating system for tokens, memory pages, and GPU execution slots.
- **CN:** 一个很好的心智模型是：vLLM 就像一个专门管理 token、内存页和 GPU 执行槽位的“微型操作系统”。

**EN:** In short, vLLM matters because it turns LLM inference from a naive per-request computation problem into a systems optimization problem.
**CN:** 简言之，vLLM 的意义在于：它把 LLM 推理从朴素的单请求计算问题，转化为了系统级优化问题。

## 2. Architecture Overview / 2. 架构概览

**EN:** The architecture spans multiple entrypoints, a multi-process V1 runtime, workers, model runners, and model modules.
**CN:** vLLM 的架构横跨多个入口、多进程 V1 运行时、工作进程、模型运行器以及模型模块。

**EN:** Understanding the process boundaries is essential because many performance and deployment decisions happen between processes, not just inside a PyTorch model.
**CN:** 理解进程边界非常关键，因为很多性能与部署决策并不是只发生在 PyTorch 模型内部，而是发生在进程之间。

**EN:** The architecture overview doc describes API server processes, engine core processes, GPU worker processes, and an optional DP coordinator process.
**CN:** 架构概览文档描述了 API 服务进程、引擎核心进程、GPU 工作进程，以及可选的数据并行协调进程。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The API server handles HTTP requests, tokenization, multimodal preprocessing, and streaming outputs back to clients.
- **CN:** API 服务器负责 HTTP 请求、分词、多模态预处理，以及将流式输出返回给客户端。

- **EN:** The engine core owns scheduling, KV cache management, and dispatch of model execution to workers.
- **CN:** 引擎核心负责调度、KV 缓存管理，以及把模型执行任务派发给工作进程。

- **EN:** Each GPU worker is responsible for one accelerator device and holds model weights plus runtime buffers for that rank.
- **CN:** 每个 GPU 工作进程负责一个加速器设备，并持有该 rank 的模型权重与运行时缓冲区。

- **EN:** When data parallelism is enabled, a DP coordinator handles load balancing and some multi-rank coordination.
- **CN:** 当启用数据并行时，DP 协调进程负责负载均衡以及部分多 rank 协调工作。

- **EN:** The class hierarchy further separates LLM clients, engine layers, workers, model runners, and actual `torch.nn.Module` model classes.
- **CN:** 类层级还进一步区分了 LLM 客户端、引擎层、工作进程、模型运行器，以及真正的 `torch.nn.Module` 模型类。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/design/arch_overview.md` is the primary architecture map and includes process-count examples for TP and DP deployments.
- **CN:** `docs/design/arch_overview.md` 是主要的架构地图，并包含 TP/DP 部署下的进程数量示例。

- **EN:** `vllm/v1/engine/core.py` shows how the engine core builds the scheduler, executor, and cache-related services.
- **CN:** `vllm/v1/engine/core.py` 展示了引擎核心如何构建调度器、执行器与缓存相关服务。

- **EN:** `vllm/v1/executor/multiproc_executor.py` and `vllm/v1/worker/gpu_worker.py` represent the worker-side runtime boundary.
- **CN:** `vllm/v1/executor/multiproc_executor.py` 与 `vllm/v1/worker/gpu_worker.py` 体现了工作进程侧的运行时边界。

- **EN:** `vllm/v1/engine/coordinator.py` defines the extra coordinator role used in data-parallel configurations.
- **CN:** `vllm/v1/engine/coordinator.py` 定义了数据并行配置下使用的额外协调者角色。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** A useful diagram description is: client → API server → engine core → GPU workers → model runner → model layers.
- **CN:** 一个实用的图示描述是：客户端 → API 服务器 → 引擎核心 → GPU 工作进程 → 模型运行器 → 模型层。

- **EN:** Another way to see it is that the API server owns request semantics, the engine owns token scheduling, and workers own tensor execution.
- **CN:** 另一种理解方式是：API 服务器负责请求语义，引擎负责 token 调度，工作进程负责张量执行。

- **EN:** The architecture intentionally isolates CPU-heavy preprocessing from GPU-heavy execution so both can scale and fail independently.
- **CN:** 该架构有意把 CPU 密集型预处理与 GPU 密集型执行隔离开来，使二者能够独立扩展并独立失效。

- **EN:** When debugging deployment issues, first identify whether the bug belongs to the frontend, engine core, distributed runtime, or model backend.
- **CN:** 调试部署问题时，第一步应判断问题属于前端、引擎核心、分布式运行时，还是模型后端。

**EN:** The architecture is modular because vLLM wants to support many models and hardware targets without rewriting the entire serving stack for each case.
**CN:** 这种架构之所以模块化，是因为 vLLM 需要支持大量模型和硬件目标，而不想为每种情况重写整套服务栈。

## 3. PagedAttention / 3. 分页注意力

**EN:** PagedAttention is the idea most strongly associated with vLLM: store KV cache in fixed-size blocks and access it through block tables instead of monolithic contiguous tensors.
**CN:** PagedAttention 是与 vLLM 绑定最紧密的概念：将 KV 缓存存成固定大小的块，并通过块表访问，而不是使用单体连续张量。

**EN:** This design reduces fragmentation and makes long-running, highly concurrent serving more practical.
**CN:** 这种设计减少了内存碎片，使长时间运行且高并发的服务场景更可行。

**EN:** The historical design doc explains the attention kernel around query reads, key reads, QK dot products, softmax, and value accumulation over paged cache blocks.
**CN:** 历史设计文档围绕分页缓存块上的 query 读取、key 读取、QK 点积、softmax 与 value 聚合来解释该注意力内核。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** In the historical doc, a “block” means a fixed number of tokens in KV cache, not a CUDA thread block.
- **CN:** 在历史文档中，“block”指的是 KV 缓存中的固定 token 块，而不是 CUDA 线程块。

- **EN:** The query tensor is read into shared memory in a layout that promotes memory coalescing across neighboring threads.
- **CN:** query 张量会以有利于相邻线程协同访问的布局读入共享内存，从而提升内存合并访问效率。

- **EN:** Key data is read from paged cache blocks into registers, again with layout choices that favor coalesced access.
- **CN:** key 数据从分页缓存块读入寄存器，其布局同样服务于高效的合并访问。

- **EN:** Each warp processes context blocks for one query/head combination, while the thread block covers the whole context for that sequence/head/partition.
- **CN:** 每个 warp 处理某个 query/head 组合上的上下文块，而线程块则覆盖该 sequence/head/partition 的整个上下文。

- **EN:** The modern code uses custom kernels in `csrc/attention/*` and wrapper ops in `vllm/_custom_ops.py` and V1 attention backends.
- **CN:** 现代代码在 `csrc/attention/*` 中实现自定义内核，并通过 `vllm/_custom_ops.py` 以及 V1 注意力后端进行封装。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/design/paged_attention.md` is explicitly marked as historical, but it remains the best conceptual explanation of the original kernel logic.
- **CN:** `docs/design/paged_attention.md` 明确标注为历史文档，但仍然是理解原始内核逻辑的最佳概念说明。

- **EN:** `csrc/attention/attention_kernels.cuh` contains the core templated kernel logic used by the current CUDA implementation.
- **CN:** `csrc/attention/attention_kernels.cuh` 包含当前 CUDA 实现所使用的核心模板化内核逻辑。

- **EN:** `csrc/attention/paged_attention_v1.cu` and `paged_attention_v2.cu` contain the launchers and specializations for the paged kernels.
- **CN:** `csrc/attention/paged_attention_v1.cu` 与 `paged_attention_v2.cu` 包含分页内核的 launcher 与特化实现。

- **EN:** `vllm/v1/attention/ops/paged_attn.py` and `chunked_prefill_paged_decode.py` show how V1 uses paged cache operations at the Python/Triton boundary.
- **CN:** `vllm/v1/attention/ops/paged_attn.py` 与 `chunked_prefill_paged_decode.py` 展示了 V1 如何在 Python/Triton 边界使用分页缓存操作。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** When reading the source, keep two levels in your head: the memory-management idea of paging, and the low-level CUDA implementation strategy.
- **CN:** 阅读源码时要同时保持两层理解：一层是分页这种内存管理思想，另一层是具体的 CUDA 实现策略。

- **EN:** PagedAttention is not only about a faster kernel; it is also about making the allocator and scheduler workable under bursty serving loads.
- **CN:** PagedAttention 不只是更快的内核，它还让分配器与调度器在突发式服务负载下变得可行。

- **EN:** The important system effect is indirection: logical sequence positions are mapped to physical cache blocks only when needed.
- **CN:** 它最重要的系统效应是“间接寻址”：逻辑序列位置只在需要时映射到物理缓存块。

- **EN:** Because the design separates logical order from physical placement, cache reuse and eviction policies become much more flexible.
- **CN:** 由于该设计把逻辑顺序与物理放置分离，缓存复用与淘汰策略也因此更加灵活。

**EN:** If you remember only one sentence, remember this: PagedAttention makes KV cache behave more like virtual memory than a giant flat tensor.
**CN:** 如果只记住一句话，请记住：PagedAttention 让 KV 缓存更像虚拟内存，而不是一个巨大的平面张量。

## 4. KV Cache Management / 4. KV 缓存管理

**EN:** The V1 KV cache manager owns allocation, reuse, eviction, prefix-cache lookup, and hybrid-model cache grouping.
**CN:** V1 的 KV 缓存管理器负责分配、复用、淘汰、前缀缓存查找以及混合模型缓存分组。

**EN:** In serving, good scheduling is useless without a memory manager that can keep up; the KV cache manager is that memory manager.
**CN:** 在服务场景中，如果内存管理跟不上，再好的调度也没有意义；KV 缓存管理器正是这个内存管理核心。

**EN:** The basic workflow is: compute block hashes, find cached full blocks, touch them, allocate new blocks, append tokens, and evict LRU blocks when necessary.
**CN:** 其基本流程是：计算块哈希、查找已缓存的完整块、标记触达、分配新块、追加 token，并在必要时按 LRU 淘汰块。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** Prefix caching in V1 is hash-based and uses parent hash plus block tokens plus extra data such as LoRA IDs, multimodal hashes, or cache salt.
- **CN:** V1 的前缀缓存基于哈希，哈希项由父哈希、块内 token，以及 LoRA ID、多模态哈希或 cache salt 等附加信息组成。

- **EN:** Only full blocks are cached, which simplifies reuse semantics and avoids partial-block ambiguity.
- **CN:** 只有完整块会被缓存，这样可以简化复用语义，并避免部分块带来的歧义。

- **EN:** The block pool pre-allocates `KVCacheBlock` objects and maintains a doubly linked free queue for O(1) movement.
- **CN:** 块池会预先分配 `KVCacheBlock` 对象，并维护一个双向链表空闲队列，以实现 O(1) 的移动操作。

- **EN:** Hybrid models can require multiple KV cache groups because full attention, sliding-window attention, and state-space layers do not reserve memory the same way.
- **CN:** 混合模型可能需要多个 KV 缓存组，因为全注意力、滑窗注意力和状态空间层的内存保留方式并不相同。

- **EN:** V1 also includes offload and transfer-related paths, including scheduler-side KV connectors, simple KV offload, and connector-based CPU/network movement.
- **CN:** V1 还包含卸载与传输相关路径，包括调度器侧 KV 连接器、简单 KV 卸载，以及基于连接器的 CPU/网络传输机制。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/design/prefix_caching.md` explains hashing, block allocation, free queues, and LRU eviction in detail.
- **CN:** `docs/design/prefix_caching.md` 详细解释了哈希、块分配、空闲队列与 LRU 淘汰。

- **EN:** `docs/design/hybrid_kv_cache_manager.md` explains KV cache groups, shared page sizes, and hybrid prefix-caching rules.
- **CN:** `docs/design/hybrid_kv_cache_manager.md` 解释了 KV 缓存组、统一页大小，以及混合前缀缓存规则。

- **EN:** `vllm/v1/core/kv_cache_manager.py` is the main interface between the scheduler and cache management code.
- **CN:** `vllm/v1/core/kv_cache_manager.py` 是调度器与缓存管理代码之间的主要接口。

- **EN:** `vllm/v1/core/block_pool.py`, `kv_cache_coordinator.py`, and `single_type_kv_cache_manager.py` implement the internal layers of the cache stack.
- **CN:** `vllm/v1/core/block_pool.py`、`kv_cache_coordinator.py` 与 `single_type_kv_cache_manager.py` 实现了缓存栈的内部层次。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** A block is both the accounting unit and the reuse unit, which is why block size is such an important systems parameter.
- **CN:** 块既是资源记账单位，也是复用单位，因此 block size 是一个极其重要的系统参数。

- **EN:** Prefix caching is powerful because it removes redundant prefill work, but it only works when requests share sufficiently stable prefixes.
- **CN:** 前缀缓存之所以强大，是因为它能消除重复的预填充计算；但前提是请求之间必须共享足够稳定的前缀。

- **EN:** Cache salting exists because shared caches in multi-tenant environments can leak information through timing behavior.
- **CN:** 之所以有 cache salt，是因为在多租户环境中，共享缓存可能通过时延侧信道泄露信息。

- **EN:** If you debug cache misses, check block size, prompt formatting, multimodal placeholders, LoRA IDs, and any per-request salt first.
- **CN:** 如果你在排查缓存未命中，优先检查 block size、prompt 格式、多模态占位符、LoRA ID 以及请求级 salt。

**EN:** Think of the KV cache manager as the memory allocator, page table, cache index, and eviction controller of the whole inference system.
**CN:** 可以把 KV 缓存管理器看作整个推理系统的内存分配器、页表、缓存索引以及淘汰控制器。

## 5. Continuous Batching / 5. 连续批处理

**EN:** Continuous batching means the engine keeps mixing, updating, and advancing requests over time instead of waiting for rigid batch boundaries.
**CN:** 连续批处理意味着引擎会持续地混合、更新和推进请求，而不是等待僵硬的批边界。

**EN:** This is one of the main reasons vLLM achieves high throughput without destroying latency.
**CN:** 这是 vLLM 在不严重牺牲延迟的情况下实现高吞吐的主要原因之一。

**EN:** In V1, the scheduler treats prompt tokens and decode tokens under a unified token-budget model rather than a strict prefill/decode split.
**CN:** 在 V1 中，调度器使用统一的 token 预算模型来处理 prompt token 与 decode token，而不是严格区分预填充与解码阶段。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The V1 guide explicitly notes that prompt and output tokens are scheduled with the same dictionary-style budget logic.
- **CN:** V1 指南明确指出，prompt token 与输出 token 通过同一种字典式预算逻辑进行调度。

- **EN:** This unified view makes chunked prefill, prefix caching, and speculative decoding compose more naturally.
- **CN:** 这种统一视角让分块预填充、前缀缓存与投机解码更容易自然组合。

- **EN:** The scheduler maintains waiting and running request structures and enforces limits such as maximum sequences and scheduled tokens.
- **CN:** 调度器维护 waiting 与 running 请求结构，并执行最大序列数、最大调度 token 数等限制。

- **EN:** Policy can be FCFS or priority-based, with FCFS used as the tie-breaker in priority mode.
- **CN:** 调度策略可以是 FCFS 或基于优先级的模式，而优先级模式中 FCFS 仍可作为平局决胜规则。

- **EN:** Continuous batching is effective because many requests in adjacent steps differ only a little, so work and state can be incrementally updated.
- **CN:** 连续批处理之所以高效，是因为相邻步骤中的很多请求只发生少量变化，因此状态和工作都可以增量更新。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/usage/v1_guide.md` summarizes the unified scheduler idea and supported policies.
- **CN:** `docs/usage/v1_guide.md` 总结了统一调度器思想与支持的调度策略。

- **EN:** `vllm/v1/core/sched/scheduler.py` is the main scheduling implementation for request admission and per-step decisions.
- **CN:** `vllm/v1/core/sched/scheduler.py` 是请求接纳与逐步调度决策的主要实现。

- **EN:** `vllm/v1/core/sched/request_queue.py` contains policy-aware queue logic.
- **CN:** `vllm/v1/core/sched/request_queue.py` 包含与策略相关的队列逻辑。

- **EN:** `vllm/v1/engine/core.py` shows how the scheduler sits inside the engine core’s main loop.
- **CN:** `vllm/v1/engine/core.py` 展示了调度器如何嵌入引擎核心的主循环。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** Continuous batching is not just “bigger batches”; it is about constantly re-optimizing the active set of requests.
- **CN:** 连续批处理并不只是“更大的 batch”，而是不断重新优化当前活跃请求集合。

- **EN:** Chunked prefill matters because very long prompts would otherwise monopolize the scheduler and hurt inter-token latency.
- **CN:** 分块预填充之所以重要，是因为超长 prompt 否则会独占调度器，并损害逐 token 延迟。

- **EN:** Speculative decoding complicates scheduling because the engine must reserve lookahead capacity and verification work.
- **CN:** 投机解码会使调度更复杂，因为引擎需要为前瞻 token 和验证步骤预留容量。

- **EN:** When tuning latency, inspect scheduler limits, streaming interval, prefill chunking, and any connector-induced stalls together.
- **CN:** 调优延迟时，应同时检查调度器限制、流式间隔、预填充分块以及连接器带来的阻塞。

**EN:** Continuous batching is the policy layer that turns many independent requests into one efficient stream of GPU work.
**CN:** 连续批处理就是那个把大量独立请求转化为一条高效 GPU 工作流的策略层。

## 6. V1 Engine Architecture / 6. V1 引擎架构

**EN:** V1 is the modern core engine architecture of vLLM, designed to simplify the old codebase while improving performance and composability.
**CN:** V1 是 vLLM 的现代核心引擎架构，目标是在简化旧代码库的同时提升性能与可组合性。

**EN:** Many current features, including prefix caching, speculative decoding, and async serving, are easiest to understand through the V1 lens.
**CN:** 包括前缀缓存、投机解码和异步服务在内的许多当前特性，都最适合通过 V1 的视角来理解。

**EN:** The `AsyncLLM` frontend builds input/output processors and talks to an `EngineCoreClient`, which manages one or more engine cores running in background processes.
**CN:** `AsyncLLM` 前端会构建输入/输出处理器，并通过 `EngineCoreClient` 与一个或多个后台引擎核心进程通信。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** `AsyncLLM` is the async wrapper used by online serving and is responsible for frontend-facing request lifecycle management.
- **CN:** `AsyncLLM` 是在线服务使用的异步封装层，负责面向前端的请求生命周期管理。

- **EN:** `EngineCoreClient` abstracts transport and execution mode, with in-process, sync multiprocessing, and async multiprocessing variants.
- **CN:** `EngineCoreClient` 抽象了传输与执行模式，包含进程内、同步多进程和异步多进程等变体。

- **EN:** The engine core itself builds the scheduler, KV cache manager, executor, and associated services, then runs a busy inference loop.
- **CN:** 引擎核心本身会构建调度器、KV 缓存管理器、执行器及相关服务，然后运行一个忙循环推理主循环。

- **EN:** In DP mode, the coordinator helps with load balancing and multi-rank coordination rather than model execution itself.
- **CN:** 在 DP 模式下，协调器负责负载均衡与多 rank 协调，而不直接执行模型。

- **EN:** V1 defaults many optimizations on by design because the project wants a zero-config experience for common deployments.
- **CN:** V1 设计上默认启用很多优化，因为项目希望在常见部署中尽量做到零配置体验。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/usage/v1_guide.md` explains why V1 exists and what it changed relative to V0.
- **CN:** `docs/usage/v1_guide.md` 解释了 V1 的动机，以及它相对 V0 有哪些变化。

- **EN:** `vllm/v1/engine/async_llm.py` is the best place to understand the async frontend wrapper.
- **CN:** `vllm/v1/engine/async_llm.py` 是理解异步前端封装层的最佳入口。

- **EN:** `vllm/v1/engine/core_client.py` shows client selection, DP-aware client variants, and transport abstraction.
- **CN:** `vllm/v1/engine/core_client.py` 展示了客户端选择、DP 感知客户端变体以及传输抽象。

- **EN:** `vllm/v1/engine/coordinator.py` and `core.py` explain how engine processes and coordinator roles are instantiated.
- **CN:** `vllm/v1/engine/coordinator.py` 与 `core.py` 解释了引擎进程与协调器角色如何被实例化。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** If you only read one runtime path end to end, read request submission in `AsyncLLM`, dispatch in `EngineCoreClient`, and scheduling in `EngineCore`.
- **CN:** 如果你只打算完整读一条运行路径，建议从 `AsyncLLM` 的请求提交、`EngineCoreClient` 的分发，再到 `EngineCore` 的调度读起。

- **EN:** The V1 mental model is cleaner because nearly every feature now plugs into a small set of core abstractions.
- **CN:** V1 的心智模型更清晰，因为几乎所有特性都接入少数几个核心抽象。

- **EN:** When bugs appear “random,” first determine whether they come from frontend async logic, inter-process transport, or worker-side execution.
- **CN:** 当 bug 看起来“随机”时，先判断它来自前端异步逻辑、进程间传输，还是工作进程侧执行。

- **EN:** The V1 guide also documents removed features, which is useful when migrating old code or expectations.
- **CN:** V1 指南还记录了被移除的特性，这对于迁移旧代码或旧预期很有帮助。

**EN:** V1 is best seen as a clean serving substrate: a small set of durable abstractions with many optimizations layered on top.
**CN:** 最好把 V1 看成一个干净的服务基座：少量稳定抽象之上叠加了大量优化。

## 7. Model Support / 7. 模型支持

**EN:** vLLM supports hundreds of Hugging Face architectures by mapping architecture names to in-tree implementation classes.
**CN:** vLLM 通过把架构名映射到仓库内实现类的方式，支持数百种 Hugging Face 架构。

**EN:** This registry-driven approach is why one serving engine can work across decoder-only LLMs, embedding models, multimodal models, MoEs, and ASR models.
**CN:** 正是这种注册表驱动方法，让同一个服务引擎能覆盖 decoder-only LLM、嵌入模型、多模态模型、MoE 模型以及 ASR 模型。

**EN:** The registry collects architecture dictionaries, capability metadata, and helper methods to resolve the right runtime class from model config.
**CN:** 注册表会汇总架构字典、能力元数据以及辅助方法，以便从模型配置中解析出正确的运行时类。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** `_TEXT_GENERATION_MODELS`, `_EMBEDDING_MODELS`, `_MULTIMODAL_MODELS`, and other dictionaries group model families by task and capability.
- **CN:** `_TEXT_GENERATION_MODELS`、`_EMBEDDING_MODELS`、`_MULTIMODAL_MODELS` 等字典按任务和能力对模型家族进行分组。

- **EN:** The registry tracks model properties such as text generation, pooling, multimodal support, and transcription support.
- **CN:** 注册表会跟踪文本生成、池化、多模态支持与转写支持等模型属性。

- **EN:** Capability interfaces such as `SupportsMultiModal`, `SupportsTranscription`, and `supports_pp` let the engine ask feature questions without hardcoding architecture names.
- **CN:** 像 `SupportsMultiModal`、`SupportsTranscription`、`supports_pp` 这样的能力接口，使引擎可以在不硬编码架构名的情况下判断功能支持。

- **EN:** `register_model()` allows lazy or external registration, which is important for plugins and out-of-tree extensions.
- **CN:** `register_model()` 支持惰性注册与外部注册，这对插件与仓库外扩展非常重要。

- **EN:** The registry is also a documentation hub because the architecture names mirror what you see in Hugging Face model configs.
- **CN:** 注册表本身也是一个文档中心，因为其中的架构名与 Hugging Face 模型配置中的名称相对应。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `vllm/model_executor/models/registry.py` is the main registry and capability-resolution file.
- **CN:** `vllm/model_executor/models/registry.py` 是主要的模型注册与能力解析文件。

- **EN:** `vllm/model_executor/models/interfaces.py` defines runtime protocols such as `SupportsMultiModal`.
- **CN:** `vllm/model_executor/models/interfaces.py` 定义了 `SupportsMultiModal` 等运行时协议。

- **EN:** `vllm/model_executor/models/interfaces_base.py` defines more generic model/task classification helpers.
- **CN:** `vllm/model_executor/models/interfaces_base.py` 定义了更通用的模型/任务分类辅助函数。

- **EN:** `docs/README.md` and `docs/models/supported_models.md` summarize supported model families from a user-facing perspective.
- **CN:** `docs/README.md` 与 `docs/models/supported_models.md` 从用户视角总结了支持的模型家族。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** When adding a new architecture, think in two layers: how to load the model, and what interfaces/capabilities it must advertise.
- **CN:** 添加新架构时，要分两层思考：如何加载模型，以及它必须声明哪些接口/能力。

- **EN:** If a model supports only some features, use capability flags instead of ad hoc special cases in the scheduler or server.
- **CN:** 如果模型只支持部分功能，应通过能力标志表达，而不是在调度器或服务器中写零散特判。

- **EN:** A registry-based design scales better than per-model conditionals because serving logic stays generic while model logic stays local.
- **CN:** 基于注册表的设计比逐模型条件分支更易扩展，因为服务逻辑可以保持通用，而模型逻辑保持本地化。

- **EN:** Always update tests when you add an architecture; the registry file itself reminds contributors to do this.
- **CN:** 添加架构时务必同步更新测试；注册表文件本身也明确提醒贡献者这样做。

**EN:** Model support in vLLM is not accidental breadth; it is a deliberate registry-and-interface architecture.
**CN:** vLLM 的广泛模型支持不是偶然堆出来的，而是注册表与接口架构的直接结果。

## 8. Tensor Parallelism / 8. 张量并行

**EN:** Tensor parallelism splits model computation across multiple GPUs by sharding weights and communicating partial results.
**CN:** 张量并行通过切分权重并通信部分结果，把模型计算分布到多张 GPU 上。

**EN:** It is the default answer when a model does not fit on one GPU but does fit within one node or a few tightly connected nodes.
**CN:** 当模型装不进单卡、但能装进单机或少量紧密互联节点时，张量并行通常是默认方案。

**EN:** vLLM uses Megatron-style model parallel groups and provides collective helpers such as all-reduce, reduce-scatter, and all-gather in its distributed state layer.
**CN:** vLLM 使用类似 Megatron 的模型并行分组，并在分布式状态层提供 all-reduce、reduce-scatter、all-gather 等集合通信辅助。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The parallelism guide recommends TP when the model is too large for one GPU but can live within a single multi-GPU node.
- **CN:** 并行化指南建议：当模型单卡放不下、但能放进单机多卡时，应优先考虑 TP。

- **EN:** `parallel_state.py` initializes the distributed environment and model-parallel groups and exposes communication helpers.
- **CN:** `parallel_state.py` 负责初始化分布式环境与模型并行组，并暴露通信辅助函数。

- **EN:** vLLM wraps collective operations because execution code wants a stable abstraction layer above raw `torch.distributed` calls.
- **CN:** vLLM 对集合通信进行了封装，因为执行代码需要一个稳定抽象层，而不是直接依赖原始 `torch.distributed` 调用。

- **EN:** The compile docs show communication files like `communication_op.py` and `parallel_state.py` in the traced execution graph for Llama models.
- **CN:** 编译文档展示了像 `communication_op.py` 与 `parallel_state.py` 这样的通信文件会出现在 Llama 模型的跟踪图中。

- **EN:** Custom all-reduce can be disabled in config, which is useful for debugging or platform-specific compatibility issues.
- **CN:** 配置中可以关闭自定义 all-reduce，这对调试或处理特定平台兼容性问题很有帮助。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/serving/parallelism_scaling.md` is the practical guide for choosing TP and PP values.
- **CN:** `docs/serving/parallelism_scaling.md` 是选择 TP 与 PP 参数的实用指南。

- **EN:** `vllm/config/parallel.py` defines `tensor_parallel_size` and related distributed options.
- **CN:** `vllm/config/parallel.py` 定义了 `tensor_parallel_size` 等相关分布式配置项。

- **EN:** `vllm/distributed/parallel_state.py` manages groups, collectives, and lower-level distributed state helpers.
- **CN:** `vllm/distributed/parallel_state.py` 管理分组、集合通信以及底层分布式状态辅助逻辑。

- **EN:** `vllm/distributed/communication_op.py` participates in the execution graph and houses communication primitives used by model code.
- **CN:** `vllm/distributed/communication_op.py` 会参与执行图，并提供模型代码使用的通信原语。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** TP gives good memory scaling, but every forward pass pays communication cost, so interconnect quality matters a lot.
- **CN:** TP 虽然能很好地扩展显存，但每次前向都要付出通信代价，因此互联质量非常关键。

- **EN:** The docs explicitly call out InfiniBand and GPUDirect RDMA as important for efficient cross-node TP.
- **CN:** 文档明确指出：对于跨节点 TP，高效使用 InfiniBand 与 GPUDirect RDMA 很重要。

- **EN:** If GPUs do not have strong intra-node interconnect, PP can sometimes outperform TP even on one node.
- **CN:** 如果 GPU 间缺少强力的节点内互联，那么即便在单机内，PP 有时也可能优于 TP。

- **EN:** Use TP first for model fit, then profile whether latency and communication overhead remain acceptable.
- **CN:** 应先用 TP 解决模型装载问题，再通过实际测量判断延迟与通信开销是否仍可接受。

**EN:** Tensor parallelism is the memory-fitting and math-splitting tool; the bill comes due in collective communication.
**CN:** 张量并行是解决显存装载与算子拆分的工具，而代价则体现在集合通信上。

## 9. Pipeline Parallelism / 9. 流水线并行

**EN:** Pipeline parallelism splits a model by layers across devices or nodes so that different stages process different microbatches.
**CN:** 流水线并行按层把模型切分到不同设备或节点上，使不同阶段处理不同的微批次。

**EN:** It becomes important when the model no longer fits within a single node, or when communication patterns make TP unattractive.
**CN:** 当模型超出单机容量，或通信模式使 TP 不再合适时，PP 就变得重要。

**EN:** The serving guide recommends a common multi-node pattern: set TP to GPUs per node and PP to number of nodes.
**CN:** 服务指南推荐一种常见的多节点模式：把 TP 设为每节点 GPU 数，把 PP 设为节点数。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** PP divides the model along depth rather than width, so each stage holds a contiguous layer range.
- **CN:** PP 是沿模型深度而不是宽度切分，因此每个 stage 持有一段连续层。

- **EN:** The architecture overview reminds you that worker count per engine core is `tensor_parallel_size × pipeline_parallel_size`.
- **CN:** 架构概览提醒你：每个引擎核心下的工作进程数等于 `tensor_parallel_size × pipeline_parallel_size`。

- **EN:** `ParallelConfig` exposes `pipeline_parallel_size` as a first-class setting alongside TP and DP.
- **CN:** `ParallelConfig` 将 `pipeline_parallel_size` 与 TP、DP 一起作为一等配置项暴露出来。

- **EN:** Dual batch overlap and ubatch thresholds in `ParallelConfig` show that microbatching and overlap are important implementation details.
- **CN:** `ParallelConfig` 中的 dual batch overlap 与 ubatch 阈值说明，微批与重叠执行是重要实现细节。

- **EN:** The speculative decoding docs also note that PP is not universally composable with every advanced feature in every version.
- **CN:** 投机解码文档还提醒我们：PP 与所有高级特性的组合并不总是在所有版本中都完全可用。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/serving/parallelism_scaling.md` explains when to choose TP, PP, or both.
- **CN:** `docs/serving/parallelism_scaling.md` 解释了何时选择 TP、PP，或同时使用二者。

- **EN:** `vllm/config/parallel.py` defines `pipeline_parallel_size` and microbatch-related controls.
- **CN:** `vllm/config/parallel.py` 定义了 `pipeline_parallel_size` 以及与微批相关的控制项。

- **EN:** `docs/design/arch_overview.md` gives the process-count view that makes PP’s worker multiplication obvious.
- **CN:** `docs/design/arch_overview.md` 给出了进程数量视角，使 PP 带来的工作进程乘法关系一目了然。

- **EN:** `vllm/v1/worker/gpu/pp_utils.py` is the kind of worker-side file to inspect when tracing PP-specific logic.
- **CN:** `vllm/v1/worker/gpu/pp_utils.py` 这类工作进程侧文件，是追踪 PP 逻辑时的重点入口。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** PP is often the easiest way to scale beyond one node because it reduces the need for every layer to communicate on every step.
- **CN:** PP 往往是扩展到多节点时最直接的方法，因为它减少了每一步都要跨所有层通信的需要。

- **EN:** The downside is pipeline bubbles and stage imbalance, especially when the model is not evenly divisible.
- **CN:** 它的代价是流水线气泡与阶段不均衡，尤其是在模型无法均匀切分时。

- **EN:** If your node has weak or no NVLink-style interconnect, PP may outperform TP even before you hit a memory limit.
- **CN:** 如果节点内缺少强力 NVLink 式互联，那么在显存成为瓶颈之前，PP 就可能已经优于 TP。

- **EN:** Treat PP as a system-level latency-throughput tradeoff, not merely as a model-fitting trick.
- **CN:** 应把 PP 看作一种系统级的延迟/吞吐权衡，而不仅仅是“把模型塞进去”的技巧。

**EN:** Pipeline parallelism moves the scaling problem from tensor collectives to stage scheduling and microbatch orchestration.
**CN:** 流水线并行把扩展问题从张量集合通信，转移到了阶段调度与微批编排上。

## 10. Speculative Decoding / 10. 投机解码

**EN:** Speculative decoding reduces inter-token latency by having a proposer generate candidate tokens and a target model verify them.
**CN:** 投机解码通过让 proposer 先生成候选 token，再由目标模型验证，从而降低逐 token 延迟。

**EN:** In the right workload regime, especially medium-to-low QPS latency-sensitive serving, speculation can materially improve user experience.
**CN:** 在合适的负载区间中，尤其是中低 QPS、对延迟敏感的服务场景里，投机解码可以显著改善用户体验。

**EN:** vLLM supports multiple proposers: draft models, EAGLE, Medusa, MTP, n-gram, suffix decoding, and custom proposer classes.
**CN:** vLLM 支持多种 proposer：draft model、EAGLE、Medusa、MTP、n-gram、suffix decoding 以及自定义 proposer 类。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The main speculative config fields include method, draft model, number of speculative tokens, and draft TP size where relevant.
- **CN:** 核心 speculative 配置字段包括 method、draft model、投机 token 数，以及在需要时的 draft TP 大小。

- **EN:** `DraftModelProposer` enforces draft-target vocabulary consistency and currently requires draft TP to match target TP to avoid cache/compile issues.
- **CN:** `DraftModelProposer` 会检查 draft 与 target 的词表一致性，并且当前要求 draft TP 与 target TP 匹配，以避免缓存和编译问题。

- **EN:** `EagleProposer` passes hidden states to the proposer model, reflecting the EAGLE family’s hidden-state-based drafting strategy.
- **CN:** `EagleProposer` 会把隐藏状态传给 proposer 模型，这体现了 EAGLE 家族基于隐藏状态进行提议的策略。

- **EN:** `MedusaProposer` generates several draft tokens by running Medusa heads over target hidden states and taking argmax per head.
- **CN:** `MedusaProposer` 会在目标隐藏状态上运行多个 Medusa 头，并对每个头取 argmax 来生成多个候选 token。

- **EN:** The n-gram proposer is deliberately lightweight and uses Numba-accelerated CPU logic to match repeated prompt substrings.
- **CN:** n-gram proposer 则刻意保持轻量，使用 Numba 加速的 CPU 逻辑来匹配重复的 prompt 子串。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/features/speculative_decoding/README.md` is the feature-level overview and method-selection guide.
- **CN:** `docs/features/speculative_decoding/README.md` 是该特性的总体说明与方法选择指南。

- **EN:** `vllm/v1/spec_decode/draft_model.py` implements draft-model-specific checks and model creation.
- **CN:** `vllm/v1/spec_decode/draft_model.py` 实现了 draft model 的专用检查与模型创建。

- **EN:** `vllm/v1/spec_decode/eagle.py`, `medusa.py`, and `ngram_proposer.py` show three very different proposer strategies.
- **CN:** `vllm/v1/spec_decode/eagle.py`、`medusa.py` 与 `ngram_proposer.py` 展示了三种风格截然不同的 proposer 策略。

- **EN:** `tests/spec_decode` and `tests/v1/spec_decode` are important if you want to understand correctness guarantees and regressions.
- **CN:** `tests/spec_decode` 与 `tests/v1/spec_decode` 对于理解正确性保证与回归问题非常重要。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** Speculation is not a universal free lunch; it works best when draft proposals are fast and acceptance rates stay high.
- **CN:** 投机解码并不是放之四海而皆准的“免费午餐”；只有当提议足够快且接受率较高时，它才真正有效。

- **EN:** Model-based proposers usually give better speedups than n-gram or suffix methods, but they also require more moving pieces.
- **CN:** 基于模型的 proposer 往往比 n-gram 或 suffix 方法带来更大加速，但其系统复杂度也更高。

- **EN:** From a scheduler perspective, speculation increases lookahead state and verification work, so it interacts tightly with batching logic.
- **CN:** 从调度器视角看，投机解码会增加前瞻状态与验证工作，因此它与批处理逻辑高度耦合。

- **EN:** When benchmarking speculation, measure acceptance rate, ITL, TTFT, and memory overhead together rather than looking at one metric only.
- **CN:** 评估投机解码时，应同时观察接受率、ITL、TTFT 与内存开销，而不是只盯着单一指标。

**EN:** Speculative decoding is vLLM’s way of turning “predict then verify” into a practical serving optimization.
**CN:** 投机解码可以理解为 vLLM 将“先猜再验”的思想工程化为可用的服务优化。

## 11. Quantization / 11. 量化

**EN:** vLLM supports a large quantization matrix: AWQ, GPTQ, FP8, MXFP4, NVFP4, compressed-tensors, ModelOpt, online quantization, and more.
**CN:** vLLM 支持非常丰富的量化矩阵：AWQ、GPTQ、FP8、MXFP4、NVFP4、compressed-tensors、ModelOpt、在线量化等。

**EN:** Quantization is essential for fitting larger models, reducing bandwidth pressure, and unlocking higher concurrency on the same hardware.
**CN:** 量化对于装载更大模型、降低带宽压力，以及在相同硬件上提升并发能力都至关重要。

**EN:** The system uses `QuantizationConfig` subclasses to decide, per layer type, which quantized method should wrap linear layers, MoE layers, and KV cache logic.
**CN:** 系统通过 `QuantizationConfig` 子类，按层类型决定哪种量化方法应包装线性层、MoE 层以及 KV 缓存逻辑。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The quantization docs list user-facing formats, while the code organizes them into config classes and method dispatchers.
- **CN:** 量化文档列出面向用户的格式，而代码则把它们组织成配置类与方法分发器。

- **EN:** `AWQConfig` focuses on 4-bit weight quantization with group size and zero-point settings, and can fall back for MoE compatibility.
- **CN:** `AWQConfig` 主要针对 4-bit 权重量化，包含 group size 与 zero-point 设置，并能在 MoE 兼容性不足时回退。

- **EN:** `AutoGPTQConfig` supports 4-bit and 8-bit styles, including dynamic per-module overrides via regex-based rules.
- **CN:** `AutoGPTQConfig` 支持 4-bit 与 8-bit 风格，还支持通过正则规则做模块级动态覆盖。

- **EN:** `Fp8Config` can route quantization to linear layers, routed experts, and attention/KV cache paths depending on the layer class.
- **CN:** `Fp8Config` 会根据层类型，把量化逻辑分发到线性层、路由专家层以及注意力/KV 缓存路径。

- **EN:** `CompressedTensorsConfig` is especially important because it maps target schemes, sparsity, transforms, and KV cache quantization in one framework.
- **CN:** `CompressedTensorsConfig` 特别重要，因为它把目标量化方案、稀疏性、变换和 KV 缓存量化统一到了一个框架中。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/features/quantization/README.md` is the user-facing index for supported quantization methods and hardware coverage.
- **CN:** `docs/features/quantization/README.md` 是支持的量化方法与硬件覆盖情况的用户入口。

- **EN:** `vllm/model_executor/layers/quantization/fp8.py`, `awq.py`, and `auto_gptq.py` show three representative in-tree quantization config styles.
- **CN:** `vllm/model_executor/layers/quantization/fp8.py`、`awq.py` 与 `auto_gptq.py` 展示了三种具有代表性的仓库内量化配置风格。

- **EN:** `vllm/model_executor/layers/quantization/compressed_tensors/compressed_tensors.py` is the core entry for compressed-tensors integration.
- **CN:** `vllm/model_executor/layers/quantization/compressed_tensors/compressed_tensors.py` 是 compressed-tensors 集成的核心入口。

- **EN:** `vllm/model_executor/layers/quantization/mxfp4.py` and `modelopt.py` cover MXFP4 and NVFP4/ModelOpt-specific paths.
- **CN:** `vllm/model_executor/layers/quantization/mxfp4.py` 与 `modelopt.py` 覆盖了 MXFP4 与 NVFP4/ModelOpt 专用路径。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** Choose quantization by hardware first, then by acceptable accuracy loss, then by kernel maturity.
- **CN:** 选择量化方案时，应先看硬件支持，再看可接受的精度损失，最后看内核成熟度。

- **EN:** Online quantization is attractive for easy deployment because it can quantize at load time without a pre-quantized checkpoint.
- **CN:** 在线量化之所以有吸引力，是因为它能在加载时完成量化，而不需要提前准备量化后的检查点。

- **EN:** Checkpoint-based quantization is usually better when you want maximum control over calibration and reproducibility.
- **CN:** 如果你希望对校准过程和可复现性有更强控制，基于检查点的量化通常更好。

- **EN:** Quantization interacts with MoE, KV cache, attention backends, and custom kernels, so do not evaluate it as an isolated feature.
- **CN:** 量化会与 MoE、KV 缓存、注意力后端和自定义内核相互影响，因此不能把它当成孤立特性来评估。

**EN:** In vLLM, quantization is not one switch but a family of per-layer execution strategies tied to hardware-aware kernels.
**CN:** 在 vLLM 中，量化不是一个单独开关，而是一整套与硬件感知内核绑定的按层执行策略家族。

## 12. Fused MoE / 12. 融合混合专家

**EN:** vLLM has deep support for Mixture-of-Experts models, including fused routing, modular expert kernels, and expert-parallel all-to-all backends.
**CN:** vLLM 对 Mixture-of-Experts 模型有很深的支持，包括融合路由、模块化专家内核以及专家并行 all-to-all 后端。

**EN:** MoE inference is hard because compute, routing, communication, and quantization all interact at once.
**CN:** MoE 推理之所以困难，是因为计算、路由、通信与量化会同时相互作用。

**EN:** The modular kernel design splits MoE execution into prepare/finalize logic, expert kernels, and top-k weighting/reduction.
**CN:** 模块化内核设计把 MoE 执行拆成 prepare/finalize 逻辑、专家内核，以及 top-k 权重应用与归约。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The modular MoE framework distinguishes “standard/contiguous” and “batched” activation formats depending on all-to-all dispatch style.
- **CN:** 模块化 MoE 框架会根据 all-to-all 分发方式区分“标准/连续”与“批式”激活格式。

- **EN:** Prepare/finalize classes handle quantization plus dispatch/combine, while expert classes handle permute, GEMMs, activation, and unpermute.
- **CN:** prepare/finalize 类负责量化与 dispatch/combine，expert 类则负责 permute、GEMM、激活以及 unpermute。

- **EN:** The feature matrix lists multiple all-to-all backends such as naive, deepep, and flashinfer variants.
- **CN:** 功能矩阵列出了多种 all-to-all 后端，例如 naive、deepep 和 flashinfer 变体。

- **EN:** Expert kernels support a broad set of quantization styles including fp8, nvfp4, mxfp4, int4, and int8 depending on backend.
- **CN:** 不同专家内核支持广泛的量化风格，包括 fp8、nvfp4、mxfp4、int4 和 int8，具体取决于后端。

- **EN:** The CUDA extension registers MoE-specific ops for routing, permutation, block alignment, grouped top-k, and specialized GEMMs.
- **CN:** CUDA 扩展注册了多种 MoE 专用算子，用于路由、置换、块对齐、grouped top-k 与专用 GEMM。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/design/fused_moe_modular_kernel.md` explains the three-part modular decomposition of MoE execution.
- **CN:** `docs/design/fused_moe_modular_kernel.md` 解释了 MoE 执行的三段式模块化分解。

- **EN:** `docs/design/moe_kernel_features.md` is the best matrix of backends, activation formats, quantization schemes, and supported kernels.
- **CN:** `docs/design/moe_kernel_features.md` 是理解后端、激活格式、量化方案与支持内核的最佳矩阵。

- **EN:** `csrc/moe/torch_bindings.cpp` shows the custom ops exposed for routing and fused MoE execution.
- **CN:** `csrc/moe/torch_bindings.cpp` 展示了为路由与融合 MoE 执行暴露的自定义算子。

- **EN:** `vllm/model_executor/layers/fused_moe/*` is the implementation tree for experts, prepare/finalize logic, and kernel selection.
- **CN:** `vllm/model_executor/layers/fused_moe/*` 是 experts、prepare/finalize 逻辑与内核选择的实现目录。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** Do not think about MoE as only “more experts”; think about dispatch format, communication topology, quantization path, and reduction path together.
- **CN:** 不要把 MoE 只理解为“更多专家”，而要同时考虑分发格式、通信拓扑、量化路径与归约路径。

- **EN:** Expert parallelism is especially sensitive to communication backend quality, because routing scatters tokens unevenly by design.
- **CN:** 专家并行对通信后端质量特别敏感，因为路由天然会不均匀地把 token 分散到各专家。

- **EN:** Kernel family selection can dominate performance more than the raw MoE idea, especially for low-latency production setups.
- **CN:** 在低延迟生产场景中，内核家族选择对性能的影响往往比“是否是 MoE”这个事实本身更大。

- **EN:** When debugging MoE correctness, inspect top-k indices, token permutation, expert map, and activation format first.
- **CN:** 调试 MoE 正确性时，应优先检查 top-k 索引、token 置换、expert map 和激活格式。

**EN:** vLLM treats MoE as a systems problem, not just a model-layer problem.
**CN:** vLLM 把 MoE 看作一个系统问题，而不仅仅是模型层问题。

## 13. LoRA Serving / 13. LoRA 服务

**EN:** vLLM can serve LoRA adapters efficiently on top of a shared base model, including per-request adapters and runtime loading.
**CN:** vLLM 可以在共享基座模型之上高效服务 LoRA 适配器，包括按请求切换适配器和运行时动态加载。

**EN:** This is operationally valuable because many customized models differ only by small adapter weights rather than full checkpoints.
**CN:** 这在运维上非常有价值，因为很多定制模型只是在小规模适配器权重上不同，而不需要整套检查点。

**EN:** At request time, the worker-side LoRA manager loads requested adapters, activates them, and can unload or evict older ones depending on policy.
**CN:** 在请求运行时，工作进程侧 LoRA 管理器会加载所需适配器、激活它们，并根据策略卸载或淘汰旧适配器。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The user-facing API supports `enable_lora=True` and request-level `LoRARequest` objects.
- **CN:** 面向用户的 API 支持 `enable_lora=True` 以及请求级 `LoRARequest` 对象。

- **EN:** The server can preload adapters via `--lora-modules` or dynamically load/unload them through dedicated HTTP endpoints.
- **CN:** 服务器既可以通过 `--lora-modules` 预加载适配器，也可以通过专用 HTTP 接口动态加载/卸载。

- **EN:** `WorkerLoRAManager` validates adapter configuration, loads weights, and coordinates the active adapter set on workers.
- **CN:** `WorkerLoRAManager` 会验证适配器配置、加载权重，并在工作进程上协调当前激活的适配器集合。

- **EN:** An LRU variant exists so adapters can be cached and evicted under capacity constraints instead of always reloaded from scratch.
- **CN:** 系统还提供 LRU 版本，使适配器可以在容量受限时被缓存和淘汰，而不必每次都重新加载。

- **EN:** The LoRA docs also cover resolver plugins, including filesystem and Hugging Face Hub resolvers for runtime discovery.
- **CN:** LoRA 文档还介绍了解析器插件，包括文件系统与 Hugging Face Hub 解析器，用于运行时发现适配器。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/features/lora.md` is the main user and operator guide for LoRA serving.
- **CN:** `docs/features/lora.md` 是 LoRA 服务的主要用户与运维指南。

- **EN:** `vllm/lora/worker_manager.py` is the core worker-side manager for loading and activating adapters.
- **CN:** `vllm/lora/worker_manager.py` 是工作进程侧加载和激活适配器的核心管理器。

- **EN:** `vllm/lora/model_manager.py` and `punica_wrapper/*` implement lower-level adapter handling and kernel-facing metadata paths.
- **CN:** `vllm/lora/model_manager.py` 与 `punica_wrapper/*` 实现了更底层的适配器处理与面向内核的元数据路径。

- **EN:** `vllm/plugins/lora_resolvers/*` contains built-in plugin resolvers for runtime LoRA lookup.
- **CN:** `vllm/plugins/lora_resolvers/*` 包含用于运行时查找 LoRA 的内置解析器插件。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** LoRA is only cheap if adapter switching, mapping, and kernel metadata stay efficient; the serving stack is designed exactly for that.
- **CN:** LoRA 只有在适配器切换、映射以及内核元数据保持高效时才真正便宜，而整个服务栈正是围绕这一点设计的。

- **EN:** Dynamic runtime loading is powerful but risky, and the docs correctly warn against enabling it in untrusted production environments.
- **CN:** 运行时动态加载很强大，但也有风险；文档明确警告不要在不受信任的生产环境中启用它。

- **EN:** If you see performance cliffs, inspect adapter-slot count, CPU cache count, adapter rank, and whether adapters are being thrashed.
- **CN:** 如果出现性能断崖，应检查适配器槽位数、CPU 缓存数量、适配器 rank，以及是否发生适配器抖动。

- **EN:** For MoE models, note the additional complexity around 2D and 3D LoRA formats and mixed-format serving.
- **CN:** 对于 MoE 模型，还要注意 2D/3D LoRA 格式以及混合格式服务带来的额外复杂性。

**EN:** LoRA serving in vLLM is a production feature, not an afterthought demo feature.
**CN:** vLLM 中的 LoRA 服务是一个生产级特性，而不是事后补上的演示功能。

## 14. Multimodal Models / 14. 多模态模型

**EN:** vLLM supports multimodal models by treating media preprocessing, placeholder-token alignment, and embedding merge as first-class runtime tasks.
**CN:** vLLM 通过把媒体预处理、占位符 token 对齐与嵌入合并视为一等运行时任务，从而支持多模态模型。

**EN:** Multimodal serving is not just “add images”; it forces the engine to manage text tokens and external media-derived features coherently.
**CN:** 多模态服务并不只是“加上图片”那么简单；它要求引擎把文本 token 与外部媒体特征以一致方式管理起来。

**EN:** The multimodal processor layer determines prompt updates, dummy text construction, processor output caching, and final correspondence between placeholders and media inputs.
**CN:** 多模态处理层负责决定 prompt 更新、dummy text 构造、处理器输出缓存，以及占位符与媒体输入之间的最终对应关系。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** `SupportsMultiModal` is the key capability protocol for models that accept multimodal inputs.
- **CN:** `SupportsMultiModal` 是接受多模态输入模型的关键能力协议。

- **EN:** The design doc emphasizes prompt update detection because placeholder expansion must stay consistent with HF processor behavior.
- **CN:** 设计文档强调 prompt update detection，因为占位符扩展必须与 HF processor 的行为保持一致。

- **EN:** Dummy text is used when tokenized text and multimodal inputs need to be processed together without letting HF processors fail.
- **CN:** 当需要把已 token 化文本与多模态输入一起处理，又不能让 HF processor 报错时，就会使用 dummy text。

- **EN:** Automatic prompt updating then patches placeholder tokens so text-side and media-side outputs still line up.
- **CN:** 随后，自动 prompt 更新机制会修补占位符 token，使文本侧与媒体侧输出重新对齐。

- **EN:** Processor output caching matters because some HF multimodal processors are slow enough to become a frontend bottleneck.
- **CN:** 处理器输出缓存之所以重要，是因为某些 HF 多模态处理器足够慢，足以成为前端瓶颈。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/design/mm_processing.md` is the clearest explanation of multimodal prompt update detection and dummy-text handling.
- **CN:** `docs/design/mm_processing.md` 是理解多模态 prompt 更新检测与 dummy text 机制的最清晰文档。

- **EN:** `vllm/model_executor/models/interfaces.py` defines `SupportsMultiModal` and related flags.
- **CN:** `vllm/model_executor/models/interfaces.py` 定义了 `SupportsMultiModal` 及相关标志。

- **EN:** `vllm/multimodal/registry.py` and `vllm/multimodal/processing/*` implement multimodal registry and processing logic.
- **CN:** `vllm/multimodal/registry.py` 与 `vllm/multimodal/processing/*` 实现了多模态注册与处理逻辑。

- **EN:** `vllm/multimodal/image.py`, `audio.py`, and `video.py` contain media-specific utilities.
- **CN:** `vllm/multimodal/image.py`、`audio.py` 与 `video.py` 包含媒体类型特定的工具函数。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** In multimodal systems, prompt formatting bugs are often just as important as tensor-shape bugs.
- **CN:** 在多模态系统中，prompt 格式错误往往和张量形状错误一样重要。

- **EN:** If an image or audio request behaves strangely, inspect placeholder tokens, processor caching, and chat-template formatting together.
- **CN:** 如果图片或音频请求表现异常，应同时检查占位符 token、处理器缓存与 chat template 格式。

- **EN:** Do not assume text tokenization and multimodal preprocessing are separable; vLLM spends real design effort making them composable.
- **CN:** 不要假设文本分词和多模态预处理天然可分离；vLLM 花了很多设计功夫让它们可组合。

- **EN:** For throughput tuning, frontend preprocessing and media loading threads can matter almost as much as backend GPU kernels.
- **CN:** 在吞吐调优中，前端预处理与媒体加载线程的重要性有时几乎不亚于后端 GPU 内核。

**EN:** Multimodal support in vLLM is fundamentally about maintaining alignment between external media and internal token-space execution.
**CN:** vLLM 的多模态支持，本质上是在维护外部媒体与内部 token 空间执行之间的对齐关系。

## 15. API Server / 15. API 服务器

**EN:** The API server turns the engine into a network service and exposes OpenAI-compatible, Anthropic-compatible, speech, pooling, and custom endpoints.
**CN:** API 服务器把引擎封装成网络服务，并暴露 OpenAI 兼容、Anthropic 兼容、语音、池化以及自定义端点。

**EN:** For many users, the API server is the product surface of vLLM.
**CN:** 对于许多用户来说，API 服务器就是 vLLM 的产品界面。

**EN:** `vllm serve` builds a FastAPI app, registers routers based on supported tasks, and uses `AsyncLLM` under the hood.
**CN:** `vllm serve` 会构建 FastAPI 应用，按支持任务注册路由，并在底层使用 `AsyncLLM`。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The OpenAI-compatible server supports completions, chat, responses, embeddings, audio transcription, audio translation, and realtime APIs.
- **CN:** OpenAI 兼容服务器支持 completions、chat、responses、embeddings、音频转写、音频翻译和 realtime API。

- **EN:** The server also supports custom endpoints such as tokenization, pooling, rerank, and score APIs.
- **CN:** 服务器还支持如 tokenization、pooling、rerank 与 score 等自定义端点。

- **EN:** Chat templates are essential because the server needs a model-specific way to convert role-based messages into raw prompts.
- **CN:** chat template 非常关键，因为服务器需要一种模型特定的方法把 role-based message 转换成原始 prompt。

- **EN:** The Anthropic Messages API is implemented by converting Anthropic-style requests into the OpenAI chat-serving path and back.
- **CN:** Anthropic Messages API 的实现方式，是先把 Anthropic 风格请求转换为 OpenAI chat 服务路径，再转换回来。

- **EN:** There is also a separate gRPC entrypoint for environments that prefer RPC over HTTP/JSON APIs.
- **CN:** 此外还提供独立的 gRPC 入口，以支持更偏好 RPC 而非 HTTP/JSON 的环境。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/serving/openai_compatible_server.md` is the main server reference for supported endpoints and extra parameters.
- **CN:** `docs/serving/openai_compatible_server.md` 是支持端点与额外参数的主要服务端参考文档。

- **EN:** `vllm/entrypoints/openai/api_server.py` builds the FastAPI app and conditionally attaches routers.
- **CN:** `vllm/entrypoints/openai/api_server.py` 构建 FastAPI 应用并按条件挂载路由。

- **EN:** `vllm/entrypoints/anthropic/api_router.py` and `anthropic/serving.py` implement the Messages API compatibility layer.
- **CN:** `vllm/entrypoints/anthropic/api_router.py` 与 `anthropic/serving.py` 实现了 Messages API 兼容层。

- **EN:** `vllm/entrypoints/grpc_server.py` provides the gRPC server using `AsyncLLM` as the backend engine.
- **CN:** `vllm/entrypoints/grpc_server.py` 提供了以 `AsyncLLM` 为后端引擎的 gRPC 服务。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** When requests fail at the API layer, check chat templates, extra-body fields, and model task support before blaming the engine.
- **CN:** 当请求在 API 层失败时，应先检查 chat template、extra-body 字段与模型任务支持，而不要立刻怀疑引擎。

- **EN:** The server can apply model-side `generation_config.json` defaults unless you disable that behavior.
- **CN:** 除非显式关闭，否则服务器会应用模型仓库中的 `generation_config.json` 默认值。

- **EN:** For production, request IDs, offline docs mode, and security configuration around model-loading endpoints all matter.
- **CN:** 在生产环境中，请求 ID、离线文档模式，以及与模型加载端点相关的安全配置都很重要。

- **EN:** Treat the API server as a frontend system with its own performance budget, not as a thin wrapper over inference.
- **CN:** 应把 API 服务器视为一个有自身性能预算的前端系统，而不是推理功能上的一层薄封装。

**EN:** The API server is where vLLM’s engine abstractions meet real client protocols.
**CN:** API 服务器就是 vLLM 引擎抽象与真实客户端协议相接的地方。

## 16. Structured Output / 16. 结构化输出

**EN:** Structured output constrains generation so the model emits JSON, regex-conforming text, grammar-conforming text, a choice item, or a tagged structure.
**CN:** 结构化输出通过约束生成过程，使模型输出 JSON、符合正则的文本、符合语法的文本、枚举选项或带标签的结构。

**EN:** For agent systems, data extraction, and reliable tool use, structure is often more important than free-form fluency.
**CN:** 对于智能体系统、数据抽取和可靠工具使用来说，结构往往比自由文本流畅性更重要。

**EN:** vLLM supports structured outputs through backends such as xgrammar and guidance, selected automatically or explicitly.
**CN:** vLLM 通过 xgrammar 与 guidance 等后端支持结构化输出，可自动选择，也可显式指定。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The modern request field is `structured_outputs`; older `guided_*` fields are deprecated and removed.
- **CN:** 现代请求字段是 `structured_outputs`；旧的 `guided_*` 字段已经废弃并移除。

- **EN:** Supported modes include `choice`, `regex`, `json`, `grammar`, and `structural_tag`.
- **CN:** 支持的模式包括 `choice`、`regex`、`json`、`grammar` 与 `structural_tag`。

- **EN:** JSON output can be driven directly from JSON Schema or from a Pydantic model’s generated schema.
- **CN:** JSON 输出既可以直接由 JSON Schema 驱动，也可以来自 Pydantic 模型生成的 schema。

- **EN:** Grammar-based generation uses context-free grammar specifications, which are more expressive than simple regex constraints.
- **CN:** 基于语法的生成使用上下文无关文法，表达能力比简单正则约束更强。

- **EN:** Reasoning mode can interact with structured outputs, but some model/parser combinations require extra enabling flags.
- **CN:** 推理模式可以与结构化输出结合使用，但某些模型/解析器组合需要额外启用标志。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/features/structured_outputs.md` is the main user guide and includes OpenAI API examples.
- **CN:** `docs/features/structured_outputs.md` 是主要用户指南，并包含 OpenAI API 示例。

- **EN:** `vllm/v1/structured_output/*` contains backend adapters and request utilities for V1 structured output support.
- **CN:** `vllm/v1/structured_output/*` 包含 V1 结构化输出支持所需的后端适配器与请求工具。

- **EN:** `vllm/entrypoints/openai/chat_completion/protocol.py` and related protocol files define the request/response shapes exposed via API.
- **CN:** `vllm/entrypoints/openai/chat_completion/protocol.py` 及相关协议文件定义了通过 API 暴露的请求/响应结构。

- **EN:** `examples/features/structured_outputs/*` provide runnable examples for practical usage.
- **CN:** `examples/features/structured_outputs/*` 提供了可直接运行的实践示例。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** When output shape matters, prefer constrained decoding over trying to coerce free-form text after the fact.
- **CN:** 当输出结构非常重要时，应优先使用约束解码，而不是事后再去修补自由文本。

- **EN:** A strong prompt still helps even when decoding is constrained, because the model must still choose semantically correct field contents.
- **CN:** 即便有约束解码，好的 prompt 仍然重要，因为模型仍需要为各字段选择语义上正确的内容。

- **EN:** Structured output backends differ in syntax and capabilities, so backend choice can affect both quality and compatibility.
- **CN:** 结构化输出后端在语法与能力上存在差异，因此后端选择会影响质量与兼容性。

- **EN:** For tool use, structured output and function calling are closely related but not identical: one constrains shape, the other adds tool semantics.
- **CN:** 对于工具使用而言，结构化输出与函数调用关系紧密但并不相同：前者约束输出形状，后者增加工具语义。

**EN:** Structured output is how vLLM turns generation into a typed interface instead of only a text stream.
**CN:** 结构化输出让 vLLM 把生成过程变成一种“带类型接口”，而不只是文本流。

## 17. CUDA Kernels / 17. CUDA 内核

**EN:** A large part of vLLM’s performance comes from custom CUDA kernels and custom-op bindings rather than relying only on generic PyTorch eager ops.
**CN:** vLLM 的很大一部分性能来自自定义 CUDA 内核与自定义算子绑定，而不仅仅依赖通用 PyTorch eager 操作。

**EN:** These kernels exist because serving workloads stress memory layout, batching shape, and communication patterns that generic kernels do not optimize for.
**CN:** 之所以需要这些内核，是因为服务负载对内存布局、批形状以及通信模式提出了通用内核未必会专门优化的要求。

**EN:** C++/CUDA code registers operators through Torch extension libraries, and Python wrappers call those ops from higher-level runtime code.
**CN:** C++/CUDA 代码通过 Torch 扩展库注册算子，而 Python 包装层则从更高层运行时代码中调用这些算子。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The main torch bindings expose paged attention, merge-attention-state, activation, normalization, rotary embedding, top-k, and quantization-related ops.
- **CN:** 主 torch 绑定暴露了分页注意力、注意力状态合并、激活、归一化、旋转位置编码、top-k 以及量化相关算子。

- **EN:** MoE has a separate binding library for routing, alignment, permutation, and specialized expert GEMMs.
- **CN:** MoE 还拥有独立的绑定库，用于路由、对齐、置换和专家专用 GEMM。

- **EN:** ROCm has its own custom op namespace and kernel implementations where the backend behavior diverges.
- **CN:** 在后端行为存在差异的地方，ROCm 也有自己的自定义算子命名空间与内核实现。

- **EN:** Kernel registration matters because it forms the bridge between compiled native code and the Python runtime used by the engine.
- **CN:** 内核注册之所以重要，是因为它构成了编译后原生代码与引擎 Python 运行时之间的桥梁。

- **EN:** A good rule of thumb is that any repeated tensor transformation in the hot path is a candidate for a fused custom op in vLLM.
- **CN:** 经验上可以认为：任何热路径中反复出现的张量变换，都是 vLLM 中潜在的融合自定义算子候选。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `csrc/torch_bindings.cpp` registers the main extension ops, including PagedAttention and many fused tensor transforms.
- **CN:** `csrc/torch_bindings.cpp` 注册了主扩展算子，包括 PagedAttention 与许多融合张量变换。

- **EN:** `csrc/moe/torch_bindings.cpp` registers MoE routing and fused expert operations.
- **CN:** `csrc/moe/torch_bindings.cpp` 注册了 MoE 路由与融合专家操作。

- **EN:** `vllm/_custom_ops.py` is the Python-side wrapper layer that calls these torch ops.
- **CN:** `vllm/_custom_ops.py` 是调用这些 torch 算子的 Python 包装层。

- **EN:** `csrc/attention/*`, `csrc/quantization/*`, and `csrc/rocm/*` contain backend-specific native implementations.
- **CN:** `csrc/attention/*`、`csrc/quantization/*` 与 `csrc/rocm/*` 包含后端特定的原生实现。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** Kernel performance is often constrained more by memory traffic and layout than by raw arithmetic intensity.
- **CN:** 内核性能往往更受内存流量与布局限制，而不是单纯受算术强度限制。

- **EN:** When reading a kernel path, always map the Python call site, custom-op wrapper, C++ binding, and CUDA source together.
- **CN:** 阅读某条内核路径时，应始终把 Python 调用点、自定义算子包装层、C++ 绑定和 CUDA 源码一起对应起来。

- **EN:** A fused kernel is valuable not just because it runs faster, but because it can reduce launches and intermediate allocations.
- **CN:** 融合内核的价值不仅在于更快，还在于减少 kernel launch 与中间张量分配。

- **EN:** If a performance regression appears only on one platform, compare its custom-kernel path before suspecting high-level scheduling.
- **CN:** 如果某个性能回归只在单一平台出现，应先比较其自定义内核路径，再考虑高层调度。

**EN:** Custom kernels are where vLLM’s systems ideas become real GPU instructions.
**CN:** 自定义内核就是 vLLM 的系统设计思想落地为真实 GPU 指令的地方。

## 18. torch.compile Integration / 18. torch.compile 集成

**EN:** In V1, `torch.compile` is a core optimization path rather than an optional experiment.
**CN:** 在 V1 中，`torch.compile` 已经是核心优化路径，而不是可有可无的实验功能。

**EN:** It matters because vLLM wants compilation to finish before serving begins, avoiding surprise latency spikes during live traffic.
**CN:** 它之所以重要，是因为 vLLM 希望在真正开始服务前就完成编译，从而避免在线流量中出现意外延迟尖峰。

**EN:** vLLM hashes relevant configs and traced code, stores compile artifacts in a dedicated cache directory, and can split the graph around attention.
**CN:** vLLM 会对相关配置与被跟踪代码求哈希，把编译产物存入专用缓存目录，并可在注意力算子处分裂计算图。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The compile cache includes model/config factors, PyTorch compiler settings, and traced source files.
- **CN:** 编译缓存的哈希因素包括模型/配置、PyTorch 编译器设置以及被跟踪的源码文件。

- **EN:** The docs explicitly state that no requests should trigger fresh compilation after the server starts accepting traffic.
- **CN:** 文档明确指出：服务器开始接收流量后，不应再由请求触发新的编译。

- **EN:** Dynamic shape handling can use backed, unbacked, or backed-size-oblivious modes with different safety/performance tradeoffs.
- **CN:** 动态形状处理可使用 backed、unbacked 或 backed-size-oblivious 模式，各自对应不同的安全性/性能权衡。

- **EN:** Attention is wrapped as a custom op in the compile path so Dynamo can treat it as a graph node rather than tracing through all internals.
- **CN:** 在编译路径中，注意力会被包装成自定义算子，这样 Dynamo 就能把它当作图节点，而不是深入跟踪其所有内部细节。

- **EN:** Static compile sizes can be precompiled and auto-tuned, trading startup time for optimized kernels on common batch sizes.
- **CN:** 静态 compile sizes 可以被预编译并自动调优，以更长启动时间换取常见 batch size 的优化内核。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/design/torch_compile.md` is the central design walkthrough for compile cache, dynamic shapes, graph splitting, and shape specialization.
- **CN:** `docs/design/torch_compile.md` 是理解编译缓存、动态形状、图切分与形状特化的核心设计说明。

- **EN:** `vllm/compilation/compiler_interface.py` and related config hashing code participate in compile-cache identity.
- **CN:** `vllm/compilation/compiler_interface.py` 及相关配置哈希代码共同参与编译缓存身份构建。

- **EN:** `vllm/model_executor/models/*` forward methods become the traced roots for compile-time graph capture.
- **CN:** `vllm/model_executor/models/*` 中的 forward 方法会成为编译时图捕获的跟踪根。

- **EN:** `vllm/compilation/*` contains wrappers and utilities that mediate compile behavior and cache usage.
- **CN:** `vllm/compilation/*` 包含调解编译行为与缓存使用的包装器和工具。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** When startup is too slow, inspect compile cache reuse before changing model code or disabling compile entirely.
- **CN:** 当启动过慢时，应先检查编译缓存是否复用，而不是立刻修改模型代码或完全关闭编译。

- **EN:** When debugging correctness, temporarily disabling compile or compile cache can help separate compiler issues from model/runtime issues.
- **CN:** 在调试正确性问题时，临时关闭编译或编译缓存，有助于区分编译器问题与模型/运行时问题。

- **EN:** Graph splitting around attention is a practical compromise: keep compilation benefits while respecting complicated attention backends.
- **CN:** 围绕注意力算子切图是一种现实折中：既保留编译收益，又尊重复杂注意力后端的特殊性。

- **EN:** Compile is especially powerful in vLLM because it is integrated with serving assumptions, not just raw training/inference scripts.
- **CN:** `torch.compile` 在 vLLM 中之所以强大，是因为它被纳入了服务系统假设，而不仅仅用于普通训练/推理脚本。

**EN:** vLLM treats compilation as an ahead-of-service optimization pipeline, not a runtime surprise.
**CN:** vLLM 把编译看作服务启动前的优化流水线，而不是运行时的意外行为。

## 19. Distributed KV Transfer / 19. 分布式 KV 传输

**EN:** Distributed KV transfer lets vLLM separate prefill and decode into different instances and move KV cache between them.
**CN:** 分布式 KV 传输让 vLLM 可以把 prefill 与 decode 放在不同实例中执行，并在二者之间传递 KV 缓存。

**EN:** This is useful when you want to tune TTFT and ITL separately or control tail latency by isolating heavy prefill work.
**CN:** 当你希望分别调优 TTFT 与 ITL，或通过隔离重型 prefill 工作来控制尾部延迟时，这项能力非常有用。

**EN:** The disaggregated prefill design runs separate vLLM instances and uses connectors to buffer, transfer, and load KV blocks across processes or hosts.
**CN:** 分离式预填充设计会运行独立的 vLLM 实例，并使用连接器在进程或主机之间缓存、传输和加载 KV 块。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The feature docs explicitly say disaggregated prefilling is for latency control and flexibility, not for increasing throughput.
- **CN:** 特性文档明确指出：分离式预填充主要是为了延迟控制与灵活性，而不是为了提升吞吐。

- **EN:** The connector abstraction is split into concepts like connector, lookup buffer, and pipe.
- **CN:** 连接器抽象被拆分为 connector、lookup buffer 与 pipe 等概念。

- **EN:** Scheduler-side and worker-side connectors cooperate so transfers can be planned at the scheduler and executed at the workers.
- **CN:** 调度器侧与工作进程侧连接器协同工作，使传输既能在调度层规划，也能在工作层执行。

- **EN:** NixlConnector emphasizes fully asynchronous send/receive and transport-backend flexibility such as UCX or LIBFABRIC.
- **CN:** NixlConnector 强调完全异步的发送/接收，以及如 UCX、LIBFABRIC 等传输后端的灵活性。

- **EN:** MooncakeConnector focuses on multi-level cache transport and RDMA-oriented movement, including support for asymmetric TP transfer planning.
- **CN:** MooncakeConnector 则强调多级缓存传输和面向 RDMA 的移动，并支持异构 TP 下的传输规划。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/features/disagg_prefill.md` explains the motivation, abstractions, and workflow for disaggregated prefill.
- **CN:** `docs/features/disagg_prefill.md` 解释了分离式预填充的动机、抽象与工作流程。

- **EN:** `docs/features/nixl_connector_usage.md` and `mooncake_connector_usage.md` explain two concrete connector families.
- **CN:** `docs/features/nixl_connector_usage.md` 与 `mooncake_connector_usage.md` 解释了两类具体连接器。

- **EN:** `docs/design/p2p_nccl_connector.md` explains a P2P NCCL-based connector path and its proxy/router architecture.
- **CN:** `docs/design/p2p_nccl_connector.md` 解释了基于 P2P NCCL 的连接器路径及其代理/路由器架构。

- **EN:** `vllm/distributed/kv_transfer/*` and connector implementations such as `mooncake_connector.py` are the core code paths.
- **CN:** `vllm/distributed/kv_transfer/*` 以及 `mooncake_connector.py` 等连接器实现是核心代码路径。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** Disaggregated serving only makes sense if the connector overhead is lower than the prefill interference you are trying to avoid.
- **CN:** 只有当连接器开销低于你试图避免的 prefill 干扰时，分离式服务才真正有意义。

- **EN:** Transport configuration matters: NIXL uses UCX-style settings, while P2P NCCL paths care about NCCL and buffer sizing.
- **CN:** 传输配置非常关键：NIXL 更依赖 UCX 风格设置，而 P2P NCCL 路径则更关心 NCCL 与缓冲区大小。

- **EN:** Load-failure policy is a real production choice: fail fast, or recompute and risk decode jitter.
- **CN:** KV 加载失败策略是一个真实的生产决策：要么快速失败，要么回退重算并承担 decode 抖动风险。

- **EN:** Think of connectors as distributed memory-management systems, not as simple socket send/recv wrappers.
- **CN:** 应把连接器理解为“分布式内存管理系统”，而不只是简单的 socket 发送/接收封装。

**EN:** Distributed KV transfer pushes vLLM beyond single-engine serving into a family of disaggregated inference topologies.
**CN:** 分布式 KV 传输让 vLLM 超越了单引擎服务，进入了多种解耦式推理拓扑的范畴。

## 20. Performance Tuning / 20. 性能调优

**EN:** Performance tuning in vLLM is always cross-layer: model choice, quantization, batching, KV cache, compilation, kernels, and network all matter.
**CN:** vLLM 的性能调优永远是跨层次的：模型选择、量化、批处理、KV 缓存、编译、内核和网络都会影响结果。

**EN:** The best optimization depends on your bottleneck: memory capacity, memory bandwidth, kernel launch overhead, preprocessing, or network communication.
**CN:** 最佳优化策略取决于瓶颈位置：显存容量、显存带宽、kernel launch 开销、预处理，还是网络通信。

**EN:** A practical loop is: fit the model, inspect KV cache size and concurrency, choose parallelism, evaluate quantization, and then tune graphs/kernels/connectors.
**CN:** 一个实用的调优循环是：先让模型装下，再检查 KV 缓存容量与并发度，然后选择并行策略、评估量化，最后再调图执行、内核与连接器。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The parallelism guide suggests reading the `GPU KV cache size` and `Maximum concurrency` logs after startup to understand serving headroom.
- **CN:** 并行化指南建议在启动后阅读 `GPU KV cache size` 与 `Maximum concurrency` 日志，以理解服务容量余量。

- **EN:** Chunked prefill and prefix caching are major levers for prompt-heavy workloads.
- **CN:** 对于 prompt 较重的负载，分块预填充与前缀缓存是两大关键杠杆。

- **EN:** CUDAGraph mode choice matters: `FULL_AND_PIECEWISE` is often fastest but also uses more memory and capture time.
- **CN:** CUDAGraph 模式选择很关键：`FULL_AND_PIECEWISE` 通常最快，但也会消耗更多显存和捕获时间。

- **EN:** Quantization can dramatically improve fit and throughput, but only if the corresponding kernel path is mature on your hardware.
- **CN:** 量化可以显著改善装载能力和吞吐，但前提是对应硬件上的内核路径已经足够成熟。

- **EN:** For distributed deployments, interconnect quality and RDMA setup can dominate the gains from algorithmic optimizations.
- **CN:** 在分布式部署中，互联质量与 RDMA 配置可能比算法层优化更能决定最终收益。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/serving/parallelism_scaling.md` is the best operational guide for TP/PP choices and distributed deployment tuning.
- **CN:** `docs/serving/parallelism_scaling.md` 是 TP/PP 选择与分布式部署调优的最佳运维指南。

- **EN:** `docs/design/cuda_graphs.md` and `docs/design/torch_compile.md` explain graph-level tuning knobs.
- **CN:** `docs/design/cuda_graphs.md` 与 `docs/design/torch_compile.md` 解释了图执行层面的调优旋钮。

- **EN:** `docs/features/quantization/*` explains the fit/throughput tradeoffs of different quantization families.
- **CN:** `docs/features/quantization/*` 解释了不同量化家族在装载能力与吞吐上的权衡。

- **EN:** `benchmarks/*` and `benchmarks/disagg_benchmarks/*` are the practical places to look for measurement workflows.
- **CN:** `benchmarks/*` 与 `benchmarks/disagg_benchmarks/*` 是寻找测量工作流的实用目录。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** Tune one bottleneck class at a time; otherwise you will not know whether a win came from memory, compute, or scheduling.
- **CN:** 一次只调一类瓶颈；否则你无法知道收益究竟来自显存、计算，还是调度。

- **EN:** Do not benchmark with unrealistic prompt/output shapes; serving systems are shape-sensitive.
- **CN:** 不要用不真实的 prompt/输出形状来压测；服务系统对形状非常敏感。

- **EN:** Measure TTFT and ITL separately, especially if you experiment with chunked prefill or disaggregated prefill.
- **CN:** 务必分别测量 TTFT 与 ITL，尤其是在尝试分块预填充或分离式预填充时。

- **EN:** Always confirm whether the expected optimized backend actually activated, rather than assuming a flag guaranteed it.
- **CN:** 一定要确认预期的优化后端是否真的被激活，而不要假设一个 flag 就足以保证它生效。

**EN:** Performance tuning in vLLM is the art of matching workload shape to the right memory, kernel, and scheduling strategy.
**CN:** vLLM 的性能调优，本质上是把负载形状与合适的内存、内核和调度策略匹配起来。

## 21. Tool Calling / 21. 工具调用

**EN:** Tool calling lets the model produce function-call-like outputs that the application can parse and execute.
**CN:** 工具调用允许模型生成类似函数调用的输出，再由应用程序解析并执行。

**EN:** This is central for agent workflows, coding assistants, retrieval systems, and many structured automation pipelines.
**CN:** 它是智能体工作流、编码助手、检索系统以及许多结构化自动化流程的核心能力。

**EN:** vLLM supports named function calling, `required`, `auto`, and `none` tool-choice modes, using either structured decoding or parser-based extraction.
**CN:** vLLM 支持 named function、`required`、`auto` 与 `none` 四种 tool_choice 模式，可结合结构化解码或解析器抽取。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** Named function calling and `tool_choice="required"` use structured outputs to ensure the function-argument schema is respected.
- **CN:** 命名函数调用与 `tool_choice="required"` 使用结构化输出，以确保函数参数遵守 schema。

- **EN:** `tool_choice="auto"` is different: the model generates more freely and a model-specific parser extracts tool calls from raw text.
- **CN:** `tool_choice="auto"` 则不同：模型会更自由地生成，再由模型特定解析器从原始文本中抽取工具调用。

- **EN:** Because `auto` is parser-based, argument validity is not guaranteed in the same strong way as schema-constrained modes.
- **CN:** 由于 `auto` 基于解析器，因此参数有效性无法像 schema 约束模式那样得到强保证。

- **EN:** Server-side flags include `--enable-auto-tool-choice`, `--tool-call-parser`, and optionally a custom chat template or parser plugin.
- **CN:** 服务端相关 flag 包括 `--enable-auto-tool-choice`、`--tool-call-parser`，以及可选的自定义 chat template 或 parser plugin。

- **EN:** The docs include parser guidance for Hermes, Mistral, Llama 3/4, Granite, and other model families.
- **CN:** 文档为 Hermes、Mistral、Llama 3/4、Granite 等模型家族提供了解析器使用指导。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `docs/features/tool_calling.md` is the main feature guide and should be your first reference.
- **CN:** `docs/features/tool_calling.md` 是主要功能指南，应作为第一参考。

- **EN:** `examples/tool_calling/*` and `examples/tool_chat_template_*.jinja` show practical request payloads and templates.
- **CN:** `examples/tool_calling/*` 与 `examples/tool_chat_template_*.jinja` 展示了实际请求负载与模板。

- **EN:** `vllm/tool_parsers/*` and parser plugin hooks are the extension points for new tool-call formats.
- **CN:** `vllm/tool_parsers/*` 及解析器插件接口，是扩展新工具调用格式的关键入口。

- **EN:** `docs/serving/integrations/claude_code.md` shows how vLLM tool calling can power Anthropic-style coding workflows.
- **CN:** `docs/serving/integrations/claude_code.md` 展示了 vLLM 的工具调用如何为 Anthropic 风格的编码工作流提供后端。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** Use schema-constrained modes when correctness matters more than naturalness.
- **CN:** 当正确性比自然性更重要时，应优先使用 schema 约束模式。

- **EN:** Use `auto` when the model’s native tool format matters and you trust the parser/model combination.
- **CN:** 当模型原生工具格式很重要，并且你信任 parser/模型组合时，可使用 `auto`。

- **EN:** Tool calling quality is heavily affected by chat template quality, not just raw model strength.
- **CN:** 工具调用质量高度依赖 chat template 质量，而不仅仅取决于模型本身强弱。

- **EN:** If tools fail intermittently, inspect parser selection, template formatting, and whether your model actually supports parallel tool calls.
- **CN:** 如果工具调用偶发失败，应检查 parser 选择、模板格式，以及模型是否真的支持并行工具调用。

**EN:** Tool calling in vLLM is a combination of constrained generation, parser engineering, and prompt/template design.
**CN:** vLLM 中的工具调用，本质上是约束生成、解析器工程与 prompt/template 设计的结合。

## 22. Speech-to-Text / 22. 语音转文字

**EN:** vLLM supports speech-to-text serving, including OpenAI-compatible transcription and translation endpoints and native Whisper support in V1.
**CN:** vLLM 支持语音转文字服务，包括 OpenAI 兼容的转写与翻译端点，以及 V1 中对 Whisper 的原生支持。

**EN:** ASR support matters because it expands vLLM beyond text generation into multimodal and speech-serving workloads.
**CN:** ASR 支持的重要性在于，它让 vLLM 从文本生成扩展到了多模态与语音服务工作负载。

**EN:** The speech serving path loads audio, resamples it, optionally chunks it, may auto-detect language, and then builds generation prompts for the model.
**CN:** 语音服务路径会加载音频、重采样、按需切块、可选自动语言检测，然后为模型构建生成 prompt。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The OpenAI-compatible server exposes `/v1/audio/transcriptions` and `/v1/audio/translations` for supported ASR models.
- **CN:** OpenAI 兼容服务器为支持的 ASR 模型暴露了 `/v1/audio/transcriptions` 与 `/v1/audio/translations`。

- **EN:** The V1 guide notes that Whisper is natively supported among encoder-decoder models.
- **CN:** V1 指南指出，在 encoder-decoder 模型中，Whisper 已获得原生支持。

- **EN:** The speech serving base class handles audio file-size checks, decode/resample, chunking, and prompt construction.
- **CN:** 语音服务基础类会处理音频文件大小检查、解码/重采样、切块以及 prompt 构造。

- **EN:** Language detection can be run explicitly by the model class, using constrained sampling over language token IDs.
- **CN:** 语言检测可以由模型类显式执行，通过对语言 token ID 进行约束采样来完成。

- **EN:** The serving layer also handles language-specific chunk separators, since some languages should not insert spaces between decoded segments.
- **CN:** 服务层还会处理与语言相关的分块拼接符，因为某些语言在合并分段结果时不应插入空格。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `vllm/entrypoints/speech_to_text/base/serving.py` is the main implementation file for the speech-serving pipeline.
- **CN:** `vllm/entrypoints/speech_to_text/base/serving.py` 是语音服务管线的主要实现文件。

- **EN:** `docs/serving/openai_compatible_server.md` documents the ASR and translation endpoints from the API perspective.
- **CN:** `docs/serving/openai_compatible_server.md` 从 API 视角记录了 ASR 与翻译端点。

- **EN:** `examples/speech_to_text/*` contains usage examples for ASR serving.
- **CN:** `examples/speech_to_text/*` 包含 ASR 服务的使用示例。

- **EN:** `tests/entrypoints/speech_to_text/*` contains endpoint and validation tests, especially around Whisper behavior.
- **CN:** `tests/entrypoints/speech_to_text/*` 包含端点与校验测试，尤其覆盖了 Whisper 相关行为。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** ASR throughput is affected not only by model execution but also by audio decoding, chunking policy, and frontend I/O.
- **CN:** ASR 吞吐不仅受模型执行影响，也受音频解码、切块策略和前端 I/O 影响。

- **EN:** Large audio files should be chunked thoughtfully, because chunk overlap and merge behavior affect both latency and quality.
- **CN:** 大音频文件应谨慎切块，因为分块重叠与合并行为会同时影响延迟和质量。

- **EN:** If multilingual behavior seems wrong, inspect explicit language settings and auto-detection prompts before changing the model.
- **CN:** 如果多语言行为异常，先检查显式语言设置与自动检测 prompt，再考虑更换模型。

- **EN:** For production APIs, think about file-size limits, timeouts, and backpressure just as seriously as for text endpoints.
- **CN:** 对于生产级语音 API，应像对待文本端点一样认真考虑文件大小限制、超时和背压。

**EN:** Speech support shows that vLLM’s serving engine is general enough to orchestrate more than text-only decoding.
**CN:** 语音支持说明，vLLM 的服务引擎已经足够通用，可以编排不止文本解码这一类任务。

## 23. Build System / 23. 构建系统

**EN:** The build system combines Python packaging with a CMake-based native extension pipeline.
**CN:** 构建系统把 Python 打包流程与基于 CMake 的原生扩展构建流水线结合在一起。

**EN:** This matters because vLLM is not “just Python”; a large portion of its performance-critical logic lives in compiled extensions.
**CN:** 这之所以重要，是因为 vLLM 并不只是“纯 Python”；其大量关键性能逻辑都存在于已编译扩展中。

**EN:** `pyproject.toml` defines the Python build backend, `setup.py` detects target device/backend and launches CMake, and `CMakeLists.txt` builds the native targets.
**CN:** `pyproject.toml` 定义 Python 构建后端，`setup.py` 检测目标设备/后端并启动 CMake，而 `CMakeLists.txt` 负责构建原生目标。

**EN:** Key technical points:
**CN:** 关键技术点：

- **EN:** The build backend is `setuptools.build_meta`, but actual native compilation is orchestrated through a custom `build_ext` path in `setup.py`.
- **CN:** 构建后端是 `setuptools.build_meta`，但实际原生编译是通过 `setup.py` 中自定义的 `build_ext` 路径协调完成的。

- **EN:** `setup.py` auto-detects CUDA, ROCm, XPU, or CPU targets and passes `VLLM_TARGET_DEVICE` into CMake.
- **CN:** `setup.py` 会自动探测 CUDA、ROCm、XPU 或 CPU 目标，并把 `VLLM_TARGET_DEVICE` 传给 CMake。

- **EN:** CMake then defines extension targets such as `_C`, `_moe_C`, `_rocm_C`, and related helper libraries.
- **CN:** 随后 CMake 会定义 `_C`、`_moe_C`、`_rocm_C` 等扩展目标以及相关辅助库。

- **EN:** Utility CMake functions handle Python discovery, torch compiler flags, CUDA arch filtering, and HIP source transformation.
- **CN:** CMake 工具函数负责 Python 发现、torch 编译器标志、CUDA 架构筛选以及 HIP 源码转换。

- **EN:** The torch bindings compiled into these extensions expose custom ops back to the Python package under `torch.ops` namespaces.
- **CN:** 这些扩展中编译出来的 torch 绑定，会把自定义算子重新暴露给 Python 包中的 `torch.ops` 命名空间。

**EN:** Key source files to read:
**CN:** 建议重点阅读的源码/文档：

- **EN:** `pyproject.toml` lists build dependencies, supported Python versions, and the `vllm` CLI entrypoint.
- **CN:** `pyproject.toml` 列出了构建依赖、支持的 Python 版本以及 `vllm` CLI 入口。

- **EN:** `setup.py` is the main Python build driver and contains device detection plus CMake invocation logic.
- **CN:** `setup.py` 是主要的 Python 构建驱动文件，包含设备检测和 CMake 调用逻辑。

- **EN:** `CMakeLists.txt` is the top-level native build graph and delegates some device-specific work to `cmake/*` files.
- **CN:** `CMakeLists.txt` 是顶层原生构建图，并把部分设备特定工作委托给 `cmake/*` 文件。

- **EN:** `cmake/utils.cmake` and `csrc/torch_bindings.cpp` are the most useful files for understanding the Python/native boundary.
- **CN:** `cmake/utils.cmake` 与 `csrc/torch_bindings.cpp` 是理解 Python/原生边界最有帮助的文件。

**EN:** Practical reading and tuning notes:
**CN:** 实践阅读与调优提示：

- **EN:** If the build fails, first identify whether it is a Python packaging issue, a compiler/toolchain issue, or a backend-detection issue.
- **CN:** 如果构建失败，首先判断它是 Python 打包问题、编译器/工具链问题，还是后端探测问题。

- **EN:** For custom kernel development, it is usually faster to reason from CMake target definitions down to the exact `csrc` files.
- **CN:** 对于自定义内核开发，通常从 CMake 目标定义一路追到具体 `csrc` 文件会更高效。

- **EN:** Because the build spans Python and native code, version mismatches in torch, CUDA, ROCm, or Python itself can surface in non-obvious ways.
- **CN:** 由于构建横跨 Python 与原生代码，torch、CUDA、ROCm 甚至 Python 版本不匹配都可能以不直观的方式暴露出来。

- **EN:** Treat the build system as part of the runtime architecture: it decides what kernels and extensions exist at all.
- **CN:** 应把构建系统视为运行时架构的一部分：它决定了到底有哪些内核和扩展会存在。

**EN:** The build system is how vLLM turns architecture ideas into importable, hardware-specific execution artifacts.
**CN:** 构建系统就是 vLLM 把架构思想转化为可导入、硬件特定执行产物的过程。

## Appendix A. Request Lifecycle Walkthrough / 附录 A：请求生命周期导览

- **EN:** A typical text request starts as an HTTP payload or Python method call.
- **CN:** 一个典型的文本请求，最初表现为 HTTP 负载或 Python 方法调用。

- **EN:** The frontend parses request fields, sampling parameters, chat messages, and optional multimodal attachments.
- **CN:** 前端会解析请求字段、采样参数、聊天消息以及可选的多模态附件。

- **EN:** Chat messages are rendered into a raw prompt using a chat template if the task is chat-based.
- **CN:** 如果任务是聊天型，聊天消息会通过 chat template 渲染成原始 prompt。

- **EN:** Tokenization and multimodal preprocessing happen before the engine core sees the request.
- **CN:** 在引擎核心真正接收请求前，会先完成分词与多模态预处理。

- **EN:** The engine client serializes the request into an engine-core request format and sends it to the correct core process.
- **CN:** 引擎客户端会把请求序列化为引擎核心请求格式，并发送到目标核心进程。

- **EN:** The scheduler admits the request into the waiting queue and evaluates token budget plus KV availability.
- **CN:** 调度器会把请求加入 waiting 队列，并评估 token 预算与 KV 可用性。

- **EN:** If prefix caching is enabled, the KV cache manager tries to find reusable full blocks first.
- **CN:** 如果启用了前缀缓存，KV 缓存管理器会先尝试寻找可复用的完整块。

- **EN:** The scheduler selects a set of requests and token counts for the next execution step.
- **CN:** 调度器会为下一执行步选择一组请求及其对应 token 数。

- **EN:** Workers prepare input tensors, block tables, and any multimodal or LoRA metadata required by the model runner.
- **CN:** 工作进程会准备输入张量、块表，以及模型运行器所需的多模态或 LoRA 元数据。

- **EN:** The model runner launches kernels or compiled graphs to execute the forward pass on GPU.
- **CN:** 模型运行器会启动内核或编译图，在 GPU 上执行前向计算。

- **EN:** If speculative decoding is enabled, proposal and verification logic is woven into that step.
- **CN:** 如果启用了投机解码，那么提议与验证逻辑会嵌入这个执行步骤中。

- **EN:** The output processor detokenizes or structures the result and streams partial outputs if needed.
- **CN:** 输出处理器会完成反分词或结构化结果整理，并在需要时流式返回部分输出。

- **EN:** Completed requests release or downgrade their KV blocks, which then re-enter free queues or cache maps.
- **CN:** 完成的请求会释放或降级其 KV 块，这些块随后会重新进入空闲队列或缓存映射。

- **EN:** Metrics, logs, and traces are emitted across frontend, engine, and worker boundaries during this lifecycle.
- **CN:** 在整个生命周期中，指标、日志与追踪信息会跨越前端、引擎与工作进程边界持续产出。

- **EN:** The same lifecycle becomes more complex, not different in kind, for multimodal, ASR, LoRA, or disaggregated-serving requests.
- **CN:** 对于多模态、ASR、LoRA 或解耦式服务请求，这一生命周期会变得更复杂，但本质并没有改变。

## Appendix B. Glossary / 附录 B：术语表

- **EN:** Block: a fixed-size unit of KV cache storage, not a GPU thread block.
- **CN:** Block：KV 缓存中的固定大小存储单元，不是 GPU 线程块。

- **EN:** Block table: the mapping from logical token positions to physical KV blocks.
- **CN:** Block table：从逻辑 token 位置映射到物理 KV 块的表。

- **EN:** Prefill: the phase that computes activations/KV for the existing prompt context.
- **CN:** Prefill：为已有 prompt 上下文计算激活与 KV 的阶段。

- **EN:** Decode: the iterative phase that generates new tokens one or a few at a time.
- **CN:** Decode：一次生成一个或少量新 token 的迭代阶段。

- **EN:** TTFT: time to first token.
- **CN:** TTFT：首 token 时间。

- **EN:** ITL: inter-token latency.
- **CN:** ITL：逐 token 间延迟。

- **EN:** Continuous batching: updating the active batch every step instead of using fixed batches.
- **CN:** Continuous batching：每一步都更新活跃 batch，而不是使用固定批次。

- **EN:** Chunked prefill: splitting large prompts into smaller schedulable chunks.
- **CN:** Chunked prefill：把大 prompt 切分为更小、可调度的块。

- **EN:** Prefix caching: reusing previously computed KV blocks for shared prompt prefixes.
- **CN:** Prefix caching：对共享 prompt 前缀复用已计算的 KV 块。

- **EN:** PagedAttention: attention over a paged/block-structured KV cache.
- **CN:** PagedAttention：在分页/分块结构 KV 缓存上的注意力计算。

- **EN:** KV cache: the stored keys and values needed for autoregressive attention.
- **CN:** KV cache：自回归注意力所需的 key/value 存储。

- **EN:** KV connector: a module that moves KV blocks across memory domains or instances.
- **CN:** KV connector：在不同内存域或实例之间移动 KV 块的模块。

- **EN:** DP: data parallelism.
- **CN:** DP：数据并行。

- **EN:** TP: tensor parallelism.
- **CN:** TP：张量并行。

- **EN:** PP: pipeline parallelism.
- **CN:** PP：流水线并行。

- **EN:** EP: expert parallelism.
- **CN:** EP：专家并行。

- **EN:** MoE: mixture of experts.
- **CN:** MoE：混合专家模型。

- **EN:** LoRA: low-rank adaptation weights served on top of a base model.
- **CN:** LoRA：叠加在基座模型上的低秩适配权重。

- **EN:** Structured output: decoding constrained by schema, regex, grammar, or enumerated choices.
- **CN:** Structured output：受 schema、正则、语法或枚举约束的解码。

- **EN:** Tool parser: a parser that extracts tool calls from model output in auto mode.
- **CN:** Tool parser：在 auto 模式下从模型输出中抽取工具调用的解析器。

- **EN:** CUDAGraph: a captured GPU execution graph replayed to reduce launch overhead.
- **CN:** CUDAGraph：被捕获并可重放的 GPU 执行图，用于降低 launch 开销。

- **EN:** Piecewise CUDA graph: graph capture applied to selected subgraphs rather than the whole model.
- **CN:** Piecewise CUDA graph：对选定子图而非整个模型进行的图捕获。

- **EN:** Batch descriptor: the runtime key used by V1 CUDA-graph dispatch logic.
- **CN:** Batch descriptor：V1 CUDA 图分发逻辑使用的运行时键。

- **EN:** Compile cache: on-disk storage of graph-transformation and kernel-compilation artifacts.
- **CN:** Compile cache：图变换与内核编译产物的磁盘缓存。

- **EN:** Dynamic shapes: compile-time handling for inputs whose sizes vary from request to request.
- **CN:** Dynamic shapes：对请求间尺寸变化输入的编译期处理方式。

- **EN:** AWQ: activation-aware weight quantization.
- **CN:** AWQ：激活感知权重量化。

- **EN:** GPTQ: post-training quantization based on approximate second-order error control.
- **CN:** GPTQ：基于近似二阶误差控制的后训练量化。

- **EN:** FP8: 8-bit floating-point quantization family.
- **CN:** FP8：8 位浮点量化家族。

- **EN:** NVFP4: NVIDIA-oriented 4-bit floating-point quantization family.
- **CN:** NVFP4：面向 NVIDIA 的 4 位浮点量化家族。

- **EN:** MXFP4: mixed-exponent 4-bit floating-point quantization family used in some MoE paths.
- **CN:** MXFP4：某些 MoE 路径使用的混合指数 4 位浮点量化家族。

- **EN:** Compressed-tensors: a configurable checkpoint format and runtime scheme mapping for quantized weights and related metadata.
- **CN:** Compressed-tensors：一种可配置的检查点格式与运行时方案映射系统，用于量化权重及相关元数据。

- **EN:** Draft model: a smaller or auxiliary model that proposes speculative tokens.
- **CN:** Draft model：用于提出投机 token 的较小模型或辅助模型。

- **EN:** EAGLE: a hidden-state-based speculative drafting method.
- **CN:** EAGLE：一种基于隐藏状态的投机提议方法。

- **EN:** Medusa: a speculative method using extra heads over target hidden states.
- **CN:** Medusa：一种在目标隐藏状态上增加额外头部的投机方法。

- **EN:** N-gram speculation: a lightweight proposer based on repeated prompt substrings.
- **CN:** N-gram speculation：一种基于 prompt 重复子串的轻量 proposer 方法。

- **EN:** Dummy text: synthetic text passed to HF processors so multimodal preprocessing remains valid.
- **CN:** Dummy text：传给 HF 处理器的合成文本，用于保持多模态预处理有效。

- **EN:** Prompt update detection: logic that figures out how processors expanded multimodal placeholders.
- **CN:** Prompt update detection：用于判断处理器如何扩展多模态占位符的逻辑。

- **EN:** Punica: the kernel/metadata path used for efficient multi-LoRA serving.
- **CN:** Punica：用于高效多 LoRA 服务的内核/元数据路径。

- **EN:** All2All backend: the communication implementation used by expert parallel MoE dispatch/combine.
- **CN:** All2All backend：专家并行 MoE 分发/合并所使用的通信实现。

## Appendix C. Recommended Source Map / 附录 C：推荐源码地图

- **EN:** Read `docs/README.md` first for product-level positioning.
- **CN:** 先读 `docs/README.md` 建立产品级定位。

- **EN:** Read `docs/design/arch_overview.md` next for system structure.
- **CN:** 再读 `docs/design/arch_overview.md` 理解系统结构。

- **EN:** Read `docs/usage/v1_guide.md` to understand why V1 exists.
- **CN:** 然后读 `docs/usage/v1_guide.md` 理解为什么会有 V1。

- **EN:** Read `vllm/v1/engine/async_llm.py` to understand the async frontend.
- **CN:** 阅读 `vllm/v1/engine/async_llm.py` 理解异步前端。

- **EN:** Read `vllm/v1/engine/core_client.py` to understand process communication.
- **CN:** 阅读 `vllm/v1/engine/core_client.py` 理解进程间通信。

- **EN:** Read `vllm/v1/engine/core.py` for the engine-core main loop.
- **CN:** 阅读 `vllm/v1/engine/core.py` 理解引擎核心主循环。

- **EN:** Read `vllm/v1/core/sched/scheduler.py` for scheduling decisions.
- **CN:** 阅读 `vllm/v1/core/sched/scheduler.py` 理解调度决策。

- **EN:** Read `vllm/v1/core/kv_cache_manager.py` for cache interface logic.
- **CN:** 阅读 `vllm/v1/core/kv_cache_manager.py` 理解缓存接口逻辑。

- **EN:** Read `vllm/v1/core/block_pool.py` for block ownership and usage accounting.
- **CN:** 阅读 `vllm/v1/core/block_pool.py` 理解块所有权与使用量统计。

- **EN:** Read `docs/design/prefix_caching.md` for hash-based cache reuse.
- **CN:** 阅读 `docs/design/prefix_caching.md` 理解基于哈希的缓存复用。

- **EN:** Read `docs/design/hybrid_kv_cache_manager.md` for mixed attention types.
- **CN:** 阅读 `docs/design/hybrid_kv_cache_manager.md` 理解混合注意力类型。

- **EN:** Read `docs/design/paged_attention.md` for the historical kernel story.
- **CN:** 阅读 `docs/design/paged_attention.md` 理解历史内核故事。

- **EN:** Read `csrc/attention/attention_kernels.cuh` for current CUDA attention logic.
- **CN:** 阅读 `csrc/attention/attention_kernels.cuh` 理解当前 CUDA 注意力逻辑。

- **EN:** Read `vllm/model_executor/models/registry.py` for architecture mapping.
- **CN:** 阅读 `vllm/model_executor/models/registry.py` 理解架构映射。

- **EN:** Read `vllm/model_executor/models/interfaces.py` for capability protocols.
- **CN:** 阅读 `vllm/model_executor/models/interfaces.py` 理解能力协议。

- **EN:** Read `docs/design/mm_processing.md` for multimodal prompt handling.
- **CN:** 阅读 `docs/design/mm_processing.md` 理解多模态 prompt 处理。

- **EN:** Read `vllm/multimodal/processing/*` after that for implementation details.
- **CN:** 随后阅读 `vllm/multimodal/processing/*` 了解实现细节。

- **EN:** Read `docs/features/tool_calling.md` for model/tool parser behavior.
- **CN:** 阅读 `docs/features/tool_calling.md` 理解模型/工具解析器行为。

- **EN:** Read `docs/features/structured_outputs.md` for constrained decoding patterns.
- **CN:** 阅读 `docs/features/structured_outputs.md` 理解约束解码模式。

- **EN:** Read `docs/serving/openai_compatible_server.md` for API details.
- **CN:** 阅读 `docs/serving/openai_compatible_server.md` 获取 API 细节。

- **EN:** Read `vllm/entrypoints/openai/api_server.py` to see router assembly.
- **CN:** 阅读 `vllm/entrypoints/openai/api_server.py` 理解路由组装。

- **EN:** Read `vllm/entrypoints/anthropic/*` to see Messages API compatibility.
- **CN:** 阅读 `vllm/entrypoints/anthropic/*` 理解 Messages API 兼容层。

- **EN:** Read `vllm/entrypoints/grpc_server.py` for the RPC path.
- **CN:** 阅读 `vllm/entrypoints/grpc_server.py` 理解 RPC 路径。

- **EN:** Read `docs/features/lora.md` before reading LoRA internals.
- **CN:** 在阅读 LoRA 内部实现前，先读 `docs/features/lora.md`。

- **EN:** Read `vllm/lora/worker_manager.py` for runtime LoRA lifecycle.
- **CN:** 阅读 `vllm/lora/worker_manager.py` 理解运行时 LoRA 生命周期。

- **EN:** Read `docs/features/speculative_decoding/README.md` for method overview.
- **CN:** 阅读 `docs/features/speculative_decoding/README.md` 理解方法总览。

- **EN:** Read `vllm/v1/spec_decode/draft_model.py` and `ngram_proposer.py` for two opposite proposer styles.
- **CN:** 阅读 `vllm/v1/spec_decode/draft_model.py` 与 `ngram_proposer.py` 体会两种相反风格的 proposer。

- **EN:** Read `docs/features/quantization/README.md` before diving into config classes.
- **CN:** 在深入量化配置类前，先读 `docs/features/quantization/README.md`。

- **EN:** Read `vllm/model_executor/layers/quantization/__init__.py` and `base_config.py` for the quantization registry model.
- **CN:** 阅读 `vllm/model_executor/layers/quantization/__init__.py` 与 `base_config.py` 理解量化注册模型。

- **EN:** Read `fp8.py`, `awq.py`, and `auto_gptq.py` as representative quantization implementations.
- **CN:** 把 `fp8.py`、`awq.py` 与 `auto_gptq.py` 当作代表性量化实现来阅读。

- **EN:** Read `compressed_tensors/compressed_tensors.py` for the most flexible scheme-mapping path.
- **CN:** 阅读 `compressed_tensors/compressed_tensors.py` 理解最灵活的方案映射路径。

- **EN:** Read `docs/design/fused_moe_modular_kernel.md` before inspecting MoE code.
- **CN:** 在查看 MoE 代码前，先读 `docs/design/fused_moe_modular_kernel.md`。

- **EN:** Read `docs/design/moe_kernel_features.md` to understand backend compatibility.
- **CN:** 阅读 `docs/design/moe_kernel_features.md` 理解后端兼容性矩阵。

- **EN:** Read `csrc/moe/torch_bindings.cpp` to see which MoE ops are native.
- **CN:** 阅读 `csrc/moe/torch_bindings.cpp` 查看哪些 MoE 操作是原生实现。

- **EN:** Read `docs/design/torch_compile.md` and `docs/design/cuda_graphs.md` together.
- **CN:** 把 `docs/design/torch_compile.md` 与 `docs/design/cuda_graphs.md` 配套阅读。

- **EN:** Read `vllm/v1/cudagraph_dispatcher.py` for runtime graph-selection logic.
- **CN:** 阅读 `vllm/v1/cudagraph_dispatcher.py` 理解运行时图选择逻辑。

- **EN:** Read `vllm/compilation/cuda_graph.py` for wrapper-level capture/replay behavior.
- **CN:** 阅读 `vllm/compilation/cuda_graph.py` 理解包装器级别的捕获/重放行为。

- **EN:** Read `docs/features/disagg_prefill.md` before connector code.
- **CN:** 在阅读连接器代码前，先读 `docs/features/disagg_prefill.md`。

- **EN:** Read `docs/features/nixl_connector_usage.md` or `mooncake_connector_usage.md` depending on transport interest.
- **CN:** 根据你的传输兴趣，阅读 `docs/features/nixl_connector_usage.md` 或 `mooncake_connector_usage.md`。

- **EN:** Read `vllm/distributed/kv_transfer/*` for the actual distributed-KV implementation tree.
- **CN:** 阅读 `vllm/distributed/kv_transfer/*` 理解实际的分布式 KV 实现树。

- **EN:** Read `docs/serving/parallelism_scaling.md` for deployment decisions.
- **CN:** 阅读 `docs/serving/parallelism_scaling.md` 进行部署决策。

- **EN:** Read `vllm/config/parallel.py` to see every distributed knob in one place.
- **CN:** 阅读 `vllm/config/parallel.py` 可以在一个地方看到所有分布式旋钮。

- **EN:** Read `vllm/distributed/parallel_state.py` when debugging collectives.
- **CN:** 调试集合通信时，请阅读 `vllm/distributed/parallel_state.py`。

- **EN:** Read `vllm/entrypoints/speech_to_text/base/serving.py` for ASR request flow.
- **CN:** 阅读 `vllm/entrypoints/speech_to_text/base/serving.py` 理解 ASR 请求流。

- **EN:** Read `pyproject.toml`, `setup.py`, and `CMakeLists.txt` together for build understanding.
- **CN:** 如需理解构建系统，请把 `pyproject.toml`、`setup.py` 与 `CMakeLists.txt` 一起阅读。

## Appendix D. Suggested Study Roadmaps / 附录 D：建议学习路径

- **EN:** If you are an operator, read sections 1, 2, 4, 5, 8, 15, 19, and 20 in that order.
- **CN:** 如果你是运维/平台工程师，建议按 1、2、4、5、8、15、19、20 的顺序阅读。

- **EN:** If you are a kernel engineer, read sections 3, 11, 12, 17, 18, and 23 first.
- **CN:** 如果你是内核工程师，建议优先阅读 3、11、12、17、18、23 章。

- **EN:** If you are adding a new model, read sections 2, 6, 7, 14, 16, and 23 first.
- **CN:** 如果你要接入一个新模型，建议优先阅读 2、6、7、14、16、23 章。

- **EN:** If you are building agents, read sections 15, 16, 21, and 22 first.
- **CN:** 如果你在构建智能体系统，建议优先阅读 15、16、21、22 章。

- **EN:** If you are chasing latency, read sections 4, 5, 10, 17, 18, 19, and 20 first.
- **CN:** 如果你主要追求低延迟，建议优先阅读 4、5、10、17、18、19、20 章。

- **EN:** If you are chasing model density, read sections 8, 9, 11, 12, and 20 first.
- **CN:** 如果你主要追求高模型密度，建议优先阅读 8、9、11、12、20 章。

- **EN:** If you feel lost in the code, anchor yourself in the V1 engine and scheduler before reading specialized subsystems.
- **CN:** 如果你在代码中感到迷失，请先回到 V1 引擎与调度器，再去阅读专门子系统。

- **EN:** If you need to debug correctness, reduce complexity: disable compile, disable advanced graphs, and use the smallest distributed shape that reproduces the bug.
- **CN:** 如果你需要调试正确性问题，请先降低复杂度：关闭 compile、关闭高级图执行，并使用能够复现问题的最小分布式形状。

- **EN:** If you need to debug performance, capture both frontend and backend signals: preprocessing time, GPU time, cache hit rate, and transport overhead.
- **CN:** 如果你需要调试性能，请同时采集前后端信号：预处理时间、GPU 时间、缓存命中率以及传输开销。

- **EN:** Use docs to get intent, then code to verify behavior, then benchmarks to validate your assumptions.
- **CN:** 建议先用文档理解意图，再用代码验证行为，最后通过基准测试验证自己的假设。

- **EN:** Read tests earlier than you think; they often encode invariants more clearly than the implementation files.
- **CN:** 测试文件往往比实现文件更清楚地表达不变量，因此应比你想象得更早去读测试。

- **EN:** Treat vLLM as a systems codebase: abstractions, queueing, memory, and transport are as important as neural-network math.
- **CN:** 应把 vLLM 视为系统代码库：抽象、队列、内存与传输和神经网络数学一样重要。

## Appendix E. Quick Production Checklist / 附录 E：生产环境快速检查清单

- **EN:** Confirm the model family, task type, and chat template are all mutually compatible.
- **CN:** 确认模型家族、任务类型与 chat template 三者相互兼容。

- **EN:** Check startup logs for KV cache size, maximum concurrency, and actual parallelism values.
- **CN:** 检查启动日志中的 KV 缓存大小、最大并发度以及实际并行参数。

- **EN:** Verify whether prefix caching is enabled and whether your prompt shapes allow it to work.
- **CN:** 确认是否启用了前缀缓存，以及你的 prompt 形状是否允许它真正发挥作用。

- **EN:** Confirm whether quantization loaded as expected, not just whether you passed the flag.
- **CN:** 确认量化是否按预期加载，而不是只确认你传了 flag。

- **EN:** Verify actual attention backend and CUDA graph mode in logs.
- **CN:** 在日志中确认实际启用的注意力后端与 CUDA 图模式。

- **EN:** Measure TTFT, ITL, throughput, and error rate separately.
- **CN:** 分别测量 TTFT、ITL、吞吐与错误率。

- **EN:** Profile frontend media/tokenization work for multimodal or ASR deployments.
- **CN:** 对于多模态或 ASR 部署，要分析前端媒体/分词处理开销。

- **EN:** Set security boundaries around runtime LoRA loading, tool servers, and model-management endpoints.
- **CN:** 为运行时 LoRA 加载、工具服务器以及模型管理端点设置清晰的安全边界。

- **EN:** For multi-node deployments, validate transport settings and network path before tuning kernels.
- **CN:** 对多节点部署，应先验证传输设置与网络路径，再去调内核。

- **EN:** For TP-heavy setups, confirm NCCL or RDMA is actually using the expected fast path.
- **CN:** 对于 TP 较重的部署，要确认 NCCL 或 RDMA 真实走在预期的高速路径上。

- **EN:** For PP-heavy setups, watch stage imbalance and pipeline bubbles.
- **CN:** 对于 PP 较重的部署，要关注阶段不均衡与流水线气泡。

- **EN:** For MoE deployments, validate expert backend, all2all backend, and load-balance strategy together.
- **CN:** 对于 MoE 部署，要同时验证专家内核后端、all2all 后端以及负载均衡策略。

- **EN:** For tool calling, test the exact parser/template/model combination you plan to ship.
- **CN:** 对于工具调用，要测试你准备上线的 parser/template/model 精确组合。

- **EN:** For ASR, enforce file-size limits and validate chunk-merge quality on long audio.
- **CN:** 对于 ASR，要设置文件大小限制，并验证长音频下的分块合并质量。

- **EN:** Keep a benchmark script and a representative traffic replay so future regressions can be detected quickly.
- **CN:** 保留一份基准脚本和一份代表性流量回放，以便快速发现未来回归。

## Closing Note / 结束语

**EN:** vLLM is best understood as an inference operating system: it schedules tokens, allocates pages, chooses kernels, manages transport, and exposes product-facing APIs on top.
**CN:** 最好把 vLLM 理解为一个“推理操作系统”：它负责调度 token、分配页面、选择内核、管理传输，并在其上暴露面向产品的 API。

**EN:** Once you understand that systems perspective, the rest of the repository becomes much easier to navigate.
**CN:** 一旦你建立了这种系统视角，整个仓库的其余部分就会更容易导航。

**EN:** Use this tutorial as a map, then verify every important idea in the source files referenced above.
**CN:** 请把本教程当作地图，再回到上文引用的源码文件中验证每一个重要观点。

