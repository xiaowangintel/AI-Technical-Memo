# layout_algorithm.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/ir/layout_algorithm.py`
- **EN:** Layout algebras
- **CN:** 模块文档说明：Layout algebras

## Line-by-Line Analysis / 逐行分析

### Lines 1-32 — File header

```python
#################################################################################################
#
# Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: BSD-3-Clause
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# 1. Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following disclaimer in the documentation
# and/or other materials provided with the distribution.
#
# 3. Neither the name of the copyright holder nor the names of its
# contributors may be used to endorse or promote products derived from
# this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
# CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
#################################################################################################
```
**EN:** Contains the file header, license notice, and opening comments for the module.
**CN:** 包含文件头、许可证声明以及模块开头的注释。

### Lines 33-35 — Docstring

```python
"""
Layout algebras
"""
```
**EN:** Docstring explains this scope: Layout algebras
**CN:** 文档字符串说明了该作用域的用途：Layout algebras

### Line 37 — From `pycute` import

```python
from pycute import Layout, composition, make_layout, flatten, product
```
**EN:** Imports `Layout, composition, make_layout, flatten, product` from `pycute` to reuse shared definitions at module scope.
**CN:** 从 `pycute` 导入 `Layout, composition, make_layout, flatten, product`，以便后续代码在模块级复用共享定义。

### Line 40 — Function `_infer_split`

```python
def _infer_split(old_shape, new_shape):
```
**EN:** Defines function `_infer_split` with parameters `old_shape, new_shape`.
**CN:** 定义函数 `_infer_split`，参数为 `old_shape, new_shape`。

#### Line 41 — Assign `old_shape`

```python
    old_shape = _tuple_to_list(old_shape)
```
**EN:** Assigns `old_shape` from `_tuple_to_list(old_shape)`, establishing state in function `_infer_split`.
**CN:** 将 `old_shape` 赋值为 `_tuple_to_list(old_shape)`，用于在函数 `_infer_split` 中建立状态。

#### Line 42 — Assign `new_shape`

```python
    new_shape = _tuple_to_list(new_shape)
```
**EN:** Assigns `new_shape` from `_tuple_to_list(new_shape)`, establishing state in function `_infer_split`.
**CN:** 将 `new_shape` 赋值为 `_tuple_to_list(new_shape)`，用于在函数 `_infer_split` 中建立状态。

#### Lines 43-44 — Conditional `len(old_shape) == 0 and len(new_shape) == 0`

```python
    if len(old_shape) == 0 and len(new_shape) == 0:
        return []
```
**EN:** Checks `len(old_shape) == 0 and len(new_shape) == 0` and selects the matching branch in function `_infer_split`.
**CN:** 检查 `len(old_shape) == 0 and len(new_shape) == 0`，并在函数 `_infer_split` 中选择匹配的分支。

#### Lines 45-49 — Conditional `len(old_shape) == 0`

```python
    if len(old_shape) == 0:
        if product(tuple(new_shape)) != 1:
            raise ValueError("Invalid reshape size")
        else:
            return new_shape
```
**EN:** Checks `len(old_shape) == 0` and selects the matching branch in function `_infer_split`.
**CN:** 检查 `len(old_shape) == 0`，并在函数 `_infer_split` 中选择匹配的分支。

#### Lines 50-54 — Conditional `len(new_shape) == 0`

```python
    if len(new_shape) == 0:
        if product(tuple(old_shape)) != 1:
            raise ValueError("Invalid reshape size")
        else:
            return old_shape
```
**EN:** Checks `len(new_shape) == 0` and selects the matching branch in function `_infer_split`.
**CN:** 检查 `len(new_shape) == 0`，并在函数 `_infer_split` 中选择匹配的分支。

#### Line 55 — Comment or spacing block

```python
    # This is done recursively by only process the last dimension at each time
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 56 — Assign `old_dim`

```python
    old_dim = old_shape[-1]
```
**EN:** Assigns `old_dim` from `old_shape[-1]`, establishing state in function `_infer_split`.
**CN:** 将 `old_dim` 赋值为 `old_shape[-1]`，用于在函数 `_infer_split` 中建立状态。

#### Line 57 — Assign `new_dim`

```python
    new_dim = new_shape[-1]
```
**EN:** Assigns `new_dim` from `new_shape[-1]`, establishing state in function `_infer_split`.
**CN:** 将 `new_dim` 赋值为 `new_shape[-1]`，用于在函数 `_infer_split` 中建立状态。

#### Line 58 — Comment or spacing block

```python
    # Exact match
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 59-60 — Conditional `old_dim == new_dim`

