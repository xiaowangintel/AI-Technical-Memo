# moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/moe.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `Mixture-of-Experts`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `Mixture-of-Experts` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Imports and module setup
````python
from typing import Optional

import torch
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 6-25: `moe_align_block_size` definition
````python
def moe_align_block_size(
    topk_ids,
    num_experts,
    block_size,
    sorted_token_ids,
    experts_ids,
    num_tokens_post_pad,
    cumsum_buffer,
    pad_sorted_token_ids=False,
):
    torch.ops.sgl_kernel.moe_align_block_size.default(
        topk_ids,
        num_experts,
        block_size,
        sorted_token_ids,
        experts_ids,
        num_tokens_post_pad,
        cumsum_buffer,
        pad_sorted_token_ids,
    )
````
**EN:** This section defines `moe_align_block_size` and implements the core logic associated with MoE alignment block size. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `moe_align_block_size`，并实现与 MoE alignment block size 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 28-54: `topk_softmax` definition
````python
def topk_softmax(
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    gating_output: torch.Tensor,
    renormalize: bool = False,
    moe_softcapping: float = 0.0,
    correction_bias: Optional[torch.Tensor] = None,
) -> None:
    """
    Compute top-k softmax for MoE routing.

    Args:
        topk_weights: Output tensor for top-k weights [num_tokens, topk]
        topk_ids: Output tensor for top-k expert indices [num_tokens, topk]
        gating_output: Gating logits [num_tokens, num_experts]
        renormalize: Whether to renormalize the top-k weights
        moe_softcapping: Tanh softcapping value (0.0 to disable)
        correction_bias: Per-expert bias correction [num_experts], must be float32 if provided
    """
    torch.ops.sgl_kernel.topk_softmax.default(
        topk_weights,
        topk_ids,
        gating_output,
        renormalize,
        moe_softcapping,
        correction_bias,
    )
````
**EN:** This section defines `topk_softmax` and implements the core logic associated with topk softmax. Docstring summary: Compute top-k softmax for MoE routing. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `topk_softmax`，并实现与 topk softmax 相关的核心逻辑。 文档字符串摘要：Compute top-k softmax for MoE routing. 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 57-80: `topk_sigmoid` definition
````python
def topk_sigmoid(
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    gating_output: torch.Tensor,
    renormalize: bool = False,
    correction_bias: Optional[torch.Tensor] = None,
) -> None:
    """
    Compute top-k sigmoid for MoE routing.

    Args:
        topk_weights: Output tensor for top-k weights [num_tokens, topk]
        topk_ids: Output tensor for top-k expert indices [num_tokens, topk]
        gating_output: Gating logits [num_tokens, num_experts]
        renormalize: Whether to renormalize the top-k weights
        correction_bias: Per-expert bias correction [num_experts], must be float32 if provided
    """
    torch.ops.sgl_kernel.topk_sigmoid.default(
        topk_weights,
        topk_ids,
        gating_output,
        renormalize,
        correction_bias,
    )
````
**EN:** This section defines `topk_sigmoid` and implements the core logic associated with topk sigmoid. Docstring summary: Compute top-k sigmoid for MoE routing. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `topk_sigmoid`，并实现与 topk sigmoid 相关的核心逻辑。 文档字符串摘要：Compute top-k sigmoid for MoE routing. 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 83-92: `moe_sum_reduce` definition
````python
def moe_sum_reduce(
    input_tensor,
    output_tensor,
    routed_scaling_factor=0,
):
    torch.ops.sgl_kernel.moe_sum_reduce.default(
        input_tensor,
        output_tensor,
        routed_scaling_factor,
    )
````
**EN:** This section defines `moe_sum_reduce` and implements the core logic associated with MoE sum reduce. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `moe_sum_reduce`，并实现与 MoE sum reduce 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 95-102: `moe_sum` definition
````python
def moe_sum(
    input_tensor: torch.Tensor,
    output_tensor: torch.Tensor,
):
    torch.ops.sgl_kernel.moe_sum.default(
        input_tensor,
        output_tensor,
    )
````
**EN:** This section defines `moe_sum` and implements the core logic associated with MoE sum. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `moe_sum`，并实现与 MoE sum 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 105-137: `moe_fused_gate` definition
````python
def moe_fused_gate(
    input_tensor,
    bias,
    num_expert_group,
    topk_group,
    topk,
    num_fused_shared_experts=0,
    routed_scaling_factor=0,
    apply_routed_scaling_factor_on_output=False,
):
    # This fused kernel function is used to select topk expert in a hierarchical 2-layer fashion
    # it split group of expert into num_expert_group, and use top2 expert weight sum in each group
    # as the group weight to select expert groups and then select topk experts within the selected groups
    # the #experts is decided by the input tensor shape and we currently only support power of 2 #experts
    # and #experts should be divisible by num_expert_group. #expert/num_expert_group <= 32 is limited for now.
    # for non-supported case, we suggest to use the biased_grouped_topk func in sglang.srt.layers.moe.topk
    # num_fused_shared_experts: if > 0, the last several experts will be
    #   replaced with shared experts. the shared experts will be divided by the
    #   routed_scaling_factor - this is intended to cancel out later when routed+shared
    #   output is scaled so that shared experts are not scaled.
    # routed_scaling_factor: if > 0, the experts will be scaled by this factor
    # apply_routed_scaling_factor_on_output: if true, output will be
    #   scaled by the routed_scaling_factor
    return torch.ops.sgl_kernel.moe_fused_gate.default(
        input_tensor,
        bias,
        num_expert_group,
        topk_group,
        topk,
        num_fused_shared_experts,
        routed_scaling_factor,
        apply_routed_scaling_factor_on_output,
    )
