# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/rotary_embedding/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `RotaryEmbedding`, and `LinearScalingRotaryEmbedding`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: RotaryEmbedding base class and LinearScalingRotaryEmbedding variant. / 该文件属于运行时算子层。它围绕 `RotaryEmbedding` 和 `LinearScalingRotaryEmbedding` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module setup and imports / 模块初始化与导入
```python
"""RotaryEmbedding base class and LinearScalingRotaryEmbedding variant."""

import torch

from sglang.multimodal_gen.runtime.layers.custom_op import CustomOp

from .utils import _apply_rotary_emb
```
**EN:** This block establishes the module context and imports `torch`, `sglang.multimodal_gen.runtime.layers.custom_op`, and `.utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `torch`、`sglang.multimodal_gen.runtime.layers.custom_op` 和 `.utils`。这些依赖为后续实现提供所需符号。

### Lines 11-13: `RotaryEmbedding` class overview / `RotaryEmbedding` 类概览
```python
class RotaryEmbedding(CustomOp):
    """Original rotary positional embedding."""
```
**EN:** This block defines class `RotaryEmbedding`. Original rotary positional embedding. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `RotaryEmbedding`。 它用于封装 rotary embedding 相关行为。 它继承自 `CustomOp`。

### Lines 14-34: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int | float,
        is_neox_style: bool,
        dtype: torch.dtype,
    ) -> None:
        super().__init__()
        self.head_size = head_size
        self.rotary_dim = rotary_dim
        self.max_position_embeddings = max_position_embeddings
        self.base = base
        self.is_neox_style = is_neox_style
        self.dtype = dtype

        cache = self._compute_cos_sin_cache()
        cache = cache.to(dtype)
        self.cos_sin_cache: torch.Tensor
        self.register_buffer("cos_sin_cache", cache, persistent=False)
```
**EN:** This block defines method `__init__` on `RotaryEmbedding`. It initializes the instance state. Key calls include `super.__init__`, `self._compute_cos_sin_cache`, `cache.to`, `self.register_buffer`, and `super`. Parameters such as `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, and `is_neox_style` drive the behavior in this section.
**CN:** 该代码块定义了 `RotaryEmbedding` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`self._compute_cos_sin_cache`、`cache.to`、`self.register_buffer` 和 `super`。 本段逻辑主要由 `head_size`、`rotary_dim`、`max_position_embeddings`、`base` 和 `is_neox_style` 等参数驱动。

### Lines 36-48: `_compute_inv_freq` implementation / `_compute_inv_freq` 实现
```python
    def _compute_inv_freq(self, base: int | float) -> torch.Tensor:
        """Compute the inverse frequency."""
        # NOTE(woosuk): To exactly match the HF implementation, we need to
        # use CPU to compute the cache and then move it to GPU. However, we
        # create the cache on GPU for faster initialization. This may cause
        # a slight numerical difference between the HF implementation and ours.
        inv_freq = 1.0 / (
            base
            ** (
                torch.arange(0, self.rotary_dim, 2, dtype=torch.float) / self.rotary_dim
            )
        )
        return inv_freq
```
**EN:** This block defines method `_compute_inv_freq` on `RotaryEmbedding`. Compute the inverse frequency. Key calls include `torch.arange`. Parameters such as `base` drive the behavior in this section.
**CN:** 该代码块定义了 `RotaryEmbedding` 的方法 `_compute_inv_freq`。 它用于计算inv freq。 关键调用包括 `torch.arange`。 本段逻辑主要由 `base` 等参数驱动。

### Lines 50-59: `_compute_cos_sin_cache` implementation / `_compute_cos_sin_cache` 实现
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        """Compute the cos and sin cache."""
        inv_freq = self._compute_inv_freq(self.base)
        t = torch.arange(self.max_position_embeddings, dtype=torch.float)

        freqs = torch.einsum("i,j -> ij", t, inv_freq)
        cos = freqs.cos()
        sin = freqs.sin()
        cache = torch.cat((cos, sin), dim=-1)
        return cache
```
**EN:** This block defines method `_compute_cos_sin_cache` on `RotaryEmbedding`. Compute the cos and sin cache. Key calls include `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `freqs.cos`, and `freqs.sin`.
**CN:** 该代码块定义了 `RotaryEmbedding` 的方法 `_compute_cos_sin_cache`。 它用于计算cos sin cache。 关键调用包括 `self._compute_inv_freq`、`torch.arange`、`torch.einsum`、`freqs.cos` 和 `freqs.sin`。

### Lines 61-62: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(self, *args, **kwargs):
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_cuda` on `RotaryEmbedding`. It executes cuda. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `RotaryEmbedding` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `self.forward_native`。

