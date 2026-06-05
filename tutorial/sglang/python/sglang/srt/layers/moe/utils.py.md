# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `MoeA2ABackend`, `MoeRunnerBackend`, `DeepEPMode`, and `DeepEPOutputDtype` and connects them to backend-specific paths such as `Triton`, `CUTLASS`, `FlashInfer`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `MoeA2ABackend`、`MoeRunnerBackend`、`DeepEPMode` 以及 `DeepEPOutputDtype` 等符号，并把这些符号连接到 `Triton`、`CUTLASS`、`FlashInfer` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import logging
import os
from contextlib import contextmanager
from enum import Enum, IntEnum
from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.distributed.parallel_state import get_moe_expert_parallel_world_size
from sglang.srt.environ import envs
from sglang.srt.layers.dp_attention import (
    get_attention_dp_size,
    is_dp_attention_enabled,
)
from sglang.srt.utils import is_npu

_is_npu = is_npu()

if TYPE_CHECKING:
    from sglang.srt.server_args import ServerArgs

from sglang.srt.server_args import get_global_server_args

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `os`, `contextlib.contextmanager`, `enum.Enum`, and `enum.IntEnum`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_npu` and `logger` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`os`、`contextlib.contextmanager`、`enum.Enum` 以及 `enum.IntEnum`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_npu` 和 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 29-40: Class `MoeA2ABackend` declaration and shared state
```python
class MoeA2ABackend(Enum):

    NONE = "none"
    DEEPEP = "deepep"
    MOONCAKE = "mooncake"
    NIXL = "nixl"
    MORI = "mori"
    ASCEND_FUSEEP = "ascend_fuseep"
    FLASHINFER = "flashinfer"
    MEGAMOE = "megamoe"
    CUSTOMIZED = "customized"
```
**EN:** This block introduces class `MoeA2ABackend` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `MoeA2ABackend`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 41-49: Internal helper `MoeA2ABackend._missing_`
```python
    @classmethod
    def _missing_(cls, value):
        if value is None:
            return cls.NONE
        for member in cls:
            if value == member.value:
                return member
        raise ValueError(f"No {cls.__name__} member for value {value}")
```
**EN:** This block defines `MoeA2ABackend._missing_` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeA2ABackend._missing_`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 50-52: `MoeA2ABackend.is_none` predicate for is none
```python
    def is_none(self):
        return self == MoeA2ABackend.NONE
```
**EN:** This block defines `MoeA2ABackend.is_none` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeA2ABackend.is_none`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 53-55: `MoeA2ABackend.is_deepep` predicate for is deepep
```python
    def is_deepep(self):
        return self == MoeA2ABackend.DEEPEP
```
**EN:** This block defines `MoeA2ABackend.is_deepep` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeA2ABackend.is_deepep`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 56-58: `MoeA2ABackend.is_mooncake` predicate for is mooncake
```python
    def is_mooncake(self):
        return self == MoeA2ABackend.MOONCAKE
```
**EN:** This block defines `MoeA2ABackend.is_mooncake` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeA2ABackend.is_mooncake`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 59-61: `MoeA2ABackend.is_nixl` predicate for is nixl
```python
    def is_nixl(self):
        return self == MoeA2ABackend.NIXL
```
**EN:** This block defines `MoeA2ABackend.is_nixl` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeA2ABackend.is_nixl`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 62-64: `MoeA2ABackend.is_flashinfer` predicate for is FlashInfer
```python
    def is_flashinfer(self):
        return self == MoeA2ABackend.FLASHINFER
```
**EN:** This block defines `MoeA2ABackend.is_flashinfer` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeA2ABackend.is_flashinfer`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 65-67: `MoeA2ABackend.is_ascend_fuseep` predicate for is ascend fuseep
```python
    def is_ascend_fuseep(self):
        return self == MoeA2ABackend.ASCEND_FUSEEP
```
**EN:** This block defines `MoeA2ABackend.is_ascend_fuseep` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeA2ABackend.is_ascend_fuseep`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 68-70: `MoeA2ABackend.is_mori` predicate for is mori
```python
    def is_mori(self):
        return self == MoeA2ABackend.MORI
```
**EN:** This block defines `MoeA2ABackend.is_mori` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeA2ABackend.is_mori`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 71-73: `MoeA2ABackend.is_megamoe` predicate for is megamoe
```python
    def is_megamoe(self):
        return self == MoeA2ABackend.MEGAMOE
```
**EN:** This block defines `MoeA2ABackend.is_megamoe` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeA2ABackend.is_megamoe`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 74-76: `MoeA2ABackend.is_customized` predicate for is customized
```python
    def is_customized(self):
        return self == MoeA2ABackend.CUSTOMIZED
```
**EN:** This block defines `MoeA2ABackend.is_customized` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeA2ABackend.is_customized`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 77-86: Function `MoeA2ABackend.supports_aiter` and its core logic
```python
    def supports_aiter(self) -> bool:
        return self in (
            MoeA2ABackend.NONE,
            MoeA2ABackend.DEEPEP,
            MoeA2ABackend.MOONCAKE,
            MoeA2ABackend.NIXL,
            MoeA2ABackend.MORI,
        )
```
**EN:** This block defines `MoeA2ABackend.supports_aiter` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeA2ABackend.supports_aiter`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 87-101: Class `MoeRunnerBackend` declaration and shared state
```python
class MoeRunnerBackend(Enum):

    AUTO = "auto"
    DEEP_GEMM = "deep_gemm"
    TRITON = "triton"
    TRITON_KERNELS = "triton_kernel"
    FLASHINFER_TRTLLM = "flashinfer_trtllm"
    FLASHINFER_TRTLLM_ROUTED = "flashinfer_trtllm_routed"
    FLASHINFER_CUTLASS = "flashinfer_cutlass"
    FLASHINFER_MXFP4 = "flashinfer_mxfp4"
    FLASHINFER_CUTEDSL = "flashinfer_cutedsl"
    CUTLASS = "cutlass"
    MARLIN = "marlin"
    AITER = "aiter"