```python
    if old_dim == new_dim:
        return _infer_split(old_shape[:-1], new_shape[:-1]) + [new_dim,]
```
**EN:** Checks `old_dim == new_dim` and selects the matching branch in function `_infer_split`.
**CN:** 检查 `old_dim == new_dim`，并在函数 `_infer_split` 中选择匹配的分支。

#### Line 61 — Comment or spacing block

```python
    # Needs split
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 62-64 — Conditional `old_dim > new_dim and old_dim % new_dim == 0`

```python
    if old_dim > new_dim and old_dim % new_dim == 0:
        residual = old_dim // new_dim
        return _infer_split(old_shape[:-1] + [residual,], new_shape[:-1]) + [new_dim,]
```
**EN:** Checks `old_dim > new_dim and old_dim % new_dim == 0` and selects the matching branch in function `_infer_split`.
**CN:** 检查 `old_dim > new_dim and old_dim % new_dim == 0`，并在函数 `_infer_split` 中选择匹配的分支。

#### Line 65 — Comment or spacing block

```python
    # Needs merge
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 66-68 — Conditional `old_dim < new_dim and new_dim % old_dim == 0`

```python
    if old_dim < new_dim and new_dim % old_dim == 0:
        residual = new_dim // old_dim
        return _infer_split(old_shape[:-1], new_shape[:-1] + [residual,]) + [old_dim,]
```
**EN:** Checks `old_dim < new_dim and new_dim % old_dim == 0` and selects the matching branch in function `_infer_split`.
**CN:** 检查 `old_dim < new_dim and new_dim % old_dim == 0`，并在函数 `_infer_split` 中选择匹配的分支。

#### Line 70 — Raise exception

```python
    raise NotImplementedError(f"Unsupported split: {old_shape} -> {new_shape}")
```
**EN:** Raises `NotImplementedError(f'Unsupported split: {old_shape} -> {new_shape}')` to signal an error or unsupported state.
**CN:** 抛出 `NotImplementedError(f'Unsupported split: {old_shape} -> {new_shape}')`，用于报告错误或不支持的状态。

### Line 72 — Function `_infer_merge`

```python
def _infer_merge(flatten_shape, shape):
```
**EN:** Defines function `_infer_merge` with parameters `flatten_shape, shape`.
**CN:** 定义函数 `_infer_merge`，参数为 `flatten_shape, shape`。

#### Line 73 — Assign `flatten_shape`

```python
    flatten_shape = _tuple_to_list(flatten_shape)
```
**EN:** Assigns `flatten_shape` from `_tuple_to_list(flatten_shape)`, establishing state in function `_infer_merge`.
**CN:** 将 `flatten_shape` 赋值为 `_tuple_to_list(flatten_shape)`，用于在函数 `_infer_merge` 中建立状态。

#### Line 74 — Assign `shape`

```python
    shape = _tuple_to_list(shape)
```
**EN:** Assigns `shape` from `_tuple_to_list(shape)`, establishing state in function `_infer_merge`.
**CN:** 将 `shape` 赋值为 `_tuple_to_list(shape)`，用于在函数 `_infer_merge` 中建立状态。

#### Line 75 — Assign `idx_flat`

```python
    idx_flat = 0
```
**EN:** Assigns `idx_flat` from `0`, establishing state in function `_infer_merge`.
**CN:** 将 `idx_flat` 赋值为 `0`，用于在函数 `_infer_merge` 中建立状态。

#### Line 76 — Assign `merged_shape`

```python
    merged_shape = []
```
**EN:** Assigns `merged_shape` from `[]`, establishing state in function `_infer_merge`.
**CN:** 将 `merged_shape` 赋值为 `[]`，用于在函数 `_infer_merge` 中建立状态。

#### Lines 77-92 — Loop over `shape`

```python
    for dim in shape:
        # Exact match
        if dim == flatten_shape[idx_flat]:
            merged_shape.append(dim)
            idx_flat += 1
        # Need group
        elif dim > flatten_shape[idx_flat] and dim % flatten_shape[idx_flat] == 0:
            residual = dim
            group = []
            while(residual > 1):
                group.append(flatten_shape[idx_flat])
                residual = residual // flatten_shape[idx_flat]
                idx_flat += 1
            merged_shape.append(group)
        else:
            raise NotImplementedError(f"Unsupported merge: {flatten_shape} -> {shape}")
```
**EN:** Iterates `dim` over `shape` to repeat a processing step.
**CN:** 让 `dim` 遍历 `shape`，从而重复执行处理步骤。

