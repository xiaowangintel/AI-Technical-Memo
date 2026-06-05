# core.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/auto_chunker/core.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `set_chunking_meta`, `update_chunking_meta`, `set_chunking_meta_if_none`, `copy_chunking_meta`, `get_chunking_meta`, `has_nop_chunking_meta`, and `...+3`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `set_chunking_meta`、`update_chunking_meta`、`set_chunking_meta_if_none`、`copy_chunking_meta`、`get_chunking_meta`、`has_nop_chunking_meta`、`另有3项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
import logging
from collections.abc import Callable, Sequence
from typing import Any

import torch
from torch._inductor import config
from torch.fx import Graph, Node
from torch.utils._ordered_set import OrderedSet

from .common import ChunkingMeta
from .utils import (
    compute_tensor_size,
    get_args_of_node_type,
    get_fake_tensor_from_node_arg,
````
- **EN**: Imports dependencies such as `logging`, `collections.abc`, `typing`, `torch`, `torch._inductor`, `torch.fx`, and `...+3` for the logic in this range.
- **CN**: 这里导入了 `logging`、`collections.abc`、`typing`、`torch`、`torch._inductor`、`torch.fx`、`另有3项` 等依赖，为后续逻辑提供基础能力。

### Lines 15-28 / 第 15-28 行
````python
    use_tangent,
)


aten = torch.ops.aten
log = torch._logging.getArtifactLogger(__name__, "auto_chunker")


def set_chunking_meta(
    node: Node, meta: ChunkingMeta | None = None, **kwargs: Any
) -> bool:
    """
    kwargs can override fields in the passed in `meta`
    """
````
- **EN**: Introduces function `set_chunking_meta`. Initializes or updates values such as `aten`, `log`, and `node`.
- **CN**: 这里定义了函数`set_chunking_meta`。初始化或更新了 `aten`、`log`、`node` 等值。

### Lines 29-42 / 第 29-42 行
````python
    if meta is None:
        meta = ChunkingMeta(**kwargs)
    else:
        # make a copy to avoid override the passed in instance
        meta = meta.copy()
        for k, v in kwargs.items():
            setattr(meta, k, v)

    old_meta = get_chunking_meta(node)
    node.meta["chunking"] = meta
    return old_meta is None or old_meta != meta


def update_chunking_meta(node: Node, **kwargs: Any) -> bool:
````
- **EN**: Introduces function `update_chunking_meta`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `meta`, `else`, and `old_meta`.
- **CN**: 这里定义了函数`update_chunking_meta`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `meta`、`else`、`old_meta` 等值。

### Lines 43-56 / 第 43-56 行
````python
    """
    Unlike set_chunking_mete, this function keeps the existing chunking
    metadata if it's not overridden.
    """
    changed = False
    meta = get_chunking_meta(node)
    if meta is None:
        meta = ChunkingMeta()
        changed = True
    for k, v in kwargs.items():
        if getattr(meta, k, None) != v:
            changed = True
            setattr(meta, k, v)

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `changed`, and `meta`. This range continues the implementation of function `update_chunking_meta`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `changed`、`meta` 等值。这一段延续了函数`update_chunking_meta` 的具体实现。

### Lines 57-70 / 第 57-70 行
````python
    node.meta["chunking"] = meta
    return changed


def set_chunking_meta_if_none(
    nodes: Sequence[Node],
    meta: ChunkingMeta,
    filter_for_nop: Callable[[Node], bool] | None = None,
) -> bool:
    """
    If filter_fop_nop returns true for a node, we set the chunking
    meta to nop instead.
    """
    changed = False
````
- **EN**: Introduces function `set_chunking_meta_if_none`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `nodes`, `meta`, `filter_for_nop`, and `changed`.
- **CN**: 这里定义了函数`set_chunking_meta_if_none`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `nodes`、`meta`、`filter_for_nop`、`changed` 等值。

### Lines 71-84 / 第 71-84 行
````python
    for node in nodes:
        if get_chunking_meta(node) is None:
            changed = True
            if filter_for_nop and filter_for_nop(node):
                set_chunking_meta(node)
            else:
                set_chunking_meta(node, meta)
    return changed


