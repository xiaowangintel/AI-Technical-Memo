# triton_kernels.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/moe_runner/triton_kernels.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with low-precision quantization data paths for the SGLang SRT runtime. It exposes symbols such as `TritonKernelsRunnerInput`, `TritonKernelsRunnerOutput`, `TritonKernelsQuantInfo`, and `TritonKernelsRunnerCore` and connects them to backend-specific paths such as `Triton` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合低精度量化数据路径。它提供了 `TritonKernelsRunnerInput`、`TritonKernelsRunnerOutput`、`TritonKernelsQuantInfo` 以及 `TritonKernelsRunnerCore` 等符号，并把这些符号连接到 `Triton` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-39: Imports, conditional backend setup, and runtime guards
```python
"""Triton kernels MoE runner backend skeleton."""

from __future__ import annotations

from dataclasses import dataclass
from typing import TYPE_CHECKING, Any, Optional

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

if TYPE_CHECKING:
    from triton_kernels.matmul_ogs import (
        GatherIndx,
        PrecisionConfig,
        RoutingData,
        ScatterIndx,
    )

    from sglang.srt.layers.moe.token_dispatcher.standard import (
        StandardCombineInput,
        StandardDispatchOutput,
    )


# ---------------------------------------------------------------------------
# Runner IO dataclasses
# ---------------------------------------------------------------------------
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.Optional`, and `torch`, so later blocks can reuse runtime, tensor, or backend helpers. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.Optional` 以及 `torch`，让后续代码可以复用运行时、张量或后端辅助逻辑。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 40-48: Class `TritonKernelsRunnerInput` declaration and shared state
```python
@dataclass
class TritonKernelsRunnerInput(RunnerInput):
    """Input bundle passed to the triton-kernels runner core."""

    hidden_states: torch.Tensor
    routing_data: "RoutingData"
    gather_indx: "GatherIndx"
    scatter_indx: "ScatterIndx"
```
**EN:** This block introduces class `TritonKernelsRunnerInput` and the state shared by its methods. It inherits from `RunnerInput`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Input bundle passed to the triton-kernels runner core.
**CN:** 该代码块引入类 `TritonKernelsRunnerInput`，并定义其方法共享的状态。 它继承自 `RunnerInput`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 49-53: Function `TritonKernelsRunnerInput.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.TRITON_KERNELS
```
**EN:** This block defines `TritonKernelsRunnerInput.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TritonKernelsRunnerInput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 54-59: Class `TritonKernelsRunnerOutput` declaration and shared state
```python
@dataclass
class TritonKernelsRunnerOutput(RunnerOutput):
    """Output bundle returned from the triton-kernels runner core."""

    hidden_states: torch.Tensor
```
**EN:** This block introduces class `TritonKernelsRunnerOutput` and the state shared by its methods. It inherits from `RunnerOutput`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Output bundle returned from the triton-kernels runner core.
**CN:** 该代码块引入类 `TritonKernelsRunnerOutput`，并定义其方法共享的状态。 它继承自 `RunnerOutput`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 60-64: Function `TritonKernelsRunnerOutput.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.TRITON_KERNELS
```
**EN:** This block defines `TritonKernelsRunnerOutput.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TritonKernelsRunnerOutput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 65-82: Class `TritonKernelsQuantInfo` declaration and shared state
```python
@dataclass
class TritonKernelsQuantInfo(MoeQuantInfo):
    """Quantization payload consumed by the triton-kernels backend."""

    w13_weight: torch.Tensor
    w2_weight: torch.Tensor
    w13_bias: Optional[torch.Tensor] = None
    w2_bias: Optional[torch.Tensor] = None
    w13_precision_config: Optional[PrecisionConfig] = None
    w2_precision_config: Optional[PrecisionConfig] = None
    global_num_experts: int = -1


# ---------------------------------------------------------------------------
# Runner core
# ---------------------------------------------------------------------------
```
**EN:** This block introduces class `TritonKernelsQuantInfo` and the state shared by its methods. It inherits from `MoeQuantInfo`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Quantization payload consumed by the triton-kernels backend.
**CN:** 该代码块引入类 `TritonKernelsQuantInfo`，并定义其方法共享的状态。 它继承自 `MoeQuantInfo`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 83-85: Class `TritonKernelsRunnerCore` declaration and shared state
```python
class TritonKernelsRunnerCore(MoeRunnerCore):
    """Execute MoE experts via the external triton_kernels package."""
