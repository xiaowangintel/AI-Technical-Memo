# shape.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/shape.py`
- **EN:** Utilities for expressing shapes
- **CN:** 模块文档说明：Utilities for expressing shapes

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
Utilities for expressing shapes
"""
```
**EN:** Docstring explains this scope: Utilities for expressing shapes
**CN:** 文档字符串说明了该作用域的用途：Utilities for expressing shapes

### Lines 37-41 — From `cutlass_library` import

```python
from cutlass_library import (
    ConvMode,
    ConvKind,
    LayoutType
)
```
**EN:** Imports `ConvMode, ConvKind, LayoutType` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `ConvMode, ConvKind, LayoutType`，以便后续代码在模块级复用共享定义。

### Lines 42-46 — From `cutlass_cppgen.backend.c_types` import

```python
from cutlass_cppgen.backend.c_types import (
    Conv2DProblemSize_,
    GemmCoord_,
    GemmCoordBatched_
)
```
**EN:** Imports `Conv2DProblemSize_, GemmCoord_, GemmCoordBatched_` from `cutlass_cppgen.backend.c_types` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.c_types` 导入 `Conv2DProblemSize_, GemmCoord_, GemmCoordBatched_`，以便后续代码在模块级复用共享定义。

### Line 49 — Class `MatrixCoord`

```python
class MatrixCoord:
```
**EN:** Declares class `MatrixCoord` deriving from `object`.
**CN:** 声明类 `MatrixCoord`，其基类为 `object`。

#### Line 50 — Function `__init__`

```python
    def __init__(self, row, col):
```
**EN:** Defines function `__init__` with parameters `self, row, col`.
**CN:** 定义函数 `__init__`，参数为 `self, row, col`。

##### Line 51 — Assign `self._row`

```python
        self._row = row
```
**EN:** Assigns `self._row` from `row`, establishing state in function `__init__`.
**CN:** 将 `self._row` 赋值为 `row`，用于在函数 `__init__` 中建立状态。

##### Line 52 — Assign `self._col`

```python
        self._col = col
```
**EN:** Assigns `self._col` from `col`, establishing state in function `__init__`.
**CN:** 将 `self._col` 赋值为 `col`，用于在函数 `__init__` 中建立状态。

#### Lines 54-55 — Function `row`

```python
    @property
    def row(self):
```
**EN:** Defines function `row` with parameters `self`.
**CN:** 定义函数 `row`，参数为 `self`。

##### Line 56 — Return

```python
        return self._row
```
**EN:** Returns `self._row` to the caller.
**CN:** 向调用方返回 `self._row`。

#### Lines 58-59 — Function `column`

```python
    @property
    def column(self):
```
**EN:** Defines function `column` with parameters `self`.
**CN:** 定义函数 `column`，参数为 `self`。

##### Line 60 — Return

```python
        return self._col
```
**EN:** Returns `self._col` to the caller.
**CN:** 向调用方返回 `self._col`。

#### Lines 62-71 — Function `leading_dimension`

```python
    def leading_dimension(self, layout: LayoutType) -> int:
        """
        Returns the leading dimension for a matrix with layout ``layout`` and shape provided by the MatrixCoord.

        :param layout: layout of matrix
        :type layout: cutlass_library.LayoutType

        :returns: leading dimension
        :rtype: int
        """
```
**EN:** Defines function `leading_dimension` with parameters `self, layout`. Purpose: Returns the leading dimension for a matrix with layout ``layout`` and shape provided by the MatrixCoord.
**CN:** 定义函数 `leading_dimension`，参数为 `self, layout`。 其用途：Returns the leading dimension for a matrix with layout ``layout`` and shape provided by the MatrixCoord.

##### Lines 72-77 — Conditional `layout == LayoutType.RowMajor`

```python
        if layout == LayoutType.RowMajor:
            return self._col
        elif layout == LayoutType.ColumnMajor:
            return self._row
        else:
            raise Exception(f'Unsupported layout for leading dimension calculation: {layout}')
```
**EN:** Checks `layout == LayoutType.RowMajor` and selects the matching branch in function `leading_dimension`.
**CN:** 检查 `layout == LayoutType.RowMajor`，并在函数 `leading_dimension` 中选择匹配的分支。

### Line 80 — Class `GemmCoord`

```python
class GemmCoord:
```
**EN:** Declares class `GemmCoord` deriving from `object`.
**CN:** 声明类 `GemmCoord`，其基类为 `object`。

#### Line 81 — Function `__init__`

```python
    def __init__(self, m: int, n: int, k: int):
```
**EN:** Defines function `__init__` with parameters `self, m, n, k`.
**CN:** 定义函数 `__init__`，参数为 `self, m, n, k`。

##### Line 82 — Assign `self._m`

```python
        self._m = m
```
**EN:** Assigns `self._m` from `m`, establishing state in function `__init__`.
**CN:** 将 `self._m` 赋值为 `m`，用于在函数 `__init__` 中建立状态。

##### Line 83 — Assign `self._n`

```python
        self._n = n
```
**EN:** Assigns `self._n` from `n`, establishing state in function `__init__`.
**CN:** 将 `self._n` 赋值为 `n`，用于在函数 `__init__` 中建立状态。

##### Line 84 — Assign `self._k`

```python
        self._k = k
```
**EN:** Assigns `self._k` from `k`, establishing state in function `__init__`.
**CN:** 将 `self._k` 赋值为 `k`，用于在函数 `__init__` 中建立状态。

#### Lines 86-87 — Function `m`

```python
    @property
    def m(self) -> int:
```
**EN:** Defines function `m` with parameters `self`.
**CN:** 定义函数 `m`，参数为 `self`。

##### Line 88 — Return

```python
        return self._m
```
**EN:** Returns `self._m` to the caller.
**CN:** 向调用方返回 `self._m`。

#### Lines 90-91 — Function `n`

```python
    @property
    def n(self) -> int:
```
**EN:** Defines function `n` with parameters `self`.
**CN:** 定义函数 `n`，参数为 `self`。

##### Line 92 — Return

```python
        return self._n
```
**EN:** Returns `self._n` to the caller.
**CN:** 向调用方返回 `self._n`。

#### Lines 94-95 — Function `k`

```python
    @property
    def k(self) -> int:
```
**EN:** Defines function `k` with parameters `self`.
**CN:** 定义函数 `k`，参数为 `self`。

##### Line 96 — Return

```python
        return self._k
```
**EN:** Returns `self._k` to the caller.
**CN:** 向调用方返回 `self._k`。

#### Lines 98-99 — Function `mk`

```python
    @property
    def mk(self) -> MatrixCoord:
```
**EN:** Defines function `mk` with parameters `self`.
**CN:** 定义函数 `mk`，参数为 `self`。

##### Line 100 — Return

```python
        return MatrixCoord(self._m, self._k)
```
**EN:** Returns `MatrixCoord(self._m, self._k)` to the caller.
**CN:** 向调用方返回 `MatrixCoord(self._m, self._k)`。

#### Lines 102-103 — Function `mn`

```python
    @property
    def mn(self) -> MatrixCoord:
```
**EN:** Defines function `mn` with parameters `self`.
**CN:** 定义函数 `mn`，参数为 `self`。

##### Line 104 — Return

```python
        return MatrixCoord(self._m, self._n)
```
**EN:** Returns `MatrixCoord(self._m, self._n)` to the caller.
**CN:** 向调用方返回 `MatrixCoord(self._m, self._n)`。

#### Lines 106-107 — Function `kn`

```python
    @property
    def kn(self) -> MatrixCoord:
```
**EN:** Defines function `kn` with parameters `self`.
**CN:** 定义函数 `kn`，参数为 `self`。

##### Line 108 — Return

```python
        return MatrixCoord(self._k, self._n)
```
**EN:** Returns `MatrixCoord(self._k, self._n)` to the caller.
**CN:** 向调用方返回 `MatrixCoord(self._k, self._n)`。

#### Lines 110-111 — Function `ctype`

```python
    @property
    def ctype(self) -> GemmCoord_:
```
**EN:** Defines function `ctype` with parameters `self`.
**CN:** 定义函数 `ctype`，参数为 `self`。

##### Line 112 — Return

```python
        return GemmCoord_(self._m, self._n, self._k)
```
**EN:** Returns `GemmCoord_(self._m, self._n, self._k)` to the caller.
**CN:** 向调用方返回 `GemmCoord_(self._m, self._n, self._k)`。

#### Line 114 — Function `batched_ctype`

```python
    def batched_ctype(self, batch_count: int) -> GemmCoordBatched_:
```
**EN:** Defines function `batched_ctype` with parameters `self, batch_count`.
**CN:** 定义函数 `batched_ctype`，参数为 `self, batch_count`。

##### Line 115 — Return

```python
        return GemmCoordBatched_(self._m, self._n, self._k, batch_count)
```
**EN:** Returns `GemmCoordBatched_(self._m, self._n, self._k, batch_count)` to the caller.
**CN:** 向调用方返回 `GemmCoordBatched_(self._m, self._n, self._k, batch_count)`。

### Line 118 — Class `Conv2DProblemSize`

```python
class Conv2DProblemSize:
```
**EN:** Declares class `Conv2DProblemSize` deriving from `object`.
**CN:** 声明类 `Conv2DProblemSize`，其基类为 `object`。

#### Lines 119-125 — Function `__init__`

```python
    def __init__(
        self, n: int, h: int, w: int, c: int,
        k: int, r: int, s: int, c_: int,
        pad_h: int, pad_w: int, stride_h: int, stride_w: int,
        dilation_h: int, dilation_w: int, mode: ConvMode=ConvMode.CrossCorrelation,
        split_k_slices: int=1, groups: int=1):
```
**EN:** Defines function `__init__` with parameters `self, n, h, w, c, k, r, s, c_, pad_h, pad_w, stride_h, stride_w, dilation_h, dilation_w, mode, split_k_slices, groups`.
**CN:** 定义函数 `__init__`，参数为 `self, n, h, w, c, k, r, s, c_, pad_h, pad_w, stride_h, stride_w, dilation_h, dilation_w, mode, split_k_slices, groups`。

##### Line 126 — Assign `self.N`

```python
        self.N = n
```
**EN:** Assigns `self.N` from `n`, establishing state in function `__init__`.
**CN:** 将 `self.N` 赋值为 `n`，用于在函数 `__init__` 中建立状态。

##### Line 127 — Assign `self.H`

```python
        self.H = h
```
**EN:** Assigns `self.H` from `h`, establishing state in function `__init__`.
**CN:** 将 `self.H` 赋值为 `h`，用于在函数 `__init__` 中建立状态。

##### Line 128 — Assign `self.W`

```python
        self.W = w
```
**EN:** Assigns `self.W` from `w`, establishing state in function `__init__`.
**CN:** 将 `self.W` 赋值为 `w`，用于在函数 `__init__` 中建立状态。

##### Line 129 — Assign `self.C`

```python
        self.C = c
```
**EN:** Assigns `self.C` from `c`, establishing state in function `__init__`.
**CN:** 将 `self.C` 赋值为 `c`，用于在函数 `__init__` 中建立状态。

##### Line 130 — Assign `self.K`

```python
        self.K = k
```
**EN:** Assigns `self.K` from `k`, establishing state in function `__init__`.
**CN:** 将 `self.K` 赋值为 `k`，用于在函数 `__init__` 中建立状态。

##### Line 131 — Assign `self.R`

```python
        self.R = r
```
**EN:** Assigns `self.R` from `r`, establishing state in function `__init__`.
**CN:** 将 `self.R` 赋值为 `r`，用于在函数 `__init__` 中建立状态。

##### Line 132 — Assign `self.S`

```python
        self.S = s
```
**EN:** Assigns `self.S` from `s`, establishing state in function `__init__`.
**CN:** 将 `self.S` 赋值为 `s`，用于在函数 `__init__` 中建立状态。

##### Line 133 — Assign `self.pad_h`

```python
        self.pad_h = pad_h
```
**EN:** Assigns `self.pad_h` from `pad_h`, establishing state in function `__init__`.
**CN:** 将 `self.pad_h` 赋值为 `pad_h`，用于在函数 `__init__` 中建立状态。

##### Line 134 — Assign `self.pad_w`

```python
        self.pad_w = pad_w
```
**EN:** Assigns `self.pad_w` from `pad_w`, establishing state in function `__init__`.
**CN:** 将 `self.pad_w` 赋值为 `pad_w`，用于在函数 `__init__` 中建立状态。

##### Line 135 — Assign `self.stride_h`

```python
        self.stride_h = stride_h
```
**EN:** Assigns `self.stride_h` from `stride_h`, establishing state in function `__init__`.
**CN:** 将 `self.stride_h` 赋值为 `stride_h`，用于在函数 `__init__` 中建立状态。

##### Line 136 — Assign `self.stride_w`

```python
        self.stride_w = stride_w
```
**EN:** Assigns `self.stride_w` from `stride_w`, establishing state in function `__init__`.
**CN:** 将 `self.stride_w` 赋值为 `stride_w`，用于在函数 `__init__` 中建立状态。

##### Line 137 — Assign `self.dilation_h`

```python
        self.dilation_h = dilation_h
```
**EN:** Assigns `self.dilation_h` from `dilation_h`, establishing state in function `__init__`.
**CN:** 将 `self.dilation_h` 赋值为 `dilation_h`，用于在函数 `__init__` 中建立状态。

##### Line 138 — Assign `self.dilation_w`

```python
        self.dilation_w = dilation_w
```
**EN:** Assigns `self.dilation_w` from `dilation_w`, establishing state in function `__init__`.
**CN:** 将 `self.dilation_w` 赋值为 `dilation_w`，用于在函数 `__init__` 中建立状态。

##### Line 139 — Assign `self.mode`

```python
        self.mode = int(mode)
```
**EN:** Assigns `self.mode` from `int(mode)`, establishing state in function `__init__`.
**CN:** 将 `self.mode` 赋值为 `int(mode)`，用于在函数 `__init__` 中建立状态。

##### Line 140 — Assign `self.split_k_slices`

```python
        self.split_k_slices = split_k_slices
```
**EN:** Assigns `self.split_k_slices` from `split_k_slices`, establishing state in function `__init__`.
**CN:** 将 `self.split_k_slices` 赋值为 `split_k_slices`，用于在函数 `__init__` 中建立状态。

##### Line 141 — Assign `self.groups`

```python
        self.groups = groups
```
**EN:** Assigns `self.groups` from `groups`, establishing state in function `__init__`.
**CN:** 将 `self.groups` 赋值为 `groups`，用于在函数 `__init__` 中建立状态。

##### Line 142 — Assign `self.P`

```python
        self.P = ((h + pad_h * 2 - r * dilation_h) // stride_h) + 1
```
**EN:** Assigns `self.P` from `(h + pad_h * 2 - r * dilation_h) // stride_h + 1`, establishing state in function `__init__`.
**CN:** 将 `self.P` 赋值为 `(h + pad_h * 2 - r * dilation_h) // stride_h + 1`，用于在函数 `__init__` 中建立状态。

##### Line 143 — Assign `self.Q`

```python
        self.Q = ((w + pad_w * 2 - s * dilation_w) // stride_w) + 1
```
**EN:** Assigns `self.Q` from `(w + pad_w * 2 - s * dilation_w) // stride_w + 1`, establishing state in function `__init__`.
**CN:** 将 `self.Q` 赋值为 `(w + pad_w * 2 - s * dilation_w) // stride_w + 1`，用于在函数 `__init__` 中建立状态。

#### Lines 145-146 — Function `ctype`

```python
    @property
    def ctype(self) -> Conv2DProblemSize_:
```
**EN:** Defines function `ctype` with parameters `self`.
**CN:** 定义函数 `ctype`，参数为 `self`。

##### Line 147 — Return

```python
        return Conv2DProblemSize_(self)
```
**EN:** Returns `Conv2DProblemSize_(self)` to the caller.
**CN:** 向调用方返回 `Conv2DProblemSize_(self)`。

#### Line 149 — Function `implicit_gemm_size`

```python
    def implicit_gemm_size(self, kind: ConvKind):
```
**EN:** Defines function `implicit_gemm_size` with parameters `self, kind`.
**CN:** 定义函数 `implicit_gemm_size`，参数为 `self, kind`。

##### Lines 150-167 — Conditional `kind == ConvKind.Fprop`

```python
        if kind == ConvKind.Fprop:
            return GemmCoord(
                self.N * self.P * self.Q,
                self.K,
                self.R * self.S * self.C // self.groups
            )
        elif kind == ConvKind.Dgrad:
            return GemmCoord(
                self.N * self.H * self.W,
                self.C,
                self.R * self.S * self.K
            )
        elif kind == ConvKind.Wgrad:
            return GemmCoord(
                self.K,
                self.R * self.S * self.C,
                self.N * self.P * self.Q
            )
```
**EN:** Checks `kind == ConvKind.Fprop` and selects the matching branch in function `implicit_gemm_size`.
**CN:** 检查 `kind == ConvKind.Fprop`，并在函数 `implicit_gemm_size` 中选择匹配的分支。

#### Lines 169-170 — Function `from_sizes`

```python
    @staticmethod
    def from_sizes(input_size, weight_size):
```
**EN:** Defines function `from_sizes` with parameters `input_size, weight_size`.
**CN:** 定义函数 `from_sizes`，参数为 `input_size, weight_size`。

##### Line 171 — Assign `K, R, S, _`

```python
        K, R, S, _ = weight_size
```
**EN:** Assigns `K, R, S, _` from `weight_size`, establishing state in function `from_sizes`.
**CN:** 将 `K, R, S, _` 赋值为 `weight_size`，用于在函数 `from_sizes` 中建立状态。

##### Line 172 — Assign `pad_h`

```python
        pad_h = R // 2
```
**EN:** Assigns `pad_h` from `R // 2`, establishing state in function `from_sizes`.
**CN:** 将 `pad_h` 赋值为 `R // 2`，用于在函数 `from_sizes` 中建立状态。

##### Line 173 — Assign `pad_w`

```python
        pad_w = S // 2
```
**EN:** Assigns `pad_w` from `S // 2`, establishing state in function `from_sizes`.
**CN:** 将 `pad_w` 赋值为 `S // 2`，用于在函数 `from_sizes` 中建立状态。

##### Line 174 — Assign `stride_h`

```python
        stride_h = 1
```
**EN:** Assigns `stride_h` from `1`, establishing state in function `from_sizes`.
**CN:** 将 `stride_h` 赋值为 `1`，用于在函数 `from_sizes` 中建立状态。

##### Line 175 — Assign `stride_w`

```python
        stride_w = 1
```
**EN:** Assigns `stride_w` from `1`, establishing state in function `from_sizes`.
**CN:** 将 `stride_w` 赋值为 `1`，用于在函数 `from_sizes` 中建立状态。

##### Line 176 — Assign `dilation_h`

```python
        dilation_h = 1
```
**EN:** Assigns `dilation_h` from `1`, establishing state in function `from_sizes`.
**CN:** 将 `dilation_h` 赋值为 `1`，用于在函数 `from_sizes` 中建立状态。

##### Line 177 — Assign `dilation_w`

```python
        dilation_w = 1
```
**EN:** Assigns `dilation_w` from `1`, establishing state in function `from_sizes`.
**CN:** 将 `dilation_w` 赋值为 `1`，用于在函数 `from_sizes` 中建立状态。

##### Lines 178-184 — Return

```python
        return Conv2DProblemSize(
            *input_size,
            *weight_size,
            pad_h, pad_w,
            stride_h, stride_w,
            dilation_h, dilation_w
        )
```
**EN:** Returns `Conv2DProblemSize(*input_size, *weight_size, pad_h, pad_w, stride_h, stride_w, dilation_h, dilation_w)` to the caller.
**CN:** 向调用方返回 `Conv2DProblemSize(*input_size, *weight_size, pad_h, pad_w, stride_h, stride_w, dilation_h, dilation_w)`。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `MatrixCoord`, `GemmCoord`, `Conv2DProblemSize`.
- **CN:** 顶层类：`MatrixCoord`, `GemmCoord`, `Conv2DProblemSize`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module organizes backend utilities, data structures, or runtime helpers used by cutlass_cppgen.
- **CN:** 该模块组织 cutlass_cppgen 所需的后端工具、数据结构或运行时辅助逻辑。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.c_types`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** none / 无
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
