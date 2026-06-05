# moe_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/runner/moe_runner.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE runner orchestration and shared-expert execution; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 执行器编排与共享专家执行；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-44 — imports and setup
```python
from collections.abc import Callable
from contextlib import nullcontext
from typing import TYPE_CHECKING

import torch
import torch.nn.functional as F

from vllm.distributed import (
    get_ep_group,
    get_pcp_group,
    tensor_model_parallel_all_reduce,
)
from vllm.forward_context import (
    ForwardContext,
    get_forward_context,
    is_forward_context_available,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
)
from vllm.model_executor.layers.fused_moe.fused_moe_method_base import (
    FusedMoEMethodBase,
)
from vllm.model_executor.layers.fused_moe.router.fused_moe_router import (
    FusedMoERouter,
)
from vllm.model_executor.layers.fused_moe.router.zero_expert_router import (
    ZeroExpertRouter,
)
from vllm.model_executor.layers.fused_moe.runner.moe_runner_interface import (
    MoERunnerInterface,
)
from vllm.model_executor.layers.fused_moe.runner.shared_experts import (
    SharedExperts,
    SharedExpertsOrder,
)
from vllm.platforms import current_platform
from vllm.utils.torch_utils import (
    _USE_LAYERNAME,
    LayerName,
    direct_register_custom_op,
)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 47-61 — function `get_layer_from_name`
```python
def get_layer_from_name(layer_name: str) -> torch.nn.Module:
    forward_context: ForwardContext = get_forward_context()
    if not _USE_LAYERNAME and layer_name == "from_forward_context":
        all_moe_layers = forward_context.all_moe_layers
        assert all_moe_layers is not None
        moe_layer_index = forward_context.moe_layer_index
        if moe_layer_index >= len(all_moe_layers):
            raise AssertionError(
                "We expected the number of MOE layers in `all_moe_layers` "
                "to be equal to the number of "
                "{vllm.moe_forward, vllm.moe_forward_shared} calls."
            )
        layer_name = all_moe_layers[moe_layer_index]
        forward_context.moe_layer_index += 1
    return forward_context.no_compile_layers[layer_name]
```
**EN:** This function defines `get_layer_from_name`. It derives and returns computed metadata or outputs needed by later stages. The main inputs are `layer_name`. Key calls include `get_forward_context`, `len`, `AssertionError`. It writes or updates `forward_context`, `all_moe_layers`, `moe_layer_index`, `layer_name`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `get_layer_from_name`。 它推导并返回后续阶段所需的元数据或输出。 其主要输入参数包括 `layer_name`。 关键调用包括 `get_forward_context`, `len`, `AssertionError`。 它会写入或更新 `forward_context`, `all_moe_layers`, `moe_layer_index`, `layer_name`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 74-82 — function `_resolve_layer_name`
```python
@torch.compiler.assume_constant_result
def _resolve_layer_name(layer_name: str | LayerName) -> str:
    from torch._library.fake_class_registry import FakeScriptObject

    if isinstance(layer_name, LayerName):
        return layer_name.value
    elif isinstance(layer_name, FakeScriptObject):
        return layer_name.real_obj.value
    return layer_name
