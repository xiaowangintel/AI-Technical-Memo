# utils.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides utils utilities shared by multiple attention implementations in SGLang. / 该模块提供 SGLang 多种注意力实现共享的 utils 工具函数。
## Line-by-Line Analysis / 逐行分析
### Lines 1-5: imports
```python
import torch
import triton
import triton.language as tl

from sglang.srt.utils import is_cuda
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 7-10: module constants
```python
_FLASHMLA_CREATE_KV_BLOCK_SIZE = 4096
FLASHMLA_CREATE_KV_BLOCK_SIZE_TRITON = tl.constexpr(_FLASHMLA_CREATE_KV_BLOCK_SIZE)

_is_cuda = is_cuda()
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 12-13: conditional branch
```python
if _is_cuda:
    from sgl_kernel import concat_mla_absorb_q
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 15-15: imports
```python
from sglang.jit_kernel.utils import is_arch_support_pdl
```
**EN:** Imports neighboring SGLang modules so this file can reuse shared attention abstractions and utilities.
**CN:** 导入相邻的 SGLang 模块，以复用共享的注意力抽象和工具函数。

### Lines 18-54: function create_flashinfer_kv_indices_triton
```python
@triton.jit
def create_flashinfer_kv_indices_triton(
    req_to_token_ptr,  # [max_batch, max_context_len]
    req_pool_indices_ptr,
    page_kernel_lens_ptr,
    kv_indptr,
    kv_start_idx,
    kv_indices_ptr,
    req_to_token_ptr_stride: tl.constexpr,
):
    BLOCK_SIZE: tl.constexpr = 512
    pid = tl.program_id(axis=0)

    # find the req pool idx, this is for batch to token
    req_pool_index = tl.load(req_pool_indices_ptr + pid)
    kv_indices_offset = tl.load(kv_indptr + pid)

    kv_start = 0
    kv_end = 0
    if kv_start_idx:
        kv_start = tl.load(kv_start_idx + pid).to(tl.int32)
        kv_end = kv_start
    kv_end += tl.load(page_kernel_lens_ptr + pid).to(tl.int32)

    num_loop = tl.cdiv(kv_end - kv_start, BLOCK_SIZE)
    for i in range(num_loop):
        # index into req_to_token_ptr needs to be int64
        offset = tl.arange(0, BLOCK_SIZE).to(tl.int64) + i * BLOCK_SIZE
        mask = offset < kv_end - kv_start
        data = tl.load(
            req_to_token_ptr
            + req_pool_index * req_to_token_ptr_stride
            + kv_start
            + offset,
            mask=mask,
        )
        tl.store(kv_indices_ptr + kv_indices_offset + offset, data, mask=mask)
```
**EN:** Factory helper that constructs create flashinfer kv indices triton and validates the prerequisites needed for this execution path.
**CN:** 该工厂辅助函数构建 create flashinfer kv indices triton，并校验此执行路径所需的前置条件。

### Lines 57-59: function get_num_page_per_block_flashmla
```python
def get_num_page_per_block_flashmla(page_size: int = 64) -> int:
    num_page_per_block = _FLASHMLA_CREATE_KV_BLOCK_SIZE // page_size
    return num_page_per_block
```
**EN:** Computes and returns get num page per block flashmla from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get num page per block flashmla。

### Lines 62-113: function create_flashmla_kv_indices_triton
```python
@triton.jit
def create_flashmla_kv_indices_triton(
    req_to_token_ptr,  # [max_batch, max_context_len]
    req_pool_indices_ptr,
    page_kernel_lens_ptr,
    kv_start_idx,
    kv_indices_ptr,
    req_to_token_ptr_stride: tl.constexpr,
    kv_indices_ptr_stride: tl.constexpr,
    PAGED_SIZE: tl.constexpr = 64,
):
    NUM_PAGE_PER_BLOCK: tl.constexpr = (
        FLASHMLA_CREATE_KV_BLOCK_SIZE_TRITON // PAGED_SIZE
    )
    pid = tl.program_id(axis=0)

    # find the req pool idx, this is for batch to token
    req_pool_index = tl.load(req_pool_indices_ptr + pid)

    kv_start = 0
    kv_end = 0
    if kv_start_idx:
        kv_start = tl.load(kv_start_idx + pid).to(tl.int32)
        kv_end = kv_start

    kv_end += tl.load(page_kernel_lens_ptr + pid).to(tl.int32)

    num_paged = tl.cdiv(kv_end - kv_start, PAGED_SIZE)
    num_pages_loop = tl.cdiv(kv_end - kv_start, FLASHMLA_CREATE_KV_BLOCK_SIZE_TRITON)

    for i in range(num_pages_loop):
        # index into req_to_token_ptr needs to be int64
