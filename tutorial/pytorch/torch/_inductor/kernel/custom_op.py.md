# custom_op.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel/custom_op.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `RangeBounds`, `ImplConfig`, `RangeImplGroup`, and `CustomOpConfig`. It exposes functions such as `_detect_collective_ops`, `_extract_tensor_inputs`, `_merge_config_and_runtime_kwargs`, `_adapt_user_input_gen_fns`, `_group_ranges_by_impl`, `_create_ranges_from_split_points`, and `...+12`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `RangeBounds`、`ImplConfig`、`RangeImplGroup`、`CustomOpConfig` 等类。同时提供 `_detect_collective_ops`、`_extract_tensor_inputs`、`_merge_config_and_runtime_kwargs`、`_adapt_user_input_gen_fns`、`_group_ranges_by_impl`、`_create_ranges_from_split_points`、`另有12项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# Owner(s): ["module: inductor"]

import contextlib
import functools
import logging
from collections.abc import Callable
from dataclasses import dataclass, field
from typing import Any

import torch
from torch._dynamo.utils import counters
from torch._inductor.codegen.subgraph import SubgraphTemplate
from torch._inductor.ir import (
    Buffer,
    ChoiceCaller,
    FixedLayout,
    ir_node_to_tensor,
    StorageBox,
    TensorBox,
)
from torch._inductor.lowering import user_lowerings, validate_ir
from torch._inductor.select_algorithm import (
    autotune_select_algorithm,
    ExternKernelChoice,
)
from torch._inductor.utils import convert_symint_to_expr
from torch._inductor.virtualized import V

````
- **EN**: Imports dependencies such as `contextlib`, `functools`, `logging`, `collections.abc`, `dataclasses`, `typing`, and `...+8` for the logic in this range.
- **CN**: 这里导入了 `contextlib`、`functools`、`logging`、`collections.abc`、`dataclasses`、`typing`、`另有8项` 等依赖，为后续逻辑提供基础能力。

### Lines 29-56 / 第 29-56 行
````python

log = logging.getLogger(__name__)

DEFAULT_RANGE_UPPER_BOUND = 65536


@dataclass(frozen=True)
class RangeBounds:
    """Inclusive range [start, end] for dimension-based dispatch."""

    start: int
    end: int | float  # float('inf') for unbounded

    def __post_init__(self) -> None:
        if self.start < 1:
            raise ValueError(f"Range start must be >= 1, got {self.start}")
        if self.end != float("inf") and self.start > self.end:
            raise ValueError(f"Invalid range: start={self.start} > end={self.end}")

    def contains(self, value: int) -> bool:
        if self.end == float("inf"):
            return value >= self.start
        return self.start <= value <= int(self.end)

    def __str__(self) -> str:
        end_str = "inf" if self.end == float("inf") else str(int(self.end))
        return f"[{self.start}, {end_str}]"

````
- **EN**: Introduces class `RangeBounds`, function `__post_init__`, function `contains`, function `__str__`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`RangeBounds`、函数`__post_init__`、函数`contains`、函数`__str__`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 57-84 / 第 57-84 行
````python

@dataclass(frozen=True)
class ImplConfig:
    """Implementation config with semantic identity (name + kwargs) for hashing."""

    impl_name: str
    impl_func: Callable[..., Any] = field(compare=False, hash=False, repr=False)
    kwargs: dict[str, Any] = field(default_factory=dict)
    config_patches: dict[str, Any] = field(
        default_factory=dict, compare=False, hash=False, repr=False
    )

    def __hash__(self) -> int:
        return hash((self.impl_name, tuple(sorted(self.kwargs.items()))))

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, ImplConfig):
            return False
        return self.impl_name == other.impl_name and self.kwargs == other.kwargs

    def __str__(self) -> str:
        if self.kwargs:
            kwargs_str = ", ".join(f"{k}={v}" for k, v in sorted(self.kwargs.items()))
            return f"{self.impl_name}({kwargs_str})"
        return self.impl_name


@dataclass
````
- **EN**: Introduces class `ImplConfig`, function `__hash__`, function `__eq__`, function `__str__`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`ImplConfig`、函数`__hash__`、函数`__eq__`、函数`__str__`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 85-112 / 第 85-112 行
````python
class RangeImplGroup:
    """Groups non-adjacent ranges using the same implementation."""

    impl_config: ImplConfig
    ranges: list[RangeBounds] = field(default_factory=list)

    def add_range(self, range_bounds: RangeBounds) -> None:
        self.ranges.append(range_bounds)
        self.ranges.sort(key=lambda r: r.start)

    def __str__(self) -> str:
        ranges_str = ", ".join(str(r) for r in self.ranges)
        return f"{self.impl_config.impl_name}: {ranges_str}"

    @property
    def impl_name(self) -> str:
        return self.impl_config.impl_name

    @property
    def impl_func(self) -> Callable[..., Any]:
        return self.impl_config.impl_func

    @property
    def impl_kwargs(self) -> dict[str, Any]:
        return self.impl_config.kwargs

    @property
    def config_patches(self) -> dict[str, Any]:
````
- **EN**: Introduces class `RangeImplGroup`, function `add_range`, function `__str__`, function `impl_name`, function `impl_func`, function `impl_kwargs`, function `config_patches`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `impl_config`, `ranges`, and `ranges_str`.
- **CN**: 这里定义了类`RangeImplGroup`、函数`add_range`、函数`__str__`、函数`impl_name`、函数`impl_func`、函数`impl_kwargs`、函数`config_patches`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `impl_config`、`ranges`、`ranges_str` 等值。

### Lines 113-140 / 第 113-140 行
````python
        return self.impl_config.config_patches


def _detect_collective_ops(choices: list) -> bool:
    """
    Detect if choices contain collective operations.
    """
    from torch._inductor.utils import is_collective_op

    for choice in choices:
        if not hasattr(choice, "gm") or choice.gm is None:
            continue

        for node in choice.gm.graph.nodes:
            if node.op == "call_function" and node.target is not None:
                op_name = str(node.target)

                if is_collective_op(op_name) or is_collective_op(
                    f"torch.ops.{op_name}"
                ):
                    return True

    return False


class CustomOpConfig:
    """Config for custom op autotuning.

````
- **EN**: Imports dependencies such as `torch._inductor.utils` for the logic in this range. Introduces function `_detect_collective_ops`, class `CustomOpConfig`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.utils` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_detect_collective_ops`、类`CustomOpConfig`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-168 / 第 141-168 行
````python
    Specifies optional decomposition function with parameter values.
    Each config creates exactly one variant.

    Args:
        decomposition: Optional functions to autotune. If not provided, default will be used.
        config_patches: Optional dict of config patches to apply during kernel codegen
            (e.g., {"coordinate_descent_tuning": True})
        **params: Parameters passed to the function

    Examples:
        CustomOpConfig(attention_impl, head_dim=32, method='chunked')
        CustomOpConfig(head_dim=32, method='chunked')
        CustomOpConfig(decomposition, config_patches={"coordinate_descent_tuning": True})
    """

    def __init__(
        self,
        decomposition: Callable[..., Any] | None = None,
        config_patches: dict[str, Any] | None = None,
        **params: Any,
    ):
        if decomposition is not None and not callable(decomposition):
            raise TypeError(
                f"decomposition must be callable, got {type(decomposition)}"
            )

        self.decomposition = decomposition
        self.config_patches = config_patches or {}
