# fused_inv_rope_fp8_quant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/ops/deepseek_v4_ops/fused_inv_rope_fp8_quant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fused inverse RoPE + block-scaled FP8 quantization kernel for DeepseekV4 attention. / 该模块位于 `attention/ops/deepseek_v4_ops` 子系统，主要围绕 `_fused_inv_rope_fp8_quant_per_head`, `fused_inv_rope_fp8_quant`, `_fused_inv_rope_fp8_quant_kernel_impl` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""
Fused inverse RoPE + block-scaled FP8 quantization kernel for DeepseekV4 attention.

Output scale format is pre-transformed (MN-major TMA-aligned; FP32 on SM90,
INT32-packed UE8M0 on SM100) so fp8_einsum skips transform_sf_into_required_layout.
"""

import torch

from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `_fused_inv_rope_fp8_quant_per_head` function / `_fused_inv_rope_fp8_quant_per_head` 函数
```python
@triton.jit(do_not_specialize=["num_tokens"])
def _fused_inv_rope_fp8_quant_per_head(
    o_ptr,
    positions_ptr,
    cos_sin_cache_ptr,
    fp8_ptr,
    scale_ptr,
    num_tokens,
    heads_per_group: tl.constexpr,
    o_stride_token,
    o_stride_head,
    cache_stride_pos,
    fp8_stride_group,
    fp8_stride_token,
    scale_stride_group,
    scale_stride_k,
    fp8_max: tl.constexpr,
    eps: tl.constexpr,
    QUANT_GROUP_SIZE: tl.constexpr,
    CHUNKS_PER_HEAD: tl.constexpr,
    ROPE_START: tl.constexpr,
    HALF_ROPE: tl.constexpr,
    TMA_ALIGNED_SCALES: tl.constexpr,
):
    # int64: stride multiply overflows int32 past num_tokens=32768 (IMA).
    pid_token = tl.program_id(0).to(tl.int64)
    pid_gh = tl.program_id(1).to(tl.int64)

    g = pid_gh // heads_per_group
    head_in_group = pid_gh % heads_per_group
    global_head = pid_gh
    qb_start = head_in_group * CHUNKS_PER_HEAD

    # Padding rows in the TMA-aligned scale buffer: fill with zero and skip quant.
    if pid_token >= num_tokens:
        if TMA_ALIGNED_SCALES:
            scale_addr = (
                scale_ptr
                + g * scale_stride_group
                + pid_token
                + head_in_group * scale_stride_k
            )
            tl.store(scale_addr, tl.zeros((), dtype=tl.int32))
        else:
            block_offsets = tl.arange(0, CHUNKS_PER_HEAD)
            qb_indices = qb_start + block_offsets
            scale_addrs = (
                scale_ptr
                + g * scale_stride_group
                + pid_token
# ... omitted for brevity ...
        fp8_ptr
        + g * fp8_stride_group
        + pid_token * fp8_stride_token
        + qb_start * QUANT_GROUP_SIZE
    )
    tl.store(fp8_base + offsets, x_quant)

    block_offsets = tl.arange(0, CHUNKS_PER_HEAD)
    qb_indices = qb_start + block_offsets
    if TMA_ALIGNED_SCALES:
        scale_bits = scales.to(tl.int32, bitcast=True)
        ue8m0_bytes = (scale_bits >> 23) & 0xFF
        packed_val = tl.sum(ue8m0_bytes << (block_offsets * 8))
        scale_addr = (
            scale_ptr
            + g * scale_stride_group
            + pid_token
            + head_in_group * scale_stride_k
        )
        tl.store(scale_addr, packed_val)
    else:
        scale_addrs = (
            scale_ptr + g * scale_stride_group + pid_token + qb_indices * scale_stride_k
        )
        tl.store(scale_addrs, scales)
```
**EN:** This function implements `_fused_inv_rope_fp8_quant_per_head` within the module. Key calls include `jit`, `to`, `arange`, `load`, `maximum`, `where`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `_fused_inv_rope_fp8_quant_per_head`，其作用域位于the module。 关键调用包括 `jit`, `to`, `arange`, `load`, `maximum`, `where`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `fused_inv_rope_fp8_quant` function / `fused_inv_rope_fp8_quant` 函数
```python
def fused_inv_rope_fp8_quant(
    o: torch.Tensor,
    positions: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    n_groups: int,
    heads_per_group: int,
    nope_dim: int = 448,
    rope_dim: int = 64,
    quant_group_size: int = 128,
    tma_aligned_scales: bool = False,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Fused inverse RoPE + block-scaled FP8 quantization.

    Args:
        o: Attention output [num_tokens, num_heads, head_dim] bf16.
        positions: Token positions [num_tokens] int64.
        cos_sin_cache: Precomputed [max_pos, rope_dim] with cos||sin.
        n_groups: Number of output groups.
        heads_per_group: Heads per group.
        nope_dim: Non-RoPE dimensions per head (default 448).
        rope_dim: RoPE dimensions per head (default 64).
        quant_group_size: FP8 quantization block size (default 128).
        tma_aligned_scales: Output INT32 packed UE8M0 for SM100 (True)
                            or FP32 for SM90 (False).

    Returns:
        o_fp8: [T, G, D] float8_e4m3fn, strides (D, T*D, 1).
        o_scale: Pre-transformed scale tensor for fp8_einsum.
    """
    from vllm.utils.deep_gemm import get_tma_aligned_size

    num_tokens, num_heads, head_dim = o.shape
    assert num_heads == n_groups * heads_per_group
    assert head_dim == nope_dim + rope_dim
    assert head_dim % quant_group_size == 0
    assert nope_dim % quant_group_size == (quant_group_size - rope_dim)
    assert rope_dim % 2 == 0
    assert cos_sin_cache.shape[-1] == rope_dim
    assert cos_sin_cache.dtype == torch.float32

    d = heads_per_group * head_dim
    num_scale_blocks = d // quant_group_size
    chunks_per_head = head_dim // quant_group_size

    fp8_dtype = torch.float8_e4m3fn
    fp8_max = torch.finfo(fp8_dtype).max

    tma_aligned_T = get_tma_aligned_size(num_tokens, 4)
    if tma_aligned_scales:
        packed_sf_k = (num_scale_blocks + 3) // 4
        scale_inner = packed_sf_k
    else:
        scale_inner = num_scale_blocks

    # Run kernel through a custom op so inductor sees an opaque boundary.
    # It's a pytorch bug, see https://github.com/vllm-project/vllm/issues/41106
    fp8_buf, scale_buf = torch.ops.vllm.fused_inv_rope_fp8_quant_kernel(
        o,
        positions,
        cos_sin_cache,
        heads_per_group,
        quant_group_size,
        chunks_per_head,
        nope_dim % quant_group_size,
        rope_dim // 2,
        tma_aligned_scales,
        fp8_max,
        tma_aligned_T,
        num_tokens,
        n_groups,
        d,
        scale_inner,
    )
    return fp8_buf.transpose(0, 1), scale_buf.transpose(0, 1)
