# communicator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/communicator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with low-precision quantization data paths for the SGLang SRT runtime. It exposes symbols such as `_fused_rmsnorm_fp8_per_token_quant`, `apply_flashinfer_allreduce_fusion`, `apply_aiter_all_reduce_fusion`, and `ScatterMode` and connects them to backend-specific paths such as `CUDA`, `Triton`, `CUTLASS`, and `FlashInfer`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合低精度量化数据路径。它提供了 `_fused_rmsnorm_fp8_per_token_quant`、`apply_flashinfer_allreduce_fusion`、`apply_aiter_all_reduce_fusion` 以及 `ScatterMode` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`CUTLASS` 以及 `FlashInfer` 等后端专用路径。

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

### Lines 14-106: Imports, conditional backend setup, and runtime guards
```python
import logging
from contextlib import contextmanager
from dataclasses import dataclass
from enum import Enum, auto
from functools import partial
from typing import Callable, Dict, List, Optional, Tuple, Union

import torch

from sglang.srt.distributed import (
    attention_tensor_model_parallel_all_reduce,
    attention_tensor_model_parallel_quant_all_reduce,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    get_tp_group,
    moe_tensor_model_parallel_all_reduce,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    use_symmetric_memory,
)
from sglang.srt.environ import envs
from sglang.srt.layers.attention.nsa.utils import (
    is_nsa_enable_prefill_cp,
    nsa_use_prefill_cp,
)
from sglang.srt.layers.dp_attention import (
    attn_tp_all_gather_into_tensor,
    attn_tp_reduce_scatter_tensor,
    dp_gather_partial,
    dp_reduce_scatter_tensor,
    dp_scatter,
    get_attention_cp_rank,
    get_attention_cp_size,
    get_attention_dp_size,
    get_attention_tp_group,
    get_attention_tp_rank,
    get_attention_tp_size,
    get_dp_global_num_tokens,
    get_global_dp_buffer,
    get_local_dp_buffer,
    get_moe_cp_rank,
    get_moe_cp_size,
    is_allocation_symmetric,
    is_dp_attention_enabled,
    is_enable_moe_cp_allgather,
    moe_cp_all_gather_into_tensor,
)
from sglang.srt.layers.flashinfer_comm_fusion import is_flashinfer_allreduce_unavailable
from sglang.srt.layers.moe import (
    get_moe_a2a_backend,
    should_use_dp_reduce_scatterv,
    should_use_flashinfer_cutlass_moe_fp4_allgather,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.server_args import get_global_server_args
from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
from sglang.srt.utils import (
    get_bool_env_var,
    is_cuda,
    is_flashinfer_available,
    is_gfx95_supported,
    is_hip,
    is_npu,
    is_sm90_supported,
    is_sm100_supported,
)

_is_cuda = is_cuda()
_is_flashinfer_available = is_flashinfer_available()
_is_sm90_supported = _is_cuda and is_sm90_supported()
_is_sm100_supported = _is_cuda and is_sm100_supported()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and is_hip()
_is_gfx95_supported = is_gfx95_supported()
_is_npu = is_npu()
_use_ag_after_qlora = envs.SGLANG_USE_AG_AFTER_QLORA.get()

if _use_aiter:
    from aiter.ops.rmsnorm import add_rmsnorm_quant as _aiter_add_rmsnorm_quant
    from aiter.ops.rmsnorm import rmsnorm_quant as _aiter_rmsnorm_quant

    from sglang.srt.layers.quantization.fp8_kernel import fp8_dtype as _aiter_fp8_dtype

    if _is_gfx95_supported:
        from aiter.ops.triton.fused_fp8_quant import fused_rms_fp8_group_quant

        from sglang.srt.layers.quantization.rocm_mxfp4_utils import (
            fused_rms_mxfp4_quant,
        )
elif _is_npu:
    from sglang.srt.hardware_backend.npu.cmo import prepare_weight_cache
```
**EN:** This section prepares the module namespace. It imports `logging`, `contextlib.contextmanager`, `dataclasses.dataclass`, `enum.Enum`, `enum.auto`, and `functools.partial`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_cuda`, `_is_flashinfer_available`, `_is_sm90_supported`, `_is_sm100_supported`, and `_use_aiter` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`contextlib.contextmanager`、`dataclasses.dataclass`、`enum.Enum`、`enum.auto` 以及 `functools.partial`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_cuda`、`_is_flashinfer_available`、`_is_sm90_supported`、`_is_sm100_supported` 以及 `_use_aiter` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 107-154: Internal helper `_fused_rmsnorm_fp8_per_token_quant`
```python
def _fused_rmsnorm_fp8_per_token_quant(
    hidden_states: torch.Tensor,
    weight: torch.Tensor,
    epsilon: float,
    residual: Optional[torch.Tensor] = None,
):
    """Fused (optional residual-add +) RMSNorm + FP8 per-token quantization.

    Only used with the aiter (ROCm) backend.

    Args:
        residual: if provided, computes hidden_states + residual before RMSNorm
                  and returns updated residual_out as second element.

    Returns:
        If residual is None:  (out_fp8, scale)
        If residual provided: ((out_fp8, scale), residual_out)
    """
    M, N = hidden_states.shape
    out_fp8 = torch.empty((M, N), dtype=_aiter_fp8_dtype, device=hidden_states.device)
    scale = torch.empty(M, dtype=torch.float32, device=hidden_states.device)
    if residual is not None:
        residual_out = torch.empty_like(hidden_states)
        _aiter_add_rmsnorm_quant(
            out_fp8,
            hidden_states,
            residual,
            residual_out,
            scale,
            weight,
            epsilon,
            0,  # group_size=0 → per-token
        )
        return (out_fp8, scale.unsqueeze(1)), residual_out
    else:
        _aiter_rmsnorm_quant(
            out_fp8,
            hidden_states,
            scale,
            weight,
            epsilon,
            0,  # group_size=0 → per-token
        )
        return (out_fp8, scale.unsqueeze(1))


# TODO: According to the discussion in https://github.com/flashinfer-ai/flashinfer/issues/1223#issuecomment-3047256465
# We set the max token num to 128 for allreduce fusion with min-latency case(use_oneshot=True).
```
**EN:** This block defines `_fused_rmsnorm_fp8_per_token_quant` and contains the main logic for this step. It mainly invokes `torch.empty`, `torch.empty_like`, `_aiter_add_rmsnorm_quant`, `_aiter_rmsnorm_quant`, and `scale.unsqueeze`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `M`, `N`, `out_fp8`, `scale`, and `residual_out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_fused_rmsnorm_fp8_per_token_quant`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`torch.empty_like`、`_aiter_add_rmsnorm_quant`、`_aiter_rmsnorm_quant` 以及 `scale.unsqueeze`，说明该流程会编排底层辅助函数或计算内核。 像 `M`、`N`、`out_fp8`、`scale` 以及 `residual_out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 155-157: Module constants and shared configuration
```python
FUSE_ALLREDUCE_MAX_BATCH_SIZE = 2048
```
**EN:** This section prepares the module namespace. Shared names such as `FUSE_ALLREDUCE_MAX_BATCH_SIZE` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `FUSE_ALLREDUCE_MAX_BATCH_SIZE` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 158-171: `apply_flashinfer_allreduce_fusion` apply step for FlashInfer all-reduce fusion
```python
def apply_flashinfer_allreduce_fusion(batch_size: int):
    return (
        # NOTE: flashinfer 0.6.1 caused performance regression on sm100 for allreduce fusion
        # Ref: https://github.com/sgl-project/sglang/issues/17237
        (_is_sm90_supported or _is_sm100_supported)
        and _is_flashinfer_available
        and batch_size > 0
        and batch_size <= FUSE_ALLREDUCE_MAX_BATCH_SIZE
        and not is_dp_attention_enabled()
        and get_global_server_args().enable_flashinfer_allreduce_fusion
        and not is_flashinfer_allreduce_unavailable()
    )
```
**EN:** This block defines `apply_flashinfer_allreduce_fusion` and contains the main logic for this step. It mainly invokes `is_dp_attention_enabled`, `get_global_server_args`, and `is_flashinfer_allreduce_unavailable`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `apply_flashinfer_allreduce_fusion`，并承载这一阶段的核心逻辑。 它主要调用 `is_dp_attention_enabled`、`get_global_server_args` 以及 `is_flashinfer_allreduce_unavailable`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 172-186: `apply_aiter_all_reduce_fusion` apply step for aiter all reduce fusion
```python
def apply_aiter_all_reduce_fusion(input_tensor: torch.Tensor):
    n = input_tensor.shape[-1]
    total_bytes = input_tensor.numel() * input_tensor.element_size()
    # Aiter's should_custom_ar uses <= max_size/2 (64 MB); match that boundary.
    return (
        _use_aiter
        and total_bytes > 0
        and n <= 16384
        and total_bytes <= 8 * 1024 * 8192
        and get_tensor_model_parallel_world_size() != 6
        and not is_dp_attention_enabled()
        and get_global_server_args().enable_aiter_allreduce_fusion
    )
```
**EN:** This block defines `apply_aiter_all_reduce_fusion` and contains the main logic for this step. It mainly invokes `input_tensor.numel`, `input_tensor.element_size`, `get_tensor_model_parallel_world_size`, `is_dp_attention_enabled`, and `get_global_server_args`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `n` and `total_bytes` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `apply_aiter_all_reduce_fusion`，并承载这一阶段的核心逻辑。 它主要调用 `input_tensor.numel`、`input_tensor.element_size`、`get_tensor_model_parallel_world_size`、`is_dp_attention_enabled` 以及 `get_global_server_args`，说明该流程会编排底层辅助函数或计算内核。 像 `n` 和 `total_bytes` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 187-201: Class `ScatterMode` declaration and shared state
```python
class ScatterMode(Enum):
    """
    Suppose we have TP=4, DP=2, enable-dp-attention, and the system handles seq a,b,c,d
    Model input/output: [ab, ab, cd, cd] for four ranks respectively
    SCATTERED: [a, b, c, d]
    TP_ATTN_FULL: [ab, ab, cd, cd], i.e. all ranks inside a TP attn group have full data of the group
    FULL: [abcd, abcd, abcd, abcd]
    MOE_FULL: full within the MoE group (cp_per_moe CP chunks), used when moe_dp_size < attn_cp_size
    """

    SCATTERED = auto()
    TP_ATTN_FULL = auto()
    FULL = auto()
    MOE_FULL = auto()