```
**EN:** This function defines `_resolve_layer_name`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `layer_name`. Key calls include `isinstance`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_resolve_layer_name`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `layer_name`。 关键调用包括 `isinstance`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 90-105 — function `_moe_forward`
```python
def _moe_forward(
    hidden_states: torch.Tensor,
    router_logits: torch.Tensor,
    shared_experts_input: torch.Tensor | None,
    input_ids: torch.Tensor | None,
    layer_name: _layer_name_type,
    hidden_dim_unpadded: int,
) -> torch.Tensor:
    layer = get_layer_from_name(_resolve_layer_name(layer_name))
    return layer.runner._forward_impl(
        layer,
        hidden_states,
        router_logits,
        shared_experts_input,
        input_ids,
    )
```
**EN:** This function defines `_moe_forward`. It executes the main forward/runtime path for this component. The main inputs are `hidden_states`, `router_logits`, `shared_experts_input`, `input_ids`, `layer_name`, `hidden_dim_unpadded`. Key calls include `get_layer_from_name`, `layer.runner._forward_impl`, `_resolve_layer_name`. It writes or updates `layer`.
**CN:** 该函数定义 `_moe_forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `hidden_states`, `router_logits`, `shared_experts_input`, `input_ids`, `layer_name`, `hidden_dim_unpadded`。 关键调用包括 `get_layer_from_name`, `layer.runner._forward_impl`, `_resolve_layer_name`。 它会写入或更新 `layer`。

### Lines 108-122 — function `_moe_forward_fake`
```python
def _moe_forward_fake(
    hidden_states: torch.Tensor,
    router_logits: torch.Tensor,
    shared_experts_input: torch.Tensor | None,
    input_ids: torch.Tensor | None,
    layer_name: _layer_name_type,
    hidden_dim_unpadded: int,
) -> torch.Tensor:
    # `hidden_dim_unpadded > 0` only on the TRT-LLM MXFP4 path, where the
    # real kernel writes narrower than `hidden_states.shape[-1]`. Plumbed
    # as an op arg (not peeked from the layer registry) to keep the fake
    # a pure shape function of its inputs and preserve subgraph dedup.
    if hidden_dim_unpadded > 0:
        return hidden_states.new_empty((*hidden_states.shape[:-1], hidden_dim_unpadded))
    return torch.empty_like(hidden_states)
```
**EN:** This function defines `_moe_forward_fake`. It executes the main forward/runtime path for this component. The main inputs are `hidden_states`, `router_logits`, `shared_experts_input`, `input_ids`, `layer_name`, `hidden_dim_unpadded`. Key calls include `torch.empty_like`, `hidden_states.new_empty`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_moe_forward_fake`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `hidden_states`, `router_logits`, `shared_experts_input`, `input_ids`, `layer_name`, `hidden_dim_unpadded`。 关键调用包括 `torch.empty_like`, `hidden_states.new_empty`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 125-140 — function `_moe_forward_shared`
```python
def _moe_forward_shared(
    hidden_states: torch.Tensor,
    router_logits: torch.Tensor,
    shared_experts_input: torch.Tensor | None,
    input_ids: torch.Tensor | None,
    layer_name: _layer_name_type,
    hidden_dim_unpadded: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    layer = get_layer_from_name(_resolve_layer_name(layer_name))
    return layer.runner._forward_impl(
        layer,
        hidden_states,
        router_logits,
        shared_experts_input,
        input_ids,
    )
```
**EN:** This function defines `_moe_forward_shared`. It executes the main forward/runtime path for this component. The main inputs are `hidden_states`, `router_logits`, `shared_experts_input`, `input_ids`, `layer_name`, `hidden_dim_unpadded`. Key calls include `get_layer_from_name`, `layer.runner._forward_impl`, `_resolve_layer_name`. It writes or updates `layer`.
**CN:** 该函数定义 `_moe_forward_shared`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `hidden_states`, `router_logits`, `shared_experts_input`, `input_ids`, `layer_name`, `hidden_dim_unpadded`。 关键调用包括 `get_layer_from_name`, `layer.runner._forward_impl`, `_resolve_layer_name`。 它会写入或更新 `layer`。