````
- **EN**: Introduces function `__init__`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Args`, `decomposition`, `config_patches`, and `Examples`.
- **CN**: 这里定义了函数`__init__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Args`、`decomposition`、`config_patches`、`Examples` 等值。

### Lines 169-196 / 第 169-196 行
````python
        self.params = params

    def get_decomposition(
        self, default_impl: Callable[..., Any] | None = None
    ) -> Callable[..., Any]:
        """Return the decomposition function for this config.
        When decomposition is not specified, return the default implementation.
        """
        if self.decomposition is not None:
            return self.decomposition

        if default_impl is not None and callable(default_impl):
            return default_impl

        raise TypeError(
            "No decomposition specified in config and no default implementation provided. "
            "Please provide a decomposition function in CustomOpConfig."
        )

    def __repr__(self) -> str:
        decomp_name = self.decomposition.__name__ if self.decomposition else "default"
        if self.params:
            params_str = ", ".join(f"{k}={v}" for k, v in self.params.items())
            return f"CustomOpConfig({decomp_name}, {params_str})"
        return f"CustomOpConfig({decomp_name})"


__all__ = [
````
- **EN**: Introduces function `get_decomposition`, function `__repr__`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `decomp_name`, `params_str`, and `__all__`.
- **CN**: 这里定义了函数`get_decomposition`、函数`__repr__`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `decomp_name`、`params_str`、`__all__` 等值。

### Lines 197-224 / 第 197-224 行
````python
    "autotune_custom_op",
    "register_custom_op_autotuning",
    "CustomOpConfig",
]


def _extract_tensor_inputs(
    args: tuple[Any, ...],
    kwargs: dict[str, Any],
    op_overload: torch._ops.OpOverload,
) -> tuple[list[Any], dict[str, Any]]:
    """Extract tensor inputs from mixed args/kwargs.
    Separates tensors (for autotuning input_nodes) from non-tensor parameters.

    Args:
        args: Positional arguments (mix of tensors and scalars)
        kwargs: Keyword arguments (mix of tensors and scalars)
        op_overload: Custom Op overload to get parameter names from schema.

    Returns:
        Tuple of (tensor_inputs_list, non_tensor_kwargs)
    """
    tensor_inputs = []
    non_tensor_kwargs = {}

    # Get schema names and extend with fallback names for any extra args
    schema_names = [arg.name for arg in op_overload._schema.arguments]
    param_names = schema_names + [
````
- **EN**: Introduces function `_extract_tensor_inputs`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `args`, `kwargs`, `op_overload`, `Args`, `Returns`, `tensor_inputs`, and `...+3`.
- **CN**: 这里定义了函数`_extract_tensor_inputs`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `args`、`kwargs`、`op_overload`、`Args`、`Returns`、`tensor_inputs`、`另有3项` 等值。

### Lines 225-252 / 第 225-252 行
````python
        f"arg_{i}" for i in range(len(schema_names), len(args))
    ]

    for i, arg in enumerate(args):
        if isinstance(arg, (TensorBox, Buffer, StorageBox)):
            tensor_inputs.append(arg)
        else:
            non_tensor_kwargs[param_names[i]] = arg

    for key, value in kwargs.items():
        if isinstance(value, (TensorBox, Buffer, StorageBox)):
            tensor_inputs.append(value)
        else:
            non_tensor_kwargs[key] = value

    return tensor_inputs, non_tensor_kwargs


def _merge_config_and_runtime_kwargs(
    config_params: dict[str, Any],
    runtime_kwargs: dict[str, Any],
) -> dict[str, Any]:
    """Merge config parameters with runtime kwargs. Config params take precedence,
    since they represent the values being autotuned.

    Args:
        config_params: Parameters from CustomOpConfig (autotuning knobs)
        runtime_kwargs: Runtime non-tensor kwargs from _extract_tensor_inputs
````
- **EN**: Introduces function `_merge_config_and_runtime_kwargs`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `config_params`, `runtime_kwargs`, and `Args`.
- **CN**: 这里定义了函数`_merge_config_and_runtime_kwargs`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`config_params`、`runtime_kwargs`、`Args` 等值。

### Lines 253-280 / 第 253-280 行
````python

    Returns:
        Merged kwargs dictionary with config values taking precedence
    """
    merged_kwargs = runtime_kwargs.copy()
    merged_kwargs.update(config_params)
    return merged_kwargs


def _adapt_user_input_gen_fns(
    inputs: list[Any],
    op_overload: torch._ops.OpOverload,
    user_input_gen_fns: dict[str, Callable[[torch.Tensor], torch.Tensor]],
) -> dict[int, Callable[[Any], torch.Tensor]]:
    """Convert user input generators from name-based to index-based format.
    Inductor autotune's input_gen_fns expects index of arg_names as key.
    """
    arg_names = [arg.name for arg in op_overload._schema.arguments]
    name_to_index = {name: i for i, name in enumerate(arg_names)}
    index_based_fns = {}

    for name, gen_fn in user_input_gen_fns.items():
        if name in name_to_index:
            index_based_fns[name_to_index[name]] = gen_fn
        else:
            raise ValueError(
                f"Unknown argument name '{name}' in input_gen_fns. "
                f"Available argument names: {list(name_to_index.keys())}"
````
- **EN**: Introduces function `_adapt_user_input_gen_fns`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Returns`, `merged_kwargs`, `inputs`, `op_overload`, `user_input_gen_fns`, `arg_names`, and `...+3`.
- **CN**: 这里定义了函数`_adapt_user_input_gen_fns`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Returns`、`merged_kwargs`、`inputs`、`op_overload`、`user_input_gen_fns`、`arg_names`、`另有3项` 等值。

### Lines 281-308 / 第 281-308 行
````python
            )

    def create_internal_input_gen_fn(
        user_function: Callable[[torch.Tensor], torch.Tensor], arg_name: str
    ) -> Callable[[Any], torch.Tensor]:
        """Create internal input generator that converts IR buffer to user's fake tensor."""

        def internal_input_gen_fn(ir_buffer: Any) -> torch.Tensor:
            fake_tensor = ir_node_to_tensor(ir_buffer, replace_symbols_with_hints=True)
            assert fake_tensor is not None, "ir_node_to_tensor returned None"
            return user_function(fake_tensor)

        return internal_input_gen_fn

    return {
        i: create_internal_input_gen_fn(
            user_gen_fn, arg_names[i] if i < len(arg_names) else f"arg_{i}"
        )
        for i, user_gen_fn in index_based_fns.items()
        if i < len(inputs)
    }


def _group_ranges_by_impl(
    range_to_best_impl: dict[RangeBounds, ImplConfig],
) -> list[RangeImplGroup]:
    """Group ranges by implementation using semantic identity (name + kwargs)."""
    from torch._inductor import config
````
- **EN**: Imports dependencies such as `torch._inductor` for the logic in this range. Introduces function `create_internal_input_gen_fn`, function `internal_input_gen_fn`, function `_group_ranges_by_impl`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor` 等依赖，为后续逻辑提供基础能力。这里定义了函数`create_internal_input_gen_fn`、函数`internal_input_gen_fn`、函数`_group_ranges_by_impl`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 309-336 / 第 309-336 行
````python

    if not range_to_best_impl:
        return []

    # Test mode: skip grouping to force torch.cond dispatch path
    if config.test_configs.force_no_impl_grouping:
        log.info("Test mode: skipping impl grouping, each range is separate group")
        groups = []
        for range_bounds, impl_config in sorted(
            range_to_best_impl.items(), key=lambda x: x[0].start
        ):
            group = RangeImplGroup(impl_config)
            group.add_range(range_bounds)
            groups.append(group)
        return groups

    # Group ranges by impl_config (uses __hash__ and __eq__ based on semantic identity)
    impl_to_group: dict[ImplConfig, RangeImplGroup] = {}

    for range_bounds, impl_config in range_to_best_impl.items():
        if impl_config not in impl_to_group:
            impl_to_group[impl_config] = RangeImplGroup(impl_config)
        impl_to_group[impl_config].add_range(range_bounds)

    # Sort groups by first range start for deterministic codegen
    groups = sorted(impl_to_group.values(), key=lambda g: g.ranges[0].start)

    # Log grouping info
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `groups`, `group`, and `impl_to_group`. This range continues the implementation of function `_group_ranges_by_impl`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `groups`、`group`、`impl_to_group` 等值。这一段延续了函数`_group_ranges_by_impl` 的具体实现。

### Lines 337-364 / 第 337-364 行
````python
    original_count = len(range_to_best_impl)
    grouped_count = len(groups)

    if grouped_count < original_count:
        log.info(
            "Implementation grouping: reduced from %d ranges to %d impl groups",
            original_count,
            grouped_count,
        )

    return groups


def _create_ranges_from_split_points(
    split_points: list[int],
) -> list[tuple[int, int] | tuple[int, float]]:
    """Convert split points into ranges for autotuning dispatch.

    Example:
        split_points=[512, 2048]
        returns:
               [(1, 512), (513, 2048), (2049, float('inf'))]
    """
    ranges: list[tuple[int, int] | tuple[int, float]] = []
    start = 1

    for split_point in split_points:
        ranges.append((start, split_point))
````
- **EN**: Introduces function `_create_ranges_from_split_points`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `original_count`, `grouped_count`, `split_points`, `Example`, `returns`, `ranges`, and `...+1`.
- **CN**: 这里定义了函数`_create_ranges_from_split_points`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `original_count`、`grouped_count`、`split_points`、`Example`、`returns`、`ranges`、`另有1项` 等值。

