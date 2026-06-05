# c_types.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/c_types.py`
- **EN:** Defines classes `GemmCoord_`, `GemmCoordBatched_`, `MatrixCoord_`, `dim3_`, `StrideBatched_`, `GenericMainloopArguments3x_` and functions `get_tile_scheduler_arguments_3x`, `get_mainloop_arguments_3x`, `get_gemm_arguments_3x`, `get_gemm_arguments`, `get_gemm_arguments_streamk`, `get_gemm_grouped_arguments` for the cutlass_cppgen package.
- **CN:** 为 cutlass_cppgen 包定义类 `GemmCoord_`, `GemmCoordBatched_`, `MatrixCoord_`, `dim3_`, `StrideBatched_`, `GenericMainloopArguments3x_`和函数 `get_tile_scheduler_arguments_3x`, `get_mainloop_arguments_3x`, `get_gemm_arguments_3x`, `get_gemm_arguments`, `get_gemm_arguments_streamk`, `get_gemm_grouped_arguments`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32 — File header

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

### Line 33 — Import `ctypes`

```python
import ctypes
```
**EN:** Imports `ctypes` so later code can use these APIs at module scope.
**CN:** 导入 `ctypes`，供后续代码在模块级使用这些 API。

### Lines 35-39 — From `cutlass_library` import

```python
from cutlass_library import (
    DataType,
    KernelScheduleType,
    TileSchedulerType
)
```
**EN:** Imports `DataType, KernelScheduleType, TileSchedulerType` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `DataType, KernelScheduleType, TileSchedulerType`，以便后续代码在模块级复用共享定义。

### Line 40 — From `cutlass_cppgen.backend.library` import

```python
from cutlass_cppgen.backend.library import DataTypeSizeBytes
```
**EN:** Imports `DataTypeSizeBytes` from `cutlass_cppgen.backend.library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.library` 导入 `DataTypeSizeBytes`，以便后续代码在模块级复用共享定义。

### Line 43 — Class `GemmCoord_`

```python
class GemmCoord_(ctypes.Structure):
```
**EN:** Declares class `GemmCoord_` deriving from `ctypes.Structure`.
**CN:** 声明类 `GemmCoord_`，其基类为 `ctypes.Structure`。

#### Lines 44-48 — Assign `_fields_`

```python
    _fields_ = [
        ("m", ctypes.c_int),
        ("n", ctypes.c_int),
        ("k", ctypes.c_int)
    ]
```
**EN:** Assigns `_fields_` from `[('m', ctypes.c_int), ('n', ctypes.c_int), ('k', ctypes.c_int)]`, establishing state in class `GemmCoord_`.
**CN:** 将 `_fields_` 赋值为 `[('m', ctypes.c_int), ('n', ctypes.c_int), ('k', ctypes.c_int)]`，用于在类 `GemmCoord_` 中建立状态。

#### Line 50 — Function `__init__`

```python
    def __init__(self, m, n, k) -> None:
```
**EN:** Defines function `__init__` with parameters `self, m, n, k`.
**CN:** 定义函数 `__init__`，参数为 `self, m, n, k`。

##### Line 51 — Assign `self.m`

```python
        self.m = m
```
**EN:** Assigns `self.m` from `m`, establishing state in function `__init__`.
**CN:** 将 `self.m` 赋值为 `m`，用于在函数 `__init__` 中建立状态。

##### Line 52 — Assign `self.n`

```python
        self.n = n
```
**EN:** Assigns `self.n` from `n`, establishing state in function `__init__`.
**CN:** 将 `self.n` 赋值为 `n`，用于在函数 `__init__` 中建立状态。

##### Line 53 — Assign `self.k`

```python
        self.k = k
```
**EN:** Assigns `self.k` from `k`, establishing state in function `__init__`.
**CN:** 将 `self.k` 赋值为 `k`，用于在函数 `__init__` 中建立状态。

### Lines 56-60 — Class `GemmCoordBatched_`

```python
class GemmCoordBatched_(ctypes.Structure):
    """
    Wrapper around a GemmCoord that also contains batch count. This is used for encoding
    batched GEMM inputs to CUTLASS 3 GEMMs.
    """
```
**EN:** Declares class `GemmCoordBatched_` deriving from `ctypes.Structure`. Purpose: Wrapper around a GemmCoord that also contains batch count.
**CN:** 声明类 `GemmCoordBatched_`，其基类为 `ctypes.Structure`。 其用途：Wrapper around a GemmCoord that also contains batch count.

#### Lines 62-67 — Assign `_fields_`

```python
    _fields_ = [
        ("m", ctypes.c_int),
        ("n", ctypes.c_int),
        ("k", ctypes.c_int),
        ("batch_count", ctypes.c_int)
    ]
```
**EN:** Assigns `_fields_` from `[('m', ctypes.c_int), ('n', ctypes.c_int), ('k', ctypes.c_int), ('batch_count', ctypes.c_int)]`, establishing state in class `GemmCoordBatched_`.
**CN:** 将 `_fields_` 赋值为 `[('m', ctypes.c_int), ('n', ctypes.c_int), ('k', ctypes.c_int), ('batch_count', ctypes.c_int)]`，用于在类 `GemmCoordBatched_` 中建立状态。

#### Line 69 — Function `__init__`

```python
    def __init__(self, gemm_coord, batch_count) -> None:
```
**EN:** Defines function `__init__` with parameters `self, gemm_coord, batch_count`.
**CN:** 定义函数 `__init__`，参数为 `self, gemm_coord, batch_count`。

##### Line 70 — Assign `self.m`

```python
        self.m = gemm_coord.m
```
**EN:** Assigns `self.m` from `gemm_coord.m`, establishing state in function `__init__`.
**CN:** 将 `self.m` 赋值为 `gemm_coord.m`，用于在函数 `__init__` 中建立状态。

##### Line 71 — Assign `self.n`

```python
        self.n = gemm_coord.n
```
**EN:** Assigns `self.n` from `gemm_coord.n`, establishing state in function `__init__`.
**CN:** 将 `self.n` 赋值为 `gemm_coord.n`，用于在函数 `__init__` 中建立状态。

##### Line 72 — Assign `self.k`

```python
        self.k = gemm_coord.k
```
**EN:** Assigns `self.k` from `gemm_coord.k`, establishing state in function `__init__`.
**CN:** 将 `self.k` 赋值为 `gemm_coord.k`，用于在函数 `__init__` 中建立状态。

##### Line 73 — Assign `self.batch_count`

```python
        self.batch_count = batch_count
```
**EN:** Assigns `self.batch_count` from `batch_count`, establishing state in function `__init__`.
**CN:** 将 `self.batch_count` 赋值为 `batch_count`，用于在函数 `__init__` 中建立状态。

### Line 76 — Class `MatrixCoord_`

```python
class MatrixCoord_(ctypes.Structure):
```
**EN:** Declares class `MatrixCoord_` deriving from `ctypes.Structure`.
**CN:** 声明类 `MatrixCoord_`，其基类为 `ctypes.Structure`。

#### Lines 77-80 — Assign `_fields_`

```python
    _fields_ = [
        ("row", ctypes.c_int),
        ("column", ctypes.c_int)
    ]
```
**EN:** Assigns `_fields_` from `[('row', ctypes.c_int), ('column', ctypes.c_int)]`, establishing state in class `MatrixCoord_`.
**CN:** 将 `_fields_` 赋值为 `[('row', ctypes.c_int), ('column', ctypes.c_int)]`，用于在类 `MatrixCoord_` 中建立状态。

### Line 83 — Class `dim3_`

```python
class dim3_(ctypes.Structure):
```
**EN:** Declares class `dim3_` deriving from `ctypes.Structure`.
**CN:** 声明类 `dim3_`，其基类为 `ctypes.Structure`。

#### Lines 84-88 — Assign `_fields_`

```python
    _fields_ = [
        ("x", ctypes.c_int),
        ("y", ctypes.c_int),
        ("z", ctypes.c_int)
    ]
```
**EN:** Assigns `_fields_` from `[('x', ctypes.c_int), ('y', ctypes.c_int), ('z', ctypes.c_int)]`, establishing state in class `dim3_`.
**CN:** 将 `_fields_` 赋值为 `[('x', ctypes.c_int), ('y', ctypes.c_int), ('z', ctypes.c_int)]`，用于在类 `dim3_` 中建立状态。

