# bnb.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/bnb.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document describes BitsAndBytes (BnB) quantization support in vLLM. BnB can quantize models to 4-bit or 8-bit precision without requiring calibration data, distinguishing it from GPTQ/AWQ. It covers reading pre-quantized checkpoints, inflight 4-bit quantization, and OpenAI-compatible server usage.

**CN:** 本文档介绍 vLLM 中对 BitsAndBytes（BnB）量化的支持。BnB 无需校准数据即可将模型量化至 4 位或 8 位精度，这是其区别于 GPTQ/AWQ 的特点，内容涵盖读取预量化检查点、在线 4 位量化和 OpenAI 兼容服务器用法。

### Key Advantage / 主要优势

**EN:** Unlike GPTQ/AWQ, BitsAndBytes does not require calibration data. The model is quantized dynamically at load time (inflight), making it easy to quantize any model without a separate quantization step.

**CN:** 与 GPTQ/AWQ 不同，BitsAndBytes 不需要校准数据，模型在加载时动态量化（在线量化），无需单独的量化步骤即可对任意模型进行量化。

### Pre-quantized Checkpoints / 预量化检查点

**EN:** vLLM automatically infers the quantization method from the model's `config.json` (a `quantization_config` section). No explicit `quantization` argument needed. Example: `unsloth/tinyllama-bnb-4bit`.

**CN:** vLLM 自动从模型的 `config.json`（`quantization_config` 部分）推断量化方式，无需显式传入 `quantization` 参数。示例：`unsloth/tinyllama-bnb-4bit`。

### Inflight Quantization / 在线量化

**EN:** For standard models, set `quantization="bitsandbytes"` explicitly. The model is loaded in BF16 and quantized to 4-bit during loading.

**CN:** 对于普通模型，显式设置 `quantization="bitsandbytes"`，模型以 BF16 加载并在加载过程中量化为 4 位。

### Server Usage / 服务端用法

**EN:** Append `--quantization bitsandbytes` to `vllm serve` arguments. Install requirement: `bitsandbytes>=0.49.2`.

**CN:** 在 `vllm serve` 参数中追加 `--quantization bitsandbytes`，安装要求：`bitsandbytes>=0.49.2`。

## Key Concepts / 关键概念

- **EN:** No calibration — BnB's primary advantage; quantization is applied directly without needing a representative dataset. **CN:** 无需校准——BnB 的主要优势，无需代表性数据集即可直接量化。
- **EN:** Inflight quantization — model weights are quantized on-the-fly during loading, not saved as a separate checkpoint. **CN:** 在线量化——模型权重在加载时即时量化，不保存为单独的检查点。
- **EN:** `quantization_config` in `config.json` — allows vLLM to auto-detect BnB quantized models without user intervention. **CN:** `config.json` 中的 `quantization_config`——允许 vLLM 无需用户干预自动检测 BnB 量化模型。
- **EN:** 4-bit vs 8-bit — BnB supports both; 4-bit (NF4/INT4) gives the largest memory savings. **CN:** 4 位与 8 位——BnB 均支持，4 位（NF4/INT4）提供最大的内存节省。
- **EN:** `dtype=torch.bfloat16` — recommended base dtype when using BnB quantization in vLLM. **CN:** `dtype=torch.bfloat16`——在 vLLM 中使用 BnB 量化时推荐的基础数据类型。
