# layout_nodes.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/evt/ir/layout_nodes.py`
- **EN:** Layout manipulation nodes and implementations
- **CN:** 模块文档说明：Layout manipulation nodes and implementations

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

### Lines 33-37 — Docstring

```python
"""
Layout manipulation nodes and implementations

The layout Nodes change the layout of intermediate nodes in epilogue visitor graph
"""
```
**EN:** Docstring explains this scope: Layout manipulation nodes and implementations
**CN:** 文档字符串说明了该作用域的用途：Layout manipulation nodes and implementations

### Line 39 — From `copy` import

```python
from copy import deepcopy
```
**EN:** Imports `deepcopy` from `copy` to reuse shared definitions at module scope.
**CN:** 从 `copy` 导入 `deepcopy`，以便后续代码在模块级复用共享定义。

### Line 41 — From `cutlass_library` import

```python
from cutlass_library import LayoutType
```
**EN:** Imports `LayoutType` from `cutlass_library` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library` 导入 `LayoutType`，以便后续代码在模块级复用共享定义。

### Line 42 — From `pycute` import

```python
from pycute import product, flatten
```
**EN:** Imports `product, flatten` from `pycute` to reuse shared definitions at module scope.
**CN:** 从 `pycute` 导入 `product, flatten`，以便后续代码在模块级复用共享定义。

### Line 44 — Import `cutlass_cppgen`

```python
import cutlass_cppgen
```
**EN:** Imports `cutlass_cppgen` so later code can use these APIs at module scope.
**CN:** 导入 `cutlass_cppgen`，供后续代码在模块级使用这些 API。

### Line 45 — From `cutlass_cppgen.backend.evt.ir.layout_algorithm` import

```python
from cutlass_cppgen.backend.evt.ir.layout_algorithm import _list_to_tuple, _tuple_to_list
```
**EN:** Imports `_list_to_tuple, _tuple_to_list` from `cutlass_cppgen.backend.evt.ir.layout_algorithm` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir.layout_algorithm` 导入 `_list_to_tuple, _tuple_to_list`，以便后续代码在模块级复用共享定义。

### Line 46 — From `cutlass_cppgen.backend.evt.ir.node` import

```python
from cutlass_cppgen.backend.evt.ir.node import NodeBase
```
**EN:** Imports `NodeBase` from `cutlass_cppgen.backend.evt.ir.node` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir.node` 导入 `NodeBase`，以便后续代码在模块级复用共享定义。

### Line 47 — From `cutlass_cppgen.backend.evt.ir.tensor` import

```python
from cutlass_cppgen.backend.evt.ir.tensor import Tensor
```
**EN:** Imports `Tensor` from `cutlass_cppgen.backend.evt.ir.tensor` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.backend.evt.ir.tensor` 导入 `Tensor`，以便后续代码在模块级复用共享定义。

### Lines 50-53 — Class `PermutationImpl`

```python
class PermutationImpl:
    """
    Detailed implementation and helper functions for permutation
    """
```
**EN:** Declares class `PermutationImpl` deriving from `object`. Purpose: Detailed implementation and helper functions for permutation
**CN:** 声明类 `PermutationImpl`，其基类为 `object`。 其用途：Detailed implementation and helper functions for permutation

#### Line 54 — Function `__init__`

```python
    def __init__(self, node) -> None:
```
**EN:** Defines function `__init__` with parameters `self, node`.
**CN:** 定义函数 `__init__`，参数为 `self, node`。

##### Line 55 — Assertion

```python
        assert "indices" in node.kwargs.keys()
```
**EN:** Asserts `'indices' in node.kwargs.keys()` to enforce an expected condition.
**CN:** 断言 `'indices' in node.kwargs.keys()`，用于保证预期条件成立。

##### Line 56 — Assign `self.indices`

```python
        self.indices = list(node.kwargs["indices"])
```
**EN:** Assigns `self.indices` from `list(node.kwargs['indices'])`, establishing state in function `__init__`.
**CN:** 将 `self.indices` 赋值为 `list(node.kwargs['indices'])`，用于在函数 `__init__` 中建立状态。

##### Line 57 — Assign `self.inverse_indices`

```python
        self.inverse_indices = self.get_inverse_indices(self.indices)
```
**EN:** Assigns `self.inverse_indices` from `self.get_inverse_indices(self.indices)`, establishing state in function `__init__`.
**CN:** 将 `self.inverse_indices` 赋值为 `self.get_inverse_indices(self.indices)`，用于在函数 `__init__` 中建立状态。

#### Line 59 — Function `get_inverse_impl`

```python
    def get_inverse_impl(self):
```
**EN:** Defines function `get_inverse_impl` with parameters `self`.
**CN:** 定义函数 `get_inverse_impl`，参数为 `self`。

##### Line 60 — Assign `inverse_impl`

```python
        inverse_impl = deepcopy(self)
```
**EN:** Assigns `inverse_impl` from `deepcopy(self)`, establishing state in function `get_inverse_impl`.
**CN:** 将 `inverse_impl` 赋值为 `deepcopy(self)`，用于在函数 `get_inverse_impl` 中建立状态。

##### Line 61 — Assign `inverse_impl.indices`

```python
        inverse_impl.indices = self.inverse_indices
```
**EN:** Assigns `inverse_impl.indices` from `self.inverse_indices`, establishing state in function `get_inverse_impl`.
**CN:** 将 `inverse_impl.indices` 赋值为 `self.inverse_indices`，用于在函数 `get_inverse_impl` 中建立状态。

##### Line 62 — Assign `inverse_impl.inverse_indices`

```python
        inverse_impl.inverse_indices = self.indices
```
**EN:** Assigns `inverse_impl.inverse_indices` from `self.indices`, establishing state in function `get_inverse_impl`.
**CN:** 将 `inverse_impl.inverse_indices` 赋值为 `self.indices`，用于在函数 `get_inverse_impl` 中建立状态。

##### Line 63 — Return

```python
        return inverse_impl
```
**EN:** Returns `inverse_impl` to the caller.
**CN:** 向调用方返回 `inverse_impl`。

#### Line 65 — Function `update`

```python
    def update(self, shape):
```
**EN:** Defines function `update` with parameters `self, shape`.
**CN:** 定义函数 `update`，参数为 `self, shape`。

##### Line 66 — Assign `num_dim`

