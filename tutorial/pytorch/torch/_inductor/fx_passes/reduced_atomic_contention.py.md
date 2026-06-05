# reduced_atomic_contention.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/reduced_atomic_contention.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `_get_min_partitions`, `_get_max_partitions`, `_get_memory_budget_fraction`, `partitioned_scatter_optimization_pass`, `validate_match`, `create_replacement`, and `...+5`. Module note: Partitioned Scatter Optimization for Reduced Atomic Contention.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `_get_min_partitions`、`_get_max_partitions`、`_get_memory_budget_fraction`、`partitioned_scatter_optimization_pass`、`validate_match`、`create_replacement`、`另有5项` 等函数。模块文档字符串给出了额外背景说明。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
"""
Partitioned Scatter Optimization for Reduced Atomic Contention.

This pass transforms high-contention index_put operations by distributing
writes across multiple partitions, reducing atomic contention.
"""

import logging
import math
from typing import Any

import torch
import torch.fx as fx
from torch._dynamo.utils import counters
from torch._inductor import config
from torch._inductor.pattern_matcher import (
    Arg,
    CallFunction,
    Match,
````
- **EN**: Imports dependencies such as `logging`, `math`, `typing`, `torch`, `torch.fx`, `torch._dynamo.utils`, and `...+2` for the logic in this range.
- **CN**: 这里导入了 `logging`、`math`、`typing`、`torch`、`torch.fx`、`torch._dynamo.utils`、`另有2项` 等依赖，为后续逻辑提供基础能力。

### Lines 21-40 / 第 21-40 行
````python
    PatternMatcherPass,
    register_graph_pattern,
)


log = logging.getLogger(__name__)
aten = torch.ops.aten
prims = torch.ops.prims


def _get_min_partitions() -> int:
    """Get minimum partitions from config."""
    return getattr(config, "partitioned_scatter_min_partitions", 2)


def _get_max_partitions() -> int:
    """Get maximum partitions from config."""
    return getattr(config, "partitioned_scatter_max_partitions", 128)


````
- **EN**: Introduces function `_get_min_partitions`, function `_get_max_partitions`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `log`, `aten`, and `prims`.
- **CN**: 这里定义了函数`_get_min_partitions`、函数`_get_max_partitions`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `log`、`aten`、`prims` 等值。

### Lines 41-60 / 第 41-60 行
````python
def _get_memory_budget_fraction() -> float:
    """Get memory budget fraction from config."""
    return getattr(config, "partitioned_scatter_memory_budget", 0.10)


partitioned_scatter_patterns = PatternMatcherPass(
    pass_name="partitioned_scatter_optimization"
)


def partitioned_scatter_optimization_pass(graph: fx.Graph) -> fx.Graph:
    """
    Apply partitioned scatter optimization to high-contention index_put operations.

    Reduces atomic contention by distributing writes across multiple buffers.
    Controlled by: config.partitioned_scatter_enabled
    """
    if not getattr(config, "partitioned_scatter_enabled", False):
        return graph

````
- **EN**: Introduces function `_get_memory_budget_fraction`, function `partitioned_scatter_optimization_pass`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `partitioned_scatter_patterns`, and `pass_name`.
- **CN**: 这里定义了函数`_get_memory_budget_fraction`、函数`partitioned_scatter_optimization_pass`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `partitioned_scatter_patterns`、`pass_name` 等值。

### Lines 61-80 / 第 61-80 行
````python
    num_matches = partitioned_scatter_patterns.apply(graph)

    if num_matches > 0:
        log.info(
            "partitioned_scatter_optimization: applied to %d operation(s)",
            num_matches,
        )
        graph.lint()

    return graph


def validate_match(match: Match) -> bool:
    """Check if pattern match should be optimized for scatter."""
    output_node = match.output_node()
    if not output_node or not hasattr(output_node, "args") or len(output_node.args) < 4:
        return False

    # Only apply when accumulating
    if output_node.args[3] is not True:
````
- **EN**: Introduces function `validate_match`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `num_matches`, and `output_node`.
- **CN**: 这里定义了函数`validate_match`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `num_matches`、`output_node` 等值。

### Lines 81-100 / 第 81-100 行
````python
        log.debug("Skipping: accumulate=False")
        return False

    # Extract metadata
    input_node = output_node.args[0]
    indices_arg = output_node.args[1]

    # Validate input_node is an FX Node
    if not isinstance(input_node, fx.Node):
        return False

    scatter_dim, index_node = _extract_scatter_dim_and_index(indices_arg)
    if scatter_dim is None or index_node is None:
        return False

    # Get tensor shapes and validate
    input_meta = _get_tensor_meta(input_node)
    index_meta = _get_tensor_meta(index_node)
    if not input_meta or not index_meta:
        return False
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_node`, `indices_arg`, `input_meta`, and `index_meta`. This range continues the implementation of function `validate_match`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_node`、`indices_arg`、`input_meta`、`index_meta` 等值。这一段延续了函数`validate_match` 的具体实现。

