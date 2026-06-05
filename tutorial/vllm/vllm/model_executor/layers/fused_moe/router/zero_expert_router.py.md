# zero_expert_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/router/zero_expert_router.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE router scoring and top-k expert selection; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 路由打分与 top-k 专家选择；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-19 — imports and setup
```python
from collections.abc import Callable

import torch

from vllm.distributed.eplb.eplb_state import EplbLayerState
from vllm.model_executor.layers.fused_moe.config import (
    RoutingMethodType,
    get_routing_method_type,
)
from vllm.model_executor.layers.fused_moe.fused_moe import (
    zero_experts_compute_triton,
)
from vllm.model_executor.layers.fused_moe.router.base_router import BaseRouter
from vllm.model_executor.layers.fused_moe.router.fused_topk_bias_router import (
    fused_topk_bias,
)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 22-115 — class `ZeroExpertRouter`
```python
class ZeroExpertRouter(BaseRouter):
    """Router that handles zero expert computation as part of routing.

    Routes over all experts (real + zero) using full e_score_correction_bias.
    Computes zero expert identity contributions as a side effect during routing.
    Remaps zero expert IDs to real expert ID 0 (with weight 0) so downstream
    MoE computation can ignore them.
    """

    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        e_score_correction_bias: torch.Tensor,
# ... omitted for brevity ...
        self._zero_expert_output = None
        return output
```
**EN:** This class defines `ZeroExpertRouter`. It inherits from `BaseRouter`. Router that handles zero expert computation as part of routing. Important methods include `__init__`, `routing_method_type`, `_compute_routing`, `zero_expert_output`. Key calls include `super.__init__`, `get_routing_method_type`, `fused_topk_bias`, `zero_experts_compute_triton`, `super`, `topk_ids.clone`. It writes or updates `e_score_correction_bias`, `num_logical_experts`, `zero_expert_type`, `scoring_func`, `renormalize`, `routed_scaling_factor`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `ZeroExpertRouter`。 它继承自 `BaseRouter`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `routing_method_type`, `_compute_routing`, `zero_expert_output`。 关键调用包括 `super.__init__`, `get_routing_method_type`, `fused_topk_bias`, `zero_experts_compute_triton`, `super`, `topk_ids.clone`。 它会写入或更新 `e_score_correction_bias`, `num_logical_experts`, `zero_expert_type`, `scoring_func`, `renormalize`, `routed_scaling_factor`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 31-56 — method `ZeroExpertRouter.__init__`
```python
    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        e_score_correction_bias: torch.Tensor,
        num_logical_experts: int,
        zero_expert_type: str,
        scoring_func: str = "softmax",
        renormalize: bool = False,
        routed_scaling_factor: float = 1.0,
        eplb_state: EplbLayerState | None = None,
        indices_type_getter: Callable[[], torch.dtype | None] | None = None,
    ):
        super().__init__(
            top_k=top_k,
            global_num_experts=global_num_experts,
            eplb_state=eplb_state,
            indices_type_getter=indices_type_getter,
        )
        self.e_score_correction_bias = e_score_correction_bias
        self.num_logical_experts = num_logical_experts
        self.zero_expert_type = zero_expert_type
        self.scoring_func = scoring_func
        self.renormalize = renormalize
        self.routed_scaling_factor = routed_scaling_factor
        self._zero_expert_output: torch.Tensor | None = None
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `top_k`, `global_num_experts`, `e_score_correction_bias`, `num_logical_experts`, `zero_expert_type`, `scoring_func`. Key calls include `super.__init__`, `super`. It writes or updates `e_score_correction_bias`, `num_logical_experts`, `zero_expert_type`, `scoring_func`, `renormalize`, `routed_scaling_factor`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `top_k`, `global_num_experts`, `e_score_correction_bias`, `num_logical_experts`, `zero_expert_type`, `scoring_func`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `e_score_correction_bias`, `num_logical_experts`, `zero_expert_type`, `scoring_func`, `renormalize`, `routed_scaling_factor`。

