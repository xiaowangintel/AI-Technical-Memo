# cuda_graphs.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/cuda_graphs.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This document reframes CUDA Graphs in vLLM as a runtime-dispatch problem instead of a feature tightly coupled to piecewise compilation. **CN:** 本文把 vLLM 中的 CUDA Graphs 重构为一个运行时分发问题，而不是与分段编译强绑定的功能。
- **EN:** The motivation section explains why older designs caused rigid tradeoffs: full graphs improved latency, piecewise graphs preserved compatibility, but the coupling made behavior hard to reason about. **CN:** 动机部分说明了旧设计为何会带来僵硬取舍：全图提升时延表现，分段图保留兼容性，但二者耦合后整体行为难以理解。
- **EN:** The new design introduces multiple `cudagraph_mode` options, including dual-mode strategies that use full graphs for uniform decode and piecewise graphs for other batches. **CN:** 新设计引入多个 `cudagraph_mode` 选项，其中包含双模式策略：对 uniform decode 使用 full graphs，对其他批次使用 piecewise graphs。
- **EN:** Architecturally, the document centers on `BatchDescriptor`, `CudagraphDispatcher`, and `CUDAGraphWrapper`, with the dispatcher acting as the single source of truth for runtime selection. **CN:** 在架构层面，文档围绕 `BatchDescriptor`、`CudagraphDispatcher` 与 `CUDAGraphWrapper` 展开，其中 dispatcher 是运行时选择逻辑的唯一真源。
- **EN:** It also captures an important compatibility policy: attention backend capability determines whether modes are preserved, downgraded, or disabled. **CN:** 文档还强调了重要的兼容性策略：attention backend 的能力会决定某种模式是保留、降级还是禁用。

## Key Concepts / 关键概念
- **EN:** **`CUDAGraphMode`** — vLLM exposes `NONE`, `PIECEWISE`, `FULL`, `FULL_DECODE_ONLY`, and `FULL_AND_PIECEWISE` as explicit operating modes. **CN:** **`CUDAGraphMode`**——vLLM 公开了 `NONE`、`PIECEWISE`、`FULL`、`FULL_DECODE_ONLY` 与 `FULL_AND_PIECEWISE` 五类工作模式。
- **EN:** **`BatchDescriptor`** — runtime batches are identified by token count, request count, uniformity, and LoRA presence to select compatible graph captures. **CN:** **`BatchDescriptor`**——运行时批次通过 token 数、请求数、是否 uniform、是否含 LoRA 等信息来唯一标识并匹配图捕获。
- **EN:** **Dispatcher-driven execution** — `CudagraphDispatcher` decides whether a batch should run in `FULL`, `PIECEWISE`, or `NONE`. **CN:** **Dispatcher 驱动执行**——`CudagraphDispatcher` 决定某个批次应走 `FULL`、`PIECEWISE` 还是 `NONE`。
- **EN:** **Nested wrappers** — a full-graph wrapper can sit outside the model while piecewise wrappers live inside compiled subgraphs, allowing coexistence. **CN:** **嵌套包装器**——全图 wrapper 可包裹在模型外层，而分段 wrapper 位于编译子图内部，从而实现共存。
- **EN:** **Backend compatibility enum** — `AttentionCGSupport` models whether a backend supports always-on, uniform-batch-only, decode-only, or no CUDA Graphs. **CN:** **后端兼容性枚举**——`AttentionCGSupport` 用于描述后端是始终支持、仅支持 uniform batch、仅支持 decode，还是完全不支持 CUDA Graphs。
- **EN:** **Mode downgrading** — unsupported graph modes are automatically mapped to the closest workable alternative based on backend capability and compile setup. **CN:** **模式降级**——若某模式超出后端能力范围，系统会依据后端能力与编译配置自动降级到最接近的可行方案。
