# chunked_sgmv_shrink.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/triton_ops/chunked_sgmv_shrink.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file provides Triton kernels or kernel wrappers for LoRA-specific matrix, embedding, or expert operations. It focuses on GPU-efficient execution for high-throughput adapter inference. / 该文件提供了面向 LoRA 的 Triton 内核或内核封装，用于矩阵、Embedding 或专家相关运算。它重点优化高吞吐适配器推理时的 GPU 执行效率。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9: Module header, imports, and shared constants
```python
import torch
import triton
import triton.language as tl

from sglang.srt.lora.triton_ops.lora_tuning_config import get_lora_shrink_config
from sglang.srt.lora.utils import LoRABatchInfo
from sglang.srt.utils import cached_triton_kernel
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 10-69: chunked lora shrink kernel function (part 1/2)
```python
@cached_triton_kernel(
    lambda _, kwargs: (kwargs["K"], kwargs["NUM_SLICES"], kwargs["BLOCK_M"])
)
@triton.jit(do_not_specialize=["num_segs"])
def _chunked_lora_shrink_kernel(
    # Pointers to matrices
    x,
    weights,
    output,
    # Information on sequence lengths,ranks and weight id
    seg_indptr,
    weight_indices,
    lora_ranks,
    permutation,
    num_segs,
    # Meta parameters
    N: tl.constexpr,  # num_slices * r
    K: tl.constexpr,  # input_dim
    NUM_SLICES: tl.constexpr,
    BLOCK_M: tl.constexpr,
    BLOCK_N: tl.constexpr,
    BLOCK_K: tl.constexpr,
):
    """
    Computes a chunked SGMV for LoRA shrink operations.

    The kernel ensures that output[seg_start:seg_start + seg_len, :rank * num_slices]
    stores the product of the input `x` and the LoRA weights for the corresponding
    sequence. This implies that when rank is 0, the kernel is essentially a no-op,
    as output[seg_start:seg_start + seg_len, :0] is trivially correct (empty).

    Args:
        x (torch.Tensor): The input activations tensor of shape `(s, K)`, where `s`
            is the sum of all sequence lengths in the batch.
        weights (torch.Tensor): The LoRA A weights for all available adapters,
            with shape `(num_lora, N, K)` where N = num_slices * r.
        output (torch.Tensor): The output tensor of shape `(s, N)`.
    """
    x_stride_1: tl.constexpr = 1
    x_stride_0: tl.constexpr = K

    w_stride_0: tl.constexpr = N * K
    w_stride_1: tl.constexpr = K
    w_stride_2: tl.constexpr = 1

    output_stride_0: tl.constexpr = N
    output_stride_1: tl.constexpr = 1

    pid_s = tl.program_id(1)
    if pid_s >= num_segs:
        return

    pid_n = tl.program_id(0)

    # Current block computes sequence with batch_id,
    # which starts from row seg_start of x with length seg_len
    w_index = tl.load(weight_indices + pid_s)
    rank = tl.load(lora_ranks + w_index)

    # If rank is 0, this kernel becomes a no-op as the output is always trivially correct.
```
**EN:** This block uses `_chunked_lora_shrink_kernel` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `_chunked_lora_shrink_kernel` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 70-120: chunked lora shrink kernel function (part 2/2)
```python
    if rank == 0:
        return

    seg_start = tl.load(seg_indptr + pid_s)
    seg_end = tl.load(seg_indptr + pid_s + 1)

    # Adjust N dim according to the specific LoRA adapter
    cur_n = tl.minimum(N, rank * NUM_SLICES)

    # Map logical sequence index to physical index
    s_offset_logical = tl.arange(0, BLOCK_M) + seg_start
    s_offset_physical = tl.load(
        permutation + s_offset_logical, mask=s_offset_logical < seg_end
    )

    n_offset = tl.arange(0, BLOCK_N) + pid_n * BLOCK_N
    k_offset = tl.arange(0, BLOCK_K)
    x_ptrs = x + (
        s_offset_physical[:, None] * x_stride_0 + k_offset[None, :] * x_stride_1
    )
    w_ptrs = (weights + w_index * w_stride_0) + (
        k_offset[:, None] * w_stride_2 + n_offset[None, :] * w_stride_1
    )

    # Iterate to compute the block in output matrix
    partial_sum = tl.zeros((BLOCK_M, BLOCK_N), dtype=tl.float32)
    for k in range(0, tl.cdiv(K, BLOCK_K)):
        x_tile = tl.load(
            x_ptrs,
            mask=(s_offset_logical[:, None] < seg_end)
            & (k_offset[None, :] < K - k * BLOCK_K),
            other=0.0,
        )
        w_tile = tl.load(
            w_ptrs,
            mask=(k_offset[:, None] < K - k * BLOCK_K) & (n_offset[None, :] < cur_n),
            other=0.0,
        )
        partial_sum += tl.dot(x_tile, w_tile)

        x_ptrs += BLOCK_K * x_stride_1
        w_ptrs += BLOCK_K * w_stride_2

    # Store result to output matrix
    partial_sum = partial_sum.to(x.dtype.element_ty)
    output_ptr = output + (
        s_offset_physical[:, None] * output_stride_0
        + n_offset[None, :] * output_stride_1
    )
    output_mask = (s_offset_logical[:, None] < seg_end) & (n_offset[None, :] < cur_n)
    tl.store(output_ptr, partial_sum, mask=output_mask)
