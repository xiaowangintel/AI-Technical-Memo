# lightning_attn.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/linear/lightning_attn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages lightning attn logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 lightning attn 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 5-8: imports
```python
import torch
import triton
import triton.language as tl
from einops import rearrange
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 11-137: function _fwd_diag_kernel
```python
@triton.jit
def _fwd_diag_kernel(
    Q,
    K,
    V,
    Out,
    S,
    b: tl.constexpr,
    h: tl.constexpr,
    n,
    d: tl.constexpr,
    e: tl.constexpr,
    BLOCK: tl.constexpr,
    NUM_BLOCK,
    CBLOCK: tl.constexpr,
):
    # This kernel computes the diagonal blocks of the attention matrix
    # Each diagonal block represents attention
    # where queries attend to keys in the same block
    off = tl.program_id(0)
    off_bh = off // NUM_BLOCK  # batch-head index
    off_block = off % NUM_BLOCK  # block index within the sequence
    off_cblock = tl.program_id(1)  # sub-block index within a block

    off_h = off_bh % h  # head index

    # Calculate base offsets for the current batch and head
    qk_offset = off_bh * n * d
    v_offset = off_bh * n * e
    o_offset = off_bh * n * e

    # Calculate offsets for the current block
# ... omitted 83 lines ...
        qkv += tl.dot(qk, v)

        # Move to the next sub-block
        K_trans_block_ptr += CBLOCK * d
        V_block_ptr += CBLOCK * e

    # Store the result
    tl.store(
        O_block_ptr,
        qkv.to(O_block_ptr.dtype.element_ty),
        mask=block_offset + q_index[:, None] < n,
    )
```
**EN:** Implements the fwd diag kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 fwd diag kernel 例程。

### Lines 140-243: function _fwd_kv_parallel
```python
@triton.jit
def _fwd_kv_parallel(
    K,
    V,
    K_decay,
    KV,
    b: tl.constexpr,
    h: tl.constexpr,
    n,
    d: tl.constexpr,
    e: tl.constexpr,
    BLOCK: tl.constexpr,
    NUM_BLOCK,
    D_FBLOCK: tl.constexpr,
    E_FBLOCK: tl.constexpr,
    NUM_FBLOCK: tl.constexpr,
    CBLOCK: tl.constexpr,
    NUM_CBLOCK: tl.constexpr,
):
    # This kernel computes the key-value outer
    # products for each block in parallel
    off_bh = tl.program_id(0)  # batch-head index
    off_block = tl.program_id(1)  # block index

    off_h = off_bh % h  # head index

    block_offset = off_block * BLOCK

    # Calculate offsets for the current block
    k_block_offset = block_offset * d
    v_block_offset = block_offset * e
    kv_block_offset = off_block * d * e
# ... omitted 60 lines ...

        # Load decay factor and compute weighted key-value outer product
        k_decay = tl.load(k_decay_ptr)
        kv += tl.dot(k_trans * k_decay, v)

        # Move to the next sub-block
        K_trans_block_ptr += CBLOCK * d
        V_block_ptr += CBLOCK * e
        k_decay_ptr += CBLOCK

    # Store the result
    tl.store(KV_block_ptr, kv.to(KV_block_ptr.dtype.element_ty))
```
**EN:** Implements the fwd kv parallel routine used by this attention module.
**CN:** 实现该注意力模块使用的 fwd kv parallel 例程。

### Lines 246-308: function _fwd_kv_reduce
```python
@triton.jit
def _fwd_kv_reduce(
    S,
    KV,
    KV_HISTORY,
    b: tl.constexpr,
    h: tl.constexpr,
    n,
    d: tl.constexpr,
    e: tl.constexpr,
    BLOCK: tl.constexpr,
    NUM_BLOCK,
    D_FBLOCK: tl.constexpr,
    E_FBLOCK: tl.constexpr,
):
    # This kernel reduces the key-value outer products
    # across blocks and updates the KV history
    off_bh = tl.program_id(0)  # batch-head index
    off_h = off_bh % h  # head index

    kv_offset = off_bh * NUM_BLOCK * d * e

    # Calculate pointer to the key-value tensor
    KV_block_ptr = (
        KV
        + kv_offset
        + tl.arange(0, D_FBLOCK)[:, None] * e
        + tl.arange(0, E_FBLOCK)[None, :]
    )

    # Load the decay rate for the current head
    s_ptrs = S + off_h
