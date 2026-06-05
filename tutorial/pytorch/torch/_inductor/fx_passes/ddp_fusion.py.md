# ddp_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/ddp_fusion.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `CommBlock`. It exposes functions such as `move_block_after`, `move_block_before`, `call_function`, `get_comm_block`, `get_all_comm_blocks`, `_fuse_allreduce_by_concat`, and `...+9`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `CommBlock` 等类。同时提供 `move_block_after`、`move_block_before`、`call_function`、`get_comm_block`、`get_all_comm_blocks`、`_fuse_allreduce_by_concat`、`另有9项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# Owner(s): ["oncall: distributed"]
import collections
import inspect
import logging
import math
import operator
from collections.abc import Callable, Generator
from dataclasses import dataclass
from functools import partial
from typing import Any, cast

import torch
import torch.fx as fx
from torch._dynamo.utils import counters
from torch.fx.passes.graph_transform_observer import GraphTransformObserver
from torch.fx.passes.shape_prop import _extract_tensor_metadata, TensorMetadata
from torch.utils._ordered_set import OrderedSet
from torch.utils._pytree import tree_flatten, tree_map, tree_unflatten

from ..fx_utils import get_fake_args_kwargs
````
- **EN**: Imports dependencies such as `collections`, `inspect`, `logging`, `math`, `operator`, `collections.abc`, and `...+11` for the logic in this range.
- **CN**: 这里导入了 `collections`、`inspect`、`logging`、`math`、`operator`、`collections.abc`、`另有11项` 等依赖，为后续逻辑提供基础能力。

### Lines 21-40 / 第 21-40 行
````python
from ..virtualized import V


aten = torch.ops.aten
logger: logging.Logger = logging.getLogger("comm_fusion")


def move_block_after(block: list[fx.Node], target_node: fx.Node) -> None:
    for node in block:
        target_node.append(node)
        target_node = node


def move_block_before(block: list[fx.Node], target_node: fx.Node) -> None:
    for node in block:
        target_node.prepend(node)
        target_node = node


def call_function(
````
- **EN**: Imports dependencies such as `..virtualized` for the logic in this range. Introduces function `move_block_after`, function `move_block_before`, function `call_function`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `aten`, `logger`, and `target_node`.
- **CN**: 这里导入了 `..virtualized` 等依赖，为后续逻辑提供基础能力。这里定义了函数`move_block_after`、函数`move_block_before`、函数`call_function`。包含分支、循环或上下文管理等控制流。初始化或更新了 `aten`、`logger`、`target_node` 等值。

### Lines 41-60 / 第 41-60 行
````python
    graph: fx.Graph,
    target: str | Callable[..., Any],
    args: tuple[fx.node.Argument, ...] | None = None,
    kwargs: dict[str, fx.node.Argument] | None = None,
) -> fx.Node:
    # We accept target as a str to avoid typing error as the type of
    # a node.target is str | Callable[..., Any].
    # This also allows us to avoid writing check for every call.
    if isinstance(target, str):
        raise RuntimeError(f"Call function should not get a str target {target=}")
    node = graph.call_function(target, args, kwargs)
    _, args, kwargs = get_fake_args_kwargs(node)
    with V.fake_mode:
        node.meta["val"] = target(*args, **kwargs)
        # node.meta["val"] may be a container. So we use tree_map here
        # to recursively extract the tensor metadata.
        node.meta["tensor_meta"] = tree_map(
            _extract_tensor_metadata, (node.meta["val"],)
        )[0]
    return node
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph`, `target`, `args`, `kwargs`, and `node`. This range continues the implementation of function `call_function`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph`、`target`、`args`、`kwargs`、`node` 等值。这一段延续了函数`call_function` 的具体实现。

### Lines 61-80 / 第 61-80 行
````python


@dataclass(unsafe_hash=True)
class CommBlock:
    shape: torch.Size | list[torch.Size]
    node_list: list[fx.Node]
    inputs: list[fx.Node]
    wait_nodes: list[fx.Node]
    comm_node: fx.Node
    outputs: OrderedSet[fx.Node]


