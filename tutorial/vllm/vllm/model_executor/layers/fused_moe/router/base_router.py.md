# base_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/router/base_router.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE router scoring and top-k expert selection; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 路由打分与 top-k 专家选择；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-13 — imports and setup
```python
from abc import abstractmethod
from collections.abc import Callable

import torch

from vllm.distributed.eplb.eplb_state import EplbLayerState
from vllm.model_executor.layers.fused_moe.router.fused_moe_router import (
    FusedMoERouter,
)
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 138-298 — class `BaseRouter`
```python
class BaseRouter(FusedMoERouter):
    """
    Base router class that provides common functionality for all router implementations.

    This class implements the template method pattern where select_experts() handles
    common pre-processing and post-processing, delegating the actual routing logic
    to the abstract _compute_routing() method.
    """

    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        eplb_state: EplbLayerState | None = None,
# ... omitted for brevity ...

        return topk_weights, topk_ids
```
**EN:** This class defines `BaseRouter`. It inherits from `FusedMoERouter`. Base router class that provides common functionality for all router implementations. Important methods include `__init__`, `set_capture_fn`, `select_experts`. Key calls include `super.__init__`, `self._validate_eplb_state`, `self._get_indices_type`, `self._compute_routing`, `self._apply_eplb_mapping`, `self._convert_indices_dtype`. It writes or updates `top_k`, `global_num_experts`, `eplb_state`, `indices_type_getter`, `capture_fn`, `indices_type`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `BaseRouter`。 它继承自 `FusedMoERouter`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `set_capture_fn`, `select_experts`。 关键调用包括 `super.__init__`, `self._validate_eplb_state`, `self._get_indices_type`, `self._compute_routing`, `self._apply_eplb_mapping`, `self._convert_indices_dtype`。 它会写入或更新 `top_k`, `global_num_experts`, `eplb_state`, `indices_type_getter`, `capture_fn`, `indices_type`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 147-173 — method `BaseRouter.__init__`
```python
    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        eplb_state: EplbLayerState | None = None,
        # TODO(bnell): Once the MK is constructed at layer init time, we
        # can make this a plain value instead of a callback.
        indices_type_getter: Callable[[], torch.dtype | None] | None = None,
    ):
        """
        Note: the indices dtype might not be available at router construction
        time, so we need to supply a callback to get it at runtime.  This is
        because the indices type is supplied by modular kernels which are
        created after MoE layer/router construction.

        Args:
            top_k: Number of experts to select per token
            global_num_experts: Total number of experts
            eplb_state: Optional EPLBLayerState for load balancing
            indices_type_getter: Optional callback to get indices dtype
        """
        super().__init__()
        self.top_k = top_k
        self.global_num_experts = global_num_experts
        self.eplb_state = eplb_state
        self.indices_type_getter = indices_type_getter
        self.capture_fn: Callable[[torch.Tensor], None] | None = None
```
**EN:** This method defines `__init__`. Note: the indices dtype might not be available at router construction time, so we need to supply a callback to get it at runtime. The main inputs are `top_k`, `global_num_experts`, `eplb_state`, `indices_type_getter`. Key calls include `super.__init__`, `super`. It writes or updates `top_k`, `global_num_experts`, `eplb_state`, `indices_type_getter`, `capture_fn`.
**CN:** 该方法定义 `__init__`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `top_k`, `global_num_experts`, `eplb_state`, `indices_type_getter`。 关键调用包括 `super.__init__`, `super`。 它会写入或更新 `top_k`, `global_num_experts`, `eplb_state`, `indices_type_getter`, `capture_fn`。

### Lines 175-177 — method `BaseRouter.set_capture_fn`
```python
    def set_capture_fn(self, capture_fn: Callable[[torch.Tensor], None] | None) -> None:
        """Set a capture callback for logical routed expert IDs."""
        self.capture_fn = capture_fn
```
**EN:** This method defines `set_capture_fn`. Set a capture callback for logical routed expert IDs. The main inputs are `capture_fn`. It writes or updates `capture_fn`.
**CN:** 该方法定义 `set_capture_fn`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `capture_fn`。 它会写入或更新 `capture_fn`。

### Lines 250-298 — method `BaseRouter.select_experts`
```python
    def select_experts(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        *,
        input_ids: torch.Tensor | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Route the input hidden states to the top-k experts based on the
        router logits.

        This method implements the template method pattern:
        1. Validates EPLB state
        2. Gets indices type
        3. Calls _compute_routing() to get topk_weights and topk_ids
        4. Applies EPLB mapping if enabled
        5. Converts indices dtype if needed

        Returns:
            (topk_weights, topk_ids)
            (tuple[torch.Tensor, torch.Tensor]):
            The weights and expert ids computation result.

            **Compatibility**: When EPLB is not enabled, the returned ids are
            equivalent to global logical ids, so should be compatible with
            plain MoE implementations without redundant experts.
        """
        # Step 1: Validate EPLB state
        self._validate_eplb_state()

        # Step 2: Get indices type.
        indices_type = self._get_indices_type()

        # Step 3: Compute routing (delegated to subclass)
        topk_weights, topk_ids = self._compute_routing(
            hidden_states, router_logits, indices_type, input_ids=input_ids
        )

        # Capture logical ids before EPLB mapping.
        if self.capture_fn is not None:
            self.capture_fn(topk_ids)

        # Step 4: Apply EPLB mapping
        topk_ids = self._apply_eplb_mapping(topk_ids)
# ... omitted for brevity ...

        return topk_weights, topk_ids
```
**EN:** This method defines `select_experts`. Route the input hidden states to the top-k experts based on the router logits. The main inputs are `hidden_states`, `router_logits`, `input_ids`. Key calls include `self._validate_eplb_state`, `self._get_indices_type`, `self._compute_routing`, `self._apply_eplb_mapping`, `self._convert_indices_dtype`, `self.capture_fn`. It writes or updates `indices_type`, `topk_weights`, `topk_ids`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `select_experts`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `router_logits`, `input_ids`。 关键调用包括 `self._validate_eplb_state`, `self._get_indices_type`, `self._compute_routing`, `self._apply_eplb_mapping`, `self._convert_indices_dtype`, `self.capture_fn`。 它会写入或更新 `indices_type`, `topk_weights`, `topk_ids`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Moe router scoring and top-k expert selection / [CN] MoE 路由打分与 top-k 专家选择
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `BaseRouter` / [CN] 核心符号：`BaseRouter`

## Dependencies / 依赖关系
- **External**: `abc`, `collections.abc`, `torch` / **外部依赖**: `abc`, `collections.abc`, `torch`
- **Internal**: `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.router.fused_moe_router`, `vllm.platforms`, `vllm.triton_utils` / **内部依赖**: `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.router.fused_moe_router`, `vllm.platforms`, `vllm.triton_utils`
- **Runtime traits**: platform-aware dispatch, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, Triton kernels, distributed collectives
