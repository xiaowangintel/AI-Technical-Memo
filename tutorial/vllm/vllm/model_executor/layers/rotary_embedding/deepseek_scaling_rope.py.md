# deepseek_scaling_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/rotary_embedding/deepseek_scaling_rope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `DeepseekScalingRotaryEmbedding` and related helpers for rotary positional embedding variants. / 实现 `DeepseekScalingRotaryEmbedding` 及其相关辅助逻辑，用于旋转位置编码变体。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-17)
```python
import math

import torch

from vllm.platforms import current_platform
from vllm.utils.flashinfer import has_flashinfer

from .base import RotaryEmbeddingBase
from .common import (
    rotate_gptj,
    rotate_neox,
    yarn_find_correction_range,
    yarn_linear_ramp_mask,
)
```
**EN:** This opening block pulls in external dependencies such as `math`, `torch` and internal modules such as `vllm.platforms`, `vllm.utils.flashinfer`, `.base`, `.common`. That import mix shows the file is part of the rotary positional embedding variants stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `math`, `torch`）以及内部模块（如 `vllm.platforms`, `vllm.utils.flashinfer`, `.base`, `.common`）。这些导入关系表明该文件属于旋转位置编码变体栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `yarn_get_mscale` (lines 20-23)
```python
def yarn_get_mscale(scale: float = 1, mscale: float = 1) -> float:
    if scale <= 1:
        return 1.0
    return 0.1 * mscale * math.log(scale) + 1.0
```
**EN:** Defines function `yarn_get_mscale` with signature `yarn_get_mscale(scale: float=1, mscale: float=1) -> float`. It mainly works with `scale`, `mscale`; adjusts rotary-position-encoding parameters or application logic. The body uses branching. Key calls include `math.log`.
**CN:** 定义函数 `yarn_get_mscale`，其签名为 `yarn_get_mscale(scale: float=1, mscale: float=1) -> float`。它主要围绕 `scale`, `mscale` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断。关键调用包括 `math.log`。

### Class `DeepseekScalingRotaryEmbedding` overview (lines 26-227)
```python
class DeepseekScalingRotaryEmbedding(RotaryEmbeddingBase):
    """RotaryEmbedding extended with YaRN method.

    Credits to Peng et al. github.com/jquesnelle/yarn
    """

    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
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
        init_cache: bool = True,
    ) -> None:
        self.scaling_factor = scaling_factor
```
**EN:** Defines class `DeepseekScalingRotaryEmbedding` with base classes `RotaryEmbeddingBase` and decorators none. It acts as an embedding layer with model-parallel awareness and exposes 8 direct methods, with notable entries `__init__`, `_compute_inv_freq`, `_compute_cos_sin_cache`, `forward_native`, `forward_static`, `forward_xpu`. Its docstring says: RotaryEmbedding extended with YaRN method.
**CN:** 定义类 `DeepseekScalingRotaryEmbedding`，其基类为 `RotaryEmbeddingBase`，装饰器为 无。它在整体实现中充当具备模型并行意识的嵌入层，并直接暴露 8 个方法，较重要的包括 `__init__`, `_compute_inv_freq`, `_compute_cos_sin_cache`, `forward_native`, `forward_static`, `forward_xpu`。 文档字符串进一步说明了该类的定位。

### Method `DeepseekScalingRotaryEmbedding.__init__` (lines 32-76)
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
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
        init_cache: bool = True,
    ) -> None:
        self.scaling_factor = scaling_factor
        self.extrapolation_factor = extrapolation_factor
        self.attn_factor = attn_factor
        self.beta_fast = beta_fast
        self.beta_slow = beta_slow
        # Get n-d magnitude scaling corrected for interpolation.
        self.mscale = float(
            yarn_get_mscale(self.scaling_factor, float(mscale))
            / yarn_get_mscale(self.scaling_factor, float(mscale_all_dim))
            * attn_factor
        )
        self.use_flashinfer = (
            self.enabled()
            and dtype in (torch.float16, torch.bfloat16)
            and current_platform.is_cuda()
            and has_flashinfer()
            and head_size in [64, 128, 256, 512]
        )
        super().__init__(
            head_size,
            rotary_dim,
            max_position_embeddings,
            base,
            is_neox_style,
            dtype,
            init_cache=init_cache,
        )
