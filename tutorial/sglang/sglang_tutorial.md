# SGLang Tutorial / SGLang 双语教程

- This tutorial is a code-oriented, bilingual guide to the SGLang repository at `sgl-project/sglang`.

- It focuses on the real project structure: `python/sglang/lang` for the frontend, `python/sglang/srt` for serving, `sgl-kernel` and `python/sglang/jit_kernel` for fast kernels, `python/sglang/multimodal_gen` for diffusion-style generation, and `sgl-model-gateway` for fleet routing.

- The goal is not to repeat marketing copy, but to help you build a correct mental model of how requests move from prompt programs to tokenized, batched, cached, scheduled, and decoded results.

- 本教程是面向代码的双语导读，覆盖 `sgl-project/sglang` 仓库中的关键子系统。

- 它重点解释真实代码结构：前端语言、服务运行时、CUDA/JIT 内核、多模态生成，以及模型网关。

- 目标不是重复宣传文案，而是帮助你建立“一个请求如何从程序表达变成高效 GPU 推理”的正确心智模型。

```text
Suggested reading order / 建议阅读顺序
1. README.md
2. python/sglang/README.md
3. python/sglang/lang/api.py + interpreter.py
4. python/sglang/srt/entrypoints/engine.py + http_server.py
5. python/sglang/srt/managers/
6. python/sglang/srt/mem_cache/ + layers/attention/
7. python/sglang/srt/models/ + model_loader/
8. docs/advanced_features/*.md
9. sgl-kernel/ + python/sglang/jit_kernel/
10. python/sglang/multimodal_gen/ + sgl-model-gateway/README.md
```

## 1. Introduction / 简介

### English

#### Overview

- SGLang is a serving framework for large language models and multimodal models, not only a prompt DSL.

- The repository combines a frontend programming layer, a high-throughput backend runtime, and specialized kernels.

- The README positions SGLang for single-GPU, multi-GPU, and cluster-scale deployments.

- Key features highlighted by the project are RadixAttention, continuous batching, structured outputs, speculative decoding, quantization, and multi-LoRA support.

- The project also covers embeddings, reward models, rerankers, vision-language models, and diffusion pipelines.

- A practical mental model is: frontend code expresses intent; SRT turns that intent into scheduled GPU work.

#### Key files

- `README.md` gives the high-level positioning, feature list, and links to docs.

- `python/sglang/README.md` summarizes the package layout and entry points.

- `python/sglang/__init__.py` shows the public API surface exported to users.

- `docs/supported_models/` shows the breadth of supported model families.

#### How to read / use it

1. Read the README first to understand what SGLang promises at the product level.

2. Then inspect the public API and package layout to see which modules correspond to those promises.

3. Next, move into `lang/` and `srt/` to understand how the frontend and runtime divide responsibilities.

4. Finally, treat kernels, multimodal generation, and the gateway as optimization and deployment layers built on top of that core split.

### 中文

#### 概览

- SGLang 是一个面向大语言模型与多模态模型的服务框架，而不只是一个提示词 DSL。

- 仓库把前端编程层、高吞吐后端运行时以及专用内核整合在一起。

- README 将 SGLang 定位为可覆盖单卡、多卡到集群级部署的推理系统。

- 项目重点特性包括 RadixAttention、连续批处理、结构化输出、推测解码、量化以及多 LoRA 支持。

- 它还覆盖嵌入模型、奖励模型、重排模型、视觉语言模型以及扩散式多模态生成。

- 一个实用心智模型是：前端代码负责表达意图，SRT 负责把意图转成被调度的 GPU 推理工作。

#### 关键文件

- `README.md` 提供项目定位、特性列表和文档入口。

- `python/sglang/README.md` 总结了包结构与核心入口。

- `python/sglang/__init__.py` 展示了面向用户的公开 API。

- `docs/supported_models/` 展示了支持模型族的广度。

#### 阅读 / 使用方式

1. 先读 README，理解 SGLang 在产品层面承诺了什么。

2. 再查看公开 API 与包结构，确认这些承诺分别对应哪些模块。

3. 然后进入 `lang/` 与 `srt/`，理解前端与运行时的职责边界。

4. 最后把内核、多模态生成与网关视为建立在核心架构之上的优化与部署层。

## 2. Architecture Overview / 架构概览

### English

#### Overview

- At the top level, SGLang splits into a frontend language (`python/sglang/lang`) and a backend runtime (`python/sglang/srt`).

- The `Engine` docstring explicitly says the runtime is composed of TokenizerManager, Scheduler, and DetokenizerManager.

- The main HTTP server and Engine live in the main process, while scheduler and detokenizer run as subprocesses.

- Communication across these components uses ZMQ IPC, which keeps boundaries explicit and debuggable.

- Below the runtime, attention backends, KV-cache managers, model runners, and kernels provide the fast path.

- Beside the core runtime, `sgl-model-gateway` adds fleet routing and `multimodal_gen` adds diffusion-style generation.

#### Key files

- `python/sglang/srt/entrypoints/engine.py` is the clearest architectural anchor.

- `python/sglang/srt/entrypoints/http_server.py` shows the serving surface and lifecycle hooks.

- `python/sglang/srt/managers/` contains the request-processing managers and scheduler helpers.

- `sgl-model-gateway/README.md` shows the architecture when you scale to multiple workers and router policies.

#### How to read / use it

1. Start from the frontend program or an OpenAI-compatible HTTP request.

2. Let TokenizerManager normalize the request, tokenize text, and prepare multimodal payloads if needed.

3. Let Scheduler batch requests, allocate KV resources, run model forward passes, and stream token IDs onward.

4. Let DetokenizerManager turn token IDs into final text chunks and return them to the caller or HTTP stream.

#### Text snapshot

```text
HTTP / OpenAI API / Frontend Program
        |
        v
TokenizerManager  --> tokenization / multimodal preprocessing / request state
        |
        v
Scheduler         --> batching / KV cache / attention backend / model forward
        |
        v
DetokenizerManager --> text chunks / stop handling / streaming output
        |
        v
Client / Stream / Router
```

### 中文

#### 概览

- 从顶层看，SGLang 分为前端语言层（`python/sglang/lang`）与后端运行时（`python/sglang/srt`）。

- `Engine` 的文档字符串明确说明运行时由 TokenizerManager、Scheduler 与 DetokenizerManager 组成。

- 主 HTTP 服务与 Engine 运行在主进程中，而调度器与反分词器运行在子进程中。

- 这些组件之间通过 ZMQ IPC 通信，因此边界清晰，也更方便调试。

- 在运行时之下，注意力后端、KV 缓存管理、模型执行器和专用内核构成了性能快路径。

- 在核心运行时旁边，`sgl-model-gateway` 负责多实例路由，`multimodal_gen` 负责扩散式多模态生成。

#### 关键文件

- `python/sglang/srt/entrypoints/engine.py` 是理解架构的最佳锚点。

- `python/sglang/srt/entrypoints/http_server.py` 展示了服务入口与生命周期钩子。

- `python/sglang/srt/managers/` 包含请求处理管理器与调度辅助组件。

- `sgl-model-gateway/README.md` 展示了在多工作节点场景下的扩展架构。

#### 阅读 / 使用方式

1. 从前端程序或 OpenAI 兼容 HTTP 请求出发。

2. 由 TokenizerManager 规范化请求、完成分词，并在需要时准备多模态输入。

3. 由 Scheduler 对请求做批处理、分配 KV 资源、执行模型前向并继续向后流转 token ID。

4. 由 DetokenizerManager 将 token ID 转成文本分片，并回传给调用方或 HTTP 流。

## 3. SGLang Frontend Language / SGLang 前端语言

### English

#### Overview

- `python/sglang/__init__.py` re-exports the public frontend APIs: `function`, `gen`, `select`, `image`, `video`, role helpers, and backends such as `OpenAI` and `RuntimeEndpoint`.

- `python/sglang/lang/api.py` defines the user-facing building blocks for structured prompting and constrained generation.

- `gen()` is more than plain sampling: it can carry dtype constraints, regex constraints, JSON schema guidance, and logprob options.

- `select()` handles small discrete choices, while `image()` and `video()` attach multimodal content into the program IR.

- `ProgramState` and `StreamExecutor` make branching, streaming, and multi-turn chat feel like normal Python code.

- The frontend is intentionally backend-agnostic: the same program can target local SRT, OpenAI-compatible APIs, or other providers through adapters.

#### Key files

- `python/sglang/lang/api.py` contains `@sgl.function`, `gen`, `select`, `image`, `video`, and role helpers.

- `python/sglang/lang/ir.py` defines IR nodes like `SglGen`, `SglSelect`, `SglImage`, `SglVideo`, and fork-related expressions.

- `python/sglang/lang/interpreter.py` implements `run_program`, `StreamExecutor`, `ProgramState`, and `ProgramStateGroup`.

- `python/sglang/lang/backend/` contains runtime, OpenAI, Anthropic, LiteLLM, and other backend adapters.

#### How to read / use it

1. Wrap a Python function with `@sgl.function` so the frontend can trace and execute it as an SGL program.

2. Use `s += ...` expressions, role scopes, `gen`, `select`, `image`, and `video` to build the program state.

3. Use `fork()` and `join()` when one prompt needs multiple continuations or branch-specific reasoning.

4. Use `text_iter()` or `text_async_iter()` when you want incremental streaming instead of waiting for the full result.

#### Text snapshot

```python
import sglang as sgl

@sgl.function
def qa(s, question, img=None):
    s += sgl.system("You are a concise assistant.")
    with s.user():
        if img is not None:
            s += sgl.image(img)
        s += question
    s += sgl.assistant(sgl.gen("answer", max_tokens=128))
```

### 中文

