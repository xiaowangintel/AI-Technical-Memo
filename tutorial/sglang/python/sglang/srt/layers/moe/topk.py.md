# topk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/topk.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `TopKConfig`, `TopKOutputChecker`, `TopKOutputFormat`, and `TopKOutput` and connects them to backend-specific paths such as `CUDA`, `Triton`, `FlashInfer`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `TopKConfig`、`TopKOutputChecker`、`TopKOutputFormat` 以及 `TopKOutput` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`FlashInfer` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: File header and module overview
```python
# Copyright 2024 SGLang Team
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

### Lines 15-197: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import logging
import math
from dataclasses import dataclass
from enum import IntEnum, auto
from typing import (
    TYPE_CHECKING,
    Callable,
    NamedTuple,
    Optional,
    Protocol,
    TypeGuard,
    runtime_checkable,
)

import torch
import torch.nn.functional as F

try:
    from triton_kernels.matmul_ogs import GatherIndx, RoutingData, ScatterIndx
    from triton_kernels.tensor import make_ragged_tensor_metadata
    from triton_kernels.topk import topk as triton_kernels_topk

    def routing(
        logits,
        n_expts_act,
        sm_first=False,
        expt_indx=None,
        simulated_ep=1,
        n_rows=None,
    ):
        if simulated_ep != 1:
            raise NotImplementedError(
                "simulated_ep routing is not supported with triton_kernels 3.6.0"
            )

        if sm_first:
            logits = torch.softmax(logits, dim=-1)

        sparse_logits = triton_kernels_topk(
            logits,
            n_expts_act,
            apply_softmax=not sm_first,
            y_indx=expt_indx,
            n_rows=n_rows,
        )
        dispatch_indx = sparse_logits.mask_metadata.row_sorted_indx
        combine_indx = sparse_logits.mask_metadata.col_sorted_indx
        ragged_metadata = make_ragged_tensor_metadata(
            sparse_logits.mask_metadata.col_sum, dispatch_indx.shape[0]
        )
        gate_scal = sparse_logits.vals.flatten()[combine_indx]
        routing_data = RoutingData(
            gate_scal,
            ragged_metadata.slice_sizes,
            logits.shape[-1],
            n_expts_act,
            ragged_metadata,
        )
        gather_indx = GatherIndx(combine_indx, dispatch_indx)
        scatter_indx = ScatterIndx(dispatch_indx, combine_indx)
        return routing_data, gather_indx, scatter_indx

except ImportError:
    pass

from sglang.jit_kernel.deepseek_v4 import mask_topk_ids
from sglang.srt.distributed import (
    get_moe_expert_parallel_rank,
    get_moe_expert_parallel_world_size,
    get_tp_group,
)
from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    use_symmetric_memory,
)
from sglang.srt.environ import envs
from sglang.srt.eplb import expert_location_dispatch
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.eplb.expert_location_dispatch import (
    ExpertLocationDispatchInfo,
    topk_ids_logical_to_physical,
)
from sglang.srt.layers.dp_attention import is_allocation_symmetric
from sglang.srt.layers.moe import get_moe_runner_backend
from sglang.srt.layers.moe.utils import is_deepep_class_backend
from sglang.srt.layers.utils import MultiPlatformOp
from sglang.srt.state_capturer.routed_experts import get_global_experts_capturer
from sglang.srt.utils import (
    cpu_has_amx_support,
    get_bool_env_var,
    get_compiler_backend,
    is_cpu,
    is_cuda,
    is_hip,
    is_musa,
    is_npu,
    is_xpu,
)
from sglang.srt.utils.patch_torch import register_fake_if_exists

if TYPE_CHECKING:
    from sglang.srt.layers.quantization import QuantizationConfig


logger = logging.getLogger(__name__)
_is_cuda = is_cuda()
_is_hip = is_hip()
_is_cpu = is_cpu()
_is_cpu_amx_available = cpu_has_amx_support()
_is_xpu = is_xpu()
_is_npu = is_npu()
_is_xpu = is_xpu()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
_is_musa = is_musa()

if _is_cuda:
    from sgl_kernel import moe_fused_gate

    try:
        from flashinfer.fused_moe import fused_topk_deepseek as _fused_topk_deepseek

        from sglang.srt.utils.custom_op import register_custom_op

        @register_custom_op(
            op_name="fused_topk_deepseek",
            mutates_args=["topk_weights", "topk_ids"],
        )
        def fused_topk_deepseek(
            gating_output: torch.Tensor,
            correction_bias: torch.Tensor,
            num_expert_group: int,
            topk_group: int,
            topk: int,
            scaling_factor: float,
            topk_weights: torch.Tensor,
            topk_ids: torch.Tensor,
            renormalize: bool,
        ) -> None:
            _fused_topk_deepseek(
                gating_output,
                correction_bias,
                num_expert_group,
                topk_group,
                topk,
                scaling_factor,
                topk_weights,
                topk_ids,
                renormalize,
            )

    except ImportError:
        fused_topk_deepseek = None

    try:
        from sgl_kernel import kimi_k2_moe_fused_gate
    except ImportError as e:
        pass

if _is_cuda or _is_hip or _is_xpu:
    from sgl_kernel import topk_softmax

    try:
        from sgl_kernel import topk_sigmoid
    except ImportError:
        pass
if _use_aiter:
    try:
        from aiter import biased_grouped_topk as aiter_biased_grouped_topk
        from aiter.fused_moe import fused_topk as aiter_fused_topk
    except ImportError:
        raise ImportError("aiter is required when SGLANG_USE_AITER is set to True")
if _is_musa:
    try:
        from mate import moe_fused_gate
    except ImportError as e:
        raise ImportError("mate is required for the biased grouped topk.")

    from sglang.srt.hardware_backend.musa.kernels.topk import topk_sigmoid, topk_softmax

# -------------------------------- TopKConfig ---------------------------------------
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `math`, `dataclasses.dataclass`, `enum.IntEnum`, and `enum.auto`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `sparse_logits`, `dispatch_indx`, `combine_indx`, `ragged_metadata`, and `gate_scal` capture configuration, cached handles, or feature flags. Control structures like `Try` and `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`math`、`dataclasses.dataclass`、`enum.IntEnum` 以及 `enum.auto`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `sparse_logits`、`dispatch_indx`、`combine_indx`、`ragged_metadata` 以及 `gate_scal` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `Try` 和 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 198-218: Class `TopKConfig` declaration and shared state
```python
@dataclass
class TopKConfig:
    top_k: int
    use_grouped_topk: bool = False
    topk_group: Optional[int] = None
    num_expert_group: Optional[int] = None
    renormalize: bool = True
    num_fused_shared_experts: int = 0
    custom_routing_function: Optional[Callable] = None
    correction_bias: Optional[torch.Tensor] = None
    torch_native: bool = False
    routed_scaling_factor: Optional[float] = None
    apply_routed_scaling_factor_on_output: bool = False
    fused_shared_experts_scaling_factor: Optional[float] = None
    output_format: Optional[TopKOutputFormat] = None
    scoring_func: str = "softmax"


# -------------------------------- TopKOutput ---------------------------------------
```
**EN:** This block introduces class `TopKConfig` and the state shared by its methods.
**CN:** 该代码块引入类 `TopKConfig`，并定义其方法共享的状态。

### Lines 219-220: Class `TopKOutputChecker` declaration and shared state
```python
class TopKOutputChecker:
```
**EN:** This block introduces class `TopKOutputChecker` and the state shared by its methods.
**CN:** 该代码块引入类 `TopKOutputChecker`，并定义其方法共享的状态。

### Lines 221-224: Function `TopKOutputChecker.format_is_standard` and its core logic
```python
    @staticmethod
    def format_is_standard(topk_output: TopKOutput) -> TypeGuard[StandardTopKOutput]:
        return isinstance(topk_output, StandardTopKOutput)
```
**EN:** This block defines `TopKOutputChecker.format_is_standard` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `isinstance`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TopKOutputChecker.format_is_standard`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `isinstance`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 225-230: Function `TopKOutputChecker.format_is_triton_kernels` and its core logic
```python
    @staticmethod
    def format_is_triton_kernels(
        topk_output: TopKOutput,
    ) -> TypeGuard[TritonKernelTopKOutput]:
        return isinstance(topk_output, TritonKernelTopKOutput)
```
**EN:** This block defines `TopKOutputChecker.format_is_triton_kernels` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `isinstance`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TopKOutputChecker.format_is_triton_kernels`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `isinstance`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 231-235: Function `TopKOutputChecker.format_is_bypassed` and its core logic
```python
    @staticmethod
    def format_is_bypassed(topk_output: TopKOutput) -> TypeGuard[BypassedTopKOutput]:
        return isinstance(topk_output, BypassedTopKOutput)
```
**EN:** This block defines `TopKOutputChecker.format_is_bypassed` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `isinstance`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TopKOutputChecker.format_is_bypassed`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `isinstance`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 236-241: Class `TopKOutputFormat` declaration and shared state
```python
class TopKOutputFormat(IntEnum):
    STANDARD = auto()
    TRITON_KERNEL = auto()
    BYPASSED = auto()
```
**EN:** This block introduces class `TopKOutputFormat` and the state shared by its methods. It inherits from `IntEnum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `TopKOutputFormat`，并定义其方法共享的状态。 它继承自 `IntEnum`，说明了它在 SRT 层栈中的接入方式。