### Lines 91-96 — Class `StrideBatched_`

```python
class StrideBatched_(ctypes.Structure):
    """
    CUTLASS 3.0 strides for operands contain one static dimension and two variable dimensions. The
    variable dimensions represent the stride along non-unit-stride dimension of the row/column major
    layout, and the batch stride. This structure encodes the two variable dimensions.
    """
```
**EN:** Declares class `StrideBatched_` deriving from `ctypes.Structure`. Purpose: CUTLASS 3.0 strides for operands contain one static dimension and two variable dimensions.
**CN:** 声明类 `StrideBatched_`，其基类为 `ctypes.Structure`。 其用途：CUTLASS 3.0 strides for operands contain one static dimension and two variable dimensions.

#### Lines 97-100 — Assign `_fields_`

```python
    _fields_ = [
        ("major_stride", ctypes.c_int64),
        ("batch_stride", ctypes.c_int64)
    ]
```
**EN:** Assigns `_fields_` from `[('major_stride', ctypes.c_int64), ('batch_stride', ctypes.c_int64)]`, establishing state in class `StrideBatched_`.
**CN:** 将 `_fields_` 赋值为 `[('major_stride', ctypes.c_int64), ('batch_stride', ctypes.c_int64)]`，用于在类 `StrideBatched_` 中建立状态。

### Lines 104-110 — Class `GenericMainloopArguments3x_`

```python
class GenericMainloopArguments3x_(ctypes.Structure):
    """
    Structure representing the superset of possible mainloop arguments.
    This structure should not be passed to kernels directly, but, rather,
    be used as an input to one of the more specific schedule arguments, which
    will each select those arguments relevant to the particular schedule.
    """
```
**EN:** Declares class `GenericMainloopArguments3x_` deriving from `ctypes.Structure`. Purpose: Structure representing the superset of possible mainloop arguments.
**CN:** 声明类 `GenericMainloopArguments3x_`，其基类为 `ctypes.Structure`。 其用途：Structure representing the superset of possible mainloop arguments.

#### Lines 111-117 — Assign `_fields_`

```python
    _fields_ = [
        ("ptr_A", ctypes.c_void_p),
        ("stride_A", StrideBatched_),
        ("ptr_B", ctypes.c_void_p),
        ("stride_B", StrideBatched_),
        ("mma_promotion_interval", ctypes.c_int)
    ]
```
**EN:** Assigns `_fields_` from `[('ptr_A', ctypes.c_void_p), ('stride_A', StrideBatched_), ('ptr_B', ctypes.c_void_p), ('stride_B', StrideBatched_), ('mma_promotion_interval', ctypes.c_int)]`, establishing state in class `GenericMainloopArguments3x_`.
**CN:** 将 `_fields_` 赋值为 `[('ptr_A', ctypes.c_void_p), ('stride_A', StrideBatched_), ('ptr_B', ctypes.c_void_p), ('stride_B', StrideBatched_), ('mma_promotion_interval', ctypes.c_int)]`，用于在类 `GenericMainloopArguments3x_` 中建立状态。

### Line 120 — Class `_PersistentTileSchedulerArguments`

```python
class _PersistentTileSchedulerArguments(ctypes.Structure):
```
**EN:** Declares class `_PersistentTileSchedulerArguments` deriving from `ctypes.Structure`.
**CN:** 声明类 `_PersistentTileSchedulerArguments`，其基类为 `ctypes.Structure`。

#### Lines 121-124 — Assign `_fields_`

```python
    _fields_ = [
        ("max_swizzle_size", ctypes.c_int),
        ("raster_order_option", ctypes.c_int),
    ]
```
**EN:** Assigns `_fields_` from `[('max_swizzle_size', ctypes.c_int), ('raster_order_option', ctypes.c_int)]`, establishing state in class `_PersistentTileSchedulerArguments`.
**CN:** 将 `_fields_` 赋值为 `[('max_swizzle_size', ctypes.c_int), ('raster_order_option', ctypes.c_int)]`，用于在类 `_PersistentTileSchedulerArguments` 中建立状态。

### Line 127 — Class `_PersistentTileSchedulerStreamKArguments`

```python
class _PersistentTileSchedulerStreamKArguments(ctypes.Structure):
```
**EN:** Declares class `_PersistentTileSchedulerStreamKArguments` deriving from `ctypes.Structure`.
**CN:** 声明类 `_PersistentTileSchedulerStreamKArguments`，其基类为 `ctypes.Structure`。

#### Lines 128-134 — Assign `_fields_`

```python
    _fields_ = [
        ("splits", ctypes.c_int),
        ("max_swizzle_size", ctypes.c_int),
        ("raster_order_option", ctypes.c_int),
        ("reduction_mode", ctypes.c_int),
        ("decomposition_mode", ctypes.c_int),
    ]
```
**EN:** Assigns `_fields_` from `[('splits', ctypes.c_int), ('max_swizzle_size', ctypes.c_int), ('raster_order_option', ctypes.c_int), ('reduction_mode', ctypes.c_int), ('decomposition_mode'...`, establishing state in class `_PersistentTileSchedulerStreamKArguments`.
**CN:** 将 `_fields_` 赋值为 `[('splits', ctypes.c_int), ('max_swizzle_size', ctypes.c_int), ('raster_order_option', ctypes.c_int), ('reduction_mode', ctypes.c_int), ('decomposition_mode'...`，用于在类 `_PersistentTileSchedulerStreamKArguments` 中建立状态。

### Lines 137-139 — Function `get_tile_scheduler_arguments_3x`

```python
def get_tile_scheduler_arguments_3x(
    tile_scheduler: TileSchedulerType,
    splits: int = 1):
```
**EN:** Defines function `get_tile_scheduler_arguments_3x` with parameters `tile_scheduler, splits`.
**CN:** 定义函数 `get_tile_scheduler_arguments_3x`，参数为 `tile_scheduler, splits`。

#### Line 140 — Assign `max_swizzle_size`

```python
    max_swizzle_size = 1
```
**EN:** Assigns `max_swizzle_size` from `1`, establishing state in function `get_tile_scheduler_arguments_3x`.
**CN:** 将 `max_swizzle_size` 赋值为 `1`，用于在函数 `get_tile_scheduler_arguments_3x` 中建立状态。

#### Line 141 — Assign `raster_order_option`

```python
    raster_order_option = 0 # Heuristic
```
**EN:** Assigns `raster_order_option` from `0`, establishing state in function `get_tile_scheduler_arguments_3x`.
**CN:** 将 `raster_order_option` 赋值为 `0`，用于在函数 `get_tile_scheduler_arguments_3x` 中建立状态。

#### Lines 142-156 — Conditional `tile_scheduler in [TileSchedulerType.Default, TileSchedulerType.Persistent]`

```python
    if tile_scheduler in [TileSchedulerType.Default, TileSchedulerType.Persistent]:
        return _PersistentTileSchedulerArguments(
            max_swizzle_size,
            raster_order_option,
        )
    elif tile_scheduler == TileSchedulerType.StreamK:
        reduction_mode = 0 # Deterministic
        decomposition_mode = 0 # Heuristic
        return _PersistentTileSchedulerStreamKArguments(
            splits,
            max_swizzle_size,
            raster_order_option,
            reduction_mode,
            decomposition_mode,
        )
```
**EN:** Checks `tile_scheduler in [TileSchedulerType.Default, TileSchedulerType.Persistent]` and selects the matching branch in function `get_tile_scheduler_arguments_3x`.
**CN:** 检查 `tile_scheduler in [TileSchedulerType.Default, TileSchedulerType.Persistent]`，并在函数 `get_tile_scheduler_arguments_3x` 中选择匹配的分支。

### Lines 159-180 — Function `get_mainloop_arguments_3x`

