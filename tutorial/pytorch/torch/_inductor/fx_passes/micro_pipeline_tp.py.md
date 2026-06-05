# micro_pipeline_tp.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/micro_pipeline_tp.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `_AllGatherMatch`, `_ReduceScatterMatch`, `_Matmul`, and `_ScaledMatmul`. It exposes functions such as `_is_last_dim`, `_is_backward`, `_compute_mm_arithmetic_intensity`, `_filter_nodes_by_target`, `_find_ancestors`, `_get_tensor`, and `...+14`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `_AllGatherMatch`、`_ReduceScatterMatch`、`_Matmul`、`_ScaledMatmul` 等类。同时提供 `_is_last_dim`、`_is_backward`、`_compute_mm_arithmetic_intensity`、`_filter_nodes_by_target`、`_find_ancestors`、`_get_tensor`、`另有14项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
import logging
import operator
from collections import defaultdict
from dataclasses import dataclass, field
from math import prod
from typing import Any, cast

import torch
from torch.utils._ordered_set import OrderedSet

from .. import config, inductor_prims
from ..pattern_matcher import (
    CallFunction,
    Ignored,
    KeywordArg,
    ListOf,
    Match,
    MULTIPLE,
    PatternExpr,
    PatternMatcherPass,
)


log = logging.getLogger(__name__)
aten = torch.ops.aten
patterns = PatternMatcherPass()

````
- **EN**: Imports dependencies such as `logging`, `operator`, `collections`, `dataclasses`, `math`, `typing`, and `...+4` for the logic in this range. Initializes or updates values such as `log`, `aten`, and `patterns`.
- **CN**: 这里导入了 `logging`、`operator`、`collections`、`dataclasses`、`math`、`typing`、`另有4项` 等依赖，为后续逻辑提供基础能力。初始化或更新了 `log`、`aten`、`patterns` 等值。

### Lines 29-56 / 第 29-56 行
````python

def _is_last_dim(t: torch.Tensor, dim: int) -> bool:
    return dim == t.ndim - 1 or dim == -1


def _is_backward(graph: torch.fx.Graph) -> bool:
    placeholders = []
    for node in graph.nodes:
        if node.op != "placeholder":
            break
        placeholders.append(node)
    return not all(node.name.startswith("primal") for node in placeholders)


def _compute_mm_arithmetic_intensity(M: int, N: int, K: int) -> float:
    return M * N * K / (M * K + N * K + M * N)


def _filter_nodes_by_target(nodes: list[torch.fx.Node], target) -> list[torch.fx.Node]:
    return [x for x in nodes if x.target == target]


def _find_ancestors(node: torch.fx.Node) -> OrderedSet[torch.fx.Node]:
    ancestors = OrderedSet[torch.fx.Node]()
    ancestors.add(node)
    cur_nodes = [node]
    while len(cur_nodes) > 0:
        new_nodes = []
````
- **EN**: Introduces function `_is_last_dim`, function `_is_backward`, function `_compute_mm_arithmetic_intensity`, function `_filter_nodes_by_target`, function `_find_ancestors`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `placeholders`, `ancestors`, `cur_nodes`, and `new_nodes`.
- **CN**: 这里定义了函数`_is_last_dim`、函数`_is_backward`、函数`_compute_mm_arithmetic_intensity`、函数`_filter_nodes_by_target`、函数`_find_ancestors`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `placeholders`、`ancestors`、`cur_nodes`、`new_nodes` 等值。

### Lines 57-84 / 第 57-84 行
````python
        for node in cur_nodes:
            for inp in node.all_input_nodes:
                if inp not in ancestors:
                    ancestors.add(inp)
                    new_nodes.append(inp)
        cur_nodes = new_nodes
    return OrderedSet(node for node in ancestors if node.op != "placeholder")


def _get_tensor(node: torch.fx.Node) -> torch.Tensor:
    val = node.meta["val"]
    assert isinstance(val, torch.Tensor)
    return val


@dataclass
class _AllGatherMatch:
    match: Match
    shard_node: torch.fx.Node
    ag_node: torch.fx.Node
    res_node: torch.fx.Node
    gather_dim: int
    group_name: "torch.distributed.distributed_c10d.GroupName"

    def replace_with(self, new_node: torch.fx.Node) -> None:
        self.res_node.replace_all_uses_with(new_node)

    def erase(self) -> None:
````
- **EN**: Introduces function `_get_tensor`, class `_AllGatherMatch`, function `replace_with`, function `erase`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_get_tensor`、类`_AllGatherMatch`、函数`replace_with`、函数`erase`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 85-112 / 第 85-112 行
````python
        for node in reversed(self.match.nodes):
            if len(node.users) == 0:
                node.graph.erase_node(node)


def find_all_gather_patterns(graph: torch.fx.Graph):
    c10d = torch.ops._c10d_functional

    def make_zero_dim_all_gather_pattern(shard):
        return CallFunction(
            c10d.wait_tensor.default,
            CallFunction(
                c10d.all_gather_into_tensor.default,
                shard,
                Ignored(),
                KeywordArg("group_name"),
            ),
        )

    # Matches funcol.all_gather_tensor with gather_dim == 0
    zero_dim_all_gather_pattern = make_zero_dim_all_gather_pattern(KeywordArg("shard"))

    def make_all_gather_split_pattern(shard):
        return CallFunction(
            operator.getitem,
            CallFunction(
                aten.split.Tensor,
                make_zero_dim_all_gather_pattern(shard),
````
- **EN**: Introduces function `find_all_gather_patterns`, function `make_zero_dim_all_gather_pattern`, function `make_all_gather_split_pattern`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `c10d`, and `zero_dim_all_gather_pattern`.
- **CN**: 这里定义了函数`find_all_gather_patterns`、函数`make_zero_dim_all_gather_pattern`、函数`make_all_gather_split_pattern`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `c10d`、`zero_dim_all_gather_pattern` 等值。

### Lines 113-140 / 第 113-140 行
````python
                Ignored(),
                _users=MULTIPLE,
            ),
            Ignored(),
        )

    def make_cat_pattern(splits):
        return CallFunction(
            aten.cat.default,
            ListOf(splits),
            KeywordArg("gather_dim"),
        )

    # Matches funcol.all_gather_tensor with gather_dim > 0
    non_zero_dim_all_gather_pattern = make_cat_pattern(
        make_all_gather_split_pattern(KeywordArg("shard")),
    )

    # Match a zero-dim all-gather in which the data is transferred as uint8 and
    # viewed back as the original dtype.
    zero_dim_type_erased_all_gather_pattern = CallFunction(
        aten.view.dtype,
        make_zero_dim_all_gather_pattern(
            KeywordArg("shard"),
        ),
        Ignored(),
    )

````
- **EN**: Introduces function `make_cat_pattern`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `_users`, `non_zero_dim_all_gather_pattern`, and `zero_dim_type_erased_all_gather_pattern`.
- **CN**: 这里定义了函数`make_cat_pattern`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `_users`、`non_zero_dim_all_gather_pattern`、`zero_dim_type_erased_all_gather_pattern` 等值。

### Lines 141-168 / 第 141-168 行
````python
    # Match a non-zero dim all-gather in which the data is transferred as uint8
    # and viewed back as the original dtype.
    non_zero_dim_type_erased_all_gather_pattern = CallFunction(
        aten.view.dtype,
        make_cat_pattern(
            CallFunction(
                aten.view.dtype,
                make_all_gather_split_pattern(
                    KeywordArg("shard"),
                ),
                Ignored(),
            ),
        ),
        Ignored(),
    )

    # If two patterns with the same res_node_target have the same suffix, the
    # longer pattern should appear first in the list.
    # e.g. supposed we have (1) A -> B -> C -> D and (2) B -> C -> D, (1)
    # should appear before (2) in the list.
    res_node_target_to_patterns = {
        aten.cat.default: [
            (non_zero_dim_all_gather_pattern, 0),
        ],
        aten.view.dtype: [
            (non_zero_dim_type_erased_all_gather_pattern, 0),
            (zero_dim_type_erased_all_gather_pattern, 0),
        ],
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `non_zero_dim_type_erased_all_gather_pattern`, and `res_node_target_to_patterns`. This range continues the implementation of function `find_all_gather_patterns`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `non_zero_dim_type_erased_all_gather_pattern`、`res_node_target_to_patterns` 等值。这一段延续了函数`find_all_gather_patterns` 的具体实现。

### Lines 169-196 / 第 169-196 行
````python
        c10d.wait_tensor.default: [
            (zero_dim_all_gather_pattern, 0),
        ],
    }

    # Match in reverse to ensure longer patterns is prioritized
    all_gathers = []
    visited_ag_nodes = OrderedSet[torch.fx.Node]()
    for node in reversed(graph.nodes):
        for target, patterns in res_node_target_to_patterns.items():
            if node.target != target:
                continue
            for pattern, ag_node_idx in patterns:
                match = pattern.match(node)
                if not match:
                    continue

                assert isinstance(match, Match)
                ag_node = match.nodes[ag_node_idx]
                assert ag_node.target == c10d.all_gather_into_tensor.default

                if ag_node in visited_ag_nodes:
                    continue
                visited_ag_nodes.add(ag_node)

                ag_match = _AllGatherMatch(
                    match=match,
                    shard_node=match.kwargs["shard"],
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `all_gathers`, `visited_ag_nodes`, `match`, `ag_node`, `ag_match`, and `shard_node`. This range continues the implementation of function `find_all_gather_patterns`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `all_gathers`、`visited_ag_nodes`、`match`、`ag_node`、`ag_match`、`shard_node` 等值。这一段延续了函数`find_all_gather_patterns` 的具体实现。

### Lines 197-224 / 第 197-224 行
````python
                    ag_node=ag_node,
                    res_node=node,
                    gather_dim=match.kwargs.get("gather_dim", 0),
                    group_name=match.kwargs["group_name"],
                )
                all_gathers.append(ag_match)

    return list(reversed(all_gathers))


@dataclass
class _ReduceScatterMatch:
    match: Match
    input_node: torch.fx.Node
    reduce_scatter_node: torch.fx.Node
    wait_tensor_node: torch.fx.Node
    reduce_op: str
    scatter_dim: int
    group_name: "torch.distributed.distributed_c10d.GroupName"

    def replace_with(self, new_node: torch.fx.Node) -> None:
        # Replace all uses of the result node (wait_tensor) with the fused node.
        self.wait_tensor_node.replace_all_uses_with(new_node)

        # If the reduce-scatter result is saved for backward, save the fused node for backward instead.
        self._update_save_for_backward(new_node)

    def _update_save_for_backward(self, new_node: torch.fx.Node) -> None:
````
- **EN**: Introduces class `_ReduceScatterMatch`, function `replace_with`, function `_update_save_for_backward`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`_ReduceScatterMatch`、函数`replace_with`、函数`_update_save_for_backward`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 225-252 / 第 225-252 行
````python
        """
        If the output node is a user of the reduce_scatter node (indicating the reduce_scatter
        result is saved for backward), this method will update the output node to use the fused node instead.
        """
        output_node = None
        for user in self.reduce_scatter_node.users:
            if user.target == "output":
                output_node = user
                break
        if output_node is not None:
            output_node.replace_input_with(self.reduce_scatter_node, new_node)

            # Assert that now the reduce scatter node has only one user (the wait_tensor) and it's not
            # saved for backward anymore.
            assert len(self.reduce_scatter_node.users) == 1, (
                "Reduce scatter node has multiple users, this is not expected"
            )

    def erase(self) -> None:
        for node in reversed(self.match.nodes):
            if len(node.users) == 0:
                node.graph.erase_node(node)


def find_reduce_scatter_patterns(graph: torch.fx.Graph):
    c10d = torch.ops._c10d_functional

    def reduce_scatter_template(inp: PatternExpr, users: int):
````
- **EN**: Introduces function `erase`, function `find_reduce_scatter_patterns`, function `reduce_scatter_template`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `output_node`, and `c10d`.
- **CN**: 这里定义了函数`erase`、函数`find_reduce_scatter_patterns`、函数`reduce_scatter_template`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `output_node`、`c10d` 等值。

### Lines 253-280 / 第 253-280 行
````python
        return CallFunction(
            c10d.wait_tensor.default,
            CallFunction(
                c10d.reduce_scatter_tensor.default,
                inp,
                KeywordArg("reduce_op"),
                Ignored(),
                KeywordArg("group_name"),
                _users=users,
            ),
        )

    # Matches funcol.reduce_scatter_tensor with scatter_dim == 0
    zero_dim_reduce_scatter_pattern_single_user = reduce_scatter_template(
        KeywordArg("input"), users=1
    )

    # Two users will occur when the reduce-scatter result is saved for backward
    zero_dim_reduce_scatter_pattern_multi_user = reduce_scatter_template(
        KeywordArg("input"), users=2
    )

    # Matches funcol.reduce_scatter_tensor with scatter_dim > 0
    non_zero_dim_reduce_scatter_pattern_single_user = reduce_scatter_template(
        CallFunction(
            aten.cat.default,
            ListOf(
                CallFunction(
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `_users`, `zero_dim_reduce_scatter_pattern_single_user`, `zero_dim_reduce_scatter_pattern_multi_user`, and `non_zero_dim_reduce_scatter_pattern_single_user`. This range continues the implementation of function `find_reduce_scatter_patterns`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `_users`、`zero_dim_reduce_scatter_pattern_single_user`、`zero_dim_reduce_scatter_pattern_multi_user`、`non_zero_dim_reduce_scatter_pattern_single_user` 等值。这一段延续了函数`find_reduce_scatter_patterns` 的具体实现。

### Lines 281-308 / 第 281-308 行
````python
                    operator.getitem,
                    CallFunction(
                        aten.split.Tensor,
                        KeywordArg("input"),
                        Ignored(),
                        KeywordArg("scatter_dim"),
                        _users=MULTIPLE,
                    ),
                    Ignored(),
                )
            ),
        ),
        users=1,
    )

    # Two users will occur when the reduce-scatter result is saved for backward
    non_zero_dim_reduce_scatter_pattern_multi_user = reduce_scatter_template(
        CallFunction(
            aten.cat.default,
            ListOf(
                CallFunction(
                    operator.getitem,
                    CallFunction(
                        aten.split.Tensor,
                        KeywordArg("input"),
                        Ignored(),
                        KeywordArg("scatter_dim"),
                        _users=MULTIPLE,
````
- **EN**: Initializes or updates values such as `_users`, `users`, and `non_zero_dim_reduce_scatter_pattern_multi_user`. This range continues the implementation of function `find_reduce_scatter_patterns`.
- **CN**: 初始化或更新了 `_users`、`users`、`non_zero_dim_reduce_scatter_pattern_multi_user` 等值。这一段延续了函数`find_reduce_scatter_patterns` 的具体实现。

### Lines 309-336 / 第 309-336 行
````python
                    ),
                    Ignored(),
                )
            ),
        ),
        users=2,
    )

    reduce_scatters = []
    for node in reversed(graph.nodes):
        if node.target == c10d.wait_tensor.default:
            if match := non_zero_dim_reduce_scatter_pattern_single_user.match(node):
                assert isinstance(match, Match)
                reduce_scatters.append(
                    _ReduceScatterMatch(
                        match=match,
                        input_node=match.kwargs["input"],
                        reduce_scatter_node=match.nodes[-2],
                        wait_tensor_node=node,
                        reduce_op=match.kwargs["reduce_op"],
                        scatter_dim=match.kwargs["scatter_dim"],
                        group_name=match.kwargs["group_name"],
                    )
                )
            elif match := zero_dim_reduce_scatter_pattern_single_user.match(node):
                assert isinstance(match, Match)
                reduce_scatters.append(
                    _ReduceScatterMatch(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `users`, `reduce_scatters`, `match`, `input_node`, `reduce_scatter_node`, `wait_tensor_node`, and `...+3`. This range continues the implementation of function `find_reduce_scatter_patterns`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `users`、`reduce_scatters`、`match`、`input_node`、`reduce_scatter_node`、`wait_tensor_node`、`另有3项` 等值。这一段延续了函数`find_reduce_scatter_patterns` 的具体实现。

### Lines 337-364 / 第 337-364 行
````python
                        match=match,
                        input_node=match.kwargs["input"],
                        reduce_scatter_node=match.nodes[0],
                        wait_tensor_node=node,
                        reduce_op=match.kwargs["reduce_op"],
                        scatter_dim=0,
                        group_name=match.kwargs["group_name"],
                    )
                )
            elif match := non_zero_dim_reduce_scatter_pattern_multi_user.match(node):
                assert isinstance(match, Match)
                reduce_scatters.append(
                    _ReduceScatterMatch(
                        match=match,
                        input_node=match.kwargs["input"],
                        reduce_scatter_node=match.nodes[-2],
                        wait_tensor_node=node,
                        reduce_op=match.kwargs["reduce_op"],
                        scatter_dim=match.kwargs["scatter_dim"],
                        group_name=match.kwargs["group_name"],
                    )
                )
            elif match := zero_dim_reduce_scatter_pattern_multi_user.match(node):
                assert isinstance(match, Match)
                reduce_scatters.append(
                    _ReduceScatterMatch(
                        match=match,
                        input_node=match.kwargs["input"],
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `match`, `input_node`, `reduce_scatter_node`, `wait_tensor_node`, `reduce_op`, `scatter_dim`, and `...+1`. This range continues the implementation of function `find_reduce_scatter_patterns`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `match`、`input_node`、`reduce_scatter_node`、`wait_tensor_node`、`reduce_op`、`scatter_dim`、`另有1项` 等值。这一段延续了函数`find_reduce_scatter_patterns` 的具体实现。

### Lines 365-392 / 第 365-392 行
````python
                        reduce_scatter_node=match.nodes[0],
                        wait_tensor_node=node,
                        reduce_op=match.kwargs["reduce_op"],
                        scatter_dim=0,
                        group_name=match.kwargs["group_name"],
                    )
                )
    return list(reversed(reduce_scatters))


@dataclass
class _Matmul:
    nodes: list[torch.fx.Node]
    arg_ancestor_nodes: OrderedSet[torch.fx.Node] = field(init=False)
    A_node: torch.fx.Node
    B_node: torch.fx.Node
    pre_mm_reshape: torch.fx.Node | None
    post_mm_reshape: torch.fx.Node | None

    def __post_init__(self):
        assert len(self.nodes) in (1, 3)
        if len(self.nodes) == 1:
            assert self.nodes[0].target in (aten.mm.default, aten._scaled_mm.default)
        else:
            assert self.nodes[0].target is aten.reshape.default
            assert self.nodes[1].target in (aten.mm.default, aten._scaled_mm.default)
            assert self.nodes[2].target is aten.reshape.default
        self.arg_ancestor_nodes = _find_ancestors(self.B_node)
````
- **EN**: Introduces class `_Matmul`, function `__post_init__`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`_Matmul`、函数`__post_init__`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 393-420 / 第 393-420 行
````python

    def replace_with(self, new_node: torch.fx.Node) -> None:
        """
        Replace the matmul with the new node.
        """
        graph = new_node.graph

        # For 2D-matmuls, we simply replace the mm node with `new_node`.
        if len(self.nodes) == 1:
            mm_node = self.nodes[0]
            assert mm_node.target in (aten.mm.default, aten._scaled_mm.default)
            mm_node.replace_all_uses_with(new_node)
            graph.erase_node(mm_node)
            return

        # An ND-matmul is reshape -> mm -> reshape sequence. We first replace
        # the second reshape node with `new_node`. Then, we ensure that the
        # original mm node in the sequence ends up with zero users by replacing
        # it with a reverse reshape of `new_node`.
        graph = new_node.graph
        assert len(self.nodes) == 3
        mm_node = self.nodes[1]
        output_reshape_node = self.nodes[2]

        assert mm_node.target in (aten.mm.default, aten._scaled_mm.default)
        assert output_reshape_node.target is aten.reshape.default

        output_reshape_node.replace_all_uses_with(new_node)
````
- **EN**: Introduces function `replace_with`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph`, `mm_node`, and `output_reshape_node`.
- **CN**: 这里定义了函数`replace_with`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph`、`mm_node`、`output_reshape_node` 等值。

### Lines 421-448 / 第 421-448 行
````python
        if len(mm_node.users) > 1:
            with graph.inserting_after(new_node):
                new_mm_node = graph.call_function(
                    aten.reshape.default,
                    args=(new_node, list(_get_tensor(mm_node).shape)),
                )
            mm_node.replace_all_uses_with(new_mm_node)

    def erase(self) -> None:
        for node in reversed(self.nodes):
            if len(node.users) == 0:
                node.graph.erase_node(node)

    @classmethod
    def from_match(cls, match: list[torch.fx.Node]) -> "_Matmul":
        assert len(match) in (1, 3)
        assert match[0].target in (
            aten.mm.default,
            aten.reshape.default,
        )
        mm_node = match[0] if len(match) == 1 else match[1]
        return _Matmul(
            nodes=match,
            A_node=cast("torch.fx.Node", match[0].args[0]),
            B_node=cast("torch.fx.Node", mm_node.args[1]),
            # _Matmul handles reshapes via custom graph manipulation logic, see `replace_with()` method.
            # TODO: explore unifying the _Matmul and _ScaledMatmul approaches to handling reshapes.
            pre_mm_reshape=None,
````
- **EN**: Introduces function `erase`, function `from_match`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`erase`、函数`from_match`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 449-476 / 第 449-476 行
````python
            post_mm_reshape=None,
        )


@dataclass
class _ScaledMatmul(_Matmul):
    A_scale_node: torch.fx.Node
    B_scale_node: torch.fx.Node
    bias_node: torch.fx.Node | None
    result_scale_node: torch.fx.Node | None
    out_dtype: torch.dtype | None
    use_fast_accum: bool
    pre_mm_reshape: torch.fx.Node | None
    post_mm_reshape: torch.fx.Node | None

    def __post_init__(self):
        super().__post_init__()
        self.arg_ancestor_nodes |= _find_ancestors(self.A_scale_node)
        self.arg_ancestor_nodes |= _find_ancestors(self.B_scale_node)

    @classmethod
    def from_match(cls, match: list[torch.fx.Node]) -> "_ScaledMatmul":
        assert len(match) in (1, 3)
        assert match[0].target in (
            aten._scaled_mm.default,
            aten.reshape.default,
        )

````
- **EN**: Introduces class `_ScaledMatmul`, function `__post_init__`, function `from_match`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了类`_ScaledMatmul`、函数`__post_init__`、函数`from_match`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 477-504 / 第 477-504 行
````python
        def get_arg(node: torch.fx.Node, idx: int, default: Any) -> Any:
            if idx >= len(node.args):
                return default
            return node.args[idx]

        # Use mm_node with 2D args for both A and B, even if this is a "reshape -> mm -> reshape" pattern.
        # We will store the reshapes in pre_mm_reshape and post_mm_reshape, to be referenced later to
        # produce the correct output shapes, reduce-scatter along the correct dimensions, etc.
        is_reshape_mm_reshape_pattern = match[0].target is aten.reshape.default
        mm_node = match[1] if is_reshape_mm_reshape_pattern else match[0]
        pre_mm_reshape = match[0] if is_reshape_mm_reshape_pattern else None
        post_mm_reshape = match[-1] if is_reshape_mm_reshape_pattern else None
        A_node = cast("torch.fx.Node", mm_node.args[0])
        B_node = cast("torch.fx.Node", mm_node.args[1])
        A_scale_node = cast("torch.fx.Node", mm_node.args[2])
        B_scale_node = cast("torch.fx.Node", mm_node.args[3])

        return _ScaledMatmul(
            nodes=match,
            A_node=A_node,
            B_node=B_node,
            A_scale_node=A_scale_node,
            B_scale_node=B_scale_node,
            bias_node=get_arg(mm_node, 4, None),
            result_scale_node=get_arg(mm_node, 5, None),
            out_dtype=get_arg(mm_node, 6, None),
            use_fast_accum=get_arg(mm_node, 7, False),
            pre_mm_reshape=pre_mm_reshape,
````
- **EN**: Introduces function `get_arg`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_reshape_mm_reshape_pattern`, `mm_node`, `pre_mm_reshape`, `post_mm_reshape`, `A_node`, `B_node`, and `...+7`.
- **CN**: 这里定义了函数`get_arg`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_reshape_mm_reshape_pattern`、`mm_node`、`pre_mm_reshape`、`post_mm_reshape`、`A_node`、`B_node`、`另有7项` 等值。

### Lines 505-532 / 第 505-532 行
````python
            post_mm_reshape=post_mm_reshape,
        )


def _find_reshape_mm_reshape(node: torch.fx.Node) -> list[_Matmul]:
    if node.target != aten.reshape.default:
        return []

    matches = []
    for mm_node in node.users:
        if mm_node.target not in (aten.mm.default, aten._scaled_mm.default):
            continue
        for reshape_node in mm_node.users:
            if reshape_node.target != aten.reshape.default:
                continue

            # Since the reshape -> mm -> reshape pattern would be subsumed into
            # the fused op, we only match the patterns where the shape of the
            # second reshape is matches the mm result produced by the fused op.
            matmul_input_node = cast("torch.fx.Node", node.args[0])
            B_node = cast("torch.fx.Node", mm_node.args[1])
            matmul_out_shape = torch.Size(
                [
                    *_get_tensor(matmul_input_node).shape[:-1],
                    _get_tensor(B_node).shape[-1],
                ]
            )
            if _get_tensor(reshape_node).shape != matmul_out_shape:
````
- **EN**: Introduces function `_find_reshape_mm_reshape`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `post_mm_reshape`, `matches`, `matmul_input_node`, `B_node`, and `matmul_out_shape`.
- **CN**: 这里定义了函数`_find_reshape_mm_reshape`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `post_mm_reshape`、`matches`、`matmul_input_node`、`B_node`、`matmul_out_shape` 等值。

### Lines 533-560 / 第 533-560 行
````python
                continue
            matches.append([node, mm_node, reshape_node])
            # If for some rare reason mm_node is being reshaped by two
            # different reshape nodes, we only include mm_node once in the
            # parsing result.
            break

    matmuls = []
    for match in matches:
        mm_node = match[1]
        if mm_node.target is aten.mm.default:
            matmul = _Matmul.from_match(match)
            matmuls.append(matmul)
        elif mm_node.target is aten._scaled_mm.default:
            matmul = _ScaledMatmul.from_match(match)
            matmuls.append(matmul)
        else:
            raise AssertionError(
                "Expect the node's target to be either aten.mm.default or "
                f"aten._scaled_mm.default. Got {mm_node.target}."
            )
    return matmuls


def _find_consumer_matmuls(node: torch.fx.Node) -> list[_Matmul]:
    """
    Find the matmuls that use `node` as the lhs argument.
    """
````
- **EN**: Introduces function `_find_consumer_matmuls`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `matmuls`, `mm_node`, `matmul`, and `else`.
- **CN**: 这里定义了函数`_find_consumer_matmuls`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `matmuls`、`mm_node`、`matmul`、`else` 等值。

### Lines 561-588 / 第 561-588 行
````python
    matmuls = []
    for user in node.users:
        # ND matmuls
        if user.target is aten.reshape.default:
            matmuls.extend(_find_reshape_mm_reshape(user))
        # 2D matmuls
        elif user.target is aten.mm.default:
            matmul = _Matmul.from_match(match=[user])
            matmuls.append(matmul)
        elif user.target is aten._scaled_mm.default:
            matmul = _ScaledMatmul.from_match([user])
            matmuls.append(matmul)
    return matmuls


def _insert_fused_all_gather_matmul(
    graph: torch.fx.Graph,
    matmuls: list[_Matmul],
    shard_node: torch.fx.Node,
    gather_dim: int,
    group_name: "torch.distributed.distributed_c10d.GroupName",
) -> torch.fx.Node:
    mm_types = OrderedSet(map(type, matmuls))
    assert len(mm_types) == 1
    mm_type = next(iter(mm_types))
    if mm_type == _Matmul:
        B_nodes = [matmul.B_node for matmul in matmuls]
        return graph.call_function(
````
- **EN**: Introduces function `_insert_fused_all_gather_matmul`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `matmuls`, `matmul`, `graph`, `shard_node`, `gather_dim`, `group_name`, and `...+3`.
- **CN**: 这里定义了函数`_insert_fused_all_gather_matmul`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `matmuls`、`matmul`、`graph`、`shard_node`、`gather_dim`、`group_name`、`另有3项` 等值。

### Lines 589-616 / 第 589-616 行
````python
            torch.ops.symm_mem.fused_all_gather_matmul.default,
            args=(shard_node, B_nodes, gather_dim, group_name),
            kwargs={"return_A": True},
        )
    elif mm_type == _ScaledMatmul:
        scaled_matmuls = cast("list[_ScaledMatmul]", matmuls)
        return graph.call_function(
            torch.ops.symm_mem.fused_all_gather_scaled_matmul.default,
            args=(
                shard_node,
                [matmul.B_node for matmul in scaled_matmuls],
                scaled_matmuls[0].A_scale_node,
                [matmul.B_scale_node for matmul in scaled_matmuls],
                gather_dim,
                group_name,
                [matmul.bias_node for matmul in scaled_matmuls],
                [matmul.result_scale_node for matmul in scaled_matmuls],
                [matmul.out_dtype for matmul in scaled_matmuls],
                [matmul.use_fast_accum for matmul in scaled_matmuls],
            ),
        )
    else:
        raise AssertionError(f"Unexpected matmul match type: {mm_type}")


def fuse_all_gather_matmul(all_gather: _AllGatherMatch) -> None:
    """
    Fused the pattern
````
- **EN**: Introduces function `fuse_all_gather_matmul`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `args`, `kwargs`, `scaled_matmuls`, and `else`.
- **CN**: 这里定义了函数`fuse_all_gather_matmul`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `args`、`kwargs`、`scaled_matmuls`、`else` 等值。

### Lines 617-644 / 第 617-644 行
````python

        A = all_gather_tensor(A_shard, gather_dim, group_name)
        C_0 = torch.matmul(A, B_0)
        C_1 = torch.matmul(A, B_1)
        C_2 = torch.matmul(A, B_2)
        ...

    into

        A, Cs = torch.ops.symm_mem.fused_all_gather_matmul(
            A_shard, [B_0, B_1, B_2, ...], gather_dim, group_name,
        )
    """
    if (
        not torch.distributed.is_available()
        or not torch.distributed.is_nccl_available()
    ):
        return

    from torch.distributed._symmetric_memory import (
        is_symm_mem_enabled_for_group,
        restride_A_shard_for_fused_all_gather_matmul,
    )

    shard_node, ag_node, ag_res_node, gather_dim, group_name = (
        all_gather.shard_node,
        all_gather.ag_node,
        all_gather.res_node,
````
- **EN**: Imports dependencies such as `torch.distributed._symmetric_memory` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `A`, `C_0`, `C_1`, and `C_2`.
- **CN**: 这里导入了 `torch.distributed._symmetric_memory` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `A`、`C_0`、`C_1`、`C_2` 等值。

### Lines 645-672 / 第 645-672 行
````python
        all_gather.gather_dim,
        all_gather.group_name,
    )

    if not is_symm_mem_enabled_for_group(group_name):
        return

    filter_matmul = None
    if _is_last_dim(_get_tensor(shard_node), gather_dim):
        # Decomposed mms should not be too small
        if _get_tensor(shard_node).shape[-1] < 1024:
            return

        # scaled_mm is not supported yet for last dim
        def _filter_out_scaled_matmul(matmul: _Matmul):
            return not isinstance(matmul, _ScaledMatmul)

        filter_matmul = _filter_out_scaled_matmul

    # Find consumer matmuls
    matmuls = _find_consumer_matmuls(ag_res_node)

    # The matmuls are only fusible if non-A args don't depend on the all-gather
    # result node
    matmuls = [
        matmul
        for matmul in matmuls
        if all_gather.res_node not in matmul.arg_ancestor_nodes
````
- **EN**: Introduces function `_filter_out_scaled_matmul`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `filter_matmul`, and `matmuls`.
- **CN**: 这里定义了函数`_filter_out_scaled_matmul`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `filter_matmul`、`matmuls` 等值。

### Lines 673-700 / 第 673-700 行
````python
    ]

    if len(matmuls) == 0 or len(OrderedSet(map(type, matmuls))) != 1:
        return

    if _is_last_dim(_get_tensor(shard_node), gather_dim) and len(
        all_gather.res_node.users
    ) > len(matmuls):
        # The result of ag-split-cat is used not only in matmuls.
        # Then it has to be materialized, which can have overhead.
        return

    if filter_matmul and not filter_matmul(matmuls[0]):
        return

    # Fuse the all_gather_tensor with the eligible matmuls
    graph = ag_node.graph
    with graph.inserting_before(ag_node):
        if not _is_last_dim(_get_tensor(shard_node), gather_dim):
            if "val" in shard_node.meta:
                restrided = restride_A_shard_for_fused_all_gather_matmul(
                    _get_tensor(shard_node),
                    gather_dim,
                )
                shard_node = graph.call_function(
                    inductor_prims.force_stride_order,
                    args=(shard_node, restrided.stride()),
                )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph`, `restrided`, `shard_node`, and `args`. This range continues the implementation of function `fuse_all_gather_matmul`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph`、`restrided`、`shard_node`、`args` 等值。这一段延续了函数`fuse_all_gather_matmul` 的具体实现。

### Lines 701-728 / 第 701-728 行
````python

        fused_node = _insert_fused_all_gather_matmul(
            graph, matmuls, shard_node, gather_dim, group_name
        )
        new_ag_node = graph.call_function(
            operator.getitem,
            args=(fused_node, 0),
        )
        new_out_nodes = graph.call_function(
            operator.getitem,
            args=(fused_node, 1),
        )
        for idx, matmul in enumerate(matmuls):
            new_out_node = graph.call_function(
                operator.getitem,
                args=(new_out_nodes, idx),
            )
            matmul.replace_with(new_out_node)
            matmul.erase()
        all_gather.replace_with(new_ag_node)
        all_gather.erase()

        # If the new_ag_node has no users, we tell the fused op to not return
        # it. This creates more optimization opportunities.
        if len(new_ag_node.users) == 0:
            graph.erase_node(new_ag_node)
            kwargs = dict(fused_node.kwargs)
            if "return_A" in kwargs:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fused_node`, `new_ag_node`, `args`, `new_out_nodes`, `new_out_node`, and `kwargs`. This range continues the implementation of function `fuse_all_gather_matmul`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `fused_node`、`new_ag_node`、`args`、`new_out_nodes`、`new_out_node`、`kwargs` 等值。这一段延续了函数`fuse_all_gather_matmul` 的具体实现。