```python
        num_dim = len(shape)
```
**EN:** Assigns `num_dim` from `len(shape)`, establishing state in function `update`.
**CN:** 将 `num_dim` 赋值为 `len(shape)`，用于在函数 `update` 中建立状态。

##### Line 67 — Assign `indices`

```python
        indices = self.indices
```
**EN:** Assigns `indices` from `self.indices`, establishing state in function `update`.
**CN:** 将 `indices` 赋值为 `self.indices`，用于在函数 `update` 中建立状态。

##### Line 68 — Assign `num_old_dim`

```python
        num_old_dim = len(indices)
```
**EN:** Assigns `num_old_dim` from `len(indices)`, establishing state in function `update`.
**CN:** 将 `num_old_dim` 赋值为 `len(indices)`，用于在函数 `update` 中建立状态。

##### Line 69 — Comment or spacing block

```python
        # Add offset
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 70-71 — Loop over `enumerate(indices)`

```python
        for i, idx in enumerate(indices):
            indices[i] = idx + num_dim - num_old_dim
```
**EN:** Iterates `(i, idx)` over `enumerate(indices)` to repeat a processing step.
**CN:** 让 `(i, idx)` 遍历 `enumerate(indices)`，从而重复执行处理步骤。

##### Line 72 — Comment or spacing block

```python
        # Add broadcast dims
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 73-74 — Loop over `range(num_dim - num_old_dim)`

```python
        for i in range(num_dim - num_old_dim):
            indices = [i,] + indices
```
**EN:** Iterates `i` over `range(num_dim - num_old_dim)` to repeat a processing step.
**CN:** 让 `i` 遍历 `range(num_dim - num_old_dim)`，从而重复执行处理步骤。

##### Line 76 — Assign `self.indices`

```python
        self.indices = indices
```
**EN:** Assigns `self.indices` from `indices`, establishing state in function `update`.
**CN:** 将 `self.indices` 赋值为 `indices`，用于在函数 `update` 中建立状态。

##### Line 77 — Assign `self.inverse_indices`

```python
        self.inverse_indices = self.get_inverse_indices(self.indices)
```
**EN:** Assigns `self.inverse_indices` from `self.get_inverse_indices(self.indices)`, establishing state in function `update`.
**CN:** 将 `self.inverse_indices` 赋值为 `self.get_inverse_indices(self.indices)`，用于在函数 `update` 中建立状态。

#### Lines 79-82 — Function `get_inverse_indices`

```python
    def get_inverse_indices(self, indices):
        """
        Get the indices for inverse permutation
        """
```
**EN:** Defines function `get_inverse_indices` with parameters `self, indices`. Purpose: Get the indices for inverse permutation
**CN:** 定义函数 `get_inverse_indices`，参数为 `self, indices`。 其用途：Get the indices for inverse permutation

##### Line 83 — Assign `num_dim`

```python
        num_dim = len(indices)
```
**EN:** Assigns `num_dim` from `len(indices)`, establishing state in function `get_inverse_indices`.
**CN:** 将 `num_dim` 赋值为 `len(indices)`，用于在函数 `get_inverse_indices` 中建立状态。

##### Line 84 — Assign `inverse_indices`

```python
        inverse_indices = [0] * num_dim
```
**EN:** Assigns `inverse_indices` from `[0] * num_dim`, establishing state in function `get_inverse_indices`.
**CN:** 将 `inverse_indices` 赋值为 `[0] * num_dim`，用于在函数 `get_inverse_indices` 中建立状态。

##### Lines 85-86 — Loop over `range(num_dim)`

```python
        for i in range(num_dim):
            inverse_indices[indices[i]] = i
```
**EN:** Iterates `i` over `range(num_dim)` to repeat a processing step.
**CN:** 让 `i` 遍历 `range(num_dim)`，从而重复执行处理步骤。

##### Line 87 — Return

```python
        return inverse_indices
```
**EN:** Returns `inverse_indices` to the caller.
**CN:** 向调用方返回 `inverse_indices`。

#### Line 89 — Function `shape_propagation`

```python
    def shape_propagation(self, input_node_meta):
```
**EN:** Defines function `shape_propagation` with parameters `self, input_node_meta`.
**CN:** 定义函数 `shape_propagation`，参数为 `self, input_node_meta`。

##### Line 90 — Assign `input_shape`

```python
        input_shape = input_node_meta.tensor.shape
```
**EN:** Assigns `input_shape` from `input_node_meta.tensor.shape`, establishing state in function `shape_propagation`.
**CN:** 将 `input_shape` 赋值为 `input_node_meta.tensor.shape`，用于在函数 `shape_propagation` 中建立状态。

##### Line 91 — Assign `output_shape`

```python
        output_shape = tuple([input_shape[idx] for idx in self.indices])
```
**EN:** Assigns `output_shape` from `tuple([input_shape[idx] for idx in self.indices])`, establishing state in function `shape_propagation`.
**CN:** 将 `output_shape` 赋值为 `tuple([input_shape[idx] for idx in self.indices])`，用于在函数 `shape_propagation` 中建立状态。

##### Line 92 — Return

```python
        return output_shape
```
**EN:** Returns `output_shape` to the caller.
**CN:** 向调用方返回 `output_shape`。

#### Lines 94-97 — Function `broadcast`

```python
    def broadcast(self, shape, node_meta: NodeBase):
        """
        Broadcast the inputs based on current shape
        """
```
**EN:** Defines function `broadcast` with parameters `self, shape, node_meta`. Purpose: Broadcast the inputs based on current shape
**CN:** 定义函数 `broadcast`，参数为 `self, shape, node_meta`。 其用途：Broadcast the inputs based on current shape

##### Line 98 — Call `self.update`

```python
        self.update(shape)
```
**EN:** Calls `self.update` for side effects or initialization work in function `broadcast`.
**CN:** 调用 `self.update` 执行副作用或初始化逻辑；该语句位于在函数 `broadcast` 中。

##### Line 99 — Assign `inverse_shape`

```python
        inverse_shape = tuple([shape[idx] for idx in self.inverse_indices])
```
**EN:** Assigns `inverse_shape` from `tuple([shape[idx] for idx in self.inverse_indices])`, establishing state in function `broadcast`.
**CN:** 将 `inverse_shape` 赋值为 `tuple([shape[idx] for idx in self.inverse_indices])`，用于在函数 `broadcast` 中建立状态。

