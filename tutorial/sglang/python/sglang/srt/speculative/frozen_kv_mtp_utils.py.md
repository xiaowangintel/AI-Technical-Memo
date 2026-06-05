# frozen_kv_mtp_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/frozen_kv_mtp_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements speculative decoding infrastructure, such as workers, metadata objects, CUDA-graph runners, or shared utilities. It coordinates draft-model execution with target-model verification to improve throughput. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31: Module header, imports, and shared constants
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

from contextlib import contextmanager
from typing import Tuple

import torch

from sglang.srt.layers.logits_processor import LogitsProcessorOutput
from sglang.srt.managers.schedule_batch import ScheduleBatch
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.speculative.frozen_kv_mtp_info import (
    FrozenKVMTPContext,
    FrozenKVMTPDraftExtendInput,
    FrozenKVMTPDraftInput,
)
from sglang.srt.speculative.spec_utils import fast_topk
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 32-48: frozen kv target view function
```python
@contextmanager
def frozen_kv_target_view(forward_batch: ForwardBatch, kv_context: FrozenKVMTPContext):
    """Build attention metadata against committed target-prefix geometry."""
    if kv_context is None:
        raise RuntimeError(
            "Frozen-KV MTP target view called before the model was bound; "
            "bind the frozen KV context first."
        )
    saved_spec_info = forward_batch.spec_info
    saved_kv_pool = forward_batch.token_to_kv_pool
    forward_batch.spec_info = None
    forward_batch.token_to_kv_pool = kv_context.target_token_to_kv_pool
    try:
        yield
    finally:
        forward_batch.spec_info = saved_spec_info
        forward_batch.token_to_kv_pool = saved_kv_pool
```
**EN:** This block uses `frozen_kv_target_view` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `frozen_kv_target_view` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 51-63: target kv pool view function
```python
@contextmanager
def target_kv_pool_view(forward_batch: ForwardBatch, kv_context: FrozenKVMTPContext):
    if kv_context is None:
        raise RuntimeError(
            "Frozen-KV MTP target KV pool view called before the model was bound; "
            "bind the frozen KV context first."
        )
    saved_kv_pool = forward_batch.token_to_kv_pool
    forward_batch.token_to_kv_pool = kv_context.target_token_to_kv_pool
    try:
        yield
    finally:
        forward_batch.token_to_kv_pool = saved_kv_pool
```
**EN:** This block uses `target_kv_pool_view` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `target_kv_pool_view` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 66-82: set frozen kv positions function
```python
def set_frozen_kv_positions(forward_batch: ForwardBatch, topk: int) -> None:
    """Rope phase = last written target slot, not advanced per draft step."""
    seq_lens = forward_batch.seq_lens
    positions = torch.clamp(seq_lens - 1, min=0).to(torch.int64)
    if (
        topk > 1
        and forward_batch.positions is not None
        and forward_batch.positions.numel() == positions.numel() * topk
    ):
        positions = positions.repeat_interleave(topk, dim=0)
    if forward_batch.positions is None:
        forward_batch.positions = positions
    else:
        if forward_batch.positions.shape == positions.shape:
            forward_batch.positions.copy_(positions)
        else:
            forward_batch.positions = positions
```
**EN:** This block uses `set_frozen_kv_positions` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `set_frozen_kv_positions` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 85-120: expand for topk draft function
```python
def expand_for_topk_draft(forward_batch: ForwardBatch, topk: int) -> None:
    """Repeat committed-prefix metadata for the active ``B * topk`` frontier."""
    if topk == 1 or forward_batch.batch_size == 0:
        return

    if forward_batch.batch_size != forward_batch.seq_lens.shape[0]:
        raise RuntimeError(
            "Frozen-KV MTP topk expansion expects an unexpanded forward "
            "batch where batch_size == len(seq_lens)."
        )

    forward_batch.batch_size *= topk
    forward_batch.req_pool_indices = forward_batch.req_pool_indices.repeat_interleave(
        topk, dim=0
    )
    forward_batch.seq_lens = forward_batch.seq_lens.repeat_interleave(topk, dim=0)
    if forward_batch.seq_lens_cpu is not None:
        forward_batch.seq_lens_cpu = forward_batch.seq_lens_cpu.repeat_interleave(
            topk, dim=0
        )
        forward_batch.seq_lens_sum = forward_batch.seq_lens_cpu.sum().item()
    else:
        forward_batch.seq_lens_sum = torch.sum(forward_batch.seq_lens).item()

    positions = torch.clamp(forward_batch.seq_lens - 1, min=0).to(torch.int64)
    forward_batch.positions = positions
    forward_batch.num_token_non_padded_cpu = positions.numel()
    if forward_batch.num_token_non_padded is not None:
        forward_batch.num_token_non_padded.fill_(positions.numel())
    if (
        forward_batch.mrope_positions is not None
        and forward_batch.mrope_positions.shape[-1] * topk == positions.numel()
    ):
        forward_batch.mrope_positions = forward_batch.mrope_positions.repeat_interleave(
            topk, dim=-1
        )
```
**EN:** This block uses `expand_for_topk_draft` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `expand_for_topk_draft` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 123-124: position for batch function
```python
def position_for_batch(batch: ScheduleBatch) -> torch.Tensor:
    return torch.clamp(batch.seq_lens - 1, min=0).to(torch.int64)
```
**EN:** This block uses `position_for_batch` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `position_for_batch` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 127-134: select last extend hidden function
```python
def select_last_extend_hidden(
    batch: ScheduleBatch, hidden_states: torch.Tensor
) -> torch.Tensor:
    if hidden_states.shape[0] == batch.batch_size():
        return hidden_states
    lens = torch.tensor(batch.extend_lens, device=hidden_states.device)
    last_indices = torch.cumsum(lens, dim=0) - 1
    return hidden_states[last_indices.to(torch.long)]
```
**EN:** This block uses `select_last_extend_hidden` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `select_last_extend_hidden` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 137-145: select last verified seed function
```python
def select_last_verified_seed(
    draft_input: FrozenKVMTPDraftExtendInput,
) -> Tuple[torch.Tensor, torch.Tensor]:
    counts = draft_input.num_accept_tokens.to(torch.long)
    last_indices = torch.cumsum(counts, dim=0) - 1
    return (
        draft_input.input_ids[last_indices],
        draft_input.hidden_states[last_indices],
    )
```
**EN:** This block uses `select_last_verified_seed` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `select_last_verified_seed` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 148-153: capture for decode function
```python
def capture_for_decode(
    logits_output: LogitsProcessorOutput, draft_input: FrozenKVMTPDraftInput, topk: int
) -> None:
    probs = torch.softmax(logits_output.next_token_logits, dim=-1)
    draft_input.topk_p, draft_input.topk_index = fast_topk(probs, topk, dim=-1)
    draft_input.hidden_states = logits_output.hidden_states
```
**EN:** This block uses `capture_for_decode` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `capture_for_decode` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Speculative decoding / 推测解码
- Frozen-KV MTP path / Frozen-KV MTP 路径
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.layers.logits_processor`
- `sglang.srt.managers.schedule_batch`
- `sglang.srt.model_executor.forward_batch_info`
- `sglang.srt.speculative.frozen_kv_mtp_info`
- `sglang.srt.speculative.spec_utils`
### External / 外部
- `__future__`
- `torch`
- `contextlib` (stdlib)
- `typing` (stdlib)
