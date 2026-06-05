# communicator_nsa_cp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/communicator_nsa_cp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement for the SGLang SRT runtime. It exposes symbols such as `nsa_enable_prefill_cp`, `NSACPLayerCommunicator`, `NSACPCommunicateSimpleFn`, and `NSACPCommunicateWithAllReduceAndLayerNormFn` and connects them to backend-specific paths such as `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运。它提供了 `nsa_enable_prefill_cp`、`NSACPLayerCommunicator`、`NSACPCommunicateSimpleFn` 以及 `NSACPCommunicateWithAllReduceAndLayerNormFn` 等符号，并把这些符号连接到 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: File header and module overview
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

### Lines 16-42: Module imports and dependency wiring
```python
from functools import partial
from typing import Callable, Optional

import torch

from sglang.srt.layers.attention.nsa.utils import (
    is_nsa_enable_prefill_cp,
    nsa_use_prefill_cp,
)
from sglang.srt.layers.communicator import (
    CommunicateContext,
    CommunicateSimpleFn,
    CommunicateSummableTensorPairFn,
    CommunicateWithAllReduceAndLayerNormFn,
    LayerCommunicator,
    LayerScatterModes,
    ScatterMode,
)
from sglang.srt.layers.dp_attention import (
    attn_cp_all_gather_into_tensor,
    attn_cp_reduce_scatter_tensor,
    get_attention_cp_group,
    get_local_dp_buffer,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** This section prepares the module namespace. It imports `functools.partial`, `typing.Callable`, `typing.Optional`, `torch`, `sglang.srt.layers.attention.nsa.utils.is_nsa_enable_prefill_cp`, and `sglang.srt.layers.attention.nsa.utils.nsa_use_prefill_cp`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `functools.partial`、`typing.Callable`、`typing.Optional`、`torch`、`sglang.srt.layers.attention.nsa.utils.is_nsa_enable_prefill_cp` 以及 `sglang.srt.layers.attention.nsa.utils.nsa_use_prefill_cp`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 43-49: Function `nsa_enable_prefill_cp` and its core logic
```python
def nsa_enable_prefill_cp():
    # After using cp, the communication mode of this part changes.
    # The three parts of prepare_attn, prepare_mlp, and postprocess_layer
    # no longer require additional communication for reduce, scatter, etc.
    return is_nsa_enable_prefill_cp()
```
**EN:** This block defines `nsa_enable_prefill_cp` and contains the main logic for this step. It mainly invokes `is_nsa_enable_prefill_cp`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `nsa_enable_prefill_cp`，并承载这一阶段的核心逻辑。 它主要调用 `is_nsa_enable_prefill_cp`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 50-50: Class `NSACPLayerCommunicator` declaration and shared state
```python
class NSACPLayerCommunicator(LayerCommunicator):
```
**EN:** This block introduces class `NSACPLayerCommunicator` and the state shared by its methods. It inherits from `LayerCommunicator`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `NSACPLayerCommunicator`，并定义其方法共享的状态。 它继承自 `LayerCommunicator`，说明了它在 SRT 层栈中的接入方式。

### Lines 51-69: `NSACPLayerCommunicator` initialization and state setup
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
        super().__init__(
            layer_scatter_modes,
            input_layernorm,
            post_attention_layernorm,
            allow_reduce_scatter,
            is_last_layer,
            qkv_latent_func,
        )
```
**EN:** This block defines `NSACPLayerCommunicator.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `NSACPLayerCommunicator.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。

