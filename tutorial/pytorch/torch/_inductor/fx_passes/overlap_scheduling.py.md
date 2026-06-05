# overlap_scheduling.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/overlap_scheduling.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `WhyNoOverlap`, `CollectiveInfo`, `CollBucket`, and `OverlapScheduler`. It exposes functions such as `make_all_device_put_sync`, `get_group_name`, `get_custom_estimation`, `estimate_collective_time`, `is_compute_node`, `estimate_roofline_runtime_ms`, and `...+12`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `WhyNoOverlap`、`CollectiveInfo`、`CollBucket`、`OverlapScheduler` 等类。同时提供 `make_all_device_put_sync`、`get_group_name`、`get_custom_estimation`、`estimate_collective_time`、`is_compute_node`、`estimate_roofline_runtime_ms`、`另有12项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
import functools
import heapq
import itertools
import logging
import sys
from collections import Counter, defaultdict
from collections.abc import Callable, Iterable
from dataclasses import dataclass, field
from typing import Any, Literal

import torch
import torch.fx as fx
from torch._dynamo.utils import counters, dynamo_timed
from torch._inductor import config
from torch._inductor.comm_analysis import estimate_fx_collective_memory_footprint
from torch._inductor.fx_passes.bucketing import (
    _default_bucket_mode,
    _get_collective_node_from_wait,
    _schedulable_wait_node,
    bucket_key,
    BucketMode,
    get_full_bucket_key,
    is_wait_tensor,
)
from torch._inductor.fx_passes.memory_estimator import MemoryTracker
from torch._logging import trace_structured
from torch.fx.operator_schemas import normalize_function
from torch.utils._ordered_set import OrderedSet
````
- **EN**: Imports dependencies such as `functools`, `heapq`, `itertools`, `logging`, `sys`, `collections`, and `...+13` for the logic in this range.
- **CN**: 这里导入了 `functools`、`heapq`、`itertools`、`logging`、`sys`、`collections`、`另有13项` 等依赖，为后续逻辑提供基础能力。

### Lines 29-56 / 第 29-56 行
````python
from torch.utils._python_dispatch import _disable_current_modes


log = logging.getLogger(__name__)

from torch._inductor.pattern_matcher import stable_topological_sort


def make_all_device_put_sync(gm: torch.fx.GraphModule) -> int:
    """
    Convert all non_blocking=True device_put operations to non_blocking=False.

    Only performs the conversion if at least one non_blocking=True device_put
    exists in the graph.

    Returns:
        The number of device_put operations converted to sync.
    """
    g = gm.graph
    device_put_nodes = list(
        g.find_nodes(op="call_function", target=torch.ops.prims.device_put.default)
    )

    # Check if any non_blocking=True device_put exists
    has_async_device_put = False
    for n in device_put_nodes:
        opt_args_kwargs = normalize_function(
            n.target,
````
- **EN**: Imports dependencies such as `torch.utils._python_dispatch`, and `torch._inductor.pattern_matcher` for the logic in this range. Introduces function `make_all_device_put_sync`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `log`, `Returns`, `g`, `device_put_nodes`, `has_async_device_put`, and `opt_args_kwargs`.
- **CN**: 这里导入了 `torch.utils._python_dispatch`、`torch._inductor.pattern_matcher` 等依赖，为后续逻辑提供基础能力。这里定义了函数`make_all_device_put_sync`。包含分支、循环或上下文管理等控制流。初始化或更新了 `log`、`Returns`、`g`、`device_put_nodes`、`has_async_device_put`、`opt_args_kwargs` 等值。

### Lines 57-84 / 第 57-84 行
````python
            args=n.args,
            kwargs=n.kwargs,
            normalize_to_only_use_kwargs=True,
        )
        if opt_args_kwargs is not None:
            _, kwargs = opt_args_kwargs
            if kwargs.get("non_blocking", False):
                has_async_device_put = True
                break

    if not has_async_device_put:
        return 0

    # Convert all non_blocking=True to non_blocking=False
    count = 0
    for n in device_put_nodes:
        opt_args_kwargs = normalize_function(
            n.target,
            args=n.args,
            kwargs=n.kwargs,
            normalize_to_only_use_kwargs=True,
        )
        if opt_args_kwargs is not None:
            _, kwargs = opt_args_kwargs
            if kwargs.get("non_blocking", False):
                kwargs["non_blocking"] = False
                n.args = n.args[0], kwargs["device"], kwargs["non_blocking"]
                n.kwargs = {}
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `args`, `kwargs`, `normalize_to_only_use_kwargs`, `has_async_device_put`, `count`, and `opt_args_kwargs`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `args`、`kwargs`、`normalize_to_only_use_kwargs`、`has_async_device_put`、`count`、`opt_args_kwargs` 等值。

### Lines 85-112 / 第 85-112 行
````python
                count += 1

    return count


@dataclass
class WhyNoOverlap:
    """Track reasons why a collective cannot overlap with compute."""

    compute_name: str
    collective_name: str

    def __init__(self, compute_node: fx.Node, collective_node: fx.Node) -> None:
        self.compute_name = compute_node.name
        self.collective_name = collective_node.name

    def __call__(self, reason: str, *args: Any) -> None:
        if log.isEnabledFor(logging.DEBUG):
            log.debug(
                "cannot overlap %s with %s: " + reason,  # noqa: G003
                self.collective_name,
                self.compute_name,
                *args,
            )


def get_group_name(n: fx.Node) -> str:
    """Extract the group name from a collective operation node."""
````
- **EN**: Introduces class `WhyNoOverlap`, function `__init__`, function `__call__`, function `get_group_name`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`WhyNoOverlap`、函数`__init__`、函数`__call__`、函数`get_group_name`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 113-140 / 第 113-140 行
````python
    opt_args_kwargs = normalize_function(
        n.target,  # type: ignore[arg-type]
        args=n.args,
        kwargs=n.kwargs,
        normalize_to_only_use_kwargs=True,
    )
    assert opt_args_kwargs is not None
    _, kwargs = opt_args_kwargs
    return kwargs["group_name"]


def get_custom_estimation(
    n: fx.Node,
    custom_runtime_estimation: Callable[[fx.Node, int | None], float | None]
    | None = None,
    override_size: int | None = None,
) -> float | None:
    if custom_runtime_estimation is None:
        return None

    return custom_runtime_estimation(n, override_size)


def estimate_collective_time(
    n: fx.Node,
    override_size: int | None = None,
    custom_runtime_estimation: Callable[[fx.Node, int | None], float | None]
    | None = None,
````
- **EN**: Introduces function `get_custom_estimation`, function `estimate_collective_time`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `opt_args_kwargs`, `args`, `kwargs`, `normalize_to_only_use_kwargs`, `n`, `custom_runtime_estimation`, and `...+1`.
- **CN**: 这里定义了函数`get_custom_estimation`、函数`estimate_collective_time`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `opt_args_kwargs`、`args`、`kwargs`、`normalize_to_only_use_kwargs`、`n`、`custom_runtime_estimation`、`另有1项` 等值。

### Lines 141-168 / 第 141-168 行
````python
    collective_estimator: Literal["analytical", "benchmark"] = "analytical",
) -> float:
    """Estimate the runtime of a collective operation, optionally with an overridden size."""
    if (
        est := get_custom_estimation(n, custom_runtime_estimation, override_size)
    ) is not None:
        return est

    if collective_estimator == "benchmark":
        from torch._inductor.fx_passes.node_runtime_estimation import (
            benchmark_collective_with_cuda_events,
        )

        cuda_val, _ = benchmark_collective_with_cuda_events(n, nruns=5)
        if cuda_val is not None:
            return cuda_val

    # Analytical model (also fallback when benchmark returns None)
    return torch._inductor.comm_analysis.estimate_nccl_collective_runtime_from_fx_node(
        n, override_size
    )


def is_compute_node(n: fx.Node) -> bool:
    """
    Should we consider this node computationally expensive ?
    Currently uses flop registration, but we could expand more generally.
    """
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.node_runtime_estimation` for the logic in this range. Introduces function `is_compute_node`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch._inductor.fx_passes.node_runtime_estimation` 等依赖，为后续逻辑提供基础能力。这里定义了函数`is_compute_node`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 169-196 / 第 169-196 行
````python
    return (
        getattr(n.target, "overloadpacket", None)
        in torch.utils.flop_counter.flop_registry
    )


def estimate_roofline_runtime_ms(node: fx.Node) -> float:
    """Estimate runtime using roofline model (max of compute and memory bound).

    Uses FLOPs for compute-bound estimate if op is in flop_registry,
    and memory bandwidth for memory-bound estimate.
    Returns 0 for view nodes (no cost).
    """
    from torch._inductor.fx_passes.fusion_regions import is_view_node
    from torch.utils._pytree import tree_flatten, tree_map
    from torch.utils._runtime_estimation import get_compute_time, get_transfer_time
    from torch.utils.flop_counter import flop_registry

    if is_view_node(node):
        return 0.0

    def _get_val(n: Any) -> Any:
        if isinstance(n, fx.Node):
            return n.meta.get("val")
        return n

    args = tree_map(_get_val, node.args)
    kwargs = tree_map(_get_val, node.kwargs)
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.fusion_regions`, `torch.utils._pytree`, `torch.utils._runtime_estimation`, and `torch.utils.flop_counter` for the logic in this range. Introduces function `estimate_roofline_runtime_ms`, function `_get_val`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch._inductor.fx_passes.fusion_regions`、`torch.utils._pytree`、`torch.utils._runtime_estimation`、`torch.utils.flop_counter` 等依赖，为后续逻辑提供基础能力。这里定义了函数`estimate_roofline_runtime_ms`、函数`_get_val`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 197-224 / 第 197-224 行
````python
    out = _get_val(node)

    if out is None:
        return 0.0

    flat_args_kwargs, _ = tree_flatten((args, kwargs))
    flat_outs, _ = tree_flatten(out)
    out_dtypes = OrderedSet([t.dtype for t in flat_outs if isinstance(t, torch.Tensor)])

    # Compute time (FLOPs-based, only if op is in flop_registry)
    # May return SymFloat if shapes are symbolic (after flop division)
    compute_ns: float = 0.0
    func_packet = getattr(node.target, "overloadpacket", None)
    if func_packet in flop_registry and len(out_dtypes) == 1:
        compute_ns = get_compute_time(func_packet, args, kwargs, out, out_dtypes.copy())
        # Extract hint from symbolic value if needed
        if isinstance(compute_ns, (torch.SymInt, torch.SymFloat)):
            compute_ns = compute_ns.node.hint if compute_ns.node.has_hint() else 0.0

    # Transfer time (memory bandwidth-based, uses size_hint internally)
    transfer_ns = get_transfer_time(flat_args_kwargs, flat_outs)

    # Roofline: max of compute and transfer, convert ns to ms
    return max(float(compute_ns), float(transfer_ns)) / 1e6


def get_hint(x: int | torch.SymInt) -> int | None:
    if isinstance(x, int):
````
- **EN**: Introduces function `get_hint`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`get_hint`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 225-252 / 第 225-252 行
````python
        return x
    assert isinstance(x, torch.SymInt)
    if not x.node.has_hint():
        return None
    return x.node.hint


def get_collective_do_bench() -> Callable[[Callable[[], Any]], float]:
    with dynamo_timed("collective_compute_do_bench"):
        return functools.partial(
            # pyrefly: ignore [bad-argument-type]
            torch._inductor.runtime.benchmarking.benchmarker.benchmark_gpu,
            warmup=5,
        )


