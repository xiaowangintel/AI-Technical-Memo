# logits_processors.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/logits_processors.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** Describes the batch-granularity logits processor framework used by the V1 engine, where processors track state for an entire persistent batch rather than isolated sequences. **CN:** 描述 V1 引擎中的 batch 粒度 logits processor 框架，其中处理器跟踪的是整个持久批次的状态，而非彼此独立的序列。
- **EN:** Explains the engine flow: when scheduler output changes a batch, processors receive a `BatchUpdate`; during sampling, the sampler invokes processors to modify logits before token selection. **CN:** 说明引擎流程：当调度器改变批次组成时，处理器会收到 `BatchUpdate`；在采样阶段，sampler 会调用处理器修改 logits 后再选择 token。
- **EN:** Separates argmax-invariant processors from non-invariant ones so greedy-only batches can skip unnecessary work. **CN:** 将保持 argmax 不变的处理器与非保持者区分开来，使纯 greedy 批次可以跳过不必要的计算。
- **EN:** Provides a programming model for built-in and custom processors, including validation, state updates, tensor moves, and ordering guarantees. **CN:** 为内置与自定义处理器提供编程模型，涵盖参数校验、状态更新、张量移动以及执行顺序保证。

## Key Concepts / 关键概念
- **EN:** Batch statefulness: one processor instance is responsible for the evolving contents and order of the active batch. **CN:** 批次状态性：单个处理器实例负责跟踪活动批次内容与顺序的持续变化。
- **EN:** `BatchUpdate`: a structured diff describing removed requests, added requests, and index moves between old and new batch layouts. **CN:** `BatchUpdate`：描述旧批次到新批次之间移除、添加与索引移动的结构化差异对象。
- **EN:** Argmax invariance: processors that do not change the argmax can be skipped for fully greedy decoding paths. **CN:** Argmax 不变性：不会改变 argmax 的处理器可在完全 greedy 的解码路径上被跳过。
- **EN:** Update/apply split: `update_state` handles scheduler-driven batch mutations, while `apply` transforms logits during sampling. **CN:** 更新/应用分离：`update_state` 处理调度器驱动的批次变化，`apply` 在采样时实际修改 logits。
- **EN:** Custom extensibility: the same lifecycle is exposed to user-defined processors as long as they respect validation and device-movement expectations. **CN:** 自定义扩展性：只要遵守校验与设备迁移要求，用户自定义处理器也能复用同一生命周期。
