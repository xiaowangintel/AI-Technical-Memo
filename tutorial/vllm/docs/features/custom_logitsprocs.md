# custom_logitsprocs.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/custom_logitsprocs.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document explains how to write, load, and use *custom logits processors* in vLLM — user-defined classes that modify the next-token probability distribution at batch granularity without modifying or recompiling vLLM. The API is in beta and subject to change.

**CN:** 本文档介绍如何在 vLLM 中编写、加载和使用*自定义 logits 处理器*——用户自定义类，以批次粒度修改下一个 token 的概率分布，无需修改或重新编译 vLLM。该 API 处于 Beta 阶段，可能会发生变化。

### Core Architecture / 核心架构

**EN:** During each engine step, a logits processor receives a `(num_requests) × (vocab_size)` tensor. It may transform only the rows corresponding to enabled requests. The transformed tensor is then passed to softmax.

**CN:** 在每个引擎步骤中，logits 处理器接收一个 `(num_requests) × (vocab_size)` 张量，只对已启用该处理器的请求所对应的行进行变换，随后将变换后的张量传递给 softmax。

### Required Methods / 必须实现的方法

**EN:** Subclass `vllm.v1.sample.logits_processor.LogitsProcessor` and implement:
- `validate_params(cls, sampling_params)` — validate custom arguments at request time.
- `__init__(self, vllm_config, device, is_pin_memory)` — initialise state.
- `apply(self, logits)` — transform the logits tensor in-place or out-of-place.
- `is_argmax_invariant(self)` — if `True`, skip this processor for fully greedy batches.
- `update_state(self, batch_update)` — handle add/remove/move operations as the persistent batch changes.

**CN:** 继承 `vllm.v1.sample.logits_processor.LogitsProcessor` 并实现：
- `validate_params(cls, sampling_params)` — 在请求阶段验证自定义参数。
- `__init__` — 初始化内部状态。
- `apply(self, logits)` — 原地或非原地变换 logits 张量。
- `is_argmax_invariant(self)` — 若为 `True`，对全贪心批次跳过此处理器。
- `update_state(self, batch_update)` — 响应持久批次的新增/移除/移动操作。

### Batch State Management / 批次状态管理

**EN:** The `BatchUpdate` abstraction describes how vLLM's model runner modifies the persistent batch: finished requests are removed, new requests are added (reusing freed slots), and batch may be condensed or reordered for efficiency. Processors must handle operations in the order: removes → adds → moves.

**CN:** `BatchUpdate` 抽象描述了 vLLM 模型运行器对持久批次的修改方式：结束的请求被移除，新请求被添加（复用已释放的槽位），批次可能被整理或重排以提高效率。处理器必须按照移除→添加→移动的顺序处理操作。

### Wrapping Legacy Request-Level Processors / 包装旧版请求级处理器

**EN:** For vLLM v0 `Callable`-style processors, subclass `AdapterLogitsProcessor` to wrap them into the new batch-level interface.

**CN:** 对于 vLLM v0 `Callable` 风格的处理器，通过继承 `AdapterLogitsProcessor` 将其包装为新的批次级接口。

### Custom Arguments Integration / 自定义参数集成

**EN:** Processors read per-request configuration from `SamplingParams.extra_args` (populated via `vllm_xargs` online or `extra_args` offline). `validate_params` should reject invalid values early.

**CN:** 处理器从 `SamplingParams.extra_args` 读取每个请求的配置（在线通过 `vllm_xargs`，离线通过 `extra_args` 填充），`validate_params` 应尽早拒绝无效值。

## Key Concepts / 关键概念

- **EN:** Batch-granularity processing — a single `apply()` call handles all active requests simultaneously for efficiency. **CN:** 批次粒度处理——单次 `apply()` 调用同时处理所有活跃请求以提升效率。
- **EN:** `is_argmax_invariant` — optimisation hint that lets vLLM skip the processor when all requests use greedy sampling. **CN:** `is_argmax_invariant`——优化提示，当所有请求均使用贪心采样时让 vLLM 跳过该处理器。
- **EN:** `update_state` / `BatchUpdate` — enables stateful processors (e.g., per-request token masks) that track which request occupies which batch slot. **CN:** `update_state` / `BatchUpdate`——支持有状态处理器（例如每请求 token 掩码）跟踪每个批次槽位对应的请求。
- **EN:** No recompile — processors are loaded at engine initialisation via a plugin/import mechanism. **CN:** 无需重新编译——处理器在引擎初始化时通过插件/导入机制加载。
- **EN:** `AdapterLogitsProcessor` — compatibility shim for wrapping v0 callable logits processors. **CN:** `AdapterLogitsProcessor`——用于包装 v0 可调用 logits 处理器的兼容层。
