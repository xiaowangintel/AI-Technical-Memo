# lightning_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/lightning_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: optimized attention kernels and integration glue / 优化注意力内核与集成逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-8 — imports and setup
```python
import torch
from einops import rearrange

from vllm.triton_utils import tl, triton
from vllm.v1.attention.backends.utils import PAD_SLOT_ID
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 11-137 — function `_fwd_diag_kernel`
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
    block_offset = off_block * BLOCK
    qk_block_offset = block_offset * d
    v_block_offset = block_offset * e
    o_block_offset = block_offset * e

    # Calculate offsets for the current sub-block
    cblock_offset = off_cblock * CBLOCK
    q_cblock_offset = cblock_offset * d
    o_cblock_offset = cblock_offset * e

    # Calculate pointers to the query, key, value, and output tensors
    Q_block_ptr = (
        Q
        + qk_offset
        + qk_block_offset
        + q_cblock_offset
        + tl.arange(0, CBLOCK)[:, None] * d
        + tl.arange(0, d)[None, :]
    )
    K_trans_block_ptr = (
        K
        + qk_offset
        + qk_block_offset
        + tl.arange(0, CBLOCK)[None, :] * d
# ... omitted for brevity ...
        mask=block_offset + q_index[:, None] < n,
    )
```
**EN:** This function defines `_fwd_diag_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `Q`, `K`, `V`, `Out`, `S`, `b`. Key calls include `tl.program_id`, `tl.load`, `tl.load.to`, `tl.zeros`, `range`, `tl.store`. It writes or updates `off`, `off_bh`, `off_block`, `off_cblock`, `off_h`, `qk_offset`. The body uses loops to cover different runtime cases.
**CN:** 该函数定义 `_fwd_diag_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `Q`, `K`, `V`, `Out`, `S`, `b`。 关键调用包括 `tl.program_id`, `tl.load`, `tl.load.to`, `tl.zeros`, `range`, `tl.store`。 它会写入或更新 `off`, `off_bh`, `off_block`, `off_cblock`, `off_h`, `qk_offset`。 函数体通过循环来覆盖不同的运行时场景。

