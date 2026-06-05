# fused_topk_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/router/fused_topk_router.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE router scoring and top-k expert selection; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 路由打分与 top-k 专家选择；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-14 — imports and setup
```python
from collections.abc import Callable

import torch

import vllm._custom_ops as ops
from vllm._aiter_ops import rocm_aiter_ops
from vllm.distributed.eplb.eplb_state import EplbLayerState
from vllm.model_executor.layers.fused_moe.config import (
    RoutingMethodType,
    get_routing_method_type,
)
from vllm.model_executor.layers.fused_moe.router.base_router import BaseRouter
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 17-32 — function `vllm_topk_softmax`
```python
def vllm_topk_softmax(
    topk_weights: torch.Tensor,
    topk_indices: torch.Tensor,
    token_expert_indices: torch.Tensor,
    gating_output: torch.Tensor,
    renormalize: bool = False,
) -> tuple[torch.Tensor, ...]:
    ops.topk_softmax(
        topk_weights,
        topk_indices,
        token_expert_indices,
        gating_output,
        renormalize,
    )

    return topk_weights, topk_indices
```
**EN:** This function defines `vllm_topk_softmax`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `topk_weights`, `topk_indices`, `token_expert_indices`, `gating_output`, `renormalize`. Key calls include `ops.topk_softmax`.
**CN:** 该函数定义 `vllm_topk_softmax`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `topk_weights`, `topk_indices`, `token_expert_indices`, `gating_output`, `renormalize`。 关键调用包括 `ops.topk_softmax`。

### Lines 35-50 — function `vllm_topk_sigmoid`
```python
def vllm_topk_sigmoid(
    topk_weights: torch.Tensor,
    topk_indices: torch.Tensor,
    token_expert_indices: torch.Tensor,
    gating_output: torch.Tensor,
    renormalize: bool = False,
) -> tuple[torch.Tensor, ...]:
    ops.topk_sigmoid(
        topk_weights,
        topk_indices,
        token_expert_indices,
        gating_output,
        renormalize,
    )

    return topk_weights, topk_indices
```
**EN:** This function defines `vllm_topk_sigmoid`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `topk_weights`, `topk_indices`, `token_expert_indices`, `gating_output`, `renormalize`. Key calls include `ops.topk_sigmoid`.
**CN:** 该函数定义 `vllm_topk_sigmoid`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `topk_weights`, `topk_indices`, `token_expert_indices`, `gating_output`, `renormalize`。 关键调用包括 `ops.topk_sigmoid`。

### Lines 53-58 — function `dispatch_topk_softmax_func`
```python
def dispatch_topk_softmax_func(
    use_rocm_aiter: bool = False,
) -> Callable[..., tuple[torch.Tensor, ...]]:
    if use_rocm_aiter:
        return rocm_aiter_ops.topk_softmax
    return vllm_topk_softmax
```
**EN:** This function defines `dispatch_topk_softmax_func`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `use_rocm_aiter`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `dispatch_topk_softmax_func`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `use_rocm_aiter`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 61-66 — function `dispatch_topk_sigmoid_func`
```python
def dispatch_topk_sigmoid_func(
    use_rocm_aiter: bool = False,
) -> Callable[..., tuple[torch.Tensor, ...]]:
    if use_rocm_aiter:
        return rocm_aiter_ops.topk_sigmoid
    return vllm_topk_sigmoid
