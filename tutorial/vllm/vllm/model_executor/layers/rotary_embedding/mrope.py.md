# mrope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/rotary_embedding/mrope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `MRotaryEmbedding` and related helpers for rotary positional embedding variants. / 实现 `MRotaryEmbedding` 及其相关辅助逻辑，用于旋转位置编码变体。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 5-11)
```python
import numpy as np
import torch

from vllm.triton_utils import tl, triton

from .base import RotaryEmbeddingBase
from .yarn_scaling_rope import YaRNScalingRotaryEmbedding, yarn_get_mscale
```
**EN:** This opening block pulls in external dependencies such as `numpy`, `torch` and internal modules such as `vllm.triton_utils`, `.base`, `.yarn_scaling_rope`. That import mix shows the file is part of the rotary positional embedding variants stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `numpy`, `torch`）以及内部模块（如 `vllm.triton_utils`, `.base`, `.yarn_scaling_rope`）。这些导入关系表明该文件属于旋转位置编码变体栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `_triton_mrope_forward` (lines 15-130)
```python
def _triton_mrope_forward(
    q_ptr,
    k_ptr,
    cos,
    sin,
    num_tokens,
    n_qh: tl.constexpr,
    n_kh: tl.constexpr,
    hd: tl.constexpr,
    rd: tl.constexpr,
    pad_n_qh: tl.constexpr,
    pad_n_kh: tl.constexpr,
    pad_hd: tl.constexpr,
    mrope_section_t: tl.constexpr,
    mrope_section_h: tl.constexpr,
    mrope_section_w: tl.constexpr,
    is_interleaved: tl.constexpr,
):
    # Adapted from
    # https://github.com/linkedin/Liger-Kernel/blob/main/src/liger_kernel/ops/qwen2vl_mrope.py
    # This version supports flatten input tensors from vllm
    # and supports cos and sin cache with shape (3, num_tokens, head_dim // 2)
    # instead of (3, bsz, seq_len, head_dim), also supports interleaved rotary
    pid = tl.program_id(0)
    # locate start address
    q_ptr = q_ptr + pid * (n_qh * hd)
    k_ptr = k_ptr + pid * (n_kh * hd)

    # ####################################################################
    # get the cos(mθ_{i...d/2}) and sin(mθ_{i...d/2}) for token position
    # m of this program instance
    # ####################################################################
    # Note: cos and sin now have shape (3, num_tokens, head_dim // 2)

    # Updated stride calculation for half head_dim
    half_rd = rd // 2
    t_cos = cos + pid * half_rd
    h_cos = t_cos + num_tokens * half_rd
# ... truncated for analysis ...
    )

    # y = [x1, x2] * [cos, cos] + [-x2, x1] * [sin, sin]
    # Since cos and sin are now half-size,
    # we use the same cos_row and sin_row for both halves
    new_q_tile_1 = q_tile_1 * cos_row - q_tile_2 * sin_row
    tl.store(q_ptr + first_half_q_offsets, new_q_tile_1, mask=first_q_mask)
    new_q_tile_2 = q_tile_2 * cos_row + q_tile_1 * sin_row
    tl.store(q_ptr + second_half_q_offsets, new_q_tile_2, mask=second_q_mask)

    new_k_tile_1 = k_tile_1 * cos_row - k_tile_2 * sin_row
    tl.store(k_ptr + first_half_k_offsets, new_k_tile_1, mask=first_k_mask)
    new_k_tile_2 = k_tile_2 * cos_row + k_tile_1 * sin_row
    tl.store(k_ptr + second_half_k_offsets, new_k_tile_2, mask=second_k_mask)
```
**EN:** Defines function `_triton_mrope_forward` with signature `_triton_mrope_forward(q_ptr, k_ptr, cos, sin, num_tokens, n_qh: tl.constexpr, n_kh: tl.constexpr, hd: tl.constexpr, rd: tl.constexpr, pad_n_qh: tl.constexpr, pad_n_kh: tl.constexpr, pad_hd: tl.constexpr, mrope_section_t: tl.constexpr, mrope_section_h: tl.constexpr, mrope_section_w: tl.constexpr, is_interleaved: tl.constexpr)`. It mainly works with `q_ptr`, `k_ptr`, `cos`, `sin`, `num_tokens`, `n_qh`, `n_kh`, `hd`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, tensor/kernel operations. Key calls include `tl.program_id`, `tl.arange`, `tl.load`, `tl.load.to`, `tl.store`.
**CN:** 定义函数 `_triton_mrope_forward`，其签名为 `_triton_mrope_forward(q_ptr, k_ptr, cos, sin, num_tokens, n_qh: tl.constexpr, n_kh: tl.constexpr, hd: tl.constexpr, rd: tl.constexpr, pad_n_qh: tl.constexpr, pad_n_kh: tl.constexpr, pad_hd: tl.constexpr, mrope_section_t: tl.constexpr, mrope_section_h: tl.constexpr, mrope_section_w: tl.constexpr, is_interleaved: tl.constexpr)`。它主要围绕 `q_ptr`, `k_ptr`, `cos`, `sin`, `num_tokens`, `n_qh`, `n_kh`, `hd` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、张量或内核操作。关键调用包括 `tl.program_id`, `tl.arange`, `tl.load`, `tl.load.to`, `tl.store`。

