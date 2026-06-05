# fuser_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/passes/utils/fuser_utils.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects graph-to-graph optimization, rewriting, and analysis passes for FX IR.
- **Purpose (CN)**: 汇集针对 FX 中间表示的图到图优化、改写与分析 passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21
```python
import copy
import heapq

import torch.fx
from torch.fx._compatibility import compatibility
from torch.fx.graph import Graph
from torch.fx.graph_module import GraphModule
from torch.fx.node import Node
from torch.fx.passes.tools_common import legalize_graph, NodeList, NodeSet  # noqa: F401
from torch.fx.passes.utils import lift_subgraph_as_module  # type: ignore[attr-defined]


@compatibility(is_backward_compatible=False)
def topo_sort(nodes: NodeList) -> NodeList:
    # Stable topological sort: among nodes with no dependency between them,
    # preserve their relative order in the input list. This uses a min-heap
    # keyed by original position instead of a FIFO queue.
    indegree_map = dict.fromkeys(nodes, 0)
    position = {node: i for i, node in enumerate(nodes)}
    candidates: list[tuple[int, Node]] = []
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 22-47
```python
    for node in nodes:
        for n in node.all_input_nodes:
            if n in indegree_map:
                indegree_map[node] += 1
        if indegree_map[node] == 0:
            heapq.heappush(candidates, (position[node], node))

    sorted_nodes: NodeList = []
    while candidates:
        _, node = heapq.heappop(candidates)
        sorted_nodes.append(node)

        for n in node.users:
            if n in indegree_map:
                indegree_map[n] -= 1
                if indegree_map[n] == 0:
                    heapq.heappush(candidates, (position[n], n))

    if len(nodes) != len(sorted_nodes):
        raise AssertionError(
            "topological sorted nodes doesn't have same length as input nodes"
        )

    return sorted_nodes
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 48-70
```python
@compatibility(is_backward_compatible=False)
def validate_partition(partition: NodeList) -> bool:
    # verify the partition doesn't form a dependency cycle in the original graph
    # returns True for valid partition, False for invalid

    partition_set = set(partition)

    outputs: NodeList = []
    for node in partition_set:
        for user_node in node.users:
            if user_node not in partition_set:
                # external user node, need to expose as an output
                outputs.append(user_node)

    # Perform BFS on the partition outputs.
    # If it reaches a node within the partition, then it found a cycle.
    # This function takes the ownership of `root_nodes` and may modify it.
    def bfs_find_cycle(root_nodes: NodeList) -> bool:
        # Set used to exclude nodes that have already been visited.
        # If a node has been visited, that node and all its children have
        # been checked for cycles.
        visited: NodeSet = set()
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 71-96
```python
        # Start with `root_nodes` and traverse through (toward child nodes)
        # their connected sub-graph. Nodes in `visited` won't be added
        # to `queue` again.
        queue: NodeList = root_nodes
        while queue:
            current = queue.pop()
            visited.add(current)
            if current in partition_set:
                # Started from partition's `output` nodes, and reached
                # another node in partition. Cycle!
                return True
            for user_node in current.users:
                if user_node in visited:
                    continue
                queue.append(user_node)
        # `root_nodes` don't cause cycle.
        return False

    # Use all output nodes as roots to traverse
    # the graph to check cycles.
    if bfs_find_cycle(outputs):
        return False

    return True
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 97-122
```python
@compatibility(is_backward_compatible=False)
def fuse_as_graphmodule(
    gm: GraphModule,
    nodes: NodeList,
    module_name: str,
    partition_lookup_table: dict[Node, int | None] | None = None,
    *,
    always_return_tuple: bool = False,
) -> tuple[GraphModule, tuple[Node, ...], tuple[Node, ...]]:
    """
    Fuse nodes in graph_module into a GraphModule.

    Args:
        gm (GraphModule): target graph_module

        nodes (List[Node]): list of nodes in `gm` to fuse, where the node must be topologically sorted

        module_name: class name for the fused GraphModule

        partition_lookup_table (Optional[Dict[Node, None]]): optional dict of nodes to speed up lookup

        always_return_tuple (bool): whether to always return a tuple, even if there is only one output

    Returns:
        fused_gm (GraphModule): fused graph module, where its node is a copy of `nodes` in `gm`
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 123-146
```python
        original_inputs (Tuple[Node, ...]): input nodes to `nodes` in original `gm`

        original_outputs (Tuple[Node, ...]): consumer nodes of `nodes` in original `gm`

    """

    # assumption: nodes are already sorted in topo order

    for node in nodes:
        if node.graph.owning_module is not gm:
            raise AssertionError(
                f"{node} doesn't belong to passed in graph module {gm._get_name()}"
            )
        if node._erased:
            raise AssertionError(f"{node} has been removed from owning graph")
        if node not in gm.graph._find_nodes_lookup_table:
            raise AssertionError(
                f"{node} is not found in graph module {gm._get_name()}"
            )

    # validates partition doesn't introduce dependency circles in the graph
    if not validate_partition(nodes):
        raise AssertionError("Invalid partition, found dependency cycles")
```
- **EN**: This block continues `fuse_as_graphmodule` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `fuse_as_graphmodule`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 147-169
```python
    # if no dict of partition nodes is provided, reconstruct it by nodes list to reduce lookup time
    if partition_lookup_table is None:
        partition_lookup_table = dict.fromkeys(nodes)

    subgraph = Graph()

    node_to_placeholder: dict[
        Node, Node
    ] = {}  # mapping of nodes from old graph to placeholder in new graph
    node_map: dict[Node, Node] = {}  # mapping of nodes from old graph to new graph

    # handles inputs through graph.node_copy's arg_transform functions
    def remap_inputs(x: Node) -> Node:
        if x.op == "get_attr":
            # TODO: do we really need copy the get_attr node into the graph?
            # do something here
            pass

        if x in partition_lookup_table:
            # x is inside subgraph, return the copied node
            # the node should have been copied already, as we are copying graph in the topological order
            return node_map[x]
```
- **EN**: Defines the `fuse_as_graphmodule` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`fuse_as_graphmodule` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 170-195
```python
        if x not in node_to_placeholder:
            # x is not in subgraph, create a new placeholder for subgraph
            placeholder_node = subgraph.placeholder(x.name, type_expr=x.type)
            # copy all meta fields, even if some fields might be irrelevant for the placeholder node
            placeholder_node.meta = copy.copy(x.meta)
            node_to_placeholder[x] = placeholder_node

        return node_to_placeholder[x]

    # copy nodes in topological order
    for node in nodes:
        new_node = subgraph.node_copy(node, remap_inputs)
        node_map[node] = new_node

    # handles outputs
    output_mapping: dict[Node, Node] = {}  # mapping from old output to new outputs

    for node in nodes:
        for user_node in node.users:
            if user_node not in partition_lookup_table:
                # external user node, need to expose as an output
                output_mapping[node] = node_map[node]

    # outs contain nodes in the new subgraph
    outs = tuple(output_mapping.values())
```
- **EN**: This block continues `fuse_as_graphmodule` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `fuse_as_graphmodule`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 196-218
```python
    if always_return_tuple:
        # always return a tuple, even if there is only one output
        subgraph.output(outs)
    else:
        # If there's a single output then return it directly, otherwise return a tuple.
        subgraph.output(outs[0] if len(outs) == 1 else outs)

    # lint to ensure correctness
    subgraph.lint()  # type: ignore[no-untyped-call]
    fused_gm: GraphModule
    fused_gm, _ = lift_subgraph_as_module(
        gm, subgraph, comp_name="", class_name=module_name
    )

    # sub_gm's input nodes in the original module
    original_inputs: tuple[Node, ...] = tuple(node_to_placeholder.keys())

    # sub_gm's outputs node in the original module
    original_outputs: tuple[Node, ...] = tuple(output_mapping.keys())

    return fused_gm, original_inputs, original_outputs
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 219-242
```python
@compatibility(is_backward_compatible=False)
def insert_subgm(
    gm: GraphModule,
    sub_gm: GraphModule,
    orig_inputs: tuple[Node, ...],
    orig_outputs: tuple[Node, ...],
    insertion_point: Node | None = None,
) -> GraphModule:
    # add sub_gm into gm
    submodule_name = sub_gm.__class__.__name__
    gm.add_submodule(submodule_name, sub_gm)

    # Use provided insertion point, or fall back to last output node for backwards compat
    if insertion_point is None:
        for node in reversed(gm.graph.nodes):
            if node in orig_outputs:
                insertion_point = node
                break
        if insertion_point is None:
            raise AssertionError(
                "Cannot determine insertion point: no insertion_point provided and "
                "orig_outputs is empty. Pass the last partition node as insertion_point."
            )
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 243-269
```python
    # Create a call_module node in main graph.
    with gm.graph.inserting_after(insertion_point):
        module_node = gm.graph.call_module(
            submodule_name, args=orig_inputs, kwargs=None
        )
        output_node = sub_gm.graph.output_node()

    # Replace uses of original outputs with the fused module outputs.
    # If there are no external outputs, skip replacement (nothing to replace).
    if orig_outputs:
        next_node = module_node.next
        with gm.graph.inserting_before(next_node):
            if len(orig_outputs) == 1 and not isinstance(output_node.args[0], tuple):
                # main_remapping[comp.orig_outputs[0]] = module_node
                orig_outputs[0].replace_all_uses_with(module_node, propagate_meta=True)
            else:
                for i, orig_output in enumerate(orig_outputs):
                    # Use Proxy to record getitem access.
                    proxy_out = torch.fx.Proxy(module_node)[i].node  # type: ignore[index]
                    orig_output.replace_all_uses_with(proxy_out, propagate_meta=True)

                module_node.meta["val"] = tuple(
                    orig_output.meta.get("val", None) for orig_output in orig_outputs
                )
    return gm
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 270-295
```python
@compatibility(is_backward_compatible=False)
def erase_nodes(gm: GraphModule, nodes: NodeList) -> None:
    # erase original nodes in inversed topological order
    for node in reversed(nodes):
        gm.graph.erase_node(node)


@compatibility(is_backward_compatible=False)
def fuse_by_partitions(
    gm: GraphModule,
    partitions: list[dict[Node, int | None]],
    prefix: str = "fused_",
    always_return_tuple: bool = False,
) -> GraphModule:
    for partition_id, partition in enumerate(partitions):
        sorted_nodes = topo_sort(list(partition))

        submodule_name = prefix + str(partition_id)
        sub_gm, orig_inputs, orig_outputs = fuse_as_graphmodule(
            gm,
            sorted_nodes,
            submodule_name,
            partition,
            always_return_tuple=always_return_tuple,
        )
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 296-303
```python
        insert_subgm(gm, sub_gm, orig_inputs, orig_outputs, sorted_nodes[-1])

        erase_nodes(gm, sorted_nodes)

    torch.fx.passes.tools_common.stable_topological_sort(gm)
    gm.graph.lint()

    return gm
```
- **EN**: This block continues `fuse_by_partitions` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `fuse_by_partitions`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch.fx`, `torch.fx._compatibility`, `torch.fx.graph`, `torch.fx.graph_module`, `torch.fx.node`, `torch.fx.passes.tools_common`, `torch.fx.passes.utils`
- **Standard library / 标准库**: `copy`, `heapq`
- **Primary symbols / 核心符号**: `topo_sort`, `validate_partition`, `fuse_as_graphmodule`, `insert_subgm`, `erase_nodes`, `fuse_by_partitions`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