### Lines 242-245: Class `TopKOutput` declaration and shared state
```python
@runtime_checkable
class TopKOutput(Protocol):
    """Protocol for top-k outputs in different formats."""
```
**EN:** This block introduces class `TopKOutput` and the state shared by its methods. It inherits from `Protocol`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Protocol for top-k outputs in different formats.
**CN:** 该代码块引入类 `TopKOutput`，并定义其方法共享的状态。 它继承自 `Protocol`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 246-251: Function `TopKOutput.format` and its core logic
```python
    @property
    def format(self) -> TopKOutputFormat:
        """The format of the output."""
        ...
```
**EN:** This block defines `TopKOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks.
**CN:** 该代码块定义了 `TopKOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。

### Lines 252-258: Class `StandardTopKOutput` declaration and shared state
```python
class StandardTopKOutput(NamedTuple):
    """Standard top-k output format."""

    topk_weights: torch.Tensor
    topk_ids: torch.Tensor
    router_logits: torch.Tensor
```
**EN:** This block introduces class `StandardTopKOutput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Standard top-k output format.
**CN:** 该代码块引入类 `StandardTopKOutput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 259-263: Function `StandardTopKOutput.format` and its core logic
```python
    @property
    def format(self) -> TopKOutputFormat:
        return TopKOutputFormat.STANDARD
```
**EN:** This block defines `StandardTopKOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `StandardTopKOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 264-270: Class `TritonKernelTopKOutput` declaration and shared state
```python
class TritonKernelTopKOutput(NamedTuple):
    """Triton kernel top-k output format."""

    routing_data: RoutingData
    gather_indx: GatherIndx
    scatter_indx: ScatterIndx
```
**EN:** This block introduces class `TritonKernelTopKOutput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Triton kernel top-k output format.
**CN:** 该代码块引入类 `TritonKernelTopKOutput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 271-275: Function `TritonKernelTopKOutput.format` and its core logic
```python
    @property
    def format(self) -> TopKOutputFormat:
        return TopKOutputFormat.TRITON_KERNEL
```
**EN:** This block defines `TritonKernelTopKOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TritonKernelTopKOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 276-284: Class `BypassedTopKOutput` declaration and shared state
```python
class BypassedTopKOutput(NamedTuple):
    """Bypassed top-k output format."""

    hidden_states: torch.Tensor
    router_logits: torch.Tensor
    topk_config: TopKConfig
    num_token_non_padded: Optional[torch.Tensor] = None
    expert_location_dispatch_info: Optional[ExpertLocationDispatchInfo] = None
```
**EN:** This block introduces class `BypassedTopKOutput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Bypassed top-k output format.
**CN:** 该代码块引入类 `BypassedTopKOutput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 285-288: Function `BypassedTopKOutput.format` and its core logic
```python
    @property
    def format(self) -> TopKOutputFormat:
        return TopKOutputFormat.BYPASSED
```
**EN:** This block defines `BypassedTopKOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `BypassedTopKOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 289-304: Function `BypassedTopKOutput.to_standard` and its core logic
```python
    def to_standard(self, layer_id: Optional[int] = None) -> "StandardTopKOutput":
        """Materialize routing tensors. Used by MoE kernels that need explicit
        topk_ids / topk_weights rather than doing routing internally."""
        return select_experts(
            hidden_states=self.hidden_states,
            router_logits=self.router_logits,
            topk_config=self.topk_config,
            layer_id=layer_id,
            num_token_non_padded=self.num_token_non_padded,
            expert_location_dispatch_info=self.expert_location_dispatch_info,
        )


# -------------------------------- TopK ---------------------------------------
```
**EN:** This block defines `BypassedTopKOutput.to_standard` and contains the main logic for this step. It mainly invokes `select_experts`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `BypassedTopKOutput.to_standard`，并承载这一阶段的核心逻辑。 它主要调用 `select_experts`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 305-313: Class `TopK` declaration and shared state
```python
class TopK(MultiPlatformOp):
    """
    Parameters:
    --top_k: The all number of top experts selected per token, including the fused shared expert(s).
    --num_fused_shared_experts: num of shared experts, can be activate both in TP or EP mode.
    --routed_scaling_factor: the scaling factor for routed experts in topk_weights.
    --fused_shared_experts_scaling_factor: scaling factor for fused shared experts on AMD-platform.
    """
```
**EN:** This block introduces class `TopK` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Parameters: --top_k: The all number of top experts selected per token, including the fused shared expert(s).
**CN:** 该代码块引入类 `TopK`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 314-379: `TopK` initialization and state setup
```python
    def __init__(
        self,
        top_k: int,
        *,
        layer_id: Optional[int] = None,
        use_grouped_topk: bool = False,
        topk_group: Optional[int] = None,
        num_expert_group: Optional[int] = None,
        renormalize: bool = True,
        num_fused_shared_experts: int = 0,
        custom_routing_function: Optional[Callable] = None,
        scoring_func: str = "softmax",
        correction_bias: Optional[torch.Tensor] = None,
        quant_config: Optional[QuantizationConfig] = None,
        routed_scaling_factor: Optional[float] = None,
        apply_routed_scaling_factor_on_output: Optional[bool] = False,
        output_format: Optional[TopKOutputFormat] = None,
        fused_shared_experts_scaling_factor: Optional[float] = None,
        is_fp4_experts: bool = False,
    ):
        # NOTE: scoring_func is not used for now, but we keep it for future use
        # see https://github.com/sgl-project/sglang/pull/4505 for more details
        super().__init__()

        if use_grouped_topk:
            assert num_expert_group is not None and topk_group is not None

        self.layer_id = layer_id
        if num_fused_shared_experts > 0:
            from sglang.srt.server_args import get_global_server_args

            try:
                self.enable_deepep_waterfill = (
                    get_global_server_args().enable_deepep_waterfill
                )
            except ValueError:
                self.enable_deepep_waterfill = False
        else:
            self.enable_deepep_waterfill = False

        self.deepep_waterfill_balancer = None
        if self.enable_deepep_waterfill:
            # TODO(ch-wan): Refactor shared-expert fusion and routed TopK fusion.
            top_k -= num_fused_shared_experts
            num_fused_shared_experts = 0
            output_format = TopKOutputFormat.STANDARD

        # flashinfer_mxfp4 backend only: True -> STANDARD (Mxfp4FlashinferTrtllmMoEMethod
        # consumes), False -> BYPASSED (flashinfer's own mxfp4 kernel). No-op otherwise.
        self.is_fp4_experts = is_fp4_experts
        self.topk_config = TopKConfig(
            top_k=top_k,
            use_grouped_topk=use_grouped_topk,
            renormalize=renormalize,
            topk_group=topk_group,
            num_expert_group=num_expert_group,
            num_fused_shared_experts=num_fused_shared_experts,
            custom_routing_function=custom_routing_function,
            correction_bias=correction_bias,
            routed_scaling_factor=routed_scaling_factor,
            apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
            fused_shared_experts_scaling_factor=fused_shared_experts_scaling_factor,
            output_format=output_format,
            scoring_func=scoring_func,
        )
```
**EN:** This block defines `TopK.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `TopKConfig`, and `get_global_server_args`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.layer_id`, `self.deepep_waterfill_balancer`, `self.is_fp4_experts`, `self.topk_config`, and `self.enable_deepep_waterfill` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `TopK.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`TopKConfig` 以及 `get_global_server_args`，说明该流程会编排底层辅助函数或计算内核。 像 `self.layer_id`、`self.deepep_waterfill_balancer`、`self.is_fp4_experts`、`self.topk_config` 以及 `self.enable_deepep_waterfill` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 380-391: Internal helper `TopK._apply_deepep_waterfill`
```python
    def _apply_deepep_waterfill(
        self, topk_output: TopKOutput, num_tokens: int
    ) -> TopKOutput:
        if self.enable_deepep_waterfill and self.deepep_waterfill_balancer is None:
            raise RuntimeError(
                "DeepEP waterfill TopK must be prepared by ModelRunner before forward."
            )
        if self.deepep_waterfill_balancer is None:
            return topk_output
        assert TopKOutputChecker.format_is_standard(topk_output)
        return self.deepep_waterfill_balancer.expand_topk(topk_output, num_tokens)
```
**EN:** This block defines `TopK._apply_deepep_waterfill` and contains the main logic for this step. It mainly invokes `TopKOutputChecker.format_is_standard`, `self.deepep_waterfill_balancer.expand_topk`, and `RuntimeError`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TopK._apply_deepep_waterfill`，并承载这一阶段的核心逻辑。 它主要调用 `TopKOutputChecker.format_is_standard`、`self.deepep_waterfill_balancer.expand_topk` 以及 `RuntimeError`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 392-410: `TopK.forward_native` NATIVE execution path
```python
    def forward_native(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        *,
        num_token_non_padded: Optional[torch.Tensor] = None,
        expert_location_dispatch_info: Optional[ExpertLocationDispatchInfo] = None,
    ) -> TopKOutput:
        self.topk_config.torch_native = True
        topk_output = select_experts(
            hidden_states=hidden_states,
            layer_id=self.layer_id,
            router_logits=router_logits,
            topk_config=self.topk_config,
            num_token_non_padded=num_token_non_padded,
            expert_location_dispatch_info=expert_location_dispatch_info,
        )
        return self._apply_deepep_waterfill(topk_output, hidden_states.shape[0])
