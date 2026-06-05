# control_dependencies.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/control_dependencies.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `ControlDeps`. It exposes functions such as `_`, `control_deps_eager`, `control_deps_autograd`, `get_subgraph_name`, `_extract_unique_nodes`, `preserve_node_ordering`, and `...+1`. Module note: Effect ordering pass for inductor.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `ControlDeps` 等类。同时提供 `_`、`control_deps_eager`、`control_deps_autograd`、`get_subgraph_name`、`_extract_unique_nodes`、`preserve_node_ordering`、`另有1项` 等函数。模块文档字符串给出了额外背景说明。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
"""
Effect ordering pass for inductor.

This pass adds ordering dependencies to FX graphs using the control_deps HOP
for precise control over scheduling constraints. When you need exact ordering between
operations (e.g., collective_start -> mm -> wait), this pass wraps operations
with control_deps to make dependencies explicit.
"""

from typing import Any

import torch.fx as fx
import torch.utils._pytree as pytree
from torch._C import DispatchKey
from torch._higher_order_ops.utils import register_fake
from torch._ops import HigherOrderOperator
from torch.utils._ordered_set import OrderedSet


````
- **EN**: Imports dependencies such as `typing`, `torch.fx`, `torch.utils._pytree`, `torch._C`, `torch._higher_order_ops.utils`, `torch._ops`, and `...+1` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `typing`、`torch.fx`、`torch.utils._pytree`、`torch._C`、`torch._higher_order_ops.utils`、`torch._ops`、`另有1项` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含分支、循环或上下文管理等控制流。

### Lines 21-40 / 第 21-40 行
````python
class ControlDeps(HigherOrderOperator):
    """
    Higher-order operator that enforces ordering by making dependencies explicit.

    Schema: control_deps(additional_deps, target, *args, **kwargs) -> result
    where:
    - additional_deps: tuple of tensors that must be computed before this op
    - subgraph: GraphModule containing the exact operation to execute
    - args/kwargs: arguments for the target function

    This ensures all tensors in additional_deps are computed before the target
    executes, creating explicit scheduling dependencies.
    """

    def __init__(self) -> None:
        super().__init__("control_deps")

    def __call__(self, additional_deps, subgraph, *args, **kwargs):
        """Call the operator with dependencies and subgraph.

````
- **EN**: Introduces class `ControlDeps`, function `__init__`, function `__call__`. Initializes or updates values such as `Schema`, and `where`.
- **CN**: 这里定义了类`ControlDeps`、函数`__init__`、函数`__call__`。初始化或更新了 `Schema`、`where` 等值。

### Lines 41-60 / 第 41-60 行
````python
        Args:
            additional_deps: Tuple of tensors that must be computed first
            subgraph: GraphModule containing the exact operation to execute
            *args: Arguments to pass to the subgraph
        """
        if not isinstance(additional_deps, (tuple, list)):
            raise TypeError(
                f"additional_deps must be tuple/list, got {type(additional_deps).__name__}"
            )
        if not (isinstance(subgraph, fx.GraphModule) or callable(subgraph)):
            raise TypeError(
                f"subgraph must be GraphModule or callable, got {type(subgraph).__name__}"
            )
        # pyrefly: ignore [missing-attribute]
        return super().__call__(additional_deps, subgraph, *args, **kwargs)


control_deps = ControlDeps()

# control_deps wraps side-effecting ops (e.g. record_event, wait_event)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Args`, `additional_deps`, `subgraph`, and `control_deps`. This range continues the implementation of function `ControlDeps.__call__`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Args`、`additional_deps`、`subgraph`、`control_deps` 等值。这一段延续了函数`ControlDeps.__call__` 的具体实现。

### Lines 61-80 / 第 61-80 行
````python
# and must not be eliminated by DCE even when its outputs are unused.
from torch.fx.node import has_side_effect


has_side_effect(control_deps)


