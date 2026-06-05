# staging_handler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/common/staging_handler.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file coordinates staging-buffer ownership, reuse, and copy scheduling. It turns lower-level buffers into a higher-level transfer workflow. / 该文件协调暂存缓冲区的所有权、复用与拷贝调度，把底层缓冲区组合成更高层次的传输流程。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: Module-level constants and helper logic
```python
"""
Staging handler for heterogeneous TP KV cache transfer.

Isolates staging scatter lifecycle from decode.py and conn.py.
Generic (backend-agnostic) code is at the top; mooncake-specific
protocol code is at the bottom.
"""

from __future__ import annotations

import dataclasses
import logging
import struct
import threading
from typing import TYPE_CHECKING, List, Optional, Tuple

import torch

logger = logging.getLogger(__name__)

if TYPE_CHECKING:
    from sglang.srt.disaggregation.decode import DecodeRequest


# ======================================================================
# Generic staging state and handler (backend-agnostic)
# ======================================================================


```
**EN:** This block contains module-level constants, helpers, or documentation for staging buffer coordination. It prepares shared state that later classes and functions build on. Notable operations include `Generic`, `getLogger`, `handler`.
**CN:** 这一段包含与暂存缓冲区协调相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `Generic`、`getLogger`、`handler`。

### Lines 30-31: Class `DecodeStagingContext` declaration
```python
@dataclasses.dataclass
class DecodeStagingContext:
```
**EN:** This block declares the class `DecodeStagingContext` and establishes its responsibility inside staging buffer coordination. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `DecodeStagingContext`.
**CN:** 这一段声明了类 `DecodeStagingContext`，并说明它在暂存缓冲区协调中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `DecodeStagingContext`。

### Lines 32-36: Supporting state inside `DecodeStagingContext`
```python
    """Staging-specific context for decode mode."""

    allocator: object = None
    room_bootstrap: dict = dataclasses.field(default_factory=dict)
    room_receivers: dict = dataclasses.field(default_factory=dict)
```
**EN:** This block adds supporting state or helper logic inside `DecodeStagingContext`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `field`.
**CN:** 这一段为 `DecodeStagingContext` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `field`。

### Lines 39-40: Class `PrefillStagingContext` declaration
```python
@dataclasses.dataclass
class PrefillStagingContext:
```
**EN:** This block declares the class `PrefillStagingContext` and establishes its responsibility inside staging buffer coordination. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `PrefillStagingContext`.
**CN:** 这一段声明了类 `PrefillStagingContext`，并说明它在暂存缓冲区协调中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `PrefillStagingContext`。

### Lines 41-53: Supporting state inside `PrefillStagingContext`
```python
    """Staging-specific context for prefill mode."""

    buffers: list = dataclasses.field(default_factory=list)
    remote_watermarks: dict = dataclasses.field(default_factory=dict)
    watermark_cv: threading.Condition = dataclasses.field(
        default_factory=threading.Condition
    )
    # (room, chunk_idx, session_id) keys for chunks already requested.
    prefetch_requested: set = dataclasses.field(default_factory=set)
    # Rooms that have already had their full prefetch fan-out triggered. Used
    # to short-circuit per-room prefetch entry on every chunk after the first.
    prefetched_rooms: set = dataclasses.field(default_factory=set)
    prefetch_sockets: dict = dataclasses.field(default_factory=dict)
```
**EN:** This block adds supporting state or helper logic inside `PrefillStagingContext`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `field`.
**CN:** 这一段为 `PrefillStagingContext` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `field`。

### Lines 56-56: Class `DecodeStagingHandler` declaration
```python
class DecodeStagingHandler:
```
**EN:** This block declares the class `DecodeStagingHandler` and establishes its responsibility inside staging buffer coordination. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `DecodeStagingHandler`.
**CN:** 这一段声明了类 `DecodeStagingHandler`，并说明它在暂存缓冲区协调中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `DecodeStagingHandler`。

### Lines 57-63: Supporting state inside `DecodeStagingHandler`
```python
    """Decode-side staging scatter lifecycle manager.

    Scatter submission can be called from the decode_thread (background) as
    soon as all writers/ranks have arrived, while event checking and freeing
    always run on the scheduler main thread.
    """

```
**EN:** This block adds supporting state or helper logic inside `DecodeStagingHandler`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `decode_thread`.
**CN:** 这一段为 `DecodeStagingHandler` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `decode_thread`。

### Lines 64-82: Method `__init__`
```python
    def __init__(
        self,
        kv_manager,
        staging_allocator,
        kv_buffer_info: dict,
        decode_tp: int,
        total_kv_heads: int,
        tp_rank: int,
        scheduler,
    ):
        self.kv_manager = kv_manager
        self.staging_allocator = staging_allocator
        self.kv_buffer_info = kv_buffer_info
        self.decode_tp = decode_tp
        self.total_kv_heads = total_kv_heads
        self.tp_rank = tp_rank
        self.scheduler = scheduler
        self._room_to_decode_req: dict = {}
        self._wm_subscribers: dict = {}
```
**EN:** This block defines the method `__init__` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 84-90: Method `register_wm_subscriber`
```python
    def register_wm_subscriber(self, receiver, session_id: str) -> None:
        """Register a prefill's bootstrap connection for watermark broadcasts."""
        if receiver is None or not getattr(receiver, "bootstrap_infos", None):
            return
        key = tuple(str(bi) for bi in receiver.bootstrap_infos)
        if key not in self._wm_subscribers:
            self._wm_subscribers[key] = (receiver, session_id)
```
**EN:** This block defines the method `register_wm_subscriber` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `register_wm_subscriber`.
**CN:** 这一段定义了method `register_wm_subscriber`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `register_wm_subscriber`。

### Lines 92-97: Method `num_writers_for`
```python
    def num_writers_for(self, decode_req) -> int:
        """Compute num_writers for a specific request based on its prefill TP."""
        prefill_tp = decode_req.kv_receiver.prefill_info.attn_tp_size
        if prefill_tp > self.decode_tp:
            return prefill_tp // max(1, self.decode_tp)
        return 1
```
**EN:** This block defines the method `num_writers_for` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `num_writers_for`.
**CN:** 这一段定义了method `num_writers_for`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `num_writers_for`。

### Lines 99-101: Method `create` signature and setup
```python
    @classmethod
    def create(cls, kv_manager, scheduler, tp_rank: int) -> "DecodeStagingHandler":
        """Factory: create handler. Raises if staging infra is missing."""
```
**EN:** This block defines the method `create` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `create`.
**CN:** 这一段定义了method `create`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `create`。

