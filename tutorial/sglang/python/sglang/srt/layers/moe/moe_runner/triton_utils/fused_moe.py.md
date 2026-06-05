# fused_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/moe_runner/triton_utils/fused_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `inplace_fused_experts`, `outplace_fused_experts`, `fused_experts`, and `moe_sum_reduce_torch_compile` and connects them to backend-specific paths such as `CUDA`, `Triton`, `NPU`, and `XPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `inplace_fused_experts`、`outplace_fused_experts`、`fused_experts` 以及 `moe_sum_reduce_torch_compile` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`NPU` 以及 `XPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: File header and module overview
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/a6221a144af772fd1a68fe7e627935dc53e81738/vllm/model_executor/layers/fused_moe/fused_moe.py
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 5-90: Imports, conditional backend setup, and runtime guards
```python
"""Fused MoE kernel."""

from __future__ import annotations

import functools
from typing import TYPE_CHECKING, Any, Dict, List, Optional

import torch
import torch.nn.functional as F
import triton.language as tl

from sglang.srt.environ import envs
from sglang.srt.layers.moe.moe_runner import MoeRunnerConfig
from sglang.srt.layers.moe.utils import get_moe_padding_size
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import (
    cpu_has_amx_support,
    get_bool_env_var,
    is_cpu,
    is_cuda,
    is_hip,
    is_musa,
    is_xpu,
    use_intel_xpu_backend,
)
from sglang.srt.utils.custom_op import register_custom_op

from .fused_moe_triton_config import get_config_dtype_str, try_get_optimal_moe_config
from .fused_moe_triton_kernels import (
    act_and_mul_triton,
    invoke_fused_moe_kernel,
    moe_sum_reduce_triton,
    support_tensor_descriptor,
)
from .moe_align_block_size import moe_align_block_size

if TYPE_CHECKING:
    from sglang.srt.layers.moe.topk import StandardTopKOutput

_is_hip = is_hip()
_is_cuda = is_cuda()
_is_cpu_amx_available = cpu_has_amx_support()
_is_cpu = is_cpu()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
_is_xpu = is_xpu()
_use_sgl_xpu = use_intel_xpu_backend()
_is_musa = is_musa()


if _is_cuda:
    from sgl_kernel import moe_sum_reduce

    from sglang.jit_kernel.activation import gelu_and_mul, silu_and_mul
elif _is_cpu and _is_cpu_amx_available:
    pass
elif _is_hip:
    from sgl_kernel import gelu_and_mul, silu_and_mul

    if _use_aiter:
        try:
            from aiter import moe_sum
        except ImportError:
            raise ImportError("aiter is required when SGLANG_USE_AITER is set to True")
    # Note: vllm_ops is not needed for HIP when _use_aiter=False
    # because the code uses moe_sum_reduce_triton as fallback (line 619)
elif _is_xpu:
    from sgl_kernel import moe_sum_reduce, silu_and_mul
elif _is_musa:
    from sgl_kernel import moe_sum_reduce

    _silu_and_mul_musa = torch.nn.SwishGLU()

# Try to import vllm_ops for non-CUDA/HIP/XPU platforms
_has_vllm_ops = False
if not _is_cuda and not _is_hip and not _is_xpu:
    try:
        from vllm import _custom_ops as vllm_ops

        _has_vllm_ops = True
    except ImportError:
        # Fallback: vllm not available, will use native PyTorch implementations
        _has_vllm_ops = False

padding_size = get_moe_padding_size(_use_aiter)
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `functools`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.Dict`, and `typing.List`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_hip`, `_is_cuda`, `_is_cpu_amx_available`, `_is_cpu`, and `_use_aiter` capture configuration, cached handles, or feature flags. Control structures like `If` and `Try` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`functools`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.Dict` 以及 `typing.List`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_hip`、`_is_cuda`、`_is_cpu_amx_available`、`_is_cpu` 以及 `_use_aiter` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 和 `Try` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 91-153: Function `inplace_fused_experts` and its core logic
```python
@register_custom_op(mutates_args=["hidden_states"])
def inplace_fused_experts(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    b1: Optional[torch.Tensor] = None,
    b2: Optional[torch.Tensor] = None,
    activation: str = "silu",
    is_gated: bool = True,
    apply_router_weight_on_input: bool = False,
    use_fp8_w8a8: bool = False,
    use_int8_w8a8: bool = False,
    use_int8_w8a16: bool = False,
    use_int4_w4a16: bool = False,
    per_channel_quant: bool = False,
    w1_scale: Optional[torch.Tensor] = None,
    w2_scale: Optional[torch.Tensor] = None,
    w1_zp: Optional[torch.Tensor] = None,
    w2_zp: Optional[torch.Tensor] = None,
    a1_scale: Optional[torch.Tensor] = None,
    a2_scale: Optional[torch.Tensor] = None,
    block_shape: Optional[List[int]] = None,
    routed_scaling_factor: Optional[float] = None,
    gemm1_alpha: Optional[float] = None,
    gemm1_limit: Optional[float] = None,
    filter_expert: bool = True,
    swiglu_limit: Optional[float] = None,
) -> None:
    fused_experts_impl(
        hidden_states,
        w1,
        w2,
        topk_weights,
        topk_ids,
        b1,
        b2,
        True,
        activation,
        is_gated,
        apply_router_weight_on_input,
        use_fp8_w8a8,
        use_int8_w8a8,
        use_int8_w8a16,
        use_int4_w4a16,
        per_channel_quant,
        w1_scale,
        w2_scale,
        w1_zp,
        w2_zp,
        a1_scale,
        a2_scale,
        block_shape,
        False,
        routed_scaling_factor,
        gemm1_alpha,
        gemm1_limit,
        filter_expert,
        swiglu_limit=swiglu_limit,
    )