### Lines 365-392 / 第 365-392 行
````python
        start = split_point + 1

    ranges.append((start, float("inf")))

    return ranges


def _create_range_input_gen_fn(
    base_gen_fn: Callable[[torch.Tensor], torch.Tensor],
    dim_index: int,
    range_start: int,
    range_end: int | float,
    range_upper_bound: int,
) -> Callable[[torch.Tensor], torch.Tensor]:
    """Create input generator that modifies target dimension to top of range.
    range_upper_bound: Size to use for benchmarking when range_end is unbounded.
    Default is DEFAULT_RANGE_UPPER_BOUND = 65536
    """
    from torch._inductor.ir import get_fill_order
    from torch._inductor.kernel.flex.common import construct_strides

    target_dim = range_upper_bound if range_end == float("inf") else int(range_end)

    def constrained_gen_fn(fake_tensor: torch.Tensor) -> torch.Tensor:
        result = base_gen_fn(fake_tensor)
        shape = list(result.shape)
        shape[dim_index] = target_dim

````
- **EN**: Imports dependencies such as `torch._inductor.ir`, and `torch._inductor.kernel.flex.common` for the logic in this range. Introduces function `_create_range_input_gen_fn`, function `constrained_gen_fn`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `start`, `base_gen_fn`, `dim_index`, `range_start`, `range_end`, `range_upper_bound`, and `...+3`.
- **CN**: 这里导入了 `torch._inductor.ir`、`torch._inductor.kernel.flex.common` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_create_range_input_gen_fn`、函数`constrained_gen_fn`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `start`、`base_gen_fn`、`dim_index`、`range_start`、`range_end`、`range_upper_bound`、`另有3项` 等值。

### Lines 393-420 / 第 393-420 行
````python
        # We modified the shape of the result, so we need to recalculate the strides
        # TODO: Refine this to a better way to more directly preserve strides
        fill_order = get_fill_order(result.stride(), shape_env=None)
        new_stride = construct_strides(shape, fill_order)

        storage_size = sum((s - 1) * st for s, st in zip(shape, new_stride)) + 1
        storage = torch.randn(storage_size, dtype=result.dtype, device=result.device)
        return storage.as_strided(shape, tuple(new_stride))

    return constrained_gen_fn


def _default_input_gen_fn(fake_tensor: torch.Tensor) -> torch.Tensor:
    """Default input generator that creates a real tensor matching the fake tensor's shape."""
    return torch.randn(
        fake_tensor.shape, dtype=fake_tensor.dtype, device=fake_tensor.device
    )


def _create_fallback_choice(
    op_overload: torch._ops.OpOverload,
) -> ExternKernelChoice:
    """Create or reuse fallback choice that calls the op eagerly.

    Since kwargs are passed at bind time via maybe_append_choice rather than
    baked into the kernel, the same ExternKernelChoice is reused across
    compilations for the same op_overload.
    """
````
- **EN**: Introduces function `_default_input_gen_fn`, function `_create_fallback_choice`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_default_input_gen_fn`、函数`_create_fallback_choice`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 421-448 / 第 421-448 行
````python
    fallback_name = (
        f"{op_overload.name().replace('::', '_').replace('.', '_')}_fallback"
    )

    existing = ExternKernelChoice.lookup(fallback_name)
    if existing is not None:
        return existing

    return ExternKernelChoice(
        kernel=op_overload,
        name=fallback_name,
        has_out_variant=False,
        op_overload=op_overload,
        use_fallback_kernel=True,
    )


def autotune_custom_op(
    name: str,
    decompositions: list[Callable[..., Any]],
    inputs: list[torch.fx.Node],
    non_tensor_args: list[dict[str, Any]],
    op_overload: torch._ops.OpOverload,
    user_input_gen_fns: dict[str, Callable[[torch.Tensor], torch.Tensor]] | None = None,
    config_patches_list: list[dict[str, Any]] | None = None,
    min_speedup_threshold: float = 1.0,
    benchmark_with_cudagraphs: bool = False,
) -> tuple[TensorBox, ChoiceCaller]:
````
- **EN**: Introduces function `autotune_custom_op`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fallback_name`, `existing`, `kernel`, `name`, `has_out_variant`, `op_overload`, and `...+8`.
- **CN**: 这里定义了函数`autotune_custom_op`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `fallback_name`、`existing`、`kernel`、`name`、`has_out_variant`、`op_overload`、`另有8项` 等值。

### Lines 449-476 / 第 449-476 行
````python
    """Autotune custom operations by comparing multiple decomposition implementations.

    Currently supports SINGLE OUTPUT custom ops only.
    TODO: Add support for multiple output custom ops (tuple/list returns).

    This function generates multiple implementation choices for a custom operation and
    uses Inductor's autotuning system to select the best performing variant at runtime.
    After selecting the best choice, applies inline fusion if the winning choice has a graph.

    Args:
        name: Unique identifier for the autotuning operation
        decompositions: List of alternative implementation functions to benchmark
        inputs: Input tensor IR nodes from compilation (TensorBox/Buffer objects)
        non_tensor_args: List of kwargs dicts, paired with corresponding decompositions arg
        op_overload: OpOverload of the custom op, used as fallback implementation
        user_input_gen_fns: Optional custom input generators for benchmarking.
                           Maps input indices to functions that take fake tensors
                           and return real tensors for performance measurement.

    Returns:
        Tuple of (IR node representing the optimized operation result, winning ChoiceCaller)

    Raises:
        TypeError: If decompositions is not a list/tuple
        RuntimeError: If no inputs or no valid choices generated
    """
    if not isinstance(decompositions, (list, tuple)):
        raise TypeError(
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `TODO`, `Args`, `name`, `decompositions`, `inputs`, `non_tensor_args`, and `...+6`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `TODO`、`Args`、`name`、`decompositions`、`inputs`、`non_tensor_args`、`另有6项` 等值。

### Lines 477-504 / 第 477-504 行
````python
            f"decompositions must be a list or tuple of callables, got {type(decompositions)}"
        )

    if not inputs:
        raise RuntimeError(f"Custom op '{name}' requires tensor inputs for autotuning")

    if len(decompositions) != len(non_tensor_args):
        raise ValueError(
            f"decompositions and non_tensor_args must have same length, "
            f"got {len(decompositions)} decompositions and {len(non_tensor_args)} kwargs"
        )

    # Convert user input generation functions BEFORE creating choices
    input_gen_fns: dict[int, Callable[[Any], torch.Tensor]] = {}
    if user_input_gen_fns:
        input_gen_fns = _adapt_user_input_gen_fns(
            inputs, op_overload, user_input_gen_fns
        )

    template = SubgraphTemplate(name=name)
    choices = template.generate_custom_op_choices(
        name=name,
        decompositions=decompositions,
        # pyrefly: ignore [bad-argument-type, no-matching-overload]
        input_nodes=list(inputs),
        non_tensor_args=non_tensor_args,
        input_gen_fns=input_gen_fns if input_gen_fns else None,
        config_patches_list=config_patches_list,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_gen_fns`, `template`, `choices`, `name`, `decompositions`, `input_nodes`, and `...+2`. This range continues the implementation of function `autotune_custom_op`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_gen_fns`、`template`、`choices`、`name`、`decompositions`、`input_nodes`、`另有2项` 等值。这一段延续了函数`autotune_custom_op` 的具体实现。

### Lines 505-532 / 第 505-532 行
````python
    )

    # Add fallback choice that calls the op eagerly (not through inductor lowering)
    # This provides a baseline to compare decompositions against
    from torch._inductor import config

    fallback_kwargs = non_tensor_args[0] if non_tensor_args else {}

    with V.fake_mode:
        # pyrefly: ignore [no-matching-overload]
        fake_inputs = [ir_node_to_tensor(inp) for inp in inputs]
        fake_output = op_overload(*fake_inputs, **fallback_kwargs)

    output_size = tuple(convert_symint_to_expr(s) for s in fake_output.shape)
    output_stride = tuple(convert_symint_to_expr(s) for s in fake_output.stride())

    fallback_choice = _create_fallback_choice(op_overload)
    fallback_choice.maybe_append_choice(
        choices=choices,
        input_nodes=list(inputs),
        layout=FixedLayout(
            device=fake_output.device,
            dtype=fake_output.dtype,
            size=output_size,
            stride=output_stride,
        ),
        **fallback_kwargs,
    )
````
- **EN**: Imports dependencies such as `torch._inductor` for the logic in this range. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor` 等依赖，为后续逻辑提供基础能力。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。

