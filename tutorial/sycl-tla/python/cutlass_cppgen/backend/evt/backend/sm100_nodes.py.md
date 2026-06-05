# sm100_nodes.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/backend/sm100_nodes.py`
- **EN:** Defines classes `Sm100AuxLoadImpl`, `Sm100AuxStoreImpl` for the cutlass_cppgen package.
- **CN:** 为 cutlass_cppgen 包定义类 `Sm100AuxLoadImpl`, `Sm100AuxStoreImpl`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32 — File header

```python
#################################################################################################
#
# Copyright (c) 2025 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Line 37 — From `cutlass_cppgen.backend.evt.ir` import

```python
from cutlass_cppgen.backend.evt.ir import AuxLoadImpl, AuxStoreImpl
```
**EN:** Imports `AuxLoadImpl, AuxStoreImpl` from `cutlass_cppgen.backend.evt.ir` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir` 导入 `AuxLoadImpl, AuxStoreImpl`，以便后续代码在模块级复用共享定义。

### Line 38 — Import `cutlass_cppgen.backend.evt.backend.sm90_nodes as sm90_nodes`

```python
import cutlass_cppgen.backend.evt.backend.sm90_nodes as sm90_nodes
```
**EN:** Imports `cutlass_cppgen.backend.evt.backend.sm90_nodes as sm90_nodes` so later code can use these APIs at module scope.
**CN:** 导入 `cutlass_cppgen.backend.evt.backend.sm90_nodes as sm90_nodes`，供后续代码在模块级使用这些 API。

### Line 40 — From `cutlass_cppgen.backend.library` import

```python
from cutlass_cppgen.backend.library import FloatRoundStyleTag
```
**EN:** Imports `FloatRoundStyleTag` from `cutlass_cppgen.backend.library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.library` 导入 `FloatRoundStyleTag`，以便后续代码在模块级复用共享定义。

### Line 43 — Assign `Sm100AccumulatorImpl`

```python
Sm100AccumulatorImpl = sm90_nodes.Sm90AccumulatorImpl
```
**EN:** Assigns `Sm100AccumulatorImpl` from `sm90_nodes.Sm90AccumulatorImpl`, establishing state at module scope.
**CN:** 将 `Sm100AccumulatorImpl` 赋值为 `sm90_nodes.Sm90AccumulatorImpl`，用于在模块级建立状态。

### Line 44 — Assign `Sm100LoadSrcImpl`

```python
Sm100LoadSrcImpl = sm90_nodes.Sm90LoadSrcImpl
```
**EN:** Assigns `Sm100LoadSrcImpl` from `sm90_nodes.Sm90LoadSrcImpl`, establishing state at module scope.
**CN:** 将 `Sm100LoadSrcImpl` 赋值为 `sm90_nodes.Sm90LoadSrcImpl`，用于在模块级建立状态。

### Line 45 — Assign `Sm100ScalarBroadcastImpl`

```python
Sm100ScalarBroadcastImpl = sm90_nodes.Sm90ScalarBroadcastImpl
```
**EN:** Assigns `Sm100ScalarBroadcastImpl` from `sm90_nodes.Sm90ScalarBroadcastImpl`, establishing state at module scope.
**CN:** 将 `Sm100ScalarBroadcastImpl` 赋值为 `sm90_nodes.Sm90ScalarBroadcastImpl`，用于在模块级建立状态。

### Line 46 — Assign `Sm100RowBroadcastImpl`

```python
Sm100RowBroadcastImpl = sm90_nodes.Sm90RowBroadcastImpl
```
**EN:** Assigns `Sm100RowBroadcastImpl` from `sm90_nodes.Sm90RowBroadcastImpl`, establishing state at module scope.
**CN:** 将 `Sm100RowBroadcastImpl` 赋值为 `sm90_nodes.Sm90RowBroadcastImpl`，用于在模块级建立状态。

### Line 47 — Assign `Sm100ColumnBroadcastImpl`

```python
Sm100ColumnBroadcastImpl = sm90_nodes.Sm90ColumnBroadcastImpl
```
**EN:** Assigns `Sm100ColumnBroadcastImpl` from `sm90_nodes.Sm90ColumnBroadcastImpl`, establishing state at module scope.
**CN:** 将 `Sm100ColumnBroadcastImpl` 赋值为 `sm90_nodes.Sm90ColumnBroadcastImpl`，用于在模块级建立状态。

### Line 48 — Assign `Sm100ComputeImpl`

```python
Sm100ComputeImpl = sm90_nodes.Sm90ComputeImpl
```
**EN:** Assigns `Sm100ComputeImpl` from `sm90_nodes.Sm90ComputeImpl`, establishing state at module scope.
**CN:** 将 `Sm100ComputeImpl` 赋值为 `sm90_nodes.Sm90ComputeImpl`，用于在模块级建立状态。

