# const_fold.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/const_fold.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
```python
import re
from collections.abc import Callable
from typing import Any

import torch.fx
from torch.fx.node import map_arg
from torch.fx.passes.split_module import split_module


__all__ = [
    "FoldedGraphModule",
    "get_unique_attr_name_in_module",
    "split_const_subgraphs",
]


class FoldedGraphModule(torch.fx.GraphModule):
    """
    FoldedGraphModule is a GraphModule which also contains another
    `const_subgraph_module` representing a subgraph which has all const attr
    inputs and which can be run once before running the main standard
    `graph`. The `const_output_names` are the ordered list names of attrs which
    represent what each respective output from the const_subgraph should be set
    on which attrs.
    """
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 27-49
```python
    def __init__(
        self,
        root: torch.nn.Module,
        graph: torch.fx.Graph,
        const_subgraph: torch.fx.Graph | None = None,
        fx_const_folded_attrs_name: str | None = None,
        device_for_folded_attrs: str = "cuda",
    ) -> None:
        super().__init__(root, graph)
        self.const_subgraph_module = (
            None
            if const_subgraph is None
            else torch.fx.GraphModule(root, const_subgraph)
        )
        self.has_folding_been_run = False
        self.fx_const_folded_attrs_name = fx_const_folded_attrs_name
        self.device_for_folded_attrs = device_for_folded_attrs

    def __call__(self, *args: object, **kwargs: object) -> Any:
        if not self.has_folding_been_run:
            self.run_folding()
        return super().__call__(*args)
```
- **EN**: Declares `FoldedGraphModule(torch.fx.GraphModule)`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `FoldedGraphModule(torch.fx.GraphModule)`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 50-75
```python
    def run_folding(self) -> None:
        # If there's no const subgraph module or attr output names to use, return
        # early as there is no const folding to perform.
        if (
            self.const_subgraph_module is None
            or self.fx_const_folded_attrs_name is None
        ):
            return

        if self.has_folding_been_run:
            raise AssertionError("Folding has already been run")
        self.has_folding_been_run = True

        # Actually run const folding subgraph. Note that single attr const fold
        # subgraphs output a single Tensor while multiple outputs are returned as
        # Tuple[Tensor,].
        folded_attrs = self.const_subgraph_module()

        def _create_param(i: torch.Tensor | int) -> torch.nn.Parameter:
            return torch.nn.Parameter(
                i.detach().clone()
                if not isinstance(i, int)
                else torch.Tensor([i]).to(device=self.device_for_folded_attrs),
                requires_grad=i.requires_grad if isinstance(i, torch.Tensor) else False,
            )
```
- **EN**: Defines the `FoldedGraphModule.run_folding` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`FoldedGraphModule.run_folding` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 76-93
```python
        params = (
            torch.nn.ParameterList([_create_param(i) for i in folded_attrs])
            if isinstance(folded_attrs, tuple)
            else _create_param(folded_attrs)
        )
        setattr(self, self.fx_const_folded_attrs_name, params)


def _inline_module(
    gm: torch.fx.GraphModule, inline_mod_name: str, run_dce: bool = True
) -> dict[torch.fx.Node, torch.fx.Node]:
    """
    Given `gm` and some graph module which is called with target name `inline_mod_name`,
    this helper will inline all of the nodes from that called graph module into `gm`.

    Returns a mapping from subgraph nodes to the newly created/mapped nodes in gm.
    """
    # Fetch the inner graph module that we want to inline inside `gm`.
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会根据运行时条件分支处理。