```
**EN:** This block introduces class `ScatterMode` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Suppose we have TP=4, DP=2, enable-dp-attention, and the system handles seq a,b,c,d Model input/output: [ab, ab, cd, cd] for four ranks respectively SCATTERED: [a, b, c, d] TP_ATTN_FULL: [ab, ab, cd, cd], i.e.
**CN:** 该代码块引入类 `ScatterMode`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 202-210: Function `ScatterMode.model_input_output` and its core logic
```python
    @staticmethod
    def model_input_output():
        """The scatter mode for model forward pass input and output data"""
        if is_nsa_enable_prefill_cp():
            return ScatterMode.SCATTERED

        return ScatterMode.TP_ATTN_FULL
```
**EN:** This block defines `ScatterMode.model_input_output` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `is_nsa_enable_prefill_cp`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ScatterMode.model_input_output`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `is_nsa_enable_prefill_cp`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 211-211: Class `AttentionInputs` declaration and shared state
```python
class AttentionInputs:
```
**EN:** This block introduces class `AttentionInputs` and the state shared by its methods.
**CN:** 该代码块引入类 `AttentionInputs`，并定义其方法共享的状态。

### Lines 212-223: `AttentionInputs` initialization and state setup
```python
    def __init__(
        self,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        qkv_latent_func: Callable,
    ):
        self.hidden_states_local = hidden_states
        self.forward_batch = forward_batch
        self.qkv_latent_func = qkv_latent_func
        self.hidden_states_ = None
        self.qkv_latent_ = None
```
**EN:** This block defines `AttentionInputs.__init__` and contains the main logic for this step. Intermediate names such as `self.hidden_states_local`, `self.forward_batch`, `self.qkv_latent_func`, `self.hidden_states_`, and `self.qkv_latent_` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `AttentionInputs.__init__`，并承载这一阶段的核心逻辑。 像 `self.hidden_states_local`、`self.forward_batch`、`self.qkv_latent_func`、`self.hidden_states_` 以及 `self.qkv_latent_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 224-229: Function `AttentionInputs.tp_all_gather_hidden_states` and its core logic
```python
    def tp_all_gather_hidden_states(self, hidden_states, forward_batch):
        total_tokens = forward_batch.input_ids.shape[0]
        output = hidden_states.new_empty((total_tokens, hidden_states.shape[-1]))
        get_tp_group().all_gather_into_tensor(output, hidden_states)
        return output
```
**EN:** This block defines `AttentionInputs.tp_all_gather_hidden_states` and contains the main logic for this step. It mainly invokes `hidden_states.new_empty`, `get_tp_group.all_gather_into_tensor`, and `get_tp_group`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `total_tokens` and `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `AttentionInputs.tp_all_gather_hidden_states`，并承载这一阶段的核心逻辑。 它主要调用 `hidden_states.new_empty`、`get_tp_group.all_gather_into_tensor` 以及 `get_tp_group`，说明该流程会编排底层辅助函数或计算内核。 像 `total_tokens` 和 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 230-242: Function `AttentionInputs.fetch_qkv_latent` and its core logic
```python
    def fetch_qkv_latent(self):
        if self.qkv_latent_ is not None:
            return self.qkv_latent_
        assert self.qkv_latent_func is not None
        self.qkv_latent_ = self.qkv_latent_func(
            self.hidden_states_local, self.forward_batch
        )
        if get_attn_tp_context().input_scattered:
            self.qkv_latent_ = self.tp_all_gather_hidden_states(
                self.qkv_latent_, self.forward_batch
            )
        return self.qkv_latent_
```
**EN:** This block defines `AttentionInputs.fetch_qkv_latent` and contains the main logic for this step. It mainly invokes `self.qkv_latent_func`, `get_attn_tp_context`, and `self.tp_all_gather_hidden_states`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.qkv_latent_` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `AttentionInputs.fetch_qkv_latent`，并承载这一阶段的核心逻辑。 它主要调用 `self.qkv_latent_func`、`get_attn_tp_context` 以及 `self.tp_all_gather_hidden_states`，说明该流程会编排底层辅助函数或计算内核。 像 `self.qkv_latent_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 243-253: Function `AttentionInputs.fetch_hidden_states` and its core logic
```python
    def fetch_hidden_states(self):
        if self.hidden_states_ is not None:
            return self.hidden_states_
        self.hidden_states_ = self.hidden_states_local
        if get_attn_tp_context().input_scattered:
            self.hidden_states_ = self.tp_all_gather_hidden_states(
                self.hidden_states_, self.forward_batch
            )
        return self.hidden_states_
```
**EN:** This block defines `AttentionInputs.fetch_hidden_states` and contains the main logic for this step. It mainly invokes `get_attn_tp_context` and `self.tp_all_gather_hidden_states`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.hidden_states_` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `AttentionInputs.fetch_hidden_states`，并承载这一阶段的核心逻辑。 它主要调用 `get_attn_tp_context` 和 `self.tp_all_gather_hidden_states`，说明该流程会编排底层辅助函数或计算内核。 像 `self.hidden_states_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 254-254: Class `AttnTpContext` declaration and shared state
```python
class AttnTpContext:
```
**EN:** This block introduces class `AttnTpContext` and the state shared by its methods.
**CN:** 该代码块引入类 `AttnTpContext`，并定义其方法共享的状态。

### Lines 255-260: `AttnTpContext` initialization and state setup
```python
    def __init__(self):
        self.allow_input_scattered = False
        self.input_scattered_ = False
        self.attn_inputs_: Optional[AttentionInputs] = None
        self.is_nsa = False
```
**EN:** This block defines `AttnTpContext.__init__` and contains the main logic for this step. Intermediate names such as `self.allow_input_scattered`, `self.input_scattered_`, `self.attn_inputs_`, and `self.is_nsa` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `AttnTpContext.__init__`，并承载这一阶段的核心逻辑。 像 `self.allow_input_scattered`、`self.input_scattered_`、`self.attn_inputs_` 以及 `self.is_nsa` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 261-282: Function `AttnTpContext.init_context` and its core logic
```python
    def init_context(self, q_lora_rank, is_nsa):
        self.is_nsa = is_nsa
        self.allow_input_scattered = (
            get_global_server_args().enable_attn_tp_input_scattered
            and (_is_cuda or _is_npu)
            and q_lora_rank is not None
            and not is_nsa
            and get_tensor_model_parallel_world_size() > 1
            and not is_dp_attention_enabled()
            and get_moe_a2a_backend().is_none()
            and not enable_moe_dense_fully_dp()
            and get_global_server_args().disable_piecewise_cuda_graph
            and get_global_server_args().speculative_algorithm != "EAGLE3"
        )
        if get_global_server_args().enable_attn_tp_input_scattered:
            if not self.allow_input_scattered:
                logging.info(
                    "attn_tp_input_scattered is not enabled while other conditions are not met"
                )
            else:
                logging.info("attn_tp_input_scattered is enabled")
```
**EN:** This block defines `AttnTpContext.init_context` and contains the main logic for this step. It mainly invokes `get_moe_a2a_backend.is_none`, `get_global_server_args`, `get_tensor_model_parallel_world_size`, `is_dp_attention_enabled`, and `enable_moe_dense_fully_dp`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.is_nsa` and `self.allow_input_scattered` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `AttnTpContext.init_context`，并承载这一阶段的核心逻辑。 它主要调用 `get_moe_a2a_backend.is_none`、`get_global_server_args`、`get_tensor_model_parallel_world_size`、`is_dp_attention_enabled` 以及 `enable_moe_dense_fully_dp`，说明该流程会编排底层辅助函数或计算内核。 像 `self.is_nsa` 和 `self.allow_input_scattered` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 283-292: Function `AttnTpContext.use_input_scattered` and its core logic
```python
    def use_input_scattered(self, forward_batch: ForwardBatch):
        return (
            self.allow_input_scattered
            and forward_batch.forward_mode.is_extend()
            and not forward_batch.forward_mode.is_target_verify()
            and not forward_batch.forward_mode.is_draft_extend()
            and forward_batch.input_ids is not None
            and not forward_batch.can_run_tbo
        )
```
**EN:** This block defines `AttnTpContext.use_input_scattered` and contains the main logic for this step. It mainly invokes `forward_batch.forward_mode.is_extend`, `forward_batch.forward_mode.is_target_verify`, and `forward_batch.forward_mode.is_draft_extend`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `AttnTpContext.use_input_scattered`，并承载这一阶段的核心逻辑。 它主要调用 `forward_batch.forward_mode.is_extend`、`forward_batch.forward_mode.is_target_verify` 以及 `forward_batch.forward_mode.is_draft_extend`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 293-296: Function `AttnTpContext.input_scattered` and its core logic
```python
    @property
    def input_scattered(self):
        return self.input_scattered_
```
**EN:** This block defines `AttnTpContext.input_scattered` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `AttnTpContext.input_scattered`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 297-299: `AttnTpContext.set_attn_inputs` setter for attention inputs
```python
    def set_attn_inputs(self, attn_inputs: AttentionInputs):
        self.attn_inputs_ = attn_inputs
