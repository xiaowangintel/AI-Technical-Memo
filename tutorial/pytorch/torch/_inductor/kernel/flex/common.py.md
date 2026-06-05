# common.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel/flex/common.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It exposes functions such as `zeros_and_scatter_lowering`, `get_fwd_subgraph_outputs`, `build_subgraph_module_buffer`, `build_subgraph_buffer`, `maybe_realize`, `freeze_irnodes`, and `...+9`. Module note: Common utilities and functions for flex attention kernels
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。同时提供 `zeros_and_scatter_lowering`、`get_fwd_subgraph_outputs`、`build_subgraph_module_buffer`、`build_subgraph_buffer`、`maybe_realize`、`freeze_irnodes`、`另有9项` 等函数。模块文档字符串给出了额外背景说明。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
"""Common utilities and functions for flex attention kernels"""

import math
from collections.abc import Sequence
from functools import partial
from pathlib import Path
from typing import Any, TYPE_CHECKING

import sympy

import torch
from torch._inductor.virtualized import V
from torch.utils._ordered_set import OrderedSet
from torch.utils._pytree import tree_map, tree_map_only


if TYPE_CHECKING:
    from torch._inductor.codegen.cuda_combined_scheduling import _IntLike
else:
````
- **EN**: Imports dependencies such as `math`, `collections.abc`, `functools`, `pathlib`, `typing`, `sympy`, and `...+5` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `math`、`collections.abc`、`functools`、`pathlib`、`typing`、`sympy`、`另有5项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。

### Lines 21-40 / 第 21-40 行
````python
    _IntLike = int | sympy.Expr


from ...ir import (
    ComputedBuffer,
    ExternKernel,
    FixedLayout,
    FlexibleLayout,
    get_fill_order,
    InputBuffer,
    IRNode,
    MutationLayoutSHOULDREMOVE,
    Scatter,
    StorageBox,
    Subgraph,
    TensorBox,
)
from ...lowering import (
    _full,
    check_and_broadcast_indices,
````
- **EN**: Imports dependencies such as `...ir`, and `...lowering` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `_IntLike`.
- **CN**: 这里导入了 `...ir`、`...lowering` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `_IntLike` 等值。

### Lines 41-60 / 第 41-60 行
````python
    expand,
    index_output_size_and_inner_fn,
    to_dtype,
)
from ...select_algorithm import realize_inputs
from ...utils import load_template


SubgraphResults = list[ComputedBuffer | None] | ComputedBuffer | None


def zeros_and_scatter_lowering(shape: list[int], indices, values):
    """To support backwards on captured buffers we register a specific lowering for our specific custom up"""
    # Always accumulate into fp32 then cast
    grad = _full(0, values.get_device(), torch.float32, shape)
    assert isinstance(grad, TensorBox)
    grad.realize()
    x_size = grad.get_size()
    values = to_dtype(values, grad.get_dtype())
    indices_loaders = [i.make_loader() if i is not None else None for i in indices]
````
- **EN**: Imports dependencies such as `...select_algorithm`, and `...utils` for the logic in this range. Introduces function `zeros_and_scatter_lowering`. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `...select_algorithm`、`...utils` 等依赖，为后续逻辑提供基础能力。这里定义了函数`zeros_and_scatter_lowering`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 61-80 / 第 61-80 行
````python
    indices, tensor_indices = check_and_broadcast_indices(indices, grad.get_device())
    # We can use the first one since they are all required to be the same size
    tensor_size = list(indices[tensor_indices[0]].get_size())
    indexed_size = [x_size[i] for i in range(len(indices))]

    expected_vals_size, inner_fn = index_output_size_and_inner_fn(
        x_size,
        indices,
        tensor_indices,
        tensor_size,
        indices_loaders,
        indexed_size,
        None,
        check=True,
    )

    values = expand(values, expected_vals_size)
    device = grad.get_device()
    assert device is not None
    scatter = Scatter(
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `tensor_size`, `indexed_size`, `check`, `values`, `device`, and `scatter`. This range continues the implementation of function `zeros_and_scatter_lowering`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `tensor_size`、`indexed_size`、`check`、`values`、`device`、`scatter` 等值。这一段延续了函数`zeros_and_scatter_lowering` 的具体实现。

### Lines 81-100 / 第 81-100 行
````python
        device=device,
        dtype=grad.get_dtype(),
        inner_fn=values.make_loader(),
        ranges=expected_vals_size,  # iter_ranges,
        output_indexer=inner_fn,
        scatter_mode="atomic_add",
    )

    buffer = ComputedBuffer(
        name=grad.data.data.name,  # type: ignore[attr-defined]
        layout=MutationLayoutSHOULDREMOVE(grad),
        data=scatter,
    )
    return buffer


def get_fwd_subgraph_outputs(
    subgraph_buffer: SubgraphResults, mask_graph_buffer: SubgraphResults
) -> list[ComputedBuffer | None]:
    subgraph_buffer = (
````
- **EN**: Introduces function `get_fwd_subgraph_outputs`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_fwd_subgraph_outputs`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 101-120 / 第 101-120 行
````python
        subgraph_buffer if isinstance(subgraph_buffer, Sequence) else [subgraph_buffer]
    )
    mask_graph_buffer = (
        mask_graph_buffer
        if isinstance(mask_graph_buffer, Sequence)
        else [mask_graph_buffer]
    )

    return [*subgraph_buffer, *mask_graph_buffer]


