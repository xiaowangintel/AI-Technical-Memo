# rocm_moe_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/rocm_moe_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with activation operators and fused elementwise paths for the SGLang SRT runtime. It exposes symbols such as `ActivationMethod`, `rocm_aiter_asm_moe_tkw1`, `rocm_fused_experts_tkw1`, and `upscale_kernel` and connects them to backend-specific paths such as `Triton` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合激活算子与融合逐元素路径。它提供了 `ActivationMethod`、`rocm_aiter_asm_moe_tkw1`、`rocm_fused_experts_tkw1` 以及 `upscale_kernel` 等符号，并把这些符号连接到 `Triton` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: File header and module overview
```python
# Adapted from https://github.com/vllm-project/vllm/blob/v0.9.1rc2/vllm/model_executor/layers/fused_moe/rocm_aiter_fused_moe.py
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 4-17: Imports, constants, and runtime setup
```python
from enum import IntEnum
from typing import Optional

import torch
import triton
import triton.language as tl

from sglang.srt.utils import get_bool_env_var, is_hip
from sglang.srt.utils.custom_op import register_custom_op

_is_hip = is_hip()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This section prepares the module namespace. It imports `enum.IntEnum`, `typing.Optional`, `torch`, `triton`, `triton.language`, and `sglang.srt.utils.get_bool_env_var`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_hip` and `_use_aiter` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `enum.IntEnum`、`typing.Optional`、`torch`、`triton`、`triton.language` 以及 `sglang.srt.utils.get_bool_env_var`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_hip` 和 `_use_aiter` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 18-26: Class `ActivationMethod` declaration and shared state
```python
class ActivationMethod(IntEnum):
    # This allows interfacing with AITER ActivationType enum
    # without importing the ActivationType enum from AITER globally.
    SILU = 0
    GELU = 1


# NOTE: for non _use_aiter case, use lazy registration to avoid overhead
# (registration may not be trigger actually, since it will not be called)
```
**EN:** This block introduces class `ActivationMethod` and the state shared by its methods. It inherits from `IntEnum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `ActivationMethod`，并定义其方法共享的状态。 它继承自 `IntEnum`，说明了它在 SRT 层栈中的接入方式。

### Lines 27-65: Function `rocm_aiter_asm_moe_tkw1` and its core logic
```python
@register_custom_op(out_shape="hidden_states", eager=_use_aiter)
def rocm_aiter_asm_moe_tkw1(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    fc1_scale: Optional[torch.Tensor] = None,
    fc2_scale: Optional[torch.Tensor] = None,
    fc1_smooth_scale: Optional[torch.Tensor] = None,
    fc2_smooth_scale: Optional[torch.Tensor] = None,
    a16: bool = False,
    per_tensor_quant_scale: Optional[torch.Tensor] = None,
    expert_mask: Optional[torch.Tensor] = None,
    activation_method: int = ActivationMethod.SILU.value,
) -> torch.Tensor:

    from aiter import ActivationType
    from aiter.fused_moe_bf16_asm import asm_moe_tkw1

    activation = ActivationType(activation_method)

    return asm_moe_tkw1(
        hidden_states,
        w1,
        w2,
        topk_weights,
        topk_ids,
        fc1_scale=fc1_scale,
        fc2_scale=fc2_scale,
        fc1_smooth_scale=fc1_smooth_scale,
        fc2_smooth_scale=fc2_smooth_scale,
        a16=a16,
        per_tensor_quant_scale=per_tensor_quant_scale,
        expert_mask=expert_mask,
        activation=activation,
    )
