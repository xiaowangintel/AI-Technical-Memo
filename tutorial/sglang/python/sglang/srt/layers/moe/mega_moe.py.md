# mega_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/mega_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `_apply_mega_moe_dg_env`, `_get_mega_moe_symm_buffer`, `should_use_mega_moe`, and `forward_mega_moe` and connects them to backend-specific paths such as `CUDA` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `_apply_mega_moe_dg_env`、`_get_mega_moe_symm_buffer`、`should_use_mega_moe` 以及 `forward_mega_moe` 等符号，并把这些符号连接到 `CUDA` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: File header and module overview
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 14-41: Imports, conditional backend setup, and runtime guards
```python
"""Mega-MoE forward path and expert-weight prep shared by Deepseek V2/V4."""

from __future__ import annotations

import os
from contextlib import nullcontext
from typing import TYPE_CHECKING, Optional

import torch

from sglang.jit_kernel.deepseek_v4 import mega_moe_pre_dispatch
from sglang.srt.environ import envs
from sglang.srt.eplb.expert_location_dispatch import ExpertLocationDispatchInfo
from sglang.srt.layers.dp_attention import get_dp_global_num_tokens
from sglang.srt.layers.moe.utils import get_moe_a2a_backend
from sglang.srt.model_executor.cuda_graph_runner import get_is_capture_mode

if TYPE_CHECKING:
    from deep_gemm import SymmBuffer

    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
    from sglang.srt.models.deepseek_v2 import DeepseekV2MoE


_MEGA_MOE_SYMM_BUFFER: dict = {}
_MEGA_MOE_DG_ENV_APPLIED = False
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `os`, `contextlib.nullcontext`, `typing.TYPE_CHECKING`, `typing.Optional`, and `torch`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_MEGA_MOE_SYMM_BUFFER` and `_MEGA_MOE_DG_ENV_APPLIED` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`os`、`contextlib.nullcontext`、`typing.TYPE_CHECKING`、`typing.Optional` 以及 `torch`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_MEGA_MOE_SYMM_BUFFER` 和 `_MEGA_MOE_DG_ENV_APPLIED` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 42-60: Internal helper `_apply_mega_moe_dg_env`
```python
def _apply_mega_moe_dg_env() -> None:
    """Forward sglang's FP4/MXF4 opt-in flags to DeepGEMM via env vars.

    DeepGEMM reads `DG_USE_FP4_ACTS` (and `DG_USE_MXF4_KIND`) at host-function
    call time — both `get_symm_buffer_for_mega_moe` and `fp8_fp4_mega_moe`.
    Forwarding once at first use is sufficient (these are static config
    flags, not per-request state) and matches the `setdefault` pattern so
    explicit `DG_USE_*` overrides from outside still win.
    """
    global _MEGA_MOE_DG_ENV_APPLIED
    if _MEGA_MOE_DG_ENV_APPLIED:
        return
    if envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_FP4_ACTS.get():
        os.environ.setdefault("DG_USE_FP4_ACTS", "1")
    if envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_MXF4_KIND.get():
        os.environ.setdefault("DG_USE_MXF4_KIND", "1")
    _MEGA_MOE_DG_ENV_APPLIED = True