#### 概览

- `python/sglang/__init__.py` 重新导出了前端公开 API：`function`、`gen`、`select`、`image`、`video`、角色辅助函数，以及 `OpenAI`、`RuntimeEndpoint` 等后端。

- `python/sglang/lang/api.py` 定义了结构化提示与受约束生成的用户侧构件。

- `gen()` 不只是普通采样；它还能携带 dtype 约束、正则约束、JSON schema 引导和 logprob 选项。

- `select()` 用于小规模离散选择，而 `image()` 与 `video()` 用于把多模态内容挂到程序 IR 中。

- `ProgramState` 与 `StreamExecutor` 让分支、多轮对话和流式输出看起来像普通 Python 代码。

- 前端刻意保持后端无关性：同一段程序既能跑本地 SRT，也能跑 OpenAI 兼容接口或其他适配器。

#### 关键文件

- `python/sglang/lang/api.py` 包含 `@sgl.function`、`gen`、`select`、`image`、`video` 与角色辅助函数。

- `python/sglang/lang/ir.py` 定义 `SglGen`、`SglSelect`、`SglImage`、`SglVideo` 及分支相关 IR 节点。

- `python/sglang/lang/interpreter.py` 实现了 `run_program`、`StreamExecutor`、`ProgramState` 与 `ProgramStateGroup`。

- `python/sglang/lang/backend/` 提供 runtime、OpenAI、Anthropic、LiteLLM 等后端适配器。

#### 阅读 / 使用方式

1. 先用 `@sgl.function` 包装 Python 函数，让前端能够把它跟踪并执行为 SGL 程序。

2. 再用 `s += ...`、角色作用域、`gen`、`select`、`image`、`video` 来构造程序状态。

3. 当一个提示需要多条续写或分支推理时，使用 `fork()` 与 `join()`。

4. 如果需要增量结果而不是一次性完整输出，使用 `text_iter()` 或 `text_async_iter()`。

## 4. Serving Runtime (SRT) / 服务运行时

### English

#### Overview

- SRT means SGLang Runtime: the backend engine that actually serves models.

- `http_server.py` is the FastAPI-based serving entry point and hosts both native endpoints and OpenAI-compatible endpoints.

- `engine.py` launches subprocesses, parses `ServerArgs`, wires ZMQ sockets, and exposes programmatic engine APIs.

- `TokenizerManager` initializes model config, tokenizer or multimodal processor, LoRA state, disaggregation state, and request dispatchers.

- `Scheduler` is the heart of the backend: it owns batching, memory pressure decisions, grammar integration, load inquiry, and output streaming.

- `DetokenizerManager` converts token IDs back into text, trims stop markers, and streams final text to the caller.

#### Key files

- `python/sglang/srt/entrypoints/http_server.py` for the HTTP/OpenAI surface.

- `python/sglang/srt/entrypoints/engine.py` for process startup and IPC setup.

- `python/sglang/srt/managers/tokenizer_manager.py` for request preparation.

- `python/sglang/srt/managers/detokenizer_manager.py` and `scheduler.py` for the core serving loop.

#### How to read / use it

1. Launch the server with `python -m sglang.launch_server` or programmatically build an `Engine`.

2. Let `ServerArgs` decide model path, memory fraction, backend choices, scheduling policy, and distributed topology.

3. Receive a request through `/generate` or `/v1/*`, then push it through tokenizer, scheduler, and detokenizer.

4. Use `/server_info`, `/get_model_info`, `/flush_cache`, `/start_profile`, and `/stop_profile` as operational control points.

#### Text snapshot

```bash
python3 -m sglang.launch_server   --model-path meta-llama/Llama-3.1-8B-Instruct   --host 0.0.0.0   --port 30000   --mem-fraction-static 0.8   --chunked-prefill-size 4096
```

### 中文

#### 概览

- SRT 即 SGLang Runtime，是实际承载模型服务的后端引擎。

- `http_server.py` 是基于 FastAPI 的服务入口，同时承载原生接口与 OpenAI 兼容接口。

- `engine.py` 负责启动子进程、解析 `ServerArgs`、连接 ZMQ，并暴露编程式 Engine API。

- `TokenizerManager` 初始化模型配置、分词器或多模态处理器、LoRA 状态、解耦推理状态与请求分发器。

- `Scheduler` 是后端核心：它掌管批处理、内存压力决策、语法约束集成、负载查询与输出流控制。

- `DetokenizerManager` 负责把 token ID 还原为文本，裁剪停止标记，并把结果流式返回。

#### 关键文件

- `python/sglang/srt/entrypoints/http_server.py`：HTTP / OpenAI 服务面。

- `python/sglang/srt/entrypoints/engine.py`：进程启动与 IPC 连接。

- `python/sglang/srt/managers/tokenizer_manager.py`：请求准备阶段。

- `python/sglang/srt/managers/detokenizer_manager.py` 与 `scheduler.py`：核心服务循环。

#### 阅读 / 使用方式

1. 使用 `python -m sglang.launch_server` 启动服务，或在代码中直接构造 `Engine`。

2. 由 `ServerArgs` 决定模型路径、显存比例、后端选择、调度策略与分布式拓扑。

3. 通过 `/generate` 或 `/v1/*` 接收请求，再依次经过 tokenizer、scheduler 与 detokenizer。

4. 把 `/server_info`、`/get_model_info`、`/flush_cache`、`/start_profile`、`/stop_profile` 当作运维控制点。

## 5. RadixAttention & KV Cache / RadixAttention 与 KV 缓存

### English

#### Overview

- `radix_cache.py` explicitly says the radix tree is used for managing the KV cache.

- `RadixKey` stores token IDs plus an optional extra key such as LoRA identity or cache salt.

- The key can also switch into bigram mode for EAGLE-style speculative behavior without materializing a new sequence representation.

- `unified_radix_cache.py` generalizes the tree into components such as full, Mamba, and SWA cache data.

- LRU structures, lock references, and host/device leaf tracking are used to prevent eviction of actively referenced nodes.

- The overall goal is simple: reuse long shared prefixes and avoid recomputing attention states that the server has already seen.

#### Key files

- `python/sglang/srt/mem_cache/radix_cache.py` for radix keys, nodes, and eviction strategies.

- `python/sglang/srt/mem_cache/unified_radix_cache.py` for the newer unified cache implementation.

- `python/sglang/srt/mem_cache/memory_pool.py` and `allocator.py` for KV pool allocation.

- `python/sglang/srt/mem_cache/chunk_cache.py` and `common.py` for chunk/page-level helpers.

#### How to read / use it

1. Match the incoming prompt against existing radix-tree prefixes at token or page granularity.

2. Lock matched nodes so in-use cache blocks cannot be evicted during active generation.

3. Allocate new KV pages only for the uncached suffix or newly generated tokens.

4. Evict cold leaves with policies such as LRU or LFU when memory pressure demands it.

#### Text snapshot

```text
Prompt tokens
   |
   +-- prefix already cached? --> yes --> reuse matched KV pages
   |                            no  --> allocate new pages
   |
   +-- continue decoding --> append new KV --> update radix tree
```

### 中文

#### 概览

- `radix_cache.py` 明确写出：基数树用于管理 KV 缓存。

- `RadixKey` 保存 token ID 以及可选的附加键，例如 LoRA 身份或 cache salt。

- 该键还可以切换到 bigram 模式，以支持 EAGLE 一类推测解码，而无需重新构造序列表达。

- `unified_radix_cache.py` 把树扩展成统一组件结构，支持 full、Mamba、SWA 等不同缓存部件。

- LRU 结构、锁引用以及 host/device 叶子节点跟踪，共同保证活跃节点不会被错误驱逐。

- 其总体目标很直接：重用长公共前缀，避免重复计算服务器已经见过的注意力状态。

#### 关键文件

- `python/sglang/srt/mem_cache/radix_cache.py`：基数键、树节点与驱逐策略。

- `python/sglang/srt/mem_cache/unified_radix_cache.py`：统一缓存实现。

- `python/sglang/srt/mem_cache/memory_pool.py` 与 `allocator.py`：KV 内存池分配。

- `python/sglang/srt/mem_cache/chunk_cache.py` 与 `common.py`：chunk/page 级工具。

#### 阅读 / 使用方式

1. 先以 token 或 page 粒度，将当前提示词与已有基数树前缀进行匹配。

2. 再锁定命中的节点，防止活动请求生成期间缓存块被驱逐。

3. 只为未命中的后缀或新生成 token 分配新的 KV page。

4. 当显存紧张时，再依据 LRU、LFU 等策略驱逐冷数据叶子节点。

## 6. Scheduling & Batching / 调度与批处理

### English

#### Overview

- `scheduler.py` keeps both a `waiting_queue` and a `running_batch`, which is the core state needed for continuous batching.

- Continuous batching means decode work for existing requests can continue while new work is admitted into future batches.

- `init_chunked_prefill()` controls long-prompt splitting through `chunked_prefill_size`.

- Chunked prefill is disabled for multimodal models on the Transformers backend to avoid partial multimodal mismatch.

- `SchedulePolicy` supports multiple request-ordering strategies, including FCFS, LPM, LOF, random, routing-key, and priority-oriented variants.

- Fairness is not only about queue order: LoRA draining, prefill delay, mixed chunking, and preemption thresholds all affect real latency behavior.

#### Key files

- `python/sglang/srt/managers/scheduler.py` for batch construction and queue evolution.

- `python/sglang/srt/managers/schedule_policy.py` for policy-specific ordering.

- `python/sglang/srt/managers/schedule_batch.py` for request batch data structures.

- `python/sglang/srt/managers/overlap_utils.py` and scheduler components for overlap and metrics.

#### How to read / use it

