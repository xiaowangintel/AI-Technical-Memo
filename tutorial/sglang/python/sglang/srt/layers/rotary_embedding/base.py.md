# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/rotary_embedding/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements linear projections and GEMM backend integration with rotary position embedding logic for the SGLang SRT runtime. It exposes symbols such as `RotaryEmbedding` and `LinearScalingRotaryEmbedding` and connects them to backend-specific paths such as `CUDA`, `NPU`, `XPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了线性投影与 GEMM 后端集成，并结合旋转位置编码逻辑。它提供了 `RotaryEmbedding` 和 `LinearScalingRotaryEmbedding` 等符号，并把这些符号连接到 `CUDA`、`NPU`、`XPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-49: Imports, conditional backend setup, and runtime guards
```python
"""RotaryEmbedding base class + LinearScalingRotaryEmbedding."""

from __future__ import annotations

from typing import TYPE_CHECKING, Dict, List, Optional, Tuple, Union

import torch

from sglang.srt.layers.rotary_embedding.utils import apply_rotary_emb
from sglang.srt.layers.utils import MultiPlatformOp
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import (
    cpu_has_amx_support,
    get_bool_env_var,
    is_cpu,
    is_cuda,
    is_hip,
    is_mps,
    is_musa,
    is_npu,
    is_xpu,
)

if TYPE_CHECKING:
    from sglang.jit_kernel.rope import FusedSetKVBufferArg  # For type check-only

_is_cuda = is_cuda()
_is_hip = is_hip()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
_is_npu = is_npu()
_is_cpu_amx_available = cpu_has_amx_support()
_is_cpu = is_cpu()
_is_xpu = is_xpu()
_is_musa = is_musa()
_is_mps = is_mps()

if _is_cuda:
    from sglang.jit_kernel.rope import apply_rope_with_cos_sin_cache_inplace

if _is_npu:
    import torch_npu
    from sgl_kernel_npu.norm.fused_rope_qk_mqa import fused_rope_qk_mqa

if _is_hip:
    from sglang.srt.layers.attention.utils import (
        fused_qk_rope_reshape_and_cache,
    )
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `typing.TYPE_CHECKING`, `typing.Dict`, `typing.List`, `typing.Optional`, and `typing.Tuple`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_cuda`, `_is_hip`, `_use_aiter`, `_is_npu`, and `_is_cpu_amx_available` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`typing.TYPE_CHECKING`、`typing.Dict`、`typing.List`、`typing.Optional` 以及 `typing.Tuple`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_cuda`、`_is_hip`、`_use_aiter`、`_is_npu` 以及 `_is_cpu_amx_available` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 50-52: Class `RotaryEmbedding` declaration and shared state
```python
class RotaryEmbedding(MultiPlatformOp):
    """Original rotary positional embedding."""
```
**EN:** This block introduces class `RotaryEmbedding` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Original rotary positional embedding.
**CN:** 该代码块引入类 `RotaryEmbedding`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 53-109: `RotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
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
        # NOTE(ByronHsu): cache needs to be in FP32 for numerical stability
        if not _is_cuda:
            cache = cache.to(dtype)

        if (
            (not (_is_cuda) or self.head_size not in [64, 128, 256, 512])
            and not (_is_cpu)
            and not (_is_xpu)
            and not (_is_npu)
            and not (_is_musa)
            and not (_is_mps)
        ):
            # rotary_embedding from sglang.jit_kernel.rope and vllm._custom_ops has the same implementation.
            # TODO: Test on different devices and remove this conditional.
            if _is_cuda:
                from sglang.jit_kernel.rope import rotary_embedding
            elif _is_hip:
                from sgl_kernel import rotary_embedding
            else:
                from vllm._custom_ops import rotary_embedding

            self.use_fallback_kernel = True
            self.fallback_rotary_embedding = rotary_embedding
        else:
            self.use_fallback_kernel = False

        self.cos_sin_cache: torch.Tensor
        self.register_buffer("cos_sin_cache", cache, persistent=False)

        self._apply_rotary_emb_wrapped = apply_rotary_emb

        # XXX (MUSA): Implement sgl_kernel.rotary_embedding support for MUSA backend
        if get_global_server_args().rl_on_policy_target is not None or _is_musa:
            self._forward_method = self.forward_native
            self._apply_rotary_emb_wrapped = torch.compile(dynamic=True)(
                apply_rotary_emb
            )
        self.position_cos, self.position_sin = None, None
