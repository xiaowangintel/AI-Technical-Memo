# fused_topk_bias_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/router/fused_topk_bias_router.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE router scoring and top-k expert selection; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 路由打分与 top-k 专家选择；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-17 — imports and setup
```python
import functools
from collections.abc import Callable

import torch
import torch.nn.functional as F

import vllm._custom_ops as ops
import vllm.envs as envs
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

### Lines 20-37 — function `vllm_topk_softmax`
```python
def vllm_topk_softmax(
    topk_weights: torch.Tensor,
    topk_indices: torch.Tensor,
    token_expert_indices: torch.Tensor,
    gating_output: torch.Tensor,
    renormalize: bool = False,
    e_score_correction_bias: torch.Tensor | None = None,
) -> tuple[torch.Tensor, ...]:
    ops.topk_softmax(
        topk_weights,
        topk_indices,
        token_expert_indices,
        gating_output,
        renormalize,
        e_score_correction_bias,
    )

    return topk_weights, topk_indices
```
**EN:** This function defines `vllm_topk_softmax`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `topk_weights`, `topk_indices`, `token_expert_indices`, `gating_output`, `renormalize`, `e_score_correction_bias`. Key calls include `ops.topk_softmax`.
**CN:** 该函数定义 `vllm_topk_softmax`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `topk_weights`, `topk_indices`, `token_expert_indices`, `gating_output`, `renormalize`, `e_score_correction_bias`。 关键调用包括 `ops.topk_softmax`。

### Lines 40-57 — function `vllm_topk_sigmoid`
```python
def vllm_topk_sigmoid(
    topk_weights: torch.Tensor,
    topk_indices: torch.Tensor,
    token_expert_indices: torch.Tensor,
    gating_output: torch.Tensor,
    renormalize: bool = False,
    e_score_correction_bias: torch.Tensor | None = None,
) -> tuple[torch.Tensor, ...]:
    ops.topk_sigmoid(
        topk_weights,
        topk_indices,
        token_expert_indices,
        gating_output,
        renormalize,
        e_score_correction_bias,
    )

    return topk_weights, topk_indices
```
**EN:** This function defines `vllm_topk_sigmoid`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `topk_weights`, `topk_indices`, `token_expert_indices`, `gating_output`, `renormalize`, `e_score_correction_bias`. Key calls include `ops.topk_sigmoid`.
**CN:** 该函数定义 `vllm_topk_sigmoid`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `topk_weights`, `topk_indices`, `token_expert_indices`, `gating_output`, `renormalize`, `e_score_correction_bias`。 关键调用包括 `ops.topk_sigmoid`。

### Lines 60-83 — function `vllm_topk_softplus_sqrt`
```python
def vllm_topk_softplus_sqrt(
    topk_weights: torch.Tensor,
    topk_indices: torch.Tensor,
    token_expert_indices: torch.Tensor,
    gating_output: torch.Tensor,
    renormalize: bool = False,
    e_score_correction_bias: torch.Tensor | None = None,
    input_tokens: torch.Tensor | None = None,
    hash_indices_table: torch.Tensor | None = None,
    routed_scaling_factor: float = 1.0,
) -> tuple[torch.Tensor, ...]:
    ops.topk_hash_softplus_sqrt(
        topk_weights,
        topk_indices,
        token_expert_indices,
        gating_output,
        renormalize,
        routed_scaling_factor,
        e_score_correction_bias,
        input_tokens,
        hash_indices_table,
    )

    return topk_weights, topk_indices
```
**EN:** This function defines `vllm_topk_softplus_sqrt`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `topk_weights`, `topk_indices`, `token_expert_indices`, `gating_output`, `renormalize`, `e_score_correction_bias`. Key calls include `ops.topk_hash_softplus_sqrt`.
**CN:** 该函数定义 `vllm_topk_softplus_sqrt`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `topk_weights`, `topk_indices`, `token_expert_indices`, `gating_output`, `renormalize`, `e_score_correction_bias`。 关键调用包括 `ops.topk_hash_softplus_sqrt`。

### Lines 86-96 — function `_aiter_get_num_expert_group`
```python
@functools.lru_cache(maxsize=8)
def _aiter_get_num_expert_group(num_experts: int) -> int:
    _AITER_MAX_EXPERTS_PER_GROUP = 32
    g = max(1, -(-num_experts // _AITER_MAX_EXPERTS_PER_GROUP))
    while num_experts % g != 0:
        g += 1
    assert num_experts % g == 0, f"{num_experts=} not divisible by {g=}"
    assert num_experts // g <= _AITER_MAX_EXPERTS_PER_GROUP, (
        f"group size {num_experts // g} exceeds limit {_AITER_MAX_EXPERTS_PER_GROUP}"
    )
    return g
```
**EN:** This function defines `_aiter_get_num_expert_group`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `num_experts`. Key calls include `functools.lru_cache`, `max`. It writes or updates `_AITER_MAX_EXPERTS_PER_GROUP`, `g`. The body uses loops to cover different runtime cases.
**CN:** 该函数定义 `_aiter_get_num_expert_group`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `num_experts`。 关键调用包括 `functools.lru_cache`, `max`。 它会写入或更新 `_AITER_MAX_EXPERTS_PER_GROUP`, `g`。 函数体通过循环来覆盖不同的运行时场景。

### Lines 99-228 — function `fused_topk_bias`
```python
def fused_topk_bias(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    scoring_func: str,
    e_score_correction_bias: torch.Tensor,
    topk: int,
    renormalize: bool,
    indices_type: torch.dtype | None = None,
    input_tokens: torch.Tensor | None = None,
    hash_indices_table: torch.Tensor | None = None,
    routed_scaling_factor: float = 1.0,
):
    if not rocm_aiter_ops.is_fused_moe_enabled():
        assert hidden_states.size(0) == gating_output.size(0), (
            "Number of tokens mismatch"
        )

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
            topk_weights, topk_ids = vllm_topk_softmax(
                topk_weights,
                topk_ids,
                token_expert_indices,
                gating_output,
                renormalize,
                e_score_correction_bias,
            )
            if routed_scaling_factor != 1.0:
                topk_weights *= routed_scaling_factor
            return topk_weights, topk_ids
# ... omitted for brevity ...
        torch.int32 if indices_type is None else indices_type
    )