### Lines 140-246 — function `_fwd_kv_parallel`
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

    # Calculate base offsets for the current batch and head
    k_offset = off_bh * n * d
    v_offset = off_bh * n * e
    kv_offset = off_bh * NUM_BLOCK * d * e

    # Calculate pointers to the key, value, and key-value tensors
    K_trans_block_ptr = (
        K
        + k_offset
        + k_block_offset
        + tl.arange(0, CBLOCK)[None, :] * d
        + tl.arange(0, D_FBLOCK)[:, None]
    )
    V_block_ptr = (
        V
        + v_offset
        + v_block_offset
        + tl.arange(0, CBLOCK)[:, None] * e
        + tl.arange(0, E_FBLOCK)[None, :]
    )
    KV_block_ptr = (
        KV
        + kv_offset
# ... omitted for brevity ...
    # Store the result
    tl.store(KV_block_ptr, kv.to(KV_block_ptr.dtype.element_ty))
```
**EN:** This function defines `_fwd_kv_parallel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `K`, `V`, `K_decay`, `KV`, `b`, `h`. Key calls include `tl.program_id`, `tl.arange`, `tl.zeros`, `min`, `range`, `tl.store`. It writes or updates `off_bh`, `off_block`, `off_h`, `block_offset`, `k_block_offset`, `v_block_offset`. The body uses loops to cover different runtime cases.
**CN:** 该函数定义 `_fwd_kv_parallel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `K`, `V`, `K_decay`, `KV`, `b`, `h`。 关键调用包括 `tl.program_id`, `tl.arange`, `tl.zeros`, `min`, `range`, `tl.store`。 它会写入或更新 `off_bh`, `off_block`, `off_h`, `block_offset`, `k_block_offset`, `v_block_offset`。 函数体通过循环来覆盖不同的运行时场景。

### Lines 249-311 — function `_fwd_kv_reduce`
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
    s = tl.load(s_ptrs)

    # Calculate pointer to the key-value history tensor
    kv_history_offset = off_bh * d * e
    KV_HISTORY_block_ptr = (
        KV_HISTORY
        + kv_history_offset
        + tl.arange(0, D_FBLOCK)[:, None] * e
        + tl.arange(0, E_FBLOCK)[None, :]
    )

    # Load the previous key-value history
    kv_pre = tl.load(KV_HISTORY_block_ptr).to(tl.float32)

    # Process all blocks in reverse order to compute the prefix sum
    for i in range(NUM_BLOCK):
        block_size = min(n - i * BLOCK, BLOCK)
        # Compute decay factor for the current block
        block_decay = tl.exp(-s.to(tl.float32) * block_size)

        # Load the current key-value outer product
        kv_cur = tl.load(KV_block_ptr).to(tl.float32)
        # Store the previous key-value history to the current block
        tl.store(KV_block_ptr, kv_pre.to(KV_block_ptr.dtype.element_ty))
# ... omitted for brevity ...
    # Store the updated key-value history
    tl.store(KV_HISTORY_block_ptr, kv_pre)
```
**EN:** This function defines `_fwd_kv_reduce`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `S`, `KV`, `KV_HISTORY`, `b`, `h`, `n`. Key calls include `tl.program_id`, `tl.load`, `tl.load.to`, `range`, `tl.store`, `min`. It writes or updates `off_bh`, `off_h`, `kv_offset`, `KV_block_ptr`, `s_ptrs`, `s`. The body uses loops to cover different runtime cases.
**CN:** 该函数定义 `_fwd_kv_reduce`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `S`, `KV`, `KV_HISTORY`, `b`, `h`, `n`。 关键调用包括 `tl.program_id`, `tl.load`, `tl.load.to`, `range`, `tl.store`, `min`。 它会写入或更新 `off_bh`, `off_h`, `kv_offset`, `KV_block_ptr`, `s_ptrs`, `s`。 函数体通过循环来覆盖不同的运行时场景。

### Lines 314-394 — function `_fwd_none_diag_kernel`
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

    # Calculate offsets for the current batch, head, and block
    q_offset = off_bh * n * d + (n_offset + c_offset) * d
    o_offset = off_bh * n * e + (n_offset + c_offset) * e + e_offset
    kv_offset = off_bh * NUM_BLOCK * d * e + off_n * d * e + e_offset

    # Calculate pointers to the query, output, and key-value tensors
    Q_block_ptr = (
        Q + q_offset + tl.arange(0, CBLOCK)[:, None] * d + tl.arange(0, d)[None, :]
    )
    O_block_ptr = (
        Out
        + o_offset
        + tl.arange(0, CBLOCK)[:, None] * e
        + tl.arange(0, E_FBLOCK)[None, :]
    )
    KV_block_ptr = (
        KV + kv_offset + tl.arange(0, d)[:, None] * e + tl.arange(0, E_FBLOCK)[None, :]
    )

    # Load the decay rate for the current head
    S_block_ptr = S + off_h
    s = tl.load(S_block_ptr)

# ... omitted for brevity ...
        O_block_ptr, qkv.to(O_block_ptr.dtype.element_ty), mask=q_index[:, None] < n
    )
```
**EN:** This function defines `_fwd_none_diag_kernel`. It implements a low-level kernel that works on tiled tensor blocks. The main inputs are `Q`, `Out`, `S`, `KV`, `b`, `h`. Key calls include `tl.program_id`, `tl.load`, `tl.arange`, `tl.load.to`, `tl.exp`, `tl.store`. It writes or updates `off_bh`, `off_h`, `off_nc`, `off_n`, `off_c`, `off_e`.
**CN:** 该函数定义 `_fwd_none_diag_kernel`。 它实现了按分块张量执行的底层内核。 其主要输入参数包括 `Q`, `Out`, `S`, `KV`, `b`, `h`。 关键调用包括 `tl.program_id`, `tl.load`, `tl.arange`, `tl.load.to`, `tl.exp`, `tl.store`。 它会写入或更新 `off_bh`, `off_h`, `off_nc`, `off_n`, `off_c`, `off_e`。

### Lines 397-525 — class `_attention`
```python
class _attention(torch.autograd.Function):
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
# ... omitted for brevity ...

        return o, torch.cat([kv, kv_history.unsqueeze(2)], dim=2)
