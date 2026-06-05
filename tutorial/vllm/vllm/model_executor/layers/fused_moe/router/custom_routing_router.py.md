# custom_routing_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/router/custom_routing_router.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE router scoring and top-k expert selection; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 路由打分与 top-k 专家选择；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-9 — imports and setup
```python
from collections.abc import Callable

import torch

from vllm.distributed.eplb.eplb_state import EplbLayerState
from vllm.model_executor.layers.fused_moe.config import RoutingMethodType
from vllm.model_executor.layers.fused_moe.router.base_router import BaseRouter
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 12-64 — class `CustomRoutingRouter`
```python
class CustomRoutingRouter(BaseRouter):
    """Router using a custom user-provided routing function."""

    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        custom_routing_function: Callable,
        eplb_state: EplbLayerState | None = None,
        renormalize: bool = True,
        indices_type_getter: Callable[[], torch.dtype | None] | None = None,
    ):
        super().__init__(
            top_k=top_k,
# ... omitted for brevity ...
            torch.int32 if indices_type is None else indices_type
        )
```
**EN:** This class defines `CustomRoutingRouter`. It inherits from `BaseRouter`. Router using a custom user-provided routing function. Important methods include `__init__`, `routing_method_type`, `_compute_routing`. Key calls include `super.__init__`, `self.custom_routing_function`, `topk_weights.to`, `topk_ids.to`, `super`. It writes or updates `custom_routing_function`, `renormalize`, `topk_weights`, `topk_ids`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `CustomRoutingRouter`。 它继承自 `BaseRouter`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `routing_method_type`, `_compute_routing`。 关键调用包括 `super.__init__`, `self.custom_routing_function`, `topk_weights.to`, `topk_ids.to`, `super`。 它会写入或更新 `custom_routing_function`, `renormalize`, `topk_weights`, `topk_ids`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 15-31 — method `CustomRoutingRouter.__init__`
```python
    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        custom_routing_function: Callable,
        eplb_state: EplbLayerState | None = None,
        renormalize: bool = True,
        indices_type_getter: Callable[[], torch.dtype | None] | None = None,
    ):
        super().__init__(
            top_k=top_k,
            global_num_experts=global_num_experts,
            eplb_state=eplb_state,
            indices_type_getter=indices_type_getter,
        )
        self.custom_routing_function = custom_routing_function
        self.renormalize = renormalize
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `top_k`, `global_num_experts`, `custom_routing_function`, `eplb_state`, `renormalize`, `indices_type_getter`. Key calls include `super.__init__`, `super`. It writes or updates `custom_routing_function`, `renormalize`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `top_k`, `global_num_experts`, `custom_routing_function`, `eplb_state`, `renormalize`, `indices_type_getter`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `custom_routing_function`, `renormalize`。

### Lines 33-44 — method `CustomRoutingRouter.routing_method_type`
```python
    @property
    def routing_method_type(self) -> RoutingMethodType:
        from vllm.model_executor.models.cohere2_moe import token_choice_with_bias
        from vllm.model_executor.models.llama4 import Llama4MoE

        # NOTE: FLASHINFER_TRTLLM support the Llama4 router.
        if self.custom_routing_function == Llama4MoE.custom_routing_function:
            return RoutingMethodType.Llama4
        # Cohere MoE uses a sigmoid -> top-k -> renormalize routing function.
        if self.custom_routing_function == token_choice_with_bias:
            return RoutingMethodType.SigmoidRenorm
        return RoutingMethodType.Custom
```
**EN:** This method defines `routing_method_type`. It computes routing scores or expert-selection behavior for MoE execution. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `routing_method_type`。 它为 MoE 执行计算路由得分或专家选择行为。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 46-64 — method `CustomRoutingRouter._compute_routing`
```python
    def _compute_routing(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        indices_type: torch.dtype | None,
        *,
        input_ids: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """Compute routing using the custom routing function."""
        topk_weights, topk_ids = self.custom_routing_function(
            hidden_states=hidden_states,
            gating_output=router_logits,
            topk=self.top_k,
            renormalize=self.renormalize,
        )

        return topk_weights.to(torch.float32), topk_ids.to(
            torch.int32 if indices_type is None else indices_type
        )
```
**EN:** This method defines `_compute_routing`. Compute routing using the custom routing function. The main inputs are `hidden_states`, `router_logits`, `indices_type`, `input_ids`. Key calls include `self.custom_routing_function`, `topk_weights.to`, `topk_ids.to`. It writes or updates `topk_weights`, `topk_ids`.
**CN:** 该方法定义 `_compute_routing`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `router_logits`, `indices_type`, `input_ids`。 关键调用包括 `self.custom_routing_function`, `topk_weights.to`, `topk_ids.to`。 它会写入或更新 `topk_weights`, `topk_ids`。

## Key Concepts / 关键概念
- [EN] Moe router scoring and top-k expert selection / [CN] MoE 路由打分与 top-k 专家选择
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `CustomRoutingRouter` / [CN] 核心符号：`CustomRoutingRouter`

## Dependencies / 依赖关系
- **External**: `collections.abc`, `torch` / **外部依赖**: `collections.abc`, `torch`
- **Internal**: `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.base_router` / **内部依赖**: `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.base_router`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