def copy_chunking_meta(dst_node: Node, src_node: Node | ChunkingMeta) -> bool:
    if isinstance(src_node, torch.fx.Node):
        src_meta = get_chunking_meta(src_node)
    else:
````
- **EN**: Introduces function `copy_chunking_meta`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `changed`, `else`, and `src_meta`.
- **CN**: 这里定义了函数`copy_chunking_meta`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `changed`、`else`、`src_meta` 等值。

### Lines 85-98 / 第 85-98 行
````python
        assert isinstance(src_node, ChunkingMeta)
        src_meta = src_node
    assert src_meta
    return set_chunking_meta(dst_node, src_meta)


def get_chunking_meta(node: Node) -> ChunkingMeta | None:
    return node.meta.get("chunking")


def has_nop_chunking_meta(node: Node) -> bool:
    return ChunkingMeta.is_nop(get_chunking_meta(node))


````
- **EN**: Introduces function `get_chunking_meta`, function `has_nop_chunking_meta`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `src_meta`.
- **CN**: 这里定义了函数`get_chunking_meta`、函数`has_nop_chunking_meta`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `src_meta` 等值。

### Lines 99-112 / 第 99-112 行
````python
def get_chunking_metas(
    nodes: Sequence[Node], skip_none: bool = False
) -> Sequence[ChunkingMeta | None]:
    return [
        get_chunking_meta(node)
        for node in nodes
        if not skip_none or get_chunking_meta(node) is not None
    ]


eligible_amplifier_node = OrderedSet(
    [
        aten.mm.default,
        aten.addmm.default,
````
- **EN**: Introduces function `get_chunking_metas`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `nodes`, and `eligible_amplifier_node`.
- **CN**: 这里定义了函数`get_chunking_metas`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `nodes`、`eligible_amplifier_node` 等值。

### Lines 113-126 / 第 113-126 行
````python
    ]
)


def find_amplifier_node(graph: Graph) -> Node | None:
    r"""
    Find the 'amplifier' node which is a node that generates large
    output with small/medium input.

    If there are multiple amplifier nodes, return the one with the largest
    amplification ratio.
    """

    amplifier_nodes_ratio = []
````
- **EN**: Introduces function `find_amplifier_node`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `amplifier_nodes_ratio`.
- **CN**: 这里定义了函数`find_amplifier_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `amplifier_nodes_ratio` 等值。

### Lines 127-140 / 第 127-140 行
````python

    for node in graph.nodes:
        if use_tangent(node):
            # enter backward part of the graph
            break

        # Only trigger chunking for a small set of nodes like matmul for now
        if node.op != "call_function" or node.target not in eligible_amplifier_node:
            continue

        input_size = compute_tensor_size(node.args, node.kwargs)
        output_size = compute_tensor_size(node)

        if input_size == 0:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_size`, and `output_size`. This range continues the implementation of function `find_amplifier_node`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `input_size`、`output_size` 等值。这一段延续了函数`find_amplifier_node` 的具体实现。

### Lines 141-154 / 第 141-154 行
````python
            continue

        ratio = output_size / input_size
        if (
            output_size > config.auto_chunker.output_size_threshold
            and ratio > config.auto_chunker.amplify_ratio_threshold
        ):
            amplifier_nodes_ratio.append((node, ratio))

    amplifier_nodes_ratio = sorted(
        amplifier_nodes_ratio, key=lambda x: x[1], reverse=True
    )
    return amplifier_nodes_ratio[0][0] if len(amplifier_nodes_ratio) > 0 else None

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ratio`, and `amplifier_nodes_ratio`. This range continues the implementation of function `find_amplifier_node`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ratio`、`amplifier_nodes_ratio` 等值。这一段延续了函数`find_amplifier_node` 的具体实现。

### Lines 155-168 / 第 155-168 行
````python