```
**EN:** This function defines `dispatch_topk_sigmoid_func`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `use_rocm_aiter`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `dispatch_topk_sigmoid_func`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `use_rocm_aiter`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 69-113 — function `fused_topk`
```python
def fused_topk(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    indices_type: torch.dtype | None = None,
    scoring_func: str = "softmax",
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    assert hidden_states.size(0) == gating_output.size(0), "Number of tokens mismatch"

    M, _ = hidden_states.size()

    topk_weights = torch.empty(
        M, topk, dtype=torch.float32, device=hidden_states.device
    )
    topk_ids = torch.empty(
        M,
        topk,
        dtype=torch.int32 if indices_type is None else indices_type,
        device=hidden_states.device,
    )
    token_expert_indices = torch.empty(
        M, topk, dtype=torch.int32, device=hidden_states.device
    )

    if scoring_func == "softmax":
        topk_func = dispatch_topk_softmax_func(
            use_rocm_aiter=rocm_aiter_ops.is_fused_moe_enabled()
        )
        topk_weights, topk_ids = topk_func(
            topk_weights, topk_ids, token_expert_indices, gating_output, renormalize
        )

        return topk_weights, topk_ids, token_expert_indices
    elif scoring_func == "sigmoid":
        topk_func = dispatch_topk_sigmoid_func(
            use_rocm_aiter=rocm_aiter_ops.is_fused_moe_enabled()
        )
        topk_weights, topk_ids = topk_func(
            topk_weights, topk_ids, token_expert_indices, gating_output, renormalize
        )

        return topk_weights, topk_ids, token_expert_indices
    else:
        raise ValueError(f"Unsupported scoring function: {scoring_func}")
```
**EN:** This function defines `fused_topk`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `hidden_states`, `gating_output`, `topk`, `renormalize`, `indices_type`, `scoring_func`. Key calls include `hidden_states.size`, `torch.empty`, `gating_output.size`, `dispatch_topk_softmax_func`, `topk_func`, `dispatch_topk_sigmoid_func`. It writes or updates `M`, `_`, `topk_weights`, `topk_ids`, `token_expert_indices`, `topk_func`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `fused_topk`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `hidden_states`, `gating_output`, `topk`, `renormalize`, `indices_type`, `scoring_func`。 关键调用包括 `hidden_states.size`, `torch.empty`, `gating_output.size`, `dispatch_topk_softmax_func`, `topk_func`, `dispatch_topk_sigmoid_func`。 它会写入或更新 `M`, `_`, `topk_weights`, `topk_ids`, `token_expert_indices`, `topk_func`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 116-165 — class `FusedTopKRouter`
```python
class FusedTopKRouter(BaseRouter):
    """Default router using standard fused top-k routing."""

    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        scoring_func: str = "softmax",
        renormalize: bool = True,
        eplb_state: EplbLayerState | None = None,
        indices_type_getter: Callable[[], torch.dtype | None] | None = None,
    ):
        super().__init__(
            top_k=top_k,
# ... omitted for brevity ...

        return topk_weights, topk_ids
```
**EN:** This class defines `FusedTopKRouter`. It inherits from `BaseRouter`. Default router using standard fused top-k routing. Important methods include `__init__`, `routing_method_type`, `_compute_routing`. Key calls include `super.__init__`, `get_routing_method_type`, `fused_topk`, `super`. It writes or updates `renormalize`, `scoring_func`, `topk_weights`, `topk_ids`, `token_expert_indices`.
**CN:** 该类定义了 `FusedTopKRouter`。 它继承自 `BaseRouter`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `routing_method_type`, `_compute_routing`。 关键调用包括 `super.__init__`, `get_routing_method_type`, `fused_topk`, `super`。 它会写入或更新 `renormalize`, `scoring_func`, `topk_weights`, `topk_ids`, `token_expert_indices`。

### Lines 119-135 — method `FusedTopKRouter.__init__`
```python
    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        scoring_func: str = "softmax",
        renormalize: bool = True,
        eplb_state: EplbLayerState | None = None,
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
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `top_k`, `global_num_experts`, `scoring_func`, `renormalize`, `eplb_state`, `indices_type_getter`. Key calls include `super.__init__`, `super`. It writes or updates `renormalize`, `scoring_func`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `top_k`, `global_num_experts`, `scoring_func`, `renormalize`, `eplb_state`, `indices_type_getter`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `renormalize`, `scoring_func`。

### Lines 137-145 — method `FusedTopKRouter.routing_method_type`
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

### Lines 147-165 — method `FusedTopKRouter._compute_routing`
```python
    def _compute_routing(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        indices_type: torch.dtype | None,
        *,
        input_ids: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """Compute routing using standard fused top-k."""
        topk_weights, topk_ids, token_expert_indices = fused_topk(
            hidden_states=hidden_states,
            gating_output=router_logits,
            topk=self.top_k,
            renormalize=self.renormalize,
            indices_type=indices_type,
            scoring_func=self.scoring_func,
        )

        return topk_weights, topk_ids
```
**EN:** This method defines `_compute_routing`. Compute routing using standard fused top-k. The main inputs are `hidden_states`, `router_logits`, `indices_type`, `input_ids`. Key calls include `fused_topk`. It writes or updates `topk_weights`, `topk_ids`, `token_expert_indices`.
**CN:** 该方法定义 `_compute_routing`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `router_logits`, `indices_type`, `input_ids`。 关键调用包括 `fused_topk`。 它会写入或更新 `topk_weights`, `topk_ids`, `token_expert_indices`。

## Key Concepts / 关键概念
- [EN] Moe router scoring and top-k expert selection / [CN] MoE 路由打分与 top-k 专家选择
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `vllm_topk_softmax`, `vllm_topk_sigmoid`, `dispatch_topk_softmax_func`, `dispatch_topk_sigmoid_func` / [CN] 核心符号：`vllm_topk_softmax`, `vllm_topk_sigmoid`, `dispatch_topk_softmax_func`, `dispatch_topk_sigmoid_func`

## Dependencies / 依赖关系
- **External**: `collections.abc`, `torch` / **外部依赖**: `collections.abc`, `torch`
- **Internal**: `vllm._custom_ops`, `vllm._aiter_ops`, `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.base_router` / **内部依赖**: `vllm._custom_ops`, `vllm._aiter_ops`, `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.base_router`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
