# int4.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/int4.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document covers INT4 W4A16 quantization (4-bit weights, 16-bit activations) using `llm-compressor`. It targets memory savings and low-QPS latency reduction. The workflow uses GPTQ with calibration data and includes best-practice guidance and tunable hyperparameters.

**CN:** 本文档介绍使用 `llm-compressor` 进行 INT4 W4A16 量化（4 位权重，16 位激活），适用于内存节省和低 QPS 延迟优化，工作流使用 GPTQ 结合校准数据，并提供最佳实践指导和可调超参数。

### Hardware Support / 硬件支持

**EN:** Supported on NVIDIA GPUs with compute capability > 8.0 (Ampere, Ada Lovelace, Hopper, Blackwell).

**CN:** 支持算力 > 8.0 的 NVIDIA GPU（Ampere、Ada Lovelace、Hopper、Blackwell）。

### Four-Step Workflow / 四步工作流

**EN:** (1) Load model with `transformers` AutoModel. (2) Prepare calibration data — 512 samples from `ultrachat_200k` with chat template applied, max sequence length 2048. (3) Apply `GPTQModifier(targets="Linear", scheme="W4A16", ignore=["lm_head"])`, save compressed. (4) Load in vLLM, evaluate with `lm_eval`.

**CN:** (1) 用 `transformers` AutoModel 加载模型。(2) 准备校准数据——从 `ultrachat_200k` 取 512 条样本，应用对话模板，最大序列长度 2048。(3) 应用 `GPTQModifier(scheme="W4A16")`，保存压缩模型。(4) 在 vLLM 中加载，用 `lm_eval` 评估。

### Best Practices / 最佳实践

**EN:** Start with 512 calibration samples; use high-variety data; apply the model's chat template; for fine-tuned models use training data for calibration. Key hyperparameters: `dampening_frac` (GPTQ influence, lower improves accuracy but risks instability) and `actorder="weight"` (activation ordering, improves accuracy without added latency).

**CN:** 从 512 条校准样本开始；使用高多样性数据；应用模型对话模板；对微调模型使用训练数据校准。关键超参数：`dampening_frac`（GPTQ 影响力，较低值提升精度但有数值不稳定风险）和 `actorder="weight"`（激活排序，提升精度且不增加延迟）。

### Evaluation Note / 评估注意事项

**EN:** Always include `add_bos_token=True` when running `lm_eval` on quantized models, as quantized models are sensitive to the presence of the BOS token.

**CN:** 对量化模型运行 `lm_eval` 时始终包含 `add_bos_token=True`，因为量化模型对 BOS token 的存在较为敏感。

## Key Concepts / 关键概念

- **EN:** W4A16 — 4-bit integer weights with 16-bit floating-point activations; reduces model size ~4× vs BF16. **CN:** W4A16——4 位整数权重 + 16 位浮点激活，模型大小约比 BF16 减少 4 倍。
- **EN:** GPTQ — calibration-based quantization algorithm using second-order Hessian approximation for accurate weight rounding. **CN:** GPTQ——基于校准数据、使用二阶 Hessian 近似的量化算法，实现精确权重舍入。
- **EN:** `GPTQModifier` — llm-compressor modifier that applies GPTQ to specified target layers. **CN:** `GPTQModifier`——llm-compressor 中将 GPTQ 应用于指定目标层的修改器。
- **EN:** `dampening_frac` — regularisation term preventing numerical instability during GPTQ inversion; lower values improve accuracy at the cost of stability. **CN:** `dampening_frac`——防止 GPTQ 求逆时数值不稳定的正则化项，较低值提升精度但牺牲稳定性。
- **EN:** `actorder="weight"` — reorders channels by weight magnitude before quantization, improving accuracy with no latency penalty. **CN:** `actorder="weight"`——量化前按权重大小对通道重排，在不增加延迟的情况下提升精度。
