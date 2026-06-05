# mm.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel/mm.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `DecomposeKSugraphTemplate`, and `ContiguousTemplate`. It exposes functions such as `lazy_register_extern_choice`, `_is_int8_mat`, `bias_addmm`, `check_supported_striding`, `decomposeK`, `contiguous_mm`, and `...+18`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `DecomposeKSugraphTemplate`、`ContiguousTemplate` 等类。同时提供 `lazy_register_extern_choice`、`_is_int8_mat`、`bias_addmm`、`check_supported_striding`、`decomposeK`、`contiguous_mm`、`另有18项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
import functools
import logging
from typing import Any

import torch
from torch._dynamo.utils import counters
from torch._inductor.autoheuristic.autoheuristic import AutoHeuristicSelectAlgorithm
from torch._inductor.autoheuristic.autoheuristic_utils import (
    AHContext,
    context_add_strides,
    context_add_using_tf32,
    mm_operations,
)
from torch._inductor.codegen.cpp_gemm_template import CppGemmTemplate
from torch._inductor.remote_gemm_autotune_cache import gen_best_config
from torch._inductor.virtualized import ops, V
from torch.fx.experimental.proxy_tensor import make_fx
from torch.nn.functional import ScalingType  # type: ignore[attr-defined]
from torch.torch_version import TorchVersion

from .. import config as inductor_config, distributed_autotune
from ..codegen.cutlass.gemm_template import CUTLASS2xGemmTemplate, CUTLASS3xGemmTemplate
from ..codegen.rocm.ck_tile_universal_gemm_template import CKTileGemmTemplate
from ..codegen.rocm.ck_universal_gemm_template import CKGemmTemplate
from ..codegen.subgraph import SubgraphChoiceCaller, SubgraphTemplate
from ..ir import Buffer, ChoiceCaller, is_triton, Layout
from ..kernel_inputs import MMKernelInputs
````
- **EN**: Imports dependencies such as `functools`, `logging`, `typing`, `torch`, `torch._dynamo.utils`, `torch._inductor.autoheuristic.autoheuristic`, and `...+14` for the logic in this range. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `functools`、`logging`、`typing`、`torch`、`torch._dynamo.utils`、`torch._inductor.autoheuristic.autoheuristic`、`另有14项` 等依赖，为后续逻辑提供基础能力。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 29-56 / 第 29-56 行
````python
from ..lowering import (
    lowerings,
    make_pointwise,
    make_reduction,
    register_lowering,
    transform_args,
)
from ..select_algorithm import (
    autotune_select_algorithm,
    ExternKernelChoice,
    KernelTemplate,
    realize_inputs,
    TritonTemplate,
)
from ..utils import (
    _use_cutlass_for_op,
    ceildiv,
    use_aten_gemm_kernels,
    use_ck_gemm_template,
    use_ck_tile_gemm_template,
    use_cpp_gemm_template,
    use_cutlass_template,
    use_decompose_k_choice,
    use_nv_universal_gemm_template,
    use_triton_blackwell_tma_template,
    use_triton_scaling_template,
    use_triton_template,
    use_triton_tma_template,
````
- **EN**: Imports dependencies such as `..lowering`, `..select_algorithm`, and `..utils` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `..lowering`、`..select_algorithm`、`..utils` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 57-84 / 第 57-84 行
````python
)
from .mm_common import (
    _is_static_problem,
    load_kernel_template,
    mm_args,
    mm_grid,
    persistent_mm_grid,
    use_native_matmul,
)


try:
    import triton

    triton_version = TorchVersion(triton.__version__)
    has_triton = True
except ImportError:
    triton_version = TorchVersion("0.0.0")
    has_triton = False

log = logging.getLogger(__name__)
aten = torch.ops.aten
prims = torch.ops.prims

