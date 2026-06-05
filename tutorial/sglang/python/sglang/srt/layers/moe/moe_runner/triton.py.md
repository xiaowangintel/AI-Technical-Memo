# triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/moe_runner/triton.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with low-precision quantization data paths for the SGLang SRT runtime. It exposes symbols such as `TritonRunnerInput`, `TritonRunnerOutput`, `TritonMoeQuantInfo`, and `TritonRunnerCore` and connects them to backend-specific paths such as `Triton` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合低精度量化数据路径。它提供了 `TritonRunnerInput`、`TritonRunnerOutput`、`TritonMoeQuantInfo` 以及 `TritonRunnerCore` 等符号，并把这些符号连接到 `Triton` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

from dataclasses import dataclass
from typing import TYPE_CHECKING, Any, List, Optional

import torch

from sglang.srt.layers.moe.moe_runner.base import (
    MoeQuantInfo,
    MoeRunnerConfig,
    MoeRunnerCore,
    RunnerInput,
    RunnerOutput,
    register_fused_func,
    register_post_permute,
    register_pre_permute,
)
from sglang.srt.layers.moe.utils import MoeRunnerBackend

if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher.standard import (
        StandardCombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.List`, and `typing.Optional`, so later blocks can reuse runtime, tensor, or backend helpers. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.List` 以及 `typing.Optional`，让后续代码可以复用运行时、张量或后端辅助逻辑。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 27-36: Class `TritonRunnerInput` declaration and shared state
```python
@dataclass
class TritonRunnerInput(RunnerInput):

    hidden_states: torch.Tensor
    topk_weights: torch.Tensor
    topk_ids: torch.Tensor
    sorted_token_ids: torch.Tensor
    expert_ids: torch.Tensor
    num_tokens_post_padded: torch.Tensor
```
**EN:** This block introduces class `TritonRunnerInput` and the state shared by its methods. It inherits from `RunnerInput`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `TritonRunnerInput`，并定义其方法共享的状态。 它继承自 `RunnerInput`，说明了它在 SRT 层栈中的接入方式。

### Lines 37-41: Function `TritonRunnerInput.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.TRITON
```
**EN:** This block defines `TritonRunnerInput.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TritonRunnerInput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 42-46: Class `TritonRunnerOutput` declaration and shared state
```python
@dataclass
class TritonRunnerOutput(RunnerOutput):

    hidden_states: torch.Tensor
```
**EN:** This block introduces class `TritonRunnerOutput` and the state shared by its methods. It inherits from `RunnerOutput`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `TritonRunnerOutput`，并定义其方法共享的状态。 它继承自 `RunnerOutput`，说明了它在 SRT 层栈中的接入方式。

### Lines 47-51: Function `TritonRunnerOutput.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.TRITON
```
**EN:** This block defines `TritonRunnerOutput.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TritonRunnerOutput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 52-71: Class `TritonMoeQuantInfo` declaration and shared state
```python
@dataclass
class TritonMoeQuantInfo(MoeQuantInfo):
    w13_weight: torch.Tensor
    w2_weight: torch.Tensor
    b13: Optional[torch.Tensor] = None
    b2: Optional[torch.Tensor] = None
    use_fp8_w8a8: bool = False
    use_int8_w8a8: bool = False
    use_int8_w8a16: bool = False
    use_int4_w4a16: bool = False
    per_channel_quant: bool = False
    w13_scale: Optional[torch.Tensor] = None
    w2_scale: Optional[torch.Tensor] = None
    w13_zp: Optional[torch.Tensor] = None
    w2_zp: Optional[torch.Tensor] = None
    a13_scale: Optional[torch.Tensor] = None
    a2_scale: Optional[torch.Tensor] = None
    block_shape: Optional[List[int]] = None
```
**EN:** This block introduces class `TritonMoeQuantInfo` and the state shared by its methods. It inherits from `MoeQuantInfo`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `TritonMoeQuantInfo`，并定义其方法共享的状态。 它继承自 `MoeQuantInfo`，说明了它在 SRT 层栈中的接入方式。

### Lines 72-73: Class `TritonRunnerCore` declaration and shared state
```python
class TritonRunnerCore(MoeRunnerCore):
```
**EN:** This block introduces class `TritonRunnerCore` and the state shared by its methods. It inherits from `MoeRunnerCore`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `TritonRunnerCore`，并定义其方法共享的状态。 它继承自 `MoeRunnerCore`，说明了它在 SRT 层栈中的接入方式。

### Lines 74-76: `TritonRunnerCore` initialization and state setup
```python
    def __init__(self, config: MoeRunnerConfig):
        super().__init__(config)
```
**EN:** This block defines `TritonRunnerCore.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `TritonRunnerCore.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。

### Lines 77-133: Function `TritonRunnerCore.run` and its core logic
```python
    def run(
        self,
        runner_input: TritonRunnerInput,
        quant_info: TritonMoeQuantInfo,
        running_state: dict,
        hooks: Optional[Any] = None,
    ) -> TritonRunnerOutput:
        from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import (
            _fused_moe_kernel_sequence,
        )

        filter_expert = (
            self.config.num_experts is None
            or self.config.num_experts != self.config.num_local_experts
        )

        out = _fused_moe_kernel_sequence(
            runner_input.hidden_states,
            quant_info.w13_weight,
            quant_info.w2_weight,
            runner_input.topk_weights,
            runner_input.topk_ids,
            runner_input.sorted_token_ids,
            runner_input.expert_ids,
            runner_input.num_tokens_post_padded,
            running_state["config"],
            running_state.get("down_config"),
            running_state.get("down_moe_use_tma", False),
            b1=quant_info.b13,
            b2=quant_info.b2,
            use_fp8_w8a8=quant_info.use_fp8_w8a8,
            use_int8_w8a8=quant_info.use_int8_w8a8,
            use_int8_w8a16=quant_info.use_int8_w8a16,
            use_int4_w4a16=quant_info.use_int4_w4a16,
            per_channel_quant=quant_info.per_channel_quant,
            w1_scale=quant_info.w13_scale,
            w2_scale=quant_info.w2_scale,
            w1_zp=quant_info.w13_zp,
            w2_zp=quant_info.w2_zp,
            a1_scale=quant_info.a13_scale,
            a2_scale=quant_info.a2_scale,
            block_shape=quant_info.block_shape,
            activation=self.config.activation,
            is_gated=self.config.is_gated,
            no_combine=self.config.no_combine,
            inplace=self.config.inplace,
            apply_router_weight_on_input=self.config.apply_router_weight_on_input,
            routed_scaling_factor=self.config.routed_scaling_factor,
            gemm1_alpha=self.config.gemm1_alpha,
            gemm1_limit=self.config.gemm1_clamp_limit,
            filter_expert=filter_expert,
            hooks=hooks,
            swiglu_limit=self.config.swiglu_limit,
        )

        return TritonRunnerOutput(hidden_states=out)
```
**EN:** This block defines `TritonRunnerCore.run` and contains the main logic for this step. It mainly invokes `_fused_moe_kernel_sequence`, `TritonRunnerOutput`, and `running_state.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `filter_expert` and `out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TritonRunnerCore.run`，并承载这一阶段的核心逻辑。 它主要调用 `_fused_moe_kernel_sequence`、`TritonRunnerOutput` 以及 `running_state.get`，说明该流程会编排底层辅助函数或计算内核。 像 `filter_expert` 和 `out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 134-138: Function `TritonRunnerCore.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.TRITON
```
**EN:** This block defines `TritonRunnerCore.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TritonRunnerCore.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 139-174: Function `fused_experts_none_to_triton` and its core logic
```python
@register_fused_func("none", "triton")
def fused_experts_none_to_triton(
    dispatch_output: StandardDispatchOutput,
    quant_info: TritonMoeQuantInfo,
    runner_config: MoeRunnerConfig,
) -> StandardCombineInput:
    from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import fused_experts
    from sglang.srt.layers.moe.token_dispatcher.standard import StandardCombineInput

    output = fused_experts(
        hidden_states=dispatch_output.hidden_states,
        w1=quant_info.w13_weight,
        w2=quant_info.w2_weight,
        topk_output=dispatch_output.topk_output,
        moe_runner_config=runner_config,
        b1=quant_info.b13,
        b2=quant_info.b2,
        use_fp8_w8a8=quant_info.use_fp8_w8a8,
        use_int8_w8a8=quant_info.use_int8_w8a8,
        use_int8_w8a16=quant_info.use_int8_w8a16,
        use_int4_w4a16=quant_info.use_int4_w4a16,
        per_channel_quant=quant_info.per_channel_quant,
        w1_scale=quant_info.w13_scale,
        w2_scale=quant_info.w2_scale,
        w1_zp=quant_info.w13_zp,
        w2_zp=quant_info.w2_zp,
        a1_scale=quant_info.a13_scale,
        a2_scale=quant_info.a2_scale,
        block_shape=quant_info.block_shape,
    )

    return StandardCombineInput(
        hidden_states=output,
    )
```
**EN:** This block defines `fused_experts_none_to_triton` and contains the main logic for this step. Decorators like `register_fused_func` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_fused_func`, `fused_experts`, and `StandardCombineInput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_experts_none_to_triton`，并承载这一阶段的核心逻辑。 像 `register_fused_func` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_fused_func`、`fused_experts` 以及 `StandardCombineInput`，说明该流程会编排底层辅助函数或计算内核。 像 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 175-231: Function `pre_permute_standard_to_triton` and its core logic
```python
@register_pre_permute("standard", "triton")
def pre_permute_standard_to_triton(
    dispatch_output: StandardDispatchOutput,
    quant_info: TritonMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> TritonRunnerInput:

    # NOTE: this is dead code as a fused func for standard format is registered.
    # This is left here for testing and examples.

    from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import (
        _prepare_fused_moe_run,
    )
    from sglang.srt.layers.moe.topk import TopKOutputChecker

    hidden_states, topk_output = (
        dispatch_output.hidden_states,
        dispatch_output.topk_output,
    )

    assert TopKOutputChecker.format_is_standard(topk_output)

    (
        config,
        down_config,
        down_moe_use_tma,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
    ) = _prepare_fused_moe_run(
        hidden_states,
        quant_info.w13_weight,
        quant_info.w2_weight,
        topk_output.topk_ids,
        use_fp8_w8a8=quant_info.use_fp8_w8a8,
        use_int8_w8a8=quant_info.use_int8_w8a8,
        use_int8_w8a16=quant_info.use_int8_w8a16,
        use_int4_w4a16=quant_info.use_int4_w4a16,
        per_channel_quant=quant_info.per_channel_quant,
        block_shape=quant_info.block_shape,
    )

    running_state["config"] = config
    running_state["down_config"] = down_config
    running_state["down_moe_use_tma"] = down_moe_use_tma

    return TritonRunnerInput(
        hidden_states=hidden_states,
        topk_weights=topk_output.topk_weights,
        topk_ids=topk_output.topk_ids,
        sorted_token_ids=sorted_token_ids,
        expert_ids=expert_ids,
        num_tokens_post_padded=num_tokens_post_padded,
    )
```
**EN:** This block defines `pre_permute_standard_to_triton` and contains the main logic for this step. Decorators like `register_pre_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_pre_permute`, `TopKOutputChecker.format_is_standard`, `_prepare_fused_moe_run`, and `TritonRunnerInput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_output`, `config`, `down_config`, and `down_moe_use_tma` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `pre_permute_standard_to_triton`，并承载这一阶段的核心逻辑。 像 `register_pre_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_pre_permute`、`TopKOutputChecker.format_is_standard`、`_prepare_fused_moe_run` 以及 `TritonRunnerInput`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_output`、`config`、`down_config` 以及 `down_moe_use_tma` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 232-247: Function `post_permute_triton_to_standard` and its core logic
```python
@register_post_permute("triton", "standard")
def post_permute_triton_to_standard(
    runner_output: TritonRunnerOutput,
    quant_info: TritonMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> StandardCombineInput:

    # NOTE: this is dead code as a fused func for standard format is registered.
    # This is left here for testing and examples.

    from sglang.srt.layers.moe.token_dispatcher.standard import StandardCombineInput

    return StandardCombineInput(
        hidden_states=runner_output.hidden_states,
    )
```
**EN:** This block defines `post_permute_triton_to_standard` and contains the main logic for this step. Decorators like `register_post_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_post_permute` and `StandardCombineInput`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `post_permute_triton_to_standard`，并承载这一阶段的核心逻辑。 像 `register_post_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_post_permute` 和 `StandardCombineInput`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `TritonRunnerInput`, `TritonRunnerOutput`, `TritonMoeQuantInfo`, `TritonRunnerCore`, and `fused_experts_none_to_triton`. / **主要符号**：核心入口包括 `TritonRunnerInput`、`TritonRunnerOutput`、`TritonMoeQuantInfo`、`TritonRunnerCore` 以及 `fused_experts_none_to_triton`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.List`, and `typing.Optional` / **标准库**：`__future__.annotations`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.List` 以及 `typing.Optional`
- **Third-party**: `torch` / **第三方依赖**：`torch`
- **Internal SGLang modules**: `sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerCore`, `sglang.srt.layers.moe.moe_runner.base.RunnerInput`, `sglang.srt.layers.moe.moe_runner.base.RunnerOutput`, `sglang.srt.layers.moe.moe_runner.base.register_fused_func`, `sglang.srt.layers.moe.moe_runner.base.register_post_permute`, `sglang.srt.layers.moe.moe_runner.base.register_pre_permute`, `sglang.srt.layers.moe.utils.MoeRunnerBackend`, `sglang.srt.layers.moe.token_dispatcher.standard.StandardCombineInput`, `sglang.srt.layers.moe.token_dispatcher.standard.StandardDispatchOutput`, and `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe.fused_experts` / **SGLang 内部模块**：`sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerCore`、`sglang.srt.layers.moe.moe_runner.base.RunnerInput`、`sglang.srt.layers.moe.moe_runner.base.RunnerOutput`、`sglang.srt.layers.moe.moe_runner.base.register_fused_func`、`sglang.srt.layers.moe.moe_runner.base.register_post_permute`、`sglang.srt.layers.moe.moe_runner.base.register_pre_permute`、`sglang.srt.layers.moe.utils.MoeRunnerBackend`、`sglang.srt.layers.moe.token_dispatcher.standard.StandardCombineInput`、`sglang.srt.layers.moe.token_dispatcher.standard.StandardDispatchOutput` 以及 `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe.fused_experts`
