# nixl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/token_dispatcher/nixl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `NixlEPBuffer`, `_NixlEPDispatcherImplBase`, `_NixlEPDispatcherImpl`, and `_Stage` and connects them to backend-specific paths such as `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合混合专家路由与专家执行。它提供了 `NixlEPBuffer`、`_NixlEPDispatcherImplBase`、`_NixlEPDispatcherImpl` 以及 `_Stage` 等符号，并把这些符号连接到 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

import logging
from enum import Enum, auto
from typing import Optional

import torch
import torch.distributed as dist

from sglang.srt.distributed.utils import get_global_tcp_store
from sglang.srt.elastic_ep.elastic_ep import ElasticEPStateManager
from sglang.srt.environ import envs
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.layers import deep_gemm_wrapper
from sglang.srt.layers.dp_attention import get_is_extend_in_batch
from sglang.srt.layers.moe.token_dispatcher.base import (
    BaseDispatcher,
    CombineInput,
    DispatchOutput,
)
from sglang.srt.layers.moe.token_dispatcher.deepep import (
    DeepEPLLCombineInput,
    DeepEPLLDispatchOutput,
)
from sglang.srt.layers.moe.topk import TopKOutput
from sglang.srt.layers.moe.utils import DeepEPMode

try:
    from nixl_ep import Buffer

    use_nixl = True
except ImportError:
    use_nixl = False

logger = logging.getLogger(__name__)

NixlEPDispatchOutput = DeepEPLLDispatchOutput
NixlEPCombineInput = DeepEPLLCombineInput
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `enum.Enum`, `enum.auto`, `typing.Optional`, and `torch`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `use_nixl`, `logger`, `NixlEPDispatchOutput`, and `NixlEPCombineInput` capture configuration, cached handles, or feature flags. Control structures like `Try` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`enum.Enum`、`enum.auto`、`typing.Optional` 以及 `torch`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `use_nixl`、`logger`、`NixlEPDispatchOutput` 以及 `NixlEPCombineInput` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `Try` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 41-47: Class `NixlEPBuffer` declaration and shared state
```python
class NixlEPBuffer:
    _buffer = None
    _hidden_size: Optional[int] = None
    _num_max_dispatch_tokens_per_rank: Optional[int] = None
    _num_experts: Optional[int] = None
    _num_local_experts: Optional[int] = None
```
**EN:** This block introduces class `NixlEPBuffer` and the state shared by its methods.
**CN:** 该代码块引入类 `NixlEPBuffer`，并定义其方法共享的状态。