# ... omitted 19 lines ...

        # Load the current key-value outer product
        kv_cur = tl.load(KV_block_ptr).to(tl.float32)
        # Store the previous key-value history to the current block
        tl.store(KV_block_ptr, kv_pre.to(KV_block_ptr.dtype.element_ty))

        # Update the key-value history with the current block
        kv_pre = block_decay * kv_pre + kv_cur
        KV_block_ptr += d * e

    # Store the updated key-value history
    tl.store(KV_HISTORY_block_ptr, kv_pre)
```
**EN:** Implements the fwd kv reduce routine used by this attention module.
**CN:** 实现该注意力模块使用的 fwd kv reduce 例程。

### Lines 311-391: function _fwd_none_diag_kernel
```python
@triton.jit
def _fwd_none_diag_kernel(
    Q,
    Out,
    S,
    KV,
    b: tl.constexpr,
    h: tl.constexpr,
    n,
    d: tl.constexpr,
    e: tl.constexpr,
    BLOCK: tl.constexpr,
    NUM_BLOCK,
    E_FBLOCK: tl.constexpr,
    CBLOCK: tl.constexpr,
    NUM_CBLOCK: tl.constexpr,
):
    # This kernel computes the non-diagonal blocks of the attention matrix
    # Each non-diagonal block represents attention
    # where queries attend to keys in different blocks
    off_bh = tl.program_id(0)  # batch-head index
    off_h = off_bh % h  # head index

    off_nc = tl.program_id(1)
    off_n = off_nc // NUM_CBLOCK  # block index
    off_c = off_nc % NUM_CBLOCK  # sub-block index
    off_e = tl.program_id(2)  # output feature block index

    n_offset = off_n * BLOCK
    c_offset = off_c * CBLOCK
    e_offset = off_e * E_FBLOCK
    block_offset = n_offset + c_offset
# ... omitted 37 lines ...
    qkv_none_diag = tl.dot(q, kv) * q_decay

    # Load diagonal attention output (computed by _fwd_diag_kernel)
    qkv_diag = tl.load(O_block_ptr, mask=q_index[:, None] < n, other=0.0).to(tl.float32)

    # Combine diagonal and non-diagonal attention outputs
    qkv = qkv_diag + qkv_none_diag

    # Store the result
    tl.store(
        O_block_ptr, qkv.to(O_block_ptr.dtype.element_ty), mask=q_index[:, None] < n
    )
```
**EN:** Implements the fwd none diag kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 fwd none diag kernel 例程。

### Lines 394-395: class _attention
```python
class _attention(torch.autograd.Function):
```
**EN:** Defines the attention type and the state it exposes to the rest of the attention stack.
**CN:** 定义 attention 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 396-523: method _attention.forward
```python
    @staticmethod
    def forward(ctx, q, k, v, s, kv_history):
        # Forward pass of the lightning attention algorithm
        q = q.contiguous()
        k = k.contiguous()
        v = v.contiguous()
        s = s.contiguous()

        # Check CUDA compute capability
        capability = torch.cuda.get_device_capability()
        if capability[0] < 8:
            raise RuntimeError(
                "Flash attention currently only supported",
                "for compute capability >= 80",
            )

        # Get input dimensions
        b, h, n, d = q.shape
        e = v.shape[-1]

        # Initialize output tensor
        o = torch.empty((b, h, n, e), dtype=q.dtype, device=q.device)

        # Set block sizes
        BLOCK = 256
        NUM_BLOCK = triton.cdiv(n, BLOCK)

        CBLOCK = 32
        NUM_CBLOCK = BLOCK // CBLOCK
        assert BLOCK % CBLOCK == 0, "BLOCK must be a multiple of CBLOCK"

        # Compute decay factors for keys
