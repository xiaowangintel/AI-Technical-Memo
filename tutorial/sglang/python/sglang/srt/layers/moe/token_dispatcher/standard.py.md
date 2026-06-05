# standard.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/token_dispatcher/standard.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `StandardDispatchOutput`, `StandardCombineInput`, and `StandardDispatcher` and connects them to backend-specific paths such as `CUDA`, `Triton`, `CUTLASS`, and `FlashInfer`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合混合专家路由与专家执行。它提供了 `StandardDispatchOutput`、`StandardCombineInput` 以及 `StandardDispatcher` 等符号，并把这些符号连接到 `CUDA`、`Triton`、`CUTLASS` 以及 `FlashInfer` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-58: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

from typing import TYPE_CHECKING, NamedTuple, Optional

import torch

from sglang.srt.distributed import (
    get_moe_expert_parallel_rank,
    get_moe_expert_parallel_world_size,
    get_tp_group,
)
from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    use_symmetric_memory,
)
from sglang.srt.layers.dp_attention import (
    get_dp_global_num_tokens,
    get_local_dp_buffer,
    is_allocation_symmetric,
)
from sglang.srt.layers.moe.moe_runner.base import MoeRunnerConfig
from sglang.srt.layers.moe.token_dispatcher.base import (
    BaseDispatcher,
    CombineInput,
    CombineInputFormat,
    DispatchOutput,
    DispatchOutputFormat,
)
from sglang.srt.layers.moe.topk import StandardTopKOutput, TopKOutput, TopKOutputChecker
from sglang.srt.layers.moe.utils import (
    get_moe_runner_backend,
    should_use_flashinfer_cutlass_moe_fp4_allgather,
)
from sglang.srt.utils.common import (
    get_bool_env_var,
    get_device,
    is_hip,
)

_is_hip = is_hip()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip

if TYPE_CHECKING:
    from sglang.srt.layers.moe.topk import TopKOutput


try:
    from flashinfer import (
        nvfp4_block_scale_interleave as nvfp4_block_scale_interleave_flashinfer,
    )

    from sglang.srt.layers.quantization.modelopt_quant import (
        fp4_quantize as fp4_quantize_flashinfer,
    )
except ImportError:
    fp4_quantize_flashinfer = None
    nvfp4_block_scale_interleave_flashinfer = None
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `typing.TYPE_CHECKING`, `typing.NamedTuple`, `typing.Optional`, `torch`, and `sglang.srt.distributed.get_moe_expert_parallel_rank`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_hip`, `_use_aiter`, `fp4_quantize_flashinfer`, and `nvfp4_block_scale_interleave_flashinfer` capture configuration, cached handles, or feature flags. Control structures like `If` and `Try` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`typing.TYPE_CHECKING`、`typing.NamedTuple`、`typing.Optional`、`torch` 以及 `sglang.srt.distributed.get_moe_expert_parallel_rank`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_hip`、`_use_aiter`、`fp4_quantize_flashinfer` 以及 `nvfp4_block_scale_interleave_flashinfer` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 和 `Try` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 59-65: Class `StandardDispatchOutput` declaration and shared state
```python
class StandardDispatchOutput(NamedTuple):
    """Standard dispatch output."""

    hidden_states: torch.Tensor
    hidden_states_scale: Optional[torch.Tensor]
    topk_output: TopKOutput
```
**EN:** This block introduces class `StandardDispatchOutput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Standard dispatch output.
**CN:** 该代码块引入类 `StandardDispatchOutput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 66-70: Function `StandardDispatchOutput.format` and its core logic
```python
    @property
    def format(self) -> DispatchOutputFormat:
        return DispatchOutputFormat.STANDARD
```
**EN:** This block defines `StandardDispatchOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `StandardDispatchOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 71-73: Module-level helper logic
```python
assert isinstance(StandardDispatchOutput, DispatchOutput)
```
**EN:** This section prepares the module namespace.
**CN:** 该部分负责准备模块命名空间。

### Lines 74-78: Class `StandardCombineInput` declaration and shared state
```python
class StandardCombineInput(NamedTuple):
    """Standard combine input."""

    hidden_states: torch.Tensor
```
**EN:** This block introduces class `StandardCombineInput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Standard combine input.
**CN:** 该代码块引入类 `StandardCombineInput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 79-83: Function `StandardCombineInput.format` and its core logic
```python
    @property
    def format(self) -> CombineInputFormat:
        return CombineInputFormat.STANDARD
```
**EN:** This block defines `StandardCombineInput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `StandardCombineInput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 84-86: Module-level helper logic
```python
assert isinstance(StandardCombineInput, CombineInput)
```
**EN:** This section prepares the module namespace.
**CN:** 该部分负责准备模块命名空间。

### Lines 87-88: Class `StandardDispatcher` declaration and shared state
```python
class StandardDispatcher(BaseDispatcher):
```
**EN:** This block introduces class `StandardDispatcher` and the state shared by its methods. It inherits from `BaseDispatcher`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `StandardDispatcher`，并定义其方法共享的状态。 它继承自 `BaseDispatcher`，说明了它在 SRT 层栈中的接入方式。

