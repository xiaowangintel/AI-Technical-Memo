# expert_location_updater.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/eplb/expert_location_updater.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the expert-parallel load balancing part of the SRT runtime and implements logic centered on `expert_location_updater`. It exposes primary entry points such as `ExpertLocationUpdater`, `_update_expert_weights`, `_update_expert_weights_with_canary`. / 该模块属于 SRT 运行时的专家并行负载均衡部分，主要实现围绕 `expert_location_updater` 的逻辑。 它对外提供的主要入口包括 `ExpertLocationUpdater`, `_update_expert_weights`, `_update_expert_weights_with_canary`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36: Module imports, constants, and setup
```python
# Copyright 2023-2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
import logging
from collections import defaultdict
from typing import Dict, List, Optional, Tuple

import einops
import torch
import torch.distributed
from torch.distributed import P2POp

from sglang.srt.elastic_ep.elastic_ep import ElasticEPStateManager
from sglang.srt.eplb.expert_location import (
    ExpertLocationMetadata,
    get_global_expert_location_metadata,
)
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import get_bool_env_var

logger = logging.getLogger(__name__)


_LOG_INPUT = get_bool_env_var("SGLANG_EXPERT_LOCATION_UPDATER_LOG_INPUT")


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 37-37: Class ExpertLocationUpdater
```python
class ExpertLocationUpdater:
```
**EN:** This range introduces `ExpertLocationUpdater` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ExpertLocationUpdater`，并定义其后续方法依赖的结构或元数据。

### Lines 38-39: Method ExpertLocationUpdater.__init__
```python
    def __init__(self):
        self._first_execution = True
```
**EN:** This callable implements `ExpertLocationUpdater.__init__` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `ExpertLocationUpdater.__init__`，主要用于初始化实例状态与默认值。

### Lines 41-75: Method ExpertLocationUpdater.update
```python
    def update(
        self,
        routed_experts_weights_of_layer: Dict[int, List[torch.Tensor]],
        new_expert_location_metadata: ExpertLocationMetadata,
        update_layer_ids: List[int],
        nnodes: int,
        rank: int,
    ):
        """
        Update experts' physical location after EPLB.

        Returns a map of layer_id to expert_ids that are missing due to rank
        failures during fault conditions when elastic EP is enabled.
        """
        if self._first_execution:
            self._first_execution = False
            torch.get_device_module().empty_cache()

        old_expert_location_metadata = get_global_expert_location_metadata()
        assert old_expert_location_metadata is not None

        missing_logical_experts_by_layers = _update_expert_weights(
            routed_experts_weights_of_layer=routed_experts_weights_of_layer,
            old_expert_location_metadata=old_expert_location_metadata,
            new_expert_location_metadata=new_expert_location_metadata,
            update_layer_ids=update_layer_ids,
            nnodes=nnodes,
            rank=rank,
        )
        old_expert_location_metadata.update(
            new_expert_location_metadata,
            update_layer_ids=update_layer_ids,
        )

        return missing_logical_experts_by_layers
```
**EN:** This callable implements `ExpertLocationUpdater.update`. It takes `routed_experts_weights_of_layer`, `new_expert_location_metadata`, `update_layer_ids`, `nnodes` and mainly updates existing runtime state. The docstring states: "Update experts' physical location after EPLB." In this range it performs defensive checks on invalid state; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `ExpertLocationUpdater.update`。它接收 `routed_experts_weights_of_layer`, `new_expert_location_metadata`, `update_layer_ids`, `nnodes`，主要用于更新现有运行时状态。 在这一范围内，它会对非法状态执行防御性检查；管理模型权重或检查点。

### Lines 78-82: Function _update_expert_weights
```python
def _update_expert_weights(**kwargs):
    if get_bool_env_var("SGLANG_EXPERT_LOCATION_UPDATER_CANARY"):
        return _update_expert_weights_with_canary(**kwargs)
    else:
        return _update_expert_weights_raw(**kwargs)
```
**EN:** This callable implements `_update_expert_weights`. It takes `**kwargs` and mainly updates existing runtime state. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `_update_expert_weights`。它接收 `**kwargs`，主要用于更新现有运行时状态。 在这一范围内，它会管理模型权重或检查点。