def benchmark_node_with_cache_key(
    n: fx.Node,
    custom_runtime_estimation: Callable[[fx.Node, int | None], float | None]
    | None = None,
) -> tuple[float, str | None]:
    """Benchmark a compute node and return (runtime, cache_key)."""
    assert is_compute_node(n)

    from torch._dynamo.testing import rand_strided

    # todo - skip unbacked, symbolic
    success, args, kwargs = torch._inductor.fx_utils.get_fake_args_kwargs(n)
````
- **EN**: Imports dependencies such as `torch._dynamo.testing` for the logic in this range. Introduces function `get_collective_do_bench`, function `benchmark_node_with_cache_key`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Processes profiling, FLOP, or benchmark information to characterize execution cost.
- **CN**: 这里导入了 `torch._dynamo.testing` 等依赖，为后续逻辑提供基础能力。这里定义了函数`get_collective_do_bench`、函数`benchmark_node_with_cache_key`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。

### Lines 253-280 / 第 253-280 行
````python

    if not success:
        return 0, None

    unbacked_tensor = False

    key = f"{str(n.target)}: "

    def to_real(t: torch.Tensor) -> torch.Tensor | None:
        shape = [get_hint(dim) for dim in t.shape]
        stride = [get_hint(s) for s in t.stride()]

        if any(s is None for s in itertools.chain(shape, stride)):
            nonlocal unbacked_tensor
            unbacked_tensor = True
            return None

        nonlocal key
        key += f"T: {shape, stride, t.dtype} "
        return rand_strided(shape, stride, device=t.device, dtype=t.dtype)  # type: ignore[arg-type]

    with _disable_current_modes():
        args, kwargs = torch.utils._pytree.tree_map_only(
            torch.Tensor,
            lambda t: to_real(t),
            (args, kwargs),
        )

````
- **EN**: Introduces function `to_real`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`to_real`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 281-308 / 第 281-308 行
````python
        if val := get_cached_node_time(key):
            return val, key

        if unbacked_tensor:
            return 0, key

        if (
            est := get_custom_estimation(n, custom_runtime_estimation, None)
        ) is not None:
            set_cached_node_time(key, est)
            return est, key

        bench = get_collective_do_bench()
        out = bench(lambda: n.target(*args, **kwargs))  # type: ignore[operator]
        set_cached_node_time(key, out)
        return out, key


def benchmark_node(
    n: fx.Node,
    custom_runtime_estimation: Callable[[fx.Node, int | None], float | None]
    | None = None,
) -> float:
    return benchmark_node_with_cache_key(n, custom_runtime_estimation)[0]


@functools.cache
def get_benchmark_cache() -> torch._inductor.codecache.LocalCache:
````
- **EN**: Introduces function `benchmark_node`, function `get_benchmark_cache`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`benchmark_node`、函数`get_benchmark_cache`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 309-336 / 第 309-336 行
````python
    return torch._inductor.codecache.LocalCache()


def get_cached_node_time(key: str) -> float:
    return get_benchmark_cache().lookup(key)  # type: ignore[return-value]


def set_cached_node_time(key: str, value: float) -> None:
    return get_benchmark_cache().set_value(key, value=value)


@dataclass
class CollectiveInfo:
    """Track info about a collective operation"""

    start_node: fx.Node
    wait_node: fx.Node
    size_bytes: int
    estimated_time_ms: float
    exposed_time_ms: float  # How much of this collective is still exposed
    hiding_nodes: OrderedSet[fx.Node] = field(default_factory=OrderedSet)

    @property
    def is_exposed(self) -> bool:
        return self.exposed_time_ms != 0


@dataclass
````
- **EN**: Introduces function `get_cached_node_time`, function `set_cached_node_time`, class `CollectiveInfo`, function `is_exposed`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里定义了函数`get_cached_node_time`、函数`set_cached_node_time`、类`CollectiveInfo`、函数`is_exposed`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 337-364 / 第 337-364 行
````python
class CollBucket:
    """Track information about a bucket of collectives."""

    collectives: list[fx.Node]  # Original collective starts
    bucketed_start: fx.Node | None = None  # After bucketing
    bucketed_wait: fx.Node | None = None  # After bucketing
    total_bytes: int = 0


def gb_to_bytes(gb: float) -> int:
    """Convert gigabytes to bytes."""
    return int(gb * 1024 * 1024 * 1024)