```
**EN:** This block defines `TopK.forward_native` and contains the main logic for this step. It mainly invokes `select_experts` and `self._apply_deepep_waterfill`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TopK.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `select_experts` 和 `self._apply_deepep_waterfill`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 411-460: `TopK.forward_cuda` CUDA execution path
```python
    def forward_cuda(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        *,
        num_token_non_padded: Optional[torch.Tensor] = None,
        expert_location_dispatch_info: Optional[ExpertLocationDispatchInfo] = None,
    ) -> TopKOutput:
        if self.topk_config.output_format is not None:
            output_format = self.topk_config.output_format
        elif get_moe_runner_backend().is_triton_kernels():
            output_format = TopKOutputFormat.TRITON_KERNEL
        elif get_moe_runner_backend().is_flashinfer_trtllm() or (
            get_moe_runner_backend().is_flashinfer_mxfp4() and not self.is_fp4_experts
        ):
            output_format = TopKOutputFormat.BYPASSED
        else:
            output_format = TopKOutputFormat.STANDARD

        if output_format == TopKOutputFormat.TRITON_KERNEL:
            # renormalize=True is equivalent to sm_first=False
            routing_data, gather_idx, scatter_idx = routing(
                router_logits,
                self.topk_config.top_k,
                sm_first=not self.topk_config.renormalize,
            )
            return TritonKernelTopKOutput(routing_data, gather_idx, scatter_idx)
        elif output_format == TopKOutputFormat.BYPASSED:
            return BypassedTopKOutput(
                hidden_states=hidden_states,
                router_logits=router_logits,
                topk_config=self.topk_config,
                num_token_non_padded=num_token_non_padded,
                expert_location_dispatch_info=expert_location_dispatch_info,
            )
        else:
            self.topk_config.torch_native = False
            with use_symmetric_memory(
                get_tp_group(), disabled=not is_allocation_symmetric()
            ):
                topk_output = select_experts(
                    hidden_states=hidden_states,
                    layer_id=self.layer_id,
                    router_logits=router_logits,
                    topk_config=self.topk_config,
                    num_token_non_padded=num_token_non_padded,
                    expert_location_dispatch_info=expert_location_dispatch_info,
                )
        return self._apply_deepep_waterfill(topk_output, hidden_states.shape[0])
```
**EN:** This block defines `TopK.forward_cuda` and contains the main logic for this step. It mainly invokes `self._apply_deepep_waterfill`, `get_moe_runner_backend.is_triton_kernels`, `routing`, `TritonKernelTopKOutput`, and `BypassedTopKOutput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output_format`, `routing_data`, `gather_idx`, `scatter_idx`, and `topk_output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TopK.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self._apply_deepep_waterfill`、`get_moe_runner_backend.is_triton_kernels`、`routing`、`TritonKernelTopKOutput` 以及 `BypassedTopKOutput`，说明该流程会编排底层辅助函数或计算内核。 像 `output_format`、`routing_data`、`gather_idx`、`scatter_idx` 以及 `topk_output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 461-478: `TopK.forward_cpu` CPU execution path
```python
    def forward_cpu(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        *,
        num_token_non_padded: Optional[torch.Tensor] = None,
        expert_location_dispatch_info: Optional[ExpertLocationDispatchInfo] = None,
    ) -> TopKOutput:
        topk_output = select_experts(
            hidden_states=hidden_states,
            layer_id=self.layer_id,
            router_logits=router_logits,
            topk_config=self.topk_config,
            num_token_non_padded=num_token_non_padded,
            expert_location_dispatch_info=expert_location_dispatch_info,
        )
        return self._apply_deepep_waterfill(topk_output, hidden_states.shape[0])
```
**EN:** This block defines `TopK.forward_cpu` and contains the main logic for this step. It mainly invokes `select_experts` and `self._apply_deepep_waterfill`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TopK.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `select_experts` 和 `self._apply_deepep_waterfill`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 479-498: `TopK.forward_npu` NPU execution path
```python
    def forward_npu(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        *,
        num_token_non_padded: Optional[torch.Tensor] = None,
        expert_location_dispatch_info: Optional[ExpertLocationDispatchInfo] = None,
    ) -> TopKOutput:

        from sglang.srt.hardware_backend.npu.moe.topk import fused_topk_npu

        return fused_topk_npu(
            hidden_states=hidden_states,
            router_logits=router_logits,
            topk_config=self.topk_config,
            num_token_non_padded=num_token_non_padded,
            expert_location_dispatch_info=expert_location_dispatch_info,
            layer_id=self.layer_id,
        )
```
**EN:** This block defines `TopK.forward_npu` and contains the main logic for this step. It mainly invokes `fused_topk_npu`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TopK.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `fused_topk_npu`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 499-524: Function `TopK.empty_topk_output` and its core logic
```python
    def empty_topk_output(self, device: torch.device) -> TopKOutput:
        topk = self.topk_config.top_k - self.topk_config.num_fused_shared_experts
        with use_symmetric_memory(
            get_tp_group(), disabled=not is_allocation_symmetric()
        ):
            topk_weights = torch.empty((0, topk), dtype=torch.float32, device=device)
            topk_ids = torch.full((0, topk), -1, dtype=torch.int32, device=device)
        # FIXME: router_logits should be of size (0, num_experts)
        router_logits = torch.empty((0, topk), dtype=torch.float32, device=device)
        topk_output = StandardTopKOutput(topk_weights, topk_ids, router_logits)
        if self.topk_config.num_fused_shared_experts > 0 and is_deepep_class_backend():
            n = self.topk_config.num_fused_shared_experts
            topk_output = topk_output._replace(
                topk_ids=topk_output.topk_ids.new_empty(
                    (0, topk_output.topk_ids.shape[-1] + n)
                ),
                topk_weights=topk_output.topk_weights.new_empty(
                    (0, topk_output.topk_weights.shape[-1] + n)
                ),
            )
        return self._apply_deepep_waterfill(topk_output, 0)


# ------------------------------- TopK implementation -------------------------------------
```
**EN:** This block defines `TopK.empty_topk_output` and contains the main logic for this step. It mainly invokes `torch.empty`, `StandardTopKOutput`, `self._apply_deepep_waterfill`, `use_symmetric_memory`, and `torch.full`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk`, `router_logits`, `topk_output`, `topk_weights`, and `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `TopK.empty_topk_output`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`StandardTopKOutput`、`self._apply_deepep_waterfill`、`use_symmetric_memory` 以及 `torch.full`，说明该流程会编排底层辅助函数或计算内核。 像 `topk`、`router_logits`、`topk_output`、`topk_weights` 以及 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 525-565: Function `fused_topk_torch_native` and its core logic
```python
def fused_topk_torch_native(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    correction_bias: torch.Tensor = None,
    scoring_func: str = "softmax",
):
    def scoring_func_impl(gating_output: torch.Tensor) -> torch.Tensor:
        if scoring_func == "softmax":
            return gating_output.softmax(dim=-1)
        elif scoring_func == "sigmoid":
            return gating_output.sigmoid()
        else:
            raise ValueError(f"Invalid scoring function: {scoring_func}")

    if correction_bias is not None:
        n_routed_experts = gating_output.shape[-1]
        scores = scoring_func_impl(gating_output)
        scores_for_choice = scores.view(
            -1, n_routed_experts
        ) + correction_bias.unsqueeze(0)
        topk_ids = torch.topk(scores_for_choice, k=topk, dim=-1, sorted=False)[1]
        topk_weights = scores.gather(1, topk_ids)
    else:
        assert (
            hidden_states.shape[0] == gating_output.shape[0]
        ), f"Number of tokens mismatch, {hidden_states.shape=} vs {gating_output.shape=}"
        M, _ = hidden_states.shape
        topk_weights = torch.empty(
            M, topk, dtype=torch.float32, device=hidden_states.device
        )
        topk_ids = torch.empty(M, topk, dtype=torch.int32, device=hidden_states.device)
        topk_weights = scoring_func_impl(gating_output.float())
        topk_weights, topk_ids = torch.topk(topk_weights, topk, dim=-1)

    if renormalize:
        topk_weights = topk_weights / topk_weights.sum(dim=-1, keepdim=True)
    return topk_weights, topk_ids
```
**EN:** This block defines `fused_topk_torch_native` and contains the main logic for this step. It mainly invokes `scoring_func_impl`, `scores.gather`, `torch.empty`, `torch.topk`, and `gating_output.softmax`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `n_routed_experts`, `scores`, `scores_for_choice`, `topk_ids`, and `topk_weights` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_topk_torch_native`，并承载这一阶段的核心逻辑。 它主要调用 `scoring_func_impl`、`scores.gather`、`torch.empty`、`torch.topk` 以及 `gating_output.softmax`，说明该流程会编排底层辅助函数或计算内核。 像 `n_routed_experts`、`scores`、`scores_for_choice`、`topk_ids` 以及 `topk_weights` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 566-584: Function `fused_topk_softmax_torch_raw_logits` and its core logic
```python
def fused_topk_softmax_torch_raw_logits(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
):
    assert (
        hidden_states.shape[0] == gating_output.shape[0]
    ), f"Number of tokens mismatch, {hidden_states.shape=} vs {gating_output.shape=}"

    _, topk_ids = torch.topk(gating_output, k=topk, dim=-1, sorted=False)
    logits = gating_output.float()
    topk_weights = logits.gather(1, topk_ids)
    if renormalize:
        topk_weights = F.softmax(topk_weights, dim=-1, dtype=torch.float32)

    return topk_weights.to(torch.float32), topk_ids.to(torch.int32)
