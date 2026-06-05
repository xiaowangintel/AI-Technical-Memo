# expert_location.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/eplb/expert_location.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the expert-parallel load balancing part of the SRT runtime and implements logic centered on `expert_location`. It exposes primary entry points such as `ExpertLocationMetadata`, `get_global_expert_location_metadata`, `set_global_expert_location_metadata`. / 该模块属于 SRT 运行时的专家并行负载均衡部分，主要实现围绕 `expert_location` 的逻辑。 它对外提供的主要入口包括 `ExpertLocationMetadata`, `get_global_expert_location_metadata`, `set_global_expert_location_metadata`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: Module imports, constants, and setup
```python
# Copyright 2023-2024 SGLang Team
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

from __future__ import annotations

import json
import logging
import random
from dataclasses import dataclass
from pathlib import Path
from typing import TYPE_CHECKING, List, Optional

import torch
import torch.distributed
import torch.nn.functional as F

if TYPE_CHECKING:
    from sglang.srt.configs.model_config import ModelConfig
    from sglang.srt.server_args import ServerArgs

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 35-46: Class ExpertLocationMetadata
```python
@dataclass
class ExpertLocationMetadata:
    physical_to_logical_map: torch.Tensor  # (layers, num_physical_experts)
    physical_to_logical_map_cpu: torch.Tensor
    logical_to_all_physical_map: torch.Tensor  # (layers, num_logical_experts, X)
    logical_to_all_physical_map_cpu: torch.Tensor  # CPU copy for performance
    logical_to_all_physical_map_num_valid: torch.Tensor  # (layers, num_logical_experts)
    # (layers, num_logical_experts)
    logical_to_rank_dispatch_physical_map: Optional[torch.Tensor]

    # -------------------------------- properties ------------------------------------

```
**EN:** This range introduces `ExpertLocationMetadata` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ExpertLocationMetadata`，并定义其后续方法依赖的结构或元数据。

### Lines 47-49: Method ExpertLocationMetadata.num_layers
```python
    @property
    def num_layers(self) -> int:
        return self.physical_to_logical_map.shape[0]
```
**EN:** This callable implements `ExpertLocationMetadata.num_layers` and mainly implements num layers.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata.num_layers`，主要用于实现 num layers 相关逻辑。

### Lines 51-53: Method ExpertLocationMetadata.num_physical_experts
```python
    @property
    def num_physical_experts(self) -> int:
        return self.physical_to_logical_map.shape[1]
```
**EN:** This callable implements `ExpertLocationMetadata.num_physical_experts` and mainly implements num physical experts.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata.num_physical_experts`，主要用于实现 num physical experts 相关逻辑。

### Lines 55-59: Method ExpertLocationMetadata.num_local_physical_experts
```python
    @property
    def num_local_physical_experts(self) -> int:
        ans, remainder = divmod(self.num_physical_experts, self.ep_size)
        assert remainder == 0
        return ans
```
**EN:** This callable implements `ExpertLocationMetadata.num_local_physical_experts` and mainly implements num local physical experts. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata.num_local_physical_experts`，主要用于实现 num local physical experts 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 61-63: Method ExpertLocationMetadata.num_logical_experts
```python
    @property
    def num_logical_experts(self) -> int:
        return self.logical_to_all_physical_map.shape[1]
```
**EN:** This callable implements `ExpertLocationMetadata.num_logical_experts` and mainly implements num logical experts.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata.num_logical_experts`，主要用于实现 num logical experts 相关逻辑。

### Lines 65-68: Method ExpertLocationMetadata.ep_size
```python
    @property
    def ep_size(self):
        # TODO change when EP size != world size
        return torch.distributed.get_world_size()
```
**EN:** This callable implements `ExpertLocationMetadata.ep_size` and mainly implements ep size. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata.ep_size`，主要用于实现 ep size 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 70-80: Method ExpertLocationMetadata.__post_init__
```python
    def __post_init__(self):
        num_layers_0, num_physical_experts_0 = self.physical_to_logical_map.shape
        num_layers_1, num_logical_experts_0, num_physical_experts_1 = (
            self.logical_to_all_physical_map.shape
        )
        num_layers_2, num_logical_experts_1 = (
            self.logical_to_all_physical_map_num_valid.shape
        )
        assert num_layers_0 == num_layers_1 == num_layers_2
        assert num_logical_experts_0 == num_logical_experts_1
        assert num_physical_experts_0 == num_physical_experts_1
