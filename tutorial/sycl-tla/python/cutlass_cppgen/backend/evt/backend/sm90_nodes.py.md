# sm90_nodes.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/backend/sm90_nodes.py`
- **EN:** Defines classes `Sm90AccumulatorImpl`, `Sm90LoadSrcImpl`, `Sm90AuxLoadImpl`, `Sm90ScalarBroadcastImpl`, `Sm90RowBroadcastImpl`, `Sm90ColumnBroadcastImpl` for the cutlass_cppgen package.
- **CN:** 为 cutlass_cppgen 包定义类 `Sm90AccumulatorImpl`, `Sm90LoadSrcImpl`, `Sm90AuxLoadImpl`, `Sm90ScalarBroadcastImpl`, `Sm90RowBroadcastImpl`, `Sm90ColumnBroadcastImpl`。

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

### Line 33 — From `pycute` import

```python
from pycute import product
```
**EN:** Imports `product` from `pycute` to reuse shared definitions at module scope.
**CN:** 从 `pycute` 导入 `product`，以便后续代码在模块级复用共享定义。

### Line 35 — From `cutlass_library` import

```python
from cutlass_library import DataTypeSize, DataTypeTag
```
**EN:** Imports `DataTypeSize, DataTypeTag` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `DataTypeSize, DataTypeTag`，以便后续代码在模块级复用共享定义。

### Lines 36-55 — From `cutlass_cppgen.backend.evt.ir` import

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

### Lines 56-60 — From `cutlass_cppgen.backend.library` import

```python
from cutlass_cppgen.backend.library import (
    FloatRoundStyleTag,
    FunctionalOp,
    op_tag,
)
```
**EN:** Imports `FloatRoundStyleTag, FunctionalOp, op_tag` from `cutlass_cppgen.backend.library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.library` 导入 `FloatRoundStyleTag, FunctionalOp, op_tag`，以便后续代码在模块级复用共享定义。

### Lines 63-65 — Class `Sm90AccumulatorImpl`

```python
class Sm90AccumulatorImpl(AccumulatorImpl):

    @property
```
**EN:** Declares class `Sm90AccumulatorImpl` deriving from `AccumulatorImpl`.
**CN:** 声明类 `Sm90AccumulatorImpl`，其基类为 `AccumulatorImpl`。

#### Lines 65-69 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 70-71 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Line 73 — Assign `self._type_decl`

```python
        self._type_decl = f"""\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90AccFetch;\n"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90AccFetch;\n'`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90AccFetch;\n'`，用于在函数 `type_decl` 中建立状态。

##### Line 74 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 77-79 — Class `Sm90LoadSrcImpl`

```python
class Sm90LoadSrcImpl(LoadSrcImpl):

    @property
```
**EN:** Declares class `Sm90LoadSrcImpl` deriving from `LoadSrcImpl`.
**CN:** 声明类 `Sm90LoadSrcImpl`，其基类为 `LoadSrcImpl`。