class OverlapScheduler:
    """
    Scheduler that reorders operations to maximize compute-collective overlap.

    The reordering is done as a scheduling pass. We maintain a priority queue of
    schedulable nodes. The nodes are ranked by:

    1) the compute node index they dominate. this allows reordering locally, such as with
    parallel mms, and also allows overlapping reduce scatter nodes outputs in the backward
    with compute by deferring their waits.

    2) whether the current node is a collective or wait that is currently exposed but has a compute
    node which it could be overlapped with.

````
- **EN**: Introduces class `CollBucket`, function `gb_to_bytes`, class `OverlapScheduler`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `collectives`, `bucketed_start`, `bucketed_wait`, and `total_bytes`.
- **CN**: 这里定义了类`CollBucket`、函数`gb_to_bytes`、类`OverlapScheduler`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `collectives`、`bucketed_start`、`bucketed_wait`、`total_bytes` 等值。

### Lines 365-392 / 第 365-392 行
````python
    3) original order in the graph for stability.

    When we schedule compute nodes, we first overlap exposed in-flight collectives, then look for unscheduled
    collectives that can be scheduled concurrently.

    TODO:
        - experiment with other priority scores / allow other mechanisms of reorder / more strict adherence to original graph
        - memory limit for deferred scheduling of reduce_scatter nodes.
    """

    def __init__(
        self,
        gm: torch.fx.GraphModule,
        max_in_flight_gb: float,
        max_compute_pre_fetch: int,
        collective_bucketing: bool,
        insert_overlap_deps: bool,
        compute_overlap_multipler: float,
        max_coll_distance: int,
        custom_runtime_estimation: Callable[[fx.Node, int | None], float | None]
        | None = None,
        collective_estimator: Literal["analytical", "benchmark"] = "analytical",
        compute_estimator: Literal["analytical", "benchmark"] = "benchmark",
        max_memory_increase_gb: float | None = 1.0,
        max_memory_increase_ratio: float | None = 0.05,
        log_final_collectives_estimations: bool = False,
        bucket_exposed_first: bool | None = None,
        enable_fusion_regions: bool = False,
````
- **EN**: Introduces function `__init__`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Initializes or updates values such as `TODO`, `gm`, `max_in_flight_gb`, `max_compute_pre_fetch`, `collective_bucketing`, `insert_overlap_deps`, and `...+10`.
- **CN**: 这里定义了函数`__init__`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。初始化或更新了 `TODO`、`gm`、`max_in_flight_gb`、`max_compute_pre_fetch`、`collective_bucketing`、`insert_overlap_deps`、`另有10项` 等值。

### Lines 393-420 / 第 393-420 行
````python
        bucket_only_internode_comms: bool = False,
        bucket_mode: BucketMode | None = None,
        max_off_bucket_gb: float | None = 0.5,
        prioritize_bucketing_during_scheduling: bool = True,
    ):
        self.gm = gm
        self.graph = gm.graph
        self.compute_overlap_multipler = compute_overlap_multipler
        self.max_node_distance = max_coll_distance
        self.max_in_flight_bytes: int = gb_to_bytes(max_in_flight_gb)
        self.custom_runtime_estimation = custom_runtime_estimation
        self.collective_bucketing = collective_bucketing
        self.insert_overlap_deps = insert_overlap_deps
        self.max_compute_pre_fetch = max_compute_pre_fetch
        # In deterministic mode, force analytical estimation to avoid GPU sync
        if config.deterministic:
            self.collective_estimator = "analytical"
            self.compute_estimator = "analytical"
        else:
            self.collective_estimator = collective_estimator
            self.compute_estimator = compute_estimator
        self.log_final_collectives_estimations = log_final_collectives_estimations
        self.bucket_exposed_first = bucket_exposed_first
        self.bucket_only_internode_comms = bucket_only_internode_comms
        self.bucket_mode = bucket_mode or _default_bucket_mode()
        self.max_off_bucket_bytes: int | None = (
            gb_to_bytes(max_off_bucket_gb) if max_off_bucket_gb is not None else None
        )
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bucket_only_internode_comms`, `bucket_mode`, `max_off_bucket_gb`, `prioritize_bucketing_during_scheduling`, and `else`. This range continues the implementation of function `OverlapScheduler.__init__`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `bucket_only_internode_comms`、`bucket_mode`、`max_off_bucket_gb`、`prioritize_bucketing_during_scheduling`、`else` 等值。这一段延续了函数`OverlapScheduler.__init__` 的具体实现。

### Lines 421-448 / 第 421-448 行
````python
        self.prioritize_bucketing_during_scheduling = (
            prioritize_bucketing_during_scheduling
        )

        # Make all to(device) non_blocking=False,
        # They can be implicitly depending by user logic on other to(device) non_blocking=True.
        # OverlapScheduler can put reads of non_blocking device_put before blocking one.
        # This results in dirty reads.
        num_device_put_converted = make_all_device_put_sync(gm)
        if num_device_put_converted > 0:
            log.warning(
                "overlap_scheduling converted %d device_put operations from "
                "non_blocking=True to non_blocking=False. This may affect performance.",
                num_device_put_converted,
            )

        # Build fusion regions (mutates gm.graph) and compute initial node runtime
        # estimates. Compute nodes use roofline model here; the alignment step in
        # run() replaces them with benchmarked + cross-rank-aligned values.
        self.node_estimations, self.region_of = gather_node_runtime_estimations(
            gm,
            custom_runtime_estimation,
            enable_fusion_regions=enable_fusion_regions,
            log_estimations=True,
        )
        if self.region_of:
            # fuse_by_partitions replaces gm.graph, so we need to update our reference
            self.graph = gm.graph
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `num_device_put_converted`, `enable_fusion_regions`, and `log_estimations`. This range continues the implementation of function `OverlapScheduler.__init__`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `num_device_put_converted`、`enable_fusion_regions`、`log_estimations` 等值。这一段延续了函数`OverlapScheduler.__init__` 的具体实现。

### Lines 449-476 / 第 449-476 行
````python

        # Build structures
        stable_topological_sort(self.graph)
        self.nodes = list(self.graph.nodes)
        self.node_idx = {n: i for i, n in enumerate(self.nodes)}
        self.node_ancestors: dict[fx.Node, OrderedSet[fx.Node]] = (
            self._collect_node_ancestors()
        )

        # Identify collectives and compute nodes
        self.collective_info: dict[fx.Node, CollectiveInfo] = {}
        self.unscheduled_collectives: OrderedSet[fx.Node] = OrderedSet()

        # Identify compute nodes early (needed for baseline memory computation)
        self.compute_nodes = [n for n in self.nodes if is_compute_node(n)]
        self.current_compute_index = 0

        # Compute baseline memory profile from original schedule
        self.original_mem_before_compute_index: list[int] = []
        self.original_peak_memory = self._compute_baseline_memory()

        # Maximum allowed peak memory = baseline + max(absolute, ratio * baseline)
        # When both limits are specified, use the more permissive one
        memory_increase_bytes = None
        if max_memory_increase_gb is not None:
            memory_increase_bytes = gb_to_bytes(max_memory_increase_gb)
        if max_memory_increase_ratio is not None:
            ratio_increase = int(self.original_peak_memory * max_memory_increase_ratio)
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `memory_increase_bytes`, and `ratio_increase`. This range continues the implementation of function `OverlapScheduler.__init__`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含分支、循环或上下文管理等控制流。初始化或更新了 `memory_increase_bytes`、`ratio_increase` 等值。这一段延续了函数`OverlapScheduler.__init__` 的具体实现。

### Lines 477-504 / 第 477-504 行
````python
            memory_increase_bytes = (
                max(memory_increase_bytes, ratio_increase)
                if memory_increase_bytes is not None
                else ratio_increase
            )
        if memory_increase_bytes is None:
            memory_increase_bytes = 0

        self.allowed_peak_memory_bytes = (
            self.original_peak_memory + memory_increase_bytes
        )

        # Track cumulative prefetch memory at each compute index
        # When we prefetch a collective at compute index i that will be used at index j,
        # it adds memory from i to j, so we need to track this cumulative effect
        self.cumulative_prefetch_mem_by_compute_index: list[int] = [
            0 for _ in range(len(self.compute_nodes))
        ]

        self.memory_tracker = MemoryTracker(self.graph)

        self.wait_to_start: dict[fx.Node, fx.Node] = {}
        self._identify_collectives()
        self.wasted_compute = 0.0

        # Calculate domination indices for both compute and reduce_scatter nodes
        self.reduce_scatter_nodes = self.graph.find_nodes(
            op="call_function",
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `memory_increase_bytes`, and `op`. This range continues the implementation of function `OverlapScheduler.__init__`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `memory_increase_bytes`、`op` 等值。这一段延续了函数`OverlapScheduler.__init__` 的具体实现。

### Lines 505-532 / 第 505-532 行
````python
            target=torch.ops._c10d_functional.reduce_scatter_tensor.default,
        )
        self.compute_index_domination = self._calculate_domination_index(
            self.compute_nodes
        )
        self.reduce_scatter_domination = self._calculate_domination_index(
            self.reduce_scatter_nodes
        )

        # Scheduling state
        self.potentially_hidden_collectives = (
            self.compute_potential_hidden_collectives()
        )
        self.potentially_hidden_waits = self.compute_potential_hidden_waits()
        self.in_degree = Counter(user for node in self.nodes for user in node.users)

        # Two separate queues: on-path (domination-based) and off-path (node_idx-based)
        self.on_path_ready: list[tuple[object, fx.Node]] = []
        self.off_path_ready: list[tuple[object, fx.Node]] = []
        # Track potential bucket sizes for off-path collectives (for batch scheduling)
        self.off_path_ready_potential_buckets: dict[object, int] = defaultdict(int)

        for node in self.nodes:
            if self.in_degree[node] == 0:
                self._add_to_ready_queue(node)

        self.in_flight: dict[fx.Node, CollectiveInfo] = {}  # start -> info
        self.in_flight_bytes = 0
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `target`. This range continues the implementation of function `OverlapScheduler.__init__`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `target` 等值。这一段延续了函数`OverlapScheduler.__init__` 的具体实现。

### Lines 533-560 / 第 533-560 行
````python
        self.scheduled: OrderedSet[fx.Node] = OrderedSet()
        self.max_compute_pre_fetch = max_compute_pre_fetch

        self.last_on_path_node_idx = -1

    def _add_to_ready_queue(self, node: fx.Node) -> None:
        if self.off_compute_path(node):
            score = self._compute_off_path_score(node)
            heapq.heappush(self.off_path_ready, (score, node))
            # Track potential bucket sizes for off-path ready collectives
            if node in self.collective_info:
                bucket_key = get_full_bucket_key(node, self.bucket_mode)
                node_bytes = self.collective_info[node].size_bytes
                self.off_path_ready_potential_buckets[bucket_key] += node_bytes
        else:
            score = self._compute_on_path_score(node)
            heapq.heappush(self.on_path_ready, (score, node))

    def _collect_node_ancestors(self) -> dict[fx.Node, OrderedSet[fx.Node]]:
        """Collect all ancestors for each node."""
        ancestors: dict[fx.Node, OrderedSet[fx.Node]] = defaultdict(OrderedSet)
        for node in self.nodes:
            for input_node in node.all_input_nodes:
                ancestors[node].add(input_node)
                ancestors[node] |= ancestors[input_node]

        return ancestors

````
- **EN**: Introduces function `_add_to_ready_queue`, function `_collect_node_ancestors`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `score`, `bucket_key`, `node_bytes`, `else`, and `ancestors`.
- **CN**: 这里定义了函数`_add_to_ready_queue`、函数`_collect_node_ancestors`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `score`、`bucket_key`、`node_bytes`、`else`、`ancestors` 等值。

### Lines 561-588 / 第 561-588 行
````python
    def _compute_baseline_memory(self) -> int:
        """
        Simulate the original schedule to compute baseline memory profile.
        Returns the peak memory observed during simulation.
        """
        baseline_tracker = MemoryTracker(self.graph)

        last_compute_max_memory = 0
        peak_memory = 0

        for node in self.nodes:
            baseline_tracker.schedule_node(node)
            current_mem = baseline_tracker.current_memory_bytes

            # Record the max memory between this and previous compute node
            last_compute_max_memory = max(last_compute_max_memory, current_mem)

            if is_compute_node(node):
                self.original_mem_before_compute_index.append(last_compute_max_memory)
                last_compute_max_memory = current_mem

            peak_memory = max(peak_memory, current_mem)

        return peak_memory

    def _prefetch_would_exceed_memory_budget(self, start_node: fx.Node) -> bool:
        """
        Check if prefetching this collective would exceed memory budget at ANY compute node
````
- **EN**: Introduces function `_compute_baseline_memory`, function `_prefetch_would_exceed_memory_budget`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_compute_baseline_memory`、函数`_prefetch_would_exceed_memory_budget`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 589-616 / 第 589-616 行
````python
        between now and when it's used.
        """
        info = self.collective_info[start_node]
        size = info.size_bytes

        domination_index = self.compute_index_domination[start_node]

        # If off-path, assume it doesn't increase memory
        if domination_index == sys.maxsize:
            return False

        # check current mem
        if (
            self.memory_tracker.current_memory_bytes + size
            > self.allowed_peak_memory_bytes
        ):
            return True

        start_index = self.current_compute_index

        # then, check future mem
        for compute_idx in range(start_index, domination_index):
            cumulative_prefetch = self.cumulative_prefetch_mem_by_compute_index[
                compute_idx
            ]

            # Check 1: Would cumulative prefetch exceed in-flight limit?
            if (cumulative_prefetch + size) > self.max_in_flight_bytes:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `info`, `size`, `domination_index`, `start_index`, and `cumulative_prefetch`. This range continues the implementation of function `OverlapScheduler._prefetch_would_exceed_memory_budget`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `info`、`size`、`domination_index`、`start_index`、`cumulative_prefetch` 等值。这一段延续了函数`OverlapScheduler._prefetch_would_exceed_memory_budget` 的具体实现。

### Lines 617-644 / 第 617-644 行
````python
                return True

            # Check 2: Would total memory (baseline + cumulative prefetch) exceed budget?
            baseline_mem = self.original_mem_before_compute_index[compute_idx]
            projected = baseline_mem + cumulative_prefetch + size

            if projected > self.allowed_peak_memory_bytes:
                return True

        return False

    def _update_cumulative_prefetch_memory(
        self, collective: fx.Node, info: CollectiveInfo
    ) -> None:
        """
        Update cumulative prefetch memory for all compute indices this collective will be live.
        """
        domination_index = self.compute_index_domination[collective]
        if domination_index == sys.maxsize:
            return

        for compute_idx in range(self.current_compute_index, domination_index):
            self.cumulative_prefetch_mem_by_compute_index[compute_idx] += (
                info.size_bytes
            )

    def off_compute_path(self, n: fx.Node) -> bool:
        """Check if a node is off the compute path (doesn't block any compute)."""
````
- **EN**: Introduces function `_update_cumulative_prefetch_memory`, function `off_compute_path`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `baseline_mem`, `projected`, and `domination_index`.
- **CN**: 这里定义了函数`_update_cumulative_prefetch_memory`、函数`off_compute_path`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `baseline_mem`、`projected`、`domination_index` 等值。

### Lines 645-672 / 第 645-672 行
````python
        return self.compute_index_domination[n] == sys.maxsize

    def dominates_reduce_scatter(self, n: fx.Node) -> bool:
        """Check if a node dominates (blocks) any reduce_scatter."""
        return self.reduce_scatter_domination[n] != sys.maxsize

    def _identify_collectives(self) -> None:
        """Identify all collective operations and process groups."""
        self.all_pgs: OrderedSet[str] = OrderedSet()

        for node in self.nodes:
            if _schedulable_wait_node(node):
                start = _get_collective_node_from_wait(node)
                assert start is not None
                assert start in self.node_estimations, (
                    f"Missing estimation for collective {start.name}. "
                    f"Ensure custom_runtime_estimation returns a value for this node."
                )
                self.wait_to_start[node] = start
                # For coalesced collectives, multiple waits share the same
                # start node. Only register the first wait as the representative.
                if start in self.collective_info:
                    continue
                coll_time_ms = self.node_estimations[start]

                info = CollectiveInfo(
                    start_node=start,
                    wait_node=node,
````
- **EN**: Introduces function `dominates_reduce_scatter`, function `_identify_collectives`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`dominates_reduce_scatter`、函数`_identify_collectives`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 673-700 / 第 673-700 行
````python
                    size_bytes=estimate_fx_collective_memory_footprint(start),
                    estimated_time_ms=coll_time_ms,
                    exposed_time_ms=coll_time_ms,  # Initially fully exposed
                )
                self.collective_info[start] = info
                self.unscheduled_collectives.add(start)
                self.all_pgs.add(get_group_name(start))

    def _calculate_domination_index(
        self, target_nodes: list[fx.Node]
    ) -> dict[fx.Node, int]:
        """
        Calculate the topological index of the earliest target node each node dominates.

        target_nodes are assigned indices based on their topological order (0, 1, 2, ...).
        For each node, returns the minimum index of target nodes it blocks/dominates.
        Returns sys.maxsize if the node doesn't block any target nodes.
        """
        target_node_index: dict[fx.Node, int] = {}
        for node in self.graph.nodes:
            if node in target_nodes:
                target_node_index[node] = len(target_node_index)

        domination_index: dict[fx.Node, int] = {}
        for node in reversed(self.graph.nodes):
            if node in target_node_index:
                domination_index[node] = target_node_index[node]
            else:
````
- **EN**: Introduces function `_calculate_domination_index`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `size_bytes`, `estimated_time_ms`, `exposed_time_ms`, `target_node_index`, `domination_index`, and `else`.
- **CN**: 这里定义了函数`_calculate_domination_index`。包含分支、循环或上下文管理等控制流。初始化或更新了 `size_bytes`、`estimated_time_ms`、`exposed_time_ms`、`target_node_index`、`domination_index`、`else` 等值。

### Lines 701-728 / 第 701-728 行
````python
                domination_index[node] = min(
                    (domination_index[succ] for succ in node.users), default=sys.maxsize
                )

        return domination_index

    def _align_compute_nodes_runtime_estimations_across_all_distributed_ranks(
        self,
    ) -> None:
        """Align runtime estimations across ranks (compute + collectives)."""
        log.info(
            "Overlap scheduling: Aligning runtime estimations across all distributed ranks"
        )

        # Benchmark compute nodes
        runtime_estimations_keys: list[str | None] = []
        runtime_estimations: list[float] = []
        compute_key_count = 0

        # Also collect analytical estimations for logging
        runtime_estimations_analytical: list[float] = []

        for n in self.compute_nodes:
            # Compute analytical estimation using roofline model
            val_analytical = estimate_roofline_runtime_ms(n)
            runtime_estimations_analytical.append(val_analytical)

            if self.compute_estimator == "benchmark":
````
- **EN**: Introduces function `_align_compute_nodes_runtime_estimations_across_all_distributed_ranks`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_align_compute_nodes_runtime_estimations_across_all_distributed_ranks`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 729-756 / 第 729-756 行
````python
                val, key = benchmark_node_with_cache_key(
                    n, self.custom_runtime_estimation
                )
            else:
                # Use analytical estimation
                val, key = val_analytical, None

            runtime_estimations.append(val)
            runtime_estimations_keys.append(key)
            compute_key_count += 1

        # Log compute estimations
        from torch._inductor.fx_passes.node_runtime_estimation import (
            _log_compute_estimations,
        )

        _log_compute_estimations(
            self.compute_nodes,
            runtime_estimations,
            runtime_estimations_analytical,
        )

        # Benchmark collectives if enabled (only CUDA events - others are deterministic)
        # Skip if custom estimation is provided for collectives
        collective_nodes: list[fx.Node] = []
        benchmarked_collective_nodes: list[
            fx.Node
        ] = []  # Track which were actually benchmarked
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.node_runtime_estimation` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里导入了 `torch._inductor.fx_passes.node_runtime_estimation` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。

### Lines 757-784 / 第 757-784 行
````python
        if self.collective_estimator == "benchmark":
            from torch._inductor.fx_passes.node_runtime_estimation import (
                benchmark_collective_with_cuda_events,
            )

            collective_nodes = [
                info.start_node for info in self.collective_info.values()
            ]

            # Benchmark CUDA events (non-deterministic, needs alignment)
            # Skip collectives with custom estimation
            for n in collective_nodes:
                if (
                    get_custom_estimation(n, self.custom_runtime_estimation, None)
                    is not None
                ):
                    continue

                # Benchmark actual size
                cuda_val, cuda_key = benchmark_collective_with_cuda_events(n, nruns=5)
                if cuda_val is not None:
                    runtime_estimations.append(cuda_val)
                    runtime_estimations_keys.append(cuda_key)
                    benchmarked_collective_nodes.append(n)

        # When both estimators are analytical, estimates are deterministic across ranks
        # (same shapes = same estimates), so skip the all_gather to avoid sync.
        import torch.distributed as dist
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.node_runtime_estimation`, and `torch.distributed` for the logic in this range. Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.fx_passes.node_runtime_estimation`、`torch.distributed` 等依赖，为后续逻辑提供基础能力。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。

### Lines 785-812 / 第 785-812 行
````python

        world_size = dist.get_world_size()

        if (
            self.compute_estimator == "analytical"
            and self.collective_estimator == "analytical"
        ):
            median_runtime_estimations = runtime_estimations
        else:
            # Single all_gather and compute medians
            from torch._subclasses.fake_tensor import unset_fake_temporarily
            from torch.distributed.distributed_c10d import _get_default_group

            pg = _get_default_group()
            with unset_fake_temporarily():
                gathered_runtime_estimations: list[list[float]] = [
                    [] for _ in range(world_size)
                ]
                dist.all_gather_object(
                    gathered_runtime_estimations, runtime_estimations, pg
                )
                median_runtime_estimations = torch.median(
                    torch.tensor(gathered_runtime_estimations), dim=0
                ).values.tolist()

        # Cache medians
        collective_keys = []
        collective_medians = []
````
- **EN**: Imports dependencies such as `torch._subclasses.fake_tensor`, and `torch.distributed.distributed_c10d` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `world_size`, `median_runtime_estimations`, `else`, `pg`, `gathered_runtime_estimations`, `collective_keys`, and `...+1`.
- **CN**: 这里导入了 `torch._subclasses.fake_tensor`、`torch.distributed.distributed_c10d` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `world_size`、`median_runtime_estimations`、`else`、`pg`、`gathered_runtime_estimations`、`collective_keys`、`另有1项` 等值。

### Lines 813-840 / 第 813-840 行
````python
        for idx, (key, median_runtime_estimation) in enumerate(
            zip(runtime_estimations_keys, median_runtime_estimations)
        ):
            if key is None:
                continue
            if idx < compute_key_count:
                # Compute node
                self.node_estimations[self.compute_nodes[idx]] = (
                    median_runtime_estimation
                )
                set_cached_node_time(key, median_runtime_estimation)
            else:
                # Collective CUDA event benchmark
                from torch._inductor.fx_passes.node_runtime_estimation import (
                    set_cached_runtime,
                )

                set_cached_runtime(key, median_runtime_estimation)

                # Update CollectiveInfo with aligned benchmark
                coll_idx = idx - compute_key_count
                coll_node = benchmarked_collective_nodes[coll_idx]
                info = self.collective_info[coll_node]
                info.estimated_time_ms = median_runtime_estimation
                info.exposed_time_ms = median_runtime_estimation
                self.node_estimations[coll_node] = median_runtime_estimation

                collective_keys.append(key)
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.node_runtime_estimation` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里导入了 `torch._inductor.fx_passes.node_runtime_estimation` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。

### Lines 841-868 / 第 841-868 行
````python
                collective_medians.append(median_runtime_estimation)

        # Log benchmarks with analytical comparisons
        if collective_keys:
            from torch._inductor.fx_passes.node_runtime_estimation import (
                _log_collective_benchmarks,
            )

            _log_collective_benchmarks(
                benchmarked_collective_nodes,
                collective_keys,
                collective_medians,
                world_size,
                "fx_collectives_node_runtime_estimation",
            )
        else:
            # No benchmarking - log analytical estimations for all collectives
            from torch._inductor.fx_passes.node_runtime_estimation import (
                _log_collective_benchmarks,
            )

            all_collective_nodes = [
                info.start_node for info in self.collective_info.values()
            ]
            if all_collective_nodes:
                _log_collective_benchmarks(
                    all_collective_nodes,
                    artifact_name="fx_collectives_analytical_estimation",
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.node_runtime_estimation` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `all_collective_nodes`, and `artifact_name`. This range continues the implementation of function `OverlapScheduler._align_compute_nodes_runtime_estimations_across_all_distributed_ranks`.
- **CN**: 这里导入了 `torch._inductor.fx_passes.node_runtime_estimation` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`all_collective_nodes`、`artifact_name` 等值。这一段延续了函数`OverlapScheduler._align_compute_nodes_runtime_estimations_across_all_distributed_ranks` 的具体实现。

### Lines 869-896 / 第 869-896 行
````python
                )

        log.info("Overlap scheduling: Runtime estimations aligned")

    def _get_next_nodes(self) -> list[fx.Node]:
        """
        Get next node(s) to schedule.

        When max_off_bucket_bytes is set, off-path collectives of the same type
        (same bucket_key) are batched together to enable bucketing them in
        overlap_preserving_bucketer. Bucket size is limited by max_off_bucket_bytes.
        """
        if self.off_path_ready:
            _, node = self.off_path_ready[0]

            should_schedule = False
            if not self.on_path_ready or node in self.scheduled:
                should_schedule = True
            elif _schedulable_wait_node(node):
                # Defer exposed waits until hidden or over memory budget
                info = self.collective_info[self.wait_to_start[node]]
                over_budget = (
                    self.memory_tracker.current_memory_bytes
                    > self.allowed_peak_memory_bytes
                )
                should_schedule = not info.is_exposed or over_budget
            elif self.max_off_bucket_bytes is not None and node in self.collective_info:
                # Batch off-path collectives: schedule when bucket threshold is reached
