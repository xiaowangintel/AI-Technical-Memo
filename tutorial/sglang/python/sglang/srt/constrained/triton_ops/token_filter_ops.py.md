# token_filter_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/triton_ops/token_filter_ops.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `token_filter_ops`. The module docstring frames it as: "Triton kernels for token filter operations." / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `token_filter_ops` 的逻辑。 它对外提供的主要入口包括 `reset_vocab_mask_kernel`, `set_token_filter_batch_kernel`, `_compute_grid`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Module imports, constants, and setup
```python
# Copyright 2026 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""Triton kernels for token filter operations."""

from collections import OrderedDict
from typing import List

import torch
import triton
import triton.language as tl

from sglang.srt.utils import get_device_core_count


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 26-54: Function reset_vocab_mask_kernel
```python
@triton.jit
def reset_vocab_mask_kernel(
    vocab_mask_ptr,
    batch_idx: int,
    num_elements: int,
    reset_value: tl.constexpr,
):
    """Reset the vocab mask for a specific batch index to a given value.

    Parameters
    ----------
    vocab_mask_ptr : tl.tensor
        Pointer to the vocab mask tensor.

    batch_idx : int
        The batch index to reset.

    num_elements : int
        Number of int32 elements in the vocab mask for each batch.

    reset_value : int
        The value to reset the vocab mask to (typically -1 or 0).
    """
    pid = tl.program_id(0)
    num_threads = tl.num_programs(0)

    for i in tl.range(pid, num_elements, num_threads):
        offset = batch_idx * num_elements + i
        tl.store(vocab_mask_ptr + offset, reset_value)
```
**EN:** This callable implements `reset_vocab_mask_kernel`. It takes `vocab_mask_ptr`, `batch_idx`, `num_elements`, `reset_value` and mainly applies configuration to mutable state. The docstring states: "Reset the vocab mask for a specific batch index to a given value."
**CN:** 这一可调用对象实现了 `reset_vocab_mask_kernel`。它接收 `vocab_mask_ptr`, `batch_idx`, `num_elements`, `reset_value`，主要用于将配置写入可变状态。

### Lines 57-105: Function set_token_filter_batch_kernel
```python
@triton.jit
def set_token_filter_batch_kernel(
    vocab_mask_ptr,
    token_ids_ptr,
    batch_idx: int,
    num_tokens: int,
    num_elements: int,
    is_allowed: tl.constexpr,
):
    """Set or clear specific tokens in the vocab mask for a batch.

    Each token ID maps to a specific bit in the int32 bitmask array.
    The kernel sets or clears those bits using atomic operations.

    Parameters
    ----------
    vocab_mask_ptr : tl.tensor
        Pointer to the vocab mask tensor.

    token_ids_ptr : tl.tensor
        Pointer to the token IDs to set/clear.

    batch_idx : int
        The batch index to modify.

    num_tokens : int
        Number of tokens to process.

    num_elements : int
        Number of int32 elements in the vocab mask for each batch.

    is_allowed : bool
        If True, set the bit to 1 (allow token).
        If False, clear the bit to 0 (block token).
    """
    pid = tl.program_id(0)
    num_threads = tl.num_programs(0)

    for i in tl.range(pid, num_tokens, num_threads):
        token_id = tl.load(token_ids_ptr + i)
        element_idx = token_id // 32
        bit_idx = token_id % 32

        offset = batch_idx * num_elements + element_idx

        if is_allowed:
            tl.atomic_or(vocab_mask_ptr + offset, 1 << bit_idx)
        else:
            tl.atomic_and(vocab_mask_ptr + offset, ~(1 << bit_idx))
```
**EN:** This callable implements `set_token_filter_batch_kernel`. It takes `vocab_mask_ptr`, `token_ids_ptr`, `batch_idx`, `num_tokens` and mainly converts data into another representation. The docstring states: "Set or clear specific tokens in the vocab mask for a batch."
**CN:** 这一可调用对象实现了 `set_token_filter_batch_kernel`。它接收 `vocab_mask_ptr`, `token_ids_ptr`, `batch_idx`, `num_tokens`，主要用于将数据转换为另一种表示。

### Lines 106-114: Module-level constants and helpers
```python


