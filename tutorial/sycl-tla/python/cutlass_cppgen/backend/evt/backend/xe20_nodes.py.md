# xe20_nodes.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/backend/xe20_nodes.py`
- **EN:** Defines classes `xe20AccumulatorImpl`, `xe20LoadSrcImpl`, `xe20AuxLoadImpl`, `xe20ScalarBroadcastImpl`, `xe20RowBroadcastImpl`, `xe20ColumnBroadcastImpl` and functions `_needs_atomic_float` for the cutlass_cppgen package.
- **CN:** 为 cutlass_cppgen 包定义类 `xe20AccumulatorImpl`, `xe20LoadSrcImpl`, `xe20AuxLoadImpl`, `xe20ScalarBroadcastImpl`, `xe20RowBroadcastImpl`, `xe20ColumnBroadcastImpl`和函数 `_needs_atomic_float`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31 — File header

```python
###################################################################################################
# Copyright (C) 2025 - 2026 Intel Corporation, All rights reserved.
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
# ################################################################################################
```
**EN:** Contains the file header, license notice, and opening comments for the module.
**CN:** 包含文件头、许可证声明以及模块开头的注释。

### Line 32 — From `pycute` import

```python
from pycute import product
```
**EN:** Imports `product` from `pycute` to reuse shared definitions at module scope.
**CN:** 从 `pycute` 导入 `product`，以便后续代码在模块级复用共享定义。

### Line 34 — From `cutlass_library` import

```python
from cutlass_library import DataType, DataTypeSize, DataTypeTag
```
**EN:** Imports `DataType, DataTypeSize, DataTypeTag` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `DataType, DataTypeSize, DataTypeTag`，以便后续代码在模块级复用共享定义。

### Lines 35-54 — From `cutlass_cppgen.backend.evt.ir` import

```python
from cutlass_cppgen.backend.evt.ir import (
    # Load Node
    AccumulatorImpl,
    AuxLoadImpl,
    ColumnBroadcastImpl,
    LoadNode,
    LoadSrcImpl,
    RowBroadcastImpl,
    ScalarBroadcastImpl,
    # Compute Node
    ComputeImpl,
    ComputeNode,
    # Store Node
    AuxStoreImpl,
    ColumnReductionImpl,
    RowReductionImpl,
    ScalarReductionImpl,
    StoreNode,
    StoreDImpl,
)
```
**EN:** Imports `AccumulatorImpl, AuxLoadImpl, ColumnBroadcastImpl, LoadNode, LoadSrcImpl, RowBroadcastImpl, ScalarBroadcastImpl, ComputeImpl, ComputeNode, AuxStoreImpl, ColumnReductionImpl, RowReductionImpl, ScalarReductionImpl, StoreNode, StoreDImpl` from `cutlass_cppgen.backend.evt.ir` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir` 导入 `AccumulatorImpl, AuxLoadImpl, ColumnBroadcastImpl, LoadNode, LoadSrcImpl, RowBroadcastImpl, ScalarBroadcastImpl, ComputeImpl, ComputeNode, AuxStoreImpl, ColumnReductionImpl, RowReductionImpl, ScalarReductionImpl, StoreNode, StoreDImpl`，以便后续代码在模块级复用共享定义。

### Lines 55-59 — From `cutlass_cppgen.backend.library` import

```python
from cutlass_cppgen.backend.library import (
    FloatRoundStyleTag,
    FunctionalOp,
    op_tag,
)
```
**EN:** Imports `FloatRoundStyleTag, FunctionalOp, op_tag` from `cutlass_cppgen.backend.library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.library` 导入 `FloatRoundStyleTag, FunctionalOp, op_tag`，以便后续代码在模块级复用共享定义。

### Line 61 — Assign `ATOMIC_GMEM_OPS`

```python
ATOMIC_GMEM_OPS = {FunctionalOp.AtomicAdd, FunctionalOp.AtomicMaximum}
```
**EN:** Assigns `ATOMIC_GMEM_OPS` from `{FunctionalOp.AtomicAdd, FunctionalOp.AtomicMaximum}`, establishing state at module scope.
**CN:** 将 `ATOMIC_GMEM_OPS` 赋值为 `{FunctionalOp.AtomicAdd, FunctionalOp.AtomicMaximum}`，用于在模块级建立状态。

### Line 63 — Function `_needs_atomic_float`

```python
def _needs_atomic_float(element_dtype, gmem_reduce_fn):
```
**EN:** Defines function `_needs_atomic_float` with parameters `element_dtype, gmem_reduce_fn`.
**CN:** 定义函数 `_needs_atomic_float`，参数为 `element_dtype, gmem_reduce_fn`。

#### Lines 64-67 — Return

```python
    return (
        gmem_reduce_fn in ATOMIC_GMEM_OPS and
        element_dtype in (DataType.f16, DataType.bf16)
    )
