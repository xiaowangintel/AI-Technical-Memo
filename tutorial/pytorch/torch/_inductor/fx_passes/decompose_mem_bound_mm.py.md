# decompose_mem_bound_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/decompose_mem_bound_mm.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `check_device`, `realize_inputs`, `should_decompose_bmm`, `should_decompose_mm`, `print_decompose_pattern`, `decompose_bmm`, and `...+2`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `check_device`、`realize_inputs`、`should_decompose_bmm`、`should_decompose_mm`、`print_decompose_pattern`、`decompose_bmm`、`另有2项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
import logging

import torch
from torch import Tensor
from torch._dynamo.utils import counters, is_node_meta_valid
from torch.fx.experimental.symbolic_shapes import (
    statically_known_false,
    statically_known_true,
)

from .. import config
from ..pattern_matcher import Arg, CallFunction, Match, register_graph_pattern
from .split_cat import construct_pattern_matcher_pass


aten = torch.ops.aten
log = logging.getLogger(__name__)

# TODO: need a better strategy for decomposing mm
````
- **EN**: Imports dependencies such as `logging`, `torch`, `torch._dynamo.utils`, `torch.fx.experimental.symbolic_shapes`, `..`, `..pattern_matcher`, and `...+1` for the logic in this range. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `aten`, and `log`.
- **CN**: 这里导入了 `logging`、`torch`、`torch._dynamo.utils`、`torch.fx.experimental.symbolic_shapes`、`..`、`..pattern_matcher`、`另有1项` 等依赖，为后续逻辑提供基础能力。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `aten`、`log` 等值。

### Lines 21-40 / 第 21-40 行
````python
# The following two constants are for CUDA device only
MIN_FIRST_DIMENSION_DECOMPOSITION = 10240
MAX_OTHER_DIMENSION_DECOMPOSITION = 32
# The following two constants are for CPU device only
CPU_MAX_FIRST_DIMENSION_DECOMPOSITION = 1
CPU_MAX_OTHER_DIMENSION_DECOMPOSITION = 2048

min_first_dimension_decomposition = MIN_FIRST_DIMENSION_DECOMPOSITION
max_other_dimension_decomposition = MAX_OTHER_DIMENSION_DECOMPOSITION
cpu_max_first_dimension_decomposition = CPU_MAX_FIRST_DIMENSION_DECOMPOSITION
cpu_max_other_dimension_decomposition = CPU_MAX_OTHER_DIMENSION_DECOMPOSITION
if "decompose_mm_pass" in config.post_grad_fusion_options:
    min_first_dimension_decomposition = config.post_grad_fusion_options[
        "decompose_mm_pass"
    ].get("min_first_dimension_decomposition", MIN_FIRST_DIMENSION_DECOMPOSITION)
    max_other_dimension_decomposition = config.post_grad_fusion_options[
        "decompose_mm_pass"
    ].get("max_other_dimension_decomposition", MAX_OTHER_DIMENSION_DECOMPOSITION)
    cpu_max_first_dimension_decomposition = config.post_grad_fusion_options[
        "decompose_mm_pass"
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `MIN_FIRST_DIMENSION_DECOMPOSITION`, `MAX_OTHER_DIMENSION_DECOMPOSITION`, `CPU_MAX_FIRST_DIMENSION_DECOMPOSITION`, `CPU_MAX_OTHER_DIMENSION_DECOMPOSITION`, `min_first_dimension_decomposition`, `max_other_dimension_decomposition`, and `...+2`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `MIN_FIRST_DIMENSION_DECOMPOSITION`、`MAX_OTHER_DIMENSION_DECOMPOSITION`、`CPU_MAX_FIRST_DIMENSION_DECOMPOSITION`、`CPU_MAX_OTHER_DIMENSION_DECOMPOSITION`、`min_first_dimension_decomposition`、`max_other_dimension_decomposition`、`另有2项` 等值。

### Lines 41-60 / 第 41-60 行
````python
    ].get(
        "cpu_max_first_dimension_decomposition", CPU_MAX_FIRST_DIMENSION_DECOMPOSITION
    )
    cpu_max_other_dimension_decomposition = config.post_grad_fusion_options[
        "decompose_mm_pass"
    ].get(
        "cpu_max_other_dimension_decomposition", CPU_MAX_OTHER_DIMENSION_DECOMPOSITION
    )


