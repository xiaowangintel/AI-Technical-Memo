# expert_map_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/expert_map_manager.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Expert Map Manager for MoE layers. / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 10-19 — imports and setup
```python
import torch

from vllm.config.parallel import ExpertPlacementStrategy
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.config import FusedMoEParallelConfig
from vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe import (
    init_aiter_topK_meta_data,
)

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 22-113 — function `determine_expert_map`
```python
def determine_expert_map(
    ep_size: int,
    ep_rank: int,
    global_num_experts: int,
    expert_placement_strategy: ExpertPlacementStrategy = "linear",
    num_fused_shared_experts: int = 0,
    return_expert_mask: bool = False,
) -> tuple[int, torch.Tensor | None, torch.Tensor | None]:
    """
    Calculates how many experts should be assigned to each rank for EP and
    creates a mapping from global to local expert index. Experts are
    distributed evenly across ranks. Any remaining are assigned to the
    last rank.

    Args:
        ep_size: The size of the expert parallel group
        ep_rank: The rank of the current process in the expert parallel
            group
        global_num_experts: The total number of experts in the model.
        expert_placement_strategy: The expert placement strategy.
        num_fused_shared_experts: Number of fused shared experts (for AITER)
        return_expert_mask: Whether to return expert mask for AITER

    Returns:
        tuple[int, Optional[torch.Tensor], Optional[torch.Tensor]]: A tuple containing:
            - local_num_experts (int): The number of experts assigned
                to the current rank.
            - expert_map (Optional[torch.Tensor]): A tensor of shape
                (global_num_experts,) mapping from global to local index.
                Contains -1 for experts not assigned to the current rank.
                Returns None if ep_size is 1.
            - expert_mask (Optional[torch.Tensor]): A tensor of shape
                (global_num_experts + num_fused_shared_experts + 1,)
                containing 1 for experts assigned to the current rank
                and 0 for sentinel.
                Returns None if ep_size is 1.
                Used only when AITER MOE is enabled.
    """
    from typing import get_args

    assert ep_size > 0
    if ep_size == 1:
        return (global_num_experts, None, None)

# ... omitted for brevity ...

    return (local_num_experts, expert_map, expert_mask)
```
**EN:** This function defines `determine_expert_map`. Calculates how many experts should be assigned to each rank for EP and creates a mapping from global to local expert index. The main inputs are `ep_size`, `ep_rank`, `global_num_experts`, `expert_placement_strategy`, `num_fused_shared_experts`, `return_expert_mask`. Key calls include `torch.full`, `torch.arange`, `torch.ones`, `torch.cat`, `min`, `ValueError`. It writes or updates `base_experts`, `remainder`, `local_num_experts`, `expert_map`, `expert_mask`, `start_idx`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `determine_expert_map`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `ep_size`, `ep_rank`, `global_num_experts`, `expert_placement_strategy`, `num_fused_shared_experts`, `return_expert_mask`。 关键调用包括 `torch.full`, `torch.arange`, `torch.ones`, `torch.cat`, `min`, `ValueError`。 它会写入或更新 `base_experts`, `remainder`, `local_num_experts`, `expert_map`, `expert_mask`, `start_idx`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 116-149 — function `determine_expert_placement_strategy`
```python
def determine_expert_placement_strategy(
    expert_placement_strategy: ExpertPlacementStrategy,
    moe_parallel_config: FusedMoEParallelConfig,
    num_expert_group: int | None,
    num_redundant_experts: int,
    enable_eplb: bool,
) -> ExpertPlacementStrategy:
    if expert_placement_strategy == "round_robin":
        round_robin_supported = (
            (num_expert_group is not None and num_expert_group > 1)
            and num_redundant_experts == 0
            and not enable_eplb
        )

        if not round_robin_supported:
            logger.warning(
                "Round-robin expert placement is only supported for "
                "models with multiple expert groups and no redundant "
                "experts. Falling back to linear expert placement."
            )
            return "linear"
        if (
            moe_parallel_config.use_all2all_kernels
            and not moe_parallel_config.needs_round_robin_routing_tables
        ):
            logger.warning(
                "Round-robin expert placement currently only supports "
                "the DeepEP low-latency or NIXL EP backend, but '%s' was configured. "
                "Falling back to linear expert placement.",
                moe_parallel_config.all2all_backend,
            )
            return "linear"

    return expert_placement_strategy