### Lines 89-114: `StandardDispatcher` initialization and state setup
```python
    def __init__(self, moe_runner_config: MoeRunnerConfig):
        super().__init__()
        self.moe_ep_size = get_moe_expert_parallel_world_size()
        backend = get_moe_runner_backend()
        self.enable_flashinfer_cutlass_moe = backend.is_flashinfer_cutlass()
        self.enable_flashinfer_mxfp4_moe = backend.is_flashinfer_mxfp4()
        self.enable_flashinfer_trtllm_routed_moe = backend.is_flashinfer_trtllm_routed()
        # Skip local expert mapping when the backend handles EP with global expert IDs:
        # - cutlass / cutedsl / trtllm_routed handle EP internally
        # - mxfp4 dispatcher mapping is already global
        self.skip_local_expert_mapping = (
            backend.is_flashinfer_cutlass()
            or backend.is_flashinfer_cutedsl()
            or backend.is_flashinfer_trtllm_routed()
            or self.enable_flashinfer_mxfp4_moe
        )
        self.num_experts = moe_runner_config.num_experts
        self.num_local_experts = moe_runner_config.num_local_experts
        self.num_local_shared_experts = moe_runner_config.num_fused_shared_experts
        self.num_local_routed_experts = (
            self.num_local_experts - self.num_local_shared_experts
        )
        self.moe_ep_rank = get_moe_expert_parallel_rank()
        self.local_expert_mapping = None
        self.expert_mask_gpu = None
```
**EN:** This block defines `StandardDispatcher.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `get_moe_expert_parallel_world_size`, `get_moe_runner_backend`, `backend.is_flashinfer_cutlass`, and `backend.is_flashinfer_mxfp4`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.moe_ep_size`, `backend`, `self.enable_flashinfer_cutlass_moe`, `self.enable_flashinfer_mxfp4_moe`, and `self.enable_flashinfer_trtllm_routed_moe` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `StandardDispatcher.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`get_moe_expert_parallel_world_size`、`get_moe_runner_backend`、`backend.is_flashinfer_cutlass` 以及 `backend.is_flashinfer_mxfp4`，说明该流程会编排底层辅助函数或计算内核。 像 `self.moe_ep_size`、`backend`、`self.enable_flashinfer_cutlass_moe`、`self.enable_flashinfer_mxfp4_moe` 以及 `self.enable_flashinfer_trtllm_routed_moe` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 115-219: Function `StandardDispatcher.dispatch` and its core logic
```python
    def dispatch(
        self, hidden_states: torch.Tensor, topk_output: TopKOutput
    ) -> StandardDispatchOutput:

        if should_use_flashinfer_cutlass_moe_fp4_allgather():
            # all-gather fp4 hidden states
            if (
                fp4_quantize_flashinfer is None
                or nvfp4_block_scale_interleave_flashinfer is None
            ):
                raise RuntimeError(
                    "FlashInfer fp4_quantize and nvfp4_block_scale_interleave "
                    "are required for the flashinfer_cutlass FP4 all-gather "
                    "path."
                )
            global_scale = self.quant_config.get("input_global_scale", None)
            assert global_scale is not None, "input_global_scale is not set"
            topk_weights, topk_ids = topk_output.topk_weights, topk_output.topk_ids

            # Quantize before comm, swizzle after.
            with use_symmetric_memory(
                get_tp_group(), disabled=not is_allocation_symmetric()
            ):
                if hidden_states.shape[0] > 0:
                    x, x_sf = fp4_quantize_flashinfer(
                        hidden_states, global_scale, is_sf_swizzled_layout=False
                    )
                else:
                    x_col = hidden_states.shape[1]
                    x = torch.zeros(
                        0, x_col // 2, dtype=torch.uint8, device=hidden_states.device
                    )
                    x_sf = torch.zeros(
                        0, x_col // 16, dtype=torch.uint8, device=hidden_states.device
                    )
            topk_weights, topk_ids, x, x_sf = get_tp_group().all_gatherv(
                [topk_weights, topk_ids, x, x_sf], sizes=get_dp_global_num_tokens()
            )
            # TODO: fuse into cutlass moe
            x_sf = nvfp4_block_scale_interleave_flashinfer(x_sf)

            hidden_states = x
            hidden_states_scale = x_sf
            topk_output = StandardTopKOutput(
                topk_weights=topk_weights,
                topk_ids=topk_ids,
                router_logits=topk_output.router_logits,  # never tested
            )
        else:
            hidden_states = hidden_states
            hidden_states_scale = None

        if (
            self.moe_ep_size > 1
            and not self.skip_local_expert_mapping
            and TopKOutputChecker.format_is_standard(topk_output)
        ):
            if self.local_expert_mapping is None:
                device = get_device()
                self.local_expert_mapping = torch.full(
                    (self.num_experts,), -1, dtype=torch.int32, device=device
                )
                self.local_expert_mapping[
                    self.moe_ep_rank
                    * self.num_local_routed_experts : (self.moe_ep_rank + 1)
                    * self.num_local_routed_experts
                ] = torch.arange(
                    0, self.num_local_routed_experts, dtype=torch.int32, device=device
                )

                if self.num_local_shared_experts > 0:
                    self.local_expert_mapping[-self.num_local_shared_experts :] = (
                        torch.arange(
                            self.num_local_routed_experts,
                            self.num_local_routed_experts
                            + self.num_local_shared_experts,
                            dtype=torch.int32,
                            device="cpu",
                        )
                    )

        if self.local_expert_mapping is not None and not self.skip_local_expert_mapping:
            if _use_aiter:
                self.expert_mask_gpu = (
                    (
                        (self.local_expert_mapping >= 0)
                        & (self.local_expert_mapping < self.num_local_experts)
                    )
                    .to(torch.int32)
                    .to(device="cuda")
                )
            else:
                if TopKOutputChecker.format_is_standard(topk_output):
                    topk_output = topk_output._replace(
                        topk_ids=self.local_expert_mapping[topk_output.topk_ids]
                    )
                elif TopKOutputChecker.format_is_triton_kernels(topk_output):
                    raise NotImplementedError()

        return StandardDispatchOutput(
            hidden_states=hidden_states,
            hidden_states_scale=hidden_states_scale,
            topk_output=topk_output,
        )
