# breakable_cuda_graph.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/breakable_cuda_graph.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Motivation Standard CUDA graphs capture an entire forward pass as a single, opaque graph. / 该文档围绕 Breakable CUDA Graph 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Motivation
**EN:** Standard CUDA graphs capture an entire forward pass as a single, opaque graph. This is great for performance, but creates two problems: 1.
**CN:** 本节围绕 Motivation 展开，概述了 CUDA, graph, captured, operations 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Debug Mode: Run Everything Eagerly
**EN:** The simplest use case is debugging. The --debug-cuda-graph flag wraps the entire decode forward pass in a graph break, so every operation runs eagerly while still going through the full CUDA graph capture/replay code path.
**CN:** 本节围绕 Debug Mode: Run Everything Eagerly 展开，概述了 CUDA, graph, --debug-cuda-graph, runs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Selective Graph Breaks in Model Code
**EN:** For production use, you can mark specific functions as "non-graphable" using the @eager_on_graph decorator. During CUDA graph capture, these functions run eagerly between captured graph segments.
**CN:** 本节围绕 Selective Graph Breaks in 模型 Code 展开，概述了 CUDA, graph, break_graph, capture 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Server Args
**EN:** This section provides a comparison table for Server Args, covering columns such as Argument, Default, Description and examples such as --debug-cuda-graph, SGLANG_USE_BREAKABLE_CUDA_GRAPH.
**CN:** 本节围绕 服务端 Args 展开，概述了 SGLANG_USE_BREAKABLE_CUDA_GRAPH, CUDA, mode, Wraps 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Capture
**EN:** Breakable CUDA graph extends PyTorch's torch.cuda.CUDAGraph by splitting a single capture into multiple segments separated by graph breaks. During capture, the flow is: `` Begin capture (segment 1) ...
**CN:** 本节围绕 Capture 展开，概述了 capture, segment, End, Begin 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Replay
**EN:** During replay: `` For each segment i: 1. Launch CUDA graph segment i 2.
**CN:** 本节围绕 Replay 展开，概述了 CUDA, graph, Launch, segment 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Output Writeback
**EN:** When a non-graph function produces output during replay, the result must be written back into the same tensor buffers that downstream graph segments reference. The mechanism handles: - **Plain tensors**: In-place copy_() into the original buffer.
**CN:** 本节围绕 Output Writeback 展开，概述了 Tensor, copy_, Plain, Dicts 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Stream Fork/Join Tracking
**EN:** Some models fork work onto secondary CUDA streams (e.g., for overlapped computation). Breakable CUDA graph hooks torch.cuda.Stream.wait_stream to track which streams are forked from the capture stream.
**CN:** 本节围绕 Stream Fork/Join Tracking 展开，概述了 CUDA, streams, graph, forked 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Compatibility
**EN:** **NVIDIA CUDA only.** Breakable CUDA graph is not supported on ROCm/HIP or other non-CUDA platforms. On unsupported platforms, --debug-cuda-graph is automatically disabled with a warning.
**CN:** 本节围绕 Compatibility 展开，概述了 CUDA, cuda-python, SGLANG_MEMORY_SAVER_CUDA_GRAPH, Cannot 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Performance
**EN:** When no graph breaks are inserted, breakable CUDA graph has minimal overhead compared to standard CUDA graph — the capture/replay path is nearly identical.
**CN:** 本节围绕 性能 展开，概述了 One, graph, CUDA, overhead 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Code Reference
**EN:** This section provides a comparison table for Code Reference, covering columns such as File, Description and examples such as python/sglang/srt/model_executor/breakable_cuda_graph/breakable_cuda_graph.py, python/sglang/srt/model_executor/breakable_cuda_graph/cuda_utils.py, python/sglang/srt/model_executor/cuda_graph_runner.py, python/sglang/srt/server_args.py.
**CN:** 本节围绕 Code Reference 展开，概述了 CUDA,  | , Core, variable 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** CUDA / **CN:** CUDA
- **EN:** graph / **CN:** graph
- **EN:** capture / **CN:** capture
- **EN:** segment / **CN:** segment
- **EN:** --debug-cuda-graph / **CN:** --debug-cuda-graph
- **EN:** SGLANG_USE_BREAKABLE_CUDA_GRAPH / **CN:** SGLANG_USE_BREAKABLE_CUDA_GRAPH
- **EN:** One / **CN:** One
- **EN:** break / **CN:** break

## Dependencies / 依赖关系
- `python/sglang/srt/model_executor/breakable_cuda_graph/breakable_cuda_graph.py`
- `python/sglang/srt/model_executor/breakable_cuda_graph/cuda_utils.py`
- `python/sglang/srt/model_executor/cuda_graph_runner.py`
- `python/sglang/srt/server_args.py`
- `python/sglang/srt/environ.py`
