# flashinfer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/token_dispatcher/flashinfer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `FlashinferDispatchOutput`, `FlashinferCombineInput`, and `FlashinferDispatcher` and connects them to backend-specific paths such as `CUDA`, `CUTLASS`, `FlashInfer`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合混合专家路由与专家执行。它提供了 `FlashinferDispatchOutput`、`FlashinferCombineInput` 以及 `FlashinferDispatcher` 等符号，并把这些符号连接到 `CUDA`、`CUTLASS`、`FlashInfer` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-43: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import logging
from typing import NamedTuple, Optional

import torch

from sglang.kernel_api_logging import debug_kernel_api
from sglang.srt.environ import envs
from sglang.srt.layers.dp_attention import get_dp_global_num_tokens
from sglang.srt.layers.moe.token_dispatcher import (
    BaseDispatcher,
    CombineInput,
    CombineInputFormat,
    DispatchOutput,
    DispatchOutputFormat,
)
from sglang.srt.layers.moe.token_dispatcher.flashinfer_utils import (
    TorchDistributedCommBackend,
)
from sglang.srt.layers.moe.topk import StandardTopKOutput, TopKOutput
from sglang.srt.layers.moe.utils import get_moe_runner_backend
from sglang.srt.server_args import get_global_server_args
from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
from sglang.srt.utils import get_int_env_var

try:
    from flashinfer import nvfp4_block_scale_interleave
    from flashinfer.comm import MoeAlltoAll, moe_a2a_get_workspace_size_per_rank
    from flashinfer.comm.mapping import Mapping
    from flashinfer.comm.mnnvl import MnnvlConfig

    from sglang.srt.layers.quantization.fp4_utils import fp4_quantize

    use_flashinfer = True
except ImportError:
    use_flashinfer = False

logger = logging.getLogger(__name__)

MOE_NVFP4_DISPATCH = envs.SGLANG_MOE_NVFP4_DISPATCH.get()
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `typing.NamedTuple`, `typing.Optional`, `torch`, and `sglang.kernel_api_logging.debug_kernel_api`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `use_flashinfer`, `logger`, and `MOE_NVFP4_DISPATCH` capture configuration, cached handles, or feature flags. Control structures like `Try` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`typing.NamedTuple`、`typing.Optional`、`torch` 以及 `sglang.kernel_api_logging.debug_kernel_api`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `use_flashinfer`、`logger` 以及 `MOE_NVFP4_DISPATCH` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `Try` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 44-52: Class `FlashinferDispatchOutput` declaration and shared state
```python
class FlashinferDispatchOutput(NamedTuple):
    """Flashinfer EP dispatch output."""

    hidden_states: torch.Tensor
    hidden_states_scale: Optional[torch.Tensor]
    topk_output: StandardTopKOutput
    # Provide an output tensor to fused_moe so it writes directly to our buffer
    moe_output: Optional[torch.Tensor] = None
```
**EN:** This block introduces class `FlashinferDispatchOutput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Flashinfer EP dispatch output.
**CN:** 该代码块引入类 `FlashinferDispatchOutput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 53-57: Function `FlashinferDispatchOutput.format` and its core logic
```python
    @property
    def format(self) -> DispatchOutputFormat:
        return DispatchOutputFormat.FLASHINFER
```
**EN:** This block defines `FlashinferDispatchOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FlashinferDispatchOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 58-60: Module-level helper logic
```python
assert isinstance(FlashinferDispatchOutput, DispatchOutput)
```
**EN:** This section prepares the module namespace.
**CN:** 该部分负责准备模块命名空间。

### Lines 61-65: Class `FlashinferCombineInput` declaration and shared state
```python
class FlashinferCombineInput(NamedTuple):
    """Flashinfer combine input."""

    hidden_states: torch.Tensor
```
**EN:** This block introduces class `FlashinferCombineInput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Flashinfer combine input.
**CN:** 该代码块引入类 `FlashinferCombineInput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 66-70: Function `FlashinferCombineInput.format` and its core logic
```python
    @property
    def format(self) -> CombineInputFormat:
        return CombineInputFormat.FLASHINFER
```
**EN:** This block defines `FlashinferCombineInput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FlashinferCombineInput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 71-73: Module-level helper logic
```python
assert isinstance(FlashinferCombineInput, CombineInput)
```
**EN:** This section prepares the module namespace.
**CN:** 该部分负责准备模块命名空间。

