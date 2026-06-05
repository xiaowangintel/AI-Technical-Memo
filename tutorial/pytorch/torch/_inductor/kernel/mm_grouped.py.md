# mm_grouped.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel/mm_grouped.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `Config`. It exposes functions such as `grouped_mm_configs`, `early_config_prune`, `grouped_mm_args`, `can_use_triton_kernel`, `create_offsets`, `_tuned_grouped_mm_common`, and `...+2`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `Config` 等类。同时提供 `grouped_mm_configs`、`early_config_prune`、`grouped_mm_args`、`can_use_triton_kernel`、`create_offsets`、`_tuned_grouped_mm_common`、`另有2项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import logging
from dataclasses import asdict, dataclass
from typing import Any

import torch
from torch._dynamo.utils import counters
from torch._inductor.codegen.cutedsl.cutedsl_template import CuteDSLTemplate
from torch._inductor.runtime.triton_compat import tl
from torch._inductor.template_heuristics.cutedsl import get_groupgemm_configs
from torch._inductor.virtualized import V
from torch.utils._triton import has_triton

from ..ir import ChoiceCaller, Layout, TensorBox
from ..lowering import register_lowering
from ..select_algorithm import (
    autotune_select_algorithm,
    ExternKernelChoice,
    realize_inputs,
    TritonTemplate,
````
- **EN**: Imports dependencies such as `logging`, `dataclasses`, `typing`, `torch`, `torch._dynamo.utils`, `torch._inductor.codegen.cutedsl.cutedsl_template`, and `...+7` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里导入了 `logging`、`dataclasses`、`typing`、`torch`、`torch._dynamo.utils`、`torch._inductor.codegen.cutedsl.cutedsl_template`、`另有7项` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 21-40 / 第 21-40 行
````python
)
from ..utils import (
    get_gpu_shared_memory,
    get_num_sms,
    has_free_symbols,
    use_aten_gemm_kernels,
    use_blackwell_cutedsl_grouped_mm,
    use_nv_universal_gemm_template,
    use_triton_template,
)
from .mm_common import (
    _is_static_problem,
    check_supported_striding,
    load_kernel_template,
    persistent_grouped_mm_grid,
)


log = logging.getLogger(__name__)
aten = torch.ops.aten
````
- **EN**: Imports dependencies such as `..utils`, and `.mm_common` for the logic in this range. Initializes or updates values such as `log`, and `aten`.
- **CN**: 这里导入了 `..utils`、`.mm_common` 等依赖，为后续逻辑提供基础能力。初始化或更新了 `log`、`aten` 等值。

### Lines 41-60 / 第 41-60 行
````python


@dataclass
class Config:
    kwargs: dict[str, int]
    num_stages: int
    num_warps: int


_NV_CONFIGS = [
    Config(
        {
            "BLOCK_M": block_size_m,
            "BLOCK_N": block_size_n,
            "BLOCK_K": block_size_k,
            "NUM_CONSUMER_GROUPS": 1,
        },
        num_stages=num_stages,
        num_warps=num_warps,
    )
````
- **EN**: Introduces class `Config`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Initializes or updates values such as `kwargs`, `num_stages`, `num_warps`, and `_NV_CONFIGS`.
- **CN**: 这里定义了类`Config`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。初始化或更新了 `kwargs`、`num_stages`、`num_warps`、`_NV_CONFIGS` 等值。

### Lines 61-80 / 第 61-80 行
````python
    for block_size_m in [16, 32, 64, 128]
    for block_size_n in [64, 128, 256]
    for block_size_k in [64, 128, 256]
    for num_stages in [3, 4]
    for num_warps in [4, 8]
]


def grouped_mm_configs():
    return _NV_CONFIGS


