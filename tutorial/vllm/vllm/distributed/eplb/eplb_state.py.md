# eplb_state.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/eplb/eplb_state.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Expert parallelism load balancer (EPLB) metrics and states / 实现专家并行负载均衡的策略、状态或工作进程逻辑。

## Line-by-Line Analysis / 逐行分析
### Module overview / 模块概览
```python
"""
Expert parallelism load balancer (EPLB) metrics and states.

# Glossary

- **Logical Expert**: An expert that is part of the model's logical structure.
  It holds a set of weights and is replicated across multiple physical
  experts.
- **Redundant Expert**: To achieve load balancing, for some popular logical
  experts, we create additional copies of the expert weights. During inference,
  each of these copies can be routed to by the same set of tokens.
- **Physical Expert**: An expert that is instantiated on a specific device.
  It is a replica of a logical expert and can be rearranged across devices.
  I.e., one logical expert may have multiple sets of weights initialized on
  different devices, and each of these sets is a physical expert.
- **Local Physical Expert**: A physical expert that is instantiated on the
  current device.

For example: DeepSeek-R1 has 256 logical experts, so each MoE layer
has 256 sets of linear layer weights in the model parameters. If we add 32
redundant experts, DeepSeek-R1 will have 256 + 32 = 288 physical experts in
total. And when deploying, we'll have 288 sets of linear layer weights for each
MoE layer. If we have 32 EP ranks, then each GPU will hold 288 / 32 = 9 local
physical experts.
"""
```
**EN:** The opening docstring defines the module scope and high-level contract. It highlights: Expert parallelism load balancer (EPLB) metrics and states.
**CN:** 开头的文档字符串给出了模块范围与高层契约。 它重点概括了文件的职责、参与方以及主要接口。

