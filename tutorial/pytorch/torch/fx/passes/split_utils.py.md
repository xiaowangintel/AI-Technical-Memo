# split_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/passes/split_utils.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects graph-to-graph optimization, rewriting, and analysis passes for FX IR.
- **Purpose (CN)**: 汇集针对 FX 中间表示的图到图优化、改写与分析 passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25
```python
import copy
from dataclasses import dataclass, field
from typing import Any, TYPE_CHECKING

import torch.fx
from torch.fx._compatibility import compatibility
from torch.fx.graph import map_arg
from torch.fx.passes.utils import HolderModule, lift_subgraph_as_module

from .tools_common import CALLABLE_NODE_OPS, is_node_output_tensor, NodeList


if TYPE_CHECKING:
    from .splitter_base import Subgraph


__all__ = [
    "getattr_recursive",
    "setattr_recursive",
    "Component",
    "split_by_tags",
    "move_non_tensor_nodes_on_boundary",
]
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 26-49
```python
@compatibility(is_backward_compatible=False)
def getattr_recursive(obj: object, name: str) -> Any:
    for layer in name.split("."):
        if isinstance(obj, torch.nn.ModuleList):
            if hasattr(obj, "_modules") and layer in obj._modules:
                obj = obj._modules[layer]
            else:
                return None
        elif hasattr(obj, layer):
            obj = getattr(obj, layer)
        else:
            return None
    return obj


@compatibility(is_backward_compatible=False)
def setattr_recursive(obj: object, attr: str, value: object) -> None:
    if "." not in attr:
        setattr(obj, attr, value)
    else:
        layer = attr.split(".")
        setattr_recursive(getattr(obj, layer[0]), ".".join(layer[1:]), value)
```
- **EN**: These decorators register or transform the following definition so it can organize reusable module behavior and state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够组织可复用的模块行为与状态。

### Lines 50-75
```python
@compatibility(is_backward_compatible=False)
@dataclass
class Component:
    """
    A component serves as a container for a subgraph we want to create afterwards.
    """

    graph: torch.fx.Graph
    order: int
    name: str

    # Stores the placeholder nodes in `graph`.
    input_placeholders: list[torch.fx.Node] = field(default_factory=list)

    # Store the nodes in original graph that are placeholder in `graph`.
    orig_inputs: list[torch.fx.Node] = field(default_factory=list)

    # Store the nodes in original graph that are outputs in `graph`.
    orig_outputs: list[torch.fx.Node] = field(default_factory=list)

    # Mapping from get_attr node in original graph to get_attr node in `graph`.
    getattr_maps: dict[torch.fx.Node, torch.fx.Node] = field(default_factory=dict)
    constructor_args: list[str] = field(default_factory=list)
    gm: torch.fx.GraphModule | None = None
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 76-98
```python
@compatibility(is_backward_compatible=False)
def split_by_tags(
    gm: torch.fx.GraphModule,
    tags: list[str],
    return_fqn_mapping: bool = False,
    return_tuple: bool = False,
    GraphModuleCls: type[torch.fx.GraphModule] = torch.fx.GraphModule,
) -> torch.fx.GraphModule | tuple[torch.fx.GraphModule, dict[str, str]]:
    """
    Splits a GraphModule using tags on its graph nodes. We honor the order of
    tags. For example, we have tags = ["a", "b", "c"], the function will create
    the initial submodules in the order of "a", "b", "c".

    To set a tag:
    gm.graph.nodes[idx].tag = "mytag"

    This will result in all nodes with the same tag being extracted and placed in their
    own submodule. For placeholder, output and get_attr node, the tag is ignored. placeholder
    and output nodes are created when needed while get_attr nodes get copied to submodules
    where they are used.

    Given the following module def:
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 99-119
```python
    class SimpleModule(torch.nn.Module):
        def __init__(self) -> None:
            super().__init__()
            self.linear1 = torch.nn.Linear(...)
            self.linear2 = torch.nn.Linear(...)
            self.linear3 = torch.nn.Linear(...)

        def forward(self, in1, in2):
            r1 = self.linear1(in1)
            r2 = self.linear2(in2)
            r3 = torch.cat([r1, r2])
            return self.linear3(r3)

    Marking the node corresponding to in1 with the tag sc.REQUEST_ONLY.lower() results in the following split:

    ro:
    def forward(self, in1):
        self = self.root
        linear1 = self.linear1(in1)
        return linear1