### Lines 143-164 — function `_moe_forward_shared_fake`
```python
def _moe_forward_shared_fake(
    hidden_states: torch.Tensor,
    router_logits: torch.Tensor,
    shared_experts_input: torch.Tensor | None,
    input_ids: torch.Tensor | None,
    layer_name: _layer_name_type,
    hidden_dim_unpadded: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    # `fused_out`: see `_moe_forward_fake` for hidden_dim_unpadded semantics.
    # `shared_out`: matches `shared_experts_input` if provided (latent MoE),
    # else `hidden_states`.
    if hidden_dim_unpadded > 0:
        fused_out = hidden_states.new_empty(
            (*hidden_states.shape[:-1], hidden_dim_unpadded)
        )
    else:
        fused_out = torch.empty_like(hidden_states)
    if shared_experts_input is not None:
        shared_out = torch.empty_like(shared_experts_input)
    else:
        shared_out = torch.empty_like(hidden_states)
    return shared_out, fused_out
```
**EN:** This function defines `_moe_forward_shared_fake`. It executes the main forward/runtime path for this component. The main inputs are `hidden_states`, `router_logits`, `shared_experts_input`, `input_ids`, `layer_name`, `hidden_dim_unpadded`. Key calls include `hidden_states.new_empty`, `torch.empty_like`. It writes or updates `fused_out`, `shared_out`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_moe_forward_shared_fake`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `hidden_states`, `router_logits`, `shared_experts_input`, `input_ids`, `layer_name`, `hidden_dim_unpadded`。 关键调用包括 `hidden_states.new_empty`, `torch.empty_like`。 它会写入或更新 `fused_out`, `shared_out`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 184-190 — function `_unpack`
```python
def _unpack(
    result: torch.Tensor | tuple[torch.Tensor, torch.Tensor],
) -> tuple[torch.Tensor | None, torch.Tensor]:
    if isinstance(result, tuple):
        return result
    else:
        return (None, result)
