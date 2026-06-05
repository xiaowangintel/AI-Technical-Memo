# simd_kernel_features.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/codegen/simd_kernel_features.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module generates backend-specific code for lowered kernels. It defines classes such as `NodeScheduleMarker`, `DisableReduction`, `EnableReduction`, `SIMDKernelFeatures`, `MemoryEstimator`, `MemoryEstimate`, and `...+5`.
- **用途（中文）**: 该模块为下沉后的内核生成后端相关代码。其中定义了 `NodeScheduleMarker`、`DisableReduction`、`EnableReduction`、`SIMDKernelFeatures`、`MemoryEstimator`、`MemoryEstimate`、`另有5项` 等类。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
from __future__ import annotations

import collections
import dataclasses
import functools
import itertools
import typing
from typing import Any

import sympy

import torch

from ...utils._ordered_set import OrderedSet
from ...utils._sympy.functions import FloorDiv, ModularIndexing
from ...utils._sympy.symbol import make_symbol, SymT
from ..dependencies import Dep, extract_loop_body_with_args, MemoryDep
from ..runtime.hints import ReductionHint
from ..scheduler import SchedulerNode
from ..utils import cache_on_self
````
- **EN**: Imports dependencies such as `__future__`, `collections`, `dataclasses`, `functools`, `itertools`, `typing`, and `...+9` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `__future__`、`collections`、`dataclasses`、`functools`、`itertools`、`typing`、`另有9项` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。

### Lines 21-40 / 第 21-40 行
````python
from ..virtualized import V


if typing.TYPE_CHECKING:
    from collections.abc import Iterable, Sequence

    from torch._inductor.tiling_utils import CoalesceVarAnalysis


class NodeScheduleMarker:
    @staticmethod
    def only_nodes(it: Iterable[NodeScheduleEntry]) -> Iterable[SchedulerNode]:
        for item in it:
            if not (item is DisableReduction or item is EnableReduction):
                yield item  # type: ignore[misc]

    @staticmethod
    def is_reduction() -> bool:
        return False

````
- **EN**: Imports dependencies such as `..virtualized`, `collections.abc`, and `torch._inductor.tiling_utils` for the logic in this range. Introduces class `NodeScheduleMarker`, function `only_nodes`, function `is_reduction`. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Applies decorators to register behavior or alter how the following definition is constructed.
- **CN**: 这里导入了 `..virtualized`、`collections.abc`、`torch._inductor.tiling_utils` 等依赖，为后续逻辑提供基础能力。这里定义了类`NodeScheduleMarker`、函数`only_nodes`、函数`is_reduction`。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。使用装饰器来注册行为，或改变后续定义的构造方式。

### Lines 41-60 / 第 41-60 行
````python

NodeScheduleEntry = SchedulerNode | type[NodeScheduleMarker]


class DisableReduction(NodeScheduleMarker):
    """
    Marker to invoke `kernel.disable_reduction()`.  This closes a
    reduction loop and allows for pointwise ops to occur on the output
    of a reduction.
    """


class EnableReduction(NodeScheduleMarker):
    """
    Marker to end a DisableReduction block.
    """

    @staticmethod
    def filter(node_schedule: list[NodeScheduleEntry]) -> Iterable[SchedulerNode]:
        """
````
- **EN**: Introduces class `DisableReduction`, class `EnableReduction`, function `filter`. Applies decorators to register behavior or alter how the following definition is constructed. Initializes or updates values such as `NodeScheduleEntry`.
- **CN**: 这里定义了类`DisableReduction`、类`EnableReduction`、函数`filter`。使用装饰器来注册行为，或改变后续定义的构造方式。初始化或更新了 `NodeScheduleEntry` 等值。

### Lines 61-80 / 第 61-80 行
````python
        Get the nodes from node_schedule skipping those in a
        DisableReduction block.
        """
        disabled = False
        for node in node_schedule:
            if node in (EnableReduction, DisableReduction):
                # Don't tile stuff outside the main reduction loop
                disabled = node is DisableReduction
            elif disabled:
                pass
            else:
                yield node  # type: ignore[misc]


