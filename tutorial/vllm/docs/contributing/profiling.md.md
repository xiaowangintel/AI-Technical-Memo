# profiling.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/profiling.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** A developer-only performance guide that compares PyTorch Profiler and NVIDIA Nsight Systems, then shows how to collect traces for offline inference, server mode, and benchmark runs.
- **CN:** 本文是面向开发者的性能分析指南，对比了 PyTorch Profiler 与 NVIDIA Nsight Systems，并展示了如何在离线推理、服务模式和 benchmark 场景中采集 trace。
- **EN:** Beyond command examples, it warns about profiling overhead, recommends multiprocessing settings for `nsys`, points to continuous profiling infrastructure, and marks older vLLM cProfile helpers as deprecated.
- **CN:** 除了命令示例外，文档还提醒分析器开销问题，给出 `nsys` 的多进程建议，介绍持续性能分析基础设施，并说明旧的 vLLM cProfile 辅助函数已经弃用。

## Key Concepts / 关键概念
- **EN:** Profiler choice — Nsight is better for low-overhead performance work, while torch profiler offers richer debugging context.
- **CN:** 分析器选择 — Nsight 更适合低开销性能分析，而 torch profiler 提供更丰富的调试上下文。
- **EN:** Torch profiler config — Profiling is enabled through `--profiler-config` and can capture shapes, memory, stacks, FLOPs, and compressed traces.
- **CN:** Torch Profiler 配置 — 可以通过 `--profiler-config` 开启分析，并控制形状、内存、调用栈、FLOPs 和压缩 trace 等内容。
- **EN:** Nsight dynamic capture — Server profiling uses `nsys profile` with CUDA capture-range settings and usually benefits from `spawn`.
- **CN:** Nsight 动态捕获 — 服务端分析通常通过带 CUDA capture-range 参数的 `nsys profile` 执行，并建议使用 `spawn`。
- **EN:** Continuous profiling — A separate CI workflow continuously profiles selected models and publishes results on a public dashboard.
- **CN:** 持续性能分析 — 独立的 CI 工作流会持续分析选定模型，并把结果发布到公开看板。
- **EN:** Deprecated Python helpers — The old `vllm.utils.profiling` helpers are slated for removal in favor of standard `cProfile`.
- **CN:** 已弃用的 Python 辅助接口 — 旧的 `vllm.utils.profiling` 辅助接口将被移除，建议直接使用标准 `cProfile`。
