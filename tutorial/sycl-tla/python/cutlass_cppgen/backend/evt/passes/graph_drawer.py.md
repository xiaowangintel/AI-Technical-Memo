# graph_drawer.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/passes/graph_drawer.py`
- **EN:** Defines classes `EVTGraphDrawer` for the cutlass_cppgen package.
- **CN:** 为 cutlass_cppgen 包定义类 `EVTGraphDrawer`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31 — File header

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

### Line 32 — From `__future__` import

```python
from __future__ import annotations
```
**EN:** Imports `annotations` from `__future__` to reuse shared definitions at module scope.
**CN:** 从 `__future__` 导入 `annotations`，以便后续代码在模块级复用共享定义。

### Line 34 — Import `subprocess`

```python
import subprocess
```
**EN:** Imports `subprocess` so later code can use these APIs at module scope.
**CN:** 导入 `subprocess`，供后续代码在模块级使用这些 API。

### Line 36 — From `cutlass_library` import

```python
from cutlass_library import DataTypeTag
```
**EN:** Imports `DataTypeTag` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `DataTypeTag`，以便后续代码在模块级复用共享定义。

### Line 38 — From `cutlass_cppgen.backend.evt.ir.dag_ir` import

```python
from cutlass_cppgen.backend.evt.ir.dag_ir import DAGIR
```
**EN:** Imports `DAGIR` from `cutlass_cppgen.backend.evt.ir.dag_ir` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir.dag_ir` 导入 `DAGIR`，以便后续代码在模块级复用共享定义。

### Lines 41-48 — Assign `_COLOR_MAP`

```python
_COLOR_MAP = {
    "load": '"AliceBlue"',
    "compute": "LemonChiffon1",
    "accumulator": "LightGrey",
    "store": "PowderBlue",
    "layout": "lightseagreen",
    "dag": "darkorange"
}
```
**EN:** Assigns `_COLOR_MAP` from `{'load': '"AliceBlue"', 'compute': 'LemonChiffon1', 'accumulator': 'LightGrey', 'store': 'PowderBlue', 'layout': 'lightseagreen', 'dag': 'darkorange'}`, establishing state at module scope.
**CN:** 将 `_COLOR_MAP` 赋值为 `{'load': '"AliceBlue"', 'compute': 'LemonChiffon1', 'accumulator': 'LightGrey', 'store': 'PowderBlue', 'layout': 'lightseagreen', 'dag': 'darkorange'}`，用于在模块级建立状态。

### Lines 51-54 — Class `EVTGraphDrawer`

```python
class EVTGraphDrawer:
    """
    Visualize a EVT DAGIR with graphviz
    """
```
**EN:** Declares class `EVTGraphDrawer` deriving from `object`. Purpose: Visualize a EVT DAGIR with graphviz
**CN:** 声明类 `EVTGraphDrawer`，其基类为 `object`。 其用途：Visualize a EVT DAGIR with graphviz

#### Lines 55-59 — Function `__init__`

```python
    def __init__(
        self,
        graph: DAGIR,
        name: str
    ):
```
**EN:** Defines function `__init__` with parameters `self, graph, name`.
**CN:** 定义函数 `__init__`，参数为 `self, graph, name`。

##### Line 60 — Assign `self._name`

```python
        self._name = name
```
**EN:** Assigns `self._name` from `name`, establishing state in function `__init__`.
**CN:** 将 `self._name` 赋值为 `name`，用于在函数 `__init__` 中建立状态。

##### Line 61 — Assign `self._dot_graphs`

```python
        self._dot_graphs = {}
```
**EN:** Assigns `self._dot_graphs` from `{}`, establishing state in function `__init__`.
**CN:** 将 `self._dot_graphs` 赋值为 `{}`，用于在函数 `__init__` 中建立状态。

##### Line 63 — Assign `self._dot_graphs[name]`

```python
        self._dot_graphs[name] = self._to_dot(graph, name)
```
**EN:** Assigns `self._dot_graphs[name]` from `self._to_dot(graph, name)`, establishing state in function `__init__`.
**CN:** 将 `self._dot_graphs[name]` 赋值为 `self._to_dot(graph, name)`，用于在函数 `__init__` 中建立状态。

#### Line 65 — Function `_get_node_style`

```python
    def _get_node_style(self, node):
```
**EN:** Defines function `_get_node_style` with parameters `self, node`.
**CN:** 定义函数 `_get_node_style`，参数为 `self, node`。

##### Lines 66-71 — Assign `template`

```python
        template = {
            "shape": "record",
            "fillcolor": "#CAFFE3",
            "style": '"filled,rounded"',
            "fontcolor": "#000000",
        }
