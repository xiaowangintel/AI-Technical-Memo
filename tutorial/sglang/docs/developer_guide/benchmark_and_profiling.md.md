# benchmark_and_profiling.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/developer_guide/benchmark_and_profiling.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Benchmark SGLang provides four benchmark tools that operate at different levels of the stack. The table below summarizes their key differences: Use bench_serving by default unless there are specific needs. / 该文档围绕 Benchmark and Profiling 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Benchmark
**EN:** This section provides a comparison table for Benchmark, covering columns such as Tool, HTTP Server, Scheduler and examples such as bench_serving, bench_one_batch_server, bench_offline_throughput, bench_one_batch.
**CN:** 本节围绕 Benchmark 展开，概述了 server, Yes, ITL, Engine 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Profile with PyTorch Profiler
**EN:** Pytorch Profiler is a convenient basic tool to inspect kernel execution time, call stack, and kernel overlap and occupancy.
**CN:** 本节围绕 Profile with PyTorch Profiler 展开，概述了 kernel, tool, time, call 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Profile a server with sglang.bench_serving
**EN:** If you call /start_profile directly and do not provide output_dir, the server uses its own SGLANG_TORCH_PROFILER_DIR (fallback: /tmp).
**CN:** 本节围绕 Profile a 服务端 with sglang.bench_serving 展开，概述了 SGLANG_TORCH_PROFILER_DIR, server, client, tmp 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Profile In PD Disaggregation Mode
**EN:** When profiling in PD disaggregation mode, prefill and decode workers **must be profiled separately** due to torch profiler limitations.
**CN:** 本节围绕 Profile In PD Disaggregation Mode 展开，概述了 decode, Profile, prefill, Llama-3.1-8B-Instruct 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Profile a server with sglang.bench_offline_throughput
**EN:** This content focuses on Profile a server with sglang.bench_offline_throughput and highlights batch, profile, --batch, --profile.
**CN:** 本节围绕 Profile a 服务端 with sglang.bench_offline_throughput 展开，概述了 batch, profile, --batch, --profile 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Profile a server with sglang.profiler
**EN:** When the server is running (e.g., processing a decoding request), you can start live profiling immediately by sending a profile request to the server. You can do this by running python3 -m sglang.profiler.
**CN:** 本节围绕 Profile a 服务端 with sglang.profiler 展开，概述了 request, python3, Terminal, above 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Profile a server with HTTP API endpoints
**EN:** SGLang provides HTTP API endpoints to control profiling on a running server. This allows you to start and stop profiling programmatically, which is useful for capturing specific workload patterns.
**CN:** 本节围绕 Profile a 服务端 with HTTP API endpoints 展开，概述了 profiling, POST, num_steps, start_step 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Profiler Trace Merger for Distributed Traces
**EN:** SGLang now supports automatic merging of profiling traces from distributed setups with multiple parallelism types (TP, DP, PP, EP). This feature is particularly useful for analyzing performance across distributed runs.
**CN:** 本节围绕 Profiler Trace Merger for Distributed Traces 展开，概述了 trace, merging, profiling, Start 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Possible PyTorch bugs
**EN:** If in any cases you encounter the following error (for example, using qwen 2.5 VL): ``bash RuntimeError: !stack.empty() INTERNAL ASSERT FAILED at "/pytorch/torch/csrc/autograd/profiler_python.cpp":983, please report a bug to PyTorch.
**CN:** 本节围绕 Possible PyTorch bugs 展开，概述了 Bug, PyTorch, Profiler, with_stack 等要点，并说明相关配置、流程、示例或限制条件。

### Section: View traces
**EN:** Trace files can be loaded and visualized from: 1. https://ui.perfetto.dev/ (any browser) 2.
**CN:** 本节围绕 View traces 展开，概述了 Trace, browser, --sharegpt-output-len, open 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Profile with Nsight
**EN:** Nsight systems is an advanced tool that exposes more profiling details, such as register and shared memory usage, annotated code regions and low-level CUDA APIs and events.
**CN:** 本节围绕 Profile with Nsight 展开，概述了 nsys, apt, install, code 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Layer-wise NVTX Profiling with Nsight Systems
**EN:** SGLang provides built-in layerwise NVTX annotations that can be combined with the CUDA Profiler for detailed per-layer profiling in Nsight Systems. This is particularly useful for identifying performance bottlenecks at the layer level.
**CN:** 本节围绕 Layer-wise NVTX Profiling with Nsight Systems 展开，概述了 NVTX, Terminal, Nsight, Systems 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Other tips
**EN:** You can benchmark a model using dummy weights by only providing the config.json file. This allows for quick testing of model variants without training.
**CN:** 本节围绕 Other tips 展开，概述了 CUDA, model, dummy, benchmark 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** profiling / **CN:** profiling
- **EN:** server / **CN:** 服务端
- **EN:** profile / **CN:** profile
- **EN:** NVTX / **CN:** NVTX
- **EN:** Terminal / **CN:** Terminal
- **EN:** SGLANG_TORCH_PROFILER_DIR / **CN:** SGLANG_TORCH_PROFILER_DIR
- **EN:** POST / **CN:** POST
- **EN:** num_steps / **CN:** num_steps

## Dependencies / 依赖关系
- `./bench_serving.md`
- `../advanced_features/pd_disaggregation.md`
- `bench_one_batch.py`
- `bench_offline_throughput.py`
