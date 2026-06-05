# optimization.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/optimization.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
```python
import copy
import logging
import operator
import time
from collections import defaultdict
from collections.abc import Callable, Iterable
from enum import Enum
from typing import Any, cast

import torch
import torch.fx as fx
import torch.nn as nn
import torch.nn.functional as F
import torch.utils.mkldnn as th_mkldnn
from torch.fx.node import Argument, Target
from torch.fx.passes.shape_prop import ShapeProp
from torch.nn.utils.fusion import fuse_conv_bn_eval, fuse_linear_bn_eval
```
- **EN**: This import block pulls in the dependencies used by later definitions, including `copy`, `logging`, `operator`, `time`.
- **CN**: 这一导入块引入后续定义所需的依赖，其中包括 `copy`, `logging`, `operator`, `time`。

### Lines 20-44
```python
__all__ = [
    "matches_module_pattern",
    "replace_node_module",
    "fuse",
    "remove_dropout",
    "extract_subgraph",
    "modules_to_mkldnn",
    "reset_modules",
    "MklSubgraph",
    "gen_mkl_autotuner",
    "use_mkl_length",
    "UnionFind",
    "optimize_for_inference",
]


def _parent_name(target: str) -> tuple[str, str]:
    """
    Splits a qualname into parent path and last atom.
    For example, `foo.bar.baz` -> (`foo.bar`, `baz`)
    """
    *parent, name = target.rsplit(".", 1)
    return parent[0] if parent else "", name
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 45-65
```python
# Works for length 2 patterns with 2 modules
def matches_module_pattern(
    pattern: Iterable[type], node: fx.Node, modules: dict[str, torch.nn.Module]
) -> bool:
    if len(node.args) == 0:
        return False
    nodes: tuple[Argument, fx.Node] = (node.args[0], node)
    for expected_type, current_node in zip(pattern, nodes):
        if not isinstance(current_node, fx.Node):
            return False
        if current_node.op != "call_module":
            return False
        if not isinstance(current_node.target, str):
            return False
        if current_node.target not in modules:
            return False
        if type(modules[current_node.target]) is not expected_type:
            return False
    return True
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 66-83
```python
def replace_node_module(
    node: fx.Node, modules: dict[str, torch.nn.Module], new_module: torch.nn.Module
) -> None:
    if not isinstance(node.target, str):
        raise AssertionError(f"Expected str target, got {type(node.target)}")
    parent_name, name = _parent_name(node.target)
    modules[node.target] = new_module
    setattr(modules[parent_name], name, new_module)


def fuse(
    model: torch.nn.Module, inplace: bool = False, no_trace: bool = False
) -> torch.nn.Module:
    """
    Fuses convolution/BN and linear/BN layers for inference purposes.
    Will deepcopy your model by default, but can modify the model inplace as well.
    """
    patterns = [
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 84-101
```python
        (nn.Conv1d, nn.BatchNorm1d),
        (nn.Conv2d, nn.BatchNorm2d),
        (nn.Conv3d, nn.BatchNorm3d),
        (nn.Linear, nn.BatchNorm1d),
    ]
    if not inplace:
        model = copy.deepcopy(model)
    if not no_trace or not isinstance(model, torch.fx.GraphModule):
        fx_model = fx.symbolic_trace(model)
    else:
        fx_model = model
    modules = dict(fx_model.named_modules())
    new_graph = copy.deepcopy(fx_model.graph)

    for pattern in patterns:
        for node in new_graph.nodes:
            if matches_module_pattern(pattern, node, modules):
                if len(node.args[0].users) > 1:
```
- **EN**: This block continues `fuse` and works to track symbolic shape constraints and shape-dependent reasoning. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `fuse`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 102-123
```python
                    # Output of conv/linear is used by other nodes
                    continue
                first_layer = modules[node.args[0].target]
                bn = modules[node.target]
                if not bn.track_running_stats:
                    continue
                if pattern[0] in [nn.Conv1d, nn.Conv2d, nn.Conv3d]:
                    fused_layer = fuse_conv_bn_eval(first_layer, bn)
                else:  # nn.Linear
                    fused_layer = fuse_linear_bn_eval(first_layer, bn)
                replace_node_module(node.args[0], modules, fused_layer)
                node.replace_all_uses_with(node.args[0])
                new_graph.erase_node(node)
    return fx.GraphModule(fx_model, new_graph)


