# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/rotary_embedding/base.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `RotaryEmbeddingBase` and related helpers for rotary positional embedding variants. / 实现 `RotaryEmbeddingBase` 及其相关辅助逻辑，用于旋转位置编码变体。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-3)
```python
"""Rotary Positional Embeddings Base Class."""
```
**EN:** This docstring gives the module author's high-level intent: Rotary Positional Embeddings Base Class. It is useful for reading the rest of the file because later classes and helpers refine this stated purpose.
**CN:** 这个文档字符串给出了模块作者的高层意图：Rotary Positional Embeddings Base Class. 在阅读后续类和辅助函数时，可以把它视为整个文件的总纲。

### Imports and module setup (lines 5-10)
```python
import torch

from vllm._aiter_ops import rocm_aiter_ops
from vllm.model_executor.custom_op import CustomOp

from .common import ApplyRotaryEmb
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm._aiter_ops`, `vllm.model_executor.custom_op`, `.common`. That import mix shows the file is part of the rotary positional embedding variants stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm._aiter_ops`, `vllm.model_executor.custom_op`, `.common`）。这些导入关系表明该文件属于旋转位置编码变体栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Class `RotaryEmbeddingBase` overview (lines 15-115)
```python
class RotaryEmbeddingBase(CustomOp):
    """Original rotary positional embedding."""

    # --8<-- [end:rotary_embedding]

    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        init_cache: bool = True,
    ) -> None:
        super().__init__()
        self.head_size = head_size
        self.rotary_dim = rotary_dim
        self.max_position_embeddings = max_position_embeddings
        self.base = base
        self.is_neox_style = is_neox_style
        self.dtype = dtype
        # TODO(mgoin): disabled for now due to failures
        # Flashinfer only supports head_size=64, 128, 256, 512.
        # https://github.com/flashinfer-ai/flashinfer/blob/ebfd655efe830048dba5d582aaa61d61d1cf9a87/include/flashinfer/utils.cuh#L174-L202
```
**EN:** Defines class `RotaryEmbeddingBase` with base classes `CustomOp` and decorators `CustomOp.register('rotary_embedding')`. It acts as a rotary-position-encoding variant and exposes 5 direct methods, with notable entries `__init__`, `_compute_inv_freq`, `_compute_cos_sin_cache`, `_match_cos_sin_cache_dtype`, `get_cos_sin`. Its docstring says: Original rotary positional embedding.
**CN:** 定义类 `RotaryEmbeddingBase`，其基类为 `CustomOp`，装饰器为 `CustomOp.register('rotary_embedding')`。它在整体实现中充当旋转位置编码变体，并直接暴露 5 个方法，较重要的包括 `__init__`, `_compute_inv_freq`, `_compute_cos_sin_cache`, `_match_cos_sin_cache_dtype`, `get_cos_sin`。 文档字符串进一步说明了该类的定位。

### Method `RotaryEmbeddingBase.__init__` (lines 20-67)
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        init_cache: bool = True,
    ) -> None:
        super().__init__()
        self.head_size = head_size
        self.rotary_dim = rotary_dim
        self.max_position_embeddings = max_position_embeddings
        self.base = base
        self.is_neox_style = is_neox_style
        self.dtype = dtype
        # TODO(mgoin): disabled for now due to failures
        # Flashinfer only supports head_size=64, 128, 256, 512.
        # https://github.com/flashinfer-ai/flashinfer/blob/ebfd655efe830048dba5d582aaa61d61d1cf9a87/include/flashinfer/utils.cuh#L174-L202
        # self.use_flashinfer = (self.enabled()
        #                        and dtype in (torch.float16, torch.bfloat16)
        #                        and current_platform.is_cuda()
        #                        and has_flashinfer()
        #                        and self.head_size in [64, 128, 256, 512])

        # Check if use_flashinfer is already set
        if not hasattr(self, "use_flashinfer"):
            self.use_flashinfer = False

        self.use_aiter = (
            self.enabled() and rocm_aiter_ops.is_triton_rotary_embed_enabled()
        )
        if self.use_aiter:
            self.rocm_aiter_triton_rotary_embedding = (
                rocm_aiter_ops.get_triton_rotary_embedding_op()
            )

        if init_cache:
            cache = self._compute_cos_sin_cache()
            if not self.use_flashinfer:
                cache = cache.to(dtype)
            self.cos_sin_cache: torch.Tensor
            self.register_buffer("cos_sin_cache", cache, persistent=False)

        self.apply_rotary_emb = ApplyRotaryEmb(
            is_neox_style=self.is_neox_style,
        )
```
**EN:** Defines function `RotaryEmbeddingBase.__init__` with signature `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, init_cache: bool=True) -> None`. It mainly works with `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `init_cache`; initializes the object state and cached resources. The body uses branching. Key calls include `super.__init__`, `ApplyRotaryEmb`, `hasattr`, `self.enabled`, `rocm_aiter_ops.is_triton_rotary_embed_enabled`, `rocm_aiter_ops.get_triton_rotary_embedding_op`.
**CN:** 定义函数 `RotaryEmbeddingBase.__init__`，其签名为 `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, init_cache: bool=True) -> None`。它主要围绕 `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `init_cache` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断。关键调用包括 `super.__init__`, `ApplyRotaryEmb`, `hasattr`, `self.enabled`, `rocm_aiter_ops.is_triton_rotary_embed_enabled`, `rocm_aiter_ops.get_triton_rotary_embedding_op`。