```
**EN:** This block defines `inplace_fused_experts` and contains the main logic for this step. Decorators like `register_custom_op` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_custom_op` and `fused_experts_impl`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `inplace_fused_experts`，并承载这一阶段的核心逻辑。 像 `register_custom_op` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_custom_op` 和 `fused_experts_impl`，说明该流程会编排底层辅助函数或计算内核。

### Lines 154-217: Function `outplace_fused_experts` and its core logic
```python
@register_custom_op(out_shape="hidden_states")
def outplace_fused_experts(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    b1: Optional[torch.Tensor] = None,
    b2: Optional[torch.Tensor] = None,
    activation: str = "silu",
    is_gated: bool = True,
    apply_router_weight_on_input: bool = False,
    use_fp8_w8a8: bool = False,
    use_int8_w8a8: bool = False,
    use_int8_w8a16: bool = False,
    use_int4_w4a16: bool = False,
    per_channel_quant: bool = False,
    w1_scale: Optional[torch.Tensor] = None,
    w2_scale: Optional[torch.Tensor] = None,
    w1_zp: Optional[torch.Tensor] = None,
    w2_zp: Optional[torch.Tensor] = None,
    a1_scale: Optional[torch.Tensor] = None,
    a2_scale: Optional[torch.Tensor] = None,
    block_shape: Optional[List[int]] = None,
    no_combine: bool = False,
    routed_scaling_factor: Optional[float] = None,
    gemm1_alpha: Optional[float] = None,
    gemm1_limit: Optional[float] = None,
    filter_expert: bool = True,
    swiglu_limit: Optional[float] = None,
) -> torch.Tensor:
    return fused_experts_impl(
        hidden_states,
        w1,
        w2,
        topk_weights,
        topk_ids,
        b1,
        b2,
        False,
        activation,
        is_gated,
        apply_router_weight_on_input,
        use_fp8_w8a8,
        use_int8_w8a8,
        use_int8_w8a16,
        use_int4_w4a16,
        per_channel_quant,
        w1_scale,
        w2_scale,
        w1_zp,
        w2_zp,
        a1_scale,
        a2_scale,
        block_shape,
        no_combine=no_combine,
        routed_scaling_factor=routed_scaling_factor,
        gemm1_alpha=gemm1_alpha,
        gemm1_limit=gemm1_limit,
        filter_expert=filter_expert,
        swiglu_limit=swiglu_limit,
    )
```
**EN:** This block defines `outplace_fused_experts` and contains the main logic for this step. Decorators like `register_custom_op` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_custom_op` and `fused_experts_impl`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `outplace_fused_experts`，并承载这一阶段的核心逻辑。 像 `register_custom_op` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_custom_op` 和 `fused_experts_impl`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 218-308: Function `fused_experts` and its core logic
```python
def fused_experts(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_output: StandardTopKOutput,
    moe_runner_config: MoeRunnerConfig,
    b1: Optional[torch.Tensor] = None,
    b2: Optional[torch.Tensor] = None,
    use_fp8_w8a8: bool = False,
    use_int8_w8a8: bool = False,
    use_int8_w8a16: bool = False,
    use_int4_w4a16: bool = False,
    per_channel_quant: bool = False,
    w1_scale: Optional[torch.Tensor] = None,
    w2_scale: Optional[torch.Tensor] = None,
    w1_zp: Optional[torch.Tensor] = None,
    w2_zp: Optional[torch.Tensor] = None,
    a1_scale: Optional[torch.Tensor] = None,
    a2_scale: Optional[torch.Tensor] = None,
    block_shape: Optional[List[int]] = None,
):
    topk_weights, topk_ids, _ = topk_output
    filter_expert = (
        moe_runner_config.num_experts is None
        or moe_runner_config.num_experts != moe_runner_config.num_local_experts
    )
    if moe_runner_config.inplace:
        assert not moe_runner_config.no_combine, "no combine + inplace makes no sense"
        inplace_fused_experts(
            hidden_states,
            w1,
            w2,
            topk_weights,
            topk_ids,
            b1,
            b2,
            moe_runner_config.activation,
            moe_runner_config.is_gated,
            moe_runner_config.apply_router_weight_on_input,
            use_fp8_w8a8,
            use_int8_w8a8,
            use_int8_w8a16,
            use_int4_w4a16,
            per_channel_quant,
            w1_scale,
            w2_scale,
            w1_zp,
            w2_zp,
            a1_scale,
            a2_scale,
            block_shape,
            moe_runner_config.routed_scaling_factor,
            moe_runner_config.gemm1_alpha,
            moe_runner_config.gemm1_clamp_limit,
            filter_expert,
            swiglu_limit=moe_runner_config.swiglu_limit,
        )
        return hidden_states
    else:
        return outplace_fused_experts(
            hidden_states,
            w1,
            w2,
            topk_weights,
            topk_ids,
            b1,
            b2,
            moe_runner_config.activation,
            moe_runner_config.is_gated,
            moe_runner_config.apply_router_weight_on_input,
            use_fp8_w8a8,
            use_int8_w8a8,
            use_int8_w8a16,
            use_int4_w4a16,
            per_channel_quant,
            w1_scale,
            w2_scale,
            w1_zp,
            w2_zp,
            a1_scale,
            a2_scale,
            block_shape,
            no_combine=moe_runner_config.no_combine,
            routed_scaling_factor=moe_runner_config.routed_scaling_factor,
            gemm1_alpha=moe_runner_config.gemm1_alpha,
            gemm1_limit=moe_runner_config.gemm1_clamp_limit,
            filter_expert=filter_expert,
            swiglu_limit=moe_runner_config.swiglu_limit,
        )