```
**EN:** This block defines `_apply_mega_moe_dg_env` and contains the main logic for this step. It mainly invokes `envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_FP4_ACTS.get`, `envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_MXF4_KIND.get`, and `os.environ.setdefault`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `_MEGA_MOE_DG_ENV_APPLIED` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_apply_mega_moe_dg_env`，并承载这一阶段的核心逻辑。 它主要调用 `envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_FP4_ACTS.get`、`envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_MXF4_KIND.get` 以及 `os.environ.setdefault`，说明该流程会编排底层辅助函数或计算内核。 像 `_MEGA_MOE_DG_ENV_APPLIED` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 61-96: Internal helper `_get_mega_moe_symm_buffer`
```python
def _get_mega_moe_symm_buffer(
    group,
    num_experts: int,
    num_max_tokens_per_rank: int,
    num_topk: int,
    hidden: int,
    intermediate_hidden: int,
) -> SymmBuffer:
    import deep_gemm

    _apply_mega_moe_dg_env()

    key = (
        id(group),
        num_max_tokens_per_rank,
        num_experts,
        num_topk,
        hidden,
        intermediate_hidden,
    )
    buf = _MEGA_MOE_SYMM_BUFFER.get(key)
    if buf is None:
        buf = deep_gemm.get_symm_buffer_for_mega_moe(
            group,
            num_experts,
            num_max_tokens_per_rank,
            num_topk,
            hidden,
            intermediate_hidden,
            use_fp8_dispatch=True,
            activation="swiglu",
        )
        _MEGA_MOE_SYMM_BUFFER[key] = buf
    return buf
```
**EN:** This block defines `_get_mega_moe_symm_buffer` and contains the main logic for this step. It mainly invokes `_apply_mega_moe_dg_env`, `_MEGA_MOE_SYMM_BUFFER.get`, `id`, and `deep_gemm.get_symm_buffer_for_mega_moe`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `key` and `buf` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_get_mega_moe_symm_buffer`，并承载这一阶段的核心逻辑。 它主要调用 `_apply_mega_moe_dg_env`、`_MEGA_MOE_SYMM_BUFFER.get`、`id` 以及 `deep_gemm.get_symm_buffer_for_mega_moe`，说明该流程会编排底层辅助函数或计算内核。 像 `key` 和 `buf` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 97-113: `should_use_mega_moe` predicate for should use mega Mixture-of-Experts
```python
def should_use_mega_moe(moe: "DeepseekV2MoE", hidden_states: torch.Tensor) -> bool:
    if not get_moe_a2a_backend().is_megamoe():
        return False
    if not getattr(moe.experts, "_mega_moe_weights_built", False):
        return False
    if get_is_capture_mode():
        return True

    global_num_tokens = get_dp_global_num_tokens()
    if global_num_tokens:
        max_tokens_per_rank = max(global_num_tokens)
    else:
        max_tokens_per_rank = hidden_states.shape[0]
    cap = envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_NUM_MAX_TOKENS_PER_RANK.get()
    return max_tokens_per_rank <= cap