#### Line 94 — Return

```python
    return merged_shape
```
**EN:** Returns `merged_shape` to the caller.
**CN:** 向调用方返回 `merged_shape`。

### Line 96 — Function `_list_to_tuple`

```python
def _list_to_tuple(nested_list):
```
**EN:** Defines function `_list_to_tuple` with parameters `nested_list`.
**CN:** 定义函数 `_list_to_tuple`，参数为 `nested_list`。

#### Lines 97-98 — Conditional `isinstance(nested_list, list) or isinstance(nested_list, tuple)`

```python
    if isinstance(nested_list, list) or isinstance(nested_list, tuple):
        return tuple(_list_to_tuple(item) for item in nested_list)
```
**EN:** Checks `isinstance(nested_list, list) or isinstance(nested_list, tuple)` and selects the matching branch in function `_list_to_tuple`.
**CN:** 检查 `isinstance(nested_list, list) or isinstance(nested_list, tuple)`，并在函数 `_list_to_tuple` 中选择匹配的分支。

#### Line 99 — Return

```python
    return nested_list
```
**EN:** Returns `nested_list` to the caller.
**CN:** 向调用方返回 `nested_list`。

### Line 101 — Function `_tuple_to_list`

```python
def _tuple_to_list(nested_tuple):
```
**EN:** Defines function `_tuple_to_list` with parameters `nested_tuple`.
**CN:** 定义函数 `_tuple_to_list`，参数为 `nested_tuple`。

#### Lines 102-103 — Conditional `isinstance(nested_tuple, list) or isinstance(nested_tuple, tuple)`

```python
    if isinstance(nested_tuple, list) or isinstance(nested_tuple, tuple):
        return list(_tuple_to_list(item) for item in nested_tuple)
```
**EN:** Checks `isinstance(nested_tuple, list) or isinstance(nested_tuple, tuple)` and selects the matching branch in function `_tuple_to_list`.
**CN:** 检查 `isinstance(nested_tuple, list) or isinstance(nested_tuple, tuple)`，并在函数 `_tuple_to_list` 中选择匹配的分支。

#### Line 104 — Return

```python
    return nested_tuple
```
**EN:** Returns `nested_tuple` to the caller.
**CN:** 向调用方返回 `nested_tuple`。

### Line 106 — Function `_reverse_tuple`

```python
def _reverse_tuple(nested_tuple: tuple):
```
**EN:** Defines function `_reverse_tuple` with parameters `nested_tuple`.
**CN:** 定义函数 `_reverse_tuple`，参数为 `nested_tuple`。

#### Lines 107-108 — Conditional `isinstance(nested_tuple, tuple)`

```python
    if isinstance(nested_tuple, tuple):
        return tuple([_reverse_tuple(item) for item in nested_tuple][::-1])
```
**EN:** Checks `isinstance(nested_tuple, tuple)` and selects the matching branch in function `_reverse_tuple`.
**CN:** 检查 `isinstance(nested_tuple, tuple)`，并在函数 `_reverse_tuple` 中选择匹配的分支。

#### Line 109 — Return

```python
    return nested_tuple
```
**EN:** Returns `nested_tuple` to the caller.
**CN:** 向调用方返回 `nested_tuple`。

### Line 111 — Function `_get_first_lhs_nonzero_stride`

```python
def _get_first_lhs_nonzero_stride(stride_list, idx):
```
**EN:** Defines function `_get_first_lhs_nonzero_stride` with parameters `stride_list, idx`.
**CN:** 定义函数 `_get_first_lhs_nonzero_stride`，参数为 `stride_list, idx`。

#### Lines 112-116 — Loop over `reversed(range(idx))`

```python
    for i in reversed(range(idx)):
        if stride_list[i] != 0:
            return i
    else:
        return None
```
**EN:** Iterates `i` over `reversed(range(idx))` to repeat a processing step.
**CN:** 让 `i` 遍历 `reversed(range(idx))`，从而重复执行处理步骤。

### Line 118 — Function `_get_first_rhs_nonzero_stride`

```python
def _get_first_rhs_nonzero_stride(stride_list, idx):
```
**EN:** Defines function `_get_first_rhs_nonzero_stride` with parameters `stride_list, idx`.
**CN:** 定义函数 `_get_first_rhs_nonzero_stride`，参数为 `stride_list, idx`。