````
**EN:** This section defines `moe_fused_gate` and implements the core logic associated with MoE fused gate. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `moe_fused_gate`，并实现与 MoE fused gate 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 140-172: `kimi_k2_moe_fused_gate` definition
````python
def kimi_k2_moe_fused_gate(
    input_tensor,
    bias,
    topk,
    renormalize=True,
    routed_scaling_factor=1.0,
    apply_routed_scaling_factor_on_output=False,
):
    """
    Simplified fused kernel for Kimi K2 model (num_expert_group=1).
    This kernel removes the grouped topk logic since all experts belong to a single group.

    Args:
        input_tensor: Gating output tensor [num_tokens, num_experts]
        bias: Correction bias tensor [num_experts]
        topk: Number of experts to select per token
        renormalize: Whether to renormalize the topk weights
        routed_scaling_factor: Scaling factor for expert weights
        apply_routed_scaling_factor_on_output: If true, apply scaling factor to output

    Returns:
        Tuple of (topk_weights, topk_ids)
        - topk_weights: [num_tokens, topk] float32 tensor
        - topk_ids: [num_tokens, topk] int32 tensor
    """
    return torch.ops.sgl_kernel.kimi_k2_moe_fused_gate.default(
        input_tensor,
        bias,
        topk,
        renormalize,
        routed_scaling_factor,
        apply_routed_scaling_factor_on_output,
    )
````
**EN:** This section defines `kimi_k2_moe_fused_gate` and implements the core logic associated with Kimi k2 MoE fused gate. Docstring summary: Simplified fused kernel for Kimi K2 model (num_expert_group=1). The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `kimi_k2_moe_fused_gate`，并实现与 Kimi k2 MoE fused gate 相关的核心逻辑。 文档字符串摘要：Simplified fused kernel for Kimi K2 model (num_expert_group=1). 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 175-214: `fp8_blockwise_scaled_grouped_mm` definition
````python
def fp8_blockwise_scaled_grouped_mm(
    output,
    a_ptrs,
    b_ptrs,
    out_ptrs,
    a_scales_ptrs,
    b_scales_ptrs,
    a,
    b,
    scales_a,
    scales_b,
    stride_a,
    stride_b,
    stride_c,
    layout_sfa,
    layout_sfb,
    problem_sizes,
    expert_offsets,
    workspace,
):
    torch.ops.sgl_kernel.fp8_blockwise_scaled_grouped_mm.default(
        output,
        a_ptrs,
        b_ptrs,
        out_ptrs,
        a_scales_ptrs,
        b_scales_ptrs,
        a,
        b,
        scales_a,
        scales_b,
        stride_a,
        stride_b,
        stride_c,
        layout_sfa,
        layout_sfb,
        problem_sizes,
        expert_offsets,
        workspace,
    )
````
**EN:** This section defines `fp8_blockwise_scaled_grouped_mm` and implements the core logic associated with FP8 blockwise scaled grouped mm. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `fp8_blockwise_scaled_grouped_mm`，并实现与 FP8 blockwise scaled grouped mm 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 217-240: `prepare_moe_input` definition
````python
def prepare_moe_input(
    topk_ids,
    expert_offsets,
    problem_sizes1,
    problem_sizes2,
    input_permutation,
    output_permutation,
    num_experts,
    n,
    k,
    blockscale_offsets: Optional[torch.Tensor] = None,
):
    torch.ops.sgl_kernel.prepare_moe_input.default(
        topk_ids,
        expert_offsets,
        blockscale_offsets,
        problem_sizes1,
        problem_sizes2,
        input_permutation,
        output_permutation,
        num_experts,
        n,
        k,
    )
````
**EN:** This section defines `prepare_moe_input`. It prepares the `MoE input` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `prepare_moe_input`。它负责准备模块中与 `MoE input` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 243-251: `apply_shuffle_mul_sum` definition
````python
def apply_shuffle_mul_sum(
    input,
    output,
    permutation,
    factors,
):
    torch.ops.sgl_kernel.apply_shuffle_mul_sum.default(
        input, output, permutation, factors
    )
````
**EN:** This section defines `apply_shuffle_mul_sum`. It applies the `shuffle multiply sum` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `apply_shuffle_mul_sum`。它负责应用模块中与 `shuffle multiply sum` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 254-289: `fused_qk_norm_rope` definition
````python
def fused_qk_norm_rope(
    qkv: torch.Tensor,
    num_heads_q: int,
    num_heads_k: int,
    num_heads_v: int,
    head_dim: int,
    eps: float,
    q_weight: torch.Tensor,
    k_weight: torch.Tensor,
    base: float,
    is_neox: bool,
    position_ids: torch.Tensor,
    factor: float,
    low: float,
    high: float,
    attention_factor: float,
    rotary_dim: Optional[int] = None,
) -> None:
    torch.ops.sgl_kernel.fused_qk_norm_rope(
        qkv,
        num_heads_q,
        num_heads_k,
        num_heads_v,
        head_dim,
        eps,
        q_weight,
        k_weight,
        base,
        is_neox,
        position_ids,
        factor,
        low,
        high,
        attention_factor,
        rotary_dim if rotary_dim is not None else head_dim,
    )
````
**EN:** This section defines `fused_qk_norm_rope` and implements the core logic associated with fused qk normalization RoPE. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `fused_qk_norm_rope`，并实现与 fused qk normalization RoPE 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `moe_align_block_size`, `topk_softmax`, `topk_sigmoid`, `moe_sum_reduce`, `moe_sum`, `moe_fused_gate`, `kimi_k2_moe_fused_gate`, `fp8_blockwise_scaled_grouped_mm`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `torch`, `typing`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