# We define each template kernel in a separate file which is the name of the input to load_kernel_template
# (e.g. triton_mm for templates/triton_mm.py.jinja).
# If you are adding a new template, please follow that pattern and add a new file with your implementation in the templates folder.
mm_template = TritonTemplate(
````
- **EN**: Imports dependencies such as `.mm_common`, and `triton` for the logic in this range. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`, `triton_version`, `has_triton`, `log`, `aten`, `prims`, and `...+1`.
- **CN**: 这里导入了 `.mm_common`、`triton` 等依赖，为后续逻辑提供基础能力。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `try`、`triton_version`、`has_triton`、`log`、`aten`、`prims`、`另有1项` 等值。

### Lines 85-112 / 第 85-112 行
````python
    name="mm",
    grid=mm_grid,
    source=load_kernel_template("triton_mm")
    if (torch.version.hip is None) or triton_version >= "3.3.0"
    # FIXME: To get around rocm failures like https://github.com/pytorch/pytorch/actions/runs/13123783322/job/36617154943
    # The only difference between the two templates is M >= BLOCK_M and N >= BLOCK_N checking.
    # See more details in https://github.com/pytorch/pytorch/pull/146293
    else load_kernel_template("triton_mm_rocm"),
    cache_codegen_enabled_for_template=True,
    prologue_loads_all_inputs=True,
)

persistent_tma_mm_template = TritonTemplate(
    name="mm_persistent_tma",
    grid=persistent_mm_grid,
    source=load_kernel_template("triton_persistent_tma_mm"),
)

# Non-TMA Triton template for persistent MM
# used on AMD
persistent_mm_template = TritonTemplate(
    name="mm_persistent",
    grid=persistent_mm_grid,
    source=load_kernel_template("triton_persistent_mm"),
)


scaled_mm_device_tma_epilogue_scaling_template = TritonTemplate(
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。

### Lines 113-140 / 第 113-140 行
````python
    name="scaled_mm_device_tma_epilogue_scaling",
    grid=persistent_mm_grid,
    source=load_kernel_template("triton_epilogue_scaled_mm"),
)


scaled_mm_device_tma_main_loop_scaling_template = TritonTemplate(
    name="scaled_mm_device_tma_main_loop_scaling",
    grid=persistent_mm_grid,
    source=load_kernel_template("triton_main_loop_scaled_mm"),
)

blackwell_ws_persistent_device_tma_mm_template = TritonTemplate(
    name="blackwell_ws_persistent_device_tma",
    grid=persistent_mm_grid,
    source=load_kernel_template("triton_blackwell_ws_persistent_device_tma_mm"),
)


# prevent duplication registration of extern functions
@functools.cache
def lazy_register_extern_choice(fn):
    return ExternKernelChoice(fn)


aten_mm = ExternKernelChoice(torch.mm, "at::mm_out", op_overload=aten.mm.out)
aten_mm_dtype = ExternKernelChoice(
    torch.mm,
````
- **EN**: Introduces function `lazy_register_extern_choice`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`lazy_register_extern_choice`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 141-168 / 第 141-168 行
````python
    "at::mm_dtype_out",
    name="mm_dtype",
    op_overload=aten.mm.dtype_out,
)

aten_addmm = ExternKernelChoice(
    torch.addmm, "at::addmm_out", op_overload=aten.addmm.out
)

aten__int_mm = ExternKernelChoice(
    torch._int_mm, "at::_int_mm_out", op_overload=aten._int_mm.out
)

aten__sparse_semi_structured_mm = ExternKernelChoice(
    torch._sparse_semi_structured_mm,
    "at::_sparse_semi_structured_mm",
    has_out_variant=False,
    op_overload=aten._sparse_semi_structured_mm.default,
)

aten__fp8_mm = ExternKernelChoice(
    torch._scaled_mm, "at::_scaled_mm_out", op_overload=aten._scaled_mm.out
)


def _is_int8_mat(mat):
    return mat.get_dtype() in (torch.int8, torch.uint8)

````
- **EN**: Introduces function `_is_int8_mat`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `name`, `op_overload`, `aten_addmm`, `aten__int_mm`, `aten__sparse_semi_structured_mm`, `has_out_variant`, and `...+1`.
- **CN**: 这里定义了函数`_is_int8_mat`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `name`、`op_overload`、`aten_addmm`、`aten__int_mm`、`aten__sparse_semi_structured_mm`、`has_out_variant`、`另有1项` 等值。

### Lines 169-196 / 第 169-196 行
````python

def bias_addmm(inp, mat1, mat2, *, out=None, alpha=1, beta=1):
    """
    Giving torch.addmm a 1D tensor calls a different (faster) cublasLt
    kernel under the hood.  There are a few shapes where this is slower,
    but they are rare.
    """
    if (inp.stride(0) == 0 and inp.size(0) != 0) or inp.size(0) == 1:
        return torch.addmm(inp[0], mat1, mat2, out=out, alpha=alpha, beta=beta)
    return torch.addmm(inp, mat1, mat2, out=out, alpha=alpha, beta=beta)


def check_supported_striding(mat_a, mat_b) -> None:
    def is_row_major(stride) -> bool:
        return V.graph.sizevars.statically_known_equals(stride[1], 1)

    def is_col_major(stride) -> bool:
        return V.graph.sizevars.statically_known_equals(stride[0], 1)

    def has_zero_dim(size) -> bool:
        return bool(
            V.graph.sizevars.statically_known_equals(size[0], 0)
            or V.graph.sizevars.statically_known_equals(size[1], 0)
        )

    # Check mat_a (self) stride requirements
    torch._check(
        is_row_major(mat_a.get_stride()) or has_zero_dim(mat_a.get_size()),
````
- **EN**: Introduces function `bias_addmm`, function `check_supported_striding`, function `is_row_major`, function `is_col_major`, function `has_zero_dim`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`bias_addmm`、函数`check_supported_striding`、函数`is_row_major`、函数`is_col_major`、函数`has_zero_dim`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 197-224 / 第 197-224 行
````python
        lambda: f"mat_a must be row_major, got stride {mat_a.get_stride()}",
    )

    # Check mat_b stride requirements
    torch._check(
        is_col_major(mat_b.get_stride()) or has_zero_dim(mat_b.get_size()),
        lambda: f"mat_b must be col_major, got stride {mat_b.get_stride()}",
    )


aten_bias_addmm = ExternKernelChoice(bias_addmm, None)


def decomposeK(a, b, k_splits):
    m = a.shape[0]
    n = b.shape[1]
    k = a.shape[1]

    k_parts = k // k_splits
    B = k_splits
    a_reshaped = torch.permute(a.reshape(m, B, k_parts), (1, 0, 2))
    b_reshaped = b.reshape(B, k_parts, n)
    result = torch.bmm(a_reshaped, b_reshaped, out_dtype=torch.float32)
    reduced_buf = torch.sum(result, 0)
    return reduced_buf.to(a.dtype)


class DecomposeKSugraphTemplate(SubgraphTemplate):
````
- **EN**: Introduces function `decomposeK`, class `DecomposeKSugraphTemplate`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `lambda`, `aten_bias_addmm`, `m`, `n`, `k`, `k_parts`, and `...+5`.
- **CN**: 这里定义了函数`decomposeK`、类`DecomposeKSugraphTemplate`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `lambda`、`aten_bias_addmm`、`m`、`n`、`k`、`k_parts`、`另有5项` 等值。

### Lines 225-252 / 第 225-252 行
````python
    def __init__(self):
        super().__init__(
            name="decompose_k",
        )

    def generate(  # type: ignore[override]
        self,
        input_nodes: list[Buffer],
        layout: Layout,
        k_split: int,
    ) -> SubgraphChoiceCaller:
        from torch._dispatch.python import enable_python_dispatcher

        from ..decomposition import select_decomp_table

        name = f"decompose_k_mm_{k_split}_split"
        description = f"{k_split=}"

        with enable_python_dispatcher():
            decompositions = select_decomp_table()
            fn = make_fx(
                functools.partial(decomposeK, k_splits=k_split),
                decompositions,
            )

            return super().generate(
                name=name,
                input_nodes=input_nodes,
````
- **EN**: Imports dependencies such as `torch._dispatch.python`, and `..decomposition` for the logic in this range. Introduces function `__init__`, function `generate`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._dispatch.python`、`..decomposition` 等依赖，为后续逻辑提供基础能力。这里定义了函数`__init__`、函数`generate`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 253-280 / 第 253-280 行
````python
                layout=layout,
                make_fx_graph=fn,
                description=description,
            )


decompose_k_subgraph_template = DecomposeKSugraphTemplate()


class ContiguousTemplate(SubgraphTemplate):
    def __init__(self, name: str, description: str, fn: Any):
        self.name = name
        self.description = description
        self.fn = fn
        super().__init__(
            name=name,
        )

    def generate(  # type: ignore[override]
        self,
        input_nodes: list[Buffer],
        layout: Layout,
    ) -> SubgraphChoiceCaller:
        from torch._dispatch.python import enable_python_dispatcher

        from ..decomposition import select_decomp_table

        with enable_python_dispatcher():
````
- **EN**: Imports dependencies such as `torch._dispatch.python`, and `..decomposition` for the logic in this range. Introduces class `ContiguousTemplate`, function `__init__`, function `generate`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `layout`, `make_fx_graph`, `description`, `decompose_k_subgraph_template`, `name`, and `input_nodes`.
- **CN**: 这里导入了 `torch._dispatch.python`、`..decomposition` 等依赖，为后续逻辑提供基础能力。这里定义了类`ContiguousTemplate`、函数`__init__`、函数`generate`。包含分支、循环或上下文管理等控制流。初始化或更新了 `layout`、`make_fx_graph`、`description`、`decompose_k_subgraph_template`、`name`、`input_nodes` 等值。

### Lines 281-308 / 第 281-308 行
````python
            decompositions = select_decomp_table()
            fn = make_fx(
                self.fn,
                decompositions,
            )

            return super().generate(
                name=self.name,
                input_nodes=input_nodes,
                layout=layout,
                make_fx_graph=fn,
                description=self.description,
            )