```
**EN:** This function defines `_unpack`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `result`. Key calls include `isinstance`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_unpack`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `result`。 关键调用包括 `isinstance`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 193-799 — class `MoERunner`
```python
class MoERunner(MoERunnerInterface):
    """
    Standard MoE runner implementation for executing Mixture of Experts layers.

    This is the primary concrete implementation of MoE execution logic, providing
    comprehensive support for standard MoE operations. It handles:
    - Expert routing and token dispatching using various routing strategies
    - Shared experts computation with optional parallel execution using CUDA streams
    - Tensor model parallel and expert parallel operations
    - Multiple quantization methods and optimized kernel selection
    - Both monolithic and decomposed expert execution paths
    - Integration with various parallel execution modes (TP, EP, DP)

    The runner orchestrates the complete MoE forward pass including routing tokens
# ... omitted for brevity ...
                hidden_states,
            )
```
**EN:** This class defines `MoERunner`. It inherits from `MoERunnerInterface`. Standard MoE runner implementation for executing Mixture of Experts layers. Important methods include `__init__`, `forward`, `shared_experts`, `is_internal_router`, `apply_routed_input_transform`, `apply_routed_output_transform`. Key calls include `super.__init__`, `self._select_forward`, `getattr`, `isinstance`, `self._maybe_apply_shared_experts`, `get_forward_context`. It writes or updates `moe_config`, `router`, `routed_input_transform`, `routed_output_transform`, `routed_scaling_factor`, `gate`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `MoERunner`。 它继承自 `MoERunnerInterface`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `forward`, `shared_experts`, `is_internal_router`, `apply_routed_input_transform`, `apply_routed_output_transform`。 关键调用包括 `super.__init__`, `self._select_forward`, `getattr`, `isinstance`, `self._maybe_apply_shared_experts`, `get_forward_context`。 它会写入或更新 `moe_config`, `router`, `routed_input_transform`, `routed_output_transform`, `routed_scaling_factor`, `gate`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 216-266 — method `MoERunner.__init__`
```python
    def __init__(
        self,
        layer_name: str,
        moe_config: FusedMoEConfig,
        router: FusedMoERouter,
        routed_input_transform: torch.nn.Module | None,
        gate: torch.nn.Module | None,
        shared_experts: torch.nn.Module | None,
        quant_method: FusedMoEMethodBase,
        enable_dbo: bool,
        shared_expert_gate: torch.nn.Module | None = None,
        routed_output_transform: torch.nn.Module | None = None,
        routed_scaling_factor: float = 1.0,
    ):
        super().__init__()
        self.moe_config = moe_config
        self.router = router
        self.routed_input_transform = routed_input_transform
        self.routed_output_transform = routed_output_transform
        self.routed_scaling_factor = routed_scaling_factor
        self.gate = gate
        self.shared_expert_gate = shared_expert_gate
        self._quant_method = quant_method
        self.enable_dbo = enable_dbo

        # When both gates are present and FSE is enabled, fuse their
        # weight matrices into [num_experts + num_shared, hidden] so one
        # F.linear produces combined logits. The topk kernel can then
        # apply routing softmax and shared expert activation (sigmoid)
        # in a single launch.
        self._fse_fuse_gate = gate is not None and shared_expert_gate is not None
        self._combined_gate_weight: torch.Tensor | None = None

        self._shared_experts: SharedExperts | None = None
        if shared_experts is not None:
            self._shared_experts = SharedExperts(
                shared_experts,
                moe_config=moe_config,
                # Note: For now we must pass quant_method along to SharedExperts so it
                # can property determine where the shared experts are supposed to be
                # called, i.e. by a MK or by the MoERunner.
                # Once the MK can be created upfront, we can just pass in the proper
                # flags derived from the quant_method's MK.
                quant_method=quant_method,
# ... omitted for brevity ...

        self._forward_entry = self._select_forward()
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `layer_name`, `moe_config`, `router`, `routed_input_transform`, `gate`, `shared_experts`. Key calls include `super.__init__`, `self._select_forward`, `SharedExperts`, `super`. It writes or updates `moe_config`, `router`, `routed_input_transform`, `routed_output_transform`, `routed_scaling_factor`, `gate`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `layer_name`, `moe_config`, `router`, `routed_input_transform`, `gate`, `shared_experts`。 关键调用包括 `super.__init__`, `self._select_forward`, `SharedExperts`, `super`。 它会写入或更新 `moe_config`, `router`, `routed_input_transform`, `routed_output_transform`, `routed_scaling_factor`, `gate`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 592-678 — method `MoERunner.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        input_ids: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """Invoke the fused moe layer.

        Input:
        - hidden_states
        - router_logits

        Output:
        - The new hidden_states.

        Calling sequence
        - forward
          - self._forward_entry (_moe_forward or _moe_forward_shared custom op)
            - _forward_impl

        Note: The existence of _moe_forward and _moe_forward_shared custom ops are due
        to the following reason:
        1. pytorch cannot handle union types in custom op signatures so
           _moe_forward and _moe_forward_shared must be split.
        """

        # Apply transform for routed experts (e.g., latent projection
        # for latent MoE)
        hidden_states, shared_experts_input = self.apply_routed_input_transform(
            hidden_states
        )

        # Record before `_maybe_pad_hidden_states` pads activations to match
        # `moe_config.hidden_dim`, e.g. after `align_trtllm_fp4_moe_hidden_dim_for_fi`
        # so routed output can be trimmed before
        # shared+routed add / latent up proj if needed.
        routed_hidden_dim = hidden_states.shape[-1]
        hidden_states, og_hidden_dim = self._maybe_pad_hidden_states(
            shared_experts_input,
            hidden_states,
        )
        hidden_dim_was_padded = hidden_states.shape[-1] > routed_hidden_dim

        result = self._forward_entry(
# ... omitted for brevity ...

        return self._maybe_add_zero_expert_output(result)
```
**EN:** This method defines `forward`. Invoke the fused moe layer. The main inputs are `hidden_states`, `router_logits`, `input_ids`. Key calls include `self.apply_routed_input_transform`, `self._maybe_pad_hidden_states`, `self._forward_entry`, `_unpack`, `self._maybe_reduce_shared_expert_output`, `self._maybe_apply_routed_scale_to_output`. It writes or updates `hidden_states`, `shared_experts_input`, `routed_hidden_dim`, `og_hidden_dim`, `hidden_dim_was_padded`, `result`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `router_logits`, `input_ids`。 关键调用包括 `self.apply_routed_input_transform`, `self._maybe_pad_hidden_states`, `self._forward_entry`, `_unpack`, `self._maybe_reduce_shared_expert_output`, `self._maybe_apply_routed_scale_to_output`。 它会写入或更新 `hidden_states`, `shared_experts_input`, `routed_hidden_dim`, `og_hidden_dim`, `hidden_dim_was_padded`, `result`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 281-283 — method `MoERunner.shared_experts`
```python
    @property
    def shared_experts(self) -> SharedExperts | None:
        return self._shared_experts
```
**EN:** This method defines `shared_experts`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `shared_experts`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 305-306 — method `MoERunner.is_internal_router`
```python
    def is_internal_router(self) -> bool:
        return self.gate is not None
```
**EN:** This method defines `is_internal_router`. It computes routing scores or expert-selection behavior for MoE execution.
**CN:** 该方法定义 `is_internal_router`。 它为 MoE 执行计算路由得分或专家选择行为。

### Lines 308-331 — method `MoERunner.apply_routed_input_transform`
```python
    def apply_routed_input_transform(
        self, hidden_states: torch.Tensor
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """Apply transform for routed experts (e.g., latent projection).

        This is called by FusedMoE.forward_native. The original hidden_states
        is saved separately so shared experts get [S, hidden_size] while
        routed experts get the transformed [S, moe_latent_size].

        Returns (possibly transformed) hidden states and the input for shared
        experts (or None if there are no shared experts).
        """
        if self.routed_input_transform is not None:
            result = self.routed_input_transform(hidden_states)
            # ReplicatedLinear returns (output, extra_bias) tuple.
            # We only need the output tensor; extra_bias is not used here.
            if isinstance(result, tuple):
                return result[0], hidden_states
            return result, hidden_states

        return (
            hidden_states,
            hidden_states if self._shared_experts is not None else None,
        )
```
**EN:** This method defines `apply_routed_input_transform`. Apply transform for routed experts (e.g., latent projection). The main inputs are `hidden_states`. Key calls include `self.routed_input_transform`, `isinstance`. It writes or updates `result`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `apply_routed_input_transform`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`。 关键调用包括 `self.routed_input_transform`, `isinstance`。 它会写入或更新 `result`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Moe runner orchestration and shared-expert execution / [CN] MoE 执行器编排与共享专家执行
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `get_layer_from_name`, `_resolve_layer_name`, `_moe_forward`, `_moe_forward_fake` / [CN] 核心符号：`get_layer_from_name`, `_resolve_layer_name`, `_moe_forward`, `_moe_forward_fake`

## Dependencies / 依赖关系
- **External**: `collections.abc`, `contextlib`, `typing`, `torch`, `torch.nn.functional` / **外部依赖**: `collections.abc`, `contextlib`, `typing`, `torch`, `torch.nn.functional`
- **Internal**: `vllm.distributed`, `vllm.forward_context`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe_method_base`, `vllm.model_executor.layers.fused_moe.router.fused_moe_router`, `vllm.model_executor.layers.fused_moe.router.zero_expert_router`, `vllm.model_executor.layers.fused_moe.runner.moe_runner_interface`, `vllm.model_executor.layers.fused_moe.runner.shared_experts`, `vllm.platforms`, `vllm.utils.torch_utils` / **内部依赖**: `vllm.distributed`, `vllm.forward_context`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe_method_base`, `vllm.model_executor.layers.fused_moe.router.fused_moe_router`, `vllm.model_executor.layers.fused_moe.router.zero_expert_router`, `vllm.model_executor.layers.fused_moe.runner.moe_runner_interface`, `vllm.model_executor.layers.fused_moe.runner.shared_experts`, `vllm.platforms`, `vllm.utils.torch_utils`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
