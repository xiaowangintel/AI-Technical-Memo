# metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/metrics.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `CppOuterLoopFusedCount`, `CachedMetricsDeltas`, `CachedMetricsHelper`, and `MetricTable`. It exposes functions such as `reset`, `get_metric_fields`, `_parse_kernel_fn_code`, `_parse_kernel_line_of_code`, `_parse_size_hints`, `_parse_reduction_hint`, and `...+12`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `CppOuterLoopFusedCount`、`CachedMetricsDeltas`、`CachedMetricsHelper`、`MetricTable` 等类。同时提供 `reset`、`get_metric_fields`、`_parse_kernel_fn_code`、`_parse_kernel_line_of_code`、`_parse_size_hints`、`_parse_reduction_hint`、`另有12项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
from __future__ import annotations

import csv
import dataclasses
import inspect
import os
import re
from dataclasses import dataclass
from functools import lru_cache
from typing import TYPE_CHECKING

from torch._inductor import config
from torch._inductor.utils import get_benchmark_name
from torch.utils._ordered_set import OrderedSet


# Prevent circular import
if TYPE_CHECKING:
    from collections.abc import Callable

````
- **EN**: Imports dependencies such as `__future__`, `csv`, `dataclasses`, `inspect`, `os`, `re`, and `...+6` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `__future__`、`csv`、`dataclasses`、`inspect`、`os`、`re`、`另有6项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含分支、循环或上下文管理等控制流。

### Lines 21-40 / 第 21-40 行
````python
    from torch._inductor.runtime.triton_compat import Config
    from torch._inductor.scheduler import BaseSchedulerNode

# counter for tracking how many kernels have been generated
generated_kernel_count = 0
generated_cpp_vec_kernel_count = 0
num_bytes_accessed = 0
nodes_num_elem: list[
    tuple[
        BaseSchedulerNode,
        int,
    ]
] = []
node_runtimes: list[tuple[BaseSchedulerNode, float]] = []

# counters for tracking fusions
ir_nodes_pre_fusion = 0

# counters for tracking to_dtype inserted
cpp_to_dtype_count = 0
````
- **EN**: Imports dependencies such as `torch._inductor.runtime.triton_compat`, and `torch._inductor.scheduler` for the logic in this range. Initializes or updates values such as `generated_kernel_count`, `generated_cpp_vec_kernel_count`, `num_bytes_accessed`, `nodes_num_elem`, `node_runtimes`, `ir_nodes_pre_fusion`, and `...+1`.
- **CN**: 这里导入了 `torch._inductor.runtime.triton_compat`、`torch._inductor.scheduler` 等依赖，为后续逻辑提供基础能力。初始化或更新了 `generated_kernel_count`、`generated_cpp_vec_kernel_count`、`num_bytes_accessed`、`nodes_num_elem`、`node_runtimes`、`ir_nodes_pre_fusion`、`另有1项` 等值。

### Lines 41-60 / 第 41-60 行
````python


@dataclasses.dataclass
class CppOuterLoopFusedCount:
    inner_kernel_number: int
    local_buffer_number: int = 0


# The length counts the number of outer loop fusions.
cpp_outer_loop_fused_inner_counts: list[CppOuterLoopFusedCount] = []

num_comprehensive_padding = 0
num_matches_for_scatter_upon_const_tensor = 0

num_loop_reordering = 0
num_auto_chunking: int = 0

# counter for parallel reduction.
parallel_reduction_count = 0

````
- **EN**: Introduces class `CppOuterLoopFusedCount`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Initializes or updates values such as `inner_kernel_number`, `local_buffer_number`, `cpp_outer_loop_fused_inner_counts`, `num_comprehensive_padding`, `num_matches_for_scatter_upon_const_tensor`, `num_loop_reordering`, and `...+2`.
- **CN**: 这里定义了类`CppOuterLoopFusedCount`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。初始化或更新了 `inner_kernel_number`、`local_buffer_number`、`cpp_outer_loop_fused_inner_counts`、`num_comprehensive_padding`、`num_matches_for_scatter_upon_const_tensor`、`num_loop_reordering`、`另有2项` 等值。

### Lines 61-80 / 第 61-80 行
````python
codegen_mix_order_reduction = 0


# reset all counters
def reset() -> None:
    global generated_kernel_count
    global generated_cpp_vec_kernel_count
    global num_bytes_accessed, nodes_num_elem
    global ir_nodes_pre_fusion
    global cpp_to_dtype_count
    global cpp_outer_loop_fused_inner_counts
    global num_comprehensive_padding
    global num_matches_for_scatter_upon_const_tensor
    global num_loop_reordering
    global parallel_reduction_count
    global codegen_mix_order_reduction
    global num_auto_chunking

    generated_kernel_count = 0
    generated_cpp_vec_kernel_count = 0
````
- **EN**: Introduces function `reset`. Initializes or updates values such as `codegen_mix_order_reduction`, `generated_kernel_count`, and `generated_cpp_vec_kernel_count`.
- **CN**: 这里定义了函数`reset`。初始化或更新了 `codegen_mix_order_reduction`、`generated_kernel_count`、`generated_cpp_vec_kernel_count` 等值。

### Lines 81-100 / 第 81-100 行
````python
    num_bytes_accessed = 0
    nodes_num_elem.clear()
    node_runtimes.clear()
    ir_nodes_pre_fusion = 0
    cpp_to_dtype_count = 0
    cpp_outer_loop_fused_inner_counts.clear()
    num_comprehensive_padding = 0
    num_matches_for_scatter_upon_const_tensor = 0
    num_loop_reordering = 0
    parallel_reduction_count = 0
    codegen_mix_order_reduction = 0
    num_auto_chunking = 0


@dataclass
class CachedMetricsDeltas:
    """
    The subset of metrics we want update across cache hits, e.g., the
    FxGraphCache.
    """
````
- **EN**: Introduces class `CachedMetricsDeltas`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里定义了类`CachedMetricsDeltas`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 101-120 / 第 101-120 行
````python

    generated_kernel_count: int
    generated_cpp_vec_kernel_count: int
    ir_nodes_pre_fusion: int
    cpp_to_dtype_count: int
    num_bytes_accessed: int
    num_matches_for_scatter_upon_const_tensor: int


def get_metric_fields() -> list[str]:
    return [field.name for field in dataclasses.fields(CachedMetricsDeltas)]


class CachedMetricsHelper:
    """
    A helper class to help calculate and apply counter deltas for those
    metrics we want to save with cache entries (e.g., FxGraphCache) and
    apply on a cache hit.
    """

````
- **EN**: Introduces function `get_metric_fields`, class `CachedMetricsHelper`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `generated_kernel_count`, `generated_cpp_vec_kernel_count`, `ir_nodes_pre_fusion`, `cpp_to_dtype_count`, `num_bytes_accessed`, and `num_matches_for_scatter_upon_const_tensor`.
- **CN**: 这里定义了函数`get_metric_fields`、类`CachedMetricsHelper`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `generated_kernel_count`、`generated_cpp_vec_kernel_count`、`ir_nodes_pre_fusion`、`cpp_to_dtype_count`、`num_bytes_accessed`、`num_matches_for_scatter_upon_const_tensor` 等值。

### Lines 121-140 / 第 121-140 行
````python
    def __init__(self) -> None:
        self.cached_metrics = {}
        for metric in get_metric_fields():
            self.cached_metrics[metric] = globals()[metric]

    def get_deltas(self) -> CachedMetricsDeltas:
        delta_metrics = {}
        for metric in get_metric_fields():
            delta_metrics[metric] = globals()[metric] - self.cached_metrics[metric]

        return CachedMetricsDeltas(**delta_metrics)

    @staticmethod
    def apply_deltas(delta: CachedMetricsDeltas) -> None:
        for metric in get_metric_fields():
            globals()[metric] += getattr(delta, metric)


REGISTERED_METRIC_TABLES: dict[str, MetricTable] = {}

````
- **EN**: Introduces function `__init__`, function `get_deltas`, function `apply_deltas`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`__init__`、函数`get_deltas`、函数`apply_deltas`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 141-160 / 第 141-160 行
````python

@dataclass
class MetricTable:
    table_name: str
    column_names: list[str]

    num_rows_added: int = 0

    def add_row(self, row_fn: Callable[[], dict[str, str | float | None]]) -> None:
        if self.table_name not in enabled_metric_tables():
            return

        row_dict = row_fn()
        assert len(self.column_names) == len(row_dict), (
            f"{len(self.column_names)} v.s. {len(row_dict)}"
        )
        assert OrderedSet(self.column_names) == OrderedSet(row_dict.keys()), (
            f"{OrderedSet(self.column_names)} v.s. {OrderedSet(row_dict.keys())}"
        )

````
- **EN**: Introduces class `MetricTable`, function `add_row`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`MetricTable`、函数`add_row`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 161-180 / 第 161-180 行
````python
        bn = get_benchmark_name()
        # assert bn is not None
        row = [bn] + [row_dict[column_name] for column_name in self.column_names]
        assert all(isinstance(i, (str, float, type(None))) for i in row)
        self._write_row(row)

    def output_filename(self) -> str:
        return f"metric_table_{self.table_name}.csv"

    def write_header(self) -> None:
        filename = self.output_filename()
        with open(filename, "w") as fd:
            writer = csv.writer(fd, lineterminator="\n")
            writer.writerow(["model_name"] + self.column_names)

    def _write_row(self, row: list[str | float | None]) -> None:
        filename = self.output_filename()
        if self.num_rows_added == 0 and not os.path.exists(filename):
            self.write_header()

````
- **EN**: Introduces function `output_filename`, function `write_header`, function `_write_row`. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`output_filename`、函数`write_header`、函数`_write_row`。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 181-200 / 第 181-200 行
````python
        self.num_rows_added += 1

        for idx, orig_val in enumerate(row):
            if isinstance(orig_val, float):
                new_val = f"{orig_val:.6f}"
            elif orig_val is None:
                new_val = ""
            else:
                new_val = orig_val
            row[idx] = new_val

        with open(filename, "a") as fd:
            writer = csv.writer(fd, lineterminator="\n")
            writer.writerow(row)

    @staticmethod
    def register_table(name: str, column_names: list[str]) -> None:
        table = MetricTable(name, column_names)
        REGISTERED_METRIC_TABLES[name] = table

````
- **EN**: Introduces function `register_table`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`register_table`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 201-220 / 第 201-220 行
````python

MetricTable.register_table(
    "slow_fusion",
    [
        "kernel1_path",
        "kernel1_latency",
        "kernel2_path",
        "kernel2_latency",
        "fused_kernel_path",
        "fused_kernel_latency",
        "slow_down_ratio",
    ],
)

# track the fusion statistics for each graph
MetricTable.register_table(
    "graph_stats",
    [
        "graph_id",
        "num_nodes_before_fusion",
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 221-240 / 第 221-240 行
````python
        "num_nodes_after_fusion",
    ],
)

# track the perf difference between persistent reduction and non-persistent
# reductions
MetricTable.register_table(
    "persistent_red_perf",
    [
        "kernel0_path",
        "kernel1_path",
        "kernel2_path",
        "kernel3_path",
        "kernel0_latency",
        "kernel1_latency",
        "kernel2_latency",
        "kernel3_latency",
        "size_hints",
        "reduction_hint",
    ],
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 241-260 / 第 241-260 行
````python
)

# Log the fusion failures due to indexing mismatch
MetricTable.register_table(
    "fusion_failure_due_to_indexing_mismatch",
    [
        "pre_grad_graph_id",
        "post_grad_graph_id",
        "node1_name",
        "node2_name",
        "node1_debug_str",
        "node2_debug_str",
        "common_buffer_names",
        "failure_reason",
    ],
)

# Log metadata for pointwise/reduction kernels. E.g., model name, kernel path, numel, rnumel, reduction hint
MetricTable.register_table(
    "kernel_metadata",
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 261-280 / 第 261-280 行
````python
    [
        "kernel_name",
        "kernel_path",
        "kernel_category",  # pointwise/reduction/foreach etc.
        "size_hints",
        "reduction_hint",
        "line_of_code",
        "num_load",
        "num_store",
        "num_for_loop",
        "num_atomic_add",
        "num_args",
        # xyz numel can be different to size_hints since size_hints are rounded
        # up to the nearest power of 2.
        # Inductor kernel will burn in the xyz numel in kernel code for static
        # shape kernels.
        # Logging them will be helpful to find unaligned shape for reduction
        "xnumel",
        "ynumel",
        "rnumel",
````
- **EN**: Implements the local expressions and calls needed for this part of the module.
- **CN**: 实现了该模块这一部分所需的局部表达式与调用逻辑。

### Lines 281-300 / 第 281-300 行
````python
        "kernel_args_num_gb",
    ],
)


def _parse_kernel_fn_code(kernel_module_code: str) -> str:
    """
    The kernel_module_code is the python module that contains kernel function code.
    kernel function is the proper triton kernel function annotated with
    @triton.jit
    """
    from .codecache import PyCodeCache
    from .wrapper_benchmark import get_triton_kernel

    mod = PyCodeCache.load(kernel_module_code)
    kernel = get_triton_kernel(mod)
    # kernel is a CachingAutotune; kernel.fn is the JITFunction;
    # kernel.fn.fn is the function being decorate by triton.jit
    return inspect.getsource(kernel.fn.fn)

````
- **EN**: Imports dependencies such as `.codecache`, and `.wrapper_benchmark` for the logic in this range. Introduces function `_parse_kernel_fn_code`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `.codecache`、`.wrapper_benchmark` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_parse_kernel_fn_code`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 301-320 / 第 301-320 行
````python

