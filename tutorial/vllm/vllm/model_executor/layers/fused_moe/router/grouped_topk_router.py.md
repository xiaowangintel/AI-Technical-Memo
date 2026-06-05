# grouped_topk_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/router/grouped_topk_router.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE router scoring and top-k expert selection; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 路由打分与 top-k 专家选择；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-26 — imports and setup
```python
from collections.abc import Callable
from functools import partial

import torch

from vllm import _custom_ops as ops
from vllm import envs as envs
from vllm._aiter_ops import rocm_aiter_ops
from vllm.distributed.eplb.eplb_state import EplbLayerState
from vllm.model_executor.custom_op import CustomOp
from vllm.model_executor.layers.fused_moe.config import (
    RoutingMethodType,
    get_routing_method_type,
)
from vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe import (
    rocm_aiter_grouped_topk,
)
from vllm.model_executor.layers.fused_moe.router.base_router import BaseRouter
from vllm.model_executor.layers.fused_moe.router.fused_topk_bias_router import (
    fused_topk_bias,
)
from vllm.model_executor.layers.fused_moe.router.fused_topk_router import fused_topk
from vllm.model_executor.utils import maybe_disable_graph_partition
from vllm.platforms import current_platform
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 29-72 — function `fused_grouped_topk`
```python
def fused_grouped_topk(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    e_score_correction_bias: torch.Tensor,
    num_expert_group: int = 0,
    topk_group: int = 0,
    scoring_func: str = "softmax",
    routed_scaling_factor: float = 1.0,
) -> tuple[torch.Tensor, torch.Tensor]:
    assert hidden_states.size(0) == gating_output.size(0), "Number of tokens mismatch"

    if scoring_func == "sigmoid":
        # Fully fused kernel path for sigmoid
        topk_values, topk_indices = ops.grouped_topk(
            gating_output,  # raw logits
            num_expert_group,
            topk_group,
            topk,
            renormalize,
            routed_scaling_factor,
            e_score_correction_bias,
            1,  # scoring_func=1 for sigmoid
        )
    elif scoring_func == "softmax":
        # Apply softmax in Python, then use fused kernel
        # TODO: Add support for softmax in kernel
        scores = torch.softmax(gating_output, dim=-1)
        topk_values, topk_indices = ops.grouped_topk(
            scores,  # pre-computed scores
            num_expert_group,
            topk_group,
            topk,
            renormalize,
            routed_scaling_factor,
            e_score_correction_bias,
            0,  # scoring_func=0 (no activation, scores already computed)
        )
    else:
        raise ValueError(f"Unsupported scoring function: {scoring_func}")

    # Fused kernel outputs float32 values and int32 indices directly
    return topk_values, topk_indices
