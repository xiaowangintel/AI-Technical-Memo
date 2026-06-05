# gemm_operation.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/gemm_operation.py`
- **EN:** Defines classes `GemmArguments2x`, `GemmArguments2xStreamK`, `GemmArguments3x`, `GemmGroupedArguments`, `GemmRTbase`, `GemmRTUniversal` and functions `leading_dimension`, `transpose_layout`, `GemmArguments` for the cutlass_cppgen package.
- **CN:** 为 cutlass_cppgen 包定义类 `GemmArguments2x`, `GemmArguments2xStreamK`, `GemmArguments3x`, `GemmGroupedArguments`, `GemmRTbase`, `GemmRTUniversal`和函数 `leading_dimension`, `transpose_layout`, `GemmArguments`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31 — File header

```python
#################################################################################################
#
# Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### Line 32 — From `__future__` import

```python
from __future__ import annotations
```
**EN:** Imports `annotations` from `__future__` to reuse shared definitions at module scope.
**CN:** 从 `__future__` 导入 `annotations`，以便后续代码在模块级复用共享定义。

### Line 34 — Import `copy`

```python
import copy
```
**EN:** Imports `copy` so later code can use these APIs at module scope.
**CN:** 导入 `copy`，供后续代码在模块级使用这些 API。

### Line 35 — Import `ctypes`

```python
import ctypes
```
**EN:** Imports `ctypes` so later code can use these APIs at module scope.
**CN:** 导入 `ctypes`，供后续代码在模块级使用这些 API。

### Line 36 — Import `enum`

```python
import enum
```
**EN:** Imports `enum` so later code can use these APIs at module scope.
**CN:** 导入 `enum`，供后续代码在模块级使用这些 API。

### Line 38 — From `cutlass_cppgen.utils.lazy_import` import

```python
from cutlass_cppgen.utils.lazy_import import lazy_import
```
**EN:** Imports `lazy_import` from `cutlass_cppgen.utils.lazy_import` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.utils.lazy_import` 导入 `lazy_import`，以便后续代码在模块级复用共享定义。

### Line 39 — Assign `cuda`

```python
cuda = lazy_import("cuda.cuda")
```
**EN:** Assigns `cuda` from `lazy_import('cuda.cuda')`, establishing state at module scope.
**CN:** 将 `cuda` 赋值为 `lazy_import('cuda.cuda')`，用于在模块级建立状态。

### Line 40 — Assign `cudart`

```python
cudart = lazy_import("cuda.cudart")
```
**EN:** Assigns `cudart` from `lazy_import('cuda.cudart')`, establishing state at module scope.
**CN:** 将 `cudart` 赋值为 `lazy_import('cuda.cudart')`，用于在模块级建立状态。

### Line 41 — Assign `dpctl`

```python
dpctl = lazy_import("dpctl")
```
**EN:** Assigns `dpctl` from `lazy_import('dpctl')`, establishing state at module scope.
**CN:** 将 `dpctl` 赋值为 `lazy_import('dpctl')`，用于在模块级建立状态。

### Line 42 — From `cutlass_library` import

```python
from cutlass_library import SubstituteTemplate
```
**EN:** Imports `SubstituteTemplate` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `SubstituteTemplate`，以便后续代码在模块级复用共享定义。

### Line 43 — From `cutlass_library.arch_constants` import

```python
from cutlass_library.arch_constants import is_intel_xe_arch
```
**EN:** Imports `is_intel_xe_arch` from `cutlass_library.arch_constants` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library.arch_constants` 导入 `is_intel_xe_arch`，以便后续代码在模块级复用共享定义。

### Line 44 — Import `numpy as np`

```python
import numpy as np
```
**EN:** Imports `numpy as np` so later code can use these APIs at module scope.
**CN:** 导入 `numpy as np`，供后续代码在模块级使用这些 API。

### Lines 47-79 — From `cutlass_library` import

```python
from cutlass_library import (
    ComplexTransformTag,
    DataType,
    DataTypeNames,
    DataTypeSize,
    DataTypeTag,
    EpilogueScheduleSuffixes,
    EpilogueScheduleTag,
    EpilogueScheduleType,
    GemmKind,
    GemmKindNames,
    GemmUniversalMode,
    KernelScheduleSuffixes,
    KernelScheduleTag,
    KernelScheduleType,
    LayoutTag,
    LayoutType,
    MathOperation,
    MathOperationTag,
    OpcodeClass,
    OpcodeClassNames,
    OpcodeClassTag,
    OperationKind,
    ShortComplexLayoutNames,
    ShortDataTypeNames,
    ShortLayoutTypeNames,
    SwizzlingFunctor,
    SwizzlingFunctorTag,
    TileSchedulerSuffixes,
    TileSchedulerTag,
    TileSchedulerType,
    get_complex_from_real
)
```
**EN:** Imports `ComplexTransformTag, DataType, DataTypeNames, DataTypeSize, DataTypeTag, EpilogueScheduleSuffixes, EpilogueScheduleTag, EpilogueScheduleType, GemmKind, GemmKindNames, GemmUniversalMode, KernelScheduleSuffixes, KernelScheduleTag, KernelScheduleType, LayoutTag, LayoutType, MathOperation, MathOperationTag, OpcodeClass, OpcodeClassNames, OpcodeClassTag, OperationKind, ShortComplexLayoutNames, ShortDataTypeNames, ShortLayoutTypeNames, SwizzlingFunctor, SwizzlingFunctorTag, TileSchedulerSuffixes, TileSchedulerTag, TileSchedulerType, get_complex_from_real` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `ComplexTransformTag, DataType, DataTypeNames, DataTypeSize, DataTypeTag, EpilogueScheduleSuffixes, EpilogueScheduleTag, EpilogueScheduleType, GemmKind, GemmKindNames, GemmUniversalMode, KernelScheduleSuffixes, KernelScheduleTag, KernelScheduleType, LayoutTag, LayoutType, MathOperation, MathOperationTag, OpcodeClass, OpcodeClassNames, OpcodeClassTag, OperationKind, ShortComplexLayoutNames, ShortDataTypeNames, ShortLayoutTypeNames, SwizzlingFunctor, SwizzlingFunctorTag, TileSchedulerSuffixes, TileSchedulerTag, TileSchedulerType, get_complex_from_real`，以便后续代码在模块级复用共享定义。

### Line 80 — From `cutlass_cppgen.backend.arguments` import

```python
from cutlass_cppgen.backend.arguments import ArgumentBase
```
**EN:** Imports `ArgumentBase` from `cutlass_cppgen.backend.arguments` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.arguments` 导入 `ArgumentBase`，以便后续代码在模块级复用共享定义。

### Lines 81-93 — From `cutlass_cppgen.backend.c_types` import

```python
from cutlass_cppgen.backend.c_types import (
    GemmCoord_,
    GemmCoordBatched_,
    GenericMainloopArguments3x_,
    StrideBatched_,
    dim3_,
    get_gemm_arguments,
    get_gemm_arguments_3x,
    get_gemm_arguments_streamk,
    get_gemm_grouped_arguments,
    get_mainloop_arguments_3x,
    get_tile_scheduler_arguments_3x,
)
```
**EN:** Imports `GemmCoord_, GemmCoordBatched_, GenericMainloopArguments3x_, StrideBatched_, dim3_, get_gemm_arguments, get_gemm_arguments_3x, get_gemm_arguments_streamk, get_gemm_grouped_arguments, get_mainloop_arguments_3x, get_tile_scheduler_arguments_3x` from `cutlass_cppgen.backend.c_types` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.c_types` 导入 `GemmCoord_, GemmCoordBatched_, GenericMainloopArguments3x_, StrideBatched_, dim3_, get_gemm_arguments, get_gemm_arguments_3x, get_gemm_arguments_streamk, get_gemm_grouped_arguments, get_mainloop_arguments_3x, get_tile_scheduler_arguments_3x`，以便后续代码在模块级复用共享定义。

### Lines 94-102 — From `cutlass_cppgen.backend.library` import

```python
from cutlass_cppgen.backend.library import (
    ApiVersion,
    EmissionType,
    SchedulerMode,
    SchedulerModeTag,
    TensorDescription,
    TileDescription,
    api_version,
)
```
**EN:** Imports `ApiVersion, EmissionType, SchedulerMode, SchedulerModeTag, TensorDescription, TileDescription, api_version` from `cutlass_cppgen.backend.library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.library` 导入 `ApiVersion, EmissionType, SchedulerMode, SchedulerModeTag, TensorDescription, TileDescription, api_version`，以便后续代码在模块级复用共享定义。

### Line 103 — From `cutlass_cppgen.backend.memory_manager` import

```python
from cutlass_cppgen.backend.memory_manager import device_mem_alloc, todevice
```
**EN:** Imports `device_mem_alloc, todevice` from `cutlass_cppgen.backend.memory_manager` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.memory_manager` 导入 `device_mem_alloc, todevice`，以便后续代码在模块级复用共享定义。

### Line 104 — From `cutlass_cppgen.backend.operation` import

```python
from cutlass_cppgen.backend.operation import ExecutableOperation, LaunchConfiguration
```
**EN:** Imports `ExecutableOperation, LaunchConfiguration` from `cutlass_cppgen.backend.operation` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.operation` 导入 `ExecutableOperation, LaunchConfiguration`，以便后续代码在模块级复用共享定义。

### Line 105 — From `cutlass_cppgen.backend.type_hint` import

```python
from cutlass_cppgen.backend.type_hint import GemmOperation, Tensor
```
**EN:** Imports `GemmOperation, Tensor` from `cutlass_cppgen.backend.type_hint` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.type_hint` 导入 `GemmOperation, Tensor`，以便后续代码在模块级复用共享定义。

### Line 106 — From `cutlass_cppgen.backend.utils.device` import

```python
from cutlass_cppgen.backend.utils.device import device_sm_count
```
**EN:** Imports `device_sm_count` from `cutlass_cppgen.backend.utils.device` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.utils.device` 导入 `device_sm_count`，以便后续代码在模块级复用共享定义。

### Line 107 — From `cutlass_cppgen.shape` import

```python
from cutlass_cppgen.shape import GemmCoord, MatrixCoord
```
**EN:** Imports `GemmCoord, MatrixCoord` from `cutlass_cppgen.shape` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.shape` 导入 `GemmCoord, MatrixCoord`，以便后续代码在模块级复用共享定义。

### Lines 110-114 — Comment or spacing block

```python
################################################################################
#
# Data structure modeling a GEMM operation
#
################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Lines 117-128 — Function `leading_dimension`

```python
def leading_dimension(layout: LayoutType, shape: MatrixCoord) -> int:
    """
    Returns the leading dimenson of a tensor with layout ``layout`` and shape ``shape``.

    :param layout: layout of the tensor
    :type layout: cutlass_cppgen.shape.LayoutType
    :param shape: shape of the tensor
    :type shape: cutlass_cppgen.shape.MatrixCoord

    :return: leading dimension of the tensor
    :rtype: int
    """
```
**EN:** Defines function `leading_dimension` with parameters `layout, shape`. Purpose: Returns the leading dimenson of a tensor with layout ``layout`` and shape ``shape``.
**CN:** 定义函数 `leading_dimension`，参数为 `layout, shape`。 其用途：Returns the leading dimenson of a tensor with layout ``layout`` and shape ``shape``.

#### Lines 129-132 — Conditional `layout == LayoutType.RowMajor`

```python
    if layout == LayoutType.RowMajor:
        return shape.column
    elif layout == LayoutType.ColumnMajor:
        return shape.row
```
**EN:** Checks `layout == LayoutType.RowMajor` and selects the matching branch in function `leading_dimension`.
**CN:** 检查 `layout == LayoutType.RowMajor`，并在函数 `leading_dimension` 中选择匹配的分支。

### Line 135 — Function `transpose_layout`

```python
def transpose_layout(layout: LayoutType) -> LayoutType:
```
**EN:** Defines function `transpose_layout` with parameters `layout`.
**CN:** 定义函数 `transpose_layout`，参数为 `layout`。

#### Lines 136-141 — Conditional `layout == LayoutType.ColumnMajor`

```python
    if layout == LayoutType.ColumnMajor:
        return LayoutType.RowMajor
    elif layout == LayoutType.RowMajor:
        return LayoutType.ColumnMajor
    else:
        raise ValueError(f"Unsupported Layout {layout}")
```
**EN:** Checks `layout == LayoutType.ColumnMajor` and selects the matching branch in function `transpose_layout`.
**CN:** 检查 `layout == LayoutType.ColumnMajor`，并在函数 `transpose_layout` 中选择匹配的分支。

### Lines 144-176 — Class `GemmArguments2x`

```python
class GemmArguments2x(ArgumentBase):
    """
    Argument wrapper for GEMM in CUTLASS 2. It encodes problem information and
    user-provide tensors into the kernel's argument

    :param operation: the GEMM operation to take the argument
    :type operation: :class:`cutlass_cppgen.backend.GemmOperationUniversal` |
     :class:`cutlass_cppgen.backend.GemmOperationGrouped`

    :param problem_size: GEMM problem size gemm(M, N, K)
    :type operation: :class:`cutlass_cppgen.shape.GemmCoord`

    :param A: tensor A
    :type A: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param B: tensor B
    :type B: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param C: tensor C
    :type C: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param D: tensor D
    :type D: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param gemm_mode: GEMM mode
    :type gemm_mode: :class:`cutlass_library.GemmUniversalMode`

    :param output_op: output operator, optional
    :type output_op: :class:`cutlass_cppgen.backend.LinearCombinationFunctorArguments`

    :param stream: cuda stream, defaults to cuda.cuda.CUstream(0)
    :type stream: :class:`cuda.cuda.CUstream`
    """
```
**EN:** Declares class `GemmArguments2x` deriving from `ArgumentBase`. Purpose: Argument wrapper for GEMM in CUTLASS 2.
**CN:** 声明类 `GemmArguments2x`，其基类为 `ArgumentBase`。 其用途：Argument wrapper for GEMM in CUTLASS 2.

#### Line 178 — Function `__init__`

```python
    def __init__(self, operation, problem_size, A, B, C, D, gemm_mode=GemmUniversalMode.Gemm, **kwargs):
```
**EN:** Defines function `__init__` with parameters `self, operation, problem_size, A, B, C, D, gemm_mode, **kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, operation, problem_size, A, B, C, D, gemm_mode, **kwargs`。

##### Line 179 — Assign `self.operation`

```python
        self.operation = operation
```
**EN:** Assigns `self.operation` from `operation`, establishing state in function `__init__`.
**CN:** 将 `self.operation` 赋值为 `operation`，用于在函数 `__init__` 中建立状态。

##### Line 181 — Assign `self.layout_A`

```python
        self.layout_A = operation.A.layout
```
**EN:** Assigns `self.layout_A` from `operation.A.layout`, establishing state in function `__init__`.
**CN:** 将 `self.layout_A` 赋值为 `operation.A.layout`，用于在函数 `__init__` 中建立状态。

##### Line 182 — Assign `self.layout_B`

```python
        self.layout_B = operation.B.layout
```
**EN:** Assigns `self.layout_B` from `operation.B.layout`, establishing state in function `__init__`.
**CN:** 将 `self.layout_B` 赋值为 `operation.B.layout`，用于在函数 `__init__` 中建立状态。

##### Line 183 — Assign `self.layout_C`

```python
        self.layout_C = operation.C.layout
```
**EN:** Assigns `self.layout_C` from `operation.C.layout`, establishing state in function `__init__`.
**CN:** 将 `self.layout_C` 赋值为 `operation.C.layout`，用于在函数 `__init__` 中建立状态。

##### Line 185 — Assign `self.element_A`

```python
        self.element_A = operation.A.element
```
**EN:** Assigns `self.element_A` from `operation.A.element`, establishing state in function `__init__`.
**CN:** 将 `self.element_A` 赋值为 `operation.A.element`，用于在函数 `__init__` 中建立状态。

##### Line 186 — Assign `self.element_B`

```python
        self.element_B = operation.B.element
```
**EN:** Assigns `self.element_B` from `operation.B.element`, establishing state in function `__init__`.
**CN:** 将 `self.element_B` 赋值为 `operation.B.element`，用于在函数 `__init__` 中建立状态。

##### Line 187 — Assign `self.element_C`

```python
        self.element_C = operation.C.element
```
**EN:** Assigns `self.element_C` from `operation.C.element`, establishing state in function `__init__`.
**CN:** 将 `self.element_C` 赋值为 `operation.C.element`，用于在函数 `__init__` 中建立状态。

##### Lines 189-190 — Conditional `operation.C.layout in [LayoutType.RowMajorInterleaved32, LayoutType.ColumnMajorInterleaved32]`

```python
        if operation.C.layout in [LayoutType.RowMajorInterleaved32, LayoutType.ColumnMajorInterleaved32]:
            raise Exception("Interleaved layout not currently supported")
```
**EN:** Checks `operation.C.layout in [LayoutType.RowMajorInterleaved32, LayoutType.ColumnMajorInterleaved32]` and selects the matching branch in function `__init__`.
**CN:** 检查 `operation.C.layout in [LayoutType.RowMajorInterleaved32, LayoutType.ColumnMajorInterleaved32]`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 192-195 — Conditional `hasattr(self.operation.epilogue_functor, 'visitor') and operation.arch not in [12, 20, 90, 100, 101, 103]`

```python
        if hasattr(self.operation.epilogue_functor, "visitor") and operation.arch not in [12, 20, 90, 100, 101, 103]:
            super().__init__(A, B, None, None, **kwargs)
        else:
            super().__init__(A, B, C, D, **kwargs)
```
**EN:** Checks `hasattr(self.operation.epilogue_functor, 'visitor') and operation.arch not in [12, 20, 90, 100, 101, 103]` and selects the matching branch in function `__init__`.
**CN:** 检查 `hasattr(self.operation.epilogue_functor, 'visitor') and operation.arch not in [12, 20, 90, 100, 101, 103]`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 197-201 — Conditional `operation.switched`

```python
        if operation.switched:
            self.problem_size = GemmCoord(problem_size.n, problem_size.m, problem_size.k)
            self.ptr_A, self.ptr_B = self.ptr_B, self.ptr_A
        else:
            self.problem_size = problem_size
```
**EN:** Checks `operation.switched` and selects the matching branch in function `__init__`.
**CN:** 检查 `operation.switched`，并在函数 `__init__` 中选择匹配的分支。

##### Line 202 — Comment or spacing block

```python
        # If the number of elements in C = problem_size.n, C is treated as the bias
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 203-205 — Conditional `hasattr(self, 'tensor_c_numel')`

```python
        if hasattr(self, "tensor_c_numel"):
            if self.tensor_c_numel == self.problem_size.n and self.problem_size.m != 1:
                self.bias = True
```
**EN:** Checks `hasattr(self, 'tensor_c_numel')` and selects the matching branch in function `__init__`.
**CN:** 检查 `hasattr(self, 'tensor_c_numel')`，并在函数 `__init__` 中选择匹配的分支。

##### Line 207 — Assign `self.lda`

```python
        self.lda = leading_dimension(self.layout_A, self.problem_size.mk)
```
**EN:** Assigns `self.lda` from `leading_dimension(self.layout_A, self.problem_size.mk)`, establishing state in function `__init__`.
**CN:** 将 `self.lda` 赋值为 `leading_dimension(self.layout_A, self.problem_size.mk)`，用于在函数 `__init__` 中建立状态。

##### Line 208 — Assign `self.ldb`

```python
        self.ldb = leading_dimension(self.layout_B, self.problem_size.kn)
```
**EN:** Assigns `self.ldb` from `leading_dimension(self.layout_B, self.problem_size.kn)`, establishing state in function `__init__`.
**CN:** 将 `self.ldb` 赋值为 `leading_dimension(self.layout_B, self.problem_size.kn)`，用于在函数 `__init__` 中建立状态。

##### Line 209 — Assign `self.ldc`

```python
        self.ldc = leading_dimension(self.layout_C, self.problem_size.mn)
```
**EN:** Assigns `self.ldc` from `leading_dimension(self.layout_C, self.problem_size.mn)`, establishing state in function `__init__`.
**CN:** 将 `self.ldc` 赋值为 `leading_dimension(self.layout_C, self.problem_size.mn)`，用于在函数 `__init__` 中建立状态。

##### Line 210 — Assign `self.ldd`

```python
        self.ldd = self.ldc
```
**EN:** Assigns `self.ldd` from `self.ldc`, establishing state in function `__init__`.
**CN:** 将 `self.ldd` 赋值为 `self.ldc`，用于在函数 `__init__` 中建立状态。

##### Lines 212-213 — Conditional `self.bias`

```python
        if self.bias:
            self.ldc = 0
```
**EN:** Checks `self.bias` and selects the matching branch in function `__init__`.
**CN:** 检查 `self.bias`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 215-222 — Conditional `'output_op' in kwargs.keys() and gemm_mode != GemmUniversalMode.GemmSplitKParallel`

```python
        if "output_op" in kwargs.keys() and gemm_mode != GemmUniversalMode.GemmSplitKParallel:
            self.output_op = kwargs["output_op"]
        else:
            if self.operation.epilogue_functor.element_epilogue in [DataType.s8, DataType.s32, DataType.u8, DataType.u32]:
                dtype = int
            else:
                dtype = float
            self.output_op = self.operation.epilogue_type(dtype(1.0), dtype(0.0))
```
**EN:** Checks `'output_op' in kwargs.keys() and gemm_mode != GemmUniversalMode.GemmSplitKParallel` and selects the matching branch in function `__init__`.
**CN:** 检查 `'output_op' in kwargs.keys() and gemm_mode != GemmUniversalMode.GemmSplitKParallel`，并在函数 `__init__` 中选择匹配的分支。

##### Line 224 — Assign `self.gemm_mode`

```python
        self.gemm_mode = gemm_mode
```
**EN:** Assigns `self.gemm_mode` from `gemm_mode`, establishing state in function `__init__`.
**CN:** 将 `self.gemm_mode` 赋值为 `gemm_mode`，用于在函数 `__init__` 中建立状态。

##### Lines 225-230 — Conditional `gemm_mode in [GemmUniversalMode.Gemm, GemmUniversalMode.GemmSplitKParallel]`

```python
        if gemm_mode in [GemmUniversalMode.Gemm, GemmUniversalMode.GemmSplitKParallel]:
            if "split_k_slices" in kwargs.keys():
                self.batch_count = kwargs["split_k_slices"]
            else:
                self.batch_count = 1
            self.split_k_slices = self.batch_count
```
**EN:** Checks `gemm_mode in [GemmUniversalMode.Gemm, GemmUniversalMode.GemmSplitKParallel]` and selects the matching branch in function `__init__`.
**CN:** 检查 `gemm_mode in [GemmUniversalMode.Gemm, GemmUniversalMode.GemmSplitKParallel]`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 232-236 — Conditional `gemm_mode in [GemmUniversalMode.Batched, GemmUniversalMode.Array]`

```python
        if gemm_mode in [GemmUniversalMode.Batched, GemmUniversalMode.Array]:
            if "batch" in kwargs.keys():
                self.batch_count = kwargs["batch"]
            else:
                self.batch_count = 1
```
**EN:** Checks `gemm_mode in [GemmUniversalMode.Batched, GemmUniversalMode.Array]` and selects the matching branch in function `__init__`.
**CN:** 检查 `gemm_mode in [GemmUniversalMode.Batched, GemmUniversalMode.Array]`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 238-247 — Conditional `'batch_strides' in kwargs`

```python
        if "batch_strides" in kwargs:
            self.batched_stride_A = kwargs["batch_strides"]["A"]
            self.batched_stride_B = kwargs["batch_strides"]["B"]
            self.batched_stride_C = kwargs["batch_strides"]["C"]
            self.batched_stride_D = kwargs["batch_strides"]["D"]
        else:
            self.batched_stride_A = self.problem_size.m * self.problem_size.k
            self.batched_stride_B = self.problem_size.n * self.problem_size.k
            self.batched_stride_C = self.problem_size.m * self.problem_size.n
            self.batched_stride_D = self.problem_size.m * self.problem_size.n
```
**EN:** Checks `'batch_strides' in kwargs` and selects the matching branch in function `__init__`.
**CN:** 检查 `'batch_strides' in kwargs`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 249-250 — Conditional `self.bias`

```python
        if self.bias:
            self.batched_stride_C = self.problem_size.n
```
**EN:** Checks `self.bias` and selects the matching branch in function `__init__`.
**CN:** 检查 `self.bias`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 252-281 — Conditional `gemm_mode == GemmUniversalMode.Array`

```python
        if gemm_mode == GemmUniversalMode.Array:
            self.ptr_A_array = []
            self.ptr_B_array = []
            self.ptr_C_array = []
            self.ptr_D_array = []

            ptr_A_addr = int(self.ptr_A)
            ptr_B_addr = int(self.ptr_B)
            ptr_C_addr = int(self.ptr_C)
            ptr_D_addr = int(self.ptr_D)

            stride_A = self.batched_stride_A * DataTypeSize[self.element_A] // 8
            stride_B = self.batched_stride_B * DataTypeSize[self.element_B] // 8
            stride_C = self.batched_stride_C * DataTypeSize[self.element_C] // 8
            stride_D = self.batched_stride_D * DataTypeSize[self.element_C] // 8
            for _ in range(self.batch_count):
                self.ptr_A_array.append(ptr_A_addr)
                self.ptr_B_array.append(ptr_B_addr)
                self.ptr_C_array.append(ptr_C_addr)
                self.ptr_D_array.append(ptr_D_addr)

                ptr_A_addr += stride_A
                ptr_B_addr += stride_B
                ptr_C_addr += stride_C
                ptr_D_addr += stride_D

            self.ptr_A_array_buffer = todevice(self.ptr_A_array, dtype=np.int64)
            self.ptr_B_array_buffer = todevice(self.ptr_B_array, dtype=np.int64)
            self.ptr_C_array_buffer = todevice(self.ptr_C_array, dtype=np.int64)
            self.ptr_D_array_buffer = todevice(self.ptr_D_array, dtype=np.int64)
```
**EN:** Checks `gemm_mode == GemmUniversalMode.Array` and selects the matching branch in function `__init__`.
**CN:** 检查 `gemm_mode == GemmUniversalMode.Array`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 283-284 — Conditional `isinstance(self.operation, GemmOperationUniversal)`

```python
        if isinstance(self.operation, GemmOperationUniversal):
            self.initialize()
```
**EN:** Checks `isinstance(self.operation, GemmOperationUniversal)` and selects the matching branch in function `__init__`.
**CN:** 检查 `isinstance(self.operation, GemmOperationUniversal)`，并在函数 `__init__` 中选择匹配的分支。

#### Line 286 — Function `get_arguments`

```python
    def get_arguments(self):
```
**EN:** Defines function `get_arguments` with parameters `self`.
**CN:** 定义函数 `get_arguments`，参数为 `self`。

##### Line 287 — Assign `problem_size_`

```python
        problem_size_ = self.problem_size.ctype
```
**EN:** Assigns `problem_size_` from `self.problem_size.ctype`, establishing state in function `get_arguments`.
**CN:** 将 `problem_size_` 赋值为 `self.problem_size.ctype`，用于在函数 `get_arguments` 中建立状态。

##### Lines 288-291 — Assign `grid_tiled_shape_`

```python
        grid_tiled_shape_ = GemmCoord(
            self.grid_tiled_shape.x,
            self.grid_tiled_shape.y,
            self.grid_tiled_shape.z ).ctype
```
**EN:** Assigns `grid_tiled_shape_` from `GemmCoord(self.grid_tiled_shape.x, self.grid_tiled_shape.y, self.grid_tiled_shape.z).ctype`, establishing state in function `get_arguments`.
**CN:** 将 `grid_tiled_shape_` 赋值为 `GemmCoord(self.grid_tiled_shape.x, self.grid_tiled_shape.y, self.grid_tiled_shape.z).ctype`，用于在函数 `get_arguments` 中建立状态。