```
**EN:** Defines function `DeepseekScalingRotaryEmbedding.__init__` with signature `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, scaling_factor: float, dtype: torch.dtype, *, extrapolation_factor: float=1, attn_factor: float=1, beta_fast: int=32, beta_slow: int=1, mscale: float=1, mscale_all_dim: float=0, init_cache: bool=True) -> None`. It mainly works with `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `scaling_factor`, `dtype`, `extrapolation_factor`; initializes the object state and cached resources. The body uses tensor/kernel operations. Key calls include `float`, `super.__init__`, `self.enabled`, `current_platform.is_cuda`, `has_flashinfer`, `super`.
**CN:** 定义函数 `DeepseekScalingRotaryEmbedding.__init__`，其签名为 `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, scaling_factor: float, dtype: torch.dtype, *, extrapolation_factor: float=1, attn_factor: float=1, beta_fast: int=32, beta_slow: int=1, mscale: float=1, mscale_all_dim: float=0, init_cache: bool=True) -> None`。它主要围绕 `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `scaling_factor`, `dtype`, `extrapolation_factor` 展开；负责初始化对象状态和缓存资源。函数体包含张量或内核操作。关键调用包括 `float`, `super.__init__`, `self.enabled`, `current_platform.is_cuda`, `has_flashinfer`, `super`。

### Method `DeepseekScalingRotaryEmbedding.forward_native` (lines 121-139)
```python
    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """PyTorch-native implementation equivalent to forward()."""
        assert key is not None
        return self.forward_static(
            positions,
            query,
            key,
            self.head_size,
            self.rotary_dim,
            self.cos_sin_cache,
            self.is_neox_style,
            offsets,
        )