### Imports and setup / 导入与初始化
```python
import threading
from collections.abc import Sequence
from dataclasses import dataclass

import torch
from torch.distributed import ProcessGroup, all_reduce

from vllm.config import ModelConfig, ParallelConfig
from vllm.distributed.parallel_state import (
    get_ep_group,
    get_eplb_group,
    get_node_count,
    in_the_same_node_as,
)
from vllm.distributed.stateless_coordinator import StatelessGroupCoordinator
from vllm.distributed.utils import StatelessProcessGroup
from vllm.logger import init_logger
from vllm.model_executor.models.interfaces import MixtureOfExperts

from .async_worker import start_async_worker
from .eplb_communicator import EplbCommunicator, create_eplb_communicator
from .eplb_utils import CpuGpuEvent
from .policy import EPLB_POLICIES, AbstractEplbPolicy, DefaultEplbPolicy
from .rebalance_execute import (
    AsyncEplbLayerResult,
    move_from_buffer,
    rearrange_expert_weights_inplace,
)
```
**EN:** This block imports `threading`, `collections.abc`, `dataclasses`, `torch`, `torch.distributed`, `vllm.config` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `threading`, `collections.abc`, `dataclasses`, `torch`, `torch.distributed`, `vllm.config`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Class `EplbStats` / 类 `EplbStats`
```python
@dataclass
class EplbStats:
    """
    Model stats used in EPLB rebalancing algorithm.
    """

    global_expert_load_window: torch.Tensor
    """
    Experts load window.
    Shape: (window_size, num_moe_layers, num_physical_experts)
    """
    num_replicas: int
    """
    Number of physical experts.
    """
    num_groups: int
    """
    Number of expert groups.
    """
    num_nodes: int
    """
    Number of nodes.
    """
    num_gpus: int
    """
    Number of GPUs.
    """
```
**EN:** Declares `EplbStats`, a dataclass. It packages structured data fields such as `global_expert_load_window`, `num_replicas`, `num_groups`, `num_nodes`, `num_gpus`. The docstring summarizes its role as: Model stats used in EPLB rebalancing algorithm.
**CN:** 声明 `EplbStats`，它是一个数据类。 它封装了 `global_expert_load_window`, `num_replicas`, `num_groups`, `num_nodes`, `num_gpus` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `EplbModelState` / 类 `EplbModelState`
```python
@dataclass
class EplbModelState:
    """EPLB metrics."""

    physical_to_logical_map: torch.Tensor
    """
    Mapping from physical experts to logical experts.

    Shape: (num_moe_layers, num_physical_experts)

    # Example

    For a 2-layer MoE model with 6 physical experts and 4 logical experts on 3
    EP ranks, the mapping could look like this:

    ```
    [[0, 1, 2, 3, 0, 1],
     [0, 2, 0, 1, 0, 3]]
    ```
    """
    logical_to_physical_map: torch.Tensor
    """
    Mapping from logical experts to physical experts.

    This is a sparse matrix, where -1 indicates no mapping.

    Shape: (num_moe_layers, num_logical_experts, num_redundant_experts + 1)

    # Example

    For a 2-layer MoE model with 6 physical experts and 4 logical experts on 3
    EP ranks, the mapping could look like this:

    ```
    [[[0, 4, -1],
      [1, 5, -1],
      [2, -1, -1],
      [3, -1, -1]],
     [[0, 2, 4],
      [3, -1, -1],
      [1, -1, -1],
      [5, -1, -1]]]
    ```
    """
    logical_replica_count: torch.Tensor
# ... truncated for analysis ...
    """
    pending_result: AsyncEplbLayerResult | None = None
    """
    Set by the async worker after all writes to expert_buffer are done. Consumed
    and reset to None by the main thread in move_to_workspace() after the contents of
    expert_buffer have been transferred out. At most one result is pending at a time.

    pending_result relies on the GIL to synchronize access between the main thread and
    the async worker.
    """
```
**EN:** Declares `EplbModelState`, a dataclass. It packages structured data fields such as `physical_to_logical_map`, `logical_to_physical_map`, `logical_replica_count`, `expert_load_pass`, `expert_load_window`. The docstring summarizes its role as: EPLB metrics.
**CN:** 声明 `EplbModelState`，它是一个数据类。 它封装了 `physical_to_logical_map`, `logical_to_physical_map`, `logical_replica_count`, `expert_load_pass`, `expert_load_window` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `EplbState` / 类 `EplbState`
```python
class EplbState:
    """
    EplbState of each expert parallel model. Key is the model config hash.
    """

    def __init__(self, parallel_config: ParallelConfig, device: torch.device):
        self.parallel_config = parallel_config
        self.device = device
        self.model_states: dict[str, EplbModelState] = {}
        self.policy: type[AbstractEplbPolicy] = DefaultEplbPolicy
        """
        Selected EPLB algorithm class
        """
        self.expert_load_window_step: int = 0
        """
        Current step in the sliding window.

        Different from `expert_rearrangement_step`, 
        each EP rank may have its own `expert_load_window_step`.
        """
        self.expert_load_window_size: int = 0
        """
        Size of the expert load sliding window.
        This is a constant and is taken from the config.
        """
        self.expert_rearrangement_step: int = 0
        """
        Steps after last rearrangement.
        Will trigger a rearrangement if it exceeds the threshold.

        NOTE: Keep in mind that all EP ranks need to have the same
        `expert_rearrangement_step` value to ensure synchronization.
        Otherwise, the rearrangement will hang at collective
        communication calls.
        """
        self.expert_rearrangement_step_interval: int = 0
        """
        Interval for expert rearrangement steps.
        This is a constant and is taken from the config.
        """
        self.should_record_tensor: torch.Tensor | None = None
        """
        Shared scalar bool tensor for all layers.  Every
        :class:`EplbLayerState` holds a reference to the **same** object so
        a single ``.fill_()`` updates all layers at once.  Allocated on the
# ... truncated for analysis ...
                max_num_replicas - num_replicas,
            ),
            value=-1,
        ).to(device)
        logical_replica_count = logical_replica_count_cpu.to(device)

        eplb_model_state.logical_to_physical_map.copy_(logical_to_physical_map)
        eplb_model_state.logical_replica_count.copy_(logical_replica_count)

        return eplb_state
```
**EN:** Declares `EplbState`, a class. Key methods include `__init__`, `build_initial_global_physical_to_logical_map`, `validate_ep_configuration`, `add_model`, `step`. The docstring summarizes its role as: EplbState of each expert parallel model. Key is the model config hash.
**CN:** 声明 `EplbState`，它是一个类。 关键方法包括 `__init__`, `build_initial_global_physical_to_logical_map`, `validate_ep_configuration`, `add_model`, `step`。 文档字符串概括了它在整体流程中的职责。

