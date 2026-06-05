# layernorm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/layernorm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with linear projections and GEMM backend integration for the SGLang SRT runtime. It exposes symbols such as `_forward_with_allreduce_fusion`, `RMSNorm`, `LayerNorm`, and `GemmaRMSNorm` and connects them to backend-specific paths such as `CUDA`, `FlashInfer`, `NPU`, and `XPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合线性投影与 GEMM 后端集成。它提供了 `_forward_with_allreduce_fusion`、`RMSNorm`、`LayerNorm` 以及 `GemmaRMSNorm` 等符号，并把这些符号连接到 `CUDA`、`FlashInfer`、`NPU` 以及 `XPU` 等后端专用路径。

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

### Lines 14-115: Imports, conditional backend setup, and runtime guards
```python
"""Fused operators for normalization layers."""

import logging
from typing import Optional, Tuple, Union

import torch
import torch.nn as nn
import torch.nn.functional as F

from sglang.srt.batch_invariant_ops import (
    is_batch_invariant_mode_enabled,
    rms_norm_batch_invariant,
)
from sglang.srt.environ import envs
from sglang.srt.layers.utils import MultiPlatformOp
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import (
    cpu_has_amx_support,
    get_bool_env_var,
    is_cpu,
    is_cuda,
    is_flashinfer_available,
    is_hip,
    is_musa,
    is_npu,
    is_xpu,
)

_is_cuda = is_cuda()
_is_flashinfer_available = is_flashinfer_available()
_is_hip = is_hip()
_is_musa = is_musa()
_is_npu = is_npu()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
_is_cpu_amx_available = cpu_has_amx_support()
_is_cpu = is_cpu()
_is_xpu = is_xpu()
_flashinfer_layernorm_available = False

if _is_cuda or _is_xpu or _is_musa:
    if _is_flashinfer_available:
        try:
            from flashinfer.norm import layernorm

            _flashinfer_layernorm_available = True
        except (ImportError, AttributeError):
            _flashinfer_layernorm_available = False
    else:
        _flashinfer_layernorm_available = False

    from sgl_kernel import (
        fused_add_rmsnorm,
        gemma_fused_add_rmsnorm,
        gemma_rmsnorm,
        rmsnorm,
    )
_has_aiter_layer_norm = False
_has_vllm_rms_norm = False
if _use_aiter:
    from aiter import layernorm2d_fwd as layer_norm
    from aiter import rmsnorm2d_fwd as rms_norm
    from aiter import rmsnorm2d_fwd_with_add as fused_add_rms_norm

    _has_aiter_layer_norm = True  # aiter provides the layer_norm functions
    _has_vllm_rms_norm = True  # aiter provides the rms_norm functions
elif _is_hip:
    try:
        from vllm._custom_ops import fused_add_rms_norm, rms_norm

        _has_vllm_rms_norm = True
    except ImportError:
        # Fallback: vllm not available, will use forward_native
        _has_vllm_rms_norm = False

if _is_cuda:
    # HF-semantics RMSNorm kernel (JIT-compiled).  Used when `cast_x_before_out_mul=True`
    # (the transformers backend path) to produce outputs that are numerically identical
    # to HuggingFace `LlamaRMSNorm`: the cast from fp32 to the activation dtype happens
    # BEFORE the weight multiply, so the multiply is done in the narrow dtype.
    _jit_rmsnorm_hf_available = False
    try:
        from sglang.jit_kernel.rmsnorm_hf import (
            is_supported_rmsnorm_hf_hidden_size,
        )
        from sglang.jit_kernel.rmsnorm_hf import rmsnorm_hf as _jit_rmsnorm_hf

        _jit_rmsnorm_hf_available = True
    except ImportError:

        def is_supported_rmsnorm_hf_hidden_size(d: int) -> bool:
            return False

        _jit_rmsnorm_hf = None


logger = logging.getLogger(__name__)

if _is_npu:
    import torch_npu
    from sgl_kernel_npu.norm.add_rmsnorm_bias import add_gemma_rms_norm
```
**EN:** This section prepares the module namespace. It imports `logging`, `typing.Optional`, `typing.Tuple`, `typing.Union`, `torch`, and `torch.nn`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_cuda`, `_is_flashinfer_available`, `_is_hip`, `_is_musa`, and `_is_npu` capture configuration, cached handles, or feature flags. Control structures like `If` and `Try` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`typing.Optional`、`typing.Tuple`、`typing.Union`、`torch` 以及 `torch.nn`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_cuda`、`_is_flashinfer_available`、`_is_hip`、`_is_musa` 以及 `_is_npu` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 和 `Try` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 116-174: Internal helper `_forward_with_allreduce_fusion`
```python
def _forward_with_allreduce_fusion(
    norm_module,
    x: torch.Tensor,
    residual: Optional[torch.Tensor],
    post_residual_addition: Optional[torch.Tensor],
    weight: torch.Tensor,
    use_attn_tp_group: bool = True,
) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
    """Shared allreduce-fused RMSNorm logic usable by any norm."""
    if residual is not None:
        from sglang.srt.distributed import (
            get_attn_tensor_model_parallel_world_size,
            get_moe_expert_parallel_world_size,
            get_moe_tensor_parallel_world_size,
            tensor_model_parallel_all_reduce,
            tensor_model_parallel_fused_allreduce_rmsnorm,
        )
        from sglang.srt.layers.flashinfer_comm_fusion import (
            flashinfer_allreduce_residual_rmsnorm,
        )

        if use_attn_tp_group:
            world_size = get_attn_tensor_model_parallel_world_size()
        else:
            if get_moe_expert_parallel_world_size() > 1:
                world_size = get_moe_expert_parallel_world_size()
            else:
                world_size = get_moe_tensor_parallel_world_size()

        if world_size > 1:
            if post_residual_addition is not None:
                residual = residual + post_residual_addition

            # Prefer AITER fused AR+RMSNorm when enabled on AMD.
            if _use_aiter:
                fused_result = tensor_model_parallel_fused_allreduce_rmsnorm(
                    x, residual, weight, norm_module.variance_epsilon
                )
                if fused_result is not None:
                    return fused_result
            else:
                fused_result = flashinfer_allreduce_residual_rmsnorm(
                    input_tensor=x,
                    residual=residual,
                    weight=weight,
                    eps=norm_module.variance_epsilon,
                    use_attn_tp_group=use_attn_tp_group,
                )
                if fused_result[0] is not None:
                    return fused_result

            # For AITER route, preserve correctness when fused path is unavailable.
            if _use_aiter and get_global_server_args().enable_aiter_allreduce_fusion:
                x = tensor_model_parallel_all_reduce(x)
                return norm_module.forward(x, residual, None)

    return norm_module.forward(x, residual, post_residual_addition)
```
**EN:** This block defines `_forward_with_allreduce_fusion` and contains the main logic for this step. It mainly invokes `norm_module.forward`, `get_attn_tensor_model_parallel_world_size`, `get_moe_expert_parallel_world_size`, `get_moe_tensor_parallel_world_size`, and `tensor_model_parallel_fused_allreduce_rmsnorm`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `world_size`, `residual`, `fused_result`, and `x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_forward_with_allreduce_fusion`，并承载这一阶段的核心逻辑。 它主要调用 `norm_module.forward`、`get_attn_tensor_model_parallel_world_size`、`get_moe_expert_parallel_world_size`、`get_moe_tensor_parallel_world_size` 以及 `tensor_model_parallel_fused_allreduce_rmsnorm`，说明该流程会编排底层辅助函数或计算内核。 像 `world_size`、`residual`、`fused_result` 以及 `x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 175-175: Class `RMSNorm` declaration and shared state
```python
class RMSNorm(MultiPlatformOp):
```
**EN:** This block introduces class `RMSNorm` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `RMSNorm`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。

### Lines 176-203: `RMSNorm` initialization and state setup
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
        var_hidden_size: Optional[int] = None,
        cast_x_before_out_mul: bool = False,
        fp32_residual: bool = False,
        has_weight: bool = True,
        weight_dtype: Optional = None,
        override_orig_dtype: Optional = None,
    ) -> None:
        super().__init__()
        self.has_weight = has_weight
        self.cast_x_before_out_mul = cast_x_before_out_mul
        self.fp32_residual = fp32_residual
        self.override_orig_dtype = override_orig_dtype
        if self.has_weight:
            self.weight = nn.Parameter(torch.ones(hidden_size, dtype=weight_dtype))
        else:
            self.weight = torch.ones(hidden_size, dtype=weight_dtype)
        self.variance_epsilon = eps
        self.hidden_size = hidden_size
        self.variance_size_override = (
            None if var_hidden_size == hidden_size else var_hidden_size
        )
        if _use_aiter:
            self._forward_method = self.forward_aiter
```
**EN:** This block defines `RMSNorm.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `nn.Parameter`, and `torch.ones`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.has_weight`, `self.cast_x_before_out_mul`, `self.fp32_residual`, `self.override_orig_dtype`, and `self.variance_epsilon` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RMSNorm.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`nn.Parameter` 以及 `torch.ones`，说明该流程会编排底层辅助函数或计算内核。 像 `self.has_weight`、`self.cast_x_before_out_mul`、`self.fp32_residual`、`self.override_orig_dtype` 以及 `self.variance_epsilon` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 204-265: `RMSNorm.forward_cuda` CUDA execution path
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if x.numel() == 0:
            if residual is not None:
                if post_residual_addition is not None:
                    residual = residual + post_residual_addition
                return x, residual
            return x
        # sgl_kernel rmsnorm requires 2D input; reshape higher-rank tensors
        needs_reshape = x.dim() != 2 and residual is None
        if needs_reshape:
            original_shape = x.shape
            x = x.contiguous().reshape(-1, original_shape[-1])
        if self.variance_size_override is not None:
            return self.forward_native(x, residual, post_residual_addition)
        if is_batch_invariant_mode_enabled():
            if (
                residual is not None
                or self.cast_x_before_out_mul
                or get_global_server_args().rl_on_policy_target == "fsdp"
            ):
                return self.forward_native(x, residual, post_residual_addition)
            return rms_norm_batch_invariant(
                x,
                self.weight.data,
                self.variance_epsilon,
            )
        if self.cast_x_before_out_mul and residual is None:
            # Use HF-semantics kernel (cast to dtype before weight multiply).
            if (
                _jit_rmsnorm_hf_available
                and x.dtype in (torch.float16, torch.bfloat16)
                and self.weight.data.dtype == x.dtype
                and is_supported_rmsnorm_hf_hidden_size(x.shape[-1])
            ):
                out = _jit_rmsnorm_hf(
                    x.contiguous(), self.weight.data, self.variance_epsilon
                )
            else:
                # Fallback: pure-Python HF semantics (already implemented in forward_native).
                out = self.forward_native(x, None, None)
            if needs_reshape:
                out = out.reshape(original_shape)
            return out
        if residual is not None:
            # TODO: Ideally we want to have (hidden_states+residual)+post_residual_addition.
            # but right now we can only have hidden_states+(residual+post_residual_addition).
            # (hidden_states+residual)+post_residual_addition != hidden_states+(residual+post_residual_addition),
            # we probably need to add another parameter to fused_add_rmsnorm
            if post_residual_addition is not None:
                residual = residual + post_residual_addition
            fused_add_rmsnorm(x, residual, self.weight.data, self.variance_epsilon)
            return x, residual
        out = rmsnorm(x, self.weight.data, self.variance_epsilon)
        if needs_reshape:
            out = out.reshape(original_shape)
        return out
