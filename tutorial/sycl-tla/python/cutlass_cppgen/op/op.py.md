# op.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/op/op.py`
- **EN:** Base operation used for defining high-level CUTLASS operations (e.g., GEMM, Conv2d)
- **CN:** 模块文档说明：Base operation used for defining high-level CUTLASS operations (e.g., GEMM, Conv2d)

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
Base operation used for defining high-level CUTLASS operations (e.g., GEMM, Conv2d)
"""
```
**EN:** Docstring explains this scope: Base operation used for defining high-level CUTLASS operations (e.g., GEMM, Conv2d)
**CN:** 文档字符串说明了该作用域的用途：Base operation used for defining high-level CUTLASS operations (e.g., GEMM, Conv2d)

### Line 37 — From `bisect` import

```python
from bisect import bisect_left
```
**EN:** Imports `bisect_left` from `bisect` to reuse shared definitions at module scope.
**CN:** 从 `bisect` 导入 `bisect_left`，以便后续代码在模块级复用共享定义。

### Lines 39-45 — From `cutlass_library` import

```python
from cutlass_library import (
    DataType,
    DataTypeSize,
    MathOperation,
    OperationKind,
    SharedMemPerCC
)
```
**EN:** Imports `DataType, DataTypeSize, MathOperation, OperationKind, SharedMemPerCC` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `DataType, DataTypeSize, MathOperation, OperationKind, SharedMemPerCC`，以便后续代码在模块级复用共享定义。

### Line 47 — Import `cutlass_cppgen`

```python
import cutlass_cppgen
```
**EN:** Imports `cutlass_cppgen` so later code can use these APIs at module scope.
**CN:** 导入 `cutlass_cppgen`，供后续代码在模块级使用这些 API。

### Line 48 — From `cutlass_cppgen` import

```python
from cutlass_cppgen import get_option_registry
```
**EN:** Imports `get_option_registry` from `cutlass_cppgen` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen` 导入 `get_option_registry`，以便后续代码在模块级复用共享定义。

### Line 49 — From `cutlass_cppgen.backend.evt` import

```python
from cutlass_cppgen.backend.evt import EpilogueFunctorVisitor
```
**EN:** Imports `EpilogueFunctorVisitor` from `cutlass_cppgen.backend.evt` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt` 导入 `EpilogueFunctorVisitor`，以便后续代码在模块级复用共享定义。

### Line 50 — From `cutlass_cppgen.backend.evt.passes.util` import

```python
from cutlass_cppgen.backend.evt.passes.util import cc_map
```
**EN:** Imports `cc_map` from `cutlass_cppgen.backend.evt.passes.util` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.passes.util` 导入 `cc_map`，以便后续代码在模块级复用共享定义。

### Line 51 — From `cutlass_cppgen.backend.utils.device` import

```python
from cutlass_cppgen.backend.utils.device import device_cc
```
**EN:** Imports `device_cc` from `cutlass_cppgen.backend.utils.device` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.utils.device` 导入 `device_cc`，以便后续代码在模块级复用共享定义。

### Line 52 — From `cutlass_cppgen.epilogue` import

```python
from cutlass_cppgen.epilogue import get_activations, get_activation_epilogue, identity
```
**EN:** Imports `get_activations, get_activation_epilogue, identity` from `cutlass_cppgen.epilogue` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.epilogue` 导入 `get_activations, get_activation_epilogue, identity`，以便后续代码在模块级复用共享定义。

### Line 53 — From `cutlass_cppgen.library_defaults` import

```python
from cutlass_cppgen.library_defaults import KernelsForDataType, _generator_ccs
```
**EN:** Imports `KernelsForDataType, _generator_ccs` from `cutlass_cppgen.library_defaults` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.library_defaults` 导入 `KernelsForDataType, _generator_ccs`，以便后续代码在模块级复用共享定义。

### Line 54 — From `cutlass_cppgen.swizzle` import

```python
from cutlass_cppgen.swizzle import get_swizzling_functors
```
**EN:** Imports `get_swizzling_functors` from `cutlass_cppgen.swizzle` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.swizzle` 导入 `get_swizzling_functors`，以便后续代码在模块级复用共享定义。

### Line 55 — From `cutlass_cppgen.utils` import

```python
from cutlass_cppgen.utils import datatypes, check
```
**EN:** Imports `datatypes, check` from `cutlass_cppgen.utils` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.utils` 导入 `datatypes, check`，以便后续代码在模块级复用共享定义。

### Lines 58-61 — Class `OperationBase`

```python
class OperationBase:
    """
    Base operation used for defining high-level CUTLASS operations (e.g., GEMM, Conv2d)
    """
```
**EN:** Declares class `OperationBase` deriving from `object`. Purpose: Base operation used for defining high-level CUTLASS operations (e.g., GEMM, Conv2d)
**CN:** 声明类 `OperationBase`，其基类为 `object`。 其用途：Base operation used for defining high-level CUTLASS operations (e.g., GEMM, Conv2d)

#### Lines 63-71 — Function `__init__`

```python
    def __init__(self, cc: int = None, kernel_cc: int = None, operation_kind = OperationKind.Gemm):
        """
        :param cc: compute capability of device for which kernels should be compiled. For example, if running on H100, this should be set to 90
        :type cc: int
        :param kernel_cc: compute capability of kernels to generate. For example, if running on SM90, but desiring to use a CUTLASS 2.x-style Ampere kernel, this should be set to 80
        :type kernel_cc: int
        :param operation_kind: class of operation that will be performed (e.g., GEMM, Conv)
        :type operation_kind: cutlass_library.OperationKind
        """
```
**EN:** Defines function `__init__` with parameters `self, cc, kernel_cc, operation_kind`. Purpose: :param cc: compute capability of device for which kernels should be compiled.
**CN:** 定义函数 `__init__`，参数为 `self, cc, kernel_cc, operation_kind`。 其用途：:param cc: compute capability of device for which kernels should be compiled.

##### Line 72 — Assign `self.operation_kind`

```python
        self.operation_kind = operation_kind
```
**EN:** Assigns `self.operation_kind` from `operation_kind`, establishing state in function `__init__`.
**CN:** 将 `self.operation_kind` 赋值为 `operation_kind`，用于在函数 `__init__` 中建立状态。

##### Line 73 — Assign `self.cc`

```python
        self.cc = cc if cc is not None else device_cc()
```
**EN:** Assigns `self.cc` from `cc if cc is not None else device_cc()`, establishing state in function `__init__`.
**CN:** 将 `self.cc` 赋值为 `cc if cc is not None else device_cc()`，用于在函数 `__init__` 中建立状态。

##### Line 74 — Assign `self.specified_kernel_cc`

```python
        self.specified_kernel_cc = kernel_cc is not None
```
**EN:** Assigns `self.specified_kernel_cc` from `kernel_cc is not None`, establishing state in function `__init__`.
**CN:** 将 `self.specified_kernel_cc` 赋值为 `kernel_cc is not None`，用于在函数 `__init__` 中建立状态。

##### Line 75 — Assign `self.current_cc`

```python
        self.current_cc = kernel_cc if kernel_cc is not None else self._find_closest_cc(self.cc)
```
**EN:** Assigns `self.current_cc` from `kernel_cc if kernel_cc is not None else self._find_closest_cc(self.cc)`, establishing state in function `__init__`.
**CN:** 将 `self.current_cc` 赋值为 `kernel_cc if kernel_cc is not None else self._find_closest_cc(self.cc)`，用于在函数 `__init__` 中建立状态。

##### Line 76 — Assign `self.tile_description`

```python
        self.tile_description = None
```
**EN:** Assigns `self.tile_description` from `None`, establishing state in function `__init__`.
**CN:** 将 `self.tile_description` 赋值为 `None`，用于在函数 `__init__` 中建立状态。

##### Line 77 — Assign `self._math_operation`

```python
        self._math_operation = None
```
**EN:** Assigns `self._math_operation` from `None`, establishing state in function `__init__`.
**CN:** 将 `self._math_operation` 赋值为 `None`，用于在函数 `__init__` 中建立状态。

##### Line 79 — Assign `self.options`

```python
        self.options = get_option_registry().options_for_cc(self.current_cc, operation_kind)
```
**EN:** Assigns `self.options` from `get_option_registry().options_for_cc(self.current_cc, operation_kind)`, establishing state in function `__init__`.
**CN:** 将 `self.options` 赋值为 `get_option_registry().options_for_cc(self.current_cc, operation_kind)`，用于在函数 `__init__` 中建立状态。

##### Lines 81-82 — Conditional `self.options is None`

```python
        if self.options is None:
            raise Exception(f"Invalid or unsupported compute capability: {self.current_cc}")
```
**EN:** Checks `self.options is None` and selects the matching branch in function `__init__`.
**CN:** 检查 `self.options is None`，并在函数 `__init__` 中选择匹配的分支。

##### Line 84 — Comment or spacing block

```python
        # Default activation function: identity
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 85 — Assign `self._activation`

```python
        self._activation = identity
```
**EN:** Assigns `self._activation` from `identity`, establishing state in function `__init__`.
**CN:** 将 `self._activation` 赋值为 `identity`，用于在函数 `__init__` 中建立状态。

#### Lines 87-96 — Function `_find_closest_cc`

```python
    def _find_closest_cc(self, cc: int) -> int:
        """
        Returns the closest CC in _generator_ccs less than or equal to `cc`

        :param cc: compute capability to query
        :type cc: int

        :returns: closest CC in _generator_ccs less than or equal to `cc`
        :rtype: int
        """
```
**EN:** Defines function `_find_closest_cc` with parameters `self, cc`. Purpose: Returns the closest CC in _generator_ccs less than or equal to `cc`
**CN:** 定义函数 `_find_closest_cc`，参数为 `self, cc`。 其用途：Returns the closest CC in _generator_ccs less than or equal to `cc`

##### Lines 97-98 — Conditional `cc in _generator_ccs`

```python
        if cc in _generator_ccs:
            return cc
```
**EN:** Checks `cc in _generator_ccs` and selects the matching branch in function `_find_closest_cc`.
**CN:** 检查 `cc in _generator_ccs`，并在函数 `_find_closest_cc` 中选择匹配的分支。

##### Line 100 — Comment or spacing block

```python
        # Find closest CC lower than this CC
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 101 — Assign `idx`

```python
        idx = bisect_left(_generator_ccs, cc)
```
**EN:** Assigns `idx` from `bisect_left(_generator_ccs, cc)`, establishing state in function `_find_closest_cc`.
**CN:** 将 `idx` 赋值为 `bisect_left(_generator_ccs, cc)`，用于在函数 `_find_closest_cc` 中建立状态。

##### Lines 102-103 — Conditional `idx == 0`

```python
        if idx == 0:
            raise Exception(f'No valid CC to fall back to for {cc}')
```
**EN:** Checks `idx == 0` and selects the matching branch in function `_find_closest_cc`.
**CN:** 检查 `idx == 0`，并在函数 `_find_closest_cc` 中选择匹配的分支。

##### Line 104 — Return

```python
        return _generator_ccs[idx-1]
```
**EN:** Returns `_generator_ccs[idx - 1]` to the caller.
**CN:** 向调用方返回 `_generator_ccs[idx - 1]`。

#### Lines 106-112 — Function `activations`

```python
    def activations(self) -> list:
        """
        Returns possible activation functions that can be used

        :return: list of activation functions that can be used
        :rtype: list
        """
```
**EN:** Defines function `activations` with parameters `self`. Purpose: Returns possible activation functions that can be used
**CN:** 定义函数 `activations`，参数为 `self`。 其用途：Returns possible activation functions that can be used

##### Line 113 — Return

```python
        return get_activations()
```
**EN:** Returns `get_activations()` to the caller.
**CN:** 向调用方返回 `get_activations()`。

#### Lines 115-121 — Function `swizzling_functors`

```python
    def swizzling_functors(self) -> list:
        """
        Returns possible swizzling functions that can be used

        :return: list of swizzling functions that can be used
        :rtype: list
        """
```
**EN:** Defines function `swizzling_functors` with parameters `self`. Purpose: Returns possible swizzling functions that can be used
**CN:** 定义函数 `swizzling_functors`，参数为 `self`。 其用途：Returns possible swizzling functions that can be used

##### Line 122 — Return

```python
        return get_swizzling_functors()
```
**EN:** Returns `get_swizzling_functors()` to the caller.
**CN:** 向调用方返回 `get_swizzling_functors()`。

#### Lines 124-130 — Function `_reset_options`

```python
    def _reset_options(self, cc: int):
        """
        Resets the kernel options based on cc

        :param cc: compute capability to reset to
        :type cc: int
        """
```
**EN:** Defines function `_reset_options` with parameters `self, cc`. Purpose: Resets the kernel options based on cc
**CN:** 定义函数 `_reset_options`，参数为 `self, cc`。 其用途：Resets the kernel options based on cc

##### Lines 131-135 — Conditional `cc != self.current_cc`

```python
        if cc != self.current_cc:
            if cc not in _generator_ccs:
                raise Exception(f'Invalid CC for CUTLASS kernels: {cc}.')
            self.current_cc = cc
            self.options = get_option_registry().options_for_cc(self.current_cc, self.operation_kind)
```
**EN:** Checks `cc != self.current_cc` and selects the matching branch in function `_reset_options`.
**CN:** 检查 `cc != self.current_cc`，并在函数 `_reset_options` 中选择匹配的分支。

#### Lines 137-157 — Function `_verify_scalar`

```python
    def _verify_scalar(self, scalar, ref_scalar, ref_dtype, name):
        """
        Verifies the following properties:
            1) Either ``scalar`` or ``ref_scakar`` must be set (i.e., not ``None``)
            2) If ``scalar`` is not ``None``, its datatype must match matches the current version
               set by the plan (i.e., those in ``ref_dtype``)

        If either of these properties does not hold, an exception is raised. If these properties hold and
        ``scalar`` is not ``None``, ``scalar`` is returned. Otherwise, ``ref_scalar`` is returned.

        :param scalar: object representing a tensor passed in to verify, or ``None`` if no tensor was passed in
        :type scalar: numpy/cupy/torch scalar
        :param ref_scalar: object representing a tensor passed in on construction of this object, or ``None`` if no tensor was passed in
        :type ref_scalar: numpy/cupy/torch scalar
        :param ref_dtype: data type for the scalar that this object was initialized to
        :param name: identifier of the scalar to verify. Used in raising exceptions
        :type name: str

        :return: valid scalar to use
        :rtype: numpy/cupy/torch scalar
        """
```
**EN:** Defines function `_verify_scalar` with parameters `self, scalar, ref_scalar, ref_dtype, name`. Purpose: Verifies the following properties:
**CN:** 定义函数 `_verify_scalar`，参数为 `self, scalar, ref_scalar, ref_dtype, name`。 其用途：Verifies the following properties:

##### Lines 158-161 — Conditional `scalar is None`

```python
        if scalar is None:
            if ref_scalar is None:
                raise Exception(f"Scalar {name} must be set.")
            return ref_scalar
```
**EN:** Checks `scalar is None` and selects the matching branch in function `_verify_scalar`.
**CN:** 检查 `scalar is None`，并在函数 `_verify_scalar` 中选择匹配的分支。

##### Lines 162-167 — Conditional `hasattr(scalar, 'dtype')`

```python
        if hasattr(scalar, "dtype"):
            dtype = datatypes.library_type(scalar.dtype)
            if dtype != ref_dtype:
                raise Exception(
                    f"Tensor {name} with type {dtype} does not match expected type {ref_dtype}."
                )
```
**EN:** Checks `hasattr(scalar, 'dtype')` and selects the matching branch in function `_verify_scalar`.
**CN:** 检查 `hasattr(scalar, 'dtype')`，并在函数 `_verify_scalar` 中选择匹配的分支。

##### Line 168 — Return

```python
        return scalar
```
**EN:** Returns `scalar` to the caller.
**CN:** 向调用方返回 `scalar`。

#### Lines 170-194 — Function `_verify_tensor`

```python
    def _verify_tensor(self, tensor, ref_tensor, ref_dtype, ref_layout, name):
        """
        Verifies the following properties:
            If ref_dtype is not void:
                1) Either ``tensor`` or ``ref_tensor`` must be set (i.e., not ``None``)
                2) If ``tensor`` is not ``None``, its datatype and layout must match matches the current versions
                set by the plan (i.e., those in ``ref_dtype`` and ``ref_layout``)
            If ref_dtype is void:
                Neither ``tensor`` nor ``ref_tensor`` are set

        If either of these properties does not hold, an exception is raised. If these properties hold and
        ``tensor`` is not ``None``, ``tensor`` is returned. Otherwise, ``ref_tensor`` is returned.

        :param tensor: object representing a tensor passed in to verify, or ``None`` if no tensor was passed in
        :type tensor: numpy/cupy/torch array/tensor object
        :param ref_tensor: object representing a tensor passed in on construction of this object, or ``None`` if no tensor was passed in
        :type ref_tensor: numpy/cupy/torch array/tensor object
        :param ref_dtype: data type for the tensor that this object was initialized to
        :param ref_layout: layout for the tensor that this object was initialized to
        :param name: identifier of the tensor to verify. Used in raising exceptions
        :type name: str

        :return: valid tensor object to use
        :rtype: numpy/cupy/torch array/tensor object
        """
```
**EN:** Defines function `_verify_tensor` with parameters `self, tensor, ref_tensor, ref_dtype, ref_layout, name`. Purpose: Verifies the following properties:
**CN:** 定义函数 `_verify_tensor`，参数为 `self, tensor, ref_tensor, ref_dtype, ref_layout, name`。 其用途：Verifies the following properties:

##### Lines 195-198 — Conditional `ref_dtype == DataType.void`

```python
        if ref_dtype == DataType.void:
            if tensor is not None or ref_tensor is not None:
                raise Exception("Operands with element DataType.void must not be provided a tensor")
            return None
```
**EN:** Checks `ref_dtype == DataType.void` and selects the matching branch in function `_verify_tensor`.
**CN:** 检查 `ref_dtype == DataType.void`，并在函数 `_verify_tensor` 中选择匹配的分支。

##### Lines 200-203 — Conditional `tensor is None`

```python
        if tensor is None:
            if ref_tensor is None:
                raise Exception(f"Tensor {name} must be set.")
            return ref_tensor
```
**EN:** Checks `tensor is None` and selects the matching branch in function `_verify_tensor`.
**CN:** 检查 `tensor is None`，并在函数 `_verify_tensor` 中选择匹配的分支。

##### Line 205 — Call `self._verify_type_and_layout`

```python
        self._verify_type_and_layout(tensor, ref_dtype, ref_layout, name)
```
**EN:** Calls `self._verify_type_and_layout` for side effects or initialization work in function `_verify_tensor`.
**CN:** 调用 `self._verify_type_and_layout` 执行副作用或初始化逻辑；该语句位于在函数 `_verify_tensor` 中。

##### Line 206 — Return

```python
        return tensor
```
**EN:** Returns `tensor` to the caller.
**CN:** 向调用方返回 `tensor`。

#### Lines 208-215 — Function `opclass`

```python
    @property
    def opclass(self) -> cutlass_cppgen.OpcodeClass:
        """
        Returns the opcode class currently in use

        :return: opcode class currently in use
        :rtype: cutlass_cppgen.OpcodeClass
        """
```
**EN:** Defines function `opclass` with parameters `self`. Purpose: Returns the opcode class currently in use
**CN:** 定义函数 `opclass`，参数为 `self`。 其用途：Returns the opcode class currently in use

##### Line 216 — Return

```python
        return self.op_class
```
**EN:** Returns `self.op_class` to the caller.
**CN:** 向调用方返回 `self.op_class`。

#### Lines 218-219 — Function `opclass`

```python
    @opclass.setter
    def opclass(self, oc: cutlass_cppgen.OpcodeClass):
```
**EN:** Defines function `opclass` with parameters `self, oc`.
**CN:** 定义函数 `opclass`，参数为 `self, oc`。

##### Lines 220-221 — Conditional `isinstance(oc, str)`

```python
        if isinstance(oc, str):
            oc = datatypes.getattr_enum(cutlass_cppgen.OpcodeClass, oc)
```
**EN:** Checks `isinstance(oc, str)` and selects the matching branch in function `opclass`.
**CN:** 检查 `isinstance(oc, str)`，并在函数 `opclass` 中选择匹配的分支。

##### Lines 222-228 — Conditional `oc in self.possible_op_classes`

```python
        if oc in self.possible_op_classes:
            self.op_class = oc
        else:
            raise Exception(
                f'Unsupported operation class {oc} for CC {self.cc} and data type combination '
                f'({self._element_a}, {self._element_b}, {self._element_accumulator}) and '
                f'layout combination ({self._layout_a}, {self._layout_b}).')
```
**EN:** Checks `oc in self.possible_op_classes` and selects the matching branch in function `opclass`.
**CN:** 检查 `oc in self.possible_op_classes`，并在函数 `opclass` 中选择匹配的分支。

##### Line 230 — Comment or spacing block

```python
        # Changing the op class also changes the possible operations available. Reset these.
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 231-233 — Assign `self.possible_operations`

```python
        self.possible_operations = self.options.operations(
            self.op_class, self._element_a, self._element_b,
            self._element_accumulator, self._layout_a, self._layout_b, self._math_operation)
```
**EN:** Assigns `self.possible_operations` from `self.options.operations(self.op_class, self._element_a, self._element_b, self._element_accumulator, self._layout_a, self._layout_b, self._math_operation)`, establishing state in function `opclass`.
**CN:** 将 `self.possible_operations` 赋值为 `self.options.operations(self.op_class, self._element_a, self._element_b, self._element_accumulator, self._layout_a, self._layout_b, self._math_operation)`，用于在函数 `opclass` 中建立状态。

##### Line 235 — Comment or spacing block

```python
        # Changing the op class changes the elements per access in the epilogue. Reset this.
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 236-237 — Conditional `self.epilogue_functor is not None`

```python
        if self.epilogue_functor is not None:
            self.epilogue_functor = self._reset_epilogue_functor_alignment(self._elements_per_access(), self.epilogue_functor)
```
**EN:** Checks `self.epilogue_functor is not None` and selects the matching branch in function `opclass`.
**CN:** 检查 `self.epilogue_functor is not None`，并在函数 `opclass` 中选择匹配的分支。

#### Lines 239-246 — Function `math_operation`

```python
    @property
    def math_operation(self) -> cutlass_cppgen.MathOperation:
        """
        Returns the math operation currently in use

        :return: math operation currently in use
        :rtype: cutlass_cppgen.MathOperation
        """
```
**EN:** Defines function `math_operation` with parameters `self`. Purpose: Returns the math operation currently in use
**CN:** 定义函数 `math_operation`，参数为 `self`。 其用途：Returns the math operation currently in use

##### Line 247 — Return

```python
        return self._math_operation
```
**EN:** Returns `self._math_operation` to the caller.
**CN:** 向调用方返回 `self._math_operation`。

#### Lines 249-250 — Function `math_operation`

```python
    @math_operation.setter
    def math_operation(self, mo: cutlass_cppgen.MathOperation):
```
**EN:** Defines function `math_operation` with parameters `self, mo`.
**CN:** 定义函数 `math_operation`，参数为 `self, mo`。

##### Lines 251-252 — Conditional `isinstance(mo, str)`

```python
        if isinstance(mo, str):
            mo = datatypes.getattr_enum(cutlass_cppgen.MathOperation, mo)
```
**EN:** Checks `isinstance(mo, str)` and selects the matching branch in function `math_operation`.
**CN:** 检查 `isinstance(mo, str)`，并在函数 `math_operation` 中选择匹配的分支。

##### Lines 254-264 — Conditional `not self.specified_kernel_cc`

```python
        if not self.specified_kernel_cc:
            if self.current_cc in [90, 100, 101, 103]:
                # CUTLASS 3.0 kernels do not use different math operations. If one is specified, we
                # revert to using a CUTLASS 2.x kernel by using SM80-tagged kernels.
                cutlass_cppgen.logger.warning("Reverting to using SM80-tagged kernel. Opclass may change.")
                self._reset_options(80)
                self._reset_operations(reset_epilogue=False)
        elif self.current_cc in [90, 100, 101, 103]:
            raise Exception("CUTLASS 3.0 kernels do not use different math operations. "
                "To use 2.x kernels with a specific math operation, do not set the `kernel_cc`"
                "parameter when constructing the plan.")
```
**EN:** Checks `not self.specified_kernel_cc` and selects the matching branch in function `math_operation`.
**CN:** 检查 `not self.specified_kernel_cc`，并在函数 `math_operation` 中选择匹配的分支。

##### Line 266 — Assign `self._math_operation`

```python
        self._math_operation = mo
```
**EN:** Assigns `self._math_operation` from `mo`, establishing state in function `math_operation`.
**CN:** 将 `self._math_operation` 赋值为 `mo`，用于在函数 `math_operation` 中建立状态。

##### Line 267 — Call `self._reset_operations`

```python
        self._reset_operations()
```
**EN:** Calls `self._reset_operations` for side effects or initialization work in function `math_operation`.
**CN:** 调用 `self._reset_operations` 执行副作用或初始化逻辑；该语句位于在函数 `math_operation` 中。

#### Line 269 — Function `_elements_per_access`

```python
    def _elements_per_access(self):
```
**EN:** Defines function `_elements_per_access` with parameters `self`.
**CN:** 定义函数 `_elements_per_access`，参数为 `self`。

##### Lines 270-275 — Conditional `self.op_class == cutlass_cppgen.OpcodeClass.Simt`

```python
        if self.op_class == cutlass_cppgen.OpcodeClass.Simt:
            return 1
        elif self._element_c != DataType.void:
            return 128 // DataTypeSize[self._element_c]
        else:
            return 128 // max(self.possible_operations.alignments("C"))
```
**EN:** Checks `self.op_class == cutlass_cppgen.OpcodeClass.Simt` and selects the matching branch in function `_elements_per_access`.
**CN:** 检查 `self.op_class == cutlass_cppgen.OpcodeClass.Simt`，并在函数 `_elements_per_access` 中选择匹配的分支。

#### Lines 277-280 — Function `_create_epilogue_functor_activation`

```python
    def _create_epilogue_functor_activation(self, activation):
        """
        Returns the epilogue functor with given activation function
        """
```
**EN:** Defines function `_create_epilogue_functor_activation` with parameters `self, activation`. Purpose: Returns the epilogue functor with given activation function
**CN:** 定义函数 `_create_epilogue_functor_activation`，参数为 `self, activation`。 其用途：Returns the epilogue functor with given activation function

##### Lines 281-284 — Conditional `self.epilogue_functor is None`

```python
        if self.epilogue_functor is None:
            elements_per_access = self._elements_per_access()
        else:
            elements_per_access = self.epilogue_functor.epilogue_vector_length
```
**EN:** Checks `self.epilogue_functor is None` and selects the matching branch in function `_create_epilogue_functor_activation`.
**CN:** 检查 `self.epilogue_functor is None`，并在函数 `_create_epilogue_functor_activation` 中选择匹配的分支。

##### Lines 286-305 — Conditional `not self.specified_kernel_cc`

```python
        if not self.specified_kernel_cc:
            if self.current_cc in [90, 100, 101, 103] and activation != identity:
                # CUTLASS 3.0 kernels in Python currently only support identity activation. If one requests a non-identity activation,
                # revert to using a CUTLASS 2.x kernel by using SM80-tagged kernels.
                cutlass_cppgen.logger.warning("Reverting to using SM80-tagged kernel. Opclass may change.")
                if self._element_c != self._element_d:
                    raise Exception("CUTLASS 2.x kernels require element C to be the same as element D")
                self._reset_options(80)
                self._reset_operations(reset_epilogue=False)
            elif (self.cc in [90, 100, 101, 103] and self.current_cc not in [90, 100, 101, 103] and activation == identity and self._math_operation is None):
                # SM80 fallback kernels are currently used. Since an identity activation is requested,
                # we can switch back to using SM90 kernels.
                self._reset_options(self.cc)
                self._reset_operations(reset_epilogue=False)
        else:
            if self.current_cc in [90, 100, 101, 103] and activation != identity:
                raise Exception("Epilogues with elementwise fusion are not currently supported "
                                "in the Python interface for 3.x kernels. To use 2.x kernels "
                                "with fused elementwise epilogues, do not set the `kernel_cc` "
                                "parameter when constructing the plan.")
```
**EN:** Checks `not self.specified_kernel_cc` and selects the matching branch in function `_create_epilogue_functor_activation`.
**CN:** 检查 `not self.specified_kernel_cc`，并在函数 `_create_epilogue_functor_activation` 中选择匹配的分支。

##### Lines 307-313 — Return

```python
        return get_activation_epilogue(
            activation,
            self._element_d,
            elements_per_access,
            self._element_accumulator,
            self._element_accumulator,
        )
```
**EN:** Returns `get_activation_epilogue(activation, self._element_d, elements_per_access, self._element_accumulator, self._element_accumulator)` to the caller.
**CN:** 向调用方返回 `get_activation_epilogue(activation, self._element_d, elements_per_access, self._element_accumulator, self._element_accumulator)`。

#### Lines 315-318 — Function `_reset_epilogue_functor_activation`

```python
    def _reset_epilogue_functor_activation(self, activation):
        """
        Set the epilogue functor based on the provided activation function
        """
```
**EN:** Defines function `_reset_epilogue_functor_activation` with parameters `self, activation`. Purpose: Set the epilogue functor based on the provided activation function
**CN:** 定义函数 `_reset_epilogue_functor_activation`，参数为 `self, activation`。 其用途：Set the epilogue functor based on the provided activation function

##### Line 319 — Assign `self.epilogue_functor`

```python
        self.epilogue_functor = self._create_epilogue_functor_activation(activation)
```
**EN:** Assigns `self.epilogue_functor` from `self._create_epilogue_functor_activation(activation)`, establishing state in function `_reset_epilogue_functor_activation`.
**CN:** 将 `self.epilogue_functor` 赋值为 `self._create_epilogue_functor_activation(activation)`，用于在函数 `_reset_epilogue_functor_activation` 中建立状态。

#### Lines 321-324 — Function `_reset_epilogue_functor_alignment`

```python
    def _reset_epilogue_functor_alignment(self, alignment, epilogue_functor):
        """
        Reset the alignment of the current epilogue functor based on alignment C
        """
```
**EN:** Defines function `_reset_epilogue_functor_alignment` with parameters `self, alignment, epilogue_functor`. Purpose: Reset the alignment of the current epilogue functor based on alignment C
**CN:** 定义函数 `_reset_epilogue_functor_alignment`，参数为 `self, alignment, epilogue_functor`。 其用途：Reset the alignment of the current epilogue functor based on alignment C

##### Lines 325-326 — Conditional `isinstance(epilogue_functor, EpilogueFunctorVisitor)`

```python
        if isinstance(epilogue_functor, EpilogueFunctorVisitor):
            return epilogue_functor
```
**EN:** Checks `isinstance(epilogue_functor, EpilogueFunctorVisitor)` and selects the matching branch in function `_reset_epilogue_functor_alignment`.
**CN:** 检查 `isinstance(epilogue_functor, EpilogueFunctorVisitor)`，并在函数 `_reset_epilogue_functor_alignment` 中选择匹配的分支。

##### Lines 328-332 — Conditional `epilogue_functor is None or not hasattr(epilogue_functor, 'activation_functor')`

```python
        if epilogue_functor is None or not hasattr(epilogue_functor, 'activation_functor'):
            # Identity epilogue does not have 'activation_functor'
            activation = identity
        else:
            activation = epilogue_functor.activation_functor
```
**EN:** Checks `epilogue_functor is None or not hasattr(epilogue_functor, 'activation_functor')` and selects the matching branch in function `_reset_epilogue_functor_alignment`.
**CN:** 检查 `epilogue_functor is None or not hasattr(epilogue_functor, 'activation_functor')`，并在函数 `_reset_epilogue_functor_alignment` 中选择匹配的分支。

##### Lines 334-340 — Assign `epilogue_functor`

```python
        epilogue_functor = get_activation_epilogue(
            activation,
            self._element_d,
            alignment,
            self._element_accumulator,
            self._element_accumulator,
        )
```
**EN:** Assigns `epilogue_functor` from `get_activation_epilogue(activation, self._element_d, alignment, self._element_accumulator, self._element_accumulator)`, establishing state in function `_reset_epilogue_functor_alignment`.
**CN:** 将 `epilogue_functor` 赋值为 `get_activation_epilogue(activation, self._element_d, alignment, self._element_accumulator, self._element_accumulator)`，用于在函数 `_reset_epilogue_functor_alignment` 中建立状态。

##### Line 341 — Return

```python
        return epilogue_functor
```
**EN:** Returns `epilogue_functor` to the caller.
**CN:** 向调用方返回 `epilogue_functor`。

#### Lines 343-347 — Function `activation`

```python
    @property
    def activation(self):
        """
        Returns the type of the current activation function used
        """
```
**EN:** Defines function `activation` with parameters `self`. Purpose: Returns the type of the current activation function used
**CN:** 定义函数 `activation`，参数为 `self`。 其用途：Returns the type of the current activation function used

##### Lines 348-351 — Conditional `hasattr(self.epilogue_functor, 'activation_functor')`

```python
        if hasattr(self.epilogue_functor, "activation_functor"):
            return self.epilogue_functor.activation_functor
        else:
            return identity
```
**EN:** Checks `hasattr(self.epilogue_functor, 'activation_functor')` and selects the matching branch in function `activation`.
**CN:** 检查 `hasattr(self.epilogue_functor, 'activation_functor')`，并在函数 `activation` 中选择匹配的分支。

#### Lines 353-362 — Function `activation`

```python
    @activation.setter
    def activation(self, act):
        """
        Sets the type of the activation function to use
        Activation can come with a set of arguments

        :param act: type of activation function to use
        :type act: str or tuple. e.g. "relu", ("leaky_relu", 0.01)

        """
```
**EN:** Defines function `activation` with parameters `self, act`. Purpose: Sets the type of the activation function to use
**CN:** 定义函数 `activation`，参数为 `self, act`。 其用途：Sets the type of the activation function to use

##### Lines 363-375 — Conditional `isinstance(act, tuple)`

```python
        if isinstance(act, tuple):
            if isinstance(act[0], str):
                act_fn = getattr(cutlass_cppgen.backend.epilogue, act[0])
            else:
                act_fn = act[0]
            self._reset_epilogue_functor_activation(act_fn)
            self._activation_args = act[1]
            self._activation = act[0]
        else:
            if isinstance(act, str):
                act = getattr(cutlass_cppgen.backend.epilogue, act)
            self._reset_epilogue_functor_activation(act)
            self._activation = act
```
**EN:** Checks `isinstance(act, tuple)` and selects the matching branch in function `activation`.
**CN:** 检查 `isinstance(act, tuple)`，并在函数 `activation` 中选择匹配的分支。

#### Lines 377-381 — Function `epilogue_visitor`

```python
    @property
    def epilogue_visitor(self):
        """
        Return the epilogue functor
        """
```
**EN:** Defines function `epilogue_visitor` with parameters `self`. Purpose: Return the epilogue functor
**CN:** 定义函数 `epilogue_visitor`，参数为 `self`。 其用途：Return the epilogue functor

##### Line 382 — Return

```python
        return self.epilogue_functor
```
**EN:** Returns `self.epilogue_functor` to the caller.
**CN:** 向调用方返回 `self.epilogue_functor`。

#### Lines 384-388 — Function `epilogue_visitor`

```python
    @epilogue_visitor.setter
    def epilogue_visitor(self, visitor):
        """
        Create the epilogue visitor
        """
```
**EN:** Defines function `epilogue_visitor` with parameters `self, visitor`. Purpose: Create the epilogue visitor
**CN:** 定义函数 `epilogue_visitor`，参数为 `self, visitor`。 其用途：Create the epilogue visitor

##### Line 389 — Assign `self.epilogue_functor`

```python
        self.epilogue_functor = EpilogueFunctorVisitor(cc_map[self.cc], visitor)
```
**EN:** Assigns `self.epilogue_functor` from `EpilogueFunctorVisitor(cc_map[self.cc], visitor)`, establishing state in function `epilogue_visitor`.
**CN:** 将 `self.epilogue_functor` 赋值为 `EpilogueFunctorVisitor(cc_map[self.cc], visitor)`，用于在函数 `epilogue_visitor` 中建立状态。

##### Lines 391-392 — Comment or spacing block

```python
        # The epilogue_functor may consume too much shared memory
        # Reset the possible operations
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 393-396 — Conditional `self.cc not in [90, 100, 101, 103]`

```python
        if self.cc not in [90, 100, 101, 103]:
            # The shared memory is only a concern for sm90+ epilogue
            # In sm80, the epilogue and mainloop share the shared memory
            return
```
**EN:** Checks `self.cc not in [90, 100, 101, 103]` and selects the matching branch in function `epilogue_visitor`.
**CN:** 检查 `self.cc not in [90, 100, 101, 103]`，并在函数 `epilogue_visitor` 中选择匹配的分支。

##### Line 398 — Assign `datatype_comb`

```python
        datatype_comb = self.possible_operations.datatype_comb
```
**EN:** Assigns `datatype_comb` from `self.possible_operations.datatype_comb`, establishing state in function `epilogue_visitor`.
**CN:** 将 `datatype_comb` 赋值为 `self.possible_operations.datatype_comb`，用于在函数 `epilogue_visitor` 中建立状态。

##### Line 399 — Assign `layout_comb`

```python
        layout_comb = self.possible_operations.layout_comb
```
**EN:** Assigns `layout_comb` from `self.possible_operations.layout_comb`, establishing state in function `epilogue_visitor`.
**CN:** 将 `layout_comb` 赋值为 `self.possible_operations.layout_comb`，用于在函数 `epilogue_visitor` 中建立状态。

##### Line 400 — Assign `new_possible_operations`

```python
        new_possible_operations = KernelsForDataType(datatype_comb, layout_comb)
```
**EN:** Assigns `new_possible_operations` from `KernelsForDataType(datatype_comb, layout_comb)`, establishing state in function `epilogue_visitor`.
**CN:** 将 `new_possible_operations` 赋值为 `KernelsForDataType(datatype_comb, layout_comb)`，用于在函数 `epilogue_visitor` 中建立状态。

##### Lines 401-418 — Loop over `self.possible_operations.all_operations`

```python
        for operation in self.possible_operations.all_operations:
            td = datatypes.td_from_profiler_op(operation)
            # Filter invalid epilogue schedules
            if cc_map[self.cc] == 90 and td.epilogue_schedule not in [
                cutlass_cppgen.EpilogueScheduleType.TmaWarpSpecialized,
                cutlass_cppgen.EpilogueScheduleType.TmaWarpSpecializedCooperative]:
                continue
            epilogue_smem_bytes = self.epilogue_functor.get_smem_size(td)

            # Verify the maximum number of mainloop stages
            mainloop_smem_per_stage = check.calculate_smem_usage_per_stage(td, OperationKind.Gemm)
            smem_capacity_bytes = SharedMemPerCC[self.cc] << 10
            mainloop_stages = (smem_capacity_bytes - epilogue_smem_bytes) // mainloop_smem_per_stage
            if mainloop_stages < 2:
                # Mainloop stages must >= 2
                continue

            new_possible_operations.add(operation)
```
**EN:** Iterates `operation` over `self.possible_operations.all_operations` to repeat a processing step.
**CN:** 让 `operation` 遍历 `self.possible_operations.all_operations`，从而重复执行处理步骤。

##### Lines 419-422 — Conditional `len(new_possible_operations.all_operations) == 0`

```python
        if len(new_possible_operations.all_operations) == 0:
            raise RuntimeError(
                "The epilogue consumes too much shared memory. "
                "No valid tile description is found in the generator.")
```
**EN:** Checks `len(new_possible_operations.all_operations) == 0` and selects the matching branch in function `epilogue_visitor`.
**CN:** 检查 `len(new_possible_operations.all_operations) == 0`，并在函数 `epilogue_visitor` 中选择匹配的分支。

##### Line 423 — Assign `self.possible_operations`

```python
        self.possible_operations = new_possible_operations
```
**EN:** Assigns `self.possible_operations` from `new_possible_operations`, establishing state in function `epilogue_visitor`.
**CN:** 将 `self.possible_operations` 赋值为 `new_possible_operations`，用于在函数 `epilogue_visitor` 中建立状态。

#### Lines 426-429 — Function `run_setup`

```python
    def run_setup(self):
        """
        Steps that must be taken before caling `plan.run()`
        """
```
**EN:** Defines function `run_setup` with parameters `self`. Purpose: Steps that must be taken before caling `plan.run()`
**CN:** 定义函数 `run_setup`，参数为 `self`。 其用途：Steps that must be taken before caling `plan.run()`

##### Line 430 — Comment or spacing block

```python
        # Initialize the memory pool if, if not already done
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 431 — Call `cutlass_cppgen.get_memory_pool`

```python
        cutlass_cppgen.get_memory_pool()
```
**EN:** Calls `cutlass_cppgen.get_memory_pool` for side effects or initialization work in function `run_setup`.
**CN:** 调用 `cutlass_cppgen.get_memory_pool` 执行副作用或初始化逻辑；该语句位于在函数 `run_setup` 中。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `OperationBase`.
- **CN:** 顶层类：`OperationBase`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module exposes high-level operation planning APIs that wrap kernel selection, compilation, and launch details.
- **CN:** 该模块暴露高层操作规划 API，对内封装内核选择、编译与启动细节。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen`, `cutlass_cppgen.backend.evt`, `cutlass_cppgen.backend.evt.passes.util`, `cutlass_cppgen.backend.utils.device`, `cutlass_cppgen.epilogue`, `cutlass_cppgen.library_defaults`, `cutlass_cppgen.swizzle`, `cutlass_cppgen.utils`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** `bisect`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