```
**EN:** This block defines `AttnTpContext.set_attn_inputs` and contains the main logic for this step. Intermediate names such as `self.attn_inputs_` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `AttnTpContext.set_attn_inputs`，并承载这一阶段的核心逻辑。 像 `self.attn_inputs_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 300-303: Function `AttnTpContext.fetch_qkv_latent` and its core logic
```python
    def fetch_qkv_latent(self):
        assert self.attn_inputs_ is not None
        return self.attn_inputs_.fetch_qkv_latent()
```
**EN:** This block defines `AttnTpContext.fetch_qkv_latent` and contains the main logic for this step. It mainly invokes `self.attn_inputs_.fetch_qkv_latent`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `AttnTpContext.fetch_qkv_latent`，并承载这一阶段的核心逻辑。 它主要调用 `self.attn_inputs_.fetch_qkv_latent`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 304-307: Function `AttnTpContext.fetch_hidden_states` and its core logic
```python
    def fetch_hidden_states(self):
        assert self.attn_inputs_ is not None
        return self.attn_inputs_.fetch_hidden_states()
```
**EN:** This block defines `AttnTpContext.fetch_hidden_states` and contains the main logic for this step. It mainly invokes `self.attn_inputs_.fetch_hidden_states`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `AttnTpContext.fetch_hidden_states`，并承载这一阶段的核心逻辑。 它主要调用 `self.attn_inputs_.fetch_hidden_states`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 308-310: Function `AttnTpContext.clear_attn_inputs` and its core logic
```python
    def clear_attn_inputs(self) -> None:
        self.attn_inputs_ = None
```
**EN:** This block defines `AttnTpContext.clear_attn_inputs` and contains the main logic for this step. Intermediate names such as `self.attn_inputs_` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `AttnTpContext.clear_attn_inputs`，并承载这一阶段的核心逻辑。 像 `self.attn_inputs_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 311-320: Function `AttnTpContext.maybe_input_scattered` and its core logic
```python
    @contextmanager
    def maybe_input_scattered(self, forward_batch: ForwardBatch):
        flag = self.use_input_scattered(forward_batch)
        old_flag = self.input_scattered
        self.input_scattered_ = flag
        yield
        self.input_scattered_ = old_flag
        self.attn_inputs_ = None
```
**EN:** This block defines `AttnTpContext.maybe_input_scattered` and contains the main logic for this step. Decorators like `contextmanager` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `self.use_input_scattered`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `flag`, `old_flag`, `self.input_scattered_`, and `self.attn_inputs_` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `AttnTpContext.maybe_input_scattered`，并承载这一阶段的核心逻辑。 像 `contextmanager` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `self.use_input_scattered`，说明该流程会编排底层辅助函数或计算内核。 像 `flag`、`old_flag`、`self.input_scattered_` 以及 `self.attn_inputs_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 321-323: Module constants and shared configuration
```python
ATTN_TP_CONTEXT = AttnTpContext()
```
**EN:** This section prepares the module namespace. Shared names such as `ATTN_TP_CONTEXT` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `ATTN_TP_CONTEXT` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 324-327: `get_attn_tp_context` getter for attention tensor parallel context
```python
def get_attn_tp_context():
    return ATTN_TP_CONTEXT
```
**EN:** This block defines `get_attn_tp_context` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_attn_tp_context`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 328-335: Class `_LayerModeComputationContext` declaration and shared state
```python
@dataclass
class _LayerModeComputationContext:
    num_layers: int
    layer_id: int
    is_layer_sparse: bool
    is_previous_layer_sparse: Optional[bool]
    is_next_layer_sparse: Optional[bool]
```
**EN:** This block introduces class `_LayerModeComputationContext` and the state shared by its methods.
**CN:** 该代码块引入类 `_LayerModeComputationContext`，并定义其方法共享的状态。

### Lines 336-346: Function `_LayerModeComputationContext.previous_layer` and its core logic
```python
    def previous_layer(self):
        assert self.is_previous_layer_sparse is not None
        return _LayerModeComputationContext(
            num_layers=self.num_layers,
            layer_id=self.layer_id - 1,
            is_layer_sparse=self.is_previous_layer_sparse,
            is_previous_layer_sparse=None,
            is_next_layer_sparse=self.is_layer_sparse,
        )
```
**EN:** This block defines `_LayerModeComputationContext.previous_layer` and contains the main logic for this step. It mainly invokes `_LayerModeComputationContext`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_LayerModeComputationContext.previous_layer`，并承载这一阶段的核心逻辑。 它主要调用 `_LayerModeComputationContext`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 347-355: Class `LayerScatterModes` declaration and shared state
```python
@dataclass
class LayerScatterModes:
    layer_input_mode: ScatterMode
    attn_mode: ScatterMode
    # Can be further split into e.g. mlp_input_mode and mlp_output_mode if needed
    mlp_mode: ScatterMode
    middle_residual_mode: ScatterMode
    layer_output_mode: ScatterMode
```
**EN:** This block introduces class `LayerScatterModes` and the state shared by its methods.
**CN:** 该代码块引入类 `LayerScatterModes`，并定义其方法共享的状态。

### Lines 356-366: Function `LayerScatterModes.init_new` and its core logic
```python
    @classmethod
    def init_new(cls, **kwargs):
        context = _LayerModeComputationContext(**kwargs)
        return cls(
            layer_input_mode=cls._compute_layer_input_mode(context),
            attn_mode=ScatterMode.TP_ATTN_FULL,
            mlp_mode=cls._compute_mlp_mode(context),
            middle_residual_mode=cls._compute_middle_residual_mode(context),
            layer_output_mode=cls._compute_layer_output_mode(context),
        )
```
**EN:** This block defines `LayerScatterModes.init_new` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `_LayerModeComputationContext`, `cls`, `cls._compute_layer_input_mode`, `cls._compute_mlp_mode`, and `cls._compute_middle_residual_mode`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `context` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerScatterModes.init_new`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `_LayerModeComputationContext`、`cls`、`cls._compute_layer_input_mode`、`cls._compute_mlp_mode` 以及 `cls._compute_middle_residual_mode`，说明该流程会编排底层辅助函数或计算内核。 像 `context` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 367-372: Internal helper `LayerScatterModes._compute_layer_input_mode`
```python
    @classmethod
    def _compute_layer_input_mode(cls, context: _LayerModeComputationContext):
        if context.layer_id == 0:
            return ScatterMode.model_input_output()
        return cls._compute_layer_output_mode(context.previous_layer())
```
**EN:** This block defines `LayerScatterModes._compute_layer_input_mode` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls._compute_layer_output_mode`, `ScatterMode.model_input_output`, and `context.previous_layer`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerScatterModes._compute_layer_input_mode`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls._compute_layer_output_mode`、`ScatterMode.model_input_output` 以及 `context.previous_layer`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 373-392: Internal helper `LayerScatterModes._compute_mlp_mode`
```python
    @classmethod
    def _compute_mlp_mode(cls, context: _LayerModeComputationContext):
        if context.is_layer_sparse:
            if (
                # Token dispatch/combine will be handled outside of LayerCommunicator for these modes.
                not get_moe_a2a_backend().is_none()
                or should_use_flashinfer_cutlass_moe_fp4_allgather()
            ):
                return ScatterMode.SCATTERED
            # NSA CP doesn't support MOE_FULL yet; fall back to FULL
            if is_enable_moe_cp_allgather() and not is_nsa_enable_prefill_cp():
                return ScatterMode.MOE_FULL
            return ScatterMode.FULL
        else:
            return (
                ScatterMode.SCATTERED
                if enable_moe_dense_fully_dp()
                else ScatterMode.FULL
            )
```
**EN:** This block defines `LayerScatterModes._compute_mlp_mode` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `should_use_flashinfer_cutlass_moe_fp4_allgather`, `is_enable_moe_cp_allgather`, `enable_moe_dense_fully_dp`, `get_moe_a2a_backend.is_none`, and `is_nsa_enable_prefill_cp`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerScatterModes._compute_mlp_mode`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `should_use_flashinfer_cutlass_moe_fp4_allgather`、`is_enable_moe_cp_allgather`、`enable_moe_dense_fully_dp`、`get_moe_a2a_backend.is_none` 以及 `is_nsa_enable_prefill_cp`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 393-401: Internal helper `LayerScatterModes._should_gather_for_tbo`
```python
    @classmethod
    def _should_gather_for_tbo(cls, context: _LayerModeComputationContext):
        return (
            not context.is_layer_sparse
            and context.is_next_layer_sparse
            and enable_moe_dense_fully_dp()
            and get_global_server_args().enable_two_batch_overlap
        )
```
**EN:** This block defines `LayerScatterModes._should_gather_for_tbo` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `enable_moe_dense_fully_dp` and `get_global_server_args`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerScatterModes._should_gather_for_tbo`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `enable_moe_dense_fully_dp` 和 `get_global_server_args`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 402-410: Internal helper `LayerScatterModes._compute_middle_residual_mode`
```python
    @classmethod
    def _compute_middle_residual_mode(cls, context: _LayerModeComputationContext):
        mlp_mode = cls._compute_mlp_mode(context)
        if mlp_mode == ScatterMode.SCATTERED:
            return ScatterMode.SCATTERED
        if mlp_mode in (ScatterMode.FULL, ScatterMode.MOE_FULL):
            return ScatterMode.TP_ATTN_FULL
        raise NotImplementedError
```
**EN:** This block defines `LayerScatterModes._compute_middle_residual_mode` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls._compute_mlp_mode`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `mlp_mode` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerScatterModes._compute_middle_residual_mode`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls._compute_mlp_mode`，说明该流程会编排底层辅助函数或计算内核。 像 `mlp_mode` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 411-424: Internal helper `LayerScatterModes._compute_layer_output_mode`
```python
    @classmethod
    def _compute_layer_output_mode(cls, context: _LayerModeComputationContext):
        mlp_mode = cls._compute_mlp_mode(context)
        if context.layer_id == context.num_layers - 1:
            return ScatterMode.model_input_output()
        if mlp_mode == ScatterMode.SCATTERED:
            if cls._should_gather_for_tbo(context):
                return ScatterMode.TP_ATTN_FULL
            return ScatterMode.SCATTERED
        if mlp_mode in (ScatterMode.FULL, ScatterMode.MOE_FULL):
            return ScatterMode.TP_ATTN_FULL
        raise NotImplementedError