```
**EN:** This block defines `rocm_aiter_asm_moe_tkw1` and contains the main logic for this step. Decorators like `register_custom_op` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_custom_op`, `ActivationType`, and `asm_moe_tkw1`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `activation` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `rocm_aiter_asm_moe_tkw1`，并承载这一阶段的核心逻辑。 像 `register_custom_op` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_custom_op`、`ActivationType` 以及 `asm_moe_tkw1`，说明该流程会编排底层辅助函数或计算内核。 像 `activation` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 66-120: Function `rocm_fused_experts_tkw1` and its core logic
```python
def rocm_fused_experts_tkw1(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    activation: str = "silu",
    apply_router_weight_on_input: bool = False,
    use_fp8_w8a8: bool = False,
    per_channel_quant: bool = False,
    w1_scale: Optional[torch.Tensor] = None,
    w2_scale: Optional[torch.Tensor] = None,
    a1_scale: Optional[torch.Tensor] = None,
    a2_scale: Optional[torch.Tensor] = None,
    block_shape: Optional[list[int]] = None,
) -> torch.Tensor:

    activation_method = (
        ActivationMethod.SILU if activation == "silu" else ActivationMethod.GELU
    )
    # All AITER Fused MoE kernels are expecting the following datatypes
    topk_weights = topk_weights.to(torch.float32)
    topk_ids = topk_ids.to(torch.int32)

    # w8a8 per-channel quantization
    if per_channel_quant and apply_router_weight_on_input and use_fp8_w8a8:
        # AITER tkw1 kernel for FP8 models with `apply_router_weight_on_input`
        # This applies topk_weights on the GEMM output of the first FC layer
        #  rather than the second FC.
        assert (
            topk_weights.dim() == 2
        ), "`topk_weights` should be in shape (num_tokens, topk)"
        assert topk_weights.shape[-1] == 1, (
            "Only support topk=1 when" " `apply_router_weight_on_input` is True"
        )

        return rocm_aiter_asm_moe_tkw1(
            hidden_states,
            w1,
            w2,
            topk_weights,
            topk_ids,
            fc1_scale=w1_scale,
            fc2_scale=w2_scale,
            fc1_smooth_scale=None,
            fc2_smooth_scale=None,
            a16=False,
            per_tensor_quant_scale=None,
            expert_mask=None,
            activation_method=activation_method,
        )
    else:
        assert False, "This should not be called."
```
**EN:** This block defines `rocm_fused_experts_tkw1` and contains the main logic for this step. It mainly invokes `topk_weights.to`, `topk_ids.to`, `rocm_aiter_asm_moe_tkw1`, and `topk_weights.dim`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `activation_method`, `topk_weights`, and `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `rocm_fused_experts_tkw1`，并承载这一阶段的核心逻辑。 它主要调用 `topk_weights.to`、`topk_ids.to`、`rocm_aiter_asm_moe_tkw1` 以及 `topk_weights.dim`，说明该流程会编排底层辅助函数或计算内核。 像 `activation_method`、`topk_weights` 以及 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 121-163: Function `upscale_kernel` and its core logic
```python
@triton.jit
def upscale_kernel(
    A_ptr,  # *fp16 / *fp32
    scale_ptr,  # *fp16 / *fp32
    Out_ptr,  # *fp16 / *fp32
    M,
    N,
    recv_token_num,
    stride_am,
    stride_an,
    stride_sm,
    stride_sn,
    stride_om,
    stride_on,
    BLOCK_N: tl.constexpr,
):
    pid_m = tl.program_id(0)  # row id
    pid_n = tl.program_id(1)  # block id along N

    recv_token_num_val = tl.load(recv_token_num)

    if pid_m >= recv_token_num_val:
        return

    # column offsets
    offs_n = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
    mask = offs_n < N

    # A[m, n]
    a_ptrs = A_ptr + pid_m * stride_am + offs_n * stride_an
    a = tl.load(a_ptrs, mask=mask, other=0.0)

    # scale index: n // 128
    scale_idx = offs_n // 128
    s_ptrs = scale_ptr + pid_m * stride_sm + scale_idx * stride_sn
    s = tl.load(s_ptrs, mask=mask, other=1.0)

    out = a * s

    out_ptrs = Out_ptr + pid_m * stride_om + offs_n * stride_on
    tl.store(out_ptrs, out, mask=mask)
```
**EN:** This block defines `upscale_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load`, `tl.store`, and `tl.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid_m`, `pid_n`, `recv_token_num_val`, `offs_n`, and `mask` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `upscale_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load`、`tl.store` 以及 `tl.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `pid_m`、`pid_n`、`recv_token_num_val`、`offs_n` 以及 `mask` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 164-191: Function `upscale` and its core logic
```python
def upscale(hidden_state, hidden_state_scale, recv_token_num, output_dtype):
    M, N = hidden_state.shape

    Out = torch.empty_like(hidden_state, dtype=output_dtype)

    BLOCK_N = 256

    grid = (M, triton.cdiv(N, BLOCK_N))

    upscale_kernel[grid](
        hidden_state,
        hidden_state_scale,
        Out,
        M,
        N,
        recv_token_num,
        hidden_state.stride(0),
        hidden_state.stride(1),
        hidden_state_scale.stride(0),
        hidden_state_scale.stride(1),
        Out.stride(0),
        Out.stride(1),
        BLOCK_N=BLOCK_N,
    )

    return Out