````
- **EN**: Introduces function `_get_next_nodes`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `should_schedule`, `info`, and `over_budget`.
- **CN**: 这里定义了函数`_get_next_nodes`。包含分支、循环或上下文管理等控制流。初始化或更新了 `should_schedule`、`info`、`over_budget` 等值。

### Lines 897-924 / 第 897-924 行
````python
                bucket_key = get_full_bucket_key(node, self.bucket_mode)
                bucket_size = self.off_path_ready_potential_buckets[bucket_key]
                should_schedule = bucket_size >= self.max_off_bucket_bytes
            elif self.dominates_reduce_scatter(node):
                # Only schedule off-path nodes that dominate reduce_scatters after original position
                should_schedule = self.node_idx[node] <= self.last_on_path_node_idx

            if should_schedule:
                heapq.heappop(self.off_path_ready)

                # If batching enabled and this is a collective, gather same-type collectives
                if (
                    self.max_off_bucket_bytes is not None
                    and node in self.collective_info
                ):
                    node_key = get_full_bucket_key(node, self.bucket_mode)
                    if node_key is not None:
                        same_type_nodes = [node]
                        total_bytes = self.collective_info[node].size_bytes
                        indices_to_remove = []

                        # Scan the off_path_ready queue for same-key collectives
                        for i, (_, candidate) in enumerate(self.off_path_ready):
                            if candidate in self.scheduled:
                                continue
                            if candidate not in self.collective_info:
                                continue
                            candidate_key = get_full_bucket_key(
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bucket_key`, `bucket_size`, `should_schedule`, `node_key`, `same_type_nodes`, `total_bytes`, and `...+2`. This range continues the implementation of function `OverlapScheduler._get_next_nodes`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `bucket_key`、`bucket_size`、`should_schedule`、`node_key`、`same_type_nodes`、`total_bytes`、`另有2项` 等值。这一段延续了函数`OverlapScheduler._get_next_nodes` 的具体实现。

### Lines 925-952 / 第 925-952 行
````python
                                candidate, self.bucket_mode
                            )
                            if candidate_key == node_key:
                                candidate_bytes = self.collective_info[
                                    candidate
                                ].size_bytes
                                # Check bucket size limit before adding
                                if (
                                    total_bytes + candidate_bytes
                                    > self.max_off_bucket_bytes
                                ):
                                    continue  # Skip but keep looking for smaller ones
                                same_type_nodes.append(candidate)
                                total_bytes += candidate_bytes
                                indices_to_remove.append(i)

                        # Remove collected nodes from heap (reverse order to preserve indices)
                        for i in reversed(indices_to_remove):
                            self.off_path_ready.pop(i)
                        if indices_to_remove:
                            heapq.heapify(self.off_path_ready)

                        return same_type_nodes

                return [node]

        return [heapq.heappop(self.on_path_ready)[1]]

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `candidate_bytes`. This range continues the implementation of function `OverlapScheduler._get_next_nodes`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `candidate_bytes` 等值。这一段延续了函数`OverlapScheduler._get_next_nodes` 的具体实现。

### Lines 953-980 / 第 953-980 行
````python
    def run(self) -> torch.fx.GraphModule:
        """Run the scheduling algorithm."""
        # All ranks must make identical decisions on overlap reordering,
        # Thus we must have identical runtime estimations across ranks.
        # For now we do benchmarking only for compute nodes.
        self._align_compute_nodes_runtime_estimations_across_all_distributed_ranks()

        while self.on_path_ready or self.off_path_ready:
            if self._should_force_wait_for_memory():
                self._force_oldest_wait()
                continue

            nodes = self._get_next_nodes()

            for node in nodes:
                # we don't always remove nodes from the heap when we schedule them
                if node in self.scheduled:
                    continue

                if node.op == "placeholder":
                    self._schedule(node)
                elif node in self.collective_info:
                    self._handle_collective_start(node)
                elif _schedulable_wait_node(node):
                    self._handle_wait(node)
                else:
                    self._handle_compute_or_other(node)

````
- **EN**: Introduces function `run`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `nodes`, and `else`.
- **CN**: 这里定义了函数`run`。包含分支、循环或上下文管理等控制流。初始化或更新了 `nodes`、`else` 等值。

### Lines 981-1008 / 第 981-1008 行
````python
                # Track progress for off-path scheduling - only for nodes from main queue
                if not self.off_compute_path(node):
                    self.last_on_path_node_idx = max(
                        self.last_on_path_node_idx, self.node_idx[node]
                    )
                else:
                    # Decrement off-path bucket bytes when scheduling
                    if node in self.collective_info:
                        bucket_key = get_full_bucket_key(node, self.bucket_mode)
                        node_bytes = self.collective_info[node].size_bytes
                        self.off_path_ready_potential_buckets[bucket_key] -= node_bytes

        self._reorder_graph()

        # Finalize: bucket collectives (if enabled), inline fusions, apply deps
        from torch._inductor.fx_passes.overlap_preserving_bucketer import (
            finalize_overlap_scheduling,
        )

        finalize_overlap_scheduling(
            gm=self.gm,
            collective_info=self.collective_info,
            scheduled=self.scheduled,
            collective_bucketing=self.collective_bucketing,
            insert_overlap_deps=self.insert_overlap_deps,
            max_bucket_memory_gb=2.0,
            max_coll_distance=self.max_node_distance,
            region_of=self.region_of,
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.overlap_preserving_bucketer` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `bucket_key`, `node_bytes`, `gm`, `collective_info`, `scheduled`, and `...+5`. This range continues the implementation of function `OverlapScheduler.run`.
- **CN**: 这里导入了 `torch._inductor.fx_passes.overlap_preserving_bucketer` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`bucket_key`、`node_bytes`、`gm`、`collective_info`、`scheduled`、`另有5项` 等值。这一段延续了函数`OverlapScheduler.run` 的具体实现。

### Lines 1009-1036 / 第 1009-1036 行
````python
            bucket_exposed_first=self.bucket_exposed_first,
            bucket_only_internode_comms=self.bucket_only_internode_comms,
            bucket_mode=self.bucket_mode,
        )

        if self.log_final_collectives_estimations:
            from torch._inductor.fx_passes.node_runtime_estimation import (
                _log_graph_collective_benchmarks,
            )

            _log_graph_collective_benchmarks(
                self.gm, "fx_collectives_estimations_after_overlap_bucketing"
            )

        return self.gm

    def _reduce_exposed_time_of_in_flight_collectives(
        self,
        node: fx.Node,
        available_compute: float,
        exclude_pg: str | None = None,
    ) -> dict[str, float]:
        """
        Reduce exposed time of in-flight collectives using available compute time.

        Collectives on different process groups can overlap simultaneously with the same
        compute, so we track remaining time separately per PG.
        """
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.node_runtime_estimation` for the logic in this range. Introduces function `_reduce_exposed_time_of_in_flight_collectives`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.fx_passes.node_runtime_estimation` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_reduce_exposed_time_of_in_flight_collectives`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1037-1064 / 第 1037-1064 行
````python
        # Initialize all PGs with full available compute (except excluded)
        remaining_time_per_pg: dict[str, float] = {
            pg: available_compute for pg in self.all_pgs if pg != exclude_pg
        }

        for start_node, info in self.in_flight.items():
            if info.exposed_time_ms == 0:
                continue

            pg_name = get_group_name(start_node)
            if pg_name == exclude_pg:
                continue

            pg_remaining = remaining_time_per_pg[pg_name]
            if pg_remaining <= 0:
                continue

            overlap_amount = min(info.exposed_time_ms, pg_remaining)
            info.exposed_time_ms -= overlap_amount
            remaining_time_per_pg[pg_name] -= overlap_amount
            info.hiding_nodes.add(node)

        return remaining_time_per_pg

    def _handle_compute_or_other(self, node: fx.Node) -> None:
        """Handle scheduling compute or other nodes and attempt to overlap with collectives."""
        runtime_estimate = self.node_estimations.get(node)

````
- **EN**: Introduces function `_handle_compute_or_other`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `remaining_time_per_pg`, `pg`, `pg_name`, `pg_remaining`, `overlap_amount`, and `runtime_estimate`.
- **CN**: 这里定义了函数`_handle_compute_or_other`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `remaining_time_per_pg`、`pg`、`pg_name`、`pg_remaining`、`overlap_amount`、`runtime_estimate` 等值。

### Lines 1065-1092 / 第 1065-1092 行
````python
        # TODO: we could consider skipping overlapping for overlapable, unary chains to collectives.
        # using these nodes for overlap prevents bucketing. potentially if chain time < latency
        if runtime_estimate is None:
            assert not is_compute_node(node), "should have estimate for compute nodes"
            self._schedule(node)
            return

        available_compute = runtime_estimate * self.compute_overlap_multipler

        # First, reduce exposed time of in-flight collectives (per PG)
        remaining_time_per_pg = self._reduce_exposed_time_of_in_flight_collectives(
            node, available_compute
        )
        # Then, schedule new collectives for overlap
        self._schedule_collectives_for_overlap(node, remaining_time_per_pg)
        self._schedule(node)

        if is_compute_node(node):
            self.current_compute_index += 1

    def _schedule(self, node: fx.Node) -> None:
        """Schedule a node."""
        assert node not in self.scheduled
        assert all(n in self.scheduled for n in node.all_input_nodes)
        self.scheduled.add(node)
        self.memory_tracker.schedule_node(node)

        log.debug(
````
- **EN**: Introduces function `_schedule`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `available_compute`, and `remaining_time_per_pg`.
- **CN**: 这里定义了函数`_schedule`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `available_compute`、`remaining_time_per_pg` 等值。

### Lines 1093-1120 / 第 1093-1120 行
````python
            "Scheduled node %s: current_memory=%d bytes, total_scheduled=%d",
            node.name,
            self.memory_tracker.get_current_memory_bytes(),
            len(self.scheduled),
        )

        for user in node.users:
            self.in_degree[user] -= 1
            if self.in_degree[user] == 0:
                self._add_to_ready_queue(user)

    def _compute_on_path_score(self, node: fx.Node) -> object:
        """Compute priority score for on-path nodes (domination-based)."""
        if _schedulable_wait_node(node):
            info = self.collective_info[self.wait_to_start[node]]
            # defer waits locally if they are exposed.
            compute_local_priority = int(info.is_exposed)
        else:
            # if we're scheduling this collective via its queue, then it was not
            # pre-fetched. we might as well maximize overlap for the
            # local, non-mm nodes prior to the next compute node.
            if self.in_overlappable_collective_unary_chain(node):
                compute_local_priority = -1
            else:
                compute_local_priority = 0

        return (
            self.compute_index_domination[node],  # what index compute it blocks
````
- **EN**: Introduces function `_compute_on_path_score`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `info`, `compute_local_priority`, and `else`.
- **CN**: 这里定义了函数`_compute_on_path_score`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `info`、`compute_local_priority`、`else` 等值。

### Lines 1121-1148 / 第 1121-1148 行
````python
            compute_local_priority,  # collective_start=-1, wait=1, or neither=0
            self.node_idx[node],  # Original order for stability
        )

    def _compute_off_path_score(self, node: fx.Node) -> object:
        """
        Off-path priority scoring.

        Nodes that dominate reduce_scatters are prioritized (lower score = higher priority)
        to ensure they get scheduled eagerly for potential overlap.
        """
        dominates_rs = 0 if self.dominates_reduce_scatter(node) else 1
        return (dominates_rs, self.node_idx[node])

    @staticmethod
    def is_cheap_fn(node: fx.Node) -> bool:
        return getattr(node.target, "is_view", False) or torch.Tag.pointwise in getattr(
            node.target, "tags", ()
        )

    def in_overlappable_collective_unary_chain(self, curr: fx.Node) -> bool:
        while True:
            if len(curr.users) != 1:
                return False

            user = next(iter(curr.users))
            if len(user.all_input_nodes) != 1:
                return False
````
- **EN**: Introduces function `_compute_off_path_score`, function `is_cheap_fn`, function `in_overlappable_collective_unary_chain`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_compute_off_path_score`、函数`is_cheap_fn`、函数`in_overlappable_collective_unary_chain`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1149-1176 / 第 1149-1176 行
````python

            if user in self.unscheduled_collectives:
                return True

            if not self.is_cheap_fn(user):
                return False

            curr = user

        return False

    def _should_force_wait_for_memory(self) -> bool:
        """Check if we need to force a wait due to memory pressure"""
        if not self.in_flight:
            return False

        return self.in_flight_bytes >= self.max_in_flight_bytes

    def _force_oldest_wait(self) -> None:
        """Schedule the oldest in flight wait"""
        self._handle_wait(self._get_oldest_wait())

    def _handle_collective_start(self, node: fx.Node) -> None:
        """Handle scheduling a collective start."""
        info = self.collective_info[node]

        if self.should_assume_bucketed(node):
            latency = estimate_collective_time(
````
- **EN**: Introduces function `_should_force_wait_for_memory`, function `_force_oldest_wait`, function `_handle_collective_start`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `curr`, `info`, and `latency`.
- **CN**: 这里定义了函数`_should_force_wait_for_memory`、函数`_force_oldest_wait`、函数`_handle_collective_start`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `curr`、`info`、`latency` 等值。

### Lines 1177-1204 / 第 1177-1204 行
````python
                node, 0, custom_runtime_estimation=self.custom_runtime_estimation
            )
            assert latency <= info.exposed_time_ms
            info.exposed_time_ms = info.exposed_time_ms - latency

        self.in_flight[node] = info
        self.in_flight_bytes += info.size_bytes
        self.unscheduled_collectives.discard(node)
        self._schedule(node)

    def _handle_wait(self, node: fx.Node) -> None:
        """Handle scheduling a wait."""
        assert node in self.wait_to_start
        coll_start = self.wait_to_start[node]
        # For coalesced collectives, multiple waits share the same start node.
        # The first wait completes the collective; subsequent waits just schedule.
        if coll_start not in self.in_flight:
            self._schedule(node)
            return

        # Scheduling a wait of a collective also forces the wait
        # of every node enqueued prior to the collective on the
        # same process group
        group_name = get_group_name(coll_start)
        to_schedule: list[fx.Node] = []
        for in_flight_coll in self.in_flight:
            if in_flight_coll == coll_start:
                break