```
**EN:** This block introduces class `MoeRunnerBackend` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `MoeRunnerBackend`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 102-104: `MoeRunnerBackend.is_auto` predicate for is auto
```python
    def is_auto(self):
        return self == MoeRunnerBackend.AUTO
```
**EN:** This block defines `MoeRunnerBackend.is_auto` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_auto`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 105-107: `MoeRunnerBackend.is_deep_gemm` predicate for is deep runtime integration GEMM
```python
    def is_deep_gemm(self):
        return self == MoeRunnerBackend.DEEP_GEMM
```
**EN:** This block defines `MoeRunnerBackend.is_deep_gemm` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_deep_gemm`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 108-110: `MoeRunnerBackend.is_triton` predicate for is Triton
```python
    def is_triton(self):
        return self == MoeRunnerBackend.TRITON
```
**EN:** This block defines `MoeRunnerBackend.is_triton` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_triton`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 111-113: `MoeRunnerBackend.is_triton_kernels` predicate for is Triton kernels
```python
    def is_triton_kernels(self):
        return self == MoeRunnerBackend.TRITON_KERNELS
```
**EN:** This block defines `MoeRunnerBackend.is_triton_kernels` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_triton_kernels`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 114-116: `MoeRunnerBackend.is_flashinfer_trtllm` predicate for is FlashInfer trtllm
```python
    def is_flashinfer_trtllm(self):
        return self == MoeRunnerBackend.FLASHINFER_TRTLLM
```
**EN:** This block defines `MoeRunnerBackend.is_flashinfer_trtllm` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_flashinfer_trtllm`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 117-119: `MoeRunnerBackend.is_flashinfer_trtllm_routed` predicate for is FlashInfer trtllm routed
```python
    def is_flashinfer_trtllm_routed(self):
        return self == MoeRunnerBackend.FLASHINFER_TRTLLM_ROUTED
```
**EN:** This block defines `MoeRunnerBackend.is_flashinfer_trtllm_routed` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_flashinfer_trtllm_routed`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 120-122: `MoeRunnerBackend.is_flashinfer_cutlass` predicate for is FlashInfer CUTLASS
```python
    def is_flashinfer_cutlass(self):
        return self == MoeRunnerBackend.FLASHINFER_CUTLASS
```
**EN:** This block defines `MoeRunnerBackend.is_flashinfer_cutlass` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_flashinfer_cutlass`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 123-125: `MoeRunnerBackend.is_flashinfer_cutedsl` predicate for is FlashInfer cutedsl
```python
    def is_flashinfer_cutedsl(self):
        return self == MoeRunnerBackend.FLASHINFER_CUTEDSL
```
**EN:** This block defines `MoeRunnerBackend.is_flashinfer_cutedsl` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_flashinfer_cutedsl`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 126-128: `MoeRunnerBackend.is_flashinfer_mxfp4` predicate for is FlashInfer mxfp 4
```python
    def is_flashinfer_mxfp4(self):
        return self == MoeRunnerBackend.FLASHINFER_MXFP4
```
**EN:** This block defines `MoeRunnerBackend.is_flashinfer_mxfp4` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_flashinfer_mxfp4`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 129-131: `MoeRunnerBackend.is_cutlass` predicate for is CUTLASS
```python
    def is_cutlass(self):
        return self == MoeRunnerBackend.CUTLASS
```
**EN:** This block defines `MoeRunnerBackend.is_cutlass` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_cutlass`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 132-134: `MoeRunnerBackend.is_marlin` predicate for is marlin
```python
    def is_marlin(self):
        return self == MoeRunnerBackend.MARLIN
```
**EN:** This block defines `MoeRunnerBackend.is_marlin` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_marlin`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 135-138: `MoeRunnerBackend.is_aiter` predicate for is aiter
```python
    def is_aiter(self):
        return self == MoeRunnerBackend.AITER
```
**EN:** This block defines `MoeRunnerBackend.is_aiter` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MoeRunnerBackend.is_aiter`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 139-144: Class `DeepEPMode` declaration and shared state
```python
class DeepEPMode(Enum):

    NORMAL = "normal"
    LOW_LATENCY = "low_latency"
    AUTO = "auto"
```
**EN:** This block introduces class `DeepEPMode` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DeepEPMode`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 145-147: Function `DeepEPMode.enable_normal` and its core logic
```python
    def enable_normal(self) -> bool:
        return self in [DeepEPMode.NORMAL, DeepEPMode.AUTO]
```
**EN:** This block defines `DeepEPMode.enable_normal` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMode.enable_normal`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 148-150: Function `DeepEPMode.enable_low_latency` and its core logic
```python
    def enable_low_latency(self) -> bool:
        return self in [DeepEPMode.LOW_LATENCY, DeepEPMode.AUTO]
