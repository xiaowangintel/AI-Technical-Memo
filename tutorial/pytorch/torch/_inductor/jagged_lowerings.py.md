# jagged_lowerings.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/jagged_lowerings.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It exposes functions such as `dense_idx_to_jagged_idx`, `get_inverse_offsets`, `jagged_idx_to_dense_idx`, and `register_jagged_ops`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。同时提供 `dense_idx_to_jagged_idx`、`get_inverse_offsets`、`jagged_idx_to_dense_idx`、`register_jagged_ops` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs

import sympy

import torch

from .ir import Pointwise, TensorBox
from .virtualized import ops


# pyre-ignore[2,3]
def dense_idx_to_jagged_idx(batch_idx, seq_idx, offsets_loader, jagged_len):
    # jagged_len + 1 is used as the upper bound,
    # because the last sequence length may be zero
    begin_idx = ops.indirect_indexing(
        offsets_loader([batch_idx]),
        jagged_len + 1,
    )
    end_idx = offsets_loader([batch_idx + 1])
    jagged_idx = begin_idx + seq_idx
````
- **EN**: Imports dependencies such as `sympy`, `torch`, `.ir`, and `.virtualized` for the logic in this range. Introduces function `dense_idx_to_jagged_idx`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `begin_idx`, `end_idx`, and `jagged_idx`.
- **CN**: 这里导入了 `sympy`、`torch`、`.ir`、`.virtualized` 等依赖，为后续逻辑提供基础能力。这里定义了函数`dense_idx_to_jagged_idx`。借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `begin_idx`、`end_idx`、`jagged_idx` 等值。

### Lines 21-40 / 第 21-40 行
````python
    return jagged_idx, end_idx


def get_inverse_offsets(
    offsets: TensorBox,
    jagged_len: int | sympy.Expr,
    realize: bool = True,
) -> TensorBox:
    """
    Returns "inverse_offsets" - the inverse of the offsets array.
    offsets maps batch index (dense) to jagged index (i.e. offset into jagged tensor).
    inverse_offsets maps jagged index to batch index.

    e.g. for offsets [0, 3, 4, 9, 10] this will return
    inverse_offsets = [0, 0, 0, 1, 2, 2, 2, 2, 2, 3]

    For the given offsets, the computed inverse_offsets are cached
    on the first call and reused in the further calls.
    """

