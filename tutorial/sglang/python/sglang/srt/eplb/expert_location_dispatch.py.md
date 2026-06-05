# expert_location_dispatch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/eplb/expert_location_dispatch.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the expert-parallel load balancing part of the SRT runtime and implements logic centered on `expert_location_dispatch`. It exposes primary entry points such as `ExpertLocationDispatchInfo`, `transform_select_experts_inputs`, `topk_ids_logical_to_physical`. / 该模块属于 SRT 运行时的专家并行负载均衡部分，主要实现围绕 `expert_location_dispatch` 的逻辑。 它对外提供的主要入口包括 `ExpertLocationDispatchInfo`, `transform_select_experts_inputs`, `topk_ids_logical_to_physical`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Module imports, constants, and setup
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

from dataclasses import dataclass
from typing import Literal, Optional

import torch

from sglang.srt.eplb.expert_location import get_global_expert_location_metadata
from sglang.srt.server_args import get_global_server_args


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 24-34: Class ExpertLocationDispatchInfo
```python
@dataclass
class ExpertLocationDispatchInfo:
    ep_dispatch_algorithm: Literal["static", "random"]
    # (num_logical_experts,)
    partial_logical_to_rank_dispatch_physical_map: Optional[torch.Tensor]
    # (num_logical_experts, X)
    partial_logical_to_all_physical_map: torch.Tensor
    # (num_logical_experts,)
    partial_logical_to_all_physical_map_num_valid: torch.Tensor
    num_physical_experts: int

```
**EN:** This range introduces `ExpertLocationDispatchInfo` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ExpertLocationDispatchInfo`，并定义其后续方法依赖的结构或元数据。

### Lines 35-61: Method ExpertLocationDispatchInfo.init_new
```python
    @classmethod
    def init_new(cls, layer_id: int):
        ep_dispatch_algorithm = get_global_server_args().ep_dispatch_algorithm
        expert_location_metadata = get_global_expert_location_metadata()
        assert expert_location_metadata is not None

        if ep_dispatch_algorithm is None:
            return None

        return cls(
            ep_dispatch_algorithm=ep_dispatch_algorithm,
            partial_logical_to_rank_dispatch_physical_map=(
                expert_location_metadata.logical_to_rank_dispatch_physical_map[
                    layer_id, :
                ]
                if expert_location_metadata.logical_to_rank_dispatch_physical_map
                is not None
                else None
            ),
            partial_logical_to_all_physical_map=expert_location_metadata.logical_to_all_physical_map[
                layer_id, :
            ],
            partial_logical_to_all_physical_map_num_valid=expert_location_metadata.logical_to_all_physical_map_num_valid[
                layer_id, :
            ],
            num_physical_experts=expert_location_metadata.num_physical_experts,
        )
```
**EN:** This callable implements `ExpertLocationDispatchInfo.init_new`. It takes `cls`, `layer_id` and mainly implements init new. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ExpertLocationDispatchInfo.init_new`。它接收 `cls`, `layer_id`，主要用于实现 init new 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 64-73: Function transform_select_experts_inputs
```python
def transform_select_experts_inputs(
    router_logits: torch.Tensor,
    correction_bias: Optional[torch.Tensor],
    info: Optional[ExpertLocationDispatchInfo],
):
    if (info is not None) and (info.ep_dispatch_algorithm == "fake"):
        router_logits.uniform_(5, 10)
        if correction_bias is not None:
            correction_bias = torch.zeros_like(correction_bias)
    return router_logits, correction_bias
```
**EN:** This callable implements `transform_select_experts_inputs`. It takes `router_logits`, `correction_bias`, `info` and mainly implements transform select experts inputs.
**CN:** 这一可调用对象实现了 `transform_select_experts_inputs`。它接收 `router_logits`, `correction_bias`, `info`，主要用于实现 transform select experts inputs 相关逻辑。

### Lines 76-86: Function topk_ids_logical_to_physical
```python
def topk_ids_logical_to_physical(
    topk_ids: torch.Tensor, info: Optional[ExpertLocationDispatchInfo]
) -> torch.Tensor:
    if info is None:
        return topk_ids

    if info.ep_dispatch_algorithm == "static":
        return _topk_ids_logical_to_physical_static(topk_ids, info)
    if info.ep_dispatch_algorithm in ["dynamic", "fake"]:
        return _topk_ids_logical_to_physical_dynamic(topk_ids, info)
    raise NotImplementedError(f"Unknown algorithm {info.ep_dispatch_algorithm}")
```
**EN:** This callable implements `topk_ids_logical_to_physical`. It takes `topk_ids`, `info` and mainly converts data into another representation. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `topk_ids_logical_to_physical`。它接收 `topk_ids`, `info`，主要用于将数据转换为另一种表示。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 89-92: Function _topk_ids_logical_to_physical_static
```python
def _topk_ids_logical_to_physical_static(
    topk_ids: torch.Tensor, info: Optional[ExpertLocationDispatchInfo]
) -> torch.Tensor:
    return info.partial_logical_to_rank_dispatch_physical_map[topk_ids]
```
**EN:** This callable implements `_topk_ids_logical_to_physical_static`. It takes `topk_ids`, `info` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `_topk_ids_logical_to_physical_static`。它接收 `topk_ids`, `info`，主要用于将数据转换为另一种表示。

### Lines 95-109: Function _topk_ids_logical_to_physical_dynamic
```python
def _topk_ids_logical_to_physical_dynamic(
    topk_ids: torch.Tensor, info: Optional[ExpertLocationDispatchInfo]
) -> torch.Tensor:
    topk_ids_original_shape = topk_ids.shape
    device = topk_ids.device
    topk_ids = topk_ids.flatten()

    chosen_dispatch_index = (
        torch.randint(0, 65536, topk_ids.shape, dtype=torch.int32, device=device)
        % info.partial_logical_to_all_physical_map_num_valid[topk_ids]
    )
    topk_ids = info.partial_logical_to_all_physical_map[topk_ids, chosen_dispatch_index]

    topk_ids = topk_ids.view(topk_ids_original_shape)
    return topk_ids
```
**EN:** This callable implements `_topk_ids_logical_to_physical_dynamic`. It takes `topk_ids`, `info` and mainly converts data into another representation.
**CN:** 这一可调用对象实现了 `_topk_ids_logical_to_physical_dynamic`。它接收 `topk_ids`, `info`，主要用于将数据转换为另一种表示。

## Key Concepts / 关键概念
- `ExpertLocationDispatchInfo`: core class or state container / 核心类或状态容器
- `transform_select_experts_inputs`: implements transform select experts inputs / 实现 transform select experts inputs 相关逻辑
- `topk_ids_logical_to_physical`: converts data into another representation / 将数据转换为另一种表示
- `_topk_ids_logical_to_physical_static`: converts data into another representation / 将数据转换为另一种表示
- `_topk_ids_logical_to_physical_dynamic`: converts data into another representation / 将数据转换为另一种表示

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.eplb.expert_location`, `sglang.srt.server_args`