### Lines 102-118: Method `create` logic (part 1)
```python
        staging_allocator = kv_manager._staging_ctx.allocator
        if staging_allocator is None:
            raise RuntimeError(
                "Staging is enabled but kv_manager._staging_ctx.allocator is None. "
                "Check that the transfer backend correctly initializes the staging allocator."
            )
        kv_buffer_info = kv_manager.kv_buffer_tensors
        if kv_buffer_info is None:
            raise RuntimeError(
                "Staging is enabled but kv_manager.kv_buffer_tensors is None. "
                "Check that set_kv_buffer_tensors() was called during kv_manager init."
            )
        decode_tp = kv_manager.attn_tp_size

        from sglang.srt.disaggregation.common.staging_buffer import (
            resolve_total_kv_heads,
        )
```
**EN:** This block continues `create` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `RuntimeError`, `set_kv_buffer_tensors`, `import`.
**CN:** 这一段延续了 `create` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `RuntimeError`、`set_kv_buffer_tensors`、`import`。

### Lines 119-129: Method `create` logic (part 2)
```python

        total_kv_heads = resolve_total_kv_heads(kv_manager.kv_args, decode_tp)
        return cls(
            kv_manager=kv_manager,
            staging_allocator=staging_allocator,
            kv_buffer_info=kv_buffer_info,
            decode_tp=decode_tp,
            total_kv_heads=total_kv_heads,
            tp_rank=tp_rank,
            scheduler=scheduler,
        )
```
**EN:** This block continues `create` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `resolve_total_kv_heads`, `cls`.
**CN:** 这一段延续了 `create` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `resolve_total_kv_heads`、`cls`。

### Lines 130-134: Supporting state inside `DecodeStagingHandler`
```python

    # ------------------------------------------------------------------
    # Registration: called from main thread (DecodeTransferQueue)
    # ------------------------------------------------------------------

```
**EN:** This block adds supporting state or helper logic inside `DecodeStagingHandler`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `thread`.
**CN:** 这一段为 `DecodeStagingHandler` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `thread`。

### Lines 135-136: Method `register_decode_req`
```python
    def register_decode_req(self, room: int, decode_req: "DecodeRequest") -> None:
        self._room_to_decode_req[room] = decode_req
```
**EN:** This block defines the method `register_decode_req` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `register_decode_req`.
**CN:** 这一段定义了method `register_decode_req`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `register_decode_req`。

### Lines 138-139: Method `unregister_decode_req`
```python
    def unregister_decode_req(self, room: int) -> None:
        self._room_to_decode_req.pop(room, None)
```
**EN:** This block defines the method `unregister_decode_req` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `unregister_decode_req`. Notable operations include `pop`.
**CN:** 这一段定义了method `unregister_decode_req`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `unregister_decode_req`。 值得注意的操作包括 `pop`。

### Lines 140-144: Supporting state inside `DecodeStagingHandler`
```python

    # ------------------------------------------------------------------
    # Scatter submission: called from decode_thread (background)
    # ------------------------------------------------------------------

```
**EN:** This block adds supporting state or helper logic inside `DecodeStagingHandler`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `decode_thread`.
**CN:** 这一段为 `DecodeStagingHandler` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `decode_thread`。

### Lines 145-152: Method `submit_chunk_scatter` signature and setup
```python
    def submit_chunk_scatter(
        self, room: int, chunk_idx: int, page_start: int, num_pages: int
    ) -> bool:
        """Submit scatter for an intermediate chunk whose writers all arrived.

        Called from decode_thread.  Records a CUDA event on decode_req so
        the main thread can later check completion and free the allocation.
        """
```
**EN:** This block defines the method `submit_chunk_scatter` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `submit_chunk_scatter`.
**CN:** 这一段定义了method `submit_chunk_scatter`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `submit_chunk_scatter`。

### Lines 153-168: Method `submit_chunk_scatter` logic (part 1)
```python
        decode_req = self._room_to_decode_req.get(room)
        if decode_req is None:
            logger.warning(
                "[STAGING] submit_chunk_scatter: room=%s not registered, "
                "chunk_idx=%s. This should not happen if register_decode_req "
                "is called at kv_receiver.init() time.",
                room,
                chunk_idx,
            )
            return False
        chunk_infos = getattr(decode_req.kv_receiver, "chunk_staging_infos", [])
        if chunk_idx >= len(chunk_infos):
            return False
        alloc_id, staging_offset, _, _, _ = chunk_infos[chunk_idx]
        if staging_offset < 0 or alloc_id < 0:
            return False
```
**EN:** This block continues `submit_chunk_scatter` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `get`, `warning`, `init`.
**CN:** 这一段延续了 `submit_chunk_scatter` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `get`、`warning`、`init`。

### Lines 169-185: Method `submit_chunk_scatter` logic (part 2)
```python

        ok = self._scatter_region(staging_offset, page_start, num_pages, decode_req)
        if ok:
            event = torch.cuda.Event()
            event.record(self.staging_allocator._scatter_stream)
            if not hasattr(decode_req, "_chunk_events"):
                decode_req._chunk_events = []
            decode_req._chunk_events.append((event, alloc_id))
            chunk_infos[chunk_idx] = (-1, -1, 0, -1, 0)
        else:
            logger.warning(
                "submit_chunk_scatter failed room=%s chunk_idx=%s tp_rank=%s",
                room,
                chunk_idx,
                self.tp_rank,
            )
        return ok
```
**EN:** This block continues `submit_chunk_scatter` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `_scatter_region`, `Event`, `record`, `append`.
**CN:** 这一段延续了 `submit_chunk_scatter` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `_scatter_region`、`Event`、`record`、`append`。

### Lines 187-189: Method `is_staging_room`
```python
    def is_staging_room(self, room: int) -> bool:
        """Check if a room is registered for staging scatter."""
        return room in self._room_to_decode_req
```
**EN:** This block defines the method `is_staging_room` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `is_staging_room`.
**CN:** 这一段定义了method `is_staging_room`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `is_staging_room`。

### Lines 191-205: Method `handle_chunk_arrived` signature and setup
```python
    def handle_chunk_arrived(
        self,
        room: int,
        chunk_idx: int,
        page_start: int,
        num_pages: int,
        writer_id: str,
        chunk_writer_counts: dict,
    ) -> bool:
        """Process a staging chunk arrival from any transport (NIXL RDMA notif or ZMQ CHUNK_READY).

        Accumulates writer arrivals in *chunk_writer_counts* and submits scatter
        once all writers for this chunk have reported in. Returns True if scatter
        was submitted.
        """
```
**EN:** This block defines the method `handle_chunk_arrived` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `handle_chunk_arrived`. Notable operations include `transport`.
**CN:** 这一段定义了method `handle_chunk_arrived`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `handle_chunk_arrived`。 值得注意的操作包括 `transport`。

