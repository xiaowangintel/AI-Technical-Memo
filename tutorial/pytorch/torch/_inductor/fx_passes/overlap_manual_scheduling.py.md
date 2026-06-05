# overlap_manual_scheduling.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/overlap_manual_scheduling.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `ManualOverlapPreservingBucketer`, and `ManualOverlapScheduler`. It exposes functions such as `manual_overlap_bucketing`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `ManualOverlapPreservingBucketer`、`ManualOverlapScheduler` 等类。同时提供 `manual_overlap_bucketing` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
from __future__ import annotations

import heapq
from collections import Counter, defaultdict
from typing import Any, TYPE_CHECKING

import torch  # noqa: TC001
import torch.fx as fx  # noqa: TC001
from torch._dynamo.graph_deduplication import _stable_topological_sort
from torch._inductor.fx_passes.bucketing import (
    _get_collective_node_from_wait,
    _schedulable_wait_node,
    BucketMode,
    is_all_gather_into_tensor as is_all_gather,
    is_fsdp_all_gather,
    is_fsdp_reduce_scatter,
    is_reduce_scatter_tensor as is_reduce_scatter,
    merge_all_gather_bucket,
    merge_reduce_scatter_bucket,
)
````
- **EN**: Imports dependencies such as `__future__`, `heapq`, `collections`, `typing`, `torch  # noqa: TC001`, `torch.fx`, and `...+2` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis.
- **CN**: 这里导入了 `__future__`、`heapq`、`collections`、`typing`、`torch  # noqa: TC001`、`torch.fx`、`另有2项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。

### Lines 21-40 / 第 21-40 行
````python
from torch._inductor.fx_passes.overlap_preserving_bucketer import (
    get_full_bucket_key,
    OverlapPreservingBucketer,
)
from torch._inductor.fx_passes.overlap_scheduling import (
    CollectiveInfo,
    is_compute_node,
    OverlapScheduler,
)
from torch.utils._ordered_set import OrderedSet

from .graph_view import get_subgraph_by_path, GraphView, make_graph_view


if TYPE_CHECKING:
    from collections.abc import Callable

import logging


````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.overlap_preserving_bucketer`, `torch._inductor.fx_passes.overlap_scheduling`, `torch.utils._ordered_set`, `.graph_view`, `collections.abc`, and `logging` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.fx_passes.overlap_preserving_bucketer`、`torch._inductor.fx_passes.overlap_scheduling`、`torch.utils._ordered_set`、`.graph_view`、`collections.abc`、`logging` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。包含分支、循环或上下文管理等控制流。

### Lines 41-60 / 第 41-60 行
````python
logger = logging.getLogger(__name__)


class ManualOverlapPreservingBucketer(OverlapPreservingBucketer):
    """
    Buckets collective operations based on user specifications.
    The actual bucket happens in bucket_collectives, where all-gathers/reduce-scatters in
        `nodes` will be buckted one single all-gather/reduce-scatter.
    """

    def __init__(
        self,
        *args: Any,
        **kwargs: Any,
    ):
        super().__init__(*args, **kwargs)
        self.node_to_wait_map: dict[fx.Node, fx.Node] = defaultdict()
        # Maps bucketed nodes to their type string, scoped to this bucketer
        # instance so metadata doesn't leak across separate invocations.
        self.bucketed_node_types: dict[fx.Node, str] = {}
````
- **EN**: Introduces class `ManualOverlapPreservingBucketer`, function `__init__`. Initializes or updates values such as `logger`.
- **CN**: 这里定义了类`ManualOverlapPreservingBucketer`、函数`__init__`。初始化或更新了 `logger` 等值。

### Lines 61-80 / 第 61-80 行
````python

    def _bucket_group(self, coll_nodes: list[fx.Node]) -> None:
        assert len(coll_nodes) > 0, "bucketed coll_nodes should have nonzero node"

        waits = [self.collective_info[n].wait_node for n in coll_nodes]
        # Use earliest wait insertion point
        first_wait = min(waits, key=lambda w: self.node_idx[w])
        # Find insertion location
        first = coll_nodes[0]
        next_node = first
        while next_node in coll_nodes:
            next_node = next_node.next

        if is_all_gather(first):
            new_nodes, replacements = merge_all_gather_bucket(
                self.graph,
                coll_nodes,
                wait_insertion_point=first_wait,
                insert_before=next_node,
                mode=self.bucket_mode,
````
- **EN**: Introduces function `_bucket_group`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `waits`, `first_wait`, `first`, `next_node`, `wait_insertion_point`, `insert_before`, and `...+1`.
- **CN**: 这里定义了函数`_bucket_group`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `waits`、`first_wait`、`first`、`next_node`、`wait_insertion_point`、`insert_before`、`另有1项` 等值。

### Lines 81-100 / 第 81-100 行
````python
            )
        elif is_reduce_scatter(first):
            new_nodes, replacements = merge_reduce_scatter_bucket(
                self.graph,
                coll_nodes,
                wait_insertion_point=first_wait,
                insert_before=next_node,
                mode=self.bucket_mode,
            )
        else:
            raise ValueError(
                "bucket non all_gather/reduce_scatter node is not supported"
            )

        logger.debug(f"bucketing nodes: {coll_nodes} into {new_nodes}")  # noqa: G004

        # Identify the new wait(s) and their collective start in a single pass
        wait_to_start = {
            n: start
            for n in new_nodes
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `wait_insertion_point`, `insert_before`, `mode`, `else`, `wait_to_start`, and `n`. This range continues the implementation of function `ManualOverlapPreservingBucketer._bucket_group`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `wait_insertion_point`、`insert_before`、`mode`、`else`、`wait_to_start`、`n` 等值。这一段延续了函数`ManualOverlapPreservingBucketer._bucket_group` 的具体实现。

### Lines 101-120 / 第 101-120 行
````python
            if (start := _get_collective_node_from_wait(n)) is not None
        }
        assert len(wait_to_start) >= 1, (
            f"Expected at least one new wait, got none in {new_nodes}"
        )
        new_waits = list(wait_to_start)
        new_start: fx.Node = wait_to_start[new_waits[0]]
        # Use last wait as the canonical wait for scheduling (same node when len == 1)
        new_wait = new_waits[-1]

        # Track bucketed node types on this bucketer instance so it doesn't leak
        # when the same graph is processed by multiple ManualOverlapScheduler
        # invocations (e.g. separate forward and backward passes).
        node_type = (
            "bucketed_all_gather" if is_all_gather(first) else "bucketed_reduce_scatter"
        )
        wait_set = OrderedSet(new_waits)
        for n in new_nodes:
            if n in wait_set:
                self.bucketed_node_types[n] = node_type + "_wait"
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_waits`, `new_start`, `new_wait`, `node_type`, and `wait_set`. This range continues the implementation of function `ManualOverlapPreservingBucketer._bucket_group`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_waits`、`new_start`、`new_wait`、`node_type`、`wait_set` 等值。这一段延续了函数`ManualOverlapPreservingBucketer._bucket_group` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
                self.node_to_wait_map[n] = new_wait
            elif n is new_start:
                self.bucketed_node_types[n] = node_type

    def manual_bucket_collectives(self, nodes: list[fx.Node]) -> None:
        """
        Bucket all all-gather/reduce-scatter nodes from nodes into one all-gather/reduce-scatter.
        """
        # Filter out valid collectives
        collectives = [n for n in nodes if n in self.collective_info]
        if collectives == []:
            return
        grouped_collectives: dict[object, OrderedSet[fx.Node]] = defaultdict(OrderedSet)
        for node in collectives:
            if not (
                is_fsdp_all_gather(node, self.node_ancestors)
                or is_fsdp_reduce_scatter(node)
            ):
                continue
            key = get_full_bucket_key(node, "custom_ops")
````
- **EN**: Introduces function `manual_bucket_collectives`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `collectives`, `grouped_collectives`, and `key`.
- **CN**: 这里定义了函数`manual_bucket_collectives`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `collectives`、`grouped_collectives`、`key` 等值。

### Lines 141-160 / 第 141-160 行
````python
            if key is not None:
                grouped_collectives[key].add(node)

        for key, nodes in grouped_collectives.items():  # type: ignore[arg-type]
            self._bucket_group(list(nodes))


class ManualOverlapScheduler(OverlapScheduler):
    """
    Scheduler that manual buckets and reorders collective nodes based on module_bucket_plans
    """

    def __init__(
        self,
        gm: fx.GraphModule,
        module_bucket_plans: list[list[str] | str],
        insert_overlap_deps: bool,
        module_stack_fn: Callable[[fx.Node], list[tuple[str, type[Any]]]] | None = None,
        bucket_mode: BucketMode | None = None,
    ):
````
- **EN**: Introduces class `ManualOverlapScheduler`, function `__init__`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `gm`, `module_bucket_plans`, `insert_overlap_deps`, `module_stack_fn`, and `bucket_mode`.
- **CN**: 这里定义了类`ManualOverlapScheduler`、函数`__init__`。包含分支、循环或上下文管理等控制流。初始化或更新了 `gm`、`module_bucket_plans`、`insert_overlap_deps`、`module_stack_fn`、`bucket_mode` 等值。

### Lines 161-180 / 第 161-180 行
````python
        # Manual overlap historically used "custom_ops" mode for bucketing
        bucket_mode = bucket_mode or "custom_ops"
        super().__init__(
            gm,
            max_in_flight_gb=0.0,
            max_compute_pre_fetch=0,
            collective_bucketing=True,
            insert_overlap_deps=insert_overlap_deps,
            compute_overlap_multipler=0.0,
            max_coll_distance=0,
            custom_runtime_estimation=None,
            collective_estimator="analytical",
            max_memory_increase_gb=None,
            max_memory_increase_ratio=None,
            bucket_mode=bucket_mode,
        )
        self.module_bucket_plans = module_bucket_plans
        self.nodes_in_subgraph: list[list[fx.Node]] = []

        self.bucketer = ManualOverlapPreservingBucketer(
````
- **EN**: Initializes or updates values such as `bucket_mode`, `max_in_flight_gb`, `max_compute_pre_fetch`, `collective_bucketing`, `insert_overlap_deps`, `compute_overlap_multipler`, and `...+5`. This range continues the implementation of function `ManualOverlapScheduler.__init__`.
- **CN**: 初始化或更新了 `bucket_mode`、`max_in_flight_gb`、`max_compute_pre_fetch`、`collective_bucketing`、`insert_overlap_deps`、`compute_overlap_multipler`、`另有5项` 等值。这一段延续了函数`ManualOverlapScheduler.__init__` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
            graph=self.graph,
            collective_info=self.collective_info,
            scheduled=OrderedSet(self.graph.nodes),
            bucket_mode=bucket_mode,
        )
        self.insert_overlap_deps = insert_overlap_deps

        self.module_stack_fn = module_stack_fn

    def _identify_collectives(self) -> None:
        """Identify all collective operations."""
        for node in self.nodes:
            if _schedulable_wait_node(node):
                start = node.args[0]
                info = CollectiveInfo(
                    start_node=start,
                    wait_node=node,
                    size_bytes=0,
                    estimated_time_ms=0,
                    exposed_time_ms=0,
````
- **EN**: Introduces function `_identify_collectives`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph`, `collective_info`, `scheduled`, `bucket_mode`, `start`, `info`, and `...+5`.
- **CN**: 这里定义了函数`_identify_collectives`。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph`、`collective_info`、`scheduled`、`bucket_mode`、`start`、`info`、`另有5项` 等值。

### Lines 201-220 / 第 201-220 行
````python
                )
                self.collective_info[start] = info
                self.wait_to_start[node] = start
                self.unscheduled_collectives.add(start)

    def _add_to_ready_queue(self, node: fx.Node) -> None:
        """Manual scheduling uses single queue ordered by original node index."""
        heapq.heappush(self.on_path_ready, (self.node_idx[node], node))

    def run(self) -> torch.fx.GraphModule:
        """Entry point to run the manual bucket algorithm"""
        # Bucket collectives in each bucket_module
        self._manual_bucket_collectives()

        # Reorder collectives with last/next bucket_module
        self._manual_reorder_graph()

        return self.gm

    def _manual_reorder_graph(self) -> None:
````
- **EN**: Introduces function `_add_to_ready_queue`, function `run`, function `_manual_reorder_graph`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_add_to_ready_queue`、函数`run`、函数`_manual_reorder_graph`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 221-240 / 第 221-240 行
````python
        """
        Reorder nodes in the FX graph to enforce manual overlap dependencies.

        forward graph (all-gathers only):
            modules are processed in order: module 0, 1, 2, ...

            before reordering:
            ag_start_0 -> ag_wait_0 -> compute_0 -> ag_start_1 -> ag_wait_1 -> compute_1 -> ...

            Reordering prefetches module i+1's parameters while computing module i
            It adds dependencies: ag_wait_i should depend on ag_start_(i+1)
            This enforces ag_start_(i+1) to happen before ag_wait_i so it overlaps with module i's compute

            after reordering:
            ag_start_0 -> ag_start_1 -> ag_wait_0 -> compute_0 -> ag_wait_1 -> compute_1 -> ...

        backward graph (all-gathers and reduce-scatters):
            modules are processed in reverse order: module N, N-1, N-2, ...

            before reordering:
````
- **EN**: This range continues the implementation of function `ManualOverlapScheduler._manual_reorder_graph`.
- **CN**: 这一段延续了函数`ManualOverlapScheduler._manual_reorder_graph` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python
            ag_start_N -> ag_wait_N -> compute_N -> rs_start_N -> rs_wait_N -> ...

            For all-gathers, prefetch module i-1's parameters while computing module i
            Adds dependencies: ag_wait_i should depend on ag_start_(i-1)
            So ag_start_(i-1) overlaps with module i's compute

            For reduce-scatters, defer rs_wait_i to happen after rs_start_(i-1)
            Adds dependencies: rs_wait_i should depend on rs_start_(i-1)
            So rs_start_i overlaps with module i-1's compute

        """
        delayed_rs_wait_nodes: list[fx.Node] = []
        current_rs_start_nodes: list[fx.Node] = []
        overlap_deps: dict[fx.Node, OrderedSet[fx.Node]] = defaultdict(OrderedSet)

        # Re-initialize after graph modification in _manual_bucket_collectives
        self.node_idx = {n: i for i, n in enumerate(self.nodes)}
        self.on_path_ready = []
        self.scheduled = OrderedSet()
        for node in self.nodes:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `delayed_rs_wait_nodes`, `current_rs_start_nodes`, and `overlap_deps`. This range continues the implementation of function `ManualOverlapScheduler._manual_reorder_graph`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `delayed_rs_wait_nodes`、`current_rs_start_nodes`、`overlap_deps` 等值。这一段延续了函数`ManualOverlapScheduler._manual_reorder_graph` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python
            if self.in_degree[node] == 0:
                self._add_to_ready_queue(node)

        # schedule reduce scatter normally in self._schedule
        while self.on_path_ready:
            _, node = heapq.heappop(self.on_path_ready)
            node_type = self.bucketer.bucketed_node_types.get(node, "")

            if node in self.scheduled:
                continue

            if node_type == "bucketed_reduce_scatter":
                # Collect reduce scatter start nodes (pre_bucket_rs and rs)
                current_rs_start_nodes.append(node)

            elif node_type == "bucketed_reduce_scatter_wait":
                # When we see a wait node from a new RS, flush delayed waits
                # with dependencies on previously collected RS start nodes
                if current_rs_start_nodes:
                    for delayed in delayed_rs_wait_nodes:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node_type`. This range continues the implementation of function `ManualOverlapScheduler._manual_reorder_graph`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `node_type` 等值。这一段延续了函数`ManualOverlapScheduler._manual_reorder_graph` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
                        for rs_start in current_rs_start_nodes:
                            overlap_deps[delayed].add(rs_start)
                    delayed_rs_wait_nodes.clear()
                    current_rs_start_nodes.clear()
                delayed_rs_wait_nodes.append(node)

            self._schedule(node)

        self.scheduled = OrderedSet(reversed(list(self.scheduled)))
        picked_ag: list[fx.Node] = []
        last_compute: fx.Node | None = None

        for node in self.scheduled:
            node_type = self.bucketer.bucketed_node_types.get(node, "")
            if node_type == "bucketed_all_gather":
                picked_ag.append(node)
                continue

            if node_type == "bucketed_all_gather_wait":
                # Connect corresponding all_gather_wait -> all_gather edges
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `picked_ag`, `last_compute`, and `node_type`. This range continues the implementation of function `ManualOverlapScheduler._manual_reorder_graph`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `picked_ag`、`last_compute`、`node_type` 等值。这一段延续了函数`ManualOverlapScheduler._manual_reorder_graph` 的具体实现。

### Lines 301-320 / 第 301-320 行
````python
                if picked_ag:
                    for ag in picked_ag:
                        overlap_deps[self.bucketer.node_to_wait_map[node]].add(ag)
                picked_ag.clear()
            if is_compute_node(node):
                last_compute = node

        if last_compute is not None and not bool(
            OrderedSet(picked_ag) & OrderedSet(self.node_ancestors[last_compute])
        ):
            for ag in picked_ag:
                overlap_deps[last_compute].add(ag)

        _stable_topological_sort(self.graph, overlap_deps)
        self.graph.lint()

        if self.insert_overlap_deps:
            from torch._inductor.fx_passes.control_dependencies import (
                preserve_node_ordering,
            )
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.control_dependencies` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `last_compute`. This range continues the implementation of function `ManualOverlapScheduler._manual_reorder_graph`.
- **CN**: 这里导入了 `torch._inductor.fx_passes.control_dependencies` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `last_compute` 等值。这一段延续了函数`ManualOverlapScheduler._manual_reorder_graph` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python

            preserve_node_ordering(self.graph, overlap_deps)

    def _manual_bucket_collectives(self) -> None:
        """Bucket nodes in each module_bucket from module_bucket_plans."""
        self._obtain_nodes_in_subgraph()
        for i, nodes in enumerate(self.nodes_in_subgraph):
            self.bucketer.manual_bucket_collectives(nodes=nodes)

        _stable_topological_sort(self.graph, {})
        self.graph.lint()
        self.nodes = list(self.graph.nodes)
        self.in_degree = Counter(user for node in self.nodes for user in node.users)

    def _schedule(self, node: fx.Node) -> None:
        """Schedule a node."""
        assert node not in self.scheduled
        assert all(n in self.scheduled for n in node.all_input_nodes)
        self.scheduled.add(node)
        for user in node.users:
````
- **EN**: Introduces function `_manual_bucket_collectives`, function `_schedule`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_manual_bucket_collectives`、函数`_schedule`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 341-360 / 第 341-360 行
````python
            self.in_degree[user] -= 1
            if self.in_degree[user] == 0:
                self._add_to_ready_queue(user)

    def _obtain_nodes_in_subgraph(self) -> None:
        """
        Obtain nodes in each subgraph from module_bucket_plans
        """
        graph_view: GraphView | None = make_graph_view(self.graph, self.module_stack_fn)
        if graph_view is None:
            return

        for module in self.module_bucket_plans:
            subgraph_view = get_subgraph_by_path(graph_view, module)
            self.nodes_in_subgraph.append(subgraph_view)

        all_subgraph_nodes = [
            node for sublist in self.nodes_in_subgraph for node in sublist
        ]
        unique_subgraph_nodes = list(OrderedSet(all_subgraph_nodes))
````
- **EN**: Introduces function `_obtain_nodes_in_subgraph`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph_view`, `subgraph_view`, `all_subgraph_nodes`, and `unique_subgraph_nodes`.
- **CN**: 这里定义了函数`_obtain_nodes_in_subgraph`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph_view`、`subgraph_view`、`all_subgraph_nodes`、`unique_subgraph_nodes` 等值。

### Lines 361-380 / 第 361-380 行
````python
        assert len(all_subgraph_nodes) <= len(unique_subgraph_nodes), (
            f"Overlapping FX nodes detected across subgraphs in `module_bucket_plans`. "
            f"Expected disjoint node sets but found "
            f"{len(all_subgraph_nodes) - len(unique_subgraph_nodes)} duplicated node(s)."
        )


def manual_overlap_bucketing(
    gm: torch.fx.GraphModule,
    module_bucket_plans: list[list[str] | str],
    insert_overlap_deps: bool = False,
    module_stack_fn: Callable[[fx.Node], list[tuple[str, type[Any]]]] | None = None,
    bucket_mode: BucketMode | None = None,
) -> torch.fx.GraphModule:
    """Schedule nodes based on user specifications in module_bucket_plans
    The manual overlapping consists of two steps:
    Step 1: bucket all-gather/reduce-scatter in each module in module_bucket_plans
    Step 2: reorder all-gather to overlap with last module_bucket &
        reorder reduce-scatter to overlap with next module_bucket
    TODO(ruisizhang123): allow users to explicitly specify which
````
- **EN**: Introduces function `manual_overlap_bucketing`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `gm`, `module_bucket_plans`, `insert_overlap_deps`, `module_stack_fn`, and `bucket_mode`.
- **CN**: 这里定义了函数`manual_overlap_bucketing`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `gm`、`module_bucket_plans`、`insert_overlap_deps`、`module_stack_fn`、`bucket_mode` 等值。

### Lines 381-400 / 第 381-400 行
````python
        module_bucket they want to overlap.

    Args:
        gm: input graph module to optimize.
        module_bucket_plans: user specified FQNs
        module_stack_fn: Optional callable for extracting module hierarchy from nodes.
            Used to construct a GraphView for identifying nodes in module_bucket_plans.
            The module_class component of the returned tuples is not used by this pass.

            See the `module_stack_fn` parameter in `make_graph_view` (graph_view.py) for
            detailed documentation on signature, return format, and usage examples.
        bucket_mode: Bucket mode for collective bucketing. None uses default.
    """
    # decode abbreviated FQNs to actual FQNs
    overlapped_gm = ManualOverlapScheduler(
        gm,
        module_bucket_plans,
        insert_overlap_deps,
        module_stack_fn,
        bucket_mode=bucket_mode,
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `Args`, `gm`, `module_bucket_plans`, `module_stack_fn`, `bucket_mode`, and `overlapped_gm`. This range continues the implementation of function `manual_overlap_bucketing`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `Args`、`gm`、`module_bucket_plans`、`module_stack_fn`、`bucket_mode`、`overlapped_gm` 等值。这一段延续了函数`manual_overlap_bucketing` 的具体实现。

### Lines 401-403 / 第 401-403 行
````python
    ).run()
    overlapped_gm.recompile()
    return overlapped_gm
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `manual_overlap_bucketing`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`manual_overlap_bucketing` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Primary classes: `ManualOverlapPreservingBucketer`, and `ManualOverlapScheduler`  
  **CN**: 主要类：`ManualOverlapPreservingBucketer`、`ManualOverlapScheduler`
- **EN**: Primary functions: `manual_overlap_bucketing`  
  **CN**: 主要函数：`manual_overlap_bucketing`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `heapq`, `collections`, `typing`, `logging`, `collections.abc`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `torch._dynamo.graph_deduplication`, `torch._inductor.fx_passes.bucketing`, `torch._inductor.fx_passes.overlap_preserving_bucketer`, `torch._inductor.fx_passes.overlap_scheduling`, `torch.utils._ordered_set`, `.graph_view`, `torch._inductor.fx_passes.control_dependencies`
