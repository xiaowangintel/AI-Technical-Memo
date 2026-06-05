# pass_shape_type_propagation.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/passes/pass_shape_type_propagation.py`
- **EN:** Shape and type propagation pass
- **CN:** 模块文档说明：Shape and type propagation pass

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
Shape and type propagation pass
"""
```
**EN:** Docstring explains this scope: Shape and type propagation pass
**CN:** 文档字符串说明了该作用域的用途：Shape and type propagation pass

### Line 37 — From `cutlass_cppgen.backend.evt.ir.node` import

```python
from cutlass_cppgen.backend.evt.ir.node import NodeBase
```
**EN:** Imports `NodeBase` from `cutlass_cppgen.backend.evt.ir.node` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir.node` 导入 `NodeBase`，以便后续代码在模块级复用共享定义。

### Line 38 — From `cutlass_cppgen.backend.evt.passes.pass_manager` import

```python
from cutlass_cppgen.backend.evt.passes.pass_manager import EVTPassBase
```
**EN:** Imports `EVTPassBase` from `cutlass_cppgen.backend.evt.passes.pass_manager` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.passes.pass_manager` 导入 `EVTPassBase`，以便后续代码在模块级复用共享定义。

### Line 39 — From `cutlass_cppgen.backend.evt.passes.pass_preprocess_red` import

```python
from cutlass_cppgen.backend.evt.passes.pass_preprocess_red import PassPreprocessRed
```
**EN:** Imports `PassPreprocessRed` from `cutlass_cppgen.backend.evt.passes.pass_preprocess_red` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.passes.pass_preprocess_red` 导入 `PassPreprocessRed`，以便后续代码在模块级复用共享定义。

### Lines 42-45 — Class `PassShapeTypePropagation`

```python
class PassShapeTypePropagation(EVTPassBase):
    """
    Propagate the shape and type of all nodes
    """
```
**EN:** Declares class `PassShapeTypePropagation` deriving from `EVTPassBase`. Purpose: Propagate the shape and type of all nodes
**CN:** 声明类 `PassShapeTypePropagation`，其基类为 `EVTPassBase`。 其用途：Propagate the shape and type of all nodes

#### Line 46 — Assign `dependencies`

```python
    dependencies = [PassPreprocessRed]
```
**EN:** Assigns `dependencies` from `[PassPreprocessRed]`, establishing state in class `PassShapeTypePropagation`.
**CN:** 将 `dependencies` 赋值为 `[PassPreprocessRed]`，用于在类 `PassShapeTypePropagation` 中建立状态。

#### Lines 48-49 — Function `call`

```python
    def call(self):
        # Propagate the node shape and type
```
**EN:** Defines function `call` with parameters `self`.
**CN:** 定义函数 `call`，参数为 `self`。

##### Lines 50-54 — Loop over `self.dag_ir.nodes_topological_order()`

```python
        for node in self.dag_ir.nodes_topological_order():
            node_meta: NodeBase = self.dag_ir.get_node_meta(node)
            input_node_metas = self.dag_ir.get_all_inputs_meta(node)
            node_meta.type_propagation(input_node_metas)
            node_meta.shape_propagation(input_node_metas)
```
**EN:** Iterates `node` over `self.dag_ir.nodes_topological_order()` to repeat a processing step.
**CN:** 让 `node` 遍历 `self.dag_ir.nodes_topological_order()`，从而重复执行处理步骤。

##### Lines 56-59 — Loop over `reversed(self.dag_ir.nodes_topological_order())`

```python
        for node in reversed(self.dag_ir.nodes_topological_order()):
            node_meta: NodeBase = self.dag_ir.get_node_meta(node)
            input_node_metas = self.dag_ir.get_all_inputs_meta(node)
            node_meta.broadcast_propagation(input_node_metas)
```
**EN:** Iterates `node` over `reversed(self.dag_ir.nodes_topological_order())` to repeat a processing step.
**CN:** 让 `node` 遍历 `reversed(self.dag_ir.nodes_topological_order())`，从而重复执行处理步骤。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `PassShapeTypePropagation`.
- **CN:** 顶层类：`PassShapeTypePropagation`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.evt.ir.node`, `cutlass_cppgen.backend.evt.passes.pass_manager`, `cutlass_cppgen.backend.evt.passes.pass_preprocess_red`
- **Standard & third-party / 标准库与第三方:** none / 无
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