### Lines 101-120 / 第 101-120 行
````python

    # Skip unsupported cases
    if isinstance(input_meta["numel"], torch.SymInt) or isinstance(
        index_meta["numel"], torch.SymInt
    ):
        log.debug("Skipping: dynamic shapes not supported")
        return False

    if input_meta["dtype"] == torch.bool or index_meta["dtype"] == torch.bool:
        log.debug("Skipping: bool dtype not supported")
        return False

    if scatter_dim >= len(input_meta["shape"]):
        log.debug("Skipping: scatter dim %d out of bounds", scatter_dim)
        return False

    # Calculate optimal partitions and check memory
    output_size = input_meta["numel"]
    index_size = index_meta["numel"]

````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `output_size`, and `index_size`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `output_size`、`index_size` 等值。

### Lines 121-140 / 第 121-140 行
````python
    # Safety check (also done in _estimate_optimal_partitions)
    if output_size == 0 or index_size == 0:
        return False

    contention_ratio = index_size / output_size

    # Check minimum index size threshold
    min_index_size = getattr(config, "partitioned_scatter_min_index_size", 4096)
    if index_size < min_index_size:
        log.debug(
            "Skipping: index size %d below threshold %d", index_size, min_index_size
        )
        return False

    # Get optimal partitions and adjust for memory constraints
    num_partitions = _estimate_optimal_partitions(output_size, index_size)
    num_partitions = _fit_to_memory_budget(
        output_size, num_partitions, input_meta["dtype"]
    )

````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `contention_ratio`, `min_index_size`, and `num_partitions`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `contention_ratio`、`min_index_size`、`num_partitions` 等值。

### Lines 141-160 / 第 141-160 行
````python
    # If reduced to < min partitions, optimization not worthwhile
    if num_partitions < _get_min_partitions():
        log.debug("Skipping: insufficient memory for minimum partitions")
        return False

    # Store optimization parameters for replacement
    match._num_partitions = num_partitions  # type: ignore[attr-defined]
    match._scatter_dim = scatter_dim  # type: ignore[attr-defined]
    match._index_node = index_node  # type: ignore[attr-defined]

    log.debug(
        "Applying optimization: %d partitions, dim=%d, contention=%.2f, "
        "output_size=%d, index_size=%d",
        num_partitions,
        scatter_dim,
        contention_ratio,
        output_size,
        index_size,
    )

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `validate_match`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`validate_match` 的具体实现。

### Lines 161-180 / 第 161-180 行
````python
    return True


@register_graph_pattern(
    CallFunction(aten.index_put.default, Arg(), Arg(), Arg(), True),
    pass_dict=partitioned_scatter_patterns,  # type: ignore[arg-type]
    extra_check=validate_match,
)
@register_graph_pattern(
    CallFunction(aten.index_put_.default, Arg(), Arg(), Arg(), True),
    pass_dict=partitioned_scatter_patterns,  # type: ignore[arg-type]
    extra_check=validate_match,
)
def create_replacement(match: Match, input_tensor, indices, values) -> None:
    """Replace high-contention index_put with partitioned scatter."""
    # Get optimization parameters (set in validate_match)
    num_partitions: int = match._num_partitions  # type: ignore[attr-defined]
    scatter_dim: int = match._scatter_dim  # type: ignore[attr-defined]
    index_node = match._index_node  # type: ignore[attr-defined]

