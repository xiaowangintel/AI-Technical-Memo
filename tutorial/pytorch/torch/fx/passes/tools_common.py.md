# tools_common.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/passes/tools_common.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects graph-to-graph optimization, rewriting, and analysis passes for FX IR.
- **Purpose (CN)**: 汇集针对 FX 中间表示的图到图优化、改写与分析 passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22
```python
import collections
import heapq
import operator
from collections.abc import Mapping
from dataclasses import dataclass
from typing import Any

import torch
import torch.fx
from torch.fx._compatibility import compatibility
from torch.fx.node import _get_qualified_name


__all__ = [
    "get_acc_ops_name",
    "get_node_target",
    "is_node_output_tensor",
    "FxNetAccFusionsFinder",
    "legalize_graph",
    "stable_topological_sort",
]
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 23-43
```python
Tensors = tuple[torch.Tensor] | list[torch.Tensor]
TensorOrTensors = torch.Tensor | Tensors
NodeList = list[torch.fx.Node]
NodeSet = set[torch.fx.Node]
Names = list[str]
CALLABLE_NODE_OPS = {"call_module", "call_function", "call_method"}


@compatibility(is_backward_compatible=False)
def get_acc_ops_name(k: str | type) -> str:
    if isinstance(k, str):
        return k
    elif k.__module__ and "acc_ops" in k.__module__:
        return f"acc_ops.{k.__name__}"
    else:
        module = k.__module__.replace(
            "torch._ops", "torch.ops"
        )  # WAR for bug in how torch.ops assigns module
        return f"{module if module else ''}.{k.__name__}"
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 44-68
```python
@compatibility(is_backward_compatible=False)
def get_node_target(
    submodules: Mapping[str, torch.nn.Module], node: torch.fx.Node
) -> str:
    """
    Given a `node` returns its target typename.

    For "call_method" node, return node.target which is the name of that method being called.
    This could potential lead to conflict but should be okay because normally it's on a tensor.

    For "call_function" node, return typename of node.target.

    For "call_module" node, return typename of the module that node.target point to.

    If seeing "_VariableFunctionsClass" in the target name string, it will be replaced by
    "torch". e.g. _VariableFunctionsClass.relu would become torch.relu.
    """

    if node.op not in CALLABLE_NODE_OPS:
        raise AssertionError(
            "Expect op types of "
            + ", ".join(CALLABLE_NODE_OPS)
            + f", but found {node.op}"
        )
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 69-91
```python
    if node.op == "call_module":
        if not isinstance(node.target, str):
            raise AssertionError(f"Expected str target, got {type(node.target)}")
        submod = submodules[node.target]
        submod_type = getattr(submod, "_base_class_origin", type(submod))
        return get_acc_ops_name(submod_type)
    elif node.op == "call_function":
        target: Any = node.target
        return (
            f"acc_ops.{target.__name__}"
            if target.__module__ is not None and "acc_ops" in target.__module__
            else _get_qualified_name(target)
        )
    else:
        if not isinstance(node.target, str):
            raise AssertionError(f"Expected str target, got {type(node.target)}")
        return node.target


@compatibility(is_backward_compatible=False)
def is_node_output_tensor(node: torch.fx.Node) -> bool:
    """Checks if the node output produces a Tensor or not.
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 92-117
```python
    NOTE: This requires to run `ShapeProp` on the containing fx graph before
    calling this function. This is because it works by checking the `type`
    metadata on the node. This metadata is produced by the `ShapeProp`.
    """
    type_ = node.meta.get("type", None)
    return type_ is not None and issubclass(type_, torch.Tensor)


@compatibility(is_backward_compatible=False)
class FxNetAccFusionsFinder:
    """
    Finds groups of connected ACC nodes that pass non-tensor data between each other.
    Such groups are called fusion groups.
    """

    def __init__(self, module: torch.fx.GraphModule, acc_nodes: NodeSet) -> None:
        self.module = module
        self.nodes = list(module.graph.nodes)
        self.acc_nodes = acc_nodes
        self.node_index = {node: i for i, node in enumerate(self.nodes)}

    @dataclass
    class FusionGroup:
        # The smallest idx of nodes in the fusion group after topological sorting all the nodes in the model.
        top_node_idx: int
```
- **EN**: These decorators register or transform the following definition so it can track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够跟踪符号形状约束以及依赖形状的推理。

### Lines 118-144
```python
        # Nodes in this fusion group.
        nodes: NodeSet

        # Inputs to this fusion group.
        inputs: NodeSet

        # Nodes that in the fusion group that haven't been processed yet.
        nodes_need_process: NodeSet

        def add_node(self, node: torch.fx.Node) -> None:
            """
            Add a node to fusion group.
            """
            if node in self.nodes:
                return

            self.nodes_need_process.add(node)
            self.nodes.add(node)
            self.inputs.discard(node)
            self.inputs.update(
                {
                    n
                    for n in node.all_input_nodes
                    if n.op in CALLABLE_NODE_OPS and n not in self.nodes
                }
            )