class SIMDKernelFeatures:
    """
    An ordered schedule of nodes that will become a single kernel.
    """

    def __init__(
````
- **EN**: Introduces class `SIMDKernelFeatures`, function `__init__`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `disabled`, and `else`.
- **CN**: 这里定义了类`SIMDKernelFeatures`、函数`__init__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `disabled`、`else` 等值。

### Lines 81-100 / 第 81-100 行
````python
        self,
        node_schedule: list[NodeScheduleEntry],
        numel: sympy.Expr,
        reduction_numel: sympy.Expr = sympy.S.One,
        coalesce_analysis: CoalesceVarAnalysis | None = None,
    ):
        self.node_schedule = node_schedule
        # numel excludes reduction_numel
        self.numel: sympy.Expr = V.graph.sizevars.simplify(numel)
        self.reduction_numel: sympy.Expr = V.graph.sizevars.simplify(reduction_numel)
        self._stats_cache: dict[tuple[sympy.Expr, ...], MemoryStats] = {}
        self.coalesce_analysis = coalesce_analysis

    @cache_on_self
    def is_reduction(self) -> bool:
        return self.reduction_numel != 1

    @cache_on_self
    def scheduler_nodes(self) -> Iterable[SchedulerNode]:
        return tuple(NodeScheduleMarker.only_nodes(self.node_schedule))
````
- **EN**: Introduces function `is_reduction`, function `scheduler_nodes`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里定义了函数`is_reduction`、函数`scheduler_nodes`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。

### Lines 101-120 / 第 101-120 行
````python

    def reduction_nodes(self) -> list[SchedulerNode]:
        return [n for n in self.scheduler_nodes() if n.is_reduction()]

    @cache_on_self
    def buf_accesses(self) -> dict[str, list[Dep]]:
        """only needed for config.benchmark_kernel"""
        buf_accesses = collections.defaultdict(list)
        for node in self.scheduler_nodes():
            for access in node.read_writes.reads | node.read_writes.writes:
                buf_accesses[access.name].append(access)
        return buf_accesses

    @cache_on_self
    def op_counts(self) -> collections.Counter[str]:
        counts: collections.Counter[str] = collections.Counter()
        for node in self.scheduler_nodes():
            counts.update(node._body.op_counts)
        return counts

````
- **EN**: Introduces function `reduction_nodes`, function `buf_accesses`, function `op_counts`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`reduction_nodes`、函数`buf_accesses`、函数`op_counts`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 121-140 / 第 121-140 行
````python
    def contains_op(self, op_name: str) -> bool:
        """True if V.ops.{op_name} is used in node_schedule"""
        return bool(self.op_counts().get(op_name))

    def get_mutations(self) -> OrderedSet[str]:
        mutations: OrderedSet[str] = OrderedSet()
        for node in self.scheduler_nodes():
            for buf in node.get_outputs():
                mutations.update(buf.get_mutations())
        return mutations

    @cache_on_self
    def select_index_dtype(self) -> torch.dtype:
        # Gather all used buffer names
        buffer_names: OrderedSet[str] = OrderedSet()
        for node in self.scheduler_nodes():
            buffer_names.update(node.get_buffer_names())
            buffer_names.update(node.used_buffer_names())
        buffers = [V.graph.get_buffer(name) for name in buffer_names]

````
- **EN**: Introduces function `contains_op`, function `get_mutations`, function `select_index_dtype`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`contains_op`、函数`get_mutations`、函数`select_index_dtype`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 141-160 / 第 141-160 行
````python
        # In theory we can separately check xnumel and rnumel are <= int_max
        # but some indexers do use the full linear index so we need to be
        # conservative here.
        total_numel = self.numel * self.reduction_numel

        from .simd import SIMDScheduling

        if SIMDScheduling.can_use_32bit_indexing(total_numel, buffers):
            return torch.int32
        return torch.int64

    def get_reduction_hint(
        self, tiling_scores: dict[str, int] | None = None
    ) -> ReductionHint:
        reductions = self.reduction_nodes()
        if len(reductions) > 0:
            hints = [self.reduction_hint(n) for n in reductions]
            if hints.count(hints[0]) == len(hints):
                reduction_hint_val = hints[0]
            else:
````
- **EN**: Imports dependencies such as `.simd` for the logic in this range. Introduces function `get_reduction_hint`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `.simd` 等依赖，为后续逻辑提供基础能力。这里定义了函数`get_reduction_hint`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 161-180 / 第 161-180 行
````python
                reduction_hint_val = ReductionHint.DEFAULT

            if (
                reduction_hint_val == ReductionHint.INNER
                and self.has_non_contiguous_pw_in_reduction_kernel()
            ):
                reduction_hint_val = ReductionHint.DEFAULT

            # Upgrade DEFAULT to INNER for inner reductions based on tiling scores
            if (
                reduction_hint_val == ReductionHint.DEFAULT
                and tiling_scores is not None
                and "x" in tiling_scores
                and "r0_" in tiling_scores
            ):
                # If reduction dimension has much better coalescing than non-reduction dimensions,
                # this is an inner reduction
                from ..codegen.triton import INNER_REDUCTION_RATIO_THRESHOLD

                r_coalesce_ratio = tiling_scores["r0_"] / max(tiling_scores["x"], 1)
````
- **EN**: Imports dependencies such as `..codegen.triton` for the logic in this range. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `reduction_hint_val`, and `r_coalesce_ratio`.
- **CN**: 这里导入了 `..codegen.triton` 等依赖，为后续逻辑提供基础能力。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `reduction_hint_val`、`r_coalesce_ratio` 等值。

### Lines 181-200 / 第 181-200 行
````python
                contiguous_red = r_coalesce_ratio >= INNER_REDUCTION_RATIO_THRESHOLD
                if contiguous_red:
                    reduction_hint_val = ReductionHint.INNER
        else:
            reduction_hint_val = ReductionHint.DEFAULT
        return reduction_hint_val

    @cache_on_self
    def buffer_read_counts(self) -> dict[str, int]:
        """Counts how many times each buffer is read within the kernel"""
        read_counts: dict[str, int] = collections.defaultdict(int)

        for node in self.scheduler_nodes():
            # node.read_writes.reads contains MemoryDep objects for each read
            for read_dep in node.read_writes.reads:
                read_counts[read_dep.name] += 1

        return dict(read_counts)  # Convert defaultdict to regular dict

    def has_non_contiguous_pw_in_reduction_kernel(self) -> bool:
````
- **EN**: Introduces function `buffer_read_counts`, function `has_non_contiguous_pw_in_reduction_kernel`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`buffer_read_counts`、函数`has_non_contiguous_pw_in_reduction_kernel`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 201-220 / 第 201-220 行
````python
        pointwise_nodes = [
            n
            for n in self.scheduler_nodes()
            if not n.is_reduction()
            and n.group[1][0] == self.numel * self.reduction_numel
        ]
        for node in pointwise_nodes:
            # An index can be an integer when loading a random seed.
            if not all(
                not isinstance(dep, MemoryDep)
                or dep.is_contiguous()
                or isinstance(dep.index, (sympy.Integer, int))
                or dep.stride1_for_last_dim()
                for dep in itertools.chain(
                    node.read_writes.reads, node.read_writes.writes
                )
            ):
                return True
        return False

````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pointwise_nodes`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `pointwise_nodes` 等值。

### Lines 221-240 / 第 221-240 行
````python
    @staticmethod
    def reduction_hint(node: Any) -> ReductionHint:
        assert node.is_reduction()
        if node.node.data.reduction_hint != ReductionHint.INNER and all(
            dep.is_contiguous()
            for dep in itertools.chain(node.read_writes.reads, node.read_writes.writes)
        ):
            return ReductionHint.INNER
        else:
            return node.node.data.reduction_hint

    def memory_stats(
        self, groups_dict: dict[str, sympy.Expr] | None = None
    ) -> MemoryStats:
        """Analysis to generate features that can be used in heuristics"""
        if groups_dict is None:
            groups = (self.numel, self.reduction_numel)
        elif groups_dict.keys() == OrderedSet(["x", "r0_"]):
            groups = (groups_dict["x"], groups_dict["r0_"])
        else:
````
- **EN**: Introduces function `reduction_hint`, function `memory_stats`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`reduction_hint`、函数`memory_stats`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 241-260 / 第 241-260 行
````python
            raise NotImplementedError(f"groups_dict={groups_dict!r}")
        result = self._stats_cache.get(groups)
        if result is None:
            self._stats_cache[groups] = result = MemoryStats.compute(
                MemoryEstimator(self, groups)
            )
        return result


class MemoryEstimator:
    """
    Estimate various properties of the kernel for use in heuristics.
    We simulate the memory effects of CSE/buffer elimination in codegen.
    """

    kernel_sizes: tuple[sympy.Expr, ...]
    outside_loop: MemoryEstimate
    loops: list[MemoryEstimate]
    persistent: MemoryEstimate
    symbols: list[sympy.Symbol]
````
- **EN**: Introduces class `MemoryEstimator`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`MemoryEstimator`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 261-280 / 第 261-280 行
````python

    def __init__(self, features: SIMDKernelFeatures, groups: Sequence[sympy.Expr]):
        self.features = features
        self.inside_reduction = features.is_reduction()
        self.store_buffer_names: OrderedSet[str] = OrderedSet()
        self.must_keep_buffers: OrderedSet[str] = OrderedSet()
        self.num_reductions_dims = 1
        self.groups = groups
        self.symbols = [make_symbol(SymT.INDEX, i) for i in range(len(groups))]
        # We are doing two estimates simultaneously:
        # 1) the first is a for a non-persistent (aka looped) reduction, using self.outside_loop/self.loops
        # we add an item to loops each corresponding to each reduction loop in the kernel
        # outside_loop is only used for broadcasting or point-wise ops that don't use the reduction dimension
        # 2) the second is for a persistent kernel, using self.persistent
        # persistent kernels don't have loops, so we only have one MemoryEstimate()
        # for point-wise ops the two estimates will be the same, they matter for reductions only
        self.outside_loop = MemoryEstimate()
        self.loops = [MemoryEstimate()]
        self.persistent = MemoryEstimate()
        self.simulate_codegen()
````
- **EN**: Introduces function `__init__`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里定义了函数`__init__`。借助符号表达式来推理索引、形状或代数性质。

### Lines 281-300 / 第 281-300 行
````python
        self.remove_kernel_local()

    def simulate_codegen(self) -> None:
        from .simd import SIMDKernel

        kernel_size_outside_loop = (*self.groups[:-1], sympy.S.One)
        kernel_size_inside_loop = tuple(self.groups)
        self.kernel_sizes = kernel_size_inside_loop

        for node in self.features.node_schedule:
            if node is DisableReduction:
                self.inside_reduction = False
                self.kernel_sizes = kernel_size_outside_loop
                continue
            elif node is EnableReduction:
                self.inside_reduction = True
                self.kernel_sizes = kernel_size_inside_loop
                self.loops.append(MemoryEstimate())
                continue
            assert isinstance(node, SchedulerNode)
````
- **EN**: Imports dependencies such as `.simd` for the logic in this range. Introduces function `simulate_codegen`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `.simd` 等依赖，为后续逻辑提供基础能力。这里定义了函数`simulate_codegen`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 301-320 / 第 301-320 行
````python
            rw = extract_loop_body_with_args(
                node._body,
                SIMDKernel.map_kernel_groups_to_node_sizes(
                    self.kernel_sizes, node.get_ranges(), self.set_ranges
                ),
                dict(zip(self.symbols, self.kernel_sizes)),
            )

            for dep in rw._reads:
                if not isinstance(dep, MemoryDep):
                    continue
                dep = dep.simplify_with_ranges()
                if not self.persistent.writes.get(dep.name):  # cache miss?
                    self.persistent.reads[dep.name].add(dep)
                # the cache behavior of looped kernels is more complex than the persistent case above
                # some operations are lifted outside the loop (if they don't use the reduction dimension)
                # other operations are inside the loop, and can only be reused within the same loop
                if not (
                    self.outside_loop.writes.get(dep.name)
                    or self.loops[-1].writes.get(dep.name)
````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `rw`, and `dep`. This range continues the implementation of function `MemoryEstimator.simulate_codegen`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。初始化或更新了 `rw`、`dep` 等值。这一段延续了函数`MemoryEstimator.simulate_codegen` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python
                ):
                    self.scope(dep).reads[dep.name].add(dep)
                    if dep.name in self.store_buffer_names and self.loops[-1].reads.get(
                        dep.name
                    ):
                        self.must_keep_buffers.add(dep.name)

            for dep in rw._writes:
                if not isinstance(dep, MemoryDep):
                    continue
                dep = dep.simplify_with_ranges()
                self.store_buffer_names.add(dep.name)
                self.persistent.writes[dep.name].add(dep)
                self.scope(dep).writes[dep.name].add(dep)

    def remove_kernel_local(self) -> None:
        # Remove any kernel-local buffers
        fused_node_names = OrderedSet(
            [n.get_name() for n in self.features.scheduler_nodes()]
        )
````
- **EN**: Introduces function `remove_kernel_local`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dep`, and `fused_node_names`.
- **CN**: 这里定义了函数`remove_kernel_local`。包含分支、循环或上下文管理等控制流。初始化或更新了 `dep`、`fused_node_names` 等值。

### Lines 341-360 / 第 341-360 行
````python
        for name in self.store_buffer_names:
            if not self.persistent.reads.get(
                name
            ) and V.graph.scheduler.can_buffer_be_removed_through_fusion(
                name, fused_node_names
            ):
                self.persistent.remove(name)
                if name not in self.must_keep_buffers:
                    # we can also remove this from the looped kernel
                    self.outside_loop.remove(name)
                    for loop in self.loops:
                        loop.remove(name)

        if not self.loops[-1]:
            self.loops.pop()  # for pointwise ops

    def scope(self, dep: MemoryDep) -> MemoryEstimate:
        """Determine how a read/write should be categorized"""
        if self.inside_reduction and (
            self.has_reduction_var(dep.index) or dep.is_indirect()
````
- **EN**: Introduces function `scope`. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`scope`。包含分支、循环或上下文管理等控制流。

### Lines 361-380 / 第 361-380 行
````python
        ):
            return self.loops[-1]
        return self.outside_loop

    def has_reduction_var(self, index: sympy.Expr) -> bool:
        for sym in self.symbols[-self.num_reductions_dims :]:
            if isinstance(sym, sympy.Symbol) and sym in index.free_symbols:
                return True
        return False

    def set_ranges(self, *lengths: list[list[sympy.Expr]]) -> list[list[sympy.Expr]]:
        assert len(self.kernel_sizes) == len(lengths)
        return [
            self.make_flat_range(sym, numel, length)
            for sym, numel, length in zip(self.symbols, self.kernel_sizes, lengths)
        ]

    @staticmethod
    def make_flat_range(
        sym: sympy.Symbol, numel: sympy.Expr, lengths: list[sympy.Expr]
````
- **EN**: Introduces function `has_reduction_var`, function `set_ranges`, function `make_flat_range`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`has_reduction_var`、函数`set_ranges`、函数`make_flat_range`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 381-400 / 第 381-400 行
````python
    ) -> list[sympy.Expr]:
        if len(lengths) == 1 and numel == lengths[0]:
            return [sym]
        divisor = sympy.S.One
        itervars = []
        for length in reversed(lengths):
            if V.graph.sizevars.statically_known_equals(divisor * length, numel):
                expr = FloorDiv(sym, divisor)
            else:
                expr = ModularIndexing(sym, divisor, length)
            itervars.append(expr)
            divisor = divisor * length
        return [*reversed(itervars)]


@dataclasses.dataclass
class MemoryEstimate:
    """Tracks the memory usage of a single loop in the generated kernel"""

    reads: dict[str, OrderedSet[MemoryDep]] = dataclasses.field(
````
- **EN**: Introduces class `MemoryEstimate`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里定义了类`MemoryEstimate`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。借助符号表达式来推理索引、形状或代数性质。

### Lines 401-420 / 第 401-420 行
````python
        default_factory=functools.partial(collections.defaultdict, OrderedSet)
    )
    writes: dict[str, OrderedSet[MemoryDep]] = dataclasses.field(
        default_factory=functools.partial(collections.defaultdict, OrderedSet)
    )

    def remove(self, name: str) -> None:
        self.reads.pop(name, None)
        self.writes.pop(name, None)

    def __bool__(self) -> bool:
        return bool(self.reads or self.writes)

    def __repr__(self) -> str:
        return f"""MemoryEstimate(
            reads={[*itertools.chain.from_iterable(self.reads.values())]!r},
            writes={[*itertools.chain.from_iterable(self.writes.values())]!r}
        )"""


````
- **EN**: Introduces function `remove`, function `__bool__`, function `__repr__`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `default_factory`, `writes`, and `reads`.
- **CN**: 这里定义了函数`remove`、函数`__bool__`、函数`__repr__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `default_factory`、`writes`、`reads` 等值。

### Lines 421-440 / 第 421-440 行
````python
@dataclasses.dataclass
class StatsForDim:
    """Memory usage stats for a block dimension in the generated kernel (different from user dimensions)"""

    # the number of load/store ops
    count_per_thread_contiguous: int = 0
    count_per_thread_broadcast: int = 0
    count_per_thread_non_contiguous: int = 0  # excludes broadcast

    # total bytes in each load/store op for a single element
    bytes_per_thread_contiguous: int = 0
    bytes_per_thread_broadcast: int = 0
    bytes_per_thread_non_contiguous: int = 0  # excludes broadcast

    # total bytes read by entire kernel
    bytes_contiguous_or_broadcast: sympy.Expr = sympy.S.Zero
    bytes_non_contiguous: sympy.Expr = sympy.S.Zero

    def __add__(self, other: typing.Self) -> StatsForDim:
        return StatsForDim(
````
- **EN**: Introduces class `StatsForDim`, function `__add__`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里定义了类`StatsForDim`、函数`__add__`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。借助符号表达式来推理索引、形状或代数性质。

### Lines 441-460 / 第 441-460 行
````python
            count_per_thread_contiguous=self.count_per_thread_contiguous
            + other.count_per_thread_contiguous,
            count_per_thread_broadcast=self.count_per_thread_broadcast
            + other.count_per_thread_broadcast,
            count_per_thread_non_contiguous=self.count_per_thread_non_contiguous
            + other.count_per_thread_non_contiguous,
            bytes_per_thread_contiguous=self.bytes_per_thread_contiguous
            + other.bytes_per_thread_contiguous,
            bytes_per_thread_broadcast=self.bytes_per_thread_broadcast
            + other.bytes_per_thread_broadcast,
            bytes_per_thread_non_contiguous=self.bytes_per_thread_non_contiguous
            + other.bytes_per_thread_non_contiguous,
            bytes_contiguous_or_broadcast=self.bytes_contiguous_or_broadcast
            + other.bytes_contiguous_or_broadcast,
            bytes_non_contiguous=self.bytes_non_contiguous + other.bytes_non_contiguous,
        )

    @property
    def count_per_thread(self) -> int:
        return (
````
- **EN**: Introduces function `count_per_thread`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `count_per_thread_contiguous`, `count_per_thread_broadcast`, `count_per_thread_non_contiguous`, `bytes_per_thread_contiguous`, `bytes_per_thread_broadcast`, `bytes_per_thread_non_contiguous`, and `...+2`.
- **CN**: 这里定义了函数`count_per_thread`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `count_per_thread_contiguous`、`count_per_thread_broadcast`、`count_per_thread_non_contiguous`、`bytes_per_thread_contiguous`、`bytes_per_thread_broadcast`、`bytes_per_thread_non_contiguous`、`另有2项` 等值。

### Lines 461-480 / 第 461-480 行
````python
            self.count_per_thread_contiguous
            + self.count_per_thread_broadcast
            + self.count_per_thread_non_contiguous
        )

    @property
    def bytes_per_thread(self) -> int:
        return (
            self.bytes_per_thread_contiguous
            + self.bytes_per_thread_broadcast
            + self.bytes_per_thread_non_contiguous
        )

    @property
    def bytes(self) -> sympy.Expr:
        return self.bytes_contiguous_or_broadcast + self.bytes_non_contiguous

    @property
    def contiguous_score(self) -> float:
        return 1.0 - self.count_per_thread_non_contiguous / max(
````
- **EN**: Introduces function `bytes_per_thread`, function `bytes`, function `contiguous_score`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`bytes_per_thread`、函数`bytes`、函数`contiguous_score`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 481-500 / 第 481-500 行
````python
            self.count_per_thread, 1
        )


@dataclasses.dataclass
class StatsForLoop:
    """Memory usage stats for single loop in the generated kernel"""

    # load/store ops
    count_per_thread: int = 0
    bytes_per_thread: int = 0

    def __add__(self, other: typing.Self) -> StatsForLoop:
        return StatsForLoop(
            count_per_thread=self.count_per_thread + other.count_per_thread,
            bytes_per_thread=self.bytes_per_thread + other.bytes_per_thread,
        )


@dataclasses.dataclass
````
- **EN**: Introduces class `StatsForLoop`, function `__add__`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`StatsForLoop`、函数`__add__`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 501-520 / 第 501-520 行
````python
class StatsForReadsOrWrites:
    """Memory usage stats that are collected for reads/writes/both"""

    dim: list[StatsForDim]
    loop: list[StatsForLoop]
    # total bytes contiguous in any dimension
    bytes_contiguous_or_broadcast: sympy.Expr = sympy.S.Zero
    bytes_non_contiguous: sympy.Expr = sympy.S.Zero

    def __add__(self, other: typing.Self) -> StatsForReadsOrWrites:
        assert len(self.dim) == len(other.dim)
        assert len(self.loop) == len(other.loop)
        return StatsForReadsOrWrites(
            dim=[a + b for a, b in zip(self.dim, other.dim)],
            loop=[a + b for a, b in zip(self.loop, other.loop)],
            bytes_contiguous_or_broadcast=self.bytes_contiguous_or_broadcast
            + self.bytes_contiguous_or_broadcast,
            bytes_non_contiguous=self.bytes_non_contiguous + other.bytes_non_contiguous,
        )

````
- **EN**: Introduces class `StatsForReadsOrWrites`, function `__add__`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `dim`, `loop`, `bytes_contiguous_or_broadcast`, and `bytes_non_contiguous`.
- **CN**: 这里定义了类`StatsForReadsOrWrites`、函数`__add__`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `dim`、`loop`、`bytes_contiguous_or_broadcast`、`bytes_non_contiguous` 等值。

### Lines 521-540 / 第 521-540 行
````python
    @property
    def count_per_thread(self) -> int:
        return self.dim[0].count_per_thread

    @property
    def bytes_per_thread(self) -> int:
        return self.dim[0].bytes_per_thread

    @property
    def bytes(self) -> sympy.Expr:
        return self.bytes_contiguous_or_broadcast + self.bytes_non_contiguous

    @classmethod
    def compute(
        cls,
        loop_deps: list[dict[str, OrderedSet[MemoryDep]]],
        index_symbols: list[sympy.Symbol],
    ) -> typing.Self:
        ndim = len(index_symbols)
        result = cls(dim := [StatsForDim() for _ in range(ndim)], [])
````
- **EN**: Introduces function `count_per_thread`, function `bytes_per_thread`, function `bytes`, function `compute`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`count_per_thread`、函数`bytes_per_thread`、函数`bytes`、函数`compute`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 541-560 / 第 541-560 行
````python
        for dep_group in loop_deps:
            result.loop.append(loop_stats := StatsForLoop())
            for name, deps in dep_group.items():
                assert deps
                contiguous_or_broadcast = [True] * ndim
                numel = sympy.S.Zero
                itemsize = V.graph.get_dtype(name).itemsize
                loop_stats.count_per_thread += len(deps)
                loop_stats.bytes_per_thread += itemsize * len(deps)
                for dep in deps:
                    strides: list[sympy.Expr] = V.graph.sizevars.stride_vars(
                        dep.index, index_symbols
                    )
                    for i in range(ndim):
                        if V.graph.sizevars.statically_known_equals(strides[i], 1):
                            dim[i].count_per_thread_contiguous += 1
                            dim[i].bytes_per_thread_contiguous += itemsize
                        elif (
                            V.graph.sizevars.statically_known_equals(strides[i], 0)
                            and not dep.is_indirect()
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `contiguous_or_broadcast`, `numel`, `itemsize`, and `strides`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `contiguous_or_broadcast`、`numel`、`itemsize`、`strides` 等值。

### Lines 561-580 / 第 561-580 行
````python
                        ):
                            dim[i].count_per_thread_broadcast += 1
                            dim[i].bytes_per_thread_broadcast += itemsize
                        else:
                            dim[i].count_per_thread_non_contiguous += 1
                            dim[i].bytes_per_thread_non_contiguous += itemsize
                            contiguous_or_broadcast[i] = False
                    numel += dep.get_numel()
                if len(deps) > 1:
                    # can't read more elements than exist in the buffer
                    numel = sympy.Min(numel, V.graph.get_numel(name))
                nbytes = numel * itemsize
                for i in range(ndim):
                    if contiguous_or_broadcast[i]:
                        dim[i].bytes_contiguous_or_broadcast += nbytes
                    else:
                        dim[i].bytes_non_contiguous += nbytes
                if any(contiguous_or_broadcast):
                    result.bytes_contiguous_or_broadcast += nbytes
                else:
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `numel`, and `nbytes`. This range continues the implementation of function `StatsForReadsOrWrites.compute`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`numel`、`nbytes` 等值。这一段延续了函数`StatsForReadsOrWrites.compute` 的具体实现。

### Lines 581-600 / 第 581-600 行
````python
                    result.bytes_non_contiguous += nbytes
        if len(result.loop) > 1:
            # the first loop represent the "outside of the loop" compute which could be long lived
            result.loop = [result.loop[0] + x for x in result.loop[1:]]
        return result


@dataclasses.dataclass
class StatsForKernelType:
    """Memory usage stats that are collected for both persistent and looped kernels"""

    reads: StatsForReadsOrWrites
    writes: StatsForReadsOrWrites
    memory: StatsForReadsOrWrites

    @classmethod
    def compute(
        cls, loops: list[MemoryEstimate], estimator: MemoryEstimator
    ) -> typing.Self:
        reads = StatsForReadsOrWrites.compute(
````
- **EN**: Introduces class `StatsForKernelType`, function `compute`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`StatsForKernelType`、函数`compute`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 601-620 / 第 601-620 行
````python
            [loop.reads for loop in loops], estimator.symbols
        )
        writes = StatsForReadsOrWrites.compute(
            [loop.writes for loop in loops], estimator.symbols
        )
        return cls(
            reads=reads,
            writes=writes,
            memory=reads + writes,
        )


@dataclasses.dataclass
class MemoryStats:
    """Memory usage stats collected for each generated kernel"""

    persistent: StatsForKernelType
    looped: StatsForKernelType

    def get(self, persistent: bool) -> StatsForKernelType:
````
- **EN**: Introduces class `MemoryStats`, function `get`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`MemoryStats`、函数`get`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 621-637 / 第 621-637 行
````python
        return self.persistent if persistent else self.looped

    @classmethod
    def compute(cls, estimator: MemoryEstimator) -> typing.Self:
        persistent = StatsForKernelType.compute([estimator.persistent], estimator)
        if len(estimator.loops) == 1 and not (
            estimator.outside_loop and estimator.loops[0]
        ):
            looped = persistent  # loops/persistent is the same in this common case
        else:
            looped = StatsForKernelType.compute(
                [estimator.outside_loop, *estimator.loops], estimator
            )
        return cls(
            persistent=persistent,
            looped=looped,
        )
````
- **EN**: Introduces function `compute`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`compute`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

## Key Concepts / 关键概念
- **EN**: Generates backend-specific code for lowered kernels  
  **CN**: 为下沉后的内核生成后端相关代码
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `collections`, `dataclasses`, `functools`, `itertools`, `typing`, `collections.abc`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `...utils._ordered_set`, `...utils._sympy.functions`, `...utils._sympy.symbol`, `..dependencies`, `..runtime.hints`, `..scheduler`, `..utils`, `..virtualized`, `torch._inductor.tiling_utils`, `.simd`, `..codegen.triton`