### Method `RotaryEmbeddingBase._compute_inv_freq` (lines 69-81)
```python
    def _compute_inv_freq(self, base: float) -> torch.Tensor:
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
**EN:** Defines function `RotaryEmbeddingBase._compute_inv_freq` with signature `_compute_inv_freq(self, base: float) -> torch.Tensor`. It mainly works with `base`; adjusts rotary-position-encoding parameters or application logic. The body uses tensor/kernel operations. Key calls include `torch.arange`.
**CN:** 定义函数 `RotaryEmbeddingBase._compute_inv_freq`，其签名为 `_compute_inv_freq(self, base: float) -> torch.Tensor`。它主要围绕 `base` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含张量或内核操作。关键调用包括 `torch.arange`。

### Method `RotaryEmbeddingBase._compute_cos_sin_cache` (lines 83-92)
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
**EN:** Defines function `RotaryEmbeddingBase._compute_cos_sin_cache` with signature `_compute_cos_sin_cache(self) -> torch.Tensor`. It mainly works with object context only; adjusts rotary-position-encoding parameters or application logic. The body uses tensor/kernel operations. Key calls include `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `freqs.cos`, `freqs.sin`, `torch.cat`.
**CN:** 定义函数 `RotaryEmbeddingBase._compute_cos_sin_cache`，其签名为 `_compute_cos_sin_cache(self) -> torch.Tensor`。它主要围绕 仅依赖对象上下文 展开；调整旋转位置编码参数或其应用逻辑。函数体包含张量或内核操作。关键调用包括 `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `freqs.cos`, `freqs.sin`, `torch.cat`。

### Method `RotaryEmbeddingBase._match_cos_sin_cache_dtype` (lines 94-110)
```python
    def _match_cos_sin_cache_dtype(self, query: torch.Tensor) -> torch.Tensor:
        # __setattr__ in nn.Module (called by `self.cos_sin_cache = ...`)
        # is expensive, so avoid calling it if possible
        cos_sin_cache = self.cos_sin_cache
        if (
            cos_sin_cache.device == query.device
            and self.cos_sin_cache.dtype == query.dtype
        ):
            return cos_sin_cache

        cos_sin_cache = cos_sin_cache.to(query.device, dtype=query.dtype)
        # Avoid mutating buffers during torch.compile (cudagraph) tracing.
        if torch.compiler.is_compiling():
            return cos_sin_cache

        self.cos_sin_cache = cos_sin_cache
        return cos_sin_cache
```
**EN:** Defines function `RotaryEmbeddingBase._match_cos_sin_cache_dtype` with signature `_match_cos_sin_cache_dtype(self, query: torch.Tensor) -> torch.Tensor`. It mainly works with `query`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `cos_sin_cache.to`, `torch.compiler.is_compiling`.
**CN:** 定义函数 `RotaryEmbeddingBase._match_cos_sin_cache_dtype`，其签名为 `_match_cos_sin_cache_dtype(self, query: torch.Tensor) -> torch.Tensor`。它主要围绕 `query` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `cos_sin_cache.to`, `torch.compiler.is_compiling`。

### Method `RotaryEmbeddingBase.get_cos_sin` (lines 112-115)
```python
    def get_cos_sin(self, seqlen: int) -> tuple[torch.Tensor, torch.Tensor]:
        cos_sin = self.cos_sin_cache[:seqlen]
        cos, sin = cos_sin.chunk(2, dim=-1)
        return cos, sin