1. Pull eligible requests from the waiting queue under memory and policy constraints.

2. Build a prefill or decode batch, optionally splitting long inputs into chunks.

3. Merge results back into the running decode batch so streaming requests keep making progress.

4. Use priority rules, prefill delay, and LoRA fairness controls to avoid pathological queue starvation.

#### Text snapshot

```text
waiting_queue --> choose requests --> prefill/decode batch --> model forward
      ^                                                        |
      |                                                        v
      +---------------- running_batch <--- merge streamed progress
```

### 中文

#### 概览

- `scheduler.py` 同时维护 `waiting_queue` 与 `running_batch`，这正是连续批处理的核心状态。

- 连续批处理意味着旧请求的 decode 可以持续推进，而新请求则在未来批次中陆续进入。

- `init_chunked_prefill()` 通过 `chunked_prefill_size` 控制长提示词切分。

- 对 Transformers 后端上的多模态模型，chunked prefill 会被关闭，以避免局部多模态块不一致。

- `SchedulePolicy` 支持多种请求排序策略，包括 FCFS、LPM、LOF、随机、routing-key 和优先级变体。

- 公平性不只由队列顺序决定：LoRA draining、prefill delay、mixed chunking 与抢占阈值都会影响真实延迟。

#### 关键文件

- `python/sglang/srt/managers/scheduler.py`：批次构造与队列演化。

- `python/sglang/srt/managers/schedule_policy.py`：不同调度策略。

- `python/sglang/srt/managers/schedule_batch.py`：请求批数据结构。

- `python/sglang/srt/managers/overlap_utils.py` 及相关组件：重叠与指标。

#### 阅读 / 使用方式

1. 在内存与策略约束下，从等待队列中挑选可执行请求。

2. 构造 prefill 或 decode 批次，并在需要时切分长输入。

3. 把结果重新并入运行中的 decode 批，让流式请求持续前进。

4. 利用优先级规则、prefill delay 与 LoRA 公平性控制，避免极端饥饿问题。

## 7. Attention Backends / 注意力后端

### English

#### Overview

- `attention_registry.py` is the central registry that maps a backend name to a backend constructor.

- For standard MHA models, common choices include FlashInfer, FA3, FA4, Triton, Torch Native, AITER, Wave, Ascend, and TRTLLM MHA.

- For MLA models, SGLang exposes FlashInfer MLA, FlashMLA, Cutlass MLA, TRTLLM MLA, FA3, Triton, and FA4 depending on hardware and constraints.

- The `nsa` backend is specialized for DeepSeek V3.2 DSA sparse attention rather than generic dense attention.

- Hybrid linear-attention wrappers also exist for models that mix full attention with GDN or KDA-style layers.

- Backend selection depends on hardware, page size, multimodal requirements, speculative top-k, and KV-cache precision support.

#### Key files

- `python/sglang/srt/layers/attention/attention_registry.py` for backend registration.

- `docs/advanced_features/attention_backend.md` for the support matrix and selection guide.

- `python/sglang/srt/layers/attention/flashinfer_backend.py` and `triton_backend.py` as representative implementations.

- `python/sglang/srt/layers/attention/nsa_backend.py` for DeepSeek sparse attention.

#### How to read / use it

1. Let the server auto-select a backend first, because the registry already encodes many hardware heuristics.

2. Override `--attention-backend` only when you have a concrete reason such as multimodal compatibility or a benchmark result.

3. For hybrid deployments, you can mix prefill and decode backends instead of forcing a single kernel family for both phases.

4. When debugging performance, always check whether your backend supports the page size, KV dtype, and speculative mode you enabled.

#### Text snapshot

```text
Examples
- A100/A40 style MHA default: often FlashInfer
- Hopper MHA default: often FA3
- Blackwell MLA path: often FlashInfer or TRTLLM MLA
- DeepSeek V3.2 sparse path: NSA
```

### 中文

#### 概览

- `attention_registry.py` 是中心注册表，把后端名称映射到实际构造函数。

- 对于标准 MHA 模型，常见选择包括 FlashInfer、FA3、FA4、Triton、Torch Native、AITER、Wave、Ascend 与 TRTLLM MHA。

- 对于 MLA 模型，SGLang 会根据硬件与约束提供 FlashInfer MLA、FlashMLA、Cutlass MLA、TRTLLM MLA、FA3、Triton 与 FA4。

- `nsa` 后端专为 DeepSeek V3.2 的 DSA 稀疏注意力设计，而不是通用致密注意力。

- 对于同时包含全注意力与线性注意力的模型，系统还提供 GDN/KDA 相关的混合包装后端。

- 后端选择取决于硬件、page size、多模态需求、推测 top-k 与 KV 缓存精度支持。

#### 关键文件

- `python/sglang/srt/layers/attention/attention_registry.py`：后端注册中心。

- `docs/advanced_features/attention_backend.md`：支持矩阵与选型指南。

- `python/sglang/srt/layers/attention/flashinfer_backend.py` 与 `triton_backend.py`：代表性实现。

- `python/sglang/srt/layers/attention/nsa_backend.py`：DeepSeek 稀疏注意力实现。

#### 阅读 / 使用方式

1. 优先让服务自动选后端，因为注册表内已经编码了大量硬件启发式规则。

2. 只有在明确知道原因时才手动覆盖 `--attention-backend`，例如多模态兼容或基准测试结果。

3. 在混合部署中，可以分别为 prefill 与 decode 选择不同后端，而不必强行统一。

4. 排查性能时，要确认当前后端是否支持你启用的 page size、KV dtype 与 speculative 模式。

## 8. Model Support / 模型支持

### English

#### Overview

- `models/registry.py` auto-discovers model implementation modules and registers each exported `EntryClass`.

- When a native implementation is unavailable, the registry appends `TransformersForCausalLM` as a fallback path.

- The supported-model docs show broad coverage: Llama, Qwen, DeepSeek, Kimi, Gemma, Mistral, GLM, Phi, MiniCPM, OLMo, Granite, and many others.

- Multimodal docs add Qwen-VL, LLaVA families, Gemma 3 multimodal, GLM-4V, Nemotron Nano VL, DotsVLM, audio transcription models, and video-capable VLMs.

- Embedding, rerank, and reward-model docs make it clear that SGLang is not only for next-token generation.

- `model_loader` and weight utilities handle architecture resolution, load format, quantization config, and weight-name mapping.

#### Key files

- `python/sglang/srt/models/registry.py` for discovery and architecture resolution.

- `python/sglang/srt/model_loader/__init__.py` for the top-level `get_model()` entry.

- `python/sglang/srt/model_loader/weight_utils.py` for weight loading and quant config parsing.

- `docs/supported_models/` for text, multimodal, embedding, rerank, and reward model coverage.

#### How to read / use it

1. Check the supported-model docs first to see whether a family already has an SGLang-native implementation.

2. If not, inspect the model config architecture name and see whether the Transformers fallback is acceptable.

3. When bringing in a new model, align its architecture string with an `EntryClass` or registry update path.

4. For multimodal models, remember that model support often also depends on processors, chat templates, and multimodal attention paths.

### 中文

#### 概览

- `models/registry.py` 会自动发现模型实现模块，并注册其中导出的 `EntryClass`。

- 当没有原生实现时，注册表会把 `TransformersForCausalLM` 追加为回退路径。

- 支持模型文档展示了很广的覆盖面：Llama、Qwen、DeepSeek、Kimi、Gemma、Mistral、GLM、Phi、MiniCPM、OLMo、Granite 等。

- 多模态文档进一步列出 Qwen-VL、LLaVA 系列、Gemma 3 多模态、GLM-4V、Nemotron Nano VL、DotsVLM、音频转写与视频输入模型。

- Embedding、Rerank 与 Reward 文档说明 SGLang 不只是 next-token generation 引擎。

- `model_loader` 与权重工具负责架构解析、加载格式、量化配置与权重名映射。

#### 关键文件

- `python/sglang/srt/models/registry.py`：模型发现与架构解析。

- `python/sglang/srt/model_loader/__init__.py`：顶层 `get_model()` 入口。

- `python/sglang/srt/model_loader/weight_utils.py`：权重加载与量化配置解析。

- `docs/supported_models/`：文本、多模态、嵌入、重排、奖励模型支持列表。

#### 阅读 / 使用方式

1. 先看支持模型文档，确认某个模型家族是否已经有 SGLang 原生实现。

2. 如果没有，再查看模型 config 的 architecture 名称，并判断 Transformers fallback 是否可接受。

3. 引入新模型时，要让它的 architecture 字符串能映射到 `EntryClass` 或注册表扩展。

4. 对多模态模型来说，模型支持通常还依赖处理器、聊天模板与多模态注意力路径。

## 9. Quantization / 量化

### English

#### Overview

- `docs/advanced_features/quantization.md` separates offline quantization from online dynamic quantization.

- Supported methods include FP8, MXFP4, AWQ, GPTQ, compressed-tensors, GGUF, ModelOpt FP8/FP4, bitsandbytes, torchao, and platform-specific variants.

- The docs warn not to add `--quantization` when you are already loading an offline-quantized checkpoint unless you intentionally want to override the behavior.

- KV cache precision is configured separately through `--kv-cache-dtype`, which matters for memory footprint and kernel compatibility.

- For blockwise FP8 and NVFP4, SGLang also exposes backend selection for GEMM execution, such as DeepGEMM, FlashInfer variants, CUTLASS, Triton, and AITER.

- The quantization docs also call out limitations: some mixed-bit, VLM, and MoE combinations need extra validation or layer-specific caution.

#### Key files

- `docs/advanced_features/quantization.md` for the main feature matrix.

- `docs/advanced_features/server_arguments.md` for user-facing flags like `--quantization` and `--kv-cache-dtype`.