```
**EN:** This function defines `determine_expert_placement_strategy`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `expert_placement_strategy`, `moe_parallel_config`, `num_expert_group`, `num_redundant_experts`, `enable_eplb`. Key calls include `logger.warning`. It writes or updates `round_robin_supported`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `determine_expert_placement_strategy`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `expert_placement_strategy`, `moe_parallel_config`, `num_expert_group`, `num_redundant_experts`, `enable_eplb`。 关键调用包括 `logger.warning`。 它会写入或更新 `round_robin_supported`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 152-516 — class `ExpertMapManager`
```python
class ExpertMapManager:
    """
    Manages expert ID mappings and placement for Expert Parallelism.

    Responsibilities:
    - Calculate local vs global expert counts
    - Map between global, local, and physical expert IDs
    - Manage placement strategies (linear, round_robin)
    - Maintain routing tables for round-robin placement
    - Support dynamic reconfiguration of EP topology

    When expert_map is required:
    - Expert Parallelism (EP) is enabled, i.e., when ep_size > 1
    - EP disabled (ep_size == 1): expert_map is None
# ... omitted for brevity ...

        return (global_to_physical, physical_to_global, local_global)
```
**EN:** This class defines `ExpertMapManager`. Manages expert ID mappings and placement for Expert Parallelism. Important methods include `__init__`, `use_ep`, `ep_size`, `ep_rank`, `tp_size`, `tp_rank`. Key calls include `self._determine_placement_strategy`, `self._calculate_expert_maps`, `self._init_routing_tables`, `self._init_aiter_shared_experts_topK_buffer`, `self._expert_map.item`, `torch.where.tolist`. It writes or updates `global_num_experts`, `moe_parallel_config`, `num_fused_shared_experts`, `rocm_aiter_enabled`, `top_k`, `max_num_batched_tokens`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `ExpertMapManager`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `use_ep`, `ep_size`, `ep_rank`, `tp_size`, `tp_rank`。 关键调用包括 `self._determine_placement_strategy`, `self._calculate_expert_maps`, `self._init_routing_tables`, `self._init_aiter_shared_experts_topK_buffer`, `self._expert_map.item`, `torch.where.tolist`。 它会写入或更新 `global_num_experts`, `moe_parallel_config`, `num_fused_shared_experts`, `rocm_aiter_enabled`, `top_k`, `max_num_batched_tokens`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 183-256 — method `ExpertMapManager.__init__`
```python
    def __init__(
        self,
        max_num_batched_tokens: int,
        top_k: int,
        global_num_experts: int,
        num_redundant_experts: int,
        num_expert_group: int | None,
        moe_parallel_config: FusedMoEParallelConfig,
        placement_strategy: ExpertPlacementStrategy,
        enable_eplb: bool,
        num_fused_shared_experts: int = 0,
        rocm_aiter_enabled: bool = False,
    ):
        """
        Initialize expert map manager.

        Args:
            global_num_experts: Total number of experts across all ranks
            moe_parallel_config: MoE parallel configuration (contains ep_size,
                                 ep_rank, backend flags)
            placement_strategy: Strategy for placing experts ('linear' or 'round_robin')
            num_fused_shared_experts: Number of fused shared experts (for AITER)
            rocm_aiter_enabled: Whether ROCm AITER fusion is enabled
        """
        self.global_num_experts = global_num_experts
        self.moe_parallel_config = moe_parallel_config
        self.num_fused_shared_experts = num_fused_shared_experts
        self.rocm_aiter_enabled = rocm_aiter_enabled
        self.top_k = top_k
        self.max_num_batched_tokens = max_num_batched_tokens

        if moe_parallel_config.use_ep:
            # Determine expert placement strategy before creating manager
            placement_strategy = determine_expert_placement_strategy(
                expert_placement_strategy=placement_strategy,
                moe_parallel_config=moe_parallel_config,
                num_expert_group=num_expert_group,
                num_redundant_experts=num_redundant_experts,
                enable_eplb=enable_eplb,
            )

        # Determine effective placement strategy
        self._placement_strategy = self._determine_placement_strategy(
            placement_strategy
# ... omitted for brevity ...
                self.get_compressed_map_string(),
            )
```
**EN:** This method defines `__init__`. Initialize expert map manager. The main inputs are `max_num_batched_tokens`, `top_k`, `global_num_experts`, `num_redundant_experts`, `num_expert_group`, `moe_parallel_config`. Key calls include `self._determine_placement_strategy`, `self._calculate_expert_maps`, `self._init_routing_tables`, `self._init_aiter_shared_experts_topK_buffer`, `determine_expert_placement_strategy`, `logger.info_once`. It writes or updates `global_num_experts`, `moe_parallel_config`, `num_fused_shared_experts`, `rocm_aiter_enabled`, `top_k`, `max_num_batched_tokens`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `max_num_batched_tokens`, `top_k`, `global_num_experts`, `num_redundant_experts`, `num_expert_group`, `moe_parallel_config`。 关键调用包括 `self._determine_placement_strategy`, `self._calculate_expert_maps`, `self._init_routing_tables`, `self._init_aiter_shared_experts_topK_buffer`, `determine_expert_placement_strategy`, `logger.info_once`。 它会写入或更新 `global_num_experts`, `moe_parallel_config`, `num_fused_shared_experts`, `rocm_aiter_enabled`, `top_k`, `max_num_batched_tokens`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 272-274 — method `ExpertMapManager.use_ep`
```python
    @property
    def use_ep(self) -> int:
        return self.moe_parallel_config.use_ep
```
**EN:** This method defines `use_ep`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `use_ep`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 276-278 — method `ExpertMapManager.ep_size`
```python
    @property
    def ep_size(self) -> int:
        return self.moe_parallel_config.ep_size
```
**EN:** This method defines `ep_size`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `ep_size`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 280-282 — method `ExpertMapManager.ep_rank`
```python
    @property
    def ep_rank(self) -> int:
        return self.moe_parallel_config.ep_rank
```
**EN:** This method defines `ep_rank`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `ep_rank`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 284-286 — method `ExpertMapManager.tp_size`
```python
    @property
    def tp_size(self) -> int:
        return self.moe_parallel_config.tp_size
```
**EN:** This method defines `tp_size`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `tp_size`。 它管理专家分发、融合内核或 MoE 特有的张量流。

### Lines 288-290 — method `ExpertMapManager.tp_rank`
```python
    @property
    def tp_rank(self) -> int:
        return self.moe_parallel_config.tp_rank
```
**EN:** This method defines `tp_rank`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow.
**CN:** 该方法定义 `tp_rank`。 它管理专家分发、融合内核或 MoE 特有的张量流。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `determine_expert_map`, `determine_expert_placement_strategy`, `ExpertMapManager` / [CN] 核心符号：`determine_expert_map`, `determine_expert_placement_strategy`, `ExpertMapManager`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm.config.parallel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe` / **内部依赖**: `vllm.config.parallel`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.experts.rocm_aiter_moe`
- **Runtime traits**: Triton kernels, distributed collectives / **运行时特征**: Triton kernels, distributed collectives