```
**EN:** Returns `gmem_reduce_fn in ATOMIC_GMEM_OPS and element_dtype in (DataType.f16, DataType.bf16)` to the caller.
**CN:** 向调用方返回 `gmem_reduce_fn in ATOMIC_GMEM_OPS and element_dtype in (DataType.f16, DataType.bf16)`。

### Lines 71-73 — Class `xe20AccumulatorImpl`

```python
class xe20AccumulatorImpl(AccumulatorImpl):

    @property
```
**EN:** Declares class `xe20AccumulatorImpl` deriving from `AccumulatorImpl`.
**CN:** 声明类 `xe20AccumulatorImpl`，其基类为 `AccumulatorImpl`。

#### Lines 73-77 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 78-79 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Line 81 — Assign `self._type_decl`

```python
        self._type_decl = f"""\nusing {self.name_camel} = cutlass::epilogue::fusion::XeAccFetch;\n"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeAccFetch;\n'`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeAccFetch;\n'`，用于在函数 `type_decl` 中建立状态。

##### Line 82 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 85-87 — Class `xe20LoadSrcImpl`

```python
class xe20LoadSrcImpl(LoadSrcImpl):

    @property
```
**EN:** Declares class `xe20LoadSrcImpl` deriving from `LoadSrcImpl`.
**CN:** 声明类 `xe20LoadSrcImpl`，其基类为 `LoadSrcImpl`。

#### Lines 87-91 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 92-93 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 95-99 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using ElementC = {DataTypeTag[self.element]};
using StrideC = {self.stride_mnl};
using {self.name_camel} = cutlass::epilogue::fusion::XeSrcFetch<{DataTypeTag[self.element]}>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing ElementC = {DataTypeTag[self.element]};\nusing StrideC = {self.stride_mnl};\nusing {self.name_camel} = cutlass::epilogue::fusion::XeSrcFetch<{DataT...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing ElementC = {DataTypeTag[self.element]};\nusing StrideC = {self.stride_mnl};\nusing {self.name_camel} = cutlass::epilogue::fusion::XeSrcFetch<{DataT...`，用于在函数 `type_decl` 中建立状态。

##### Line 100 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 103-105 — Class `xe20AuxLoadImpl`

```python
class xe20AuxLoadImpl(AuxLoadImpl):

    @property
```
**EN:** Declares class `xe20AuxLoadImpl` deriving from `AuxLoadImpl`.
**CN:** 声明类 `xe20AuxLoadImpl`，其基类为 `AuxLoadImpl`。

#### Lines 105-110 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type using XeAuxLoad directly (no descriptor needed)
        XeAuxLoad auto-deduces copy operation from Element type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type using XeAuxLoad directly (no descriptor needed)
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type using XeAuxLoad directly (no descriptor needed)

##### Lines 111-112 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 114-119 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::XeAuxLoad<
    {DataTypeTag[self.element]},
    {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeAuxLoad<\n {DataTypeTag[self.element]},\n {self.stride_mnl}\n>;\n'`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeAuxLoad<\n {DataTypeTag[self.element]},\n {self.stride_mnl}\n>;\n'`，用于在函数 `type_decl` 中建立状态。

##### Line 120 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Line 123 — Class `xe20ScalarBroadcastImpl`

```python
class xe20ScalarBroadcastImpl(ScalarBroadcastImpl):
```
**EN:** Declares class `xe20ScalarBroadcastImpl` deriving from `ScalarBroadcastImpl`.
**CN:** 声明类 `xe20ScalarBroadcastImpl`，其基类为 `ScalarBroadcastImpl`。

#### Line 124 — Function `__init__`

```python
    def __init__(self, node: LoadNode) -> None:
```
**EN:** Defines function `__init__` with parameters `self, node`.
**CN:** 定义函数 `__init__`，参数为 `self, node`。

##### Line 125 — Call `super().__init__`

```python
        super().__init__(node)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 126 — Assign `self.broadcast_count`

```python
        self.broadcast_count = 1
```
**EN:** Assigns `self.broadcast_count` from `1`, establishing state in function `__init__`.
**CN:** 将 `self.broadcast_count` 赋值为 `1`，用于在函数 `__init__` 中建立状态。

##### Line 127 — Assign `self.reduction_fn`

```python
        self.reduction_fn = FunctionalOp.Multiplies
```
**EN:** Assigns `self.reduction_fn` from `FunctionalOp.Multiplies`, establishing state in function `__init__`.
**CN:** 将 `self.reduction_fn` 赋值为 `FunctionalOp.Multiplies`，用于在函数 `__init__` 中建立状态。

#### Lines 129-133 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 134-135 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 137-141 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::XeScalarBroadcast<
    {DataTypeTag[self.element]}, {self.stride_mnl}, {self.broadcast_count}, {op_tag(self.reduction_fn)}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeScalarBroadcast<\n {DataTypeTag[self.element]}, {self.stride_mnl}, {self.broadcast_count}, {op_tag...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeScalarBroadcast<\n {DataTypeTag[self.element]}, {self.stride_mnl}, {self.broadcast_count}, {op_tag...`，用于在函数 `type_decl` 中建立状态。

##### Line 142 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 145-146 — Class `xe20RowBroadcastImpl`

```python
class xe20RowBroadcastImpl(RowBroadcastImpl):
    @property
```
**EN:** Declares class `xe20RowBroadcastImpl` deriving from `RowBroadcastImpl`.
**CN:** 声明类 `xe20RowBroadcastImpl`，其基类为 `RowBroadcastImpl`。

#### Lines 146-150 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 151-152 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 154-159 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::XeRowBroadcast<
    0 /*Stages*/, TileShape_MNK, {DataTypeTag[self.element]}, {DataTypeTag[self.element_output]},
    {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeRowBroadcast<\n 0 /*Stages*/, TileShape_MNK, {DataTypeTag[self.element]}, {DataTypeTag[self.elemen...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeRowBroadcast<\n 0 /*Stages*/, TileShape_MNK, {DataTypeTag[self.element]}, {DataTypeTag[self.elemen...`，用于在函数 `type_decl` 中建立状态。

##### Line 160 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 163-165 — Class `xe20ColumnBroadcastImpl`

```python
class xe20ColumnBroadcastImpl(ColumnBroadcastImpl):

    @property
```
**EN:** Declares class `xe20ColumnBroadcastImpl` deriving from `ColumnBroadcastImpl`.
**CN:** 声明类 `xe20ColumnBroadcastImpl`，其基类为 `ColumnBroadcastImpl`。

#### Lines 165-169 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 170-171 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 173-178 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::XeColBroadcast<
    0 /*Stages*/, TileShape_MNK, {DataTypeTag[self.element]}, {DataTypeTag[self.element_output]},
    {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeColBroadcast<\n 0 /*Stages*/, TileShape_MNK, {DataTypeTag[self.element]}, {DataTypeTag[self.elemen...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeColBroadcast<\n 0 /*Stages*/, TileShape_MNK, {DataTypeTag[self.element]}, {DataTypeTag[self.elemen...`，用于在函数 `type_decl` 中建立状态。

##### Line 179 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 182-184 — Class `xe20ComputeImpl`

```python
class xe20ComputeImpl(ComputeImpl):

    @property
```
**EN:** Declares class `xe20ComputeImpl` deriving from `ComputeImpl`.
**CN:** 声明类 `xe20ComputeImpl`，其基类为 `ComputeImpl`。

#### Lines 184-188 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 189-190 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 192-197 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::XeCompute<
    {op_tag(self.fn)}, {DataTypeTag[self.element_output]}, {DataTypeTag[self.element_compute]},
    {FloatRoundStyleTag[self.round_style]}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeCompute<\n {op_tag(self.fn)}, {DataTypeTag[self.element_output]}, {DataTypeTag[self.element_comput...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeCompute<\n {op_tag(self.fn)}, {DataTypeTag[self.element_output]}, {DataTypeTag[self.element_comput...`，用于在函数 `type_decl` 中建立状态。

##### Line 198 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 201-203 — Class `xe20AuxStoreImpl`

```python
class xe20AuxStoreImpl(AuxStoreImpl):

    @property
```
**EN:** Declares class `xe20AuxStoreImpl` deriving from `AuxStoreImpl`.
**CN:** 声明类 `xe20AuxStoreImpl`，其基类为 `AuxStoreImpl`。

#### Lines 203-207 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 208-209 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 211-216 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::XeAuxStore<
    {DataTypeTag[self.element]},
    {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeAuxStore<\n {DataTypeTag[self.element]},\n {self.stride_mnl}\n>;\n'`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeAuxStore<\n {DataTypeTag[self.element]},\n {self.stride_mnl}\n>;\n'`，用于在函数 `type_decl` 中建立状态。

##### Line 217 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 220-222 — Class `xe20StoreDImpl`

```python
class xe20StoreDImpl(StoreDImpl):

    @property
```
**EN:** Declares class `xe20StoreDImpl` deriving from `StoreDImpl`.
**CN:** 声明类 `xe20StoreDImpl`，其基类为 `StoreDImpl`。

#### Lines 222-226 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 227-230 — Return

```python
        return f"""
