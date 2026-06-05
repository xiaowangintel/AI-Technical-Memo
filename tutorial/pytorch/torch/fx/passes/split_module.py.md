# split_module.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/passes/split_module.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects graph-to-graph optimization, rewriting, and analysis passes for FX IR.
- **Purpose (CN)**: 汇集针对 FX 中间表示的图到图优化、改写与分析 passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```python
import inspect
import logging
from collections import OrderedDict
from collections.abc import Callable
from typing import Any

import torch
from torch.fx._compatibility import compatibility
from torch.fx._lazy_graph_module import _make_graph_module
from torch.fx._utils import lazy_format_graph_code
from torch.fx.graph_module import GraphModule
from torch.fx.node import Node


__all__ = ["Partition", "split_module"]
log = _LOGGER = logging.getLogger(__name__)


@compatibility(is_backward_compatible=True)
class Partition:
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 21-43
```python
    def __init__(self, name: str) -> None:
        self.name: str = name
        self.submod_name = f"submod_{name}"
        self.node_names: list[str] = []
        self.inputs: dict[str, None] = {}
        self.outputs: dict[str, None] = {}
        self.dependencies: dict[str, None] = {}
        self.dependents: dict[str, None] = {}
        self.graph: torch.fx.graph.Graph = torch.fx.graph.Graph()
        self.environment: dict[Node, Node] = {}
        self.targets: dict[str, Any] = {}

    def __repr__(self) -> str:
        return (
            f"name: {self.name},\n"
            f" nodes: {self.node_names},\n"
            f" inputs: {self.inputs},\n"
            f" outputs: {self.outputs},\n"
            f" partitions depended on: {self.dependencies},\n"
            f" partition dependents: {self.dependents}"
        )
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 44-69
```python
def _get_attr_from_qualname(mod: torch.nn.Module, qualname: str) -> Any:
    attr_val = mod
    for atom in qualname.split("."):  # type: ignore[union-attr]
        if not hasattr(attr_val, atom):
            raise AttributeError(f"Node target {qualname} not found!")
        attr_val = getattr(attr_val, atom)
    return attr_val


# Creates subgraphs out of main graph
@compatibility(is_backward_compatible=True)
def split_module(
    m: GraphModule,
    root_m: torch.nn.Module,
    split_callback: Callable[[Node], int],
    qualname_map: dict[str, str] | None = None,
    keep_original_order: bool | None = False,
    keep_original_node_name: bool | None = False,
    keep_original_input_name: bool = True,
    *,
    partition_affix: str | None = None,
    tuple_return: bool = False,
) -> GraphModule:
    """
    Creates subgraphs out of main graph
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 70-96
```python
    Args:
        m (GraphModule): Graph module to split
        root_m (torch.nn.Module): root nn module. Not currently used. Included
            because the root nn module is usually transformed via
            torch.fx._symbolic_trace.symbolic_trace (see example below)
        split_callback (Callable[[Node], int]): Callable function
            that maps a given Node instance to a numeric partition identifier.
            split_module will use this function as the policy for which operations
            appear in which partitions in the output Module.
        qualname_map: Optional[Dict[str, str]]: optional output parameter that returns a
            mapping from new target names in the module after split to old target
            names in the original module.
        keep_original_order: Optional[bool]: keep the original order of the GraphModule
            or use the Topological order of the new constructed GraphModule
        keep_original_node_name: Optional[bool]: If the partitioned graphs should
            have the same node names as the original graph.
        keep_original_input_name: bool: If the partitioned graphs should
            have the same input names as the original graph.
        partition_affix: Optional[str]: If specified, the submodules' names will contain
            the affix, e.g. "submod_<affix>_<idx>".
        tuple_return: bool: If True, submodule outputs are always wrapped in a tuple,
            even when there is only a single output value.  This makes all subgraphs
            conform to the convention expected by ``torch._inductor.compile_fx``.

    Returns:
        GraphModule: the module after split.
```
- **EN**: This block continues `split_module` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `split_module`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 97-121
```python
    Example:

        This is a sample setup:

            import torch
            from torch.fx._symbolic_trace import symbolic_trace
            from torch.fx.graph_module import GraphModule
            from torch.fx.node import Node
            from torch.fx.passes.split_module import split_module

            class MyModule(torch.nn.Module):
                def __init__(self) -> None:
                    super().__init__()
                    self.param = torch.nn.Parameter(torch.rand(3, 4))
                    self.linear = torch.nn.Linear(4, 5)

                def forward(self, x, y):
                    z = self.linear(x + self.param).clamp(min=0.0, max=1.0)
                    w = self.linear(y).clamp(min=0.0, max=1.0)
                    return z + w

            # symbolically trace model
            my_module = MyModule()
            my_module_traced = symbolic_trace(my_module)
