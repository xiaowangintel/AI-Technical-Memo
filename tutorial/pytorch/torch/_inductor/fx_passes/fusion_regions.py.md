# fusion_regions.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/fusion_regions.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `FusionRegion`. It exposes functions such as `is_view_node`, `is_fusible_node`, `_get_contiguous_fusible_spans`, `_find_connected_components`, `build_fusion_regions`, `collapse_fusion_regions`, and `...+1`. Module note: Detect fusion regions for overlap scheduling.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `FusionRegion` 等类。同时提供 `is_view_node`、`is_fusible_node`、`_get_contiguous_fusible_spans`、`_find_connected_components`、`build_fusion_regions`、`collapse_fusion_regions`、`另有1项` 等函数。模块文档字符串给出了额外背景说明。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
"""Detect fusion regions for overlap scheduling."""

import operator
from dataclasses import dataclass, field

import torch
import torch.fx as fx
from torch._logging import trace_structured
from torch.utils._ordered_set import OrderedSet
from torch.utils._runtime_estimation import get_num_bytes


@dataclass
class FusionRegion:
    """Represents a connected set of fusible operations that will fuse together."""

    subgraph_node: fx.Node  # The call_module node for this fusion
    subgraph_module: fx.GraphModule  # The subgraph module
    total_bytes: int = field(default=0, init=False)  # Total input + output bytes
    cost_ms: float = field(default=0.0, init=False)  # Estimated cost in milliseconds
````
- **EN**: Imports dependencies such as `operator`, `dataclasses`, `torch`, `torch.fx`, `torch._logging`, `torch.utils._ordered_set`, and `...+1` for the logic in this range. Introduces class `FusionRegion`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields.
- **CN**: 这里导入了 `operator`、`dataclasses`、`torch`、`torch.fx`、`torch._logging`、`torch.utils._ordered_set`、`另有1项` 等依赖，为后续逻辑提供基础能力。这里定义了类`FusionRegion`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。

### Lines 21-40 / 第 21-40 行
````python

    def __post_init__(self) -> None:
        """Compute cost based on subgraph's placeholder inputs and output node."""
        self.total_bytes, self.cost_ms = self._compute_cost()

    def _compute_cost(self) -> tuple[int, float]:
        from torch.utils._pytree import tree_flatten
        from torch.utils._runtime_estimation import get_transfer_time

        subgraph = self.subgraph_module
        input_vals = [
            n.meta.get("val") for n in subgraph.graph.find_nodes(op="placeholder")
        ]
        output_vals = [
            n.meta.get("val")
            for n in torch._inductor.utils.output_node(subgraph).all_input_nodes
        ]
        flat_inputs, _ = tree_flatten(input_vals)
        flat_outputs, _ = tree_flatten(output_vals)

````
- **EN**: Imports dependencies such as `torch.utils._pytree`, and `torch.utils._runtime_estimation` for the logic in this range. Introduces function `__post_init__`, function `_compute_cost`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `subgraph`, `input_vals`, and `output_vals`.
- **CN**: 这里导入了 `torch.utils._pytree`、`torch.utils._runtime_estimation` 等依赖，为后续逻辑提供基础能力。这里定义了函数`__post_init__`、函数`_compute_cost`。包含分支、循环或上下文管理等控制流。初始化或更新了 `subgraph`、`input_vals`、`output_vals` 等值。

### Lines 41-60 / 第 41-60 行
````python
        transfer_time_ns = get_transfer_time(flat_inputs, flat_outputs)
        total_bytes = sum(
            get_num_bytes(t)
            for t in flat_inputs + flat_outputs
            if isinstance(t, torch.Tensor)
        )
        return total_bytes, transfer_time_ns / 1e6


def is_view_node(n: fx.Node) -> bool:
    """Check if a node is a view operation (zero cost, no memory allocation)."""

    return isinstance(n.target, torch._ops.OpOverload) and (
        n.target.is_view and n.target.namespace in ("aten", "prims")
    )


def is_fusible_node(n: fx.Node) -> bool:
    """Check if a node is fusible based on whether it has an inductor lowering.

````
- **EN**: Introduces function `is_view_node`, function `is_fusible_node`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `transfer_time_ns`, and `total_bytes`.
- **CN**: 这里定义了函数`is_view_node`、函数`is_fusible_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `transfer_time_ns`、`total_bytes` 等值。

### Lines 61-80 / 第 61-80 行
````python
    A node is fusible if:
    - It has a lowering in torch._inductor.lowering.lowerings
    - It does NOT have a flop counter (expensive compute ops like mm/conv)
    - It is NOT a registered fallback (ops that fall back to eager)
    - It is NOT a collective or wait op
    - For aten.cat, it must have <= max_pointwise_cat_inputs inputs
    """
    if n.op != "call_function":
        return False

    target = n.target
    if not isinstance(target, torch._ops.OpOverload):
        return False

    # Exclude collectives and waits (they have their own scheduling)
    if target.namespace == "_c10d_functional":
        return False

    from torch._inductor.lowering import fallbacks, lowerings
    from torch.utils.flop_counter import flop_registry
