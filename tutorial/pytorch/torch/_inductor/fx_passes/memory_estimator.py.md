# memory_estimator.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/memory_estimator.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It defines classes such as `StorageKey`, `GraphAliasTracker`, and `MemoryTracker`. It exposes functions such as `_size_of_default`, `device_filter`, `build_memory_profile`, `get_fwd_bwd_interactions`, `_is_releasable`, and `get_peak_memory`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。其中定义了 `StorageKey`、`GraphAliasTracker`、`MemoryTracker` 等类。同时提供 `_size_of_default`、`device_filter`、`build_memory_profile`、`get_fwd_bwd_interactions`、`_is_releasable`、`get_peak_memory` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import itertools
import logging
from collections import defaultdict
from collections.abc import Callable
from dataclasses import dataclass

import torch
import torch.fx as fx
from torch.fx.experimental.symbolic_shapes import optimization_hint
from torch.utils._ordered_set import OrderedSet
from torch.utils._pytree import tree_map_only


log = logging.getLogger(__name__)


@dataclass(frozen=True)
class StorageKey:
    storage: torch.UntypedStorage
    device: torch.device
````
- **EN**: Imports dependencies such as `itertools`, `logging`, `collections`, `collections.abc`, `dataclasses`, `torch`, and `...+4` for the logic in this range. Introduces class `StorageKey`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields.
- **CN**: 这里导入了 `itertools`、`logging`、`collections`、`collections.abc`、`dataclasses`、`torch`、`另有4项` 等依赖，为后续逻辑提供基础能力。这里定义了类`StorageKey`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。

### Lines 21-40 / 第 21-40 行
````python

    def __hash__(self) -> int:
        return self.storage._cdata

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, StorageKey):
            return False
        return (
            self.storage._cdata == other.storage._cdata and self.device == other.device
        )


class GraphAliasTracker:
    """
    Tracks storage allocation and usage relationships in an FX graph.

    Differentiates between:
    - Fresh allocations: nodes that allocate new storage (not views/aliases)
    - Uses: nodes that use a storage as input
    """
````
- **EN**: Introduces function `__hash__`, function `__eq__`, class `GraphAliasTracker`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`__hash__`、函数`__eq__`、类`GraphAliasTracker`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 41-60 / 第 41-60 行
````python

    def __init__(self, nodes: list[fx.Node]):
        # Map from node to the fresh storages it allocates (not views/aliases)
        self.node_to_fresh_allocations: dict[fx.Node, OrderedSet[StorageKey]] = {}

        # Map from storage to the node that originally allocated it
        self.storage_to_allocator: dict[StorageKey, fx.Node] = {}

        # Map from node to all storages it uses as inputs
        self.node_to_storage_uses: dict[fx.Node, OrderedSet[StorageKey]] = {}

        # Map from storage to all nodes that use it
        self.storage_to_uses: dict[StorageKey, OrderedSet[fx.Node]] = defaultdict(
            OrderedSet
        )

        # Map from storage to the last node that uses it
        self.storage_to_last_user: dict[StorageKey, fx.Node] = {}

        # Map from node to storages that have their last use at that node
````
- **EN**: Introduces function `__init__`.
- **CN**: 这里定义了函数`__init__`。

### Lines 61-80 / 第 61-80 行
````python
        self.node_to_storages_last_used: dict[fx.Node, OrderedSet[StorageKey]] = (
            defaultdict(OrderedSet)
        )

        # Track all output storages for each node (for building usage graph)
        self.node_to_output_storages: dict[fx.Node, OrderedSet[StorageKey]] = {}

        # First pass: build storage allocations and track uses
        for node in nodes:
            # Get output storages
            output_storages = self._get_output_storages(node)
            self.node_to_output_storages[node] = output_storages

            # Track fresh allocations
            fresh_allocations: OrderedSet[StorageKey] = OrderedSet()
            for storage_key in output_storages:
                if storage_key not in self.storage_to_allocator:
                    self.storage_to_allocator[storage_key] = node
                    fresh_allocations.add(storage_key)
            self.node_to_fresh_allocations[node] = fresh_allocations
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `output_storages`, and `fresh_allocations`. This range continues the implementation of function `GraphAliasTracker.__init__`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `output_storages`、`fresh_allocations` 等值。这一段延续了函数`GraphAliasTracker.__init__` 的具体实现。