def contiguous_mm(a, b):
    return torch.mm(a, b.contiguous())


def contiguous_addmm(inp, a, b):
    return torch.addmm(inp, a, b.contiguous())


mm_contiguous_subgraph_template = ContiguousTemplate(
    "contiguous_mm", "contiguous mm", contiguous_mm
)
addmm_contiguous_subgraph_template = ContiguousTemplate(
    "contiguous_addmm", "contiguous addmm", contiguous_addmm
````
- **EN**: Introduces function `contiguous_mm`, function `contiguous_addmm`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `decompositions`, `fn`, `name`, `input_nodes`, `layout`, `make_fx_graph`, and `...+3`.
- **CN**: 这里定义了函数`contiguous_mm`、函数`contiguous_addmm`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `decompositions`、`fn`、`name`、`input_nodes`、`layout`、`make_fx_graph`、`另有3项` 等值。

### Lines 309-336 / 第 309-336 行
````python
)


@register_lowering(aten.mm, type_promotion_kind=None)
def tuned_mm(mat1, mat2, out_dtype=None, *, layout=None):
    """
    Lowering for autotuning aten.mm with different backends (Aten, Triton, CUTLASS, etc.)
    """
    if out_dtype is not None:
        input_dtype = mat1.get_dtype()
        torch._check(
            mat2.get_dtype() == input_dtype,
            lambda: "input dtypes must be the same",
        )
        torch._check(
            mat1.get_device().type in ("cuda", "xpu"),
            lambda: "out_dtype is only supported for CUDA or XPU",
        )
        torch._check(
            out_dtype == input_dtype
            or (
                out_dtype == torch.float32
                and input_dtype in (torch.float16, torch.bfloat16)
            ),
            lambda: "out_dtype must be the same as input dtype or fp32 for fp16/bf16 inputs",
        )

    # Lower matmul-related operations (e.g., torch.matmul / torch.bmm / torch.addmm)
````
- **EN**: Introduces function `tuned_mm`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`tuned_mm`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 337-364 / 第 337-364 行
````python
    # into native matmul IR using `ops.dot`. When we see a matmul pattern
    # (C[y, x] = A[y, r] * B[r, x]), the core idea is to emulate a broadcasted
    # multiply followed by a sum.
    #
    # For example, given `C = torch.matmul(A, B)`, this can be rewritten as:
    #
    #     Prod = A.unsqueeze(-1) * B.unsqueeze(0)
    #     C = Prod.sum(dim=1)
    #
    # Instead of explicitly using `ops.mul` and `ops.reduction("sum")`, we lower
    # these into `ops.dot` (pointwise) and `ops.reduction("dot")`. These IR nodes
    # are semantically equivalent to the `ops.mul` + `ops.reduction("sum")`
    # combination, but are lowered to `tl.dot` during the code generation phase.
    if use_native_matmul(mat1, mat2):
        mat1 = lowerings[aten.unsqueeze](mat1, -1)
        mat2 = lowerings[aten.unsqueeze](mat2, 0)
        args, kwargs = transform_args(
            args=[mat1, mat2],
            kwargs={},
            broadcast=True,
            type_promotion_kind=None,
            convert_input_to_bool=False,
        )  # Handles broadcasting the arguments

        if inductor_config.triton.codegen_upcast_to_fp32 and mat1.dtype in [
            torch.float16,
            torch.bfloat16,
        ]:
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mat1`, `mat2`, `args`, `kwargs`, `broadcast`, `type_promotion_kind`, and `...+1`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `mat1`、`mat2`、`args`、`kwargs`、`broadcast`、`type_promotion_kind`、`另有1项` 等值。

### Lines 365-392 / 第 365-392 行
````python

            def _to_dtype(x):
                return ops.to_dtype(x, mat1.dtype, use_compute_types=False)

            args = [make_pointwise(_to_dtype)(x) for x in args]
        mul_pointwise = make_pointwise(ops.dot)(*args)
        dot_reduction = make_reduction("dot")(mul_pointwise, 1)

        return dot_reduction

    # TODO(coconutruben): integrate into MMKernelInputs when all callsites use that
    m, n, k, layout, mat1, mat2 = mm_args(
        mat1, mat2, layout=layout, out_dtype=out_dtype
    )
    static_shape, is_nonzero = _is_static_problem(layout)
    name = "mm"

    # Create MMKernelInputs for standard MM at the top
    kernel_inputs = MMKernelInputs([mat1, mat2], out_dtype=out_dtype)

    # below is for getting an overview logging info of inductor mms
    counters["aten_mm_info"][f"aten.mm_{m}_{n}_{k}"] += 1
    log.info(
        "Tuned aten.mm: m=%s, n=%s, k=%s, mat1_dtype=%s, mat2_dtype=%s, output_layout=%s",
        m,
        n,
        k,
        mat1.get_dtype(),
````
- **EN**: Introduces function `_to_dtype`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `args`, `mul_pointwise`, `dot_reduction`, `name`, and `kernel_inputs`.
- **CN**: 这里定义了函数`_to_dtype`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `args`、`mul_pointwise`、`dot_reduction`、`name`、`kernel_inputs` 等值。

### Lines 393-420 / 第 393-420 行
````python
        mat2.get_dtype(),
        layout,
    )

    choices: list[ChoiceCaller] = []
    static_shape, is_nonzero = _is_static_problem(layout)

    aten_handler: ExternKernelChoice = aten_mm
    aten_extra_kwargs: dict[str, Any] = {}
    if out_dtype is not None:
        aten_handler = aten_mm_dtype
        aten_extra_kwargs = {"out_dtype": out_dtype}

    templates_to_use: list[ExternKernelChoice | KernelTemplate] = []
    kwarg_overrides: dict[str, dict[str, Any]] = {}
    if use_aten_gemm_kernels():
        templates_to_use.append(aten_handler)
        if aten_extra_kwargs:
            kwarg_overrides[aten_handler.uid] = aten_extra_kwargs

    if (
        out_dtype is None
        and is_nonzero
        and use_triton_template(layout, check_max_autotune=True)
    ):
        if use_decompose_k_choice(m, n, k):
            templates_to_use.append(decompose_k_subgraph_template)
        # Triton Templates typically perform very poorly for large K.
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `choices`, `aten_handler`, `aten_extra_kwargs`, `templates_to_use`, and `kwarg_overrides`. This range continues the implementation of function `tuned_mm`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `choices`、`aten_handler`、`aten_extra_kwargs`、`templates_to_use`、`kwarg_overrides` 等值。这一段延续了函数`tuned_mm` 的具体实现。

### Lines 421-448 / 第 421-448 行
````python
        # Its highly unlikely that if we want to use decompose_k, then
        # Triton will ever win.
        #
        # To be conservative we increase this threshold for N/M by 2.
        is_exhaustive = inductor_config.max_autotune_gemm_search_space == "exhaustive"
        if is_exhaustive or not use_decompose_k_choice(m, n, k, threshold_multiple=2):
            templates_to_use.append(mm_template)

            if use_triton_blackwell_tma_template(
                mat1, mat2, output_layout=layout, add_guards=True
            ):
                templates_to_use.append(blackwell_ws_persistent_device_tma_mm_template)
            elif use_triton_tma_template(
                mat1, mat2, output_layout=layout, add_guards=True
            ):
                if torch.version.hip is None:
                    templates_to_use.append(persistent_tma_mm_template)
                else:
                    templates_to_use.append(persistent_mm_template)

        templates_to_use.append(mm_contiguous_subgraph_template)

    choices.extend(
        V.choices.get_template_configs(
            kernel_inputs,
            templates_to_use,
            "mm",
            kwarg_overrides=kwarg_overrides,
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `is_exhaustive`, `else`, and `kwarg_overrides`. This range continues the implementation of function `tuned_mm`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `is_exhaustive`、`else`、`kwarg_overrides` 等值。这一段延续了函数`tuned_mm` 的具体实现。

### Lines 449-476 / 第 449-476 行
````python
        )
    )

    if (
        out_dtype is None
        and is_nonzero
        and use_cutlass_template(layout, m, n, k)
        and _use_cutlass_for_op("mm")
    ):
        CUTLASS3xGemmTemplate.add_cutlass_gemm_choices(
            choices, layout, kernel_inputs.nodes()
        )

    if out_dtype is None and is_nonzero and use_ck_gemm_template(layout, m, n, k):
        CKGemmTemplate.add_ck_gemm_choices(choices, layout, kernel_inputs.nodes())
    if out_dtype is None and is_nonzero and use_ck_tile_gemm_template(layout, m, n, k):
        CKTileGemmTemplate.add_choices(choices, layout, kernel_inputs.nodes())

    if (
        out_dtype is None
        and is_nonzero
        and use_nv_universal_gemm_template(layout, m, n, k, mat1, mat2)
    ):
        from ..codegen.nv_universal_gemm import add_nv_universal_gemm_choices

        add_nv_universal_gemm_choices(choices, layout, kernel_inputs)

    if out_dtype is None and use_cpp_gemm_template(layout, mat1, mat2):
````
- **EN**: Imports dependencies such as `..codegen.nv_universal_gemm` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `tuned_mm`.
- **CN**: 这里导入了 `..codegen.nv_universal_gemm` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。这一段延续了函数`tuned_mm` 的具体实现。

### Lines 477-504 / 第 477-504 行
````python
        CppGemmTemplate.add_choices(
            choices,
            layout,
            kernel_inputs.nodes(),
        )

    input_nodes = [mat1, mat2]
    if (
        out_dtype is None
        and is_nonzero
        and use_triton_template(layout)
        and torch._inductor.config.run_autoheuristic(name)
        and is_triton(mat1)
    ):
        always_included = []
        if use_aten_gemm_kernels():
            always_included.append("extern_mm")
        num_choices_before_extra_configs = len(choices)
        choices.extend(
            V.choices.get_template_configs(
                # TODO(coconutruben): remove once we deprecate ah
                # mm-extra is a hack to keep the ah functionality alive
                # while we transition to the unified kwargs retrieval
                kernel_inputs,
                [mm_template],
                "mm-ah",
            )
        )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_nodes`, `always_included`, and `num_choices_before_extra_configs`. This range continues the implementation of function `tuned_mm`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `input_nodes`、`always_included`、`num_choices_before_extra_configs` 等值。这一段延续了函数`tuned_mm` 的具体实现。

