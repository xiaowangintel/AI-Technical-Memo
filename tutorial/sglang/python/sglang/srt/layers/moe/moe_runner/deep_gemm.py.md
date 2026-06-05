# deep_gemm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/moe_runner/deep_gemm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with linear projections and GEMM backend integration for the SGLang SRT runtime. It exposes symbols such as `_cast_to_e8m0_with_rounding_up`, `copy_list_to_gpu_no_ce`, `DeepGemmRunnerInput`, and `DeepGemmRunnerOutput` and connects them to backend-specific paths such as `CUDA`, `Triton`, `NPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合线性投影与 GEMM 后端集成。它提供了 `_cast_to_e8m0_with_rounding_up`、`copy_list_to_gpu_no_ce`、`DeepGemmRunnerInput` 以及 `DeepGemmRunnerOutput` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`NPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-65: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

from dataclasses import dataclass
from typing import TYPE_CHECKING, Any, List, Optional, Tuple

import einops
import torch

from sglang.jit_kernel.deepseek_v4 import silu_and_mul_masked_post_quant
from sglang.srt.environ import envs
from sglang.srt.layers import deep_gemm_wrapper
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
from sglang.srt.utils import (
    ceil_div,
    dispose_tensor,
    get_bool_env_var,
    is_cuda,
    is_hip,
    is_musa,
    is_npu,
)
from sglang.srt.utils.offloader import get_offloader

if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher.deepep import (
        DeepEPLLCombineInput,
        DeepEPLLDispatchOutput,
        DeepEPNormalCombineInput,
        DeepEPNormalDispatchOutput,
    )
    from sglang.srt.layers.moe.token_dispatcher.standard import (
        StandardCombineInput,
        StandardDispatchOutput,
    )

_is_hip = is_hip()
_is_npu = is_npu()
_is_cuda = is_cuda()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
_is_musa = is_musa()

# Imported only for the SGLANG_OPT_FIX_MEGA_MOE_MEMORY=False fallback path.
if not (_is_npu or _is_hip) and _is_cuda:
    from sglang.jit_kernel.activation import silu_and_mul as _legacy_silu_and_mul
elif _is_musa:
    _silu_and_mul_musa = torch.nn.SwishGLU()
else:
    _legacy_silu_and_mul = None


_MASKED_GEMM_FAST_ACT = get_bool_env_var("SGLANG_MASKED_GEMM_FAST_ACT")
_DEEPGEMM_ON_H20 = get_bool_env_var("SGLANG_DEEPGEMM_ON_H20")


# TODO(kaixih@nvidia): ideally we should merge this logic into
# `fill_gateup_input_triton_kernel` to directly generate e8m0 scale.
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.List`, and `typing.Optional`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_hip`, `_is_npu`, `_is_cuda`, `_use_aiter`, and `_is_musa` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.List` 以及 `typing.Optional`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_hip`、`_is_npu`、`_is_cuda`、`_use_aiter` 以及 `_is_musa` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 66-79: Internal helper `_cast_to_e8m0_with_rounding_up`
```python
@torch.compile(disable=_is_hip or _is_npu)
def _cast_to_e8m0_with_rounding_up(x: torch.Tensor) -> torch.Tensor:
    temp = x.to(torch.float32).view(torch.int32)
    exp = torch.bitwise_right_shift(temp, 23)
    mant = torch.bitwise_and(temp, 0x7FFFFF)
    is_ru = torch.logical_and(
        torch.logical_and((mant > 0), (exp != 0xFE)),
        ~torch.logical_and((exp == 0), (mant <= 0x400000)),
    )
    exp = torch.where(is_ru, exp + 1, exp)
    new_x = exp.to(torch.uint8).view(torch.int)
    return new_x.transpose(1, 2).contiguous().transpose(1, 2)
```
**EN:** This block defines `_cast_to_e8m0_with_rounding_up` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.compile`, `x.to.view`, `torch.bitwise_right_shift`, `torch.bitwise_and`, and `torch.logical_and`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `temp`, `exp`, `mant`, `is_ru`, and `new_x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_cast_to_e8m0_with_rounding_up`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.compile`、`x.to.view`、`torch.bitwise_right_shift`、`torch.bitwise_and` 以及 `torch.logical_and`，说明该流程会编排底层辅助函数或计算内核。 像 `temp`、`exp`、`mant`、`is_ru` 以及 `new_x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 80-88: Function `copy_list_to_gpu_no_ce` and its core logic
```python
def copy_list_to_gpu_no_ce(arr: List[int]):
    from sgl_kernel.elementwise import copy_to_gpu_no_ce

    tensor_cpu = torch.tensor(arr, dtype=torch.int32, device="cpu")
    tensor_gpu = torch.empty_like(tensor_cpu, device="cuda")
    copy_to_gpu_no_ce(tensor_cpu, tensor_gpu)
    return tensor_gpu
```
**EN:** This block defines `copy_list_to_gpu_no_ce` and contains the main logic for this step. It mainly invokes `torch.tensor`, `torch.empty_like`, and `copy_to_gpu_no_ce`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `tensor_cpu` and `tensor_gpu` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `copy_list_to_gpu_no_ce`，并承载这一阶段的核心逻辑。 它主要调用 `torch.tensor`、`torch.empty_like` 以及 `copy_to_gpu_no_ce`，说明该流程会编排底层辅助函数或计算内核。 像 `tensor_cpu` 和 `tensor_gpu` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 89-97: Class `DeepGemmRunnerInput` declaration and shared state
```python
@dataclass
class DeepGemmRunnerInput(RunnerInput):
    hidden_states: torch.Tensor
    hidden_states_scale: torch.Tensor
    use_masked_gemm: bool
    masked_m: Optional[torch.Tensor] = None
    expected_m: Optional[int] = None
    m_indices: Optional[torch.Tensor] = None
```
**EN:** This block introduces class `DeepGemmRunnerInput` and the state shared by its methods. It inherits from `RunnerInput`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DeepGemmRunnerInput`，并定义其方法共享的状态。 它继承自 `RunnerInput`，说明了它在 SRT 层栈中的接入方式。

