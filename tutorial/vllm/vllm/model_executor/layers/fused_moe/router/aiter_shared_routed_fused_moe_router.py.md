# aiter_shared_routed_fused_moe_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/router/aiter_shared_routed_fused_moe_router.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE router scoring and top-k expert selection; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 路由打分与 top-k 专家选择；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-16 — imports and setup
```python
from collections.abc import Callable

import torch

from vllm._aiter_ops import rocm_aiter_ops
from vllm.distributed.eplb.eplb_state import EplbLayerState
from vllm.model_executor.layers.fused_moe.config import (
    RoutingMethodType,
    get_routing_method_type,
)
from vllm.model_executor.layers.fused_moe.router.base_router import BaseRouter
from vllm.model_executor.layers.fused_moe.router.fused_topk_router import (
    dispatch_topk_softmax_func,
)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 19-141 — class `AiterSharedRoutedFusedMoERouter`
```python
class AiterSharedRoutedFusedMoERouter(BaseRouter):
    """
    ROCm AITER router for models with fused shared experts (e.g. Qwen3-MoE).

    When the AITER topk_softmax kernel supports sigmoid fusion, the routing
    softmax and shared-expert sigmoid are computed in a single kernel launch.
    Otherwise the shared-expert weights are injected into the pre-allocated
    AITER buffer via a fallback path.

    Only instantiated when rocm_aiter fused-MoE is active and
    num_fused_shared_experts > 0.
    """

    def __init__(
# ... omitted for brevity ...

        return topk_weights, topk_ids
```
**EN:** This class defines `AiterSharedRoutedFusedMoERouter`. It inherits from `BaseRouter`. ROCm AITER router for models with fused shared experts (e.g. Important methods include `__init__`, `routing_method_type`, `_compute_routing`. Key calls include `super.__init__`, `get_routing_method_type`, `hidden_states.size`, `torch.empty`, `rocm_aiter_ops.fuse_sigmoid_in_kernel`, `dispatch_topk_softmax_func`. It writes or updates `renormalize`, `scoring_func`, `num_fused_shared_experts`, `M`, `topk`, `num_fse`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `AiterSharedRoutedFusedMoERouter`。 它继承自 `BaseRouter`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `routing_method_type`, `_compute_routing`。 关键调用包括 `super.__init__`, `get_routing_method_type`, `hidden_states.size`, `torch.empty`, `rocm_aiter_ops.fuse_sigmoid_in_kernel`, `dispatch_topk_softmax_func`。 它会写入或更新 `renormalize`, `scoring_func`, `num_fused_shared_experts`, `M`, `topk`, `num_fse`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 32-50 — method `AiterSharedRoutedFusedMoERouter.__init__`
```python
    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        num_fused_shared_experts: int,
        eplb_state: EplbLayerState | None = None,
        scoring_func: str = "softmax",
        renormalize: bool = True,
        indices_type_getter: Callable[[], torch.dtype | None] | None = None,
    ):
        super().__init__(
            top_k=top_k,
            global_num_experts=global_num_experts,
            eplb_state=eplb_state,
            indices_type_getter=indices_type_getter,
        )
        self.renormalize = renormalize
        self.scoring_func = scoring_func
        self.num_fused_shared_experts = num_fused_shared_experts
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `top_k`, `global_num_experts`, `num_fused_shared_experts`, `eplb_state`, `scoring_func`, `renormalize`. Key calls include `super.__init__`, `super`. It writes or updates `renormalize`, `scoring_func`, `num_fused_shared_experts`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `top_k`, `global_num_experts`, `num_fused_shared_experts`, `eplb_state`, `scoring_func`, `renormalize`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `renormalize`, `scoring_func`, `num_fused_shared_experts`。

### Lines 52-60 — method `AiterSharedRoutedFusedMoERouter.routing_method_type`
```python
    @property
    def routing_method_type(self) -> RoutingMethodType:
        return get_routing_method_type(
            scoring_func=self.scoring_func,
            top_k=self.top_k,
            renormalize=self.renormalize,
            num_expert_group=None,
            has_e_score_bias=False,
        )
```
**EN:** This method defines `routing_method_type`. It computes routing scores or expert-selection behavior for MoE execution. Key calls include `get_routing_method_type`.
**CN:** 该方法定义 `routing_method_type`。 它为 MoE 执行计算路由得分或专家选择行为。 关键调用包括 `get_routing_method_type`。

### Lines 62-141 — method `AiterSharedRoutedFusedMoERouter._compute_routing`
```python
    def _compute_routing(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        indices_type: torch.dtype | None,
        *,
        input_ids: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        assert hidden_states.size(0) == router_logits.size(0), (
            "Number of tokens mismatch"
        )

        from vllm.model_executor.layers.fused_moe.rocm_aiter_fused_moe import (
            aiter_topK_meta_data,
        )

        M = hidden_states.size(0)
        topk = self.top_k
        num_fse = self.num_fused_shared_experts

        token_expert_indices = torch.empty(
            M, topk, dtype=torch.int32, device=hidden_states.device
        )

        if rocm_aiter_ops.fuse_sigmoid_in_kernel(aiter_topK_meta_data):
            total_topk_weights, total_topk_ids = aiter_topK_meta_data  # type: ignore[misc]
            total_topk_weights_slice = total_topk_weights[:M]
            topk_ids_slice = total_topk_ids[:M, :topk]

            topk_func = dispatch_topk_softmax_func(use_rocm_aiter=True)
            topk_func(
                total_topk_weights_slice,
                topk_ids_slice,
                token_expert_indices,
                router_logits,
                self.renormalize,
                num_fse,
                "sigmoid",
            )
            return total_topk_weights_slice, total_topk_ids[:M]

        routing_logits = router_logits[:, :-num_fse]
        shared_logits = router_logits[:, -num_fse:]

# ... omitted for brevity ...

        return topk_weights, topk_ids
```
**EN:** This method defines `_compute_routing`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `hidden_states`, `router_logits`, `indices_type`, `input_ids`. Key calls include `hidden_states.size`, `torch.empty`, `rocm_aiter_ops.fuse_sigmoid_in_kernel`, `dispatch_topk_softmax_func`, `topk_func`, `router_logits.size`. It writes or updates `M`, `topk`, `num_fse`, `token_expert_indices`, `routing_logits`, `shared_logits`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `_compute_routing`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `hidden_states`, `router_logits`, `indices_type`, `input_ids`。 关键调用包括 `hidden_states.size`, `torch.empty`, `rocm_aiter_ops.fuse_sigmoid_in_kernel`, `dispatch_topk_softmax_func`, `topk_func`, `router_logits.size`。 它会写入或更新 `M`, `topk`, `num_fse`, `token_expert_indices`, `routing_logits`, `shared_logits`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Moe router scoring and top-k expert selection / [CN] MoE 路由打分与 top-k 专家选择
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `AiterSharedRoutedFusedMoERouter` / [CN] 核心符号：`AiterSharedRoutedFusedMoERouter`

## Dependencies / 依赖关系
- **External**: `collections.abc`, `torch` / **外部依赖**: `collections.abc`, `torch`
- **Internal**: `vllm._aiter_ops`, `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.base_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_router` / **内部依赖**: `vllm._aiter_ops`, `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.base_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_router`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
