# bmm.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel/bmm.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It exposes functions such as `bmm_grid`, `_has_broadcast_batch_dim`, `tuned_bmm`, and `tuned_baddbmm`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。同时提供 `bmm_grid`、`_has_broadcast_batch_dim`、`tuned_bmm`、`tuned_baddbmm` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import logging
from typing import TYPE_CHECKING

import torch
from torch._dynamo.utils import counters
from torch._inductor.codegen.rocm.ck_universal_gemm_template import CKGemmTemplate
from torch._inductor.kernel.mm_common import load_kernel_template

from .. import config as inductor_config, ir, lowering as L
from ..kernel_inputs import MMKernelInputs
from ..lowering import lowerings, make_pointwise, make_reduction, transform_args
from ..runtime.runtime_utils import get_max_y_grid
from ..select_algorithm import (
    autotune_select_algorithm,
    ExternKernelChoice,
    SymbolicGridFn,
    TritonTemplate,
)
from ..utils import (
````
- **EN**: Imports dependencies such as `logging`, `typing`, `torch`, `torch._dynamo.utils`, `torch._inductor.codegen.rocm.ck_universal_gemm_template`, `torch._inductor.kernel.mm_common`, and `...+6` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis.
- **CN**: 这里导入了 `logging`、`typing`、`torch`、`torch._dynamo.utils`、`torch._inductor.codegen.rocm.ck_universal_gemm_template`、`torch._inductor.kernel.mm_common`、`另有6项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。

### Lines 21-40 / 第 21-40 行
````python
    _use_cutlass_for_op,
    use_aten_gemm_kernels,
    use_ck_gemm_template,
    use_cpp_bmm_template,
    use_cutlass_template,
    use_nv_universal_gemm_template,
    use_triton_template,
)
from ..virtualized import ops, V
from .mm_common import (
    _is_static_problem,
    is_batch_stride_largest_or_zero,
    mm_args,
    use_native_matmul,
)


if TYPE_CHECKING:
    from ..ir import ChoiceCaller
    from ..select_algorithm import KernelTemplate
````
- **EN**: Imports dependencies such as `..virtualized`, `.mm_common`, `..ir`, and `..select_algorithm` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `..virtualized`、`.mm_common`、`..ir`、`..select_algorithm` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。包含分支、循环或上下文管理等控制流。

### Lines 41-60 / 第 41-60 行
````python

log = logging.getLogger(__name__)
aten = torch.ops.aten


@SymbolicGridFn
def bmm_grid(b, m, n, meta, *, cdiv, max):
    tiles = cdiv(m, meta["BLOCK_M"]) * cdiv(n, meta["BLOCK_N"])
    # Split batch across grid_y and grid_z to avoid exceeding CUDA grid_y limit.
    # When b <= max_y_grid, grid_z = 1 and behavior is identical to the original.
    max_y_grid = get_max_y_grid()
    grid_z = max(cdiv(b, max_y_grid), 1)
    grid_y = cdiv(b, grid_z)
    return (tiles, grid_y, grid_z)


# We define each template kernel in a separate file which is the name of the input to load_kernel_template
# (e.g. triton_bmm for templates/triton_bmm.py.jinja).
# If you are adding a new template, please follow that pattern and add a new file with your implementation in the templates folder.
bmm_template = TritonTemplate(
````
- **EN**: Introduces function `bmm_grid`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`bmm_grid`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 61-80 / 第 61-80 行
````python
    name="bmm",
    grid=bmm_grid,
    source=load_kernel_template("triton_bmm"),
    cache_codegen_enabled_for_template=True,
)

aten_bmm = ExternKernelChoice(torch.bmm, "at::bmm_out", op_overload=aten.bmm.out)
aten_bmm_dtype = ExternKernelChoice(
    torch.bmm,
    "at::_bmm_out_dtype_xpu" if torch.xpu._is_compiled() else "at::_bmm_out_dtype_cuda",
    name="bmm_dtype",
    op_overload=aten.bmm.dtype_out,
)
aten_baddbmm = ExternKernelChoice(
    torch.baddbmm, "at::baddbmm_out", op_overload=aten.baddbmm.out
)


def _has_broadcast_batch_dim(mat1, mat2):
    """Check if either input has a broadcast batch dimension (stride=0).
````
- **EN**: Introduces function `_has_broadcast_batch_dim`. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Initializes or updates values such as `name`, `grid`, `source`, `cache_codegen_enabled_for_template`, `aten_bmm`, `aten_bmm_dtype`, and `...+2`.
- **CN**: 这里定义了函数`_has_broadcast_batch_dim`。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。初始化或更新了 `name`、`grid`、`source`、`cache_codegen_enabled_for_template`、`aten_bmm`、`aten_bmm_dtype`、`另有2项` 等值。

### Lines 81-100 / 第 81-100 行
````python

    The Triton bmm template can trigger CUDA IMA during autotuning with
    stride-0 inputs; the aten bmm fallback handles broadcast correctly.
    """
    return V.graph.sizevars.statically_known_equals(
        mat1.get_stride()[0], 0
    ) or V.graph.sizevars.statically_known_equals(mat2.get_stride()[0], 0)


@L.register_lowering(aten.bmm)
def tuned_bmm(mat1, mat2, out_dtype=None, *, layout=None):
    """
    Lowering for autotuning aten.bmm with different backends (Aten, Triton, CUTLASS, etc.)
    """
    if all(x.get_device().type == "cpu" for x in [mat1, mat2]):
        # decompose to small ops when memory bound
        if mat1.get_size()[1] == 1 or mat2.get_size()[2] == 1:
            mat1 = L.unsqueeze(mat1, -1)
            mat2 = L.unsqueeze(mat2, 1)
            return L.sum_(L.mul(mat1, mat2), axis=2)
````
- **EN**: Introduces function `tuned_bmm`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`tuned_bmm`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 101-120 / 第 101-120 行
````python

        def is_valid_to_require_contiguous(t):
            if not ir.is_storage_and_layout(t):
                return True
            _, layout = ir.as_storage_and_layout(t, freeze=False)
            return isinstance(layout, ir.FlexibleLayout)

        def is_preferred_layout_as_bmm_input(sizes, strides):
            # contiguous on one of the last two dims
            return (
                strides[-1] == 1 and (sizes[-2] == 1 or strides[-2] >= sizes[-1])
            ) or (strides[-2] == 1 and (sizes[-1] == 1 or strides[-1] >= sizes[-2]))

        # Make the input of bmm contiguous
        # if it is not contiguous on either of the last two dims,
        # because bmm cpu implementation would do contiguous() if not.
        # This is to avoid additional copies in bmm.
        def may_require_contiguous(t, meta_t):
            sizes = meta_t.meta["val"].size()
            strides = meta_t.meta["val"].stride()
````
- **EN**: Introduces function `is_valid_to_require_contiguous`, function `is_preferred_layout_as_bmm_input`, function `may_require_contiguous`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`is_valid_to_require_contiguous`、函数`is_preferred_layout_as_bmm_input`、函数`may_require_contiguous`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 121-140 / 第 121-140 行
````python
            if not is_preferred_layout_as_bmm_input(sizes, strides):
                t = ir.ExternKernel.require_contiguous(t)
            return t

        if is_valid_to_require_contiguous(mat1):
            meta_mat1 = V.graph.current_node.args[0]
            mat1 = may_require_contiguous(mat1, meta_mat1)
        if is_valid_to_require_contiguous(mat2):
            meta_mat2 = V.graph.current_node.args[1]
            mat2 = may_require_contiguous(mat2, meta_mat2)

    if use_native_matmul(mat1, mat2):
        mat1 = lowerings[aten.unsqueeze](mat1, -1)
        mat2 = lowerings[aten.unsqueeze](mat2, 1)
        args, kwargs = transform_args(
            args=[mat1, mat2],
            kwargs={},
            broadcast=True,
            type_promotion_kind=None,
            convert_input_to_bool=False,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `t`, `meta_mat1`, `mat1`, `meta_mat2`, `mat2`, `args`, and `...+4`. This range continues the implementation of function `tuned_bmm`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `t`、`meta_mat1`、`mat1`、`meta_mat2`、`mat2`、`args`、`另有4项` 等值。这一段延续了函数`tuned_bmm` 的具体实现。

### Lines 141-160 / 第 141-160 行
````python
        )  # Handles broadcasting the arguments

        if inductor_config.triton.codegen_upcast_to_fp32 and mat1.dtype in [
            torch.float16,
            torch.bfloat16,
        ]:

            def _to_dtype(x):
                return ops.to_dtype(x, mat1.dtype, use_compute_types=False)

            args = [make_pointwise(_to_dtype)(x) for x in args]

        mul_pointwise = make_pointwise(ops.dot)(*args)
        dot_reduction = make_reduction("dot")(mul_pointwise, 2)

        return dot_reduction

    # TODO(coconutruben): integrate into MMKernelInputs when all callsites use that
    m, n, k, layout, mat1, mat2 = mm_args(
        mat1, mat2, layout=layout, out_dtype=out_dtype
````
- **EN**: Introduces function `_to_dtype`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_to_dtype`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 161-180 / 第 161-180 行
````python
    )
    name = "bmm"

    # Create MMKernelInputs for BMM at the top
    kernel_inputs = MMKernelInputs([mat1, mat2], out_dtype=out_dtype)

    # below is for getting an overview logging info of inductor mms
    batch_size = mat1.get_size()[0]  # Extract batch dimension
    counters["aten_mm_info"][f"aten.bmm_{batch_size}_{m}_{n}_{k}"] += 1
    log.info(
        "Tuned aten.bmm: batch=%s, m=%s, n=%s, k=%s, mat1_dtype=%s, mat2_dtype=%s, output_layout=%s",
        batch_size,
        m,
        n,
        k,
        mat1.get_dtype(),
        mat2.get_dtype(),
        layout,
    )