### Lines 48-109: `NixlEPBuffer.get_nixl_buffer` getter for nixl buffer
```python
    @classmethod
    def get_nixl_buffer(
        cls,
        group: dist.ProcessGroup,
        hidden_size: int,
        deepep_mode: DeepEPMode,
        num_max_dispatch_tokens_per_rank: int = -1,
        num_experts: int = -1,
        num_local_experts: int = -1,
    ):
        if cls._buffer is not None:
            return cls._buffer

        cls._hidden_size = hidden_size
        cls._num_max_dispatch_tokens_per_rank = num_max_dispatch_tokens_per_rank
        cls._num_experts = num_experts
        cls._num_local_experts = num_local_experts

        num_rdma_bytes = 0
        if deepep_mode.enable_normal():
            raise NotImplementedError("Normal mode is not supported for Nixl EP yet.")
        if deepep_mode.enable_low_latency():
            assert num_max_dispatch_tokens_per_rank != -1
            assert num_experts != -1 and num_experts % group.size() == 0
            num_rdma_bytes = Buffer.get_rdma_size_hint(
                num_max_dispatch_tokens_per_rank,
                hidden_size,
                group.size(),
                num_experts,
            )

        rank = dist.get_rank(group)
        world_size = dist.get_world_size(group)

        # Get the global TCPStore for coordination
        tcp_store = get_global_tcp_store()
        if tcp_store is None:
            raise RuntimeError(
                "Global TCPStore is not initialized. "
                "Make sure init_distributed_environment was called before using NIXL EP."
            )

        logger.info(
            f"Using NIXL EP (world_size={world_size}, rank={rank}, "
            f"num_experts={cls._num_experts}, num_experts_per_rank={cls._num_local_experts}) "
        )

        cls._buffer = Buffer(
            rank=rank,
            tcp_store_group=tcp_store,
        )

        cls._buffer.update_memory_buffers(
            num_ranks=world_size,
            num_experts_per_rank=cls._num_local_experts,
            num_rdma_bytes=num_rdma_bytes,
        )
        all_ranks = list(range(world_size))
        cls._buffer.connect_ranks(all_ranks)

        return cls._buffer
```
**EN:** This block defines `NixlEPBuffer.get_nixl_buffer` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `deepep_mode.enable_normal`, `deepep_mode.enable_low_latency`, `dist.get_rank`, `dist.get_world_size`, and `get_global_tcp_store`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_rdma_bytes`, `rank`, `world_size`, `tcp_store`, and `all_ranks` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NixlEPBuffer.get_nixl_buffer`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `deepep_mode.enable_normal`、`deepep_mode.enable_low_latency`、`dist.get_rank`、`dist.get_world_size` 以及 `get_global_tcp_store`，说明该流程会编排底层辅助函数或计算内核。 像 `num_rdma_bytes`、`rank`、`world_size`、`tcp_store` 以及 `all_ranks` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 110-118: Function `NixlEPBuffer.clean_buffer` and its core logic
```python
    @classmethod
    def clean_buffer(cls):
        cls._buffer.clean_buffer(
            cls._num_max_dispatch_tokens_per_rank,
            cls._hidden_size,
            cls._num_experts,
        )
```
**EN:** This block defines `NixlEPBuffer.clean_buffer` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `cls._buffer.clean_buffer`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `NixlEPBuffer.clean_buffer`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `cls._buffer.clean_buffer`，说明该流程会编排底层辅助函数或计算内核。

### Lines 119-119: Class `_NixlEPDispatcherImplBase` declaration and shared state
```python
class _NixlEPDispatcherImplBase:
```
**EN:** This block introduces class `_NixlEPDispatcherImplBase` and the state shared by its methods.
**CN:** 该代码块引入类 `_NixlEPDispatcherImplBase`，并定义其方法共享的状态。

### Lines 120-166: `_NixlEPDispatcherImplBase` initialization and state setup
```python
    def __init__(
        self,
        group: torch.distributed.ProcessGroup,
        router_topk: int,
        permute_fusion: bool,
        num_experts: int,
        num_local_experts: int,
        hidden_size: int,
        params_dtype: torch.dtype,
        deepep_mode: DeepEPMode,
    ):
        if not use_nixl:
            raise ImportError(
                "NixlEP is not installed. Please install NixlEP package from "
                "https://github.com/ai-dynamo/nixl."
            )

        self.group = group
        self.router_topk = router_topk
        self.permute_fusion = permute_fusion
        self.num_experts = num_experts
        self.num_local_experts = num_local_experts
        self.hidden_size = hidden_size
        self.params_dtype = params_dtype
        self.deepep_mode = deepep_mode

        self.num_max_dispatch_tokens_per_rank = (
            envs.SGLANG_NIXL_EP_NUM_MAX_DISPATCH_TOKENS_PER_RANK.get()
        )
        # NixlEP internode_ll dispatch uses FINISHED_SUM_TAG=1024
        # and the logic requires num-tokens-sent-from-one-rank-to-another-rank less than it
        assert self.num_max_dispatch_tokens_per_rank <= 1024
        elastic_state = ElasticEPStateManager.instance()
        self.active_ranks = (
            elastic_state.active_ranks if elastic_state is not None else None
        )
        self._mask_buffer = (
            torch.zeros_like(self.active_ranks)
            if self.active_ranks is not None
            else None
        )

        self.handle = None
        self.quant_config = None
        self.overlap_args = None
        self.meta_overlap_args = None
```
**EN:** This block defines `_NixlEPDispatcherImplBase.__init__` and contains the main logic for this step. It mainly invokes `envs.SGLANG_NIXL_EP_NUM_MAX_DISPATCH_TOKENS_PER_RANK.get`, `ElasticEPStateManager.instance`, `ImportError`, and `torch.zeros_like`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.group`, `self.router_topk`, `self.permute_fusion`, `self.num_experts`, and `self.num_local_experts` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_NixlEPDispatcherImplBase.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `envs.SGLANG_NIXL_EP_NUM_MAX_DISPATCH_TOKENS_PER_RANK.get`、`ElasticEPStateManager.instance`、`ImportError` 以及 `torch.zeros_like`，说明该流程会编排底层辅助函数或计算内核。 像 `self.group`、`self.router_topk`、`self.permute_fusion`、`self.num_experts` 以及 `self.num_local_experts` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 167-169: `_NixlEPDispatcherImplBase.set_quant_config` setter for quantization config
```python
    def set_quant_config(self, quant_config: dict) -> None:
        self.quant_config = quant_config