```
**EN:** This function defines `fused_grouped_topk`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `hidden_states`, `gating_output`, `topk`, `renormalize`, `e_score_correction_bias`, `num_expert_group`. Key calls include `hidden_states.size`, `gating_output.size`, `ops.grouped_topk`, `torch.softmax`, `ValueError`. It writes or updates `topk_values`, `topk_indices`, `scores`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `fused_grouped_topk`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `hidden_states`, `gating_output`, `topk`, `renormalize`, `e_score_correction_bias`, `num_expert_group`。 关键调用包括 `hidden_states.size`, `gating_output.size`, `ops.grouped_topk`, `torch.softmax`, `ValueError`。 它会写入或更新 `topk_values`, `topk_indices`, `scores`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 76-162 — function `grouped_topk`
```python
@torch.compile(
    dynamic=True,
    backend=current_platform.simple_compile_backend,
    options=maybe_disable_graph_partition(current_platform.simple_compile_backend),
)
def grouped_topk(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    num_expert_group: int = 0,
    topk_group: int = 0,
    scoring_func: str = "softmax",
    routed_scaling_factor: float = 1.0,
    e_score_correction_bias: torch.Tensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    if (
        envs.VLLM_USE_FUSED_MOE_GROUPED_TOPK
        and current_platform.is_cuda()
        and num_expert_group <= 32
        and topk <= 32
        and e_score_correction_bias is not None
    ):
        return fused_grouped_topk(
            hidden_states=hidden_states,
            gating_output=gating_output,
            topk=topk,
            renormalize=renormalize,
            e_score_correction_bias=e_score_correction_bias,
            num_expert_group=num_expert_group,
            topk_group=topk_group,
            scoring_func=scoring_func,
            routed_scaling_factor=routed_scaling_factor,
        )

    assert hidden_states.size(0) == gating_output.size(0), "Number of tokens mismatch"

    if scoring_func == "softmax":
        scores = torch.softmax(gating_output, dim=-1)
    elif scoring_func == "sigmoid":
        scores = gating_output.sigmoid()
    else:
        raise ValueError(f"Unsupported scoring function: {scoring_func}")

# ... omitted for brevity ...
        topk_weights = topk_weights * routed_scaling_factor
    return topk_weights.to(torch.float32), topk_ids.to(torch.int32)
```
**EN:** This function defines `grouped_topk`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `hidden_states`, `gating_output`, `topk`, `renormalize`, `num_expert_group`, `topk_group`. Key calls include `torch.compile`, `scores.size`, `torch.zeros_like`, `group_mask.scatter_`, `group_mask.unsqueeze.expand.reshape`, `scores.masked_fill`. It writes or updates `num_token`, `use_sorted`, `group_idx`, `group_mask`, `score_mask`, `tmp_scores`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `grouped_topk`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `hidden_states`, `gating_output`, `topk`, `renormalize`, `num_expert_group`, `topk_group`。 关键调用包括 `torch.compile`, `scores.size`, `torch.zeros_like`, `group_mask.scatter_`, `group_mask.unsqueeze.expand.reshape`, `scores.masked_fill`。 它会写入或更新 `num_token`, `use_sorted`, `group_idx`, `group_mask`, `score_mask`, `tmp_scores`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 166-244 — class `GroupedTopk`
```python
@CustomOp.register("grouped_topk")
class GroupedTopk(CustomOp):
    """GroupedTopk used by the Deepseek-V2 and Deepseek-V3 model."""

    # --8<-- [end:grouped_topk]

    def __init__(
        self,
        topk: int,
        renormalize: bool,
        num_expert_group: int = 0,
        topk_group: int = 0,
        scoring_func: str = "softmax",
        routed_scaling_factor: float = 1.0,
# ... omitted for brevity ...
                hidden_states, gating_output, e_score_correction_bias
            )
```
**EN:** This class defines `GroupedTopk`. It inherits from `CustomOp`. GroupedTopk used by the Deepseek-V2 and Deepseek-V3 model. Important methods include `__init__`, `forward_native`, `forward_cuda`, `forward_hip`. Key calls include `CustomOp.register`, `super.__init__`, `self.native_impl`, `self.forward_native`, `rocm_aiter_ops.is_fused_moe_enabled`, `rocm_aiter_grouped_topk`. It writes or updates `native_impl`, `topk`, `renormalize`, `num_expert_group`, `topk_group`, `scoring_func`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `GroupedTopk`。 它继承自 `CustomOp`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `forward_native`, `forward_cuda`, `forward_hip`。 关键调用包括 `CustomOp.register`, `super.__init__`, `self.native_impl`, `self.forward_native`, `rocm_aiter_ops.is_fused_moe_enabled`, `rocm_aiter_grouped_topk`。 它会写入或更新 `native_impl`, `topk`, `renormalize`, `num_expert_group`, `topk_group`, `scoring_func`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 172-190 — method `GroupedTopk.__init__`
```python
    def __init__(
        self,
        topk: int,
        renormalize: bool,
        num_expert_group: int = 0,
        topk_group: int = 0,
        scoring_func: str = "softmax",
        routed_scaling_factor: float = 1.0,
        num_fused_shared_experts: int = 0,
    ) -> None:
        super().__init__()
        self.native_impl = grouped_topk
        self.topk = topk
        self.renormalize = renormalize
        self.num_expert_group = num_expert_group
        self.topk_group = topk_group
        self.scoring_func = scoring_func
        self.routed_scaling_factor = routed_scaling_factor
        self.num_fused_shared_experts = num_fused_shared_experts
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `topk`, `renormalize`, `num_expert_group`, `topk_group`, `scoring_func`, `routed_scaling_factor`. Key calls include `super.__init__`, `super`. It writes or updates `native_impl`, `topk`, `renormalize`, `num_expert_group`, `topk_group`, `scoring_func`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `topk`, `renormalize`, `num_expert_group`, `topk_group`, `scoring_func`, `routed_scaling_factor`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `native_impl`, `topk`, `renormalize`, `num_expert_group`, `topk_group`, `scoring_func`。

### Lines 192-208 — method `GroupedTopk.forward_native`
```python
    def forward_native(
        self,
        hidden_states: torch.Tensor,
        gating_output: torch.Tensor,
        e_score_correction_bias: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        return self.native_impl(
            hidden_states,
            gating_output,
            self.topk,
            self.renormalize,
            self.num_expert_group,
            self.topk_group,
            self.scoring_func,
            self.routed_scaling_factor,
            e_score_correction_bias,
        )
```
**EN:** This method defines `forward_native`. It executes the main forward/runtime path for this component. The main inputs are `hidden_states`, `gating_output`, `e_score_correction_bias`. Key calls include `self.native_impl`.
**CN:** 该方法定义 `forward_native`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `hidden_states`, `gating_output`, `e_score_correction_bias`。 关键调用包括 `self.native_impl`。

### Lines 210-218 — method `GroupedTopk.forward_cuda`
```python
    def forward_cuda(
        self,
        hidden_states: torch.Tensor,
        gating_output: torch.Tensor,
        e_score_correction_bias: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        return self.forward_native(
            hidden_states, gating_output, e_score_correction_bias
        )
```
**EN:** This method defines `forward_cuda`. It executes the main forward/runtime path for this component. The main inputs are `hidden_states`, `gating_output`, `e_score_correction_bias`. Key calls include `self.forward_native`.
**CN:** 该方法定义 `forward_cuda`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `hidden_states`, `gating_output`, `e_score_correction_bias`。 关键调用包括 `self.forward_native`。

### Lines 220-244 — method `GroupedTopk.forward_hip`
```python
    def forward_hip(
        self,
        hidden_states: torch.Tensor,
        gating_output: torch.Tensor,
        e_score_correction_bias: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        if rocm_aiter_ops.is_fused_moe_enabled():
            if not rocm_aiter_ops.is_fusion_moe_shared_experts_enabled():
                assert self.num_fused_shared_experts == 0
            return rocm_aiter_grouped_topk(
                hidden_states,
                gating_output,
                self.topk,
                self.renormalize,
                self.num_expert_group,
                self.topk_group,
                self.scoring_func,
                self.routed_scaling_factor,
                e_score_correction_bias,
                self.num_fused_shared_experts,
            )
        else:
            return self.forward_native(
                hidden_states, gating_output, e_score_correction_bias
            )
```
**EN:** This method defines `forward_hip`. It executes the main forward/runtime path for this component. The main inputs are `hidden_states`, `gating_output`, `e_score_correction_bias`. Key calls include `rocm_aiter_ops.is_fused_moe_enabled`, `rocm_aiter_grouped_topk`, `self.forward_native`, `rocm_aiter_ops.is_fusion_moe_shared_experts_enabled`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward_hip`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `hidden_states`, `gating_output`, `e_score_correction_bias`。 关键调用包括 `rocm_aiter_ops.is_fused_moe_enabled`, `rocm_aiter_grouped_topk`, `self.forward_native`, `rocm_aiter_ops.is_fusion_moe_shared_experts_enabled`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 247-351 — class `GroupedTopKRouter`
```python
class GroupedTopKRouter(BaseRouter):
    """Router using grouped top-k routing (e.g., DeepSeekV2/V3)."""

    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        num_expert_group: int,
        topk_group: int,
        renormalize: bool = True,
        scoring_func: str = "softmax",
        routed_scaling_factor: float = 1.0,
        e_score_correction_bias: torch.Tensor | None = None,
        num_fused_shared_experts: int = 0,
# ... omitted for brevity ...

        return topk_weights, topk_ids
```
**EN:** This class defines `GroupedTopKRouter`. It inherits from `BaseRouter`. Router using grouped top-k routing (e.g., DeepSeekV2/V3). Important methods include `__init__`, `routing_method_type`. Key calls include `super.__init__`, `get_routing_method_type`, `rocm_aiter_ops.is_fused_moe_enabled`, `grouped_topk_impl`, `valid_grouping`, `partial`. It writes or updates `num_expert_group`, `topk_group`, `renormalize`, `scoring_func`, `routed_scaling_factor`, `e_score_correction_bias`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `GroupedTopKRouter`。 它继承自 `BaseRouter`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `routing_method_type`。 关键调用包括 `super.__init__`, `get_routing_method_type`, `rocm_aiter_ops.is_fused_moe_enabled`, `grouped_topk_impl`, `valid_grouping`, `partial`。 它会写入或更新 `num_expert_group`, `topk_group`, `renormalize`, `scoring_func`, `routed_scaling_factor`, `e_score_correction_bias`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 250-276 — method `GroupedTopKRouter.__init__`
```python
    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        num_expert_group: int,
        topk_group: int,
        renormalize: bool = True,
        scoring_func: str = "softmax",
        routed_scaling_factor: float = 1.0,
        e_score_correction_bias: torch.Tensor | None = None,
        num_fused_shared_experts: int = 0,
        eplb_state: EplbLayerState | None = None,
        indices_type_getter: Callable[[], torch.dtype | None] | None = None,
    ):
        super().__init__(
            top_k=top_k,
            global_num_experts=global_num_experts,
            eplb_state=eplb_state,
            indices_type_getter=indices_type_getter,
        )
        self.num_expert_group = num_expert_group
        self.topk_group = topk_group
        self.renormalize = renormalize
        self.scoring_func = scoring_func
        self.routed_scaling_factor = routed_scaling_factor
        self.e_score_correction_bias = e_score_correction_bias
        self.num_fused_shared_experts = num_fused_shared_experts
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `top_k`, `global_num_experts`, `num_expert_group`, `topk_group`, `renormalize`, `scoring_func`. Key calls include `super.__init__`, `super`. It writes or updates `num_expert_group`, `topk_group`, `renormalize`, `scoring_func`, `routed_scaling_factor`, `e_score_correction_bias`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `top_k`, `global_num_experts`, `num_expert_group`, `topk_group`, `renormalize`, `scoring_func`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `num_expert_group`, `topk_group`, `renormalize`, `scoring_func`, `routed_scaling_factor`, `e_score_correction_bias`。