### Lines 74-76: Class `FlashinferDispatcher` declaration and shared state
```python
class FlashinferDispatcher(BaseDispatcher):
    """Main dispatcher class for Flashinfer A2A backend."""
```
**EN:** This block introduces class `FlashinferDispatcher` and the state shared by its methods. It inherits from `BaseDispatcher`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Main dispatcher class for Flashinfer A2A backend.
**CN:** 该代码块引入类 `FlashinferDispatcher`，并定义其方法共享的状态。 它继承自 `BaseDispatcher`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 77-172: `FlashinferDispatcher` initialization and state setup
```python
    def __init__(
        self,
        group: torch.distributed.ProcessGroup,
        router_topk: int,
        num_experts: int = None,
        num_local_experts: int = None,  # Unused
        hidden_size: int = None,
        params_dtype: torch.dtype = None,  # Unused
    ):
        super().__init__()
        if not use_flashinfer:
            raise ImportError(
                "Flashinfer is not installed or does not support A2A. "
                "Please install the appropriate version of Flashinfer."
            )

        self.ep_size = group.size()
        self.ep_rank = group.rank()
        self.router_topk = router_topk
        self.hidden_size = hidden_size
        self.num_experts = num_experts
        self.num_local_experts = num_local_experts

        # TODO: Can other moe runners use payload_in_workspace too?
        self.payload_in_workspace = get_moe_runner_backend().is_flashinfer_cutlass()

        # TODO: Can this be a server arg and shared with deepep/mooncakeep?
        self.max_num_tokens = (
            get_int_env_var("SGLANG_FLASHINFER_NUM_MAX_DISPATCH_TOKENS_PER_RANK", 1024)
            * self.ep_size
        )

        # Calculate workspace size. For eagle mode, use the larger workspace size since nextn layer will be unquantized.
        speculative_algo = SpeculativeAlgorithm.from_string(
            get_global_server_args().speculative_algorithm
        )
        if MOE_NVFP4_DISPATCH and not speculative_algo.is_eagle():
            total_dispatch_payload_size_per_token = (
                hidden_size // 2  # nvfp4 hidden states
                + hidden_size // 16  # fp8 scaling factors
                + self.router_topk * 4  # int32 topks ids
                + self.router_topk * 4  # float32 topk weights
            )
        else:
            total_dispatch_payload_size_per_token = (
                hidden_size * 2  # bf16 hidden states
                + self.router_topk * 4  # int32 topks ids
                + self.router_topk * 4  # float32 topk weights
            )
        combine_payload_size_per_token = hidden_size * 2  # bf16 hidden states
        self.workspace_size = moe_a2a_get_workspace_size_per_rank(
            ep_size=self.ep_size,
            max_num_tokens=self.max_num_tokens,
            total_dispatch_payload_size_per_token=total_dispatch_payload_size_per_token,
            combine_payload_size_per_token=combine_payload_size_per_token,
        )

        self.mapping = Mapping(
            rank=self.ep_rank,
            tp_size=self.ep_size,
            moe_ep_size=self.ep_size,
            world_size=self.ep_size,
            gpus_per_node=torch.cuda.device_count(),
            pp_size=1,
            cp_size=1,
        )
        self.moe_a2a = MoeAlltoAll(
            mapping=self.mapping,
            max_num_tokens=self.max_num_tokens,
            top_k=self.router_topk,
            num_experts=self.num_experts,
            workspace_size_per_rank=self.workspace_size,
            mnnvl_config=MnnvlConfig(comm_backend=TorchDistributedCommBackend(group)),
        )

        # Preallocate dummy tensors (to overcome numLocalTokens > 0 restriction)
        self.dummy_x = torch.empty(
            (1, hidden_size),
            dtype=torch.bfloat16,
            device="cuda",
        )
        # -1 will be ignored by flashinfer cutlass moe
        self.dummy_topk_ids = torch.full(
            (1, self.router_topk), -1, dtype=torch.int32, device="cuda"
        )
        # Hack for dispatch with dummy token - will route the dummy token to this rank so it doesn't require any transfer.
        self.dummy_topk_ids_current_rank = torch.full(
            (1, self.router_topk),
            self.ep_rank * self.num_local_experts,
            dtype=torch.int32,
            device="cuda",
        )
        self.dummy_topk_weights = torch.zeros(
            (1, self.router_topk), dtype=torch.float32, device="cuda"
        )
```
**EN:** This block defines `FlashinferDispatcher.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `group.size`, `group.rank`, `get_moe_runner_backend.is_flashinfer_cutlass`, and `SpeculativeAlgorithm.from_string`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.ep_size`, `self.ep_rank`, `self.router_topk`, `self.hidden_size`, and `self.num_experts` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `FlashinferDispatcher.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`group.size`、`group.rank`、`get_moe_runner_backend.is_flashinfer_cutlass` 以及 `SpeculativeAlgorithm.from_string`，说明该流程会编排底层辅助函数或计算内核。 像 `self.ep_size`、`self.ep_rank`、`self.router_topk`、`self.hidden_size` 以及 `self.num_experts` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 173-250: Function `FlashinferDispatcher.dispatch` and its core logic
```python
    @debug_kernel_api
    def dispatch(
        self, hidden_states: torch.Tensor, topk_output: TopKOutput
    ) -> FlashinferDispatchOutput:
        output_dtype = hidden_states.dtype
        x = hidden_states
        x_sf = None
        topk_ids = topk_output.topk_ids
        topk_weights = topk_output.topk_weights

        # Handle case where there are no tokens on this DP worker
        # moe_a2a.dispatch requires at least one token
        self.has_dummy_token = False
        if x.shape[0] == 0:
            logger.warning("No tokens on this DP worker, using dummy token")
            self.has_dummy_token = True
            x = self.dummy_x
            topk_ids = self.dummy_topk_ids
            topk_weights = self.dummy_topk_weights

        global_scale = self.quant_config.get("input_global_scale", None)
        if global_scale is not None:
            if x.shape[0] > 0:
                x, x_sf = fp4_quantize(x, global_scale, is_sf_swizzled_layout=False)
            else:
                x = torch.zeros(
                    0, self.hidden_size // 2, dtype=torch.uint8, device=x.device
                )
                x_sf = torch.zeros(
                    0, self.hidden_size // 16, dtype=torch.uint8, device=x.device
                )

        payloads = []
        payloads.append(x)
        if x_sf is not None:
            payloads.append(x_sf)
            expert_id_payload_index = 2
        else:
            expert_id_payload_index = 1
        payloads.append(topk_ids)
        payloads.append(topk_weights)

        self.runtime_max_tokens_per_rank = (
            max(get_dp_global_num_tokens())
            if get_dp_global_num_tokens() is not None
            else x.shape[0]
        )
        recv_tensors = self.moe_a2a.dispatch(
            self.dummy_topk_ids_current_rank if self.has_dummy_token else topk_ids,
            payloads,
            self.runtime_max_tokens_per_rank,
            invalid_token_expert_id=-1,
            expert_id_payload_index=expert_id_payload_index,
        )
        if x_sf is not None:
            x_recv, x_sf_recv, topk_ids_recv, topk_weights_recv = recv_tensors
            x_sf = x_sf_recv.view(-1, x_sf_recv.shape[-1])
            # TODO: fuse interleave into cutlass moe
            x_sf = nvfp4_block_scale_interleave(x_sf)
        else:
            x_recv, topk_ids_recv, topk_weights_recv = recv_tensors
        x = x_recv.view(-1, x_recv.shape[-1])
        topk_ids = topk_ids_recv.view(-1, topk_ids_recv.shape[-1])
        topk_weights = topk_weights_recv.view(-1, topk_weights_recv.shape[-1])

        # Provide an output tensor to fused_moe so it writes directly to our buffer
        moe_output = None
        if self.payload_in_workspace:
            moe_output = self.moe_a2a.get_combine_payload_tensor_in_workspace(
                self.runtime_max_tokens_per_rank, self.hidden_size, output_dtype
            ).view(-1, self.hidden_size)
        return FlashinferDispatchOutput(
            x,
            x_sf,
            StandardTopKOutput(topk_weights, topk_ids, topk_output.router_logits),
            moe_output,
        )