```
**EN:** This block defines `RMSNorm.forward_cuda` and contains the main logic for this step. It mainly invokes `is_batch_invariant_mode_enabled`, `rmsnorm`, `x.numel`, `x.contiguous.reshape`, and `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `needs_reshape`, `out`, `original_shape`, `x`, and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNorm.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `is_batch_invariant_mode_enabled`、`rmsnorm`、`x.numel`、`x.contiguous.reshape` 以及 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 像 `needs_reshape`、`out`、`original_shape`、`x` 以及 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 266-280: `RMSNorm.forward_npu` NPU execution path
```python
    def forward_npu(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if residual is not None:
            if post_residual_addition is not None:
                residual = residual + post_residual_addition
            out, _, residual_out = torch_npu.npu_add_rms_norm(
                residual, x, self.weight.data, self.variance_epsilon
            )
            return out, residual_out
        return torch_npu.npu_rms_norm(x, self.weight.data, self.variance_epsilon)[0]
```
**EN:** This block defines `RMSNorm.forward_npu` and contains the main logic for this step. It mainly invokes `torch_npu.npu_add_rms_norm` and `torch_npu.npu_rms_norm`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `out`, `_`, `residual_out`, and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNorm.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `torch_npu.npu_add_rms_norm` 和 `torch_npu.npu_rms_norm`，说明该流程会编排底层辅助函数或计算内核。 像 `out`、`_`、`residual_out` 以及 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 281-314: `RMSNorm.forward_aiter` AITER execution path
```python
    def forward_aiter(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        # Aiter's RMSNorm kernels expect 2D contiguous inputs. Keep the
        # already-safe layout as a zero-copy path, and only normalize strided or
        # higher-rank views such as Q/K slices from packed QKV projections.
        needs_reshape = x.dim() != 2 and residual is None
        if needs_reshape:
            original_shape = x.shape
            x = x.contiguous().reshape(-1, original_shape[-1])
        elif not x.is_contiguous():
            x = x.contiguous()
        if residual is not None:
            residual_out = torch.empty_like(x)
            output = torch.empty_like(x)
            if post_residual_addition is not None:
                residual = residual + post_residual_addition
            fused_add_rms_norm(
                output,
                x,
                residual,
                residual_out,
                self.weight.data,
                self.variance_epsilon,
            )
            return output, residual_out
        output = rms_norm(x, self.weight.data, self.variance_epsilon)
        if needs_reshape:
            output = output.reshape(original_shape)
        return output
```
**EN:** This block defines `RMSNorm.forward_aiter` and contains the main logic for this step. It mainly invokes `rms_norm`, `x.contiguous.reshape`, `torch.empty_like`, `fused_add_rms_norm`, and `output.reshape`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `needs_reshape`, `output`, `original_shape`, `x`, and `residual_out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNorm.forward_aiter`，并承载这一阶段的核心逻辑。 它主要调用 `rms_norm`、`x.contiguous.reshape`、`torch.empty_like`、`fused_add_rms_norm` 以及 `output.reshape`，说明该流程会编排底层辅助函数或计算内核。 像 `needs_reshape`、`output`、`original_shape`、`x` 以及 `residual_out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 315-340: `RMSNorm.forward_hip` HIP execution path
```python
    def forward_hip(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        # Fallback to native implementation if vllm is not available
        if not _has_vllm_rms_norm:
            return self.forward_native(x, residual, post_residual_addition)

        if not x.is_contiguous():
            # NOTE: Remove this if aiter kernel supports discontinuous input
            x = x.contiguous()
        if residual is not None:
            out = torch.empty_like(x)
            residual_out = torch.empty_like(x)
            if post_residual_addition is not None:
                residual = residual + post_residual_addition
            fused_add_rms_norm(
                out, x, residual_out, residual, self.weight.data, self.variance_epsilon
            )
            return out, residual_out
        out = torch.empty_like(x)
        rms_norm(out, x, self.weight.data, self.variance_epsilon)
        return out
```
**EN:** This block defines `RMSNorm.forward_hip` and contains the main logic for this step. It mainly invokes `torch.empty_like`, `rms_norm`, `self.forward_native`, `x.is_contiguous`, and `x.contiguous`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `out`, `x`, `residual_out`, and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNorm.forward_hip`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty_like`、`rms_norm`、`self.forward_native`、`x.is_contiguous` 以及 `x.contiguous`，说明该流程会编排底层辅助函数或计算内核。 像 `out`、`x`、`residual_out` 以及 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 341-360: `RMSNorm.forward_musa` MUSA execution path
```python
    def forward_musa(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if not x.is_contiguous():
            x = x.contiguous()

        if residual is not None:
            if post_residual_addition is not None:
                residual = residual + post_residual_addition
            fused_add_rmsnorm(x, residual, self.weight.data, self.variance_epsilon)
            return x, residual

        out = nn.functional.rms_norm(
            x, (self.hidden_size,), self.weight.data, self.variance_epsilon
        )
        return out
```
**EN:** This block defines `RMSNorm.forward_musa` and contains the main logic for this step. It mainly invokes `nn.functional.rms_norm`, `x.is_contiguous`, `x.contiguous`, and `fused_add_rmsnorm`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `out`, `x`, and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNorm.forward_musa`，并承载这一阶段的核心逻辑。 它主要调用 `nn.functional.rms_norm`、`x.is_contiguous`、`x.contiguous` 以及 `fused_add_rmsnorm`，说明该流程会编排底层辅助函数或计算内核。 像 `out`、`x` 以及 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 361-410: `RMSNorm.forward_native` NATIVE execution path
```python
    def forward_native(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if not x.is_contiguous():
            x = x.contiguous()
        orig_dtype = self.override_orig_dtype or x.dtype
        x = x.to(torch.float32)
        if residual is not None:
            x = x + residual.to(torch.float32)
            if post_residual_addition is not None:
                x = x + post_residual_addition.to(torch.float32)
            if self.fp32_residual:
                residual = x.clone()
            else:
                residual = x.to(orig_dtype)

        hidden_size = x.shape[-1]
        if hidden_size != self.hidden_size:
            raise ValueError(
                "Expected hidden_size to be "
                f"{self.hidden_size}, but found: {hidden_size}"
            )

        if self.variance_size_override is None:
            x_var = x
        else:
            if hidden_size < self.variance_size_override:
                raise ValueError(
                    "Expected hidden_size to be at least "
                    f"{self.variance_size_override}, but found: {hidden_size}"
                )

            x_var = x[..., : self.variance_size_override]

        variance = x_var.pow(2).mean(dim=-1, keepdim=True)
        x = x * torch.rsqrt(variance + self.variance_epsilon)

        if self.cast_x_before_out_mul:
            x = self.weight * x.to(orig_dtype)
        else:
            x = (x * self.weight).to(orig_dtype)

        if residual is None:
            return x
        else:
            return x, residual
```
**EN:** This block defines `RMSNorm.forward_native` and contains the main logic for this step. It mainly invokes `x.to`, `x_var.pow.mean`, `x.is_contiguous`, `x.contiguous`, and `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `orig_dtype`, `x`, `hidden_size`, `variance`, and `x_var` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNorm.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `x.to`、`x_var.pow.mean`、`x.is_contiguous`、`x.contiguous` 以及 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `orig_dtype`、`x`、`hidden_size`、`variance` 以及 `x_var` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 411-430: `RMSNorm.forward_cpu` CPU execution path
```python
    def forward_cpu(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if _is_cpu_amx_available:
            if residual is not None:
                if post_residual_addition is not None:
                    residual = residual + post_residual_addition
                torch.ops.sgl_kernel.fused_add_rmsnorm_cpu(
                    x, residual, self.weight.data, self.variance_epsilon
                )
                return x, residual
            return torch.ops.sgl_kernel.rmsnorm_cpu(
                x, self.weight.data, self.variance_epsilon
            )
        else:
            return self.forward_native(x, residual, post_residual_addition)
```
**EN:** This block defines `RMSNorm.forward_cpu` and contains the main logic for this step. It mainly invokes `torch.ops.sgl_kernel.rmsnorm_cpu`, `self.forward_native`, and `torch.ops.sgl_kernel.fused_add_rmsnorm_cpu`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNorm.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.ops.sgl_kernel.rmsnorm_cpu`、`self.forward_native` 以及 `torch.ops.sgl_kernel.fused_add_rmsnorm_cpu`，说明该流程会编排底层辅助函数或计算内核。 像 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 431-457: `RMSNorm.forward_xpu` XPU execution path
```python
    def forward_xpu(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if self.variance_size_override is not None:
            return self.forward_native(x, residual, post_residual_addition)
        if is_batch_invariant_mode_enabled():
            if (
                residual is not None
                or get_global_server_args().rl_on_policy_target == "fsdp"
            ):
                return self.forward_native(x, residual, post_residual_addition)
            return rms_norm_batch_invariant(
                x,
                self.weight.data,
                self.variance_epsilon,
            )
        if residual is not None:
            if post_residual_addition is not None:
                residual = residual + post_residual_addition
            fused_add_rmsnorm(x, residual, self.weight.data, self.variance_epsilon)
            return x, residual
        out = rmsnorm(x, self.weight.data, self.variance_epsilon)
        return out
```
**EN:** This block defines `RMSNorm.forward_xpu` and contains the main logic for this step. It mainly invokes `is_batch_invariant_mode_enabled`, `rmsnorm`, `self.forward_native`, `rms_norm_batch_invariant`, and `fused_add_rmsnorm`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `out` and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNorm.forward_xpu`，并承载这一阶段的核心逻辑。 它主要调用 `is_batch_invariant_mode_enabled`、`rmsnorm`、`self.forward_native`、`rms_norm_batch_invariant` 以及 `fused_add_rmsnorm`，说明该流程会编排底层辅助函数或计算内核。 像 `out` 和 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 458-470: `RMSNorm.forward_with_allreduce_fusion` WITH_ALLREDUCE_FUSION execution path
```python
    def forward_with_allreduce_fusion(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
        use_attn_tp_group: bool = True,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        """Forward with allreduce fusion, prioritizing flashinfer fused operations."""
        return _forward_with_allreduce_fusion(
            self, x, residual, post_residual_addition, self.weight, use_attn_tp_group
        )
```
**EN:** This block defines `RMSNorm.forward_with_allreduce_fusion` and contains the main logic for this step. It mainly invokes `_forward_with_allreduce_fusion`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNorm.forward_with_allreduce_fusion`，并承载这一阶段的核心逻辑。 它主要调用 `_forward_with_allreduce_fusion`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 471-471: Class `LayerNorm` declaration and shared state
```python
class LayerNorm(MultiPlatformOp):
```
**EN:** This block introduces class `LayerNorm` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `LayerNorm`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。

### Lines 472-489: `LayerNorm` initialization and state setup
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
        elementwise_affine: bool = True,
        bias: bool = True,
        dtype: torch.dtype = torch.float32,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.variance_epsilon = eps
        self.elementwise_affine = elementwise_affine
        self.use_bias = bias
        self.dtype = dtype

        self.bias = nn.Parameter(torch.zeros(hidden_size, dtype=self.dtype))
        self.weight = nn.Parameter(torch.ones(hidden_size, dtype=self.dtype))
```
**EN:** This block defines `LayerNorm.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `nn.Parameter`, `torch.zeros`, and `torch.ones`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.hidden_size`, `self.variance_epsilon`, `self.elementwise_affine`, `self.use_bias`, and `self.dtype` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `LayerNorm.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`nn.Parameter`、`torch.zeros` 以及 `torch.ones`，说明该流程会编排底层辅助函数或计算内核。 像 `self.hidden_size`、`self.variance_epsilon`、`self.elementwise_affine`、`self.use_bias` 以及 `self.dtype` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 490-502: `LayerNorm.forward_cuda` CUDA execution path
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
    ) -> torch.Tensor:
        if (
            _flashinfer_layernorm_available
            and x.dtype == torch.bfloat16
            and self.dtype == torch.float32
        ):
            return layernorm(x, self.weight, self.bias, self.variance_epsilon)
        else:
            return self.forward_native(x)
```
**EN:** This block defines `LayerNorm.forward_cuda` and contains the main logic for this step. It mainly invokes `layernorm` and `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerNorm.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `layernorm` 和 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 503-518: `LayerNorm.forward_native` NATIVE execution path
```python
    def forward_native(
        self,
        x: torch.Tensor,
    ) -> torch.Tensor:
        weight = self.weight if self.elementwise_affine else None
        bias = self.bias if self.use_bias else None
        orig_dtype = x.dtype
        x = x.to(self.dtype)
        return F.layer_norm(
            x,
            (self.hidden_size,),
            weight=weight,
            bias=bias,
            eps=self.variance_epsilon,
        ).to(orig_dtype)
```
**EN:** This block defines `LayerNorm.forward_native` and contains the main logic for this step. It mainly invokes `x.to`, `F.layer_norm.to`, and `F.layer_norm`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `weight`, `bias`, `orig_dtype`, and `x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerNorm.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `x.to`、`F.layer_norm.to` 以及 `F.layer_norm`，说明该流程会编排底层辅助函数或计算内核。 像 `weight`、`bias`、`orig_dtype` 以及 `x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 519-535: `LayerNorm.forward_hip` HIP execution path
```python
    def forward_hip(
        self,
        x: torch.Tensor,
    ) -> torch.Tensor:
        if (
            _has_aiter_layer_norm
            and x.dtype in (torch.bfloat16, torch.float16)
            and x.dtype == self.dtype
        ):
            orig_shape = x.shape
            x = x.reshape(-1, self.hidden_size)
            return layer_norm(x, self.weight, self.bias, self.variance_epsilon).view(
                orig_shape
            )
        else:
            return self.forward_native(x)
```
**EN:** This block defines `LayerNorm.forward_hip` and contains the main logic for this step. It mainly invokes `x.reshape`, `layer_norm.view`, `self.forward_native`, and `layer_norm`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `orig_shape` and `x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerNorm.forward_hip`，并承载这一阶段的核心逻辑。 它主要调用 `x.reshape`、`layer_norm.view`、`self.forward_native` 以及 `layer_norm`，说明该流程会编排底层辅助函数或计算内核。 像 `orig_shape` 和 `x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 536-541: `LayerNorm.forward_npu` NPU execution path
```python
    def forward_npu(
        self,
        x: torch.Tensor,
    ) -> torch.Tensor:
        return self.forward_native(x)
```
**EN:** This block defines `LayerNorm.forward_npu` and contains the main logic for this step. It mainly invokes `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerNorm.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 542-554: `LayerNorm.forward_cpu` CPU execution path
```python
    def forward_cpu(
        self,
        x: torch.Tensor,
    ) -> torch.Tensor:
        if _is_cpu_amx_available:
            bias_data = self.bias.data if self.use_bias else None
            return torch.ops.sgl_kernel.layernorm_cpu(
                x, self.weight.data, bias_data, self.variance_epsilon
            )
        else:
            return self.forward_native(x)
```
**EN:** This block defines `LayerNorm.forward_cpu` and contains the main logic for this step. It mainly invokes `torch.ops.sgl_kernel.layernorm_cpu` and `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bias_data` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `LayerNorm.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.ops.sgl_kernel.layernorm_cpu` 和 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 像 `bias_data` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 555-555: Class `GemmaRMSNorm` declaration and shared state
```python
class GemmaRMSNorm(MultiPlatformOp):
```
**EN:** This block introduces class `GemmaRMSNorm` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `GemmaRMSNorm`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。

### Lines 556-569: `GemmaRMSNorm` initialization and state setup
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
    ) -> None:
        super().__init__()
        self.weight = nn.Parameter(torch.zeros(hidden_size))
        self.variance_epsilon = eps
        self.register_buffer("gemma_weight", self.weight.data + 1.0, persistent=False)
        # (Chen-0210) Gemma weight = standard_weight + 1. Precompute once.
        # If TRTLLM allreduce fusion ever provides gemma-style norm
        # natively, this can be removed.
        self.weight.weight_loader = self._weight_loader
```
**EN:** This block defines `GemmaRMSNorm.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `nn.Parameter`, `self.register_buffer`, and `torch.zeros`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.weight` and `self.variance_epsilon` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `GemmaRMSNorm.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`nn.Parameter`、`self.register_buffer` 以及 `torch.zeros`，说明该流程会编排底层辅助函数或计算内核。 像 `self.weight` 和 `self.variance_epsilon` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 570-574: Internal helper `GemmaRMSNorm._weight_loader`
```python
    def _weight_loader(self, param: torch.Tensor, loaded_weight: torch.Tensor) -> None:
        assert param.size() == loaded_weight.size()
        param.data.copy_(loaded_weight)
        self.gemma_weight = param.data + 1.0
```
**EN:** This block defines `GemmaRMSNorm._weight_loader` and contains the main logic for this step. It mainly invokes `param.data.copy_`, `param.size`, and `loaded_weight.size`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.gemma_weight` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `GemmaRMSNorm._weight_loader`，并承载这一阶段的核心逻辑。 它主要调用 `param.data.copy_`、`param.size` 以及 `loaded_weight.size`，说明该流程会编排底层辅助函数或计算内核。 像 `self.gemma_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 575-596: Internal helper `GemmaRMSNorm._forward_impl`
```python
    def _forward_impl(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        needs_reshape = x.dim() != 2 and residual is None
        if needs_reshape:
            original_shape = x.shape
            x = x.contiguous().reshape(-1, original_shape[-1])
        if residual is not None:
            if post_residual_addition is not None:
                residual = residual + post_residual_addition
            gemma_fused_add_rmsnorm(
                x, residual, self.weight.data, self.variance_epsilon
            )
            return x, residual
        out = gemma_rmsnorm(x, self.weight.data, self.variance_epsilon)
        if needs_reshape:
            out = out.reshape(original_shape)
        return out
```
**EN:** This block defines `GemmaRMSNorm._forward_impl` and contains the main logic for this step. It mainly invokes `gemma_rmsnorm`, `x.contiguous.reshape`, `gemma_fused_add_rmsnorm`, `out.reshape`, and `x.dim`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `needs_reshape`, `out`, `original_shape`, `x`, and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GemmaRMSNorm._forward_impl`，并承载这一阶段的核心逻辑。 它主要调用 `gemma_rmsnorm`、`x.contiguous.reshape`、`gemma_fused_add_rmsnorm`、`out.reshape` 以及 `x.dim`，说明该流程会编排底层辅助函数或计算内核。 像 `needs_reshape`、`out`、`original_shape`、`x` 以及 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 597-616: `GemmaRMSNorm.forward_native` NATIVE execution path
```python
    def forward_native(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        orig_dtype = x.dtype
        if residual is not None:
            if post_residual_addition is not None:
                residual = residual + post_residual_addition
            x = x + residual
            residual = x

        x = x.float()
        variance = x.pow(2).mean(dim=-1, keepdim=True)
        x = x * torch.rsqrt(variance + self.variance_epsilon)
        x = x * (1.0 + self.weight.float())
        x = x.to(orig_dtype)
        return x if residual is None else (x, residual)
```
**EN:** This block defines `GemmaRMSNorm.forward_native` and contains the main logic for this step. It mainly invokes `x.float`, `x.pow.mean`, `x.to`, `torch.rsqrt`, and `x.pow`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `orig_dtype`, `x`, `variance`, and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GemmaRMSNorm.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `x.float`、`x.pow.mean`、`x.to`、`torch.rsqrt` 以及 `x.pow`，说明该流程会编排底层辅助函数或计算内核。 像 `orig_dtype`、`x`、`variance` 以及 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 617-624: `GemmaRMSNorm.forward_cuda` CUDA execution path
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        return self._forward_impl(x, residual, post_residual_addition)
```
**EN:** This block defines `GemmaRMSNorm.forward_cuda` and contains the main logic for this step. It mainly invokes `self._forward_impl`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GemmaRMSNorm.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self._forward_impl`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 625-665: `GemmaRMSNorm.forward_hip` HIP execution path
```python
    def forward_hip(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if not _has_vllm_rms_norm:
            return self.forward_native(x, residual, post_residual_addition)

        w = self.gemma_weight
        if _use_aiter:
            # aiter API: rms_norm(input, weight, eps) -> output
            #            fused_add_rms_norm(output, input, residual, residual_out, weight, eps)
            if residual is not None:
                output = torch.empty_like(x)
                residual_out = torch.empty_like(x)
                if post_residual_addition is not None:
                    residual = residual + post_residual_addition
                fused_add_rms_norm(
                    output, x, residual, residual_out, w, self.variance_epsilon
                )
                return output, residual_out
            return rms_norm(x, w, self.variance_epsilon)
        else:
            # vllm API: rms_norm(out, input, weight, eps) -> None (in-place)
            #           fused_add_rms_norm(out, input, residual_out, residual, weight, eps)
            if not x.is_contiguous():
                x = x.contiguous()
            if residual is not None:
                out = torch.empty_like(x)
                residual_out = torch.empty_like(x)
                if post_residual_addition is not None:
                    residual = residual + post_residual_addition
                fused_add_rms_norm(
                    out, x, residual_out, residual, w, self.variance_epsilon
                )
                return out, residual_out
            out = torch.empty_like(x)
            rms_norm(out, x, w, self.variance_epsilon)
            return out
```
**EN:** This block defines `GemmaRMSNorm.forward_hip` and contains the main logic for this step. It mainly invokes `self.forward_native`, `rms_norm`, `torch.empty_like`, `fused_add_rms_norm`, and `x.is_contiguous`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `w`, `out`, `output`, `residual_out`, and `x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GemmaRMSNorm.forward_hip`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`、`rms_norm`、`torch.empty_like`、`fused_add_rms_norm` 以及 `x.is_contiguous`，说明该流程会编排底层辅助函数或计算内核。 像 `w`、`out`、`output`、`residual_out` 以及 `x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 666-684: `GemmaRMSNorm.forward_cpu` CPU execution path
```python
    def forward_cpu(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if _is_cpu_amx_available:
            if residual is not None:
                if post_residual_addition is not None:
                    residual = residual + post_residual_addition
                torch.ops.sgl_kernel.gemma_fused_add_rmsnorm_cpu(
                    x, residual, self.weight.data, self.variance_epsilon
                )
                return x, residual
            return torch.ops.sgl_kernel.gemma_rmsnorm_cpu(
                x, self.weight.data, self.variance_epsilon
            )
        return self.forward_native(x, residual, post_residual_addition)
```
**EN:** This block defines `GemmaRMSNorm.forward_cpu` and contains the main logic for this step. It mainly invokes `self.forward_native`, `torch.ops.sgl_kernel.gemma_rmsnorm_cpu`, and `torch.ops.sgl_kernel.gemma_fused_add_rmsnorm_cpu`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GemmaRMSNorm.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`、`torch.ops.sgl_kernel.gemma_rmsnorm_cpu` 以及 `torch.ops.sgl_kernel.gemma_fused_add_rmsnorm_cpu`，说明该流程会编排底层辅助函数或计算内核。 像 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 685-703: `GemmaRMSNorm.forward_npu` NPU execution path
```python
    def forward_npu(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if envs.SGLANG_NPU_FORWARD_NATIVE_GEMMA_RMS_NORM.get():
            return self.forward_native(x, residual)
        if residual is not None:
            if post_residual_addition is not None:
                residual = residual + post_residual_addition
            norm_out, residual = add_gemma_rms_norm(
                x, self.weight, residual, self.variance_epsilon
            )
            return norm_out, residual

        x, _ = torch_npu.npu_gemma_rms_norm(x, self.weight, self.variance_epsilon)
        return x
```
**EN:** This block defines `GemmaRMSNorm.forward_npu` and contains the main logic for this step. It mainly invokes `envs.SGLANG_NPU_FORWARD_NATIVE_GEMMA_RMS_NORM.get`, `torch_npu.npu_gemma_rms_norm`, `self.forward_native`, and `add_gemma_rms_norm`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x`, `_`, `norm_out`, and `residual` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GemmaRMSNorm.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `envs.SGLANG_NPU_FORWARD_NATIVE_GEMMA_RMS_NORM.get`、`torch_npu.npu_gemma_rms_norm`、`self.forward_native` 以及 `add_gemma_rms_norm`，说明该流程会编排底层辅助函数或计算内核。 像 `x`、`_`、`norm_out` 以及 `residual` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 704-711: `GemmaRMSNorm.forward_xpu` XPU execution path
```python
    def forward_xpu(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        return self._forward_impl(x, residual, post_residual_addition)
```
**EN:** This block defines `GemmaRMSNorm.forward_xpu` and contains the main logic for this step. It mainly invokes `self._forward_impl`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GemmaRMSNorm.forward_xpu`，并承载这一阶段的核心逻辑。 它主要调用 `self._forward_impl`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 712-729: `GemmaRMSNorm.forward_with_allreduce_fusion` WITH_ALLREDUCE_FUSION execution path
```python
    def forward_with_allreduce_fusion(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
        post_residual_addition: Optional[torch.Tensor] = None,
        use_attn_tp_group: bool = True,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        """Forward with allreduce fusion; uses 1 + weight for fused kernels."""
        return _forward_with_allreduce_fusion(
            self,
            x,
            residual,
            post_residual_addition,
            self.gemma_weight,
            use_attn_tp_group=True,
        )
```
**EN:** This block defines `GemmaRMSNorm.forward_with_allreduce_fusion` and contains the main logic for this step. It mainly invokes `_forward_with_allreduce_fusion`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GemmaRMSNorm.forward_with_allreduce_fusion`，并承载这一阶段的核心逻辑。 它主要调用 `_forward_with_allreduce_fusion`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 730-730: Class `Gemma3RMSNorm` declaration and shared state
```python
class Gemma3RMSNorm(MultiPlatformOp):
```
**EN:** This block introduces class `Gemma3RMSNorm` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `Gemma3RMSNorm`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。

### Lines 731-736: `Gemma3RMSNorm` initialization and state setup
```python
    def __init__(self, dim: int, eps: float = 1e-6):
        super().__init__()
        self.eps = eps
        self.weight = nn.Parameter(torch.zeros(dim))
        # Re-dispatch
```
**EN:** This block defines `Gemma3RMSNorm.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `nn.Parameter`, and `torch.zeros`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.eps` and `self.weight` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Gemma3RMSNorm.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`nn.Parameter` 以及 `torch.zeros`，说明该流程会编排底层辅助函数或计算内核。 像 `self.eps` 和 `self.weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 737-739: Internal helper `Gemma3RMSNorm._norm`
```python
    def _norm(self, x):
        return x * torch.rsqrt(x.pow(2).mean(-1, keepdim=True) + self.eps)
```
**EN:** This block defines `Gemma3RMSNorm._norm` and contains the main logic for this step. It mainly invokes `torch.rsqrt`, `x.pow.mean`, and `x.pow`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma3RMSNorm._norm`，并承载这一阶段的核心逻辑。 它主要调用 `torch.rsqrt`、`x.pow.mean` 以及 `x.pow`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 740-746: `Gemma3RMSNorm.forward_native` NATIVE execution path
```python
    def forward_native(self, x):
        output = self._norm(x.float())
        # Llama does x.to(float16) * w whilst Gemma3 is (x * w).to(float16)
        # See https://github.com/huggingface/transformers/pull/29402
        output = output * (1.0 + self.weight.float())
        return output.type_as(x)
```
**EN:** This block defines `Gemma3RMSNorm.forward_native` and contains the main logic for this step. It mainly invokes `self._norm`, `output.type_as`, `x.float`, and `self.weight.float`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma3RMSNorm.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `self._norm`、`output.type_as`、`x.float` 以及 `self.weight.float`，说明该流程会编排底层辅助函数或计算内核。 像 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 747-751: `Gemma3RMSNorm.forward_cpu` CPU execution path
```python
    def forward_cpu(self, x):
        if _is_cpu_amx_available and x.stride(-1) == 1:
            return torch.ops.sgl_kernel.gemma3_rmsnorm_cpu(x, self.weight, self.eps)
        return self.forward_native(x)
```
**EN:** This block defines `Gemma3RMSNorm.forward_cpu` and contains the main logic for this step. It mainly invokes `self.forward_native`, `torch.ops.sgl_kernel.gemma3_rmsnorm_cpu`, and `x.stride`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma3RMSNorm.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`、`torch.ops.sgl_kernel.gemma3_rmsnorm_cpu` 以及 `x.stride`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 752-754: `Gemma3RMSNorm.forward_cuda` CUDA execution path
```python
    def forward_cuda(self, x):
        return self.forward_native(x)
```
**EN:** This block defines `Gemma3RMSNorm.forward_cuda` and contains the main logic for this step. It mainly invokes `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma3RMSNorm.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 755-758: `Gemma3RMSNorm.forward_npu` NPU execution path
```python
    def forward_npu(self, x):
        output, _ = torch_npu.npu_gemma_rms_norm(x, self.weight, self.eps)
        return output
```
**EN:** This block defines `Gemma3RMSNorm.forward_npu` and contains the main logic for this step. It mainly invokes `torch_npu.npu_gemma_rms_norm`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output` and `_` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma3RMSNorm.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `torch_npu.npu_gemma_rms_norm`，说明该流程会编排底层辅助函数或计算内核。 像 `output` 和 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 759-762: Function `Gemma3RMSNorm.extra_repr` and its core logic
```python
    def extra_repr(self):
        return f"{tuple(self.weight.shape)}, eps={self.eps}"
```
**EN:** This block defines `Gemma3RMSNorm.extra_repr` and contains the main logic for this step. It mainly invokes `tuple`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma3RMSNorm.extra_repr`，并承载这一阶段的核心逻辑。 它主要调用 `tuple`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 763-763: Class `Gemma4RMSNorm` declaration and shared state
```python
class Gemma4RMSNorm(MultiPlatformOp):
```
**EN:** This block introduces class `Gemma4RMSNorm` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `Gemma4RMSNorm`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。

### Lines 764-781: `Gemma4RMSNorm` initialization and state setup
```python
    def __init__(
        self,
        dim: int,
        eps: float = 1e-6,
        scale_shift: float = 0.0,
        with_scale: bool = True,
    ):
        super().__init__()
        self.with_scale = with_scale

        if self.with_scale:
            self.weight = nn.Parameter(torch.ones(dim))
        else:
            self.register_buffer("weight", torch.ones(dim), persistent=False)

        self.eps = eps
        self.scale_shift = scale_shift
```
**EN:** This block defines `Gemma4RMSNorm.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `nn.Parameter`, `self.register_buffer`, and `torch.ones`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.with_scale`, `self.eps`, `self.scale_shift`, and `self.weight` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `Gemma4RMSNorm.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`nn.Parameter`、`self.register_buffer` 以及 `torch.ones`，说明该流程会编排底层辅助函数或计算内核。 像 `self.with_scale`、`self.eps`、`self.scale_shift` 以及 `self.weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 782-788: Internal helper `Gemma4RMSNorm.__repr__`
```python
    def __repr__(self):
        dim = self.weight.shape[0]
        return (
            f"{self.__class__.__name__}(dim={dim}, eps={self.eps}, "
            f"with_scale={self.with_scale}, scale_shift={self.scale_shift})"
        )
```
**EN:** This block defines `Gemma4RMSNorm.__repr__` and contains the main logic for this step. Intermediate names such as `dim` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma4RMSNorm.__repr__`，并承载这一阶段的核心逻辑。 像 `dim` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 789-792: Internal helper `Gemma4RMSNorm._norm`
```python
    def _norm(self, x):
        mean_squared = x.pow(2).mean(-1, keepdim=True) + self.eps
        return x * torch.pow(mean_squared, -0.5)
```
**EN:** This block defines `Gemma4RMSNorm._norm` and contains the main logic for this step. It mainly invokes `x.pow.mean`, `torch.pow`, and `x.pow`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `mean_squared` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma4RMSNorm._norm`，并承载这一阶段的核心逻辑。 它主要调用 `x.pow.mean`、`torch.pow` 以及 `x.pow`，说明该流程会编排底层辅助函数或计算内核。 像 `mean_squared` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 793-798: `Gemma4RMSNorm.forward_native` NATIVE execution path
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        normed_output = self._norm(x.float())
        if self.with_scale:
            normed_output = normed_output * (self.weight.float() + self.scale_shift)
        return normed_output.type_as(x)
```
**EN:** This block defines `Gemma4RMSNorm.forward_native` and contains the main logic for this step. It mainly invokes `self._norm`, `normed_output.type_as`, `x.float`, and `self.weight.float`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `normed_output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma4RMSNorm.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `self._norm`、`normed_output.type_as`、`x.float` 以及 `self.weight.float`，说明该流程会编排底层辅助函数或计算内核。 像 `normed_output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 799-805: `Gemma4RMSNorm.forward_cpu` CPU execution path
```python
    def forward_cpu(self, x: torch.Tensor) -> torch.Tensor:
        if _is_cpu_amx_available:
            return torch.ops.sgl_kernel.gemma4_rmsnorm_cpu(
                x, self.weight.data, self.eps, self.scale_shift, self.with_scale
            )
        return self.forward_native(x)
```
**EN:** This block defines `Gemma4RMSNorm.forward_cpu` and contains the main logic for this step. It mainly invokes `self.forward_native` and `torch.ops.sgl_kernel.gemma4_rmsnorm_cpu`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma4RMSNorm.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native` 和 `torch.ops.sgl_kernel.gemma4_rmsnorm_cpu`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 806-825: `Gemma4RMSNorm.forward_cuda` CUDA execution path
```python
    def forward_cuda(self, x: torch.Tensor) -> torch.Tensor:
        if x.numel() == 0:
            return x
        needs_reshape = x.dim() != 2
        if needs_reshape:
            original_shape = x.shape
            x = x.contiguous().reshape(-1, original_shape[-1])
        if self.with_scale and self.scale_shift == 1.0:
            # gemma_rmsnorm: norm(x) * (1 + weight)
            out = gemma_rmsnorm(x, self.weight.data, self.eps)
        else:
            # rmsnorm: norm(x) * weight
            # with_scale=False → weight is ones → norm(x) * 1 = norm(x)
            # scale_shift=0.0 → standard RMSNorm without +1 shift
            out = rmsnorm(x, self.weight.data, self.eps)

        if needs_reshape:
            out = out.reshape(original_shape)
        return out
```
**EN:** This block defines `Gemma4RMSNorm.forward_cuda` and contains the main logic for this step. It mainly invokes `x.numel`, `x.dim`, `x.contiguous.reshape`, `gemma_rmsnorm`, and `rmsnorm`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `needs_reshape`, `original_shape`, `x`, and `out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma4RMSNorm.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `x.numel`、`x.dim`、`x.contiguous.reshape`、`gemma_rmsnorm` 以及 `rmsnorm`，说明该流程会编排底层辅助函数或计算内核。 像 `needs_reshape`、`original_shape`、`x` 以及 `out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 826-831: `Gemma4RMSNorm.forward_hip` HIP execution path
```python
    def forward_hip(self, x: torch.Tensor) -> torch.Tensor:
        # sgl_kernel's gemma_rmsnorm is not available on ROCm;
        # delegate to the pure-PyTorch implementation.
        return self.forward_native(x)
```
**EN:** This block defines `Gemma4RMSNorm.forward_hip` and contains the main logic for this step. It mainly invokes `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `Gemma4RMSNorm.forward_hip`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 832-832: Class `RMSNormWithoutScale` declaration and shared state
```python
class RMSNormWithoutScale(MultiPlatformOp):
```
**EN:** This block introduces class `RMSNormWithoutScale` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `RMSNormWithoutScale`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。

### Lines 833-837: `RMSNormWithoutScale` initialization and state setup
```python
    def __init__(self, hidden_size: int, eps=1e-6):
        super().__init__()
        self.hidden_size = hidden_size
        self.eps = eps
```
**EN:** This block defines `RMSNormWithoutScale.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.hidden_size` and `self.eps` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RMSNormWithoutScale.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self.hidden_size` 和 `self.eps` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 838-840: Internal helper `RMSNormWithoutScale._norm`
```python
    def _norm(self, x):
        return x * torch.rsqrt(x.pow(2).mean(-1, keepdim=True) + self.eps)
```
**EN:** This block defines `RMSNormWithoutScale._norm` and contains the main logic for this step. It mainly invokes `torch.rsqrt`, `x.pow.mean`, and `x.pow`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNormWithoutScale._norm`，并承载这一阶段的核心逻辑。 它主要调用 `torch.rsqrt`、`x.pow.mean` 以及 `x.pow`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 841-847: `RMSNormWithoutScale.forward_native` NATIVE execution path
```python
    def forward_native(self, x):
        orig_dtype = x.dtype
        x = x.float()
        variance = x.pow(2).mean(dim=-1, keepdim=True)
        x = x * torch.rsqrt(variance + self.eps)
        return x.to(orig_dtype)
```
**EN:** This block defines `RMSNormWithoutScale.forward_native` and contains the main logic for this step. It mainly invokes `x.float`, `x.pow.mean`, `x.to`, `torch.rsqrt`, and `x.pow`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `orig_dtype`, `x`, and `variance` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNormWithoutScale.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `x.float`、`x.pow.mean`、`x.to`、`torch.rsqrt` 以及 `x.pow`，说明该流程会编排底层辅助函数或计算内核。 像 `orig_dtype`、`x` 以及 `variance` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 848-850: `RMSNormWithoutScale.forward_cuda` CUDA execution path
```python
    def forward_cuda(self, x):
        return self.forward_native(x)
```
**EN:** This block defines `RMSNormWithoutScale.forward_cuda` and contains the main logic for this step. It mainly invokes `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNormWithoutScale.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 851-852: Function `RMSNormWithoutScale.extra_repr` and its core logic
```python
    def extra_repr(self):
        return f"{self.hidden_size}, eps={self.eps}"
```
**EN:** This block defines `RMSNormWithoutScale.extra_repr` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RMSNormWithoutScale.extra_repr`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_forward_with_allreduce_fusion`, `RMSNorm`, `LayerNorm`, `GemmaRMSNorm`, and `Gemma3RMSNorm`. / **主要符号**：核心入口包括 `_forward_with_allreduce_fusion`、`RMSNorm`、`LayerNorm`、`GemmaRMSNorm` 以及 `Gemma3RMSNorm`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Activation math**: Captures fused nonlinear transforms and their device-specific fast paths. / **激活数学**：描述融合非线性变换及其设备专用快速路径。

## Dependencies / 依赖关系
- **Standard library**: `logging`, `typing.Optional`, `typing.Tuple`, and `typing.Union` / **标准库**：`logging`、`typing.Optional`、`typing.Tuple` 以及 `typing.Union`
- **Third-party**: `torch`, `torch.nn`, `torch.nn.functional`, `sgl_kernel.fused_add_rmsnorm`, `sgl_kernel.gemma_fused_add_rmsnorm`, `sgl_kernel.gemma_rmsnorm`, `sgl_kernel.rmsnorm`, `aiter.layernorm2d_fwd`, `aiter.rmsnorm2d_fwd`, and `aiter.rmsnorm2d_fwd_with_add` / **第三方依赖**：`torch`、`torch.nn`、`torch.nn.functional`、`sgl_kernel.fused_add_rmsnorm`、`sgl_kernel.gemma_fused_add_rmsnorm`、`sgl_kernel.gemma_rmsnorm`、`sgl_kernel.rmsnorm`、`aiter.layernorm2d_fwd`、`aiter.rmsnorm2d_fwd` 以及 `aiter.rmsnorm2d_fwd_with_add`
- **Internal SGLang modules**: `sglang.srt.batch_invariant_ops.is_batch_invariant_mode_enabled`, `sglang.srt.batch_invariant_ops.rms_norm_batch_invariant`, `sglang.srt.environ.envs`, `sglang.srt.layers.utils.MultiPlatformOp`, `sglang.srt.server_args.get_global_server_args`, `sglang.srt.utils.cpu_has_amx_support`, `sglang.srt.utils.get_bool_env_var`, `sglang.srt.utils.is_cpu`, `sglang.srt.utils.is_cuda`, `sglang.srt.utils.is_flashinfer_available`, `sglang.srt.utils.is_hip`, and `sglang.srt.utils.is_musa` / **SGLang 内部模块**：`sglang.srt.batch_invariant_ops.is_batch_invariant_mode_enabled`、`sglang.srt.batch_invariant_ops.rms_norm_batch_invariant`、`sglang.srt.environ.envs`、`sglang.srt.layers.utils.MultiPlatformOp`、`sglang.srt.server_args.get_global_server_args`、`sglang.srt.utils.cpu_has_amx_support`、`sglang.srt.utils.get_bool_env_var`、`sglang.srt.utils.is_cpu`、`sglang.srt.utils.is_cuda`、`sglang.srt.utils.is_flashinfer_available`、`sglang.srt.utils.is_hip` 以及 `sglang.srt.utils.is_musa`