def early_config_prune(g, m, dtsize, configs, named_args):
    pruned_configs = []
    for config in configs:
        kw = config.kwargs
        BLOCK_M, BLOCK_N, BLOCK_K, num_stages, num_warps, num_consumer_groups = (
            kw["BLOCK_M"],
            kw["BLOCK_N"],
            kw["BLOCK_K"],
````
- **EN**: Introduces function `grouped_mm_configs`, function `early_config_prune`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `pruned_configs`, and `kw`.
- **CN**: 这里定义了函数`grouped_mm_configs`、函数`early_config_prune`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `pruned_configs`、`kw` 等值。

### Lines 81-100 / 第 81-100 行
````python
            config.num_stages,
            config.num_warps,
            getattr(config, "num_consumer_groups", 0),
        )

        # 1. Prune NV configs depending on g and m.
        if not has_free_symbols((g, m)):
            a_is_2d, b_is_2d = named_args["A_IS_2D"], named_args["B_IS_2D"]
            m_avg = m // g if a_is_2d and not b_is_2d else m
            if m_avg <= 16:
                if BLOCK_M > 32:
                    continue
            elif m_avg <= 32:
                if BLOCK_M > 64:
                    continue
            elif m_avg <= 64:
                if BLOCK_M <= 16:
                    continue
            else:
                if BLOCK_M <= 32:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `m_avg`, and `else`. This range continues the implementation of function `early_config_prune`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `m_avg`、`else` 等值。这一段延续了函数`early_config_prune` 的具体实现。

### Lines 101-120 / 第 101-120 行
````python
                    continue

        # 2. make sure we have enough smem
        max_shared_memory = get_gpu_shared_memory()

        required_shared_memory = (BLOCK_M + BLOCK_N) * BLOCK_K * num_stages * dtsize
        if required_shared_memory > max_shared_memory:
            continue

        use_warp_specialization = num_consumer_groups >= 1

        # 3. make sure we can partition for ws
        if use_warp_specialization:
            if num_warps != 4:
                continue

            # "tritongpu-warp-spec-data-partition"
            m_slice = BLOCK_M // num_consumer_groups
            n_slice = BLOCK_N // num_consumer_groups
            if m_slice < 64 and n_slice < 256:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `max_shared_memory`, `required_shared_memory`, `use_warp_specialization`, `m_slice`, and `n_slice`. This range continues the implementation of function `early_config_prune`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `max_shared_memory`、`required_shared_memory`、`use_warp_specialization`、`m_slice`、`n_slice` 等值。这一段延续了函数`early_config_prune` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
                continue

        pruned_configs.append(config)

    return pruned_configs


triton_grouped_mm_template = TritonTemplate(
    name="grouped_mm",
    grid=persistent_grouped_mm_grid,
    source=load_kernel_template("triton_mm_grouped"),
)

triton_scaled_grouped_mm_template = TritonTemplate(
    name="scaled_grouped_mm",
    grid=persistent_grouped_mm_grid,
    source=load_kernel_template("triton_mm_grouped"),
)

cutedsl_grouped_mm_template = CuteDSLTemplate(
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `triton_grouped_mm_template`, `name`, `grid`, `source`, `triton_scaled_grouped_mm_template`, and `cutedsl_grouped_mm_template`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `triton_grouped_mm_template`、`name`、`grid`、`source`、`triton_scaled_grouped_mm_template`、`cutedsl_grouped_mm_template` 等值。

### Lines 141-160 / 第 141-160 行
````python
    name="grouped_gemm_cutedsl",
    source=load_kernel_template("cutedsl_mm_grouped"),
)


def grouped_mm_args(
    mat1: TensorBox,
    mat2: TensorBox,
    offs: TensorBox | None,
    layout=None,
    out_dtype=None,
):
    mat1, mat2 = realize_inputs(mat1, mat2)
    if offs is not None:
        realize_inputs(offs)
    mat1_size = mat1.get_size()
    mat2_size = mat2.get_size()

    m1dim, m2dim = len(mat1_size), len(mat2_size)

````
- **EN**: Introduces function `grouped_mm_args`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `name`, `source`, `mat1`, `mat2`, `offs`, `layout`, and `...+3`.
- **CN**: 这里定义了函数`grouped_mm_args`。包含分支、循环或上下文管理等控制流。初始化或更新了 `name`、`source`、`mat1`、`mat2`、`offs`、`layout`、`另有3项` 等值。

### Lines 161-180 / 第 161-180 行
````python
    assert m1dim == 2 or m1dim == 3
    assert m2dim == 2 or m2dim == 3

    if layout is None:
        from torch._inductor.ir import FixedLayout

        if out_dtype is None:
            out_dtype = mat1.get_dtype()
        alignment = 16 // out_dtype.itemsize

        if m1dim == 2:
            if m2dim == 2:
                assert offs is not None
                out_size = [offs.get_size()[0], mat1_size[0], mat2_size[1]]
            else:
                out_size = [mat1_size[0], mat2_size[-1]]
        else:
            if m2dim == 2:
                out_size = [mat1_size[1], mat2_size[1]]
            else:
````
- **EN**: Imports dependencies such as `torch._inductor.ir` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `out_dtype`, `alignment`, `out_size`, and `else`.
- **CN**: 这里导入了 `torch._inductor.ir` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `out_dtype`、`alignment`、`out_size`、`else` 等值。

### Lines 181-200 / 第 181-200 行
````python
                out_size = [mat1_size[0], mat1_size[1], mat2_size[-1]]
        size_padded = (out_size[-1] + alignment - 1) // alignment * alignment
        if len(out_size) == 2:
            out_stride = [size_padded, 1]
        else:
            out_stride = [out_size[1] * size_padded, size_padded, 1]

        layout = FixedLayout(
            mat1.get_device(),
            out_dtype,
            out_size,
            out_stride,
        )
    else:
        assert out_dtype is None, "out_dtype is ignored if layout is specified."

    return (mat1_size, mat2_size, layout, mat1, mat2, offs)


aten__grouped_mm = ExternKernelChoice(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `out_size`, `size_padded`, `out_stride`, `else`, `layout`, and `aten__grouped_mm`. This range continues the implementation of function `grouped_mm_args`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `out_size`、`size_padded`、`out_stride`、`else`、`layout`、`aten__grouped_mm` 等值。这一段延续了函数`grouped_mm_args` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python
    torch._grouped_mm,
    "at::_grouped_mm",
    op_overload=aten._grouped_mm.default,
    has_out_variant=False,
)


aten__scaled_grouped_mm = ExternKernelChoice(
    torch._scaled_grouped_mm,
    "at::_scaled_grouped_mm",
    op_overload=aten._scaled_grouped_mm.default,
    has_out_variant=False,
)


def can_use_triton_kernel(
    mat_a: TensorBox,
    mat_b: TensorBox,
    offs: TensorBox | None,
    bias: TensorBox | None,
````
- **EN**: Introduces function `can_use_triton_kernel`. Initializes or updates values such as `op_overload`, `has_out_variant`, `aten__scaled_grouped_mm`, `mat_a`, `mat_b`, `offs`, and `...+1`.
- **CN**: 这里定义了函数`can_use_triton_kernel`。初始化或更新了 `op_overload`、`has_out_variant`、`aten__scaled_grouped_mm`、`mat_a`、`mat_b`、`offs`、`另有1项` 等值。

### Lines 221-240 / 第 221-240 行
````python
    scale_result: TensorBox | None,
) -> bool:
    if not (
        torch.cuda.is_available()
        and torch.cuda.get_device_capability() >= (9, 0)
        and not torch.version.hip
    ):
        return False
    if not has_triton():
        return False

    # The _grouped_mm()/_scaled_grouped_mm() operator do not support
    # bias nor scale_result yet.
    if bias is not None:
        return False
    if scale_result is not None:
        return False

    if len(mat_a.get_size()) == 2 or len(mat_b.get_size()) == 2:
        return offs is not None
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `scale_result`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `scale_result` 等值。

### Lines 241-260 / 第 241-260 行
````python
    else:
        return offs is None


def create_offsets(offs_box, m1_is_2d, m2_is_2d, m, n, k, alignment):
    if m1_is_2d:
        if m2_is_2d:
            end = k
        else:
            end = m
    else:
        if m2_is_2d:
            end = n
        else:
            return None

    end_hint = V.graph.sizevars.optimization_hint(end)
    noffs_hint = V.graph.sizevars.optimization_hint(offs_box.get_size()[0])
    offs = torch.arange(1, noffs_hint + 1, dtype=torch.float32) * (
        end_hint / noffs_hint
````
- **EN**: Introduces function `create_offsets`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`create_offsets`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 261-280 / 第 261-280 行
````python
    )
    offs[:-1] = (offs[:-1] / alignment).round() * alignment
    offs[-1] = end_hint
    return offs.to(dtype=offs_box.get_dtype(), device=offs_box.get_device())