```
**EN:** This block defines `FlashinferDispatcher.dispatch` and contains the main logic for this step. Decorators like `debug_kernel_api` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `self.quant_config.get`, `payloads.append`, `self.moe_a2a.dispatch`, `x_recv.view`, and `topk_ids_recv.view`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output_dtype`, `x`, `x_sf`, `topk_ids`, and `topk_weights` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FlashinferDispatcher.dispatch`，并承载这一阶段的核心逻辑。 像 `debug_kernel_api` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `self.quant_config.get`、`payloads.append`、`self.moe_a2a.dispatch`、`x_recv.view` 以及 `topk_ids_recv.view`，说明该流程会编排底层辅助函数或计算内核。 像 `output_dtype`、`x`、`x_sf`、`topk_ids` 以及 `topk_weights` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 251-269: Function `FlashinferDispatcher.combine` and its core logic
```python
    @debug_kernel_api
    def combine(self, combine_input: FlashinferCombineInput) -> torch.Tensor:
        hidden_states = combine_input.hidden_states
        output_hidden_size = hidden_states.shape[-1]
        hidden_states = self.moe_a2a.combine(
            hidden_states.view(
                self.ep_size, self.runtime_max_tokens_per_rank, output_hidden_size
            ),
            self.runtime_max_tokens_per_rank,
            payload_in_workspace=self.payload_in_workspace,
        )

        # Remove dummy token if it was added in dispatch
        if self.has_dummy_token:
            hidden_states = hidden_states[1:, :]

        del self.runtime_max_tokens_per_rank
        del self.has_dummy_token
        return hidden_states