```
**EN:** This class defines `_attention`. It inherits from `torch.autograd.Function`. It handles attention-specific state, metadata, or tensor transforms. Important methods include `forward`. Key calls include `q.contiguous`, `k.contiguous`, `v.contiguous`, `s.contiguous`, `torch.cuda.get_device_capability`, `torch.empty`. It writes or updates `q`, `k`, `v`, `s`, `capability`, `b`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `_attention`。 它继承自 `torch.autograd.Function`。 它处理注意力相关的状态、元数据或张量变换。 重要方法包括 `forward`。 关键调用包括 `q.contiguous`, `k.contiguous`, `v.contiguous`, `s.contiguous`, `torch.cuda.get_device_capability`, `torch.empty`。 它会写入或更新 `q`, `k`, `v`, `s`, `capability`, `b`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 398-525 — method `_attention.forward`
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
        array = torch.arange(0, BLOCK, device=q.device) + 1
        k_decay = torch.exp(-s * (BLOCK - array.reshape(1, -1)))

        # Step 1: Compute diagonal blocks of attention
        grid = (b * h * NUM_BLOCK, NUM_CBLOCK)
        _fwd_diag_kernel[grid](
            q,
            k,
            v,
            o,
            s,
            b,
# ... omitted for brevity ...

        return o, torch.cat([kv, kv_history.unsqueeze(2)], dim=2)
```
**EN:** This method defines `forward`. It executes the main forward/runtime path for this component. The main inputs are `ctx`, `q`, `k`, `v`, `s`, `kv_history`. Key calls include `q.contiguous`, `k.contiguous`, `v.contiguous`, `s.contiguous`, `torch.cuda.get_device_capability`, `torch.empty`. It writes or updates `q`, `k`, `v`, `s`, `capability`, `b`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `ctx`, `q`, `k`, `v`, `s`, `kv_history`。 关键调用包括 `q.contiguous`, `k.contiguous`, `v.contiguous`, `s.contiguous`, `torch.cuda.get_device_capability`, `torch.empty`。 它会写入或更新 `q`, `k`, `v`, `s`, `capability`, `b`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 532-587 — function `lightning_attention`
```python
def lightning_attention(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    ed: torch.Tensor,
    block_size: int = 256,
    kv_history: torch.Tensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
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

    # Initialize or clone key-value history
    if kv_history is None:
        kv_history = torch.zeros(
            (q.shape[0], q.shape[1], d, e), dtype=torch.float32, device=q.device
        )
# ... omitted for brevity ...
        output = output + o
    return output, kv
```
**EN:** This function defines `lightning_attention`. Apply lightning attention algorithm to compute attention efficiently. The main inputs are `q`, `k`, `v`, `ed`, `block_size`, `kv_history`. Key calls include `len`, `range`, `ed.dim`, `ed.view`, `arr.append`, `torch.zeros`. It writes or updates `d`, `e`, `m`, `arr`, `n`, `output`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `lightning_attention`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `q`, `k`, `v`, `ed`, `block_size`, `kv_history`。 关键调用包括 `len`, `range`, `ed.dim`, `ed.view`, `arr.append`, `torch.zeros`。 它会写入或更新 `d`, `e`, `m`, `arr`, `n`, `output`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 590-670 — function `_linear_attn_decode_kernel`
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
    pad_slot_id: tl.constexpr,
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
    slot_id = tl.load(slot_idx + pid_b).to(tl.int64)

    # Skip if slot_id is PAD_SLOT_ID (padding)
    if slot_id == pad_slot_id:
        return

    batch_id = pid_b
    head_id = pid_h

    # Load decay rate for the current head
    ratio = tl.load(slope_rate + pid_h)

    # Calculate offsets for dimensions
    qk_d_offsets = tl.arange(0, D)
    v_d_offsets = tl.arange(0, BLOCK_SIZE) + pid_d * BLOCK_SIZE
    cache_d_offsets = (
        qk_d_offsets[:, None] * cache_d0_stride + v_d_offsets[None, :] * cache_d1_stride
    )

    # Calculate offsets for the current batch and head
    q_offset = batch_id * qkv_b_stride + head_id * qkv_h_stride
    k_offset = batch_id * qkv_b_stride + head_id * qkv_h_stride
    v_offset = batch_id * qkv_b_stride + head_id * qkv_h_stride

    cache_offset = slot_id * cache_b_stride + head_id * cache_h_stride

    # Create masks for loading tensors
