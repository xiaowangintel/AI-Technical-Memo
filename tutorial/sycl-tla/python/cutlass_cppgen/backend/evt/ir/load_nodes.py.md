# load_nodes.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/ir/load_nodes.py`
- **EN:** Load nodes and implementations
- **CN:** 模块文档说明：Load nodes and implementations

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
Load nodes and implementations
"""
```
**EN:** Docstring explains this scope: Load nodes and implementations
**CN:** 文档字符串说明了该作用域的用途：Load nodes and implementations

### Line 37 — Import `ctypes`

```python
import ctypes
```
**EN:** Imports `ctypes` so later code can use these APIs at module scope.
**CN:** 导入 `ctypes`，供后续代码在模块级使用这些 API。

### Line 39 — From `cutlass_cppgen.backend.c_types` import

```python
from cutlass_cppgen.backend.c_types import tuple_factory
```
**EN:** Imports `tuple_factory` from `cutlass_cppgen.backend.c_types` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.c_types` 导入 `tuple_factory`，以便后续代码在模块级复用共享定义。

### Line 40 — From `cutlass_cppgen.backend.epilogue` import

```python
from cutlass_cppgen.backend.epilogue import dtype2ctype, to_ctype_value
```
**EN:** Imports `dtype2ctype, to_ctype_value` from `cutlass_cppgen.backend.epilogue` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.epilogue` 导入 `dtype2ctype, to_ctype_value`，以便后续代码在模块级复用共享定义。

### Line 41 — From `cutlass_cppgen.backend.evt.ir.node` import

```python
from cutlass_cppgen.backend.evt.ir.node import NodeBase, ImplBase
```
**EN:** Imports `NodeBase, ImplBase` from `cutlass_cppgen.backend.evt.ir.node` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir.node` 导入 `NodeBase, ImplBase`，以便后续代码在模块级复用共享定义。

### Lines 44-47 — Class `LoadImplBase`

```python
class LoadImplBase(ImplBase):
    """
    Base class for load node implementations
    """
```
**EN:** Declares class `LoadImplBase` deriving from `ImplBase`. Purpose: Base class for load node implementations
**CN:** 声明类 `LoadImplBase`，其基类为 `ImplBase`。 其用途：Base class for load node implementations

#### Line 48 — Assign `reserved_names`

```python
    reserved_names = ["accum", "C"]
```
**EN:** Assigns `reserved_names` from `['accum', 'C']`, establishing state in class `LoadImplBase`.
**CN:** 将 `reserved_names` 赋值为 `['accum', 'C']`，用于在类 `LoadImplBase` 中建立状态。

#### Line 49 — Function `__init__`

```python
    def __init__(self, node) -> None:
```
**EN:** Defines function `__init__` with parameters `self, node`.
**CN:** 定义函数 `__init__`，参数为 `self, node`。

##### Line 50 — Call `super().__init__`

```python
        super().__init__(node)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 51 — Assign `self.element`

```python
        self.element = node.element
```
**EN:** Assigns `self.element` from `node.element`, establishing state in function `__init__`.
**CN:** 将 `self.element` 赋值为 `node.element`，用于在函数 `__init__` 中建立状态。

##### Line 52 — Assign `self.element_output`

```python
        self.element_output = node.element_output
```
**EN:** Assigns `self.element_output` from `node.element_output`, establishing state in function `__init__`.
**CN:** 将 `self.element_output` 赋值为 `node.element_output`，用于在函数 `__init__` 中建立状态。

##### Line 53 — Assign `self.stride`

```python
        self.stride = node.tensor.stride
```
**EN:** Assigns `self.stride` from `node.tensor.stride`, establishing state in function `__init__`.
**CN:** 将 `self.stride` 赋值为 `node.tensor.stride`，用于在函数 `__init__` 中建立状态。

### Lines 56-59 — Class `AccumulatorImpl`

```python
class AccumulatorImpl(LoadImplBase):
    """
    Accumulator node implementation
    """
```
**EN:** Declares class `AccumulatorImpl` deriving from `LoadImplBase`. Purpose: Accumulator node implementation
**CN:** 声明类 `AccumulatorImpl`，其基类为 `LoadImplBase`。 其用途：Accumulator node implementation

#### Lines 61-62 — Function `match`

```python
    @staticmethod
    def match(node, problem_size: tuple):
```
**EN:** Defines function `match` with parameters `node, problem_size`.
**CN:** 定义函数 `match`，参数为 `node, problem_size`。

##### Line 63 — Return

```python
        return node.name == "accum" and node.tensor.shape == problem_size
```
**EN:** Returns `node.name == 'accum' and node.tensor.shape == problem_size` to the caller.
**CN:** 向调用方返回 `node.name == 'accum' and node.tensor.shape == problem_size`。

### Lines 66-69 — Class `LoadSrcImpl`

```python
class LoadSrcImpl(LoadImplBase):
    """
    Load C implementation
    """
```
**EN:** Declares class `LoadSrcImpl` deriving from `LoadImplBase`. Purpose: Load C implementation
**CN:** 声明类 `LoadSrcImpl`，其基类为 `LoadImplBase`。 其用途：Load C implementation

#### Lines 70-71 — Function `name_camel`

```python
    @property
    def name_camel(self) -> str:
```
**EN:** Defines function `name_camel` with parameters `self`.
**CN:** 定义函数 `name_camel`，参数为 `self`。

##### Line 72 — Return

```python
        return "TensorC"
```
**EN:** Returns `'TensorC'` to the caller.
**CN:** 向调用方返回 `'TensorC'`。

#### Lines 74-75 — Function `argument_type_c`

```python
    @property
    def argument_type_c(self):
```
**EN:** Defines function `argument_type_c` with parameters `self`.
**CN:** 定义函数 `argument_type_c`，参数为 `self`。

##### Line 76 — Assign `stride_mnl`

```python
        stride_mnl = self.get_stride_mnl()
```
**EN:** Assigns `stride_mnl` from `self.get_stride_mnl()`, establishing state in function `argument_type_c`.
**CN:** 将 `stride_mnl` 赋值为 `self.get_stride_mnl()`，用于在函数 `argument_type_c` 中建立状态。

##### Line 77 — Assign `tuple_type`

```python
        tuple_type = tuple_factory(stride_mnl, self.stride_dtype)
```
**EN:** Assigns `tuple_type` from `tuple_factory(stride_mnl, self.stride_dtype)`, establishing state in function `argument_type_c`.
**CN:** 将 `tuple_type` 赋值为 `tuple_factory(stride_mnl, self.stride_dtype)`，用于在函数 `argument_type_c` 中建立状态。

##### Line 78 — Class `_Argument`

```python
        class _Argument(ctypes.Structure):
```
**EN:** Declares class `_Argument` deriving from `ctypes.Structure`.
**CN:** 声明类 `_Argument`，其基类为 `ctypes.Structure`。

###### Lines 79-82 — Assign `_fields_`

```python
            _fields_ = [
                ("ptr_C", ctypes.c_void_p),
                ("stride_C", tuple_type)
            ]
```
**EN:** Assigns `_fields_` from `[('ptr_C', ctypes.c_void_p), ('stride_C', tuple_type)]`, establishing state in class `_Argument`.
**CN:** 将 `_fields_` 赋值为 `[('ptr_C', ctypes.c_void_p), ('stride_C', tuple_type)]`，用于在类 `_Argument` 中建立状态。

###### Line 83 — Function `__init__`

```python
            def __init__(self, ptr) -> None:
```
**EN:** Defines function `__init__` with parameters `self, ptr`.
**CN:** 定义函数 `__init__`，参数为 `self, ptr`。

###### Line 84 — Assign `self.ptr_C`

```python
                self.ptr_C = ptr
```
**EN:** Assigns `self.ptr_C` from `ptr`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_C` 赋值为 `ptr`，用于在函数 `__init__` 中建立状态。

###### Line 85 — Assign `self.stride_C`

```python
                self.stride_C = tuple_type(stride_mnl)
```
**EN:** Assigns `self.stride_C` from `tuple_type(stride_mnl)`, establishing state in function `__init__`.
**CN:** 将 `self.stride_C` 赋值为 `tuple_type(stride_mnl)`，用于在函数 `__init__` 中建立状态。

##### Line 87 — Return

```python
        return _Argument
```
**EN:** Returns `_Argument` to the caller.
**CN:** 向调用方返回 `_Argument`。

#### Lines 89-90 — Function `match`

```python
    @staticmethod
    def match(node, problem_size: tuple):
```
**EN:** Defines function `match` with parameters `node, problem_size`.
**CN:** 定义函数 `match`，参数为 `node, problem_size`。

##### Line 91 — Return

```python
        return node.name == "C" and node.tensor.shape == problem_size
```
**EN:** Returns `node.name == 'C' and node.tensor.shape == problem_size` to the caller.
**CN:** 向调用方返回 `node.name == 'C' and node.tensor.shape == problem_size`。

### Lines 94-97 — Class `AuxLoadImpl`

```python
class AuxLoadImpl(LoadImplBase):
    """
    Load arbitrary tensor
    """
```
**EN:** Declares class `AuxLoadImpl` deriving from `LoadImplBase`. Purpose: Load arbitrary tensor
**CN:** 声明类 `AuxLoadImpl`，其基类为 `LoadImplBase`。 其用途：Load arbitrary tensor

#### Lines 98-99 — Function `argument_type`

```python
    @property
    def argument_type(self):
```
**EN:** Defines function `argument_type` with parameters `self`.
**CN:** 定义函数 `argument_type`，参数为 `self`。

##### Line 100 — Assign `stride_mnl`

```python
        stride_mnl = self.get_stride_mnl()
```
**EN:** Assigns `stride_mnl` from `self.get_stride_mnl()`, establishing state in function `argument_type`.
**CN:** 将 `stride_mnl` 赋值为 `self.get_stride_mnl()`，用于在函数 `argument_type` 中建立状态。

##### Line 101 — Assign `name`

```python
        name = self.name
```
**EN:** Assigns `name` from `self.name`, establishing state in function `argument_type`.
**CN:** 将 `name` 赋值为 `self.name`，用于在函数 `argument_type` 中建立状态。

##### Line 102 — Assign `tuple_type`

```python
        tuple_type = tuple_factory(stride_mnl, self.stride_dtype)
```
**EN:** Assigns `tuple_type` from `tuple_factory(stride_mnl, self.stride_dtype)`, establishing state in function `argument_type`.
**CN:** 将 `tuple_type` 赋值为 `tuple_factory(stride_mnl, self.stride_dtype)`，用于在函数 `argument_type` 中建立状态。

##### Line 103 — Assign `element_type`

```python
        element_type = self.element
```
**EN:** Assigns `element_type` from `self.element`, establishing state in function `argument_type`.
**CN:** 将 `element_type` 赋值为 `self.element`，用于在函数 `argument_type` 中建立状态。

##### Line 104 — Class `_Argument`

```python
        class _Argument(ctypes.Structure):
```
**EN:** Declares class `_Argument` deriving from `ctypes.Structure`.
**CN:** 声明类 `_Argument`，其基类为 `ctypes.Structure`。

###### Lines 105-109 — Assign `_fields_`

```python
            _fields_ = [
                ("ptr_aux", ctypes.c_void_p),
                ("null_default", dtype2ctype[element_type]),
                ("dAux", tuple_type)
            ]
```
**EN:** Assigns `_fields_` from `[('ptr_aux', ctypes.c_void_p), ('null_default', dtype2ctype[element_type]), ('dAux', tuple_type)]`, establishing state in class `_Argument`.
**CN:** 将 `_fields_` 赋值为 `[('ptr_aux', ctypes.c_void_p), ('null_default', dtype2ctype[element_type]), ('dAux', tuple_type)]`，用于在类 `_Argument` 中建立状态。

###### Line 110 — Function `__init__`

```python
            def __init__(self, kwargs) -> None:
```
**EN:** Defines function `__init__` with parameters `self, kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, kwargs`。

###### Line 111 — Assign `ptr`

```python
                ptr = kwargs[name]
```
**EN:** Assigns `ptr` from `kwargs[name]`, establishing state in function `__init__`.
**CN:** 将 `ptr` 赋值为 `kwargs[name]`，用于在函数 `__init__` 中建立状态。

###### Line 112 — Assign `self.ptr_aux`

```python
                self.ptr_aux = ptr
```
**EN:** Assigns `self.ptr_aux` from `ptr`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_aux` 赋值为 `ptr`，用于在函数 `__init__` 中建立状态。

###### Line 113 — Assign `self.null_default`

```python
                self.null_default = to_ctype_value(0, element_type)
```
**EN:** Assigns `self.null_default` from `to_ctype_value(0, element_type)`, establishing state in function `__init__`.
**CN:** 将 `self.null_default` 赋值为 `to_ctype_value(0, element_type)`，用于在函数 `__init__` 中建立状态。

###### Line 114 — Assign `self.dAux`

```python
                self.dAux = tuple_type(stride_mnl)
```
**EN:** Assigns `self.dAux` from `tuple_type(stride_mnl)`, establishing state in function `__init__`.
**CN:** 将 `self.dAux` 赋值为 `tuple_type(stride_mnl)`，用于在函数 `__init__` 中建立状态。

##### Line 116 — Return

```python
        return _Argument
```
**EN:** Returns `_Argument` to the caller.
**CN:** 向调用方返回 `_Argument`。

#### Lines 118-119 — Function `match`

```python
    @staticmethod
    def match(node, problem_size: tuple):
```
**EN:** Defines function `match` with parameters `node, problem_size`.
**CN:** 定义函数 `match`，参数为 `node, problem_size`。

##### Lines 120-121 — Conditional `node.name in LoadImplBase.reserved_names`

```python
        if node.name in LoadImplBase.reserved_names:
            return False
```
**EN:** Checks `node.name in LoadImplBase.reserved_names` and selects the matching branch in function `match`.
**CN:** 检查 `node.name in LoadImplBase.reserved_names`，并在函数 `match` 中选择匹配的分支。

##### Line 122 — Assign `strideMN`

```python
        strideMN = node.tensor.stride[-2:]
```
**EN:** Assigns `strideMN` from `node.tensor.stride[-2:]`, establishing state in function `match`.
**CN:** 将 `strideMN` 赋值为 `node.tensor.stride[-2:]`，用于在函数 `match` 中建立状态。

##### Lines 123-127 — Conditional `strideMN[0] == 1 and strideMN[1] != 0 or (strideMN[0] != 0 and strideMN[1] == 1)`

```python
        if (strideMN[0] == 1 and strideMN[1] != 0 or
            strideMN[0] != 0 and strideMN[1] == 1 ):
            return True
        else:
            return False
```
**EN:** Checks `strideMN[0] == 1 and strideMN[1] != 0 or (strideMN[0] != 0 and strideMN[1] == 1)` and selects the matching branch in function `match`.
**CN:** 检查 `strideMN[0] == 1 and strideMN[1] != 0 or (strideMN[0] != 0 and strideMN[1] == 1)`，并在函数 `match` 中选择匹配的分支。

### Lines 130-133 — Class `RowBroadcastImpl`

```python
class RowBroadcastImpl(LoadImplBase):
    """
    Broadcast a row vector
    """
```
**EN:** Declares class `RowBroadcastImpl` deriving from `LoadImplBase`. Purpose: Broadcast a row vector
**CN:** 声明类 `RowBroadcastImpl`，其基类为 `LoadImplBase`。 其用途：Broadcast a row vector

#### Line 134 — Function `__init__`

```python
    def __init__(self, node) -> None:
```
**EN:** Defines function `__init__` with parameters `self, node`.
**CN:** 定义函数 `__init__`，参数为 `self, node`。

##### Line 135 — Call `super().__init__`

```python
        super().__init__(node)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 136 — Assign `self.stride_dtype`

```python
        self.stride_dtype = "int"
```
**EN:** Assigns `self.stride_dtype` from `'int'`, establishing state in function `__init__`.
**CN:** 将 `self.stride_dtype` 赋值为 `'int'`，用于在函数 `__init__` 中建立状态。

#### Lines 138-139 — Function `argument_type`

```python
    @property
    def argument_type(self):
```
**EN:** Defines function `argument_type` with parameters `self`.
**CN:** 定义函数 `argument_type`，参数为 `self`。

##### Line 140 — Assign `stride_mnl`

```python
        stride_mnl = self.get_stride_mnl()
```
**EN:** Assigns `stride_mnl` from `self.get_stride_mnl()`, establishing state in function `argument_type`.
**CN:** 将 `stride_mnl` 赋值为 `self.get_stride_mnl()`，用于在函数 `argument_type` 中建立状态。

##### Line 141 — Assign `name`

```python
        name = self.name
```
**EN:** Assigns `name` from `self.name`, establishing state in function `argument_type`.
**CN:** 将 `name` 赋值为 `self.name`，用于在函数 `argument_type` 中建立状态。

##### Line 142 — Assign `tuple_type`

```python
        tuple_type = tuple_factory(stride_mnl, self.stride_dtype)
```
**EN:** Assigns `tuple_type` from `tuple_factory(stride_mnl, self.stride_dtype)`, establishing state in function `argument_type`.
**CN:** 将 `tuple_type` 赋值为 `tuple_factory(stride_mnl, self.stride_dtype)`，用于在函数 `argument_type` 中建立状态。

##### Line 143 — Assign `element_type`

```python
        element_type = self.element
```
**EN:** Assigns `element_type` from `self.element`, establishing state in function `argument_type`.
**CN:** 将 `element_type` 赋值为 `self.element`，用于在函数 `argument_type` 中建立状态。

##### Line 144 — Class `_Argument`

```python
        class _Argument(ctypes.Structure):
```
**EN:** Declares class `_Argument` deriving from `ctypes.Structure`.
**CN:** 声明类 `_Argument`，其基类为 `ctypes.Structure`。

###### Lines 145-149 — Assign `_fields_`

```python
            _fields_ = [
                ("ptr_row", ctypes.c_void_p),
                ("null_default", dtype2ctype[element_type]),
                ("dRow", tuple_type)
            ]
```
**EN:** Assigns `_fields_` from `[('ptr_row', ctypes.c_void_p), ('null_default', dtype2ctype[element_type]), ('dRow', tuple_type)]`, establishing state in class `_Argument`.
**CN:** 将 `_fields_` 赋值为 `[('ptr_row', ctypes.c_void_p), ('null_default', dtype2ctype[element_type]), ('dRow', tuple_type)]`，用于在类 `_Argument` 中建立状态。

###### Line 150 — Function `__init__`

```python
            def __init__(self, kwargs) -> None:
```
**EN:** Defines function `__init__` with parameters `self, kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, kwargs`。

###### Line 151 — Assign `ptr`

```python
                ptr = kwargs[name]
```
**EN:** Assigns `ptr` from `kwargs[name]`, establishing state in function `__init__`.
**CN:** 将 `ptr` 赋值为 `kwargs[name]`，用于在函数 `__init__` 中建立状态。

###### Line 152 — Assign `self.ptr_row`

```python
                self.ptr_row = ptr
```
**EN:** Assigns `self.ptr_row` from `ptr`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_row` 赋值为 `ptr`，用于在函数 `__init__` 中建立状态。

###### Line 153 — Assign `self.null_default`

```python
                self.null_default = to_ctype_value(0, element_type)
```
**EN:** Assigns `self.null_default` from `to_ctype_value(0, element_type)`, establishing state in function `__init__`.
**CN:** 将 `self.null_default` 赋值为 `to_ctype_value(0, element_type)`，用于在函数 `__init__` 中建立状态。

###### Line 154 — Assign `self.dRow`

```python
                self.dRow = tuple_type(stride_mnl)
```
**EN:** Assigns `self.dRow` from `tuple_type(stride_mnl)`, establishing state in function `__init__`.
**CN:** 将 `self.dRow` 赋值为 `tuple_type(stride_mnl)`，用于在函数 `__init__` 中建立状态。

##### Line 156 — Return

```python
        return _Argument
```
**EN:** Returns `_Argument` to the caller.
**CN:** 向调用方返回 `_Argument`。

#### Lines 158-159 — Function `match`

```python
    @staticmethod
    def match(node, problem_size: tuple):
```
**EN:** Defines function `match` with parameters `node, problem_size`.
**CN:** 定义函数 `match`，参数为 `node, problem_size`。

##### Lines 160-161 — Conditional `node.name in LoadImplBase.reserved_names`

```python
        if node.name in LoadImplBase.reserved_names:
            return False
```
**EN:** Checks `node.name in LoadImplBase.reserved_names` and selects the matching branch in function `match`.
**CN:** 检查 `node.name in LoadImplBase.reserved_names`，并在函数 `match` 中选择匹配的分支。

##### Line 163 — Assign `strideMN`

```python
        strideMN = node.tensor.stride[-2:]
```
**EN:** Assigns `strideMN` from `node.tensor.stride[-2:]`, establishing state in function `match`.
**CN:** 将 `strideMN` 赋值为 `node.tensor.stride[-2:]`，用于在函数 `match` 中建立状态。

##### Lines 164-167 — Conditional `strideMN == (0, 1)`

```python
        if strideMN == (0, 1):
            return True
        else:
            return False
```
**EN:** Checks `strideMN == (0, 1)` and selects the matching branch in function `match`.
**CN:** 检查 `strideMN == (0, 1)`，并在函数 `match` 中选择匹配的分支。

### Lines 170-173 — Class `ColumnBroadcastImpl`

```python
class ColumnBroadcastImpl(LoadImplBase):
    """
    Broadcast a column vector
    """
```
**EN:** Declares class `ColumnBroadcastImpl` deriving from `LoadImplBase`. Purpose: Broadcast a column vector
**CN:** 声明类 `ColumnBroadcastImpl`，其基类为 `LoadImplBase`。 其用途：Broadcast a column vector

#### Line 174 — Function `__init__`

```python
    def __init__(self, node) -> None:
```
**EN:** Defines function `__init__` with parameters `self, node`.
**CN:** 定义函数 `__init__`，参数为 `self, node`。

##### Line 175 — Call `super().__init__`

```python
        super().__init__(node)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 176 — Assign `self.stride_dtype`

```python
        self.stride_dtype = "int"
```
**EN:** Assigns `self.stride_dtype` from `'int'`, establishing state in function `__init__`.
**CN:** 将 `self.stride_dtype` 赋值为 `'int'`，用于在函数 `__init__` 中建立状态。

#### Lines 178-179 — Function `argument_type`

```python
    @property
    def argument_type(self):
```
**EN:** Defines function `argument_type` with parameters `self`.
**CN:** 定义函数 `argument_type`，参数为 `self`。

##### Line 180 — Assign `stride_mnl`

```python
        stride_mnl = self.get_stride_mnl()
```
**EN:** Assigns `stride_mnl` from `self.get_stride_mnl()`, establishing state in function `argument_type`.
**CN:** 将 `stride_mnl` 赋值为 `self.get_stride_mnl()`，用于在函数 `argument_type` 中建立状态。

##### Line 181 — Assign `name`

```python
        name = self.name
```
**EN:** Assigns `name` from `self.name`, establishing state in function `argument_type`.
**CN:** 将 `name` 赋值为 `self.name`，用于在函数 `argument_type` 中建立状态。

##### Line 182 — Assign `tuple_type`

```python
        tuple_type = tuple_factory(stride_mnl, self.stride_dtype)
```
**EN:** Assigns `tuple_type` from `tuple_factory(stride_mnl, self.stride_dtype)`, establishing state in function `argument_type`.
**CN:** 将 `tuple_type` 赋值为 `tuple_factory(stride_mnl, self.stride_dtype)`，用于在函数 `argument_type` 中建立状态。

##### Line 183 — Assign `element_type`

```python
        element_type = self.element
```
**EN:** Assigns `element_type` from `self.element`, establishing state in function `argument_type`.
**CN:** 将 `element_type` 赋值为 `self.element`，用于在函数 `argument_type` 中建立状态。

##### Line 184 — Class `_Argument`

```python
        class _Argument(ctypes.Structure):
```
**EN:** Declares class `_Argument` deriving from `ctypes.Structure`.
**CN:** 声明类 `_Argument`，其基类为 `ctypes.Structure`。

###### Lines 185-189 — Assign `_fields_`

```python
            _fields_ = [
                ("ptr_col", ctypes.c_void_p),
                ("null_default", dtype2ctype[element_type]),
                ("dCol", tuple_type)
            ]
```
**EN:** Assigns `_fields_` from `[('ptr_col', ctypes.c_void_p), ('null_default', dtype2ctype[element_type]), ('dCol', tuple_type)]`, establishing state in class `_Argument`.
**CN:** 将 `_fields_` 赋值为 `[('ptr_col', ctypes.c_void_p), ('null_default', dtype2ctype[element_type]), ('dCol', tuple_type)]`，用于在类 `_Argument` 中建立状态。

###### Line 190 — Function `__init__`

```python
            def __init__(self, kwargs) -> None:
```
**EN:** Defines function `__init__` with parameters `self, kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, kwargs`。

###### Line 191 — Assign `ptr`

```python
                ptr = kwargs[name]
```
**EN:** Assigns `ptr` from `kwargs[name]`, establishing state in function `__init__`.
**CN:** 将 `ptr` 赋值为 `kwargs[name]`，用于在函数 `__init__` 中建立状态。

###### Line 192 — Assign `self.ptr_col`

```python
                self.ptr_col = int(ptr)
```
**EN:** Assigns `self.ptr_col` from `int(ptr)`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_col` 赋值为 `int(ptr)`，用于在函数 `__init__` 中建立状态。

###### Line 193 — Assign `self.null_default`

```python
                self.null_default = to_ctype_value(0, element_type)
```
**EN:** Assigns `self.null_default` from `to_ctype_value(0, element_type)`, establishing state in function `__init__`.
**CN:** 将 `self.null_default` 赋值为 `to_ctype_value(0, element_type)`，用于在函数 `__init__` 中建立状态。

###### Line 194 — Assign `self.dCol`

```python
                self.dCol = tuple_type(stride_mnl)
```
**EN:** Assigns `self.dCol` from `tuple_type(stride_mnl)`, establishing state in function `__init__`.
**CN:** 将 `self.dCol` 赋值为 `tuple_type(stride_mnl)`，用于在函数 `__init__` 中建立状态。

##### Line 196 — Return

```python
        return _Argument
```
**EN:** Returns `_Argument` to the caller.
**CN:** 向调用方返回 `_Argument`。

#### Lines 198-199 — Function `match`

```python
    @staticmethod
    def match(node, problem_size: tuple):
```
**EN:** Defines function `match` with parameters `node, problem_size`.
**CN:** 定义函数 `match`，参数为 `node, problem_size`。

##### Lines 200-201 — Conditional `node.name in LoadImplBase.reserved_names`

```python
        if node.name in LoadImplBase.reserved_names:
            return False
```
**EN:** Checks `node.name in LoadImplBase.reserved_names` and selects the matching branch in function `match`.
**CN:** 检查 `node.name in LoadImplBase.reserved_names`，并在函数 `match` 中选择匹配的分支。

##### Line 203 — Assign `strideMN`

```python
        strideMN = node.tensor.stride[-2:]
```
**EN:** Assigns `strideMN` from `node.tensor.stride[-2:]`, establishing state in function `match`.
**CN:** 将 `strideMN` 赋值为 `node.tensor.stride[-2:]`，用于在函数 `match` 中建立状态。

##### Lines 204-207 — Conditional `strideMN == (1, 0)`

```python
        if strideMN == (1, 0):
            return True
        else:
            return False
```
**EN:** Checks `strideMN == (1, 0)` and selects the matching branch in function `match`.
**CN:** 检查 `strideMN == (1, 0)`，并在函数 `match` 中选择匹配的分支。

### Lines 210-213 — Class `ScalarBroadcastImpl`

```python
class ScalarBroadcastImpl(LoadImplBase):
    """
    Broadcast a scalar
    """
```
**EN:** Declares class `ScalarBroadcastImpl` deriving from `LoadImplBase`. Purpose: Broadcast a scalar
**CN:** 声明类 `ScalarBroadcastImpl`，其基类为 `LoadImplBase`。 其用途：Broadcast a scalar

#### Line 214 — Function `__init__`

```python
    def __init__(self, node) -> None:
```
**EN:** Defines function `__init__` with parameters `self, node`.
**CN:** 定义函数 `__init__`，参数为 `self, node`。

##### Line 215 — Call `super().__init__`

```python
        super().__init__(node)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 216 — Assign `self.stride_dtype`

```python
        self.stride_dtype = "int"
```
**EN:** Assigns `self.stride_dtype` from `'int'`, establishing state in function `__init__`.
**CN:** 将 `self.stride_dtype` 赋值为 `'int'`，用于在函数 `__init__` 中建立状态。

#### Lines 218-219 — Function `argument_type`

```python
    @property
    def argument_type(self):
```
**EN:** Defines function `argument_type` with parameters `self`.
**CN:** 定义函数 `argument_type`，参数为 `self`。

##### Line 220 — Assign `stride_mnl`

```python
        stride_mnl = self.get_stride_mnl()
```
**EN:** Assigns `stride_mnl` from `self.get_stride_mnl()`, establishing state in function `argument_type`.
**CN:** 将 `stride_mnl` 赋值为 `self.get_stride_mnl()`，用于在函数 `argument_type` 中建立状态。

##### Line 221 — Assign `name`

```python
        name = self.name
```
**EN:** Assigns `name` from `self.name`, establishing state in function `argument_type`.
**CN:** 将 `name` 赋值为 `self.name`，用于在函数 `argument_type` 中建立状态。

##### Line 222 — Assign `tuple_type`

```python
        tuple_type = tuple_factory(stride_mnl, self.stride_dtype)
```
**EN:** Assigns `tuple_type` from `tuple_factory(stride_mnl, self.stride_dtype)`, establishing state in function `argument_type`.
**CN:** 将 `tuple_type` 赋值为 `tuple_factory(stride_mnl, self.stride_dtype)`，用于在函数 `argument_type` 中建立状态。

##### Line 223 — Assign `element_type`

```python
        element_type = self.element
```
**EN:** Assigns `element_type` from `self.element`, establishing state in function `argument_type`.
**CN:** 将 `element_type` 赋值为 `self.element`，用于在函数 `argument_type` 中建立状态。

##### Lines 225-253 — Conditional `self.tensor.is_constant`

```python
        if self.tensor.is_constant:
            value = self.tensor.value
            class _Argument(ctypes.Structure):
                _fields_ = [
                    ("scalars", dtype2ctype[element_type]),
                    ("scalar_ptrs", ctypes.c_void_p),
                    ("dScalar", tuple_type)
                ]
                def __init__(self, kwargs) -> None:
                    self.scalars = to_ctype_value(value, element_type)
                    self.scalar_ptrs = 0
                    self.dScalar = tuple_type(stride_mnl)

        else:
            class _Argument(ctypes.Structure):
                _fields_ = [
                    ("scalars", dtype2ctype[element_type]),
                    ("scalar_ptrs", ctypes.c_void_p),
                    ("dScalar", tuple_type)
                ]
                def __init__(self, kwargs) -> None:
                    scalar_or_ptr = kwargs[name]
                    if isinstance(scalar_or_ptr, float):
                        self.scalars = to_ctype_value(scalar_or_ptr, element_type)
                        self.scalar_ptrs = 0
                    else:
                        self.scalar_ptrs = int(scalar_or_ptr)

                    self.dScalar = tuple_type(stride_mnl)
```
**EN:** Checks `self.tensor.is_constant` and selects the matching branch in function `argument_type`.
**CN:** 检查 `self.tensor.is_constant`，并在函数 `argument_type` 中选择匹配的分支。

##### Line 255 — Return

```python
        return _Argument
```
**EN:** Returns `_Argument` to the caller.
**CN:** 向调用方返回 `_Argument`。

#### Lines 257-258 — Function `match`

```python
    @staticmethod
    def match(node, problem_size: tuple):
```
**EN:** Defines function `match` with parameters `node, problem_size`.
**CN:** 定义函数 `match`，参数为 `node, problem_size`。

##### Lines 259-260 — Conditional `node.name in LoadImplBase.reserved_names`

```python
        if node.name in LoadImplBase.reserved_names:
            return False
```
**EN:** Checks `node.name in LoadImplBase.reserved_names` and selects the matching branch in function `match`.
**CN:** 检查 `node.name in LoadImplBase.reserved_names`，并在函数 `match` 中选择匹配的分支。

##### Line 262 — Assign `strideMN`

```python
        strideMN = node.tensor.stride[-2:]
```
**EN:** Assigns `strideMN` from `node.tensor.stride[-2:]`, establishing state in function `match`.
**CN:** 将 `strideMN` 赋值为 `node.tensor.stride[-2:]`，用于在函数 `match` 中建立状态。

##### Lines 263-266 — Conditional `strideMN == (0, 0)`

```python
        if strideMN == (0, 0):
            return True
        else:
            return False
```
**EN:** Checks `strideMN == (0, 0)` and selects the matching branch in function `match`.
**CN:** 检查 `strideMN == (0, 0)`，并在函数 `match` 中选择匹配的分支。

### Lines 269-272 — Class `LoadNode`

```python
class LoadNode(NodeBase):
    """
    Load Node
    """
```
**EN:** Declares class `LoadNode` deriving from `NodeBase`. Purpose: Load Node
**CN:** 声明类 `LoadNode`，其基类为 `NodeBase`。 其用途：Load Node

#### Line 273 — Assign `cnt`

```python
    cnt = 0
```
**EN:** Assigns `cnt` from `0`, establishing state in class `LoadNode`.
**CN:** 将 `cnt` 赋值为 `0`，用于在类 `LoadNode` 中建立状态。

#### Lines 274-278 — Assign `possible_impls`

```python
    possible_impls = [
        AccumulatorImpl, LoadSrcImpl, AuxLoadImpl,
        RowBroadcastImpl, ColumnBroadcastImpl,
        ScalarBroadcastImpl
    ]
```
**EN:** Assigns `possible_impls` from `[AccumulatorImpl, LoadSrcImpl, AuxLoadImpl, RowBroadcastImpl, ColumnBroadcastImpl, ScalarBroadcastImpl]`, establishing state in class `LoadNode`.
**CN:** 将 `possible_impls` 赋值为 `[AccumulatorImpl, LoadSrcImpl, AuxLoadImpl, RowBroadcastImpl, ColumnBroadcastImpl, ScalarBroadcastImpl]`，用于在类 `LoadNode` 中建立状态。

#### Line 279 — Function `__init__`

```python
    def __init__(self, name: str) -> None:
```
**EN:** Defines function `__init__` with parameters `self, name`.
**CN:** 定义函数 `__init__`，参数为 `self, name`。

##### Lines 280-282 — Conditional `name is None`

```python
        if name is None:
            name = f"load{LoadNode.cnt}"
            LoadNode.cnt += 1
```
**EN:** Checks `name is None` and selects the matching branch in function `__init__`.
**CN:** 检查 `name is None`，并在函数 `__init__` 中选择匹配的分支。

##### Line 283 — Call `super().__init__`

```python
        super().__init__(name)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 284 — Assign `self.op`

```python
        self.op = "load"
```
**EN:** Assigns `self.op` from `'load'`, establishing state in function `__init__`.
**CN:** 将 `self.op` 赋值为 `'load'`，用于在函数 `__init__` 中建立状态。

#### Lines 286-289 — Function `type_propagation`

```python
    def type_propagation(self, *args, **kwargs):
        """
        Load node loads tensor under type `tensor.element` and returns an array of type `tensor.element`.
        """
```
**EN:** Defines function `type_propagation` with parameters `self, *args, **kwargs`. Purpose: Load node loads tensor under type `tensor.element` and returns an array of type `tensor.element`.
**CN:** 定义函数 `type_propagation`，参数为 `self, *args, **kwargs`。 其用途：Load node loads tensor under type `tensor.element` and returns an array of type `tensor.element`.

##### Lines 290-291 — Conditional `self.tensor is None`

```python
        if self.tensor is None:
            raise RuntimeError(f"The tensor of node {self.name} is unknown.")
```
**EN:** Checks `self.tensor is None` and selects the matching branch in function `type_propagation`.
**CN:** 检查 `self.tensor is None`，并在函数 `type_propagation` 中选择匹配的分支。

##### Line 293 — Assign `self.element`

```python
        self.element = self.tensor.element
```
**EN:** Assigns `self.element` from `self.tensor.element`, establishing state in function `type_propagation`.
**CN:** 将 `self.element` 赋值为 `self.tensor.element`，用于在函数 `type_propagation` 中建立状态。

##### Line 294 — Assign `self.element_output`

```python
        self.element_output = self.tensor.element
```
**EN:** Assigns `self.element_output` from `self.tensor.element`, establishing state in function `type_propagation`.
**CN:** 将 `self.element_output` 赋值为 `self.tensor.element`，用于在函数 `type_propagation` 中建立状态。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `LoadImplBase`, `AccumulatorImpl`, `LoadSrcImpl`, `AuxLoadImpl`, `RowBroadcastImpl`, `ColumnBroadcastImpl`, `ScalarBroadcastImpl`, `LoadNode`.
- **CN:** 顶层类：`LoadImplBase`, `AccumulatorImpl`, `LoadSrcImpl`, `AuxLoadImpl`, `RowBroadcastImpl`, `ColumnBroadcastImpl`, `ScalarBroadcastImpl`, `LoadNode`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.c_types`, `cutlass_cppgen.backend.epilogue`, `cutlass_cppgen.backend.evt.ir.node`
- **Standard & third-party / 标准库与第三方:** `ctypes`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