##### Line 100 — Call `node_meta.tensor.broadcast`

```python
        node_meta.tensor.broadcast(inverse_shape)
```
**EN:** Calls `node_meta.tensor.broadcast` for side effects or initialization work in function `broadcast`.
**CN:** 调用 `node_meta.tensor.broadcast` 执行副作用或初始化逻辑；该语句位于在函数 `broadcast` 中。

#### Lines 102-105 — Function `apply_to_user`

```python
    def apply_to_user(self, usr_meta: NodeBase):
        """
        Propagate the permutation to the users of the current nodes
        """
```
**EN:** Defines function `apply_to_user` with parameters `self, usr_meta`. Purpose: Propagate the permutation to the users of the current nodes
**CN:** 定义函数 `apply_to_user`，参数为 `self, usr_meta`。 其用途：Propagate the permutation to the users of the current nodes

##### Line 106 — Call `usr_meta.tensor.permute`

```python
        usr_meta.tensor.permute(self.inverse_indices)
```
**EN:** Calls `usr_meta.tensor.permute` for side effects or initialization work in function `apply_to_user`.
**CN:** 调用 `usr_meta.tensor.permute` 执行副作用或初始化逻辑；该语句位于在函数 `apply_to_user` 中。

##### Lines 107-109 — Conditional `hasattr(usr_meta, 'store_tensor')`

```python
        if hasattr(usr_meta, "store_tensor"):
            if usr_meta.store_tensor is not None:
                usr_meta.store_tensor.permute(self.inverse_indices)
```
**EN:** Checks `hasattr(usr_meta, 'store_tensor')` and selects the matching branch in function `apply_to_user`.
**CN:** 检查 `hasattr(usr_meta, 'store_tensor')`，并在函数 `apply_to_user` 中选择匹配的分支。

#### Lines 111-114 — Function `apply_to_input`

```python
    def apply_to_input(self, input_meta: NodeBase):
        """
        Propagate the permutation to inputs of the current nodes
        """
```
**EN:** Defines function `apply_to_input` with parameters `self, input_meta`. Purpose: Propagate the permutation to inputs of the current nodes
**CN:** 定义函数 `apply_to_input`，参数为 `self, input_meta`。 其用途：Propagate the permutation to inputs of the current nodes

##### Line 115 — Call `input_meta.tensor.permute`

```python
        input_meta.tensor.permute(self.indices)
```
**EN:** Calls `input_meta.tensor.permute` for side effects or initialization work in function `apply_to_input`.
**CN:** 调用 `input_meta.tensor.permute` 执行副作用或初始化逻辑；该语句位于在函数 `apply_to_input` 中。

##### Lines 116-118 — Conditional `hasattr(input_meta, 'store_tensor')`

```python
        if hasattr(input_meta, "store_tensor"):
            if input_meta.store_tensor is not None:
                input_meta.store_tensor.permute(self.indices)
```
**EN:** Checks `hasattr(input_meta, 'store_tensor')` and selects the matching branch in function `apply_to_input`.
**CN:** 检查 `hasattr(input_meta, 'store_tensor')`，并在函数 `apply_to_input` 中选择匹配的分支。

### Lines 121-124 — Class `ReshapeImpl`

```python
class ReshapeImpl:
    """
    Detailed implementation and helper functions for reshape
    """
```
**EN:** Declares class `ReshapeImpl` deriving from `object`. Purpose: Detailed implementation and helper functions for reshape
**CN:** 声明类 `ReshapeImpl`，其基类为 `object`。 其用途：Detailed implementation and helper functions for reshape

#### Line 125 — Function `__init__`

```python
    def __init__(self, node) -> None:
```
**EN:** Defines function `__init__` with parameters `self, node`.
**CN:** 定义函数 `__init__`，参数为 `self, node`。

##### Line 126 — Assign `self.node`

```python
        self.node = node
```
**EN:** Assigns `self.node` from `node`, establishing state in function `__init__`.
**CN:** 将 `self.node` 赋值为 `node`，用于在函数 `__init__` 中建立状态。

##### Line 127 — Assertion

```python
        assert "new_shape" in node.kwargs.keys()
```
**EN:** Asserts `'new_shape' in node.kwargs.keys()` to enforce an expected condition.
**CN:** 断言 `'new_shape' in node.kwargs.keys()`，用于保证预期条件成立。

##### Line 128 — Assign `self.output_shape`

```python
        self.output_shape = _list_to_tuple(node.kwargs["new_shape"])
```
**EN:** Assigns `self.output_shape` from `_list_to_tuple(node.kwargs['new_shape'])`, establishing state in function `__init__`.
**CN:** 将 `self.output_shape` 赋值为 `_list_to_tuple(node.kwargs['new_shape'])`，用于在函数 `__init__` 中建立状态。

#### Line 130 — Function `get_inverse_impl`

```python
    def get_inverse_impl(self):
```
**EN:** Defines function `get_inverse_impl` with parameters `self`.
**CN:** 定义函数 `get_inverse_impl`，参数为 `self`。

##### Line 131 — Assign `inverse_impl`

```python
        inverse_impl = deepcopy(self)
```
**EN:** Assigns `inverse_impl` from `deepcopy(self)`, establishing state in function `get_inverse_impl`.
**CN:** 将 `inverse_impl` 赋值为 `deepcopy(self)`，用于在函数 `get_inverse_impl` 中建立状态。

##### Line 132 — Assign `inverse_impl.output_shape`

```python
        inverse_impl.output_shape = self.input_shape
```
**EN:** Assigns `inverse_impl.output_shape` from `self.input_shape`, establishing state in function `get_inverse_impl`.
**CN:** 将 `inverse_impl.output_shape` 赋值为 `self.input_shape`，用于在函数 `get_inverse_impl` 中建立状态。

##### Line 133 — Assign `inverse_impl.input_shape`

```python
        inverse_impl.input_shape = self.output_shape
```
**EN:** Assigns `inverse_impl.input_shape` from `self.output_shape`, establishing state in function `get_inverse_impl`.
**CN:** 将 `inverse_impl.input_shape` 赋值为 `self.output_shape`，用于在函数 `get_inverse_impl` 中建立状态。

##### Line 134 — Return

```python
        return inverse_impl
```
**EN:** Returns `inverse_impl` to the caller.
**CN:** 向调用方返回 `inverse_impl`。