### Line 49 — Assign `Sm100StoreDImpl`

```python
Sm100StoreDImpl = sm90_nodes.Sm90StoreDImpl
```
**EN:** Assigns `Sm100StoreDImpl` from `sm90_nodes.Sm90StoreDImpl`, establishing state at module scope.
**CN:** 将 `Sm100StoreDImpl` 赋值为 `sm90_nodes.Sm90StoreDImpl`，用于在模块级建立状态。

### Line 50 — Assign `Sm100ColumnReductionImpl`

```python
Sm100ColumnReductionImpl = sm90_nodes.Sm90ColumnReductionImpl
```
**EN:** Assigns `Sm100ColumnReductionImpl` from `sm90_nodes.Sm90ColumnReductionImpl`, establishing state at module scope.
**CN:** 将 `Sm100ColumnReductionImpl` 赋值为 `sm90_nodes.Sm90ColumnReductionImpl`，用于在模块级建立状态。

### Line 51 — Assign `Sm100RowReductionImpl`

```python
Sm100RowReductionImpl = sm90_nodes.Sm90RowReductionImpl
```
**EN:** Assigns `Sm100RowReductionImpl` from `sm90_nodes.Sm90RowReductionImpl`, establishing state at module scope.
**CN:** 将 `Sm100RowReductionImpl` 赋值为 `sm90_nodes.Sm90RowReductionImpl`，用于在模块级建立状态。

### Line 52 — Assign `Sm100ScalarReductionImpl`

```python
Sm100ScalarReductionImpl = sm90_nodes.Sm90ScalarReductionImpl
```
**EN:** Assigns `Sm100ScalarReductionImpl` from `sm90_nodes.Sm90ScalarReductionImpl`, establishing state at module scope.
**CN:** 将 `Sm100ScalarReductionImpl` 赋值为 `sm90_nodes.Sm90ScalarReductionImpl`，用于在模块级建立状态。

### Lines 55-57 — Class `Sm100AuxLoadImpl`

```python
class Sm100AuxLoadImpl(AuxLoadImpl):

    @property
```
**EN:** Declares class `Sm100AuxLoadImpl` deriving from `AuxLoadImpl`.
**CN:** 声明类 `Sm100AuxLoadImpl`，其基类为 `AuxLoadImpl`。

#### Lines 57-61 — Function `descriptor`

```python
    @property
    def descriptor(self) -> str:
        """
        Descriptor for Aux Load
        """
```
**EN:** Defines function `descriptor` with parameters `self`. Purpose: Descriptor for Aux Load
**CN:** 定义函数 `descriptor`，参数为 `self`。 其用途：Descriptor for Aux Load

##### Line 62 — Return

```python
        return f"{self.name_camel}Descriptor"
```
**EN:** Returns `f'{self.name_camel}Descriptor'` to the caller.
**CN:** 向调用方返回 `f'{self.name_camel}Descriptor'`。

#### Lines 64-67 — Function `decl_descriptor`

```python
    def decl_descriptor(self) -> str:
        """
        Declare the descriptor type
        """
```
**EN:** Defines function `decl_descriptor` with parameters `self`. Purpose: Declare the descriptor type
**CN:** 定义函数 `decl_descriptor`，参数为 `self`。 其用途：Declare the descriptor type

##### Line 68 — Return

```python
        return f"\nusing {self.descriptor} = cutlass::epilogue::collective::detail::Sm100AuxLoadDescriptor<EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.element]}>;\n"
```
**EN:** Returns `f'\nusing {self.descriptor} = cutlass::epilogue::collective::detail::Sm100AuxLoadDescriptor<EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.element]...` to the caller.
**CN:** 向调用方返回 `f'\nusing {self.descriptor} = cutlass::epilogue::collective::detail::Sm100AuxLoadDescriptor<EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.element]...`。

#### Lines 70-74 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 75-76 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Line 78 — Assign `self._type_decl`

```python
        self._type_decl = self.decl_descriptor()
```
**EN:** Assigns `self._type_decl` from `self.decl_descriptor()`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `self.decl_descriptor()`，用于在函数 `type_decl` 中建立状态。

##### Lines 79-84 — Update `self._type_decl`

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

##### Line 85 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

#### Lines 87-90 — Function `get_smem_size`

```python
    def get_smem_size(self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles):
        """
        Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d
        """
```
**EN:** Defines function `get_smem_size` with parameters `self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles`. Purpose: Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d
**CN:** 定义函数 `get_smem_size`，参数为 `self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles`。 其用途：Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d

##### Line 91 — Return