### Lines 70-95: Internal helper `NSACPLayerCommunicator._post_init_communicate`
```python
    def _post_init_communicate(self):
        # SCATTERED in attn tp is different from SCATTERED in global tp when dp_size > 1
        if self.layer_scatter_modes.mlp_mode != ScatterMode.SCATTERED:
            assert (
                self._context.attn_dp_size == 1
            ), f"dp_size should be 1 when moe_runner_backend is none"
        self._communicate_simple_fn = NSACPCommunicateSimpleFn.get_fn(
            input_mode=ScatterMode.SCATTERED,
            output_mode=ScatterMode.SCATTERED,
            context=self._context,
        )
        self._communicate_with_all_reduce_and_layer_norm_fn = NSACPCommunicateWithAllReduceAndLayerNormFn.get_fn(
            hidden_states_input_mode=ScatterMode.SCATTERED,
            residual_input_mode=ScatterMode.SCATTERED,
            hidden_states_output_mode=self.layer_scatter_modes.mlp_mode,  # SCATTERED, FULL
            residual_output_mode=ScatterMode.SCATTERED,
            context=self._context,
        )
        self._communicate_summable_tensor_pair_fn = NSACPCommunicateSummableTensorPairFn.get_fn(
            hidden_states_input_mode=self.layer_scatter_modes.mlp_mode,  # SCATTERED, FULL
            residual_input_mode=ScatterMode.SCATTERED,
            output_mode=ScatterMode.SCATTERED,
            context=self._context,
        )
```
**EN:** This block defines `NSACPLayerCommunicator._post_init_communicate` and contains the main logic for this step. It mainly invokes `NSACPCommunicateSimpleFn.get_fn`, `NSACPCommunicateWithAllReduceAndLayerNormFn.get_fn`, and `NSACPCommunicateSummableTensorPairFn.get_fn`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._communicate_simple_fn`, `self._communicate_with_all_reduce_and_layer_norm_fn`, and `self._communicate_summable_tensor_pair_fn` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `NSACPLayerCommunicator._post_init_communicate`，并承载这一阶段的核心逻辑。 它主要调用 `NSACPCommunicateSimpleFn.get_fn`、`NSACPCommunicateWithAllReduceAndLayerNormFn.get_fn` 以及 `NSACPCommunicateSummableTensorPairFn.get_fn`，说明该流程会编排底层辅助函数或计算内核。 像 `self._communicate_simple_fn`、`self._communicate_with_all_reduce_and_layer_norm_fn` 以及 `self._communicate_summable_tensor_pair_fn` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 96-96: Class `NSACPCommunicateSimpleFn` declaration and shared state
```python
class NSACPCommunicateSimpleFn(CommunicateSimpleFn):
```
**EN:** This block introduces class `NSACPCommunicateSimpleFn` and the state shared by its methods. It inherits from `CommunicateSimpleFn`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `NSACPCommunicateSimpleFn`，并定义其方法共享的状态。 它继承自 `CommunicateSimpleFn`，说明了它在 SRT 层栈中的接入方式。

### Lines 97-108: `NSACPCommunicateSimpleFn.get_fn` getter for fn
```python
    @staticmethod
    def get_fn(
        input_mode: ScatterMode,
        output_mode: ScatterMode,
        context: CommunicateContext,
    ):
        if context.is_same_group_size(input_mode, output_mode):
            return NSACPCommunicateSimpleFn._trivial

        raise NotImplementedError(f"{input_mode=} {output_mode=}")
