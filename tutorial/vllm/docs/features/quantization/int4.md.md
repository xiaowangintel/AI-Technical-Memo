# int4.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/int4.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Explains INT4 W4A16 as a vLLM-supported weight-only compression path that targets memory savings and low-latency inference, especially for lower-QPS deployments.
  **CN:** 文档将 INT4 W4A16 描述为 vLLM 支持的权重量化方案，重点是节省显存并保持低延迟，尤其适合较低 QPS 的部署场景。
- **EN:** The tutorial-like flow is explicit: install tooling, load the model, prepare representative calibration data from Ultrachat, run GPTQ-based quantization, then evaluate the saved compressed checkpoint in vLLM.
  **CN:** 其教程式流程非常明确：安装工具、加载模型、用 Ultrachat 准备代表性校准数据、执行基于 GPTQ 的量化，然后在 vLLM 中评估保存后的压缩模型。
- **EN:** Compared with shorter feature pages, this one spends more effort on tuning advice, including calibration diversity, BOS-token sensitivity, and recipe hyperparameters such as `dampening_frac` and `actorder`.
  **CN:** 与更简短的功能页相比，这篇文档更强调调参建议，包括校准数据多样性、BOS token 敏感性，以及 `dampening_frac`、`actorder` 等配方超参数。

## Key Concepts / 关键概念
- **EN:** Hardware support: INT4 computation is documented for NVIDIA GPUs with compute capability above 8.0.
  **CN:** 硬件支持：文档说明 INT4 计算适用于计算能力高于 8.0 的 NVIDIA GPU。
- **EN:** Calibration need: W4A16 quantization requires sample data to estimate weight updates and scales.
  **CN:** 校准需求：W4A16 量化需要样本数据来估计权重更新和缩放系数。
- **EN:** Core recipe: the primary example uses `GPTQModifier(targets="Linear", scheme="W4A16", ignore=["lm_head"])`.
  **CN:** 核心配方：主示例使用 `GPTQModifier(targets="Linear", scheme="W4A16", ignore=["lm_head"])`。
- **EN:** Evaluation caution: the page explicitly warns to include `add_bos_token=True` during lm-eval runs.
  **CN:** 评估注意：文档明确提醒在 lm-eval 中加入 `add_bos_token=True`。
- **EN:** Advanced tuning: the extended recipe shows grouped INT4 quantization with `group_size=128` and optional `actorder="weight"`.
  **CN:** 高级调优：扩展示例展示了 `group_size=128` 的分组 INT4 量化，并可选 `actorder="weight"`。
