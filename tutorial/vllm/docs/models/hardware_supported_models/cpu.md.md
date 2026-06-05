# cpu.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/hardware_supported_models/cpu.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page is a hardware compatibility matrix for the CPU backend, centered on Intel Xeon platforms and the model families validated there.
  **CN:** 本页是面向 CPU 后端的硬件兼容性矩阵，重点围绕 Intel Xeon 平台及其上已验证的模型家族。
- **EN:** Rather than teaching one workflow, it acts as a deployment reference for choosing supported text-only, multimodal, and quantized models on CPU.
  **CN:** 它并非讲解单一工作流，而是作为部署参考，帮助用户在 CPU 上选择受支持的纯文本、多模态和量化模型。

## Key Concepts / 关键概念
- **EN:** Validated hardware tables identify supported Intel Xeon generations.
  **CN:** 已验证硬件表列出了受支持的 Intel Xeon 代际。
- **EN:** Recommended-model sections map architectures to concrete Hugging Face checkpoints.
  **CN:** 推荐模型部分将架构映射到具体的 Hugging Face 检查点。
- **EN:** The document supports capacity planning for CPU-only inference.
  **CN:** 该文档有助于进行仅用 CPU 推理的容量规划。
