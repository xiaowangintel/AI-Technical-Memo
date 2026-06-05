# partitioner_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/experimental/partitioner_utils.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Holds experimental tracing, shape reasoning, and transformation infrastructure built on FX.
- **Purpose (CN)**: 存放基于 FX 的实验性跟踪、形状推理与变换基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-23
```python
from enum import Enum
from typing import NamedTuple

from torch.fx.node import map_arg, Node


class Partition:
    """Partition class contains all the information about an individual partition.
    It also provides necessary methods for manipulation the partition.
    """

    def __init__(self, partition_id: int) -> None:
        self.nodes: set[Node] = set()
        self.partition_id = partition_id
        self.parents: set[Partition] = set()
        self.children: set[Partition] = set()
        self.bfs_level: int = -1
        self.used_mem_bytes: int = 0
        self.logical_device_ids: list[int] = []

    def __str__(self) -> str:
        return str(self.partition_id)
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果。

### Lines 24-41
```python
    def recalculate_mem_size(self) -> None:
        self.used_mem_bytes = 0
        for node in self.nodes:
            self.used_mem_bytes += get_extra_size_of(node, self.nodes)

    def add_node(self, node: Node) -> None:
        input_nodes: dict[Node, None] = {}
        map_arg(node.args, input_nodes.setdefault)
        map_arg(node.kwargs, input_nodes.setdefault)
        # Add current node's input nodes if they are placeholder or constants
        for n in input_nodes:
            if n.op in {"placeholder", "get_attr"}:
                self.nodes.add(n)
        self.nodes.add(node)
        self.recalculate_mem_size()

    def remove_node(self, node: Node) -> None:
        # Remove a node only if the node is in the partition
```
- **EN**: Declares `Partition`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `Partition`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 42-64
```python
        if node in self.nodes:
            self.nodes.remove(node)
            # Collect the node's input nodes
            input_nodes: dict[Node, None] = {}
            map_arg(node.args, input_nodes.setdefault)
            map_arg(node.kwargs, input_nodes.setdefault)
            # Check if an input node is a placeholder or get_attr,
            # and this input node is not used by some other nodes in this partition,
            # the remove this input node
            for input_node in input_nodes:
                if all(
                    n not in self.nodes for n in input_node.users
                ) and input_node.op in {"placeholder", "get_attr"}:
                    self.nodes.remove(input_node)
            self.recalculate_mem_size()


class Device(NamedTuple):
    name: str
    available_mem_bytes: int
    logical_id: int
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 65-88
```python
class NodeLatency(NamedTuple):
    # Latency due to the memory bandwidth
    mem_latency_sec: float
    # Latency due to the computation
    computer_latency_sec: float


class PartitionLatency(NamedTuple):
    # Sum of all nodes' memory latency on the critical path
    mem_latency_sec: float
    # Sum of all nodes' compute latency on the critical path
    computer_latency_sec: float
    # Latency of the critical path
    overall_latency_sec: float


class PartitionMode(Enum):
    size_based = 0
    sparse_nn = 1
    cost_aware = 2
    kl_based = 3
    aot_based = 4
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。

### Lines 89-106
```python
class PartitionerConfig(NamedTuple):
    devices: list[Device]
    mode: PartitionMode = PartitionMode.size_based
    transfer_rate_bytes_per_sec: float = 0.0
    node_to_latency_mapping: dict[Node, NodeLatency] = {}
    node_to_partition_mapping: dict[Node, int] = {}
    partition_to_logical_device_mapping: dict[int, list[int]] = {}
    # Saturate host by replicating partitions to the remaining idle devices.
    saturate_host: bool = False


