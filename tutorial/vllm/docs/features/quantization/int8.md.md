# int8.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/int8.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Introduces INT8 W8A8 as a balanced option for quantizing both weights and activations, aiming to reduce memory footprint while preserving solid inference quality.
  **CN:** 该文档将 INT8 W8A8 介绍为同时量化权重和激活的平衡方案，目标是在降低内存占用的同时保持较好的推理质量。
- **EN:** The procedural structure mirrors the INT4 guide: install llm-compressor, prepare calibration data, apply the recipe, save the model, and evaluate it with vLLM plus lm-eval.
  **CN:** 其流程结构与 INT4 指南基本一致：安装 llm-compressor、准备校准数据、应用配方、保存模型，并结合 vLLM 与 lm-eval 进行评估。
- **EN:** The most important caveat is hardware-specific: INT8 works on Turing/Ampere/Ada/Hopper, but the page explicitly warns that Blackwell GPUs should use FP8 instead.
  **CN:** 最重要的注意事项是硬件差异：INT8 支持 Turing/Ampere/Ada/Hopper，但文档明确警告 Blackwell GPU 应改用 FP8。

## Key Concepts / 关键概念
- **EN:** Blackwell limitation: compute capability >= 10.0 is called out as unsupported for INT8 in this feature.
  **CN:** Blackwell 限制：文档明确指出计算能力 >= 10.0 的 GPU 不支持此 INT8 功能。
- **EN:** Calibration flow: activation quantization requires representative samples to estimate activation scales.
  **CN:** 校准流程：激活量化需要代表性样本来估计激活缩放系数。
- **EN:** Combined recipe: the example pairs `SmoothQuantModifier` with `GPTQModifier(..., scheme="W8A8")`.
  **CN:** 组合配方：示例将 `SmoothQuantModifier` 与 `GPTQModifier(..., scheme="W8A8")` 联合使用。
- **EN:** Saved artifact: the walkthrough saves a `-W8A8-Dynamic-Per-Token` checkpoint for direct vLLM loading.
  **CN:** 保存产物：示例最终保存了一个 `-W8A8-Dynamic-Per-Token` 检查点，供 vLLM 直接加载。
- **EN:** Evaluation note: as with other quantized flows, the page recommends `add_bos_token=True` during evaluation.
  **CN:** 评估说明：与其他量化流程一样，文档建议评估时设置 `add_bos_token=True`。