```
**EN:** This block defines `LayerScatterModes._compute_layer_output_mode` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls._compute_mlp_mode`, `ScatterMode.model_input_output`, and `cls._should_gather_for_tbo`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `mlp_mode` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerScatterModes._compute_layer_output_mode`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls._compute_mlp_mode`、`ScatterMode.model_input_output` 以及 `cls._should_gather_for_tbo`，说明该流程会编排底层辅助函数或计算内核。 像 `mlp_mode` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 425-428: Function `enable_moe_dense_fully_dp` and its core logic
```python
def enable_moe_dense_fully_dp():
    return get_global_server_args().moe_dense_tp_size == 1
```
**EN:** This block defines `enable_moe_dense_fully_dp` and contains the main logic for this step. It mainly invokes `get_global_server_args`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `enable_moe_dense_fully_dp`，并承载这一阶段的核心逻辑。 它主要调用 `get_global_server_args`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 429-429: Class `LayerCommunicator` declaration and shared state
```python
class LayerCommunicator:
```
**EN:** This block introduces class `LayerCommunicator` and the state shared by its methods.
**CN:** 该代码块引入类 `LayerCommunicator`，并定义其方法共享的状态。

### Lines 430-452: `LayerCommunicator` initialization and state setup
```python
    def __init__(
        self,
        layer_scatter_modes: LayerScatterModes,
        input_layernorm: torch.nn.Module,
        post_attention_layernorm: torch.nn.Module,
        # Reduce scatter requires skipping all-reduce in model code after MoE/MLP, so only enable for models which have that implemented. Remove flag once done for all models that use LayerCommunicator.
        allow_reduce_scatter: bool = False,
        is_last_layer: bool = False,
        qkv_latent_func: Optional[Callable] = None,
    ):
        self.layer_scatter_modes = layer_scatter_modes
        self.input_layernorm = input_layernorm
        self.post_attention_layernorm = post_attention_layernorm
        self.allow_reduce_scatter = allow_reduce_scatter
        self.is_last_layer = is_last_layer
        self.qkv_latent_func = qkv_latent_func

        self._context = CommunicateContext.init_new()
        self._post_init_communicate()
        self._speculative_algo = SpeculativeAlgorithm.from_string(
            get_global_server_args().speculative_algorithm
        )
```
**EN:** This block defines `LayerCommunicator.__init__` and contains the main logic for this step. It mainly invokes `CommunicateContext.init_new`, `self._post_init_communicate`, `SpeculativeAlgorithm.from_string`, and `get_global_server_args`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.layer_scatter_modes`, `self.input_layernorm`, `self.post_attention_layernorm`, `self.allow_reduce_scatter`, and `self.is_last_layer` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `LayerCommunicator.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `CommunicateContext.init_new`、`self._post_init_communicate`、`SpeculativeAlgorithm.from_string` 以及 `get_global_server_args`，说明该流程会编排底层辅助函数或计算内核。 像 `self.layer_scatter_modes`、`self.input_layernorm`、`self.post_attention_layernorm`、`self.allow_reduce_scatter` 以及 `self.is_last_layer` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 453-476: Internal helper `LayerCommunicator._post_init_communicate`
```python
    def _post_init_communicate(self):
        self._communicate_simple_fn = CommunicateSimpleFn.get_fn(
            input_mode=self.layer_scatter_modes.layer_input_mode,
            output_mode=self.layer_scatter_modes.attn_mode,
            context=self._context,
        )
        self._communicate_with_all_reduce_and_layer_norm_fn = (
            CommunicateWithAllReduceAndLayerNormFn.get_fn(
                hidden_states_input_mode=self.layer_scatter_modes.attn_mode,
                residual_input_mode=self.layer_scatter_modes.layer_input_mode,
                hidden_states_output_mode=self.layer_scatter_modes.mlp_mode,
                residual_output_mode=self.layer_scatter_modes.middle_residual_mode,
                context=self._context,
            )
        )
        self._communicate_summable_tensor_pair_fn = (
            CommunicateSummableTensorPairFn.get_fn(
                hidden_states_input_mode=self.layer_scatter_modes.mlp_mode,
                residual_input_mode=self.layer_scatter_modes.middle_residual_mode,
                output_mode=self.layer_scatter_modes.layer_output_mode,
                context=self._context,
            )
        )
```
**EN:** This block defines `LayerCommunicator._post_init_communicate` and contains the main logic for this step. It mainly invokes `CommunicateSimpleFn.get_fn`, `CommunicateWithAllReduceAndLayerNormFn.get_fn`, and `CommunicateSummableTensorPairFn.get_fn`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._communicate_simple_fn`, `self._communicate_with_all_reduce_and_layer_norm_fn`, and `self._communicate_summable_tensor_pair_fn` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `LayerCommunicator._post_init_communicate`，并承载这一阶段的核心逻辑。 它主要调用 `CommunicateSimpleFn.get_fn`、`CommunicateWithAllReduceAndLayerNormFn.get_fn` 以及 `CommunicateSummableTensorPairFn.get_fn`，说明该流程会编排底层辅助函数或计算内核。 像 `self._communicate_simple_fn`、`self._communicate_with_all_reduce_and_layer_norm_fn` 以及 `self._communicate_summable_tensor_pair_fn` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 477-502: `LayerCommunicator.prepare_attn_and_capture_last_layer_outputs` step for attention and capture last layer outputs
```python
    def prepare_attn_and_capture_last_layer_outputs(
        self,
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        captured_last_layer_outputs: Optional[List[torch.Tensor]] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ):
        hidden_states, residual = self.prepare_attn(
            hidden_states,
            residual,
            forward_batch,
            post_residual_addition=post_residual_addition,
        )
        if captured_last_layer_outputs is not None:
            gathered_last_layer_output = self._communicate_simple_fn(
                hidden_states=residual,
                forward_batch=forward_batch,
                context=self._context,
            )
            if gathered_last_layer_output is residual:
                # Clone to avoid modifying the original residual by Custom RMSNorm inplace operation
                gathered_last_layer_output = residual.clone()
            captured_last_layer_outputs.append(gathered_last_layer_output)
        return hidden_states, residual