```
**EN:** Defines function `RotaryEmbeddingBase.get_cos_sin` with signature `get_cos_sin(self, seqlen: int) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `seqlen`; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `cos_sin.chunk`.
**CN:** 定义函数 `RotaryEmbeddingBase.get_cos_sin`，其签名为 `get_cos_sin(self, seqlen: int) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `seqlen` 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `cos_sin.chunk`。

### Class `RotaryEmbedding` overview (lines 118-303)
```python
class RotaryEmbedding(RotaryEmbeddingBase):
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        init_cache: bool = True,
    ) -> None:
        super().__init__(
            head_size=head_size,
            rotary_dim=rotary_dim,
            max_position_embeddings=max_position_embeddings,
            base=base,
            is_neox_style=is_neox_style,
            dtype=dtype,
            init_cache=init_cache,
        )

    @staticmethod
    def forward_static(
        positions: torch.Tensor,
        query: torch.Tensor,
```
**EN:** Defines class `RotaryEmbedding` with base classes `RotaryEmbeddingBase` and decorators none. It acts as an embedding layer with model-parallel awareness and exposes 8 direct methods, with notable entries `__init__`, `forward_static`, `forward_native`, `forward_cuda`, `forward_hip`, `forward_xpu`.
**CN:** 定义类 `RotaryEmbedding`，其基类为 `RotaryEmbeddingBase`，装饰器为 无。它在整体实现中充当具备模型并行意识的嵌入层，并直接暴露 8 个方法，较重要的包括 `__init__`, `forward_static`, `forward_native`, `forward_cuda`, `forward_hip`, `forward_xpu`。

### Method `RotaryEmbedding.__init__` (lines 119-137)
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        init_cache: bool = True,
    ) -> None:
        super().__init__(
            head_size=head_size,
            rotary_dim=rotary_dim,
            max_position_embeddings=max_position_embeddings,
            base=base,
            is_neox_style=is_neox_style,
            dtype=dtype,
            init_cache=init_cache,
        )
```
**EN:** Defines function `RotaryEmbedding.__init__` with signature `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, init_cache: bool=True) -> None`. It mainly works with `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `init_cache`; initializes the object state and cached resources. The body uses mostly straightforward data movement and object wiring. Key calls include `super.__init__`, `super`.
**CN:** 定义函数 `RotaryEmbedding.__init__`，其签名为 `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, init_cache: bool=True) -> None`。它主要围绕 `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `init_cache` 展开；负责初始化对象状态和缓存资源。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `super.__init__`, `super`。

### Method `RotaryEmbedding.forward_static` (lines 140-180)
```python
    def forward_static(
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None,
        head_size: int,
        rotary_dim: int,
        cos_sin_cache: torch.Tensor,
        is_neox_style: bool,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """A PyTorch-native implementation of forward()."""
        positions = positions.flatten()
        num_tokens = positions.shape[0]
        cos_sin = cos_sin_cache.index_select(0, positions)
        cos, sin = cos_sin.chunk(2, dim=-1)

        query_shape = query.shape
        query = query.view(num_tokens, -1, head_size)
        query_rot = query[..., :rotary_dim]
        query_pass = query[..., rotary_dim:]
        query_rot = ApplyRotaryEmb.forward_static(
            query_rot,
            cos,
            sin,
            is_neox_style,
        )
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        # key may be None in some cases, e.g. cross-layer KV sharing
        if key is not None:
            key_shape = key.shape
            key = key.view(num_tokens, -1, head_size)
            key_rot = key[..., :rotary_dim]
            key_pass = key[..., rotary_dim:]
            key_rot = ApplyRotaryEmb.forward_static(
                key_rot,
                cos,
                sin,
                is_neox_style,
            )
            key = torch.cat((key_rot, key_pass), dim=-1).reshape(key_shape)
        return query, key
```
**EN:** Defines function `RotaryEmbedding.forward_static` with signature `forward_static(positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None, head_size: int, rotary_dim: int, cos_sin_cache: torch.Tensor, is_neox_style: bool) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `head_size`, `rotary_dim`, `cos_sin_cache`, `is_neox_style`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `positions.flatten`, `cos_sin_cache.index_select`, `cos_sin.chunk`, `query.view`, `ApplyRotaryEmb.forward_static`, `torch.cat.reshape`.
**CN:** 定义函数 `RotaryEmbedding.forward_static`，其签名为 `forward_static(positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None, head_size: int, rotary_dim: int, cos_sin_cache: torch.Tensor, is_neox_style: bool) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `head_size`, `rotary_dim`, `cos_sin_cache`, `is_neox_style` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `positions.flatten`, `cos_sin_cache.index_select`, `cos_sin.chunk`, `query.view`, `ApplyRotaryEmb.forward_static`, `torch.cat.reshape`。