### Lines 505-532 / 第 505-532 行
````python

        # using AutoHeuristic for ranking
        ah_choices = mm_autoheuristic(
            mat1,
            mat2,
            m,
            n,
            k,
            choices,
            name,
            input_nodes,
            mm_operations(),
            None,
            top_k=10,
            always_included=always_included,
        )
        if not torch._inductor.config.collect_autoheuristic(name):
            # if we are collecting data, we do not want to modify choices
            if ah_choices is not None and len(ah_choices) > 0:
                # the order in which autoheuristic returns choices is not the same as
                # as the order of choices, which affects things like epilogue fusion.
                # once epilogue fusion benchmarks choices in sorted order, I think we can
                # just use the order returned by autoheuristic
                choices = [choice for choice in choices if choice in ah_choices]
            else:
                choices = choices[:num_choices_before_extra_configs]

    if out_dtype is None:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ah_choices`, `top_k`, `always_included`, `choices`, and `else`. This range continues the implementation of function `tuned_mm`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `ah_choices`、`top_k`、`always_included`、`choices`、`else` 等值。这一段延续了函数`tuned_mm` 的具体实现。

### Lines 533-560 / 第 533-560 行
````python
        for k in inductor_config.external_matmul:
            choices.append(
                lazy_register_extern_choice(k).bind(kernel_inputs.nodes(), layout)
            )

    best_config_future = None
    if out_dtype is None and torch._inductor.config.remote_gemm_autotune_cache:
        # Purposely not awaiting the future here - this kicks off the best config lookup at lowering time
        # The future will be awaited at scheduling time in select_algorithm.py
        best_config_future = gen_best_config(mat1, mat2)

    if box := distributed_autotune.maybe_autotune_remote(
        name, choices, kernel_inputs.nodes(), layout
    ):
        return box

    node, _ = autotune_select_algorithm(
        name,
        choices,
        kernel_inputs.nodes(),
        layout,
        best_config_future=best_config_future,
    )
    return node


@register_lowering(aten._int_mm, type_promotion_kind=None)
def tuned_int_mm(mat1, mat2, *, layout=None):
````
- **EN**: Introduces function `tuned_int_mm`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`tuned_int_mm`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 561-588 / 第 561-588 行
````python
    # TODO(coconutruben): integrate into MMKernelInputs when all callsites use that
    m, n, k, layout, mat1, mat2 = mm_args(
        mat1, mat2, layout=layout, out_dtype=torch.int32
    )
    name = "int_mm"
    # below is for getting an overview logging info of inductor mms
    counters["aten_mm_info"][f"aten._int_mm_{m}_{n}_{k}"] += 1
    log.info(
        "Tuned aten._int_mm: m=%s, n=%s, k=%s, mat1_dtype=%s, mat2_dtype=%s, output_layout=%s",
        m,
        n,
        k,
        mat1.get_dtype(),
        mat2.get_dtype(),
        layout,
    )

    static_shape, is_nonzero = _is_static_problem(layout)
    use_cutlass = static_shape and is_nonzero and use_cutlass_template(layout, m, n, k)
    choices: list[ChoiceCaller] = []

    # Create MMKernelInputs for Int MM
    kernel_inputs = MMKernelInputs([mat1, mat2], out_dtype=torch.int32)

    # Collect all templates for unified call
    templates_to_use: list[ExternKernelChoice | KernelTemplate] = []
    if use_aten_gemm_kernels():
        templates_to_use.append(aten__int_mm)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `use_cutlass`, `choices`, `kernel_inputs`, and `templates_to_use`. This range continues the implementation of function `tuned_int_mm`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`use_cutlass`、`choices`、`kernel_inputs`、`templates_to_use` 等值。这一段延续了函数`tuned_int_mm` 的具体实现。

### Lines 589-616 / 第 589-616 行
````python

    if is_nonzero and use_triton_template(
        layout, enable_int32=True, check_max_autotune=False
    ):
        templates_to_use.append(mm_template)

    # Single unified call for all templates
    choices.extend(
        V.choices.get_template_configs(kernel_inputs, templates_to_use, name)
    )

    if use_cutlass and _use_cutlass_for_op(name):
        CUTLASS3xGemmTemplate.add_cutlass_gemm_choices(
            choices, layout, kernel_inputs.nodes(), fuseable=True, non_fuseable=True
        )

    node, _ = autotune_select_algorithm(name, choices, kernel_inputs.nodes(), layout)
    return node


@register_lowering(aten.addmm, type_promotion_kind=None)
def tuned_addmm(inp, mat1, mat2, *, alpha=1, beta=1, layout=None):
    """
    Lowering for autotuning aten.addmm with different backends (Aten, Triton, CUTLASS, etc.)
    """
    if use_native_matmul(mat1, mat2):
        if beta == 0:
            arg1 = 0
````
- **EN**: Introduces function `tuned_addmm`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`tuned_addmm`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 617-644 / 第 617-644 行
````python
        else:
            arg1 = lowerings[aten.mul](beta, inp)

        if alpha == 0:
            arg2 = 0
        else:
            arg2 = lowerings[aten.mul](alpha, lowerings[aten.mm](mat1, mat2))

        return lowerings[aten.add](arg1, arg2)

    # TODO(coconutruben): integrate into MMKernelInputs when all callsites use that
    m, n, k, layout, mat1, mat2, inp_expanded = mm_args(mat1, mat2, inp, layout=layout)
    static_shape, is_nonzero = _is_static_problem(layout)
    name = "addmm"

    # Create MMKernelInputs for AddMM at the top
    kernel_inputs = MMKernelInputs(
        [inp_expanded, mat1, mat2], scalars=dict(alpha=alpha, beta=beta)
    )
    kernel_inputs_aten = MMKernelInputs(
        [inp, mat1, mat2], scalars=dict(alpha=alpha, beta=beta)
    )

    choices: list[ChoiceCaller] = []

    # below is for getting an overview logging info of inductor mms
    counters["aten_mm_info"][f"aten.addmm_{m}_{n}_{k}"] += 1
    log.info(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `arg1`, `arg2`, `name`, `kernel_inputs`, `kernel_inputs_aten`, and `...+1`. This range continues the implementation of function `tuned_addmm`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`arg1`、`arg2`、`name`、`kernel_inputs`、`kernel_inputs_aten`、`另有1项` 等值。这一段延续了函数`tuned_addmm` 的具体实现。

### Lines 645-672 / 第 645-672 行
````python
        "Tuned aten.addmm: m=%s, n=%s, k=%s, mat1_dtype=%s, mat2_dtype=%s, output_layout=%s",
        m,
        n,
        k,
        mat1.get_dtype(),
        mat2.get_dtype(),
        layout,
    )
    if (not is_nonzero) or (
        not (inductor_config.max_autotune or inductor_config.max_autotune_gemm)
    ):
        choices.extend(
            V.choices.get_template_configs(
                kernel_inputs_aten,
                [aten_addmm],
                name,
            )
        )
        node, _ = autotune_select_algorithm(
            name, choices, kernel_inputs.nodes(), layout
        )
        return node

    templates_to_use: list[ExternKernelChoice | KernelTemplate] = []

    if use_aten_gemm_kernels():
        aten_templates: list[ExternKernelChoice | KernelTemplate] = [aten_addmm]
        if (
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `templates_to_use`, and `aten_templates`. This range continues the implementation of function `tuned_addmm`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `templates_to_use`、`aten_templates` 等值。这一段延续了函数`tuned_addmm` 的具体实现。

### Lines 673-700 / 第 673-700 行
````python
            inp.get_stride()[0] == 0
            and len(inp.get_size()) == 2
            and inductor_config.triton.autotune_cublasLt
            and not V.graph.cpp_wrapper  # bias_addmm only has a Python implementation
        ):
            aten_templates.append(aten_bias_addmm)

        # On ROCm, ATen choices use original bias input; non-ROCm keeps unified inputs.
        choices.extend(
            V.choices.get_template_configs(kernel_inputs_aten, aten_templates, name)
        )

    if is_nonzero and use_triton_template(layout, check_max_autotune=False):
        templates_to_use.append(mm_template)

        if use_triton_blackwell_tma_template(
            mat1, mat2, output_layout=layout, add_guards=True
        ):
            templates_to_use.append(blackwell_ws_persistent_device_tma_mm_template)
        elif use_triton_tma_template(mat1, mat2, output_layout=layout, add_guards=True):
            if torch.version.hip is None:
                templates_to_use.append(persistent_tma_mm_template)
            else:
                templates_to_use.append(persistent_mm_template)

        templates_to_use.append(addmm_contiguous_subgraph_template)

    # Single unified call for all templates
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`. This range continues the implementation of function `tuned_addmm`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。这一段延续了函数`tuned_addmm` 的具体实现。

