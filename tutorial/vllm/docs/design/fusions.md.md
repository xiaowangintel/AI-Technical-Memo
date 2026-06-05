# fusions.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/fusions.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** Summarizes vLLM's compile-time fusion passes, framing them as Inductor/TorchDynamo optimizations that are layered on top of model execution rather than model-specific rewrites. **CN:** 总结 vLLM 的编译期融合 pass，将其描述为叠加在模型执行之上的 Inductor/TorchDynamo 优化，而不是特定模型的手工改写。
- **EN:** Provides a quick-reference table for each fusion with config names, fused operators, default optimization level, speedup range, graph requirements, and platform support. **CN:** 提供每个融合项的速查表，列出配置名、被融合算子、默认优化等级、加速范围、图捕获要求以及平台支持情况。
- **EN:** Explains how users control passes through `PassConfig`, Python APIs, and CLI flags, with user settings overriding optimization-level defaults. **CN:** 说明用户如何通过 `PassConfig`、Python API 与 CLI 开关控制这些 pass，且用户显式设置会覆盖优化等级默认值。
- **EN:** The later sections dive into individual passes such as normalization/quantization fusions, RoPE+KV-cache integration, sequence parallelism, and asynchronous tensor-parallel overlap. **CN:** 后续章节逐项展开具体 pass，例如归一化/量化融合、RoPE+KV Cache 融合、sequence parallelism，以及异步 tensor-parallel 重叠执行。

## Key Concepts / 关键概念
- **EN:** `PassConfig`: a per-pass configuration object that can enable, disable, or tune specific graph rewrites. **CN:** `PassConfig`：逐个 pass 的配置对象，可用于启用、禁用或调优特定图重写。
- **EN:** Optimization levels: defaults are grouped by level so users can adopt a conservative or aggressive fusion set. **CN:** 优化等级：默认值按等级分组，便于用户选择更保守或更激进的融合集合。
- **EN:** Full-graph visibility: some fusions only fire when TorchDynamo captures a sufficiently large graph. **CN:** 全图可见性：某些融合只有在 TorchDynamo 捕获到足够大的完整计算图时才会生效。
- **EN:** Hardware specificity: several passes are limited to CUDA, ROCm, Punica/AITER, or certain quantization backends. **CN:** 硬件特异性：多个 pass 只适用于 CUDA、ROCm、Punica/AITER 或特定量化后端。
- **EN:** Performance gating: the document repeatedly ties fusions to token thresholds, tensor shapes, or execution modes instead of promising unconditional speedups. **CN:** 性能门槛：文档多次将融合效果与 token 阈值、张量形状或执行模式关联，而不是承诺无条件加速。