```
**EN:** This block defines `DeepEPMode.enable_low_latency` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMode.enable_low_latency`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 151-159: Function `DeepEPMode.resolve` and its core logic
```python
    def resolve(self, is_extend_in_batch: bool) -> DeepEPMode:
        if self != DeepEPMode.AUTO:
            return self

        if is_extend_in_batch:
            return DeepEPMode.NORMAL
        else:
            return DeepEPMode.LOW_LATENCY
```
**EN:** This block defines `DeepEPMode.resolve` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMode.resolve`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 160-162: `DeepEPMode.is_normal` predicate for is normal
```python
    def is_normal(self) -> bool:
        return self == DeepEPMode.NORMAL
```
**EN:** This block defines `DeepEPMode.is_normal` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMode.is_normal`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 163-165: `DeepEPMode.is_low_latency` predicate for is low latency
```python
    def is_low_latency(self) -> bool:
        return self == DeepEPMode.LOW_LATENCY
```
**EN:** This block defines `DeepEPMode.is_low_latency` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMode.is_low_latency`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 166-169: `DeepEPMode.is_auto` predicate for is auto
```python
    def is_auto(self) -> bool:
        return self == DeepEPMode.AUTO
```
**EN:** This block defines `DeepEPMode.is_auto` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMode.is_auto`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 170-185: Class `DeepEPOutputDtype` declaration and shared state
```python
class DeepEPOutputDtype(Enum):
    """
    Describes the dispatch output data type for DeepEP.

    - BF16: dispatch hidden states in bf16
    - FP8: dispatch hidden states in fp8
    - INT8: dispatch hidden states in int8
    - NVFP4: dispatch hidden states in nvfp4
    """

    BF16 = "bf16"
    FP8 = "fp8"
    INT8 = "int8"
    NVFP4 = "nvfp4"
```
**EN:** This block introduces class `DeepEPOutputDtype` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Describes the dispatch output data type for DeepEP.
**CN:** 该代码块引入类 `DeepEPOutputDtype`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 186-239: `get_deepep_output_dtype` getter for deepep output dtype
```python
def get_deepep_output_dtype(self) -> DeepEPOutputDtype:
    """
    Automatically choose the dispatch output dtype for DeepEP.

    The decision follows several checks in priority order:
    0. Parse server argument.
    1. Parse deprecated environment variables.
    2. If quant_config contains input_global_scale → NVFP4 path.
    3. Parse quant config
    4. If flashinfer_cutedsl or is_cutlass backend is active → BF16 (it quantizes hidden_states internally).
    5. Otherwise default for NPU → BF16 (the default for NPU).
    6. Otherwise → FP8 (the default for most models like DeepSeek-V3).
    """

    # 0. Parse server argument.
    server_args = get_global_server_args()
    if server_args and server_args.deepep_dispatcher_output_dtype != "auto":
        return DeepEPOutputDtype(server_args.deepep_dispatcher_output_dtype)

    # 1. Parse deprecated environment variables.
    if envs.SGLANG_DEEPEP_BF16_DISPATCH.get():
        logger.warning_once(
            "Warning: The env variable SGLANG_DEEPEP_BF16_DISPATCH deprecated "
            "and will be removed in future releases. Please use a new "
            "`--deepep-dispatcher-output-dtype bf16` argument instead."
        )
        return DeepEPOutputDtype.BF16

    # 2. NVFP4 is detected inside dispatch_a / _dispatch_core via quant_config; no need to infer here.
    if self.quant_config is not None:
        input_global_scale = self.quant_config.get("input_global_scale", None)
        if input_global_scale is not None:
            return DeepEPOutputDtype.NVFP4

        # 3. Parse quant config to determine the output dtype of dispatcher
        dispatcher_output_dtype = self.quant_config.get("dispatcher_output_dtype", None)
        if dispatcher_output_dtype is not None:
            return DeepEPOutputDtype(dispatcher_output_dtype)

    # 4. flashinfer_cutedsl and is_cutlass expects BF16 dispatch
    if (
        get_moe_runner_backend().is_flashinfer_cutedsl()
        or get_moe_runner_backend().is_cutlass()
    ):
        return DeepEPOutputDtype.BF16

    # 5. Default on NPU → BF16
    if _is_npu:
        return DeepEPOutputDtype.BF16

    # 6. Default → FP8
    return DeepEPOutputDtype.FP8
```
**EN:** This block defines `get_deepep_output_dtype` and contains the main logic for this step. It mainly invokes `get_global_server_args`, `envs.SGLANG_DEEPEP_BF16_DISPATCH.get`, `DeepEPOutputDtype`, `logger.warning_once`, and `self.quant_config.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `server_args`, `input_global_scale`, and `dispatcher_output_dtype` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_deepep_output_dtype`，并承载这一阶段的核心逻辑。 它主要调用 `get_global_server_args`、`envs.SGLANG_DEEPEP_BF16_DISPATCH.get`、`DeepEPOutputDtype`、`logger.warning_once` 以及 `self.quant_config.get`，说明该流程会编排底层辅助函数或计算内核。 像 `server_args`、`input_global_scale` 以及 `dispatcher_output_dtype` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 240-252: Module constants and shared configuration
```python
MOE_A2A_BACKEND: Optional[MoeA2ABackend] = None
MOE_RUNNER_BACKEND: Optional[MoeRunnerBackend] = None
SPECULATIVE_MOE_RUNNER_BACKEND: Optional[MoeRunnerBackend] = None
SPECULATIVE_MOE_A2A_BACKEND: Optional[MoeA2ABackend] = None
DEEPEP_MODE: Optional[DeepEPMode] = None
IS_TBO_ENABLED: Optional[bool] = None
IS_SBO_ENABLED: Optional[bool] = None
TBO_TOKEN_DISTRIBUTION_THRESHOLD: Optional[float] = None
DEEPEP_CONFIG: Optional[str] = None
DISABLE_FLASHINFER_CUTLASS_MOE_FP4_ALLGATHER: Optional[bool] = None
MOE_QUANTIZATION: Optional[str] = None
```
**EN:** This section prepares the module namespace. Shared names such as `MOE_A2A_BACKEND`, `MOE_RUNNER_BACKEND`, `SPECULATIVE_MOE_RUNNER_BACKEND`, `SPECULATIVE_MOE_A2A_BACKEND`, and `DEEPEP_MODE` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `MOE_A2A_BACKEND`、`MOE_RUNNER_BACKEND`、`SPECULATIVE_MOE_RUNNER_BACKEND`、`SPECULATIVE_MOE_A2A_BACKEND` 以及 `DEEPEP_MODE` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 253-288: Function `initialize_moe_config` and its core logic
```python
def initialize_moe_config(server_args: ServerArgs):
    global MOE_A2A_BACKEND
    global MOE_RUNNER_BACKEND
    global SPECULATIVE_MOE_RUNNER_BACKEND
    global SPECULATIVE_MOE_A2A_BACKEND
    global DEEPEP_MODE
    global DEEPEP_CONFIG
    global IS_TBO_ENABLED
    global IS_SBO_ENABLED
    global TBO_TOKEN_DISTRIBUTION_THRESHOLD
    global DISABLE_FLASHINFER_CUTLASS_MOE_FP4_ALLGATHER
    global MOE_QUANTIZATION

    MOE_A2A_BACKEND = MoeA2ABackend(server_args.moe_a2a_backend)
    MOE_RUNNER_BACKEND = MoeRunnerBackend(server_args.moe_runner_backend)
    SPECULATIVE_MOE_RUNNER_BACKEND = (
        MoeRunnerBackend(server_args.speculative_moe_runner_backend)
        if server_args.speculative_moe_runner_backend is not None
        else MOE_RUNNER_BACKEND
    )
    SPECULATIVE_MOE_A2A_BACKEND = (
        MoeA2ABackend(server_args.speculative_moe_a2a_backend)
        if server_args.speculative_moe_a2a_backend is not None
        else MOE_A2A_BACKEND
    )
    DEEPEP_MODE = DeepEPMode(server_args.deepep_mode)
    DEEPEP_CONFIG = server_args.deepep_config or ""
    IS_TBO_ENABLED = server_args.enable_two_batch_overlap
    IS_SBO_ENABLED = server_args.enable_single_batch_overlap
    TBO_TOKEN_DISTRIBUTION_THRESHOLD = server_args.tbo_token_distribution_threshold
    DISABLE_FLASHINFER_CUTLASS_MOE_FP4_ALLGATHER = (
        server_args.disable_flashinfer_cutlass_moe_fp4_allgather
    )
    MOE_QUANTIZATION = server_args.quantization
```
**EN:** This block defines `initialize_moe_config` and contains the main logic for this step. It mainly invokes `MoeA2ABackend`, `MoeRunnerBackend`, and `DeepEPMode`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `MOE_A2A_BACKEND`, `MOE_RUNNER_BACKEND`, `SPECULATIVE_MOE_RUNNER_BACKEND`, `SPECULATIVE_MOE_A2A_BACKEND`, and `DEEPEP_MODE` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `initialize_moe_config`，并承载这一阶段的核心逻辑。 它主要调用 `MoeA2ABackend`、`MoeRunnerBackend` 以及 `DeepEPMode`，说明该流程会编排底层辅助函数或计算内核。 像 `MOE_A2A_BACKEND`、`MOE_RUNNER_BACKEND`、`SPECULATIVE_MOE_RUNNER_BACKEND`、`SPECULATIVE_MOE_A2A_BACKEND` 以及 `DEEPEP_MODE` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 289-295: `get_moe_a2a_backend` getter for Mixture-of-Experts a 2 a backend
```python
def get_moe_a2a_backend() -> MoeA2ABackend:
    global MOE_A2A_BACKEND
    if MOE_A2A_BACKEND is None:
        MOE_A2A_BACKEND = MoeA2ABackend.NONE
    return MOE_A2A_BACKEND
```
**EN:** This block defines `get_moe_a2a_backend` and contains the main logic for this step. Intermediate names such as `MOE_A2A_BACKEND` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_moe_a2a_backend`，并承载这一阶段的核心逻辑。 像 `MOE_A2A_BACKEND` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 296-302: `get_moe_runner_backend` getter for Mixture-of-Experts runner backend
```python
def get_moe_runner_backend() -> MoeRunnerBackend:
    global MOE_RUNNER_BACKEND
    if MOE_RUNNER_BACKEND is None:
        MOE_RUNNER_BACKEND = MoeRunnerBackend.AUTO
    return MOE_RUNNER_BACKEND