def _parse_kernel_line_of_code(proper_kernel_fn_code: str) -> int:
    """
    Return the line of code for the kernel excluding the decorators.
    """
    return len(proper_kernel_fn_code.splitlines())


def _parse_size_hints(kernel_module_code: str, kernel_category: str) -> str | None:
    if kernel_category == "foreach":
        # foreach kernel does not have size_hints
        return None
    m = re.search(r"size_hints=(\[[0-9, ]*\]),", kernel_module_code)
    assert m, "size_hints missing!"
    return m.group(1)


def _parse_reduction_hint(kernel_category: str, kernel_module_code: str) -> str | None:
    if kernel_category not in ("reduction", "persistent_reduction"):
        return None
````
- **EN**: Introduces function `_parse_kernel_line_of_code`, function `_parse_size_hints`, function `_parse_reduction_hint`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `m`.
- **CN**: 这里定义了函数`_parse_kernel_line_of_code`、函数`_parse_size_hints`、函数`_parse_reduction_hint`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `m` 等值。

### Lines 321-340 / 第 321-340 行
````python
    m = re.search(r"reduction_hint=ReductionHint\.(\w*),", kernel_module_code)
    assert m, "reduction_hint not found in kernel source code!"
    return m.group(1)


def _count_pattern(proper_kernel_fn_code: str, pattern: str) -> int:
    return proper_kernel_fn_code.count(pattern)