````
- **EN**: Introduces function `_handle_wait`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `coll_start`, `group_name`, and `to_schedule`.
- **CN**: 这里定义了函数`_handle_wait`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `coll_start`、`group_name`、`to_schedule` 等值。

### Lines 1205-1232 / 第 1205-1232 行
````python
            if get_group_name(in_flight_coll) == group_name:
                to_schedule.append(in_flight_coll)

        for coll_to_schedule in to_schedule:
            self._handle_wait(self.collective_info[coll_to_schedule].wait_node)

        # If we are waiting on an exposed collective, use this time to
        # overlap on other PGs.
        info = self.collective_info[coll_start]
        if info.exposed_time_ms > 0:
            exposed_time = info.exposed_time_ms
            exclude_pg = group_name

            remaining_time_per_pg = self._reduce_exposed_time_of_in_flight_collectives(
                node, exposed_time, exclude_pg=exclude_pg
            )
            self._schedule_collectives_for_overlap(
                node, remaining_time_per_pg, exclude_pg=exclude_pg
            )

        self.in_flight_bytes -= self.in_flight[coll_start].size_bytes
        del self.in_flight[coll_start]
        self._schedule(node)

    def _schedule_collectives_for_overlap(
        self,
        overlap_node: fx.Node,
        remaining_time_per_pg: dict[str, float],
````
- **EN**: Introduces function `_schedule_collectives_for_overlap`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `info`, `exposed_time`, `exclude_pg`, `remaining_time_per_pg`, and `overlap_node`.
- **CN**: 这里定义了函数`_schedule_collectives_for_overlap`。包含分支、循环或上下文管理等控制流。初始化或更新了 `info`、`exposed_time`、`exclude_pg`、`remaining_time_per_pg`、`overlap_node` 等值。

### Lines 1233-1260 / 第 1233-1260 行
````python
        exclude_pg: str | None = None,
    ) -> None:
        """Opportunistically schedule collectives that can be hidden by available overlap time."""
        if not remaining_time_per_pg or all(
            t <= 0 for t in remaining_time_per_pg.values()
        ):
            return

        overlap_node_ancestors = self.node_ancestors[overlap_node]

        # Compile candidates - limit by distance to bound compile time
        candidates = []
        for i, collective in enumerate(self.unscheduled_collectives):
            if i > self.max_node_distance:
                break

            pg_name = get_group_name(collective)
            if pg_name == exclude_pg:
                continue

            if (
                not self.off_compute_path(collective)
                and self.compute_index_domination[collective]
                - self.current_compute_index
                > self.max_compute_pre_fetch
            ):
                continue

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `exclude_pg`, `overlap_node_ancestors`, `candidates`, and `pg_name`. This range continues the implementation of function `OverlapScheduler._schedule_collectives_for_overlap`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `exclude_pg`、`overlap_node_ancestors`、`candidates`、`pg_name` 等值。这一段延续了函数`OverlapScheduler._schedule_collectives_for_overlap` 的具体实现。

