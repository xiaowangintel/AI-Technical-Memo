# fused_marlin_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/fused_moe_triton/fused_marlin_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with activation operators and fused elementwise paths for the SGLang SRT runtime. It exposes symbols such as `get_scalar_type`, `swiglu_limit_func`, and `fused_marlin_moe` and connects them to backend-specific paths such as `CUDA`, `Triton`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合激活算子与融合逐元素路径。它提供了 `get_scalar_type`、`swiglu_limit_func` 以及 `fused_marlin_moe` 等符号，并把这些符号连接到 `CUDA`、`Triton` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports, conditional backend setup, and runtime guards
```python
from typing import Optional

import torch
import torch.nn.functional as F

from sglang.srt.utils import is_cuda
from sglang.srt.utils.custom_op import register_custom_op

_is_cuda = is_cuda()

if _is_cuda:
    from sgl_kernel import moe_sum_reduce

    from sglang.jit_kernel.activation import silu_and_mul
    from sglang.jit_kernel.moe_wna16_marlin import moe_wna16_marlin_gemm
```
**EN:** This section prepares the module namespace. It imports `typing.Optional`, `torch`, `torch.nn.functional`, `sglang.srt.utils.is_cuda`, `sglang.srt.utils.custom_op.register_custom_op`, and `sgl_kernel.moe_sum_reduce`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_cuda` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `typing.Optional`、`torch`、`torch.nn.functional`、`sglang.srt.utils.is_cuda`、`sglang.srt.utils.custom_op.register_custom_op` 以及 `sgl_kernel.moe_sum_reduce`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_cuda` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 18-34: `get_scalar_type` getter for scalar type
```python
def get_scalar_type(num_bits: int, has_zp: bool, scales: Optional[torch.Tensor] = None):
    from sgl_kernel.scalar_type import scalar_types

    if (
        not has_zp
        and num_bits == 4
        and scales is not None
        and scales.dtype == torch.float8_e8m0fnu
    ):
        return scalar_types.float4_e2m1f
    if has_zp:
        assert num_bits == 4
        return scalar_types.uint4
    else:
        return scalar_types.uint4b8 if num_bits == 4 else scalar_types.uint8b128
```
**EN:** This block defines `get_scalar_type` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_scalar_type`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 35-50: Function `swiglu_limit_func` and its core logic
```python
def swiglu_limit_func(
    output: torch.Tensor,
    input: torch.Tensor,  # first half is gate, second half is up
    swiglu_limit: float = 0.0,
) -> None:
    d = input.shape[1] // 2
    gate = input[:, :d]
    up = input[:, d:]

    if swiglu_limit > 0:
        gate = torch.clamp(gate, max=swiglu_limit)
        up = torch.clamp(up, min=-swiglu_limit, max=swiglu_limit)

    output.copy_(F.silu(gate) * up)