def _count_args(proper_kernel_fn_code: str) -> int:
    def_line = proper_kernel_fn_code.splitlines()[0]
    assert def_line.startswith("def ")
    start_idx = def_line.index("(")
    end_idx = def_line.index("):")
    decl_csv = def_line[start_idx + 1 : end_idx]
    comps = decl_csv.split(",")
    return len(comps)


def _parse_proper_kernel_fn_code(kernel_fn_code: str) -> str:
````
- **EN**: Introduces function `_count_pattern`, function `_count_args`, function `_parse_proper_kernel_fn_code`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `m`, `def_line`, `start_idx`, `end_idx`, `decl_csv`, and `comps`.
- **CN**: 这里定义了函数`_count_pattern`、函数`_count_args`、函数`_parse_proper_kernel_fn_code`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `m`、`def_line`、`start_idx`、`end_idx`、`decl_csv`、`comps` 等值。

### Lines 341-360 / 第 341-360 行
````python
    """
    Skip decorators.
    """
    start_pos = kernel_fn_code.index("def ")
    return kernel_fn_code[start_pos:]


def _parse_numel(proper_kernel_fn_code: str, numel_arg_name: str) -> int | None:
    m = re.search(f"{numel_arg_name} = ([\\d]+)", proper_kernel_fn_code)
    if m:
        return int(m.group(1))
    else:
        return None