```
**EN:** This block defines `should_use_mega_moe` and contains the main logic for this step. It mainly invokes `get_is_capture_mode`, `get_dp_global_num_tokens`, `envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_NUM_MAX_TOKENS_PER_RANK.get`, `get_moe_a2a_backend.is_megamoe`, and `getattr`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `global_num_tokens`, `cap`, and `max_tokens_per_rank` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `should_use_mega_moe`，并承载这一阶段的核心逻辑。 它主要调用 `get_is_capture_mode`、`get_dp_global_num_tokens`、`envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_NUM_MAX_TOKENS_PER_RANK.get`、`get_moe_a2a_backend.is_megamoe` 以及 `getattr`，说明该流程会编排底层辅助函数或计算内核。 像 `global_num_tokens`、`cap` 以及 `max_tokens_per_rank` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 114-150: `forward_mega_moe` MEGA_MOE execution path
```python
def forward_mega_moe(
    moe: "DeepseekV2MoE",
    hidden_states: torch.Tensor,
    forward_batch: Optional["ForwardBatch"] = None,
    input_ids_global: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    num_tokens = hidden_states.shape[0]

    sbo_overlap_flag = (
        moe.alt_stream is not None
        and moe.num_fused_shared_experts == 0
        and num_tokens > 0
        and get_is_capture_mode()
    )

    if sbo_overlap_flag:
        current_stream = torch.cuda.current_stream()
        moe.alt_stream.wait_stream(current_stream)
        shared_output = moe._forward_shared_experts(hidden_states)
        mega_stream_ctx = torch.cuda.stream(moe.alt_stream)
    else:
        shared_output = moe._forward_shared_experts(hidden_states)
        mega_stream_ctx = nullcontext()

    with mega_stream_ctx:
        y = _run_mega_routed(
            moe, hidden_states, forward_batch, input_ids_global, num_tokens
        )

    if sbo_overlap_flag:
        current_stream.wait_stream(moe.alt_stream)

    if shared_output is not None:
        y.add_(shared_output)
    return y
```
**EN:** This block defines `forward_mega_moe` and contains the main logic for this step. It mainly invokes `get_is_capture_mode`, `torch.cuda.current_stream`, `moe.alt_stream.wait_stream`, `moe._forward_shared_experts`, and `torch.cuda.stream`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens`, `sbo_overlap_flag`, `current_stream`, `shared_output`, and `mega_stream_ctx` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `forward_mega_moe`，并承载这一阶段的核心逻辑。 它主要调用 `get_is_capture_mode`、`torch.cuda.current_stream`、`moe.alt_stream.wait_stream`、`moe._forward_shared_experts` 以及 `torch.cuda.stream`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens`、`sbo_overlap_flag`、`current_stream`、`shared_output` 以及 `mega_stream_ctx` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 151-269: Internal helper `_run_mega_routed`
```python
def _run_mega_routed(
    moe: "DeepseekV2MoE",
    hidden_states: torch.Tensor,
    forward_batch: Optional["ForwardBatch"],
    input_ids_global: Optional[torch.Tensor],
    num_tokens: int,
) -> torch.Tensor:
    import deep_gemm

    from sglang.srt.distributed.parallel_state import get_moe_ep_group

    hidden_size = moe.config.hidden_size

    if num_tokens > 0:
        router_logits = moe.gate(hidden_states, forward_batch=forward_batch)
        topk_kwargs = {"input_ids": input_ids_global} if moe.is_hash else {}
        topk_output = moe.topk(
            hidden_states,
            router_logits,
            num_token_non_padded=(
                forward_batch.num_token_non_padded
                if forward_batch is not None
                else None
            ),
            expert_location_dispatch_info=ExpertLocationDispatchInfo.init_new(
                layer_id=moe.layer_id,
            ),
            **topk_kwargs,
        )
        topk_ids = topk_output.topk_ids
        topk_weights = topk_output.topk_weights
    else:
        topk_ids = None
        topk_weights = None

    ep_group = get_moe_ep_group().device_group
    num_experts = moe.experts.num_experts
    top_k = moe.config.num_experts_per_tok + moe.num_fused_shared_experts
    intermediate_size = moe.config.moe_intermediate_size
    num_max_tokens_per_rank = (
        envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_NUM_MAX_TOKENS_PER_RANK.get()
    )
    assert num_tokens <= num_max_tokens_per_rank, (
        f"mega MoE: num_tokens={num_tokens} exceeds cap "
        f"SGLANG_OPT_DEEPGEMM_MEGA_MOE_NUM_MAX_TOKENS_PER_RANK="
        f"{num_max_tokens_per_rank}; raise the env var or shrink "
        f"cuda_graph_max_bs / chunked_prefill_size accordingly"
    )

    buf = _get_mega_moe_symm_buffer(
        ep_group,
        num_experts=num_experts,
        num_max_tokens_per_rank=num_max_tokens_per_rank,
        num_topk=top_k,
        hidden=hidden_size,
        intermediate_hidden=intermediate_size,
    )

    if num_tokens > 0:
        topk_ids_in = topk_ids.to(torch.int32)
        topk_weights_in = topk_weights.to(torch.float32)
    else:
        topk_ids_in = hidden_states.new_empty((0, top_k), dtype=torch.int32)
        topk_weights_in = hidden_states.new_empty((0, top_k), dtype=torch.float32)

    use_fp4_acts = envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_FP4_ACTS.get()
    if use_fp4_acts:
        # FP4 path goes through DeepGEMM's mega_moe_pre_dispatch which
        # handles the E2M1 packing variant. The jit implementation
        # only emits FP8.
        deep_gemm.mega_moe_pre_dispatch(
            hidden_states,
            topk_ids_in,
            topk_weights_in,
            buf.x,
            buf.x_sf,
            buf.topk_idx,
            buf.topk_weights,
            num_tokens=num_tokens,
            group_size=32,
            use_fp4_acts=True,
        )
    else:
        mega_moe_pre_dispatch(
            hidden_states,
            topk_ids_in,
            topk_weights_in,
            buf.x,
            buf.x_sf,
            buf.topk_idx,
            buf.topk_weights,
            quant_group_size=32,
        )

    # Allocate at least one row so y has a non-null CUDA data_ptr;
    # the DeepGEMM tvm-ffi binding rejects nullptr in convert_to_torch_tensor().
    y = torch.empty(
        (max(num_tokens, 1), hidden_size),
        dtype=torch.bfloat16,
        device=hidden_states.device,
    )
    swiglu_limit = getattr(moe.config, "swiglu_limit", None)
    deep_gemm.fp8_fp4_mega_moe(
        y,
        moe.experts.mega_l1_weights,
        moe.experts.mega_l2_weights,
        buf,
        recipe=(1, 1, 32),
        activation="swiglu",
        activation_clamp=swiglu_limit,
        fast_math=True,
    )
    y = y[:num_tokens]

    if not moe.experts.should_fuse_routed_scaling_factor_in_topk:
        y.mul_(moe.routed_scaling_factor)
    return y
```
**EN:** This block defines `_run_mega_routed` and contains the main logic for this step. It mainly invokes `envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_NUM_MAX_TOKENS_PER_RANK.get`, `_get_mega_moe_symm_buffer`, `envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_FP4_ACTS.get`, `torch.empty`, and `getattr`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_size`, `ep_group`, `num_experts`, `top_k`, and `intermediate_size` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_run_mega_routed`，并承载这一阶段的核心逻辑。 它主要调用 `envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_NUM_MAX_TOKENS_PER_RANK.get`、`_get_mega_moe_symm_buffer`、`envs.SGLANG_OPT_DEEPGEMM_MEGA_MOE_USE_FP4_ACTS.get`、`torch.empty` 以及 `getattr`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_size`、`ep_group`、`num_experts`、`top_k` 以及 `intermediate_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 270-332: `build_mega_moe_experts_weights` constructor for mega Mixture-of-Experts experts weights
```python
def build_mega_moe_experts_weights(experts) -> None:
    from deep_gemm import (
        transform_sf_into_required_layout,
        transform_weights_for_mega_moe,
    )
    from deep_gemm.mega import _interleave_l1_weights, _transpose_sf_for_utccp

    if getattr(experts, "_mega_moe_weights_built", False):
        return

    w13 = experts.w13_weight.data
    w13_sf_fp32 = experts.w13_weight_scale_inv.data
    w2 = experts.w2_weight.data
    w2_sf_fp32 = experts.w2_weight_scale_inv.data

    num_groups, n1, half_k1 = w13.shape
    k1 = half_k1 * 2
    _, n2, half_k2 = w2.shape
    k2 = half_k2 * 2

    w13_sf = transform_sf_into_required_layout(
        w13_sf_fp32,
        mn=n1,
        k=k1,
        recipe=(1, 32),
        num_groups=num_groups,
        disable_ue8m0_cast=False,
    )
    w2_sf = transform_sf_into_required_layout(
        w2_sf_fp32,
        mn=n2,
        k=k2,
        recipe=(1, 32),
        num_groups=num_groups,
        disable_ue8m0_cast=False,
    )

    if envs.SGLANG_OPT_FIX_MEGA_MOE_MEMORY.get():
        # Build the interleaved L1 weight + scale once; share the weight buffer
        # between `w13_weight.data` (normal deep-ep path) and `mega_l1_weights[0]`
        # (mega moe path). Mega moe additionally needs a UTCCP-transposed scale;
        # the deep-ep path consumes the non-transposed interleaved scale and a
        # swizzle-aware activation kernel. L2 weight is untouched by the mega
        # transform, so the existing `w2_weight.data` is shared directly.
        w13_interleaved, w13_sf_interleaved = _interleave_l1_weights((w13, w13_sf))
        w13_sf_utccp = _transpose_sf_for_utccp(w13_sf_interleaved)
        w2_sf_utccp = _transpose_sf_for_utccp(w2_sf)

        experts.w13_weight.data = w13_interleaved
        experts.w13_weight_scale_inv.data = w13_sf_interleaved
        experts.w2_weight_scale_inv.data = w2_sf
        experts.w13_weight_scale_inv.format_ue8m0 = True
        experts.w2_weight_scale_inv.format_ue8m0 = True

        experts.mega_l1_weights = (experts.w13_weight.data, w13_sf_utccp)
        experts.mega_l2_weights = (experts.w2_weight.data, w2_sf_utccp)
    else:
        l1_pair, l2_pair = transform_weights_for_mega_moe((w13, w13_sf), (w2, w2_sf))

        experts.mega_l1_weights = l1_pair
        experts.mega_l2_weights = l2_pair

    experts._mega_moe_weights_built = True
```
**EN:** This block defines `build_mega_moe_experts_weights` and contains the main logic for this step. It mainly invokes `getattr`, `transform_sf_into_required_layout`, `envs.SGLANG_OPT_FIX_MEGA_MOE_MEMORY.get`, `_interleave_l1_weights`, and `_transpose_sf_for_utccp`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `w13`, `w13_sf_fp32`, `w2`, `w2_sf_fp32`, and `num_groups` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `build_mega_moe_experts_weights`，并承载这一阶段的核心逻辑。 它主要调用 `getattr`、`transform_sf_into_required_layout`、`envs.SGLANG_OPT_FIX_MEGA_MOE_MEMORY.get`、`_interleave_l1_weights` 以及 `_transpose_sf_for_utccp`，说明该流程会编排底层辅助函数或计算内核。 像 `w13`、`w13_sf_fp32`、`w2`、`w2_sf_fp32` 以及 `num_groups` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_apply_mega_moe_dg_env`, `_get_mega_moe_symm_buffer`, `should_use_mega_moe`, `forward_mega_moe`, and `_run_mega_routed`. / **主要符号**：核心入口包括 `_apply_mega_moe_dg_env`、`_get_mega_moe_symm_buffer`、`should_use_mega_moe`、`forward_mega_moe` 以及 `_run_mega_routed`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `os`, `contextlib.nullcontext`, `typing.TYPE_CHECKING`, and `typing.Optional` / **标准库**：`__future__.annotations`、`os`、`contextlib.nullcontext`、`typing.TYPE_CHECKING` 以及 `typing.Optional`
- **Third-party**: `torch`, `deep_gemm.SymmBuffer`, `deep_gemm`, `deep_gemm.transform_sf_into_required_layout`, `deep_gemm.transform_weights_for_mega_moe`, `deep_gemm.mega._interleave_l1_weights`, and `deep_gemm.mega._transpose_sf_for_utccp` / **第三方依赖**：`torch`、`deep_gemm.SymmBuffer`、`deep_gemm`、`deep_gemm.transform_sf_into_required_layout`、`deep_gemm.transform_weights_for_mega_moe`、`deep_gemm.mega._interleave_l1_weights` 以及 `deep_gemm.mega._transpose_sf_for_utccp`
- **Internal SGLang modules**: `sglang.jit_kernel.deepseek_v4.mega_moe_pre_dispatch`, `sglang.srt.environ.envs`, `sglang.srt.eplb.expert_location_dispatch.ExpertLocationDispatchInfo`, `sglang.srt.layers.dp_attention.get_dp_global_num_tokens`, `sglang.srt.layers.moe.utils.get_moe_a2a_backend`, `sglang.srt.model_executor.cuda_graph_runner.get_is_capture_mode`, `sglang.srt.model_executor.forward_batch_info.ForwardBatch`, `sglang.srt.models.deepseek_v2.DeepseekV2MoE`, and `sglang.srt.distributed.parallel_state.get_moe_ep_group` / **SGLang 内部模块**：`sglang.jit_kernel.deepseek_v4.mega_moe_pre_dispatch`、`sglang.srt.environ.envs`、`sglang.srt.eplb.expert_location_dispatch.ExpertLocationDispatchInfo`、`sglang.srt.layers.dp_attention.get_dp_global_num_tokens`、`sglang.srt.layers.moe.utils.get_moe_a2a_backend`、`sglang.srt.model_executor.cuda_graph_runner.get_is_capture_mode`、`sglang.srt.model_executor.forward_batch_info.ForwardBatch`、`sglang.srt.models.deepseek_v2.DeepseekV2MoE` 以及 `sglang.srt.distributed.parallel_state.get_moe_ep_group`
