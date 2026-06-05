# conv2d_operation.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/conv2d_operation.py`
- **EN:** Defines classes `Conv2dArguments`, `Conv2dRT`, `Conv2dOperation`, `EmitConv2dInstance` for the cutlass_cppgen package.
- **CN:** 为 cutlass_cppgen 包定义类 `Conv2dArguments`, `Conv2dRT`, `Conv2dOperation`, `EmitConv2dInstance`。

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

### Line 34 — Import `ctypes`

```python
import ctypes
```
**EN:** Imports `ctypes` so later code can use these APIs at module scope.
**CN:** 导入 `ctypes`，供后续代码在模块级使用这些 API。

### Line 35 — From `typing` import

```python
from typing import Union
```
**EN:** Imports `Union` from `typing` to reuse shared definitions at module scope.
**CN:** 从 `typing` 导入 `Union`，以便后续代码在模块级复用共享定义。

### Line 37 — From `cutlass_cppgen.utils.lazy_import` import

```python
from cutlass_cppgen.utils.lazy_import import lazy_import
```
**EN:** Imports `lazy_import` from `cutlass_cppgen.utils.lazy_import` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.utils.lazy_import` 导入 `lazy_import`，以便后续代码在模块级复用共享定义。

### Line 38 — Assign `cuda`

```python
cuda = lazy_import("cuda.cuda")
```
**EN:** Assigns `cuda` from `lazy_import('cuda.cuda')`, establishing state at module scope.
**CN:** 将 `cuda` 赋值为 `lazy_import('cuda.cuda')`，用于在模块级建立状态。

### Line 39 — From `cutlass_library` import

```python
from cutlass_library import SubstituteTemplate
```
**EN:** Imports `SubstituteTemplate` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `SubstituteTemplate`，以便后续代码在模块级复用共享定义。

### Line 40 — Import `numpy as np`

```python
import numpy as np
```
**EN:** Imports `numpy as np` so later code can use these APIs at module scope.
**CN:** 导入 `numpy as np`，供后续代码在模块级使用这些 API。

### Lines 42-66 — From `cutlass_library` import

```python
from cutlass_library import (
    ConvKindNames,
    ConvKindTag,
    DataTypeNames,
    DataTypeSize,
    DataTypeTag,
    IteratorAlgorithmNames,
    IteratorAlgorithmTag,
    LayoutTag,
    LayoutType,
    MathOperation,
    MathOperationTag,
    OpcodeClass,
    OpcodeClassNames,
    OpcodeClassTag,
    OperationKind,
    ShortDataTypeNames,
    ShortLayoutTypeNames,
    SplitKMode,
    StrideSupport,
    StrideSupportTag,
    SwizzlingFunctor,
    SwizzlingFunctorTag,
    get_complex_from_real,
)
```
**EN:** Imports `ConvKindNames, ConvKindTag, DataTypeNames, DataTypeSize, DataTypeTag, IteratorAlgorithmNames, IteratorAlgorithmTag, LayoutTag, LayoutType, MathOperation, MathOperationTag, OpcodeClass, OpcodeClassNames, OpcodeClassTag, OperationKind, ShortDataTypeNames, ShortLayoutTypeNames, SplitKMode, StrideSupport, StrideSupportTag, SwizzlingFunctor, SwizzlingFunctorTag, get_complex_from_real` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `ConvKindNames, ConvKindTag, DataTypeNames, DataTypeSize, DataTypeTag, IteratorAlgorithmNames, IteratorAlgorithmTag, LayoutTag, LayoutType, MathOperation, MathOperationTag, OpcodeClass, OpcodeClassNames, OpcodeClassTag, OperationKind, ShortDataTypeNames, ShortLayoutTypeNames, SplitKMode, StrideSupport, StrideSupportTag, SwizzlingFunctor, SwizzlingFunctorTag, get_complex_from_real`，以便后续代码在模块级复用共享定义。

### Line 68 — From `cutlass_cppgen.backend.arguments` import

```python
from cutlass_cppgen.backend.arguments import ArgumentBase
```
**EN:** Imports `ArgumentBase` from `cutlass_cppgen.backend.arguments` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.arguments` 导入 `ArgumentBase`，以便后续代码在模块级复用共享定义。

### Line 69 — From `cutlass_cppgen.backend.c_types` import

```python
from cutlass_cppgen.backend.c_types import dim3_, get_conv2d_arguments
```
**EN:** Imports `dim3_, get_conv2d_arguments` from `cutlass_cppgen.backend.c_types` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.c_types` 导入 `dim3_, get_conv2d_arguments`，以便后续代码在模块级复用共享定义。

### Lines 70-74 — From `cutlass_cppgen.backend.library` import

```python
from cutlass_cppgen.backend.library import (
    EmissionType,
    TensorDescription,
    TileDescription,
)
```
**EN:** Imports `EmissionType, TensorDescription, TileDescription` from `cutlass_cppgen.backend.library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.library` 导入 `EmissionType, TensorDescription, TileDescription`，以便后续代码在模块级复用共享定义。

### Line 75 — From `cutlass_cppgen.backend.memory_manager` import

```python
from cutlass_cppgen.backend.memory_manager import device_mem_alloc
```
**EN:** Imports `device_mem_alloc` from `cutlass_cppgen.backend.memory_manager` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.memory_manager` 导入 `device_mem_alloc`，以便后续代码在模块级复用共享定义。

### Line 76 — From `cutlass_cppgen.backend.operation` import

```python
from cutlass_cppgen.backend.operation import ExecutableOperation, LaunchConfiguration
```
**EN:** Imports `ExecutableOperation, LaunchConfiguration` from `cutlass_cppgen.backend.operation` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.operation` 导入 `ExecutableOperation, LaunchConfiguration`，以便后续代码在模块级复用共享定义。

### Line 77 — From `cutlass_cppgen.backend.utils.device` import

```python
from cutlass_cppgen.backend.utils.device import to_device_ptr
```
**EN:** Imports `to_device_ptr` from `cutlass_cppgen.backend.utils.device` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.utils.device` 导入 `to_device_ptr`，以便后续代码在模块级复用共享定义。

### Line 78 — From `cutlass_cppgen.shape` import

```python
from cutlass_cppgen.shape import GemmCoord
```
**EN:** Imports `GemmCoord` from `cutlass_cppgen.shape` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.shape` 导入 `GemmCoord`，以便后续代码在模块级复用共享定义。

### Lines 81-104 — Class `Conv2dArguments`

```python
class Conv2dArguments(ArgumentBase):
    """
    Argument wrapper for Conv2d. It encodes problem information and
    user-provide tensors into the kernel's argument.

    :param operation: the Conv2d operation to take the argument
    :type operation: :class:`cutlass_cppgen.backend.Conv2dOperation`
    :param problem_size: the Conv2d problem size
    :type problem_size: :class:`cutlass_cppgen.shape.Conv2dProblemSize`
    :param A: tensor A
    :type A: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray
    :param B: tensor B
    :type B: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray
    :param C: tensor C
    :type C: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray
    :param D: tensor D
    :type D: cuda.CUdeviceptr | numpy.ndarray | torch.Tensor | cupy.ndarray
    :param split_k_mode: conv2d split K mode, defaults to cutlass_library.library.SplitKMode.Serial
    :type split_k_mode: cutlass_library.library.SplitKMode, optional
    :param output_op: output operator, optional
    :type output_op: :class:`cutlass_cppgen.backend.LinearCombinationFunctorArguments`
    :param stream: cuda stream, defaults to cuda.cuda.CUstream(0)
    :type stream: :class:`cuda.cuda.CUstream`
    """
```
**EN:** Declares class `Conv2dArguments` deriving from `ArgumentBase`. Purpose: Argument wrapper for Conv2d.
**CN:** 声明类 `Conv2dArguments`，其基类为 `ArgumentBase`。 其用途：Argument wrapper for Conv2d.

#### Lines 106-107 — Function `__init__`

```python
    def __init__(self, operation, problem_size, A, B, C, D,
        split_k_mode=SplitKMode.Serial, **kwargs, ) -> None:
```
**EN:** Defines function `__init__` with parameters `self, operation, problem_size, A, B, C, D, split_k_mode, **kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, operation, problem_size, A, B, C, D, split_k_mode, **kwargs`。

##### Line 108 — Assign `self.operation`

```python
        self.operation = operation