```
- **EN**: Declares `FxNetAccFusionsFinder`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `FxNetAccFusionsFinder`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 145-170
```python
    def recursive_add_node(
        self,
        fusion_group: "FxNetAccFusionsFinder.FusionGroup",
        inputs: NodeSet | NodeList,
        visited: NodeSet | None = None,
    ) -> bool:
        """
        Start from inputs and going reverse topological order. If any upstream node
        is in the fusion group, add all the nodes in this path to fusion group.
        """
        for arg in inputs:
            # skip the node if already seen
            if visited is not None:
                if arg in visited:
                    continue
                visited.add(arg)

            # Skip placeholder and get_attr because they won't be in the fusion group.
            if arg.op not in CALLABLE_NODE_OPS:
                continue

            # If the node has smaller idx, it's already an upstream node of the fusion
            # group. We don't need to check it anymore.
            if self.node_index[arg] < fusion_group.top_node_idx:
                continue
```
- **EN**: Defines the `FxNetAccFusionsFinder.recursive_add_node` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`FxNetAccFusionsFinder.recursive_add_node` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 171-196
```python
            # If the node is in the fusion group, return True.
            if arg in fusion_group.nodes:
                return True

            # Check the upstream nodes of the node, if any of them is in the fusion group
            # we'll add this node to fusion group and return True.
            if self.recursive_add_node(fusion_group, arg.all_input_nodes, visited):
                fusion_group.add_node(arg)
                return True

        return False

    def __call__(self) -> dict[torch.fx.Node, NodeSet]:
        result: dict[torch.fx.Node, NodeSet] = {}
        acc_nodes = list(self.acc_nodes)

        for node in acc_nodes:
            if node in result:
                continue
            if node.op not in CALLABLE_NODE_OPS:
                continue
            if "tensor_meta" in node.meta:
                continue
            if node not in self.acc_nodes:
                continue
```
- **EN**: Declares `FxNetAccFusionsFinder`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `FxNetAccFusionsFinder`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 197-218
```python
            fusion_group: FxNetAccFusionsFinder.FusionGroup = self.FusionGroup(
                top_node_idx=self.node_index[node],
                nodes={node},
                inputs=set(node.all_input_nodes),
                nodes_need_process={node},
            )
            while fusion_group.nodes_need_process:
                node = fusion_group.nodes_need_process.pop()
                self.recursive_add_node(
                    fusion_group,
                    fusion_group.inputs,
                    visited=set(),
                )

                # Optionally add downstream nodes
                if "tensor_meta" not in node.meta:
                    for user in node.users:
                        if user.op not in CALLABLE_NODE_OPS:
                            continue
                        if user in fusion_group.nodes:
                            continue
```
- **EN**: This block continues `FxNetAccFusionsFinder.__call__` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `FxNetAccFusionsFinder.__call__`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 219-244
```python
                        fusion_group.add_node(user)
                        self.recursive_add_node(
                            fusion_group,
                            fusion_group.inputs,
                            visited=set(),
                        )

                # Add some upstream nodes
                for arg in node.all_input_nodes:
                    if arg.op not in CALLABLE_NODE_OPS:
                        continue
                    if "tensor_meta" in arg.meta:
                        continue
                    if arg in fusion_group.nodes:
                        continue

                    fusion_group.add_node(arg)
                    fusion_group.top_node_idx = min(
                        fusion_group.top_node_idx, self.node_index[arg]
                    )
                    self.recursive_add_node(
                        fusion_group,
                        fusion_group.inputs,
                        visited=set(),
                    )
```
- **EN**: This block continues `FxNetAccFusionsFinder.__call__` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `FxNetAccFusionsFinder.__call__`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 245-268
```python
            if not (set(fusion_group.nodes) <= self.acc_nodes):
                self.acc_nodes -= fusion_group.nodes
            else:
                for n in fusion_group.nodes:
                    result[n] = fusion_group.nodes

        return result


@compatibility(is_backward_compatible=False)
def legalize_graph(gm: torch.fx.GraphModule) -> torch.fx.GraphModule:
    """
    Replace the graph of the given GraphModule with one that contains the same nodes as the
    original, but in topologically sorted order.

    This is used by the merge_matmul transformation below, which disturbs the topologically sorted
    order of its input GraphModule, so that this order is restored before further transformation.

    Arguments:
        gm: The graph module to topologically sort. It is modified in-place.

    Returns:
        The graph module in-place sorted
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 269-286
```python
    Warning:
        This topological sort is NOT stable, it will NOT preserve the original node order.
        If you need a stable topological sort, use stable_topological_sort instead.
    """

    # These operators are used for making runtime assertions before any
    # data-dependent operators occur. We want to prioritize sorting these to
    # ensure that these assertions appear before any data-dependent operations
    # in the graph.
    PRIORITIZED_OPS = [
        operator.add,
        operator.mul,
        operator.sub,
        operator.floordiv,
        operator.truediv,
        operator.mod,
        operator.le,
        operator.lt,
```
- **EN**: This block continues `legalize_graph` and works to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 该代码块继续实现 `legalize_graph`，用于构建、遍历或改写图结构及其元数据。

### Lines 287-304
```python
        operator.ge,
        operator.gt,
        operator.eq,
        operator.ne,
        torch.ops.aten.sym_constrain_range.default,
        torch.ops.aten.sym_constrain_range_for_size.default,
        torch.ops.aten._assert_async.msg,
        torch.ops.aten.scalar_tensor.default,
        torch.ops.aten._assert_scalar.default,
    ]

    indeg = dict.fromkeys(gm.graph.nodes, 0)
    new_graph = torch.fx.Graph()
    # Track how many unfulfilled dependencies each node has
    for node in gm.graph.nodes:
        for user in node.users:
            indeg[user] += 1
    queue: collections.deque[torch.fx.Node] = collections.deque()