### Lines 729-756 / 第 729-756 行
````python
                kwargs["return_A"] = False
                fused_node.kwargs = kwargs

    # Raise ancestors of non-A args that are topologically ordered between
    # ag_res_node and the matmul above fused_node.
    order = {node: idx for idx, node in enumerate(graph.nodes)}
    nodes_to_raise = sorted(
        OrderedSet(x for matmul in matmuls for x in matmul.arg_ancestor_nodes),
        key=lambda x: order[x],
    )
    for node in nodes_to_raise:
        if order[node] > order[fused_node]:
            fused_node.prepend(node)


def _scatter_dim_after_reshape(
    reshape_node: torch.fx.Node, orig_scatter_dim: int
) -> int:
    """
    Given a reshape node and the original scatter dim for the target tensor,
    returns the new scatter dim for the reshaped tensor.
    """
    # if there was no pre-mm reshape, scatter dim will not change.
    if not reshape_node:
        return orig_scatter_dim

    reshape_op_output_tensor = _get_tensor(reshape_node)
    assert reshape_op_output_tensor.ndim == 2, (
````
- **EN**: Introduces function `_scatter_dim_after_reshape`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `order`, `nodes_to_raise`, `key`, `reshape_node`, and `reshape_op_output_tensor`.
- **CN**: 这里定义了函数`_scatter_dim_after_reshape`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `order`、`nodes_to_raise`、`key`、`reshape_node`、`reshape_op_output_tensor` 等值。

### Lines 757-784 / 第 757-784 行
````python
        "reshape must produce 2D tensor for scaled_mm"
    )

    assert len(reshape_node.args) >= 1, "reshape node must have at least 1 arg"
    input_tensor_node = cast(torch.fx.Node, reshape_node.args[0])
    reshape_op_input_tensor = _get_tensor(input_tensor_node)
    assert reshape_op_input_tensor.ndim > reshape_op_output_tensor.ndim, (
        "reshape must be from 3D+ to 2D"
    )

    # Note: for a N-D tensor to be reshaped into 2D, either the leading dims or ending dims must
    # be collapsed to a single dim. First determine which of these happened.
    input_shape = reshape_op_input_tensor.shape
    output_shape = reshape_op_output_tensor.shape
    leading_dims_collapsed = output_shape[0] == prod(input_shape[:-1])

    # Case 1: scatter dim 0 always maps to 0 after any reshape from 3D+ to 2D, regardless if
    # leading dims or ending dims were collapsed.
    if orig_scatter_dim == 0:
        return 0

    # Case 2: scatter dim "ndim-1" always maps to 1 after any reshape from 3D+ to 2D, regardless if
    # leading dims or ending dims were collapsed.
    if orig_scatter_dim == reshape_op_input_tensor.ndim - 1:
        return 1

    # Case 3: scatter dim was one of the middle dims (between 0 and ndim-1).
    # if the leading dims were collapsed, the new scatter dim will be 0.
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_tensor_node`, `reshape_op_input_tensor`, `input_shape`, `output_shape`, and `leading_dims_collapsed`. This range continues the implementation of function `_scatter_dim_after_reshape`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_tensor_node`、`reshape_op_input_tensor`、`input_shape`、`output_shape`、`leading_dims_collapsed` 等值。这一段延续了函数`_scatter_dim_after_reshape` 的具体实现。

