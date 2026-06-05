# langchain.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/serving/integrations/langchain.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This short integration note shows that LangChain can call vLLM through the `langchain_community.llms.VLLM` wrapper.
  **CN:** 这份简短的集成说明展示了 LangChain 可通过 `langchain_community.llms.VLLM` 封装来调用 vLLM。
- **EN:** The emphasis is on practical instantiation: installation, model name, sampling parameters, and optional tensor parallel settings for distributed inference.
  **CN:** 重点在于实际实例化方式：包括安装、模型名、采样参数，以及用于分布式推理的可选 tensor parallel 设置。

## Key Concepts / 关键概念
- **EN:** Install support with `pip install langchain langchain_community -q`.
  **CN:** 使用 `pip install langchain langchain_community -q` 安装支持。
- **EN:** The example uses LangChain's `VLLM` class for local inference calls.
  **CN:** 示例通过 LangChain 的 `VLLM` 类发起本地推理调用。
- **EN:** `tensor_parallel_size` can be passed when scaling beyond a single GPU.
  **CN:** 扩展到多 GPU 时可以传入 `tensor_parallel_size`。