```
**EN:** This block defines `_NixlEPDispatcherImplBase.set_quant_config` and contains the main logic for this step. Intermediate names such as `self.quant_config` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_NixlEPDispatcherImplBase.set_quant_config`，并承载这一阶段的核心逻辑。 像 `self.quant_config` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 170-173: `_NixlEPDispatcherImplBase.set_overlap_args` setter for overlap args
```python
    def set_overlap_args(self, combine_overlap_args, meta_overlap_args) -> None:
        self.overlap_args = combine_overlap_args
        self.meta_overlap_args = meta_overlap_args
```
**EN:** This block defines `_NixlEPDispatcherImplBase.set_overlap_args` and contains the main logic for this step. Intermediate names such as `self.overlap_args` and `self.meta_overlap_args` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_NixlEPDispatcherImplBase.set_overlap_args`，并承载这一阶段的核心逻辑。 像 `self.overlap_args` 和 `self.meta_overlap_args` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 174-180: `_NixlEPDispatcherImplBase.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        raise NotImplementedError
```
**EN:** This block defines `_NixlEPDispatcherImplBase.dispatch_a` and contains the main logic for this step.
**CN:** 该代码块定义了 `_NixlEPDispatcherImplBase.dispatch_a`，并承载这一阶段的核心逻辑。

### Lines 181-183: `_NixlEPDispatcherImplBase.dispatch_b` step for b
```python
    def dispatch_b(self, *args, **kwargs):
        raise NotImplementedError
```
**EN:** This block defines `_NixlEPDispatcherImplBase.dispatch_b` and contains the main logic for this step.
**CN:** 该代码块定义了 `_NixlEPDispatcherImplBase.dispatch_b`，并承载这一阶段的核心逻辑。

### Lines 184-191: Function `_NixlEPDispatcherImplBase.combine_a` and its core logic
```python
    def combine_a(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
    ):
        raise NotImplementedError
```
**EN:** This block defines `_NixlEPDispatcherImplBase.combine_a` and contains the main logic for this step.
**CN:** 该代码块定义了 `_NixlEPDispatcherImplBase.combine_a`，并承载这一阶段的核心逻辑。

### Lines 192-194: Function `_NixlEPDispatcherImplBase.combine_b` and its core logic
```python
    def combine_b(self, *args, **kwargs):
        raise NotImplementedError
```
**EN:** This block defines `_NixlEPDispatcherImplBase.combine_b` and contains the main logic for this step.
**CN:** 该代码块定义了 `_NixlEPDispatcherImplBase.combine_b`，并承载这一阶段的核心逻辑。

### Lines 195-198: Internal helper `_NixlEPDispatcherImplBase._get_buffer`
```python
    def _get_buffer(self):
        raise NotImplementedError
```
**EN:** This block defines `_NixlEPDispatcherImplBase._get_buffer` and contains the main logic for this step.
**CN:** 该代码块定义了 `_NixlEPDispatcherImplBase._get_buffer`，并承载这一阶段的核心逻辑。

### Lines 199-199: Class `_NixlEPDispatcherImpl` declaration and shared state
```python
class _NixlEPDispatcherImpl(_NixlEPDispatcherImplBase):
```
**EN:** This block introduces class `_NixlEPDispatcherImpl` and the state shared by its methods. It inherits from `_NixlEPDispatcherImplBase`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_NixlEPDispatcherImpl`，并定义其方法共享的状态。 它继承自 `_NixlEPDispatcherImplBase`，说明了它在 SRT 层栈中的接入方式。

