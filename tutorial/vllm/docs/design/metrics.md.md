# metrics.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/metrics.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** The document frames metrics as a V1 observability design, with Prometheus as the primary production surface and periodic info-log reporting as a lighter debugging path.  
  **CN:** 本文将指标体系定位为 V1 的可观测性设计，强调 Prometheus 是生产环境的主要暴露方式，而周期性 info 日志则用于更轻量的调试与排查。
- **EN:** It separates server-level gauges/counters from request-level histograms, and explicitly treats server metrics as the context needed to explain SLO-style request metrics such as TTFT, TPOT, queue time, and end-to-end latency.  
  **CN:** 文档把指标分成服务级 Gauge/Counter 与请求级 Histogram，并明确指出服务级指标是解释 TTFT、TPOT、排队时间和端到端延迟等 SLO 指标的上下文。
- **EN:** A core design choice is to keep overhead away from the EngineCore hot path by collecting scheduler stats and request event timestamps in the engine core, then computing most intervals in the frontend/API process from `EngineCoreOutputs`.  
  **CN:** 核心设计选择是尽量把开销移出 EngineCore 热路径：在引擎核心记录调度统计和请求事件时间戳，再由前端/API 进程基于 `EngineCoreOutputs` 计算大多数时间区间。
- **EN:** The doc also covers KV-cache residency sampling, Prometheus/logging publishers, multiprocessing caveats, special metrics like cache config and LoRA info, deprecated metrics, and future extensions such as parallel sampling, speculative decoding, autoscaling, and tracing.  
  **CN:** 文档还覆盖了 KV 缓存驻留采样、Prometheus/日志发布器、多进程限制、缓存配置与 LoRA 等特殊指标，以及废弃指标和并行采样、推测解码、自动扩缩容、链路追踪等后续扩展方向。

## Key Concepts / 关键概念
- **EN:** **Server-level vs request-level metrics** — global engine health metrics complement per-request latency/size histograms.  
  **CN:** **服务级与请求级指标** —— 全局引擎健康度指标用于补充单请求延迟/长度直方图。
- **EN:** **EngineCore event timeline** — `QUEUED`, `SCHEDULED`, `PREEMPTED`, and `NEW_TOKENS` are the basis for queue, prefill, decode, inference, and inter-token intervals.  
  **CN:** **EngineCore 事件时间线** —— `QUEUED`、`SCHEDULED`、`PREEMPTED` 与 `NEW_TOKENS` 构成排队、prefill、decode、推理和 token 间隔等指标的计算基础。
- **EN:** **Monotonic timestamp discipline** — interval calculations rely on monotonic clocks from the same process to avoid clock-shift errors and cross-process inconsistencies.  
  **CN:** **单调时钟约束** —— 时间区间必须使用同一进程内的单调时钟计算，以避免系统时钟漂移和跨进程时基不一致。
- **EN:** **KV cache residency metrics** — sampled block lifetime, idle-before-eviction, and reuse-gap histograms help diagnose stranded or long-lived cache usage.  
  **CN:** **KV 缓存驻留指标** —— 采样块的生命周期、驱逐前空闲时间和复用间隔直方图可用于识别滞留或超长驻留的缓存。
- **EN:** **Metric lifecycle management** — the document emphasizes careful naming, deprecation notices, downstream compatibility, and skepticism about adding metrics with non-trivial overhead.  
  **CN:** **指标生命周期管理** —— 文档强调命名规范、废弃提示、下游兼容性，以及对新增高开销指标保持谨慎。