def _tuned_grouped_mm_common(
    operator_name: str,
    algorithm_name: str,
    extern_kernel_choice: ExternKernelChoice,
    kernel_template: TritonTemplate,
    mat_a: TensorBox,
    mat_b: TensorBox,
    scale_a: TensorBox | None = None,
    scale_b: TensorBox | None = None,
    offs: TensorBox | None = None,
    bias: TensorBox | None = None,
    scale_result: TensorBox | None = None,
    out_dtype: torch.dtype | None = None,
    use_fast_accum: bool | None = None,
````
- **EN**: Introduces function `_tuned_grouped_mm_common`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_tuned_grouped_mm_common`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 281-300 / 第 281-300 行
````python
    layout: Layout | None = None,
) -> TensorBox:
    assert (scale_a is None) == (scale_b is None)
    assert scale_result is None or scale_a is not None

    m1_size, m2_size, layout, mat_a, mat_b, offs = grouped_mm_args(
        mat_a, mat_b, offs, layout=layout, out_dtype=out_dtype
    )
    counters["aten_mm_info"][operator_name] += 1
    log_message = f"Tuned {operator_name}: mat1_shape=%s, mat2_shape=%s, mat1_dtype=%s, mat2_dtype=%s, output_layout=%s"
    log.info(
        log_message,
        m1_size,
        m2_size,
        mat_a.get_dtype(),
        mat_b.get_dtype(),
        layout,
    )

    if scale_a is not None and scale_b is not None:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `layout`, and `log_message`. This range continues the implementation of function `_tuned_grouped_mm_common`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `layout`、`log_message` 等值。这一段延续了函数`_tuned_grouped_mm_common` 的具体实现。

### Lines 301-320 / 第 301-320 行
````python
        check_supported_striding(mat_a, mat_b)

    # workaround for Inductor not supporting optional tensor input arguments
    input_nodes: list[Any] = [mat_a, mat_b]
    if scale_a is not None:
        input_nodes.append(realize_inputs(scale_a))
    if scale_b is not None:
        input_nodes.append(realize_inputs(scale_b))
    if offs is not None:
        input_nodes.append(realize_inputs(offs))

    if use_fast_accum is None:
        aten_choice = extern_kernel_choice.bind(
            input_nodes,
            layout,
            out_dtype=out_dtype,
        )
    else:
        aten_choice = extern_kernel_choice.bind(
            input_nodes,
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_nodes`, `aten_choice`, `out_dtype`, and `else`. This range continues the implementation of function `_tuned_grouped_mm_common`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `input_nodes`、`aten_choice`、`out_dtype`、`else` 等值。这一段延续了函数`_tuned_grouped_mm_common` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python
            layout,
            out_dtype=out_dtype,
            use_fast_accum=use_fast_accum,
        )
    if use_fast_accum is None:
        use_fast_accum = False

    choices: list[ChoiceCaller] = []
    if use_aten_gemm_kernels():
        choices.append(aten_choice)

    _, is_nonzero = _is_static_problem(layout)

    # Checking only for the equality of corresponding dims of
    # multiplicands here, relying on meta function checks for
    # everything else.
    if len(m1_size) == 2:
        if len(m2_size) == 2:
            m, k1 = m1_size
            k2, n = m2_size
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `out_dtype`, `use_fast_accum`, and `choices`. This range continues the implementation of function `_tuned_grouped_mm_common`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `out_dtype`、`use_fast_accum`、`choices` 等值。这一段延续了函数`_tuned_grouped_mm_common` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python

            g = offs.get_size()[0]
            k = V.graph.sizevars.check_equals(k1, k2)
            a_is_2d, b_is_2d = True, True
        else:
            g1 = offs.layout.size[0]
            m, k1 = m1_size
            g2, k2, n = m2_size
            g = V.graph.sizevars.check_equals_and_simplify(g1, g2)
            k = V.graph.sizevars.check_equals(k1, k2)
            a_is_2d, b_is_2d = True, False
    else:
        if len(m2_size) == 2:
            g1 = offs.layout.size[0]
            g2, m, k1 = m1_size
            k2, n = m2_size
            g = V.graph.sizevars.check_equals_and_simplify(g1, g2)
            k = V.graph.sizevars.check_equals(k1, k2)
            a_is_2d, b_is_2d = False, True
        else:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `g`, `k`, `else`, and `g1`. This range continues the implementation of function `_tuned_grouped_mm_common`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `g`、`k`、`else`、`g1` 等值。这一段延续了函数`_tuned_grouped_mm_common` 的具体实现。

