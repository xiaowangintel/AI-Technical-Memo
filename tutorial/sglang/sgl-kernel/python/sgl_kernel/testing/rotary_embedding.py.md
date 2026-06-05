# rotary_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/testing/rotary_embedding.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `rotary embedding`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `rotary embedding` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Imports and module setup
````python
from dataclasses import dataclass
from typing import Optional, Tuple, Union

import torch

from sglang.jit_kernel.rope import FusedSetKVBufferArg as _JitFusedSetKVBufferArg
from sglang.jit_kernel.rope import (
    apply_rope_with_cos_sin_cache_inplace as _jit_apply_rope_with_cos_sin_cache_inplace,
)
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 12-28: `FusedSetKVBufferArg` definition
````python
@dataclass
class FusedSetKVBufferArg:
    value: torch.Tensor
    k_buffer: torch.Tensor
    v_buffer: torch.Tensor
    cache_loc: torch.Tensor
    # Kept for backward compatibility with old sgl_kernel test/bench callsites.
    k_scale: Optional[float] = None
    v_scale: Optional[float] = None

    def to_jit(self) -> _JitFusedSetKVBufferArg:
        return _JitFusedSetKVBufferArg(
            value=self.value,
            k_buffer=self.k_buffer,
            v_buffer=self.v_buffer,
            cache_loc=self.cache_loc,
        )
````
**EN:** This section defines the class `FusedSetKVBufferArg`, grouping related state and behavior around fused Set KVBuffer Arg.
**CN:** 该部分定义类 `FusedSetKVBufferArg`，把与 fused Set KVBuffer Arg 相关的状态和行为组织在一起。

### Lines 29-31: Comments and local context
````python


# vLLM torch native
````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 32-58: `_apply_rotary_emb` definition
````python
def _apply_rotary_emb(
    x: torch.Tensor,
    cos: torch.Tensor,
    sin: torch.Tensor,
    is_neox_style: bool,
) -> torch.Tensor:
    """
    Args:
        x: [num_tokens, num_heads, head_size]
        cos: [num_tokens, head_size // 2]
        sin: [num_tokens, head_size // 2]
        is_neox_style: Whether to use the Neox-style or GPT-J-style rotary
            positional embeddings.
    """
    cos = cos.unsqueeze(-2).to(x.dtype)
    sin = sin.unsqueeze(-2).to(x.dtype)
    if is_neox_style:
        x1, x2 = torch.chunk(x, 2, dim=-1)
    else:
        x1 = x[..., ::2]
        x2 = x[..., 1::2]
    o1 = x1 * cos - x2 * sin
    o2 = x2 * cos + x1 * sin
    if is_neox_style:
        return torch.cat((o1, o2), dim=-1)
    else:
        return torch.stack((o1, o2), dim=-1).flatten(-2)
