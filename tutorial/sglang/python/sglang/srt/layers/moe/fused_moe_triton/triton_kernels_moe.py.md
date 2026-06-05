# triton_kernels_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/fused_moe_triton/triton_kernels_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with low-precision quantization data paths for the SGLang SRT runtime. It exposes symbols such as `quantize`, `triton_kernel_moe_forward`, `triton_kernel_fused_experts`, and `triton_kernel_moe_with_bias_forward` and connects them to backend-specific paths such as `CUDA`, `Triton`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合低精度量化数据路径。它提供了 `quantize`、`triton_kernel_moe_forward`、`triton_kernel_fused_experts` 以及 `triton_kernel_moe_with_bias_forward` 等符号，并把这些符号连接到 `CUDA`、`Triton` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: File header and module overview
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/pull/18595/files#diff-f426a6de78c82ffec568eff6811bfbf0043dab5f87f1a8c0cffdbdcb8a81e035
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 5-34: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

from typing import TYPE_CHECKING, Optional

import torch
from triton_kernels.matmul_ogs import (
    FlexCtx,
    FnSpecs,
    FusedActivation,
    GatherIndx,
    PrecisionConfig,
    RoutingData,
    ScatterIndx,
    matmul_ogs,
)
from triton_kernels.numerics import InFlexData
from triton_kernels.swiglu import swiglu_fn

from sglang.srt.utils import is_cuda

if is_cuda():
    from sglang.jit_kernel.activation import gelu_and_mul, silu_and_mul
else:
    from sgl_kernel import gelu_and_mul, silu_and_mul