# ... omitted 8 lines ...
        data = tl.load(
            req_to_token_ptr
            + req_pool_index * req_to_token_ptr_stride
            + kv_start
            + paged_offset,
            mask=mask,
        )
        tl.store(
            kv_indices_ptr + pid * kv_indices_ptr_stride + paged_offset_out,
            data // PAGED_SIZE,
            mask=mask_out,
        )
```
**EN:** Factory helper that constructs create flashmla kv indices triton and validates the prerequisites needed for this execution path.
**CN:** 该工厂辅助函数构建 create flashmla kv indices triton，并校验此执行路径所需的前置条件。

### Lines 116-152: function concat_and_cast_mha_k_kernel
```python
@triton.jit
def concat_and_cast_mha_k_kernel(
    k_ptr,
    k_nope_ptr,
    k_rope_ptr,
    head_cnt: tl.constexpr,
    k_stride0: tl.constexpr,
    k_stride1: tl.constexpr,
    nope_stride0: tl.constexpr,
    nope_stride1: tl.constexpr,
    rope_stride0: tl.constexpr,
    nope_dim: tl.constexpr,
    rope_dim: tl.constexpr,
):
    pid_loc = tl.program_id(0)
    head_range = tl.arange(0, head_cnt)

    k_head_ptr = k_ptr + pid_loc * k_stride0 + head_range[:, None] * k_stride1

    nope_offs = tl.arange(0, nope_dim)

    src_nope_ptr = (
        k_nope_ptr
        + pid_loc * nope_stride0
        + head_range[:, None] * nope_stride1
        + nope_offs[None, :]
    )
    dst_nope_ptr = k_head_ptr + nope_offs[None, :]

    src_nope = tl.load(src_nope_ptr)
    tl.store(dst_nope_ptr, src_nope)

    rope_offs = tl.arange(0, rope_dim)
    src_rope_ptr = k_rope_ptr + pid_loc * rope_stride0 + rope_offs[None, :]
    dst_rope_ptr = k_head_ptr + nope_dim + rope_offs[None, :]
    src_rope = tl.load(src_rope_ptr)
    tl.store(dst_rope_ptr, src_rope)
```
**EN:** Implements the concat and cast mha k kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 concat and cast mha k kernel 例程。

### Lines 155-190: function concat_and_cast_mha_k_triton
```python
def concat_and_cast_mha_k_triton(
    k: torch.Tensor,
    k_nope: torch.Tensor,
    k_rope: torch.Tensor,
):
    # The source data type will be implicitly converted to the target data type.
    assert (
        len(k.shape) == 3 and len(k_nope.shape) == 3 and len(k_rope.shape) == 3
    ), f"shape should be 3d, but got {k.shape=}, {k_nope.shape=}, {k_rope.shape=}"
    assert (
        k.shape[0] == k_nope.shape[0] and k.shape[0] == k_rope.shape[0]
    ), f"invalid shape, got {k.shape=}, {k_nope.shape=}, {k_rope.shape=}"
    assert (
        k.shape[1] == k_nope.shape[1] and 1 == k_rope.shape[1]
    ), f"invalid shape, got {k.shape=}, {k_nope.shape=}, {k_rope.shape=}"
    assert (
        k.shape[-1] == k_nope.shape[-1] + k_rope.shape[-1]
    ), f"invalid shape, got {k.shape=}, {k_nope.shape=}, {k_rope.shape=}"

    nope_dim = k_nope.shape[-1]
    rope_dim = k_rope.shape[-1]
    grid = (k.shape[0],)

    concat_and_cast_mha_k_kernel[grid](
        k,
        k_nope,
        k_rope,
        k.shape[1],
        k.stride(0),
        k.stride(1),
        k_nope.stride(0),
        k_nope.stride(1),
        k_rope.stride(0),
        nope_dim,
        rope_dim,
    )
