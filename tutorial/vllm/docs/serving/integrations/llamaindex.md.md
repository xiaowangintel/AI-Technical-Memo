# llamaindex.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/serving/integrations/llamaindex.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page shows the minimal path for using vLLM from LlamaIndex via the `llama_index.llms.vllm.Vllm` class.
  **CN:** 本页展示了通过 `llama_index.llms.vllm.Vllm` 类在 LlamaIndex 中使用 vLLM 的最小路径。
- **EN:** The guide is intentionally lightweight, focusing on installation and constructor arguments such as model name, tensor parallel size, and extra vLLM kwargs.
  **CN:** 该指南刻意保持轻量，重点介绍安装以及模型名、tensor parallel size、额外 vLLM 参数等构造器参数。

## Key Concepts / 关键概念
- **EN:** Install the integration with `pip install llama-index-llms-vllm -q`.
  **CN:** 通过 `pip install llama-index-llms-vllm -q` 安装集成。
- **EN:** `Vllm(...)` wraps vLLM inference inside the LlamaIndex ecosystem.
  **CN:** `Vllm(...)` 在 LlamaIndex 生态内封装了 vLLM 推理。
- **EN:** `vllm_kwargs` exposes lower-level vLLM tuning such as GPU memory utilization.
  **CN:** `vllm_kwargs` 可暴露更底层的 vLLM 调优项，例如 GPU 内存利用率。