### Lines 533-560 / 第 533-560 行
````python

    if not choices:
        raise RuntimeError(f"No valid choices generated for {name}")

    is_collective = _detect_collective_ops(choices)

    # Run autotuning and get both result and winning choice
    selected_result, winning_choice = autotune_select_algorithm(
        name=name,
        choices=choices,
        input_nodes=list(inputs),
        layout=choices[0].layout,
        input_gen_fns=input_gen_fns,
        is_collective=is_collective,
        min_speedup_threshold=min_speedup_threshold,
        benchmark_with_cudagraphs=benchmark_with_cudagraphs,
    )

    # Test mode: force specific choice to win
    force_choice = config.test_configs.force_custom_op_decomposition
    if force_choice is True and winning_choice.gm is None:
        # Force decomposition: pick first choice with a graph
        for choice in choices:
            if choice.gm is not None:
                log.info(
                    "Test mode: forcing decomposition %s over fallback",
                    getattr(choice, "name", type(choice).__name__),
                )
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_collective`, `name`, `choices`, `input_nodes`, `layout`, `input_gen_fns`, and `...+3`. This range continues the implementation of function `autotune_custom_op`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_collective`、`name`、`choices`、`input_nodes`、`layout`、`input_gen_fns`、`另有3项` 等值。这一段延续了函数`autotune_custom_op` 的具体实现。

### Lines 561-588 / 第 561-588 行
````python
                winning_choice = choice
                selected_result = choice.output_node()
                break
    elif force_choice is False and winning_choice.gm is not None:
        # Force fallback: pick first choice without a graph
        for choice in choices:
            if choice.gm is None:
                log.info(
                    "Test mode: forcing fallback %s over decomposition",
                    getattr(choice, "name", type(choice).__name__),
                )
                winning_choice = choice
                selected_result = choice.output_node()
                break

        # Always inline when winning_choice has a graph; callers extract choice metadata separately
    if winning_choice.gm is not None:
        log.debug(
            "Inlining winning choice: %s (name=%s)",
            getattr(winning_choice, "name", type(winning_choice).__name__),
            name,
        )
        from torch._inductor.codegen.subgraph import inline_subgraph_to_ir_nodes

        ops_before = len(V.graph.operations)
        result = inline_subgraph_to_ir_nodes(winning_choice.gm, inputs, name)

        # Tag inlined operations with config_patches from the winning choice
````
- **EN**: Imports dependencies such as `torch._inductor.codegen.subgraph` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `winning_choice`, `selected_result`, `ops_before`, and `result`. This range continues the implementation of function `autotune_custom_op`.
- **CN**: 这里导入了 `torch._inductor.codegen.subgraph` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `winning_choice`、`selected_result`、`ops_before`、`result` 等值。这一段延续了函数`autotune_custom_op` 的具体实现。

### Lines 589-616 / 第 589-616 行
````python
        config_patches = winning_choice.config_patches
        if config_patches:
            for op in V.graph.operations[ops_before:]:
                op.set_config_patches(config_patches.copy())

        return result, winning_choice

    log.debug(
        "Winning choice does not support inlining: %s (name=%s)",
        getattr(winning_choice, "name", type(winning_choice).__name__),
        name,
    )
    return selected_result, winning_choice


def _generate_dynamic_configs(
    tensor_inputs: list[Buffer],
    config_generator: Callable[[dict[str, torch.Tensor]], list[CustomOpConfig]],
    op_overload: torch._ops.OpOverload,
    operation_name: str,
) -> list[CustomOpConfig]:
    """Generate configs dynamically based on input tensors at lowering time."""
    # Get parameter names from op schema instead of impl signature
    schema = op_overload._schema
    param_names = [arg.name for arg in schema.arguments if not arg.kwarg_only]

    with V.fake_mode:
        fake_tensors = [ir_node_to_tensor(inp) for inp in tensor_inputs]
````
- **EN**: Introduces function `_generate_dynamic_configs`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `config_patches`, `tensor_inputs`, `config_generator`, `op_overload`, `operation_name`, `schema`, and `...+2`.
- **CN**: 这里定义了函数`_generate_dynamic_configs`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `config_patches`、`tensor_inputs`、`config_generator`、`op_overload`、`operation_name`、`schema`、`另有2项` 等值。

### Lines 617-644 / 第 617-644 行
````python

    fake_tensors_dict = dict(zip(param_names, fake_tensors))

    configs = config_generator(fake_tensors_dict)

    if not isinstance(configs, (list, tuple)):
        raise TypeError(
            f"config_generator must return a list or tuple of CustomOpConfig, "
            f"got {type(configs)}"
        )
    if not configs:
        log.info(
            "config_generator returned empty list for %s, will use default lowering",
            operation_name,
        )
        return []

    return list(configs)


