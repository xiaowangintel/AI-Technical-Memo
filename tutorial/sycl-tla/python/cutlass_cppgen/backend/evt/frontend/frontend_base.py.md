# frontend_base.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/frontend/frontend_base.py`
- **EN:** Base class for Python EVT Frontend
- **CN:** 模块文档说明：Base class for Python EVT Frontend

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
Base class for Python EVT Frontend
"""
```
**EN:** Docstring explains this scope: Base class for Python EVT Frontend
**CN:** 文档字符串说明了该作用域的用途：Base class for Python EVT Frontend

### Line 37 — From `typing` import

```python
from typing import Union
```
**EN:** Imports `Union` from `typing` to reuse shared definitions at module scope.
**CN:** 从 `typing` 导入 `Union`，以便后续代码在模块级复用共享定义。

### Line 39 — From `cutlass_library` import

```python
from cutlass_library import DataType
```
**EN:** Imports `DataType` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `DataType`，以便后续代码在模块级复用共享定义。

### Lines 40-46 — From `cutlass_cppgen.backend.evt.ir` import

```python
from cutlass_cppgen.backend.evt.ir import (
    ComputeNode,
    DAGIR,
    LayoutNode,
    LoadNode,
    StoreNode,
)
```
**EN:** Imports `ComputeNode, DAGIR, LayoutNode, LoadNode, StoreNode` from `cutlass_cppgen.backend.evt.ir` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir` 导入 `ComputeNode, DAGIR, LayoutNode, LoadNode, StoreNode`，以便后续代码在模块级复用共享定义。

### Lines 47-58 — From `cutlass_cppgen.backend.evt.passes` import

```python
from cutlass_cppgen.backend.evt.passes import (
    EVTGraphDrawer,
    EVTPassManager,
    GetSmemSize,
    PassDAG2Tree,
    PassGetArgumentType,
    PassGetImpl,
    PassFixElementD,
    PassLayoutManipulateElimination,
    PassPreprocessRed,
    PassShapeTypePropagation,
)
```
**EN:** Imports `EVTGraphDrawer, EVTPassManager, GetSmemSize, PassDAG2Tree, PassGetArgumentType, PassGetImpl, PassFixElementD, PassLayoutManipulateElimination, PassPreprocessRed, PassShapeTypePropagation` from `cutlass_cppgen.backend.evt.passes` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.passes` 导入 `EVTGraphDrawer, EVTPassManager, GetSmemSize, PassDAG2Tree, PassGetArgumentType, PassGetImpl, PassFixElementD, PassLayoutManipulateElimination, PassPreprocessRed, PassShapeTypePropagation`，以便后续代码在模块级复用共享定义。

### Line 59 — From `cutlass_cppgen.backend.evt.passes.util` import

```python
from cutlass_cppgen.backend.evt.passes.util import cc_map
```
**EN:** Imports `cc_map` from `cutlass_cppgen.backend.evt.passes.util` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.passes.util` 导入 `cc_map`，以便后续代码在模块级复用共享定义。

### Line 60 — From `cutlass_cppgen.backend.utils` import

```python
from cutlass_cppgen.backend.utils import device_cc
```
**EN:** Imports `device_cc` from `cutlass_cppgen.backend.utils` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.utils` 导入 `device_cc`，以便后续代码在模块级复用共享定义。

### Line 61 — From `cutlass_cppgen.epilogue.evt_ops` import

```python
from cutlass_cppgen.epilogue.evt_ops import permute, reshape
```
**EN:** Imports `permute, reshape` from `cutlass_cppgen.epilogue.evt_ops` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.epilogue.evt_ops` 导入 `permute, reshape`，以便后续代码在模块级复用共享定义。

### Line 62 — From `cutlass_cppgen.utils.datatypes` import

```python
from cutlass_cppgen.utils.datatypes import library_type
```
**EN:** Imports `library_type` from `cutlass_cppgen.utils.datatypes` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.utils.datatypes` 导入 `library_type`，以便后续代码在模块级复用共享定义。

### Line 65 — Class `EVTFrontendBase`

```python
class EVTFrontendBase:
```
**EN:** Declares class `EVTFrontendBase` deriving from `object`.
**CN:** 声明类 `EVTFrontendBase`，其基类为 `object`。

#### Lines 66-69 — Assign `layout_fns`

```python
    layout_fns = {
        "permute": permute,
        "reshape": reshape
    }
```
**EN:** Assigns `layout_fns` from `{'permute': permute, 'reshape': reshape}`, establishing state in class `EVTFrontendBase`.
**CN:** 将 `layout_fns` 赋值为 `{'permute': permute, 'reshape': reshape}`，用于在类 `EVTFrontendBase` 中建立状态。

#### Line 71 — Function `__init__`

```python
    def __init__(self, cc, element_compute=DataType.f32, additional_passes=[], **kwargs) -> None:
```
**EN:** Defines function `__init__` with parameters `self, cc, element_compute, additional_passes, **kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, cc, element_compute, additional_passes, **kwargs`。

##### Line 72 — Assign `self.cc`

```python
        self.cc = cc
```
**EN:** Assigns `self.cc` from `cc`, establishing state in function `__init__`.
**CN:** 将 `self.cc` 赋值为 `cc`，用于在函数 `__init__` 中建立状态。

##### Line 73 — Assign `self.element_compute`

```python
        self.element_compute = library_type(element_compute)
```
**EN:** Assigns `self.element_compute` from `library_type(element_compute)`, establishing state in function `__init__`.
**CN:** 将 `self.element_compute` 赋值为 `library_type(element_compute)`，用于在函数 `__init__` 中建立状态。

##### Line 74 — Assign `self.dag_ir`

```python
        self.dag_ir = DAGIR(self.cc, self.element_compute)
```
**EN:** Assigns `self.dag_ir` from `DAGIR(self.cc, self.element_compute)`, establishing state in function `__init__`.
**CN:** 将 `self.dag_ir` 赋值为 `DAGIR(self.cc, self.element_compute)`，用于在函数 `__init__` 中建立状态。

##### Line 75 — Assign `self.compute_cnt`

```python
        self.compute_cnt = 0
```
**EN:** Assigns `self.compute_cnt` from `0`, establishing state in function `__init__`.
**CN:** 将 `self.compute_cnt` 赋值为 `0`，用于在函数 `__init__` 中建立状态。

##### Line 76 — Assign `self.layout_cnt`

```python
        self.layout_cnt = 0
```
**EN:** Assigns `self.layout_cnt` from `0`, establishing state in function `__init__`.
**CN:** 将 `self.layout_cnt` 赋值为 `0`，用于在函数 `__init__` 中建立状态。

##### Line 77 — Assign `self.imm_cnt`

```python
        self.imm_cnt = 0
```
**EN:** Assigns `self.imm_cnt` from `0`, establishing state in function `__init__`.
**CN:** 将 `self.imm_cnt` 赋值为 `0`，用于在函数 `__init__` 中建立状态。

##### Lines 79-89 — Assign `self.pass_manager`

```python
        self.pass_manager = EVTPassManager(
            self.dag_ir,
            [
                PassPreprocessRed,
                PassGetArgumentType,
                PassShapeTypePropagation,
                PassLayoutManipulateElimination,
                PassGetImpl,
                PassDAG2Tree,
                PassFixElementD
            ] + additional_passes)
```
**EN:** Assigns `self.pass_manager` from `EVTPassManager(self.dag_ir, [PassPreprocessRed, PassGetArgumentType, PassShapeTypePropagation, PassLayoutManipulateElimination, PassGetImpl, PassDAG2Tree, Pa...`, establishing state in function `__init__`.
**CN:** 将 `self.pass_manager` 赋值为 `EVTPassManager(self.dag_ir, [PassPreprocessRed, PassGetArgumentType, PassShapeTypePropagation, PassLayoutManipulateElimination, PassGetImpl, PassDAG2Tree, Pa...`，用于在函数 `__init__` 中建立状态。

##### Lines 91-94 — Conditional `self.cc == 80`

```python
        if self.cc == 80:
            self._epilogue_stages = 1
        else:
            self._epilogue_stages = None
```
**EN:** Checks `self.cc == 80` and selects the matching branch in function `__init__`.
**CN:** 检查 `self.cc == 80`，并在函数 `__init__` 中选择匹配的分支。

#### Lines 96-97 — Function `epilogue_stages`

```python
    @property
    def epilogue_stages(self):
```
**EN:** Defines function `epilogue_stages` with parameters `self`.
**CN:** 定义函数 `epilogue_stages`，参数为 `self`。

##### Line 98 — Return

```python
        return self._epilogue_stages
```
**EN:** Returns `self._epilogue_stages` to the caller.
**CN:** 向调用方返回 `self._epilogue_stages`。

#### Lines 100-101 — Function `epilogue_stages`

```python
    @epilogue_stages.setter
    def epilogue_stages(self, stages):
```
**EN:** Defines function `epilogue_stages` with parameters `self, stages`.
**CN:** 定义函数 `epilogue_stages`，参数为 `self, stages`。

##### Line 102 — Assign `self._epilogue_stages`

```python
        self._epilogue_stages = stages
```
**EN:** Assigns `self._epilogue_stages` from `stages`, establishing state in function `epilogue_stages`.
**CN:** 将 `self._epilogue_stages` 赋值为 `stages`，用于在函数 `epilogue_stages` 中建立状态。

#### Line 105 — Function `parse`

```python
    def parse(self, *args, **kwargs):
```
**EN:** Defines function `parse` with parameters `self, *args, **kwargs`.
**CN:** 定义函数 `parse`，参数为 `self, *args, **kwargs`。

##### Line 106 — Raise exception

```python
        raise NotImplementedError(f"The 'parse' function must be overloaded in frontend class")
```
**EN:** Raises `NotImplementedError(f"The 'parse' function must be overloaded in frontend class")` to signal an error or unsupported state.
**CN:** 抛出 `NotImplementedError(f"The 'parse' function must be overloaded in frontend class")`，用于报告错误或不支持的状态。

#### Lines 108-109 — Function `trace`

```python
    def trace(self, *args, **kwargs):
        # Parse the input
```
**EN:** Defines function `trace` with parameters `self, *args, **kwargs`.
**CN:** 定义函数 `trace`，参数为 `self, *args, **kwargs`。

##### Line 110 — Call `self.parse`

```python
        self.parse(*args, **kwargs)
```
**EN:** Calls `self.parse` for side effects or initialization work in function `trace`.
**CN:** 调用 `self.parse` 执行副作用或初始化逻辑；该语句位于在函数 `trace` 中。

##### Line 112 — Comment or spacing block

```python
        # Verify the DAG IR to ensure that "D" is the output node with out_degree = 0
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 113-117 — Conditional `cc_map[self.cc] in [12, 20, 90, 100]`

```python
        if cc_map[self.cc] in [12, 20, 90, 100]:
            if (self.dag_ir.out_degree("D") != 0):
                raise RuntimeError(
                    f"On SM90 or higher, D is expected to be a output node with 0 users to "
                    f"enable smem reuse between C and D, but got {self.dag_ir.out_degree('D')}")
```
**EN:** Checks `cc_map[self.cc] in [12, 20, 90, 100]` and selects the matching branch in function `trace`.
**CN:** 检查 `cc_map[self.cc] in [12, 20, 90, 100]`，并在函数 `trace` 中选择匹配的分支。

##### Line 119 — Comment or spacing block

```python
        # Run the passes
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 120 — Call `self.pass_manager`

```python
        self.pass_manager()
```
**EN:** Calls `self.pass_manager` for side effects or initialization work in function `trace`.
**CN:** 调用 `self.pass_manager` 执行副作用或初始化逻辑；该语句位于在函数 `trace` 中。

##### Line 121 — Comment or spacing block

```python
        # Set the epilogue type
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 122 — Assign `self.epilogue_thread_type`

```python
        self.epilogue_thread_type = self.dag_ir.epilogue_thread_type
```
**EN:** Assigns `self.epilogue_thread_type` from `self.dag_ir.epilogue_thread_type`, establishing state in function `trace`.
**CN:** 将 `self.epilogue_thread_type` 赋值为 `self.dag_ir.epilogue_thread_type`，用于在函数 `trace` 中建立状态。

##### Lines 123-125 — Conditional `cc_map[self.cc] in [12, 20, 90, 100]`

```python
        if cc_map[self.cc] in [12, 20, 90, 100]:
            self.arg_c_type = self.dag_ir.arg_c_type
            self.arg_d_type = self.dag_ir.arg_d_type
```
**EN:** Checks `cc_map[self.cc] in [12, 20, 90, 100]` and selects the matching branch in function `trace`.
**CN:** 检查 `cc_map[self.cc] in [12, 20, 90, 100]`，并在函数 `trace` 中选择匹配的分支。

##### Line 126 — Assign `self.reduction_names`

```python
        self.reduction_names = self.dag_ir.reduction_names
```
**EN:** Assigns `self.reduction_names` from `self.dag_ir.reduction_names`, establishing state in function `trace`.
**CN:** 将 `self.reduction_names` 赋值为 `self.dag_ir.reduction_names`，用于在函数 `trace` 中建立状态。

#### Lines 128-130 — Comment or spacing block

```python
    #
    # Helper functions for DAG IR manipulation
    #
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 132 — Function `add_node`

```python
    def add_node(self, node):
```
**EN:** Defines function `add_node` with parameters `self, node`.
**CN:** 定义函数 `add_node`，参数为 `self, node`。

##### Line 133 — Call `self.dag_ir.add_node`

```python
        self.dag_ir.add_node(node)
```
**EN:** Calls `self.dag_ir.add_node` for side effects or initialization work in function `add_node`.
**CN:** 调用 `self.dag_ir.add_node` 执行副作用或初始化逻辑；该语句位于在函数 `add_node` 中。

#### Line 135 — Function `add_edge`

```python
    def add_edge(self, src, tgt, weight=0):
```
**EN:** Defines function `add_edge` with parameters `self, src, tgt, weight`.
**CN:** 定义函数 `add_edge`，参数为 `self, src, tgt, weight`。

##### Line 136 — Call `self.dag_ir.add_edge`

```python
        self.dag_ir.add_edge(src, tgt, weight=weight)
```
**EN:** Calls `self.dag_ir.add_edge` for side effects or initialization work in function `add_edge`.
**CN:** 调用 `self.dag_ir.add_edge` 执行副作用或初始化逻辑；该语句位于在函数 `add_edge` 中。

#### Lines 138-141 — Function `set_tensor`

```python
    def set_tensor(self, node_name, example):
        """
        Add an example tensor to node {node_name} in the DAG IR
        """
```
**EN:** Defines function `set_tensor` with parameters `self, node_name, example`. Purpose: Add an example tensor to node {node_name} in the DAG IR
**CN:** 定义函数 `set_tensor`，参数为 `self, node_name, example`。 其用途：Add an example tensor to node {node_name} in the DAG IR

##### Line 142 — Assign `meta`

```python
        meta = self.dag_ir.get_node_meta(node_name)
```
**EN:** Assigns `meta` from `self.dag_ir.get_node_meta(node_name)`, establishing state in function `set_tensor`.
**CN:** 将 `meta` 赋值为 `self.dag_ir.get_node_meta(node_name)`，用于在函数 `set_tensor` 中建立状态。

##### Line 143 — Assign `meta.tensor`

```python
        meta.tensor = {"tensor": example}
```
**EN:** Assigns `meta.tensor` from `{'tensor': example}`, establishing state in function `set_tensor`.
**CN:** 将 `meta.tensor` 赋值为 `{'tensor': example}`，用于在函数 `set_tensor` 中建立状态。

#### Lines 145-148 — Function `set_store_tensor`

```python
    def set_store_tensor(self, node_name, example):
        """
        Add an example tensor to node {node_name} in the DAG IR
        """
```
**EN:** Defines function `set_store_tensor` with parameters `self, node_name, example`. Purpose: Add an example tensor to node {node_name} in the DAG IR
**CN:** 定义函数 `set_store_tensor`，参数为 `self, node_name, example`。 其用途：Add an example tensor to node {node_name} in the DAG IR

##### Line 149 — Assign `meta`

```python
        meta = self.dag_ir.get_node_meta(node_name)
```
**EN:** Assigns `meta` from `self.dag_ir.get_node_meta(node_name)`, establishing state in function `set_store_tensor`.
**CN:** 将 `meta` 赋值为 `self.dag_ir.get_node_meta(node_name)`，用于在函数 `set_store_tensor` 中建立状态。

##### Line 150 — Assign `meta.store_tensor`

```python
        meta.store_tensor = {"tensor": example}
```
**EN:** Assigns `meta.store_tensor` from `{'tensor': example}`, establishing state in function `set_store_tensor`.
**CN:** 将 `meta.store_tensor` 赋值为 `{'tensor': example}`，用于在函数 `set_store_tensor` 中建立状态。

#### Lines 152-155 — Function `mark_output`

```python
    def mark_output(self, node_name):
        """
        Mark a store node as output
        """
```
**EN:** Defines function `mark_output` with parameters `self, node_name`. Purpose: Mark a store node as output
**CN:** 定义函数 `mark_output`，参数为 `self, node_name`。 其用途：Mark a store node as output

##### Line 156 — Assign `meta`

```python
        meta = self.dag_ir.get_node_meta(node_name)
```
**EN:** Assigns `meta` from `self.dag_ir.get_node_meta(node_name)`, establishing state in function `mark_output`.
**CN:** 将 `meta` 赋值为 `self.dag_ir.get_node_meta(node_name)`，用于在函数 `mark_output` 中建立状态。

##### Lines 157-160 — Conditional `not isinstance(meta, StoreNode)`

```python
        if not isinstance(meta, StoreNode):
            raise ValueError(
                f"Only StoreNodes can be marked as output. "
                f"Got {type(meta).__name__}: {node_name}")
```
**EN:** Checks `not isinstance(meta, StoreNode)` and selects the matching branch in function `mark_output`.
**CN:** 检查 `not isinstance(meta, StoreNode)`，并在函数 `mark_output` 中选择匹配的分支。

##### Line 161 — Assign `meta.is_output`

```python
        meta.is_output = True
```
**EN:** Assigns `meta.is_output` from `True`, establishing state in function `mark_output`.
**CN:** 将 `meta.is_output` 赋值为 `True`，用于在函数 `mark_output` 中建立状态。

#### Line 163 — Comment or spacing block

```python
    # Add node with specific type
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 165-172 — Function `add_load_node`

```python
    def add_load_node(self, name, example):
        """
        Add a Load node to DAG IR
        :param name: name of the loaded variable
        :type name: str
        :param example: example input
        :type example: np.ndarray|torch.Tensor|cupy.ndarray|float
        """
```
**EN:** Defines function `add_load_node` with parameters `self, name, example`. Purpose: Add a Load node to DAG IR
**CN:** 定义函数 `add_load_node`，参数为 `self, name, example`。 其用途：Add a Load node to DAG IR

##### Lines 173-174 — Conditional `name is None`

```python
        if name is None:
            raise ValueError(f"Name is not provided.")
```
**EN:** Checks `name is None` and selects the matching branch in function `add_load_node`.
**CN:** 检查 `name is None`，并在函数 `add_load_node` 中选择匹配的分支。

##### Lines 175-176 — Conditional `example is None`

```python
        if example is None:
            raise ValueError(f"Example input for {name} is not provided.")
```
**EN:** Checks `example is None` and selects the matching branch in function `add_load_node`.
**CN:** 检查 `example is None`，并在函数 `add_load_node` 中选择匹配的分支。

##### Line 177 — Assign `load_node`

```python
        load_node = LoadNode(name)
```
**EN:** Assigns `load_node` from `LoadNode(name)`, establishing state in function `add_load_node`.
**CN:** 将 `load_node` 赋值为 `LoadNode(name)`，用于在函数 `add_load_node` 中建立状态。

##### Line 178 — Assign `load_node.tensor`

```python
        load_node.tensor = {"tensor": example}
```
**EN:** Assigns `load_node.tensor` from `{'tensor': example}`, establishing state in function `add_load_node`.
**CN:** 将 `load_node.tensor` 赋值为 `{'tensor': example}`，用于在函数 `add_load_node` 中建立状态。

##### Line 179 — Comment or spacing block

```python
        # Special logics for accumulator
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 180-185 — Conditional `name == 'accum'`

```python
        if name == "accum":
            if load_node.tensor.rank == 2:
                new_shape = tuple([1, ] + list(load_node.tensor.shape))
                load_node.tensor.broadcast(new_shape)
            elif load_node.tensor.rank < 2 or load_node.tensor.rank > 3:
                raise ValueError(f"Expect example inputs for 'accum' be a rank-2 or rank-3 tensor. Got {load_node.tensor.shape}.")
```
**EN:** Checks `name == 'accum'` and selects the matching branch in function `add_load_node`.
**CN:** 检查 `name == 'accum'`，并在函数 `add_load_node` 中选择匹配的分支。

##### Line 186 — Call `self.add_node`

```python
        self.add_node(load_node)
```
**EN:** Calls `self.add_node` for side effects or initialization work in function `add_load_node`.
**CN:** 调用 `self.add_node` 执行副作用或初始化逻辑；该语句位于在函数 `add_load_node` 中。

#### Lines 188-193 — Function `add_imm`

```python
    def add_imm(self, value: Union[float,int]):
        """
        Add an immediate scalar value to DAG IR
        :param value: the value of the immediate scalar
        :type value: float
        """
```
**EN:** Defines function `add_imm` with parameters `self, value`. Purpose: Add an immediate scalar value to DAG IR
**CN:** 定义函数 `add_imm`，参数为 `self, value`。 其用途：Add an immediate scalar value to DAG IR

##### Lines 194-197 — Exception handling

```python
        try:
            value = float(value)
        except:
            raise ValueError(f"{type(value).__name__} cannot be converted to float.")
```
**EN:** Wraps the enclosed logic with exception handling and optional cleanup paths.
**CN:** 使用异常处理包装该逻辑，并提供回退或清理路径。

##### Line 199 — Assign `name`

```python
        name = f"imm_{value}_k{self.imm_cnt}".replace('.', '_')
```
**EN:** Assigns `name` from `f'imm_{value}_k{self.imm_cnt}'.replace('.', '_')`, establishing state in function `add_imm`.
**CN:** 将 `name` 赋值为 `f'imm_{value}_k{self.imm_cnt}'.replace('.', '_')`，用于在函数 `add_imm` 中建立状态。

##### Line 200 — Update `self.imm_cnt`

```python
        self.imm_cnt += 1
```
**EN:** Updates `self.imm_cnt` with `+=` using `1`.
**CN:** 使用 `1` 对 `self.imm_cnt` 执行 `+=` 更新。

##### Line 201 — Assign `load_node`

```python
        load_node = LoadNode(name)
```
**EN:** Assigns `load_node` from `LoadNode(name)`, establishing state in function `add_imm`.
**CN:** 将 `load_node` 赋值为 `LoadNode(name)`，用于在函数 `add_imm` 中建立状态。

##### Line 202 — Assign `load_node.tensor`

```python
        load_node.tensor = {"tensor": value, "is_constant": True}
```
**EN:** Assigns `load_node.tensor` from `{'tensor': value, 'is_constant': True}`, establishing state in function `add_imm`.
**CN:** 将 `load_node.tensor` 赋值为 `{'tensor': value, 'is_constant': True}`，用于在函数 `add_imm` 中建立状态。

##### Line 203 — Call `self.add_node`

```python
        self.add_node(load_node)
```
**EN:** Calls `self.add_node` for side effects or initialization work in function `add_imm`.
**CN:** 调用 `self.add_node` 执行副作用或初始化逻辑；该语句位于在函数 `add_imm` 中。

##### Line 204 — Return

```python
        return name
```
**EN:** Returns `name` to the caller.
**CN:** 向调用方返回 `name`。

#### Lines 206-213 — Function `add_compute_node`

```python
    def add_compute_node(self, op, name=None):
        """
        Add a compute node.
        :param op: the computation op
        :param name: the node name (optional)
        :type name: str
        :return: the name of the compute node
        """
```
**EN:** Defines function `add_compute_node` with parameters `self, op, name`. Purpose: Add a compute node.
**CN:** 定义函数 `add_compute_node`，参数为 `self, op, name`。 其用途：Add a compute node.

##### Lines 214-216 — Conditional `name is None`

```python
        if name is None:
            name = f"compute_{self.compute_cnt}"
            self.compute_cnt += 1
```
**EN:** Checks `name is None` and selects the matching branch in function `add_compute_node`.
**CN:** 检查 `name is None`，并在函数 `add_compute_node` 中选择匹配的分支。

##### Lines 217-220 — Assign `compute_node`

```python
        compute_node = ComputeNode(
            name=name, fn=op,
            element_output=self.element_compute,
            element_compute=self.element_compute)
```
**EN:** Assigns `compute_node` from `ComputeNode(name=name, fn=op, element_output=self.element_compute, element_compute=self.element_compute)`, establishing state in function `add_compute_node`.
**CN:** 将 `compute_node` 赋值为 `ComputeNode(name=name, fn=op, element_output=self.element_compute, element_compute=self.element_compute)`，用于在函数 `add_compute_node` 中建立状态。

##### Line 221 — Call `self.add_node`

```python
        self.add_node(compute_node)
```
**EN:** Calls `self.add_node` for side effects or initialization work in function `add_compute_node`.
**CN:** 调用 `self.add_node` 执行副作用或初始化逻辑；该语句位于在函数 `add_compute_node` 中。

##### Line 222 — Return

```python
        return compute_node.name
```
**EN:** Returns `compute_node.name` to the caller.
**CN:** 向调用方返回 `compute_node.name`。

#### Lines 224-232 — Function `add_layout_node`

```python
    def add_layout_node(self, op, kwargs, name=None):
        """
        Add a layout node.
        :param op: the layout op
        :type op: evt_ops
        :param name: the node name (optional)
        :type name: str
        :return: the name of the layout node
        """
```
**EN:** Defines function `add_layout_node` with parameters `self, op, kwargs, name`. Purpose: Add a layout node.
**CN:** 定义函数 `add_layout_node`，参数为 `self, op, kwargs, name`。 其用途：Add a layout node.

##### Lines 233-235 — Conditional `name is None`

```python
        if name is None:
            name = f"layout_{self.layout_cnt}"
            self.layout_cnt += 1
```
**EN:** Checks `name is None` and selects the matching branch in function `add_layout_node`.
**CN:** 检查 `name is None`，并在函数 `add_layout_node` 中选择匹配的分支。

##### Line 236 — Assign `layout_node`

```python
        layout_node = LayoutNode(name=name, fn=op, kwargs=kwargs)
```
**EN:** Assigns `layout_node` from `LayoutNode(name=name, fn=op, kwargs=kwargs)`, establishing state in function `add_layout_node`.
**CN:** 将 `layout_node` 赋值为 `LayoutNode(name=name, fn=op, kwargs=kwargs)`，用于在函数 `add_layout_node` 中建立状态。

##### Line 237 — Call `self.add_node`

```python
        self.add_node(layout_node)
```
**EN:** Calls `self.add_node` for side effects or initialization work in function `add_layout_node`.
**CN:** 调用 `self.add_node` 执行副作用或初始化逻辑；该语句位于在函数 `add_layout_node` 中。

##### Line 238 — Return

```python
        return layout_node.name
```
**EN:** Returns `layout_node.name` to the caller.
**CN:** 向调用方返回 `layout_node.name`。

#### Line 240 — Function `add_store_node`

```python
    def add_store_node(self, name):
```
**EN:** Defines function `add_store_node` with parameters `self, name`.
**CN:** 定义函数 `add_store_node`，参数为 `self, name`。

##### Line 241 — Assign `store_node`

```python
        store_node = StoreNode(name)
```
**EN:** Assigns `store_node` from `StoreNode(name)`, establishing state in function `add_store_node`.
**CN:** 将 `store_node` 赋值为 `StoreNode(name)`，用于在函数 `add_store_node` 中建立状态。

##### Line 242 — Call `self.add_node`

```python
        self.add_node(store_node)
```
**EN:** Calls `self.add_node` for side effects or initialization work in function `add_store_node`.
**CN:** 调用 `self.add_node` 执行副作用或初始化逻辑；该语句位于在函数 `add_store_node` 中。

#### Lines 244-246 — Comment or spacing block

```python
    #
    # Visualization The DAG IR
    #
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 248-252 — Function `visualize`

```python
    def visualize(self, name="dag_ir"):
        """
        Visualize the dag ir with svg file
        :param name: the name of the graph
        """
```
**EN:** Defines function `visualize` with parameters `self, name`. Purpose: Visualize the dag ir with svg file
**CN:** 定义函数 `visualize`，参数为 `self, name`。 其用途：Visualize the dag ir with svg file

##### Line 253 — Assign `drawer`

```python
        drawer = EVTGraphDrawer(self.dag_ir, name)
```
**EN:** Assigns `drawer` from `EVTGraphDrawer(self.dag_ir, name)`, establishing state in function `visualize`.
**CN:** 将 `drawer` 赋值为 `EVTGraphDrawer(self.dag_ir, name)`，用于在函数 `visualize` 中建立状态。

##### Lines 254-261 — Exception handling

```python
        try:
            for name, graph in drawer.get_dot_graph():
                graph.write_svg(f"./{name}.svg")
        except:
            raise RuntimeError(
                "'dot' is not found in path. GraphDrawer is disabled. "
                "Please install it with 'sudo apt-get install graphviz'."
            )
```
**EN:** Wraps the enclosed logic with exception handling and optional cleanup paths.
**CN:** 使用异常处理包装该逻辑，并提供回退或清理路径。

#### Lines 263-265 — Comment or spacing block

```python
    #
    # Get shared memory size
    #
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 267-270 — Function `get_smem_size`

```python
    def get_smem_size(self, tile_description):
        """
        Get the shared memory size of the epilogue
        """
```
**EN:** Defines function `get_smem_size` with parameters `self, tile_description`. Purpose: Get the shared memory size of the epilogue
**CN:** 定义函数 `get_smem_size`，参数为 `self, tile_description`。 其用途：Get the shared memory size of the epilogue

##### Line 271 — Assign `smem_size`

```python
        smem_size = GetSmemSize(self.dag_ir)(tile_description)
```
**EN:** Assigns `smem_size` from `GetSmemSize(self.dag_ir)(tile_description)`, establishing state in function `get_smem_size`.
**CN:** 将 `smem_size` 赋值为 `GetSmemSize(self.dag_ir)(tile_description)`，用于在函数 `get_smem_size` 中建立状态。

##### Line 272 — Return

```python
        return smem_size
```
**EN:** Returns `smem_size` to the caller.
**CN:** 向调用方返回 `smem_size`。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `EVTFrontendBase`.
- **CN:** 顶层类：`EVTFrontendBase`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.evt.ir`, `cutlass_cppgen.backend.evt.passes`, `cutlass_cppgen.backend.evt.passes.util`, `cutlass_cppgen.backend.utils`, `cutlass_cppgen.epilogue.evt_ops`, `cutlass_cppgen.utils.datatypes`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** `typing`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