```python
def get_mainloop_arguments_3x(
    kernel_schedule: KernelScheduleType,
    element_A,
    element_B,
    alignment_A: int,
    alignment_B: int,
    use_sycl: bool = False) -> ctypes.Structure:
    """
    Returns the ctypes structure to be used for the 3.x kernel's mainloop parameters.

    :param kernel_schedule: type of kernel schedule to be used in the mainloop
    :type kernel_schedule: cutlass_library.KernelScheduleType
    :param element_A: data type of operand A
    :param element_B: data type of operand B
    :param alignment_A: alignment of operand A
    :type alignment_A: int
    :param alignment_B: alignment of operand B
    :type alignment_B: int

    :returns: ctypes structure to be used for the 3.x kernel's mainloop parameters
    :rtype: ctypes.Structure
    """
```
**EN:** Defines function `get_mainloop_arguments_3x` with parameters `kernel_schedule, element_A, element_B, alignment_A, alignment_B, use_sycl`. Purpose: Returns the ctypes structure to be used for the 3.x kernel's mainloop parameters.
**CN:** 定义函数 `get_mainloop_arguments_3x`，参数为 `kernel_schedule, element_A, element_B, alignment_A, alignment_B, use_sycl`。 其用途：Returns the ctypes structure to be used for the 3.x kernel's mainloop parameters.

#### Line 181 — Class `_MainloopArgumentsTma`

```python
    class _MainloopArgumentsTma(ctypes.Structure):
```
**EN:** Declares class `_MainloopArgumentsTma` deriving from `ctypes.Structure`.
**CN:** 声明类 `_MainloopArgumentsTma`，其基类为 `ctypes.Structure`。

##### Lines 182-188 — Assign `_fields_`

```python
        _fields_ = [
            ("ptr_A", ctypes.c_void_p),
            ("stride_A", StrideBatched_),
            ("ptr_B", ctypes.c_void_p),
            ("stride_B", StrideBatched_),
            ("mma_promotion_interval", ctypes.c_int)
        ]
```
**EN:** Assigns `_fields_` from `[('ptr_A', ctypes.c_void_p), ('stride_A', StrideBatched_), ('ptr_B', ctypes.c_void_p), ('stride_B', StrideBatched_), ('mma_promotion_interval', ctypes.c_int)]`, establishing state in class `_MainloopArgumentsTma`.
**CN:** 将 `_fields_` 赋值为 `[('ptr_A', ctypes.c_void_p), ('stride_A', StrideBatched_), ('ptr_B', ctypes.c_void_p), ('stride_B', StrideBatched_), ('mma_promotion_interval', ctypes.c_int)]`，用于在类 `_MainloopArgumentsTma` 中建立状态。

##### Lines 190-191 — Function `from_generic_mainloop_args`

```python
        @staticmethod
        def from_generic_mainloop_args(args: GenericMainloopArguments3x_):
```
**EN:** Defines function `from_generic_mainloop_args` with parameters `args`.
**CN:** 定义函数 `from_generic_mainloop_args`，参数为 `args`。

###### Lines 192-195 — Return

```python
            return _MainloopArgumentsTma(
                args.ptr_A, args.stride_A, args.ptr_B, args.stride_B,
                args.mma_promotion_interval
            )
```
**EN:** Returns `_MainloopArgumentsTma(args.ptr_A, args.stride_A, args.ptr_B, args.stride_B, args.mma_promotion_interval)` to the caller.
**CN:** 向调用方返回 `_MainloopArgumentsTma(args.ptr_A, args.stride_A, args.ptr_B, args.stride_B, args.mma_promotion_interval)`。

#### Line 197 — Class `_MainloopArgumentsMultistage`

```python
    class _MainloopArgumentsMultistage(ctypes.Structure):
```
**EN:** Declares class `_MainloopArgumentsMultistage` deriving from `ctypes.Structure`.
**CN:** 声明类 `_MainloopArgumentsMultistage`，其基类为 `ctypes.Structure`。

##### Lines 198-203 — Assign `_fields_`

```python
        _fields_ = [
            ("ptr_A", ctypes.c_void_p),
            ("stride_A", StrideBatched_),
            ("ptr_B", ctypes.c_void_p),
            ("stride_B", StrideBatched_),
        ]
```
**EN:** Assigns `_fields_` from `[('ptr_A', ctypes.c_void_p), ('stride_A', StrideBatched_), ('ptr_B', ctypes.c_void_p), ('stride_B', StrideBatched_)]`, establishing state in class `_MainloopArgumentsMultistage`.
**CN:** 将 `_fields_` 赋值为 `[('ptr_A', ctypes.c_void_p), ('stride_A', StrideBatched_), ('ptr_B', ctypes.c_void_p), ('stride_B', StrideBatched_)]`，用于在类 `_MainloopArgumentsMultistage` 中建立状态。

##### Lines 205-206 — Function `from_generic_mainloop_args`

```python
        @staticmethod
        def from_generic_mainloop_args(args: GenericMainloopArguments3x_):
```
**EN:** Defines function `from_generic_mainloop_args` with parameters `args`.
**CN:** 定义函数 `from_generic_mainloop_args`，参数为 `args`。

###### Lines 207-209 — Return

```python
            return _MainloopArgumentsMultistage(
                args.ptr_A, args.stride_A, args.ptr_B, args.stride_B,
            )
```
**EN:** Returns `_MainloopArgumentsMultistage(args.ptr_A, args.stride_A, args.ptr_B, args.stride_B)` to the caller.
**CN:** 向调用方返回 `_MainloopArgumentsMultistage(args.ptr_A, args.stride_A, args.ptr_B, args.stride_B)`。

#### Lines 211-219 — Conditional `use_sycl`

```python
    if use_sycl:
        # For SYCL, we don't have the additional 'mma_promotion_interval' arg.
        return _MainloopArgumentsMultistage
    else:
        # Currently all 3.x kernels (CpAsync and Tma) for Nvidia devices have
        # the same argument structure. Should that become not the case, this is
        # the place to return custom ctypes structures based on selected kernel
        # schedule.
        return _MainloopArgumentsTma
```
**EN:** Checks `use_sycl` and selects the matching branch in function `get_mainloop_arguments_3x`.
**CN:** 检查 `use_sycl`，并在函数 `get_mainloop_arguments_3x` 中选择匹配的分支。

### Line 222 — Function `get_gemm_arguments_3x`

```python
def get_gemm_arguments_3x(mainloop_arguments, epilogue_functor, scheduler_args, default_epilogue):
```
**EN:** Defines function `get_gemm_arguments_3x` with parameters `mainloop_arguments, epilogue_functor, scheduler_args, default_epilogue`.
**CN:** 定义函数 `get_gemm_arguments_3x`，参数为 `mainloop_arguments, epilogue_functor, scheduler_args, default_epilogue`。

#### Lines 223-226 — Conditional `not default_epilogue and hasattr(epilogue_functor, 'epilogue_type_evt')`

```python
    if not default_epilogue and hasattr(epilogue_functor, "epilogue_type_evt"):
        _EpilogueOutputOpParams = epilogue_functor.epilogue_type_evt
    else:
        _EpilogueOutputOpParams = epilogue_functor.epilogue_type
```
**EN:** Checks `not default_epilogue and hasattr(epilogue_functor, 'epilogue_type_evt')` and selects the matching branch in function `get_gemm_arguments_3x`.
**CN:** 检查 `not default_epilogue and hasattr(epilogue_functor, 'epilogue_type_evt')`，并在函数 `get_gemm_arguments_3x` 中选择匹配的分支。

#### Lines 228-248 — Conditional `hasattr(epilogue_functor, 'visitor')`

```python
    if hasattr(epilogue_functor, "visitor"):
        class _EpilogueArguments(ctypes.Structure):
            _fields_ = [
                ("epilogue", _EpilogueOutputOpParams),
                ("arg_C", epilogue_functor.arg_c_type),
                ("arg_D", epilogue_functor.arg_d_type)
            ]

            def __init__(self, output_op, ptr_c, stride_c, ptr_d, stride_d) -> None:
                self.epilogue = output_op
                self.arg_C = epilogue_functor.arg_c_type(ptr_c)
                self.arg_D = epilogue_functor.arg_d_type(ptr_d)
    else:
        class _EpilogueArguments(ctypes.Structure):
            _fields_ = [
                ("epilogue", _EpilogueOutputOpParams),
                ("ptr_C", ctypes.c_void_p),
                ("stride_C", StrideBatched_),
                ("ptr_D", ctypes.c_void_p),
                ("stride_D", StrideBatched_),
            ]
```
**EN:** Checks `hasattr(epilogue_functor, 'visitor')` and selects the matching branch in function `get_gemm_arguments_3x`.
**CN:** 检查 `hasattr(epilogue_functor, 'visitor')`，并在函数 `get_gemm_arguments_3x` 中选择匹配的分支。