```
**EN:** This block defines `NSACPCommunicateSimpleFn.get_fn` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `context.is_same_group_size` and `NotImplementedError`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NSACPCommunicateSimpleFn.get_fn`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `context.is_same_group_size` 和 `NotImplementedError`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 109-116: Class `NSACPCommunicateWithAllReduceAndLayerNormFn` declaration and shared state
```python
class NSACPCommunicateWithAllReduceAndLayerNormFn(
    CommunicateWithAllReduceAndLayerNormFn
):
    """Besides communication, needs to
    1. All reduce in tp_attn_group on hidden_states
    2. Apply layer norm
    """
```
**EN:** This block introduces class `NSACPCommunicateWithAllReduceAndLayerNormFn` and the state shared by its methods. It inherits from `CommunicateWithAllReduceAndLayerNormFn`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Besides communication, needs to 1.
**CN:** 该代码块引入类 `NSACPCommunicateWithAllReduceAndLayerNormFn`，并定义其方法共享的状态。 它继承自 `CommunicateWithAllReduceAndLayerNormFn`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 117-140: `NSACPCommunicateWithAllReduceAndLayerNormFn.get_fn` getter for fn
```python
    @staticmethod
    def get_fn(
        hidden_states_input_mode: ScatterMode,
        residual_input_mode: ScatterMode,
        hidden_states_output_mode: ScatterMode,
        residual_output_mode: ScatterMode,
        context: CommunicateContext,
    ):
        assert hidden_states_input_mode == ScatterMode.SCATTERED
        assert residual_input_mode == ScatterMode.SCATTERED
        assert residual_output_mode == ScatterMode.SCATTERED
        if hidden_states_output_mode == ScatterMode.SCATTERED:
            return NSACPCommunicateWithAllReduceAndLayerNormFn._simple

        if hidden_states_output_mode == ScatterMode.FULL:
            return partial(
                NSACPCommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual,
                residual_input_mode=residual_input_mode,
            )

        raise NotImplementedError(
            f"{hidden_states_input_mode=} {residual_input_mode=} {hidden_states_output_mode=} {residual_output_mode=}"
        )
```
**EN:** This block defines `NSACPCommunicateWithAllReduceAndLayerNormFn.get_fn` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `NotImplementedError` and `partial`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NSACPCommunicateWithAllReduceAndLayerNormFn.get_fn`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `NotImplementedError` 和 `partial`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 141-167: Internal helper `NSACPCommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual`
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
        if hidden_states.shape[0] != 0:
            hidden_states, residual = layernorm(hidden_states, residual)
        # for prefill: attn tp scattered -> full
        # for decode: attn tp full -> full
        if nsa_use_prefill_cp(forward_batch):
            assert context.attn_dp_size == 1
            hidden_states, local_hidden_states = (
                get_local_dp_buffer(get_attention_cp_group()),
                hidden_states,
            )
            attn_cp_all_gather_into_tensor(
                hidden_states,
                local_hidden_states,
            )
        return hidden_states, residual
```
**EN:** This block defines `NSACPCommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `nsa_use_prefill_cp`, `layernorm`, `attn_cp_all_gather_into_tensor`, `get_local_dp_buffer`, and `get_attention_cp_group`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `residual`, and `local_hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NSACPCommunicateWithAllReduceAndLayerNormFn._gather_hidden_states_and_residual`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `nsa_use_prefill_cp`、`layernorm`、`attn_cp_all_gather_into_tensor`、`get_local_dp_buffer` 以及 `get_attention_cp_group`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`residual` 以及 `local_hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 168-170: Class `NSACPCommunicateSummableTensorPairFn` declaration and shared state
```python
class NSACPCommunicateSummableTensorPairFn(CommunicateSummableTensorPairFn):
    """It is allowed to make (hidden_states, residual) := (hidden_states + residual, None) if needed."""
```
**EN:** This block introduces class `NSACPCommunicateSummableTensorPairFn` and the state shared by its methods. It inherits from `CommunicateSummableTensorPairFn`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: It is allowed to make (hidden_states, residual) := (hidden_states + residual, None) if needed.
**CN:** 该代码块引入类 `NSACPCommunicateSummableTensorPairFn`，并定义其方法共享的状态。 它继承自 `CommunicateSummableTensorPairFn`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 171-197: `NSACPCommunicateSummableTensorPairFn.get_fn` getter for fn
```python
    @staticmethod
    def get_fn(
        hidden_states_input_mode: ScatterMode,
        residual_input_mode: ScatterMode,
        output_mode: ScatterMode,
        context: CommunicateContext,
    ):
        # Check exact enum match first: even if group sizes happen to be equal
        # (e.g. tp_size == attn_cp_size makes FULL and SCATTERED both size 1),
        # FULL and SCATTERED have different data layouts under CP and require
        # an explicit scatter operation.
        if (
            (hidden_states_input_mode == ScatterMode.FULL)
            and (residual_input_mode == ScatterMode.SCATTERED)
            and (output_mode == ScatterMode.SCATTERED)
        ):
            return NSACPCommunicateSummableTensorPairFn._scatter_hidden_states

        if context.is_same_group_size(
            hidden_states_input_mode, output_mode
        ) and context.is_same_group_size(residual_input_mode, output_mode):
            return NSACPCommunicateSummableTensorPairFn._trivial

        raise NotImplementedError(
            f"{hidden_states_input_mode=} {residual_input_mode=} {output_mode=}"
        )
```
**EN:** This block defines `NSACPCommunicateSummableTensorPairFn.get_fn` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `NotImplementedError` and `context.is_same_group_size`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NSACPCommunicateSummableTensorPairFn.get_fn`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `NotImplementedError` 和 `context.is_same_group_size`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 198-215: Internal helper `NSACPCommunicateSummableTensorPairFn._scatter_hidden_states`
```python
    @staticmethod
    def _scatter_hidden_states(
        hidden_states: torch.Tensor,
        residual: torch.Tensor,
        forward_batch: ForwardBatch,
        context: CommunicateContext,
        allow_reduce_scatter: bool = False,
    ):
        # for prefill: full -> attn tp scattered
        # for decode: full -> attn tp full
        if nsa_use_prefill_cp(forward_batch):
            assert context.attn_dp_size == 1
            input_hidden_states = hidden_states
            hidden_states = hidden_states.tensor_split(context.attn_cp_size)[
                context.attn_cp_rank
            ]
            attn_cp_reduce_scatter_tensor(hidden_states, input_hidden_states)
        return hidden_states, residual
