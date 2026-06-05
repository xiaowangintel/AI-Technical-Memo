# moe_kernel_features.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/moe_kernel_features.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** The document is a selection guide for vLLM MoE kernels, comparing both modular and non-modular implementations so developers can match kernels to expert-parallel communication backends and quantization needs.  
  **CN:** 本文是一份 vLLM MoE kernel 选型指南，通过对模块化与非模块化实现的对比，帮助开发者把 kernel 与专家并行通信后端及量化需求正确匹配。
- **EN:** Its first major section catalogs modular all2all backends for `FusedMoE`, focusing on activation format, supported quantization types/formats, async support, and whether top-k weights can be applied on inputs for topk==1 models.  
  **CN:** 第一大部分梳理了 `FusedMoE` 的模块化 all2all 后端，重点比较激活格式、支持的量化类型/格式、异步支持，以及对 topk==1 模型是否支持在输入侧应用 top-k 权重。
- **EN:** The second section compares experts kernels across activation layout, quantization support, activation functions, modular adapters, and source entry points, making compatibility constraints explicit rather than implicit.  
  **CN:** 第二部分对 experts kernel 的激活布局、量化支持、激活函数、模块化适配器和源码入口进行比较，把兼容性约束显式化而不是隐含处理。
- **EN:** The closing “families” table summarizes which prepare/finalize backends and experts subclasses are intended to work together, especially for DeepEP high-throughput, DeepEP low-latency, and FlashInfer combinations.  
  **CN:** 末尾的“families”表总结了哪些 prepare/finalize 后端与 experts 子类被设计为协同工作，尤其针对 DeepEP 高吞吐、DeepEP 低延迟和 FlashInfer 组合。

## Key Concepts / 关键概念
- **EN:** **All2all backend selection** — backend choice determines dispatch/combine behavior, async capability, and the quantized activation format passed between stages.  
  **CN:** **All2all 后端选择** —— 后端决定 dispatch/combine 行为、异步能力，以及阶段之间传递的量化激活格式。
- **EN:** **Standard vs batched activation formats** — prepare/finalize and experts kernels must agree on activation layout to interoperate.  
  **CN:** **标准与 batched 激活格式** —— prepare/finalize 与 experts kernel 必须在激活布局上保持一致才能互操作。
- **EN:** **Quantization compatibility** — kernel/backend support is expressed by quantization type and format (grouped, per-activation, per-tensor, block size variants).  
  **CN:** **量化兼容性** —— kernel/后端支持情况通过量化类型与格式来表达（分组、逐激活、逐张量及不同 block 大小）。
- **EN:** **Async modular backends** — async-capable backends enable features such as DBO and shared-expert overlap.  
  **CN:** **异步模块化后端** —— 支持异步的后端可以启用 DBO 和 shared expert overlap 等能力。
- **EN:** **Kernel families** — the document groups known-good backend/expert combinations to reduce trial-and-error when composing modular MoE paths.  
  **CN:** **Kernel 家族** —— 文档把已知可协同的后端/专家组合归类，减少模块化 MoE 组装时的试错成本。