```
**EN:** This block defines `RotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `self._compute_cos_sin_cache`, `self.register_buffer`, `cache.to`, and `torch.compile`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.head_size`, `self.rotary_dim`, `self.max_position_embeddings`, `self.base`, and `self.is_neox_style` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`self._compute_cos_sin_cache`、`self.register_buffer`、`cache.to` 以及 `torch.compile`，说明该流程会编排底层辅助函数或计算内核。 像 `self.head_size`、`self.rotary_dim`、`self.max_position_embeddings`、`self.base` 以及 `self.is_neox_style` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 110-118: Internal helper `RotaryEmbedding._match_cos_sin_cache_dtype`
```python
    def _match_cos_sin_cache_dtype(self, query: torch.Tensor) -> None:
        # __setattr__ in nn.Module (called by `self.cos_sin_cache = ...`)
        # is expensive, so avoid calling it if possible
        if (
            self.cos_sin_cache.device != query.device
            or self.cos_sin_cache.dtype != query.dtype
        ):
            self.cos_sin_cache = self.cos_sin_cache.to(query.device, dtype=query.dtype)
```
**EN:** This block defines `RotaryEmbedding._match_cos_sin_cache_dtype` and contains the main logic for this step. It mainly invokes `self.cos_sin_cache.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.cos_sin_cache` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RotaryEmbedding._match_cos_sin_cache_dtype`，并承载这一阶段的核心逻辑。 它主要调用 `self.cos_sin_cache.to`，说明该流程会编排底层辅助函数或计算内核。 像 `self.cos_sin_cache` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 119-140: Internal helper `RotaryEmbedding._compute_inv_freq`
```python
    def _compute_inv_freq(self, base: Union[int, float]) -> torch.Tensor:
        """Compute the inverse frequency."""
        # NOTE(woosuk): To exactly match the HF implementation, we need to
        # use CPU to compute the cache and then move it to GPU. However, we
        # create the cache on GPU for faster initialization. This may cause
        # a slight numerical difference between the HF implementation and ours.
        init_device = (
            "cpu" if get_global_server_args().rl_on_policy_target is not None else None
        )
        inv_freq = 1.0 / (
            base
            ** (
                torch.arange(
                    0, self.rotary_dim, 2, dtype=torch.float, device=init_device
                )
                / self.rotary_dim
            )
        )
        if get_global_server_args().rl_on_policy_target is not None:
            inv_freq = inv_freq.cuda()
        return inv_freq
```
**EN:** This block defines `RotaryEmbedding._compute_inv_freq` and contains the main logic for this step. It mainly invokes `inv_freq.cuda`, `get_global_server_args`, and `torch.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `init_device` and `inv_freq` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RotaryEmbedding._compute_inv_freq`，并承载这一阶段的核心逻辑。 它主要调用 `inv_freq.cuda`、`get_global_server_args` 以及 `torch.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `init_device` 和 `inv_freq` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 141-151: Internal helper `RotaryEmbedding._compute_cos_sin_cache`
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
**EN:** This block defines `RotaryEmbedding._compute_cos_sin_cache` and contains the main logic for this step. It mainly invokes `self._compute_inv_freq`, `torch.arange`, `torch.einsum`, `freqs.cos`, and `freqs.sin`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freq`, `t`, `freqs`, `cos`, and `sin` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RotaryEmbedding._compute_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `self._compute_inv_freq`、`torch.arange`、`torch.einsum`、`freqs.cos` 以及 `freqs.sin`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freq`、`t`、`freqs`、`cos` 以及 `sin` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 152-184: Internal helper `RotaryEmbedding._ensure_cos_sin_cache_length`
```python
    def _ensure_cos_sin_cache_length(self, needed_max_pos: int):
        """Ensure cos_sin_cache length > needed_max_pos."""
        from sglang.srt.environ import envs

        cur_len = int(self.cos_sin_cache.shape[0])
        if needed_max_pos < cur_len:
            return

        # Align to reduce realloc frequency
        align = envs.SGLANG_ROPE_CACHE_ALIGN.get()
        new_len = ((needed_max_pos + align) // align) * align
        device = self.cos_sin_cache.device
        dtype = self.cos_sin_cache.dtype

        # Compute inv_freq on same device
        inv_freq = self._compute_inv_freq(self.base).to(device=device)

        # Incremental computation for new positions only
        start = cur_len
        t_new = torch.arange(start, new_len, dtype=inv_freq.dtype, device=device)
        if t_new.numel() == 0:
            return

        freqs_new = torch.einsum("i,j->ij", t_new, inv_freq)
        cos_new = freqs_new.cos()
        sin_new = freqs_new.sin()
        new_rows = torch.cat((cos_new, sin_new), dim=-1).to(dtype=dtype)

        # Update cache with new rows
        self.cos_sin_cache = torch.cat((self.cos_sin_cache, new_rows), dim=0).to(
            device=device, dtype=dtype
        )
```
**EN:** This block defines `RotaryEmbedding._ensure_cos_sin_cache_length` and contains the main logic for this step. It mainly invokes `int`, `envs.SGLANG_ROPE_CACHE_ALIGN.get`, `self._compute_inv_freq.to`, `torch.arange`, and `torch.einsum`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cur_len`, `align`, `new_len`, `device`, and `dtype` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RotaryEmbedding._ensure_cos_sin_cache_length`，并承载这一阶段的核心逻辑。 它主要调用 `int`、`envs.SGLANG_ROPE_CACHE_ALIGN.get`、`self._compute_inv_freq.to`、`torch.arange` 以及 `torch.einsum`，说明该流程会编排底层辅助函数或计算内核。 像 `cur_len`、`align`、`new_len`、`device` 以及 `dtype` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 185-200: `RotaryEmbedding.get_cos_sin_with_position` getter for cos sin with position
```python
    def get_cos_sin_with_position(self, positions):
        assert positions.ndim == 1, (
            "2D positions (multimodal RoPE) are not supported by the base "
            "RotaryEmbedding. Override this method in a subclass (e.g. MRotaryEmbedding)."
        )
        cos_sin = self.cos_sin_cache.index_select(0, positions.flatten())
        last_dim = cos_sin.size()[-1]
        cos, sin = (
            cos_sin.reshape(-1, 2, last_dim // 2).repeat(1, 1, 2).chunk(2, dim=-2)
        )
        # BSNH
        self.position_cos, self.position_sin = (
            cos.view(-1, 1, 1, last_dim).contiguous(),
            sin.view(-1, 1, 1, last_dim).contiguous(),
        )
```
**EN:** This block defines `RotaryEmbedding.get_cos_sin_with_position` and contains the main logic for this step. It mainly invokes `self.cos_sin_cache.index_select`, `cos_sin.reshape.repeat.chunk`, `positions.flatten`, `cos_sin.size`, and `cos.view.contiguous`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cos_sin`, `last_dim`, `cos`, `sin`, and `self.position_cos` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RotaryEmbedding.get_cos_sin_with_position`，并承载这一阶段的核心逻辑。 它主要调用 `self.cos_sin_cache.index_select`、`cos_sin.reshape.repeat.chunk`、`positions.flatten`、`cos_sin.size` 以及 `cos.view.contiguous`，说明该流程会编排底层辅助函数或计算内核。 像 `cos_sin`、`last_dim`、`cos`、`sin` 以及 `self.position_cos` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 201-205: `RotaryEmbedding.get_cos_sin` getter for cos sin
```python
    def get_cos_sin(self, seqlen: int) -> tuple[torch.Tensor, torch.Tensor]:
        cos_sin = self.cos_sin_cache[:seqlen]
        cos, sin = cos_sin.chunk(2, dim=-1)
        return cos, sin
```
**EN:** This block defines `RotaryEmbedding.get_cos_sin` and contains the main logic for this step. It mainly invokes `cos_sin.chunk`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cos_sin`, `cos`, and `sin` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RotaryEmbedding.get_cos_sin`，并承载这一阶段的核心逻辑。 它主要调用 `cos_sin.chunk`，说明该流程会编排底层辅助函数或计算内核。 像 `cos_sin`、`cos` 以及 `sin` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 206-247: `RotaryEmbedding.forward_native` NATIVE execution path
```python
    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
        fused_set_kv_buffer_arg: Optional[FusedSetKVBufferArg] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        """A PyTorch-native implementation of forward()."""
        assert (
            fused_set_kv_buffer_arg is None
        ), "fused_set_kv_buffer_arg is not supported for native implementation"

        if offsets is not None:
            positions = positions + offsets

        positions = positions.flatten()
        num_tokens = positions.shape[0]

        if hasattr(self, "sin_cos_cache"):
            cos_sin = self.sin_cos_cache
        else:
            cos_sin = self.cos_sin_cache.index_select(0, positions)
        cos, sin = cos_sin.chunk(2, dim=-1)

        query_shape = query.shape
        query = query.view(num_tokens, -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = self._apply_rotary_emb_wrapped(
            query_rot, cos, sin, self.is_neox_style
        )
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        key_shape = key.shape
        key = key.view(num_tokens, -1, self.head_size)
        key_rot = key[..., : self.rotary_dim]
        key_pass = key[..., self.rotary_dim :]
        key_rot = self._apply_rotary_emb_wrapped(key_rot, cos, sin, self.is_neox_style)
        key = torch.cat((key_rot, key_pass), dim=-1).reshape(key_shape)
        return query, key
```
**EN:** This block defines `RotaryEmbedding.forward_native` and contains the main logic for this step. It mainly invokes `positions.flatten`, `hasattr`, `cos_sin.chunk`, `query.view`, and `self._apply_rotary_emb_wrapped`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `positions`, `num_tokens`, `cos`, `sin`, and `query_shape` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RotaryEmbedding.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `positions.flatten`、`hasattr`、`cos_sin.chunk`、`query.view` 以及 `self._apply_rotary_emb_wrapped`，说明该流程会编排底层辅助函数或计算内核。 像 `positions`、`num_tokens`、`cos`、`sin` 以及 `query_shape` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 248-307: `RotaryEmbedding.forward_npu` NPU execution path
```python
    def forward_npu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
        fused_set_kv_buffer_arg: Optional[FusedSetKVBufferArg] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        """A PyTorch-npu implementation of forward()."""
        assert (
            fused_set_kv_buffer_arg is None
        ), "fused_set_kv_buffer_arg is not supported for npu implementation"
        if (
            query.dtype == torch.bfloat16
            and self.cos_sin_cache.dtype == torch.float
            or key.ndim == 3
        ):
            if hasattr(self, "sin_cos_cache"):
                cos_sin = self.sin_cos_cache
            else:
                cos_sin = self.cos_sin_cache.index_select(0, positions)

            if query.shape[0] * query.shape[1] < 65535:
                return fused_rope_qk_mqa(
                    query,
                    key,
                    cos_sin,
                    self.rotary_dim,
                    self.is_neox_style,
                )
            else:
                return self.forward_native(positions, query, key, offsets)
        if self.is_neox_style:
            rotary_mode = "half"
        else:
            rotary_mode = "interleave"

        mrope_section = [0, 0, 0]
        # The npu_mrope kernel only supports 1D or 2D tensors for query and key.
        # Therefore, when their dimensions exceed 2D, we flatten query and key to 2D tensors before computation
        # and reshape their original shapes afterward.
        query_shape = query.shape
        key_shape = key.shape
        query = query.reshape(query.shape[0], -1)
        key = key.reshape(key.shape[0], -1)

        query_out, key_out = torch_npu.npu_mrope(
            positions,
            query,
            key,
            self.cos_sin_cache,
            self.head_size,
            mrope_section=mrope_section,
            rotary_mode=rotary_mode,
        )

        query_out = query_out.reshape(query_shape)
        key_out = key_out.reshape(key_shape)
        return query_out, key_out
```
**EN:** This block defines `RotaryEmbedding.forward_npu` and contains the main logic for this step. It mainly invokes `query.reshape`, `key.reshape`, `torch_npu.npu_mrope`, `query_out.reshape`, and `key_out.reshape`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `mrope_section`, `query_shape`, `key_shape`, `query`, and `key` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RotaryEmbedding.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `query.reshape`、`key.reshape`、`torch_npu.npu_mrope`、`query_out.reshape` 以及 `key_out.reshape`，说明该流程会编排底层辅助函数或计算内核。 像 `mrope_section`、`query_shape`、`key_shape`、`query` 以及 `key` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 308-334: `RotaryEmbedding.forward_cpu` CPU execution path
```python
    def forward_cpu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
        fused_set_kv_buffer_arg: Optional[FusedSetKVBufferArg] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        assert (
            fused_set_kv_buffer_arg is None
        ), "fused_set_kv_buffer_arg is not supported for cpu implementation"

        positions = torch.add(positions, offsets) if offsets is not None else positions
        if _is_cpu_amx_available:
            return torch.ops.sgl_kernel.rotary_embedding_cpu(
                positions,
                query,
                key,
                self.head_size,
                self.cos_sin_cache,
                self.is_neox_style,
            )
        else:
            return self.forward_native(
                positions, query, key, offsets, fused_set_kv_buffer_arg
            )
```
**EN:** This block defines `RotaryEmbedding.forward_cpu` and contains the main logic for this step. It mainly invokes `torch.add`, `torch.ops.sgl_kernel.rotary_embedding_cpu`, and `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `positions` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RotaryEmbedding.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.add`、`torch.ops.sgl_kernel.rotary_embedding_cpu` 以及 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 像 `positions` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 335-402: `RotaryEmbedding.forward_cuda` CUDA execution path
```python
    def forward_cuda(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
        fused_set_kv_buffer_arg: Optional[Union[FusedSetKVBufferArg, dict]] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        if not self.use_fallback_kernel:
            batch_size = positions.size(0)
            q_rope = query.view(batch_size, -1, self.head_size)
            k_rope = key.view(batch_size, -1, self.head_size)
            if self.head_size != self.rotary_dim:
                q_rope = q_rope[..., : self.rotary_dim]
                k_rope = k_rope[..., : self.rotary_dim]
            apply_rope_with_cos_sin_cache_inplace(
                positions=positions,
                q=q_rope,
                k=k_rope,
                cos_sin_cache=self.cos_sin_cache,
                is_neox=self.is_neox_style,
                fused_args=fused_set_kv_buffer_arg,
            )
        else:

            if fused_set_kv_buffer_arg is not None and _is_hip:
                extra_args = fused_set_kv_buffer_arg

                k_cache_shape = fused_set_kv_buffer_arg["key_cache"].shape
                qk_head_dim = k_cache_shape[-1]
                tp_k_head_num = k_cache_shape[-2]

                key = key.view(-1, tp_k_head_num, qk_head_dim)

                tokens = key.shape[0]

                query = query.view(tokens, -1, qk_head_dim)

                query, key, k_cache, v_cache = fused_qk_rope_reshape_and_cache(
                    q=query,
                    k=key,
                    pos=positions,
                    cos_sin=self.cos_sin_cache,
                    is_neox=self.is_neox_style,
                    flash_layout=True,
                    offs=None,
                    q_out=query,
                    k_out=key,
                    output_zeros=False,
                    **extra_args,
                )
            else:
                assert (
                    fused_set_kv_buffer_arg is None
                ), "save kv cache is not supported for fallback_rotary_embedding."
                self.cos_sin_cache = self.cos_sin_cache.to(
                    query.device, dtype=query.dtype
                )
                self.fallback_rotary_embedding(
                    positions,
                    query,
                    key,
                    self.head_size,
                    self.cos_sin_cache,
                    self.is_neox_style,
                )
        return query, key
```
**EN:** This block defines `RotaryEmbedding.forward_cuda` and contains the main logic for this step. It mainly invokes `positions.size`, `query.view`, `key.view`, `apply_rope_with_cos_sin_cache_inplace`, and `fused_qk_rope_reshape_and_cache`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `batch_size`, `q_rope`, `k_rope`, `extra_args`, and `k_cache_shape` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RotaryEmbedding.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `positions.size`、`query.view`、`key.view`、`apply_rope_with_cos_sin_cache_inplace` 以及 `fused_qk_rope_reshape_and_cache`，说明该流程会编排底层辅助函数或计算内核。 像 `batch_size`、`q_rope`、`k_rope`、`extra_args` 以及 `k_cache_shape` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 403-408: Function `RotaryEmbedding.extra_repr` and its core logic
```python
    def extra_repr(self) -> str:
        s = f"head_size={self.head_size}, rotary_dim={self.rotary_dim}"
        s += f", max_position_embeddings={self.max_position_embeddings}"
        s += f", base={self.base}, is_neox_style={self.is_neox_style}"
        return s
```
**EN:** This block defines `RotaryEmbedding.extra_repr` and contains the main logic for this step. Intermediate names such as `s` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RotaryEmbedding.extra_repr`，并承载这一阶段的核心逻辑。 像 `s` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 409-431: `RotaryEmbedding.forward_xpu` XPU execution path
```python
    def forward_xpu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
        fused_set_kv_buffer_arg: Optional[FusedSetKVBufferArg] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        assert (
            fused_set_kv_buffer_arg is None
        ), "fused_set_kv_buffer_arg is not supported for xpu implementation"
        positions = torch.add(positions, offsets) if offsets is not None else positions

        return torch.ops.sgl_kernel.rotary_embedding(
            positions,
            query,
            key,
            self.head_size,
            self.cos_sin_cache,
            self.is_neox_style,
        )
```
**EN:** This block defines `RotaryEmbedding.forward_xpu` and contains the main logic for this step. It mainly invokes `torch.ops.sgl_kernel.rotary_embedding` and `torch.add`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `positions` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RotaryEmbedding.forward_xpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.ops.sgl_kernel.rotary_embedding` 和 `torch.add`，说明该流程会编排底层辅助函数或计算内核。 像 `positions` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 432-460: Class `LinearScalingRotaryEmbedding` declaration and shared state
```python
class LinearScalingRotaryEmbedding(RotaryEmbedding):
    """RotaryEmbedding extended with linear scaling.

    It supports multiple scaling factors. Since multiple LoRA adapters may have
    different scaling factors, we need multiple cos/sin caches. In this way,
    instead of running rotary embedding kernel per lora, we can run multiple
    lora in a batched way.

    In addition to that, we also keep the cos/sin cache for the scaling factor
    of 1 (default) at all times.

    Exemplary for two scaling factors x=1, y and z with embeddings
    [[x11, x12, ... x1m], ..., [xn1, xn2, ..., xnm]] and
    [[y11, y12, ... y1o], ..., [yn1, yn2, ..., yno]], and
    [[z11, z12, ... z1p], ..., [zn1, zn2, ..., znp]],

    we construct the cos/sin cache as follows:
    [[x11, x12, ... x1m, y11, y12, ... y1o, z11, z12, ... z1p],
        ...
     [xn1, xn2, ... xnm, yn1, yn2, ... yno, zn1, zn2, ... znp]]

    We then use offsets to index into the cos/sin cache for
    the respective scaling factors.

    The offset to cache can be accessed via `scaling_factor_to_offset` API.

    Credits to the Reddit user /u/kaiokendev
    """
```
**EN:** This block introduces class `LinearScalingRotaryEmbedding` and the state shared by its methods. It inherits from `RotaryEmbedding`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: RotaryEmbedding extended with linear scaling.
**CN:** 该代码块引入类 `LinearScalingRotaryEmbedding`，并定义其方法共享的状态。 它继承自 `RotaryEmbedding`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 461-479: `LinearScalingRotaryEmbedding` initialization and state setup
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        scaling_factors: Union[List[float], float],
        dtype: torch.dtype,
    ) -> None:
        if isinstance(scaling_factors, float):
            scaling_factors = [scaling_factors]
        self.scaling_factors: List[float] = scaling_factors  # noqa
        super().__init__(
            head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype
        )
        # Lazy initialized.
        self._scaling_factor_to_offset: Dict[float, int]
```
**EN:** This block defines `LinearScalingRotaryEmbedding.__init__` and contains the main logic for this step. It mainly invokes `isinstance` and `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.scaling_factors`, `self._scaling_factor_to_offset`, and `scaling_factors` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `LinearScalingRotaryEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance` 和 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self.scaling_factors`、`self._scaling_factor_to_offset` 以及 `scaling_factors` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 480-513: Internal helper `LinearScalingRotaryEmbedding._compute_cos_sin_cache`
```python
    def _compute_cos_sin_cache(self) -> torch.Tensor:
        inv_freq = self._compute_inv_freq(self.base)
        cache_list: List[torch.Tensor] = []
        # offsets to the next cache in a tensor.
        # Each offset corresponds to the same index in scaling_factors.
        offsets: List[int] = []
        for scaling_factor in self.scaling_factors:
            # NOTE(woosuk): self.max_position_embeddings is the original
            # maximum length before applying the rope scaling.
            # Thus, the maximum length after applying the rope scaling is
            # self.max_position_embeddings * self.scaling_factor.
            max_len = self.max_position_embeddings * scaling_factor
            t = torch.arange(max_len, dtype=torch.float)
            t = t / scaling_factor

            freqs = torch.einsum("i,j -> ij", t, inv_freq)
            cos = freqs.cos()
            sin = freqs.sin()
            cache = torch.cat((cos, sin), dim=-1)
            if not cache_list:
                offset = 0
            else:
                last_offset = offsets[-1]
                next_max_len = cache_list[-1].shape[0]
                offset = last_offset + next_max_len
            offsets.append(offset)
            cache_list.append(cache)
        self._scaling_factor_to_offset = {
            float(scaling_factor): offsets[i]
            for i, scaling_factor in enumerate(self.scaling_factors)
        }
        assert len(self.scaling_factors) == len(offsets)
        return torch.cat(cache_list, dim=0)
```
**EN:** This block defines `LinearScalingRotaryEmbedding._compute_cos_sin_cache` and contains the main logic for this step. It mainly invokes `self._compute_inv_freq`, `torch.cat`, `torch.arange`, `torch.einsum`, and `freqs.cos`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inv_freq`, `cache_list`, `offsets`, `self._scaling_factor_to_offset`, and `max_len` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LinearScalingRotaryEmbedding._compute_cos_sin_cache`，并承载这一阶段的核心逻辑。 它主要调用 `self._compute_inv_freq`、`torch.cat`、`torch.arange`、`torch.einsum` 以及 `freqs.cos`，说明该流程会编排底层辅助函数或计算内核。 像 `inv_freq`、`cache_list`、`offsets`、`self._scaling_factor_to_offset` 以及 `max_len` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 514-516: Function `LinearScalingRotaryEmbedding.scaling_factor_to_offset` and its core logic
```python
    @property
    def scaling_factor_to_offset(self) -> Dict[float, int]:
        return self._scaling_factor_to_offset
```
**EN:** This block defines `LinearScalingRotaryEmbedding.scaling_factor_to_offset` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LinearScalingRotaryEmbedding.scaling_factor_to_offset`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `RotaryEmbedding` and `LinearScalingRotaryEmbedding`. / **主要符号**：核心入口包括 `RotaryEmbedding` 和 `LinearScalingRotaryEmbedding`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `typing.TYPE_CHECKING`, `typing.Dict`, `typing.List`, `typing.Optional`, `typing.Tuple`, and `typing.Union` / **标准库**：`__future__.annotations`、`typing.TYPE_CHECKING`、`typing.Dict`、`typing.List`、`typing.Optional`、`typing.Tuple` 以及 `typing.Union`
- **Third-party**: `torch`, `torch_npu`, `sgl_kernel_npu.norm.fused_rope_qk_mqa.fused_rope_qk_mqa`, `sgl_kernel.rotary_embedding`, and `vllm._custom_ops.rotary_embedding` / **第三方依赖**：`torch`、`torch_npu`、`sgl_kernel_npu.norm.fused_rope_qk_mqa.fused_rope_qk_mqa`、`sgl_kernel.rotary_embedding` 以及 `vllm._custom_ops.rotary_embedding`
- **Internal SGLang modules**: `sglang.srt.layers.rotary_embedding.utils.apply_rotary_emb`, `sglang.srt.layers.utils.MultiPlatformOp`, `sglang.srt.server_args.get_global_server_args`, `sglang.srt.utils.cpu_has_amx_support`, `sglang.srt.utils.get_bool_env_var`, `sglang.srt.utils.is_cpu`, `sglang.srt.utils.is_cuda`, `sglang.srt.utils.is_hip`, `sglang.srt.utils.is_mps`, `sglang.srt.utils.is_musa`, `sglang.srt.utils.is_npu`, and `sglang.srt.utils.is_xpu` / **SGLang 内部模块**：`sglang.srt.layers.rotary_embedding.utils.apply_rotary_emb`、`sglang.srt.layers.utils.MultiPlatformOp`、`sglang.srt.server_args.get_global_server_args`、`sglang.srt.utils.cpu_has_amx_support`、`sglang.srt.utils.get_bool_env_var`、`sglang.srt.utils.is_cpu`、`sglang.srt.utils.is_cuda`、`sglang.srt.utils.is_hip`、`sglang.srt.utils.is_mps`、`sglang.srt.utils.is_musa`、`sglang.srt.utils.is_npu` 以及 `sglang.srt.utils.is_xpu`