```
**EN:** This block defines `LayerCommunicator.prepare_attn_and_capture_last_layer_outputs` and contains the main logic for this step. It mainly invokes `self.prepare_attn`, `self._communicate_simple_fn`, `captured_last_layer_outputs.append`, and `residual.clone`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `residual`, and `gathered_last_layer_output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerCommunicator.prepare_attn_and_capture_last_layer_outputs`，并承载这一阶段的核心逻辑。 它主要调用 `self.prepare_attn`、`self._communicate_simple_fn`、`captured_last_layer_outputs.append` 以及 `residual.clone`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`residual` 以及 `gathered_last_layer_output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 503-649: `LayerCommunicator.prepare_attn` step for attention
```python
    def prepare_attn(
        self,
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        quant_format: str = "",
        post_residual_addition: Optional[torch.Tensor] = None,
    ):
        if get_attn_tp_context().input_scattered:
            hidden_states, residual = self._tp_reduce_scatter(
                hidden_states,
                residual,
            )
        if hidden_states.shape[0] == 0:
            residual = hidden_states
        else:
            if (
                residual is not None
                and hasattr(hidden_states, "_sglang_needs_allreduce_fusion")
                and hidden_states._sglang_needs_allreduce_fusion
            ):
                if (
                    apply_aiter_all_reduce_fusion(hidden_states)
                    or apply_flashinfer_allreduce_fusion(hidden_states.shape[0])
                ) and hasattr(self.input_layernorm, "forward_with_allreduce_fusion"):
                    hidden_states, residual = (
                        self.input_layernorm.forward_with_allreduce_fusion(
                            hidden_states, residual, use_attn_tp_group=False
                        )
                    )
                else:
                    hidden_states = moe_tensor_model_parallel_all_reduce(hidden_states)
                    hidden_states, residual = self.input_layernorm(
                        hidden_states, residual
                    )
            else:
                if residual is None:
                    residual = hidden_states

                    if _use_aiter and _is_gfx95_supported and ("mxfp4" in quant_format):
                        hidden_states, *_, _ = fused_rms_mxfp4_quant(
                            hidden_states,
                            self.input_layernorm.weight,
                            self.input_layernorm.variance_epsilon,
                            None,
                            None,
                            None,
                            None,
                        )
                    elif _use_aiter and _is_gfx95_supported and (quant_format == "fp8"):
                        # aiter (ROCm gfx95) fused RMSNorm + FP8 group quant.
                        # When NSA is active, also preserve the unquantized bf16
                        # output as a 3-tuple (fp8, scale, bf16) so the NSA
                        # indexer can skip redundant FP8 dequantization.
                        _nsa_needs_bf16 = get_attn_tp_context().is_nsa
                        hidden_states, _unq_bf16, _, _res = fused_rms_fp8_group_quant(
                            hidden_states,
                            self.input_layernorm.weight,
                            self.input_layernorm.variance_epsilon,
                            inp2=None,
                            inp2_weight=None,
                            inp2_epsilon=None,
                            group_size=128,
                            dtype_quant=torch.float8_e4m3fn,
                            res1=None,
                            output_unquantized_inp1=_nsa_needs_bf16,
                        )
                        if _nsa_needs_bf16:
                            hidden_states = (
                                hidden_states[0],
                                hidden_states[1],
                                _unq_bf16,
                            )

                    elif _use_aiter and (quant_format == "fp8_per_token"):
                        hidden_states = _fused_rmsnorm_fp8_per_token_quant(
                            hidden_states,
                            self.input_layernorm.weight.data,
                            self.input_layernorm.variance_epsilon,
                        )

                    else:
                        hidden_states = self.input_layernorm(hidden_states)
                else:
                    if _use_aiter and _is_gfx95_supported and ("mxfp4" in quant_format):
                        hidden_states, *_, residual = fused_rms_mxfp4_quant(
                            hidden_states,
                            self.input_layernorm.weight,
                            self.input_layernorm.variance_epsilon,
                            None,
                            None,
                            None,
                            residual,
                        )
                    elif _use_aiter and _is_gfx95_supported and (quant_format == "fp8"):
                        # aiter (ROCm gfx95) fused RMSNorm + FP8 group quant
                        # with residual addition. When NSA is active, pack
                        # the unquantized bf16 as a 3-tuple (fp8, scale, bf16).
                        _nsa_needs_bf16 = get_attn_tp_context().is_nsa
                        hidden_states, _unq_bf16, _, residual = (
                            fused_rms_fp8_group_quant(
                                hidden_states,
                                self.input_layernorm.weight,
                                self.input_layernorm.variance_epsilon,
                                inp2=None,
                                inp2_weight=None,
                                inp2_epsilon=None,
                                group_size=128,
                                dtype_quant=torch.float8_e4m3fn,
                                res1=residual,
                                output_unquantized_inp1=_nsa_needs_bf16,
                            )
                        )
                        if _nsa_needs_bf16:
                            hidden_states = (
                                hidden_states[0],
                                hidden_states[1],
                                _unq_bf16,
                            )
                    elif _use_aiter and (quant_format == "fp8_per_token"):
                        if post_residual_addition is not None:
                            residual = residual + post_residual_addition
                        hidden_states, residual = _fused_rmsnorm_fp8_per_token_quant(
                            hidden_states,
                            self.input_layernorm.weight.data,
                            self.input_layernorm.variance_epsilon,
                            residual=residual,
                        )
                    else:
                        hidden_states, residual = self.input_layernorm(
                            hidden_states,
                            residual,
                            post_residual_addition,
                        )

        hidden_states = self._communicate_simple_fn(
            hidden_states=hidden_states,
            forward_batch=forward_batch,
            context=self._context,
        )
        if self.qkv_latent_func is not None:
            attn_inputs = AttentionInputs(
                hidden_states, forward_batch, self.qkv_latent_func
            )
            get_attn_tp_context().set_attn_inputs(attn_inputs)
        return hidden_states, residual
```
**EN:** This block defines `LayerCommunicator.prepare_attn` and contains the main logic for this step. It mainly invokes `self._communicate_simple_fn`, `get_attn_tp_context`, `self._tp_reduce_scatter`, `AttentionInputs`, and `get_attn_tp_context.set_attn_inputs`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `residual`, `attn_inputs`, `_`, and `_nsa_needs_bf16` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerCommunicator.prepare_attn`，并承载这一阶段的核心逻辑。 它主要调用 `self._communicate_simple_fn`、`get_attn_tp_context`、`self._tp_reduce_scatter`、`AttentionInputs` 以及 `get_attn_tp_context.set_attn_inputs`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`residual`、`attn_inputs`、`_` 以及 `_nsa_needs_bf16` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 650-668: Internal helper `LayerCommunicator._tp_reduce_scatter`
```python
    def _tp_reduce_scatter(
        self,
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        if hidden_states.shape[0] == 0:
            return hidden_states, hidden_states
        assert (
            hidden_states.shape[0] % self._context.tp_size == 0
        ), f"Expected total tokens {hidden_states.shape[0]} % tp_size {self._context.tp_size} to be 0"
        local_tokens = hidden_states.shape[0] // self._context.tp_size
        output = hidden_states.new_empty(local_tokens, *hidden_states.shape[1:])
        get_tp_group().reduce_scatter_tensor(output, hidden_states)
        if residual is not None:
            residual = residual.tensor_split(self._context.tp_size)[
                self._context.tp_rank
            ]
        return output, residual
```
**EN:** This block defines `LayerCommunicator._tp_reduce_scatter` and contains the main logic for this step. It mainly invokes `hidden_states.new_empty`, `get_tp_group.reduce_scatter_tensor`, `get_tp_group`, and `residual.tensor_split`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `local_tokens`, `output`, and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerCommunicator._tp_reduce_scatter`，并承载这一阶段的核心逻辑。 它主要调用 `hidden_states.new_empty`、`get_tp_group.reduce_scatter_tensor`、`get_tp_group` 以及 `residual.tensor_split`，说明该流程会编排底层辅助函数或计算内核。 像 `local_tokens`、`output` 以及 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 669-686: `LayerCommunicator.prepare_mlp` step for mlp
```python
    def prepare_mlp(
        self,
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        cache=None,
    ):
        if cache is not None:
            self._context.cache = cache

        return self._communicate_with_all_reduce_and_layer_norm_fn(
            hidden_states=hidden_states,
            residual=residual,
            forward_batch=forward_batch,
            layernorm=self.post_attention_layernorm,
            context=self._context,
        )
```
**EN:** This block defines `LayerCommunicator.prepare_mlp` and contains the main logic for this step. It mainly invokes `self._communicate_with_all_reduce_and_layer_norm_fn`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerCommunicator.prepare_mlp`，并承载这一阶段的核心逻辑。 它主要调用 `self._communicate_with_all_reduce_and_layer_norm_fn`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 687-700: Function `LayerCommunicator.postprocess_layer` and its core logic
```python
    def postprocess_layer(
        self,
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
    ):
        return self._communicate_summable_tensor_pair_fn(
            hidden_states=hidden_states,
            residual=residual,
            forward_batch=forward_batch,
            context=self._context,
            allow_reduce_scatter=self.allow_reduce_scatter,
        )
```
**EN:** This block defines `LayerCommunicator.postprocess_layer` and contains the main logic for this step. It mainly invokes `self._communicate_summable_tensor_pair_fn`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerCommunicator.postprocess_layer`，并承载这一阶段的核心逻辑。 它主要调用 `self._communicate_summable_tensor_pair_fn`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 701-718: `LayerCommunicator.should_use_reduce_scatter` predicate for should use reduce scatter
```python
    def should_use_reduce_scatter(self, forward_batch: ForwardBatch):
        if not self.allow_reduce_scatter:
            return False
        if (
            self._communicate_summable_tensor_pair_fn
            is CommunicateSummableTensorPairFn._scatter_hidden_states
        ):
            if should_use_dp_reduce_scatterv():
                return True
            if forward_batch.dp_padding_mode.is_max_len():
                return True
        if nsa_use_prefill_cp(forward_batch):
            return True
        if get_attn_tp_context().input_scattered and not self.is_last_layer:
            return True
        return False

    # NOTE: This function will cause torch recompilation
```
**EN:** This block defines `LayerCommunicator.should_use_reduce_scatter` and contains the main logic for this step. It mainly invokes `nsa_use_prefill_cp`, `should_use_dp_reduce_scatterv`, `forward_batch.dp_padding_mode.is_max_len`, and `get_attn_tp_context`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerCommunicator.should_use_reduce_scatter`，并承载这一阶段的核心逻辑。 它主要调用 `nsa_use_prefill_cp`、`should_use_dp_reduce_scatterv`、`forward_batch.dp_padding_mode.is_max_len` 以及 `get_attn_tp_context`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 719-759: `LayerCommunicator.should_fuse_mlp_allreduce_with_next_layer` predicate for should fuse mlp all-reduce with next layer
```python
    def should_fuse_mlp_allreduce_with_next_layer(
        self, forward_batch: ForwardBatch
    ) -> bool:
        # When MOE_FULL is active (moe_cp allgather), fusion must be disabled because
        # the fusion path skips postprocess_layer which contains the moe_cp scatter.
        # Without scatter, hidden_states remain at MOE_FULL size while residual is at
        # TP_ATTN_FULL size, causing a shape mismatch.
        if is_enable_moe_cp_allgather():
            return False

        if (
            is_dp_attention_enabled()
            and self._speculative_algo is not None
            and self._speculative_algo.is_eagle()
        ):
            return False

        if get_attn_tp_context().input_scattered:
            return False

        batch_size = (
            forward_batch.input_ids.shape[0]
            if hasattr(forward_batch, "input_ids")
            else 0
        )

        return (
            (
                apply_flashinfer_allreduce_fusion(batch_size)
                or (
                    _use_aiter
                    and batch_size > 0
                    and get_tensor_model_parallel_world_size() != 6
                    and get_global_server_args().enable_aiter_allreduce_fusion
                )
            )
            and (not self.is_last_layer)
            and (self._context.tp_size > 1)
        )
```
**EN:** This block defines `LayerCommunicator.should_fuse_mlp_allreduce_with_next_layer` and contains the main logic for this step. It mainly invokes `is_enable_moe_cp_allgather`, `is_dp_attention_enabled`, `self._speculative_algo.is_eagle`, `get_attn_tp_context`, and `hasattr`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `batch_size` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerCommunicator.should_fuse_mlp_allreduce_with_next_layer`，并承载这一阶段的核心逻辑。 它主要调用 `is_enable_moe_cp_allgather`、`is_dp_attention_enabled`、`self._speculative_algo.is_eagle`、`get_attn_tp_context` 以及 `hasattr`，说明该流程会编排底层辅助函数或计算内核。 像 `batch_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 760-771: Class `CommunicateContext` declaration and shared state
```python
@dataclass
class CommunicateContext:
    process_group_sizes: Dict[ScatterMode, int]
    attn_tp_rank: int
    attn_tp_size: int
    attn_dp_size: int
    attn_cp_rank: int
    attn_cp_size: int
    tp_size: int
    cache = None
    tp_rank: int
```
**EN:** This block introduces class `CommunicateContext` and the state shared by its methods.
**CN:** 该代码块引入类 `CommunicateContext`，并定义其方法共享的状态。

### Lines 772-774: `CommunicateContext.is_same_group_size` predicate for is same group size
```python
    def is_same_group_size(self, a: ScatterMode, b: ScatterMode):
        return self.process_group_sizes[a] == self.process_group_sizes[b]
```
**EN:** This block defines `CommunicateContext.is_same_group_size` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateContext.is_same_group_size`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 775-805: Function `CommunicateContext.init_new` and its core logic
```python
    @classmethod
    def init_new(cls):
        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()
        attn_dp_size = get_attention_dp_size()
        attn_cp_size = get_attention_cp_size()
        attn_cp_rank = get_attention_cp_rank()
        tp_size = get_tensor_model_parallel_world_size()
        tp_rank = get_tensor_model_parallel_rank()
        moe_cp_size = get_moe_cp_size()
        process_group_sizes = {
            ScatterMode.SCATTERED: 1,
            ScatterMode.TP_ATTN_FULL: attn_tp_size,
            # TODO: support --moe-dense-tp-size > 1
            # With context parallel enabled, we should exclude
            # the attn_cp_size from the total tp_size
            ScatterMode.FULL: tp_size // attn_cp_size,
            ScatterMode.MOE_FULL: tp_size // (attn_cp_size // moe_cp_size),
        }
        return cls(
            process_group_sizes=process_group_sizes,
            attn_tp_rank=attn_tp_rank,
            attn_tp_size=attn_tp_size,
            attn_dp_size=attn_dp_size,
            attn_cp_rank=attn_cp_rank,
            attn_cp_size=attn_cp_size,
            tp_size=tp_size,
            tp_rank=tp_rank,
        )
```
**EN:** This block defines `CommunicateContext.init_new` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_attention_tp_rank`, `get_attention_tp_size`, `get_attention_dp_size`, `get_attention_cp_size`, and `get_attention_cp_rank`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `attn_tp_rank`, `attn_tp_size`, `attn_dp_size`, `attn_cp_size`, and `attn_cp_rank` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateContext.init_new`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_attention_tp_rank`、`get_attention_tp_size`、`get_attention_dp_size`、`get_attention_cp_size` 以及 `get_attention_cp_rank`，说明该流程会编排底层辅助函数或计算内核。 像 `attn_tp_rank`、`attn_tp_size`、`attn_dp_size`、`attn_cp_size` 以及 `attn_cp_rank` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 806-806: Class `CommunicateSimpleFn` declaration and shared state
```python
class CommunicateSimpleFn:
```
**EN:** This block introduces class `CommunicateSimpleFn` and the state shared by its methods.
**CN:** 该代码块引入类 `CommunicateSimpleFn`，并定义其方法共享的状态。

### Lines 807-824: `CommunicateSimpleFn.get_fn` getter for fn
```python
    @staticmethod
    def get_fn(
        input_mode: ScatterMode,
        output_mode: ScatterMode,
        context: CommunicateContext,
    ):
        if context.is_same_group_size(input_mode, output_mode):
            return CommunicateSimpleFn._trivial

        if (input_mode == ScatterMode.SCATTERED) and (
            output_mode == ScatterMode.TP_ATTN_FULL
        ):
            if _use_ag_after_qlora:
                return CommunicateSimpleFn._trivial
            return CommunicateSimpleFn._scattered_to_tp_attn_full

        raise NotImplementedError(f"{input_mode=} {output_mode=}")
```
**EN:** This block defines `CommunicateSimpleFn.get_fn` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `context.is_same_group_size` and `NotImplementedError`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateSimpleFn.get_fn`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `context.is_same_group_size` 和 `NotImplementedError`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 825-832: Internal helper `CommunicateSimpleFn._trivial`
```python
    @staticmethod
    def _trivial(
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        context: CommunicateContext,
    ) -> torch.Tensor:
        return hidden_states
```
**EN:** This block defines `CommunicateSimpleFn._trivial` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateSimpleFn._trivial`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 833-871: Internal helper `CommunicateSimpleFn._scattered_to_tp_attn_full`
```python
    @staticmethod
    def _scattered_to_tp_attn_full(
        hidden_states: Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]],
        forward_batch: ForwardBatch,
        context: CommunicateContext,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if isinstance(hidden_states, tuple):
            gathered_hidden_states = []
            for local_hidden_states in hidden_states:
                with use_symmetric_memory(
                    get_tp_group(),
                    disabled=not is_allocation_symmetric(),
                ):
                    output = torch.empty(
                        (
                            local_hidden_states.shape[0] * context.attn_tp_size,
                            *local_hidden_states.shape[1:],
                        ),
                        dtype=local_hidden_states.dtype,
                        device=local_hidden_states.device,
                    )
                attn_tp_all_gather_into_tensor(
                    output,
                    local_hidden_states,
                )
                gathered_hidden_states.append(output)
            return tuple(gathered_hidden_states)

        hidden_states, local_hidden_states = (
            get_local_dp_buffer(get_attention_tp_group()),
            hidden_states,
        )
        attn_tp_all_gather_into_tensor(
            hidden_states,
            local_hidden_states,
        )
        return hidden_states