````
- **EN**: Imports dependencies such as `torch._inductor.lowering`, and `torch.utils.flop_counter` for the logic in this range. Processes profiling, FLOP, or benchmark information to characterize execution cost. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch._inductor.lowering`、`torch.utils.flop_counter` 等依赖，为后续逻辑提供基础能力。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 81-100 / 第 81-100 行
````python

    # Must have a lowering
    if target not in lowerings:
        return False

    # Exclude fallbacks (ops that fall back to eager execution)
    if target in fallbacks:
        return False

    # Exclude ops with flop counters (expensive compute ops like mm, conv, etc.)
    overload_packet = target.overloadpacket
    if overload_packet in flop_registry:
        return False

    # Special case: cat is only fusible if it has few enough inputs
    if target == torch.ops.aten.cat.default:
        inputs = n.args[0] if n.args else []
        if isinstance(inputs, (list, tuple)):
            import torch._inductor.config as inductor_config

````
- **EN**: Imports dependencies such as `torch._inductor.config` for the logic in this range. Processes profiling, FLOP, or benchmark information to characterize execution cost. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch._inductor.config` 等依赖，为后续逻辑提供基础能力。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 101-120 / 第 101-120 行
````python
            if len(inputs) > inductor_config.max_pointwise_cat_inputs:
                return False

    return True


def _get_contiguous_fusible_spans(gm: fx.GraphModule) -> list[list[fx.Node]]:
    """Get contiguous spans of fusible nodes from the graph.

    Walks the graph in topological order and groups consecutive fusible
    nodes into spans. Non-fusible nodes act as span boundaries.
    """
    spans: list[list[fx.Node]] = []
    current_span: list[fx.Node] = []

    for node in gm.graph.nodes:
        if is_fusible_node(node):
            current_span.append(node)
        else:
            # Non-fusible node ends the current span
````
- **EN**: Introduces function `_get_contiguous_fusible_spans`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `spans`, `current_span`, and `else`.
- **CN**: 这里定义了函数`_get_contiguous_fusible_spans`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `spans`、`current_span`、`else` 等值。

### Lines 121-140 / 第 121-140 行
````python
            if current_span:
                spans.append(current_span)
                current_span = []

    if current_span:
        spans.append(current_span)

    return spans


def _find_connected_components(span: list[fx.Node]) -> list[list[fx.Node]]:
    """Find connected components within a span of fusible nodes.

    Two nodes are connected if one is an input to the other (direct data dependency).
    """
    if not span:
        return []

    from torch.fx.experimental.optimization import UnionFind

````
- **EN**: Imports dependencies such as `torch.fx.experimental.optimization` for the logic in this range. Introduces function `_find_connected_components`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch.fx.experimental.optimization` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_find_connected_components`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
    span_set = OrderedSet(span)
    node_to_idx = {n: i for i, n in enumerate(span)}

    uf = UnionFind(len(span))
    for i in range(len(span)):
        uf.make_set(i)

    # Union nodes based on input edges
    for node in span:
        node_idx = node_to_idx[node]
        for inp in node.all_input_nodes:
            if inp in span_set:
                uf.join(node_idx, node_to_idx[inp])

    # Group by root
    root_to_nodes: dict[int, list[fx.Node]] = {}
    for node in span:
        root = uf.find(node_to_idx[node])
        if root not in root_to_nodes:
            root_to_nodes[root] = []
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `span_set`, `node_to_idx`, `uf`, `node_idx`, `root_to_nodes`, and `root`. This range continues the implementation of function `_find_connected_components`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `span_set`、`node_to_idx`、`uf`、`node_idx`、`root_to_nodes`、`root` 等值。这一段延续了函数`_find_connected_components` 的具体实现。

### Lines 161-180 / 第 161-180 行
````python
        root_to_nodes[root].append(node)

    return list(root_to_nodes.values())