### Lines 206-221: Method `handle_chunk_arrived` logic (part 1)
```python
        chunk_writer_counts[room][chunk_idx].append((page_start, num_pages, writer_id))
        decode_req = self._room_to_decode_req.get(room)
        if decode_req is None:
            logger.warning(
                "Staging chunk arrived for unregistered room=%s chunk=%d, skipping",
                room,
                chunk_idx,
            )
            return False
        writers_arrived = len(chunk_writer_counts[room][chunk_idx])
        num_writers = self.num_writers_for(decode_req)
        if writers_arrived >= num_writers:
            self.submit_chunk_scatter(room, chunk_idx, page_start, num_pages)
            del chunk_writer_counts[room][chunk_idx]
            return True
        return False
```
**EN:** This block continues `handle_chunk_arrived` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `append`, `get`, `warning`, `num_writers_for`.
**CN:** 这一段延续了 `handle_chunk_arrived` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `append`、`get`、`warning`、`num_writers_for`。

### Lines 223-229: Method `submit_last_scatter_async` signature and setup
```python
    def submit_last_scatter_async(self, room: int) -> bool:
        """Submit scatter for the last chunk when all ranks report Success.

        Called from decode_thread.  Sets ``_scatter_event`` **before**
        ``_staging_last_scatter_submitted`` so the main thread sees the
        event when it checks the flag (CPython GIL guarantees ordering).
        """
```
**EN:** This block defines the method `submit_last_scatter_async` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `submit_last_scatter_async`. Notable operations include `flag`.
**CN:** 这一段定义了method `submit_last_scatter_async`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `submit_last_scatter_async`。 值得注意的操作包括 `flag`。

### Lines 230-248: Method `submit_last_scatter_async` logic (part 1)
```python
        decode_req = self._room_to_decode_req.get(room)
        if decode_req is None:
            logger.warning(
                "[STAGING] submit_last_scatter_async: room=%s not registered. "
                "This should not happen if register_decode_req is called at "
                "kv_receiver.init() time.",
                room,
            )
            return False
        alloc_id = self._submit_last_scatter(decode_req)
        if alloc_id >= 0:
            event = torch.cuda.Event()
            event.record(self.staging_allocator._scatter_stream)
            decode_req._scatter_event = event
            decode_req._scatter_alloc_id = alloc_id
            decode_req._staging_last_scatter_submitted = True
        else:
            decode_req._staging_scatter_done = True
        return True
```
**EN:** This block continues `submit_last_scatter_async` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `get`, `warning`, `init`, `_submit_last_scatter`.
**CN:** 这一段延续了 `submit_last_scatter_async` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `get`、`warning`、`init`、`_submit_last_scatter`。

### Lines 249-253: Supporting state inside `DecodeStagingHandler`
```python

    # ------------------------------------------------------------------
    # Event check + free: called from main thread (pop_transferred)
    # ------------------------------------------------------------------

```
**EN:** This block adds supporting state or helper logic inside `DecodeStagingHandler`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `thread`.
**CN:** 这一段为 `DecodeStagingHandler` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `thread`。

### Lines 254-258: Method `is_done`
```python
    def is_done(self, decode_req: "DecodeRequest") -> bool:
        """Return True if staging scatter is complete for this request."""
        if not getattr(decode_req, "_staging_scatter_done", False):
            return False
        return not getattr(decode_req, "_chunk_events", None)
```
**EN:** This block defines the method `is_done` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `is_done`.
**CN:** 这一段定义了method `is_done`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `is_done`。

### Lines 260-266: Method `advance_scatter` signature and setup
```python
    def advance_scatter(self, decode_req: "DecodeRequest") -> None:
        """Check CUDA events and free completed staging allocations.

        Scatter kernels have already been submitted by the decode_thread
        (via submit_chunk_scatter / submit_last_scatter_async).  This
        method only polls the recorded events and releases staging memory.
        """
```
**EN:** This block defines the method `advance_scatter` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `advance_scatter`. Notable operations include `decode_thread`.
**CN:** 这一段定义了method `advance_scatter`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `advance_scatter`。 值得注意的操作包括 `decode_thread`。

### Lines 267-284: Method `advance_scatter` logic (part 1)
```python
        room = decode_req.req.bootstrap_room
        chunk_events = getattr(decode_req, "_chunk_events", None)
        if chunk_events:
            for i in range(len(chunk_events) - 1, -1, -1):
                event, alloc_id = chunk_events[i]
                if event.query():
                    chunk_events.pop(i)
                    self._free_and_send_watermark(alloc_id, decode_req)

        if not getattr(decode_req, "_staging_last_scatter_submitted", False):
            return

        event = getattr(decode_req, "_scatter_event", None)
        if event is not None and event.query():
            self._free_and_send_watermark(decode_req._scatter_alloc_id, decode_req)
            decode_req._scatter_event = None
            decode_req._scatter_alloc_id = -1
            decode_req._staging_scatter_done = True
```
**EN:** This block continues `advance_scatter` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `query`, `pop`, `_free_and_send_watermark`.
**CN:** 这一段延续了 `advance_scatter` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `query`、`pop`、`_free_and_send_watermark`。

### Lines 285-289: Supporting state inside `DecodeStagingHandler`
```python

    # ------------------------------------------------------------------
    # Internal methods
    # ------------------------------------------------------------------

```
**EN:** This block adds supporting state or helper logic inside `DecodeStagingHandler`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `DecodeStagingHandler` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 290-302: Method `_scatter_region` signature and setup
```python
    def _scatter_region(
        self,
        staging_offset: int,
        page_start: int,
        num_pages: int,
        decode_req: "DecodeRequest",
    ) -> bool:
        """Submit scatter kernels for a staging region to scatter_stream.

        May be called from the decode_thread (background).  All GPU work
        runs on scatter_stream so that the decode_thread never blocks on
        the default stream (which carries the main-thread forward pass).
        """
```
**EN:** This block defines the method `_scatter_region` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `_scatter_region`. Notable operations include `decode_thread`, `stream`.
**CN:** 这一段定义了method `_scatter_region`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `_scatter_region`。 值得注意的操作包括 `decode_thread`、`stream`。