#### Lines 119-123 — Loop over `range(idx + 1, len(stride_list))`

```python
    for i in range(idx+1, len(stride_list)):
        if stride_list[i] != 0:
            return i
        else:
            return None
```
**EN:** Iterates `i` over `range(idx + 1, len(stride_list))` to repeat a processing step.
**CN:** 让 `i` 遍历 `range(idx + 1, len(stride_list))`，从而重复执行处理步骤。

### Lines 125-131 — Function `reshape`

```python
def reshape(layout, new_shape):
    """
    General reshape of input layout.
    It takes two steps:
    1. split the dimensions of the old layout
    2. merge the splitted dimensions according to the new shape
    """
```
**EN:** Defines function `reshape` with parameters `layout, new_shape`. Purpose: General reshape of input layout.
**CN:** 定义函数 `reshape`，参数为 `layout, new_shape`。 其用途：General reshape of input layout.

#### Lines 132-135 — Comment or spacing block

```python
    #
    # Step 1: Split the dimensions of the old layout
    #
    # 1.1 Flat old and new shape
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 136 — Assign `old_flatten_shape`

```python
    old_flatten_shape = list(flatten(layout.shape))
```
**EN:** Assigns `old_flatten_shape` from `list(flatten(layout.shape))`, establishing state in function `reshape`.
**CN:** 将 `old_flatten_shape` 赋值为 `list(flatten(layout.shape))`，用于在函数 `reshape` 中建立状态。

#### Line 137 — Assign `new_flatten_shape`

```python
    new_flatten_shape = list(flatten(new_shape))
```
**EN:** Assigns `new_flatten_shape` from `list(flatten(new_shape))`, establishing state in function `reshape`.
**CN:** 将 `new_flatten_shape` 赋值为 `list(flatten(new_shape))`，用于在函数 `reshape` 中建立状态。

#### Line 139 — Comment or spacing block

```python
    # 1.2 Infer the flatten splitted shape
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 140 — Assign `splitted_flatten_shape`

```python
    splitted_flatten_shape = _infer_split(old_flatten_shape, new_flatten_shape)
```
**EN:** Assigns `splitted_flatten_shape` from `_infer_split(old_flatten_shape, new_flatten_shape)`, establishing state in function `reshape`.
**CN:** 将 `splitted_flatten_shape` 赋值为 `_infer_split(old_flatten_shape, new_flatten_shape)`，用于在函数 `reshape` 中建立状态。

#### Line 142 — Comment or spacing block

```python
    # 1.3 Unflat the splitted shape based on the old shape
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 143 — Assign `splited_shape`

```python
    splited_shape = _infer_merge(splitted_flatten_shape, old_flatten_shape)
```
**EN:** Assigns `splited_shape` from `_infer_merge(splitted_flatten_shape, old_flatten_shape)`, establishing state in function `reshape`.
**CN:** 将 `splited_shape` 赋值为 `_infer_merge(splitted_flatten_shape, old_flatten_shape)`，用于在函数 `reshape` 中建立状态。

#### Lines 145-147 — Comment or spacing block

```python
    # 1.4 Infer the type of each split
    # If the split type is in row-major (R), the dimension list is reversed because
    # the cute::composition only support column-major split
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 148 — Assign `split_type`

```python
    split_type = []  # the type of each split (ColumnMajor or RowMajor)
```
**EN:** Assigns `split_type` from `[]`, establishing state in function `reshape`.
**CN:** 将 `split_type` 赋值为 `[]`，用于在函数 `reshape` 中建立状态。

#### Line 149 — Assign `permuted_splitted_shape`

```python
    permuted_splitted_shape = []
```
**EN:** Assigns `permuted_splitted_shape` from `[]`, establishing state in function `reshape`.
**CN:** 将 `permuted_splitted_shape` 赋值为 `[]`，用于在函数 `reshape` 中建立状态。

#### Line 150 — Assign `old_flatten_stride`

```python
    old_flatten_stride = list(flatten(layout.stride))
```
**EN:** Assigns `old_flatten_stride` from `list(flatten(layout.stride))`, establishing state in function `reshape`.
**CN:** 将 `old_flatten_stride` 赋值为 `list(flatten(layout.stride))`，用于在函数 `reshape` 中建立状态。

#### Lines 151-207 — Loop over `enumerate(splited_shape)`