```
**EN:** This callable implements `ExpertLocationMetadata.__post_init__` and mainly implements post init. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata.__post_init__`，主要用于实现 post init 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 81-83: Class-level scaffolding for ExpertLocationMetadata
```python

    # -------------------------------- construction ------------------------------------

```
**EN:** This callable implements `None.ExpertLocationMetadata` and mainly implements Expert Location Metadata.
**CN:** 这一可调用对象实现了 `None.ExpertLocationMetadata`，主要用于实现 Expert Location Metadata 相关逻辑。

### Lines 84-109: Method ExpertLocationMetadata.init_trivial
```python
    @staticmethod
    def init_trivial(
        server_args: ServerArgs, model_config: ModelConfig, moe_ep_rank: int
    ):
        """Trivial location - logical expert i corresponds to physical expert i"""
        common = ExpertLocationMetadata._init_common(server_args, model_config)

        if common is None:
            return None

        num_physical_experts = common["num_physical_experts"]
        model_config_for_expert_location = common["model_config_for_expert_location"]
        num_layers = model_config_for_expert_location.num_layers
        num_logical_experts = model_config_for_expert_location.num_logical_experts

        physical_to_logical_map = (
            torch.arange(0, num_physical_experts).repeat(num_layers, 1)
            % num_logical_experts
        )

        return ExpertLocationMetadata.init_by_mapping(
            server_args,
            model_config,
            physical_to_logical_map=physical_to_logical_map,
            moe_ep_rank=moe_ep_rank,
        )
```
**EN:** This callable implements `ExpertLocationMetadata.init_trivial`. It takes `server_args`, `model_config`, `moe_ep_rank` and mainly implements init trivial. The docstring states: "Trivial location - logical expert i corresponds to physical expert i"
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata.init_trivial`。它接收 `server_args`, `model_config`, `moe_ep_rank`，主要用于实现 init trivial 相关逻辑。

### Lines 111-141: Method ExpertLocationMetadata.init_by_mapping
```python
    @staticmethod
    def init_by_mapping(
        server_args: ServerArgs,
        model_config: ModelConfig,
        physical_to_logical_map,
        moe_ep_rank: int = None,
    ):
        if not isinstance(physical_to_logical_map, torch.Tensor):
            physical_to_logical_map = torch.tensor(physical_to_logical_map)
        physical_to_logical_map = physical_to_logical_map.to(server_args.device)

        common = ExpertLocationMetadata._init_common(server_args, model_config)

        if common is None:
            return None

        model_config_for_expert_location = common["model_config_for_expert_location"]
        logical_to_all_physical_map = _compute_logical_to_all_physical_map(
            server_args=server_args,
            physical_to_logical_map=physical_to_logical_map,
            num_logical_experts=model_config_for_expert_location.num_logical_experts,
            ep_size=common["ep_size"],
            moe_ep_rank=moe_ep_rank,
        )

        return ExpertLocationMetadata._init_raw(
            server_args=server_args,
            ep_size=common["ep_size"],
            physical_to_logical_map=physical_to_logical_map,
            logical_to_all_physical_map=logical_to_all_physical_map,
        )
```
**EN:** This callable implements `ExpertLocationMetadata.init_by_mapping`. It takes `server_args`, `model_config`, `physical_to_logical_map`, `moe_ep_rank` and mainly implements init by mapping.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata.init_by_mapping`。它接收 `server_args`, `model_config`, `physical_to_logical_map`, `moe_ep_rank`，主要用于实现 init by mapping 相关逻辑。