_cached_num_sms = None
_cached_token_id_tensors: OrderedDict[tuple[int, tuple[int, ...]], torch.Tensor] = (
    OrderedDict()
)
_MAX_TOKEN_ID_TENSOR_CACHE_SIZE = 32


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 115-121: Function _compute_grid
```python
def _compute_grid(work_items: int):
    global _cached_num_sms
    if _cached_num_sms is None:
        _cached_num_sms = get_device_core_count()
    if _cached_num_sms > 0:
        return (min(_cached_num_sms, work_items),)
    return (work_items,)
```
**EN:** This callable implements `_compute_grid`. It takes `work_items` and mainly implements compute grid.
**CN:** 这一可调用对象实现了 `_compute_grid`。它接收 `work_items`，主要用于实现 compute grid 相关逻辑。

### Lines 124-137: Function _get_cached_token_ids_tensor
```python
def _get_cached_token_ids_tensor(
    token_ids: List[int], device: torch.device
) -> torch.Tensor:
    key = (device.index or 0, tuple(token_ids))
    cached = _cached_token_id_tensors.get(key)
    if cached is not None:
        _cached_token_id_tensors.move_to_end(key)
        return cached

    token_ids_tensor = torch.tensor(token_ids, dtype=torch.int32, device=device)
    _cached_token_id_tensors[key] = token_ids_tensor
    if len(_cached_token_id_tensors) > _MAX_TOKEN_ID_TENSOR_CACHE_SIZE:
        _cached_token_id_tensors.popitem(last=False)
    return token_ids_tensor
```
**EN:** This callable implements `_get_cached_token_ids_tensor`. It takes `token_ids`, `device` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `_get_cached_token_ids_tensor`。它接收 `token_ids`, `device`，主要用于将数据转换为另一种表示。

### Lines 140-175: Function set_token_filter_triton
```python
def set_token_filter_triton(
    vocab_mask: torch.Tensor,
    token_ids: List[int],
    batch_idx: int,
    is_allowed: bool = True,
    reset_vocab_mask: bool = True,
):
    """Set or clear specific tokens in the vocab mask using Triton."""
    assert vocab_mask.device.type == "cuda"

    num_elements = vocab_mask.shape[1]

    if reset_vocab_mask:
        reset_value = 0 if is_allowed else -1
        reset_vocab_mask_kernel[_compute_grid(num_elements)](
            vocab_mask,
            batch_idx,
            num_elements,
            reset_value,
            num_warps=4,
        )

    if not token_ids:
        return

    num_tokens = len(token_ids)
    token_ids_tensor = _get_cached_token_ids_tensor(token_ids, vocab_mask.device)
    set_token_filter_batch_kernel[_compute_grid(num_tokens)](
        vocab_mask,
        token_ids_tensor,
        batch_idx,
        num_tokens,
        num_elements,
        is_allowed,
        num_warps=4,
    )
```
**EN:** This callable implements `set_token_filter_triton`. It takes `vocab_mask`, `token_ids`, `batch_idx`, `is_allowed` and mainly converts data into another representation. The docstring states: "Set or clear specific tokens in the vocab mask using Triton." In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `set_token_filter_triton`。它接收 `vocab_mask`, `token_ids`, `batch_idx`, `is_allowed`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查。

## Key Concepts / 关键概念
- `reset_vocab_mask_kernel`: applies configuration to mutable state / 将配置写入可变状态
- `set_token_filter_batch_kernel`: converts data into another representation / 将数据转换为另一种表示
- `_MAX_TOKEN_ID_TENSOR_CACHE_SIZE`: module constant or capability flag / 模块常量或能力标记
- `_compute_grid`: implements compute grid / 实现 compute grid 相关逻辑
- `_get_cached_token_ids_tensor`: converts data into another representation / 将数据转换为另一种表示
- `set_token_filter_triton`: converts data into another representation / 将数据转换为另一种表示

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections`, `typing`
- **Third-party / 第三方**: `torch`, `triton`, `triton.language`
- **Internal modules / 内部模块**: `sglang.srt.utils`
