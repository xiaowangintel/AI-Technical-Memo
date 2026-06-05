# shared_experts.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/runner/shared_experts.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE runner orchestration and shared-expert execution; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 执行器编排与共享专家执行；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-24 — imports and setup
```python
from enum import IntEnum

import torch

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
)
from vllm.model_executor.layers.quantization.base_config import (
    QuantizeMethodBase,
)
from vllm.platforms import current_platform
from vllm.utils.torch_utils import (
    aux_stream,
    current_stream,
)
from vllm.v1.worker.ubatching import (
    dbo_current_ubatch_id,
)

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 27-38 — class `SharedExpertsOrder`
```python
class SharedExpertsOrder(IntEnum):
    # No shared experts.
    NONE = (0,)

    # No overlap - defensively called before MK.
    NO_OVERLAP = (1,)

    # Overlapped with dispatch/combine in DP/EP - called by the MK.
    MK_INTERNAL_OVERLAPPED = (2,)

    # Overlapped with the gate, router, experts in aux stream.
    MULTI_STREAM_OVERLAPPED = (3,)
```
**EN:** This class defines `SharedExpertsOrder`. It inherits from `IntEnum`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. It writes or updates `NONE`, `NO_OVERLAP`, `MK_INTERNAL_OVERLAPPED`, `MULTI_STREAM_OVERLAPPED`.
**CN:** 该类定义了 `SharedExpertsOrder`。 它继承自 `IntEnum`。 它管理专家分发、融合内核或 MoE 特有的张量流。 它会写入或更新 `NONE`, `NO_OVERLAP`, `MK_INTERNAL_OVERLAPPED`, `MULTI_STREAM_OVERLAPPED`。

### Lines 41-178 — class `SharedExperts`
```python
class SharedExperts:
    def __init__(
        self,
        layer: torch.nn.Module,
        moe_config: FusedMoEConfig,
        quant_method: QuantizeMethodBase,
        enable_dbo: bool,
    ):
        from vllm.model_executor.layers.fused_moe.fused_moe_method_base import (
            FusedMoEMethodBase,
        )

        # quant_method must be a FusedMoEMethodBase but we can't use the type
        # due to circular imports.
# ... omitted for brevity ...

        assert self._output[self._output_idx] is not None