### Function `triton_mrope` (lines 133-187)
```python
def triton_mrope(
    q: torch.Tensor,
    k: torch.Tensor,
    cos: torch.Tensor,
    sin: torch.Tensor,
    mrope_section: list[int],
    head_size: int,
    rotary_dim: int,
    mrope_interleaved: bool,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Qwen2VL mrope kernel.

    Args:
        q: [num_tokens, num_heads * head_size]
        k: [num_tokens, num_kv_heads * head_size]
        cos: [3, num_tokens, head_size //2 ]
            (T/H/W positions with multimodal inputs)
        sin: [3, num_tokens, head_size //2 ]
            (T/H/W positions with multimodal inputs)
        mrope_section: [t, h, w]
        head_size: int
    """
    n_row, n_q_head_head_dim = q.shape
    n_q_head = n_q_head_head_dim // head_size
    n_kv_head = k.shape[1] // head_size
    pad_hd = triton.next_power_of_2(head_size)
    pad_n_q_head = triton.next_power_of_2(n_q_head)
    pad_n_kv_head = triton.next_power_of_2(n_kv_head)

    # ensure tensors passed into the kernel are contiguous.
    # It will be no-op if they are already contiguous
    q = q.contiguous()
    k = k.contiguous()
    cos = cos.contiguous()
    sin = sin.contiguous()

    _triton_mrope_forward[(n_row,)](
        q,
        k,
        cos,
        sin,
        n_row,
        n_q_head,
        n_kv_head,
        head_size,
        rotary_dim,
        pad_n_q_head,
        pad_n_kv_head,
        pad_hd,
        mrope_section[0],
        mrope_section[1],
        mrope_section[2],
        mrope_interleaved,
    )
    return q, k
```
**EN:** Defines function `triton_mrope` with signature `triton_mrope(q: torch.Tensor, k: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor, mrope_section: list[int], head_size: int, rotary_dim: int, mrope_interleaved: bool) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `q`, `k`, `cos`, `sin`, `mrope_section`, `head_size`, `rotary_dim`, `mrope_interleaved`; adjusts rotary-position-encoding parameters or application logic. The body uses tensor/kernel operations. Key calls include `triton.next_power_of_2`, `q.contiguous`, `k.contiguous`, `cos.contiguous`, `sin.contiguous`, `_triton_mrope_forward`.
**CN:** 定义函数 `triton_mrope`，其签名为 `triton_mrope(q: torch.Tensor, k: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor, mrope_section: list[int], head_size: int, rotary_dim: int, mrope_interleaved: bool) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `q`, `k`, `cos`, `sin`, `mrope_section`, `head_size`, `rotary_dim`, `mrope_interleaved` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含张量或内核操作。关键调用包括 `triton.next_power_of_2`, `q.contiguous`, `k.contiguous`, `cos.contiguous`, `sin.contiguous`, `_triton_mrope_forward`。

### Function `apply_interleaved_rope` (lines 190-198)
```python
def apply_interleaved_rope(x: torch.Tensor, mrope_section: list[int]) -> torch.Tensor:
    """Apply interleaved MRoPE to 3D rotary embeddings.
    Reorganizes frequency layout from chunked [TTT...HHH...WWW] to
    interleaved [THTHWHTHW...TT], preserving frequency continuity.
    """
    x_t = x[0].clone()
    x_t[..., 1 : mrope_section[1] * 3 : 3] = x[1, ..., 1 : mrope_section[1] * 3 : 3]
    x_t[..., 2 : mrope_section[2] * 3 : 3] = x[2, ..., 2 : mrope_section[2] * 3 : 3]
    return x_t
```
**EN:** Defines function `apply_interleaved_rope` with signature `apply_interleaved_rope(x: torch.Tensor, mrope_section: list[int]) -> torch.Tensor`. It mainly works with `x`, `mrope_section`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include `x.clone`.
**CN:** 定义函数 `apply_interleaved_rope`，其签名为 `apply_interleaved_rope(x: torch.Tensor, mrope_section: list[int]) -> torch.Tensor`。它主要围绕 `x`, `mrope_section` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `x.clone`。