```
**EN:** This block defines `upscale` and contains the main logic for this step. It mainly invokes `torch.empty_like`, `upscale_kernel`, `triton.cdiv`, `hidden_state.stride`, and `hidden_state_scale.stride`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `M`, `N`, `Out`, `BLOCK_N`, and `grid` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `upscale`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty_like`、`upscale_kernel`、`triton.cdiv`、`hidden_state.stride` 以及 `hidden_state_scale.stride`，说明该流程会编排底层辅助函数或计算内核。 像 `M`、`N`、`Out`、`BLOCK_N` 以及 `grid` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 192-276: Function `upscale_fp4x2_block32_kernel` and its core logic
```python
@triton.jit
def upscale_fp4x2_block32_kernel(
    A_u8_ptr,  # *uint8  (view from float4_e2m1fn_x2)
    S_u8_ptr,  # *uint8  (view from float8_e8m0fnu), shape (M, N_fp4/32)
    Out_ptr,  # *fp16/fp32/bf16, shape (M, N_fp4)
    N_FP4: tl.constexpr,
    recv_token_num,
    stride_am,
    stride_an,  # A strides (in uint8 elements) for (M, packed_N)
    stride_sm,
    stride_sn,  # S strides (in uint8 elements) for (M, N_FP4/32)
    stride_om,
    stride_on,  # Out strides (in output elements) for (M, N_FP4)
    BLOCK_N: tl.constexpr,
    OUT_DTYPE: tl.constexpr,  # tl.float16 / tl.float32 / tl.bfloat16
):
    pid_m = tl.program_id(0)
    pid_n = tl.program_id(1)

    recv_token_num_val = tl.load(recv_token_num)
    if pid_m >= recv_token_num_val:
        return

    offs = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
    mask = offs < N_FP4

    # --------------------------
    # Load packed fp4x2 byte
    # --------------------------
    byte_idx = offs >> 1  # offs // 2
    is_hi = (offs & 1) != 0  # select high nibble?

    a_ptrs = A_u8_ptr + pid_m * stride_am + byte_idx * stride_an
    a_byte = tl.load(a_ptrs, mask=mask, other=0).to(tl.int32)

    lo = a_byte & 0xF
    hi = (a_byte >> 4) & 0xF
    code = tl.where(is_hi, hi, lo).to(tl.int32)  # 0..15

    # --------------------------
    # Decode float4_e2m1fn
    # layout: [sign|exp(2)|mant(1)]
    # bias=1, finite-only
    # --------------------------
    sign = (code >> 3) & 0x1
    exp = (code >> 1) & 0x3
    mant = code & 0x1

    mant_f = mant.to(tl.float32) * 0.5
    is_sub = exp == 0

    # normal: 2^(exp-bias) * (1 + mant/2), bias=1
    e_norm = (exp - 1).to(tl.float32)
    val_norm = tl.exp2(e_norm) * (1.0 + mant_f)

    # subnorm/zero: mant/2 * 2^(1-bias) = mant/2
    val_sub = mant_f

    val = tl.where(is_sub, val_sub, val_norm)
    val = tl.where(sign != 0, -val, val)  # apply sign

    # --------------------------
    # Per-token block32 scale: scale_idx = offs // 32
    # scale dtype: float8_e8m0fnu stored in uint8
    # decode: e==0 -> 0
    #         e in [1..254] -> 2^(e-127)
    #         e==255 -> clamp to 254
    # --------------------------
    scale_idx = offs >> 5  # offs // 32

    s_ptrs = S_u8_ptr + pid_m * stride_sm + scale_idx * stride_sn
    e = tl.load(s_ptrs, mask=mask, other=0).to(tl.int32)

    e = tl.minimum(e, 254)  # clamp 255->254
    is_zero = e == 0
    exp_s = (e - 127).to(tl.float32)
    s = tl.exp2(exp_s)
    s = tl.where(is_zero, 0.0, s)

    out = (val * s).to(OUT_DTYPE)

    out_ptrs = Out_ptr + pid_m * stride_om + offs * stride_on
    tl.store(out_ptrs, out, mask=mask)
```
**EN:** This block defines `upscale_fp4x2_block32_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.load`, `tl.load.to`, `tl.where.to`, and `to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid_m`, `pid_n`, `recv_token_num_val`, `offs`, and `mask` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `upscale_fp4x2_block32_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.load`、`tl.load.to`、`tl.where.to` 以及 `to`，说明该流程会编排底层辅助函数或计算内核。 像 `pid_m`、`pid_n`、`recv_token_num_val`、`offs` 以及 `mask` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 277-329: Function `upscale_mxfp4` and its core logic
```python
def upscale_mxfp4(hidden_state, hidden_state_scale, recv_token_num, output_dtype):
    """
    hidden_state: (M, packed_N) torch.float4_e2m1fn_x2
    hidden_state_scale: (M, packed_N*2/32) = (M, N_fp4/32) torch.float8_e8m0fnu
    output: (M, N_fp4) output_dtype
    """
    assert hidden_state.dtype == torch.float4_e2m1fn_x2, hidden_state.dtype
    assert hidden_state_scale.dtype == torch.float8_e8m0fnu, hidden_state_scale.dtype
    assert hidden_state.is_contiguous() or True  # stride-based load OK

    M, packed_N = hidden_state.shape
    N_fp4 = packed_N * 2

    # scale second dim must be N_fp4/32
    assert hidden_state_scale.shape[0] == M
    assert hidden_state_scale.shape[1] == (N_fp4 // 32), (
        hidden_state_scale.shape,
        N_fp4,
    )

    # Triton doesn't (reliably) accept torch.float4/float8 pointers directly.
    # Use raw uint8 views.
    A_u8 = hidden_state.view(torch.uint8)
    S_u8 = hidden_state_scale.view(torch.uint8)

    Out = torch.empty((M, N_fp4), dtype=output_dtype, device=hidden_state.device)

    BLOCK_N = 256
    grid = (M, triton.cdiv(N_fp4, BLOCK_N))

    OUT_TL = (
        tl.float16
        if output_dtype == torch.float16
        else tl.bfloat16 if output_dtype == torch.bfloat16 else tl.float32
    )

    upscale_fp4x2_block32_kernel[grid](
        A_u8,
        S_u8,
        Out,
        N_FP4=N_fp4,
        recv_token_num=recv_token_num,
        stride_am=A_u8.stride(0),
        stride_an=A_u8.stride(1),
        stride_sm=S_u8.stride(0),
        stride_sn=S_u8.stride(1),
        stride_om=Out.stride(0),
        stride_on=Out.stride(1),
        BLOCK_N=BLOCK_N,
        OUT_DTYPE=OUT_TL,
        num_warps=4,
    )
    return Out
```
**EN:** This block defines `upscale_mxfp4` and contains the main logic for this step. It mainly invokes `hidden_state.view`, `hidden_state_scale.view`, `torch.empty`, `upscale_fp4x2_block32_kernel`, and `hidden_state.is_contiguous`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `M`, `packed_N`, `N_fp4`, `A_u8`, and `S_u8` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `upscale_mxfp4`，并承载这一阶段的核心逻辑。 它主要调用 `hidden_state.view`、`hidden_state_scale.view`、`torch.empty`、`upscale_fp4x2_block32_kernel` 以及 `hidden_state.is_contiguous`，说明该流程会编排底层辅助函数或计算内核。 像 `M`、`packed_N`、`N_fp4`、`A_u8` 以及 `S_u8` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `ActivationMethod`, `rocm_aiter_asm_moe_tkw1`, `rocm_fused_experts_tkw1`, `upscale_kernel`, and `upscale`. / **主要符号**：核心入口包括 `ActivationMethod`、`rocm_aiter_asm_moe_tkw1`、`rocm_fused_experts_tkw1`、`upscale_kernel` 以及 `upscale`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `enum.IntEnum` and `typing.Optional` / **标准库**：`enum.IntEnum` 和 `typing.Optional`
- **Third-party**: `torch`, `triton`, `triton.language`, `aiter.ActivationType`, and `aiter.fused_moe_bf16_asm.asm_moe_tkw1` / **第三方依赖**：`torch`、`triton`、`triton.language`、`aiter.ActivationType` 以及 `aiter.fused_moe_bf16_asm.asm_moe_tkw1`
- **Internal SGLang modules**: `sglang.srt.utils.get_bool_env_var`, `sglang.srt.utils.is_hip`, and `sglang.srt.utils.custom_op.register_custom_op` / **SGLang 内部模块**：`sglang.srt.utils.get_bool_env_var`、`sglang.srt.utils.is_hip` 以及 `sglang.srt.utils.custom_op.register_custom_op`
