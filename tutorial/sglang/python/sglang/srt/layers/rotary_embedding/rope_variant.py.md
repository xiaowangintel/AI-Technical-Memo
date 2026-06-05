# rope_variant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/rotary_embedding/rope_variant.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements rotary position embedding logic with embedding lookup and preprocessing for the SGLang SRT runtime. It exposes symbols such as `Phi3LongRoPEScaledRotaryEmbedding`, `FourierRotaryEmbedding`, `DeepseekScalingRotaryEmbedding`, and `Llama3RotaryEmbedding` and connects them to backend-specific paths such as `CUDA`, `NPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了旋转位置编码逻辑，并结合嵌入查找与预处理。它提供了 `Phi3LongRoPEScaledRotaryEmbedding`、`FourierRotaryEmbedding`、`DeepseekScalingRotaryEmbedding` 以及 `Llama3RotaryEmbedding` 等符号，并把这些符号连接到 `CUDA`、`NPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Imports, conditional backend setup, and runtime guards
```python
"""RoPE scaling variants: Phi3LongRoPE, FourierRoPE, DeepseekScaling, Llama3,
Llama4Vision, DynamicNTK, DynamicNTKAlpha, DualChunkRotaryEmbedding."""

from __future__ import annotations

import math
from typing import List, Optional, Tuple, Union

import torch
import torch.nn as nn
import torch.nn.functional as F

from sglang.srt.layers.rotary_embedding.base import RotaryEmbedding
from sglang.srt.layers.rotary_embedding.utils import (
    apply_rotary_pos_emb_native,
    rotate_gptj,
    rotate_neox,
)
from sglang.srt.layers.rotary_embedding.yarn import (
    yarn_find_correction_range,
    yarn_get_mscale,
    yarn_linear_ramp_mask,
)
from sglang.srt.layers.utils import MultiPlatformOp
from sglang.srt.utils import cpu_has_amx_support, get_device, is_cuda, is_hip, is_npu

_is_cuda = is_cuda()
_is_hip = is_hip()
_is_npu = is_npu()
_is_cpu_amx_available = cpu_has_amx_support()

if _is_npu:
    import torch_npu
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `math`, `typing.List`, `typing.Optional`, `typing.Tuple`, and `typing.Union`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_cuda`, `_is_hip`, `_is_npu`, and `_is_cpu_amx_available` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`math`、`typing.List`、`typing.Optional`、`typing.Tuple` 以及 `typing.Union`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_cuda`、`_is_hip`、`_is_npu` 以及 `_is_cpu_amx_available` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 36-38: Class `Phi3LongRoPEScaledRotaryEmbedding` declaration and shared state
```python
class Phi3LongRoPEScaledRotaryEmbedding(nn.Module):
    """Phi3 family of models scaled rotary embedding."""
```
**EN:** This block introduces class `Phi3LongRoPEScaledRotaryEmbedding` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Phi3 family of models scaled rotary embedding.
**CN:** 该代码块引入类 `Phi3LongRoPEScaledRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 39-101: `Phi3LongRoPEScaledRotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        original_max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        dtype: torch.dtype,
        short_factor: List[float],
        long_factor: List[float],
        short_mscale: Optional[float] = None,
        long_mscale: Optional[float] = None,
    ):
        super().__init__()

        if is_neox_style is False:
            raise ValueError(
                "`Phi3LongRoPEScaledRotaryEmbedding` only supports neox_style."
            )

        self.rotary_dim = rotary_dim
        self.head_size = head_size
        self.max_position_embeddings = max_position_embeddings
        self.original_max_position_embeddings = original_max_position_embeddings
        self.base = base
        self.short_factor = short_factor
        self.long_factor = long_factor

        scale = self.max_position_embeddings / self.original_max_position_embeddings
        if scale <= 1.0:
            scaling_factor = 1.0
        else:
            scaling_factor = math.sqrt(
                1 + math.log(scale) / math.log(self.original_max_position_embeddings)
            )
        if short_mscale is None:
            short_mscale = scaling_factor
        if long_mscale is None:
            long_mscale = scaling_factor

        self.short_mscale = short_mscale
        self.long_mscale = long_mscale

        short_cache = self._compute_cos_sin_cache(
            original_max_position_embeddings, short_factor, short_mscale
        )
        short_cache = short_cache.to(dtype)
        self.register_buffer("short_cos_sin_cache", short_cache, persistent=False)

        long_cache = self._compute_cos_sin_cache(
            max_position_embeddings, long_factor, long_mscale
        )
        long_cache = long_cache.to(dtype)
        self.register_buffer("long_cos_sin_cache", long_cache, persistent=False)

        long_short_cache = torch.cat(
            [self.short_cos_sin_cache, self.long_cos_sin_cache], dim=0
        )
        self.register_buffer(
            "long_short_cos_sin_cache", long_short_cache, persistent=False
        )
```
**EN:** This block defines `Phi3LongRoPEScaledRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `self._compute_cos_sin_cache`, `short_cache.to`, `self.register_buffer`, and `long_cache.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.rotary_dim`, `self.head_size`, `self.max_position_embeddings`, `self.original_max_position_embeddings`, and `self.base` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Phi3LongRoPEScaledRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`self._compute_cos_sin_cache`、`short_cache.to`、`self.register_buffer` 以及 `long_cache.to`，说明该流程会编排底层辅助函数或计算内核。 像 `self.rotary_dim`、`self.head_size`、`self.max_position_embeddings`、`self.original_max_position_embeddings` 以及 `self.base` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 102-115: Internal helper `Phi3LongRoPEScaledRotaryEmbedding._compute_inv_freq`
```python
    def _compute_inv_freq(self, rescale_factors: List[float]) -> torch.Tensor:
        rescale_factors = torch.tensor(rescale_factors, dtype=torch.float32)
        inv_freq = 1.0 / (
            rescale_factors
            * (
                self.base
                ** (
                    torch.arange(0, self.rotary_dim, 2, dtype=torch.float)
                    / self.rotary_dim
                )
            )
        )
        return inv_freq