```
**EN:** This function implements `fused_inv_rope_fp8_quant` within the module. The docstring frames it as: Fused inverse RoPE + block-scaled FP8 quantization. Key calls include `get_tma_aligned_size`, `fused_inv_rope_fp8_quant_kernel`, `finfo`, `transpose`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `fused_inv_rope_fp8_quant`，其作用域位于the module。 关键调用包括 `get_tma_aligned_size`, `fused_inv_rope_fp8_quant_kernel`, `finfo`, `transpose`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_fused_inv_rope_fp8_quant_kernel_impl` function / `_fused_inv_rope_fp8_quant_kernel_impl` 函数
```python
def _fused_inv_rope_fp8_quant_kernel_impl(
    o: torch.Tensor,
    positions: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    heads_per_group: int,
    quant_group_size: int,
    chunks_per_head: int,
    rope_start: int,
    half_rope: int,
    tma_aligned_scales: bool,
    fp8_max: float,
    tma_aligned_T: int,
    num_tokens: int,
    n_groups: int,
    d: int,
    scale_inner: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    fp8_buf = torch.empty(
        (n_groups, num_tokens, d),
        dtype=torch.float8_e4m3fn,
        device=o.device,
    )
    scale_dtype = torch.int32 if tma_aligned_scales else torch.float32
    scale_buf = torch.empty(
        n_groups * scale_inner * tma_aligned_T,
        dtype=scale_dtype,
        device=o.device,
    ).as_strided(
        (n_groups, num_tokens, scale_inner),
        (scale_inner * tma_aligned_T, 1, tma_aligned_T),
    )
    grid = (tma_aligned_T, n_groups * heads_per_group)
    pdl_kwargs = {} if current_platform.is_rocm() else {"launch_pdl": False}
    _fused_inv_rope_fp8_quant_per_head[grid](
        o,
        positions,
        cos_sin_cache,
        fp8_buf,
        scale_buf,
        num_tokens,
        heads_per_group=heads_per_group,
        o_stride_token=o.stride(0),
        o_stride_head=o.stride(1),
        cache_stride_pos=cos_sin_cache.stride(0),
        fp8_stride_group=fp8_buf.stride(0),
        fp8_stride_token=fp8_buf.stride(1),
        scale_stride_group=scale_buf.stride(0),
        scale_stride_k=scale_buf.stride(2),
        fp8_max=fp8_max,
        eps=1e-10,
        QUANT_GROUP_SIZE=quant_group_size,
        CHUNKS_PER_HEAD=chunks_per_head,
        ROPE_START=rope_start,
        HALF_ROPE=half_rope,
        TMA_ALIGNED_SCALES=tma_aligned_scales,
        num_stages=1,
        **pdl_kwargs,
        num_warps=1,
    )
    return fp8_buf, scale_buf
```
**EN:** This function implements `_fused_inv_rope_fp8_quant_kernel_impl` within the module. Key calls include `empty`, `as_strided`, `is_rocm`, `stride`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_fused_inv_rope_fp8_quant_kernel_impl`，其作用域位于the module。 关键调用包括 `empty`, `as_strided`, `is_rocm`, `stride`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_fused_inv_rope_fp8_quant_kernel_fake` function / `_fused_inv_rope_fp8_quant_kernel_fake` 函数
```python
def _fused_inv_rope_fp8_quant_kernel_fake(
    o: torch.Tensor,
    positions: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    heads_per_group: int,
    quant_group_size: int,
    chunks_per_head: int,
    rope_start: int,
    half_rope: int,
    tma_aligned_scales: bool,
    fp8_max: float,
    tma_aligned_T: int,
    num_tokens: int,
    n_groups: int,
    d: int,
    scale_inner: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    fp8_buf = torch.empty(
        (n_groups, num_tokens, d),
        dtype=torch.float8_e4m3fn,
        device=o.device,
    )
    scale_dtype = torch.int32 if tma_aligned_scales else torch.float32
    scale_buf = torch.empty(
        n_groups * scale_inner * tma_aligned_T,
        dtype=scale_dtype,
        device=o.device,
    ).as_strided(
        (n_groups, num_tokens, scale_inner),
        (scale_inner * tma_aligned_T, 1, tma_aligned_T),
    )
    return fp8_buf, scale_buf
```
**EN:** This function implements `_fused_inv_rope_fp8_quant_kernel_fake` within the module. Key calls include `empty`, `as_strided`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_fused_inv_rope_fp8_quant_kernel_fake`，其作用域位于the module。 关键调用包括 `empty`, `as_strided`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### Module setup / 模块初始化
```python
direct_register_custom_op(
    op_name="fused_inv_rope_fp8_quant_kernel",
    op_func=_fused_inv_rope_fp8_quant_kernel_impl,
    fake_impl=_fused_inv_rope_fp8_quant_kernel_fake,
)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

## Key Concepts / 关键概念
- `_fused_inv_rope_fp8_quant_per_head`: top-level helper or orchestration entry point. / `_fused_inv_rope_fp8_quant_per_head`：顶层辅助函数或编排入口。
- `fused_inv_rope_fp8_quant`: top-level helper or orchestration entry point. / `fused_inv_rope_fp8_quant`：顶层辅助函数或编排入口。
- `_fused_inv_rope_fp8_quant_kernel_impl`: top-level helper or orchestration entry point. / `_fused_inv_rope_fp8_quant_kernel_impl`：顶层辅助函数或编排入口。
- `_fused_inv_rope_fp8_quant_kernel_fake`: top-level helper or orchestration entry point. / `_fused_inv_rope_fp8_quant_kernel_fake`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.platforms`, `vllm.triton_utils`, `vllm.utils.torch_utils`, `vllm.utils.deep_gemm`