# Register fake implementation for tracing
@register_fake(control_deps)
def _(additional_deps, subgraph, *args, **kwargs):
    """Fake tensor implementation - execute the subgraph."""
    return subgraph(*args, **kwargs)


# Register eager execution implementation
@control_deps.py_impl(DispatchKey.CompositeExplicitAutograd)
def control_deps_eager(additional_deps, subgraph, *args, **kwargs):
    """Eager implementation - just execute the subgraph."""
    return subgraph(*args, **kwargs)

````
- **EN**: Imports dependencies such as `torch.fx.node` for the logic in this range. Introduces function `_`, function `control_deps_eager`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `torch.fx.node` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_`、函数`control_deps_eager`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 81-100 / 第 81-100 行
````python

# Autograd impl needed because additional_deps tensors may have autograd state,
# causing dispatch through AutogradCUDA even in post-autograd graphs.
@control_deps.py_impl(DispatchKey.Autograd)
def control_deps_autograd(additional_deps, subgraph, *args, **kwargs):
    return subgraph(*args, **kwargs)


def get_subgraph_name(gm: fx.GraphModule, name):
    name = f"subgraph_{name}"

    if not hasattr(gm, name):
        return name

    i = 0
    while hasattr(gm, f"{name}_{i}"):
        i += 1

    return f"{name}_{i}"

````
- **EN**: Introduces function `control_deps_autograd`, function `get_subgraph_name`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`control_deps_autograd`、函数`get_subgraph_name`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 101-120 / 第 101-120 行
````python

def _extract_unique_nodes(
    args: tuple[Any, ...], kwargs: dict[str, Any]
) -> tuple[list[fx.Node], list[Any], Any]:
    """Extract unique fx.Node instances from args/kwargs using pytree.

    Args:
        args: The positional arguments (may contain nested structures with fx.Node)
        kwargs: The keyword arguments (may contain nested structures with fx.Node)

    Returns:
        - Ordered list of unique fx.Node instances (preserves first occurrence order)
        - Flattened list of all items from args/kwargs
        - The pytree spec for reconstructing the original structure
    """
    flat_args_kwargs, spec = pytree.tree_flatten((args, kwargs))
    unique_nodes: list[fx.Node] = []
    seen: OrderedSet[fx.Node] = OrderedSet()
    for item in flat_args_kwargs:
        if isinstance(item, fx.Node) and item not in seen:
````
- **EN**: Introduces function `_extract_unique_nodes`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `args`, `Args`, `kwargs`, `Returns`, `unique_nodes`, and `seen`.
- **CN**: 这里定义了函数`_extract_unique_nodes`。包含分支、循环或上下文管理等控制流。初始化或更新了 `args`、`Args`、`kwargs`、`Returns`、`unique_nodes`、`seen` 等值。

### Lines 121-140 / 第 121-140 行
````python
            unique_nodes.append(item)
            seen.add(item)
    return unique_nodes, flat_args_kwargs, spec


def preserve_node_ordering(
    graph: fx.Graph,
    additional_deps_map: dict[fx.Node, OrderedSet[fx.Node]],
    verbose: bool = False,
) -> None:
    """
    Preserve node ordering using control_deps HOP with subgraph.

    This function wraps operations with control_deps that:
    1. Makes additional dependencies explicit (first argument)
    2. Creates a subgraph internally to preserve the exact original operation
    3. Preserves the original node names

    Args:
        graph: The FX graph to modify
````
- **EN**: Introduces function `preserve_node_ordering`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `graph`, `additional_deps_map`, `verbose`, and `Args`.
- **CN**: 这里定义了函数`preserve_node_ordering`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `graph`、`additional_deps_map`、`verbose`、`Args` 等值。

### Lines 141-160 / 第 141-160 行
````python
        additional_deps_map: Mapping from dependent nodes to their dependencies
        verbose: If True, print debug information
    """
    if not additional_deps_map:
        return

    # Track replacements so we can update dependencies
    replacements: dict[fx.Node, fx.Node] = {}

    # Process each node that needs additional dependencies
    for dependent_node, dep_nodes in additional_deps_map.items():
        assert dependent_node.op == "call_function", dependent_node.op

        original_name = dependent_node.name
        original_args = dependent_node.args
        original_kwargs = dependent_node.kwargs
        original_meta = dependent_node.meta.copy()

        updated_dep_nodes = [replacements.get(dep, dep) for dep in dep_nodes]

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `additional_deps_map`, `verbose`, `replacements`, `original_name`, `original_args`, `original_kwargs`, and `...+2`. This range continues the implementation of function `preserve_node_ordering`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `additional_deps_map`、`verbose`、`replacements`、`original_name`、`original_args`、`original_kwargs`、`另有2项` 等值。这一段延续了函数`preserve_node_ordering` 的具体实现。

### Lines 161-180 / 第 161-180 行
````python
        # Create a subgraph that preserves the exact original operation
        subgraph_module = _create_subgraph_for_node(graph, dependent_node)

        owning_mod = graph.owning_module
        assert owning_mod is not None
        subgraph_attr_name = get_subgraph_name(owning_mod, original_name)
        setattr(graph.owning_module, subgraph_attr_name, subgraph_module)

        # Create control_deps call with:
        # 1. Additional dependencies as first arg (explicit)
        # 2. Subgraph via get_attr (like b2b gemm pass)
        # 3. Original arguments (only fx.Node args and kwargs are passed)
        with graph.inserting_before(dependent_node):
            # Create get_attr node for the subgraph
            get_subgraph = graph.get_attr(subgraph_attr_name)

            # Extract unique nodes from nested args/kwargs
            node_args, _, _ = _extract_unique_nodes(original_args, original_kwargs)

            # Create with temporary name first
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `subgraph_module`, `owning_mod`, `subgraph_attr_name`, and `get_subgraph`. This range continues the implementation of function `preserve_node_ordering`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `subgraph_module`、`owning_mod`、`subgraph_attr_name`、`get_subgraph` 等值。这一段延续了函数`preserve_node_ordering` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
            ordered_node = graph.call_function(
                control_deps,
                args=(
                    tuple(updated_dep_nodes),  # additional_deps
                    get_subgraph,  # subgraph via get_attr (like b2b gemm)
                    *node_args,  # original node arguments (from both args and kwargs)
                ),
                kwargs={},
                name=f"__temp_{original_name}",  # Temporary name to avoid conflict
            )

        # Copy metadata from original node
        ordered_node.meta = original_meta
        # this will be constrained on the target node in subgraph if it exists
        ordered_node.meta.pop("eager_input_vals", None)

        # Replace all uses of the original node with the ordered version
        dependent_node.replace_all_uses_with(ordered_node)

        # Remove the original node from the graph
````
- **EN**: Initializes or updates values such as `ordered_node`, `args`, `kwargs`, and `name`. This range continues the implementation of function `preserve_node_ordering`.
- **CN**: 初始化或更新了 `ordered_node`、`args`、`kwargs`、`name` 等值。这一段延续了函数`preserve_node_ordering` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python
        graph.erase_node(dependent_node)

        # Now rename the ordered node to the original name
        ordered_node.name = original_name  # PRESERVE ORIGINAL NAME

        # Track the replacement for future dependencies
        replacements[dependent_node] = ordered_node


def _create_subgraph_for_node(
    graph: fx.Graph, node: fx.Node, additional_deps=None
) -> fx.GraphModule:
    """
    Create a subgraph that exactly recreates a node's operation optionally passing through additional dependencies.

    The subgraph takes only the fx.Node arguments and recreates the operation
    with the exact target, args structure, and kwargs.

    Args:
        graph: The parent graph
````
- **EN**: Introduces function `_create_subgraph_for_node`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph`, and `Args`.
- **CN**: 这里定义了函数`_create_subgraph_for_node`。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph`、`Args` 等值。

### Lines 221-240 / 第 221-240 行
````python
        node: The node to wrap in a subgraph
        additional_deps: Additional dependencies to pass through the subgraph

    Returns:
        A GraphModule containing the subgraph
    """
    # Get the owning module
    owning_module = graph.owning_module

    # Create a new graph for the subgraph
    subgraph = fx.Graph(owning_module)

    # Extract unique nodes and get flattened structure + spec
    unique_nodes, flat_args_kwargs, spec = _extract_unique_nodes(node.args, node.kwargs)

    # Create placeholders for each unique node
    node_to_placeholder: dict[fx.Node, fx.Node] = {}
    for idx, orig_node in enumerate(unique_nodes):
        placeholder = subgraph.placeholder(f"arg_{idx}")
        if "val" in orig_node.meta:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node`, `additional_deps`, `Returns`, `owning_module`, `subgraph`, `node_to_placeholder`, and `...+1`. This range continues the implementation of function `_create_subgraph_for_node`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `node`、`additional_deps`、`Returns`、`owning_module`、`subgraph`、`node_to_placeholder`、`另有1项` 等值。这一段延续了函数`_create_subgraph_for_node` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python
            placeholder.meta.update(orig_node.meta)
        node_to_placeholder[orig_node] = placeholder

    # Replace fx.Node instances with their placeholders
    def replace_nodes(item: Any) -> Any:
        if isinstance(item, fx.Node):
            return node_to_placeholder[item]
        return item

    additional_deps_placeholders = []
    for idx, dep in enumerate(additional_deps or ()):
        placeholder = subgraph.placeholder(f"dep_{idx}")
        if "val" in dep.meta:
            placeholder.meta.update(dep.meta)
        additional_deps_placeholders.append(placeholder)

    new_flat = [replace_nodes(item) for item in flat_args_kwargs]
    new_args, new_kwargs = pytree.tree_unflatten(new_flat, spec)

    # Recreate the exact original operation in the subgraph