if TYPE_CHECKING:
    from sglang.srt.layers.moe.moe_runner import MoeRunnerConfig
    from sglang.srt.layers.moe.topk import TopKOutput
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `typing.TYPE_CHECKING`, `typing.Optional`, `torch`, `triton_kernels.matmul_ogs.FlexCtx`, and `triton_kernels.matmul_ogs.FnSpecs`, so later blocks can reuse runtime, tensor, or backend helpers. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`typing.TYPE_CHECKING`、`typing.Optional`、`torch`、`triton_kernels.matmul_ogs.FlexCtx` 以及 `triton_kernels.matmul_ogs.FnSpecs`，让后续代码可以复用运行时、张量或后端辅助逻辑。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 35-39: Function `quantize` and its core logic
```python
def quantize(w, dtype, dev, **opt):
    if dtype == "bf16":
        return w.to(torch.bfloat16), InFlexData()
```
**EN:** This block defines `quantize` and contains the main logic for this step. It mainly invokes `w.to` and `InFlexData`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `quantize`，并承载这一阶段的核心逻辑。 它主要调用 `w.to` 和 `InFlexData`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 40-86: Function `triton_kernel_moe_forward` and its core logic
```python
def triton_kernel_moe_forward(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_output: TopKOutput,
    moe_runner_config: MoeRunnerConfig,
    apply_router_weight_on_input: bool = False,
    use_fp8_w8a8: bool = False,
    per_channel_quant: bool = False,
    global_num_experts: int = -1,
    expert_map: Optional[torch.Tensor] = None,
    w1_scale: Optional[torch.Tensor] = None,
    w2_scale: Optional[torch.Tensor] = None,
    a1_scale: Optional[torch.Tensor] = None,
    a2_scale: Optional[torch.Tensor] = None,
    block_shape: Optional[list[int]] = None,
) -> torch.Tensor:

    from sglang.srt.layers.moe.topk import TopKOutputChecker

    assert TopKOutputChecker.format_is_triton_kernels(topk_output)

    routing_data, gather_idx, scatter_idx = topk_output

    return triton_kernel_fused_experts(
        hidden_states,
        w1,
        w2,
        routing_data,
        gather_idx,
        scatter_idx,
        inplace=False,  # triton kernel doesn't support inplace
        activation=moe_runner_config.activation,
        apply_router_weight_on_input=apply_router_weight_on_input,
        use_fp8_w8a8=use_fp8_w8a8,
        per_channel_quant=per_channel_quant,
        global_num_experts=global_num_experts,
        expert_map=expert_map,
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        a1_scale=a1_scale,
        a2_scale=a2_scale,
        block_shape=block_shape,
    )


# This is a triton implementation of the fused_experts function
```
**EN:** This block defines `triton_kernel_moe_forward` and contains the main logic for this step. It mainly invokes `TopKOutputChecker.format_is_triton_kernels` and `triton_kernel_fused_experts`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `routing_data`, `gather_idx`, and `scatter_idx` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `triton_kernel_moe_forward`，并承载这一阶段的核心逻辑。 它主要调用 `TopKOutputChecker.format_is_triton_kernels` 和 `triton_kernel_fused_experts`，说明该流程会编排底层辅助函数或计算内核。 像 `routing_data`、`gather_idx` 以及 `scatter_idx` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 87-174: Function `triton_kernel_fused_experts` and its core logic
```python
def triton_kernel_fused_experts(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    routing_data: RoutingData,
    gather_indx: GatherIndx,
    scatter_indx: ScatterIndx,
    inplace: bool = False,
    activation: str = "silu",
    apply_router_weight_on_input: bool = False,
    use_fp8_w8a8: bool = False,
    per_channel_quant: bool = False,
    global_num_experts: int = -1,
    expert_map: Optional[torch.Tensor] = None,
    w1_scale: Optional[torch.Tensor] = None,
    w2_scale: Optional[torch.Tensor] = None,
    a1_scale: Optional[torch.Tensor] = None,
    a2_scale: Optional[torch.Tensor] = None,
    block_shape: Optional[list[int]] = None,
) -> torch.Tensor:

    assert use_fp8_w8a8 is False, "use_fp8_w8a8 is not supported"
    assert per_channel_quant is False, "per_channel_quant is not supported"
    assert expert_map is None, "expert_map is not supported"
    assert w1_scale is None, "w1_scale is not supported"
    assert w2_scale is None, "w2_scale is not supported"
    assert a1_scale is None, "a1_scale is not supported"
    assert a2_scale is None, "a2_scale is not supported"
    assert block_shape is None, "block_shape is not supported"

    # type check
    assert hidden_states.dtype == torch.bfloat16, "hidden_states must be bfloat16"
    assert w1.dtype == torch.bfloat16, "w1 must be bfloat16"
    assert w2.dtype == torch.bfloat16, "w2 must be bfloat16"

    # Shape check
    assert hidden_states.ndim == 2, "hidden_states must be 2D"
    assert (
        hidden_states.shape[-1] == w1.shape[-2]
    ), f"hidden_states shape[-1] {hidden_states.shape} must be equal to w1 shape[-2] {w1.shape}"
    assert (
        w2.shape[-1] == w1.shape[1]
    ), f"w2 shape[-1] {w2.shape[-1]} must be equal to w1 shape[1] {w1.shape[1]}"

    # feature check
    assert inplace is False, "Inplace is not supported in new triton MoE kernel"

    M, K = hidden_states.shape
    E, _, N = w1.shape
    n_expts_act = routing_data.n_expts_act
    dtype = hidden_states.dtype

    if global_num_experts == -1:
        global_num_experts = E

    # consistent with default implementation
    intermediate_cache2 = torch.empty(
        (M * n_expts_act, N // 2), device="cuda", dtype=dtype
    )

    intermediate_cache1 = matmul_ogs(
        hidden_states,
        w1,
        None,
        routing_data,
        gather_indx=gather_indx,
        gammas=routing_data.gate_scal if apply_router_weight_on_input else None,
    )

    if activation == "silu":
        silu_and_mul(intermediate_cache1.view(-1, N), intermediate_cache2)
    elif activation == "gelu":
        gelu_and_mul(intermediate_cache1.view(-1, N), intermediate_cache2)
    else:
        raise ValueError(f"Unsupported FusedMoe activation: {activation}")

    intermediate_cache3 = matmul_ogs(
        intermediate_cache2,
        w2,
        None,
        routing_data,
        scatter_indx=scatter_indx,
        gammas=None if apply_router_weight_on_input else routing_data.gate_scal,
    )

    return intermediate_cache3
```
**EN:** This block defines `triton_kernel_fused_experts` and contains the main logic for this step. It mainly invokes `torch.empty`, `matmul_ogs`, `silu_and_mul`, `intermediate_cache1.view`, and `gelu_and_mul`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `M`, `K`, `E`, `_`, and `N` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `triton_kernel_fused_experts`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`matmul_ogs`、`silu_and_mul`、`intermediate_cache1.view` 以及 `gelu_and_mul`，说明该流程会编排底层辅助函数或计算内核。 像 `M`、`K`、`E`、`_` 以及 `N` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 175-229: Function `triton_kernel_moe_with_bias_forward` and its core logic
```python
def triton_kernel_moe_with_bias_forward(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w1_pcg,
    b1: torch.Tensor,
    w2: torch.Tensor,
    w2_pcg,
    b2: torch.Tensor,
    topk_output: TopKOutput,
    moe_runner_config: MoeRunnerConfig,
    apply_router_weight_on_input: bool = False,
    use_fp8_w8a8: bool = False,
    per_channel_quant: bool = False,
    global_num_experts: int = -1,
    expert_map: Optional[torch.Tensor] = None,
    w1_scale: Optional[torch.Tensor] = None,
    w2_scale: Optional[torch.Tensor] = None,
    a1_scale: Optional[torch.Tensor] = None,
    a2_scale: Optional[torch.Tensor] = None,
    block_shape: Optional[list[int]] = None,
) -> torch.Tensor:
    from sglang.srt.layers.moe.topk import TopKOutputChecker

    assert TopKOutputChecker.format_is_triton_kernels(topk_output)

    routing_data, gather_idx, scatter_idx = topk_output

    return triton_kernel_fused_experts_with_bias(
        hidden_states,
        w1=w1,
        w1_pcg=w1_pcg,
        b1=b1,
        w2=w2,
        w2_pcg=w2_pcg,
        b2=b2,
        routing_data=routing_data,
        gather_indx=gather_idx,
        scatter_indx=scatter_idx,
        inplace=False,  # triton kernel doesn't support inplace
        activation=moe_runner_config.activation,
        apply_router_weight_on_input=apply_router_weight_on_input,
        use_fp8_w8a8=use_fp8_w8a8,
        per_channel_quant=per_channel_quant,
        global_num_experts=global_num_experts,
        expert_map=expert_map,
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        a1_scale=a1_scale,
        a2_scale=a2_scale,
        block_shape=block_shape,
        gemm1_alpha=moe_runner_config.gemm1_alpha,
        gemm1_clamp_limit=moe_runner_config.gemm1_clamp_limit,
    )
```
**EN:** This block defines `triton_kernel_moe_with_bias_forward` and contains the main logic for this step. It mainly invokes `TopKOutputChecker.format_is_triton_kernels` and `triton_kernel_fused_experts_with_bias`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `routing_data`, `gather_idx`, and `scatter_idx` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `triton_kernel_moe_with_bias_forward`，并承载这一阶段的核心逻辑。 它主要调用 `TopKOutputChecker.format_is_triton_kernels` 和 `triton_kernel_fused_experts_with_bias`，说明该流程会编排底层辅助函数或计算内核。 像 `routing_data`、`gather_idx` 以及 `scatter_idx` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 230-337: Function `triton_kernel_fused_experts_with_bias` and its core logic
```python
def triton_kernel_fused_experts_with_bias(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w1_pcg,
    b1: torch.Tensor,
    w2: torch.Tensor,
    w2_pcg,
    b2: torch.Tensor,
    routing_data: RoutingData,
    gather_indx: GatherIndx,
    scatter_indx: ScatterIndx,
    inplace: bool = False,
    activation: str = "silu",
    apply_router_weight_on_input: bool = False,
    use_fp8_w8a8: bool = False,
    per_channel_quant: bool = False,
    global_num_experts: int = -1,
    expert_map: Optional[torch.Tensor] = None,
    w1_scale: Optional[torch.Tensor] = None,
    w2_scale: Optional[torch.Tensor] = None,
    a1_scale: Optional[torch.Tensor] = None,
    a2_scale: Optional[torch.Tensor] = None,
    block_shape: Optional[list[int]] = None,
    gemm1_alpha: Optional[float] = None,
    gemm1_clamp_limit: Optional[float] = None,
) -> torch.Tensor:
    assert use_fp8_w8a8 is False, "use_fp8_w8a8 is not supported"
    assert per_channel_quant is False, "per_channel_quant is not supported"
    assert expert_map is None, "expert_map is not supported"
    assert w1_scale is None, "w1_scale is not supported"
    assert w2_scale is None, "w2_scale is not supported"
    assert a1_scale is None, "a1_scale is not supported"
    assert a2_scale is None, "a2_scale is not supported"
    assert block_shape is None, "block_shape is not supported"

    # type check
    assert hidden_states.dtype == torch.bfloat16, "hidden_states must be bfloat16"
    for w in (w1, w2):
        # TODO assert bf16 or mxfp4
        # assert (w.dtype == torch.bfloat16) or check-is-mxfp4, f"w must be bfloat16 or mxfp4 {w1.dtype=}"
        pass

    # Shape check
    assert hidden_states.ndim == 2, "hidden_states must be 2D"
    assert (
        hidden_states.shape[-1] == w1.shape[-2]
    ), f"hidden_states shape[-1] {hidden_states.shape} must be equal to w1 shape[-2] {w1.shape}"
    assert (
        w2.shape[-1] == w1.shape[1]
    ), f"w2 shape[-1] {w2.shape[-1]} must be equal to w1 shape[1] {w1.shape[1]}"

    # feature check
    assert inplace is False, "Inplace is not supported in new triton MoE kernel"

    M, K = hidden_states.shape
    E, _, N = w1.shape
    n_expts_act = routing_data.n_expts_act

    if global_num_experts == -1:
        global_num_experts = E

    # TODO maybe completely remove this branch
    if w1.dtype == torch.bfloat16:
        device = "cuda"
        optg = dict()
        w1, w1_flex = quantize(w1, "bf16", device, **optg)
        w1_pcg = PrecisionConfig(flex_ctx=FlexCtx(rhs_data=w1_flex))

        w2, w2_flex = quantize(w2, "bf16", device, **optg)
        w2_pcg = PrecisionConfig(flex_ctx=FlexCtx(rhs_data=w2_flex))

    act = FusedActivation(
        FnSpecs("swiglu", swiglu_fn, ("alpha", "limit"), reduction_n=2),
        (gemm1_alpha, gemm1_clamp_limit),
    )

    intermediate_cache = torch.empty(
        (1, M * n_expts_act, N // 2),
        device=hidden_states.device,
        dtype=hidden_states.dtype,
    )
    output = torch.empty(
        (1, M, K), device=hidden_states.device, dtype=hidden_states.dtype
    )

    matmul_ogs(
        hidden_states,
        w1,
        b1,
        routing_data,
        gather_indx=gather_indx,
        precision_config=w1_pcg,
        gammas=routing_data.gate_scal if apply_router_weight_on_input else None,
        fused_activation=act,
        y=intermediate_cache,
    )

    matmul_ogs(
        intermediate_cache.view(M * n_expts_act, N // 2),
        w2,
        b2,
        routing_data,
        scatter_indx=scatter_indx,
        precision_config=w2_pcg,
        gammas=None if apply_router_weight_on_input else routing_data.gate_scal,
        y=output,
    )
    return output.view(M, K)
```
**EN:** This block defines `triton_kernel_fused_experts_with_bias` and contains the main logic for this step. It mainly invokes `FusedActivation`, `torch.empty`, `matmul_ogs`, `output.view`, and `dict`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `M`, `K`, `E`, `_`, and `N` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `triton_kernel_fused_experts_with_bias`，并承载这一阶段的核心逻辑。 它主要调用 `FusedActivation`、`torch.empty`、`matmul_ogs`、`output.view` 以及 `dict`，说明该流程会编排底层辅助函数或计算内核。 像 `M`、`K`、`E`、`_` 以及 `N` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `quantize`, `triton_kernel_moe_forward`, `triton_kernel_fused_experts`, `triton_kernel_moe_with_bias_forward`, and `triton_kernel_fused_experts_with_bias`. / **主要符号**：核心入口包括 `quantize`、`triton_kernel_moe_forward`、`triton_kernel_fused_experts`、`triton_kernel_moe_with_bias_forward` 以及 `triton_kernel_fused_experts_with_bias`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `typing.TYPE_CHECKING`, and `typing.Optional` / **标准库**：`__future__.annotations`、`typing.TYPE_CHECKING` 以及 `typing.Optional`
- **Third-party**: `torch`, `triton_kernels.matmul_ogs.FlexCtx`, `triton_kernels.matmul_ogs.FnSpecs`, `triton_kernels.matmul_ogs.FusedActivation`, `triton_kernels.matmul_ogs.GatherIndx`, `triton_kernels.matmul_ogs.PrecisionConfig`, `triton_kernels.matmul_ogs.RoutingData`, `triton_kernels.matmul_ogs.ScatterIndx`, `triton_kernels.matmul_ogs.matmul_ogs`, and `triton_kernels.numerics.InFlexData` / **第三方依赖**：`torch`、`triton_kernels.matmul_ogs.FlexCtx`、`triton_kernels.matmul_ogs.FnSpecs`、`triton_kernels.matmul_ogs.FusedActivation`、`triton_kernels.matmul_ogs.GatherIndx`、`triton_kernels.matmul_ogs.PrecisionConfig`、`triton_kernels.matmul_ogs.RoutingData`、`triton_kernels.matmul_ogs.ScatterIndx`、`triton_kernels.matmul_ogs.matmul_ogs` 以及 `triton_kernels.numerics.InFlexData`
- **Internal SGLang modules**: `sglang.srt.utils.is_cuda`, `sglang.jit_kernel.activation.gelu_and_mul`, `sglang.jit_kernel.activation.silu_and_mul`, `sglang.srt.layers.moe.moe_runner.MoeRunnerConfig`, `sglang.srt.layers.moe.topk.TopKOutput`, and `sglang.srt.layers.moe.topk.TopKOutputChecker` / **SGLang 内部模块**：`sglang.srt.utils.is_cuda`、`sglang.jit_kernel.activation.gelu_and_mul`、`sglang.jit_kernel.activation.silu_and_mul`、`sglang.srt.layers.moe.moe_runner.MoeRunnerConfig`、`sglang.srt.layers.moe.topk.TopKOutput` 以及 `sglang.srt.layers.moe.topk.TopKOutputChecker`