```
**EN:** This block uses `_chunked_lora_shrink_kernel` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `_chunked_lora_shrink_kernel` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 123-182: chunked sgmv lora shrink forward function (part 1/2)
```python
def chunked_sgmv_lora_shrink_forward(
    x: torch.Tensor,
    weights: torch.Tensor,
    batch_info: LoRABatchInfo,
    num_slices: int,
) -> torch.Tensor:
    # x: (s, input_dim)
    # weights: (num_lora, num_slices * r, input_dim)
    # output: (s, num_slices * r)
    # num_slices: qkv=3, gate_up=2, others=1
    # when called with multiple slices, the weights.shape[-2] will be num_slices * r
    # input_dim is much larger than r

    assert x.is_contiguous()
    assert weights.is_contiguous()
    assert len(x.shape) == 2
    assert len(weights.shape) == 3

    # Block shapes — use auto-tuned config if available, else defaults
    BLOCK_M = batch_info.max_len
    # weights shape is (num_lora, num_slices * rank, input_dim)
    MAX_RANK = weights.shape[1] // num_slices
    config = get_lora_shrink_config(
        K=weights.shape[2], R=MAX_RANK, num_slices=num_slices, chunk_size=BLOCK_M
    )
    BLOCK_N = config["BLOCK_N"]
    BLOCK_K = config["BLOCK_K"]

    S = x.shape[0]
    N = weights.shape[1]
    K = weights.shape[2]
    assert x.shape[-1] == K

    num_segments = batch_info.num_segments
    grid = (
        triton.cdiv(N, BLOCK_N),
        batch_info.bs if batch_info.use_cuda_graph else num_segments,
    )

    # Optional launch params from tuned config
    extra_kwargs = {}
    if "num_warps" in config:
        extra_kwargs["num_warps"] = config["num_warps"]
    if "num_stages" in config:
        extra_kwargs["num_stages"] = config["num_stages"]

    output = torch.empty((S, N), device=x.device, dtype=x.dtype)
    _chunked_lora_shrink_kernel[grid](
        x=x,
        weights=weights,
        output=output,
        seg_indptr=batch_info.seg_indptr,
        weight_indices=batch_info.weight_indices,
        lora_ranks=batch_info.lora_ranks,
        permutation=batch_info.permutation,
        num_segs=num_segments,
        # constants
        N=N,
        K=K,
        NUM_SLICES=num_slices,
```
**EN:** This block uses `chunked_sgmv_lora_shrink_forward` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `chunked_sgmv_lora_shrink_forward` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 183-189: chunked sgmv lora shrink forward function (part 2/2)
```python
        BLOCK_M=BLOCK_M,
        BLOCK_N=BLOCK_N,
        BLOCK_K=BLOCK_K,
        **extra_kwargs,
    )

    return output
```
**EN:** This block uses `chunked_sgmv_lora_shrink_forward` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `chunked_sgmv_lora_shrink_forward` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- Triton GPU kernels / Triton GPU 内核
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算
- Custom GPU kernels / 自定义 GPU 内核

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.lora.triton_ops.lora_tuning_config`
- `sglang.srt.lora.utils`
- `sglang.srt.utils`
### External / 外部
- `torch`
- `triton`
