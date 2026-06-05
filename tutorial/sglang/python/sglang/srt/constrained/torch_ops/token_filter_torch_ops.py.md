# token_filter_torch_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/torch_ops/token_filter_torch_ops.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `token_filter_torch_ops`. The module docstring frames it as: "Torch fallback for token filter operations (non-CUDA devices and HIP)." / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `token_filter_torch_ops` 的逻辑。 它对外提供的主要入口包括 `set_token_filter_torch`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Module imports, constants, and setup
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
"""Torch fallback for token filter operations (non-CUDA devices and HIP).

Sets or clears specific bits in an int32 bitmask by token ID.  The token list
is typically tiny (< 10 entries); aggregation is done in Python with the actual
bitmask operations using torch tensor indexing.
"""

import ctypes
from typing import List

import torch


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 27-63: Function set_token_filter_torch
```python
def set_token_filter_torch(
    vocab_mask: torch.Tensor,
    token_ids: List[int],
    batch_idx: int,
    is_allowed: bool = True,
    reset_vocab_mask: bool = True,
):
    if reset_vocab_mask:
        vocab_mask[batch_idx].fill_(-1 if (not is_allowed) else 0)

    if not token_ids:
        return

    # Aggregate bit masks per int32 element to handle duplicate indices.
    aggregated: dict[int, int] = {}
    for token_id in token_ids:
        element_idx = token_id // 32
        bit_idx = token_id % 32
        aggregated[element_idx] = aggregated.get(element_idx, 0) | (1 << bit_idx)

    row = vocab_mask[batch_idx]
    element_indices = torch.tensor(
        list(aggregated.keys()), dtype=torch.long, device=row.device
    )
    bitmasks = torch.tensor(
        [
            ctypes.c_int32(mask if is_allowed else ~mask).value
            for mask in aggregated.values()
        ],
        dtype=row.dtype,
        device=row.device,
    )

    if is_allowed:
        row[element_indices] = torch.bitwise_or(row[element_indices], bitmasks)
    else:
        row[element_indices] = torch.bitwise_and(row[element_indices], bitmasks)
```
**EN:** This callable implements `set_token_filter_torch`. It takes `vocab_mask`, `token_ids`, `batch_idx`, `is_allowed` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `set_token_filter_torch`。它接收 `vocab_mask`, `token_ids`, `batch_idx`, `is_allowed`，主要用于将数据转换为另一种表示。

## Key Concepts / 关键概念
- `set_token_filter_torch`: converts data into another representation / 将数据转换为另一种表示

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ctypes`, `typing`
- **Third-party / 第三方**: `torch`