def reorder_nodes(graph: Graph) -> Graph:
    """
    Create a new graph to be like:
    1. all nodes run before `chunking_subgraph_nodes`
    2. all nodes in `chunking_subgraph_nodes`
    3. all nodes run after `chunking_subgraph_nodes`

    Return a new graph so it's easier to fallback.
    """
    from .applier import is_chunking_subgraph_input

    # `pre_chunking_nodes` are all nodes that only depends on
    # nodes inside `pre_chuning_nodes`
````
- **EN**: Imports dependencies such as `.applier` for the logic in this range. Introduces function `reorder_nodes`.
- **CN**: 这里导入了 `.applier` 等依赖，为后续逻辑提供基础能力。这里定义了函数`reorder_nodes`。

### Lines 169-182 / 第 169-182 行
````python
    pre_chunking_nodes: OrderedSet[Node] = OrderedSet()

    for node in graph.nodes:
        if node.op == "placeholder" or is_chunking_subgraph_input(node):
            # these nodes have chunking meta but they should be placed
            # before we do chunking
            pre_chunking_nodes.add(node)

        if get_chunking_meta(node) is not None:
            continue
        if all(arg in pre_chunking_nodes for arg in get_args_of_node_type(node)):
            pre_chunking_nodes.add(node)

    post_chunking_nodes = []
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pre_chunking_nodes`, and `post_chunking_nodes`. This range continues the implementation of function `reorder_nodes`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `pre_chunking_nodes`、`post_chunking_nodes` 等值。这一段延续了函数`reorder_nodes` 的具体实现。

### Lines 183-196 / 第 183-196 行
````python

    def _copy_node(typestr: str, node: Node) -> None:
        if log.isEnabledFor(logging.DEBUG):
            fake_tensor = get_fake_tensor_from_node_arg(node)
            shape = list(fake_tensor.shape) if fake_tensor is not None else "?"
            log.debug(" - %s: %s %s", typestr, shape, node.format_node())
        env[node] = new_graph.node_copy(node, lambda x: env[x])

    # add pre_chunking_nodes
    new_graph = Graph()
    env: dict[Node, Node] = {}
    for node in pre_chunking_nodes:
        _copy_node("prechunking", node)

````
- **EN**: Introduces function `_copy_node`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fake_tensor`, `shape`, `new_graph`, and `env`.
- **CN**: 这里定义了函数`_copy_node`。包含分支、循环或上下文管理等控制流。初始化或更新了 `fake_tensor`、`shape`、`new_graph`、`env` 等值。

### Lines 197-210 / 第 197-210 行
````python
    # add nodes in the chunking subgraph
    for node in graph.nodes:
        if node in pre_chunking_nodes:
            continue
        elif get_chunking_meta(node):
            _copy_node("chunking", node)
        else:
            post_chunking_nodes.append(node)

    for node in post_chunking_nodes:
        _copy_node("postchuking", node)

    assert graph._len == new_graph._len
    new_graph.eliminate_dead_code()
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`. This range continues the implementation of function `reorder_nodes`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。这一段延续了函数`reorder_nodes` 的具体实现。

### Lines 211-219 / 第 211-219 行
````python
    new_graph.lint()

    # Need replace the scale_by node in the chunking meta with the new node
    for node in new_graph.nodes:
        meta = get_chunking_meta(node)
        if meta and meta.scale_by is not None:
            meta.scale_by = env[meta.scale_by]

    return new_graph
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `meta`. This range continues the implementation of function `reorder_nodes`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `meta` 等值。这一段延续了函数`reorder_nodes` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Primary functions: `set_chunking_meta`, `update_chunking_meta`, `set_chunking_meta_if_none`, `copy_chunking_meta`, `get_chunking_meta`, `has_nop_chunking_meta`, and `...+3`  
  **CN**: 主要函数：`set_chunking_meta`、`update_chunking_meta`、`set_chunking_meta_if_none`、`copy_chunking_meta`、`get_chunking_meta`、`has_nop_chunking_meta`、`另有3项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor`, `torch.fx`, `torch.utils._ordered_set`, `.common`, `.utils`, `.applier`
