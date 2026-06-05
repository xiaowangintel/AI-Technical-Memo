# auto_awq.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/auto_awq.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Explains how AutoAWQ can create 4-bit INT4 models for vLLM, but the page opens with a strong deprecation warning and points users to llm-compressor AWQ examples as the preferred path.
  **CN:** 说明 AutoAWQ 可用于生成适配 vLLM 的 4-bit INT4 模型，但文档开头明确提示该库已弃用，并建议改用 llm-compressor 中的 AWQ 示例。
- **EN:** The structure is practical: install AutoAWQ, quantize a sample Mistral model with a small config dict, then run the result in vLLM through either a CLI example or the Python LLM API.
  **CN:** 文档结构偏实操：先安装 AutoAWQ，再用一个配置字典量化示例 Mistral 模型，最后分别给出 vLLM 的命令行和 Python LLM API 推理方式。
- **EN:** It frames the main value as lower memory usage and latency, while also implying that this route is mainly relevant for existing AWQ assets rather than the newest recommended workflow.
  **CN:** 它将核心价值定位为降低显存占用和延迟，同时也暗示这条路径更适合已有 AWQ 资产，而不是最新推荐的量化流程。

## Key Concepts / 关键概念
- **EN:** Deprecation and migration: AutoAWQ is deprecated; llm-compressor AWQ examples are the recommended replacement.
  **CN:** 弃用与迁移：AutoAWQ 已弃用，推荐迁移到 llm-compressor 的 AWQ 示例。
- **EN:** INT4 workflow: the example uses `zero_point`, `q_group_size=128`, `w_bit=4`, and `version="GEMM"` for quantization.
  **CN:** INT4 流程：示例使用 `zero_point`、`q_group_size=128`、`w_bit=4` 和 `version="GEMM"` 完成量化。
- **EN:** Inference integration: vLLM can serve AWQ models with `--quantization awq` or `LLM(..., quantization="AWQ")`.
  **CN:** 推理集成：vLLM 可通过 `--quantization awq` 或 `LLM(..., quantization="AWQ")` 加载 AWQ 模型。
- **EN:** Model sourcing: users can either quantize their own checkpoints or start from AWQ checkpoints already published on Hugging Face.
  **CN:** 模型来源：既可以自己量化模型，也可以直接使用 Hugging Face 上已发布的 AWQ 检查点。
