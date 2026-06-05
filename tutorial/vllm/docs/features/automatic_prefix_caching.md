# automatic_prefix_caching.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/automatic_prefix_caching.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document describes Automatic Prefix Caching (APC), a vLLM performance optimisation that caches the KV (Key-Value) cache of processed queries so that subsequent requests sharing the same prefix can skip recomputing the shared portion.

**CN:** 本文档介绍自动前缀缓存（APC），这是 vLLM 的一项性能优化机制，可缓存已处理查询的 KV（键值）缓存，使共享相同前缀的后续请求跳过重复计算。

### Enabling APC / 启用方式

**EN:** Set `enable_prefix_caching=True` in the vLLM engine constructor. An offline example is referenced at `examples/features/automatic_prefix_caching/automatic_prefix_caching_offline.py`.

**CN:** 在 vLLM 引擎构造函数中设置 `enable_prefix_caching=True`。离线示例见 `examples/features/automatic_prefix_caching/automatic_prefix_caching_offline.py`。

### Workload Benefits / 适用场景

**EN:** Two high-value scenarios are described: (1) **Long document query** — the same large document (e.g., a software manual) is queried many times; APC processes the document only once. (2) **Multi-round conversation** — chat history grows across turns; APC reuses the processing result of previous turns.

**CN:** 文档描述了两个高价值场景：(1) **长文档查询**——同一大型文档（如软件手册）被多次查询，APC 只需处理一次。(2) **多轮对话**——历史消息逐轮累积，APC 可复用已处理结果。

### Caveats / 注意事项

**EN:** APC reduces prefill latency only; it does not speed up token generation (decoding). Gains are minimal if: the model spends most time generating long answers, or requests have no common prefix.

**CN:** APC 仅减少预填充延迟，不加速 token 生成（解码）阶段。在以下情况下收益很小：模型大部分时间用于生成长回答，或各请求之间没有公共前缀。

## Key Concepts / 关键概念

- **EN:** KV cache — intermediate attention key/value tensors stored for reuse. **CN:** KV 缓存——存储并复用的注意力键值张量中间结果。
- **EN:** Prefill phase — computing attention over the entire input prompt; this is what APC accelerates. **CN:** 预填充阶段——对整个输入提示词计算注意力，APC 加速的正是这一阶段。
- **EN:** Prefix sharing — two requests share a prefix if the leading token sequence is identical. **CN:** 前缀共享——两个请求的首部 token 序列完全相同时即可共享前缀。
- **EN:** `enable_prefix_caching=True` — the single flag to activate APC. **CN:** `enable_prefix_caching=True`——激活 APC 的唯一标志位。
- **EN:** No accuracy loss — APC is mathematically equivalent to recomputing the shared prefix. **CN:** 无精度损失——APC 与重新计算共享前缀在数学上完全等价。
