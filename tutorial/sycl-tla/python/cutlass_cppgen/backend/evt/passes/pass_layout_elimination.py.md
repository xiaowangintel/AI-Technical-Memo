# pass_layout_elimination.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/passes/pass_layout_elimination.py`
- **EN:** Eliminate layout manipulation nodes
- **CN:** 模块文档说明：Eliminate layout manipulation nodes

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
Eliminate layout manipulation nodes
"""
```
**EN:** Docstring explains this scope: Eliminate layout manipulation nodes
**CN:** 文档字符串说明了该作用域的用途：Eliminate layout manipulation nodes

### Line 37 — From `copy` import

```python
from copy import deepcopy
```
**EN:** Imports `deepcopy` from `copy` to reuse shared definitions at module scope.
**CN:** 从 `copy` 导入 `deepcopy`，以便后续代码在模块级复用共享定义。

### Line 39 — From `cutlass_cppgen.backend.evt.ir` import

```python
from cutlass_cppgen.backend.evt.ir import DAGIR, LayoutNode
```
**EN:** Imports `DAGIR, LayoutNode` from `cutlass_cppgen.backend.evt.ir` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir` 导入 `DAGIR, LayoutNode`，以便后续代码在模块级复用共享定义。

### Line 40 — From `cutlass_cppgen.backend.evt.passes.pass_manager` import

```python
from cutlass_cppgen.backend.evt.passes.pass_manager import EVTPassBase
```
**EN:** Imports `EVTPassBase` from `cutlass_cppgen.backend.evt.passes.pass_manager` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.passes.pass_manager` 导入 `EVTPassBase`，以便后续代码在模块级复用共享定义。

### Line 41 — From `cutlass_cppgen.backend.evt.passes.pass_shape_type_propagation` import

```python
from cutlass_cppgen.backend.evt.passes.pass_shape_type_propagation import PassShapeTypePropagation
```
**EN:** Imports `PassShapeTypePropagation` from `cutlass_cppgen.backend.evt.passes.pass_shape_type_propagation` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.passes.pass_shape_type_propagation` 导入 `PassShapeTypePropagation`，以便后续代码在模块级复用共享定义。

### Lines 44-47 — Class `PassLayoutManipulateElimination`

```python
class PassLayoutManipulateElimination(EVTPassBase):
    """
    Eliminate layout manipulation nodes
    """
```
**EN:** Declares class `PassLayoutManipulateElimination` deriving from `EVTPassBase`. Purpose: Eliminate layout manipulation nodes
**CN:** 声明类 `PassLayoutManipulateElimination`，其基类为 `EVTPassBase`。 其用途：Eliminate layout manipulation nodes

#### Line 48 — Assign `dependencies`

```python
    dependencies = [PassShapeTypePropagation]
```
**EN:** Assigns `dependencies` from `[PassShapeTypePropagation]`, establishing state in class `PassLayoutManipulateElimination`.
**CN:** 将 `dependencies` 赋值为 `[PassShapeTypePropagation]`，用于在类 `PassLayoutManipulateElimination` 中建立状态。

#### Line 50 — Function `__init__`

```python
    def __init__(self, dag_ir: DAGIR) -> None:
```
**EN:** Defines function `__init__` with parameters `self, dag_ir`.
**CN:** 定义函数 `__init__`，参数为 `self, dag_ir`。

##### Line 51 — Call `super().__init__`

```python
        super().__init__(dag_ir)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 52 — Assign `self.copy_cnt`

```python
        self.copy_cnt = 0
```
**EN:** Assigns `self.copy_cnt` from `0`, establishing state in function `__init__`.
**CN:** 将 `self.copy_cnt` 赋值为 `0`，用于在函数 `__init__` 中建立状态。

#### Line 54 — Function `call`

```python
    def call(self):
```
**EN:** Defines function `call` with parameters `self`.
**CN:** 定义函数 `call`，参数为 `self`。

##### Line 55 — Assign `self.layout_nodes_worklist`

```python
        self.layout_nodes_worklist = self.get_all_layout_nodes()
```
**EN:** Assigns `self.layout_nodes_worklist` from `self.get_all_layout_nodes()`, establishing state in function `call`.
**CN:** 将 `self.layout_nodes_worklist` 赋值为 `self.get_all_layout_nodes()`，用于在函数 `call` 中建立状态。

##### Line 56 — Comment or spacing block

```python
        # Run while loop utill all layout nodes are eliminated
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 57-66 — While `len(self.layout_nodes_worklist) > 0`

```python
        while(len(self.layout_nodes_worklist) > 0):
            node = self.layout_nodes_worklist.pop(0)
            # for node in layout_nodes:
            # Step 1: get the propagation direction
            direction = self.get_propagation_direction(node)
            self.visited = []
            getattr(self, f"propagate_to_{direction}")(self.dag_ir.get_node_meta(node), node)
            # Eliminate the current node
            input_node = self.dag_ir.get_all_inputs(node)[0]
            self.dag_ir.replace_all_uses_with(node, input_node)
```
**EN:** Repeats the block while `len(self.layout_nodes_worklist) > 0` stays true.
**CN:** 当 `len(self.layout_nodes_worklist) > 0` 为真时重复执行该代码块。

###### Line 67 — Comment or spacing block

```python
            # layout_nodes = self.get_all_layout_nodes()
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 69 — Function `get_all_layout_nodes`

```python
    def get_all_layout_nodes(self):
```
**EN:** Defines function `get_all_layout_nodes` with parameters `self`.
**CN:** 定义函数 `get_all_layout_nodes`，参数为 `self`。

##### Line 70 — Assign `layout_nodes`

```python
        layout_nodes = []
```
**EN:** Assigns `layout_nodes` from `[]`, establishing state in function `get_all_layout_nodes`.
**CN:** 将 `layout_nodes` 赋值为 `[]`，用于在函数 `get_all_layout_nodes` 中建立状态。

##### Lines 71-73 — Loop over `reversed(self.dag_ir.node_metas_topological_order())`

```python
        for node_meta in reversed(self.dag_ir.node_metas_topological_order()):
            if isinstance(node_meta, LayoutNode):
                layout_nodes.append(node_meta.name)
```
**EN:** Iterates `node_meta` over `reversed(self.dag_ir.node_metas_topological_order())` to repeat a processing step.
**CN:** 让 `node_meta` 遍历 `reversed(self.dag_ir.node_metas_topological_order())`，从而重复执行处理步骤。

##### Line 74 — Return

```python
        return layout_nodes
```
**EN:** Returns `layout_nodes` to the caller.
**CN:** 向调用方返回 `layout_nodes`。

#### Lines 76-79 — Function `get_propagation_direction`

```python
    def get_propagation_direction(self, node: str):
        """
        The logic is propagating all layout nodes away from the accumulator node.
        """
```
**EN:** Defines function `get_propagation_direction` with parameters `self, node`. Purpose: The logic is propagating all layout nodes away from the accumulator node.
**CN:** 定义函数 `get_propagation_direction`，参数为 `self, node`。 其用途：The logic is propagating all layout nodes away from the accumulator node.

##### Line 80 — Assign `self.visited`

```python
        self.visited = []
```
**EN:** Assigns `self.visited` from `[]`, establishing state in function `get_propagation_direction`.
**CN:** 将 `self.visited` 赋值为 `[]`，用于在函数 `get_propagation_direction` 中建立状态。

##### Line 81 — Call `self.get_influenced_users`

```python
        self.get_influenced_users(node)
```
**EN:** Calls `self.get_influenced_users` for side effects or initialization work in function `get_propagation_direction`.
**CN:** 调用 `self.get_influenced_users` 执行副作用或初始化逻辑；该语句位于在函数 `get_propagation_direction` 中。

##### Line 82 — Assign `nodes_influenced_dir_users`

```python
        nodes_influenced_dir_users = self.visited
```
**EN:** Assigns `nodes_influenced_dir_users` from `self.visited`, establishing state in function `get_propagation_direction`.
**CN:** 将 `nodes_influenced_dir_users` 赋值为 `self.visited`，用于在函数 `get_propagation_direction` 中建立状态。

##### Line 83 — Assign `self.visited`

```python
        self.visited = []
```
**EN:** Assigns `self.visited` from `[]`, establishing state in function `get_propagation_direction`.
**CN:** 将 `self.visited` 赋值为 `[]`，用于在函数 `get_propagation_direction` 中建立状态。

##### Line 84 — Call `self.get_influenced_inputs`

```python
        self.get_influenced_inputs(node)
```
**EN:** Calls `self.get_influenced_inputs` for side effects or initialization work in function `get_propagation_direction`.
**CN:** 调用 `self.get_influenced_inputs` 执行副作用或初始化逻辑；该语句位于在函数 `get_propagation_direction` 中。

##### Line 85 — Assign `nodes_influenced_dir_inputs`

```python
        nodes_influenced_dir_inputs = self.visited
```
**EN:** Assigns `nodes_influenced_dir_inputs` from `self.visited`, establishing state in function `get_propagation_direction`.
**CN:** 将 `nodes_influenced_dir_inputs` 赋值为 `self.visited`，用于在函数 `get_propagation_direction` 中建立状态。

##### Lines 87-92 — Conditional `'accum' in nodes_influenced_dir_users and 'accum' not in nodes_influenced_dir_inputs`

```python
        if "accum" in nodes_influenced_dir_users and "accum" not in nodes_influenced_dir_inputs:
            return "inputs"
        elif "accum" not in nodes_influenced_dir_users and "accum" in nodes_influenced_dir_inputs:
            return "users"
        else:
            raise RuntimeError("Unsolved propagation direction")
```
**EN:** Checks `'accum' in nodes_influenced_dir_users and 'accum' not in nodes_influenced_dir_inputs` and selects the matching branch in function `get_propagation_direction`.
**CN:** 检查 `'accum' in nodes_influenced_dir_users and 'accum' not in nodes_influenced_dir_inputs`，并在函数 `get_propagation_direction` 中选择匹配的分支。

#### Line 94 — Comment or spacing block

```python
    # Get all influenced nodes if we propagate along the user direction
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 95 — Function `get_influenced_users`

```python
    def get_influenced_users(self, node: str):
```
**EN:** Defines function `get_influenced_users` with parameters `self, node`.
**CN:** 定义函数 `get_influenced_users`，参数为 `self, node`。

##### Lines 96-97 — Conditional `node in self.visited`

```python
        if node in self.visited:
            return
```
**EN:** Checks `node in self.visited` and selects the matching branch in function `get_influenced_users`.
**CN:** 检查 `node in self.visited`，并在函数 `get_influenced_users` 中选择匹配的分支。

##### Line 98 — Call `self.visited.append`

```python
        self.visited.append(node)
```
**EN:** Calls `self.visited.append` for side effects or initialization work in function `get_influenced_users`.
**CN:** 调用 `self.visited.append` 执行副作用或初始化逻辑；该语句位于在函数 `get_influenced_users` 中。

##### Line 100 — Assign `users`

```python
        users = self.dag_ir.get_users(node)
```
**EN:** Assigns `users` from `self.dag_ir.get_users(node)`, establishing state in function `get_influenced_users`.
**CN:** 将 `users` 赋值为 `self.dag_ir.get_users(node)`，用于在函数 `get_influenced_users` 中建立状态。

##### Lines 101-102 — Loop over `users`

```python
        for user in users:
            self.get_influenced_users(user)
```
**EN:** Iterates `user` over `users` to repeat a processing step.
**CN:** 让 `user` 遍历 `users`，从而重复执行处理步骤。

##### Line 103 — Assign `user_inputs`

```python
        user_inputs = []
```
**EN:** Assigns `user_inputs` from `[]`, establishing state in function `get_influenced_users`.
**CN:** 将 `user_inputs` 赋值为 `[]`，用于在函数 `get_influenced_users` 中建立状态。

##### Lines 104-105 — Loop over `users`

```python
        for user in users:
            user_inputs.append(set(self.dag_ir.get_all_inputs(user)))
```
**EN:** Iterates `user` over `users` to repeat a processing step.
**CN:** 让 `user` 遍历 `users`，从而重复执行处理步骤。

##### Lines 106-110 — Conditional `len(user_inputs) > 0`

```python
        if len(user_inputs) > 0:
            user_inputs = set.union(*user_inputs)
            user_inputs.remove(node)
            for input in user_inputs:
                self.get_influenced_inputs(input)
```
**EN:** Checks `len(user_inputs) > 0` and selects the matching branch in function `get_influenced_users`.
**CN:** 检查 `len(user_inputs) > 0`，并在函数 `get_influenced_users` 中选择匹配的分支。

#### Line 112 — Comment or spacing block

```python
    # Get all influenced nodes if we propagate along the input direction
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 113 — Function `get_influenced_inputs`

```python
    def get_influenced_inputs(self, node: str):
```
**EN:** Defines function `get_influenced_inputs` with parameters `self, node`.
**CN:** 定义函数 `get_influenced_inputs`，参数为 `self, node`。

##### Lines 114-115 — Conditional `node in self.visited`

```python
        if node in self.visited:
            return
```
**EN:** Checks `node in self.visited` and selects the matching branch in function `get_influenced_inputs`.
**CN:** 检查 `node in self.visited`，并在函数 `get_influenced_inputs` 中选择匹配的分支。

##### Line 116 — Call `self.visited.append`

```python
        self.visited.append(node)
```
**EN:** Calls `self.visited.append` for side effects or initialization work in function `get_influenced_inputs`.
**CN:** 调用 `self.visited.append` 执行副作用或初始化逻辑；该语句位于在函数 `get_influenced_inputs` 中。

##### Line 118 — Assign `inputs`

```python
        inputs = self.dag_ir.get_all_inputs(node)
```
**EN:** Assigns `inputs` from `self.dag_ir.get_all_inputs(node)`, establishing state in function `get_influenced_inputs`.
**CN:** 将 `inputs` 赋值为 `self.dag_ir.get_all_inputs(node)`，用于在函数 `get_influenced_inputs` 中建立状态。

##### Lines 119-120 — Loop over `inputs`

```python
        for input in inputs:
            self.get_influenced_inputs(input)
```
**EN:** Iterates `input` over `inputs` to repeat a processing step.
**CN:** 让 `input` 遍历 `inputs`，从而重复执行处理步骤。

##### Line 121 — Assign `input_users`

```python
        input_users = []
```
**EN:** Assigns `input_users` from `[]`, establishing state in function `get_influenced_inputs`.
**CN:** 将 `input_users` 赋值为 `[]`，用于在函数 `get_influenced_inputs` 中建立状态。

##### Lines 122-123 — Loop over `inputs`

```python
        for input in inputs:
            input_users.append(set(self.dag_ir.get_users(input)))
```
**EN:** Iterates `input` over `inputs` to repeat a processing step.
**CN:** 让 `input` 遍历 `inputs`，从而重复执行处理步骤。

##### Lines 124-128 — Conditional `len(input_users) > 0`

```python
        if len(input_users) > 0:
            input_users = set.union(*input_users)
            input_users.remove(node)
            for user in input_users:
                self.get_influenced_users(user)
```
**EN:** Checks `len(input_users) > 0` and selects the matching branch in function `get_influenced_inputs`.
**CN:** 检查 `len(input_users) > 0`，并在函数 `get_influenced_inputs` 中选择匹配的分支。

#### Line 130 — Function `add_copy_before`

```python
    def add_copy_before(self, layout_node_meta: LayoutNode, target: str):
```
**EN:** Defines function `add_copy_before` with parameters `self, layout_node_meta, target`.
**CN:** 定义函数 `add_copy_before`，参数为 `self, layout_node_meta, target`。

##### Line 131 — Assign `copied_node_meta`

```python
        copied_node_meta = deepcopy(layout_node_meta)
```
**EN:** Assigns `copied_node_meta` from `deepcopy(layout_node_meta)`, establishing state in function `add_copy_before`.
**CN:** 将 `copied_node_meta` 赋值为 `deepcopy(layout_node_meta)`，用于在函数 `add_copy_before` 中建立状态。

##### Line 132 — Assign `copied_node`

```python
        copied_node = f"{copied_node_meta.name}_copy{self.copy_cnt}"
```
**EN:** Assigns `copied_node` from `f'{copied_node_meta.name}_copy{self.copy_cnt}'`, establishing state in function `add_copy_before`.
**CN:** 将 `copied_node` 赋值为 `f'{copied_node_meta.name}_copy{self.copy_cnt}'`，用于在函数 `add_copy_before` 中建立状态。

##### Line 133 — Update `self.copy_cnt`

```python
        self.copy_cnt += 1
```
**EN:** Updates `self.copy_cnt` with `+=` using `1`.
**CN:** 使用 `1` 对 `self.copy_cnt` 执行 `+=` 更新。

##### Line 134 — Assign `copied_node_meta.name`

```python
        copied_node_meta.name = copied_node
```
**EN:** Assigns `copied_node_meta.name` from `copied_node`, establishing state in function `add_copy_before`.
**CN:** 将 `copied_node_meta.name` 赋值为 `copied_node`，用于在函数 `add_copy_before` 中建立状态。

##### Line 135 — Call `self.dag_ir.add_node`

```python
        self.dag_ir.add_node(copied_node_meta)
```
**EN:** Calls `self.dag_ir.add_node` for side effects or initialization work in function `add_copy_before`.
**CN:** 调用 `self.dag_ir.add_node` 执行副作用或初始化逻辑；该语句位于在函数 `add_copy_before` 中。

##### Line 136 — Comment or spacing block

```python
        # Add edges
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 137 — Assign `target_inputs`

```python
        target_inputs = self.dag_ir.get_all_inputs(target)
```
**EN:** Assigns `target_inputs` from `self.dag_ir.get_all_inputs(target)`, establishing state in function `add_copy_before`.
**CN:** 将 `target_inputs` 赋值为 `self.dag_ir.get_all_inputs(target)`，用于在函数 `add_copy_before` 中建立状态。

##### Lines 138-140 — Loop over `target_inputs`

```python
        for src in target_inputs:
            self.dag_ir.remove_edge(src, target)
            self.dag_ir.add_edge(src, copied_node)
```
**EN:** Iterates `src` over `target_inputs` to repeat a processing step.
**CN:** 让 `src` 遍历 `target_inputs`，从而重复执行处理步骤。

##### Line 141 — Call `self.dag_ir.add_edge`

```python
        self.dag_ir.add_edge(copied_node, target)
```
**EN:** Calls `self.dag_ir.add_edge` for side effects or initialization work in function `add_copy_before`.
**CN:** 调用 `self.dag_ir.add_edge` 执行副作用或初始化逻辑；该语句位于在函数 `add_copy_before` 中。

##### Line 142 — Call `self.layout_nodes_worklist.append`

```python
        self.layout_nodes_worklist.append(copied_node)
```
**EN:** Calls `self.layout_nodes_worklist.append` for side effects or initialization work in function `add_copy_before`.
**CN:** 调用 `self.layout_nodes_worklist.append` 执行副作用或初始化逻辑；该语句位于在函数 `add_copy_before` 中。

#### Line 144 — Function `add_copy_after`

```python
    def add_copy_after(self, layout_node_meta: LayoutNode, target: str):
```
**EN:** Defines function `add_copy_after` with parameters `self, layout_node_meta, target`.
**CN:** 定义函数 `add_copy_after`，参数为 `self, layout_node_meta, target`。

##### Line 145 — Assign `copied_node_meta`

```python
        copied_node_meta = deepcopy(layout_node_meta)
```
**EN:** Assigns `copied_node_meta` from `deepcopy(layout_node_meta)`, establishing state in function `add_copy_after`.
**CN:** 将 `copied_node_meta` 赋值为 `deepcopy(layout_node_meta)`，用于在函数 `add_copy_after` 中建立状态。

##### Line 146 — Assign `copied_node`

```python
        copied_node = f"{copied_node_meta.name}_copy{self.copy_cnt}"
```
**EN:** Assigns `copied_node` from `f'{copied_node_meta.name}_copy{self.copy_cnt}'`, establishing state in function `add_copy_after`.
**CN:** 将 `copied_node` 赋值为 `f'{copied_node_meta.name}_copy{self.copy_cnt}'`，用于在函数 `add_copy_after` 中建立状态。

##### Line 147 — Update `self.copy_cnt`

```python
        self.copy_cnt += 1
```
**EN:** Updates `self.copy_cnt` with `+=` using `1`.
**CN:** 使用 `1` 对 `self.copy_cnt` 执行 `+=` 更新。

##### Line 148 — Assign `copied_node_meta.name`

```python
        copied_node_meta.name = copied_node
```
**EN:** Assigns `copied_node_meta.name` from `copied_node`, establishing state in function `add_copy_after`.
**CN:** 将 `copied_node_meta.name` 赋值为 `copied_node`，用于在函数 `add_copy_after` 中建立状态。

##### Line 149 — Call `self.dag_ir.add_node`

```python
        self.dag_ir.add_node(copied_node_meta)
```
**EN:** Calls `self.dag_ir.add_node` for side effects or initialization work in function `add_copy_after`.
**CN:** 调用 `self.dag_ir.add_node` 执行副作用或初始化逻辑；该语句位于在函数 `add_copy_after` 中。

##### Line 150 — Comment or spacing block

```python
        # Add edges
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 151 — Assign `users`

```python
        users = self.dag_ir.get_users(target)
```
**EN:** Assigns `users` from `self.dag_ir.get_users(target)`, establishing state in function `add_copy_after`.
**CN:** 将 `users` 赋值为 `self.dag_ir.get_users(target)`，用于在函数 `add_copy_after` 中建立状态。

##### Lines 152-154 — Loop over `users`

```python
        for user in users:
            self.dag_ir.remove_edge(target, user)
            self.dag_ir.add_edge(copied_node, user)
```
**EN:** Iterates `user` over `users` to repeat a processing step.
**CN:** 让 `user` 遍历 `users`，从而重复执行处理步骤。

##### Line 155 — Call `self.dag_ir.add_edge`

```python
        self.dag_ir.add_edge(target, copied_node)
```
**EN:** Calls `self.dag_ir.add_edge` for side effects or initialization work in function `add_copy_after`.
**CN:** 调用 `self.dag_ir.add_edge` 执行副作用或初始化逻辑；该语句位于在函数 `add_copy_after` 中。

##### Line 156 — Call `self.layout_nodes_worklist.append`

```python
        self.layout_nodes_worklist.append(copied_node)
```
**EN:** Calls `self.layout_nodes_worklist.append` for side effects or initialization work in function `add_copy_after`.
**CN:** 调用 `self.layout_nodes_worklist.append` 执行副作用或初始化逻辑；该语句位于在函数 `add_copy_after` 中。

#### Line 158 — Comment or spacing block

```python
    # Propagate the layout `node` along the user direction
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 159-162 — Function `propagate_to_users`

```python
    def propagate_to_users(self, layout_node_meta: LayoutNode, node: str):
        """
        Propagate layout node to users
        """
```
**EN:** Defines function `propagate_to_users` with parameters `self, layout_node_meta, node`. Purpose: Propagate layout node to users
**CN:** 定义函数 `propagate_to_users`，参数为 `self, layout_node_meta, node`。 其用途：Propagate layout node to users

##### Lines 163-165 — Conditional `node in self.visited`

```python
        if node in self.visited:
            # Avoid applying twice
            return
```
**EN:** Checks `node in self.visited` and selects the matching branch in function `propagate_to_users`.
**CN:** 检查 `node in self.visited`，并在函数 `propagate_to_users` 中选择匹配的分支。

##### Line 166 — Call `self.visited.append`

```python
        self.visited.append(node)
```
**EN:** Calls `self.visited.append` for side effects or initialization work in function `propagate_to_users`.
**CN:** 调用 `self.visited.append` 执行副作用或初始化逻辑；该语句位于在函数 `propagate_to_users` 中。

##### Line 168 — Assign `node_meta`

```python
        node_meta = self.dag_ir.get_node_meta(node)
```
**EN:** Assigns `node_meta` from `self.dag_ir.get_node_meta(node)`, establishing state in function `propagate_to_users`.
**CN:** 将 `node_meta` 赋值为 `self.dag_ir.get_node_meta(node)`，用于在函数 `propagate_to_users` 中建立状态。

##### Lines 169-175 — Conditional `layout_node_meta.name != node`

```python
        if layout_node_meta.name != node:
            if isinstance(node_meta, LayoutNode):
                # Layout node is not transparent with layout node
                self.add_copy_before(layout_node_meta, node)
                return
            else:
                layout_node_meta.apply_to_user(node_meta)
```
**EN:** Checks `layout_node_meta.name != node` and selects the matching branch in function `propagate_to_users`.
**CN:** 检查 `layout_node_meta.name != node`，并在函数 `propagate_to_users` 中选择匹配的分支。

##### Line 177 — Assign `users`

```python
        users = self.dag_ir.get_users(node)
```
**EN:** Assigns `users` from `self.dag_ir.get_users(node)`, establishing state in function `propagate_to_users`.
**CN:** 将 `users` 赋值为 `self.dag_ir.get_users(node)`，用于在函数 `propagate_to_users` 中建立状态。

##### Line 178 — Assign `user_inputs`

```python
        user_inputs = []
```
**EN:** Assigns `user_inputs` from `[]`, establishing state in function `propagate_to_users`.
**CN:** 将 `user_inputs` 赋值为 `[]`，用于在函数 `propagate_to_users` 中建立状态。

##### Lines 179-180 — Loop over `users`

```python
        for user in users:
            user_inputs.append(set(self.dag_ir.get_all_inputs(user)))
```
**EN:** Iterates `user` over `users` to repeat a processing step.
**CN:** 让 `user` 遍历 `users`，从而重复执行处理步骤。

##### Lines 181-182 — Loop over `users`

```python
        for user in users:
            self.propagate_to_users(layout_node_meta, user)
```
**EN:** Iterates `user` over `users` to repeat a processing step.
**CN:** 让 `user` 遍历 `users`，从而重复执行处理步骤。

##### Lines 183-187 — Conditional `len(user_inputs) > 0`

```python
        if len(user_inputs) > 0:
            user_inputs = set.union(*user_inputs)
            user_inputs.remove(node)
            for input in user_inputs:
                self.propagate_to_inputs(layout_node_meta.get_inverse_node(), input)
```
**EN:** Checks `len(user_inputs) > 0` and selects the matching branch in function `propagate_to_users`.
**CN:** 检查 `len(user_inputs) > 0`，并在函数 `propagate_to_users` 中选择匹配的分支。

#### Line 189 — Comment or spacing block

```python
    # Propagate the layout `node` along the input direction
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 190-193 — Function `propagate_to_inputs`

```python
    def propagate_to_inputs(self, layout_node_meta: LayoutNode, node: str):
        """
        Propagate layout node to inputs
        """
```
**EN:** Defines function `propagate_to_inputs` with parameters `self, layout_node_meta, node`. Purpose: Propagate layout node to inputs
**CN:** 定义函数 `propagate_to_inputs`，参数为 `self, layout_node_meta, node`。 其用途：Propagate layout node to inputs

##### Lines 194-196 — Conditional `node in self.visited`

```python
        if node in self.visited:
            # Avoid applying twice
            return
```
**EN:** Checks `node in self.visited` and selects the matching branch in function `propagate_to_inputs`.
**CN:** 检查 `node in self.visited`，并在函数 `propagate_to_inputs` 中选择匹配的分支。

##### Line 197 — Call `self.visited.append`

```python
        self.visited.append(node)
```
**EN:** Calls `self.visited.append` for side effects or initialization work in function `propagate_to_inputs`.
**CN:** 调用 `self.visited.append` 执行副作用或初始化逻辑；该语句位于在函数 `propagate_to_inputs` 中。

##### Line 199 — Assign `node_meta`

```python
        node_meta = self.dag_ir.get_node_meta(node)
```
**EN:** Assigns `node_meta` from `self.dag_ir.get_node_meta(node)`, establishing state in function `propagate_to_inputs`.
**CN:** 将 `node_meta` 赋值为 `self.dag_ir.get_node_meta(node)`，用于在函数 `propagate_to_inputs` 中建立状态。

##### Lines 200-206 — Conditional `layout_node_meta.name != node`

```python
        if layout_node_meta.name != node:
            if isinstance(node_meta, LayoutNode):
                # Layout node is not transparent with layout node
                self.add_copy_after(layout_node_meta, node)
                return
            else:
                layout_node_meta.apply_to_input(node_meta)
```
**EN:** Checks `layout_node_meta.name != node` and selects the matching branch in function `propagate_to_inputs`.
**CN:** 检查 `layout_node_meta.name != node`，并在函数 `propagate_to_inputs` 中选择匹配的分支。

##### Line 207 — Assign `inputs`

```python
        inputs = self.dag_ir.get_all_inputs(node)
```
**EN:** Assigns `inputs` from `self.dag_ir.get_all_inputs(node)`, establishing state in function `propagate_to_inputs`.
**CN:** 将 `inputs` 赋值为 `self.dag_ir.get_all_inputs(node)`，用于在函数 `propagate_to_inputs` 中建立状态。

##### Line 208 — Assign `input_users`

```python
        input_users = []
```
**EN:** Assigns `input_users` from `[]`, establishing state in function `propagate_to_inputs`.
**CN:** 将 `input_users` 赋值为 `[]`，用于在函数 `propagate_to_inputs` 中建立状态。

##### Lines 209-210 — Loop over `inputs`

```python
        for input in inputs:
            input_users.append(set(self.dag_ir.get_users(input)))
```
**EN:** Iterates `input` over `inputs` to repeat a processing step.
**CN:** 让 `input` 遍历 `inputs`，从而重复执行处理步骤。

##### Lines 211-212 — Loop over `inputs`

```python
        for input in inputs:
            self.propagate_to_inputs(layout_node_meta, input)
```
**EN:** Iterates `input` over `inputs` to repeat a processing step.
**CN:** 让 `input` 遍历 `inputs`，从而重复执行处理步骤。

##### Lines 213-217 — Conditional `len(input_users) > 0`

```python
        if len(input_users) > 0:
            input_users = set.union(*input_users)
            input_users.remove(node)
            for user in input_users:
                self.propagate_to_users(layout_node_meta.get_inverse_node(), user)
```
**EN:** Checks `len(input_users) > 0` and selects the matching branch in function `propagate_to_inputs`.
**CN:** 检查 `len(input_users) > 0`，并在函数 `propagate_to_inputs` 中选择匹配的分支。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `PassLayoutManipulateElimination`.
- **CN:** 顶层类：`PassLayoutManipulateElimination`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen.backend.evt.ir`, `cutlass_cppgen.backend.evt.passes.pass_manager`, `cutlass_cppgen.backend.evt.passes.pass_shape_type_propagation`
- **Standard & third-party / 标准库与第三方:** `copy`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