```
**EN:** Implements the concat and cast mha k triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 concat and cast mha k triton 例程。

### Lines 193-244: function pad_sequence_with_mask_kernel
```python
@triton.jit
def pad_sequence_with_mask_kernel(
    input_ptr,  # (total_tokens, hidden)
    offsets_ptr,  # (B,)
    lengths_ptr,  # (B,)
    output_ptr,  # (B, max_len, hidden)
    mask_ptr,  # (B, max_len)
    max_len,
    hidden_dim,
    BLOCK_M: tl.constexpr,  # seq block
    BLOCK_D: tl.constexpr,  # hidden block
):
    b = tl.program_id(0)  # batch index
    m = tl.program_id(1)  # seq block index

    offset = tl.load(offsets_ptr + b)
    length = tl.load(lengths_ptr + b)

    seq_ids = m * BLOCK_M + tl.arange(0, BLOCK_M)
    hid_ids = tl.arange(0, BLOCK_D)

    seq_mask = seq_ids < max_len
    valid_token = seq_ids < length

    # input index
    in_token = offset + seq_ids
    in_ptr = input_ptr + in_token[:, None] * hidden_dim + hid_ids[None, :]

    # output index
    out_ptr = (
        output_ptr
        + b * max_len * hidden_dim
# ... omitted 8 lines ...
    )

    tl.store(
        out_ptr,
        values,
        mask=seq_mask[:, None] & (hid_ids[None, :] < hidden_dim),
    )

    # attention mask
    if tl.program_id(2) == 0:
        mask_out_ptr = mask_ptr + b * max_len + seq_ids
        tl.store(mask_out_ptr, valid_token, mask=seq_mask)
```
**EN:** Implements the pad sequence with mask kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 pad sequence with mask kernel 例程。

### Lines 247-288: function pad_sequence_with_mask
```python
def pad_sequence_with_mask(
    input_emb,  # (total_tokens, hidden)
    offsets,  # (B,)
    lengths,  # (B,)
    max_len,
):
    B = offsets.shape[0]
    hidden_dim = input_emb.shape[1]

    output = torch.zeros(
        (B, max_len, hidden_dim),
        device=input_emb.device,
        dtype=input_emb.dtype,
    )
    attn_mask = torch.empty(
        (B * max_len),
        device=input_emb.device,
        dtype=torch.bool,
    )

    BLOCK_D = triton.next_power_of_2(hidden_dim)
    BLOCK_M = triton.next_power_of_2(max_len)

    grid = (
        B,
        triton.cdiv(max_len, BLOCK_M),
        1,
    )

    pad_sequence_with_mask_kernel[grid](
        input_emb,
        offsets,
        lengths,
        output,
        attn_mask,
        max_len,
        hidden_dim,
        BLOCK_M=BLOCK_M,
        BLOCK_D=BLOCK_D,
    )

    return B, output, attn_mask
```
**EN:** Implements the pad sequence with mask routine used by this attention module.
**CN:** 实现该注意力模块使用的 pad sequence with mask 例程。

### Lines 291-315: function seqlens_expand_kernel
```python
@triton.jit
def seqlens_expand_kernel(
    extend_seq_lens_ptr,  # [N]
    seq_lens_ptr,  # [N]
    offsets_ptr,  # [N+1]
    output_ptr,  # [sum(extend_seq_lens)]
    N,
    BLOCK: tl.constexpr,
):
    pid = tl.program_id(0)

    if pid >= N:
        return

    qo_len = tl.load(extend_seq_lens_ptr + pid)
    kv_len = tl.load(seq_lens_ptr + pid)

    start = kv_len - qo_len + 1
    out_offset = tl.load(offsets_ptr + pid)

    offs = tl.arange(0, BLOCK)
    mask = offs < qo_len

    values = start + offs
    tl.store(output_ptr + out_offset + offs, values, mask=mask)