### Lines 81-100 / 第 81-100 行
````python

            # Track input storage uses (safe because inputs were already processed)
            input_storages = self._get_input_storages(node)
            self.node_to_storage_uses[node] = input_storages
            for storage_key in input_storages:
                self.storage_to_uses[storage_key].add(node)

        # Second pass: find last users (iterate in reverse)
        for node in reversed(nodes):
            input_storages = self.node_to_storage_uses[node]
            for storage_key in input_storages:
                if storage_key not in self.storage_to_last_user:
                    self.storage_to_last_user[storage_key] = node
                    self.node_to_storages_last_used[node].add(storage_key)

    @staticmethod
    def _get_output_storages(node: fx.Node) -> OrderedSet[StorageKey]:
        """
        Get all storages from a node's outputs.

````
- **EN**: Introduces function `_get_output_storages`. Applies decorators to register behavior or alter how the following definition is constructed. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_storages`.
- **CN**: 这里定义了函数`_get_output_storages`。使用装饰器来注册行为，或改变后续定义的构造方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_storages` 等值。

### Lines 101-120 / 第 101-120 行
````python
        Uses pytree to handle arbitrary nested structures.
        """
        val = node.meta.get("val")
        if val is None:
            return OrderedSet()

        storages: OrderedSet[StorageKey] = OrderedSet()

        def collect_storage(tensor: torch._subclasses.FakeTensor) -> None:
            storages.add(StorageKey(tensor.untyped_storage(), tensor.device))

        # Use tree_map_only to handle FakeTensors in nested structures
        tree_map_only(torch._subclasses.FakeTensor, collect_storage, val)

        return storages

    def _get_input_storages(self, node: fx.Node) -> OrderedSet[StorageKey]:
        """
        Get all storages from a node's inputs.
        """
````
- **EN**: Introduces function `collect_storage`, function `_get_input_storages`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`collect_storage`、函数`_get_input_storages`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 121-140 / 第 121-140 行
````python
        input_storages: OrderedSet[StorageKey] = OrderedSet()

        for input_node in node.all_input_nodes:
            input_storages.update(self.node_to_output_storages[input_node])

        return input_storages

    def get_fresh_allocations(self, node: fx.Node) -> OrderedSet[StorageKey]:
        """Get all fresh storage allocations by this node (not views/aliases)."""
        return self.node_to_fresh_allocations[node]

    def get_storage_uses(self, node: fx.Node) -> OrderedSet[StorageKey]:
        """Get all storages that this node uses as inputs."""
        return self.node_to_storage_uses[node]

    def get_storages_last_used(
        self,
        node: fx.Node,
    ) -> OrderedSet[StorageKey]:
        """
````
- **EN**: Introduces function `get_fresh_allocations`, function `get_storage_uses`, function `get_storages_last_used`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_storages`, and `node`.
- **CN**: 这里定义了函数`get_fresh_allocations`、函数`get_storage_uses`、函数`get_storages_last_used`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_storages`、`node` 等值。

### Lines 141-160 / 第 141-160 行
````python
        Get storages whose last use is at this node.
        """
        return self.node_to_storages_last_used[node]


def _size_of_default(num_bytes: int | torch.SymInt) -> int:
    return optimization_hint(num_bytes)


def device_filter(device: torch.device) -> bool:
    return device.type != "cpu"