- `python/sglang/jit_kernel/` for AWQ, GPTQ Marlin, FP8, NVFP4, and related kernels.

- `python/sglang/srt/layers/quantization/` for runtime-side quantization layers and configs.

#### How to read / use it

1. Decide first whether you are serving a pre-quantized checkpoint or quantizing dynamically at runtime.

2. Match the quantization method to your hardware, because backend availability differs across NVIDIA, AMD, Ascend, and other platforms.

3. Choose KV cache dtype separately from weight precision, because cache kernels can have different support and trade-offs.

4. Benchmark the final configuration instead of assuming that a lower bit width automatically means a faster or better deployment.

#### Text snapshot

```bash
python3 -m sglang.launch_server   --model-path hugging-quants/Meta-Llama-3.1-8B-Instruct-AWQ-INT4   --host 0.0.0.0   --port 30000
```

### 中文

#### 概览

- `docs/advanced_features/quantization.md` 将离线量化与在线动态量化明确区分开来。

- 支持的方法包括 FP8、MXFP4、AWQ、GPTQ、compressed-tensors、GGUF、ModelOpt FP8/FP4、bitsandbytes、torchao 以及平台特定变体。

- 文档特别提醒：如果加载的是离线量化检查点，就不要再额外传 `--quantization`，除非你明确想覆盖默认行为。

- KV cache 精度通过 `--kv-cache-dtype` 单独配置，这会影响显存占用与内核兼容性。

- 对于 blockwise FP8 与 NVFP4，SGLang 还允许为 GEMM 选择不同后端，例如 DeepGEMM、FlashInfer 变体、CUTLASS、Triton 与 AITER。

- 文档也列出了限制：某些 mixed-bit、VLM 与 MoE 组合仍需要额外验证或逐层规避。

#### 关键文件

- `docs/advanced_features/quantization.md`：主要量化支持矩阵。

- `docs/advanced_features/server_arguments.md`：`--quantization`、`--kv-cache-dtype` 等参数。

- `python/sglang/jit_kernel/`：AWQ、GPTQ Marlin、FP8、NVFP4 等相关内核。

- `python/sglang/srt/layers/quantization/`：运行时量化层与配置。

#### 阅读 / 使用方式

1. 先判断你是要服务预量化检查点，还是要在运行时做动态量化。

2. 再让量化方法与硬件匹配，因为不同平台支持的后端并不相同。

3. 把 KV cache dtype 与权重量化精度分开考虑，因为缓存内核的支持矩阵往往不同。

4. 最终一定要用基准测试验证，而不要假设位宽更低就一定更快或更好。

## 10. Parallelism & Distribution / 并行与分布式

### English

#### Overview

- `server_arguments.md` exposes tensor parallelism, pipeline parallelism, data parallelism, attention context parallelism, MoE data parallelism, and expert parallelism.

- The docs explicitly show TP and DP combinations and recommend the model gateway as the preferred DP routing layer.

- `docs/advanced_features/expert_parallelism.md` splits MoE scaling into an all-to-all communication backend and a MoE runner backend.

- Supported EP communication backends include DeepEP, Mooncake, NIXL-EP, MORI, FlashInfer, and Ascend fused EP.

- Multi-node serving uses `--dist-init-addr`, `--nnodes`, and `--node-rank` to form a distributed topology.

- PD disaggregation is a major distribution pattern in SGLang: prefill and decode become separate services connected by a router and KV transfer layer.

#### Key files

- `docs/advanced_features/server_arguments.md` for TP/DP/PP/EP flags.

- `docs/advanced_features/expert_parallelism.md` for MoE scaling design.

- `docs/advanced_features/pd_disaggregation.md` for prefill/decode separation.

- `docs/references/multi_node_deployment/deploy_on_k8s.md` for distributed Kubernetes deployment.

#### How to read / use it

1. Start with single-node TP when the model is too large for one GPU but the topology is still simple.

2. Add DP when you want more throughput and have enough replicated memory budget.

3. Use EP for MoE models when expert weights or all-to-all efficiency become the main bottleneck.

4. Use PD disaggregation when prefill and decode have clearly different resource shapes and deserve independent scaling.

#### Text snapshot

```bash
# Example multi-node TP launch
python -m sglang.launch_server   --model-path meta-llama/Meta-Llama-3-8B-Instruct   --tp 4   --dist-init-addr node0:50000   --nnodes 2   --node-rank 0
```

### 中文

#### 概览

- `server_arguments.md` 暴露了 tensor parallelism、pipeline parallelism、data parallelism、attention context parallelism、MoE data parallelism 与 expert parallelism。

- 文档明确展示了 TP 与 DP 的组合方式，并推荐使用 model gateway 作为更理想的 DP 路由层。

- `docs/advanced_features/expert_parallelism.md` 把 MoE 扩展拆分为 all-to-all 通信后端与 MoE runner 后端两个维度。

- 支持的 EP 通信后端包括 DeepEP、Mooncake、NIXL-EP、MORI、FlashInfer 与 Ascend fused EP。

- 多机服务通过 `--dist-init-addr`、`--nnodes` 与 `--node-rank` 来形成分布式拓扑。

- PD 解耦推理是 SGLang 的重要分布式模式：prefill 与 decode 被拆成独立服务，再由路由器与 KV 传输层连接。

#### 关键文件

- `docs/advanced_features/server_arguments.md`：TP / DP / PP / EP 参数。

- `docs/advanced_features/expert_parallelism.md`：MoE 扩展设计。

- `docs/advanced_features/pd_disaggregation.md`：prefill / decode 分离。

- `docs/references/multi_node_deployment/deploy_on_k8s.md`：分布式 Kubernetes 部署。

#### 阅读 / 使用方式

1. 当模型单卡放不下，但拓扑仍较简单时，先从单机 TP 开始。

2. 当你希望提高吞吐，且能承受副本内存成本时，再加入 DP。

3. 对于 MoE 模型，当专家权重或 all-to-all 成为主瓶颈时，使用 EP。

4. 当 prefill 与 decode 的资源形态明显不同、值得独立扩缩容时，使用 PD 解耦推理。

## 11. Speculative Decoding / 推测解码

### English

#### Overview

- `speculative_decoding.md` shows that SGLang supports EAGLE-2, EAGLE-3, MTP, standalone draft-model decoding, NGRAM speculation, and SpecV2 overlap scheduling.

- The core knobs are draft model path, speculative steps, branching top-k, number of draft tokens, and acceptance thresholds.

- The runtime flow is conceptually draft generation -> target verification -> draft extend/catch-up -> next round.

- Because verification is done by the target model, speculative speedups depend heavily on acceptance rate rather than only draft speed.

- `adaptive_speculative_decoding.md` adds an EMA-based controller that switches among prebuilt step tiers such as `[1, 3, 7]`.

- Attention backend constraints matter: some backends support only top-k=1 or require special care in DP-attention or hybrid-attention settings.

#### Key files

- `docs/advanced_features/speculative_decoding.md` for the main algorithm guide.

- `docs/advanced_features/adaptive_speculative_decoding.md` for adaptive tier switching.

- `python/sglang/jit_kernel/fused_metadata_copy.py` for speculative-related fused modes like verify and draft-extend.

- `python/sglang/srt/` worker and scheduler code for the runtime orchestration.

#### How to read / use it

1. Pick a speculative algorithm that matches your model: EAGLE/EAGLE3 for many LLMs, MTP for models with multi-token heads, STANDALONE if you have a smaller draft model.

2. Tune steps, top-k, and draft-token count together, because these parameters trade compute, memory, and acceptance rate against one another.

3. Inspect `/server_info` or benchmarks to see whether speculative acceptance is good enough to justify the extra machinery.

4. If workload acceptance changes over time, consider the adaptive controller instead of hard-coding one static step count.

#### Text snapshot

```text
draft model -> propose tokens/tree
      |
      v
target model -> verify accepted prefix
      |
      v
if accepted: commit more tokens
if rejected: rollback tail and continue
```

### 中文

#### 概览

- `speculative_decoding.md` 展示了 SGLang 支持 EAGLE-2、EAGLE-3、MTP、独立 draft model、NGRAM 推测以及 SpecV2 重叠调度。

- 核心参数包括 draft 模型路径、推测步数、分支 top-k、draft token 数量以及接受阈值。

- 运行时流程可以概括为：draft 生成 -> target 校验 -> draft 补齐 / 追赶 -> 下一轮。

- 由于校验由 target model 完成，所以推测收益高度依赖接受率，而不仅仅是 draft 模型本身有多快。

- `adaptive_speculative_decoding.md` 进一步加入了基于 EMA 的控制器，可在 `[1, 3, 7]` 等预构建档位间切换。

- 注意力后端约束同样重要：某些后端只支持 top-k=1，或在 DP attention / hybrid attention 场景下需要特殊处理。

#### 关键文件

- `docs/advanced_features/speculative_decoding.md`：主算法指南。

- `docs/advanced_features/adaptive_speculative_decoding.md`：自适应档位切换。

- `python/sglang/jit_kernel/fused_metadata_copy.py`：verify 与 draft-extend 等相关融合模式。

- `python/sglang/srt/` 中的 worker 与 scheduler 代码：运行时编排。

#### 阅读 / 使用方式

1. 先选择与模型匹配的推测算法：EAGLE/EAGLE3 适合很多 LLM，MTP 适合带多 token 头的模型，STANDALONE 适合已有小 draft 模型的场景。

2. 共同调节 steps、top-k 与 draft token 数，因为这些参数会共同影响计算量、显存与接受率。

3. 通过 `/server_info` 或基准测试观察接受率，判断推测解码是否真正值得启用。