### Lines 1261-1288 / 第 1261-1288 行
````python
            candidates.append(collective)

        def get_priority(n: fx.Node) -> int:
            dominates_next_compute = (
                self.compute_index_domination[n] == self.current_compute_index + 1
            )
            if dominates_next_compute:
                return 0  # Dominates next compute layer - most urgent
            elif self.off_compute_path(n) and self.dominates_reduce_scatter(n):
                return 1  # Off-path but blocks reduce_scatter
            elif not self.off_compute_path(n):
                return 2  # On-path but not immediate
            else:
                return 3  # Off-path, doesn't block reduce_scatter

        candidates.sort(
            key=lambda n: (
                get_priority(n),
                self.compute_index_domination[n],
                self.node_idx[n],
            ),
        )

        if self.prioritize_bucketing_during_scheduling:
            # group candidates by bucket key first so same-bucket
            # collectives are scheduled together, maximizing bucketing opportunities
            bucket_groups: dict[object, list[fx.Node]] = defaultdict(list)
            for coll in candidates:
````
- **EN**: Introduces function `get_priority`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dominates_next_compute`, `else`, `key`, and `bucket_groups`.
- **CN**: 这里定义了函数`get_priority`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dominates_next_compute`、`else`、`key`、`bucket_groups` 等值。

### Lines 1289-1316 / 第 1289-1316 行
````python
                key = get_full_bucket_key(coll, self.bucket_mode)
                bucket_groups[key].append(coll)

            # Sort bucket groups by minimum domination index, larger groups first as tiebreaker
            sorted_bucket_keys = sorted(
                bucket_groups.keys(),
                key=lambda k: (
                    min(self.compute_index_domination[c] for c in bucket_groups[k]),
                    -len(bucket_groups[k]),
                ),
            )

            # Flatten back to ordered candidate list
            candidates = []
            for b_key in sorted_bucket_keys:
                group = bucket_groups[b_key]
                group.sort(
                    key=lambda n: (self.compute_index_domination[n], self.node_idx[n])
                )
                candidates.extend(group)

        for collective in candidates:
            pg_name = get_group_name(collective)
            pg_available_time = remaining_time_per_pg[pg_name]

            if pg_available_time <= 0:
                continue

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `key`, `sorted_bucket_keys`, `candidates`, `group`, `pg_name`, and `pg_available_time`. This range continues the implementation of function `OverlapScheduler._schedule_collectives_for_overlap`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `key`、`sorted_bucket_keys`、`candidates`、`group`、`pg_name`、`pg_available_time` 等值。这一段延续了函数`OverlapScheduler._schedule_collectives_for_overlap` 的具体实现。

### Lines 1317-1344 / 第 1317-1344 行
````python
            why = WhyNoOverlap(overlap_node, collective)
            info = self.collective_info[collective]

            if (
                collective in overlap_node_ancestors
                or overlap_node in self.node_ancestors[collective]
            ):
                why("dependency conflict")
                continue

            # Check if prefetching would exceed memory budget
            if self._prefetch_would_exceed_memory_budget(collective):
                why("prefetch would exceed memory budget")
                continue

            # Try to free memory by forcing hidden waits
            while (
                self.in_flight
                and (self.max_in_flight_bytes - self.in_flight_bytes) < info.size_bytes
                and self._wait_is_hidden(self._get_oldest_wait(), overlap_node)
            ):
                self._force_oldest_wait()

            if (self.max_in_flight_bytes - self.in_flight_bytes) < info.size_bytes:
                why("in-flight memory limit")
                continue

            # Check if we can reach this collective without scheduling compute, other collectives, or waits
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `why`, and `info`. This range continues the implementation of function `OverlapScheduler._schedule_collectives_for_overlap`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `why`、`info` 等值。这一段延续了函数`OverlapScheduler._schedule_collectives_for_overlap` 的具体实现。

### Lines 1345-1372 / 第 1345-1372 行
````python
            path = self._find_schedulable_path(collective, overlap_node, why)
            if path is None:
                continue

            log.debug(
                "Overlapping collective %s with node %s: coll_domination=%d, current_depth=%d",
                collective.name,
                overlap_node.name,
                self.compute_index_domination[collective],
                self.current_compute_index,
            )

            # TODO: We previously tracked path compute time and added it back to available
            # overlap time. With per-PG tracking this is complex: if there were in-flight
            # collectives on one PG but not another, we can't add path time back to the PG
            # that wasn't in-flight

            # Schedule path and collective
            self._schedule_path_to_collective(path, overlap_node)
            self._handle_collective_start(collective)
            self._update_cumulative_prefetch_memory(collective, info)

            # Update exposed time for this collective
            overlap_amount = min(pg_available_time, info.exposed_time_ms)
            info.exposed_time_ms -= overlap_amount
            info.hiding_nodes.add(overlap_node)

            # Update available time for this PG
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `path`, and `overlap_amount`. This range continues the implementation of function `OverlapScheduler._schedule_collectives_for_overlap`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `path`、`overlap_amount` 等值。这一段延续了函数`OverlapScheduler._schedule_collectives_for_overlap` 的具体实现。

### Lines 1373-1400 / 第 1373-1400 行
````python
            remaining_time_per_pg[pg_name] -= overlap_amount

            if sum(remaining_time_per_pg.values()) == 0:
                break

        if remaining_time_per_pg:
            self.wasted_compute += min(remaining_time_per_pg.values())

    def _find_schedulable_path(
        self, target: fx.Node, curr_overlap_node: fx.Node | None, why: WhyNoOverlap
    ) -> OrderedSet[fx.Node] | None:
        """Find path to target by collecting unscheduled dependencies."""
        # Get unscheduled ancestors
        unscheduled_ancestors = self.node_ancestors[target] - self.scheduled

        # only schedule non distributed, non compute nodes
        for node in unscheduled_ancestors:
            if is_compute_node(node):
                why("path blocked by compute node %s", node.name)
                return None

            if node in self.unscheduled_collectives:
                why("path blocked by unscheduled collective %s", node.name)
                return None

            # if we schedule a wait tensor whose start collective is hidden by the
            # current compute node we are scheduling, then we are effectively exposing it.
            # similarly, dont schedule a wait of a collective that could be otherwise hidden,
````
- **EN**: Introduces function `_find_schedulable_path`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `unscheduled_ancestors`.
- **CN**: 这里定义了函数`_find_schedulable_path`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `unscheduled_ancestors` 等值。

### Lines 1401-1428 / 第 1401-1428 行
````python
            # thus forcing it to be exposed.
            # however, if it is already hidden it's fine to schedule it
            if _schedulable_wait_node(node):
                info = self.collective_info[self.wait_to_start[node]]
                if (not info.is_exposed) and (
                    curr_overlap_node not in info.hiding_nodes
                ):
                    continue

                why(
                    "path blocked by wait node %s (exposed=%s, hidden_by_curr_overlap=%s)",
                    node.name,
                    info.is_exposed,
                    curr_overlap_node in info.hiding_nodes,
                )
                return None

            # Skip c10 ops and dtensor shard ops - they should be scheduled via main loop
            target_str = str(node.target)
            if "c10" in target_str or "_dtensor" in target_str:
                log.debug(
                    "Skipping c10/dtensor op %s in path to collective",
                    node.name,
                )
                return None

        return unscheduled_ancestors

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `info`, and `target_str`. This range continues the implementation of function `OverlapScheduler._find_schedulable_path`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `info`、`target_str` 等值。这一段延续了函数`OverlapScheduler._find_schedulable_path` 的具体实现。

### Lines 1429-1456 / 第 1429-1456 行
````python
    def should_assume_bucketed(self, node: fx.Node) -> bool:
        """
        Check if there's an in-flight collective that can be bucketed with the given node. If so, assume they will bucket.
        This is a optimistic heuristic to account for latency reduction with bucketing. The two nodes may not get bucketed.
        """
        if not torch._inductor.config.test_configs.assume_bucketing_reduces_latency:
            return False

        key = bucket_key(node, mode=self.bucket_mode)
        if key is None:
            return False

        for in_flight_coll in self.in_flight:
            if bucket_key(in_flight_coll, mode=self.bucket_mode) == key:
                return True

        return False

    def _get_oldest_wait(self) -> fx.Node:
        oldest_start = next(iter(self.in_flight))
        return self.collective_info[oldest_start].wait_node

    def _wait_is_hidden(
        self, wait_node: fx.Node, overlap_node: fx.Node | None = None
    ) -> bool:
        assert is_wait_tensor(wait_node)
        info = self.collective_info[self.wait_to_start[wait_node]]
        return not info.is_exposed and overlap_node not in info.hiding_nodes
````
- **EN**: Introduces function `should_assume_bucketed`, function `_get_oldest_wait`, function `_wait_is_hidden`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `key`, `oldest_start`, and `info`.
- **CN**: 这里定义了函数`should_assume_bucketed`、函数`_get_oldest_wait`、函数`_wait_is_hidden`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `key`、`oldest_start`、`info` 等值。

