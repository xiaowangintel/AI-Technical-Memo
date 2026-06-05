# pass_preprocess_red.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/passes/pass_preprocess_red.py`
- **EN:** Preprocess the reduction nodes.
- **CN:** 模块文档说明：Preprocess the reduction nodes.

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

### Lines 33-39 — Docstring

```python
"""
Preprocess the reduction nodes.

The parser treats reduction as Compute(op=(reg_reduce_fn, gmem_reduce_fn)) - Store()
This pass fuses these into a single store node, and then replaces all uses of the
current node with the new store node.
"""
```
**EN:** Docstring explains this scope: Preprocess the reduction nodes.
**CN:** 文档字符串说明了该作用域的用途：Preprocess the reduction nodes.

### Line 41 — From `cutlass_cppgen.backend.evt.ir` import

```python
from cutlass_cppgen.backend.evt.ir import ComputeNode, StoreNode
```
**EN:** Imports `ComputeNode, StoreNode` from `cutlass_cppgen.backend.evt.ir` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir` 导入 `ComputeNode, StoreNode`，以便后续代码在模块级复用共享定义。

### Line 42 — From `cutlass_cppgen.backend.evt.passes.pass_manager` import

```python
from cutlass_cppgen.backend.evt.passes.pass_manager import EVTPassBase
```
**EN:** Imports `EVTPassBase` from `cutlass_cppgen.backend.evt.passes.pass_manager` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.passes.pass_manager` 导入 `EVTPassBase`，以便后续代码在模块级复用共享定义。

### Lines 45-48 — Class `PassPreprocessRed`

```python
class PassPreprocessRed(EVTPassBase):
    """
    Preprocess red nodes
    """
```
**EN:** Declares class `PassPreprocessRed` deriving from `EVTPassBase`. Purpose: Preprocess red nodes
**CN:** 声明类 `PassPreprocessRed`，其基类为 `EVTPassBase`。 其用途：Preprocess red nodes

#### Lines 50-51 — Function `call`

```python
    def call(self):
        # Step 1: find the compute nodes with op=red
```
**EN:** Defines function `call` with parameters `self`.
**CN:** 定义函数 `call`，参数为 `self`。

##### Line 52 — Assign `red_compute_nodes`

```python
        red_compute_nodes = []
```
**EN:** Assigns `red_compute_nodes` from `[]`, establishing state in function `call`.
**CN:** 将 `red_compute_nodes` 赋值为 `[]`，用于在函数 `call` 中建立状态。

##### Lines 53-62 — Loop over `self.dag_ir.nodes_meta`

```python
        for node_meta in self.dag_ir.nodes_meta:
            if isinstance(node_meta, ComputeNode):
                if type(node_meta.fn) == tuple:
                    # To keep the frontend simple, the reduction nodes
                    # are parsed into compute nodes by default
                    # The simple heuristic to distinguish between compute
                    # and reduction node is that compute node is a single function,
                    # while the reduction node is a tuple of functions for
                    # in-register reduction and atomic global memory reduction
                    red_compute_nodes.append(node_meta.name)
```
**EN:** Iterates `node_meta` over `self.dag_ir.nodes_meta` to repeat a processing step.
**CN:** 让 `node_meta` 遍历 `self.dag_ir.nodes_meta`，从而重复执行处理步骤。

##### Line 64 — Comment or spacing block

```python
        # Step 2: for each compute, merge it with the succeeding store
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 65-97 — Loop over `red_compute_nodes`

```python
        for node in red_compute_nodes:
            # Verify
            users = self.dag_ir.get_users(node)
            inputs = self.dag_ir.get_all_inputs(node)
            # Has a single user
            assert len(users) == 1
            assert len(inputs) == 1
            user = users[0]
            input = inputs[0]

            user_meta = self.dag_ir.get_node_meta(user)
            # Must be a store node
            assert isinstance(user_meta, StoreNode)
            # With output degree == 0
            assert self.dag_ir.out_degree(user) == 0
            # Register the reduce op
            node_meta = self.dag_ir.get_node_meta(node)
            user_meta.reg_reduce_fn, user_meta.gmem_reduce_fn = node_meta.fn
            user_meta.element_compute = node_meta.element_compute
            user_meta.round_style = node_meta.round_style

            # Replace all uses
            self.dag_ir.remove_edge(input, node)
            input_users = self.dag_ir.get_users(input)
            for iu in input_users:
                weight = self.dag_ir.get_edge_weight(input, iu)
                self.dag_ir.add_edge(user, iu, weight)
                self.dag_ir.remove_edge(input, iu)
            self.dag_ir.add_edge(input, user)
            self.dag_ir.remove_node(node)

            # Register the reduction name
            self.dag_ir.reduction_names.append(user)
```
**EN:** Iterates `node` over `red_compute_nodes` to repeat a processing step.
**CN:** 让 `node` 遍历 `red_compute_nodes`，从而重复执行处理步骤。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `PassPreprocessRed`.
- **CN:** 顶层类：`PassPreprocessRed`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.evt.ir`, `cutlass_cppgen.backend.evt.passes.pass_manager`
- **Standard & third-party / 标准库与第三方:** none / 无
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
