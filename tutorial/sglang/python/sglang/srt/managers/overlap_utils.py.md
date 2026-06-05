# overlap_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/overlap_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements overlap utils logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 overlap 工具 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-4: Import runtime dependencies / 导入运行时依赖
```python
from dataclasses import dataclass
from typing import TYPE_CHECKING, Optional
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 6-6: Import runtime dependencies / 导入运行时依赖
```python
import torch
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-9: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.speculative.spec_utils import spec_need_hidden_states
from sglang.srt.utils import is_cuda, is_hip
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 11-15: Provide supporting module logic / 提供辅助模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import ScheduleBatch
    from sglang.srt.managers.scheduler import GenerationBatchResult
    from sglang.srt.speculative.eagle_info import EagleDraftInput
    from sglang.srt.speculative.spec_info import SpeculativeAlgorithm
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 17-18: Provide supporting module logic / 提供辅助模块逻辑
```python
_is_cuda = is_cuda()
_is_hip = is_hip()
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 21-26: Implement resolve future token ids native / 实现resolve future Token ids native
```python
def _resolve_future_token_ids_native(input_ids, future_token_ids_map):
    input_ids[:] = torch.where(
        input_ids < 0,
        future_token_ids_map[torch.clamp(-input_ids, min=0)],
        input_ids,
    )
```
**EN:** This block implements the function `_resolve_future_token_ids_native(input_ids, future_token_ids_map)`. It focuses on handling the overlap utils responsibilities represented by `_resolve_future_token_ids_native`, providing reusable behavior for the overlap utils pipeline.
**CN:** 该代码块实现函数 `_resolve_future_token_ids_native(input_ids, future_token_ids_map)`。它围绕 `_resolve_future_token_ids_native` 所承担的 overlap 工具 相关职责展开，为对应处理链路提供可复用能力。

### Lines 29-39: Provide supporting module logic / 提供辅助模块逻辑
```python
if _is_cuda or _is_hip:
    from sglang.jit_kernel.resolve_future_token_ids import (
        resolve_future_token_ids_cuda,
    )

    _resolve_future_token_ids = resolve_future_token_ids_cuda
else:
    _resolve_future_token_ids = _resolve_future_token_ids_native


@dataclass
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 40-42: Define class FutureIndices / 定义类 FutureIndices
```python
class FutureIndices:
    indices: torch.Tensor
    interval: Optional[slice] = None
```
**EN:** This block declares the class `FutureIndices`. It centers on coordinating overlap utils behavior.
**CN:** 该代码块声明类 `FutureIndices`。它负责承载与 overlap 工具 相关的核心状态与行为。

### Lines 45-45: Provide supporting module logic / 提供辅助模块逻辑
```python
class FutureMap:
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 46-81: Initialize FutureMap / 初始化 FutureMap
```python
    def __init__(
        self,
        max_running_requests: int,
        chunked_prefill_size: int,
        context_len: int,
        device: torch.device,
        spec_algo: SpeculativeAlgorithm,
    ):
        # FIXME: the calculation of future_limit and future_buffer_len maybe too conservative
        self.future_ct = 0

        # Circular buffer layout (wraps in this order):
        # Running decode batch -> Prefill chunk 1 -> ... -> Prefill chunk N
        # A running decode batch's result will be resolved after all prefill chunks are done.
        # reserve `max_num_chunks` extra future slots on top of `max_running_requests * 3`.
        max_num_chunks = (
            (context_len + chunked_prefill_size - 1) // chunked_prefill_size
            if chunked_prefill_size
            else 0
        )
        self.future_limit = max_running_requests * (3 + max_num_chunks)
        # Adding 2 * max_running_requests to future_limit ensures the buffer is sufficiently large.
        self.future_buffer_len = self.future_limit + 2 * max_running_requests
        self.device = device
        self.spec_algo = spec_algo

        if self.spec_algo.is_none():
            # For non-speculative decoding, we only need to store the token ids.
            self.buf_initialized = True
            self.token_ids_buf = torch.empty(
                (self.future_buffer_len,), dtype=torch.int64, device=self.device
            )
        else:
            # For speculative decoding, we lazily initialize the buffers
            # This is to make the shape derivation easier.
            self.buf_initialized = False