### Lines 200-209: `_NixlEPDispatcherImpl` initialization and state setup
```python
    def __init__(self, return_recv_hook: bool, **kwargs):
        super().__init__(**kwargs)

        """
        num_max_dispatch_tokens_per_rank: the actual batch size in the decoding engine should be less than 256
        https://github.com/ai-dynamo/nixl
        """
        self.return_recv_hook = return_recv_hook
        self.device_module = torch.get_device_module()
```
**EN:** This block defines `_NixlEPDispatcherImpl.__init__` and contains the main logic for this step. It mainly invokes `super.__init__` and `torch.get_device_module`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.return_recv_hook` and `self.device_module` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_NixlEPDispatcherImpl.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__` 和 `torch.get_device_module`，说明该流程会编排底层辅助函数或计算内核。 像 `self.return_recv_hook` 和 `self.device_module` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 210-235: `_NixlEPDispatcherImpl.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        buffer = self._get_buffer()
        topk_weights, topk_ids = topk_output.topk_weights, topk_output.topk_ids
        topk_ids = topk_ids.to(torch.int64)
        expected_m = (
            hidden_states.shape[0] * buffer.group_size * topk_ids.shape[1]
            + self.num_experts
        ) // self.num_experts
        hidden_states, masked_m, event, hook = self._dispatch_core(
            hidden_states,
            topk_ids,
        )
        return (
            hidden_states,
            topk_ids,
            topk_weights,
            masked_m,
            expected_m,
            event,
            hook,
        )
```
**EN:** This block defines `_NixlEPDispatcherImpl.dispatch_a` and contains the main logic for this step. It mainly invokes `self._get_buffer`, `topk_ids.to`, and `self._dispatch_core`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `buffer`, `topk_weights`, `topk_ids`, `expected_m`, and `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_NixlEPDispatcherImpl.dispatch_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_buffer`、`topk_ids.to` 以及 `self._dispatch_core`，说明该流程会编排底层辅助函数或计算内核。 像 `buffer`、`topk_weights`、`topk_ids`、`expected_m` 以及 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 236-266: `_NixlEPDispatcherImpl.dispatch_b` step for b
```python
    def dispatch_b(
        self,
        hidden_states,
        topk_ids,
        topk_weights,
        masked_m,
        expected_m,
        event,
        hook,
    ):
        hook() if self.return_recv_hook else event.current_stream_wait()

        get_global_expert_distribution_recorder().on_deepep_dispatch_low_latency(
            masked_m
        )

        if isinstance(hidden_states, tuple):
            hidden_states, hidden_states_scale = hidden_states
        else:
            hidden_states_scale = None

        nixl_output = NixlEPDispatchOutput(
            hidden_states,
            hidden_states_scale,
            topk_ids,
            topk_weights,
            masked_m,
            expected_m,
        )
        return nixl_output
```
**EN:** This block defines `_NixlEPDispatcherImpl.dispatch_b` and contains the main logic for this step. It mainly invokes `get_global_expert_distribution_recorder.on_deepep_dispatch_low_latency`, `isinstance`, `NixlEPDispatchOutput`, `hook`, and `event.current_stream_wait`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `nixl_output`, `hidden_states`, and `hidden_states_scale` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_NixlEPDispatcherImpl.dispatch_b`，并承载这一阶段的核心逻辑。 它主要调用 `get_global_expert_distribution_recorder.on_deepep_dispatch_low_latency`、`isinstance`、`NixlEPDispatchOutput`、`hook` 以及 `event.current_stream_wait`，说明该流程会编排底层辅助函数或计算内核。 像 `nixl_output`、`hidden_states` 以及 `hidden_states_scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 267-291: Internal helper `_NixlEPDispatcherImpl._dispatch_core`
```python
    def _dispatch_core(
        self,
        hidden_states: torch.Tensor,
        topk_idx: torch.Tensor,
    ):
        use_fp8 = not envs.SGLANG_NIXL_EP_BF16_DISPATCH.get()

        buffer = self._get_buffer()
        packed_recv_hidden, self.packed_recv_count, self.handle, event, hook = (
            buffer.dispatch(
                hidden_states,
                topk_idx,
                self.num_max_dispatch_tokens_per_rank,
                self.num_experts,
                use_fp8=use_fp8,
                async_finish=not self.return_recv_hook,
                return_recv_hook=self.return_recv_hook,
                round_scale=deep_gemm_wrapper.ENABLE_JIT_DEEPGEMM
                and deep_gemm_wrapper.DEEPGEMM_BLACKWELL,
                use_ue8m0=deep_gemm_wrapper.ENABLE_JIT_DEEPGEMM
                and deep_gemm_wrapper.DEEPGEMM_BLACKWELL,
            )
        )
        return packed_recv_hidden, self.packed_recv_count, event, hook
```
**EN:** This block defines `_NixlEPDispatcherImpl._dispatch_core` and contains the main logic for this step. It mainly invokes `self._get_buffer`, `buffer.dispatch`, and `envs.SGLANG_NIXL_EP_BF16_DISPATCH.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `use_fp8`, `buffer`, `packed_recv_hidden`, `self.packed_recv_count`, and `self.handle` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_NixlEPDispatcherImpl._dispatch_core`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_buffer`、`buffer.dispatch` 以及 `envs.SGLANG_NIXL_EP_BF16_DISPATCH.get`，说明该流程会编排底层辅助函数或计算内核。 像 `use_fp8`、`buffer`、`packed_recv_hidden`、`self.packed_recv_count` 以及 `self.handle` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 292-304: Function `_NixlEPDispatcherImpl.combine_a` and its core logic
```python
    def combine_a(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
    ):
        hidden_states, event, hook = self._combine_core(
            hidden_states,
            topk_ids,
            topk_weights,
        )
        return hidden_states, event, hook
