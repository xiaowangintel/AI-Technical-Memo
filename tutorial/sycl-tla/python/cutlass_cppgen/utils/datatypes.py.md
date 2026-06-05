# datatypes.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/utils/datatypes.py`
- **EN:** Utility functions for converting between frontend datatypes and CUTLASS datatypes
- **CN:** 模块文档说明：Utility functions for converting between frontend datatypes and CUTLASS datatypes

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
Utility functions for converting between frontend datatypes and CUTLASS datatypes
"""
```
**EN:** Docstring explains this scope: Utility functions for converting between frontend datatypes and CUTLASS datatypes
**CN:** 文档字符串说明了该作用域的用途：Utility functions for converting between frontend datatypes and CUTLASS datatypes

### Line 37 — Import `cutlass_cppgen`

```python
import cutlass_cppgen
```
**EN:** Imports `cutlass_cppgen` so later code can use these APIs at module scope.
**CN:** 导入 `cutlass_cppgen`，供后续代码在模块级使用这些 API。

### Lines 38-42 — From `cutlass_library` import

```python
from cutlass_library import (
    DataTypeSize,
    MathOperation,
    MathInstruction
)
```
**EN:** Imports `DataTypeSize, MathOperation, MathInstruction` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `DataTypeSize, MathOperation, MathInstruction`，以便后续代码在模块级复用共享定义。

### Lines 43-45 — From `cutlass_cppgen.backend.library` import

```python
from cutlass_cppgen.backend.library import (
    TileDescription,
)
```
**EN:** Imports `TileDescription` from `cutlass_cppgen.backend.library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.library` 导入 `TileDescription`，以便后续代码在模块级复用共享定义。

### Line 47 — Assign `bfloat16_available`

```python
bfloat16_available = None
```
**EN:** Assigns `bfloat16_available` from `None`, establishing state at module scope.
**CN:** 将 `bfloat16_available` 赋值为 `None`，用于在模块级建立状态。

### Line 48 — Assign `cupy_available`

```python
cupy_available = None
```
**EN:** Assigns `cupy_available` from `None`, establishing state at module scope.
**CN:** 将 `cupy_available` 赋值为 `None`，用于在模块级建立状态。

### Line 49 — Assign `numpy_available`

```python
numpy_available = None
```
**EN:** Assigns `numpy_available` from `None`, establishing state at module scope.
**CN:** 将 `numpy_available` 赋值为 `None`，用于在模块级建立状态。

### Line 50 — Assign `torch_available`

```python
torch_available = None
```
**EN:** Assigns `torch_available` from `None`, establishing state at module scope.
**CN:** 将 `torch_available` 赋值为 `None`，用于在模块级建立状态。

### Line 51 — Assign `_library_to_cupy_dict`

```python
_library_to_cupy_dict = None
```
**EN:** Assigns `_library_to_cupy_dict` from `None`, establishing state at module scope.
**CN:** 将 `_library_to_cupy_dict` 赋值为 `None`，用于在模块级建立状态。

### Line 52 — Assign `_library_to_numpy_dict`

```python
_library_to_numpy_dict = None
```
**EN:** Assigns `_library_to_numpy_dict` from `None`, establishing state at module scope.
**CN:** 将 `_library_to_numpy_dict` 赋值为 `None`，用于在模块级建立状态。

### Line 53 — Assign `_library_to_torch_dict`

```python
_library_to_torch_dict = None
```
**EN:** Assigns `_library_to_torch_dict` from `None`, establishing state at module scope.
**CN:** 将 `_library_to_torch_dict` 赋值为 `None`，用于在模块级建立状态。

### Line 54 — Assign `_torch_to_library_dict`

```python
_torch_to_library_dict = None
```
**EN:** Assigns `_torch_to_library_dict` from `None`, establishing state at module scope.
**CN:** 将 `_torch_to_library_dict` 赋值为 `None`，用于在模块级建立状态。

### Line 57 — Function `is_numpy_available`

```python
def is_numpy_available():
```
**EN:** Defines function `is_numpy_available` with parameters ``.
**CN:** 定义函数 `is_numpy_available`，参数为 ``。

#### Line 58 — Global

```python
    global numpy_available, _library_to_numpy_dict
```
**EN:** Implements a `Global` statement in function `is_numpy_available`.
**CN:** 实现了一个 `Global` 语句；该语句位于在函数 `is_numpy_available` 中。

#### Lines 59-73 — Conditional `numpy_available is None`

```python
    if numpy_available is None:
        try:
            import numpy as np

            numpy_available = True
            _library_to_numpy_dict = {
                cutlass_cppgen.DataType.f16: np.float16,
                cutlass_cppgen.DataType.f32: np.float32,
                cutlass_cppgen.DataType.f64: np.float64,
                cutlass_cppgen.DataType.s8: np.int8,
                cutlass_cppgen.DataType.s32: np.int32,
            }
        except ImportError:
            numpy_available = False
            _library_to_numpy_dict = {}
```
**EN:** Checks `numpy_available is None` and selects the matching branch in function `is_numpy_available`.
**CN:** 检查 `numpy_available is None`，并在函数 `is_numpy_available` 中选择匹配的分支。

#### Line 74 — Return

```python
    return numpy_available
```
**EN:** Returns `numpy_available` to the caller.
**CN:** 向调用方返回 `numpy_available`。

### Line 77 — Function `is_numpy_tensor`

```python
def is_numpy_tensor(inp) -> bool:
```
**EN:** Defines function `is_numpy_tensor` with parameters `inp`.
**CN:** 定义函数 `is_numpy_tensor`，参数为 `inp`。

#### Lines 78-80 — Conditional `is_numpy_available()`

```python
    if is_numpy_available():
        import numpy as np
        return isinstance(inp, np.ndarray)
```
**EN:** Checks `is_numpy_available()` and selects the matching branch in function `is_numpy_tensor`.
**CN:** 检查 `is_numpy_available()`，并在函数 `is_numpy_tensor` 中选择匹配的分支。

#### Line 81 — Return

```python
    return False
```
**EN:** Returns `False` to the caller.
**CN:** 向调用方返回 `False`。

### Line 84 — Function `numpy_library_type`

```python
def numpy_library_type(inp) -> cutlass_cppgen.DataType:
```
**EN:** Defines function `numpy_library_type` with parameters `inp`.
**CN:** 定义函数 `numpy_library_type`，参数为 `inp`。

#### Lines 85-96 — Conditional `is_numpy_available()`

```python
    if is_numpy_available():
        import numpy as np
        if inp == np.float16:
            return cutlass_cppgen.DataType.f16
        elif inp == np.float32:
            return cutlass_cppgen.DataType.f32
        elif inp == np.float64:
            return cutlass_cppgen.DataType.f64
        elif inp == np.int8:
            return cutlass_cppgen.DataType.s8
        elif inp == np.int32:
            return cutlass_cppgen.DataType.s32
```
**EN:** Checks `is_numpy_available()` and selects the matching branch in function `numpy_library_type`.
**CN:** 检查 `is_numpy_available()`，并在函数 `numpy_library_type` 中选择匹配的分支。

#### Line 97 — Return

```python
    return None
```
**EN:** Returns `None` to the caller.
**CN:** 向调用方返回 `None`。

### Line 100 — Function `numpy_type`

```python
def numpy_type(inp):
```
**EN:** Defines function `numpy_type` with parameters `inp`.
**CN:** 定义函数 `numpy_type`，参数为 `inp`。

#### Line 101 — Return

```python
    return _library_to_numpy_dict.get(inp, None)
```
**EN:** Returns `_library_to_numpy_dict.get(inp, None)` to the caller.
**CN:** 向调用方返回 `_library_to_numpy_dict.get(inp, None)`。

### Line 104 — Function `is_cupy_available`

```python
def is_cupy_available():
```
**EN:** Defines function `is_cupy_available` with parameters ``.
**CN:** 定义函数 `is_cupy_available`，参数为 ``。

#### Line 105 — Global

```python
    global cupy_available
```
**EN:** Implements a `Global` statement in function `is_cupy_available`.
**CN:** 实现了一个 `Global` 语句；该语句位于在函数 `is_cupy_available` 中。

#### Lines 106-120 — Conditional `cupy_available is None`

```python
    if cupy_available is None:
        try:
            import cupy as cp

            cupy_available = True
            _library_to_cupy_dict = {
                cutlass_cppgen.DataType.f16: cp.float16,
                cutlass_cppgen.DataType.f32: cp.float32,
                cutlass_cppgen.DataType.f64: cp.float64,
                cutlass_cppgen.DataType.s8: cp.int8,
                cutlass_cppgen.DataType.s32: cp.int32,
            }
        except ImportError:
            cupy_available = False
            _library_to_cupy_dict = {}
```
**EN:** Checks `cupy_available is None` and selects the matching branch in function `is_cupy_available`.
**CN:** 检查 `cupy_available is None`，并在函数 `is_cupy_available` 中选择匹配的分支。

#### Line 121 — Return

```python
    return cupy_available
```
**EN:** Returns `cupy_available` to the caller.
**CN:** 向调用方返回 `cupy_available`。

### Line 124 — Function `is_cupy_tensor`

```python
def is_cupy_tensor(inp) -> bool:
```
**EN:** Defines function `is_cupy_tensor` with parameters `inp`.
**CN:** 定义函数 `is_cupy_tensor`，参数为 `inp`。

#### Lines 125-127 — Conditional `is_cupy_available()`

```python
    if is_cupy_available():
        import cupy as cp
        return isinstance(inp, cp.ndarray)
```
**EN:** Checks `is_cupy_available()` and selects the matching branch in function `is_cupy_tensor`.
**CN:** 检查 `is_cupy_available()`，并在函数 `is_cupy_tensor` 中选择匹配的分支。

#### Line 128 — Return

```python
    return False
```
**EN:** Returns `False` to the caller.
**CN:** 向调用方返回 `False`。

### Line 131 — Function `cupy_library_type`

```python
def cupy_library_type(inp) -> cutlass_cppgen.DataType:
```
**EN:** Defines function `cupy_library_type` with parameters `inp`.
**CN:** 定义函数 `cupy_library_type`，参数为 `inp`。

#### Lines 132-139 — Conditional `is_cupy_available()`

```python
    if is_cupy_available():
        import cupy as cp
        if inp == cp.float16:
            return cutlass_cppgen.DataType.f16
        elif inp == cp.float32:
            return cutlass_cppgen.DataType.f32
        elif inp == cp.float64:
            return cutlass_cppgen.DataType.f64
```
**EN:** Checks `is_cupy_available()` and selects the matching branch in function `cupy_library_type`.
**CN:** 检查 `is_cupy_available()`，并在函数 `cupy_library_type` 中选择匹配的分支。

#### Line 140 — Return

```python
    return None
```
**EN:** Returns `None` to the caller.
**CN:** 向调用方返回 `None`。

### Line 143 — Function `cupy_type`

```python
def cupy_type(inp):
```
**EN:** Defines function `cupy_type` with parameters `inp`.
**CN:** 定义函数 `cupy_type`，参数为 `inp`。

#### Line 144 — Return

```python
    return _library_to_cupy_dict.get(inp, None)
```
**EN:** Returns `_library_to_cupy_dict.get(inp, None)` to the caller.
**CN:** 向调用方返回 `_library_to_cupy_dict.get(inp, None)`。

### Line 147 — Function `is_torch_available`

```python
def is_torch_available():
```
**EN:** Defines function `is_torch_available` with parameters ``.
**CN:** 定义函数 `is_torch_available`，参数为 ``。

#### Line 148 — Global

```python
    global torch_available, _library_to_torch_dict, _torch_to_library_dict
```
**EN:** Implements a `Global` statement in function `is_torch_available`.
**CN:** 实现了一个 `Global` 语句；该语句位于在函数 `is_torch_available` 中。

#### Lines 149-193 — Conditional `torch_available is None`

```python
    if torch_available is None:
        try:
            import torch

            torch_available = True
            _torch_to_library_dict = {
                torch.half: cutlass_cppgen.DataType.f16,
                torch.float16: cutlass_cppgen.DataType.f16,
                torch.bfloat16: cutlass_cppgen.DataType.bf16,
                torch.float: cutlass_cppgen.DataType.f32,
                torch.float32: cutlass_cppgen.DataType.f32,
                torch.double: cutlass_cppgen.DataType.f64,
                torch.float64: cutlass_cppgen.DataType.f64,
                torch.int8: cutlass_cppgen.DataType.s8,
                torch.int32: cutlass_cppgen.DataType.s32,
                torch.uint8: cutlass_cppgen.DataType.u8,
            }

            _library_to_torch_dict = {
                cutlass_cppgen.DataType.f16: torch.half,
                cutlass_cppgen.DataType.f16: torch.float16,
                cutlass_cppgen.DataType.bf16: torch.bfloat16,
                cutlass_cppgen.DataType.f32: torch.float,
                cutlass_cppgen.DataType.f32: torch.float32,
                cutlass_cppgen.DataType.f64: torch.double,
                cutlass_cppgen.DataType.f64: torch.float64,
                cutlass_cppgen.DataType.s8: torch.int8,
                cutlass_cppgen.DataType.s32: torch.int32,
                cutlass_cppgen.DataType.u8: torch.uint8,
            }

            def possibly_add_type(torch_type_name, cutlass_type):
                # Only try adding the type if the version of torch being used supports it
                if hasattr(torch, torch_type_name):
                    torch_type = getattr(torch, torch_type_name)
                    _torch_to_library_dict[torch_type] = cutlass_type
                    _library_to_torch_dict[cutlass_type] = torch_type

            possibly_add_type("float8_e4m3fn", cutlass_cppgen.DataType.e4m3)
            possibly_add_type("float8_e5m2", cutlass_cppgen.DataType.e5m2)

        except ImportError:
            torch_available = False
            _torch_to_library_dict = {}
            _library_to_torch_dict = {}
```
**EN:** Checks `torch_available is None` and selects the matching branch in function `is_torch_available`.
**CN:** 检查 `torch_available is None`，并在函数 `is_torch_available` 中选择匹配的分支。

#### Line 194 — Return

```python
    return torch_available
```
**EN:** Returns `torch_available` to the caller.
**CN:** 向调用方返回 `torch_available`。

### Line 197 — Function `is_torch_tensor`

```python
def is_torch_tensor(inp) -> bool:
```
**EN:** Defines function `is_torch_tensor` with parameters `inp`.
**CN:** 定义函数 `is_torch_tensor`，参数为 `inp`。

#### Lines 198-200 — Conditional `is_torch_available()`

```python
    if is_torch_available():
        import torch
        return isinstance(inp, torch.Tensor)
```
**EN:** Checks `is_torch_available()` and selects the matching branch in function `is_torch_tensor`.
**CN:** 检查 `is_torch_available()`，并在函数 `is_torch_tensor` 中选择匹配的分支。

#### Line 201 — Return

```python
    return False
```
**EN:** Returns `False` to the caller.
**CN:** 向调用方返回 `False`。

### Line 203 — Function `is_xpu_available`

```python
def is_xpu_available():
```
**EN:** Defines function `is_xpu_available` with parameters ``.
**CN:** 定义函数 `is_xpu_available`，参数为 ``。

#### Lines 204-206 — Conditional `is_torch_available()`

```python
    if is_torch_available():
        import torch
        return torch.xpu.is_available()
```
**EN:** Checks `is_torch_available()` and selects the matching branch in function `is_xpu_available`.
**CN:** 检查 `is_torch_available()`，并在函数 `is_xpu_available` 中选择匹配的分支。

#### Line 207 — Return

```python
    return False
```
**EN:** Returns `False` to the caller.
**CN:** 向调用方返回 `False`。

### Line 209 — Function `is_xpu_tensor`

```python
def is_xpu_tensor(inp) -> bool:
```
**EN:** Defines function `is_xpu_tensor` with parameters `inp`.
**CN:** 定义函数 `is_xpu_tensor`，参数为 `inp`。

#### Lines 210-211 — Conditional `is_torch_tensor(inp)`

```python
    if is_torch_tensor(inp):
        return inp.device.type == "xpu"
```
**EN:** Checks `is_torch_tensor(inp)` and selects the matching branch in function `is_xpu_tensor`.
**CN:** 检查 `is_torch_tensor(inp)`，并在函数 `is_xpu_tensor` 中选择匹配的分支。

#### Line 212 — Return

```python
    return False
```
**EN:** Returns `False` to the caller.
**CN:** 向调用方返回 `False`。

### Line 215 — Function `torch_library_type`

```python
def torch_library_type(inp) -> cutlass_cppgen.DataType:
```
**EN:** Defines function `torch_library_type` with parameters `inp`.
**CN:** 定义函数 `torch_library_type`，参数为 `inp`。

#### Line 216 — Return

```python
    return _torch_to_library_dict.get(inp, None)
```
**EN:** Returns `_torch_to_library_dict.get(inp, None)` to the caller.
**CN:** 向调用方返回 `_torch_to_library_dict.get(inp, None)`。

### Line 219 — Function `torch_type`

```python
def torch_type(inp):
```
**EN:** Defines function `torch_type` with parameters `inp`.
**CN:** 定义函数 `torch_type`，参数为 `inp`。

#### Line 220 — Return

```python
    return _library_to_torch_dict.get(inp, None)
```
**EN:** Returns `_library_to_torch_dict.get(inp, None)` to the caller.
**CN:** 向调用方返回 `_library_to_torch_dict.get(inp, None)`。

### Line 223 — Function `is_bfloat16_available`

```python
def is_bfloat16_available():
```
**EN:** Defines function `is_bfloat16_available` with parameters ``.
**CN:** 定义函数 `is_bfloat16_available`，参数为 ``。

#### Line 224 — Global

```python
    global bfloat16_available
```
**EN:** Implements a `Global` statement in function `is_bfloat16_available`.
**CN:** 实现了一个 `Global` 语句；该语句位于在函数 `is_bfloat16_available` 中。

#### Lines 226-232 — Conditional `bfloat16_available is None`

```python
    if bfloat16_available is None:
        try:
            import bfloat16

            bfloat16_available = True
        except ImportError:
            bfloat16_available = False
```
**EN:** Checks `bfloat16_available is None` and selects the matching branch in function `is_bfloat16_available`.
**CN:** 检查 `bfloat16_available is None`，并在函数 `is_bfloat16_available` 中选择匹配的分支。

#### Line 233 — Return

```python
    return bfloat16_available
```
**EN:** Returns `bfloat16_available` to the caller.
**CN:** 向调用方返回 `bfloat16_available`。

### Line 236 — Function `bfloat16_library_type`

```python
def bfloat16_library_type(inp) -> cutlass_cppgen.DataType:
```
**EN:** Defines function `bfloat16_library_type` with parameters `inp`.
**CN:** 定义函数 `bfloat16_library_type`，参数为 `inp`。

#### Lines 237-240 — Conditional `is_bfloat16_available()`

```python
    if is_bfloat16_available():
        import bfloat16
        if inp == bfloat16.bfloat16:
            return cutlass_cppgen.DataType.bf16
```
**EN:** Checks `is_bfloat16_available()` and selects the matching branch in function `bfloat16_library_type`.
**CN:** 检查 `is_bfloat16_available()`，并在函数 `bfloat16_library_type` 中选择匹配的分支。

### Line 243 — Function `bfloat16_type`

```python
def bfloat16_type(inp):
```
**EN:** Defines function `bfloat16_type` with parameters `inp`.
**CN:** 定义函数 `bfloat16_type`，参数为 `inp`。

#### Lines 244-247 — Conditional `is_bfloat16_available()`

```python
    if is_bfloat16_available():
        import bfloat16
        if inp == cutlass_cppgen.DataType.bf16:
            return bfloat16.bfloat16
```
**EN:** Checks `is_bfloat16_available()` and selects the matching branch in function `bfloat16_type`.
**CN:** 检查 `is_bfloat16_available()`，并在函数 `bfloat16_type` 中选择匹配的分支。

### Line 250 — Function `library_type`

```python
def library_type(inp):
```
**EN:** Defines function `library_type` with parameters `inp`.
**CN:** 定义函数 `library_type`，参数为 `inp`。

#### Lines 251-252 — Conditional `inp in DataTypeSize`

```python
    if inp in DataTypeSize:
        return inp
```
**EN:** Checks `inp in DataTypeSize` and selects the matching branch in function `library_type`.
**CN:** 检查 `inp in DataTypeSize`，并在函数 `library_type` 中选择匹配的分支。

#### Lines 254-262 — Loop over `[bfloat16_library_type, cupy_library_type, numpy_library_type, torch_library_type]`

```python
    for cvt_fn in [
        bfloat16_library_type,
        cupy_library_type,
        numpy_library_type,
        torch_library_type,
    ]:
        out = cvt_fn(inp)
        if out is not None:
            return out
```
**EN:** Iterates `cvt_fn` over `[bfloat16_library_type, cupy_library_type, numpy_library_type, torch_library_type]` to repeat a processing step.
**CN:** 让 `cvt_fn` 遍历 `[bfloat16_library_type, cupy_library_type, numpy_library_type, torch_library_type]`，从而重复执行处理步骤。

#### Line 264 — Raise exception

```python
    raise Exception(f"No available conversion from type {inp} to a library type.")
```
**EN:** Raises `Exception(f'No available conversion from type {inp} to a library type.')` to signal an error or unsupported state.
**CN:** 抛出 `Exception(f'No available conversion from type {inp} to a library type.')`，用于报告错误或不支持的状态。

### Line 267 — Function `_tensor_from_numpy`

```python
def _tensor_from_numpy(np_tensor):
```
**EN:** Defines function `_tensor_from_numpy` with parameters `np_tensor`.
**CN:** 定义函数 `_tensor_from_numpy`，参数为 `np_tensor`。

#### Line 268 — Assign `dtype`

```python
    dtype = library_type(np_tensor.dtype)
```
**EN:** Assigns `dtype` from `library_type(np_tensor.dtype)`, establishing state in function `_tensor_from_numpy`.
**CN:** 将 `dtype` 赋值为 `library_type(np_tensor.dtype)`，用于在函数 `_tensor_from_numpy` 中建立状态。

#### Lines 269-272 — Conditional `np_tensor.flags.c_contiguous`

```python
    if np_tensor.flags.c_contiguous:
        layout = cutlass_cppgen.LayoutType.RowMajor
    elif np_tensor.flags.f_contiguous:
        layout = cutlass_cppgen.LayoutType.ColumnMajor
```
**EN:** Checks `np_tensor.flags.c_contiguous` and selects the matching branch in function `_tensor_from_numpy`.
**CN:** 检查 `np_tensor.flags.c_contiguous`，并在函数 `_tensor_from_numpy` 中选择匹配的分支。

#### Line 273 — Return

```python
    return (dtype, layout)
```
**EN:** Returns `(dtype, layout)` to the caller.
**CN:** 向调用方返回 `(dtype, layout)`。

### Line 276 — Function `_tensor_from_torch`

```python
def _tensor_from_torch(pt_tensor):
```
**EN:** Defines function `_tensor_from_torch` with parameters `pt_tensor`.
**CN:** 定义函数 `_tensor_from_torch`，参数为 `pt_tensor`。

#### Line 277 — Assign `dtype`

```python
    dtype = library_type(pt_tensor.dtype)
```
**EN:** Assigns `dtype` from `library_type(pt_tensor.dtype)`, establishing state in function `_tensor_from_torch`.
**CN:** 将 `dtype` 赋值为 `library_type(pt_tensor.dtype)`，用于在函数 `_tensor_from_torch` 中建立状态。

#### Line 278 — Return

```python
    return (dtype, cutlass_cppgen.LayoutType.RowMajor)
```
**EN:** Returns `(dtype, cutlass_cppgen.LayoutType.RowMajor)` to the caller.
**CN:** 向调用方返回 `(dtype, cutlass_cppgen.LayoutType.RowMajor)`。

### Line 281 — Function `get_datatype_and_layout`

```python
def get_datatype_and_layout(tensor):
```
**EN:** Defines function `get_datatype_and_layout` with parameters `tensor`.
**CN:** 定义函数 `get_datatype_and_layout`，参数为 `tensor`。

#### Lines 282-289 — Conditional `is_numpy_tensor(tensor) or is_cupy_tensor(tensor)`

```python
    if (is_numpy_tensor(tensor) or is_cupy_tensor(tensor)):
        return _tensor_from_numpy(tensor)
    elif is_torch_tensor(tensor):
        return _tensor_from_torch(tensor)
    elif isinstance(tensor, float) or isinstance(tensor, int):
        return (cutlass_cppgen.DataType.f32, cutlass_cppgen.LayoutType.RowMajor)
    else:
        raise Exception(f"Unable to convert tensor of type {type(tensor)} to Python-bound CUTLASS datatype and layout.")
```
**EN:** Checks `is_numpy_tensor(tensor) or is_cupy_tensor(tensor)` and selects the matching branch in function `get_datatype_and_layout`.
**CN:** 检查 `is_numpy_tensor(tensor) or is_cupy_tensor(tensor)`，并在函数 `get_datatype_and_layout` 中选择匹配的分支。

### Line 292 — Function `get_tensor_shape`

```python
def get_tensor_shape(tensor, op="GEMM"):
```
**EN:** Defines function `get_tensor_shape` with parameters `tensor, op`.
**CN:** 定义函数 `get_tensor_shape`，参数为 `tensor, op`。

#### Lines 293-305 — Conditional `is_numpy_tensor(tensor) or is_cupy_tensor(tensor)`

```python
    if (is_numpy_tensor(tensor) or is_cupy_tensor(tensor)):
        return tensor.shape
    elif is_torch_tensor(tensor):
        size = tensor.size()
        if op == "CONV":
            # PyTorch Tensors have shape NCHW
            return (size[0], size[2], size[3], size[1])
        else:
            return tuple(tensor.size())
    elif isinstance(tensor, float) or isinstance(tensor, int):
        return (1,)
    else:
        raise Exception(f"Unable to convert tensor of type {type(tensor)} to Python-bound CUTLASS datatype and layout.")
```
**EN:** Checks `is_numpy_tensor(tensor) or is_cupy_tensor(tensor)` and selects the matching branch in function `get_tensor_shape`.
**CN:** 检查 `is_numpy_tensor(tensor) or is_cupy_tensor(tensor)`，并在函数 `get_tensor_shape` 中选择匹配的分支。

### Line 308 — Assign `_math_operation_value_map`

```python
_math_operation_value_map = {x.value: x for x in MathOperation}
```
**EN:** Assigns `_math_operation_value_map` from `{x.value: x for x in MathOperation}`, establishing state at module scope.
**CN:** 将 `_math_operation_value_map` 赋值为 `{x.value: x for x in MathOperation}`，用于在模块级建立状态。

### Line 311 — Function `backend_math_operation`

```python
def backend_math_operation(math_op: MathOperation):
```
**EN:** Defines function `backend_math_operation` with parameters `math_op`.
**CN:** 定义函数 `backend_math_operation`，参数为 `math_op`。

#### Lines 312-313 — Conditional `math_op.value not in _math_operation_value_map.keys()`

```python
    if math_op.value not in _math_operation_value_map.keys():
        raise Exception(f"Unable to convert math operation of type {math_op} to backend math operation.")
```
**EN:** Checks `math_op.value not in _math_operation_value_map.keys()` and selects the matching branch in function `backend_math_operation`.
**CN:** 检查 `math_op.value not in _math_operation_value_map.keys()`，并在函数 `backend_math_operation` 中选择匹配的分支。

#### Line 314 — Return

```python
    return _math_operation_value_map[math_op.value]
```
**EN:** Returns `_math_operation_value_map[math_op.value]` to the caller.
**CN:** 向调用方返回 `_math_operation_value_map[math_op.value]`。

### Lines 317-320 — Function `construct_backend_td`

```python
def construct_backend_td(td: cutlass_cppgen.TileDescription,
                         kernel_schedule: cutlass_cppgen.KernelScheduleType,
                         epilogue_schedule: cutlass_cppgen.EpilogueScheduleType,
                         tile_scheduler: cutlass_cppgen.TileSchedulerType) -> TileDescription:
```
**EN:** Defines function `construct_backend_td` with parameters `td, kernel_schedule, epilogue_schedule, tile_scheduler`.
**CN:** 定义函数 `construct_backend_td`，参数为 `td, kernel_schedule, epilogue_schedule, tile_scheduler`。

#### Line 321 — Assign `mi`

```python
    mi = td.math_instruction
```
**EN:** Assigns `mi` from `td.math_instruction`, establishing state in function `construct_backend_td`.
**CN:** 将 `mi` 赋值为 `td.math_instruction`，用于在函数 `construct_backend_td` 中建立状态。

#### Lines 322-329 — Assign `backend_mi`

```python
    backend_mi = MathInstruction(
        mi.instruction_shape,
        mi.element_a,
        mi.element_b,
        mi.element_accumulator,
        mi.opcode_class,
        backend_math_operation(mi.math_operation)
    )
```
**EN:** Assigns `backend_mi` from `MathInstruction(mi.instruction_shape, mi.element_a, mi.element_b, mi.element_accumulator, mi.opcode_class, backend_math_operation(mi.math_operation))`, establishing state in function `construct_backend_td`.
**CN:** 将 `backend_mi` 赋值为 `MathInstruction(mi.instruction_shape, mi.element_a, mi.element_b, mi.element_accumulator, mi.opcode_class, backend_math_operation(mi.math_operation))`，用于在函数 `construct_backend_td` 中建立状态。

#### Line 330 — Assign `cluster_shape`

```python
    cluster_shape = td.cluster_shape if hasattr(td, "cluster_shape") else [1, 1, 1]
```
**EN:** Assigns `cluster_shape` from `td.cluster_shape if hasattr(td, 'cluster_shape') else [1, 1, 1]`, establishing state in function `construct_backend_td`.
**CN:** 将 `cluster_shape` 赋值为 `td.cluster_shape if hasattr(td, 'cluster_shape') else [1, 1, 1]`，用于在函数 `construct_backend_td` 中建立状态。

#### Lines 331-332 — Return

```python
    return TileDescription(td.threadblock_shape, td.stages, td.warp_count,
                           backend_mi, cluster_shape, kernel_schedule, epilogue_schedule, tile_scheduler)
```
**EN:** Returns `TileDescription(td.threadblock_shape, td.stages, td.warp_count, backend_mi, cluster_shape, kernel_schedule, epilogue_schedule, tile_scheduler)` to the caller.
**CN:** 向调用方返回 `TileDescription(td.threadblock_shape, td.stages, td.warp_count, backend_mi, cluster_shape, kernel_schedule, epilogue_schedule, tile_scheduler)`。

### Lines 335-343 — Function `td_from_profiler_op`

```python
def td_from_profiler_op(op) -> TileDescription:
    """
    Converts the profiler's TileDescription in ``op`` into the backend TileDescription

    :param op: profiler Operation

    :returns: backend TileDescription
    :rtype: cutlass_cppgen.backend.TileDescription
    """
```
**EN:** Defines function `td_from_profiler_op` with parameters `op`. Purpose: Converts the profiler's TileDescription in ``op`` into the backend TileDescription
**CN:** 定义函数 `td_from_profiler_op`，参数为 `op`。 其用途：Converts the profiler's TileDescription in ``op`` into the backend TileDescription

#### Line 344 — Assign `kschedule`

```python
    kschedule = op.kernel_schedule if hasattr(op, 'kernel_schedule') else None
```
**EN:** Assigns `kschedule` from `op.kernel_schedule if hasattr(op, 'kernel_schedule') else None`, establishing state in function `td_from_profiler_op`.
**CN:** 将 `kschedule` 赋值为 `op.kernel_schedule if hasattr(op, 'kernel_schedule') else None`，用于在函数 `td_from_profiler_op` 中建立状态。

#### Line 345 — Assign `eschedule`

```python
    eschedule = op.epilogue_schedule if hasattr(op, 'epilogue_schedule') else None
```
**EN:** Assigns `eschedule` from `op.epilogue_schedule if hasattr(op, 'epilogue_schedule') else None`, establishing state in function `td_from_profiler_op`.
**CN:** 将 `eschedule` 赋值为 `op.epilogue_schedule if hasattr(op, 'epilogue_schedule') else None`，用于在函数 `td_from_profiler_op` 中建立状态。

#### Line 346 — Assign `tschedule`

```python
    tschedule = op.tile_scheduler if hasattr(op, 'tile_scheduler') else None
```
**EN:** Assigns `tschedule` from `op.tile_scheduler if hasattr(op, 'tile_scheduler') else None`, establishing state in function `td_from_profiler_op`.
**CN:** 将 `tschedule` 赋值为 `op.tile_scheduler if hasattr(op, 'tile_scheduler') else None`，用于在函数 `td_from_profiler_op` 中建立状态。

#### Line 347 — Return

```python
    return construct_backend_td(op.tile_description, kschedule, eschedule, tschedule)
```
**EN:** Returns `construct_backend_td(op.tile_description, kschedule, eschedule, tschedule)` to the caller.
**CN:** 向调用方返回 `construct_backend_td(op.tile_description, kschedule, eschedule, tschedule)`。

### Lines 350-359 — Function `td_from_profiler_td`

```python
def td_from_profiler_td(td: TileDescription) -> TileDescription:
    """
    Converts the profiler's TileDescription into the backend TileDescription

    :param td: profiler TileDescription
    :type td: cutlass_cppgen.TileDescription

    :returns: backend TileDescription
    :rtype: cutlass_cppgen.backend.TileDescription
    """
```
**EN:** Defines function `td_from_profiler_td` with parameters `td`. Purpose: Converts the profiler's TileDescription into the backend TileDescription
**CN:** 定义函数 `td_from_profiler_td`，参数为 `td`。 其用途：Converts the profiler's TileDescription into the backend TileDescription

#### Line 360 — Return

```python
    return construct_backend_td(td, kernel_schedule=None, epilogue_schedule=None, tile_scheduler=None)
```
**EN:** Returns `construct_backend_td(td, kernel_schedule=None, epilogue_schedule=None, tile_scheduler=None)` to the caller.
**CN:** 向调用方返回 `construct_backend_td(td, kernel_schedule=None, epilogue_schedule=None, tile_scheduler=None)`。

### Line 363 — Function `to_camel_case`

```python
def to_camel_case(snake_str):
```
**EN:** Defines function `to_camel_case` with parameters `snake_str`.
**CN:** 定义函数 `to_camel_case`，参数为 `snake_str`。

#### Line 364 — Return

```python
    return "".join(x.capitalize() for x in snake_str.lower().split("_"))
```
**EN:** Returns `''.join((x.capitalize() for x in snake_str.lower().split('_')))` to the caller.
**CN:** 向调用方返回 `''.join((x.capitalize() for x in snake_str.lower().split('_')))`。

### Lines 367-368 — Function `getattr_enum`

```python
def getattr_enum(obj, attr_name):
    # The attr_name is under the snake_case
```
**EN:** Defines function `getattr_enum` with parameters `obj, attr_name`.
**CN:** 定义函数 `getattr_enum`，参数为 `obj, attr_name`。

#### Line 369 — Assign `camel_attr`

```python
    camel_attr = to_camel_case(attr_name)
```
**EN:** Assigns `camel_attr` from `to_camel_case(attr_name)`, establishing state in function `getattr_enum`.
**CN:** 将 `camel_attr` 赋值为 `to_camel_case(attr_name)`，用于在函数 `getattr_enum` 中建立状态。

#### Lines 370-373 — Conditional `hasattr(obj, camel_attr)`

```python
    if hasattr(obj, camel_attr):
        return getattr(obj, camel_attr)
    else:
        raise Exception(f"Invalid option: {attr_name}")
```
**EN:** Checks `hasattr(obj, camel_attr)` and selects the matching branch in function `getattr_enum`.
**CN:** 检查 `hasattr(obj, camel_attr)`，并在函数 `getattr_enum` 中选择匹配的分支。

## Key Concepts / 关键概念

- **EN:** Top-level classes: none.
- **CN:** 顶层类：无。
- **EN:** Top-level functions: `is_numpy_available`, `is_numpy_tensor`, `numpy_library_type`, `numpy_type`, `is_cupy_available`, `is_cupy_tensor`, `cupy_library_type`, `cupy_type`, `is_torch_available`, `is_torch_tensor`, `is_xpu_available`, `is_xpu_tensor`, `torch_library_type`, `torch_type`, `is_bfloat16_available`, `bfloat16_library_type`, `bfloat16_type`, `library_type`, `_tensor_from_numpy`, `_tensor_from_torch`, `get_datatype_and_layout`, `get_tensor_shape`, `backend_math_operation`, `construct_backend_td`, `td_from_profiler_op`, `td_from_profiler_td`, `to_camel_case`, `getattr_enum`.
- **CN:** 顶层函数：`is_numpy_available`, `is_numpy_tensor`, `numpy_library_type`, `numpy_type`, `is_cupy_available`, `is_cupy_tensor`, `cupy_library_type`, `cupy_type`, `is_torch_available`, `is_torch_tensor`, `is_xpu_available`, `is_xpu_tensor`, `torch_library_type`, `torch_type`, `is_bfloat16_available`, `bfloat16_library_type`, `bfloat16_type`, `library_type`, `_tensor_from_numpy`, `_tensor_from_torch`, `get_datatype_and_layout`, `get_tensor_shape`, `backend_math_operation`, `construct_backend_td`, `td_from_profiler_op`, `td_from_profiler_td`, `to_camel_case`, `getattr_enum`。
- **EN:** This module organizes backend utilities, data structures, or runtime helpers used by cutlass_cppgen.
- **CN:** 该模块组织 cutlass_cppgen 所需的后端工具、数据结构或运行时辅助逻辑。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen`, `cutlass_cppgen.backend.library`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** `bfloat16`, `cupy`, `numpy`, `torch`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