```
**EN:** Implements the seqlens expand kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 seqlens expand kernel 例程。

### Lines 318-349: function seqlens_expand_triton
```python
def seqlens_expand_triton(
    extend_seq_lens: torch.Tensor,
    seq_lens: torch.Tensor,
    total_len: int,
    max_q_len: int,
):
    """
    extend_seq_lens: [N], int32, CUDA
    seq_lens:        [N], int32, CUDA
    """
    assert extend_seq_lens.is_cuda
    assert seq_lens.is_cuda

    N = extend_seq_lens.numel()

    offsets = torch.zeros(N + 1, device=extend_seq_lens.device, dtype=torch.int32)
    offsets[1:] = torch.cumsum(extend_seq_lens, dim=0)
    output = torch.empty(total_len, device=extend_seq_lens.device, dtype=torch.int32)

    BLOCK = triton.next_power_of_2(max_q_len)
    grid = (N,)

    seqlens_expand_kernel[grid](
        extend_seq_lens,
        seq_lens,
        offsets,
        output,
        N,
        BLOCK=BLOCK,
    )

    return output
```
**EN:** Implements the seqlens expand triton routine used by this attention module.
**CN:** 实现该注意力模块使用的 seqlens expand triton 例程。

### Lines 359-384: function canonicalize_stride
```python
def canonicalize_stride(tensor: torch.Tensor) -> torch.Tensor:
    """
    Adjust degenerate strides for a tensor, make it canonical.
    """
    sizes = tensor.size()
    strides = tensor.stride()
    ndim = tensor.dim()

    need_fix = any(
        sizes[i] == 1 and strides[i] == strides[i + 1] for i in range(ndim - 1)
    )

    if not need_fix:
        return tensor

    # canonicalize the stride
    # Example:
    # - shape: [num_pages, 1, 64, 128]
    # - stride: [8192, 128, 128, 1] (wrong!)
    # Gives new stride: [8192, 8192, 128 ,1] (correct!)
    new_strides = [0] * ndim
    new_strides[-1] = 1
    for i in range(ndim - 2, -1, -1):
        new_strides[i] = new_strides[i + 1] * sizes[i + 1]

    return tensor.as_strided(sizes, new_strides)
