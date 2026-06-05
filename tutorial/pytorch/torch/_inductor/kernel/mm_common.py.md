# mm_common.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel/mm_common.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It exposes functions such as `mm_grid`, `persistent_mm_grid`, `persistent_grouped_mm_grid`, `acc_type`, `mm_args`, `addmm_epilogue`, and `...+5`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。同时提供 `mm_grid`、`persistent_mm_grid`、`persistent_grouped_mm_grid`、`acc_type`、`mm_args`、`addmm_epilogue`、`另有5项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import logging
from collections.abc import Sequence
from functools import partial
from pathlib import Path
from typing import Any

import torch
from torch._inductor.select_algorithm import realize_inputs, SymbolicGridFn
from torch._inductor.utils import get_current_backend, sympy_product
from torch._inductor.virtualized import V
from torch.fx.experimental.symbolic_shapes import has_free_unbacked_symbols

from .. import config
from ..codegen.wrapper import PythonWrapperCodegen
from ..ir import _IntLike, Layout, TensorBox
from ..utils import load_template


log = logging.getLogger(__name__)
````
- **EN**: Imports dependencies such as `logging`, `collections.abc`, `functools`, `pathlib`, `typing`, `torch`, and `...+8` for the logic in this range. Initializes or updates values such as `log`.
- **CN**: 这里导入了 `logging`、`collections.abc`、`functools`、`pathlib`、`typing`、`torch`、`另有8项` 等依赖，为后续逻辑提供基础能力。初始化或更新了 `log` 等值。

### Lines 21-40 / 第 21-40 行
````python


@SymbolicGridFn
def mm_grid(m, n, meta, *, cdiv):
    """
    The CUDA grid size for matmul triton templates.
    """
    return (cdiv(m, meta["BLOCK_M"]) * cdiv(n, meta["BLOCK_N"]), 1, 1)


@SymbolicGridFn
def persistent_mm_grid(M: int, N: int, meta: dict[str, Any], *, cdiv, min):
    """Defines the grid for persistent kernels."""
    return (
        min(meta["NUM_SMS"], cdiv(M, meta["BLOCK_M"]) * cdiv(N, meta["BLOCK_N"])),
        1,
        1,
    )


````
- **EN**: Introduces function `mm_grid`, function `persistent_mm_grid`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`mm_grid`、函数`persistent_mm_grid`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 41-60 / 第 41-60 行
````python
@SymbolicGridFn
def persistent_grouped_mm_grid(*args):
    meta = args[-1]
    return (meta["NUM_SMS"], 1, 1)


def acc_type(dtype):
    if dtype in (torch.float16, torch.bfloat16):
        return "tl.float32"
    return f"tl.{dtype}".replace("torch.", "")


def mm_args(
    mat1,
    mat2,
    *others,
    layout=None,
    out_dtype=None,
    use_4x2_dim=False,
    mat2_transposed=False,
````
- **EN**: Introduces function `persistent_grouped_mm_grid`, function `acc_type`, function `mm_args`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`persistent_grouped_mm_grid`、函数`acc_type`、函数`mm_args`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 61-80 / 第 61-80 行
````python
):
    """
    Common arg processing for mm,bmm,addmm,etc
    """
    mat1, mat2 = realize_inputs(mat1, mat2)
    *b1, m, k1 = mat1.get_size()
    if mat2_transposed:
        *b2, n, k2 = mat2.get_size()
    else:
        *b2, k2, n = mat2.get_size()
    b = [V.graph.sizevars.check_equals_and_simplify(a, b) for a, b in zip(b1, b2)]
    if use_4x2_dim:
        k2 = k2 * 2
    k = V.graph.sizevars.check_equals_and_simplify(k1, k2)
    if layout is None:
        from torch._inductor.ir import FixedLayout

        if out_dtype is None:
            out_dtype = mat1.get_dtype()

````
- **EN**: Imports dependencies such as `torch._inductor.ir` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `b`, `k2`, `k`, and `out_dtype`. This range continues the implementation of function `mm_args`.
- **CN**: 这里导入了 `torch._inductor.ir` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`b`、`k2`、`k`、`out_dtype` 等值。这一段延续了函数`mm_args` 的具体实现。

### Lines 81-100 / 第 81-100 行
````python
        layout = FixedLayout(
            mat1.get_device(),
            out_dtype,
            [*b, m, n],
        )
    else:
        assert out_dtype is None, "out_dtype is ignored if layout is specified."
    from ..lowering import expand

    others = [realize_inputs(expand(x, layout.size)) for x in others]

    return [m, n, k, layout, mat1, mat2, *others]