````
- **EN**: Introduces function `get_inverse_offsets`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_inverse_offsets`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 41-60 / 第 41-60 行
````python
    if hasattr(offsets, "inverse_offsets"):
        # inverse_offsets are already computed
        # for these offsets: can reuse
        return offsets.inverse_offsets

    # ops.bucketize takes offsets.get_name() which doesn't exist on Pointwise
    # kernels, i.e. we need to realize it before using. In other words, we need
    # offsets to be in global memory so that we can binary search over the
    # entire tensor
    offsets.realize()
    device: torch.device = offsets.get_device_or_error()
    dtype: torch.dtype = offsets.get_dtype()

    # pyre-ignore[2,3]
    def inner_fn(index):
        idx = index[0]
        bucket = ops.bucketize(
            values=ops.index_expr(idx, dtype),
            boundaries=(
                offsets.get_name(),
````
- **EN**: Introduces function `inner_fn`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`inner_fn`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 61-80 / 第 61-80 行
````python
                offsets.get_size()[-1],
                offsets.get_size()[0] * offsets.get_stride()[0],
                offsets.get_stride()[-1],
            ),
            boundary_indices=0,
            indexing_dtype=dtype,
            right=True,
        )
        # ops.bucketize above returns 1-based bucket indices,
        # but we need 0-based, hence we subtract 1 from batch
        return bucket - 1

    inverse_offsets = Pointwise.create(
        device=device,
        dtype=dtype,
        inner_fn=inner_fn,
        ranges=[jagged_len],
    )

    if realize:
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 81-100 / 第 81-100 行
````python
        # "freeze" the node so that it doesn't get inlined downstream.
        inverse_offsets.realize()

    # cache inverse_offsets for further reuse
    offsets.inverse_offsets = inverse_offsets  # type: ignore[attr-defined]

    return inverse_offsets


def jagged_idx_to_dense_idx(
    jagged_idx,  # pyre-ignore[2]
    inverse_offsets_loader,  # pyre-ignore[2]
    offsets_loader,  # pyre-ignore[2]
    batch_size: int | sympy.Expr,
    max_seq_len: int | sympy.Expr,
    offsets_dtype: torch.dtype,
) -> tuple[sympy.Expr, sympy.Expr]:
    batch_idx = ops.indirect_indexing(
        inverse_offsets_loader([jagged_idx]),
        batch_size + 1,
````
- **EN**: Introduces function `jagged_idx_to_dense_idx`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`jagged_idx_to_dense_idx`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 101-120 / 第 101-120 行
````python
    )
    batch_start = offsets_loader([batch_idx])
    seq = ops.index_expr(jagged_idx, offsets_dtype) - batch_start
    # check=False because there may be sequences longer than max_seq_len
    seq_idx = ops.indirect_indexing(seq, max_seq_len, check=False)
    return batch_idx, seq_idx


def register_jagged_ops():
    # Avoid circular import by importing here
    from .lowering import fallback_handler, is_integer_type, register_lowering

    # pyre-ignore[56]
    @register_lowering(torch.ops.aten._jagged_to_padded_dense_forward.default)
    def _jagged_to_padded_dense_forward(
        jagged_values: TensorBox,
        jagged_offsets: list[TensorBox],
        max_lengths: list[int],  # list of ints/SymInts
        padding_value: float = 0.0,
    ) -> TensorBox:
````
- **EN**: Imports dependencies such as `.lowering` for the logic in this range. Introduces function `register_jagged_ops`, function `_jagged_to_padded_dense_forward`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `.lowering` 等依赖，为后续逻辑提供基础能力。这里定义了函数`register_jagged_ops`、函数`_jagged_to_padded_dense_forward`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 121-140 / 第 121-140 行
````python
        device = jagged_values.get_device_or_error()
        dtype = jagged_values.get_dtype()

        jagged_values_size = jagged_values.get_size()

        # only handle the common case of a single jagged dimension
        if (
            len(jagged_offsets) != 1
            or device.type != "cuda"
            or device != jagged_offsets[0].get_device()
            or len(jagged_values_size) != 2
            or len(jagged_offsets[0].get_size()) != 1
            or len(max_lengths) != len(jagged_offsets)
            or not is_integer_type(jagged_offsets[0])
        ):
            return fallback_handler(
                torch.ops.aten._jagged_to_padded_dense_forward.default,
                add_to_fallback_set=False,
            )(
                jagged_values,
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
                jagged_offsets,
                max_lengths,
                padding_value,
            )

        offsets: TensorBox = jagged_offsets[0]  # type: ignore[assignment]
        offsets_len = offsets.get_size()[0]
        offsets_dtype = offsets.get_dtype()
        batch_size = offsets_len - 1
        max_seq_len = max_lengths[0]
        embedding_len = jagged_values_size[1]
        jagged_len = jagged_values_size[0]

        output_size = [batch_size, max_seq_len, embedding_len]

        values_loader = jagged_values.make_loader()
        offsets_loader = offsets.make_loader()

        # pyre-ignore[2,3,53]
        def inner_fn(index):
````
- **EN**: Introduces function `inner_fn`. Initializes or updates values such as `offsets`, `offsets_len`, `offsets_dtype`, `batch_size`, `max_seq_len`, `embedding_len`, and `...+4`.
- **CN**: 这里定义了函数`inner_fn`。初始化或更新了 `offsets`、`offsets_len`、`offsets_dtype`、`batch_size`、`max_seq_len`、`embedding_len`、`另有4项` 等值。

### Lines 161-180 / 第 161-180 行
````python
            # dense tensor size: [B, N, D]
            batch_idx, seq_idx, emb_idx = index
            jagged_idx, end_idx = dense_idx_to_jagged_idx(
                batch_idx=batch_idx,
                seq_idx=seq_idx,
                offsets_loader=offsets_loader,
                jagged_len=jagged_len,
            )
            return ops.masked(
                ops.lt(
                    ops.index_expr(jagged_idx, offsets_dtype),
                    end_idx,
                ),
                lambda: values_loader([jagged_idx, emb_idx]),
                padding_value,
            )

        return Pointwise.create(
            device=device,
            dtype=dtype,
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `batch_idx`, `seq_idx`, `offsets_loader`, `jagged_len`, `lambda`, `device`, and `...+1`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `batch_idx`、`seq_idx`、`offsets_loader`、`jagged_len`、`lambda`、`device`、`另有1项` 等值。

### Lines 181-200 / 第 181-200 行
````python
            inner_fn=inner_fn,
            ranges=output_size,
        )

    def _dense_to_jagged_forward_impl(
        fallback_op,  # pyre-ignore[2]
        dense: TensorBox,
        jagged_offsets: list[TensorBox],
        jagged_len: int | None = None,
    ) -> TensorBox:
        device = dense.get_device_or_error()
        dtype = dense.get_dtype()

        dense_size = dense.get_size()

        # only handle the common case of a single jagged dimension
        if (
            len(jagged_offsets) != 1
            or device.type != "cuda"
            or device != jagged_offsets[0].get_device()
````
- **EN**: Introduces function `_dense_to_jagged_forward_impl`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_dense_to_jagged_forward_impl`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。