```
**EN:** Implements the canonicalize stride routine used by this attention module.
**CN:** 实现该注意力模块使用的 canonicalize stride 例程。

### Lines 387-470: function mla_quantize_and_rope_for_fp8
```python
def mla_quantize_and_rope_for_fp8(
    q_nope: torch.Tensor,
    q_rope: torch.Tensor,
    k_nope: torch.Tensor,
    k_rope: torch.Tensor,
    pos_ids: torch.Tensor,
    cos_sin_cache: torch.Tensor,
    is_neox: bool,
    kv_lora_rank: int,
    qk_rope_head_dim: int,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    import flashinfer.rope

    """Quantize and apply RoPE for FP8 attention path.

        This function handles the FP8 quantization and RoPE application for MLA attention.
        It takes separate query/key nope and rope components, applies RoPE to the rope parts,
        quantizes all components to FP8, and merges the query components into a single tensor.

        Args:
            q_nope: Query no-position-encoding component [seq_len, num_heads, kv_lora_rank]
                - expected dtype: torch.bfloat16
            q_rope: Query RoPE component [seq_len, num_heads, qk_rope_head_dim]
                - expected dtype: torch.bfloat16
            k_nope: Key no-position-encoding component [seq_len, num_heads, kv_lora_rank]
                - expected dtype: torch.bfloat16
            k_rope: Key RoPE component [seq_len, num_heads, qk_rope_head_dim]
                - expected dtype: torch.bfloat16
            pos_ids: Position indices for each token
                - expected dtype: torch.int64 or torch.int32
            cos_sin_cache: Precomputed cosine/sine cache for RoPE
                - expected dtype: matches q_/k_ input dtype (torch.bfloat16)
# ... omitted 40 lines ...
        # Output tensor slicing: q_out contains [nope_part, rope_part]
        q_rope_out=q_out[..., kv_lora_rank:],  # RoPE part goes to end
        k_rope_out=k_rope_out,
        q_nope_out=q_out[..., :kv_lora_rank],  # Nope part goes to beginning
        k_nope_out=k_nope_out,
        # Quantization scales (set to 1.0 for no additional scaling)
        quant_scale_q=1.0,
        quant_scale_kv=1.0,
        enable_pdl=is_arch_support_pdl(),
    )

    return q_out, k_nope_out, k_rope_out
```
**EN:** Implements the mla quantize and rope for fp8 routine used by this attention module.
**CN:** 实现该注意力模块使用的 mla quantize and rope for fp8 例程。

### Lines 473-477: function concat_mla_absorb_q_general
```python
def concat_mla_absorb_q_general(q_nope, q_rope):
    if _is_cuda and q_nope.shape[-1] == 512 and q_rope.shape[-1] == 64:
        return concat_mla_absorb_q(q_nope, q_rope)
    else:
        return torch.cat([q_nope, q_rope], dim=-1)
```
**EN:** Implements the concat mla absorb q general routine used by this attention module.
**CN:** 实现该注意力模块使用的 concat mla absorb q general 例程。

### Lines 480-601: function reshape_and_cache_flash
```python
@triton.jit
def reshape_and_cache_flash(
    key_ptr,
    value_ptr,
    key_cache_ptr,
    value_cache_ptr,
    slot_mapping_ptr,
    swa_slot_mapping_ptr,
    k_scale_ptr,
    v_scale_ptr,
    block_stride,
    key_stride,
    value_stride,
    num_heads,
    head_size,
    block_size,
    HEAD_BLOCK: tl.constexpr,
    BLOCK_D: tl.constexpr,
    HAS_SWA: tl.constexpr,
    USE_SCALE: tl.constexpr,
):
    """
    Triton kernel for reshaping per-token K/V tensors into paged KV cache layout.

    Source layout:
        key/value: [num_tokens, num_heads, head_size]

    Target cache layout:
        cache: [num_blocks, block_size, num_heads, head_size]

    Each Triton program instance handles:
        - one token (program_id(0))
# ... omitted 78 lines ...

        k = k / k_scale
        v = v / v_scale

    # ----------------------------------
    # target layout
    # [block_idx, block_offset, head, dim]
    # ----------------------------------
    tgt = block_idx * block_stride + block_offset * num_heads * head_size + offs

    tl.store(key_cache_ptr + tgt, k, mask=mask)
    tl.store(value_cache_ptr + tgt, v, mask=mask)
```
**EN:** Implements the reshape and cache flash routine used by this attention module.
**CN:** 实现该注意力模块使用的 reshape and cache flash 例程。

### Lines 604-663: function launch_reshape_and_cache_flash
```python
def launch_reshape_and_cache_flash(
    key,
    value,
    key_cache,
    value_cache,
    slot_mapping,
    swa_slot_mapping=None,
    k_scale=None,
    v_scale=None,
):
    """
    Launch wrapper for reshape_and_cache_flash Triton kernel.

    This wrapper prepares launch configuration and dispatches the Triton kernel
    that writes token-major K/V tensors into paged KV cache layout.

    Args:
        key: Source key tensor [num_tokens, num_heads, head_size]
        value: Source value tensor [num_tokens, num_heads, head_size]
        key_cache: Destination key cache [num_blocks, block_size, num_heads, head_size]
        value_cache: Destination value cache [num_blocks, block_size, num_heads, head_size]
        slot_mapping: Token-to-cache slot mapping
        swa_slot_mapping: Optional SWA remapping table
        k_scale: Optional key scaling factor
        v_scale: Optional value scaling factor
    """

    num_tokens = key.shape[0]
    num_heads = key.shape[1]
    head_size = key.shape[2]

    HEAD_BLOCK = 4
# ... omitted 16 lines ...
        v_scale if v_scale is not None else key,
        key_cache.stride(0),
        key.stride(0),
        value.stride(0),
        num_heads,
        head_size,
        key_cache.shape[1],
        HEAD_BLOCK=HEAD_BLOCK,
        BLOCK_D=BLOCK_D,
        HAS_SWA=(swa_slot_mapping is not None),
        USE_SCALE=(k_scale is not None),
    )