```
**EN:** This block implements the initializer `__init__(max_running_requests, chunked_prefill_size, context_len, device, spec_algo)` for `FutureMap`. It prepares the object state and connects the instance to the surrounding overlap utils workflow.
**CN:** 该代码块实现 `FutureMap` 的初始化方法 `__init__(max_running_requests, chunked_prefill_size, context_len, device, spec_algo)`。它负责准备对象状态，并把实例接入 overlap 工具 相关的运行流程。

### Lines 83-119: Implement lazy init buf / 实现lazy init buf
```python
    def _lazy_init_buf(self, draft_input: EagleDraftInput):
        self.buf_initialized = True

        # Get a reference for each tensor
        topk_p0 = draft_input.topk_p[0]
        topk_index0 = draft_input.topk_index[0]
        bonus_token0 = draft_input.bonus_tokens[0]
        new_seq_lens0 = draft_input.new_seq_lens[0]

        self.topk_p_buf = torch.empty(
            (self.future_buffer_len, *topk_p0.shape),
            dtype=topk_p0.dtype,
            device=self.device,
        )
        self.topk_index_buf = torch.empty(
            (self.future_buffer_len, *topk_index0.shape),
            dtype=topk_index0.dtype,
            device=self.device,
        )
        self.bonus_tokens_buf = torch.empty(
            (self.future_buffer_len, *bonus_token0.shape),
            dtype=bonus_token0.dtype,
            device=self.device,
        )
        self.new_seq_lens_buf = torch.empty(
            (self.future_buffer_len, *new_seq_lens0.shape),
            dtype=new_seq_lens0.dtype,
            device=self.device,
        )

        if spec_need_hidden_states():
            hidden_states0 = draft_input.hidden_states[0]
            self.hidden_states_buf = torch.empty(
                (self.future_buffer_len, *hidden_states0.shape),
                dtype=hidden_states0.dtype,
                device=self.device,
            )
```
**EN:** This block implements the method `_lazy_init_buf(draft_input)` on `FutureMap`. It focuses on handling the overlap utils responsibilities represented by `_lazy_init_buf`, so the class can advance the overlap utils workflow in a self-contained way.
**CN:** 该代码块实现 `FutureMap` 上的方法 `_lazy_init_buf(draft_input)`。它围绕 `_lazy_init_buf` 所承担的 overlap 工具 相关职责展开，使该类能够独立推进相应流程。

### Lines 121-128: Implement alloc future indices / 实现alloc future indices
```python
    def alloc_future_indices(self, bs: int) -> FutureIndices:
        """Update the circular buffer pointer and allocate future indices."""
        cur_future_ct = self.future_ct
        self.future_ct = (cur_future_ct + bs) % self.future_limit
        start = cur_future_ct + 1
        end = cur_future_ct + 1 + bs
        indices = torch.arange(start, end, dtype=torch.int64, device=self.device)
        return FutureIndices(indices=indices, interval=slice(start, end))
```
**EN:** This block implements the method `alloc_future_indices(bs)` on `FutureMap`. It focuses on Update the circular buffer pointer and allocate future indices., so the class can advance the overlap utils workflow in a self-contained way.
**CN:** 该代码块实现 `FutureMap` 上的方法 `alloc_future_indices(bs)`。它围绕 `alloc_future_indices` 所承担的 overlap 工具 相关职责展开，使该类能够独立推进相应流程。

### Lines 130-151: Implement resolve future / 实现resolve future
```python
    def resolve_future(self, batch: ScheduleBatch):
        if self.spec_algo.is_none():
            _resolve_future_token_ids(batch.input_ids, self.token_ids_buf)
        else:
            # TODO(lsyin): write future indices into spec_info.future_indices
            draft_input: EagleDraftInput = batch.spec_info
            if draft_input is None:
                # FIXME(lsyin): No future exists, only for prefill batch, not compatible with mixed mode
                return
            indices = draft_input.future_indices.indices
            # The indices tensor was allocated on the default stream but is
            # used here on the forward stream. Meanwhile, the old spec_info
            # holding this tensor will lose all Python references (replaced at
            # batch.spec_info), so the caching allocator (torch GC) could
            # reclaim the memory before the GPU finishes reading it.
            indices.record_stream(torch.get_device_module(self.device).current_stream())
            draft_input.topk_p = self.topk_p_buf[indices]
            draft_input.topk_index = self.topk_index_buf[indices]
            draft_input.bonus_tokens = self.bonus_tokens_buf[indices]
            draft_input.new_seq_lens = self.new_seq_lens_buf[indices]
            if spec_need_hidden_states():
                draft_input.hidden_states = self.hidden_states_buf[indices]