### Lines 98-102: Function `DeepGemmRunnerInput.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.DEEP_GEMM
```
**EN:** This block defines `DeepGemmRunnerInput.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepGemmRunnerInput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 103-106: Class `DeepGemmRunnerOutput` declaration and shared state
```python
@dataclass
class DeepGemmRunnerOutput(RunnerOutput):
    hidden_states: torch.Tensor
```
**EN:** This block introduces class `DeepGemmRunnerOutput` and the state shared by its methods. It inherits from `RunnerOutput`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DeepGemmRunnerOutput`，并定义其方法共享的状态。 它继承自 `RunnerOutput`，说明了它在 SRT 层栈中的接入方式。

### Lines 107-111: Function `DeepGemmRunnerOutput.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.DEEP_GEMM
```
**EN:** This block defines `DeepGemmRunnerOutput.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepGemmRunnerOutput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 112-123: Class `DeepGemmMoeQuantInfo` declaration and shared state
```python
@dataclass
class DeepGemmMoeQuantInfo(MoeQuantInfo):
    w13_weight: torch.Tensor
    w2_weight: torch.Tensor
    use_fp8: bool
    w13_scale: Optional[torch.Tensor] = None
    w2_scale: Optional[torch.Tensor] = None
    block_shape: Optional[List[int]] = None
    # DSV4 mxfp4 layout flag; selects recipe_a=(1,128)/recipe_b=(1,32) downstream.
    is_fp4_experts: bool = False
```
**EN:** This block introduces class `DeepGemmMoeQuantInfo` and the state shared by its methods. It inherits from `MoeQuantInfo`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DeepGemmMoeQuantInfo`，并定义其方法共享的状态。 它继承自 `MoeQuantInfo`，说明了它在 SRT 层栈中的接入方式。

### Lines 124-124: Class `DeepGemmRunnerCore` declaration and shared state
```python
class DeepGemmRunnerCore(MoeRunnerCore):
```
**EN:** This block introduces class `DeepGemmRunnerCore` and the state shared by its methods. It inherits from `MoeRunnerCore`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `DeepGemmRunnerCore`，并定义其方法共享的状态。 它继承自 `MoeRunnerCore`，说明了它在 SRT 层栈中的接入方式。

### Lines 125-135: `DeepGemmRunnerCore` initialization and state setup
```python
    def __init__(self, config: MoeRunnerConfig):
        super().__init__(config)
        assert self.config.activation == "silu"
        assert self.config.is_gated
        self.swiglu_limit = self.config.swiglu_limit
        self.use_swizzle = False
        if envs.SGLANG_OPT_FIX_MEGA_MOE_MEMORY.get():
            assert envs.SGLANG_OPT_SWIGLU_CLAMP_FUSION.get()
            assert envs.SGLANG_OPT_USE_JIT_EP_ACTIVATION.get()
            self.use_swizzle = True