### Lines 94-117
```python
    inline_mod = dict(gm.named_modules())[inline_mod_name]
    if not isinstance(inline_mod, torch.fx.GraphModule):
        raise AssertionError(f"Expected GraphModule, got {type(inline_mod)}")
    call_mod_node_to_replace = None
    for node in gm.graph.nodes:
        if node.op == "call_module" and node.target == inline_mod_name:
            call_mod_node_to_replace = node
            break
    if call_mod_node_to_replace is None:
        raise AssertionError(f"Could not find call_module node for {inline_mod_name}")

    # Now actually do the swap. Note that we have to keep track of new nodes that are
    # copied into `gm` -- we do this via replacement_mapping.
    call_mod_args = call_mod_node_to_replace.args
    call_mod_kwargs = call_mod_node_to_replace.kwargs

    replacement_mapping: dict[torch.fx.Node, torch.fx.Node] = {}
    ph_count = 0

    def replacement_fn(node: torch.fx.Node) -> torch.fx.Node:
        new_node = replacement_mapping[node]
        new_node.meta = node.meta.copy()
        return new_node
```
- **EN**: Defines the `_inline_module` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`_inline_module` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 118-138
```python
    for inline_node in inline_mod.graph.nodes:
        if inline_node.op == "placeholder":
            replacement_mapping[inline_node] = (
                call_mod_kwargs[inline_node.name]
                if inline_node.name in call_mod_kwargs
                else call_mod_args[ph_count]
            )

            ph_count += 1
            continue

        if inline_node.op == "output":
            outputs = inline_node.args[0]
            output_replacements = map_arg(outputs, replacement_fn)

            # If output is a tuple, we need to handle getitem users specially.
            # Capture users before replace_all_uses_with modifies them.
            getitem_users: list[torch.fx.Node] = []
            if isinstance(output_replacements, (list, tuple)):
                import operator
```
- **EN**: This block continues `_inline_module` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_inline_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 139-163
```python
                getitem_users = [
                    user
                    for user in call_mod_node_to_replace.users
                    if user.op == "call_function"
                    and user.target is operator.getitem
                    and isinstance(user.args[1], int)
                ]

            call_mod_node_to_replace.replace_all_uses_with(output_replacements)

            # Inline getitem nodes that now index into the tuple literal
            for user in getitem_users:
                idx = user.args[1]
                if not isinstance(idx, int):
                    raise AssertionError(f"Expected int index, got {type(idx)}")
                user.replace_all_uses_with(output_replacements[idx])
                gm.graph.erase_node(user)
                replacement_mapping[user] = output_replacements[idx]

            continue

        with gm.graph.inserting_before(call_mod_node_to_replace):
            new_node = gm.graph.node_copy(inline_node, replacement_fn)
        replacement_mapping[inline_node] = new_node
```
- **EN**: This block continues `_inline_module` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_inline_module`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 164-190
```python
    # Explicitly remove the module that was just inlined,
    # this module may contain impure ops so cannot be dead code eliminated,
    # this module is unneeded as it's just inlined back to main graph.
    gm.graph.erase_node(call_mod_node_to_replace)
    if run_dce:
        gm.graph.eliminate_dead_code()

    return replacement_mapping


def get_unique_attr_name_in_module(mod_traced: torch.fx.GraphModule, name: str) -> str:
    """
    Make sure the name is unique (in a module) and can represents an attr.
    """
    # Delete all characters that are illegal in a Python identifier.
    name = re.sub("[^0-9a-zA-Z_]+", "_", name)
    if name[0].isdigit():
        name = f"_{name}"
    # Now make sure it is in fact unique to the module by incrementing suffix value.
    while hasattr(mod_traced, name):
        match = re.match(r"(.*)_(\d+)$", name)
        if match is None:
            name = name + "_1"
        else:
            base, num = match.group(1, 2)
            name = f"{base}_{int(num) + 1}"
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 191-213
```python
    return name


def split_const_subgraphs(
    module: torch.nn.Module | torch.fx.GraphModule,
    skip_folding_node_fn: Callable[[torch.fx.Node], bool] | None = None,
    device_for_folded_attrs: str = "cpu",
) -> FoldedGraphModule:
    """
    Looks through `module` for any nodes that have all constant attribute inputs
    and separates them out into their own constant subgraph, and returns a
    FoldedGraphModule which runs that constant subgraph on the first run to set
    attributes on the module prior to running the non-constant portion of the
    graph.
    """

    import sympy

    if not isinstance(module, torch.fx.GraphModule):
        mod_traced = torch.fx.symbolic_trace(module)
    else:
        mod_traced = module
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 214-233
```python
    def _subgraph_has_impure_ops(module: torch.fx.GraphModule) -> bool:
        """
        Return True if a GraphModule type subgraph contains any impure op, else False.
        """
        if not isinstance(module, torch.fx.GraphModule):
            raise AssertionError(
                "caller should only pass GraphModule to subgraph_has_impure_ops check"
            )
        for node in module.graph.nodes:
            if node.op == "call_function" and node.is_impure():
                return True
            if (
                node.op == "call_module"
                # pyrefly: ignore [not-callable]
                and (submodule := module.get_submodule(node.target))
                and isinstance(submodule, torch.fx.GraphModule)
            ):
                return _subgraph_has_impure_ops(submodule)
        return False
