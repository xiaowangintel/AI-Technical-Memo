# mooncake.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/token_dispatcher/mooncake.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `MooncakeDispatchOutput`, `MooncakeCombineInput`, `EPBuffer`, and `_MooncakeEPDispatcherImpl` and connects them to backend-specific paths such as `CUDA` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合混合专家路由与专家执行。它提供了 `MooncakeDispatchOutput`、`MooncakeCombineInput`、`EPBuffer` 以及 `_MooncakeEPDispatcherImpl` 等符号，并把这些符号连接到 `CUDA` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Imports, constants, and runtime setup
```python
from __future__ import annotations

import logging
from dataclasses import dataclass
from enum import Enum, auto
from typing import NamedTuple, Optional

import torch
import torch.distributed as dist

from sglang.srt.elastic_ep.elastic_ep import ElasticEPStateManager
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.layers.dp_attention import get_is_extend_in_batch
from sglang.srt.layers.moe.token_dispatcher.base import (
    BaseDispatcher,
    CombineInput,
    CombineInputFormat,
    DispatchOutput,
    DispatchOutputFormat,
)
from sglang.srt.layers.moe.topk import TopKOutput
from sglang.srt.layers.moe.utils import DeepEPMode
from sglang.srt.utils import get_int_env_var

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `dataclasses.dataclass`, `enum.Enum`, `enum.auto`, and `typing.NamedTuple`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`dataclasses.dataclass`、`enum.Enum`、`enum.auto` 以及 `typing.NamedTuple`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 28-37: Class `MooncakeDispatchOutput` declaration and shared state
```python
class MooncakeDispatchOutput(NamedTuple):
    """Mooncake EP dispatch output."""

    hidden_states: torch.Tensor
    hidden_states_scale: Optional[torch.Tensor]
    topk_ids: torch.Tensor
    topk_weights: torch.Tensor
    masked_m: torch.Tensor
    expected_m: int
```
**EN:** This block introduces class `MooncakeDispatchOutput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Mooncake EP dispatch output.
**CN:** 该代码块引入类 `MooncakeDispatchOutput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 38-42: Function `MooncakeDispatchOutput.format` and its core logic
```python
    @property
    def format(self) -> DispatchOutputFormat:
        return DispatchOutputFormat.DEEPEP_LL
```
**EN:** This block defines `MooncakeDispatchOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MooncakeDispatchOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 43-45: Module-level helper logic
```python
assert isinstance(MooncakeDispatchOutput, DispatchOutput)
```
**EN:** This section prepares the module namespace.
**CN:** 该部分负责准备模块命名空间。

### Lines 46-50: Class `MooncakeCombineInput` declaration and shared state
```python
class MooncakeCombineInput(NamedTuple):
    """Mooncake EP combine input."""

    pass
```
**EN:** This block introduces class `MooncakeCombineInput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Mooncake EP combine input.
**CN:** 该代码块引入类 `MooncakeCombineInput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 51-55: Function `MooncakeCombineInput.format` and its core logic
```python
    @property
    def format(self) -> CombineInputFormat:
        return CombineInputFormat.DEEPEP_LL
```
**EN:** This block defines `MooncakeCombineInput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MooncakeCombineInput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 56-58: Module-level helper logic
```python
assert isinstance(MooncakeCombineInput, CombineInput)
```
**EN:** This section prepares the module namespace.
**CN:** 该部分负责准备模块命名空间。

### Lines 59-64: Class `EPBuffer` declaration and shared state
```python
class EPBuffer:
    _buffer = None
    _hidden_size: Optional[int] = None
    _num_max_dispatch_tokens_per_rank: Optional[int] = None
    _num_experts: Optional[int] = None