````
- **EN**: Introduces function `replace_nodes`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `additional_deps_placeholders`, `placeholder`, and `new_flat`.
- **CN**: 这里定义了函数`replace_nodes`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `additional_deps_placeholders`、`placeholder`、`new_flat` 等值。

### Lines 261-280 / 第 261-280 行
````python
    assert callable(node.target)
    result = subgraph.call_function(
        node.target,
        tuple(new_args),
        new_kwargs,  # type: ignore[arg-type]
    )

    # Copy metadata from the original node
    result.meta.update(node.meta)

    if additional_deps_placeholders:
        outputs = tuple([result] + additional_deps_placeholders)
        out = subgraph.output(outputs)
        out.meta["val"] = tuple(output.meta.get("val") for output in outputs)
    else:
        out = subgraph.output(result)
        if "val" in result.meta:
            out.meta["val"] = result.meta["val"]

    return fx.GraphModule(owning_module, subgraph)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `result`, `outputs`, `out`, and `else`. This range continues the implementation of function `_create_subgraph_for_node`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `result`、`outputs`、`out`、`else` 等值。这一段延续了函数`_create_subgraph_for_node` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Primary classes: `ControlDeps`  
  **CN**: 主要类：`ControlDeps`
- **EN**: Primary functions: `_`, `control_deps_eager`, `control_deps_autograd`, `get_subgraph_name`, `_extract_unique_nodes`, `preserve_node_ordering`, and `...+1`  
  **CN**: 主要函数：`_`、`control_deps_eager`、`control_deps_autograd`、`get_subgraph_name`、`_extract_unique_nodes`、`preserve_node_ordering`、`另有1项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch.fx`, `torch.utils._pytree`, `torch._C`, `torch._higher_order_ops.utils`, `torch._ops`, `torch.utils._ordered_set`, `torch.fx.node`