```
**EN:** This block defines `fused_experts` and contains the main logic for this step. It mainly invokes `inplace_fused_experts` and `outplace_fused_experts`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_weights`, `topk_ids`, `_`, and `filter_expert` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_experts`，并承载这一阶段的核心逻辑。 它主要调用 `inplace_fused_experts` 和 `outplace_fused_experts`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_weights`、`topk_ids`、`_` 以及 `filter_expert` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 309-314: Function `moe_sum_reduce_torch_compile` and its core logic
```python
@torch.compile
def moe_sum_reduce_torch_compile(x, out, routed_scaling_factor):
    torch.sum(x, dim=1, out=out)
    out.mul_(routed_scaling_factor)
```
**EN:** This block defines `moe_sum_reduce_torch_compile` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.sum` and `out.mul_`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `moe_sum_reduce_torch_compile`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.sum` 和 `out.mul_`，说明该流程会编排底层辅助函数或计算内核。

### Lines 315-323: Internal helper `_swiglu_silu_clamp_mul`
```python
@torch.compile
def _swiglu_silu_clamp_mul(x, gemm1_limit):
    gate, up = x.chunk(2, dim=-1)
    gate = F.silu(gate)
    gate = gate.clamp(min=None, max=gemm1_limit)
    up = up.clamp(min=-gemm1_limit, max=gemm1_limit)
    return gate * up
```
**EN:** This block defines `_swiglu_silu_clamp_mul` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `x.chunk`, `F.silu`, `gate.clamp`, and `up.clamp`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `gate` and `up` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_swiglu_silu_clamp_mul`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `x.chunk`、`F.silu`、`gate.clamp` 以及 `up.clamp`，说明该流程会编排底层辅助函数或计算内核。 像 `gate` 和 `up` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 324-333: Function `swiglu_gpt_oss_sigmoid_alpha` and its core logic
```python
@torch.compile
def swiglu_gpt_oss_sigmoid_alpha(x, gemm1_alpha, gemm1_limit):
    # NOTE: This variant uses gemm1_alpha, unlike _swiglu_silu_clamp_mul.
    # At present, only GPT-OSS uses this variant.
    gate, up = x[..., ::2], x[..., 1::2]
    gate = gate.clamp(min=None, max=gemm1_limit)
    up = up.clamp(min=-gemm1_limit, max=gemm1_limit)
    return gate * torch.sigmoid(gate * gemm1_alpha) * (up + 1)
```
**EN:** This block defines `swiglu_gpt_oss_sigmoid_alpha` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `gate.clamp`, `up.clamp`, and `torch.sigmoid`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `gate` and `up` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `swiglu_gpt_oss_sigmoid_alpha`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `gate.clamp`、`up.clamp` 以及 `torch.sigmoid`，说明该流程会编排底层辅助函数或计算内核。 像 `gate` 和 `up` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 334-338: Internal helper `_down_moe_use_tma`
```python
@functools.lru_cache()
def _down_moe_use_tma():
    return support_tensor_descriptor()