```
**EN:** This block introduces class `TritonKernelsRunnerCore` and the state shared by its methods. It inherits from `MoeRunnerCore`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Execute MoE experts via the external triton_kernels package.
**CN:** 该代码块引入类 `TritonKernelsRunnerCore`，并定义其方法共享的状态。 它继承自 `MoeRunnerCore`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 86-148: Function `TritonKernelsRunnerCore.run` and its core logic
```python
    def run(
        self,
        runner_input: TritonKernelsRunnerInput,
        quant_info: TritonKernelsQuantInfo,
        running_state: dict,
        hooks: Optional[Any] = None,
    ) -> TritonKernelsRunnerOutput:
        from sglang.srt.layers.moe.fused_moe_triton.triton_kernels_moe import (
            triton_kernel_fused_experts,
            triton_kernel_fused_experts_with_bias,
        )

        assert (
            self.config.is_gated
        ), "Only gated MoEs are supported for Triton Kernels runner"

        hidden_states = runner_input.hidden_states

        common_kwargs = dict(
            routing_data=runner_input.routing_data,
            gather_indx=runner_input.gather_indx,
            scatter_indx=None if self.config.no_combine else runner_input.scatter_indx,
            inplace=False,
            activation=self.config.activation,
            apply_router_weight_on_input=self.config.apply_router_weight_on_input,
            global_num_experts=quant_info.global_num_experts,
        )

        has_bias = quant_info.w13_bias is not None or quant_info.w2_bias is not None

        if has_bias:
            assert (
                quant_info.w13_bias is not None and quant_info.w2_bias is not None
            ), "Bias execution requires both w13_bias and w2_bias"
            output = triton_kernel_fused_experts_with_bias(
                hidden_states=hidden_states,
                w1=quant_info.w13_weight,
                w1_pcg=quant_info.w13_precision_config,
                b1=quant_info.w13_bias,
                w2=quant_info.w2_weight,
                w2_pcg=quant_info.w2_precision_config,
                b2=quant_info.w2_bias,
                gemm1_alpha=self.config.gemm1_alpha,
                gemm1_clamp_limit=self.config.gemm1_clamp_limit,
                **common_kwargs,
            )
        else:
            output = triton_kernel_fused_experts(
                hidden_states=hidden_states,
                w1=quant_info.w13_weight,
                w2=quant_info.w2_weight,
                **common_kwargs,
            )

        if self.config.no_combine:
            tokens = runner_input.hidden_states.shape[0]
            hidden = runner_input.hidden_states.shape[-1]
            total_rows = output.shape[0]
            top_k = total_rows // tokens
            output = output.view(tokens, top_k, hidden)

        return TritonKernelsRunnerOutput(hidden_states=output)
```
**EN:** This block defines `TritonKernelsRunnerCore.run` and contains the main logic for this step. It mainly invokes `dict`, `TritonKernelsRunnerOutput`, `triton_kernel_fused_experts_with_bias`, `triton_kernel_fused_experts`, and `output.view`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `common_kwargs`, `has_bias`, `output`, and `tokens` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TritonKernelsRunnerCore.run`，并承载这一阶段的核心逻辑。 它主要调用 `dict`、`TritonKernelsRunnerOutput`、`triton_kernel_fused_experts_with_bias`、`triton_kernel_fused_experts` 以及 `output.view`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`common_kwargs`、`has_bias`、`output` 以及 `tokens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 149-158: Function `TritonKernelsRunnerCore.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.TRITON_KERNELS