```
**EN:** This block defines `DeepGemmRunnerCore.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `envs.SGLANG_OPT_FIX_MEGA_MOE_MEMORY.get`, `envs.SGLANG_OPT_SWIGLU_CLAMP_FUSION.get`, and `envs.SGLANG_OPT_USE_JIT_EP_ACTIVATION.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.swiglu_limit` and `self.use_swizzle` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DeepGemmRunnerCore.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`envs.SGLANG_OPT_FIX_MEGA_MOE_MEMORY.get`、`envs.SGLANG_OPT_SWIGLU_CLAMP_FUSION.get` 以及 `envs.SGLANG_OPT_USE_JIT_EP_ACTIVATION.get`，说明该流程会编排底层辅助函数或计算内核。 像 `self.swiglu_limit` 和 `self.use_swizzle` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 136-163: Function `DeepGemmRunnerCore.run` and its core logic
```python
    def run(
        self,
        runner_input: DeepGemmRunnerInput,
        quant_info: DeepGemmMoeQuantInfo,
        running_state: dict,
        hooks: Optional[Any] = None,
    ) -> DeepGemmRunnerOutput:
        weight_dtype = quant_info.w13_weight.dtype
        if not runner_input.use_masked_gemm:
            if weight_dtype == torch.bfloat16:
                hidden_states = self._run_bf16_contiguous_gemm(
                    runner_input, quant_info, running_state
                )
            else:
                hidden_states = self._run_contiguous_gemm(
                    runner_input, quant_info, running_state
                )
        else:
            if weight_dtype == torch.bfloat16:
                hidden_states = self._run_masked_bf16_gemm(
                    runner_input, quant_info, running_state
                )
            else:
                hidden_states = self._run_masked_gemm(
                    runner_input, quant_info, running_state
                )
        return DeepGemmRunnerOutput(hidden_states=hidden_states)
```
**EN:** This block defines `DeepGemmRunnerCore.run` and contains the main logic for this step. It mainly invokes `DeepGemmRunnerOutput`, `self._run_bf16_contiguous_gemm`, `self._run_contiguous_gemm`, `self._run_masked_bf16_gemm`, and `self._run_masked_gemm`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `weight_dtype` and `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepGemmRunnerCore.run`，并承载这一阶段的核心逻辑。 它主要调用 `DeepGemmRunnerOutput`、`self._run_bf16_contiguous_gemm`、`self._run_contiguous_gemm`、`self._run_masked_bf16_gemm` 以及 `self._run_masked_gemm`，说明该流程会编排底层辅助函数或计算内核。 像 `weight_dtype` 和 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 164-296: Internal helper `DeepGemmRunnerCore._run_contiguous_gemm`
```python
    def _run_contiguous_gemm(
        self,
        runner_input: DeepGemmRunnerInput,
        quant_info: DeepGemmMoeQuantInfo,
        running_state: dict,
    ) -> torch.Tensor:
        from sglang.jit_kernel.deepseek_v4 import silu_and_mul_contig_post_quant
        from sglang.srt.layers.moe.ep_moe.kernels import tma_align_input_scale
        from sglang.srt.layers.quantization.fp8_kernel import (
            create_per_token_group_quant_fp8_output_scale,
        )

        hidden_states = runner_input.hidden_states
        hidden_states_scale = runner_input.hidden_states_scale
        all_tokens = running_state["all_tokens"]
        hidden_states_device = running_state["hidden_states_device"]
        hidden_states_dtype = running_state["hidden_states_dtype"]
        hidden_states_shape = running_state["hidden_states_shape"]
        m_indices = runner_input.m_indices

        N = quant_info.w13_weight.size(1)
        K = hidden_states_shape[1]
        scale_block_size = 128

        recipe_a, recipe_b = (
            ((1, 128), (1, 32)) if quant_info.is_fp4_experts else (None, None)
        )

        w13_weight_fp8 = (
            quant_info.w13_weight,
            quant_info.w13_scale,
        )
        w2_weight_fp8 = (quant_info.w2_weight, quant_info.w2_scale)

        gateup_output = torch.empty(
            (all_tokens, N),
            device=hidden_states_device,
            dtype=torch.bfloat16,
        )
        if deep_gemm_wrapper.DEEPGEMM_NEED_TMA_ALIGNED_SCALES:
            hidden_states_scale = tma_align_input_scale(hidden_states_scale)

        deep_gemm_wrapper.grouped_gemm_nt_f8f8bf16_contig(
            (hidden_states, hidden_states_scale),
            w13_weight_fp8,
            gateup_output,
            m_indices,
            recipe_a=recipe_a,
            recipe_b=recipe_b,
        )

        dispose_tensor(hidden_states)
        dispose_tensor(hidden_states_scale)

        if envs.SGLANG_OPT_FIX_MEGA_MOE_MEMORY.get():
            swiglu_limit_arg: Optional[float] = self.swiglu_limit

            down_input_fp8 = torch.empty(
                (all_tokens, N // 2),
                device=gateup_output.device,
                dtype=torch.float8_e4m3fn,
            )
            down_input_scale = create_per_token_group_quant_fp8_output_scale(
                x_shape=(all_tokens, N // 2),
                device=gateup_output.device,
                group_size=scale_block_size,
                column_major_scales=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
                scale_tma_aligned=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
                scale_ue8m0=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
            )
            silu_and_mul_contig_post_quant(
                input=gateup_output,
                output=down_input_fp8,
                output_scale=down_input_scale,
                quant_group_size=scale_block_size,
                scale_ue8m0=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
                transposed=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
                swiglu_limit=swiglu_limit_arg,
                swizzle=self.use_swizzle,
            )
            del gateup_output
        else:
            # Hacky byte-equal fallback that reproduces the optimize-branch
            # code path exactly: bf16 silu_and_mul then a separate per-token
            # group fp8 quant. Kept behind the mega-moe-memory flag.
            from sglang.srt.layers.quantization.fp8_kernel import (
                sglang_per_token_group_quant_fp8,
            )

            if self.swiglu_limit is not None:
                gateup_output = _apply_swiglu_limit(
                    gateup_output, swiglu_limit=self.swiglu_limit
                )

            if not _is_musa:
                down_input = torch.empty(
                    (all_tokens, N // 2),
                    device=gateup_output.device,
                    dtype=torch.bfloat16,
                )
                _legacy_silu_and_mul(gateup_output.view(-1, N), down_input)
            else:
                down_input = _silu_and_mul_musa(gateup_output.view(-1, N))
            del gateup_output

            down_input_fp8, down_input_scale = sglang_per_token_group_quant_fp8(
                down_input,
                scale_block_size,
                column_major_scales=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
                scale_tma_aligned=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
                scale_ue8m0=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
            )
            del down_input

        down_output = torch.empty(
            (all_tokens, K),
            device=hidden_states_device,
            dtype=torch.bfloat16,
        )
        if deep_gemm_wrapper.DEEPGEMM_NEED_TMA_ALIGNED_SCALES:
            down_input_scale = tma_align_input_scale(down_input_scale)

        deep_gemm_wrapper.grouped_gemm_nt_f8f8bf16_contig(
            (down_input_fp8, down_input_scale),
            w2_weight_fp8,
            down_output,
            m_indices,
            recipe_a=recipe_a,
            recipe_b=recipe_b,
        )

        return down_output
```
**EN:** This block defines `DeepGemmRunnerCore._run_contiguous_gemm` and contains the main logic for this step. It mainly invokes `quant_info.w13_weight.size`, `torch.empty`, `deep_gemm_wrapper.grouped_gemm_nt_f8f8bf16_contig`, `dispose_tensor`, and `envs.SGLANG_OPT_FIX_MEGA_MOE_MEMORY.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `hidden_states_scale`, `all_tokens`, `hidden_states_device`, and `hidden_states_dtype` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepGemmRunnerCore._run_contiguous_gemm`，并承载这一阶段的核心逻辑。 它主要调用 `quant_info.w13_weight.size`、`torch.empty`、`deep_gemm_wrapper.grouped_gemm_nt_f8f8bf16_contig`、`dispose_tensor` 以及 `envs.SGLANG_OPT_FIX_MEGA_MOE_MEMORY.get`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`hidden_states_scale`、`all_tokens`、`hidden_states_device` 以及 `hidden_states_dtype` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 297-361: Internal helper `DeepGemmRunnerCore._run_bf16_contiguous_gemm`
```python
    def _run_bf16_contiguous_gemm(
        self,
        runner_input: DeepGemmRunnerInput,
        quant_info: DeepGemmMoeQuantInfo,
        running_state: dict,
    ) -> torch.Tensor:

        hidden_states = runner_input.hidden_states
        all_tokens = running_state["all_tokens"]
        hidden_states_device = running_state["hidden_states_device"]
        hidden_states_shape = running_state["hidden_states_shape"]
        m_indices = runner_input.m_indices

        N = quant_info.w13_weight.size(1)
        K = hidden_states_shape[1]

        w13_weight = quant_info.w13_weight
        w2_weight = quant_info.w2_weight

        # GroupGemm-1: (M, K) (E, N, K) -> (M, N)
        gateup_output = torch.empty(
            (all_tokens, N),
            device=hidden_states_device,
            dtype=torch.bfloat16,
        )

        deep_gemm_wrapper.grouped_gemm_nt_bf16_contig(
            hidden_states,
            w13_weight,
            gateup_output,
            m_indices,
        )

        dispose_tensor(hidden_states)

        # Act: (M, N) -> (M, N/2)
        if not _is_musa:
            down_input = torch.empty(
                (
                    all_tokens,
                    N // 2,
                ),
                device=gateup_output.device,
                dtype=torch.bfloat16,
            )
            _legacy_silu_and_mul(gateup_output.view(-1, N), down_input)
        else:
            down_input = _silu_and_mul_musa(gateup_output.view(-1, N))
        del gateup_output

        # GroupGemm-2: (M, N/2) (E, K, N/2) -> (M, K)
        down_output = torch.empty(
            (all_tokens, K),
            device=hidden_states_device,
            dtype=torch.bfloat16,
        )
        deep_gemm_wrapper.grouped_gemm_nt_bf16_contig(
            down_input,
            w2_weight,
            down_output,
            m_indices,
        )

        return down_output
```
**EN:** This block defines `DeepGemmRunnerCore._run_bf16_contiguous_gemm` and contains the main logic for this step. It mainly invokes `quant_info.w13_weight.size`, `torch.empty`, `deep_gemm_wrapper.grouped_gemm_nt_bf16_contig`, `dispose_tensor`, and `_legacy_silu_and_mul`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `all_tokens`, `hidden_states_device`, `hidden_states_shape`, and `m_indices` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepGemmRunnerCore._run_bf16_contiguous_gemm`，并承载这一阶段的核心逻辑。 它主要调用 `quant_info.w13_weight.size`、`torch.empty`、`deep_gemm_wrapper.grouped_gemm_nt_bf16_contig`、`dispose_tensor` 以及 `_legacy_silu_and_mul`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`all_tokens`、`hidden_states_device`、`hidden_states_shape` 以及 `m_indices` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 362-495: Internal helper `DeepGemmRunnerCore._run_masked_gemm`
```python
    def _run_masked_gemm(
        self,
        runner_input: DeepGemmRunnerInput,
        quant_info: DeepGemmMoeQuantInfo,
        running_state: dict,
    ) -> torch.Tensor:
        from sglang.srt.layers import deep_gemm_wrapper

        hidden_states = runner_input.hidden_states
        hidden_states_scale = runner_input.hidden_states_scale
        masked_m = runner_input.masked_m
        expected_m = runner_input.expected_m

        w13_weight = quant_info.w13_weight
        w2_weight = quant_info.w2_weight
        w13_scale = quant_info.w13_scale
        w2_scale = quant_info.w2_scale

        recipe_a, recipe_b = (
            ((1, 128), (1, 32)) if quant_info.is_fp4_experts else (None, None)
        )

        hidden_states_device = running_state["hidden_states_device"]

        # GroupGemm-0
        if deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0:
            if hidden_states_scale.dtype != torch.int:
                b, s_mn, s_k = hidden_states_scale.shape
                assert (
                    s_mn % 4 == 0 and s_k % 4 == 0
                ), f"scales must be aligned to 4, but got ({b}, {s_mn}, {s_k})"
                hidden_states_scale = _cast_to_e8m0_with_rounding_up(
                    hidden_states_scale
                )
        elif deep_gemm_wrapper.DEEPGEMM_NEED_TMA_ALIGNED_SCALES:
            hidden_states_scale = deep_gemm_wrapper.get_mn_major_tma_aligned_tensor(
                hidden_states_scale
            )

        num_groups, m, k = hidden_states.shape
        n = w13_weight.size(1)
        gateup_output = torch.empty(
            (num_groups, m, n), device=hidden_states_device, dtype=torch.bfloat16
        )
        deep_gemm_wrapper.grouped_gemm_nt_f8f8bf16_masked(
            (hidden_states, hidden_states_scale),
            (w13_weight, w13_scale),
            gateup_output,
            masked_m,
            expected_m,
            recipe_a=recipe_a,
            recipe_b=recipe_b,
        )
        dispose_tensor(hidden_states)
        dispose_tensor(hidden_states_scale)

        swiglu_limit_arg: Optional[float] = None
        if self.swiglu_limit is not None:
            # DeepSeek V4: clamped swiglu requires JIT EP activation; the
            # FAST_ACT fused-quant path doesn't carry a swiglu_limit arg.
            assert (
                not _MASKED_GEMM_FAST_ACT
            ), "DeepSeek V4 does not support SGLANG_MASKED_GEMM_FAST_ACT"
            assert (
                envs.SGLANG_OPT_USE_JIT_EP_ACTIVATION.get()
            ), "DeepSeek V4 requires SGLANG_OPT_USE_JIT_EP_ACTIVATION=True"

            if envs.SGLANG_OPT_SWIGLU_CLAMP_FUSION.get():
                swiglu_limit_arg = self.swiglu_limit
            else:
                gateup_output = einops.rearrange(
                    gateup_output, "grp tok hidden -> (grp tok) hidden"
                )
                gateup_output = _apply_swiglu_limit(
                    gateup_output, swiglu_limit=self.swiglu_limit
                )
                gateup_output = einops.rearrange(
                    gateup_output, "(grp tok) hidden -> grp tok hidden", grp=num_groups
                )

        # Act
        down_input, down_input_scale = _varlen_deep_gemm_silu_mul_quant(
            gateup_output,
            masked_m,
            group_size=128,
            topk=self.config.top_k,
            swiglu_limit=swiglu_limit_arg,
            swizzle=self.use_swizzle,
        )
        del gateup_output

        # GroupGemm-1
        n = w2_weight.shape[1]

        if deep_gemm_wrapper.DEEPGEMM_NEED_TMA_ALIGNED_SCALES:
            down_input_scale = deep_gemm_wrapper.get_mn_major_tma_aligned_tensor(
                down_input_scale
            )

        down_output = torch.empty(
            (num_groups, m, n), device=hidden_states_device, dtype=torch.bfloat16
        )

        down_gemm_overlap_args = running_state.get("down_gemm_overlap_args", None)
        if down_gemm_overlap_args is None:
            gemm_overlap_args_dict = {}
        else:
            down_gemm_overlap_args.start_event.record()
            max_block_n = (
                160 if (_DEEPGEMM_ON_H20 and runner_input.expected_m <= 64) else 256
            )
            gemm_overlap_args_dict = {
                "overlap_args": down_gemm_overlap_args,
                "max_block_n": max_block_n,
            }

        deep_gemm_return_value = deep_gemm_wrapper.grouped_gemm_nt_f8f8bf16_masked(
            (down_input, down_input_scale),
            (w2_weight, w2_scale),
            down_output,
            masked_m,
            expected_m,
            recipe_a=recipe_a,
            recipe_b=recipe_b,
            **gemm_overlap_args_dict,
        )
        meta_overlap_args = running_state.get("meta_overlap_args", None)
        if meta_overlap_args is not None:
            block_m, threshold = deep_gemm_return_value
            meta_overlap_args["block_m"] = block_m
            meta_overlap_args["threshold"] = threshold

        return down_output