```
**EN:** This block defines `FlashinferDispatcher.combine` and contains the main logic for this step. Decorators like `debug_kernel_api` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `self.moe_a2a.combine` and `hidden_states.view`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states` and `output_hidden_size` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FlashinferDispatcher.combine`，并承载这一阶段的核心逻辑。 像 `debug_kernel_api` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `self.moe_a2a.combine` 和 `hidden_states.view`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states` 和 `output_hidden_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `FlashinferDispatchOutput`, `FlashinferCombineInput`, and `FlashinferDispatcher`. / **主要符号**：核心入口包括 `FlashinferDispatchOutput`、`FlashinferCombineInput` 以及 `FlashinferDispatcher`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `typing.NamedTuple`, and `typing.Optional` / **标准库**：`__future__.annotations`、`logging`、`typing.NamedTuple` 以及 `typing.Optional`
- **Third-party**: `torch`, `flashinfer.nvfp4_block_scale_interleave`, `flashinfer.comm.MoeAlltoAll`, `flashinfer.comm.moe_a2a_get_workspace_size_per_rank`, `flashinfer.comm.mapping.Mapping`, and `flashinfer.comm.mnnvl.MnnvlConfig` / **第三方依赖**：`torch`、`flashinfer.nvfp4_block_scale_interleave`、`flashinfer.comm.MoeAlltoAll`、`flashinfer.comm.moe_a2a_get_workspace_size_per_rank`、`flashinfer.comm.mapping.Mapping` 以及 `flashinfer.comm.mnnvl.MnnvlConfig`
- **Internal SGLang modules**: `sglang.kernel_api_logging.debug_kernel_api`, `sglang.srt.environ.envs`, `sglang.srt.layers.dp_attention.get_dp_global_num_tokens`, `sglang.srt.layers.moe.token_dispatcher.BaseDispatcher`, `sglang.srt.layers.moe.token_dispatcher.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.CombineInputFormat`, `sglang.srt.layers.moe.token_dispatcher.DispatchOutput`, `sglang.srt.layers.moe.token_dispatcher.DispatchOutputFormat`, `sglang.srt.layers.moe.token_dispatcher.flashinfer_utils.TorchDistributedCommBackend`, `sglang.srt.layers.moe.topk.StandardTopKOutput`, `sglang.srt.layers.moe.topk.TopKOutput`, and `sglang.srt.layers.moe.utils.get_moe_runner_backend` / **SGLang 内部模块**：`sglang.kernel_api_logging.debug_kernel_api`、`sglang.srt.environ.envs`、`sglang.srt.layers.dp_attention.get_dp_global_num_tokens`、`sglang.srt.layers.moe.token_dispatcher.BaseDispatcher`、`sglang.srt.layers.moe.token_dispatcher.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.CombineInputFormat`、`sglang.srt.layers.moe.token_dispatcher.DispatchOutput`、`sglang.srt.layers.moe.token_dispatcher.DispatchOutputFormat`、`sglang.srt.layers.moe.token_dispatcher.flashinfer_utils.TorchDistributedCommBackend`、`sglang.srt.layers.moe.topk.StandardTopKOutput`、`sglang.srt.layers.moe.topk.TopKOutput` 以及 `sglang.srt.layers.moe.utils.get_moe_runner_backend`
