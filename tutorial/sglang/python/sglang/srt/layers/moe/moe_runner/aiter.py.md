# aiter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/moe_runner/aiter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with low-precision quantization data paths for the SGLang SRT runtime. It exposes symbols such as `AiterQuantType`, `AiterMoeQuantInfo`, `AiterRunnerInput`, and `AiterRunnerOutput` and connects them to backend-specific paths such as `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合低精度量化数据路径。它提供了 `AiterQuantType`、`AiterMoeQuantInfo`、`AiterRunnerInput` 以及 `AiterRunnerOutput` 等符号，并把这些符号连接到 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

from dataclasses import dataclass
from enum import Enum
from typing import TYPE_CHECKING, Any, Optional, Union

import torch

from sglang.srt.layers.moe.moe_runner.base import (
    MoeQuantInfo,
    MoeRunnerConfig,
    MoeRunnerCore,
    RunnerInput,
    RunnerOutput,
    register_post_permute,
    register_pre_permute,
)
from sglang.srt.layers.moe.utils import MoeRunnerBackend
from sglang.srt.utils import get_int_env_var

if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher.base import CombineInput
    from sglang.srt.layers.moe.token_dispatcher.deepep import (
        DeepEPLLDispatchOutput,
        DeepEPNormalDispatchOutput,
    )
    from sglang.srt.layers.moe.token_dispatcher.moriep import (
        MoriEPLLDispatchOutput,
        MoriEPNormalDispatchOutput,
    )
    from sglang.srt.layers.moe.token_dispatcher.standard import (
        StandardCombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `dataclasses.dataclass`, `enum.Enum`, `typing.TYPE_CHECKING`, `typing.Any`, and `typing.Optional`, so later blocks can reuse runtime, tensor, or backend helpers. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`dataclasses.dataclass`、`enum.Enum`、`typing.TYPE_CHECKING`、`typing.Any` 以及 `typing.Optional`，让后续代码可以复用运行时、张量或后端辅助逻辑。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 37-43: Class `AiterQuantType` declaration and shared state
```python
class AiterQuantType(str, Enum):
    NONE = "No"
    PER_TOKEN = "per_Token"
    PER_128X128 = "per_128x128"
    PER_1X32 = "per_1x32"
```
**EN:** This block introduces class `AiterQuantType` and the state shared by its methods. It inherits from `str` and `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `AiterQuantType`，并定义其方法共享的状态。 它继承自 `str` 和 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 44-60: Class `AiterMoeQuantInfo` declaration and shared state
```python
@dataclass
class AiterMoeQuantInfo(MoeQuantInfo):
    w13_weight: torch.Tensor
    w2_weight: torch.Tensor
    quant_type: AiterQuantType = AiterQuantType.NONE
    w13_scale: Optional[torch.Tensor] = None
    w2_scale: Optional[torch.Tensor] = None
    a13_scale: Optional[torch.Tensor] = None
    a2_scale: Optional[torch.Tensor] = None
    b13: Optional[torch.Tensor] = None
    b2: Optional[torch.Tensor] = None
    expert_mask: Optional[torch.Tensor] = None
    doweight_stage1: bool = False
    hidden_pad: int = 0
    intermediate_pad: int = 0
```
**EN:** This block introduces class `AiterMoeQuantInfo` and the state shared by its methods. It inherits from `MoeQuantInfo`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `AiterMoeQuantInfo`，并定义其方法共享的状态。 它继承自 `MoeQuantInfo`，说明了它在 SRT 层栈中的接入方式。

### Lines 61-75: Class `AiterRunnerInput` declaration and shared state
```python
@dataclass
class AiterRunnerInput(RunnerInput):
    hidden_states: torch.Tensor
    topk_ids: torch.Tensor  # int32
    topk_weights: torch.Tensor  # float32
    # Effective activation quant_type (may differ from quant_info.quant_type
    # after the dispatch-aware decision in mori pre_permute).
    quant_type: AiterQuantType
    # Per-token activation scale produced by an EP dispatcher (mori). Falls
    # back to quant_info.a13_scale when None.
    a1_scale: Optional[torch.Tensor] = None
    # Mori-only fused_moe kwargs.
    num_local_tokens: Optional[torch.Tensor] = None
    output_dtype: Optional[torch.dtype] = None
```
**EN:** This block introduces class `AiterRunnerInput` and the state shared by its methods. It inherits from `RunnerInput`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `AiterRunnerInput`，并定义其方法共享的状态。 它继承自 `RunnerInput`，说明了它在 SRT 层栈中的接入方式。

### Lines 76-80: Function `AiterRunnerInput.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.AITER
```
**EN:** This block defines `AiterRunnerInput.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `AiterRunnerInput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 81-84: Class `AiterRunnerOutput` declaration and shared state
```python
@dataclass
class AiterRunnerOutput(RunnerOutput):
    hidden_states: torch.Tensor
```
**EN:** This block introduces class `AiterRunnerOutput` and the state shared by its methods. It inherits from `RunnerOutput`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `AiterRunnerOutput`，并定义其方法共享的状态。 它继承自 `RunnerOutput`，说明了它在 SRT 层栈中的接入方式。

### Lines 85-89: Function `AiterRunnerOutput.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.AITER
```
**EN:** This block defines `AiterRunnerOutput.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `AiterRunnerOutput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 90-92: Module constants and shared configuration
```python
_AITER_ACTIVATIONS = {"silu": "Silu", "swiglu": "Swiglu"}
```
**EN:** This section prepares the module namespace. Shared names such as `_AITER_ACTIVATIONS` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `_AITER_ACTIVATIONS` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 93-98: Internal helper `_aiter_activation`
```python
def _aiter_activation(activation: str):
    from aiter import ActivationType

    return getattr(ActivationType, _AITER_ACTIVATIONS.get(activation, "Gelu"))
```
**EN:** This block defines `_aiter_activation` and contains the main logic for this step. It mainly invokes `getattr` and `_AITER_ACTIVATIONS.get`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_aiter_activation`，并承载这一阶段的核心逻辑。 它主要调用 `getattr` 和 `_AITER_ACTIVATIONS.get`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 99-104: Internal helper `_aiter_quant_type`
```python
def _aiter_quant_type(quant_type: AiterQuantType):
    from aiter import QuantType

    return getattr(QuantType, quant_type.value)
```
**EN:** This block defines `_aiter_quant_type` and contains the main logic for this step. It mainly invokes `getattr`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_aiter_quant_type`，并承载这一阶段的核心逻辑。 它主要调用 `getattr`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 105-105: Class `AiterRunnerCore` declaration and shared state
```python
class AiterRunnerCore(MoeRunnerCore):
```
**EN:** This block introduces class `AiterRunnerCore` and the state shared by its methods. It inherits from `MoeRunnerCore`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `AiterRunnerCore`，并定义其方法共享的状态。 它继承自 `MoeRunnerCore`，说明了它在 SRT 层栈中的接入方式。

### Lines 106-153: Function `AiterRunnerCore.run` and its core logic
```python
    def run(
        self,
        runner_input: AiterRunnerInput,
        quant_info: AiterMoeQuantInfo,
        running_state: dict,
        hooks: Optional[Any] = None,
    ) -> AiterRunnerOutput:
        assert not self.config.no_combine, "no_combine=True is not supported by AITER"

        if runner_input.hidden_states.shape[0] == 0:
            return AiterRunnerOutput(hidden_states=runner_input.hidden_states)

        from aiter.fused_moe import fused_moe

        a1_scale = (
            runner_input.a1_scale
            if runner_input.a1_scale is not None
            else quant_info.a13_scale
        )

        extra: dict = {}
        if runner_input.num_local_tokens is not None:
            extra["num_local_tokens"] = runner_input.num_local_tokens
        if runner_input.output_dtype is not None:
            extra["dtype"] = runner_input.output_dtype

        output = fused_moe(
            hidden_states=runner_input.hidden_states,
            w1=quant_info.w13_weight,
            w2=quant_info.w2_weight,
            topk_weight=runner_input.topk_weights,
            topk_ids=runner_input.topk_ids,
            quant_type=_aiter_quant_type(runner_input.quant_type),
            activation=_aiter_activation(self.config.activation),
            w1_scale=quant_info.w13_scale,
            w2_scale=quant_info.w2_scale,
            a1_scale=a1_scale,
            a2_scale=quant_info.a2_scale,
            bias1=quant_info.b13,
            bias2=quant_info.b2,
            expert_mask=quant_info.expert_mask,
            doweight_stage1=quant_info.doweight_stage1,
            hidden_pad=quant_info.hidden_pad,
            intermediate_pad=quant_info.intermediate_pad,
            **extra,
        )
        return AiterRunnerOutput(hidden_states=output)
```
**EN:** This block defines `AiterRunnerCore.run` and contains the main logic for this step. It mainly invokes `fused_moe`, `AiterRunnerOutput`, `_aiter_quant_type`, and `_aiter_activation`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `a1_scale`, `extra`, and `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `AiterRunnerCore.run`，并承载这一阶段的核心逻辑。 它主要调用 `fused_moe`、`AiterRunnerOutput`、`_aiter_quant_type` 以及 `_aiter_activation`，说明该流程会编排底层辅助函数或计算内核。 像 `a1_scale`、`extra` 以及 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 154-163: Function `AiterRunnerCore.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.AITER


# ---------------------------------------------------------------------------
# Pre-permute: dispatch_output -> AiterRunnerInput
# ---------------------------------------------------------------------------
```
**EN:** This block defines `AiterRunnerCore.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `AiterRunnerCore.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 164-190: Function `pre_permute_standard_to_aiter` and its core logic
```python
@register_pre_permute("standard", "aiter")
def pre_permute_standard_to_aiter(
    dispatch_output: StandardDispatchOutput,
    quant_info: AiterMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> AiterRunnerInput:
    hidden_states = dispatch_output.hidden_states
    topk_weights, topk_ids, _ = dispatch_output.topk_output
    topk_weights = topk_weights.to(torch.float32)

    if runner_config.apply_router_weight_on_input and not quant_info.doweight_stage1:
        # Pre-scale at the Python level for kernels that don't honor doweight_stage1.
        assert (
            topk_weights.dim() == 2 and topk_weights.shape[-1] == 1
        ), "apply_router_weight_on_input requires topk=1"
        hidden_states = hidden_states * topk_weights.to(hidden_states.dtype)
        topk_weights = torch.ones_like(topk_weights)

    return AiterRunnerInput(
        hidden_states=hidden_states,
        topk_ids=topk_ids.to(torch.int32),
        topk_weights=topk_weights,
        quant_type=quant_info.quant_type,
    )
```
**EN:** This block defines `pre_permute_standard_to_aiter` and contains the main logic for this step. Decorators like `register_pre_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_pre_permute`, `topk_weights.to`, `AiterRunnerInput`, `torch.ones_like`, and `topk_ids.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_weights`, `topk_ids`, and `_` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `pre_permute_standard_to_aiter`，并承载这一阶段的核心逻辑。 像 `register_pre_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_pre_permute`、`topk_weights.to`、`AiterRunnerInput`、`torch.ones_like` 以及 `topk_ids.to`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_weights`、`topk_ids` 以及 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 191-196: Internal helper `_is_mori_dispatch_output`
```python
def _is_mori_dispatch_output(dispatch_output: Any) -> bool:
    # MoriEP{Normal,LL}DispatchOutput carry the post-mori-permute origin_topk_*
    # tensors that the standard DeepEP outputs lack.
    return hasattr(dispatch_output, "origin_topk_ids")
```
**EN:** This block defines `_is_mori_dispatch_output` and contains the main logic for this step. It mainly invokes `hasattr`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_is_mori_dispatch_output`，并承载这一阶段的核心逻辑。 它主要调用 `hasattr`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 197-226: Internal helper `_resolve_mori_quant_type`
```python
def _resolve_mori_quant_type(
    dispatch_a1_dtype: torch.dtype,
    dispatch_scale: Optional[torch.Tensor],
    weight_quant: AiterQuantType,
) -> AiterQuantType:
    """Pick the activation quant_type for AITER when the dispatch path may have
    pre-quantized hidden_states. Mirrors the original MoriEPMoE.run_moe_core
    decision tree."""
    is_fp8_quant = weight_quant in (
        AiterQuantType.PER_128X128,
        AiterQuantType.PER_TOKEN,
    )
    is_w4a4 = weight_quant == AiterQuantType.PER_1X32
    is_fp4_dispatch = dispatch_a1_dtype == torch.float4_e2m1fn_x2
    has_dispatch_scale = dispatch_scale is not None

    if is_w4a4:
        # W4A4 weights always run as per_1x32; FP8 dispatch is upscaled to BF16
        # before this point so dispatch_scale won't conflict.
        return AiterQuantType.PER_1X32
    if is_fp8_quant:
        return weight_quant
    # BF16 weights: lift to the dispatch-side quant type when scales are provided.
    if has_dispatch_scale and is_fp4_dispatch:
        return AiterQuantType.PER_1X32
    if has_dispatch_scale and not is_fp4_dispatch:
        return AiterQuantType.PER_128X128
    return AiterQuantType.NONE
```
**EN:** This block defines `_resolve_mori_quant_type` and contains the main logic for this step. Intermediate names such as `is_fp8_quant`, `is_w4a4`, `is_fp4_dispatch`, and `has_dispatch_scale` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_resolve_mori_quant_type`，并承载这一阶段的核心逻辑。 像 `is_fp8_quant`、`is_w4a4`、`is_fp4_dispatch` 以及 `has_dispatch_scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 227-324: Internal helper `_pre_permute_deepep_to_aiter`
```python
def _pre_permute_deepep_to_aiter(
    dispatch_output: Union[
        DeepEPNormalDispatchOutput,
        DeepEPLLDispatchOutput,
        MoriEPNormalDispatchOutput,
        MoriEPLLDispatchOutput,
    ],
    quant_info: AiterMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> AiterRunnerInput:
    is_mori = _is_mori_dispatch_output(dispatch_output)

    hidden_states = dispatch_output.hidden_states
    topk_ids = dispatch_output.topk_ids.to(torch.int32)
    topk_weights = dispatch_output.topk_weights.to(torch.float32)
    a1_scale: Optional[torch.Tensor] = None
    num_local_tokens: Optional[torch.Tensor] = None
    output_dtype: Optional[torch.dtype] = None
    quant_type = quant_info.quant_type

    if is_mori:
        from sglang.srt.layers.moe.rocm_moe_utils import upscale, upscale_mxfp4

        a1_scale = dispatch_output.hidden_states_scale
        num_local_tokens = dispatch_output.num_recv_tokens_per_expert
        output_dtype = dispatch_output.out_dtype

        # Truncate dispatch tensors to the configured cap; mori combine only
        # reads [0, totalRecvTokenNum), so the truncated result needs no
        # padding back.
        mori_max = get_int_env_var("SGLANG_MORI_MOE_MAX_INPUT_TOKENS", 0)
        if mori_max > 0:
            hidden_states = hidden_states[:mori_max]
            if a1_scale is not None:
                a1_scale = a1_scale[:mori_max]
            topk_ids = topk_ids[:mori_max]
            topk_weights = topk_weights[:mori_max]

        # Upscale dispatched activations when there is no AITER kernel for the
        # weight/activation dtype pair.
        weight_quant = quant_info.quant_type
        is_fp8_quant = weight_quant in (
            AiterQuantType.PER_128X128,
            AiterQuantType.PER_TOKEN,
        )
        is_w4a4 = weight_quant == AiterQuantType.PER_1X32
        is_fp4_dispatch = hidden_states.dtype == torch.float4_e2m1fn_x2

        if is_w4a4 and a1_scale is not None and not is_fp4_dispatch:
            # W4A4 weights with FP8 dispatch: dequant FP8->BF16 first; the
            # FP4 per_1x32 path needs BF16 input.
            hidden_states = upscale(
                hidden_states, a1_scale, num_local_tokens, output_dtype
            )
            a1_scale = None
        elif is_fp8_quant and is_fp4_dispatch and a1_scale is not None:
            # FP8 weights + FP4 dispatch: no kernel for the fp4x2/fp8 pair;
            # dequant FP4->BF16 and let fused_moe re-quantize to FP8.
            hidden_states = upscale_mxfp4(
                hidden_states, a1_scale, num_local_tokens, output_dtype
            )
            a1_scale = None

        quant_type = _resolve_mori_quant_type(
            hidden_states.dtype, a1_scale, weight_quant
        )

        running_state["aiter_combine_topk_ids"] = dispatch_output.origin_topk_ids
        running_state["aiter_combine_topk_weights"] = (
            dispatch_output.origin_topk_weights
        )
    else:
        # DeepEP marks invalid topk slots with idx == -1; AITER cannot accept
        # negative ids, so reroute them to the sink slot at index
        # num_local_experts (masked off by quant_info.expert_mask which has
        # shape (num_local_experts + 1,)).
        topk_ids = torch.where(
            topk_ids == -1,
            torch.full_like(topk_ids, runner_config.num_local_experts),
            topk_ids,
        )
        running_state["aiter_combine_topk_ids"] = dispatch_output.topk_ids
        running_state["aiter_combine_topk_weights"] = dispatch_output.topk_weights

    running_state["aiter_combine_is_mori"] = is_mori

    return AiterRunnerInput(
        hidden_states=hidden_states,
        topk_ids=topk_ids,
        topk_weights=topk_weights,
        quant_type=quant_type,
        a1_scale=a1_scale,
        num_local_tokens=num_local_tokens,
        output_dtype=output_dtype,
    )
```
**EN:** This block defines `_pre_permute_deepep_to_aiter` and contains the main logic for this step. It mainly invokes `_is_mori_dispatch_output`, `dispatch_output.topk_ids.to`, `dispatch_output.topk_weights.to`, `AiterRunnerInput`, and `get_int_env_var`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `is_mori`, `hidden_states`, `topk_ids`, `topk_weights`, and `a1_scale` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_pre_permute_deepep_to_aiter`，并承载这一阶段的核心逻辑。 它主要调用 `_is_mori_dispatch_output`、`dispatch_output.topk_ids.to`、`dispatch_output.topk_weights.to`、`AiterRunnerInput` 以及 `get_int_env_var`，说明该流程会编排底层辅助函数或计算内核。 像 `is_mori`、`hidden_states`、`topk_ids`、`topk_weights` 以及 `a1_scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 325-333: Module-level helper logic
```python
register_pre_permute("deepep_normal", "aiter")(_pre_permute_deepep_to_aiter)
register_pre_permute("deepep_ll", "aiter")(_pre_permute_deepep_to_aiter)


# ---------------------------------------------------------------------------
# Post-permute: AiterRunnerOutput -> CombineInput
# ---------------------------------------------------------------------------
```
**EN:** This section prepares the module namespace.
**CN:** 该部分负责准备模块命名空间。

### Lines 334-345: Function `post_permute_aiter_to_standard` and its core logic
```python
@register_post_permute("aiter", "standard")
def post_permute_aiter_to_standard(
    runner_output: AiterRunnerOutput,
    quant_info: AiterMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> StandardCombineInput:
    from sglang.srt.layers.moe.token_dispatcher.standard import StandardCombineInput

    return StandardCombineInput(hidden_states=runner_output.hidden_states)
```
**EN:** This block defines `post_permute_aiter_to_standard` and contains the main logic for this step. Decorators like `register_post_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_post_permute` and `StandardCombineInput`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `post_permute_aiter_to_standard`，并承载这一阶段的核心逻辑。 像 `register_post_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_post_permute` 和 `StandardCombineInput`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 346-374: Internal helper `_post_permute_aiter_to_deepep`
```python
def _post_permute_aiter_to_deepep(
    runner_output: AiterRunnerOutput,
    quant_info: AiterMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
    is_normal: bool,
) -> CombineInput:
    if running_state.get("aiter_combine_is_mori"):
        from sglang.srt.layers.moe.token_dispatcher.moriep import (
            MoriEPLLCombineInput,
            MoriEPNormalCombineInput,
        )

        cls = MoriEPNormalCombineInput if is_normal else MoriEPLLCombineInput
    else:
        from sglang.srt.layers.moe.token_dispatcher.deepep import (
            DeepEPLLCombineInput,
            DeepEPNormalCombineInput,
        )

        cls = DeepEPNormalCombineInput if is_normal else DeepEPLLCombineInput

    return cls(
        hidden_states=runner_output.hidden_states,
        topk_ids=running_state["aiter_combine_topk_ids"],
        topk_weights=running_state["aiter_combine_topk_weights"],
    )
```
**EN:** This block defines `_post_permute_aiter_to_deepep` and contains the main logic for this step. It mainly invokes `running_state.get` and `cls`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `cls` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_post_permute_aiter_to_deepep`，并承载这一阶段的核心逻辑。 它主要调用 `running_state.get` 和 `cls`，说明该流程会编排底层辅助函数或计算内核。 像 `cls` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 375-386: Function `post_permute_aiter_to_deepep_normal` and its core logic
```python
@register_post_permute("aiter", "deepep_normal")
def post_permute_aiter_to_deepep_normal(
    runner_output: AiterRunnerOutput,
    quant_info: AiterMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> CombineInput:
    return _post_permute_aiter_to_deepep(
        runner_output, quant_info, runner_config, running_state, is_normal=True
    )
```
**EN:** This block defines `post_permute_aiter_to_deepep_normal` and contains the main logic for this step. Decorators like `register_post_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_post_permute` and `_post_permute_aiter_to_deepep`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `post_permute_aiter_to_deepep_normal`，并承载这一阶段的核心逻辑。 像 `register_post_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_post_permute` 和 `_post_permute_aiter_to_deepep`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 387-396: Function `post_permute_aiter_to_deepep_ll` and its core logic
```python
@register_post_permute("aiter", "deepep_ll")
def post_permute_aiter_to_deepep_ll(
    runner_output: AiterRunnerOutput,
    quant_info: AiterMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> CombineInput:
    return _post_permute_aiter_to_deepep(
        runner_output, quant_info, runner_config, running_state, is_normal=False
    )
```
**EN:** This block defines `post_permute_aiter_to_deepep_ll` and contains the main logic for this step. Decorators like `register_post_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_post_permute` and `_post_permute_aiter_to_deepep`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `post_permute_aiter_to_deepep_ll`，并承载这一阶段的核心逻辑。 像 `register_post_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_post_permute` 和 `_post_permute_aiter_to_deepep`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `AiterQuantType`, `AiterMoeQuantInfo`, `AiterRunnerInput`, `AiterRunnerOutput`, and `_aiter_activation`. / **主要符号**：核心入口包括 `AiterQuantType`、`AiterMoeQuantInfo`、`AiterRunnerInput`、`AiterRunnerOutput` 以及 `_aiter_activation`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `dataclasses.dataclass`, `enum.Enum`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.Optional`, and `typing.Union` / **标准库**：`__future__.annotations`、`dataclasses.dataclass`、`enum.Enum`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.Optional` 以及 `typing.Union`
- **Third-party**: `torch`, `aiter.ActivationType`, `aiter.QuantType`, and `aiter.fused_moe.fused_moe` / **第三方依赖**：`torch`、`aiter.ActivationType`、`aiter.QuantType` 以及 `aiter.fused_moe.fused_moe`
- **Internal SGLang modules**: `sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerCore`, `sglang.srt.layers.moe.moe_runner.base.RunnerInput`, `sglang.srt.layers.moe.moe_runner.base.RunnerOutput`, `sglang.srt.layers.moe.moe_runner.base.register_post_permute`, `sglang.srt.layers.moe.moe_runner.base.register_pre_permute`, `sglang.srt.layers.moe.utils.MoeRunnerBackend`, `sglang.srt.utils.get_int_env_var`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLDispatchOutput`, and `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPNormalDispatchOutput` / **SGLang 内部模块**：`sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerCore`、`sglang.srt.layers.moe.moe_runner.base.RunnerInput`、`sglang.srt.layers.moe.moe_runner.base.RunnerOutput`、`sglang.srt.layers.moe.moe_runner.base.register_post_permute`、`sglang.srt.layers.moe.moe_runner.base.register_pre_permute`、`sglang.srt.layers.moe.utils.MoeRunnerBackend`、`sglang.srt.utils.get_int_env_var`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLDispatchOutput` 以及 `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPNormalDispatchOutput`