#### Line 250 — Class `_HardwareInfo`

```python
    class _HardwareInfo(ctypes.Structure):
```
**EN:** Declares class `_HardwareInfo` deriving from `ctypes.Structure`.
**CN:** 声明类 `_HardwareInfo`，其基类为 `ctypes.Structure`。

##### Lines 251-257 — Assign `_fields_`

```python
        _fields_ = [
            ("device_id", ctypes.c_int),
            ("sm_count", ctypes.c_int),
            ("max_active_clusters", ctypes.c_int),
            ("cluster_shape", dim3_),
            ("cluster_shape_fallback", dim3_),
        ]
```
**EN:** Assigns `_fields_` from `[('device_id', ctypes.c_int), ('sm_count', ctypes.c_int), ('max_active_clusters', ctypes.c_int), ('cluster_shape', dim3_), ('cluster_shape_fallback', dim3_)]`, establishing state in class `_HardwareInfo`.
**CN:** 将 `_fields_` 赋值为 `[('device_id', ctypes.c_int), ('sm_count', ctypes.c_int), ('max_active_clusters', ctypes.c_int), ('cluster_shape', dim3_), ('cluster_shape_fallback', dim3_)]`，用于在类 `_HardwareInfo` 中建立状态。

#### Line 259 — Class `_GemmArguments`

```python
    class _GemmArguments(ctypes.Structure):
```
**EN:** Declares class `_GemmArguments` deriving from `ctypes.Structure`.
**CN:** 声明类 `_GemmArguments`，其基类为 `ctypes.Structure`。

##### Lines 260-267 — Assign `_fields_`

```python
        _fields_ = [
            ("mode", ctypes.c_int),
            ("problem_size", GemmCoordBatched_),
            ("mainloop", mainloop_arguments),
            ("epilogue", _EpilogueArguments),
            ("hw_info", _HardwareInfo),
            ("scheduler", type(scheduler_args)),
        ]
```
**EN:** Assigns `_fields_` from `[('mode', ctypes.c_int), ('problem_size', GemmCoordBatched_), ('mainloop', mainloop_arguments), ('epilogue', _EpilogueArguments), ('hw_info', _HardwareInfo),...`, establishing state in class `_GemmArguments`.
**CN:** 将 `_fields_` 赋值为 `[('mode', ctypes.c_int), ('problem_size', GemmCoordBatched_), ('mainloop', mainloop_arguments), ('epilogue', _EpilogueArguments), ('hw_info', _HardwareInfo),...`，用于在类 `_GemmArguments` 中建立状态。

#### Line 269 — Return

```python
    return _GemmArguments, _EpilogueArguments, _EpilogueOutputOpParams, _HardwareInfo
```
**EN:** Returns `(_GemmArguments, _EpilogueArguments, _EpilogueOutputOpParams, _HardwareInfo)` to the caller.
**CN:** 向调用方返回 `(_GemmArguments, _EpilogueArguments, _EpilogueOutputOpParams, _HardwareInfo)`。

### Line 272 — Function `get_gemm_arguments`

```python
def get_gemm_arguments(epilogue_functor):
```
**EN:** Defines function `get_gemm_arguments` with parameters `epilogue_functor`.
**CN:** 定义函数 `get_gemm_arguments`，参数为 `epilogue_functor`。

#### Line 273 — Assign `_EpilogueOutputOpParams`

```python
    _EpilogueOutputOpParams = epilogue_functor.epilogue_type
```
**EN:** Assigns `_EpilogueOutputOpParams` from `epilogue_functor.epilogue_type`, establishing state in function `get_gemm_arguments`.
**CN:** 将 `_EpilogueOutputOpParams` 赋值为 `epilogue_functor.epilogue_type`，用于在函数 `get_gemm_arguments` 中建立状态。

#### Line 275 — Class `_GemmArguments`

```python
    class _GemmArguments(ctypes.Structure):
```
**EN:** Declares class `_GemmArguments` deriving from `ctypes.Structure`.
**CN:** 声明类 `_GemmArguments`，其基类为 `ctypes.Structure`。

##### Lines 276-302 — Assign `_fields_`

```python
        _fields_ = [
            # Arguments from UniversalArgumentsBase
            ("mode", ctypes.c_int),
            ("problem_size", GemmCoord_),
            ("batch_count", ctypes.c_int),
            ("batch_stride_D", ctypes.c_longlong),
            # Remaining arguments
            ("epilogue", _EpilogueOutputOpParams),
            ("ptr_A", ctypes.c_void_p),
            ("ptr_B", ctypes.c_void_p),
            ("ptr_C", ctypes.c_void_p),
            ("ptr_D", ctypes.c_void_p),
            ("batch_stride_A", ctypes.c_longlong),
            ("batch_stride_B", ctypes.c_longlong),
            ("batch_stride_C", ctypes.c_longlong),
            ("stride_a", ctypes.c_longlong),
            ("stride_b", ctypes.c_longlong),
            ("stride_c", ctypes.c_longlong),
            ("stride_d", ctypes.c_longlong),
            ("lda", ctypes.c_longlong),
            ("ldb", ctypes.c_longlong),
            ("ldc", ctypes.c_longlong),
            ("ldd", ctypes.c_longlong),
            ("ptr_gather_A_indices", ctypes.c_void_p),
            ("ptr_gather_B_indices", ctypes.c_void_p),
            ("ptr_scatter_D_indices", ctypes.c_void_p)
        ]
```
**EN:** Assigns `_fields_` from `[('mode', ctypes.c_int), ('problem_size', GemmCoord_), ('batch_count', ctypes.c_int), ('batch_stride_D', ctypes.c_longlong), ('epilogue', _EpilogueOutputOpPa...`, establishing state in class `_GemmArguments`.
**CN:** 将 `_fields_` 赋值为 `[('mode', ctypes.c_int), ('problem_size', GemmCoord_), ('batch_count', ctypes.c_int), ('batch_stride_D', ctypes.c_longlong), ('epilogue', _EpilogueOutputOpPa...`，用于在类 `_GemmArguments` 中建立状态。

#### Line 304 — Return

```python
    return _GemmArguments, _EpilogueOutputOpParams
```
**EN:** Returns `(_GemmArguments, _EpilogueOutputOpParams)` to the caller.
**CN:** 向调用方返回 `(_GemmArguments, _EpilogueOutputOpParams)`。

### Line 307 — Function `get_gemm_arguments_streamk`

```python
def get_gemm_arguments_streamk(epilogue_functor):
```
**EN:** Defines function `get_gemm_arguments_streamk` with parameters `epilogue_functor`.
**CN:** 定义函数 `get_gemm_arguments_streamk`，参数为 `epilogue_functor`。

#### Line 308 — Assign `_EpilogueOutputOpParams`

```python
    _EpilogueOutputOpParams = epilogue_functor.epilogue_type
```
**EN:** Assigns `_EpilogueOutputOpParams` from `epilogue_functor.epilogue_type`, establishing state in function `get_gemm_arguments_streamk`.
**CN:** 将 `_EpilogueOutputOpParams` 赋值为 `epilogue_functor.epilogue_type`，用于在函数 `get_gemm_arguments_streamk` 中建立状态。

#### Line 310 — Class `_GemmArguments`

```python
    class _GemmArguments(ctypes.Structure):
```
**EN:** Declares class `_GemmArguments` deriving from `ctypes.Structure`.
**CN:** 声明类 `_GemmArguments`，其基类为 `ctypes.Structure`。

##### Lines 311-333 — Assign `_fields_`