```
**EN:** Implements the launch reshape and cache flash routine used by this attention module.
**CN:** 实现该注意力模块使用的 launch reshape and cache flash 例程。

### Lines 666-685: function _get_gptj_rotated_x
```python
@triton.jit
def _get_gptj_rotated_x(
    x,
    x_rotated_mask,
    BLOCK_D: tl.constexpr,
    BLOCK_D_HALF: tl.constexpr,
):
    # GPT-J rotary layout:
    # Pair adjacent dimensions and apply:
    # [x0, x1, x2, x3] -> [-x1, x0, -x3, x2]

    # Apply sign inversion on odd positions.
    x_rotated = tl.where(x_rotated_mask, x, -x)
    # Reshape into (D/2, 2) pairs.
    x_rotated = tl.reshape(x_rotated, (BLOCK_D_HALF, 2))
    # Swap each pair.
    x_rotated = tl.flip(x_rotated, 1)
    # Flatten back to original shape.
    x_rotated = tl.reshape(x_rotated, (BLOCK_D,))
    return x_rotated
```
**EN:** Implements the get gptj rotated x routine used by this attention module.
**CN:** 实现该注意力模块使用的 get gptj rotated x 例程。

### Lines 688-708: function _get_neox_rotated_x
```python
@triton.jit
def _get_neox_rotated_x(
    x,
    x_rotated_mask,
    BLOCK_D: tl.constexpr,
    BLOCK_D_HALF: tl.constexpr,
):
    # GPT-NeoX rotary layout:
    # Split head dimension into two halves:
    # [x0, x1, x2, x3] -> [-x2, -x3, x0, x1]

    # Keep first half positive, second half negative.
    x_rotated = tl.where(x_rotated_mask, x, -x)
    # Reshape into (2, D/2).
    x_rotated = tl.reshape(x_rotated, (2, BLOCK_D_HALF))
    # Reverse each half.
    x_rotated = tl.flip(x_rotated, 1)
    # Flatten and reverse full vector.
    x_rotated = tl.reshape(x_rotated, (BLOCK_D,))
    x_rotated = tl.flip(x_rotated, 0)
    return x_rotated
```
**EN:** Implements the get neox rotated x routine used by this attention module.
**CN:** 实现该注意力模块使用的 get neox rotated x 例程。

### Lines 711-740: function _unit_rope
```python
@triton.jit
def _unit_rope(
    x_ptrs,
    cos,
    sin,
    d_pe_offs,
    IS_NEOX: tl.constexpr,
    BLOCK_D_pe: tl.constexpr,
    BLOCK_D_HALF_pe: tl.constexpr,
):
    # Load one full attention head vector.
    x_pe = tl.load(x_ptrs)

    # Stage 1: Build rotated vector according to rotary layout.
    if IS_NEOX:
        x_rotated_mask = d_pe_offs < BLOCK_D_HALF_pe
        x_pe_rotated = _get_neox_rotated_x(
            x_pe, x_rotated_mask, BLOCK_D_pe, BLOCK_D_HALF_pe
        )
    else:
        x_rotated_mask = d_pe_offs % 2 == 0
        x_pe_rotated = _get_gptj_rotated_x(
            x_pe, x_rotated_mask, BLOCK_D_pe, BLOCK_D_HALF_pe
        )

    # Stage 2: Apply RoPE transform:
    # x' = x*cos + rotate(x)*sin
    x_pe = x_pe * cos + x_pe_rotated * sin

    return x_pe
```
**EN:** Implements the unit rope routine used by this attention module.
**CN:** 实现该注意力模块使用的 unit rope 例程。

### Lines 743-755: function _load_cos_sin
```python
@triton.jit
def _load_cos_sin(
    cos_sin_ptr,
    pos,
    d_cos_offs,
    stride_t,
    stride_d,
    freq_dim,
):
    base = pos * stride_t
    cos = tl.load(cos_sin_ptr + base + d_cos_offs * stride_d)
    sin = tl.load(cos_sin_ptr + base + (d_cos_offs + freq_dim) * stride_d)
    return cos, sin