##### Lines 293-327 — Conditional `self.gemm_mode == GemmUniversalMode.Array`

```python
        if self.gemm_mode == GemmUniversalMode.Array:
            arguments = self.operation.argument_type(
                # Arguments from UniversalArgumentsBase
                self.gemm_mode,
                problem_size_,
                self.batch_count,
                0,
                # Remaining arguments
                self.output_op,
                int(self.ptr_A_array_buffer.ptr),
                int(self.ptr_B_array_buffer.ptr),
                int(self.ptr_C_array_buffer.ptr),
                int(self.ptr_D_array_buffer.ptr),
                0, 0, 0,
                self.lda, self.ldb, self.ldc, self.ldd,
                self.lda, self.ldb, self.ldc, self.ldd,
                0, 0, 0
            )
        else:
            arguments = self.operation.argument_type(
                # Arguments from UniversalArgumentsBase
                self.gemm_mode, problem_size_, self.batch_count, self.batched_stride_D,
                # Remaining arguments
                self.output_op,
                int(self.ptr_A),
                int(self.ptr_B),
                int(self.ptr_C),
                int(self.ptr_D),
                self.batched_stride_A,
                self.batched_stride_B,
                self.batched_stride_C,
                self.lda, self.ldb, self.ldc, self.ldd,
                self.lda, self.ldb, self.ldc, self.ldd,
                0, 0, 0
            )
```
**EN:** Checks `self.gemm_mode == GemmUniversalMode.Array` and selects the matching branch in function `get_arguments`.
**CN:** 检查 `self.gemm_mode == GemmUniversalMode.Array`，并在函数 `get_arguments` 中选择匹配的分支。

##### Line 329 — Assign `self.arguments`

```python
        self.arguments = arguments, grid_tiled_shape_, self.gemm_k_size
```
**EN:** Assigns `self.arguments` from `(arguments, grid_tiled_shape_, self.gemm_k_size)`, establishing state in function `get_arguments`.
**CN:** 将 `self.arguments` 赋值为 `(arguments, grid_tiled_shape_, self.gemm_k_size)`，用于在函数 `get_arguments` 中建立状态。

#### Line 331 — Function `initialize`

```python
    def initialize(self):
```
**EN:** Defines function `initialize` with parameters `self`.
**CN:** 定义函数 `initialize`，参数为 `self`。

##### Line 332 — Assign `launch_config`

```python
        launch_config = self.operation.rt_module.plan(self)
```
**EN:** Assigns `launch_config` from `self.operation.rt_module.plan(self)`, establishing state in function `initialize`.
**CN:** 将 `launch_config` 赋值为 `self.operation.rt_module.plan(self)`，用于在函数 `initialize` 中建立状态。

##### Line 334 — Comment or spacing block

```python
        # Get the host and device workspace
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 335 — Assign `device_workspace_size`

```python
        device_workspace_size = self.operation.rt_module.get_device_workspace_size(self)