```
**EN:** Defines function `DeepseekScalingRotaryEmbedding.forward_native` with signature `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses validation/error handling. Key calls include `self.forward_static`.
**CN:** 定义函数 `DeepseekScalingRotaryEmbedding.forward_native`，其签名为 `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含校验或报错逻辑。关键调用包括 `self.forward_static`。

### Method `DeepseekScalingRotaryEmbedding.forward_static` (lines 142-181)
```python
    def forward_static(
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None,
        head_size: int,
        rotary_dim: int,
        cos_sin_cache: torch.Tensor,
        is_neox_style: bool,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """A static implementation of forward()."""
        assert key is not None
        query_rot = query[..., :rotary_dim]
        key_rot = key[..., :rotary_dim]
        if rotary_dim < head_size:
            query_pass = query[..., rotary_dim:]
            key_pass = key[..., rotary_dim:]

        cos_sin = cos_sin_cache[
            torch.add(positions, offsets) if offsets is not None else positions
        ]
        cos, sin = cos_sin.chunk(2, dim=-1)
        if is_neox_style:
            cos = torch.cat((cos, cos), dim=-1).unsqueeze(-2)
            sin = torch.cat((sin, sin), dim=-1).unsqueeze(-2)
        else:
            cos = cos.repeat_interleave(2, dim=-1).unsqueeze(-2)
            sin = sin.repeat_interleave(2, dim=-1).unsqueeze(-2)

        rotate_fn = rotate_neox if is_neox_style else rotate_gptj
        query_rot = query_rot * cos + rotate_fn(query_rot) * sin
        key_rot = key_rot * cos + rotate_fn(key_rot) * sin

        if rotary_dim < head_size:
            query = torch.cat((query_rot, query_pass), dim=-1)
            key = torch.cat((key_rot, key_pass), dim=-1)
        else:
            query = query_rot
            key = key_rot
        return query, key
```
**EN:** Defines function `DeepseekScalingRotaryEmbedding.forward_static` with signature `forward_static(positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None, head_size: int, rotary_dim: int, cos_sin_cache: torch.Tensor, is_neox_style: bool, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `head_size`, `rotary_dim`, `cos_sin_cache`, `is_neox_style`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `cos_sin.chunk`, `torch.cat.unsqueeze`, `cos.repeat_interleave.unsqueeze`, `sin.repeat_interleave.unsqueeze`, `torch.cat`, `torch.add`.
**CN:** 定义函数 `DeepseekScalingRotaryEmbedding.forward_static`，其签名为 `forward_static(positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None, head_size: int, rotary_dim: int, cos_sin_cache: torch.Tensor, is_neox_style: bool, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `head_size`, `rotary_dim`, `cos_sin_cache`, `is_neox_style`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `cos_sin.chunk`, `torch.cat.unsqueeze`, `cos.repeat_interleave.unsqueeze`, `sin.repeat_interleave.unsqueeze`, `torch.cat`, `torch.add`。

### Method `DeepseekScalingRotaryEmbedding.forward_xpu` (lines 183-198)
```python
    def forward_xpu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        return torch.ops.vllm.xpu_ops_deepseek_scaling_rope(
            positions,
            query,
            key,
            offsets,
            self._match_cos_sin_cache_dtype(query),
            self.rotary_dim,
            self.is_neox_style,
        )
```
**EN:** Defines function `DeepseekScalingRotaryEmbedding.forward_xpu` with signature `forward_xpu(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses tensor/kernel operations. Key calls include `torch.ops.vllm.xpu_ops_deepseek_scaling_rope`, `self._match_cos_sin_cache_dtype`.
**CN:** 定义函数 `DeepseekScalingRotaryEmbedding.forward_xpu`，其签名为 `forward_xpu(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含张量或内核操作。关键调用包括 `torch.ops.vllm.xpu_ops_deepseek_scaling_rope`, `self._match_cos_sin_cache_dtype`。

### Method `DeepseekScalingRotaryEmbedding.forward_hip` (lines 200-207)
```python
    def forward_hip(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        return self.forward_native(positions, query, key, offsets)
```
**EN:** Defines function `DeepseekScalingRotaryEmbedding.forward_hip` with signature `forward_hip(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include `self.forward_native`.
**CN:** 定义函数 `DeepseekScalingRotaryEmbedding.forward_hip`，其签名为 `forward_hip(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.forward_native`。

### Method `DeepseekScalingRotaryEmbedding.forward_cuda` (lines 209-227)
```python
    def forward_cuda(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        if self.use_flashinfer:
            torch.ops.vllm.flashinfer_rotary_embedding(
                torch.add(positions, offsets) if offsets is not None else positions,
                query,
                key,
                self.head_size,
                self.cos_sin_cache,
                self.is_neox_style,
            )
            return query, key
        else:
            return self.forward_native(positions, query, key, offsets)
```
**EN:** Defines function `DeepseekScalingRotaryEmbedding.forward_cuda` with signature `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `torch.ops.vllm.flashinfer_rotary_embedding`, `self.forward_native`, `torch.add`.
**CN:** 定义函数 `DeepseekScalingRotaryEmbedding.forward_cuda`，其签名为 `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `torch.ops.vllm.flashinfer_rotary_embedding`, `self.forward_native`, `torch.add`。

### Class `DeepseekV4ScalingRotaryEmbedding` overview (lines 230-344)
```python
class DeepseekV4ScalingRotaryEmbedding(DeepseekScalingRotaryEmbedding):
    """RotaryEmbedding extended with YaRN method.

    Credits to Peng et al. github.com/jquesnelle/yarn

    Compared to DeepseekScalingRotaryEmbedding:
    - Applies RoPE to the last rotary_dim
    - The forward method requires an inverse parameter to indicate
      whether to negate the sin
    - Supports applying RoPE to query only (without key)
    - cos_sin_cache stored as fp32 for higher precision RoPE
    """

    def __init__(self, *args, **kwargs):
        # Avoid compute cache repeatedly
        kwargs.pop("init_cache", None)
        super().__init__(*args, **kwargs, init_cache=False)
        cache_fp32 = self._compute_cos_sin_cache()
        self.register_buffer("cos_sin_cache", cache_fp32, persistent=False)

    def _compute_cos_sin_cache(self) -> torch.Tensor:
        inv_freq = self._compute_inv_freq(self.scaling_factor)
        t = torch.arange(
            self.max_position_embeddings * self.scaling_factor,
            device=current_platform.device_type,
```
**EN:** Defines class `DeepseekV4ScalingRotaryEmbedding` with base classes `DeepseekScalingRotaryEmbedding` and decorators none. It acts as an embedding layer with model-parallel awareness and exposes 5 direct methods, with notable entries `__init__`, `_compute_cos_sin_cache`, `forward_native`, `forward_hip`, `forward_cuda`. Its docstring says: RotaryEmbedding extended with YaRN method.
**CN:** 定义类 `DeepseekV4ScalingRotaryEmbedding`，其基类为 `DeepseekScalingRotaryEmbedding`，装饰器为 无。它在整体实现中充当具备模型并行意识的嵌入层，并直接暴露 5 个方法，较重要的包括 `__init__`, `_compute_cos_sin_cache`, `forward_native`, `forward_hip`, `forward_cuda`。 文档字符串进一步说明了该类的定位。

### Method `DeepseekV4ScalingRotaryEmbedding.__init__` (lines 243-248)
```python
    def __init__(self, *args, **kwargs):
        # Avoid compute cache repeatedly
        kwargs.pop("init_cache", None)
        super().__init__(*args, **kwargs, init_cache=False)
        cache_fp32 = self._compute_cos_sin_cache()
        self.register_buffer("cos_sin_cache", cache_fp32, persistent=False)
```
**EN:** Defines function `DeepseekV4ScalingRotaryEmbedding.__init__` with signature `__init__(self, *args, **kwargs)`. It mainly works with `*args`, `**kwargs`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `kwargs.pop`, `super.__init__`, `self._compute_cos_sin_cache`, `self.register_buffer`, `super`.
**CN:** 定义函数 `DeepseekV4ScalingRotaryEmbedding.__init__`，其签名为 `__init__(self, *args, **kwargs)`。它主要围绕 `*args`, `**kwargs` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `kwargs.pop`, `super.__init__`, `self._compute_cos_sin_cache`, `self.register_buffer`, `super`。

### Method `DeepseekV4ScalingRotaryEmbedding._compute_cos_sin_cache` (lines 250-261)
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        inv_freq = self._compute_inv_freq(self.scaling_factor)
        t = torch.arange(
            self.max_position_embeddings * self.scaling_factor,
            device=current_platform.device_type,
            dtype=torch.float32,
        )
        freqs = torch.einsum("i,j -> ij", t, inv_freq)
        cos = freqs.cos() * self.mscale
        sin = freqs.sin() * self.mscale
        cache = torch.cat((cos, sin), dim=-1)
        return cache
```
**EN:** Defines function `DeepseekV4ScalingRotaryEmbedding._compute_cos_sin_cache` with signature `_compute_cos_sin_cache(self) -> torch.Tensor`. It mainly works with object context only; adjusts rotary-position-encoding parameters or application logic. The body uses tensor/kernel operations. Key calls include `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `torch.cat`, `freqs.cos`, `freqs.sin`.
**CN:** 定义函数 `DeepseekV4ScalingRotaryEmbedding._compute_cos_sin_cache`，其签名为 `_compute_cos_sin_cache(self) -> torch.Tensor`。它主要围绕 仅依赖对象上下文 展开；调整旋转位置编码参数或其应用逻辑。函数体包含张量或内核操作。关键调用包括 `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `torch.cat`, `freqs.cos`, `freqs.sin`。

### Method `DeepseekV4ScalingRotaryEmbedding.forward_native` (lines 263-306)
```python
    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
        inverse: bool = False,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """PyTorch-native implementation equivalent to forward()."""

        head_size = query.size(-1)
        query_rot = query[..., -self.rotary_dim :]
        key_rot = key[..., -self.rotary_dim :] if key is not None else None

        if self.rotary_dim < head_size:
            query_pass = query[..., : -self.rotary_dim]
            key_pass = key[..., : -self.rotary_dim] if key is not None else None

        cos_sin = self.cos_sin_cache[
            torch.add(positions, offsets) if offsets is not None else positions
        ]
        cos, sin = cos_sin.chunk(2, dim=-1)
        if self.is_neox_style:
            cos = torch.cat((cos, cos), dim=-1).unsqueeze(-2)
            sin = torch.cat((sin, sin), dim=-1).unsqueeze(-2)
        else:
            cos = cos.repeat_interleave(2, dim=-1).unsqueeze(-2)
            sin = sin.repeat_interleave(2, dim=-1).unsqueeze(-2)
        if inverse:
            sin = -sin
        rotate_fn = rotate_neox if self.is_neox_style else rotate_gptj
        orig_dtype = query.dtype
        query_rot = (query_rot * cos + rotate_fn(query_rot) * sin).to(orig_dtype)
        if key_rot is not None:
            key_rot = (key_rot * cos + rotate_fn(key_rot) * sin).to(orig_dtype)

        if self.rotary_dim < head_size:
            query = torch.cat((query_pass, query_rot), dim=-1)
            key = torch.cat((key_pass, key_rot), dim=-1) if key is not None else None
        else:
            query = query_rot
            key = key_rot

        return query, key
```
**EN:** Defines function `DeepseekV4ScalingRotaryEmbedding.forward_native` with signature `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None, inverse: bool=False) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`, `inverse`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `query.size`, `cos_sin.chunk`, `to`, `torch.cat.unsqueeze`, `cos.repeat_interleave.unsqueeze`, `sin.repeat_interleave.unsqueeze`.
**CN:** 定义函数 `DeepseekV4ScalingRotaryEmbedding.forward_native`，其签名为 `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None, inverse: bool=False) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets`, `inverse` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `query.size`, `cos_sin.chunk`, `to`, `torch.cat.unsqueeze`, `cos.repeat_interleave.unsqueeze`, `sin.repeat_interleave.unsqueeze`。

### Method `DeepseekV4ScalingRotaryEmbedding.forward_hip` (lines 308-316)
```python
    def forward_hip(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
        inverse: bool = False,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        return self.forward_native(positions, query, key, offsets)
```
**EN:** Defines function `DeepseekV4ScalingRotaryEmbedding.forward_hip` with signature `forward_hip(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None, inverse: bool=False) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`, `inverse`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include `self.forward_native`.
**CN:** 定义函数 `DeepseekV4ScalingRotaryEmbedding.forward_hip`，其签名为 `forward_hip(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None, inverse: bool=False) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets`, `inverse` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.forward_native`。

### Method `DeepseekV4ScalingRotaryEmbedding.forward_cuda` (lines 318-344)
```python
    def forward_cuda(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
        inverse: bool = False,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        from vllm import _custom_ops as ops

        # The indexer and attention have different head_dim,
        # we obtain the corresponding head_dim via the query.
        head_size = query.size(-1)
        rope_dim_offset = head_size - self.rotary_dim
        # ops.rotary_embedding() is an in-place operation
        # that updates the query and key tensors.
        ops.rotary_embedding(
            torch.add(positions, offsets) if offsets is not None else positions,
            query,
            key,
            head_size,
            self.cos_sin_cache,
            self.is_neox_style,
            rope_dim_offset=rope_dim_offset,
            inverse=inverse,
        )
        return query, key
```
**EN:** Defines function `DeepseekV4ScalingRotaryEmbedding.forward_cuda` with signature `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None, inverse: bool=False) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`, `inverse`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `query.size`, `ops.rotary_embedding`, `torch.add`.
**CN:** 定义函数 `DeepseekV4ScalingRotaryEmbedding.forward_cuda`，其签名为 `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None, inverse: bool=False) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets`, `inverse` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `query.size`, `ops.rotary_embedding`, `torch.add`。

## Key Concepts / 关键概念
- **EN:** The file implements a RoPE-related variant, meaning frequency scaling, cache construction, and position application are key ideas.
  **CN:** 该文件实现了与 RoPE 相关的变体，因此频率缩放、缓存构建和位置应用是关键概念。
- **EN:** Top-level classes include `DeepseekScalingRotaryEmbedding`, `DeepseekV4ScalingRotaryEmbedding`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `DeepseekScalingRotaryEmbedding`, `DeepseekV4ScalingRotaryEmbedding`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `yarn_get_mscale` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `yarn_get_mscale` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `math`, `torch`
- **Internal / 内部**: `vllm.platforms`, `vllm.utils.flashinfer`, `.base`, `.common`