4. 如果工作负载的接受率随时间变化明显，就考虑自适应控制器，而不是写死一个固定步数。

## 12. Structured Output / 结构化输出

### English

#### Overview

- Structured generation starts at the frontend, where `gen()` can carry `regex`, `json_schema`, and other constraints.

- `GrammarManager` sits in the scheduler and asynchronously compiles grammar objects for requests that need constrained decoding.

- The base grammar layer supports multiple backends: `xgrammar`, `outlines`, `llguidance`, and `none`.

- `server_args.py` shows that the runtime defaults to `xgrammar`, which is also the fastest and most feature-rich path in practice.

- `xgrammar_backend.py` uses token bitmasks and Triton/CUDA/Torch helper ops to apply vocab constraints efficiently on device.

- `outlines_backend.py` is conceptually simpler: it can turn JSON schema into regex guides and then filter token choices accordingly.

#### Key files

- `python/sglang/srt/constrained/grammar_manager.py` for scheduling-side grammar handling.

- `python/sglang/srt/constrained/base_grammar_backend.py` for backend abstraction and cache behavior.

- `python/sglang/srt/constrained/xgrammar_backend.py` for high-performance token masking.

- `python/sglang/srt/constrained/outlines_backend.py` for regex/JSON-schema guidance using Outlines.

#### How to read / use it

1. Pass a regex, JSON schema, EBNF, or structural tag from the request or frontend program.

2. Let the grammar backend compile that structure, cache it, and synchronize readiness across ranks if needed.

3. During generation, build a vocab mask for the next token and set disallowed logits to negative infinity.

4. Use this path for JSON extraction, tool-call formatting, typed fields, and other schema-sensitive output contracts.

#### Text snapshot

```python
# Frontend-side example
sgl.gen(
    "result",
    max_tokens=128,
    json_schema='{"type":"object","properties":{"answer":{"type":"string"}}}'
)
```

### 中文

#### 概览

- 结构化生成起始于前端：`gen()` 可以携带 `regex`、`json_schema` 等约束。

- `GrammarManager` 位于调度器中，负责为需要受约束解码的请求异步编译 grammar 对象。

- 基础语法层支持多个后端：`xgrammar`、`outlines`、`llguidance` 与 `none`。

- `server_args.py` 表明运行时默认使用 `xgrammar`，这在实践中也是速度最快、能力最强的路径。

- `xgrammar_backend.py` 使用 token bitmask 与 Triton/CUDA/Torch 辅助算子，在设备侧高效施加词表约束。

- `outlines_backend.py` 的概念更直观：它可以把 JSON schema 转成 regex guide，再据此过滤 token 选择。

#### 关键文件

- `python/sglang/srt/constrained/grammar_manager.py`：调度侧语法处理。

- `python/sglang/srt/constrained/base_grammar_backend.py`：后端抽象与缓存逻辑。

- `python/sglang/srt/constrained/xgrammar_backend.py`：高性能 token masking。

- `python/sglang/srt/constrained/outlines_backend.py`：基于 Outlines 的 regex / JSON schema 引导。

#### 阅读 / 使用方式

1. 从请求或前端程序中传入 regex、JSON schema、EBNF 或 structural tag。

2. 由 grammar backend 编译并缓存结构化约束，并在需要时跨 rank 同步就绪状态。

3. 在生成过程中，为下一 token 构造 vocab mask，并把非法 token 的 logit 置为负无穷。

4. 这种路径适合 JSON 抽取、工具调用格式化、类型字段输出以及任何依赖 schema 的结果契约。

## 13. Custom CUDA Kernels (sgl-kernel) / 自定义 CUDA 内核

### English

#### Overview

- `sgl-kernel` is a dedicated C++/CUDA project rather than a tiny helper extension.

- `CMakeLists.txt` fetches and integrates CUTLASS, Triton, FlashInfer, a vendored FlashAttention source tree, and MSCCL++.

- Build flags expose BF16, FP8, FP4, FA3, SM90A, and SM100A support, which tells you the project targets modern accelerator families aggressively.

- The kernel tree covers GEMM, MoE, fused normalization/rope operations, flash-attention variants, and quantization helpers.

- These kernels matter when generic PyTorch or unfused Triton code would leave too much performance on the table.

- Treat `sgl-kernel` as the compiled fast-path library that many higher runtime decisions eventually rely on.

#### Key files

- `sgl-kernel/CMakeLists.txt` for dependencies, architecture flags, and build options.

- `sgl-kernel/csrc/` and `include/` for the actual CUDA/C++ kernel sources.

- `sgl-kernel/python/` for Python-facing bindings where relevant.

- `sgl-kernel/tests/` and `benchmark/` for correctness and performance validation.

#### How to read / use it

1. Read the CMake file first to see which third-party libraries and GPU architectures the project assumes.

2. Map a runtime feature such as FP8 GEMM or fused MoE to its corresponding kernel family under `csrc/` or the JIT layer.

3. Check tests and benchmarks before changing a kernel, because performance regressions are often hardware-specific.

4. Remember that a kernel is only useful when the higher runtime layers actually select it for a given model, dtype, and backend combination.

#### Text snapshot

```text
sgl-kernel stack
- build system: CMake + CUDA + Torch
- deps: CUTLASS / Triton / FlashInfer / FlashAttention / MSCCL++
- outputs: fused kernels for GEMM, MoE, quantization, attention, rope, norm
```

### 中文

#### 概览

- `sgl-kernel` 是一个独立的 C++/CUDA 子工程，而不是一个很小的辅助扩展。

- `CMakeLists.txt` 会拉取并集成 CUTLASS、Triton、FlashInfer、内置 FlashAttention 源树以及 MSCCL++。

- 构建选项暴露了 BF16、FP8、FP4、FA3、SM90A、SM100A 等支持，这说明项目非常积极地面向现代加速器做优化。

- 内核树覆盖 GEMM、MoE、融合归一化 / rope、flash-attention 变体以及量化辅助算子。

- 这些内核的意义在于：当通用 PyTorch 或未融合 Triton 代码损失过多性能时，它们提供更强快路径。

- 可以把 `sgl-kernel` 视为许多高层运行时决策最终依赖的编译型性能库。

#### 关键文件

- `sgl-kernel/CMakeLists.txt`：依赖、架构标志与构建选项。

- `sgl-kernel/csrc/` 与 `include/`：实际 CUDA / C++ 内核源码。

- `sgl-kernel/python/`：必要时提供 Python 侧绑定。

- `sgl-kernel/tests/` 与 `benchmark/`：正确性和性能验证。

#### 阅读 / 使用方式

1. 先读 CMake 文件，确认项目假设了哪些第三方库与 GPU 架构。

2. 再把某个运行时特性（如 FP8 GEMM 或 fused MoE）映射到 `csrc/` 或 JIT 层中的对应内核家族。

3. 修改内核前先看测试与基准，因为性能回归通常具有强硬件相关性。

4. 记住：只有当高层运行时在给定模型、dtype 与 backend 组合下真的选择了该内核时，它才会发挥作用。

## 14. JIT Kernel Compilation / JIT 内核编译

### English

#### Overview

- `python/sglang/jit_kernel/` complements `sgl-kernel` with just-in-time compiled kernels and wrappers.

- `jit_kernel/utils.py` resolves kernel paths, include directories, CUDA architecture, and dependency registration.

- `load_jit()` can compile inline or file-based C++/CUDA modules and expose typed wrappers back to Python.

- The JIT directory contains kernels for AWQ, GPTQ Marlin, FlashAttention v3/v4, fused metadata copy, HiCache, MoE, rope, FP8, NVFP4, and more.

- JIT helps SGLang adapt to the current machine and installed libraries without rebuilding the entire project every time.

- It is especially useful when the runtime wants a specialized kernel only under certain device capabilities or configuration combinations.

#### Key files

- `python/sglang/jit_kernel/utils.py` for path resolution, compilation flags, and architecture detection.

- `python/sglang/jit_kernel/flash_attention_v4.py` for a representative high-value JIT kernel wrapper.

- `python/sglang/jit_kernel/fused_metadata_copy.py` for speculative and metadata fusion helpers.

- `python/sglang/jit_kernel/__main__.py` for development support such as generating `.clangd` information.

#### How to read / use it

1. Resolve the local kernel root and device architecture first, because JIT flags depend on the active GPU.

2. Register extra dependencies like CUTLASS include paths when a kernel family requires them.

3. Compile the module with a unique marker name so multiple kernel variants can coexist safely.

4. Load the compiled functions back into Python and let higher runtime components choose when to call them.

#### Text snapshot

```text
load_jit(...)
  -> resolve kernel path
  -> detect cuda arch
  -> gather include paths / deps
  -> compile module
  -> return callable wrappers
```

### 中文

#### 概览

- `python/sglang/jit_kernel/` 对 `sgl-kernel` 形成补充，提供按需即时编译的内核与包装器。

- `jit_kernel/utils.py` 负责解析内核路径、包含目录、CUDA 架构以及依赖注册。

- `load_jit()` 可以编译 inline 或基于文件的 C++/CUDA 模块，并把类型化包装重新暴露给 Python。

- JIT 目录中包含 AWQ、GPTQ Marlin、FlashAttention v3/v4、fused metadata copy、HiCache、MoE、rope、FP8、NVFP4 等内核。

- JIT 让 SGLang 可以根据当前机器与已安装库自适应，而不必每次都重编整个工程。

- 当运行时只在某些设备能力或配置组合下才需要专门内核时，这种方式尤其有价值。

#### 关键文件

- `python/sglang/jit_kernel/utils.py`：路径解析、编译标志与架构检测。

- `python/sglang/jit_kernel/flash_attention_v4.py`：典型高价值 JIT 内核包装。