```
**EN:** This block defines `CommunicateSimpleFn._scattered_to_tp_attn_full` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `isinstance`, `attn_tp_all_gather_into_tensor`, `tuple`, `get_local_dp_buffer`, and `gathered_hidden_states.append`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `local_hidden_states`, `gathered_hidden_states`, and `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateSimpleFn._scattered_to_tp_attn_full`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `isinstance`、`attn_tp_all_gather_into_tensor`、`tuple`、`get_local_dp_buffer` 以及 `gathered_hidden_states.append`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`local_hidden_states`、`gathered_hidden_states` 以及 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 872-877: Class `CommunicateWithAllReduceAndLayerNormFn` declaration and shared state
```python
class CommunicateWithAllReduceAndLayerNormFn:
    """Besides communication, needs to
    1. All reduce in tp_attn_group on hidden_states
    2. Apply layer norm
    """
```
**EN:** This block introduces class `CommunicateWithAllReduceAndLayerNormFn` and the state shared by its methods. The class docstring summarizes its role: Besides communication, needs to 1.
**CN:** 该代码块引入类 `CommunicateWithAllReduceAndLayerNormFn`，并定义其方法共享的状态。 类级文档进一步概括了它的职责。

### Lines 878-938: `CommunicateWithAllReduceAndLayerNormFn.get_fn` getter for fn
```python
    @staticmethod
    def get_fn(
        hidden_states_input_mode: ScatterMode,
        residual_input_mode: ScatterMode,
        hidden_states_output_mode: ScatterMode,
        residual_output_mode: ScatterMode,
        context: CommunicateContext,
    ):

        if (
            context.is_same_group_size(
                hidden_states_input_mode, hidden_states_output_mode
            )
            and context.is_same_group_size(residual_input_mode, residual_output_mode)
            and context.attn_tp_size == 1
        ):
            return CommunicateWithAllReduceAndLayerNormFn._simple

        if (
            (hidden_states_input_mode == ScatterMode.TP_ATTN_FULL)
            and (
                residual_input_mode in [ScatterMode.SCATTERED, ScatterMode.TP_ATTN_FULL]
            )
            and (hidden_states_output_mode == ScatterMode.FULL)
            and (residual_output_mode == ScatterMode.TP_ATTN_FULL)
        ):
            return partial(
                CommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual,
                residual_input_mode=residual_input_mode,
            )

        if (
            (hidden_states_input_mode == ScatterMode.TP_ATTN_FULL)
            and (
                residual_input_mode in [ScatterMode.SCATTERED, ScatterMode.TP_ATTN_FULL]
            )
            and (hidden_states_output_mode == ScatterMode.MOE_FULL)
            and (residual_output_mode == ScatterMode.TP_ATTN_FULL)
        ):
            return partial(
                CommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual_moe,
                residual_input_mode=residual_input_mode,
            )

        if (
            (hidden_states_input_mode == ScatterMode.TP_ATTN_FULL)
            and (
                residual_input_mode in [ScatterMode.SCATTERED, ScatterMode.TP_ATTN_FULL]
            )
            and (hidden_states_output_mode == ScatterMode.SCATTERED)
            and (residual_output_mode == ScatterMode.SCATTERED)
        ):
            return partial(
                CommunicateWithAllReduceAndLayerNormFn._scatter_hidden_states_and_residual,
                residual_input_mode=residual_input_mode,
            )

        raise NotImplementedError(
            f"{hidden_states_input_mode=} {residual_input_mode=} {hidden_states_output_mode=} {residual_output_mode=}"
        )
```
**EN:** This block defines `CommunicateWithAllReduceAndLayerNormFn.get_fn` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `NotImplementedError`, `context.is_same_group_size`, and `partial`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateWithAllReduceAndLayerNormFn.get_fn`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `NotImplementedError`、`context.is_same_group_size` 以及 `partial`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 939-951: Internal helper `CommunicateWithAllReduceAndLayerNormFn._simple`
```python
    @staticmethod
    def _simple(
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        layernorm: torch.nn.Module,
        context: CommunicateContext,
    ):
        # TODO move these `if shape != 0` into LayerNorm itself
        if hidden_states.shape[0] != 0:
            hidden_states, residual = layernorm(hidden_states, residual)
        return hidden_states, residual
```
**EN:** This block defines `CommunicateWithAllReduceAndLayerNormFn._simple` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `layernorm`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states` and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateWithAllReduceAndLayerNormFn._simple`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `layernorm`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states` 和 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 952-1026: Internal helper `CommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual`
```python
    @staticmethod
    def _gather_hidden_states_and_residual(
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        layernorm: torch.nn.Module,
        context: CommunicateContext,
        *,
        residual_input_mode,
    ):
        if get_attn_tp_context().input_scattered:
            return CommunicateWithAllReduceAndLayerNormFn._tp_all_reduce_with_scattered_residual(
                hidden_states,
                residual,
                layernorm,
                context,
            )

        if residual_input_mode == ScatterMode.SCATTERED and context.attn_tp_size > 1:
            residual, local_residual = (
                get_local_dp_buffer(get_attention_tp_group()),
                residual,
            )
            attn_tp_all_gather_into_tensor(residual, local_residual)
        if context.attn_dp_size != 1:
            # Perform layernorm on smaller data before comm. Only valid when attn_tp_size is 1 (tp_size == dp_size)
            use_layer_norm_before_gather = context.attn_tp_size == 1
            if use_layer_norm_before_gather and hidden_states.shape[0] != 0:
                with use_symmetric_memory(
                    get_tp_group(),
                    disabled=not is_allocation_symmetric(),
                ):
                    hidden_states, residual = layernorm(hidden_states, residual)
            elif context.attn_tp_rank == 0:
                hidden_states += residual

            hidden_states, local_hidden_states = (
                get_global_dp_buffer(get_tp_group()),
                hidden_states,
            )
            dp_gather_partial(hidden_states, local_hidden_states, forward_batch)

            if not use_layer_norm_before_gather:
                dp_scatter(residual, hidden_states, forward_batch)
                if hidden_states.shape[0] != 0:
                    hidden_states = layernorm(hidden_states)
        else:
            handled = False
            if (
                apply_aiter_all_reduce_fusion(hidden_states)
                or apply_flashinfer_allreduce_fusion(hidden_states.shape[0])
            ) and hasattr(layernorm, "forward_with_allreduce_fusion"):
                hidden_states, residual = layernorm.forward_with_allreduce_fusion(
                    hidden_states, residual, use_attn_tp_group=True
                )
                handled = True

            if not handled:
                quantize_communications = (
                    not forward_batch.forward_mode.is_decode_or_idle()
                    and get_global_server_args().enable_quant_communications
                )
                if quantize_communications:
                    hidden_states = attention_tensor_model_parallel_quant_all_reduce(
                        hidden_states
                    )
                else:
                    hidden_states = attention_tensor_model_parallel_all_reduce(
                        hidden_states
                    )
                if _is_npu and context.cache is not None:
                    _ = prepare_weight_cache(hidden_states, context.cache)
                hidden_states, residual = layernorm(hidden_states, residual)
        return hidden_states, residual