### Lines 785-812 / 第 785-812 行
````python
    # if the ending dims were collapsed, the new scatter dim will be 1.
    return 0 if leading_dims_collapsed else 1


def _find_producer_matmul(node: torch.fx.Node) -> _Matmul | None:
    """
    Returns producer matmul node if found, otherwise returns None.
    """
    if node.target is aten.mm.default:
        return _Matmul.from_match(match=[node])
    elif node.target is aten._scaled_mm.default:
        return _ScaledMatmul.from_match(match=[node])
    elif node.target is aten.reshape.default:
        reshape_node_1 = node

        mm_node = reshape_node_1.args[0]
        assert isinstance(mm_node, torch.fx.Node)
        if mm_node.target not in (aten.mm.default, aten._scaled_mm.default):
            return None

        reshape_node_0 = mm_node.args[0]
        assert isinstance(reshape_node_0, torch.fx.Node)
        if reshape_node_0.target != aten.reshape.default:
            return None

        if mm_node.target is aten.mm.default:
            return _Matmul.from_match(match=[reshape_node_0, mm_node, reshape_node_1])
        elif mm_node.target is aten._scaled_mm.default:
````
- **EN**: Introduces function `_find_producer_matmul`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `reshape_node_1`, `mm_node`, and `reshape_node_0`.
- **CN**: 这里定义了函数`_find_producer_matmul`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `reshape_node_1`、`mm_node`、`reshape_node_0` 等值。