def _prepare_configs_and_decompositions(
    processed_configs: list[CustomOpConfig] | None,
    config_generator: Callable[[dict[str, torch.Tensor]], list[CustomOpConfig]] | None,
    tensor_inputs: list[Any],
    default_impl: Callable[..., Any],
    op_overload: torch._ops.OpOverload,
    runtime_kwargs: dict[str, Any],
    name: str,
````
- **EN**: Introduces function `_prepare_configs_and_decompositions`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fake_tensors_dict`, `configs`, `processed_configs`, `config_generator`, `tensor_inputs`, `default_impl`, and `...+3`.
- **CN**: 这里定义了函数`_prepare_configs_and_decompositions`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `fake_tensors_dict`、`configs`、`processed_configs`、`config_generator`、`tensor_inputs`、`default_impl`、`另有3项` 等值。

### Lines 645-672 / 第 645-672 行
````python
) -> tuple[list[Callable], list[dict[str, Any]], list[dict[str, Any]]]:
    """Prepare decompositions and merged kwargs from configs.

    Handles both static configs and dynamic config generation.
    Merges config params with runtime kwargs (runtime takes precedence).
    """
    # Get configs: either generate dynamically or use static configs
    if config_generator is not None:
        configs_to_use = _generate_dynamic_configs(
            tensor_inputs, config_generator, op_overload, name
        )
    else:
        assert processed_configs is not None
        configs_to_use = processed_configs

    # Prepare decompositions and kwargs for autotuning
    decompositions = []
    non_tensor_args = []
    config_patches_list = []

    for cfg in configs_to_use:
        decomp = cfg.get_decomposition(default_impl=default_impl)
        decompositions.append(decomp)

        # Merge config params with runtime kwargs (runtime takes precedence)
        merged_kwargs = _merge_config_and_runtime_kwargs(cfg.params, runtime_kwargs)
        non_tensor_args.append(merged_kwargs)

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `configs_to_use`, `else`, `decompositions`, `non_tensor_args`, `config_patches_list`, `decomp`, and `...+1`. This range continues the implementation of function `_prepare_configs_and_decompositions`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `configs_to_use`、`else`、`decompositions`、`non_tensor_args`、`config_patches_list`、`decomp`、`另有1项` 等值。这一段延续了函数`_prepare_configs_and_decompositions` 的具体实现。

### Lines 673-700 / 第 673-700 行
````python
        # Collect config_patches for each config
        config_patches_list.append(cfg.config_patches)

    return decompositions, non_tensor_args, config_patches_list


def _standard_lowering_fn(
    processed_configs: list[CustomOpConfig],
    default_impl: Callable[..., Any],
    name: str,
    op_overload: torch._ops.OpOverload,
    input_gen_fns: dict[str, Callable[[torch.Tensor], torch.Tensor]] | None,
    tensor_inputs: list[Any],
    runtime_kwargs: dict[str, Any],
    config_generator: Callable[[dict[str, torch.Tensor]], list[CustomOpConfig]]
    | None = None,
    min_speedup_threshold: float = 1.0,
    benchmark_with_cudagraphs: bool = False,
) -> Any:
    """Standard autotuning lowering function.

    Returns None if no configs/decompositions available, signaling caller to
    use normal lowering.
    """
    decompositions, non_tensor_args, config_patches_list = (
        _prepare_configs_and_decompositions(
            processed_configs,
            config_generator,
````
- **EN**: Introduces function `_standard_lowering_fn`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `processed_configs`, `default_impl`, `name`, `op_overload`, `input_gen_fns`, `tensor_inputs`, and `...+4`.
- **CN**: 这里定义了函数`_standard_lowering_fn`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `processed_configs`、`default_impl`、`name`、`op_overload`、`input_gen_fns`、`tensor_inputs`、`另有4项` 等值。

### Lines 701-728 / 第 701-728 行
````python
            tensor_inputs,
            default_impl,
            op_overload,
            runtime_kwargs,
            name,
        )
    )

    # If no decompositions, signal caller to use normal lowering
    if not decompositions:
        return None

    result, _ = autotune_custom_op(
        name=name,
        decompositions=decompositions,
        inputs=tensor_inputs,
        non_tensor_args=non_tensor_args,
        config_patches_list=config_patches_list,
        op_overload=op_overload,
        user_input_gen_fns=input_gen_fns,
        min_speedup_threshold=min_speedup_threshold,
        benchmark_with_cudagraphs=benchmark_with_cudagraphs,
    )

    validate_ir(result)
    return result


````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `decompositions`, `inputs`, `non_tensor_args`, `config_patches_list`, `op_overload`, and `...+3`. This range continues the implementation of function `_standard_lowering_fn`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`decompositions`、`inputs`、`non_tensor_args`、`config_patches_list`、`op_overload`、`另有3项` 等值。这一段延续了函数`_standard_lowering_fn` 的具体实现。

### Lines 729-756 / 第 729-756 行
````python
def _apply_config_patches_recursive(
    operations: list,
    config_patches: dict[str, Any],
) -> None:
    """Apply config_patches to operations, including those inside subgraphs."""
    for op in operations:
        if hasattr(op, "set_config_patches"):
            op.set_config_patches(config_patches.copy())

        # Recurse into any subgraphs (Conditional, WhileLoop, InvokeSubgraph, etc.)
        for subgraph in op.get_subgraphs():
            if subgraph.graph:
                _apply_config_patches_recursive(
                    subgraph.graph.operations, config_patches
                )


def _lower_single_impl(
    impl: Callable[..., Any],
    impl_kwargs: dict[str, Any],
    runtime_kwargs: dict[str, Any],
    tensor_inputs: list[Any],
    name: str,
    config_patches: dict[str, Any] | None = None,
) -> Any:
    """Lower a single implementation by tracing and inlining it.

    Uses error_on_new_guards() during tracing to detect if the impl adds guards.
````
- **EN**: Introduces function `_apply_config_patches_recursive`, function `_lower_single_impl`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `operations`, `config_patches`, `impl`, `impl_kwargs`, `runtime_kwargs`, `tensor_inputs`, and `...+1`.
- **CN**: 这里定义了函数`_apply_config_patches_recursive`、函数`_lower_single_impl`。包含分支、循环或上下文管理等控制流。初始化或更新了 `operations`、`config_patches`、`impl`、`impl_kwargs`、`runtime_kwargs`、`tensor_inputs`、`另有1项` 等值。

### Lines 757-784 / 第 757-784 行
````python
    Returns None if the impl adds guards, signaling caller to skip this choice.
    """
    from torch._inductor.codegen.subgraph import inline_subgraph_to_ir_nodes
    from torch.fx.experimental.proxy_tensor import make_fx
    from torch.fx.experimental.symbolic_shapes import _ShapeEnvGuardError

    from ..decomposition import select_decomp_table

    merged_kwargs = _merge_config_and_runtime_kwargs(impl_kwargs, runtime_kwargs)

    def impl_wrapper(*tensors):
        return impl(*tensors, **merged_kwargs)

    shape_env = V.fake_mode.shape_env

    with V.fake_mode:
        fake_inputs = tuple(ir_node_to_tensor(inp) for inp in tensor_inputs)
        decomposition_table = select_decomp_table()

        context = (
            shape_env.error_on_new_guards
            if shape_env is not None
            else contextlib.nullcontext
        )
        try:
            with context():
                impl_gm = make_fx(
                    impl_wrapper,
````
- **EN**: Imports dependencies such as `torch._inductor.codegen.subgraph`, `torch.fx.experimental.proxy_tensor`, `torch.fx.experimental.symbolic_shapes`, and `..decomposition` for the logic in this range. Introduces function `impl_wrapper`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor.codegen.subgraph`、`torch.fx.experimental.proxy_tensor`、`torch.fx.experimental.symbolic_shapes`、`..decomposition` 等依赖，为后续逻辑提供基础能力。这里定义了函数`impl_wrapper`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 785-812 / 第 785-812 行
````python
                    decomposition_table=decomposition_table,
                    tracing_mode="symbolic",
                )(*fake_inputs)
        except (_ShapeEnvGuardError, AssertionError) as e:
            is_guard_error = isinstance(e, _ShapeEnvGuardError) or (
                isinstance(e, AssertionError)
                and "Guard attempted while ShapeEnv guards are frozen" in str(e)
            )
            if not is_guard_error:
                raise
            log.info(
                "Implementation %s adds guards, skipping custom op lowering",
                impl.__name__,
            )
            counters["inductor"]["custom_op_decomp_guard_skips"] += 1
            return None

    log.info("Inlining implementation: %s", impl.__name__)
    ops_before = len(V.graph.operations)
    result = inline_subgraph_to_ir_nodes(impl_gm, tensor_inputs, name)

    if config_patches:
        _apply_config_patches_recursive(V.graph.operations[ops_before:], config_patches)

    validate_ir(result)
    return result


````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `decomposition_table`, `tracing_mode`, `is_guard_error`, `ops_before`, and `result`. This range continues the implementation of function `_lower_single_impl`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `decomposition_table`、`tracing_mode`、`is_guard_error`、`ops_before`、`result` 等值。这一段延续了函数`_lower_single_impl` 的具体实现。

