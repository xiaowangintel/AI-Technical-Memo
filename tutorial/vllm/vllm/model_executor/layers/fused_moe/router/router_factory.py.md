# router_factory.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/router/router_factory.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE router scoring and top-k expert selection; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 路由打分与 top-k 专家选择；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-36 — imports and setup
```python
from collections.abc import Callable

import torch

import vllm.envs as envs
from vllm._aiter_ops import rocm_aiter_ops
from vllm.distributed.eplb.eplb_state import EplbLayerState
from vllm.model_executor.layers.fused_moe.config import (
    RoutingMethodType,
)
from vllm.model_executor.layers.fused_moe.router.aiter_shared_routed_fused_moe_router import (  # noqa: E501
    AiterSharedRoutedFusedMoERouter,
)
from vllm.model_executor.layers.fused_moe.router.custom_routing_router import (
    CustomRoutingRouter,
)
from vllm.model_executor.layers.fused_moe.router.fused_moe_router import (
    FusedMoERouter,
)
from vllm.model_executor.layers.fused_moe.router.fused_topk_bias_router import (
    FusedTopKBiasRouter,
)
from vllm.model_executor.layers.fused_moe.router.fused_topk_router import (
    FusedTopKRouter,
)
from vllm.model_executor.layers.fused_moe.router.grouped_topk_router import (
    GroupedTopKRouter,
)
from vllm.model_executor.layers.fused_moe.router.routing_simulator_router import (
    RoutingSimulatorRouter,
)
from vllm.model_executor.layers.fused_moe.router.zero_expert_router import (
    ZeroExpertRouter,
)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 39-222 — function `create_fused_moe_router`
```python
def create_fused_moe_router(
    # common parameters
    top_k: int,
    global_num_experts: int,
    renormalize: bool = True,
    indices_type_getter: Callable[[], torch.dtype | None] | None = None,
    # grouped topk parameters
    use_grouped_topk: bool = False,
    num_expert_group: int | None = None,
    topk_group: int | None = None,
    scoring_func: str = "softmax",
    num_fused_shared_experts: int = 0,
    # grouped topk + fused topk bias parameters
    routed_scaling_factor: float = 1.0,
    e_score_correction_bias: torch.Tensor | None = None,
    # custom routing parameters
    custom_routing_function: Callable | None = None,
    # eplb parameters
    eplb_state: EplbLayerState | None = None,
    # zero expert parameters
    zero_expert_type: str | None = None,
    num_logical_experts: int | None = None,
    hash_indices_table: torch.Tensor | None = None,
) -> FusedMoERouter:
    """
    Factory function to create the appropriate FusedMoERouter subclass based on
    the provided parameters.

    The selection logic follows this priority order:
    1. RoutingSimulatorRouter - if VLLM_MOE_ROUTING_SIMULATION_STRATEGY env var is set
    2. ZeroExpertRouter - if zero_expert_type is not None
    3. GroupedTopKRouter - if use_grouped_topk is True
    4. CustomRoutingRouter - if custom_routing_function is not None
    5. FusedTopKBiasRouter - if e_score_correction_bias is not None
    6. AiterSharedRoutedFusedMoERouter - if num_fused_shared_experts > 0
    7. FusedTopKRouter - default fallback

    Common arguments:
        top_k: Number of experts to select per token
        global_num_experts: Total number of experts in the model
        renormalize: Whether to renormalize the routing weights
        indices_type_getter: Function to get the desired indices dtype
        routing_method_type: Optional explicit routing method type

# ... omitted for brevity ...
        indices_type_getter=indices_type_getter,
    )
```
**EN:** This function defines `create_fused_moe_router`. Factory function to create the appropriate FusedMoERouter subclass based on the provided parameters. The main inputs are `top_k`, `global_num_experts`, `renormalize`, `indices_type_getter`, `use_grouped_topk`, `num_expert_group`. Key calls include `FusedTopKRouter`, `RoutingSimulatorRouter`, `ZeroExpertRouter`, `GroupedTopKRouter`, `CustomRoutingRouter`, `FusedTopKBiasRouter`. It writes or updates `routing_strategy`, `grouped_topk_router`, `use_grouped_topk`, `num_expert_group`, `topk_group`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `create_fused_moe_router`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `top_k`, `global_num_experts`, `renormalize`, `indices_type_getter`, `use_grouped_topk`, `num_expert_group`。 关键调用包括 `FusedTopKRouter`, `RoutingSimulatorRouter`, `ZeroExpertRouter`, `GroupedTopKRouter`, `CustomRoutingRouter`, `FusedTopKBiasRouter`。 它会写入或更新 `routing_strategy`, `grouped_topk_router`, `use_grouped_topk`, `num_expert_group`, `topk_group`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Moe router scoring and top-k expert selection / [CN] MoE 路由打分与 top-k 专家选择
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `create_fused_moe_router` / [CN] 核心符号：`create_fused_moe_router`

## Dependencies / 依赖关系
- **External**: `collections.abc`, `torch` / **外部依赖**: `collections.abc`, `torch`
- **Internal**: `vllm.envs`, `vllm._aiter_ops`, `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.aiter_shared_routed_fused_moe_router`, `vllm.model_executor.layers.fused_moe.router.custom_routing_router`, `vllm.model_executor.layers.fused_moe.router.fused_moe_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_bias_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_router`, `vllm.model_executor.layers.fused_moe.router.grouped_topk_router` / **内部依赖**: `vllm.envs`, `vllm._aiter_ops`, `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.aiter_shared_routed_fused_moe_router`, `vllm.model_executor.layers.fused_moe.router.custom_routing_router`, `vllm.model_executor.layers.fused_moe.router.fused_moe_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_bias_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_router`, `vllm.model_executor.layers.fused_moe.router.grouped_topk_router`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
