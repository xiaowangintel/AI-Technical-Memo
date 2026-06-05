# quark.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/quark.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document covers AMD Quark quantization toolkit integration with vLLM for AMD GPU deployment. Quark supports FP8, INT8, and other formats with algorithms including AWQ, GPTQ, Rotation, and SmoothQuant. The workflow involves a 5-step process from model loading to evaluation.

**CN:** 本文档介绍 AMD Quark 量化工具包与 vLLM 的集成，面向 AMD GPU 部署，支持 FP8、INT8 等格式及 AWQ、GPTQ、Rotation、SmoothQuant 等算法，工作流包含从模型加载到评估的 5 步流程。

### Quark's Strengths / Quark 的优势

**EN:** Specialised support for LLM weight, activation, and KV cache quantization. Cutting-edge algorithms: AWQ, GPTQ, Rotation, and SmoothQuant. Designed for AMD GPU (ROCm) inference workloads.

**CN:** 专为 LLM 权重、激活和 KV 缓存量化提供支持，集成 AWQ、GPTQ、Rotation、SmoothQuant 等前沿算法，专为 AMD GPU（ROCm）推理工作负载设计。

### 5-Step Workflow / 5 步工作流

**EN:** (1) Load model with transformers. (2) Prepare PyTorch DataLoader with calibration data. (3) Set quantization config (e.g., FP8 per-tensor on weights, activations, KV cache with AutoSmoothQuant). (4) Quantize and export to HuggingFace-compatible checkpoint. (5) Evaluate with `lm_eval` and vLLM.

**CN:** (1) 用 transformers 加载模型。(2) 用 PyTorch DataLoader 准备校准数据。(3) 设置量化配置（如对权重、激活、KV 缓存应用 FP8 逐张量量化，配合 AutoSmoothQuant）。(4) 量化并导出为 HuggingFace 兼容检查点。(5) 用 `lm_eval` 和 vLLM 评估。

### Installation / 安装

**EN:** `pip install amd-quark`. Requires vLLM and `lm-eval[api]>=0.4.12` for evaluation.

**CN:** `pip install amd-quark`。评估需要安装 vLLM 和 `lm-eval[api]>=0.4.12`。

### Calibration Data / 校准数据

**EN:** Uses PyTorch DataLoader (not a simple list); 512 samples batch size 1 from `mit-han-lab/pile-val-backup`, truncated to `MAX_SEQ_LEN=512` for the example.

**CN:** 使用 PyTorch DataLoader（非简单列表）；示例中从 `mit-han-lab/pile-val-backup` 取 512 条样本，批次大小 1，截断至 `MAX_SEQ_LEN=512`。

## Key Concepts / 关键概念

- **EN:** AMD Quark — AMD's quantization toolkit; the recommended solution for producing quantized models targeting AMD GPU inference via vLLM. **CN:** AMD Quark——AMD 的量化工具包，是生产面向 AMD GPU 推理的量化模型的推荐方案。
- **EN:** AutoSmoothQuant — Quark's automatic variant of SmoothQuant that selects per-layer smoothing strengths automatically. **CN:** AutoSmoothQuant——Quark 的自动版 SmoothQuant，自动选择每层的平滑强度。
- **EN:** KV cache quantization support — Quark can quantize weights, activations, and KV cache in a single workflow. **CN:** KV 缓存量化支持——Quark 可在单一工作流中同时量化权重、激活和 KV 缓存。
- **EN:** `quark.docs.amd.com` — official documentation for Quark config and calibration dataset guidance. **CN:** `quark.docs.amd.com`——Quark 配置和校准数据集指导的官方文档。
- **EN:** PyTorch DataLoader calibration — Quark requires calibration data in DataLoader format, unlike simpler list-based approaches in other tools. **CN:** PyTorch DataLoader 校准——Quark 要求校准数据采用 DataLoader 格式，与其他工具更简单的列表方式不同。