```
**EN:** This block defines `_down_moe_use_tma` and contains the main logic for this step. Decorators like `functools.lru_cache` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `functools.lru_cache` and `support_tensor_descriptor`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_down_moe_use_tma`，并承载这一阶段的核心逻辑。 像 `functools.lru_cache` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `functools.lru_cache` 和 `support_tensor_descriptor`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 339-400: Internal helper `_prepare_fused_moe_run`
```python
def _prepare_fused_moe_run(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_ids: torch.Tensor,
    *,
    use_fp8_w8a8: bool,
    use_int8_w8a8: bool,
    use_int8_w8a16: bool,
    use_int4_w4a16: bool,
    per_channel_quant: bool,
    block_shape: Optional[List[int]],
):
    """Resolve config, down_config, TMA flag, and aligned expert routing ids.

    Shared by ``fused_experts_impl`` and ``pre_permute_standard_to_triton`` so
    both paths compute alignment from the same source.
    """
    padded_size = padding_size
    if not (use_fp8_w8a8 or use_int8_w8a8) or block_shape is not None or _use_aiter:
        padded_size = 0

    num_tokens = hidden_states.shape[0]
    E = w1.shape[0]
    config_dtype = get_config_dtype_str(
        use_fp8_w8a8=use_fp8_w8a8,
        use_int8_w8a8=use_int8_w8a8,
        use_int8_w8a16=use_int8_w8a16,
        use_int4_w4a16=use_int4_w4a16,
        dtype=hidden_states.dtype,
    )

    config, (down_config, _) = try_get_optimal_moe_config(
        w1.shape,
        (w2.shape[0], w2.shape[1], w2.shape[2] - padded_size),
        topk_ids.shape[1],
        config_dtype,
        num_tokens,
        block_shape=block_shape,
        per_channel_quant=per_channel_quant,
        return_down_config=True,
    )
    down_moe_use_tma = (
        _down_moe_use_tma()
        and down_config is not None
        and down_config.pop("USE_TMA", False)
    )

    sorted_token_ids, expert_ids, num_tokens_post_padded = moe_align_block_size(
        topk_ids, config["BLOCK_SIZE_M"], E
    )

    return (
        config,
        down_config,
        down_moe_use_tma,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
    )
```
**EN:** This block defines `_prepare_fused_moe_run` and contains the main logic for this step. It mainly invokes `get_config_dtype_str`, `try_get_optimal_moe_config`, `moe_align_block_size`, `_down_moe_use_tma`, and `down_config.pop`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `padded_size`, `num_tokens`, `E`, `config_dtype`, and `config` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_prepare_fused_moe_run`，并承载这一阶段的核心逻辑。 它主要调用 `get_config_dtype_str`、`try_get_optimal_moe_config`、`moe_align_block_size`、`_down_moe_use_tma` 以及 `down_config.pop`，说明该流程会编排底层辅助函数或计算内核。 像 `padded_size`、`num_tokens`、`E`、`config_dtype` 以及 `config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 401-787: Internal helper `_fused_moe_kernel_sequence`
```python
def _fused_moe_kernel_sequence(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    sorted_token_ids: torch.Tensor,
    expert_ids: torch.Tensor,
    num_tokens_post_padded: torch.Tensor,
    config: Dict[str, Any],
    down_config: Optional[Dict[str, Any]],
    down_moe_use_tma: bool,
    *,
    b1: Optional[torch.Tensor],
    b2: Optional[torch.Tensor],
    use_fp8_w8a8: bool,
    use_int8_w8a8: bool,
    use_int8_w8a16: bool,
    use_int4_w4a16: bool,
    per_channel_quant: bool,
    w1_scale: Optional[torch.Tensor],
    w2_scale: Optional[torch.Tensor],
    w1_zp: Optional[torch.Tensor],
    w2_zp: Optional[torch.Tensor],
    a1_scale: Optional[torch.Tensor],
    a2_scale: Optional[torch.Tensor],
    block_shape: Optional[List[int]],
    activation: str,
    is_gated: bool,
    no_combine: bool,
    inplace: bool,
    apply_router_weight_on_input: bool,
    routed_scaling_factor: Optional[float],
    gemm1_alpha: Optional[float],
    gemm1_limit: Optional[float],
    filter_expert: bool,
    hooks: Optional[Any] = None,
    swiglu_limit: Optional[float] = None,
) -> torch.Tensor:
    """Run the MoE kernel/activation/kernel/combine sequence in a single shot.

    Inputs are already aligned and the block-size config is already resolved.
    Supports optional LoRA hooks that fire between the two kernels and before
    combine. Returns ``out_hidden_states``.
    """
    num_tokens = hidden_states.shape[0]
    E, N, _ = w1.shape
    topk = topk_ids.shape[1]
    compute_type = tl.bfloat16 if hidden_states.dtype == torch.bfloat16 else tl.float16

    padded_tokens = (
        min(num_tokens * topk, E + 1) * (config["BLOCK_SIZE_M"] - 1)
        if down_moe_use_tma
        else 0
    )
    total_tokens = num_tokens * topk + padded_tokens

    if no_combine:
        assert not inplace
        out_hidden_states = torch.empty(
            (num_tokens, topk, w2.shape[1]),
            device=hidden_states.device,
            dtype=hidden_states.dtype,
        )
    elif inplace:
        out_hidden_states = hidden_states
    else:
        out_hidden_states = torch.empty_like(hidden_states)

    use_fused_moe_sum_all_reduce = (
        get_global_server_args().enable_fused_moe_sum_all_reduce
        and (not no_combine)
        and (topk > 2)
        and (not use_int8_w8a16)
        and (not use_int4_w4a16)
    )

    intermediate_cache1 = torch.empty(
        (total_tokens, N),
        device=hidden_states.device,
        dtype=hidden_states.dtype,
    )

    invoke_fused_moe_kernel(
        hidden_states,
        w1,
        b1,
        intermediate_cache1,
        a1_scale,
        w1_scale,
        w1_zp,
        topk_weights,
        topk_ids,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        apply_router_weight_on_input,
        topk,
        config,
        compute_type=compute_type,
        use_fp8_w8a8=use_fp8_w8a8,
        use_int8_w8a8=use_int8_w8a8,
        use_int8_w8a16=use_int8_w8a16,
        use_int4_w4a16=use_int4_w4a16,
        per_channel_quant=per_channel_quant,
        block_shape=block_shape,
        c_sorted=down_moe_use_tma,
        filter_expert=filter_expert,
    )

    if hooks and hooks.after_gate_up:
        # Hooks expect intermediate_cache1 shaped (num_tokens, topk, N); the
        # underlying buffer is laid out as (total_tokens, N) where
        # total_tokens = num_tokens * topk (+ TMA padding). Slice off any
        # padding and reshape for the hook, which writes in-place on the view.
        hooks.after_gate_up(
            hidden_states,
            intermediate_cache1[: num_tokens * topk].view(num_tokens, topk, N),
            topk_weights,
            topk_ids,
        )

    intermediate_cache2 = torch.empty(
        (total_tokens, N // 2),
        device=hidden_states.device,
        dtype=hidden_states.dtype,
    )

    # Activation function with multiplication
    if activation == "silu" and is_gated:
        # - gemm1_alpha != None: GPT-OSS-style swiglu(alpha, limit)
        # - gemm1_alpha == None and gemm1_limit != None: silu+clamp+mul(limit-only)
        # - swiglu_limit != None: DeepSeek V4 swiglu clamp + silu_and_mul (CUDA/HIP only)
        if gemm1_alpha is not None:
            assert gemm1_limit is not None
            intermediate_cache2 = swiglu_gpt_oss_sigmoid_alpha(
                intermediate_cache1.view(-1, N), gemm1_alpha, gemm1_limit
            )
        elif gemm1_limit is not None:
            intermediate_cache2 = _swiglu_silu_clamp_mul(
                intermediate_cache1.view(-1, N), gemm1_limit
            )
        elif swiglu_limit is not None:
            # DeepSeek V4: swiglu clamp before silu_and_mul.
            # Two paths gated by SGLANG_OPT_SWIGLU_CLAMP_FUSION:
            #   fusion=True: clamp fused into act_and_mul_triton or silu_and_mul_clamp
            #   fusion=False: explicit clamp_ on intermediate_cache1 (path checker)
            assert swiglu_limit == 10
            assert intermediate_cache1.shape == (total_tokens, N)
            assert _is_cuda or _is_hip, "DeepSeek V4 only supports CUDA/HIP downstream"

            swiglu_limit_for_triton: Optional[float] = None
            swiglu_limit_for_silu_and_mul_clamp: Optional[float] = None

            if envs.SGLANG_OPT_SWIGLU_CLAMP_FUSION.get():
                if filter_expert:
                    swiglu_limit_for_triton = swiglu_limit
                else:
                    assert (
                        _is_cuda
                    ), "fused silu_and_mul_clamp kernel is CUDA-only; HIP must disable SWIGLU_CLAMP_FUSION"
                    swiglu_limit_for_silu_and_mul_clamp = swiglu_limit
            else:
                half = N // 2
                intermediate_cache1[:, :half].clamp_(max=swiglu_limit)
                intermediate_cache1[:, half:].clamp_(
                    min=-swiglu_limit, max=swiglu_limit
                )

            if not filter_expert:
                if swiglu_limit_for_silu_and_mul_clamp is not None:
                    from sglang.jit_kernel.deepseek_v4 import silu_and_mul_clamp

                    silu_and_mul_clamp(
                        intermediate_cache1.view(-1, N),
                        intermediate_cache2,
                        swiglu_limit_for_silu_and_mul_clamp,
                    )
                else:
                    silu_and_mul(intermediate_cache1.view(-1, N), intermediate_cache2)
            else:
                act_and_mul_triton(
                    intermediate_cache1.view(-1, N),
                    intermediate_cache2,
                    config,
                    topk_ids,
                    expert_ids,
                    down_moe_use_tma,
                    activation,
                    swiglu_limit=swiglu_limit_for_triton,
                )
        elif _is_cuda or _is_hip or _is_xpu:
            if filter_expert and _is_cuda:
                # HIP/XPU fall through to the unfiltered path: the down kernel
                # zeros filtered rows without reading their input.
                silu_and_mul(
                    intermediate_cache1.view(-1, N),
                    intermediate_cache2,
                    expert_ids=(expert_ids if down_moe_use_tma else topk_ids.view(-1)),
                    expert_step=(config["BLOCK_SIZE_M"] if down_moe_use_tma else 1),
                )
            else:
                silu_and_mul(intermediate_cache1.view(-1, N), intermediate_cache2)
        elif _is_musa:
            intermediate_cache2 = _silu_and_mul_musa(intermediate_cache1.view(-1, N))
        else:
            if _has_vllm_ops:
                vllm_ops.silu_and_mul(
                    intermediate_cache2, intermediate_cache1.view(-1, N)
                )
            else:
                # Fallback: native PyTorch silu_and_mul
                x = intermediate_cache1.view(-1, N)
                d = x.shape[-1] // 2
                intermediate_cache2.copy_(F.silu(x[..., :d]) * x[..., d:])
    elif activation == "gelu" and is_gated:
        assert gemm1_alpha is None, "gemm1_alpha is not supported for gelu"
        assert gemm1_limit is None, "gemm1_limit is not supported for gelu"
        if _is_cuda or _is_hip:
            if filter_expert and _is_cuda:
                gelu_and_mul(
                    intermediate_cache1.view(-1, N),
                    intermediate_cache2,
                    expert_ids=(expert_ids if down_moe_use_tma else topk_ids.view(-1)),
                    expert_step=(config["BLOCK_SIZE_M"] if down_moe_use_tma else 1),
                )
            else:
                gelu_and_mul(intermediate_cache1.view(-1, N), intermediate_cache2)
        else:
            if _has_vllm_ops:
                vllm_ops.gelu_and_mul(
                    intermediate_cache2, intermediate_cache1.view(-1, N)
                )
            else:
                # Fallback: native PyTorch gelu_and_mul
                x = intermediate_cache1.view(-1, N)
                d = x.shape[-1] // 2
                intermediate_cache2.copy_(F.gelu(x[..., :d]) * x[..., d:])
    # Activation function without multiplication
    elif activation == "silu" and not is_gated:
        intermediate_cache2 = F.silu(intermediate_cache1.view(-1, N))
    elif activation == "gelu" and not is_gated:
        intermediate_cache2 = F.gelu(intermediate_cache1.view(-1, N))
    elif activation == "relu2" and not is_gated:
        intermediate_cache2 = torch.square(F.relu(intermediate_cache1.view(-1, N)))
    else:
        raise ValueError(f"Unsupported activation: {activation=}, with {is_gated=}")

    del intermediate_cache1

    intermediate_cache3 = torch.empty(
        (num_tokens, topk, w2.shape[1]),
        device=hidden_states.device,
        dtype=hidden_states.dtype,
    )

    # LoRA hooks force the second kernel to write to intermediate_cache3 so
    # hooks.after_down can inspect/modify it before reduction.
    _use_intermediate = not no_combine and (topk != 1 or hooks)

    out_slice = None
    if use_fused_moe_sum_all_reduce:
        out_slice = out_hidden_states
        out_slice.zero_()

    invoke_fused_moe_kernel(
        intermediate_cache2,
        w2,
        b2,
        (
            out_slice
            if use_fused_moe_sum_all_reduce
            else (
                intermediate_cache3
                if _use_intermediate
                else out_hidden_states.unsqueeze(0)
            )
        ),
        a2_scale,
        w2_scale,
        w2_zp,
        topk_weights,
        topk_ids,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        not apply_router_weight_on_input and not no_combine,
        1,
        down_config or config,
        compute_type=compute_type,
        use_fp8_w8a8=use_fp8_w8a8,
        use_int8_w8a8=use_int8_w8a8,
        use_int8_w8a16=use_int8_w8a16,
        use_int4_w4a16=use_int4_w4a16,
        per_channel_quant=per_channel_quant,
        block_shape=block_shape,
        a_use_tma=down_moe_use_tma,
        b_use_tma=down_moe_use_tma,
        filter_expert=filter_expert,
        fuse_sum_all_reduce=use_fused_moe_sum_all_reduce,
        router_topk=topk,
    )

    if hooks and hooks.after_down:
        hooks.after_down(
            intermediate_cache2, intermediate_cache3, topk_weights, topk_ids
        )

    del intermediate_cache2

    if routed_scaling_factor is None:
        routed_scaling_factor = 1.0

    if no_combine:
        pass
    elif _is_cuda or _is_musa:
        if use_fused_moe_sum_all_reduce:
            if routed_scaling_factor != 1.0:
                assert out_slice is not None
                out_slice.mul_(routed_scaling_factor)
        elif topk == 1 and routed_scaling_factor == 1.0 and not _use_intermediate:
            pass  # we wrote directly into out_hidden_states
        elif topk == 2 and routed_scaling_factor == 1.0:
            torch.add(
                intermediate_cache3[:, 0],
                intermediate_cache3[:, 1],
                out=out_hidden_states,
            ).squeeze(dim=1)
        else:
            # According to micro benchmark results, torch.compile can get better performance for small token.
            if num_tokens <= 32:
                moe_sum_reduce_torch_compile(
                    intermediate_cache3.view(*intermediate_cache3.shape),
                    out_hidden_states,
                    routed_scaling_factor,
                )
            else:
                moe_sum_reduce(
                    intermediate_cache3.view(*intermediate_cache3.shape),
                    out_hidden_states,
                    routed_scaling_factor,
                )
    elif _is_hip:
        if _use_aiter:
            moe_sum(
                intermediate_cache3.view(*intermediate_cache3.shape),
                out_hidden_states,
            )
        else:
            # According to micro benchmark results, torch.compile can get better performance for small token.
            if num_tokens <= 32:
                moe_sum_reduce_torch_compile(
                    intermediate_cache3.view(*intermediate_cache3.shape),
                    out_hidden_states,
                    routed_scaling_factor,
                )
            else:
                moe_sum_reduce_triton(
                    intermediate_cache3.view(*intermediate_cache3.shape),
                    out_hidden_states,
                    routed_scaling_factor,
                )
    elif _is_xpu:
        moe_sum_reduce(
            intermediate_cache3.view(*intermediate_cache3.shape),
            out_hidden_states,
            routed_scaling_factor,
        )
    else:
        if _has_vllm_ops:
            vllm_ops.moe_sum(
                intermediate_cache3.view(*intermediate_cache3.shape),
                out_hidden_states,
            )
        else:
            # Fallback: use triton moe_sum_reduce when vllm is not available
            moe_sum_reduce_triton(
                intermediate_cache3.view(*intermediate_cache3.shape),
                out_hidden_states,
                routed_scaling_factor,
            )

    del intermediate_cache3

    return out_hidden_states
```
**EN:** This block defines `_fused_moe_kernel_sequence` and contains the main logic for this step. It mainly invokes `torch.empty`, `invoke_fused_moe_kernel`, `hooks.after_gate_up`, `out_slice.zero_`, and `hooks.after_down`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens`, `E`, `N`, `_`, and `topk` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_fused_moe_kernel_sequence`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`invoke_fused_moe_kernel`、`hooks.after_gate_up`、`out_slice.zero_` 以及 `hooks.after_down`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens`、`E`、`N`、`_` 以及 `topk` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 788-895: Function `fused_experts_impl` and its core logic
```python
def fused_experts_impl(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    b1: Optional[torch.Tensor] = None,
    b2: Optional[torch.Tensor] = None,
    inplace: bool = False,
    activation: str = "silu",
    is_gated: bool = True,
    apply_router_weight_on_input: bool = False,
    use_fp8_w8a8: bool = False,
    use_int8_w8a8: bool = False,
    use_int8_w8a16: bool = False,
    use_int4_w4a16: bool = False,
    per_channel_quant: bool = False,
    w1_scale: Optional[torch.Tensor] = None,
    w2_scale: Optional[torch.Tensor] = None,
    w1_zp: Optional[torch.Tensor] = None,
    w2_zp: Optional[torch.Tensor] = None,
    a1_scale: Optional[torch.Tensor] = None,
    a2_scale: Optional[torch.Tensor] = None,
    block_shape: Optional[List[int]] = None,
    no_combine: bool = False,
    routed_scaling_factor: Optional[float] = None,
    gemm1_alpha: Optional[float] = None,
    gemm1_limit: Optional[float] = None,
    filter_expert: bool = True,
    swiglu_limit: Optional[float] = None,
):
    padded_size = padding_size
    if not (use_fp8_w8a8 or use_int8_w8a8) or block_shape is not None or _use_aiter:
        padded_size = 0

    # Check constraints.
    if use_int4_w4a16:
        assert hidden_states.shape[1] // 2 == w1.shape[2], "Hidden size mismatch"
    else:
        assert (
            hidden_states.shape[1] == w1.shape[2] - padded_size
        ), f"Hidden size mismatch"
    assert topk_weights.shape == topk_ids.shape, "topk shape mismatch"
    assert hidden_states.is_contiguous(), "Hidden_states must be contiguous"
    assert w1.is_contiguous(), "Expert weights1 must be contiguous"
    assert w2.is_contiguous(), "Expert weights2 must be contiguous"
    assert hidden_states.dtype in [torch.float32, torch.float16, torch.bfloat16]

    (
        config,
        down_config,
        down_moe_use_tma,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
    ) = _prepare_fused_moe_run(
        hidden_states,
        w1,
        w2,
        topk_ids,
        use_fp8_w8a8=use_fp8_w8a8,
        use_int8_w8a8=use_int8_w8a8,
        use_int8_w8a16=use_int8_w8a16,
        use_int4_w4a16=use_int4_w4a16,
        per_channel_quant=per_channel_quant,
        block_shape=block_shape,
    )

    return _fused_moe_kernel_sequence(
        hidden_states,
        w1,
        w2,
        topk_weights,
        topk_ids,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        config,
        down_config,
        down_moe_use_tma,
        b1=b1,
        b2=b2,
        use_fp8_w8a8=use_fp8_w8a8,
        use_int8_w8a8=use_int8_w8a8,
        use_int8_w8a16=use_int8_w8a16,
        use_int4_w4a16=use_int4_w4a16,
        per_channel_quant=per_channel_quant,
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        w1_zp=w1_zp,
        w2_zp=w2_zp,
        a1_scale=a1_scale,
        a2_scale=a2_scale,
        block_shape=block_shape,
        activation=activation,
        is_gated=is_gated,
        no_combine=no_combine,
        inplace=inplace,
        apply_router_weight_on_input=apply_router_weight_on_input,
        routed_scaling_factor=routed_scaling_factor,
        gemm1_alpha=gemm1_alpha,
        gemm1_limit=gemm1_limit,
        filter_expert=filter_expert,
        hooks=None,
        swiglu_limit=swiglu_limit,
    )
