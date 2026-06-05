# bitmask_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/triton_ops/bitmask_ops.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `bitmask_ops`. It exposes primary entry points such as `apply_token_bitmask_inplace_kernel`, `apply_token_bitmask_inplace_triton`. / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `bitmask_ops` 的逻辑。 它对外提供的主要入口包括 `apply_token_bitmask_inplace_kernel`, `apply_token_bitmask_inplace_triton`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module imports, constants, and setup
```python
# Adapt from
# https://github.com/mlc-ai/xgrammar/blob/v0.1.17/python/xgrammar/kernels/apply_token_bitmask_inplace_triton.py

from typing import List, Optional, Union

import torch
import triton
import triton.language as tl

from sglang.srt.utils import get_device_core_count


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; handles grammar or regular-expression constraints.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；处理语法或正则约束。

### Lines 13-72: Function apply_token_bitmask_inplace_kernel (part 1/2)
```python
@triton.jit
def apply_token_bitmask_inplace_kernel(
    logits_ptr,
    bitmask_ptr,
    indices_ptr,
    num_rows,
    vocab_size,
    logits_strides,
    bitmask_strides,
    NUM_SMS: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    """Apply a bitmask to logits in-place using Triton. The bitmask is a 01 bitwise compressed tensor,
    where 0 means the token is masked and 1 means the token is not masked. After applying the bitmask,
    the masked logits will be set to -inf.

    Parameters
    ----------
    logits_ptr : tl.tensor
        Pointer to the logits tensor to apply the bitmask to.

    bitmask_ptr : tl.tensor
        Pointer to the bitmask tensor to apply.

    indices_ptr : Optional[tl.tensor]
        Optional pointer to indices tensor specifying which rows to apply the mask to.

    num_rows : int
        Number of rows to process. If indices_ptr is provided, this is the number of unique indices.

    vocab_size : int
        Size of the vocabulary dimension. If the logits does not have a vocab padding, this is the
        same as the logits's second dimension. Otherwise, this is the actual size of the vocabulary.

    logits_strides : int
        Stride between rows in the logits tensor.

    bitmask_strides : int
        Stride between rows in the bitmask tensor.

    NUM_SMS : int
        Number of streaming multiprocessors to use.

    BLOCK_SIZE : int
        Size of processing blocks.
    """

    pid = tl.program_id(0)
    num_blocks = tl.cdiv(vocab_size, BLOCK_SIZE)
    for work_id in tl.range(pid, num_rows * num_blocks, NUM_SMS):
        row_id = work_id // num_blocks
        block_offset = (work_id % num_blocks) * BLOCK_SIZE
        batch_id = row_id if indices_ptr is None else tl.load(indices_ptr + row_id)
        offsets = block_offset + tl.arange(0, BLOCK_SIZE)
        bitmask_offsets = block_offset // 32 + tl.arange(0, BLOCK_SIZE // 32)
        vocab_mask = offsets < vocab_size
        packed_bitmask_mask = bitmask_offsets < bitmask_strides
        packed_bitmask = tl.load(
            bitmask_ptr + batch_id * bitmask_strides + bitmask_offsets,
            packed_bitmask_mask,
```
**EN:** This callable implements `apply_token_bitmask_inplace_kernel`. It takes `logits_ptr`, `bitmask_ptr`, `indices_ptr`, `num_rows` and mainly converts data into another representation. The docstring states: "Apply a bitmask to logits in-place using Triton." This chunk is part 1 of 2 for the same logical block.
**CN:** 这一可调用对象实现了 `apply_token_bitmask_inplace_kernel`。它接收 `logits_ptr`, `bitmask_ptr`, `indices_ptr`, `num_rows`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 1/2 部分。

### Lines 73-81: Function apply_token_bitmask_inplace_kernel (part 2/2)
```python
        )
        bitmask = ((packed_bitmask[:, None] >> (tl.arange(0, 32)[None, :])) & 1) == 0
        bitmask = bitmask.reshape(BLOCK_SIZE)

        tl.store(
            logits_ptr + batch_id * logits_strides + offsets,
            -float("inf"),
            vocab_mask & bitmask,
        )
```
**EN:** This callable implements `apply_token_bitmask_inplace_kernel`. It takes `logits_ptr`, `bitmask_ptr`, `indices_ptr`, `num_rows` and mainly converts data into another representation. The docstring states: "Apply a bitmask to logits in-place using Triton." This chunk is part 2 of 2 for the same logical block.
**CN:** 这一可调用对象实现了 `apply_token_bitmask_inplace_kernel`。它接收 `logits_ptr`, `bitmask_ptr`, `indices_ptr`, `num_rows`，主要用于将数据转换为另一种表示。 该片段是同一逻辑块的第 2/2 部分。

### Lines 84-141: Function apply_token_bitmask_inplace_triton
```python
def apply_token_bitmask_inplace_triton(
    logits: torch.Tensor,
    bitmask: torch.Tensor,
    indices: Optional[Union[List[int], torch.Tensor]] = None,
):
    NUM_SMS = get_device_core_count()
    BLOCK_SIZE = 4096
    BITS_PER_BLOCK = 32

    # Check input dtype
    assert bitmask.dtype == torch.int32, "bitmask must be of type int32"

    # Check input tensor shapes.
    logits_shape = logits.shape
    bitmask_shape = bitmask.shape
    if logits.ndim == 1:
        logits_shape = (1, logits_shape[0])
    if bitmask.ndim == 1:
        bitmask_shape = (1, bitmask_shape[0])

    required_bitmask_width = (logits_shape[1] + BITS_PER_BLOCK - 1) // BITS_PER_BLOCK
    assert required_bitmask_width >= bitmask_shape[1], (
        f"Bitmask width too large: allow at most {required_bitmask_width} int32s for "
        f"logits' width {logits_shape[1]}, but got {bitmask_shape[1]}"
    )

    vocab_size = min(logits_shape[1], bitmask_shape[1] * BITS_PER_BLOCK)

    num_rows = None
    if isinstance(indices, list) or isinstance(indices, torch.Tensor):
        indices = torch.tensor(indices, dtype=torch.int32, device=logits.device)
        num_rows = indices.shape[0]
    else:
        assert (
            logits_shape[0] == bitmask_shape[0]
        ), f"batch size mismatch: logits {logits_shape[0]} vs bitmask {bitmask_shape[0]}"
        num_rows = logits_shape[0]

    if NUM_SMS > 0:
        grid = (NUM_SMS,)
    else:
        num_blocks = triton.cdiv(vocab_size, BLOCK_SIZE)
        grid = (num_rows * num_blocks,)
        NUM_SMS = triton.next_power_of_2(grid[0])

    apply_token_bitmask_inplace_kernel[grid](
        logits,
        bitmask,
        indices,
        num_rows,
        vocab_size,
        logits_shape[1],
        bitmask_shape[1],
        NUM_SMS,
        BLOCK_SIZE,
        num_warps=BLOCK_SIZE // 32 // (16 // logits.element_size()),
        num_stages=3,
    )
```
**EN:** This callable implements `apply_token_bitmask_inplace_triton`. It takes `logits`, `bitmask`, `indices` and mainly converts data into another representation. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `apply_token_bitmask_inplace_triton`。它接收 `logits`, `bitmask`, `indices`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查。

## Key Concepts / 关键概念
- `apply_token_bitmask_inplace_kernel`: converts data into another representation / 将数据转换为另一种表示
- `apply_token_bitmask_inplace_triton`: converts data into another representation / 将数据转换为另一种表示

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: `torch`, `triton`, `triton.language`
- **Internal modules / 内部模块**: `sglang.srt.utils`