### Lines 701-728 / 第 701-728 行
````python
    choices.extend(
        V.choices.get_template_configs(kernel_inputs, templates_to_use, name)
    )

    if (
        is_nonzero
        and use_cutlass_template(layout, m, n, k)
        and _use_cutlass_for_op(name)
    ):
        CUTLASS3xGemmTemplate.add_cutlass_gemm_choices(
            choices,
            layout,
            # reorder here because CUTLASS expects (x, w, bias) but torch
            # is bias, x, w
            kernel_inputs.nodes(reorder=[1, 2, 0]),
            alpha=alpha,
            beta=beta,
            input_reorder=[2, 0, 1],
        )

    if is_nonzero and use_ck_gemm_template(layout, m, n, k):
        CKGemmTemplate.add_ck_gemm_choices(
            choices,
            layout,
            # reorder here because CK expects (x, w, bias) but torch
            # is bias, x, w
            kernel_inputs.nodes(reorder=[1, 2, 0]),
            alpha=alpha,
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `alpha`, `beta`, and `input_reorder`. This range continues the implementation of function `tuned_addmm`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `alpha`、`beta`、`input_reorder` 等值。这一段延续了函数`tuned_addmm` 的具体实现。

### Lines 729-756 / 第 729-756 行
````python
            beta=beta,
            input_reorder=[2, 0, 1],
        )

    if use_cpp_gemm_template(layout, mat1, mat2):
        CppGemmTemplate.add_choices(
            choices,
            layout,
            kernel_inputs.nodes(),
            alpha=alpha,
            beta=beta,
            has_bias=True,
        )

    node, _ = autotune_select_algorithm(name, choices, kernel_inputs.nodes(), layout)
    return node