```python
        return (DataTypeSize[self.element] * stages_c * product(epilogue_tile_mn) // 8, 128)
```
**EN:** Returns `(DataTypeSize[self.element] * stages_c * product(epilogue_tile_mn) // 8, 128)` to the caller.
**CN:** 向调用方返回 `(DataTypeSize[self.element] * stages_c * product(epilogue_tile_mn) // 8, 128)`。

### Lines 94-96 — Class `Sm100AuxStoreImpl`

```python
class Sm100AuxStoreImpl(AuxStoreImpl):

    @property
```
**EN:** Declares class `Sm100AuxStoreImpl` deriving from `AuxStoreImpl`.
**CN:** 声明类 `Sm100AuxStoreImpl`，其基类为 `AuxStoreImpl`。

#### Lines 96-100 — Function `descriptor`

```python
    @property
    def descriptor(self) -> str:
        """
        Descriptor for Aux Load
        """
```
**EN:** Defines function `descriptor` with parameters `self`. Purpose: Descriptor for Aux Load
**CN:** 定义函数 `descriptor`，参数为 `self`。 其用途：Descriptor for Aux Load

##### Line 101 — Return

```python
        return f"{self.name_camel}Descriptor"
```
**EN:** Returns `f'{self.name_camel}Descriptor'` to the caller.
**CN:** 向调用方返回 `f'{self.name_camel}Descriptor'`。

#### Lines 103-106 — Function `decl_descriptor`

```python
    def decl_descriptor(self) -> str:
        """
        Declare the descriptor type
        """
```
**EN:** Defines function `decl_descriptor` with parameters `self`. Purpose: Declare the descriptor type
**CN:** 定义函数 `decl_descriptor`，参数为 `self`。 其用途：Declare the descriptor type

##### Lines 107-111 — Return

```python
        return f"""
using {self.descriptor} = cutlass::epilogue::collective::detail::Sm100AuxStoreDescriptor<
    EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.element]}
>;
"""
```
**EN:** Returns `f'\nusing {self.descriptor} = cutlass::epilogue::collective::detail::Sm100AuxStoreDescriptor<\n EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.elem...` to the caller.
**CN:** 向调用方返回 `f'\nusing {self.descriptor} = cutlass::epilogue::collective::detail::Sm100AuxStoreDescriptor<\n EpilogueDescriptor, {self.stride_mnl}, {DataTypeTag[self.elem...`。

#### Lines 112-116 — Function `type_decl`

```python
    @property
    def type_decl(self):
        """
        Return the string defining the type
        """
```
**EN:** Defines function `type_decl` with parameters `self`. Purpose: Return the string defining the type
**CN:** 定义函数 `type_decl`，参数为 `self`。 其用途：Return the string defining the type

##### Lines 117-118 — Conditional `self._type_decl is not None`

```python
        if self._type_decl is not None:
            return self._type_decl
```
**EN:** Checks `self._type_decl is not None` and selects the matching branch in function `type_decl`.
**CN:** 检查 `self._type_decl is not None`，并在函数 `type_decl` 中选择匹配的分支。

##### Line 120 — Assign `self._type_decl`

```python
        self._type_decl = self.decl_descriptor()
```
**EN:** Assigns `self._type_decl` from `self.decl_descriptor()`, establishing state in function `type_decl`.
**CN:** 将 `self._type_decl` 赋值为 `self.decl_descriptor()`，用于在函数 `type_decl` 中建立状态。

##### Lines 121-127 — Update `self._type_decl`

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

##### Line 128 — Return

```python
        return self._type_decl
```
**EN:** Returns `self._type_decl` to the caller.
**CN:** 向调用方返回 `self._type_decl`。

#### Lines 130-133 — Function `get_smem_size`

```python
    def get_smem_size(self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles):
        """
        Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d
        """
```
**EN:** Defines function `get_smem_size` with parameters `self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles`. Purpose: Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d
**CN:** 定义函数 `get_smem_size`，参数为 `self, cta_tile_mnk, epilogue_tile_mn, stages_c, stages_d, epi_tiles`。 其用途：Get the shared memory size based on epilogue_tile_mn, stages_c, and stages_d

##### Line 134 — Return

```python
        return (DataTypeSize[self.element] * stages_d * product(epilogue_tile_mn) // 8, 128)
```
**EN:** Returns `(DataTypeSize[self.element] * stages_d * product(epilogue_tile_mn) // 8, 128)` to the caller.
**CN:** 向调用方返回 `(DataTypeSize[self.element] * stages_d * product(epilogue_tile_mn) // 8, 128)`。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `Sm100AuxLoadImpl`, `Sm100AuxStoreImpl`.
- **CN:** 顶层类：`Sm100AuxLoadImpl`, `Sm100AuxStoreImpl`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.evt.backend.sm90_nodes`, `cutlass_cppgen.backend.evt.ir`, `cutlass_cppgen.backend.library`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** `pycute`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