````
- **EN**: Initializes or updates values such as `name`, `kernel_inputs`, and `batch_size`. This range continues the implementation of function `tuned_bmm`.
- **CN**: 初始化或更新了 `name`、`kernel_inputs`、`batch_size` 等值。这一段延续了函数`tuned_bmm` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
    aten_handler: ExternKernelChoice = aten_bmm
    aten_extra_kwargs = {}
    if out_dtype:
        assert mat1.get_device().type in ("cuda", "xpu"), (
            "out_dtype is only supported for CUDA or XPU"
        )
        aten_handler = aten_bmm_dtype
        aten_extra_kwargs = {"out_dtype": out_dtype}

    choices: list[ChoiceCaller] = []

    # Collect all templates for unified call
    templates_to_use: list[ExternKernelChoice | KernelTemplate] = []
    kwarg_overrides = {}

    if use_aten_gemm_kernels():
        templates_to_use.append(aten_handler)
        kwarg_overrides[aten_handler.uid] = aten_extra_kwargs

    if use_triton_template(layout, check_max_autotune=False):
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `aten_handler`, `aten_extra_kwargs`, `choices`, `templates_to_use`, and `kwarg_overrides`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `aten_handler`、`aten_extra_kwargs`、`choices`、`templates_to_use`、`kwarg_overrides` 等值。

