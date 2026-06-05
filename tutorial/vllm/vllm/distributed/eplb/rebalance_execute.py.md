# rebalance_execute.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/eplb/rebalance_execute.py`
- **Repository**: vllm-project/vllm
- **Purpose**: The actual execution of the rearrangement / 实现专家并行负载均衡的策略、状态或工作进程逻辑。

## Line-by-Line Analysis / 逐行分析
### Module overview / 模块概览
```python
"""
The actual execution of the rearrangement.

This involves the exchange of expert weights between GPUs.
"""
```
**EN:** The opening docstring defines the module scope and high-level contract. It highlights: The actual execution of the rearrangement.
**CN:** 开头的文档字符串给出了模块范围与高层契约。 它重点概括了文件的职责、参与方以及主要接口。

### Imports and setup / 导入与初始化
```python
from collections.abc import Sequence
from dataclasses import dataclass

import numpy as np
import torch
from torch.distributed import ProcessGroup, all_gather

from vllm.distributed.eplb.eplb_communicator import EplbCommunicator
from vllm.distributed.eplb.eplb_utils import CpuGpuEvent
from vllm.logger import init_logger
```
**EN:** This block imports `collections.abc`, `dataclasses`, `numpy`, `torch`, `torch.distributed`, `vllm.distributed.eplb.eplb_communicator` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `collections.abc`, `dataclasses`, `numpy`, `torch`, `torch.distributed`, `vllm.distributed.eplb.eplb_communicator`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Class `TransferMetadata` / 类 `TransferMetadata`
```python
@dataclass
class TransferMetadata:
    """Metadata describing a completed EPLB buffer transfer."""

    is_unchanged: np.ndarray
    """Mask of (num_local_experts,) indicating experts unchanged after rebalance."""
    is_received_locally: np.ndarray
    """Mask of (num_local_experts,) indicating experts received from local data."""
    recv_primary_mask: np.ndarray
    """Mask of (num_local_experts,) indicating primary experts received."""
    recv_count: int
    """Number of received experts for the layer."""
    recv_expert_ids: np.ndarray
    """Expert ids (num_local_experts,) of remote primary experts."""
    recv_dst_rows: np.ndarray
    """Target expert indices (num_local_experts,) in local tensors to send."""
```
**EN:** Declares `TransferMetadata`, a dataclass. It packages structured data fields such as `is_unchanged`, `is_received_locally`, `recv_primary_mask`, `recv_count`, `recv_expert_ids`. The docstring summarizes its role as: Metadata describing a completed EPLB buffer transfer.
**CN:** 声明 `TransferMetadata`，它是一个数据类。 它封装了 `is_unchanged`, `is_received_locally`, `recv_primary_mask`, `recv_count`, `recv_expert_ids` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `AsyncEplbLayerResult` / 类 `AsyncEplbLayerResult`
```python
@dataclass
class AsyncEplbLayerResult:
    """
    The result of one completed async EPLB layer transfer.
    """

    layer_idx: int
    """Index of the MoE layer that was transferred."""
    new_physical_to_logical_map: torch.Tensor
    """
    New physical→logical mapping for layers_idx, on CPU.
    Shape: (num_physical_experts)
    """
    transfer_metadata: TransferMetadata
    """Metadata describing what was received during transfer_layer."""
    consumed_event: CpuGpuEvent
    """
    Event used to synchronize access to the intermediate buffer. The async worker calls
    wait() after it finishes transferring weights to the intermediate buffer. The main
    thread calls record() after it finishes transferring weights out of the intermediate
    buffer in _move_to_workspace()
    """
```
**EN:** Declares `AsyncEplbLayerResult`, a dataclass. It packages structured data fields such as `layer_idx`, `new_physical_to_logical_map`, `transfer_metadata`, `consumed_event`. The docstring summarizes its role as: The result of one completed async EPLB layer transfer.
**CN:** 声明 `AsyncEplbLayerResult`，它是一个数据类。 它封装了 `layer_idx`, `new_physical_to_logical_map`, `transfer_metadata`, `consumed_event` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Function `get_ep_ranks_with_experts_batch` / 函数 `get_ep_ranks_with_experts_batch`
```python
def get_ep_ranks_with_experts_batch(
    expert_ids: np.ndarray,
    num_local_experts: int,
    old_indices: np.ndarray,
    new_indices: np.ndarray,
) -> tuple[dict[int, list[int]], dict[int, list[int]]]:
    """
    Get the ranks of the experts that need to be exchanged.

    Args:
        expert_ids: 1D array of expert indices to query.
        num_local_experts: The number of local experts.
        old_indices: The old indices of the experts.
        new_indices: The new indices of the experts.

    Returns:
        A tuple of two dictionaries mapping expert_id to:
        - ranks_to_send: The ranks that have this expert and need to send.
        - ranks_to_recv: The ranks that need to receive this expert.
    """
    ranks_to_send_map: dict[int, list[int]] = {}
    ranks_to_recv_map: dict[int, list[int]] = {}

    # Fast path: if no experts, return empty dicts
    if expert_ids.size == 0:
        return ranks_to_send_map, ranks_to_recv_map

    unique_experts = np.unique(expert_ids)
    num_positions = len(old_indices)
    position_indices = np.arange(num_positions, dtype=np.int32)

    # Vectorized approach: find all positions matching any query expert in one pass
    # Use np.isin to get boolean masks for all relevant positions at once
    old_relevant_mask = np.isin(old_indices, unique_experts)
    new_relevant_mask = np.isin(new_indices, unique_experts)

    # Process old_indices (send ranks)
    if np.any(old_relevant_mask):
        old_relevant_positions = position_indices[old_relevant_mask]
        old_relevant_experts = old_indices[old_relevant_mask]
        old_relevant_ranks = old_relevant_positions // num_local_experts

        # Sort by expert first, then by position (to maintain first-appearance order)
        sort_order = np.lexsort((old_relevant_positions, old_relevant_experts))
        sorted_experts = old_relevant_experts[sort_order]
# ... truncated for analysis ...

    # Handle experts that only appear in old (send only) or new (recv only)
    for expert in unique_experts:
        expert = int(expert)
        if expert not in ranks_to_send_map:
            ranks_to_send_map[expert] = []
        if expert not in ranks_to_recv_map:
            ranks_to_recv_map[expert] = []

    return ranks_to_send_map, ranks_to_recv_map
```
**EN:** `get_ep_ranks_with_experts_batch` retrieves state or computed results for this module. The docstring frames it as: Get the ranks of the experts that need to be exchanged. It primarily works with arguments like `expert_ids`, `num_local_experts`, `old_indices`, `new_indices`. Key calls include `np.unique`, `len`, `np.arange`.
**CN:** `get_ep_ranks_with_experts_batch` 负责获取状态或计算结果。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `expert_ids`, `num_local_experts`, `old_indices`, `new_indices` 这样的参数。 关键调用包括 `np.unique`, `len`, `np.arange`。