#### Line 136 — Function `shape_propagation`

```python
    def shape_propagation(self, input_node_meta):
```
**EN:** Defines function `shape_propagation` with parameters `self, input_node_meta`.
**CN:** 定义函数 `shape_propagation`，参数为 `self, input_node_meta`。

##### Line 137 — Assign `self.input_shape`

```python
        self.input_shape = input_node_meta.tensor.shape
```
**EN:** Assigns `self.input_shape` from `input_node_meta.tensor.shape`, establishing state in function `shape_propagation`.
**CN:** 将 `self.input_shape` 赋值为 `input_node_meta.tensor.shape`，用于在函数 `shape_propagation` 中建立状态。

##### Line 138 — Return

```python
        return _list_to_tuple(self.output_shape)
```
**EN:** Returns `_list_to_tuple(self.output_shape)` to the caller.
**CN:** 向调用方返回 `_list_to_tuple(self.output_shape)`。

#### Lines 140-143 — Function `broadcast`

```python
    def broadcast(self, shape, node_meta: NodeBase):
        """
        Broadcast the inputs based on current shape.
        """
```
**EN:** Defines function `broadcast` with parameters `self, shape, node_meta`. Purpose: Broadcast the inputs based on current shape.
**CN:** 定义函数 `broadcast`，参数为 `self, shape, node_meta`。 其用途：Broadcast the inputs based on current shape.

##### Line 144 — Comment or spacing block

```python
        # Step 1: infer split
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 145 — Assign `flatten_split_shape`

```python
        flatten_split_shape = self.infer_split(flatten(self.input_shape), flatten(self.output_shape))
```
**EN:** Assigns `flatten_split_shape` from `self.infer_split(flatten(self.input_shape), flatten(self.output_shape))`, establishing state in function `broadcast`.
**CN:** 将 `flatten_split_shape` 赋值为 `self.infer_split(flatten(self.input_shape), flatten(self.output_shape))`，用于在函数 `broadcast` 中建立状态。

##### Line 146 — Assign `split_input_shape`

```python
        split_input_shape = self.infer_merge(flatten_split_shape, self.input_shape)
```
**EN:** Assigns `split_input_shape` from `self.infer_merge(flatten_split_shape, self.input_shape)`, establishing state in function `broadcast`.
**CN:** 将 `split_input_shape` 赋值为 `self.infer_merge(flatten_split_shape, self.input_shape)`，用于在函数 `broadcast` 中建立状态。

##### Line 147 — Assign `split_output_shape`

```python
        split_output_shape = self.infer_merge(flatten_split_shape, self.output_shape)
```
**EN:** Assigns `split_output_shape` from `self.infer_merge(flatten_split_shape, self.output_shape)`, establishing state in function `broadcast`.
**CN:** 将 `split_output_shape` 赋值为 `self.infer_merge(flatten_split_shape, self.output_shape)`，用于在函数 `broadcast` 中建立状态。

##### Line 149 — Comment or spacing block

```python
        # broadcast shape -> split_output_shape -> flatten_split_shape
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 150-154 — Conditional `len(shape) - len(split_output_shape) > 0`

```python
        if len(shape) - len(split_output_shape) > 0:
            for _ in range(len(shape) - len(split_output_shape)):
                split_output_shape = [1,] + split_output_shape
                flatten_split_shape = [1,] + flatten_split_shape
                split_input_shape = [1,] + split_input_shape
```
**EN:** Checks `len(shape) - len(split_output_shape) > 0` and selects the matching branch in function `broadcast`.
**CN:** 检查 `len(shape) - len(split_output_shape) > 0`，并在函数 `broadcast` 中选择匹配的分支。

##### Line 155 — Assign `broadcast_factor`

```python
        broadcast_factor = []
```
**EN:** Assigns `broadcast_factor` from `[]`, establishing state in function `broadcast`.
**CN:** 将 `broadcast_factor` 赋值为 `[]`，用于在函数 `broadcast` 中建立状态。

##### Lines 156-167 — Loop over `zip(shape, split_output_shape)`

```python
        for dim, old_dim in zip(shape, split_output_shape):
            if not isinstance(dim, list):
                dim = [dim,]
            if not isinstance(old_dim, list):
                old_dim = [old_dim,]
            if product(tuple(dim)) == product(tuple(old_dim)):
                broadcast_factor += [1] * len(old_dim)
            elif product(tuple(old_dim)) == 1:
                assert len(dim) == 1
                broadcast_factor.append(dim[0])
            else:
                raise NotImplementedError(f"Invalid Broadcast: {old_dim} -> {dim}")
```
**EN:** Iterates `(dim, old_dim)` over `zip(shape, split_output_shape)` to repeat a processing step.
**CN:** 让 `(dim, old_dim)` 遍历 `zip(shape, split_output_shape)`，从而重复执行处理步骤。

##### Line 169 — Comment or spacing block

```python
        # flatten_split_shape -> split_input_shape
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 170 — Assign `factor_idx`

```python
        factor_idx = 0
```
**EN:** Assigns `factor_idx` from `0`, establishing state in function `broadcast`.
**CN:** 将 `factor_idx` 赋值为 `0`，用于在函数 `broadcast` 中建立状态。

##### Line 171 — Assign `broadcast_split_input_shape`

```python
        broadcast_split_input_shape = []
```
**EN:** Assigns `broadcast_split_input_shape` from `[]`, establishing state in function `broadcast`.
**CN:** 将 `broadcast_split_input_shape` 赋值为 `[]`，用于在函数 `broadcast` 中建立状态。

##### Lines 172-181 — Loop over `split_input_shape`

```python
        for dim in split_input_shape:
            if isinstance(dim, list):
                new_dim = []
                for d in dim:
                    new_dim.append(d * broadcast_factor[factor_idx])
                    factor_idx += 1
                broadcast_split_input_shape.append(new_dim)
            else:
                broadcast_split_input_shape.append(dim * broadcast_factor[factor_idx])
                factor_idx += 1
```
**EN:** Iterates `dim` over `split_input_shape` to repeat a processing step.
**CN:** 让 `dim` 遍历 `split_input_shape`，从而重复执行处理步骤。

##### Line 182 — Assign `broadcast_split_input_shape`

```python
        broadcast_split_input_shape = _list_to_tuple(broadcast_split_input_shape)