def addmm_epilogue(dtype, alpha, beta):
    def epilogue(acc, bias):
        if alpha != 1:
            acc = V.ops.mul(acc, V.ops.constant(alpha, dtype))
        if beta != 1:
            bias = V.ops.mul(bias, V.ops.constant(beta, dtype))
````
- **EN**: Imports dependencies such as `..lowering` for the logic in this range. Introduces function `addmm_epilogue`, function `epilogue`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `..lowering` 等依赖，为后续逻辑提供基础能力。这里定义了函数`addmm_epilogue`、函数`epilogue`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 101-120 / 第 101-120 行
````python
        return V.ops.add(acc, bias)

    return epilogue


def scale_mm_epilogue():
    """
    Create an epilogue function that applies scaling to matrix multiplication result
    using the given scale factors.

    Args:
        dtype: The data type of the output
        scale_a: Scale factor for matrix A
        scale_b: Scale factor for matrix B

    Returns:
        Epilogue function that takes the accumulator and applies scaling
    """

    def epilogue(acc, inv_a_scale, inv_b_scale, bias=None):
````
- **EN**: Introduces function `scale_mm_epilogue`, function `epilogue`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `Args`, `dtype`, `scale_a`, `scale_b`, and `Returns`.
- **CN**: 这里定义了函数`scale_mm_epilogue`、函数`epilogue`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `Args`、`dtype`、`scale_a`、`scale_b`、`Returns` 等值。

### Lines 121-140 / 第 121-140 行
````python
        # The epilogue function receives the accumulator (result of mat1 @ mat2)
        # and applies the scaling factors
        # In the original scaled_mm, we use inverse scales, so we multiply by them
        mul_scales = V.ops.mul(inv_a_scale, inv_b_scale)
        mul_acc = V.ops.mul(acc, mul_scales)
        if bias is not None:
            return V.ops.add(mul_acc, bias)
        else:
            return mul_acc

    return epilogue


def use_native_matmul(mat1, mat2):
    if not config.triton.native_matmul:
        return False

    # If tma matmul is on, don't do native matmul
    if (
        config.triton.enable_persistent_tma_matmul
````
- **EN**: Introduces function `use_native_matmul`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`use_native_matmul`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
        and torch.utils._triton.has_triton_tma_device()
    ):
        raise AssertionError("native matmul doesn't support tma codegen yet")

    # Currently only enable native matmul for default indexing
    # TODO : support block ptr
    if config.triton.use_block_ptr:
        raise AssertionError("native matmul doesn't support block_ptr codegen yet")

    # Currently only enable native matmul for triton on GPU.
    device_type = mat1.get_device().type
    if not (
        device_type in ("cuda", "xpu") and get_current_backend(device_type) == "triton"
    ):
        return False

    # Currently, tl.dot only supports following dtypes
    triton_supported_dtype = [
        torch.int8,
        torch.uint8,
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 161-180 / 第 161-180 行
````python
        torch.float16,
        torch.bfloat16,
        torch.float32,
    ]
    if mat1.dtype not in triton_supported_dtype:
        return False
    if mat2.dtype not in triton_supported_dtype:
        return False

    # (..., M, K) @ (..., K, N)
    m, k, n = mat1.get_size()[-2], mat1.get_size()[-1], mat2.get_size()[-1]

    # If the shape has unbacked symbols, don't do native matmul.
    # This is related to the behavior of statically_known_multiple_of on unbacked symints.
    # Since statically_known_multiple_of just returns False for unbacked symbols
    # due to the expensive cost, codegen fails when there is a unbacked symbol.
    # In particular, it fails at _split_iteration_ranges in codegen/simd.py.
    # See this : https://github.com/pytorch/pytorch/pull/131649
    if any(map(has_free_unbacked_symbols, [m, k, n])):
        return False
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 181-200 / 第 181-200 行
````python

    # Consider the shape (m,k,n) > 1
    # TODO : support when size = 1
    if (
        V.graph.sizevars.statically_known_leq(m, 1)
        or V.graph.sizevars.statically_known_leq(k, 1)
        or V.graph.sizevars.statically_known_leq(n, 1)
    ):
        return False

    return True