using ElementD = {DataTypeTag[self.element]};
using StrideD = {self.stride_mnl};
"""
```
**EN:** Returns `f'\nusing ElementD = {DataTypeTag[self.element]};\nusing StrideD = {self.stride_mnl};\n'` to the caller.
**CN:** 向调用方返回 `f'\nusing ElementD = {DataTypeTag[self.element]};\nusing StrideD = {self.stride_mnl};\n'`。

### Lines 233-234 — Class `xe20ColumnReductionImpl`

```python
class xe20ColumnReductionImpl(ColumnReductionImpl):
```
**EN:** Declares class `xe20ColumnReductionImpl` deriving from `ColumnReductionImpl`.
**CN:** 声明类 `xe20ColumnReductionImpl`，其基类为 `ColumnReductionImpl`。

#### Line 235 — Function `__init__`

```python
    def __init__(self, node) -> None:
```
**EN:** Defines function `__init__` with parameters `self, node`.
**CN:** 定义函数 `__init__`，参数为 `self, node`。

##### Line 236 — Call `super().__init__`

```python
      super().__init__(node)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Lines 237-241 — Conditional `_needs_atomic_float(self.element, self.gmem_reduce_fn)`

```python
      if _needs_atomic_float(self.element, self.gmem_reduce_fn):
            raise RuntimeError(
                f"Xe20 column reduction '{self.name}' uses {DataTypeTag[self.element]} with {op_tag(self.gmem_reduce_fn)}, "
                "which requires a float output because sycl::atomic_ref does not support half/bfloat16. Please declare the reduction tensor as float32."
            )
```
**EN:** Checks `_needs_atomic_float(self.element, self.gmem_reduce_fn)` and selects the matching branch in function `__init__`.
**CN:** 检查 `_needs_atomic_float(self.element, self.gmem_reduce_fn)`，并在函数 `__init__` 中选择匹配的分支。

#### Lines 243-247 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 248-249 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 251-258 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::XeColReduction<
    {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_reduce_fn)}, 0,
    TileShape_MNK, {DataTypeTag[self.element]},
    {DataTypeTag[self.element_compute]}, {FloatRoundStyleTag[self.round_style]},
    {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeColReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_redu...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeColReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_redu...`，用于在函数 `type_decl` 中建立状态。

##### Line 259 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 262-263 — Class `xe20RowReductionImpl`

```python
class xe20RowReductionImpl(RowReductionImpl):
```
**EN:** Declares class `xe20RowReductionImpl` deriving from `RowReductionImpl`.
**CN:** 声明类 `xe20RowReductionImpl`，其基类为 `RowReductionImpl`。

#### Line 264 — Function `__init__`

```python
    def __init__(self, node) -> None:
```
**EN:** Defines function `__init__` with parameters `self, node`.
**CN:** 定义函数 `__init__`，参数为 `self, node`。

##### Line 265 — Call `super().__init__`

```python
        super().__init__(node)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Lines 266-270 — Conditional `_needs_atomic_float(self.element, self.gmem_reduce_fn)`

```python
        if _needs_atomic_float(self.element, self.gmem_reduce_fn):
            raise RuntimeError(
                f"Xe20 row reduction '{self.name}' uses {DataTypeTag[self.element]} with {op_tag(self.gmem_reduce_fn)}, "
                "which requires a float output because sycl::atomic_ref does not support half/bfloat16. Please declare the reduction tensor as float32."
            )
```
**EN:** Checks `_needs_atomic_float(self.element, self.gmem_reduce_fn)` and selects the matching branch in function `__init__`.
**CN:** 检查 `_needs_atomic_float(self.element, self.gmem_reduce_fn)`，并在函数 `__init__` 中选择匹配的分支。

#### Lines 272-276 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 277-278 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 280-287 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::XeRowReduction<
    {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_reduce_fn)}, 0 /* Stages */,
    TileShape_MNK, {DataTypeTag[self.element]},
    {DataTypeTag[self.element_compute]}, {FloatRoundStyleTag[self.round_style]},
    {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeRowReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_redu...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeRowReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_redu...`，用于在函数 `type_decl` 中建立状态。

##### Line 288 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 291-292 — Class `xe20ScalarReductionImpl`

```python
class xe20ScalarReductionImpl(ScalarReductionImpl):
```
**EN:** Declares class `xe20ScalarReductionImpl` deriving from `ScalarReductionImpl`.
**CN:** 声明类 `xe20ScalarReductionImpl`，其基类为 `ScalarReductionImpl`。

#### Line 293 — Function `__init__`

```python
    def __init__(self, node) -> None:
```
**EN:** Defines function `__init__` with parameters `self, node`.
**CN:** 定义函数 `__init__`，参数为 `self, node`。

##### Line 294 — Call `super().__init__`

```python
        super().__init__(node)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Lines 295-299 — Conditional `_needs_atomic_float(self.element, self.gmem_reduce_fn)`

```python
        if _needs_atomic_float(self.element, self.gmem_reduce_fn):
            raise RuntimeError(
                f"Xe20 scalar reduction '{self.name}' uses {DataTypeTag[self.element]} with {op_tag(self.gmem_reduce_fn)}, "
                "which requires a float output because sycl::atomic_ref does not support half/bfloat16. Please declare the reduction tensor as float32."
            )
```
**EN:** Checks `_needs_atomic_float(self.element, self.gmem_reduce_fn)` and selects the matching branch in function `__init__`.
**CN:** 检查 `_needs_atomic_float(self.element, self.gmem_reduce_fn)`，并在函数 `__init__` 中选择匹配的分支。

#### Lines 301-305 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 306-307 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 309-315 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::XeScalarReduction<
    {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_reduce_fn)},
    {DataTypeTag[self.element]}, {DataTypeTag[self.element_compute]},
    {FloatRoundStyleTag[self.round_style]}, {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeScalarReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_reduce_fn)},\n {DataTypeTag[sel...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::XeScalarReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_reduce_fn)},\n {DataTypeTag[sel...`，用于在函数 `type_decl` 中建立状态。

##### Line 316 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `xe20AccumulatorImpl`, `xe20LoadSrcImpl`, `xe20AuxLoadImpl`, `xe20ScalarBroadcastImpl`, `xe20RowBroadcastImpl`, `xe20ColumnBroadcastImpl`, `xe20ComputeImpl`, `xe20AuxStoreImpl`, `xe20StoreDImpl`, `xe20ColumnReductionImpl`, `xe20RowReductionImpl`, `xe20ScalarReductionImpl`.
- **CN:** 顶层类：`xe20AccumulatorImpl`, `xe20LoadSrcImpl`, `xe20AuxLoadImpl`, `xe20ScalarBroadcastImpl`, `xe20RowBroadcastImpl`, `xe20ColumnBroadcastImpl`, `xe20ComputeImpl`, `xe20AuxStoreImpl`, `xe20StoreDImpl`, `xe20ColumnReductionImpl`, `xe20RowReductionImpl`, `xe20ScalarReductionImpl`。
- **EN:** Top-level functions: `_needs_atomic_float`.
- **CN:** 顶层函数：`_needs_atomic_float`。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.evt.ir`, `cutlass_cppgen.backend.library`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** `pycute`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
