# propagate_scale_by.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/auto_chunker/propagate_scale_by.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `_register_propagate_rule`, `register_propagate_rule`, `propagate_scale_by`, `propagate_div`, `propagate_where`, `propagate_requires_no_scaling`, and `...+4`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `_register_propagate_rule`、`register_propagate_rule`、`propagate_scale_by`、`propagate_div`、`propagate_where`、`propagate_requires_no_scaling`、`另有4项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import functools
import logging
from collections.abc import Callable, Sequence
from typing import TypeAlias

import torch
from torch.fx import Node

from .common import CantChunk
from .core import get_chunking_meta, get_chunking_metas, log, update_chunking_meta
from .utils import (
    format_node_with_chunking_meta,
    get_args_of_node_type,
    get_scale_by_from_metas,
    get_scale_by_from_node,
)


aten = torch.ops.aten
prims = torch.ops.prims
````
- **EN**: Imports dependencies such as `functools`, `logging`, `collections.abc`, `typing`, `torch`, `torch.fx`, and `...+3` for the logic in this range. Initializes or updates values such as `aten`, and `prims`.
- **CN**: 这里导入了 `functools`、`logging`、`collections.abc`、`typing`、`torch`、`torch.fx`、`另有3项` 等依赖，为后续逻辑提供基础能力。初始化或更新了 `aten`、`prims` 等值。

### Lines 21-40 / 第 21-40 行
````python

_HandlerType: TypeAlias = Callable[[Node], bool]
propagate_rules: dict[torch._ops.OpOverload, _HandlerType] = {}


def _register_propagate_rule(
    aten_op: torch._ops.OpOverload | Sequence[torch._ops.OpOverload],
    handler: _HandlerType,
) -> _HandlerType:
    if not isinstance(aten_op, (list, tuple)):
        aten_op = [aten_op]  # type: ignore[assignment, list-item]

    assert isinstance(aten_op, (list, tuple)), f"{type(aten_op)=}"
    for op in aten_op:
        assert isinstance(op, torch._ops.OpOverload)
        propagate_rules[op] = handler
    return handler