```
**EN:** This block defines `fused_topk_softmax_torch_raw_logits` and contains the main logic for this step. It mainly invokes `torch.topk`, `gating_output.float`, `logits.gather`, `F.softmax`, and `topk_weights.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `_`, `topk_ids`, `logits`, and `topk_weights` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_topk_softmax_torch_raw_logits`，并承载这一阶段的核心逻辑。 它主要调用 `torch.topk`、`gating_output.float`、`logits.gather`、`F.softmax` 以及 `topk_weights.to`，说明该流程会编排底层辅助函数或计算内核。 像 `_`、`topk_ids`、`logits` 以及 `topk_weights` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 585-601: Function `fused_topk_cpu` and its core logic
```python
def fused_topk_cpu(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    correction_bias: torch.Tensor = None,
    scoring_func: str = "softmax",
):
    topk_weights, topk_ids = torch.ops.sgl_kernel.topk_softmax_cpu(
        hidden_states=hidden_states,
        gating_output=gating_output,
        topk=topk,
        renormalize=renormalize,
    )
    return topk_weights, topk_ids
```
**EN:** This block defines `fused_topk_cpu` and contains the main logic for this step. It mainly invokes `torch.ops.sgl_kernel.topk_softmax_cpu`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_weights` and `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_topk_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.ops.sgl_kernel.topk_softmax_cpu`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_weights` 和 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 602-614: `apply_topk_weights_cpu` apply step for top-k weights CPU
```python
def apply_topk_weights_cpu(need_apply, topk_weights, inputs):
    if not need_apply:
        return inputs, topk_weights

    # TODO: fuse below processing in fused_experts_cpu kernel
    inputs = inputs * topk_weights.to(inputs.dtype)
    topk_weights = torch.ones_like(
        topk_weights, dtype=torch.float32
    )  # clear topk_weights as already applied

    return inputs, topk_weights
```
**EN:** This block defines `apply_topk_weights_cpu` and contains the main logic for this step. It mainly invokes `torch.ones_like` and `topk_weights.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inputs` and `topk_weights` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `apply_topk_weights_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.ones_like` 和 `topk_weights.to`，说明该流程会编排底层辅助函数或计算内核。 像 `inputs` 和 `topk_weights` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 615-676: Function `fused_topk` and its core logic
```python
def fused_topk(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    correction_bias: Optional[torch.Tensor] = None,
    scoring_func: str = "softmax",
):
    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"

    M, _ = hidden_states.shape

    topk_weights = torch.empty(
        M, topk, dtype=torch.float32, device=hidden_states.device
    )
    topk_ids = torch.empty(M, topk, dtype=torch.int32, device=hidden_states.device)

    if scoring_func == "softmax":
        if _use_aiter:

            # Use fused_topk instead of topk_softmax to auto dispatch to the correct kernel
            topk_weights, topk_ids = aiter_fused_topk(
                hidden_states,
                gating_output,
                topk,
                renormalize,
                topk_ids=topk_ids,
                topk_weights=topk_weights,
            )
        else:
            topk_softmax(
                topk_weights,
                topk_ids,
                gating_output,
                renormalize,
            )
    elif scoring_func == "sigmoid":
        if _use_aiter and correction_bias is not None:
            aiter_biased_grouped_topk(
                gating_output,
                correction_bias.to(dtype=gating_output.dtype),
                topk_weights,
                topk_ids,
                num_expert_group=1,
                topk_group=1,
                need_renorm=renormalize,
            )
        else:
            topk_sigmoid(
                topk_weights,
                topk_ids,
                gating_output,
                renormalize,
                correction_bias,
            )
    else:
        raise ValueError(f"Invalid scoring function: {scoring_func}")

    return topk_weights, topk_ids