```
- **EN**: This block continues `legalize_graph` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `legalize_graph`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构。

### Lines 305-331
```python
    # Add all nodes with no dependencies to the queue
    for node in gm.graph.nodes:
        if indeg[node] == 0:
            queue.append(node)
    env: dict[torch.fx.Node, torch.fx.Node] = {}
    # Pop nodes from the queue, and add nodes that have had all their
    # dependencies fulfilled
    while len(queue) > 0:
        cur = queue.popleft()
        env[cur] = new_graph.node_copy(cur, lambda x: env[x])
        for user in cur.users:
            indeg[user] -= 1
            if indeg[user] == 0:
                if user.op == "call_function" and user.target in PRIORITIZED_OPS:
                    queue.appendleft(user)
                else:
                    queue.append(user)
    # If the new graph's size is not as large as the old one, then there must be
    # a cycle (i.e. some node's dependencies were not satisfied.)
    if len(new_graph.nodes) < len(gm.graph.nodes):
        raise RuntimeError(
            f"Input graph has cycles, unable to add {[node for node in indeg if indeg[node] != 0]}"
        )
    new_graph._codegen = gm.graph._codegen
    gm.graph = new_graph
    return gm
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 332-356
```python

@compatibility(is_backward_compatible=False)
def stable_topological_sort(gm: torch.fx.GraphModule) -> torch.fx.GraphModule:
    """
    Replace the graph of the given GraphModule with one that contains the same nodes as the
    original, but in topologically sorted order while preserving the original node order
    as much as possible.

    This function performs a stable topological sort where nodes appear in an order that:
    1. Respects data dependencies (topological ordering)
    2. Preserves the original node order when there are no dependency constraints

    The algorithm uses Kahn's algorithm with a priority queue: nodes with all dependencies
    satisfied are added to a min-heap, ordered by their original position. This ensures
    we always process the earliest node in the original order among ready nodes.

    Arguments:
        gm: The graph module to topologically sort. It is modified in-place.

    Returns:
        The graph module in-place sorted
    """
    indeg = dict.fromkeys(gm.graph.nodes, 0)
    new_graph = torch.fx.Graph()
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 357-381
```python
    # Build node to original index mapping
    node_to_id: dict[torch.fx.Node, int] = {
        node: idx for idx, node in enumerate(gm.graph.nodes)
    }

    # Track how many unfulfilled dependencies each node has
    for node in gm.graph.nodes:
        for user in node.users:
            indeg[user] += 1

    # Priority queue: (original_index, node)
    # Use min-heap to always process the node with smallest original index
    ready_queue: list[tuple[int, torch.fx.Node]] = []
    for node in gm.graph.nodes:
        if indeg[node] == 0:
            heapq.heappush(ready_queue, (node_to_id[node], node))

    env: dict[torch.fx.Node, torch.fx.Node] = {}

    # Process nodes
    while ready_queue:
        # Pop node with smallest original index
        _, cur = heapq.heappop(ready_queue)
        env[cur] = new_graph.node_copy(cur, lambda x: env[x])
```
- **EN**: This block continues `stable_topological_sort` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `stable_topological_sort`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 382-396
```python
        # Update in-degrees and add newly ready nodes
        for user in cur.users:
            indeg[user] -= 1
            if indeg[user] == 0:
                heapq.heappush(ready_queue, (node_to_id[user], user))

    # Check if all nodes were processed
    if len(new_graph.nodes) != len(gm.graph.nodes):
        raise AssertionError(
            f"Input graph has cycles, unable to add {[node for node in indeg if indeg[node] != 0]}"
        )

    new_graph._codegen = gm.graph._codegen
    gm.graph = new_graph
    return gm
```
- **EN**: This block continues `stable_topological_sort` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `stable_topological_sort`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
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
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx`, `torch.fx._compatibility`, `torch.fx.node`
- **Standard library / 标准库**: `collections`, `heapq`, `operator`, `collections.abc`, `dataclasses`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `CALLABLE_NODE_OPS`, `get_acc_ops_name`, `get_node_target`, `is_node_output_tensor`, `FxNetAccFusionsFinder`, `legalize_graph`, `stable_topological_sort`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