```
**EN:** This block defines `get_moe_runner_backend` and contains the main logic for this step. Intermediate names such as `MOE_RUNNER_BACKEND` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_moe_runner_backend`，并承载这一阶段的核心逻辑。 像 `MOE_RUNNER_BACKEND` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 303-312: `get_speculative_moe_runner_backend` getter for speculative Mixture-of-Experts runner backend
```python
def get_speculative_moe_runner_backend() -> MoeRunnerBackend:
    global SPECULATIVE_MOE_RUNNER_BACKEND
    if SPECULATIVE_MOE_RUNNER_BACKEND is None:
        logger.warning(
            "SPECULATIVE_MOE_RUNNER_BACKEND is not initialized, using auto backend"
        )
        SPECULATIVE_MOE_RUNNER_BACKEND = MoeRunnerBackend.AUTO
    return SPECULATIVE_MOE_RUNNER_BACKEND
```
**EN:** This block defines `get_speculative_moe_runner_backend` and contains the main logic for this step. It mainly invokes `logger.warning`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `SPECULATIVE_MOE_RUNNER_BACKEND` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_speculative_moe_runner_backend`，并承载这一阶段的核心逻辑。 它主要调用 `logger.warning`，说明该流程会编排底层辅助函数或计算内核。 像 `SPECULATIVE_MOE_RUNNER_BACKEND` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 313-322: `get_speculative_moe_a2a_backend` getter for speculative Mixture-of-Experts a 2 a backend
```python
def get_speculative_moe_a2a_backend() -> MoeA2ABackend:
    global SPECULATIVE_MOE_A2A_BACKEND
    if SPECULATIVE_MOE_A2A_BACKEND is None:
        logger.warning(
            "SPECULATIVE_MOE_A2A_BACKEND is not initialized, using none backend"
        )
        SPECULATIVE_MOE_A2A_BACKEND = MoeA2ABackend.NONE
    return SPECULATIVE_MOE_A2A_BACKEND