def get_comm_block(comm_node: fx.Node) -> CommBlock | None:
    """
    Given a collective node (e.g., allreduce), find out all the nodes belong to
    this communication.

    Args:
        comm_node(fx.Node): The target communication/collective node.
    Returns:
````
- **EN**: Introduces class `CommBlock`, function `get_comm_block`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Initializes or updates values such as `shape`, `node_list`, `inputs`, `wait_nodes`, `comm_node`, `outputs`, and `...+2`.
- **CN**: 这里定义了类`CommBlock`、函数`get_comm_block`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。初始化或更新了 `shape`、`node_list`、`inputs`、`wait_nodes`、`comm_node`、`outputs`、`另有2项` 等值。

### Lines 81-100 / 第 81-100 行
````python
        The CommBlock that encapsulates the related nodes (e.g., wait_node) of
        the given comm_node.
    """
    node_list = []
    wait_nodes = []
    inputs, _ = tree_flatten((comm_node.args, comm_node.kwargs))
    input_nodes = [inp for inp in inputs if isinstance(inp, fx.Node)]
    # If the users of the wait node are following items, we consinder them
    # to be a part of the output.
    intermediate_outputs = ("split", "reshape", "getitem", "detach", "alias")

    first_user = next(iter(comm_node.users))
    if (
        len(comm_node.users) == 1
        and first_user.target is torch.ops._c10d_functional.wait_tensor.default
    ):
        # Collective with only one output
        node_list = [comm_node, first_user]
        wait_nodes.append(first_user)
    elif len(comm_node.users) > 1 and first_user.target is operator.getitem:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node_list`, `wait_nodes`, `input_nodes`, `intermediate_outputs`, and `first_user`. This range continues the implementation of function `get_comm_block`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `node_list`、`wait_nodes`、`input_nodes`、`intermediate_outputs`、`first_user` 等值。这一段延续了函数`get_comm_block` 的具体实现。

### Lines 101-120 / 第 101-120 行
````python
        # Collective with only more than one output
        node_list.append(comm_node)
        for user in comm_node.users:
            if user.target != operator.getitem:
                return None
            if len(user.users) != 1:
                return None
            wait_node = next(iter(user.users))
            if wait_node.target != torch.ops._c10d_functional.wait_tensor.default:
                return None
            wait_nodes.append(wait_node)
            node_list.append(user)
        node_list.extend(wait_nodes)
    else:
        return None

    # Identify all the outputs of this collective block.
    outputs = OrderedSet[fx.Node]()
    nodes = collections.deque(wait_nodes)
    while nodes:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `wait_node`, `else`, `outputs`, and `nodes`. This range continues the implementation of function `get_comm_block`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `wait_node`、`else`、`outputs`、`nodes` 等值。这一段延续了函数`get_comm_block` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
        node = nodes.popleft()
        for user in node.users:
            if isinstance(user, fx.Node) and user.name.startswith(intermediate_outputs):
                nodes.append(user)
                node_list.append(user)
            else:
                outputs.add(node)
                break

    tensor_meta = input_nodes[0].meta["tensor_meta"]
    shape: torch.Size | list[torch.Size]
    if isinstance(tensor_meta, TensorMetadata):
        shape = tensor_meta.shape
    elif isinstance(tensor_meta, (list, tuple)):
        shape = [tm.shape for tm in tensor_meta]
    else:
        logger.warning("Unexpected type of tensor_meta %s", type(tensor_meta))
        return None

    return CommBlock(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node`, `else`, `tensor_meta`, and `shape`. This range continues the implementation of function `get_comm_block`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `node`、`else`、`tensor_meta`、`shape` 等值。这一段延续了函数`get_comm_block` 的具体实现。

### Lines 141-160 / 第 141-160 行
````python
        shape=shape,
        node_list=node_list,
        wait_nodes=wait_nodes,
        comm_node=comm_node,
        inputs=input_nodes,
        outputs=outputs,
    )


def get_all_comm_blocks(
    graph: fx.Graph,
    comm_ops: tuple[torch._ops.OpOverload, ...],
    comm_filter: Callable[..., bool] | None = None,
) -> list[CommBlock]:
    if comm_filter is None:

        def always_true(comm_block: CommBlock) -> bool:
            return True

        comm_filter = always_true