### Function `move_to_buffer` / 函数 `move_to_buffer`
```python
def move_to_buffer(
    num_local_experts: int,
    old_indices: np.ndarray,
    new_indices: np.ndarray,
    expert_weights: Sequence[torch.Tensor],
    expert_weights_buffers: Sequence[torch.Tensor],
    cuda_stream: torch.cuda.Stream | None,
    ep_rank: int,
    communicator: EplbCommunicator,
) -> TransferMetadata:
    """
    Rearranges expert weights during EPLB rebalancing.

    Args:
        num_local_experts: Number of local experts.
        old_indices: (num_experts_total,) ndarray of current (old)
            global-to-local expert assignments.
        new_indices: (num_experts_total,) ndarray of desired (new)
            global-to-local assignments after rebalance.
        expert_weights: Original expert weights for the layer.
        expert_weights_buffers: Intermediate buffers (one per tensor).
        cuda_stream: CUDA stream for async copies (can be None for sync mode).
        ep_rank: Rank of this process in expert parallel group.
        communicator: EplbCommunicator instance for P2P communication.

    Returns:
        TransferMetadata: Metadata needed for completing remote weight transfers.
    """
    assert old_indices.shape == new_indices.shape
    recv_primary_mask = np.zeros((num_local_experts,), dtype=np.bool_)
    send_expert_ids = np.full((num_local_experts,), -1, dtype=np.int64)
    send_src_rows = np.full((num_local_experts,), -1, dtype=np.int32)
    recv_expert_ids = np.full((num_local_experts,), -1, dtype=np.int64)
    recv_dst_rows = np.full((num_local_experts,), -1, dtype=np.int32)

    base = ep_rank * num_local_experts
    local_rows = np.arange(num_local_experts, dtype=np.int32)
    local_global = base + local_rows

    old_local_expert_ids = old_indices[local_global]
    new_local_expert_ids = new_indices[local_global]

    # Unchanged mask
    is_unchanged = old_local_expert_ids == new_local_expert_ids
# ... truncated for analysis ...
    communicator.execute(old_indices=old_indices)
    # wait for the communication to finish
    return TransferMetadata(
        is_unchanged=is_unchanged,
        is_received_locally=is_received_locally,
        recv_primary_mask=recv_primary_mask,
        recv_count=recv_count,
        recv_expert_ids=recv_expert_ids,
        recv_dst_rows=recv_dst_rows,
    )
```
**EN:** `move_to_buffer` implements a focused helper routine for this module. The docstring frames it as: Rearranges expert weights during EPLB rebalancing. It primarily works with arguments like `num_local_experts`, `old_indices`, `new_indices`, `expert_weights`. Key calls include `np.zeros`, `np.full`, `np.arange`.
**CN:** `move_to_buffer` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `num_local_experts`, `old_indices`, `new_indices`, `expert_weights` 这样的参数。 关键调用包括 `np.zeros`, `np.full`, `np.arange`。