### Lines 201-220 / 第 201-220 行
````python
        if not _has_broadcast_batch_dim(mat1, mat2):
            templates_to_use.append(bmm_template)

    # Single unified call for all templates
    choices.extend(
        V.choices.get_template_configs(
            kernel_inputs,
            templates_to_use,
            name,
            kwarg_overrides=kwarg_overrides,
        )
    )
    _, is_nonzero = _is_static_problem(layout)
    batch_stride_largest_or_zero = is_batch_stride_largest_or_zero(mat1, mat2, layout)
    if (
        batch_stride_largest_or_zero
        and is_nonzero
        and use_cutlass_template(layout, m, n, k)
        and _use_cutlass_for_op(name)
    ):
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kwarg_overrides`, and `batch_stride_largest_or_zero`. This range continues the implementation of function `tuned_bmm`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `kwarg_overrides`、`batch_stride_largest_or_zero` 等值。这一段延续了函数`tuned_bmm` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
        from ..codegen.cutlass.gemm_template import CUTLASS3xGemmTemplate

        CUTLASS3xGemmTemplate.add_cutlass_gemm_choices(
            choices, layout, kernel_inputs.nodes()
        )  # type: ignore[arg-type]

    if use_cpp_bmm_template(layout, mat1, mat2):
        from ..codegen.cpp_bmm_template import CppBmmTemplate

        CppBmmTemplate.add_choices(
            choices,
            layout,
            kernel_inputs.nodes(),
        )

    if use_ck_gemm_template(layout, m, n, k):
        CKGemmTemplate.add_ck_gemm_choices(choices, layout, kernel_inputs.nodes())

    if is_nonzero and use_nv_universal_gemm_template(layout, m, n, k, mat1, mat2):
        from ..codegen.nv_universal_gemm import add_nv_universal_gemm_choices
````
- **EN**: Imports dependencies such as `..codegen.cutlass.gemm_template`, `..codegen.cpp_bmm_template`, and `..codegen.nv_universal_gemm` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `tuned_bmm`.
- **CN**: 这里导入了 `..codegen.cutlass.gemm_template`、`..codegen.cpp_bmm_template`、`..codegen.nv_universal_gemm` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。这一段延续了函数`tuned_bmm` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python

        add_nv_universal_gemm_choices(choices, layout, kernel_inputs)

    node, _ = autotune_select_algorithm(name, choices, kernel_inputs.nodes(), layout)
    return node


@L.register_lowering(aten.baddbmm)
def tuned_baddbmm(inp, mat1, mat2, *, alpha=1, beta=1, layout=None):
    """
    Lowering for autotuning aten.mm with different backends (Aten, Triton, CUTLASS, etc.)
    """
    if use_native_matmul(mat1, mat2):
        if beta == 0:
            arg1 = 0
        else:
            arg1 = lowerings[aten.mul](beta, inp)

        if alpha == 0:
            arg2 = 0
````
- **EN**: Introduces function `tuned_baddbmm`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`tuned_baddbmm`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 261-280 / 第 261-280 行
````python
        else:
            arg2 = lowerings[aten.mul](alpha, lowerings[aten.bmm](mat1, mat2))

        return lowerings[aten.add](arg1, arg2)

    # TODO(coconutruben): integrate into MMKernelInputs when all callsites use that
    m, n, k, layout, mat1, mat2, inp = mm_args(mat1, mat2, inp, layout=layout)

    # Create MMKernelInputs for BadDBMM at the top
    kernel_inputs = MMKernelInputs(
        [inp, mat1, mat2], scalars=dict(alpha=alpha, beta=beta)
    )

    # below is for getting an overview logging info of inductor mms
    batch_size = mat1.get_size()[0]
    counters["aten_mm_info"][f"aten.baddbmm_{batch_size}_{m}_{n}_{k}"] += 1
    log.info(
        "Tuned aten.baddbmm: batch_size=%s, m=%s, n=%s, k=%s, mat1_dtype=%s, mat2_dtype=%s, inp=%s, output_layout=%s",
        batch_size,
        m,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `arg2`, `kernel_inputs`, and `batch_size`. This range continues the implementation of function `tuned_baddbmm`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`arg2`、`kernel_inputs`、`batch_size` 等值。这一段延续了函数`tuned_baddbmm` 的具体实现。

### Lines 281-300 / 第 281-300 行
````python
        n,
        k,
        mat1.get_dtype(),
        mat2.get_dtype(),
        inp.get_dtype(),
        layout,
    )
    name = "baddbmm"
    # options to tune from
    choices: list[ChoiceCaller] = []

    # Collect all templates for unified call
    templates_to_use: list[ExternKernelChoice | KernelTemplate] = []
    if use_aten_gemm_kernels():
        templates_to_use.append(aten_baddbmm)

    if use_triton_template(layout, check_max_autotune=False):
        if not _has_broadcast_batch_dim(mat1, mat2):
            templates_to_use.append(bmm_template)

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `choices`, and `templates_to_use`. This range continues the implementation of function `tuned_baddbmm`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`choices`、`templates_to_use` 等值。这一段延续了函数`tuned_baddbmm` 的具体实现。

### Lines 301-307 / 第 301-307 行
````python
    # Single unified call for all templates
    choices.extend(
        V.choices.get_template_configs(kernel_inputs, templates_to_use, name)
    )

    node, _ = autotune_select_algorithm(name, choices, kernel_inputs.nodes(), layout)
    return node
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `tuned_baddbmm`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`tuned_baddbmm` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
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

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._dynamo.utils`, `torch._inductor.codegen.rocm.ck_universal_gemm_template`, `torch._inductor.kernel.mm_common`, `..`, `..kernel_inputs`, `..lowering`, `..runtime.runtime_utils`, `..select_algorithm`, `..utils`, `..virtualized`, `.mm_common`, `..ir`, `..codegen.cutlass.gemm_template`, `..codegen.cpp_bmm_template`, `..codegen.nv_universal_gemm`