````
- **EN**: Introduces function `get_all_comm_blocks`, function `always_true`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `shape`, `node_list`, `wait_nodes`, `comm_node`, `inputs`, `outputs`, and `...+3`.
- **CN**: 这里定义了函数`get_all_comm_blocks`、函数`always_true`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `shape`、`node_list`、`wait_nodes`、`comm_node`、`inputs`、`outputs`、`另有3项` 等值。

### Lines 161-180 / 第 161-180 行
````python

    blocks = []
    for node in graph.nodes:
        if node.target not in comm_ops:
            continue
        comm_block = get_comm_block(node)
        if comm_block is not None and comm_filter(comm_block):
            blocks.append(comm_block)
    return blocks


def _fuse_allreduce_by_concat(
    graph: fx.Graph,
    last_input_node: fx.Node,
    all_input_nodes: list[fx.Node],
    last_comm_block: CommBlock,
) -> CommBlock:
    """Given a list of inputs in order, create a fused allreduce using concat."""
    # Flatten all the inputs to the all_reduce nodes.
    with graph.inserting_after(last_input_node):
````
- **EN**: Introduces function `_fuse_allreduce_by_concat`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `blocks`, `comm_block`, `graph`, `last_input_node`, `all_input_nodes`, and `last_comm_block`.
- **CN**: 这里定义了函数`_fuse_allreduce_by_concat`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `blocks`、`comm_block`、`graph`、`last_input_node`、`all_input_nodes`、`last_comm_block` 等值。

### Lines 181-200 / 第 181-200 行
````python
        cat_inputs = []
        for input_node in all_input_nodes:
            assert isinstance(input_node.args[0], fx.Node)
            input_node = input_node.args[0]
            cat_inputs.append(
                call_function(graph, aten.flatten.using_ints, (input_node,))
            )

    # Concat all the flattened nodes.
    with graph.inserting_after(cat_inputs[0]):
        cat_node = call_function(graph, aten.cat, (cat_inputs,))

    # Insert the fused div node and remove the input div nodes.
    # This is an optimization and is not mandatory for fusion.
    divisors = [div.args[1] for div in all_input_nodes]
    assert all(divisor == divisors[0] for divisor in divisors)
    with graph.inserting_after(cat_node):
        div_node = call_function(graph, last_input_node.target, (cat_node, divisors[0]))

    # Create a new Comm/all_reduce node.
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `cat_inputs`, `input_node`, `cat_node`, `divisors`, and `div_node`. This range continues the implementation of function `_fuse_allreduce_by_concat`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `cat_inputs`、`input_node`、`cat_node`、`divisors`、`div_node` 等值。这一段延续了函数`_fuse_allreduce_by_concat` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python
    last_comm_node = last_comm_block.comm_node
    last_wait_node = last_comm_block.wait_nodes[0]
    with graph.inserting_after(div_node):
        flatten_args, spec = tree_flatten((last_comm_node.args, last_comm_node.kwargs))
        flatten_args[0] = div_node
        args, kwargs = tree_unflatten(flatten_args, spec)
        fused_comm_node = call_function(graph, last_comm_node.target, args, kwargs)

    # Create a new Wait node.
    with graph.inserting_after(fused_comm_node):
        flatten_args, spec = tree_flatten((last_wait_node.args, last_wait_node.kwargs))
        flatten_args[0] = fused_comm_node
        args, kwargs = tree_unflatten(flatten_args, spec)
        fused_wait_node = call_function(graph, last_wait_node.target, args, kwargs)

    # Move the fused all_reduce and its args to right after the input node
    nodes_to_move = cat_inputs + [cat_node, div_node, fused_comm_node, fused_wait_node]
    # pyrefly: ignore [bad-argument-type]
    move_block_after(nodes_to_move, last_input_node)

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `last_comm_node`, `last_wait_node`, `fused_comm_node`, `fused_wait_node`, and `nodes_to_move`. This range continues the implementation of function `_fuse_allreduce_by_concat`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `last_comm_node`、`last_wait_node`、`fused_comm_node`、`fused_wait_node`、`nodes_to_move` 等值。这一段延续了函数`_fuse_allreduce_by_concat` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
    return CommBlock(
        shape=cast(TensorMetadata, cat_node.meta.get("tensor_meta")).shape,
        node_list=[fused_comm_node, fused_wait_node],
        wait_nodes=[fused_wait_node],
        comm_node=fused_comm_node,
        inputs=[div_node],
        outputs=OrderedSet([fused_wait_node]),
    )


