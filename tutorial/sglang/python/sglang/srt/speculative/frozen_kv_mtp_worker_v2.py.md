# frozen_kv_mtp_worker_v2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/frozen_kv_mtp_worker_v2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Overlap-scheduling placeholder for frozen-KV MTP (raises until implemented). / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24: Module header, imports, and shared constants
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
"""Overlap-scheduling placeholder for frozen-KV MTP (raises until implemented)."""

from __future__ import annotations

from typing import Optional

from sglang.srt.managers.tp_worker import TpModelWorker
from sglang.srt.server_args import ServerArgs
from sglang.srt.speculative.frozen_kv_mtp_worker import FrozenKVMTPWorker
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 25-25: FrozenKVMTPWorkerV2 class declaration
```python
class FrozenKVMTPWorkerV2(FrozenKVMTPWorker):
```
**EN:** This block declares the `FrozenKVMTPWorkerV2` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `FrozenKVMTPWorkerV2` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 26-42: FrozenKVMTPWorkerV2 initializer
```python
    def __init__(
        self,
        server_args: ServerArgs,
        gpu_id: int,
        tp_rank: int,
        dp_rank: Optional[int],
        moe_ep_rank: int,
        attn_cp_rank: int,
        moe_dp_rank: int,
        nccl_port: int,
        target_worker: TpModelWorker,
    ):
        raise NotImplementedError(
            "FrozenKVMTPWorkerV2 (overlap scheduling for Frozen-KV MTP) is "
            "not yet implemented. Pass --disable-overlap-schedule to use "
            "FrozenKVMTPWorker."
        )
```
**EN:** This block initializes the `FrozenKVMTPWorkerV2` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `FrozenKVMTPWorkerV2` 对象，建立后续方法依赖的状态、缓冲区与引用。

## Key Concepts / 关键概念
- Worker coordination / Worker 协调
- Speculative decoding / 推测解码
- Frozen-KV MTP path / Frozen-KV MTP 路径
- Integration with model workers / 与模型 worker 集成

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.managers.tp_worker`
- `sglang.srt.server_args`
- `sglang.srt.speculative.frozen_kv_mtp_worker`
### External / 外部
- `__future__`
- `typing` (stdlib)