```python
        _fields_ = [
            ("mode", ctypes.c_int),
            ("problem_size", GemmCoord_),
            ("batch_count", ctypes.c_int),
            ("epilogue", _EpilogueOutputOpParams),
            ("ptr_A", ctypes.c_void_p),
            ("ptr_B", ctypes.c_void_p),
            ("ptr_C", ctypes.c_void_p),
            ("ptr_D", ctypes.c_void_p),
            ("batch_stride_A", ctypes.c_longlong),
            ("batch_stride_B", ctypes.c_longlong),
            ("batch_stride_C", ctypes.c_longlong),
            ("batch_stride_D", ctypes.c_longlong),
            ("stride_a", ctypes.c_longlong),
            ("stride_b", ctypes.c_longlong),
            ("stride_c", ctypes.c_longlong),
            ("stride_d", ctypes.c_longlong),
            ("lda", ctypes.c_longlong),
            ("ldb", ctypes.c_longlong),
            ("ldc", ctypes.c_longlong),
            ("ldd", ctypes.c_longlong),
            ("avail_sms", ctypes.c_int)
        ]
```
**EN:** Assigns `_fields_` from `[('mode', ctypes.c_int), ('problem_size', GemmCoord_), ('batch_count', ctypes.c_int), ('epilogue', _EpilogueOutputOpParams), ('ptr_A', ctypes.c_void_p), ('pt...`, establishing state in class `_GemmArguments`.
**CN:** 将 `_fields_` 赋值为 `[('mode', ctypes.c_int), ('problem_size', GemmCoord_), ('batch_count', ctypes.c_int), ('epilogue', _EpilogueOutputOpParams), ('ptr_A', ctypes.c_void_p), ('pt...`，用于在类 `_GemmArguments` 中建立状态。

#### Line 335 — Return

```python
    return _GemmArguments, _EpilogueOutputOpParams
```
**EN:** Returns `(_GemmArguments, _EpilogueOutputOpParams)` to the caller.
**CN:** 向调用方返回 `(_GemmArguments, _EpilogueOutputOpParams)`。

### Lines 338-340 — Comment or spacing block

```python
###########################################################################################
# GEMM Grouped
###########################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Line 343 — Function `get_gemm_grouped_arguments`

```python
def get_gemm_grouped_arguments(epilogue_functor):
```
**EN:** Defines function `get_gemm_grouped_arguments` with parameters `epilogue_functor`.
**CN:** 定义函数 `get_gemm_grouped_arguments`，参数为 `epilogue_functor`。

#### Line 344 — Assign `_EpilogueOutputOpParams`

```python
    _EpilogueOutputOpParams = epilogue_functor.epilogue_type
```
**EN:** Assigns `_EpilogueOutputOpParams` from `epilogue_functor.epilogue_type`, establishing state in function `get_gemm_grouped_arguments`.
**CN:** 将 `_EpilogueOutputOpParams` 赋值为 `epilogue_functor.epilogue_type`，用于在函数 `get_gemm_grouped_arguments` 中建立状态。

#### Line 346 — Class `_GEMMGroupedArguments`

```python
    class _GEMMGroupedArguments(ctypes.Structure):
```
**EN:** Declares class `_GEMMGroupedArguments` deriving from `ctypes.Structure`.
**CN:** 声明类 `_GEMMGroupedArguments`，其基类为 `ctypes.Structure`。

##### Lines 347-361 — Assign `_fields_`

```python
        _fields_ = [
            ("problem_sizes", ctypes.c_void_p),
            ("problem_count", ctypes.c_int),
            ("threadblock_count", ctypes.c_int),
            ("output_op", _EpilogueOutputOpParams),
            ("ptr_A", ctypes.c_void_p),
            ("ptr_B", ctypes.c_void_p),
            ("ptr_C", ctypes.c_void_p),
            ("ptr_D", ctypes.c_void_p),
            ("lda", ctypes.c_void_p),
            ("ldb", ctypes.c_void_p),
            ("ldc", ctypes.c_void_p),
            ("ldd", ctypes.c_void_p),
            ("host_problem_sizes", ctypes.c_void_p)
        ]