### Lines 303-325: Method `_scatter_region` logic (part 1)
```python
        from sglang.srt.disaggregation.common.staging_buffer import (
            scatter_staging_to_kv,
        )

        k_buffers = self.kv_buffer_info["k_buffers"]
        v_buffers = self.kv_buffer_info["v_buffers"]
        page_size = self.kv_buffer_info["page_size"]
        dst_tp_rank = self.kv_manager.kv_args.engine_rank % self.decode_tp

        device = k_buffers[0].device
        torch.cuda.set_device(device)

        if not hasattr(self.staging_allocator, "_scatter_stream"):
            self.staging_allocator._scatter_stream = torch.cuda.Stream(device=device)

        scatter_stream = self.staging_allocator._scatter_stream

        staging_view = self.staging_allocator.buffer.buffer[staging_offset:]

        req_pool_idx = decode_req.req.req_pool_idx
        token_start = page_start * page_size
        token_end = token_start + num_pages * page_size
        prefill_tp = decode_req.kv_receiver.prefill_info.attn_tp_size
```
**EN:** This block continues `_scatter_region` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `import`, `set_device`, `Stream`.
**CN:** 这一段延续了 `_scatter_region` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `import`、`set_device`、`Stream`。

### Lines 326-348: Method `_scatter_region` logic (part 2)
```python

        with torch.cuda.stream(scatter_stream):
            kv_indices = self.scheduler.req_to_token_pool.req_to_token[
                req_pool_idx, token_start:token_end
            ]
            if page_size > 1:
                page_idx_tensor = kv_indices[::page_size] // page_size
            else:
                page_idx_tensor = kv_indices

            scatter_staging_to_kv(
                staging_view,
                k_buffers,
                v_buffers,
                page_idx_tensor,
                page_size,
                prefill_tp,
                self.decode_tp,
                dst_tp_rank,
                self.total_kv_heads,
            )

        return True
```
**EN:** This block continues `_scatter_region` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `stream`, `scatter_staging_to_kv`.
**CN:** 这一段延续了 `_scatter_region` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `stream`、`scatter_staging_to_kv`。

### Lines 350-370: Method `_submit_last_scatter`
```python
    def _submit_last_scatter(self, decode_req: "DecodeRequest") -> int:
        """Submit scatter for the last chunk. Returns alloc_id >= 0, or -1."""
        receiver = decode_req.kv_receiver
        chunk_infos = getattr(receiver, "chunk_staging_infos", [])
        if not chunk_infos:
            return -1

        last_info = chunk_infos[-1]
        alloc_id, staging_offset, _, _, last_num_pages = last_info
        if staging_offset < 0 or alloc_id < 0:
            return -1

        seq_len = len(decode_req.req.origin_input_ids)
        ps = self.scheduler.token_to_kv_pool_allocator.page_size
        total_pages = (seq_len + ps - 1) // ps
        page_start = total_pages - last_num_pages

        ok = self._scatter_region(
            staging_offset, page_start, last_num_pages, decode_req
        )
        return alloc_id if ok else -1
```
**EN:** This block defines the method `_submit_last_scatter` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `_submit_last_scatter`. Notable operations include `_scatter_region`.
**CN:** 这一段定义了method `_submit_last_scatter`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `_submit_last_scatter`。 值得注意的操作包括 `_scatter_region`。

### Lines 372-392: Method `_free_and_send_watermark`
```python
    def _free_and_send_watermark(
        self, alloc_id: int, decode_req: "DecodeRequest"
    ) -> None:
        """Free a staging allocation and broadcast watermark to all prefills."""
        self.staging_allocator.free(alloc_id)
        post_wm = self.staging_allocator.get_watermark()
        room = decode_req.req.bootstrap_room
        wm_round, wm_tail = post_wm
        wm_round_b = str(wm_round).encode("ascii")
        wm_tail_b = str(wm_tail).encode("ascii")
        for _key, (receiver, session_id) in list(self._wm_subscribers.items()):
            sid_b = session_id.encode("ascii")
            for bootstrap_info in receiver.bootstrap_infos:
                try:
                    sock, lock = receiver._connect_to_bootstrap_server(bootstrap_info)
                    with lock:
                        sock.send_multipart(
                            [b"WATERMARK", wm_round_b, wm_tail_b, sid_b]
                        )
                except Exception:
                    pass
```
**EN:** This block defines the method `_free_and_send_watermark` on `DecodeStagingHandler`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `_free_and_send_watermark`. Notable operations include `free`, `get_watermark`, `encode`, `items`.
**CN:** 这一段定义了method `_free_and_send_watermark`（属于 `DecodeStagingHandler`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `_free_and_send_watermark`。 值得注意的操作包括 `free`、`get_watermark`、`encode`、`items`。

### Lines 395-403: Function `is_watermark_ready`
```python
def is_watermark_ready(
    staging_state, session_id: str, alloc_round: int, alloc_end: int
) -> bool:
    """Non-blocking check: is the staging region safe to write?"""
    if alloc_round <= 0:
        return True
    prev_round = alloc_round - 1
    wm_round, wm_tail = staging_state.remote_watermarks.get(session_id, (0, 0))
    return prev_round < wm_round or (prev_round == wm_round and alloc_end <= wm_tail)
```
**EN:** This block defines the function `is_watermark_ready`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `is_watermark_ready`. Notable operations include `get`, `or`.
**CN:** 这一段定义了function `is_watermark_ready`，介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `is_watermark_ready`。 值得注意的操作包括 `get`、`or`。

### Lines 406-418: Function `handle_watermark_msg`
```python
def handle_watermark_msg(staging_ctx, msg_parts) -> None:
    """Process a WATERMARK message and update remote watermark tracking."""
    wm_round = int(msg_parts[1].decode("ascii"))
    wm_tail = int(msg_parts[2].decode("ascii"))
    wm_session = msg_parts[3].decode("ascii") if len(msg_parts) > 3 else ""
    with staging_ctx.watermark_cv:
        prev = staging_ctx.remote_watermarks.get(wm_session, (0, 0))
        if (wm_round, wm_tail) > prev:
            staging_ctx.remote_watermarks[wm_session] = (
                wm_round,
                wm_tail,
            )
        staging_ctx.watermark_cv.notify_all()
```
**EN:** This block defines the function `handle_watermark_msg`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `handle_watermark_msg`. Notable operations include `decode`, `get`, `notify_all`.
**CN:** 这一段定义了function `handle_watermark_msg`，介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `handle_watermark_msg`。 值得注意的操作包括 `decode`、`get`、`notify_all`。