````
**EN:** This section defines `_apply_rotary_emb` and implements the core logic associated with apply rotary emb. Docstring summary: Args: x: [num_tokens, num_heads, head_size] cos: [num_tokens, head_size // 2] sin: [num_tokens, head_size // 2] is_neox_style: Whether to use the Neox-style or GPT-J-style rotary positional embeddings.
**CN:** 该部分定义 `_apply_rotary_emb`，并实现与 apply rotary emb 相关的核心逻辑。 文档字符串摘要：Args: x: [num_tokens, num_heads, head_size] cos: [num_tokens, head_size // 2] sin: [num_tokens, head_size // 2] is_neox_style: Whether to use the Neox-style or GPT-J-style rotary positional embeddings.

### Lines 61-143: `RotaryEmbedding` definition
````python
class RotaryEmbedding(torch.nn.Module):
    # Reference: https://github.com/vllm-project/vllm/blob/main/vllm/model_executor/layers/rotary_embedding.py
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
        self.cos_sin_cache: torch.Tensor
        self.register_buffer("cos_sin_cache", cache, persistent=False)

    def _compute_inv_freq(self, base: Union[int, float]) -> torch.Tensor:
        inv_freq = 1.0 / (
            base
            ** (
                torch.arange(0, self.rotary_dim, 2, dtype=torch.float) / self.rotary_dim
            )
        )
        return inv_freq

    def _compute_cos_sin_cache(self) -> torch.Tensor:
        """Compute the cos and sin cache."""
        inv_freq = self._compute_inv_freq(self.base)
        t = torch.arange(self.max_position_embeddings, dtype=torch.float)

        freqs = torch.einsum("i,j -> ij", t, inv_freq)
        cos = freqs.cos()
        sin = freqs.sin()
        cache = torch.cat((cos, sin), dim=-1)
        return cache

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
        cos_sin = self.cos_sin_cache.index_select(0, positions)

        cos, sin = cos_sin.chunk(2, dim=-1)

        query_shape = query.shape
        query = query.view(num_tokens, -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = _apply_rotary_emb(query_rot, cos, sin, self.is_neox_style)
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        key_shape = key.shape
        key = key.view(num_tokens, -1, self.head_size)
        key_rot = key[..., : self.rotary_dim]
        key_pass = key[..., self.rotary_dim :]
        key_rot = _apply_rotary_emb(key_rot, cos, sin, self.is_neox_style)
        key = torch.cat((key_rot, key_pass), dim=-1).reshape(key_shape)

        # Modification: convert to the correct dtype
        query = query.to(self.dtype)
        key = key.to(self.dtype)
        return query, key
````
**EN:** This section defines the class `RotaryEmbedding`, grouping related state and behavior around rotary Embedding. It also performs explicit assertions or shape checks before continuing.
**CN:** 该部分定义类 `RotaryEmbedding`，把与 rotary Embedding 相关的状态和行为组织在一起。 它还会在继续执行前进行显式断言或形状检查。

### Lines 146-171: `FlashInferRotaryEmbedding` definition
````python
class FlashInferRotaryEmbedding(RotaryEmbedding):
    def forward_cuda(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
        fused_set_kv_buffer_arg: Optional[FusedSetKVBufferArg] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:

        query_view = query.view(query.shape[0], -1, self.head_size)
        key_view = key.view(key.shape[0], -1, self.head_size)
        _jit_apply_rope_with_cos_sin_cache_inplace(
            q=query_view,
            k=key_view,
            cos_sin_cache=self.cos_sin_cache,
            positions=positions,
            is_neox=self.is_neox_style,
            fused_args=(
                fused_set_kv_buffer_arg.to_jit()
                if fused_set_kv_buffer_arg is not None
                else None
            ),
        )

        return query, key
````
**EN:** This section defines the class `FlashInferRotaryEmbedding`, grouping related state and behavior around flash Infer rotary Embedding.
**CN:** 该部分定义类 `FlashInferRotaryEmbedding`，把与 flash Infer rotary Embedding 相关的状态和行为组织在一起。

### Lines 174-196: `SglKernelRotaryEmbedding` definition
````python
class SglKernelRotaryEmbedding(RotaryEmbedding):
    def forward_cuda(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        offsets: Optional[torch.Tensor] = None,
        fused_set_kv_buffer_arg: Optional[FusedSetKVBufferArg] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        assert (
            fused_set_kv_buffer_arg is None
        ), "fused_set_kv_buffer_arg is not supported for sgl-kernel implementation"
        if self.cos_sin_cache.dtype != query.dtype:
            self.cos_sin_cache = self.cos_sin_cache.to(query.dtype)
        torch.ops.sgl_kernel.rotary_embedding(
            positions,
            query,
            key,
            self.head_size,
            self.cos_sin_cache,
            self.is_neox_style,
        )
        return query, key
````
**EN:** This section defines the class `SglKernelRotaryEmbedding`, grouping related state and behavior around SGL kernel rotary Embedding. It also performs explicit assertions or shape checks before continuing. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义类 `SglKernelRotaryEmbedding`，把与 SGL kernel rotary Embedding 相关的状态和行为组织在一起。 它还会在继续执行前进行显式断言或形状检查。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 199-243: `MHATokenToKVPool` definition
````python
class MHATokenToKVPool:
    KV_POOL_SIZE = 16384

    def __init__(
        self,
        head_num: int,
        head_dim: int,
    ):
        self.head_num = head_num
        self.head_dim = head_dim
        self.size = MHATokenToKVPool.KV_POOL_SIZE
        self.page_size = 1
        self.store_dtype = torch.bfloat16
        self.device = "cuda"
        self.layer_num = 1
        self.start_layer = 0
        self._create_buffers()

    def _create_buffers(self):
        self.k_buffer = [
            torch.zeros(
                (self.size + self.page_size, self.head_num, self.head_dim),
                dtype=self.store_dtype,
                device=self.device,
            )
            for _ in range(self.layer_num)
        ]
        self.v_buffer = [
            torch.zeros(
                (self.size + self.page_size, self.head_num, self.head_dim),
                dtype=self.store_dtype,
                device=self.device,
            )
            for _ in range(self.layer_num)
        ]

    def set_kv_buffer(
        self,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
        cache_v: torch.Tensor,
    ):
        layer_id = 0
        self.k_buffer[layer_id - self.start_layer][loc] = cache_k
        self.v_buffer[layer_id - self.start_layer][loc] = cache_v
````
**EN:** This section defines the class `MHATokenToKVPool`, grouping related state and behavior around MHAToken To KVPool.
**CN:** 该部分定义类 `MHATokenToKVPool`，把与 MHAToken To KVPool 相关的状态和行为组织在一起。

### Lines 246-271: `create_inputs` definition
````python
def create_inputs(
    head_size: int,
    batch_size: int,
    seq_len: int,
    device,
    dtype: torch.dtype,
    num_q_heads: int,
    num_kv_heads: int,
):
    pos_ids = torch.arange(seq_len, device=device).repeat(batch_size)
    query = torch.randn(
        batch_size * seq_len, num_q_heads * head_size, dtype=dtype, device=device
    )
    key = torch.randn(
        batch_size * seq_len, num_kv_heads * head_size, dtype=dtype, device=device
    )
    value = torch.randn(
        batch_size * seq_len, num_kv_heads * head_size, dtype=dtype, device=device
    )
    out_cache_loc = torch.randperm(
        MHATokenToKVPool.KV_POOL_SIZE, dtype=torch.int64, device=device
    )[: batch_size * seq_len].clone()

    return dict(
        pos_ids=pos_ids, query=query, key=key, value=value, out_cache_loc=out_cache_loc
    )
````
**EN:** This section defines `create_inputs` and implements the core logic associated with create inputs.
**CN:** 该部分定义 `create_inputs`，并实现与 create inputs 相关的核心逻辑。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `FusedSetKVBufferArg`, `_apply_rotary_emb`, `RotaryEmbedding`, `FlashInferRotaryEmbedding`, `SglKernelRotaryEmbedding`, `MHATokenToKVPool`, `create_inputs`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.jit_kernel.rope`
- **External / 外部**: `dataclasses`, `torch`, `typing`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