@register_lowering(aten._sparse_semi_structured_mm, type_promotion_kind=None)
def tuned_sparse_semi_structured_mm(
    mat1, mat1_meta, mat2, *, out_dtype=None, layout=None
):
    from torch._inductor.select_algorithm import realize_inputs

    # TODO(coconturuben): support V.choices.get_mm_configs for sparse_semi_structured_mm
    mat1, mat1_meta, mat2 = realize_inputs(mat1, mat1_meta, mat2)
    m1, k1 = mat1.get_size()
    m2, _ = mat1_meta.get_size()
````
- **EN**: Imports dependencies such as `torch._inductor.select_algorithm` for the logic in this range. Introduces function `tuned_sparse_semi_structured_mm`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `torch._inductor.select_algorithm` 等依赖，为后续逻辑提供基础能力。这里定义了函数`tuned_sparse_semi_structured_mm`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 757-784 / 第 757-784 行
````python
    k2, n = mat2.get_size()
    m = V.graph.sizevars.check_equals_and_simplify(m1, m2)
    k = V.graph.sizevars.check_equals_and_simplify(2 * k1, k2)
    if layout is None:
        from torch._inductor.ir import FixedLayout

        layout = FixedLayout(
            mat2.get_device(),
            out_dtype if out_dtype else mat2.get_dtype(),
            [m, n],
            [n, 1],
        )
    else:
        assert out_dtype is None, "out_dtype is ignored if layout is specified."

    choices = (
        [
            aten__sparse_semi_structured_mm.bind(
                (mat1, mat1_meta, mat2), layout, out_dtype=out_dtype
            )
        ]
        if use_aten_gemm_kernels()
        else []
    )

    if (
        m * n != 0
        and use_cutlass_template(layout, m, n, k)
````
- **EN**: Imports dependencies such as `torch._inductor.ir` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `m`, `k`, `layout`, `else`, and `choices`.
- **CN**: 这里导入了 `torch._inductor.ir` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `m`、`k`、`layout`、`else`、`choices` 等值。

### Lines 785-812 / 第 785-812 行
````python
        and _use_cutlass_for_op("sparse_semi_structured_mm")
    ):
        CUTLASS2xGemmTemplate.add_cutlass_gemm_choices(
            choices, layout, [mat1, mat2, mat1_meta], fuseable=True, non_fuseable=True
        )

    node, _ = autotune_select_algorithm(
        "sparse_semi_structured_mm", choices, (mat1, mat1_meta, mat2), layout
    )
    return node


scaling_pairs = [
    (ScalingType.TensorWise, ScalingType.TensorWise),
    (ScalingType.RowWise, ScalingType.RowWise),
    (ScalingType.BlockWise1x128, ScalingType.BlockWise128x128),
    (ScalingType.BlockWise1x128, ScalingType.BlockWise1x128),
    (ScalingType.BlockWise128x128, ScalingType.BlockWise1x128),
]


epilogue_scaling_types = [ScalingType.TensorWise, ScalingType.RowWise]
main_loop_scaling_types = [ScalingType.BlockWise1x128, ScalingType.BlockWise128x128]


def _is_tensorwise_scaling(sz: Any) -> bool:
    return (len(sz) == 0) or all(
        V.graph.sizevars.statically_known_equals(d, 1) for d in sz
````
- **EN**: Introduces function `_is_tensorwise_scaling`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `scaling_pairs`, `epilogue_scaling_types`, and `main_loop_scaling_types`.
- **CN**: 这里定义了函数`_is_tensorwise_scaling`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `scaling_pairs`、`epilogue_scaling_types`、`main_loop_scaling_types` 等值。

### Lines 813-840 / 第 813-840 行
````python
    )


def _is_rowwise_scaling(sz: Any, transpose: bool) -> bool:
    idx = 0 if transpose else -1
    return V.graph.sizevars.statically_known_equals(sz[idx], 1)


def _is_blockwise1xTILESIZE_scaling(
    sz: Any, tensor_sz: Any, tile_size: int, transpose: bool
) -> bool:
    lhs = 1 if transpose else 0
    rhs = 0 if transpose else 1
    return V.graph.sizevars.statically_known_equals(
        sz[lhs], tensor_sz[lhs]
    ) and V.graph.sizevars.statically_known_equals(
        sz[rhs], ceildiv(tensor_sz[rhs], tile_size)
    )


def _is_blockwise128x128_scaling(sz: Any, tensor_sz: Any) -> bool:
    return V.graph.sizevars.statically_known_equals(
        sz[0], ceildiv(tensor_sz[0], 128)
    ) and V.graph.sizevars.statically_known_equals(sz[1], ceildiv(tensor_sz[1], 128))


def is_desired_scaling(
    t: Any,
````
- **EN**: Introduces function `_is_rowwise_scaling`, function `_is_blockwise1xTILESIZE_scaling`, function `_is_blockwise128x128_scaling`, function `is_desired_scaling`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `idx`, `sz`, `lhs`, `rhs`, and `t`.
- **CN**: 这里定义了函数`_is_rowwise_scaling`、函数`_is_blockwise1xTILESIZE_scaling`、函数`_is_blockwise128x128_scaling`、函数`is_desired_scaling`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `idx`、`sz`、`lhs`、`rhs`、`t` 等值。

### Lines 841-868 / 第 841-868 行
````python
    scale_size: torch.Tensor,
    scaling_type: ScalingType,
    transpose: bool = False,
) -> bool:
    match scaling_type:
        case ScalingType.TensorWise:
            return _is_tensorwise_scaling(scale_size)
        case ScalingType.RowWise:
            return _is_rowwise_scaling(scale_size, transpose)
        case ScalingType.BlockWise1x128:
            return _is_blockwise1xTILESIZE_scaling(
                scale_size, t.get_size(), 128, transpose
            )
        case ScalingType.BlockWise128x128:
            return _is_blockwise128x128_scaling(scale_size, t.get_size())
        case _:
            raise AssertionError(f"Unsupported scaling type {scaling_type}")


def get_tile_size(scale_option) -> int:
    match scale_option:
        case ScalingType.BlockWise128x128:
            return 128
        case ScalingType.BlockWise1x128:
            return 128
        case _:
            raise AssertionError(
                f"Unsupported scaling type {scale_option} in get_tile_size"
````
- **EN**: Introduces function `get_tile_size`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `scale_size`, `scaling_type`, and `transpose`.
- **CN**: 这里定义了函数`get_tile_size`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `scale_size`、`scaling_type`、`transpose` 等值。

### Lines 869-896 / 第 869-896 行
````python
            )