### Lines 58-66 — method `ZeroExpertRouter.routing_method_type`
```python
    @property
    def routing_method_type(self) -> RoutingMethodType:
        return get_routing_method_type(
            scoring_func=self.scoring_func,
            top_k=self.top_k,
            renormalize=self.renormalize,
            num_expert_group=None,
            has_e_score_bias=True,
        )
```
**EN:** This method defines `routing_method_type`. It computes routing scores or expert-selection behavior for MoE execution. Key calls include `get_routing_method_type`.
**CN:** 该方法定义 `routing_method_type`。 它为 MoE 执行计算路由得分或专家选择行为。 关键调用包括 `get_routing_method_type`。

### Lines 68-108 — method `ZeroExpertRouter._compute_routing`
```python
    def _compute_routing(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        indices_type: torch.dtype | None,
        *,
        input_ids: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """Compute routing with full bias, compute zero expert output,
        mask zero expert IDs."""
        topk_weights, topk_ids = fused_topk_bias(
            hidden_states=hidden_states,
            gating_output=router_logits,
            e_score_correction_bias=self.e_score_correction_bias.data,
            topk=self.top_k,
            renormalize=self.renormalize,
            scoring_func=self.scoring_func,
            indices_type=indices_type,
        )

        if self.routed_scaling_factor != 1.0:
            topk_weights *= self.routed_scaling_factor

        # Compute zero expert output using pre-EPLB topk_ids/weights.
        # zero_experts_compute_triton modifies its inputs in-place, so
        # pass clones.
        self._zero_expert_output = zero_experts_compute_triton(
            expert_indices=topk_ids.clone(),
            expert_scales=topk_weights.clone(),
            num_experts=self.num_logical_experts,
            zero_expert_type=self.zero_expert_type,
            hidden_states=hidden_states,
        )

        # Mask zero expert entries: remap zero expert IDs to 0 with weight 0
        # so downstream MoE computation ignores them.
        zero_mask = topk_ids >= self.num_logical_experts
        topk_ids[zero_mask] = 0
        topk_weights[zero_mask] = 0.0

        return topk_weights, topk_ids
```
**EN:** This method defines `_compute_routing`. Compute routing with full bias, compute zero expert output, mask zero expert IDs. The main inputs are `hidden_states`, `router_logits`, `indices_type`, `input_ids`. Key calls include `fused_topk_bias`, `zero_experts_compute_triton`, `topk_ids.clone`, `topk_weights.clone`. It writes or updates `topk_weights`, `topk_ids`, `_zero_expert_output`, `zero_mask`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `_compute_routing`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `router_logits`, `indices_type`, `input_ids`。 关键调用包括 `fused_topk_bias`, `zero_experts_compute_triton`, `topk_ids.clone`, `topk_weights.clone`。 它会写入或更新 `topk_weights`, `topk_ids`, `_zero_expert_output`, `zero_mask`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 110-115 — method `ZeroExpertRouter.zero_expert_output`
```python
    @property
    def zero_expert_output(self) -> torch.Tensor | None:
        """Retrieve and clear the zero expert output."""
        output = self._zero_expert_output
        self._zero_expert_output = None
        return output
```
**EN:** This method defines `zero_expert_output`. Retrieve and clear the zero expert output. It writes or updates `output`, `_zero_expert_output`.
**CN:** 该方法定义 `zero_expert_output`。 该函数/方法的文档字符串直接说明了它的职责。 它会写入或更新 `output`, `_zero_expert_output`。

## Key Concepts / 关键概念
- [EN] Moe router scoring and top-k expert selection / [CN] MoE 路由打分与 top-k 专家选择
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `ZeroExpertRouter` / [CN] 核心符号：`ZeroExpertRouter`

## Dependencies / 依赖关系
- **External**: `collections.abc`, `torch` / **外部依赖**: `collections.abc`, `torch`
- **Internal**: `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe`, `vllm.model_executor.layers.fused_moe.router.base_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_bias_router` / **内部依赖**: `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe`, `vllm.model_executor.layers.fused_moe.router.base_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_bias_router`
- **Runtime traits**: Triton kernels, distributed collectives / **运行时特征**: Triton kernels, distributed collectives
