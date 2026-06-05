# fuseep.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/token_dispatcher/fuseep.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `FuseEPDispatchOutput`, `FuseEPCombineInput`, and `NpuFuseEPDispatcher` and connects them to backend-specific paths such as `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合混合专家路由与专家执行。它提供了 `FuseEPDispatchOutput`、`FuseEPCombineInput` 以及 `NpuFuseEPDispatcher` 等符号，并把这些符号连接到 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Imports, constants, and runtime setup
```python
from __future__ import annotations

import logging
from typing import NamedTuple

import torch

from sglang.srt.environ import envs
from sglang.srt.layers.moe.token_dispatcher.base import (
    BaseDispatcher,
    CombineInput,
    CombineInputFormat,
    DispatchOutput,
    DispatchOutputFormat,
)
from sglang.srt.layers.moe.token_dispatcher.deepep import DeepEPBuffer
from sglang.srt.layers.moe.topk import TopKOutput
from sglang.srt.layers.moe.utils import DeepEPMode

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `logging`, `typing.NamedTuple`, `torch`, `sglang.srt.environ.envs`, and `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`logging`、`typing.NamedTuple`、`torch`、`sglang.srt.environ.envs` 以及 `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 23-27: Class `FuseEPDispatchOutput` declaration and shared state
```python
class FuseEPDispatchOutput(NamedTuple):
    """DeepEP low latency dispatch output."""

    hidden_state: torch.Tensor
```
**EN:** This block introduces class `FuseEPDispatchOutput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: DeepEP low latency dispatch output.
**CN:** 该代码块引入类 `FuseEPDispatchOutput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 28-32: Function `FuseEPDispatchOutput.format` and its core logic
```python
    @property
    def format(self) -> DispatchOutputFormat:
        return DispatchOutputFormat.DEEPEP_LL
```
**EN:** This block defines `FuseEPDispatchOutput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FuseEPDispatchOutput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 33-37: Class `FuseEPCombineInput` declaration and shared state
```python
class FuseEPCombineInput(NamedTuple):
    """DeepEP low latency combine input."""

    hidden_state: torch.Tensor
```
**EN:** This block introduces class `FuseEPCombineInput` and the state shared by its methods. It inherits from `NamedTuple`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: DeepEP low latency combine input.
**CN:** 该代码块引入类 `FuseEPCombineInput`，并定义其方法共享的状态。 它继承自 `NamedTuple`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 38-42: Function `FuseEPCombineInput.format` and its core logic
```python
    @property
    def format(self) -> CombineInputFormat:
        return CombineInputFormat.DEEPEP_LL
```
**EN:** This block defines `FuseEPCombineInput.format` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FuseEPCombineInput.format`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 43-43: Class `NpuFuseEPDispatcher` declaration and shared state
```python
class NpuFuseEPDispatcher(BaseDispatcher):
```
**EN:** This block introduces class `NpuFuseEPDispatcher` and the state shared by its methods. It inherits from `BaseDispatcher`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `NpuFuseEPDispatcher`，并定义其方法共享的状态。 它继承自 `BaseDispatcher`，说明了它在 SRT 层栈中的接入方式。

### Lines 44-68: `NpuFuseEPDispatcher` initialization and state setup
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
    ):
        self.group = group
        self.router_topk = router_topk
        self.permute_fusion = permute_fusion
        self.num_experts = num_experts
        self.num_local_experts = num_local_experts
        self.hidden_size = hidden_size
        self.params_dtype = params_dtype
        self.deepep_mode = deepep_mode

        self.params_bytes = 2
        self.num_max_dispatch_tokens_per_rank = (
            envs.SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK.get()
        )
```
**EN:** This block defines `NpuFuseEPDispatcher.__init__` and contains the main logic for this step. It mainly invokes `envs.SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.group`, `self.router_topk`, `self.permute_fusion`, `self.num_experts`, and `self.num_local_experts` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `NpuFuseEPDispatcher.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `envs.SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK.get`，说明该流程会编排底层辅助函数或计算内核。 像 `self.group`、`self.router_topk`、`self.permute_fusion`、`self.num_experts` 以及 `self.num_local_experts` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 69-85: Function `NpuFuseEPDispatcher.dispatch` and its core logic
```python
    def dispatch(
        self, hidden_states: torch.Tensor, topk_output: TopKOutput, **kwargs
    ) -> DispatchOutput:
        hidden_states, _ = self._get_buffer().fused_deep_moe(
            hidden_states,
            topk_idx=topk_output.topk_ids,
            topk_weights=topk_output.topk_weights,
            gmm1_permuted_weight=kwargs["gmm1_permuted_weight"],
            gmm1_permuted_weight_scale=kwargs["gmm1_permuted_weight_scale"],
            gmm2_weight=kwargs["gmm2_weight"],
            gmm2_weight_scale=kwargs["gmm2_weight_scale"],
            num_max_dispatch_tokens_per_rank=self.num_max_dispatch_tokens_per_rank,
            num_experts=self.num_experts,
            fuse_mode=envs.SGLANG_NPU_FUSED_MOE_MODE.get(),
        )
        return FuseEPDispatchOutput(hidden_states)