```
- **EN**: Defines the `split_by_tags` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`split_by_tags` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 120-140
```python
    main:
    def forward(self, in2, linear1):
        self = self.root
        linear2 = self.linear2(in2)
        cat_1 = torch.cat([linear1, linear2])
        linear3 = self.linear3(cat_1)
        return linear3

    main:
    def forward(self, in1, in2):
        self = self.root
        ro_0 = self.ro_0(in1)
        main_1 = self.main_1(in2, ro_0)
        return main_1

    Returns:
        split_gm: torch fx graph after split
        orig_to_split_fqn_mapping: a map between the original fqn and the fqn
            after split for call_module and get_attr.
    """
```
- **EN**: Defines the `split_by_tags` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`split_by_tags` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 141-167
```python
    def flatten(x: torch.fx.node.Argument) -> NodeList:
        """
        Stores nodes in x to a list and returns the list.
        """
        r: NodeList = []
        map_arg(x, r.append)
        return r

    # Mapping from node in original module to node in created submodule.
    node_remapping: dict[torch.fx.Node, torch.fx.Node] = {}

    # Mapping from node in original module or created submodules to
    # corresponding component.
    node_to_component: dict[torch.fx.Node, Component] = {}

    # Mapping from tag to the corresponding component.
    tag_to_component: dict[str, Component] = {}

    # Stores all components.
    all_components: list[Component] = []

    # Stores nodes that will be used in main graph.
    used_in_main: dict[torch.fx.Node, None] = {}

    # Main graph after split.
    main_g = torch.fx.Graph()
```
- **EN**: Defines the `split_by_tags` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`split_by_tags` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 168-193
```python
    # Mapping from node in original module to node in main graph after split.
    main_remapping: dict[torch.fx.Node, torch.fx.Node] = {}

    # Output node of original module.
    output_node: torch.fx.Node | None = None

    # Create a component for each tag, we don't expect to create other components afterwards.
    for tag in tags:
        comp = Component(torch.fx.Graph(), len(all_components), f"{tag}")
        all_components.append(comp)
        tag_to_component[tag] = comp

    # Traverse the nodes in original graph and take care of them.
    for node in gm.graph.nodes:
        if node.op == "output":
            if output_node is not None:
                raise RuntimeError("Multiple output nodes in graph!")
            output_node = node
            continue

        # Placeholders in the original graph get copied to main graph.
        if node.op == "placeholder":
            main_remapping[node] = main_g.placeholder(node.name, type_expr=node.type)
            main_remapping[node].meta = copy.copy(node.meta)
            continue
```
- **EN**: This block continues `split_by_tags` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_by_tags`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 194-215
```python
        # Get_attr nodes are ignored because we are not tagging them.
        # Instead, we copy them directly to the submodules use them afterwards.
        if node.op == "get_attr":
            continue

        # Now we process callable nodes which are nodes with op of call_module,
        # call_function or call_method. Every callable nodes should be tagged.
        if not hasattr(node, "tag"):
            raise AssertionError(f"Node does not have tag: {node.format_node()}")

        upstream_components = [
            node_to_component[x]
            for x in flatten(node.args) + flatten(node.kwargs)
            if x.op not in {"placeholder", "get_attr"}
        ]

        comp = tag_to_component[node.tag]
        node_to_component[node] = comp

        # Max order of upperstream components.
        mx = max((c.order for c in upstream_components), default=0)
```
- **EN**: This block continues `split_by_tags` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_by_tags`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 216-238
```python
        # Expect the component for `node` has higher order then its upstream components.
        if comp.order < mx:
            raise AssertionError(
                f"Component {comp.name} order must be >= max of its upstream components, "
                f"order={comp.order} and max={mx}"
            )

        # Map a input of `node` to nodes in the component's graph.
        def remap_func(x: torch.fx.Node) -> torch.fx.Node:
            # If input is a get_attr node, copy it to current component's graph.
            # Returns the get_attr node in current component's graph.
            if x.op == "get_attr":
                if not isinstance(x.target, str):
                    raise RuntimeError(
                        f"Expected get_attr node target to be a str, got {type(x.target)}"
                    )
                if x not in comp.getattr_maps:
                    comp.getattr_maps[x] = comp.graph.get_attr(
                        x.target, type_expr=x.type
                    )
                    comp.getattr_maps[x].meta = copy.copy(x.meta)
                return comp.getattr_maps[x]
```
- **EN**: Defines the `split_by_tags` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`split_by_tags` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 239-263
```python
            # If input is not a placeholder, it should have been put into a component
            # already. If it's the current component then we return the corresponding
            # node in the component.
            if x.op != "placeholder" and node_to_component[x] == comp:
                return node_remapping[x]

            # If input is a placeholder or it's in other components, we want to make it
            # as a placeholder in current component's graph.
            if x not in comp.orig_inputs:
                comp.orig_inputs.append(x)
                placeholder = comp.graph.placeholder(x.name, type_expr=x.type)
                placeholder.meta = copy.copy(x.meta)
                comp.input_placeholders.append(placeholder)
                used_in_main[x] = None

            return comp.input_placeholders[comp.orig_inputs.index(x)]

        n = comp.graph.node_copy(node, remap_func)
        n.tag = node.tag  # type: ignore[attr-defined]
        node_remapping[node] = n
        node_to_component[n] = comp

    if output_node is None:
        raise RuntimeError("Graph had no output node!")
```
- **EN**: This block continues `split_by_tags` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_by_tags`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 264-285
```python
    for x in flatten(output_node.args[0]):
        if x.op == "get_attr":
            # We don't need components mapping for nodes of type "get_attr"
            # that are consumed by the output. Only need to make sure we create
            # corresponding counterparts in the resulting graph.
            main_remapping[x] = main_g.get_attr(x.name, type_expr=x.type)
        else:
            # All component results consumed by the output node should be
            # marked as "used in main".
            used_in_main[x] = None

    # If a node is used in main graph then we mark it as an output in the component
    # it belongs to.
    for n in used_in_main:
        if n.op != "placeholder":
            node_to_component[n].orig_outputs.append(n)

    # Now we create a graphmodule for each component.
    orig_to_split_fqn_mapping: dict[str, str] = {}
    for comp in all_components:
        outs = tuple(map(node_remapping.__getitem__, comp.orig_outputs))
```
- **EN**: This block continues `split_by_tags` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_by_tags`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 286-306
```python
        if return_tuple:
            comp.graph.output(outs)
        else:
            # Take care of the args of FX output node. If there's a single
            # output then the output node args is like (output_single), else
            # if there're multiple outputs then the output node args is like
            # ((output_0, output_1, ...)).
            comp.graph.output(outs[0] if len(outs) == 1 else outs)

        comp.gm, comp_orig_to_split_fqn_mapping = lift_subgraph_as_module(
            gm, subgraph=comp.graph, comp_name=comp.name
        )
        orig_to_split_fqn_mapping.update(comp_orig_to_split_fqn_mapping)

        # Create a call_module node in main graph.
        main_node = main_g.call_module(
            comp.name,
            args=tuple(map(main_remapping.__getitem__, comp.orig_inputs)),
            kwargs=None,
        )
```
- **EN**: This block continues `split_by_tags` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_by_tags`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 307-330
```python
        if len(outs) == 1 and not return_tuple:
            main_remapping[comp.orig_outputs[0]] = main_node
        else:
            for i, o in enumerate(comp.orig_outputs):
                # Use Proxy to record getitem access.
                main_remapping[o] = torch.fx.Proxy(main_node)[i].node  # type: ignore[index]

    main_g.output(map_arg(output_node.args[0], main_remapping.__getitem__))
    main_root = HolderModule({comp.name: comp.gm for comp in all_components})
    main_g._codegen = gm.graph._codegen

    # If the output nodes consumes get_attr directly in the original graph,
    # then we need to make sure get_attr is copied to the new graph.
    for x in flatten(output_node.args[0]):
        if x.op == "get_attr":
            setattr(main_root, x.name, getattr_recursive(gm, x.target))  # type: ignore[arg-type]

    result_gm = GraphModuleCls(main_root, main_g)
    if return_fqn_mapping:
        return result_gm, orig_to_split_fqn_mapping

    return result_gm
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 331-357
```python
@compatibility(is_backward_compatible=False)
def move_non_tensor_nodes_on_boundary(subgraphs: list["Subgraph"]) -> None:
    """
    Move non-tensor nodes on the boundary between subgraphs.

    For each subgraph:

    1. Find nodes whose type is not tensor and any of its children is in another
       subgraph, put them in a queue for next step

    2. Do a BFS on those nodes in the queue,  and run a DFS for each node, let's say node X and it is in subgraph A:

       a. if it is in to_subgraph, return (continue DFS)
       b. if it is in from_subgraph, collect the nodes to nodes_to_move, and continue DFS
       c. otherwise, this means it cannot be moved
       d. also check if node X's parent should be put into the queue. (The queue may
          have duplicated nodes, just process the node once)

    Args:
        subgraphs: List of subgraphs containing nodes to be processed
    """
    # Create a mapping from node to subgraph for quick lookup
    node_to_subgraph: dict[torch.fx.Node, int] = {}
    for i, subgraph in enumerate(subgraphs):
        for node in subgraph.nodes:
            node_to_subgraph[node] = i
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 358-375
```python
    def get_children_in_graph(node: torch.fx.Node) -> list[torch.fx.Node]:
        """Get children nodes that are in callable ops and in some subgraph"""
        return [
            user
            for user in node.users
            if user.op in CALLABLE_NODE_OPS and user in node_to_subgraph
        ]

    def get_parents_in_graph(node: torch.fx.Node) -> list[torch.fx.Node]:
        """Get parent nodes that are in callable ops and in some subgraph"""
        return [
            arg
            for arg in node.all_input_nodes
            if arg.op in CALLABLE_NODE_OPS and arg in node_to_subgraph
        ]

    def has_children_in_other_subgraph(
        node: torch.fx.Node, current_subgraph_idx: int
```
- **EN**: Defines the `move_non_tensor_nodes_on_boundary` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`move_non_tensor_nodes_on_boundary` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 376-401
```python
    ) -> bool:
        """
        Check if the node has any children in a subgraph different from current_subgraph_idx.
        This is the requirement used in both step 1 and step d.
        """
        children = get_children_in_graph(node)
        return any(
            node_to_subgraph[child] != current_subgraph_idx for child in children
        )

    def can_move_node_and_dependencies(
        node: torch.fx.Node, from_subgraph: int, to_subgraph: int
    ) -> tuple[bool, set[torch.fx.Node]]:
        """
        Check if node and its dependencies can be moved from from_subgraph to to_subgraph.
        Returns (can_move, nodes_to_move)

        For node X, do a DFS on its descendants, for each node:
        - if it is in to_subgraph, return (continue DFS)
        - if it is in from_subgraph, collect the nodes to nodes_to_move, and continue DFS
        - otherwise, this means it cannot be moved
        """
        nodes_to_move = set()
        visited = set()
        can_move = True
```
- **EN**: Defines the `move_non_tensor_nodes_on_boundary` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`move_non_tensor_nodes_on_boundary` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 402-425
```python
        def dfs(current_node: torch.fx.Node) -> None:
            nonlocal can_move, nodes_to_move

            if current_node in visited:
                return
            visited.add(current_node)

            # Check current node's subgraph
            if current_node not in node_to_subgraph:
                return  # Skip nodes not in any subgraph

            current_subgraph = node_to_subgraph[current_node]

            if current_subgraph == to_subgraph:
                # If it is in to_subgraph, just end DFS
                return
            elif current_subgraph == from_subgraph:
                # If it is in from_subgraph, collect it and continue DFS
                nodes_to_move.add(current_node)
            else:
                # Otherwise, this means it cannot be moved
                can_move = False
                return
```
- **EN**: Defines the `move_non_tensor_nodes_on_boundary.can_move_node_and_dependencies.dfs` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`move_non_tensor_nodes_on_boundary.can_move_node_and_dependencies.dfs` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 426-450
```python
            # Continue DFS on children
            children = get_children_in_graph(current_node)
            for child in children:
                if can_move:  # Only continue if we haven't already failed
                    dfs(child)

        # Start DFS from the original node
        dfs(node)

        return can_move, nodes_to_move

    # For each subgraph, find non-tensor nodes with children in other subgraphs
    for subgraph_idx, subgraph in enumerate(subgraphs):
        # non acc nodes cannot be moved to downstream acc graph, so skip
        if not subgraph.is_acc:
            continue
        # Step 1: Find non-tensor nodes with children in other subgraphs
        queue: list[torch.fx.Node] = []
        processed: set[torch.fx.Node] = set()

        for node in subgraph.nodes:
            # Check if node is non-tensor
            if is_node_output_tensor(node):
                continue
```
- **EN**: This block continues `move_non_tensor_nodes_on_boundary` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `move_non_tensor_nodes_on_boundary`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 451-476
```python
            # Check if node meets step 1 requirement: any children in another subgraph
            if has_children_in_other_subgraph(node, subgraph_idx):
                queue.append(node)

        # Step 2: BFS to move nodes that meet the criteria
        while queue:
            current_node = queue.pop(0)

            # Skip if already processed (queue may have duplicates)
            if current_node in processed:
                continue
            processed.add(current_node)

            # Skip if node is no longer in this subgraph (may have been moved)
            if (
                current_node not in node_to_subgraph
                or node_to_subgraph[current_node] != subgraph_idx
            ):
                continue

            children = get_children_in_graph(current_node)
            if len(children) == 0:
                raise AssertionError(
                    "Only node that has children in other subgraph can be moved"
                )
```
- **EN**: This block continues `move_non_tensor_nodes_on_boundary` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `move_non_tensor_nodes_on_boundary`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 477-496
```python
            # Find target subgraph. The children should all be in the same subgraph except current subgraph
            target_subgraph_candidates = set()
            for child in children:
                child_subgraph = node_to_subgraph[child]
                if child_subgraph != subgraph_idx:
                    target_subgraph_candidates.add(child_subgraph)
            # If multiple children live in different subgraphs, the node cannot be moved. User needs to find other ways to move it.
            if len(target_subgraph_candidates) != 1:
                print(
                    f"Cannot move non-tensor node {current_node.name} on boundary because it has children in multiple subgraphs"
                )
                continue

            target_subgraph = target_subgraph_candidates.pop()

            # Check if we can move this node and its dependencies
            can_move, nodes_to_move = can_move_node_and_dependencies(
                current_node, subgraph_idx, target_subgraph
            )
```
- **EN**: This block continues `move_non_tensor_nodes_on_boundary` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `move_non_tensor_nodes_on_boundary`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 497-514
```python
            if can_move:
                # Move all nodes in nodes_to_move to target subgraph
                for node_to_move in nodes_to_move:
                    # Remove from current subgraph
                    subgraph.nodes.remove(node_to_move)
                    # Add to target subgraph
                    subgraphs[target_subgraph].nodes.append(node_to_move)
                    # Update mapping
                    node_to_subgraph[node_to_move] = target_subgraph
                    print(
                        f"In order move the non-tensor node {current_node.name} on boundary, "
                        f"moved node {node_to_move.name} from {'acc' if subgraph.is_acc else 'gpu'}_{subgraph_idx} "
                        f"to {'acc' if subgraphs[target_subgraph].is_acc else 'gpu'}_{target_subgraph}"
                    )

                # Add parents to the queue if they're non-tensor and not already processed
                # and meet the requirement from step 1 (any children in another subgraph)
                parents = get_parents_in_graph(current_node)
```
- **EN**: This block continues `move_non_tensor_nodes_on_boundary` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `move_non_tensor_nodes_on_boundary`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 515-527
```python
                for parent in parents:
                    if (
                        not is_node_output_tensor(parent)
                        and parent not in processed
                        and parent in node_to_subgraph
                        and node_to_subgraph[parent] == subgraph_idx
                    ):
                        # Check if parent meets step 1 requirement: any children in another subgraph
                        if not has_children_in_other_subgraph(parent, subgraph_idx):
                            raise AssertionError(
                                f"Parent {parent.name} should have children in another subgraph"
                            )
                        queue.append(parent)
```
- **EN**: This block continues `move_non_tensor_nodes_on_boundary` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `move_non_tensor_nodes_on_boundary`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

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
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch.fx`, `torch.fx._compatibility`, `torch.fx.graph`, `torch.fx.passes.utils`, `.tools_common`, `.splitter_base`
- **Standard library / 标准库**: `copy`, `dataclasses`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `getattr_recursive`, `setattr_recursive`, `Component`, `split_by_tags`, `move_non_tensor_nodes_on_boundary`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