def _is_static_problem(layout: Layout) -> tuple[bool, bool]:
    """
    Check if input tensors and output layout have static shapes and non-zero sizes.

    Args:
        layout: Output layout object with a 'size' attribute.

````
- **EN**: Introduces function `_is_static_problem`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Args`, and `layout`.
- **CN**: 这里定义了函数`_is_static_problem`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Args`、`layout` 等值。

### Lines 201-220 / 第 201-220 行
````python
    Returns:
        Tuple[bool, bool]: (is_static, is_nonzero)
            is_static: True if all shapes are statically known
            is_nonzero: True if all dimensions are non-zero
    """
    static_shape = True
    static_size = PythonWrapperCodegen.statically_known_list_of_ints_or_none(
        layout.size
    )
    if static_size is None:
        nonzero = True
        for s in layout.size:
            sz = PythonWrapperCodegen.statically_known_int_or_none(s)
            if sz is not None and sz == 0:
                nonzero = False
                break
        return False, nonzero
    numel = 1
    for dim in static_size:
        numel *= dim
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Returns`, `is_static`, `is_nonzero`, `static_shape`, `static_size`, `nonzero`, and `...+2`. This range continues the implementation of function `_is_static_problem`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Returns`、`is_static`、`is_nonzero`、`static_shape`、`static_size`、`nonzero`、`另有2项` 等值。这一段延续了函数`_is_static_problem` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
    nonzero = numel > 0
    return static_shape, nonzero


def check_supported_striding(mat_a: TensorBox, mat_b: TensorBox) -> None:
    def is_row_major(stride: Sequence[_IntLike]) -> bool:
        return stride[-1] == 1

    def is_col_major(stride: Sequence[_IntLike]) -> bool:
        return stride[-2] == 1

    def has_zero_dim(size: Sequence[_IntLike]) -> bool:
        return bool(size[0] == 0 or size[1] == 0)

    # Check mat_a (self) stride requirements
    torch._check(
        is_row_major(mat_a.get_stride()) or has_zero_dim(mat_a.get_size()),
        lambda: f"mat_a must be row_major, got stride {mat_a.get_stride()}",
    )

````
- **EN**: Introduces function `check_supported_striding`, function `is_row_major`, function `is_col_major`, function `has_zero_dim`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `nonzero`, and `lambda`.
- **CN**: 这里定义了函数`check_supported_striding`、函数`is_row_major`、函数`is_col_major`、函数`has_zero_dim`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `nonzero`、`lambda` 等值。

### Lines 241-260 / 第 241-260 行
````python
    # Check mat_b stride requirements
    torch._check(
        is_col_major(mat_b.get_stride()) or has_zero_dim(mat_b.get_size()),
        lambda: f"mat_b must be col_major, got stride {mat_b.get_stride()}",
    )


def is_batch_stride_largest_or_zero(mat1, mat2, layout) -> bool:
    """
    Checking if the batch stride is the largest in the stride.
    """
    sizes = [mat1.get_size(), mat2.get_size(), layout.size]
    strides = [mat1.get_stride(), mat2.get_stride(), layout.stride]
    for size, stride in zip(sizes, strides):
        assert len(size) == len(stride) == 3, "Expect 3D tensors"
        if stride[0] != 0 and stride[0] != sympy_product(size[1:]):
            return False

    return True

````
- **EN**: Introduces function `is_batch_stride_largest_or_zero`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `lambda`, `sizes`, and `strides`.
- **CN**: 这里定义了函数`is_batch_stride_largest_or_zero`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `lambda`、`sizes`、`strides` 等值。

### Lines 261-266 / 第 261-266 行
````python

_KERNEL_TEMPLATE_DIR = Path(__file__).parent / "templates"
load_kernel_template = partial(load_template, template_dir=_KERNEL_TEMPLATE_DIR)

_KERNEL_TEMPLATE_FB_DIR = Path(__file__).parent.parent / "fb" / "tlx_templates"
load_fb_kernel_template = partial(load_template, template_dir=_KERNEL_TEMPLATE_FB_DIR)
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Initializes or updates values such as `_KERNEL_TEMPLATE_DIR`, `load_kernel_template`, `_KERNEL_TEMPLATE_FB_DIR`, and `load_fb_kernel_template`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。初始化或更新了 `_KERNEL_TEMPLATE_DIR`、`load_kernel_template`、`_KERNEL_TEMPLATE_FB_DIR`、`load_fb_kernel_template` 等值。

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
- **EN**: Primary functions: `mm_grid`, `persistent_mm_grid`, `persistent_grouped_mm_grid`, `acc_type`, `mm_args`, `addmm_epilogue`, and `...+5`  
  **CN**: 主要函数：`mm_grid`、`persistent_mm_grid`、`persistent_grouped_mm_grid`、`acc_type`、`mm_args`、`addmm_epilogue`、`另有5项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `collections.abc`, `functools`, `pathlib`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.select_algorithm`, `torch._inductor.utils`, `torch._inductor.virtualized`, `torch.fx.experimental.symbolic_shapes`, `..`, `..codegen.wrapper`, `..ir`, `..utils`, `..lowering`, `torch._inductor.ir`
