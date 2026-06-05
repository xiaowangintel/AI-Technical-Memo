# frozen_kv_mtp_info.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/frozen_kv_mtp_info.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements speculative decoding infrastructure, such as workers, metadata objects, CUDA-graph runners, or shared utilities. It coordinates draft-model execution with target-model verification to improve throughput. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28: Module header, imports, and shared constants
```python
# Copyright 2026 SGLang Team
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

from dataclasses import dataclass, fields
from typing import Dict

from sglang.srt.mem_cache.memory_pool import KVCache
from sglang.srt.speculative.eagle_info import (
    EagleDraftExtendInput,
    EagleDraftInput,
    EagleVerifyInput,
    EagleVerifyOutput,
)
from sglang.srt.speculative.spec_info import SpecInput, SpecInputType
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 30-35: FrozenKVMTPContext class declaration
```python
class FrozenKVMTPContext:
    """Target KV pool + assistant-logical -> target-physical layer map."""

    target_token_to_kv_pool: KVCache
    physical_layer_ids: Dict[int, int]
```
**EN:** This block declares the `FrozenKVMTPContext` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `FrozenKVMTPContext` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 36-42: FrozenKVMTPContext.get_physical_layer_id method
```python
    def get_physical_layer_id(self, idx: int) -> int:
        if idx not in self.physical_layer_ids:
            raise KeyError(
                f"FrozenKVMTPContext has no physical layer id for assistant "
                f"logical index {idx}; available: {sorted(self.physical_layer_ids)}"
            )
        return self.physical_layer_ids[idx]
```
**EN:** This block uses `FrozenKVMTPContext.get_physical_layer_id` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPContext.get_physical_layer_id` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 46-52: FrozenKVMTPDraftInput class declaration
```python
class FrozenKVMTPDraftInput(EagleDraftInput):
    """Draft input for Frozen-KV MTP.

    Frozen-KV MTP currently reuses the EAGLE scheduler/attention contract, but
    has a dedicated type so algorithm-specific behavior can move here over time.
    """
```
**EN:** This block declares the `FrozenKVMTPDraftInput` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `FrozenKVMTPDraftInput` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 53-54: FrozenKVMTPDraftInput.__post_init__ method
```python
    def __post_init__(self):
        SpecInput.__init__(self, SpecInputType.FROZEN_KV_MTP_DRAFT)
```
**EN:** This block uses `FrozenKVMTPDraftInput.__post_init__` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPDraftInput.__post_init__` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 58-60: FrozenKVMTPDraftExtendInput class declaration
```python
class FrozenKVMTPDraftExtendInput(EagleDraftExtendInput):
    """Draft-extend input for Frozen-KV MTP. Tag-only subclass."""
```
**EN:** This block declares the `FrozenKVMTPDraftExtendInput` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `FrozenKVMTPDraftExtendInput` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 61-62: FrozenKVMTPDraftExtendInput.__post_init__ method
```python
    def __post_init__(self):
        SpecInput.__init__(self, SpecInputType.FROZEN_KV_MTP_DRAFT_EXTEND)
```
**EN:** This block uses `FrozenKVMTPDraftExtendInput.__post_init__` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPDraftExtendInput.__post_init__` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 66-68: FrozenKVMTPVerifyInput class declaration
```python
class FrozenKVMTPVerifyInput(EagleVerifyInput):
    """Verify input for Frozen-KV MTP."""
```
**EN:** This block declares the `FrozenKVMTPVerifyInput` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `FrozenKVMTPVerifyInput` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 69-70: FrozenKVMTPVerifyInput.__post_init__ method
```python
    def __post_init__(self):
        SpecInput.__init__(self, SpecInputType.FROZEN_KV_MTP_VERIFY)
```
**EN:** This block uses `FrozenKVMTPVerifyInput.__post_init__` to initialize state and buffers. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPVerifyInput.__post_init__` 来初始化状态与缓冲区。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 72-77: FrozenKVMTPVerifyInput.verify method
```python
    def verify(self, *args, **kwargs) -> EagleVerifyOutput:
        output = super().verify(*args, **kwargs)
        output.draft_extend_input = _to_frozen_kv_mtp_draft_extend_input(
            output.draft_extend_input
        )
        return output
```
**EN:** This block uses `FrozenKVMTPVerifyInput.verify` to verify draft results against the target model. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FrozenKVMTPVerifyInput.verify` 来根据目标模型校验草稿结果。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 78-82: Module-level supporting statements
```python


FrozenKVMTPVerifyOutput = EagleVerifyOutput
```
**EN:** This block contains supporting statements for the module, such as constants, attributes, registrations, or small helper expressions that shape the surrounding control flow.
**CN:** 该代码块包含模块的配套语句，例如常量、属性、注册逻辑或小型辅助表达式，它们共同塑造了周边控制流程。

### Lines 83-93: to frozen kv mtp draft extend input function
```python
def _to_frozen_kv_mtp_draft_extend_input(
    draft_extend_input: EagleDraftExtendInput,
) -> FrozenKVMTPDraftExtendInput:
    if isinstance(draft_extend_input, FrozenKVMTPDraftExtendInput):
        return draft_extend_input
    return FrozenKVMTPDraftExtendInput(
        **{
            field.name: getattr(draft_extend_input, field.name)
            for field in fields(EagleDraftExtendInput)
        }
    )
```
**EN:** This block uses `_to_frozen_kv_mtp_draft_extend_input` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `_to_frozen_kv_mtp_draft_extend_input` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Speculative decoding / 推测解码
- Frozen-KV MTP path / Frozen-KV MTP 路径

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.mem_cache.memory_pool`
- `sglang.srt.speculative.eagle_info`
- `sglang.srt.speculative.spec_info`
### External / 外部
- `__future__`
- `dataclasses` (stdlib)
- `typing` (stdlib)