```
**EN:** This block introduces class `EPBuffer` and the state shared by its methods.
**CN:** 该代码块引入类 `EPBuffer`，并定义其方法共享的状态。

### Lines 65-103: `EPBuffer.get_ep_buffer` getter for expert parallel buffer
```python
    @classmethod
    def get_ep_buffer(
        cls,
        group: dist.ProcessGroup,
        hidden_size: int,
        param_bytes: int,
        deepep_mode: DeepEPMode,
        num_max_dispatch_tokens_per_rank: int = -1,
        num_experts: int = -1,
    ):
        if cls._buffer is not None:
            return cls._buffer

        # Lazy import Buffer to avoid creating CUDA context at module import time
        from mooncake.mooncake_ep_buffer import Buffer

        cls._hidden_size = hidden_size
        cls._num_max_dispatch_tokens_per_rank = num_max_dispatch_tokens_per_rank
        cls._num_experts = num_experts

        num_ep_buffer_bytes = 0
        if deepep_mode.enable_normal():
            raise NotImplementedError(
                "Normal mode is not supported for Mooncake EP yet."
            )
        if deepep_mode.enable_low_latency():
            assert num_max_dispatch_tokens_per_rank != -1
            assert num_experts != -1 and num_experts % group.size() == 0
            num_ep_buffer_bytes = Buffer.get_ep_buffer_size_hint(
                num_max_dispatch_tokens_per_rank,
                hidden_size,
                group.size(),
                num_experts,
            )

        cls._buffer = Buffer(group, num_ep_buffer_bytes)
        return cls._buffer
```
**EN:** This block defines `EPBuffer.get_ep_buffer` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `deepep_mode.enable_normal`, `deepep_mode.enable_low_latency`, `Buffer`, `NotImplementedError`, and `Buffer.get_ep_buffer_size_hint`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_ep_buffer_bytes` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `EPBuffer.get_ep_buffer`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `deepep_mode.enable_normal`、`deepep_mode.enable_low_latency`、`Buffer`、`NotImplementedError` 以及 `Buffer.get_ep_buffer_size_hint`，说明该流程会编排底层辅助函数或计算内核。 像 `num_ep_buffer_bytes` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 104-104: Class `_MooncakeEPDispatcherImpl` declaration and shared state
```python
class _MooncakeEPDispatcherImpl:
```
**EN:** This block introduces class `_MooncakeEPDispatcherImpl` and the state shared by its methods.
**CN:** 该代码块引入类 `_MooncakeEPDispatcherImpl`，并定义其方法共享的状态。

### Lines 105-147: `_MooncakeEPDispatcherImpl` initialization and state setup
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
        return_recv_hook: bool,
        deepep_mode: DeepEPMode,
    ):
        try:
            from mooncake.mooncake_ep_buffer import Buffer  # noqa: F401
        except ImportError:
            raise ImportError(
                "Mooncake EP is not installed. Please install Mooncake package at "
                "https://github.com/kvcache-ai/Mooncake/blob/main/doc/en/build.md "
                "with EP support to run SGLang with Mooncake EP."
            )
        self.group = group
        self.router_topk = router_topk
        self.permute_fusion = permute_fusion
        self.num_experts = num_experts
        self.num_local_experts = num_local_experts
        self.hidden_size = hidden_size
        self.params_dtype = params_dtype
        self.return_recv_hook = return_recv_hook
        self.deepep_mode = deepep_mode

        self.params_bytes = 2
        self.num_max_dispatch_tokens_per_rank = get_int_env_var(
            "SGLANG_MOONCAKE_EP_NUM_MAX_DISPATCH_TOKENS_PER_RANK", 128
        )
        # Mooncake EP dispatch uses FINISHED_SUM_TAG=1024
        # and the logic requires num-tokens-sent-from-one-rank-to-another-rank less than it
        assert self.num_max_dispatch_tokens_per_rank <= 1024

        self.first_execution = True
        self.timeout_us = 10000000

        self.handle = None