def build_fusion_regions(
    gm: fx.GraphModule,
) -> dict[fx.Node, OrderedSet[fx.Node]]:
    """Build fusion regions from contiguous spans of fusible nodes.

    1. Identify contiguous spans of fusible nodes (separated by non-fusible nodes)
    2. Find connected components within each span
    3. Return regions that have 2+ non-view nodes

    This ensures fusion regions are strictly local - no reordering across
    non-fusible node boundaries.

    Returns a dict mapping each node to its fusion group (OrderedSet of nodes).
    """
    # Build node -> topo index map for sorting
````
- **EN**: Introduces function `build_fusion_regions`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `gm`.
- **CN**: 这里定义了函数`build_fusion_regions`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `gm` 等值。

### Lines 181-200 / 第 181-200 行
````python
    node_to_idx: dict[fx.Node, int] = {n: i for i, n in enumerate(gm.graph.nodes)}

    # Step 1: Get contiguous spans of fusible nodes
    spans = _get_contiguous_fusible_spans(gm)

    # Step 2: Find connected components within each span
    region_of: dict[fx.Node, OrderedSet[fx.Node]] = {}

    for span in spans:
        if len(span) < 2:
            continue

        components = _find_connected_components(span)

        for component in components:
            # Skip regions with fewer than 2 non-view nodes (views have no cost)
            non_view_count = sum(1 for n in component if not is_view_node(n))
            if non_view_count < 2:
                continue

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node_to_idx`, `spans`, `region_of`, `components`, and `non_view_count`. This range continues the implementation of function `build_fusion_regions`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `node_to_idx`、`spans`、`region_of`、`components`、`non_view_count` 等值。这一段延续了函数`build_fusion_regions` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python
            # Sort nodes in topological order to preserve original ordering
            sorted_component = sorted(component, key=lambda n: node_to_idx[n])
            node_set = OrderedSet(sorted_component)

            for node in sorted_component:
                region_of[node] = node_set

    return region_of


def collapse_fusion_regions(
    gm: fx.GraphModule,
    region_of: dict[fx.Node, OrderedSet[fx.Node]],
) -> dict[fx.Node, FusionRegion]:
    """
    Collapse fusion regions into call_module nodes using fuse_by_partitions.
    Returns new_region_of mapping module nodes to FusionRegions.
    """
    from torch.fx.passes.utils.fuser_utils import fuse_by_partitions

````
- **EN**: Imports dependencies such as `torch.fx.passes.utils.fuser_utils` for the logic in this range. Introduces function `collapse_fusion_regions`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch.fx.passes.utils.fuser_utils` 等依赖，为后续逻辑提供基础能力。这里定义了函数`collapse_fusion_regions`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 221-240 / 第 221-240 行
````python
    if not region_of:
        return {}

    # Get unique node sets (regions with <2 nodes already filtered in build_fusion_regions)
    unique_regions: list[tuple[OrderedSet[fx.Node], int]] = []
    seen_region_ids: OrderedSet[int] = OrderedSet()
    for node_set in region_of.values():
        region_id = id(node_set)
        if region_id not in seen_region_ids:
            seen_region_ids.add(region_id)
            unique_regions.append((node_set, region_id))

    if not unique_regions:
        return {}

    # Log graph before fusion
    trace_structured(
        "artifact",
        metadata_fn=lambda: {
            "name": "fusion_regions_before",
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `unique_regions`, `seen_region_ids`, `region_id`, and `metadata_fn`. This range continues the implementation of function `collapse_fusion_regions`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `unique_regions`、`seen_region_ids`、`region_id`、`metadata_fn` 等值。这一段延续了函数`collapse_fusion_regions` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python
            "encoding": "string",
        },
        payload_fn=lambda: gm.print_readable(print_output=False),
    )

    # Build partitions list for fuse_by_partitions
    partitions = [dict.fromkeys(nodes) for nodes, _ in unique_regions]

    # Fuse all partitions at once
    fuse_by_partitions(gm, partitions, prefix="_fusion_region_")

    # Log graph after fusion
    trace_structured(
        "artifact",
        metadata_fn=lambda: {
            "name": "fusion_regions_after",
            "encoding": "string",
        },
        payload_fn=lambda: gm.print_readable(print_output=False),
    )