### Lines 1457-1484 / 第 1457-1484 行
````python

    def _schedule_path_to_collective(
        self, path: OrderedSet[fx.Node], curr_overlap_node: fx.Node
    ) -> None:
        """Schedule all nodes needed to reach a collective."""

        assert all(n not in self.scheduled for n in path)
        for node in sorted(path, key=lambda n: self.node_idx[n]):
            assert not (is_compute_node(node) or node in self.unscheduled_collectives)
            if _schedulable_wait_node(node):
                # When we schedule wait tensors, we also force realization of all
                # collectives enqueued prior to their corresponding collective.
                # It's possible the scheduling of one wait tensor here has forced
                # another in the path. If so, skip scheduling it.
                if node in self.scheduled:
                    continue

                info = self.collective_info[self.wait_to_start[node]]
                assert curr_overlap_node not in info.hiding_nodes
                self._handle_wait(node)
                continue

            self._schedule(node)

    def reorder_graph(self) -> None:
        output_node = self.graph.output_node()
        for node in self.scheduled:
            if node.op == "placeholder":
````
- **EN**: Introduces function `_schedule_path_to_collective`, function `reorder_graph`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `info`, and `output_node`.
- **CN**: 这里定义了函数`_schedule_path_to_collective`、函数`reorder_graph`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `info`、`output_node` 等值。

### Lines 1485-1512 / 第 1485-1512 行
````python
                continue
            output_node.prepend(node)
        self.graph.lint()

    def _reorder_graph(self) -> None:
        """Reorder graph based on schedule."""
        exposed = [
            c
            for c in self.collective_info.values()
            if c.exposed_time_ms == c.estimated_time_ms
        ]

        potentially_hidden_collectives = self.compute_potential_hidden_collectives()
        bad_exposed = [
            c for c in exposed if c.start_node in potentially_hidden_collectives
        ]

        # Compute total exposed and potential exposed time
        total_exposed = sum(c.exposed_time_ms for c in self.collective_info.values())
        hideable_exposed_ms = sum(
            self.collective_info[c].exposed_time_ms
            for c in potentially_hidden_collectives
        )
        total_potential_exposed = sum(
            c.estimated_time_ms for c in self.collective_info.values()
        )

        counters["inductor"]["overlap_scheduling_exposed"] += len(exposed)
````
- **EN**: Introduces function `_reorder_graph`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `exposed`, `potentially_hidden_collectives`, `bad_exposed`, `total_exposed`, `hideable_exposed_ms`, and `total_potential_exposed`.
- **CN**: 这里定义了函数`_reorder_graph`。包含分支、循环或上下文管理等控制流。初始化或更新了 `exposed`、`potentially_hidden_collectives`、`bad_exposed`、`total_exposed`、`hideable_exposed_ms`、`total_potential_exposed` 等值。

### Lines 1513-1540 / 第 1513-1540 行
````python
        counters["inductor"]["overlap_scheduling_bad_exposed"] += len(bad_exposed)
        counters["inductor"]["overlap_scheduling_potentially_hidden"] += len(
            potentially_hidden_collectives
        )
        counters["inductor"]["overlap_original_mem"] = self.original_peak_memory
        counters["inductor"]["rescheduled_mem"] = self.memory_tracker.peak_memory

        log.info(
            "Overlap scheduling results: exposed=%d, bad_exposed=%d, potentially_hidden=%d, "
            "original_peak_memory=%d bytes, rescheduled_peak_memory=%d bytes, "
            "total_exposed_ms=%.2f, hideable_exposed_ms=%.2f, total_potential_exposed_ms=%.2f, "
            "wasted_compute_ms=%.2f",
            len(exposed),
            len(bad_exposed),
            len(potentially_hidden_collectives),
            self.original_peak_memory,
            self.memory_tracker.peak_memory,
            total_exposed,
            hideable_exposed_ms,
            total_potential_exposed,
            self.wasted_compute,
        )

        self.reorder_graph()

    def _bucket_collectives(self) -> None:
        from torch._inductor.fx_passes.overlap_preserving_bucketer import (
            OverlapPreservingBucketer,
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.overlap_preserving_bucketer` for the logic in this range. Introduces function `_bucket_collectives`.
- **CN**: 这里导入了 `torch._inductor.fx_passes.overlap_preserving_bucketer` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_bucket_collectives`。

### Lines 1541-1568 / 第 1541-1568 行
````python
        )

        bucketer = OverlapPreservingBucketer(
            graph=self.graph,
            collective_info=self.collective_info,
            scheduled=self.scheduled,
            max_bucket_memory_gb=2.0,  # Could make this configurable
            max_coll_distance=self.max_node_distance,
            insert_overlap_deps=self.insert_overlap_deps,
            bucket_mode=self.bucket_mode,
            bucket_exposed_first=self.bucket_exposed_first,
            bucket_only_internode_comms=self.bucket_only_internode_comms,
        )
        bucketer.bucket_collectives()

    def compute_potential_hidden_nodes(
        self, nodes_to_check: Iterable[fx.Node]
    ) -> dict[fx.Node, fx.Node]:
        """
        Returns a dict containing a mapping of nodes which could potentially be hidden to their hiding node
        """

        def could_be_hidden(start: fx.Node) -> fx.Node | None:
            for compute_node in self.compute_nodes:
                if (
                    start not in self.node_ancestors[compute_node]
                    and compute_node not in self.node_ancestors[start]
                ):
````
- **EN**: Introduces function `compute_potential_hidden_nodes`, function `could_be_hidden`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bucketer`, `graph`, `collective_info`, `scheduled`, `max_bucket_memory_gb`, `max_coll_distance`, and `...+4`.
- **CN**: 这里定义了函数`compute_potential_hidden_nodes`、函数`could_be_hidden`。包含分支、循环或上下文管理等控制流。初始化或更新了 `bucketer`、`graph`、`collective_info`、`scheduled`、`max_bucket_memory_gb`、`max_coll_distance`、`另有4项` 等值。

### Lines 1569-1596 / 第 1569-1596 行
````python
                    return compute_node

            return None

        # TODO: We could potentially limit compute nodes per overlap time,
        # today, this is optimistic, and just serves to avoid deferring
        # collectives/waits that have no possible overlap as well as for analysis of how
        # successfully we hid compute
        potentially_hidden = {}
        for node in nodes_to_check:
            if mm := could_be_hidden(node):
                potentially_hidden[node] = mm

        return potentially_hidden

    def compute_potential_hidden_collectives(self) -> dict[fx.Node, fx.Node]:
        """Compute which collective operations could be hidden by compute."""
        return self.compute_potential_hidden_nodes(self.collective_info.keys())

    def compute_potential_hidden_waits(self) -> dict[fx.Node, fx.Node]:
        """Compute which wait operations could be hidden by compte."""
        wait_nodes = [info.wait_node for info in self.collective_info.values()]
        return self.compute_potential_hidden_nodes(wait_nodes)


def gather_node_runtime_estimations(
    gm: torch.fx.GraphModule,
    custom_runtime_estimation: Callable[[fx.Node, int | None], float | None]
````
- **EN**: Introduces function `compute_potential_hidden_collectives`, function `compute_potential_hidden_waits`, function `gather_node_runtime_estimations`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `potentially_hidden`, `wait_nodes`, `gm`, and `custom_runtime_estimation`.
- **CN**: 这里定义了函数`compute_potential_hidden_collectives`、函数`compute_potential_hidden_waits`、函数`gather_node_runtime_estimations`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `potentially_hidden`、`wait_nodes`、`gm`、`custom_runtime_estimation` 等值。

### Lines 1597-1624 / 第 1597-1624 行
````python
    | None = None,
    collective_estimator: Literal["analytical", "benchmark"] = "analytical",
    enable_fusion_regions: bool = False,
    log_estimations: bool = False,
) -> tuple[dict[fx.Node, float], dict[fx.Node, Any]]:
    """Gather initial runtime estimations for all nodes without scheduling.

    Uses analytical models (roofline) for compute nodes — the alignment step
    in OverlapScheduler.run() replaces these with benchmarked + cross-rank-aligned
    values. Collectives use bandwidth formulas or CUDA events depending on
    collective_estimator.

    When enable_fusion_regions is True, builds and collapses fusion regions
    (mutating gm's graph), then includes their costs in the estimations.

    Args:
        collective_estimator: "analytical" uses bandwidth formulas,
            "benchmark" uses CUDA events for collectives.
        log_estimations: When True, log compute and collective estimations
            via trace_structured for tlparse.

    Returns (estimations, fusion_region_of) where estimations maps fx.Node to
    runtime in ms, and fusion_region_of maps call_module nodes to FusionRegion
    objects (empty dict if fusion regions are disabled).
    """
    # Build and collapse fusion regions first (mutates gm)
    fusion_region_of: dict[fx.Node, Any] = {}
    if enable_fusion_regions:
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `collective_estimator`, `enable_fusion_regions`, `log_estimations`, `Args`, and `fusion_region_of`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `collective_estimator`、`enable_fusion_regions`、`log_estimations`、`Args`、`fusion_region_of` 等值。

### Lines 1625-1652 / 第 1625-1652 行
````python
        from torch._inductor.fx_passes.fusion_regions import (
            build_fusion_regions,
            collapse_fusion_regions,
        )

        fusion_region_of = build_fusion_regions(gm)
        if fusion_region_of:
            fusion_region_of = collapse_fusion_regions(gm, fusion_region_of)

    estimations: dict[fx.Node, float] = {}
    nodes = list(gm.graph.nodes)

    # Collectives
    collective_nodes: list[fx.Node] = []
    for node in nodes:
        if _schedulable_wait_node(node):
            start = _get_collective_node_from_wait(node)
            assert start is not None
            if start in estimations:
                continue
            estimations[start] = estimate_collective_time(
                start,
                custom_runtime_estimation=custom_runtime_estimation,
                collective_estimator=collective_estimator,
            )
            collective_nodes.append(start)

    # Compute nodes (matmul, bmm, etc.) — analytical estimates only.
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.fusion_regions` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fusion_region_of`, `estimations`, `nodes`, `collective_nodes`, `start`, `custom_runtime_estimation`, and `...+1`.
- **CN**: 这里导入了 `torch._inductor.fx_passes.fusion_regions` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `fusion_region_of`、`estimations`、`nodes`、`collective_nodes`、`start`、`custom_runtime_estimation`、`另有1项` 等值。

### Lines 1653-1680 / 第 1653-1680 行
````python
    # The alignment step in run() replaces these with benchmarked + aligned values.
    compute_nodes: list[fx.Node] = []
    compute_analytical: list[float] = []

    for node in nodes:
        if is_compute_node(node):
            est = estimate_roofline_runtime_ms(node)
            if custom_runtime_estimation is not None:
                custom_est = custom_runtime_estimation(node, None)
                if custom_est is not None:
                    est = custom_est
            estimations[node] = est
            compute_nodes.append(node)
            compute_analytical.append(est)
        elif node.op == "call_function" and node not in estimations:
            if custom_runtime_estimation is not None:
                est = custom_runtime_estimation(node, None)
                if est is not None:
                    estimations[node] = est
            else:
                est = estimate_roofline_runtime_ms(node)
                if est > 0:
                    estimations[node] = est

    # Fusion region costs (call_module nodes from collapse_fusion_regions)
    for node, region in fusion_region_of.items():
        estimations[node] = region.cost_ms  # pyrefly: ignore[missing-attribute]

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `compute_nodes`, `compute_analytical`, `est`, `custom_est`, and `else`. This range continues the implementation of function `gather_node_runtime_estimations`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `compute_nodes`、`compute_analytical`、`est`、`custom_est`、`else` 等值。这一段延续了函数`gather_node_runtime_estimations` 的具体实现。

### Lines 1681-1708 / 第 1681-1708 行
````python
    # Logging
    if log_estimations and compute_nodes:
        from torch._inductor.fx_passes.node_runtime_estimation import (
            _log_compute_estimations,
        )

        _log_compute_estimations(
            compute_nodes,
            compute_analytical,
            compute_analytical,
        )

    if log_estimations and collective_nodes:
        from torch._inductor.fx_passes.node_runtime_estimation import (
            _log_collective_benchmarks,
        )

        _log_collective_benchmarks(
            collective_nodes,
            artifact_name="fx_collectives_analytical_estimation",
        )

    return estimations, fusion_region_of


def align_estimations_across_ranks(
    estimations: dict[fx.Node, float],
) -> dict[fx.Node, float]:
````
- **EN**: Imports dependencies such as `torch._inductor.fx_passes.node_runtime_estimation` for the logic in this range. Introduces function `align_estimations_across_ranks`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.fx_passes.node_runtime_estimation` 等依赖，为后续逻辑提供基础能力。这里定义了函数`align_estimations_across_ranks`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1709-1736 / 第 1709-1736 行
````python
    """Align runtime estimations across distributed ranks using median.

    All ranks must make identical scheduling decisions, so we gather each
    rank's values and take the median. All nodes in estimations are aligned.

    Returns a new estimations dict with aligned values.
    """
    import torch.distributed as dist
    from torch._subclasses.fake_tensor import unset_fake_temporarily
    from torch.distributed.distributed_c10d import _get_default_group

    nodes = list(estimations.keys())
    if not nodes:
        return {}

    local_values = [estimations[n] for n in nodes]

    world_size = dist.get_world_size()
    pg = _get_default_group()

    with unset_fake_temporarily():
        gathered: list[list[float]] = [[] for _ in range(world_size)]
        dist.all_gather_object(gathered, local_values, pg)
        medians = torch.median(torch.tensor(gathered), dim=0).values.tolist()

    return dict(zip(nodes, medians))