# ... omitted 84 lines ...
            BLOCK=BLOCK,
            NUM_BLOCK=NUM_BLOCK,
            E_FBLOCK=E_FBLOCK,
            CBLOCK=CBLOCK,
            NUM_CBLOCK=NUM_CBLOCK,
        )

        # Save tensors for backward pass
        ctx.save_for_backward(q, k, v, s, kv)
        ctx.BLOCK = BLOCK

        return o, torch.cat([kv, kv_history.unsqueeze(2)], dim=2)
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 527-527: module constants
```python
lightning_attention_ = _attention.apply
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 530-578: function lightning_attention
```python
def lightning_attention(q, k, v, ed, block_size=256, kv_history=None):
    """
    Apply lightning attention algorithm
    to compute attention efficiently.

    Args:
        q: Query tensor of shape [batch, heads, seq_len, dim]
        k: Key tensor of shape [batch, heads, seq_len, dim]
        v: Value tensor of shape [batch, heads, seq_len, dim_v]
        ed: Decay rate tensor of shape [heads]
        block_size: Size of blocks for block-sparse attention
        kv_history: Optional key-value history from previous computations

    Returns:
        output: Attention output
        kv: Updated key-value history
    """
    d = q.shape[-1]
    e = v.shape[-1]

    if ed.dim() == 1:
        ed = ed.view(1, -1, 1, 1)

    # Split the computation into chunks for better parallelism
    m = 128 if d >= 128 else 64
    assert d % m == 0, f"Dimension d ({d}) must be divisible by m ({m})"
    arr = [m * i for i in range(d // m + 1)]
    if arr[-1] != d:
        arr.append(d)
    n = len(arr)
    output = 0

# ... omitted 5 lines ...
    else:
        kv_history = kv_history.clone().contiguous()

    # Process each chunk and accumulate results
    for i in range(n - 1):
        s = arr[i]
        e = arr[i + 1]
        q1 = q[..., s:e]
        k1 = k[..., s:e]
        o, kv = lightning_attention_(q1, k1, v, ed, kv_history)
        output = output + o
    return output, kv
```
**EN:** Implements the lightning attention routine used by this attention module.
**CN:** 实现该注意力模块使用的 lightning attention 例程。

### Lines 581-660: function _linear_attn_decode_kernel
```python
@triton.jit
def _linear_attn_decode_kernel(
    q_ptr,
    k_ptr,
    v_ptr,
    kv_cache_ptr,
    slope_rate,
    slot_idx,
    output_ptr,
    D: tl.constexpr,
    qkv_b_stride,
    qkv_h_stride,
    cache_b_stride,
    cache_h_stride,
    cache_d0_stride,
    cache_d1_stride,
    BLOCK_SIZE: tl.constexpr,
):
    """
    Kernel for linear attention decoding with KV cache.

    This kernel computes attention for a single token using the KV cache.
    """
    pid_b = tl.program_id(0)  # batch index
    pid_h = tl.program_id(1)  # head index
    pid_d = tl.program_id(2)  # dimension block index

    # Load slot index for the current batch
    slot_id = tl.load(slot_idx + pid_b)

    # Skip if slot_id is -1 (padding)
    if slot_id == -1:
# ... omitted 36 lines ...
    ratio = tl.exp(-ratio)
    kv_ptr = kv_cache_ptr + cache_offset + cache_d_offsets
    kv_cache_old = tl.load(kv_ptr, mask=kv_mask, other=0.0)
    kv_outer = kv_outer + ratio * kv_cache_old

    # Compute attention output
    output = q[:, None].to(tl.float32) * kv_outer
    output = tl.sum(output, axis=0)

    # Update KV cache and store output
    tl.store(kv_ptr, kv_outer, mask=kv_mask)
    tl.store(output_ptr + q_offset + v_d_offsets, output, mask=v_mask)
```
**EN:** Implements the linear attn decode kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 linear attn decode kernel 例程。

### Lines 663-727: function linear_decode_forward_triton
```python
def linear_decode_forward_triton(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    kv_caches: torch.Tensor,
    slope_rate: torch.Tensor,
    slot_idx: torch.Tensor,
    BLOCK_SIZE: int = 32,
) -> torch.Tensor:
    """
    Perform linear attention decoding using Triton kernels.

    Args:
        q: Query tensor of shape [B, H, 1, D]
        k: Key tensor of shape [B, H, 1, D]
        v: Value tensor of shape [B, H, 1, D]
        kv_caches: Key-value cache tensor
        slope_rate: Decay rate tensor
        slot_idx: Slot indices for batches
        BLOCK_SIZE: Size of blocks for processing

    Returns:
        output: Attention output tensor
    """
    B, H, _, D = q.shape
    assert k.shape == (B, H, 1, D)
    assert v.shape == (B, H, 1, D)

    # Initialize output tensor
    output = torch.empty_like(q)

    # Set grid dimensions for the kernel
# ... omitted 21 lines ...
        qkv_b_stride,
        qkv_h_stride,
        cache_b_stride,
        cache_h_stride,
        cache_d0_stride,
        cache_d1_stride,
        BLOCK_SIZE=BLOCK_SIZE,
    )

    # Reshape output and return
    output = rearrange(output, "b h n d -> b n (h d)")
    return output.squeeze(1).contiguous()
```
**EN:** Runs the forward-path logic for linear decode forward triton, transforming tensors and dispatching the required compute steps.
**CN:** 执行 linear decode forward triton 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 730-740: class BailingLinearKernel
```python
class BailingLinearKernel:
    """
    Linear attention kernel implementation for Bailing models.

    This class is adapted from MiniMaxText01LinearKernel in vllm:
    https://github.com/vllm-project/vllm/blob/a9138e85b14047e06300685b48e3485b995425fb/vllm/model_executor/models/minimax_text_01.py#L289

    The implementation maintains the same functionality while being renamed to
    match our Bailing model naming convention.
    """
```
**EN:** Defines the bailing linear kernel type and the state it exposes to the rest of the attention stack.
**CN:** 定义 bailing linear kernel 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 741-767: method BailingLinearKernel.jit_linear_forward_prefix
```python
    @staticmethod
    def jit_linear_forward_prefix(
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        kv_caches: torch.Tensor,
        slope_rate: torch.Tensor,
        block_size: int,
        layer_idx: int = None,
        **kwargs,
    ) -> torch.Tensor:

        slope_rate = slope_rate.to(torch.float32)
        should_pad_dim = q.dim() == 3
        if should_pad_dim:
            q = q.unsqueeze(0)
            k = k.unsqueeze(0)
            v = v.unsqueeze(0)
        b, h, n, d = q.shape
        e = d
        kv_history = kv_caches.reshape(1, h, d, e).contiguous()
        output, kv_history = lightning_attention(
            q, k, v, slope_rate, block_size=block_size, kv_history=kv_history
        )
        kv_caches.copy_(kv_history[:, :, -1, :, :].reshape(h, d, e))
        assert output.shape[0] == 1, "batch size must be 1"
        return output.squeeze(0).transpose(0, 1).reshape([n, h * d]).contiguous()
```
**EN:** Runs the forward-path logic for jit linear forward prefix, transforming tensors and dispatching the required compute steps.
**CN:** 执行 jit linear forward prefix 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Position encoding handling / **CN:** 位置编码处理
- **EN:** Compressed or sparse attention helpers / **CN:** 压缩或稀疏注意力辅助逻辑

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language`
- `einops.rearrange`