```
**EN:** Assigns `broadcast_split_input_shape` from `_list_to_tuple(broadcast_split_input_shape)`, establishing state in function `broadcast`.
**CN:** 将 `broadcast_split_input_shape` 赋值为 `_list_to_tuple(broadcast_split_input_shape)`，用于在函数 `broadcast` 中建立状态。

##### Line 183 — Call `node_meta.tensor.reshape`

```python
        node_meta.tensor.reshape(_list_to_tuple(split_input_shape))
```
**EN:** Calls `node_meta.tensor.reshape` for side effects or initialization work in function `broadcast`.
**CN:** 调用 `node_meta.tensor.reshape` 执行副作用或初始化逻辑；该语句位于在函数 `broadcast` 中。

##### Line 184 — Call `node_meta.tensor.broadcast`

```python
        node_meta.tensor.broadcast(broadcast_split_input_shape)
```
**EN:** Calls `node_meta.tensor.broadcast` for side effects or initialization work in function `broadcast`.
**CN:** 调用 `node_meta.tensor.broadcast` 执行副作用或初始化逻辑；该语句位于在函数 `broadcast` 中。

##### Line 185 — Comment or spacing block

```python
        # Last reshape op to clean up
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 186 — Assign `broadcast_input_shape`

```python
        broadcast_input_shape = tuple([product(dim) for dim in broadcast_split_input_shape])
```
**EN:** Assigns `broadcast_input_shape` from `tuple([product(dim) for dim in broadcast_split_input_shape])`, establishing state in function `broadcast`.
**CN:** 将 `broadcast_input_shape` 赋值为 `tuple([product(dim) for dim in broadcast_split_input_shape])`，用于在函数 `broadcast` 中建立状态。

##### Line 187 — Call `node_meta.tensor.reshape`

```python
        node_meta.tensor.reshape(broadcast_input_shape)
```
**EN:** Calls `node_meta.tensor.reshape` for side effects or initialization work in function `broadcast`.
**CN:** 调用 `node_meta.tensor.reshape` 执行副作用或初始化逻辑；该语句位于在函数 `broadcast` 中。

##### Line 188 — Comment or spacing block

```python
        # Update the input shape and output shape
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 189 — Assign `self.input_shape`

```python
        self.input_shape = _list_to_tuple(node_meta.tensor.shape)
```
**EN:** Assigns `self.input_shape` from `_list_to_tuple(node_meta.tensor.shape)`, establishing state in function `broadcast`.
**CN:** 将 `self.input_shape` 赋值为 `_list_to_tuple(node_meta.tensor.shape)`，用于在函数 `broadcast` 中建立状态。

##### Line 190 — Assign `self.output_shape`

```python
        self.output_shape = _list_to_tuple(shape)
```
**EN:** Assigns `self.output_shape` from `_list_to_tuple(shape)`, establishing state in function `broadcast`.
**CN:** 将 `self.output_shape` 赋值为 `_list_to_tuple(shape)`，用于在函数 `broadcast` 中建立状态。

#### Lines 192-195 — Function `apply_to_user`

```python
    def apply_to_user(self, user_meta: NodeBase):
        """
        Propagate the reshape to user nodes
        """
```
**EN:** Defines function `apply_to_user` with parameters `self, user_meta`. Purpose: Propagate the reshape to user nodes
**CN:** 定义函数 `apply_to_user`，参数为 `self, user_meta`。 其用途：Propagate the reshape to user nodes

##### Line 196 — Call `user_meta.tensor.reshape`

```python
        user_meta.tensor.reshape(tuple(self.input_shape))
```
**EN:** Calls `user_meta.tensor.reshape` for side effects or initialization work in function `apply_to_user`.
**CN:** 调用 `user_meta.tensor.reshape` 执行副作用或初始化逻辑；该语句位于在函数 `apply_to_user` 中。

##### Lines 197-199 — Conditional `hasattr(user_meta, 'store_tensor')`

```python
        if hasattr(user_meta, "store_tensor"):
            if user_meta.store_tensor is not None:
                user_meta.store_tensor.reshape(tuple(self.input_shape))
```
**EN:** Checks `hasattr(user_meta, 'store_tensor')` and selects the matching branch in function `apply_to_user`.
**CN:** 检查 `hasattr(user_meta, 'store_tensor')`，并在函数 `apply_to_user` 中选择匹配的分支。

#### Lines 201-204 — Function `apply_to_input`

```python
    def apply_to_input(self, input_meta: NodeBase):
        """
        Propagate the reshape to input nodes
        """
```
**EN:** Defines function `apply_to_input` with parameters `self, input_meta`. Purpose: Propagate the reshape to input nodes
**CN:** 定义函数 `apply_to_input`，参数为 `self, input_meta`。 其用途：Propagate the reshape to input nodes

##### Line 205 — Call `input_meta.tensor.reshape`

```python
        input_meta.tensor.reshape(tuple(self.output_shape))
```
**EN:** Calls `input_meta.tensor.reshape` for side effects or initialization work in function `apply_to_input`.
**CN:** 调用 `input_meta.tensor.reshape` 执行副作用或初始化逻辑；该语句位于在函数 `apply_to_input` 中。

##### Lines 206-208 — Conditional `hasattr(input_meta, 'store_tensor')`

```python
        if hasattr(input_meta, "store_tensor"):
            if input_meta.store_tensor is not None:
                input_meta.store_tensor.reshape(tuple(self.output_shape))
```
**EN:** Checks `hasattr(input_meta, 'store_tensor')` and selects the matching branch in function `apply_to_input`.
**CN:** 检查 `hasattr(input_meta, 'store_tensor')`，并在函数 `apply_to_input` 中选择匹配的分支。

#### Lines 210-212 — Comment or spacing block

```python
    #
    # Helper functions
    #
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Lines 214-217 — Function `infer_split`

```python
    def infer_split(self, input_shape, output_shape):
        """
        Infer the flatten splitted shape that can be merged to both input_shape and output_shape
        """
```
**EN:** Defines function `infer_split` with parameters `self, input_shape, output_shape`. Purpose: Infer the flatten splitted shape that can be merged to both input_shape and output_shape
**CN:** 定义函数 `infer_split`，参数为 `self, input_shape, output_shape`。 其用途：Infer the flatten splitted shape that can be merged to both input_shape and output_shape

##### Line 218 — Assign `input_shape`