def remove_dropout(model: nn.Module) -> nn.Module:
    """
    Removes all dropout layers from the module.
    """
    fx_model = fx.symbolic_trace(model)
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 124-141
```python
    class DropoutRemover(torch.fx.Transformer):
        def call_module(
            self, target: Target, args: tuple[Argument, ...], kwargs: dict[str, Any]
        ) -> Any:
            if isinstance(self.submodules[target], nn.Dropout):
                if len(args) != 1:
                    raise AssertionError(f"Expected 1 arg for Dropout, got {len(args)}")
                return args[0]
            else:
                return super().call_module(target, args, kwargs)

    return DropoutRemover(fx_model).transform()


def extract_subgraph(
    orig_module: nn.Module,
    nodes: list[fx.Node],
    inputs: list[fx.Node],
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 142-159
```python
    outputs: list[fx.Node],
) -> fx.GraphModule:
    """
    Given lists of nodes from an existing graph that represent a subgraph, returns a submodule that executes that subgraph.
    """
    new_graph = fx.Graph()
    env: dict[fx.Node, fx.Node] = {}
    for input in inputs:
        new_node = new_graph.placeholder(input.name)
        env[input] = new_node
    for node in nodes:
        new_node = new_graph.node_copy(node, lambda x: env[x])
        env[node] = new_node
    new_graph.output([env[output] for output in outputs])
    new_graph.lint()
    return fx.GraphModule(orig_module, new_graph)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 160-186
```python
mkldnn_supported = [
    nn.Conv2d,
    nn.Linear,
    nn.BatchNorm2d,
    nn.ReLU,
    nn.MaxPool2d,
    nn.AvgPool2d,
    nn.AdaptiveAvgPool2d,
    torch.relu,
    torch.transpose,
    torch.sigmoid,
    F.relu,
    F.avg_pool2d,
    F.adaptive_avg_pool2d,
]
# These are operators that may not be convertible into MKLDNN ops (e.g. the
# args are scalar values). Thus, we only include them in the subgraph if their
# arguments are already in MKLDNN.
# TODO: Determine whether this can be removed after type inference.
mkldnn_supported_unknown = [operator.add, operator.mul]
mkldnn_map = {
    nn.Conv2d: th_mkldnn.MkldnnConv2d,
    nn.Linear: th_mkldnn.MkldnnLinear,
    nn.BatchNorm2d: lambda a, _: th_mkldnn.MkldnnBatchNorm(a),
}
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。

### Lines 187-210
```python
def modules_to_mkldnn(
    nodes: list[fx.Node], modules: dict[str, nn.Module]
) -> dict[nn.Module, nn.Module]:
    """
    For each node, if it's a module that can be preconverted into MKLDNN,
    then we do so and create a mapping to allow us to convert from the MKLDNN
    version of the module to the original.
    """
    old_modules: dict[nn.Module, nn.Module] = {}
    for node in nodes:
        if node.op == "call_module":
            if not isinstance(node.target, str):
                raise AssertionError(f"Expected str target, got {type(node.target)}")
            cur_module = modules[node.target]
            if type(cur_module) in mkldnn_map:
                # pyrefly: ignore [bad-index, index-error]
                new_module = mkldnn_map[type(cur_module)](cur_module, torch.float)
                if not isinstance(new_module, nn.Module):
                    raise AssertionError(f"Expected nn.Module, got {type(new_module)}")
                old_modules[new_module] = copy.deepcopy(cur_module)
                replace_node_module(node, modules, new_module)
    return old_modules
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 211-236
```python
def reset_modules(
    nodes: Iterable[fx.Node],
    modules: dict[str, nn.Module],
    old_modules: dict[nn.Module, nn.Module],
) -> None:
    """
    Maps each module that's been changed with `modules_to_mkldnn` back to its
    original.
    """
    for node in nodes:
        if node.op == "call_module":
            if not isinstance(node.target, str):
                raise AssertionError(f"Expected str target, got {type(node.target)}")
            cur_module = modules[node.target]
            if cur_module in old_modules:
                replace_node_module(node, modules, old_modules[cur_module])


class MklSubgraph:
    def __init__(self, fx_graph: fx.Graph) -> None:
        self.fx_graph = fx_graph
        self.nodes: list[fx.Node] = []
        self.start_nodes: list[fx.Node] = []
        self.end_nodes: list[fx.Node] = []
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 237-261
```python
def gen_mkl_autotuner(
    example_inputs: list[torch.Tensor], iters: int = 10, warmup: int = 1
) -> Callable[[MklSubgraph], bool]:
    """
    This generates a heuristic that can be passed into `optimize_for_inference` that
    determines whether a subgraph should be run in MKL by running it with the example_inputs.

    Example usage:
        heuristic = gen_mkl_autotuner(example_inputs, iters=10)
        fast_model = optimization.optimize_for_inference(model, heuristic)
    """
    fx_model = None
    old_modules = None

    def use_mkl_heuristic(graph: MklSubgraph) -> bool:
        nonlocal fx_model, old_modules
        input_nodes = graph.start_nodes
        if fx_model is None:
            fx_model = graph.fx_graph.owning_module
            old_modules = graph.fx_graph.old_modules  # type: ignore[attr-defined]
            ShapeProp(fx_model).propagate(example_inputs)
        sample_inputs = [torch.randn(node.shape) for node in input_nodes]  # type: ignore[attr-defined]
        output_args = cast(list[fx.Node], [node.args[0] for node in graph.end_nodes])
        submodule = extract_subgraph(fx_model, graph.nodes, input_nodes, output_args)