### Lines 83-85: Module-level logic
```python


# can add watchdog as well
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 86-132: Function _update_expert_weights_with_canary
```python
def _update_expert_weights_with_canary(
    routed_experts_weights_of_layer: Dict[int, List[torch.Tensor]],
    old_expert_location_metadata: ExpertLocationMetadata,
    new_expert_location_metadata: ExpertLocationMetadata,
    update_layer_ids: List[int],
    nnodes: int,
    rank: int,
):
    num_local_physical_experts = old_expert_location_metadata.num_local_physical_experts

    def _get_canary_value(meta: ExpertLocationMetadata, layer_id: int):
        return meta.physical_to_logical_map_cpu[
            layer_id,
            num_local_physical_experts * rank : num_local_physical_experts * (rank + 1),
        ]

    routed_experts_weights_of_layer = {
        k: [x for x in v] for k, v in routed_experts_weights_of_layer.items()
    }
    for layer_id in update_layer_ids:
        canary_tensor = (
            _get_canary_value(old_expert_location_metadata, layer_id)
            .clone()
            .to(device=get_global_server_args().device, non_blocking=True)
        )
        routed_experts_weights_of_layer[layer_id].append(canary_tensor)

    missing_logical_experts_by_layers = _update_expert_weights_raw(
        routed_experts_weights_of_layer=routed_experts_weights_of_layer,
        old_expert_location_metadata=old_expert_location_metadata,
        new_expert_location_metadata=new_expert_location_metadata,
        update_layer_ids=update_layer_ids,
        nnodes=nnodes,
        rank=rank,
    )

    for layer_id in update_layer_ids:
        # can optimize speed if needed
        expect_value = _get_canary_value(new_expert_location_metadata, layer_id)
        actual_value = routed_experts_weights_of_layer[layer_id][-1].cpu()
        assert torch.all(expect_value == actual_value), (
            f"{expect_value=} {actual_value=} {layer_id=} "
            f"{old_expert_location_metadata.physical_to_logical_map_cpu.tolist()=} "
            f"{new_expert_location_metadata.physical_to_logical_map_cpu.tolist()=} "
        )

    return missing_logical_experts_by_layers
```
**EN:** This callable implements `_update_expert_weights_with_canary`. It takes `routed_experts_weights_of_layer`, `old_expert_location_metadata`, `new_expert_location_metadata`, `update_layer_ids` and mainly updates existing runtime state. In this range it performs defensive checks on invalid state; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `_update_expert_weights_with_canary`。它接收 `routed_experts_weights_of_layer`, `old_expert_location_metadata`, `new_expert_location_metadata`, `update_layer_ids`，主要用于更新现有运行时状态。 在这一范围内，它会对非法状态执行防御性检查；管理模型权重或检查点。

### Lines 135-175: Function _update_expert_weights_raw
```python
def _update_expert_weights_raw(
    routed_experts_weights_of_layer: Dict[int, List[torch.Tensor]],
    old_expert_location_metadata: ExpertLocationMetadata,
    new_expert_location_metadata: ExpertLocationMetadata,
    update_layer_ids: List[int],
    nnodes: int,
    rank: int,
):
    log_metrics = get_bool_env_var("SGLANG_EXPERT_LOCATION_UPDATER_LOG_METRICS")

    temp_buffers = create_temp_buffers(
        routed_experts_weights_of_layer[update_layer_ids[0]]
    )

    world_size = torch.distributed.get_world_size()
    num_local_physical_experts = old_expert_location_metadata.num_local_physical_experts
    num_gpu_per_node = world_size // nnodes

    missing_logical_experts_by_layers: Dict[int, List[int]] = {}

    for layer_id in update_layer_ids:
        missing_logical_experts_info: List[int] = []
        update_expert_weights_single_layer(
            routed_experts_weights=routed_experts_weights_of_layer[layer_id],
            temp_buffers=temp_buffers,
            old_physical_to_logical_map=old_expert_location_metadata.physical_to_logical_map_cpu[
                layer_id
            ].tolist(),
            new_physical_to_logical_map=new_expert_location_metadata.physical_to_logical_map_cpu[
                layer_id
            ].tolist(),
            num_local_physical_experts=num_local_physical_experts,
            num_gpu_per_node=num_gpu_per_node,
            rank=rank,
            world_size=world_size,
            missing_logical_experts_info=missing_logical_experts_info,
            log_metrics=log_metrics,
        )
        if len(missing_logical_experts_info) > 0:
            missing_logical_experts_by_layers[layer_id] = missing_logical_experts_info
    return missing_logical_experts_by_layers