```
**EN:** Assigns `self.operation` from `operation`, establishing state in function `__init__`.
**CN:** 将 `self.operation` 赋值为 `operation`，用于在函数 `__init__` 中建立状态。

##### Line 109 — Assign `self.conv_kind`

```python
        self.conv_kind = operation.conv_kind
```
**EN:** Assigns `self.conv_kind` from `operation.conv_kind`, establishing state in function `__init__`.
**CN:** 将 `self.conv_kind` 赋值为 `operation.conv_kind`，用于在函数 `__init__` 中建立状态。

##### Line 110 — Assign `self.layout_A`

```python
        self.layout_A = operation.A.layout
```
**EN:** Assigns `self.layout_A` from `operation.A.layout`, establishing state in function `__init__`.
**CN:** 将 `self.layout_A` 赋值为 `operation.A.layout`，用于在函数 `__init__` 中建立状态。

##### Line 111 — Assign `self.layout_B`

```python
        self.layout_B = operation.B.layout
```
**EN:** Assigns `self.layout_B` from `operation.B.layout`, establishing state in function `__init__`.
**CN:** 将 `self.layout_B` 赋值为 `operation.B.layout`，用于在函数 `__init__` 中建立状态。

##### Line 112 — Assign `self.layout_C`

```python
        self.layout_C = operation.C.layout
```
**EN:** Assigns `self.layout_C` from `operation.C.layout`, establishing state in function `__init__`.
**CN:** 将 `self.layout_C` 赋值为 `operation.C.layout`，用于在函数 `__init__` 中建立状态。

##### Line 114 — Assign `self.element_A`

```python
        self.element_A = operation.A.element
```
**EN:** Assigns `self.element_A` from `operation.A.element`, establishing state in function `__init__`.
**CN:** 将 `self.element_A` 赋值为 `operation.A.element`，用于在函数 `__init__` 中建立状态。

##### Line 115 — Assign `self.element_B`

```python
        self.element_B = operation.B.element
```
**EN:** Assigns `self.element_B` from `operation.B.element`, establishing state in function `__init__`.
**CN:** 将 `self.element_B` 赋值为 `operation.B.element`，用于在函数 `__init__` 中建立状态。

##### Line 116 — Assign `self.element_C`

```python
        self.element_C = operation.C.element
```
**EN:** Assigns `self.element_C` from `operation.C.element`, establishing state in function `__init__`.
**CN:** 将 `self.element_C` 赋值为 `operation.C.element`，用于在函数 `__init__` 中建立状态。

##### Lines 118-119 — Conditional `self.layout_C == LayoutType.TensorNC32HW32`

```python
        if self.layout_C == LayoutType.TensorNC32HW32:
            raise Exception("Layout type TensorNC32HW32 is not currently supported")
```
**EN:** Checks `self.layout_C == LayoutType.TensorNC32HW32` and selects the matching branch in function `__init__`.
**CN:** 检查 `self.layout_C == LayoutType.TensorNC32HW32`，并在函数 `__init__` 中选择匹配的分支。

##### Line 121 — Call `super().__init__`

```python
        super().__init__(A, B, C, D, **kwargs)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Lines 123-128 — Conditional `'split_k_slices' in kwargs.keys() and kwargs['split_k_slices'] > 1`

```python
        if "split_k_slices" in kwargs.keys() and kwargs["split_k_slices"] > 1:
            self.split_k_mode = split_k_mode
            self.split_k_slices = kwargs["split_k_slices"]
        else:
            self.split_k_mode = SplitKMode.Serial
            self.split_k_slices = 1
```
**EN:** Checks `'split_k_slices' in kwargs.keys() and kwargs['split_k_slices'] > 1` and selects the matching branch in function `__init__`.
**CN:** 检查 `'split_k_slices' in kwargs.keys() and kwargs['split_k_slices'] > 1`，并在函数 `__init__` 中选择匹配的分支。

##### Lines 130-133 — Conditional `'output_op' in kwargs.keys() and self.split_k_mode != SplitKMode.Parallel`

```python
        if "output_op" in kwargs.keys() and self.split_k_mode != SplitKMode.Parallel:
            self.output_op = kwargs["output_op"]
        else:
            self.output_op = self.operation.epilogue_type(1.0, 0.0)
```
**EN:** Checks `'output_op' in kwargs.keys() and self.split_k_mode != SplitKMode.Parallel` and selects the matching branch in function `__init__`.
**CN:** 检查 `'output_op' in kwargs.keys() and self.split_k_mode != SplitKMode.Parallel`，并在函数 `__init__` 中选择匹配的分支。

##### Line 135 — Assign `self.problem_size`

```python
        self.problem_size = problem_size
```
**EN:** Assigns `self.problem_size` from `problem_size`, establishing state in function `__init__`.
**CN:** 将 `self.problem_size` 赋值为 `problem_size`，用于在函数 `__init__` 中建立状态。

##### Line 136 — Assign `self.problem_size.split_k_slices`

```python
        self.problem_size.split_k_slices = self.split_k_slices
```
**EN:** Assigns `self.problem_size.split_k_slices` from `self.split_k_slices`, establishing state in function `__init__`.
**CN:** 将 `self.problem_size.split_k_slices` 赋值为 `self.split_k_slices`，用于在函数 `__init__` 中建立状态。

##### Line 138 — Call `self.initialize`

```python
        self.initialize()
```
**EN:** Calls `self.initialize` for side effects or initialization work in function `__init__`.
**CN:** 调用 `self.initialize` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

#### Line 140 — Function `get_arguments`

```python
    def get_arguments(self):
```
**EN:** Defines function `get_arguments` with parameters `self`.
**CN:** 定义函数 `get_arguments`，参数为 `self`。

##### Line 141 — Assign `tc_numel`

```python
        tc_numel = -1
```
**EN:** Assigns `tc_numel` from `-1`, establishing state in function `get_arguments`.
**CN:** 将 `tc_numel` 赋值为 `-1`，用于在函数 `get_arguments` 中建立状态。

##### Lines 142-143 — Conditional `hasattr(self, 'tensor_c_numel')`

```python
        if hasattr(self, "tensor_c_numel"):
            tc_numel = self.tensor_c_numel
```
**EN:** Checks `hasattr(self, 'tensor_c_numel')` and selects the matching branch in function `get_arguments`.
**CN:** 检查 `hasattr(self, 'tensor_c_numel')`，并在函数 `get_arguments` 中选择匹配的分支。

##### Lines 145-155 — Assign `self.c_arguments`

```python
        self.c_arguments = self.operation.argument_type(
            int(self.conv_kind),
            self.problem_size.ctype,
            int(to_device_ptr(self.ptr_A)),
            int(to_device_ptr(self.ptr_B)),
            int(to_device_ptr(self.ptr_C)),
            int(to_device_ptr(self.ptr_D)),
            tc_numel,
            self.output_op,
            int(self.split_k_mode)
        )
```
**EN:** Assigns `self.c_arguments` from `self.operation.argument_type(int(self.conv_kind), self.problem_size.ctype, int(to_device_ptr(self.ptr_A)), int(to_device_ptr(self.ptr_B)), int(to_device_ptr(...`, establishing state in function `get_arguments`.
**CN:** 将 `self.c_arguments` 赋值为 `self.operation.argument_type(int(self.conv_kind), self.problem_size.ctype, int(to_device_ptr(self.ptr_A)), int(to_device_ptr(self.ptr_B)), int(to_device_ptr(...`，用于在函数 `get_arguments` 中建立状态。

#### Line 157 — Function `initialize`

```python
    def initialize(self):
```
**EN:** Defines function `initialize` with parameters `self`.
**CN:** 定义函数 `initialize`，参数为 `self`。

##### Line 158 — Assign `self.launch_config`

```python
        self.launch_config = self.operation.rt_module.plan(self)
```
**EN:** Assigns `self.launch_config` from `self.operation.rt_module.plan(self)`, establishing state in function `initialize`.
**CN:** 将 `self.launch_config` 赋值为 `self.operation.rt_module.plan(self)`，用于在函数 `initialize` 中建立状态。

