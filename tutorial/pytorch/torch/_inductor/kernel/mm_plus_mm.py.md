# mm_plus_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel/mm_plus_mm.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It exposes functions such as `tuned_mm_plus_mm`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。同时提供 `tuned_mm_plus_mm` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
# mypy: allow-untyped-defs

import logging
from typing import TYPE_CHECKING

import torch

from .. import config as inductor_config
from ..kernel_inputs import MMKernelInputs
from ..lowering import lowerings
from ..select_algorithm import (
    autotune_select_algorithm,
    ExternKernelChoice,
    TritonTemplate,
````
- **EN**: Imports dependencies such as `logging`, `typing`, `torch`, `..`, `..kernel_inputs`, `..lowering`, and `...+1` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis.
- **CN**: 这里导入了 `logging`、`typing`、`torch`、`..`、`..kernel_inputs`、`..lowering`、`另有1项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。

### Lines 15-28 / 第 15-28 行
````python
)
from ..utils import use_aten_gemm_kernels, use_triton_template
from ..virtualized import V
from .mm_common import mm_args, mm_grid


if TYPE_CHECKING:
    from torch._inductor.ir import ChoiceCaller
    from torch._inductor.select_algorithm import KernelTemplate

log = logging.getLogger(__name__)

aten = torch.ops.aten

````
- **EN**: Imports dependencies such as `..utils`, `..virtualized`, `.mm_common`, `torch._inductor.ir`, and `torch._inductor.select_algorithm` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `log`, and `aten`.
- **CN**: 这里导入了 `..utils`、`..virtualized`、`.mm_common`、`torch._inductor.ir`、`torch._inductor.select_algorithm` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。包含分支、循环或上下文管理等控制流。初始化或更新了 `log`、`aten` 等值。

### Lines 29-42 / 第 29-42 行
````python
aten_mm_plus_mm = ExternKernelChoice(
    torch.ops.inductor._mm_plus_mm, "torch::inductor::_mm_plus_mm"
)

mm_plus_mm_template = TritonTemplate(
    name="mm_plus_mm",
    grid=mm_grid,
    debug=False,
    source=r"""
{{def_kernel("A", "B", "C", "D")}}
    M = {{size("A", 0)}}
    N = {{size("B", 1)}}
    K1 = {{size("A", 1)}}
    if M * N == 0:
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `aten_mm_plus_mm`, `mm_plus_mm_template`, `name`, `grid`, `debug`, `source`, and `...+3`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `aten_mm_plus_mm`、`mm_plus_mm_template`、`name`、`grid`、`debug`、`source`、`另有3项` 等值。

### Lines 43-56 / 第 43-56 行
````python
        # early exit due to zero-size input(s)
        return
    # K2 = {{size("C", 1)}}
    stride_am = {{stride("A", 0)}}
    stride_ak = {{stride("A", 1)}}
    stride_bk = {{stride("B", 0)}}
    stride_bn = {{stride("B", 1)}}
    stride_cm = {{stride("C", 0)}}
    stride_ck = {{stride("C", 1)}}
    stride_dk = {{stride("D", 0)}}
    stride_dn = {{stride("D", 1)}}

    # based on triton.ops.matmul
    pid = tl.program_id(0).to(INDEX_DTYPE)
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `stride_am`, `stride_ak`, `stride_bk`, `stride_bn`, `stride_cm`, `stride_ck`, and `...+3`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `stride_am`、`stride_ak`、`stride_bk`、`stride_bn`、`stride_cm`、`stride_ck`、`另有3项` 等值。

### Lines 57-70 / 第 57-70 行
````python
    grid_m = (M + BLOCK_M - 1) // BLOCK_M
    grid_n = (N + BLOCK_N - 1) // BLOCK_N

    # re-order program ID for better L2 performance
    width = GROUP_M * grid_n
    group_id = pid // width
    group_size = min(grid_m - group_id * GROUP_M, GROUP_M)
    pid_m = group_id * GROUP_M + (pid % group_size)
    pid_n = (pid % width) // (group_size)
    tl.assume(pid_m >= 0)
    tl.assume(pid_n >= 0)

    rm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
    rn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
````
- **EN**: Initializes or updates values such as `grid_m`, `grid_n`, `width`, `group_id`, `group_size`, `pid_m`, and `...+3`.
- **CN**: 初始化或更新了 `grid_m`、`grid_n`、`width`、`group_id`、`group_size`、`pid_m`、`另有3项` 等值。

### Lines 71-84 / 第 71-84 行
````python

    if (((stride_am == 1 and stride_ak == M) or (stride_am == K1 and stride_ak == 1))
        and ((stride_cm == 1 and stride_ck == M) or (stride_cm == K1 and stride_ck == 1))):
        ram = tl.max_contiguous(tl.multiple_of(rm % M, BLOCK_M), BLOCK_M)
    else:
        ram = rm % M

    if (((stride_bk == 1 and stride_bn == K1) or (stride_bk == N and stride_bn == 1))
        and ((stride_dk == 1 and stride_dn == K1) or (stride_dk == N and stride_dn == 1))):
        rbn = tl.max_contiguous(tl.multiple_of(rn % N, BLOCK_N), BLOCK_N)
    else:
        rbn = rn % N

    rk = tl.arange(0, BLOCK_K)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `ram`, `else`, `rbn`, and `rk`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `ram`、`else`、`rbn`、`rk` 等值。

### Lines 85-98 / 第 85-98 行
````python
    A = A + (ram[:, None] * stride_am + rk[None, :] * stride_ak)
    B = B + (rk[:, None] * stride_bk + rbn[None, :] * stride_bn)
    C = C + (ram[:, None] * stride_cm + rk[None, :] * stride_ck)
    D = D + (rk[:, None] * stride_dk + rbn[None, :] * stride_dn)

    acc = tl.zeros((BLOCK_M, BLOCK_N), dtype=ACC_TYPE)
    for k1 in range(K1, 0, -BLOCK_K):
        # First matmul with A @ B
        if EVEN_K:
            a = tl.load(A)
            b = tl.load(B)
        else:
            a = tl.load(A, mask=rk[None, :] < k1, other=0.)
            b = tl.load(B, mask=rk[:, None] < k1, other=0.)
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `A`, `B`, `C`, `D`, `acc`, `a`, and `...+2`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `A`、`B`、`C`、`D`、`acc`、`a`、`另有2项` 等值。

### Lines 99-112 / 第 99-112 行
````python
        acc += tl.dot(a, b, allow_tf32=ALLOW_TF32)
        A += BLOCK_K * stride_ak
        B += BLOCK_K * stride_bk

    for k2 in range(K1, 0, -BLOCK_K):

        # Second matmul with C @ D
        if EVEN_K:
            c = tl.load(C)
            d = tl.load(D)
        else:
            c = tl.load(C, mask=rk[None, :] < k2, other=0.)
            d = tl.load(D, mask=rk[:, None] < k2, other=0.)
        acc += tl.dot(c, d, allow_tf32=ALLOW_TF32)
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `c`, `d`, and `else`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `c`、`d`、`else` 等值。

### Lines 113-126 / 第 113-126 行
````python
        C += BLOCK_K * stride_ck
        D += BLOCK_K * stride_dk


    idx_m = rm[:, None]
    idx_n = rn[None, :]
    mask = (idx_m < M) & (idx_n < N)

    # inductor generates a suffix
    {{store_output(("idx_m", "idx_n"), "acc", "mask", val_shape=("BLOCK_M", "BLOCK_N"))}}
""",
    cache_codegen_enabled_for_template=True,
)

````
- **EN**: Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Initializes or updates values such as `idx_m`, `idx_n`, `mask`, and `cache_codegen_enabled_for_template`.
- **CN**: 这一段处理缓存状态或缓存路径，以避免重复构建相同工作。初始化或更新了 `idx_m`、`idx_n`、`mask`、`cache_codegen_enabled_for_template` 等值。

### Lines 127-140 / 第 127-140 行
````python

def tuned_mm_plus_mm(mat1, mat2, mat3, mat4, *, layout=None):
    """
    Computes mm(mat1, mat2) + mm(mat3, mat4)
    """
    # TODO(coconutruben): integrate into MMKernelInputs when all callsites use that
    m1, n1, k1, layout1, mat1, mat2 = mm_args(mat1, mat2, layout=layout)
    m2, n2, _, layout2, mat3, mat4 = mm_args(mat3, mat4, layout=layout)

    # Optimization is optional, because we can always just not do the fusion
    if (
        m1 * n1 == 0
        or m2 * n2 == 0
        or not V.graph.sizevars.statically_known_list_equals(
````
- **EN**: Introduces function `tuned_mm_plus_mm`. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`tuned_mm_plus_mm`。包含分支、循环或上下文管理等控制流。

### Lines 141-154 / 第 141-154 行
````python
            mat1.get_size(), mat3.get_size()
        )
        or not V.graph.sizevars.statically_known_list_equals(
            mat2.get_size(), mat4.get_size()
        )
        or inductor_config.triton.native_matmul
    ):
        # TODO(jansel): support different K values when this is fixed:
        # https://github.com/triton-lang/triton/issues/967
        return lowerings[aten.add](
            lowerings[aten.mm](mat1, mat2), lowerings[aten.mm](mat3, mat4)
        )

    # Create MMKernelInputs for MM Plus MM (matrices are at indices 0, 1 for first pair)
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. This range continues the implementation of function `tuned_mm_plus_mm`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`tuned_mm_plus_mm` 的具体实现。

### Lines 155-168 / 第 155-168 行
````python
    # Note: This is a special case with 4 matrices, but we use the first pair for M, N, K extraction
    kernel_inputs = MMKernelInputs([mat1, mat2, mat3, mat4], mat1_idx=0, mat2_idx=1)

    assert layout1 == layout2
    # options to tune from
    choices: list[ChoiceCaller] = []

    # Collect all templates for unified call
    templates_to_use: list[ExternKernelChoice | KernelTemplate] = []
    if use_aten_gemm_kernels():
        templates_to_use.append(aten_mm_plus_mm)

    if use_triton_template(layout1, check_max_autotune=False):
        templates_to_use.append(mm_plus_mm_template)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_inputs`, `choices`, and `templates_to_use`. This range continues the implementation of function `tuned_mm_plus_mm`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_inputs`、`choices`、`templates_to_use` 等值。这一段延续了函数`tuned_mm_plus_mm` 的具体实现。

### Lines 169-178 / 第 169-178 行
````python

    # Single unified call for all templates
    choices.extend(
        V.choices.get_template_configs(kernel_inputs, templates_to_use, "mm_plus_mm")
    )

    node, _ = autotune_select_algorithm(
        "mm_plus_mm", choices, kernel_inputs.nodes(), layout1
    )
    return node
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `tuned_mm_plus_mm`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`tuned_mm_plus_mm` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `tuned_mm_plus_mm`  
  **CN**: 主要函数：`tuned_mm_plus_mm`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `..`, `..kernel_inputs`, `..lowering`, `..select_algorithm`, `..utils`, `..virtualized`, `.mm_common`, `torch._inductor.ir`, `torch._inductor.select_algorithm`