def check_device(a: Tensor, b: Tensor, device="cuda") -> bool:
    return (a.device.type == b.device.type) and (b.device.type == device)


def realize_inputs(inputs: list[torch.fx.Node]):
    for inp in inputs:
        if isinstance(inp, torch.fx.node.Node):
            inp.meta["inductor_realize_to_strides"] = True


````
- **EN**: Introduces function `check_device`, function `realize_inputs`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`check_device`、函数`realize_inputs`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 61-80 / 第 61-80 行
````python
def should_decompose_bmm(mat1, mat2) -> bool:
    if is_node_meta_valid(mat1) and is_node_meta_valid(mat2):
        mat1 = mat1.meta["val"]
        mat2 = mat2.meta["val"]
    else:
        return False
    if len(mat1.shape) != 3 or len(mat2.shape) != 3:
        return False
    if check_device(mat1, mat2, device="cuda") or check_device(
        mat1, mat2, device="xpu"
    ):
        if mat1.shape[0] < min_first_dimension_decomposition:
            return False
        # 2 of m, n, k must be <= MAX_OTHER_DIMENSION_DECOMPOSITION
        # use bool() to deal with BooleanAtom type
        if (
            bool(mat1.shape[1] < max_other_dimension_decomposition)
            + bool(mat1.shape[2] < max_other_dimension_decomposition)
            + bool(mat2.shape[2] < max_other_dimension_decomposition)
            < 2
````
- **EN**: Introduces function `should_decompose_bmm`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`should_decompose_bmm`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 81-100 / 第 81-100 行
````python
        ):
            return False
        return True
    elif check_device(mat1, mat2, device="cpu"):
        if (
            mat1.shape[0] <= cpu_max_first_dimension_decomposition
            and mat2.shape[0] <= cpu_max_first_dimension_decomposition
        ):
            return True
    return False


