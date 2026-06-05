# emitter_base.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/backend/emitter_base.py`
- **EN:** Base class for Epilogue Visitor Emitter
- **CN:** 模块文档说明：Base class for Epilogue Visitor Emitter

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
Base class for Epilogue Visitor Emitter
"""
```
**EN:** Docstring explains this scope: Base class for Epilogue Visitor Emitter
**CN:** 文档字符串说明了该作用域的用途：Base class for Epilogue Visitor Emitter

### Line 37 — From `cutlass_library` import

```python
from cutlass_library import DataTypeTag
```
**EN:** Imports `DataTypeTag` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `DataTypeTag`，以便后续代码在模块级复用共享定义。

### Line 38 — From `cutlass_cppgen.backend.evt.ir` import

```python
from cutlass_cppgen.backend.evt.ir import TopoVisitorNode, DAGIR
```
**EN:** Imports `TopoVisitorNode, DAGIR` from `cutlass_cppgen.backend.evt.ir` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir` 导入 `TopoVisitorNode, DAGIR`，以便后续代码在模块级复用共享定义。

### Line 39 — From `cutlass_library.arch_constants` import

```python
from cutlass_library.arch_constants import ( INTEL_XE12, INTEL_XE20)
```
**EN:** Imports `INTEL_XE12, INTEL_XE20` from `cutlass_library.arch_constants` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library.arch_constants` 导入 `INTEL_XE12, INTEL_XE20`，以便后续代码在模块级复用共享定义。

### Line 42 — Class `FusionCallbacks`

```python
class FusionCallbacks:
```
**EN:** Declares class `FusionCallbacks` deriving from `object`.
**CN:** 声明类 `FusionCallbacks`，其基类为 `object`。

#### Lines 43-52 — Function `__init__`

```python
    def __init__(self, dag_ir: DAGIR, cc: int, emit_CD=True) -> None:
        """
        Emit the EVT fusion callbacks
        :param dag_ir: the DAG IR holding the epilogue visitor
        :param cc: compute capability
        :param emit_CD: whether to emit nodes C & D as a part of the fusion callbacks
                        For Sm90, set emit_CD=False, as Tensor C & D are hardcoded in the collective API
                        so that their shared memory can be explicitly reused
                        For Sm89, set emit_CD=True as they are treated as normal AuxLoad & AuxStore nodes.
        """
```
**EN:** Defines function `__init__` with parameters `self, dag_ir, cc, emit_CD`. Purpose: Emit the EVT fusion callbacks
**CN:** 定义函数 `__init__`，参数为 `self, dag_ir, cc, emit_CD`。 其用途：Emit the EVT fusion callbacks

##### Line 53 — Assign `self.dag_ir`

```python
        self.dag_ir = dag_ir
```
**EN:** Assigns `self.dag_ir` from `dag_ir`, establishing state in function `__init__`.
**CN:** 将 `self.dag_ir` 赋值为 `dag_ir`，用于在函数 `__init__` 中建立状态。

##### Line 54 — Assign `self.emit_CD`

```python
        self.emit_CD = emit_CD
```
**EN:** Assigns `self.emit_CD` from `emit_CD`, establishing state in function `__init__`.
**CN:** 将 `self.emit_CD` 赋值为 `emit_CD`，用于在函数 `__init__` 中建立状态。

##### Line 55 — Assign `self.cc`

```python
        self.cc = cc
```
**EN:** Assigns `self.cc` from `cc`, establishing state in function `__init__`.
**CN:** 将 `self.cc` 赋值为 `cc`，用于在函数 `__init__` 中建立状态。

##### Line 56 — Assign `self.evt_cc`

```python
        self.evt_cc = 90 if cc >= 90 else cc
```
**EN:** Assigns `self.evt_cc` from `90 if cc >= 90 else cc`, establishing state in function `__init__`.
**CN:** 将 `self.evt_cc` 赋值为 `90 if cc >= 90 else cc`，用于在函数 `__init__` 中建立状态。

##### Lines 57-60 — Conditional `self.cc not in [INTEL_XE12, INTEL_XE20] and self.cc < 90`

```python
        if self.cc not in [INTEL_XE12, INTEL_XE20] and self.cc < 90:
            self.namespace = "threadblock"
        else:
            self.namespace = "fusion"
```
**EN:** Checks `self.cc not in [INTEL_XE12, INTEL_XE20] and self.cc < 90` and selects the matching branch in function `__init__`.
**CN:** 检查 `self.cc not in [INTEL_XE12, INTEL_XE20] and self.cc < 90`，并在函数 `__init__` 中选择匹配的分支。

#### Lines 62-64 — Comment or spacing block

```python
    #
    # Helper functions
    #
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 66-69 — Function `get_visitor_name`

```python
    def get_visitor_name(self, node: str):
        """
        Get the visitor name
        """
```
**EN:** Defines function `get_visitor_name` with parameters `self, node`. Purpose: Get the visitor name
**CN:** 定义函数 `get_visitor_name`，参数为 `self, node`。 其用途：Get the visitor name

##### Line 70 — Assign `meta`

```python
        meta = self.dag_ir.get_node_meta(node)
```
**EN:** Assigns `meta` from `self.dag_ir.get_node_meta(node)`, establishing state in function `get_visitor_name`.
**CN:** 将 `meta` 赋值为 `self.dag_ir.get_node_meta(node)`，用于在函数 `get_visitor_name` 中建立状态。

##### Lines 71-74 — Conditional `not isinstance(meta, TopoVisitorNode) and self.dag_ir.in_degree(node) > 0`

```python
        if not isinstance(meta, TopoVisitorNode) and self.dag_ir.in_degree(node) > 0:
            return f"EVT{meta.name_camel}"
        else:
            return meta.name_camel
```
**EN:** Checks `not isinstance(meta, TopoVisitorNode) and self.dag_ir.in_degree(node) > 0` and selects the matching branch in function `get_visitor_name`.
**CN:** 检查 `not isinstance(meta, TopoVisitorNode) and self.dag_ir.in_degree(node) > 0`，并在函数 `get_visitor_name` 中选择匹配的分支。

#### Line 76 — Function `emit`

```python
    def emit(self):
```
**EN:** Defines function `emit` with parameters `self`.
**CN:** 定义函数 `emit`，参数为 `self`。

##### Line 77 — Assign `node_metas`

```python
        node_metas = self.dag_ir.node_metas_topological_order()
```
**EN:** Assigns `node_metas` from `self.dag_ir.node_metas_topological_order()`, establishing state in function `emit`.
**CN:** 将 `node_metas` 赋值为 `self.dag_ir.node_metas_topological_order()`，用于在函数 `emit` 中建立状态。

##### Line 78 — Assign `epilogue_str`

```python
        epilogue_str = ""
```
**EN:** Assigns `epilogue_str` from `''`, establishing state in function `emit`.
**CN:** 将 `epilogue_str` 赋值为 `''`，用于在函数 `emit` 中建立状态。

##### Lines 79-80 — Comment or spacing block

```python
        # Step 1: emit individual node type decl
        #         emit the EVT & DAG connector
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 81-89 — Loop over `node_metas`

```python
        for meta in node_metas:
            if not meta.disabled:
                epilogue_str += self.emit_node(meta)
            if not self.emit_CD and meta.name == "D":
                continue
            if isinstance(meta, TopoVisitorNode):
                epilogue_str += self.emit_dag(meta)
            else:
                epilogue_str += self.emit_evt(meta)
```
**EN:** Iterates `meta` over `node_metas` to repeat a processing step.
**CN:** 让 `meta` 遍历 `node_metas`，从而重复执行处理步骤。

##### Line 91 — Comment or spacing block

```python
        # Step 2: post-processing & get callback name
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 92-100 — Conditional `not self.emit_CD`

```python
        if not self.emit_CD:
            if not self.dag_ir.has_node("C"):
                epilogue_str += "using ElementC = void;\nusing StrideC = StrideD;\n"
            output_node = self.dag_ir.get_all_inputs("D")[0]
            # The callback is the src of node D
            callback_name = self.get_visitor_name(output_node)
        else:
            # The callback is the last node in the topological order
            callback_name = self.get_visitor_name(node_metas[-1].name)
```
**EN:** Checks `not self.emit_CD` and selects the matching branch in function `emit`.
**CN:** 检查 `not self.emit_CD`，并在函数 `emit` 中选择匹配的分支。

##### Line 101 — Return

```python
        return epilogue_str, callback_name
```
**EN:** Returns `(epilogue_str, callback_name)` to the caller.
**CN:** 向调用方返回 `(epilogue_str, callback_name)`。

#### Line 103 — Function `emit_evt`

```python
    def emit_evt(self, node):
```
**EN:** Defines function `emit_evt` with parameters `self, node`.
**CN:** 定义函数 `emit_evt`，参数为 `self, node`。

##### Lines 104-105 — Conditional `self.dag_ir.in_degree(node.name) == 0`

```python
        if self.dag_ir.in_degree(node.name) == 0:
            return ""
```
**EN:** Checks `self.dag_ir.in_degree(node.name) == 0` and selects the matching branch in function `emit_evt`.
**CN:** 检查 `self.dag_ir.in_degree(node.name) == 0`，并在函数 `emit_evt` 中选择匹配的分支。

##### Lines 107-118 — Conditional `self.cc in [INTEL_XE12, INTEL_XE20]`

```python
        if self.cc in [INTEL_XE12, INTEL_XE20]:
            evt_tmp = (
                f"\nusing EVT{node.name_camel} = cutlass::epilogue::{self.namespace}::"
                f"XeEVT<\n"
                f"    {node.name_camel},\n"
            )
        else:
           evt_tmp = (
                f"\nusing EVT{node.name_camel} = cutlass::epilogue::{self.namespace}::"
                f"Sm{self.evt_cc}EVT<\n"
                f"    {node.name_camel},\n"
            )
```
**EN:** Checks `self.cc in [INTEL_XE12, INTEL_XE20]` and selects the matching branch in function `emit_evt`.
**CN:** 检查 `self.cc in [INTEL_XE12, INTEL_XE20]`，并在函数 `emit_evt` 中选择匹配的分支。

##### Line 119 — Assign `sorted_children`

```python
        sorted_children = self.dag_ir.get_all_inputs(node.name)
```
**EN:** Assigns `sorted_children` from `self.dag_ir.get_all_inputs(node.name)`, establishing state in function `emit_evt`.
**CN:** 将 `sorted_children` 赋值为 `self.dag_ir.get_all_inputs(node.name)`，用于在函数 `emit_evt` 中建立状态。

##### Line 120 — Assign `evt_node_strs`

```python
        evt_node_strs = [f"    {self.get_visitor_name(child_name)}" for child_name in sorted_children]
```
**EN:** Assigns `evt_node_strs` from `[f' {self.get_visitor_name(child_name)}' for child_name in sorted_children]`, establishing state in function `emit_evt`.
**CN:** 将 `evt_node_strs` 赋值为 `[f' {self.get_visitor_name(child_name)}' for child_name in sorted_children]`，用于在函数 `emit_evt` 中建立状态。

##### Line 121 — Update `evt_tmp`

```python
        evt_tmp += ",\n".join(evt_node_strs) + ">;\n"
```
**EN:** Updates `evt_tmp` with `+=` using `',\n'.join(evt_node_strs) + '>;\n'`.
**CN:** 使用 `',\n'.join(evt_node_strs) + '>;\n'` 对 `evt_tmp` 执行 `+=` 更新。

##### Line 123 — Return

```python
        return evt_tmp
```
**EN:** Returns `evt_tmp` to the caller.
**CN:** 向调用方返回 `evt_tmp`。

#### Line 125 — Function `emit_dag`

```python
    def emit_dag(self, node):
```
**EN:** Defines function `emit_dag` with parameters `self, node`.
**CN:** 定义函数 `emit_dag`，参数为 `self, node`。

##### Line 126 — Assign `subgraph`

```python
        subgraph = node.subgraph
```
**EN:** Assigns `subgraph` from `node.subgraph`, establishing state in function `emit_dag`.
**CN:** 将 `subgraph` 赋值为 `node.subgraph`，用于在函数 `emit_dag` 中建立状态。

##### Line 127 — Assign `subgraph_nodes`

```python
        subgraph_nodes = subgraph.nodes_topological_order()
```
**EN:** Assigns `subgraph_nodes` from `subgraph.nodes_topological_order()`, establishing state in function `emit_dag`.
**CN:** 将 `subgraph_nodes` 赋值为 `subgraph.nodes_topological_order()`，用于在函数 `emit_dag` 中建立状态。

##### Line 128 — Comment or spacing block

```python
        # Emit the Edge Tuple
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 129 — Assign `edge_tuples`

```python
        edge_tuples = "cute::tuple<\n"
```
**EN:** Assigns `edge_tuples` from `'cute::tuple<\n'`, establishing state in function `emit_dag`.
**CN:** 将 `edge_tuples` 赋值为 `'cute::tuple<\n'`，用于在函数 `emit_dag` 中建立状态。

##### Line 130 — Assign `edge_tuple_list`

```python
        edge_tuple_list = []
```
**EN:** Assigns `edge_tuple_list` from `[]`, establishing state in function `emit_dag`.
**CN:** 将 `edge_tuple_list` 赋值为 `[]`，用于在函数 `emit_dag` 中建立状态。

##### Lines 131-138 — Loop over `subgraph_nodes[:-1]`

```python
        for n in subgraph_nodes[:-1]:
            in_edges = subgraph.in_edges(n)
            edge_weights = [subgraph.get_edge_weight(edge[0], edge[1]) for edge in in_edges]
            sorted_children = [edge[0] for _, edge in sorted(zip(edge_weights, in_edges))]
            edge_tuple = "        cute::seq<"
            edge_str = [str(subgraph_nodes.index(child)) for child in sorted_children]
            edge_tuple += ", ".join(edge_str) + ">"
            edge_tuple_list.append(edge_tuple)
```
**EN:** Iterates `n` over `subgraph_nodes[:-1]` to repeat a processing step.
**CN:** 让 `n` 遍历 `subgraph_nodes[:-1]`，从而重复执行处理步骤。

##### Line 139 — Update `edge_tuples`

```python
        edge_tuples += ",\n".join(edge_tuple_list) + "\n    >"
```
**EN:** Updates `edge_tuples` with `+=` using `',\n'.join(edge_tuple_list) + '\n >'`.
**CN:** 使用 `',\n'.join(edge_tuple_list) + '\n >'` 对 `edge_tuples` 执行 `+=` 更新。

##### Line 141 — Comment or spacing block

```python
        # Emit the node list
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 142 — Assign `dag_nodes`

```python
        dag_nodes = ""
```
**EN:** Assigns `dag_nodes` from `''`, establishing state in function `emit_dag`.
**CN:** 将 `dag_nodes` 赋值为 `''`，用于在函数 `emit_dag` 中建立状态。

##### Line 143 — Assign `dag_node_strs`

```python
        dag_node_strs = []
```
**EN:** Assigns `dag_node_strs` from `[]`, establishing state in function `emit_dag`.
**CN:** 将 `dag_node_strs` 赋值为 `[]`，用于在函数 `emit_dag` 中建立状态。

##### Lines 144-149 — Loop over `subgraph_nodes[:-1]`

```python
        for n in subgraph_nodes[:-1]:
            n_meta = subgraph.get_node_meta(n)
            if n_meta.disabled:
                dag_node_strs.append(f"    {self.get_visitor_name(n)}")
            else:
                dag_node_strs.append(f"    {n_meta.name_camel}")
```
**EN:** Iterates `n` over `subgraph_nodes[:-1]` to repeat a processing step.
**CN:** 让 `n` 遍历 `subgraph_nodes[:-1]`，从而重复执行处理步骤。

##### Line 150 — Assign `dag_nodes`

```python
        dag_nodes = ",\n".join(dag_node_strs)
```
**EN:** Assigns `dag_nodes` from `',\n'.join(dag_node_strs)`, establishing state in function `emit_dag`.
**CN:** 将 `dag_nodes` 赋值为 `',\n'.join(dag_node_strs)`，用于在函数 `emit_dag` 中建立状态。

##### Lines 152-169 — Conditional `self.cc in [INTEL_XE12, INTEL_XE20]`

```python
        if self.cc in [INTEL_XE12, INTEL_XE20]:
            return (
                f"\nusing {node.name_camel} = cutlass::epilogue::{self.namespace}::"
                f"XeTopologicalVisitor<\n"
                f"    {DataTypeTag[node.subgraph.element_compute]},\n"
                f"    {edge_tuples},\n"
                f"{dag_nodes}\n"
                ">;\n"
            )
        else:
            return (
                f"\nusing {node.name_camel} = cutlass::epilogue::{self.namespace}::"
                f"Sm{self.evt_cc}TopologicalVisitor<\n"
                f"    {DataTypeTag[node.subgraph.element_compute]},\n"
                f"    {edge_tuples},\n"
                f"{dag_nodes}\n"
                ">;\n"
            )
```
**EN:** Checks `self.cc in [INTEL_XE12, INTEL_XE20]` and selects the matching branch in function `emit_dag`.
**CN:** 检查 `self.cc in [INTEL_XE12, INTEL_XE20]`，并在函数 `emit_dag` 中选择匹配的分支。

#### Line 171 — Function `emit_node`

```python
    def emit_node(self, node):
```
**EN:** Defines function `emit_node` with parameters `self, node`.
**CN:** 定义函数 `emit_node`，参数为 `self, node`。

##### Lines 172-179 — Conditional `isinstance(node, TopoVisitorNode)`

```python
        if isinstance(node, TopoVisitorNode):
            emission = ""
            for node in node.subgraph.node_metas_topological_order():
                if not node.disabled:
                    emission += self.emit_node(node)
            return emission
        else:
            return node.underlying_impl.type_decl
```
**EN:** Checks `isinstance(node, TopoVisitorNode)` and selects the matching branch in function `emit_node`.
**CN:** 检查 `isinstance(node, TopoVisitorNode)`，并在函数 `emit_node` 中选择匹配的分支。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `FusionCallbacks`.
- **CN:** 顶层类：`FusionCallbacks`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.evt.ir`, `cutlass_library`, `cutlass_library.arch_constants`
- **Standard & third-party / 标准库与第三方:** none / 无
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