- `python/sglang/jit_kernel/fused_metadata_copy.py`：推测解码与元数据融合辅助。

- `python/sglang/jit_kernel/__main__.py`：如生成 `.clangd` 等开发支持。

#### 阅读 / 使用方式

1. 先解析本地内核根目录与设备架构，因为 JIT 标志依赖当前 GPU。

2. 当某个内核家族需要额外依赖时，再注册 CUTLASS 等包含路径。

3. 以唯一标识名编译模块，让多个内核变体可以安全共存。

4. 把编译后的函数加载回 Python，再由高层运行时决定何时调用它们。

## 15. Multimodal Generation / 多模态生成

### English

#### Overview

- `python/sglang/multimodal_gen/` is a separate subsystem for accelerated image and video generation, rather than the text/VLM serving runtime itself.

- `multimodal_gen/registry.py` is the central registry that discovers pipelines and maps model identifiers to pipeline config and sampling-parameter classes.

- The registry lists a large set of image/video families such as Flux, Qwen Image, Wan, MOVA, Hunyuan, Stable Diffusion 3, LTX2, Sana, GlmImage, ErnieImage, and JoyImageEdit.

- The model-info resolver can choose between native SGLang pipelines and diffusers-based fallback behavior.

- This means the project supports both token-generation serving and diffusion-style content generation, but through different runtime stacks.

- When you study SGLang, it is worth keeping these two worlds distinct: SRT for text/VLM serving, multimodal_gen for diffusion/image/video generation pipelines.

#### Key files

- `python/sglang/multimodal_gen/registry.py` for pipeline discovery and model resolution.

- `python/sglang/multimodal_gen/configs/` for pipeline config and sampling-param definitions.

- `python/sglang/multimodal_gen/runtime/` for runtime servers and pipeline implementations.

- `docs/diffusion/` for user-facing diffusion deployment and performance guides.

#### How to read / use it

1. Resolve a model path to a pipeline class plus sampling/config classes.

2. Choose a backend mode such as native SGLang or diffusers fallback depending on pipeline support.

3. Instantiate the composed pipeline and feed sampling parameters appropriate to the image/video task.

4. Treat diffusion serving as its own execution architecture, even though it lives in the same repository.

#### Text snapshot

```text
Model path
   |
   v
registry.get_model_info()
   |
   +--> native SGLang pipeline
   |
   +--> diffusers fallback pipeline
   |
   v
sampling params + runtime pipeline execution
```

### 中文

#### 概览

- `python/sglang/multimodal_gen/` 是一个独立子系统，负责加速图像与视频生成，而不是文本 / VLM 服务运行时本身。

- `multimodal_gen/registry.py` 是中心注册表，它会发现 pipeline，并把模型标识映射到 pipeline config 与采样参数类。

- 注册表列出了大量图像 / 视频家族，如 Flux、Qwen Image、Wan、MOVA、Hunyuan、Stable Diffusion 3、LTX2、Sana、GlmImage、ErnieImage 与 JoyImageEdit。

- 模型信息解析器可以在原生 SGLang pipeline 与 diffusers 回退行为之间做选择。

- 这意味着项目既支持 token 生成式服务，也支持扩散式内容生成，但两者使用不同的运行时栈。

- 学习 SGLang 时，最好明确区分这两个世界：SRT 负责文本 / VLM，multimodal_gen 负责扩散 / 图像 / 视频 pipeline。

#### 关键文件

- `python/sglang/multimodal_gen/registry.py`：pipeline 发现与模型解析。

- `python/sglang/multimodal_gen/configs/`：pipeline 配置与采样参数定义。

- `python/sglang/multimodal_gen/runtime/`：运行时服务与 pipeline 实现。

- `docs/diffusion/`：用户侧扩散部署与性能指南。

#### 阅读 / 使用方式

1. 先把模型路径解析为 pipeline 类以及对应的 sampling/config 类。

2. 再根据支持情况选择原生 SGLang backend 或 diffusers fallback。

3. 实例化组合型 pipeline，并传入适合图像 / 视频任务的采样参数。

4. 即使它们都在同一仓库中，也要把扩散服务视为独立执行架构来理解。

## 16. LoRA & Adapter Support / LoRA 与适配器支持

### English

#### Overview

- SGLang treats LoRA as a first-class serving feature rather than an afterthought.

- The advanced LoRA docs reference S-LoRA and Punica-style ideas for batching multiple adapters efficiently.

- You can preload adapters with `--enable-lora` and `--lora-paths`, then bound memory behavior with `--max-loras-per-batch` and `--max-loaded-loras`.

- The OpenAI-compatible API supports adapter selection directly in the `model` field using `base-model:adapter-name` syntax.

- The HTTP server also exposes dynamic `/load_lora_adapter` and `/unload_lora_adapter` endpoints for hot-swapping adapters at runtime.

- Scheduler-side helpers such as `LoRADrainer` and overlap loading exist because adapter fairness and adapter load latency are real production problems.

#### Key files

- `docs/advanced_features/lora.ipynb` for the conceptual and operational guide.

- `examples/runtime/lora.py` for OpenAI-compatible usage with `model:adapter` syntax.

- `python/sglang/srt/entrypoints/http_server.py` for dynamic load/unload endpoints.

- `docs/advanced_features/server_arguments.md` for LoRA memory, rank, and target-module flags.

#### How to read / use it

1. Preload a small adapter set when you know the steady-state workload ahead of time.

2. Use `model:adapter` for per-request selection on OpenAI-compatible endpoints, because it is cleaner than older extra-body fields.

3. Dynamically load or unload adapters when the workload is broad or fast-changing.

4. Enable overlap loading only when adapter transfer is the true bottleneck, because it can reduce cross-adapter batching opportunities.

#### Text snapshot

```python
# OpenAI-compatible adapter selection
client.chat.completions.create(
    model="meta-llama/Llama-3.1-8B-Instruct:sql",
    messages=[{"role": "user", "content": "Convert to SQL."}]
)
```

### 中文

#### 概览

- SGLang 把 LoRA 当作一等服务特性，而不是事后补充。

- 高级 LoRA 文档明确提到了 S-LoRA 与 Punica 风格的多适配器批处理思想。

- 你可以用 `--enable-lora` 与 `--lora-paths` 预加载适配器，再通过 `--max-loras-per-batch` 与 `--max-loaded-loras` 约束内存行为。

- OpenAI 兼容 API 支持直接在 `model` 字段中使用 `base-model:adapter-name` 语法选择适配器。

- HTTP 服务还暴露了 `/load_lora_adapter` 与 `/unload_lora_adapter` 端点，以便运行时热切换适配器。

- 调度器里的 `LoRADrainer` 与 overlap loading 之所以存在，是因为适配器公平性与加载延迟都是实际生产问题。

#### 关键文件

- `docs/advanced_features/lora.ipynb`：概念与操作指南。

- `examples/runtime/lora.py`：`model:adapter` 语法的 OpenAI 兼容示例。

- `python/sglang/srt/entrypoints/http_server.py`：动态加载 / 卸载端点。

- `docs/advanced_features/server_arguments.md`：LoRA 内存、rank 与目标模块参数。

#### 阅读 / 使用方式

1. 如果你能提前预估稳定工作负载，就预加载一小组常用适配器。

2. 在 OpenAI 兼容接口上，用 `model:adapter` 做逐请求选择，比旧式 extra-body 字段更清晰。

3. 当工作负载覆盖面大、变化快时，使用动态加载 / 卸载。

4. 只有在适配器传输确实成为瓶颈时才启用 overlap loading，因为它可能削弱跨适配器批处理机会。

## 17. Performance Optimization / 性能优化

### English

#### Overview

- Performance in SGLang is the product of runtime policy, memory settings, backend selection, and kernel quality.

- `benchmark_and_profiling.md` organizes the main benchmark tools across four levels: online serving, one-batch-over-HTTP, offline engine throughput, and raw model-runner latency.

- `torch_compile_cache.md` explains how to reuse `torch.compile` cache by setting `TORCHINDUCTOR_CACHE_DIR`.

- CUDA graph support and piecewise CUDA graph features reduce CPU overhead when shapes are stable enough to capture.

- The HTTP server exposes `/start_profile` and `/stop_profile`, and the repo also ships `sglang.profiler` plus benchmark-driven profiling flows.

- Typical tuning knobs include `mem_fraction_static`, `chunked_prefill_size`, `max_running_requests`, attention backend, quantization method, and overlap-related flags.

#### Key files

- `docs/developer_guide/benchmark_and_profiling.md` for benchmark scope and profiler workflow.

- `docs/references/torch_compile_cache.md` for distributed compile-cache reuse.

- `README.md` benchmark section and release blogs for high-level performance context.

- `python/sglang/srt/server_args.py` for the real tuning surface.

#### How to read / use it

1. Benchmark at the right layer: server-level if you care about real serving, runner-level if you care about kernel behavior.

2. Tune memory and batch-shape knobs before changing kernels, because poor scheduling choices can erase low-level kernel gains.

3. Use profiler endpoints or `sglang.profiler` when you need to capture a live decode path.

4. Treat performance claims as workload-specific; always verify against your own prompt lengths, concurrency, and hardware.

#### Text snapshot

```bash
python3 -m sglang.bench_serving --backend sglang   --max-concurrency 16   --num-prompts 80   --random-input-len 256   --random-output-len 32   --dataset-name random
```

### 中文

#### 概览

- SGLang 的性能来自运行时策略、内存设置、后端选择与内核质量的共同作用。

- `benchmark_and_profiling.md` 把基准工具分成四层：在线服务、单批 HTTP、离线 Engine 吞吐，以及原始 ModelRunner 延迟。