def _parse_kernel_args_num_gb(
    kernel_fn_code: str, kernel_category: str
) -> float | None:
    """
    inductor meta looks like:
````
- **EN**: Introduces function `_parse_numel`, function `_parse_kernel_args_num_gb`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `start_pos`, `m`, `else`, and `kernel_fn_code`.
- **CN**: 这里定义了函数`_parse_numel`、函数`_parse_kernel_args_num_gb`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `start_pos`、`m`、`else`、`kernel_fn_code` 等值。

### Lines 361-380 / 第 361-380 行
````python
        inductor_meta={... 'mutated_arg_names': [], 'no_x_dim': False, 'kernel_num_gb': 2.0},
    """
    m = re.search(r".kernel_num_gb.:\s*([0-9.]+)", kernel_fn_code)
    if m:
        return float(m.group(1))
    else:
        """
        There are a few cases that kernel_num_gdb field can be missing:
        1. the field will be missing if config.benchmark_kernel and
           config.profile_bandwidth are false
        2. even if config.benchmark_kernel or config.profile_bandwidth is true.
           foreach kernel does not have kernel_num_gb field in the metadata
        """
        return None


def log_kernel_metadata(
    kernel_name: str, kernel_path: str, kernel_module_code: str
) -> None:
    """
````
- **EN**: Introduces function `log_kernel_metadata`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `inductor_meta`, `m`, `else`, and `kernel_name`.
- **CN**: 这里定义了函数`log_kernel_metadata`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `inductor_meta`、`m`、`else`、`kernel_name` 等值。

### Lines 381-400 / 第 381-400 行
````python
    An utility to log kernel metadata. We may parse metadata from kernel source code here.

    It's fine to parse the generated kernel code here since the logging is
    disabled by default. It would hurt compilation time.
    """
    from .wrapper_benchmark import get_kernel_category_by_source_code

    kernel_category = get_kernel_category_by_source_code(kernel_module_code)
    reduction_hint = _parse_reduction_hint(kernel_category, kernel_module_code)
    size_hints = _parse_size_hints(kernel_module_code, kernel_category)
    kernel_fn_code = _parse_kernel_fn_code(kernel_module_code)

    proper_kernel_fn_code = _parse_proper_kernel_fn_code(kernel_fn_code)

    # the line of code excluding the decortors
    kernel_line_of_code = _parse_kernel_line_of_code(proper_kernel_fn_code)

    get_metric_table("kernel_metadata").add_row(
        lambda: {
            "kernel_name": kernel_name,
````
- **EN**: Imports dependencies such as `.wrapper_benchmark` for the logic in this range. Initializes or updates values such as `kernel_category`, `reduction_hint`, `size_hints`, `kernel_fn_code`, `proper_kernel_fn_code`, `kernel_line_of_code`, and `...+1`. This range continues the implementation of function `log_kernel_metadata`.
- **CN**: 这里导入了 `.wrapper_benchmark` 等依赖，为后续逻辑提供基础能力。初始化或更新了 `kernel_category`、`reduction_hint`、`size_hints`、`kernel_fn_code`、`proper_kernel_fn_code`、`kernel_line_of_code`、`另有1项` 等值。这一段延续了函数`log_kernel_metadata` 的具体实现。

### Lines 401-420 / 第 401-420 行
````python
            "kernel_path": kernel_path,
            "kernel_category": kernel_category,
            "size_hints": size_hints,
            "reduction_hint": reduction_hint,
            "line_of_code": kernel_line_of_code,
            "num_load": _count_pattern(proper_kernel_fn_code, "tl.load"),
            "num_store": _count_pattern(proper_kernel_fn_code, "tl.store"),
            "num_for_loop": _count_pattern(proper_kernel_fn_code, "for "),
            "num_atomic_add": _count_pattern(proper_kernel_fn_code, "tl.atomic_add"),
            "num_args": _count_args(proper_kernel_fn_code),
            "xnumel": _parse_numel(proper_kernel_fn_code, "xnumel"),
            "ynumel": _parse_numel(proper_kernel_fn_code, "ynumel"),
            "rnumel": _parse_numel(proper_kernel_fn_code, "rnumel"),
            "kernel_args_num_gb": _parse_kernel_args_num_gb(
                kernel_fn_code, kernel_category
            ),
        }
    )


