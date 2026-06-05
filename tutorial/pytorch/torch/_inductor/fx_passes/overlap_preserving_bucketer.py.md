# overlap_preserving_bucketer.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/overlap_preserving_bucketer.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `WhyNoBucket`, `PGEvent`, and `OverlapPreservingBucketer`. It exposes functions such as `is_collective_or_wait`, and `finalize_overlap_scheduling`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `WhyNoBucket`、`PGEvent`、`OverlapPreservingBucketer` 等类。同时提供 `is_collective_or_wait`、`finalize_overlap_scheduling` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
import itertools
import logging
from collections import defaultdict
from dataclasses import dataclass
from typing import Any, Literal, Optional

import torch
import torch.fx as fx
from torch._dynamo.utils import counters
from torch._inductor.augmented_graph_helper import AugmentedGraphHelper
from torch._inductor.fx_passes.bucketing import (
    _default_bucket_mode,
    _get_collective_node_from_wait,
    _schedulable_wait_node,
    BucketMode,
    get_full_bucket_key,
    has_mergeable_all_gather_convert_dtype,
    is_all_gather_into_tensor as is_all_gather,
    is_reduce_scatter_tensor as is_reduce_scatter,
)
from torch._inductor.fx_passes.fsdp import is_fsdp_all_gather
from torch._inductor.fx_passes.overlap_scheduling import (
    CollBucket,
    CollectiveInfo,
    get_group_name,
    is_compute_node,
    log as overlap_scheduling_log,
)
````
- **EN**: Imports dependencies such as `itertools`, `logging`, `collections`, `dataclasses`, `typing`, `torch`, and `...+6` for the logic in this range.
- **CN**: 这里导入了 `itertools`、`logging`、`collections`、`dataclasses`、`typing`、`torch`、`另有6项` 等依赖，为后续逻辑提供基础能力。

### Lines 29-56 / 第 29-56 行
````python
from torch._logging import trace_structured
from torch.utils._ordered_set import OrderedSet


bucket_log = logging.getLogger(__name__)


@dataclass
class WhyNoBucket:
    name1: str
    name2: str
    reason: str
    args: tuple[Any, ...]

    def __init__(self, node1: fx.Node, node2: fx.Node) -> None:
        self.name1 = node1.name
        self.name2 = node2.name
        self.reason = ""
        self.args = ()

    def __call__(self, reason: str, *args: Any) -> None:
        if bucket_log.isEnabledFor(logging.DEBUG):
            bucket_log.debug(
                "cannot bucket %s with %s: " + reason,
                self.name1,
                self.name2,
                *args,
            )
````
- **EN**: Imports dependencies such as `torch._logging`, and `torch.utils._ordered_set` for the logic in this range. Introduces class `WhyNoBucket`, function `__init__`, function `__call__`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields.
- **CN**: 这里导入了 `torch._logging`、`torch.utils._ordered_set` 等依赖，为后续逻辑提供基础能力。这里定义了类`WhyNoBucket`、函数`__init__`、函数`__call__`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。

### Lines 57-84 / 第 57-84 行
````python


def is_collective_or_wait(n: fx.Node) -> bool:
    """Check if node is a collective start or wait."""
    if _schedulable_wait_node(n):
        return True
    # Collective starts have exactly one use: the wait_tensor
    if len(n.users) == 1:
        user = next(iter(n.users.keys()))
        if _schedulable_wait_node(user):
            return True
    return False


@dataclass
class PGEvent:
    """
    Represents an important event in a process group timeline. Either
    a collective start, wait, or hiding compute. Each node is linked
    to its prev and next and these dependencies are reflected
    in the augmented graph.

    We want to enforce a sequential ordering of collective starts and waits
    because NCCL collectives on the same process group execute on the same CUDA
    stream, creating implicit dependencies between all operations on that PG.

    A wait of a particular collective will implicitly force realization of all collectives
    enqueued prior to that collective.
````
- **EN**: Introduces function `is_collective_or_wait`, class `PGEvent`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`is_collective_or_wait`、类`PGEvent`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。处理与设备相关的行为、能力信息或派发决策。

### Lines 85-112 / 第 85-112 行
````python
    """

    node: fx.Node
    event_type: Literal["compute", "starts", "waits"]
    position: int
    prev: Optional["PGEvent"] = None
    next: Optional["PGEvent"] = None

    @property
    def is_start(self) -> bool:
        return self.event_type == "starts"

    @property
    def is_wait(self) -> bool:
        return self.event_type == "waits"

    @property
    def is_compute(self) -> bool:
        return self.event_type == "compute"

    def unlink(self) -> tuple[Optional["PGEvent"], Optional["PGEvent"]]:
        """Remove this event from the linked list, return (prev, next)."""
        prev_event, next_event = self.prev, self.next
        if self.prev:
            self.prev.next = self.next
        if self.next:
            self.next.prev = self.prev
        self.prev = None
````
- **EN**: Introduces function `is_start`, function `is_wait`, function `is_compute`, function `unlink`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`is_start`、函数`is_wait`、函数`is_compute`、函数`unlink`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 113-140 / 第 113-140 行
````python
        self.next = None
        return prev_event, next_event

    def insert_between(
        self, prev_event: Optional["PGEvent"], next_event: Optional["PGEvent"]
    ) -> None:
        """Insert this event between prev_event and next_event in the linked list."""
        if prev_event:
            prev_event.next = self
        self.prev = prev_event

        if next_event:
            next_event.prev = self
        self.next = next_event


