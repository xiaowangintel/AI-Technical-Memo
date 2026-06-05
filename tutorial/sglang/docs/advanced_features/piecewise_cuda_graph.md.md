# piecewise_cuda_graph.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/piecewise_cuda_graph.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Motivation Standard CUDA graphs capture the entire model forward pass as a single graph. This works well for decode (fixed batch size), but not for extend/prefill where the number of tokens varies across iterations. / 该文档围绕 Piecewise CUDA Graph 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Motivation
**EN:** Standard CUDA graphs capture the entire model forward pass as a single graph. This works well for decode (fixed batch size), but not for extend/prefill where the number of tokens varies across iterations.
**CN:** 本节围绕 Motivation 展开，概述了 PCG, CUDA, graph, MoE 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage
**EN:** PCG is enabled by default for supported configurations. No extra flags needed: ``bash python3 -m sglang.launch_server \ --model-path meta-llama/Llama-3.1-8B-Instruct ``
**CN:** 本节围绕 Usage 展开，概述了 PCG, extra, flags, needed 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Disable PCG
**EN:** ``bash python3 -m sglang.launch_server \ --model-path meta-llama/Llama-3.1-8B-Instruct \ --disable-piecewise-cuda-graph ``
**CN:** 本节围绕 Disable PCG 展开，概述了 python3, model-path, --model-path, sglang.launch_server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Custom capture sizes
**EN:** ``bash python3 -m sglang.launch_server \ --model-path meta-llama/Llama-3.1-8B-Instruct \ --piecewise-cuda-graph-max-tokens 2048 ``
**CN:** 本节围绕 Custom capture sizes 展开，概述了 python3, model-path, --model-path, sglang.launch_server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Server Args
**EN:** This section provides a comparison table for Server Args, covering columns such as Argument, Default, Description and examples such as --disable-piecewise-cuda-graph, --enforce-piecewise-cuda-graph, --piecewise-cuda-graph-max-tokens, --piecewise-cuda-graph-tokens.
**CN:** 本节围绕 服务端 Args 展开，概述了 PCG, --enforce-piecewise-cuda-graph, MLA, eager 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Bug Report
**EN:** PCG is enabled by default but is still in an experimental stage. Since PCG relies on torch.compile to trace the model's forward pass, most bugs are introduced by torch compile tracing failures (e.g., untraceable ops, dynamic control flow, or graph breaks).
**CN:** 本节围绕 Bug Report 展开，概述了 PCG, torch.compile, --disable-piecewise-cuda-graph, but 等要点，并说明相关配置、流程、示例或限制条件。

### Section: For Users
**EN:** If you see an error message like the following during server startup, it is a PCG bug: `` Piecewise CUDA Graph is enabled by default as an experimental feature. To work around this error, add --disable-piecewise-cuda-graph to your launch command.
**CN:** 本节围绕 For Users 展开，概述了 error, command, PCG, GPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: For Developers
**EN:** Since PCG relies on torch.compile to trace the model's forward pass, newly developed CUDA kernels (both JIT kernels and sgl-kernels) are typically not compatible with torch.compile out of the box.
**CN:** 本节围绕 For Developers 展开，概述了 JIT, Tensor, torch.Tensor, torch.compile 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Torch compile backend
**EN:** PCG uses torch.compile with a custom backend (SGLangBackend) to split and compile the model's forward pass. When is_in_piecewise_cuda_graph() returns True, the wrapper dispatches to the compiled callable; otherwise it falls back to the original forward.
**CN:** 本节围绕 Torch compile backend 展开，概述了 CUDAPiecewiseBackend, graph, SGLangBackend, split_gm 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Piecewise cuda graph runner
**EN:** PiecewiseCudaGraphRunner orchestrates the full lifecycle through three phases: - **Compile** — Warms up JIT kernels with a dummy forward pass, then wraps the model with torch.compile, triggering Dynamo tracing to split the FX graph and create CUDAPiecewiseBackend instances for each subgraph piece.
**CN:** 本节围绕 Piecewise cuda graph runner 展开，概述了 CUDA, CUDAPiecewiseBackend, PiecewiseCudaGraphRunner, JIT 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Memory optimization
**EN:** The memory cost of PCG comes from two parts: **torch memory allocator** and **non-torch memory**.
**CN:** 本节围绕 Memory optimization 展开，概述了 memory, CUDA, overhead, PCG 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Shape configuration
**EN:** This section provides a comparison table for Shape configuration, covering columns such as Token range, Step size and examples such as 4 – 32, 48 – 256, 288 – 512, 576 – 1024.
**CN:** 本节围绕 Shape 配置 展开，概述了 MLA, size, token, models 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Compatibility
**EN:** PCG is auto-disabled in the following scenarios. We are actively working on expanding compatibility — support for many of these will be coming soon.
**CN:** 本节围绕 Compatibility 展开，概述了 DeepseekV32ForCausalLM, PCG, VLM, LLM 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Code Reference
**EN:** This section provides a comparison table for Code Reference, covering columns such as File, Description and examples such as python/sglang/srt/model_executor/piecewise_cuda_graph_runner.py, python/sglang/srt/compilation/compile.py, python/sglang/srt/compilation/backend.py, python/sglang/srt/compilation/cuda_piecewise_backend.py.
**CN:** 本节围绕 Code Reference 展开，概述了  | , Main, graph, Global 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** PCG / **CN:** PCG
- **EN:** CUDA / **CN:** CUDA
- **EN:** graph / **CN:** graph
- **EN:** torch.compile / **CN:** torch.compile
- **EN:** CUDAPiecewiseBackend / **CN:** CUDAPiecewiseBackend
- **EN:** capture / **CN:** capture
- **EN:** memory / **CN:** memory
- **EN:** SGLangBackend / **CN:** SGLangBackend

## Dependencies / 依赖关系
- `python/sglang/srt/utils/custom_op.py`
- `python/sglang/srt/model_executor/piecewise_cuda_graph_runner.py`
- `python/sglang/srt/compilation/compile.py`
- `python/sglang/srt/compilation/backend.py`
- `python/sglang/srt/compilation/cuda_piecewise_backend.py`
- `python/sglang/srt/compilation/piecewise_context_manager.py`
- `python/sglang/srt/compilation/compilation_config.py`
- `python/sglang/srt/server_args.py`