```
**EN:** This block defines `_NixlEPDispatcherImpl.combine_a` and contains the main logic for this step. It mainly invokes `self._combine_core`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `event`, and `hook` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_NixlEPDispatcherImpl.combine_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._combine_core`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`event` 以及 `hook` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 305-308: Function `_NixlEPDispatcherImpl.combine_b` and its core logic
```python
    def combine_b(self, hidden_states, event, hook):
        hook() if self.return_recv_hook else event.current_stream_wait()
        return hidden_states
```
**EN:** This block defines `_NixlEPDispatcherImpl.combine_b` and contains the main logic for this step. It mainly invokes `hook` and `event.current_stream_wait`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_NixlEPDispatcherImpl.combine_b`，并承载这一阶段的核心逻辑。 它主要调用 `hook` 和 `event.current_stream_wait`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 309-331: Internal helper `_NixlEPDispatcherImpl._combine_core`
```python
    def _combine_core(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
    ):
        buffer = self._get_buffer()

        combined_hidden_states, event, hook = buffer.combine(
            x=hidden_states,
            topk_idx=topk_ids,
            topk_weights=topk_weights,
            handle=self.handle,
            async_finish=not self.return_recv_hook,
            return_recv_hook=self.return_recv_hook,
        )
        if self._mask_buffer is not None:
            buffer.query_mask_buffer(self._mask_buffer)
            self.active_ranks.copy_(1 - self._mask_buffer)

        self.packed_recv_count = self.handle = None
        return combined_hidden_states, event, hook
```
**EN:** This block defines `_NixlEPDispatcherImpl._combine_core` and contains the main logic for this step. It mainly invokes `self._get_buffer`, `buffer.combine`, `buffer.query_mask_buffer`, and `self.active_ranks.copy_`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `buffer`, `combined_hidden_states`, `event`, `hook`, and `self.packed_recv_count` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_NixlEPDispatcherImpl._combine_core`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_buffer`、`buffer.combine`、`buffer.query_mask_buffer` 以及 `self.active_ranks.copy_`，说明该流程会编排底层辅助函数或计算内核。 像 `buffer`、`combined_hidden_states`、`event`、`hook` 以及 `self.packed_recv_count` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 332-342: Internal helper `_NixlEPDispatcherImpl._get_buffer`
```python
    def _get_buffer(self):
        return NixlEPBuffer.get_nixl_buffer(
            self.group,
            self.hidden_size,
            self.deepep_mode,
            self.num_max_dispatch_tokens_per_rank,
            self.num_experts,
            self.num_local_experts,
        )