# This is used by the Deepseek V2/V3/R1 series models
```
**EN:** This block defines `fused_topk` and contains the main logic for this step. It mainly invokes `torch.empty`, `aiter_fused_topk`, `topk_softmax`, `ValueError`, and `aiter_biased_grouped_topk`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `M`, `_`, `topk_weights`, and `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_topk`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`aiter_fused_topk`、`topk_softmax`、`ValueError` 以及 `aiter_biased_grouped_topk`，说明该流程会编排底层辅助函数或计算内核。 像 `M`、`_`、`topk_weights` 以及 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 677-741: Function `grouped_topk_gpu` and its core logic
```python
@torch.compile(dynamic=True, backend=get_compiler_backend(), disable=_is_npu)
def grouped_topk_gpu(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    num_expert_group: Optional[int] = None,
    topk_group: Optional[int] = None,
    num_fused_shared_experts: int = 0,
    routed_scaling_factor: Optional[float] = None,
    apply_routed_scaling_factor_on_output: Optional[bool] = False,
):
    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"

    scores = torch.softmax(gating_output, dim=-1)
    num_token = scores.shape[0]
    num_experts = scores.shape[1]
    group_scores = (
        scores.view(num_token, num_expert_group, -1).max(dim=-1).values
    )  # [n, n_group]
    group_idx = torch.topk(group_scores, k=topk_group, dim=-1, sorted=False)[
        1
    ]  # [n, top_k_group]
    group_mask = torch.zeros_like(group_scores)  # [n, n_group]
    group_mask.scatter_(1, group_idx, 1)  # [n, n_group]
    score_mask = (
        group_mask.unsqueeze(-1)
        .expand(num_token, num_expert_group, scores.shape[-1] // num_expert_group)
        .reshape(num_token, -1)
    )  # [n, e]
    tmp_scores = scores.masked_fill(~score_mask.bool(), 0.0)  # [n, e]
    topk_weights, topk_ids = torch.topk(
        tmp_scores,
        k=topk,
        dim=-1,
        sorted=(True if num_fused_shared_experts > 0 else False),
    )
    if num_fused_shared_experts:
        topk_ids[:, -1] = torch.randint(
            low=num_experts,
            high=num_experts + num_fused_shared_experts,
            size=(topk_ids.size(0),),
            dtype=topk_ids.dtype,
            device=topk_ids.device,
        )
        if routed_scaling_factor is not None:
            topk_weights[:, -1] = (
                topk_weights[:, :-1].sum(dim=-1) / routed_scaling_factor
            )

    if renormalize:
        topk_weights_sum = (
            topk_weights.sum(dim=-1, keepdim=True)
            if num_fused_shared_experts == 0
            else topk_weights[:, :-1].sum(dim=-1, keepdim=True)
        )
        topk_weights = topk_weights / topk_weights_sum
        if apply_routed_scaling_factor_on_output:
            topk_weights *= routed_scaling_factor

    topk_weights, topk_ids = topk_weights.to(torch.float32), topk_ids.to(torch.int32)

    return topk_weights, topk_ids
```
**EN:** This block defines `grouped_topk_gpu` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.compile`, `torch.softmax`, `torch.zeros_like`, `group_mask.scatter_`, and `group_mask.unsqueeze.expand.reshape`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `scores`, `num_token`, `num_experts`, `group_scores`, and `group_idx` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `grouped_topk_gpu`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.compile`、`torch.softmax`、`torch.zeros_like`、`group_mask.scatter_` 以及 `group_mask.unsqueeze.expand.reshape`，说明该流程会编排底层辅助函数或计算内核。 像 `scores`、`num_token`、`num_experts`、`group_scores` 以及 `group_idx` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 742-767: Function `grouped_topk_cpu` and its core logic
```python
def grouped_topk_cpu(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    num_expert_group: Optional[int] = None,
    topk_group: Optional[int] = None,
    num_fused_shared_experts: int = 0,
    routed_scaling_factor: Optional[float] = None,
    apply_routed_scaling_factor_on_output: Optional[bool] = False,
):
    assert not apply_routed_scaling_factor_on_output
    return torch.ops.sgl_kernel.grouped_topk_cpu(
        hidden_states,
        gating_output,
        topk,
        renormalize,
        num_expert_group,
        topk_group,
        num_fused_shared_experts,
        routed_scaling_factor,
        # num_token_non_padded must be None since it is not supported in kernel
        num_token_non_padded=None,
    )
```
**EN:** This block defines `grouped_topk_cpu` and contains the main logic for this step. It mainly invokes `torch.ops.sgl_kernel.grouped_topk_cpu`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `grouped_topk_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.ops.sgl_kernel.grouped_topk_cpu`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 768-805: Function `kimi_k2_biased_topk_impl` and its core logic
```python
@torch.compile(dynamic=True, backend=get_compiler_backend(), disable=_is_npu)
def kimi_k2_biased_topk_impl(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    correction_bias: torch.Tensor,
    topk: int,
    renormalize: bool,
    routed_scaling_factor: Optional[float] = None,
    apply_routed_scaling_factor_on_output: Optional[bool] = False,
):
    """
    Optimized version for num_expert_group=1 case (e.g., Kimi K2 with 384 experts).
    Simplifies the grouped topk logic by removing unnecessary group masking operations.
    Note: This function assumes num_fused_shared_experts=0.
    """
    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"

    scores = gating_output.sigmoid()
    num_token = scores.shape[0]

    # When num_expert_group=1, no need for group masking
    # Directly compute scores with correction bias
    tmp_scores = scores.view(num_token, -1) + correction_bias.unsqueeze(0)

    # Directly select topk experts (no need to sort since num_fused_shared_experts=0)
    _, topk_ids = torch.topk(tmp_scores, k=topk, dim=-1, sorted=False)
    topk_weights = scores.gather(1, topk_ids)

    if renormalize:
        topk_weights_sum = topk_weights.sum(dim=-1, keepdim=True)
        topk_weights = topk_weights / topk_weights_sum
        if apply_routed_scaling_factor_on_output:
            topk_weights *= routed_scaling_factor

    topk_weights, topk_ids = topk_weights.to(torch.float32), topk_ids.to(torch.int32)
    return topk_weights, topk_ids
```
**EN:** This block defines `kimi_k2_biased_topk_impl` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.compile`, `gating_output.sigmoid`, `torch.topk`, `scores.gather`, and `scores.view`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `scores`, `num_token`, `tmp_scores`, `_`, and `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `kimi_k2_biased_topk_impl`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.compile`、`gating_output.sigmoid`、`torch.topk`、`scores.gather` 以及 `scores.view`，说明该流程会编排底层辅助函数或计算内核。 像 `scores`、`num_token`、`tmp_scores`、`_` 以及 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 806-865: Function `biased_topk_impl` and its core logic
```python
@torch.compile(dynamic=True, backend=get_compiler_backend(), disable=_is_npu)
def biased_topk_impl(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    correction_bias: torch.Tensor,
    topk: int,
    renormalize: bool,
    scoring_func: str = "sigmoid",
    num_fused_shared_experts: int = 0,
    routed_scaling_factor: Optional[float] = None,
    num_token_non_padded: Optional[torch.Tensor] = None,
    expert_location_dispatch_info: Optional[ExpertLocationDispatchInfo] = None,
    apply_routed_scaling_factor_on_output: Optional[bool] = False,
):
    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"

    if scoring_func == "sigmoid":
        scores = gating_output.sigmoid()
    elif scoring_func == "sqrtsoftplus":
        scores = torch.nn.functional.softplus(gating_output).sqrt()

    num_token = scores.shape[0]
    num_experts = scores.shape[1]

    scores_for_choice = scores.view(num_token, -1) + correction_bias.unsqueeze(0)
    _, topk_ids = torch.topk(
        scores_for_choice,
        k=topk,
        dim=-1,
        sorted=(True if num_fused_shared_experts > 0 else False),
    )
    topk_weights = scores.gather(1, topk_ids)

    if num_fused_shared_experts:
        topk_ids[:, -1] = torch.randint(
            low=num_experts,
            high=num_experts + num_fused_shared_experts,
            size=(topk_ids.size(0),),
            dtype=topk_ids.dtype,
            device=topk_ids.device,
        )
        if routed_scaling_factor is not None:
            topk_weights[:, -1] = (
                topk_weights[:, :-1].sum(dim=-1) / routed_scaling_factor
            )

    if renormalize:
        topk_weights_sum = (
            topk_weights.sum(dim=-1, keepdim=True)
            if num_fused_shared_experts == 0
            else topk_weights[:, :-1].sum(dim=-1, keepdim=True)
        )
        topk_weights = topk_weights / topk_weights_sum
        if apply_routed_scaling_factor_on_output:
            topk_weights *= routed_scaling_factor

    topk_weights, topk_ids = topk_weights.to(torch.float32), topk_ids.to(torch.int32)
    return topk_weights, topk_ids
```
**EN:** This block defines `biased_topk_impl` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.compile`, `torch.topk`, `scores.gather`, `gating_output.sigmoid`, and `scores.view`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_token`, `num_experts`, `scores_for_choice`, `_`, and `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `biased_topk_impl`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.compile`、`torch.topk`、`scores.gather`、`gating_output.sigmoid` 以及 `scores.view`，说明该流程会编排底层辅助函数或计算内核。 像 `num_token`、`num_experts`、`scores_for_choice`、`_` 以及 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 866-896: Function `biased_topk_jit_kernel_impl` and its core logic
```python
def biased_topk_jit_kernel_impl(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    correction_bias: torch.Tensor,
    topk: int,
    renormalize: bool,
    scoring_func: str = "sigmoid",
    num_fused_shared_experts: int = 0,
    routed_scaling_factor: Optional[float] = None,
    num_token_non_padded: Optional[torch.Tensor] = None,
    expert_location_dispatch_info: Optional[ExpertLocationDispatchInfo] = None,
    apply_routed_scaling_factor_on_output: Optional[bool] = False,
):
    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"

    from sglang.jit_kernel.moe_fused_gate import moe_fused_gate

    topk_weights, topk_ids = moe_fused_gate(
        gating_output,
        correction_bias,
        topk=topk,
        scoring_func=scoring_func,
        num_fused_shared_experts=num_fused_shared_experts,
        renormalize=renormalize,
        routed_scaling_factor=routed_scaling_factor,
        apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
    )
    topk_weights, topk_ids = topk_weights.to(torch.float32), topk_ids.to(torch.int32)
    return topk_weights, topk_ids
```
**EN:** This block defines `biased_topk_jit_kernel_impl` and contains the main logic for this step. It mainly invokes `moe_fused_gate`, `topk_weights.to`, and `topk_ids.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_weights` and `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `biased_topk_jit_kernel_impl`，并承载这一阶段的核心逻辑。 它主要调用 `moe_fused_gate`、`topk_weights.to` 以及 `topk_ids.to`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_weights` 和 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 897-969: Function `biased_grouped_topk_impl` and its core logic
```python
@torch.compile(dynamic=True, backend=get_compiler_backend(), disable=_is_npu)
def biased_grouped_topk_impl(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    correction_bias: torch.Tensor,
    topk: int,
    renormalize: bool,
    num_expert_group: Optional[int] = None,
    topk_group: Optional[int] = None,
    num_fused_shared_experts: int = 0,
    routed_scaling_factor: Optional[float] = None,
    apply_routed_scaling_factor_on_output: Optional[bool] = False,
):
    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"

    scores = gating_output.sigmoid()
    num_token = scores.shape[0]
    num_experts = scores.shape[1]
    scores_for_choice = scores.view(num_token, -1) + correction_bias.unsqueeze(0)
    group_scores = (
        scores_for_choice.view(num_token, num_expert_group, -1)
        .topk(2, dim=-1)[0]
        .sum(dim=-1)
    )  # [n, n_group]
    group_idx = torch.topk(group_scores, k=topk_group, dim=-1, sorted=False)[
        1
    ]  # [n, top_k_group]
    group_mask = torch.zeros_like(group_scores)  # [n, n_group]
    group_mask.scatter_(1, group_idx, 1)  # [n, n_group]
    score_mask = (
        group_mask.unsqueeze(-1)
        .expand(num_token, num_expert_group, scores.shape[-1] // num_expert_group)
        .reshape(num_token, -1)
    )  # [n, e]
    tmp_scores = scores_for_choice.masked_fill(
        ~score_mask.bool(), float("-inf")
    )  # [n, e]
    _, topk_ids = torch.topk(
        tmp_scores,
        k=topk,
        dim=-1,
        sorted=(True if num_fused_shared_experts > 0 else False),
    )
    topk_weights = scores.gather(1, topk_ids)

    if num_fused_shared_experts:
        topk_ids[:, -1] = torch.randint(
            low=num_experts,
            high=num_experts + num_fused_shared_experts,
            size=(topk_ids.size(0),),
            dtype=topk_ids.dtype,
            device=topk_ids.device,
        )
        if routed_scaling_factor is not None:
            topk_weights[:, -1] = (
                topk_weights[:, :-1].sum(dim=-1) / routed_scaling_factor
            )

    if renormalize:
        topk_weights_sum = (
            topk_weights.sum(dim=-1, keepdim=True)
            if num_fused_shared_experts == 0
            else topk_weights[:, :-1].sum(dim=-1, keepdim=True)
        )
        topk_weights = topk_weights / topk_weights_sum
        if apply_routed_scaling_factor_on_output:
            topk_weights *= routed_scaling_factor

    topk_weights, topk_ids = topk_weights.to(torch.float32), topk_ids.to(torch.int32)

    return topk_weights, topk_ids
```
**EN:** This block defines `biased_grouped_topk_impl` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.compile`, `gating_output.sigmoid`, `scores_for_choice.view.topk.sum`, `torch.zeros_like`, and `group_mask.scatter_`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `scores`, `num_token`, `num_experts`, `scores_for_choice`, and `group_scores` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `biased_grouped_topk_impl`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.compile`、`gating_output.sigmoid`、`scores_for_choice.view.topk.sum`、`torch.zeros_like` 以及 `group_mask.scatter_`，说明该流程会编排底层辅助函数或计算内核。 像 `scores`、`num_token`、`num_experts`、`scores_for_choice` 以及 `group_scores` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 970-973: `is_power_of_two` predicate for is power of two
```python
def is_power_of_two(n):
    return n > 0 and math.log2(n).is_integer()
```
**EN:** This block defines `is_power_of_two` and contains the main logic for this step. It mainly invokes `math.log2.is_integer` and `math.log2`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_power_of_two`，并承载这一阶段的核心逻辑。 它主要调用 `math.log2.is_integer` 和 `math.log2`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 974-987: Internal helper `_mask_topk_ids_padded_region`
```python
def _mask_topk_ids_padded_region(
    topk_ids: torch.Tensor,
    num_token_non_padded: Optional[torch.Tensor] = None,
) -> None:
    if num_token_non_padded is None:
        return
    # TODO: let the kernel support other dtypes
    if _is_cuda and topk_ids.dtype == torch.int32:
        mask_topk_ids(topk_ids, num_token_non_padded)
    else:
        indices = torch.arange(0, topk_ids.shape[0], device=topk_ids.device)
        topk_ids[indices >= num_token_non_padded, :] = -1
```
**EN:** This block defines `_mask_topk_ids_padded_region` and contains the main logic for this step. It mainly invokes `mask_topk_ids` and `torch.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `indices` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_mask_topk_ids_padded_region`，并承载这一阶段的核心逻辑。 它主要调用 `mask_topk_ids` 和 `torch.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `indices` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 988-996: Internal helper `_biased_grouped_topk_postprocess`
```python
@torch.compile(dynamic=True, backend=get_compiler_backend())
def _biased_grouped_topk_postprocess(
    topk_ids, expert_location_dispatch_info, num_token_non_padded
):
    topk_ids = topk_ids_logical_to_physical(topk_ids, expert_location_dispatch_info)
    _mask_topk_ids_padded_region(topk_ids, num_token_non_padded)
    return topk_ids
```
**EN:** This block defines `_biased_grouped_topk_postprocess` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.compile`, `topk_ids_logical_to_physical`, `_mask_topk_ids_padded_region`, and `get_compiler_backend`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_biased_grouped_topk_postprocess`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.compile`、`topk_ids_logical_to_physical`、`_mask_topk_ids_padded_region` 以及 `get_compiler_backend`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 997-1176: Function `biased_grouped_topk_gpu` and its core logic
```python
def biased_grouped_topk_gpu(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    correction_bias: torch.Tensor,
    topk: int,
    renormalize: bool,
    num_expert_group: Optional[int] = None,
    topk_group: Optional[int] = None,
    num_fused_shared_experts: int = 0,
    routed_scaling_factor: Optional[float] = None,
    apply_routed_scaling_factor_on_output: Optional[bool] = False,
):

    num_tokens = gating_output.shape[0]
    num_experts = gating_output.shape[1]
    experts_per_group = (
        num_experts // num_expert_group if num_expert_group else num_experts
    )

    # topk for routed experts only (shared experts are appended separately below)
    topk_routed = topk - num_fused_shared_experts
    if (
        _is_cuda
        and fused_topk_deepseek is not None
        and is_power_of_two(num_experts)
        # flashinfer constraints (applied to routed experts only)
        and topk_routed <= 8
        and topk_group <= num_expert_group
        and topk_group * num_expert_group >= topk_routed
        and (
            (experts_per_group <= 32 and experts_per_group * topk_group <= 128)
            if num_expert_group > 1
            else num_experts <= 384
        )
    ):
        # Pre-allocate output tensors (flashinfer mutates them in-place)
        topk_weights = torch.empty(
            (num_tokens, topk_routed), dtype=torch.float32, device=gating_output.device
        )
        topk_ids = torch.empty(
            (num_tokens, topk_routed), dtype=torch.int32, device=gating_output.device
        )

        # flashinfer always applies the scaling_factor internally
        scaling_factor = 1.0
        if routed_scaling_factor is not None and apply_routed_scaling_factor_on_output:
            scaling_factor = routed_scaling_factor

        # flashinfer's fused_topk_deepseek
        fused_topk_deepseek(
            gating_output.to(dtype=torch.float32),
            correction_bias,
            num_expert_group,
            topk_group,
            topk_routed,
            scaling_factor,
            topk_weights,
            topk_ids,
            True,
        )

        if num_fused_shared_experts > 0:
            # Append shared expert columns: ID = num_experts (first shared slot),
            # weight = sum(routed) / scaling_factor (matching biased_grouped_topk_impl).
            # DeepEP fusion will overwrite both in _remap_topk_ids_for_deepep_fusion.
            topk_ids = F.pad(topk_ids, (0, num_fused_shared_experts), value=num_experts)
            topk_weights = F.pad(topk_weights, (0, num_fused_shared_experts))
            if routed_scaling_factor is not None:
                topk_weights[:, topk_routed:] = (
                    topk_weights[:, :topk_routed].sum(dim=-1, keepdim=True)
                    / routed_scaling_factor
                )

        return topk_weights, topk_ids

    elif (
        _is_cuda
        # moe_fused_gate kernel ensures that num_experts/num_expert_group does not exceed MAX_VPT=32 now. And when kernel can handle MAX_VPT > 32, we can remove this assertion.
        and experts_per_group <= 32
        and is_power_of_two(num_experts)
    ):
        topk_weights, topk_ids = moe_fused_gate(
            gating_output.to(dtype=torch.float32),
            correction_bias,
            num_expert_group,
            topk_group,
            topk,
            num_fused_shared_experts,
            routed_scaling_factor if routed_scaling_factor is not None else 1.0,
            apply_routed_scaling_factor_on_output,
        )

        return topk_weights, topk_ids

    elif _use_aiter:
        assert not apply_routed_scaling_factor_on_output, "Not implemented"
        token = gating_output.shape[0]
        device = gating_output.device
        assert (
            hidden_states.shape[0] == gating_output.shape[0]
        ), f"Number of tokens mismatch: hidden_states.shape[0] = {hidden_states.shape[0]}, gating_output.shape[0] = {gating_output.shape[0]}"
        topk_weights = torch.empty((token, topk), dtype=torch.float32, device=device)
        topk_ids = torch.empty((token, topk), dtype=torch.int32, device=device)
        aiter_biased_grouped_topk(
            gating_output,
            correction_bias.to(dtype=gating_output.dtype),
            topk_weights,
            topk_ids,
            num_expert_group,
            topk_group,
            renormalize,
            routed_scaling_factor if routed_scaling_factor is not None else 1.0,
        )
        return topk_weights, topk_ids
    elif _is_musa and (
        gating_output.shape[1] // num_expert_group <= 32
        or (num_expert_group == 1 and gating_output.shape[1] in {160, 256, 384})
    ):
        topk_weights, topk_ids = moe_fused_gate(
            gating_output.to(dtype=torch.float32),
            correction_bias,
            num_expert_group,
            topk_group,
            topk,
            num_fused_shared_experts,
            routed_scaling_factor if routed_scaling_factor is not None else 1.0,
            True,
            apply_routed_scaling_factor_on_output,
        )
    else:
        # Use optimized path for Kimi K2 (384 experts with num_expert_group=1)
        num_experts = gating_output.shape[1]
        if _is_cuda and num_experts == 384 and num_expert_group == 1:
            return kimi_k2_moe_fused_gate(
                gating_output.to(dtype=torch.float32),
                correction_bias,
                topk=topk,
                renormalize=renormalize,
                routed_scaling_factor=routed_scaling_factor,
                apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
            )
        elif (
            _is_cuda
            and num_expert_group == 1
            and topk_group == 1
            and num_fused_shared_experts == 0
            and num_experts <= 512
            and topk <= 8
        ):
            from sglang.jit_kernel.grouped_topk import grouped_topk as jit_grouped_topk

            scaling = (
                routed_scaling_factor if routed_scaling_factor is not None else 1.0
            )
            if not apply_routed_scaling_factor_on_output:
                scaling = 1.0
            return jit_grouped_topk(
                gating_output.to(dtype=torch.float32),
                correction_bias.to(dtype=torch.float32),
                num_expert_group,
                topk_group,
                topk,
                renormalize,
                scaling,
            )
        else:
            return biased_grouped_topk_impl(
                hidden_states,
                gating_output,
                correction_bias,
                topk,
                renormalize,
                num_expert_group,
                topk_group,
                num_fused_shared_experts=num_fused_shared_experts,
                routed_scaling_factor=routed_scaling_factor,
                apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
            )
```
**EN:** This block defines `biased_grouped_topk_gpu` and contains the main logic for this step. It mainly invokes `is_power_of_two`, `torch.empty`, `fused_topk_deepseek`, `gating_output.to`, and `F.pad`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_tokens`, `num_experts`, `experts_per_group`, `topk_routed`, and `topk_weights` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `biased_grouped_topk_gpu`，并承载这一阶段的核心逻辑。 它主要调用 `is_power_of_two`、`torch.empty`、`fused_topk_deepseek`、`gating_output.to` 以及 `F.pad`，说明该流程会编排底层辅助函数或计算内核。 像 `num_tokens`、`num_experts`、`experts_per_group`、`topk_routed` 以及 `topk_weights` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1177-1204: Function `biased_grouped_topk_cpu` and its core logic
```python
def biased_grouped_topk_cpu(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    correction_bias: torch.Tensor,
    topk: int,
    renormalize: bool,
    num_expert_group: Optional[int] = None,
    topk_group: Optional[int] = None,
    compiled: bool = True,
    num_fused_shared_experts: int = 0,
    routed_scaling_factor: Optional[float] = None,
    apply_routed_scaling_factor_on_output: Optional[bool] = False,
):
    return torch.ops.sgl_kernel.biased_grouped_topk_cpu(
        hidden_states,
        gating_output,
        correction_bias,
        topk,
        renormalize,
        num_expert_group,
        topk_group,
        num_fused_shared_experts,
        routed_scaling_factor if apply_routed_scaling_factor_on_output else None,
        # num_token_non_padded must be None since it is not supported in kernel
        num_token_non_padded=None,
    )
```
**EN:** This block defines `biased_grouped_topk_cpu` and contains the main logic for this step. It mainly invokes `torch.ops.sgl_kernel.biased_grouped_topk_cpu`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `biased_grouped_topk_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.ops.sgl_kernel.biased_grouped_topk_cpu`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 1205-1215: Module-level helper logic
```python
if _is_cpu and _is_cpu_amx_available:
    biased_grouped_topk = biased_grouped_topk_cpu
    grouped_topk = grouped_topk_cpu
    fused_topk_native = fused_topk_cpu
    fused_topk = fused_topk_cpu
else:
    biased_grouped_topk = biased_grouped_topk_gpu
    grouped_topk = grouped_topk_gpu
    fused_topk_native = fused_topk_torch_native
```
**EN:** This section prepares the module namespace. Shared names such as `biased_grouped_topk`, `grouped_topk`, `fused_topk_native`, and `fused_topk` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 像 `biased_grouped_topk`、`grouped_topk`、`fused_topk_native` 以及 `fused_topk` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 1216-1259: Internal helper `_remap_topk_for_deepep`
```python
def _remap_topk_for_deepep(
    topk_ids: torch.Tensor,
    topk_weights: torch.Tensor,
    num_fused_shared_experts: int,
    n_routed_experts: int,
    topk_config: TopKConfig,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Remap TopK output to DeepEP interleaved expert layout.

    DeepEP dispatch needs each rank's shared expert at a unique ID so tokens
    route to the correct rank. The layout interleaves shared slots among
    routed experts: [routed_0..L-1, shared, routed_L..2L-1, shared, ...].

    Routed IDs:  e -> e + e // num_local_routed
    Shared IDs:  ep_rank * num_local_experts + num_local_routed
    Shared weight: 1 / routed_scaling_factor (compensates post-MoE scaling)
    """
    if topk_ids.shape[0] == 0:
        return topk_ids, topk_weights

    ep_size = get_moe_expert_parallel_world_size()
    ep_rank = get_moe_expert_parallel_rank()
    num_local_routed = n_routed_experts // ep_size
    num_local_experts = num_local_routed + num_fused_shared_experts

    # Remap routed IDs: insert gaps for shared expert slots (single fused op)
    routed = topk_ids[:, :-num_fused_shared_experts]
    topk_ids[:, :-num_fused_shared_experts] = routed + routed // num_local_routed

    # Set shared expert IDs to route to home rank (vectorized)
    topk_ids[:, -num_fused_shared_experts:] = (
        ep_rank * num_local_experts
        + num_local_routed
        + torch.arange(num_fused_shared_experts, device=topk_ids.device)
    )

    # Override shared weight: 1/routed_scaling_factor so net contribution = 1.0
    routed_scaling_factor = topk_config.routed_scaling_factor
    if routed_scaling_factor is not None and routed_scaling_factor != 0:
        topk_weights[:, -num_fused_shared_experts:] = 1.0 / routed_scaling_factor

    return topk_ids, topk_weights
```
**EN:** This block defines `_remap_topk_for_deepep` and contains the main logic for this step. It mainly invokes `get_moe_expert_parallel_world_size`, `get_moe_expert_parallel_rank`, and `torch.arange`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `ep_size`, `ep_rank`, `num_local_routed`, `num_local_experts`, and `routed` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_remap_topk_for_deepep`，并承载这一阶段的核心逻辑。 它主要调用 `get_moe_expert_parallel_world_size`、`get_moe_expert_parallel_rank` 以及 `torch.arange`，说明该流程会编排底层辅助函数或计算内核。 像 `ep_size`、`ep_rank`、`num_local_routed`、`num_local_experts` 以及 `routed` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1260-1329: Internal helper `_post_process_topk_ids`
```python
def _post_process_topk_ids(
    topk_ids: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_config: TopKConfig,
    router_logits: torch.Tensor,
    layer_id: int,
    num_token_non_padded: Optional[torch.Tensor] = None,
    expert_location_dispatch_info: Optional[ExpertLocationDispatchInfo] = None,
) -> torch.Tensor:
    num_fused_shared_experts = topk_config.num_fused_shared_experts
    fused_shared_experts_scaling_factor = (
        topk_config.fused_shared_experts_scaling_factor
    )
    if (cap := get_global_experts_capturer()) is not None:
        cap.capture(
            layer_id=layer_id,
            topk_indices=topk_ids,
        )
    if _is_cuda:
        # When shared experts are fused (appended as extra columns in topk_ids),
        # EPLB dispatch must only remap the routed expert columns.
        # The shared expert column (value = n_routed_experts) would be out-of-bounds
        # for the logical-to-physical dispatch table.
        if num_fused_shared_experts > 0 and is_deepep_class_backend():
            shared_cols = topk_ids[:, -num_fused_shared_experts:]
            routed_cols = topk_ids[:, :-num_fused_shared_experts]
            routed_cols = _biased_grouped_topk_postprocess(
                routed_cols, expert_location_dispatch_info, num_token_non_padded
            )
            topk_ids = torch.cat([routed_cols, shared_cols], dim=-1)
        else:
            topk_ids = _biased_grouped_topk_postprocess(
                topk_ids, expert_location_dispatch_info, num_token_non_padded
            )

    if num_fused_shared_experts > 0 and _use_aiter:
        M, N = router_logits.shape
        scale_factor = (
            1.0
            if fused_shared_experts_scaling_factor is None
            else fused_shared_experts_scaling_factor
        )

        # Lazy import to avoid circular-import issues
        from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe_triton_kernels import (
            fused_append_shared_experts,
        )

        topk_ids, topk_weights = fused_append_shared_experts(
            topk_ids,
            topk_weights,
            num_fused_shared_experts,
            scale_factor,
            N,  # base id for shared experts
        )

    # DeepEP: remap to interleaved expert layout where each rank's shared
    # expert has a unique ID for dispatch routing.
    if num_fused_shared_experts > 0 and is_deepep_class_backend():
        topk_ids, topk_weights = _remap_topk_for_deepep(
            topk_ids,
            topk_weights,
            num_fused_shared_experts,
            router_logits.shape[1],
            topk_config,
        )

    return topk_ids, topk_weights
```
**EN:** This block defines `_post_process_topk_ids` and contains the main logic for this step. It mainly invokes `cap.capture`, `fused_append_shared_experts`, `is_deepep_class_backend`, `_remap_topk_for_deepep`, and `get_global_experts_capturer`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_fused_shared_experts`, `fused_shared_experts_scaling_factor`, `M`, `N`, and `scale_factor` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_post_process_topk_ids`，并承载这一阶段的核心逻辑。 它主要调用 `cap.capture`、`fused_append_shared_experts`、`is_deepep_class_backend`、`_remap_topk_for_deepep` 以及 `get_global_experts_capturer`，说明该流程会编排底层辅助函数或计算内核。 像 `num_fused_shared_experts`、`fused_shared_experts_scaling_factor`、`M`、`N` 以及 `scale_factor` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1330-1481: Function `select_experts` and its core logic
```python
def select_experts(
    hidden_states: torch.Tensor,
    router_logits: torch.Tensor,
    topk_config: TopKConfig,
    *,
    layer_id: Optional[int] = None,
    num_token_non_padded: Optional[torch.Tensor] = None,
    expert_location_dispatch_info: Optional[ExpertLocationDispatchInfo] = None,
) -> StandardTopKOutput:
    top_k = topk_config.top_k
    use_grouped_topk = topk_config.use_grouped_topk
    topk_group = topk_config.topk_group
    num_expert_group = topk_config.num_expert_group
    renormalize = topk_config.renormalize
    num_fused_shared_experts = topk_config.num_fused_shared_experts
    custom_routing_function = topk_config.custom_routing_function
    correction_bias = topk_config.correction_bias
    torch_native = topk_config.torch_native
    routed_scaling_factor = topk_config.routed_scaling_factor
    apply_routed_scaling_factor_on_output = (
        topk_config.apply_routed_scaling_factor_on_output
    )

    scoring_func = topk_config.scoring_func

    (
        router_logits,
        correction_bias,
    ) = expert_location_dispatch.transform_select_experts_inputs(
        router_logits=router_logits,
        correction_bias=correction_bias,
        info=expert_location_dispatch_info,
    )

    # DeepSeek V2/V3/R1 series models use grouped_top_k
    # remove num_fused_shared_experts from grouped_topk/biased_grouped_topk
    num_routed_topk = top_k - num_fused_shared_experts
    if use_grouped_topk:
        assert topk_group is not None
        assert num_expert_group is not None
        if correction_bias is None:
            topk_weights, topk_ids = grouped_topk(
                hidden_states=hidden_states,
                gating_output=router_logits,
                topk=num_routed_topk if _use_aiter else top_k,
                renormalize=renormalize,
                num_expert_group=num_expert_group,
                topk_group=topk_group,
                num_fused_shared_experts=num_fused_shared_experts,
                routed_scaling_factor=routed_scaling_factor,
                apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
            )
        else:
            topk_weights, topk_ids = biased_grouped_topk(
                hidden_states=hidden_states,
                gating_output=router_logits,
                correction_bias=correction_bias,
                topk=num_routed_topk if _use_aiter else top_k,
                renormalize=renormalize,
                num_expert_group=num_expert_group,
                topk_group=topk_group,
                num_fused_shared_experts=num_fused_shared_experts,
                routed_scaling_factor=routed_scaling_factor,
                apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
            )
    elif torch_native and custom_routing_function is None:
        assert (
            num_token_non_padded is None
        ), "num_token_non_padded is not yet supported in fused_topk_native"
        assert expert_location_dispatch_info is None
        assert not apply_routed_scaling_factor_on_output, "Not implemented"
        topk_weights, topk_ids = fused_topk_native(
            hidden_states=hidden_states,
            gating_output=router_logits,
            topk=num_routed_topk if _use_aiter else top_k,
            renormalize=renormalize,
            correction_bias=correction_bias,
            scoring_func=scoring_func,
        )
    elif custom_routing_function is None:
        assert not apply_routed_scaling_factor_on_output, "Not implemented"
        if scoring_func == "sqrtsoftplus":
            _biased_topk = (
                biased_topk_jit_kernel_impl
                if envs.SGLANG_OPT_USE_JIT_KERNEL_FUSED_TOPK.get()
                else biased_topk_impl
            )

            topk_weights, topk_ids = _biased_topk(
                hidden_states=hidden_states,
                gating_output=router_logits,
                correction_bias=correction_bias,
                topk=num_routed_topk if _use_aiter else top_k,
                renormalize=renormalize,
                scoring_func=scoring_func,
                num_fused_shared_experts=num_fused_shared_experts,
                routed_scaling_factor=routed_scaling_factor,
                num_token_non_padded=num_token_non_padded,
                expert_location_dispatch_info=expert_location_dispatch_info,
                apply_routed_scaling_factor_on_output=apply_routed_scaling_factor_on_output,
            )
        elif (
            get_moe_runner_backend().is_flashinfer_trtllm_routed()
            and scoring_func == "softmax"
            and correction_bias is None
        ):
            # flashinfer_trtllm_routed uses raw-logits topk
            topk_weights, topk_ids = fused_topk_softmax_torch_raw_logits(
                hidden_states=hidden_states,
                gating_output=router_logits,
                topk=num_routed_topk if _use_aiter else top_k,
                renormalize=renormalize,
            )
        else:
            # Qwen3MOE uses fused_topk
            topk_weights, topk_ids = fused_topk(
                hidden_states=hidden_states,
                gating_output=router_logits,
                topk=num_routed_topk if _use_aiter else top_k,
                renormalize=renormalize,
                correction_bias=correction_bias,
                scoring_func=scoring_func,
            )
    else:
        assert (
            num_token_non_padded is None
        ), "num_token_non_padded is not yet supported in custom_routing_function"
        assert expert_location_dispatch_info is None
        assert not apply_routed_scaling_factor_on_output, "Not implemented"
        topk_weights, topk_ids = custom_routing_function(
            hidden_states=hidden_states,
            gating_output=router_logits,
            topk=num_routed_topk if _use_aiter else top_k,
            renormalize=renormalize,
        )

    topk_ids, topk_weights = _post_process_topk_ids(
        topk_ids=topk_ids,
        topk_weights=topk_weights,
        topk_config=topk_config,
        router_logits=router_logits,
        num_token_non_padded=num_token_non_padded,
        layer_id=layer_id,
        expert_location_dispatch_info=expert_location_dispatch_info,
    )

    get_global_expert_distribution_recorder().on_select_experts(topk_ids=topk_ids)

    return StandardTopKOutput(topk_weights, topk_ids, router_logits)


# Register fake implementations for torch.compile support
```
**EN:** This block defines `select_experts` and contains the main logic for this step. It mainly invokes `expert_location_dispatch.transform_select_experts_inputs`, `_post_process_topk_ids`, `get_global_expert_distribution_recorder.on_select_experts`, `StandardTopKOutput`, and `grouped_topk`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `top_k`, `use_grouped_topk`, `topk_group`, `num_expert_group`, and `renormalize` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `select_experts`，并承载这一阶段的核心逻辑。 它主要调用 `expert_location_dispatch.transform_select_experts_inputs`、`_post_process_topk_ids`、`get_global_expert_distribution_recorder.on_select_experts`、`StandardTopKOutput` 以及 `grouped_topk`，说明该流程会编排底层辅助函数或计算内核。 像 `top_k`、`use_grouped_topk`、`topk_group`、`num_expert_group` 以及 `renormalize` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 1482-1524: Module-level helper logic
```python
if _is_cuda:

    @torch.library.register_fake("sgl_kernel::moe_fused_gate")
    def _moe_fused_gate(
        input_tensor,
        bias,
        num_expert_group,
        topk_group,
        topk,
        num_fused_shared_experts=0,
        routed_scaling_factor=0,
        apply_routed_scaling_factor_on_output=False,
    ):
        num_rows = input_tensor.shape[0]
        topk_weights = torch.empty(
            (num_rows, topk), dtype=torch.float32, device=input_tensor.device
        )
        topk_ids = torch.empty(
            (num_rows, topk), dtype=torch.int32, device=input_tensor.device
        )
        return topk_weights, topk_ids

    @register_fake_if_exists("sgl_kernel::kimi_k2_moe_fused_gate")
    def _kimi_k2_moe_fused_gate(
        input_tensor,
        bias,
        topk,
        renormalize,
        routed_scaling_factor,
        apply_routed_scaling_factor_on_output,
    ):
        num_rows = input_tensor.shape[0]
        topk_weights = input_tensor.new_empty(
            num_rows,
            topk,
            dtype=torch.float32,
        )
        topk_ids = input_tensor.new_empty(
            num_rows,
            topk,
            dtype=torch.int32,
        )
        return topk_weights, topk_ids
```
**EN:** This section prepares the module namespace. Shared names such as `num_rows`, `topk_weights`, and `topk_ids` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 像 `num_rows`、`topk_weights` 以及 `topk_ids` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `TopKConfig`, `TopKOutputChecker`, `TopKOutputFormat`, `TopKOutput`, and `StandardTopKOutput`. / **主要符号**：核心入口包括 `TopKConfig`、`TopKOutputChecker`、`TopKOutputFormat`、`TopKOutput` 以及 `StandardTopKOutput`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `math`, `dataclasses.dataclass`, `enum.IntEnum`, `enum.auto`, `typing.TYPE_CHECKING`, `typing.Callable`, `typing.NamedTuple`, and `typing.Optional` / **标准库**：`__future__.annotations`、`logging`、`math`、`dataclasses.dataclass`、`enum.IntEnum`、`enum.auto`、`typing.TYPE_CHECKING`、`typing.Callable`、`typing.NamedTuple` 以及 `typing.Optional`
- **Third-party**: `torch`, `torch.nn.functional`, `triton_kernels.matmul_ogs.GatherIndx`, `triton_kernels.matmul_ogs.RoutingData`, `triton_kernels.matmul_ogs.ScatterIndx`, `triton_kernels.tensor.make_ragged_tensor_metadata`, `triton_kernels.topk.topk`, `sgl_kernel.moe_fused_gate`, `sgl_kernel.topk_softmax`, and `flashinfer.fused_moe.fused_topk_deepseek` / **第三方依赖**：`torch`、`torch.nn.functional`、`triton_kernels.matmul_ogs.GatherIndx`、`triton_kernels.matmul_ogs.RoutingData`、`triton_kernels.matmul_ogs.ScatterIndx`、`triton_kernels.tensor.make_ragged_tensor_metadata`、`triton_kernels.topk.topk`、`sgl_kernel.moe_fused_gate`、`sgl_kernel.topk_softmax` 以及 `flashinfer.fused_moe.fused_topk_deepseek`
- **Internal SGLang modules**: `sglang.jit_kernel.deepseek_v4.mask_topk_ids`, `sglang.srt.distributed.get_moe_expert_parallel_rank`, `sglang.srt.distributed.get_moe_expert_parallel_world_size`, `sglang.srt.distributed.get_tp_group`, `sglang.srt.distributed.device_communicators.pynccl_allocator.use_symmetric_memory`, `sglang.srt.environ.envs`, `sglang.srt.eplb.expert_location_dispatch`, `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`, `sglang.srt.eplb.expert_location_dispatch.ExpertLocationDispatchInfo`, `sglang.srt.eplb.expert_location_dispatch.topk_ids_logical_to_physical`, `sglang.srt.layers.dp_attention.is_allocation_symmetric`, and `sglang.srt.layers.moe.get_moe_runner_backend` / **SGLang 内部模块**：`sglang.jit_kernel.deepseek_v4.mask_topk_ids`、`sglang.srt.distributed.get_moe_expert_parallel_rank`、`sglang.srt.distributed.get_moe_expert_parallel_world_size`、`sglang.srt.distributed.get_tp_group`、`sglang.srt.distributed.device_communicators.pynccl_allocator.use_symmetric_memory`、`sglang.srt.environ.envs`、`sglang.srt.eplb.expert_location_dispatch`、`sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`、`sglang.srt.eplb.expert_location_dispatch.ExpertLocationDispatchInfo`、`sglang.srt.eplb.expert_location_dispatch.topk_ids_logical_to_physical`、`sglang.srt.layers.dp_attention.is_allocation_symmetric` 以及 `sglang.srt.layers.moe.get_moe_runner_backend`