```
- **EN**: Defines the `gen_mkl_autotuner` function; this block introduces logic that track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 定义`gen_mkl_autotuner` 函数；该代码块引入了用于跟踪符号形状约束以及依赖形状的推理的逻辑。

### Lines 262-287
```python
        def benchmark(f: Callable[[], object]) -> float:
            for _ in range(warmup):
                f()
            begin = time.time()
            for _ in range(iters):
                f()
            return time.time() - begin

        mkl_time = benchmark(
            lambda: [
                i.to_dense() for i in submodule(*[i.to_mkldnn() for i in sample_inputs])
            ]
        )

        reset_modules(
            submodule.graph.nodes,
            dict(submodule.named_modules()),
            # pyrefly: ignore [bad-argument-type]  # old_modules is set before this point
            old_modules,
        )
        no_mkl_time = benchmark(lambda: submodule(*sample_inputs))
        return mkl_time < no_mkl_time

    return use_mkl_heuristic
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 288-314
```python
def use_mkl_length(graph: MklSubgraph) -> bool:
    """
    This is a heuristic that can be passed into `optimize_for_inference` that
    determines whether a subgraph should be run in MKL by checking if there
    are more than 2 nodes in it
    """
    return len(graph.nodes) > 2


class UnionFind:
    def __init__(self, n: int) -> None:
        self.parent: list[int | None] = [None] * n
        self.size: list[int] = [0] * n

    def make_set(self, v: int) -> None:
        self.parent[v] = v
        self.size[v] = 1

    def find(self, v: int) -> int:
        par = self.parent[v]
        if v == par:
            return v
        if par is None:
            raise AssertionError("Parent is None")
        self.parent[v] = self.find(par)
        return cast(int, self.parent[v])
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 315-339
```python
    def join(self, a: int, b: int) -> int | None:
        a, b = self.find(a), self.find(b)
        if a == b:
            return a
        if self.size[a] < self.size[b]:
            a, b = b, a
        self.parent[b] = a
        self.size[a] += self.size[b]