def build_memory_profile(
    graph: fx.Graph,
    is_releasable: Callable[[fx.Node], bool],
    size_of: Callable[[int | torch.SymInt], int] | None = None,
) -> list[int]:
    """
    Function to estimate the memory profile of an input FX graph.
````
- **EN**: Introduces function `_size_of_default`, function `device_filter`, function `build_memory_profile`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_size_of_default`、函数`device_filter`、函数`build_memory_profile`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 161-180 / 第 161-180 行
````python

    Args:
    - graph (fx.Graph): The input FX graph for which the memory profile
      is to be estimated.
    - is_releasable (Callable[[fx.Node], bool]): A function that
      determines if a node's memory can be released (e.g. primal nodes
      cannot be released).
    - size_of (Callable[[int | torch.SymInt], int]): A function that converts
      byte counts (possibly symbolic) to concrete integers.

    Returns:
    - List[int]: A list representing the memory profile over the execution
      of the graph, where each entry corresponds to the memory usage at
      a particular point in the execution.
    """

    size_of = size_of or _size_of_default
    nodes = list(graph.nodes)
    alias_info = GraphAliasTracker(nodes)

````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Initializes or updates values such as `Args`, `Returns`, `size_of`, `nodes`, and `alias_info`. This range continues the implementation of function `build_memory_profile`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。初始化或更新了 `Args`、`Returns`、`size_of`、`nodes`、`alias_info` 等值。这一段延续了函数`build_memory_profile` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
    # Build memory profile
    current_memory = 0

    for node in itertools.chain(
        graph.find_nodes(op="placeholder"), graph.find_nodes(op="get_attr")
    ):
        for storage_key in alias_info.get_fresh_allocations(node):
            if device_filter(storage_key.device):
                current_memory += size_of(storage_key.storage.nbytes())

    memory_profile = [current_memory]

    for node in nodes:
        if node.op in ("placeholder", "get_attr", "output"):
            continue

        # Process allocations
        for storage_key in alias_info.get_fresh_allocations(node):
            if device_filter(storage_key.device):
                current_memory += size_of(storage_key.storage.nbytes())
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `current_memory`, and `memory_profile`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `current_memory`、`memory_profile` 等值。

### Lines 201-220 / 第 201-220 行
````python

        memory_profile.append(current_memory)

        # Process deallocations
        # pyrefly: ignore [bad-assignment]
        for storage_key in alias_info.get_storages_last_used(node):
            allocator = alias_info.storage_to_allocator[storage_key]
            if is_releasable(allocator):
                if device_filter(storage_key.device):
                    current_memory -= size_of(storage_key.storage.nbytes())

        memory_profile.append(current_memory)

    return memory_profile