```python
    for idx, dim in enumerate(splited_shape):
        if not isinstance(dim, list):
            permuted_splitted_shape.append(dim)
            split_type.append("C")
        else:
            lhs_stride = _get_first_lhs_nonzero_stride(old_flatten_stride, idx)
            rhs_stride = _get_first_rhs_nonzero_stride(old_flatten_stride, idx)
            # Special case for single tuple
            # Use column-major by default
            if lhs_stride is None and rhs_stride is None:
                permuted_splitted_shape.append(dim)
                split_type.append("C")
            else:
                if lhs_stride is not None and rhs_stride is not None:
                    # We consider shape[idx]:stride[idx]
                    # Case 1: stride[idx - 1] <= stride[idx] <= stride[idx + 1]: column major
                    if lhs_stride <= old_flatten_stride[idx] and old_flatten_stride[idx] <= rhs_stride:
                        permuted_splitted_shape.append(dim)
                        split_type.append("C")
                    # Case 2: stride[idx - 1] > stride[idx] > stride[idx + 1]: row major
                    elif lhs_stride > old_flatten_stride[idx] and old_flatten_stride[idx] > rhs_stride:
                        permuted_splitted_shape.append([d for d in reversed(dim)])
                        split_type.append("R")
                    # Case 3: stride[idx - 1] <= stride[idx] > stride[idx + 1]: concave
                    elif lhs_stride <= old_flatten_stride[idx] and old_flatten_stride[idx] > rhs_stride:
                        if lhs_stride >= rhs_stride:
                            permuted_splitted_shape.append(dim)
                            split_type.append("C")
                        else:
                            permuted_splitted_shape.append([d for d in reversed(dim)])
                            split_type.append("R")
                    # Case 4: stride[idx - 1] > stride[idx] <= stride[idx + 1]: concave
                    elif lhs_stride > old_flatten_stride[idx] and old_flatten_stride[idx] <= rhs_stride:
                        if lhs_stride >= rhs_stride:
                            permuted_splitted_shape.append(dim)
                            split_type.append("C")
                        else:
                            permuted_splitted_shape.append([d for d in reversed(dim)])
                            split_type.append("R")
                    else:
                        raise NotImplementedError()
                elif lhs_stride is None:
                    # Case 1: dim's stride < dim+1's stride, expand in column major
                    if old_flatten_stride[idx] > rhs_stride:
                        permuted_splitted_shape.append([d for d in reversed(dim)])
                        split_type.append("R")
                    else:
                        permuted_splitted_shape.append(dim)
                        split_type.append("C")
                else:
                    # Case 1: dim's stride > dim-1's stride
                    if old_flatten_stride[idx] < lhs_stride:
                        permuted_splitted_shape.append([d for d in reversed(dim)])
                        split_type.append("R")
                    else:
                        permuted_splitted_shape.append(dim)
                        split_type.append("C")
```
**EN:** Iterates `(idx, dim)` over `enumerate(splited_shape)` to repeat a processing step.
**CN:** 让 `(idx, dim)` 遍历 `enumerate(splited_shape)`，从而重复执行处理步骤。

#### Line 209 — Comment or spacing block

```python
    # 1.4 Generate the splitted layout
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 210 — Assign `permuted_splitted_layout`

```python
    permuted_splitted_layout = composition(layout, Layout(_list_to_tuple(permuted_splitted_shape)))
```
**EN:** Assigns `permuted_splitted_layout` from `composition(layout, Layout(_list_to_tuple(permuted_splitted_shape)))`, establishing state in function `reshape`.
**CN:** 将 `permuted_splitted_layout` 赋值为 `composition(layout, Layout(_list_to_tuple(permuted_splitted_shape)))`，用于在函数 `reshape` 中建立状态。

#### Line 212 — Comment or spacing block

```python
    # 1.5 Reverse the permutation in 1.4 before merge
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 213 — Assign `splitted_shape`

```python
    splitted_shape = []
```
**EN:** Assigns `splitted_shape` from `[]`, establishing state in function `reshape`.
**CN:** 将 `splitted_shape` 赋值为 `[]`，用于在函数 `reshape` 中建立状态。

#### Line 214 — Assign `splitted_stride`

```python
    splitted_stride = []
```
**EN:** Assigns `splitted_stride` from `[]`, establishing state in function `reshape`.
**CN:** 将 `splitted_stride` 赋值为 `[]`，用于在函数 `reshape` 中建立状态。

#### Lines 215-224 — Loop over `zip(permuted_splitted_layout.shape, permuted_splitted_layout.stride, split_type)`