### Lines 421-441: Function `handle_staging_rsp`
```python
def handle_staging_rsp(msg_parts, transfer_infos: dict) -> None:
    """Process a STAGING_RSP message and update transfer info with allocation."""
    stg_room = int(msg_parts[1].decode("ascii"))
    stg_chunk_idx = int(msg_parts[2].decode("ascii"))
    stg_offset = int(msg_parts[3].decode("ascii"))
    stg_round = int(msg_parts[4].decode("ascii"))
    stg_end = int(msg_parts[5].decode("ascii"))
    stg_session = msg_parts[6].decode("ascii")
    room_infos = transfer_infos.get(stg_room, {})
    tinfo = room_infos.get(stg_session)
    if tinfo is not None:
        if tinfo.staging is None:
            tinfo.staging = StagingTransferInfo()
        tinfo.staging.set_chunk(stg_chunk_idx, stg_offset, stg_round, stg_end)
    else:
        logger.warning(
            "STAGING_RSP RECV but tinfo=None room=%s chunk=%d session=%s",
            stg_room,
            stg_chunk_idx,
            stg_session,
        )
```
**EN:** This block defines the function `handle_staging_rsp`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `handle_staging_rsp`. Notable operations include `decode`, `get`, `StagingTransferInfo`, `set_chunk`.
**CN:** 这一段定义了function `handle_staging_rsp`，介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `handle_staging_rsp`。 值得注意的操作包括 `decode`、`get`、`StagingTransferInfo`、`set_chunk`。

### Lines 442-448: Module-level constants and helper logic
```python


# ======================================================================
# Staging data structures and protocol utilities
# ======================================================================


```
**EN:** This block contains module-level constants, helpers, or documentation for staging buffer coordination. It prepares shared state that later classes and functions build on.
**CN:** 这一段包含与暂存缓冲区协调相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。

### Lines 449-450: Class `StagingTransferInfo` declaration
```python
@dataclasses.dataclass
class StagingTransferInfo:
```
**EN:** This block declares the class `StagingTransferInfo` and establishes its responsibility inside staging buffer coordination. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `StagingTransferInfo`.
**CN:** 这一段声明了类 `StagingTransferInfo`，并说明它在暂存缓冲区协调中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `StagingTransferInfo`。

### Lines 451-456: Supporting state inside `StagingTransferInfo`
```python
    """Per-chunk staging allocation info attached to a TransferInfo."""

    offsets: List[int] = dataclasses.field(default_factory=lambda: [-1])
    rounds: List[int] = dataclasses.field(default_factory=lambda: [0])
    ends: List[int] = dataclasses.field(default_factory=lambda: [-1])

```
**EN:** This block adds supporting state or helper logic inside `StagingTransferInfo`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `field`.
**CN:** 这一段为 `StagingTransferInfo` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `field`。

### Lines 457-464: Method `set_chunk`
```python
    def set_chunk(self, idx: int, offset: int, rnd: int, end: int):
        while len(self.offsets) <= idx:
            self.offsets.append(-1)
            self.rounds.append(0)
            self.ends.append(-1)
        self.offsets[idx] = offset
        self.rounds[idx] = rnd
        self.ends[idx] = end
```
**EN:** This block defines the method `set_chunk` on `StagingTransferInfo`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `set_chunk`. Notable operations include `append`.
**CN:** 这一段定义了method `set_chunk`（属于 `StagingTransferInfo`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `set_chunk`。 值得注意的操作包括 `append`。

### Lines 467-468: Class `StagingRegisterInfo` declaration
```python
@dataclasses.dataclass
class StagingRegisterInfo:
```
**EN:** This block declares the class `StagingRegisterInfo` and establishes its responsibility inside staging buffer coordination. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `StagingRegisterInfo`.
**CN:** 这一段声明了类 `StagingRegisterInfo`，并说明它在暂存缓冲区协调中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `StagingRegisterInfo`。

### Lines 469-473: Supporting state inside `StagingRegisterInfo`
```python
    """Staging buffer registration info attached to a KVArgsRegisterInfo."""

    base_ptr: int = 0
    total_size: int = 0

```
**EN:** This block adds supporting state or helper logic inside `StagingRegisterInfo`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `StagingRegisterInfo` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 474-489: Method `from_zmq_fields`
```python
    @classmethod
    def from_zmq_fields(
        cls, msg: list, msg_start_offset: int
    ) -> Optional["StagingRegisterInfo"]:
        i = msg_start_offset
        base_ptr = (
            struct.unpack("Q", msg[i])[0] if len(msg) > i and len(msg[i]) == 8 else 0
        )
        total_size = (
            int(msg[i + 1].decode("ascii"))
            if len(msg) > i + 1 and len(msg[i + 1]) > 0
            else 0
        )
        if base_ptr == 0 and total_size == 0:
            return None
        return cls(base_ptr=base_ptr, total_size=total_size)
```
**EN:** This block defines the method `from_zmq_fields` on `StagingRegisterInfo`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `from_zmq_fields`. Notable operations include `unpack`, `decode`, `cls`.
**CN:** 这一段定义了method `from_zmq_fields`（属于 `StagingRegisterInfo`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `from_zmq_fields`。 值得注意的操作包括 `unpack`、`decode`、`cls`。

### Lines 492-492: Class `PrefillStagingStrategy` declaration
```python
class PrefillStagingStrategy:
```
**EN:** This block declares the class `PrefillStagingStrategy` and establishes its responsibility inside staging buffer coordination. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `PrefillStagingStrategy`.
**CN:** 这一段声明了类 `PrefillStagingStrategy`，并说明它在暂存缓冲区协调中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `PrefillStagingStrategy`。

### Lines 493-498: Supporting state inside `PrefillStagingStrategy`
```python
    """Prefill-side staging transfer: readiness check + gather-RDMA execution.

    Encapsulates the decision logic (chunk index calculation, staging offset
    lookup, watermark readiness) and delegates actual RDMA to the kv_manager.
    """

```
**EN:** This block adds supporting state or helper logic inside `PrefillStagingStrategy`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `logic`.
**CN:** 这一段为 `PrefillStagingStrategy` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `logic`。