def register_propagate_rule(
````
- **EN**: Introduces function `_register_propagate_rule`, function `register_propagate_rule`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_register_propagate_rule`、函数`register_propagate_rule`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 41-60 / 第 41-60 行
````python
    aten_op: torch._ops.OpOverload | Sequence[torch._ops.OpOverload],
) -> Callable[[_HandlerType], _HandlerType]:
    return functools.partial(_register_propagate_rule, aten_op)


def propagate_scale_by(nodes_with_chunking_meta: Sequence[Node]) -> None:
    """
    The input is a list of nodes that have chunking metadata.
    The nodes are already sorted in topological order.
    """
    for node in nodes_with_chunking_meta:
        arg_nodes = get_args_of_node_type(node)
        arg_metas = get_chunking_metas(arg_nodes)

        if all(arg_meta is None for arg_meta in arg_metas):
            # should be graph input of the chunking subgraph
            continue

        if log.isEnabledFor(logging.DEBUG):
            print("Propagate scale_by:")
````
- **EN**: Introduces function `propagate_scale_by`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`propagate_scale_by`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 61-80 / 第 61-80 行
````python
            format_node_with_chunking_meta(node, True)

        assert all(arg_meta is not None for arg_meta in arg_metas), node.format_node()

        # None of the input has scale_by set
        if all(arg_meta.scale_by is None for arg_meta in arg_metas):  # type: ignore[union-attr]
            continue

        target = node.target
        if (
            not isinstance(target, torch._ops.OpOverload)
            or target not in propagate_rules
        ):
            raise CantChunk(
                f"Missing scale_by propagation rule for target {target}: {node.format_node()}"
            )

        if not propagate_rules[target](node):
            raise CantChunk(
                f"scale_by propagate rule for {target} fail: {node.format_node()}"
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `target`. This range continues the implementation of function `propagate_scale_by`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `target` 等值。这一段延续了函数`propagate_scale_by` 的具体实现。

### Lines 81-100 / 第 81-100 行
````python
            )


@register_propagate_rule(
    [
        aten.div.Tensor,
    ]
)
def propagate_div(div_node: Node) -> bool:
    lhs_node, rhs_node = div_node.args[:2]
    assert isinstance(lhs_node, Node)
    lhs_scale_by = get_scale_by_from_node(lhs_node)

    # When gradient accumulation is enabled, rhs_node can be a constant
    # representing the gradient accumulation steps
    rhs_scale_by = (
        get_scale_by_from_node(rhs_node) if isinstance(rhs_node, Node) else None
    )
    if lhs_scale_by and rhs_scale_by is None:
        update_chunking_meta(div_node, scale_by=lhs_scale_by)
````
- **EN**: Introduces function `propagate_div`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`propagate_div`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 101-120 / 第 101-120 行
````python
        return True
    return False


@register_propagate_rule(
    [
        aten.where.self,
    ]
)
def propagate_where(where_node: Node) -> bool:
    cond_node, true_node, false_node = where_node.args
    assert isinstance(cond_node, Node)
    assert isinstance(true_node, Node)
    assert isinstance(false_node, Node)
    cond_meta, true_meta, false_meta = get_chunking_metas(
        [cond_node, true_node, false_node]
    )
    out_meta = get_chunking_meta(where_node)

    assert true_meta is not None
````
- **EN**: Introduces function `propagate_where`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`propagate_where`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 121-140 / 第 121-140 行
````python
    assert false_meta is not None
    if true_meta.scale_by and not false_meta.scale_by:
        # the false_node must be all zero
        if false_node.target != aten.full.default:
            return False
        if false_node.args[1] != 0.0:
            return False
        assert out_meta is not None
        out_meta.scale_by = true_meta.scale_by
        return True
    return False


@register_propagate_rule(
    [
        aten.exp.default,
        aten.log.default,
        aten.tanh.default,
        aten.eq.Tensor,
    ]
````
- **EN**: Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
)
def propagate_requires_no_scaling(out_node: Node) -> bool:
    """
    For nonlinear ops (exp, log, tanh) scale_by cannot be propagated
    through since f(S*x) != S*f(x). For boolean-output ops (eq) scale_by
    is meaningless. These ops only appear in the chunking subgraph when
    scale_by is None (e.g. the final gradient is 1).
    """
    args_node = get_args_of_node_type(out_node)
    args_meta = get_chunking_metas(args_node)
    out_meta = get_chunking_meta(out_node)

    scale_by = get_scale_by_from_metas(*args_meta)  # type: ignore[arg-type]
    assert scale_by is None, (
        f"Nonlinear op {out_node.target} requires scale_by=None, got {scale_by}"
    )
    assert out_meta is not None
    out_meta.scale_by = None
    return True

````
- **EN**: Introduces function `propagate_requires_no_scaling`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `args_node`, `args_meta`, `out_meta`, and `scale_by`.
- **CN**: 这里定义了函数`propagate_requires_no_scaling`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `args_node`、`args_meta`、`out_meta`、`scale_by` 等值。

### Lines 161-180 / 第 161-180 行
````python

@register_propagate_rule(
    [
        aten.mul.Tensor,
        prims.convert_element_type.default,
        aten.neg.default,
        aten.sum.dim_IntList,
        aten.sum.default,  # sum to scalar
        aten.amax.default,
        aten.mm.default,
        aten.permute.default,
        aten.expand.default,
        aten.squeeze.dim,
        aten.unsqueeze.default,
        aten.gather.default,
        aten.scatter_add.default,
        aten.view.default,
    ]
)
def propagate_general_copy(out_node: Node) -> bool:
````
- **EN**: Introduces function `propagate_general_copy`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`propagate_general_copy`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 181-200 / 第 181-200 行
````python
    """
    A rule that holds for multiple ops: the scale_by of the output is
    set to the only scale_by of input nodes or None if no input has scale_by
    set.
    """
    args_node = get_args_of_node_type(out_node)
    args_meta = get_chunking_metas(args_node)
    out_meta = get_chunking_meta(out_node)

    scale_by = get_scale_by_from_metas(*args_meta)  # type: ignore[arg-type]
    assert out_meta is not None
    out_meta.scale_by = scale_by
    return True


@register_propagate_rule(aten.scatter.value)
def propagate_scatter_value(out_node: Node) -> bool:
    # The backward of scatter.value always has value=0 (gradient of a constant),
    # so S * scatter(x, idx, 0) = scatter(S*x, idx, 0) holds.
    value = out_node.args[3]
````
- **EN**: Introduces function `propagate_scatter_value`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`propagate_scatter_value`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 201-220 / 第 201-220 行
````python
    if value != 0:
        return False
    return propagate_general_copy(out_node)


@register_propagate_rule(
    [
        aten.add.Tensor,
        aten.sub.Tensor,
    ]
)
def propagate_add_sub(out_node: Node) -> bool:
    """
    The scale_by node of the two arguments must be the same.
    """
    lhs_node, rhs_node = get_args_of_node_type(out_node)
    assert isinstance(lhs_node, Node)
    assert isinstance(rhs_node, Node)
    lhs_meta, rhs_meta = get_chunking_metas([lhs_node, rhs_node])
    assert lhs_meta is not None
````
- **EN**: Introduces function `propagate_add_sub`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`propagate_add_sub`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 221-240 / 第 221-240 行
````python
    assert rhs_meta is not None
    if lhs_meta.scale_by is rhs_meta.scale_by:
        update_chunking_meta(out_node, scale_by=lhs_meta.scale_by)
        return True
    return False


@register_propagate_rule(
    [
        prims.fma.default,
    ]
)
def propagate_fma(out_node: Node) -> bool:
    mul_lhs, mul_rhs, add_rhs = out_node.args[:3]
    assert isinstance(mul_lhs, Node)
    assert isinstance(mul_rhs, Node)
    assert isinstance(add_rhs, Node)
    mul_lhs_meta, mul_rhs_meta, add_rhs_meta = get_chunking_metas(
        [mul_lhs, mul_rhs, add_rhs]
    )
````
- **EN**: Introduces function `propagate_fma`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`propagate_fma`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 241-248 / 第 241-248 行
````python
    assert mul_lhs_meta is not None
    assert mul_rhs_meta is not None
    add_lhs_scale_by = get_scale_by_from_metas(mul_lhs_meta, mul_rhs_meta)
    assert add_rhs_meta is not None
    if add_lhs_scale_by is add_rhs_meta.scale_by:
        update_chunking_meta(out_node, scale_by=add_lhs_scale_by)
        return True
    return False
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `add_lhs_scale_by`. This range continues the implementation of function `propagate_fma`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `add_lhs_scale_by` 等值。这一段延续了函数`propagate_fma` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Primary functions: `_register_propagate_rule`, `register_propagate_rule`, `propagate_scale_by`, `propagate_div`, `propagate_where`, `propagate_requires_no_scaling`, and `...+4`  
  **CN**: 主要函数：`_register_propagate_rule`、`register_propagate_rule`、`propagate_scale_by`、`propagate_div`、`propagate_where`、`propagate_requires_no_scaling`、`另有4项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `logging`, `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `.common`, `.core`, `.utils`