### Lines 143-187: Method ExpertLocationMetadata.init_by_eplb
```python
    @staticmethod
    def init_by_eplb(
        server_args: ServerArgs, model_config: ModelConfig, logical_count: torch.Tensor
    ):
        if not isinstance(logical_count, torch.Tensor):
            logical_count = torch.tensor(logical_count)
        if len(logical_count.shape) == 2:
            logical_count = logical_count.unsqueeze(0)
        logical_count = logical_count.to(server_args.device)

        common = ExpertLocationMetadata._init_common(server_args, model_config)

        if common is None:
            return None

        model_config_for_expert_location = common["model_config_for_expert_location"]
        num_physical_experts = common["num_physical_experts"]
        num_groups = model_config_for_expert_location.num_groups
        num_nodes = server_args.nnodes

        from sglang.srt.eplb import eplb_algorithms

        physical_to_logical_map, logical_to_all_physical_map, expert_count = (
            eplb_algorithms.rebalance_experts(
                tokens_per_expert=logical_count,
                num_physical_experts=num_physical_experts,
                num_local_physical_experts=num_physical_experts // common["ep_size"],
                num_groups=num_groups,
                num_nodes=num_nodes,
                algorithm=eplb_algorithms.compute_algorithm(
                    raw_algorithm=server_args.eplb_algorithm,
                    num_groups=num_groups,
                    num_nodes=num_nodes,
                ),
            )
        )

        return ExpertLocationMetadata._init_raw(
            server_args=server_args,
            ep_size=common["ep_size"],
            physical_to_logical_map=physical_to_logical_map.to(server_args.device),
            logical_to_all_physical_map=logical_to_all_physical_map.to(
                server_args.device
            ),
        )
```
**EN:** This callable implements `ExpertLocationMetadata.init_by_eplb`. It takes `server_args`, `model_config`, `logical_count` and mainly implements init by eplb. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata.init_by_eplb`。它接收 `server_args`, `model_config`, `logical_count`，主要用于实现 init by eplb 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 189-211: Method ExpertLocationMetadata._init_common
```python
    @staticmethod
    def _init_common(server_args: ServerArgs, model_config: ModelConfig):
        model_config_for_expert_location = (
            ModelConfigForExpertLocation.from_model_config(model_config)
        )

        if model_config_for_expert_location is None:
            return None

        num_physical_experts = (
            model_config_for_expert_location.num_logical_experts
            + server_args.ep_num_redundant_experts
        )
        ep_size = server_args.ep_size
        assert num_physical_experts % ep_size == 0
        num_local_physical_experts = num_physical_experts // ep_size

        return dict(
            model_config_for_expert_location=model_config_for_expert_location,
            num_physical_experts=num_physical_experts,
            num_local_physical_experts=num_local_physical_experts,
            ep_size=ep_size,
        )
```
**EN:** This callable implements `ExpertLocationMetadata._init_common`. It takes `server_args`, `model_config` and mainly implements init common. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata._init_common`。它接收 `server_args`, `model_config`，主要用于实现 init common 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 213-250: Method ExpertLocationMetadata._init_raw
```python
    @staticmethod
    def _init_raw(
        server_args: ServerArgs,
        ep_size: int,
        physical_to_logical_map: torch.Tensor,
        logical_to_all_physical_map: torch.Tensor,
    ):
        _, num_physical_experts = physical_to_logical_map.shape

        logical_to_all_physical_map_padded = F.pad(
            logical_to_all_physical_map,
            (0, num_physical_experts - logical_to_all_physical_map.shape[-1]),
            value=-1,
        )

        logical_to_all_physical_map_num_valid = torch.count_nonzero(
            logical_to_all_physical_map != -1, dim=-1
        )

        return ExpertLocationMetadata(
            physical_to_logical_map=physical_to_logical_map,
            physical_to_logical_map_cpu=physical_to_logical_map.cpu(),
            logical_to_all_physical_map=logical_to_all_physical_map_padded,
            logical_to_all_physical_map_cpu=logical_to_all_physical_map_padded.cpu(),
            logical_to_all_physical_map_num_valid=logical_to_all_physical_map_num_valid,
            logical_to_rank_dispatch_physical_map=(
                compute_logical_to_rank_dispatch_physical_map(
                    server_args=server_args,
                    logical_to_all_physical_map=logical_to_all_physical_map,
                    ep_size=ep_size,
                    num_physical_experts=num_physical_experts,
                    # TODO improve when we have real EP rank
                    ep_rank=torch.distributed.get_rank() % ep_size,
                )
                if server_args.ep_dispatch_algorithm == "static"
                else None
            ),
        )
```
**EN:** This callable implements `ExpertLocationMetadata._init_raw`. It takes `server_args`, `ep_size`, `physical_to_logical_map`, `logical_to_all_physical_map` and mainly implements init raw. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata._init_raw`。它接收 `server_args`, `ep_size`, `physical_to_logical_map`, `logical_to_all_physical_map`，主要用于实现 init raw 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 251-253: Class-level scaffolding for ExpertLocationMetadata
```python

    # -------------------------------- mutation ------------------------------------