### Class `EplbLayerState` / 类 `EplbLayerState`
```python
@dataclass
class EplbLayerState:
    """Runtime EPLB data stored in the MoE layer."""

    expert_load_view: torch.Tensor | None = None
    logical_to_physical_map: torch.Tensor | None = None
    logical_replica_count: torch.Tensor | None = None
    should_record_tensor: torch.Tensor | None = None
    """
    Shared scalar bool tensor controlling whether to accumulate expert load
    metrics during this forward pass.  All layers reference the **same**
    tensor object, which is owned and updated by :class:`EplbState`.

    Set to ``False`` for the first ``step_interval - window_size`` steps of
    each rearrangement period: those steps would be overwritten in the
    sliding window before the next rearrangement, so recording them wastes
    GPU work.
    """

    def set_layer_state(
        self,
        moe_layer_idx: int,
        expert_load_view: torch.Tensor,
        logical_to_physical_map: torch.Tensor,
        logical_replica_count: torch.Tensor,
    ) -> None:
        self.expert_load_view = expert_load_view[moe_layer_idx]
        self.logical_to_physical_map = logical_to_physical_map[moe_layer_idx]
        self.logical_replica_count = logical_replica_count[moe_layer_idx]
```
**EN:** Declares `EplbLayerState`, a dataclass. It packages structured data fields such as `expert_load_view`, `logical_to_physical_map`, `logical_replica_count`, `should_record_tensor`. The docstring summarizes its role as: Runtime EPLB data stored in the MoE layer.
**CN:** 声明 `EplbLayerState`，它是一个数据类。 它封装了 `expert_load_view`, `logical_to_physical_map`, `logical_replica_count`, `should_record_tensor` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Function `_node_count_with_rank_mapping` / 函数 `_node_count_with_rank_mapping`
```python
def _node_count_with_rank_mapping(
    pg: ProcessGroup | StatelessProcessGroup,
    rank_mapping: dict[int, int],
) -> int:
    if isinstance(pg, ProcessGroup):
        world_size = torch.distributed.get_world_size(group=pg)
    else:
        world_size = pg.world_size

    if world_size == 1:
        return 1

    # Build node assignment map
    node_assignment = [0] * world_size  # rank -> node_id
    next_node_id = 0

    for current_rank in range(world_size):
        if node_assignment[current_rank] != 0:
            continue  # Already assigned to a node

        assert current_rank in rank_mapping
        if rank_mapping[current_rank] == -1:
            continue  # Pending shutdown

        # Assign current rank to a new node
        next_node_id += 1
        node_assignment[current_rank] = next_node_id

        # Find all ranks on the same node as current_rank
        same_node_flags = in_the_same_node_as(pg, current_rank)
        for other_rank, is_same_node in enumerate(same_node_flags):
            if is_same_node and node_assignment[other_rank] == 0:
                node_assignment[other_rank] = next_node_id

    return next_node_id
```
**EN:** `_node_count_with_rank_mapping` implements a focused helper routine for this module. It primarily works with arguments like `pg`, `rank_mapping`. Key calls include `isinstance`, `range`, `torch.distributed.get_world_size`.
**CN:** `_node_count_with_rank_mapping` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `pg`, `rank_mapping` 这样的参数。 关键调用包括 `isinstance`, `range`, `torch.distributed.get_world_size`。

