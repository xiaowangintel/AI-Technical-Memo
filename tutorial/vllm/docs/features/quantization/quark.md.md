# quark.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/quark.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This is a broad integration guide for AMD Quark in vLLM. It starts with Quark's role on AMD GPUs and highlights support for weight, activation, and KV-cache quantization plus algorithms such as AWQ, GPTQ, Rotation, and SmoothQuant.
- **CN:** 这是一份较完整的 AMD Quark 与 vLLM 集成指南。文档先介绍 Quark 在 AMD GPU 上的定位，并强调其对权重、激活、KV-cache 量化以及 AWQ、GPTQ、Rotation、SmoothQuant 等算法的支持。
- **EN:** The main body follows a concrete five-step PTQ workflow: load a Transformers model, prepare a calibration dataloader, define FP8-per-tensor and KV-cache configs (including an external JSON algorithm config), quantize/freeze/export to Hugging Face safetensors, and evaluate through vLLM or `lm_eval`.
- **CN:** 主体部分按五步 PTQ 流程展开：加载 Transformers 模型、准备校准数据加载器、定义 FP8 per-tensor 与 KV-cache 配置（含外部 JSON 算法配置）、量化/冻结/导出为 Hugging Face safetensors，并通过 vLLM 或 `lm_eval` 评测。
- **EN:** Beyond the base example, the page covers Quark's convenience script, OCP MX offline models (MXFP4/MXFP6) with simulation fallback on hardware lacking native MX support, and layerwise mixed-precision AMP checkpoints that vLLM can reload directly.
- **CN:** 除基础示例外，文档还扩展到 Quark 的便捷脚本、OCP MX 离线模型（MXFP4/MXFP6）及其在缺乏原生 MX 支持硬件上的模拟执行，以及 vLLM 可直接重载的分层混合精度 AMP 检查点。

## Key Concepts / 关键概念
- **EN:** The documented dependency set is explicit: install `amd-quark`, then pair it with `vllm` and `lm-eval` for deployment and evaluation.
  **CN:** 文档明确列出了依赖组合：先安装 `amd-quark`，再配合 `vllm` 与 `lm-eval` 完成部署和评测。
- **EN:** Quantization configuration is a central theme, including global specs, layer-specific KV-cache patterns, excluded layers, and external JSON algorithm presets such as AutoSmoothQuant.
  **CN:** 量化配置是全文核心，包括全局规格、KV-cache 层模式、排除层，以及 AutoSmoothQuant 这类外部 JSON 算法预设。
- **EN:** vLLM-side loading is tightly coupled to export choices: the example uses Hugging Face safetensors, `quantization="quark"`, and `kv_cache_dtype="fp8"`.
  **CN:** vLLM 侧加载方式与导出选择紧密耦合：示例采用 Hugging Face safetensors，并通过 `quantization="quark"` 与 `kv_cache_dtype="fp8"` 完成推理。
- **EN:** The MX/OCP section adds an important caveat: current integration supports dynamic activation quantization only, while simulation kernels allow evaluation on devices without native MX instructions.
  **CN:** MX/OCP 部分补充了重要限制：当前集成仅支持动态激活量化；但模拟内核允许在没有原生 MX 指令的设备上进行评估。
- **EN:** Mixed precision is presented as an accuracy-throughput tradeoff strategy, not just another format, with ready-made checkpoint examples for practical reuse.
  **CN:** 文档把混合精度描述为在精度与吞吐之间取平衡的策略，而不仅仅是另一种格式，并给出可直接复用的现成检查点示例。

