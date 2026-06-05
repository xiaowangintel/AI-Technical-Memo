# yarn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/rotary_embedding/yarn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements linear projections and GEMM backend integration with rotary position embedding logic for the SGLang SRT runtime. It exposes symbols such as `yarn_find_correction_dim`, `yarn_find_correction_range`, `yarn_linear_ramp_mask`, and `yarn_get_mscale_simple` and organizes the supporting helpers, abstractions, and runtime decisions around them. / 该模块为 SGLang 的 SRT 运行时实现了线性投影与 GEMM 后端集成，并结合旋转位置编码逻辑。它提供了 `yarn_find_correction_dim`、`yarn_find_correction_range`、`yarn_linear_ramp_mask` 以及 `yarn_get_mscale_simple` 等符号，并围绕它们组织辅助函数、抽象层以及运行时决策。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Module imports and dependency wiring
```python
"""YaRNScalingRotaryEmbedding + YaRN helper functions."""

from __future__ import annotations

import math
from typing import Tuple

import torch

from sglang.srt.layers.rotary_embedding.base import RotaryEmbedding


# Inverse dim formula to find dim based on number of rotations
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `math`, `typing.Tuple`, `torch`, and `sglang.srt.layers.rotary_embedding.base.RotaryEmbedding`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`math`、`typing.Tuple`、`torch` 以及 `sglang.srt.layers.rotary_embedding.base.RotaryEmbedding`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 14-25: Function `yarn_find_correction_dim` and its core logic
```python
def yarn_find_correction_dim(
    num_rotations: int,
    dim: int,
    base: float = 10000,
    max_position_embeddings: int = 2048,
) -> float:
    return (dim * math.log(max_position_embeddings / (num_rotations * 2 * math.pi))) / (
        2 * math.log(base)
    )


# Find dim range bounds based on rotations
```
**EN:** This block defines `yarn_find_correction_dim` and contains the main logic for this step. It mainly invokes `math.log`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `yarn_find_correction_dim`，并承载这一阶段的核心逻辑。 它主要调用 `math.log`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 26-41: Function `yarn_find_correction_range` and its core logic
```python
def yarn_find_correction_range(
    low_rot: int,
    high_rot: int,
    dim: int,
    base: float = 10000,
    max_position_embeddings: int = 2048,
    truncate: bool = True,
) -> Tuple[int, int]:
    low = yarn_find_correction_dim(low_rot, dim, base, max_position_embeddings)
    high = yarn_find_correction_dim(high_rot, dim, base, max_position_embeddings)
    if truncate:
        low = math.floor(low)
        high = math.ceil(high)
    return max(low, 0), min(high, dim - 1)  # Clamp values just in case
```
**EN:** This block defines `yarn_find_correction_range` and contains the main logic for this step. It mainly invokes `yarn_find_correction_dim`, `math.floor`, `math.ceil`, `max`, and `min`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `low` and `high` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `yarn_find_correction_range`，并承载这一阶段的核心逻辑。 它主要调用 `yarn_find_correction_dim`、`math.floor`、`math.ceil`、`max` 以及 `min`，说明该流程会编排底层辅助函数或计算内核。 像 `low` 和 `high` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 42-52: Function `yarn_linear_ramp_mask` and its core logic
```python
def yarn_linear_ramp_mask(
    low: float, high: float, dim: int, dtype: torch.dtype, device: torch.device = None
) -> torch.Tensor:
    if low == high:
        high += 0.001  # Prevent singularity

    linear_func = (torch.arange(dim, dtype=dtype, device=device) - low) / (high - low)
    ramp_func = torch.clamp(linear_func, 0, 1)
    return ramp_func
```
**EN:** This block defines `yarn_linear_ramp_mask` and contains the main logic for this step. It mainly invokes `torch.clamp` and `torch.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `linear_func`, `ramp_func`, and `high` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `yarn_linear_ramp_mask`，并承载这一阶段的核心逻辑。 它主要调用 `torch.clamp` 和 `torch.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `linear_func`、`ramp_func` 以及 `high` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 53-58: Function `yarn_get_mscale_simple` and its core logic
```python
def yarn_get_mscale_simple(scale: float = 1) -> float:
    if scale <= 1:
        return 1.0
    return 0.1 * math.log(scale) + 1.0
```
**EN:** This block defines `yarn_get_mscale_simple` and contains the main logic for this step. It mainly invokes `math.log`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `yarn_get_mscale_simple`，并承载这一阶段的核心逻辑。 它主要调用 `math.log`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 59-64: Function `yarn_get_mscale` and its core logic
```python
def yarn_get_mscale(scale: float = 1, mscale: float = 1) -> float:
    if scale <= 1:
        return 1.0
    return 0.1 * mscale * math.log(scale) + 1.0
```
**EN:** This block defines `yarn_get_mscale` and contains the main logic for this step. It mainly invokes `math.log`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `yarn_get_mscale`，并承载这一阶段的核心逻辑。 它主要调用 `math.log`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 65-70: Class `YaRNScalingRotaryEmbedding` declaration and shared state
```python
class YaRNScalingRotaryEmbedding(RotaryEmbedding):
    """RotaryEmbedding extended with YaRN method.

    Credits to Peng et al. github.com/jquesnelle/yarn
    """