````
- **EN**: Imports dependencies such as `torch.distributed`, `torch._subclasses.fake_tensor`, and `torch.distributed.distributed_c10d` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `nodes`, `local_values`, `world_size`, `pg`, `gathered`, and `medians`.
- **CN**: 这里导入了 `torch.distributed`、`torch._subclasses.fake_tensor`、`torch.distributed.distributed_c10d` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `nodes`、`local_values`、`world_size`、`pg`、`gathered`、`medians` 等值。

### Lines 1737-1764 / 第 1737-1764 行
````python
def schedule_overlap_bucketing(
    gm: torch.fx.GraphModule,
    max_in_flight_gb: float = 5,
    max_compute_pre_fetch: int = 200,
    collective_bucketing: bool = False,
    insert_overlap_deps: bool = False,
    compute_overlap_multipler: float = 1.0,
    max_coll_distance: int = 200,
    custom_runtime_estimation: Callable[[fx.Node, int | None], float | None]
    | None = None,
    collective_estimator: Literal["analytical", "benchmark"] = "analytical",
    compute_estimator: Literal["analytical", "benchmark"] = "benchmark",
    max_memory_increase_gb: float | None = 1.0,
    max_memory_increase_ratio: float | None = 0.05,
    log_final_collectives_estimations: bool = False,
    bucket_exposed_first: bool | None = None,
    enable_fusion_regions: bool = False,
    bucket_only_internode_comms=False,
    prioritize_bucketing_during_scheduling: bool = True,
    max_off_bucket_gb: float | None = 0.5,
    bucket_mode: BucketMode | None = None,
) -> torch.fx.GraphModule:
    """Schedule nodes to maximize compute-collective overlap.

    Args:
        gm: Input graph module to optimize.
        max_in_flight_gb: Maximum GB of concurrent collective data. Too much in flight memory
            can cause memory fragmentation within the CUDA Caching Allocator.
````
- **EN**: Introduces function `schedule_overlap_bucketing`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `gm`, `max_in_flight_gb`, `max_compute_pre_fetch`, `collective_bucketing`, `insert_overlap_deps`, `compute_overlap_multipler`, and `...+14`.
- **CN**: 这里定义了函数`schedule_overlap_bucketing`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `gm`、`max_in_flight_gb`、`max_compute_pre_fetch`、`collective_bucketing`、`insert_overlap_deps`、`compute_overlap_multipler`、`另有14项` 等值。

### Lines 1765-1792 / 第 1765-1792 行
````python
        max_compute_pre_fetch: Maximum mm nodes to pre fetch. Note: should already be limited by max_in_flight_gb and
            max_memory_increase_gb
        collective_bucketing: Enable overlap-preserving collective bucketing.
        insert_overlap_deps: Insert overlap dependencies using control deps operator. This should only be used if
            compiling with inductor, or for subsequent passes before removing the ops prior to execution.
        compute_overlap_multipler: Scale factor for compute time used to hide collectives. This can be used
            to address over or under aggressive overlapping.
        max_coll_distance: Maximum pre fetch or bucketing candidates. Mainly intended for compile time
        custom_runtime_estimation: Override runtime estimation for specific nodes. Called as
            custom_runtime_estimation(node, override_size) -> float | None. To pass pre-computed
            estimations, wrap a dict: lambda node, _: estimations.get(node).
        collective_estimator: Method for estimating collective runtime. "analytical" uses bandwidth formulas,
            "benchmark" uses CUDA events with power-of-2 rounding and interpolation.
        compute_estimator: Method for estimating compute (ATen op) runtime. "analytical" uses roofline model
            estimates (deterministic, no GPU sync), "benchmark" uses GPU benchmarking (more accurate).
        max_memory_increase_gb: Maximum GB increase above baseline memory (absolute cap). If None, no absolute limit.
        max_memory_increase_ratio: Maximum increase as ratio of baseline peak memory. If None, no ratio limit.
            Uses minimum of absolute and ratio limits when both are specified.
        enable_fusion_regions: Enable fusion region detection and cost estimation for fusible ops.
        bucket_mode: Bucketing mode for grouping collectives.
    """
    if not any(is_wait_tensor(n) for n in gm.graph.nodes):
        return gm

    trace_structured(
        "artifact",
        metadata_fn=lambda: {
            "name": "overlap_scheduling_graph_before",
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1793-1820 / 第 1793-1820 行
````python
            "encoding": "string",
        },
        payload_fn=lambda: gm.print_readable(False),
    )
    ret = OverlapScheduler(
        gm,
        compute_overlap_multipler=compute_overlap_multipler,
        max_in_flight_gb=max_in_flight_gb,
        max_coll_distance=max_coll_distance,
        max_compute_pre_fetch=max_compute_pre_fetch,
        custom_runtime_estimation=custom_runtime_estimation,
        collective_bucketing=collective_bucketing,
        insert_overlap_deps=insert_overlap_deps,
        collective_estimator=collective_estimator,
        compute_estimator=compute_estimator,
        max_memory_increase_gb=max_memory_increase_gb,
        max_memory_increase_ratio=max_memory_increase_ratio,
        log_final_collectives_estimations=log_final_collectives_estimations,
        bucket_exposed_first=bucket_exposed_first,
        enable_fusion_regions=enable_fusion_regions,
        bucket_only_internode_comms=bucket_only_internode_comms,
        prioritize_bucketing_during_scheduling=prioritize_bucketing_during_scheduling,
        max_off_bucket_gb=max_off_bucket_gb,
        bucket_mode=bucket_mode,
    ).run()
    trace_structured(
        "artifact",
        metadata_fn=lambda: {
````
- **EN**: Initializes or updates values such as `payload_fn`, `ret`, `compute_overlap_multipler`, `max_in_flight_gb`, `max_coll_distance`, `max_compute_pre_fetch`, and `...+15`. This range continues the implementation of function `schedule_overlap_bucketing`.
- **CN**: 初始化或更新了 `payload_fn`、`ret`、`compute_overlap_multipler`、`max_in_flight_gb`、`max_coll_distance`、`max_compute_pre_fetch`、`另有15项` 等值。这一段延续了函数`schedule_overlap_bucketing` 的具体实现。

### Lines 1821-1848 / 第 1821-1848 行
````python
            "name": "overlap_scheduling_graph_after",
            "encoding": "string",
        },
        payload_fn=lambda: ret.print_readable(False),
    )
    return ret


def schedule_overlap_bucketing_from_inductor_configs(
    gm: torch.fx.GraphModule,
) -> torch.fx.GraphModule:
    """Schedule nodes to maximize compute-collective overlap using inductor configs.

    Reads configuration from torch._inductor.config.aten_distributed_optimizations
    and calls schedule_overlap_bucketing with those settings.
    """
    if not any(is_wait_tensor(n) for n in gm.graph.nodes):
        return gm

    from torch._inductor import config

    dist_opts = config.aten_distributed_optimizations

    kwargs: dict[str, object] = {}

    config_keys = (
        "collective_bucketing",
        "max_compute_pre_fetch",
````
- **EN**: Imports dependencies such as `torch._inductor` for the logic in this range. Introduces function `schedule_overlap_bucketing_from_inductor_configs`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor` 等依赖，为后续逻辑提供基础能力。这里定义了函数`schedule_overlap_bucketing_from_inductor_configs`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1849-1869 / 第 1849-1869 行
````python
        "custom_runtime_estimation",
        "insert_overlap_deps",
        "collective_estimator",
        "compute_estimator",
        "max_memory_increase_gb",
        "max_memory_increase_ratio",
        "compute_overlap_multipler",
        "max_in_flight_gb",
        "max_coll_distance",
        "log_final_collectives_estimations",
        "bucket_exposed_first",
        "bucket_only_internode_comms",
        "enable_fusion_regions",
        "prioritize_bucketing_during_scheduling",
        "bucket_mode",
    )
    for key in config_keys:
        if (val := getattr(dist_opts, key, None)) is not None:
            kwargs[key] = val

    return schedule_overlap_bucketing(gm, **kwargs)  # type: ignore[arg-type]
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `schedule_overlap_bucketing_from_inductor_configs`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`schedule_overlap_bucketing_from_inductor_configs` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `heapq`, `itertools`, `logging`, `sys`, `collections`, `collections.abc`, `dataclasses`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `torch._dynamo.utils`, `torch._inductor`, `torch._inductor.comm_analysis`, `torch._inductor.fx_passes.bucketing`, `torch._inductor.fx_passes.memory_estimator`, `torch._logging`, `torch.fx.operator_schemas`, `torch.utils._ordered_set`, `torch.utils._python_dispatch`, `torch._inductor.pattern_matcher`, `torch._inductor.fx_passes.fusion_regions`, `torch.utils._pytree`, `torch.utils._runtime_estimation`, `torch.utils.flop_counter`, `torch._dynamo.testing`, `torch.distributed`, `torch._subclasses.fake_tensor`, `torch.distributed.distributed_c10d`, `...+2`