### Lines 64-92: `forward_native` implementation / `forward_native` 实现
```python
    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """A PyTorch-native implementation of forward()."""
        if offsets is not None:
            positions = positions + offsets
        positions = positions.flatten()
        num_tokens = positions.shape[0]
        cos_sin = self.cos_sin_cache.index_select(0, positions)
        cos, sin = cos_sin.chunk(2, dim=-1)

        query_shape = query.shape
        query = query.reshape(num_tokens, -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = _apply_rotary_emb(query_rot, cos, sin, self.is_neox_style)
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        key_shape = key.shape
        key = key.reshape(num_tokens, -1, self.head_size)
        key_rot = key[..., : self.rotary_dim]
        key_pass = key[..., self.rotary_dim :]
        key_rot = _apply_rotary_emb(key_rot, cos, sin, self.is_neox_style)
        key = torch.cat((key_rot, key_pass), dim=-1).reshape(key_shape)
        return query, key
```
**EN:** This block defines method `forward_native` on `RotaryEmbedding`. A PyTorch-native implementation of forward(). Key calls include `positions.flatten`, `self.cos_sin_cache.index_select`, `cos_sin.chunk`, `query.reshape`, and `_apply_rotary_emb`. The implementation branches on conditions. Parameters such as `positions`, `query`, `key`, and `offsets` drive the behavior in this section.
**CN:** 该代码块定义了 `RotaryEmbedding` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `positions.flatten`、`self.cos_sin_cache.index_select`、`cos_sin.chunk`、`query.reshape` 和 `_apply_rotary_emb`。 实现中包含条件分支。 本段逻辑主要由 `positions`、`query`、`key` 和 `offsets` 等参数驱动。

### Lines 94-98: `extra_repr` implementation / `extra_repr` 实现
```python
    def extra_repr(self) -> str:
        s = f"head_size={self.head_size}, rotary_dim={self.rotary_dim}"
        s += f", max_position_embeddings={self.max_position_embeddings}"
        s += f", base={self.base}, is_neox_style={self.is_neox_style}"
        return s
```
**EN:** This block defines method `extra_repr` on `RotaryEmbedding`. It handles extra repr logic.
**CN:** 该代码块定义了 `RotaryEmbedding` 的方法 `extra_repr`。 它用于处理 extra repr 相关逻辑。

### Lines 101-101: `LinearScalingRotaryEmbedding` class overview / `LinearScalingRotaryEmbedding` 类概览
```python
class LinearScalingRotaryEmbedding(RotaryEmbedding):
```
**EN:** This block defines class `LinearScalingRotaryEmbedding`. It encapsulates linear scaling rotary embedding behavior. It inherits from `RotaryEmbedding`.
**CN:** 该代码块定义了类 `LinearScalingRotaryEmbedding`。 它用于封装 linear scaling rotary embedding 相关行为。 它继承自 `RotaryEmbedding`。

### Lines 102-120: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int | float,
        is_neox_style: bool,
        dtype: torch.dtype,
        scaling_factor: float,
    ) -> None:
        self.scaling_factor = float(scaling_factor)
        super().__init__(
            head_size=head_size,
            rotary_dim=rotary_dim,
            max_position_embeddings=max_position_embeddings,
            base=base,
            is_neox_style=is_neox_style,
            dtype=dtype,
        )
```
**EN:** This block defines method `__init__` on `LinearScalingRotaryEmbedding`. It initializes the instance state. Key calls include `float`, `super.__init__`, and `super`. Parameters such as `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, and `is_neox_style` drive the behavior in this section.
**CN:** 该代码块定义了 `LinearScalingRotaryEmbedding` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `float`、`super.__init__` 和 `super`。 本段逻辑主要由 `head_size`、`rotary_dim`、`max_position_embeddings`、`base` 和 `is_neox_style` 等参数驱动。

### Lines 122-130: `_compute_cos_sin_cache` implementation / `_compute_cos_sin_cache` 实现
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        inv_freq = self._compute_inv_freq(self.base)
        t = torch.arange(self.max_position_embeddings, dtype=torch.float)
        t = t / self.scaling_factor
        freqs = torch.einsum("i,j -> ij", t, inv_freq)
        cos = freqs.cos()
        sin = freqs.sin()
        cache = torch.cat((cos, sin), dim=-1)
        return cache
```
**EN:** This block defines method `_compute_cos_sin_cache` on `LinearScalingRotaryEmbedding`. It computes cos sin cache. Key calls include `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `freqs.cos`, and `freqs.sin`.
**CN:** 该代码块定义了 `LinearScalingRotaryEmbedding` 的方法 `_compute_cos_sin_cache`。 它用于计算cos sin cache。 关键调用包括 `self._compute_inv_freq`、`torch.arange`、`torch.einsum`、`freqs.cos` 和 `freqs.sin`。

## Key Concepts / 关键概念
- `RotaryEmbedding`: Original rotary positional embedding. / 核心类，用于封装 rotary embedding 相关行为。
- `LinearScalingRotaryEmbedding`: Primary class that encapsulates linear scaling rotary embedding behavior. / 核心类，用于封装 linear scaling rotary embedding 相关行为。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.custom_op`, `.utils`

- **Total lines / 总行数**: 130