```
**EN:** Assigns `template` from `{'shape': 'record', 'fillcolor': '#CAFFE3', 'style': '"filled,rounded"', 'fontcolor': '#000000'}`, establishing state in function `_get_node_style`.
**CN:** 将 `template` 赋值为 `{'shape': 'record', 'fillcolor': '#CAFFE3', 'style': '"filled,rounded"', 'fontcolor': '#000000'}`，用于在函数 `_get_node_style` 中建立状态。

##### Lines 72-75 — Conditional `node.op in _COLOR_MAP`

```python
        if node.op in _COLOR_MAP:
            template["fillcolor"] = _COLOR_MAP[node.op]
        else:
            raise NotImplementedError("unknown node op")
```
**EN:** Checks `node.op in _COLOR_MAP` and selects the matching branch in function `_get_node_style`.
**CN:** 检查 `node.op in _COLOR_MAP`，并在函数 `_get_node_style` 中选择匹配的分支。

##### Lines 76-78 — Conditional `node.disabled`

```python
        if node.disabled:
            template["fontcolor"] = "grey"
            template["fillcolor"] = "white"
```
**EN:** Checks `node.disabled` and selects the matching branch in function `_get_node_style`.
**CN:** 检查 `node.disabled`，并在函数 `_get_node_style` 中选择匹配的分支。

##### Line 79 — Return

```python
        return template
```
**EN:** Returns `template` to the caller.
**CN:** 向调用方返回 `template`。

#### Line 81 — Function `_get_node_label`

```python
    def _get_node_label(self, node):
```
**EN:** Defines function `_get_node_label` with parameters `self, node`.
**CN:** 定义函数 `_get_node_label`，参数为 `self, node`。

##### Line 82 — Assign `label`

```python
        label = "{" + f"name={node.name}|op={node.op}"
```
**EN:** Assigns `label` from `'{' + f'name={node.name}|op={node.op}'`, establishing state in function `_get_node_label`.
**CN:** 将 `label` 赋值为 `'{' + f'name={node.name}|op={node.op}'`，用于在函数 `_get_node_label` 中建立状态。

##### Lines 83-86 — Conditional `node.op == 'layout'`

```python
        if node.op == "layout":
            label += f"|fn={node.fn.__name__}"
            for key in node.kwargs:
                label += f"|{key}={node.kwargs[key]}"
```
**EN:** Checks `node.op == 'layout'` and selects the matching branch in function `_get_node_label`.
**CN:** 检查 `node.op == 'layout'`，并在函数 `_get_node_label` 中选择匹配的分支。

##### Lines 87-96 — Conditional `node.underlying_impl is not None`

```python
        if node.underlying_impl is not None:
            label += f"|impl={type(node.underlying_impl).__name__}"
            if node.op == "load":
                label += f"|element_output={DataTypeTag[node.underlying_impl.element]}"
            elif node.op == "compute":
                label += f"|element_compute={DataTypeTag[node.underlying_impl.element_compute]}|element_output={DataTypeTag[node.underlying_impl.element_output]}"
            elif node.op == "store":
                label += f"|element_store={DataTypeTag[node.underlying_impl.element]}|element_output={DataTypeTag[node.underlying_impl.element_output]}"
            elif node.op == "dag":
                label += f"|element_output={DataTypeTag[node.underlying_impl.element_output]}"
```
**EN:** Checks `node.underlying_impl is not None` and selects the matching branch in function `_get_node_label`.
**CN:** 检查 `node.underlying_impl is not None`，并在函数 `_get_node_label` 中选择匹配的分支。

##### Lines 97-100 — Conditional `node.tensor is not None`

```python
        if node.tensor is not None:
            shape = node.tensor.shape
            stride = node.tensor.stride
            label += f"|shape={shape}|stride={stride}"
```
**EN:** Checks `node.tensor is not None` and selects the matching branch in function `_get_node_label`.
**CN:** 检查 `node.tensor is not None`，并在函数 `_get_node_label` 中选择匹配的分支。

##### Lines 102-106 — Conditional `hasattr(node, 'store_tensor')`

```python
        if hasattr(node, "store_tensor"):
            if node.store_tensor is not None:
                store_shape = node.store_tensor.shape
                store_stride = node.store_tensor.stride
                label += f"|store_shape={store_shape}|stride_stride={store_stride}"
```
**EN:** Checks `hasattr(node, 'store_tensor')` and selects the matching branch in function `_get_node_label`.
**CN:** 检查 `hasattr(node, 'store_tensor')`，并在函数 `_get_node_label` 中选择匹配的分支。

##### Line 108 — Update `label`

```python
        label += "}"