def _fuse_with_coalesced_op(
    graph: fx.Graph,
    last_input_node: fx.Node,
    all_input_nodes: list[fx.Node],
    last_comm_block: CommBlock,
) -> CommBlock:
    """Given a list of inputs in order, create a fused allreduce by coalesced."""
    last_comm_node = last_comm_block.comm_node
    last_wait_node = last_comm_block.wait_nodes[0]

````
- **EN**: Introduces function `_fuse_with_coalesced_op`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `shape`, `node_list`, `wait_nodes`, `comm_node`, `inputs`, `outputs`, and `...+6`.
- **CN**: 这里定义了函数`_fuse_with_coalesced_op`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `shape`、`node_list`、`wait_nodes`、`comm_node`、`inputs`、`outputs`、`另有6项` 等值。

### Lines 241-260 / 第 241-260 行
````python
    # Insert the fused div node and remove the input div nodes.
    # This is an optimization and is not mandatory for fusion.
    dividends = [div.args[0] for div in all_input_nodes]
    divisors = [div.args[1] for div in all_input_nodes]
    assert all(divisor == divisors[0] for divisor in divisors)
    with graph.inserting_before(last_input_node):
        last_input_node = call_function(
            graph, aten._foreach_div.Scalar, (dividends, divisors[0])
        )
    input_node = last_input_node

    # Create a new Comm/all_reduce_coalesced node.
    with graph.inserting_after(last_comm_node):
        flatten_args, spec = tree_flatten((last_comm_node.args, last_comm_node.kwargs))
        flatten_args[0] = input_node
        args, kwargs = tree_unflatten(flatten_args, spec)
        fused_comm_node = call_function(
            graph, torch.ops._c10d_functional.all_reduce_coalesced.default, args, kwargs
        )

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dividends`, `divisors`, `last_input_node`, `input_node`, and `fused_comm_node`. This range continues the implementation of function `_fuse_with_coalesced_op`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dividends`、`divisors`、`last_input_node`、`input_node`、`fused_comm_node` 等值。这一段延续了函数`_fuse_with_coalesced_op` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python
    # Create a new wait node.
    getitem_nodes = []
    wait_nodes = []
    flatten_args, spec = tree_flatten((last_wait_node.args, last_wait_node.kwargs))
    for idx in range(len(all_input_nodes)):
        with graph.inserting_after(fused_comm_node):
            gi_node = call_function(graph, operator.getitem, (fused_comm_node, idx))
        getitem_nodes.append(gi_node)
        flatten_args[0] = gi_node
        args, kwargs = tree_unflatten(flatten_args, spec)
        with graph.inserting_after(gi_node):
            wait_nodes.append(call_function(graph, last_wait_node.target, args, kwargs))

    # Move the new all_reduce_coalesced and its args to right after the input node
    nodes_to_move = [fused_comm_node] + getitem_nodes + wait_nodes
    move_block_after(nodes_to_move, last_input_node)

    return CommBlock(
        shape=[
            tm.shape
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `getitem_nodes`, `wait_nodes`, `gi_node`, `nodes_to_move`, and `shape`. This range continues the implementation of function `_fuse_with_coalesced_op`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `getitem_nodes`、`wait_nodes`、`gi_node`、`nodes_to_move`、`shape` 等值。这一段延续了函数`_fuse_with_coalesced_op` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
            for tm in cast(
                list[TensorMetadata], fused_comm_node.meta.get("tensor_meta")
            )
        ],
        node_list=[fused_comm_node] + getitem_nodes + wait_nodes,
        wait_nodes=wait_nodes,
        comm_node=fused_comm_node,
        inputs=[input_node],
        outputs=OrderedSet(wait_nodes),
    )