```
**EN:** This callable implements `_update_expert_weights_raw`. It takes `routed_experts_weights_of_layer`, `old_expert_location_metadata`, `new_expert_location_metadata`, `update_layer_ids` and mainly updates existing runtime state. In this range it coordinates distributed communication; records metrics or tracing signals; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `_update_expert_weights_raw`。它接收 `routed_experts_weights_of_layer`, `old_expert_location_metadata`, `new_expert_location_metadata`, `update_layer_ids`，主要用于更新现有运行时状态。 在这一范围内，它会协调分布式通信；记录指标或追踪信号；管理模型权重或检查点。

### Lines 178-179: Function create_temp_buffers
```python
def create_temp_buffers(sample_tensors):
    return [torch.empty_like(tensor) for tensor in sample_tensors]
```
**EN:** This callable implements `create_temp_buffers`. It takes `sample_tensors` and mainly constructs new objects or contexts.
**CN:** 这一可调用对象实现了 `create_temp_buffers`。它接收 `sample_tensors`，主要用于构造新的对象或上下文。

### Lines 182-241: Function update_expert_weights_single_layer (part 1/6)
```python
def update_expert_weights_single_layer(
    routed_experts_weights: List[torch.Tensor],
    temp_buffers: List[torch.Tensor],
    old_physical_to_logical_map: List[int],  # (num_physical_Experts,)
    new_physical_to_logical_map: List[int],  # (num_physical_Experts,)
    num_local_physical_experts: int,
    num_gpu_per_node: int,
    rank: int,
    world_size: Optional[int] = None,
    missing_logical_experts_info: Optional[List[int]] = None,
    debug: bool = False,
    log_metrics: bool = False,
):
    assert all(
        tensor.shape[0] == num_local_physical_experts
        for tensor in routed_experts_weights
    ), f"{num_local_physical_experts=} {[x.shape for x in routed_experts_weights]=}"
    assert isinstance(old_physical_to_logical_map, list)
    assert isinstance(new_physical_to_logical_map, list)

    if _LOG_INPUT:
        logger.info(
            "update_expert_weights_single_layer "
            f"{[x.shape for x in routed_experts_weights]=} "
            f"{[x.shape for x in temp_buffers]=} "
            f"{old_physical_to_logical_map=} "
            f"{new_physical_to_logical_map=} "
            f"{num_local_physical_experts=} "
            f"{num_gpu_per_node=} "
            f"{rank=} "
            f"{world_size=} "
        )

    output_logs = [] if debug else None

    num_physical_experts = len(old_physical_to_logical_map)
    num_tensors = len(routed_experts_weights)

    self_node_id = rank // num_gpu_per_node

    local_expert_location_range = (
        rank * num_local_physical_experts,
        (rank + 1) * num_local_physical_experts,
    )

    def _entrypoint():
        # List[Tuple[logical_expert_id, List[P2POp]]]
        p2p_op_infos: List[Tuple[int, List[P2POp]]] = []
        # List[Tuple[temp_buffers_expert_location, routed_experts_weights_expert_location]]
        buffer2weight_copy_infos: List[Tuple[int, int]] = []

        _handle_recv(buffer2weight_copy_infos, p2p_op_infos)
        _create_isend_ops(p2p_op_infos)
        _filter_p2p_ops(p2p_op_infos)
        _execute_p2p_ops(p2p_op_infos)
        _execute_buffer2weight_copies(buffer2weight_copy_infos)

        if log_metrics:
            _log_p2p_op_metrics(
                p2p_op_infos,
```
**EN:** This callable implements `update_expert_weights_single_layer`. It takes `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map` and mainly updates existing runtime state. This chunk is part 1 of 6 for the same logical block. In this range it performs defensive checks on invalid state; emits logs for diagnostics; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `update_expert_weights_single_layer`。它接收 `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map`，主要用于更新现有运行时状态。 该片段是同一逻辑块的第 1/6 部分。 在这一范围内，它会对非法状态执行防御性检查；输出日志以便诊断；记录指标或追踪信号。

### Lines 242-301: Function update_expert_weights_single_layer (part 2/6)
```python
                world_size=world_size,
                num_gpu_per_node=num_gpu_per_node,
                self_node_id=self_node_id,
            )

        if debug:
            output_logs.append(f"{p2p_op_infos=}")
            output_logs.append(f"{buffer2weight_copy_infos=}")

    def _handle_recv(buffer2weight_copy_infos, p2p_op_infos):
        for dst_expert_location in range(*local_expert_location_range):
            _handle_recv_of_dst_expert_location(
                dst_expert_location, buffer2weight_copy_infos, p2p_op_infos
            )

    def _handle_recv_of_dst_expert_location(
        dst_expert_location: int, buffer2weight_copy_infos, p2p_op_infos
    ):
        logical_expert_id = new_physical_to_logical_map[dst_expert_location]

        # case 1: unchanged
        if old_physical_to_logical_map[dst_expert_location] == logical_expert_id:
            if debug:
                output_logs.append(
                    f"handle_recv_of_dst_expert_location {dst_expert_location=} case=unchanged"
                )
            return

        # case 2: same-gpu
        for src_expert_location in range(*local_expert_location_range):
            if old_physical_to_logical_map[src_expert_location] == logical_expert_id:
                for i in range(num_tensors):
                    _get_tensor(temp_buffers, i, dst_expert_location).copy_(
                        _get_tensor(routed_experts_weights, i, src_expert_location)
                    )
                buffer2weight_copy_infos.append(
                    (dst_expert_location, dst_expert_location)
                )
                if debug:
                    output_logs.append(
                        f"handle_recv_of_dst_expert_location {dst_expert_location=} case=same-gpu {src_expert_location=}"
                    )
                return

        # case 3: free-rider
        for src_expert_location in range(
            rank * num_local_physical_experts, dst_expert_location
        ):
            if new_physical_to_logical_map[src_expert_location] == logical_expert_id:
                buffer2weight_copy_infos.append(
                    (src_expert_location, dst_expert_location)
                )
                if debug:
                    output_logs.append(
                        f"handle_recv_of_dst_expert_location {dst_expert_location=} case=free-rider {src_expert_location=}"
                    )
                return

        same_node_mapping, cross_node_mapping, need_comm_self_node_dst_ranks = (
            _compute_comm_info(logical_expert_id=logical_expert_id)
```
**EN:** This callable implements `update_expert_weights_single_layer`. It takes `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map` and mainly updates existing runtime state. This chunk is part 2 of 6 for the same logical block. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `update_expert_weights_single_layer`。它接收 `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map`，主要用于更新现有运行时状态。 该片段是同一逻辑块的第 2/6 部分。 在这一范围内，它会管理模型权重或检查点。

### Lines 302-361: Function update_expert_weights_single_layer (part 3/6)
```python
        )

        # case 4: same-node
        if rank in need_comm_self_node_dst_ranks:
            chosen_src_rank = same_node_mapping.chunk_value_from_element_value(
                element_value=rank
            )
            _create_p2p_recv_and_buffer2weight_copy(
                buffer2weight_copy_infos,
                p2p_op_infos,
                src_rank=chosen_src_rank,
                logical_expert_id=logical_expert_id,
                dst_expert_location=dst_expert_location,
            )
            if debug:
                output_logs.append(
                    f"handle_recv_of_dst_expert_location {dst_expert_location=} case=same-node {chosen_src_rank=}"
                )
            return

        # case 5: cross-node
        # Future work: can optimize when there are multiple ranks in the same dst node that uses the same logical expert
        chosen_src_rank = cross_node_mapping.chunk_value_from_element_value(
            element_value=rank
        )
        _create_p2p_recv_and_buffer2weight_copy(
            buffer2weight_copy_infos,
            p2p_op_infos,
            src_rank=chosen_src_rank,
            logical_expert_id=logical_expert_id,
            dst_expert_location=dst_expert_location,
        )
        if debug:
            output_logs.append(
                f"handle_recv_of_dst_expert_location {dst_expert_location=} case=cross-node {chosen_src_rank=}"
            )
        return

    def _create_p2p_recv_and_buffer2weight_copy(
        buffer2weight_copy_infos,
        p2p_op_infos,
        *,
        logical_expert_id: int,
        src_rank: int,
        dst_expert_location: int,
    ):
        p2p_op_infos.append(
            (
                logical_expert_id,
                [
                    P2POp(
                        op=torch.distributed.irecv,
                        tensor=_get_tensor(temp_buffers, i, dst_expert_location),
                        peer=src_rank,
                    )
                    for i in range(num_tensors)
                ],
            )
        )
        buffer2weight_copy_infos.append((dst_expert_location, dst_expert_location))
