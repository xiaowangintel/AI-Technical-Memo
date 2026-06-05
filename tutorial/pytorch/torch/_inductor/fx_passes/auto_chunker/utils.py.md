# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/auto_chunker/utils.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `is_tangent_node`, `get_args_of_node_type`, `use_tangent`, `compute_tensor_size`, `get_fake_tensor_from_node_arg`, `get_nodes_with_chunking_meta`, and `...+10`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `is_tangent_node`、`get_args_of_node_type`、`use_tangent`、`compute_tensor_size`、`get_fake_tensor_from_node_arg`、`get_nodes_with_chunking_meta`、`另有10项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
from collections.abc import Sequence
from typing import Any

import torch
from torch.fx import Graph, GraphModule, Node
from torch.utils._pytree import tree_flatten

from .common import CantChunk, ChunkingMeta


def is_tangent_node(node: Node) -> bool:
    return node.op == "placeholder" and "tangent" in node.name


````
- **EN**: Imports dependencies such as `collections.abc`, `typing`, `torch`, `torch.fx`, `torch.utils._pytree`, and `.common` for the logic in this range. Introduces function `is_tangent_node`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `collections.abc`、`typing`、`torch`、`torch.fx`、`torch.utils._pytree`、`.common` 等依赖，为后续逻辑提供基础能力。这里定义了函数`is_tangent_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 15-28 / 第 15-28 行
````python
def get_args_of_node_type(node: Node) -> Sequence[Node]:
    return [x for x in tree_flatten((node.args, node.kwargs))[0] if isinstance(x, Node)]


def use_tangent(node: Node) -> bool:
    """
    Whether the fx node uses tangent input.
    """

    return any(
        is_tangent_node(arg)  # type: ignore[operator]
        for arg in get_args_of_node_type(node)
    )

````
- **EN**: Introduces function `get_args_of_node_type`, function `use_tangent`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_args_of_node_type`、函数`use_tangent`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 29-42 / 第 29-42 行
````python

def compute_tensor_size(*args: Any, count_bytes: bool = True, **kwargs: Any) -> int:
    """
    Compute total tensor sizes from fx.Node in args & kwargs.
    """
    flat_args, _ = tree_flatten((args, kwargs))
    tot = 0
    for arg in flat_args:
        if (fake_tensor := get_fake_tensor_from_node_arg(arg)) is None:
            continue
        tot += fake_tensor.numel() * (fake_tensor.dtype.itemsize if count_bytes else 1)
    return tot


````
- **EN**: Introduces function `compute_tensor_size`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`compute_tensor_size`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 43-56 / 第 43-56 行
````python
def get_fake_tensor_from_node_arg(
    node: torch.fx.node.Argument,
) -> torch.Tensor | None:
    if (
        not hasattr(node, "meta")
        or ("val" not in node.meta)  # type: ignore[union-attr]
        or not isinstance(node.meta["val"], torch.Tensor)  # type: ignore[union-attr]
    ):
        return None
    return node.meta["val"]  # type: ignore[union-attr]


def get_nodes_with_chunking_meta(graph: torch.fx.Graph) -> Sequence[Node]:
    from .core import get_chunking_meta
````
- **EN**: Imports dependencies such as `.core` for the logic in this range. Introduces function `get_fake_tensor_from_node_arg`, function `get_nodes_with_chunking_meta`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `.core` 等依赖，为后续逻辑提供基础能力。这里定义了函数`get_fake_tensor_from_node_arg`、函数`get_nodes_with_chunking_meta`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 57-70 / 第 57-70 行
````python

    output = []
    for node in graph.nodes:
        if get_chunking_meta(node):
            output.append(node)
    return output


def format_node_with_chunking_meta(
    node: torch.fx.Node, include_args: bool = False
) -> None:
    """
    Print the node with chunking metadata for the current node if exists.

````
- **EN**: Introduces function `format_node_with_chunking_meta`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `output`, and `node`.
- **CN**: 这里定义了函数`format_node_with_chunking_meta`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `output`、`node` 等值。

### Lines 71-84 / 第 71-84 行
````python
    If include_args is True, also print chuning metadata for Node arguments.
    """
    from torch._inductor.runtime.runtime_utils import green_text

    from .core import get_chunking_meta

    fake_tensor = get_fake_tensor_from_node_arg(node)
    shape = list(fake_tensor.shape) if fake_tensor is not None else "?"
    print(f"  {shape} {node.format_node()}")

    if meta := get_chunking_meta(node):
        print(f"    {green_text(str(meta))}")

    if include_args:
````
- **EN**: Imports dependencies such as `torch._inductor.runtime.runtime_utils`, and `.core` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fake_tensor`, and `shape`. This range continues the implementation of function `format_node_with_chunking_meta`.
- **CN**: 这里导入了 `torch._inductor.runtime.runtime_utils`、`.core` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `fake_tensor`、`shape` 等值。这一段延续了函数`format_node_with_chunking_meta` 的具体实现。

### Lines 85-98 / 第 85-98 行
````python
        for arg in get_args_of_node_type(node):
            if arg_meta := get_chunking_meta(arg):
                print(f"    {arg}: {green_text(str(arg_meta))}")


def has_any_chunking_meta(*node_list: Node) -> bool:
    from .core import get_chunking_meta

    return any(get_chunking_meta(node) for node in node_list)


def get_first_chunking_meta(*node_list: Node) -> ChunkingMeta | None:
    """
    Get the first non-none chunking metadata if there is any.
````
- **EN**: Imports dependencies such as `.core` for the logic in this range. Introduces function `has_any_chunking_meta`, function `get_first_chunking_meta`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `.core` 等依赖，为后续逻辑提供基础能力。这里定义了函数`has_any_chunking_meta`、函数`get_first_chunking_meta`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 99-112 / 第 99-112 行
````python
    """
    from .core import get_chunking_meta

    for node in node_list:
        if (meta := get_chunking_meta(node)) is not None:
            return meta

    return None


def get_scale_by_from_metas(*metas: ChunkingMeta) -> Node | None:
    """
    If there are multiple ChunkingMeta having the scale_by field,
    raise a CantChunk exception.
````
- **EN**: Imports dependencies such as `.core` for the logic in this range. Introduces function `get_scale_by_from_metas`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `.core` 等依赖，为后续逻辑提供基础能力。这里定义了函数`get_scale_by_from_metas`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 113-126 / 第 113-126 行
````python

    If no ChunkingMeta has scale_by field, return None.
    Other wise return the only scale_by field.
    """

    scale_by_list = []

    # don't do dedup on the scale_by field on purpose for this API
    for meta in metas:
        if meta.scale_by is not None:
            scale_by_list.append(meta.scale_by)

    if len(scale_by_list) > 1:
        raise CantChunk("Multiple scale_by")
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scale_by_list`. This range continues the implementation of function `get_scale_by_from_metas`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `scale_by_list` 等值。这一段延续了函数`get_scale_by_from_metas` 的具体实现。

### Lines 127-140 / 第 127-140 行
````python

    return scale_by_list[0] if len(scale_by_list) == 1 else None


def get_scale_by_from_node(node: Node) -> Node | None:
    from .core import get_chunking_meta

    meta = get_chunking_meta(node)
    return meta.scale_by if meta is not None else None


def get_node_is_scalar(nodes: Sequence[Node]) -> dict[Node, bool]:
    """
    Returns a dict map a node to 'is_scalar'.
````
- **EN**: Imports dependencies such as `.core` for the logic in this range. Introduces function `get_scale_by_from_node`, function `get_node_is_scalar`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `meta`.
- **CN**: 这里导入了 `.core` 等依赖，为后续逻辑提供基础能力。这里定义了函数`get_scale_by_from_node`、函数`get_node_is_scalar`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `meta` 等值。

### Lines 141-154 / 第 141-154 行
````python
    """
    node_is_scalar = {}
    for node in nodes:
        ft = get_fake_tensor_from_node_arg(node)
        assert ft is not None
        node_is_scalar[node] = ft.numel() == 1
    return node_is_scalar


def get_node_ndim(nodes: Sequence[Node]) -> dict[Node, int]:
    """
    Returns a dict map a node to 'ndim'.
    """
    node_ndim = {}
````
- **EN**: Introduces function `get_node_ndim`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node_is_scalar`, `ft`, and `node_ndim`.
- **CN**: 这里定义了函数`get_node_ndim`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `node_is_scalar`、`ft`、`node_ndim` 等值。

### Lines 155-168 / 第 155-168 行
````python
    for node in nodes:
        ft = get_fake_tensor_from_node_arg(node)
        assert ft is not None
        node_ndim[node] = ft.ndim
    return node_ndim


def is_chunked_by_dim(node: Node, dim: int) -> bool:
    from .core import get_chunking_meta

    meta = get_chunking_meta(node)
    return meta is not None and meta.chunked_by_dim(dim)


````
- **EN**: Imports dependencies such as `.core` for the logic in this range. Introduces function `is_chunked_by_dim`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `.core` 等依赖，为后续逻辑提供基础能力。这里定义了函数`is_chunked_by_dim`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 169-182 / 第 169-182 行
````python
def tangent_has_chunking_meta(gm: GraphModule) -> bool:
    from .core import get_chunking_meta

    return any(
        is_tangent_node(node) and get_chunking_meta(node) is not None
        for node in gm.graph.find_nodes(op="placeholder", sort=False)
    )


def get_tangent_nodes(graph: Graph) -> Sequence[Node]:
    tangents = []
    for node in graph.find_nodes(op="placeholder", sort=False):
        if is_tangent_node(node):
            tangents.append(node)
````
- **EN**: Imports dependencies such as `.core` for the logic in this range. Introduces function `tangent_has_chunking_meta`, function `get_tangent_nodes`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `.core` 等依赖，为后续逻辑提供基础能力。这里定义了函数`tangent_has_chunking_meta`、函数`get_tangent_nodes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 183-183 / 第 183-183 行
````python
    return tangents
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `get_tangent_nodes`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`get_tangent_nodes` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `is_tangent_node`, `get_args_of_node_type`, `use_tangent`, `compute_tensor_size`, `get_fake_tensor_from_node_arg`, `get_nodes_with_chunking_meta`, and `...+10`  
  **CN**: 主要函数：`is_tangent_node`、`get_args_of_node_type`、`use_tangent`、`compute_tensor_size`、`get_fake_tensor_from_node_arg`、`get_nodes_with_chunking_meta`、`另有10项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `torch.utils._pytree`, `.common`, `.core`, `torch._inductor.runtime.runtime_utils`