def get_fwd_bwd_interactions(
    fwd_graph: fx.Graph,
    bwd_graph: fx.Graph,
    size_of: Callable[[int | torch.SymInt], int] | None = None,
````
- **EN**: Introduces function `get_fwd_bwd_interactions`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_fwd_bwd_interactions`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 221-240 / 第 221-240 行
````python
) -> tuple[int, OrderedSet[str]]:
    """
    Analyze the interactions between the forward (fwd) and backward (bwd) graphs
    to determine memory usage characteristics.

    Args:
    - fwd_graph (fx.Graph): The forward graph representing the forward pass.
    - bwd_graph (fx.Graph): The backward graph representing the backward pass.
    - size_of (Callable[[int | torch.SymInt], int]): A function that converts
      byte counts (possibly symbolic) to concrete integers.

    Returns:
    - tuple[int, OrderedSet[str]]: A tuple containing:
        1. The baseline memory usage during the backward pass, accounting for
           storages that persist from the forward pass (i.e., in fwd output but
           not in bwd input).
        2. A set of node names whose storage cannot be released during the bwd pass.
           These include nodes that use storage from primals or are in bwd input
           but not in fwd output.
    """
````
- **EN**: Initializes or updates values such as `Args`, and `Returns`. This range continues the implementation of function `get_fwd_bwd_interactions`.
- **CN**: 初始化或更新了 `Args`、`Returns` 等值。这一段延续了函数`get_fwd_bwd_interactions` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python

    size_of = size_of or _size_of_default

    # Build alias info for forward graph
    fwd_nodes = list(fwd_graph.nodes)
    fwd_alias_info = GraphAliasTracker(fwd_nodes)

    # Identify storages allocated by primal placeholder nodes
    primal_storages: OrderedSet[StorageKey] = OrderedSet()
    for node in fwd_graph.find_nodes(op="placeholder"):
        if node.name.startswith("primals"):
            primal_storages.update(fwd_alias_info.get_fresh_allocations(node))

    # Get storages in forward output
    fwd_output_node = next(iter(reversed(fwd_graph.nodes)))[-1]
    assert fwd_output_node.op == "output"
    fwd_output_storages = fwd_alias_info.get_storage_uses(fwd_output_node)

    # Node names that should not be deleted during memory profile estimation of bwd_graph
    do_not_delete: OrderedSet[str] = OrderedSet()
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `size_of`, `fwd_nodes`, `fwd_alias_info`, `primal_storages`, `fwd_output_node`, `fwd_output_storages`, and `...+1`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `size_of`、`fwd_nodes`、`fwd_alias_info`、`primal_storages`、`fwd_output_node`、`fwd_output_storages`、`另有1项` 等值。

### Lines 261-280 / 第 261-280 行
````python

    # Collect all storages in backward inputs and identify nodes to not delete
    bwd_input_storages: OrderedSet[StorageKey] = OrderedSet()
    for node in bwd_graph.find_nodes(op="placeholder"):
        node_storages = GraphAliasTracker._get_output_storages(node)
        bwd_input_storages.update(node_storages)

        # Check if this node uses primal storage
        if node_storages & primal_storages:
            do_not_delete.add(node.name)

        # Check if this node's storages are not in forward outputs
        # (meaning it's an external input to backward pass)
        if not (node_storages & fwd_output_storages):
            do_not_delete.add(node.name)

    # Calculate baseline memory: storages in fwd output but not in bwd input
    # These storages persist throughout the backward pass
    baseline_storages = fwd_output_storages - bwd_input_storages
    bwd_baseline_memory = 0
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `bwd_input_storages`, `node_storages`, `baseline_storages`, and `bwd_baseline_memory`. This range continues the implementation of function `get_fwd_bwd_interactions`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `bwd_input_storages`、`node_storages`、`baseline_storages`、`bwd_baseline_memory` 等值。这一段延续了函数`get_fwd_bwd_interactions` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
    for storage_key in baseline_storages:
        if storage_key.device.type != "cpu":
            bwd_baseline_memory += size_of(storage_key.storage.nbytes())

    return bwd_baseline_memory, do_not_delete


def _is_releasable(n: fx.Node) -> bool:
    # Storages of primals cannot be released during fwd or bwd pass.
    return not n.name.startswith("primals")


def get_peak_memory(
    fwd_graph: fx.Graph,
    bwd_graph: fx.Graph,
) -> int:
    fwd_peak_memory = max(build_memory_profile(fwd_graph, _is_releasable))

    bwd_baseline_memory, bwd_do_not_delete = get_fwd_bwd_interactions(
        fwd_graph,
````
- **EN**: Introduces function `_is_releasable`, function `get_peak_memory`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_is_releasable`、函数`get_peak_memory`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 301-320 / 第 301-320 行
````python
        bwd_graph,
    )

    def _is_bwd_releasable(n: fx.Node) -> bool:
        # Storages of nodes in bwd_do_not_delete cannot be released
        # during the bwd pass.
        return _is_releasable(n) and n.name not in bwd_do_not_delete

    bwd_peak_memory = bwd_baseline_memory + max(
        build_memory_profile(bwd_graph, _is_bwd_releasable)
    )
    return max(
        fwd_peak_memory,
        bwd_peak_memory,
    )