```
**EN:** This block defines `NpuFuseEPDispatcher.dispatch` and contains the main logic for this step. It mainly invokes `self._get_buffer.fused_deep_moe`, `FuseEPDispatchOutput`, `self._get_buffer`, and `envs.SGLANG_NPU_FUSED_MOE_MODE.get`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states` and `_` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NpuFuseEPDispatcher.dispatch`，并承载这一阶段的核心逻辑。 它主要调用 `self._get_buffer.fused_deep_moe`、`FuseEPDispatchOutput`、`self._get_buffer` 以及 `envs.SGLANG_NPU_FUSED_MOE_MODE.get`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states` 和 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 86-88: Function `NpuFuseEPDispatcher.combine` and its core logic
```python
    def combine(self, combine_input: CombineInput, **kwargs) -> torch.Tensor:
        pass
```
**EN:** This block defines `NpuFuseEPDispatcher.combine` and contains the main logic for this step.
**CN:** 该代码块定义了 `NpuFuseEPDispatcher.combine`，并承载这一阶段的核心逻辑。

### Lines 89-98: Internal helper `NpuFuseEPDispatcher._get_buffer`
```python
    def _get_buffer(self):
        DeepEPBuffer.set_dispatch_mode_as_low_latency()
        return DeepEPBuffer.get_deepep_buffer(
            self.group,
            self.hidden_size,
            self.params_bytes,
            self.deepep_mode,
            self.num_max_dispatch_tokens_per_rank,
            self.num_experts,
        )
```
**EN:** This block defines `NpuFuseEPDispatcher._get_buffer` and contains the main logic for this step. It mainly invokes `DeepEPBuffer.set_dispatch_mode_as_low_latency` and `DeepEPBuffer.get_deepep_buffer`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NpuFuseEPDispatcher._get_buffer`，并承载这一阶段的核心逻辑。 它主要调用 `DeepEPBuffer.set_dispatch_mode_as_low_latency` 和 `DeepEPBuffer.get_deepep_buffer`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `FuseEPDispatchOutput`, `FuseEPCombineInput`, and `NpuFuseEPDispatcher`. / **主要符号**：核心入口包括 `FuseEPDispatchOutput`、`FuseEPCombineInput` 以及 `NpuFuseEPDispatcher`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `logging`, and `typing.NamedTuple` / **标准库**：`__future__.annotations`、`logging` 以及 `typing.NamedTuple`
- **Third-party**: `torch` / **第三方依赖**：`torch`
- **Internal SGLang modules**: `sglang.srt.environ.envs`, `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputFormat`, `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPBuffer`, `sglang.srt.layers.moe.topk.TopKOutput`, and `sglang.srt.layers.moe.utils.DeepEPMode` / **SGLang 内部模块**：`sglang.srt.environ.envs`、`sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputFormat`、`sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPBuffer`、`sglang.srt.layers.moe.topk.TopKOutput` 以及 `sglang.srt.layers.moe.utils.DeepEPMode`