````
- **EN**: This range continues the implementation of function `log_kernel_metadata`.
- **CN**: 这一段延续了函数`log_kernel_metadata` 的具体实现。

### Lines 421-440 / 第 421-440 行
````python
def purge_old_log_files() -> None:
    """
    Purge the old log file at the beginning when the benchmark script runs.
    Should do it in the parent process rather than the child processes running
    each individual model.
    """
    for name, table in REGISTERED_METRIC_TABLES.items():
        if name in enabled_metric_tables():
            filename = table.output_filename()
            if os.path.exists(filename):
                os.unlink(filename)

            table.write_header()


def enabled_metric_tables() -> OrderedSet[str]:
    return enabled_metric_tables_impl(config.enabled_metric_tables)


@lru_cache
````
- **EN**: Introduces function `purge_old_log_files`, function `enabled_metric_tables`. Applies decorators to register behavior or alter how the following definition is constructed. Touches the filesystem to load, validate, or store compiler artifacts. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里定义了函数`purge_old_log_files`、函数`enabled_metric_tables`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段会访问文件系统，用于加载、校验或保存编译产物。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 441-460 / 第 441-460 行
````python
def enabled_metric_tables_impl(config_str: str) -> OrderedSet[str]:
    enabled: OrderedSet[str] = OrderedSet()
    for name in config_str.split(","):
        name = name.strip()
        if not name:
            continue
        assert name in REGISTERED_METRIC_TABLES, (
            f"Metric table name {name} is not registered"
        )
        enabled.add(name)
    return enabled