```
**EN:** This block defines `CommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_attn_tp_context`, `CommunicateWithAllReduceAndLayerNormFn._tp_all_reduce_with_scattered_residual`, `attn_tp_all_gather_into_tensor`, `dp_gather_partial`, and `get_local_dp_buffer`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `residual`, `local_residual`, `use_layer_norm_before_gather`, `hidden_states`, and `local_hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_attn_tp_context`、`CommunicateWithAllReduceAndLayerNormFn._tp_all_reduce_with_scattered_residual`、`attn_tp_all_gather_into_tensor`、`dp_gather_partial` 以及 `get_local_dp_buffer`，说明该流程会编排底层辅助函数或计算内核。 像 `residual`、`local_residual`、`use_layer_norm_before_gather`、`hidden_states` 以及 `local_hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1027-1047: Internal helper `CommunicateWithAllReduceAndLayerNormFn._scatter_hidden_states_and_residual`
```python
    @staticmethod
    def _scatter_hidden_states_and_residual(
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        layernorm: torch.nn.Module,
        context: CommunicateContext,
        *,
        residual_input_mode,
    ):
        input_hidden_states = hidden_states
        hidden_states = hidden_states.tensor_split(context.attn_tp_size)[
            context.attn_tp_rank
        ]
        attn_tp_reduce_scatter_tensor(hidden_states, input_hidden_states)
        if residual_input_mode == ScatterMode.TP_ATTN_FULL:
            residual = residual.tensor_split(context.attn_tp_size)[context.attn_tp_rank]
        if hidden_states.shape[0] != 0:
            hidden_states, residual = layernorm(hidden_states, residual)
        return hidden_states, residual
```
**EN:** This block defines `CommunicateWithAllReduceAndLayerNormFn._scatter_hidden_states_and_residual` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `attn_tp_reduce_scatter_tensor`, `hidden_states.tensor_split`, `layernorm`, and `residual.tensor_split`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `input_hidden_states`, `hidden_states`, and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateWithAllReduceAndLayerNormFn._scatter_hidden_states_and_residual`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `attn_tp_reduce_scatter_tensor`、`hidden_states.tensor_split`、`layernorm` 以及 `residual.tensor_split`，说明该流程会编排底层辅助函数或计算内核。 像 `input_hidden_states`、`hidden_states` 以及 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1048-1063: Internal helper `CommunicateWithAllReduceAndLayerNormFn._tp_all_reduce_with_scattered_residual`
```python
    @staticmethod
    def _tp_all_reduce_with_scattered_residual(
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        layernorm: torch.nn.Module,
        context: CommunicateContext,
    ):
        if hidden_states.shape[0] == 0:
            return hidden_states, hidden_states

        scattered_states = hidden_states.tensor_split(context.tp_size)[context.tp_rank]
        scattered_states += residual
        residual = tensor_model_parallel_all_reduce(hidden_states)
        hidden_states = layernorm(residual)
        return hidden_states, residual
```
**EN:** This block defines `CommunicateWithAllReduceAndLayerNormFn._tp_all_reduce_with_scattered_residual` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tensor_model_parallel_all_reduce`, `layernorm`, and `hidden_states.tensor_split`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `scattered_states`, `residual`, and `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateWithAllReduceAndLayerNormFn._tp_all_reduce_with_scattered_residual`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tensor_model_parallel_all_reduce`、`layernorm` 以及 `hidden_states.tensor_split`，说明该流程会编排底层辅助函数或计算内核。 像 `scattered_states`、`residual` 以及 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1064-1135: Internal helper `CommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual_moe`
```python
    @staticmethod
    def _gather_hidden_states_and_residual_moe(
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch,
        layernorm: torch.nn.Module,
        context: CommunicateContext,
        *,
        residual_input_mode,
    ):
        """Allgather tokens for MoE when moe_dp_size < attn_cp_size.

        Steps:
          1. Standard attn-TP all-reduce + optional DP allgather + layernorm (same as
             _gather_hidden_states_and_residual for the dp>1 case, or simple all-reduce
             + layernorm for dp==1).
          2. moe_cp allgather: gather tokens from cp_per_moe CP ranks so each rank holds
             all tokens for its MoE group.

        Residual is left at TP_ATTN_FULL throughout.
        """
        # Early return on empty tensor is safe for MOE_CP because:
        # - During CP extend: zigzag split guarantees all CP ranks have non-zero tokens,
        #   so no rank hits this path while others proceed to the allgather.
        # - During decode: moe_cp allgather is skipped (guarded by is_context_parallel_extend).
        # - CUDA graph warmup: not applicable when --disable-piecewise-cuda-graph is used.
        if hidden_states.shape[0] == 0:
            return hidden_states, residual

        # Step 1: Standard all-reduce/DP-allgather + layernorm (reuse existing logic).
        hidden_states, residual = (
            CommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual(
                hidden_states=hidden_states,
                residual=residual,
                forward_batch=forward_batch,
                layernorm=layernorm,
                context=context,
                residual_input_mode=residual_input_mode,
            )
        )

        # Step 2: moe_cp allgather — gather across cp_per_moe CP ranks.
        # Only active during prefill (context-parallel extend); decode keeps existing path.
        moe_cp_size = get_moe_cp_size()
        if (
            moe_cp_size > 1
            and hidden_states.shape[0] > 0
            and forward_batch.forward_mode.is_context_parallel_extend()
            and forward_batch.attn_cp_metadata is not None
        ):
            # Zigzag split can produce unequal token counts across CP ranks
            # (when seq_len % (cp_size * 2) != 0). NCCL allgather requires
            # equal input sizes, so pad to the max per-rank token count.
            per_rank_tokens = forward_batch.attn_cp_metadata.per_rank_actual_token
            max_tokens = max(per_rank_tokens)
            pad_size = max_tokens - hidden_states.shape[0]
            if pad_size > 0:
                hidden_states = torch.nn.functional.pad(
                    hidden_states, [0, 0, 0, pad_size]
                )

            output = torch.empty(
                (max_tokens * moe_cp_size, hidden_states.shape[1]),
                dtype=hidden_states.dtype,
                device=hidden_states.device,
            )
            moe_cp_all_gather_into_tensor(output, hidden_states)
            hidden_states = output

        return hidden_states, residual
```
**EN:** This block defines `CommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual_moe` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `CommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual`, `get_moe_cp_size`, `forward_batch.forward_mode.is_context_parallel_extend`, `max`, and `torch.empty`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `residual`, `moe_cp_size`, `per_rank_tokens`, and `max_tokens` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual_moe`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `CommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual`、`get_moe_cp_size`、`forward_batch.forward_mode.is_context_parallel_extend`、`max` 以及 `torch.empty`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`residual`、`moe_cp_size`、`per_rank_tokens` 以及 `max_tokens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1136-1138: Class `CommunicateSummableTensorPairFn` declaration and shared state
```python
class CommunicateSummableTensorPairFn:
    """It is allowed to make (hidden_states, residual) := (hidden_states + residual, None) if needed."""
```
**EN:** This block introduces class `CommunicateSummableTensorPairFn` and the state shared by its methods. The class docstring summarizes its role: It is allowed to make (hidden_states, residual) := (hidden_states + residual, None) if needed.
**CN:** 该代码块引入类 `CommunicateSummableTensorPairFn`，并定义其方法共享的状态。 类级文档进一步概括了它的职责。

### Lines 1139-1154: Function `CommunicateSummableTensorPairFn.execute` and its core logic
```python
    @classmethod
    def execute(
        cls,
        hidden_states_input_mode,
        residual_input_mode,
        output_mode,
        context,
        **kwargs,
    ):
        return cls.get_fn(
            hidden_states_input_mode=hidden_states_input_mode,
            residual_input_mode=residual_input_mode,
            output_mode=output_mode,
            context=context,
        )(context=context, **kwargs)
```
**EN:** This block defines `CommunicateSummableTensorPairFn.execute` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls.get_fn`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateSummableTensorPairFn.execute`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls.get_fn`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 1155-1198: `CommunicateSummableTensorPairFn.get_fn` getter for fn
```python
    @staticmethod
    def get_fn(
        hidden_states_input_mode: ScatterMode,
        residual_input_mode: ScatterMode,
        output_mode: ScatterMode,
        context: CommunicateContext,
    ):
        if context.is_same_group_size(
            hidden_states_input_mode, output_mode
        ) and context.is_same_group_size(residual_input_mode, output_mode):
            return CommunicateSummableTensorPairFn._trivial

        if (
            (hidden_states_input_mode == ScatterMode.FULL)
            and (residual_input_mode == ScatterMode.TP_ATTN_FULL)
            and (output_mode == ScatterMode.TP_ATTN_FULL)
        ):
            return CommunicateSummableTensorPairFn._scatter_hidden_states

        if (
            (hidden_states_input_mode == ScatterMode.SCATTERED)
            and (residual_input_mode == ScatterMode.SCATTERED)
            and (output_mode == ScatterMode.TP_ATTN_FULL)
        ):
            return CommunicateSummableTensorPairFn._gather

        if (
            (hidden_states_input_mode == ScatterMode.TP_ATTN_FULL)
            and (residual_input_mode == ScatterMode.TP_ATTN_FULL)
            and (output_mode == ScatterMode.SCATTERED)
        ):
            return CommunicateSummableTensorPairFn._scatter

        if (
            (hidden_states_input_mode == ScatterMode.MOE_FULL)
            and (residual_input_mode == ScatterMode.TP_ATTN_FULL)
            and (output_mode == ScatterMode.TP_ATTN_FULL)
        ):
            return CommunicateSummableTensorPairFn._scatter_hidden_states_moe

        raise NotImplementedError(
            f"{hidden_states_input_mode=} {residual_input_mode=} {output_mode=}"
        )
```
**EN:** This block defines `CommunicateSummableTensorPairFn.get_fn` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `NotImplementedError` and `context.is_same_group_size`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateSummableTensorPairFn.get_fn`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `NotImplementedError` 和 `context.is_same_group_size`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 1199-1208: Internal helper `CommunicateSummableTensorPairFn._trivial`
```python
    @staticmethod
    def _trivial(
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        context: CommunicateContext,
        **kwargs,
    ):
        return hidden_states, residual