##### Line 160 — Call `self.get_arguments`

```python
        self.get_arguments()
```
**EN:** Calls `self.get_arguments` for side effects or initialization work in function `initialize`.
**CN:** 调用 `self.get_arguments` 执行副作用或初始化逻辑；该语句位于在函数 `initialize` 中。

##### Line 162 — Comment or spacing block

```python
        # Allocate and initialize device workspace
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 163 — Assign `device_workspace_size`

```python
        device_workspace_size = self.operation.rt_module.get_workspace_size(self.c_arguments)
```
**EN:** Assigns `device_workspace_size` from `self.operation.rt_module.get_workspace_size(self.c_arguments)`, establishing state in function `initialize`.
**CN:** 将 `device_workspace_size` 赋值为 `self.operation.rt_module.get_workspace_size(self.c_arguments)`，用于在函数 `initialize` 中建立状态。

##### Lines 164-170 — Conditional `device_workspace_size > 0`

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

##### Line 172 — Assign `self.semaphore`

```python
        self.semaphore = 0
```
**EN:** Assigns `self.semaphore` from `0`, establishing state in function `initialize`.
**CN:** 将 `self.semaphore` 赋值为 `0`，用于在函数 `initialize` 中建立状态。

##### Lines 173-178 — Conditional `workspace_ptr is not None and self.split_k_mode == SplitKMode.Parallel`

```python
        if workspace_ptr is not None and self.split_k_mode == SplitKMode.Parallel:
            self.ptr_D = workspace_ptr
            # Reset arguments now that ptr_D has been updated
            self.get_arguments()
        elif workspace_ptr is not None and self.split_k_mode == SplitKMode.Serial:
            self.semaphore = workspace_ptr
```
**EN:** Checks `workspace_ptr is not None and self.split_k_mode == SplitKMode.Parallel` and selects the matching branch in function `initialize`.
**CN:** 检查 `workspace_ptr is not None and self.split_k_mode == SplitKMode.Parallel`，并在函数 `initialize` 中选择匹配的分支。

##### Lines 180-181 — Assign `params_`

```python
        params_ = self.operation.rt_module.get_args(
            self.c_arguments, ctypes.c_void_p(int(self.semaphore)))
```
**EN:** Assigns `params_` from `self.operation.rt_module.get_args(self.c_arguments, ctypes.c_void_p(int(self.semaphore)))`, establishing state in function `initialize`.
**CN:** 将 `params_` 赋值为 `self.operation.rt_module.get_args(self.c_arguments, ctypes.c_void_p(int(self.semaphore)))`，用于在函数 `initialize` 中建立状态。

##### Line 182 — Assign `self.host_workspace`

```python
        self.host_workspace = bytearray(params_.contents)
```
**EN:** Assigns `self.host_workspace` from `bytearray(params_.contents)`, establishing state in function `initialize`.
**CN:** 将 `self.host_workspace` 赋值为 `bytearray(params_.contents)`，用于在函数 `initialize` 中建立状态。

##### Line 183 — Assign `self.device_workspace`

```python
        self.device_workspace = None
```
**EN:** Assigns `self.device_workspace` from `None`, establishing state in function `initialize`.
**CN:** 将 `self.device_workspace` 赋值为 `None`，用于在函数 `initialize` 中建立状态。

#### Lines 185-189 — Function `sync`

```python
    def sync(self):
        """
        Synchronize the arguments. If the input tensor is in host,
        copy it from device to host.
        """
```
**EN:** Defines function `sync` with parameters `self`. Purpose: Synchronize the arguments.
**CN:** 定义函数 `sync`，参数为 `self`。 其用途：Synchronize the arguments.

##### Line 190 — Return

```python
        return super().sync()
```
**EN:** Returns `super().sync()` to the caller.
**CN:** 向调用方返回 `super().sync()`。

### Lines 193-196 — Class `Conv2dRT`

```python
class Conv2dRT(ExecutableOperation):
    """
    Conv2dRT manages the CUTLASS runtime components
    """
```
**EN:** Declares class `Conv2dRT` deriving from `ExecutableOperation`. Purpose: Conv2dRT manages the CUTLASS runtime components
**CN:** 声明类 `Conv2dRT`，其基类为 `ExecutableOperation`。 其用途：Conv2dRT manages the CUTLASS runtime components

#### Lines 198-214 — Assign `KernelTemplate`

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
**EN:** Assigns `KernelTemplate` from `'\nextern "C"\n__global__ void\n${operation_name}(${operation_name}${operation_suffix}::Params params) {\n\n // Dynamic shared memory base pointer\n extern _...`, establishing state in class `Conv2dRT`.
**CN:** 将 `KernelTemplate` 赋值为 `'\nextern "C"\n__global__ void\n${operation_name}(${operation_name}${operation_suffix}::Params params) {\n\n // Dynamic shared memory base pointer\n extern _...`，用于在类 `Conv2dRT` 中建立状态。

