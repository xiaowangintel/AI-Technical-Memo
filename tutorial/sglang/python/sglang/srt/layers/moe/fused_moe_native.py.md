# fused_moe_native.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/fused_moe_native.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `fused_moe_forward_native` and `moe_forward_native` and connects them to backend-specific paths such as `Triton`, `NPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `fused_moe_forward_native` 和 `moe_forward_native` 等符号，并把这些符号连接到 `Triton`、`NPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Module imports and dependency wiring
```python
"""
Torch-native implementation for FusedMoE. This is used for torch.compile.
It is based on https://github.com/pytorch-labs/gpt-fast/blob/32971d3129541c5bfb4f715abc33d1c5f408d204/mixtral-moe/model.py#L204
"""

import torch
from torch.nn import functional as F

from sglang.srt.layers.activation import GeluAndMul, SiluAndMul
from sglang.srt.layers.moe.moe_runner import MoeRunnerConfig
from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import (
    swiglu_gpt_oss_sigmoid_alpha,
)
from sglang.srt.layers.moe.token_dispatcher import (
    StandardCombineInput,
    StandardDispatchOutput,
)
from sglang.srt.layers.moe.topk import StandardTopKOutput
```
**EN:** This section prepares the module namespace. It imports `torch`, `torch.nn.functional`, `sglang.srt.layers.activation.GeluAndMul`, `sglang.srt.layers.activation.SiluAndMul`, `sglang.srt.layers.moe.moe_runner.MoeRunnerConfig`, and `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe.swiglu_gpt_oss_sigmoid_alpha`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `torch`、`torch.nn.functional`、`sglang.srt.layers.activation.GeluAndMul`、`sglang.srt.layers.activation.SiluAndMul`、`sglang.srt.layers.moe.moe_runner.MoeRunnerConfig` 以及 `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe.swiglu_gpt_oss_sigmoid_alpha`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 21-51: Function `fused_moe_forward_native` and its core logic
```python
def fused_moe_forward_native(
    layer: torch.nn.Module,
    dispatch_output: StandardDispatchOutput,
) -> StandardCombineInput:

    x, x_scale, topk_output = dispatch_output
    moe_runner_config = layer.moe_runner_config

    if moe_runner_config.apply_router_weight_on_input:
        raise NotImplementedError()

    topk_weights, topk_ids, _ = topk_output

    w13_weights = layer.w13_weight[topk_ids]
    w1_weights, w3_weights = torch.chunk(w13_weights, 2, dim=2)
    w2_weights = layer.w2_weight[topk_ids]
    x1 = torch.einsum("ti,taoi -> tao", x, w1_weights)
    if moe_runner_config.activation == "silu":
        x1 = F.silu(x1)
    elif moe_runner_config.activation == "gelu":
        x1 = F.gelu(x1)
    else:
        raise ValueError(f"Unsupported activation: {moe_runner_config.activation=}")
    x3 = torch.einsum("ti, taoi -> tao", x, w3_weights)
    expert_outs = torch.einsum("tao, taio -> tai", (x1 * x3), w2_weights)
    expert_outs = torch.einsum(
        "tai,ta -> ti", expert_outs, topk_weights.to(expert_outs.dtype)
    )
    return StandardCombineInput(hidden_states=expert_outs)
```
**EN:** This block defines `fused_moe_forward_native` and contains the main logic for this step. It mainly invokes `torch.chunk`, `torch.einsum`, `StandardCombineInput`, `NotImplementedError`, and `F.silu`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x`, `x_scale`, `topk_output`, `moe_runner_config`, and `topk_weights` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_moe_forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `torch.chunk`、`torch.einsum`、`StandardCombineInput`、`NotImplementedError` 以及 `F.silu`，说明该流程会编排底层辅助函数或计算内核。 像 `x`、`x_scale`、`topk_output`、`moe_runner_config` 以及 `topk_weights` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 52-147: Function `moe_forward_native` and its core logic
```python
def moe_forward_native(
    layer: torch.nn.Module,
    x: torch.Tensor,
    topk_output: StandardTopKOutput,
    moe_runner_config: MoeRunnerConfig,
) -> torch.Tensor:

    if moe_runner_config.apply_router_weight_on_input:
        raise NotImplementedError()

    topk_weights, topk_ids, _ = topk_output

    # Ref code from https://huggingface.co/deepseek-ai/DeepSeek-V2/blob/e0828e3cc0a03408724b80c3cc92c8e072db8d01/modeling_deepseek.py#L589
    len_experts = layer.num_experts

    cnts = topk_ids.new_zeros((topk_ids.shape[0], len_experts))
    cnts.scatter_(1, topk_ids.to(torch.int64), 1)
    tokens_per_expert = cnts.sum(dim=0)
    idxs = topk_ids.view(-1).argsort()

    sorted_tokens = x[idxs // topk_ids.shape[1]]
    tokens_per_expert = tokens_per_expert.cpu().numpy()

    if moe_runner_config.activation == "silu":
        act = SiluAndMul()
    elif moe_runner_config.activation == "gelu":
        act = GeluAndMul()
    else:
        raise ValueError(f"Unsupported activation: {moe_runner_config.activation=}")

    # Get bias terms if available
    w13_bias = getattr(layer, "w13_weight_bias", None)
    w2_bias = getattr(layer, "w2_weight_bias", None)
    outputs = []
    start_idx = 0
    for i, num_tokens in enumerate(tokens_per_expert):
        end_idx = start_idx + num_tokens
        if num_tokens == 0:
            continue
        tokens_for_this_expert = sorted_tokens[start_idx:end_idx]

        layer_w13_weight = layer.w13_weight[i]
        layer_w2_weight = layer.w2_weight[i]

        # Store original dtype
        original_dtype = tokens_for_this_expert.dtype

        # Get bias terms if available for this expert
        layer_w13_bias = w13_bias[i] if w13_bias is not None else None
        layer_w2_bias = w2_bias[i] if w2_bias is not None else None

        # Apply w13 linear
        gate_up = F.linear(tokens_for_this_expert, layer_w13_weight)

        # Add bias if present (for models like GPT-OSS)
        if layer_w13_bias is not None:
            gate_up_fp32 = gate_up.float() + layer_w13_bias
            gate_up = gate_up_fp32.to(original_dtype)

        # Apply activation
        if (
            moe_runner_config.activation == "silu"
            and moe_runner_config.gemm1_alpha is not None
        ):
            assert moe_runner_config.gemm1_clamp_limit is not None
            gate_up = swiglu_gpt_oss_sigmoid_alpha(
                gate_up,
                moe_runner_config.gemm1_alpha,
                moe_runner_config.gemm1_clamp_limit,
            )
        else:
            gate_up = act(gate_up)

        # Apply w2 linear
        expert_out = F.linear(gate_up, layer_w2_weight)

        # Add bias if present (for models like GPT-OSS)
        if layer_w2_bias is not None:
            expert_out = expert_out.float() + layer_w2_bias
            expert_out = expert_out.to(original_dtype)

        outputs.append(expert_out)
        start_idx = end_idx

    outs = torch.cat(outputs, dim=0) if len(outputs) else sorted_tokens.new_empty(0)
    new_x = torch.empty_like(outs)

    new_x[idxs] = outs
    final_out = (
        new_x.view(*topk_ids.shape, -1)
        .type(topk_weights.dtype)
        .mul_(topk_weights.unsqueeze(dim=-1))
        .sum(dim=1)
        .type(new_x.dtype)
    )
    return final_out
```
**EN:** This block defines `moe_forward_native` and contains the main logic for this step. It mainly invokes `topk_ids.new_zeros`, `cnts.scatter_`, `cnts.sum`, `topk_ids.view.argsort`, and `tokens_per_expert.cpu.numpy`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_weights`, `topk_ids`, `_`, `len_experts`, and `cnts` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `moe_forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `topk_ids.new_zeros`、`cnts.scatter_`、`cnts.sum`、`topk_ids.view.argsort` 以及 `tokens_per_expert.cpu.numpy`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_weights`、`topk_ids`、`_`、`len_experts` 以及 `cnts` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `fused_moe_forward_native` and `moe_forward_native`. / **主要符号**：核心入口包括 `fused_moe_forward_native` 和 `moe_forward_native`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Activation math**: Captures fused nonlinear transforms and their device-specific fast paths. / **激活数学**：描述融合非线性变换及其设备专用快速路径。

## Dependencies / 依赖关系
- **Third-party**: `torch` and `torch.nn.functional` / **第三方依赖**：`torch` 和 `torch.nn.functional`
- **Internal SGLang modules**: `sglang.srt.layers.activation.GeluAndMul`, `sglang.srt.layers.activation.SiluAndMul`, `sglang.srt.layers.moe.moe_runner.MoeRunnerConfig`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe.swiglu_gpt_oss_sigmoid_alpha`, `sglang.srt.layers.moe.token_dispatcher.StandardCombineInput`, `sglang.srt.layers.moe.token_dispatcher.StandardDispatchOutput`, and `sglang.srt.layers.moe.topk.StandardTopKOutput` / **SGLang 内部模块**：`sglang.srt.layers.activation.GeluAndMul`、`sglang.srt.layers.activation.SiluAndMul`、`sglang.srt.layers.moe.moe_runner.MoeRunnerConfig`、`sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe.swiglu_gpt_oss_sigmoid_alpha`、`sglang.srt.layers.moe.token_dispatcher.StandardCombineInput`、`sglang.srt.layers.moe.token_dispatcher.StandardDispatchOutput` 以及 `sglang.srt.layers.moe.topk.StandardTopKOutput`