```
**EN:** This block defines `fused_experts_impl` and contains the main logic for this step. It mainly invokes `hidden_states.is_contiguous`, `w1.is_contiguous`, `w2.is_contiguous`, `_prepare_fused_moe_run`, and `_fused_moe_kernel_sequence`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `padded_size`, `config`, `down_config`, `down_moe_use_tma`, and `sorted_token_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_experts_impl`，并承载这一阶段的核心逻辑。 它主要调用 `hidden_states.is_contiguous`、`w1.is_contiguous`、`w2.is_contiguous`、`_prepare_fused_moe_run` 以及 `_fused_moe_kernel_sequence`，说明该流程会编排底层辅助函数或计算内核。 像 `padded_size`、`config`、`down_config`、`down_moe_use_tma` 以及 `sorted_token_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 896-998: Function `fused_moe` and its core logic
```python
def fused_moe(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_output: StandardTopKOutput,
    moe_runner_config: MoeRunnerConfig = MoeRunnerConfig(),
    b1: Optional[torch.Tensor] = None,
    b2: Optional[torch.Tensor] = None,
    use_fp8_w8a8: bool = False,
    use_int8_w8a8: bool = False,
    use_int8_w8a16: bool = False,
    use_int4_w4a16: bool = False,
    per_channel_quant: bool = False,
    w1_scale: Optional[torch.Tensor] = None,
    w2_scale: Optional[torch.Tensor] = None,
    w1_zp: Optional[torch.Tensor] = None,
    w2_zp: Optional[torch.Tensor] = None,
    a1_scale: Optional[torch.Tensor] = None,
    a2_scale: Optional[torch.Tensor] = None,
    block_shape: Optional[List[int]] = None,
) -> torch.Tensor:
    """
    This function computes a Mixture of Experts (MoE) layer using two sets of
    weights, w1 and w2, and top-k gating mechanism.

    Parameters:
    - hidden_states (torch.Tensor): The input tensor to the MoE layer.
    - w1 (torch.Tensor): The first set of expert weights.
    - w2 (torch.Tensor): The second set of expert weights.
    - topk_output (StandardTopKOutput): The top-k output of the experts.
    - moe_runner_config (MoeRunnerConfig): The configuration for the MoE runner.
    - b1 (Optional[torch.Tensor]): Optional bias for w1.
    - b2 (Optional[torch.Tensor]): Optional bias for w2.
    - use_fp8_w8a8 (bool): If True, use fp8 arithmetic to compute the inner
        products for w1 and w2. Defaults to False.
    - use_int8_w8a8 (bool): If True, use int8 arithmetic to compute the inner
        products for w1 and w2. Defaults to False.
    - use_int8_w8a16 (bool): If True, use fp8 arithmetic to compute the inner
        products for w1 and w2. Defaults to False.
    - use_int4_w4a16 (bool): If True, use matmul of int4 weight and bf16/fp16
        activation to compute the inner products for w1 and w2.
        Defaults to False.
    - w1_scale (Optional[torch.Tensor]): Optional scale to be used for
        w1.
    - w2_scale (Optional[torch.Tensor]): Optional scale to be used for
        w2.
    - a1_scale (Optional[torch.Tensor]): Optional scale to be used for
        a1.
    - a2_scale (Optional[torch.Tensor]): Optional scale to be used for
        a2.
    - block_shape: (Optional[List[int]]): Optional block size for block-wise
        quantization.
    - gemm1_alpha (Optional[float]): Optional gemm1_alpha for the activation
        function.
    - gemm1_limit (Optional[float]): Optional gemm1_limit for the swiglu activation
        function.

    Returns:
    - torch.Tensor: The output tensor after applying the MoE layer.
    """
    if _use_sgl_xpu:
        topk_weight, topk_ids, _ = topk_output
        from sgl_kernel import fused_experts as sgl_fused_experts

        return sgl_fused_experts(
            hidden_states,
            w1,
            w2,
            topk_weight,
            topk_ids,
            b1=b1,
            b2=b2,
            use_fp8_w8a8=use_fp8_w8a8,
            w1_scale=w1_scale,
            w2_scale=w2_scale,
            w1_zp=w1_zp,
            w2_zp=w2_zp,
            a1_scale=a1_scale,
            a2_scale=a2_scale,
            block_shape=block_shape,
        )

    return fused_experts(
        hidden_states,
        w1,
        w2,
        topk_output,
        moe_runner_config=moe_runner_config,
        b1=b1,
        b2=b2,
        use_fp8_w8a8=use_fp8_w8a8,
        use_int8_w8a8=use_int8_w8a8,
        use_int8_w8a16=use_int8_w8a16,
        use_int4_w4a16=use_int4_w4a16,
        per_channel_quant=per_channel_quant,
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        w1_zp=w1_zp,
        w2_zp=w2_zp,
        a1_scale=a1_scale,
        a2_scale=a2_scale,
        block_shape=block_shape,
    )
```
**EN:** This block defines `fused_moe` and contains the main logic for this step. It mainly invokes `MoeRunnerConfig`, `fused_experts`, and `sgl_fused_experts`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_weight`, `topk_ids`, and `_` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_moe`，并承载这一阶段的核心逻辑。 它主要调用 `MoeRunnerConfig`、`fused_experts` 以及 `sgl_fused_experts`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_weight`、`topk_ids` 以及 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `inplace_fused_experts`, `outplace_fused_experts`, `fused_experts`, `moe_sum_reduce_torch_compile`, and `_swiglu_silu_clamp_mul`. / **主要符号**：核心入口包括 `inplace_fused_experts`、`outplace_fused_experts`、`fused_experts`、`moe_sum_reduce_torch_compile` 以及 `_swiglu_silu_clamp_mul`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `functools`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.Dict`, `typing.List`, and `typing.Optional` / **标准库**：`__future__.annotations`、`functools`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.Dict`、`typing.List` 以及 `typing.Optional`
- **Third-party**: `torch`, `torch.nn.functional`, `triton.language`, `sgl_kernel.moe_sum_reduce`, `vllm._custom_ops`, `sgl_kernel.fused_experts`, `sgl_kernel.gelu_and_mul`, `sgl_kernel.silu_and_mul`, and `aiter.moe_sum` / **第三方依赖**：`torch`、`torch.nn.functional`、`triton.language`、`sgl_kernel.moe_sum_reduce`、`vllm._custom_ops`、`sgl_kernel.fused_experts`、`sgl_kernel.gelu_and_mul`、`sgl_kernel.silu_and_mul` 以及 `aiter.moe_sum`
- **Internal SGLang modules**: `sglang.srt.environ.envs`, `sglang.srt.layers.moe.moe_runner.MoeRunnerConfig`, `sglang.srt.layers.moe.utils.get_moe_padding_size`, `sglang.srt.server_args.get_global_server_args`, `sglang.srt.utils.cpu_has_amx_support`, `sglang.srt.utils.get_bool_env_var`, `sglang.srt.utils.is_cpu`, `sglang.srt.utils.is_cuda`, `sglang.srt.utils.is_hip`, `sglang.srt.utils.is_musa`, `sglang.srt.utils.is_xpu`, and `sglang.srt.utils.use_intel_xpu_backend` / **SGLang 内部模块**：`sglang.srt.environ.envs`、`sglang.srt.layers.moe.moe_runner.MoeRunnerConfig`、`sglang.srt.layers.moe.utils.get_moe_padding_size`、`sglang.srt.server_args.get_global_server_args`、`sglang.srt.utils.cpu_has_amx_support`、`sglang.srt.utils.get_bool_env_var`、`sglang.srt.utils.is_cpu`、`sglang.srt.utils.is_cuda`、`sglang.srt.utils.is_hip`、`sglang.srt.utils.is_musa`、`sglang.srt.utils.is_xpu` 以及 `sglang.srt.utils.use_intel_xpu_backend`