```
**EN:** This block defines `DeepGemmRunnerCore._run_masked_gemm` and contains the main logic for this step. It mainly invokes `w13_weight.size`, `torch.empty`, `deep_gemm_wrapper.grouped_gemm_nt_f8f8bf16_masked`, `dispose_tensor`, and `_varlen_deep_gemm_silu_mul_quant`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `hidden_states_scale`, `masked_m`, `expected_m`, and `w13_weight` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepGemmRunnerCore._run_masked_gemm`，并承载这一阶段的核心逻辑。 它主要调用 `w13_weight.size`、`torch.empty`、`deep_gemm_wrapper.grouped_gemm_nt_f8f8bf16_masked`、`dispose_tensor` 以及 `_varlen_deep_gemm_silu_mul_quant`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`hidden_states_scale`、`masked_m`、`expected_m` 以及 `w13_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 496-559: Internal helper `DeepGemmRunnerCore._run_masked_bf16_gemm`
```python
    def _run_masked_bf16_gemm(
        self,
        runner_input: DeepGemmRunnerInput,
        quant_info: DeepGemmMoeQuantInfo,
        running_state: dict,
    ) -> torch.Tensor:
        from sglang.srt.layers import deep_gemm_wrapper
        from sglang.srt.layers.moe.ep_moe.kernels import silu_and_mul_masked_fwd

        hidden_states = runner_input.hidden_states
        masked_m = runner_input.masked_m
        expected_m = runner_input.expected_m

        w13_weight = quant_info.w13_weight
        w2_weight = quant_info.w2_weight

        hidden_states_device = running_state["hidden_states_device"]

        # GroupGemm-0
        num_groups, m, k = hidden_states.shape
        n = w13_weight.size(1)
        gateup_output = torch.empty(
            (num_groups, m, n), device=hidden_states_device, dtype=torch.bfloat16
        )
        deep_gemm_wrapper.grouped_gemm_nt_bf16_masked(
            hidden_states,
            w13_weight,
            gateup_output,
            masked_m,
            expected_m,
        )
        dispose_tensor(hidden_states)

        down_input = torch.empty(
            (
                gateup_output.shape[0],
                gateup_output.shape[1],
                gateup_output.shape[2] // 2,
            ),
            device=hidden_states_device,
            dtype=torch.bfloat16,
        )

        # Act
        silu_and_mul_masked_fwd(gateup_output, down_input, masked_m)
        del gateup_output

        # GroupGemm-1
        n = w2_weight.shape[1]

        down_output = torch.empty(
            (num_groups, m, n), device=hidden_states_device, dtype=torch.bfloat16
        )
        deep_gemm_wrapper.grouped_gemm_nt_bf16_masked(
            down_input,
            w2_weight,
            down_output,
            masked_m,
            expected_m,
        )
        # Note: BF16 masked gemm doesn't support overlap_args, so no return value unpack

        return down_output