#### Lines 216-318 — Assign `HostTemplate`

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

  using ElementA = typename ${operation_name}_base::ElementA;
  using ElementB = typename ${operation_name}_base::ElementB;
  using ElementC = typename ${operation_name}_base::ElementC;
  using LayoutA = typename ${operation_name}_base::LayoutA;
  using LayoutB = typename ${operation_name}_base::LayoutB;
  using LayoutC = typename ${operation_name}_base::LayoutC;
  using EpilogueOutputOp = typename ${operation_name}_base::EpilogueOutputOp;

  struct ${operation_name}_TemporaryArgs {
    int conv_kind;
    cutlass::conv::Conv2dProblemSize problem_size;
    ElementA* ptr_A;
    ElementB* ptr_B;
    ElementC* ptr_C;
    ElementC* ptr_D;
    int tensor_c_numel;
    typename EpilogueOutputOp::Params epilogue_params;
    int split_k_mode;
  };

  typename ${operation_name}${operation_suffix}::Arguments
  construct_arguments(${operation_name}_TemporaryArgs args) {
    cutlass::conv::Operator conv_operator = static_cast<cutlass::conv::Operator>(args.conv_kind);
    auto tc_A = cutlass::conv::implicit_gemm_tensor_a_extent(conv_operator, args.problem_size);
    auto tc_B = cutlass::conv::implicit_gemm_tensor_b_extent(conv_operator, args.problem_size);
    auto tc_C = cutlass::conv::implicit_gemm_tensor_c_extent(conv_operator, args.problem_size);
    auto tc_D = cutlass::conv::implicit_gemm_tensor_c_extent(conv_operator, args.problem_size);

    auto size_C = tc_C.at(0) * tc_C.at(1) * tc_C.at(2) * tc_C.at(3);
    if (args.tensor_c_numel >= 0 && args.tensor_c_numel == tc_C.at(3) && args.tensor_c_numel < size_C) {
      // C is interpreted as bias
      tc_C = {0, 0, 0, 0};
    }

    cutlass::TensorRef<ElementA, LayoutA> tref_A(args.ptr_A, LayoutA::packed(tc_A));
    cutlass::TensorRef<ElementB, LayoutA> tref_B(args.ptr_B, LayoutB::packed(tc_B));
    cutlass::TensorRef<ElementC, LayoutA> tref_C(args.ptr_C, LayoutC::packed(tc_C));
    cutlass::TensorRef<ElementC, LayoutA> tref_D(args.ptr_D, LayoutC::packed(tc_D));

    return {
      args.problem_size,
      tref_A,
      tref_B,
      tref_C,
      tref_D,
      args.epilogue_params,
      static_cast<cutlass::conv::SplitKMode>(args.split_k_mode)
    };
  }

  // Get the params as byte array
  char* ${operation_name}_get_params(${operation_name}_TemporaryArgs args, int *semaphore=nullptr) {
    auto arguments = construct_arguments(args);
    typename ${operation_name}${operation_suffix}::Params* params;
    params = new ${operation_name}${operation_suffix}::Params(arguments, semaphore);

    char *bytes = ((char*)(params));
    char *output = new char[sizeof(${operation_name}${operation_suffix}::Params)];
    for (unsigned int i = 0; i < sizeof(${operation_name}${operation_suffix}::Params); i ++)
      output[i] = bytes[i];

    return output;
  }

  dim3 ${operation_name}_get_grid_shape(
    int conv_kind,
    cutlass::conv::Conv2dProblemSize problem_size,
    cutlass::gemm::GemmCoord tile_size,
    int split_k_slices
  ) {

    using Swizzle = typename ${operation_name}_base::ThreadblockSwizzle;
    auto tiled_shape = Swizzle::get_tiled_shape(
      static_cast<cutlass::conv::Operator>(conv_kind),
      problem_size,
      tile_size,
      split_k_slices);

    return Swizzle::get_grid_shape(tiled_shape);
  }

  size_t ${operation_name}_get_workspace_size(${operation_name}_TemporaryArgs args) {
    auto arguments = construct_arguments(args);

    // Temporarily define device::-level Conv2d so that we can call get_workspace_size
    using DeviceConv = cutlass::conv::device::ImplicitGemmConvolution<${operation_name}_base>;
    return DeviceConv::get_workspace_size(arguments);
  }
}

    """
```
**EN:** Assigns `HostTemplate` from `'\nextern "C" {\n // Get the size of params in bytes\n int ${operation_name}_get_param_size(){\n return sizeof(${operation_name}${operation_suffix}::Params);...`, establishing state in class `Conv2dRT`.
**CN:** 将 `HostTemplate` 赋值为 `'\nextern "C" {\n // Get the size of params in bytes\n int ${operation_name}_get_param_size(){\n return sizeof(${operation_name}${operation_suffix}::Params);...`，用于在类 `Conv2dRT` 中建立状态。

#### Line 320 — Function `__init__`

```python
    def __init__(self, operation: "Conv2dOperation"):
```
**EN:** Defines function `__init__` with parameters `self, operation`.
**CN:** 定义函数 `__init__`，参数为 `self, operation`。

##### Line 321 — Call `super().__init__`

```python
        super().__init__(operation)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Lines 322-325 — Assign `self.extra_funcs`

```python
        self.extra_funcs = {
            "get_grid_shape": dim3_,
            "get_workspace_size": ctypes.c_uint64
        }
```
**EN:** Assigns `self.extra_funcs` from `{'get_grid_shape': dim3_, 'get_workspace_size': ctypes.c_uint64}`, establishing state in function `__init__`.
**CN:** 将 `self.extra_funcs` 赋值为 `{'get_grid_shape': dim3_, 'get_workspace_size': ctypes.c_uint64}`，用于在函数 `__init__` 中建立状态。

##### Line 326 — Assign `self.argument_type, self.epilogue_type`

```python
        self.argument_type, self.epilogue_type = get_conv2d_arguments(operation.epilogue_functor)
```
**EN:** Assigns `self.argument_type, self.epilogue_type` from `get_conv2d_arguments(operation.epilogue_functor)`, establishing state in function `__init__`.
**CN:** 将 `self.argument_type, self.epilogue_type` 赋值为 `get_conv2d_arguments(operation.epilogue_functor)`，用于在函数 `__init__` 中建立状态。

##### Line 327 — Assign `self.argtype`

```python
        self.argtype = [ctypes.POINTER(self.argument_type), ctypes.c_void_p]
```
**EN:** Assigns `self.argtype` from `[ctypes.POINTER(self.argument_type), ctypes.c_void_p]`, establishing state in function `__init__`.
**CN:** 将 `self.argtype` 赋值为 `[ctypes.POINTER(self.argument_type), ctypes.c_void_p]`，用于在函数 `__init__` 中建立状态。

##### Line 328 — Assign `self.conv_kind`

```python
        self.conv_kind = operation.conv_kind
```
**EN:** Assigns `self.conv_kind` from `operation.conv_kind`, establishing state in function `__init__`.
**CN:** 将 `self.conv_kind` 赋值为 `operation.conv_kind`，用于在函数 `__init__` 中建立状态。

##### Line 330 — Annotated assign `self.operation`

```python
        self.operation: Conv2dOperation = operation
```
**EN:** Declares `self.operation` with an annotation and initializes it from `operation` in function `__init__`.
**CN:** 带类型标注地声明 `self.operation`，并用 `operation` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 332 — Assign `self.emitter`

```python
        self.emitter = EmitConv2dInstance("_type")
```
**EN:** Assigns `self.emitter` from `EmitConv2dInstance('_type')`, establishing state in function `__init__`.
**CN:** 将 `self.emitter` 赋值为 `EmitConv2dInstance('_type')`，用于在函数 `__init__` 中建立状态。

##### Line 334 — Assign `self.threads`

```python
        self.threads = operation.tile_description.num_threads
```
**EN:** Assigns `self.threads` from `operation.tile_description.num_threads`, establishing state in function `__init__`.
**CN:** 将 `self.threads` 赋值为 `operation.tile_description.num_threads`，用于在函数 `__init__` 中建立状态。

##### Line 336 — Assign `self.swizzle_functor`

```python
        self.swizzle_functor = operation.swizzling_functor
```
**EN:** Assigns `self.swizzle_functor` from `operation.swizzling_functor`, establishing state in function `__init__`.
**CN:** 将 `self.swizzle_functor` 赋值为 `operation.swizzling_functor`，用于在函数 `__init__` 中建立状态。

#### Line 338 — Function `emit`

```python
    def emit(self):
```
**EN:** Defines function `emit` with parameters `self`.
**CN:** 定义函数 `emit`，参数为 `self`。

##### Line 339 — Return

```python
        return self.emitter.emit(self.operation)
```
**EN:** Returns `self.emitter.emit(self.operation)` to the caller.
**CN:** 向调用方返回 `self.emitter.emit(self.operation)`。

#### Line 341 — Function `plan`

```python
    def plan(self, arguments: Conv2dArguments):
```
**EN:** Defines function `plan` with parameters `self, arguments`.
**CN:** 定义函数 `plan`，参数为 `self, arguments`。

##### Lines 342-346 — Assign `tile_size`

```python
        tile_size = GemmCoord(
            self.operation.tile_description.threadblock_shape[0],
            self.operation.tile_description.threadblock_shape[1],
            self.operation.tile_description.threadblock_shape[2],
        )
```
**EN:** Assigns `tile_size` from `GemmCoord(self.operation.tile_description.threadblock_shape[0], self.operation.tile_description.threadblock_shape[1], self.operation.tile_description.threadb...`, establishing state in function `plan`.
**CN:** 将 `tile_size` 赋值为 `GemmCoord(self.operation.tile_description.threadblock_shape[0], self.operation.tile_description.threadblock_shape[1], self.operation.tile_description.threadb...`，用于在函数 `plan` 中建立状态。

##### Lines 348-353 — Assign `grid`

```python
        grid = self.get_grid_shape(
            int(self.conv_kind),
            arguments.problem_size.ctype,
            tile_size.ctype,
            arguments.split_k_slices
        )
```
**EN:** Assigns `grid` from `self.get_grid_shape(int(self.conv_kind), arguments.problem_size.ctype, tile_size.ctype, arguments.split_k_slices)`, establishing state in function `plan`.
**CN:** 将 `grid` 赋值为 `self.get_grid_shape(int(self.conv_kind), arguments.problem_size.ctype, tile_size.ctype, arguments.split_k_slices)`，用于在函数 `plan` 中建立状态。

##### Lines 355-357 — Return

```python
        return LaunchConfiguration(
            [grid.x, grid.y, grid.z], [self.threads, 1, 1],
            self.shared_memory_capacity)
```
**EN:** Returns `LaunchConfiguration([grid.x, grid.y, grid.z], [self.threads, 1, 1], self.shared_memory_capacity)` to the caller.
**CN:** 向调用方返回 `LaunchConfiguration([grid.x, grid.y, grid.z], [self.threads, 1, 1], self.shared_memory_capacity)`。

#### Line 359 — Function `initialize`

```python
    def initialize(self):
```
**EN:** Defines function `initialize` with parameters `self`.
**CN:** 定义函数 `initialize`，参数为 `self`。

##### Lines 360-363 — Assign `err`

```python
        err, = cuda.cuFuncSetAttribute(
            self.kernel,
            attrib=cuda.CUfunction_attribute.CU_FUNC_ATTRIBUTE_MAX_DYNAMIC_SHARED_SIZE_BYTES,
            value=self.shared_memory_capacity)
```
**EN:** Assigns `err` from `cuda.cuFuncSetAttribute(self.kernel, attrib=cuda.CUfunction_attribute.CU_FUNC_ATTRIBUTE_MAX_DYNAMIC_SHARED_SIZE_BYTES, value=self.shared_memory_capacity)`, establishing state in function `initialize`.
**CN:** 将 `err` 赋值为 `cuda.cuFuncSetAttribute(self.kernel, attrib=cuda.CUfunction_attribute.CU_FUNC_ATTRIBUTE_MAX_DYNAMIC_SHARED_SIZE_BYTES, value=self.shared_memory_capacity)`，用于在函数 `initialize` 中建立状态。

##### Lines 364-365 — Conditional `err != cuda.CUresult.CUDA_SUCCESS`

```python
        if err != cuda.CUresult.CUDA_SUCCESS:
            raise RuntimeError(f"CUDA Error: {err}")
```
**EN:** Checks `err != cuda.CUresult.CUDA_SUCCESS` and selects the matching branch in function `initialize`.
**CN:** 检查 `err != cuda.CUresult.CUDA_SUCCESS`，并在函数 `initialize` 中选择匹配的分支。

### Lines 368-408 — Class `Conv2dOperation`

```python
class Conv2dOperation:
    """
    CUTLASS Conv2d operation description.

    :param conv_kind: convolution operator
    :type conv_kind: :class:`cutlass_library.library.ConvKind`

    :param iterator_algorithm: Selects among several implementation
    variants trading off performance with simplicity
    :type iterator_algorithm: :class:`cutlass_library.library.IteratorAlgorithm`

    :param arch: GPU compute capability (sm_xx)
    :type arch: int

    :param tile_description: tile description
    :type tile_description: :class:`cutlass_cppgen.backend.TileDescription`

    :param A: tensor A description
    :type A: :class:`cutlass_cppgen.backend.TensorDescription`

    :param B: tensor B description
    :type B: :class:`cutlass_cppgen.backend.TensorDescription`

    :param C: tensor C description
    :type C: :class:`cutlass_cppgen.backend.TensorDescription`

    :param D: tensor D description
    :type D: :class:`cutlass_cppgen.backend.TensorDescription`

    :param element_epilogue: element type for computation in epilogue \
    :type element_epilogue: cutlass_library.library.DataType

    :param stride_support: distinguish among partial specializations that \
    accelerate certain problems where convolution stride is unit \
    :type stride_support: :class:`cutlass_library.library.StrideSupport`

    :param epilogue_functor: convolution epilogue functor
    :type epilogue_functor: :class:`EpilogueFunctor`

    :param swizzling_functor: threadblock swizzling functor
    """
```
**EN:** Declares class `Conv2dOperation` deriving from `object`. Purpose: CUTLASS Conv2d operation description.
**CN:** 声明类 `Conv2dOperation`，其基类为 `object`。 其用途：CUTLASS Conv2d operation description.

#### Lines 409-423 — Function `__init__`

```python
    def __init__(
        self,
        conv_kind,
        iterator_algorithm,
        arch: int,
        tile_description: TileDescription,
        A: TensorDescription,
        B: TensorDescription,
        C: TensorDescription,
        stride_support,
        epilogue_functor,
        swizzling_functor=SwizzlingFunctor.Identity1,
        emission_type=EmissionType.Kernel,
        **kwargs
    ):
```
**EN:** Defines function `__init__` with parameters `self, conv_kind, iterator_algorithm, arch, tile_description, A, B, C, stride_support, epilogue_functor, swizzling_functor, emission_type, **kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, conv_kind, iterator_algorithm, arch, tile_description, A, B, C, stride_support, epilogue_functor, swizzling_functor, emission_type, **kwargs`。

##### Line 424 — Annotated assign `self.operation_kind`

```python
        self.operation_kind: OperationKind = OperationKind.Conv2d
```
**EN:** Declares `self.operation_kind` with an annotation and initializes it from `OperationKind.Conv2d` in function `__init__`.
**CN:** 带类型标注地声明 `self.operation_kind`，并用 `OperationKind.Conv2d` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 425 — Annotated assign `self.arch`

```python
        self.arch: int = arch
```
**EN:** Declares `self.arch` with an annotation and initializes it from `arch` in function `__init__`.
**CN:** 带类型标注地声明 `self.arch`，并用 `arch` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 426 — Annotated assign `self.tile_description`

```python
        self.tile_description: TileDescription = tile_description
```
**EN:** Declares `self.tile_description` with an annotation and initializes it from `tile_description` in function `__init__`.
**CN:** 带类型标注地声明 `self.tile_description`，并用 `tile_description` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 427 — Assign `self.conv_kind`

```python
        self.conv_kind = conv_kind
```
**EN:** Assigns `self.conv_kind` from `conv_kind`, establishing state in function `__init__`.
**CN:** 将 `self.conv_kind` 赋值为 `conv_kind`，用于在函数 `__init__` 中建立状态。

##### Line 428 — Annotated assign `self.A`

```python
        self.A: TensorDescription = A
```
**EN:** Declares `self.A` with an annotation and initializes it from `A` in function `__init__`.
**CN:** 带类型标注地声明 `self.A`，并用 `A` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 429 — Annotated assign `self.B`

```python
        self.B: TensorDescription = B
```
**EN:** Declares `self.B` with an annotation and initializes it from `B` in function `__init__`.
**CN:** 带类型标注地声明 `self.B`，并用 `B` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 430 — Annotated assign `self.C`

```python
        self.C: TensorDescription = C
```
**EN:** Declares `self.C` with an annotation and initializes it from `C` in function `__init__`.
**CN:** 带类型标注地声明 `self.C`，并用 `C` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 431 — Assign `self.epilogue_functor`

```python
        self.epilogue_functor = epilogue_functor
```
**EN:** Assigns `self.epilogue_functor` from `epilogue_functor`, establishing state in function `__init__`.
**CN:** 将 `self.epilogue_functor` 赋值为 `epilogue_functor`，用于在函数 `__init__` 中建立状态。

##### Line 432 — Assign `self.iterator_algorithm`

```python
        self.iterator_algorithm = iterator_algorithm
```
**EN:** Assigns `self.iterator_algorithm` from `iterator_algorithm`, establishing state in function `__init__`.
**CN:** 将 `self.iterator_algorithm` 赋值为 `iterator_algorithm`，用于在函数 `__init__` 中建立状态。

##### Line 433 — Assign `self.stride_support`

```python
        self.stride_support = stride_support
```
**EN:** Assigns `self.stride_support` from `stride_support`, establishing state in function `__init__`.
**CN:** 将 `self.stride_support` 赋值为 `stride_support`，用于在函数 `__init__` 中建立状态。

##### Line 434 — Assign `self.swizzling_functor`

```python
        self.swizzling_functor = swizzling_functor
```
**EN:** Assigns `self.swizzling_functor` from `swizzling_functor`, establishing state in function `__init__`.
**CN:** 将 `self.swizzling_functor` 赋值为 `swizzling_functor`，用于在函数 `__init__` 中建立状态。

##### Line 436 — Assign `self.emission_type`

```python
        self.emission_type = emission_type
```
**EN:** Assigns `self.emission_type` from `emission_type`, establishing state in function `__init__`.
**CN:** 将 `self.emission_type` 赋值为 `emission_type`，用于在函数 `__init__` 中建立状态。

##### Line 438 — Annotated assign `self.rt_module`

```python
        self.rt_module: Conv2dRT = Conv2dRT(self)
```
**EN:** Declares `self.rt_module` with an annotation and initializes it from `Conv2dRT(self)` in function `__init__`.
**CN:** 带类型标注地声明 `self.rt_module`，并用 `Conv2dRT(self)` 初始化它；该语句用于在函数 `__init__` 中定义状态。

##### Line 439 — Assign `self.argument_type`

```python
        self.argument_type = self.rt_module.argument_type
```
**EN:** Assigns `self.argument_type` from `self.rt_module.argument_type`, establishing state in function `__init__`.
**CN:** 将 `self.argument_type` 赋值为 `self.rt_module.argument_type`，用于在函数 `__init__` 中建立状态。

##### Line 440 — Assign `self.epilogue_type`

```python
        self.epilogue_type = self.rt_module.epilogue_type
```
**EN:** Assigns `self.epilogue_type` from `self.rt_module.epilogue_type`, establishing state in function `__init__`.
**CN:** 将 `self.epilogue_type` 赋值为 `self.rt_module.epilogue_type`，用于在函数 `__init__` 中建立状态。

#### Lines 442-448 — Function `run`

```python
    def run(self, arguments: Conv2dArguments) -> cuda.CUresult:
        """
        Launch the cuda kernel with input arguments

        :param arguments: conv2d arguments
        :type arguments: :class:`cutlass_cppgen.backend.Conv2dArguments`
        """
```
**EN:** Defines function `run` with parameters `self, arguments`. Purpose: Launch the cuda kernel with input arguments
**CN:** 定义函数 `run`，参数为 `self, arguments`。 其用途：Launch the cuda kernel with input arguments

##### Line 450 — Comment or spacing block

```python
        # launch the kernel
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 451-456 — Assign `err`

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

##### Lines 458-459 — Conditional `err != cuda.CUresult.CUDA_SUCCESS`

```python
        if err != cuda.CUresult.CUDA_SUCCESS:
            raise RuntimeError(f"CUDA Error {err}")
```
**EN:** Checks `err != cuda.CUresult.CUDA_SUCCESS` and selects the matching branch in function `run`.
**CN:** 检查 `err != cuda.CUresult.CUDA_SUCCESS`，并在函数 `run` 中选择匹配的分支。

##### Line 461 — Return

```python
        return err
```
**EN:** Returns `err` to the caller.
**CN:** 向调用方返回 `err`。

#### Lines 463-465 — Comment or spacing block

```python
    #
    # Get function name
    #
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 467-468 — Function `procedural_name`

```python
    def procedural_name(self):
        """The full procedural name indicates architecture, extended name, tile size, and layout."""
```
**EN:** Defines function `procedural_name` with parameters `self`. Purpose: The full procedural name indicates architecture, extended name, tile size, and layout.
**CN:** 定义函数 `procedural_name`，参数为 `self`。 其用途：The full procedural name indicates architecture, extended name, tile size, and layout.

##### Line 469 — Return

```python
        return self.configuration_name()
```
**EN:** Returns `self.configuration_name()` to the caller.
**CN:** 向调用方返回 `self.configuration_name()`。

#### Lines 471-472 — Function `configuration_name`

```python
    def configuration_name(self):
        """The full procedural name indicates architecture, extended name, tile size, and layout."""
```
**EN:** Defines function `configuration_name` with parameters `self`. Purpose: The full procedural name indicates architecture, extended name, tile size, and layout.
**CN:** 定义函数 `configuration_name`，参数为 `self`。 其用途：The full procedural name indicates architecture, extended name, tile size, and layout.

##### Lines 474-476 — Assign `opcode_class_name`

```python
        opcode_class_name = OpcodeClassNames[
            self.tile_description.math_instruction.opcode_class
        ]
```
**EN:** Assigns `opcode_class_name` from `OpcodeClassNames[self.tile_description.math_instruction.opcode_class]`, establishing state in function `configuration_name`.
**CN:** 将 `opcode_class_name` 赋值为 `OpcodeClassNames[self.tile_description.math_instruction.opcode_class]`，用于在函数 `configuration_name` 中建立状态。

##### Lines 478-483 — Assign `threadblock`

```python
        threadblock = "%dx%d_%dx%d" % (
            self.tile_description.threadblock_shape[0],
            self.tile_description.threadblock_shape[1],
            self.tile_description.threadblock_shape[2],
            self.tile_description.stages,
        )
```
**EN:** Assigns `threadblock` from `'%dx%d_%dx%d' % (self.tile_description.threadblock_shape[0], self.tile_description.threadblock_shape[1], self.tile_description.threadblock_shape[2], self.til...`, establishing state in function `configuration_name`.
**CN:** 将 `threadblock` 赋值为 `'%dx%d_%dx%d' % (self.tile_description.threadblock_shape[0], self.tile_description.threadblock_shape[1], self.tile_description.threadblock_shape[2], self.til...`，用于在函数 `configuration_name` 中建立状态。

##### Lines 485-488 — Conditional `self.stride_support == StrideSupport.Unity`

```python
        if self.stride_support == StrideSupport.Unity:
            configuration_name = "cutlass_sm${arch}_${opcode_class}_${extended_name}_${threadblock}_${layout}_unity_stride_align${alignment}"
        else:
            configuration_name = "cutlass_sm${arch}_${opcode_class}_${extended_name}_${threadblock}_${layout}_align${alignment}"
```
**EN:** Checks `self.stride_support == StrideSupport.Unity` and selects the matching branch in function `configuration_name`.
**CN:** 检查 `self.stride_support == StrideSupport.Unity`，并在函数 `configuration_name` 中选择匹配的分支。

##### Lines 490-500 — Return

```python
        return SubstituteTemplate(
            configuration_name,
            {
                "arch": str(self.arch),
                "opcode_class": opcode_class_name,
                "extended_name": self.extended_name(),
                "threadblock": threadblock,
                "layout": self.layout_name(),
                "alignment": "%d" % self.A.alignment
            },
        )
```
**EN:** Returns `SubstituteTemplate(configuration_name, {'arch': str(self.arch), 'opcode_class': opcode_class_name, 'extended_name': self.extended_name(), 'threadblock': thre...` to the caller.
**CN:** 向调用方返回 `SubstituteTemplate(configuration_name, {'arch': str(self.arch), 'opcode_class': opcode_class_name, 'extended_name': self.extended_name(), 'threadblock': thre...`。

#### Lines 502-503 — Function `extended_name`

```python
    def extended_name(self):
        """Append data types if they differ from compute type."""
```
**EN:** Defines function `extended_name` with parameters `self`. Purpose: Append data types if they differ from compute type.
**CN:** 定义函数 `extended_name`，参数为 `self`。 其用途：Append data types if they differ from compute type.

##### Lines 504-511 — Conditional `self.C.element != self.tile_description.math_instruction.element_accumulator and self.A.element != self.tile_description.math_instruction.element_accumulator`

```python
        if self.C.element != self.tile_description.math_instruction.element_accumulator and \
                self.A.element != self.tile_description.math_instruction.element_accumulator:
            extended_name = "${element_c}_${core_name}_${element_a}"
        elif self.C.element == self.tile_description.math_instruction.element_accumulator and  \
                self.A.element != self.tile_description.math_instruction.element_accumulator:
            extended_name = "${core_name}_${element_a}"
        else:
            extended_name = "${core_name}"
```
**EN:** Checks `self.C.element != self.tile_description.math_instruction.element_accumulator and self.A.element != self.tile_description.math_instruction.element_accumulator` and selects the matching branch in function `extended_name`.
**CN:** 检查 `self.C.element != self.tile_description.math_instruction.element_accumulator and self.A.element != self.tile_description.math_instruction.element_accumulator`，并在函数 `extended_name` 中选择匹配的分支。

##### Lines 513-517 — Assign `extended_name`

```python
        extended_name = SubstituteTemplate(extended_name, {
            "element_a": DataTypeNames[self.A.element],
            "element_c": DataTypeNames[self.C.element],
            "core_name": self.core_name(),
        })
```
**EN:** Assigns `extended_name` from `SubstituteTemplate(extended_name, {'element_a': DataTypeNames[self.A.element], 'element_c': DataTypeNames[self.C.element], 'core_name': self.core_name()})`, establishing state in function `extended_name`.
**CN:** 将 `extended_name` 赋值为 `SubstituteTemplate(extended_name, {'element_a': DataTypeNames[self.A.element], 'element_c': DataTypeNames[self.C.element], 'core_name': self.core_name()})`，用于在函数 `extended_name` 中建立状态。

##### Line 519 — Return

```python
        return extended_name
```
**EN:** Returns `extended_name` to the caller.
**CN:** 向调用方返回 `extended_name`。

#### Line 521 — Function `layout_name`

```python
    def layout_name(self):
```
**EN:** Defines function `layout_name` with parameters `self`.
**CN:** 定义函数 `layout_name`，参数为 `self`。

##### Line 522 — Return

```python
        return "%s" % (ShortLayoutTypeNames[self.A.layout])
```
**EN:** Returns `'%s' % ShortLayoutTypeNames[self.A.layout]` to the caller.
**CN:** 向调用方返回 `'%s' % ShortLayoutTypeNames[self.A.layout]`。

#### Lines 524-525 — Function `core_name`

```python
    def core_name(self):
        """The basic operation kind is prefixed with a letter indicating the accumulation type."""
```
**EN:** Defines function `core_name` with parameters `self`. Purpose: The basic operation kind is prefixed with a letter indicating the accumulation type.
**CN:** 定义函数 `core_name`，参数为 `self`。 其用途：The basic operation kind is prefixed with a letter indicating the accumulation type.

##### Line 527 — Assign `intermediate_type`

```python
        intermediate_type = ""
```
**EN:** Assigns `intermediate_type` from `''`, establishing state in function `core_name`.
**CN:** 将 `intermediate_type` 赋值为 `''`，用于在函数 `core_name` 中建立状态。

##### Lines 529-536 — Conditional `self.tile_description.math_instruction.opcode_class == OpcodeClass.TensorOp`

```python
        if self.tile_description.math_instruction.opcode_class == OpcodeClass.TensorOp:
            inst_shape = "%dx%dx%d" % tuple(
                self.tile_description.math_instruction.instruction_shape)
            if self.tile_description.math_instruction.element_a != self.A.element and \
                    self.tile_description.math_instruction.element_a != self.accumulator_type():
                intermediate_type = DataTypeNames[self.tile_description.math_instruction.element_a]
        else:
            inst_shape = ""
```
**EN:** Checks `self.tile_description.math_instruction.opcode_class == OpcodeClass.TensorOp` and selects the matching branch in function `core_name`.
**CN:** 检查 `self.tile_description.math_instruction.opcode_class == OpcodeClass.TensorOp`，并在函数 `core_name` 中选择匹配的分支。

##### Lines 538-544 — Return

```python
        return "%s%s%s%s_%s" % (
            ShortDataTypeNames[self.accumulator_type()],
            inst_shape,
            intermediate_type,
            ConvKindNames[self.conv_kind],
            IteratorAlgorithmNames[self.iterator_algorithm]
        )
```
**EN:** Returns `'%s%s%s%s_%s' % (ShortDataTypeNames[self.accumulator_type()], inst_shape, intermediate_type, ConvKindNames[self.conv_kind], IteratorAlgorithmNames[self.itera...` to the caller.
**CN:** 向调用方返回 `'%s%s%s%s_%s' % (ShortDataTypeNames[self.accumulator_type()], inst_shape, intermediate_type, ConvKindNames[self.conv_kind], IteratorAlgorithmNames[self.itera...`。

#### Line 546 — Function `is_complex`

```python
    def is_complex(self):
```
**EN:** Defines function `is_complex` with parameters `self`.
**CN:** 定义函数 `is_complex`，参数为 `self`。

##### Lines 547-550 — Assign `complex_operators`

```python
        complex_operators = [
            MathOperation.multiply_add_complex,
            MathOperation.multiply_add_complex_gaussian,
        ]
```
**EN:** Assigns `complex_operators` from `[MathOperation.multiply_add_complex, MathOperation.multiply_add_complex_gaussian]`, establishing state in function `is_complex`.
**CN:** 将 `complex_operators` 赋值为 `[MathOperation.multiply_add_complex, MathOperation.multiply_add_complex_gaussian]`，用于在函数 `is_complex` 中建立状态。

##### Line 551 — Return

```python
        return self.tile_description.math_instruction.math_operation in complex_operators
```
**EN:** Returns `self.tile_description.math_instruction.math_operation in complex_operators` to the caller.
**CN:** 向调用方返回 `self.tile_description.math_instruction.math_operation in complex_operators`。

#### Line 553 — Function `accumulator_type`

```python
    def accumulator_type(self):
```
**EN:** Defines function `accumulator_type` with parameters `self`.
**CN:** 定义函数 `accumulator_type`，参数为 `self`。

##### Line 554 — Assign `accum`

```python
        accum = self.tile_description.math_instruction.element_accumulator
```
**EN:** Assigns `accum` from `self.tile_description.math_instruction.element_accumulator`, establishing state in function `accumulator_type`.
**CN:** 将 `accum` 赋值为 `self.tile_description.math_instruction.element_accumulator`，用于在函数 `accumulator_type` 中建立状态。

##### Lines 556-557 — Conditional `self.is_complex()`

```python
        if self.is_complex():
            return get_complex_from_real(accum)
```
**EN:** Checks `self.is_complex()` and selects the matching branch in function `accumulator_type`.
**CN:** 检查 `self.is_complex()`，并在函数 `accumulator_type` 中选择匹配的分支。

##### Line 559 — Return

```python
        return accum
```
**EN:** Returns `accum` to the caller.
**CN:** 向调用方返回 `accum`。

#### Lines 561-568 — Function `device_op`

```python
    def device_op(self):
        """
        Returns a new Conv2dOperation object that is constructed with emission type
        ``EmissionType.Device``.

        :return: operation ready for device-level code emission
        :rtype: Conv2dOperation
        """
```
**EN:** Defines function `device_op` with parameters `self`. Purpose: Returns a new Conv2dOperation object that is constructed with emission type
**CN:** 定义函数 `device_op`，参数为 `self`。 其用途：Returns a new Conv2dOperation object that is constructed with emission type

##### Lines 569-572 — Return

```python
        return Conv2dOperation(
            self.conv_kind, self.iterator_algorithm, self.arch, self.tile_description,
            self.A, self.B, self.C, self.stride_support, self.epilogue_functor, self.swizzling_functor,
            emission_type=EmissionType.Device)
```
**EN:** Returns `Conv2dOperation(self.conv_kind, self.iterator_algorithm, self.arch, self.tile_description, self.A, self.B, self.C, self.stride_support, self.epilogue_functor...` to the caller.
**CN:** 向调用方返回 `Conv2dOperation(self.conv_kind, self.iterator_algorithm, self.arch, self.tile_description, self.A, self.B, self.C, self.stride_support, self.epilogue_functor...`。

### Lines 575-579 — Comment or spacing block

```python
###################################################################################################
#
# Emits single instances of a CUTLASS device-wide operator
#
###################################################################################################
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Line 582 — Class `EmitConv2dInstance`

```python
class EmitConv2dInstance:
```
**EN:** Declares class `EmitConv2dInstance` deriving from `object`.
**CN:** 声明类 `EmitConv2dInstance`，其基类为 `object`。

#### Line 583 — Function `__init__`

```python
    def __init__(self, operation_suffix=""):
```
**EN:** Defines function `__init__` with parameters `self, operation_suffix`.
**CN:** 定义函数 `__init__`，参数为 `self, operation_suffix`。

##### Line 584 — Assign `self.operation_suffix`

```python
        self.operation_suffix = operation_suffix
```
**EN:** Assigns `self.operation_suffix` from `operation_suffix`, establishing state in function `__init__`.
**CN:** 将 `self.operation_suffix` 赋值为 `operation_suffix`，用于在函数 `__init__` 中建立状态。

##### Lines 585-591 — Assign `self.includes`

```python
        self.includes = [
            "cutlass/cutlass.h",
            "cutlass/conv/kernel/default_conv2d_fprop.h",
            "cutlass/conv/kernel/default_conv2d_dgrad.h",
            "cutlass/conv/kernel/default_conv2d_wgrad.h",
            "cutlass/conv/device/implicit_gemm_convolution.h"
        ]
```
**EN:** Assigns `self.includes` from `['cutlass/cutlass.h', 'cutlass/conv/kernel/default_conv2d_fprop.h', 'cutlass/conv/kernel/default_conv2d_dgrad.h', 'cutlass/conv/kernel/default_conv2d_wgrad.h...`, establishing state in function `__init__`.
**CN:** 将 `self.includes` 赋值为 `['cutlass/cutlass.h', 'cutlass/conv/kernel/default_conv2d_fprop.h', 'cutlass/conv/kernel/default_conv2d_dgrad.h', 'cutlass/conv/kernel/default_conv2d_wgrad.h...`，用于在函数 `__init__` 中建立状态。

##### Lines 592-621 — Assign `self.template`

```python
        self.template = """