```
**EN:** This block defines `swiglu_limit_func` and contains the main logic for this step. It mainly invokes `output.copy_`, `torch.clamp`, and `F.silu`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `d`, `gate`, and `up` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `swiglu_limit_func`，并承载这一阶段的核心逻辑。 它主要调用 `output.copy_`、`torch.clamp` 以及 `F.silu`，说明该流程会编排底层辅助函数或计算内核。 像 `d`、`gate` 以及 `up` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 51-269: Function `fused_marlin_moe` and its core logic
```python
@register_custom_op(out_shape="hidden_states")
def fused_marlin_moe(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    w1_scale: torch.Tensor,
    w2_scale: torch.Tensor,
    gating_output: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    global_num_experts: int = -1,
    expert_map: Optional[torch.Tensor] = None,
    g_idx1: Optional[torch.Tensor] = None,
    g_idx2: Optional[torch.Tensor] = None,
    sort_indices1: Optional[torch.Tensor] = None,
    sort_indices2: Optional[torch.Tensor] = None,
    w1_zeros: Optional[torch.Tensor] = None,
    w2_zeros: Optional[torch.Tensor] = None,
    workspace: Optional[torch.Tensor] = None,
    num_bits: int = 8,
    is_k_full: bool = True,
    inplace: bool = False,
    routed_scaling_factor: Optional[float] = None,
    clamp_limit: Optional[float] = None,
) -> torch.Tensor:
    """
    This function computes a Mixture of Experts (MoE) layer using two sets of
    weights, w1 and w2, and top-k gating mechanism.

    Parameters:
    - hidden_states (torch.Tensor): The input tensor to the MoE layer.
    - w1 (torch.Tensor): The first set of expert weights.
    - w2 (torch.Tensor): The second set of expert weights.
    - w1_scale (torch.Tensor): Scale to be used for w1.
    - w2_scale (torch.Tensor): Scale to be used for w2.
    - gating_output (torch.Tensor): The output of the gating operation
        (before softmax).
    - g_idx1 (Optional[torch.Tensor]): The first set of act_order indices.
    - g_idx2 (Optional[torch.Tensor]): The second set of act_order indices.
    - sort_indices1 (Optional[torch.Tensor]): The first act_order input
        permutation.
    - sort_indices2 (Optional[torch.Tensor]): The second act_order input
        permutation.
    - topk_weights (torch.Tensor): Top-k weights.
    - topk_ids (torch.Tensor): Indices of topk-k elements.
    - w1_zeros (Optional[torch.Tensor]): Optional zero points to be used for w1.
    - w2_zeros (Optional[torch.Tensor]): Optional zero points to be used for w2.
    - num_bits (int): The number of bits in expert weights quantization.

    Returns:
    - torch.Tensor: The output tensor after applying the MoE layer.
    """
    from sglang.srt.layers.moe.fused_moe_triton import moe_align_block_size

    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"
    assert hidden_states.shape[1] == w1.shape[1] * 16, "Hidden size mismatch w1"
    assert hidden_states.shape[1] == w2.shape[2] // (
        num_bits // 2
    ), "Hidden size mismatch w2"
    assert hidden_states.is_contiguous(), "Hidden_states must be contiguous"
    assert w1.is_contiguous(), "Expert weights1 must be contiguous"
    assert w2.is_contiguous(), "Expert weights2 must be contiguous"
    assert hidden_states.dtype in [torch.float16, torch.bfloat16]
    is_mxfp4_marlin = (
        num_bits == 4
        and w1_zeros is None
        and w2_zeros is None
        and w1_scale.dtype == torch.float8_e8m0fnu
        and w2_scale.dtype == torch.float8_e8m0fnu
    )
    if is_mxfp4_marlin:
        assert hidden_states.dtype == torch.bfloat16, (
            "MXFP4 Marlin with E8M0 scales is only instantiated for bfloat16 "
            f"activations, got {hidden_states.dtype}"
        )
    else:
        assert (
            hidden_states.dtype == w1_scale.dtype
        ), f"moe_wna16_marlin_gemm assumes hidden_states.dtype ({hidden_states.dtype}) == w1_scale.dtype ({w1_scale.dtype})"
        assert (
            hidden_states.dtype == w2_scale.dtype
        ), f"moe_wna16_marlin_gemm assumes hidden_states.dtype ({hidden_states.dtype}) == w2_scale.dtype ({w2_scale.dtype})"
    assert num_bits in [4, 8]

    M, K = hidden_states.shape
    E = w1.shape[0]
    N = w2.shape[1] * 16
    topk = topk_ids.shape[1]

    # M block size selection logic
    # TODO: tune this further for specific models
    for block_size_m in [8, 16, 32, 48, 64]:
        if M * topk / E / block_size_m < 0.9:
            break

    if global_num_experts == -1:
        global_num_experts = E
    sorted_token_ids, expert_ids, num_tokens_post_padded = moe_align_block_size(
        topk_ids, block_size_m, global_num_experts
    )

    if workspace is None:
        max_workspace_size = (max(2 * N, K) // 64) * (
            sorted_token_ids.size(0) // block_size_m
        )
        device = hidden_states.device
        sms = torch.cuda.get_device_properties(device).multi_processor_count
        max_workspace_size = min(max_workspace_size, sms * 4)
        workspace = torch.zeros(
            max_workspace_size, dtype=torch.int, device=device, requires_grad=False
        )

    scalar_type1 = get_scalar_type(num_bits, w1_zeros is not None, w1_scale)
    scalar_type2 = get_scalar_type(num_bits, w2_zeros is not None, w2_scale)

    intermediate_cache2 = torch.empty(
        (M * topk_ids.shape[1], N),
        device=hidden_states.device,
        dtype=hidden_states.dtype,
    )
    intermediate_cache13 = torch.empty(
        (M * topk_ids.shape[1] * max(2 * N, K),),
        device=hidden_states.device,
        dtype=hidden_states.dtype,
    )
    intermediate_cache1 = intermediate_cache13[: M * topk_ids.shape[1] * 2 * N]
    intermediate_cache1 = intermediate_cache1.view(-1, 2 * N)
    intermediate_cache3 = intermediate_cache13[: M * topk_ids.shape[1] * K]
    intermediate_cache3 = intermediate_cache3.view(-1, K)

    use_atomic_add = (
        hidden_states.dtype == torch.half
        or torch.cuda.get_device_capability(hidden_states.device)[0] >= 9
    ) and (not is_mxfp4_marlin)

    intermediate_cache1 = moe_wna16_marlin_gemm(
        hidden_states,
        intermediate_cache1,
        w1,
        None,  # b_bias_or_none
        w1_scale,
        None,  # global_scale_or_none
        w1_zeros,
        g_idx1,
        sort_indices1,
        workspace,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        topk_weights,
        moe_block_size=block_size_m,
        top_k=topk,
        mul_topk_weights=False,
        is_ep=expert_map is not None,
        b_q_type=scalar_type1,
        size_m=M,
        size_n=2 * N,
        size_k=K,
        is_k_full=is_k_full,
        use_atomic_add=use_atomic_add,
        use_fp32_reduce=True,
        is_zp_float=False,
    )

    if clamp_limit is not None:
        swiglu_limit_func(
            intermediate_cache2,
            intermediate_cache1.view(-1, 2 * N),
            clamp_limit,
        )
    else:
        silu_and_mul(intermediate_cache1.view(-1, 2 * N), intermediate_cache2)

    if expert_map is not None:
        intermediate_cache3.zero_()

    intermediate_cache3 = moe_wna16_marlin_gemm(
        intermediate_cache2,
        intermediate_cache3,
        w2,
        None,  # b_bias_or_none
        w2_scale,
        None,  # global_scale_or_none
        w2_zeros,
        g_idx2,
        sort_indices2,
        workspace,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        topk_weights,
        moe_block_size=block_size_m,
        top_k=1,
        mul_topk_weights=True,
        is_ep=expert_map is not None,
        b_q_type=scalar_type2,
        size_m=M * topk,
        size_n=K,
        size_k=N,
        is_k_full=is_k_full,
        use_atomic_add=use_atomic_add,
        use_fp32_reduce=True,
        is_zp_float=False,
    ).view(-1, topk, K)

    output = hidden_states if inplace else torch.empty_like(hidden_states)

    if is_mxfp4_marlin:
        return torch.sum(intermediate_cache3, dim=1, out=output)
    else:
        if routed_scaling_factor is None:
            routed_scaling_factor = 1.0

        moe_sum_reduce(
            intermediate_cache3,
            output,
            routed_scaling_factor,
        )
        return output