```python
        input_shape = _tuple_to_list(input_shape)
```
**EN:** Assigns `input_shape` from `_tuple_to_list(input_shape)`, establishing state in function `infer_split`.
**CN:** 将 `input_shape` 赋值为 `_tuple_to_list(input_shape)`，用于在函数 `infer_split` 中建立状态。

##### Line 219 — Assign `output_shape`

```python
        output_shape = _tuple_to_list(output_shape)
```
**EN:** Assigns `output_shape` from `_tuple_to_list(output_shape)`, establishing state in function `infer_split`.
**CN:** 将 `output_shape` 赋值为 `_tuple_to_list(output_shape)`，用于在函数 `infer_split` 中建立状态。

##### Lines 220-221 — Conditional `len(input_shape) == 0 and len(output_shape) == 0`

```python
        if len(input_shape) == 0 and len(output_shape) == 0:
            return []
```
**EN:** Checks `len(input_shape) == 0 and len(output_shape) == 0` and selects the matching branch in function `infer_split`.
**CN:** 检查 `len(input_shape) == 0 and len(output_shape) == 0`，并在函数 `infer_split` 中选择匹配的分支。

##### Lines 222-226 — Conditional `len(input_shape) == 0`

```python
        if len(input_shape) == 0:
            if product(tuple(output_shape)) != 1:
                raise ValueError("Invalid reshape size")
            else:
                return output_shape
```
**EN:** Checks `len(input_shape) == 0` and selects the matching branch in function `infer_split`.
**CN:** 检查 `len(input_shape) == 0`，并在函数 `infer_split` 中选择匹配的分支。

##### Lines 227-231 — Conditional `len(output_shape) == 0`

```python
        if len(output_shape) == 0:
            if product(tuple(input_shape)) != 1:
                raise ValueError("Invalid reshape size")
            else:
                return input_shape
```
**EN:** Checks `len(output_shape) == 0` and selects the matching branch in function `infer_split`.
**CN:** 检查 `len(output_shape) == 0`，并在函数 `infer_split` 中选择匹配的分支。

##### Line 232 — Comment or spacing block

```python
        # This is done recursively by only process the last dimension at each time
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Line 233 — Assign `old_dim`

```python
        old_dim = input_shape[-1]
```
**EN:** Assigns `old_dim` from `input_shape[-1]`, establishing state in function `infer_split`.
**CN:** 将 `old_dim` 赋值为 `input_shape[-1]`，用于在函数 `infer_split` 中建立状态。

##### Line 234 — Assign `new_dim`

```python
        new_dim = output_shape[-1]
```
**EN:** Assigns `new_dim` from `output_shape[-1]`, establishing state in function `infer_split`.
**CN:** 将 `new_dim` 赋值为 `output_shape[-1]`，用于在函数 `infer_split` 中建立状态。

##### Line 235 — Comment or spacing block

```python
        # Exact match
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 236-237 — Conditional `old_dim == new_dim`

```python
        if old_dim == new_dim:
            return self.infer_split(input_shape[:-1], output_shape[:-1]) + [new_dim,]
```
**EN:** Checks `old_dim == new_dim` and selects the matching branch in function `infer_split`.
**CN:** 检查 `old_dim == new_dim`，并在函数 `infer_split` 中选择匹配的分支。

##### Line 238 — Comment or spacing block

```python
        # Needs split
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 239-241 — Conditional `old_dim > new_dim and old_dim % new_dim == 0`

```python
        if old_dim > new_dim and old_dim % new_dim == 0:
            residual = old_dim // new_dim
            return self.infer_split(input_shape[:-1] + [residual,], output_shape[:-1]) + [new_dim,]
```
**EN:** Checks `old_dim > new_dim and old_dim % new_dim == 0` and selects the matching branch in function `infer_split`.
**CN:** 检查 `old_dim > new_dim and old_dim % new_dim == 0`，并在函数 `infer_split` 中选择匹配的分支。

##### Line 242 — Comment or spacing block

```python
        # Needs merge
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

##### Lines 243-245 — Conditional `old_dim < new_dim and new_dim % old_dim == 0`

```python
        if old_dim < new_dim and new_dim % old_dim == 0:
            residual = new_dim // old_dim
            return self.infer_split(input_shape[:-1], output_shape[:-1] + [residual,]) + [old_dim,]
```
**EN:** Checks `old_dim < new_dim and new_dim % old_dim == 0` and selects the matching branch in function `infer_split`.
**CN:** 检查 `old_dim < new_dim and new_dim % old_dim == 0`，并在函数 `infer_split` 中选择匹配的分支。

##### Line 247 — Raise exception

```python
        raise NotImplementedError(f"Unsupported split: {input_shape} -> {output_shape}")
```
**EN:** Raises `NotImplementedError(f'Unsupported split: {input_shape} -> {output_shape}')` to signal an error or unsupported state.
**CN:** 抛出 `NotImplementedError(f'Unsupported split: {input_shape} -> {output_shape}')`，用于报告错误或不支持的状态。

#### Line 249 — Function `infer_merge`

```python
    def infer_merge(self, flatten_shape, shape):
```
**EN:** Defines function `infer_merge` with parameters `self, flatten_shape, shape`.
**CN:** 定义函数 `infer_merge`，参数为 `self, flatten_shape, shape`。

##### Line 250 — Assign `flatten_shape`

```python
        flatten_shape = _tuple_to_list(flatten_shape)
```
**EN:** Assigns `flatten_shape` from `_tuple_to_list(flatten_shape)`, establishing state in function `infer_merge`.
**CN:** 将 `flatten_shape` 赋值为 `_tuple_to_list(flatten_shape)`，用于在函数 `infer_merge` 中建立状态。

##### Line 251 — Assign `shape`

```python
        shape = _tuple_to_list(shape)
```
**EN:** Assigns `shape` from `_tuple_to_list(shape)`, establishing state in function `infer_merge`.
**CN:** 将 `shape` 赋值为 `_tuple_to_list(shape)`，用于在函数 `infer_merge` 中建立状态。

##### Line 252 — Assign `idx_flat`

```python
        idx_flat = len(flatten_shape) - 1
```
**EN:** Assigns `idx_flat` from `len(flatten_shape) - 1`, establishing state in function `infer_merge`.
**CN:** 将 `idx_flat` 赋值为 `len(flatten_shape) - 1`，用于在函数 `infer_merge` 中建立状态。