```
**EN:** This callable implements `update_expert_weights_single_layer`. It takes `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map` and mainly updates existing runtime state. This chunk is part 3 of 6 for the same logical block. In this range it coordinates distributed communication; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `update_expert_weights_single_layer`。它接收 `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map`，主要用于更新现有运行时状态。 该片段是同一逻辑块的第 3/6 部分。 在这一范围内，它会协调分布式通信；管理模型权重或检查点。

### Lines 362-421: Function update_expert_weights_single_layer (part 4/6)
```python

    def _create_isend_ops(p2p_op_infos):
        handled_logical_expert_ids = set()
        for src_expert_location in range(*local_expert_location_range):
            logical_expert_id = old_physical_to_logical_map[src_expert_location]

            if logical_expert_id in handled_logical_expert_ids:
                continue
            handled_logical_expert_ids.add(logical_expert_id)

            _create_isend_ops_of_logical_expert_id(
                logical_expert_id, src_expert_location, p2p_op_infos
            )

    def _create_isend_ops_of_logical_expert_id(
        logical_expert_id, src_expert_location, p2p_op_infos
    ):
        same_node_mapping, cross_node_mapping, need_comm_self_node_dst_ranks = (
            _compute_comm_info(logical_expert_id=logical_expert_id)
        )

        same_node_dst_ranks = same_node_mapping.element_values_from_chunk_value(
            chunk_value=rank
        )
        cross_node_dst_ranks = cross_node_mapping.element_values_from_chunk_value(
            chunk_value=rank
        )
        all_dst_ranks = same_node_dst_ranks + cross_node_dst_ranks

        if debug:
            output_logs.append(
                f"create_isend_ops_of_logical_expert_id {logical_expert_id=} {src_expert_location=} {same_node_dst_ranks=} {cross_node_dst_ranks=}"
            )

        p2p_op_infos.append(
            (
                logical_expert_id,
                [
                    P2POp(
                        op=torch.distributed.isend,
                        tensor=_get_tensor(
                            routed_experts_weights, i, src_expert_location
                        ),
                        peer=dst_rank,
                    )
                    for dst_rank in all_dst_ranks
                    for i in range(num_tensors)
                ],
            )
        )

    def _compute_comm_info(logical_expert_id: int):
        all_src_ranks = _deduplicate_ordered(
            [
                x // num_local_physical_experts
                for x in range(num_physical_experts)
                if old_physical_to_logical_map[x] == logical_expert_id
            ]
        )
        all_src_nodes = [x // num_gpu_per_node for x in all_src_ranks]
```
**EN:** This callable implements `update_expert_weights_single_layer`. It takes `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map` and mainly updates existing runtime state. This chunk is part 4 of 6 for the same logical block. In this range it coordinates distributed communication; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `update_expert_weights_single_layer`。它接收 `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map`，主要用于更新现有运行时状态。 该片段是同一逻辑块的第 4/6 部分。 在这一范围内，它会协调分布式通信；管理模型权重或检查点。

### Lines 422-481: Function update_expert_weights_single_layer (part 5/6)
```python
        self_node_src_ranks = [
            x for x in all_src_ranks if x // num_gpu_per_node == self_node_id
        ]

        need_comm_dst_ranks = _deduplicate_ordered(
            [
                x // num_local_physical_experts
                for x in range(num_physical_experts)
                if new_physical_to_logical_map[x] == logical_expert_id
                and x // num_local_physical_experts not in all_src_ranks
            ]
        )
        need_comm_self_node_dst_ranks = (
            [x for x in need_comm_dst_ranks if x // num_gpu_per_node == self_node_id]
            if len(self_node_src_ranks) > 0
            else []
        )
        need_comm_cross_node_dst_ranks = [
            x
            for x in need_comm_dst_ranks
            if (x // num_gpu_per_node) not in all_src_nodes
        ]

        same_node_mapping = _ChunkUtils(
            chunk_values=self_node_src_ranks,
            element_values=need_comm_self_node_dst_ranks,
        )

        cross_node_mapping = _ChunkUtils(
            chunk_values=all_src_ranks,
            element_values=need_comm_cross_node_dst_ranks,
        )

        return same_node_mapping, cross_node_mapping, need_comm_self_node_dst_ranks

    def _filter_p2p_ops(p2p_op_infos):
        elastic_ep_state = ElasticEPStateManager.instance()
        if elastic_ep_state is not None and missing_logical_experts_info is not None:
            # Filter out inactive P2P ops and record missing expert IDs in missing_logical_experts_info
            is_active = elastic_ep_state.active_ranks_cpu
            for i, (logical_expert_id, ops) in enumerate(p2p_op_infos):
                has_isend = any(op.op == torch.distributed.isend for op in ops)
                has_irecv = any(op.op == torch.distributed.irecv for op in ops)
                assert not (has_isend and has_irecv), (
                    "Each p2p_op_infos entry is expected to contain only send "
                    "or only recv ops."
                )

                if has_isend:
                    p2p_op_infos[i] = (
                        logical_expert_id,
                        [op for op in ops if is_active[op.peer]],
                    )
                elif has_irecv:
                    if any(not is_active[op.peer] for op in ops):
                        missing_logical_experts_info.append(logical_expert_id)
                        p2p_op_infos[i] = (logical_expert_id, [])

    def _execute_p2p_ops(p2p_op_infos):
        sorted_infos = sorted(p2p_op_infos, key=lambda info: info[0])
```
**EN:** This callable implements `update_expert_weights_single_layer`. It takes `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map` and mainly updates existing runtime state. This chunk is part 5 of 6 for the same logical block. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `update_expert_weights_single_layer`。它接收 `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map`，主要用于更新现有运行时状态。 该片段是同一逻辑块的第 5/6 部分。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 482-513: Function update_expert_weights_single_layer (part 6/6)
```python
        p2p_ops = [op for _, ops in sorted_infos for op in ops]
        if len(p2p_ops) == 0:
            return

        reqs = torch.distributed.batch_isend_irecv(p2p_ops)
        for req in reqs:
            req.wait()

    def _execute_buffer2weight_copies(buffer2weight_copy_infos):
        for (
            temp_buffers_expert_location,
            routed_experts_weights_expert_location,
        ) in buffer2weight_copy_infos:
            for i in range(num_tensors):
                _get_tensor(
                    routed_experts_weights, i, routed_experts_weights_expert_location
                ).copy_(_get_tensor(temp_buffers, i, temp_buffers_expert_location))

    def _get_tensor(tensors, tensor_index: int, expert_location: int) -> torch.Tensor:
        return tensors[tensor_index][_get_local_expert_location(expert_location)]

    def _get_local_expert_location(expert_location: int) -> int:
        assert (
            local_expert_location_range[0]
            <= expert_location
            < local_expert_location_range[1]
        )
        return expert_location % num_local_physical_experts

    _entrypoint()

    return output_logs
```
**EN:** This callable implements `update_expert_weights_single_layer`. It takes `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map` and mainly updates existing runtime state. This chunk is part 6 of 6 for the same logical block. In this range it performs defensive checks on invalid state; coordinates distributed communication; manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `update_expert_weights_single_layer`。它接收 `routed_experts_weights`, `temp_buffers`, `old_physical_to_logical_map`, `new_physical_to_logical_map`，主要用于更新现有运行时状态。 该片段是同一逻辑块的第 6/6 部分。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信；管理模型权重或检查点。

### Lines 516-516: Class _ChunkUtils
```python
class _ChunkUtils:
```
**EN:** This range introduces `_ChunkUtils` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `_ChunkUtils`，并定义其后续方法依赖的结构或元数据。

### Lines 517-519: Method _ChunkUtils.__init__
```python
    def __init__(self, *, chunk_values: List, element_values: List):
        self.chunk_values = chunk_values
        self.element_values = element_values
```
**EN:** This callable implements `_ChunkUtils.__init__` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `_ChunkUtils.__init__`，主要用于初始化实例状态与默认值。

### Lines 521-527: Method _ChunkUtils.chunk_value_from_element_value
```python
    def chunk_value_from_element_value(self, element_value):
        chunk_index = self._chunk_index_from_element_index(
            num_elements=len(self.element_values),
            num_chunks=len(self.chunk_values),
            element_index=self.element_values.index(element_value),
        )
        return self.chunk_values[chunk_index]
```
**EN:** This callable implements `_ChunkUtils.chunk_value_from_element_value`. It takes `element_value` and mainly constructs data from an external representation.
**CN:** 这一可调用对象实现了 `_ChunkUtils.chunk_value_from_element_value`。它接收 `element_value`，主要用于从外部表示构造数据。

### Lines 529-537: Method _ChunkUtils.element_values_from_chunk_value
```python
    def element_values_from_chunk_value(self, chunk_value) -> List:
        if len(self.element_values) == 0:
            return []
        element_slice = self._element_slice_from_chunk_index(
            num_elements=len(self.element_values),
            num_chunks=len(self.chunk_values),
            chunk_index=self.chunk_values.index(chunk_value),
        )
        return self.element_values[element_slice]
```
**EN:** This callable implements `_ChunkUtils.element_values_from_chunk_value`. It takes `chunk_value` and mainly constructs data from an external representation.
**CN:** 这一可调用对象实现了 `_ChunkUtils.element_values_from_chunk_value`。它接收 `chunk_value`，主要用于从外部表示构造数据。

### Lines 539-551: Method _ChunkUtils._chunk_index_from_element_index
```python
    @staticmethod
    def _chunk_index_from_element_index(
        num_elements: int, num_chunks: int, element_index: int
    ) -> int:
        short_chunk_size, num_long_chunks = divmod(num_elements, num_chunks)
        num_elements_for_long_chunks = num_long_chunks * (short_chunk_size + 1)
        if element_index < num_elements_for_long_chunks:
            return element_index // (short_chunk_size + 1)
        else:
            return (
                num_long_chunks
                + (element_index - num_elements_for_long_chunks) // short_chunk_size
            )
```
**EN:** This callable implements `_ChunkUtils._chunk_index_from_element_index`. It takes `num_elements`, `num_chunks`, `element_index` and mainly constructs data from an external representation.
**CN:** 这一可调用对象实现了 `_ChunkUtils._chunk_index_from_element_index`。它接收 `num_elements`, `num_chunks`, `element_index`，主要用于从外部表示构造数据。

### Lines 553-560: Method _ChunkUtils._element_slice_from_chunk_index
```python
    @staticmethod
    def _element_slice_from_chunk_index(
        num_elements: int, num_chunks: int, chunk_index: int
    ) -> slice:
        short_chunk_size, num_long_chunks = divmod(num_elements, num_chunks)
        start = chunk_index * short_chunk_size + min(chunk_index, num_long_chunks)
        end = start + short_chunk_size + int(chunk_index < num_long_chunks)
        return slice(start, end)
```
**EN:** This callable implements `_ChunkUtils._element_slice_from_chunk_index`. It takes `num_elements`, `num_chunks`, `chunk_index` and mainly constructs data from an external representation.
**CN:** 这一可调用对象实现了 `_ChunkUtils._element_slice_from_chunk_index`。它接收 `num_elements`, `num_chunks`, `chunk_index`，主要用于从外部表示构造数据。

### Lines 563-568: Function _deduplicate_ordered
```python
def _deduplicate_ordered(arr: List[int]):
    output = []
    for item in arr:
        if len(output) == 0 or item != output[-1]:
            output.append(item)
    return output
```
**EN:** This callable implements `_deduplicate_ordered`. It takes `arr` and mainly implements deduplicate ordered.
**CN:** 这一可调用对象实现了 `_deduplicate_ordered`。它接收 `arr`，主要用于实现 deduplicate ordered 相关逻辑。

### Lines 571-603: Function _log_p2p_op_metrics
```python
def _log_p2p_op_metrics(
    p2p_op_infos: List[Tuple[int, List[P2POp]]],
    num_gpu_per_node: int,
    world_size: int,
    self_node_id: int,
):
    text = ""
    all_ops = [op for _, ops in p2p_op_infos for op in ops]

    for direction, ops in _group_by(all_ops, _get_direction_from_op).items():
        nbytes_of_gpu = [0] * world_size
        for op in ops:
            nbytes_of_gpu[op.peer] += op.tensor.nbytes
        nbytes_of_gpu = torch.tensor(nbytes_of_gpu, dtype=torch.int64)

        nbytes_of_node = einops.reduce(
            nbytes_of_gpu,
            "(num_nodes num_gpu_per_node) -> num_nodes",
            num_gpu_per_node=num_gpu_per_node,
            reduction="sum",
        )

        nbytes_curr_node = nbytes_of_node[self_node_id]
        nbytes_cross_node = torch.sum(nbytes_of_node) - nbytes_curr_node

        text += (
            f"{direction}_nbytes_of_gpu={nbytes_of_gpu.tolist()} "
            f"{direction}_nbytes_of_node={nbytes_of_node.tolist()} "
            f"{direction}_nbytes_curr_node={nbytes_curr_node.item()} "
            f"{direction}_nbytes_cross_node={nbytes_cross_node.item()} "
        )

    logger.info(f"[ExpertLocationUpdater] {text}")
```
**EN:** This callable implements `_log_p2p_op_metrics`. It takes `p2p_op_infos`, `num_gpu_per_node`, `world_size`, `self_node_id` and mainly implements log p2p op metrics. In this range it emits logs for diagnostics; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `_log_p2p_op_metrics`。它接收 `p2p_op_infos`, `num_gpu_per_node`, `world_size`, `self_node_id`，主要用于实现 log p2p op metrics 相关逻辑。 在这一范围内，它会输出日志以便诊断；记录指标或追踪信号。

### Lines 606-611: Function _get_direction_from_op
```python
def _get_direction_from_op(op: P2POp):
    if op.op == torch.distributed.isend:
        return "isend"
    if op.op == torch.distributed.irecv:
        return "irecv"
    raise NotImplementedError
```
**EN:** This callable implements `_get_direction_from_op`. It takes `op` and mainly constructs data from an external representation. In this range it performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `_get_direction_from_op`。它接收 `op`，主要用于从外部表示构造数据。 在这一范围内，它会对非法状态执行防御性检查；协调分布式通信。

### Lines 614-618: Function _group_by
```python
def _group_by(items, keyfunc):
    ans = defaultdict(list)
    for item in items:
        ans[keyfunc(item)].append(item)
    return dict(ans)
```
**EN:** This callable implements `_group_by`. It takes `items`, `keyfunc` and mainly implements group by.
**CN:** 这一可调用对象实现了 `_group_by`。它接收 `items`, `keyfunc`，主要用于实现 group by 相关逻辑。

## Key Concepts / 关键概念
- `_LOG_INPUT`: module constant or capability flag / 模块常量或能力标记
- `ExpertLocationUpdater`: core class or state container / 核心类或状态容器
- `_update_expert_weights`: updates existing runtime state / 更新现有运行时状态
- `_update_expert_weights_with_canary`: updates existing runtime state / 更新现有运行时状态
- `_update_expert_weights_raw`: updates existing runtime state / 更新现有运行时状态
- `create_temp_buffers`: constructs new objects or contexts / 构造新的对象或上下文
- `update_expert_weights_single_layer`: updates existing runtime state / 更新现有运行时状态
- `_ChunkUtils`: core class or state container / 核心类或状态容器
- `_deduplicate_ordered`: implements deduplicate ordered / 实现 deduplicate ordered 相关逻辑
- `_log_p2p_op_metrics`: implements log p2p op metrics / 实现 log p2p op metrics 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `collections`, `typing`
- **Third-party / 第三方**: `einops`, `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.srt.elastic_ep.elastic_ep`, `sglang.srt.eplb.expert_location`, `sglang.srt.server_args`, `sglang.srt.utils`