def get_scaling_options(
    mat_a: Any,
    mat_b: Any,
    scale_a_size: torch.Tensor,
    scale_b_size: torch.Tensor,
) -> tuple[ScalingType, ScalingType]:
    for scale_option_a, scale_option_b in scaling_pairs:
        if is_desired_scaling(
            mat_a, scale_a_size, scale_option_a
        ) and is_desired_scaling(mat_b, scale_b_size, scale_option_b, transpose=True):
            return scale_option_a, scale_option_b

    raise AssertionError(
        f"Inductor Triton does not support scale_a.shape = {scale_a_size}, scale_b.shape = {scale_b_size}"
    )  # verify that shapes are supported by at least one existing pairing


@register_lowering(aten._scaled_mm.default, type_promotion_kind=None)  # type: ignore[misc]
def tuned_scaled_mm(
    mat_a,
    mat_b,
    scale_a,
    scale_b,
    bias=None,
    scale_result=None,
````
- **EN**: Introduces function `get_scaling_options`, function `tuned_scaled_mm`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`get_scaling_options`、函数`tuned_scaled_mm`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 897-924 / 第 897-924 行
````python
    out_dtype=None,
    use_fast_accum=False,
    layout=None,
):
    """
    Performs an optimized matrix multiplication where scaling factors are applied
    to the inputs and/or output.

    Args:
        mat1 (Tensor): First input matrix
        mat2 (Tensor): Second input matrix
        scale1 (Tensor): Scale factor applied to mat1 (supports broadcasting)
        scale2 (Tensor): Scale factor applied to mat2 (supports broadcasting)
        bias (Tensor, optional): Optional bias tensor to add to the result
        layout: Layout hint for optimization

    Returns:
        Tensor: The result of the scaled matrix multiplication
    """
    # TODO(coconutruben): integrate into MMKernelInputs when all callsites use that
    m, n, k, layout, mat_a, mat_b = mm_args(
        mat_a, mat_b, layout=layout, out_dtype=out_dtype
    )
    # below is for getting an overview logging info of inductor mms
    counters["aten_mm_info"][f"aten._scaled_mm.default_{m}_{n}_{k}"] += 1
    log.info(
        "Tuned aten._scaled_mm.default: m=%s, n=%s, k=%s, mat1_dtype=%s, mat2_dtype=%s, output_layout=%s",
        m,
````
- **EN**: Initializes or updates values such as `out_dtype`, `use_fast_accum`, `layout`, `Args`, `Returns`, and `Tensor`. This range continues the implementation of function `tuned_scaled_mm`.
- **CN**: 初始化或更新了 `out_dtype`、`use_fast_accum`、`layout`、`Args`、`Returns`、`Tensor` 等值。这一段延续了函数`tuned_scaled_mm` 的具体实现。

### Lines 925-952 / 第 925-952 行
````python
        n,
        k,
        mat_a.get_dtype(),
        mat_b.get_dtype(),
        layout,
    )
    name = "scaled_mm"
    check_supported_striding(mat_a, mat_b)

    scale_a_real, scale_b_real = realize_inputs(scale_a, scale_b)

    input_nodes: list[Any]

    if not bias:
        input_nodes = [mat_a, mat_b, scale_a_real, scale_b_real]
    else:
        bias_real = realize_inputs(bias)
        input_nodes = [mat_a, mat_b, scale_a_real, scale_b_real, bias_real]

    # Create MMKernelInputs for Scaled MM (matrices are at indices 0, 1)
    kernel_inputs = MMKernelInputs(
        input_nodes, mat1_idx=0, mat2_idx=1, out_dtype=out_dtype
    )

    choices: list[ChoiceCaller] = []

    # Collect all templates for unified call
    templates_to_use: list[ExternKernelChoice | KernelTemplate] = []
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `input_nodes`, `else`, `bias_real`, `kernel_inputs`, `choices`, and `...+1`. This range continues the implementation of function `tuned_scaled_mm`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`input_nodes`、`else`、`bias_real`、`kernel_inputs`、`choices`、`另有1项` 等值。这一段延续了函数`tuned_scaled_mm` 的具体实现。

