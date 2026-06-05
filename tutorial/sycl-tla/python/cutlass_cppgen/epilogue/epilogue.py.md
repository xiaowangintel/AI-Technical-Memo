# epilogue.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/epilogue/epilogue.py`
- **EN:** Registry of elementwise epilogues
- **CN:** 模块文档说明：Registry of elementwise epilogues

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

### Lines 33-44 — Docstring

```python
"""
Registry of elementwise epilogues

Elementwise epilogues can be added to many CUTLASS kernels in the CUTLAS Python interface via
code like the following for GEMM:

.. highlight:: python
.. code-block:: python

    plan = cutlass_cppgen.op.Gemm(element=cutlass_cppgen.DataType.f32, layout=cutlass_cppgen.LayoutType.RowMajor)
    plan.activation = cutlass_cppgen.epilogue.relu
"""
```
**EN:** Docstring explains this scope: Registry of elementwise epilogues
**CN:** 文档字符串说明了该作用域的用途：Registry of elementwise epilogues

### Line 46 — From `cutlass_cppgen.backend` import

```python
from cutlass_cppgen.backend import epilogue, device_cc
```
**EN:** Imports `epilogue, device_cc` from `cutlass_cppgen.backend` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend` 导入 `epilogue, device_cc`，以便后续代码在模块级复用共享定义。

### Line 49 — Assign `gelu`

```python
gelu = epilogue.gelu
```
**EN:** Assigns `gelu` from `epilogue.gelu`, establishing state at module scope.
**CN:** 将 `gelu` 赋值为 `epilogue.gelu`，用于在模块级建立状态。

### Line 50 — Assign `hardswish`

```python
hardswish = epilogue.hardswish
```
**EN:** Assigns `hardswish` from `epilogue.hardswish`, establishing state at module scope.
**CN:** 将 `hardswish` 赋值为 `epilogue.hardswish`，用于在模块级建立状态。

### Line 51 — Assign `identity`

```python
identity = epilogue.identity
```
**EN:** Assigns `identity` from `epilogue.identity`, establishing state at module scope.
**CN:** 将 `identity` 赋值为 `epilogue.identity`，用于在模块级建立状态。

### Line 52 — Assign `leaky_relu`

```python
leaky_relu = epilogue.leaky_relu
```
**EN:** Assigns `leaky_relu` from `epilogue.leaky_relu`, establishing state at module scope.
**CN:** 将 `leaky_relu` 赋值为 `epilogue.leaky_relu`，用于在模块级建立状态。

### Line 53 — Assign `relu`

```python
relu = epilogue.relu
```
**EN:** Assigns `relu` from `epilogue.relu`, establishing state at module scope.
**CN:** 将 `relu` 赋值为 `epilogue.relu`，用于在模块级建立状态。

### Line 54 — Assign `sigmoid`

```python
sigmoid = epilogue.sigmoid
```
**EN:** Assigns `sigmoid` from `epilogue.sigmoid`, establishing state at module scope.
**CN:** 将 `sigmoid` 赋值为 `epilogue.sigmoid`，用于在模块级建立状态。

### Line 55 — Assign `silu`

```python
silu = epilogue.silu
```
**EN:** Assigns `silu` from `epilogue.silu`, establishing state at module scope.
**CN:** 将 `silu` 赋值为 `epilogue.silu`，用于在模块级建立状态。

### Line 56 — Assign `tanh`

```python
tanh = epilogue.tanh
```
**EN:** Assigns `tanh` from `epilogue.tanh`, establishing state at module scope.
**CN:** 将 `tanh` 赋值为 `epilogue.tanh`，用于在模块级建立状态。

### Line 59 — Assign `_activations`

```python
_activations = [gelu, hardswish, identity, leaky_relu, relu, sigmoid, silu, tanh]
```
**EN:** Assigns `_activations` from `[gelu, hardswish, identity, leaky_relu, relu, sigmoid, silu, tanh]`, establishing state at module scope.
**CN:** 将 `_activations` 赋值为 `[gelu, hardswish, identity, leaky_relu, relu, sigmoid, silu, tanh]`，用于在模块级建立状态。

### Lines 62-68 — Function `get_activations`

```python
def get_activations() -> list:
    """
    Returns a list of available activation functions

    :return: list of available activation functions
    :rtype: list
    """
```
**EN:** Defines function `get_activations` with parameters ``. Purpose: Returns a list of available activation functions
**CN:** 定义函数 `get_activations`，参数为 ``。 其用途：Returns a list of available activation functions

#### Line 69 — Return

```python
    return _activations
```
**EN:** Returns `_activations` to the caller.
**CN:** 向调用方返回 `_activations`。

### Lines 72-91 — Function `get_activation_epilogue`

```python
def get_activation_epilogue(
    activation,
    element_output,
    elements_per_access,
    element_accumulator,
    element_compute,
):
    """
    Return an epilogue corresponding to the activation function, data types, and alignment
    used in the kernel

    :param activation: elementwise activation function to use
    :param element_output: data type of the output
    :param elements_per_access: alignment of operand C of the kernel
    :type elements_per_access: int
    :param element_accumulator: data type of the accumulated output C
    :param element_compute: data type in which compute operations should be performed

    :return: epilogue functor
    """