### Class `MRotaryEmbedding` overview (lines 201-414)
```python
class MRotaryEmbedding(RotaryEmbeddingBase):
    """Rotary Embedding with Multimodal Sections."""

    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        mrope_section: list[int] | None = None,
        mrope_interleaved: bool = False,
        # YaRN parameters.
        *,
        scaling_factor: float | None = None,
        extrapolation_factor: float = 1,
        attn_factor: float = 1,
        beta_fast: int = 32,
        beta_slow: int = 1,
        truncate: bool = True,
    ) -> None:
        self.scaling_factor = scaling_factor
        self.extrapolation_factor = extrapolation_factor
        self.attn_factor = attn_factor
```
**EN:** Defines class `MRotaryEmbedding` with base classes `RotaryEmbeddingBase` and decorators none. It acts as an embedding layer with model-parallel awareness and exposes 8 direct methods, with notable entries `__init__`, `_compute_inv_freq`, `_compute_cos_sin_cache`, `forward_native`, `forward_cuda`, `forward_cpu`. Its docstring says: Rotary Embedding with Multimodal Sections.
**CN:** 定义类 `MRotaryEmbedding`，其基类为 `RotaryEmbeddingBase`，装饰器为 无。它在整体实现中充当具备模型并行意识的嵌入层，并直接暴露 8 个方法，较重要的包括 `__init__`, `_compute_inv_freq`, `_compute_cos_sin_cache`, `forward_native`, `forward_cuda`, `forward_cpu`。 文档字符串进一步说明了该类的定位。