def build_subgraph_module_buffer(
    args: list[TensorBox],
    graph_module: torch.fx.GraphModule,
) -> SubgraphResults:
    """This function's goal is to take in the required args and produce the subgraph buffer
    The subgraph buffer is a ComputedBuffer that will be inlined into the triton template

    Args:
        args: The args that are passed into the subgraph. Contains both fixed and lifted inputs.
````
- **EN**: Introduces function `build_subgraph_module_buffer`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`build_subgraph_module_buffer`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 121-140 / 第 121-140 行
````python
        subgraph: The Subgraph ir for which to produce the output node
    """
    # This one we gotta keep lazy
    from ...subgraph_lowering import PointwiseSubgraphLowering

    pw_subgraph = PointwiseSubgraphLowering(
        graph_module,
        root_graph_lowering=V.graph,
        allowed_mutations=OrderedSet([torch.ops.flex_lib.zeros_and_scatter.default]),
        additional_lowerings={
            torch.ops.flex_lib.zeros_and_scatter.default: zeros_and_scatter_lowering
        },
    )
    with V.set_graph_handler(pw_subgraph):  # type: ignore[arg-type]
        pw_subgraph.run(*args)

    def convert_output_node_to_buffer(output_buffer) -> ComputedBuffer | None:
        if output_buffer is None:
            return None
        if isinstance(output_buffer, ComputedBuffer):
````
- **EN**: Imports dependencies such as `...subgraph_lowering` for the logic in this range. Introduces function `convert_output_node_to_buffer`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `...subgraph_lowering` 等依赖，为后续逻辑提供基础能力。这里定义了函数`convert_output_node_to_buffer`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
            # These nodes are coming from the output of zeros_and_scatter
            return output_buffer
        assert isinstance(output_buffer, TensorBox), (
            "The output node for flex attention's subgraph must be a TensorBox, but got: ",
            type(output_buffer),
        )
        assert isinstance(output_buffer.data, StorageBox), (
            "The output node for the flex attention subgraph must be a StorageBox, but got: ",
            type(output_buffer),
        )
        device = output_buffer.data.get_device()
        assert device is not None
        subgraph_buffer = ComputedBuffer(
            name=None,
            layout=FlexibleLayout(
                device=device,
                dtype=output_buffer.data.get_dtype(),
                size=output_buffer.data.get_size(),
            ),
            data=output_buffer.data.data,  # type: ignore[arg-type]
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `device`, `subgraph_buffer`, `name`, `layout`, `dtype`, `size`, and `...+1`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `device`、`subgraph_buffer`、`name`、`layout`、`dtype`、`size`、`另有1项` 等值。

### Lines 161-180 / 第 161-180 行
````python
        )
        return subgraph_buffer

    return tree_map(convert_output_node_to_buffer, pw_subgraph.graph_outputs)


def build_subgraph_buffer(args: list[TensorBox], subgraph: Subgraph) -> SubgraphResults:
    return build_subgraph_module_buffer(args, subgraph.graph_module)


def maybe_realize(args: list[IRNode | None]):
    """Accepts a list of optional IRNodes and returns a list of realized IRNodes"""
    return tree_map(
        lambda x: (
            realize_inputs(x)
            if x is not None and not isinstance(x, sympy.Symbol)
            else x
        ),
        args,
    )
````
- **EN**: Introduces function `build_subgraph_buffer`, function `maybe_realize`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`build_subgraph_buffer`、函数`maybe_realize`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 181-200 / 第 181-200 行
````python


