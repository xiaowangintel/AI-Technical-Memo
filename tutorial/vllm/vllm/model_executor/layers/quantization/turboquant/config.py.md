# config.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/turboquant/config.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines configuration objects for quantization backends, schemes, and utilities. / 定义量化后端、方案与工具的配置对象。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""TurboQuant configuration."""
```
**EN:** This docstring gives the module author's high-level intent: TurboQuant configuration. It is useful for reading the rest of the file because later classes and helpers refine this stated purpose.
**CN:** 这个文档字符串给出了模块作者的高层意图：TurboQuant configuration. 在阅读后续类和辅助函数时，可以把它视为整个文件的总纲。

### Imports and module setup (lines 5-10)
```python
from __future__ import annotations

import logging
import math
from dataclasses import dataclass
from typing import TYPE_CHECKING
```
**EN:** This opening block pulls in external dependencies such as `__future__`, `logging`, `math`, `dataclasses`, `typing` and internal modules such as no internal imports. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `__future__`, `logging`, `math`, `dataclasses`, `typing`）以及内部模块（如 no internal imports）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 15-41)
```python
logger = logging.getLogger(__name__)

# Named TQ presets: each maps to frozen config parameters.
# key_quant_bits: 8 = FP8 keys, 3-4 = MSE (Lloyd-Max) quantized keys.
# value_quant_bits: 3-4 = uniform quantized values.
TQ_PRESETS: dict[str, dict] = {
    "turboquant_k8v4": {
        "key_quant_bits": 8,
        "value_quant_bits": 4,
        "norm_correction": False,
    },
    "turboquant_4bit_nc": {
        "key_quant_bits": 4,
        "value_quant_bits": 4,
        "norm_correction": True,
    },
    "turboquant_k3v4_nc": {
        "key_quant_bits": 3,
        "value_quant_bits": 4,
        "norm_correction": True,
    },
    "turboquant_3bit_nc": {
        "key_quant_bits": 3,
        "value_quant_bits": 3,
        "norm_correction": True,
    },
}
```
**EN:** This block defines module-level metadata or constants such as `logger`, `TQ_PRESETS`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`, `TQ_PRESETS`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `_get_full_attention_layer_indices` (lines 229-254)
```python
def _get_full_attention_layer_indices(model_config: ModelConfig) -> list[int]:
    """Global indices of full-attention layers in a hybrid model.

    Covers the conventions used across vLLM: ``layer_types`` (Qwen3.5/Next),
    ``layers_block_type`` (Jamba/Zamba2), ``attn_type_list`` (Minimax).
    """
    text_cfg = model_config.hf_text_config
    hf_cfg = model_config.hf_config

    layer_types = getattr(text_cfg, "layer_types", None)
    if layer_types is not None:
        return [
            i for i, t in enumerate(layer_types) if t in ("full_attention", "attention")
        ]

    layers_block_type = getattr(text_cfg, "layers_block_type", None)
    if layers_block_type is not None:
        return [
            i for i, t in enumerate(layers_block_type) if t in ("attention", "hybrid")
        ]

    attn_type_list = getattr(hf_cfg, "attn_type_list", None)
    if attn_type_list is not None:
        return [i for i, t in enumerate(attn_type_list) if t == 1]

    return []
```
**EN:** Defines function `_get_full_attention_layer_indices` with signature `_get_full_attention_layer_indices(model_config: ModelConfig) -> list[int]`. It mainly works with `model_config`; implements one step in the quantized-weight execution flow. The body uses branching, comprehensions. Key calls include `getattr`, `enumerate`.
**CN:** 定义函数 `_get_full_attention_layer_indices`，其签名为 `_get_full_attention_layer_indices(model_config: ModelConfig) -> list[int]`。它主要围绕 `model_config` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、推导式。关键调用包括 `getattr`, `enumerate`。