```
**EN:** Assigns `_fields_` from `[('problem_sizes', ctypes.c_void_p), ('problem_count', ctypes.c_int), ('threadblock_count', ctypes.c_int), ('output_op', _EpilogueOutputOpParams), ('ptr_A', ...`, establishing state in class `_GEMMGroupedArguments`.
**CN:** 将 `_fields_` 赋值为 `[('problem_sizes', ctypes.c_void_p), ('problem_count', ctypes.c_int), ('threadblock_count', ctypes.c_int), ('output_op', _EpilogueOutputOpParams), ('ptr_A', ...`，用于在类 `_GEMMGroupedArguments` 中建立状态。

#### Line 363 — Return

```python
    return _GEMMGroupedArguments, _EpilogueOutputOpParams
```
**EN:** Returns `(_GEMMGroupedArguments, _EpilogueOutputOpParams)` to the caller.
**CN:** 向调用方返回 `(_GEMMGroupedArguments, _EpilogueOutputOpParams)`。

### Lines 366-368 — Comment or spacing block

```python
############################################################################################
# Convolution2D
############################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Line 371 — Class `Conv2DProblemSize_`

```python
class Conv2DProblemSize_(ctypes.Structure):
```
**EN:** Declares class `Conv2DProblemSize_` deriving from `ctypes.Structure`.
**CN:** 声明类 `Conv2DProblemSize_`，其基类为 `ctypes.Structure`。

#### Lines 372-391 — Assign `_fields_`

```python
    _fields_ = [
        ("N", ctypes.c_int),
        ("H", ctypes.c_int),
        ("W", ctypes.c_int),
        ("C", ctypes.c_int),
        ("P", ctypes.c_int),
        ("Q", ctypes.c_int),
        ("K", ctypes.c_int),
        ("R", ctypes.c_int),
        ("S", ctypes.c_int),
        ("pad_h", ctypes.c_int),
        ("pad_w", ctypes.c_int),
        ("stride_h", ctypes.c_int),
        ("stride_w", ctypes.c_int),
        ("dilation_h", ctypes.c_int),
        ("dilation_w", ctypes.c_int),
        ("mode", ctypes.c_int),  # kCrossCorrelation: 0, kConvolution: 1
        ("split_k_slices", ctypes.c_int),
        ("groups", ctypes.c_int)
    ]
```
**EN:** Assigns `_fields_` from `[('N', ctypes.c_int), ('H', ctypes.c_int), ('W', ctypes.c_int), ('C', ctypes.c_int), ('P', ctypes.c_int), ('Q', ctypes.c_int), ('K', ctypes.c_int), ('R', cty...`, establishing state in class `Conv2DProblemSize_`.
**CN:** 将 `_fields_` 赋值为 `[('N', ctypes.c_int), ('H', ctypes.c_int), ('W', ctypes.c_int), ('C', ctypes.c_int), ('P', ctypes.c_int), ('Q', ctypes.c_int), ('K', ctypes.c_int), ('R', cty...`，用于在类 `Conv2DProblemSize_` 中建立状态。

#### Line 393 — Function `__init__`

```python
    def __init__(self, problem_size) -> None:
```
**EN:** Defines function `__init__` with parameters `self, problem_size`.
**CN:** 定义函数 `__init__`，参数为 `self, problem_size`。

##### Lines 394-395 — Loop over `self._fields_`

```python
        for field_name, _ in self._fields_:
            setattr(self, field_name, getattr(problem_size, field_name))
```
**EN:** Iterates `(field_name, _)` over `self._fields_` to repeat a processing step.
**CN:** 让 `(field_name, _)` 遍历 `self._fields_`，从而重复执行处理步骤。

### Line 398 — Class `Layout4D`

```python
class Layout4D(ctypes.Structure):
```
**EN:** Declares class `Layout4D` deriving from `ctypes.Structure`.
**CN:** 声明类 `Layout4D`，其基类为 `ctypes.Structure`。

#### Line 399 — Assign `_fields_`

```python
    _fields_ = [("stride", ctypes.c_int * 3)]
```
**EN:** Assigns `_fields_` from `[('stride', ctypes.c_int * 3)]`, establishing state in class `Layout4D`.
**CN:** 将 `_fields_` 赋值为 `[('stride', ctypes.c_int * 3)]`，用于在类 `Layout4D` 中建立状态。

#### Line 401 — Function `__init__`

```python
    def __init__(self, tensor_ref):
```
**EN:** Defines function `__init__` with parameters `self, tensor_ref`.
**CN:** 定义函数 `__init__`，参数为 `self, tensor_ref`。

##### Line 402 — Assign `stride`

```python
        stride = tensor_ref.stride()
```
**EN:** Assigns `stride` from `tensor_ref.stride()`, establishing state in function `__init__`.
**CN:** 将 `stride` 赋值为 `tensor_ref.stride()`，用于在函数 `__init__` 中建立状态。

##### Line 403 — Call `setattr`

```python
        setattr(self, "stride", (stride.at(0), stride.at(1), stride.at(2)))
```
**EN:** Calls `setattr` for side effects or initialization work in function `__init__`.
**CN:** 调用 `setattr` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

### Line 406 — Class `TensorRef_`

```python
class TensorRef_(ctypes.Structure):
```
**EN:** Declares class `TensorRef_` deriving from `ctypes.Structure`.
**CN:** 声明类 `TensorRef_`，其基类为 `ctypes.Structure`。

#### Lines 407-410 — Assign `_fields_`

```python
    _fields_ = [
        ("ptr", ctypes.c_void_p),
        ("layout", Layout4D)
    ]
```
**EN:** Assigns `_fields_` from `[('ptr', ctypes.c_void_p), ('layout', Layout4D)]`, establishing state in class `TensorRef_`.
**CN:** 将 `_fields_` 赋值为 `[('ptr', ctypes.c_void_p), ('layout', Layout4D)]`，用于在类 `TensorRef_` 中建立状态。

#### Line 412 — Function `__init__`

```python
    def __init__(self, tensor_ref):
```
**EN:** Defines function `__init__` with parameters `self, tensor_ref`.
**CN:** 定义函数 `__init__`，参数为 `self, tensor_ref`。

##### Line 413 — Call `setattr`

```python
        setattr(self, "ptr", tensor_ref.data())
```
**EN:** Calls `setattr` for side effects or initialization work in function `__init__`.
**CN:** 调用 `setattr` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 414 — Call `setattr`

```python
        setattr(self, "layout", Layout4D(tensor_ref.layout()))
```
**EN:** Calls `setattr` for side effects or initialization work in function `__init__`.
**CN:** 调用 `setattr` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

### Line 417 — Class `TensorRef2D_`

```python
class TensorRef2D_(ctypes.Structure):
```
**EN:** Declares class `TensorRef2D_` deriving from `ctypes.Structure`.
**CN:** 声明类 `TensorRef2D_`，其基类为 `ctypes.Structure`。

#### Lines 418-421 — Assign `_fields_`

```python
    _fields_ = [
        ("ptr", ctypes.c_void_p),
        ("stride", ctypes.c_int)
    ]
```
**EN:** Assigns `_fields_` from `[('ptr', ctypes.c_void_p), ('stride', ctypes.c_int)]`, establishing state in class `TensorRef2D_`.
**CN:** 将 `_fields_` 赋值为 `[('ptr', ctypes.c_void_p), ('stride', ctypes.c_int)]`，用于在类 `TensorRef2D_` 中建立状态。

### Line 424 — Function `get_conv2d_arguments`

```python
def get_conv2d_arguments(epilogue_functor):
```
**EN:** Defines function `get_conv2d_arguments` with parameters `epilogue_functor`.
**CN:** 定义函数 `get_conv2d_arguments`，参数为 `epilogue_functor`。

#### Line 425 — Assign `_EpilogueOutputOpParams`

```python
    _EpilogueOutputOpParams = epilogue_functor.epilogue_type
```
**EN:** Assigns `_EpilogueOutputOpParams` from `epilogue_functor.epilogue_type`, establishing state in function `get_conv2d_arguments`.
**CN:** 将 `_EpilogueOutputOpParams` 赋值为 `epilogue_functor.epilogue_type`，用于在函数 `get_conv2d_arguments` 中建立状态。

#### Line 427 — Class `_Conv2dArguments`

```python
    class _Conv2dArguments(ctypes.Structure):
```
**EN:** Declares class `_Conv2dArguments` deriving from `ctypes.Structure`.
**CN:** 声明类 `_Conv2dArguments`，其基类为 `ctypes.Structure`。

##### Lines 428-438 — Assign `_fields_`

```python
        _fields_ = [
            ("conv_kind", ctypes.c_int),
            ("problem_size", Conv2DProblemSize_),
            ("ptr_A", ctypes.c_void_p),
            ("ptr_B", ctypes.c_void_p),
            ("ptr_C", ctypes.c_void_p),
            ("ptr_D", ctypes.c_void_p),
            ("tensor_C_numel", ctypes.c_int),
            ("output_op", _EpilogueOutputOpParams),
            ("split_k_mode", ctypes.c_int)
        ]
```
**EN:** Assigns `_fields_` from `[('conv_kind', ctypes.c_int), ('problem_size', Conv2DProblemSize_), ('ptr_A', ctypes.c_void_p), ('ptr_B', ctypes.c_void_p), ('ptr_C', ctypes.c_void_p), ('ptr...`, establishing state in class `_Conv2dArguments`.
**CN:** 将 `_fields_` 赋值为 `[('conv_kind', ctypes.c_int), ('problem_size', Conv2DProblemSize_), ('ptr_A', ctypes.c_void_p), ('ptr_B', ctypes.c_void_p), ('ptr_C', ctypes.c_void_p), ('ptr...`，用于在类 `_Conv2dArguments` 中建立状态。

#### Line 440 — Return

```python
    return _Conv2dArguments, _EpilogueOutputOpParams
```
**EN:** Returns `(_Conv2dArguments, _EpilogueOutputOpParams)` to the caller.
**CN:** 向调用方返回 `(_Conv2dArguments, _EpilogueOutputOpParams)`。

### Lines 443-445 — Comment or spacing block

```python
############################################################################################
# Reduction
############################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Line 448 — Function `get_reduction_params`

```python
def get_reduction_params(epilogue_functor):
```
**EN:** Defines function `get_reduction_params` with parameters `epilogue_functor`.
**CN:** 定义函数 `get_reduction_params`，参数为 `epilogue_functor`。

#### Line 449 — Assign `_EpilogueOutputParams`

```python
    _EpilogueOutputParams = epilogue_functor.epilogue_type
```
**EN:** Assigns `_EpilogueOutputParams` from `epilogue_functor.epilogue_type`, establishing state in function `get_reduction_params`.
**CN:** 将 `_EpilogueOutputParams` 赋值为 `epilogue_functor.epilogue_type`，用于在函数 `get_reduction_params` 中建立状态。

#### Line 451 — Class `_ReductionParams`

```python
    class _ReductionParams(ctypes.Structure):
```
**EN:** Declares class `_ReductionParams` deriving from `ctypes.Structure`.
**CN:** 声明类 `_ReductionParams`，其基类为 `ctypes.Structure`。

##### Lines 452-460 — Assign `_fields_`

```python
        _fields_ = [
            ("problem_size", MatrixCoord_),
            ("partitions", ctypes.c_int),
            ("partition_stride", ctypes.c_longlong),
            ("workspace", TensorRef2D_),
            ("destination", TensorRef2D_),
            ("source", TensorRef2D_),
            ("output_op", _EpilogueOutputParams),
        ]
```
**EN:** Assigns `_fields_` from `[('problem_size', MatrixCoord_), ('partitions', ctypes.c_int), ('partition_stride', ctypes.c_longlong), ('workspace', TensorRef2D_), ('destination', TensorRe...`, establishing state in class `_ReductionParams`.
**CN:** 将 `_fields_` 赋值为 `[('problem_size', MatrixCoord_), ('partitions', ctypes.c_int), ('partition_stride', ctypes.c_longlong), ('workspace', TensorRef2D_), ('destination', TensorRe...`，用于在类 `_ReductionParams` 中建立状态。

#### Line 462 — Return

```python
    return _ReductionParams, _EpilogueOutputParams
```
**EN:** Returns `(_ReductionParams, _EpilogueOutputParams)` to the caller.
**CN:** 向调用方返回 `(_ReductionParams, _EpilogueOutputParams)`。

### Lines 465-467 — Comment or spacing block

```python
###########################################################################################
# Epilogue Visitor Type Factory
###########################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Line 469 — Class `Empty`

```python
class Empty(ctypes.Structure):
```
**EN:** Declares class `Empty` deriving from `ctypes.Structure`.
**CN:** 声明类 `Empty`，其基类为 `ctypes.Structure`。

#### Line 470 — Assign `_fields_`

```python
    _fields_ = []
```
**EN:** Assigns `_fields_` from `[]`, establishing state in class `Empty`.
**CN:** 将 `_fields_` 赋值为 `[]`，用于在类 `Empty` 中建立状态。

#### Line 472 — Function `__init__`

```python
    def __init__(self, *arg) -> None:
```
**EN:** Defines function `__init__` with parameters `self, *arg`.
**CN:** 定义函数 `__init__`，参数为 `self, *arg`。

##### Line 473 — Pass

```python
        pass
```
**EN:** Leaves a placeholder branch with no runtime action.
**CN:** 保留一个不执行任何运行期动作的占位分支。

### Line 475 — Class `EmptyByte`

```python
class EmptyByte(ctypes.Structure):
```
**EN:** Declares class `EmptyByte` deriving from `ctypes.Structure`.
**CN:** 声明类 `EmptyByte`，其基类为 `ctypes.Structure`。

#### Lines 476-478 — Assign `_fields_`

```python
    _fields_ = [
        ("byte", ctypes.c_byte)
    ]
```
**EN:** Assigns `_fields_` from `[('byte', ctypes.c_byte)]`, establishing state in class `EmptyByte`.
**CN:** 将 `_fields_` 赋值为 `[('byte', ctypes.c_byte)]`，用于在类 `EmptyByte` 中建立状态。

#### Line 480 — Function `__init__`

```python
    def __init__(self, *arg) -> None:
```
**EN:** Defines function `__init__` with parameters `self, *arg`.
**CN:** 定义函数 `__init__`，参数为 `self, *arg`。

##### Line 481 — Pass

```python
        pass
```
**EN:** Leaves a placeholder branch with no runtime action.
**CN:** 保留一个不执行任何运行期动作的占位分支。

### Line 483 — Class `EBO`

```python
class EBO:
```
**EN:** Declares class `EBO` deriving from `object`.
**CN:** 声明类 `EBO`，其基类为 `object`。

#### Line 484 — Function `__init__`

```python
    def __init__(self, index: int, type) -> None:
```
**EN:** Defines function `__init__` with parameters `self, index, type`.
**CN:** 定义函数 `__init__`，参数为 `self, index, type`。

##### Line 485 — Assign `self.index`

```python
        self.index = index
```
**EN:** Assigns `self.index` from `index`, establishing state in function `__init__`.
**CN:** 将 `self.index` 赋值为 `index`，用于在函数 `__init__` 中建立状态。

##### Line 486 — Assign `self.type`

```python
        self.type = type
```
**EN:** Assigns `self.type` from `type`, establishing state in function `__init__`.
**CN:** 将 `self.type` 赋值为 `type`，用于在函数 `__init__` 中建立状态。

#### Line 488 — Function `__eq__`

```python
    def __eq__(self, other) -> bool:
```
**EN:** Defines function `__eq__` with parameters `self, other`.
**CN:** 定义函数 `__eq__`，参数为 `self, other`。

##### Lines 489-490 — Conditional `isinstance(other, EBO)`

```python
        if isinstance(other, EBO):
            return self.index == other.index and self.type == other.type
```
**EN:** Checks `isinstance(other, EBO)` and selects the matching branch in function `__eq__`.
**CN:** 检查 `isinstance(other, EBO)`，并在函数 `__eq__` 中选择匹配的分支。

##### Line 491 — Return

```python
        return False
```
**EN:** Returns `False` to the caller.
**CN:** 向调用方返回 `False`。

#### Line 493 — Function `__hash__`

```python
    def __hash__(self) -> int:
```
**EN:** Defines function `__hash__` with parameters `self`.
**CN:** 定义函数 `__hash__`，参数为 `self`。

##### Line 494 — Return

```python
        return hash((self.index, self.type))
```
**EN:** Returns `hash((self.index, self.type))` to the caller.
**CN:** 向调用方返回 `hash((self.index, self.type))`。

#### Line 496 — Function `__ne__`

```python
    def __ne__(self, other):
```
**EN:** Defines function `__ne__` with parameters `self, other`.
**CN:** 定义函数 `__ne__`，参数为 `self, other`。

##### Line 497 — Return

```python
        return not self.__eq__(other)
```
**EN:** Returns `not self.__eq__(other)` to the caller.
**CN:** 向调用方返回 `not self.__eq__(other)`。

#### Line 499 — Function `__str__`

```python
    def __str__(self) -> str:
```
**EN:** Defines function `__str__` with parameters `self`.
**CN:** 定义函数 `__str__`，参数为 `self`。

##### Line 500 — Return

```python
        return f"<{self.index}, {self.type}>"
```
**EN:** Returns `f'<{self.index}, {self.type}>'` to the caller.
**CN:** 向调用方返回 `f'<{self.index}, {self.type}>'`。

### Lines 503-515 — Function `tuple_factory_`

```python
def tuple_factory_(input_tuple, dtype, constants=[0,1]):
    """
    The factory function generating cute::Tuple with input tuple
    :param input_tuple: the input tuple
    :type input_tuple: tuple
    :param dtype: the data type for non-constant values
    :type dtype: str, "int32_t", "int", "int64_t"
    :param constant: the values that will be treated as constants
    :type constant: list[int]

    :return: ctype structure representing the cute::Tuple
    :return: the empty base classes of the tuple
    """
```
**EN:** Defines function `tuple_factory_` with parameters `input_tuple, dtype, constants`. Purpose: The factory function generating cute::Tuple with input tuple
**CN:** 定义函数 `tuple_factory_`，参数为 `input_tuple, dtype, constants`。 其用途：The factory function generating cute::Tuple with input tuple

#### Line 517 — Comment or spacing block

```python
    # The empty base classes of the current tuple
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 518 — Assign `empty_bases`

```python
    empty_bases = []
```
**EN:** Assigns `empty_bases` from `[]`, establishing state in function `tuple_factory_`.
**CN:** 将 `empty_bases` 赋值为 `[]`，用于在函数 `tuple_factory_` 中建立状态。

#### Line 519 — Comment or spacing block

```python
    # The first non empty base class
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 520 — Assign `first_non_empty_base`

```python
    first_non_empty_base = None
```
**EN:** Assigns `first_non_empty_base` from `None`, establishing state in function `tuple_factory_`.
**CN:** 将 `first_non_empty_base` 赋值为 `None`，用于在函数 `tuple_factory_` 中建立状态。

#### Line 521 — Comment or spacing block

```python
    # The ctype fields of the current tuple
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 522 — Assign `ctype_fields`

```python
    ctype_fields = []
```
**EN:** Assigns `ctype_fields` from `[]`, establishing state in function `tuple_factory_`.
**CN:** 将 `ctype_fields` 赋值为 `[]`，用于在函数 `tuple_factory_` 中建立状态。

#### Lines 524-542 — Loop over `enumerate(input_tuple)`

```python
    for idx, entry in enumerate(input_tuple):
        # For nested tuples
        if isinstance(entry, tuple):
            sub_tuple_ctype, sub_empty_bases = tuple_factory_(entry, dtype, constants)
            if ctypes.sizeof(sub_tuple_ctype) == 0:
                # The empty tuple base class is also an empty EBO
                empty_bases.append(EBO(idx, entry))
            else:
                if first_non_empty_base is None:
                    first_non_empty_base = sub_empty_bases
            ctype_fields.append((f"entry_{idx}", sub_tuple_ctype))
        else:
            if entry in constants:
                empty_bases.append(EBO(idx, entry))
                ctype_fields.append((f"entry_{idx}", Empty))
            else:
                ctype_fields.append((f"entry_{idx}", dtype))
                if first_non_empty_base is None:
                    first_non_empty_base = []
```
**EN:** Iterates `(idx, entry)` over `enumerate(input_tuple)` to repeat a processing step.
**CN:** 让 `(idx, entry)` 遍历 `enumerate(input_tuple)`，从而重复执行处理步骤。

#### Line 544 — Comment or spacing block

```python
    # Create the ctype tuple
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 545 — Class `TupleType`

```python
    class TupleType(ctypes.Structure):
```
**EN:** Declares class `TupleType` deriving from `ctypes.Structure`.
**CN:** 声明类 `TupleType`，其基类为 `ctypes.Structure`。

##### Line 546 — Assign `_fields_`

```python
        _fields_ = ctype_fields
```
**EN:** Assigns `_fields_` from `ctype_fields`, establishing state in class `TupleType`.
**CN:** 将 `_fields_` 赋值为 `ctype_fields`，用于在类 `TupleType` 中建立状态。

##### Line 548 — Function `__init__`

```python
        def __init__(self, args) -> None:
```
**EN:** Defines function `__init__` with parameters `self, args`.
**CN:** 定义函数 `__init__`，参数为 `self, args`。

###### Line 549 — Assign `fields`

```python
            fields = self._fields_
```
**EN:** Assigns `fields` from `self._fields_`, establishing state in function `__init__`.
**CN:** 将 `fields` 赋值为 `self._fields_`，用于在函数 `__init__` 中建立状态。

###### Line 551 — Assertion

```python
            assert len(fields) == len(args)
```
**EN:** Asserts `len(fields) == len(args)` to enforce an expected condition.
**CN:** 断言 `len(fields) == len(args)`，用于保证预期条件成立。

###### Lines 552-555 — Loop over `zip(fields, args)`

```python
            for field, arg in zip(fields, args):
                name = field[0]
                field_type = field[1]
                setattr(self, name, field_type(arg))
```
**EN:** Iterates `(field, arg)` over `zip(fields, args)` to repeat a processing step.
**CN:** 让 `(field, arg)` 遍历 `zip(fields, args)`，从而重复执行处理步骤。

#### Line 557 — Return

```python
    return TupleType, empty_bases
```
**EN:** Returns `(TupleType, empty_bases)` to the caller.
**CN:** 向调用方返回 `(TupleType, empty_bases)`。

### Lines 559-571 — Function `tuple_factory`

```python
def tuple_factory(input_tuple, dtype: str, constants=[0,1]):
    """
    The factory function generating cute::Tuple with input tuple
    :param input_tuple: the input tuple
    :type input_tuple: tuple
    :param dtype: the data type for non-constant values
    :type dtype: str, "int32_t", "int", "int64_t"
    :param constant: the values that will be treated as constants
    :type constant: list[int]

    :return: ctype structure representing the cute::Tuple
    :return: the empty base classes of the tuple
    """
```
**EN:** Defines function `tuple_factory` with parameters `input_tuple, dtype, constants`. Purpose: The factory function generating cute::Tuple with input tuple
**CN:** 定义函数 `tuple_factory`，参数为 `input_tuple, dtype, constants`。 其用途：The factory function generating cute::Tuple with input tuple

#### Line 572 — Comment or spacing block

```python
    # Step 1: convert the dtype
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 573-578 — Conditional `dtype == 'int64_t'`

```python
    if dtype == "int64_t":
        dtype = ctypes.c_longlong
    elif dtype in ["int", "int32_t"]:
        dtype = ctypes.c_int32
    else:
        raise NotImplementedError(f"Type {dtype} is not supported")
```
**EN:** Checks `dtype == 'int64_t'` and selects the matching branch in function `tuple_factory`.
**CN:** 检查 `dtype == 'int64_t'`，并在函数 `tuple_factory` 中选择匹配的分支。

#### Line 580 — Assign `tuple_type, _`

```python
    tuple_type, _ = tuple_factory_(input_tuple, dtype, constants)
```
**EN:** Assigns `tuple_type, _` from `tuple_factory_(input_tuple, dtype, constants)`, establishing state in function `tuple_factory`.
**CN:** 将 `tuple_type, _` 赋值为 `tuple_factory_(input_tuple, dtype, constants)`，用于在函数 `tuple_factory` 中建立状态。

#### Lines 582-583 — Conditional `ctypes.sizeof(tuple_type) == 0`

```python
    if ctypes.sizeof(tuple_type) == 0:
        return EmptyByte
```
**EN:** Checks `ctypes.sizeof(tuple_type) == 0` and selects the matching branch in function `tuple_factory`.
**CN:** 检查 `ctypes.sizeof(tuple_type) == 0`，并在函数 `tuple_factory` 中选择匹配的分支。

#### Line 584 — Return

```python
    return tuple_type
```
**EN:** Returns `tuple_type` to the caller.
**CN:** 向调用方返回 `tuple_type`。

### Lines 587-595 — Function `visitor_factory`

```python
def visitor_factory(node_types, node_names):
    """
    Creates the argument type of epilogue visitor type

    :param node_types: list of argument types under ctypes
    :param node_names: list of argument names under str

    :return: tuple type in ctypes.Structure
    """
```
**EN:** Defines function `visitor_factory` with parameters `node_types, node_names`. Purpose: Creates the argument type of epilogue visitor type
**CN:** 定义函数 `visitor_factory`，参数为 `node_types, node_names`。 其用途：Creates the argument type of epilogue visitor type

#### Line 596 — Assign `ctypes_field`

```python
    ctypes_field = []
```
**EN:** Assigns `ctypes_field` from `[]`, establishing state in function `visitor_factory`.
**CN:** 将 `ctypes_field` 赋值为 `[]`，用于在函数 `visitor_factory` 中建立状态。

#### Lines 597-599 — Comment or spacing block

```python
    # Struct is used when number of nodes < 4
    # Because the Sm90VisitorImplBase has specification up to 4 nodes
    # in `include/cutlass/epilogue/fusion/sm90_visitor_tma_warpspecialized.hpp`
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 600-629 — Conditional `len(node_types) <= 4`

```python
    if len(node_types) <= 4:
        for idx, node_type in enumerate(node_types):
            if ctypes.sizeof(node_type) == 0:
                # Special case for empty struct
                # 1 byte placeholder is used for correct alignment
                ctypes_field.append((node_names[idx], ctypes.c_byte))
            else:
                ctypes_field.append((node_names[idx], node_type))

        class VisitorType(ctypes.Structure):
            _fields_ = ctypes_field

            def __init__(self, kwargs) -> None:
                for field in self._fields_:
                    fname, ftype = field
                    if ftype != ctypes.c_byte:
                        setattr(self, fname, ftype(kwargs))

    # For cases with more than 4 nodes, tuple is used
    else:
        for idx, node_type in enumerate(node_types):
            ctypes_field.append((node_names[idx], node_type))

        class VisitorType(ctypes.Structure):
            _fields_ = ctypes_field

            def __init__(self, kwargs) -> None:
                for field in self._fields_:
                    fname, ftype = field
                    setattr(self, fname, ftype(kwargs))
```
**EN:** Checks `len(node_types) <= 4` and selects the matching branch in function `visitor_factory`.
**CN:** 检查 `len(node_types) <= 4`，并在函数 `visitor_factory` 中选择匹配的分支。

#### Line 631 — Return

```python
    return VisitorType
```
**EN:** Returns `VisitorType` to the caller.
**CN:** 向调用方返回 `VisitorType`。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `GemmCoord_`, `GemmCoordBatched_`, `MatrixCoord_`, `dim3_`, `StrideBatched_`, `GenericMainloopArguments3x_`, `_PersistentTileSchedulerArguments`, `_PersistentTileSchedulerStreamKArguments`, `Conv2DProblemSize_`, `Layout4D`, `TensorRef_`, `TensorRef2D_`, `Empty`, `EmptyByte`, `EBO`.
- **CN:** 顶层类：`GemmCoord_`, `GemmCoordBatched_`, `MatrixCoord_`, `dim3_`, `StrideBatched_`, `GenericMainloopArguments3x_`, `_PersistentTileSchedulerArguments`, `_PersistentTileSchedulerStreamKArguments`, `Conv2DProblemSize_`, `Layout4D`, `TensorRef_`, `TensorRef2D_`, `Empty`, `EmptyByte`, `EBO`。
- **EN:** Top-level functions: `get_tile_scheduler_arguments_3x`, `get_mainloop_arguments_3x`, `get_gemm_arguments_3x`, `get_gemm_arguments`, `get_gemm_arguments_streamk`, `get_gemm_grouped_arguments`, `get_conv2d_arguments`, `get_reduction_params`, `tuple_factory_`, `tuple_factory`, `visitor_factory`.
- **CN:** 顶层函数：`get_tile_scheduler_arguments_3x`, `get_mainloop_arguments_3x`, `get_gemm_arguments_3x`, `get_gemm_arguments`, `get_gemm_arguments_streamk`, `get_gemm_grouped_arguments`, `get_conv2d_arguments`, `get_reduction_params`, `tuple_factory_`, `tuple_factory`, `visitor_factory`。
- **EN:** This module organizes backend utilities, data structures, or runtime helpers used by cutlass_cppgen.
- **CN:** 该模块组织 cutlass_cppgen 所需的后端工具、数据结构或运行时辅助逻辑。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.library`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** `ctypes`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