```
**EN:** Updates `label` with `+=` using `'}'`.
**CN:** 使用 `'}'` 对 `label` 执行 `+=` 更新。

##### Line 109 — Return

```python
        return label
```
**EN:** Returns `label` to the caller.
**CN:** 向调用方返回 `label`。

#### Lines 111-115 — Function `_to_dot`

```python
    def _to_dot(
        self,
        graph: DAGIR,
        name: str
    ):
```
**EN:** Defines function `_to_dot` with parameters `self, graph, name`.
**CN:** 定义函数 `_to_dot`，参数为 `self, graph, name`。

##### Line 116 — Import `pydot`

```python
        import pydot
```
**EN:** Imports `pydot` so later code can use these APIs in function `_to_dot`.
**CN:** 导入 `pydot`，供后续代码在函数 `_to_dot` 中使用这些 API。

##### Line 117 — Assign `dot_graph`

```python
        dot_graph = pydot.Dot(name, randir="TB")
```
**EN:** Assigns `dot_graph` from `pydot.Dot(name, randir='TB')`, establishing state in function `_to_dot`.
**CN:** 将 `dot_graph` 赋值为 `pydot.Dot(name, randir='TB')`，用于在函数 `_to_dot` 中建立状态。

##### Lines 118-127 — Loop over `graph.nodes_meta`

```python
        for node in graph.nodes_meta:
            style = self._get_node_style(node)
            label = self._get_node_label(node)
            dot_node = pydot.Node(
                node.name, label=label, **style
            )
            dot_graph.add_node(dot_node)
            if node.op == "dag":
                dot_subgraph = self._to_dot(node.subgraph, name=node.name)
                self._dot_graphs[node.name] = dot_subgraph
```
**EN:** Iterates `node` over `graph.nodes_meta` to repeat a processing step.
**CN:** 让 `node` 遍历 `graph.nodes_meta`，从而重复执行处理步骤。

##### Line 129 — Comment or spacing block

```python
        # Add edges
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 130-132 — Loop over `graph.edges`

```python
        for src, dst in graph.edges:
            weight = graph.get_edge_weight(src, dst)
            dot_graph.add_edge(pydot.Edge(src, dst, label=weight))
```
**EN:** Iterates `(src, dst)` over `graph.edges` to repeat a processing step.
**CN:** 让 `(src, dst)` 遍历 `graph.edges`，从而重复执行处理步骤。

##### Line 134 — Return

```python
        return dot_graph
```
**EN:** Returns `dot_graph` to the caller.
**CN:** 向调用方返回 `dot_graph`。

#### Line 136 — Function `get_dot_graph`

```python
    def get_dot_graph(self) -> pydot.Dot:
```
**EN:** Defines function `get_dot_graph` with parameters `self`.
**CN:** 定义函数 `get_dot_graph`，参数为 `self`。

##### Line 137 — Return

```python
        return [(key, self.get_dot_graph_by_name(key)) for key in self._dot_graphs.keys()]
```
**EN:** Returns `[(key, self.get_dot_graph_by_name(key)) for key in self._dot_graphs.keys()]` to the caller.
**CN:** 向调用方返回 `[(key, self.get_dot_graph_by_name(key)) for key in self._dot_graphs.keys()]`。

#### Line 139 — Function `get_dot_graph_by_name`

```python
    def get_dot_graph_by_name(self, name) -> pydot.Dot:
```
**EN:** Defines function `get_dot_graph_by_name` with parameters `self, name`.
**CN:** 定义函数 `get_dot_graph_by_name`，参数为 `self, name`。

##### Line 140 — Return

```python
        return self._dot_graphs[name]
```
**EN:** Returns `self._dot_graphs[name]` to the caller.
**CN:** 向调用方返回 `self._dot_graphs[name]`。

#### Line 142 — Function `get_main_dot_graph`

```python
    def get_main_dot_graph(self) -> pydot.Dot:
```
**EN:** Defines function `get_main_dot_graph` with parameters `self`.
**CN:** 定义函数 `get_main_dot_graph`，参数为 `self`。

##### Line 143 — Return

```python
        return self._dot_graphs[self._name]
```
**EN:** Returns `self._dot_graphs[self._name]` to the caller.
**CN:** 向调用方返回 `self._dot_graphs[self._name]`。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `EVTGraphDrawer`.
- **CN:** 顶层类：`EVTGraphDrawer`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.evt.ir.dag_ir`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** `__future__`, `pydot`, `subprocess`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