```
**EN:** This callable implements `None.ExpertLocationMetadata` and mainly implements Expert Location Metadata.
**CN:** 这一可调用对象实现了 `None.ExpertLocationMetadata`，主要用于实现 Expert Location Metadata 相关逻辑。

### Lines 254-281: Method ExpertLocationMetadata.update
```python
    def update(
        self,
        other: "ExpertLocationMetadata",
        update_layer_ids: List[int],
    ):
        for field in [
            "ep_size",
        ]:
            assert getattr(self, field) == getattr(other, field)

        for field in [
            "physical_to_logical_map",
            "physical_to_logical_map_cpu",
            "logical_to_all_physical_map",
            "logical_to_all_physical_map_cpu",
            "logical_to_all_physical_map_num_valid",
            "logical_to_rank_dispatch_physical_map",
        ]:
            other_field = getattr(other, field)
            self_field = getattr(self, field)
            assert (other_field is not None) == (self_field is not None)
            if self_field is not None:
                mask_update = torch.tensor(
                    [i in update_layer_ids for i in range(self.num_layers)]
                )
                mask_update = mask_update.view(*([-1] + [1] * (self_field.dim() - 1)))
                mask_update = mask_update.to(self_field.device, non_blocking=True)
                self_field[...] = torch.where(mask_update, other_field, self_field)
```
**EN:** This callable implements `ExpertLocationMetadata.update`. It takes `other`, `update_layer_ids` and mainly updates existing runtime state. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata.update`。它接收 `other`, `update_layer_ids`，主要用于更新现有运行时状态。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 282-284: Class-level scaffolding for ExpertLocationMetadata
```python

    # -------------------------------- usage ------------------------------------

```
**EN:** This callable implements `None.ExpertLocationMetadata` and mainly implements Expert Location Metadata.
**CN:** 这一可调用对象实现了 `None.ExpertLocationMetadata`，主要用于实现 Expert Location Metadata 相关逻辑。

### Lines 285-305: Method ExpertLocationMetadata.logical_to_all_physical
```python
    def logical_to_all_physical(
        self,
        layer_id: int,
        logical_expert_id: int,
        require_global_experts: bool = False,
    ) -> List[int]:
        # Use CPU copy to avoid GPU→CPU sync on every call, which is expensive in update weights scenario
        if require_global_experts:
            num_physical_experts = self.logical_to_all_physical_map_cpu[layer_id].shape[
                -1
            ]
            return list(
                range(logical_expert_id, num_physical_experts, self.num_logical_experts)
            )
        return [
            physical_expert_id
            for physical_expert_id in self.logical_to_all_physical_map_cpu[
                layer_id, logical_expert_id
            ].tolist()
            if physical_expert_id != -1
        ]
```
**EN:** This callable implements `ExpertLocationMetadata.logical_to_all_physical`. It takes `layer_id`, `logical_expert_id`, `require_global_experts` and mainly converts data into another representation. In this range it manages model weights or checkpoints.
**CN:** 这一可调用对象实现了 `ExpertLocationMetadata.logical_to_all_physical`。它接收 `layer_id`, `logical_expert_id`, `require_global_experts`，主要用于将数据转换为另一种表示。 在这一范围内，它会管理模型权重或检查点。

### Lines 306-310: Module-level constants and helpers
```python


_global_expert_location_metadata: Optional[ExpertLocationMetadata] = None


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 311-312: Function get_global_expert_location_metadata
```python
def get_global_expert_location_metadata():
    return _global_expert_location_metadata
```
**EN:** This callable implements `get_global_expert_location_metadata` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `get_global_expert_location_metadata`，主要用于获取某个值或派生视图。

### Lines 315-318: Function set_global_expert_location_metadata
```python
def set_global_expert_location_metadata(value):
    global _global_expert_location_metadata
    assert _global_expert_location_metadata is None
    _global_expert_location_metadata = value
