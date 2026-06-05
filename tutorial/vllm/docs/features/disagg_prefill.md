# disagg_prefill.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/disagg_prefill.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document introduces vLLM's *disaggregated prefilling* feature (experimental), where prefill and decode phases of LLM inference run in separate vLLM instances. It explains motivation, supported connectors, key abstractions, workflow diagrams, and development guidance for third-party connector authors.

**CN:** 本文档介绍 vLLM 的*解耦预填充*功能（实验性），将 LLM 推理的预填充阶段和解码阶段分别运行在独立的 vLLM 实例中，内容包含动机、支持的连接器、关键抽象、工作流示意图及第三方连接器开发者指南。

### Motivation / 动机

**EN:** Two reasons: (1) Independent tuning of TTFT and ITL by assigning different parallelism strategies to prefill/decode instances; (2) Control of tail ITL — without disaggregation, prefill jobs interleave with decode, inflating tail latency. Note: does NOT improve throughput.

**CN:** 两个原因：(1) 通过为预填充/解码实例分配不同并行策略，独立调优 TTFT 和 ITL；(2) 控制尾部 ITL——不解耦时，预填充任务与解码交错进行，拉高尾部延迟。注意：不提高吞吐量。

### Supported Connectors / 支持的连接器

**EN:** ExampleConnector, LMCacheConnectorV1, NixlConnector (fully async), P2pNcclConnector, MooncakeConnector, MultiConnector, OffloadingConnector, FlexKVConnectorV1.

**CN:** ExampleConnector、LMCacheConnectorV1、NixlConnector（完全异步）、P2pNcclConnector、MooncakeConnector、MultiConnector、OffloadingConnector、FlexKVConnectorV1。

### Key Abstractions / 关键抽象

**EN:** Three building blocks: **Connector** (retrieves KV caches from producer to consumer), **LookupBuffer** (SQL-like `insert`/`drop_select` API for KV cache storage, where `insert` is non-blocking and `drop_select` is blocking), **Pipe** (single-direction FIFO for tensor transmission via `send_tensor`/`recv_tensor`).

**CN:** 三个核心组件：**Connector**（从生产者向消费者检索 KV 缓存）、**LookupBuffer**（类 SQL 的 `insert`/`drop_select` KV 缓存存储接口，`insert` 非阻塞，`drop_select` 阻塞）、**Pipe**（通过 `send_tensor`/`recv_tensor` 进行单向 FIFO 张量传输）。

### Connector Roles / 连接器角色

**EN:** Each vLLM process has a **Scheduler connector** (co-located with the scheduler; schedules KV transfers) and one or more **Worker connectors** (co-located with workers; execute layer-by-layer KV store/load).

**CN:** 每个 vLLM 进程包含一个**调度器连接器**（与调度器同进程，负责调度 KV 传输）和一个或多个 **Worker 连接器**（与 Worker 同进程，执行逐层 KV 存储/加载）。

### Third-Party Integration / 第三方集成

**EN:** Three recommended approaches: fully-customised Connector, database-like LookupBuffer, or distributed P2P Pipe. The vLLM team actively reviews and merges third-party connector PRs.

**CN:** 推荐三种集成方式：完全自定义 Connector、类数据库 LookupBuffer，或分布式 P2P Pipe。vLLM 团队积极审核并合并第三方连接器 PR。

## Key Concepts / 关键概念

- **EN:** Prefill instance / Decode instance — two independent vLLM servers; prefill produces KV caches, decode consumes them. **CN:** 预填充实例/解码实例——两个独立的 vLLM 服务，预填充产生 KV 缓存，解码消费它们。
- **EN:** TTFT vs ITL — disaggregated prefilling lets operators tune time-to-first-token and inter-token latency independently. **CN:** TTFT 与 ITL——解耦预填充允许运营者独立调优首 token 时延和 token 间时延。
- **EN:** `drop_select` blocking — the decode instance blocks until matching KV caches arrive from the prefill instance. **CN:** `drop_select` 阻塞——解码实例阻塞等待预填充实例的匹配 KV 缓存到达。
- **EN:** MultiConnector — composes multiple connectors (e.g., NixlConnector + ExampleConnector) in a single deployment. **CN:** MultiConnector——在单次部署中组合多个连接器（如 NixlConnector + ExampleConnector）。
- **EN:** OffloadingConnector — offloads KV data to CPU memory with configurable block size and total byte budget. **CN:** OffloadingConnector——将 KV 数据卸载到 CPU 内存，支持配置块大小和总字节预算。