### Function `move_from_buffer` / 函数 `move_from_buffer`
```python
def move_from_buffer(
    expert_weights: Sequence[torch.Tensor],
    expert_weights_buffers: list[torch.Tensor],
    transfer_metadata: TransferMetadata,
    new_indices: np.ndarray,
    ep_rank: int,
) -> None:
    """
    Copies expert weights from communication buffers back to the target weight tensors
    after EPLB rebalancing.

    Args:
        expert_weights: List of the actual MoE layer weights used in the execution.
        expert_weights_buffers: Intermediate buffers containing the experts weights
            after the transfer is completed.
        transfer_metadata: TransferMetadata containing transfer metadata.
        new_indices: (num_experts_total,) mapping from local rows to desired
            (possibly global) expert id, after rebalance.
        ep_rank: Rank of the process in the expert parallel group.
    """
    is_unchanged = transfer_metadata.is_unchanged
    is_received_locally = transfer_metadata.is_received_locally
    recv_primary_mask = transfer_metadata.recv_primary_mask
    recv_count = transfer_metadata.recv_count
    recv_expert_ids = transfer_metadata.recv_expert_ids
    recv_dst_rows = transfer_metadata.recv_dst_rows
    num_local_experts = is_unchanged.shape[0]

    # Mask for rows to copy back from buffers:
    # copy if locally received OR remote primary recv
    copy_mask = np.logical_or(is_received_locally, recv_primary_mask)
    dest_mask_np = np.logical_and(~is_unchanged, copy_mask)
    if bool(dest_mask_np.any()):
        dest_indices = np.nonzero(dest_mask_np)[0].tolist()
        for dst in dest_indices:
            for w, b in zip(expert_weights, expert_weights_buffers):
                w[dst].copy_(b[dst], non_blocking=True)

    if recv_count == 0:
        return

    # Duplicate remote received rows to non-primary duplicate dsts
    base = ep_rank * num_local_experts
    local_experts = new_indices[base + np.arange(num_local_experts, dtype=np.int32)]
    duplicate_mask = np.logical_and(
# ... truncated for analysis ...
    )
    if not bool(valid.any()):
        return

    matched_dst_rows = dup_dst_rows[valid]
    matched_src_rows = prim_dsts_sorted[pos[valid]]

    for dst, src in zip(matched_dst_rows.tolist(), matched_src_rows.tolist()):
        for w in expert_weights:
            w[dst].copy_(w[src], non_blocking=True)
```
**EN:** `move_from_buffer` implements a focused helper routine for this module. The docstring frames it as: Copies expert weights from communication buffers back to the target weight tensors after EPLB rebalancing. It primarily works with arguments like `expert_weights`, `expert_weights_buffers`, `transfer_metadata`, `new_indices`. Key calls include `np.logical_or`, `np.logical_and`, `bool`.
**CN:** `move_from_buffer` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `expert_weights`, `expert_weights_buffers`, `transfer_metadata`, `new_indices` 这样的参数。 关键调用包括 `np.logical_or`, `np.logical_and`, `bool`。