### Lines 499-504: Method `__init__`
```python
    def __init__(self, kv_manager, staging_buffer):
        self.kv_manager = kv_manager
        self.staging_buffer = staging_buffer
        page_size = kv_manager.kv_buffer_tensors["page_size"]
        cps = kv_manager.server_args.chunked_prefill_size or 8192
        self.full_chunk_pages = max(1, cps // page_size)
```
**EN:** This block defines the method `__init__` on `PrefillStagingStrategy`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `PrefillStagingStrategy`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 506-525: Method `check_ready` signature and setup
```python
    def check_ready(
        self,
        req,
        kv_chunk_index_start: int,
        num_chunk_pages: int,
        session_id: Optional[str] = None,
    ) -> Tuple[bool, int, int, int, int]:
        """Check if staging offset and watermark are ready for this chunk.

        Args:
            req: transfer request with a ``.staging`` attribute.
            kv_chunk_index_start: page-level start index for this chunk.
            num_chunk_pages: number of pages in this chunk.
            session_id: identifier used for watermark lookup. Falls back to
                ``req.mooncake_session_id`` when *None* (mooncake compat).

        Returns (ready, chunk_idx, offset, round, end).
        offset == ALLOC_OVERSIZED means permanent failure (fall back to slice).
        offset == -1 means allocation pending (re-enqueue).
        """
```
**EN:** This block defines the method `check_ready` on `PrefillStagingStrategy`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `check_ready`. Notable operations include `Returns`, `failure`, `pending`.
**CN:** 这一段定义了method `check_ready`（属于 `PrefillStagingStrategy`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `check_ready`。 值得注意的操作包括 `Returns`、`failure`、`pending`。

### Lines 526-550: Method `check_ready` logic (part 1)
```python
        from sglang.srt.disaggregation.common.staging_buffer import StagingAllocator

        chunk_idx = (
            kv_chunk_index_start // self.full_chunk_pages
            if self.full_chunk_pages > 0
            else 0
        )

        stg = req.staging
        if stg is None or chunk_idx >= len(stg.offsets):
            return (False, chunk_idx, -1, 0, -1)

        c_offset = stg.offsets[chunk_idx]
        if c_offset == StagingAllocator.ALLOC_OVERSIZED:
            return (False, chunk_idx, StagingAllocator.ALLOC_OVERSIZED, 0, -1)
        if c_offset < 0:
            return (False, chunk_idx, -1, 0, -1)

        c_round = stg.rounds[chunk_idx]
        c_end = stg.ends[chunk_idx]

        if session_id is None:
            session_id = req.mooncake_session_id
        if not self.kv_manager._is_watermark_ready(session_id, c_round, c_end):
            return (False, chunk_idx, c_offset, c_round, c_end)
```
**EN:** This block continues `check_ready` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `_is_watermark_ready`.
**CN:** 这一段延续了 `check_ready` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `_is_watermark_ready`。

### Lines 551-552: Method `check_ready` logic (part 2)
```python

        return (True, chunk_idx, c_offset, c_round, c_end)
```
**EN:** This block continues `check_ready` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow.
**CN:** 这一段延续了 `check_ready` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。

### Lines 554-565: Method `transfer` signature and setup
```python
    def transfer(
        self,
        session_id: str,
        prefill_kv_indices,
        dst_staging_ptr: int,
        dst_staging_size: int,
        target_info,
    ) -> int:
        """Execute staged transfer (gather + RDMA).

        Returns 0 on success, -1 to signal fallback to slice path.
        """
```
**EN:** This block defines the method `transfer` on `PrefillStagingStrategy`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `transfer`. Notable operations include `transfer`.
**CN:** 这一段定义了method `transfer`（属于 `PrefillStagingStrategy`），介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `transfer`。 值得注意的操作包括 `transfer`。

### Lines 566-581: Method `transfer` logic (part 1)
```python
        try:
            return self.kv_manager.send_kvcache_staged(
                session_id,
                prefill_kv_indices,
                dst_staging_ptr,
                dst_staging_size,
                target_info.dst_tp_rank,
                target_info.dst_attn_tp_size,
                target_info.dst_kv_item_len,
                staging_buffer=self.staging_buffer,
            )
        except Exception as e:
            raise RuntimeError(
                f"[Staging] KV transfer via staging buffer failed: {e}. "
                f"session={session_id}"
            ) from e
```
**EN:** This block continues `transfer` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `send_kvcache_staged`, `RuntimeError`.
**CN:** 这一段延续了 `transfer` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `send_kvcache_staged`、`RuntimeError`。

### Lines 584-601: Function `_get_custom_mem_pool`
```python
def _get_custom_mem_pool(device: str):
    """Get custom memory pool for staging buffer allocation (backend-agnostic).

    Returns (custom_mem_pool, pool_type) tuple. custom_mem_pool may be None
    if no custom pool is configured.
    """
    from sglang.srt.disaggregation.mooncake.utils import (
        init_mooncake_custom_mem_pool,
    )

    _, custom_mem_pool, pool_type = init_mooncake_custom_mem_pool(device)
    if custom_mem_pool is None:
        logger.info(
            "Staging buffer using cudaMalloc (no custom mem pool). "
            "This works for all GPU architectures. "
            "For NVLink/MNNVL transport, set SGLANG_MOONCAKE_CUSTOM_MEM_POOL."
        )
    return custom_mem_pool, pool_type
```
**EN:** This block defines the function `_get_custom_mem_pool`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `_get_custom_mem_pool`. Notable operations include `allocation`, `Returns`, `import`, `init_mooncake_custom_mem_pool`.
**CN:** 这一段定义了function `_get_custom_mem_pool`，介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `_get_custom_mem_pool`。 值得注意的操作包括 `allocation`、`Returns`、`import`、`init_mooncake_custom_mem_pool`。

### Lines 604-614: Function `init_staging_buffers` signature and setup
```python
def init_staging_buffers(register_fn, kv_args, count: int) -> list:
    """Create prefill-side staging buffers and register them with the transport.

    Args:
        register_fn: callable(ptr: int, size: int) that registers a memory
            region with the transport backend.
        kv_args: KVArgs with gpu_id.
        count: number of staging buffers to create.

    Returns list of StagingBuffer instances.
    """
```
**EN:** This block defines the function `init_staging_buffers`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `init_staging_buffers`. Notable operations include `callable`.
**CN:** 这一段定义了function `init_staging_buffers`，介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `init_staging_buffers`。 值得注意的操作包括 `callable`。

### Lines 615-630: Function `init_staging_buffers` logic (part 1)
```python
    from sglang.srt.disaggregation.common.staging_buffer import StagingBuffer
    from sglang.srt.environ import envs

    size_mb = envs.SGLANG_DISAGG_STAGING_BUFFER_SIZE_MB.get()
    size_bytes = size_mb * 1024 * 1024
    gpu_id = kv_args.gpu_id
    device = f"cuda:{gpu_id}"

    custom_mem_pool, _ = _get_custom_mem_pool(device)

    buffers = []
    for _ in range(count):
        buf = StagingBuffer(size_bytes, device, gpu_id, custom_mem_pool=custom_mem_pool)
        register_fn(buf.get_ptr(), buf.get_size())
        buffers.append(buf)
    return buffers
```
**EN:** This block continues `init_staging_buffers` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `get`, `_get_custom_mem_pool`, `StagingBuffer`, `register_fn`.
**CN:** 这一段延续了 `init_staging_buffers` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `get`、`_get_custom_mem_pool`、`StagingBuffer`、`register_fn`。