#### Lines 79-83 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 84-85 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 87-91 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using ElementC = {DataTypeTag[self.element]};
using StrideC = {self.stride_mnl};
using {self.name_camel} = cutlass::epilogue::fusion::Sm90SrcFetch<{DataTypeTag[self.element]}>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing ElementC = {DataTypeTag[self.element]};\nusing StrideC = {self.stride_mnl};\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90SrcFetch<{Dat...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing ElementC = {DataTypeTag[self.element]};\nusing StrideC = {self.stride_mnl};\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90SrcFetch<{Dat...`，用于在函数 `type_decl` 中建立状态。

##### Line 92 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 95-97 — Class `Sm90AuxLoadImpl`

```python
class Sm90AuxLoadImpl(AuxLoadImpl):

    @property
```
**EN:** Declares class `Sm90AuxLoadImpl` deriving from `AuxLoadImpl`.
**CN:** 声明类 `Sm90AuxLoadImpl`，其基类为 `AuxLoadImpl`。

#### Lines 97-101 — Function `descriptor`

```python
    @property
    def descriptor(self) -> str:
        """
        Descriptor for Aux Load
        """
```
**EN:** Defines function `descriptor` with parameters `self`. Purpose: Descriptor for Aux Load
**CN:** 定义函数 `descriptor`，参数为 `self`。 其用途：Descriptor for Aux Load

##### Line 102 — Return

```python
        return f"{self.name_camel}Descriptor"
```
**EN:** Returns `f'{self.name_camel}Descriptor'` to the caller.
**CN:** 向调用方返回 `f'{self.name_camel}Descriptor'`。

#### Lines 104-107 — Function `decl_descriptor`

```python
    def decl_descriptor(self) -> str:
        """
        Declare the descriptor type
        """
```
**EN:** Defines function `decl_descriptor` with parameters `self`. Purpose: Declare the descriptor type
**CN:** 定义函数 `decl_descriptor`，参数为 `self`。 其用途：Declare the descriptor type

##### Line 108 — Return

```python
        return f"\nusing {self.descriptor} = cutlass::epilogue::collective::detail::AuxLoadDescriptor<EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.element]}>;\n"
```
**EN:** Returns `f'\nusing {self.descriptor} = cutlass::epilogue::collective::detail::AuxLoadDescriptor<EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.element]}>;\n'` to the caller.
**CN:** 向调用方返回 `f'\nusing {self.descriptor} = cutlass::epilogue::collective::detail::AuxLoadDescriptor<EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.element]}>;\n'`。

#### Lines 110-114 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 115-116 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Line 118 — Assign `self._type_decl`

```python
        self._type_decl = self.decl_descriptor()
```
**EN:** Assigns `self._type_decl` from `self.decl_descriptor()`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `self.decl_descriptor()`，用于在函数 `type_decl` 中建立状态。

##### Lines 119-124 — Update `self._type_decl`

```python
        self._type_decl += f"""
using {self.name_camel} = cutlass::epilogue::fusion::Sm90AuxLoad<
    {self.descriptor}::Stages, typename {self.descriptor}::EpilogueTile, {DataTypeTag[self.element]},
    {self.stride_mnl}, typename {self.descriptor}::SmemLayoutAtom, typename {self.descriptor}::CopyOpS2R
>;
"""
```
**EN:** Updates `self._type_decl` with `+=` using `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90AuxLoad<\n {self.descriptor}::Stages, typename {self.descriptor}::EpilogueTile, {DataTypeTag[sel...`.
**CN:** 使用 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90AuxLoad<\n {self.descriptor}::Stages, typename {self.descriptor}::EpilogueTile, {DataTypeTag[sel...` 对 `self._type_decl` 执行 `+=` 更新。

##### Line 125 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

#### Lines 127-130 — Function `get_smem_size`

```python
    def get_smem_size(self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles):
        """
        Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d
        """
```
**EN:** Defines function `get_smem_size` with parameters `self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles`. Purpose: Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d
**CN:** 定义函数 `get_smem_size`，参数为 `self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles`。 其用途：Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d

##### Line 131 — Return

```python
        return (DataTypeSize[self.element] * stages_c * product(epilogue_tile_mn) // 8, 128)
```
**EN:** Returns `(DataTypeSize[self.element] * stages_c * product(epilogue_tile_mn) // 8, 128)` to the caller.
**CN:** 向调用方返回 `(DataTypeSize[self.element] * stages_c * product(epilogue_tile_mn) // 8, 128)`。

### Line 134 — Class `Sm90ScalarBroadcastImpl`

```python
class Sm90ScalarBroadcastImpl(ScalarBroadcastImpl):
```
**EN:** Declares class `Sm90ScalarBroadcastImpl` deriving from `ScalarBroadcastImpl`.
**CN:** 声明类 `Sm90ScalarBroadcastImpl`，其基类为 `ScalarBroadcastImpl`。

#### Line 135 — Function `__init__`

```python
    def __init__(self, node: LoadNode) -> None:
```
**EN:** Defines function `__init__` with parameters `self, node`.
**CN:** 定义函数 `__init__`，参数为 `self, node`。

##### Line 136 — Call `super().__init__`

```python
        super().__init__(node)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 137 — Assign `self.broadcast_count`

```python
        self.broadcast_count = 1
```
**EN:** Assigns `self.broadcast_count` from `1`, establishing state in function `__init__`.
**CN:** 将 `self.broadcast_count` 赋值为 `1`，用于在函数 `__init__` 中建立状态。

##### Line 138 — Assign `self.reduction_fn`

```python
        self.reduction_fn = FunctionalOp.Multiplies
```
**EN:** Assigns `self.reduction_fn` from `FunctionalOp.Multiplies`, establishing state in function `__init__`.
**CN:** 将 `self.reduction_fn` 赋值为 `FunctionalOp.Multiplies`，用于在函数 `__init__` 中建立状态。

#### Lines 140-144 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 145-146 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 148-152 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::Sm90ScalarBroadcast<
    {DataTypeTag[self.element]}, {self.stride_mnl}, {self.broadcast_count}, {op_tag(self.reduction_fn)}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90ScalarBroadcast<\n {DataTypeTag[self.element]}, {self.stride_mnl}, {self.broadcast_count}, {op_t...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90ScalarBroadcast<\n {DataTypeTag[self.element]}, {self.stride_mnl}, {self.broadcast_count}, {op_t...`，用于在函数 `type_decl` 中建立状态。

##### Line 153 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 156-157 — Class `Sm90RowBroadcastImpl`

```python
class Sm90RowBroadcastImpl(RowBroadcastImpl):
    @property
```
**EN:** Declares class `Sm90RowBroadcastImpl` deriving from `RowBroadcastImpl`.
**CN:** 声明类 `Sm90RowBroadcastImpl`，其基类为 `RowBroadcastImpl`。

#### Lines 157-161 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 162-163 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 165-170 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::Sm90RowBroadcast<
    0 /*Stages*/, typename EpilogueDescriptor::TileShape, {DataTypeTag[self.element]}, {DataTypeTag[self.element_output]},
    {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90RowBroadcast<\n 0 /*Stages*/, typename EpilogueDescriptor::TileShape, {DataTypeTag[self.element]...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90RowBroadcast<\n 0 /*Stages*/, typename EpilogueDescriptor::TileShape, {DataTypeTag[self.element]...`，用于在函数 `type_decl` 中建立状态。

##### Line 171 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 174-176 — Class `Sm90ColumnBroadcastImpl`

```python
class Sm90ColumnBroadcastImpl(ColumnBroadcastImpl):

    @property
```
**EN:** Declares class `Sm90ColumnBroadcastImpl` deriving from `ColumnBroadcastImpl`.
**CN:** 声明类 `Sm90ColumnBroadcastImpl`，其基类为 `ColumnBroadcastImpl`。

#### Lines 176-180 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 181-182 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 184-189 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::Sm90ColBroadcast<
    0 /*Stages*/, typename EpilogueDescriptor::TileShape, {DataTypeTag[self.element]}, {DataTypeTag[self.element_output]},
    {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90ColBroadcast<\n 0 /*Stages*/, typename EpilogueDescriptor::TileShape, {DataTypeTag[self.element]...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90ColBroadcast<\n 0 /*Stages*/, typename EpilogueDescriptor::TileShape, {DataTypeTag[self.element]...`，用于在函数 `type_decl` 中建立状态。

##### Line 190 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 193-195 — Class `Sm90ComputeImpl`

```python
class Sm90ComputeImpl(ComputeImpl):

    @property
```
**EN:** Declares class `Sm90ComputeImpl` deriving from `ComputeImpl`.
**CN:** 声明类 `Sm90ComputeImpl`，其基类为 `ComputeImpl`。

#### Lines 195-199 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 200-201 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 203-208 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::Sm90Compute<
    {op_tag(self.fn)}, {DataTypeTag[self.element_output]}, {DataTypeTag[self.element_compute]},
    {FloatRoundStyleTag[self.round_style]}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90Compute<\n {op_tag(self.fn)}, {DataTypeTag[self.element_output]}, {DataTypeTag[self.element_comp...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90Compute<\n {op_tag(self.fn)}, {DataTypeTag[self.element_output]}, {DataTypeTag[self.element_comp...`，用于在函数 `type_decl` 中建立状态。

##### Line 209 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 212-214 — Class `Sm90AuxStoreImpl`

```python
class Sm90AuxStoreImpl(AuxStoreImpl):

    @property
```
**EN:** Declares class `Sm90AuxStoreImpl` deriving from `AuxStoreImpl`.
**CN:** 声明类 `Sm90AuxStoreImpl`，其基类为 `AuxStoreImpl`。

#### Lines 214-218 — Function `descriptor`

```python
    @property
    def descriptor(self) -> str:
        """
        Descriptor for Aux Load
        """
```
**EN:** Defines function `descriptor` with parameters `self`. Purpose: Descriptor for Aux Load
**CN:** 定义函数 `descriptor`，参数为 `self`。 其用途：Descriptor for Aux Load

##### Line 219 — Return

```python
        return f"{self.name_camel}Descriptor"
```
**EN:** Returns `f'{self.name_camel}Descriptor'` to the caller.
**CN:** 向调用方返回 `f'{self.name_camel}Descriptor'`。

#### Lines 221-224 — Function `decl_descriptor`

```python
    def decl_descriptor(self) -> str:
        """
        Declare the descriptor type
        """
```
**EN:** Defines function `decl_descriptor` with parameters `self`. Purpose: Declare the descriptor type
**CN:** 定义函数 `decl_descriptor`，参数为 `self`。 其用途：Declare the descriptor type

##### Lines 225-229 — Return

```python
        return f"""
using {self.descriptor} = cutlass::epilogue::collective::detail::AuxStoreDescriptor<
    EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.element]}
>;
"""
```
**EN:** Returns `f'\nusing {self.descriptor} = cutlass::epilogue::collective::detail::AuxStoreDescriptor<\n EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.element]}...` to the caller.
**CN:** 向调用方返回 `f'\nusing {self.descriptor} = cutlass::epilogue::collective::detail::AuxStoreDescriptor<\n EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.element]}...`。

#### Lines 230-234 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 235-236 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Line 238 — Assign `self._type_decl`

```python
        self._type_decl = self.decl_descriptor()
```
**EN:** Assigns `self._type_decl` from `self.decl_descriptor()`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `self.decl_descriptor()`，用于在函数 `type_decl` 中建立状态。

##### Lines 239-245 — Update `self._type_decl`

```python
        self._type_decl += f"""
using {self.name_camel} = cutlass::epilogue::fusion::Sm90AuxStore<
    {self.descriptor}::Stages, typename {self.descriptor}::EpilogueTile, {DataTypeTag[self.element]},
    {FloatRoundStyleTag[self.round_style]}, {self.stride_mnl}, typename {self.descriptor}::SmemLayoutAtom,
    typename {self.descriptor}::CopyOpR2S
>;
"""
```
**EN:** Updates `self._type_decl` with `+=` using `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90AuxStore<\n {self.descriptor}::Stages, typename {self.descriptor}::EpilogueTile, {DataTypeTag[se...`.
**CN:** 使用 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90AuxStore<\n {self.descriptor}::Stages, typename {self.descriptor}::EpilogueTile, {DataTypeTag[se...` 对 `self._type_decl` 执行 `+=` 更新。

##### Line 246 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

#### Lines 248-251 — Function `get_smem_size`

```python
    def get_smem_size(self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles):
        """
        Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d
        """
```
**EN:** Defines function `get_smem_size` with parameters `self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles`. Purpose: Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d
**CN:** 定义函数 `get_smem_size`，参数为 `self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles`。 其用途：Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d

##### Line 252 — Return

```python
        return (DataTypeSize[self.element] * stages_d * product(epilogue_tile_mn) // 8, 128)
```
**EN:** Returns `(DataTypeSize[self.element] * stages_d * product(epilogue_tile_mn) // 8, 128)` to the caller.
**CN:** 向调用方返回 `(DataTypeSize[self.element] * stages_d * product(epilogue_tile_mn) // 8, 128)`。

### Lines 255-257 — Class `Sm90StoreDImpl`

```python
class Sm90StoreDImpl(StoreDImpl):

    @property
```
**EN:** Declares class `Sm90StoreDImpl` deriving from `StoreDImpl`.
**CN:** 声明类 `Sm90StoreDImpl`，其基类为 `StoreDImpl`。

#### Lines 257-261 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 262-265 — Return

```python
        return f"""
using ElementD = {DataTypeTag[self.element]};
using StrideD = {self.stride_mnl};
"""
```
**EN:** Returns `f'\nusing ElementD = {DataTypeTag[self.element]};\nusing StrideD = {self.stride_mnl};\n'` to the caller.
**CN:** 向调用方返回 `f'\nusing ElementD = {DataTypeTag[self.element]};\nusing StrideD = {self.stride_mnl};\n'`。

### Lines 268-270 — Class `Sm90ColumnReductionImpl`

```python
class Sm90ColumnReductionImpl(ColumnReductionImpl):

    @property
```
**EN:** Declares class `Sm90ColumnReductionImpl` deriving from `ColumnReductionImpl`.
**CN:** 声明类 `Sm90ColumnReductionImpl`，其基类为 `ColumnReductionImpl`。

#### Lines 270-274 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 275-276 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 278-285 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::Sm90ColReduction<
    {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_reduce_fn)}, 0,
    typename EpilogueDescriptor::TileShape, {DataTypeTag[self.element]},
    {DataTypeTag[self.element_compute]}, {FloatRoundStyleTag[self.round_style]},
    {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90ColReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_re...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90ColReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_re...`，用于在函数 `type_decl` 中建立状态。

##### Line 286 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 289-292 — Class `Sm90RowReductionImpl`

```python
class Sm90RowReductionImpl(RowReductionImpl):


    @property
```
**EN:** Declares class `Sm90RowReductionImpl` deriving from `RowReductionImpl`.
**CN:** 声明类 `Sm90RowReductionImpl`，其基类为 `RowReductionImpl`。

#### Lines 292-296 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 297-298 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 300-307 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::Sm90RowReduction<
    {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_reduce_fn)}, 0 /* Stages */,
    typename EpilogueDescriptor::TileShape, {DataTypeTag[self.element]},
    {DataTypeTag[self.element_compute]}, {FloatRoundStyleTag[self.round_style]},
    {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90RowReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_re...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90RowReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_re...`，用于在函数 `type_decl` 中建立状态。

##### Line 308 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

### Lines 311-314 — Class `Sm90ScalarReductionImpl`

```python
class Sm90ScalarReductionImpl(ScalarReductionImpl):


    @property
```
**EN:** Declares class `Sm90ScalarReductionImpl` deriving from `ScalarReductionImpl`.
**CN:** 声明类 `Sm90ScalarReductionImpl`，其基类为 `ScalarReductionImpl`。

#### Lines 314-318 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 319-320 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Lines 322-328 — Assign `self._type_decl`

```python
        self._type_decl = f"""
using {self.name_camel} = cutlass::epilogue::fusion::Sm90ScalarReduction<
    {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_reduce_fn)},
    {DataTypeTag[self.element]}, {DataTypeTag[self.element_compute]},
    {FloatRoundStyleTag[self.round_style]}, {self.stride_mnl}
>;
"""
```
**EN:** Assigns `self._type_decl` from `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90ScalarReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_reduce_fn)},\n {DataTypeTag[s...`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `f'\nusing {self.name_camel} = cutlass::epilogue::fusion::Sm90ScalarReduction<\n {op_tag(self.reg_reduce_fn)}, {op_tag(self.gmem_reduce_fn)},\n {DataTypeTag[s...`，用于在函数 `type_decl` 中建立状态。

##### Line 329 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `Sm90AccumulatorImpl`, `Sm90LoadSrcImpl`, `Sm90AuxLoadImpl`, `Sm90ScalarBroadcastImpl`, `Sm90RowBroadcastImpl`, `Sm90ColumnBroadcastImpl`, `Sm90ComputeImpl`, `Sm90AuxStoreImpl`, `Sm90StoreDImpl`, `Sm90ColumnReductionImpl`, `Sm90RowReductionImpl`, `Sm90ScalarReductionImpl`.
- **CN:** 顶层类：`Sm90AccumulatorImpl`, `Sm90LoadSrcImpl`, `Sm90AuxLoadImpl`, `Sm90ScalarBroadcastImpl`, `Sm90RowBroadcastImpl`, `Sm90ColumnBroadcastImpl`, `Sm90ComputeImpl`, `Sm90AuxStoreImpl`, `Sm90StoreDImpl`, `Sm90ColumnReductionImpl`, `Sm90RowReductionImpl`, `Sm90ScalarReductionImpl`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.evt.ir`, `cutlass_cppgen.backend.library`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** `pycute`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