def should_decompose_mm(mat1, mat2) -> bool:
    """
    Determines whether matrix multiplication (mm) should be decomposed into pointwise operations
    based on the input matrices' metadata, shapes, device placement, and configuration options.
    Args:
        mat1: The first matrix operand. Expected to be an object with a `.meta` attribute containing
              a "val" key, or a tensor-like object with a `.shape` attribute.
        mat2: The second matrix operand. Same requirements as `mat1`.
````
- **EN**: Introduces function `should_decompose_mm`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`should_decompose_mm`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 101-120 / 第 101-120 行
````python
    Returns:
        bool: True if the matrix multiplication should be decomposed according to the following logic:
            - Both inputs must have valid node metadata.
            - Both matrices must be 2-dimensional.
            - If the configuration option `skip_dynamic_shape_dim_check` is False:
                - Decomposition is only considered for statically-shaped matrices.
                - For CUDA devices: `mat1.shape[0]` must be at least `min_first_dimension_decomposition`,
                  and both dimensions of `mat2` must be less than `max_other_dimension_decomposition`.
                - For CPU devices: All relevant dimensions must be less than or equal to their respective
                  CPU decomposition thresholds.
            - If `skip_dynamic_shape_dim_check` is True:
                - Decomposition is considered for dynamic shapes as well, using a combination of
                  `statically_known_true` and `statically_known_false` checks to handle uncertainty.
                - The same dimension and device checks apply, but allow for dynamic/static uncertainty.
            - Returns False if any of the above conditions are not met.
    Notes:
        - Relies on helper functions such as `is_node_meta_valid`, `check_device`, `statically_known_true`,
          and `statically_known_false`, as well as configuration values like
          `min_first_dimension_decomposition`, `max_other_dimension_decomposition`, etc.
        - Designed for use in graph optimization or fusion passes where decomposing large or dynamic
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Initializes or updates values such as `Returns`, `bool`, and `Notes`. This range continues the implementation of function `should_decompose_mm`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。初始化或更新了 `Returns`、`bool`、`Notes` 等值。这一段延续了函数`should_decompose_mm` 的具体实现。

### Lines 121-140 / 第 121-140 行
````python
          matrix multiplications can improve performance or memory usage.
    """
    if is_node_meta_valid(mat1) and is_node_meta_valid(mat2):
        mat1 = mat1.meta["val"]
        mat2 = mat2.meta["val"]
    else:
        return False
    if len(mat1.shape) != 2 or len(mat2.shape) != 2:
        return False
    # case 1: we skip decompose mm if the input is dynamic shape
    if not config.post_grad_fusion_options["decompose_mm_pass"].get(
        "skip_dynamic_shape_dim_check", False
    ):
        return (
            (
                check_device(mat1, mat2, device="cuda")
                or check_device(mat1, mat2, device="xpu")
            )
            and statically_known_true(
                mat1.shape[0] >= min_first_dimension_decomposition
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mat1`, `mat2`, and `else`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `mat1`、`mat2`、`else` 等值。

### Lines 141-160 / 第 141-160 行
````python
            )
            and statically_known_true(mat2.shape[0] < max_other_dimension_decomposition)
            and statically_known_true(mat2.shape[1] < max_other_dimension_decomposition)
        ) or (
            check_device(mat1, mat2, device="cpu")
            and statically_known_true(
                mat1.shape[0] <= cpu_max_first_dimension_decomposition
            )
            and statically_known_true(
                mat2.shape[0] <= cpu_max_other_dimension_decomposition
            )
            and statically_known_true(
                mat2.shape[1] <= cpu_max_other_dimension_decomposition
            )
        )
    # case 2: we decompose mm if the input is dynamic shape
    else:
        return (
            (
                check_device(mat1, mat2, device="cuda")
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `else` 等值。

### Lines 161-180 / 第 161-180 行
````python
                or check_device(mat1, mat2, device="xpu")
            )
            and (
                statically_known_true(
                    mat1.shape[0] >= min_first_dimension_decomposition
                )
                or not statically_known_false(
                    mat1.shape[0] >= min_first_dimension_decomposition
                )
            )
            and (
                statically_known_true(mat2.shape[0] < max_other_dimension_decomposition)
                or not statically_known_false(
                    mat2.shape[0] < max_other_dimension_decomposition
                )
            )
            and (
                statically_known_true(mat2.shape[1] < max_other_dimension_decomposition)
                or not statically_known_false(
                    mat2.shape[1] < max_other_dimension_decomposition
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. This range continues the implementation of function `should_decompose_mm`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。这一段延续了函数`should_decompose_mm` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
                )
            )
        ) or (
            check_device(mat1, mat2, device="cpu")
            and (
                statically_known_true(
                    mat1.shape[0] <= cpu_max_first_dimension_decomposition
                )
                or not statically_known_false(
                    mat1.shape[0] <= cpu_max_first_dimension_decomposition
                )
            )
            and (
                statically_known_true(
                    mat2.shape[0] <= cpu_max_other_dimension_decomposition
                )
                or not statically_known_false(
                    mat2.shape[0] <= cpu_max_other_dimension_decomposition
                )
            )
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. This range continues the implementation of function `should_decompose_mm`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。这一段延续了函数`should_decompose_mm` 的具体实现。

### Lines 201-220 / 第 201-220 行
````python
            and (
                statically_known_true(
                    mat2.shape[1] <= cpu_max_other_dimension_decomposition
                )
                or not statically_known_false(
                    mat2.shape[1] <= cpu_max_other_dimension_decomposition
                )
            )
        )


def print_decompose_pattern(match: Match, inputs: list[torch.fx.Node]):
    node = match.nodes[-1]
    log.debug(
        "Decompose %s with input shape: %s",
        node.target,
        ", ".join(
            str(input.meta["val"].shape) if "val" in input.meta else "None"
            for input in inputs
        ),
````
- **EN**: Introduces function `print_decompose_pattern`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `node`.
- **CN**: 这里定义了函数`print_decompose_pattern`。包含分支、循环或上下文管理等控制流。初始化或更新了 `node` 等值。

### Lines 221-240 / 第 221-240 行
````python
    )


@register_graph_pattern(
    CallFunction(aten.bmm, Arg(), Arg()),
    pass_dict=construct_pattern_matcher_pass("decompose_mm_pass"),
)
def decompose_bmm(match: Match, mat1: torch.fx.Node, mat2: torch.fx.Node):
    def repl(mat1, mat2):
        return torch.sum(mat1[:, :, :, None] * mat2[:, None, :, :], dim=-2).to(
            mat1.dtype
        )

    if should_decompose_bmm(mat1, mat2):
        counters["inductor"]["decompose_bmm"] += 1
        # pyrefly: ignore [bad-argument-type]
        match.replace_by_example(repl, [mat1, mat2])
        print_decompose_pattern(match, [mat1, mat2])
        realize_inputs([mat1, mat2])
    return
````
- **EN**: Introduces function `decompose_bmm`, function `repl`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`decompose_bmm`、函数`repl`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 241-260 / 第 241-260 行
````python


@register_graph_pattern(
    CallFunction(aten.addmm, Arg(), Arg(), Arg()),
    pass_dict=construct_pattern_matcher_pass("decompose_mm_pass"),
)
def decompose_addmm(
    match: Match,
    mat1: torch.fx.Node,
    mat2: torch.fx.Node,
    mat3: torch.fx.Node,
):
    def repl(mat1, mat2, mat3):
        return (
            torch.sum(mat2[:, :, None] * mat3[None, :, :], dim=-2).to(mat2.dtype) + mat1
        )

    if should_decompose_mm(mat2, mat3):
        counters["inductor"]["decompose_addmm"] += 1
        # pyrefly: ignore [bad-argument-type]
````
- **EN**: Introduces function `decompose_addmm`, function `repl`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`decompose_addmm`、函数`repl`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 261-280 / 第 261-280 行
````python
        match.replace_by_example(repl, [mat1, mat2, mat3])
        print_decompose_pattern(match, [mat1, mat2, mat3])
        realize_inputs([mat1, mat2, mat3])
    return


@register_graph_pattern(
    CallFunction(aten.mm, Arg(), Arg()),
    pass_dict=construct_pattern_matcher_pass("decompose_mm_pass"),
)
def decompose_mm(
    match: Match,
    mat1: torch.fx.Node,
    mat2: torch.fx.Node,
):
    def repl(mat1, mat2):
        return torch.sum(mat1[:, :, None] * mat2[None, :, :], dim=-2).to(mat1.dtype)

    if should_decompose_mm(mat1, mat2):
        counters["inductor"]["decompose_mm"] += 1
````
- **EN**: Introduces function `decompose_mm`, function `repl`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`decompose_mm`、函数`repl`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 281-285 / 第 281-285 行
````python
        # pyrefly: ignore [bad-argument-type]
        match.replace_by_example(repl, [mat1, mat2])
        print_decompose_pattern(match, [mat1, mat2])
        realize_inputs([mat1, mat2])
    return
````
- **EN**: Includes returns or checks that define the contract of this code path. This range continues the implementation of function `decompose_mm`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`decompose_mm` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `check_device`, `realize_inputs`, `should_decompose_bmm`, `should_decompose_mm`, `print_decompose_pattern`, `decompose_bmm`, and `...+2`  
  **CN**: 主要函数：`check_device`、`realize_inputs`、`should_decompose_bmm`、`should_decompose_mm`、`print_decompose_pattern`、`decompose_bmm`、`另有2项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._dynamo.utils`, `torch.fx.experimental.symbolic_shapes`, `..`, `..pattern_matcher`, `.split_cat`
