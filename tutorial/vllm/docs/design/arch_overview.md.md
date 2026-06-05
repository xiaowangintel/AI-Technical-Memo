# arch_overview.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/arch_overview.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This document maps vLLM from user-facing entrypoints (`LLM` and `vllm serve`) down to engine, worker, model runner, and model internals. **CN:** 本文从用户入口（`LLM` 与 `vllm serve`）一路梳理到 engine、worker、model runner 和 model 等内部结构。
- **EN:** It explains the V1 multi-process serving architecture, separating API server, engine core, GPU workers, and an optional DP coordinator to improve throughput and resource isolation. **CN:** 文档解释了 V1 的多进程服务架构，将 API server、engine core、GPU worker 以及可选的 DP coordinator 分离，以提升吞吐并隔离职责。
- **EN:** The process-count section is operationally useful because it ties deployment shape (`TP`, `DP`, API server count, GPU count) to actual process totals. **CN:** 进程数量汇总非常偏运维，明确把 `TP`、`DP`、API server 数量和 GPU 数量映射到最终进程总数。
- **EN:** The `LLMEngine`/`AsyncLLMEngine` section highlights the core request lifecycle: input processing, scheduling, model execution, and output processing, with async serving built on top. **CN:** `LLMEngine`/`AsyncLLMEngine` 部分强调了核心请求生命周期：输入处理、调度、模型执行和输出处理，其中在线服务基于异步封装实现。
- **EN:** The class-hierarchy discussion focuses on three design choices: shared `VllmConfig`, uniform model constructors, and doing sharding/quantization during initialization for large-model scalability. **CN:** 类层次结构部分重点说明三项设计取舍：共享 `VllmConfig`、统一模型构造函数，以及在初始化阶段完成分片/量化以支持超大模型。

## Key Concepts / 关键概念
- **EN:** **Entrypoints** — `LLM` targets offline inference, while `vllm serve` exposes an OpenAI-compatible server path. **CN:** **入口**——`LLM` 面向离线推理，`vllm serve` 提供兼容 OpenAI 的服务入口。
- **EN:** **V1 process architecture** — API processes talk to engine cores over ZMQ, and engine cores coordinate GPU workers per parallel rank. **CN:** **V1 进程架构**——API 进程通过 ZMQ 与 engine core 通信，而 engine core 按并行 rank 协调 GPU worker。
- **EN:** **Engine abstractions** — `LLMEngine` is the synchronous core, and `AsyncLLMEngine` adds background processing for concurrent online workloads. **CN:** **Engine 抽象**——`LLMEngine` 是同步核心，`AsyncLLMEngine` 在其上增加后台循环以支持并发在线请求。
- **EN:** **Worker stack** — each worker owns a model runner, and each model runner owns the actual `torch.nn.Module`. **CN:** **Worker 栈**——每个 worker 持有一个 model runner，而每个 model runner 再持有真实的 `torch.nn.Module`。
- **EN:** **`VllmConfig` as shared state** — a deep class hierarchy stays extensible by passing one comprehensive config object throughout the system. **CN:** **`VllmConfig` 作为共享状态**——通过在系统中传递统一配置对象，深层类层次仍能保持可扩展性。
- **EN:** **Initialization-time transformation** — sharding and quantization occur during model construction to avoid loading full weights on every device. **CN:** **初始化期变换**——分片与量化在模型构造阶段完成，避免每个设备都先加载完整权重。