```
**EN:** This block defines `NSACPCommunicateSummableTensorPairFn._scatter_hidden_states` and contains the main logic for this step. Decorators like `staticmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `nsa_use_prefill_cp`, `attn_cp_reduce_scatter_tensor`, and `hidden_states.tensor_split`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `input_hidden_states` and `hidden_states` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NSACPCommunicateSummableTensorPairFn._scatter_hidden_states`，并承载这一阶段的核心逻辑。 像 `staticmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `nsa_use_prefill_cp`、`attn_cp_reduce_scatter_tensor` 以及 `hidden_states.tensor_split`，说明该流程会编排底层辅助函数或计算内核。 像 `input_hidden_states` 和 `hidden_states` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `nsa_enable_prefill_cp`, `NSACPLayerCommunicator`, `NSACPCommunicateSimpleFn`, `NSACPCommunicateWithAllReduceAndLayerNormFn`, and `NSACPCommunicateSummableTensorPairFn`. / **主要符号**：核心入口包括 `nsa_enable_prefill_cp`、`NSACPLayerCommunicator`、`NSACPCommunicateSimpleFn`、`NSACPCommunicateWithAllReduceAndLayerNormFn` 以及 `NSACPCommunicateSummableTensorPairFn`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。

## Dependencies / 依赖关系
- **Standard library**: `functools.partial`, `typing.Callable`, and `typing.Optional` / **标准库**：`functools.partial`、`typing.Callable` 以及 `typing.Optional`
- **Third-party**: `torch` / **第三方依赖**：`torch`
- **Internal SGLang modules**: `sglang.srt.layers.attention.nsa.utils.is_nsa_enable_prefill_cp`, `sglang.srt.layers.attention.nsa.utils.nsa_use_prefill_cp`, `sglang.srt.layers.communicator.CommunicateContext`, `sglang.srt.layers.communicator.CommunicateSimpleFn`, `sglang.srt.layers.communicator.CommunicateSummableTensorPairFn`, `sglang.srt.layers.communicator.CommunicateWithAllReduceAndLayerNormFn`, `sglang.srt.layers.communicator.LayerCommunicator`, `sglang.srt.layers.communicator.LayerScatterModes`, `sglang.srt.layers.communicator.ScatterMode`, `sglang.srt.layers.dp_attention.attn_cp_all_gather_into_tensor`, `sglang.srt.layers.dp_attention.attn_cp_reduce_scatter_tensor`, and `sglang.srt.layers.dp_attention.get_attention_cp_group` / **SGLang 内部模块**：`sglang.srt.layers.attention.nsa.utils.is_nsa_enable_prefill_cp`、`sglang.srt.layers.attention.nsa.utils.nsa_use_prefill_cp`、`sglang.srt.layers.communicator.CommunicateContext`、`sglang.srt.layers.communicator.CommunicateSimpleFn`、`sglang.srt.layers.communicator.CommunicateSummableTensorPairFn`、`sglang.srt.layers.communicator.CommunicateWithAllReduceAndLayerNormFn`、`sglang.srt.layers.communicator.LayerCommunicator`、`sglang.srt.layers.communicator.LayerScatterModes`、`sglang.srt.layers.communicator.ScatterMode`、`sglang.srt.layers.dp_attention.attn_cp_all_gather_into_tensor`、`sglang.srt.layers.dp_attention.attn_cp_reduce_scatter_tensor` 以及 `sglang.srt.layers.dp_attention.get_attention_cp_group`