### Lines 813-840 / 第 813-840 行
````python
            return _ScaledMatmul.from_match(
                match=[reshape_node_0, mm_node, reshape_node_1]
            )
    return None


def _insert_fused_matmul_reduce_scatter(
    graph: torch.fx.Graph,
    matmul: _Matmul,
    reduce_op: str,
    orig_scatter_dim: int,
    group_name: "torch.distributed.distributed_c10d.GroupName",
    scatter_dim_after_reshape: int,  # only used for reshape -> scaled_mm -> reshape pattern
    output_shape: list[int],  # only used for reshape -> scaled_mm -> reshape pattern
) -> torch.fx.Node:
    if type(matmul) is _Matmul:
        return graph.call_function(
            torch.ops.symm_mem.fused_matmul_reduce_scatter.default,
            args=(
                matmul.A_node,
                matmul.B_node,
                reduce_op,
                orig_scatter_dim,
                group_name,
            ),
        )
    elif type(matmul) is _ScaledMatmul:
        return graph.call_function(
````
- **EN**: Introduces function `_insert_fused_matmul_reduce_scatter`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `match`, `graph`, `matmul`, `reduce_op`, `orig_scatter_dim`, `group_name`, and `...+3`.
- **CN**: 这里定义了函数`_insert_fused_matmul_reduce_scatter`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `match`、`graph`、`matmul`、`reduce_op`、`orig_scatter_dim`、`group_name`、`另有3项` 等值。

### Lines 841-868 / 第 841-868 行
````python
            torch.ops.symm_mem.fused_scaled_matmul_reduce_scatter.default,
            args=(
                matmul.A_node,
                matmul.B_node,
                matmul.A_scale_node,
                matmul.B_scale_node,
                reduce_op,
                orig_scatter_dim,
                scatter_dim_after_reshape,
                group_name,
                output_shape,
                matmul.bias_node,
                matmul.result_scale_node,
                matmul.out_dtype,
                matmul.use_fast_accum,
            ),
        )
    else:
        raise AssertionError(f"Unexpected matmul match type: {type(matmul)}")


