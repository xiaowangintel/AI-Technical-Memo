# gptqmodel.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/gptqmodel.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Presents GPTQModel as a flexible route for creating 4-bit or 8-bit GPTQ checkpoints, with the value proposition tied to lower memory use and faster inference.
  **CN:** 该文档将 GPTQModel 定位为生成 4-bit 或 8-bit GPTQ 检查点的灵活方案，其价值主张是更低显存占用和更快推理。
- **EN:** It goes beyond a basic introduction by highlighting vLLM kernel integration: compatible checkpoints can use Marlin and Machete kernels on Ampere/Hopper GPUs for stronger batching and latency behavior.
  **CN:** 它不只做基础介绍，还特别强调与 vLLM 内核的集成：兼容模型可在 Ampere/Hopper GPU 上利用 Marlin 与 Machete 内核提升批处理能力和时延表现。
- **EN:** The workflow is straightforward: install GPTQModel, quantize a model using a C4 calibration slice and a `QuantizeConfig`, then run the produced checkpoint through either a CLI example or the Python LLM API.
  **CN:** 整体流程很直接：安装 GPTQModel，使用 C4 校准数据和 `QuantizeConfig` 量化模型，然后通过命令行或 Python LLM API 运行生成的检查点。

## Key Concepts / 关键概念
- **EN:** Bit-width support: the document explicitly covers both INT4 and INT8 GPTQ outputs.
  **CN:** 位宽支持：文档明确覆盖 INT4 与 INT8 两种 GPTQ 输出。
- **EN:** Kernel acceleration: Marlin and Machete are called out as optimized vLLM kernels for compatible GPTQModel checkpoints.
  **CN:** 内核加速：Marlin 与 Machete 被点名为适配 GPTQModel 的 vLLM 优化内核。
- **EN:** Advanced control: GPTQModel supports dynamic per-module quantization overrides for layer-level tuning.
  **CN:** 高级控制：GPTQModel 支持按模块动态覆盖量化配置，可进行层级细粒度调优。
- **EN:** Quantization example: the sample uses `bits=4`, `group_size=128`, and a 1024-sample C4 subset.
  **CN:** 量化示例：示例采用 `bits=4`、`group_size=128`，并使用 1024 条 C4 子集数据进行量化。
- **EN:** vLLM integration: once quantized, the model is loaded like a normal vLLM checkpoint without extra quantization flags in the shown examples.
  **CN:** vLLM 集成：在给出的示例中，量化完成后的模型可像普通 vLLM 检查点一样直接加载，无需额外量化参数。