### Function `transfer_layer` / 函数 `transfer_layer`
```python
def transfer_layer(
    old_layer_indices: torch.Tensor,
    new_layer_indices: torch.Tensor,
    expert_weights: Sequence[torch.Tensor],
    expert_weights_buffer: Sequence[torch.Tensor],
    ep_group: ProcessGroup,
    communicator: EplbCommunicator,
    is_profile: bool = False,
    cuda_stream: torch.cuda.Stream | None = None,
    rank_mapping: dict[int, int] | None = None,
) -> TransferMetadata:
    """
    Rearranges the expert weights in place according to the new expert indices.

    The value of the indices arguments are logical indices of the experts,
    while keys are physical.

    Args:
        old_layer_indices: Shape (num_physical_experts,).
        new_layer_indices: Shape (num_physical_experts,).
        expert_weights: Iterable of weight tensors for this layer, each with shape
            (num_local_physical_experts, hidden_size_i).
            For example, a linear layer may have up and down projection.
        expert_weights_buffer: Intermediate buffers (one per weight tensor).
        ep_group: The device process group for expert parallelism.
        communicator: EplbCommunicator instance for P2P communication.
        is_profile (bool): If `True`, do not perform any actual weight copy.
            This is used during profile run, where we only perform dummy
            communications to reserve enough memory for the buffers.
        cuda_stream: CUDA stream for async copies (can be None for sync mode).
        rank_mapping: Optional rank mapping for elastic expert parallelism.

    Returns:
        TransferMetadata: Metadata needed for completing remote weight transfers,
            including is_unchanged and is_received_locally masks.
    """
    ep_size = ep_group.size()
    if rank_mapping is not None:
        # Add a layer dimension for compatibility with mapping functions
        old_layer_indices_2d = old_layer_indices.unsqueeze(0)
        new_layer_indices_2d = new_layer_indices.unsqueeze(0)

        if len(rank_mapping) == ep_group.size():
            # scale down
            new_layer_indices_2d = _map_new_expert_indices_with_rank_mapping(
# ... truncated for analysis ...
    return move_to_buffer(
        num_local_experts=num_local_physical_experts,
        old_indices=old_layer_indices_np,
        new_indices=new_layer_indices_np,
        expert_weights=expert_weights,
        expert_weights_buffers=expert_weights_buffer,
        cuda_stream=cuda_stream,
        ep_rank=ep_group.rank(),
        communicator=communicator,
    )
```
**EN:** `transfer_layer` implements a focused helper routine for this module. The docstring frames it as: Rearranges the expert weights in place according to the new expert indices. It primarily works with arguments like `old_layer_indices`, `new_layer_indices`, `expert_weights`, `expert_weights_buffer`. Key calls include `ep_group.size`, `old_layer_indices.cpu().numpy`, `new_layer_indices.cpu().numpy`.
**CN:** `transfer_layer` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `old_layer_indices`, `new_layer_indices`, `expert_weights`, `expert_weights_buffer` 这样的参数。 关键调用包括 `ep_group.size`, `old_layer_indices.cpu().numpy`, `new_layer_indices.cpu().numpy`。