### Function `compute_logical_maps` / 函数 `compute_logical_maps`
```python
def compute_logical_maps(
    physical_to_logical_map: torch.Tensor,
    num_logical_experts: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    """
    Derive logical_to_physical_map and logical_replica_count from
    physical_to_logical_map.

    Args:
        physical_to_logical_map: [num_layers, num_physical_experts], logical
            expert index for each physical expert slot
        num_logical_experts: total number of logical experts

    Returns:
        logical_to_physical_map: [num_layers, num_logical_experts, max_replicas],
            physical slots per logical expert; -1 where unused
        logical_replica_count: [num_layers, num_logical_experts], number of
            physical replicas per logical expert
    """
    device = physical_to_logical_map.device
    assert physical_to_logical_map.device.type == "cpu"

    dtype = physical_to_logical_map.dtype

    # If computing maps for a single layer, unsqueeze a single element layer dimension
    per_layer = physical_to_logical_map.dim() == 1
    physical_to_logical_map_view = physical_to_logical_map
    if per_layer:
        physical_to_logical_map_view = physical_to_logical_map.unsqueeze(0)
    assert len(physical_to_logical_map_view.shape) == 2
    num_layers, num_physical = physical_to_logical_map_view.shape

    valid_mask = physical_to_logical_map_view >= 0
    logical_replica_count = torch.zeros(
        num_layers,
        num_logical_experts,
        dtype=dtype,
        device=device,
    )
    logical_replica_count.scatter_add_(
        1,
        physical_to_logical_map_view.clamp(min=0),
        valid_mask.to(dtype),
    )
# ... truncated for analysis ...
        # Use the current running count as the replica index, then increment it.
        replica_idx = running_count[valid_layers, valid_experts]
        logical_to_physical_map_out[valid_layers, valid_experts, replica_idx] = phys_idx
        running_count[valid_layers, valid_experts] += 1

    # If computing maps for a single layer, squeeze out the extra layer dimension
    # before returning
    if per_layer:
        return logical_to_physical_map_out.squeeze(0), logical_replica_count.squeeze(0)
    return logical_to_physical_map_out, logical_replica_count
```
**EN:** `compute_logical_maps` implements a focused helper routine for this module. The docstring frames it as: Derive logical_to_physical_map and logical_replica_count from physical_to_logical_map. It primarily works with arguments like `physical_to_logical_map`, `num_logical_experts`. Key calls include `torch.zeros`, `logical_replica_count.scatter_add_`, `int`.
**CN:** `compute_logical_maps` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `physical_to_logical_map`, `num_logical_experts` 这样的参数。 关键调用包括 `torch.zeros`, `logical_replica_count.scatter_add_`, `int`。

### Function `_pad_out_tensor` / 函数 `_pad_out_tensor`
```python
def _pad_out_tensor(src: torch.Tensor, dst: torch.Tensor) -> None:
    src_padding = dst.shape[-1] - src.shape[-1]
    assert src_padding >= 0
    new_src = torch.nn.functional.pad(src, (0, src_padding), value=-1)
    dst.copy_(new_src)
```
**EN:** `_pad_out_tensor` implements a focused helper routine for this module. It primarily works with arguments like `src`, `dst`. Key calls include `torch.nn.functional.pad`, `dst.copy_`.
**CN:** `_pad_out_tensor` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `src`, `dst` 这样的参数。 关键调用包括 `torch.nn.functional.pad`, `dst.copy_`。

### Function `_commit_eplb_maps_for_layer` / 函数 `_commit_eplb_maps_for_layer`
```python
def _commit_eplb_maps_for_layer(
    model_state: EplbModelState,
    new_physical_to_logical_map: torch.Tensor,
    layer: int,
) -> None:
    """
    Per-layer version of _commit_eplb_maps that's used by the sync portion of EPLB
    when running async EPLB. Copies all of the new_* maps into model_state. After this
    function completes, the new mappings will become the current mappings and will be
    visible to the model.
    """

    # Commit physical_to_logical_map
    src = new_physical_to_logical_map
    dst = model_state.physical_to_logical_map[layer]
    assert src.shape == dst.shape, (
        "The number of physical experts must stay the same while running Async EPLB. "
        f"Current number of physical experts: {dst.shape[0]}. New number of physical "
        f"experts {src.shape[0]}."
    )
    dst.copy_(src, non_blocking=True)

    num_logical_experts = model_state.logical_to_physical_map.shape[1]
    new_logical, new_replica_count = compute_logical_maps(src, num_logical_experts)
    # Commit logical_to_physical_map
    _pad_out_tensor(
        src=new_logical,
        dst=model_state.logical_to_physical_map[layer],
    )

    # Commit logical_replica_count
    src = new_replica_count
    dst = model_state.logical_replica_count[layer]
    assert src.shape == dst.shape
    dst.copy_(src, non_blocking=True)
```
**EN:** `_commit_eplb_maps_for_layer` implements a focused helper routine for this module. The docstring frames it as: Per-layer version of _commit_eplb_maps that's used by the sync portion of EPLB when running async EPLB. Copies all of the new_* maps into.... It primarily works with arguments like `model_state`, `new_physical_to_logical_map`, `layer`. Key calls include `dst.copy_`, `compute_logical_maps`, `_pad_out_tensor`.
**CN:** `_commit_eplb_maps_for_layer` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `model_state`, `new_physical_to_logical_map`, `layer` 这样的参数。 关键调用包括 `dst.copy_`, `compute_logical_maps`, `_pad_out_tensor`。