### Lines 361-380 / 第 361-380 行
````python
            g1, m, k1 = m1_size
            g2, k2, n = m2_size
            g = V.graph.sizevars.check_equals_and_simplify(g1, g2)
            k = V.graph.sizevars.check_equals(k1, k2)
            a_is_2d, b_is_2d = False, False

    if (
        is_nonzero
        and use_triton_template(layout)
        and can_use_triton_kernel(mat_a, mat_b, offs, bias, scale_result)
    ):
        scaled = scale_a is not None

        a_is_k_major = mat_a.get_stride()[-1] == 1
        b_is_k_major = mat_b.get_stride()[-2] == 1

        triton_has_make_tensor_descriptor = hasattr(tl, "make_tensor_descriptor")
        triton_has_experimental_make_tensor_descriptor = hasattr(
            tl, "_experimental_make_tensor_descriptor"
        )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `g`, `k`, `scaled`, `a_is_k_major`, `b_is_k_major`, `triton_has_make_tensor_descriptor`, and `...+1`. This range continues the implementation of function `_tuned_grouped_mm_common`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `g`、`k`、`scaled`、`a_is_k_major`、`b_is_k_major`、`triton_has_make_tensor_descriptor`、`另有1项` 等值。这一段延续了函数`_tuned_grouped_mm_common` 的具体实现。

### Lines 381-400 / 第 381-400 行
````python
        use_tma_load = (
            triton_has_make_tensor_descriptor
            or triton_has_experimental_make_tensor_descriptor
        )
        kwargs = {
            "SCALED": scaled,
            "A_IS_2D": a_is_2d,
            "B_IS_2D": b_is_2d,
            "A_IS_K_MAJOR": a_is_k_major,
            "B_IS_K_MAJOR": b_is_k_major,
            "USE_FAST_ACCUM": use_fast_accum,
            "NUM_SMS": get_num_sms(),
            "USE_TMA_LOAD": use_tma_load,
            "USE_EXPERIMENTAL_MAKE_TENSOR_DESCRIPTOR": triton_has_experimental_make_tensor_descriptor,
        }

        for config in early_config_prune(
            g, m, mat_a.dtype.itemsize, grouped_mm_configs(), kwargs
        ):
            kernel_template.maybe_append_choice(
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `use_tma_load`, and `kwargs`. This range continues the implementation of function `_tuned_grouped_mm_common`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `use_tma_load`、`kwargs` 等值。这一段延续了函数`_tuned_grouped_mm_common` 的具体实现。

### Lines 401-420 / 第 401-420 行
````python
                choices,
                input_nodes=input_nodes,
                layout=layout,
                num_stages=config.num_stages,
                num_warps=config.num_warps,
                **kwargs,
                **config.kwargs,
            )

    if use_blackwell_cutedsl_grouped_mm(
        mat_a, mat_b, layout, a_is_2d, b_is_2d, offs, bias, scale_result
    ):
        for config in get_groupgemm_configs():
            kwargs = dict(
                ACC_DTYPE="cutlass.Float32",
            )

            cutedsl_grouped_mm_template.maybe_append_choice(
                choices,
                input_nodes=input_nodes,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_nodes`, `layout`, `num_stages`, `num_warps`, `kwargs`, and `ACC_DTYPE`. This range continues the implementation of function `_tuned_grouped_mm_common`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_nodes`、`layout`、`num_stages`、`num_warps`、`kwargs`、`ACC_DTYPE` 等值。这一段延续了函数`_tuned_grouped_mm_common` 的具体实现。

### Lines 421-440 / 第 421-440 行
````python
                layout=layout,
                **kwargs,
                **asdict(config),
            )

    if (
        is_nonzero
        and a_is_2d
        and not b_is_2d
        and offs is not None
        and use_nv_universal_gemm_template(layout, m, n, k, mat_a, mat_b, offs, g)
    ):
        from torch._inductor.codegen.nv_universal_gemm.nv_universal_gemm import (
            add_nv_universal_grouped_gemm_choices,
        )

        add_nv_universal_grouped_gemm_choices(
            choices,
            layout,
            input_nodes,
````
- **EN**: Imports dependencies such as `torch._inductor.codegen.nv_universal_gemm.nv_universal_gemm` for the logic in this range. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `layout`. This range continues the implementation of function `_tuned_grouped_mm_common`.
- **CN**: 这里导入了 `torch._inductor.codegen.nv_universal_gemm.nv_universal_gemm` 等依赖，为后续逻辑提供基础能力。包含分支、循环或上下文管理等控制流。初始化或更新了 `layout` 等值。这一段延续了函数`_tuned_grouped_mm_common` 的具体实现。

### Lines 441-460 / 第 441-460 行
````python
            accumulator_type=torch.float32,
        )

    input_gen_fns = {}
    if offs is not None:
        input_offs_idx = 2 if scale_a is None else 4
        alignment = 16 // mat_a.dtype.itemsize
        input_gen_fns[input_offs_idx] = lambda x: create_offsets(
            x, a_is_2d, b_is_2d, m, n, k, alignment
        )
    node, _ = autotune_select_algorithm(
        algorithm_name, choices, input_nodes, layout, input_gen_fns=input_gen_fns
    )
    return node