```
**EN:** This block defines `get_speculative_moe_a2a_backend` and contains the main logic for this step. It mainly invokes `logger.warning`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `SPECULATIVE_MOE_A2A_BACKEND` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_speculative_moe_a2a_backend`，并承载这一阶段的核心逻辑。 它主要调用 `logger.warning`，说明该流程会编排底层辅助函数或计算内核。 像 `SPECULATIVE_MOE_A2A_BACKEND` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 323-330: `get_deepep_mode` getter for deepep mode
```python
def get_deepep_mode() -> DeepEPMode:
    global DEEPEP_MODE
    if DEEPEP_MODE is None:
        logger.warning("DEEPEP_MODE is not initialized, using auto mode")
        DEEPEP_MODE = DeepEPMode.AUTO
    return DEEPEP_MODE
```
**EN:** This block defines `get_deepep_mode` and contains the main logic for this step. It mainly invokes `logger.warning`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `DEEPEP_MODE` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_deepep_mode`，并承载这一阶段的核心逻辑。 它主要调用 `logger.warning`，说明该流程会编排底层辅助函数或计算内核。 像 `DEEPEP_MODE` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 331-338: `get_deepep_config` getter for deepep config
```python
def get_deepep_config() -> str:
    global DEEPEP_CONFIG
    if DEEPEP_CONFIG is None:
        logger.warning("DEEPEP_CONFIG is not initialized, using default config")
        DEEPEP_CONFIG = ""
    return DEEPEP_CONFIG
```
**EN:** This block defines `get_deepep_config` and contains the main logic for this step. It mainly invokes `logger.warning`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `DEEPEP_CONFIG` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_deepep_config`，并承载这一阶段的核心逻辑。 它主要调用 `logger.warning`，说明该流程会编排底层辅助函数或计算内核。 像 `DEEPEP_CONFIG` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 339-345: `is_tbo_enabled` predicate for is tbo enabled
```python
def is_tbo_enabled() -> bool:
    global IS_TBO_ENABLED
    if IS_TBO_ENABLED is None:
        IS_TBO_ENABLED = False
    return IS_TBO_ENABLED
```
**EN:** This block defines `is_tbo_enabled` and contains the main logic for this step. Intermediate names such as `IS_TBO_ENABLED` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_tbo_enabled`，并承载这一阶段的核心逻辑。 像 `IS_TBO_ENABLED` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 346-352: `is_sbo_enabled` predicate for is sbo enabled
```python
def is_sbo_enabled() -> bool:
    global IS_SBO_ENABLED
    if IS_SBO_ENABLED is None:
        IS_SBO_ENABLED = False
    return IS_SBO_ENABLED
```
**EN:** This block defines `is_sbo_enabled` and contains the main logic for this step. Intermediate names such as `IS_SBO_ENABLED` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_sbo_enabled`，并承载这一阶段的核心逻辑。 像 `IS_SBO_ENABLED` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 353-358: `is_deepep_class_backend` predicate for is deepep class backend
```python
def is_deepep_class_backend() -> bool:
    """Check if the MoE backend is DeepEP-family (DeepEP, Mooncake, or Mori)."""
    b = get_moe_a2a_backend()
    return b.is_deepep() or b.is_mooncake() or b.is_mori()
```
**EN:** This block defines `is_deepep_class_backend` and contains the main logic for this step. It mainly invokes `get_moe_a2a_backend`, `b.is_deepep`, `b.is_mooncake`, and `b.is_mori`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `b` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_deepep_class_backend`，并承载这一阶段的核心逻辑。 它主要调用 `get_moe_a2a_backend`、`b.is_deepep`、`b.is_mooncake` 以及 `b.is_mori`，说明该流程会编排底层辅助函数或计算内核。 像 `b` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 359-366: `is_flashinfer_cutedsl_v1_path` predicate for is FlashInfer cutedsl v 1 path
```python
def is_flashinfer_cutedsl_v1_path() -> bool:
    """CuteDSL v1 + DeepEP low-latency path (no MoeRunner, no autotune)."""
    return (
        get_moe_runner_backend().is_flashinfer_cutedsl()
        and get_moe_a2a_backend().is_deepep()
    )
