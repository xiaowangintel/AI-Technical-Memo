# partitioner.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/fx/passes/infra/partitioner.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Collects graph-to-graph optimization, rewriting, and analysis passes for FX IR.
- **Purpose (CN)**: 汇集针对 FX 中间表示的图到图优化、改写与分析 passes。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
import collections
import itertools
import logging
import operator
from collections.abc import Iterable, Sequence

from torch.fx.graph_module import GraphModule
from torch.fx.node import _get_qualified_name, Node
from torch.fx.passes.operator_support import OperatorSupportBase
from torch.fx.passes.utils.fuser_utils import fuse_by_partitions


logger = logging.getLogger(__name__)
logger.setLevel(logging.WARNING)


class Partition:
    def __init__(
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。

### Lines 19-43
```python
        self,
        id: int | None = None,
        nodes: Iterable[Node] | None = None,
        node_orders: Iterable[int] | None = None,
    ) -> None:
        self.id = id
        self.nodes: dict[Node, int | None] = {}
        if nodes is not None:
            if node_orders is None:
                self.nodes = dict.fromkeys(nodes, None)
            else:
                nodes_list = list(nodes)
                node_orders_list = list(node_orders)
                if len(nodes_list) != len(node_orders_list):
                    raise AssertionError(
                        "nodes and node_orders must have the same length"
                    )
                self.nodes = dict(zip(nodes_list, node_orders_list))

    def __repr__(self) -> str:
        return str(self.nodes)

    def add_node(self, node: Node, node_order: int | None = None) -> None:
        self.nodes.update({node: node_order})
```
- **EN**: Declares `Partition`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `Partition`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 44-64
```python
    def remove_node(self, node: Node) -> None:
        del self.nodes[node]

    def size(self) -> int:
        return len(self.nodes)


class _DependencyViewer:
    def __init__(self, graph_module: GraphModule) -> None:
        self.downstreams = collections.defaultdict(set)

        for node in reversed(graph_module.graph.nodes):
            for output_node in node.users:
                # add output_node and output_node's downstream dependency
                self.downstreams[node].add(output_node)
                self.downstreams[node].update(self.downstreams[output_node])

    def downstreams_of(self, node: Node) -> set[Node]:
        return self.downstreams[node]
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构。

### Lines 65-89
```python
class CapabilityBasedPartitioner:
    def __init__(
        self,
        graph_module: GraphModule,
        operator_support: OperatorSupportBase,
        allows_single_node_partition: bool = False,
        non_compute_ops: Sequence[str] | None = None,
        allowed_single_node_partition_ops: Sequence[str] | None = None,
    ) -> None:
        self.graph_module = graph_module
        self.operator_support = operator_support
        self.allows_single_node_partition = allows_single_node_partition
        self.non_compute_ops = non_compute_ops if non_compute_ops is not None else []
        self.allowed_single_node_partition_ops = (
            allowed_single_node_partition_ops
            if allowed_single_node_partition_ops is not None
            else []
        )
        self.dependency_viewer = _DependencyViewer(graph_module)

    def _is_node_supported(self, node: Node) -> bool:
        return self.operator_support.is_node_supported(
            dict(self.graph_module.named_modules()), node
        )
```
- **EN**: Declares `CapabilityBasedPartitioner`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `CapabilityBasedPartitioner`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 90-111
```python
    def propose_partitions(self) -> list[Partition]:
        # partition_map is a mapping from partition id to a set of partition id's.
        # The value set contains all the partition ids that can be reached by doing a
        # DFS starting from the partition id in the key.
        partition_map: dict[int, set[int]] = collections.defaultdict(set)

        # assumptions: nodes in candidate list is sorted in topological order
        assignment: dict[Node, int] = {}  # mapping from node to partition_id
        partitions_by_id: dict[
            int, Partition
        ] = {}  # mapping from partition_id to partition
        nodes_order: dict[
            Node, int
        ] = {}  # mapping from nodes to reversed topological order
        partitions_order: dict[
            int, int
        ] = {}  # mapping from partition_id to minimum topo order of nodes in partition
        partition_users: dict[
            int, set[Node]
        ] = {}  # mapping from partition_id to partition users
        new_partition_id = itertools.count()
```
- **EN**: Defines the `CapabilityBasedPartitioner.propose_partitions` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`CapabilityBasedPartitioner.propose_partitions` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 112-129
```python
        # try to merge partition other_id into partition self_id
        # merge only happens if the end graph doesn't contain cyclic dependency
        # returns `True` when merge happens, `False` otherwise.
        def maybe_merge_partition(self_id: int, other_id: int) -> tuple[int, bool]:
            # merged_nodes is the union of nodes in two partition to-be-merged
            self_nodes = partitions_by_id[self_id].nodes
            other_nodes = partitions_by_id[other_id].nodes

            def dfs_iter_find_cycle(all_user_nodes: set[Node]) -> bool:
                for user_node in all_user_nodes:
                    visited_partition_ids = set()

                    for path_node in self.dependency_viewer.downstreams_of(user_node):
                        # If any of the nodes in the dfs path of this node are in the merged_nodes
                        # list then there is a cycle in the graph.
                        if path_node in self_nodes or path_node in other_nodes:
                            return True
```
- **EN**: Defines the `CapabilityBasedPartitioner.propose_partitions` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`CapabilityBasedPartitioner.propose_partitions` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 130-152
```python
                        # If any of the nodes in the dfs path of this node are in the assignment
                        # map then we have to make sure that the partitions that these nodes belong
                        # to do not form a cycle with the current partitions being merged. This means
                        # iterating through all the nodes in all the parititons that are traversed in
                        # the dfs path and checking if they are in the merged_nodes list.
                        if path_node in assignment:
                            partition_id = assignment[path_node]
                            # If the partition id has already been visited then we know that it doesn't
                            # form a cycle with the current partitions being merged.
                            if partition_id in visited_partition_ids:
                                continue
                            p_map = partition_map[partition_id]
                            if self_id in p_map or other_id in p_map:
                                return True

                            visited_partition_ids.add(partition_id)

                return False

            # find new partition users if merge.
            all_user_nodes = partition_users[self_id] | partition_users[other_id]
            all_user_nodes.difference_update(other_nodes, self_nodes)
```
- **EN**: This block continues `CapabilityBasedPartitioner.propose_partitions.maybe_merge_partition` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CapabilityBasedPartitioner.propose_partitions.maybe_merge_partition`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 153-176
```python
            # check if merge would create cyclic dependency.
            if dfs_iter_find_cycle(all_user_nodes):
                # return false indicating cyclic dependency found and
                # merge is aborted
                return self_id, False

            # merge the smaller partition into the larger.
            merge_id, removed_id = self_id, other_id
            if len(self_nodes) < len(other_nodes):
                merge_id, removed_id = removed_id, merge_id
            # no cyclic dependency found, move forward with the merge
            # updating partition nodes
            partitions_by_id[merge_id].nodes.update(partitions_by_id[removed_id].nodes)
            # updating assignment map
            for node in partitions_by_id[removed_id].nodes:
                assignment[node] = merge_id
            # delete other partition
            del partitions_by_id[removed_id]

            partitions_order[merge_id] = min(
                partitions_order[merge_id], partitions_order[removed_id]
            )
            del partitions_order[removed_id]
```
- **EN**: This block continues `CapabilityBasedPartitioner.propose_partitions.maybe_merge_partition` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CapabilityBasedPartitioner.propose_partitions.maybe_merge_partition`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 177-201
```python
            partition_map[merge_id] = partition_map[merge_id].union(
                partition_map[removed_id]
            )
            del partition_map[removed_id]

            partition_users[merge_id] = all_user_nodes
            del partition_users[removed_id]

            return merge_id, True

        def merge_single_node(
            node: Node, node_order: int | None, id: int | None
        ) -> None:
            def _update_partition_map(node: Node, id: int) -> None:
                # Iterate through all the users of this node and update the partition map to indicate
                # that there is a path from the partition id of this node to the target partition id.
                for user_node in node.users:
                    target_id = assignment.get(user_node)
                    if target_id is not None:
                        partition_map[id].add(target_id)
                        partition_map[id].update(partition_map[target_id])

            if node in assignment:
                partitions_by_id[assignment[node]].remove_node(node)
```
- **EN**: Defines the `CapabilityBasedPartitioner.propose_partitions` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`CapabilityBasedPartitioner.propose_partitions` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 202-222
```python
            if id is None:
                assignment.pop(node)
            elif id not in partitions_by_id:
                assignment[node] = id
                if node_order is None:
                    raise AssertionError("node_order is required for new partitions")
                partitions_by_id[id] = Partition(
                    id=id, nodes=[node], node_orders=[node_order]
                )
                partition_users[id] = set(node.users)
                _update_partition_map(node, id)
            else:
                assignment[node] = id
                partitions_by_id[id].add_node(node, node_order)

        logger.debug("Proposing partitions...")

        for node_order, node in enumerate(reversed(self.graph_module.graph.nodes)):
            # use Dict as an ordered set to ensure deterministic partitioning result, don't care value
            merge_candidates: dict[int, None] = {}
```
- **EN**: This block continues `CapabilityBasedPartitioner.propose_partitions` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CapabilityBasedPartitioner.propose_partitions`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 223-248
```python
            # Note a limited horizontal fusion is enabled:
            #   when `node` is not supported, the code below attempts to fuse consumer of `node`.
            #
            # I don't see a need to add a knob to disable horizontal fusion yet, we can short-cut
            # the fusion by adding an `else` block here to skip horizontal fusion.
            if self._is_node_supported(node) and node not in assignment:
                partition_id = next(new_partition_id)
                nodes_order[node] = partition_id
                partitions_order[partition_id] = partition_id
                merge_single_node(node, node_order, partition_id)
                merge_candidates[partition_id] = None

            # merge all possible partitions
            for partition_id, _ in sorted(
                partitions_order.items(), key=operator.itemgetter(1)
            ):
                merge_candidates[partition_id] = None

            merge_candidates_list = list(merge_candidates.keys())
            if len(merge_candidates_list) > 1:
                self_id = merge_candidates_list[0]
                for other_id in merge_candidates_list[1:]:
                    # note: merge partitions if it doesn't create cyclic dependency
                    # in the graph, otherwise, this is a no-op
                    self_id, _ = maybe_merge_partition(self_id, other_id)
```
- **EN**: This block continues `CapabilityBasedPartitioner.propose_partitions` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CapabilityBasedPartitioner.propose_partitions`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 249-272
```python
        # sort partition nodes based on descending node order
        for partition in partitions_by_id.values():
            partition.nodes = dict(
                sorted(
                    partition.nodes.items(), key=operator.itemgetter(1), reverse=True
                )
            )

        # post processing to re-assign "getitem" nodes into upstream partition
        # Run iteratively until no more changes, to handle nested getitem chains
        # (e.g., getitem_619 = getitem_618[0] where getitem_618 = with_effects_167[1])
        logger.debug("Reassigning getitem nodes to its producer node's partition...")
        while True:
            nodes_reassignment: dict[Node, int] = {}
            for node in self.graph_module.graph.nodes:
                is_tuple_output = True
                for user in node.users:
                    if (
                        user.op != "call_function"
                        or _get_qualified_name(user.target) != "_operator.getitem"
                    ):  # type: ignore[arg-type]
                        is_tuple_output = False
                        break
```
- **EN**: This block continues `CapabilityBasedPartitioner.propose_partitions` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CapabilityBasedPartitioner.propose_partitions`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 273-290
```python
                # node has tuple outputs, re-assign all following getitem node into node's partition
                if is_tuple_output:
                    id = assignment.get(node)  # type: ignore[arg-type]
                    for user in node.users:
                        if assignment.get(user) != id:  # type: ignore[arg-type]
                            nodes_reassignment[user] = id  # type: ignore[assignment]

            # no more re-assignments
            if not nodes_reassignment:
                break

            for node, id in nodes_reassignment.items():
                merge_single_node(node, None, id)

        # filter out single node partitions
        if not self.allows_single_node_partition:
            logger.debug("Filtering out single node partitions...")
            default_non_compute_ops = {"torch.ops.aten.view", "_operator.getitem"}
```
- **EN**: This block continues `CapabilityBasedPartitioner.propose_partitions` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CapabilityBasedPartitioner.propose_partitions`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 291-312
```python
            non_compute_ops = default_non_compute_ops.union(set(self.non_compute_ops))
            partitions_to_remove: list[int] = []
            for id, partition in partitions_by_id.items():
                compute_node_count = 0
                for node in partition.nodes:
                    if node.op == "call_function":
                        if not callable(node.target):
                            raise AssertionError(
                                f"Expected callable target, got {type(node.target)}"
                            )
                        if _get_qualified_name(node.target) not in non_compute_ops:
                            compute_node_count += 1
                        if (
                            _get_qualified_name(node.target)
                            in self.allowed_single_node_partition_ops
                        ):
                            compute_node_count += 1
                if compute_node_count <= 1:
                    partitions_to_remove.append(id)
            for id in partitions_to_remove:
                del partitions_by_id[id]
```
- **EN**: This block continues `CapabilityBasedPartitioner.propose_partitions` and works to build, traverse, or rewrite graph structures and their metadata. It also validates assumptions before proceeding; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CapabilityBasedPartitioner.propose_partitions`，用于构建、遍历或改写图结构及其元数据。 同时它还会在继续前校验关键假设；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 313-337
```python
        logger.debug("Partitions proposed:")
        for id, partition in partitions_by_id.items():
            logger.debug(
                "partition #%s: %s", id, [node.name for node in partition.nodes]
            )

        return [
            partition for partition in partitions_by_id.values() if partition.size() > 0
        ]

    def fuse_partitions(
        self, partitions: list[Partition], prefix: str = "fused_"
    ) -> GraphModule:
        logger.debug("Fusing partitions...")
        # fuse_by_partitions expects partitions in List[Dict[Node, None]]: [ {node0 : None}, {node1 : None} ]
        return fuse_by_partitions(
            self.graph_module,
            [partition.nodes for partition in partitions],
            prefix=prefix,
        )

    # remove non-compute-ops that sits at the boundary of a partition.
    def remove_bookend_non_compute_ops(self, partitions: list[Partition]) -> None:
        non_compute_ops = set(self.non_compute_ops)
```
- **EN**: Declares `CapabilityBasedPartitioner`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `CapabilityBasedPartitioner`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 338-355
```python
        def is_non_compute_node(node: Node) -> bool:
            return (
                node.op == "call_function"
                and _get_qualified_name(node.target) in non_compute_ops  # type: ignore[arg-type]
            )

        # cache transparent nodes
        transparent_input_nodes: dict[Node, bool] = {}
        transparent_output_nodes: dict[Node, bool] = {}

        def is_transparent_input_node(
            node: Node, partition: set[Node], removed_nodes: set[Node]
        ) -> bool:
            if (
                node.op == "placeholder"
                or (node not in partition)
                or (node in removed_nodes)
            ):
```
- **EN**: Defines the `CapabilityBasedPartitioner.remove_bookend_non_compute_ops` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`CapabilityBasedPartitioner.remove_bookend_non_compute_ops` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 356-373
```python
                return True
            if node in transparent_input_nodes:
                return transparent_input_nodes[node]
            if is_non_compute_node(node):
                for input_n in node.all_input_nodes:
                    if not is_transparent_input_node(input_n, partition, removed_nodes):
                        transparent_input_nodes[node] = False
                        return False
                transparent_input_nodes[node] = True
                return True
            transparent_input_nodes[node] = False
            return False

        def is_transparent_output_node(
            node: Node, partition: set[Node], removed_nodes: set[Node]
        ) -> bool:
            if (
                node.op == "placeholder"
```
- **EN**: Defines the `CapabilityBasedPartitioner.remove_bookend_non_compute_ops` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`CapabilityBasedPartitioner.remove_bookend_non_compute_ops` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

### Lines 374-391
```python
                or (node not in partition)
                or (node in removed_nodes)
            ):
                return True
            if node in transparent_output_nodes:
                return transparent_output_nodes[node]
            if is_non_compute_node(node):
                for output_n in node.users:
                    if not is_transparent_output_node(
                        output_n, partition, removed_nodes
                    ):
                        transparent_output_nodes[node] = False
                        return False
                transparent_output_nodes[node] = True
                return True
            transparent_output_nodes[node] = False
            return False
```
- **EN**: This block continues `CapabilityBasedPartitioner.remove_bookend_non_compute_ops` and works to build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CapabilityBasedPartitioner.remove_bookend_non_compute_ops`，用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 392-409
```python
        for partition in partitions:
            # Note it's ok to use `set` here, since we are only query if a node
            # has been removed. We are NEVER going to iterate on nodes inside
            # the set.
            remove_node: set[Node] = set()
            for node in partition.nodes:
                if is_non_compute_node(node) and (
                    is_transparent_input_node(node, set(partition.nodes), remove_node)
                    or is_transparent_output_node(
                        node, set(partition.nodes), remove_node
                    )
                ):
                    remove_node.add(node)

            if len(remove_node) != 0:
                for node in remove_node:
                    partition.nodes.pop(node, None)
```
- **EN**: This block continues `CapabilityBasedPartitioner` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CapabilityBasedPartitioner`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 410-413
```python
    def partition_and_fuse(self, prefix: str = "fused_") -> GraphModule:
        partitions = self.propose_partitions()
        fused_gm = self.fuse_partitions(partitions, prefix=prefix)
        return fused_gm
```
- **EN**: Defines the `CapabilityBasedPartitioner.partition_and_fuse` method; this block introduces logic that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 定义`CapabilityBasedPartitioner.partition_and_fuse` 方法；该代码块引入了用于构建、遍历或改写图结构及其元数据的逻辑。

## Key Concepts / 关键概念

- **FX infrastructure / FX 基础设施**
  - EN: The file extends the FX toolkit used for symbolic tracing and graph transformation.
  - CN: 该文件扩展了用于符号跟踪和图变换的 FX 工具链。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
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

- **Internal PyTorch modules / PyTorch 内部模块**: `torch.fx.graph_module`, `torch.fx.node`, `torch.fx.passes.operator_support`, `torch.fx.passes.utils.fuser_utils`
- **Standard library / 标准库**: `collections`, `itertools`, `logging`, `operator`, `collections.abc`
- **Primary symbols / 核心符号**: `Partition`, `_DependencyViewer`, `CapabilityBasedPartitioner`
- **Relationship summary / 关系总结**: Dependencies mainly support FX graph IR, tracing machinery, and graph transformation utilities. / 依赖关系主要支撑 FX 图中间表示、跟踪机制和图变换工具。