```
**EN:** This block defines `fused_marlin_moe` and contains the main logic for this step. Decorators like `register_custom_op` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_custom_op`, `hidden_states.is_contiguous`, `w1.is_contiguous`, `w2.is_contiguous`, and `moe_align_block_size`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `is_mxfp4_marlin`, `M`, `K`, `E`, and `N` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_marlin_moe`，并承载这一阶段的核心逻辑。 像 `register_custom_op` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_custom_op`、`hidden_states.is_contiguous`、`w1.is_contiguous`、`w2.is_contiguous` 以及 `moe_align_block_size`，说明该流程会编排底层辅助函数或计算内核。 像 `is_mxfp4_marlin`、`M`、`K`、`E` 以及 `N` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `get_scalar_type`, `swiglu_limit_func`, and `fused_marlin_moe`. / **主要符号**：核心入口包括 `get_scalar_type`、`swiglu_limit_func` 以及 `fused_marlin_moe`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `typing.Optional` / **标准库**：`typing.Optional`
- **Third-party**: `torch`, `torch.nn.functional`, `sgl_kernel.moe_sum_reduce`, and `sgl_kernel.scalar_type.scalar_types` / **第三方依赖**：`torch`、`torch.nn.functional`、`sgl_kernel.moe_sum_reduce` 以及 `sgl_kernel.scalar_type.scalar_types`
- **Internal SGLang modules**: `sglang.srt.utils.is_cuda`, `sglang.srt.utils.custom_op.register_custom_op`, `sglang.jit_kernel.activation.silu_and_mul`, `sglang.jit_kernel.moe_wna16_marlin.moe_wna16_marlin_gemm`, and `sglang.srt.layers.moe.fused_moe_triton.moe_align_block_size` / **SGLang 内部模块**：`sglang.srt.utils.is_cuda`、`sglang.srt.utils.custom_op.register_custom_op`、`sglang.jit_kernel.activation.silu_and_mul`、`sglang.jit_kernel.moe_wna16_marlin.moe_wna16_marlin_gemm` 以及 `sglang.srt.layers.moe.fused_moe_triton.moe_align_block_size`