// Conv2d${conv_kind_name} ${iterator_algorithm_name} kernel instance "${operation_name}"
using ${operation_name}_base =
typename cutlass::conv::kernel::DefaultConv2d${conv_kind_name}<
  ${element_a},
  ${layout_a},
  ${element_b},
  ${layout_b},
  ${element_c},
  ${layout_c},
  ${element_accumulator},
  ${opcode_class},
  ${arch},
  cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
  cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k} >,
  cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
  ${epilogue_functor},
  ${swizzling_functor},
  ${stages},
  ${math_operator},
  ${iterator_algorithm},
  ${stride_support},
  ${align_a},
  ${align_b}
>::Kernel;

struct ${operation_name}${operation_suffix}:
  public ${operation_name}_base { };

"""
```
**EN:** Assigns `self.template` from `'\n// Conv2d${conv_kind_name} ${iterator_algorithm_name} kernel instance "${operation_name}"\nusing ${operation_name}_base =\ntypename cutlass::conv::kernel:...`, establishing state in function `__init__`.
**CN:** 将 `self.template` 赋值为 `'\n// Conv2d${conv_kind_name} ${iterator_algorithm_name} kernel instance "${operation_name}"\nusing ${operation_name}_base =\ntypename cutlass::conv::kernel:...`，用于在函数 `__init__` 中建立状态。

##### Lines 623-651 — Assign `self.template_device`

```python
        self.template_device = """
