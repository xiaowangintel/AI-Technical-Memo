# source_matcher_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/passes/utils/source_matcher_utils.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects graph-to-graph optimization, rewriting, and analysis passes for FX IR.
- **Purpose (CN)**: 汇集针对 FX 中间表示的图到图优化、改写与分析 passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
import logging
import os
from collections.abc import Callable
from dataclasses import dataclass, field
from typing import Any

from torch.fx._compatibility import compatibility
from torch.fx.graph import Graph
from torch.fx.node import Node


__all__ = ["get_source_partitions", "check_subgraphs_connected", "SourcePartition"]


# Set`PYTORCH_MATCHER_LOGLEVEL=INFO` to see debug logs
def _init_logger() -> logging.Logger:
    logger = logging.getLogger(__name__)
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 19-33
```python
    level = os.environ.get("PYTORCH_MATCHER_LOGLEVEL", "WARNING").upper()
    logger.setLevel(level)
    console = logging.StreamHandler()
    formatter = logging.Formatter("%(filename)s > %(message)s")
    console.setFormatter(formatter)
    console.setLevel(level)
    # add the handlers to the logger
    logger.addHandler(console)
    logger.propagate = False
    return logger


logger = _init_logger()
```
- **EN**: This module-level block helps initialize learnable tensors and related state. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于初始化可学习张量及相关状态。 同时它还会计算并返回中间值或结果。

### Lines 34-50
```python
@compatibility(is_backward_compatible=False)
@dataclass
class SourcePartition:
    # Nodes in a particular partition
    nodes: list[Node]

    # The source these nodes decomposed from
    source: Any

    # Nodes in the graph that are needed as inputs to the partition
    # These do not include the params of the partition
    input_nodes: list[Node] = field(default_factory=list)

    # Nodes in the partition that are being used by nodes outside of the
    # partition
    output_nodes: list[Node] = field(default_factory=list)
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 51-67
```python
    # Parameters that are being used
    params: list[Node] = field(default_factory=list)


@compatibility(is_backward_compatible=False)  # type: ignore[misc]
def get_source_partitions(
    graph: Graph,
    wanted_sources: list[Any],
    filter_fn: Callable[[Node], bool] | None = None,
) -> dict[Any, list[SourcePartition]]:
    """
    Args:
        graph: The graph we want to partition
        wanted_sources: List of sources of nodes that were decomposed from this
            source. This can be a function (ex. torch.nn.functional.linear) or a
            leaf module type (ex. torch.nn.Linear).
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 68-85
```python
    Returns:
        Dictionary mapping sources that were given to a list of SourcePartitions
        that correspond to the list of nodes that were decomposed from the given
        source.
    """
    modules: dict[type, dict[str, list[Node]]] = {}

    def add_to_partition(src: Any, fqn: str, node: Node) -> None:
        diff_modules = modules.setdefault(src, {})
        partition = diff_modules.setdefault(fqn, [])
        partition.append(node)

    for node in graph.nodes:
        # The metadata source_fn should contain a tuple of a unique name for the
        # source, and the source function if the node is decomposed from a
        # function, or the type of module if the node is decomposed from a leaf
        # module
```
- **EN**: Defines the `get_source_partitions` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`get_source_partitions` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 86-97
```python
        # TODO: Bypass "torch_fn" when "source_fn_stack" because now "torch_fn" can
        # be different from "source_fn_stack", for example for the add_ node
        # decomposed from batch norm. We should remove the check on "source_fn_stack"
        # after we fix "torch_fn". T199561090
        source_fn_st = node.meta.get("source_fn_stack", None)
        if source_fn_st is None:
            matched = False
            torch_fn = node.meta.get("torch_fn", None)
            if torch_fn is not None:
                node_fqn, source_fn = torch_fn
                source_fn_name = source_fn.split(".")[1]
                if source_fn_name in wanted_sources:
```
- **EN**: This block continues `get_source_partitions` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `get_source_partitions`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 98-109
```python
                    add_to_partition(source_fn_name, node_fqn, node)
                    matched = True
            # Fallback: when source_fn_stack is not populated (e.g. strict=False export),
            # use nn_module_stack to resolve the originating module type.
            # Only apply to call_function nodes to avoid incorrectly including
            # placeholder, get_attr, or output nodes in partitions.
            if not matched and node.op == "call_function":
                nn_module_stack = node.meta.get("nn_module_stack", None)
                if nn_module_stack:
                    # Get the innermost module (last entry in the ordered dict)
                    innermost_fqn, innermost_cls = list(nn_module_stack.values())[-1]
                    for src in wanted_sources:
```
- **EN**: This block continues `get_source_partitions` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `get_source_partitions`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 110-124
```python
                        if isinstance(src, type):
                            if isinstance(innermost_cls, type) and issubclass(
                                innermost_cls, src
                            ):
                                add_to_partition(src, innermost_fqn, node)
                                break
                            elif isinstance(innermost_cls, str):
                                src_str = src.__module__ + "." + src.__qualname__
                                if innermost_cls == src_str:
                                    add_to_partition(src, innermost_fqn, node)
                                    break
                        elif innermost_cls == src:
                            add_to_partition(src, innermost_fqn, node)
                            break
```
- **EN**: This block continues `get_source_partitions` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `get_source_partitions`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 125-138
```python
        if source_fn_st is not None:
            source_fn = source_fn_st[-1]
            if source_fn[1] in wanted_sources:
                add_to_partition(source_fn[1], source_fn[0], node)

    def make_partition(nodes: list[Node], module_type: type) -> SourcePartition:
        input_nodes = set()
        output_nodes = set()
        params = set()
        for node in nodes:
            for arg in node.args:
                if isinstance(arg, Node) and arg not in nodes and arg.op != "get_attr":
                    input_nodes.add(arg)
```
- **EN**: Defines the `get_source_partitions` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`get_source_partitions` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 139-155
```python
            if node.op == "get_attr":
                params.add(node)
                # get_attr nodes won't be output nodes
                continue

            for user in node.users:
                if user not in nodes:
                    output_nodes.add(node)

        return SourcePartition(
            nodes,
            module_type,
            list(input_nodes),
            list(output_nodes),
            list(params),  # type: ignore[arg-type]
        )
```
- **EN**: This block continues `get_source_partitions` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `get_source_partitions`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 156-173
```python
    ret: dict[type[Any], list[SourcePartition]] = {}

    if filter_fn:
        # for each partition, we apply filter_fn to filter out all partitions that doesn't satisfy the
        # filter condition
        filtered_modules = {}
        for tp, name_to_partition in modules.items():
            filtered_name_to_partition = {
                name: partition
                for name, partition in name_to_partition.items()
                if all(map(filter_fn, partition))
            }
            filtered_modules[tp] = filtered_name_to_partition
        modules = filtered_modules

    for k, v in modules.items():
        ret[k] = [make_partition(partition, k) for partition in v.values()]
```
- **EN**: This block continues `get_source_partitions` and works to organize reusable module behavior and state. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `get_source_partitions`，用于组织可复用的模块行为与状态。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 174-186
```python
    return ret


@compatibility(is_backward_compatible=False)  # type: ignore[misc]
def check_subgraphs_connected(
    subgraph1: SourcePartition, subgraph2: SourcePartition
) -> bool:
    """
    Given two subgraphs A and B (in the form of a list of nodes), checks if
    A has nodes connecting to at least one node in B -- aka there exists a node
    in B that uses a node in A (not the other way around).
    """
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 187-191
```python
    for node in reversed(subgraph1.nodes):
        for user in node.users:
            if user in subgraph2.nodes:
                return True
    return False
```
- **EN**: This block continues `check_subgraphs_connected` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `check_subgraphs_connected`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch.fx._compatibility`, `torch.fx.graph`, `torch.fx.node`
- **Standard library / 标准库**: `logging`, `os`, `collections.abc`, `dataclasses`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `_init_logger`, `SourcePartition`, `get_source_partitions`, `check_subgraphs_connected`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