def _scatter_fused_allreduce_waits(
    graph: fx.Graph,
    fused_comm_block: CommBlock,
    orig_comm_blocks: list[CommBlock],
    node_indices: dict[fx.Node, int],
    split_and_reshape: bool = True,
) -> None:
    """
````
- **EN**: Introduces function `_scatter_fused_allreduce_waits`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node_list`, `wait_nodes`, `comm_node`, `inputs`, `outputs`, `graph`, and `...+4`.
- **CN**: 这里定义了函数`_scatter_fused_allreduce_waits`。包含分支、循环或上下文管理等控制流。初始化或更新了 `node_list`、`wait_nodes`、`comm_node`、`inputs`、`outputs`、`graph`、`另有4项` 等值。

### Lines 301-320 / 第 301-320 行
````python
    Scatters the result of the fused communication node to the original users.
    If the fused method is concat splitting the output and reshape will be inserted,
    before inserting getitem. Otherwise getitem will be used as the users of the
    wait node.
    """

    # Before we mass up the order, we need to get the index of the last wait node
    # in orig_comm_blocks. This index will be later used to determine what users
    # nodes need to be move to maintain a correct topological sort order.
    last_wait_node_idx = 0

    for node in graph.nodes:
        last_wait_node_idx = max(
            node_indices.get(node, last_wait_node_idx), last_wait_node_idx
        )
        if node == orig_comm_blocks[-1].wait_nodes[0]:
            break

    if split_and_reshape:
        fused_wait_node = fused_comm_block.wait_nodes[0]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `last_wait_node_idx`, and `fused_wait_node`. This range continues the implementation of function `_scatter_fused_allreduce_waits`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `last_wait_node_idx`、`fused_wait_node` 等值。这一段延续了函数`_scatter_fused_allreduce_waits` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python
        with graph.inserting_after(fused_wait_node):
            split_node = call_function(
                graph,
                aten.split,
                (
                    fused_wait_node,
                    [math.prod(cast(list[int], cb.shape)) for cb in orig_comm_blocks],
                ),
            )
        with graph.inserting_after(split_node):
            fused_outputs = []
            for idx, comm_block in enumerate(orig_comm_blocks):
                split_idx_node = call_function(
                    graph, operator.getitem, (split_node, idx)
                )
                with graph.inserting_after(split_idx_node):
                    fused_outputs.append(
                        call_function(
                            graph, aten.reshape, (split_idx_node, comm_block.shape)
                        )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `split_node`, `fused_outputs`, and `split_idx_node`. This range continues the implementation of function `_scatter_fused_allreduce_waits`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `split_node`、`fused_outputs`、`split_idx_node` 等值。这一段延续了函数`_scatter_fused_allreduce_waits` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python
                    )
    else:
        fused_outputs = fused_comm_block.wait_nodes

    # Scatter the fused outputs.
    incorrect_order_nodes = []
    for comm_block, fused_output in zip(orig_comm_blocks, fused_outputs):
        # Some descendant users of the orig_comm_blocks may be scheduled before
        # the fused all_reduce. For example, the user nodes of the very first
        # all_reduce may be scheduled before the second all_reduce. Since the
        # fused all_reduce is inserted right after the last all_reduce, the
        # order can be wrong.
        # `incorrect_order_nodes` records these nodes.

        orig_wait = comm_block.wait_nodes[0]
        nodes = collections.deque(list(orig_wait.users))
        while nodes:
            user_node = nodes.popleft()
            if not isinstance(user_node, fx.Node):
                continue
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `fused_outputs`, `incorrect_order_nodes`, `orig_wait`, `nodes`, and `user_node`. This range continues the implementation of function `_scatter_fused_allreduce_waits`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`fused_outputs`、`incorrect_order_nodes`、`orig_wait`、`nodes`、`user_node` 等值。这一段延续了函数`_scatter_fused_allreduce_waits` 的具体实现。