### Class `TurboQuantConfig` overview (lines 45-226)
```python
class TurboQuantConfig:
    """Configuration for TurboQuant KV-cache quantization.

    Applies Hadamard rotation followed by per-coordinate Lloyd-Max scalar
    quantization for keys, and uniform quantization for values.

    Historical note: this is the scalar case of the HIGGS quantization
    method (Malinovskii et al., "Pushing the Limits of Large Language Model
    Quantization via the Linearity Theorem", NAACL 2025; preprint
    arXiv:2411.17525): rotation + optimized grid + optional re-normalization,
    applied to KV cache compression. A first application of this approach to
    KV-cache compression is in "Cache Me If You Must: Adaptive Key-Value
    Quantization for Large Language Models" (Shutova et al., ICML 2025;
    preprint arXiv:2501.19392). Both these references pre-date the
    TurboQuant paper.

    QJL is intentionally omitted — community consensus (5+ independent
    groups) found it hurts attention quality by amplifying variance through
    softmax.

    Named presets (use via --kv-cache-dtype):
        turboquant_k8v4:   FP8 keys + 4-bit values, 2.6x, +1.17% PPL
        turboquant_4bit_nc: 4-bit MSE keys + 4-bit values + NC, 3.8x, +2.71%
        turboquant_k3v4_nc: 3-bit MSE keys + 4-bit values + NC, ~3.5x, +10.63%
        turboquant_3bit_nc: 3-bit MSE keys + 3-bit values + NC, 4.9x, +20.59%
```
**EN:** Defines class `TurboQuantConfig` with base classes no explicit base class and decorators `dataclass`. It acts as a configuration holder and backend selector and exposes 12 direct methods, with notable entries `key_fp8`, `mse_bits`, `key_mse_bits`, `centroid_bits`, `n_centroids`, `key_packed_size`. Its docstring says: Configuration for TurboQuant KV-cache quantization.
**CN:** 定义类 `TurboQuantConfig`，其基类为 无显式基类，装饰器为 `dataclass`。它在整体实现中充当配置容器与后端选择器，并直接暴露 12 个方法，较重要的包括 `key_fp8`, `mse_bits`, `key_mse_bits`, `centroid_bits`, `n_centroids`, `key_packed_size`。 文档字符串进一步说明了该类的定位。

### Method `TurboQuantConfig.mse_bits` (lines 94-103)
```python
    def mse_bits(self) -> int:
        """MSE quantizer bit-width (determines centroid count: 2^mse_bits).

        For MSE key modes, equals key_quant_bits.
        For FP8 key mode, falls back to value_quant_bits (centroids are still
        needed for continuation-prefill dequant and decode kernel params).
        """
        if self.key_fp8:
            return self.value_quant_bits
        return self.key_quant_bits
```
**EN:** Defines function `TurboQuantConfig.mse_bits` with signature `mse_bits(self) -> int`. It mainly works with object context only; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include no notable helper calls.
**CN:** 定义函数 `TurboQuantConfig.mse_bits`，其签名为 `mse_bits(self) -> int`。它主要围绕 仅依赖对象上下文 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 无显著辅助调用。

### Method `TurboQuantConfig.key_packed_size` (lines 122-136)
```python
    def key_packed_size(self) -> int:
        """Packed bytes for a single KEY vector.

        FP8 mode (key_quant_bits=8):
          head_dim bytes (1 byte per element, no overhead).

        TQ mode:
          - MSE indices: ceil(head_dim * key_mse_bits / 8) bytes
          - vec_norm:     2 bytes (float16)
        """
        if self.key_fp8:
            return self.head_dim  # 1 byte per element
        mse_bytes = math.ceil(self.head_dim * self.key_mse_bits / 8)
        norm_bytes = 2  # vec_norm fp16
        return mse_bytes + norm_bytes
```
**EN:** Defines function `TurboQuantConfig.key_packed_size` with signature `key_packed_size(self) -> int`. It mainly works with object context only; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include `math.ceil`.
**CN:** 定义函数 `TurboQuantConfig.key_packed_size`，其签名为 `key_packed_size(self) -> int`。它主要围绕 仅依赖对象上下文 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 `math.ceil`。

### Method `TurboQuantConfig.value_packed_size` (lines 144-150)
```python
    def value_packed_size(self) -> int:
        """Packed bytes for a single VALUE vector.

        Uniform quantization: ceil(head_dim * bits / 8) + 4 bytes (scale + zero fp16).
        """
        data_bytes = math.ceil(self.head_dim * self.value_quant_bits / 8)
        return data_bytes + 4  # +2 scale(fp16) +2 zero(fp16)
```
**EN:** Defines function `TurboQuantConfig.value_packed_size` with signature `value_packed_size(self) -> int`. It mainly works with object context only; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include `math.ceil`.
**CN:** 定义函数 `TurboQuantConfig.value_packed_size`，其签名为 `value_packed_size(self) -> int`。它主要围绕 仅依赖对象上下文 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `math.ceil`。