def fuse_matmul_reduce_scatter(reduce_scatter: _ReduceScatterMatch) -> None:
    """
    Fused the pattern

        reduce_scatter_tensor(A @ B, scatter_dim, group_name)

    into
````
- **EN**: Introduces function `fuse_matmul_reduce_scatter`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `args`, and `else`.
- **CN**: 这里定义了函数`fuse_matmul_reduce_scatter`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `args`、`else` 等值。

### Lines 869-896 / 第 869-896 行
````python

        torch.ops.symm_mem.fused_matmul_reduce_scatter(
            A, B, scatter_dim, group_name,
        )

    Returns boolean indicating if fusion was successful or not.
    """
    if (
        not torch.distributed.is_available()
        or not torch.distributed.is_nccl_available()
    ):
        return

    from torch.distributed._symmetric_memory import (
        is_symm_mem_enabled_for_group,
        restride_A_for_fused_matmul_reduce_scatter,
    )

    (
        input_node,
        _reduce_scatter_node,
        rs_wait_tensor_node,
        reduce_op,
        orig_scatter_dim,
        group_name,
    ) = (
        reduce_scatter.input_node,
        reduce_scatter.reduce_scatter_node,
````
- **EN**: Imports dependencies such as `torch.distributed._symmetric_memory` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `fuse_matmul_reduce_scatter`.
- **CN**: 这里导入了 `torch.distributed._symmetric_memory` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`fuse_matmul_reduce_scatter` 的具体实现。

### Lines 897-924 / 第 897-924 行
````python
        reduce_scatter.wait_tensor_node,
        reduce_scatter.reduce_op,
        reduce_scatter.scatter_dim,
        reduce_scatter.group_name,
    )

    if not is_symm_mem_enabled_for_group(group_name):
        return

    filter_matmul = None
    if _is_last_dim(_get_tensor(input_node), orig_scatter_dim):
        # scaled_mm is not supported yet for last dim mm+rs
        def _filter_out_scaled_matmul(matmul: _Matmul):
            return not isinstance(matmul, _ScaledMatmul)

        filter_matmul = _filter_out_scaled_matmul

    # Currently fused_matmul_reduce_scatter doesn't return the matmul result,
    # so we can't apply the fusion if the matmul result is used by multiple
    # users. This is not a fundamental limitation of the fused op and can be
    # addressed if needed.
    if len(input_node.users) != 1:
        log.warning(
            "matmul result has more than one user, skipping fused_matmul_reduce_scatter fusion."
        )
        return

    matmul = _find_producer_matmul(input_node)