```
**EN:** This block implements the method `resolve_future(batch)` on `FutureMap`. It focuses on handling the overlap utils responsibilities represented by `resolve_future`, so the class can advance the overlap utils workflow in a self-contained way.
**CN:** 该代码块实现 `FutureMap` 上的方法 `resolve_future(batch)`。它围绕 `resolve_future` 所承担的 overlap 工具 相关职责展开，使该类能够独立推进相应流程。

### Lines 153-158: Implement is empty slice / 实现is empty slice
```python
    def is_empty_slice(self, s: slice) -> bool:
        start, stop, step = s.indices(self.future_buffer_len)
        if step > 0:
            return start >= stop
        else:
            return start <= stop
```
**EN:** This block implements the method `is_empty_slice(s)` on `FutureMap`. It focuses on handling the overlap utils responsibilities represented by `is_empty_slice`, so the class can advance the overlap utils workflow in a self-contained way.
**CN:** 该代码块实现 `FutureMap` 上的方法 `is_empty_slice(s)`。它围绕 `is_empty_slice` 所承担的 overlap 工具 相关职责展开，使该类能够独立推进相应流程。

### Lines 160-171: Implement store to map / 实现store to map
```python
    def store_to_map(
        self, future_indices: FutureIndices, batch_result: GenerationBatchResult
    ):
        if self.spec_algo.is_none():
            intv = future_indices.interval
            if self.is_empty_slice(intv):
                # idle indices in dp attention do not need store info
                return
            self.token_ids_buf[intv] = batch_result.next_token_ids
        else:
            draft_input: EagleDraftInput = batch_result.next_draft_input
            self.store_to_map_for_new_batch(future_indices, draft_input)
```
**EN:** This block implements the method `store_to_map(future_indices, batch_result)` on `FutureMap`. It focuses on handling the overlap utils responsibilities represented by `store_to_map`, so the class can advance the overlap utils workflow in a self-contained way.
**CN:** 该代码块实现 `FutureMap` 上的方法 `store_to_map(future_indices, batch_result)`。它围绕 `store_to_map` 所承担的 overlap 工具 相关职责展开，使该类能够独立推进相应流程。

### Lines 173-189: Implement store to map for new batch / 实现store to map for new 批处理
```python
    def store_to_map_for_new_batch(
        self, future_indices: FutureIndices, draft_input: EagleDraftInput
    ):
        intv = future_indices.interval
        if self.is_empty_slice(intv):
            # idle indices in dp attention do not need store info
            return

        if not self.buf_initialized:
            self._lazy_init_buf(draft_input)

        self.topk_p_buf[intv] = draft_input.topk_p
        self.topk_index_buf[intv] = draft_input.topk_index
        self.bonus_tokens_buf[intv] = draft_input.bonus_tokens
        self.new_seq_lens_buf[intv] = draft_input.new_seq_lens
        if spec_need_hidden_states():
            self.hidden_states_buf[intv] = draft_input.hidden_states
```
**EN:** This block implements the method `store_to_map_for_new_batch(future_indices, draft_input)` on `FutureMap`. It focuses on handling the overlap utils responsibilities represented by `store_to_map_for_new_batch`, so the class can advance the overlap utils workflow in a self-contained way.
**CN:** 该代码块实现 `FutureMap` 上的方法 `store_to_map_for_new_batch(future_indices, draft_input)`。它围绕 `store_to_map_for_new_batch` 所承担的 overlap 工具 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: FutureIndices, FutureMap
- **Main callables / 主要可调用对象**: _resolve_future_token_ids_native
- **Domain focus / 领域焦点**: overlap utils / overlap 工具
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: dataclasses, typing
- **Third-party / 第三方库**: __future__, torch
- **Local Modules / 本地模块**: sglang.jit_kernel.resolve_future_token_ids, sglang.srt.managers.schedule_batch, sglang.srt.managers.scheduler, sglang.srt.speculative.eagle_info, sglang.srt.speculative.spec_info, sglang.srt.speculative.spec_utils, sglang.srt.utils
