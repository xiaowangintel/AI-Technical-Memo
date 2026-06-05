# gptqmodel.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/gptqmodel.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document describes INT4/INT8 GPTQ quantization using GPTQModel from ModelCloud.AI. It highlights advanced features like dynamic per-module quantization and high-performance Marlin/Machete kernels, covering installation, quantization workflow, and running models in vLLM.

**CN:** 本文档介绍使用 ModelCloud.AI 的 GPTQModel 进行 INT4/INT8 GPTQ 量化，重点介绍动态逐模块量化和高性能 Marlin/Machete 内核等高级特性，内容涵盖安装、量化工作流及在 vLLM 中运行模型。

### Key Differentiator / 主要特点

**EN:** GPTQModel supports **Dynamic per-module quantization** — different layers/modules within a model can use different quantization parameters. This is fully integrated into vLLM.

**CN:** GPTQModel 支持**动态逐模块量化**——模型中不同层/模块可使用不同的量化参数，已完全集成到 vLLM 中。

### Marlin and Machete Kernels / Marlin 和 Machete 内核

**EN:** Compatible GPTQ models leverage vLLM's Marlin (Ampere A100+) and Machete (Hopper H100+) custom kernels for world-class TPS and token latency performance.

**CN:** 兼容的 GPTQ 模型可利用 vLLM 的 Marlin（Ampere A100+）和 Machete（Hopper H100+）自定义内核，实现顶级的 TPS 和 token 延迟性能。

### Quantization Workflow / 量化工作流

**EN:** Install `gptqmodel`, load model, prepare calibration dataset (e.g., allenai/c4), set `QuantizeConfig(bits=4, group_size=128)`, call `model.quantize(calibration_dataset)`, then `model.save(path)`. Over 5000 pre-quantized GPTQ models available on HuggingFace.

**CN:** 安装 `gptqmodel`，加载模型，准备校准数据集（如 allenai/c4），设置 `QuantizeConfig(bits=4, group_size=128)`，调用 `model.quantize(calibration_dataset)` 后保存。HuggingFace 上有超过 5000 个预量化 GPTQ 模型可用。

### Running in vLLM / 在 vLLM 中运行

**EN:** Load GPTQModel quantized models directly via `LLM(model="...")` without specifying quantization — vLLM auto-detects from config. CLI equivalent uses `python examples/deployment/llm_engine_example.py --model <model_id>`.

**CN:** 通过 `LLM(model="...")` 直接加载 GPTQModel 量化模型，无需指定量化参数——vLLM 自动从配置文件检测。CLI 方式使用 `python examples/deployment/llm_engine_example.py --model <model_id>`。

## Key Concepts / 关键概念

- **EN:** GPTQ (Generative Pre-trained Transformer Quantization) — a calibration-based post-training quantization algorithm using second-order weight updates. **CN:** GPTQ（生成式预训练 Transformer 量化）——基于校准数据的训练后量化算法，使用二阶权重更新。
- **EN:** Dynamic quantization — per-module quantization parameters allow fine-grained accuracy/performance control. **CN:** 动态量化——逐模块量化参数允许细粒度的精度/性能控制。
- **EN:** Marlin kernel — vLLM's optimised GPTQ kernel for Ampere GPUs; maximises batching throughput. **CN:** Marlin 内核——vLLM 针对 Ampere GPU 的优化 GPTQ 内核，最大化批处理吞吐量。
- **EN:** Machete kernel — vLLM's optimised GPTQ kernel for Hopper GPUs; targets H100+ performance. **CN:** Machete 内核——vLLM 针对 Hopper GPU 的优化 GPTQ 内核，面向 H100+ 性能。
- **EN:** Calibration dataset — GPTQ requires representative data (unlike BnB); quality affects quantized model accuracy. **CN:** 校准数据集——GPTQ（与 BnB 不同）需要代表性数据，质量影响量化模型精度。
