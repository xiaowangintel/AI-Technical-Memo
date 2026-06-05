# inc.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/inc.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This page centers on Intel AutoRound and frames it as a broad quantization toolkit that spans low-bit integer, FP-like, and GGUF outputs, while noting that vLLM currently enables only selected Intel-platform recipes.
  **CN:** 该文档以 Intel AutoRound 为核心，将其描述为覆盖低比特整数、FP 类格式和 GGUF 输出的广泛量化工具，同时说明 vLLM 目前只在 Intel 平台启用了部分配方。
- **EN:** Its structure mixes capability overview and hands-on usage: feature list first, then supported recipes on Intel, followed by CLI quantization, Python API quantization, deployment, and lm-eval evaluation examples.
  **CN:** 文档结构兼顾能力概览与实操指南：先列特性，再说明 Intel 平台当前支持的配方，然后给出 CLI 量化、Python API 量化、部署和 lm-eval 评估示例。
- **EN:** A key deployment nuance is the current `--enforce-eager` requirement for `wNa16` models on Intel GPU/CPU, which signals that this integration still has runtime constraints.
  **CN:** 一个关键部署细节是：当前在 Intel GPU/CPU 上部署 `wNa16` 模型需要加 `--enforce-eager`，说明这项集成仍带有运行时限制。

## Key Concepts / 关键概念
- **EN:** Toolkit scope: AutoRound advertises INT2/3/4/8, MXFP8, MXFP4, NVFP4, and GGUF outputs.
  **CN:** 工具范围：AutoRound 宣称支持 INT2/3/4/8、MXFP8、MXFP4、NVFP4 和 GGUF 输出。
- **EN:** Current vLLM recipes: the page says Intel-platform support currently focuses on `W4A16` and `W8A16`.
  **CN:** 当前 vLLM 配方：文档说明 Intel 平台当前主要支持 `W4A16` 与 `W8A16`。
- **EN:** Export flexibility: Python quantization can save in `auto_round`, `auto_gptq`, or `auto_awq` formats.
  **CN:** 导出灵活性：Python 量化可保存为 `auto_round`、`auto_gptq` 或 `auto_awq` 格式。
- **EN:** Deployment path: the sample deploys an AutoRound INT4 model directly with `vllm serve` and memory-related flags.
  **CN:** 部署路径：示例展示了如何通过 `vllm serve` 和显存相关参数直接部署 AutoRound INT4 模型。
- **EN:** Evaluation integration: the page ties Intel quantized checkpoints back into the standard `lm_eval --model vllm` workflow.
  **CN:** 评估集成：文档将 Intel 量化模型接回标准的 `lm_eval --model vllm` 评估流程。
