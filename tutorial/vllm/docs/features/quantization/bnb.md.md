# bnb.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/bnb.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Introduces BitsAndBytes as a lightweight quantization path in vLLM focused on memory savings and efficiency without requiring calibration data.
  **CN:** 该文档将 BitsAndBytes 介绍为 vLLM 中一种轻量量化路径，重点是节省显存、提升效率，并且不需要校准数据。
- **EN:** Its organization is split by usage mode: read a pre-quantized checkpoint by relying on the model config, or request in-flight 4-bit quantization explicitly.
  **CN:** 内容按使用模式展开：一种是依赖模型配置直接读取预量化检查点，另一种是显式开启在线 4-bit 量化。
- **EN:** The page also connects the same setting to the OpenAI-compatible server, making it clear that Python API and serving workflows share the same quantization choice.
  **CN:** 文档还把同一配置映射到 OpenAI 兼容服务端参数，说明 Python API 与服务化部署在量化选择上是一致的。

## Key Concepts / 关键概念
- **EN:** No-calibration angle: compared with many other methods, BitsAndBytes avoids a separate calibration stage.
  **CN:** 无需校准：与许多其他量化方法相比，BitsAndBytes 不需要单独的校准阶段。
- **EN:** Config-driven loading: vLLM reads `quantization_config` from the model config for pre-quantized checkpoints.
  **CN:** 配置驱动加载：对于预量化模型，vLLM 会读取配置文件中的 `quantization_config`。
- **EN:** In-flight 4-bit mode: dynamic loading of an original model requires `quantization="bitsandbytes"`.
  **CN:** 在线 4-bit 模式：对原始模型进行加载时，需要显式设置 `quantization="bitsandbytes"`。
- **EN:** Server parity: the OpenAI-compatible server uses the same idea through `--quantization bitsandbytes`.
  **CN:** 服务端一致性：OpenAI 兼容服务端通过 `--quantization bitsandbytes` 使用相同能力。