```
**EN:** This block defines `_NixlEPDispatcherImpl._get_buffer` and contains the main logic for this step. It mainly invokes `NixlEPBuffer.get_nixl_buffer`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_NixlEPDispatcherImpl._get_buffer`，并承载这一阶段的核心逻辑。 它主要调用 `NixlEPBuffer.get_nixl_buffer`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 343-349: Class `_Stage` declaration and shared state
```python
class _Stage(Enum):
    INITIAL = auto()
    AFTER_DISPATCH_A = auto()
    AFTER_DISPATCH_B = auto()
    AFTER_COMBINE_A = auto()
```
**EN:** This block introduces class `_Stage` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_Stage`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 350-350: Class `NixlEPDispatcher` declaration and shared state
```python
class NixlEPDispatcher(BaseDispatcher):
```
**EN:** This block introduces class `NixlEPDispatcher` and the state shared by its methods. It inherits from `BaseDispatcher`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `NixlEPDispatcher`，并定义其方法共享的状态。 它继承自 `BaseDispatcher`，说明了它在 SRT 层栈中的接入方式。

### Lines 351-386: `NixlEPDispatcher` initialization and state setup
```python
    def __init__(
        self,
        group: torch.distributed.ProcessGroup,
        router_topk: int,
        permute_fusion: bool = False,
        num_experts: int = None,
        num_local_experts: int = None,
        hidden_size: int = None,
        params_dtype: torch.dtype = None,
        deepep_mode: DeepEPMode = DeepEPMode.LOW_LATENCY,
        async_finish: bool = False,
        return_recv_hook: bool = False,
    ):
        self.deepep_mode = deepep_mode

        common_kwargs = dict(
            group=group,
            router_topk=router_topk,
            permute_fusion=permute_fusion,
            num_experts=num_experts,
            num_local_experts=num_local_experts,
            hidden_size=hidden_size,
            params_dtype=params_dtype,
            deepep_mode=deepep_mode,
        )

        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher = _NixlEPDispatcherImpl(
                return_recv_hook=return_recv_hook,
                **common_kwargs,
            )
        if self.deepep_mode.enable_normal():
            raise NotImplementedError("Normal mode is not supported for Nixl EP yet.")

        self._stage = _Stage.INITIAL
```
**EN:** This block defines `NixlEPDispatcher.__init__` and contains the main logic for this step. It mainly invokes `dict`, `self.deepep_mode.enable_low_latency`, `self.deepep_mode.enable_normal`, `_NixlEPDispatcherImpl`, and `NotImplementedError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.deepep_mode`, `common_kwargs`, `self._stage`, and `self._low_latency_dispatcher` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `NixlEPDispatcher.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `dict`、`self.deepep_mode.enable_low_latency`、`self.deepep_mode.enable_normal`、`_NixlEPDispatcherImpl` 以及 `NotImplementedError`，说明该流程会编排底层辅助函数或计算内核。 像 `self.deepep_mode`、`common_kwargs`、`self._stage` 以及 `self._low_latency_dispatcher` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 387-395: Function `NixlEPDispatcher.dispatch` and its core logic
```python
    def dispatch(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ) -> DispatchOutput:
        self.dispatch_a(hidden_states=hidden_states, topk_output=topk_output)
        ret = self.dispatch_b()
        return ret
```
**EN:** This block defines `NixlEPDispatcher.dispatch` and contains the main logic for this step. It mainly invokes `self.dispatch_a` and `self.dispatch_b`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `ret` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NixlEPDispatcher.dispatch`，并承载这一阶段的核心逻辑。 它主要调用 `self.dispatch_a` 和 `self.dispatch_b`，说明该流程会编排底层辅助函数或计算内核。 像 `ret` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 396-407: `NixlEPDispatcher.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        self._update_stage(_Stage.INITIAL, _Stage.AFTER_DISPATCH_A)
        inner_state = self._get_impl().dispatch_a(
            hidden_states=hidden_states,
            topk_output=topk_output,
        )
        self._dispatch_intermediate_state = inner_state
