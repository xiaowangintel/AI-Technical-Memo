# fp8.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/fp8.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This page presents FP8 as a high-performance path for weight and activation quantization, emphasizing roughly 2x memory reduction and up to 1.6x throughput gains with limited accuracy loss.
  **CN:** 该文档将 FP8 定位为高性能的权重与激活量化方案，强调约 2 倍显存节省和最高约 1.6 倍吞吐提升，同时精度损失较小。
- **EN:** The main workflow is clear and staged: install llm-compressor, load a model with transformers, apply `FP8_DYNAMIC` to `Linear` layers, save the checkpoint, then evaluate it inside vLLM with lm-eval.
  **CN:** 主流程分层清晰：安装 llm-compressor，用 transformers 加载模型，对 `Linear` 层施加 `FP8_DYNAMIC`，保存检查点，再在 vLLM 中结合 lm-eval 做评估。
- **EN:** The document is careful about hardware and deployment nuance: Hopper/Ada are the official W8A8 targets, Turing/Ampere fall back to weight-only FP8, and online FP8 quantization is available but offers smaller latency gains.
  **CN:** 文档对硬件与部署细节说明充分：Hopper/Ada 是官方 W8A8 目标平台，Turing/Ampere 走权重量化路径，同时也支持在线 FP8 量化，但延迟收益更有限。

## Key Concepts / 关键概念
- **EN:** FP8 formats: E4M3 trades range for precision, while E5M2 expands dynamic range at lower precision.
  **CN:** FP8 格式：E4M3 以较小动态范围换取更高精度，E5M2 则以更低精度换取更大动态范围。
- **EN:** Recipe design: the recommended quantization targets all `Linear` layers and excludes `lm_head`.
  **CN:** 配方设计：推荐方案覆盖所有 `Linear` 层，并排除 `lm_head`。
- **EN:** Calibration requirement: the documented `FP8_DYNAMIC` flow does not need calibration data.
  **CN:** 校准需求：文档中的 `FP8_DYNAMIC` 流程不需要校准数据。
- **EN:** Evaluation caveat: `lm_eval` should include `add_bos_token=True` because quantized models may be BOS-sensitive.
  **CN:** 评估注意事项：由于量化模型可能对 BOS token 敏感，`lm_eval` 需要设置 `add_bos_token=True`。
- **EN:** Runtime option: vLLM can also quantize BF16/FP16 models online via `quantization="fp8"` or `--quantization="fp8"`.
  **CN:** 运行时选项：vLLM 也可通过 `quantization="fp8"` 或 `--quantization="fp8"` 对 BF16/FP16 模型在线量化。