```
**EN:** Implements the load cos sin routine used by this attention module.
**CN:** 实现该注意力模块使用的 load cos sin 例程。

### Lines 758-1203: function _fused_qk_rope_reshape_and_cache_kernel
```python
@triton.jit
def _fused_qk_rope_reshape_and_cache_kernel(
    q_ptr,
    k_ptr,
    v_ptr,
    pos_ptr,
    cos_sin_ptr,
    offs_ptr,
    key_cache_ptr,
    value_cache_ptr,
    slot_mapping_ptr,
    swa_slot_mapping_ptr,
    q_out_ptr,
    k_out_ptr,
    zeros_out_ptr,
    T,
    T_slot,
    q_stride_t,
    q_stride_h,
    q_stride_d,
    k_stride_t,
    k_stride_h,
    k_stride_d,
    v_stride_t,
    v_stride_h,
    v_stride_d,
    cos_sin_stride_t,
    cos_sin_stride_d,
    q_out_stride_t,
    q_out_stride_h,
    q_out_stride_d,
    k_out_stride_t,
# ... omitted 402 lines ...
                        + d_pe_offs * value_cache_stride_d
                        + x_off * value_cache_stride_x
                    )
                else:
                    v_out_ptrs = (
                        value_cache_ptr
                        + pid_t_slot * value_cache_stride_t
                        + pid_hk * value_cache_stride_h
                        + d_pe_offs * value_cache_stride_d
                        + pid_b * value_cache_stride_b
                    )
                tl.store(v_out_ptrs, v.to(value_cache_ptr.dtype.element_ty))
```
**EN:** Implements the fused qk rope reshape and cache kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 fused qk rope reshape and cache kernel 例程。

### Lines 1206-1396: function fused_qk_rope_reshape_and_cache
```python
def fused_qk_rope_reshape_and_cache(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    key_cache: torch.Tensor,
    value_cache: torch.Tensor,
    slot_mapping: torch.Tensor,
    pos: torch.Tensor,
    cos_sin: torch.Tensor,
    k_scale: torch.Tensor,
    v_scale: torch.Tensor,
    is_neox: bool,
    flash_layout: bool,
    apply_scale: bool = True,
    offs: torch.Tensor = None,
    q_out: torch.Tensor = None,
    k_out: torch.Tensor = None,
    output_zeros: bool = True,
    zeros_out: torch.Tensor = None,
    swa_slot_mapping=None,
):
    """
    Perform RoPE on q and k and along the last dimension and copy k and v in to key_cache and value_cache inplace

    Key parameters:
    - q: shape (T, QH, D).
    - k: shape (T_slot, KH, D).
    - v: shape (T_slot, KH, D).
    - if flash_layout:
    -     key_cache: shape (T_cache, block_size, KH, D).
    -     value_cache: shape (T_cache, block_size, KH, D).
    - else:
# ... omitted 147 lines ...
        VALUE_SHUFFLE_LAYOUT=value_shuffle_layout,
        HAVE_POS=(offs is not None),
        HAVE_K_SCALE=(k_scale is not None and apply_scale),
        HAVE_V_SCALE=(v_scale is not None and apply_scale),
        HAVE_ZEROS=output_zeros,
        HAS_SWA=(swa_slot_mapping is not None),
        num_warps=1,
    )

    if zeros_out is not None:
        return q_out.view(-1, qh * d), k_out, key_cache, value_cache, zeros_out
    return q_out.view(-1, qh * d), k_out, key_cache, value_cache
```
**EN:** Implements the fused qk rope reshape and cache routine used by this attention module.
**CN:** 实现该注意力模块使用的 fused qk rope reshape and cache 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** FlashInfer execution path / **CN:** FlashInfer 执行路径
- **EN:** Flash-style fused attention kernels / **CN:** Flash 风格融合注意力内核
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language`
- `sglang.srt.utils.is_cuda`
- `sglang.jit_kernel.utils.is_arch_support_pdl`
- `sgl_kernel.concat_mla_absorb_q`
- `flashinfer.rope`