### Function `rearrange_expert_weights_inplace` / 函数 `rearrange_expert_weights_inplace`
```python
def rearrange_expert_weights_inplace(
    old_global_expert_indices: torch.Tensor,
    new_global_expert_indices: torch.Tensor,
    expert_weights: Sequence[Sequence[torch.Tensor]],
    ep_group: ProcessGroup,
    communicator: EplbCommunicator,
    is_profile: bool = False,
    rank_mapping: dict[int, int] | None = None,
) -> None:
    """
    Rearranges the expert weights in place according to the new expert indices.

    The value of the indices arguments are logical indices of the experts,
    while keys are physical.

    Args:
        old_global_expert_indices: Shape (num_moe_layers, num_physical_experts).
        new_global_expert_indices: Shape (num_moe_layers, num_physical_experts).
        expert_weights: A sequence of shape (num_moe_layers)(weight_count)
            of tensors of shape (num_local_physical_experts, hidden_size_i).
            For example, a linear layer may have up and down projection,
            so weight_count = 2. Each weight's hidden size can be different.
        ep_group: The device process group for expert parallelism.
        communicator: EplbCommunicator instance for P2P communication.
        is_profile (bool): If `True`, do not perform any actual weight copy.
            This is used during profile run, where we only perform dummy
            communications to reserve enough memory for the buffers.
        rank_mapping: A dictionary mapping old rank to new rank.
    """
    if rank_mapping is not None:
        if len(rank_mapping) == ep_group.size():
            # scale down
            new_global_expert_indices = _map_new_expert_indices_with_rank_mapping(
                new_global_expert_indices,
                rank_mapping,
            )
        else:
            # scale up
            old_global_expert_indices = _map_old_expert_indices_with_rank_mapping(
                old_global_expert_indices,
                rank_mapping,
                ep_group.size(),
            )

    assert old_global_expert_indices.shape[1] == new_global_expert_indices.shape[1]
# ... truncated for analysis ...
            communicator=communicator,
        )

        move_from_buffer(
            expert_weights=expert_weights[layer_idx],
            expert_weights_buffers=weights_buffer,
            transfer_metadata=transfer_metadata,
            new_indices=new_global_expert_indices_cpu[layer_idx],
            ep_rank=ep_rank,
        )
```
**EN:** `rearrange_expert_weights_inplace` implements a focused helper routine for this module. The docstring frames it as: Rearranges the expert weights in place according to the new expert indices. It primarily works with arguments like `old_global_expert_indices`, `new_global_expert_indices`, `expert_weights`, `ep_group`. Key calls include `ep_group.size`, `ep_group.rank`, `list`.
**CN:** `rearrange_expert_weights_inplace` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `old_global_expert_indices`, `new_global_expert_indices`, `expert_weights`, `ep_group` 这样的参数。 关键调用包括 `ep_group.size`, `ep_group.rank`, `list`。

### Function `_map_old_expert_indices_with_rank_mapping` / 函数 `_map_old_expert_indices_with_rank_mapping`
```python
def _map_old_expert_indices_with_rank_mapping(
    old_global_expert_indices: torch.Tensor,
    rank_mapping: dict[int, int],
    new_ep_size: int,
) -> torch.Tensor:
    """
    Map the old global expert indices to the new global expert indices.

    Args:
        old_global_expert_indices:
            Shape (num_layers, old_ep_size * num_local_physical_experts).
        rank_mapping: Mapping from old rank to new rank.
        new_ep_size: New expert parallelism size.

    Returns:
        Mapped expert indices with shape
        (num_layers, new_ep_size * num_local_physical_experts).
    """
    num_layers, old_num_physical_experts = old_global_expert_indices.shape
    assert rank_mapping, "Rank mapping is required"

    # Get sizes from parameters and rank_mapping
    old_ep_size = len(rank_mapping)
    num_local_physical_experts = old_num_physical_experts // old_ep_size
    new_num_physical_experts = new_ep_size * num_local_physical_experts

    # Create mapped tensor with new shape, initialized to -1
    mapped_expert_indices = torch.full(
        (num_layers, new_num_physical_experts),
        fill_value=-1,
        dtype=old_global_expert_indices.dtype,
        device=old_global_expert_indices.device,
    )

    # Handle rank mapping (scale up/down with rank changes)
    for old_rank in range(old_ep_size):
        new_rank = rank_mapping.get(old_rank)
        if new_rank is not None and new_rank >= 0 and new_rank < new_ep_size:
            # This old rank exists in the new configuration
            old_start_idx = old_rank * num_local_physical_experts
            old_end_idx = (old_rank + 1) * num_local_physical_experts
            new_start_idx = new_rank * num_local_physical_experts
            new_end_idx = (new_rank + 1) * num_local_physical_experts

            mapped_expert_indices[:, new_start_idx:new_end_idx] = (
                old_global_expert_indices[:, old_start_idx:old_end_idx]
            )
        # If new_rank is None or >= new_ep_size, the experts remain -1
        # (scale down case)

    return mapped_expert_indices
```
**EN:** `_map_old_expert_indices_with_rank_mapping` implements a focused helper routine for this module. The docstring frames it as: Map the old global expert indices to the new global expert indices. It primarily works with arguments like `old_global_expert_indices`, `rank_mapping`, `new_ep_size`. Key calls include `len`, `torch.full`, `range`.
**CN:** `_map_old_expert_indices_with_rank_mapping` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `old_global_expert_indices`, `rank_mapping`, `new_ep_size` 这样的参数。 关键调用包括 `len`, `torch.full`, `range`。