```
**EN:** This block defines `DeepGemmRunnerCore._run_masked_bf16_gemm` and contains the main logic for this step. It mainly invokes `w13_weight.size`, `torch.empty`, `deep_gemm_wrapper.grouped_gemm_nt_bf16_masked`, `dispose_tensor`, and `silu_and_mul_masked_fwd`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `masked_m`, `expected_m`, `w13_weight`, and `w2_weight` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepGemmRunnerCore._run_masked_bf16_gemm`，并承载这一阶段的核心逻辑。 它主要调用 `w13_weight.size`、`torch.empty`、`deep_gemm_wrapper.grouped_gemm_nt_bf16_masked`、`dispose_tensor` 以及 `silu_and_mul_masked_fwd`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`masked_m`、`expected_m`、`w13_weight` 以及 `w2_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 560-564: Function `DeepGemmRunnerCore.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.DEEP_GEMM
```
**EN:** This block defines `DeepGemmRunnerCore.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepGemmRunnerCore.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 565-615: Function `pre_permute_standard_to_deep_gemm` and its core logic
```python
@register_pre_permute("standard", "deep_gemm")
def pre_permute_standard_to_deep_gemm(
    dispatch_output: StandardDispatchOutput,
    quant_info: DeepGemmMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> DeepGemmRunnerInput:
    from sglang.srt.layers.moe.ep_moe.kernels import moe_ep_deepgemm_preprocess

    hidden_states, topk_output = (
        dispatch_output.hidden_states,
        dispatch_output.topk_output,
    )
    topk_weights, topk_ids, _ = topk_output

    hidden_states_shape = hidden_states.shape
    hidden_states_dtype = hidden_states.dtype
    hidden_states_device = hidden_states.device
    hidden_states_ref = hidden_states

    topk_weights, topk_ids = topk_weights, topk_ids

    # PreReorder
    masked_m, expected_m, src2dst, hidden_states, hidden_states_scale = (
        moe_ep_deepgemm_preprocess(
            topk_ids,
            runner_config.num_local_experts,
            hidden_states,
            runner_config.top_k,
            quant_info.block_shape,
        )
    )

    dispose_tensor(hidden_states_ref)

    running_state["topk_ids"] = topk_ids
    running_state["topk_weights"] = topk_weights
    running_state["hidden_states_shape"] = hidden_states_shape
    running_state["hidden_states_dtype"] = hidden_states_dtype
    running_state["hidden_states_device"] = hidden_states_device
    running_state["src2dst"] = src2dst

    return DeepGemmRunnerInput(
        hidden_states=hidden_states,
        hidden_states_scale=hidden_states_scale,
        use_masked_gemm=True,
        masked_m=masked_m,
        expected_m=expected_m,
    )
```
**EN:** This block defines `pre_permute_standard_to_deep_gemm` and contains the main logic for this step. Decorators like `register_pre_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_pre_permute`, `moe_ep_deepgemm_preprocess`, `dispose_tensor`, and `DeepGemmRunnerInput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_output`, `topk_weights`, `topk_ids`, and `_` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `pre_permute_standard_to_deep_gemm`，并承载这一阶段的核心逻辑。 像 `register_pre_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_pre_permute`、`moe_ep_deepgemm_preprocess`、`dispose_tensor` 以及 `DeepGemmRunnerInput`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_output`、`topk_weights`、`topk_ids` 以及 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 616-656: Function `post_permute_deep_gemm_to_standard` and its core logic
```python
@register_post_permute("deep_gemm", "standard")
def post_permute_deep_gemm_to_standard(
    runner_output: DeepGemmRunnerOutput,
    quant_info: DeepGemmMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> StandardCombineInput:
    from sglang.srt.layers.moe.ep_moe.kernels import post_reorder_triton_kernel
    from sglang.srt.layers.moe.token_dispatcher.standard import StandardCombineInput

    hidden_states_shape = running_state["hidden_states_shape"]
    hidden_states_dtype = running_state["hidden_states_dtype"]
    hidden_states_device = running_state["hidden_states_device"]
    src2dst = running_state["src2dst"]
    topk_ids = running_state["topk_ids"]
    topk_weights = running_state["topk_weights"]

    output = torch.empty(
        hidden_states_shape, dtype=hidden_states_dtype, device=hidden_states_device
    )
    post_reorder_triton_kernel[(hidden_states_shape[0],)](
        runner_output.hidden_states,
        output,
        src2dst,
        topk_ids,
        topk_weights,
        runner_config.top_k,
        hidden_states_shape[1],
        BLOCK_SIZE=512,
    )

    dispose_tensor(runner_output.hidden_states)

    if runner_config.routed_scaling_factor is not None:
        output *= runner_config.routed_scaling_factor

    return StandardCombineInput(
        hidden_states=output,
    )
```
**EN:** This block defines `post_permute_deep_gemm_to_standard` and contains the main logic for this step. Decorators like `register_post_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_post_permute`, `torch.empty`, `post_reorder_triton_kernel`, `dispose_tensor`, and `StandardCombineInput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states_shape`, `hidden_states_dtype`, `hidden_states_device`, `src2dst`, and `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `post_permute_deep_gemm_to_standard`，并承载这一阶段的核心逻辑。 像 `register_post_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_post_permute`、`torch.empty`、`post_reorder_triton_kernel`、`dispose_tensor` 以及 `StandardCombineInput`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states_shape`、`hidden_states_dtype`、`hidden_states_device`、`src2dst` 以及 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 657-682: Function `pre_permute_deepep_ll_to_deep_gemm` and its core logic
```python
@register_pre_permute("deepep_ll", "deep_gemm")
def pre_permute_deepep_ll_to_deep_gemm(
    dispatch_output: DeepEPLLDispatchOutput,
    quant_info: DeepGemmMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> DeepGemmRunnerInput:
    hidden_states, hidden_states_scale, topk_ids, topk_weights, masked_m, expected_m = (
        dispatch_output
    )

    running_state["topk_ids"] = topk_ids
    running_state["topk_weights"] = topk_weights
    running_state["hidden_states_shape"] = hidden_states.shape
    running_state["hidden_states_dtype"] = hidden_states.dtype
    running_state["hidden_states_device"] = hidden_states.device

    return DeepGemmRunnerInput(
        hidden_states=hidden_states,
        hidden_states_scale=hidden_states_scale,
        use_masked_gemm=True,
        masked_m=masked_m,
        expected_m=expected_m,
    )
```
**EN:** This block defines `pre_permute_deepep_ll_to_deep_gemm` and contains the main logic for this step. Decorators like `register_pre_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_pre_permute` and `DeepGemmRunnerInput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `hidden_states_scale`, `topk_ids`, `topk_weights`, and `masked_m` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `pre_permute_deepep_ll_to_deep_gemm`，并承载这一阶段的核心逻辑。 像 `register_pre_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_pre_permute` 和 `DeepGemmRunnerInput`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`hidden_states_scale`、`topk_ids`、`topk_weights` 以及 `masked_m` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 683-698: Function `post_permute_deep_gemm_to_deepep_ll` and its core logic
```python
@register_post_permute("deep_gemm", "deepep_ll")
def post_permute_deep_gemm_to_deepep_ll(
    runner_output: DeepGemmRunnerOutput,
    quant_info: DeepGemmMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> DeepEPLLCombineInput:
    from sglang.srt.layers.moe.token_dispatcher.deepep import DeepEPLLCombineInput

    return DeepEPLLCombineInput(
        hidden_states=runner_output.hidden_states,
        topk_ids=running_state["topk_ids"],
        topk_weights=running_state["topk_weights"],
    )
```
**EN:** This block defines `post_permute_deep_gemm_to_deepep_ll` and contains the main logic for this step. Decorators like `register_post_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_post_permute` and `DeepEPLLCombineInput`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `post_permute_deep_gemm_to_deepep_ll`，并承载这一阶段的核心逻辑。 像 `register_post_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_post_permute` 和 `DeepEPLLCombineInput`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 699-791: Function `pre_permute_deepep_normal_to_deep_gemm` and its core logic
```python
@register_pre_permute("deepep_normal", "deep_gemm")
def pre_permute_deepep_normal_to_deep_gemm(
    dispatch_output: DeepEPNormalDispatchOutput,
    quant_info: DeepGemmMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> DeepGemmRunnerInput:
    from sglang.srt.layers.moe.ep_moe.kernels import ep_scatter

    (
        hidden_states,
        hidden_states_scale,
        topk_ids,
        topk_weights,
        num_recv_tokens_per_expert,
    ) = dispatch_output
    assert runner_config.activation == "silu"

    all_tokens = sum(num_recv_tokens_per_expert)
    running_state["all_tokens"] = all_tokens

    K = hidden_states.shape[1]

    hidden_states_shape = hidden_states.shape
    hidden_states_device = hidden_states.device
    hidden_states_dtype = hidden_states.dtype

    running_state["hidden_states_shape"] = hidden_states_shape
    running_state["hidden_states_device"] = hidden_states_device
    running_state["hidden_states_dtype"] = hidden_states_dtype
    running_state["topk_ids"] = topk_ids
    running_state["topk_weights"] = topk_weights

    input_tensor = torch.empty(
        (all_tokens, K),
        device=hidden_states.device,
        dtype=hidden_states.dtype,
    )
    if deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0:
        # TODO check whether need `zeros`
        input_tensor_scale = torch.zeros(
            (ceil_div(K // 128, 4), all_tokens),
            device=hidden_states.device,
            dtype=torch.int,
        ).transpose(0, 1)
    else:
        input_tensor_scale = torch.empty(
            (all_tokens, K // 128),
            device=hidden_states.device,
            dtype=torch.float32,
        )
    m_indices = torch.empty(all_tokens, device=hidden_states.device, dtype=torch.int32)
    output_index = torch.empty_like(topk_ids)

    if get_offloader().forbid_copy_engine_usage:
        num_recv_tokens_per_expert_gpu = copy_list_to_gpu_no_ce(
            num_recv_tokens_per_expert
        )
    else:
        num_recv_tokens_per_expert_gpu = torch.tensor(
            num_recv_tokens_per_expert,
            dtype=torch.int32,
            pin_memory=True,
            device="cpu",
        ).cuda(non_blocking=True)
    expert_start_loc = torch.empty_like(num_recv_tokens_per_expert_gpu)

    ep_scatter(
        hidden_states,
        hidden_states_scale,
        topk_ids,
        num_recv_tokens_per_expert_gpu,
        expert_start_loc,
        input_tensor,
        input_tensor_scale,
        m_indices,
        output_index,
        scale_ue8m0=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
    )
    dispose_tensor(hidden_states)
    if hidden_states_scale is not None:
        dispose_tensor(hidden_states_scale)

    running_state["output_index"] = output_index

    return DeepGemmRunnerInput(
        hidden_states=input_tensor,
        hidden_states_scale=input_tensor_scale,
        use_masked_gemm=False,
        m_indices=m_indices,
    )
```
**EN:** This block defines `pre_permute_deepep_normal_to_deep_gemm` and contains the main logic for this step. Decorators like `register_pre_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_pre_permute`, `sum`, `torch.empty`, `torch.empty_like`, and `ep_scatter`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `hidden_states_scale`, `topk_ids`, `topk_weights`, and `num_recv_tokens_per_expert` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `pre_permute_deepep_normal_to_deep_gemm`，并承载这一阶段的核心逻辑。 像 `register_pre_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_pre_permute`、`sum`、`torch.empty`、`torch.empty_like` 以及 `ep_scatter`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`hidden_states_scale`、`topk_ids`、`topk_weights` 以及 `num_recv_tokens_per_expert` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 792-820: Function `post_permute_deep_gemm_to_deepep_normal` and its core logic
```python
@register_post_permute("deep_gemm", "deepep_normal")
def post_permute_deep_gemm_to_deepep_normal(
    runner_output: DeepGemmRunnerOutput,
    quant_info: DeepGemmMoeQuantInfo,
    runner_config: MoeRunnerConfig,
    running_state: dict,
) -> DeepEPNormalCombineInput:
    from sglang.srt.layers.moe.ep_moe.kernels import ep_gather
    from sglang.srt.layers.moe.token_dispatcher.deepep import DeepEPNormalCombineInput

    hidden_states = runner_output.hidden_states
    topk_ids = running_state["topk_ids"]
    topk_weights = running_state["topk_weights"]
    output_index = running_state["output_index"]

    gather_out = torch.empty(
        running_state["hidden_states_shape"],
        device=running_state["hidden_states_device"],
        dtype=torch.bfloat16,
    )
    ep_gather(hidden_states, topk_ids, topk_weights, output_index, gather_out)

    return DeepEPNormalCombineInput(
        hidden_states=gather_out,
        topk_ids=running_state["topk_ids"],
        topk_weights=running_state["topk_weights"],
    )
```
**EN:** This block defines `post_permute_deep_gemm_to_deepep_normal` and contains the main logic for this step. Decorators like `register_post_permute` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_post_permute`, `torch.empty`, `ep_gather`, and `DeepEPNormalCombineInput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_ids`, `topk_weights`, `output_index`, and `gather_out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `post_permute_deep_gemm_to_deepep_normal`，并承载这一阶段的核心逻辑。 像 `register_post_permute` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_post_permute`、`torch.empty`、`ep_gather` 以及 `DeepEPNormalCombineInput`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_ids`、`topk_weights`、`output_index` 以及 `gather_out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 821-910: Internal helper `_varlen_deep_gemm_silu_mul_quant`
```python
def _varlen_deep_gemm_silu_mul_quant(
    gateup_output: torch.Tensor,
    masked_m: Optional[torch.Tensor],
    group_size: int,
    topk: int,
    swiglu_limit: Optional[float] = None,
    swizzle: bool = False,
) -> Tuple[torch.Tensor, torch.Tensor]:
    from sglang.srt.layers.moe.ep_moe.kernels import silu_and_mul_masked_post_quant_fwd
    from sglang.srt.layers.quantization.fp8_kernel import (
        sglang_per_token_group_quant_8bit,
    )

    if _MASKED_GEMM_FAST_ACT:
        assert not swizzle, (
            "SGLANG_OPT_FIX_MEGA_MOE_MEMORY is incompatible with "
            "SGLANG_MASKED_GEMM_FAST_ACT (swizzled layout only supported by JIT act)"
        )
        assert (
            swiglu_limit is None
        ), "swiglu_limit (DeepSeek V4) is not supported together with SGLANG_MASKED_GEMM_FAST_ACT"
        return sglang_per_token_group_quant_8bit(
            x=gateup_output,
            dst_dtype=torch.float8_e4m3fn,
            group_size=group_size,
            masked_m=masked_m,
            column_major_scales=True,
            scale_tma_aligned=True,
            scale_ue8m0=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
            fuse_silu_and_mul=True,
            enable_v2=True,
        )

    assert masked_m is not None
    hidden_states_device = gateup_output.device
    E, N, D_2 = gateup_output.shape
    D = D_2 // 2
    del D_2
    G = D // group_size
    down_input = torch.empty(
        (E, N, D),
        device=hidden_states_device,
        dtype=torch.float8_e4m3fn,
    )

    if envs.SGLANG_OPT_USE_JIT_EP_ACTIVATION.get():
        assert N % 4 == 0 and G % 4 == 0
        packed_ue8m0 = deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0
        down_input_scale = torch.empty(
            (E, G // 4, N) if packed_ue8m0 else (E, N, G),
            device=hidden_states_device,
            dtype=torch.int32 if packed_ue8m0 else torch.float32,
        )
        silu_and_mul_masked_post_quant(
            gateup_output,
            down_input,
            down_input_scale,
            group_size,
            masked_m,
            scale_ue8m0=packed_ue8m0,
            topk=topk,
            transposed=packed_ue8m0,
            swiglu_limit=swiglu_limit,
            swizzle=swizzle,
        )
        if packed_ue8m0:
            down_input_scale = down_input_scale.transpose(-1, -2)
    else:
        assert (
            swiglu_limit is None
        ), "swiglu_limit (DeepSeek V4) requires SGLANG_OPT_USE_JIT_EP_ACTIVATION=True"
        assert (
            not swizzle
        ), "SGLANG_OPT_FIX_MEGA_MOE_MEMORY requires SGLANG_OPT_USE_JIT_EP_ACTIVATION=True"
        down_input_scale = torch.empty(
            (E, N, G),
            device=hidden_states_device,
            dtype=torch.float32,
        )
        silu_and_mul_masked_post_quant_fwd(
            gateup_output,
            down_input,
            down_input_scale,
            group_size,
            masked_m,
            scale_ue8m0=deep_gemm_wrapper.DEEPGEMM_SCALE_UE8M0,
        )
    return down_input, down_input_scale
```
**EN:** This block defines `_varlen_deep_gemm_silu_mul_quant` and contains the main logic for this step. It mainly invokes `torch.empty`, `envs.SGLANG_OPT_USE_JIT_EP_ACTIVATION.get`, `sglang_per_token_group_quant_8bit`, `silu_and_mul_masked_post_quant`, and `silu_and_mul_masked_post_quant_fwd`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states_device`, `E`, `N`, `D_2`, and `D` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_varlen_deep_gemm_silu_mul_quant`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`envs.SGLANG_OPT_USE_JIT_EP_ACTIVATION.get`、`sglang_per_token_group_quant_8bit`、`silu_and_mul_masked_post_quant` 以及 `silu_and_mul_masked_post_quant_fwd`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states_device`、`E`、`N`、`D_2` 以及 `D` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 911-928: Internal helper `_apply_swiglu_limit`
```python
def _apply_swiglu_limit(
    gateup_output: torch.Tensor, swiglu_limit: float
) -> torch.Tensor:
    assert swiglu_limit == 10

    num_tokens, hidden_size_x2 = gateup_output.shape
    assert gateup_output.dtype == torch.bfloat16

    gate, up = torch.chunk(gateup_output, chunks=2, dim=-1)
    assert gate.shape == (num_tokens, hidden_size_x2 // 2)
    assert up.shape == (num_tokens, hidden_size_x2 // 2)

    up = torch.clamp(up, min=-swiglu_limit, max=swiglu_limit)
    gate = torch.clamp(gate, max=swiglu_limit)

    out = torch.cat([gate, up], dim=-1)
    assert out.shape == (num_tokens, hidden_size_x2)
    return out
```
**EN:** This block defines `_apply_swiglu_limit` and contains the main logic for this step. It mainly invokes `torch.chunk`, `torch.clamp`, and `torch.cat`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens`, `hidden_size_x2`, `gate`, `up`, and `out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_apply_swiglu_limit`，并承载这一阶段的核心逻辑。 它主要调用 `torch.chunk`、`torch.clamp` 以及 `torch.cat`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens`、`hidden_size_x2`、`gate`、`up` 以及 `out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `_cast_to_e8m0_with_rounding_up`, `copy_list_to_gpu_no_ce`, `DeepGemmRunnerInput`, `DeepGemmRunnerOutput`, and `DeepGemmMoeQuantInfo`. / **主要符号**：核心入口包括 `_cast_to_e8m0_with_rounding_up`、`copy_list_to_gpu_no_ce`、`DeepGemmRunnerInput`、`DeepGemmRunnerOutput` 以及 `DeepGemmMoeQuantInfo`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.List`, `typing.Optional`, and `typing.Tuple` / **标准库**：`__future__.annotations`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.List`、`typing.Optional` 以及 `typing.Tuple`
- **Third-party**: `einops`, `torch`, and `sgl_kernel.elementwise.copy_to_gpu_no_ce` / **第三方依赖**：`einops`、`torch` 以及 `sgl_kernel.elementwise.copy_to_gpu_no_ce`
- **Internal SGLang modules**: `sglang.jit_kernel.deepseek_v4.silu_and_mul_masked_post_quant`, `sglang.srt.environ.envs`, `sglang.srt.layers.deep_gemm_wrapper`, `sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerCore`, `sglang.srt.layers.moe.moe_runner.base.RunnerInput`, `sglang.srt.layers.moe.moe_runner.base.RunnerOutput`, `sglang.srt.layers.moe.moe_runner.base.register_post_permute`, `sglang.srt.layers.moe.moe_runner.base.register_pre_permute`, `sglang.srt.layers.moe.utils.MoeRunnerBackend`, and `sglang.srt.utils.ceil_div` / **SGLang 内部模块**：`sglang.jit_kernel.deepseek_v4.silu_and_mul_masked_post_quant`、`sglang.srt.environ.envs`、`sglang.srt.layers.deep_gemm_wrapper`、`sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerCore`、`sglang.srt.layers.moe.moe_runner.base.RunnerInput`、`sglang.srt.layers.moe.moe_runner.base.RunnerOutput`、`sglang.srt.layers.moe.moe_runner.base.register_post_permute`、`sglang.srt.layers.moe.moe_runner.base.register_pre_permute`、`sglang.srt.layers.moe.utils.MoeRunnerBackend` 以及 `sglang.srt.utils.ceil_div`