##### Line 253 — Assign `merged_shape`

```python
        merged_shape = []
```
**EN:** Assigns `merged_shape` from `[]`, establishing state in function `infer_merge`.
**CN:** 将 `merged_shape` 赋值为 `[]`，用于在函数 `infer_merge` 中建立状态。

##### Lines 254-269 — Loop over `reversed(shape)`

```python
        for dim in reversed(shape):
            # Exact match
            if dim == flatten_shape[idx_flat]:
                merged_shape.append(dim)
                idx_flat -= 1
            # need group
            elif dim > flatten_shape[idx_flat] and dim % flatten_shape[idx_flat] == 0:
                residual = dim
                group = []
                while(residual > 1):
                    group.append(flatten_shape[idx_flat])
                    residual = residual // flatten_shape[idx_flat]
                    idx_flat -= 1
                merged_shape.append(group[::-1])
            else:
                raise NotImplementedError(f"Unsupported merge: {flatten_shape} -> {shape}")
```
**EN:** Iterates `dim` over `reversed(shape)` to repeat a processing step.
**CN:** 让 `dim` 遍历 `reversed(shape)`，从而重复执行处理步骤。

##### Line 271 — Return

```python
        return merged_shape[::-1]
```
**EN:** Returns `merged_shape[::-1]` to the caller.
**CN:** 向调用方返回 `merged_shape[::-1]`。

### Lines 274-277 — Class `LayoutNode`

```python
class LayoutNode(NodeBase):
    """
    Layout manipulation nodes
    """
```
**EN:** Declares class `LayoutNode` deriving from `NodeBase`. Purpose: Layout manipulation nodes
**CN:** 声明类 `LayoutNode`，其基类为 `NodeBase`。 其用途：Layout manipulation nodes

#### Lines 278-281 — Assign `fn_to_impl`

```python
    fn_to_impl = {
        "permute": PermutationImpl,
        "reshape": ReshapeImpl
    }
```
**EN:** Assigns `fn_to_impl` from `{'permute': PermutationImpl, 'reshape': ReshapeImpl}`, establishing state in class `LayoutNode`.
**CN:** 将 `fn_to_impl` 赋值为 `{'permute': PermutationImpl, 'reshape': ReshapeImpl}`，用于在类 `LayoutNode` 中建立状态。

#### Line 282 — Function `__init__`

```python
    def __init__(self, name: str, fn, kwargs: dict) -> None:
```
**EN:** Defines function `__init__` with parameters `self, name, fn, kwargs`.
**CN:** 定义函数 `__init__`，参数为 `self, name, fn, kwargs`。

##### Line 283 — Call `super().__init__`

```python
        super().__init__(name)
```
**EN:** Calls `super().__init__` for side effects or initialization work in function `__init__`.
**CN:** 调用 `super().__init__` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

##### Line 284 — Assign `self.op`

```python
        self.op = "layout"
```
**EN:** Assigns `self.op` from `'layout'`, establishing state in function `__init__`.
**CN:** 将 `self.op` 赋值为 `'layout'`，用于在函数 `__init__` 中建立状态。

##### Line 285 — Assign `self.fn`

```python
        self.fn = fn
```
**EN:** Assigns `self.fn` from `fn`, establishing state in function `__init__`.
**CN:** 将 `self.fn` 赋值为 `fn`，用于在函数 `__init__` 中建立状态。

##### Line 286 — Assign `self.kwargs`

```python
        self.kwargs = kwargs
```
**EN:** Assigns `self.kwargs` from `kwargs`, establishing state in function `__init__`.
**CN:** 将 `self.kwargs` 赋值为 `kwargs`，用于在函数 `__init__` 中建立状态。

##### Line 287 — Assign `self.underlying_impl`

```python
        self.underlying_impl = self.fn_to_impl[self.fn.__name__](self)
```
**EN:** Assigns `self.underlying_impl` from `self.fn_to_impl[self.fn.__name__](self)`, establishing state in function `__init__`.
**CN:** 将 `self.underlying_impl` 赋值为 `self.fn_to_impl[self.fn.__name__](self)`，用于在函数 `__init__` 中建立状态。

#### Line 289 — Function `get_inverse_node`

```python
    def get_inverse_node(self):
```
**EN:** Defines function `get_inverse_node` with parameters `self`.
**CN:** 定义函数 `get_inverse_node`，参数为 `self`。

##### Line 290 — Assign `inverse_node`

```python
        inverse_node = deepcopy(self)
```
**EN:** Assigns `inverse_node` from `deepcopy(self)`, establishing state in function `get_inverse_node`.
**CN:** 将 `inverse_node` 赋值为 `deepcopy(self)`，用于在函数 `get_inverse_node` 中建立状态。

##### Line 291 — Assign `inverse_node.underlying_impl`

```python
        inverse_node.underlying_impl = self.underlying_impl.get_inverse_impl()
```
**EN:** Assigns `inverse_node.underlying_impl` from `self.underlying_impl.get_inverse_impl()`, establishing state in function `get_inverse_node`.
**CN:** 将 `inverse_node.underlying_impl` 赋值为 `self.underlying_impl.get_inverse_impl()`，用于在函数 `get_inverse_node` 中建立状态。

##### Line 292 — Return

```python
        return inverse_node
```
**EN:** Returns `inverse_node` to the caller.
**CN:** 向调用方返回 `inverse_node`。

#### Line 294 — Function `shape_propagation`

```python
    def shape_propagation(self, input_node_metas):
```
**EN:** Defines function `shape_propagation` with parameters `self, input_node_metas`.
**CN:** 定义函数 `shape_propagation`，参数为 `self, input_node_metas`。

##### Lines 295-296 — Conditional `self._tensor is not None`

```python
        if self._tensor is not None:
            return
```
**EN:** Checks `self._tensor is not None` and selects the matching branch in function `shape_propagation`.
**CN:** 检查 `self._tensor is not None`，并在函数 `shape_propagation` 中选择匹配的分支。

##### Line 297 — Assertion

```python
        assert len(input_node_metas) == 1, "Layout node can only have one input node"
```
**EN:** Asserts `len(input_node_metas) == 1` to enforce an expected condition.
**CN:** 断言 `len(input_node_metas) == 1`，用于保证预期条件成立。

##### Line 299 — Assign `output_shape`