```
**EN:** This block defines `is_flashinfer_cutedsl_v1_path` and contains the main logic for this step. It mainly invokes `get_moe_runner_backend.is_flashinfer_cutedsl`, `get_moe_a2a_backend.is_deepep`, `get_moe_runner_backend`, and `get_moe_a2a_backend`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_flashinfer_cutedsl_v1_path`，并承载这一阶段的核心逻辑。 它主要调用 `get_moe_runner_backend.is_flashinfer_cutedsl`、`get_moe_a2a_backend.is_deepep`、`get_moe_runner_backend` 以及 `get_moe_a2a_backend`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 367-376: `get_tbo_token_distribution_threshold` getter for tbo token distribution threshold
```python
def get_tbo_token_distribution_threshold() -> float:
    global TBO_TOKEN_DISTRIBUTION_THRESHOLD
    if TBO_TOKEN_DISTRIBUTION_THRESHOLD is None:
        logger.warning(
            "TBO_TOKEN_DISTRIBUTION_THRESHOLD is not initialized, using 0.48"
        )
        TBO_TOKEN_DISTRIBUTION_THRESHOLD = 0.48
    return TBO_TOKEN_DISTRIBUTION_THRESHOLD
```
**EN:** This block defines `get_tbo_token_distribution_threshold` and contains the main logic for this step. It mainly invokes `logger.warning`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `TBO_TOKEN_DISTRIBUTION_THRESHOLD` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_tbo_token_distribution_threshold`，并承载这一阶段的核心逻辑。 它主要调用 `logger.warning`，说明该流程会编排底层辅助函数或计算内核。 像 `TBO_TOKEN_DISTRIBUTION_THRESHOLD` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 377-387: Function `filter_moe_weight_param_global_expert` and its core logic
```python
def filter_moe_weight_param_global_expert(name, x, num_local_experts):
    """
    Filter out for MoE expert parameters that requires global expert.
    """
    return (
        not getattr(x, "_sglang_require_global_experts", False)
        and x.data.ndim > 0
        and x.data.shape[0] == num_local_experts
    )
```
**EN:** This block defines `filter_moe_weight_param_global_expert` and contains the main logic for this step. It mainly invokes `getattr`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `filter_moe_weight_param_global_expert`，并承载这一阶段的核心逻辑。 它主要调用 `getattr`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 388-401: `should_use_flashinfer_cutlass_moe_fp4_allgather` predicate for should use FlashInfer CUTLASS Mixture-of-Experts fp 4 all-gather
```python
def should_use_flashinfer_cutlass_moe_fp4_allgather():
    """
    Perform FP4 quantize before all-gather for flashinfer cutlass moe to reduce communication cost for high-throughput serving.
    """
    return (
        not DISABLE_FLASHINFER_CUTLASS_MOE_FP4_ALLGATHER
        and get_moe_a2a_backend().is_none()
        and get_moe_runner_backend().is_flashinfer_cutlass()
        and is_dp_attention_enabled()
        and MOE_QUANTIZATION == "modelopt_fp4"
        and get_moe_expert_parallel_world_size() == get_attention_dp_size()
    )
```
**EN:** This block defines `should_use_flashinfer_cutlass_moe_fp4_allgather` and contains the main logic for this step. It mainly invokes `get_moe_a2a_backend.is_none`, `get_moe_runner_backend.is_flashinfer_cutlass`, `is_dp_attention_enabled`, `get_moe_expert_parallel_world_size`, and `get_attention_dp_size`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `should_use_flashinfer_cutlass_moe_fp4_allgather`，并承载这一阶段的核心逻辑。 它主要调用 `get_moe_a2a_backend.is_none`、`get_moe_runner_backend.is_flashinfer_cutlass`、`is_dp_attention_enabled`、`get_moe_expert_parallel_world_size` 以及 `get_attention_dp_size`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 402-416: `should_use_dp_reduce_scatterv` predicate for should use data parallel reduce scatterv
```python
def should_use_dp_reduce_scatterv():
    """
    Use reduce_scatterv in the standard dispatcher's combine() for DP attention
    with EP, replacing the default all-reduce + dp_scatter path.
    Only changes the combine (post-kernel) communication; dispatch is unchanged.
    """
    return (
        not should_use_flashinfer_cutlass_moe_fp4_allgather()
        and get_moe_a2a_backend().is_none()
        and is_dp_attention_enabled()
        and get_attention_dp_size() > 1
        and get_moe_expert_parallel_world_size() == get_attention_dp_size()
    )