# ---------------------------------------------------------------------------
# Permute / fused hooks
# ---------------------------------------------------------------------------
```
**EN:** This block defines `TritonKernelsRunnerCore.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TritonKernelsRunnerCore.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 159-184: Function `pre_permute_standard_to_triton_kernels` and its core logic
```python
@register_pre_permute("standard", "triton_kernel")
def pre_permute_standard_to_triton_kernels(
    dispatch_output: "StandardDispatchOutput",
    quant_info: TritonKernelsQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> TritonKernelsRunnerInput:
    from sglang.srt.layers.moe.topk import TopKOutputChecker

    hidden_states = dispatch_output.hidden_states
    topk_output = dispatch_output.topk_output

    assert TopKOutputChecker.format_is_triton_kernels(
        topk_output
    ), "Triton-kernel runner expects TritonKernelTopKOutput"

    routing_data, gather_indx, scatter_indx = topk_output

    return TritonKernelsRunnerInput(
        hidden_states=hidden_states,
        routing_data=routing_data,
        gather_indx=gather_indx,
        scatter_indx=scatter_indx,
    )
```
**EN:** This block defines `pre_permute_standard_to_triton_kernels` and contains the main logic for this step. Decorators like `register_pre_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_pre_permute`, `TopKOutputChecker.format_is_triton_kernels`, and `TritonKernelsRunnerInput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_output`, `routing_data`, `gather_indx`, and `scatter_indx` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `pre_permute_standard_to_triton_kernels`，并承载这一阶段的核心逻辑。 像 `register_pre_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_pre_permute`、`TopKOutputChecker.format_is_triton_kernels` 以及 `TritonKernelsRunnerInput`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_output`、`routing_data`、`gather_indx` 以及 `scatter_indx` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 185-203: Function `post_permute_triton_kernels_to_standard` and its core logic
```python
@register_post_permute("triton_kernel", "standard")
def post_permute_triton_kernels_to_standard(
    runner_output: TritonKernelsRunnerOutput,
    quant_info: TritonKernelsQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> StandardCombineInput:
    from sglang.srt.layers.moe.token_dispatcher.standard import StandardCombineInput

    hidden_states = runner_output.hidden_states

    if (
        runner_config.routed_scaling_factor is not None
        and runner_config.routed_scaling_factor != 1.0
        and not runner_config.no_combine
    ):
        hidden_states.mul_(runner_config.routed_scaling_factor)

    return StandardCombineInput(hidden_states=hidden_states)
```
**EN:** This block defines `post_permute_triton_kernels_to_standard` and contains the main logic for this step. Decorators like `register_post_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_post_permute`, `StandardCombineInput`, and `hidden_states.mul_`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `post_permute_triton_kernels_to_standard`，并承载这一阶段的核心逻辑。 像 `register_post_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_post_permute`、`StandardCombineInput` 以及 `hidden_states.mul_`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `TritonKernelsRunnerInput`, `TritonKernelsRunnerOutput`, `TritonKernelsQuantInfo`, `TritonKernelsRunnerCore`, and `pre_permute_standard_to_triton_kernels`. / **主要符号**：核心入口包括 `TritonKernelsRunnerInput`、`TritonKernelsRunnerOutput`、`TritonKernelsQuantInfo`、`TritonKernelsRunnerCore` 以及 `pre_permute_standard_to_triton_kernels`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.Any`, and `typing.Optional` / **标准库**：`__future__.annotations`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.Any` 以及 `typing.Optional`
- **Third-party**: `torch`, `triton_kernels.matmul_ogs.GatherIndx`, `triton_kernels.matmul_ogs.PrecisionConfig`, `triton_kernels.matmul_ogs.RoutingData`, and `triton_kernels.matmul_ogs.ScatterIndx` / **第三方依赖**：`torch`、`triton_kernels.matmul_ogs.GatherIndx`、`triton_kernels.matmul_ogs.PrecisionConfig`、`triton_kernels.matmul_ogs.RoutingData` 以及 `triton_kernels.matmul_ogs.ScatterIndx`
- **Internal SGLang modules**: `sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerCore`, `sglang.srt.layers.moe.moe_runner.base.RunnerInput`, `sglang.srt.layers.moe.moe_runner.base.RunnerOutput`, `sglang.srt.layers.moe.moe_runner.base.register_post_permute`, `sglang.srt.layers.moe.moe_runner.base.register_pre_permute`, `sglang.srt.layers.moe.utils.MoeRunnerBackend`, `sglang.srt.layers.moe.token_dispatcher.standard.StandardCombineInput`, `sglang.srt.layers.moe.token_dispatcher.standard.StandardDispatchOutput`, `sglang.srt.layers.moe.topk.TopKOutputChecker`, and `sglang.srt.layers.moe.fused_moe_triton.triton_kernels_moe.triton_kernel_fused_experts` / **SGLang 内部模块**：`sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerCore`、`sglang.srt.layers.moe.moe_runner.base.RunnerInput`、`sglang.srt.layers.moe.moe_runner.base.RunnerOutput`、`sglang.srt.layers.moe.moe_runner.base.register_post_permute`、`sglang.srt.layers.moe.moe_runner.base.register_pre_permute`、`sglang.srt.layers.moe.utils.MoeRunnerBackend`、`sglang.srt.layers.moe.token_dispatcher.standard.StandardCombineInput`、`sglang.srt.layers.moe.token_dispatcher.standard.StandardDispatchOutput`、`sglang.srt.layers.moe.topk.TopKOutputChecker` 以及 `sglang.srt.layers.moe.fused_moe_triton.triton_kernels_moe.triton_kernel_fused_experts`