```python
    for shape_dim, stride_dim, type in zip(
            permuted_splitted_layout.shape,
            permuted_splitted_layout.stride,
            split_type):
        if type == "C":
            splitted_shape.append(shape_dim)
            splitted_stride.append(stride_dim)
        else:
            splitted_shape.append(tuple([d for d in reversed(shape_dim)]))
            splitted_stride.append(tuple([d for d in reversed(stride_dim)]))
```
**EN:** Iterates `(shape_dim, stride_dim, type)` over `zip(permuted_splitted_layout.shape, permuted_splitted_layout.stride, split_type)` to repeat a processing step.
**CN:** 让 `(shape_dim, stride_dim, type)` 遍历 `zip(permuted_splitted_layout.shape, permuted_splitted_layout.stride, split_type)`，从而重复执行处理步骤。

#### Line 225 — Assign `splitted_layout`

```python
    splitted_layout = Layout(tuple(splitted_shape), tuple(splitted_stride))
```
**EN:** Assigns `splitted_layout` from `Layout(tuple(splitted_shape), tuple(splitted_stride))`, establishing state in function `reshape`.
**CN:** 将 `splitted_layout` 赋值为 `Layout(tuple(splitted_shape), tuple(splitted_stride))`，用于在函数 `reshape` 中建立状态。

#### Lines 228-231 — Comment or spacing block

```python
    #
    # Step 2: Merge the splitted dimensions according to the new shape
    #
    # 2.1 Merge layout
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 232 — Assign `merged_layout`

```python
    merged_layout = composition(splitted_layout, Layout(new_shape))
```
**EN:** Assigns `merged_layout` from `composition(splitted_layout, Layout(new_shape))`, establishing state in function `reshape`.
**CN:** 将 `merged_layout` 赋值为 `composition(splitted_layout, Layout(new_shape))`，用于在函数 `reshape` 中建立状态。

#### Line 234 — Comment or spacing block

```python
    # 2.2 Cleaning up
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 235 — Assign `output_layout`

```python
    output_layout = composition(merged_layout, Layout(new_shape))
```
**EN:** Assigns `output_layout` from `composition(merged_layout, Layout(new_shape))`, establishing state in function `reshape`.
**CN:** 将 `output_layout` 赋值为 `composition(merged_layout, Layout(new_shape))`，用于在函数 `reshape` 中建立状态。

#### Line 236 — Return

```python
    return output_layout
```
**EN:** Returns `output_layout` to the caller.
**CN:** 向调用方返回 `output_layout`。

### Lines 239-242 — Function `permutation`

```python
def permutation(layout, permutation):
    """
    Permute the layout
    """
```
**EN:** Defines function `permutation` with parameters `layout, permutation`. Purpose: Permute the layout
**CN:** 定义函数 `permutation`，参数为 `layout, permutation`。 其用途：Permute the layout

#### Line 243 — Assign `new_shape`

```python
    new_shape = tuple([layout.shape[idx] for idx in permutation])
```
**EN:** Assigns `new_shape` from `tuple([layout.shape[idx] for idx in permutation])`, establishing state in function `permutation`.
**CN:** 将 `new_shape` 赋值为 `tuple([layout.shape[idx] for idx in permutation])`，用于在函数 `permutation` 中建立状态。

#### Line 244 — Assign `new_stride`

```python
    new_stride = tuple([layout.stride[idx] for idx in permutation])
```
**EN:** Assigns `new_stride` from `tuple([layout.stride[idx] for idx in permutation])`, establishing state in function `permutation`.
**CN:** 将 `new_stride` 赋值为 `tuple([layout.stride[idx] for idx in permutation])`，用于在函数 `permutation` 中建立状态。

#### Line 245 — Return

```python
    return Layout(new_shape, new_stride)
```
**EN:** Returns `Layout(new_shape, new_stride)` to the caller.
**CN:** 向调用方返回 `Layout(new_shape, new_stride)`。

### Line 248 — Function `_broadcast`

```python
def _broadcast(layout, new_shape):
```
**EN:** Defines function `_broadcast` with parameters `layout, new_shape`.
**CN:** 定义函数 `_broadcast`，参数为 `layout, new_shape`。

#### Lines 249-258 — Conditional `len(layout) == 1 and isinstance(new_shape, int)`