```
**EN:** This block introduces class `YaRNScalingRotaryEmbedding` and the state shared by its methods. It inherits from `RotaryEmbedding`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: RotaryEmbedding extended with YaRN method.
**CN:** 该代码块引入类 `YaRNScalingRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `RotaryEmbedding`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 71-98: `YaRNScalingRotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        scaling_factor: float,
        dtype: torch.dtype,
        *,
        extrapolation_factor: float = 1,
        attn_factor: float = 1,
        beta_fast: int = 32,
        beta_slow: int = 1,
        truncate: bool = True,
    ) -> None:
        self.scaling_factor = scaling_factor
        self.extrapolation_factor = extrapolation_factor
        self.attn_factor = attn_factor
        self.beta_fast = beta_fast
        self.beta_slow = beta_slow
        self.truncate = truncate
        # Get n-d magnitude scaling corrected for interpolation
        self.mscale = float(yarn_get_mscale_simple(self.scaling_factor) * attn_factor)
        super().__init__(
            head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype
        )
```
**EN:** This block defines `YaRNScalingRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `float`, `super.__init__`, and `yarn_get_mscale_simple`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.scaling_factor`, `self.extrapolation_factor`, `self.attn_factor`, `self.beta_fast`, and `self.beta_slow` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `YaRNScalingRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `float`、`super.__init__` 以及 `yarn_get_mscale_simple`，说明该流程会编排底层辅助函数或计算内核。 像 `self.scaling_factor`、`self.extrapolation_factor`、`self.attn_factor`、`self.beta_fast` 以及 `self.beta_slow` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 99-124: Internal helper `YaRNScalingRotaryEmbedding._compute_inv_freq`
```python
    def _compute_inv_freq(self, scaling_factor: float) -> torch.Tensor:
        pos_freqs = self.base ** (
            torch.arange(0, self.rotary_dim, 2, dtype=torch.float) / self.rotary_dim
        )
        inv_freq_extrapolation = 1.0 / pos_freqs
        inv_freq_interpolation = 1.0 / (scaling_factor * pos_freqs)

        low, high = yarn_find_correction_range(
            self.beta_fast,
            self.beta_slow,
            self.rotary_dim,
            self.base,
            self.max_position_embeddings,
            self.truncate,
        )
        # Get n-d rotational scaling corrected for extrapolation
        inv_freq_mask = (
            1
            - yarn_linear_ramp_mask(low, high, self.rotary_dim // 2, dtype=torch.float)
        ) * self.extrapolation_factor
        inv_freq = (
            inv_freq_interpolation * (1 - inv_freq_mask)
            + inv_freq_extrapolation * inv_freq_mask
        )
        return inv_freq
```
**EN:** This block defines `YaRNScalingRotaryEmbedding._compute_inv_freq` and contains the main logic for this step. It mainly invokes `yarn_find_correction_range`, `torch.arange`, and `yarn_linear_ramp_mask`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pos_freqs`, `inv_freq_extrapolation`, `inv_freq_interpolation`, `low`, and `high` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `YaRNScalingRotaryEmbedding._compute_inv_freq`，并承载这一阶段的核心逻辑。 它主要调用 `yarn_find_correction_range`、`torch.arange` 以及 `yarn_linear_ramp_mask`，说明该流程会编排底层辅助函数或计算内核。 像 `pos_freqs`、`inv_freq_extrapolation`、`inv_freq_interpolation`、`low` 以及 `high` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 125-134: Internal helper `YaRNScalingRotaryEmbedding._compute_cos_sin_cache`
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        inv_freq = self._compute_inv_freq(self.scaling_factor)
        t = torch.arange(
            self.max_position_embeddings * self.scaling_factor, dtype=torch.float32
        )
        freqs = torch.einsum("i,j -> ij", t, inv_freq)
        cos = freqs.cos() * self.mscale
        sin = freqs.sin() * self.mscale
        cache = torch.cat((cos, sin), dim=-1)
        return cache
```
**EN:** This block defines `YaRNScalingRotaryEmbedding._compute_cos_sin_cache` and contains the main logic for this step. It mainly invokes `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `torch.cat`, and `freqs.cos`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freq`, `t`, `freqs`, `cos`, and `sin` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `YaRNScalingRotaryEmbedding._compute_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `self._compute_inv_freq`、`torch.arange`、`torch.einsum`、`torch.cat` 以及 `freqs.cos`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freq`、`t`、`freqs`、`cos` 以及 `sin` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `yarn_find_correction_dim`, `yarn_find_correction_range`, `yarn_linear_ramp_mask`, `yarn_get_mscale_simple`, and `yarn_get_mscale`. / **主要符号**：核心入口包括 `yarn_find_correction_dim`、`yarn_find_correction_range`、`yarn_linear_ramp_mask`、`yarn_get_mscale_simple` 以及 `yarn_get_mscale`。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。
- **Embedding pipeline**: Describes how IDs or features are mapped into model-space tensors. / **嵌入流水线**：说明 ID 或特征如何映射到模型空间张量。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `math`, and `typing.Tuple` / **标准库**：`__future__.annotations`、`math` 以及 `typing.Tuple`
- **Third-party**: `torch` / **第三方依赖**：`torch`
- **Internal SGLang modules**: `sglang.srt.layers.rotary_embedding.base.RotaryEmbedding` / **SGLang 内部模块**：`sglang.srt.layers.rotary_embedding.base.RotaryEmbedding`