```
**EN:** This callable implements `set_global_expert_location_metadata`. It takes `value` and mainly applies configuration to mutable state. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `set_global_expert_location_metadata`。它接收 `value`，主要用于将配置写入可变状态。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 321-364: Function broadcast_global_expert_location_metadata
```python
def broadcast_global_expert_location_metadata(
    src_rank: int = 0, group: Optional[torch.distributed.ProcessGroup] = None
):
    """Broadcast the global ExpertLocationMetadata from src_rank to all ranks.

    This is used in Elastic EP rank recovery to ensure that all ranks (including
    newly recovered ones) share exactly the same expert location metadata.

    Note: The caller must ensure src_rank is a healthy rank. In recovery scenarios,
    this function is called after try_recover_ranks succeeds, at which point all
    ranks (including src_rank=0) have recovered and are ready.
    """
    metadata = get_global_expert_location_metadata()
    assert metadata is not None

    # Ensure device tensors are contiguous before broadcasting in-place
    metadata.physical_to_logical_map = metadata.physical_to_logical_map.contiguous()
    metadata.logical_to_all_physical_map = (
        metadata.logical_to_all_physical_map.contiguous()
    )
    metadata.logical_to_all_physical_map_num_valid = (
        metadata.logical_to_all_physical_map_num_valid.contiguous()
    )
    if metadata.logical_to_rank_dispatch_physical_map is not None:
        metadata.logical_to_rank_dispatch_physical_map = (
            metadata.logical_to_rank_dispatch_physical_map.contiguous()
        )

    device_tensors = [
        metadata.physical_to_logical_map,
        metadata.logical_to_all_physical_map,
        metadata.logical_to_all_physical_map_num_valid,
    ]
    if metadata.logical_to_rank_dispatch_physical_map is not None:
        device_tensors.append(metadata.logical_to_rank_dispatch_physical_map)

    for tensor in device_tensors:
        torch.distributed.broadcast(tensor, src=src_rank, group=group)

    # After broadcasting device tensors, refresh corresponding CPU copies
    metadata.physical_to_logical_map_cpu = metadata.physical_to_logical_map.cpu()
    metadata.logical_to_all_physical_map_cpu = (
        metadata.logical_to_all_physical_map.cpu()
    )
```
**EN:** This callable implements `broadcast_global_expert_location_metadata`. It takes `src_rank`, `group` and mainly implements broadcast global expert location metadata. The docstring states: "Broadcast the global ExpertLocationMetadata from src_rank to all ranks." In this range it sets up imports and shared symbols; performs defensive checks on invalid state; coordinates distributed communication.
**CN:** 这一可调用对象实现了 `broadcast_global_expert_location_metadata`。它接收 `src_rank`, `group`，主要用于实现 broadcast global expert location metadata 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；协调分布式通信。

### Lines 367-424: Function _compute_logical_to_all_physical_map
```python
def _compute_logical_to_all_physical_map(
    server_args: ServerArgs,
    physical_to_logical_map: torch.Tensor,
    num_logical_experts: int,
    ep_size: int,
    moe_ep_rank: int,
):
    # This is rarely called, so we use for loops for maximum clarity

    num_layers, num_physical_experts = physical_to_logical_map.shape

    logical_to_all_physical_map = [
        [[] for _ in range(num_logical_experts)] for _ in range(num_layers)
    ]

    # Find out the candidate physical experts for each logical expert on each layer
    for layer_id in range(num_layers):
        for physical_expert_id in range(num_physical_experts):
            logical_expert_id = physical_to_logical_map[
                layer_id, physical_expert_id
            ].item()
            logical_to_all_physical_map[layer_id][logical_expert_id].append(
                physical_expert_id
            )

    # Replace by the physical expert on local GPU or node if possible
    if moe_ep_rank is not None:
        num_gpus_per_node = server_args.ep_size // server_args.nnodes
        num_local_gpu_physical_experts = num_physical_experts // ep_size
        num_local_node_physical_experts = (
            num_local_gpu_physical_experts * num_gpus_per_node
        )
        for layer_id in range(num_layers):
            for logical_expert_id in range(num_logical_experts):
                # Try to find the nearest physical expert
                nearest_expert = _find_nearest_expert(
                    candidate_physical_expert_ids=logical_to_all_physical_map[layer_id][
                        logical_expert_id
                    ],
                    num_local_gpu_physical_experts=num_local_gpu_physical_experts,
                    moe_ep_rank=moe_ep_rank,
                    num_gpus_per_node=num_gpus_per_node,
                    num_local_node_physical_experts=num_local_node_physical_experts,
                )

                # Replace by the nearest physical expert
                if nearest_expert != -1:
                    logical_to_all_physical_map[layer_id][logical_expert_id] = [
                        nearest_expert
                    ]

    logical_to_all_physical_map = _pad_nested_array(
        logical_to_all_physical_map, pad_value=-1
    )

    return torch.tensor(
        logical_to_all_physical_map, device=physical_to_logical_map.device
    )