```
**EN:** This block defines `CommunicateSummableTensorPairFn._trivial` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateSummableTensorPairFn._trivial`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 1209-1236: Internal helper `CommunicateSummableTensorPairFn._scatter_hidden_states`
```python
    @staticmethod
    def _scatter_hidden_states(
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        context: CommunicateContext,
        allow_reduce_scatter: bool = False,
    ):
        if get_tensor_model_parallel_world_size() == get_attention_dp_size():
            group = get_tp_group()
        else:
            group = get_attention_tp_group()
        hidden_states, global_hidden_states = (
            get_local_dp_buffer(group),
            hidden_states,
        )
        if should_use_dp_reduce_scatterv():
            get_tp_group().reduce_scatterv(
                global_hidden_states,
                output=hidden_states,
                sizes=get_dp_global_num_tokens(),
            )
        elif allow_reduce_scatter and forward_batch.dp_padding_mode.is_max_len():
            dp_reduce_scatter_tensor(hidden_states, global_hidden_states)
        else:
            dp_scatter(hidden_states, global_hidden_states, forward_batch)
        return hidden_states, residual
```
**EN:** This block defines `CommunicateSummableTensorPairFn._scatter_hidden_states` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `should_use_dp_reduce_scatterv`, `get_tensor_model_parallel_world_size`, `get_attention_dp_size`, `get_tp_group`, and `get_attention_tp_group`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `global_hidden_states`, and `group` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateSummableTensorPairFn._scatter_hidden_states`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `should_use_dp_reduce_scatterv`、`get_tensor_model_parallel_world_size`、`get_attention_dp_size`、`get_tp_group` 以及 `get_attention_tp_group`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`global_hidden_states` 以及 `group` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1237-1256: Internal helper `CommunicateSummableTensorPairFn._gather`
```python
    @staticmethod
    def _gather(
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        context: CommunicateContext,
        **kwargs,
    ):
        hidden_states += residual
        residual = None
        hidden_states, local_hidden_states = (
            get_local_dp_buffer(get_attention_tp_group()),
            hidden_states,
        )
        attn_tp_all_gather_into_tensor(
            hidden_states,
            local_hidden_states,
        )
        return hidden_states, residual
```
**EN:** This block defines `CommunicateSummableTensorPairFn._gather` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `attn_tp_all_gather_into_tensor`, `get_local_dp_buffer`, and `get_attention_tp_group`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `residual`, and `local_hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateSummableTensorPairFn._gather`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `attn_tp_all_gather_into_tensor`、`get_local_dp_buffer` 以及 `get_attention_tp_group`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`residual` 以及 `local_hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1257-1268: Internal helper `CommunicateSummableTensorPairFn._scatter`
```python
    @staticmethod
    def _scatter(
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        context: CommunicateContext,
    ):
        assert residual is None, "not yet handled residual!=None"
        tensor_list = list(hidden_states.tensor_split(context.attn_tp_size))
        hidden_states = tensor_list[context.attn_tp_rank]
        return hidden_states, residual
```
**EN:** This block defines `CommunicateSummableTensorPairFn._scatter` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `list` and `hidden_states.tensor_split`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `tensor_list` and `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateSummableTensorPairFn._scatter`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `list` 和 `hidden_states.tensor_split`，说明该流程会编排底层辅助函数或计算内核。 像 `tensor_list` 和 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1269-1314: Internal helper `CommunicateSummableTensorPairFn._scatter_hidden_states_moe`
```python
    @staticmethod
    def _scatter_hidden_states_moe(
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        context: CommunicateContext,
        **kwargs,
    ):
        """Scatter MoE output back to TP_ATTN_FULL after MOE_FULL computation.

        After moe_tensor_model_parallel_all_reduce (which runs unconditionally since
        use_reduce_scatter=False for this path), all ranks in the moe_cp group hold the
        full MoE result for all cp_per_moe token chunks. We simply slice out this rank's
        CP-local portion.

        If DP>1, further scatter back to the local DP slice.
        """
        # Only scatter back during prefill; decode was never allgathered so no-op.
        # Safe w.r.t. empty tensors: same reasoning as _gather_hidden_states_and_residual_moe
        # — CP extend always has non-zero tokens per rank, and decode skips this path.
        moe_cp_size = get_moe_cp_size()
        if (
            moe_cp_size > 1
            and forward_batch.forward_mode.is_context_parallel_extend()
            and forward_batch.attn_cp_metadata is not None
        ):
            moe_cp_rank = get_moe_cp_rank()
            # The allgather was padded to max_tokens_per_rank (equal chunks).
            # Extract this rank's actual (non-padded) tokens from its chunk.
            per_rank_tokens = forward_batch.attn_cp_metadata.per_rank_actual_token
            max_tokens_per_rank = max(per_rank_tokens)
            actual_local_tokens = per_rank_tokens[moe_cp_rank]
            hidden_states = hidden_states.narrow(
                0, moe_cp_rank * max_tokens_per_rank, actual_local_tokens
            ).contiguous()

        # DP scatter (if DP attention is enabled)
        if context.attn_dp_size > 1:
            hidden_states_output, global_hidden_states = (
                get_local_dp_buffer(),
                hidden_states,
            )
            dp_scatter(hidden_states_output, global_hidden_states, forward_batch)
            hidden_states = hidden_states_output

        return hidden_states, residual
```
**EN:** This block defines `CommunicateSummableTensorPairFn._scatter_hidden_states_moe` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `get_moe_cp_size`, `forward_batch.forward_mode.is_context_parallel_extend`, `get_moe_cp_rank`, `max`, and `hidden_states.narrow.contiguous`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `moe_cp_size`, `moe_cp_rank`, `per_rank_tokens`, `max_tokens_per_rank`, and `actual_local_tokens` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `CommunicateSummableTensorPairFn._scatter_hidden_states_moe`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `get_moe_cp_size`、`forward_batch.forward_mode.is_context_parallel_extend`、`get_moe_cp_rank`、`max` 以及 `hidden_states.narrow.contiguous`，说明该流程会编排底层辅助函数或计算内核。 像 `moe_cp_size`、`moe_cp_rank`、`per_rank_tokens`、`max_tokens_per_rank` 以及 `actual_local_tokens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_fused_rmsnorm_fp8_per_token_quant`, `apply_flashinfer_allreduce_fusion`, `apply_aiter_all_reduce_fusion`, `ScatterMode`, and `AttentionInputs`. / **主要符号**：核心入口包括 `_fused_rmsnorm_fp8_per_token_quant`、`apply_flashinfer_allreduce_fusion`、`apply_aiter_all_reduce_fusion`、`ScatterMode` 以及 `AttentionInputs`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `logging`, `contextlib.contextmanager`, `dataclasses.dataclass`, `enum.Enum`, `enum.auto`, `functools.partial`, `typing.Callable`, `typing.Dict`, `typing.List`, and `typing.Optional` / **标准库**：`logging`、`contextlib.contextmanager`、`dataclasses.dataclass`、`enum.Enum`、`enum.auto`、`functools.partial`、`typing.Callable`、`typing.Dict`、`typing.List` 以及 `typing.Optional`
- **Third-party**: `torch`, `aiter.ops.rmsnorm.add_rmsnorm_quant`, `aiter.ops.rmsnorm.rmsnorm_quant`, and `aiter.ops.triton.fused_fp8_quant.fused_rms_fp8_group_quant` / **第三方依赖**：`torch`、`aiter.ops.rmsnorm.add_rmsnorm_quant`、`aiter.ops.rmsnorm.rmsnorm_quant` 以及 `aiter.ops.triton.fused_fp8_quant.fused_rms_fp8_group_quant`
- **Internal SGLang modules**: `sglang.srt.distributed.attention_tensor_model_parallel_all_reduce`, `sglang.srt.distributed.attention_tensor_model_parallel_quant_all_reduce`, `sglang.srt.distributed.get_tensor_model_parallel_rank`, `sglang.srt.distributed.get_tensor_model_parallel_world_size`, `sglang.srt.distributed.get_tp_group`, `sglang.srt.distributed.moe_tensor_model_parallel_all_reduce`, `sglang.srt.distributed.tensor_model_parallel_all_reduce`, `sglang.srt.distributed.device_communicators.pynccl_allocator.use_symmetric_memory`, `sglang.srt.environ.envs`, `sglang.srt.layers.attention.nsa.utils.is_nsa_enable_prefill_cp`, `sglang.srt.layers.attention.nsa.utils.nsa_use_prefill_cp`, and `sglang.srt.layers.dp_attention.attn_tp_all_gather_into_tensor` / **SGLang 内部模块**：`sglang.srt.distributed.attention_tensor_model_parallel_all_reduce`、`sglang.srt.distributed.attention_tensor_model_parallel_quant_all_reduce`、`sglang.srt.distributed.get_tensor_model_parallel_rank`、`sglang.srt.distributed.get_tensor_model_parallel_world_size`、`sglang.srt.distributed.get_tp_group`、`sglang.srt.distributed.moe_tensor_model_parallel_all_reduce`、`sglang.srt.distributed.tensor_model_parallel_all_reduce`、`sglang.srt.distributed.device_communicators.pynccl_allocator.use_symmetric_memory`、`sglang.srt.environ.envs`、`sglang.srt.layers.attention.nsa.utils.is_nsa_enable_prefill_cp`、`sglang.srt.layers.attention.nsa.utils.nsa_use_prefill_cp` 以及 `sglang.srt.layers.dp_attention.attn_tp_all_gather_into_tensor`