````
- **EN**: Introduces function `_filter_out_scaled_matmul`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `filter_matmul`, and `matmul`.
- **CN**: 这里定义了函数`_filter_out_scaled_matmul`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `filter_matmul`、`matmul` 等值。

### Lines 925-952 / 第 925-952 行
````python

    if matmul is None:
        log.warning(
            "no producer matmul found for reduce scatter, skipping fuse_matmul_reduce_scatter fusion"
        )
        return

    if filter_matmul and not filter_matmul(matmul):
        return

    if rs_wait_tensor_node in matmul.arg_ancestor_nodes:
        log.warning(
            "reduce-scatter result node is an ancestor of matmul, skipping fuse_matmul_reduce_scatter fusion"
        )
        return

    # We need to track 3 values for the fused scaled mm reduce scatter implementation:
    #   1. The scatter dim before the reshape, which was assigned using the original (a,b,c) @ (c,d) = (a,b,d) dims.
    #   2. The scatter dim after the reshape, to use when we are doing the 2D (a*b,c) @ (c,d) = (a,b,d) scaled mm op.
    #   3. Store expected potentially 3D+ mm output shape, so we can reshape the 2D mm output to the intended
    #      3D+ shape before applying reduce-scatter, and to prevent shape errors with subsequent ops.

    # If 'A' was reshaped from 3D+ -> 2D for the mm, we need to determine the new scattter dim after the reshape
    # for the fused matmul reduce scatter implementation to use.
    if matmul.pre_mm_reshape:
        scatter_dim_after_maybe_reshape = _scatter_dim_after_reshape(
            matmul.pre_mm_reshape, orig_scatter_dim
        )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scatter_dim_after_maybe_reshape`. This range continues the implementation of function `fuse_matmul_reduce_scatter`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `scatter_dim_after_maybe_reshape` 等值。这一段延续了函数`fuse_matmul_reduce_scatter` 的具体实现。

### Lines 953-980 / 第 953-980 行
````python
    else:
        scatter_dim_after_maybe_reshape = orig_scatter_dim

    # If the 2D mm output was reshaped from 2D -> 3D+, we need to store the intended output shape for the
    # fused matmul reduce scatter implementation to use.
    if matmul.post_mm_reshape:
        output_shape = list(_get_tensor(matmul.post_mm_reshape).shape)
    else:
        A_orig_shape = list(_get_tensor(matmul.A_node).shape)
        B_shape = list(_get_tensor(matmul.B_node).shape)
        output_shape = [*A_orig_shape[:-1], B_shape[-1]]

    graph = rs_wait_tensor_node.graph
    with graph.inserting_before(rs_wait_tensor_node):
        # Restride A tensor before fused op, for optimal perf in fused matmul reduce scatter
        if "val" in matmul.A_node.meta:
            restrided = restride_A_for_fused_matmul_reduce_scatter(
                _get_tensor(matmul.A_node),
                scatter_dim_after_maybe_reshape,
            )
            matmul.A_node = graph.call_function(
                inductor_prims.force_stride_order,
                args=(matmul.A_node, restrided.stride()),
            )

        # Replace matched subgraph with fused matmul reduce scatter node
        fused_node = _insert_fused_matmul_reduce_scatter(
            graph,
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `scatter_dim_after_maybe_reshape`, `output_shape`, `A_orig_shape`, `B_shape`, `graph`, and `...+3`. This range continues the implementation of function `fuse_matmul_reduce_scatter`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`scatter_dim_after_maybe_reshape`、`output_shape`、`A_orig_shape`、`B_shape`、`graph`、`另有3项` 等值。这一段延续了函数`fuse_matmul_reduce_scatter` 的具体实现。

### Lines 981-1008 / 第 981-1008 行
````python
            matmul,
            reduce_op,
            orig_scatter_dim,
            group_name,
            scatter_dim_after_maybe_reshape,
            output_shape,
        )
        reduce_scatter.replace_with(fused_node)
        reduce_scatter.erase()
        matmul.erase()

    order = {node: idx for idx, node in enumerate(graph.nodes)}
    nodes_to_raise = sorted(
        matmul.arg_ancestor_nodes,
        key=lambda x: order[x],
    )
    for node in nodes_to_raise:
        if order[node] > order[fused_node]:
            fused_node.prepend(node)

    log.debug("successfully fused matmul reduce scatter")


def _get_node_to_ancestors(
    graph: torch.fx.Graph,
) -> dict[torch.fx.Node, OrderedSet[torch.fx.Node]]:
    """
    Compute the ancestors for all nodes in a graph.
````
- **EN**: Introduces function `_get_node_to_ancestors`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `order`, `nodes_to_raise`, `key`, and `graph`.
- **CN**: 这里定义了函数`_get_node_to_ancestors`。包含分支、循环或上下文管理等控制流。初始化或更新了 `order`、`nodes_to_raise`、`key`、`graph` 等值。

### Lines 1009-1036 / 第 1009-1036 行
````python
    """
    node_to_ancestors = defaultdict(OrderedSet[torch.fx.Node])  # type: ignore[var-annotated]
    for node in graph.nodes:
        node_to_ancestors[node] = OrderedSet(node.all_input_nodes)
        for dep in node.all_input_nodes:
            node_to_ancestors[node] |= node_to_ancestors[dep]

    return node_to_ancestors


def _get_collective_to_overlappable_nodes(
    graph: torch.fx.Graph,
) -> dict[torch.fx.Node, list[torch.fx.Node]]:
    """
    For each collective in the graph, find nodes that are neither ancestors nor
    descendants of the collective.
    """

    def is_collective(node) -> bool:
        # Only consider all-gather and reduce-scatter in the context of
        # micro-pipeline TP.
        return node.target in [
            torch.ops._c10d_functional.all_gather_into_tensor.default,
            torch.ops._c10d_functional.reduce_scatter_tensor.default,
        ]

    node_to_ancestors = _get_node_to_ancestors(graph)
    collective_to_overlappable_nodes = defaultdict(list)
````
- **EN**: Introduces function `_get_collective_to_overlappable_nodes`, function `is_collective`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node_to_ancestors`, `graph`, and `collective_to_overlappable_nodes`.
- **CN**: 这里定义了函数`_get_collective_to_overlappable_nodes`、函数`is_collective`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `node_to_ancestors`、`graph`、`collective_to_overlappable_nodes` 等值。