def get_extra_size_of(node: Node, nodes: set[Node]) -> int:
    """Given a node and a set of nodes,
    this function return the extra size that needed
    if this node is included in this set.
    """
    # Find all its input nodes
    input_nodes: dict[Node, None] = {}
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 107-132
```python
    map_arg(node.args, input_nodes.setdefault)
    map_arg(node.kwargs, input_nodes.setdefault)
    # Calculate total size of related nodes
    total_size_of_input_nodes = 0
    for n in input_nodes:
        # Make sure this node hasn't been in this set yet
        if n not in nodes:
            size_bytes = getattr(n, "size_bytes", None)
            if size_bytes:
                total_size_of_input_nodes += size_bytes.output_size
            else:
                raise RuntimeError("node has no size_bytes attr")
    # Don't forget the op node itself
    size_bytes = getattr(node, "size_bytes", None)
    if size_bytes:
        total_size_of_input_nodes += size_bytes.total_size
    else:
        raise RuntimeError("node has no size_bytes attr")
    return total_size_of_input_nodes


def get_latency_of_one_partition(
    partition: Partition, node_to_latency_mapping: dict[Node, NodeLatency]
) -> PartitionLatency:
    """Given a partition and its nodes' latency, return a PartitionLatency for this partition"""
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；遍历输入、节点或簿记结构。

### Lines 133-152
```python
    def get_top_nodes(partition: Partition) -> list[Node]:
        """Given a partition, return a list of nodes on the top bfs level"""
        top_nodes: list[Node] = []
        for node in partition.nodes:
            # Skip placeholder and get_attr nodes
            if node.op in {"placeholder", "get_attr"}:
                continue
            input_nodes: dict[Node, None] = {}
            map_arg(node.args, input_nodes.setdefault)
            map_arg(node.kwargs, input_nodes.setdefault)
            # If a node has no input nodes in this partition,
            # or its input nodes in this partition are placeholders and get_attrs
            # this node is on the top bfs level in this partition
            if not any(
                n in partition.nodes and n.op not in {"placeholder", "get_attr"}
                for n in input_nodes
            ):
                top_nodes.append(node)
        return top_nodes
```
- **EN**: Defines the `get_latency_of_one_partition` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`get_latency_of_one_partition` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 153-170
```python
    def dfs_helper(node: Node, partition_latency: PartitionLatency) -> PartitionLatency:
        """Given a top node of a partition, this function returns
        the latency of the critical path in the partition
        """
        node_latency = node_to_latency_mapping[node]
        # Calculate the current overall latency of the partition
        overall_latency_sec = partition_latency.overall_latency_sec + max(
            node_latency.computer_latency_sec, node_latency.mem_latency_sec
        )
        # Update the mem latency of this path
        mem_latency_sec = (
            partition_latency.mem_latency_sec + node_latency.mem_latency_sec
        )
        # Update the compute latency of this path
        computer_latency_sec = (
            partition_latency.computer_latency_sec + node_latency.computer_latency_sec
        )
        # Get all users of this node that are in this partition
```
- **EN**: Defines the `get_latency_of_one_partition.dfs_helper` function; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`get_latency_of_one_partition.dfs_helper` 函数；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 171-194
```python
        users = set(node.users).intersection(partition.nodes)
        if users:
            max_latency = PartitionLatency(
                mem_latency_sec=0.0, computer_latency_sec=0.0, overall_latency_sec=0.0
            )
            for n in users:
                # Get new partition latency recursively
                new_partition_latency = dfs_helper(
                    n,
                    PartitionLatency(
                        mem_latency_sec, computer_latency_sec, overall_latency_sec
                    ),
                )
                if (
                    new_partition_latency.overall_latency_sec
                    > max_latency.overall_latency_sec
                ):
                    max_latency = new_partition_latency
            return max_latency
        # If there is no user, the node is at bottom of the partition
        return PartitionLatency(
            mem_latency_sec, computer_latency_sec, overall_latency_sec
        )
```
- **EN**: This block continues `get_latency_of_one_partition` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `get_latency_of_one_partition`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 195-216
```python
    # Main part starts
    # Get all top level nodes of this partition
    top_nodes = get_top_nodes(partition)
    critical_path_latency = PartitionLatency(
        mem_latency_sec=0.0, computer_latency_sec=0.0, overall_latency_sec=0.0
    )
    # Go through all top nodes and find the largest latency (critical pass latency)
    for node in top_nodes:
        partition_latency = dfs_helper(
            node,
            PartitionLatency(
                mem_latency_sec=0.0, computer_latency_sec=0.0, overall_latency_sec=0.0
            ),
        )
        if (
            partition_latency.overall_latency_sec
            > critical_path_latency.overall_latency_sec
        ):
            critical_path_latency = partition_latency
    return critical_path_latency
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 217-234
```python
def get_partition_to_latency_mapping(
    partitions: list[Partition], node_to_latency_mapping: dict[Node, NodeLatency]
) -> dict[Partition, PartitionLatency]:
    """Given all the partitions and node_to_latency_mapping dictionary,
    return a mapping dictionary of each partition to its overall latency
    """
    partition_to_latency_mapping: dict[Partition, PartitionLatency] = {}
    # Go through each partition and get its latency
    for partition in partitions:
        partition_latency = get_latency_of_one_partition(
            partition, node_to_latency_mapping
        )
        partition_to_latency_mapping[partition] = partition_latency
    return partition_to_latency_mapping


def get_comm_latency_between(
    parent_partition: Partition,
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 235-252
```python
    child_partition: Partition,
    transfer_rate_bytes_per_sec: float,
) -> float:
    """Given two partitions (parent and child),
    calculate the communication latency between the two.
    """
    # If two partitions are on the same device, the comm latency is 0.
    if (
        parent_partition.logical_device_ids != []
        and child_partition.logical_device_ids != []
        and parent_partition.logical_device_ids == child_partition.logical_device_ids
    ):
        return 0.0
    # Keep tracking the communication size between parent and child
    comm_size = 0
    # Keep tracking all the counted node
    visited_nodes = set()
    # Go through all nodes in the child partition