```
**EN:** This callable implements `_compute_logical_to_all_physical_map`. It takes `server_args`, `physical_to_logical_map`, `num_logical_experts`, `ep_size` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `_compute_logical_to_all_physical_map`。它接收 `server_args`, `physical_to_logical_map`, `num_logical_experts`, `ep_size`，主要用于将数据转换为另一种表示。

### Lines 427-433: Function _pad_nested_array
```python
def _pad_nested_array(arr, pad_value):
    max_len = max(len(inner) for outer in arr for inner in outer)
    padded = [
        [inner + [pad_value] * (max_len - len(inner)) for inner in outer]
        for outer in arr
    ]
    return padded
```
**EN:** This callable implements `_pad_nested_array`. It takes `arr`, `pad_value` and mainly implements pad nested array.
**CN:** 这一可调用对象实现了 `_pad_nested_array`。它接收 `arr`, `pad_value`，主要用于实现 pad nested array 相关逻辑。

### Lines 434-436: Module-level logic
```python


# TODO optimize performance (rewrite and/or run in separate process with overlap)
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 437-490: Function compute_logical_to_rank_dispatch_physical_map
```python
def compute_logical_to_rank_dispatch_physical_map(
    server_args: ServerArgs,
    logical_to_all_physical_map: torch.Tensor,
    ep_size: int,
    num_physical_experts: int,
    ep_rank: int,
    seed: int = 42,
):
    r = random.Random(seed)

    device = logical_to_all_physical_map.device
    logical_to_all_physical_map = logical_to_all_physical_map.cpu()

    num_local_gpu_physical_experts = num_physical_experts // ep_size
    num_gpus_per_node = server_args.ep_size // server_args.nnodes
    num_local_node_physical_experts = num_local_gpu_physical_experts * num_gpus_per_node
    num_layers, num_logical_experts, _ = logical_to_all_physical_map.shape
    dtype = logical_to_all_physical_map.dtype

    result_list = [
        [[-1] * num_logical_experts for _ in range(num_layers)] for _ in range(ep_size)
    ]

    for layer_id in range(num_layers):
        for logical_expert_id in range(num_logical_experts):
            candidate_physical_expert_ids = _logical_to_all_physical_raw(
                logical_to_all_physical_map, layer_id, logical_expert_id
            )

            remaining_ranks = []
            for moe_ep_rank in range(ep_size):
                val = _find_nearest_expert(
                    candidate_physical_expert_ids=candidate_physical_expert_ids,
                    num_local_gpu_physical_experts=num_local_gpu_physical_experts,
                    moe_ep_rank=moe_ep_rank,
                    num_gpus_per_node=num_gpus_per_node,
                    num_local_node_physical_experts=num_local_node_physical_experts,
                )

                result_list[moe_ep_rank][layer_id][logical_expert_id] = val
                if val == -1:
                    remaining_ranks.append(moe_ep_rank)

            if remaining_ranks:
                choices = _fair_choices(
                    candidate_physical_expert_ids, k=len(remaining_ranks), r=r
                )
                for moe_ep_rank, choice in zip(remaining_ranks, choices, strict=True):
                    result_list[moe_ep_rank][layer_id][logical_expert_id] = choice

    logical_to_rank_dispatch_physical_map = torch.tensor(result_list, dtype=dtype)
    assert torch.all(logical_to_rank_dispatch_physical_map != -1)

    return logical_to_rank_dispatch_physical_map[ep_rank, :, :].to(device)
```
**EN:** This callable implements `compute_logical_to_rank_dispatch_physical_map`. It takes `server_args`, `logical_to_all_physical_map`, `ep_size`, `num_physical_experts` and mainly converts data into another representation. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `compute_logical_to_rank_dispatch_physical_map`。它接收 `server_args`, `logical_to_all_physical_map`, `ep_size`, `num_physical_experts`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 493-502: Function _logical_to_all_physical_raw
```python
def _logical_to_all_physical_raw(
    logical_to_all_physical_map, layer_id: int, logical_expert_id: int
) -> List[int]:
    return [
        physical_expert_id
        for physical_expert_id in logical_to_all_physical_map[
            layer_id, logical_expert_id
        ].tolist()
        if physical_expert_id != -1
    ]
```
**EN:** This callable implements `_logical_to_all_physical_raw`. It takes `logical_to_all_physical_map`, `layer_id`, `logical_expert_id` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `_logical_to_all_physical_raw`。它接收 `logical_to_all_physical_map`, `layer_id`, `logical_expert_id`，主要用于将数据转换为另一种表示。