### Lines 361-380 / 第 361-380 行
````python

            if node_indices[user_node] < last_wait_node_idx:
                incorrect_order_nodes.append(user_node)
                nodes.extend(list(user_node.users))

        orig_wait.replace_all_uses_with(fused_output)

    last_fused_result = fused_outputs[0]
    fused_outputs_set = OrderedSet(fused_outputs)
    for node in graph.nodes:
        if node in fused_outputs_set:
            last_fused_result = node

    # Move the incorrect_order_nodes to right after the last fused_result.
    incorrect_order_nodes = sorted(
        incorrect_order_nodes, key=lambda node: node_indices[node]
    )
    move_block_after(incorrect_order_nodes, last_fused_result)


````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `last_fused_result`, `fused_outputs_set`, and `incorrect_order_nodes`. This range continues the implementation of function `_scatter_fused_allreduce_waits`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `last_fused_result`、`fused_outputs_set`、`incorrect_order_nodes` 等值。这一段延续了函数`_scatter_fused_allreduce_waits` 的具体实现。

### Lines 381-400 / 第 381-400 行
````python
def _fuse_allreduce(
    graph: fx.Graph,
    comm_blocks: list[CommBlock],
    node_indices: dict[fx.Node, int],
    use_concat: bool,
) -> CommBlock:
    """Given a list of allreduce CommBlock, fuse the CommBlocks into one CommBlock."""

    if len(comm_blocks) == 1:
        return comm_blocks[0]

    # Find the last input node of all the CommBlocks. This node will be served
    # as the inserting point of the new collective op.
    last_input_node = comm_blocks[0].inputs[0]
    last_input_index = -1
    all_input_nodes = []
    for comm_block in comm_blocks:
        input_node = comm_block.inputs[0]
        all_input_nodes.append(input_node)
        index = node_indices[input_node]
````
- **EN**: Introduces function `_fuse_allreduce`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph`, `comm_blocks`, `node_indices`, `use_concat`, `last_input_node`, `last_input_index`, and `...+3`.
- **CN**: 这里定义了函数`_fuse_allreduce`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph`、`comm_blocks`、`node_indices`、`use_concat`、`last_input_node`、`last_input_index`、`另有3项` 等值。

### Lines 401-420 / 第 401-420 行
````python
        if index >= last_input_index:
            assert index != last_input_index
            last_input_node = input_node
            last_input_index = index

    if use_concat:
        fused_comm_block = _fuse_allreduce_by_concat(
            graph, last_input_node, all_input_nodes, comm_blocks[-1]
        )
    else:
        fused_comm_block = _fuse_with_coalesced_op(
            graph, last_input_node, all_input_nodes, comm_blocks[-1]
        )

    _scatter_fused_allreduce_waits(
        graph, fused_comm_block, comm_blocks, node_indices, split_and_reshape=use_concat
    )

    for comm_block in comm_blocks:
        for wait in comm_block.wait_nodes:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `last_input_node`, `last_input_index`, `fused_comm_block`, and `else`. This range continues the implementation of function `_fuse_allreduce`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `last_input_node`、`last_input_index`、`fused_comm_block`、`else` 等值。这一段延续了函数`_fuse_allreduce` 的具体实现。

### Lines 421-440 / 第 421-440 行
````python
            graph.erase_node(wait)
        graph.erase_node(comm_block.comm_node)
    graph.eliminate_dead_code()

    return fused_comm_block


def _bucket_size_fusion(
    graph: fx.Graph, comm_blocks: list[CommBlock], bucket_size_mb: int
) -> Generator[list[CommBlock], None, None]:
    MB = 1024**2
    bucket_size = 1 * MB
    bucket_cap_size = bucket_size_mb * MB
    curr_size = 0
    curr_blocks = []

    count = 0
    fuse_count = 0
    for i, block in enumerate(comm_blocks):
        curr_blocks.append(block)
````
- **EN**: Introduces function `_bucket_size_fusion`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph`, `MB`, `bucket_size`, `bucket_cap_size`, `curr_size`, `curr_blocks`, and `...+2`.
- **CN**: 这里定义了函数`_bucket_size_fusion`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph`、`MB`、`bucket_size`、`bucket_cap_size`、`curr_size`、`curr_blocks`、`另有2项` 等值。