### Method `TurboQuantConfig.slot_size_aligned` (lines 161-168)
```python
    def slot_size_aligned(self) -> int:
        """Slot size rounded up to next even number.

        Even-number is required so effective_head_size = slot_size_aligned // 2
        is integral.
        """
        s = self.slot_size
        return s + (s % 2)  # round up to even
```
**EN:** Defines function `TurboQuantConfig.slot_size_aligned` with signature `slot_size_aligned(self) -> int`. It mainly works with object context only; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `TurboQuantConfig.slot_size_aligned`，其签名为 `slot_size_aligned(self) -> int`。它主要围绕 仅依赖对象上下文 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `TurboQuantConfig.get_boundary_skip_layers` (lines 171-206)
```python
    def get_boundary_skip_layers(
        model_config: ModelConfig,
        n: int = 2,
    ) -> list[str]:
        """Layer indices to skip TQ compression (boundary protection).

        For hybrid models (attention + Mamba/linear-attention), boundary
        protection is disabled — hybrids typically have only 8-12
        full-attention layers and a hard n=2 on each side would cover
        ~40 % of them.  The dense GSM8K baselines that motivate n=2
        don't apply to hybrids.

        For dense models, skips first N and last N attention layers.
        Empirically required for aggressive presets (k3v4_nc, 3bit_nc)
        — without it GSM8K drops ~30 points on Qwen3-4B.
        """
        if model_config.is_hybrid:
            attn_indices = _get_full_attention_layer_indices(model_config)
            if not attn_indices:
                raise NotImplementedError(
                    "TurboQuant KV cache requires identifiable "
                    "full-attention layers, but none were found in "
                    "the hybrid model config."
                )
            logger.info("TQ hybrid: full-attention layers %s", attn_indices)
            return []

        num_layers = model_config.hf_text_config.num_hidden_layers
        if n <= 0 or num_layers <= 0:
            return []
        n = min(n, num_layers // 2)  # don't skip more than half
        first = list(range(n))
        last = list(range(num_layers - n, num_layers))
        # Deduplicate (if num_layers <= 2*n)
        indices = sorted(set(first + last))
        return [str(i) for i in indices]
```
**EN:** Defines function `TurboQuantConfig.get_boundary_skip_layers` with signature `get_boundary_skip_layers(model_config: ModelConfig, n: int=2) -> list[str]`. It mainly works with `model_config`, `n`; returns a derived property or capability check. The body uses branching, comprehensions, validation/error handling. Key calls include `min`, `list`, `sorted`, `_get_full_attention_layer_indices`, `logger.info`, `range`.
**CN:** 定义函数 `TurboQuantConfig.get_boundary_skip_layers`，其签名为 `get_boundary_skip_layers(model_config: ModelConfig, n: int=2) -> list[str]`。它主要围绕 `model_config`, `n` 展开；返回派生属性或能力判断结果。函数体包含分支判断、推导式、校验或报错逻辑。关键调用包括 `min`, `list`, `sorted`, `_get_full_attention_layer_indices`, `logger.info`, `range`。

### Method `TurboQuantConfig.from_cache_dtype` (lines 209-226)
```python
    def from_cache_dtype(cache_dtype: str, head_dim: int) -> TurboQuantConfig:
        """Create config from a named preset.

        Valid presets: turboquant_k8v4, turboquant_4bit_nc, etc.
        """
        if cache_dtype not in TQ_PRESETS:
            valid = ", ".join(TQ_PRESETS.keys())
            raise ValueError(
                f"Unknown TurboQuant cache dtype: {cache_dtype!r}. "
                f"Valid presets: {valid}"
            )
        preset = TQ_PRESETS[cache_dtype]
        return TurboQuantConfig(
            head_dim=head_dim,
            key_quant_bits=preset["key_quant_bits"],
            value_quant_bits=preset["value_quant_bits"],
            norm_correction=preset["norm_correction"],
        )
```
**EN:** Defines function `TurboQuantConfig.from_cache_dtype` with signature `from_cache_dtype(cache_dtype: str, head_dim: int) -> TurboQuantConfig`. It mainly works with `cache_dtype`, `head_dim`; implements one step in the quantized-weight execution flow. The body uses branching, validation/error handling. Key calls include `TurboQuantConfig`, `', '.join`, `ValueError`, `TQ_PRESETS.keys`.
**CN:** 定义函数 `TurboQuantConfig.from_cache_dtype`，其签名为 `from_cache_dtype(cache_dtype: str, head_dim: int) -> TurboQuantConfig`。它主要围绕 `cache_dtype`, `head_dim` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `TurboQuantConfig`, `', '.join`, `ValueError`, `TQ_PRESETS.keys`。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level classes include `TurboQuantConfig`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `TurboQuantConfig`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `_get_full_attention_layer_indices` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `_get_full_attention_layer_indices` 为主要类提供了过程式入口。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `math`, `dataclasses`, `typing`
- **Internal / 内部**: None / 无