```
**EN:** This block defines `Phi3LongRoPEScaledRotaryEmbedding._compute_inv_freq` and contains the main logic for this step. It mainly invokes `torch.tensor` and `torch.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `rescale_factors` and `inv_freq` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Phi3LongRoPEScaledRotaryEmbedding._compute_inv_freq`，并承载这一阶段的核心逻辑。 它主要调用 `torch.tensor` 和 `torch.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `rescale_factors` 和 `inv_freq` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 116-129: Internal helper `Phi3LongRoPEScaledRotaryEmbedding._compute_cos_sin_cache`
```python
    def _compute_cos_sin_cache(
        self,
        max_position_embeddings: int,
        rescale_factors: List[float],
        mscale: float,
    ) -> torch.Tensor:
        inv_freq = self._compute_inv_freq(rescale_factors)
        t = torch.arange(max_position_embeddings, dtype=torch.float)
        freqs = torch.einsum("i,j -> ij", t, inv_freq)
        cos = freqs.cos() * mscale
        sin = freqs.sin() * mscale
        cache = torch.cat((cos, sin), dim=-1)
        return cache
```
**EN:** This block defines `Phi3LongRoPEScaledRotaryEmbedding._compute_cos_sin_cache` and contains the main logic for this step. It mainly invokes `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `torch.cat`, and `freqs.cos`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freq`, `t`, `freqs`, `cos`, and `sin` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Phi3LongRoPEScaledRotaryEmbedding._compute_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `self._compute_inv_freq`、`torch.arange`、`torch.einsum`、`torch.cat` 以及 `freqs.cos`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freq`、`t`、`freqs`、`cos` 以及 `sin` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 130-171: `Phi3LongRoPEScaledRotaryEmbedding.forward` main forward path
```python
    def forward(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        query = query.unflatten(1, (-1, self.head_size))
        key = key.unflatten(1, (-1, self.head_size))

        k = self.original_max_position_embeddings
        long_prompt_offset = (
            torch.any(positions > k).float() * torch.full_like(positions, k)
        ).long()
        idx = (
            torch.add(positions, long_prompt_offset)
            if long_prompt_offset is not None
            else positions
        )
        self.long_short_cos_sin_cache: torch.Tensor = self.long_short_cos_sin_cache.to(
            idx.device
        )
        idx = torch.add(idx, offsets) if offsets is not None else idx
        cos_sin = torch.index_select(self.long_short_cos_sin_cache, 0, idx)

        cos, sin = cos_sin.chunk(2, dim=-1)
        cos = cos.repeat(1, 2).unsqueeze(-2)
        sin = sin.repeat(1, 2).unsqueeze(-2)

        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = query_rot * cos + rotate_neox(query_rot) * sin
        query = torch.cat((query_rot, query_pass), dim=-1)

        key_rot = key[..., : self.rotary_dim]
        key_pass = key[..., self.rotary_dim :]
        key_rot = key_rot * cos + rotate_neox(key_rot) * sin
        key = torch.cat((key_rot, key_pass), dim=-1)

        return query.flatten(-2), key.flatten(-2)
```
**EN:** This block defines `Phi3LongRoPEScaledRotaryEmbedding.forward` and contains the main logic for this step. It mainly invokes `query.unflatten`, `key.unflatten`, `long`, `self.long_short_cos_sin_cache.to`, and `torch.index_select`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `query`, `key`, `k`, `long_prompt_offset`, and `idx` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Phi3LongRoPEScaledRotaryEmbedding.forward`，并承载这一阶段的核心逻辑。 它主要调用 `query.unflatten`、`key.unflatten`、`long`、`self.long_short_cos_sin_cache.to` 以及 `torch.index_select`，说明该流程会编排底层辅助函数或计算内核。 像 `query`、`key`、`k`、`long_prompt_offset` 以及 `idx` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 172-174: Class `FourierRotaryEmbedding` declaration and shared state
```python
class FourierRotaryEmbedding(nn.Module):
    """Fourier RotaryEmbedding extended."""
```
**EN:** This block introduces class `FourierRotaryEmbedding` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Fourier RotaryEmbedding extended.
**CN:** 该代码块引入类 `FourierRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 175-227: `FourierRotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        dtype: torch.dtype,
        num_kv_heads: int,
        *,
        fope_init_factor: float = 0.1,
        fope_sep_head: bool = True,
        num_inv_freq: int = None,
        device: Optional[str] = "cuda",
    ) -> None:
        self.fope_init_factor = fope_init_factor
        self.fope_sep_head = fope_sep_head
        self.num_inv_freq = num_inv_freq
        self.num_kv_heads = num_kv_heads
        self.device = device

        super().__init__()
        self.head_size = head_size
        self.rotary_dim = rotary_dim
        self.max_position_embeddings = max_position_embeddings
        self.base = base
        self.is_neox_style = is_neox_style
        self.dtype = dtype

        self.inv_freq: torch.Tensor
        self.register_buffer(
            "inv_freq", self._compute_inv_freq(self.base), persistent=False
        )
        self.input_dim = self.inv_freq.shape[-1]
        self.output_dim = self.inv_freq.shape[-1]
        self.cos_coef = nn.Parameter(
            torch.empty(
                self.num_kv_heads, self.input_dim, self.output_dim, dtype=torch.float32
            ),
            requires_grad=False,
        )
        self.sin_coef = nn.Parameter(
            torch.empty(
                self.num_kv_heads, self.input_dim, self.output_dim, dtype=torch.float32
            ),
            requires_grad=False,
        )
        self.cos_sin_cache: torch.Tensor
        self.register_buffer(
            "cos_sin_cache", self._compute_cos_sin_cache(), persistent=False
        )
        self.update_buffer = False
```
**EN:** This block defines `FourierRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `self.register_buffer`, `nn.Parameter`, `self._compute_inv_freq`, and `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.fope_init_factor`, `self.fope_sep_head`, `self.num_inv_freq`, `self.num_kv_heads`, and `self.device` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `FourierRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`self.register_buffer`、`nn.Parameter`、`self._compute_inv_freq` 以及 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 像 `self.fope_init_factor`、`self.fope_sep_head`、`self.num_inv_freq`、`self.num_kv_heads` 以及 `self.device` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 228-250: Internal helper `FourierRotaryEmbedding._compute_inv_freq`
```python
    def _compute_inv_freq(self, base: Union[int, float]) -> torch.Tensor:
        inv_freq = 1.0 / (
            base
            ** (
                torch.arange(0, self.rotary_dim, 2, dtype=torch.int64).to(
                    device=self.device, dtype=torch.float
                )
                / self.rotary_dim
            )
        )
        assert (
            inv_freq[:-1] > inv_freq[1:]
        ).all(), "Expected inv_freq to be in decreasing order"
        inv_freq_idx_selected = torch.ones_like(inv_freq, dtype=torch.bool)
        if self.num_inv_freq is not None:
            inv_freq_idx_selected[self.num_inv_freq :] = False
        else:
            inv_freq_idx_selected = inv_freq > (
                2.0 * torch.pi / self.max_position_embeddings
            )
        inv_freq = inv_freq[inv_freq_idx_selected]
        return inv_freq