// Conv2d operation ${operation_name}

using Conv2d${conv_kind_name}Kernel = typename cutlass::conv::kernel::DefaultConv2d${conv_kind_name}<
  ${element_a},
  ${layout_a},
  ${element_b},
  ${layout_b},
  ${element_c},
  ${layout_c},
  ${element_accumulator},
  ${opcode_class},
  ${arch},
  cutlass::gemm::GemmShape<${threadblock_shape_m}, ${threadblock_shape_n}, ${threadblock_shape_k}>,
  cutlass::gemm::GemmShape<${warp_shape_m}, ${warp_shape_n}, ${warp_shape_k} >,
  cutlass::gemm::GemmShape<${instruction_shape_m}, ${instruction_shape_n}, ${instruction_shape_k}>,
  ${epilogue_functor},
  ${swizzling_functor},
  ${stages},
  ${math_operator},
  ${iterator_algorithm},
  ${stride_support},
  ${align_a},
  ${align_b}
>::Kernel;

using DeviceKernel =
    typename cutlass::conv::device::ImplicitGemmConvolution<Conv2d${conv_kind_name}Kernel>;
"""
```
**EN:** Assigns `self.template_device` from `'\n// Conv2d operation ${operation_name}\n\nusing Conv2d${conv_kind_name}Kernel = typename cutlass::conv::kernel::DefaultConv2d${conv_kind_name}<\n ${element...`, establishing state in function `__init__`.
**CN:** 将 `self.template_device` 赋值为 `'\n// Conv2d operation ${operation_name}\n\nusing Conv2d${conv_kind_name}Kernel = typename cutlass::conv::kernel::DefaultConv2d${conv_kind_name}<\n ${element...`，用于在函数 `__init__` 中建立状态。

#### Line 653 — Function `emit`

```python
    def emit(self, operation):
```
**EN:** Defines function `emit` with parameters `self, operation`.
**CN:** 定义函数 `emit`，参数为 `self, operation`。

##### Lines 654-655 — Assign `warp_shape`

```python
        warp_shape = [int(operation.tile_description.threadblock_shape[idx] /
                          operation.tile_description.warp_count[idx]) for idx in range(3)]
```
**EN:** Assigns `warp_shape` from `[int(operation.tile_description.threadblock_shape[idx] / operation.tile_description.warp_count[idx]) for idx in range(3)]`, establishing state in function `emit`.
**CN:** 将 `warp_shape` 赋值为 `[int(operation.tile_description.threadblock_shape[idx] / operation.tile_description.warp_count[idx]) for idx in range(3)]`，用于在函数 `emit` 中建立状态。

##### Lines 657-658 — Assign `epilogue_vector_length`

```python
        epilogue_vector_length = int(min(
            operation.C.alignment * DataTypeSize[operation.C.element], 128) / DataTypeSize[operation.C.element])
```
**EN:** Assigns `epilogue_vector_length` from `int(min(operation.C.alignment * DataTypeSize[operation.C.element], 128) / DataTypeSize[operation.C.element])`, establishing state in function `emit`.
**CN:** 将 `epilogue_vector_length` 赋值为 `int(min(operation.C.alignment * DataTypeSize[operation.C.element], 128) / DataTypeSize[operation.C.element])`，用于在函数 `emit` 中建立状态。

##### Lines 660-693 — Assign `values`

```python
        values = {
            "operation_name": operation.procedural_name(),
            "operation_suffix": self.operation_suffix,
            "conv_kind": ConvKindTag[operation.conv_kind],
            "conv_kind_name": ConvKindNames[operation.conv_kind].capitalize(),
            "element_a": DataTypeTag[operation.A.element],
            "layout_a": LayoutTag[operation.A.layout],
            "element_b": DataTypeTag[operation.B.element],
            "layout_b": LayoutTag[operation.B.layout],
            "element_c": DataTypeTag[operation.C.element],
            "layout_c": LayoutTag[operation.C.layout],
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
            "epilogue_vector_length": str(epilogue_vector_length),
            "epilogue_functor": operation.epilogue_functor.emit(),
            "swizzling_functor": SwizzlingFunctorTag[operation.swizzling_functor],
            "stages": str(operation.tile_description.stages),
            "iterator_algorithm": IteratorAlgorithmTag[operation.iterator_algorithm],
            "iterator_algorithm_name": IteratorAlgorithmNames[operation.iterator_algorithm].capitalize(),
            "stride_support": StrideSupportTag[operation.stride_support],
            "math_operator": "cutlass::arch::OpMultiplyAddComplex" if operation.is_complex() else MathOperationTag[operation.tile_description.math_instruction.math_operation],
            "align_a": str(operation.A.alignment),
            "align_b": str(operation.B.alignment),
        }
```
**EN:** Assigns `values` from `{'operation_name': operation.procedural_name(), 'operation_suffix': self.operation_suffix, 'conv_kind': ConvKindTag[operation.conv_kind], 'conv_kind_name': C...`, establishing state in function `emit`.
**CN:** 将 `values` 赋值为 `{'operation_name': operation.procedural_name(), 'operation_suffix': self.operation_suffix, 'conv_kind': ConvKindTag[operation.conv_kind], 'conv_kind_name': C...`，用于在函数 `emit` 中建立状态。

##### Lines 695-698 — Conditional `operation.emission_type == EmissionType.Kernel`

```python
        if operation.emission_type == EmissionType.Kernel:
            conv2d_template = self.template
        else:
            conv2d_template = self.template_device
```
**EN:** Checks `operation.emission_type == EmissionType.Kernel` and selects the matching branch in function `emit`.
**CN:** 检查 `operation.emission_type == EmissionType.Kernel`，并在函数 `emit` 中选择匹配的分支。

##### Line 700 — Return

```python
        return SubstituteTemplate(conv2d_template, values)
```
**EN:** Returns `SubstituteTemplate(conv2d_template, values)` to the caller.
**CN:** 向调用方返回 `SubstituteTemplate(conv2d_template, values)`。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `Conv2dArguments`, `Conv2dRT`, `Conv2dOperation`, `EmitConv2dInstance`.
- **CN:** 顶层类：`Conv2dArguments`, `Conv2dRT`, `Conv2dOperation`, `EmitConv2dInstance`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module organizes backend utilities, data structures, or runtime helpers used by cutlass_cppgen.
- **CN:** 该模块组织 cutlass_cppgen 所需的后端工具、数据结构或运行时辅助逻辑。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.arguments`, `cutlass_cppgen.backend.c_types`, `cutlass_cppgen.backend.library`, `cutlass_cppgen.backend.memory_manager`, `cutlass_cppgen.backend.operation`, `cutlass_cppgen.backend.utils.device`, `cutlass_cppgen.shape`, `cutlass_cppgen.utils.lazy_import`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** `__future__`, `ctypes`, `numpy`, `typing`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