### Function `_map_new_expert_indices_with_rank_mapping` / 函数 `_map_new_expert_indices_with_rank_mapping`
```python
def _map_new_expert_indices_with_rank_mapping(
    new_global_expert_indices: torch.Tensor,
    rank_mapping: dict[int, int],
) -> torch.Tensor:
    num_layers, new_num_physical_experts = new_global_expert_indices.shape
    assert rank_mapping, "Rank mapping is required"

    # Get sizes from parameters and rank_mapping
    old_ep_size = len(rank_mapping)
    new_ep_size = sum(new_rank != -1 for new_rank in rank_mapping.values())
    num_local_physical_experts = new_num_physical_experts // new_ep_size
    old_num_physical_experts = old_ep_size * num_local_physical_experts

    mapped_expert_indices = torch.full(
        (num_layers, old_num_physical_experts),
        fill_value=-1,
        dtype=new_global_expert_indices.dtype,
        device=new_global_expert_indices.device,
    )

    for old_rank in range(old_ep_size):
        new_rank = rank_mapping[old_rank]
        if new_rank >= 0 and new_rank < new_ep_size:
            old_start_idx = old_rank * num_local_physical_experts
            old_end_idx = (old_rank + 1) * num_local_physical_experts
            new_start_idx = new_rank * num_local_physical_experts
            new_end_idx = (new_rank + 1) * num_local_physical_experts

            mapped_expert_indices[:, old_start_idx:old_end_idx] = (
                new_global_expert_indices[:, new_start_idx:new_end_idx]
            )

    return mapped_expert_indices
```
**EN:** `_map_new_expert_indices_with_rank_mapping` implements a focused helper routine for this module. It primarily works with arguments like `new_global_expert_indices`, `rank_mapping`. Key calls include `len`, `sum`, `torch.full`.
**CN:** `_map_new_expert_indices_with_rank_mapping` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `new_global_expert_indices`, `rank_mapping` 这样的参数。 关键调用包括 `len`, `sum`, `torch.full`。

### Module constants / 模块常量
```python
__all__ = ["transfer_layer", "move_from_buffer", "TransferMetadata"]
```
**EN:** This section defines module-level aliases, constants, or shared state such as `__all__`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `__all__`，供后续代码复用。

## Key Concepts / 关键概念
- Expert-parallel load balancing / 专家并行负载均衡
- `TransferMetadata`: dataclass interface or data carrier / `TransferMetadata`：数据类接口或数据载体
- `AsyncEplbLayerResult`: dataclass interface or data carrier / `AsyncEplbLayerResult`：数据类接口或数据载体
- `get_ep_ranks_with_experts_batch`: module-level helper or API entry / `get_ep_ranks_with_experts_batch`：模块级辅助函数或 API 入口
- `move_to_buffer`: module-level helper or API entry / `move_to_buffer`：模块级辅助函数或 API 入口
- `move_from_buffer`: module-level helper or API entry / `move_from_buffer`：模块级辅助函数或 API 入口

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `dataclasses`
- **Third-party / 第三方**: `numpy`, `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `vllm.distributed.eplb.eplb_communicator`, `vllm.distributed.eplb.eplb_utils`, `vllm.logger`
