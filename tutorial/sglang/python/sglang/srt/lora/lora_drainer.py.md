# lora_drainer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/lora_drainer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements core LoRA runtime logic such as adapter metadata, memory management, layer wrapping, or orchestration. It is part of SGLang's infrastructure for serving models with dynamic adapters. / 该文件实现了核心 LoRA 运行时逻辑，例如适配器元数据、内存管理、层封装或整体编排。它属于 SGLang 支持动态适配器服务能力的基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27: Module header, imports, and shared constants
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

import logging
import time
from collections import defaultdict
from dataclasses import dataclass
from typing import Dict, List, Optional

from sglang.srt.managers.schedule_batch import Req

logger = logging.getLogger(__name__)

DRAIN_SCHEDULE_TOLERANCE = 1.2
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 29-34: AdapterStats class declaration
```python
class AdapterStats:
    num_waiting_reqs: int = 0
    max_wait_time_secs: float = 0.0
    max_remaining_tokens: int = 0
    is_draining_for: Optional[str] = None
```
**EN:** This block declares the `AdapterStats` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `AdapterStats` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 35-38: AdapterStats._reset_stats method
```python
    def _reset_stats(self):
        self.num_waiting_reqs = 0
        self.max_wait_time_secs = 0.0
        self.max_remaining_tokens = 0
```
**EN:** This block uses `AdapterStats._reset_stats` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AdapterStats._reset_stats` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 40-43: AdapterStats.is_starving method
```python
    def is_starving(self, drain_wait_threshold: float):
        return (
            self.max_wait_time_secs > drain_wait_threshold and self.num_waiting_reqs > 0
        )
```
**EN:** This block uses `AdapterStats.is_starving` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `AdapterStats.is_starving` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 46-53: LoRADrainer class declaration
```python
class LoRADrainer:
    """
    Drainer for LoRA requests that manages draining. It tracks:
    - Number of waiting requests per adapter
    - Maximum wait time for requests needing each adapter
    - Maximum number of tokens needed for running requests for each adapter
    """
```
**EN:** This block declares the `LoRADrainer` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `LoRADrainer` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 54-59: LoRADrainer initializer
```python
    def __init__(self, max_loras_per_batch: int, max_wait_time_secs: float = 0.0):
        self.max_loras_per_batch = max_loras_per_batch
        self.max_wait_time_secs = max_wait_time_secs
        self.adapter_to_stats: Dict[Optional[str], AdapterStats] = defaultdict(
            AdapterStats
        )
```
**EN:** This block initializes the `LoRADrainer` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `LoRADrainer` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 61-74: LoRADrainer.update_draining_state method
```python
    def update_draining_state(
        self,
        waiting_queue: List[Req],
        running_reqs: List[Req],
    ) -> None:
        """
        Update LoRA drainer state based on current waiting queue and running requests.

        This method updates adapter statistics, identifies starving adapters that need
        to be scheduled, and marks adapters for draining to make room for starving ones.
        """
        self._update_adapter_stats(waiting_queue, running_reqs)
        self._update_draining_loras(running_reqs)
        self._update_fully_drained_loras(running_reqs)