```
**EN:** This block defines `should_use_dp_reduce_scatterv` and contains the main logic for this step. It mainly invokes `get_moe_a2a_backend.is_none`, `is_dp_attention_enabled`, `should_use_flashinfer_cutlass_moe_fp4_allgather`, `get_attention_dp_size`, and `get_moe_expert_parallel_world_size`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `should_use_dp_reduce_scatterv`，并承载这一阶段的核心逻辑。 它主要调用 `get_moe_a2a_backend.is_none`、`is_dp_attention_enabled`、`should_use_flashinfer_cutlass_moe_fp4_allgather`、`get_attention_dp_size` 以及 `get_moe_expert_parallel_world_size`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 417-449: `should_skip_post_experts_all_reduce` predicate for should skip post experts all reduce
```python
def should_skip_post_experts_all_reduce(
    *,
    is_tp_path: bool,
    use_reduce_scatter: bool = False,
    should_allreduce_fusion: bool = False,
) -> bool:
    """Whether to skip the post-experts all-reduce (EP or TP) because a
    downstream component will fuse, replace, or absorb it.

    Skip reasons, in order:
      - ``should_allreduce_fusion``: LayerCommunicator will fuse the all-reduce
        with the next layer's residual all-reduce.
      - ``use_reduce_scatter``: LayerCommunicator's post-attention scatter will
        do reduce-scatter, which would double-reduce on top of an all-reduce.
      - ``should_use_dp_reduce_scatterv()``: the standard dispatcher's combine
        path replaces the all-reduce with a reduce-scatterv.
      - ``should_use_flashinfer_cutlass_moe_fp4_allgather()`` (TP path only):
        the flashinfer cutlass FP4 kernel performs an all-gather that absorbs
        the post-experts TP all-reduce. Not relevant to the EP all-reduce.

    The first two args are layer-context flags from ``LayerCommunicator`` and
    default to ``False`` for models that don't use it. Pass ``is_tp_path=True``
    for the post-experts TP all-reduce, ``False`` for the EP all-reduce.
    """
    if should_allreduce_fusion or use_reduce_scatter:
        return True
    if should_use_dp_reduce_scatterv():
        return True
    if is_tp_path and should_use_flashinfer_cutlass_moe_fp4_allgather():
        return True
    return False
```
**EN:** This block defines `should_skip_post_experts_all_reduce` and contains the main logic for this step. It mainly invokes `should_use_dp_reduce_scatterv` and `should_use_flashinfer_cutlass_moe_fp4_allgather`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `should_skip_post_experts_all_reduce`，并承载这一阶段的核心逻辑。 它主要调用 `should_use_dp_reduce_scatterv` 和 `should_use_flashinfer_cutlass_moe_fp4_allgather`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 450-464: Function `speculative_moe_backend_context` and its core logic
```python
@contextmanager
def speculative_moe_backend_context():
    """
    Context manager to temporarily use the speculative MoE backend for draft model operations.
    This ensures that draft models in speculative decoding use the configured speculative backend.
    """
    global MOE_RUNNER_BACKEND
    original_backend = MOE_RUNNER_BACKEND
    try:
        MOE_RUNNER_BACKEND = get_speculative_moe_runner_backend()
        yield
    finally:
        MOE_RUNNER_BACKEND = original_backend
```
**EN:** This block defines `speculative_moe_backend_context` and contains the main logic for this step. Decorators like `contextmanager` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_speculative_moe_runner_backend`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `original_backend` and `MOE_RUNNER_BACKEND` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `speculative_moe_backend_context`，并承载这一阶段的核心逻辑。 像 `contextmanager` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_speculative_moe_runner_backend`，说明该流程会编排底层辅助函数或计算内核。 像 `original_backend` 和 `MOE_RUNNER_BACKEND` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 465-490: Function `speculative_moe_a2a_backend_context` and its core logic
```python
@contextmanager
def speculative_moe_a2a_backend_context():
    """
    Context manager to temporarily use the speculative MoE A2A backend for draft model operations.
    This ensures that draft models in speculative decoding use the configured speculative A2A backend.
    """
    global MOE_A2A_BACKEND
    global DISABLE_FLASHINFER_CUTLASS_MOE_FP4_ALLGATHER
    original_backend = MOE_A2A_BACKEND
    original_disable_flashinfer_cutlass_moe_fp4_allgather = (
        DISABLE_FLASHINFER_CUTLASS_MOE_FP4_ALLGATHER
    )
    try:
        MOE_A2A_BACKEND = get_speculative_moe_a2a_backend()
        # Disable FP4 allgather for spec decode since MTP layers are unquantized
        DISABLE_FLASHINFER_CUTLASS_MOE_FP4_ALLGATHER = True
        yield
    finally:
        MOE_A2A_BACKEND = original_backend
        DISABLE_FLASHINFER_CUTLASS_MOE_FP4_ALLGATHER = (
            original_disable_flashinfer_cutlass_moe_fp4_allgather
        )


# The type of method in top-K routing, for use in torch custom op
# Please keep this in sync with the counterpart defined in https://github.com/flashinfer-ai/flashinfer/blob/main/include/flashinfer/trtllm/fused_moe/runner.h
```
**EN:** This block defines `speculative_moe_a2a_backend_context` and contains the main logic for this step. Decorators like `contextmanager` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_speculative_moe_a2a_backend`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `original_backend`, `original_disable_flashinfer_cutlass_moe_fp4_allgather`, `MOE_A2A_BACKEND`, and `DISABLE_FLASHINFER_CUTLASS_MOE_FP4_ALLGATHER` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `speculative_moe_a2a_backend_context`，并承载这一阶段的核心逻辑。 像 `contextmanager` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_speculative_moe_a2a_backend`，说明该流程会编排底层辅助函数或计算内核。 像 `original_backend`、`original_disable_flashinfer_cutlass_moe_fp4_allgather`、`MOE_A2A_BACKEND` 以及 `DISABLE_FLASHINFER_CUTLASS_MOE_FP4_ALLGATHER` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 491-507: Class `RoutingMethodType` declaration and shared state
```python
class RoutingMethodType(IntEnum):
    # Default: Softmax -> TopK
    Default = (0,)
    # Renormalize: TopK -> Softmax
    Renormalize = (1,)
    # DeepSeekV3: Sigmoid -> RoutingBiasAdd -> Top2 in group -> Top4 groups -> Top8 experts from the Top4 groups
    DeepSeekV3 = (2,)
    # Llama4: Top1 -> Sigmoid
    Llama4 = (3,)
    # Qwen3: Softmax -> TopK -> Renormalize
    RenormalizeNaive = (4,)
    # TopK only (no softmax)
    TopK = (5,)
    # Unspecified
    Unspecified = 6