def optimize_for_inference(
    model: torch.nn.Module,
    pass_config: dict[str, Any] | None = None,
    tracer: type[fx.Tracer] = fx.Tracer,
) -> torch.nn.Module:
    """
    Performs a set of optimization passes to optimize a model for the
    purposes of inference. Specifically, the passes that are run are:
    1. Conv/BN fusion
    2. Dropout removal
    3. MKL layout optimizations

    The third optimization takes a function `use_mkl_heuristic` that's used
    to determine whether a subgraph should be explicitly run in MKL layout.
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 340-363
```python
    Note: As FX does not currently handle aliasing, this pass currently
    assumes nothing aliases. If that isn't true, use at your own risk.
    """
    default_pass_config = {
        "conv_bn_fuse": True,
        "remove_dropout": True,
        "mkldnn_layout_optimize": {"heuristic": use_mkl_length},
    }
    if pass_config is None:
        pass_config = {}
    default_pass_config.update(pass_config)

    if default_pass_config["conv_bn_fuse"]:
        model = fuse(model)
    if default_pass_config["remove_dropout"]:
        model = remove_dropout(model)
    if default_pass_config["mkldnn_layout_optimize"] is False:
        return model
    if not isinstance(default_pass_config["mkldnn_layout_optimize"], dict):
        raise RuntimeError("mkldnn_layout_optimize config is not a dict")
    if "heuristic" not in default_pass_config["mkldnn_layout_optimize"]:
        raise RuntimeError("Heuristic not found in mkldnn_layout_optimize config")
    use_mkl_heuristic = default_pass_config["mkldnn_layout_optimize"]["heuristic"]
```
- **EN**: This block continues `optimize_for_inference` and works to validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `optimize_for_inference`，用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 364-381
```python
    cur_tracer = tracer()
    fx_graph = cur_tracer.trace(copy.deepcopy(model))
    fx.GraphModule(cur_tracer.root, fx_graph)
    modules: dict[str, nn.Module] = dict(model.named_modules())

    class MklSupport(Enum):
        NO = 1
        YES = 2
        UNKNOWN = 3

    # Inserts to_mkldnn and to_dense around every node we want to be a MKLDNN node.
    # If the op is in `mkldnn_supported` then we always treat it as a MKLDNN node.
    # However, if it's in `mkldnn_supported_unknown`, then we only treat it as
    # a MKLDNN node if its inputs are MKLDNN nodes.
    for node in list(fx_graph.nodes):
        supports_mkldnn = MklSupport.NO
        if node.op == "call_module":
            cur_module = modules[node.target]
```
- **EN**: Defines the `optimize_for_inference` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`optimize_for_inference` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 382-408
```python
            if type(cur_module) in mkldnn_supported:
                supports_mkldnn = MklSupport.YES
                sample_parameter = next(cur_module.parameters(), None)
                if sample_parameter is not None:
                    if sample_parameter.dtype != torch.float:
                        raise AssertionError(
                            "this pass is only for torch.float modules"
                        )
                    if sample_parameter.device != torch.device("cpu"):
                        raise AssertionError("this pass is only for CPU modules")
        elif node.op == "call_function":
            if node.target in mkldnn_supported:
                supports_mkldnn = MklSupport.YES
            elif node.target in mkldnn_supported_unknown:
                supports_mkldnn = MklSupport.UNKNOWN

        if supports_mkldnn != MklSupport.NO:
            if supports_mkldnn == MklSupport.UNKNOWN:
                if not any(arg.target == "to_dense" for arg in node.args):
                    continue
            with fx_graph.inserting_before(node):
                mkldnn_args = fx.map_arg(
                    node.args, lambda n: fx_graph.call_method("to_mkldnn", (n,))
                )

            node.args = cast(tuple[fx.node.Argument], mkldnn_args)
```
- **EN**: This block continues `optimize_for_inference` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `optimize_for_inference`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 409-432
```python
            with fx_graph.inserting_after(node):
                dense_x = fx_graph.create_node("call_method", "to_dense", (node,))
                node.replace_all_uses_with(dense_x)
                dense_x.args = (node,)

    # Does pre-conversion of all modules into MKLDNN (when possible)
    old_modules = modules_to_mkldnn(list(fx_graph.nodes), modules)
    fx_graph.old_modules = old_modules  # type: ignore[attr-defined]

    # optimizes all a -> to_dense -> to_mkldnn -> b patterns into a -> b
    for node in fx_graph.nodes:
        if node.op == "call_method" and node.target == "to_dense":
            prv_node = node.args[0]
            users = list(node.users)
            for user in users:
                if user.op == "call_method" and user.target == "to_mkldnn":
                    user.replace_all_uses_with(prv_node)
                    fx_graph.erase_node(user)
            if len(node.users) == 0:
                fx_graph.erase_node(node)

    num_nodes = len(fx_graph.nodes)
    uf = UnionFind(num_nodes)