````
- **EN**: Introduces function `create_replacement`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`create_replacement`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 181-200 / 第 181-200 行
````python
    def repl(input_tensor, index_node, values):
        """Partitioned scatter implementation that will be traced."""
        dim_size = input_tensor.shape[scatter_dim]
        num_operations = index_node.numel()

        # Flatten if needed
        if len(index_node.shape) > 1:
            flat_index = index_node.reshape(num_operations)
            values_ndim = len(index_node.shape)
            flat_values = values.reshape(
                [num_operations] + list(values.shape[values_ndim:])
            )
        else:
            flat_index = index_node
            flat_values = values

        # Generate operation IDs and assign to partitions
        operation_ids = torch.ops.prims.iota.default(
            num_operations,
            start=0,
````
- **EN**: Introduces function `repl`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dim_size`, `num_operations`, `flat_index`, `values_ndim`, `flat_values`, `else`, and `...+2`.
- **CN**: 这里定义了函数`repl`。包含分支、循环或上下文管理等控制流。初始化或更新了 `dim_size`、`num_operations`、`flat_index`、`values_ndim`、`flat_values`、`else`、`另有2项` 等值。

### Lines 201-220 / 第 201-220 行
````python
            step=1,
            dtype=flat_index.dtype,
            device=flat_index.device,
            requires_grad=False,
        )
        partition_ids = torch.ops.aten.bitwise_and.Scalar(
            operation_ids, num_partitions - 1
        )

        # Create expanded buffer
        expanded_shape = list(input_tensor.shape)
        expanded_shape[scatter_dim] *= num_partitions
        expanded_buffer = torch.ops.aten.full.default(
            expanded_shape,
            0,
            dtype=flat_values.dtype,
            layout=torch.strided,
            device=flat_values.device,
            pin_memory=False,
        )
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `step`, `dtype`, `device`, `requires_grad`, `partition_ids`, `expanded_shape`, and `...+3`. This range continues the implementation of function `create_replacement.repl`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `step`、`dtype`、`device`、`requires_grad`、`partition_ids`、`expanded_shape`、`另有3项` 等值。这一段延续了函数`create_replacement.repl` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python

        # Adjust indices for partitioning
        partition_offsets = partition_ids * dim_size
        adjusted_index = flat_index + partition_offsets

        # Reconstruct indices list for scatter
        if isinstance(indices, (list, tuple)):
            adjusted_indices = [
                adjusted_index if i == scatter_dim else idx
                for i, idx in enumerate(indices)
            ]
        else:
            adjusted_indices = [adjusted_index]

        # Scatter with reduced contention
        scattered_buffer = torch.ops.aten.index_put.default(
            expanded_buffer, adjusted_indices, flat_values, True
        )

        # Reshape for reduction
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `partition_offsets`, `adjusted_index`, `adjusted_indices`, `else`, and `scattered_buffer`. This range continues the implementation of function `create_replacement.repl`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `partition_offsets`、`adjusted_index`、`adjusted_indices`、`else`、`scattered_buffer` 等值。这一段延续了函数`create_replacement.repl` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python
        reduce_shape = list(expanded_shape)
        reduce_shape[scatter_dim] = num_partitions
        reduce_shape.insert(scatter_dim + 1, dim_size)
        reshaped = torch.ops.aten.view.default(scattered_buffer, reduce_shape)

        # Sum across partitions (preserve dtype for int types)
        if flat_values.dtype in [torch.int8, torch.int16, torch.int32, torch.uint8]:
            reduced = torch.ops.aten.sum.dim_IntList(
                reshaped, [scatter_dim], dtype=flat_values.dtype
            )
        else:
            reduced = torch.ops.aten.sum.dim_IntList(reshaped, [scatter_dim])

        # Add to original input
        return input_tensor + reduced

    counters["inductor"]["partitioned_scatter_applied"] += 1
    # pyrefly: ignore [bad-argument-type]
    match.replace_by_example(repl, [input_tensor, index_node, values])

````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `reduce_shape`, `reshaped`, `reduced`, and `else`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `reduce_shape`、`reshaped`、`reduced`、`else` 等值。

### Lines 261-280 / 第 261-280 行
````python

def _get_max_partitions_for_size(output_size: int) -> int:
    """
    Get maximum partitions based on output tensor size.

    Larger tensors use fewer partitions to limit memory overhead.
    """
    if output_size >= 100_000_000:  # >= 100M elements
        return 4
    elif output_size >= 10_000_000:  # >= 10M elements
        return 8
    elif output_size >= 1_000_000:  # >= 1M elements
        return 16
    else:  # < 1M elements
        return _get_max_partitions()


def _estimate_optimal_partitions(output_size: int, index_size: int) -> int:
    """Estimate optimal number of partitions based on contention ratio."""
    # Safety check for edge cases
````
- **EN**: Introduces function `_get_max_partitions_for_size`, function `_estimate_optimal_partitions`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这里定义了函数`_get_max_partitions_for_size`、函数`_estimate_optimal_partitions`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 281-300 / 第 281-300 行
````python
    if output_size == 0 or index_size == 0:
        return _get_min_partitions()

    contention_ratio = index_size / output_size

    # Size-aware partition limits (larger tensors = fewer partitions to limit memory)
    max_partitions_for_size = _get_max_partitions_for_size(output_size)

    # Contention-based calculation - square root scaling
    # Use max to ensure we never go below min_partitions for the base calculation
    base_partitions = max(_get_min_partitions(), int(math.sqrt(contention_ratio) * 16))

    # Round to power of 2 and apply limits
    partitions = 2 ** math.ceil(math.log2(base_partitions))
    return min(partitions, max_partitions_for_size, _get_max_partitions())


def _fit_to_memory_budget(
    output_size: int, num_partitions: int, dtype: torch.dtype
) -> int:
````
- **EN**: Introduces function `_fit_to_memory_budget`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_fit_to_memory_budget`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 301-320 / 第 301-320 行
````python
    """
    Reduce partitions to fit memory budget if needed.

    Returns the maximum number of partitions that fit in memory budget.
    Returns input num_partitions if it fits, or a reduced count, or 0 if
    even min_partitions doesn't fit.
    """
    if not torch.cuda.is_available():
        return num_partitions

    try:
        _, total_memory = torch.cuda.mem_get_info()
        element_bytes = dtype.itemsize if hasattr(dtype, "itemsize") else 4
        budget = total_memory * _get_memory_budget_fraction()

        # Try reducing partitions (must be power of 2) until we fit
        current_partitions = num_partitions
        min_partitions = _get_min_partitions()
        while current_partitions >= min_partitions:
            overhead = output_size * element_bytes * (current_partitions - 1)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 321-340 / 第 321-340 行
````python

            if overhead <= budget:
                # Only format debug string if debug logging is enabled
                if current_partitions < num_partitions and log.isEnabledFor(
                    logging.DEBUG
                ):
                    log.debug(
                        "Reduced partitions from %d to %d to fit memory budget "
                        "(%.2fGB / %.2fGB)",
                        num_partitions,
                        current_partitions,
                        overhead / 1e9,
                        budget / 1e9,
                    )
                return current_partitions

            # Reduce by half (maintain power of 2)
            current_partitions //= 2

        # If min_partitions doesn't fit in memory, return 0
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `_fit_to_memory_budget`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`_fit_to_memory_budget` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python
        if log.isEnabledFor(logging.DEBUG):
            overhead = output_size * element_bytes * (min_partitions - 1)
            log.debug(
                "Insufficient memory even for %d partitions: %.2fGB > %.2fGB",
                min_partitions,
                overhead / 1e9,
                budget / 1e9,
            )
        return 0

    except Exception:
        log.debug("Memory check failed, proceeding with %s", num_partitions)
        return num_partitions  # Assume we have enough memory if we can't check


def _extract_scatter_dim_and_index(
    indices_arg: Any,
) -> tuple[int | None, fx.Node | None]:
    """Extract scatter dimension and index node from indices argument."""
    # Case 1: Single index → dim=0
````
- **EN**: Introduces function `_extract_scatter_dim_and_index`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `overhead`, and `indices_arg`.
- **CN**: 这里定义了函数`_extract_scatter_dim_and_index`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `overhead`、`indices_arg` 等值。

### Lines 361-380 / 第 361-380 行
````python
    if not isinstance(indices_arg, (list, tuple)):
        return 0, indices_arg

    # List with Nones → position of non-None is dim
    index_node = None
    scatter_dim = None

    # Case 2 -> Find the first non-None index as the scatter dimension
    for dim, idx in enumerate(indices_arg):
        if idx is not None:
            if index_node is not None:
                # Multiple indices not supported
                return None, None
            index_node = idx
            scatter_dim = dim

    return scatter_dim, index_node


def _get_tensor_meta(node: fx.Node) -> dict[str, Any] | None:
````
- **EN**: Introduces function `_get_tensor_meta`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `index_node`, and `scatter_dim`.
- **CN**: 这里定义了函数`_get_tensor_meta`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `index_node`、`scatter_dim` 等值。

### Lines 381-397 / 第 381-397 行
````python
    """Extract tensor metadata from FX node."""
    if not hasattr(node, "meta") or "val" not in node.meta:
        return None

    val = node.meta["val"]
    if not isinstance(val, (torch.Tensor, type(val))) or not hasattr(val, "shape"):
        return None

    return {
        "shape": tuple(val.shape),
        "dtype": val.dtype,
        "device": val.device,
        "numel": val.numel(),
    }


__all__ = ["partitioned_scatter_optimization_pass"]
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `_get_min_partitions`, `_get_max_partitions`, `_get_memory_budget_fraction`, `partitioned_scatter_optimization_pass`, `validate_match`, `create_replacement`, and `...+5`  
  **CN**: 主要函数：`_get_min_partitions`、`_get_max_partitions`、`_get_memory_budget_fraction`、`partitioned_scatter_optimization_pass`、`validate_match`、`create_replacement`、`另有5项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `math`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `torch._dynamo.utils`, `torch._inductor`, `torch._inductor.pattern_matcher`