@register_lowering(aten._grouped_mm.default, type_promotion_kind=None)
def tuned_grouped_mm(
    mat_a: TensorBox,
    mat_b: TensorBox,
````
- **EN**: Introduces function `tuned_grouped_mm`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`tuned_grouped_mm`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 461-480 / 第 461-480 行
````python
    offs: TensorBox | None = None,
    bias: TensorBox | None = None,
    out_dtype: torch.dtype | None = None,
    layout: Layout | None = None,
) -> TensorBox:
    """Auto-tuning for _grouped_mm() operator."""

    return _tuned_grouped_mm_common(
        "aten._grouped_mm.default",
        "grouped_mm",
        aten__grouped_mm,
        triton_grouped_mm_template,
        mat_a,
        mat_b,
        None,
        None,
        offs,
        bias,
        None,
        out_dtype,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `offs`, `bias`, `out_dtype`, and `layout`. This range continues the implementation of function `tuned_grouped_mm`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `offs`、`bias`、`out_dtype`、`layout` 等值。这一段延续了函数`tuned_grouped_mm` 的具体实现。

### Lines 481-500 / 第 481-500 行
````python
        None,
        layout,
    )


@register_lowering(aten._scaled_grouped_mm.default, type_promotion_kind=None)
def tuned_scaled_grouped_mm(
    mat_a: TensorBox,
    mat_b: TensorBox,
    scale_a: TensorBox,
    scale_b: TensorBox,
    offs: TensorBox | None = None,
    bias: TensorBox | None = None,
    scale_result: TensorBox | None = None,
    out_dtype: torch.dtype | None = None,
    use_fast_accum: bool = False,
    layout: Layout | None = None,
) -> TensorBox:
    """Auto-tuning for _scaled_grouped_mm() operator."""

````
- **EN**: Introduces function `tuned_scaled_grouped_mm`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`tuned_scaled_grouped_mm`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 501-519 / 第 501-519 行
````python
    # matching _scaled_grouped_mm_cuda Blas.cpp implementation
    out_dtype = out_dtype or torch.bfloat16

    return _tuned_grouped_mm_common(
        "aten._scaled_grouped_mm.default",
        "scaled_grouped_mm",
        aten__scaled_grouped_mm,
        triton_scaled_grouped_mm_template,
        mat_a,
        mat_b,
        scale_a,
        scale_b,
        offs,
        bias,
        scale_result,
        out_dtype,
        use_fast_accum,
        layout,
    )
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `out_dtype`. This range continues the implementation of function `tuned_scaled_grouped_mm`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `out_dtype` 等值。这一段延续了函数`tuned_scaled_grouped_mm` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
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
- **Standard library / 标准库**: `logging`, `dataclasses`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._dynamo.utils`, `torch._inductor.codegen.cutedsl.cutedsl_template`, `torch._inductor.runtime.triton_compat`, `torch._inductor.template_heuristics.cutedsl`, `torch._inductor.virtualized`, `torch.utils._triton`, `..ir`, `..lowering`, `..select_algorithm`, `..utils`, `.mm_common`, `torch._inductor.ir`, `torch._inductor.codegen.nv_universal_gemm.nv_universal_gemm`