class MemoryTracker:
    """
    Tracks memory usage for alternative scheduling orders of an FX graph.
````
- **EN**: Introduces function `_is_bwd_releasable`, class `MemoryTracker`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `bwd_peak_memory`.
- **CN**: 这里定义了函数`_is_bwd_releasable`、类`MemoryTracker`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `bwd_peak_memory` 等值。

### Lines 321-340 / 第 321-340 行
````python

    This class enables tracking memory usage as nodes are scheduled in a different
    order than the original graph.
    """

    def __init__(
        self,
        graph: fx.Graph,
        is_releasable: Callable[[fx.Node], bool] | None = None,
        device_filter: Callable[[torch.device], bool] | None = None,
    ):
        """
        Initialize memory tracker for alternative scheduling of the given graph.

        Args:
            graph: FX graph to track memory for under alternative scheduling
            is_releaseable: do we consider this input to the graph to release memory
            upon final use, or is allocated for the duration of the graph ?
            by default, we assume all nodes but those that start with "primals" to be releasable
            device_filter: Function to determine which devices to track (default: non-CPU)
````
- **EN**: Introduces function `__init__`. Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `graph`, `is_releasable`, `device_filter`, `Args`, and `is_releaseable`.
- **CN**: 这里定义了函数`__init__`。处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `graph`、`is_releasable`、`device_filter`、`Args`、`is_releaseable` 等值。

### Lines 341-360 / 第 341-360 行
````python
        """

        self.graph = graph
        self.nodes = list(graph.nodes)
        self.device_filter = device_filter or (lambda device: device.type != "cpu")
        self.scheduled: OrderedSet[fx.Node] = OrderedSet()

        # Memory tracking using GraphAliasTracker
        self.alias_tracker = GraphAliasTracker(self.nodes)
        self.current_live_storages: OrderedSet[StorageKey] = OrderedSet()
        self.current_memory_bytes = 0
        self.is_releasable = _is_releasable if is_releasable is None else is_releasable

        # Initialize live storages with placeholders and get_attr nodes
        for node in self.nodes:
            if node.op in ("placeholder", "get_attr"):
                fresh_allocations = self.alias_tracker.get_fresh_allocations(node)
                for storage_key in fresh_allocations:
                    if self.device_filter(storage_key.device):
                        self.current_live_storages.add(storage_key)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fresh_allocations`. This range continues the implementation of function `MemoryTracker.__init__`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `fresh_allocations` 等值。这一段延续了函数`MemoryTracker.__init__` 的具体实现。

### Lines 361-380 / 第 361-380 行
````python
                        self.current_memory_bytes += self._get_storage_size(storage_key)

        self.peak_memory = self.current_memory_bytes

        log.debug(
            "Memory tracker initialized with initial memory: %d MB",
            self.current_memory_bytes // (1024 * 1024),
        )

    def schedule_node(self, node: fx.Node) -> None:
        """
        Schedule a node and update memory tracking for the new scheduling order.

        Args:
            node: The node being scheduled (potentially out of original order)
        """
        assert node not in self.scheduled, "should not schedule node twice"
        self.scheduled.add(node)
        self._update_memory_for_node(node)

````
- **EN**: Introduces function `schedule_node`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `Args`, and `node`.
- **CN**: 这里定义了函数`schedule_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `Args`、`node` 等值。