```
**EN:** This block defines `_MooncakeEPDispatcherImpl.__init__` and contains the main logic for this step. It mainly invokes `get_int_env_var` and `ImportError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.group`, `self.router_topk`, `self.permute_fusion`, `self.num_experts`, and `self.num_local_experts` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_MooncakeEPDispatcherImpl.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `get_int_env_var` 和 `ImportError`，说明该流程会编排底层辅助函数或计算内核。 像 `self.group`、`self.router_topk`、`self.permute_fusion`、`self.num_experts` 以及 `self.num_local_experts` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 148-174: `_MooncakeEPDispatcherImpl.dispatch_a` step for a
```python
    def dispatch_a(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ):
        topk_ids, topk_weights = topk_output.topk_ids, topk_output.topk_weights
        buffer = self._get_buffer()
        topk_ids = topk_ids.to(torch.int64)
        expected_m = (
            hidden_states.shape[0] * buffer.group_size * topk_ids.shape[1]
            + self.num_experts
        ) // self.num_experts
        hidden_states, masked_m, event, hook = self._dispatch_core(
            hidden_states,
            topk_ids,
            use_fp8=True,
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
**EN:** This block defines `_MooncakeEPDispatcherImpl.dispatch_a` and contains the main logic for this step. It mainly invokes `self._get_buffer`, `topk_ids.to`, and `self._dispatch_core`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `topk_ids`, `topk_weights`, `buffer`, `expected_m`, and `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MooncakeEPDispatcherImpl.dispatch_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_buffer`、`topk_ids.to` 以及 `self._dispatch_core`，说明该流程会编排底层辅助函数或计算内核。 像 `topk_ids`、`topk_weights`、`buffer`、`expected_m` 以及 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 175-204: `_MooncakeEPDispatcherImpl.dispatch_b` step for b
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

        return MooncakeDispatchOutput(
            hidden_states,
            hidden_states_scale,
            topk_ids,
            topk_weights,
            masked_m,
            expected_m,
        )
```
**EN:** This block defines `_MooncakeEPDispatcherImpl.dispatch_b` and contains the main logic for this step. It mainly invokes `get_global_expert_distribution_recorder.on_deepep_dispatch_low_latency`, `isinstance`, `MooncakeDispatchOutput`, `hook`, and `event.current_stream_wait`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states` and `hidden_states_scale` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MooncakeEPDispatcherImpl.dispatch_b`，并承载这一阶段的核心逻辑。 它主要调用 `get_global_expert_distribution_recorder.on_deepep_dispatch_low_latency`、`isinstance`、`MooncakeDispatchOutput`、`hook` 以及 `event.current_stream_wait`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states` 和 `hidden_states_scale` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 205-227: Internal helper `_MooncakeEPDispatcherImpl._dispatch_core`
```python
    def _dispatch_core(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        use_fp8: bool = False,
    ):
        buffer = self._get_buffer()
        active_ranks = ElasticEPStateManager.instance().active_ranks
        packed_recv_hidden, packed_recv_count, self.handle, event, hook = (
            buffer.dispatch(
                hidden_states,
                topk_ids,
                active_ranks,
                self.num_max_dispatch_tokens_per_rank,
                self.num_experts,
                -1 if self.first_execution else self.timeout_us,
                use_fp8=use_fp8,
                async_finish=not self.return_recv_hook,
                return_recv_hook=self.return_recv_hook,
            )
        )
        return packed_recv_hidden, packed_recv_count, event, hook
```
**EN:** This block defines `_MooncakeEPDispatcherImpl._dispatch_core` and contains the main logic for this step. It mainly invokes `self._get_buffer`, `buffer.dispatch`, and `ElasticEPStateManager.instance`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `buffer`, `active_ranks`, `packed_recv_hidden`, `packed_recv_count`, and `self.handle` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MooncakeEPDispatcherImpl._dispatch_core`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_buffer`、`buffer.dispatch` 以及 `ElasticEPStateManager.instance`，说明该流程会编排底层辅助函数或计算内核。 像 `buffer`、`active_ranks`、`packed_recv_hidden`、`packed_recv_count` 以及 `self.handle` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 228-240: Function `_MooncakeEPDispatcherImpl.combine_a` and its core logic
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
**EN:** This block defines `_MooncakeEPDispatcherImpl.combine_a` and contains the main logic for this step. It mainly invokes `self._combine_core`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `event`, and `hook` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MooncakeEPDispatcherImpl.combine_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._combine_core`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`event` 以及 `hook` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 241-244: Function `_MooncakeEPDispatcherImpl.combine_b` and its core logic
```python
    def combine_b(self, hidden_states, event, hook):
        hook() if self.return_recv_hook else event.current_stream_wait()
        return hidden_states
```
**EN:** This block defines `_MooncakeEPDispatcherImpl.combine_b` and contains the main logic for this step. It mainly invokes `hook` and `event.current_stream_wait`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MooncakeEPDispatcherImpl.combine_b`，并承载这一阶段的核心逻辑。 它主要调用 `hook` 和 `event.current_stream_wait`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 245-266: Internal helper `_MooncakeEPDispatcherImpl._combine_core`
```python
    def _combine_core(
        self,
        hidden_states: torch.Tensor,
        topk_ids: torch.Tensor,
        topk_weights: torch.Tensor,
    ):
        buffer = self._get_buffer()
        active_ranks = ElasticEPStateManager.instance().active_ranks
        combined_hidden_states, event, hook = buffer.combine(
            hidden_states,
            topk_ids,
            topk_weights,
            active_ranks,
            -1 if self.first_execution else self.timeout_us,
            self.handle,
            async_finish=not self.return_recv_hook,
            return_recv_hook=self.return_recv_hook,
        )
        self.first_execution = False
        self.handle = None
        return combined_hidden_states, event, hook
```
**EN:** This block defines `_MooncakeEPDispatcherImpl._combine_core` and contains the main logic for this step. It mainly invokes `self._get_buffer`, `buffer.combine`, and `ElasticEPStateManager.instance`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `buffer`, `active_ranks`, `combined_hidden_states`, `event`, and `hook` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MooncakeEPDispatcherImpl._combine_core`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_buffer`、`buffer.combine` 以及 `ElasticEPStateManager.instance`，说明该流程会编排底层辅助函数或计算内核。 像 `buffer`、`active_ranks`、`combined_hidden_states`、`event` 以及 `hook` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 267-277: Internal helper `_MooncakeEPDispatcherImpl._get_buffer`
```python
    def _get_buffer(self):
        return EPBuffer.get_ep_buffer(
            self.group,
            self.hidden_size,
            self.params_bytes,
            self.deepep_mode,
            self.num_max_dispatch_tokens_per_rank,
            self.num_experts,
        )
```
**EN:** This block defines `_MooncakeEPDispatcherImpl._get_buffer` and contains the main logic for this step. It mainly invokes `EPBuffer.get_ep_buffer`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_MooncakeEPDispatcherImpl._get_buffer`，并承载这一阶段的核心逻辑。 它主要调用 `EPBuffer.get_ep_buffer`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 278-285: Class `_Stage` declaration and shared state
```python
@dataclass
class _Stage(Enum):
    INITIAL = auto()
    AFTER_DISPATCH_A = auto()
    AFTER_DISPATCH_B = auto()
    AFTER_COMBINE_A = auto()
```
**EN:** This block introduces class `_Stage` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `_Stage`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。

### Lines 286-286: Class `MooncakeEPDispatcher` declaration and shared state
```python
class MooncakeEPDispatcher(BaseDispatcher):
```
**EN:** This block introduces class `MooncakeEPDispatcher` and the state shared by its methods. It inherits from `BaseDispatcher`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `MooncakeEPDispatcher`，并定义其方法共享的状态。 它继承自 `BaseDispatcher`，说明了它在 SRT 层栈中的接入方式。

### Lines 287-320: `MooncakeEPDispatcher` initialization and state setup
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
        deepep_mode: DeepEPMode = DeepEPMode.AUTO,
        async_finish: bool = False,
        return_recv_hook: bool = False,
    ):
        super().__init__()

        self.deepep_mode = deepep_mode

        if self.deepep_mode.enable_low_latency():
            self._low_latency_dispatcher = _MooncakeEPDispatcherImpl(
                group=group,
                router_topk=router_topk,
                permute_fusion=permute_fusion,
                num_experts=num_experts,
                num_local_experts=num_local_experts,
                hidden_size=hidden_size,
                params_dtype=params_dtype,
                return_recv_hook=return_recv_hook,
                deepep_mode=deepep_mode,
            )
        if self.deepep_mode.enable_normal():
            raise NotImplementedError

        self._stage = _Stage.INITIAL
```
**EN:** This block defines `MooncakeEPDispatcher.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `self.deepep_mode.enable_low_latency`, `self.deepep_mode.enable_normal`, and `_MooncakeEPDispatcherImpl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.deepep_mode`, `self._stage`, and `self._low_latency_dispatcher` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MooncakeEPDispatcher.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`self.deepep_mode.enable_low_latency`、`self.deepep_mode.enable_normal` 以及 `_MooncakeEPDispatcherImpl`，说明该流程会编排底层辅助函数或计算内核。 像 `self.deepep_mode`、`self._stage` 以及 `self._low_latency_dispatcher` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 321-329: Function `MooncakeEPDispatcher.dispatch` and its core logic
```python
    def dispatch(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
    ) -> DispatchOutput:
        self.dispatch_a(hidden_states, topk_output)
        ret = self.dispatch_b()
        return ret
```
**EN:** This block defines `MooncakeEPDispatcher.dispatch` and contains the main logic for this step. It mainly invokes `self.dispatch_a` and `self.dispatch_b`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `ret` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MooncakeEPDispatcher.dispatch`，并承载这一阶段的核心逻辑。 它主要调用 `self.dispatch_a` 和 `self.dispatch_b`，说明该流程会编排底层辅助函数或计算内核。 像 `ret` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 330-341: `MooncakeEPDispatcher.dispatch_a` step for a
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
**EN:** This block defines `MooncakeEPDispatcher.dispatch_a` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.dispatch_a`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` and `self._dispatch_intermediate_state` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MooncakeEPDispatcher.dispatch_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.dispatch_a` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 和 `self._dispatch_intermediate_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 342-347: `MooncakeEPDispatcher.dispatch_b` step for b
```python
    def dispatch_b(self):
        self._update_stage(_Stage.AFTER_DISPATCH_A, _Stage.AFTER_DISPATCH_B)
        inner_state = self._dispatch_intermediate_state
        del self._dispatch_intermediate_state
        return self._get_impl().dispatch_b(*inner_state)
```
**EN:** This block defines `MooncakeEPDispatcher.dispatch_b` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.dispatch_b`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MooncakeEPDispatcher.dispatch_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.dispatch_b` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 348-355: Function `MooncakeEPDispatcher.combine` and its core logic
```python
    def combine(
        self,
        combine_input: CombineInput,
    ) -> torch.Tensor:
        self.combine_a(combine_input)
        ret = self.combine_b()
        return ret
```
**EN:** This block defines `MooncakeEPDispatcher.combine` and contains the main logic for this step. It mainly invokes `self.combine_a` and `self.combine_b`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `ret` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MooncakeEPDispatcher.combine`，并承载这一阶段的核心逻辑。 它主要调用 `self.combine_a` 和 `self.combine_b`，说明该流程会编排底层辅助函数或计算内核。 像 `ret` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 356-368: Function `MooncakeEPDispatcher.combine_a` and its core logic
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
**EN:** This block defines `MooncakeEPDispatcher.combine_a` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.combine_a`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_ids`, `topk_weights`, `inner_state`, and `self._combine_intermediate_state` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MooncakeEPDispatcher.combine_a`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.combine_a` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_ids`、`topk_weights`、`inner_state` 以及 `self._combine_intermediate_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 369-374: Function `MooncakeEPDispatcher.combine_b` and its core logic
```python
    def combine_b(self):
        self._update_stage(_Stage.AFTER_COMBINE_A, _Stage.INITIAL)
        inner_state = self._combine_intermediate_state
        del self._combine_intermediate_state
        return self._get_impl().combine_b(*inner_state)
```
**EN:** This block defines `MooncakeEPDispatcher.combine_b` and contains the main logic for this step. It mainly invokes `self._update_stage`, `self._get_impl.combine_b`, and `self._get_impl`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `inner_state` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MooncakeEPDispatcher.combine_b`，并承载这一阶段的核心逻辑。 它主要调用 `self._update_stage`、`self._get_impl.combine_b` 以及 `self._get_impl`，说明该流程会编排底层辅助函数或计算内核。 像 `inner_state` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 375-384: Internal helper `MooncakeEPDispatcher._get_impl`
```python
    def _get_impl(self) -> _MooncakeEPDispatcherImpl:
        is_extend_in_batch = get_is_extend_in_batch()
        resolved_deepep_mode = self.deepep_mode.resolve(is_extend_in_batch)
        if resolved_deepep_mode == DeepEPMode.NORMAL:
            raise NotImplementedError
        elif resolved_deepep_mode == DeepEPMode.LOW_LATENCY:
            return self._low_latency_dispatcher
        else:
            raise ValueError(f"Invalid deepep_mode: {self.deepep_mode}")
```
**EN:** This block defines `MooncakeEPDispatcher._get_impl` and contains the main logic for this step. It mainly invokes `get_is_extend_in_batch`, `self.deepep_mode.resolve`, and `ValueError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `is_extend_in_batch` and `resolved_deepep_mode` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MooncakeEPDispatcher._get_impl`，并承载这一阶段的核心逻辑。 它主要调用 `get_is_extend_in_batch`、`self.deepep_mode.resolve` 以及 `ValueError`，说明该流程会编排底层辅助函数或计算内核。 像 `is_extend_in_batch` 和 `resolved_deepep_mode` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 385-387: Internal helper `MooncakeEPDispatcher._update_stage`
```python
    def _update_stage(self, old_stage, new_stage):
        assert self._stage == old_stage
        self._stage = new_stage
```
**EN:** This block defines `MooncakeEPDispatcher._update_stage` and contains the main logic for this step. Intermediate names such as `self._stage` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `MooncakeEPDispatcher._update_stage`，并承载这一阶段的核心逻辑。 像 `self._stage` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `MooncakeDispatchOutput`, `MooncakeCombineInput`, `EPBuffer`, `_MooncakeEPDispatcherImpl`, and `_Stage`. / **主要符号**：核心入口包括 `MooncakeDispatchOutput`、`MooncakeCombineInput`、`EPBuffer`、`_MooncakeEPDispatcherImpl` 以及 `_Stage`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, `dataclasses.dataclass`, `enum.Enum`, `enum.auto`, `typing.NamedTuple`, and `typing.Optional` / **标准库**：`__future__.annotations`、`logging`、`dataclasses.dataclass`、`enum.Enum`、`enum.auto`、`typing.NamedTuple` 以及 `typing.Optional`
- **Third-party**: `torch`, `torch.distributed`, and `mooncake.mooncake_ep_buffer.Buffer` / **第三方依赖**：`torch`、`torch.distributed` 以及 `mooncake.mooncake_ep_buffer.Buffer`
- **Internal SGLang modules**: `sglang.srt.elastic_ep.elastic_ep.ElasticEPStateManager`, `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`, `sglang.srt.layers.dp_attention.get_is_extend_in_batch`, `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputFormat`, `sglang.srt.layers.moe.topk.TopKOutput`, `sglang.srt.layers.moe.utils.DeepEPMode`, and `sglang.srt.utils.get_int_env_var` / **SGLang 内部模块**：`sglang.srt.elastic_ep.elastic_ep.ElasticEPStateManager`、`sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`、`sglang.srt.layers.dp_attention.get_is_extend_in_batch`、`sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputFormat`、`sglang.srt.layers.moe.topk.TopKOutput`、`sglang.srt.layers.moe.utils.DeepEPMode` 以及 `sglang.srt.utils.get_int_env_var`