### Function `_commit_eplb_maps` / 函数 `_commit_eplb_maps`
```python
def _commit_eplb_maps(
    model_state: EplbModelState,
    new_physical_to_logical_map: torch.Tensor,
) -> None:
    """
    Copies all of the new_* maps into model_state. After this function completes,
    the new mappings will become the current mappings and will be visible to the
    model.
    """

    # Commit physical_to_logical_map
    src = new_physical_to_logical_map
    dst = model_state.physical_to_logical_map

    # Rare Case: When the number of physical experts has changed, discard the old
    # physical to logical expert map and use the new one. This only happens when the
    # number of GPUs available to vLLM changes while vLLM is running. Otherwise copy the
    # new map into the old one.
    if src.shape[1] != dst.shape[1]:
        model_state.physical_to_logical_map = src.to(dst.device)
    else:
        dst.copy_(src, non_blocking=True)

    num_logical_experts = model_state.logical_to_physical_map.shape[1]
    new_logical, new_replica_count = compute_logical_maps(src, num_logical_experts)
    # Commit logical_to_physical_map
    _pad_out_tensor(
        src=new_logical,
        dst=model_state.logical_to_physical_map,
    )

    # Commit logical_replica_count
    src = new_replica_count
    dst = model_state.logical_replica_count
    dst.copy_(src, non_blocking=True)
```
**EN:** `_commit_eplb_maps` implements a focused helper routine for this module. The docstring frames it as: Copies all of the new_* maps into model_state. After this function completes, the new mappings will become the current mappings and will be.... It primarily works with arguments like `model_state`, `new_physical_to_logical_map`. Key calls include `compute_logical_maps`, `_pad_out_tensor`, `dst.copy_`.
**CN:** `_commit_eplb_maps` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `model_state`, `new_physical_to_logical_map` 这样的参数。 关键调用包括 `compute_logical_maps`, `_pad_out_tensor`, `dst.copy_`。

### Function `_move_to_workspace` / 函数 `_move_to_workspace`
```python
def _move_to_workspace(
    model_state: EplbModelState,
    ep_rank: int,
) -> None:
    result = model_state.pending_result
    assert result is not None
    move_from_buffer(
        expert_weights=model_state.model.expert_weights[result.layer_idx],
        expert_weights_buffers=model_state.expert_buffer,
        transfer_metadata=result.transfer_metadata,
        new_indices=result.new_physical_to_logical_map.numpy(),
        ep_rank=ep_rank,
    )

    _commit_eplb_maps_for_layer(
        model_state,
        new_physical_to_logical_map=result.new_physical_to_logical_map,
        layer=result.layer_idx,
    )

    if result.layer_idx == model_state.model.num_moe_layers - 1:
        model_state.rebalanced = False

    # Reset pending_result before unblocking the async worker
    model_state.pending_result = None
    result.consumed_event.record()
```
**EN:** `_move_to_workspace` implements a focused helper routine for this module. It primarily works with arguments like `model_state`, `ep_rank`. Key calls include `move_from_buffer`, `_commit_eplb_maps_for_layer`, `result.consumed_event.record`.
**CN:** `_move_to_workspace` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `model_state`, `ep_rank` 这样的参数。 关键调用包括 `move_from_buffer`, `_commit_eplb_maps_for_layer`, `result.consumed_event.record`。

## Key Concepts / 关键概念
- Expert-parallel load balancing / 专家并行负载均衡
- `EplbStats`: dataclass interface or data carrier / `EplbStats`：数据类接口或数据载体
- `EplbModelState`: dataclass interface or data carrier / `EplbModelState`：数据类接口或数据载体
- `EplbState`: class interface or data carrier / `EplbState`：类接口或数据载体
- `EplbLayerState`: dataclass interface or data carrier / `EplbLayerState`：数据类接口或数据载体
- `_node_count_with_rank_mapping`: module-level helper or API entry / `_node_count_with_rank_mapping`：模块级辅助函数或 API 入口

## Dependencies / 依赖关系
- **Standard library / 标准库**: `threading`, `collections.abc`, `dataclasses`
- **Third-party / 第三方**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `vllm.config`, `vllm.distributed.parallel_state`, `vllm.distributed.stateless_coordinator`, `vllm.distributed.utils`, `vllm.logger`, `vllm.model_executor.models.interfaces`, `.async_worker`, `.eplb_communicator`, `.eplb_utils`, `.policy`, `.rebalance_execute`