```
**EN:** Defines function `get_activation_epilogue` with parameters `activation, element_output, elements_per_access, element_accumulator, element_compute`. Purpose: Return an epilogue corresponding to the activation function, data types, and alignment
**CN:** 定义函数 `get_activation_epilogue`，参数为 `activation, element_output, elements_per_access, element_accumulator, element_compute`。 其用途：Return an epilogue corresponding to the activation function, data types, and alignment

#### Lines 92-95 — Conditional `activation not in _activations`

```python
    if activation not in _activations:
        raise Exception(
            f"Unsupported activation type {activation}. Available activations are: {_activations}"
        )
```
**EN:** Checks `activation not in _activations` and selects the matching branch in function `get_activation_epilogue`.
**CN:** 检查 `activation not in _activations`，并在函数 `get_activation_epilogue` 中选择匹配的分支。

#### Lines 97-108 — Conditional `activation == identity`

```python
    if activation == identity:
        return epilogue.LinearCombination(
            element_output, elements_per_access, element_accumulator, element_compute
        )
    else:
        return epilogue.LinearCombinationGeneric(
            activation,
            element_output,
            elements_per_access,
            element_accumulator,
            element_compute,
        )
```
**EN:** Checks `activation == identity` and selects the matching branch in function `get_activation_epilogue`.
**CN:** 检查 `activation == identity`，并在函数 `get_activation_epilogue` 中选择匹配的分支。

### Lines 111-113 — Docstring

```python
"""
Frontend for EVT that generates epilogue functor through tracing the input function
"""
```
**EN:** Docstring explains this scope: Frontend for EVT that generates epilogue functor through tracing the input function
**CN:** 文档字符串说明了该作用域的用途：Frontend for EVT that generates epilogue functor through tracing the input function

### Line 114 — From `cutlass_cppgen.backend.evt.frontend` import

```python
from cutlass_cppgen.backend.evt.frontend import PythonASTFrontend
```
**EN:** Imports `PythonASTFrontend` from `cutlass_cppgen.backend.evt.frontend` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.frontend` 导入 `PythonASTFrontend`，以便后续代码在模块级复用共享定义。

### Lines 117-146 — Function `trace`

```python
def trace(fn, example_tensors, **kwargs):
    """
    Trace `fn(**example_tensors)` and generates epilogue visitor

    :param fn or str: Python callable or string of the epilogue function
    :param example_tensors: example inputs for fn
    :type example_tensors: dict

    .. highlight:: python
    .. code-block:: python
        import cutlass_cppgen.backend.evt

        # Define epilogue function as Python callable
        def example_fn(accum, C, alpha, beta, gamma):
            D = ((accum + C) * alpha - gamma) / beta
            return D

        # Define the example tensors
        example_inputs = {
            "accum": torch.empty(size=(6, 512, 512), dtype=torch.float16, device="cuda"),
            "C": torch.empty(size=(6, 512, 512), dtype=torch.float16, device="cuda"),
            "alpha": 1.5,
            "beta": 0.5,
            "gamma": 2.5,
            "D": torch.empty(size=(6, 512, 512), dtype=torch.float16, device="cuda")
        }

        # Generate the epilogue functor
        epilogue_visitor = cutlass_cppgen.epilogue.trace(example_fn, example_inputs)
    """
```
**EN:** Defines function `trace` with parameters `fn, example_tensors, **kwargs`. Purpose: Trace `fn(**example_tensors)` and generates epilogue visitor
**CN:** 定义函数 `trace`，参数为 `fn, example_tensors, **kwargs`。 其用途：Trace `fn(**example_tensors)` and generates epilogue visitor

#### Lines 147-176 — Conditional `callable(fn)`

```python
    if callable(fn):
        class EpilogueFunctor(PythonASTFrontend):
            def __init__(self, cc=None, **kwargs):
                if not cc:
                    cc = device_cc()
                super().__init__(cc, **kwargs)
            pass
        setattr(EpilogueFunctor, "__call__", staticmethod(fn))

        epilogue_functor = EpilogueFunctor(**kwargs)
        epilogue_functor.trace(example_tensors)
        return epilogue_functor
    elif isinstance(fn, str):
        class EpilogueFunctor(PythonASTFrontend):
            def __init__(self, cc=None, **kwargs):
                self.source = textwrap.dedent(fn)
                if not cc:
                    cc = device_cc()
                super().__init__(cc, **kwargs)

            def parse(self, example_inputs) -> None:
                self.example_inputs = example_inputs
                self.ast = ast.parse(self.source)
                self.visit(self.ast)

        epilogue_functor = EpilogueFunctor(**kwargs)
        epilogue_functor.trace(example_tensors)
        return epilogue_functor
    else:
        raise NotImplementedError("Expect a callable Python function")
```
**EN:** Checks `callable(fn)` and selects the matching branch in function `trace`.
**CN:** 检查 `callable(fn)`，并在函数 `trace` 中选择匹配的分支。

## Key Concepts / 关键概念

- **EN:** Top-level classes: none.
- **CN:** 顶层类：无。
- **EN:** Top-level functions: `get_activations`, `get_activation_epilogue`, `trace`.
- **CN:** 顶层函数：`get_activations`, `get_activation_epilogue`, `trace`。
- **EN:** This module defines epilogue-side abstractions and fused post-processing behavior.
- **CN:** 该模块定义 epilogue 侧抽象以及融合后的后处理行为。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend`, `cutlass_cppgen.backend.evt.frontend`
- **Standard & third-party / 标准库与第三方:** none / 无
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
