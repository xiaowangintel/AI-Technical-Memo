# subgraph_rewriter.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/subgraph_rewriter.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.fx symbolic tracing, graph IR manipulation, and transformation utilities. Key symbols exposed here include `__all__`, `Match`, `ReplacedPatterns`, `_replace_attributes`.
- **Purpose (CN)**: 实现 torch.fx 的符号跟踪、图中间表示操作与变换工具。 这里暴露的关键符号包括 `__all__`, `Match`, `ReplacedPatterns`, `_replace_attributes`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25
```python
import copy
from collections.abc import Callable
from dataclasses import dataclass
from typing import Any, NamedTuple, TYPE_CHECKING

import torch

from ._compatibility import compatibility
from ._symbolic_trace import symbolic_trace
from .graph import Graph
from .graph_module import GraphModule
from .node import Node


if TYPE_CHECKING:
    from .passes.utils.matcher_with_name_node_map_utils import InternalMatch

__all__ = [
    "Match",
    "replace_pattern",
    "replace_pattern_with_filters",
    "ReplacedPatterns",
]
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 26-50
```python
@compatibility(is_backward_compatible=True)
class Match(NamedTuple):
    # Node from which the match was found
    anchor: Node
    # Maps nodes in the pattern subgraph to nodes in the larger graph
    nodes_map: dict[Node, Node]


@compatibility(is_backward_compatible=False)
@dataclass
class ReplacedPatterns:
    # Node from which the match was found
    anchor: Node
    # Maps nodes in the pattern subgraph to nodes in the larger graph
    nodes_map: dict[Node, Node]
    # List of nodes that were added into the graph
    replacements: list[Node]


