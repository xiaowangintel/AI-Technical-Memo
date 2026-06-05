# torchao.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/torchao.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This short page introduces TorchAO as a PyTorch-native optimization library and positions it as a source of quantized checkpoints that can later be consumed through the broader ecosystem.
- **CN:** 这是一页较短的文档，主要把 TorchAO 介绍为 PyTorch 原生优化库，并将其定位为可用于生成量化检查点、再接入更广泛生态的工具。
- **EN:** The practical focus is installation and checkpoint creation: it recommends a nightly TorchAO wheel matched to the local CUDA version, then shows how to quantize a Hugging Face causal LM with `TorchAoConfig(Int8WeightOnlyConfig())`.
- **CN:** 实用重点在于安装与检查点生成：文档推荐安装与本机 CUDA 版本匹配的 TorchAO nightly 轮子，然后展示如何通过 `TorchAoConfig(Int8WeightOnlyConfig())` 量化 Hugging Face 因果语言模型。
- **EN:** The example ends with publishing the tokenizer and quantized model to the Hugging Face Hub, and the final note points readers to a hosted TorchAO Quantization Space for UI-driven workflows.
- **CN:** 示例最后将 tokenizer 与量化模型发布到 Hugging Face Hub，文末还给出一个可视化操作的 TorchAO Quantization Space 入口。

## Key Concepts / 关键概念
- **EN:** TorchAO is presented as composable with native PyTorch features such as `torch.compile` and FSDP, signaling that quantization sits within a larger optimization stack.
  **CN:** 文档强调 TorchAO 可与 `torch.compile`、FSDP 等原生 PyTorch 特性组合使用，说明量化只是更大优化栈中的一部分。
- **EN:** The showcased recipe is weight-only INT8 quantization applied at model load through Transformers integration.
  **CN:** 文中展示的核心配方是通过 Transformers 集成，在模型加载阶段应用 INT8 weight-only 量化。
- **EN:** Distribution is part of the workflow: after quantization, artifacts are pushed to the Hugging Face Hub rather than kept as a purely local export.
  **CN:** 分发也是工作流的一部分：量化后不仅本地保存，还会把产物推送到 Hugging Face Hub。
- **EN:** The linked Hugging Face Space indicates that TorchAO workflows are not limited to code-only usage and can be exposed through a simpler UI.
  **CN:** 所链接的 Hugging Face Space 表明 TorchAO 工作流并不局限于写代码，也可以通过更简化的界面完成。