```
- **EN**: Defines the `split_const_subgraphs` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`split_const_subgraphs` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 234-258
```python
    # Build up a list of const_nodes, defined as nodes that are themselves
    # get_attrs, or have all get_attr or other constant node inputs.
    const_nodes: set[torch.fx.Node] = set()
    found_const_folding = False
    for node in mod_traced.graph.nodes:
        # Skip over placeholders/outputs because they can't be const folded and
        # we don't want to add tags to them.
        if node.op in {"placeholder", "output"}:
            continue

        # If the node itself is constant, or all of its inputs are constant,
        # then tag it as constant.
        if node.op != "get_attr" and not set(node.all_input_nodes).issubset(
            const_nodes
        ):
            continue

        # If provided skip folding function says to skip, then skip.
        if skip_folding_node_fn and skip_folding_node_fn(node):
            continue

        # Skip folding side-effectful functions
        if node.is_impure():
            continue
```
- **EN**: This block continues `split_const_subgraphs` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_const_subgraphs`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 259-281
```python
        # Skip folding nodes that have symbolic fill_value
        if isinstance(node.kwargs.get("fill_value", None), sympy.Expr):
            continue

        # Skip folding submodules that have impure ops
        if (
            node.op == "call_module"
            # pyrefly: ignore [not-callable]
            and (target_mod := mod_traced.get_submodule(node.target))
            and isinstance(target_mod, torch.fx.GraphModule)
            and _subgraph_has_impure_ops(target_mod)
        ):
            continue

        # Must be a constant foldable node at this point.
        const_nodes.add(node)
        if node.op != "get_attr":
            found_const_folding = True

    # If we did not find any const folding then return early without a const fold subgraph.
    if not found_const_folding:
        return FoldedGraphModule(mod_traced, mod_traced.graph)
```
- **EN**: This block continues `split_const_subgraphs` and works to track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_const_subgraphs`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 282-304
```python
    # Partition the module into two: submod_0 for constant folding subgraph, and
    # submod_1 for the rest.
    def mod_partition(node: torch.fx.Node) -> int:
        return 0 if node in const_nodes else 1

    split = split_module(mod_traced, module, mod_partition)

    const_mod_name, non_const_mod_name = "submod_0", "submod_1"
    # Safely get submod_1 in case there are no non-const nodes
    const_gm = getattr(split, const_mod_name)
    if not isinstance(const_gm, torch.fx.GraphModule):
        raise AssertionError(
            f"Expected GraphModule for {const_mod_name}, got {type(const_gm)}"
        )
    non_const_mod = getattr(split, non_const_mod_name, None)
    non_const_gm: torch.fx.GraphModule | None = None
    if non_const_mod is not None:
        if not isinstance(non_const_mod, torch.fx.GraphModule):
            raise AssertionError(
                f"Expected GraphModule for {non_const_mod_name}, got {type(non_const_mod)}"
            )
        non_const_gm = non_const_mod
```
- **EN**: Defines the `split_const_subgraphs` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`split_const_subgraphs` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 305-330
```python
    # The module that a call_module node refers to gets copied to submodules during split.
    # The path to the module also gets inlined, i.e. mod.a.b -> mod_a_b. Here we need to
    # attach inlined modules to `split` as it's the owning module now.
    if non_const_gm is not None:
        for node in non_const_gm.graph.nodes:
            if node.op == "call_module":
                setattr(split, node.target, getattr(non_const_gm, node.target))
    for node in const_gm.graph.nodes:
        if node.op == "call_module":
            setattr(split, node.target, getattr(const_gm, node.target))

    # split_module currently does not use get_attrs for attrs. Instead it passes
    # them in as args from the parent module, which used get_attrs. Here we set
    # them as get_attrs inside const_gm, allowing for running folding without
    # somehow a priori knowing the attrs that should be passed as args. We can
    # unconditionally do this for all placeholders because we know all
    # placeholders to const_gm must be constants accessible via get_attr.
    call_const_gm_args = None
    for node in split.graph.nodes:
        if node.op == "call_module":
            if node.target == const_mod_name:
                call_const_gm_args = node.args
                break
    if call_const_gm_args is None:
        raise AssertionError("Could not find call_module node for const_gm")