```python
        output_shape = self.underlying_impl.shape_propagation(input_node_metas[0])
```
**EN:** Assigns `output_shape` from `self.underlying_impl.shape_propagation(input_node_metas[0])`, establishing state in function `shape_propagation`.
**CN:** 将 `output_shape` 赋值为 `self.underlying_impl.shape_propagation(input_node_metas[0])`，用于在函数 `shape_propagation` 中建立状态。

##### Lines 301-304 — Assign `self._tensor`

```python
        self._tensor = Tensor(
            element=self.element_output,
            shape=output_shape, layout_tag=LayoutType.RowMajor
        )
```
**EN:** Assigns `self._tensor` from `Tensor(element=self.element_output, shape=output_shape, layout_tag=LayoutType.RowMajor)`, establishing state in function `shape_propagation`.
**CN:** 将 `self._tensor` 赋值为 `Tensor(element=self.element_output, shape=output_shape, layout_tag=LayoutType.RowMajor)`，用于在函数 `shape_propagation` 中建立状态。

##### Line 306 — Return

```python
        return super().shape_propagation(input_node_metas)
```
**EN:** Returns `super().shape_propagation(input_node_metas)` to the caller.
**CN:** 向调用方返回 `super().shape_propagation(input_node_metas)`。

#### Lines 308-311 — Function `type_propagation`

```python
    def type_propagation(self, input_node_metas: 'list[NodeBase]'):
        """
        The store nodes has element_output = element_input
        """
```
**EN:** Defines function `type_propagation` with parameters `self, input_node_metas`. Purpose: The store nodes has element_output = element_input
**CN:** 定义函数 `type_propagation`，参数为 `self, input_node_metas`。 其用途：The store nodes has element_output = element_input

##### Line 312 — Assertion

```python
        assert len(input_node_metas) == 1, "Layout node can only have one input node"
```
**EN:** Asserts `len(input_node_metas) == 1` to enforce an expected condition.
**CN:** 断言 `len(input_node_metas) == 1`，用于保证预期条件成立。

##### Line 313 — Assign `self.element_output`

```python
        self.element_output = input_node_metas[0].element_output
```
**EN:** Assigns `self.element_output` from `input_node_metas[0].element_output`, establishing state in function `type_propagation`.
**CN:** 将 `self.element_output` 赋值为 `input_node_metas[0].element_output`，用于在函数 `type_propagation` 中建立状态。

#### Lines 315-318 — Function `broadcast_propagation`

```python
    def broadcast_propagation(self, input_node_metas: 'list[NodeBase]'):
        """
        Propagate the broadcast in the reversed topological order
        """
```
**EN:** Defines function `broadcast_propagation` with parameters `self, input_node_metas`. Purpose: Propagate the broadcast in the reversed topological order
**CN:** 定义函数 `broadcast_propagation`，参数为 `self, input_node_metas`。 其用途：Propagate the broadcast in the reversed topological order

##### Lines 319-320 — Conditional `self.tensor is None`

```python
        if self.tensor is None:
            raise RuntimeError(f"The tensor of node {self.name} is unknown.")
```
**EN:** Checks `self.tensor is None` and selects the matching branch in function `broadcast_propagation`.
**CN:** 检查 `self.tensor is None`，并在函数 `broadcast_propagation` 中选择匹配的分支。

##### Line 321 — Assign `shape`

```python
        shape = self.tensor.shape
```
**EN:** Assigns `shape` from `self.tensor.shape`, establishing state in function `broadcast_propagation`.
**CN:** 将 `shape` 赋值为 `self.tensor.shape`，用于在函数 `broadcast_propagation` 中建立状态。

##### Lines 323-324 — Loop over `input_node_metas`

```python
        for child in input_node_metas:
            self.underlying_impl.broadcast(shape, child)
```
**EN:** Iterates `child` over `input_node_metas` to repeat a processing step.
**CN:** 让 `child` 遍历 `input_node_metas`，从而重复执行处理步骤。

#### Lines 326-329 — Function `apply_to_user`

```python
    def apply_to_user(self, usr_meta: NodeBase):
        """
        Propagate the permutation to user nodes
        """
```
**EN:** Defines function `apply_to_user` with parameters `self, usr_meta`. Purpose: Propagate the permutation to user nodes
**CN:** 定义函数 `apply_to_user`，参数为 `self, usr_meta`。 其用途：Propagate the permutation to user nodes

##### Line 330 — Call `self.underlying_impl.apply_to_user`

```python
        self.underlying_impl.apply_to_user(usr_meta)
```
**EN:** Calls `self.underlying_impl.apply_to_user` for side effects or initialization work in function `apply_to_user`.
**CN:** 调用 `self.underlying_impl.apply_to_user` 执行副作用或初始化逻辑；该语句位于在函数 `apply_to_user` 中。

#### Lines 332-335 — Function `apply_to_input`

```python
    def apply_to_input(self, input_meta: NodeBase):
        """
        Propagate the permutation to input nodes
        """
```
**EN:** Defines function `apply_to_input` with parameters `self, input_meta`. Purpose: Propagate the permutation to input nodes
**CN:** 定义函数 `apply_to_input`，参数为 `self, input_meta`。 其用途：Propagate the permutation to input nodes

##### Line 336 — Call `self.underlying_impl.apply_to_input`

```python
        self.underlying_impl.apply_to_input(input_meta)
```
**EN:** Calls `self.underlying_impl.apply_to_input` for side effects or initialization work in function `apply_to_input`.
**CN:** 调用 `self.underlying_impl.apply_to_input` 执行副作用或初始化逻辑；该语句位于在函数 `apply_to_input` 中。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `PermutationImpl`, `ReshapeImpl`, `LayoutNode`.
- **CN:** 顶层类：`PermutationImpl`, `ReshapeImpl`, `LayoutNode`。
- **EN:** Top-level functions: none.
- **CN:** 顶层函数：无。
- **EN:** This module participates in the EVT pipeline, covering frontend parsing, IR construction, optimization passes, or backend emission.
- **CN:** 该模块属于 EVT 流水线的一部分，涉及前端解析、中间表示构建、优化 pass 或后端代码生成。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen`, `cutlass_cppgen.backend.evt.ir.layout_algorithm`, `cutlass_cppgen.backend.evt.ir.node`, `cutlass_cppgen.backend.evt.ir.tensor`, `cutlass_library`
- **Standard & third-party / 标准库与第三方:** `copy`, `pycute`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