### Lines 505-508: Function _compute_gpu_id_of_physical_expert
```python
def _compute_gpu_id_of_physical_expert(
    physical_expert_id: int, num_local_gpu_physical_experts: int
) -> int:
    return physical_expert_id // num_local_gpu_physical_experts
```
**EN:** This callable implements `_compute_gpu_id_of_physical_expert`. It takes `physical_expert_id`, `num_local_gpu_physical_experts` and mainly implements compute gpu id of physical expert.
**CN:** 这一可调用对象实现了 `_compute_gpu_id_of_physical_expert`。它接收 `physical_expert_id`, `num_local_gpu_physical_experts`，主要用于实现 compute gpu id of physical expert 相关逻辑。

### Lines 511-514: Function _compute_node_id_of_physical_expert
```python
def _compute_node_id_of_physical_expert(
    physical_expert_id: int, num_local_host_physical_experts: int
) -> int:
    return physical_expert_id // num_local_host_physical_experts
```
**EN:** This callable implements `_compute_node_id_of_physical_expert`. It takes `physical_expert_id`, `num_local_host_physical_experts` and mainly implements compute node id of physical expert.
**CN:** 这一可调用对象实现了 `_compute_node_id_of_physical_expert`。它接收 `physical_expert_id`, `num_local_host_physical_experts`，主要用于实现 compute node id of physical expert 相关逻辑。

### Lines 517-554: Function _find_nearest_expert
```python
def _find_nearest_expert(
    candidate_physical_expert_ids: List[int],
    num_local_gpu_physical_experts: int,
    moe_ep_rank: int,
    num_gpus_per_node: int,
    num_local_node_physical_experts: int,
) -> int:
    # 1. If only one candidate, return it directly
    if len(candidate_physical_expert_ids) == 1:
        return candidate_physical_expert_ids[0]

    # 2. Prefer same-GPU experts
    same_gpu_physical_expert_ids = [
        physical_expert_id
        for physical_expert_id in candidate_physical_expert_ids
        if _compute_gpu_id_of_physical_expert(
            physical_expert_id, num_local_gpu_physical_experts
        )
        == moe_ep_rank
    ]
    if len(same_gpu_physical_expert_ids) > 0:
        return same_gpu_physical_expert_ids[0]

    # 3. Otherwise, prefer same-node experts
    node_rank = moe_ep_rank // num_gpus_per_node
    same_node_physical_expert_ids = [
        physical_expert_id
        for physical_expert_id in candidate_physical_expert_ids
        if _compute_node_id_of_physical_expert(
            physical_expert_id, num_local_node_physical_experts
        )
        == node_rank
    ]
    if len(same_node_physical_expert_ids) > 0:
        return same_node_physical_expert_ids[0]

    # 4. At last, leave it as -1 to indicate not found.
    return -1
```
**EN:** This callable implements `_find_nearest_expert`. It takes `candidate_physical_expert_ids`, `num_local_gpu_physical_experts`, `moe_ep_rank`, `num_gpus_per_node` and mainly implements find nearest expert.
**CN:** 这一可调用对象实现了 `_find_nearest_expert`。它接收 `candidate_physical_expert_ids`, `num_local_gpu_physical_experts`, `moe_ep_rank`, `num_gpus_per_node`，主要用于实现 find nearest expert 相关逻辑。

### Lines 557-561: Function _fair_choices
```python
def _fair_choices(arr: List, k: int, r: random.Random) -> List:
    quotient, remainder = divmod(k, len(arr))
    ans = arr * quotient + r.sample(arr, k=remainder)
    r.shuffle(ans)
    return ans
```
**EN:** This callable implements `_fair_choices`. It takes `arr`, `k`, `r` and mainly implements fair choices.
**CN:** 这一可调用对象实现了 `_fair_choices`。它接收 `arr`, `k`, `r`，主要用于实现 fair choices 相关逻辑。