### Method `MRotaryEmbedding.__init__` (lines 204-251)
```python
    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: float,
        is_neox_style: bool,
        dtype: torch.dtype,
        mrope_section: list[int] | None = None,
        mrope_interleaved: bool = False,
        # YaRN parameters.
        *,
        scaling_factor: float | None = None,
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
        if self.scaling_factor is not None:
            # Get n-d magnitude scaling corrected for interpolation
            self.mscale = float(yarn_get_mscale(self.scaling_factor) * attn_factor)
        else:
            self.mscale = 1.0

        # In Qwen2.5-VL, the maximum index value is related to the duration of
        # the input video. We enlarge max_position_embeddings to 4 times to get
        # a larger the cos and sin cache.
        self.cache_max_position_num = max_position_embeddings * 4
        super().__init__(
            head_size,
            rotary_dim,
            self.cache_max_position_num,
            base,
            is_neox_style,
            dtype,
        )

        self.mrope_section = mrope_section
        self.mrope_interleaved = mrope_interleaved
        if self.mrope_section:
            assert sum(self.mrope_section) == rotary_dim // 2
```
**EN:** Defines function `MRotaryEmbedding.__init__` with signature `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, mrope_section: list[int] | None=None, mrope_interleaved: bool=False, *, scaling_factor: float | None=None, extrapolation_factor: float=1, attn_factor: float=1, beta_fast: int=32, beta_slow: int=1, truncate: bool=True) -> None`. It mainly works with `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `mrope_section`, `mrope_interleaved`; initializes the object state and cached resources. The body uses branching, validation/error handling. Key calls include `super.__init__`, `float`, `super`, `sum`, `yarn_get_mscale`.
**CN:** 定义函数 `MRotaryEmbedding.__init__`，其签名为 `__init__(self, head_size: int, rotary_dim: int, max_position_embeddings: int, base: float, is_neox_style: bool, dtype: torch.dtype, mrope_section: list[int] | None=None, mrope_interleaved: bool=False, *, scaling_factor: float | None=None, extrapolation_factor: float=1, attn_factor: float=1, beta_fast: int=32, beta_slow: int=1, truncate: bool=True) -> None`。它主要围绕 `head_size`, `rotary_dim`, `max_position_embeddings`, `base`, `is_neox_style`, `dtype`, `mrope_section`, `mrope_interleaved` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑。关键调用包括 `super.__init__`, `float`, `super`, `sum`, `yarn_get_mscale`。

### Method `MRotaryEmbedding.forward_native` (lines 263-322)
```python
    def forward_native(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """PyTorch-native implementation equivalent to forward().

        Args:
            positions:
                [num_tokens,] (text only) or
                [3, num_tokens] (T/H/W positions with multimodal inputs)
            query: [num_tokens, num_heads * head_size]
            key: [num_tokens, num_kv_heads * head_size]
        """
        assert positions.ndim == 1 or positions.ndim == 2
        assert key is not None

        cos_sin_cache = self._match_cos_sin_cache_dtype(query)
        num_tokens = positions.shape[-1]
        cos_sin = cos_sin_cache[positions]
        cos, sin = cos_sin.chunk(2, dim=-1)
        if positions.ndim == 2:
            assert self.mrope_section
            if self.mrope_interleaved:
                cos = apply_interleaved_rope(cos, self.mrope_section)
                sin = apply_interleaved_rope(sin, self.mrope_section)
            else:
                cos = torch.cat(
                    [m[i] for i, m in enumerate(cos.split(self.mrope_section, dim=-1))],
                    dim=-1,
                )
                sin = torch.cat(
                    [m[i] for i, m in enumerate(sin.split(self.mrope_section, dim=-1))],
                    dim=-1,
                )

        query_shape = query.shape
        query = query.view(num_tokens, -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = self.apply_rotary_emb.forward_native(
            query_rot,
            cos,
            sin,
        )
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        key_shape = key.shape
        key = key.view(num_tokens, -1, self.head_size)
        key_rot = key[..., : self.rotary_dim]
        key_pass = key[..., self.rotary_dim :]
        key_rot = self.apply_rotary_emb.forward_native(
            key_rot,
            cos,
            sin,
        )
        key = torch.cat((key_rot, key_pass), dim=-1).reshape(key_shape)
        return query, key
```
**EN:** Defines function `MRotaryEmbedding.forward_native` with signature `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, comprehensions, validation/error handling, tensor/kernel operations. Key calls include `self._match_cos_sin_cache_dtype`, `cos_sin.chunk`, `query.view`, `self.apply_rotary_emb.forward_native`, `torch.cat.reshape`, `key.view`.
**CN:** 定义函数 `MRotaryEmbedding.forward_native`，其签名为 `forward_native(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、推导式、校验或报错逻辑、张量或内核操作。关键调用包括 `self._match_cos_sin_cache_dtype`, `cos_sin.chunk`, `query.view`, `self.apply_rotary_emb.forward_native`, `torch.cat.reshape`, `key.view`。

### Method `MRotaryEmbedding.forward_cuda` (lines 324-375)
```python
    def forward_cuda(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        assert positions.ndim == 1 or positions.ndim == 2
        assert key is not None

        cos_sin_cache = self._match_cos_sin_cache_dtype(query)
        num_tokens = positions.shape[-1]
        cos_sin = cos_sin_cache[positions]
        cos, sin = cos_sin.chunk(2, dim=-1)
        query_shape = query.shape
        key_shape = key.shape
        if positions.ndim == 2:
            assert self.mrope_section

            q, k = triton_mrope(
                query,
                key,
                cos,
                sin,
                self.mrope_section,
                self.head_size,
                self.rotary_dim,
                self.mrope_interleaved,
            )

            return q.reshape(query_shape), k.reshape(key_shape)

        query = query.view(num_tokens, -1, self.head_size)
        query_rot = query[..., : self.rotary_dim]
        query_pass = query[..., self.rotary_dim :]
        query_rot = self.apply_rotary_emb(
            query_rot,
            cos,
            sin,
        )
        query = torch.cat((query_rot, query_pass), dim=-1).reshape(query_shape)

        key = key.view(num_tokens, -1, self.head_size)
        key_rot = key[..., : self.rotary_dim]
        key_pass = key[..., self.rotary_dim :]
        key_rot = self.apply_rotary_emb(
            key_rot,
            cos,
            sin,
        )
        key = torch.cat((key_rot, key_pass), dim=-1).reshape(key_shape)
        return query, key
```
**EN:** Defines function `MRotaryEmbedding.forward_cuda` with signature `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `self._match_cos_sin_cache_dtype`, `cos_sin.chunk`, `query.view`, `self.apply_rotary_emb`, `torch.cat.reshape`, `key.view`.
**CN:** 定义函数 `MRotaryEmbedding.forward_cuda`，其签名为 `forward_cuda(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `self._match_cos_sin_cache_dtype`, `cos_sin.chunk`, `query.view`, `self.apply_rotary_emb`, `torch.cat.reshape`, `key.view`。

### Method `MRotaryEmbedding.forward_cpu` (lines 377-384)
```python
    def forward_cpu(
        self,
        positions: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor | None = None,
        offsets: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        return self.forward_native(positions, query, key, offsets)
```
**EN:** Defines function `MRotaryEmbedding.forward_cpu` with signature `forward_cpu(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`. It mainly works with `positions`, `query`, `key`, `offsets`; adjusts rotary-position-encoding parameters or application logic. The body uses mostly straightforward data movement and object wiring. Key calls include `self.forward_native`.
**CN:** 定义函数 `MRotaryEmbedding.forward_cpu`，其签名为 `forward_cpu(self, positions: torch.Tensor, query: torch.Tensor, key: torch.Tensor | None=None, offsets: torch.Tensor | None=None) -> tuple[torch.Tensor, torch.Tensor | None]`。它主要围绕 `positions`, `query`, `key`, `offsets` 展开；调整旋转位置编码参数或其应用逻辑。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `self.forward_native`。

### Method `MRotaryEmbedding.get_next_input_positions` (lines 387-399)
```python
    def get_next_input_positions(
        mrope_position_delta: int,
        context_len: int,
        seq_len: int,
    ) -> list[list[int]]:
        return [
            list(
                range(
                    context_len + mrope_position_delta, seq_len + mrope_position_delta
                )
            )
            for _ in range(3)
        ]
```
**EN:** Defines function `MRotaryEmbedding.get_next_input_positions` with signature `get_next_input_positions(mrope_position_delta: int, context_len: int, seq_len: int) -> list[list[int]]`. It mainly works with `mrope_position_delta`, `context_len`, `seq_len`; returns a derived property or capability check. The body uses comprehensions. Key calls include `list`, `range`.
**CN:** 定义函数 `MRotaryEmbedding.get_next_input_positions`，其签名为 `get_next_input_positions(mrope_position_delta: int, context_len: int, seq_len: int) -> list[list[int]]`。它主要围绕 `mrope_position_delta`, `context_len`, `seq_len` 展开；返回派生属性或能力判断结果。函数体包含推导式。关键调用包括 `list`, `range`。

### Method `MRotaryEmbedding.get_next_input_positions_tensor` (lines 402-414)
```python
    def get_next_input_positions_tensor(
        out: np.ndarray,
        out_offset: int,
        mrope_position_delta: int,
        context_len: int,
        num_new_tokens: int,
    ):
        values = np.arange(
            mrope_position_delta + context_len,
            mrope_position_delta + context_len + num_new_tokens,
            dtype=out.dtype,
        )
        out[:, out_offset : out_offset + num_new_tokens] = values
```
**EN:** Defines function `MRotaryEmbedding.get_next_input_positions_tensor` with signature `get_next_input_positions_tensor(out: np.ndarray, out_offset: int, mrope_position_delta: int, context_len: int, num_new_tokens: int)`. It mainly works with `out`, `out_offset`, `mrope_position_delta`, `context_len`, `num_new_tokens`; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `np.arange`.
**CN:** 定义函数 `MRotaryEmbedding.get_next_input_positions_tensor`，其签名为 `get_next_input_positions_tensor(out: np.ndarray, out_offset: int, mrope_position_delta: int, context_len: int, num_new_tokens: int)`。它主要围绕 `out`, `out_offset`, `mrope_position_delta`, `context_len`, `num_new_tokens` 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `np.arange`。

## Key Concepts / 关键概念
- **EN:** The file implements a RoPE-related variant, meaning frequency scaling, cache construction, and position application are key ideas.
  **CN:** 该文件实现了与 RoPE 相关的变体，因此频率缩放、缓存构建和位置应用是关键概念。
- **EN:** Top-level classes include `MRotaryEmbedding`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `MRotaryEmbedding`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `_triton_mrope_forward`, `triton_mrope`, `apply_interleaved_rope` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `_triton_mrope_forward`, `triton_mrope`, `apply_interleaved_rope` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `numpy`, `torch`
- **Internal / 内部**: `vllm.triton_utils`, `.base`, `.yarn_scaling_rope`