```
**EN:** This block uses `LoRADrainer.update_draining_state` to refresh state using new information. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRADrainer.update_draining_state` 来利用新信息刷新状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 76-99: LoRADrainer._update_adapter_stats method
```python
    def _update_adapter_stats(
        self,
        waiting_queue: List[Req],
        running_reqs: List[Req],
    ) -> None:
        for stats in self.adapter_to_stats.values():
            stats._reset_stats()

        for req in waiting_queue:
            stats = self.adapter_to_stats[req.lora_id]

            stats.num_waiting_reqs += 1
            stats.max_wait_time_secs = max(
                stats.max_wait_time_secs,
                time.monotonic() - req.time_stats.wait_queue_entry_time,
            )

        for req in running_reqs:
            stats = self.adapter_to_stats[req.lora_id]

            stats.max_remaining_tokens = max(
                stats.max_remaining_tokens,
                req.sampling_params.max_new_tokens - len(req.output_ids),
            )
```
**EN:** This block uses `LoRADrainer._update_adapter_stats` to refresh state using new information. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRADrainer._update_adapter_stats` 来利用新信息刷新状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 101-158: LoRADrainer._update_draining_loras method
```python
    def _update_draining_loras(self, running_reqs: List[Req]) -> None:
        """
        Select LoRA adapters to drain based on starvation detection.

        This method identifies adapters in the waiting queue that are "starving"
        (waiting too long) and marks currently running adapters as "draining"
        to make room for the starving adapters. Draining adapters will not
        accept new requests, allowing them to complete and free up slots.
        """
        running_adapter_ids = {req.lora_id for req in running_reqs}
        if len(running_adapter_ids) < self.max_loras_per_batch:
            return None

        starving_adapters = set()
        draining_for_adapters = set()
        for adapter_id, stats in self.adapter_to_stats.items():
            if stats.is_starving(self.max_wait_time_secs):
                starving_adapters.add(adapter_id)

            draining_for_adapter = stats.is_draining_for
            if draining_for_adapter is not None:
                draining_for_adapters.add(draining_for_adapter)

        new_starving_adapters = starving_adapters - draining_for_adapters
        if not new_starving_adapters:
            return None

        sorted_new_starving_adapters = sorted(
            new_starving_adapters,
            key=lambda adapter: self.adapter_to_stats[adapter].max_wait_time_secs,
            reverse=True,
        )

        eligible_to_drain_adapters = {
            adapter
            for adapter in running_adapter_ids
            if self.adapter_to_stats[adapter].is_draining_for is None
        }

        for starving_adapter in sorted_new_starving_adapters:
            if not eligible_to_drain_adapters:
                break

            min_eligible_adapter = min(
                eligible_to_drain_adapters,
                key=lambda adapter_id: self.adapter_to_stats[
                    adapter_id
                ].max_remaining_tokens,
            )

            self.adapter_to_stats[min_eligible_adapter].is_draining_for = (
                starving_adapter
            )
            logger.debug(
                f"LoRA adapter {min_eligible_adapter} is draining for {starving_adapter}"
            )

            eligible_to_drain_adapters.remove(min_eligible_adapter)
```
**EN:** This block uses `LoRADrainer._update_draining_loras` to refresh state using new information. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRADrainer._update_draining_loras` 来利用新信息刷新状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 160-174: LoRADrainer._update_fully_drained_loras method
```python
    def _update_fully_drained_loras(self, running_reqs: List[Req]) -> None:
        """
        Clear draining state for adapters that have fully drained.

        An adapter is considered fully drained when it was marked as draining
        but no longer has any running requests.
        """
        running_adapter_ids = {req.lora_id for req in running_reqs}
        for adapter_id, stats in self.adapter_to_stats.items():
            if stats.is_draining_for is None:
                continue

            if adapter_id not in running_adapter_ids:
                logger.debug(f"LoRA adapter {adapter_id} finished draining")
                stats.is_draining_for = None
```
**EN:** This block uses `LoRADrainer._update_fully_drained_loras` to refresh state using new information. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRADrainer._update_fully_drained_loras` 来利用新信息刷新状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 176-191: LoRADrainer.can_schedule method
```python
    def can_schedule(self, req: Req) -> bool:
        """
        Check if a request can be scheduled based on draining state.

        If the adapter for this request is currently draining, only allow
        scheduling if the request's max_new_tokens is within tolerance of
        the max remaining tokens for the draining adapter.
        """
        stats = self.adapter_to_stats[req.lora_id]
        if not stats.is_draining_for:
            return True

        return (
            req.sampling_params.max_new_tokens
            <= stats.max_remaining_tokens * DRAIN_SCHEDULE_TOLERANCE
        )
```
**EN:** This block uses `LoRADrainer.can_schedule` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `LoRADrainer.can_schedule` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.managers.schedule_batch`
### External / 外部
- `collections` (stdlib)
- `dataclasses` (stdlib)
- `logging` (stdlib)
- `time` (stdlib)
- `typing` (stdlib)