### Lines 564-569: Class ModelConfigForExpertLocation
```python
@dataclass
class ModelConfigForExpertLocation:
    num_layers: int
    num_logical_experts: int
    num_groups: Optional[int] = None

```
**EN:** This range introduces `ModelConfigForExpertLocation` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ModelConfigForExpertLocation`，并定义其后续方法依赖的结构或元数据。

### Lines 570-580: Method ModelConfigForExpertLocation.from_model_config
```python
    @staticmethod
    def from_model_config(model_config: ModelConfig):
        from sglang.srt.model_loader import get_model_architecture

        model_class, _ = get_model_architecture(model_config)
        if hasattr(model_class, "get_model_config_for_expert_location"):
            return model_class.get_model_config_for_expert_location(
                model_config.hf_config
            )
        else:
            return None
```
**EN:** This callable implements `ModelConfigForExpertLocation.from_model_config`. It takes `model_config` and mainly constructs data from an external representation. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `ModelConfigForExpertLocation.from_model_config`。它接收 `model_config`，主要用于从外部表示构造数据。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 583-622: Function compute_initial_expert_location_metadata
```python
def compute_initial_expert_location_metadata(
    server_args: ServerArgs,
    model_config: ModelConfig,
    moe_ep_rank: int,
) -> Optional[ExpertLocationMetadata]:
    data = server_args.init_expert_location
    if data == "trivial":
        return ExpertLocationMetadata.init_trivial(
            server_args, model_config, moe_ep_rank
        )

    # TODO unify with the utils function
    if data.endswith(".pt"):
        data_dict = torch.load(data, weights_only=True)
    elif data.endswith(".json"):
        data_dict = json.loads(Path(data).read_text())
    else:
        data_dict = json.loads(data)

    if "physical_to_logical_map" in data_dict:
        logger.info(
            "init_expert_location from init_by_mapping using ServerArgs.init_expert_location"
        )
        return ExpertLocationMetadata.init_by_mapping(
            server_args,
            model_config,
            **data_dict,
            moe_ep_rank=moe_ep_rank,
        )
    elif "logical_count" in data_dict:
        logger.info(
            "init_expert_location from init_by_eplb using ServerArgs.init_expert_location"
        )
        return ExpertLocationMetadata.init_by_eplb(
            server_args, model_config, logical_count=data_dict["logical_count"]
        )
    else:
        raise NotImplementedError(
            f"Unknown init_expert_location format ({list(data_dict.keys())=})"
        )
```
**EN:** This callable implements `compute_initial_expert_location_metadata`. It takes `server_args`, `model_config`, `moe_ep_rank` and mainly implements compute initial expert location metadata. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `compute_initial_expert_location_metadata`。它接收 `server_args`, `model_config`, `moe_ep_rank`，主要用于实现 compute initial expert location metadata 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

## Key Concepts / 关键概念
- `ExpertLocationMetadata`: core class or state container / 核心类或状态容器
- `get_global_expert_location_metadata`: retrieves a value or derived view / 获取某个值或派生视图
- `set_global_expert_location_metadata`: applies configuration to mutable state / 将配置写入可变状态
- `broadcast_global_expert_location_metadata`: implements broadcast global expert location metadata / 实现 broadcast global expert location metadata 相关逻辑
- `_compute_logical_to_all_physical_map`: converts data into another representation / 将数据转换为另一种表示
- `_pad_nested_array`: implements pad nested array / 实现 pad nested array 相关逻辑
- `compute_logical_to_rank_dispatch_physical_map`: converts data into another representation / 将数据转换为另一种表示
- `_logical_to_all_physical_raw`: converts data into another representation / 将数据转换为另一种表示
- `_compute_gpu_id_of_physical_expert`: implements compute gpu id of physical expert / 实现 compute gpu id of physical expert 相关逻辑
- `_compute_node_id_of_physical_expert`: implements compute node id of physical expert / 实现 compute node id of physical expert 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `json`, `logging`, `random`, `dataclasses`, `pathlib`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`, `torch.nn.functional`
- **Internal modules / 内部模块**: `sglang.srt.configs.model_config`, `sglang.srt.server_args`, `sglang.srt.eplb`, `sglang.srt.model_loader`