- `torch_compile_cache.md` 说明了如何通过设置 `TORCHINDUCTOR_CACHE_DIR` 复用 `torch.compile` 缓存。

- CUDA graph 与 piecewise CUDA graph 会在形状足够稳定时减少 CPU 端开销。

- HTTP 服务暴露 `/start_profile` 与 `/stop_profile`，仓库也提供 `sglang.profiler` 以及基于 benchmark 的 profiling 流程。

- 常见调优旋钮包括 `mem_fraction_static`、`chunked_prefill_size`、`max_running_requests`、attention backend、量化方法与 overlap 相关参数。

#### 关键文件

- `docs/developer_guide/benchmark_and_profiling.md`：基准范围与 profiler 工作流。

- `docs/references/torch_compile_cache.md`：多机 compile-cache 复用。

- `README.md` 的 benchmark 部分与 release blog：高层性能背景。

- `python/sglang/srt/server_args.py`：真实调优面。

#### 阅读 / 使用方式

1. 先在正确层级做基准：关注真实服务就测 server-level，关注内核行为就测 runner-level。

2. 在改内核前先调内存与批形状参数，因为糟糕的调度选择会抹掉低层优化收益。

3. 当你需要捕获真实 decode 路径时，使用 profiler 端点或 `sglang.profiler`。

4. 把任何性能结论都视为工作负载相关，并始终在自己的 prompt 长度、并发和硬件上验证。

## 18. Deployment Guide / 部署指南

### English

#### Overview

- The repository includes Dockerfiles for CUDA, ROCm, XPU, NPU, Xeon, SageMaker, and gateway-specific deployment scenarios.

- `docker/compose.yaml` shows a compact single-service deployment with host networking, HF cache mounting, shared IPC, and a `/health` check.

- `docker/k8s-sglang-service.yaml` shows a Kubernetes Deployment, Service, PVC, and readiness/liveness probes using `/health` and `/health_generate`.

- `deploy_on_k8s.md` covers a more advanced multi-node setup using LeaderWorkerSet and RDMA-aware launch flags.

- `http_server.py` exposes both native endpoints like `/generate` and OpenAI-style endpoints like `/v1/chat/completions`, `/v1/completions`, `/v1/embeddings`, `/v1/rerank`, and `/v1/classify`.

- `sgl-model-gateway` becomes important when you need multi-worker routing, PD-aware traffic shaping, OpenAI proxying, gRPC, or control-plane health and policy logic.

#### Key files

- `docker/compose.yaml` for a compact Docker deployment example.

- `docker/k8s-sglang-service.yaml` for a basic Kubernetes deployment pattern.

- `docs/references/multi_node_deployment/deploy_on_k8s.md` for multi-node details.

- `sgl-model-gateway/README.md` for fleet routing, policies, and OpenAI/gRPC gateway mode.

#### How to read / use it

1. Start with a single worker and health probes before you add routers or distributed launch complexity.

2. Choose endpoint style based on your client: native `/generate` for SGLang internals, `/v1/*` for OpenAI-compatible tooling.

3. Move to Kubernetes or the gateway only when you truly need elastic routing, policy control, or prefill/decode separation.

4. Keep `/server_info`, `/get_model_info`, metrics, and profiling endpoints in your operational toolbox from day one.

#### Text snapshot

```bash
curl http://127.0.0.1:30000/health
curl http://127.0.0.1:30000/get_model_info
curl http://127.0.0.1:30000/server_info
curl http://127.0.0.1:30000/v1/models
```

### 中文

#### 概览

- 仓库内提供了面向 CUDA、ROCm、XPU、NPU、Xeon、SageMaker 以及 gateway 的多种 Dockerfile。

- `docker/compose.yaml` 展示了一个紧凑的单服务部署：host 网络、HF 缓存挂载、共享 IPC 与 `/health` 检查。

- `docker/k8s-sglang-service.yaml` 展示了 Kubernetes Deployment、Service、PVC 以及基于 `/health` 和 `/health_generate` 的探针。

- `deploy_on_k8s.md` 进一步覆盖了基于 LeaderWorkerSet 和 RDMA 的多节点部署。

- `http_server.py` 同时暴露原生端点（如 `/generate`）与 OpenAI 风格端点（如 `/v1/chat/completions`、`/v1/completions`、`/v1/embeddings`、`/v1/rerank`、`/v1/classify`）。

- 当你需要多工作节点路由、PD 感知流量调度、OpenAI 代理、gRPC 或控制面策略逻辑时，`sgl-model-gateway` 就会变得关键。

#### 关键文件

- `docker/compose.yaml`：紧凑的 Docker 部署示例。

- `docker/k8s-sglang-service.yaml`：基础 Kubernetes 部署模式。

- `docs/references/multi_node_deployment/deploy_on_k8s.md`：多节点部署细节。

- `sgl-model-gateway/README.md`：集群路由、策略与 OpenAI / gRPC 网关模式。

#### 阅读 / 使用方式

1. 先从单实例与健康探针开始，再逐步引入路由器或分布式启动复杂度。

2. 根据客户端生态选择接口风格：SGLang 内部工具可用原生 `/generate`，通用生态可用 `/v1/*`。

3. 只有在确实需要弹性路由、策略控制或 prefill/decode 分离时，再迁移到 Kubernetes 或 gateway。

4. 从第一天开始，就把 `/server_info`、`/get_model_info`、metrics 与 profiling 端点纳入运维工具箱。

## 19. Developer Guide / 开发者指南

### English

#### Overview

- `python/sglang/README.md` gives a compact but useful map of the top-level Python package organization.

- `support_new_models.md` says that, for many language models, adding support can be as simple as adding one file under `python/sglang/srt/models/` and exporting `EntryClass`.

- For multimodal models, the developer guide also calls out extra work: model-config registration, chat template handling, multimodal processors, token padding, image features, and vision attention adaptation.

- `attention_backend.md` documents the steps to add a new attention backend, including forward paths and CUDA graph capture/replay hooks.

- The `test/` tree is broad: registered models, lora, disaggregation, kernels, openai server, perf, quantization, scheduler, radix cache, reasoning, and more.

- A healthy contribution loop is: reproduce -> compare against reference -> add tests/benchmarks -> document model or backend support.

#### Key files

- `python/sglang/README.md` for top-level code organization.

- `docs/supported_models/extending/support_new_models.md` for model-porting guidance.

- `docs/advanced_features/attention_backend.md` for backend extension guidance.

- `test/` and `test/registered/` for coverage patterns and expected maintenance style.

#### How to read / use it

1. When adding a model, start from the closest existing implementation rather than from a blank file.

2. When adding an attention backend, first implement the non-CUDA-graph path, then add capture/replay support.

3. Compare SGLang outputs against Hugging Face or another trusted baseline before claiming correctness.

4. Add tests and benchmark notes to make the new path maintainable by people who did not write the original patch.

#### Text snapshot

```text
Useful contributor loop
1. read nearest existing implementation
2. make a surgical change
3. compare against reference output
4. add tests and benchmarks
5. update docs / supported-model lists
```

### 中文

#### 概览

- `python/sglang/README.md` 提供了精炼但非常有用的顶层 Python 包结构图。

- `support_new_models.md` 说明：对许多语言模型而言，新增支持往往只需要在 `python/sglang/srt/models/` 下新增一个文件并导出 `EntryClass`。

- 对多模态模型，开发指南还强调了额外工作：模型配置注册、聊天模板处理、多模态处理器、token padding、图像特征提取与视觉注意力适配。

- `attention_backend.md` 记录了新增注意力后端的步骤，包括前向路径与 CUDA graph 的 capture/replay 钩子。

- `test/` 树非常全面：registered models、lora、disaggregation、kernels、openai server、perf、quantization、scheduler、radix cache、reasoning 等都有覆盖。

- 健康的贡献闭环应当是：复现 -> 与参考基线比较 -> 增加测试 / 基准 -> 更新模型或后端文档。

#### 关键文件

- `python/sglang/README.md`：顶层代码组织。

- `docs/supported_models/extending/support_new_models.md`：模型移植指南。

- `docs/advanced_features/attention_backend.md`：后端扩展指南。

- `test/` 与 `test/registered/`：覆盖模式与维护风格。

#### 阅读 / 使用方式

1. 新增模型时，优先从最相近的现有实现开始，而不是从空文件起步。

2. 新增注意力后端时，先实现非 CUDA-graph 路径，再补齐 capture/replay 支持。

3. 在宣称正确之前，先把 SGLang 输出与 Hugging Face 或其他可信基线对齐比较。

4. 补充测试与基准说明，让后续维护者即使不是原作者也能稳定接手。

## Appendix A. Quick Start Cheatsheet / 附录 A：快速开始速查

### English

- Start a basic server: `python -m sglang.launch_server --model-path meta-llama/Llama-3.1-8B-Instruct --host 0.0.0.0 --port 30000`.

- Query native endpoint: `POST /generate` with `text` and `sampling_params`.

- Query chat endpoint: `POST /v1/chat/completions` with an OpenAI-style messages array.

- Get model metadata: `GET /get_model_info` or `GET /model_info`.

- Get runtime info: `GET /server_info`.

- Flush prefix cache: `POST /flush_cache`.

- Start profiling: `POST /start_profile`.

- Stop profiling: `POST /stop_profile`.

- Use the frontend locally: create an `sgl.function` and run it against `sgl.Runtime("http://127.0.0.1:30000")`.

- Use OpenAI-compatible clients by pointing `base_url` to `http://127.0.0.1:30000/v1`.

### 中文

- 启动基础服务：`python -m sglang.launch_server --model-path meta-llama/Llama-3.1-8B-Instruct --host 0.0.0.0 --port 30000`。

- 调用原生接口：向 `POST /generate` 发送 `text` 与 `sampling_params`。