### Lines 381-400 / 第 381-400 行
````python
    def get_current_memory_bytes(self) -> int:
        """Get current live memory in bytes under the current scheduling."""
        return self.current_memory_bytes

    def _get_storage_size(self, storage_key: StorageKey) -> int:
        """Get the size of a storage in bytes, handling symbolic shapes."""
        size_bytes = storage_key.storage.nbytes()
        return optimization_hint(size_bytes)

    def _get_storages_freed_by_node(self, node: fx.Node) -> OrderedSet[StorageKey]:
        """Get storages that would be freed if we schedule this node."""
        freed_storages: OrderedSet[StorageKey] = OrderedSet()

        input_storages = self.alias_tracker.get_storage_uses(node)
        for storage_key in input_storages:
            if not self.device_filter(storage_key.device):
                continue

            # Invariant: if a node uses a storage, it must be live
            assert storage_key in self.current_live_storages, (
````
- **EN**: Introduces function `get_current_memory_bytes`, function `_get_storage_size`, function `_get_storages_freed_by_node`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_current_memory_bytes`、函数`_get_storage_size`、函数`_get_storages_freed_by_node`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 401-420 / 第 401-420 行
````python
                "all input storages should be currently allocated"
            )

            if not self.is_releasable(
                self.alias_tracker.storage_to_allocator[storage_key]
            ):
                continue

            all_uses = self.alias_tracker.storage_to_uses[storage_key]

            # If no more unscheduled uses remain, the storage can be freed
            if all(u in self.scheduled for u in all_uses):
                freed_storages.add(storage_key)

        return freed_storages

    def _update_memory_for_node(self, node: fx.Node) -> None:
        """Update memory tracking when a node is scheduled."""
        if node.op in ("placeholder", "get_attr", "output"):
            return
````
- **EN**: Introduces function `_update_memory_for_node`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `all_uses`.
- **CN**: 这里定义了函数`_update_memory_for_node`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `all_uses` 等值。

### Lines 421-440 / 第 421-440 行
````python

        # Add fresh allocations
        fresh_allocations = self.alias_tracker.get_fresh_allocations(node)
        alloc_bytes = 0
        for storage_key in fresh_allocations:
            if (
                self.device_filter(storage_key.device)
                and storage_key not in self.current_live_storages
            ):
                size = self._get_storage_size(storage_key)
                self.current_live_storages.add(storage_key)
                self.current_memory_bytes += size
                alloc_bytes += size

        self.peak_memory = max(self.current_memory_bytes, self.peak_memory)

        # Remove storages that are no longer used
        storages_to_free = self._get_storages_freed_by_node(node)
        freed_bytes = 0
        for storage_key in storages_to_free:
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fresh_allocations`, `alloc_bytes`, `size`, `storages_to_free`, and `freed_bytes`. This range continues the implementation of function `MemoryTracker._update_memory_for_node`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `fresh_allocations`、`alloc_bytes`、`size`、`storages_to_free`、`freed_bytes` 等值。这一段延续了函数`MemoryTracker._update_memory_for_node` 的具体实现。

### Lines 441-453 / 第 441-453 行
````python
            if storage_key in self.current_live_storages:
                size = self._get_storage_size(storage_key)
                self.current_live_storages.remove(storage_key)
                self.current_memory_bytes -= size
                freed_bytes += size

        log.debug(
            "Scheduled %s: memory change %d allocs, %d frees, current memory: %d MB",
            node.name,
            len(fresh_allocations),
            len(storages_to_free),
            self.current_memory_bytes // (1024 * 1024),
        )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `size`. This range continues the implementation of function `MemoryTracker._update_memory_for_node`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `size` 等值。这一段延续了函数`MemoryTracker._update_memory_for_node` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Primary classes: `StorageKey`, `GraphAliasTracker`, and `MemoryTracker`  
  **CN**: 主要类：`StorageKey`、`GraphAliasTracker`、`MemoryTracker`
- **EN**: Primary functions: `_size_of_default`, `device_filter`, `build_memory_profile`, `get_fwd_bwd_interactions`, `_is_releasable`, and `get_peak_memory`  
  **CN**: 主要函数：`_size_of_default`、`device_filter`、`build_memory_profile`、`get_fwd_bwd_interactions`、`_is_releasable`、`get_peak_memory`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `itertools`, `logging`, `collections`, `collections.abc`, `dataclasses`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `torch.fx.experimental.symbolic_shapes`, `torch.utils._ordered_set`, `torch.utils._pytree`