```
**EN:** This block introduces class `RoutingMethodType` and the state shared by its methods. It inherits from `IntEnum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `RoutingMethodType`，并定义其方法共享的状态。 它继承自 `IntEnum`，说明了它在 SRT 层栈中的接入方式。

### Lines 508-512: Module constants and shared configuration
```python
AITER_PADDING_SIZE = 128
TRITON_PADDING_SIZE = 128


# Unit of padding - context dependent
```
**EN:** This section prepares the module namespace. Shared names such as `AITER_PADDING_SIZE` and `TRITON_PADDING_SIZE` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `AITER_PADDING_SIZE` 和 `TRITON_PADDING_SIZE` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 513-523: `get_moe_padding_size` getter for Mixture-of-Experts padding size
```python
def get_moe_padding_size(is_aiter_moe):
    if is_aiter_moe:
        return AITER_PADDING_SIZE
    else:
        return (
            TRITON_PADDING_SIZE
            if bool(int(os.getenv("SGLANG_MOE_PADDING", "0")))
            else 0
        )
```
**EN:** This block defines `get_moe_padding_size` and contains the main logic for this step. It mainly invokes `bool`, `int`, and `os.getenv`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_moe_padding_size`，并承载这一阶段的核心逻辑。 它主要调用 `bool`、`int` 以及 `os.getenv`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 524-553: `get_moe_weight_sizes` getter for Mixture-of-Experts weight sizes
```python
def get_moe_weight_sizes(inter_dim, is_concat, is_packed, is_aiter_moe):
    """
    Calculate dimensions for MoE weight tensors.

    Args:
        inter_dim: Base intermediate dimension.
        is_concat: If True, fusions W1 (gate) and W3 (up) projections.
        is_packed: If True, uses 4-bit quantization (two FP4 elements per byte).
        is_aiter_moe: If True, applies Aiter-specific kernel padding alignment.
    """
    # w2_down_dim is the packing rank, but w13_up_dim not (of matrix to matmul)
    w13_up_dim = 2 * inter_dim if is_concat else inter_dim
    w2_down_dim = inter_dim // 2 if is_packed else inter_dim

    if is_aiter_moe:
        padding_size = get_moe_padding_size(True)
        align_aiter = lambda n: ((n + padding_size - 1) // padding_size) * padding_size
        is_padded = (w2_down_dim % padding_size) > 0
        if is_padded:
            # w2_down_dim, padding & aligned, unit: parameter dtype
            w2_down_dim = align_aiter(w2_down_dim)
        # up proj + gate fusion : 2x
        if is_concat:
            w13_up_dim = w2_down_dim * 2
        # packed
        if hasattr(torch, "float4_e2m1fn_x2") and is_packed:
            # w13_up_dim (row rank of matmul matrix) is not packing dim, *2 to recover
            w13_up_dim *= 2

    return (w13_up_dim, w2_down_dim, False if not is_aiter_moe else is_padded)
```
**EN:** This block defines `get_moe_weight_sizes` and contains the main logic for this step. It mainly invokes `get_moe_padding_size`, `align_aiter`, and `hasattr`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `w13_up_dim`, `w2_down_dim`, `padding_size`, `align_aiter`, and `is_padded` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_moe_weight_sizes`，并承载这一阶段的核心逻辑。 它主要调用 `get_moe_padding_size`、`align_aiter` 以及 `hasattr`，说明该流程会编排底层辅助函数或计算内核。 像 `w13_up_dim`、`w2_down_dim`、`padding_size`、`align_aiter` 以及 `is_padded` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `MoeA2ABackend`, `MoeRunnerBackend`, `DeepEPMode`, `DeepEPOutputDtype`, and `get_deepep_output_dtype`. / **主要符号**：核心入口包括 `MoeA2ABackend`、`MoeRunnerBackend`、`DeepEPMode`、`DeepEPOutputDtype` 以及 `get_deepep_output_dtype`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `os`, `contextlib.contextmanager`, `enum.Enum`, `enum.IntEnum`, `typing.TYPE_CHECKING`, and `typing.Optional` / **标准库**：`__future__.annotations`、`logging`、`os`、`contextlib.contextmanager`、`enum.Enum`、`enum.IntEnum`、`typing.TYPE_CHECKING` 以及 `typing.Optional`
- **Third-party**: `torch` / **第三方依赖**：`torch`
- **Internal SGLang modules**: `sglang.srt.distributed.parallel_state.get_moe_expert_parallel_world_size`, `sglang.srt.environ.envs`, `sglang.srt.layers.dp_attention.get_attention_dp_size`, `sglang.srt.layers.dp_attention.is_dp_attention_enabled`, `sglang.srt.utils.is_npu`, `sglang.srt.server_args.get_global_server_args`, and `sglang.srt.server_args.ServerArgs` / **SGLang 内部模块**：`sglang.srt.distributed.parallel_state.get_moe_expert_parallel_world_size`、`sglang.srt.environ.envs`、`sglang.srt.layers.dp_attention.get_attention_dp_size`、`sglang.srt.layers.dp_attention.is_dp_attention_enabled`、`sglang.srt.utils.is_npu`、`sglang.srt.server_args.get_global_server_args` 以及 `sglang.srt.server_args.ServerArgs`