# ... omitted for brevity ...
    tl.store(kv_ptr, kv_outer, mask=kv_mask)
    tl.store(output_ptr + q_offset + v_d_offsets, output, mask=v_mask)
```
**EN:** This function defines `_linear_attn_decode_kernel`. Kernel for linear attention decoding with KV cache. The main inputs are `q_ptr`, `k_ptr`, `v_ptr`, `kv_cache_ptr`, `slope_rate`, `slot_idx`. Key calls include `tl.program_id`, `tl.load.to`, `tl.load`, `tl.arange`, `tl.exp`, `tl.sum`. It writes or updates `pid_b`, `pid_h`, `pid_d`, `slot_id`, `batch_id`, `head_id`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_linear_attn_decode_kernel`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `q_ptr`, `k_ptr`, `v_ptr`, `kv_cache_ptr`, `slope_rate`, `slot_idx`。 关键调用包括 `tl.program_id`, `tl.load.to`, `tl.load`, `tl.arange`, `tl.exp`, `tl.sum`。 它会写入或更新 `pid_b`, `pid_h`, `pid_d`, `slot_id`, `batch_id`, `head_id`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 673-738 — function `linear_decode_forward_triton`
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
    grid = (B, H, D // BLOCK_SIZE)

    # Calculate strides for tensors
    qkv_b_stride = q.stride(0)
    qkv_h_stride = q.stride(1)

    cache_b_stride = kv_caches.stride(0)
    cache_h_stride = kv_caches.stride(1)
    cache_d0_stride = kv_caches.stride(2)
    cache_d1_stride = kv_caches.stride(3)

    # Launch the kernel
# ... omitted for brevity ...
    output = rearrange(output, "b h n d -> b n (h d)")
    return output.squeeze(1).contiguous()
```
**EN:** This function defines `linear_decode_forward_triton`. Perform linear attention decoding using Triton kernels. The main inputs are `q`, `k`, `v`, `kv_caches`, `slope_rate`, `slot_idx`. Key calls include `torch.empty_like`, `q.stride`, `kv_caches.stride`, `_linear_attn_decode_kernel`, `rearrange`, `output.squeeze.contiguous`. It writes or updates `B`, `H`, `_`, `D`, `output`, `grid`.
**CN:** 该函数定义 `linear_decode_forward_triton`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `q`, `k`, `v`, `kv_caches`, `slope_rate`, `slot_idx`。 关键调用包括 `torch.empty_like`, `q.stride`, `kv_caches.stride`, `_linear_attn_decode_kernel`, `rearrange`, `output.squeeze.contiguous`。 它会写入或更新 `B`, `H`, `_`, `D`, `output`, `grid`。

## Key Concepts / 关键概念
- [EN] Optimized attention kernels and integration glue / [CN] 优化注意力内核与集成逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Core symbols: `_fwd_diag_kernel`, `_fwd_kv_parallel`, `_fwd_kv_reduce`, `_fwd_none_diag_kernel` / [CN] 核心符号：`_fwd_diag_kernel`, `_fwd_kv_parallel`, `_fwd_kv_reduce`, `_fwd_none_diag_kernel`

## Dependencies / 依赖关系
- **External**: `torch`, `einops` / **外部依赖**: `torch`, `einops`
- **Internal**: `vllm.triton_utils`, `vllm.v1.attention.backends.utils` / **内部依赖**: `vllm.triton_utils`, `vllm.v1.attention.backends.utils`
- **Runtime traits**: Triton kernels, distributed collectives / **运行时特征**: Triton kernels, distributed collectives