```
**EN:** This block defines `FourierRotaryEmbedding._compute_inv_freq` and contains the main logic for this step. It mainly invokes `all`, `torch.ones_like`, `torch.arange.to`, and `torch.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freq` and `inv_freq_idx_selected` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FourierRotaryEmbedding._compute_inv_freq`，并承载这一阶段的核心逻辑。 它主要调用 `all`、`torch.ones_like`、`torch.arange.to` 以及 `torch.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freq` 和 `inv_freq_idx_selected` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 251-284: Internal helper `FourierRotaryEmbedding._compute_cos_sin_cache`
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        t = torch.arange(
            self.max_position_embeddings, dtype=torch.float, device=self.device
        )
        freqs = torch.einsum("i,j -> ij", t, self.inv_freq)
        if self.fope_sep_head:
            pos_cos = freqs.cos().unsqueeze(0).expand(self.num_kv_heads, -1, -1)
            pos_sin = freqs.sin().unsqueeze(0).expand(self.num_kv_heads, -1, -1)
        else:
            pos_cos = freqs.cos()
            pos_sin = freqs.sin()
        if self.fope_sep_head:
            sin = torch.einsum("htD, hDd -> thd", pos_sin, self.sin_coef.float())
            cos = torch.einsum("htD, hDd -> thd", pos_cos, self.cos_coef.float())
        else:
            sin = torch.einsum("tD, Dd -> td", pos_sin, self.sin_coef.float())
            cos = torch.einsum("tD, Dd -> td", pos_cos, self.cos_coef.float())
        sin = F.pad(
            input=sin,
            pad=(0, self.head_size // 2 - sin.size(-1)),
            mode="constant",
            value=1,
        )
        cos = F.pad(
            input=cos,
            pad=(0, self.head_size // 2 - cos.size(-1)),
            mode="constant",
            value=1,
        )
        sin = torch.cat((sin, sin), dim=-1)
        cos = torch.cat((cos, cos), dim=-1)
        cache = torch.cat((cos, sin), dim=-1)
        return cache
```
**EN:** This block defines `FourierRotaryEmbedding._compute_cos_sin_cache` and contains the main logic for this step. It mainly invokes `torch.arange`, `torch.einsum`, `F.pad`, `torch.cat`, and `freqs.cos.unsqueeze.expand`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `t`, `freqs`, `sin`, `cos`, and `cache` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FourierRotaryEmbedding._compute_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `torch.arange`、`torch.einsum`、`F.pad`、`torch.cat` 以及 `freqs.cos.unsqueeze.expand`，说明该流程会编排底层辅助函数或计算内核。 像 `t`、`freqs`、`sin`、`cos` 以及 `cache` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 285-324: `FourierRotaryEmbedding.forward` main forward path
```python
    def forward(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
        **kwargs,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        if not self.update_buffer:
            self.cos_sin_cache = self._compute_cos_sin_cache()
            self.update_buffer = True
        query = query.unflatten(-1, (-1, self.head_size))
        key = key.unflatten(-1, (-1, self.head_size))
        positions_with_offsets = (
            torch.add(positions, offsets) if offsets is not None else positions
        )
        cos_sin = torch.index_select(self.cos_sin_cache, 0, positions_with_offsets).to(
            dtype=query.dtype
        )
        cos, sin = cos_sin.chunk(2, dim=-1)
        assert (
            query.dim() == key.dim() == 3
        ), "Expected query key (seq_len, heads, head_dim)"
        assert cos.dim() <= 3 and sin.dim() <= 3
        need_reshape = False
        if cos.dim() == 3:
            need_reshape = True
            query_shape = query.shape
            key_shape = key.shape
            cos = cos.flatten(0, 1)
            sin = sin.flatten(0, 1)
            seq_len = cos.size(0)
            query = query.reshape(seq_len, -1, query.size(-1))
            key = key.reshape(seq_len, -1, key.size(-1))
        query, key = apply_rotary_pos_emb_native(query, key, cos, sin)
        if need_reshape:
            query = query.reshape(query_shape)
            key = key.reshape(key_shape)
        return query.flatten(-2), key.flatten(-2)
```
**EN:** This block defines `FourierRotaryEmbedding.forward` and contains the main logic for this step. It mainly invokes `query.unflatten`, `key.unflatten`, `torch.index_select.to`, `cos_sin.chunk`, and `apply_rotary_pos_emb_native`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `query`, `key`, `positions_with_offsets`, `cos_sin`, and `cos` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FourierRotaryEmbedding.forward`，并承载这一阶段的核心逻辑。 它主要调用 `query.unflatten`、`key.unflatten`、`torch.index_select.to`、`cos_sin.chunk` 以及 `apply_rotary_pos_emb_native`，说明该流程会编排底层辅助函数或计算内核。 像 `query`、`key`、`positions_with_offsets`、`cos_sin` 以及 `cos` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 325-333: Function `FourierRotaryEmbedding.extra_repr` and its core logic
```python
    def extra_repr(self) -> str:
        s = f"head_size={self.head_size}, rotary_dim={self.rotary_dim}"
        s += f", max_position_embeddings={self.max_position_embeddings}"
        s += f", base={self.base}, is_neox_style={self.is_neox_style}"
        s += f", fope_init_factor={self.fope_init_factor}, fope_sep_head={self.fope_sep_head}"
        s += f", num_inv_freq={self.num_inv_freq}, num_kv_heads={self.num_kv_heads}"
        return s
```
**EN:** This block defines `FourierRotaryEmbedding.extra_repr` and contains the main logic for this step. Intermediate names such as `s` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FourierRotaryEmbedding.extra_repr`，并承载这一阶段的核心逻辑。 像 `s` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 334-339: Class `DeepseekScalingRotaryEmbedding` declaration and shared state
```python
class DeepseekScalingRotaryEmbedding(RotaryEmbedding):
    """RotaryEmbedding extended with YaRN method.

    Credits to Peng et al. github.com/jquesnelle/yarn
    """
```
**EN:** This block introduces class `DeepseekScalingRotaryEmbedding` and the state shared by its methods. It inherits from `RotaryEmbedding`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: RotaryEmbedding extended with YaRN method.
**CN:** 该代码块引入类 `DeepseekScalingRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `RotaryEmbedding`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 340-378: `DeepseekScalingRotaryEmbedding` initialization and state setup
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
        mscale: float = 1,
        mscale_all_dim: float = 0,
        device: Optional[str] = None,
    ) -> None:
        self.scaling_factor = scaling_factor
        self.extrapolation_factor = extrapolation_factor
        self.attn_factor = attn_factor
        self.beta_fast = beta_fast
        self.beta_slow = beta_slow
        self.mscale = float(
            yarn_get_mscale(self.scaling_factor, float(mscale))
            / yarn_get_mscale(self.scaling_factor, float(mscale_all_dim))
            * attn_factor
        )
        self.cos_cached_total = None
        self.sin_cached_total = None
        self.cos_cached = None
        self.sin_cached = None
        self.device = device if device is not None else get_device()
        super().__init__(
            head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype
        )
        if _is_hip:
            self._forward_method = self.forward_native
```
**EN:** This block defines `DeepseekScalingRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `float`, `super.__init__`, `get_device`, and `yarn_get_mscale`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.scaling_factor`, `self.extrapolation_factor`, `self.attn_factor`, `self.beta_fast`, and `self.beta_slow` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DeepseekScalingRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `float`、`super.__init__`、`get_device` 以及 `yarn_get_mscale`，说明该流程会编排底层辅助函数或计算内核。 像 `self.scaling_factor`、`self.extrapolation_factor`、`self.attn_factor`、`self.beta_fast` 以及 `self.beta_slow` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 379-404: Internal helper `DeepseekScalingRotaryEmbedding._compute_inv_freq`
```python
    def _compute_inv_freq(self, scaling_factor: float) -> torch.Tensor:
        pos_freqs = self.base ** (
            torch.arange(0, self.rotary_dim, 2, dtype=torch.float, device=self.device)
            / self.rotary_dim
        )
        inv_freq_extrapolation = 1.0 / pos_freqs
        inv_freq_interpolation = 1.0 / (scaling_factor * pos_freqs)
        low, high = yarn_find_correction_range(
            self.beta_fast,
            self.beta_slow,
            self.rotary_dim,
            self.base,
            self.max_position_embeddings,
        )
        inv_freq_mask = (
            1
            - yarn_linear_ramp_mask(
                low, high, self.rotary_dim // 2, dtype=torch.float, device=self.device
            )
        ) * self.extrapolation_factor
        inv_freq = (
            inv_freq_interpolation * (1 - inv_freq_mask)
            + inv_freq_extrapolation * inv_freq_mask
        )
        return inv_freq
```
**EN:** This block defines `DeepseekScalingRotaryEmbedding._compute_inv_freq` and contains the main logic for this step. It mainly invokes `yarn_find_correction_range`, `torch.arange`, and `yarn_linear_ramp_mask`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pos_freqs`, `inv_freq_extrapolation`, `inv_freq_interpolation`, `low`, and `high` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepseekScalingRotaryEmbedding._compute_inv_freq`，并承载这一阶段的核心逻辑。 它主要调用 `yarn_find_correction_range`、`torch.arange` 以及 `yarn_linear_ramp_mask`，说明该流程会编排底层辅助函数或计算内核。 像 `pos_freqs`、`inv_freq_extrapolation`、`inv_freq_interpolation`、`low` 以及 `high` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 405-421: Internal helper `DeepseekScalingRotaryEmbedding._compute_cos_sin_cache`
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        inv_freq = self._compute_inv_freq(self.scaling_factor)
        t = torch.arange(
            self.max_position_embeddings * self.scaling_factor,
            device=self.device,
            dtype=torch.float32,
        )
        freqs = torch.einsum("i,j -> ij", t, inv_freq)
        cos = freqs.cos() * self.mscale
        sin = freqs.sin() * self.mscale
        cache = torch.cat((cos, sin), dim=-1)
        if _is_npu:
            emb = torch.cat((freqs, freqs), dim=-1)
            self.cos_cached_total = torch.cos(emb) * self.mscale
            self.sin_cached_total = torch.sin(emb) * self.mscale
        return cache
```
**EN:** This block defines `DeepseekScalingRotaryEmbedding._compute_cos_sin_cache` and contains the main logic for this step. It mainly invokes `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `torch.cat`, and `freqs.cos`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freq`, `t`, `freqs`, `cos`, and `sin` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepseekScalingRotaryEmbedding._compute_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `self._compute_inv_freq`、`torch.arange`、`torch.einsum`、`torch.cat` 以及 `freqs.cos`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freq`、`t`、`freqs`、`cos` 以及 `sin` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 422-424: `DeepseekScalingRotaryEmbedding.get_cos_cached_total` getter for cos cached total
```python
    def get_cos_cached_total(self):
        return self.cos_cached_total
```
**EN:** This block defines `DeepseekScalingRotaryEmbedding.get_cos_cached_total` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepseekScalingRotaryEmbedding.get_cos_cached_total`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 425-427: `DeepseekScalingRotaryEmbedding.get_sin_cached_total` getter for sin cached total
```python
    def get_sin_cached_total(self):
        return self.sin_cached_total
```
**EN:** This block defines `DeepseekScalingRotaryEmbedding.get_sin_cached_total` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepseekScalingRotaryEmbedding.get_sin_cached_total`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 428-450: `DeepseekScalingRotaryEmbedding.get_cos_sin_cache` getter for cos sin cache
```python
    def get_cos_sin_cache(
        self, positions, dtype, offsets: Optional[torch.Tensor] = None
    ):
        self.cos_cached = (
            self.cos_cached_total[
                torch.add(positions, offsets) if offsets is not None else positions
            ]
            .unsqueeze(-2)
            .unsqueeze(-2)
            .to(dtype)
        )
        self.sin_cached = (
            self.sin_cached_total[
                torch.add(positions, offsets) if offsets is not None else positions
            ]
            .unsqueeze(-2)
            .unsqueeze(-2)
            .to(dtype)
        )
        cos = self.cos_cached.to(positions.device)
        sin = self.sin_cached.to(positions.device)
        return cos, sin
```
**EN:** This block defines `DeepseekScalingRotaryEmbedding.get_cos_sin_cache` and contains the main logic for this step. It mainly invokes `self.cos_cached_total.unsqueeze.unsqueeze.to`, `self.sin_cached_total.unsqueeze.unsqueeze.to`, `self.cos_cached.to`, `self.sin_cached.to`, and `self.cos_cached_total.unsqueeze.unsqueeze`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.cos_cached`, `self.sin_cached`, `cos`, and `sin` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepseekScalingRotaryEmbedding.get_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `self.cos_cached_total.unsqueeze.unsqueeze.to`、`self.sin_cached_total.unsqueeze.unsqueeze.to`、`self.cos_cached.to`、`self.sin_cached.to` 以及 `self.cos_cached_total.unsqueeze.unsqueeze`，说明该流程会编排底层辅助函数或计算内核。 像 `self.cos_cached`、`self.sin_cached`、`cos` 以及 `sin` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 451-485: `DeepseekScalingRotaryEmbedding.forward_native` NATIVE execution path
```python
    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        """PyTorch-native implementation equivalent to forward()."""
        dtype = query.dtype
        query_rot = query[..., : self.rotary_dim]
        key_rot = key[..., : self.rotary_dim]
        if self.rotary_dim < self.head_size:
            query_pass = query[..., self.rotary_dim :]
            key_pass = key[..., self.rotary_dim :]
        cos_sin = self.cos_sin_cache[
            torch.add(positions, offsets) if offsets is not None else positions
        ]
        cos, sin = cos_sin.chunk(2, dim=-1)
        if self.is_neox_style:
            cos = cos.repeat(1, 1, 2).unsqueeze(-2)
            sin = sin.repeat(1, 1, 2).unsqueeze(-2)
        else:
            cos = cos.repeat_interleave(2, dim=-1).unsqueeze(-2)
            sin = sin.repeat_interleave(2, dim=-1).unsqueeze(-2)
        rotate_fn = rotate_neox if self.is_neox_style else rotate_gptj
        query_rot = query_rot * cos + rotate_fn(query_rot) * sin
        key_rot = key_rot * cos + rotate_fn(key_rot) * sin
        if self.rotary_dim < self.head_size:
            query = torch.cat((query_rot, query_pass), dim=-1)
            key = torch.cat((key_rot, key_pass), dim=-1)
        else:
            query = query_rot
            key = key_rot
        return query.to(dtype), key.to(dtype)
```
**EN:** This block defines `DeepseekScalingRotaryEmbedding.forward_native` and contains the main logic for this step. It mainly invokes `cos_sin.chunk`, `cos.repeat.unsqueeze`, `sin.repeat.unsqueeze`, `cos.repeat_interleave.unsqueeze`, and `sin.repeat_interleave.unsqueeze`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `dtype`, `query_rot`, `key_rot`, `cos_sin`, and `cos` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepseekScalingRotaryEmbedding.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `cos_sin.chunk`、`cos.repeat.unsqueeze`、`sin.repeat.unsqueeze`、`cos.repeat_interleave.unsqueeze` 以及 `sin.repeat_interleave.unsqueeze`，说明该流程会编排底层辅助函数或计算内核。 像 `dtype`、`query_rot`、`key_rot`、`cos_sin` 以及 `cos` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 486-520: `DeepseekScalingRotaryEmbedding.forward_npu` NPU execution path
```python
    def forward_npu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        num_tokens, num_q_heads, _ = query.shape
        num_k_heads = key.shape[1]
        cos, sin = self.get_cos_sin_cache(positions, query.dtype, offsets)
        query_rot = query[..., : self.rotary_dim]
        key_rot = key[..., : self.rotary_dim]
        if self.rotary_dim < self.head_size:
            query_pass = query[..., self.rotary_dim :]
            key_pass = key[..., self.rotary_dim :]
        query_rot = torch_npu.npu_interleave_rope(
            query_rot.reshape(num_tokens, num_q_heads, 1, self.rotary_dim),
            cos,
            sin,
        )
        key_rot = torch_npu.npu_interleave_rope(
            key_rot.reshape(num_tokens, num_k_heads, 1, self.rotary_dim),
            cos,
            sin,
        )
        query_rot = query_rot.reshape(num_tokens, -1, self.rotary_dim)
        key_rot = key_rot.reshape(num_tokens, -1, self.rotary_dim)
        if self.rotary_dim < self.head_size:
            query = torch.cat((query_rot, query_pass), dim=-1)
            key = torch.cat((key_rot, key_pass), dim=-1)
        else:
            query = query_rot
            key = key_rot
        return query, key
```
**EN:** This block defines `DeepseekScalingRotaryEmbedding.forward_npu` and contains the main logic for this step. It mainly invokes `self.get_cos_sin_cache`, `torch_npu.npu_interleave_rope`, `query_rot.reshape`, `key_rot.reshape`, and `torch.cat`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens`, `num_q_heads`, `_`, `num_k_heads`, and `cos` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepseekScalingRotaryEmbedding.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `self.get_cos_sin_cache`、`torch_npu.npu_interleave_rope`、`query_rot.reshape`、`key_rot.reshape` 以及 `torch.cat`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens`、`num_q_heads`、`_`、`num_k_heads` 以及 `cos` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 521-536: `DeepseekScalingRotaryEmbedding.forward_cpu` CPU execution path
```python
    def forward_cpu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        positions = torch.add(positions, offsets) if offsets is not None else positions
        if _is_cpu_amx_available:
            return torch.ops.sgl_kernel.rotary_embedding_cpu(
                positions, query, key, self.head_size, self.cos_sin_cache, False
            )
        else:
            return self.forward_native(positions, query, key, offsets)
```
**EN:** This block defines `DeepseekScalingRotaryEmbedding.forward_cpu` and contains the main logic for this step. It mainly invokes `torch.add`, `torch.ops.sgl_kernel.rotary_embedding_cpu`, and `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `positions` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepseekScalingRotaryEmbedding.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.add`、`torch.ops.sgl_kernel.rotary_embedding_cpu` 以及 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 像 `positions` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 537-538: Class `Llama3RotaryEmbedding` declaration and shared state
```python
class Llama3RotaryEmbedding(RotaryEmbedding):
```
**EN:** This block introduces class `Llama3RotaryEmbedding` and the state shared by its methods. It inherits from `RotaryEmbedding`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `Llama3RotaryEmbedding`，并定义其方法共享的状态。 它继承自 `RotaryEmbedding`，说明了它在 SRT 层栈中的接入方式。

### Lines 539-559: `Llama3RotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        dtype: torch.dtype,
        scaling_factor: float,
        low_freq_factor: float,
        high_freq_factor: float,
        orig_max_position: int,
    ) -> None:
        self.scaling_factor = scaling_factor
        self.low_freq_factor = low_freq_factor
        self.high_freq_factor = high_freq_factor
        self.orig_max_position = orig_max_position
        super().__init__(
            head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype
        )
```
**EN:** This block defines `Llama3RotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.scaling_factor`, `self.low_freq_factor`, `self.high_freq_factor`, and `self.orig_max_position` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Llama3RotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self.scaling_factor`、`self.low_freq_factor`、`self.high_freq_factor` 以及 `self.orig_max_position` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 560-582: Internal helper `Llama3RotaryEmbedding._compute_inv_freq`
```python
    def _compute_inv_freq(self, base: Union[int, float]) -> torch.Tensor:
        inv_freqs = super()._compute_inv_freq(base)
        low_freq_wavelen = self.orig_max_position / self.low_freq_factor
        high_freq_wavelen = self.orig_max_position / self.high_freq_factor
        wave_len = 2 * math.pi / inv_freqs
        if self.low_freq_factor != self.high_freq_factor:
            smooth = (self.orig_max_position / wave_len - self.low_freq_factor) / (
                self.high_freq_factor - self.low_freq_factor
            )
        else:
            smooth = 0
        new_freqs = torch.where(
            wave_len < high_freq_wavelen,
            inv_freqs,
            torch.where(
                wave_len > low_freq_wavelen,
                inv_freqs / self.scaling_factor,
                (1 - smooth) * inv_freqs / self.scaling_factor + smooth * inv_freqs,
            ),
        )
        return new_freqs
```
**EN:** This block defines `Llama3RotaryEmbedding._compute_inv_freq` and contains the main logic for this step. It mainly invokes `super._compute_inv_freq` and `torch.where`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freqs`, `low_freq_wavelen`, `high_freq_wavelen`, `wave_len`, and `new_freqs` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Llama3RotaryEmbedding._compute_inv_freq`，并承载这一阶段的核心逻辑。 它主要调用 `super._compute_inv_freq` 和 `torch.where`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freqs`、`low_freq_wavelen`、`high_freq_wavelen`、`wave_len` 以及 `new_freqs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 583-584: Class `Llama4VisionRotaryEmbedding` declaration and shared state
```python
class Llama4VisionRotaryEmbedding(RotaryEmbedding):
```
**EN:** This block introduces class `Llama4VisionRotaryEmbedding` and the state shared by its methods. It inherits from `RotaryEmbedding`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `Llama4VisionRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `RotaryEmbedding`，说明了它在 SRT 层栈中的接入方式。

### Lines 585-597: `Llama4VisionRotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        dtype: torch.dtype,
    ):
        super().__init__(
            head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype
        )
