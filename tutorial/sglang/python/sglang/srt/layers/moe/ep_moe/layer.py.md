# layer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/ep_moe/layer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `DeepEPMoE`, `NpuFuseEPMoE`, and `get_moe_impl_class` and connects them to backend-specific paths such as `CUDA`, `Triton`, `CUTLASS`, and `FlashInfer`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `DeepEPMoE`、`NpuFuseEPMoE` 以及 `get_moe_impl_class` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`CUTLASS` 以及 `FlashInfer` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-47: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING, Any, Dict, Optional, Union

import torch

from sglang.srt.compilation.piecewise_context_manager import is_in_piecewise_cuda_graph
from sglang.srt.environ import envs
from sglang.srt.hardware_backend.npu.utils import FusedMoEMode, npu_format_cast
from sglang.srt.layers import deep_gemm_wrapper
from sglang.srt.layers.moe import (
    get_deepep_mode,
    get_moe_a2a_backend,
    get_moe_runner_backend,
)
from sglang.srt.layers.moe.fused_moe_triton.layer import (
    FusedMoE,
    moe_forward_piecewise_cuda_graph_impl,
)
from sglang.srt.layers.moe.token_dispatcher.deepep import (
    DeepEPLLCombineInput,
    DeepEPNormalCombineInput,
)
from sglang.srt.layers.moe.topk import TopKOutput, TopKOutputChecker
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.quantization.fp8 import Fp8Config
from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz
from sglang.srt.layers.quantization.w4afp8 import W4AFp8Config, W4AFp8MoEMethod
from sglang.srt.utils import get_bool_env_var, is_hip, is_npu

if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        DeepEPLLDispatchOutput,
        DeepEPNormalDispatchOutput,
        DispatchOutput,
    )

_is_hip = is_hip()
_is_npu = is_npu()
_is_fp8_fnuz = is_fp8_fnuz()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip


logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.Dict`, and `typing.Optional`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_hip`, `_is_npu`, `_is_fp8_fnuz`, `_use_aiter`, and `logger` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.Dict` 以及 `typing.Optional`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_hip`、`_is_npu`、`_is_fp8_fnuz`、`_use_aiter` 以及 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 48-55: Class `DeepEPMoE` declaration and shared state
```python
class DeepEPMoE(FusedMoE):
    """
    MoE Expert Parallel Impl based on DeepEP (https://github.com/deepseek-ai/DeepEP/tree/main)
    Mooncake EP shares the same class, as they expose the same interface.
    """

    _has_printed = False
```
**EN:** This block introduces class `DeepEPMoE` and the state shared by its methods. It inherits from `FusedMoE`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: MoE Expert Parallel Impl based on DeepEP (https://github.com/deepseek-ai/DeepEP/tree/main) Mooncake EP shares the same class, as they expose the same interface.
**CN:** 该代码块引入类 `DeepEPMoE`，并定义其方法共享的状态。 它继承自 `FusedMoE`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 56-148: `DeepEPMoE` initialization and state setup
```python
    def __init__(
        self,
        num_experts: int,
        top_k: int,
        hidden_size: int,
        intermediate_size: int,
        layer_id: int,
        num_fused_shared_experts: int = 0,
        params_dtype: Optional[torch.dtype] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        activation: str = "silu",
        routed_scaling_factor: Optional[float] = None,
        **kwargs,
    ):
        super().__init__(
            num_experts=num_experts,
            top_k=top_k,
            hidden_size=hidden_size,
            intermediate_size=intermediate_size,
            layer_id=layer_id,
            num_fused_shared_experts=num_fused_shared_experts,
            params_dtype=params_dtype,
            quant_config=quant_config,
            prefix=prefix,
            activation=activation,
            routed_scaling_factor=routed_scaling_factor,
            **kwargs,
        )
        if _use_aiter:
            self.deprecate_flag = True
        elif _is_npu:
            self.deprecate_flag = False
        elif deep_gemm_wrapper.ENABLE_JIT_DEEPGEMM and isinstance(
            quant_config, Fp8Config
        ):
            self.deprecate_flag = True
        elif (
            deep_gemm_wrapper.ENABLE_JIT_DEEPGEMM
            and envs.SGLANG_DEEPEP_BF16_DISPATCH.get()
        ):
            self.deprecate_flag = True
        elif (
            get_moe_runner_backend().is_flashinfer_cutedsl()
            and quant_config is not None
            and quant_config.get_name() == "modelopt_fp4"
        ):
            self.deprecate_flag = True
        elif (
            quant_config is None
            and self.w13_weight.dtype == torch.bfloat16
            and get_moe_runner_backend().is_deep_gemm()
            and get_moe_a2a_backend().is_deepep()
            and get_deepep_mode().enable_low_latency()
            and not _is_npu
            and not _is_hip
        ):
            assert (
                deep_gemm_wrapper.ENABLE_JIT_DEEPGEMM
            ), "Unquantized DeepEP low-latency MoE requires DeepGEMM BF16"
            self.deprecate_flag = True
        else:
            self.deprecate_flag = False

        if self.deprecate_flag:
            return

        if isinstance(quant_config, Fp8Config):
            self.use_block_quant = getattr(self.quant_method, "block_quant", False)
            self.use_fp8_w8a8 = True
            self.fp8_dtype = torch.float8_e4m3fn
            self.use_w4afp8 = False
        elif isinstance(quant_config, W4AFp8Config):
            self.use_w4afp8 = True
            self.use_fp8_w8a8 = False
            self.use_block_quant = False
        else:
            self.use_w4afp8 = False
            self.use_fp8_w8a8 = False
            self.use_block_quant = False

        self.deepep_mode = get_deepep_mode()
        if (
            self.deepep_mode.enable_low_latency()
            and not _is_npu
            and not _is_hip
            and quant_config is not None
        ):
            # AMD HIP and NPU support low_latency DeepEP without DeepGEMM.
            assert (
                deep_gemm_wrapper.ENABLE_JIT_DEEPGEMM
            ), f"DeepEP {self.deepep_mode} mode requires deep_gemm"
```
**EN:** This block defines `DeepEPMoE.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `isinstance`, `get_deepep_mode`, `getattr`, and `self.deepep_mode.enable_low_latency`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.deepep_mode`, `self.deprecate_flag`, `self.use_block_quant`, `self.use_fp8_w8a8`, and `self.fp8_dtype` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `DeepEPMoE.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`isinstance`、`get_deepep_mode`、`getattr` 以及 `self.deepep_mode.enable_low_latency`，说明该流程会编排底层辅助函数或计算内核。 像 `self.deepep_mode`、`self.deprecate_flag`、`self.use_block_quant`、`self.use_fp8_w8a8` 以及 `self.fp8_dtype` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 149-167: `DeepEPMoE.forward` main forward path
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        if is_in_piecewise_cuda_graph():
            assert TopKOutputChecker.format_is_standard(
                topk_output
            ), "Only standard topk output is supported for piecewise cuda graph"
            return moe_forward_piecewise_cuda_graph_impl(
                hidden_states,
                topk_output.topk_weights,
                topk_output.topk_ids,
                topk_output.router_logits,
                self.layer_id,
            )
        else:
            return self.forward_impl(hidden_states, topk_output)
```
**EN:** This block defines `DeepEPMoE.forward` and contains the main logic for this step. It mainly invokes `is_in_piecewise_cuda_graph`, `TopKOutputChecker.format_is_standard`, `moe_forward_piecewise_cuda_graph_impl`, and `self.forward_impl`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMoE.forward`，并承载这一阶段的核心逻辑。 它主要调用 `is_in_piecewise_cuda_graph`、`TopKOutputChecker.format_is_standard`、`moe_forward_piecewise_cuda_graph_impl` 以及 `self.forward_impl`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 168-185: `DeepEPMoE.forward_impl` IMPL execution path
```python
    def forward_impl(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):

        if self.deprecate_flag:
            return super().forward_impl(
                hidden_states,
                topk_output,
            )

        dispatch_output = self.dispatcher.dispatch(
            hidden_states=hidden_states, topk_output=topk_output
        )
        combine_input = self.run_moe_core(dispatch_output)
        return self.dispatcher.combine(combine_input=combine_input)
```
**EN:** This block defines `DeepEPMoE.forward_impl` and contains the main logic for this step. It mainly invokes `self.dispatcher.dispatch`, `self.run_moe_core`, `self.dispatcher.combine`, and `super.forward_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `dispatch_output` and `combine_input` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMoE.forward_impl`，并承载这一阶段的核心逻辑。 它主要调用 `self.dispatcher.dispatch`、`self.run_moe_core`、`self.dispatcher.combine` 以及 `super.forward_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `dispatch_output` 和 `combine_input` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 186-195: Function `DeepEPMoE.dispatch` and its core logic
```python
    def dispatch(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        return self.dispatcher.dispatch(
            hidden_states=hidden_states,
            topk_output=topk_output,
        )
```
**EN:** This block defines `DeepEPMoE.dispatch` and contains the main logic for this step. It mainly invokes `self.dispatcher.dispatch`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMoE.dispatch`，并承载这一阶段的核心逻辑。 它主要调用 `self.dispatcher.dispatch`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 196-235: Function `DeepEPMoE.run_moe_core` and its core logic
```python
    def run_moe_core(
        self,
        dispatch_output: DispatchOutput,
    ):

        if self.deprecate_flag:
            return super().run_moe_core(dispatch_output)

        from sglang.srt.layers.moe.token_dispatcher import DispatchOutputChecker

        if _is_npu:
            assert DispatchOutputChecker.format_is_deepep(dispatch_output)
            output = self.forward_npu(dispatch_output)
        elif DispatchOutputChecker.format_is_deepep_normal(dispatch_output):
            if self.quant_config is None:
                raise NotImplementedError(
                    "Unquantized DeepEP MoE currently supports low_latency mode only"
                )
            elif self.use_w4afp8:
                output = self.forward_cutlass_w4afp8(dispatch_output)
            else:
                assert False, "forward_deepgemm_contiguous is deprecated"
        elif DispatchOutputChecker.format_is_deepep_ll(dispatch_output):
            if self.use_w4afp8:
                output = self.forward_cutlass_w4afp8_masked(dispatch_output)
            else:
                assert False, "forward_deepgemm_masked is deprecated"

        combine_input_wrapper = (
            DeepEPNormalCombineInput
            if DispatchOutputChecker.format_is_deepep_normal(dispatch_output)
            else DeepEPLLCombineInput
        )

        return combine_input_wrapper(
            hidden_states=output,
            topk_ids=dispatch_output.topk_ids,
            topk_weights=dispatch_output.topk_weights,
        )
```
**EN:** This block defines `DeepEPMoE.run_moe_core` and contains the main logic for this step. It mainly invokes `combine_input_wrapper`, `super.run_moe_core`, `DispatchOutputChecker.format_is_deepep`, `self.forward_npu`, and `DispatchOutputChecker.format_is_deepep_normal`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `combine_input_wrapper` and `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMoE.run_moe_core`，并承载这一阶段的核心逻辑。 它主要调用 `combine_input_wrapper`、`super.run_moe_core`、`DispatchOutputChecker.format_is_deepep`、`self.forward_npu` 以及 `DispatchOutputChecker.format_is_deepep_normal`，说明该流程会编排底层辅助函数或计算内核。 像 `combine_input_wrapper` 和 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 236-249: Function `DeepEPMoE.combine` and its core logic
```python
    def combine(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
        overlap_args: Optional[Dict[str, Any]] = None,
    ):
        return self.dispatcher.combine(
            hidden_states=hidden_states,
            topk_ids=topk_ids,
            topk_weights=topk_weights,
            overlap_args=overlap_args,
        )
```
**EN:** This block defines `DeepEPMoE.combine` and contains the main logic for this step. It mainly invokes `self.dispatcher.combine`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMoE.combine`，并承载这一阶段的核心逻辑。 它主要调用 `self.dispatcher.combine`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 250-260: `DeepEPMoE.forward_cutlass_w4afp8` CUTLASS_W4AFP8 execution path
```python
    def forward_cutlass_w4afp8(
        self,
        dispatch_output: DeepEPNormalDispatchOutput,
    ):
        assert self.moe_runner_config.activation == "silu"
        assert isinstance(self.quant_method, W4AFp8MoEMethod)
        return self.quant_method.apply_deepep_normal(
            layer=self,
            dispatch_output=dispatch_output,
        )
```
**EN:** This block defines `DeepEPMoE.forward_cutlass_w4afp8` and contains the main logic for this step. It mainly invokes `isinstance` and `self.quant_method.apply_deepep_normal`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMoE.forward_cutlass_w4afp8`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance` 和 `self.quant_method.apply_deepep_normal`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 261-271: `DeepEPMoE.forward_cutlass_w4afp8_masked` CUTLASS_W4AFP8_MASKED execution path
```python
    def forward_cutlass_w4afp8_masked(
        self,
        dispatch_output: DeepEPLLDispatchOutput,
    ):
        assert self.moe_runner_config.activation == "silu"
        assert isinstance(self.quant_method, W4AFp8MoEMethod)
        return self.quant_method.apply_deepep_ll(
            layer=self,
            dispatch_output=dispatch_output,
        )
```
**EN:** This block defines `DeepEPMoE.forward_cutlass_w4afp8_masked` and contains the main logic for this step. It mainly invokes `isinstance` and `self.quant_method.apply_deepep_ll`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMoE.forward_cutlass_w4afp8_masked`，并承载这一阶段的核心逻辑。 它主要调用 `isinstance` 和 `self.quant_method.apply_deepep_ll`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 272-346: `DeepEPMoE.forward_npu` NPU execution path
```python
    def forward_npu(
        self,
        dispatch_output: Union[DeepEPNormalDispatchOutput, DeepEPLLDispatchOutput],
    ):
        assert self.quant_method is not None
        assert self.moe_runner_config.activation == "silu"

        from sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu import (
            npu_fused_moe_without_routing_weights_bf16,
        )
        from sglang.srt.layers.moe.token_dispatcher import DispatchOutputChecker

        # NOTE: Ascend's Dispatch & Combine does not support FP16
        output_dtype = torch.bfloat16
        group_list_type = 1

        if DispatchOutputChecker.format_is_deepep_normal(dispatch_output):
            if TYPE_CHECKING:
                assert isinstance(dispatch_output, DeepEPNormalDispatchOutput)
            hidden_states, hidden_states_scale, _, _, num_recv_tokens_per_expert = (
                dispatch_output
            )

            group_list = torch.tensor(
                num_recv_tokens_per_expert,
                dtype=torch.int64,
                device=hidden_states.device,
            )

            if self.w13_weight.dtype == torch.bfloat16:
                hidden_states = npu_fused_moe_without_routing_weights_bf16(
                    self, hidden_states, group_list_type, group_list, output_dtype
                )
            else:
                hidden_states = self.quant_method.apply_without_routing_weights(
                    self,
                    hidden_states,
                    hidden_states_scale,
                    group_list_type,
                    group_list,
                    output_dtype,
                )
        elif DispatchOutputChecker.format_is_deepep_ll(dispatch_output):
            if TYPE_CHECKING:
                assert isinstance(dispatch_output, DeepEPLLDispatchOutput)
            (
                hidden_states,
                hidden_states_scale,
                topk_ids,
                topk_weights,
                group_list,
                _,
            ) = dispatch_output

            group_list = group_list.to(torch.int64)

            if self.w13_weight.dtype == torch.bfloat16:
                hidden_states = npu_fused_moe_without_routing_weights_bf16(
                    self, hidden_states, group_list_type, group_list, output_dtype
                )
            else:
                hidden_states = self.quant_method.apply_without_routing_weights(
                    self,
                    hidden_states,
                    hidden_states_scale,
                    group_list_type,
                    group_list,
                    output_dtype,
                )
        else:
            raise ValueError(f"Not Supported DeepEP format {dispatch_output.format}")

        return hidden_states
```
**EN:** This block defines `DeepEPMoE.forward_npu` and contains the main logic for this step. It mainly invokes `DispatchOutputChecker.format_is_deepep_normal`, `torch.tensor`, `DispatchOutputChecker.format_is_deepep_ll`, `isinstance`, and `npu_fused_moe_without_routing_weights_bf16`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output_dtype`, `group_list_type`, `hidden_states`, `hidden_states_scale`, and `_` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `DeepEPMoE.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `DispatchOutputChecker.format_is_deepep_normal`、`torch.tensor`、`DispatchOutputChecker.format_is_deepep_ll`、`isinstance` 以及 `npu_fused_moe_without_routing_weights_bf16`，说明该流程会编排底层辅助函数或计算内核。 像 `output_dtype`、`group_list_type`、`hidden_states`、`hidden_states_scale` 以及 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 347-347: Class `NpuFuseEPMoE` declaration and shared state
```python
class NpuFuseEPMoE(DeepEPMoE):
```
**EN:** This block introduces class `NpuFuseEPMoE` and the state shared by its methods. It inherits from `DeepEPMoE`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `NpuFuseEPMoE`，并定义其方法共享的状态。 它继承自 `DeepEPMoE`，说明了它在 SRT 层栈中的接入方式。

### Lines 348-381: `NpuFuseEPMoE` initialization and state setup
```python
    def __init__(
        self,
        num_experts: int,
        top_k: int,
        hidden_size: int,
        intermediate_size: int,
        layer_id: int,
        num_fused_shared_experts: int = 0,
        params_dtype: Optional[torch.dtype] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        activation: str = "silu",
        routed_scaling_factor: Optional[float] = None,
        **kwargs,
    ):
        super().__init__(
            num_experts=num_experts,
            top_k=top_k,
            hidden_size=hidden_size,
            intermediate_size=intermediate_size,
            layer_id=layer_id,
            num_fused_shared_experts=num_fused_shared_experts,
            params_dtype=params_dtype,
            quant_config=quant_config,
            prefix=prefix,
            activation=activation,
            routed_scaling_factor=routed_scaling_factor,
            **kwargs,
        )

        self.quant_method.process_weights_after_loading = (
            self._process_weights_after_loading
        )
```
**EN:** This block defines `NpuFuseEPMoE.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `NpuFuseEPMoE.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。

### Lines 382-398: `NpuFuseEPMoE.forward` main forward path
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
        forward_shared_experts=None,
        alt_stream=None,
        disable_sbo=False,
    ):
        return self.dispatcher.dispatch(
            hidden_states=hidden_states,
            topk_output=topk_output,
            gmm1_permuted_weight=self.w13_weight,
            gmm1_permuted_weight_scale=self.w13_weight_scale,
            gmm2_weight=self.w2_weight,
            gmm2_weight_scale=self.w2_weight_scale,
        ).hidden_state
```
**EN:** This block defines `NpuFuseEPMoE.forward` and contains the main logic for this step. It mainly invokes `self.dispatcher.dispatch`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NpuFuseEPMoE.forward`，并承载这一阶段的核心逻辑。 它主要调用 `self.dispatcher.dispatch`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 399-414: Function `NpuFuseEPMoE.permute_w13_weight_scale` and its core logic
```python
    def permute_w13_weight_scale(self, w: torch.Tensor, tile_n: int):
        if tile_n % 2 != 0:
            raise ValueError(f"tile_n must be even, got {tile_n}")

        *dims, n = w.shape
        if n % tile_n != 0:
            raise ValueError(f"Last dimension {n} must be divisible by tile_n {tile_n}")

        w_reshaped = w.reshape(*dims, 2, n // tile_n, tile_n // 2)

        # Permute the last two dimensions.
        perm_order = list(range(len(dims))) + [-2, -3, -1]
        w_permuted = w_reshaped.permute(perm_order)

        return w_permuted.reshape(*dims, n)
```
**EN:** This block defines `NpuFuseEPMoE.permute_w13_weight_scale` and contains the main logic for this step. It mainly invokes `w.reshape`, `w_reshaped.permute`, `w_permuted.reshape`, `ValueError`, and `list`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `n`, `w_reshaped`, `perm_order`, and `w_permuted` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NpuFuseEPMoE.permute_w13_weight_scale`，并承载这一阶段的核心逻辑。 它主要调用 `w.reshape`、`w_reshaped.permute`、`w_permuted.reshape`、`ValueError` 以及 `list`，说明该流程会编排底层辅助函数或计算内核。 像 `n`、`w_reshaped`、`perm_order` 以及 `w_permuted` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 415-438: Function `NpuFuseEPMoE.reshape_w13_weight` and its core logic
```python
    def reshape_w13_weight(self, weight: torch.Tensor, dim: int, chunk_size: int = 64):
        # Achieving greater computing power through reshape on Ascend.
        original_shape = weight.shape
        if dim < 0:
            dim += len(original_shape)

        if original_shape[dim] % (2 * chunk_size) != 0:
            raise ValueError(
                f"Dimension {dim} size {original_shape[dim]} must be divisible by {2 * chunk_size}"
            )

        new_shape = (
            *original_shape[:dim],
            2,
            original_shape[dim] // (2 * chunk_size),
            chunk_size,
            *original_shape[dim + 1 :],
        )

        weight = weight.view(new_shape)
        weight = weight.transpose(dim, dim + 1).contiguous()

        return weight.view(*original_shape[:dim], -1, *original_shape[dim + 1 :])
```
**EN:** This block defines `NpuFuseEPMoE.reshape_w13_weight` and contains the main logic for this step. It mainly invokes `weight.view`, `weight.transpose.contiguous`, `len`, `ValueError`, and `weight.transpose`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `original_shape`, `new_shape`, `weight`, and `dim` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NpuFuseEPMoE.reshape_w13_weight`，并承载这一阶段的核心逻辑。 它主要调用 `weight.view`、`weight.transpose.contiguous`、`len`、`ValueError` 以及 `weight.transpose`，说明该流程会编排底层辅助函数或计算内核。 像 `original_shape`、`new_shape`、`weight` 以及 `dim` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 439-445: Function `NpuFuseEPMoE.release_weight_cache` and its core logic
```python
    def release_weight_cache(self, weight: torch.Tensor):
        # .contiguous() introduces additional memory overhead and needs to be released using resize_(0)
        origin_weight = weight.data.transpose(1, 2)
        new_weight = origin_weight.contiguous()
        origin_weight.untyped_storage().resize_(0)
        return new_weight
```
**EN:** This block defines `NpuFuseEPMoE.release_weight_cache` and contains the main logic for this step. It mainly invokes `weight.data.transpose`, `origin_weight.contiguous`, `origin_weight.untyped_storage.resize_`, and `origin_weight.untyped_storage`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `origin_weight` and `new_weight` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NpuFuseEPMoE.release_weight_cache`，并承载这一阶段的核心逻辑。 它主要调用 `weight.data.transpose`、`origin_weight.contiguous`、`origin_weight.untyped_storage.resize_` 以及 `origin_weight.untyped_storage`，说明该流程会编排底层辅助函数或计算内核。 像 `origin_weight` 和 `new_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 446-455: Function `NpuFuseEPMoE.scale_from_float_to_int64` and its core logic
```python
    def scale_from_float_to_int64(self, scale):
        import numpy as np

        scale = torch.from_numpy(
            np.frombuffer(
                scale.cpu().to(torch.float32).numpy().tobytes(), dtype=np.int32
            ).astype(np.int64)
        ).to(scale.device)
        return torch.nn.Parameter(scale, requires_grad=False)
```
**EN:** This block defines `NpuFuseEPMoE.scale_from_float_to_int64` and contains the main logic for this step. It mainly invokes `torch.from_numpy.to`, `torch.nn.Parameter`, `torch.from_numpy`, `np.frombuffer.astype`, and `np.frombuffer`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `scale` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NpuFuseEPMoE.scale_from_float_to_int64`，并承载这一阶段的核心逻辑。 它主要调用 `torch.from_numpy.to`、`torch.nn.Parameter`、`torch.from_numpy`、`np.frombuffer.astype` 以及 `np.frombuffer`，说明该流程会编排底层辅助函数或计算内核。 像 `scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 456-507: Internal helper `NpuFuseEPMoE._process_weights_after_loading`
```python
    def _process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if (
            envs.SGLANG_NPU_FUSED_MOE_MODE.get()
            == FusedMoEMode.DISPATCH_FFN_COMBINE.value
        ):
            w13_weight = self.release_weight_cache(layer.w13_weight)
            layer.w13_weight.data = npu_format_cast(w13_weight)
            w2_weight = self.release_weight_cache(layer.w2_weight)
            layer.w2_weight.data = npu_format_cast(w2_weight)

            layer.w13_weight_scale.data = layer.w13_weight_scale.data.view(
                layer.w13_weight_scale.data.shape[0], -1
            )
            w2_scale = layer.w2_weight_scale.data.squeeze(-1).contiguous()
            layer.w2_weight_scale = torch.nn.Parameter(
                w2_scale.to(torch.float32), requires_grad=False
            )

            layer.w13_weight_scale = self.scale_from_float_to_int64(
                layer.w13_weight_scale.data
            )
            layer.w2_weight_scale = self.scale_from_float_to_int64(
                layer.w2_weight_scale.data
            )
        else:
            cpu_w13 = layer.w13_weight.data.transpose(1, 2).cpu()
            layer.w13_weight.data = self.reshape_w13_weight(cpu_w13, -1).npu()
            w13_scale = layer.w13_weight_scale.data.squeeze(-1).contiguous()
            w13_scale = self.permute_w13_weight_scale(w13_scale, 128)
            layer.w13_weight_scale = torch.nn.Parameter(
                w13_scale.to(torch.float32), requires_grad=False
            )
            layer.w13_weight.data = npu_format_cast(layer.w13_weight.data)
            layer.w2_weight.data = npu_format_cast(layer.w2_weight.data)

            w2_scale = layer.w2_weight_scale.data.squeeze(-1).contiguous()
            layer.w2_weight_scale = torch.nn.Parameter(
                w2_scale.to(torch.float32), requires_grad=False
            )

        if hasattr(layer, "w13_weight_offset"):
            layer.w13_weight_offset = torch.nn.Parameter(
                layer.w13_weight_offset.data.squeeze(-1).contiguous(),
                requires_grad=False,
            )
        if hasattr(layer, "w2_weight_offset"):
            layer.w2_weight_offset = torch.nn.Parameter(
                layer.w2_weight_offset.data.squeeze(-1).contiguous(),
                requires_grad=False,
            )
```
**EN:** This block defines `NpuFuseEPMoE._process_weights_after_loading` and contains the main logic for this step. It mainly invokes `hasattr`, `envs.SGLANG_NPU_FUSED_MOE_MODE.get`, `self.release_weight_cache`, `npu_format_cast`, and `layer.w13_weight_scale.data.view`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `w13_weight`, `w2_weight`, `w2_scale`, `cpu_w13`, and `w13_scale` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `NpuFuseEPMoE._process_weights_after_loading`，并承载这一阶段的核心逻辑。 它主要调用 `hasattr`、`envs.SGLANG_NPU_FUSED_MOE_MODE.get`、`self.release_weight_cache`、`npu_format_cast` 以及 `layer.w13_weight_scale.data.view`，说明该流程会编排底层辅助函数或计算内核。 像 `w13_weight`、`w2_weight`、`w2_scale`、`cpu_w13` 以及 `w13_scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 508-520: `get_moe_impl_class` getter for Mixture-of-Experts impl class
```python
def get_moe_impl_class(quant_config: Optional[QuantizationConfig]):
    # [TODO] kk, temporary solution
    if (
        get_moe_a2a_backend().is_mori()
        or get_moe_a2a_backend().is_deepep()
        or get_moe_a2a_backend().is_mooncake()
        or get_moe_a2a_backend().is_nixl()
    ):
        return DeepEPMoE
    if get_moe_a2a_backend().is_ascend_fuseep():
        return NpuFuseEPMoE

    return FusedMoE
```
**EN:** This block defines `get_moe_impl_class` and contains the main logic for this step. It mainly invokes `get_moe_a2a_backend.is_ascend_fuseep`, `get_moe_a2a_backend.is_mori`, `get_moe_a2a_backend.is_deepep`, `get_moe_a2a_backend.is_mooncake`, and `get_moe_a2a_backend.is_nixl`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_moe_impl_class`，并承载这一阶段的核心逻辑。 它主要调用 `get_moe_a2a_backend.is_ascend_fuseep`、`get_moe_a2a_backend.is_mori`、`get_moe_a2a_backend.is_deepep`、`get_moe_a2a_backend.is_mooncake` 以及 `get_moe_a2a_backend.is_nixl`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `DeepEPMoE`, `NpuFuseEPMoE`, and `get_moe_impl_class`. / **主要符号**：核心入口包括 `DeepEPMoE`、`NpuFuseEPMoE` 以及 `get_moe_impl_class`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `typing.TYPE_CHECKING`, `typing.Any`, `typing.Dict`, `typing.Optional`, and `typing.Union` / **标准库**：`__future__.annotations`、`logging`、`typing.TYPE_CHECKING`、`typing.Any`、`typing.Dict`、`typing.Optional` 以及 `typing.Union`
- **Third-party**: `torch` and `numpy` / **第三方依赖**：`torch` 和 `numpy`
- **Internal SGLang modules**: `sglang.srt.compilation.piecewise_context_manager.is_in_piecewise_cuda_graph`, `sglang.srt.environ.envs`, `sglang.srt.hardware_backend.npu.utils.FusedMoEMode`, `sglang.srt.hardware_backend.npu.utils.npu_format_cast`, `sglang.srt.layers.deep_gemm_wrapper`, `sglang.srt.layers.moe.get_deepep_mode`, `sglang.srt.layers.moe.get_moe_a2a_backend`, `sglang.srt.layers.moe.get_moe_runner_backend`, `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`, `sglang.srt.layers.moe.fused_moe_triton.layer.moe_forward_piecewise_cuda_graph_impl`, `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLCombineInput`, and `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPNormalCombineInput` / **SGLang 内部模块**：`sglang.srt.compilation.piecewise_context_manager.is_in_piecewise_cuda_graph`、`sglang.srt.environ.envs`、`sglang.srt.hardware_backend.npu.utils.FusedMoEMode`、`sglang.srt.hardware_backend.npu.utils.npu_format_cast`、`sglang.srt.layers.deep_gemm_wrapper`、`sglang.srt.layers.moe.get_deepep_mode`、`sglang.srt.layers.moe.get_moe_a2a_backend`、`sglang.srt.layers.moe.get_moe_runner_backend`、`sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`、`sglang.srt.layers.moe.fused_moe_triton.layer.moe_forward_piecewise_cuda_graph_impl`、`sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLCombineInput` 以及 `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPNormalCombineInput`