- 调用聊天接口：向 `POST /v1/chat/completions` 发送 OpenAI 风格的 `messages`。

- 获取模型元数据：`GET /get_model_info` 或 `GET /model_info`。

- 获取运行时信息：`GET /server_info`。

- 清空前缀缓存：`POST /flush_cache`。

- 开始性能分析：`POST /start_profile`。

- 停止性能分析：`POST /stop_profile`。

- 本地使用前端：编写 `sgl.function`，并通过 `sgl.Runtime("http://127.0.0.1:30000")` 运行。

- 如需兼容 OpenAI 客户端，把 `base_url` 指向 `http://127.0.0.1:30000/v1`。

## Appendix B. Feature-to-File Map / 附录 B：功能到文件映射

```text
Feature / 功能                             | Primary files / 主要文件
Frontend API / 前端 API                  | python/sglang/__init__.py, python/sglang/lang/api.py
Program interpreter / 程序解释器         | python/sglang/lang/interpreter.py
Frontend IR / 前端 IR                    | python/sglang/lang/ir.py
Runtime engine / 运行时引擎              | python/sglang/srt/entrypoints/engine.py
HTTP server / HTTP 服务                  | python/sglang/srt/entrypoints/http_server.py
Tokenizer manager / 分词管理             | python/sglang/srt/managers/tokenizer_manager.py
Scheduler / 调度器                       | python/sglang/srt/managers/scheduler.py
Detokenizer / 反分词管理                 | python/sglang/srt/managers/detokenizer_manager.py
Schedule policy / 调度策略               | python/sglang/srt/managers/schedule_policy.py
Radix cache / 基数树缓存                 | python/sglang/srt/mem_cache/radix_cache.py
Unified radix cache / 统一缓存           | python/sglang/srt/mem_cache/unified_radix_cache.py
Memory pool / 内存池                     | python/sglang/srt/mem_cache/memory_pool.py
Attention registry / 注意力注册表        | python/sglang/srt/layers/attention/attention_registry.py
FlashInfer backend / FlashInfer 后端     | python/sglang/srt/layers/attention/flashinfer_backend.py
Triton backend / Triton 后端             | python/sglang/srt/layers/attention/triton_backend.py
NSA backend / NSA 稀疏后端               | python/sglang/srt/layers/attention/nsa_backend.py
Model registry / 模型注册表              | python/sglang/srt/models/registry.py
Model loader / 模型加载器                | python/sglang/srt/model_loader/__init__.py
Weight utils / 权重工具                  | python/sglang/srt/model_loader/weight_utils.py
Grammar manager / 语法管理               | python/sglang/srt/constrained/grammar_manager.py
XGrammar backend / XGrammar 后端         | python/sglang/srt/constrained/xgrammar_backend.py
Outlines backend / Outlines 后端         | python/sglang/srt/constrained/outlines_backend.py
Disaggregation prefill / 解耦预填充      | python/sglang/srt/disaggregation/prefill.py
Disaggregation decode / 解耦解码         | python/sglang/srt/disaggregation/decode.py
JIT kernel loader / JIT 内核加载         | python/sglang/jit_kernel/utils.py
Custom CUDA kernels / 自定义 CUDA 内核   | sgl-kernel/
Multimodal generation / 多模态生成        | python/sglang/multimodal_gen/
Model gateway / 模型网关                 | sgl-model-gateway/
Benchmarks / 基准工具                    | python/sglang/bench_serving.py, bench_one_batch.py
Tests / 测试                             | test/, test/registered/
```

## Appendix C. End-to-End Request Trace / 附录 C：端到端请求轨迹

### English

1. A client sends either a frontend-generated call or an HTTP/OpenAI-compatible request.

2. The HTTP layer validates payload shape and chooses the right serving path.

3. TokenizerManager reads model config and prepares text or multimodal input.

4. If priority scheduling is enabled, default priority may be attached here.

5. If structured decoding is requested, GrammarManager starts or reuses grammar compilation.

6. Scheduler receives the request over IPC and places it into the waiting queue.

7. SchedulePolicy orders waiting work according to cache reuse, fairness, or priority rules.

8. Memory managers check whether prefix cache pages and free KV space are available.

9. A prefill or decode batch is assembled, possibly using chunked prefill.

10. The chosen attention backend and model runner execute the forward pass.

11. Batch results update KV cache state, request state, metrics, and streaming buffers.

12. DetokenizerManager decodes token IDs into text, trims stop strings or tokens, and returns output.

### 中文

1. 客户端发送一个由前端程序生成的调用，或一个 HTTP / OpenAI 兼容请求。

2. HTTP 层先校验载荷结构，并选择合适的服务路径。

3. TokenizerManager 读取模型配置，并准备文本或多模态输入。

4. 如果启用了优先级调度，默认优先级可能会在这里补上。

5. 如果请求了结构化解码，GrammarManager 会启动或复用语法编译过程。

6. Scheduler 通过 IPC 收到请求，并把它放入等待队列。

7. SchedulePolicy 按缓存复用、公平性或优先级规则为等待请求排序。

8. 内存管理器检查前缀缓存页与空闲 KV 空间是否足够。

9. 系统构造 prefill 或 decode 批次，并在需要时使用 chunked prefill。

10. 选中的 attention backend 与 model runner 执行前向计算。

11. 批结果会更新 KV 缓存状态、请求状态、指标与流式缓冲区。

12. DetokenizerManager 将 token ID 解码为文本，裁剪 stop string / token，并返回输出。

## Appendix D. Glossary / 附录 D：术语表

- **SRT**: Serving Runtime，SGLang 的核心后端运行时。

- **Frontend language**: SGLang 前端语言层，用 Python 结构表达生成程序。

- **TokenizerManager**: 负责分词、请求准备与部分前置状态管理。

- **Scheduler**: 负责批处理、缓存、调度与模型执行编排。

- **DetokenizerManager**: 负责把 token ID 还原为文本输出。

- **RadixAttention**: 利用基数树前缀缓存实现高复用率 KV 命中。

- **KV cache**: Key/Value attention states reused across decoding steps.

- **Continuous batching**: 在持续处理旧请求的同时吸收新请求的批处理方式。

- **Chunked prefill**: 将长 prompt 切成块，以降低长上下文预填充的瞬时开销。

- **Structured output**: 通过 schema / regex / grammar 约束输出格式。

- **XGrammar**: SGLang 默认的高性能语法约束后端。

- **Outlines**: 基于 regex / JSON-schema 引导的约束生成后端。

- **FlashInfer**: 常见高性能注意力与推理内核生态。

- **FA3 / FA4**: FlashAttention 3 / 4，对不同 GPU 代际有不同优势。

- **MLA**: Multi-head Latent Attention，DeepSeek 等模型使用的注意力形式。

- **NSA**: DeepSeek V3.2 的稀疏注意力后端。

- **TP**: Tensor Parallelism，张量并行。

- **DP**: Data Parallelism，数据并行。

- **PP**: Pipeline Parallelism，流水并行。

- **EP**: Expert Parallelism，MoE 专家并行。

- **PD disaggregation**: 将 prefill 与 decode 拆分为独立服务。

- **Speculative decoding**: 用 draft 路径先预测，再由 target 验证的加速解码方式。

- **LoRA**: 低秩适配器，可在不复制基础模型的情况下切换能力。

- **HiCache**: 分层 / 扩展式缓存相关能力，用于更大范围的 KV 管理。

- **Model gateway**: 负责多实例路由、策略、控制面与 OpenAI/gRPC 网关。

- **JIT kernel**: 按需即时编译的内核，不必重编整个工程。

- **sgl-kernel**: SGLang 的编译型 C++/CUDA 内核库。

- **Diffusion pipeline**: 用于图像 / 视频生成的扩散式执行流水线。

- **OpenAI-compatible API**: 兼容 `/v1/*` 风格接口，便于复用现有客户端生态。

- **ServerArgs**: 集中定义服务启动与性能调优参数的数据结构。

## Appendix E. Recommended Reading Path / 附录 E：推荐阅读路径

### English

1. Read `README.md` and `python/sglang/README.md` back-to-back.

2. Read `python/sglang/__init__.py` to see what users import first.

3. Read `python/sglang/lang/api.py` and `interpreter.py` to understand the frontend contract.

4. Read `engine.py` and `http_server.py` to understand runtime startup and HTTP exposure.

5. Read `tokenizer_manager.py`, `scheduler.py`, and `detokenizer_manager.py` in that order.

6. Read radix-cache and memory-pool code to understand the real performance story.

7. Read `attention_registry.py` and the attention-backend docs before touching kernels.

8. Read model registry and loader code before adding a new architecture.

9. Read quantization, speculative-decoding, and LoRA docs before changing production flags.

10. Read deployment docs and the model gateway README last, after the single-worker flow is clear.

### 中文

1. 把 `README.md` 与 `python/sglang/README.md` 连续读完。

2. 再看 `python/sglang/__init__.py`，理解用户最先导入的 API 面。

3. 接着阅读 `python/sglang/lang/api.py` 与 `interpreter.py`，掌握前端契约。

4. 然后阅读 `engine.py` 与 `http_server.py`，理解运行时启动与 HTTP 暴露。

5. 再按顺序读 `tokenizer_manager.py`、`scheduler.py` 与 `detokenizer_manager.py`。

6. 随后阅读 radix cache 与 memory pool 代码，理解真实性能来源。

7. 在碰内核前，先读 `attention_registry.py` 与 attention backend 文档。

8. 在新增架构前，先读模型注册与加载器代码。

9. 在修改生产参数前，先读量化、推测解码与 LoRA 文档。

10. 最后在单实例流程完全清楚后，再读部署文档与 model gateway README。