```
**EN:** Assigns `device_workspace_size` from `self.operation.rt_module.get_device_workspace_size(self)`, establishing state in function `initialize`.
**CN:** 将 `device_workspace_size` 赋值为 `self.operation.rt_module.get_device_workspace_size(self)`，用于在函数 `initialize` 中建立状态。

##### Lines 337-343 — Conditional `device_workspace_size > 0`

```python
        if device_workspace_size > 0:
            self.workspace_buffer = device_mem_alloc(device_workspace_size)
            workspace_ptr = self.workspace_buffer.ptr
            err, = cuda.cuMemsetD32(
                workspace_ptr, 0, device_workspace_size // 4)
        else:
            workspace_ptr = None
```
**EN:** Checks `device_workspace_size > 0` and selects the matching branch in function `initialize`.
**CN:** 检查 `device_workspace_size > 0`，并在函数 `initialize` 中选择匹配的分支。

##### Line 345 — Assign `device_workspace`

```python
        device_workspace = 0
```
**EN:** Assigns `device_workspace` from `0`, establishing state in function `initialize`.
**CN:** 将 `device_workspace` 赋值为 `0`，用于在函数 `initialize` 中建立状态。

##### Lines 346-350 — Conditional `workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel`

```python
        if workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel:
            # In GEMM splik-K parallel, the D pointer is redirected to the workspace
            self.ptr_D = cuda.CUdeviceptr(workspace_ptr)
        elif workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.Gemm:
            device_workspace = workspace_ptr
```
**EN:** Checks `workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel` and selects the matching branch in function `initialize`.
**CN:** 检查 `workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel`，并在函数 `initialize` 中选择匹配的分支。

##### Line 352 — Call `self.get_arguments`

```python
        self.get_arguments()
```
**EN:** Calls `self.get_arguments` for side effects or initialization work in function `initialize`.
**CN:** 调用 `self.get_arguments` 执行副作用或初始化逻辑；该语句位于在函数 `initialize` 中。

##### Line 354 — Assign `arguments, grid_tiled_shape, gemm_k_size`

```python
        arguments, grid_tiled_shape, gemm_k_size = self.arguments
```
**EN:** Assigns `arguments, grid_tiled_shape, gemm_k_size` from `self.arguments`, establishing state in function `initialize`.
**CN:** 将 `arguments, grid_tiled_shape, gemm_k_size` 赋值为 `self.arguments`，用于在函数 `initialize` 中建立状态。

##### Lines 355-356 — Assign `res_arg`

```python
        res_arg = self.operation.rt_module.get_args(
            ctypes.byref(arguments), ctypes.c_void_p(int(device_workspace)))
```
**EN:** Assigns `res_arg` from `self.operation.rt_module.get_args(ctypes.byref(arguments), ctypes.c_void_p(int(device_workspace)))`, establishing state in function `initialize`.
**CN:** 将 `res_arg` 赋值为 `self.operation.rt_module.get_args(ctypes.byref(arguments), ctypes.c_void_p(int(device_workspace)))`，用于在函数 `initialize` 中建立状态。

##### Line 357 — Assign `host_workspace`

```python
        host_workspace = bytearray(res_arg.contents)
```
**EN:** Assigns `host_workspace` from `bytearray(res_arg.contents)`, establishing state in function `initialize`.
**CN:** 将 `host_workspace` 赋值为 `bytearray(res_arg.contents)`，用于在函数 `initialize` 中建立状态。

##### Line 359 — Assign `device_workspace`

```python
        device_workspace = None
```
**EN:** Assigns `device_workspace` from `None`, establishing state in function `initialize`.
**CN:** 将 `device_workspace` 赋值为 `None`，用于在函数 `initialize` 中建立状态。

##### Line 361 — Assign `self.host_workspace`

```python
        self.host_workspace = host_workspace
```
**EN:** Assigns `self.host_workspace` from `host_workspace`, establishing state in function `initialize`.
**CN:** 将 `self.host_workspace` 赋值为 `host_workspace`，用于在函数 `initialize` 中建立状态。

##### Line 362 — Assign `self.device_workspace`

```python
        self.device_workspace = device_workspace
```
**EN:** Assigns `self.device_workspace` from `device_workspace`, establishing state in function `initialize`.
**CN:** 将 `self.device_workspace` 赋值为 `device_workspace`，用于在函数 `initialize` 中建立状态。

##### Line 363 — Assign `self.launch_config`

```python
        self.launch_config = launch_config
```
**EN:** Assigns `self.launch_config` from `launch_config`, establishing state in function `initialize`.
**CN:** 将 `self.launch_config` 赋值为 `launch_config`，用于在函数 `initialize` 中建立状态。

#### Line 365 — Function `sync`

```python
    def sync(self, stream_sync=True):
```
**EN:** Defines function `sync` with parameters `self, stream_sync`.
**CN:** 定义函数 `sync`，参数为 `self, stream_sync`。

##### Line 366 — Call `super().sync`

```python
        super().sync(stream_sync)
```
**EN:** Calls `super().sync` for side effects or initialization work in function `sync`.
**CN:** 调用 `super().sync` 执行副作用或初始化逻辑；该语句位于在函数 `sync` 中。

##### Lines 367-368 — Conditional `hasattr(self.output_op, 'sync')`

```python
        if hasattr(self.output_op, "sync"):
            self.output_op.sync()
```
**EN:** Checks `hasattr(self.output_op, 'sync')` and selects the matching branch in function `sync`.
**CN:** 检查 `hasattr(self.output_op, 'sync')`，并在函数 `sync` 中选择匹配的分支。

### Lines 371-400 — Class `GemmArguments2xStreamK`

```python
class GemmArguments2xStreamK(GemmArguments2x):
    """
    Argument wrapper for stream-K GEMMs in CUTLASS 2. It encodes problem information and
    user-provide tensors into the kernel's argument

    :param operation: the GEMM operation to take the argument
    :type operation: :class:`cutlass_cppgen.backend.GemmOperationUniversal` |
     :class:`cutlass_cppgen.backend.GemmOperationGrouped`

    :param problem_size: GEMM problem size gemm(M, N, K)
    :type operation: :class:`cutlass_cppgen.shape.GemmCoord`

    :param A: tensor A
    :type A: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param B: tensor B
    :type B: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param C: tensor C
    :type C: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param D: tensor D
    :type D: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param gemm_mode: GEMM mode
    :type gemm_mode: :class:`cutlass_library.GemmUniversalMode`

    :param output_op: output operator, optional
    :type output_op: :class:`cutlass_cppgen.backend.LinearCombinationFunctorArguments`
    """
```
**EN:** Declares class `GemmArguments2xStreamK` deriving from `GemmArguments2x`. Purpose: Argument wrapper for stream-K GEMMs in CUTLASS 2.
**CN:** 声明类 `GemmArguments2xStreamK`，其基类为 `GemmArguments2x`。 其用途：Argument wrapper for stream-K GEMMs in CUTLASS 2.

#### Line 402 — Function `__init__`

```python
    def __init__(self, operation, problem_size, A, B, C, D, gemm_mode=GemmUniversalMode.Gemm, **kwargs):
```
**EN:** Defines function `__init__` with parameters `self, operation, problem_size, A, B, C, D, gemm_mode, **kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, operation, problem_size, A, B, C, D, gemm_mode, **kwargs`。

##### Lines 403-404 — Conditional `gemm_mode not in [GemmUniversalMode.Gemm, GemmUniversalMode.Batched]`

```python
        if gemm_mode not in [GemmUniversalMode.Gemm, GemmUniversalMode.Batched]:
            raise Exception(f"Unsupported GEMM mode {gemm_mode}.")
```
**EN:** Checks `gemm_mode not in [GemmUniversalMode.Gemm, GemmUniversalMode.Batched]` and selects the matching branch in function `__init__`.
**CN:** 检查 `gemm_mode not in [GemmUniversalMode.Gemm, GemmUniversalMode.Batched]`，并在函数 `__init__` 中选择匹配的分支。

##### Line 406 — Call `super().__init__`

```python
        super().__init__(operation, problem_size, A, B, C, D, gemm_mode, **kwargs)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

#### Line 408 — Function `get_arguments`

```python
    def get_arguments(self):
```
**EN:** Defines function `get_arguments` with parameters `self`.
**CN:** 定义函数 `get_arguments`，参数为 `self`。

##### Line 409 — Assign `batch_stride_A`

```python
        batch_stride_A = self.problem_size.m * self.problem_size.k
```
**EN:** Assigns `batch_stride_A` from `self.problem_size.m * self.problem_size.k`, establishing state in function `get_arguments`.
**CN:** 将 `batch_stride_A` 赋值为 `self.problem_size.m * self.problem_size.k`，用于在函数 `get_arguments` 中建立状态。

##### Line 410 — Assign `batch_stride_B`

```python
        batch_stride_B = self.problem_size.k * self.problem_size.n
```
**EN:** Assigns `batch_stride_B` from `self.problem_size.k * self.problem_size.n`, establishing state in function `get_arguments`.
**CN:** 将 `batch_stride_B` 赋值为 `self.problem_size.k * self.problem_size.n`，用于在函数 `get_arguments` 中建立状态。

##### Line 411 — Assign `batch_stride_C`

```python
        batch_stride_C = self.problem_size.m * self.problem_size.n
```
**EN:** Assigns `batch_stride_C` from `self.problem_size.m * self.problem_size.n`, establishing state in function `get_arguments`.
**CN:** 将 `batch_stride_C` 赋值为 `self.problem_size.m * self.problem_size.n`，用于在函数 `get_arguments` 中建立状态。

##### Line 412 — Assign `batch_stride_D`

```python
        batch_stride_D = self.problem_size.m * self.problem_size.n
```
**EN:** Assigns `batch_stride_D` from `self.problem_size.m * self.problem_size.n`, establishing state in function `get_arguments`.
**CN:** 将 `batch_stride_D` 赋值为 `self.problem_size.m * self.problem_size.n`，用于在函数 `get_arguments` 中建立状态。

##### Lines 414-430 — Assign `arguments`

```python
        arguments = self.operation.argument_type(
            self.gemm_mode,
            GemmCoord_(self.problem_size.m, self.problem_size.n, self.problem_size.k),
            self.batch_count,
            self.output_op,
            int(self.ptr_A),
            int(self.ptr_B),
            int(self.ptr_C),
            int(self.ptr_D),
            batch_stride_A,
            batch_stride_B,
            batch_stride_C,
            batch_stride_D,
            self.lda, self.ldb, self.ldc, self.ldd,  # strides
            self.lda, self.ldb, self.ldc, self.ldd,
            -1,  # avail_sms
        )
```
**EN:** Assigns `arguments` from `self.operation.argument_type(self.gemm_mode, GemmCoord_(self.problem_size.m, self.problem_size.n, self.problem_size.k), self.batch_count, self.output_op, int...`, establishing state in function `get_arguments`.
**CN:** 将 `arguments` 赋值为 `self.operation.argument_type(self.gemm_mode, GemmCoord_(self.problem_size.m, self.problem_size.n, self.problem_size.k), self.batch_count, self.output_op, int...`，用于在函数 `get_arguments` 中建立状态。

##### Line 431 — Return

```python
        return arguments
```
**EN:** Returns `arguments` to the caller.
**CN:** 向调用方返回 `arguments`。

#### Lines 433-434 — Function `initialize`

```python
    def initialize(self):
        # Get the host and device workspace
```
**EN:** Defines function `initialize` with parameters `self`.
**CN:** 定义函数 `initialize`，参数为 `self`。

##### Lines 435-439 — Assign `device_workspace_size`

```python
        device_workspace_size = self.operation.rt_module.get_device_workspace_size(
            self,
            device_sm_count(),
            self.operation.rt_module.occupancy
        )
```
**EN:** Assigns `device_workspace_size` from `self.operation.rt_module.get_device_workspace_size(self, device_sm_count(), self.operation.rt_module.occupancy)`, establishing state in function `initialize`.
**CN:** 将 `device_workspace_size` 赋值为 `self.operation.rt_module.get_device_workspace_size(self, device_sm_count(), self.operation.rt_module.occupancy)`，用于在函数 `initialize` 中建立状态。

##### Lines 441-447 — Conditional `device_workspace_size > 0`

```python
        if device_workspace_size > 0:
            self.workspace_buffer = device_mem_alloc(device_workspace_size)
            workspace_ptr = self.workspace_buffer.ptr
            err, = cuda.cuMemsetD32(
                workspace_ptr, 0, device_workspace_size // 4)
        else:
            workspace_ptr = None
```
**EN:** Checks `device_workspace_size > 0` and selects the matching branch in function `initialize`.
**CN:** 检查 `device_workspace_size > 0`，并在函数 `initialize` 中选择匹配的分支。

##### Line 449 — Assign `device_workspace`

```python
        device_workspace = 0
```
**EN:** Assigns `device_workspace` from `0`, establishing state in function `initialize`.
**CN:** 将 `device_workspace` 赋值为 `0`，用于在函数 `initialize` 中建立状态。

##### Lines 450-454 — Conditional `workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel`

```python
        if workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel:
            # In GEMM splik-K parallel, the D pointer is redirected to the workspace
            self.ptr_D = cuda.CUdeviceptr(workspace_ptr)
        elif workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.Gemm:
            device_workspace = workspace_ptr
```
**EN:** Checks `workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel` and selects the matching branch in function `initialize`.
**CN:** 检查 `workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel`，并在函数 `initialize` 中选择匹配的分支。

##### Line 456 — Assign `arguments`

```python
        arguments = self.get_arguments()
```
**EN:** Assigns `arguments` from `self.get_arguments()`, establishing state in function `initialize`.
**CN:** 将 `arguments` 赋值为 `self.get_arguments()`，用于在函数 `initialize` 中建立状态。

##### Lines 458-463 — Assign `res_arg`

```python
        res_arg = self.operation.rt_module.get_args(
            ctypes.byref(arguments),
            ctypes.c_void_p(int(device_workspace)),
            device_sm_count(),
            self.operation.rt_module.occupancy
        )
```
**EN:** Assigns `res_arg` from `self.operation.rt_module.get_args(ctypes.byref(arguments), ctypes.c_void_p(int(device_workspace)), device_sm_count(), self.operation.rt_module.occupancy)`, establishing state in function `initialize`.
**CN:** 将 `res_arg` 赋值为 `self.operation.rt_module.get_args(ctypes.byref(arguments), ctypes.c_void_p(int(device_workspace)), device_sm_count(), self.operation.rt_module.occupancy)`，用于在函数 `initialize` 中建立状态。

##### Line 464 — Assign `host_workspace`

```python
        host_workspace = bytearray(res_arg.contents)
```
**EN:** Assigns `host_workspace` from `bytearray(res_arg.contents)`, establishing state in function `initialize`.
**CN:** 将 `host_workspace` 赋值为 `bytearray(res_arg.contents)`，用于在函数 `initialize` 中建立状态。

##### Lines 466-470 — Assign `grid`

```python
        grid = self.operation.rt_module.get_grid_shape(
            ctypes.byref(arguments),
            device_sm_count(),
            self.operation.rt_module.occupancy
        )
```
**EN:** Assigns `grid` from `self.operation.rt_module.get_grid_shape(ctypes.byref(arguments), device_sm_count(), self.operation.rt_module.occupancy)`, establishing state in function `initialize`.
**CN:** 将 `grid` 赋值为 `self.operation.rt_module.get_grid_shape(ctypes.byref(arguments), device_sm_count(), self.operation.rt_module.occupancy)`，用于在函数 `initialize` 中建立状态。

##### Line 472 — Assign `device_workspace`

```python
        device_workspace = None
```
**EN:** Assigns `device_workspace` from `None`, establishing state in function `initialize`.
**CN:** 将 `device_workspace` 赋值为 `None`，用于在函数 `initialize` 中建立状态。

##### Line 474 — Assign `self.host_workspace`

```python
        self.host_workspace = host_workspace
```
**EN:** Assigns `self.host_workspace` from `host_workspace`, establishing state in function `initialize`.
**CN:** 将 `self.host_workspace` 赋值为 `host_workspace`，用于在函数 `initialize` 中建立状态。

##### Line 475 — Assign `self.device_workspace`

```python
        self.device_workspace = device_workspace
```
**EN:** Assigns `self.device_workspace` from `device_workspace`, establishing state in function `initialize`.
**CN:** 将 `self.device_workspace` 赋值为 `device_workspace`，用于在函数 `initialize` 中建立状态。

##### Lines 476-480 — Assign `self.launch_config`

```python
        self.launch_config = LaunchConfiguration(
            [grid.m, grid.n, grid.k],
            [self.operation.rt_module.threads, 1, 1],
            self.operation.rt_module.shared_memory_capacity
        )
```
**EN:** Assigns `self.launch_config` from `LaunchConfiguration([grid.m, grid.n, grid.k], [self.operation.rt_module.threads, 1, 1], self.operation.rt_module.shared_memory_capacity)`, establishing state in function `initialize`.
**CN:** 将 `self.launch_config` 赋值为 `LaunchConfiguration([grid.m, grid.n, grid.k], [self.operation.rt_module.threads, 1, 1], self.operation.rt_module.shared_memory_capacity)`，用于在函数 `initialize` 中建立状态。

### Lines 483-512 — Class `GemmArguments3x`

```python
class GemmArguments3x(GemmArguments2x):
    """
    Argument wrapper for GEMM in CUTLASS 3. It encodes problem information and
    user-provide tensors into the kernel's argument

    :param operation: the GEMM operation to take the argument
    :type operation: :class:`cutlass_cppgen.backend.GemmOperationUniversal` |
     :class:`cutlass_cppgen.backend.GemmOperationGrouped`

    :param problem_size: GEMM problem size gemm(M, N, K)
    :type operation: :class:`cutlass_cppgen.shape.GemmCoord`

    :param A: tensor A
    :type A: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param B: tensor B
    :type B: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param C: tensor C
    :type C: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param D: tensor D
    :type D: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param gemm_mode: GEMM mode
    :type gemm_mode: GemmUniversalMode

    :param output_op: output operator, optional
    :type output_op: :class:`cutlass_cppgen.backend.LinearCombinationFunctorArguments`
    """
```
**EN:** Declares class `GemmArguments3x` deriving from `GemmArguments2x`. Purpose: Argument wrapper for GEMM in CUTLASS 3.
**CN:** 声明类 `GemmArguments3x`，其基类为 `GemmArguments2x`。 其用途：Argument wrapper for GEMM in CUTLASS 3.

#### Line 514 — Function `__init__`

```python
    def __init__(self, operation, problem_size, A, B, C, D, gemm_mode=GemmUniversalMode.Gemm, **kwargs):
```
**EN:** Defines function `__init__` with parameters `self, operation, problem_size, A, B, C, D, gemm_mode, **kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, operation, problem_size, A, B, C, D, gemm_mode, **kwargs`。

##### Lines 515-516 — Conditional `gemm_mode not in [GemmUniversalMode.Gemm, GemmUniversalMode.Batched]`

```python
        if gemm_mode not in [GemmUniversalMode.Gemm, GemmUniversalMode.Batched]:
            raise Exception(f"Unsupported GEMM mode {gemm_mode}.")
```
**EN:** Checks `gemm_mode not in [GemmUniversalMode.Gemm, GemmUniversalMode.Batched]` and selects the matching branch in function `__init__`.
**CN:** 检查 `gemm_mode not in [GemmUniversalMode.Gemm, GemmUniversalMode.Batched]`，并在函数 `__init__` 中选择匹配的分支。

##### Line 518 — Call `super().__init__`

```python
        super().__init__(operation, problem_size, A, B, C, D, gemm_mode, **kwargs)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

#### Line 520 — Function `get_arguments`

```python
    def get_arguments(self):
```
**EN:** Defines function `get_arguments` with parameters `self`.
**CN:** 定义函数 `get_arguments`，参数为 `self`。

##### Line 521 — Assign `use_sycl`

```python
        use_sycl = isinstance(self.stream, dpctl.SyclQueue)
```
**EN:** Assigns `use_sycl` from `isinstance(self.stream, dpctl.SyclQueue)`, establishing state in function `get_arguments`.
**CN:** 将 `use_sycl` 赋值为 `isinstance(self.stream, dpctl.SyclQueue)`，用于在函数 `get_arguments` 中建立状态。

##### Lines 522-529 — Assign `mainloop_args`

```python
        mainloop_args = get_mainloop_arguments_3x(
            self.operation.tile_description.kernel_schedule,
            self.operation.A.element,
            self.operation.B.element,
            self.operation.A.alignment,
            self.operation.B.alignment,
            use_sycl
        )
```
**EN:** Assigns `mainloop_args` from `get_mainloop_arguments_3x(self.operation.tile_description.kernel_schedule, self.operation.A.element, self.operation.B.element, self.operation.A.alignment, se...`, establishing state in function `get_arguments`.
**CN:** 将 `mainloop_args` 赋值为 `get_mainloop_arguments_3x(self.operation.tile_description.kernel_schedule, self.operation.A.element, self.operation.B.element, self.operation.A.alignment, se...`，用于在函数 `get_arguments` 中建立状态。

##### Line 530 — Assign `scheduler_args`

```python
        scheduler_args = get_tile_scheduler_arguments_3x(self.operation.tile_description.tile_scheduler)
```
**EN:** Assigns `scheduler_args` from `get_tile_scheduler_arguments_3x(self.operation.tile_description.tile_scheduler)`, establishing state in function `get_arguments`.
**CN:** 将 `scheduler_args` 赋值为 `get_tile_scheduler_arguments_3x(self.operation.tile_description.tile_scheduler)`，用于在函数 `get_arguments` 中建立状态。

##### Line 531 — Assign `uses_default_epilogue`

```python
        uses_default_epilogue = self.operation.rt_module.uses_default_epilogue()
```
**EN:** Assigns `uses_default_epilogue` from `self.operation.rt_module.uses_default_epilogue()`, establishing state in function `get_arguments`.
**CN:** 将 `uses_default_epilogue` 赋值为 `self.operation.rt_module.uses_default_epilogue()`，用于在函数 `get_arguments` 中建立状态。

##### Lines 532-533 — Assign `argument_type, epilogue_args, epilogue_type, hw_info`

```python
        argument_type, epilogue_args, epilogue_type, hw_info = get_gemm_arguments_3x(
            mainloop_args, self.operation.epilogue_functor, scheduler_args, uses_default_epilogue)
```
**EN:** Assigns `argument_type, epilogue_args, epilogue_type, hw_info` from `get_gemm_arguments_3x(mainloop_args, self.operation.epilogue_functor, scheduler_args, uses_default_epilogue)`, establishing state in function `get_arguments`.
**CN:** 将 `argument_type, epilogue_args, epilogue_type, hw_info` 赋值为 `get_gemm_arguments_3x(mainloop_args, self.operation.epilogue_functor, scheduler_args, uses_default_epilogue)`，用于在函数 `get_arguments` 中建立状态。

##### Line 535 — Assign `problem_size_`

```python
        problem_size_ = GemmCoordBatched_(self.problem_size, self.batch_count)
```
**EN:** Assigns `problem_size_` from `GemmCoordBatched_(self.problem_size, self.batch_count)`, establishing state in function `get_arguments`.
**CN:** 将 `problem_size_` 赋值为 `GemmCoordBatched_(self.problem_size, self.batch_count)`，用于在函数 `get_arguments` 中建立状态。

##### Lines 537-546 — Conditional `self.batch_count > 1`

```python
        if self.batch_count > 1:
            bsA = self.batched_stride_A
            bsB = self.batched_stride_B
            bsC = self.batched_stride_C
            bsD = self.batched_stride_D
        else:
            bsA = 0
            bsB = 0
            bsC = 0
            bsD = 0
```
**EN:** Checks `self.batch_count > 1` and selects the matching branch in function `get_arguments`.
**CN:** 检查 `self.batch_count > 1`，并在函数 `get_arguments` 中选择匹配的分支。

##### Line 547 — Assign `stride_A`

```python
        stride_A = StrideBatched_(self.lda, bsA)
```
**EN:** Assigns `stride_A` from `StrideBatched_(self.lda, bsA)`, establishing state in function `get_arguments`.
**CN:** 将 `stride_A` 赋值为 `StrideBatched_(self.lda, bsA)`，用于在函数 `get_arguments` 中建立状态。

##### Line 548 — Assign `stride_B`

```python
        stride_B = StrideBatched_(self.ldb, bsB)
```
**EN:** Assigns `stride_B` from `StrideBatched_(self.ldb, bsB)`, establishing state in function `get_arguments`.
**CN:** 将 `stride_B` 赋值为 `StrideBatched_(self.ldb, bsB)`，用于在函数 `get_arguments` 中建立状态。

##### Line 549 — Assign `stride_C`

```python
        stride_C = StrideBatched_(self.ldc, bsC)
```
**EN:** Assigns `stride_C` from `StrideBatched_(self.ldc, bsC)`, establishing state in function `get_arguments`.
**CN:** 将 `stride_C` 赋值为 `StrideBatched_(self.ldc, bsC)`，用于在函数 `get_arguments` 中建立状态。

##### Line 550 — Assign `stride_D`

```python
        stride_D = StrideBatched_(self.ldd, bsD)
```
**EN:** Assigns `stride_D` from `StrideBatched_(self.ldd, bsD)`, establishing state in function `get_arguments`.
**CN:** 将 `stride_D` 赋值为 `StrideBatched_(self.ldd, bsD)`，用于在函数 `get_arguments` 中建立状态。

##### Line 552 — Comment or spacing block

```python
        # Superset of potential mainloop arguments
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 553-559 — Assign `generic_args`

```python
        generic_args = GenericMainloopArguments3x_(
            int(self.ptr_A),
            stride_A,
            int(self.ptr_B),
            stride_B,
            4 # mma_promotion_interval
        )
```
**EN:** Assigns `generic_args` from `GenericMainloopArguments3x_(int(self.ptr_A), stride_A, int(self.ptr_B), stride_B, 4)`, establishing state in function `get_arguments`.
**CN:** 将 `generic_args` 赋值为 `GenericMainloopArguments3x_(int(self.ptr_A), stride_A, int(self.ptr_B), stride_B, 4)`，用于在函数 `get_arguments` 中建立状态。

##### Line 561 — Comment or spacing block

```python
        # Set of mainloop arguments needed for this kernel
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 562 — Assign `mainloop`

```python
        mainloop = mainloop_args.from_generic_mainloop_args(generic_args)
```
**EN:** Assigns `mainloop` from `mainloop_args.from_generic_mainloop_args(generic_args)`, establishing state in function `get_arguments`.
**CN:** 将 `mainloop` 赋值为 `mainloop_args.from_generic_mainloop_args(generic_args)`，用于在函数 `get_arguments` 中建立状态。

##### Lines 564-565 — Conditional `not uses_default_epilogue and hasattr(self.output_op, 'to_evt_params')`

```python
        if not uses_default_epilogue and hasattr(self.output_op, "to_evt_params"):
            self.output_op = self.output_op.to_evt_params()
```
**EN:** Checks `not uses_default_epilogue and hasattr(self.output_op, 'to_evt_params')` and selects the matching branch in function `get_arguments`.
**CN:** 检查 `not uses_default_epilogue and hasattr(self.output_op, 'to_evt_params')`，并在函数 `get_arguments` 中选择匹配的分支。

##### Lines 567-573 — Assign `epilogue`

```python
        epilogue = epilogue_args(
            self.output_op,
            int(self.ptr_C),
            stride_C,
            int(self.ptr_D),
            stride_D,
        )
```
**EN:** Assigns `epilogue` from `epilogue_args(self.output_op, int(self.ptr_C), stride_C, int(self.ptr_D), stride_D)`, establishing state in function `get_arguments`.
**CN:** 将 `epilogue` 赋值为 `epilogue_args(self.output_op, int(self.ptr_C), stride_C, int(self.ptr_D), stride_D)`，用于在函数 `get_arguments` 中建立状态。

##### Line 575 — Comment or spacing block

```python
        # Set hardware info
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 576-580 — Assign `hw_info_`

```python
        hw_info_ = hw_info(
            0, device_sm_count(), 0,
            dim3_(0,0,0),
            dim3_(0,0,0),
        )
```
**EN:** Assigns `hw_info_` from `hw_info(0, device_sm_count(), 0, dim3_(0, 0, 0), dim3_(0, 0, 0))`, establishing state in function `get_arguments`.
**CN:** 将 `hw_info_` 赋值为 `hw_info(0, device_sm_count(), 0, dim3_(0, 0, 0), dim3_(0, 0, 0))`，用于在函数 `get_arguments` 中建立状态。

##### Lines 582-589 — Assign `self.arguments`

```python
        self.arguments = argument_type(
            int(self.gemm_mode),
            problem_size_,
            mainloop,
            epilogue,
            hw_info_,
            scheduler_args
        )
```
**EN:** Assigns `self.arguments` from `argument_type(int(self.gemm_mode), problem_size_, mainloop, epilogue, hw_info_, scheduler_args)`, establishing state in function `get_arguments`.
**CN:** 将 `self.arguments` 赋值为 `argument_type(int(self.gemm_mode), problem_size_, mainloop, epilogue, hw_info_, scheduler_args)`，用于在函数 `get_arguments` 中建立状态。

##### Line 590 — Return

```python
        return self.arguments
```
**EN:** Returns `self.arguments` to the caller.
**CN:** 向调用方返回 `self.arguments`。

#### Lines 592-593 — Function `initialize`

```python
    def initialize(self):
        # Get the host and evice workspace
```
**EN:** Defines function `initialize` with parameters `self`.
**CN:** 定义函数 `initialize`，参数为 `self`。

##### Line 594 — Assign `device_workspace_size`

```python
        device_workspace_size = self.operation.rt_module.get_device_workspace_size(self)
```
**EN:** Assigns `device_workspace_size` from `self.operation.rt_module.get_device_workspace_size(self)`, establishing state in function `initialize`.
**CN:** 将 `device_workspace_size` 赋值为 `self.operation.rt_module.get_device_workspace_size(self)`，用于在函数 `initialize` 中建立状态。

##### Lines 596-602 — Conditional `device_workspace_size > 0`

```python
        if device_workspace_size > 0:
            self.workspace_buffer = device_mem_alloc(device_workspace_size)
            workspace_ptr = self.workspace_buffer.ptr
            err, = cuda.cuMemsetD32(
                workspace_ptr, 0, device_workspace_size // 4)
        else:
            workspace_ptr = None
```
**EN:** Checks `device_workspace_size > 0` and selects the matching branch in function `initialize`.
**CN:** 检查 `device_workspace_size > 0`，并在函数 `initialize` 中选择匹配的分支。

##### Line 604 — Assign `device_workspace`

```python
        device_workspace = 0
```
**EN:** Assigns `device_workspace` from `0`, establishing state in function `initialize`.
**CN:** 将 `device_workspace` 赋值为 `0`，用于在函数 `initialize` 中建立状态。

##### Lines 605-609 — Conditional `workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel`

```python
        if workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel:
            # In GEMM splik-K parallel, the D pointer is redirected to the workspace
            self.ptr_D = cuda.CUdeviceptr(workspace_ptr)
        elif workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.Gemm:
            device_workspace = workspace_ptr
```
**EN:** Checks `workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel` and selects the matching branch in function `initialize`.
**CN:** 检查 `workspace_ptr is not None and self.gemm_mode == GemmUniversalMode.GemmSplitKParallel`，并在函数 `initialize` 中选择匹配的分支。

##### Line 611 — Call `self.get_arguments`

```python
        self.get_arguments()
```
**EN:** Calls `self.get_arguments` for side effects or initialization work in function `initialize`.
**CN:** 调用 `self.get_arguments` 执行副作用或初始化逻辑；该语句位于在函数 `initialize` 中。

##### Lines 612-615 — Assign `res_arg`

```python
        res_arg = self.operation.rt_module.get_args(
            ctypes.byref(self.arguments),
            ctypes.c_void_p(int(device_workspace)),
        )
```
**EN:** Assigns `res_arg` from `self.operation.rt_module.get_args(ctypes.byref(self.arguments), ctypes.c_void_p(int(device_workspace)))`, establishing state in function `initialize`.
**CN:** 将 `res_arg` 赋值为 `self.operation.rt_module.get_args(ctypes.byref(self.arguments), ctypes.c_void_p(int(device_workspace)))`，用于在函数 `initialize` 中建立状态。

##### Line 616 — Assign `host_workspace`

```python
        host_workspace = bytearray(res_arg.contents)
```
**EN:** Assigns `host_workspace` from `bytearray(res_arg.contents)`, establishing state in function `initialize`.
**CN:** 将 `host_workspace` 赋值为 `bytearray(res_arg.contents)`，用于在函数 `initialize` 中建立状态。

##### Lines 618-621 — Assign `grid`

```python
        grid = self.operation.rt_module.get_grid_shape(
            ctypes.byref(self.arguments),
            ctypes.c_void_p(int(device_workspace)),
        )
```
**EN:** Assigns `grid` from `self.operation.rt_module.get_grid_shape(ctypes.byref(self.arguments), ctypes.c_void_p(int(device_workspace)))`, establishing state in function `initialize`.
**CN:** 将 `grid` 赋值为 `self.operation.rt_module.get_grid_shape(ctypes.byref(self.arguments), ctypes.c_void_p(int(device_workspace)))`，用于在函数 `initialize` 中建立状态。

##### Line 622 — Assign `block`

```python
        block = self.operation.rt_module.get_block_shape()
```
**EN:** Assigns `block` from `self.operation.rt_module.get_block_shape()`, establishing state in function `initialize`.
**CN:** 将 `block` 赋值为 `self.operation.rt_module.get_block_shape()`，用于在函数 `initialize` 中建立状态。

##### Line 624 — Assign `device_workspace`

```python
        device_workspace = None
```
**EN:** Assigns `device_workspace` from `None`, establishing state in function `initialize`.
**CN:** 将 `device_workspace` 赋值为 `None`，用于在函数 `initialize` 中建立状态。

##### Line 626 — Assign `self.host_workspace`

```python
        self.host_workspace = host_workspace
```
**EN:** Assigns `self.host_workspace` from `host_workspace`, establishing state in function `initialize`.
**CN:** 将 `self.host_workspace` 赋值为 `host_workspace`，用于在函数 `initialize` 中建立状态。

##### Line 627 — Assign `self.device_workspace`

```python
        self.device_workspace = device_workspace
```
**EN:** Assigns `self.device_workspace` from `device_workspace`, establishing state in function `initialize`.
**CN:** 将 `self.device_workspace` 赋值为 `device_workspace`，用于在函数 `initialize` 中建立状态。

##### Lines 628-632 — Assign `self.launch_config`

```python
        self.launch_config = LaunchConfiguration(
            [grid.x, grid.y, grid.z],
            [block.x, block.y, block.z],
            self.operation.rt_module.shared_memory_capacity,
        )
```
**EN:** Assigns `self.launch_config` from `LaunchConfiguration([grid.x, grid.y, grid.z], [block.x, block.y, block.z], self.operation.rt_module.shared_memory_capacity)`, establishing state in function `initialize`.
**CN:** 将 `self.launch_config` 赋值为 `LaunchConfiguration([grid.x, grid.y, grid.z], [block.x, block.y, block.z], self.operation.rt_module.shared_memory_capacity)`，用于在函数 `initialize` 中建立状态。

### Lines 635-664 — Function `GemmArguments`

```python
def GemmArguments(operation, problem_size, A, B, C, D, gemm_mode=GemmUniversalMode.Gemm, **kwargs):
    """
    Argument wrapper for GEMM in CUTLASS 2 or 3. It returns either 2x arguments
    or 3x arguments depending on the `arch` field specified in `operation`.

    :param operation: the GEMM operation to take the argument
    :type operation: :class:`cutlass_cppgen.backend.GemmOperationUniversal` |
     :class:`cutlass_cppgen.backend.GemmOperationGrouped`

    :param problem_size: GEMM problem size gemm(M, N, K)
    :type operation: :class:`cutlass_cppgen.shape.GemmCoord`

    :param A: tensor A
    :type A: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param B: tensor B
    :type B: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param C: tensor C
    :type C: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param D: tensor D
    :type D: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray

    :param gemm_mode: GEMM mode
    :type gemm_mode: :class:`cutlass_library.GemmUniversalMode`

    :param output_op: output operator, optional
    :type output_op: :class:`cutlass_cppgen.backend.LinearCombinationFunctorArguments`
    """
```
**EN:** Defines function `GemmArguments` with parameters `operation, problem_size, A, B, C, D, gemm_mode, **kwargs`. Purpose: Argument wrapper for GEMM in CUTLASS 2 or 3.
**CN:** 定义函数 `GemmArguments`，参数为 `operation, problem_size, A, B, C, D, gemm_mode, **kwargs`。 其用途：Argument wrapper for GEMM in CUTLASS 2 or 3.

#### Lines 665-670 — Conditional `operation.swizzling_functor == SwizzlingFunctor.StreamK`

```python
    if operation.swizzling_functor == SwizzlingFunctor.StreamK:
        if operation.api == ApiVersion.v3x:
            raise Exception("Stream K is currently only supported in CUTLASS 2.x")
        ArgClass = GemmArguments2xStreamK
    else:
        ArgClass = GemmArguments3x if operation.api == ApiVersion.v3x else GemmArguments2x
```
**EN:** Checks `operation.swizzling_functor == SwizzlingFunctor.StreamK` and selects the matching branch in function `GemmArguments`.
**CN:** 检查 `operation.swizzling_functor == SwizzlingFunctor.StreamK`，并在函数 `GemmArguments` 中选择匹配的分支。

#### Line 671 — Return

```python
    return ArgClass(operation, problem_size, A, B, C, D, gemm_mode, **kwargs)
```
**EN:** Returns `ArgClass(operation, problem_size, A, B, C, D, gemm_mode, **kwargs)` to the caller.
**CN:** 向调用方返回 `ArgClass(operation, problem_size, A, B, C, D, gemm_mode, **kwargs)`。

### Lines 674-702 — Class `GemmGroupedArguments`

```python
class GemmGroupedArguments:
    """
    Argument wrapper for GEMM Grouped. It encodes problem information and
    user-provide tensors into the kernel's argument

    :param operation: the GEMM Grouped operation to take the argument
    :type operation: :class:`cutlass_cppgen.backend.GemmOperationGrouped`

    :param problem_size: list of GEMM problem size gemm(M, N, K)
    :type operation: list[:class:`cutlass_cppgen.shape.GemmCoord`]

    :param A: list of tensor A
    :type A: list[cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray]

    :param B: list of tensor B
    :type B: list[cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray]

    :param C: list of tensor C
    :type C: list[cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray]

    :param D: list of tensor D
    :type D: list[cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray]

    :param output_op: output operator, optional
    :type output_op: :class:`cutlass_cppgen.backend.LinearCombinationFunctorArguments`

    :param stream: cuda stream, defaults to cuda.cuda.CUstream(0)
    :type stream: :class:`cuda.cuda.CUstream`
    """
```
**EN:** Declares class `GemmGroupedArguments` deriving from `object`. Purpose: Argument wrapper for GEMM Grouped.
**CN:** 声明类 `GemmGroupedArguments`，其基类为 `object`。 其用途：Argument wrapper for GEMM Grouped.

#### Lines 704-705 — Function `__init__`

```python
    def __init__(self, operation, problem_sizes, A, B, C, D, **kwargs):
        # Get number of problems in the group
```
**EN:** Defines function `__init__` with parameters `self, operation, problem_sizes, A, B, C, D, **kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, operation, problem_sizes, A, B, C, D, **kwargs`。

##### Line 706 — Assign `self.problem_count`

```python
        self.problem_count = len(problem_sizes)
```
**EN:** Assigns `self.problem_count` from `len(problem_sizes)`, establishing state in function `__init__`.
**CN:** 将 `self.problem_count` 赋值为 `len(problem_sizes)`，用于在函数 `__init__` 中建立状态。

##### Line 708 — Comment or spacing block

```python
        # Check the input arguments
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 709 — Assertion

```python
        assert len(A) == self.problem_count
```
**EN:** Asserts `len(A) == self.problem_count` to enforce an expected condition.
**CN:** 断言 `len(A) == self.problem_count`，用于保证预期条件成立。

##### Line 710 — Assertion

```python
        assert len(B) == self.problem_count
```
**EN:** Asserts `len(B) == self.problem_count` to enforce an expected condition.
**CN:** 断言 `len(B) == self.problem_count`，用于保证预期条件成立。

##### Line 711 — Assertion

```python
        assert len(C) == self.problem_count
```
**EN:** Asserts `len(C) == self.problem_count` to enforce an expected condition.
**CN:** 断言 `len(C) == self.problem_count`，用于保证预期条件成立。

##### Line 712 — Assertion

```python
        assert len(D) == self.problem_count
```
**EN:** Asserts `len(D) == self.problem_count` to enforce an expected condition.
**CN:** 断言 `len(D) == self.problem_count`，用于保证预期条件成立。

##### Line 714 — Assign `problem_size_host`

```python
        problem_size_host = []
```
**EN:** Assigns `problem_size_host` from `[]`, establishing state in function `__init__`.
**CN:** 将 `problem_size_host` 赋值为 `[]`，用于在函数 `__init__` 中建立状态。

##### Line 715 — Assign `self.ptr_A_host`

```python
        self.ptr_A_host = []
```
**EN:** Assigns `self.ptr_A_host` from `[]`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_A_host` 赋值为 `[]`，用于在函数 `__init__` 中建立状态。

##### Line 716 — Assign `self.ptr_B_host`

```python
        self.ptr_B_host = []
```
**EN:** Assigns `self.ptr_B_host` from `[]`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_B_host` 赋值为 `[]`，用于在函数 `__init__` 中建立状态。

##### Line 717 — Assign `self.ptr_C_host`

```python
        self.ptr_C_host = []
```
**EN:** Assigns `self.ptr_C_host` from `[]`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_C_host` 赋值为 `[]`，用于在函数 `__init__` 中建立状态。

##### Line 718 — Assign `self.ptr_D_host`

```python
        self.ptr_D_host = []
```
**EN:** Assigns `self.ptr_D_host` from `[]`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_D_host` 赋值为 `[]`，用于在函数 `__init__` 中建立状态。

##### Line 720 — Assign `lda_host`

```python
        lda_host = []
```
**EN:** Assigns `lda_host` from `[]`, establishing state in function `__init__`.
**CN:** 将 `lda_host` 赋值为 `[]`，用于在函数 `__init__` 中建立状态。

##### Line 721 — Assign `ldb_host`

```python
        ldb_host = []
```
**EN:** Assigns `ldb_host` from `[]`, establishing state in function `__init__`.
**CN:** 将 `ldb_host` 赋值为 `[]`，用于在函数 `__init__` 中建立状态。

##### Line 722 — Assign `ldc_host`

```python
        ldc_host = []
```
**EN:** Assigns `ldc_host` from `[]`, establishing state in function `__init__`.
**CN:** 将 `ldc_host` 赋值为 `[]`，用于在函数 `__init__` 中建立状态。

##### Line 723 — Assign `ldd_host`

```python
        ldd_host = []
```
**EN:** Assigns `ldd_host` from `[]`, establishing state in function `__init__`.
**CN:** 将 `ldd_host` 赋值为 `[]`，用于在函数 `__init__` 中建立状态。

##### Line 725 — Assign `self.partitions`

```python
        self.partitions = 1
```
**EN:** Assigns `self.partitions` from `1`, establishing state in function `__init__`.
**CN:** 将 `self.partitions` 赋值为 `1`，用于在函数 `__init__` 中建立状态。

##### Line 727 — Assign `self.operation`

```python
        self.operation = operation
```
**EN:** Assigns `self.operation` from `operation`, establishing state in function `__init__`.
**CN:** 将 `self.operation` 赋值为 `operation`，用于在函数 `__init__` 中建立状态。

##### Line 729 — Comment or spacing block

```python
        # Get the threadblock
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 730 — Assign `threadblock_shape`

```python
        threadblock_shape = operation.tile_description.threadblock_shape
```
**EN:** Assigns `threadblock_shape` from `operation.tile_description.threadblock_shape`, establishing state in function `__init__`.
**CN:** 将 `threadblock_shape` 赋值为 `operation.tile_description.threadblock_shape`，用于在函数 `__init__` 中建立状态。

##### Lines 731-735 — Assign `self.threadblock_shape`

```python
        self.threadblock_shape = GemmCoord(
            threadblock_shape[0],
            threadblock_shape[1],
            threadblock_shape[2],
        )
```
**EN:** Assigns `self.threadblock_shape` from `GemmCoord(threadblock_shape[0], threadblock_shape[1], threadblock_shape[2])`, establishing state in function `__init__`.
**CN:** 将 `self.threadblock_shape` 赋值为 `GemmCoord(threadblock_shape[0], threadblock_shape[1], threadblock_shape[2])`，用于在函数 `__init__` 中建立状态。

##### Line 736 — Assign `self.threadblock_swizzle`

```python
        self.threadblock_swizzle = operation.swizzling_functor
```
**EN:** Assigns `self.threadblock_swizzle` from `operation.swizzling_functor`, establishing state in function `__init__`.
**CN:** 将 `self.threadblock_swizzle` 赋值为 `operation.swizzling_functor`，用于在函数 `__init__` 中建立状态。

##### Line 738 — Assign `self.total_tiles`

```python
        self.total_tiles = 0
```
**EN:** Assigns `self.total_tiles` from `0`, establishing state in function `__init__`.
**CN:** 将 `self.total_tiles` 赋值为 `0`，用于在函数 `__init__` 中建立状态。

##### Line 740 — Assign `self.gemm_arguments`

```python
        self.gemm_arguments = []
```
**EN:** Assigns `self.gemm_arguments` from `[]`, establishing state in function `__init__`.
**CN:** 将 `self.gemm_arguments` 赋值为 `[]`，用于在函数 `__init__` 中建立状态。

##### Line 742 — Assign `self.stream`

```python
        self.stream = kwargs.get("stream", cuda.CUstream(0))
```
**EN:** Assigns `self.stream` from `kwargs.get('stream', cuda.CUstream(0))`, establishing state in function `__init__`.
**CN:** 将 `self.stream` 赋值为 `kwargs.get('stream', cuda.CUstream(0))`，用于在函数 `__init__` 中建立状态。

##### Line 744 — Comment or spacing block

```python
        # Process the input arguments
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 745-779 — Loop over `enumerate(problem_sizes)`

```python
        for idx, problem_size in enumerate(problem_sizes):
            M, N, K = problem_size.m, problem_size.n, problem_size.k
            temp_argument = GemmArguments2x(
                operation=operation,
                problem_size=GemmCoord(M, N, K),
                A=A[idx], B=B[idx], C=C[idx], D=D[idx])
            self.gemm_arguments.append(temp_argument)

            problem_size_host.append(
                [temp_argument.problem_size.m,
                 temp_argument.problem_size.n,
                 temp_argument.problem_size.k]
            )

            self.ptr_A_host.append(int(temp_argument.ptr_A))
            lda_host.append(temp_argument.lda)

            self.ptr_B_host.append(int(temp_argument.ptr_B))
            ldb_host.append(temp_argument.ldb)

            self.ptr_C_host.append(int(temp_argument.ptr_C))
            ldc_host.append(temp_argument.ldc)

            self.ptr_D_host.append(int(temp_argument.ptr_D))
            ldd_host.append(temp_argument.ldd)

            # Get number of tiles
            grid = self.operation.rt_module.get_grid_shape(
                self.operation.rt_module.get_tiled_shape(
                    temp_argument.problem_size.ctype,
                    self.threadblock_shape.ctype,
                    temp_argument.batch_count
                )
            )
            self.total_tiles += grid.x * grid.y * grid.z
```
**EN:** Iterates `(idx, problem_size)` over `enumerate(problem_sizes)` to repeat a processing step.
**CN:** 让 `(idx, problem_size)` 遍历 `enumerate(problem_sizes)`，从而重复执行处理步骤。

##### Line 781 — Assign `self.problem_size_buffer`

```python
        self.problem_size_buffer = todevice(problem_size_host, np.int32)
```
**EN:** Assigns `self.problem_size_buffer` from `todevice(problem_size_host, np.int32)`, establishing state in function `__init__`.
**CN:** 将 `self.problem_size_buffer` 赋值为 `todevice(problem_size_host, np.int32)`，用于在函数 `__init__` 中建立状态。

##### Line 782 — Assign `self.ptr_A_buffer`

```python
        self.ptr_A_buffer = todevice(self.ptr_A_host, np.int64)
```
**EN:** Assigns `self.ptr_A_buffer` from `todevice(self.ptr_A_host, np.int64)`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_A_buffer` 赋值为 `todevice(self.ptr_A_host, np.int64)`，用于在函数 `__init__` 中建立状态。

##### Line 783 — Assign `self.ptr_B_buffer`

```python
        self.ptr_B_buffer = todevice(self.ptr_B_host, np.int64)
```
**EN:** Assigns `self.ptr_B_buffer` from `todevice(self.ptr_B_host, np.int64)`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_B_buffer` 赋值为 `todevice(self.ptr_B_host, np.int64)`，用于在函数 `__init__` 中建立状态。

##### Line 784 — Assign `self.ptr_C_buffer`

```python
        self.ptr_C_buffer = todevice(self.ptr_C_host, np.int64)
```
**EN:** Assigns `self.ptr_C_buffer` from `todevice(self.ptr_C_host, np.int64)`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_C_buffer` 赋值为 `todevice(self.ptr_C_host, np.int64)`，用于在函数 `__init__` 中建立状态。

##### Line 785 — Assign `self.ptr_D_buffer`

```python
        self.ptr_D_buffer = todevice(self.ptr_D_host, np.int64)
```
**EN:** Assigns `self.ptr_D_buffer` from `todevice(self.ptr_D_host, np.int64)`, establishing state in function `__init__`.
**CN:** 将 `self.ptr_D_buffer` 赋值为 `todevice(self.ptr_D_host, np.int64)`，用于在函数 `__init__` 中建立状态。

##### Line 787 — Assign `self.lda_buffer`

```python
        self.lda_buffer = todevice(lda_host, np.int64)
```
**EN:** Assigns `self.lda_buffer` from `todevice(lda_host, np.int64)`, establishing state in function `__init__`.
**CN:** 将 `self.lda_buffer` 赋值为 `todevice(lda_host, np.int64)`，用于在函数 `__init__` 中建立状态。

##### Line 788 — Assign `self.ldb_buffer`

```python
        self.ldb_buffer = todevice(ldb_host, np.int64)
```
**EN:** Assigns `self.ldb_buffer` from `todevice(ldb_host, np.int64)`, establishing state in function `__init__`.
**CN:** 将 `self.ldb_buffer` 赋值为 `todevice(ldb_host, np.int64)`，用于在函数 `__init__` 中建立状态。

##### Line 789 — Assign `self.ldc_buffer`

```python
        self.ldc_buffer = todevice(ldc_host, np.int64)
```
**EN:** Assigns `self.ldc_buffer` from `todevice(ldc_host, np.int64)`, establishing state in function `__init__`.
**CN:** 将 `self.ldc_buffer` 赋值为 `todevice(ldc_host, np.int64)`，用于在函数 `__init__` 中建立状态。

##### Line 790 — Assign `self.ldd_buffer`

```python
        self.ldd_buffer = todevice(ldd_host, np.int64)
```
**EN:** Assigns `self.ldd_buffer` from `todevice(ldd_host, np.int64)`, establishing state in function `__init__`.
**CN:** 将 `self.ldd_buffer` 赋值为 `todevice(ldd_host, np.int64)`，用于在函数 `__init__` 中建立状态。

##### Lines 792-797 — Conditional `'output_op' in kwargs.keys()`

```python
        if "output_op" in kwargs.keys():
            self.alpha = kwargs["output_op"].alpha
            self.beta = kwargs["output_op"].beta
        else:
            self.alpha = 1.0
            self.beta = 0.0
```
**EN:** Checks `'output_op' in kwargs.keys()` and selects the matching branch in function `__init__`.
**CN:** 检查 `'output_op' in kwargs.keys()`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 799-802 — Conditional `'output_op' in kwargs.keys()`

```python
        if "output_op" in kwargs.keys():
            self.output_op = kwargs["output_op"]
        else:
            self.output_op = self.operation.epilogue_type(1.0, 0.0)
```
**EN:** Checks `'output_op' in kwargs.keys()` and selects the matching branch in function `__init__`.
**CN:** 检查 `'output_op' in kwargs.keys()`，并在函数 `__init__` 中选择匹配的分支。

##### Line 804 — Comment or spacing block

```python
        # Get host problem size
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 805 — Assign `self.host_problem_size_ptr`

```python
        self.host_problem_size_ptr = np.array(problem_size_host, dtype=np.int32).__array_interface__["data"][0]
```
**EN:** Assigns `self.host_problem_size_ptr` from `np.array(problem_size_host, dtype=np.int32).__array_interface__['data'][0]`, establishing state in function `__init__`.
**CN:** 将 `self.host_problem_size_ptr` 赋值为 `np.array(problem_size_host, dtype=np.int32).__array_interface__['data'][0]`，用于在函数 `__init__` 中建立状态。

##### Line 807 — Assign `self.arguments`

```python
        self.arguments = self.get_arguments()
```
**EN:** Assigns `self.arguments` from `self.get_arguments()`, establishing state in function `__init__`.
**CN:** 将 `self.arguments` 赋值为 `self.get_arguments()`，用于在函数 `__init__` 中建立状态。

##### Line 809 — Call `self.initialize`

```python
        self.initialize()
```
**EN:** Calls `self.initialize` for side effects or initialization work in function `__init__`.
**CN:** 调用 `self.initialize` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

#### Line 811 — Function `get_arguments`

```python
    def get_arguments(self):
```
**EN:** Defines function `get_arguments` with parameters `self`.
**CN:** 定义函数 `get_arguments`，参数为 `self`。

##### Lines 812-826 — Return

```python
        return self.operation.argument_type(
            self.problem_size_buffer.ptr,
            self.problem_count,
            self.total_tiles,
            self.output_op,
            self.ptr_A_buffer.ptr,
            self.ptr_B_buffer.ptr,
            self.ptr_C_buffer.ptr,
            self.ptr_D_buffer.ptr,
            self.lda_buffer.ptr,
            self.ldb_buffer.ptr,
            self.ldc_buffer.ptr,
            self.ldd_buffer.ptr,
            ctypes.c_void_p(int(self.host_problem_size_ptr)),
        )
```
**EN:** Returns `self.operation.argument_type(self.problem_size_buffer.ptr, self.problem_count, self.total_tiles, self.output_op, self.ptr_A_buffer.ptr, self.ptr_B_buffer.ptr...` to the caller.
**CN:** 向调用方返回 `self.operation.argument_type(self.problem_size_buffer.ptr, self.problem_count, self.total_tiles, self.output_op, self.ptr_A_buffer.ptr, self.ptr_B_buffer.ptr...`。

#### Lines 828-829 — Function `initialize`

```python
    def initialize(self):
        # Get launch configuration
```
**EN:** Defines function `initialize` with parameters `self`.
**CN:** 定义函数 `initialize`，参数为 `self`。

##### Line 830 — Assign `launch_config`

```python
        launch_config = self.operation.rt_module.plan(self)
```
**EN:** Assigns `launch_config` from `self.operation.rt_module.plan(self)`, establishing state in function `initialize`.
**CN:** 将 `launch_config` 赋值为 `self.operation.rt_module.plan(self)`，用于在函数 `initialize` 中建立状态。

##### Line 832 — Comment or spacing block

```python
        # Get the host and evice workspace
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 833 — Assign `device_workspace_size`

```python
        device_workspace_size = self.operation.rt_module.get_device_workspace_size(self)
```
**EN:** Assigns `device_workspace_size` from `self.operation.rt_module.get_device_workspace_size(self)`, establishing state in function `initialize`.
**CN:** 将 `device_workspace_size` 赋值为 `self.operation.rt_module.get_device_workspace_size(self)`，用于在函数 `initialize` 中建立状态。

##### Lines 835-841 — Conditional `device_workspace_size > 0`

```python
        if device_workspace_size > 0:
            self.workspace_buffer = device_mem_alloc(device_workspace_size)
            workspace_ptr = self.workspace_buffer.ptr
            err, = cuda.cuMemsetD32(
                workspace_ptr, 0, device_workspace_size // 4)
        else:
            workspace_ptr = None
```
**EN:** Checks `device_workspace_size > 0` and selects the matching branch in function `initialize`.
**CN:** 检查 `device_workspace_size > 0`，并在函数 `initialize` 中选择匹配的分支。

##### Lines 843-847 — Conditional `self.operation.precompute_mode == SchedulerMode.Host`

```python
        if self.operation.precompute_mode == SchedulerMode.Host:
            device_workspace_ptr = self.operation.rt_module.host_precompute(
                self, self.operation.rt_module.get_workspace_size(self),)
        else:
            device_workspace_ptr = 0
```
**EN:** Checks `self.operation.precompute_mode == SchedulerMode.Host` and selects the matching branch in function `initialize`.
**CN:** 检查 `self.operation.precompute_mode == SchedulerMode.Host`，并在函数 `initialize` 中选择匹配的分支。

##### Lines 849-853 — Assign `result`

```python
        result = self.operation.rt_module.get_args(
            ctypes.byref(self.arguments),
            self.total_tiles,
            ctypes.c_void_p(int(device_workspace_ptr)),
        )
```
**EN:** Assigns `result` from `self.operation.rt_module.get_args(ctypes.byref(self.arguments), self.total_tiles, ctypes.c_void_p(int(device_workspace_ptr)))`, establishing state in function `initialize`.
**CN:** 将 `result` 赋值为 `self.operation.rt_module.get_args(ctypes.byref(self.arguments), self.total_tiles, ctypes.c_void_p(int(device_workspace_ptr)))`，用于在函数 `initialize` 中建立状态。

##### Line 854 — Assign `host_workspace`

```python
        host_workspace = bytearray(result.contents)
```
**EN:** Assigns `host_workspace` from `bytearray(result.contents)`, establishing state in function `initialize`.
**CN:** 将 `host_workspace` 赋值为 `bytearray(result.contents)`，用于在函数 `initialize` 中建立状态。

##### Line 856 — Assign `device_workspace`

```python
        device_workspace = None
```
**EN:** Assigns `device_workspace` from `None`, establishing state in function `initialize`.
**CN:** 将 `device_workspace` 赋值为 `None`，用于在函数 `initialize` 中建立状态。

##### Line 858 — Assign `self.host_workspace`

```python
        self.host_workspace = host_workspace
```
**EN:** Assigns `self.host_workspace` from `host_workspace`, establishing state in function `initialize`.
**CN:** 将 `self.host_workspace` 赋值为 `host_workspace`，用于在函数 `initialize` 中建立状态。

##### Line 859 — Assign `self.device_workspace`

```python
        self.device_workspace = device_workspace
```
**EN:** Assigns `self.device_workspace` from `device_workspace`, establishing state in function `initialize`.
**CN:** 将 `self.device_workspace` 赋值为 `device_workspace`，用于在函数 `initialize` 中建立状态。

##### Line 860 — Assign `self.launch_config`

```python
        self.launch_config = launch_config
```
**EN:** Assigns `self.launch_config` from `launch_config`, establishing state in function `initialize`.
**CN:** 将 `self.launch_config` 赋值为 `launch_config`，用于在函数 `initialize` 中建立状态。

#### Line 862 — Function `sync`

```python
    def sync(self):
```
**EN:** Defines function `sync` with parameters `self`.
**CN:** 定义函数 `sync`，参数为 `self`。

##### Line 863 — Assign `err`

```python
        err, = cudart.cudaDeviceSynchronize()
```
**EN:** Assigns `err` from `cudart.cudaDeviceSynchronize()`, establishing state in function `sync`.
**CN:** 将 `err` 赋值为 `cudart.cudaDeviceSynchronize()`，用于在函数 `sync` 中建立状态。

##### Lines 864-865 — Conditional `err != cuda.CUresult.CUDA_SUCCESS`

```python
        if err != cuda.CUresult.CUDA_SUCCESS:
            raise RuntimeError("CUDA Error %s" % str(err))
```
**EN:** Checks `err != cuda.CUresult.CUDA_SUCCESS` and selects the matching branch in function `sync`.
**CN:** 检查 `err != cuda.CUresult.CUDA_SUCCESS`，并在函数 `sync` 中选择匹配的分支。

##### Lines 866-867 — Loop over `self.gemm_arguments`

```python
        for arg in self.gemm_arguments:
            arg.sync(stream_sync=False)
```
**EN:** Iterates `arg` over `self.gemm_arguments` to repeat a processing step.
**CN:** 让 `arg` 遍历 `self.gemm_arguments`，从而重复执行处理步骤。

### Lines 870-872 — Comment or spacing block

```python
################################################################################
# Base class for GEMM runtime module
################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Lines 875-878 — Class `GemmRTbase`

```python
class GemmRTbase(ExecutableOperation):
    """
    GemmRT manages the CUTLASS runtime components
    """
```
**EN:** Declares class `GemmRTbase` deriving from `ExecutableOperation`. Purpose: GemmRT manages the CUTLASS runtime components
**CN:** 声明类 `GemmRTbase`，其基类为 `ExecutableOperation`。 其用途：GemmRT manages the CUTLASS runtime components

#### Lines 880-894 — Assign `KernelTemplate`

```python
    KernelTemplate = r"""
extern "C"
__global__ void
${operation_name}(${operation_name}${operation_suffix}::Params params) {

  // Dynamic shared memory base pointer
  extern __shared__ int SharedStorageBase[];

  // Declare pointer to dynamic shared memory.
  ${operation_name}${operation_suffix}::SharedStorage *shared_storage =
      reinterpret_cast<${operation_name}${operation_suffix}::SharedStorage *>(SharedStorageBase);

  ${operation_name}${operation_suffix}::invoke(params, *shared_storage);
}
  """
```
**EN:** Assigns `KernelTemplate` from `'\nextern "C"\n__global__ void\n${operation_name}(${operation_name}${operation_suffix}::Params params) {\n\n // Dynamic shared memory base pointer\n extern _...`, establishing state in class `GemmRTbase`.
**CN:** 将 `KernelTemplate` 赋值为 `'\nextern "C"\n__global__ void\n${operation_name}(${operation_name}${operation_suffix}::Params params) {\n\n // Dynamic shared memory base pointer\n extern _...`，用于在类 `GemmRTbase` 中建立状态。

#### Line 896 — Function `__init__`

```python
    def __init__(self, operation: "GemmOperation"):
```
**EN:** Defines function `__init__` with parameters `self, operation`.
**CN:** 定义函数 `__init__`，参数为 `self, operation`。

##### Line 897 — Call `super().__init__`

```python
        super().__init__(operation)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 899 — Assign `self.operation`

```python
        self.operation = operation
```
**EN:** Assigns `self.operation` from `operation`, establishing state in function `__init__`.
**CN:** 将 `self.operation` 赋值为 `operation`，用于在函数 `__init__` 中建立状态。

##### Line 900 — Assign `threadblock_shape`

```python
        threadblock_shape = operation.tile_description.threadblock_shape
```
**EN:** Assigns `threadblock_shape` from `operation.tile_description.threadblock_shape`, establishing state in function `__init__`.
**CN:** 将 `threadblock_shape` 赋值为 `operation.tile_description.threadblock_shape`，用于在函数 `__init__` 中建立状态。

##### Lines 901-902 — Assign `self.threadblock_shape`

```python
        self.threadblock_shape = GemmCoord(
            threadblock_shape[0], threadblock_shape[1], threadblock_shape[2])
```
**EN:** Assigns `self.threadblock_shape` from `GemmCoord(threadblock_shape[0], threadblock_shape[1], threadblock_shape[2])`, establishing state in function `__init__`.
**CN:** 将 `self.threadblock_shape` 赋值为 `GemmCoord(threadblock_shape[0], threadblock_shape[1], threadblock_shape[2])`，用于在函数 `__init__` 中建立状态。

##### Line 903 — Assign `self.threadblock_swizzle`

```python
        self.threadblock_swizzle = operation.swizzling_functor
```
**EN:** Assigns `self.threadblock_swizzle` from `operation.swizzling_functor`, establishing state in function `__init__`.
**CN:** 将 `self.threadblock_swizzle` 赋值为 `operation.swizzling_functor`，用于在函数 `__init__` 中建立状态。

##### Line 905 — Comment or spacing block

```python
        # Threads per threadblock
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 906 — Assign `self.threads`

```python
        self.threads = operation.tile_description.num_threads
```
**EN:** Assigns `self.threads` from `operation.tile_description.num_threads`, establishing state in function `__init__`.
**CN:** 将 `self.threads` 赋值为 `operation.tile_description.num_threads`，用于在函数 `__init__` 中建立状态。

#### Line 908 — Function `emit`

```python
    def emit(self):
```
**EN:** Defines function `emit` with parameters `self`.
**CN:** 定义函数 `emit`，参数为 `self`。

##### Line 909 — Return

```python
        return self.emitter.emit(self.operation)
```
**EN:** Returns `self.emitter.emit(self.operation)` to the caller.
**CN:** 向调用方返回 `self.emitter.emit(self.operation)`。

#### Line 911 — Function `can_implement`

```python
    def can_implement(self, configuration, arguments):
```
**EN:** Defines function `can_implement` with parameters `self, configuration, arguments`.
**CN:** 定义函数 `can_implement`，参数为 `self, configuration, arguments`。

##### Line 912 — Raise exception

```python
        raise NotImplementedError()
```
**EN:** Raises `NotImplementedError()` to signal an error or unsupported state.
**CN:** 抛出 `NotImplementedError()`，用于报告错误或不支持的状态。

#### Line 914 — Function `get_host_workspace_size`

```python
    def get_host_workspace_size(self, arguments):
```
**EN:** Defines function `get_host_workspace_size` with parameters `self, arguments`.
**CN:** 定义函数 `get_host_workspace_size`，参数为 `self, arguments`。

##### Line 915 — Raise exception

```python
        raise NotImplementedError()
```
**EN:** Raises `NotImplementedError()` to signal an error or unsupported state.
**CN:** 抛出 `NotImplementedError()`，用于报告错误或不支持的状态。

#### Line 917 — Function `get_device_workspace_size`

```python
    def get_device_workspace_size(self, arguments):
```
**EN:** Defines function `get_device_workspace_size` with parameters `self, arguments`.
**CN:** 定义函数 `get_device_workspace_size`，参数为 `self, arguments`。

##### Line 918 — Return

```python
        return 0
```
**EN:** Returns `0` to the caller.
**CN:** 向调用方返回 `0`。

#### Line 920 — Function `initialize`

```python
    def initialize(self):
```
**EN:** Defines function `initialize` with parameters `self`.
**CN:** 定义函数 `initialize`，参数为 `self`。

##### Lines 921-922 — Conditional `is_intel_xe_arch(self.operation.arch)`

```python
        if is_intel_xe_arch(self.operation.arch):
            return
```
**EN:** Checks `is_intel_xe_arch(self.operation.arch)` and selects the matching branch in function `initialize`.
**CN:** 检查 `is_intel_xe_arch(self.operation.arch)`，并在函数 `initialize` 中选择匹配的分支。

##### Lines 924-927 — Assign `err`

```python
        err, = cuda.cuFuncSetAttribute(
            self.kernel,
            attrib=cuda.CUfunction_attribute.CU_FUNC_ATTRIBUTE_MAX_DYNAMIC_SHARED_SIZE_BYTES,
            value=self.shared_memory_capacity)
```
**EN:** Assigns `err` from `cuda.cuFuncSetAttribute(self.kernel, attrib=cuda.CUfunction_attribute.CU_FUNC_ATTRIBUTE_MAX_DYNAMIC_SHARED_SIZE_BYTES, value=self.shared_memory_capacity)`, establishing state in function `initialize`.
**CN:** 将 `err` 赋值为 `cuda.cuFuncSetAttribute(self.kernel, attrib=cuda.CUfunction_attribute.CU_FUNC_ATTRIBUTE_MAX_DYNAMIC_SHARED_SIZE_BYTES, value=self.shared_memory_capacity)`，用于在函数 `initialize` 中建立状态。

##### Lines 928-931 — Conditional `err != cuda.CUresult.CUDA_SUCCESS`

```python
        if err != cuda.CUresult.CUDA_SUCCESS:
            raise RuntimeError(
                f"CUDA error on call to cuFuncSetAttribute: {cuda.cuGetErrorString(err)[1]}"
            )
```
**EN:** Checks `err != cuda.CUresult.CUDA_SUCCESS` and selects the matching branch in function `initialize`.
**CN:** 检查 `err != cuda.CUresult.CUDA_SUCCESS`，并在函数 `initialize` 中选择匹配的分支。

### Lines 934-936 — Comment or spacing block

```python
################################################################################
# Runtime module for GEMM Universal
################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Lines 939-942 — Class `GemmRTUniversal`

```python
class GemmRTUniversal(GemmRTbase):
    """
    GemmRTUniversal manages the CUTLASS runtime components
    """
```
**EN:** Declares class `GemmRTUniversal` deriving from `GemmRTbase`. Purpose: GemmRTUniversal manages the CUTLASS runtime components
**CN:** 声明类 `GemmRTUniversal`，其基类为 `GemmRTbase`。 其用途：GemmRTUniversal manages the CUTLASS runtime components

#### Lines 944-985 — Assign `HostTemplate`

```python
    HostTemplate = r"""
extern "C" {
  // Get the size of params in bytes
  int ${operation_name}_get_param_size(){
    return sizeof(${operation_name}${operation_suffix}::Params);
  }

  // Get the size of dynamic shared memory in bytes
  int ${operation_name}_shared_memory_size() {
    return int(sizeof(${operation_name}${operation_suffix}::SharedStorage));
  }

  // Get the params as byte array
  char* ${operation_name}_get_params(${operation_name}_base::Arguments* argument, int* workspace){
    ${operation_name}_base::Params* params;
    params = new ${operation_name}_base::Params(*argument,
                                                -1, // SM count. Only used for stream-K
                                                -1  // Occupancy. Only used for stream-K
                                                );

    // Semaphore holds the pointer to the workspace in the Params struct
    params->semaphore = workspace;

    char *bytes = ((char*)(params));
    char *output = new char[sizeof(${operation_name}_base::Params)];
    for (unsigned int i = 0; i < sizeof(${operation_name}_base::Params); i ++)
        output[i] = bytes[i];

    return output;
  }

  cutlass::gemm::GemmCoord ${operation_name}_get_tiled_shape(
    cutlass::gemm::GemmCoord problem_size, cutlass::gemm::GemmCoord tile_size, int split_k_slices) {
    return ${operation_name}_base::ThreadblockSwizzle::get_tiled_shape(
        problem_size, tile_size, split_k_slices);
  }

  dim3 ${operation_name}_get_grid_shape(cutlass::gemm::GemmCoord tiled_shape) {
    return ${operation_name}_base::ThreadblockSwizzle::get_grid_shape(tiled_shape);
  }
}
  """
```
**EN:** Assigns `HostTemplate` from `'\nextern "C" {\n // Get the size of params in bytes\n int ${operation_name}_get_param_size(){\n return sizeof(${operation_name}${operation_suffix}::Params);...`, establishing state in class `GemmRTUniversal`.
**CN:** 将 `HostTemplate` 赋值为 `'\nextern "C" {\n // Get the size of params in bytes\n int ${operation_name}_get_param_size(){\n return sizeof(${operation_name}${operation_suffix}::Params);...`，用于在类 `GemmRTUniversal` 中建立状态。

#### Line 987 — Function `__init__`

```python
    def __init__(self, operation):
```
**EN:** Defines function `__init__` with parameters `self, operation`.
**CN:** 定义函数 `__init__`，参数为 `self, operation`。

##### Line 988 — Call `super(GemmRTUniversal, self).__init__`

```python
        super(GemmRTUniversal, self).__init__(operation)
```
**EN:** Calls `super(GemmRTUniversal, self).__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super(GemmRTUniversal, self).__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Lines 989-992 — Assign `self.extra_funcs`

```python
        self.extra_funcs = {
            "get_tiled_shape": GemmCoord_,
            "get_grid_shape": dim3_,
        }
```
**EN:** Assigns `self.extra_funcs` from `{'get_tiled_shape': GemmCoord_, 'get_grid_shape': dim3_}`, establishing state in function `__init__`.
**CN:** 将 `self.extra_funcs` 赋值为 `{'get_tiled_shape': GemmCoord_, 'get_grid_shape': dim3_}`，用于在函数 `__init__` 中建立状态。

##### Lines 993-994 — Assign `self.emitter`

```python
        self.emitter = EmitGemmUniversalInstance(
            "_type", operation.direct_store)
```
**EN:** Assigns `self.emitter` from `EmitGemmUniversalInstance('_type', operation.direct_store)`, establishing state in function `__init__`.
**CN:** 将 `self.emitter` 赋值为 `EmitGemmUniversalInstance('_type', operation.direct_store)`，用于在函数 `__init__` 中建立状态。

##### Line 996 — Assign `self.argument_type, self.epilogue_type`

```python
        self.argument_type, self.epilogue_type = get_gemm_arguments(operation.epilogue_functor)
```
**EN:** Assigns `self.argument_type, self.epilogue_type` from `get_gemm_arguments(operation.epilogue_functor)`, establishing state in function `__init__`.
**CN:** 将 `self.argument_type, self.epilogue_type` 赋值为 `get_gemm_arguments(operation.epilogue_functor)`，用于在函数 `__init__` 中建立状态。

##### Lines 997-1000 — Assign `self.argtype`

```python
        self.argtype = [
            ctypes.POINTER(self.argument_type),
            ctypes.POINTER(GemmCoord_), ctypes.c_int, ctypes.c_void_p
        ]
```
**EN:** Assigns `self.argtype` from `[ctypes.POINTER(self.argument_type), ctypes.POINTER(GemmCoord_), ctypes.c_int, ctypes.c_void_p]`, establishing state in function `__init__`.
**CN:** 将 `self.argtype` 赋值为 `[ctypes.POINTER(self.argument_type), ctypes.POINTER(GemmCoord_), ctypes.c_int, ctypes.c_void_p]`，用于在函数 `__init__` 中建立状态。

#### Line 1002 — Function `plan`

```python
    def plan(self, arguments):
```
**EN:** Defines function `plan` with parameters `self, arguments`.
**CN:** 定义函数 `plan`，参数为 `self, arguments`。

##### Lines 1003-1007 — Assign `grid`

```python
        grid = self.get_tiled_shape(
            arguments.problem_size.ctype,
            self.threadblock_shape.ctype,
            arguments.batch_count
        )
```
**EN:** Assigns `grid` from `self.get_tiled_shape(arguments.problem_size.ctype, self.threadblock_shape.ctype, arguments.batch_count)`, establishing state in function `plan`.
**CN:** 将 `grid` 赋值为 `self.get_tiled_shape(arguments.problem_size.ctype, self.threadblock_shape.ctype, arguments.batch_count)`，用于在函数 `plan` 中建立状态。

##### Line 1009 — Assign `gemm_k_size`

```python
        gemm_k_size = arguments.problem_size.k
```
**EN:** Assigns `gemm_k_size` from `arguments.problem_size.k`, establishing state in function `plan`.
**CN:** 将 `gemm_k_size` 赋值为 `arguments.problem_size.k`，用于在函数 `plan` 中建立状态。

##### Lines 1010-1019 — Conditional `arguments.gemm_mode in [GemmUniversalMode.Gemm, GemmUniversalMode.GemmSplitKParallel]`

```python
        if arguments.gemm_mode in [GemmUniversalMode.Gemm, GemmUniversalMode.GemmSplitKParallel]:
            alignk = max(max(128 // DataTypeSize[self.operation.A.element],
                         128 // DataTypeSize[self.operation.B.element]), 1)

            gemm_k_size = (((arguments.problem_size.k + arguments.batch_count - 1) //
                           arguments.batch_count + alignk - 1) // alignk) * alignk

            if gemm_k_size:
                grid_z = (arguments.problem_size.k + gemm_k_size - 1) // gemm_k_size
                grid = GemmCoord(grid.m, grid.n, grid_z).ctype
```
**EN:** Checks `arguments.gemm_mode in [GemmUniversalMode.Gemm, GemmUniversalMode.GemmSplitKParallel]` and selects the matching branch in function `plan`.
**CN:** 检查 `arguments.gemm_mode in [GemmUniversalMode.Gemm, GemmUniversalMode.GemmSplitKParallel]`，并在函数 `plan` 中选择匹配的分支。

##### Line 1021 — Assign `arguments.grid_tiled_shape`

```python
        arguments.grid_tiled_shape = dim3_(grid.m, grid.n, grid.k)
```
**EN:** Assigns `arguments.grid_tiled_shape` from `dim3_(grid.m, grid.n, grid.k)`, establishing state in function `plan`.
**CN:** 将 `arguments.grid_tiled_shape` 赋值为 `dim3_(grid.m, grid.n, grid.k)`，用于在函数 `plan` 中建立状态。

##### Line 1022 — Assign `grid`

```python
        grid = self.get_grid_shape(grid)
```
**EN:** Assigns `grid` from `self.get_grid_shape(grid)`, establishing state in function `plan`.
**CN:** 将 `grid` 赋值为 `self.get_grid_shape(grid)`，用于在函数 `plan` 中建立状态。

##### Line 1023 — Assign `arguments.gemm_k_size`

```python
        arguments.gemm_k_size = gemm_k_size
```
**EN:** Assigns `arguments.gemm_k_size` from `gemm_k_size`, establishing state in function `plan`.
**CN:** 将 `arguments.gemm_k_size` 赋值为 `gemm_k_size`，用于在函数 `plan` 中建立状态。

##### Lines 1024-1027 — Return

```python
        return LaunchConfiguration(
            [grid.x, grid.y, grid.z],
            [self.threads, 1, 1],
            self.shared_memory_capacity)
```
**EN:** Returns `LaunchConfiguration([grid.x, grid.y, grid.z], [self.threads, 1, 1], self.shared_memory_capacity)` to the caller.
**CN:** 向调用方返回 `LaunchConfiguration([grid.x, grid.y, grid.z], [self.threads, 1, 1], self.shared_memory_capacity)`。

#### Line 1029 — Function `get_device_workspace_size`

```python
    def get_device_workspace_size(self, arguments: GemmArguments):
```
**EN:** Defines function `get_device_workspace_size` with parameters `self, arguments`.
**CN:** 定义函数 `get_device_workspace_size`，参数为 `self, arguments`。

##### Line 1030 — Assign `workspace_bytes`

```python
        workspace_bytes = 0
```
**EN:** Assigns `workspace_bytes` from `0`, establishing state in function `get_device_workspace_size`.
**CN:** 将 `workspace_bytes` 赋值为 `0`，用于在函数 `get_device_workspace_size` 中建立状态。

##### Lines 1031-1036 — Conditional `arguments.gemm_mode == GemmUniversalMode.GemmSplitKParallel`

```python
        if arguments.gemm_mode == GemmUniversalMode.GemmSplitKParallel:
            workspace_bytes = (DataTypeSize[arguments.operation.C.element]
             * arguments.batched_stride_D * arguments.grid_tiled_shape.z // 8)
        elif (arguments.gemm_mode == GemmUniversalMode.Gemm and
            arguments.split_k_slices > 1):
            workspace_bytes = 4 * arguments.grid_tiled_shape.x * arguments.grid_tiled_shape.y
```
**EN:** Checks `arguments.gemm_mode == GemmUniversalMode.GemmSplitKParallel` and selects the matching branch in function `get_device_workspace_size`.
**CN:** 检查 `arguments.gemm_mode == GemmUniversalMode.GemmSplitKParallel`，并在函数 `get_device_workspace_size` 中选择匹配的分支。

##### Line 1038 — Return

```python
        return workspace_bytes
```
**EN:** Returns `workspace_bytes` to the caller.
**CN:** 向调用方返回 `workspace_bytes`。

### Lines 1041-1044 — Class `GemmRTUniversalStreamK`

```python
class GemmRTUniversalStreamK(GemmRTUniversal):
    """
    Manages the CUTLASS runtime components for 2.x stream K kernels
    """
```
**EN:** Declares class `GemmRTUniversalStreamK` deriving from `GemmRTUniversal`. Purpose: Manages the CUTLASS runtime components for 2.x stream K kernels
**CN:** 声明类 `GemmRTUniversalStreamK`，其基类为 `GemmRTUniversal`。 其用途：Manages the CUTLASS runtime components for 2.x stream K kernels

#### Lines 1046-1086 — Assign `HostTemplate`

```python
    HostTemplate = r"""
extern "C" {
  // Get the size of params in bytes
  int ${operation_name}_get_param_size(){
    return sizeof(${operation_name}${operation_suffix}::Params);
  }

  // Get the size of dynamic shared memory in bytes
  int ${operation_name}_shared_memory_size() {
    return int(sizeof(${operation_name}${operation_suffix}::SharedStorage));
  }

  using GemmType = ${operation_name}_base;

  // Get the params as byte array
  char* ${operation_name}_get_params(GemmType::Arguments* argument, int* workspace,
                                     int sm_count, int occupancy) {
    GemmType::Params* params;
    params = new GemmType::Params(*argument, sm_count, occupancy);

    params->init_workspace(workspace);

    char *bytes = ((char*)(params));
    char *output = new char[sizeof(GemmType::Params)];
    for (unsigned int i = 0; i < sizeof(GemmType::Params); i ++)
        output[i] = bytes[i];

    return output;
  }

  dim3 ${operation_name}_get_grid_shape(GemmType::Arguments* args, int device_sms, int sm_occupancy) {
    typename GemmType::Params params(*args, device_sms, sm_occupancy);
    return params.get_grid_dims();
  }

  uint64_t ${operation_name}_get_kernel_workspace_size(GemmType::Arguments* args, int device_sms, int sm_occupancy) {
    typename GemmType::Params params(*args, device_sms, sm_occupancy);
    return params.get_workspace_size();
  }
}
  """
```
**EN:** Assigns `HostTemplate` from `'\nextern "C" {\n // Get the size of params in bytes\n int ${operation_name}_get_param_size(){\n return sizeof(${operation_name}${operation_suffix}::Params);...`, establishing state in class `GemmRTUniversalStreamK`.
**CN:** 将 `HostTemplate` 赋值为 `'\nextern "C" {\n // Get the size of params in bytes\n int ${operation_name}_get_param_size(){\n return sizeof(${operation_name}${operation_suffix}::Params);...`，用于在类 `GemmRTUniversalStreamK` 中建立状态。

#### Line 1088 — Function `__init__`

```python
    def __init__(self, operation: "GemmOperation"):
```
**EN:** Defines function `__init__` with parameters `self, operation`.
**CN:** 定义函数 `__init__`，参数为 `self, operation`。

##### Line 1089 — Call `super(GemmRTUniversalStreamK, self).__init__`

```python
        super(GemmRTUniversalStreamK, self).__init__(operation)
```
**EN:** Calls `super(GemmRTUniversalStreamK, self).__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super(GemmRTUniversalStreamK, self).__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Lines 1090-1093 — Assign `self.extra_funcs`

```python
        self.extra_funcs = {
            "get_grid_shape": GemmCoord_,
            "get_kernel_workspace_size": ctypes.c_uint64,
        }
```
**EN:** Assigns `self.extra_funcs` from `{'get_grid_shape': GemmCoord_, 'get_kernel_workspace_size': ctypes.c_uint64}`, establishing state in function `__init__`.
**CN:** 将 `self.extra_funcs` 赋值为 `{'get_grid_shape': GemmCoord_, 'get_kernel_workspace_size': ctypes.c_uint64}`，用于在函数 `__init__` 中建立状态。

##### Line 1094 — Assign `self._occupancy`

```python
        self._occupancy = None
```
**EN:** Assigns `self._occupancy` from `None`, establishing state in function `__init__`.
**CN:** 将 `self._occupancy` 赋值为 `None`，用于在函数 `__init__` 中建立状态。

##### Line 1095 — Assign `self.argument_type, self.epilogue_type`

```python
        self.argument_type, self.epilogue_type  = get_gemm_arguments_streamk(operation.epilogue_functor)
```
**EN:** Assigns `self.argument_type, self.epilogue_type` from `get_gemm_arguments_streamk(operation.epilogue_functor)`, establishing state in function `__init__`.
**CN:** 将 `self.argument_type, self.epilogue_type` 赋值为 `get_gemm_arguments_streamk(operation.epilogue_functor)`，用于在函数 `__init__` 中建立状态。

#### Lines 1097-1098 — Function `occupancy`

```python
    @property
    def occupancy(self):
```
**EN:** Defines function `occupancy` with parameters `self`.
**CN:** 定义函数 `occupancy`，参数为 `self`。

##### Lines 1099-1107 — Conditional `self._occupancy is None`

```python
        if self._occupancy is None:
            err, self._occupancy = cuda.cuOccupancyMaxActiveBlocksPerMultiprocessorWithFlags(
                self.kernel, self.threads, self.shared_memory_capacity,
                cuda.CUoccupancy_flags.CU_OCCUPANCY_DISABLE_CACHING_OVERRIDE)

            if err != cuda.CUresult.CUDA_SUCCESS:
                raise RuntimeError(
                    "CUDA error on call to cuOccupancyMaxActiveBlocksPerMultiprocessorWithFlags: "
                    f"{cuda.cuGetErrorString(err)[1]}")
```
**EN:** Checks `self._occupancy is None` and selects the matching branch in function `occupancy`.
**CN:** 检查 `self._occupancy is None`，并在函数 `occupancy` 中选择匹配的分支。

##### Line 1108 — Return

```python
        return self._occupancy
```
**EN:** Returns `self._occupancy` to the caller.
**CN:** 向调用方返回 `self._occupancy`。

#### Line 1110 — Function `get_device_workspace_size`

```python
    def get_device_workspace_size(self, arguments: GemmArguments2xStreamK, device_sms: int, sm_occupancy: int):
```
**EN:** Defines function `get_device_workspace_size` with parameters `self, arguments, device_sms, sm_occupancy`.
**CN:** 定义函数 `get_device_workspace_size`，参数为 `self, arguments, device_sms, sm_occupancy`。

##### Line 1111 — Return

```python
        return self.get_kernel_workspace_size(ctypes.byref(arguments.get_arguments()), device_sms, sm_occupancy)
```
**EN:** Returns `self.get_kernel_workspace_size(ctypes.byref(arguments.get_arguments()), device_sms, sm_occupancy)` to the caller.
**CN:** 向调用方返回 `self.get_kernel_workspace_size(ctypes.byref(arguments.get_arguments()), device_sms, sm_occupancy)`。

### Lines 1114-1116 — Comment or spacing block

```python
################################################################################
# Runtime module for GEMM Universal within CUTLASS 3
################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Lines 1119-1122 — Class `GemmRTUniversal3x`

```python
class GemmRTUniversal3x(GemmRTUniversal):
    """
    Manages the CUTLASS runtime components for 3.x kernels
    """
```
**EN:** Declares class `GemmRTUniversal3x` deriving from `GemmRTUniversal`. Purpose: Manages the CUTLASS runtime components for 3.x kernels
**CN:** 声明类 `GemmRTUniversal3x`，其基类为 `GemmRTUniversal`。 其用途：Manages the CUTLASS runtime components for 3.x kernels

#### Lines 1124-1145 — Assign `KernelTemplate`

```python
    KernelTemplate = r"""

using Operator = ${operation_name}${operation_suffix};
extern "C"
#if defined(CUTLASS_ENABLE_SYCL)
SYCL_EXTERNAL SYCL_EXT_ONEAPI_FUNCTION_PROPERTY(
    (sycl::ext::oneapi::experimental::nd_range_kernel<
     3>)) [[sycl::reqd_sub_group_size(16)]]
void ${operation_name}(typename Operator::Params const params, 
                       sycl::ext::oneapi::experimental::work_group_memory<char[]> mem) {
  auto* smem = &mem[0];
#else
__global__ __launch_bounds__(Operator::MaxThreadsPerBlock, Operator::MinBlocksPerMultiprocessor)
void ${operation_name}(__grid_constant__ typename Operator::Params const params) {
  // Dynamic shared memory base pointer
  extern __shared__ char smem[];
#endif
  // Declare pointer to dynamic shared memory.
  Operator op;
  op(params, smem);
}
  """
```
**EN:** Assigns `KernelTemplate` from `'\n\nusing Operator = ${operation_name}${operation_suffix};\nextern "C"\n#if defined(CUTLASS_ENABLE_SYCL)\nSYCL_EXTERNAL SYCL_EXT_ONEAPI_FUNCTION_PROPERTY(\n...`, establishing state in class `GemmRTUniversal3x`.
**CN:** 将 `KernelTemplate` 赋值为 `'\n\nusing Operator = ${operation_name}${operation_suffix};\nextern "C"\n#if defined(CUTLASS_ENABLE_SYCL)\nSYCL_EXTERNAL SYCL_EXT_ONEAPI_FUNCTION_PROPERTY(\n...`，用于在类 `GemmRTUniversal3x` 中建立状态。

#### Lines 1146-1200 — Assign `HostTemplate`

```python
    HostTemplate = r"""
extern "C" {
  // Get the size of params in bytes
  int ${operation_name}_get_param_size(){
    return sizeof(${operation_name}${operation_suffix}::Params);
  }

  // Get the size of dynamic shared memory in bytes
  int ${operation_name}_shared_memory_size() {
    return ${operation_name}${operation_suffix}::SharedStorageSize;
  }

  using GemmType = ${operation_name}_base;

  bool ${operation_name}_uses_default_epilogue() {
    return std::is_same_v<GemmType::CollectiveEpilogue::DispatchPolicy, cutlass::gemm::EpilogueDefault>;
  }

  // Get the workspace size
  uint64_t ${operation_name}_get_kernel_workspace_size(GemmType::Arguments* argument) {
    return GemmType::get_workspace_size(*argument);
  }

  // Get the params as byte array
  char* ${operation_name}_get_params(GemmType::Arguments* argument, int* workspace){
    GemmType::Params params = GemmType::to_underlying_arguments(*argument, workspace);
    char *bytes = ((char*)(&params));
    char *output = new char[sizeof(GemmType::Params)];
    for (unsigned int i = 0; i < sizeof(GemmType::Params); i ++)
        output[i] = bytes[i];

    return output;
  }

  // Get the total number of blocks for a persistent kernel
  uint64_t ${operation_name}_get_persistent_tiled_blk_shape_mnl(GemmType::ProblemShape problem) {
    auto problem_shape_MNKL = append<4>(problem, Int<1>{});
    auto [problem_blocks_m, problem_blocks_n, problem_blocks_l] =
        cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90::get_tiled_cta_shape_mnl(
            problem_shape_MNKL, GemmType::TileShape{}, GemmType::DispatchPolicy::ClusterShape{});
    return problem_blocks_m * problem_blocks_n * problem_blocks_l;
  }

  // Get the grid shape
  dim3 ${operation_name}_get_grid_shape(GemmType::Arguments* args, int* workspace) {
    auto tmp_params = GemmType::to_underlying_arguments(*args, workspace);
    return GemmType::get_grid_shape(tmp_params);
  }

  // Get the block shape
  dim3 ${operation_name}_get_block_shape() {
    return GemmType::get_block_shape();
  }
}
  """
```
**EN:** Assigns `HostTemplate` from `'\nextern "C" {\n // Get the size of params in bytes\n int ${operation_name}_get_param_size(){\n return sizeof(${operation_name}${operation_suffix}::Params);...`, establishing state in class `GemmRTUniversal3x`.
**CN:** 将 `HostTemplate` 赋值为 `'\nextern "C" {\n // Get the size of params in bytes\n int ${operation_name}_get_param_size(){\n return sizeof(${operation_name}${operation_suffix}::Params);...`，用于在类 `GemmRTUniversal3x` 中建立状态。

#### Line 1202 — Function `__init__`

```python
    def __init__(self, operation):
```
**EN:** Defines function `__init__` with parameters `self, operation`.
**CN:** 定义函数 `__init__`，参数为 `self, operation`。

##### Line 1203 — Call `super(GemmRTUniversal3x, self).__init__`

```python
        super(GemmRTUniversal3x, self).__init__(operation)
```
**EN:** Calls `super(GemmRTUniversal3x, self).__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super(GemmRTUniversal3x, self).__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Lines 1204-1210 — Assign `self.extra_funcs`

```python
        self.extra_funcs = {
            "get_grid_shape": dim3_,
            "get_block_shape": dim3_,
            "get_persistent_tiled_blk_shape_mnl": ctypes.c_uint64,
            "get_kernel_workspace_size": ctypes.c_uint64,
            "uses_default_epilogue": ctypes.c_bool,
        }
```
**EN:** Assigns `self.extra_funcs` from `{'get_grid_shape': dim3_, 'get_block_shape': dim3_, 'get_persistent_tiled_blk_shape_mnl': ctypes.c_uint64, 'get_kernel_workspace_size': ctypes.c_uint64, 'use...`, establishing state in function `__init__`.
**CN:** 将 `self.extra_funcs` 赋值为 `{'get_grid_shape': dim3_, 'get_block_shape': dim3_, 'get_persistent_tiled_blk_shape_mnl': ctypes.c_uint64, 'get_kernel_workspace_size': ctypes.c_uint64, 'use...`，用于在函数 `__init__` 中建立状态。

##### Line 1211 — Assign `self.emitter`

```python
        self.emitter = EmitGemmUniversalInstance3x("_type")
```
**EN:** Assigns `self.emitter` from `EmitGemmUniversalInstance3x('_type')`, establishing state in function `__init__`.
**CN:** 将 `self.emitter` 赋值为 `EmitGemmUniversalInstance3x('_type')`，用于在函数 `__init__` 中建立状态。

#### Line 1213 — Function `get_device_workspace_size`

```python
    def get_device_workspace_size(self, arguments: GemmArguments3x):
```
**EN:** Defines function `get_device_workspace_size` with parameters `self, arguments`.
**CN:** 定义函数 `get_device_workspace_size`，参数为 `self, arguments`。

##### Line 1214 — Return

```python
        return self.get_kernel_workspace_size(ctypes.byref(arguments.get_arguments()))
```
**EN:** Returns `self.get_kernel_workspace_size(ctypes.byref(arguments.get_arguments()))` to the caller.
**CN:** 向调用方返回 `self.get_kernel_workspace_size(ctypes.byref(arguments.get_arguments()))`。

### Lines 1217-1218 — Class `EmitGemmUniversalInstance3x`

```python
class EmitGemmUniversalInstance3x:
    """Responsible for emitting a CUTLASS 3 template definition"""
```
**EN:** Declares class `EmitGemmUniversalInstance3x` deriving from `object`. Purpose: Responsible for emitting a CUTLASS 3 template definition
**CN:** 声明类 `EmitGemmUniversalInstance3x`，其基类为 `object`。 其用途：Responsible for emitting a CUTLASS 3 template definition

#### Line 1220 — Function `__init__`

```python
    def __init__(self, operation_suffix=""):
```
**EN:** Defines function `__init__` with parameters `self, operation_suffix`.
**CN:** 定义函数 `__init__`，参数为 `self, operation_suffix`。

##### Line 1221 — Assign `self.operation_suffix`

```python
        self.operation_suffix = operation_suffix
```
**EN:** Assigns `self.operation_suffix` from `operation_suffix`, establishing state in function `__init__`.
**CN:** 将 `self.operation_suffix` 赋值为 `operation_suffix`，用于在函数 `__init__` 中建立状态。

##### Lines 1222-1233 — Assign `self.includes`

```python
        self.includes = [
            "cutlass/cutlass.h",
            "cute/tensor.hpp",
            "cute/atom/mma_atom.hpp",
            "cutlass/numeric_types.h",
            "cutlass/gemm/collective/collective_builder.hpp",
            "cutlass/gemm/kernel/sm90_tile_scheduler.hpp",
            "cutlass/gemm/kernel/gemm_universal.hpp",
            "cutlass/epilogue/collective/collective_builder.hpp",
            "cutlass/epilogue/collective/default_epilogue.hpp",
            "cutlass/epilogue/thread/linear_combination.h"
        ]
```
**EN:** Assigns `self.includes` from `['cutlass/cutlass.h', 'cute/tensor.hpp', 'cute/atom/mma_atom.hpp', 'cutlass/numeric_types.h', 'cutlass/gemm/collective/collective_builder.hpp', 'cutlass/gemm...`, establishing state in function `__init__`.
**CN:** 将 `self.includes` 赋值为 `['cutlass/cutlass.h', 'cute/tensor.hpp', 'cute/atom/mma_atom.hpp', 'cutlass/numeric_types.h', 'cutlass/gemm/collective/collective_builder.hpp', 'cutlass/gemm...`，用于在函数 `__init__` 中建立状态。

##### Lines 1234-1272 — Assign `self.gemm_template_kernel`

```python
        self.gemm_template_kernel = """
using namespace cute;

using CollectiveEpilogue =
  typename cutlass::epilogue::collective::CollectiveBuilder<
    ${arch}, ${opcode_class},
    cute::Shape<cute::_${threadblock_shape_m}, cute::_${threadblock_shape_n}, cute::_${threadblock_shape_k}>,
    cute::Shape<cute::_${cluster_m},cute::_${cluster_n},cute::_${cluster_k}>,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ${element_accumulator}, ${element_epilogue},
    ${element_c}, ${layout_c}, ${align_c},
    ${element_d}, ${layout_d}, ${align_d},
    ${epilogue_schedule}
  >::CollectiveOp;

using CollectiveMainloop =
  typename cutlass::gemm::collective::CollectiveBuilder<
    ${arch}, ${opcode_class},
    ${element_a}, ${layout_a}, ${align_a},
    ${element_b}, ${layout_b}, ${align_b},
    ${element_accumulator},
    cute::Shape<cute::_${threadblock_shape_m}, cute::_${threadblock_shape_n}, cute::_${threadblock_shape_k}>,
    cute::Shape<cute::_${cluster_m},cute::_${cluster_n},cute::_${cluster_k}>,
    ${stage_count_type},
    ${kernel_schedule}
  >::CollectiveOp;

// Gemm operator ${operation_name}
using ${operation_name}_base = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>,
    CollectiveMainloop,
    CollectiveEpilogue,
    ${tile_scheduler}
>;

// Define named type
struct ${operation_name}${operation_suffix} :
  public ${operation_name}_base { };
"""
```
**EN:** Assigns `self.gemm_template_kernel` from `'\nusing namespace cute;\n\nusing CollectiveEpilogue =\n typename cutlass::epilogue::collective::CollectiveBuilder<\n ${arch}, ${opcode_class},\n cute::Shape...`, establishing state in function `__init__`.
**CN:** 将 `self.gemm_template_kernel` 赋值为 `'\nusing namespace cute;\n\nusing CollectiveEpilogue =\n typename cutlass::epilogue::collective::CollectiveBuilder<\n ${arch}, ${opcode_class},\n cute::Shape...`，用于在函数 `__init__` 中建立状态。

##### Lines 1273-1314 — Assign `self.gemm_template_kernel_visitor`

```python
        self.gemm_template_kernel_visitor = """
using namespace cute;

${callback_decl}

using CollectiveEpilogue =
  typename cutlass::epilogue::collective::CollectiveBuilder<
    ${arch}, ${opcode_class},
    cute::Shape<cute::_${threadblock_shape_m}, cute::_${threadblock_shape_n}, cute::_${threadblock_shape_k}>,
    cute::Shape<cute::_${cluster_m},cute::_${cluster_n},cute::_${cluster_k}>,
    cutlass::epilogue::collective::EpilogueTileAuto,
    ${element_accumulator}, ${element_epilogue},
    ElementC, StrideC, ${align_c},
    ElementD, StrideD, ${align_d},
    ${epilogue_schedule},
    ${callback_name}
  >::CollectiveOp;

using CollectiveMainloop =
  typename cutlass::gemm::collective::CollectiveBuilder<
    ${arch}, ${opcode_class},
    ${element_a}, ${layout_a}, ${align_a},
    ${element_b}, ${layout_b}, ${align_b},
    ${element_accumulator},
    cute::Shape<cute::_${threadblock_shape_m}, cute::_${threadblock_shape_n}, cute::_${threadblock_shape_k}>,
    cute::Shape<cute::_${cluster_m},cute::_${cluster_n},cute::_${cluster_k}>,
    ${stage_count_type},
    ${kernel_schedule}
  >::CollectiveOp;

// Gemm operator ${operation_name}
using ${operation_name}_base = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>,
    CollectiveMainloop,
    CollectiveEpilogue,
    ${tile_scheduler}
>;

// Define named type
struct ${operation_name}${operation_suffix} :
  public ${operation_name}_base { };
"""
```
**EN:** Assigns `self.gemm_template_kernel_visitor` from `'\nusing namespace cute;\n\n${callback_decl}\n\nusing CollectiveEpilogue =\n typename cutlass::epilogue::collective::CollectiveBuilder<\n ${arch}, ${opcode_c...`, establishing state in function `__init__`.
**CN:** 将 `self.gemm_template_kernel_visitor` 赋值为 `'\nusing namespace cute;\n\n${callback_decl}\n\nusing CollectiveEpilogue =\n typename cutlass::epilogue::collective::CollectiveBuilder<\n ${arch}, ${opcode_c...`，用于在函数 `__init__` 中建立状态。

##### Lines 1316-1320 — Assign `self.gemm_template_device`

```python
        self.gemm_template_device = self.gemm_template_kernel + """

// Define device-level operator
using DeviceKernel = cutlass::gemm::device::GemmUniversalAdapter<${operation_name}${operation_suffix}>;
"""
```
**EN:** Assigns `self.gemm_template_device` from `self.gemm_template_kernel + '\n\n// Define device-level operator\nusing DeviceKernel = cutlass::gemm::device::GemmUniversalAdapter<${operation_name}${operati...`, establishing state in function `__init__`.
**CN:** 将 `self.gemm_template_device` 赋值为 `self.gemm_template_kernel + '\n\n// Define device-level operator\nusing DeviceKernel = cutlass::gemm::device::GemmUniversalAdapter<${operation_name}${operati...`，用于在函数 `__init__` 中建立状态。

#### Lines 1322-1323 — Function `emit`

```python
    def emit(self, operation):
        # Support built-in epilogue functors or user-defined functions
```
**EN:** Defines function `emit` with parameters `self, operation`.
**CN:** 定义函数 `emit`，参数为 `self, operation`。

##### Lines 1324-1329 — Conditional `is_intel_xe_arch(operation.arch)`

```python
        if is_intel_xe_arch(operation.arch):
            stage_count_type = "cutlass::gemm::collective::StageCountAuto"
        elif operation.tile_description.stages is None or operation.tile_description.stages == 0:
            stage_count_type = "cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(sizeof(typename CollectiveEpilogue::SharedStorage))>"
        else:
            stage_count_type = "_" + str(operation.tile_description.stages)
```
**EN:** Checks `is_intel_xe_arch(operation.arch)` and selects the matching branch in function `emit`.
**CN:** 检查 `is_intel_xe_arch(operation.arch)`，并在函数 `emit` 中选择匹配的分支。

##### Lines 1331-1334 — Conditional `operation.emission_type == EmissionType.Kernel`

```python
        if operation.emission_type == EmissionType.Kernel:
            gemm_template = self.gemm_template_kernel
        else:
            gemm_template = self.gemm_template_device
```
**EN:** Checks `operation.emission_type == EmissionType.Kernel` and selects the matching branch in function `emit`.
**CN:** 检查 `operation.emission_type == EmissionType.Kernel`，并在函数 `emit` 中选择匹配的分支。

##### Line 1336 — Assign `kschedule`

```python
        kschedule = KernelScheduleType.ScheduleAuto
```
**EN:** Assigns `kschedule` from `KernelScheduleType.ScheduleAuto`, establishing state in function `emit`.
**CN:** 将 `kschedule` 赋值为 `KernelScheduleType.ScheduleAuto`，用于在函数 `emit` 中建立状态。

##### Line 1337 — Assign `eschedule`

```python
        eschedule = EpilogueScheduleType.ScheduleAuto
```
**EN:** Assigns `eschedule` from `EpilogueScheduleType.ScheduleAuto`, establishing state in function `emit`.
**CN:** 将 `eschedule` 赋值为 `EpilogueScheduleType.ScheduleAuto`，用于在函数 `emit` 中建立状态。

##### Line 1338 — Assign `tschedule`

```python
        tschedule = TileSchedulerType.Default
```
**EN:** Assigns `tschedule` from `TileSchedulerType.Default`, establishing state in function `emit`.
**CN:** 将 `tschedule` 赋值为 `TileSchedulerType.Default`，用于在函数 `emit` 中建立状态。

##### Lines 1339-1340 — Conditional `operation.tile_description.kernel_schedule is not None`

```python
        if operation.tile_description.kernel_schedule is not None:
            kschedule = operation.tile_description.kernel_schedule
```
**EN:** Checks `operation.tile_description.kernel_schedule is not None` and selects the matching branch in function `emit`.
**CN:** 检查 `operation.tile_description.kernel_schedule is not None`，并在函数 `emit` 中选择匹配的分支。

##### Lines 1341-1342 — Conditional `operation.tile_description.epilogue_schedule is not None`

```python
        if operation.tile_description.epilogue_schedule is not None:
            eschedule = operation.tile_description.epilogue_schedule
```
**EN:** Checks `operation.tile_description.epilogue_schedule is not None` and selects the matching branch in function `emit`.
**CN:** 检查 `operation.tile_description.epilogue_schedule is not None`，并在函数 `emit` 中选择匹配的分支。

##### Lines 1343-1344 — Conditional `operation.tile_description.tile_scheduler is not None`

```python
        if operation.tile_description.tile_scheduler is not None:
            tschedule = operation.tile_description.tile_scheduler
```
**EN:** Checks `operation.tile_description.tile_scheduler is not None` and selects the matching branch in function `emit`.
**CN:** 检查 `operation.tile_description.tile_scheduler is not None`，并在函数 `emit` 中选择匹配的分支。

##### Line 1346 — Assign `arch`

```python
        arch = f"cutlass::arch::Xe{operation.arch}" if is_intel_xe_arch(operation.arch) else f"cutlass::arch::Sm{operation.arch}"
```
**EN:** Assigns `arch` from `f'cutlass::arch::Xe{operation.arch}' if is_intel_xe_arch(operation.arch) else f'cutlass::arch::Sm{operation.arch}'`, establishing state in function `emit`.
**CN:** 将 `arch` 赋值为 `f'cutlass::arch::Xe{operation.arch}' if is_intel_xe_arch(operation.arch) else f'cutlass::arch::Sm{operation.arch}'`，用于在函数 `emit` 中建立状态。

##### Lines 1347-1376 — Assign `values`

```python
        values = {
            "operation_name": operation.procedural_name(),
            "operation_suffix": self.operation_suffix,
            "element_a": DataTypeTag[operation.A.element],
            "layout_a": LayoutTag[operation.A.layout],
            "element_b": DataTypeTag[operation.B.element],
            "layout_b": LayoutTag[operation.B.layout],
            "element_c": DataTypeTag[operation.C.element],
            "layout_c": LayoutTag[operation.C.layout],
            "element_d": DataTypeTag[operation.epilogue_functor.element_output],
            "layout_d": LayoutTag[operation.C.layout],
            "element_accumulator": DataTypeTag[operation.accumulator_type()],
            "element_epilogue": DataTypeTag[operation.epilogue_functor.element_epilogue],
            "opcode_class": OpcodeClassTag[operation.tile_description.math_instruction.opcode_class],
            "arch": arch,
            "threadblock_shape_m": str(operation.tile_description.threadblock_shape[0]),
            "threadblock_shape_n": str(operation.tile_description.threadblock_shape[1]),
            "threadblock_shape_k": str(operation.tile_description.threadblock_shape[2]),
            "cluster_m": str(operation.tile_description.cluster_shape[0]),
            "cluster_n": str(operation.tile_description.cluster_shape[1]),
            "cluster_k": str(operation.tile_description.cluster_shape[2]),
            "align_a": str(operation.A.alignment),
            "align_b": str(operation.B.alignment),
            "align_c": str(operation.C.alignment),
            "align_d": str(operation.C.alignment),
            "stage_count_type": stage_count_type,
            "kernel_schedule": KernelScheduleTag[kschedule],
            "epilogue_schedule": EpilogueScheduleTag[eschedule],
            "tile_scheduler": TileSchedulerTag[tschedule]
        }
```
**EN:** Assigns `values` from `{'operation_name': operation.procedural_name(), 'operation_suffix': self.operation_suffix, 'element_a': DataTypeTag[operation.A.element], 'layout_a': LayoutT...`, establishing state in function `emit`.
**CN:** 将 `values` 赋值为 `{'operation_name': operation.procedural_name(), 'operation_suffix': self.operation_suffix, 'element_a': DataTypeTag[operation.A.element], 'layout_a': LayoutT...`，用于在函数 `emit` 中建立状态。

##### Lines 1377-1385 — Conditional `hasattr(operation.epilogue_functor, 'visitor')`

```python
        if hasattr(operation.epilogue_functor, "visitor"):
            callback_name, callback_decl = operation.epilogue_functor.emit(operation)
            values["callback_name"] = callback_name
            values["callback_decl"] = callback_decl
            return SubstituteTemplate(self.gemm_template_kernel_visitor, values)

        else:
            values["epilogue_functor"] = operation.epilogue_functor.emit()
            return SubstituteTemplate(gemm_template, values)
```
**EN:** Checks `hasattr(operation.epilogue_functor, 'visitor')` and selects the matching branch in function `emit`.
**CN:** 检查 `hasattr(operation.epilogue_functor, 'visitor')`，并在函数 `emit` 中选择匹配的分支。

### Lines 1388-1390 — Comment or spacing block

```python
###################################################################################################
# Runtime module for GEMM Grouped
###################################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Lines 1393-1396 — Class `GemmRTGrouped`

```python
class GemmRTGrouped(GemmRTbase):
    """
    GemmRTGrouped manages the CUTLASS runtime components
    """
```
**EN:** Declares class `GemmRTGrouped` deriving from `GemmRTbase`. Purpose: GemmRTGrouped manages the CUTLASS runtime components
**CN:** 声明类 `GemmRTGrouped`，其基类为 `GemmRTbase`。 其用途：GemmRTGrouped manages the CUTLASS runtime components

#### Lines 1398-1414 — Assign `KernelTemplate`

```python
    KernelTemplate = r"""
extern "C"
__global__ void
${operation_name}(${operation_name}${operation_suffix}::Params params) {

  // Dynamic shared memory base pointer
  extern __shared__ int SharedStorageBase[];

  // Declare pointer to dynamic shared memory.
  ${operation_name}${operation_suffix}::SharedStorage *shared_storage =
      reinterpret_cast<${operation_name}${operation_suffix}::SharedStorage *>(SharedStorageBase);

  ${operation_name}${operation_suffix} op;

  op(params, *shared_storage);
}
  """
```
**EN:** Assigns `KernelTemplate` from `'\nextern "C"\n__global__ void\n${operation_name}(${operation_name}${operation_suffix}::Params params) {\n\n // Dynamic shared memory base pointer\n extern _...`, establishing state in class `GemmRTGrouped`.
**CN:** 将 `KernelTemplate` 赋值为 `'\nextern "C"\n__global__ void\n${operation_name}(${operation_name}${operation_suffix}::Params params) {\n\n // Dynamic shared memory base pointer\n extern _...`，用于在类 `GemmRTGrouped` 中建立状态。

#### Lines 1416-1464 — Assign `HostTemplate`

```python
    HostTemplate = r"""
  extern "C" {

    // precompute scheduling information
     char * ${operation_name}_precompute(${operation_name}_base::Arguments const &args, int tile_count, size_t workspace_bytes) {
      char* host_workspace = new char[workspace_bytes];
      ${operation_name}_base::ProblemVisitor::host_precompute(
        args.host_problem_sizes,
        args.problem_count,
        args.threadblock_count,
        (void*)host_workspace
      );
      return host_workspace;
    }

    // Get the size of params in bytes
    int ${operation_name}_get_param_size(){
      return sizeof(${operation_name}${operation_suffix}::Params);
    }

    // Get the size of dynamic shared memory in bytes
    int ${operation_name}_shared_memory_size() {
      return int(sizeof(${operation_name}${operation_suffix}::SharedStorage));
    }

    // Get the params as byte array
    char* ${operation_name}_get_params(${operation_name}_base::Arguments* argument, int tile_count, void* workspace=nullptr){
      ${operation_name}_base::Params* params;
      params = new ${operation_name}_base::Params(*argument, workspace, tile_count);

      char *bytes = ((char*)(params));
      char *output = new char[sizeof(${operation_name}_base::Params)];
      for (unsigned int i = 0; i < sizeof(${operation_name}_base::Params); i ++)
          output[i] = bytes[i];

      return output;
    }

    cutlass::gemm::GemmCoord ${operation_name}_get_tiled_shape(
        cutlass::gemm::GemmCoord problem_size, cutlass::gemm::GemmCoord tile_size, int split_k_slices) {
        return ${operation_name}_base::ThreadblockSwizzle::get_tiled_shape(
            problem_size, tile_size, split_k_slices);
    }

    dim3 ${operation_name}_get_grid_shape(cutlass::gemm::GemmCoord tiled_shape) {
        return ${operation_name}_base::ThreadblockSwizzle::get_grid_shape(tiled_shape);
    }
  }
  """
```
**EN:** Assigns `HostTemplate` from `'\n extern "C" {\n\n // precompute scheduling information\n char * ${operation_name}_precompute(${operation_name}_base::Arguments const &args, int tile_count...`, establishing state in class `GemmRTGrouped`.
**CN:** 将 `HostTemplate` 赋值为 `'\n extern "C" {\n\n // precompute scheduling information\n char * ${operation_name}_precompute(${operation_name}_base::Arguments const &args, int tile_count...`，用于在类 `GemmRTGrouped` 中建立状态。

#### Line 1466 — Function `__init__`

```python
    def __init__(self, operation: "GemmOperation"):
```
**EN:** Defines function `__init__` with parameters `self, operation`.
**CN:** 定义函数 `__init__`，参数为 `self, operation`。

##### Line 1467 — Call `super(GemmRTGrouped, self).__init__`

```python
        super(GemmRTGrouped, self).__init__(operation)
```
**EN:** Calls `super(GemmRTGrouped, self).__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super(GemmRTGrouped, self).__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Lines 1468-1472 — Assign `self.extra_funcs`

```python
        self.extra_funcs = {
            "precompute": None,
            "get_tiled_shape": GemmCoord_,
            "get_grid_shape": dim3_,
        }
```
**EN:** Assigns `self.extra_funcs` from `{'precompute': None, 'get_tiled_shape': GemmCoord_, 'get_grid_shape': dim3_}`, establishing state in function `__init__`.
**CN:** 将 `self.extra_funcs` 赋值为 `{'precompute': None, 'get_tiled_shape': GemmCoord_, 'get_grid_shape': dim3_}`，用于在函数 `__init__` 中建立状态。

##### Line 1473 — Assign `self.emitter`

```python
        self.emitter = EmitGemmGroupedInstance("_type")
```
**EN:** Assigns `self.emitter` from `EmitGemmGroupedInstance('_type')`, establishing state in function `__init__`.
**CN:** 将 `self.emitter` 赋值为 `EmitGemmGroupedInstance('_type')`，用于在函数 `__init__` 中建立状态。

##### Line 1474 — Assign `self.argument_type, self.epilogue_type`

```python
        self.argument_type, self.epilogue_type = get_gemm_grouped_arguments(operation.epilogue_functor)
```
**EN:** Assigns `self.argument_type, self.epilogue_type` from `get_gemm_grouped_arguments(operation.epilogue_functor)`, establishing state in function `__init__`.
**CN:** 将 `self.argument_type, self.epilogue_type` 赋值为 `get_gemm_grouped_arguments(operation.epilogue_functor)`，用于在函数 `__init__` 中建立状态。

##### Line 1475 — Assign `self.argtype`

```python
        self.argtype = [ctypes.POINTER(self.argument_type), ctypes.c_int, ctypes.c_void_p]
```
**EN:** Assigns `self.argtype` from `[ctypes.POINTER(self.argument_type), ctypes.c_int, ctypes.c_void_p]`, establishing state in function `__init__`.
**CN:** 将 `self.argtype` 赋值为 `[ctypes.POINTER(self.argument_type), ctypes.c_int, ctypes.c_void_p]`，用于在函数 `__init__` 中建立状态。

#### Line 1477 — Function `host_precompute`

```python
    def host_precompute(self, arguments, workspace_bytes):
```
**EN:** Defines function `host_precompute` with parameters `self, arguments, workspace_bytes`.
**CN:** 定义函数 `host_precompute`，参数为 `self, arguments, workspace_bytes`。

##### Lines 1478-1479 — Assign `self.precompute.argtype`

```python
        self.precompute.argtype = [
            self.argtype[0], ctypes.c_int, ctypes.c_longlong]
```
**EN:** Assigns `self.precompute.argtype` from `[self.argtype[0], ctypes.c_int, ctypes.c_longlong]`, establishing state in function `host_precompute`.
**CN:** 将 `self.precompute.argtype` 赋值为 `[self.argtype[0], ctypes.c_int, ctypes.c_longlong]`，用于在函数 `host_precompute` 中建立状态。

##### Line 1480 — Assign `self.precompute.restype`

```python
        self.precompute.restype = ctypes.POINTER(ctypes.c_byte * workspace_bytes)
```
**EN:** Assigns `self.precompute.restype` from `ctypes.POINTER(ctypes.c_byte * workspace_bytes)`, establishing state in function `host_precompute`.
**CN:** 将 `self.precompute.restype` 赋值为 `ctypes.POINTER(ctypes.c_byte * workspace_bytes)`，用于在函数 `host_precompute` 中建立状态。

##### Lines 1482-1485 — Assign `problem_info`

```python
        problem_info = self.precompute(
            ctypes.byref(arguments.arguments),
            arguments.total_tiles,
            workspace_bytes)
```
**EN:** Assigns `problem_info` from `self.precompute(ctypes.byref(arguments.arguments), arguments.total_tiles, workspace_bytes)`, establishing state in function `host_precompute`.
**CN:** 将 `problem_info` 赋值为 `self.precompute(ctypes.byref(arguments.arguments), arguments.total_tiles, workspace_bytes)`，用于在函数 `host_precompute` 中建立状态。

##### Line 1486 — Assign `problem_info_array`

```python
        problem_info_array = bytearray(problem_info.contents)
```
**EN:** Assigns `problem_info_array` from `bytearray(problem_info.contents)`, establishing state in function `host_precompute`.
**CN:** 将 `problem_info_array` 赋值为 `bytearray(problem_info.contents)`，用于在函数 `host_precompute` 中建立状态。

##### Line 1488 — Comment or spacing block

```python
        # copy to device memory
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 1489 — Return

```python
        return todevice(problem_info_array).ptr
```
**EN:** Returns `todevice(problem_info_array).ptr` to the caller.
**CN:** 向调用方返回 `todevice(problem_info_array).ptr`。

#### Line 1491 — Function `plan`

```python
    def plan(self, arguments):
```
**EN:** Defines function `plan` with parameters `self, arguments`.
**CN:** 定义函数 `plan`，参数为 `self, arguments`。

##### Lines 1492-1496 — Return

```python
        return LaunchConfiguration(
            [arguments.total_tiles, 1, 1],
            [self.threads, 1, 1],
            self.shared_memory_capacity,
        )
```
**EN:** Returns `LaunchConfiguration([arguments.total_tiles, 1, 1], [self.threads, 1, 1], self.shared_memory_capacity)` to the caller.
**CN:** 向调用方返回 `LaunchConfiguration([arguments.total_tiles, 1, 1], [self.threads, 1, 1], self.shared_memory_capacity)`。

#### Line 1498 — Function `get_workspace_size`

```python
    def get_workspace_size(self, arguments):
```
**EN:** Defines function `get_workspace_size` with parameters `self, arguments`.
**CN:** 定义函数 `get_workspace_size`，参数为 `self, arguments`。

##### Lines 1499-1504 — Conditional `self.operation.precompute_mode == SchedulerMode.Device`

```python
        if self.operation.precompute_mode == SchedulerMode.Device:
            return 0
        elif self.operation.precompute_mode == SchedulerMode.Host:
            total_tiles = arguments.total_tiles
            entries_per_block = 1
            return 8 * entries_per_block * total_tiles  # three int32_t
```
**EN:** Checks `self.operation.precompute_mode == SchedulerMode.Device` and selects the matching branch in function `get_workspace_size`.
**CN:** 检查 `self.operation.precompute_mode == SchedulerMode.Device`，并在函数 `get_workspace_size` 中选择匹配的分支。

### Lines 1507-1509 — Comment or spacing block

```python
################################################################################
# Runtime module for GEMM and grouped GEMM
################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Lines 1512-1515 — Class `GemmOperationBase`

```python
class GemmOperationBase:
    """
    CUTLASS GEMM operation
    """
```
**EN:** Declares class `GemmOperationBase` deriving from `object`. Purpose: CUTLASS GEMM operation
**CN:** 声明类 `GemmOperationBase`，其基类为 `object`。 其用途：CUTLASS GEMM operation

#### Lines 1517-1521 — Function `__init__`

```python
    def __init__(
        self, gemm_kind, arch, tile_description: TileDescription,
        A: TensorDescription, B: TensorDescription, C: TensorDescription,
        epilogue_functor, swizzling_functor=SwizzlingFunctor.Identity1,
        api=ApiVersion.v2x, emission_type=EmissionType.Kernel, **kwargs):
```
**EN:** Defines function `__init__` with parameters `self, gemm_kind, arch, tile_description, A, B, C, epilogue_functor, swizzling_functor, api, emission_type, **kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, gemm_kind, arch, tile_description, A, B, C, epilogue_functor, swizzling_functor, api, emission_type, **kwargs`。

##### Line 1522 — Annotated assign `self.operation_kind`

```python
        self.operation_kind: OperationKind = OperationKind.Gemm
```
**EN:** Declares `self.operation_kind` with an annotation and initializes it from `OperationKind.Gemm` in function `__init__`.
**CN:** 带类型标注地声明 `self.operation_kind`，并用 `OperationKind.Gemm` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 1523 — Annotated assign `self.arch`

```python
        self.arch: int = arch
```
**EN:** Declares `self.arch` with an annotation and initializes it from `arch` in function `__init__`.
**CN:** 带类型标注地声明 `self.arch`，并用 `arch` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 1524 — Annotated assign `self.tile_description`

```python
        self.tile_description: TileDescription = tile_description
```
**EN:** Declares `self.tile_description` with an annotation and initializes it from `tile_description` in function `__init__`.
**CN:** 带类型标注地声明 `self.tile_description`，并用 `tile_description` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 1525 — Annotated assign `self.gemm_kind`

```python
        self.gemm_kind: GemmKind = gemm_kind
```
**EN:** Declares `self.gemm_kind` with an annotation and initializes it from `gemm_kind` in function `__init__`.
**CN:** 带类型标注地声明 `self.gemm_kind`，并用 `gemm_kind` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 1527 — Assign `self.api`

```python
        self.api = api
```
**EN:** Assigns `self.api` from `api`, establishing state in function `__init__`.
**CN:** 将 `self.api` 赋值为 `api`，用于在函数 `__init__` 中建立状态。

##### Line 1528 — Assign `self.prefix`

```python
        self.prefix = "3x" if self.api == ApiVersion.v3x else ""
```
**EN:** Assigns `self.prefix` from `'3x' if self.api == ApiVersion.v3x else ''`, establishing state in function `__init__`.
**CN:** 将 `self.prefix` 赋值为 `'3x' if self.api == ApiVersion.v3x else ''`，用于在函数 `__init__` 中建立状态。

##### Line 1529 — Assign `self.emission_type`

```python
        self.emission_type = emission_type
```
**EN:** Assigns `self.emission_type` from `emission_type`, establishing state in function `__init__`.
**CN:** 将 `self.emission_type` 赋值为 `emission_type`，用于在函数 `__init__` 中建立状态。

##### Lines 1531-1533 — Comment or spacing block

```python
        # Optionally swap the TensorDescriptions for operands A and B and transpose their
        # layouts. This is needed to mimic the transpose performed by device::GemmUniversal.
        # The code below uses deep copy to avoid overwritting the original TensorDescription
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 1534-1536 — Assign `self.switched`

```python
        self.switched = (self.api != ApiVersion.v3x and
                         self.emission_type == EmissionType.Kernel and
                         C.layout == LayoutType.ColumnMajor)
```
**EN:** Assigns `self.switched` from `self.api != ApiVersion.v3x and self.emission_type == EmissionType.Kernel and (C.layout == LayoutType.ColumnMajor)`, establishing state in function `__init__`.
**CN:** 将 `self.switched` 赋值为 `self.api != ApiVersion.v3x and self.emission_type == EmissionType.Kernel and (C.layout == LayoutType.ColumnMajor)`，用于在函数 `__init__` 中建立状态。

##### Line 1538 — Assign `self.A, self.B, self.C`

```python
        self.A, self.B, self.C = GemmOperationBase.get_operands(A, B, C, self.switched)
```
**EN:** Assigns `self.A, self.B, self.C` from `GemmOperationBase.get_operands(A, B, C, self.switched)`, establishing state in function `__init__`.
**CN:** 将 `self.A, self.B, self.C` 赋值为 `GemmOperationBase.get_operands(A, B, C, self.switched)`，用于在函数 `__init__` 中建立状态。

##### Line 1540 — Assign `self.epilogue_functor`

```python
        self.epilogue_functor = epilogue_functor
```
**EN:** Assigns `self.epilogue_functor` from `epilogue_functor`, establishing state in function `__init__`.
**CN:** 将 `self.epilogue_functor` 赋值为 `epilogue_functor`，用于在函数 `__init__` 中建立状态。

##### Line 1541 — Assign `self.swizzling_functor`

```python
        self.swizzling_functor = swizzling_functor
```
**EN:** Assigns `self.swizzling_functor` from `swizzling_functor`, establishing state in function `__init__`.
**CN:** 将 `self.swizzling_functor` 赋值为 `swizzling_functor`，用于在函数 `__init__` 中建立状态。

##### Lines 1543-1546 — Conditional `'direct_store' in kwargs`

```python
        if "direct_store" in kwargs:
            self.direct_store = kwargs["direct_store"]
        else:
            self.direct_store = False
```
**EN:** Checks `'direct_store' in kwargs` and selects the matching branch in function `__init__`.
**CN:** 检查 `'direct_store' in kwargs`，并在函数 `__init__` 中选择匹配的分支。

#### Lines 1548-1563 — Function `get_operands`

```python
    @staticmethod
    def get_operands(A: TensorDescription, B: TensorDescription, C: TensorDescription, swap: bool):
        """
        Makes copies of A, B, and C, and possibly transposes their order. If ``swap`` is set,
        A and B are swapped, and the layout of A, B, and C are transposed.

        :param A: description of operand A
        :type A: TensorDescription
        :param B: description of operand B
        :type B: TensorDescription
        :param C: description of operand C
        :type C: TensorDescription

        :return: descriptions of operands A, B, and C
        :rtype: tuple[TileDescription]
        """
```
**EN:** Defines function `get_operands` with parameters `A, B, C, swap`. Purpose: Makes copies of A, B, and C, and possibly transposes their order.
**CN:** 定义函数 `get_operands`，参数为 `A, B, C, swap`。 其用途：Makes copies of A, B, and C, and possibly transposes their order.

##### Lines 1564-1574 — Conditional `swap`

```python
        if swap:
            A_out = copy.deepcopy(B)
            B_out = copy.deepcopy(A)
            C_out = copy.deepcopy(C)
            A_out.layout = transpose_layout(A_out.layout)
            B_out.layout = transpose_layout(B_out.layout)
            C_out.layout = transpose_layout(C_out.layout)
        else:
            A_out = copy.deepcopy(A)
            B_out = copy.deepcopy(B)
            C_out = copy.deepcopy(C)
```
**EN:** Checks `swap` and selects the matching branch in function `get_operands`.
**CN:** 检查 `swap`，并在函数 `get_operands` 中选择匹配的分支。

##### Line 1575 — Return

```python
        return A_out, B_out, C_out
```
**EN:** Returns `(A_out, B_out, C_out)` to the caller.
**CN:** 向调用方返回 `(A_out, B_out, C_out)`。

#### Lines 1577-1580 — Function `run`

```python
    def run(self, arguments: GemmArguments) :
        """
        Configure and launch the cuda kernel with input arguments
        """
```
**EN:** Defines function `run` with parameters `self, arguments`. Purpose: Configure and launch the cuda kernel with input arguments
**CN:** 定义函数 `run`，参数为 `self, arguments`。 其用途：Configure and launch the cuda kernel with input arguments

##### Lines 1581-1582 — Conditional `self.emission_type == EmissionType.Device`

```python
        if self.emission_type == EmissionType.Device:
            raise Exception('Running a kernel via PyCUTLASS is only enabled with emission type "Kernel"')
```
**EN:** Checks `self.emission_type == EmissionType.Device` and selects the matching branch in function `run`.
**CN:** 检查 `self.emission_type == EmissionType.Device`，并在函数 `run` 中选择匹配的分支。

##### Lines 1584-1589 — Assign `err`

```python
        err = self.rt_module.run(
            arguments.host_workspace,
            arguments.device_workspace,
            arguments.launch_config,
            arguments.stream
        )
```
**EN:** Assigns `err` from `self.rt_module.run(arguments.host_workspace, arguments.device_workspace, arguments.launch_config, arguments.stream)`, establishing state in function `run`.
**CN:** 将 `err` 赋值为 `self.rt_module.run(arguments.host_workspace, arguments.device_workspace, arguments.launch_config, arguments.stream)`，用于在函数 `run` 中建立状态。

##### Lines 1591-1592 — Conditional `err != 0`

```python
        if err != 0:
            raise RuntimeError("CUDA Error %s" % str(err))
```
**EN:** Checks `err != 0` and selects the matching branch in function `run`.
**CN:** 检查 `err != 0`，并在函数 `run` 中选择匹配的分支。

##### Line 1594 — Return

```python
        return err
```
**EN:** Returns `err` to the caller.
**CN:** 向调用方返回 `err`。

#### Line 1596 — Function `is_complex`

```python
    def is_complex(self):
```
**EN:** Defines function `is_complex` with parameters `self`.
**CN:** 定义函数 `is_complex`，参数为 `self`。

##### Lines 1597-1601 — Assign `complex_operators`

```python
        complex_operators = [
            MathOperation.multiply_add_complex,
            MathOperation.multiply_add_complex_gaussian,
            MathOperation.multiply_add_complex_fast_f32,
        ]
```
**EN:** Assigns `complex_operators` from `[MathOperation.multiply_add_complex, MathOperation.multiply_add_complex_gaussian, MathOperation.multiply_add_complex_fast_f32]`, establishing state in function `is_complex`.
**CN:** 将 `complex_operators` 赋值为 `[MathOperation.multiply_add_complex, MathOperation.multiply_add_complex_gaussian, MathOperation.multiply_add_complex_fast_f32]`，用于在函数 `is_complex` 中建立状态。

##### Line 1602 — Return

```python
        return self.tile_description.math_instruction.math_operation in complex_operators
```
**EN:** Returns `self.tile_description.math_instruction.math_operation in complex_operators` to the caller.
**CN:** 向调用方返回 `self.tile_description.math_instruction.math_operation in complex_operators`。

#### Line 1604 — Function `is_planar_complex`

```python
    def is_planar_complex(self):
```
**EN:** Defines function `is_planar_complex` with parameters `self`.
**CN:** 定义函数 `is_planar_complex`，参数为 `self`。

##### Line 1605 — Return

```python
        return self.gemm_kind in (GemmKind.PlanarComplex, GemmKind.PlanarComplexArray)
```
**EN:** Returns `self.gemm_kind in (GemmKind.PlanarComplex, GemmKind.PlanarComplexArray)` to the caller.
**CN:** 向调用方返回 `self.gemm_kind in (GemmKind.PlanarComplex, GemmKind.PlanarComplexArray)`。

#### Line 1607 — Function `accumulator_type`

```python
    def accumulator_type(self):
```
**EN:** Defines function `accumulator_type` with parameters `self`.
**CN:** 定义函数 `accumulator_type`，参数为 `self`。

##### Line 1608 — Assign `accum`

```python
        accum = self.tile_description.math_instruction.element_accumulator
```
**EN:** Assigns `accum` from `self.tile_description.math_instruction.element_accumulator`, establishing state in function `accumulator_type`.
**CN:** 将 `accum` 赋值为 `self.tile_description.math_instruction.element_accumulator`，用于在函数 `accumulator_type` 中建立状态。

##### Lines 1610-1611 — Conditional `self.is_complex()`

```python
        if self.is_complex():
            return get_complex_from_real(accum)
```
**EN:** Checks `self.is_complex()` and selects the matching branch in function `accumulator_type`.
**CN:** 检查 `self.is_complex()`，并在函数 `accumulator_type` 中选择匹配的分支。

##### Line 1613 — Return

```python
        return accum
```
**EN:** Returns `accum` to the caller.
**CN:** 向调用方返回 `accum`。

#### Line 1615 — Function `short_math_name`

```python
    def short_math_name(self):
```
**EN:** Defines function `short_math_name` with parameters `self`.
**CN:** 定义函数 `short_math_name`，参数为 `self`。

##### Lines 1616-1617 — Conditional `self.tile_description.math_instruction.math_operation == MathOperation.multiply_add_complex_gaussian`

```python
        if self.tile_description.math_instruction.math_operation == MathOperation.multiply_add_complex_gaussian:
            return "g%s" % ShortDataTypeNames[self.accumulator_type()]
```
**EN:** Checks `self.tile_description.math_instruction.math_operation == MathOperation.multiply_add_complex_gaussian` and selects the matching branch in function `short_math_name`.
**CN:** 检查 `self.tile_description.math_instruction.math_operation == MathOperation.multiply_add_complex_gaussian`，并在函数 `short_math_name` 中选择匹配的分支。

##### Line 1618 — Return

```python
        return ShortDataTypeNames[self.accumulator_type()]
```
**EN:** Returns `ShortDataTypeNames[self.accumulator_type()]` to the caller.
**CN:** 向调用方返回 `ShortDataTypeNames[self.accumulator_type()]`。

#### Lines 1620-1621 — Function `core_name`

```python
    def core_name(self):
        """The basic operation kind is prefixed with a letter indicating the accumulation type."""
```
**EN:** Defines function `core_name` with parameters `self`. Purpose: The basic operation kind is prefixed with a letter indicating the accumulation type.
**CN:** 定义函数 `core_name`，参数为 `self`。 其用途：The basic operation kind is prefixed with a letter indicating the accumulation type.

##### Line 1623 — Assign `inst_shape`

```python
        inst_shape = ""
```
**EN:** Assigns `inst_shape` from `''`, establishing state in function `core_name`.
**CN:** 将 `inst_shape` 赋值为 `''`，用于在函数 `core_name` 中建立状态。

##### Line 1624 — Assign `inst_operation`

```python
        inst_operation = ""
```
**EN:** Assigns `inst_operation` from `''`, establishing state in function `core_name`.
**CN:** 将 `inst_operation` 赋值为 `''`，用于在函数 `core_name` 中建立状态。

##### Line 1625 — Assign `intermediate_type`

```python
        intermediate_type = ""
```
**EN:** Assigns `intermediate_type` from `''`, establishing state in function `core_name`.
**CN:** 将 `intermediate_type` 赋值为 `''`，用于在函数 `core_name` 中建立状态。

##### Lines 1627-1629 — Assign `math_operations_map`

```python
        math_operations_map = {
            MathOperation.xor_popc: "xor",
        }
```
**EN:** Assigns `math_operations_map` from `{MathOperation.xor_popc: 'xor'}`, establishing state in function `core_name`.
**CN:** 将 `math_operations_map` 赋值为 `{MathOperation.xor_popc: 'xor'}`，用于在函数 `core_name` 中建立状态。

##### Lines 1631-1649 — Conditional `self.tile_description.math_instruction.opcode_class == OpcodeClass.TensorOp or self.tile_description.math_instruction.opcode_class == OpcodeClass.WmmaTensorOp`

```python
        if (self.tile_description.math_instruction.opcode_class == OpcodeClass.TensorOp or
            self.tile_description.math_instruction.opcode_class == OpcodeClass.WmmaTensorOp):
            math_op = self.tile_description.math_instruction.math_operation
            math_op_string = math_operations_map[math_op] if math_op in math_operations_map.keys() else ""

            if self.tile_description.math_instruction.instruction_shape is not None:
                if self.api == ApiVersion.v3x and self.arch >= 90:
                    inst_shape = "%dx%dx%d" % tuple(
                        self.tile_description.math_instruction.instruction_shape)
                else:
                    inst_shape = "%d%d%d" % tuple(
                        self.tile_description.math_instruction.instruction_shape)
            else:
                inst_shape = "Default"
            inst_shape += math_op_string

            if (self.tile_description.math_instruction.element_a != self.A.element and
                self.tile_description.math_instruction.element_a != self.tile_description.math_instruction.element_accumulator):
                intermediate_type = DataTypeNames[self.tile_description.math_instruction.element_a]
```
**EN:** Checks `self.tile_description.math_instruction.opcode_class == OpcodeClass.TensorOp or self.tile_description.math_instruction.opcode_class == OpcodeClass.WmmaTensorOp` and selects the matching branch in function `core_name`.
**CN:** 检查 `self.tile_description.math_instruction.opcode_class == OpcodeClass.TensorOp or self.tile_description.math_instruction.opcode_class == OpcodeClass.WmmaTensorOp`，并在函数 `core_name` 中选择匹配的分支。

##### Line 1651 — Return

```python
        return "%s%s%s%s" % (self.short_math_name(), inst_shape, intermediate_type, GemmKindNames[self.gemm_kind])
```
**EN:** Returns `'%s%s%s%s' % (self.short_math_name(), inst_shape, intermediate_type, GemmKindNames[self.gemm_kind])` to the caller.
**CN:** 向调用方返回 `'%s%s%s%s' % (self.short_math_name(), inst_shape, intermediate_type, GemmKindNames[self.gemm_kind])`。

#### Lines 1653-1654 — Function `extended_name`

```python
    def extended_name(self):
        """Append data types if they differ from compute type."""
```
**EN:** Defines function `extended_name` with parameters `self`. Purpose: Append data types if they differ from compute type.
**CN:** 定义函数 `extended_name`，参数为 `self`。 其用途：Append data types if they differ from compute type.

##### Lines 1655-1665 — Conditional `self.is_complex()`

```python
        if self.is_complex():
            extended_name = "${core_name}"
        else:
            if (self.C.element != self.tile_description.math_instruction.element_accumulator and
                self.A.element != self.tile_description.math_instruction.element_accumulator):
                extended_name = "${element_c}_${core_name}_${element_a}"
            elif (self.C.element == self.tile_description.math_instruction.element_accumulator and
                self.A.element != self.tile_description.math_instruction.element_accumulator):
                extended_name = "${core_name}_${element_a}"
            else:
                extended_name = "${core_name}"
```
**EN:** Checks `self.is_complex()` and selects the matching branch in function `extended_name`.
**CN:** 检查 `self.is_complex()`，并在函数 `extended_name` 中选择匹配的分支。

##### Lines 1667-1671 — Assign `extended_name`

```python
        extended_name = SubstituteTemplate(extended_name, {
            "element_a": DataTypeNames[self.A.element],
            "element_c": DataTypeNames[self.C.element],
            "core_name": self.core_name(),
        })
```
**EN:** Assigns `extended_name` from `SubstituteTemplate(extended_name, {'element_a': DataTypeNames[self.A.element], 'element_c': DataTypeNames[self.C.element], 'core_name': self.core_name()})`, establishing state in function `extended_name`.
**CN:** 将 `extended_name` 赋值为 `SubstituteTemplate(extended_name, {'element_a': DataTypeNames[self.A.element], 'element_c': DataTypeNames[self.C.element], 'core_name': self.core_name()})`，用于在函数 `extended_name` 中建立状态。

##### Line 1673 — Return

```python
        return extended_name
```
**EN:** Returns `extended_name` to the caller.
**CN:** 向调用方返回 `extended_name`。

#### Lines 1675-1676 — Function `extended_name_3x`

```python
    def extended_name_3x(self):
        """Generates a string representing the MMA atom. Assumes accumulator type is C type."""
```
**EN:** Defines function `extended_name_3x` with parameters `self`. Purpose: Generates a string representing the MMA atom.
**CN:** 定义函数 `extended_name_3x`，参数为 `self`。 其用途：Generates a string representing the MMA atom.

##### Lines 1677-1683 — Assign `extended_name`

```python
        extended_name = "{core_name}_{element_a}_{element_b}_{element_acc}_{element_c}_{element_d}".format(
            element_a=DataTypeNames[self.A.element],
            element_b=DataTypeNames[self.B.element],
            element_acc=DataTypeNames[self.accumulator_type()],
            element_c=DataTypeNames[self.C.element],
            element_d=DataTypeNames[self.epilogue_functor.element_output],
            core_name=self.core_name())
```
**EN:** Assigns `extended_name` from `'{core_name}_{element_a}_{element_b}_{element_acc}_{element_c}_{element_d}'.format(element_a=DataTypeNames[self.A.element], element_b=DataTypeNames[self.B.el...`, establishing state in function `extended_name_3x`.
**CN:** 将 `extended_name` 赋值为 `'{core_name}_{element_a}_{element_b}_{element_acc}_{element_c}_{element_d}'.format(element_a=DataTypeNames[self.A.element], element_b=DataTypeNames[self.B.el...`，用于在函数 `extended_name_3x` 中建立状态。

##### Line 1684 — Return

```python
        return extended_name
```
**EN:** Returns `extended_name` to the caller.
**CN:** 向调用方返回 `extended_name`。

#### Line 1686 — Function `layout_name`

```python
    def layout_name(self):
```
**EN:** Defines function `layout_name` with parameters `self`.
**CN:** 定义函数 `layout_name`，参数为 `self`。

##### Lines 1687-1691 — Conditional `self.is_complex() or self.is_planar_complex()`

```python
        if self.is_complex() or self.is_planar_complex():
            return "%s%s" % (
                ShortComplexLayoutNames[(self.A.layout, self.A.complex_transform)],
                ShortComplexLayoutNames[(self.B.layout, self.B.complex_transform)]
            )
```
**EN:** Checks `self.is_complex() or self.is_planar_complex()` and selects the matching branch in function `layout_name`.
**CN:** 检查 `self.is_complex() or self.is_planar_complex()`，并在函数 `layout_name` 中选择匹配的分支。

##### Line 1692 — Return

```python
        return "%s%s" % (ShortLayoutTypeNames[self.A.layout], ShortLayoutTypeNames[self.B.layout])
```
**EN:** Returns `'%s%s' % (ShortLayoutTypeNames[self.A.layout], ShortLayoutTypeNames[self.B.layout])` to the caller.
**CN:** 向调用方返回 `'%s%s' % (ShortLayoutTypeNames[self.A.layout], ShortLayoutTypeNames[self.B.layout])`。

#### Line 1694 — Comment or spacing block

```python
    # Generates a short string representing the ABC layout tags (e.g. ntn or tnn)
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 1695 — Function `layout_name_3x`

```python
    def layout_name_3x(self):
```
**EN:** Defines function `layout_name_3x` with parameters `self`.
**CN:** 定义函数 `layout_name_3x`，参数为 `self`。

##### Lines 1696-1705 — Conditional `self.is_complex() or self.is_planar_complex()`

```python
        if self.is_complex() or self.is_planar_complex():
            return "{}{}{}".format(
                ShortComplexLayoutNames[(self.A.layout, self.A.complex_transform)],
                ShortComplexLayoutNames[(self.B.layout, self.B.complex_transform)],
                ShortComplexLayoutNames[(self.C.layout, self.C.complex_transform)])
        else:
            return "{}{}{}".format(
                ShortLayoutTypeNames[self.A.layout],
                ShortLayoutTypeNames[self.B.layout],
                ShortLayoutTypeNames[self.C.layout])
```
**EN:** Checks `self.is_complex() or self.is_planar_complex()` and selects the matching branch in function `layout_name_3x`.
**CN:** 检查 `self.is_complex() or self.is_planar_complex()`，并在函数 `layout_name_3x` 中选择匹配的分支。

#### Line 1707 — Comment or spacing block

```python
    # Generates a short string representing underlying kernel schedule type
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 1708 — Function `kernel_schedule_name_3x`

```python
    def kernel_schedule_name_3x(self):
```
**EN:** Defines function `kernel_schedule_name_3x` with parameters `self`.
**CN:** 定义函数 `kernel_schedule_name_3x`，参数为 `self`。

##### Lines 1709-1712 — Conditional `self.tile_description.kernel_schedule is None`

```python
        if self.tile_description.kernel_schedule is None:
            return KernelScheduleSuffixes[KernelScheduleType.ScheduleAuto]
        else:
            return KernelScheduleSuffixes[self.tile_description.kernel_schedule]
```
**EN:** Checks `self.tile_description.kernel_schedule is None` and selects the matching branch in function `kernel_schedule_name_3x`.
**CN:** 检查 `self.tile_description.kernel_schedule is None`，并在函数 `kernel_schedule_name_3x` 中选择匹配的分支。

#### Line 1714 — Comment or spacing block

```python
    # Generates a short string representing underlying epilogue schedule type
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 1715 — Function `epilogue_schedule_name_3x`

```python
    def epilogue_schedule_name_3x(self):
```
**EN:** Defines function `epilogue_schedule_name_3x` with parameters `self`.
**CN:** 定义函数 `epilogue_schedule_name_3x`，参数为 `self`。

##### Lines 1716-1719 — Conditional `self.tile_description.epilogue_schedule is None`

```python
        if self.tile_description.epilogue_schedule is None:
            return EpilogueScheduleSuffixes[EpilogueScheduleType.ScheduleAuto]
        else:
            return EpilogueScheduleSuffixes[self.tile_description.epilogue_schedule]
```
**EN:** Checks `self.tile_description.epilogue_schedule is None` and selects the matching branch in function `epilogue_schedule_name_3x`.
**CN:** 检查 `self.tile_description.epilogue_schedule is None`，并在函数 `epilogue_schedule_name_3x` 中选择匹配的分支。

#### Lines 1721-1722 — Function `procedural_name`

```python
    def procedural_name(self):
        """The full procedural name indicates architecture, extended name, tile size, and layout."""
```
**EN:** Defines function `procedural_name` with parameters `self`. Purpose: The full procedural name indicates architecture, extended name, tile size, and layout.
**CN:** 定义函数 `procedural_name`，参数为 `self`。 其用途：The full procedural name indicates architecture, extended name, tile size, and layout.

##### Line 1723 — Assign `opcode_class_name`

```python
        opcode_class_name = OpcodeClassNames[self.tile_description.math_instruction.opcode_class]
```
**EN:** Assigns `opcode_class_name` from `OpcodeClassNames[self.tile_description.math_instruction.opcode_class]`, establishing state in function `procedural_name`.
**CN:** 将 `opcode_class_name` 赋值为 `OpcodeClassNames[self.tile_description.math_instruction.opcode_class]`，用于在函数 `procedural_name` 中建立状态。

##### Lines 1724-1757 — Conditional `self.api == ApiVersion.v3x and (self.arch >= 90 or is_intel_xe_arch(self.arch))`

```python
        if self.api == ApiVersion.v3x and (self.arch >= 90 or is_intel_xe_arch(self.arch)):
            arch_prefix="sm"
            if is_intel_xe_arch(self.arch):
                arch_prefix="Xe"
            
            kernel_name_template = "cutlass{p}_{sm_or_xe}{ar}_{op}_{ex}_{tbm}x{tbn}x{tbk}_{cm}x{cn}x{ck}_{l}_{s}_align{al}{k}{e}"
            return kernel_name_template.format(
                p=self.prefix,
                sm_or_xe=arch_prefix,
                ar=self.arch,
                op=opcode_class_name,
                ex=self.extended_name_3x(),
                tbm=self.tile_description.threadblock_shape[0],
                tbn=self.tile_description.threadblock_shape[1],
                tbk=self.tile_description.threadblock_shape[2],
                cm=self.tile_description.cluster_shape[0],
                cn=self.tile_description.cluster_shape[1],
                ck=self.tile_description.cluster_shape[2],
                l=self.tile_description.stages,
                s=self.layout_name_3x(),
                al=str(self.A.alignment),
                k=self.kernel_schedule_name_3x(),
                e=self.epilogue_schedule_name_3x()
            )
        else:
            threadblock = self.tile_description.procedural_name_2x()
            return "cutlass{p}_{op}_{ex}_{tb}_{l}_align{a}".format(
                p=self.prefix,
                op=opcode_class_name,
                ex=self.extended_name(),
                tb=threadblock,
                l=self.layout_name(),
                a=str(self.A.alignment)
            )
```
**EN:** Checks `self.api == ApiVersion.v3x and (self.arch >= 90 or is_intel_xe_arch(self.arch))` and selects the matching branch in function `procedural_name`.
**CN:** 检查 `self.api == ApiVersion.v3x and (self.arch >= 90 or is_intel_xe_arch(self.arch))`，并在函数 `procedural_name` 中选择匹配的分支。

#### Lines 1759-1760 — Function `configuration_name`

```python
    def configuration_name(self):
        """The full procedural name indicates architecture, extended name, tile size, and layout."""
```
**EN:** Defines function `configuration_name` with parameters `self`. Purpose: The full procedural name indicates architecture, extended name, tile size, and layout.
**CN:** 定义函数 `configuration_name`，参数为 `self`。 其用途：The full procedural name indicates architecture, extended name, tile size, and layout.

##### Line 1761 — Return

```python
        return self.procedural_name()
```
**EN:** Returns `self.procedural_name()` to the caller.
**CN:** 向调用方返回 `self.procedural_name()`。

### Line 1764 — Class `GemmOperationUniversal`

```python
class GemmOperationUniversal(GemmOperationBase):
```
**EN:** Declares class `GemmOperationUniversal` deriving from `GemmOperationBase`.
**CN:** 声明类 `GemmOperationUniversal`，其基类为 `GemmOperationBase`。

#### Lines 1765-1766 — Function `__init__`

```python
    def __init__(self, arch, tile_description: TileDescription, A: TensorDescription, B, C,
        epilogue_functor, swizzling_functor=SwizzlingFunctor.Identity1, **kwargs):
```
**EN:** Defines function `__init__` with parameters `self, arch, tile_description, A, B, C, epilogue_functor, swizzling_functor, **kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, arch, tile_description, A, B, C, epilogue_functor, swizzling_functor, **kwargs`。

##### Line 1767 — Assign `api`

```python
        api = api_version(arch, tile_description.math_instruction.opcode_class, A.element)
```
**EN:** Assigns `api` from `api_version(arch, tile_description.math_instruction.opcode_class, A.element)`, establishing state in function `__init__`.
**CN:** 将 `api` 赋值为 `api_version(arch, tile_description.math_instruction.opcode_class, A.element)`，用于在函数 `__init__` 中建立状态。

##### Lines 1768-1770 — Call `super(GemmOperationUniversal, self).__init__`

```python
        super(GemmOperationUniversal, self).__init__(GemmKind.Universal, arch, tile_description,
                                                     A, B, C, epilogue_functor, swizzling_functor,
                                                     api=api, **kwargs, )
```
**EN:** Calls `super(GemmOperationUniversal, self).__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super(GemmOperationUniversal, self).__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Lines 1771-1779 — Conditional `api == ApiVersion.v3x`

```python
        if api == ApiVersion.v3x:
            if swizzling_functor == SwizzlingFunctor.StreamK:
                raise Exception("Stream K swizzle functor is currently only supported for CUTLASS 2.x kernels")
            self.rt_module = GemmRTUniversal3x(self)
        else:
            if swizzling_functor == SwizzlingFunctor.StreamK:
                self.rt_module = GemmRTUniversalStreamK(self)
            else:
                self.rt_module = GemmRTUniversal(self)
```
**EN:** Checks `api == ApiVersion.v3x` and selects the matching branch in function `__init__`.
**CN:** 检查 `api == ApiVersion.v3x`，并在函数 `__init__` 中选择匹配的分支。

##### Line 1780 — Assign `self.argument_type`

```python
        self.argument_type = self.rt_module.argument_type
```
**EN:** Assigns `self.argument_type` from `self.rt_module.argument_type`, establishing state in function `__init__`.
**CN:** 将 `self.argument_type` 赋值为 `self.rt_module.argument_type`，用于在函数 `__init__` 中建立状态。

##### Line 1781 — Assign `self.epilogue_type`

```python
        self.epilogue_type = self.rt_module.epilogue_type
```
**EN:** Assigns `self.epilogue_type` from `self.rt_module.epilogue_type`, establishing state in function `__init__`.
**CN:** 将 `self.epilogue_type` 赋值为 `self.rt_module.epilogue_type`，用于在函数 `__init__` 中建立状态。

#### Lines 1783-1791 — Function `device_op`

```python
    def device_op(self):
        """
        Returns a new GemmOperationUniversal object that is constructed with emission type
        ``EmissionType.Device``. Since the device-emitted kernel does not require swapping,
        any swappng performed by the kernel-emitted operation is reversed.

        :return: operation ready for device-level code emission
        :rtype: GemmUniversalOperation
        """
```
**EN:** Defines function `device_op` with parameters `self`. Purpose: Returns a new GemmOperationUniversal object that is constructed with emission type
**CN:** 定义函数 `device_op`，参数为 `self`。 其用途：Returns a new GemmOperationUniversal object that is constructed with emission type

##### Line 1792 — Assign `A, B, C`

```python
        A, B, C = GemmOperationBase.get_operands(self.A, self.B, self.C, self.switched)
```
**EN:** Assigns `A, B, C` from `GemmOperationBase.get_operands(self.A, self.B, self.C, self.switched)`, establishing state in function `device_op`.
**CN:** 将 `A, B, C` 赋值为 `GemmOperationBase.get_operands(self.A, self.B, self.C, self.switched)`，用于在函数 `device_op` 中建立状态。

##### Lines 1793-1795 — Return

```python
        return GemmOperationUniversal(self.arch, self.tile_description, A, B, C,
                                      self.epilogue_functor, self.swizzling_functor,
                                      emission_type=EmissionType.Device, direct_store=self.direct_store)
```
**EN:** Returns `GemmOperationUniversal(self.arch, self.tile_description, A, B, C, self.epilogue_functor, self.swizzling_functor, emission_type=EmissionType.Device, direct_st...` to the caller.
**CN:** 向调用方返回 `GemmOperationUniversal(self.arch, self.tile_description, A, B, C, self.epilogue_functor, self.swizzling_functor, emission_type=EmissionType.Device, direct_st...`。

### Line 1798 — Class `GemmOperationGrouped`

```python
class GemmOperationGrouped(GemmOperationBase):
```
**EN:** Declares class `GemmOperationGrouped` deriving from `GemmOperationBase`.
**CN:** 声明类 `GemmOperationGrouped`，其基类为 `GemmOperationBase`。

#### Lines 1799-1800 — Function `__init__`

```python
    def __init__(self, arch, tile_description: TileDescription, A: TensorDescription, B, C,
        epilogue_functor, swizzling_functor=SwizzlingFunctor.Identity1, **kwargs):
```
**EN:** Defines function `__init__` with parameters `self, arch, tile_description, A, B, C, epilogue_functor, swizzling_functor, **kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, arch, tile_description, A, B, C, epilogue_functor, swizzling_functor, **kwargs`。

##### Lines 1801-1802 — Call `super(GemmOperationGrouped, self).__init__`

```python
        super(GemmOperationGrouped, self).__init__(GemmKind.Grouped, arch, tile_description,
                                                   A, B, C, epilogue_functor, swizzling_functor, **kwargs)
```
**EN:** Calls `super(GemmOperationGrouped, self).__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super(GemmOperationGrouped, self).__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 1803 — Assertion

```python
        assert "precompute_mode" in kwargs.keys(), "missing keyword arguement 'precompute_mode'."
```
**EN:** Asserts `'precompute_mode' in kwargs.keys()` to enforce an expected condition.
**CN:** 断言 `'precompute_mode' in kwargs.keys()`，用于保证预期条件成立。

##### Line 1804 — Assign `self.precompute_mode`

```python
        self.precompute_mode = kwargs["precompute_mode"]
```
**EN:** Assigns `self.precompute_mode` from `kwargs['precompute_mode']`, establishing state in function `__init__`.
**CN:** 将 `self.precompute_mode` 赋值为 `kwargs['precompute_mode']`，用于在函数 `__init__` 中建立状态。

##### Line 1805 — Assign `self.rt_module`

```python
        self.rt_module = GemmRTGrouped(self)
```
**EN:** Assigns `self.rt_module` from `GemmRTGrouped(self)`, establishing state in function `__init__`.
**CN:** 将 `self.rt_module` 赋值为 `GemmRTGrouped(self)`，用于在函数 `__init__` 中建立状态。

##### Line 1806 — Assign `self.argument_type`

```python
        self.argument_type = self.rt_module.argument_type
```
**EN:** Assigns `self.argument_type` from `self.rt_module.argument_type`, establishing state in function `__init__`.
**CN:** 将 `self.argument_type` 赋值为 `self.rt_module.argument_type`，用于在函数 `__init__` 中建立状态。

##### Line 1807 — Assign `self.epilogue_type`

```python
        self.epilogue_type = self.rt_module.epilogue_type
```
**EN:** Assigns `self.epilogue_type` from `self.rt_module.epilogue_type`, establishing state in function `__init__`.
**CN:** 将 `self.epilogue_type` 赋值为 `self.rt_module.epilogue_type`，用于在函数 `__init__` 中建立状态。

#### Lines 1809-1817 — Function `device_op`

```python
    def device_op(self):
        """
        Returns a new GemmOperationGrouped object that is constructed with emission type
        ``EmissionType.Device``. Since the device-emitted kernel does not require swapping,
        any swappng performed by the kernel-emitted operation is reversed.

        :return: operation ready for device-level code emission
        :rtype: GemmOperationGrouped
        """
```
**EN:** Defines function `device_op` with parameters `self`. Purpose: Returns a new GemmOperationGrouped object that is constructed with emission type
**CN:** 定义函数 `device_op`，参数为 `self`。 其用途：Returns a new GemmOperationGrouped object that is constructed with emission type

##### Line 1818 — Assign `A, B, C`

```python
        A, B, C = GemmOperationBase.get_operands(self.A, self.B, self.C, self.switched)
```
**EN:** Assigns `A, B, C` from `GemmOperationBase.get_operands(self.A, self.B, self.C, self.switched)`, establishing state in function `device_op`.
**CN:** 将 `A, B, C` 赋值为 `GemmOperationBase.get_operands(self.A, self.B, self.C, self.switched)`，用于在函数 `device_op` 中建立状态。

##### Lines 1819-1822 — Return

```python
        return GemmOperationGrouped(
            self.arch, self.tile_description, A, B, C, self.epilogue_functor,
            self.swizzling_functor, emission_type=EmissionType.Device,
            direct_store=self.direct_store, precompute_mode=self.precompute_mode, )
```
**EN:** Returns `GemmOperationGrouped(self.arch, self.tile_description, A, B, C, self.epilogue_functor, self.swizzling_functor, emission_type=EmissionType.Device, direct_stor...` to the caller.
**CN:** 向调用方返回 `GemmOperationGrouped(self.arch, self.tile_description, A, B, C, self.epilogue_functor, self.swizzling_functor, emission_type=EmissionType.Device, direct_stor...`。

### Lines 1825-1829 — Comment or spacing block

```python
###################################################################################################
#
# Emits single instances of a CUTLASS device-wide operator
#
###################################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Lines 1832-1833 — Class `EmitGemmUniversalInstance`

```python
class EmitGemmUniversalInstance:
    """Responsible for emitting a CUTLASS template definition"""
```
**EN:** Declares class `EmitGemmUniversalInstance` deriving from `object`. Purpose: Responsible for emitting a CUTLASS template definition
**CN:** 声明类 `EmitGemmUniversalInstance`，其基类为 `object`。 其用途：Responsible for emitting a CUTLASS template definition

#### Lines 1835-1839 — Function `__init__`

```python
    def __init__(
        self,
        operation_suffix="",
        direct_store=False
    ):
```
**EN:** Defines function `__init__` with parameters `self, operation_suffix, direct_store`.
**CN:** 定义函数 `__init__`，参数为 `self, operation_suffix, direct_store`。

##### Line 1840 — Assign `self.operation_suffix`

```python
        self.operation_suffix = operation_suffix
```
**EN:** Assigns `self.operation_suffix` from `operation_suffix`, establishing state in function `__init__`.
**CN:** 将 `self.operation_suffix` 赋值为 `operation_suffix`，用于在函数 `__init__` 中建立状态。

##### Line 1841 — Assign `self.direct_store`

```python
        self.direct_store = direct_store
```
**EN:** Assigns `self.direct_store` from `direct_store`, establishing state in function `__init__`.
**CN:** 将 `self.direct_store` 赋值为 `direct_store`，用于在函数 `__init__` 中建立状态。

##### Lines 1842-1852 — Assign `self.includes`

```python
        self.includes = [
            "cutlass/cutlass.h",
            "cutlass/gemm_coord.h",
            "cutlass/numeric_types.h",
            "cutlass/arch/arch.h",
            "cutlass/arch/mma.h",
            "cutlass/layout/matrix.h",
            "cutlass/gemm/device/gemm.h",
            "cutlass/gemm/device/gemm_universal_adapter.h",
            "cutlass/gemm/kernel/default_gemm_universal.h",
        ]
```
**EN:** Assigns `self.includes` from `['cutlass/cutlass.h', 'cutlass/gemm_coord.h', 'cutlass/numeric_types.h', 'cutlass/arch/arch.h', 'cutlass/arch/mma.h', 'cutlass/layout/matrix.h', 'cutlass/gem...`, establishing state in function `__init__`.
**CN:** 将 `self.includes` 赋值为 `['cutlass/cutlass.h', 'cutlass/gemm_coord.h', 'cutlass/numeric_types.h', 'cutlass/arch/arch.h', 'cutlass/arch/mma.h', 'cutlass/layout/matrix.h', 'cutlass/gem...`，用于在函数 `__init__` 中建立状态。

##### Lines 1853-1856 — Conditional `self.direct_store`

```python
        if self.direct_store:
            self.includes.append(
                "cutlass/epilogue/threadblock/default_epilogue_direct_store.h"
            )
```
**EN:** Checks `self.direct_store` and selects the matching branch in function `__init__`.
**CN:** 检查 `self.direct_store`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 1857-1879 — Assign `self.gemm_template_kernel`

```python
        self.gemm_template_kernel = """
// Gemm operator ${operation_name}
using ${operation_name}_base =
  typename cutlass::gemm::kernel::DefaultGemmUniversal<
    ${element_a}, ${layout_a}, ${transform_a}, ${align_a},
    ${element_b}, ${layout_b}, ${transform_b}, ${align_b},
    ${element_c}, ${layout_c},
    ${element_accumulator},
    ${opcode_class},
    ${arch},
    cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
    cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
    cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
    ${epilogue_functor},
    ${swizzling_functor},
    ${stages},
    ${math_operation}
>::GemmKernel;

// Define named type
struct ${operation_name}${operation_suffix} :
  public ${operation_name}_base { };
"""
```
**EN:** Assigns `self.gemm_template_kernel` from `'\n// Gemm operator ${operation_name}\nusing ${operation_name}_base =\n typename cutlass::gemm::kernel::DefaultGemmUniversal<\n ${element_a}, ${layout_a}, ${...`, establishing state in function `__init__`.
**CN:** 将 `self.gemm_template_kernel` 赋值为 `'\n// Gemm operator ${operation_name}\nusing ${operation_name}_base =\n typename cutlass::gemm::kernel::DefaultGemmUniversal<\n ${element_a}, ${layout_a}, ${...`，用于在函数 `__init__` 中建立状态。

##### Lines 1881-1916 — Assign `self.gemm_template_device`

```python
        self.gemm_template_device = """
// Gemm operator ${operation_name}
using DeviceKernel =
    typename cutlass::gemm::device::GemmUniversal<
        // Data type and layout of operand A
        ${element_a}, ${layout_a},
        // Data type and layout of operand B
        ${element_b}, ${layout_b},
        // Data type and layout of operand C
        ${element_c}, ${layout_c},
        // Data type of accumulator
        ${element_accumulator},
        // Class of operation
        ${opcode_class},
        // Compute capability of the target kernel
        ${arch},
        // Threadblock tile shape
        cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
        // Warp tile shape
        cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
        // Instruction shape
        cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
        // Epilogue functor
        ${epilogue_functor},
        // Swizzling function
        ${swizzling_functor},
        // Number of pipeline stages
        ${stages},
        // Alignment of operands A and B
        ${align_a}, ${align_b},
        // Type of math operation
        ${math_operation},
        // Complex transform types of operands A and B
        ${transform_a}, ${transform_b}
    >;
"""
```
**EN:** Assigns `self.gemm_template_device` from `'\n// Gemm operator ${operation_name}\nusing DeviceKernel =\n typename cutlass::gemm::device::GemmUniversal<\n // Data type and layout of operand A\n ${eleme...`, establishing state in function `__init__`.
**CN:** 将 `self.gemm_template_device` 赋值为 `'\n// Gemm operator ${operation_name}\nusing DeviceKernel =\n typename cutlass::gemm::device::GemmUniversal<\n // Data type and layout of operand A\n ${eleme...`，用于在函数 `__init__` 中建立状态。

##### Lines 1917-1948 — Assign `self.gemm_template_direct_store`

```python
        self.gemm_template_direct_store = """
// Gemm operator ${operation_name}
using ${operation_name}_default =
  typename cutlass::gemm::kernel::DefaultGemmUniversal<
    ${element_a}, ${layout_a}, ${transform_a}, ${align_a},
    ${element_b}, ${layout_b}, ${transform_b}, ${align_b},
    ${element_c}, ${layout_c},
    ${element_accumulator},
    ${opcode_class},
    ${arch},
    cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
    cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
    cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
    ${epilogue_functor},
    ${swizzling_functor},
    ${stages},
    ${math_operation}
>::GemmKernel;

using ${operation_name}_base =
  cutlass::gemm::kernel::GemmUniversal<
    ${operation_name}_default::Mma,
    cutlass::epilogue::threadblock::DefaultEpilogueDirectStore<
      ${operation_name}_default::Epilogue
    >::Epilogue,
    ${operation_name}_default::ThreadblockSwizzle
  >;

// Define named type
struct ${operation_name}${operation_suffix} :
  public ${operation_name}_base { };
"""
```
**EN:** Assigns `self.gemm_template_direct_store` from `'\n// Gemm operator ${operation_name}\nusing ${operation_name}_default =\n typename cutlass::gemm::kernel::DefaultGemmUniversal<\n ${element_a}, ${layout_a},...`, establishing state in function `__init__`.
**CN:** 将 `self.gemm_template_direct_store` 赋值为 `'\n// Gemm operator ${operation_name}\nusing ${operation_name}_default =\n typename cutlass::gemm::kernel::DefaultGemmUniversal<\n ${element_a}, ${layout_a},...`，用于在函数 `__init__` 中建立状态。

##### Lines 1949-1984 — Assign `self.gemm_template_kernel_visitor`

```python
        self.gemm_template_kernel_visitor = """

using OutputTileThreadMap = cutlass::epilogue::threadblock::OutputTileThreadLayout<
    cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
    cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
    ${element_c},
    ${align_c},
    ${epilogue_stages} /* epilogue stages */
>;

${callback_decl}

// Gemm operator ${operation_name}
using ${operation_name}_base =
    typename cutlass::gemm::kernel::DefaultGemmWithVisitor<
    ${element_a}, ${layout_a}, ${transform_a}, ${align_a},
    ${element_b}, ${layout_b}, ${transform_b}, ${align_b},
    ${element_c}, ${layout_c}, ${align_c},
    ${element_accumulator},
    ${element_epilogue},
    ${opcode_class},
    ${arch},
    cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
    cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
    cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
    ${callback_name},
    ${swizzling_functor},
    ${stages},
    ${math_operation},
    ${epilogue_stages} /* epilogue stages */
>::GemmKernel;

// Define named type
struct ${operation_name}${operation_suffix} :
  public ${operation_name}_base { };
"""
```
**EN:** Assigns `self.gemm_template_kernel_visitor` from `'\n\nusing OutputTileThreadMap = cutlass::epilogue::threadblock::OutputTileThreadLayout<\n cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_sha...`, establishing state in function `__init__`.
**CN:** 将 `self.gemm_template_kernel_visitor` 赋值为 `'\n\nusing OutputTileThreadMap = cutlass::epilogue::threadblock::OutputTileThreadLayout<\n cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_sha...`，用于在函数 `__init__` 中建立状态。

#### Line 1986 — Function `instance_template`

```python
    def instance_template(self):
```
**EN:** Defines function `instance_template` with parameters `self`.
**CN:** 定义函数 `instance_template`，参数为 `self`。

##### Lines 1987-1993 — Return

```python
        return """
${compile_guard_start}
  manifest.append(new ${gemm_kind}<
      cutlass::gemm::device::GemmUniversalAdapter<${operation_name}>
    >("${operation_name}"));
${compile_guard_end}
"""
```
**EN:** Returns `'\n${compile_guard_start}\n manifest.append(new ${gemm_kind}<\n cutlass::gemm::device::GemmUniversalAdapter<${operation_name}>\n >("${operation_name}"));\n${...` to the caller.
**CN:** 向调用方返回 `'\n${compile_guard_start}\n manifest.append(new ${gemm_kind}<\n cutlass::gemm::device::GemmUniversalAdapter<${operation_name}>\n >("${operation_name}"));\n${...`。

#### Line 1995 — Function `emit`

```python
    def emit(self, operation):
```
**EN:** Defines function `emit` with parameters `self, operation`.
**CN:** 定义函数 `emit`，参数为 `self, operation`。

##### Line 1996 — Assign `threadblock_shape`

```python
        threadblock_shape = operation.tile_description.threadblock_shape
```
**EN:** Assigns `threadblock_shape` from `operation.tile_description.threadblock_shape`, establishing state in function `emit`.
**CN:** 将 `threadblock_shape` 赋值为 `operation.tile_description.threadblock_shape`，用于在函数 `emit` 中建立状态。

##### Line 1997 — Assign `warp_count`

```python
        warp_count = operation.tile_description.warp_count
```
**EN:** Assigns `warp_count` from `operation.tile_description.warp_count`, establishing state in function `emit`.
**CN:** 将 `warp_count` 赋值为 `operation.tile_description.warp_count`，用于在函数 `emit` 中建立状态。

##### Line 1999 — Assign `warp_shape`

```python
        warp_shape = [threadblock_shape[idx] // warp_count[idx] for idx in range(3)]
```
**EN:** Assigns `warp_shape` from `[threadblock_shape[idx] // warp_count[idx] for idx in range(3)]`, establishing state in function `emit`.
**CN:** 将 `warp_shape` 赋值为 `[threadblock_shape[idx] // warp_count[idx] for idx in range(3)]`，用于在函数 `emit` 中建立状态。

##### Lines 2001-2002 — Assign `instance_layout_A, instance_layout_B, instance_layout_C`

```python
        instance_layout_A, instance_layout_B, instance_layout_C = \
            (operation.A.layout, operation.B.layout, operation.C.layout)
```
**EN:** Assigns `instance_layout_A, instance_layout_B, instance_layout_C` from `(operation.A.layout, operation.B.layout, operation.C.layout)`, establishing state in function `emit`.
**CN:** 将 `instance_layout_A, instance_layout_B, instance_layout_C` 赋值为 `(operation.A.layout, operation.B.layout, operation.C.layout)`，用于在函数 `emit` 中建立状态。

##### Lines 2004-2010 — Conditional `operation.emission_type == EmissionType.Kernel`

```python
        if operation.emission_type == EmissionType.Kernel:
            if self.direct_store:
                gemm_template = self.gemm_template_direct_store
            else:
                gemm_template = self.gemm_template_kernel
        else:
            gemm_template = self.gemm_template_device
```
**EN:** Checks `operation.emission_type == EmissionType.Kernel` and selects the matching branch in function `emit`.
**CN:** 检查 `operation.emission_type == EmissionType.Kernel`，并在函数 `emit` 中选择匹配的分支。

##### Lines 2012-2040 — Assign `values`

```python
        values = {
            "operation_name": operation.procedural_name(),
            "operation_suffix": self.operation_suffix,
            "element_a": DataTypeTag[operation.A.element],
            "layout_a": LayoutTag[instance_layout_A],
            "element_b": DataTypeTag[operation.B.element],
            "layout_b": LayoutTag[instance_layout_B],
            "element_c": DataTypeTag[operation.C.element],
            "layout_c": LayoutTag[instance_layout_C],
            "element_accumulator": DataTypeTag[operation.accumulator_type()],
            "opcode_class": OpcodeClassTag[operation.tile_description.math_instruction.opcode_class],
            "arch": "cutlass::arch::Sm%d" % operation.arch,
            "threadblock_shape_m": str(operation.tile_description.threadblock_shape[0]),
            "threadblock_shape_n": str(operation.tile_description.threadblock_shape[1]),
            "threadblock_shape_k": str(operation.tile_description.threadblock_shape[2]),
            "warp_shape_m": str(warp_shape[0]),
            "warp_shape_n": str(warp_shape[1]),
            "warp_shape_k": str(warp_shape[2]),
            "instruction_shape_m": str(operation.tile_description.math_instruction.instruction_shape[0]),
            "instruction_shape_n": str(operation.tile_description.math_instruction.instruction_shape[1]),
            "instruction_shape_k": str(operation.tile_description.math_instruction.instruction_shape[2]),
            "swizzling_functor": SwizzlingFunctorTag[operation.swizzling_functor],
            "stages": str(operation.tile_description.stages),
            "align_a": str(operation.A.alignment),
            "align_b": str(operation.B.alignment),
            "transform_a": ComplexTransformTag[operation.A.complex_transform],
            "transform_b": ComplexTransformTag[operation.B.complex_transform],
            "math_operation": MathOperationTag[operation.tile_description.math_instruction.math_operation],
        }
```
**EN:** Assigns `values` from `{'operation_name': operation.procedural_name(), 'operation_suffix': self.operation_suffix, 'element_a': DataTypeTag[operation.A.element], 'layout_a': LayoutT...`, establishing state in function `emit`.
**CN:** 将 `values` 赋值为 `{'operation_name': operation.procedural_name(), 'operation_suffix': self.operation_suffix, 'element_a': DataTypeTag[operation.A.element], 'layout_a': LayoutT...`，用于在函数 `emit` 中建立状态。

##### Lines 2042-2060 — Conditional `hasattr(operation.epilogue_functor, 'visitor')`

```python
        if hasattr(operation.epilogue_functor, "visitor"):
            self.includes += [
                "cutlass/epilogue/threadblock/fusion/visitors.hpp",
                "cutlass/gemm/kernel/default_gemm_universal_with_visitor.h"
            ]
            callback_name, callback_decl = operation.epilogue_functor.emit(operation)
            values["callback_name"] = callback_name
            values["callback_decl"] = callback_decl
            values["align_c"] = str(operation.C.alignment)
            values["element_epilogue"] = DataTypeTag[operation.epilogue_functor.element_epilogue]
            if hasattr(operation.epilogue_functor, "epilogue_stages"):
                epilogue_stages = operation.epilogue_functor.epilogue_stages
            else:
                epilogue_stages = 1
            values["epilogue_stages"] = str(epilogue_stages)
            return SubstituteTemplate(self.gemm_template_kernel_visitor, values)
        else:
            values["epilogue_functor"] = operation.epilogue_functor.emit()
            return SubstituteTemplate(gemm_template, values)
```
**EN:** Checks `hasattr(operation.epilogue_functor, 'visitor')` and selects the matching branch in function `emit`.
**CN:** 检查 `hasattr(operation.epilogue_functor, 'visitor')`，并在函数 `emit` 中选择匹配的分支。

### Lines 2063-2064 — Class `EmitGemmGroupedInstance`

```python
class EmitGemmGroupedInstance:
    """Responsible for emitting a CUTLASS template definition"""
```
**EN:** Declares class `EmitGemmGroupedInstance` deriving from `object`. Purpose: Responsible for emitting a CUTLASS template definition
**CN:** 声明类 `EmitGemmGroupedInstance`，其基类为 `object`。 其用途：Responsible for emitting a CUTLASS template definition

#### Line 2066 — Function `__init__`

```python
    def __init__(self, operation_suffix=""):
```
**EN:** Defines function `__init__` with parameters `self, operation_suffix`.
**CN:** 定义函数 `__init__`，参数为 `self, operation_suffix`。

##### Line 2067 — Assign `self.operation_suffix`

```python
        self.operation_suffix = operation_suffix
```
**EN:** Assigns `self.operation_suffix` from `operation_suffix`, establishing state in function `__init__`.
**CN:** 将 `self.operation_suffix` 赋值为 `operation_suffix`，用于在函数 `__init__` 中建立状态。

##### Lines 2068-2076 — Assign `self.includes`

```python
        self.includes = [
            "cutlass/cutlass.h",
            "cutlass/numeric_types.h",
            "cutlass/arch/arch.h",
            "cutlass/arch/mma.h",
            "cutlass/layout/matrix.h",
            "cutlass/gemm/kernel/gemm_grouped.h",
            "cutlass/gemm/kernel/default_gemm_grouped.h",
        ]
```
**EN:** Assigns `self.includes` from `['cutlass/cutlass.h', 'cutlass/numeric_types.h', 'cutlass/arch/arch.h', 'cutlass/arch/mma.h', 'cutlass/layout/matrix.h', 'cutlass/gemm/kernel/gemm_grouped.h'...`, establishing state in function `__init__`.
**CN:** 将 `self.includes` 赋值为 `['cutlass/cutlass.h', 'cutlass/numeric_types.h', 'cutlass/arch/arch.h', 'cutlass/arch/mma.h', 'cutlass/layout/matrix.h', 'cutlass/gemm/kernel/gemm_grouped.h'...`，用于在函数 `__init__` 中建立状态。

##### Lines 2077-2100 — Assign `self.gemm_template_kernel`

```python
        self.gemm_template_kernel = """
// Gemm operator ${operation_name}
using ${operation_name}_base =
  typename cutlass::gemm::kernel::DefaultGemmGrouped<
    ${element_a}, ${layout_a}, ${transform_a}, ${align_a},
    ${element_b}, ${layout_b}, ${transform_b}, ${align_b},
    ${element_c}, ${layout_c},
    ${element_accumulator},
    ${opcode_class},
    ${arch},
    cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
    cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k}>,
    cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
    ${epilogue_functor},
    ${swizzling_functor},
    ${stages},
    ${precompute_mode},
    ${math_operation}
>::GemmKernel;

// Define named type
struct ${operation_name}${operation_suffix} :
  public ${operation_name}_base { };
"""
```
**EN:** Assigns `self.gemm_template_kernel` from `'\n// Gemm operator ${operation_name}\nusing ${operation_name}_base =\n typename cutlass::gemm::kernel::DefaultGemmGrouped<\n ${element_a}, ${layout_a}, ${tr...`, establishing state in function `__init__`.
**CN:** 将 `self.gemm_template_kernel` 赋值为 `'\n// Gemm operator ${operation_name}\nusing ${operation_name}_base =\n typename cutlass::gemm::kernel::DefaultGemmGrouped<\n ${element_a}, ${layout_a}, ${tr...`，用于在函数 `__init__` 中建立状态。

##### Lines 2101-2106 — Assign `self.gemm_template_device`

```python
        self.gemm_template_device = (
            self.gemm_template_kernel
            + """
using DeviceKernel = cutlass::gemm::device::GemmGrouped<${operation_name}_base>;
"""
        )
```
**EN:** Assigns `self.gemm_template_device` from `self.gemm_template_kernel + '\nusing DeviceKernel = cutlass::gemm::device::GemmGrouped<${operation_name}_base>;\n'`, establishing state in function `__init__`.
**CN:** 将 `self.gemm_template_device` 赋值为 `self.gemm_template_kernel + '\nusing DeviceKernel = cutlass::gemm::device::GemmGrouped<${operation_name}_base>;\n'`，用于在函数 `__init__` 中建立状态。

#### Line 2108 — Function `instance_template`

```python
    def instance_template(self):
```
**EN:** Defines function `instance_template` with parameters `self`.
**CN:** 定义函数 `instance_template`，参数为 `self`。

##### Lines 2109-2115 — Return

```python
        return """
${compile_guard_start}
  manifest.append(new ${gemm_kind}<
    cutlass::gemm::device::GemmGrouped<${operation_name}>
  >("${operation_name}"));
${compile_guard_end}
"""
```
**EN:** Returns `'\n${compile_guard_start}\n manifest.append(new ${gemm_kind}<\n cutlass::gemm::device::GemmGrouped<${operation_name}>\n >("${operation_name}"));\n${compile_g...` to the caller.
**CN:** 向调用方返回 `'\n${compile_guard_start}\n manifest.append(new ${gemm_kind}<\n cutlass::gemm::device::GemmGrouped<${operation_name}>\n >("${operation_name}"));\n${compile_g...`。

#### Line 2117 — Function `emit`

```python
    def emit(self, operation):
```
**EN:** Defines function `emit` with parameters `self, operation`.
**CN:** 定义函数 `emit`，参数为 `self, operation`。

##### Line 2118 — Assign `threadblock_shape`

```python
        threadblock_shape = operation.tile_description.threadblock_shape
```
**EN:** Assigns `threadblock_shape` from `operation.tile_description.threadblock_shape`, establishing state in function `emit`.
**CN:** 将 `threadblock_shape` 赋值为 `operation.tile_description.threadblock_shape`，用于在函数 `emit` 中建立状态。

##### Line 2119 — Assign `warp_count`

```python
        warp_count = operation.tile_description.warp_count
```
**EN:** Assigns `warp_count` from `operation.tile_description.warp_count`, establishing state in function `emit`.
**CN:** 将 `warp_count` 赋值为 `operation.tile_description.warp_count`，用于在函数 `emit` 中建立状态。

##### Line 2121 — Assign `warp_shape`

```python
        warp_shape = [threadblock_shape[idx] // warp_count[idx] for idx in range(3)]
```
**EN:** Assigns `warp_shape` from `[threadblock_shape[idx] // warp_count[idx] for idx in range(3)]`, establishing state in function `emit`.
**CN:** 将 `warp_shape` 赋值为 `[threadblock_shape[idx] // warp_count[idx] for idx in range(3)]`，用于在函数 `emit` 中建立状态。

##### Lines 2123-2124 — Assign `instance_layout_A, instance_layout_B, instance_layout_C`

```python
        instance_layout_A, instance_layout_B, instance_layout_C = \
            (operation.A.layout, operation.B.layout, operation.C.layout)
```
**EN:** Assigns `instance_layout_A, instance_layout_B, instance_layout_C` from `(operation.A.layout, operation.B.layout, operation.C.layout)`, establishing state in function `emit`.
**CN:** 将 `instance_layout_A, instance_layout_B, instance_layout_C` 赋值为 `(operation.A.layout, operation.B.layout, operation.C.layout)`，用于在函数 `emit` 中建立状态。

##### Line 2126 — Comment or spacing block

```python
        # Support built-in epilogue functors or user-defined functions
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 2127 — Assign `epilogue_functor`

```python
        epilogue_functor = operation.epilogue_functor.emit()
```
**EN:** Assigns `epilogue_functor` from `operation.epilogue_functor.emit()`, establishing state in function `emit`.
**CN:** 将 `epilogue_functor` 赋值为 `operation.epilogue_functor.emit()`，用于在函数 `emit` 中建立状态。

##### Lines 2129-2159 — Assign `values`

```python
        values = {
            "operation_name": operation.procedural_name(),
            "operation_suffix": self.operation_suffix,
            "element_a": DataTypeTag[operation.A.element],
            "layout_a": LayoutTag[instance_layout_A],
            "element_b": DataTypeTag[operation.B.element],
            "layout_b": LayoutTag[instance_layout_B],
            "element_c": DataTypeTag[operation.C.element],
            "layout_c": LayoutTag[instance_layout_C],
            "element_accumulator": DataTypeTag[operation.accumulator_type()],
            "opcode_class": OpcodeClassTag[operation.tile_description.math_instruction.opcode_class],
            "arch": "cutlass::arch::Sm%d" % operation.arch,
            "threadblock_shape_m": str(operation.tile_description.threadblock_shape[0]),
            "threadblock_shape_n": str(operation.tile_description.threadblock_shape[1]),
            "threadblock_shape_k": str(operation.tile_description.threadblock_shape[2]),
            "warp_shape_m": str(warp_shape[0]),
            "warp_shape_n": str(warp_shape[1]),
            "warp_shape_k": str(warp_shape[2]),
            "instruction_shape_m": str(operation.tile_description.math_instruction.instruction_shape[0]),
            "instruction_shape_n": str(operation.tile_description.math_instruction.instruction_shape[1]),
            "instruction_shape_k": str(operation.tile_description.math_instruction.instruction_shape[2]),
            "epilogue_functor": epilogue_functor,
            "swizzling_functor": SwizzlingFunctorTag[operation.swizzling_functor],
            "stages": str(operation.tile_description.stages),
            "align_a": str(operation.A.alignment),
            "align_b": str(operation.B.alignment),
            "transform_a": ComplexTransformTag[operation.A.complex_transform],
            "transform_b": ComplexTransformTag[operation.B.complex_transform],
            "precompute_mode": SchedulerModeTag[operation.precompute_mode],
            "math_operation": MathOperationTag[operation.tile_description.math_instruction.math_operation],
        }
```
**EN:** Assigns `values` from `{'operation_name': operation.procedural_name(), 'operation_suffix': self.operation_suffix, 'element_a': DataTypeTag[operation.A.element], 'layout_a': LayoutT...`, establishing state in function `emit`.
**CN:** 将 `values` 赋值为 `{'operation_name': operation.procedural_name(), 'operation_suffix': self.operation_suffix, 'element_a': DataTypeTag[operation.A.element], 'layout_a': LayoutT...`，用于在函数 `emit` 中建立状态。

##### Lines 2161-2164 — Conditional `operation.emission_type == EmissionType.Kernel`

```python
        if operation.emission_type == EmissionType.Kernel:
            gemm_template = self.gemm_template_kernel
        else:
            gemm_template = self.gemm_template_device
```
**EN:** Checks `operation.emission_type == EmissionType.Kernel` and selects the matching branch in function `emit`.
**CN:** 检查 `operation.emission_type == EmissionType.Kernel`，并在函数 `emit` 中选择匹配的分支。

##### Line 2166 — Return

```python
        return SubstituteTemplate(gemm_template, values)
```
**EN:** Returns `SubstituteTemplate(gemm_template, values)` to the caller.
**CN:** 向调用方返回 `SubstituteTemplate(gemm_template, values)`。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `GemmArguments2x`, `GemmArguments2xStreamK`, `GemmArguments3x`, `GemmGroupedArguments`, `GemmRTbase`, `GemmRTUniversal`, `GemmRTUniversalStreamK`, `GemmRTUniversal3x`, `EmitGemmUniversalInstance3x`, `GemmRTGrouped`, `GemmOperationBase`, `GemmOperationUniversal`, `GemmOperationGrouped`, `EmitGemmUniversalInstance`, `EmitGemmGroupedInstance`.
- **CN:** 顶层类：`GemmArguments2x`, `GemmArguments2xStreamK`, `GemmArguments3x`, `GemmGroupedArguments`, `GemmRTbase`, `GemmRTUniversal`, `GemmRTUniversalStreamK`, `GemmRTUniversal3x`, `EmitGemmUniversalInstance3x`, `GemmRTGrouped`, `GemmOperationBase`, `GemmOperationUniversal`, `GemmOperationGrouped`, `EmitGemmUniversalInstance`, `EmitGemmGroupedInstance`。
- **EN:** Top-level functions: `leading_dimension`, `transpose_layout`, `GemmArguments`.
- **CN:** 顶层函数：`leading_dimension`, `transpose_layout`, `GemmArguments`。
- **EN:** This module organizes backend utilities, data structures, or runtime helpers used by cutlass_cppgen.
- **CN:** 该模块组织 cutlass_cppgen 所需的后端工具、数据结构或运行时辅助逻辑。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.arguments`, `cutlass_cppgen.backend.c_types`, `cutlass_cppgen.backend.library`, `cutlass_cppgen.backend.memory_manager`, `cutlass_cppgen.backend.operation`, `cutlass_cppgen.backend.type_hint`, `cutlass_cppgen.backend.utils.device`, `cutlass_cppgen.shape`, `cutlass_cppgen.utils.lazy_import`, `cutlass_library`, `cutlass_library.arch_constants`
- **Standard & third-party / 标准库与第三方:** `__future__`, `copy`, `ctypes`, `enum`, `numpy`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