```
**EN:** This block defines `NixlEPDispatcher.dispatch_a` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.dispatch_a`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` and `self._dispatch_intermediate_state` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `NixlEPDispatcher.dispatch_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.dispatch_a` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 和 `self._dispatch_intermediate_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 408-413: `NixlEPDispatcher.dispatch_b` step for b
```python
    def dispatch_b(self):
        self._update_stage(_Stage.AFTER_DISPATCH_A, _Stage.AFTER_DISPATCH_B)
        inner_state = self._dispatch_intermediate_state
        del self._dispatch_intermediate_state
        return self._get_impl().dispatch_b(*inner_state)
```
**EN:** This block defines `NixlEPDispatcher.dispatch_b` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.dispatch_b`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NixlEPDispatcher.dispatch_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.dispatch_b` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 414-421: Function `NixlEPDispatcher.combine` and its core logic
```python
    def combine(
        self,
        combine_input: CombineInput,
    ) -> torch.Tensor:
        self.combine_a(combine_input)
        ret = self.combine_b()
        return ret
```
**EN:** This block defines `NixlEPDispatcher.combine` and contains the main logic for this step. It mainly invokes `self.combine_a` and `self.combine_b`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `ret` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NixlEPDispatcher.combine`，并承载这一阶段的核心逻辑。 它主要调用 `self.combine_a` 和 `self.combine_b`，说明该流程会编排底层辅助函数或计算内核。 像 `ret` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 422-434: Function `NixlEPDispatcher.combine_a` and its core logic
```python
    def combine_a(
        self,
        combine_input: CombineInput,
    ):
        hidden_states, topk_ids, topk_weights = combine_input
        self._update_stage(_Stage.AFTER_DISPATCH_B, _Stage.AFTER_COMBINE_A)
        inner_state = self._get_impl().combine_a(
            hidden_states=hidden_states,
            topk_ids=topk_ids,
            topk_weights=topk_weights,
        )
        self._combine_intermediate_state = inner_state
```
**EN:** This block defines `NixlEPDispatcher.combine_a` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.combine_a`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_ids`, `topk_weights`, `inner_state`, and `self._combine_intermediate_state` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `NixlEPDispatcher.combine_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.combine_a` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_ids`、`topk_weights`、`inner_state` 以及 `self._combine_intermediate_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 435-440: Function `NixlEPDispatcher.combine_b` and its core logic
```python
    def combine_b(self):
        self._update_stage(_Stage.AFTER_COMBINE_A, _Stage.INITIAL)
        inner_state = self._combine_intermediate_state
        del self._combine_intermediate_state
        return self._get_impl().combine_b(*inner_state)
```
**EN:** This block defines `NixlEPDispatcher.combine_b` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.combine_b`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NixlEPDispatcher.combine_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.combine_b` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 441-450: Internal helper `NixlEPDispatcher._get_impl`
```python
    def _get_impl(self) -> _NixlEPDispatcherImplBase:
        is_extend_in_batch = get_is_extend_in_batch()
        resolved_deepep_mode = self.deepep_mode.resolve(is_extend_in_batch)
        if resolved_deepep_mode == DeepEPMode.NORMAL:
            raise NotImplementedError("Normal mode is not supported for Nixl EP yet.")
        elif resolved_deepep_mode == DeepEPMode.LOW_LATENCY:
            return self._low_latency_dispatcher
        else:
            raise ValueError(f"Invalid deepep_mode: {self.deepep_mode}")
