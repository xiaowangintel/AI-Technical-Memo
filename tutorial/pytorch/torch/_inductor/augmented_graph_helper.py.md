# augmented_graph_helper.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/augmented_graph_helper.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `AugmentedGraphHelper`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `AugmentedGraphHelper` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
import logging
from collections import defaultdict

import torch
import torch.fx as fx
from torch._logging import trace_structured
from torch.utils._ordered_set import OrderedSet


log = logging.getLogger(__name__)


class AugmentedGraphHelper:
    """
    Graph helper that augments the original graph with additional
    dependencies and uses, plus tracks node equivalences for coalescing.

    TODO: if this becomes too large of compile time, consider binding
    graphcycles.cc
    """
````
- **EN**: Imports dependencies such as `logging`, `collections`, `torch`, `torch.fx`, `torch._logging`, and `torch.utils._ordered_set` for the logic in this range. Introduces class `AugmentedGraphHelper`. Initializes or updates values such as `log`, and `TODO`.
- **CN**: 这里导入了 `logging`、`collections`、`torch`、`torch.fx`、`torch._logging`、`torch.utils._ordered_set` 等依赖，为后续逻辑提供基础能力。这里定义了类`AugmentedGraphHelper`。初始化或更新了 `log`、`TODO` 等值。

### Lines 21-40 / 第 21-40 行
````python

    def __init__(
        self,
        graph: fx.Graph,
        node_ancestors: dict[fx.Node, OrderedSet[fx.Node]] | None = None,
    ):
        # Each node starts in its own singleton set
        self.graph = graph
        self.merge_sets = {node: OrderedSet([node]) for node in graph.nodes}

        # Extra dependencies: node depends on dep (dep must come before node)
        self.extra_deps: dict[fx.Node, OrderedSet[fx.Node]] = defaultdict(OrderedSet)
        # Extra uses: reverse of extra_deps (node is used by user)
        self.extra_uses: dict[fx.Node, OrderedSet[fx.Node]] = defaultdict(OrderedSet)
        # Note: only reflect original ancestors, not maintained through additional deps
        # or merge sets
        self.node_ancestors = node_ancestors

    def add_extra_dep(self, *, n: fx.Node, dep: fx.Node) -> None:
        """Add extra dependency: node depends on dep."""
````
- **EN**: Introduces function `__init__`, function `add_extra_dep`. Initializes or updates values such as `graph`, and `node_ancestors`.
- **CN**: 这里定义了函数`__init__`、函数`add_extra_dep`。初始化或更新了 `graph`、`node_ancestors` 等值。

### Lines 41-60 / 第 41-60 行
````python
        self.extra_deps[n].add(dep)
        self.extra_uses[dep].add(n)

    def remove_extra_dep(self, *, n: fx.Node, dep: fx.Node) -> None:
        if dep in self.extra_deps[n]:
            self.extra_deps[n].discard(dep)
            self.extra_uses[dep].discard(n)

    def merge_to_set(self, existing_node: fx.Node, new_node: fx.Node) -> None:
        """
        Merge new_node into existing_node's set. The new node must be a singleton set.
        """
        existing_set = self.merge_sets[existing_node]
        new_set = self.merge_sets[new_node]
        assert len(new_set) == 1

        # Add all nodes from new_set to existing_set
        existing_set.update(new_set)

        # Update all nodes from new_set to point to existing_set
````
- **EN**: Introduces function `remove_extra_dep`, function `merge_to_set`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `existing_set`, and `new_set`.
- **CN**: 这里定义了函数`remove_extra_dep`、函数`merge_to_set`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `existing_set`、`new_set` 等值。

### Lines 61-80 / 第 61-80 行
````python
        for node in new_set:
            self.merge_sets[node] = existing_set

    def unmerge_node(self, node: fx.Node) -> None:
        """Remove a node from its merge set, making it singleton."""
        old_set = self.merge_sets[node]

        # If already singleton, nothing to do
        if len(old_set) == 1:
            return

        # Remove from old set
        old_set.remove(node)

        # Make node singleton
        self.merge_sets[node] = OrderedSet([node])

    def get_merged_deps(self, node: fx.Node) -> OrderedSet[fx.Node]:
        """
        Get all dependencies of a node considering merges and extra deps.
````
- **EN**: Introduces function `unmerge_node`, function `get_merged_deps`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `old_set`.
- **CN**: 这里定义了函数`unmerge_node`、函数`get_merged_deps`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `old_set` 等值。

### Lines 81-100 / 第 81-100 行
````python
        Combines:
        1. Direct deps (all_input_nodes) of node and its merge equivalents
        2. Extra deps of node and its merge equivalents
        """
        deps: OrderedSet[fx.Node] = OrderedSet()

        # For each node in the merge set
        for merged_node in self.merge_sets[node]:
            # Add direct dependencies from all_input_nodes
            deps.update(merged_node.all_input_nodes)
            # Add extra dependencies
            deps.update(self.extra_deps[merged_node])

        return deps

    def has_cycle(self) -> bool:
        return torch._dynamo.graph_deduplication._has_cycle(
            self.graph, self.get_all_extra_deps()
        )