### Lines 1037-1064 / 第 1037-1064 行
````python
    for node in graph.nodes:
        if not is_collective(node):
            continue
        for x in graph.nodes:
            if (
                node not in node_to_ancestors[x]
                and x not in node_to_ancestors[node]
                and x.op == "call_function"
            ):
                collective_to_overlappable_nodes[node].append(x)

    return collective_to_overlappable_nodes


def _get_unexposed_collectives(graph: torch.fx.Graph) -> list[torch.fx.Node]:
    """
    Find all unexposed collectives in the graph.

    Because we don't have the runtime estimate, this function is a rough
    estimation using the following strong/hand-wavy assumptions:

    - Only a predefined set of "compute intensive" operation can hide a collective.
    - Any "compute intensive" operation can hide exactly one collective.
    """

    def _is_compute_intensive(node: torch.fx.Node) -> bool:
        return node.target is torch.ops.aten.mm.default

````
- **EN**: Introduces function `_get_unexposed_collectives`, function `_is_compute_intensive`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_get_unexposed_collectives`、函数`_is_compute_intensive`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1065-1092 / 第 1065-1092 行
````python
    collective_to_overlapping_candidates = defaultdict(list)
    available_nodes = OrderedSet[torch.fx.Node]()
    collective_to_overlappable_nodes = _get_collective_to_overlappable_nodes(graph)
    for collective, overlappable_nodes in collective_to_overlappable_nodes.items():
        candidates = [x for x in overlappable_nodes if _is_compute_intensive(x)]
        collective_to_overlapping_candidates[collective] = candidates
        available_nodes.update(candidates)

    unexposed_collectives = []
    for (
        collective,
        overlapping_candidates,
    ) in collective_to_overlapping_candidates.items():
        # Each collective consumes exactly one overlapping candidate
        for x in overlapping_candidates:
            if x in available_nodes:
                unexposed_collectives.append(collective)
                available_nodes.remove(x)
                break
    return unexposed_collectives


def micro_pipeline_tp_pass(graph: torch.fx.Graph):
    all_gathers = find_all_gather_patterns(graph)
    reduce_scatters = find_reduce_scatter_patterns(graph)

    # When a collective can be hidden through either simple overlapping or
    # micro-pipeline TP, we prefer simple overlapping to avoid the overhead
````
- **EN**: Introduces function `micro_pipeline_tp_pass`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `collective_to_overlapping_candidates`, `available_nodes`, `collective_to_overlappable_nodes`, `candidates`, `unexposed_collectives`, `all_gathers`, and `...+1`.
- **CN**: 这里定义了函数`micro_pipeline_tp_pass`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `collective_to_overlapping_candidates`、`available_nodes`、`collective_to_overlappable_nodes`、`candidates`、`unexposed_collectives`、`all_gathers`、`另有1项` 等值。

### Lines 1093-1114 / 第 1093-1114 行
````python
    # associated with decomposition. If reorder_for_compute_comm_overlap is
    # enabled, we identify collectives that can be hidden through simple
    # overlapping and exclude them from micro-pipeline TP candidates.
    if config.reorder_for_compute_comm_overlap:
        unexposed_collectives = _get_unexposed_collectives(graph)
        all_gathers = [x for x in all_gathers if x.ag_node not in unexposed_collectives]
        reduce_scatters = [
            x
            for x in reduce_scatters
            if x.reduce_scatter_node not in unexposed_collectives
        ]

    if not all_gathers and not reduce_scatters:
        log.warning(
            "async TP found no matching all-gather/reduce-scatter patterns for fusion"
        )

    for all_gather in all_gathers:
        fuse_all_gather_matmul(all_gather)

    for reduce_scatter in reduce_scatters:
        fuse_matmul_reduce_scatter(reduce_scatter)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `unexposed_collectives`, `all_gathers`, and `reduce_scatters`. This range continues the implementation of function `micro_pipeline_tp_pass`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `unexposed_collectives`、`all_gathers`、`reduce_scatters` 等值。这一段延续了函数`micro_pipeline_tp_pass` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `_AllGatherMatch`, `_ReduceScatterMatch`, `_Matmul`, and `_ScaledMatmul`  
  **CN**: 主要类：`_AllGatherMatch`、`_ReduceScatterMatch`、`_Matmul`、`_ScaledMatmul`
- **EN**: Primary functions: `_is_last_dim`, `_is_backward`, `_compute_mm_arithmetic_intensity`, `_filter_nodes_by_target`, `_find_ancestors`, `_get_tensor`, and `...+14`  
  **CN**: 主要函数：`_is_last_dim`、`_is_backward`、`_compute_mm_arithmetic_intensity`、`_filter_nodes_by_target`、`_find_ancestors`、`_get_tensor`、`另有14项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `operator`, `collections`, `dataclasses`, `math`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils._ordered_set`, `..`, `..pattern_matcher`, `torch.distributed._symmetric_memory`