### Lines 633-654: Function `init_staging_allocator`
```python
def init_staging_allocator(register_fn, kv_args):
    """Create decode-side staging ring-buffer allocator and register with transport.

    Args:
        register_fn: callable(ptr: int, size: int) that registers a memory
            region with the transport backend.
        kv_args: KVArgs with gpu_id.

    Returns a StagingAllocator instance.
    """
    from sglang.srt.disaggregation.common.staging_buffer import StagingAllocator
    from sglang.srt.environ import envs

    pool_size_mb = envs.SGLANG_DISAGG_STAGING_POOL_SIZE_MB.get()
    pool_size_bytes = pool_size_mb * 1024 * 1024
    gpu_id = kv_args.gpu_id
    device = f"cuda:{gpu_id}"

    custom_mem_pool, _ = _get_custom_mem_pool(device)
    allocator = StagingAllocator(pool_size_bytes, device, gpu_id, custom_mem_pool)
    register_fn(allocator.get_base_ptr(), allocator.get_total_size())
    return allocator
```
**EN:** This block defines the function `init_staging_allocator`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `init_staging_allocator`. Notable operations include `callable`, `get`, `_get_custom_mem_pool`, `StagingAllocator`.
**CN:** 这一段定义了function `init_staging_allocator`，介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `init_staging_allocator`。 值得注意的操作包括 `callable`、`get`、`_get_custom_mem_pool`、`StagingAllocator`。

### Lines 657-671: Function `handle_staging_req` signature and setup
```python
def handle_staging_req(
    msg,
    staging_allocator,
    kv_args,
    attn_tp_size: int,
    prefill_attn_tp_size: int,
    kv_buffer_tensors,
    room_receivers: dict,
    room_bootstrap: dict,
):
    """Allocate staging for a chunk on-demand and send STAGING_RSP to prefill.

    Deduplicates: multiple prefill TP ranks requesting the same (room, chunk_idx)
    only allocate once.  Sends ALLOC_OVERSIZED on permanent failure.
    """
```
**EN:** This block defines the function `handle_staging_req`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `handle_staging_req`. Notable operations include `same`.
**CN:** 这一段定义了function `handle_staging_req`，介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `handle_staging_req`。 值得注意的操作包括 `same`。

### Lines 672-696: Function `handle_staging_req` logic (part 1)
```python
    from sglang.srt.disaggregation.common.staging_buffer import StagingAllocator

    room = int(msg[1].decode("ascii"))
    chunk_idx = int(msg[2].decode("ascii"))
    chunk_num_pages = int(msg[3].decode("ascii"))
    session_id = msg[4].decode("ascii")

    if staging_allocator is None:
        logger.warning(
            "STAGING_REQ ignored: allocator is None room=%s chunk=%s",
            room,
            chunk_idx,
        )
        return

    receiver = room_receivers.get(room)
    if receiver is None:
        logger.warning(
            "STAGING_REQ dropped: no receiver for room=%s chunk=%s session=%s",
            room,
            chunk_idx,
            session_id,
        )
        return
    infos = getattr(receiver, "chunk_staging_infos", [])
```
**EN:** This block continues `handle_staging_req` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `decode`, `warning`, `get`.
**CN:** 这一段延续了 `handle_staging_req` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `decode`、`warning`、`get`。

### Lines 697-728: Function `handle_staging_req` logic (part 2)
```python

    if chunk_idx < len(infos) and infos[chunk_idx][0] >= 0:
        _, offset, rnd, end, _ = infos[chunk_idx]
    elif (
        chunk_idx < len(infos)
        and infos[chunk_idx][1] == StagingAllocator.ALLOC_OVERSIZED
    ):
        offset, rnd, end = StagingAllocator.ALLOC_OVERSIZED, 0, -1
    else:
        from sglang.srt.disaggregation.common.staging_buffer import (
            compute_staging_layout,
            resolve_total_kv_heads,
        )

        page_size = kv_args.page_size
        kv_item_lens = kv_args.kv_item_lens
        num_kv_layers = len(kv_item_lens) // 2
        decode_bytes_per_token = kv_item_lens[0] // page_size
        total_kv_heads = resolve_total_kv_heads(kv_args, attn_tp_size)
        dst_heads_per_rank = max(1, total_kv_heads // max(1, attn_tp_size))
        bytes_per_head_per_token = decode_bytes_per_token // dst_heads_per_rank
        dst_tp_rank = kv_args.engine_rank % max(1, attn_tp_size)

        chunk_tokens = chunk_num_pages * page_size
        _, _, required = compute_staging_layout(
            prefill_attn_tp_size,
            attn_tp_size,
            dst_tp_rank,
            total_kv_heads,
            chunk_tokens,
            bytes_per_head_per_token,
            num_kv_layers,
```
**EN:** This block continues `handle_staging_req` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `elif`, `import`, `resolve_total_kv_heads`, `compute_staging_layout`.
**CN:** 这一段延续了 `handle_staging_req` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `elif`、`import`、`resolve_total_kv_heads`、`compute_staging_layout`。

### Lines 729-755: Function `handle_staging_req` logic (part 3)
```python
        )
        result = staging_allocator.assign(required)
        if result is None:
            logger.error(
                "[STAGING_REQ] alloc failed room=%s chunk=%d (need %d bytes, "
                "buffer total=%d bytes). Increase SGLANG_DISAGG_STAGING_POOL_SIZE_MB.",
                room,
                chunk_idx,
                required,
                staging_allocator.total_size,
            )
            offset, rnd, end = StagingAllocator.ALLOC_OVERSIZED, 0, -1
            while len(infos) <= chunk_idx:
                infos.append((-1, -1, 0, -1, 0))
            infos[chunk_idx] = (
                -1,
                StagingAllocator.ALLOC_OVERSIZED,
                0,
                -1,
                chunk_num_pages,
            )
        else:
            alloc_id, offset, rnd = result
            end = offset + required
            while len(infos) <= chunk_idx:
                infos.append((-1, -1, 0, -1, 0))
            infos[chunk_idx] = (alloc_id, offset, rnd, end, chunk_num_pages)
```
**EN:** This block continues `handle_staging_req` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `assign`, `error`, `d`, `append`.
**CN:** 这一段延续了 `handle_staging_req` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `assign`、`error`、`d`、`append`。