```
- **EN**: This block continues `optimize_for_inference` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `optimize_for_inference`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 433-450
```python
    def get_color(n: fx.Node) -> int | None:
        if hasattr(n, "color"):  # Current node is part of a MKL subgraph
            return uf.find(n.color)
        if hasattr(n, "start_color"):  # Current node is input to MKL subgraph
            return uf.find(n.start_color)
        return None

    # This code is to find each MKLDNN subgraph. Each MKLDNN subgraph consists
    # of input nodes (which are only `to_mkldnn` calls), output nodes
    # (`to_dense` calls), and intermediate nodes, which are run entirely on
    # MKLDNN layout tensors.
    #
    # Specifically, this code does a flood fill on a directed acyclic graph
    # (DAG), starting from each possible "start node" (i.e: `to_mkldnn` nodes).
    # If every node only had one input, this would be sufficient. However, in
    # the case that a node has multiple inputs coming from different start
    # nodes (i.e. colors), we need to join these 2 colors into 1. That's done
    # using a Disjoint Set Union.
```
- **EN**: Defines the `optimize_for_inference` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`optimize_for_inference` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 451-475
```python
    for cur_idx, node in enumerate(fx_graph.nodes):
        if node.op == "call_method" and node.target == "to_mkldnn":
            node.start_color = cur_idx
            uf.make_set(cur_idx)
        elif node.op == "call_method" and node.target == "to_dense":
            if get_color(node.args[0]) is None:
                raise AssertionError("Expected color for to_dense input")
            node.end_color = get_color(node.args[0])
        else:
            cur_colors = [
                get_color(i)
                for i in node.all_input_nodes
                if isinstance(i, fx.Node)
                if get_color(i) is not None
            ]

            if len(cur_colors) == 0:
                continue
            if any(i is None for i in cur_colors):
                raise AssertionError("Found None in cur_colors")
            sorted_colors: list[int] = sorted(cur_colors)  # type: ignore[arg-type]
            node.color = sorted_colors[0]
            for other_color in sorted_colors[1:]:
                uf.join(sorted_colors[0], other_color)
```
- **EN**: This block continues `optimize_for_inference` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `optimize_for_inference`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 476-499
```python
    mkldnn_graphs: dict[int, MklSubgraph] = defaultdict(lambda: MklSubgraph(fx_graph))
    for node in fx_graph.nodes:
        if hasattr(node, "color"):
            mkldnn_graphs[uf.find(node.color)].nodes.append(node)
        if hasattr(node, "start_color"):
            mkldnn_graphs[uf.find(node.start_color)].start_nodes.append(node)
        if hasattr(node, "end_color"):
            mkldnn_graphs[uf.find(node.end_color)].end_nodes.append(node)

    # Now that we have all the subgraphs, we need to decide which MKLDNN
    # subgraphs we actually want to keep in MKLDNN.
    for graph in mkldnn_graphs.values():
        if not use_mkl_heuristic(graph):
            for node in graph.start_nodes + graph.end_nodes:
                prv = node.args[0]
                node.replace_all_uses_with(prv)  # type: ignore[arg-type]
                fx_graph.erase_node(node)
            reset_modules(graph.nodes, modules, old_modules)

    mkldnn_conversions = 0
    for node in fx_graph.nodes:
        if node.target == "to_mkldnn" or node.target == "to_dense":
            mkldnn_conversions += 1
```
- **EN**: This block continues `optimize_for_inference` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `optimize_for_inference`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 500-503
```python
    logging.getLogger(__name__).info("mkldnn conversions: %s", mkldnn_conversions)
    fx_graph.lint()
    result = fx.GraphModule(model, fx_graph)
    return result
```
- **EN**: This block continues `optimize_for_inference` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `optimize_for_inference`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.fx`, `torch.nn`, `torch.nn.functional`, `torch.utils.mkldnn`, `torch.fx.node`, `torch.fx.passes.shape_prop`, `torch.nn.utils.fusion`
- **Standard library / 标准库**: `copy`, `logging`, `operator`, `time`, `collections`, `collections.abc`, `enum`, `typing`
- **Primary symbols / 核心符号**: `__all__`, `_parent_name`, `matches_module_pattern`, `replace_node_module`, `fuse`, `remove_dropout`, `extract_subgraph`, `modules_to_mkldnn`, `reset_modules`, `MklSubgraph`, `gen_mkl_autotuner`, `use_mkl_length`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