````
- **EN**: Initializes or updates values such as `payload_fn`, `partitions`, and `metadata_fn`. This range continues the implementation of function `collapse_fusion_regions`.
- **CN**: 初始化或更新了 `payload_fn`、`partitions`、`metadata_fn` 等值。这一段延续了函数`collapse_fusion_regions` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python

    # Build new_region_of by finding the call_module nodes
    new_region_of: dict[fx.Node, FusionRegion] = {}

    for region_idx in range(len(unique_regions)):
        subgraph_name = f"_fusion_region_{region_idx}"

        # Find the call_module node
        module_nodes = list(gm.graph.find_nodes(op="call_module", target=subgraph_name))
        assert len(module_nodes) == 1, (
            f"Expected 1 call_module for {subgraph_name}, got {len(module_nodes)}"
        )
        module_node = module_nodes[0]

        subgraph_module = getattr(gm, subgraph_name)

        # Create FusionRegion with all required info
        region = FusionRegion(
            subgraph_node=module_node,
            subgraph_module=subgraph_module,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_region_of`, `subgraph_name`, `module_nodes`, `module_node`, `subgraph_module`, `region`, and `...+1`. This range continues the implementation of function `collapse_fusion_regions`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_region_of`、`subgraph_name`、`module_nodes`、`module_node`、`subgraph_module`、`region`、`另有1项` 等值。这一段延续了函数`collapse_fusion_regions` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
        )

        new_region_of[module_node] = region

    return new_region_of


def expand_fusion_regions(
    gm: fx.GraphModule,
    region_of: dict[fx.Node, FusionRegion],
) -> dict[fx.Node, fx.Node | None]:
    """
    Expand call_module nodes back to their original nodes using _inline_module.

    Returns a mapping from erased module nodes to their replacement (last inlined node).
    This is used with transfer_erased_node_deps to update dependencies.
    """
    from torch.fx.experimental.const_fold import _inline_module

    result: dict[fx.Node, fx.Node | None] = {}
````
- **EN**: Imports dependencies such as `torch.fx.experimental.const_fold` for the logic in this range. Introduces function `expand_fusion_regions`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `gm`, `region_of`, and `result`.
- **CN**: 这里导入了 `torch.fx.experimental.const_fold` 等依赖，为后续逻辑提供基础能力。这里定义了函数`expand_fusion_regions`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `gm`、`region_of`、`result` 等值。

### Lines 301-320 / 第 301-320 行
````python

    if not region_of:
        return result

    for module_node, region in list(region_of.items()):
        if module_node.op != "call_module":
            continue

        subgraph_name = module_node.target
        assert isinstance(subgraph_name, str)
        assert hasattr(gm, subgraph_name), (
            f"Expected submodule {subgraph_name} to exist"
        )

        # Users of module_node are get_items that will be removed from the graph
        for user in module_node.users:
            if user.op == "call_function" and user.target == operator.getitem:
                result[user] = None

        # Get the output arg from the subgraph to determine what will replace module_node
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `subgraph_name`. This range continues the implementation of function `expand_fusion_regions`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `subgraph_name` 等值。这一段延续了函数`expand_fusion_regions` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python
        output_arg = torch._inductor.utils.output_node(region.subgraph_module).args[0]

        # Inline the module and get the mapping from subgraph nodes to new nodes.
        # Skip DCE since the graph may not be in a topo ordered state
        subgraph_to_new = _inline_module(gm, subgraph_name, run_dce=False)

        # Map module_node to the replacement for the output arg
        # For multi-output (tuple), use the last element (latest in topo order)
        # so dependencies are only satisfied after all outputs are computed
        if isinstance(output_arg, (list, tuple)):
            if output_arg:
                last_arg = output_arg[-1]
                assert isinstance(last_arg, fx.Node)
                result[module_node] = subgraph_to_new[last_arg]
        elif isinstance(output_arg, fx.Node) and output_arg in subgraph_to_new:
            result[module_node] = subgraph_to_new[output_arg]

        delattr(gm, subgraph_name)

    return result
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `output_arg`, `subgraph_to_new`, and `last_arg`. This range continues the implementation of function `expand_fusion_regions`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `output_arg`、`subgraph_to_new`、`last_arg` 等值。这一段延续了函数`expand_fusion_regions` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Primary classes: `FusionRegion`  
  **CN**: 主要类：`FusionRegion`
- **EN**: Primary functions: `is_view_node`, `is_fusible_node`, `_get_contiguous_fusible_spans`, `_find_connected_components`, `build_fusion_regions`, `collapse_fusion_regions`, and `...+1`  
  **CN**: 主要函数：`is_view_node`、`is_fusible_node`、`_get_contiguous_fusible_spans`、`_find_connected_components`、`build_fusion_regions`、`collapse_fusion_regions`、`另有1项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `operator`, `dataclasses`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `torch._logging`, `torch.utils._ordered_set`, `torch.utils._runtime_estimation`, `torch._inductor.lowering`, `torch.utils.flop_counter`, `torch.fx.experimental.optimization`, `torch.fx.passes.utils.fuser_utils`, `torch.fx.experimental.const_fold`, `torch.utils._pytree`, `torch._inductor.config`