### Lines 201-220 / 第 201-220 行
````python
            or len(jagged_offsets[0].get_size()) != 1
            or len(dense_size) != 3
            or jagged_len is None
            or not is_integer_type(jagged_offsets[0])
        ):
            return fallback_handler(fallback_op, add_to_fallback_set=False)(
                dense,
                jagged_offsets,
                jagged_len,
            )

        offsets: TensorBox = jagged_offsets[0]  # type: ignore[assignment]
        offsets_dtype = offsets.get_dtype()
        batch_size = dense_size[0]
        max_seq_len = dense_size[1]
        embedding_len = dense_size[-1]

        output_size = [jagged_len, embedding_len]

        dense_loader = dense.make_loader()
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `offsets`, `offsets_dtype`, `batch_size`, `max_seq_len`, `embedding_len`, `output_size`, and `...+1`. This range continues the implementation of function `register_jagged_ops._dense_to_jagged_forward_impl`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `offsets`、`offsets_dtype`、`batch_size`、`max_seq_len`、`embedding_len`、`output_size`、`另有1项` 等值。这一段延续了函数`register_jagged_ops._dense_to_jagged_forward_impl` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
        offsets_loader = offsets.make_loader()

        inverse_offsets = get_inverse_offsets(
            offsets=offsets,
            jagged_len=jagged_len,
        )
        inverse_offsets_loader = inverse_offsets.make_loader()

        # pyre-ignore[2,3,53]
        def inner_fn(index):
            # jagged tensor size: [sum_B(N_B), D]
            jagged_idx, emb_idx = index
            batch_idx, seq_idx = jagged_idx_to_dense_idx(
                jagged_idx=jagged_idx,
                offsets_loader=offsets_loader,
                inverse_offsets_loader=inverse_offsets_loader,
                batch_size=batch_size,
                max_seq_len=max_seq_len,
                offsets_dtype=offsets_dtype,
            )
````
- **EN**: Introduces function `inner_fn`. Initializes or updates values such as `offsets_loader`, `inverse_offsets`, `offsets`, `jagged_len`, `inverse_offsets_loader`, `jagged_idx`, and `...+3`.
- **CN**: 这里定义了函数`inner_fn`。初始化或更新了 `offsets_loader`、`inverse_offsets`、`offsets`、`jagged_len`、`inverse_offsets_loader`、`jagged_idx`、`另有3项` 等值。

### Lines 241-260 / 第 241-260 行
````python
            return ops.masked(
                ops.lt(
                    ops.index_expr(seq_idx, offsets_dtype),
                    ops.index_expr(max_seq_len, offsets_dtype),
                ),
                lambda: dense_loader([batch_idx, seq_idx, emb_idx]),
                0.0,  # jagged sequence longer than max_seq_len
            )

        return Pointwise.create(
            device=device,
            dtype=dtype,
            inner_fn=inner_fn,
            ranges=output_size,
        )

    # pyre-ignore[56]
    @register_lowering(torch.ops.aten._padded_dense_to_jagged_forward)
    def _dense_to_jagged_forward(
        dense: TensorBox,
````
- **EN**: Introduces function `_dense_to_jagged_forward`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`_dense_to_jagged_forward`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 261-269 / 第 261-269 行
````python
        jagged_offsets: list[TensorBox],
        jagged_len: int | None = None,
    ) -> TensorBox:
        return _dense_to_jagged_forward_impl(
            fallback_op=torch.ops.aten._padded_dense_to_jagged_forward.default,
            dense=dense,
            jagged_offsets=jagged_offsets,
            jagged_len=jagged_len,
        )
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `jagged_offsets`, `jagged_len`, `fallback_op`, and `dense`. This range continues the implementation of function `register_jagged_ops._dense_to_jagged_forward`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `jagged_offsets`、`jagged_len`、`fallback_op`、`dense` 等值。这一段延续了函数`register_jagged_ops._dense_to_jagged_forward` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `dense_idx_to_jagged_idx`, `get_inverse_offsets`, `jagged_idx_to_dense_idx`, and `register_jagged_ops`  
  **CN**: 主要函数：`dense_idx_to_jagged_idx`、`get_inverse_offsets`、`jagged_idx_to_dense_idx`、`register_jagged_ops`

## Dependencies / 依赖关系
- **Standard library / 标准库**: None / 无
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `.ir`, `.virtualized`, `.lowering`