```
**EN:** This block defines `Llama4VisionRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `Llama4VisionRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。

### Lines 598-602: Internal helper `Llama4VisionRotaryEmbedding._compute_inv_freq`
```python
    def _compute_inv_freq(self, base: Union[int, float]) -> torch.Tensor:
        inv_freqs = super()._compute_inv_freq(base)
        inv_freqs = inv_freqs[: (self.rotary_dim // 2)]
        return inv_freqs
```
**EN:** This block defines `Llama4VisionRotaryEmbedding._compute_inv_freq` and contains the main logic for this step. It mainly invokes `super._compute_inv_freq`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freqs` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Llama4VisionRotaryEmbedding._compute_inv_freq`，并承载这一阶段的核心逻辑。 它主要调用 `super._compute_inv_freq`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freqs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 603-624: Internal helper `Llama4VisionRotaryEmbedding._compute_cos_sin_cache`
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        inv_freq = self._compute_inv_freq(self.base)
        num_patches = self.max_position_embeddings
        img_idx = torch.arange(num_patches, dtype=torch.int32).reshape(num_patches, 1)
        img_idx = torch.cat([img_idx, img_idx[:1]], dim=0)
        img_idx[-1, -1] = -2  # set to ID_CLS_TOKEN
        num_patches_single_dim = int(math.sqrt(num_patches))
        frequencies_x = img_idx % num_patches_single_dim
        frequencies_y = img_idx // num_patches_single_dim
        freqs_x = (
            (frequencies_x + 1)[..., None] * inv_freq[None, None, :]
        ).repeat_interleave(2, dim=-1)
        freqs_y = (
            (frequencies_y + 1)[..., None] * inv_freq[None, None, :]
        ).repeat_interleave(2, dim=-1)
        freqs = torch.cat([freqs_x, freqs_y], dim=-1).float().contiguous()[..., ::2]
        freqs = freqs.masked_fill(img_idx.reshape(-1, 1, 1) < 0, 0)
        cache = torch.view_as_complex(
            torch.stack([torch.cos(freqs), torch.sin(freqs)], dim=-1)
        )
        return cache
```
**EN:** This block defines `Llama4VisionRotaryEmbedding._compute_cos_sin_cache` and contains the main logic for this step. It mainly invokes `self._compute_inv_freq`, `torch.arange.reshape`, `torch.cat`, `int`, and `repeat_interleave`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freq`, `num_patches`, `img_idx`, `num_patches_single_dim`, and `frequencies_x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Llama4VisionRotaryEmbedding._compute_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `self._compute_inv_freq`、`torch.arange.reshape`、`torch.cat`、`int` 以及 `repeat_interleave`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freq`、`num_patches`、`img_idx`、`num_patches_single_dim` 以及 `frequencies_x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 625-642: `Llama4VisionRotaryEmbedding.forward` main forward path
```python
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        self.cos_sin_cache: torch.Tensor = self.cos_sin_cache.to(query.device)
        query_ = torch.view_as_complex(query.float().reshape(*query.shape[:-1], -1, 2))
        key_ = torch.view_as_complex(key.float().reshape(*key.shape[:-1], -1, 2))
        broadcast_shape = [
            d if i == 1 or i == (query_.ndim - 1) else 1
            for i, d in enumerate(query_.shape)
        ]
        freqs_ci = self.cos_sin_cache.view(*broadcast_shape)
        query_out = torch.view_as_real(query_ * freqs_ci).flatten(3)
        key_out = torch.view_as_real(key_ * freqs_ci).flatten(3)
        return query_out.type_as(query), key_out.type_as(key)
```
**EN:** This block defines `Llama4VisionRotaryEmbedding.forward` and contains the main logic for this step. It mainly invokes `self.cos_sin_cache.to`, `torch.view_as_complex`, `self.cos_sin_cache.view`, `torch.view_as_real.flatten`, and `query.float.reshape`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.cos_sin_cache`, `query_`, `key_`, `broadcast_shape`, and `freqs_ci` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Llama4VisionRotaryEmbedding.forward`，并承载这一阶段的核心逻辑。 它主要调用 `self.cos_sin_cache.to`、`torch.view_as_complex`、`self.cos_sin_cache.view`、`torch.view_as_real.flatten` 以及 `query.float.reshape`，说明该流程会编排底层辅助函数或计算内核。 像 `self.cos_sin_cache`、`query_`、`key_`、`broadcast_shape` 以及 `freqs_ci` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 643-648: Class `DynamicNTKAlphaRotaryEmbedding` declaration and shared state
```python
class DynamicNTKAlphaRotaryEmbedding(RotaryEmbedding):
    """RotaryEmbedding extended with Dynamic NTK scaling.

    Credits to the Reddit users /u/bloc97 and /u/emozilla
    """
```
**EN:** This block introduces class `DynamicNTKAlphaRotaryEmbedding` and the state shared by its methods. It inherits from `RotaryEmbedding`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: RotaryEmbedding extended with Dynamic NTK scaling.
**CN:** 该代码块引入类 `DynamicNTKAlphaRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `RotaryEmbedding`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 649-663: `DynamicNTKAlphaRotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        scaling_alpha: float,
        dtype: torch.dtype,
    ) -> None:
        self.scaling_alpha = scaling_alpha
        super().__init__(
            head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype
        )
```
**EN:** This block defines `DynamicNTKAlphaRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.scaling_alpha` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DynamicNTKAlphaRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self.scaling_alpha` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 664-677: Internal helper `DynamicNTKAlphaRotaryEmbedding._compute_cos_sin_cache`
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        max_len = self.max_position_embeddings
        base = self.base * self.scaling_alpha ** (
            self.rotary_dim / (self.rotary_dim - 2)
        )
        inv_freq = self._compute_inv_freq(base)
        t = torch.arange(max_len, dtype=torch.float)
        freqs = torch.einsum("i,j -> ij", t, inv_freq)
        cos = freqs.cos()
        sin = freqs.sin()
        cache = torch.cat((cos, sin), dim=-1)
        return cache
```
**EN:** This block defines `DynamicNTKAlphaRotaryEmbedding._compute_cos_sin_cache` and contains the main logic for this step. It mainly invokes `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `freqs.cos`, and `freqs.sin`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `max_len`, `base`, `inv_freq`, `t`, and `freqs` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DynamicNTKAlphaRotaryEmbedding._compute_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `self._compute_inv_freq`、`torch.arange`、`torch.einsum`、`freqs.cos` 以及 `freqs.sin`，说明该流程会编排底层辅助函数或计算内核。 像 `max_len`、`base`、`inv_freq`、`t` 以及 `freqs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 678-680: Class `DualChunkRotaryEmbedding` declaration and shared state
```python
class DualChunkRotaryEmbedding(MultiPlatformOp):
    """Rotary positional embedding for Dual Chunk Attention."""
```
**EN:** This block introduces class `DualChunkRotaryEmbedding` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Rotary positional embedding for Dual Chunk Attention.
**CN:** 该代码块引入类 `DualChunkRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 681-712: `DualChunkRotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        dtype: torch.dtype,
        chunk_size: int,
        local_size: int,
    ) -> None:
        super().__init__()
        self.head_size = head_size
        self.rotary_dim = rotary_dim
        self.max_position_embeddings = max_position_embeddings
        self.base = base
        self.is_neox_style = is_neox_style
        self.chunk_size = chunk_size
        self.local_size = local_size
        self.dtype = dtype
        self.device = torch.device(f"cuda:{torch.cuda.current_device()}")
        q_cache, qc_cache, k_cache, qc_no_clamp_cache, q_inter_cache = (
            self._compute_cos_sin_cache()
        )
        self.register_buffer("cos_sin_q_cache", q_cache, persistent=False)
        self.register_buffer("cos_sin_qc_cache", qc_cache, persistent=False)
        self.register_buffer("cos_sin_k_cache", k_cache, persistent=False)
        self.register_buffer(
            "cos_sin_qc_no_clamp_cache", qc_no_clamp_cache, persistent=False
        )
        self.register_buffer("cos_sin_q_inter_cache", q_inter_cache, persistent=False)
```
**EN:** This block defines `DualChunkRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `torch.device`, `self._compute_cos_sin_cache`, `self.register_buffer`, and `torch.cuda.current_device`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.head_size`, `self.rotary_dim`, `self.max_position_embeddings`, `self.base`, and `self.is_neox_style` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DualChunkRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`torch.device`、`self._compute_cos_sin_cache`、`self.register_buffer` 以及 `torch.cuda.current_device`，说明该流程会编排底层辅助函数或计算内核。 像 `self.head_size`、`self.rotary_dim`、`self.max_position_embeddings`、`self.base` 以及 `self.is_neox_style` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 713-721: Internal helper `DualChunkRotaryEmbedding._compute_inv_freq`
```python
    def _compute_inv_freq(self, base: Union[int, float]) -> torch.Tensor:
        inv_freq = 1.0 / (
            base
            ** (
                torch.arange(0, self.rotary_dim, 2, dtype=torch.float) / self.rotary_dim
            )
        )
        return inv_freq
```
**EN:** This block defines `DualChunkRotaryEmbedding._compute_inv_freq` and contains the main logic for this step. It mainly invokes `torch.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freq` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DualChunkRotaryEmbedding._compute_inv_freq`，并承载这一阶段的核心逻辑。 它主要调用 `torch.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freq` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 722-755: Internal helper `DualChunkRotaryEmbedding._compute_cos_sin_cache`
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        inv_freq = self._compute_inv_freq(self.base)
        chunk_len = self.chunk_size - self.local_size
        q_t = torch.arange(chunk_len, dtype=torch.float)
        qc_t = (torch.arange(chunk_len, dtype=torch.float) + chunk_len).clamp(
            max=self.chunk_size
        )
        k_t = torch.arange(self.max_position_embeddings, dtype=torch.float) % chunk_len
        qc_no_clamp_t = torch.arange(chunk_len, dtype=torch.float) + chunk_len
        q_inter_t = torch.arange(chunk_len, dtype=torch.float) + self.chunk_size

        q_freqs = torch.outer(q_t, inv_freq)
        qc_freqs = torch.outer(qc_t, inv_freq)
        k_freqs = torch.outer(k_t, inv_freq)
        qc_no_clamp_freqs = torch.outer(qc_no_clamp_t, inv_freq)
        q_inter_freqs = torch.outer(q_inter_t, inv_freq)

        q_cache = torch.cat((q_freqs.cos(), q_freqs.sin()), dim=-1).to(
            dtype=self.dtype, device=self.device
        )
        qc_cache = torch.cat((qc_freqs.cos(), qc_freqs.sin()), dim=-1).to(
            dtype=self.dtype, device=self.device
        )
        k_cache = torch.cat((k_freqs.cos(), k_freqs.sin()), dim=-1).to(
            dtype=self.dtype, device=self.device
        )
        qc_no_clamp_cache = torch.cat(
            (qc_no_clamp_freqs.cos(), qc_no_clamp_freqs.sin()), dim=-1
        ).to(dtype=self.dtype, device=self.device)
        q_inter_cache = torch.cat(
            (q_inter_freqs.cos(), q_inter_freqs.sin()), dim=-1
        ).to(dtype=self.dtype, device=self.device)
        return q_cache, qc_cache, k_cache, qc_no_clamp_cache, q_inter_cache
```
**EN:** This block defines `DualChunkRotaryEmbedding._compute_cos_sin_cache` and contains the main logic for this step. It mainly invokes `self._compute_inv_freq`, `torch.arange`, `clamp`, `torch.outer`, and `torch.cat.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freq`, `chunk_len`, `q_t`, `qc_t`, and `k_t` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DualChunkRotaryEmbedding._compute_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `self._compute_inv_freq`、`torch.arange`、`clamp`、`torch.outer` 以及 `torch.cat.to`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freq`、`chunk_len`、`q_t`、`qc_t` 以及 `k_t` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 756-811: `DualChunkRotaryEmbedding.forward` main forward path
```python
    def forward(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        query = query.view(*query.shape[:-1], -1, self.head_size)
        key = key.view(*key.shape[:-1], -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        key_rot = key[..., : self.rotary_dim]
        if self.rotary_dim < self.head_size:
            query_pass = query[..., self.rotary_dim :]
            key_pass = key[..., self.rotary_dim :]
        else:
            query_pass = None
            key_pass = None

        positions_with_offsets = (
            torch.add(positions, offsets) if offsets is not None else positions
        )
        key = self._apply_rotary_embedding(
            self.cos_sin_k_cache[positions_with_offsets], key_rot, key_pass
        )
        chunk_len = self.chunk_size - self.local_size
        query = self._apply_rotary_embedding(
            self.cos_sin_q_cache[positions_with_offsets % chunk_len],
            query_rot,
            query_pass,
        )
        query_succ = self._apply_rotary_embedding(
            self.cos_sin_qc_cache[positions_with_offsets % chunk_len],
            query_rot,
            query_pass,
        )
        query_inter = self._apply_rotary_embedding(
            self.cos_sin_qc_cache[chunk_len - 1].repeat(positions.shape[0], 1),
            query_rot,
            query_pass,
        )
        query_succ_critical = self._apply_rotary_embedding(
            self.cos_sin_qc_no_clamp_cache[positions_with_offsets % chunk_len],
            query_rot,
            query_pass,
        )
        query_inter_critical = self._apply_rotary_embedding(
            self.cos_sin_q_inter_cache[positions_with_offsets % chunk_len],
            query_rot,
            query_pass,
        )
        query = torch.cat(
            (query, query_succ, query_inter, query_succ_critical, query_inter_critical),
            dim=-1,
        )
        return query, key
```
**EN:** This block defines `DualChunkRotaryEmbedding.forward` and contains the main logic for this step. It mainly invokes `query.view`, `key.view`, `self._apply_rotary_embedding`, `torch.cat`, and `torch.add`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `query`, `key`, `query_rot`, `key_rot`, and `positions_with_offsets` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DualChunkRotaryEmbedding.forward`，并承载这一阶段的核心逻辑。 它主要调用 `query.view`、`key.view`、`self._apply_rotary_embedding`、`torch.cat` 以及 `torch.add`，说明该流程会编排底层辅助函数或计算内核。 像 `query`、`key`、`query_rot`、`key_rot` 以及 `positions_with_offsets` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 812-827: Internal helper `DualChunkRotaryEmbedding._apply_rotary_embedding`
```python
    def _apply_rotary_embedding(self, cos_sin, hidden_rot, hidden_pass):
        cos, sin = cos_sin.chunk(2, dim=-1)
        if self.is_neox_style:
            cos = cos.repeat(1, 1, 2).unsqueeze(-2)
            sin = sin.repeat(1, 1, 2).unsqueeze(-2)
        else:
            cos = cos.repeat_interleave(2, dim=-1).unsqueeze(-2)
            sin = sin.repeat_interleave(2, dim=-1).unsqueeze(-2)
        rotate_fn = rotate_neox if self.is_neox_style else rotate_gptj
        hidden_rot = hidden_rot * cos + rotate_fn(hidden_rot) * sin
        if self.rotary_dim < self.head_size:
            hidden = torch.cat((hidden_rot, hidden_pass), dim=-1)
        else:
            hidden = hidden_rot
        return hidden.flatten(-2).squeeze(0)
```
**EN:** This block defines `DualChunkRotaryEmbedding._apply_rotary_embedding` and contains the main logic for this step. It mainly invokes `cos_sin.chunk`, `hidden.flatten.squeeze`, `cos.repeat.unsqueeze`, `sin.repeat.unsqueeze`, and `cos.repeat_interleave.unsqueeze`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cos`, `sin`, `rotate_fn`, `hidden_rot`, and `hidden` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DualChunkRotaryEmbedding._apply_rotary_embedding`，并承载这一阶段的核心逻辑。 它主要调用 `cos_sin.chunk`、`hidden.flatten.squeeze`、`cos.repeat.unsqueeze`、`sin.repeat.unsqueeze` 以及 `cos.repeat_interleave.unsqueeze`，说明该流程会编排底层辅助函数或计算内核。 像 `cos`、`sin`、`rotate_fn`、`hidden_rot` 以及 `hidden` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 828-835: Function `DualChunkRotaryEmbedding.extra_repr` and its core logic
```python
    def extra_repr(self) -> str:
        s = f"head_size={self.head_size}, rotary_dim={self.rotary_dim}"
        s += f", max_position_embeddings={self.max_position_embeddings}"
        s += f", base={self.base}, is_neox_style={self.is_neox_style}"
        s += f", chunk_size={self.chunk_size}, local_size={self.local_size}"
        return s
```
**EN:** This block defines `DualChunkRotaryEmbedding.extra_repr` and contains the main logic for this step. Intermediate names such as `s` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DualChunkRotaryEmbedding.extra_repr`，并承载这一阶段的核心逻辑。 像 `s` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 836-841: Class `DynamicNTKScalingRotaryEmbedding` declaration and shared state
```python
class DynamicNTKScalingRotaryEmbedding(RotaryEmbedding):
    """RotaryEmbedding extended with Dynamic NTK scaling.

    Credits to the Reddit users /u/bloc97 and /u/emozilla
    """
```
**EN:** This block introduces class `DynamicNTKScalingRotaryEmbedding` and the state shared by its methods. It inherits from `RotaryEmbedding`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: RotaryEmbedding extended with Dynamic NTK scaling.
**CN:** 该代码块引入类 `DynamicNTKScalingRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `RotaryEmbedding`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 842-856: `DynamicNTKScalingRotaryEmbedding` initialization and state setup
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
    ) -> None:
        self.scaling_factor = scaling_factor
        super().__init__(
            head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype
        )
```
**EN:** This block defines `DynamicNTKScalingRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.scaling_factor` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DynamicNTKScalingRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self.scaling_factor` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 857-871: Internal helper `DynamicNTKScalingRotaryEmbedding._compute_cos_sin_cache`
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        max_len = self.max_position_embeddings * self.scaling_factor
        base = self.base * (
            (self.scaling_factor * max_len / self.max_position_embeddings)
            - (self.scaling_factor - 1)
        ) ** (self.rotary_dim / (self.rotary_dim - 2))
        inv_freq = self._compute_inv_freq(base)
        t = torch.arange(max_len, dtype=torch.float)
        freqs = torch.einsum("i,j -> ij", t, inv_freq)
        cos = freqs.cos()
        sin = freqs.sin()
        cache = torch.cat((cos, sin), dim=-1)
        return cache
```
**EN:** This block defines `DynamicNTKScalingRotaryEmbedding._compute_cos_sin_cache` and contains the main logic for this step. It mainly invokes `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `freqs.cos`, and `freqs.sin`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `max_len`, `base`, `inv_freq`, `t`, and `freqs` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DynamicNTKScalingRotaryEmbedding._compute_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `self._compute_inv_freq`、`torch.arange`、`torch.einsum`、`freqs.cos` 以及 `freqs.sin`，说明该流程会编排底层辅助函数或计算内核。 像 `max_len`、`base`、`inv_freq`、`t` 以及 `freqs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 872-882: Class `Gemma4RotaryEmbedding` declaration and shared state
```python
class Gemma4RotaryEmbedding(RotaryEmbedding):
    """Gemma4-specific RoPE with cross-mixing.

    Instead of rotating the first `rotary_dim` dimensions contiguously,
    splits the head into two halves and applies rotation across both.

    For a head_dim of D and rotary_dim of R:
    - Standard RoPE rotates: [0, R)
    - Gemma4 RoPE rotates: [0, R/2) cross-mixed with [D/2, D/2 + R/2)
    """
```
**EN:** This block introduces class `Gemma4RotaryEmbedding` and the state shared by its methods. It inherits from `RotaryEmbedding`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Gemma4-specific RoPE with cross-mixing.
**CN:** 该代码块引入类 `Gemma4RotaryEmbedding`，并定义其方法共享的状态。 它继承自 `RotaryEmbedding`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 883-906: `Gemma4RotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
    ) -> None:
        # Store angles before calling super().__init__
        # rotary_dim is already scaled by partial_rotary_factor in get_rope
        # For Gemma4: head_size=512, partial_rotary_factor=0.25 -> rotary_dim=128
        self.rope_angles = rotary_dim // 2  # Number of rotation angles per half
        self.nope_angles = (head_size // 2) - self.rope_angles  # Non-rotated per half

        super().__init__(
            head_size,
            head_size,
            max_position_embeddings,
            base,
            is_neox_style,
            dtype,
        )
```
**EN:** This block defines `Gemma4RotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.rope_angles` and `self.nope_angles` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Gemma4RotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self.rope_angles` 和 `self.nope_angles` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 907-926: Internal helper `Gemma4RotaryEmbedding._compute_inv_freq`
```python
    def _compute_inv_freq(self, base: float) -> torch.Tensor:
        """Compute frequencies only for the rotated dimensions.

        Non-rotated dims are padded with 0.0 to produce identity rotation.
        """
        freq_exponents = (
            torch.arange(0, 2 * self.rope_angles, 2, dtype=torch.float) / self.head_size
        )
        inv_freq = 1.0 / (base**freq_exponents)

        # Zero-pad for non-rotated dims (identity rotation: cos=1, sin=0)
        if self.nope_angles > 0:
            inv_freq = torch.cat(
                [
                    inv_freq,
                    torch.zeros(self.nope_angles, dtype=torch.float),
                ]
            )
        return inv_freq
```
**EN:** This block defines `Gemma4RotaryEmbedding._compute_inv_freq` and contains the main logic for this step. It mainly invokes `torch.arange`, `torch.cat`, and `torch.zeros`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `freq_exponents` and `inv_freq` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma4RotaryEmbedding._compute_inv_freq`，并承载这一阶段的核心逻辑。 它主要调用 `torch.arange`、`torch.cat` 以及 `torch.zeros`，说明该流程会编排底层辅助函数或计算内核。 像 `freq_exponents` 和 `inv_freq` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 927-932: Function `Gemma4RotaryEmbedding.extra_repr` and its core logic
```python
    def extra_repr(self) -> str:
        s = f"head_size={self.head_size}, rotary_dim={self.rotary_dim}"
        s += f", rope_angles={self.rope_angles}, nope_angles={self.nope_angles}"
        s += f", max_position_embeddings={self.max_position_embeddings}"
        s += f", base={self.base}, is_neox_style={self.is_neox_style}"
        return s
```
**EN:** This block defines `Gemma4RotaryEmbedding.extra_repr` and contains the main logic for this step. Intermediate names such as `s` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma4RotaryEmbedding.extra_repr`，并承载这一阶段的核心逻辑。 像 `s` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `Phi3LongRoPEScaledRotaryEmbedding`, `FourierRotaryEmbedding`, `DeepseekScalingRotaryEmbedding`, `Llama3RotaryEmbedding`, and `Llama4VisionRotaryEmbedding`. / **主要符号**：核心入口包括 `Phi3LongRoPEScaledRotaryEmbedding`、`FourierRotaryEmbedding`、`DeepseekScalingRotaryEmbedding`、`Llama3RotaryEmbedding` 以及 `Llama4VisionRotaryEmbedding`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `math`, `typing.List`, `typing.Optional`, `typing.Tuple`, and `typing.Union` / **标准库**：`__future__.annotations`、`math`、`typing.List`、`typing.Optional`、`typing.Tuple` 以及 `typing.Union`
- **Third-party**: `torch`, `torch.nn`, `torch.nn.functional`, and `torch_npu` / **第三方依赖**：`torch`、`torch.nn`、`torch.nn.functional` 以及 `torch_npu`
- **Internal SGLang modules**: `sglang.srt.layers.rotary_embedding.base.RotaryEmbedding`, `sglang.srt.layers.rotary_embedding.utils.apply_rotary_pos_emb_native`, `sglang.srt.layers.rotary_embedding.utils.rotate_gptj`, `sglang.srt.layers.rotary_embedding.utils.rotate_neox`, `sglang.srt.layers.rotary_embedding.yarn.yarn_find_correction_range`, `sglang.srt.layers.rotary_embedding.yarn.yarn_get_mscale`, `sglang.srt.layers.rotary_embedding.yarn.yarn_linear_ramp_mask`, `sglang.srt.layers.utils.MultiPlatformOp`, `sglang.srt.utils.cpu_has_amx_support`, `sglang.srt.utils.get_device`, `sglang.srt.utils.is_cuda`, and `sglang.srt.utils.is_hip` / **SGLang 内部模块**：`sglang.srt.layers.rotary_embedding.base.RotaryEmbedding`、`sglang.srt.layers.rotary_embedding.utils.apply_rotary_pos_emb_native`、`sglang.srt.layers.rotary_embedding.utils.rotate_gptj`、`sglang.srt.layers.rotary_embedding.utils.rotate_neox`、`sglang.srt.layers.rotary_embedding.yarn.yarn_find_correction_range`、`sglang.srt.layers.rotary_embedding.yarn.yarn_get_mscale`、`sglang.srt.layers.rotary_embedding.yarn.yarn_linear_ramp_mask`、`sglang.srt.layers.utils.MultiPlatformOp`、`sglang.srt.utils.cpu_has_amx_support`、`sglang.srt.utils.get_device`、`sglang.srt.utils.is_cuda` 以及 `sglang.srt.utils.is_hip`