### Lines 813-840 / 第 813-840 行
````python
def _range_based_lowering_fn(
    processed_configs: list[CustomOpConfig],
    default_impl: Callable[..., Any],
    name: str,
    op_overload: torch._ops.OpOverload,
    input_gen_fns: dict[str, Callable[[torch.Tensor], torch.Tensor]] | None,
    tensor_name: str,
    dim_index: int,
    ranges: list[tuple[int, int | float]],
    tensor_inputs: list[Any],
    runtime_kwargs: dict[str, Any],
    range_upper_bound: int,
    config_generator: Callable[[dict[str, torch.Tensor]], list[CustomOpConfig]]
    | None = None,
    min_speedup_threshold: float = 1.0,
    benchmark_with_cudagraphs: bool = False,
) -> Any:
    """Range-based autotuning lowering function."""
    from torch._inductor.codegen.subgraph import inline_subgraph_to_ir_nodes
    from torch.fx.experimental.proxy_tensor import make_fx

    from ..decomposition import select_decomp_table

    log.info("=== Range-based Autotuning for %s ===", name)
    log.info("Dispatch on: %s[%d], Ranges: %s", tensor_name, dim_index, ranges)

    decompositions, non_tensor_args, config_patches_list = (
        _prepare_configs_and_decompositions(
````
- **EN**: Imports dependencies such as `torch._inductor.codegen.subgraph`, `torch.fx.experimental.proxy_tensor`, and `..decomposition` for the logic in this range. Introduces function `_range_based_lowering_fn`. Initializes or updates values such as `processed_configs`, `default_impl`, `name`, `op_overload`, `input_gen_fns`, `tensor_name`, and `...+8`.
- **CN**: 这里导入了 `torch._inductor.codegen.subgraph`、`torch.fx.experimental.proxy_tensor`、`..decomposition` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_range_based_lowering_fn`。初始化或更新了 `processed_configs`、`default_impl`、`name`、`op_overload`、`input_gen_fns`、`tensor_name`、`另有8项` 等值。

### Lines 841-868 / 第 841-868 行
````python
            processed_configs,
            config_generator,
            tensor_inputs,
            default_impl,
            op_overload,
            runtime_kwargs,
            name,
        )
    )

    range_to_best_impl_map: dict[RangeBounds, ImplConfig] = {}

    # Benchmark each range and collect winning implementations
    for range_start, range_end in ranges:
        if input_gen_fns and tensor_name in input_gen_fns:
            base_gen_fn = input_gen_fns[tensor_name]
        else:
            base_gen_fn = _default_input_gen_fn

        range_gen_fn = _create_range_input_gen_fn(
            base_gen_fn, dim_index, range_start, range_end, range_upper_bound
        )
        range_input_gen_fns = {**(input_gen_fns or {}), tensor_name: range_gen_fn}

        range_name = f"{name}_range_{int(range_start)}_{int(range_end) if range_end != float('inf') else 'inf'}"

        # pyrefly: ignore [not-iterable]
        autotuned_result, winning_choice = autotune_custom_op(
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `range_to_best_impl_map`, `base_gen_fn`, `else`, `range_gen_fn`, `range_input_gen_fns`, and `range_name`. This range continues the implementation of function `_range_based_lowering_fn`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。包含分支、循环或上下文管理等控制流。初始化或更新了 `range_to_best_impl_map`、`base_gen_fn`、`else`、`range_gen_fn`、`range_input_gen_fns`、`range_name` 等值。这一段延续了函数`_range_based_lowering_fn` 的具体实现。

### Lines 869-896 / 第 869-896 行
````python
            name=range_name,
            decompositions=decompositions,
            inputs=tensor_inputs,
            non_tensor_args=non_tensor_args,
            op_overload=op_overload,
            user_input_gen_fns=range_input_gen_fns,
            min_speedup_threshold=min_speedup_threshold,
            benchmark_with_cudagraphs=benchmark_with_cudagraphs,
            config_patches_list=config_patches_list,
        )

        if winning_choice.decomposition is not None:
            winning_impl = winning_choice.decomposition
            winning_kwargs = winning_choice.decomposition_kwargs
        else:
            # Fallback was selected (ExternKernelCaller)
            winning_impl = default_impl
            winning_kwargs = non_tensor_args[0] if non_tensor_args else {}
            log.info(
                "   Range [%s, %s]: Fallback (default_impl) selected",
                range_start,
                range_end if range_end != float("inf") else "inf",
            )

        winning_config_patches = winning_choice.config_patches or {}

        # Create dataclass instances for cleaner code
        range_bounds = RangeBounds(range_start, range_end)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `decompositions`, `inputs`, `non_tensor_args`, `op_overload`, `user_input_gen_fns`, and `...+8`. This range continues the implementation of function `_range_based_lowering_fn`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`decompositions`、`inputs`、`non_tensor_args`、`op_overload`、`user_input_gen_fns`、`另有8项` 等值。这一段延续了函数`_range_based_lowering_fn` 的具体实现。

### Lines 897-924 / 第 897-924 行
````python
        impl_config = ImplConfig(
            impl_name=winning_impl.__name__,
            impl_func=winning_impl,
            kwargs=winning_kwargs,
            config_patches=winning_config_patches,
        )
        range_to_best_impl_map[range_bounds] = impl_config

        log.info(
            "   Range %s -> %s",
            range_bounds,
            impl_config.impl_name,
        )

    # Group ranges by implementation
    from torch.fx.experimental.symbolic_shapes import _ShapeEnvGuardError

    impl_groups = _group_ranges_by_impl(range_to_best_impl_map)

    log.info("After grouping by implementation: %d impl groups", len(impl_groups))
    for group in impl_groups:
        log.info("   %s", group)

    # If only one impl group remains, just inline that implementation
    if len(impl_groups) == 1:
        group = impl_groups[0]
        log.info("Only one implementation after grouping, directly inlining")
        return _lower_single_impl(
````
- **EN**: Imports dependencies such as `torch.fx.experimental.symbolic_shapes` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `impl_config`, `impl_name`, `impl_func`, `kwargs`, `config_patches`, `impl_groups`, and `...+1`.
- **CN**: 这里导入了 `torch.fx.experimental.symbolic_shapes` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `impl_config`、`impl_name`、`impl_func`、`kwargs`、`config_patches`、`impl_groups`、`另有1项` 等值。

### Lines 925-952 / 第 925-952 行
````python
            group.impl_func,
            group.impl_kwargs,
            runtime_kwargs,
            tensor_inputs,
            name,
            config_patches=group.config_patches,
        )

    def dispatch_fn(*fake_tensors):
        """Build nested torch.cond dispatch: cond(pred1, impl1, cond(pred2, impl2, ...))."""
        num_impl_groups = len(impl_groups)
        if num_impl_groups < 2:
            raise RuntimeError(
                f"dispatch_fn requires at least 2 impl groups, got {num_impl_groups}"
            )

        dim_value = fake_tensors[0].size(dim_index)

        def build_range_predicate(ranges_list: list[RangeBounds]) -> torch.Tensor:
            predicates = []
            for rb in ranges_list:
                end = int(rb.end) if rb.end != float("inf") else None
                if end is None:
                    predicates.append(dim_value >= rb.start)
                else:
                    predicates.append((dim_value >= rb.start) & (dim_value <= end))

            result = predicates[0]
````
- **EN**: Introduces function `dispatch_fn`, function `build_range_predicate`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `config_patches`, `num_impl_groups`, `dim_value`, `predicates`, `end`, `else`, and `...+1`.
- **CN**: 这里定义了函数`dispatch_fn`、函数`build_range_predicate`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `config_patches`、`num_impl_groups`、`dim_value`、`predicates`、`end`、`else`、`另有1项` 等值。

### Lines 953-980 / 第 953-980 行
````python
            for pred in predicates[1:]:
                result = result | pred
            return result  # pyrefly: ignore [bad-return]

        def build_nested_cond(idx: int):
            if idx >= num_impl_groups:
                raise RuntimeError(f"Invalid impl group index: {idx}")

            group = impl_groups[idx]
            merged_kwargs = _merge_config_and_runtime_kwargs(
                group.impl_kwargs, runtime_kwargs
            )

            @torch._dynamo.dont_skip_tracing
            def group_fn(*ops):
                return group.impl_func(*ops, **merged_kwargs)

            if idx == num_impl_groups - 1:
                return group_fn

            next_fn = build_nested_cond(idx + 1)

            @torch._dynamo.dont_skip_tracing
            def cond_wrapper(*ops, _ranges=group.ranges):
                return torch.cond(
                    pred=build_range_predicate(_ranges),
                    true_fn=group_fn,
                    false_fn=next_fn,
````
- **EN**: Introduces function `build_nested_cond`, function `group_fn`, function `cond_wrapper`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`build_nested_cond`、函数`group_fn`、函数`cond_wrapper`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 981-1008 / 第 981-1008 行
````python
                    operands=ops,
                )

            return cond_wrapper

        return build_nested_cond(0)(*fake_tensors)

    with V.fake_mode:
        fake_inputs = tuple(ir_node_to_tensor(inp) for inp in tensor_inputs)
        decomposition_table = select_decomp_table()
        shape_env = V.fake_mode.shape_env

        log.info("Tracing torch.cond dispatch with symbolic shapes...")

        try:
            context = (
                shape_env.error_on_new_guards
                if shape_env is not None
                else contextlib.nullcontext
            )
            with context():
                dispatch_gm = make_fx(
                    dispatch_fn,
                    decomposition_table=decomposition_table,
                    tracing_mode="symbolic",
                )(*fake_inputs)

            log.info("Successfully traced torch.cond dispatch")
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `operands`, `fake_inputs`, `decomposition_table`, `shape_env`, `try`, `context`, and `...+2`. This range continues the implementation of function `_range_based_lowering_fn`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `operands`、`fake_inputs`、`decomposition_table`、`shape_env`、`try`、`context`、`另有2项` 等值。这一段延续了函数`_range_based_lowering_fn` 的具体实现。

### Lines 1009-1036 / 第 1009-1036 行
````python
            log.info("Traced graph:\n%s", dispatch_gm.graph)

        except (_ShapeEnvGuardError, AssertionError) as e:
            is_guard_error = isinstance(e, _ShapeEnvGuardError) or (
                isinstance(e, AssertionError)
                and "Guard attempted while ShapeEnv guards are frozen" in str(e)
            )
            if not is_guard_error:
                raise
            log.info("Dispatch function adds guards, skipping custom op lowering")
            counters["inductor"]["custom_op_decomp_guard_skips"] += 1
            return None

        except Exception:
            log.exception("make_fx tracing FAILED")
            raise

    ops_before = len(V.graph.operations)
    result = inline_subgraph_to_ir_nodes(dispatch_gm, tensor_inputs, f"{name}_dispatch")

    # Apply config_patches from all impl groups to inlined operations
    # TODO - consider conflicting patches
    merged_patches: dict[str, Any] = {}
    for group in impl_groups:
        merged_patches.update(group.config_patches)
    if merged_patches:
        _apply_config_patches_recursive(V.graph.operations[ops_before:], merged_patches)

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_guard_error`, `ops_before`, `result`, and `merged_patches`. This range continues the implementation of function `_range_based_lowering_fn`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_guard_error`、`ops_before`、`result`、`merged_patches` 等值。这一段延续了函数`_range_based_lowering_fn` 的具体实现。

### Lines 1037-1064 / 第 1037-1064 行
````python
    log.info(
        "Successfully created torch.cond dispatch for %d impl groups", len(impl_groups)
    )

    validate_ir(result)
    return result


def _create_autotuning_lowering(
    processed_configs: list[CustomOpConfig],
    default_impl: Callable[..., Any],
    name: str,
    op_overload: torch._ops.OpOverload,
    input_gen_fns: dict[str, Callable[[torch.Tensor], torch.Tensor]] | None,
    range_upper_bound: int,
    is_range_based: bool = False,
    config_generator: Callable[[dict[str, torch.Tensor]], list[CustomOpConfig]]
    | None = None,
    dispatch_on: tuple[str, int] | None = None,
    split_points: list[int] | None = None,
    min_speedup_threshold: float = 1.0,
    benchmark_with_cudagraphs: bool = False,
) -> Callable[..., Any]:
    """Create the lowering function for autotuning."""
    if not is_range_based:
        # Standard autotuning path
        @functools.wraps(op_overload)
        def standard_lowering_wrapper(*args: Any, **kwargs: Any) -> Any:
````
- **EN**: Introduces function `_create_autotuning_lowering`, function `standard_lowering_wrapper`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_create_autotuning_lowering`、函数`standard_lowering_wrapper`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1065-1092 / 第 1065-1092 行
````python
            tensor_inputs, runtime_kwargs = _extract_tensor_inputs(
                args, kwargs, op_overload
            )
            return _standard_lowering_fn(
                processed_configs=processed_configs,
                default_impl=default_impl,
                name=name,
                op_overload=op_overload,
                input_gen_fns=input_gen_fns,
                tensor_inputs=tensor_inputs,
                runtime_kwargs=runtime_kwargs,
                config_generator=config_generator,
                min_speedup_threshold=min_speedup_threshold,
                benchmark_with_cudagraphs=benchmark_with_cudagraphs,
            )

        return standard_lowering_wrapper

    # Range-based autotuning path
    # pyrefly: ignore [not-iterable]
    tensor_name, dim_index = dispatch_on
    # pyrefly: ignore [bad-argument-type]
    ranges = _create_ranges_from_split_points(split_points)

    @functools.wraps(op_overload)
    def range_based_lowering_wrapper(*args: Any, **kwargs: Any) -> Any:
        tensor_inputs, runtime_kwargs = _extract_tensor_inputs(
            args, kwargs, op_overload
````
- **EN**: Introduces function `range_based_lowering_wrapper`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `processed_configs`, `default_impl`, `name`, `op_overload`, `input_gen_fns`, `tensor_inputs`, and `...+5`.
- **CN**: 这里定义了函数`range_based_lowering_wrapper`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `processed_configs`、`default_impl`、`name`、`op_overload`、`input_gen_fns`、`tensor_inputs`、`另有5项` 等值。

### Lines 1093-1120 / 第 1093-1120 行
````python
        )
        return _range_based_lowering_fn(
            processed_configs=processed_configs,
            default_impl=default_impl,
            name=name,
            op_overload=op_overload,
            input_gen_fns=input_gen_fns,
            tensor_name=tensor_name,
            dim_index=dim_index,
            ranges=ranges,
            tensor_inputs=tensor_inputs,
            runtime_kwargs=runtime_kwargs,
            range_upper_bound=range_upper_bound,
            config_generator=config_generator,
            min_speedup_threshold=min_speedup_threshold,
            benchmark_with_cudagraphs=benchmark_with_cudagraphs,
        )

    return range_based_lowering_wrapper


def register_custom_op_autotuning(
    custom_op: torch._library.custom_ops.CustomOpDef | torch._ops.OpOverload,
    configs: list[CustomOpConfig] | list[Callable[..., Any]] | None = None,
    config_generator: Callable[[dict[str, torch.Tensor]], list[CustomOpConfig]]
    | None = None,
    name: str | None = None,
    input_gen_fns: dict[str, Callable[[torch.Tensor], torch.Tensor]] | None = None,
````
- **EN**: Introduces function `register_custom_op_autotuning`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `processed_configs`, `default_impl`, `name`, `op_overload`, `input_gen_fns`, `tensor_name`, and `...+10`.
- **CN**: 这里定义了函数`register_custom_op_autotuning`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `processed_configs`、`default_impl`、`name`、`op_overload`、`input_gen_fns`、`tensor_name`、`另有10项` 等值。

### Lines 1121-1148 / 第 1121-1148 行
````python
    dispatch_on: dict[str, Any] | None = None,
    split_points: list[int] | None = None,
    min_speedup_threshold: float = 1.0,
    benchmark_with_cudagraphs: bool = False,
) -> None:
    """Register custom op for autotuning with custom_op configs where each config
    specifies a decomposition implementation function with its parameter values.
    It also supports Range-based autotuning to benchmark per range and generate
    runtime dispatch.

    Args:
        custom_op: Custom operation (CustomOpDef from @torch.library.custom_op) or
                   OpOverload (e.g., torch.ops.aten.mm.default)
        configs: List of CustomOpConfig objects for static inputs. Mutually exclusive with config_generator.
        config_generator: Dynamic config generator function that takes a dict mapping
                          parameter names to fake tensors, and returns list[CustomOpConfig]
                          based on input tensor properties. Mutually exclusive with configs.
        name: Operation name (default: "{op_name}_autotuned")
        input_gen_fns: Custom input generators for benchmarking
        dispatch_on: Dict for range-based dispatch with keys:
            - 'tensor_name': Name of tensor parameter to dispatch on
            - 'dim': Dimension index to check size
            - 'unbounded_size' (optional): Benchmark size for the unbounded (last) range, such
                as [2048, inf] -> [2048, unbounded_size]. Set based on your expected workload size.
                Default is DEFAULT_RANGE_UPPER_BOUND=65536.
        split_points: List of range endpoints in ascending order for range-based autotuning
        min_speedup_threshold: Only pick a non-fallback choice if it beats the fallback
            by at least this ratio. Default is 1.0 (any speedup wins). Set to e.g. 1.1
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Initializes or updates values such as `dispatch_on`, `split_points`, `min_speedup_threshold`, `benchmark_with_cudagraphs`, `Args`, `custom_op`, and `...+4`. This range continues the implementation of function `register_custom_op_autotuning`.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。初始化或更新了 `dispatch_on`、`split_points`、`min_speedup_threshold`、`benchmark_with_cudagraphs`、`Args`、`custom_op`、`另有4项` 等值。这一段延续了函数`register_custom_op_autotuning` 的具体实现。

### Lines 1149-1176 / 第 1149-1176 行
````python
            to require 10% speedup over fallback.
        benchmark_with_cudagraphs: If True, benchmark the fallback kernel using CUDA graph
            capture and replay for fair comparison with compiled kernels. Default is False.

    The default/fallback implementation is automatically derived:
    - For CustomOpDef: Uses the decorated function
    - For OpOverload: Traces the op call, which falls through to normal inductor lowering

    Examples:
        # Static configs
        @torch.library.custom_op("mylib::attention", mutates_args=())
        def my_attention(query, key, value, head_dim=32):
            ...

        register_custom_op_autotuning(
            my_attention,
            configs=[
                CustomOpConfig(attention_impl, head_dim=32, method='chunked'),
                CustomOpConfig(attention_impl, head_dim=64, method='tiled'),
                CustomOpConfig(head_dim=128),  # No decomposition specified, use default
            ],
            input_gen_fns={
                "query": lambda fake: torch.randn_like(fake, device='cuda'),
                "key": lambda fake: torch.randn_like(fake, device='cuda'),
                "value": lambda fake: torch.randn_like(fake, device='cuda'),
            },
        )

````
- **EN**: Introduces function `my_attention`. Applies decorators to register behavior or alter how the following definition is constructed. Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`my_attention`。使用装饰器来注册行为，或改变后续定义的构造方式。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。

### Lines 1177-1204 / 第 1177-1204 行
````python
        # Dynamic config generation based on input tensor properties
        def generate_k_split_configs(fake_tensors: dict[str, torch.Tensor]) -> list[CustomOpConfig]:
            # Access tensor shapes, dtypes, devices, etc.
            m, k = fake_tensors["mat1"].shape
            _, n = fake_tensors["mat2"].shape
            k_splits = ... # compute possible k splits based on tensor properties
            return [CustomOpConfig(k_splits=k) for k in k_splits]

        register_custom_op_autotuning(
            matmul_decomposeK_op,
            config_generator=generate_k_split_configs,
            input_gen_fns={...},
        )

    Range-based Example:
        register_custom_op_autotuning(
            my_op,
            configs=[CustomOpConfig(impl1), CustomOpConfig(impl2), CustomOpConfig(impl3)],
            dispatch_on={
                # Dispatch based on x.shape[1]
                "tensor_name": "x",
                "dim": 1,
                # Optional Benchmark size used for the unbounded range [2049, inf].
                # Since inf is not a concrete value, we use range_upper_bound as the benchmark size.
                # Default value is 65536 (DEFAULT_RANGE_UPPER_BOUND) if not provided.
                "range_upper_bound": 8192,
            },
            split_points=[512, 2048],  # Creates ranges: [1,512], [513,2048], [2049, 8192]
````
- **EN**: Introduces function `generate_k_split_configs`. Processes profiling, FLOP, or benchmark information to characterize execution cost. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`generate_k_split_configs`。处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1205-1232 / 第 1205-1232 行
````python
        )
    """
    from torch._library.custom_ops import CustomOpDef

    # Handle both CustomOpDef and OpOverload - derive impl_fn automatically
    # Both cases call through op_overload so fake handlers are used during tracing
    if isinstance(custom_op, CustomOpDef):
        op_overload = custom_op._opoverload
    elif isinstance(custom_op, torch._ops.OpOverload):
        op_overload = custom_op
    else:
        raise TypeError(
            f"custom_op must be a CustomOpDef or OpOverload, got {type(custom_op)}."
        )

    # impl_fn calls through op_overload so fake handlers are used during tracing
    def impl_fn(*args, **kwargs):
        return op_overload(*args, **kwargs)

    # Validate configs and config_generator are mutually exclusive
    if configs is not None and config_generator is not None:
        raise ValueError(
            "Cannot specify both 'configs' and 'config_generator'. "
            "Use 'config_generator' for shape-dependent configs."
        )

    if configs is None and config_generator is None:
        raise ValueError("Must specify either 'configs' or 'config_generator'")
````
- **EN**: Imports dependencies such as `torch._library.custom_ops` for the logic in this range. Introduces function `impl_fn`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._library.custom_ops` 等依赖，为后续逻辑提供基础能力。这里定义了函数`impl_fn`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1233-1260 / 第 1233-1260 行
````python

    # Process and validate static configs at registration time
    static_configs = None
    if configs is not None:
        if not isinstance(configs, (list, tuple)):
            raise TypeError(f"configs must be a list or tuple, got {type(configs)}")

        static_configs = []
        for cfg in configs:
            if isinstance(cfg, CustomOpConfig):
                static_configs.append(cfg)
            else:
                raise TypeError(
                    f"Each config must be a CustomOpConfig object, got {type(cfg)}"
                )

        if not static_configs:
            raise ValueError("At least one config must be provided")

    if name is None:
        name = f"{op_overload._name}_autotuned"

    # Validate range-based parameters
    is_range_based = dispatch_on is not None or split_points is not None
    dispatch_on_tuple: tuple[str, int] | None = None
    range_upper_bound = DEFAULT_RANGE_UPPER_BOUND
    if is_range_based:
        if dispatch_on is None or split_points is None:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `static_configs`, `else`, `name`, `is_range_based`, `dispatch_on_tuple`, and `range_upper_bound`. This range continues the implementation of function `register_custom_op_autotuning`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `static_configs`、`else`、`name`、`is_range_based`、`dispatch_on_tuple`、`range_upper_bound` 等值。这一段延续了函数`register_custom_op_autotuning` 的具体实现。

### Lines 1261-1288 / 第 1261-1288 行
````python
            raise ValueError(
                "Both dispatch_on and split_points must be specified for range-based autotuning"
            )
        if not isinstance(dispatch_on, dict):
            raise ValueError(
                "dispatch_on must be a dict with 'tensor_name' and 'dim' keys, "
                f"e.g., {{'tensor_name': 'x', 'dim': 1}}. Got: {type(dispatch_on)}"
            )
        if "tensor_name" not in dispatch_on or "dim" not in dispatch_on:
            raise ValueError(
                "dispatch_on must contain 'tensor_name' and 'dim' keys, "
                f"e.g., {{'tensor_name': 'x', 'dim': 1}}. Got keys: {list(dispatch_on.keys())}"
            )
        if not isinstance(dispatch_on["tensor_name"], str):
            raise ValueError(
                f"dispatch_on['tensor_name'] must be a string (tensor parameter name), "
                f"got {type(dispatch_on['tensor_name'])}"
            )
        if not isinstance(dispatch_on["dim"], int):
            raise ValueError(
                f"dispatch_on['dim'] must be an integer (dimension index), "
                f"got {type(dispatch_on['dim'])}"
            )
        dispatch_on_tuple = (dispatch_on["tensor_name"], dispatch_on["dim"])
        range_upper_bound = dispatch_on.get(
            "range_upper_bound", DEFAULT_RANGE_UPPER_BOUND
        )
        if not isinstance(range_upper_bound, int) or range_upper_bound <= 0:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `dispatch_on_tuple`, and `range_upper_bound`. This range continues the implementation of function `register_custom_op_autotuning`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `dispatch_on_tuple`、`range_upper_bound` 等值。这一段延续了函数`register_custom_op_autotuning` 的具体实现。

### Lines 1289-1316 / 第 1289-1316 行
````python
            raise ValueError(
                f"dispatch_on['range_upper_bound'] must be a positive integer, "
                f"got {range_upper_bound}"
            )
        if not isinstance(split_points, list) or len(split_points) == 0:
            raise ValueError("split_points must be a non-empty list of integers")
        if sorted(split_points) != split_points:
            raise ValueError("split_points must be sorted in ascending order")

    # Create and register the lowering function
    lowering_fn = _create_autotuning_lowering(
        # pyrefly: ignore [bad-argument-type]
        processed_configs=static_configs,
        default_impl=impl_fn,
        name=name,
        op_overload=op_overload,
        input_gen_fns=input_gen_fns,
        is_range_based=is_range_based,
        config_generator=config_generator,
        dispatch_on=dispatch_on_tuple,
        split_points=split_points,
        range_upper_bound=range_upper_bound,
        min_speedup_threshold=min_speedup_threshold,
        benchmark_with_cudagraphs=benchmark_with_cudagraphs,
    )

    # Register in user_lowerings which takes priority over built-in lowerings
    # The dispatch in graph.py checks user_lowerings first with recursion guard
````
- **EN**: Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `lowering_fn`, `processed_configs`, `default_impl`, `name`, `op_overload`, `input_gen_fns`, and `...+7`.
- **CN**: 构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `lowering_fn`、`processed_configs`、`default_impl`、`name`、`op_overload`、`input_gen_fns`、`另有7项` 等值。

### Lines 1317-1317 / 第 1317-1317 行
````python
    user_lowerings[op_overload] = lowering_fn
````
- **EN**: This range continues the implementation of function `register_custom_op_autotuning`.
- **CN**: 这一段延续了函数`register_custom_op_autotuning` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `RangeBounds`, `ImplConfig`, `RangeImplGroup`, and `CustomOpConfig`  
  **CN**: 主要类：`RangeBounds`、`ImplConfig`、`RangeImplGroup`、`CustomOpConfig`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `functools`, `logging`, `collections.abc`, `dataclasses`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._dynamo.utils`, `torch._inductor.codegen.subgraph`, `torch._inductor.ir`, `torch._inductor.lowering`, `torch._inductor.select_algorithm`, `torch._inductor.utils`, `torch._inductor.virtualized`, `torch._inductor`, `torch._inductor.kernel.flex.common`, `torch.fx.experimental.proxy_tensor`, `torch.fx.experimental.symbolic_shapes`, `..decomposition`, `torch._library.custom_ops`