def is_metric_table_enabled(name: str) -> bool:
    return name in enabled_metric_tables()


def get_metric_table(name: str) -> MetricTable:
    assert name in REGISTERED_METRIC_TABLES, f"Metric table {name} is not defined"
    return REGISTERED_METRIC_TABLES[name]
````
- **EN**: Introduces function `enabled_metric_tables_impl`, function `is_metric_table_enabled`, function `get_metric_table`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`enabled_metric_tables_impl`、函数`is_metric_table_enabled`、函数`get_metric_table`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 461-480 / 第 461-480 行
````python


MetricTable.register_table(
    "kernel_autotune",
    [
        "kernel_path",
        "kernel_name",
        "triton_config",
        "latency_ms",
    ],
)


def log_kernel_autotune_result(
    kernel_path: str, kernel_name: str, config: Config, latency: float
) -> None:
    get_metric_table("kernel_autotune").add_row(
        lambda: {
            "kernel_path": kernel_path,
            "kernel_name": kernel_name,
````
- **EN**: Introduces function `log_kernel_autotune_result`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `kernel_path`, and `lambda`.
- **CN**: 这里定义了函数`log_kernel_autotune_result`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `kernel_path`、`lambda` 等值。

### Lines 481-484 / 第 481-484 行
````python
            "triton_config": str(config),
            "latency_ms": latency,
        }
    )
````
- **EN**: This range continues the implementation of function `log_kernel_autotune_result`.
- **CN**: 这一段延续了函数`log_kernel_autotune_result` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Scheduling decisions  
  **CN**: 调度决策
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `csv`, `dataclasses`, `inspect`, `os`, `re`, `functools`, `typing`, `collections.abc`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch._inductor`, `torch._inductor.utils`, `torch.utils._ordered_set`, `torch._inductor.runtime.triton_compat`, `torch._inductor.scheduler`, `.codecache`, `.wrapper_benchmark`