### Lines 756-775: Function `handle_staging_req` logic (part 4)
```python

    bootstrap_infos = room_bootstrap.get(room)
    if bootstrap_infos:
        for bi in bootstrap_infos:
            try:
                sock, lock = receiver._connect_to_bootstrap_server(bi)
                with lock:
                    sock.send_multipart(
                        [
                            b"STAGING_RSP",
                            str(room).encode("ascii"),
                            str(chunk_idx).encode("ascii"),
                            str(offset).encode("ascii"),
                            str(rnd).encode("ascii"),
                            str(end).encode("ascii"),
                            session_id.encode("ascii"),
                        ]
                    )
            except Exception:
                pass
```
**EN:** This block continues `handle_staging_req` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `get`, `_connect_to_bootstrap_server`, `send_multipart`, `encode`.
**CN:** 这一段延续了 `handle_staging_req` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `get`、`_connect_to_bootstrap_server`、`send_multipart`、`encode`。

### Lines 778-790: Function `prefetch_staging_reqs` signature and setup
```python
def prefetch_staging_reqs(
    room: int,
    transfer_infos: dict,
    kv_buffer_tensors: dict,
    chunked_prefill_size: int,
    staging_requested: set,
    prefetch_sockets: dict,
) -> None:
    """Send STAGING_REQ for all chunks before the prefill forward starts.

    Called from the scheduler right after batch formation, so that decode
    allocates staging during the GPU forward pass.
    """
```
**EN:** This block defines the function `prefetch_staging_reqs`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer coordination. Definitions introduced here include `prefetch_staging_reqs`.
**CN:** 这一段定义了function `prefetch_staging_reqs`，介绍了参数、初始化步骤，以及这部分暂存缓冲区协调逻辑的主要入口。 此处引入的定义包括 `prefetch_staging_reqs`。

### Lines 791-811: Function `prefetch_staging_reqs` logic (part 1)
```python
    import zmq

    from sglang.srt.utils.network import NetworkAddress

    page_size = kv_buffer_tensors["page_size"]
    cps = chunked_prefill_size or 8192
    full_chunk_pages = max(1, cps // page_size)

    for session_id, tinfo in transfer_infos[room].items():
        # mooncake exposes is_dummy as a dataclass bool field, NIXL exposes it
        # as a method (it consults decode_prefix_len). Normalize via callable()
        # so this shared helper works for either backend; treating a bound
        # method as truthy (the previous behavior) silently dropped every
        # STAGING_REQ on NIXL and deadlocked the prefill transfer worker.
        is_dummy_attr = tinfo.is_dummy
        if is_dummy_attr() if callable(is_dummy_attr) else is_dummy_attr:
            continue
        total_pages = len(tinfo.dst_kv_indices)
        if total_pages == 0:
            continue
        num_chunks = (total_pages + full_chunk_pages - 1) // full_chunk_pages
```
**EN:** This block continues `prefetch_staging_reqs` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `items`, `method`, `callable`, `truthy`.
**CN:** 这一段延续了 `prefetch_staging_reqs` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `items`、`method`、`callable`、`truthy`。

### Lines 812-835: Function `prefetch_staging_reqs` logic (part 2)
```python

        for chunk_idx in range(num_chunks):
            stg_key = (room, chunk_idx, session_id)
            if stg_key in staging_requested:
                continue
            staging_requested.add(stg_key)

            remaining = total_pages - chunk_idx * full_chunk_pages
            chunk_pages = min(full_chunk_pages, remaining)
            try:
                na = NetworkAddress(tinfo.endpoint, tinfo.dst_port)
                ep = na.to_tcp()
                if ep not in prefetch_sockets:
                    sock = zmq.Context().socket(zmq.PUSH)
                    if na.is_ipv6:
                        sock.setsockopt(zmq.IPV6, 1)
                    sock.connect(ep)
                    prefetch_sockets[ep] = sock
                prefetch_sockets[ep].send_multipart(
                    [
                        b"STAGING_REQ",
                        str(room).encode("ascii"),
                        str(chunk_idx).encode("ascii"),
                        str(chunk_pages).encode("ascii"),
```
**EN:** This block continues `prefetch_staging_reqs` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `add`, `NetworkAddress`, `to_tcp`, `Context`.
**CN:** 这一段延续了 `prefetch_staging_reqs` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `add`、`NetworkAddress`、`to_tcp`、`Context`。

### Lines 836-840: Function `prefetch_staging_reqs` logic (part 3)
```python
                        session_id.encode("ascii"),
                    ]
                )
            except Exception:
                staging_requested.discard(stg_key)
```
**EN:** This block continues `prefetch_staging_reqs` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer coordination workflow. Notable operations include `encode`, `discard`.
**CN:** 这一段延续了 `prefetch_staging_reqs` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区协调工作流。 值得注意的操作包括 `encode`、`discard`。

## Key Concepts / 关键概念
- `DecodeStagingContext`: Class that encapsulates decode staging context behavior in this module. / `DecodeStagingContext`：封装与“解码暂存context”相关行为的类。
- `PrefillStagingContext`: Class that encapsulates prefill staging context behavior in this module. / `PrefillStagingContext`：封装与“预填充暂存context”相关行为的类。
- `DecodeStagingHandler`: Class that encapsulates decode staging handler behavior in this module. / `DecodeStagingHandler`：封装与“解码暂存处理器”相关行为的类。
- `is_watermark_ready`: Function that performs is watermark ready for the surrounding workflow. / `is_watermark_ready`：在周边工作流中执行“iswatermarkready”相关任务的函数。
- `handle_watermark_msg`: Function that performs handle watermark msg for the surrounding workflow. / `handle_watermark_msg`：在周边工作流中执行“handlewatermarkmsg”相关任务的函数。
- `handle_staging_rsp`: Function that performs handle staging rsp for the surrounding workflow. / `handle_staging_rsp`：在周边工作流中执行“handle暂存rsp”相关任务的函数。
- `StagingTransferInfo`: Class that encapsulates staging transfer info behavior in this module. / `StagingTransferInfo`：封装与“暂存传输info”相关行为的类。
- `StagingRegisterInfo`: Class that encapsulates staging register info behavior in this module. / `StagingRegisterInfo`：封装与“暂存registerinfo”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `logging`, `struct`, `threading`, `typing`
- **External packages / 外部依赖**: `torch`, `zmq`
- **Internal modules / 内部模块**: `sglang.srt.disaggregation.decode`, `sglang.srt.disaggregation.mooncake.utils`, `sglang.srt.disaggregation.common.staging_buffer`, `sglang.srt.environ`, `sglang.srt.utils.network`