```
- **EN**: This block continues `split_const_subgraphs` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_const_subgraphs`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 331-348
```python
    # Here we do the actual replacement of placeholders to get_attrs. Note that here we
    # set the const_gm.graph into a new root_const_gm with split as the root module,
    # because we are fetching attributes directly from the root module, instead of
    # fetching them from const_gm. Example: The const_gm must have some format like:
    # graph():
    #    %inp : [num_users=1] = placeholder[target=const_inp]
    #    %add : [num_users=1] = call_function[target=operator.add](args = (%inp, %inp), kwargs = {})
    #    return add
    # We replace that with the following, which does not have any placeholders:
    # graph():
    #    %inp_1 : [num_users=1] = get_attr[target=const_inp]
    #    %add : [num_users=1] = call_function[target=operator.add](args = (%inp_1, %inp_1), kwargs = {})
    #    return add
    root_const_gm = torch.fx.GraphModule(split, const_gm.graph)

    # The order of placeholders in the const_gm graph should match the order of
    # args in the outer module, so we can simply use an index for the
    # placeholder mapping
```
- **EN**: This block continues `split_const_subgraphs` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `split_const_subgraphs`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 349-372
```python
    ph_idx = 0
    for node in root_const_gm.graph.nodes:
        if node.op == "output":
            multiple_outputs = isinstance(node.args[0], tuple)
            continue
        if node.op != "placeholder":
            continue
        if ph_idx >= len(call_const_gm_args):
            raise AssertionError(
                f"Placeholder index {ph_idx} out of range for args "
                f"(len={len(call_const_gm_args)})"
            )
        in_node = call_const_gm_args[ph_idx]
        ph_idx += 1
        if in_node.op != "get_attr":
            raise AssertionError(f"Expected get_attr, got {in_node.op}")
        with root_const_gm.graph.inserting_before(node):
            new_node = root_const_gm.graph.get_attr(in_node.target)
        new_node.meta = node.meta.copy()
        node.replace_all_uses_with(new_node)
        root_const_gm.graph.erase_node(node)
    if "multiple_outputs" not in locals():
        raise AssertionError("multiple_outputs not set in loop")
```
- **EN**: This block continues `split_const_subgraphs` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `split_const_subgraphs`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 373-398
```python
    # Now find the call to const_gm inside split, and replace it with a getattr to the
    # folded tensor(s) that result from constant folding. Note that we don't need to
    # worry about whether this is one or more tensors because the original graph
    # correctly uses getitem to extract individual tensors if there are multiple folded.
    fx_const_folded_attrs_name = get_unique_attr_name_in_module(
        mod_traced, "_FX_CONST_FOLDED_ATTRS"
    )
    setattr(
        split,
        fx_const_folded_attrs_name,
        torch.nn.ParameterList() if multiple_outputs else torch.nn.Parameter(),  # type: ignore[possibly-undefined]
    )
    for node in split.graph.nodes:
        if node.op == "call_module" and node.target == const_mod_name:
            with node.graph.inserting_before(node):
                folded_attrs = node.graph.get_attr(fx_const_folded_attrs_name)
            folded_attrs.meta = node.meta.copy()
            node.replace_all_uses_with(folded_attrs)
            break

    # Finally, inline the non-constant submod (if it exists) into the split submod.
    # This is so that the original caller who may have passed in a graph module will
    # get back out a graph module whose graph is traced to the same granularity.
    if hasattr(split, non_const_mod_name):
        _inline_module(split, non_const_mod_name)
```
- **EN**: This block continues `split_const_subgraphs` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `split_const_subgraphs`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 399-407
```python
    split.graph.eliminate_dead_code()

    return FoldedGraphModule(
        split,
        split.graph,
        root_const_gm.graph,
        fx_const_folded_attrs_name,
        device_for_folded_attrs,
    )
```
- **EN**: This block continues `split_const_subgraphs` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `split_const_subgraphs`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch.fx`, `torch.fx.node`, `torch.fx.passes.split_module`
- **Standard library / 标准库**: `re`, `collections.abc`, `typing`, `operator`
- **Third-party packages / 第三方包**: `sympy`
- **Primary symbols / 核心符号**: `__all__`, `FoldedGraphModule`, `_inline_module`, `get_unique_attr_name_in_module`, `split_const_subgraphs`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