### Lines 953-980 / 第 953-980 行
````python
    kwarg_overrides = {}

    if use_aten_gemm_kernels():
        templates_to_use.append(aten__fp8_mm)
        kwarg_overrides[aten__fp8_mm.uid] = dict(
            out_dtype=out_dtype, use_fast_accum=use_fast_accum
        )

    _, is_nonzero = _is_static_problem(layout)

    if (
        # We dont have triton lowerings for the MX variants yet
        scale_a.dtype == torch.float32
        and is_nonzero
        and use_triton_template(layout, enable_float8=True, check_max_autotune=False)
    ):
        overriders = dict(USE_FAST_ACCUM=use_fast_accum)

        scale_a_size, scale_b_size = scale_a_real.shape, scale_b_real.shape

        scale_option_a, scale_option_b = get_scaling_options(
            mat_a, mat_b, scale_a_size, scale_b_size
        )

        # TODO (paulzhan): There is no template that exists for bias and TMA
        # Don't run tma template currently if bias exist
        if (
            use_triton_tma_template(mat_a, mat_b, output_layout=layout, add_guards=True)
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kwarg_overrides`, `out_dtype`, and `overriders`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `kwarg_overrides`、`out_dtype`、`overriders` 等值。

### Lines 981-1008 / 第 981-1008 行
````python
            and not bias
        ):
            overriders["SCALE_RECIPE_A"] = scale_option_a.value
            overriders["SCALE_RECIPE_B"] = scale_option_b.value

            if use_triton_scaling_template(
                scale_option_a, scale_option_b, epilogue_scaling_types
            ):
                templates_to_use.append(scaled_mm_device_tma_epilogue_scaling_template)
                kwarg_overrides[scaled_mm_device_tma_epilogue_scaling_template.uid] = (
                    overriders
                )
            elif use_triton_scaling_template(
                scale_option_a, scale_option_b, main_loop_scaling_types
            ):
                overriders["TILE_SIZE_A"] = get_tile_size(scale_option_a)
                overriders["TILE_SIZE_B"] = get_tile_size(scale_option_b)

                templates_to_use.append(scaled_mm_device_tma_main_loop_scaling_template)
                kwarg_overrides[scaled_mm_device_tma_main_loop_scaling_template.uid] = (
                    overriders
                )
            else:
                raise AssertionError(
                    "Inductor Triton does not support scaling options that are present "
                    + "in both epilogue scaling and main loop scaling"
                )

````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 1009-1036 / 第 1009-1036 行
````python
        if (
            use_triton_blackwell_tma_template(
                mat_a, mat_b, output_layout=layout, add_guards=True
            )
            and not bias
        ):
            templates_to_use.append(blackwell_ws_persistent_device_tma_mm_template)
            kwarg_overrides[blackwell_ws_persistent_device_tma_mm_template.uid] = (
                overriders
            )

        if use_triton_scaling_template(
            scale_option_a, scale_option_b, epilogue_scaling_types
        ):
            templates_to_use.append(mm_template)
            kwarg_overrides[mm_template.uid] = overriders

    # Single unified call for all templates
    choices.extend(
        V.choices.get_template_configs(
            kernel_inputs,
            templates_to_use,
            name,
            kwarg_overrides=kwarg_overrides,
        )
    )

    # NVGEMM get_kernels() will return empty if the scaling mode/dtype is unsupported
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kwarg_overrides`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `kwarg_overrides` 等值。

### Lines 1037-1064 / 第 1037-1064 行
````python
    if is_nonzero and use_nv_universal_gemm_template(layout, m, n, k, mat_a, mat_b):
        from ..codegen.nv_universal_gemm import add_nv_universal_scaled_gemm_choices

        add_nv_universal_scaled_gemm_choices(
            choices,
            layout,
            input_nodes,
            kernel_inputs=kernel_inputs,
        )

    # Early return for MX variants
    if scale_a.dtype != torch.float32:
        node, _ = autotune_select_algorithm(name, choices, input_nodes, layout)
        return node

    if (
        is_nonzero
        and use_cutlass_template(layout, m, n, k)
        and _use_cutlass_for_op(name)
    ):
        CUTLASS3xGemmTemplate.add_cutlass_gemm_choices(
            choices,
            layout,
            kernel_inputs.nodes(),  # type: ignore[arg-type]
            use_fast_accum=use_fast_accum,  # type: ignore[arg-type]
        )

    if is_nonzero and use_ck_gemm_template(layout, m, n, k):
````
- **EN**: Imports dependencies such as `..codegen.nv_universal_gemm` for the logic in this range. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `..codegen.nv_universal_gemm` 等依赖，为后续逻辑提供基础能力。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1065-1092 / 第 1065-1092 行
````python
        CKGemmTemplate.add_ck_gemm_choices(choices, layout, kernel_inputs.nodes())

    node, _ = autotune_select_algorithm(name, choices, kernel_inputs.nodes(), layout)
    return node


@functools.cache
def _is_sm7x_or_older_gpu(index: int | None) -> bool:
    props = torch.cuda.get_device_properties(index or 0)
    return props.major <= 7


def dims_are_int(dims):
    return all(isinstance(dim, int) for dim in dims)


def mm_autoheuristic(
    mat1,
    mat2,
    m,
    n,
    k,
    choices,
    name,
    input_nodes,
    ops,
    precondition,
    top_k: int | None = None,
````
- **EN**: Introduces function `_is_sm7x_or_older_gpu`, function `dims_are_int`, function `mm_autoheuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`_is_sm7x_or_older_gpu`、函数`dims_are_int`、函数`mm_autoheuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

### Lines 1093-1120 / 第 1093-1120 行
````python
    always_included=None,
):
    m, n, k = get_size_hints(mat1, mat2, m, n, k)
    if not dims_are_int([m, n, k]):
        return None
    mat1_stride, mat2_stride = get_size_hints_strides(mat1, mat2)

    def get_context(m, k, n, mat1, mat2, mat1_stride, mat2_stride):
        context = AHContext()
        context.add_feature("m", m)
        context.add_feature("k", k)
        context.add_feature("n", n)
        context.add_feature("mat1_dtype", mat1.layout.dtype, is_categorical=True)
        context.add_feature("mat2_dtype", mat2.layout.dtype, is_categorical=True)
        context_add_strides(context, "mat1", mat1_stride)
        context_add_strides(context, "mat2", mat2_stride)
        context.add_feature(
            "mat1_iscontig", mat1.layout.is_contiguous(), is_categorical=True
        )
        context.add_feature(
            "mat2_iscontig", mat2.layout.is_contiguous(), is_categorical=True
        )
        if name == "mm":
            context_add_using_tf32(context, mat1.layout.dtype)
        return context

    def fallback():
        return None
````
- **EN**: Introduces function `get_context`, function `fallback`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_context`、函数`fallback`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1121-1148 / 第 1121-1148 行
````python

    context = get_context(m, k, n, mat1, mat2, mat1_stride, mat2_stride)
    autoheuristic = AutoHeuristicSelectAlgorithm(
        fallback=fallback,
        choices=choices,
        input_nodes=input_nodes,
        context=context,
        name=name,
        augment_context=ops,
        precondition=precondition,
    )

    if top_k is not None:
        # TODO: is there a cleaner way to ensure aten.mm is always included?
        return autoheuristic.get_top_k_choices_caller(
            top_k, always_included=always_included
        )

    return autoheuristic.get_choice_caller()


def get_size_hints(mat1, mat2, m, n, k):
    if not isinstance(m, int) or not isinstance(k, int):
        (m, k) = V.graph.sizevars.optimization_hints(mat1.get_size())

    if not isinstance(n, int) or not isinstance(k, int):
        (k, n) = V.graph.sizevars.optimization_hints(mat2.get_size())
    return m, n, k
````
- **EN**: Introduces function `get_size_hints`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `context`, `autoheuristic`, `fallback`, `choices`, `input_nodes`, `name`, and `...+2`.
- **CN**: 这里定义了函数`get_size_hints`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `context`、`autoheuristic`、`fallback`、`choices`、`input_nodes`、`name`、`另有2项` 等值。

### Lines 1149-1160 / 第 1149-1160 行
````python


def get_size_hints_strides(mat1, mat2):
    mat1_stride = mat1.layout.stride
    mat2_stride = mat2.layout.stride
    strides = [mat1_stride, mat2_stride]
    strides_hints = []
    for stride in strides:
        if not isinstance(stride, int):
            stride = V.graph.sizevars.optimization_hints(stride)
        strides_hints.append(stride)
    return strides_hints[0], strides_hints[1]
````
- **EN**: Introduces function `get_size_hints_strides`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mat1_stride`, `mat2_stride`, `strides`, `strides_hints`, and `stride`.
- **CN**: 这里定义了函数`get_size_hints_strides`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `mat1_stride`、`mat2_stride`、`strides`、`strides_hints`、`stride` 等值。

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
- **Standard library / 标准库**: `functools`, `logging`, `typing`
- **Third-party / 第三方**: `triton`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._dynamo.utils`, `torch._inductor.autoheuristic.autoheuristic`, `torch._inductor.autoheuristic.autoheuristic_utils`, `torch._inductor.codegen.cpp_gemm_template`, `torch._inductor.remote_gemm_autotune_cache`, `torch._inductor.virtualized`, `torch.fx.experimental.proxy_tensor`, `torch.nn.functional`, `torch.torch_version`, `..`, `..codegen.cutlass.gemm_template`, `..codegen.rocm.ck_tile_universal_gemm_template`, `..codegen.rocm.ck_universal_gemm_template`, `..codegen.subgraph`, `..ir`, `..kernel_inputs`, `..lowering`, `..select_algorithm`, `..utils`, `...+6`
