# cuda_graphs_multimodal.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/cuda_graphs_multimodal.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This document extends the CUDA Graphs story from decoder execution to the vision encoder side, explicitly treating encoder graphs as independent from decoder graphs. **CN:** 本文把 CUDA Graphs 的讨论从解码器扩展到视觉编码器一侧，并明确说明 encoder graph 与 decoder graph 相互独立。
- **EN:** Its central design idea is budget-based capture: multiple encoder graphs are pre-captured at different token budgets and replayed according to the incoming multimodal batch. **CN:** 其核心设计是基于预算的捕获：系统会针对不同 token budget 预先捕获多个 encoder graph，并在运行时按输入多模态批次进行回放。
- **EN:** Runtime execution relies on greedy bin-packing, selecting the smallest budget that fits a sub-batch and falling back to eager mode when an item exceeds all budgets. **CN:** 运行时执行依赖贪心装箱：为每个子批次选择能容纳它的最小 budget；若某项超过全部 budget，则退回 eager 执行。
- **EN:** The design is intentionally model-agnostic via the `SupportsEncoderCudaGraph` protocol, which pushes model-specific capture/replay logic behind a stable interface. **CN:** 该设计通过 `SupportsEncoderCudaGraph` 协议保持模型无关性，把模型特定的捕获/回放逻辑封装在稳定接口之后。
- **EN:** Beyond image inference, the document covers video support, data-parallel distribution for encoder work, configuration knobs, and benchmarked latency gains on Blackwell GPUs. **CN:** 除图像推理外，文档还覆盖了视频支持、encoder 侧的数据并行分发、配置项，以及在 Blackwell GPU 上测得的时延收益。

## Key Concepts / 关键概念
- **EN:** **Budget-based capture** — each `BudgetGraphMetadata` entry binds a token budget to a captured graph and preallocated input/output buffers. **CN:** **基于预算的捕获**——每个 `BudgetGraphMetadata` 条目都把 token budget 与已捕获图及预分配输入/输出缓冲区绑定起来。
- **EN:** **Greedy packing** — incoming items are sorted and packed into sub-batches so replay uses the smallest feasible graph budget. **CN:** **贪心打包**——系统会先排序输入项，再将其装入子批次，以便复用“刚好够用”的最小图预算。
- **EN:** **Protocol-based integration** — models opt in by implementing `SupportsEncoderCudaGraph` methods for budgets, packing, buffers, replay, and eager fallback. **CN:** **基于协议的接入**——模型通过实现 `SupportsEncoderCudaGraph` 中关于预算、打包、缓冲区、回放和 eager 回退的方法来接入该能力。
- **EN:** **Data-parallel encoder execution** — with `mm_encoder_tp_mode="data"`, work is load-balanced across TP ranks and then gathered back in original order. **CN:** **编码器数据并行执行**——在 `mm_encoder_tp_mode="data"` 下，工作会按负载均衡分散到各 TP rank，随后再按原顺序聚合回来。
- **EN:** **Video-specific constraints** — video replay needs different input keys and frame-aware buffers, and EVS disables video graphs because token counts become data-dependent. **CN:** **视频特定约束**——视频回放需要不同输入键与帧相关缓冲区，而 EVS 会让 token 数依赖输入数据，因此会禁用视频图捕获。
- **EN:** **Operational tuning** — `cudagraph_mm_encoder`, token budgets, max vision items, and max frames per batch are the main knobs exposed in `CompilationConfig`. **CN:** **运行调优**——`CompilationConfig` 中主要暴露了 `cudagraph_mm_encoder`、token budgets、每批最大视觉项数和最大帧数等控制开关。