def _replace_attributes(gm: GraphModule, replacement: torch.nn.Module) -> None:
    gm.delete_all_unused_submodules()

    if isinstance(replacement, GraphModule):
        replacement.graph.lint()
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 51-70
```python
    def try_get_attr(gm: torch.nn.Module, target: str) -> Any | None:
        module_path, _, attr_name = target.rpartition(".")
        try:
            mod: torch.nn.Module = gm.get_submodule(module_path)
        except AttributeError:
            return None
        attr = getattr(mod, attr_name, None)
        return attr

    for node in gm.graph.nodes:
        if node.op == "call_module" or node.op == "get_attr":
            gm_attr = try_get_attr(gm, node.target)
            replacement_attr = try_get_attr(replacement, node.target)

            # CASE 1: This target already exists as an attribute in our
            # result GraphModule. Whether or not it exists in
            # `replacement`, the existing submodule takes precedence.
            if gm_attr is not None:
                continue
```
- **EN**: Defines the `_replace_attributes` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`_replace_attributes` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 71-94
```python
            # CASE 2: The target exists as an attribute in `replacement`
            # only, so we need to copy it over.
            elif replacement_attr is not None:
                new_attr = copy.deepcopy(replacement_attr)
                if isinstance(replacement_attr, torch.nn.Module):
                    gm.add_submodule(node.target, new_attr)
                else:
                    setattr(gm, node.target, new_attr)

            # CASE 3: The target doesn't exist as an attribute in `gm`
            # or `replacement`
            else:
                raise RuntimeError(
                    'Attempted to create a "',
                    node.op,
                    '" node during subgraph rewriting '
                    f"with target {node.target}, but "
                    "the referenced attribute does not "
                    "exist in the replacement GraphModule",
                )

    gm.graph.lint()
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 95-118
```python
@compatibility(is_backward_compatible=True)
def replace_pattern(
    gm: GraphModule,
    pattern: Callable[..., Any] | GraphModule,
    replacement: Callable[..., Any] | GraphModule,
) -> list[Match]:
    """
    Matches all possible non-overlapping sets of operators and their
    data dependencies (``pattern``) in the Graph of a GraphModule
    (``gm``), then replaces each of these matched subgraphs with another
    subgraph (``replacement``).

    Args:
        ``gm``: The GraphModule that wraps the Graph to operate on
        ``pattern``: The subgraph to match in ``gm`` for replacement
        ``replacement``: The subgraph to replace ``pattern`` with

    Returns:
        List[Match]: A list of ``Match`` objects representing the places
        in the original graph that ``pattern`` was matched to. The list
        is empty if there are no matches. ``Match`` is defined as:

        .. code-block:: python
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 119-145
```python
            class Match(NamedTuple):
                # Node from which the match was found
                anchor: Node
                # Maps nodes in the pattern subgraph to nodes in the larger graph
                nodes_map: Dict[Node, Node]

    Examples:

    .. code-block:: python

        import torch
        from torch.fx import symbolic_trace, subgraph_rewriter


        class M(torch.nn.Module):
            def __init__(self) -> None:
                super().__init__()

            def forward(self, x, w1, w2):
                m1 = torch.cat([w1, w2]).sum()
                m2 = torch.cat([w1, w2]).sum()
                return x + torch.max(m1) + torch.max(m2)


        def pattern(w1, w2):
            return torch.cat([w1, w2])
```
- **EN**: Defines the `replace_pattern` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`replace_pattern` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 146-166
```python

        def replacement(w1, w2):
            return torch.stack([w1, w2])


        traced_module = symbolic_trace(M())

        subgraph_rewriter.replace_pattern(traced_module, pattern, replacement)

    The above code will first match ``pattern`` in the ``forward``
    method of ``traced_module``. Pattern-matching is done based on
    use-def relationships, not node names. For example, if you had
    ``p = torch.cat([a, b])`` in ``pattern``, you could match
    ``m = torch.cat([a, b])`` in the original ``forward`` function,
    despite the variable names being different (``p`` vs ``m``).

    The ``return`` statement in ``pattern`` is matched based on its
    value only; it may or may not match to the ``return`` statement in
    the larger graph. In other words, the pattern doesn't have to extend
    to the end of the larger graph.
```
- **EN**: Defines the `replace_pattern` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`replace_pattern` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 167-192
```python
    When the pattern is matched, it will be removed from the larger
    function and replaced by ``replacement``. If there are multiple
    matches for ``pattern`` in the larger function, each non-overlapping
    match will be replaced. In the case of a match overlap, the first
    found match in the set of overlapping matches will be replaced.
    ("First" here being defined as the first in a topological ordering
    of the Nodes' use-def relationships. In most cases, the first Node
    is the parameter that appears directly after ``self``, while the
    last Node is whatever the function returns.)

    One important thing to note is that the parameters of the
    ``pattern`` Callable must be used in the Callable itself,
    and the parameters of the ``replacement`` Callable must match
    the pattern. The first rule is why, in the above code block, the
    ``forward`` function has parameters ``x, w1, w2``, but the
    ``pattern`` function only has parameters ``w1, w2``. ``pattern``
    doesn't use ``x``, so it shouldn't specify ``x`` as a parameter.
    As an example of the second rule, consider replacing

    .. code-block:: python

        def pattern(x, y):
            return torch.neg(x) + torch.relu(y)

    with
```
- **EN**: Defines the `replace_pattern` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`replace_pattern` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 193-210
```python
    .. code-block:: python

        def replacement(x, y):
            return torch.relu(x)

    In this case, ``replacement`` needs the same number of parameters
    as ``pattern`` (both ``x`` and ``y``), even though the parameter
    ``y`` isn't used in ``replacement``.

    After calling ``subgraph_rewriter.replace_pattern``, the generated
    Python code looks like this:

    .. code-block:: python

        def forward(self, x, w1, w2):
            stack_1 = torch.stack([w1, w2])
            sum_1 = stack_1.sum()
            stack_2 = torch.stack([w1, w2])
```
- **EN**: Defines the `replace_pattern` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`replace_pattern` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 211-228
```python
            sum_2 = stack_2.sum()
            max_1 = torch.max(sum_1)
            add_1 = x + max_1
            max_2 = torch.max(sum_2)
            add_2 = add_1 + max_2
            return add_2
    """
    match_and_replacements = _replace_pattern(gm, pattern, replacement)
    return [
        Match(anchor=m.anchor, nodes_map=m.nodes_map) for m in match_and_replacements
    ]


