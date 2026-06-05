# triton.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/template_heuristics/triton.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `BaseConfig`, `GemmConfig`, `DepthwiseConvConfig`, `BlackwellGPUGemmConfig`, `FlexConfig`, `FlexBwDConfig`, and `...+61`. It exposes functions such as `get_shared_memory_checker_opts`.
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `BaseConfig`、`GemmConfig`、`DepthwiseConvConfig`、`BlackwellGPUGemmConfig`、`FlexConfig`、`FlexBwDConfig`、`另有61项` 等类。同时提供 `get_shared_memory_checker_opts` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36 / 第 1-36 行
````python
from __future__ import annotations

import dataclasses
import itertools
import math
import os
from functools import partial
from threading import Lock
from typing import Any, TYPE_CHECKING

import sympy

import torch
from torch._inductor.template_heuristics.triton_addmm import AddMMConfigMixin
from torch.utils._ordered_set import OrderedSet
from torch.utils._sympy.functions import Mod
from torch.utils._triton import has_triton_stable_tma_api

from .. import config, config as inductor_config
from ..kernel.bmm import bmm_template
from ..kernel.mm import (
    blackwell_ws_persistent_device_tma_mm_template,
    get_scaling_options,
    get_tile_size,
    mm_template,
    persistent_mm_template,
    persistent_tma_mm_template,
    scaled_mm_device_tma_epilogue_scaling_template,
    scaled_mm_device_tma_main_loop_scaling_template,
)
from ..kernel.mm_plus_mm import mm_plus_mm_template
from ..kernel_inputs import KernelInputs, MMKernelInputs
from ..utils import (
    get_backend_num_stages,
    get_default_kpack,
    get_num_sms,
````
- **EN**: Imports dependencies such as `__future__`, `dataclasses`, `itertools`, `math`, `os`, `functools`, and `...+14` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `__future__`、`dataclasses`、`itertools`、`math`、`os`、`functools`、`另有14项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。借助符号表达式来推理索引、形状或代数性质。

### Lines 37-72 / 第 37-72 行
````python
    get_tma_workspace_arg,
    TMA_DESCRIPTOR_SIZE,
    using_b200,
)
from ..virtualized import V
from .gemm import GemmMaxAutotuneTemplateConfigHeuristics
from .registry import register_template_heuristic


if TYPE_CHECKING:
    from collections.abc import Callable, Generator

    from triton import Config as TritonConfig

else:
    from torch._inductor.runtime.triton_compat import Config as TritonConfig


# Gemm Configs
@dataclasses.dataclass
class BaseConfig:
    """
    Base Gemm configuration used for most backends (CPU, CUDA)
    """

    block_m: int
    block_n: int
    block_k: int
    num_stages: int
    num_warps: int
    hint_override: int | None = dataclasses.field(kw_only=True, default=None)


@dataclasses.dataclass
class GemmConfig(BaseConfig):
    """
````
- **EN**: Imports dependencies such as `..virtualized`, `.gemm`, `.registry`, `collections.abc`, `triton`, and `torch._inductor.runtime.triton_compat` for the logic in this range. Introduces class `BaseConfig`, class `GemmConfig`. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Applies decorators to register behavior or alter how the following definition is constructed.
- **CN**: 这里导入了 `..virtualized`、`.gemm`、`.registry`、`collections.abc`、`triton`、`torch._inductor.runtime.triton_compat` 等依赖，为后续逻辑提供基础能力。这里定义了类`BaseConfig`、类`GemmConfig`。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。使用装饰器来注册行为，或改变后续定义的构造方式。

### Lines 73-108 / 第 73-108 行
````python
    Gemm configuration used for most backends (CPU, CUDA)
    """

    group_m: int = dataclasses.field(kw_only=True, default=8)


ConvConfig = BaseConfig


@dataclasses.dataclass
class DepthwiseConvConfig:
    """
    Configuration for depthwise conv1d Triton template.
    Uses BLOCK_N x BLOCK_L x BLOCK_C tiling (channels-last NLC layout).
    Matches the hand-written NLC kernel from depthwise_conv1d_benchmark.py.
    """

    block_n: int
    block_l: int
    block_c: int
    num_stages: int
    num_warps: int


@dataclasses.dataclass
class BlackwellGPUGemmConfig(GemmConfig):
    """
    Gemm configuration used for templates with features explicitly
    targeting Nvidia Blackwell GPUs
    """

    # epilogue_subtile must be a power of 2 (1 means no subtiling)
    epilogue_subtile: int = dataclasses.field(kw_only=True, default=1)
    warp_specialize: bool = dataclasses.field(kw_only=True, default=True)
    flatten: bool = dataclasses.field(kw_only=True, default=True)

````
- **EN**: Introduces class `DepthwiseConvConfig`, class `BlackwellGPUGemmConfig`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里定义了类`DepthwiseConvConfig`、类`BlackwellGPUGemmConfig`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 109-144 / 第 109-144 行
````python

# FlexAttention Configs
@dataclasses.dataclass
class FlexConfig:
    """
    Base Config class for flex attention
    - FlexAttn forward and backward will use this. For flex decoding,
      please use FlexDecodingConfig.

    NOTE:
    For flex_attn bwd block_m and block_n are reused for block_m1, block_m2, block_n1, block_n2

    """

    block_m: int
    block_n: int
    num_stages: int
    num_warps: int


@dataclasses.dataclass
class FlexBwDConfig:
    """
    Base Config class for flex attention backward
    - FlexAttn backward will use this.

    Note: flex bwd configs

    Kernel Constraints:
      * BLOCK_N1 % BLOCK_M1 == 0
      * BLOCK_M2 % BLOCK_N2 == 0

    Pattern 1 - Symmetric Pairing (M, N, N, M):
    - Used in autotune configs
    - block_m1=M, block_n1=N, block_m2=N, block_n2=M
    - Only requires checking BLOCK_N % BLOCK_M == 0
````
- **EN**: Introduces class `FlexConfig`, class `FlexBwDConfig`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Initializes or updates values such as `NOTE`, `block_m`, `block_n`, `num_stages`, `num_warps`, and `Note`.
- **CN**: 这里定义了类`FlexConfig`、类`FlexBwDConfig`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。初始化或更新了 `NOTE`、`block_m`、`block_n`、`num_stages`、`num_warps`、`Note` 等值。

### Lines 145-180 / 第 145-180 行
````python
    - Second constraint (BLOCK_M2 % BLOCK_N2) automatically satisfied

    Pattern 2 - Independent Parameters (M1, N1, M2, N2):
    - Used in exhaustive search for maximum flexibility
    - All four parameters can be set independently
    - Requires checking both constraints

    """

    block_m1: int
    block_n1: int
    block_m2: int
    block_n2: int
    num_stages: int
    num_warps: int


@dataclasses.dataclass
class FlexDecodeConfig:
    """
    Config class for flex decoding
    """

    block_n: int
    num_stages: int
    num_warps: int


# ROCm classes
@dataclasses.dataclass
class ROCmGemmConfig(GemmConfig):
    """
    ROCm subclass for GEMMs, with AMD backend specific tuneable kernargs
    """

    matrix_instr_nonkdim: int = 16
````
- **EN**: Introduces class `FlexDecodeConfig`, class `ROCmGemmConfig`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Initializes or updates values such as `block_m1`, `block_n1`, `block_m2`, `block_n2`, `num_stages`, `num_warps`, and `...+2`.
- **CN**: 这里定义了类`FlexDecodeConfig`、类`ROCmGemmConfig`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。初始化或更新了 `block_m1`、`block_n1`、`block_m2`、`block_n2`、`num_stages`、`num_warps`、`另有2项` 等值。

### Lines 181-216 / 第 181-216 行
````python
    waves_per_eu: int = 0
    kpack: int = 1


@dataclasses.dataclass
class ROCmConvConfig(ConvConfig):
    """
    ROCm subclass for Conv, with AMD backend specific tuneable kernargs
    """

    matrix_instr_nonkdim: int = 16
    waves_per_eu: int = 0
    kpack: int = 1


@dataclasses.dataclass
class ROCmFlexConfig(FlexConfig):
    """
    ROCm subclass for FlexAttn, with AMD backend specific tuneable kernargs
    """

    matrix_instr_nonkdim: int = 0
    waves_per_eu: int = 0
    kpack: int = 1


@dataclasses.dataclass
class ROCmFlexBwDConfig(FlexBwDConfig):
    """
    ROCm subclass for FlexAttn backward, with AMD backend specific tuneable kernargs
    """

    matrix_instr_nonkdim: int = 0
    waves_per_eu: int = 0
    kpack: int = 1

````
- **EN**: Introduces class `ROCmConvConfig`, class `ROCmFlexConfig`, class `ROCmFlexBwDConfig`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Initializes or updates values such as `waves_per_eu`, `kpack`, and `matrix_instr_nonkdim`.
- **CN**: 这里定义了类`ROCmConvConfig`、类`ROCmFlexConfig`、类`ROCmFlexBwDConfig`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。初始化或更新了 `waves_per_eu`、`kpack`、`matrix_instr_nonkdim` 等值。

### Lines 217-252 / 第 217-252 行
````python

@dataclasses.dataclass
class ROCmFlexDecodeConfig(FlexDecodeConfig):
    """
    ROCm subclass for FlexDecode, with AMD backend specific tuneable kernargs
    """

    matrix_instr_nonkdim: int = 0
    waves_per_eu: int = 0
    kpack: int = 1


class BaseHeuristicSingleton(type):
    """
    Thread-safe implementation of single to be used in the config heuristic subclasses
    to ensure heavy __init__ calls are not repeatedly run
    """

    _instances: dict[type[Any], Any] = {}
    _lock: Lock = Lock()

    def __call__(
        cls: BaseHeuristicSingleton, *args: Any, **kwargs: Any
    ) -> BaseConfigHeuristic:
        with cls._lock:
            if cls not in cls._instances:
                instance = super().__call__()
                cls._instances[cls] = instance
            return cls._instances[cls]


class BaseConfigHeuristic(metaclass=BaseHeuristicSingleton):
    """
    Base class for mm_configs, device specific triton kernels config inherit from here
    """

````
- **EN**: Introduces class `ROCmFlexDecodeConfig`, class `BaseHeuristicSingleton`, function `__call__`, class `BaseConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里定义了类`ROCmFlexDecodeConfig`、类`BaseHeuristicSingleton`、函数`__call__`、类`BaseConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 253-288 / 第 253-288 行
````python
    def __init__(self) -> None:
        # Whether the heuristic is used for int8. Use this when the heuristic is int8 exclusive
        # but prefer the preprocess_mm_configs argument when it's used for both
        self.has_int8_tensor: bool = False
        # Whether to scale configs at all
        # TODO(coconutruben): remove this once mm_plus_mm and tests support scaling
        self.should_scale_configs: bool = True
        # List of dictionaries to store the kernel configs. Configs that evaluate to true
        # will be utilised on the target platform. The configs are as follows:
        # (BLOCK_M, BLOCK_N, BLOCK_K, num_stages, num_warps)
        self.mm_configs: list[BaseConfig] = [
            GemmConfig(32, 32, 16, 1, 2),
            GemmConfig(32, 32, 128, 2, 4),
            GemmConfig(32, 64, 32, 5, 8),
            GemmConfig(64, 32, 32, 5, 8),
            GemmConfig(64, 32, 128, 5, 4),
            GemmConfig(64, 64, 16, 2, 4),
            GemmConfig(64, 64, 32, 2, 4),
            GemmConfig(64, 64, 64, 3, 8),
            GemmConfig(64, 64, 128, 5, 4),
            GemmConfig(64, 128, 32, 3, 4),
            GemmConfig(64, 128, 32, 4, 8),
            GemmConfig(64, 128, 64, 3, 4),
            GemmConfig(64, 128, 128, 4, 4),
            GemmConfig(128, 64, 32, 3, 4),
            GemmConfig(128, 64, 32, 4, 8),
            GemmConfig(128, 128, 32, 2, 8),
            GemmConfig(128, 128, 32, 3, 4),
            GemmConfig(128, 128, 64, 3, 4),
            GemmConfig(128, 128, 64, 5, 8),
            GemmConfig(128, 128, 128, 4, 8),
        ]

        # Exhaustive search for mm configs
        self.exhaustive_configs: list[BaseConfig] = [
            GemmConfig(
````
- **EN**: Introduces function `__init__`. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`__init__`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 289-324 / 第 289-324 行
````python
                BLOCK_M, BLOCK_N, BLOCK_K, num_stages, num_warps, group_m=group_m
            )
            for BLOCK_M, BLOCK_N, BLOCK_K in itertools.product(
                [16, 32, 64, 128, 256], repeat=3
            )
            for num_stages in [1, 2, 3, 4, 5]
            for num_warps in [2, 4, 8]
            for group_m in [8]
        ]

        # these are only used in tuned_mm when AutoHeuristic is enabled
        # the idea is that when AutoHeuristic collects data to learn a heuristic, more configs are autotuned
        # when the learned heuristic is used, the learned heuristic reduces the number of configs down to 10
        # which saves compilation time (since less configs are autotuned) and potentially increase performance
        # because the learned heuristic might predict a config that is not part mm_configs
        self.extra_mm_configs: list[BaseConfig] = [
            GemmConfig(16, 32, 16, 3, 2),
            GemmConfig(16, 32, 32, 4, 2),
            GemmConfig(16, 32, 32, 5, 2),
            GemmConfig(64, 64, 128, 3, 4),
            GemmConfig(128, 64, 32, 2, 2),
            GemmConfig(128, 64, 64, 3, 8),
            GemmConfig(128, 64, 128, 4, 8),
            GemmConfig(128, 128, 32, 4, 4),
            GemmConfig(128, 128, 64, 3, 8),
            GemmConfig(128, 128, 64, 5, 4),
        ]

        self.int8_mm_configs: list[BaseConfig] = [
            GemmConfig(64, 64, 32, 2, 4),
            GemmConfig(64, 128, 32, 3, 4),
            GemmConfig(128, 64, 32, 3, 4),
            GemmConfig(64, 128, 32, 4, 8),
            GemmConfig(128, 64, 32, 4, 8),
            GemmConfig(64, 32, 32, 5, 8),
            GemmConfig(32, 64, 32, 5, 8),
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 包含分支、循环或上下文管理等控制流。这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 325-360 / 第 325-360 行
````python
            GemmConfig(128, 128, 32, 2, 8),
            GemmConfig(64, 64, 64, 3, 8),
            GemmConfig(128, 256, 128, 3, 8),
            GemmConfig(256, 128, 128, 3, 8),
        ]

        self.mixed_mm_configs: list[BaseConfig] = [
            GemmConfig(16, 128, 256, 3, 4),
            GemmConfig(16, 128, 256, 5, 8),
        ]

        self.persistent_mm_configs: list[BaseConfig] = [
            GemmConfig(128, 256, 64, 3, 8),
            GemmConfig(128, 128, 64, 3, 8),
            GemmConfig(128, 128, 128, 3, 8),
            GemmConfig(128, 128, 128, 3, 4),
            GemmConfig(128, 128, 64, 4, 8),
            GemmConfig(128, 128, 64, 5, 8),
            GemmConfig(256, 128, 64, 4, 8),
            GemmConfig(128, 128, 64, 5, 4),
        ]

        self.blackwell_persistent_mm_configs: list[BaseConfig] = [
            BlackwellGPUGemmConfig(
                128,
                256,
                64,
                4,
                8,
                epilogue_subtile=2,
                warp_specialize=True,
                flatten=True,
            ),
            BlackwellGPUGemmConfig(
                256,
                128,
````
- **EN**: Initializes or updates values such as `epilogue_subtile`, `warp_specialize`, and `flatten`. This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 初始化或更新了 `epilogue_subtile`、`warp_specialize`、`flatten` 等值。这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 361-396 / 第 361-396 行
````python
                64,
                3,
                8,
                epilogue_subtile=2,
                warp_specialize=True,
                flatten=True,
            ),
            BlackwellGPUGemmConfig(
                128,
                256,
                128,
                2,
                8,
                epilogue_subtile=2,
                warp_specialize=True,
                flatten=True,
            ),
            BlackwellGPUGemmConfig(
                128,
                256,
                64,
                3,
                8,
                epilogue_subtile=2,
                warp_specialize=True,
                flatten=True,
            ),
            BlackwellGPUGemmConfig(
                128,
                128,
                128,
                3,
                4,
                epilogue_subtile=2,
                warp_specialize=True,
                flatten=True,
````
- **EN**: Initializes or updates values such as `epilogue_subtile`, `warp_specialize`, and `flatten`. This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 初始化或更新了 `epilogue_subtile`、`warp_specialize`、`flatten` 等值。这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 397-432 / 第 397-432 行
````python
            ),
            BlackwellGPUGemmConfig(
                256,
                128,
                64,
                3,
                8,
                epilogue_subtile=2,
                warp_specialize=True,
                flatten=True,
            ),
            BlackwellGPUGemmConfig(
                128,
                128,
                128,
                3,
                8,
                epilogue_subtile=2,
                warp_specialize=True,
                flatten=True,
            ),
            # Include no-subtiling. Always required for testing.
            BlackwellGPUGemmConfig(
                256,
                128,
                64,
                3,
                8,
                epilogue_subtile=1,
                warp_specialize=True,
                flatten=True,
            ),
            BlackwellGPUGemmConfig(
                128,
                128,
                128,
````
- **EN**: Initializes or updates values such as `epilogue_subtile`, `warp_specialize`, and `flatten`. This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 初始化或更新了 `epilogue_subtile`、`warp_specialize`、`flatten` 等值。这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 433-468 / 第 433-468 行
````python
                3,
                8,
                epilogue_subtile=1,
                warp_specialize=True,
                flatten=True,
            ),
            # Include subtile=4. Always required for testing.
            BlackwellGPUGemmConfig(
                256,
                128,
                64,
                4,
                8,
                epilogue_subtile=4,
                warp_specialize=True,
                flatten=True,
            ),
            BlackwellGPUGemmConfig(
                128,
                128,
                128,
                4,
                8,
                epilogue_subtile=4,
                warp_specialize=True,
                flatten=True,
            ),
        ]

        self.blackwell_persistent_addmm_configs: list[BaseConfig] = [
            # Include each subtiling factor for testing.
            BlackwellGPUGemmConfig(
                256,
                128,
                64,
                2,
````
- **EN**: Initializes or updates values such as `epilogue_subtile`, `warp_specialize`, and `flatten`. This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 初始化或更新了 `epilogue_subtile`、`warp_specialize`、`flatten` 等值。这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 469-504 / 第 469-504 行
````python
                4,
                epilogue_subtile=2,
                warp_specialize=True,
                flatten=True,
            ),
            BlackwellGPUGemmConfig(
                256,
                128,
                64,
                2,
                4,
                epilogue_subtile=1,
                warp_specialize=True,
                flatten=True,
            ),
            BlackwellGPUGemmConfig(
                256,
                128,
                64,
                2,
                4,
                epilogue_subtile=4,
                warp_specialize=True,
                flatten=True,
            ),
        ]

        self.scaled_mm_configs: list[BaseConfig] = [
            GemmConfig(128, 256, 32, 3, 8),
            GemmConfig(256, 128, 32, 3, 8),
            GemmConfig(256, 64, 32, 4, 4),
            GemmConfig(64, 256, 32, 4, 4),
            GemmConfig(128, 128, 32, 4, 4),
            GemmConfig(128, 64, 32, 4, 4),
            GemmConfig(64, 128, 32, 4, 4),
            GemmConfig(128, 32, 32, 4, 4),
````
- **EN**: Initializes or updates values such as `epilogue_subtile`, `warp_specialize`, and `flatten`. This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 初始化或更新了 `epilogue_subtile`、`warp_specialize`、`flatten` 等值。这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 505-540 / 第 505-540 行
````python
            GemmConfig(64, 32, 32, 5, 2),
            GemmConfig(256, 128, 128, 3, 8),
            GemmConfig(256, 64, 128, 4, 4),
            GemmConfig(64, 256, 128, 4, 4),
            GemmConfig(128, 128, 128, 4, 4),
            GemmConfig(128, 64, 64, 4, 4),
            GemmConfig(64, 128, 64, 4, 4),
            GemmConfig(128, 32, 64, 4, 4),
            GemmConfig(64, 32, 64, 5, 2),
            GemmConfig(16, 32, 32, 2, 2),
            GemmConfig(16, 64, 32, 2, 2),
            GemmConfig(16, 128, 32, 2, 4),
            GemmConfig(16, 256, 32, 2, 4),
            GemmConfig(16, 32, 64, 2, 2),
            GemmConfig(16, 64, 64, 2, 2),
            GemmConfig(16, 128, 64, 2, 4),
            GemmConfig(16, 256, 64, 2, 4),
            GemmConfig(32, 32, 32, 2, 2),
            GemmConfig(32, 64, 32, 2, 2),
            GemmConfig(32, 128, 32, 2, 4),
            GemmConfig(32, 256, 32, 2, 4),
            GemmConfig(32, 32, 64, 2, 2),
            GemmConfig(32, 64, 64, 2, 2),
            GemmConfig(32, 128, 64, 2, 4),
            GemmConfig(32, 256, 64, 2, 4),
            GemmConfig(16, 32, 32, 3, 2),
            GemmConfig(16, 64, 32, 3, 2),
            GemmConfig(16, 128, 32, 3, 4),
            GemmConfig(16, 256, 32, 3, 4),
            GemmConfig(16, 32, 64, 3, 2),
            GemmConfig(16, 64, 64, 3, 2),
            GemmConfig(16, 128, 64, 3, 4),
            GemmConfig(16, 256, 64, 3, 4),
            GemmConfig(32, 32, 32, 3, 2),
            GemmConfig(32, 64, 32, 3, 2),
            GemmConfig(32, 128, 32, 3, 4),
````
- **EN**: This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 541-576 / 第 541-576 行
````python
            GemmConfig(32, 256, 32, 3, 4),
            GemmConfig(32, 32, 64, 3, 2),
            GemmConfig(32, 64, 64, 3, 2),
            GemmConfig(32, 128, 64, 3, 4),
            GemmConfig(32, 256, 64, 3, 4),
            GemmConfig(16, 32, 32, 4, 2),
            GemmConfig(16, 64, 32, 4, 2),
            GemmConfig(16, 128, 32, 4, 4),
            GemmConfig(16, 256, 32, 4, 4),
            GemmConfig(16, 32, 64, 4, 2),
            GemmConfig(16, 64, 64, 4, 2),
            GemmConfig(16, 128, 64, 4, 4),
            GemmConfig(16, 256, 64, 4, 4),
            GemmConfig(32, 32, 32, 4, 2),
            GemmConfig(32, 64, 32, 4, 2),
            GemmConfig(32, 128, 32, 4, 4),
            GemmConfig(32, 256, 32, 4, 4),
            GemmConfig(32, 32, 64, 4, 2),
            GemmConfig(32, 64, 64, 4, 2),
            GemmConfig(32, 128, 64, 4, 4),
            GemmConfig(32, 256, 64, 4, 4),
            GemmConfig(16, 32, 32, 5, 2),
            GemmConfig(16, 64, 32, 5, 2),
            GemmConfig(16, 128, 32, 5, 4),
            GemmConfig(16, 256, 32, 5, 4),
            GemmConfig(16, 32, 64, 5, 2),
            GemmConfig(16, 64, 64, 5, 2),
            GemmConfig(16, 128, 64, 5, 4),
            GemmConfig(16, 256, 64, 5, 4),
            GemmConfig(32, 32, 32, 5, 2),
            GemmConfig(32, 64, 32, 5, 2),
            GemmConfig(32, 128, 32, 5, 4),
            GemmConfig(32, 256, 32, 5, 4),
            GemmConfig(32, 32, 64, 5, 2),
            GemmConfig(32, 64, 64, 5, 2),
            GemmConfig(32, 128, 64, 5, 4),
````
- **EN**: This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 577-612 / 第 577-612 行
````python
            GemmConfig(32, 256, 64, 5, 4),
            GemmConfig(16, 32, 32, 6, 2),
            GemmConfig(16, 64, 32, 6, 2),
            GemmConfig(16, 128, 32, 6, 4),
            GemmConfig(16, 256, 32, 6, 4),
            GemmConfig(16, 32, 64, 6, 2),
            GemmConfig(16, 64, 64, 6, 2),
            GemmConfig(16, 128, 64, 6, 4),
            GemmConfig(16, 256, 64, 6, 4),
            GemmConfig(32, 32, 32, 6, 2),
            GemmConfig(32, 64, 32, 6, 2),
            GemmConfig(32, 128, 32, 6, 4),
            GemmConfig(32, 256, 32, 6, 4),
            GemmConfig(32, 32, 64, 6, 2),
            GemmConfig(32, 64, 64, 6, 2),
            GemmConfig(32, 128, 64, 6, 4),
            GemmConfig(32, 256, 64, 6, 4),
            GemmConfig(64, 16, 256, 5, 4),
            GemmConfig(64, 32, 256, 5, 4),
            GemmConfig(64, 128, 128, 2, 4),
            GemmConfig(64, 128, 128, 3, 4),
            GemmConfig(128, 128, 128, 2, 4),
            GemmConfig(128, 256, 128, 4, 8),
            GemmConfig(256, 128, 128, 2, 4),
            GemmConfig(256, 128, 128, 2, 8),
        ]

        self.scaled_persistent_mm_configs: list[BaseConfig] = [
            GemmConfig(128, 128, 64, 3, 8),
            GemmConfig(128, 128, 128, 3, 8),
            GemmConfig(128, 128, 128, 4, 8),
            GemmConfig(128, 128, 128, 4, 4),
            GemmConfig(128, 128, 128, 3, 4),
            GemmConfig(128, 128, 128, 5, 4),
            GemmConfig(128, 128, 128, 5, 8),
            GemmConfig(128, 128, 128, 6, 8),
````
- **EN**: This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 613-648 / 第 613-648 行
````python
            GemmConfig(128, 128, 64, 4, 8),
            GemmConfig(64, 32, 256, 5, 4),
            GemmConfig(128, 256, 128, 3, 8),
            GemmConfig(64, 128, 256, 4, 4),
            GemmConfig(64, 256, 128, 4, 4),
        ]

        self.blackwell_scaled_persistent_mm_configs = [
            BlackwellGPUGemmConfig(
                block_m=c.block_m,
                block_n=c.block_n,
                block_k=c.block_k,
                num_stages=c.num_stages,
                num_warps=c.num_warps,
                hint_override=c.hint_override,
                group_m=8,
                epilogue_subtile=2,
                warp_specialize=True,
                flatten=True,
            )
            for c in self.scaled_persistent_mm_configs
        ]

        # TODO: Unify with other gemm patterns, mm_plus_mm currently follows
        # slightly different pattern than rest
        self.mm_plus_mm_configs: list[BaseConfig] = [
            GemmConfig(64, 64, 32, 2, 4),
            GemmConfig(64, 64, 32, 3, 8),
            GemmConfig(64, 64, 32, 4, 16),
            GemmConfig(64, 32, 32, 4, 8),
            GemmConfig(32, 64, 32, 4, 8),
            GemmConfig(128, 128, 32, 1, 8),
            GemmConfig(64, 64, 64, 1, 8),
            GemmConfig(32, 32, 128, 1, 8),
            GemmConfig(64, 64, 16, 2, 4),
            GemmConfig(32, 32, 16, 1, 2),
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `block_m`, `block_n`, `block_k`, `num_stages`, `num_warps`, `hint_override`, and `...+4`. This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `block_m`、`block_n`、`block_k`、`num_stages`、`num_warps`、`hint_override`、`另有4项` 等值。这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 649-684 / 第 649-684 行
````python
        ]

        self.conv_configs: list[BaseConfig] = [
            # BLOCK_K=16 configs
            ConvConfig(64, 256, 16, 2, 4),
            ConvConfig(256, 64, 16, 2, 4),
            ConvConfig(1024, 16, 16, 1, 8),
            # BLOCK_K=32 configs
            ConvConfig(128, 128, 32, 2, 8),
            ConvConfig(64, 64, 32, 2, 4),
            ConvConfig(64, 256, 32, 2, 8),
            ConvConfig(256, 64, 32, 2, 8),
            # BLOCK_K=64 configs
            ConvConfig(128, 128, 64, 3, 8),
            ConvConfig(64, 128, 64, 4, 4),
            ConvConfig(128, 64, 64, 4, 4),
            ConvConfig(256, 128, 64, 2, 8),
            ConvConfig(128, 256, 64, 2, 8),
            # BLOCK_K=128 configs - optimal when IN_C=128 (single iteration over channels)
            ConvConfig(128, 128, 128, 2, 8),
            ConvConfig(128, 128, 128, 3, 8),
            ConvConfig(64, 128, 128, 4, 4),
            ConvConfig(256, 128, 128, 2, 8),
            ConvConfig(128, 256, 128, 2, 8),
        ]

        # Depthwise conv1d configs: BLOCK_N x BLOCK_L x BLOCK_C tiling
        # Derived from autotuning results on H100 for depthwise conv1d
        # channels-last (NLC) layout with shape x=[3072, 128, 202]
        # Matches _nlc_autotune_configs from depthwise_conv1d_benchmark.py
        self.depthwise_conv_configs: list[DepthwiseConvConfig] = [
            # BLOCK_C=32, BLOCK_L=32
            DepthwiseConvConfig(
                block_n=16, block_l=32, block_c=32, num_stages=4, num_warps=8
            ),
            DepthwiseConvConfig(
````
- **EN**: Initializes or updates values such as `block_n`. This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 初始化或更新了 `block_n` 等值。这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 685-720 / 第 685-720 行
````python
                block_n=16, block_l=32, block_c=32, num_stages=4, num_warps=4
            ),
            DepthwiseConvConfig(
                block_n=32, block_l=32, block_c=32, num_stages=5, num_warps=8
            ),
            DepthwiseConvConfig(
                block_n=32, block_l=32, block_c=32, num_stages=4, num_warps=4
            ),
            # BLOCK_C=32, BLOCK_L=64
            DepthwiseConvConfig(
                block_n=16, block_l=64, block_c=32, num_stages=4, num_warps=8
            ),
            DepthwiseConvConfig(
                block_n=16, block_l=64, block_c=32, num_stages=4, num_warps=4
            ),
            DepthwiseConvConfig(
                block_n=32, block_l=64, block_c=32, num_stages=3, num_warps=8
            ),
            # BLOCK_C=32, BLOCK_L=256
            DepthwiseConvConfig(
                block_n=16, block_l=256, block_c=32, num_stages=5, num_warps=8
            ),
            DepthwiseConvConfig(
                block_n=16, block_l=256, block_c=32, num_stages=4, num_warps=4
            ),
            DepthwiseConvConfig(
                block_n=32, block_l=256, block_c=32, num_stages=3, num_warps=8
            ),
            # BLOCK_C=64
            DepthwiseConvConfig(
                block_n=16, block_l=32, block_c=64, num_stages=4, num_warps=8
            ),
            DepthwiseConvConfig(
                block_n=16, block_l=32, block_c=64, num_stages=4, num_warps=4
            ),
            DepthwiseConvConfig(
````
- **EN**: Initializes or updates values such as `block_n`. This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 初始化或更新了 `block_n` 等值。这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 721-756 / 第 721-756 行
````python
                block_n=16, block_l=64, block_c=64, num_stages=3, num_warps=8
            ),
            # BLOCK_C=128
            DepthwiseConvConfig(
                block_n=16, block_l=32, block_c=128, num_stages=3, num_warps=8
            ),
            DepthwiseConvConfig(
                block_n=16, block_l=32, block_c=128, num_stages=3, num_warps=4
            ),
        ]

        self.flex_attn_fwd_autotune_configs: list[FlexConfig] = [
            FlexConfig(128, 64, 3, 4),
            FlexConfig(128, 128, 3, 4),
            FlexConfig(128, 128, 2, 8),
            FlexConfig(128, 128, 1, 8),
            FlexConfig(64, 128, 3, 4),
            FlexConfig(64, 64, 3, 4),
        ]

        self.flex_attn_bwd_autotune_configs: list[FlexBwDConfig] = [
            # See Note: flex bwd configs
            FlexBwDConfig(BLOCK_M, BLOCK_N, BLOCK_N, BLOCK_M, s, w)
            for BLOCK_M in [32, 64]
            for BLOCK_N in [32, 64, 128]
            for s in [1, 3, 4, 5]  # num_stages
            for w in ([4, 8] if BLOCK_M >= 128 or BLOCK_N >= 128 else [4])
            if BLOCK_N % BLOCK_M == 0
        ]

        self.flex_decode_autotune_configs: list[FlexDecodeConfig] = [
            FlexDecodeConfig(64, 3, 2),
            FlexDecodeConfig(32, 3, 2),
            FlexDecodeConfig(128, 3, 2),
        ]

````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `block_n`. This range continues the implementation of function `BaseConfigHeuristic.__init__`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `block_n` 等值。这一段延续了函数`BaseConfigHeuristic.__init__` 的具体实现。

### Lines 757-792 / 第 757-792 行
````python
        self.exhaustive_flex_attn_fwd_configs: list[FlexConfig] = [
            FlexConfig(BLOCK_M, BLOCK_N, num_stages, num_warps)
            for BLOCK_M in [16, 32, 64, 128]
            for BLOCK_N in [32, 64, 128]
            for num_stages in [1, 3, 4, 5]
            for num_warps in [2, 4, 8]
        ]

        self.exhaustive_flex_attn_bwd_configs: list[FlexBwDConfig] = [
            # See Note: flex bwd configs
            FlexBwDConfig(BLOCK_M1, BLOCK_N1, BLOCK_M2, BLOCK_N2, num_stages, num_warps)
            for BLOCK_M1 in [16, 32, 64, 128]
            for BLOCK_N1 in [16, 32, 64, 128]
            for BLOCK_M2 in [16, 32, 64, 128]
            for BLOCK_N2 in [16, 32, 64, 128]
            for num_stages in [1, 3, 4]
            for num_warps in [2, 4, 8]
            if BLOCK_N1 % BLOCK_M1 == 0
            and BLOCK_M2 % BLOCK_N2 == 0  # kernel static assertions
        ]

        self.exhaustive_flex_decode_configs: list[FlexDecodeConfig] = [
            FlexDecodeConfig(block_n, num_stages, num_warps)
            for block_n in [16, 32, 64, 128]
            for num_stages in [1, 3, 4, 5]
            for num_warps in [2, 4, 8]
        ]

    def _get_extra_config_key_and_kwargs(
        self, conf: BaseConfig
    ) -> tuple[tuple[int | None, ...], dict[str, Any]]:
        """Hook for subclasses to extend config dedup key and kwargs."""
        return (), {}

    def _finalize_mm_configs(
        self,
````
- **EN**: Introduces function `_get_extra_config_key_and_kwargs`, function `_finalize_mm_configs`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_get_extra_config_key_and_kwargs`、函数`_finalize_mm_configs`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 793-828 / 第 793-828 行
````python
        configs: list[BaseConfig],
    ) -> Generator[TritonConfig, None, None]:
        """
        Finalizes configs after scaling, applying additional constraints.
        """
        used: OrderedSet[tuple[int | None, ...]] = OrderedSet()

        max_mm_configs = config.test_configs.max_mm_configs

        for conf in configs:
            # Each warp computes a 16x16 tile = 256 elements
            num_warps = min(conf.num_warps, conf.block_m * conf.block_n // 256)

            # Construct key for finding duplicate configs
            key: tuple[int | None, ...] = (
                conf.block_m,
                conf.block_n,
                conf.block_k,
                conf.num_stages,
                conf.hint_override,
                num_warps,
            )

            # Check if gemm specific arg exists - add to key if does
            group_m = getattr(conf, "group_m", None)
            if group_m is not None:
                key += (group_m,)

            # Add BlackwellGPUGemmConfig specific fields to key if present
            if isinstance(conf, BlackwellGPUGemmConfig):
                key += (conf.epilogue_subtile, conf.warp_specialize, conf.flatten)

            extra_key, extra_kwargs = self._get_extra_config_key_and_kwargs(conf)
            key += extra_key

            if key not in used and (
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `configs`, `used`, `max_mm_configs`, `num_warps`, `key`, and `group_m`. This range continues the implementation of function `BaseConfigHeuristic._finalize_mm_configs`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `configs`、`used`、`max_mm_configs`、`num_warps`、`key`、`group_m` 等值。这一段延续了函数`BaseConfigHeuristic._finalize_mm_configs` 的具体实现。

### Lines 829-864 / 第 829-864 行
````python
                max_mm_configs is None or len(used) < max_mm_configs
            ):
                used.add(key)
                kwargs: dict[str, Any] = {
                    "BLOCK_M": conf.block_m,
                    "BLOCK_N": conf.block_n,
                    "BLOCK_K": conf.block_k,
                    "hint_override": conf.hint_override,
                }
                if group_m is not None:
                    kwargs["GROUP_M"] = group_m

                # Add BlackwellGPUGemmConfig specific fields if present
                if isinstance(conf, BlackwellGPUGemmConfig):
                    kwargs["EPILOGUE_SUBTILE"] = conf.epilogue_subtile
                    kwargs["WARP_SPECIALIZE"] = conf.warp_specialize
                    kwargs["FLATTEN"] = conf.flatten

                kwargs.update(extra_kwargs)

                yield self.triton_config(conf.num_stages, num_warps, **kwargs)

    def _scale_mm_configs(
        self,
        m: int,
        n: int,
        k: int,
        configs: list[BaseConfig],
        scale: float,
        has_int8_tensor: bool,
        exclude: Callable[[sympy.Integer, sympy.Integer, sympy.Integer], bool],
        hint_override: int | None = None,
    ) -> list[BaseConfig]:
        """
        Scales and filters matrix multiplication configs based on input size.
        """
````
- **EN**: Introduces function `_scale_mm_configs`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_scale_mm_configs`。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 865-900 / 第 865-900 行
````python
        if not self.should_scale_configs:
            return configs
        from ..runtime.runtime_utils import next_power_of_2

        min_block_size = 16
        min_block_size_k = 32 if (has_int8_tensor or self.has_int8_tensor) else 16

        scaled_configs = []
        for hint_override in [None] + config.multi_kernel_hints:
            m_hint = max(
                next_power_of_2(
                    V.graph.sizevars.optimization_hint_with_override(
                        m,
                        hint_override=hint_override,
                    )
                ),
                min_block_size,
            )
            n_hint = max(
                next_power_of_2(
                    V.graph.sizevars.optimization_hint_with_override(
                        n,
                        hint_override=hint_override,
                    )
                ),
                min_block_size,
            )
            k_hint = max(
                next_power_of_2(
                    V.graph.sizevars.optimization_hint_with_override(
                        k,
                        hint_override=hint_override,
                    )
                ),
                min_block_size_k,
            )
````
- **EN**: Imports dependencies such as `..runtime.runtime_utils` for the logic in this range. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `min_block_size`, `min_block_size_k`, `scaled_configs`, `m_hint`, `hint_override`, `n_hint`, and `...+1`.
- **CN**: 这里导入了 `..runtime.runtime_utils` 等依赖，为后续逻辑提供基础能力。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `min_block_size`、`min_block_size_k`、`scaled_configs`、`m_hint`、`hint_override`、`n_hint`、`另有1项` 等值。

### Lines 901-936 / 第 901-936 行
````python

            for c in configs:
                block_m = max(min(int(c.block_m * scale), m_hint), min_block_size)
                block_n = max(min(int(c.block_n * scale), n_hint), min_block_size)
                block_k = max(min(int(c.block_k * scale), k_hint), min_block_size_k)
                if not exclude(block_m, block_n, block_k):
                    # This copy is expensive, so avoid it if we can.
                    if (block_m, block_n, block_k, hint_override) != (
                        c.block_m,
                        c.block_n,
                        c.block_k,
                        c.hint_override,
                    ):
                        c = dataclasses.replace(
                            c,
                            block_m=block_m,
                            block_n=block_n,
                            block_k=block_k,
                            hint_override=hint_override,
                        )

                    scaled_configs.append(c)

        return scaled_configs

    # Estimate theoretical maximum shared memory
    def get_shared_memory_estimation(
        self,
        gemm_config: BaseConfig,
        dtype_size: int,
        has_sm_layout_conversion: bool,
        layout_conversion_byte_size: int,
    ):
        shared_mem_loads = dtype_size * (
            gemm_config.block_m * gemm_config.block_k
            + gemm_config.block_n * gemm_config.block_k
````
- **EN**: Introduces function `get_shared_memory_estimation`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `block_m`, `block_n`, `block_k`, `c`, `hint_override`, `gemm_config`, and `...+4`.
- **CN**: 这里定义了函数`get_shared_memory_estimation`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `block_m`、`block_n`、`block_k`、`c`、`hint_override`、`gemm_config`、`另有4项` 等值。

### Lines 937-972 / 第 937-972 行
````python
        )

        # Extra bytes to account for barriers in boundary conditions
        extra_bytes = 128

        # In persistent tma case, the layout conversion from mma -> blocked layout
        # is not free and takes additional shared memory, while next loads are prefetched
        # For addmm, the conversion is in the acc dtype, as it is needed before the bias addition
        # For mm, the conversion is in the output dtype, as it happens before the store
        if has_sm_layout_conversion:
            element_bits = layout_conversion_byte_size * 8
            # 8 bytes of padding for fp16/bf16
            max_padding = 128 // element_bits
            block_n = max_padding + gemm_config.block_n
            shared_mem_epilogue = (
                layout_conversion_byte_size * gemm_config.block_m * block_n
            )
        else:
            shared_mem_epilogue = 0

        return (
            shared_mem_loads * gemm_config.num_stages
            + shared_mem_epilogue
            + extra_bytes
        )

    def _get_exceeding_shared_memory_checker(
        self,
        has_sm_layout_conversion: bool,
        layout_conversion_byte_size: int,
    ) -> Callable[[BaseConfig, int], bool] | None:
        """
        Returns a function that checks whether a given configuration exceeds the available shared memory for the device.
        based on the config's theoretical maximum shared memory used.
        If the device does not report available shared memory, returns None.
        """
````
- **EN**: Introduces function `_get_exceeding_shared_memory_checker`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_get_exceeding_shared_memory_checker`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 973-1008 / 第 973-1008 行
````python

        try:
            device = torch.cuda.current_device()
            props = torch.cuda.get_device_properties(device)
            if hasattr(props, "shared_memory_per_block_optin"):  # for NVidia GPUs
                sm_available = int(props.shared_memory_per_block_optin)
            elif hasattr(props, "shared_memory_per_block"):  # for ROCm
                sm_available = int(props.shared_memory_per_block)
            else:
                return None

        except Exception:
            # If CUDA is not available or properties cannot be queried, return None
            return None

        # TODO make a BaseDeviceConfigHeuristics to handle different device configuration in its own implementation.
        def exceeds(gemm_config: BaseConfig, dtype_size: int) -> bool:
            estimation = self.get_shared_memory_estimation(
                gemm_config,
                dtype_size,
                has_sm_layout_conversion,
                layout_conversion_byte_size,
            )
            return estimation > sm_available

        return exceeds

    def _prune_exceeding_max_shared_mem_configs(
        self,
        configs: list[BaseConfig],
        dtype_size: int,
        has_sm_layout_conversion: bool = False,
        layout_conversion_byte_size: int = 0,
    ) -> list[BaseConfig]:
        if dtype_size <= 0:
            return configs
````
- **EN**: Introduces function `exceeds`, function `_prune_exceeding_max_shared_mem_configs`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`exceeds`、函数`_prune_exceeding_max_shared_mem_configs`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1009-1044 / 第 1009-1044 行
````python

        is_exceeding_shared_memory = self._get_exceeding_shared_memory_checker(
            has_sm_layout_conversion, layout_conversion_byte_size
        )
        if is_exceeding_shared_memory is None:
            return configs

        return [c for c in configs if not is_exceeding_shared_memory(c, dtype_size)]

    def _prune_reg_spill_configs(
        self,
        configs: list[BaseConfig],
    ) -> list[BaseConfig]:
        pruned_configs = []
        for gemm_config in configs:
            NUM_REG = 255
            acc_regs = math.ceil(
                gemm_config.block_m * gemm_config.block_n / (gemm_config.num_warps * 32)
            )
            # Lower bound for register spillage, if exceeds the kernel will certainly spill
            if acc_regs > NUM_REG:
                continue

            pruned_configs.append(gemm_config)

        return pruned_configs

    def _filter_configs(self, configs: list[BaseConfig]) -> list[BaseConfig]:
        """
        Filter configs based on specific requirements.
        Subclasses can override this to implement custom filtering logic.
        """
        return configs

    def preprocess_mm_configs(
        self,
````
- **EN**: Introduces function `_prune_reg_spill_configs`, function `_filter_configs`, function `preprocess_mm_configs`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_prune_reg_spill_configs`、函数`_filter_configs`、函数`preprocess_mm_configs`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1045-1080 / 第 1045-1080 行
````python
        m: int,
        n: int,
        k: int,
        configs: list[BaseConfig],
        has_int8_tensor: bool = False,
        scale: float = 1.0,
        exclude: Callable[
            [sympy.Integer, sympy.Integer, sympy.Integer], bool
        ] = lambda m, n, k: False,
        dtype_size: int = 0,
        op_name: str = "mm",  # For preprocessing overrides e.g. on CPU
        **kwargs,
    ) -> Generator[TritonConfig, None, None]:
        configs = self._filter_configs(configs)
        scaled_configs = self._scale_mm_configs(
            m, n, k, configs, scale, has_int8_tensor, exclude
        )

        # Filter out configs that require more shared memory than is available.
        # Theoretical upper bound, will over-prune configs. Off by default for maximum
        # performance
        if config.max_autotune_prune_choices_based_on_shared_mem:
            scaled_configs = self._prune_exceeding_max_shared_mem_configs(
                scaled_configs,
                dtype_size,
                kwargs.get("has_sm_layout_conversion", False),
                kwargs.get("layout_conversion_byte_size", 0),
            )

        if config.max_autotune_gemm_search_space == "EXHAUSTIVE":
            scaled_configs = self._prune_reg_spill_configs(scaled_configs)
        return self._finalize_mm_configs(scaled_configs)

    def triton_config(
        self, num_stages: int, num_warps: int, **kwargs: Any
    ) -> TritonConfig:
````
- **EN**: Introduces function `triton_config`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`triton_config`。借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1081-1116 / 第 1081-1116 行
````python
        return TritonConfig(kwargs, num_stages=num_stages, num_warps=num_warps)

    def get_mm_configs(self) -> partial[Generator[TritonConfig, None, None]]:
        return partial(self.preprocess_mm_configs, configs=self.mm_configs)

    def get_exhaustive_mm_configs(self) -> partial[Generator[TritonConfig, None, None]]:
        return partial(self.preprocess_mm_configs, configs=self.exhaustive_configs)

    def get_conv_configs(self) -> partial[Generator[TritonConfig, None, None]]:
        return partial(
            self.preprocess_mm_configs, configs=self.conv_configs, op_name="conv"
        )

    def get_depthwise_conv_configs(self) -> list[TritonConfig]:
        """Return TritonConfig list for depthwise conv1d autotuning."""
        return [
            TritonConfig(
                {
                    "BLOCK_N": cfg.block_n,
                    "BLOCK_L": cfg.block_l,
                    "BLOCK_C": cfg.block_c,
                },
                num_stages=cfg.num_stages,
                num_warps=cfg.num_warps,
            )
            for cfg in self.depthwise_conv_configs
        ]

    # Flex attn helpers
    def get_flex_attn_fwd_configs(self, head_dim: int, dtype: Any) -> list[FlexConfig]:
        flex_attn_fwd_configs: list[FlexConfig] = []

        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
                return self.exhaustive_flex_attn_fwd_configs
            flex_attn_fwd_configs += self.flex_attn_fwd_autotune_configs
````
- **EN**: Introduces function `get_mm_configs`, function `get_exhaustive_mm_configs`, function `get_conv_configs`, function `get_depthwise_conv_configs`, function `get_flex_attn_fwd_configs`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_mm_configs`、函数`get_exhaustive_mm_configs`、函数`get_conv_configs`、函数`get_depthwise_conv_configs`、函数`get_flex_attn_fwd_configs`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1117-1152 / 第 1117-1152 行
````python

        if head_dim <= 256:
            if dtype == torch.float32:
                default_config = FlexConfig(64, 64, 3, 4)
            else:
                default_config = FlexConfig(128, 64, 3, 4)
        else:
            if dtype == torch.float32:
                default_config = FlexConfig(32, 16, 3, 4)
            else:
                default_config = FlexConfig(64, 32, 3, 4)

        if default_config not in flex_attn_fwd_configs:
            flex_attn_fwd_configs.append(default_config)

        return flex_attn_fwd_configs

    def get_flex_attn_bwd_configs(
        self, head_dim: int, dtype: Any
    ) -> list[FlexBwDConfig]:
        flex_attn_bwd_configs: list[FlexBwDConfig] = []

        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
                return self.exhaustive_flex_attn_bwd_configs
            flex_attn_bwd_configs += self.flex_attn_bwd_autotune_configs

        default_config = FlexBwDConfig(16, 16, 16, 16, 1, 4)

        if default_config not in flex_attn_bwd_configs:
            flex_attn_bwd_configs.append(default_config)

        return flex_attn_bwd_configs

    def get_flex_decode_configs(
        self, head_dim: int, dtype: Any
````
- **EN**: Introduces function `get_flex_attn_bwd_configs`, function `get_flex_decode_configs`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_flex_attn_bwd_configs`、函数`get_flex_decode_configs`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1153-1188 / 第 1153-1188 行
````python
    ) -> list[FlexDecodeConfig]:
        flex_decode_configs: list[FlexDecodeConfig] = []

        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
                return self.exhaustive_flex_decode_configs
            flex_decode_configs += self.flex_decode_autotune_configs

        default_config = FlexDecodeConfig(block_n=64, num_stages=1, num_warps=2)

        if default_config not in flex_decode_configs:
            flex_decode_configs.append(default_config)

        return flex_decode_configs


class CPUConfigHeuristic(BaseConfigHeuristic):
    """
    CPU-specific config heuristic with CPU-specific optimizations.
    """

    def _get_cpu_exclude_function(
        self, method: str = "bmm"
    ) -> Callable[[sympy.Integer, sympy.Integer, sympy.Integer], bool]:
        """
        Get CPU-specific exclude function based on method type.
        Returns a function that can be used as exclude condition.
        Moved from mm_common._is_large_block_for_cpu and refactored to return a function.
        """
        if method in ("conv"):

            def exclude_conv(
                m: sympy.Integer, n: sympy.Integer, k: sympy.Integer
            ) -> bool:
                # Thresholds are experimentally determined to reduce Triton CPU compile times
                if m > 256 or n > 256 or k > 256:
````
- **EN**: Introduces class `CPUConfigHeuristic`, function `_get_cpu_exclude_function`, function `exclude_conv`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了类`CPUConfigHeuristic`、函数`_get_cpu_exclude_function`、函数`exclude_conv`。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 1189-1224 / 第 1189-1224 行
````python
                    return True
                return m * n * k > 2**17

            return exclude_conv
        elif method in ("mm", "addmm", "int_mm"):

            def exclude_mm(
                m: sympy.Integer, n: sympy.Integer, k: sympy.Integer
            ) -> bool:
                return m * n > 2**13

            return exclude_mm
        else:  # Default to bmm implementation for unknown methods

            def exclude_bmm(
                m: sympy.Integer, n: sympy.Integer, k: sympy.Integer
            ) -> bool:
                if m > 128 or n > 128 or k > 128:
                    return True
                return m * n > 2**12

            return exclude_bmm

    def preprocess_mm_configs(
        self,
        m: int,
        n: int,
        k: int,
        configs: list[BaseConfig],
        has_int8_tensor: bool = False,
        scale: float = 1.0,
        exclude: Callable[
            [sympy.Integer, sympy.Integer, sympy.Integer], bool
        ] = lambda m, n, k: False,
        dtype_size: int = 0,
        op_name: str = "mm",  # For preprocessing overrides e.g. on CPU
````
- **EN**: Introduces function `exclude_mm`, function `exclude_bmm`, function `preprocess_mm_configs`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`exclude_mm`、函数`exclude_bmm`、函数`preprocess_mm_configs`。借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1225-1260 / 第 1225-1260 行
````python
        **kwargs,
    ) -> Generator[TritonConfig, None, None]:
        """
        CPU-specific preprocessing that applies CPU-specific scaling (0.5) and exclusion logic.
        """
        # Get CPU-specific exclude function based on operation type
        cpu_exclude_fn = self._get_cpu_exclude_function(op_name)

        # Apply CPU-specific scaling (0.5) and exclusion logic
        return super().preprocess_mm_configs(
            m,
            n,
            k,
            configs=configs,
            has_int8_tensor=has_int8_tensor,
            scale=0.5,
            exclude=cpu_exclude_fn,
            dtype_size=dtype_size,
            op_name=op_name,
            **kwargs,
        )


class CUDAConfigHeuristic(BaseConfigHeuristic):
    """
    Child class for CUDA device specific gemm/flex attention/conv/ configs.
    """

    def __init__(self) -> None:
        super().__init__()
        self.sm_120_default_flex_config = {
            (torch.float32, 64): FlexConfig(128, 32, 2, 4),
            (torch.float32, 128): FlexConfig(128, 32, 2, 4),
            (torch.float32, 256): FlexConfig(64, 16, 2, 4),
            (torch.bfloat16, 64): FlexConfig(128, 64, 2, 4),
            (torch.bfloat16, 128): FlexConfig(128, 64, 2, 8),
````
- **EN**: Introduces class `CUDAConfigHeuristic`, function `__init__`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`CUDAConfigHeuristic`、函数`__init__`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1261-1296 / 第 1261-1296 行
````python
            (torch.bfloat16, 256): FlexConfig(32, 64, 2, 4),
            (torch.float16, 64): FlexConfig(128, 64, 2, 4),
            (torch.float16, 128): FlexConfig(128, 64, 2, 8),
            (torch.float16, 256): FlexConfig(32, 64, 2, 4),
        }

        self.sm_100_default_flex_config = {
            (torch.float32, 64): FlexConfig(128, 32, 3, 4),
            (torch.float32, 128): FlexConfig(32, 64, 3, 4),
            (torch.float32, 192): FlexConfig(32, 64, 2, 4),
            (torch.float32, 256): FlexConfig(32, 32, 3, 4),
            (torch.bfloat16, 64): FlexConfig(128, 128, 3, 4),
            (torch.bfloat16, 128): FlexConfig(128, 64, 3, 8),
            (torch.bfloat16, 192): FlexConfig(128, 128, 1, 8),
            (torch.bfloat16, 256): FlexConfig(64, 32, 3, 4),
            (torch.float16, 64): FlexConfig(128, 128, 3, 4),
            (torch.float16, 128): FlexConfig(128, 64, 3, 8),
            (torch.float16, 192): FlexConfig(128, 128, 1, 8),
            (torch.float16, 256): FlexConfig(64, 32, 3, 4),
        }

        self.h100_default_flex_config = {
            (torch.float32, 64): FlexConfig(128, 32, 3, 4),
            (torch.float32, 128): FlexConfig(32, 64, 3, 4),
            (torch.float32, 192): FlexConfig(32, 64, 1, 8),
            (torch.float32, 256): FlexConfig(32, 32, 3, 4),
            (torch.bfloat16, 64): FlexConfig(128, 128, 3, 4),
            (torch.bfloat16, 128): FlexConfig(128, 64, 3, 8),
            (torch.bfloat16, 256): FlexConfig(64, 32, 3, 4),
            (torch.float16, 64): FlexConfig(128, 128, 3, 4),
            (torch.float16, 128): FlexConfig(128, 64, 3, 8),
            (torch.float16, 256): FlexConfig(64, 32, 3, 4),
        }

        self.a100_default_flex_config = {
            (torch.float32, 64): FlexConfig(128, 32, 3, 4),
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. This range continues the implementation of function `CUDAConfigHeuristic.__init__`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。这一段延续了函数`CUDAConfigHeuristic.__init__` 的具体实现。

### Lines 1297-1332 / 第 1297-1332 行
````python
            (torch.float32, 128): FlexConfig(128, 32, 3, 4),
            (torch.float32, 256): FlexConfig(64, 16, 3, 4),
            (torch.bfloat16, 64): FlexConfig(128, 64, 3, 4),
            (torch.bfloat16, 128): FlexConfig(128, 64, 3, 8),
            (torch.bfloat16, 256): FlexConfig(32, 64, 3, 4),
            (torch.float16, 64): FlexConfig(128, 64, 3, 4),
            (torch.float16, 128): FlexConfig(128, 64, 3, 8),
            (torch.float16, 256): FlexConfig(32, 64, 3, 4),
        }

        # Overwriting the configs omitting BLOCK_N of size 128 that cause ULFs
        self.flex_attn_bwd_autotune_configs: list[FlexBwDConfig] = [
            # See Note: flex bwd configs
            FlexBwDConfig(BLOCK_M, BLOCK_N, BLOCK_N, BLOCK_M, s, 4)
            for BLOCK_M in [32, 64]
            for BLOCK_N in [32, 64]
            for s in [1, 3, 4, 5]  # num_stages
            if BLOCK_N % BLOCK_M == 0
        ]

    def get_flex_attn_fwd_configs(self, head_dim: int, dtype: Any) -> list[FlexConfig]:
        capability = torch.cuda.get_device_capability()
        flex_attn_fwd_configs: list[FlexConfig] = []

        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
                return self.exhaustive_flex_attn_fwd_configs
            flex_attn_fwd_configs += self.flex_attn_fwd_autotune_configs

        if head_dim <= 256:
            if dtype == torch.float32:
                default_config = FlexConfig(64, 64, 3, 4)
            else:
                default_config = FlexConfig(64, 64, 3, 4)
            # here we are using sm_120_default_flex_config on THOR as well
            if capability >= (11, 0):
````
- **EN**: Introduces function `get_flex_attn_fwd_configs`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_flex_attn_fwd_configs`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1333-1368 / 第 1333-1368 行
````python
                default_config = self.sm_120_default_flex_config.get(
                    (dtype, head_dim), default_config
                )
            elif capability >= (10, 0):
                default_config = self.sm_100_default_flex_config.get(
                    (dtype, head_dim), default_config
                )
            elif capability == (9, 0):
                default_config = self.h100_default_flex_config.get(
                    (dtype, head_dim), default_config
                )
            elif capability >= (8, 0):
                default_config = self.a100_default_flex_config.get(
                    (dtype, head_dim), default_config
                )
        else:
            if dtype == torch.float32:
                default_config = FlexConfig(32, 16, 3, 4)
            else:
                default_config = FlexConfig(64, 32, 3, 4)

        if default_config not in flex_attn_fwd_configs:
            flex_attn_fwd_configs.append(default_config)

        return flex_attn_fwd_configs

    def get_flex_attn_bwd_configs(
        self, head_dim: int, dtype: Any
    ) -> list[FlexBwDConfig]:
        capability = torch.cuda.get_device_capability()
        flex_attn_bwd_configs: list[FlexBwDConfig] = []
        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
                return self.exhaustive_flex_attn_bwd_configs
            flex_attn_bwd_configs += self.flex_attn_bwd_autotune_configs

````
- **EN**: Introduces function `get_flex_attn_bwd_configs`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_flex_attn_bwd_configs`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1369-1404 / 第 1369-1404 行
````python
        major, minor = capability
        if dtype == torch.float32:
            capability_class = "float32"
        elif major == 12:
            capability_class = "sm12x"
        elif major == 11:
            capability_class = "sm11x"
        elif major >= 10:
            capability_class = "sm10x"
        elif capability == (9, 0):
            capability_class = "sm90"
        elif major >= 8:
            capability_class = "sm8x"
        else:
            capability_class = "baseline"

        # fmt: off
        config_map = {
            "float32": lambda h: FlexBwDConfig(16, 16, 16, 16, 1, 4),
            "baseline": lambda h: FlexBwDConfig(16, 16, 16, 16, 1, 4),
            "sm90": lambda h: (
                FlexBwDConfig(64, 64, 64, 64, 3, 4) if h < 64 else
                FlexBwDConfig(64, 128, 128, 64, 3, 8) if h <= 128 else
                FlexBwDConfig(64, 64, 64, 64, 2, 4)
            ),
            "sm10x": lambda h: (
                FlexBwDConfig(64, 128, 128, 64, 3, 4) if h <= 128 else
                FlexBwDConfig(64, 64, 64, 64, 1, 8) if h <= 192 else
                FlexBwDConfig(64, 64, 64, 64, 1, 4)
            ),
            "sm8x": lambda h: (
                FlexBwDConfig(32, 128, 128, 32, 3, 4)
                if h < 64
                else FlexBwDConfig(
                    64, 64, 64, 64, 3 if minor == 6 and h == 128 else 2, 4
                )
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `capability_class`, `else`, and `config_map`. This range continues the implementation of function `CUDAConfigHeuristic.get_flex_attn_bwd_configs`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `capability_class`、`else`、`config_map` 等值。这一段延续了函数`CUDAConfigHeuristic.get_flex_attn_bwd_configs` 的具体实现。

### Lines 1405-1440 / 第 1405-1440 行
````python
            ),
            "sm11x": lambda h: (
                FlexBwDConfig(32, 128, 128, 32, 3, 4)
                if h < 64
                else FlexBwDConfig(
                    64, 64, 64, 64, 1 if h >= 128 else 2, 4
                )
            ),
            "sm12x": lambda h: (
                FlexBwDConfig(32, 128, 128, 32, 3, 4)
                if h < 64
                else FlexBwDConfig(
                    64, 64, 64, 64, 1 if h >= 128 else 2, 4
                )
            ),
        }
        # fmt: on

        if head_dim <= 256:
            default_config = config_map[capability_class](head_dim)
        else:
            default_config = FlexBwDConfig(16, 16, 16, 16, 1, 4)

        if default_config not in flex_attn_bwd_configs:
            flex_attn_bwd_configs.append(default_config)

        return flex_attn_bwd_configs

    def get_flex_decode_configs(
        self, head_dim: int, dtype: Any
    ) -> list[FlexDecodeConfig]:
        capability = torch.cuda.get_device_capability()

        default_config = FlexDecodeConfig(64, 1, 2)

        flex_decode_configs: list[FlexDecodeConfig] = []
````
- **EN**: Introduces function `get_flex_decode_configs`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_flex_decode_configs`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1441-1476 / 第 1441-1476 行
````python

        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
                return self.exhaustive_flex_decode_configs
            flex_decode_configs += self.flex_decode_autotune_configs

        if capability in [(9, 0), (10, 0), (10, 3)]:  # sm_90, sm_100, sm_103
            if head_dim > 128 and dtype == torch.float32:
                default_config = FlexDecodeConfig(64, 1, 2)
            else:
                default_config = FlexDecodeConfig(64, 3, 2)
        if capability == (11, 0):
            default_config = FlexDecodeConfig(16, 1, 2)
        else:
            default_config = FlexDecodeConfig(64, 1, 2)

        if default_config not in flex_decode_configs:
            flex_decode_configs.append(default_config)

        return flex_decode_configs


class ROCmConfigHeuristic(BaseConfigHeuristic):
    """
    Child class for ROCm specific gemm/flex attention/conv/ configs.
    """

    def __init__(self) -> None:
        super().__init__()

        self.default_num_stages = get_backend_num_stages()

        self.mm_configs: list[BaseConfig] = [
            ROCmGemmConfig(
                16, 16, 256, self.default_num_stages, 4, group_m=4, waves_per_eu=2
            ),
````
- **EN**: Introduces class `ROCmConfigHeuristic`, function `__init__`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`ROCmConfigHeuristic`、函数`__init__`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1477-1512 / 第 1477-1512 行
````python
            ROCmGemmConfig(32, 16, 256, self.default_num_stages, 4, group_m=4),
            ROCmGemmConfig(
                32, 32, 16, self.default_num_stages, 4, group_m=8, waves_per_eu=2
            ),
            ROCmGemmConfig(32, 32, 128, self.default_num_stages, 4, group_m=8),
            ROCmGemmConfig(32, 64, 64, self.default_num_stages, 4, group_m=8),
            ROCmGemmConfig(
                64, 16, 128, self.default_num_stages, 4, group_m=8, waves_per_eu=2
            ),
            ROCmGemmConfig(64, 32, 32, self.default_num_stages, 4, group_m=8),
            ROCmGemmConfig(64, 32, 64, self.default_num_stages, 4, group_m=8),
            ROCmGemmConfig(64, 32, 64, self.default_num_stages, 8, group_m=8),
            ROCmGemmConfig(64, 32, 128, self.default_num_stages, 4, group_m=8),
            ROCmGemmConfig(64, 64, 16, self.default_num_stages, 4, group_m=8),
            ROCmGemmConfig(64, 64, 64, self.default_num_stages, 4, group_m=4),
            ROCmGemmConfig(64, 64, 128, self.default_num_stages, 8, group_m=16),
            ROCmGemmConfig(64, 64, 256, self.default_num_stages, 8, group_m=4),
            ROCmGemmConfig(
                64, 128, 32, self.default_num_stages, 4, group_m=4, waves_per_eu=2
            ),
            ROCmGemmConfig(64, 128, 32, self.default_num_stages, 8, group_m=8),
            ROCmGemmConfig(64, 128, 64, self.default_num_stages, 8, group_m=4),
            ROCmGemmConfig(64, 128, 128, self.default_num_stages, 8, group_m=4),
            ROCmGemmConfig(128, 32, 32, self.default_num_stages, 4, group_m=8),
            ROCmGemmConfig(128, 32, 64, self.default_num_stages, 4, group_m=8),
            ROCmGemmConfig(
                128, 64, 32, self.default_num_stages, 4, group_m=8, waves_per_eu=2
            ),
            ROCmGemmConfig(128, 64, 64, self.default_num_stages, 4, group_m=16),
            ROCmGemmConfig(128, 64, 128, self.default_num_stages, 8, group_m=4),
            ROCmGemmConfig(
                128, 128, 32, self.default_num_stages, 4, group_m=16, waves_per_eu=2
            ),
            ROCmGemmConfig(128, 128, 32, self.default_num_stages, 8, group_m=16),
            ROCmGemmConfig(
                128, 128, 32, self.default_num_stages, 8, group_m=16, waves_per_eu=2
````
- **EN**: This range continues the implementation of function `ROCmConfigHeuristic.__init__`.
- **CN**: 这一段延续了函数`ROCmConfigHeuristic.__init__` 的具体实现。

### Lines 1513-1548 / 第 1513-1548 行
````python
            ),
            ROCmGemmConfig(128, 128, 64, self.default_num_stages, 4, group_m=16),
            ROCmGemmConfig(128, 128, 64, self.default_num_stages, 8, group_m=8),
            ROCmGemmConfig(128, 128, 128, self.default_num_stages, 8, group_m=16),
            ROCmGemmConfig(
                128, 256, 32, self.default_num_stages, 4, group_m=16, waves_per_eu=2
            ),
            ROCmGemmConfig(128, 256, 64, self.default_num_stages, 8, group_m=4),
            ROCmGemmConfig(256, 64, 64, self.default_num_stages, 8, group_m=4),
            ROCmGemmConfig(
                256, 128, 32, self.default_num_stages, 4, group_m=4, waves_per_eu=2
            ),
            ROCmGemmConfig(256, 128, 32, self.default_num_stages, 8, group_m=16),
            ROCmGemmConfig(256, 128, 64, self.default_num_stages, 8, group_m=4),
            ROCmGemmConfig(256, 256, 64, self.default_num_stages, 8, group_m=4),
        ]

        # Exhaustive search for mm configs
        self.exhaustive_configs: list[BaseConfig] = [
            ROCmGemmConfig(
                BLOCK_M,
                BLOCK_N,
                BLOCK_K,
                num_stages,
                num_warps,
                group_m=group_m,
                matrix_instr_nonkdim=matrix_instr_nonkdim,
                waves_per_eu=waves_per_eu,
                kpack=kpack,
            )
            for BLOCK_M, BLOCK_N, BLOCK_K in itertools.product(
                [16, 32, 64, 128, 256], repeat=3
            )
            for num_stages in [1, self.default_num_stages]
            for num_warps in [4, 8]
            for group_m in [4, 8, 16]
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `group_m`, `matrix_instr_nonkdim`, `waves_per_eu`, and `kpack`. This range continues the implementation of function `ROCmConfigHeuristic.__init__`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `group_m`、`matrix_instr_nonkdim`、`waves_per_eu`、`kpack` 等值。这一段延续了函数`ROCmConfigHeuristic.__init__` 的具体实现。

### Lines 1549-1584 / 第 1549-1584 行
````python
            for matrix_instr_nonkdim in [0, 16]
            for waves_per_eu in [0, 2]
            for kpack in [1, 2]
        ]

        # Architecture-aware default kpack for flex configs
        default_kpack = get_default_kpack()

        self.default_flex_config = {
            (torch.float32, 64): ROCmFlexConfig(128, 32, 1, 4, kpack=default_kpack),
            (torch.float32, 128): ROCmFlexConfig(128, 32, 1, 4, kpack=default_kpack),
            (torch.float32, 256): ROCmFlexConfig(64, 16, 1, 4, kpack=default_kpack),
            (torch.bfloat16, 64): ROCmFlexConfig(128, 64, 2, 4, kpack=default_kpack),
            (torch.bfloat16, 128): ROCmFlexConfig(128, 64, 2, 4, kpack=default_kpack),
            (torch.bfloat16, 256): ROCmFlexConfig(32, 64, 2, 4, kpack=default_kpack),
            (torch.float16, 64): ROCmFlexConfig(128, 64, 2, 4, kpack=default_kpack),
            (torch.float16, 128): ROCmFlexConfig(128, 64, 2, 4, kpack=default_kpack),
            (torch.float16, 256): ROCmFlexConfig(32, 64, 2, 4, kpack=default_kpack),
        }

        self.flex_attn_fwd_autotune_configs: list[FlexConfig] = [
            ROCmFlexConfig(BLOCK1, BLOCK2, 1, w, kpack=default_kpack)
            for BLOCK1 in [16, 64, 128]
            for BLOCK2 in [16, 32, 64, 128]
            for w in [4, 8]
        ]

        self.flex_attn_bwd_autotune_configs: list[FlexBwDConfig] = [
            # See Note: flex bwd configs
            ROCmFlexBwDConfig(
                BLOCK1, BLOCK2, BLOCK2, BLOCK1, 1, w, mfma, kpack=default_kpack
            )
            for BLOCK1 in [16, 32, 64]
            for BLOCK2 in [32, 64, 128]
            for w in ([4, 8] if BLOCK1 >= 128 or BLOCK2 >= 128 else [4])
            for mfma in [0, 16]
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `default_kpack`. This range continues the implementation of function `ROCmConfigHeuristic.__init__`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `default_kpack` 等值。这一段延续了函数`ROCmConfigHeuristic.__init__` 的具体实现。

### Lines 1585-1620 / 第 1585-1620 行
````python
            if BLOCK2 % BLOCK1 == 0
        ]

        self.flex_decode_autotune_configs: list[FlexDecodeConfig] = [
            ROCmFlexDecodeConfig(32, 1, 4, kpack=default_kpack),
            ROCmFlexDecodeConfig(64, 1, 4, kpack=default_kpack),
            ROCmFlexDecodeConfig(128, 1, 4, kpack=default_kpack),
            ROCmFlexDecodeConfig(32, 1, 8, kpack=default_kpack),
            ROCmFlexDecodeConfig(64, 1, 8, kpack=default_kpack),
            ROCmFlexDecodeConfig(128, 1, 8, kpack=default_kpack),
        ]

        self.exhaustive_flex_attn_fwd_configs: list[FlexConfig] = [
            ROCmFlexConfig(BLOCK_M, BLOCK_N, num_stages, num_warps, mfma, wpeu, kpack)
            for BLOCK_M in [16, 32, 64, 128]
            for BLOCK_N in [32, 64, 128]
            for num_stages in [1, 2]
            for num_warps in [2, 4, 8]
            for mfma in [0, 16]
            for wpeu in [0, int(8 // num_warps)]
            for kpack in [1, 2]
        ]

        self.exhaustive_flex_attn_bwd_configs: list[FlexBwDConfig] = [
            # See Note: flex bwd configs
            ROCmFlexBwDConfig(
                BLOCK_M1,
                BLOCK_N1,
                BLOCK_M2,
                BLOCK_N2,
                num_stages,
                num_warps,
                mfma,
                wpeu,
                kpack,
            )
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `ROCmConfigHeuristic.__init__`.
- **CN**: 包含分支、循环或上下文管理等控制流。这一段延续了函数`ROCmConfigHeuristic.__init__` 的具体实现。

### Lines 1621-1656 / 第 1621-1656 行
````python
            for BLOCK_M1 in [16, 32, 64, 128]
            for BLOCK_N1 in [16, 32, 64, 128]
            for BLOCK_M2 in [16, 32, 64, 128]
            for BLOCK_N2 in [16, 32, 64, 128]
            for num_stages in [1, 2]
            for num_warps in [2, 4, 8]
            for mfma in [0, 16]
            for wpeu in [0, int(8 // num_warps)]
            for kpack in [1, 2]
            if BLOCK_N1 % BLOCK_M1 == 0
            and BLOCK_M2 % BLOCK_N2 == 0  # kernel static assertions
        ]

        self.exhaustive_flex_decode_configs: list[FlexDecodeConfig] = [
            ROCmFlexDecodeConfig(
                block_n, num_stages, num_warps, mfma, wpeu, kpack=kpack
            )
            for block_n in [16, 32, 64, 128]
            for num_stages in [1, 2]
            for num_warps in [2, 4, 8]
            for mfma in [0, 16]
            for wpeu in [0, int(8 // num_warps)]
            for kpack in [1, 2]
        ]

    def _prune_exhaustive_configs(
        self,
        configs: list[BaseConfig],
        dtype_size: int,
    ) -> list[BaseConfig]:
        # these cause AMD compile to crash
        pruned_configs = [
            c
            for c in configs
            if not (
                (
````
- **EN**: Introduces function `_prune_exhaustive_configs`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `configs`, `dtype_size`, and `pruned_configs`.
- **CN**: 这里定义了函数`_prune_exhaustive_configs`。包含分支、循环或上下文管理等控制流。初始化或更新了 `configs`、`dtype_size`、`pruned_configs` 等值。

### Lines 1657-1692 / 第 1657-1692 行
````python
                    getattr(c, "matrix_instr_nonkdim", 0) == 2
                    and getattr(c, "kpack", 0) == 2
                )
                or (c.block_k <= 16 and getattr(c, "kpack", 0) == 2)
            )
        ]
        return pruned_configs

    def _filter_configs(self, configs: list[BaseConfig]) -> list[BaseConfig]:
        """
        ROCm specific filtering
        """
        for c in configs:
            c.num_stages = self.default_num_stages
        return super()._filter_configs(configs)

    def _finalize_mm_configs(
        self,
        configs: list[BaseConfig],
    ) -> Generator[TritonConfig, None, None]:
        """
        Finalizes configs after scaling, applying additional constraints.
        """
        used: OrderedSet[tuple[int, ...]] = OrderedSet()

        max_mm_configs = config.test_configs.max_mm_configs

        for conf in configs:
            # Each warp computes a 16x16 tile = 256 elements
            conf.num_warps = min(conf.num_warps, conf.block_m * conf.block_n // 256)

            # Defaults for AMD triton backend kern args if not set
            matrix_instr_nonkdim: int = getattr(conf, "matrix_instr_nonkdim", 16)
            waves_per_eu: int = getattr(conf, "waves_per_eu", 0)
            # Use explicit kpack if set, otherwise determine optimal value based on
            # architecture and BLOCK_K
````
- **EN**: Introduces function `_filter_configs`, function `_finalize_mm_configs`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_filter_configs`、函数`_finalize_mm_configs`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1693-1728 / 第 1693-1728 行
````python
            kpack: int = getattr(conf, "kpack", get_default_kpack(conf.block_k))

            if matrix_instr_nonkdim != 0 and (
                conf.block_m % matrix_instr_nonkdim != 0
                or conf.block_n % matrix_instr_nonkdim != 0
            ):
                #  block_m and block_n must be a multiple of matrix_instr_nonkdim
                continue

            # Construct key for finding duplicate configs
            key: tuple[int, ...] = (
                conf.block_m,
                conf.block_n,
                conf.block_k,
                conf.num_stages,
                conf.num_warps,
                waves_per_eu,
                matrix_instr_nonkdim,
                kpack,
            )

            # Check if gemm specific arg exists - add to key if does
            group_m = getattr(conf, "group_m", None)
            # AMD GPU crashes if group_m = 0
            if group_m is not None and group_m <= 0:
                group_m = 8
            if group_m is not None:
                key += (group_m,)

            if waves_per_eu != 0:
                waves_per_eu = int(8 // conf.num_warps)

            if key not in used and (
                max_mm_configs is None or len(used) < max_mm_configs
            ):
                used.add(key)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kpack`, `key`, `group_m`, and `waves_per_eu`. This range continues the implementation of function `ROCmConfigHeuristic._finalize_mm_configs`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `kpack`、`key`、`group_m`、`waves_per_eu` 等值。这一段延续了函数`ROCmConfigHeuristic._finalize_mm_configs` 的具体实现。

### Lines 1729-1764 / 第 1729-1764 行
````python
                kwargs = {
                    "BLOCK_M": conf.block_m,
                    "BLOCK_N": conf.block_n,
                    "BLOCK_K": conf.block_k,
                    "num_stages": conf.num_stages,
                    "num_warps": conf.num_warps,
                    "matrix_instr_nonkdim": matrix_instr_nonkdim,
                    "waves_per_eu": waves_per_eu,
                    "kpack": kpack,
                }
                if group_m is not None:
                    kwargs["GROUP_M"] = group_m
                yield self.triton_config(**kwargs)

    def get_flex_attn_fwd_configs(self, head_dim: int, dtype: Any) -> list[FlexConfig]:
        flex_attn_fwd_configs: list[FlexConfig] = []

        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
                return self.exhaustive_flex_attn_fwd_configs
            flex_attn_fwd_configs += self.flex_attn_fwd_autotune_configs

        default_kpack = get_default_kpack()
        if head_dim <= 256:
            if dtype == torch.float32:
                default_config = ROCmFlexConfig(64, 64, 1, 4, kpack=default_kpack)
            else:
                default_config = ROCmFlexConfig(128, 64, 2, 4, kpack=default_kpack)
            default_config = self.default_flex_config.get(
                (dtype, head_dim), default_config
            )
        else:
            if dtype == torch.float32:
                default_config = ROCmFlexConfig(32, 16, 1, 4, kpack=default_kpack)
            else:
                default_config = ROCmFlexConfig(64, 32, 2, 4, kpack=default_kpack)
````
- **EN**: Introduces function `get_flex_attn_fwd_configs`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_flex_attn_fwd_configs`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1765-1800 / 第 1765-1800 行
````python

        if default_config not in flex_attn_fwd_configs:
            flex_attn_fwd_configs.append(default_config)

        return flex_attn_fwd_configs

    def get_flex_attn_bwd_configs(
        self, head_dim: int, dtype: Any
    ) -> list[FlexBwDConfig]:
        flex_attn_bwd_configs: list[FlexBwDConfig] = []

        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
                return self.exhaustive_flex_attn_bwd_configs
            flex_attn_bwd_configs += self.flex_attn_bwd_autotune_configs

        default_kpack = get_default_kpack()
        if dtype == torch.float32:
            default_config = ROCmFlexBwDConfig(
                16, 16, 16, 16, 1, 4, kpack=default_kpack
            )
        elif head_dim <= 256:
            if head_dim == 64:
                default_config = ROCmFlexBwDConfig(
                    64, 64, 64, 64, 1, 4, kpack=default_kpack
                )
            elif head_dim == 128:
                default_config = ROCmFlexBwDConfig(
                    64, 128, 128, 64, 1, 4, kpack=default_kpack
                )
            else:
                default_config = ROCmFlexBwDConfig(
                    64, 64, 64, 64, 1, 4, kpack=default_kpack
                )
        else:
            default_config = ROCmFlexBwDConfig(
````
- **EN**: Introduces function `get_flex_attn_bwd_configs`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_flex_attn_bwd_configs`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1801-1836 / 第 1801-1836 行
````python
                16, 16, 16, 16, 1, 4, kpack=default_kpack
            )

        if default_config not in flex_attn_bwd_configs:
            flex_attn_bwd_configs.append(default_config)

        return flex_attn_bwd_configs

    def get_flex_decode_configs(
        self, head_dim: int, dtype: Any
    ) -> list[FlexDecodeConfig]:
        flex_decode_configs: list[FlexDecodeConfig] = []

        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
                return self.exhaustive_flex_decode_configs
            flex_decode_configs += self.flex_decode_autotune_configs

        default_kpack = get_default_kpack()
        default_config = ROCmFlexDecodeConfig(64, 1, 4, kpack=default_kpack)

        if default_config not in flex_decode_configs:
            flex_decode_configs.append(default_config)

        return flex_decode_configs


class XPUConfigHeuristic(BaseConfigHeuristic):
    """
    Placeholder child class for Intel GPU specific overrides.
    """

    def __init__(self) -> None:
        super().__init__()
        self.mm_configs = self.mm_configs + [
            GemmConfig(32, 64, 128, 2, 2),
````
- **EN**: Introduces function `get_flex_decode_configs`, class `XPUConfigHeuristic`, function `__init__`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_flex_decode_configs`、类`XPUConfigHeuristic`、函数`__init__`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 1837-1872 / 第 1837-1872 行
````python
            GemmConfig(64, 64, 32, 2, 8),
        ]
        self.xpu_default_flex_config = {
            (torch.float32, 64): FlexConfig(128, 32, 1, 16),
            (torch.float32, 128): FlexConfig(128, 32, 1, 16),
            (torch.float32, 256): FlexConfig(64, 16, 1, 8),
            (torch.bfloat16, 64): FlexConfig(128, 64, 1, 16),
            (torch.bfloat16, 128): FlexConfig(128, 64, 1, 16),
            (torch.bfloat16, 256): FlexConfig(32, 64, 1, 4),
            (torch.float16, 64): FlexConfig(128, 64, 1, 16),
            (torch.float16, 128): FlexConfig(128, 64, 1, 16),
            (torch.float16, 256): FlexConfig(32, 64, 1, 4),
        }
        self.flex_attn_fwd_autotune_configs: list[FlexConfig] = [
            FlexConfig(32, 16, 2, 4),
            FlexConfig(128, 64, 2, 16),
            FlexConfig(128, 64, 2, 8),
            FlexConfig(128, 32, 2, 16),
            FlexConfig(128, 32, 2, 8),
        ]
        self.flex_attn_bwd_autotune_configs: list[FlexBwDConfig] = [
            FlexBwDConfig(32, 32, 32, 32, 2, 4),
            FlexBwDConfig(64, 64, 64, 64, 2, 4),
        ]
        self.flex_decode_autotune_configs: list[FlexDecodeConfig] = []

        if not bool(os.getenv("CI")):
            self.flex_attn_bwd_autotune_configs += [
                # See Note: flex bwd configs
                FlexBwDConfig(BLOCK1, BLOCK2, BLOCK2, BLOCK1, s, w)
                for BLOCK1 in [32, 64]
                for BLOCK2 in [32, 64, 128]
                for s in [1, 3, 4, 5]  # num_stages
                for w in ([4, 8] if BLOCK1 >= 128 or BLOCK2 >= 128 else [4])
                if BLOCK2 % BLOCK1 == 0
            ]
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `XPUConfigHeuristic.__init__`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。这一段延续了函数`XPUConfigHeuristic.__init__` 的具体实现。

### Lines 1873-1908 / 第 1873-1908 行
````python
            self.flex_decode_autotune_configs += [
                FlexDecodeConfig(32, 1, 2),
                FlexDecodeConfig(32, 1, 1),
                FlexDecodeConfig(32, 2, 2),
                FlexDecodeConfig(32, 2, 1),
                FlexDecodeConfig(64, 1, 2),
                FlexDecodeConfig(64, 1, 1),
                FlexDecodeConfig(64, 2, 2),
                FlexDecodeConfig(64, 2, 1),
            ]

    def get_flex_attn_fwd_configs(self, head_dim: int, dtype: Any) -> list[FlexConfig]:
        flex_attn_fwd_configs: list[FlexConfig] = []

        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
                return self.exhaustive_flex_attn_fwd_configs
            flex_attn_fwd_configs += self.flex_attn_fwd_autotune_configs

        if head_dim <= 256:
            if dtype == torch.float32:
                default_config = FlexConfig(64, 64, 1, 8)
            else:
                default_config = FlexConfig(128, 64, 1, 16)
            default_config = self.xpu_default_flex_config.get(
                (dtype, head_dim), default_config
            )
        else:
            if dtype == torch.float32:
                default_config = FlexConfig(32, 16, 1, 4)
            else:
                default_config = FlexConfig(64, 32, 1, 8)

        if default_config not in flex_attn_fwd_configs:
            flex_attn_fwd_configs.append(default_config)

````
- **EN**: Introduces function `get_flex_attn_fwd_configs`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_flex_attn_fwd_configs`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1909-1944 / 第 1909-1944 行
````python
        return flex_attn_fwd_configs

    def get_flex_attn_bwd_configs(
        self, head_dim: int, dtype: Any
    ) -> list[FlexBwDConfig]:
        flex_attn_bwd_configs: list[FlexBwDConfig] = []

        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
                return self.exhaustive_flex_attn_bwd_configs
            flex_attn_bwd_configs += self.flex_attn_bwd_autotune_configs

        if dtype == torch.float32:
            default_config = FlexBwDConfig(16, 16, 16, 16, 1, 4)
        elif head_dim <= 256:
            if head_dim == 64:
                default_config = FlexBwDConfig(64, 64, 64, 64, 1, 8)
            elif head_dim == 128:
                default_config = FlexBwDConfig(64, 64, 64, 64, 1, 8)
            else:
                default_config = FlexBwDConfig(64, 64, 64, 64, 1, 8)
        else:  # modest hardware or extremely large head_dim
            default_config = FlexBwDConfig(16, 16, 16, 16, 1, 4)

        if default_config not in flex_attn_bwd_configs:
            flex_attn_bwd_configs.append(default_config)

        return flex_attn_bwd_configs

    def get_flex_decode_configs(
        self, head_dim: int, dtype: Any
    ) -> list[FlexDecodeConfig]:
        flex_decode_configs: list[FlexDecodeConfig] = []

        if config.max_autotune:
            if config.max_autotune_flex_search_space == "EXHAUSTIVE":
````
- **EN**: Introduces function `get_flex_attn_bwd_configs`, function `get_flex_decode_configs`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_flex_attn_bwd_configs`、函数`get_flex_decode_configs`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1945-1980 / 第 1945-1980 行
````python
                return self.exhaustive_flex_decode_configs
            flex_decode_configs += self.flex_decode_autotune_configs

        default_config = FlexDecodeConfig(64, 1, 2)

        if default_config not in flex_decode_configs:
            flex_decode_configs.append(default_config)

        return flex_decode_configs

    def _prune_exhaustive_configs(
        self,
        configs: list[BaseConfig],
        dtype_size: int,
    ) -> list[BaseConfig]:
        return configs


class MTIAConfigHeuristic(BaseConfigHeuristic):
    """
    Placeholder child class for MTIA specific overrides.
    """


# Template-specific mixin classes
class MMTemplateConfigMixin(GemmMaxAutotuneTemplateConfigHeuristics):
    """
    Mixin class that converts config lists to template kwargs.
    This handles the logic that was previously in choices.get_mm_configs.

    This mixin expects to be used with BaseConfigHeuristic or its subclasses.
    """

    # Type annotations to ensure the mixin works with BaseConfigHeuristic
    get_mm_configs: Callable[[], partial[Generator[TritonConfig, None, None]]]
    get_exhaustive_mm_configs: Callable[
````
- **EN**: Introduces function `_prune_exhaustive_configs`, class `MTIAConfigHeuristic`, class `MMTemplateConfigMixin`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `default_config`, `configs`, `dtype_size`, `get_mm_configs`, and `get_exhaustive_mm_configs`.
- **CN**: 这里定义了函数`_prune_exhaustive_configs`、类`MTIAConfigHeuristic`、类`MMTemplateConfigMixin`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `default_config`、`configs`、`dtype_size`、`get_mm_configs`、`get_exhaustive_mm_configs` 等值。

### Lines 1981-2016 / 第 1981-2016 行
````python
        [], partial[Generator[TritonConfig, None, None]]
    ]
    _filter_configs: Callable[[list[BaseConfig]], list[BaseConfig]]

    def get_extra_kwargs(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
    ) -> dict[str, Any]:
        assert isinstance(kernel_inputs, MMKernelInputs)
        m, n, k = kernel_inputs.mnk_symbolic()
        # Calculate allow_tf32
        allow_tf32 = torch.backends.cuda.matmul.fp32_precision == "tf32" and (
            not inductor_config.force_same_precision
            or ((m % 16) == 0 and (n % 16) == 0 and (k % 8) == 0)
        )

        return {
            "ALLOW_TF32": allow_tf32,
        }

    def _valid(self, kernel_inputs: KernelInputs) -> bool:
        return True

    def _get_config_generator(
        self,
    ) -> partial[Generator[TritonConfig, None, None]]:
        """
        Get the appropriate config generator based on search space.
        Can be overridden by subclasses for template-specific behavior.
        """
        # Handle exhaustive search case
        if config.max_autotune_gemm_search_space == "EXHAUSTIVE":
            return self.get_exhaustive_mm_configs()
        else:
            return self.get_mm_configs()
````
- **EN**: Introduces function `get_extra_kwargs`, function `_valid`, function `_get_config_generator`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_extra_kwargs`、函数`_valid`、函数`_get_config_generator`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 2017-2052 / 第 2017-2052 行
````python

    def _get_template_configs_impl(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
        **kwargs,
    ) -> Generator[dict[str, Any], None, None]:
        """
        Convert config lists to template kwargs.
        This replaces the logic from choices.get_mm_configs and inlines mm_options.
        """
        assert isinstance(kernel_inputs, MMKernelInputs), (
            f"{self.__class__.__name__} requires MMKernelInputs"
        )
        input_nodes = kernel_inputs.nodes()
        if len(input_nodes) < 2:
            raise ValueError(f"Need at least 2 input tensors, got {len(input_nodes)}")
        if not self._valid(kernel_inputs):
            return

        # Extract M, N, K from kernel_inputs
        m, n, k = kernel_inputs.mnk_symbolic()

        # Extract dtype and device_type from kernel_inputs
        dtype = kernel_inputs.dtype()

        # Get the appropriate config generator
        configs = self._get_config_generator()

        # Generate and process configs
        for c in configs(
            m,
            n,
            k,
            dtype_size=dtype.itemsize,
            op_name=op_name,
````
- **EN**: Introduces function `_get_template_configs_impl`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_get_template_configs_impl`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 2053-2088 / 第 2053-2088 行
````python
            **kwargs,
        ):
            template_kwargs = self._convert_config_to_template_kwargs(
                c,
                m,
                n,
                k,
                kernel_inputs.out_dtype(),
            )
            yield template_kwargs

    def _convert_config_to_template_kwargs(
        self,
        triton_config: TritonConfig,
        m: sympy.Integer | sympy.Symbol,
        n: sympy.Integer | sympy.Symbol,
        k: sympy.Integer | sympy.Symbol,
        out_dtype: torch.dtype,
    ) -> dict[str, Any]:
        """
        Convert triton config to template kwargs.
        Moved from mm_common.mm_options.
        """
        # Calculate EVEN_K symbolic. (It isn't worth guarding on this)
        even_k_symbolic = sympy.Eq(Mod(k, triton_config.kwargs["BLOCK_K"]), 0)
        even_k_symbolic = V.graph.sizevars.statically_known_true(even_k_symbolic)

        # Build options dict

        options_dict = dict(
            EVEN_K=even_k_symbolic,
            USE_FAST_ACCUM=False,  # Option for _scaled_mm
            ACC_TYPE=self._get_acc_type(out_dtype),
            OUT_DTYPE=self._get_out_dtype(out_dtype),
            num_stages=triton_config.num_stages,
            num_warps=triton_config.num_warps,
````
- **EN**: Introduces function `_convert_config_to_template_kwargs`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`_convert_config_to_template_kwargs`。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 2089-2124 / 第 2089-2124 行
````python
            **triton_config.kwargs,
        )

        # If GROUP_M not specified then default to 8
        if "GROUP_M" not in triton_config.kwargs:
            group_m = triton_config.kwargs.get("GROUP_M", 8)
            options_dict["GROUP_M"] = group_m

        return options_dict

    @staticmethod
    def _dtype_to_triton(dtype: torch.dtype) -> str:
        """Convert a torch dtype to a triton type string."""
        return f"tl.{dtype}".replace("torch.", "")

    def _get_acc_type(self, dtype: torch.dtype) -> str:
        """
        Get accumulator type for the given dtype.
        Moved from mm_common.acc_type.
        """
        if dtype in (torch.float16, torch.bfloat16):
            return "tl.float32"
        return self._dtype_to_triton(dtype)

    def _get_out_dtype(self, dtype: torch.dtype) -> str:
        """Get output dtype as a triton type string."""
        return self._dtype_to_triton(dtype)


# INT8 specific mixin to filter correctly
class INT8MMTemplateConfigMixin(MMTemplateConfigMixin):
    """
    Ensure that we feed in has_int8_tensor=True
    """

    def __init__(self) -> None:
````
- **EN**: Introduces function `_dtype_to_triton`, function `_get_acc_type`, function `_get_out_dtype`, class `INT8MMTemplateConfigMixin`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`_dtype_to_triton`、函数`_get_acc_type`、函数`_get_out_dtype`、类`INT8MMTemplateConfigMixin`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 2125-2160 / 第 2125-2160 行
````python
        super().__init__()
        self.has_int8_tensor = True


# MMPlusMM specific mixin to avoid running _scale_mm_configs
class MMPlusMMTemplateConfigMixin(MMTemplateConfigMixin):
    """
    Ensure that _should_scale_configs is False
    """

    # TODO(coconutruben): remove this once all tests work
    # with proper scaling on mm_plus_mm
    def __init__(self) -> None:
        super().__init__()
        self.should_scale_configs = False

    def _get_template_configs_impl(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
        **kwargs,
    ) -> Generator[dict[str, Any], None, None]:
        assert isinstance(kernel_inputs, MMKernelInputs), "Expect MMKernelInputs"
        m, n, k = kernel_inputs.mnk_symbolic()
        for template_kwargs in super()._get_template_configs_impl(
            kernel_inputs, op_name, **kwargs
        ):
            # Apply BLOCK_K constraint specific to mm_plus_mm
            # see https://github.com/triton-lang/triton/issues/1298
            # BLOCK_K = K causes llvm error
            if V.graph.sizevars.statically_known_lt(
                template_kwargs.get("BLOCK_K", k), k
            ):
                yield template_kwargs


````
- **EN**: Introduces class `MMPlusMMTemplateConfigMixin`, function `__init__`, function `_get_template_configs_impl`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`MMPlusMMTemplateConfigMixin`、函数`__init__`、函数`_get_template_configs_impl`。保留了指向设计说明、规范或厂商数据手册的注释引用。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 2161-2196 / 第 2161-2196 行
````python
class TMAWorkspaceMixin(MMTemplateConfigMixin):
    """
    Small mixin to ensure that the workspace arg is correct for TMA
    and TMA specific filtering can happen.
    """

    def get_extra_kwargs(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
    ) -> dict[str, Any]:
        kwargs = super().get_extra_kwargs(kernel_inputs, op_name)
        kwargs["workspace_arg"] = get_tma_workspace_arg(
            num_tma_descriptors=2,
            device=kernel_inputs.device(),
        )
        return kwargs

    # pyrefly: ignore [bad-override]
    def _filter_configs(self, configs: list[BaseConfig]) -> list[BaseConfig]:
        """
        TMA specific filtering, as num_warps=2 not safe for TMA
        """
        configs = [c for c in configs if c.num_warps != 2]
        return super()._filter_configs(configs)


def get_shared_memory_checker_opts(op_name: str, dtype_size: int):
    return {
        "has_sm_layout_conversion": True,
        # addmm requires the acc dtype for layout conversion due to adding bias
        # mm just input dtype
        "layout_conversion_byte_size": 4 if op_name == "addmm" else dtype_size,
    }


````
- **EN**: Introduces class `TMAWorkspaceMixin`, function `get_extra_kwargs`, function `_filter_configs`, function `get_shared_memory_checker_opts`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`TMAWorkspaceMixin`、函数`get_extra_kwargs`、函数`_filter_configs`、函数`get_shared_memory_checker_opts`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 2197-2232 / 第 2197-2232 行
````python
# TMA-specific mixin for TMA templates
class TMATemplateConfigMixin(TMAWorkspaceMixin, MMTemplateConfigMixin):
    """
    TMA-specific mixin that uses persistent configs and adds TMA options.
    This inherits from MMTemplateConfigMixin and overrides config generation.
    """

    def _get_template_configs_impl(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
        **kwargs,
    ) -> Generator[dict[str, Any], None, None]:
        """
        Generate TMA template configs by calling super and adding TMA-specific options.
        """
        assert isinstance(kernel_inputs, MMKernelInputs), (
            "TMATemplateConfigMixin requires MMKernelInputs"
        )
        mat1, mat2 = kernel_inputs.mat1mat2()
        tma_opts = {
            "A_ROW_MAJOR": not mat1.layout.is_transposed(),
            "B_ROW_MAJOR": not mat2.layout.is_transposed(),
            "NUM_SMS": get_num_sms(),
            "TMA_SIZE": TMA_DESCRIPTOR_SIZE,
            "TMA_EXPERIMENTAL_API": not has_triton_stable_tma_api(),
            "tma_store": config.triton.enable_template_tma_store,
            "transpose_discontiguous_tensor_descriptors_override": True,
        }

        # Get base template configs from superclass
        for template_kwargs in super()._get_template_configs_impl(
            kernel_inputs,
            op_name,
            **get_shared_memory_checker_opts(
                op_name, dtype_size=kernel_inputs.dtype().itemsize
````
- **EN**: Introduces class `TMATemplateConfigMixin`, function `_get_template_configs_impl`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`TMATemplateConfigMixin`、函数`_get_template_configs_impl`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 2233-2268 / 第 2233-2268 行
````python
            ),
        ):
            yield {**template_kwargs, **tma_opts}


# TMA mixins for Blackwell templates
class BlackwellTMATemplateConfigMixin(TMATemplateConfigMixin):
    def _get_template_configs_impl(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
        **kwargs,
    ) -> Generator[dict[str, Any], None, None]:
        """
        Generate TMA template configs by calling super and adding TMA-specific options.
        """
        # Get base template configs from superclass
        for template_kwargs in super()._get_template_configs_impl(
            kernel_inputs,
            op_name,
            **kwargs,
        ):
            # Some Triton versions requires num_warps >= 4 for WS
            # to avoid compilation issues. Triton disables WS if num_warps < 4
            # or num_stages < 2. Similar issues have been seen with num_stages=1
            constraints_violated = (
                template_kwargs["num_warps"] < 4 or template_kwargs["num_stages"] < 2
            )
            ws = (
                template_kwargs.get("WARP_SPECIALIZE", True)
                and not constraints_violated
            )
            flatten = template_kwargs.get("FLATTEN", True) and not constraints_violated
            yield {
                **template_kwargs,
                "NUM_SMS": get_num_sms(),
````
- **EN**: Introduces class `BlackwellTMATemplateConfigMixin`, function `_get_template_configs_impl`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`BlackwellTMATemplateConfigMixin`、函数`_get_template_configs_impl`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 2269-2304 / 第 2269-2304 行
````python
                "WARP_SPECIALIZE": ws,
                "FLATTEN": flatten,
            }

    @staticmethod
    def _generate_exhaustive_configs() -> list[BaseConfig]:
        configs: list[BaseConfig] = []
        for BLOCK_M, BLOCK_N, BLOCK_K in itertools.product(
            [32, 64, 128, 256],
            repeat=3,
        ):
            for num_stages in [2, 3, 4, 5, 6]:
                # AutoWS doesn't work with num_warps < 4
                for num_warps in [4, 8]:
                    for EPILOGUE_SUBTILE in [1, 2, 4]:
                        configs.append(
                            BlackwellGPUGemmConfig(
                                block_m=BLOCK_M,
                                block_n=BLOCK_N,
                                block_k=BLOCK_K,
                                num_stages=num_stages,
                                num_warps=num_warps,
                                group_m=8,
                                epilogue_subtile=EPILOGUE_SUBTILE,
                                warp_specialize=True,
                                flatten=True,
                            )
                        )
        return configs


# Scaled MM-specific mixin for scaled MM templates
class BaseScaledMMConfigMixin(MMTemplateConfigMixin):
    """
    This is a base that handles the common case for ScaledMM

````
- **EN**: Introduces function `_generate_exhaustive_configs`, class `BaseScaledMMConfigMixin`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_generate_exhaustive_configs`、类`BaseScaledMMConfigMixin`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 2305-2340 / 第 2305-2340 行
````python
    The TMA and non-TMA should build on top of this
    """

    def adjust_kernel_inputs(
        self, kernel_inputs: KernelInputs, op_name: str
    ) -> KernelInputs:
        """
        for scaled_mm, we need to unsqueeze scale tensors, and bias
        """
        assert isinstance(kernel_inputs, MMKernelInputs), (
            "Expect MMKernelInputs for scaled MM"
        )
        inputs = super().adjust_kernel_inputs(kernel_inputs, op_name)
        nodes = inputs.nodes()
        mat_a, mat_b, scale_a, scale_b, *bias = nodes
        bias = bias[0] if bias else None
        # Prepare triton input nodes and create kernel_inputs at the top
        from ..lowering import lowerings as L

        aten = torch.ops.aten
        if bias and len(mat_b.get_size()) == len(bias.get_size()) + 1:
            # Need to unsqueeze bias from [N] -> [1, N]
            bias = L[aten.unsqueeze](bias, 0)

        if len(scale_a.get_size()) == 0 or len(scale_b.get_size()) == 0:
            assert len(scale_a.get_size()) == len(scale_b.get_size())
            # Need to unsqueeze scale from [] -> [1, 1]
            scale_a = L[aten.unsqueeze](L[aten.unsqueeze](scale_a, 0), 1)
            scale_b = L[aten.unsqueeze](L[aten.unsqueeze](scale_b, 0), 1)
        nodes = [mat_a, mat_b, scale_a, scale_b]
        if bias:
            nodes.append(bias)
        return MMKernelInputs(
            nodes,
            mat1_idx=kernel_inputs._mat1_idx,
            mat2_idx=kernel_inputs._mat2_idx,
````
- **EN**: Imports dependencies such as `..lowering` for the logic in this range. Introduces function `adjust_kernel_inputs`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `..lowering` 等依赖，为后续逻辑提供基础能力。这里定义了函数`adjust_kernel_inputs`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 2341-2376 / 第 2341-2376 行
````python
            out_dtype=kernel_inputs._out_dtype,
        )

    def _get_template_configs_impl(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
        **kwargs,
    ) -> Generator[dict[str, Any], None, None]:
        """
        Generate scaled MM template configs with scaled MM-specific options.
        Handles the remaining logic from mm_common, including assertions.
        """
        kernel_inputs = self.adjust_kernel_inputs(kernel_inputs, op_name)
        input_nodes = kernel_inputs.nodes()
        # Initial assertion from mm_common.scaled_mm_options
        assert len(input_nodes) >= 4, (
            f"scaled_mm requires at least 4 inputs, got {len(input_nodes)}"
        )

        # Extract scale tensors (typically scale_a and scale_b are input_nodes[2] and input_nodes[3])
        scale_a = input_nodes[2]
        scale_b = input_nodes[3]

        # Scale compatibility assertion from mm_common.scaled_mm_options
        def are_compatible_scales(size_a: Any, size_b: Any) -> bool:
            # Same sized scales are compatible
            if len(size_a) == len(size_b):
                return True

            # Both need to be scalars or len(1) tensors
            if len(size_a) <= 1 and len(size_b) <= 1:
                return True

            return False

````
- **EN**: Introduces function `_get_template_configs_impl`, function `are_compatible_scales`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `out_dtype`, `kernel_inputs`, `op_name`, `input_nodes`, `scale_a`, and `scale_b`.
- **CN**: 这里定义了函数`_get_template_configs_impl`、函数`are_compatible_scales`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `out_dtype`、`kernel_inputs`、`op_name`、`input_nodes`、`scale_a`、`scale_b` 等值。

### Lines 2377-2412 / 第 2377-2412 行
````python
        size_a, size_b = scale_a.get_size(), scale_b.get_size()
        assert are_compatible_scales(size_a, size_b), (
            "Expect scale_a and scale_b to be either both scalars (including single-element tensors) "
            f"or 1-dimensional tensors with the same size. Got scale_a: {len(size_a)} and scale_b: {len(size_b)}."
        )

        assert isinstance(kernel_inputs, MMKernelInputs), (
            f"{self.__class__.__name__} requires MMKernelInputs"
        )

        if not self._valid(kernel_inputs):
            return

        # Get base template configs from superclass
        for template_kwargs in super()._get_template_configs_impl(
            kernel_inputs, op_name, **kwargs
        ):
            # Add scaled MM-specific options (moved from mm_common.scaled_mm_options)
            # Override accumulator type for scaled MM
            template_kwargs["ACC_TYPE"] = "tl.float32"

            yield template_kwargs


class ScaledMMConfigMixin(BaseScaledMMConfigMixin):
    """Mixing for scaled mm with the regular mm template"""

    def get_extra_kwargs(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
    ) -> dict[str, Any]:
        kwargs = super().get_extra_kwargs(kernel_inputs, op_name)
        from ..kernel.mm_common import scale_mm_epilogue

        return {
````
- **EN**: Imports dependencies such as `..kernel.mm_common` for the logic in this range. Introduces class `ScaledMMConfigMixin`, function `get_extra_kwargs`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `..kernel.mm_common` 等依赖，为后续逻辑提供基础能力。这里定义了类`ScaledMMConfigMixin`、函数`get_extra_kwargs`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 2413-2448 / 第 2413-2448 行
````python
            **kwargs,
            "suffix_args": kernel_inputs.count - 2,
            "epilogue_fn": scale_mm_epilogue(),
            "epilogue_fn_hash": "scale_mm_epilogue",
        }

    def _valid(self, kernel_inputs: KernelInputs) -> bool:
        assert isinstance(kernel_inputs, MMKernelInputs), (
            "Expect MMKernelInputs for ScaledMMConfigMixin"
        )
        _, _, k = kernel_inputs.mnk_symbolic()
        if V.graph.sizevars.guard_or_false(sympy.Le(k, 16)):
            # Triton crashes however uncommon for real workloads
            return False

        # On NVIDIA B200 GPUs, K dim must be >= 32 for tcgen05.mma.kind::f8f6f4.* PTX instruction to be valid
        # source: https://docs.nvidia.com/cuda/parallel-thread-execution/#tcgen05-matrix-shape
        if using_b200() and V.graph.sizevars.guard_or_false(sympy.Lt(k, 32)):
            return False
        return True

    # pyrefly: ignore [bad-override]
    def _filter_configs(self, configs: list[BaseConfig]) -> list[BaseConfig]:
        """
        Filter out bad configs for specific hardware.
        On AMD MI350X (GFX 9.5+), skip configs with BLOCK_K<=64 due to lack of corresponding MFMA instructions.
        """

        def should_skip_mi350x_config(config: BaseConfig) -> bool:
            """Skip config if BLOCK_K<=64 on MI350X (GFX 9.5+)"""
            try:
                return (
                    config.block_k <= 64
                    and torch.version.hip is not None
                    and torch.cuda.get_device_capability() >= (9, 5)
                )
````
- **EN**: Introduces function `_valid`, function `_filter_configs`, function `should_skip_mi350x_config`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里定义了函数`_valid`、函数`_filter_configs`、函数`should_skip_mi350x_config`。保留了指向设计说明、规范或厂商数据手册的注释引用。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 2449-2484 / 第 2449-2484 行
````python
            except RuntimeError:
                # If no HIP GPUs are available, we can't check device capability
                # so we don't skip any configs
                return False

        filtered_configs = [c for c in configs if not should_skip_mi350x_config(c)]
        return super()._filter_configs(filtered_configs)


# Scaled TMA-specific mixin for scaled MM templates with TMA
class ScaledTMAConfigMixin(TMAWorkspaceMixin, BaseScaledMMConfigMixin):
    """
    Scaled TMA-specific mixin that extends BaseScaledMMConfigMixin with TMA functionality.
    This is for scaled MM templates that use device TMA.
    This inherits from BaseScaledMMConfigMixin and adds TMA-specific options.
    """

    # pyrefly: ignore [bad-override]
    def _filter_configs(self, configs: list[BaseConfig]) -> list[BaseConfig]:
        """
        TMA specific filtering:
        - num_warps=2 not safe for TMA
        - block_k >= 32 required for TMA (requires inner-most dimension >= 32)
        """
        configs = [c for c in configs if c.num_warps != 2 and c.block_k >= 32]
        return super()._filter_configs(configs)

    def _get_template_configs_impl(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
        **kwargs,
    ) -> Generator[dict[str, Any], None, None]:
        """
        Generate scaled TMA template configs with both scaled MM and TMA-specific options.
        """
````
- **EN**: Introduces class `ScaledTMAConfigMixin`, function `_filter_configs`, function `_get_template_configs_impl`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了类`ScaledTMAConfigMixin`、函数`_filter_configs`、函数`_get_template_configs_impl`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 2485-2520 / 第 2485-2520 行
````python
        # Get base scaled MM template configs from superclass
        for template_kwargs in super()._get_template_configs_impl(
            kernel_inputs,
            op_name,
            **kwargs,
        ):
            # Add TMA-specific options for device TMA scaled MM
            template_kwargs["TMA_SIZE"] = TMA_DESCRIPTOR_SIZE
            template_kwargs["NUM_SMS"] = get_num_sms()
            template_kwargs["TMA_EXPERIMENTAL_API"] = not has_triton_stable_tma_api()

            yield template_kwargs


# Scaled Blackwell TMA-specific mixin for scaled MM templates with TMA
class ScaledBlackwellTMAConfigMixin(
    BlackwellTMATemplateConfigMixin, ScaledMMConfigMixin
):
    """
    Scaled Blackwell TMA-specific mixin that extends ScaledMMConfigMixin with TMA functionality.
    This is for scaled MM templates that use device TMA on Blackwell.
    This inherits from ScaledMMConfigMixin, which inherits the scale_mm_epilogue, and adds TMA-specific options.
    """

    def _filter_configs(self, configs: list[BaseConfig]) -> list[BaseConfig]:
        """
        Warp specialization-specific filtering (BlackwellTMATemplateConfigMixin)
        (compilation issues occur in some versions of Triton)
        - num_warps < 4 unsafe for warpspec
        - num_stages < 2 unsafe for warpspec

        TMA-specific filtering:
        - block_k >= 32 required for TMA (requires inner-most dimension >= 32)
        """
        configs = [c for c in configs if c.block_k >= 32]
        return super()._filter_configs(configs)
````
- **EN**: Introduces class `ScaledBlackwellTMAConfigMixin`, function `_filter_configs`. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`ScaledBlackwellTMAConfigMixin`、函数`_filter_configs`。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 2521-2556 / 第 2521-2556 行
````python


# Template-specific heuristic classes using multiple inheritance


@register_template_heuristic(
    mm_template.uid,
    "cuda",
    register=torch.version.hip is None,
)
@register_template_heuristic(
    bmm_template.uid,
    "cuda",
    register=torch.version.hip is None,
)
class CUDAMMTemplateConfigHeuristic(MMTemplateConfigMixin, CUDAConfigHeuristic):
    """Standard MM template heuristic for CUDA"""


@register_template_heuristic(
    mm_template.uid, "cuda", register=torch.version.hip is None, op_name="addmm"
)
@register_template_heuristic(
    bmm_template.uid, "cuda", register=torch.version.hip is None, op_name="baddbmm"
)
class CUDAAddMMTemplateConfigHeuristic(AddMMConfigMixin, CUDAMMTemplateConfigHeuristic):
    """Addmm specific mixin for CUDA"""


# TODO(coconutruben): deprecate once autoheuristic is deprecated
@register_template_heuristic(
    mm_template.uid,
    "cuda",
    register=torch.version.hip is None,
    op_name="mm-ah",
)
````
- **EN**: Introduces class `CUDAMMTemplateConfigHeuristic`, class `CUDAAddMMTemplateConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`CUDAMMTemplateConfigHeuristic`、类`CUDAAddMMTemplateConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 2557-2592 / 第 2557-2592 行
````python
class CUDAMMAHTemplateConfigHeuristic(MMTemplateConfigMixin, CUDAConfigHeuristic):
    """Standard MM template heuristic for CUDA using the extra mm configs only (for autoheuristic)"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use scaled_mm_configs
        self.mm_configs = self.extra_mm_configs
        self.exhaustive_configs = self.extra_mm_configs


@register_template_heuristic(
    persistent_tma_mm_template.uid,
    "cuda",
    register=torch.version.hip is None,
)
class CUDAPersistentTMATemplateConfigHeuristic(
    TMATemplateConfigMixin, CUDAConfigHeuristic
):
    """Persistent TMA template heuristic for CUDA"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use persistent_mm_configs
        self.mm_configs = self.persistent_mm_configs


@register_template_heuristic(
    persistent_mm_template.uid,
    "cuda",
    register=torch.version.hip is not None,
)
class PersistentMMTemplateConfigHeuristic(
    MMTemplateConfigMixin,
    ROCmConfigHeuristic,  # type: ignore[misc]
):
    """Persistent MM template heuristic (no TMA, standard pointer loads)"""
````
- **EN**: Introduces class `CUDAMMAHTemplateConfigHeuristic`, function `__init__`, class `CUDAPersistentTMATemplateConfigHeuristic`, function `__init__`, class `PersistentMMTemplateConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`CUDAMMAHTemplateConfigHeuristic`、函数`__init__`、类`CUDAPersistentTMATemplateConfigHeuristic`、函数`__init__`、类`PersistentMMTemplateConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 2593-2628 / 第 2593-2628 行
````python

    def __init__(self) -> None:
        super().__init__()
        self.mm_configs = self.persistent_mm_configs

    def _get_template_configs_impl(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
        **kwargs,
    ) -> Generator[dict[str, Any], None, None]:
        for template_kwargs in super()._get_template_configs_impl(
            kernel_inputs, op_name, **kwargs
        ):
            yield {**template_kwargs, "NUM_SMS": get_num_sms()}


@register_template_heuristic(
    blackwell_ws_persistent_device_tma_mm_template.uid,
    "cuda",
    register=torch.version.hip is None,
)
class CUDABlackwellPersistentTMATemplateConfigHeuristic(
    BlackwellTMATemplateConfigMixin, CUDAConfigHeuristic
):
    """Blackwell Persistent TMA template"""

    def __init__(self) -> None:
        super().__init__()
        self.mm_configs = self.blackwell_persistent_mm_configs
        self.exhaustive_configs = self._generate_exhaustive_configs()


@register_template_heuristic(
    persistent_tma_mm_template.uid,
    "cuda",
````
- **EN**: Introduces function `__init__`, function `_get_template_configs_impl`, class `CUDABlackwellPersistentTMATemplateConfigHeuristic`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`__init__`、函数`_get_template_configs_impl`、类`CUDABlackwellPersistentTMATemplateConfigHeuristic`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 2629-2664 / 第 2629-2664 行
````python
    register=torch.version.hip is None,
    op_name="addmm",
)
class CUDAAddmmPersistentTMATemplateConfigHeuristic(
    AddMMConfigMixin, CUDAPersistentTMATemplateConfigHeuristic
):
    """Addmm specific mixin for CUDA"""


@register_template_heuristic(
    blackwell_ws_persistent_device_tma_mm_template.uid,
    "cuda",
    register=torch.version.hip is None,
    op_name="addmm",
)
class CUDABlackwellAddmmPersistentTMATemplateConfigHeuristic(
    AddMMConfigMixin, CUDABlackwellPersistentTMATemplateConfigHeuristic
):
    """Addmm extension for DataCenter Blackwell Templates"""

    def __init__(self) -> None:
        super().__init__()
        # NOTE: to ensure that we pass tests, addmm needs a small config
        self.mm_configs = (
            self.blackwell_persistent_mm_configs
            + self.blackwell_persistent_addmm_configs
        )
        self.exhaustive_configs = self._generate_exhaustive_configs()


@register_template_heuristic(
    mm_template.uid, "cuda", register=torch.version.hip is None, op_name="scaled_mm"
)
class CUDAScaledMMTemplateConfigHeuristic(ScaledMMConfigMixin, CUDAConfigHeuristic):
    """Scaled MM template heuristic for CUDA"""

````
- **EN**: Introduces class `CUDAAddmmPersistentTMATemplateConfigHeuristic`, class `CUDABlackwellAddmmPersistentTMATemplateConfigHeuristic`, function `__init__`, class `CUDAScaledMMTemplateConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`CUDAAddmmPersistentTMATemplateConfigHeuristic`、类`CUDABlackwellAddmmPersistentTMATemplateConfigHeuristic`、函数`__init__`、类`CUDAScaledMMTemplateConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 2665-2700 / 第 2665-2700 行
````python
    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use scaled_mm_configs
        self.mm_configs = self.scaled_mm_configs

    def _filter_configs(self, configs: list[BaseConfig]) -> list[BaseConfig]:
        configs = [c for c in configs if c.block_k >= 32]
        return super()._filter_configs(configs)


@register_template_heuristic(
    scaled_mm_device_tma_epilogue_scaling_template.uid,
    "cuda",
    register=torch.version.hip is None,
    op_name="scaled_mm",
)
class CUDAScaledTMAEpilogueScalingTemplateConfigHeuristic(
    ScaledTMAConfigMixin, CUDAConfigHeuristic
):
    """Scaled TMA template heuristic for CUDA: epilogue scaling variants (TensorWise, RowWise)"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use scaled_persistent_mm_configs for TMA
        self.mm_configs = self.scaled_persistent_mm_configs


@register_template_heuristic(
    scaled_mm_device_tma_main_loop_scaling_template.uid,
    "cuda",
    register=torch.version.hip is None,
    op_name="scaled_mm",
)
class CUDAScaledTMAMainLoopScalingTemplateConfigHeuristic(
    ScaledTMAConfigMixin, CUDAConfigHeuristic
):
````
- **EN**: Introduces function `__init__`, function `_filter_configs`, class `CUDAScaledTMAEpilogueScalingTemplateConfigHeuristic`, function `__init__`, class `CUDAScaledTMAMainLoopScalingTemplateConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`__init__`、函数`_filter_configs`、类`CUDAScaledTMAEpilogueScalingTemplateConfigHeuristic`、函数`__init__`、类`CUDAScaledTMAMainLoopScalingTemplateConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 2701-2736 / 第 2701-2736 行
````python
    """
    Scaled TMA template heuristic for CUDA:
        main loop scaling variants (BlockWise1x128, BlockWise1x32, BlockWise1x16, BlockWise128x128)
    """

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use scaled_persistent_mm_configs for TMA
        self.mm_configs = self.scaled_persistent_mm_configs

    def _get_template_configs_impl(
        self,
        kernel_inputs: KernelInputs,
        op_name: str,
        **kwargs,
    ) -> Generator[dict[str, Any], None, None]:
        """
        Generate main loop scaling kernel inputs.
        """
        mat_a, mat_b, scale_a, scale_b = kernel_inputs._input_nodes
        scale_a_size, scale_b_size = scale_a.get_size(), scale_b.get_size()

        scale_option_a, scale_option_b = get_scaling_options(
            mat_a, mat_b, scale_a_size, scale_b_size
        )
        tile_size_a = get_tile_size(scale_option_a)
        tile_size_b = get_tile_size(scale_option_b)

        # Get base scaled MM template configs from superclass
        for template_kwargs in super()._get_template_configs_impl(
            kernel_inputs,
            op_name,
            **kwargs,
        ):
            # Add scaling-specific options for main loop scaling variants

````
- **EN**: Introduces function `__init__`, function `_get_template_configs_impl`. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `kernel_inputs`, `op_name`, `tile_size_a`, and `tile_size_b`.
- **CN**: 这里定义了函数`__init__`、函数`_get_template_configs_impl`。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `kernel_inputs`、`op_name`、`tile_size_a`、`tile_size_b` 等值。

### Lines 2737-2772 / 第 2737-2772 行
````python
            # Inductor templates require compile-time constants passed in as tl.constexpr values.
            # In cases in which the block size (BLOCK_*) is smaller than the tile size (128, 32, 16),
            # scales must be broadcasted to BLOCK_* (rather than to a tile_sizextile_size chunk).

            template_kwargs["TILE_SIZE_A"] = tile_size_a
            template_kwargs["TILE_SIZE_B"] = tile_size_b

            template_kwargs["MIN_BLOCK_TILE_AM"] = min(
                template_kwargs["BLOCK_M"], tile_size_a
            )
            template_kwargs["MIN_BLOCK_TILE_AK"] = min(
                template_kwargs["BLOCK_K"], tile_size_a
            )
            template_kwargs["MIN_BLOCK_TILE_BK"] = min(
                template_kwargs["BLOCK_K"], tile_size_b
            )
            template_kwargs["MIN_BLOCK_TILE_BN"] = min(
                template_kwargs["BLOCK_N"], tile_size_b
            )

            yield template_kwargs


@register_template_heuristic(
    # regular Blackwell MM template + scaling epilogue from ScaledMMConfigMixin
    blackwell_ws_persistent_device_tma_mm_template.uid,
    "cuda",
    register=torch.version.hip is None,
    op_name="scaled_mm",
)
class CUDAScaledBlackwellTMATemplateConfigHeuristic(
    ScaledBlackwellTMAConfigMixin, CUDAConfigHeuristic
):
    """Scaled Blackwell TMA template heuristic for CUDA"""

    def __init__(self) -> None:
````
- **EN**: Introduces class `CUDAScaledBlackwellTMATemplateConfigHeuristic`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`CUDAScaledBlackwellTMATemplateConfigHeuristic`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 2773-2808 / 第 2773-2808 行
````python
        super().__init__()
        # TODO: Tune scaled_persistent_mm_configs for Blackwell
        self.mm_configs = self.blackwell_persistent_addmm_configs


@register_template_heuristic(
    mm_plus_mm_template.uid,
    "cuda",
    register=torch.version.hip is None,
)
class CUDAMMPlusMMTemplateConfigHeuristic(
    MMPlusMMTemplateConfigMixin, CUDAConfigHeuristic
):
    """MM Plus MM template heuristic for CUDA"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use mm_plus_mm_configs
        self.mm_configs = self.mm_plus_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.mm_plus_mm_configs


@register_template_heuristic(
    mm_template.uid,
    "cuda",
    register=torch.version.hip is None,
    op_name="int_mm",
)
class CUDAInt8MMTemplateConfigHeuristic(INT8MMTemplateConfigMixin, CUDAConfigHeuristic):
    """Int8 MM template heuristic for CUDA"""

    def __init__(self) -> None:
````
- **EN**: Introduces class `CUDAMMPlusMMTemplateConfigHeuristic`, function `__init__`, class `CUDAInt8MMTemplateConfigHeuristic`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了类`CUDAMMPlusMMTemplateConfigHeuristic`、函数`__init__`、类`CUDAInt8MMTemplateConfigHeuristic`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 2809-2844 / 第 2809-2844 行
````python
        super().__init__()
        # Override mm_configs to use int8_mm_configs
        self.mm_configs = self.int8_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.int8_mm_configs


# ROCm template-specific classes


@register_template_heuristic(
    mm_template.uid,
    "cuda",
    register=torch.version.hip is not None,
)
@register_template_heuristic(
    bmm_template.uid,
    "cuda",
    register=torch.version.hip is not None,
)
class ROCmMMTemplateConfigHeuristic(MMTemplateConfigMixin, ROCmConfigHeuristic):
    """Standard MM template heuristic for ROCm"""


# TODO(coconutruben): replace with template.name once templates are importable
@register_template_heuristic(
    mm_template.uid, "cuda", register=torch.version.hip is not None, op_name="addmm"
)
# TODO(coconutruben): replace with template.name once templates are importable
@register_template_heuristic(
    bmm_template.uid, "cuda", register=torch.version.hip is not None, op_name="baddbmm"
)
class ROCmAddMMTemplateConfigHeuristic(AddMMConfigMixin, ROCmMMTemplateConfigHeuristic):
````
- **EN**: Introduces class `ROCmMMTemplateConfigHeuristic`, class `ROCmAddMMTemplateConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`ROCmMMTemplateConfigHeuristic`、类`ROCmAddMMTemplateConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 2845-2880 / 第 2845-2880 行
````python
    """Addmm specific mixin for ROCm"""


@register_template_heuristic(
    persistent_mm_template.uid,
    "cuda",
    register=torch.version.hip is not None,
    op_name="addmm",
)
class ROCmAddMMPersistentTemplateConfigHeuristic(
    AddMMConfigMixin, PersistentMMTemplateConfigHeuristic
):
    """Addmm specific mixin for persistent MM on ROCm"""


# TODO(coconutruben): deprecate once autoheuristic is deprecated
@register_template_heuristic("mm-ah", "cuda", register=torch.version.hip is not None)
class ROCmMMAHTemplateConfigHeuristic(MMTemplateConfigMixin, ROCmConfigHeuristic):
    """Standard MM template heuristic for ROCm using the extra mm configs only (for autoheuristic)"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use scaled_mm_configs
        self.mm_configs = self.extra_mm_configs
        self.exhaustive_configs = self.extra_mm_configs


@register_template_heuristic(
    mm_template.uid,
    "cuda",
    register=torch.version.hip is not None,
    op_name="scaled_mm",
)
class ROCmScaledMMTemplateConfigHeuristic(ScaledMMConfigMixin, ROCmConfigHeuristic):
    """Scaled MM template heuristic for ROCm (non-TMA)"""

````
- **EN**: Introduces class `ROCmAddMMPersistentTemplateConfigHeuristic`, class `ROCmMMAHTemplateConfigHeuristic`, function `__init__`, class `ROCmScaledMMTemplateConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`ROCmAddMMPersistentTemplateConfigHeuristic`、类`ROCmMMAHTemplateConfigHeuristic`、函数`__init__`、类`ROCmScaledMMTemplateConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 2881-2916 / 第 2881-2916 行
````python
    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use scaled_mm_configs
        self.mm_configs = self.scaled_mm_configs

    def _filter_configs(self, configs: list[BaseConfig]) -> list[BaseConfig]:
        configs = [c for c in configs if c.block_k >= 32]
        return super()._filter_configs(configs)


@register_template_heuristic(
    mm_template.uid,
    "cuda",
    register=torch.version.hip is not None,
    op_name="int_mm",
)
class ROCmInt8MMTemplateConfigHeuristic(INT8MMTemplateConfigMixin, ROCmConfigHeuristic):
    """Int8 MM template heuristic for ROCm"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use int8_mm_configs
        self.mm_configs = self.int8_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.int8_mm_configs


@register_template_heuristic(
    mm_plus_mm_template.uid,
    "cuda",
    register=torch.version.hip is not None,
)
class ROCmMMPlusMMTemplateConfigHeuristic(
````
- **EN**: Introduces function `__init__`, function `_filter_configs`, class `ROCmInt8MMTemplateConfigHeuristic`, function `__init__`, class `ROCmMMPlusMMTemplateConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`__init__`、函数`_filter_configs`、类`ROCmInt8MMTemplateConfigHeuristic`、函数`__init__`、类`ROCmMMPlusMMTemplateConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 2917-2952 / 第 2917-2952 行
````python
    MMPlusMMTemplateConfigMixin, ROCmConfigHeuristic
):
    """MM Plus MM template heuristic for ROCm"""

    def __init__(self) -> None:
        super().__init__()
        # self.default_num_stages is used to make sure all configs have that in ROCm land
        # for mm_plus_mm, we actually just want stages = 1, as pipelining brings no benefits
        self.default_num_stages = 1
        # Override mm_configs to use mm_plus_mm_configs
        self.mm_configs = self.mm_plus_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.mm_plus_mm_configs


# CPU template-specific classes


@register_template_heuristic(mm_template.uid, "cpu")
@register_template_heuristic(bmm_template.uid, "cpu")
class CPUMMTemplateConfigHeuristic(MMTemplateConfigMixin, CPUConfigHeuristic):
    """Standard MM template heuristic for CPU"""


@register_template_heuristic(mm_template.uid, "cpu", op_name="addmm")
@register_template_heuristic(bmm_template.uid, "cpu", op_name="baddbmm")
class CPUAddmmTemplateConfigHeuristic(AddMMConfigMixin, CPUMMTemplateConfigHeuristic):
    """Addmm specific mixin for CPU"""


@register_template_heuristic(mm_template.uid, "cpu", op_name="scaled_mm")
class CPUScaledMMTemplateConfigHeuristic(ScaledMMConfigMixin, CPUConfigHeuristic):
    """Scaled MM template heuristic for CPU (non-TMA)"""
````
- **EN**: Introduces function `__init__`, class `CPUMMTemplateConfigHeuristic`, class `CPUAddmmTemplateConfigHeuristic`, class `CPUScaledMMTemplateConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`__init__`、类`CPUMMTemplateConfigHeuristic`、类`CPUAddmmTemplateConfigHeuristic`、类`CPUScaledMMTemplateConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 2953-2988 / 第 2953-2988 行
````python

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use scaled_mm_configs
        self.mm_configs = self.scaled_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.scaled_mm_configs


@register_template_heuristic(mm_template.uid, "cpu", op_name="int_mm")
class CPUInt8MMTemplateConfigHeuristic(INT8MMTemplateConfigMixin, CPUConfigHeuristic):
    """Int8 MM template heuristic for CPU"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use int8_mm_configs
        self.mm_configs = self.int8_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.int8_mm_configs


@register_template_heuristic(mm_plus_mm_template.uid, "cpu")
class CPUMMPlusMMTemplateConfigHeuristic(
    MMPlusMMTemplateConfigMixin, CPUConfigHeuristic
):
    """MM Plus MM template heuristic for CPU"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use mm_plus_mm_configs
````
- **EN**: Introduces function `__init__`, class `CPUInt8MMTemplateConfigHeuristic`, function `__init__`, class `CPUMMPlusMMTemplateConfigHeuristic`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`__init__`、类`CPUInt8MMTemplateConfigHeuristic`、函数`__init__`、类`CPUMMPlusMMTemplateConfigHeuristic`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 2989-3024 / 第 2989-3024 行
````python
        self.mm_configs = self.mm_plus_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.mm_plus_mm_configs


# XPU template-specific classes


@register_template_heuristic(mm_template.uid, "xpu")
@register_template_heuristic(bmm_template.uid, "xpu")
class XPUMMTemplateConfigHeuristic(MMTemplateConfigMixin, XPUConfigHeuristic):
    """Standard MM template heuristic for XPU"""

    def __init__(self) -> None:
        super().__init__()

        # TODO(etaf): Design proper exhaustive search space for XPU.
        self.exhaustive_configs = self.mm_configs


@register_template_heuristic(mm_template.uid, "xpu", op_name="addmm")
@register_template_heuristic(bmm_template.uid, "xpu", op_name="baddbmm")
class XPUAddmmTemplateConfigHeuristic(AddMMConfigMixin, XPUMMTemplateConfigHeuristic):
    """Addmm specific mixin for XPU"""


@register_template_heuristic(
    persistent_tma_mm_template.uid,
    "xpu",
)
class XPUPersistentTMATemplateConfigHeuristic(
    TMATemplateConfigMixin, XPUConfigHeuristic
):
````
- **EN**: Introduces class `XPUMMTemplateConfigHeuristic`, function `__init__`, class `XPUAddmmTemplateConfigHeuristic`, class `XPUPersistentTMATemplateConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`XPUMMTemplateConfigHeuristic`、函数`__init__`、类`XPUAddmmTemplateConfigHeuristic`、类`XPUPersistentTMATemplateConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 3025-3060 / 第 3025-3060 行
````python
    """Persistent TMA template heuristic for XPU"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use persistent_mm_configs
        self.mm_configs = self.persistent_mm_configs


@register_template_heuristic(persistent_tma_mm_template.uid, "xpu", op_name="addmm")
class XPUAddmmPersistentTMATemplateConfigHeuristic(
    AddMMConfigMixin, XPUPersistentTMATemplateConfigHeuristic
):
    """Addmm specific mixin for XPU"""


@register_template_heuristic(mm_template.uid, "xpu", op_name="scaled_mm")
class XPUScaledMMTemplateConfigHeuristic(ScaledMMConfigMixin, XPUConfigHeuristic):
    """Scaled MM template heuristic for XPU (non-TMA)"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use scaled_mm_configs
        self.mm_configs = self.scaled_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.scaled_mm_configs


@register_template_heuristic(mm_template.uid, "xpu", op_name="int_mm")
class XPUInt8MMTemplateConfigHeuristic(INT8MMTemplateConfigMixin, XPUConfigHeuristic):
    """Int8 MM template heuristic for XPU"""

    def __init__(self) -> None:
        super().__init__()
````
- **EN**: Introduces function `__init__`, class `XPUAddmmPersistentTMATemplateConfigHeuristic`, class `XPUScaledMMTemplateConfigHeuristic`, function `__init__`, class `XPUInt8MMTemplateConfigHeuristic`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`__init__`、类`XPUAddmmPersistentTMATemplateConfigHeuristic`、类`XPUScaledMMTemplateConfigHeuristic`、函数`__init__`、类`XPUInt8MMTemplateConfigHeuristic`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 3061-3096 / 第 3061-3096 行
````python
        # Override mm_configs to use int8_mm_configs
        self.mm_configs = self.int8_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.int8_mm_configs


@register_template_heuristic(mm_plus_mm_template.uid, "xpu")
class XPUMMPlusMMTemplateConfigHeuristic(
    MMPlusMMTemplateConfigMixin, XPUConfigHeuristic
):
    """MM Plus MM template heuristic for XPU"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use mm_plus_mm_configs
        self.mm_configs = self.mm_plus_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.mm_plus_mm_configs


# MTIA template-specific classes


@register_template_heuristic(mm_template.uid, "mtia")
@register_template_heuristic(bmm_template.uid, "mtia")
class MTIAMMTemplateConfigHeuristic(MMTemplateConfigMixin, MTIAConfigHeuristic):
    """Standard MM template heuristic for MTIA"""


@register_template_heuristic(mm_template.uid, "mtia", op_name="addmm")
````
- **EN**: Introduces class `XPUMMPlusMMTemplateConfigHeuristic`, function `__init__`, class `MTIAMMTemplateConfigHeuristic`. Applies decorators to register behavior or alter how the following definition is constructed. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`XPUMMPlusMMTemplateConfigHeuristic`、函数`__init__`、类`MTIAMMTemplateConfigHeuristic`。使用装饰器来注册行为，或改变后续定义的构造方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 3097-3132 / 第 3097-3132 行
````python
@register_template_heuristic(bmm_template.uid, "mtia", op_name="baddbmm")
class MTIAAddMMTemplateConfigHeuristic(AddMMConfigMixin, MTIAMMTemplateConfigHeuristic):
    """Addmm specific mixin for MTIA"""


@register_template_heuristic(mm_template.uid, "mtia", op_name="scaled_mm")
class MTIAScaledMMTemplateConfigHeuristic(ScaledMMConfigMixin, MTIAConfigHeuristic):
    """Scaled MM template heuristic for MTIA (non-TMA)"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use scaled_mm_configs
        self.mm_configs = self.scaled_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.scaled_mm_configs


@register_template_heuristic(mm_template.uid, "mtia", op_name="int_mm")
class MTIAInt8MMTemplateConfigHeuristic(INT8MMTemplateConfigMixin, MTIAConfigHeuristic):
    """Int8 MM template heuristic for MTIA"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use int8_mm_configs
        self.mm_configs = self.int8_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.int8_mm_configs


@register_template_heuristic(mm_plus_mm_template.uid, "mtia")
````
- **EN**: Introduces class `MTIAAddMMTemplateConfigHeuristic`, class `MTIAScaledMMTemplateConfigHeuristic`, function `__init__`, class `MTIAInt8MMTemplateConfigHeuristic`, function `__init__`. Applies decorators to register behavior or alter how the following definition is constructed. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了类`MTIAAddMMTemplateConfigHeuristic`、类`MTIAScaledMMTemplateConfigHeuristic`、函数`__init__`、类`MTIAInt8MMTemplateConfigHeuristic`、函数`__init__`。使用装饰器来注册行为，或改变后续定义的构造方式。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 3133-3146 / 第 3133-3146 行
````python
class MTIAMMPlusMMTemplateConfigHeuristic(
    MMPlusMMTemplateConfigMixin, MTIAConfigHeuristic
):
    """MM Plus MM template heuristic for MTIA"""

    def __init__(self) -> None:
        super().__init__()
        # Override mm_configs to use mm_plus_mm_configs
        self.mm_configs = self.mm_plus_mm_configs
        # NOTE: overriding exhaustive configs here to be the same as mm_configs
        # as we haven't validated exhaustive support here yet
        # TODO(coconutruben): remove this once we have validated exhaustive support
        # for scaled_mm
        self.exhaustive_configs = self.mm_plus_mm_configs
````
- **EN**: Introduces class `MTIAMMPlusMMTemplateConfigHeuristic`, function `__init__`.
- **CN**: 这里定义了类`MTIAMMPlusMMTemplateConfigHeuristic`、函数`__init__`。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
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
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `itertools`, `math`, `os`, `functools`, `threading`, `typing`, `collections.abc`
- **Third-party / 第三方**: `sympy`, `triton`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.template_heuristics.triton_addmm`, `torch.utils._ordered_set`, `torch.utils._sympy.functions`, `torch.utils._triton`, `..`, `..kernel.bmm`, `..kernel.mm`, `..kernel.mm_plus_mm`, `..kernel_inputs`, `..utils`, `..virtualized`, `.gemm`, `.registry`, `torch._inductor.runtime.triton_compat`, `..runtime.runtime_utils`, `..lowering`, `..kernel.mm_common`