```
**EN:** This function defines `fused_topk_bias`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `hidden_states`, `gating_output`, `scoring_func`, `e_score_correction_bias`, `topk`, `renormalize`. Key calls include `scores.gather`, `topk_weights.to`, `rocm_aiter_ops.is_fused_moe_enabled`, `hidden_states.size`, `torch.empty`, `gating_output.softmax`. It writes or updates `n_routed_experts`, `topk_weights`, `M`, `_`, `topk_ids`, `token_expert_indices`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `fused_topk_bias`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `hidden_states`, `gating_output`, `scoring_func`, `e_score_correction_bias`, `topk`, `renormalize`。 关键调用包括 `scores.gather`, `topk_weights.to`, `rocm_aiter_ops.is_fused_moe_enabled`, `hidden_states.size`, `torch.empty`, `gating_output.softmax`。 它会写入或更新 `n_routed_experts`, `topk_weights`, `M`, `_`, `topk_ids`, `token_expert_indices`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 231-294 — class `FusedTopKBiasRouter`
```python
class FusedTopKBiasRouter(BaseRouter):
    """Router using fused top-k with e_score_correction_bias."""

    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        e_score_correction_bias: torch.Tensor | None = None,
        renormalize: bool = True,
        routed_scaling_factor: float = 1.0,
        eplb_state: EplbLayerState | None = None,
        indices_type_getter: Callable[[], torch.dtype | None] | None = None,
        *,
        scoring_func: str = "sigmoid",
# ... omitted for brevity ...

        return topk_weights, topk_ids
```
**EN:** This class defines `FusedTopKBiasRouter`. It inherits from `BaseRouter`. Router using fused top-k with e_score_correction_bias. Important methods include `__init__`, `routing_method_type`. Key calls include `super.__init__`, `get_routing_method_type`, `fused_topk_bias`, `super`. It writes or updates `e_score_correction_bias`, `renormalize`, `scoring_func`, `routed_scaling_factor`, `_hash_indices_table`, `topk_weights`.
**CN:** 该类定义了 `FusedTopKBiasRouter`。 它继承自 `BaseRouter`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `routing_method_type`。 关键调用包括 `super.__init__`, `get_routing_method_type`, `fused_topk_bias`, `super`。 它会写入或更新 `e_score_correction_bias`, `renormalize`, `scoring_func`, `routed_scaling_factor`, `_hash_indices_table`, `topk_weights`。

### Lines 234-258 — method `FusedTopKBiasRouter.__init__`
```python
    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        e_score_correction_bias: torch.Tensor | None = None,
        renormalize: bool = True,
        routed_scaling_factor: float = 1.0,
        eplb_state: EplbLayerState | None = None,
        indices_type_getter: Callable[[], torch.dtype | None] | None = None,
        *,
        scoring_func: str = "sigmoid",
        hash_indices_table: torch.Tensor | None = None,
    ):
        super().__init__(
            top_k=top_k,
            global_num_experts=global_num_experts,
            eplb_state=eplb_state,
            indices_type_getter=indices_type_getter,
        )
        self.e_score_correction_bias = e_score_correction_bias
        self.renormalize = renormalize
        self.scoring_func = scoring_func
        self.routed_scaling_factor = routed_scaling_factor
        self.scoring_func = scoring_func
        self._hash_indices_table = hash_indices_table
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `top_k`, `global_num_experts`, `e_score_correction_bias`, `renormalize`, `routed_scaling_factor`, `eplb_state`. Key calls include `super.__init__`, `super`. It writes or updates `e_score_correction_bias`, `renormalize`, `scoring_func`, `routed_scaling_factor`, `_hash_indices_table`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `top_k`, `global_num_experts`, `e_score_correction_bias`, `renormalize`, `routed_scaling_factor`, `eplb_state`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `e_score_correction_bias`, `renormalize`, `scoring_func`, `routed_scaling_factor`, `_hash_indices_table`。

### Lines 260-268 — method `FusedTopKBiasRouter.routing_method_type`
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

## Key Concepts / 关键概念
- [EN] Moe router scoring and top-k expert selection / [CN] MoE 路由打分与 top-k 专家选择
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `vllm_topk_softmax`, `vllm_topk_sigmoid`, `vllm_topk_softplus_sqrt`, `_aiter_get_num_expert_group` / [CN] 核心符号：`vllm_topk_softmax`, `vllm_topk_sigmoid`, `vllm_topk_softplus_sqrt`, `_aiter_get_num_expert_group`

## Dependencies / 依赖关系
- **External**: `functools`, `collections.abc`, `torch`, `torch.nn.functional` / **外部依赖**: `functools`, `collections.abc`, `torch`, `torch.nn.functional`
- **Internal**: `vllm._custom_ops`, `vllm.envs`, `vllm._aiter_ops`, `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.base_router` / **内部依赖**: `vllm._custom_ops`, `vllm.envs`, `vllm._aiter_ops`, `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.base_router`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