### Lines 278-286 — method `GroupedTopKRouter.routing_method_type`
```python
    @property
    def routing_method_type(self) -> RoutingMethodType:
        return get_routing_method_type(
            scoring_func=self.scoring_func,
            top_k=self.top_k,
            renormalize=self.renormalize,
            num_expert_group=self.num_expert_group,
            has_e_score_bias=self.e_score_correction_bias is not None,
        )
```
**EN:** This method defines `routing_method_type`. It computes routing scores or expert-selection behavior for MoE execution. Key calls include `get_routing_method_type`.
**CN:** 该方法定义 `routing_method_type`。 它为 MoE 执行计算路由得分或专家选择行为。 关键调用包括 `get_routing_method_type`。

## Key Concepts / 关键概念
- [EN] Moe router scoring and top-k expert selection / [CN] MoE 路由打分与 top-k 专家选择
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `fused_grouped_topk`, `grouped_topk`, `GroupedTopk`, `GroupedTopKRouter` / [CN] 核心符号：`fused_grouped_topk`, `grouped_topk`, `GroupedTopk`, `GroupedTopKRouter`

## Dependencies / 依赖关系
- **External**: `collections.abc`, `functools`, `torch` / **外部依赖**: `collections.abc`, `functools`, `torch`
- **Internal**: `vllm`, `vllm._aiter_ops`, `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe`, `vllm.model_executor.layers.fused_moe.router.base_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_bias_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_router`, `vllm.model_executor.utils` / **内部依赖**: `vllm`, `vllm._aiter_ops`, `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe`, `vllm.model_executor.layers.fused_moe.router.base_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_bias_router`, `vllm.model_executor.layers.fused_moe.router.fused_topk_router`, `vllm.model_executor.utils`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