### Method `RotaryEmbedding.forward_cuda` (lines 200-231)
```python
    def forward_cuda(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        if self.use_flashinfer:
            torch.ops.vllm.flashinfer_rotary_embedding(
                positions,
                query,
                key,
                self.head_size,
                self.cos_sin_cache,
                self.is_neox_style,
            )
            return query, key

        from vllm import _custom_ops as ops

        cos_sin_cache = self._match_cos_sin_cache_dtype(query)

        # ops.rotary_embedding() is an in-place operation
        # that updates the query and key tensors.
        ops.rotary_embedding(
            positions,
            query,
            key,
            self.head_size,
            cos_sin_cache,
            self.is_neox_style,
        )
        return query, key
```
**EN:** Defines function `RotaryEmbedding.forward_cuda` with signature `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `self._match_cos_sin_cache_dtype`, `ops.rotary_embedding`, `torch.ops.vllm.flashinfer_rotary_embedding`.
**CN:** 定义函数 `RotaryEmbedding.forward_cuda`，其签名为 `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `self._match_cos_sin_cache_dtype`, `ops.rotary_embedding`, `torch.ops.vllm.flashinfer_rotary_embedding`。

### Method `RotaryEmbedding.forward_hip` (lines 233-250)
```python
    def forward_hip(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        if self.use_aiter:
            cos_sin_cache = self._match_cos_sin_cache_dtype(query)
            self.rocm_aiter_triton_rotary_embedding(
                positions,
                query,
                key,
                self.head_size,
                cos_sin_cache,
                self.is_neox_style,
            )
            return query, key
        return self.forward_cuda(positions, query, key)
```
**EN:** Defines function `RotaryEmbedding.forward_hip` with signature `forward_hip(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `self.forward_cuda`, `self._match_cos_sin_cache_dtype`, `self.rocm_aiter_triton_rotary_embedding`.
**CN:** 定义函数 `RotaryEmbedding.forward_hip`，其签名为 `forward_hip(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `self.forward_cuda`, `self._match_cos_sin_cache_dtype`, `self.rocm_aiter_triton_rotary_embedding`。

### Method `RotaryEmbedding.forward_xpu` (lines 252-275)
```python
    def forward_xpu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        self._match_cos_sin_cache_dtype(query)
        # ops.rotary_embedding() is an in-place operation
        # that updates the query and key tensors.
        if key is None:
            return self.forward_native(positions, query, key)
        else:
            from vllm import _custom_ops as ops

            cos_sin_cache = self._match_cos_sin_cache_dtype(query)
            ops.rotary_embedding(
                positions,
                query,
                key,
                self.head_size,
                cos_sin_cache,
                self.is_neox_style,
            )
        return query, key
```
**EN:** Defines function `RotaryEmbedding.forward_xpu` with signature `forward_xpu(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `self._match_cos_sin_cache_dtype`, `self.forward_native`, `ops.rotary_embedding`.
**CN:** 定义函数 `RotaryEmbedding.forward_xpu`，其签名为 `forward_xpu(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `self._match_cos_sin_cache_dtype`, `self.forward_native`, `ops.rotary_embedding`。

### Method `RotaryEmbedding.forward_cpu` (lines 277-297)
```python
    def forward_cpu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        from vllm import _custom_ops as ops

        cos_sin_cache = self._match_cos_sin_cache_dtype(query)

        # ops.rotary_embedding() is an in-place operation
        # that updates the query and key tensors.
        ops.rotary_embedding(
            positions,
            query,
            key,
            self.head_size,
            cos_sin_cache,
            self.is_neox_style,
        )
        return query, key
```
**EN:** Defines function `RotaryEmbedding.forward_cpu` with signature `forward_cpu(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`; adjusts rotary-position-encoding parameters or application logic. The body uses tensor/kernel operations. Key calls include `self._match_cos_sin_cache_dtype`, `ops.rotary_embedding`.
**CN:** 定义函数 `RotaryEmbedding.forward_cpu`，其签名为 `forward_cpu(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含张量或内核操作。关键调用包括 `self._match_cos_sin_cache_dtype`, `ops.rotary_embedding`。

## Key Concepts / 关键概念
- **EN:** The file implements a RoPE-related variant, meaning frequency scaling, cache construction, and position application are key ideas.
  **CN:** 该文件实现了与 RoPE 相关的变体，因此频率缩放、缓存构建和位置应用是关键概念。
- **EN:** Top-level classes include `RotaryEmbeddingBase`, `RotaryEmbedding`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `RotaryEmbeddingBase`, `RotaryEmbedding`，它们组织了主要的可复用抽象。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm._aiter_ops`, `vllm.model_executor.custom_op`, `.common`