````
- **EN**: Introduces function `has_cycle`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Combines`, and `deps`.
- **CN**: 这里定义了函数`has_cycle`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Combines`、`deps` 等值。

### Lines 101-120 / 第 101-120 行
````python
    def _get_all_ancestors(self, node: fx.Node) -> OrderedSet[fx.Node]:
        """Transitive ancestors through both data deps and extra deps."""
        ancestors: OrderedSet[fx.Node] = OrderedSet()
        stack: list[fx.Node] = list(node.all_input_nodes)
        stack.extend(self.extra_deps.get(node, ()))
        while stack:
            n = stack.pop()
            if n not in ancestors:
                ancestors.add(n)
                stack.extend(n.all_input_nodes)
                stack.extend(self.extra_deps.get(n, ()))
        return ancestors

    def has_path(self, source: fx.Node, target: fx.Node) -> bool:
        """Check if there's a path from source to target."""
        # we should not be checking path from node to itself
        assert self.merge_sets[source] is not self.merge_sets[target]

        # search backwards from target to source
        visited: OrderedSet[fx.Node] = OrderedSet()
````
- **EN**: Introduces function `_get_all_ancestors`, function `has_path`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ancestors`, `stack`, `n`, and `visited`.
- **CN**: 这里定义了函数`_get_all_ancestors`、函数`has_path`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `ancestors`、`stack`、`n`、`visited` 等值。

### Lines 121-140 / 第 121-140 行
````python
        queue = [target]
        visited.add(target)

        while queue:
            current = queue.pop()

            for dep in self.get_merged_deps(current):
                # Check if we reached source or its equivalent
                if dep in self.merge_sets[source]:
                    return True

                if dep in visited:
                    continue

                # We are searching from target, so this node is necessarily an ancestor
                # of target.
                # If dep is an ancestor of source, any path through dep to source would imply a cycle
                if self.node_ancestors:
                    source_set = self.merge_sets[source]
                    is_ancestor_of_source = any(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `queue`, `current`, `source_set`, and `is_ancestor_of_source`. This range continues the implementation of function `AugmentedGraphHelper.has_path`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `queue`、`current`、`source_set`、`is_ancestor_of_source` 等值。这一段延续了函数`AugmentedGraphHelper.has_path` 的具体实现。

### Lines 141-160 / 第 141-160 行
````python
                        dep in self.node_ancestors[s] for s in source_set
                    )
                    # Add to visited to avoid recomputing this check if we see dep again
                    if is_ancestor_of_source:
                        visited.add(dep)
                        continue

                visited.add(dep)
                queue.append(dep)

        return False

    def transfer_erased_node_deps(
        self, erased_to_new: dict[fx.Node, fx.Node | None]
    ) -> None:
        """
        Transfer all extra dependencies from erased nodes to their replacements, handling
        cross-dependencies between erased nodes correctly.

        Skips deps where both endpoints resolve to replacement nodes from the
````
- **EN**: Introduces function `transfer_erased_node_deps`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`transfer_erased_node_deps`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 161-180 / 第 161-180 行
````python
        same erasure batch — these are intra-bucket deps that would create
        cycles (e.g. new_start <-> new_wait within the same bucket).
        """
        erased_merge_sets: dict[fx.Node, fx.Node | None] = {}

        for replaced, new in erased_to_new.items():
            for equiv in self.merge_sets[replaced]:
                erased_merge_sets[equiv] = new

        # Transfer dependencies
        for old_node, new_node in erased_merge_sets.items():
            if new_node is None:
                # Clean up references to removed node
                for extra_use in list(self.extra_uses[old_node]):
                    updated_use = erased_merge_sets.get(extra_use, extra_use)
                    if updated_use is not None:
                        self.extra_deps[updated_use].discard(old_node)
                for extra_dep in list(self.extra_deps[old_node]):
                    updated_dep = erased_merge_sets.get(extra_dep, extra_dep)
                    if updated_dep is not None:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `erased_merge_sets`, `updated_use`, and `updated_dep`. This range continues the implementation of function `AugmentedGraphHelper.transfer_erased_node_deps`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `erased_merge_sets`、`updated_use`、`updated_dep` 等值。这一段延续了函数`AugmentedGraphHelper.transfer_erased_node_deps` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
                        self.extra_uses[updated_dep].discard(old_node)
            else:
                # Transfer dependencies FROM old_node (what old_node depended on)
                for extra_dep in self.extra_deps[old_node]:
                    updated_dep = erased_merge_sets.get(extra_dep, extra_dep)
                    if updated_dep is not None and updated_dep != new_node:
                        # Skip if reverse dep already exists (extra or data)
                        if new_node in self.extra_deps.get(
                            updated_dep, ()
                        ) or new_node in OrderedSet(updated_dep.all_input_nodes):
                            continue
                        self.extra_deps[new_node].add(updated_dep)
                        self.extra_uses[updated_dep].discard(old_node)
                        self.extra_uses[updated_dep].add(new_node)

                # Transfer dependencies TO old_node (what depended on old_node)
                for extra_use in self.extra_uses[old_node]:
                    updated_use = erased_merge_sets.get(extra_use, extra_use)
                    if updated_use is not None and updated_use != new_node:
                        # Skip if reverse dep already exists (extra or data)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `updated_dep`, and `updated_use`. This range continues the implementation of function `AugmentedGraphHelper.transfer_erased_node_deps`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`updated_dep`、`updated_use` 等值。这一段延续了函数`AugmentedGraphHelper.transfer_erased_node_deps` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python
                        if updated_use in self.extra_deps.get(
                            new_node, ()
                        ) or updated_use in OrderedSet(new_node.all_input_nodes):
                            continue
                        self.extra_deps[updated_use].discard(old_node)
                        self.extra_deps[updated_use].add(new_node)
                        self.extra_uses[new_node].add(updated_use)

        # Clean up erased nodes
        for old_node in erased_merge_sets:
            self.extra_deps[old_node].clear()
            self.extra_uses[old_node].clear()
            del self.merge_sets[old_node]

    def remove_erased_extra_deps(self) -> None:
        """Remove extra deps referencing erased nodes."""
        for node in list(self.extra_deps):
            if node._erased:
                for dep in list(self.extra_deps[node]):
                    self.remove_extra_dep(n=node, dep=dep)
````
- **EN**: Introduces function `remove_erased_extra_deps`. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`remove_erased_extra_deps`。包含分支、循环或上下文管理等控制流。

### Lines 221-240 / 第 221-240 行
````python
                continue
            for dep in list(self.extra_deps[node]):
                if dep._erased:
                    self.remove_extra_dep(n=node, dep=dep)

    def check_and_maybe_autofix_cyclic_extra_deps(
        self, *, autofix: bool = False
    ) -> None:
        """Check for and optionally remove extra deps that create cycles.

        Args:
            autofix: If True, silently remove cyclic deps.  If False (default),
                raise an error so the root cause gets investigated.
        """
        if not self.has_cycle():
            return
        removed = []
        for node in list(self.extra_deps):
            for dep in list(self.extra_deps[node]):
                ancestors = self._get_all_ancestors(dep)
````
- **EN**: Introduces function `check_and_maybe_autofix_cyclic_extra_deps`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Args`, `autofix`, `removed`, and `ancestors`.
- **CN**: 这里定义了函数`check_and_maybe_autofix_cyclic_extra_deps`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Args`、`autofix`、`removed`、`ancestors` 等值。

### Lines 241-260 / 第 241-260 行
````python
                if node in ancestors:
                    removed.append((node.name, dep.name))
                    self.remove_extra_dep(n=node, dep=dep)
        if not removed:
            return
        msg = (
            f"Overlap scheduling: detected {len(removed)} cyclic extra "
            f"dep(s): {removed}. Please report this to the overlap "
            f"scheduling developers."
        )
        log.warning(msg)
        trace_structured(
            "artifact",
            metadata_fn=lambda: {
                "name": "inductor_overlap_cyclic_extra_deps",
                "encoding": "string",
            },
            payload_fn=lambda: msg,
        )
        if not autofix:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `msg`, `metadata_fn`, and `payload_fn`. This range continues the implementation of function `AugmentedGraphHelper.check_and_maybe_autofix_cyclic_extra_deps`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `msg`、`metadata_fn`、`payload_fn` 等值。这一段延续了函数`AugmentedGraphHelper.check_and_maybe_autofix_cyclic_extra_deps` 的具体实现。

### Lines 261-276 / 第 261-276 行
````python
            raise RuntimeError(
                f"{msg}\nTo unblock, set "
                f"torch._inductor.config.aten_distributed_optimizations"
                f".overlap_scheduling_autofix_cycles = True"
            )

    def get_all_extra_deps(self) -> dict[fx.Node, OrderedSet[fx.Node]]:
        """
        Get all extra dependencies in a format suitable for topological sort.
        Returns a copy to avoid external modifications.
        """
        return {
            node: OrderedSet(deps)
            for node, deps in self.extra_deps.items()
            if deps  # Only include nodes with non-empty deps
        }
````
- **EN**: Introduces function `get_all_extra_deps`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node`.
- **CN**: 这里定义了函数`get_all_extra_deps`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `node` 等值。

## Key Concepts / 关键概念
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Primary classes: `AugmentedGraphHelper`  
  **CN**: 主要类：`AugmentedGraphHelper`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `collections`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `torch._logging`, `torch.utils._ordered_set`