class OverlapPreservingBucketer:
    """
    Buckets collective operations while preserving compute-collective overlap relationships.
    Uses an augmented graph to track dependencies between compute and collective operations.
    """

    def __init__(
        self,
        graph: fx.Graph,
        collective_info: dict[fx.Node, CollectiveInfo],
        scheduled: OrderedSet[fx.Node],
        max_bucket_memory_gb: float = 1.0,
````
- **EN**: Introduces function `insert_between`, class `OverlapPreservingBucketer`, function `__init__`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `graph`, `collective_info`, `scheduled`, and `max_bucket_memory_gb`.
- **CN**: 这里定义了函数`insert_between`、类`OverlapPreservingBucketer`、函数`__init__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `graph`、`collective_info`、`scheduled`、`max_bucket_memory_gb` 等值。

### Lines 141-168 / 第 141-168 行
````python
        max_coll_distance: int = 1000,
        insert_overlap_deps: bool = False,
        collective_bucketing: bool = True,
        bucket_mode: BucketMode | None = None,
        bucket_exposed_first: bool | None = None,
        region_of: dict[fx.Node, Any] | None = None,
        bucket_only_internode_comms: bool = False,
    ):
        self.graph = graph
        self.collective_info = collective_info
        self.scheduled = scheduled
        self.max_bucket_memory_gb = max_bucket_memory_gb
        self.node_idx = {n: i for i, n in enumerate(scheduled)}
        self.max_coll_distance = max_coll_distance
        self.insert_overlap_deps = insert_overlap_deps
        self.bucket_exposed_first = bucket_exposed_first
        self.bucket_only_internode_comms = bucket_only_internode_comms
        self.bucket_mode = bucket_mode or _default_bucket_mode()
        self.collective_bucketing = collective_bucketing
        self.region_of: dict[fx.Node, Any] = region_of or {}
        self.node_to_event: dict[fx.Node, PGEvent] = {}
        self.all_hiding_nodes: OrderedSet[fx.Node] = OrderedSet()

        # Compute ancestors including original graph edges and hiding interval dependencies
        self.node_ancestors = self._compute_node_ancestors()
        self.aug_graph = AugmentedGraphHelper(self.graph, self.node_ancestors)

        # Build timelines and add constraints to aug_graph
````
- **EN**: Initializes or updates values such as `max_coll_distance`, `insert_overlap_deps`, `collective_bucketing`, `bucket_mode`, `bucket_exposed_first`, `region_of`, and `...+1`. This range continues the implementation of function `OverlapPreservingBucketer.__init__`.
- **CN**: 初始化或更新了 `max_coll_distance`、`insert_overlap_deps`、`collective_bucketing`、`bucket_mode`、`bucket_exposed_first`、`region_of`、`另有1项` 等值。这一段延续了函数`OverlapPreservingBucketer.__init__` 的具体实现。

### Lines 169-196 / 第 169-196 行
````python
        self.pg_to_timeline_head: dict[str, PGEvent | None] = self.build_timelines()
        self._add_hiding_interval_constraints()

    def _compute_node_ancestors(self) -> dict[fx.Node, OrderedSet[fx.Node]]:
        """
        Compute ancestor sets for all nodes including:
        1. Original graph edges
        2. Hiding interval deps: collective_start -> hiding_node -> wait
        """
        augmented_inputs: dict[fx.Node, OrderedSet[fx.Node]] = defaultdict(OrderedSet)
        for start, info in self.collective_info.items():
            if info.is_exposed:
                continue
            for hiding_node in info.hiding_nodes:
                augmented_inputs[hiding_node].add(start)
                augmented_inputs[info.wait_node].add(hiding_node)

        node_ancestors: dict[fx.Node, OrderedSet[fx.Node]] = defaultdict(OrderedSet)
        for node in self.scheduled:
            for input_node in itertools.chain(
                augmented_inputs[node], node.all_input_nodes
            ):
                node_ancestors[node].add(input_node)
                node_ancestors[node] |= node_ancestors[input_node]

        return node_ancestors

    def build_timelines(self) -> dict[str, PGEvent | None]:
````
- **EN**: Introduces function `_compute_node_ancestors`, function `build_timelines`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `augmented_inputs`, and `node_ancestors`.
- **CN**: 这里定义了函数`_compute_node_ancestors`、函数`build_timelines`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `augmented_inputs`、`node_ancestors` 等值。

### Lines 197-224 / 第 197-224 行
````python
        "Construct each process groups ordered series of event"
        all_pgs: OrderedSet[str] = OrderedSet()
        for start in self.collective_info:
            pg = get_group_name(start)
            all_pgs.add(pg)

        pg_timeline: dict[str, PGEvent | None] = {}
        for pg in all_pgs:
            pg_timeline[pg] = self.build_timeline(pg)

        return pg_timeline

    def build_timeline(self, pg: str) -> PGEvent | None:
        """
        Build a timeline of important events (starts, waits, hiding compute) for this process group
        and constrain this ordering in the augmented graph.

        Sequential dependencies are added between all events because NCCL collectives on the same
        process group execute on the same CUDA stream, enforcing LIFO semantics where later-issued
        collectives must complete before earlier ones can finish.
        """

        head = None
        prev_event = None
        position = 0
        hiding_nodes = OrderedSet()

        for node in self.scheduled:
````
- **EN**: Introduces function `build_timeline`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`build_timeline`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 225-252 / 第 225-252 行
````python
            node_type = None

            # Determine if this node is relevant for this PG
            if node in self.collective_info and get_group_name(node) == pg:
                node_type = "starts"
                hiding_nodes |= self.collective_info[node].hiding_nodes
            elif _schedulable_wait_node(node):
                wait_coll = _get_collective_node_from_wait(node)
                if isinstance(wait_coll, fx.Node) and get_group_name(wait_coll) == pg:
                    node_type = "waits"
                # Wait for a different PG but hiding a collective on this PG
                elif node in hiding_nodes:
                    node_type = "compute"
            elif is_compute_node(node) or node in hiding_nodes:
                node_type = "compute"

            if node_type is None:
                continue

            event = PGEvent(node=node, event_type=node_type, position=position)  # type: ignore[arg-type]

            event.insert_between(prev_event, None)

            # Add sequential dependency to augmented graph
            if prev_event:
                self.aug_graph.add_extra_dep(n=event.node, dep=prev_event.node)
            else:
                head = event
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node_type`, `wait_coll`, `event`, `else`, and `head`. This range continues the implementation of function `OverlapPreservingBucketer.build_timeline`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `node_type`、`wait_coll`、`event`、`else`、`head` 等值。这一段延续了函数`OverlapPreservingBucketer.build_timeline` 的具体实现。

### Lines 253-280 / 第 253-280 行
````python

            prev_event = event
            position += 1

        return head

    def _populate_node_to_event(self, pg: str) -> None:
        """Populate node_to_event mapping for a specific PG's timeline."""
        self.node_to_event.clear()
        head = self.pg_to_timeline_head[pg]
        curr = head
        while curr is not None:
            self.node_to_event[curr.node] = curr
            curr = curr.next

    def _add_hiding_interval_constraints(self) -> None:
        """
        Add hiding interval constraints: start -> compute -> wait.
        """
        for start, info in self.collective_info.items():
            if info.is_exposed:
                continue
            for hn in info.hiding_nodes:
                # Enforce: start -> compute -> wait
                self.aug_graph.add_extra_dep(n=hn, dep=start)
                self.aug_graph.add_extra_dep(n=info.wait_node, dep=hn)

            self.all_hiding_nodes |= info.hiding_nodes
````
- **EN**: Introduces function `_populate_node_to_event`, function `_add_hiding_interval_constraints`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `prev_event`, `head`, and `curr`.
- **CN**: 这里定义了函数`_populate_node_to_event`、函数`_add_hiding_interval_constraints`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `prev_event`、`head`、`curr` 等值。

### Lines 281-308 / 第 281-308 行
````python

    def identify_internode_group_names(self) -> OrderedSet[str]:
        # Identify internode comm groups.
        # Temporary uses FSDP pattern as heuristic for outer groups.
        checked_pgs = OrderedSet()
        internode_pgs = OrderedSet()
        for start in self.collective_info:
            pg = get_group_name(start)
            if is_all_gather(start) and pg not in checked_pgs:
                if is_fsdp_all_gather(start):
                    internode_pgs.add(pg)
                checked_pgs.add(pg)
        return internode_pgs

    def _bucket_collectives_impl(self) -> list[CollBucket]:
        """Find and apply bucket transformations for collectives."""
        pg_collectives: dict[str, OrderedSet[fx.Node]] = defaultdict(OrderedSet)
        internode_pgs = self.identify_internode_group_names()

        for start in self.collective_info:
            pg = get_group_name(start)
            pg_collectives[pg].add(start)

        all_buckets: list[CollBucket] = []
        for pg, collectives in pg_collectives.items():
            if self.bucket_only_internode_comms and pg not in internode_pgs:
                continue

````
- **EN**: Introduces function `identify_internode_group_names`, function `_bucket_collectives_impl`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `checked_pgs`, `internode_pgs`, `pg`, `pg_collectives`, and `all_buckets`.
- **CN**: 这里定义了函数`identify_internode_group_names`、函数`_bucket_collectives_impl`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `checked_pgs`、`internode_pgs`、`pg`、`pg_collectives`、`all_buckets` 等值。

### Lines 309-336 / 第 309-336 行
````python
            # Populate node_to_event for this PG's timeline
            self._populate_node_to_event(pg)

            grouped_collectives: dict[object, OrderedSet[fx.Node]] = defaultdict(
                OrderedSet
            )
            for start in collectives:
                key = get_full_bucket_key(start, self.bucket_mode)
                if key[1] is not None:
                    grouped_collectives[key].add(start)

            for key, collective_group in grouped_collectives.items():
                bucket_log.debug(
                    "bucketing collective group with key %s: %s",
                    key,
                    [n.name for n in collective_group],
                )
                buckets = self._find_buckets(collective_group, internode_pgs)
                all_buckets.extend(buckets)

        for coll_bucket in all_buckets:
            if len(coll_bucket.collectives) <= 1:
                continue

            counters["inductor"]["collective_buckets"] += 1
            self._apply_bucket(coll_bucket)
        return all_buckets

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `grouped_collectives`, `key`, and `buckets`. This range continues the implementation of function `OverlapPreservingBucketer._bucket_collectives_impl`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `grouped_collectives`、`key`、`buckets` 等值。这一段延续了函数`OverlapPreservingBucketer._bucket_collectives_impl` 的具体实现。

### Lines 337-364 / 第 337-364 行
````python
    def _apply_deps_and_effect_tokens(self) -> None:
        """Apply topological sort and effect tokens to preserve overlap."""
        from torch._dynamo.graph_deduplication import _stable_topological_sort

        # Clean up any remaining erased node references and cycles
        self.aug_graph.remove_erased_extra_deps()
        autofix = torch._inductor.config.aten_distributed_optimizations.overlap_scheduling_autofix_cycles
        self.aug_graph.check_and_maybe_autofix_cyclic_extra_deps(autofix=autofix)
        additional_deps = self.aug_graph.get_all_extra_deps()

        for n, deps in additional_deps.items():
            torch._check(
                not n._erased, lambda: f"Erased node deps not transferred: {n}"
            )
            for d in deps:
                torch._check(
                    not d._erased, lambda: f"Erased node deps not transferred: {d}"
                )

        _stable_topological_sort(self.graph, additional_deps)

        if self.insert_overlap_deps:
            # Filter out collective-to-collective deps (handled by NCCL stream ordering)
            filtered_deps: dict[fx.Node, OrderedSet[fx.Node]] = {}
            for node, deps in additional_deps.items():
                filtered_node_deps: OrderedSet[fx.Node] = OrderedSet()
                for dep in deps:
                    if not (is_collective_or_wait(node) and is_collective_or_wait(dep)):
````
- **EN**: Imports dependencies such as `torch._dynamo.graph_deduplication` for the logic in this range. Introduces function `_apply_deps_and_effect_tokens`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `autofix`, `additional_deps`, `filtered_deps`, and `filtered_node_deps`.
- **CN**: 这里导入了 `torch._dynamo.graph_deduplication` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_apply_deps_and_effect_tokens`。包含分支、循环或上下文管理等控制流。初始化或更新了 `autofix`、`additional_deps`、`filtered_deps`、`filtered_node_deps` 等值。

### Lines 365-392 / 第 365-392 行
````python
                        filtered_node_deps.add(dep)
                if filtered_node_deps:
                    filtered_deps[node] = filtered_node_deps

            if filtered_deps:
                from torch._inductor.fx_passes.control_dependencies import (
                    preserve_node_ordering,
                )

                preserve_node_ordering(self.graph, filtered_deps)

    def bucket_collectives(self) -> None:
        """Run the full bucketing and dep application flow.

        Order is important:
        1. Bucketing - merge collectives into buckets
        2. Inline fusions - expand call_module back to original nodes
        3. Transfer deps - move deps from erased nodes to their replacements
        4. Add control deps - apply effect tokens and topo sort

        Steps 2-3 MUST happen before step 4, because control deps need to
        reference the final inlined nodes, not the erased fusion modules.
        """
        # Step 1: Bucket collectives
        all_buckets: list[CollBucket] | None = None
        if self.collective_bucketing:
            all_buckets = self._bucket_collectives_impl()

````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.control_dependencies` for the logic in this range. Introduces function `bucket_collectives`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `all_buckets`.
- **CN**: 这里导入了 `torch._inductor.fx_passes.control_dependencies` 等依赖，为后续逻辑提供基础能力。这里定义了函数`bucket_collectives`。包含分支、循环或上下文管理等控制流。初始化或更新了 `all_buckets` 等值。

### Lines 393-420 / 第 393-420 行
````python
        # Step 2: Inline fusion regions (expand call_module -> original nodes)
        replaced: dict[fx.Node, fx.Node | None] = {}
        if self.region_of:
            from torch._inductor.fx_passes.fusion_regions import expand_fusion_regions

            gm = self.graph.owning_module
            replaced = expand_fusion_regions(gm, self.region_of)

        # Step 3: Transfer deps from erased fusion modules to inlined nodes
        if replaced:
            self.aug_graph.transfer_erased_node_deps(replaced)

        # Step 4: Add control deps (MUST be after inline + transfer)
        self._apply_deps_and_effect_tokens()
        self.graph.lint()

        if (
            overlap_scheduling_log.isEnabledFor(logging.DEBUG)
            and all_buckets is not None
        ):
            log_strs: list[str] = []
            stats_num_buckets_per_key = defaultdict(int)
            stats_num_bucketed_collectives_per_key = defaultdict(int)
            stats_num_total_collectives_per_key = defaultdict(int)

            def _bucket_key(node):
                return get_full_bucket_key(node, self.bucket_mode)

````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.fusion_regions` for the logic in this range. Introduces function `_bucket_key`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.fx_passes.fusion_regions` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_bucket_key`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 421-448 / 第 421-448 行
````python
            for start, info in self.collective_info.items():
                stats_num_total_collectives_per_key[_bucket_key(start)] += 1

            for i, bucket in enumerate(all_buckets):
                bucket_n = len(bucket.collectives)
                if bucket_n == 0:
                    continue
                node = bucket.collectives[0]
                key = _bucket_key(node)
                stats_num_buckets_per_key[key] += 1
                stats_num_bucketed_collectives_per_key[key] += bucket_n
                log_strs.append(f"bucket[{i}] key:{key} len:{bucket_n}:{bucket}")
                for coll in bucket.collectives:
                    info = self.collective_info[coll]
                    hns = info.hiding_nodes
                    log_strs.append(f"coll:{coll} hiding_nodes:{hns}")

            bucket_log_strs: list[str] = []
            for key, num_buckets in stats_num_buckets_per_key.items():
                num_colls = stats_num_bucketed_collectives_per_key[key]
                bucket_log_strs.append(
                    f"bucket key stats {key}: {num_colls} in {num_buckets}"
                    f" buckets of total:{stats_num_total_collectives_per_key[key]}"
                )
            bucket_log_strs.append("")
            # Add stats to the beginning
            log_strs[:0] = bucket_log_strs
            bucket_logs = "\n".join(log_strs)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bucket_n`, `node`, `key`, `info`, `hns`, `bucket_log_strs`, and `...+2`. This range continues the implementation of function `OverlapPreservingBucketer.bucket_collectives`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `bucket_n`、`node`、`key`、`info`、`hns`、`bucket_log_strs`、`另有2项` 等值。这一段延续了函数`OverlapPreservingBucketer.bucket_collectives` 的具体实现。

### Lines 449-476 / 第 449-476 行
````python
            trace_structured(
                "artifact",
                metadata_fn=lambda: {
                    "name": "inductor_fx_passes_overlap_bucketing",
                    "encoding": "string",
                },
                payload_fn=lambda: bucket_logs,
            )

    def _compute_overlap_ratio(self, node: fx.Node) -> float:
        """
        Compute what fraction of the collective's time is hidden by compute.
        Returns 0.0 for fully exposed, 1.0 for fully hidden.
        """
        info = self.collective_info[node]
        if info.estimated_time_ms <= 0:
            return 0.0
        hidden_time = info.estimated_time_ms - info.exposed_time_ms
        return hidden_time / info.estimated_time_ms

    def _should_bucket_exposed_first(
        self,
        collective_group: OrderedSet[fx.Node],
        current_pg: str,
        internode_pgs: OrderedSet[str],
    ) -> bool:
        """Determine whether to bucket exposed collectives first."""
        if self.bucket_exposed_first is None:
````
- **EN**: Introduces function `_compute_overlap_ratio`, function `_should_bucket_exposed_first`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `metadata_fn`, `payload_fn`, `info`, `hidden_time`, `collective_group`, `current_pg`, and `...+1`.
- **CN**: 这里定义了函数`_compute_overlap_ratio`、函数`_should_bucket_exposed_first`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `metadata_fn`、`payload_fn`、`info`、`hidden_time`、`collective_group`、`current_pg`、`另有1项` 等值。

### Lines 477-504 / 第 477-504 行
````python
            return current_pg in internode_pgs
        return self.bucket_exposed_first

    def _find_buckets(
        self,
        collective_group: OrderedSet[fx.Node],
        internode_pgs: OrderedSet[str],
    ) -> list[CollBucket]:
        """Find valid buckets within a group of similar collectives."""
        max_bucket_bytes = int(self.max_bucket_memory_gb * 1024 * 1024 * 1024)
        buckets = []
        processed: OrderedSet[fx.Node] = OrderedSet()
        if len(collective_group) == 0:
            return []

        current_pg = get_group_name(next(iter(collective_group)))

        bucket_exposed_first = self._should_bucket_exposed_first(
            collective_group, current_pg, internode_pgs
        )
        if bucket_exposed_first:
            # Sort by overlap ratio (ascending) to bucket least hidden collectives first.
            # Exposed collectives benefit most from bucketing since their latency is on the
            # critical path. Prioritizing them also preserves hiding relationships for
            # already-hidden collectives, which have less to gain from bucketing.
            sorted_collectives = sorted(
                collective_group,
                key=lambda n: (self._compute_overlap_ratio(n), self.node_idx[n]),
````
- **EN**: Introduces function `_find_buckets`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `collective_group`, `internode_pgs`, `max_bucket_bytes`, `buckets`, `processed`, `current_pg`, and `...+3`.
- **CN**: 这里定义了函数`_find_buckets`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `collective_group`、`internode_pgs`、`max_bucket_bytes`、`buckets`、`processed`、`current_pg`、`另有3项` 等值。

### Lines 505-532 / 第 505-532 行
````python
            )
        else:
            sorted_collectives = sorted(
                collective_group,
                key=lambda n: self.node_idx[n],
            )

        for i, start_node in enumerate(sorted_collectives):
            if start_node in processed:
                continue

            if (
                start_node in self.all_hiding_nodes
                or self.collective_info[start_node].wait_node in self.all_hiding_nodes
            ):
                continue

            # Initialize bucket with first collective
            bucket_info = CollBucket(
                collectives=[start_node],
                total_bytes=self.collective_info[start_node].size_bytes,
            )
            processed.add(start_node)

            # Greedy optimization: stop after consecutive failures
            consecutive_failures = 0
            max_consecutive_failures = 20
            start_node_idx = self.node_idx[start_node]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `sorted_collectives`, `key`, `bucket_info`, `collectives`, `total_bytes`, and `...+3`. This range continues the implementation of function `OverlapPreservingBucketer._find_buckets`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`sorted_collectives`、`key`、`bucket_info`、`collectives`、`total_bytes`、`另有3项` 等值。这一段延续了函数`OverlapPreservingBucketer._find_buckets` 的具体实现。

### Lines 533-560 / 第 533-560 行
````python

            # Check candidates in sorted order, break when beyond max distance
            for candidate in sorted_collectives[i + 1 : i + 1 + self.max_coll_distance]:
                candidate_bytes = self.collective_info[candidate].size_bytes
                # proxy on memory use, if we see a too large bucket,
                # dont look for another, later bucket
                if bucket_info.total_bytes + candidate_bytes > max_bucket_bytes:
                    break

                if candidate in processed:
                    continue

                candidate_node_idx = self.node_idx[candidate]
                if (
                    bucket_exposed_first
                    and abs(candidate_node_idx - start_node_idx)
                    > max_consecutive_failures
                ):
                    # Since collectives are sorted by overlap ratio rather than graph
                    # position, skip candidates too far apart in the graph to avoid
                    # creating buckets that block future bucketing opportunities.
                    continue

                if self._can_add_to_bucket(bucket_info, candidate):
                    bucket_info.collectives.append(candidate)
                    bucket_info.total_bytes += candidate_bytes
                    processed.add(candidate)
                    consecutive_failures = 0  # Reset on success
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `candidate_bytes`, `candidate_node_idx`, and `consecutive_failures`. This range continues the implementation of function `OverlapPreservingBucketer._find_buckets`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `candidate_bytes`、`candidate_node_idx`、`consecutive_failures` 等值。这一段延续了函数`OverlapPreservingBucketer._find_buckets` 的具体实现。

### Lines 561-588 / 第 561-588 行
````python
                else:
                    consecutive_failures += 1
                    if consecutive_failures >= max_consecutive_failures:
                        break

            if len(bucket_info.collectives) > 1:
                buckets.append(bucket_info)

        return buckets

    def _ancestor_dep(self, n1: fx.Node, n2: fx.Node) -> bool:
        """Check if there's an ancestor relationship between two nodes."""
        return n1 in self.node_ancestors[n2] or n2 in self.node_ancestors[n1]

    def _get_intervals(
        self, event: PGEvent
    ) -> tuple[tuple[int, int] | None, list[tuple[int, int]]]:
        """Get (execution_interval, hiding_intervals) for a collective event.

        Returns:
            (execution_interval, hiding_intervals) where:
            - execution_interval is (start_pos, wait_pos) or None
            - hiding_intervals is a list of (start_pos, compute_pos) tuples, one for each hiding node

        Works for both start and wait events by looking up the collective info.
        """
        # For start events, directly use the node
        if event.is_start:
````
- **EN**: Introduces function `_ancestor_dep`, function `_get_intervals`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, and `Returns`.
- **CN**: 这里定义了函数`_ancestor_dep`、函数`_get_intervals`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`Returns` 等值。

### Lines 589-616 / 第 589-616 行
````python
            coll = event.node
        # For wait events, look up the start node from the event's args
        elif event.is_wait:
            coll = _get_collective_node_from_wait(event.node)
            if coll is None:
                return None, []
        else:
            return None, []

        if coll not in self.collective_info:
            return None, []

        info = self.collective_info[coll]
        start_event = self.node_to_event[coll]
        wait_event = self.node_to_event[info.wait_node]

        execution_interval = (start_event.position, wait_event.position)

        hiding_intervals = []
        if info.hiding_nodes:
            for hiding_node in info.hiding_nodes:
                hiding_intervals.append(
                    (
                        start_event.position,
                        self.node_to_event[hiding_node].position,
                    )
                )

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `coll`, `else`, `info`, `start_event`, `wait_event`, `execution_interval`, and `...+1`. This range continues the implementation of function `OverlapPreservingBucketer._get_intervals`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `coll`、`else`、`info`、`start_event`、`wait_event`、`execution_interval`、`另有1项` 等值。这一段延续了函数`OverlapPreservingBucketer._get_intervals` 的具体实现。

### Lines 617-644 / 第 617-644 行
````python
        return execution_interval, hiding_intervals

    def _preserves_hiding_intervals(
        self,
        bucket_info: CollBucket,
        candidate: fx.Node,
        start_pos: fx.Node,
        wait_pos: fx.Node,
        why: WhyNoBucket,
    ) -> bool:
        """
        Check that (start_pos, wait_pos) doesn't violate any hiding intervals or collectives.

        Collects all execution and hiding intervals in the affected timeline regions,
        then checks:
        1. All bucket hiding compute stays between new start/wait
        2. No other collective's compute interval is enclosed by bucket execution interval
        3. No other collective's execution interval encloses bucket compute intervals
        """
        # Collect all collectives being bucketed
        all_bucketed_colls = [candidate] + list(bucket_info.collectives)
        all_bucketed_waits = [
            self.collective_info[coll].wait_node for coll in all_bucketed_colls
        ]

        # Collect hiding compute positions for the bucket
        bucket_hiding_compute_positions = []
        for coll in all_bucketed_colls:
````
- **EN**: Introduces function `_preserves_hiding_intervals`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bucket_info`, `candidate`, `start_pos`, `wait_pos`, `why`, `all_bucketed_colls`, and `...+2`.
- **CN**: 这里定义了函数`_preserves_hiding_intervals`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `bucket_info`、`candidate`、`start_pos`、`wait_pos`、`why`、`all_bucketed_colls`、`另有2项` 等值。

### Lines 645-672 / 第 645-672 行
````python
            for coll_hiding_node in self.collective_info[coll].hiding_nodes:
                bucket_hiding_compute_positions.append(
                    self.node_to_event[coll_hiding_node].position
                )

        # Get new positions
        new_start_event = self.node_to_event[start_pos]
        new_wait_event = self.node_to_event[wait_pos]

        # Check 1: All bucket hiding compute must be between new start and wait
        for compute_pos in bucket_hiding_compute_positions:
            if not (new_start_event.position < compute_pos < new_wait_event.position):
                why(
                    "hiding compute at pos %d not between start %d and wait %d",
                    compute_pos,
                    new_start_event.position,
                    new_wait_event.position,
                )
                return False

        def get_wait(n: fx.Node) -> fx.Node:
            return self.collective_info[n].wait_node

        def get_pos(n: fx.Node) -> int:
            return self.node_to_event[n].position

        latest_start_pos = max(get_pos(candidate), get_pos(bucket_info.collectives[0]))
        earliest_wait_pos = min(
````
- **EN**: Introduces function `get_wait`, function `get_pos`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_start_event`, `new_wait_event`, `latest_start_pos`, and `earliest_wait_pos`.
- **CN**: 这里定义了函数`get_wait`、函数`get_pos`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_start_event`、`new_wait_event`、`latest_start_pos`、`earliest_wait_pos` 等值。

### Lines 673-700 / 第 673-700 行
````python
            get_pos(get_wait(candidate)), get_pos(get_wait(bucket_info.collectives[0]))
        )

        # Bucket execution interval
        bucket_execution_interval = (new_start_event.position, new_wait_event.position)

        # Because collectives on the same PG operate under LIFO semantics,
        # it's only possible for us to force an early realization of an unrelated collective
        # by delaying a start or raising a wait.
        # We search in the interval from old_start -> new_start, to see if would be
        # forcing another collective to be realized prior to its hiding nodes.
        # Similarly, we search from old_wait -> new_wait, in the reverse direction,
        # to check the same thing.

        execution_intervals = [bucket_execution_interval]
        hiding_intervals = [
            (bucket_execution_interval[0], pos)
            for pos in bucket_hiding_compute_positions
        ]

        curr_event = new_start_event.next
        while curr_event is not None and curr_event.position < latest_start_pos:
            if (
                curr_event.node not in all_bucketed_colls
                and curr_event.node not in all_bucketed_waits
            ):
                exec_interval, hiding_interval_list = self._get_intervals(curr_event)
                if exec_interval:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bucket_execution_interval`, `execution_intervals`, `hiding_intervals`, and `curr_event`. This range continues the implementation of function `OverlapPreservingBucketer._preserves_hiding_intervals`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `bucket_execution_interval`、`execution_intervals`、`hiding_intervals`、`curr_event` 等值。这一段延续了函数`OverlapPreservingBucketer._preserves_hiding_intervals` 的具体实现。

### Lines 701-728 / 第 701-728 行
````python
                    execution_intervals.append(exec_interval)
                hiding_intervals.extend(hiding_interval_list)
            curr_event = curr_event.next

        curr_event = new_wait_event.prev
        while curr_event is not None and curr_event.position > earliest_wait_pos:
            if (
                curr_event.node not in all_bucketed_colls
                and curr_event.node not in all_bucketed_waits
            ):
                exec_interval, hiding_interval_list = self._get_intervals(curr_event)
                if exec_interval:
                    execution_intervals.append(exec_interval)
                hiding_intervals.extend(hiding_interval_list)
            curr_event = curr_event.prev

        # Check: no hiding interval should be enclosed by any execution interval
        def enclosed_interval(inner: tuple[int, int], outer: tuple[int, int]) -> bool:
            return outer[0] < inner[0] and inner[1] < outer[1]

        for hiding_interval in hiding_intervals:
            for execution_interval in execution_intervals:
                if enclosed_interval(hiding_interval, execution_interval):
                    why(
                        "hiding interval %s enclosed by execution interval %s",
                        hiding_interval,
                        execution_interval,
                    )
````
- **EN**: Introduces function `enclosed_interval`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `curr_event`.
- **CN**: 这里定义了函数`enclosed_interval`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `curr_event` 等值。

### Lines 729-756 / 第 729-756 行
````python
                    return False

        return True

    def remove_from_event(self, node: fx.Node) -> tuple[PGEvent | None, PGEvent | None]:
        """Remove node from timeline and return (prev_event, next_event)."""
        event = self.node_to_event[node]
        assert not event.is_compute, "Cannot remove compute events from timeline"

        prev_event, next_event = event.unlink()

        # Remove augmented graph dependency
        if prev_event:
            self.aug_graph.remove_extra_dep(n=node, dep=prev_event.node)
        if next_event:
            self.aug_graph.remove_extra_dep(n=next_event.node, dep=node)

        # Add bypass dependency
        if prev_event and next_event:
            self.aug_graph.add_extra_dep(n=next_event.node, dep=prev_event.node)

        return prev_event, next_event

    def restore_to_event(
        self,
        node: fx.Node,
        prev_event: PGEvent | None,
        next_event: PGEvent | None,
````
- **EN**: Introduces function `remove_from_event`, function `restore_to_event`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `event`, `node`, `prev_event`, and `next_event`.
- **CN**: 这里定义了函数`remove_from_event`、函数`restore_to_event`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `event`、`node`、`prev_event`、`next_event` 等值。

### Lines 757-784 / 第 757-784 行
````python
    ) -> None:
        """Restore node to timeline after failed merge attempt."""
        event = self.node_to_event[node]

        # Reinsert into linked list
        event.insert_between(prev_event, next_event)
        if prev_event:
            self.aug_graph.add_extra_dep(n=node, dep=prev_event.node)
        if next_event and not prev_event:
            self.aug_graph.add_extra_dep(n=next_event.node, dep=node)

        # Remove bypass dependency
        if prev_event and next_event:
            self.aug_graph.remove_extra_dep(n=next_event.node, dep=prev_event.node)

    def _try_timeline_position(
        self,
        bucket_info: CollBucket,
        candidate: fx.Node,
        start_pos: fx.Node,
        wait_pos: fx.Node,
        why: WhyNoBucket,
    ) -> bool:
        """
        Try a specific timeline position for the candidate.
        Returns True if valid and merges are successful.
        """
        candidate_info = self.collective_info[candidate]
````
- **EN**: Introduces function `_try_timeline_position`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `event`, `bucket_info`, `candidate`, `start_pos`, `wait_pos`, `why`, and `...+1`.
- **CN**: 这里定义了函数`_try_timeline_position`。包含分支、循环或上下文管理等控制流。初始化或更新了 `event`、`bucket_info`、`candidate`、`start_pos`、`wait_pos`、`why`、`另有1项` 等值。

### Lines 785-812 / 第 785-812 行
````python
        candidate_wait = candidate_info.wait_node

        # Quick check: does this violate hiding intervals?
        if not self._preserves_hiding_intervals(
            bucket_info, candidate, start_pos, wait_pos, why
        ):
            return False

        # Determine which start needs to move
        existing_coll = bucket_info.collectives[0]
        if start_pos == existing_coll:
            start_to_move = candidate
        else:
            assert start_pos == candidate
            start_to_move = existing_coll

        # Remove start from timeline
        start_prev, start_next = self.remove_from_event(start_to_move)

        # Check if starts can be merged
        if self.aug_graph.has_path(existing_coll, candidate) or self.aug_graph.has_path(
            candidate, existing_coll
        ):
            # Restore start constraints
            self.restore_to_event(start_to_move, start_prev, start_next)
            why("path exists between starts")
            return False

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `candidate_wait`, `existing_coll`, `start_to_move`, and `else`. This range continues the implementation of function `OverlapPreservingBucketer._try_timeline_position`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `candidate_wait`、`existing_coll`、`start_to_move`、`else` 等值。这一段延续了函数`OverlapPreservingBucketer._try_timeline_position` 的具体实现。

### Lines 813-840 / 第 813-840 行
````python
        # Merge starts
        self.aug_graph.merge_to_set(existing_coll, candidate)

        # Determine which wait needs to move
        existing_wait = self.collective_info[existing_coll].wait_node
        candidate_wait = self.collective_info[candidate].wait_node

        if wait_pos == existing_wait:
            wait_to_move = candidate_wait
        else:
            wait_to_move = existing_wait

        # Remove wait from timeline
        wait_prev, wait_next = self.remove_from_event(wait_to_move)

        # Check if waits can be merged
        if self.aug_graph.has_path(
            existing_wait, candidate_wait
        ) or self.aug_graph.has_path(candidate_wait, existing_wait):
            # Restore wait constraints
            self.restore_to_event(wait_to_move, wait_prev, wait_next)
            # Unmerge the start we just merged
            self.aug_graph.unmerge_node(candidate)
            # Restore start constraints
            self.restore_to_event(start_to_move, start_prev, start_next)
            why("path exists between waits")
            return False

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `existing_wait`, `candidate_wait`, `wait_to_move`, and `else`. This range continues the implementation of function `OverlapPreservingBucketer._try_timeline_position`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `existing_wait`、`candidate_wait`、`wait_to_move`、`else` 等值。这一段延续了函数`OverlapPreservingBucketer._try_timeline_position` 的具体实现。

### Lines 841-868 / 第 841-868 行
````python
        # Merge waits - success!
        self.aug_graph.merge_to_set(existing_wait, candidate_wait)

        # Update node_to_event for moved nodes
        target_start_event = self.node_to_event[start_pos]
        target_wait_event = self.node_to_event[wait_pos]

        self.node_to_event[candidate] = target_start_event
        self.node_to_event[candidate_wait] = target_wait_event
        self.node_to_event[existing_coll] = target_start_event
        self.node_to_event[existing_wait] = target_wait_event

        return True

    def _has_ancestor_conflicts(
        self, bucket_info: CollBucket, candidate: fx.Node
    ) -> bool:
        """
        Check if candidate has ancestor conflicts with bucket collectives.
        Returns True if there are conflicts.
        """
        candidate_info = self.collective_info[candidate]
        candidate_wait = candidate_info.wait_node

        for coll in bucket_info.collectives:
            if (
                coll in self.node_ancestors[candidate]
                or candidate in self.node_ancestors[coll]
````
- **EN**: Introduces function `_has_ancestor_conflicts`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `target_start_event`, `target_wait_event`, `candidate_info`, and `candidate_wait`.
- **CN**: 这里定义了函数`_has_ancestor_conflicts`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `target_start_event`、`target_wait_event`、`candidate_info`、`candidate_wait` 等值。

### Lines 869-896 / 第 869-896 行
````python
            ):
                return True

            # Check if waits are ancestors of each other
            coll_wait = self.collective_info[coll].wait_node
            if (
                coll_wait in self.node_ancestors[candidate_wait]
                or candidate_wait in self.node_ancestors[coll_wait]
            ):
                return True

            # Check if existing hiding node conflicts with candidate wait
            for old_hiding_node in self.collective_info[coll].hiding_nodes:
                if candidate_wait in self.node_ancestors[old_hiding_node]:
                    return True

            # Check if candidate hiding node conflicts with existing wait
            for new_hiding_node in candidate_info.hiding_nodes:
                if coll_wait in self.node_ancestors[new_hiding_node]:
                    return True

        return False

    def _can_add_to_bucket(
        self,
        bucket_info: CollBucket,
        candidate: fx.Node,
    ) -> bool:
````
- **EN**: Introduces function `_can_add_to_bucket`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `coll_wait`, `bucket_info`, and `candidate`.
- **CN**: 这里定义了函数`_can_add_to_bucket`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `coll_wait`、`bucket_info`、`candidate` 等值。

### Lines 897-924 / 第 897-924 行
````python
        """
        Check if candidate can be added to bucket without breaking comm/compute overlap.

        Strategy: Try all timeline positions - combinations of [existing_start, candidate_start]
        x [existing_wait, candidate_wait]. For each position, verify:
        1. Hiding intervals preserved - for any (start, hiding_compute, wait) interval, no other
           collective's (start, wait) pair falls between start and hiding_compute, which would
           force realization and break overlap due to LIFO semantics
        2. Topologically valid (no dependency cycles)

        Return True if any timeline position satisfies both constraints.
        """
        existing_coll = bucket_info.collectives[0]
        why = WhyNoBucket(existing_coll, candidate)

        candidate_info = self.collective_info[candidate]

        if (
            candidate in self.all_hiding_nodes
            or candidate_info.wait_node in self.all_hiding_nodes
        ):
            why("nyi: bucketing collective used for overlap")
            return False

        # Step 1: Quick check using precomputed ancestors
        # These ancestors are computed prior to adding augmented dependencies and not updated,
        # so if any of these checks fail then the merge will not be topologically valid
        # even ignoring comm/compute overlap
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Strategy`, `existing_coll`, `why`, and `candidate_info`. This range continues the implementation of function `OverlapPreservingBucketer._can_add_to_bucket`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Strategy`、`existing_coll`、`why`、`candidate_info` 等值。这一段延续了函数`OverlapPreservingBucketer._can_add_to_bucket` 的具体实现。

### Lines 925-952 / 第 925-952 行
````python
        if self._has_ancestor_conflicts(bucket_info, candidate):
            why("has ancestor conflicts")
            return False

        # Step 2: Try different rail positions
        existing_wait = self.collective_info[existing_coll].wait_node

        candidate_start = candidate
        candidate_wait = candidate_info.wait_node

        # Try combinations in order of likelihood to succeed
        # (early start, later wait is most likely to work)
        combinations = [
            (
                existing_coll,
                candidate_wait,
            ),  # Move candidate start early, keep wait late
            (
                existing_coll,
                existing_wait,
            ),  # Move candidate start early, move wait early
            (candidate_start, candidate_wait),  # Keep both in place
            (candidate_start, existing_wait),  # Keep start in place, move wait early
        ]

        for i, (start_pos, wait_pos) in enumerate(combinations):
            if self._try_timeline_position(
                bucket_info, candidate, start_pos, wait_pos, why
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `existing_wait`, `candidate_start`, `candidate_wait`, and `combinations`. This range continues the implementation of function `OverlapPreservingBucketer._can_add_to_bucket`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `existing_wait`、`candidate_start`、`candidate_wait`、`combinations` 等值。这一段延续了函数`OverlapPreservingBucketer._can_add_to_bucket` 的具体实现。

### Lines 953-980 / 第 953-980 行
````python
            ):
                bucket_log.debug(
                    "bucketed %s with %s using timeline position %d: (start=%s, wait=%s)",
                    candidate.name,
                    existing_coll.name,
                    i + 1,
                    start_pos.name,
                    wait_pos.name,
                )
                return True

        why("all timeline positions failed")
        return False

    def _apply_bucket(self, bucket_info: CollBucket) -> None:
        """
        Apply bucketing transformation.

        Dependencies are added to aug_graph.extra_deps and transferred from old nodes.
        """

        from torch._inductor.fx_passes.bucketing import (
            is_all_reduce_tensor,
            merge_all_gather_bucket,
            merge_all_reduce_bucket,
            merge_reduce_scatter_bucket,
        )

````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.bucketing` for the logic in this range. Introduces function `_apply_bucket`. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch._inductor.fx_passes.bucketing` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_apply_bucket`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 981-1008 / 第 981-1008 行
````python
        bucket = bucket_info.collectives

        # Collect old nodes BEFORE they're erased
        old_starts = list(bucket)
        old_waits = [self.collective_info[n].wait_node for n in bucket]

        fused_convert_dtypes = []
        for n in old_starts:
            if has_mergeable_all_gather_convert_dtype(n):
                fused_convert_dtypes.append(n.args[0])

        # Find where to place the bucketed operations
        next_node = bucket[0]
        while next_node in bucket:
            next_node = next_node.next

        # Don't use wait_insertion_point - let merge functions place waits naturally
        # The wait_insertion_point feature tries to move waits to a specific location,
        # but this can cause issues when that location is one of the nodes being erased
        # Create bucketed collective (this will erase old nodes)
        if is_all_gather(bucket[0]):
            new_nodes, replacements = merge_all_gather_bucket(
                self.graph,
                bucket,
                insert_before=next_node,
                mode=self.bucket_mode,
            )
        elif is_all_reduce_tensor(bucket[0]):
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bucket`, `old_starts`, `old_waits`, `fused_convert_dtypes`, `next_node`, `insert_before`, and `...+1`. This range continues the implementation of function `OverlapPreservingBucketer._apply_bucket`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `bucket`、`old_starts`、`old_waits`、`fused_convert_dtypes`、`next_node`、`insert_before`、`另有1项` 等值。这一段延续了函数`OverlapPreservingBucketer._apply_bucket` 的具体实现。

### Lines 1009-1036 / 第 1009-1036 行
````python
            new_nodes, replacements = merge_all_reduce_bucket(
                self.graph,
                bucket,
                mode=self.bucket_mode,
                insert_before=next_node,
            )
        else:
            assert is_reduce_scatter(bucket[0])
            new_nodes, replacements = merge_reduce_scatter_bucket(
                self.graph,
                bucket,
                insert_before=next_node,
                mode=self.bucket_mode,
            )

        # Identify the new wait(s) and their collective start in a single pass
        wait_to_start = {
            n: start
            for n in new_nodes
            if (start := _get_collective_node_from_wait(n)) is not None
        }
        new_waits = list(wait_to_start)

        # Create mapping of all erased nodes to their replacements
        erased_to_new: dict[fx.Node, fx.Node | None] = {}
        new_start = wait_to_start[new_waits[0]]
        if len(new_waits) == 1:
            # Standard bucketing: single start + single wait
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mode`, `insert_before`, `else`, `wait_to_start`, `n`, `new_waits`, and `...+2`. This range continues the implementation of function `OverlapPreservingBucketer._apply_bucket`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `mode`、`insert_before`、`else`、`wait_to_start`、`n`、`new_waits`、`另有2项` 等值。这一段延续了函数`OverlapPreservingBucketer._apply_bucket` 的具体实现。

### Lines 1037-1064 / 第 1037-1064 行
````python
            new_wait = new_waits[0]
            for old_start in old_starts:
                erased_to_new[old_start] = new_start
            for old_wait in old_waits:
                erased_to_new[old_wait] = new_wait
        else:
            # Coalesced bucketing: single start + N waits (one per original tensor)
            assert len(new_waits) == len(old_waits)
            for old_start in old_starts:
                erased_to_new[old_start] = new_start
            erased_to_new.update(dict(zip(old_waits, new_waits)))

        # Handle convert_element_type nodes that were fused and erased
        # The bucketed operation may have a _pre_bucket op that handles dtype conversion
        if fused_convert_dtypes:
            # In custom_ops mode, the _pre_bucket_all_gather node handles dtype conversion
            # In default mode, convert nodes are just erased — map them to new_start
            new_convert_dtypes_node = new_start.kwargs.get("out")
            if (
                isinstance(new_convert_dtypes_node, fx.Node)
                and new_convert_dtypes_node.target
                == torch.ops.bucketing._pre_bucket_all_gather.default
            ):
                replacement = new_convert_dtypes_node
            else:
                replacement = new_start

            for n in fused_convert_dtypes:
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `new_wait`, `else`, `new_convert_dtypes_node`, and `replacement`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `new_wait`、`else`、`new_convert_dtypes_node`、`replacement` 等值。

### Lines 1065-1092 / 第 1065-1092 行
````python
                erased_to_new[n] = replacement

        # Transfer all dependencies from old nodes to new nodes
        self.aug_graph.transfer_erased_node_deps(erased_to_new)


def finalize_overlap_scheduling(
    gm: fx.GraphModule,
    collective_info: dict[fx.Node, CollectiveInfo],
    scheduled: OrderedSet[fx.Node],
    *,
    collective_bucketing: bool = False,
    insert_overlap_deps: bool = False,
    max_bucket_memory_gb: float = 2.0,
    max_coll_distance: int = 1000,
    region_of: dict[fx.Node, Any] | None = None,
    bucket_exposed_first: bool | None = None,
    bucket_only_internode_comms: bool = False,
    bucket_mode: BucketMode | None = None,
) -> None:
    """
    Finalize overlap scheduling by applying deps, inlining fusions, and optionally bucketing.

    This is the main entry point for post-scheduling graph transformations:
    1. Bucket collectives (if collective_bucketing=True)
    2. Inline fusion regions back to original nodes
    3. Transfer deps from erased nodes to replacements
    4. Apply topological sort and effect tokens
````
- **EN**: Introduces function `finalize_overlap_scheduling`. Initializes or updates values such as `gm`, `collective_info`, `scheduled`, `collective_bucketing`, `insert_overlap_deps`, `max_bucket_memory_gb`, and `...+5`.
- **CN**: 这里定义了函数`finalize_overlap_scheduling`。初始化或更新了 `gm`、`collective_info`、`scheduled`、`collective_bucketing`、`insert_overlap_deps`、`max_bucket_memory_gb`、`另有5项` 等值。

### Lines 1093-1117 / 第 1093-1117 行
````python

    Args:
        gm: The graph module to modify
        collective_info: Dict mapping collective start nodes to their CollectiveInfo
        scheduled: Ordered set of scheduled nodes
        collective_bucketing: Whether to bucket collectives
        insert_overlap_deps: Whether to insert effect tokens for overlap deps
        max_bucket_memory_gb: Maximum memory for a bucket in GB
        max_coll_distance: Maximum distance for bucketing candidates
        region_of: Optional dict mapping module nodes to FusionRegions
    """
    bucketer = OverlapPreservingBucketer(
        graph=gm.graph,
        collective_info=collective_info,
        scheduled=scheduled,
        max_bucket_memory_gb=max_bucket_memory_gb,
        max_coll_distance=max_coll_distance,
        insert_overlap_deps=insert_overlap_deps,
        collective_bucketing=collective_bucketing,
        bucket_exposed_first=bucket_exposed_first,
        bucket_only_internode_comms=bucket_only_internode_comms,
        region_of=region_of,
        bucket_mode=bucket_mode,
    )
    bucketer.bucket_collectives()
````
- **EN**: Initializes or updates values such as `Args`, `gm`, `collective_info`, `scheduled`, `collective_bucketing`, `insert_overlap_deps`, and `...+8`. This range continues the implementation of function `finalize_overlap_scheduling`.
- **CN**: 初始化或更新了 `Args`、`gm`、`collective_info`、`scheduled`、`collective_bucketing`、`insert_overlap_deps`、`另有8项` 等值。这一段延续了函数`finalize_overlap_scheduling` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `WhyNoBucket`, `PGEvent`, and `OverlapPreservingBucketer`  
  **CN**: 主要类：`WhyNoBucket`、`PGEvent`、`OverlapPreservingBucketer`
- **EN**: Primary functions: `is_collective_or_wait`, and `finalize_overlap_scheduling`  
  **CN**: 主要函数：`is_collective_or_wait`、`finalize_overlap_scheduling`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `logging`, `collections`, `dataclasses`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `torch._dynamo.utils`, `torch._inductor.augmented_graph_helper`, `torch._inductor.fx_passes.bucketing`, `torch._inductor.fx_passes.fsdp`, `torch._inductor.fx_passes.overlap_scheduling`, `torch._logging`, `torch.utils._ordered_set`, `torch._dynamo.graph_deduplication`, `torch._inductor.fx_passes.fusion_regions`, `torch._inductor.fx_passes.control_dependencies`
