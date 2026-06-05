# torchao.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/torchao.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document introduces TorchAO quantization support in vLLM. TorchAO is PyTorch's architecture optimisation library providing high-performance dtypes and kernels that compose with `torch.compile`, FSDP, and other native PyTorch features.

**CN:** 本文档介绍 vLLM 中的 TorchAO 量化支持。TorchAO 是 PyTorch 的架构优化库，提供与 `torch.compile`、FSDP 等原生 PyTorch 特性组合使用的高性能数据类型和内核。

### Installation / 安装

**EN:** Install latest nightly: `pip install --pre torchao>=10.0.0 --index-url https://download.pytorch.org/whl/nightly/cu126`. Choose the CUDA version matching your system.

**CN:** 安装最新 nightly：`pip install --pre torchao>=10.0.0 --index-url https://download.pytorch.org/whl/nightly/cu126`。选择与系统匹配的 CUDA 版本。

### Quantizing HuggingFace Models / 量化 HuggingFace 模型

**EN:** Use `TorchAoConfig` from transformers with a quantization config (e.g., `Int8WeightOnlyConfig()`) when loading a model, then push to HuggingFace Hub with `push_to_hub(..., safe_serialization=False)`. A HuggingFace Spaces UI is also available for no-code quantization.

**CN:** 加载模型时使用 transformers 的 `TorchAoConfig` 配合量化配置（如 `Int8WeightOnlyConfig()`），然后以 `push_to_hub(..., safe_serialization=False)` 推送到 HuggingFace Hub。也可使用 HuggingFace Spaces 的 UI 进行无代码量化。

### PyTorch Composability / PyTorch 可组合性

**EN:** A key advantage of TorchAO is that quantized models remain compatible with `torch.compile`, FSDP, and other PyTorch-native tools — making it suitable for both inference and training-adjacent workflows.

**CN:** TorchAO 的关键优势是量化模型与 `torch.compile`、FSDP 等 PyTorch 原生工具保持兼容，适用于推理和训练邻近工作流。

### `safe_serialization=False` Note / 注意事项

**EN:** TorchAO quantized models must be saved with `safe_serialization=False` when pushing to the HuggingFace Hub, as the default safetensors format may not support all TorchAO quantized weight types.

**CN:** 将 TorchAO 量化模型推送到 HuggingFace Hub 时必须使用 `safe_serialization=False`，因为默认的 safetensors 格式可能不支持所有 TorchAO 量化权重类型。

## Key Concepts / 关键概念

- **EN:** TorchAO — PyTorch's built-in architecture optimization library with composable quantization and sparsity primitives. **CN:** TorchAO——PyTorch 内置的架构优化库，提供可组合的量化和稀疏性原语。
- **EN:** `Int8WeightOnlyConfig` — a TorchAO quantization configuration for 8-bit weight-only quantization. **CN:** `Int8WeightOnlyConfig`——TorchAO 中用于 8 位仅权重量化的量化配置。
- **EN:** `TorchAoConfig` — the transformers integration class that wraps TorchAO quantization for use with `AutoModelForCausalLM.from_pretrained`. **CN:** `TorchAoConfig`——transformers 集成类，将 TorchAO 量化包装为可与 `AutoModelForCausalLM.from_pretrained` 配合使用的形式。
- **EN:** `torch.compile` compatibility — TorchAO's quantization is designed to be transparent to the compiler, enabling kernel fusion and other optimizations. **CN:** `torch.compile` 兼容性——TorchAO 的量化设计对编译器透明，支持内核融合等优化。
- **EN:** Nightly install required — TorchAO's vLLM integration requires the nightly build (>=10.0.0) rather than the stable release. **CN:** 需安装 Nightly 版本——TorchAO 的 vLLM 集成需要 nightly 版本（>=10.0.0）而非稳定版。
