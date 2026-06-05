# basic.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/model/basic.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Walks contributors from raw PyTorch model code to a vLLM-compatible implementation by standardizing module construction, embedding access, forward signatures, and weight loading.
- **CN:** 本文带领贡献者把原始 PyTorch 模型代码改造成兼容 vLLM 的实现，重点统一模块构造、embedding 接口、forward 签名以及权重加载逻辑。
- **EN:** It also goes beyond the minimal path by introducing tensor-parallel and quantization-friendly layers, then answering advanced questions such as sliding-window and Mamba-style architectures.
- **CN:** 它不仅覆盖最小接入路径，还介绍了张量并行与量化友好的层，并进一步回答滑动窗口、Mamba 类架构等更高级的问题。

## Key Concepts / 关键概念
- **EN:** `prefix` propagation — Every vLLM module should receive a stable prefix so runtime registration and selective quantization work correctly.
- **CN:** `prefix` 传递 — 每个 vLLM 模块都应接收稳定的 `prefix`，以便运行时注册和选择性量化正确工作。
- **EN:** Embedding/forward adaptation — `embed_input_ids` and a flattened `forward` signature align model code with vLLM execution expectations.
- **CN:** Embedding 与 forward 适配 — 通过 `embed_input_ids` 和扁平化的 `forward` 签名，让模型代码符合 vLLM 的执行约定。
- **EN:** Parallel-friendly layers — Special linear and embedding layers are provided for tensor parallelism and quantization support.
- **CN:** 并行友好层 — 文档提供了支持张量并行和量化的专用线性层与 embedding 层。
- **EN:** Weight loading — Custom `load_weights` logic must map Hugging Face checkpoints into vLLM layer structure.
- **CN:** 权重加载 — 自定义 `load_weights` 需要把 Hugging Face checkpoint 正确映射到 vLLM 的层结构中。
- **EN:** Advanced architectures — The FAQ covers sliding windows, Mamba-only, hybrid, and custom mamba-like layers.
- **CN:** 高级架构 — FAQ 还讨论了滑动窗口、纯 Mamba、混合架构以及自定义 mamba-like 层。