def freeze_irnodes(tree: Any) -> Any:
    """Freeze layouts for every IRNode contained in a pytree."""

    if tree is None:
        return None

    def _freeze(node: IRNode) -> IRNode:
        try:
            node.freeze_layout()
        except NotImplementedError:
            pass
        return node

    return tree_map_only(IRNode, _freeze, tree)


def create_placeholder(
    name: str,
````
- **EN**: Introduces function `freeze_irnodes`, function `_freeze`, function `create_placeholder`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`, and `name`.
- **CN**: 这里定义了函数`freeze_irnodes`、函数`_freeze`、函数`create_placeholder`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `try`、`name` 等值。

### Lines 201-220 / 第 201-220 行
````python
    dtype: torch.dtype,
    device: torch.device,
    size: list[int] | None = None,
) -> TensorBox:
    """Creates a placeholder input buffers for producing subgraph_output."""
    input_buffer = InputBuffer(
        name=name,
        layout=FixedLayout(
            device,
            dtype,
            size if size else [],
            FlexibleLayout.contiguous_strides(size) if size else [],
        ),
    )
    return TensorBox.create(input_buffer)


def construct_strides(
    sizes: Sequence[_IntLike],
    fill_order: Sequence[int],
````
- **EN**: Introduces function `construct_strides`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`construct_strides`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 221-240 / 第 221-240 行
````python
) -> Sequence[_IntLike]:
    """From a list of sizes and a fill order, construct the strides of the permuted tensor."""
    # Initialize strides
    assert len(sizes) == len(fill_order), (
        "Length of sizes must match the length of the fill order"
    )
    strides: list[_IntLike] = [0] * len(sizes)

    # Start with stride 1 for the innermost dimension
    current_stride: _IntLike = 1

    # Iterate through the fill order populating strides
    for dim in fill_order:
        strides[dim] = current_stride
        current_stride *= sizes[dim]

    return strides


def infer_dense_strides(
````
- **EN**: Introduces function `infer_dense_strides`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `strides`, and `current_stride`.
- **CN**: 这里定义了函数`infer_dense_strides`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `strides`、`current_stride` 等值。

### Lines 241-260 / 第 241-260 行
````python
    size: Sequence[_IntLike],
    orig_strides: Sequence[_IntLike],
):
    """This is a mirror of the same function in aten/src/ATen/ExpandUtils.cpp

    Args:
        size: The size of the output tensor
        orig_strides: The strides of the input tensor
    Returns:
        List[int]: Dense non-overlapping strides that preserve the input tensor's layout permutation.
        The returned strides follow the same stride propagation rules as TensorIterator. This matches
        The behavior of empty_like()
    """
    fill_order = get_fill_order(orig_strides, V.graph.sizevars.shape_env)
    strides = construct_strides(size, fill_order)

    # Attention kernels require stride[-1]=1 for efficient memory access.
    # Ensure this by moving last dim to front of fill_order if needed.
    if strides[-1] != 1:
        last_dim = len(size) - 1
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `size`, `orig_strides`, `Args`, `Returns`, `fill_order`, `strides`, and `...+1`. This range continues the implementation of function `infer_dense_strides`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `size`、`orig_strides`、`Args`、`Returns`、`fill_order`、`strides`、`另有1项` 等值。这一段延续了函数`infer_dense_strides` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python
        fill_order = list(fill_order)
        fill_order.remove(last_dim)
        fill_order = [last_dim] + fill_order
        strides = construct_strides(size, fill_order)

    return strides


def create_indices_fake(x) -> torch.Tensor:
    """Create a fake indices that is used for autotuning."""
    size = V.graph.sizevars.optimization_hints(x.get_size())
    indices = torch.arange(0, size[-1], dtype=x.get_dtype(), device=x.get_device())
    indices = indices.expand(size).contiguous()
    return indices


def create_num_blocks_fake_generator(sparse_indices):
    """Create a fake num_blocks that is used for autotuning.

    The idea here is that we need to create a real tensor with real data
````
- **EN**: Introduces function `create_indices_fake`, function `create_num_blocks_fake_generator`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`create_indices_fake`、函数`create_num_blocks_fake_generator`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 281-300 / 第 281-300 行
````python
    that's representative for benchmarking.
    For example, returning all zeros for the `kv_num_blocks` input would mean
    that we are computing 0 blocks for each row, which would provide bogus
    autotuning results.

    In this case, we choose to use min(16, max_block) blocks, because I
    (Horace) think it'll probably result in pretty representative performance.
    If it's too short then prefetching won't help. If it's too long then
    autotuning will take longer for no good reason.
    """

    def create_num_blocks_fake(x) -> torch.Tensor:
        num_blocks_for_autotuning = V.graph.sizevars.optimization_hint(
            sparse_indices.shape[-1]
        )
        size = V.graph.sizevars.optimization_hints(x.get_size())
        return torch.full(
            size,
            num_blocks_for_autotuning,
            dtype=x.get_dtype(),
````
- **EN**: Introduces function `create_num_blocks_fake`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `num_blocks_for_autotuning`, `size`, and `dtype`.
- **CN**: 这里定义了函数`create_num_blocks_fake`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `num_blocks_for_autotuning`、`size`、`dtype` 等值。

### Lines 301-320 / 第 301-320 行
````python
            device=x.get_device(),
        )

    return create_num_blocks_fake


def contiguous_last_dim(x):
    """Ensure that realized IR node has a contiguous stride in the last dimension."""
    strides = x.maybe_get_stride()
    if strides and strides[-1] != 1:
        contiguous_stride_order = list(reversed(range(len(x.get_size()))))
        return ExternKernel.require_stride_order(x, contiguous_stride_order)
    return x


def set_head_dim_values(
    kernel_options: dict[str, Any], qk_head_dim, v_head_dim, graph_sizevars
):
    """
    Mutates kernel options, adding head dimension calculations.
````
- **EN**: Introduces function `contiguous_last_dim`, function `set_head_dim_values`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`contiguous_last_dim`、函数`set_head_dim_values`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 321-340 / 第 321-340 行
````python

    Args:
        kernel_options: Dictionary to populate with options
        qk_head_dim: Query/Key head dimension
        v_head_dim: Value head dimension
        graph_sizevars: Graph size variables object with guard_int method

    """
    # QK dimensions
    qk_head_dim_static = graph_sizevars.guard_int(qk_head_dim)
    kernel_options.setdefault("QK_HEAD_DIM", qk_head_dim_static)
    kernel_options.setdefault(
        "QK_HEAD_DIM_ROUNDED", next_power_of_two(qk_head_dim_static)
    )

    # V dimensions
    v_head_dim_static = graph_sizevars.guard_int(v_head_dim)
    kernel_options.setdefault("V_HEAD_DIM", v_head_dim_static)
    kernel_options.setdefault(
        "V_HEAD_DIM_ROUNDED", next_power_of_two(v_head_dim_static)
````
- **EN**: Initializes or updates values such as `Args`, `kernel_options`, `qk_head_dim`, `v_head_dim`, `graph_sizevars`, `qk_head_dim_static`, and `...+1`. This range continues the implementation of function `set_head_dim_values`.
- **CN**: 初始化或更新了 `Args`、`kernel_options`、`qk_head_dim`、`v_head_dim`、`graph_sizevars`、`qk_head_dim_static`、`另有1项` 等值。这一段延续了函数`set_head_dim_values` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python
    )

    # Safety flag
    kernel_options.setdefault(
        "SAFE_HEAD_DIM",
        is_power_of_2(qk_head_dim_static) and is_power_of_2(v_head_dim_static),
    )


def is_power_of_2(n):
    return n != 0 and ((n & (n - 1)) == 0)


def next_power_of_two(n):
    if n <= 0:
        return 1
    return 2 ** math.ceil(math.log2(n))


_FLEX_TEMPLATE_DIR = Path(__file__).parent / "templates"
````
- **EN**: Introduces function `is_power_of_2`, function `next_power_of_two`. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`is_power_of_2`、函数`next_power_of_two`。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 361-364 / 第 361-364 行
````python
load_flex_template = partial(load_template, template_dir=_FLEX_TEMPLATE_DIR)


# Template strings have been moved to templates/common.py.jinja
````
- **EN**: Initializes or updates values such as `load_flex_template`.
- **CN**: 初始化或更新了 `load_flex_template` 等值。

## Key Concepts / 关键概念
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `zeros_and_scatter_lowering`, `get_fwd_subgraph_outputs`, `build_subgraph_module_buffer`, `build_subgraph_buffer`, `maybe_realize`, `freeze_irnodes`, and `...+9`  
  **CN**: 主要函数：`zeros_and_scatter_lowering`、`get_fwd_subgraph_outputs`、`build_subgraph_module_buffer`、`build_subgraph_buffer`、`maybe_realize`、`freeze_irnodes`、`另有9项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `math`, `collections.abc`, `functools`, `pathlib`, `typing`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.virtualized`, `torch.utils._ordered_set`, `torch.utils._pytree`, `...ir`, `...lowering`, `...select_algorithm`, `...utils`, `torch._inductor.codegen.cuda_combined_scheduling`, `...subgraph_lowering`