### Lines 441-460 / 第 441-460 行
````python
        itemsize = block.comm_node.meta["tensor_meta"].dtype.itemsize
        curr_size += cast(torch.Size, block.shape).numel() * itemsize
        count += 1
        if curr_size < bucket_size and i != len(comm_blocks) - 1:
            continue

        fuse_count += 1
        if torch.distributed.get_rank() == 0:
            logger.info(
                "DDP bucketing: block%d, count=%d, curr_size=%d, bucket_size=%d",
                fuse_count,
                count,
                curr_size,
                bucket_size,
            )

        # Set the debug counters
        counters["inductor"]["ddp_buckets"] = fuse_count
        yield curr_blocks

````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `itemsize`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `itemsize` 等值。

### Lines 461-480 / 第 461-480 行
````python
        bucket_size = bucket_cap_size
        curr_blocks = []
        curr_size = 0
        count = 0


def _fuse_ddp_communication(
    graph: fx.Graph, algorithm_fn: Callable[..., Any], fusion_fn: Callable[..., Any]
) -> None:
    for output in reversed(graph.nodes):
        if output.op == "output":
            break

    def ddp_reducer_filter(block: CommBlock) -> bool:
        if (
            not isinstance(block.comm_node.args[0], fx.Node)
            or block.comm_node.args[0].target != aten.div.Tensor
        ):
            return False

