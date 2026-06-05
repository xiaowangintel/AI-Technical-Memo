# xpu.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/models/hardware_supported_models/xpu.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page catalogs validated Intel GPU/XPU hardware and the models known to run on that backend.
  **CN:** 本页汇总了已验证的 Intel GPU/XPU 硬件，以及已知可在该后端运行的模型。
- **EN:** Its most important value is the capability matrix: readers can compare architectures against BF16/FP16, compressed-tensors FP8, and MXFP4 support columns.
  **CN:** 其最重要的价值在于能力矩阵：读者可以对照不同架构在 BF16/FP16、compressed-tensors FP8 与 MXFP4 列上的支持情况。

## Key Concepts / 关键概念
- **EN:** Validated hardware is listed before model recommendations.
  **CN:** 文档先列出已验证硬件，再给出模型推荐。
- **EN:** Precision and quantization support are first-class selection criteria on XPU.
  **CN:** 在 XPU 场景下，精度与量化支持是一级选型标准。
- **EN:** The page serves as a practical support matrix rather than a usage tutorial.
  **CN:** 该页面更像实用支持矩阵，而非具体操作教程。
