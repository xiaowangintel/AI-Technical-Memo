# dbo.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/dbo.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This document introduces Dual Batch Overlap (DBO) as a MoE-focused optimization that overlaps sparse all-to-all communication with nearby computation. **CN:** 本文介绍了 Dual Batch Overlap（DBO），它是一个面向 MoE 的优化机制，用来把稀疏 all-to-all 通信与邻近计算重叠执行。
- **EN:** The system works by splitting one batch into two microbatches and executing them on two CPU worker threads that ping-pong at explicit yield points inside the fused MoE kernel path. **CN:** 该系统通过把一个 batch 切成两个 microbatch，并让两个 CPU worker 线程在 fused MoE kernel 路径中的显式 yield 点之间来回切换执行。
- **EN:** The document is explicit about deployment scope: DBO currently targets DP+EP, requires DeepEP, and depends on workload-specific all-to-all backend choices. **CN:** 文档明确了部署范围：DBO 目前只面向 DP+EP 场景，需要 DeepEP，并依赖与负载类型匹配的 all-to-all 后端选择。
- **EN:** Much of the design is about control flow and synchronization—uniform DP-rank decisions for microbatching, thread orchestration in `UBatchWrapper`, and hook/yield coordination in `UBatchContext`. **CN:** 设计重点主要在控制流与同步机制，包括 DP rank 间统一决定是否切分 microbatch、`UBatchWrapper` 的线程编排，以及 `UBatchContext` 中的 hook/yield 协调。
- **EN:** It also notes a practical limitation: DBO only supports full CUDA Graphs, though replay can avoid multithreading overhead after capture. **CN:** 文档还指出了一个现实限制：DBO 仅支持 full CUDA Graphs，不过一旦图已捕获，回放时可以避免多线程与 CPU 同步开销。

## Key Concepts / 关键概念
- **EN:** **Communication-computation overlap** — the main target is hiding MoE all-to-all latency behind useful compute. **CN:** **通信与计算重叠**——核心目标是把 MoE 的 all-to-all 延迟隐藏在有效计算之后。
- **EN:** **Microbatch / ubatch split** — one batch is divided into two halves so two threads can alternate execution phases. **CN:** **Microbatch / ubatch 切分**——一个 batch 被划分为两个半批次，从而让两个线程交替推进执行阶段。
- **EN:** **`GPUModelRunner` coordination** — microbatching is only allowed when all DP ranks can apply it consistently after padding and feasibility checks. **CN:** **`GPUModelRunner` 协调**——只有在所有 DP rank 经填充与可行性检查后都能一致启用切分时，microbatching 才会真正生效。
- **EN:** **`UBatchWrapper`** — this wrapper owns thread lifecycle, sliced attention metadata, and DBO-specific CUDA Graph handling. **CN:** **`UBatchWrapper`**——该包装器负责线程生命周期、切分后的 attention metadata，以及 DBO 专属的 CUDA Graph 管理。
- **EN:** **`UBatchContext`** — synchronization happens through yield points and optional recv hooks, enabling the two threads to ping-pong deterministically. **CN:** **`UBatchContext`**——同步通过 yield 点和可选 recv hook 实现，使两个线程能以确定性的方式“乒乓切换”。
- **EN:** **Runtime knobs** — `--enable-dbo`, token thresholds, DP size, expert parallelism, and DeepEP backend selection jointly determine whether DBO can run. **CN:** **运行时开关**——`--enable-dbo`、token 阈值、DP 大小、专家并行以及 DeepEP 后端选择共同决定 DBO 是否能运行。