```python
    if len(layout) == 1 and isinstance(new_shape, int):
        old_dim = layout.shape
        old_stride = layout.stride
        new_dim = new_shape
        if old_dim == new_dim:
            return Layout(old_dim, old_stride)
        elif old_dim == 1:
            return Layout(new_dim, 0)
        else:
            raise NotImplementedError(f"Invalid Broadcast: {old_dim} -> {new_dim}")
```
**EN:** Checks `len(layout) == 1 and isinstance(new_shape, int)` and selects the matching branch in function `_broadcast`.
**CN:** 检查 `len(layout) == 1 and isinstance(new_shape, int)`，并在函数 `_broadcast` 中选择匹配的分支。

#### Line 260 — Comment or spacing block

```python
    # Align the dimensions
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 261 — Assign `old_shape`

```python
    old_shape = layout.shape
```
**EN:** Assigns `old_shape` from `layout.shape`, establishing state in function `_broadcast`.
**CN:** 将 `old_shape` 赋值为 `layout.shape`，用于在函数 `_broadcast` 中建立状态。

#### Lines 262-266 — Conditional `isinstance(old_shape, int)`

```python
    if isinstance(old_shape, int):
        old_shape = (old_shape,)
        sub_layouts = [layout,]
    else:
        sub_layouts = [sub_layout for sub_layout in layout]
```
**EN:** Checks `isinstance(old_shape, int)` and selects the matching branch in function `_broadcast`.
**CN:** 检查 `isinstance(old_shape, int)`，并在函数 `_broadcast` 中选择匹配的分支。

#### Line 267 — Assign `rhs_broadcast_layouts`

```python
    rhs_broadcast_layouts = [Layout(1, 0)] * (len(new_shape) - len(old_shape))
```
**EN:** Assigns `rhs_broadcast_layouts` from `[Layout(1, 0)] * (len(new_shape) - len(old_shape))`, establishing state in function `_broadcast`.
**CN:** 将 `rhs_broadcast_layouts` 赋值为 `[Layout(1, 0)] * (len(new_shape) - len(old_shape))`，用于在函数 `_broadcast` 中建立状态。

#### Line 268 — Comment or spacing block

```python
    # Get the broadcasted layout
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 269 — Assign `broadcast_layouts`

```python
    broadcast_layouts = []
```
**EN:** Assigns `broadcast_layouts` from `[]`, establishing state in function `_broadcast`.
**CN:** 将 `broadcast_layouts` 赋值为 `[]`，用于在函数 `_broadcast` 中建立状态。

#### Lines 270-278 — Exception handling

```python
    try:
        layout = make_layout(*sub_layouts, *rhs_broadcast_layouts)
        broadcast_layouts = []
        for idx, sub_layout in enumerate(layout):
            broadcast_layouts.append(_broadcast(sub_layout, new_shape[idx]))
    except NotImplementedError:
        layout = make_layout(*rhs_broadcast_layouts, *sub_layouts)
        for idx, sub_layout in enumerate(layout):
            broadcast_layouts.append(_broadcast(sub_layout, new_shape[idx]))
```
**EN:** Wraps the enclosed logic with exception handling and optional cleanup paths.
**CN:** 使用异常处理包装该逻辑，并提供回退或清理路径。

#### Line 279 — Return

```python
    return make_layout(*broadcast_layouts)
```
**EN:** Returns `make_layout(*broadcast_layouts)` to the caller.
**CN:** 向调用方返回 `make_layout(*broadcast_layouts)`。

### Lines 282-287 — Function `broadcast`

```python
def broadcast(layout, new_shape):
    """
    Broadcast the new layout based on the input shape
    The broadcasted shape equals to the new shape
    The stride of broadcasted dimensions are 0
    """
```
**EN:** Defines function `broadcast` with parameters `layout, new_shape`. Purpose: Broadcast the new layout based on the input shape
**CN:** 定义函数 `broadcast`，参数为 `layout, new_shape`。 其用途：Broadcast the new layout based on the input shape

#### Line 288 — Return

```python
    return _broadcast(layout, new_shape)
```
**EN:** Returns `_broadcast(layout, new_shape)` to the caller.
**CN:** 向调用方返回 `_broadcast(layout, new_shape)`。

### Lines 291-294 — Function `debroadcast`

```python
def debroadcast(layout, dims):
    """
    Squeeze the 0-stride
    """
```
**EN:** Defines function `debroadcast` with parameters `layout, dims`. Purpose: Squeeze the 0-stride
**CN:** 定义函数 `debroadcast`，参数为 `layout, dims`。 其用途：Squeeze the 0-stride

#### Lines 295-297 — Loop over `dims`

