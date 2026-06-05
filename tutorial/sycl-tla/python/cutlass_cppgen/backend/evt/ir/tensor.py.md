# tensor.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/ir/tensor.py`
- **EN:** High-level class for tensor
- **CN:** 模块文档说明：High-level class for tensor

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
High-level class for tensor
"""
```
**EN:** Docstring explains this scope: High-level class for tensor
**CN:** 文档字符串说明了该作用域的用途：High-level class for tensor

### Line 37 — From `cutlass_library` import

```python
from cutlass_library import LayoutType
```
**EN:** Imports `LayoutType` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `LayoutType`，以便后续代码在模块级复用共享定义。

### Lines 39-46 — From `cutlass_cppgen.backend.evt.ir.layout_algorithm` import

```python
from cutlass_cppgen.backend.evt.ir.layout_algorithm import (
    Layout,
    broadcast,
    canonicalization,
    permutation,
    reshape,
    _reverse_tuple
)
```
**EN:** Imports `Layout, broadcast, canonicalization, permutation, reshape, _reverse_tuple` from `cutlass_cppgen.backend.evt.ir.layout_algorithm` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir.layout_algorithm` 导入 `Layout, broadcast, canonicalization, permutation, reshape, _reverse_tuple`，以便后续代码在模块级复用共享定义。

### Line 47 — From `cutlass_cppgen.utils.datatypes` import

```python
from cutlass_cppgen.utils.datatypes import get_datatype_and_layout, get_tensor_shape, library_type
```
**EN:** Imports `get_datatype_and_layout, get_tensor_shape, library_type` from `cutlass_cppgen.utils.datatypes` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.utils.datatypes` 导入 `get_datatype_and_layout, get_tensor_shape, library_type`，以便后续代码在模块级复用共享定义。

### Lines 50-53 — Class `Tensor`

```python
class Tensor:
    """
    The tensor abstracts the data type
    """
```
**EN:** Declares class `Tensor` deriving from `object`. Purpose: The tensor abstracts the data type
**CN:** 声明类 `Tensor`，其基类为 `object`。 其用途：The tensor abstracts the data type

#### Line 54 — Function `__init__`

```python
    def __init__(self, tensor=None, element=None, shape=None, stride=None,layout_tag=None, is_constant=False) -> None:
```
**EN:** Defines function `__init__` with parameters `self, tensor, element, shape, stride, layout_tag, is_constant`.
**CN:** 定义函数 `__init__`，参数为 `self, tensor, element, shape, stride, layout_tag, is_constant`。

##### Lines 55-66 — Conditional `element is not None and tensor is not None`

```python
        if element is not None and tensor is not None:
            raise Exception(f"Must not specify both element and tensor")
        elif shape is not None and tensor is not None:
            raise Exception(f"Must not specify both shape and tensor")
        elif layout_tag is not None and tensor is not None:
            raise Exception(f"Must not specify both layout_tag and tensor")
        elif (element is None or (layout_tag is None and stride is None) or shape is None) and (tensor is None) :
            raise Exception(f"Must specify one of (element, shape, layout/stride) or (tensor)")
        elif stride is not None and tensor is not None:
            raise Exception(f"Must not specify both stride and tensor")
        elif stride is not None and layout_tag is not None:
            raise Exception(f"Must not specify layout_tag when stride is provided")
```
**EN:** Checks `element is not None and tensor is not None` and selects the matching branch in function `__init__`.
**CN:** 检查 `element is not None and tensor is not None`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 68-89 — Conditional `isinstance(tensor, Tensor)`

```python
        if isinstance(tensor, Tensor):
            # Directly copy all the attributes
            self.__dict__.update(vars(tensor))
        else:
            if tensor is None:
                self.element = library_type(element)
            else:
                self.element, layout_tag = get_datatype_and_layout(tensor)
                shape = get_tensor_shape(tensor)
            if stride is not None:
                self.layout = Layout(shape[::-1], stride[::-1])
            else:
                if layout_tag == LayoutType.RowMajor:
                    self.layout = Layout(shape[::-1])
                elif layout_tag == LayoutType.ColumnMajor:
                    self.layout = permutation(Layout(shape), [idx for idx in reversed(range(len(shape)))])
            self.layout = canonicalization(self.layout)

            self.is_constant = is_constant
            # Save the tensor value if it is constant
            if is_constant and tensor is not None:
                self.value = tensor
```
**EN:** Checks `isinstance(tensor, Tensor)` and selects the matching branch in function `__init__`.
**CN:** 检查 `isinstance(tensor, Tensor)`，并在函数 `__init__` 中选择匹配的分支。

#### Lines 91-95 — Function `shape`

```python
    @property
    def shape(self):
        """
        Returns the RowMajor layout shape
        """
```
**EN:** Defines function `shape` with parameters `self`. Purpose: Returns the RowMajor layout shape
**CN:** 定义函数 `shape`，参数为 `self`。 其用途：Returns the RowMajor layout shape

##### Line 96 — Return

```python
        return _reverse_tuple(self.layout.shape)
```
**EN:** Returns `_reverse_tuple(self.layout.shape)` to the caller.
**CN:** 向调用方返回 `_reverse_tuple(self.layout.shape)`。

#### Lines 98-102 — Function `stride`

```python
    @property
    def stride(self):
        """
        Returns the RowMajor layout stride
        """
```
**EN:** Defines function `stride` with parameters `self`. Purpose: Returns the RowMajor layout stride
**CN:** 定义函数 `stride`，参数为 `self`。 其用途：Returns the RowMajor layout stride

##### Line 103 — Return

```python
        return _reverse_tuple(self.layout.stride)
```
**EN:** Returns `_reverse_tuple(self.layout.stride)` to the caller.
**CN:** 向调用方返回 `_reverse_tuple(self.layout.stride)`。

#### Lines 105-109 — Function `rank`

```python
    @property
    def rank(self):
        """
        Returns the rank of the tensor
        """
```
**EN:** Defines function `rank` with parameters `self`. Purpose: Returns the rank of the tensor
**CN:** 定义函数 `rank`，参数为 `self`。 其用途：Returns the rank of the tensor

##### Line 110 — Return

```python
        return len(self.shape)
```
**EN:** Returns `len(self.shape)` to the caller.
**CN:** 向调用方返回 `len(self.shape)`。

#### Lines 112-114 — Comment or spacing block

```python
    #
    # Layout Algorithms
    #
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 116-119 — Function `broadcast`

```python
    def broadcast(self, shape):
        """
        Broadcast self.layout to shape
        """
```
**EN:** Defines function `broadcast` with parameters `self, shape`. Purpose: Broadcast self.layout to shape
**CN:** 定义函数 `broadcast`，参数为 `self, shape`。 其用途：Broadcast self.layout to shape

##### Line 120 — Assertion

```python
        assert isinstance(shape, tuple)
```
**EN:** Asserts `isinstance(shape, tuple)` to enforce an expected condition.
**CN:** 断言 `isinstance(shape, tuple)`，用于保证预期条件成立。

##### Line 121 — Assign `self.layout`

```python
        self.layout = broadcast(self.layout, _reverse_tuple(shape))
```
**EN:** Assigns `self.layout` from `broadcast(self.layout, _reverse_tuple(shape))`, establishing state in function `broadcast`.
**CN:** 将 `self.layout` 赋值为 `broadcast(self.layout, _reverse_tuple(shape))`，用于在函数 `broadcast` 中建立状态。

#### Lines 123-126 — Function `reshape`

```python
    def reshape(self, shape):
        """
        Reshape self.layout to shape
        """
```
**EN:** Defines function `reshape` with parameters `self, shape`. Purpose: Reshape self.layout to shape
**CN:** 定义函数 `reshape`，参数为 `self, shape`。 其用途：Reshape self.layout to shape

##### Line 127 — Assertion

```python
        assert isinstance(shape, tuple)
```
**EN:** Asserts `isinstance(shape, tuple)` to enforce an expected condition.
**CN:** 断言 `isinstance(shape, tuple)`，用于保证预期条件成立。

##### Line 128 — Assign `reverse_shape`

```python
        reverse_shape = _reverse_tuple(shape)
```
**EN:** Assigns `reverse_shape` from `_reverse_tuple(shape)`, establishing state in function `reshape`.
**CN:** 将 `reverse_shape` 赋值为 `_reverse_tuple(shape)`，用于在函数 `reshape` 中建立状态。

##### Line 129 — Assign `self.layout`

```python
        self.layout = reshape(self.layout, reverse_shape)
```
**EN:** Assigns `self.layout` from `reshape(self.layout, reverse_shape)`, establishing state in function `reshape`.
**CN:** 将 `self.layout` 赋值为 `reshape(self.layout, reverse_shape)`，用于在函数 `reshape` 中建立状态。

#### Lines 131-134 — Function `permute`

```python
    def permute(self, indices):
        """
        Permute self.layout according to indices
        """
```
**EN:** Defines function `permute` with parameters `self, indices`. Purpose: Permute self.layout according to indices
**CN:** 定义函数 `permute`，参数为 `self, indices`。 其用途：Permute self.layout according to indices

##### Line 135 — Assign `length`

```python
        length = len(indices)
```
**EN:** Assigns `length` from `len(indices)`, establishing state in function `permute`.
**CN:** 将 `length` 赋值为 `len(indices)`，用于在函数 `permute` 中建立状态。

##### Line 136 — Assign `indices`

```python
        indices = [length - idx - 1 for idx in indices]
```
**EN:** Assigns `indices` from `[length - idx - 1 for idx in indices]`, establishing state in function `permute`.
**CN:** 将 `indices` 赋值为 `[length - idx - 1 for idx in indices]`，用于在函数 `permute` 中建立状态。

##### Line 137 — Assign `self.layout`

```python
        self.layout = permutation(self.layout, indices[::-1])
```
**EN:** Assigns `self.layout` from `permutation(self.layout, indices[::-1])`, establishing state in function `permute`.
**CN:** 将 `self.layout` 赋值为 `permutation(self.layout, indices[::-1])`，用于在函数 `permute` 中建立状态。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `Tensor`.
- **CN:** 顶层类：`Tensor`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.evt.ir.layout_algorithm`, `cutlass_cppgen.utils.datatypes`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** none / 无
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