```
- **EN**: Defines the `split_module` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`split_module` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 122-139
```python
            # random mod partitioning
            partition_counter = 0
            NPARTITIONS = 3

            def mod_partition(node: Node):
                global partition_counter
                partition = partition_counter % NPARTITIONS
                partition_counter = (partition_counter + 1) % NPARTITIONS
                return partition

            # split module in module with submodules
            module_with_submodules = split_module(
                my_module_traced, my_module, mod_partition
            )

        Output looks like this. Original graph is broken into partitions

            > print(module_with_submodules)
```
- **EN**: Defines the `split_module` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`split_module` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 140-163
```python
            GraphModule(
                (submod_0): GraphModule(
                    (linear): Linear(in_features=4, out_features=5, bias=True)
                )
                (submod_1): GraphModule(
                    (linear): Linear(in_features=4, out_features=5, bias=True)
                )
                (submod_2): GraphModule()
            )

            def forward(self, x, y):
                param = self.param
                submod_0 = self.submod_0(x, param, y);  x = param = y = None
                getitem = submod_0[0]
                getitem_1 = submod_0[1];  submod_0 = None
                submod_1 = self.submod_1(getitem, getitem_1);  getitem = getitem_1 = None
                getitem_2 = submod_1[0]
                getitem_3 = submod_1[1];  submod_1 = None
                submod_2 = self.submod_2(getitem_2, getitem_3);  getitem_2 = getitem_3 = None
                return submod_2

        Output of split module is the same as output of input traced module.
        This is an example within a test setting:
```
- **EN**: Defines the `split_module` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`split_module` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 164-181
```python
            > orig_out = my_module_traced(x, y)
            > submodules_out = module_with_submodules(x, y)
            > self.assertEqual(orig_out, submodules_out)
            True
    """

    log.debug(
        "%s",
        lazy_format_graph_code("pre split_module", m, colored=True),
    )

    def construct_graph(
        node: Node,
        base_mod_env: dict[str, Node],
        base_mod_attrs: dict[str, torch.fx.graph_module.GraphModule],
    ) -> tuple[dict[str, Node], dict[str, torch.fx.graph_module.GraphModule]]:
        if node.op == "placeholder":
            default_value = (
```
- **EN**: Defines the `split_module` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`split_module` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 182-199
```python
                node.args[0] if len(node.args) > 0 else inspect.Signature.empty
            )
            if keep_original_node_name:
                args = (
                    () if default_value is inspect.Signature.empty else (default_value,)
                )
                base_mod_env[node.name] = base_mod_graph.create_node(
                    "placeholder",
                    node.name,
                    args=args,  # type: ignore[arg-type]
                    type_expr=node.type,
                )
            else:
                base_mod_env[node.name] = base_mod_graph.placeholder(
                    node.target,  # type: ignore[arg-type]
                    type_expr=node.type,
                    default_value=default_value,
                )
```
- **EN**: This block continues `split_module.construct_graph` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module.construct_graph`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 200-221
```python
            base_mod_env[node.name].meta = node.meta.copy()
        elif node.op == "get_attr":
            base_mod_env[node.name] = base_mod_graph.get_attr(node.target)  # type: ignore[arg-type]
            base_mod_env[node.name].meta = node.meta.copy()
            if not isinstance(node.target, str):
                raise AssertionError(f"Expected str target, got {type(node.target)}")
            attr_val = _get_attr_from_qualname(m, node.target)
            base_mod_attrs[node.target] = attr_val  # type: ignore[index]
        return base_mod_env, base_mod_attrs

    import sympy

    partitions: dict[str, Partition] = {}
    orig_nodes: dict[str, Node] = {}
    symbol_to_node: dict[sympy.Symbol, Node] = {}

    def record_cross_partition_use(def_node: Node, use_node: Node | None) -> None:
        from torch.fx.experimental.symbolic_shapes import free_symbols

        defined = getattr(def_node, "_fx_partition", None)
        used = getattr(use_node, "_fx_partition", None)
```
- **EN**: Defines the `split_module` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`split_module` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 222-239
```python
        log.debug(
            "record_cross_partition_use %s (%s) %s (%s)",
            def_node.name,
            defined,
            use_node.name if use_node is not None else "-",
            used,
        )

        if defined != used:
            if defined is not None:
                def_partition = partitions[defined]
                def_partition.outputs.setdefault(def_node.name)
                if used is not None:
                    def_partition.dependents.setdefault(used)

            if used is not None:
                use_partition = partitions[used]
                use_partition.inputs.setdefault(def_node.name)
```
- **EN**: This block continues `split_module.record_cross_partition_use` and works to build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module.record_cross_partition_use`，用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 240-264
```python
                # We have made def_node an input to the use_partition.  If
                # this input has symbolic symbols in its size, those also must
                # be made as inputs to the partition
                if (def_val := def_node.meta.get("example_value")) is not None:
                    for s in sorted(free_symbols(def_val), key=str):
                        s_node = symbol_to_node[s]
                        use_partition.inputs.setdefault(s_node.name)
                        if symbol_to_node[s].op != "placeholder":
                            # If the node that defines the symbol is not a
                            # placeholder, we must make it an output of the
                            # partition.  Note that this may be in a different
                            # partition than defined!  Although, this doesn't
                            # really make a difference for correctness, since
                            # defined is guaranteed to have the symbol in
                            # scope and can return it; you just get less
                            # optimal codegen in this case.
                            s_defined = getattr(s_node, "_fx_partition", None)
                            if s_defined is not None:
                                s_def_partition = partitions[s_defined]
                                s_def_partition.outputs.setdefault(s_node.name)
                                s_def_partition.dependents.setdefault(used)
                                use_partition.dependencies.setdefault(s_defined)
                if defined is not None:
                    use_partition.dependencies.setdefault(defined)
```
- **EN**: This block continues `split_module` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 265-284
```python
    def instantiate_node_partition_mapping(node: Node) -> None:
        partition_idx = split_callback(node)
        partition_name = str(partition_idx)
        if partition_affix is not None:
            # For example, if user specifies partition_affix = "pp", then the
            # partition name will be "pp_0", "pp_1", etc
            partition_name = "_".join([partition_affix, partition_name])

        log.debug(
            "instantiate_node_partition_mapping %s (%s)", node.name, partition_name
        )

        # add node to partitions
        partition = partitions.get(partition_name)
        if partition is None:
            partitions[partition_name] = partition = Partition(partition_name)

        partition.node_names.append(node.name)
        node._fx_partition = partition_name
```
- **EN**: Defines the `split_module` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`split_module` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 285-311
```python
    # Global State Nodes are nodes which by their global state effects,
    # "taint" all downstream nodes while they are active.
    GLOBAL_STATE_NODES = [
        torch.amp._enter_autocast,
        torch.amp._exit_autocast,
        torch._C._set_grad_enabled,
    ]

    # For grad regions:
    # ------------------------
    # 1. first region: we do nothing
    # 2. subsequent regions: we insert the set_grad at the beginning
    grad_regions: OrderedDict[Node, set[int]] = OrderedDict()

    # For autocast regions:
    # ------------------------
    # 1. first region: we will only insert the _exit at the end
    # 2. intermediate regions: we will insert both the
    #    _enter at the beginning and _exit at the end
    # 3. last region: we will only insert _enter at the beginning
    # We will do so in the order in which the autocasts were instantiated.
    autocast_regions: OrderedDict[Node, set[int]] = OrderedDict()
    autocast_exits: dict[Node, Node | None] = {}

    active_grad = None
    active_autocasts = set()
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。

### Lines 312-332
```python
    for node in m.graph.nodes:
        # This will prefer placeholder bindings, because those come first.
        # This is a little dangerous though: it is possible that an unbacked
        # symbol is used without any binding site for it, in which case we
        # will get a KeyError not able to find it.  I'd like to fix this by
        # having passes.runtime_assert establish some invariants that I can
        # rely on later, but this needs some extra work.  Quick fix first.
        # See https://github.com/pytorch/pytorch/issues/130534
        if (
            (val := node.meta.get("example_value")) is not None
            and isinstance(val, (torch.SymInt, torch.SymFloat))
            and isinstance(s0 := val.node.expr, sympy.Symbol)
            and s0 not in symbol_to_node
        ):
            symbol_to_node[val.node.expr] = node

        if node.op in ["placeholder", "get_attr", "output"]:
            continue

        instantiate_node_partition_mapping(node)
```
- **EN**: This block continues `split_module` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 333-350
```python
        if node.op == "call_function" and node.target in GLOBAL_STATE_NODES:
            if node.target is torch._C._set_grad_enabled:
                if len(node.args) != 1:
                    raise AssertionError(
                        f"Expected 1 arg for _set_grad_enabled, got {len(node.args)}"
                    )
                if not isinstance(node.args[0], bool):
                    raise AssertionError(f"Expected bool arg, got {type(node.args[0])}")
                active_grad = node
                grad_regions[active_grad] = set({split_callback(node)})
            elif node.target is torch.amp._enter_autocast:
                # Should all be python constants
                if not all(not isinstance(arg, Node) for arg in node.args):
                    raise AssertionError(
                        "Expected all args to be python constants, not Nodes"
                    )
                active_autocasts.add(node)
                autocast_regions[node] = set({split_callback(node)})
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 351-374
```python
                autocast_exits[node] = None
            elif node.target is torch.amp._exit_autocast:
                if len(node.args) != 1:
                    raise AssertionError(
                        f"Expected 1 arg for _exit_autocast, got {len(node.args)}"
                    )
                autocast_regions[node.args[0]].add(split_callback(node))
                active_autocasts.remove(node.args[0])
                autocast_exits[node.args[0]] = node

        if active_grad is not None:
            grad_regions[active_grad].add(split_callback(node))

        for a in active_autocasts:
            autocast_regions[a].add(split_callback(node))

    if not all(v is not None for v in autocast_exits.values()):
        raise AssertionError("autocast must exit")

    # pyrefly: ignore [bad-assignment]
    autocast_regions = {k: sorted(v) for k, v in autocast_regions.items()}
    # pyrefly: ignore [bad-assignment]
    grad_regions = {k: sorted(v) for k, v in grad_regions.items()}
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 375-395
```python
    if _LOGGER.isEnabledFor(logging.DEBUG):
        _LOGGER.debug("autocast_regions: %s", autocast_regions)
        _LOGGER.debug("grad_regions: %s", grad_regions)

    assert_monotonically_increasing = bool(autocast_regions) or bool(grad_regions)

    # split nodes into partitions
    highest_partition = -1
    for node in m.graph.nodes:
        orig_nodes[node.name] = node

        # TODO currently placeholders/parameters aren't put into random partitions,
        # rather they're added to the graphs where they are used down below
        if node.op in ["placeholder", "get_attr"]:
            continue
        if node.op == "output":
            torch.fx.graph.map_arg(
                node.args[0], lambda n: record_cross_partition_use(n, None)
            )
            continue
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 396-421
```python
        if assert_monotonically_increasing:
            pid = split_callback(node)
            if highest_partition > pid:
                raise AssertionError(
                    "autocast or set_grad_enabled require monotonically increasing "
                    f"partitions: highest: {highest_partition}, this node's: {pid}"
                )
            highest_partition = pid

        # do not capture cross-partition dependencies for global state nodes as they will be
        # self-contained - their setup and unwind will be isolated to each partition submodule.
        if node.target not in GLOBAL_STATE_NODES:
            torch.fx.graph.map_arg(
                node.args, lambda def_node: record_cross_partition_use(def_node, node)
            )
            torch.fx.graph.map_arg(
                node.kwargs, lambda def_node: record_cross_partition_use(def_node, node)
            )

    original_partition_order = list(partitions.keys())
    # find partitions with no dependencies
    root_partitions: list[str] = []
    for partition_name, partition in partitions.items():
        if not len(partition.dependencies):
            root_partitions.append(partition_name)
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 422-439
```python
    # check partitions for circular dependencies and create topological partition ordering
    sorted_partitions: list[str] = []
    while root_partitions:
        root_partition = root_partitions.pop()
        sorted_partitions.append(root_partition)
        for dependent in partitions[root_partition].dependents:
            partitions[dependent].dependencies.pop(root_partition)  # noqa: B909
            if not partitions[dependent].dependencies:
                root_partitions.append(dependent)
    if len(sorted_partitions) != len(partitions):
        raise RuntimeError("cycle exists between partitions!")

    # Enter prelude
    for regions_mapping in [autocast_regions, grad_regions]:
        for node, regions in regions_mapping.items():
            if len(regions) == 0:
                raise AssertionError("Expected at least one region for node")
            # pyrefly: ignore [bad-index]
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 440-463
```python
            partitions[str(regions[0])].environment[node] = node
            # pyrefly: ignore [bad-index, index-error]
            # pyrefly: ignore [bad-index, index-error]
            for r in regions[1:]:
                partition = partitions[str(r)]
                new_node = partition.graph.create_node(
                    op=node.op,
                    target=node.target,
                    args=tuple(arg for arg in node.args),
                    kwargs={},
                    type_expr=node.type,
                )
                new_node.meta = (
                    node.meta.copy()
                )  # is it really a good idea to copy this?
                partition.environment[node] = new_node

    # add placeholders to partition inputs
    for partition_name in sorted_partitions:
        partition = partitions[partition_name]
        new_inputs: dict[str, None] = {}

        counter = 0
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构。

### Lines 464-488
```python
        for inp in partition.inputs:
            orig_node = orig_nodes[inp]
            # We don't pass in get_attr nodes as inputs to the partition, but
            # instead set them as targets and use getattr within the module

            def add_placeholder() -> Node:
                if keep_original_input_name:
                    name = inp
                else:
                    nonlocal counter
                    name = f"arg_{counter}"
                    counter += 1
                placeholder = partition.graph.placeholder(
                    name,
                    type_expr=orig_nodes[inp].type,
                )
                new_inputs[inp] = None
                return placeholder

            if orig_node.op == "get_attr":
                if not isinstance(orig_node.target, str):
                    raise AssertionError(
                        f"Expected str target, got {type(orig_node.target)}"
                    )
```
- **EN**: Defines the `split_module` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`split_module` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 489-512
```python
                orig_attr = _get_attr_from_qualname(m, orig_node.target)
                if isinstance(orig_attr, torch.nn.Module):
                    placeholder = partition.graph.get_attr(orig_node.target)
                    partition.targets[orig_node.target] = orig_attr
                else:
                    placeholder = add_placeholder()
            else:
                placeholder = add_placeholder()
            placeholder.meta = orig_nodes[inp].meta.copy()
            partition.environment[orig_nodes[inp]] = placeholder
        partition.inputs = new_inputs

    # Transform nodes and collect targets for partition's submodule
    for node in m.graph.nodes:
        if hasattr(node, "_fx_partition"):
            partition = partitions[node._fx_partition]

            # swap out old graph nodes in kw/args with references to new nodes in this submodule
            environment = partition.environment
            gathered_args = torch.fx.graph.map_arg(node.args, lambda n: environment[n])
            gathered_kwargs = torch.fx.graph.map_arg(
                node.kwargs, lambda n: environment[n]
            )
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 513-530
```python
            if node.op not in ["call_module", "get_attr"]:
                target = node.target
            else:
                target_attr = _get_attr_from_qualname(m, node.target)
                target = node.target.replace(".", "_")
                partition.targets[target] = target_attr
                # Fill in the passed-in mapping from new qualname to old qualname
                if qualname_map is not None:
                    # When creating the split module later, the submodules will have
                    # path prefix matching the corresponding partition's submod_name
                    qualname = f"{partition.submod_name}.{target}"
                    qualname_map[qualname] = node.target

            if not isinstance(gathered_args, tuple):
                raise AssertionError(
                    f"Expected tuple for gathered_args, got {type(gathered_args)}"
                )
            if not isinstance(gathered_kwargs, dict):
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 531-548
```python
                raise AssertionError(
                    f"Expected dict for gathered_kwargs, got {type(gathered_kwargs)}"
                )
            name = node.name if keep_original_node_name else None
            new_node = partition.graph.create_node(
                op=node.op,
                target=target,
                args=gathered_args,
                kwargs=gathered_kwargs,
                type_expr=node.type,
                name=name,
            )
            new_node.meta = node.meta.copy()
            partition.environment[node] = new_node

    # Exit epilogue
    for regions_mapping in [autocast_regions]:
        for node in reversed(regions_mapping):
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 549-568
```python
            regions = regions_mapping[node]
            if len(regions) == 0:
                raise AssertionError("Expected at least one region")
            # pyrefly: ignore [bad-index, index-error]
            for r in regions[:-1]:
                partition = partitions[str(r)]
                exit_node = autocast_exits[node]
                if exit_node is None:
                    raise AssertionError("Missing exit node")
                new_node = partition.graph.create_node(
                    op=exit_node.op,
                    target=exit_node.target,
                    args=(partition.environment[node],),
                    kwargs={},
                    type_expr=exit_node.type,
                )
                new_node.meta = (
                    exit_node.meta.copy()
                )  # is it really a good idea to copy this?
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 569-595
```python
    # original module environment dict mapping node names to nodes
    orig_mod_env: dict[str, Node] = {}
    # Set up values to construct base module
    base_mod_env: dict[str, Node] = {}
    base_mod_graph: torch.fx.graph.Graph = torch.fx.graph.Graph()
    base_mod_attrs: dict[str, torch.fx.graph_module.GraphModule] = {}
    if not keep_original_order:
        for node in m.graph.nodes:
            base_mod_env, base_mod_attrs = construct_graph(
                node, base_mod_env, base_mod_attrs
            )

    else:
        # Go through the graph to construct the mapping dict
        for node in m.graph.nodes:
            orig_mod_env[node.name] = node

    # Do some things iterating over the partitions in topological order again:
    # 1) Finish off submodule Graphs by setting corresponding outputs
    # 2) Construct GraphModules for each submodule
    # 3) Construct the base graph by emitting calls to those submodules in
    #    topological order or original order specified by keep_original_order

    construct_order_partitions = (
        sorted_partitions if not keep_original_order else original_partition_order
    )
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 596-622
```python
    already_constructed_attr_nodes = set()

    # We actually need to insert the placeholder nodes in the original order
    # otherwise graph signature will be wrong.
    original_order = [node for node in m.graph.nodes if node.op == "placeholder"]

    for partition_name in construct_order_partitions:
        partition = partitions[partition_name]

        # Set correct output values
        output_vals = tuple(
            partition.environment[orig_nodes[name]] for name in partition.outputs
        )

        if len(output_vals) == 1 and not tuple_return:
            partition.graph.output(output_vals[0])
        else:
            partition.graph.output(output_vals)

        if keep_original_order:
            # first get the attr nodes required by this partition
            orig_mod_attr_nodes: list[Node] = [
                orig_mod_env[key]
                for key in partition.inputs
                if key not in original_order
            ]
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 623-649
```python
            for node in original_order:
                if node in already_constructed_attr_nodes:
                    continue  # already added this attr to the base graph
                base_mod_env, _based_mod_attrs = construct_graph(
                    node, base_mod_env, base_mod_attrs
                )
                already_constructed_attr_nodes.add(node)

            # Construct GraphModule for this partition
            for node in orig_mod_attr_nodes:  # type: ignore[attr-defined]
                if node in already_constructed_attr_nodes:
                    continue
                base_mod_env, base_mod_attrs = construct_graph(
                    node, base_mod_env, base_mod_attrs
                )
                already_constructed_attr_nodes.add(node)

        base_mod_attrs[partition.submod_name] = _make_graph_module(
            partition.targets, partition.graph
        )

        # Emit call in base graph to this submodule
        output_val = base_mod_graph.call_module(
            partition.submod_name,
            tuple(base_mod_env[name] for name in partition.inputs),
        )
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 650-675
```python
        num_outputs = len(partition.outputs)
        if num_outputs > 1 or (num_outputs == 1 and tuple_return):
            # Unpack return values from submodule
            output_val_proxy = torch.fx.proxy.Proxy(output_val)
            for i, output_name in enumerate(partition.outputs):
                base_mod_env[output_name] = output_val_proxy[i].node  # type: ignore[index]
        elif num_outputs == 1:
            base_mod_env[next(iter(partition.outputs))] = output_val

    # When keep_original_order=True and if the graph doesn't have any
    # `call_function` node then `base_mod_graph`, `base_mod_env` and `base_mod_attrs`
    # are never populated.
    # For this case, we call `construct_graph` here which takes care of updating them.
    if keep_original_order and not base_mod_env:
        for node in m.graph.nodes:
            base_mod_env, base_mod_attrs = construct_graph(
                node, base_mod_env, base_mod_attrs
            )

    # Add output node to `base_mod_graph` (i.e. the split graph) which will be returned.
    for node in m.graph.nodes:
        if node.op == "output":
            base_mod_graph.output(
                torch.fx.graph.map_arg(node.args[0], lambda n: base_mod_env[n.name])
            )
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 676-681
```python
    ret = _make_graph_module(base_mod_attrs, base_mod_graph)
    log.debug(
        "%s",
        lazy_format_graph_code("post split_module", ret, colored=True),
    )
    return ret
```
- **EN**: This block continues `split_module` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `split_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

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
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx._compatibility`, `torch.fx._lazy_graph_module`, `torch.fx._utils`, `torch.fx.graph_module`, `torch.fx.node`, `torch.fx.experimental.symbolic_shapes`
- **Standard library / 标准库**: `inspect`, `logging`, `collections`, `collections.abc`, `typing`
- **Third-party packages / 第三方包**: `sympy`
- **Primary symbols / 核心符号**: `__all__`, `_LOGGER`, `Partition`, `_get_attr_from_qualname`, `split_module`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