```python
    for dim in dims:
        if layout.stride[dim] != 0:
            raise ValueError(f"Dim{dim} cannot be debroadcasted as it has stride {layout.stride[dim]}")
```
**EN:** Iterates `dim` over `dims` to repeat a processing step.
**CN:** 让 `dim` 遍历 `dims`，从而重复执行处理步骤。

#### Line 298 — Assign `new_shape`

```python
    new_shape = tuple([s for idx, s in enumerate(layout.shape) if idx not in dims])
```
**EN:** Assigns `new_shape` from `tuple([s for idx, s in enumerate(layout.shape) if idx not in dims])`, establishing state in function `debroadcast`.
**CN:** 将 `new_shape` 赋值为 `tuple([s for idx, s in enumerate(layout.shape) if idx not in dims])`，用于在函数 `debroadcast` 中建立状态。

#### Line 299 — Assign `new_stride`

```python
    new_stride = tuple([s for idx, s in enumerate(layout.stride) if idx not in dims])
```
**EN:** Assigns `new_stride` from `tuple([s for idx, s in enumerate(layout.stride) if idx not in dims])`, establishing state in function `debroadcast`.
**CN:** 将 `new_stride` 赋值为 `tuple([s for idx, s in enumerate(layout.stride) if idx not in dims])`，用于在函数 `debroadcast` 中建立状态。

#### Line 300 — Return

```python
    return Layout(new_shape, new_stride)
```
**EN:** Returns `Layout(new_shape, new_stride)` to the caller.
**CN:** 向调用方返回 `Layout(new_shape, new_stride)`。

### Line 303 — Function `canonicalization_`

```python
def canonicalization_(shapes, strides):
```
**EN:** Defines function `canonicalization_` with parameters `shapes, strides`.
**CN:** 定义函数 `canonicalization_`，参数为 `shapes, strides`。

#### Lines 304-316 — Conditional `isinstance(shapes, tuple)`

```python
    if isinstance(shapes, tuple):
        c_shapes = []
        c_strides = []
        for shape, stride in zip(shapes, strides):
            c_shape, c_stride = canonicalization_(shape, stride)
            c_shapes.append(c_shape)
            c_strides.append(c_stride)
        return tuple(c_shapes), tuple(c_strides)
    else:
        if shapes == 1:
            return 1, 0
        else:
            return shapes, strides
```
**EN:** Checks `isinstance(shapes, tuple)` and selects the matching branch in function `canonicalization_`.
**CN:** 检查 `isinstance(shapes, tuple)`，并在函数 `canonicalization_` 中选择匹配的分支。

### Lines 318-322 — Function `canonicalization`

```python
def canonicalization(layout):
    """
    Canonicalize the input layout
    1. set the stride of shape "1" to 0
    """
```
**EN:** Defines function `canonicalization` with parameters `layout`. Purpose: Canonicalize the input layout
**CN:** 定义函数 `canonicalization`，参数为 `layout`。 其用途：Canonicalize the input layout

#### Line 323 — Assign `new_shape, new_stride`

```python
    new_shape, new_stride = canonicalization_(layout.shape, layout.stride)
```
**EN:** Assigns `new_shape, new_stride` from `canonicalization_(layout.shape, layout.stride)`, establishing state in function `canonicalization`.
**CN:** 将 `new_shape, new_stride` 赋值为 `canonicalization_(layout.shape, layout.stride)`，用于在函数 `canonicalization` 中建立状态。

#### Line 324 — Return

```python
    return Layout(new_shape, new_stride)
```
**EN:** Returns `Layout(new_shape, new_stride)` to the caller.
**CN:** 向调用方返回 `Layout(new_shape, new_stride)`。

## Key Concepts / 关键概念

- **EN:** Top-level classes: none.
- **CN:** 顶层类：无。
- **EN:** Top-level functions: `_infer_split`, `_infer_merge`, `_list_to_tuple`, `_tuple_to_list`, `_reverse_tuple`, `_get_first_lhs_nonzero_stride`, `_get_first_rhs_nonzero_stride`, `reshape`, `permutation`, `_broadcast`, `broadcast`, `debroadcast`, `canonicalization_`, `canonicalization`.
- **CN:** 顶层函数：`_infer_split`, `_infer_merge`, `_list_to_tuple`, `_tuple_to_list`, `_reverse_tuple`, `_get_first_lhs_nonzero_stride`, `_get_first_rhs_nonzero_stride`, `reshape`, `permutation`, `_broadcast`, `broadcast`, `debroadcast`, `canonicalization_`, `canonicalization`。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** none / 无
- **Standard & third-party / 标准库与第三方:** `pycute`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