```
- **EN**: This block continues `get_comm_latency_between` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `get_comm_latency_between`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 253-277
```python
    # If a node has input nodes from the parent partition,
    # the output size of those input nodes will be counted
    # and added to comm_size
    for node in child_partition.nodes:
        input_nodes: dict[Node, None] = {}
        map_arg(node.args, input_nodes.setdefault)
        map_arg(node.kwargs, input_nodes.setdefault)
        for n in input_nodes:
            if n in parent_partition.nodes and n not in visited_nodes:
                size_bytes = getattr(n, "size_bytes", None)
                if size_bytes is not None:
                    comm_size += size_bytes.output_size
                visited_nodes.add(n)
    return comm_size / transfer_rate_bytes_per_sec


def get_latency_of_partitioned_graph(
    partitions: list[Partition],
    partition_to_latency_mapping: dict[Partition, PartitionLatency],
    transfer_rate_bytes_per_sec: float,
) -> float:
    """Given all partitions in a graph, find the critical path among all partitions
    and return its latency as the latency of the whole graph
    """
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 278-299
```python
    def dfs_helper(partition: Partition, latency_so_far_sec: float) -> float:
        """This function helps to recursively get the latency of a path of partitions"""
        # Update latency by adding current partition's latency
        latency_so_far_sec += partition_to_latency_mapping[
            partition
        ].overall_latency_sec

        if partition.children:
            max_latency_sec = 0.0
            for child in partition.children:
                # Calculate latency between
                comm_latency_sec = get_comm_latency_between(
                    partition, child, transfer_rate_bytes_per_sec
                )
                new_latency_sec = dfs_helper(
                    child, latency_so_far_sec + comm_latency_sec
                )
                if new_latency_sec > max_latency_sec:
                    max_latency_sec = new_latency_sec
            return max_latency_sec
        return latency_so_far_sec
```
- **EN**: Defines the `get_latency_of_partitioned_graph` function; this block introduces logic that manipulate FX graphs, tracing helpers, or transformation utilities.
- **CN**: 定义`get_latency_of_partitioned_graph` 函数；该代码块引入了用于操作 FX 图、跟踪辅助逻辑或变换工具的逻辑。

### Lines 300-316
```python
    def get_top_partitions(partitions: list[Partition]) -> list[Partition]:
        """This function is to return all the partitions without parents
        as the starting points of all the paths
        """
        # If a partition has no parents, then it is a top partition
        top_partitions = [
            partition for partition in partitions if len(partition.parents) == 0
        ]
        return top_partitions

    top_partitions = get_top_partitions(partitions)
    critical_path_latency_sec = 0.0
    for partition in top_partitions:
        latency_sec = dfs_helper(partition, 0.0)
        if latency_sec > critical_path_latency_sec:
            critical_path_latency_sec = latency_sec
    return critical_path_latency_sec
```
- **EN**: Defines the `get_latency_of_partitioned_graph` function; this block introduces logic that manipulate FX graphs, tracing helpers, or transformation utilities.
- **CN**: 定义`get_latency_of_partitioned_graph` 函数；该代码块引入了用于操作 FX 图、跟踪辅助逻辑或变换工具的逻辑。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
- **Program transforms / 程序变换**
  - EN: The implementation rewrites existing program structure into a more useful form.
  - CN: 该实现会把现有程序结构改写成更有用的形式。
- **API boundaries / API 边界**
  - EN: Imports, exports, and helper definitions in this file define how adjacent modules cooperate.
  - CN: 本文件中的导入、导出与辅助定义共同界定了相邻模块之间的协作边界。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch.fx.node`
- **Standard library / 标准库**: `enum`, `typing`
- **Primary symbols / 核心符号**: `Partition`, `Device`, `NodeLatency`, `PartitionLatency`, `PartitionMode`, `PartitionerConfig`, `get_extra_size_of`, `get_latency_of_one_partition`, `get_partition_to_latency_mapping`, `get_comm_latency_between`, `get_latency_of_partitioned_graph`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