````
- **EN**: Introduces function `_fuse_ddp_communication`, function `ddp_reducer_filter`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bucket_size`, `curr_blocks`, `curr_size`, `count`, and `graph`.
- **CN**: 这里定义了函数`_fuse_ddp_communication`、函数`ddp_reducer_filter`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `bucket_size`、`curr_blocks`、`curr_size`、`count`、`graph` 等值。

### Lines 481-500 / 第 481-500 行
````python
        if len(block.wait_nodes[0].users) != 1:
            # gradient/wait node should only be used by one user
            return False

        # Two cases:
        # 1. gradient/wait node should be directly used by the output
        # if gradient is None before bwd.
        # 2. gradient/wait node should be directly used by copy_.
        if (
            output not in block.wait_nodes[0].users
            and next(iter(block.wait_nodes[0].users)).target != aten.copy_.default
        ):
            return False

        return True

    ops = (
        torch.ops._c10d_functional.all_reduce_.default,
        torch.ops._c10d_functional.all_reduce.default,
    )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ops`. This range continues the implementation of function `_fuse_ddp_communication.ddp_reducer_filter`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ops` 等值。这一段延续了函数`_fuse_ddp_communication.ddp_reducer_filter` 的具体实现。

### Lines 501-520 / 第 501-520 行
````python
    comm_blocks = get_all_comm_blocks(graph, ops, comm_filter=ddp_reducer_filter)
    node_indices = {node: i for i, node in enumerate(graph.nodes)}

    for block in algorithm_fn(graph, comm_blocks):
        fusion_fn(graph, block, node_indices)


def fuse_ddp_with_coalesced_op(graph: fx.Graph, bucket_size_mb: int) -> None:
    _fuse_ddp_communication(
        graph,
        partial(_bucket_size_fusion, bucket_size_mb=bucket_size_mb),
        partial(_fuse_allreduce, use_concat=False),
    )


def fuse_ddp_with_concat_op(graph: fx.Graph, bucket_size_mb: int) -> None:
    _fuse_ddp_communication(
        graph,
        partial(_bucket_size_fusion, bucket_size_mb=bucket_size_mb),
        partial(_fuse_allreduce, use_concat=True),
````
- **EN**: Introduces function `fuse_ddp_with_coalesced_op`, function `fuse_ddp_with_concat_op`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `comm_blocks`, and `node_indices`.
- **CN**: 这里定义了函数`fuse_ddp_with_coalesced_op`、函数`fuse_ddp_with_concat_op`。包含分支、循环或上下文管理等控制流。初始化或更新了 `comm_blocks`、`node_indices` 等值。

### Lines 521-540 / 第 521-540 行
````python
    )


def schedule_comm_wait(graph: fx.Graph) -> None:
    """
    Delay the execution of wait tensors of allreduce until its first user.

    This algorithm considers the intermediate users, like split, getitem,
    of the wait node and schedule those intermediate users as well.
    This will result in a better overlapping result.
    """
    ops = (
        torch.ops._c10d_functional.all_reduce_.default,
        torch.ops._c10d_functional.all_reduce.default,
        torch.ops._c10d_functional.all_reduce_coalesced.default,
        torch.ops._c10d_functional.all_reduce_coalesced_.default,
    )
    comm_blocks = get_all_comm_blocks(graph, ops)
    if not comm_blocks:
        return
````
- **EN**: Introduces function `schedule_comm_wait`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ops`, and `comm_blocks`.
- **CN**: 这里定义了函数`schedule_comm_wait`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ops`、`comm_blocks` 等值。

### Lines 541-560 / 第 541-560 行
````python

    # Find all the end users.
    allreduce_users = OrderedSet[fx.Node]()
    for allreduce in comm_blocks:
        for output in allreduce.outputs:
            allreduce_users.update(output.users)

    node_indices = {node: i for i, node in enumerate(graph.nodes)}
    for allreduce in comm_blocks:
        # Find the earliest/first user -- target_node.
        assert len(allreduce.outputs) >= 1, (
            f"Found a allreduce that has zero outputs/users -- {allreduce}."
        )
        # Initialize the target node to avoid typing issues.
        target_node = next(iter(next(iter(allreduce.outputs)).users))
        target_node_index = 2**31
        for user in (user for output in allreduce.outputs for user in output.users):
            index = node_indices[user]
            if index < target_node_index:
                target_node = user
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `allreduce_users`, `node_indices`, `target_node`, `target_node_index`, and `index`. This range continues the implementation of function `schedule_comm_wait`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `allreduce_users`、`node_indices`、`target_node`、`target_node_index`、`index` 等值。这一段延续了函数`schedule_comm_wait` 的具体实现。

### Lines 561-580 / 第 561-580 行
````python
                target_node_index = index

        # Move wait nodes and all the subsequent nodes in the comm_block to
        # before the first user -- target_node.
        wait_idx = -1
        for wait_idx, node in enumerate(allreduce.node_list):
            if node == allreduce.wait_nodes[0]:
                break
        assert wait_idx >= 0
        move_block_before(allreduce.node_list[wait_idx:], target_node)


def fuse_ddp_communication(
    graph: fx.Graph, passes: list[Callable[..., None] | str], bucket_size_mb: int
) -> None:
    for i, pa in enumerate(passes):
        with GraphTransformObserver(
            graph.owning_module, f"fuse_ddp_communication_pass_{i}"
        ):
            if isinstance(pa, str):
````
- **EN**: Introduces function `fuse_ddp_communication`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `target_node_index`, `wait_idx`, and `graph`.
- **CN**: 这里定义了函数`fuse_ddp_communication`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `target_node_index`、`wait_idx`、`graph` 等值。

### Lines 581-589 / 第 581-589 行
````python
                func = globals()[pa]
            else:
                func = pa
            if "bucket_size_mb" in OrderedSet(
                v.name for v in inspect.signature(func).parameters.values()
            ):
                func(graph, bucket_size_mb=bucket_size_mb)
            else:
                func(graph)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `func`, and `else`. This range continues the implementation of function `fuse_ddp_communication`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `func`、`else` 等值。这一段延续了函数`fuse_ddp_communication` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `CommBlock`  
  **CN**: 主要类：`CommBlock`
- **EN**: Primary functions: `move_block_after`, `move_block_before`, `call_function`, `get_comm_block`, `get_all_comm_blocks`, `_fuse_allreduce_by_concat`, and `...+9`  
  **CN**: 主要函数：`move_block_after`、`move_block_before`、`call_function`、`get_comm_block`、`get_all_comm_blocks`、`_fuse_allreduce_by_concat`、`另有9项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections`, `inspect`, `logging`, `math`, `operator`, `collections.abc`, `dataclasses`, `functools`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `torch._dynamo.utils`, `torch.fx.passes.graph_transform_observer`, `torch.fx.passes.shape_prop`, `torch.utils._ordered_set`, `torch.utils._pytree`, `..fx_utils`, `..virtualized`