```
**EN:** This class defines `SharedExperts`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`, `apply`, `maybe_sync_shared_experts_stream`, `output`. Key calls include `isinstance`, `self._determine_shared_experts_order`, `current_stream.wait_stream`, `logger.debug_once`, `aux_stream`, `current_platform.is_cuda`. It writes or updates `enable_dbo`, `_output`, `_layer`, `_moe_config`, `_quant_method`, `parallel_config`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `SharedExperts`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`, `apply`, `maybe_sync_shared_experts_stream`, `output`。 关键调用包括 `isinstance`, `self._determine_shared_experts_order`, `current_stream.wait_stream`, `logger.debug_once`, `aux_stream`, `current_platform.is_cuda`。 它会写入或更新 `enable_dbo`, `_output`, `_layer`, `_moe_config`, `_quant_method`, `parallel_config`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 42-79 — method `SharedExperts.__init__`
```python
    def __init__(
        self,
        layer: torch.nn.Module,
        moe_config: FusedMoEConfig,
        quant_method: QuantizeMethodBase,
        enable_dbo: bool,
    ):
        from vllm.model_executor.layers.fused_moe.fused_moe_method_base import (
            FusedMoEMethodBase,
        )

        # quant_method must be a FusedMoEMethodBase but we can't use the type
        # due to circular imports.
        assert isinstance(quant_method, FusedMoEMethodBase)

        # The SharedExperts need to handle DBO since they can be called from
        # an MK's finalize method.  We keep a list of outputs indexed by current
        # DBO ubatch id to handle this case.  If DBO is not enabled, the
        # index is always 0 and the second output list element is ignored.
        self.enable_dbo = enable_dbo
        self._output: list[torch.Tensor | None] = [None, None]
        self._layer = layer
        self._moe_config = moe_config
        self._quant_method = quant_method

        # Allow disabling of the separate shared experts stream for
        # debug purposes.
        # TODO: Remove this after more extensive testings with TP/DP
        # and other execution modes
        if envs.VLLM_DISABLE_SHARED_EXPERTS_STREAM:
            logger.debug_once("Disabling MoE shared_experts cuda stream")
            self._stream = None
        else:
            # TODO(rob): enable shared expert overlap with non-cuda-alike.
            # aux_stream() returns None on non-cuda-alike platforms.
            self._stream = aux_stream()
            if self._stream is not None:
                logger.debug_once("Enabled separate cuda stream for MoE shared_experts")
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `layer`, `moe_config`, `quant_method`, `enable_dbo`. Key calls include `isinstance`, `logger.debug_once`, `aux_stream`. It writes or updates `enable_dbo`, `_output`, `_layer`, `_moe_config`, `_quant_method`, `_stream`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `layer`, `moe_config`, `quant_method`, `enable_dbo`。 关键调用包括 `isinstance`, `logger.debug_once`, `aux_stream`。 它会写入或更新 `enable_dbo`, `_output`, `_layer`, `_moe_config`, `_quant_method`, `_stream`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 159-178 — method `SharedExperts.apply`
```python
    def apply(
        self,
        shared_experts_input: torch.Tensor,
        order: SharedExpertsOrder,
    ):
        experts_order = self._determine_shared_experts_order(shared_experts_input)

        if order != experts_order:
            return None

        assert self._output[self._output_idx] is None

        if order == SharedExpertsOrder.MULTI_STREAM_OVERLAPPED:
            self._output[self._output_idx] = self._run_in_aux_stream(
                shared_experts_input
            )
        else:
            self._output[self._output_idx] = self._layer(shared_experts_input)

        assert self._output[self._output_idx] is not None
```
**EN:** This method defines `apply`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `shared_experts_input`, `order`. Key calls include `self._determine_shared_experts_order`, `self._run_in_aux_stream`, `self._layer`. It writes or updates `experts_order`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `shared_experts_input`, `order`。 关键调用包括 `self._determine_shared_experts_order`, `self._run_in_aux_stream`, `self._layer`。 它会写入或更新 `experts_order`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 114-133 — method `SharedExperts.maybe_sync_shared_experts_stream`
```python
    def maybe_sync_shared_experts_stream(
        self,
        shared_experts_input: torch.Tensor,
    ):
        experts_order = self._determine_shared_experts_order(shared_experts_input)

        if experts_order == SharedExpertsOrder.MULTI_STREAM_OVERLAPPED:
            assert self._stream is not None
            assert self._moe_config.disable_inplace

            # Record that the clone will be used by shared_experts_stream
            # to avoid gc issue from deallocation of hidden_states_clone
            # For more details: https://docs.pytorch.org/docs/stable/generated/torch.Tensor.record_stream.html # noqa: E501
            # NOTE: We don't need shared_output.record_stream(current_stream())
            # because we synch the streams before using shared_output.
            shared_experts_input.record_stream(self._stream)

            # Mark sync start point for the aux stream since we will
            # run in parallel with router/gate.
            self._stream.wait_stream(current_stream())
```
**EN:** This method defines `maybe_sync_shared_experts_stream`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `shared_experts_input`. Key calls include `self._determine_shared_experts_order`, `shared_experts_input.record_stream`, `self._stream.wait_stream`, `current_stream`. It writes or updates `experts_order`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `maybe_sync_shared_experts_stream`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `shared_experts_input`。 关键调用包括 `self._determine_shared_experts_order`, `shared_experts_input.record_stream`, `self._stream.wait_stream`, `current_stream`。 它会写入或更新 `experts_order`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 152-157 — method `SharedExperts.output`
```python
    @property
    def output(self) -> torch.Tensor:
        assert self._output[self._output_idx] is not None
        output = self._output[self._output_idx]
        self._output[self._output_idx] = None
        return output
```
**EN:** This method defines `output`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. It writes or updates `output`.
**CN:** 该方法定义 `output`。 它管理专家分发、融合内核或 MoE 特有的张量流。 它会写入或更新 `output`。

## Key Concepts / 关键概念
- [EN] Moe runner orchestration and shared-expert execution / [CN] MoE 执行器编排与共享专家执行
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `SharedExpertsOrder`, `SharedExperts` / [CN] 核心符号：`SharedExpertsOrder`, `SharedExperts`

## Dependencies / 依赖关系
- **External**: `enum`, `torch` / **外部依赖**: `enum`, `torch`
- **Internal**: `vllm.envs`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.base_config`, `vllm.platforms`, `vllm.utils.torch_utils`, `vllm.v1.worker.ubatching` / **内部依赖**: `vllm.envs`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.quantization.base_config`, `vllm.platforms`, `vllm.utils.torch_utils`, `vllm.v1.worker.ubatching`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