# Experimental API, not backward compatible
@compatibility(is_backward_compatible=False)
def replace_pattern_with_filters(
    gm: GraphModule,
    pattern: Callable[..., Any] | Graph | GraphModule,
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 229-250
```python
    replacement: Callable[..., Any] | Graph | GraphModule | None = None,
    match_filters: list[Callable[["InternalMatch", Graph, Graph], bool]] | None = None,
    ignore_literals: bool = False,
    # Placed at the end to avoid breaking backward compatibility
    replacement_callback: Callable[["InternalMatch", Graph, Graph], Graph]
    | None = None,
    node_name_match: str = "",
) -> list[ReplacedPatterns]:
    """
    See replace_pattern for documentation. This function is an overload with an additional match_filter argument.

    Args:
        ``match_filters``: A list of functions that take in
            (match: InternalMatch, original_graph: Graph, pattern_graph: Graph) and return a boolean indicating
            whether the match satisfies the condition.
            See matcher_utils.py for definition of InternalMatch.
        ``replacement_callback``: A function that takes in a match and returns a
            Graph to be used as the replacement. This allows you to construct a
            replacement graph based on the match.
        ``replacement_callback``: Node name to match. If not empty, it will try to match the node name.
    """
```
- **EN**: This block continues `replace_pattern_with_filters` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `replace_pattern_with_filters`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 251-277
```python
    return _replace_pattern(
        gm,
        pattern,
        replacement,
        match_filters,
        ignore_literals,
        replacement_callback,
        node_name_match,
    )


def _replace_pattern(
    gm: GraphModule,
    pattern: Callable[..., Any] | Graph | GraphModule,
    replacement: Callable[..., Any] | Graph | GraphModule | None = None,
    match_filters: list[Callable[["InternalMatch", Graph, Graph], bool]] | None = None,
    ignore_literals: bool = False,
    # Placed at the end to avoid breaking backward compatibility
    replacement_callback: Callable[["InternalMatch", Graph, Graph], Graph]
    | None = None,
    node_name_match: str = "",
) -> list[ReplacedPatterns]:
    from torch.fx.passes.utils.matcher_utils import InternalMatch, SubgraphMatcher

    if match_filters is None:
        match_filters = []
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 278-298
```python
    # Get the graphs for `gm`, `pattern`, `replacement`
    original_graph: Graph = gm.graph

    if isinstance(pattern, GraphModule):
        pattern_graph = pattern.graph
    elif isinstance(pattern, Graph):
        pattern_graph = pattern
    else:
        pattern_graph = symbolic_trace(pattern).graph  # type: ignore[arg-type]

    matcher = SubgraphMatcher(
        pattern_graph,
        match_output=False,
        match_placeholder=False,
        remove_overlapping_matches=True,
        ignore_literals=ignore_literals,
    )
    _matches: list[InternalMatch] = matcher.match(
        original_graph, node_name_match=node_name_match
    )
```
- **EN**: This block continues `_replace_pattern` and works to track symbolic shape constraints and shape-dependent reasoning. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `_replace_pattern`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会根据运行时条件分支处理。

### Lines 299-324
```python
    # Filter out matches that don't match the filter
    _matches = [
        m
        for m in _matches
        if all(
            match_filter(m, original_graph, pattern_graph)
            for match_filter in match_filters
        )
    ]

    if isinstance(replacement, GraphModule):
        common_replacement_graph = replacement.graph
    elif isinstance(replacement, Graph):
        common_replacement_graph = replacement
    elif callable(replacement):
        common_replacement_graph = symbolic_trace(replacement).graph
    else:
        if replacement_callback is None:
            raise AssertionError(
                "Must provide either a replacement GraphModule or a replacement callback"
            )
        common_replacement_graph = None  # type: ignore[assignment]

    # As we progressively replace nodes, we'll need to keep track of how the match results should change
    match_changed_node: dict[Node, Node] = {}
```
- **EN**: This block continues `_replace_pattern` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_replace_pattern`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 325-342
```python
    match_and_replacements = []
    for match in _matches:
        if replacement_callback is not None:
            replacement_graph = replacement_callback(
                match, original_graph, pattern_graph
            )
        else:
            if common_replacement_graph is None:
                raise AssertionError(
                    "Must provide either a replacement GraphModule or a replacement callback"
                )
            replacement_graph = common_replacement_graph
        replacement_placeholders = [
            n for n in replacement_graph.nodes if n.op == "placeholder"
        ]

        # Build connecting between replacement graph's input and original graph input producer node
```
- **EN**: This block continues `_replace_pattern` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_replace_pattern`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 343-369
```python
        # Initialize `val_map` with mappings from placeholder nodes in
        # `replacement` to their corresponding node in `original_graph`
        if len(match.placeholder_nodes) != len(replacement_placeholders):
            raise AssertionError(
                f"Placeholder count mismatch: {len(match.placeholder_nodes)} vs "
                f"{len(replacement_placeholders)}"
            )
        val_map: dict[Node, Node] = {}
        for rn, gn in zip(replacement_placeholders, match.placeholder_nodes):
            if isinstance(gn, Node):
                val_map[rn] = match_changed_node.get(gn, gn)
                if gn != val_map[rn]:
                    # Update match.placeholder_nodes and match.nodes_map with the node that replaced gn
                    gn_ind = match.placeholder_nodes.index(gn)
                    match.placeholder_nodes[gn_ind] = match_changed_node[gn]
                    map_key = list(match.nodes_map.keys())[
                        list(match.nodes_map.values()).index(gn)
                    ]
                    match.nodes_map[map_key] = match_changed_node[gn]
            else:
                val_map[rn] = gn

        # Copy the replacement graph over
        user_nodes: set[Node] = set()
        for n in match.returning_nodes:
            user_nodes.update(n.users)
```
- **EN**: This block continues `_replace_pattern` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_replace_pattern`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 370-387
```python
        first_user_node = None
        if len(user_nodes) == 0:
            first_user_node = None
        elif len(user_nodes) == 1:
            first_user_node = next(iter(user_nodes))
        else:
            # If there are multiple user nodes, we need to find the first user node
            # in the current execution order of the `original_graph`
            for n in original_graph.nodes:
                if n in user_nodes:
                    first_user_node = n
                    break

        first_next_node = None
        if first_user_node is None:
            # no users, so we insert the replacement graph before the first next
            # node of returning nodes
            next_node = None
```
- **EN**: This block continues `_replace_pattern` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_replace_pattern`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 388-411
```python
            for n in reversed(original_graph.nodes):
                if n in match.returning_nodes:
                    first_next_node = next_node
                    break
                else:
                    next_node = n
        insert_point = (
            first_user_node if first_user_node is not None else first_next_node
        )
        if insert_point is None:
            raise AssertionError("The insert point can't be None")
        with original_graph.inserting_before(insert_point):
            copied_returning_nodes = original_graph.graph_copy(
                replacement_graph, val_map
            )

        if isinstance(copied_returning_nodes, Node):
            copied_returning_nodes = (copied_returning_nodes,)

        # Get a list of nodes that have been replaced into the graph
        replacement_nodes: list[Node] = [
            v for v in val_map.values() if v not in match.placeholder_nodes
        ]
```
- **EN**: This block continues `_replace_pattern` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_replace_pattern`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 412-437
```python
        # Hook the output Node of the replacement subgraph in to the
        # original Graph at the correct location
        if len(match.returning_nodes) != len(copied_returning_nodes):  # type: ignore[arg-type]
            raise AssertionError(
                f"Returning nodes count mismatch: {len(match.returning_nodes)} vs "
                f"{len(copied_returning_nodes)}"  # pyrefly: ignore [bad-argument-type]
            )
        for gn, copied_node in zip(match.returning_nodes, copied_returning_nodes):  # type: ignore[arg-type]
            # pyrefly: ignore [bad-argument-type]
            gn.replace_all_uses_with(copied_node)
            # pyrefly: ignore [unsupported-operation]
            match_changed_node[gn] = copied_node
        # Remove the original nodes
        for node in reversed(pattern_graph.nodes):
            if node.op != "placeholder" and node.op != "output":
                gn = match.nodes_map[node]
                gm.graph.erase_node(gn)

        match_and_replacements.append(
            ReplacedPatterns(
                anchor=match.anchors[0],
                nodes_map=match.nodes_map,
                replacements=replacement_nodes,
            )
        )
```
- **EN**: This block continues `_replace_pattern` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_replace_pattern`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 438-447
```python
    # Update the passed-in GraphModule to reflect the new state of
    # `original_graph`
    gm.recompile()

    # If `replacement` was an nn.Module, we'll need to make sure that
    # all the submodules have been copied over correctly
    if isinstance(replacement, torch.nn.Module):
        _replace_attributes(gm, replacement)

    return match_and_replacements
```
- **EN**: This block continues `_replace_pattern` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_replace_pattern`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

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
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `._compatibility`, `._symbolic_trace`, `.graph`, `.graph_module`, `.node`, `.passes.utils.matcher_with_name_node_map_utils`, `torch.fx.passes.utils.matcher_utils`
- **Standard library / 标准库**: `copy`, `collections.abc`, `dataclasses`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `Match`, `ReplacedPatterns`, `_replace_attributes`, `replace_pattern`, `replace_pattern_with_filters`, `_replace_pattern`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