```
**EN:** This block defines `StandardDispatcher.dispatch` and contains the main logic for this step. It mainly invokes `should_use_flashinfer_cutlass_moe_fp4_allgather`, `StandardDispatchOutput`, `self.quant_config.get`, `get_tp_group.all_gatherv`, and `nvfp4_block_scale_interleave_flashinfer`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `global_scale`, `topk_weights`, `topk_ids`, `x`, and `x_sf` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `StandardDispatcher.dispatch`，并承载这一阶段的核心逻辑。 它主要调用 `should_use_flashinfer_cutlass_moe_fp4_allgather`、`StandardDispatchOutput`、`self.quant_config.get`、`get_tp_group.all_gatherv` 以及 `nvfp4_block_scale_interleave_flashinfer`，说明该流程会编排底层辅助函数或计算内核。 像 `global_scale`、`topk_weights`、`topk_ids`、`x` 以及 `x_sf` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 220-232: Function `StandardDispatcher.combine` and its core logic
```python
    def combine(self, combine_input: StandardCombineInput) -> torch.Tensor:
        (hidden_states,) = combine_input
        if should_use_flashinfer_cutlass_moe_fp4_allgather():
            hidden_states, global_hidden_states = (
                get_local_dp_buffer(get_tp_group()),
                hidden_states,
            )
            get_tp_group().reduce_scatterv(
                global_hidden_states,
                output=hidden_states,
                sizes=get_dp_global_num_tokens(),
            )
        return hidden_states
```
**EN:** This block defines `StandardDispatcher.combine` and contains the main logic for this step. It mainly invokes `should_use_flashinfer_cutlass_moe_fp4_allgather`, `get_tp_group.reduce_scatterv`, `get_local_dp_buffer`, `get_tp_group`, and `get_dp_global_num_tokens`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states` and `global_hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `StandardDispatcher.combine`，并承载这一阶段的核心逻辑。 它主要调用 `should_use_flashinfer_cutlass_moe_fp4_allgather`、`get_tp_group.reduce_scatterv`、`get_local_dp_buffer`、`get_tp_group` 以及 `get_dp_global_num_tokens`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states` 和 `global_hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `StandardDispatchOutput`, `StandardCombineInput`, and `StandardDispatcher`. / **主要符号**：核心入口包括 `StandardDispatchOutput`、`StandardCombineInput` 以及 `StandardDispatcher`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `typing.TYPE_CHECKING`, `typing.NamedTuple`, and `typing.Optional` / **标准库**：`__future__.annotations`、`typing.TYPE_CHECKING`、`typing.NamedTuple` 以及 `typing.Optional`
- **Third-party**: `torch` and `flashinfer.nvfp4_block_scale_interleave` / **第三方依赖**：`torch` 和 `flashinfer.nvfp4_block_scale_interleave`
- **Internal SGLang modules**: `sglang.srt.distributed.get_moe_expert_parallel_rank`, `sglang.srt.distributed.get_moe_expert_parallel_world_size`, `sglang.srt.distributed.get_tp_group`, `sglang.srt.distributed.device_communicators.pynccl_allocator.use_symmetric_memory`, `sglang.srt.layers.dp_attention.get_dp_global_num_tokens`, `sglang.srt.layers.dp_attention.get_local_dp_buffer`, `sglang.srt.layers.dp_attention.is_allocation_symmetric`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`, `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`, and `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput` / **SGLang 内部模块**：`sglang.srt.distributed.get_moe_expert_parallel_rank`、`sglang.srt.distributed.get_moe_expert_parallel_world_size`、`sglang.srt.distributed.get_tp_group`、`sglang.srt.distributed.device_communicators.pynccl_allocator.use_symmetric_memory`、`sglang.srt.layers.dp_attention.get_dp_global_num_tokens`、`sglang.srt.layers.dp_attention.get_local_dp_buffer`、`sglang.srt.layers.dp_attention.is_allocation_symmetric`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`、`sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat` 以及 `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`