```
**EN:** This block defines `NixlEPDispatcher._get_impl` and contains the main logic for this step. It mainly invokes `get_is_extend_in_batch`, `self.deepep_mode.resolve`, `NotImplementedError`, and `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `is_extend_in_batch` and `resolved_deepep_mode` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NixlEPDispatcher._get_impl`，并承载这一阶段的核心逻辑。 它主要调用 `get_is_extend_in_batch`、`self.deepep_mode.resolve`、`NotImplementedError` 以及 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `is_extend_in_batch` 和 `resolved_deepep_mode` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 451-455: `NixlEPDispatcher.set_quant_config` setter for quantization config
```python
    def set_quant_config(self, quant_config: dict):
        super().set_quant_config(quant_config)
        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher.set_quant_config(quant_config)
```
**EN:** This block defines `NixlEPDispatcher.set_quant_config` and contains the main logic for this step. It mainly invokes `super.set_quant_config`, `self.deepep_mode.enable_low_latency`, and `self._low_latency_dispatcher.set_quant_config`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `NixlEPDispatcher.set_quant_config`，并承载这一阶段的核心逻辑。 它主要调用 `super.set_quant_config`、`self.deepep_mode.enable_low_latency` 以及 `self._low_latency_dispatcher.set_quant_config`，说明该流程会编排底层辅助函数或计算内核。

### Lines 456-462: `NixlEPDispatcher.set_overlap_args` setter for overlap args
```python
    def set_overlap_args(self, combine_overlap_args, meta_overlap_args):
        super().set_overlap_args(combine_overlap_args, meta_overlap_args)
        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher.set_overlap_args(
                combine_overlap_args, meta_overlap_args
            )
```
**EN:** This block defines `NixlEPDispatcher.set_overlap_args` and contains the main logic for this step. It mainly invokes `super.set_overlap_args`, `self.deepep_mode.enable_low_latency`, and `self._low_latency_dispatcher.set_overlap_args`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `NixlEPDispatcher.set_overlap_args`，并承载这一阶段的核心逻辑。 它主要调用 `super.set_overlap_args`、`self.deepep_mode.enable_low_latency` 以及 `self._low_latency_dispatcher.set_overlap_args`，说明该流程会编排底层辅助函数或计算内核。

### Lines 463-465: Internal helper `NixlEPDispatcher._update_stage`
```python
    def _update_stage(self, old_stage, new_stage):
        assert self._stage == old_stage
        self._stage = new_stage
```
**EN:** This block defines `NixlEPDispatcher._update_stage` and contains the main logic for this step. Intermediate names such as `self._stage` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `NixlEPDispatcher._update_stage`，并承载这一阶段的核心逻辑。 像 `self._stage` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `NixlEPBuffer`, `_NixlEPDispatcherImplBase`, `_NixlEPDispatcherImpl`, `_Stage`, and `NixlEPDispatcher`. / **主要符号**：核心入口包括 `NixlEPBuffer`、`_NixlEPDispatcherImplBase`、`_NixlEPDispatcherImpl`、`_Stage` 以及 `NixlEPDispatcher`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `enum.Enum`, `enum.auto`, and `typing.Optional` / **标准库**：`__future__.annotations`、`logging`、`enum.Enum`、`enum.auto` 以及 `typing.Optional`
- **Third-party**: `torch`, `torch.distributed`, and `nixl_ep.Buffer` / **第三方依赖**：`torch`、`torch.distributed` 以及 `nixl_ep.Buffer`
- **Internal SGLang modules**: `sglang.srt.distributed.utils.get_global_tcp_store`, `sglang.srt.elastic_ep.elastic_ep.ElasticEPStateManager`, `sglang.srt.environ.envs`, `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`, `sglang.srt.layers.deep_gemm_wrapper`, `sglang.srt.layers.dp_attention.get_is_extend_in_batch`, `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`, `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLCombineInput`, `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLDispatchOutput`, and `sglang.srt.layers.moe.topk.TopKOutput` / **SGLang 内部模块**：`sglang.srt.distributed.utils.get_global_tcp_store`、`sglang.srt.elastic_ep.elastic_ep.ElasticEPStateManager`、`sglang.srt.environ.envs`、`sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`、`sglang.srt.layers.deep_gemm_wrapper`、`sglang.srt.layers.dp_attention.get_is_extend_in_batch`、`sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`、`sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLCombineInput`、`sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLDispatchOutput` 以及 `sglang.srt.layers.moe.topk.TopKOutput`
