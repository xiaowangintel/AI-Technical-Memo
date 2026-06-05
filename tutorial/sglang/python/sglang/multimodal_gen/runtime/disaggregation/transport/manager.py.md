# manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/transport/manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `StagedTransfer`, `PendingReceive`, and `DiffusionTransferManager`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Per-instance transfer manager for disaggregated diffusion roles. / 该文件属于解耦运行时层。它围绕 `StagedTransfer`、`PendingReceive` 和 `DiffusionTransferManager` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-16: module setup and imports / 模块初始化与导入
```python
"""Per-instance transfer manager for disaggregated diffusion roles."""

import logging
import threading
from dataclasses import dataclass, field

import torch

from sglang.multimodal_gen.runtime.disaggregation.transport.buffer import (
    SlotHandle,
    TransferTensorBuffer,
)
from sglang.multimodal_gen.runtime.disaggregation.transport.engine import (
    BaseTransferEngine,
)
```
**EN:** This block establishes the module context and imports `logging`, `threading`, `dataclasses`, `torch`, `sglang.multimodal_gen.runtime.disaggregation.transport.buffer`, and `sglang.multimodal_gen.runtime.disaggregation.transport.engine`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `logging`、`threading`、`dataclasses`、`torch`、`sglang.multimodal_gen.runtime.disaggregation.transport.buffer` 和 `sglang.multimodal_gen.runtime.disaggregation.transport.engine`。这些依赖为后续实现提供所需符号。

### Lines 18-18: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 协同工作。

### Lines 22-22: `StagedTransfer` class overview / `StagedTransfer` 类概览
```python
class StagedTransfer:
```
**EN:** This block defines class `StagedTransfer`. It encapsulates staged transfer behavior.
**CN:** 该代码块定义了类 `StagedTransfer`。 它用于封装 staged transfer 相关行为。

### Lines 23-26: supporting statements / 辅助语句
```python
    request_id: str
    slot: SlotHandle
    manifest: dict
    scalar_fields: dict = field(default_factory=dict)
```
**EN:** This block gathers supporting statements inside `StagedTransfer`. It updates names such as `request_id`, `slot`, `manifest`, and `scalar_fields`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `StagedTransfer` 内部的辅助语句。 它会更新 `request_id`、`slot`、`manifest` 和 `scalar_fields` 等名称。 代码会与 `field` 协同工作。

### Lines 30-30: `PendingReceive` class overview / `PendingReceive` 类概览
```python
class PendingReceive:
```
**EN:** This block defines class `PendingReceive`. It encapsulates pending receive behavior.
**CN:** 该代码块定义了类 `PendingReceive`。 它用于封装 pending receive 相关行为。

### Lines 31-32: supporting statements / 辅助语句
```python
    request_id: str
    slot: SlotHandle
```
**EN:** This block gathers supporting statements inside `PendingReceive`. It updates names such as `request_id`, and `slot`.
**CN:** 该代码块汇集了位于 `PendingReceive` 内部的辅助语句。 它会更新 `request_id` 和 `slot` 等名称。

### Lines 35-40: `DiffusionTransferManager` class overview / `DiffusionTransferManager` 类概览
```python
class DiffusionTransferManager:
    """Manages tensor transfers for a single role instance.

    Owns a TransferTensorBuffer (memory pool) and a BaseTransferEngine (RDMA or mock).
    """
```
**EN:** This block defines class `DiffusionTransferManager`. Manages tensor transfers for a single role instance. Owns a TransferTensorBuffer (memory pool) and a BaseTransferEngine (RDMA or mock).
**CN:** 该代码块定义了类 `DiffusionTransferManager`。 它用于封装 diffusion transfer manager 相关行为。

### Lines 41-59: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        engine: BaseTransferEngine,
        buffer: TransferTensorBuffer,
    ):
        self._engine = engine
        self._buffer = buffer
        self._lock = threading.Lock()

        self._engine.register_buffer(self._buffer.pool_data_ptr, self._buffer.pool_size)

        self._staged: dict[str, StagedTransfer] = {}
        self._pending_receives: dict[str, PendingReceive] = {}

        logger.info(
            "DiffusionTransferManager initialized: session=%s, pool=%d bytes",
            self._engine.session_id,
            self._buffer.pool_size,
        )
```
**EN:** This block defines method `__init__` on `DiffusionTransferManager`. It initializes the instance state. Key calls include `threading.Lock`, `self._engine.register_buffer`, and `logger.info`. Parameters such as `engine`, and `buffer` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `threading.Lock`、`self._engine.register_buffer` 和 `logger.info`。 本段逻辑主要由 `engine` 和 `buffer` 等参数驱动。

### Lines 61-63: `session_id` implementation / `session_id` 实现
```python
    @property
    def session_id(self) -> str:
        return self._engine.session_id
```
**EN:** This block defines method `session_id` on `DiffusionTransferManager`. It handles session id logic.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `session_id`。 它用于处理 session id 相关逻辑。

### Lines 65-67: `pool_data_ptr` implementation / `pool_data_ptr` 实现
```python
    @property
    def pool_data_ptr(self) -> int:
        return self._buffer.pool_data_ptr
```
**EN:** This block defines method `pool_data_ptr` on `DiffusionTransferManager`. It handles pool data ptr logic.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `pool_data_ptr`。 它用于处理 pool data ptr 相关逻辑。

### Lines 69-71: `pool_size` implementation / `pool_size` 实现
```python
    @property
    def pool_size(self) -> int:
        return self._buffer.pool_size
```
**EN:** This block defines method `pool_size` on `DiffusionTransferManager`. It handles pool size logic.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `pool_size`。 它用于处理 pool size 相关逻辑。

### Lines 73-133: `stage_tensors` implementation / `stage_tensors` 实现
```python
    def stage_tensors(
        self,
        request_id: str,
        tensor_fields: dict[str, torch.Tensor | list[torch.Tensor] | None],
        scalar_fields: dict | None = None,
        stream: torch.cuda.Stream | None = None,
    ) -> StagedTransfer | None:
        """Stage GPU tensors into the local TransferBuffer. Returns None on allocation failure."""
        total_size = 0
        for name, t in tensor_fields.items():
            if t is None:
                continue
            if isinstance(t, list):
                for ti in t:
                    total_size += ti.nelement() * ti.element_size()
            else:
                total_size += t.nelement() * t.element_size()

        if total_size == 0:
            staged = StagedTransfer(
                request_id=request_id,
                slot=None,
                manifest={},
                scalar_fields=scalar_fields or {},
            )
            with self._lock:
                self._staged[request_id] = staged
            return staged

        slot = self._buffer.allocate(total_size, request_id)
        if slot is None:
            logger.warning(
                "TransferManager: failed to allocate %d bytes for %s",
                total_size,
                request_id,
            )
            return None

        manifest = self._buffer.write_tensors_from_gpu(slot, tensor_fields, stream)

        if stream is not None:
            stream.synchronize()
        elif torch.cuda.is_available():
            torch.cuda.synchronize()

        staged = StagedTransfer(
            request_id=request_id,
            slot=slot,
            manifest=manifest,
            scalar_fields=scalar_fields or {},
        )
        with self._lock:
            self._staged[request_id] = staged

        logger.debug(
            "TransferManager: staged %s (%d bytes, offset=%d)",
            request_id,
            total_size,
            slot.offset,
        )
        return staged
```
**EN:** This block defines method `stage_tensors` on `DiffusionTransferManager`. Stage GPU tensors into the local TransferBuffer. Returns None on allocation failure. Key calls include `tensor_fields.items`, `self._buffer.allocate`, `self._buffer.write_tensors_from_gpu`, `StagedTransfer`, and `logger.debug`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `request_id`, `tensor_fields`, `scalar_fields`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `stage_tensors`。 它用于处理 stage tensors 相关逻辑。 关键调用包括 `tensor_fields.items`、`self._buffer.allocate`、`self._buffer.write_tensors_from_gpu`、`StagedTransfer` 和 `logger.debug`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `request_id`、`tensor_fields`、`scalar_fields` 和 `stream` 等参数驱动。

### Lines 135-201: `stage_tensors_async` implementation / `stage_tensors_async` 实现
```python
    def stage_tensors_async(
        self,
        request_id: str,
        tensor_fields: dict[str, torch.Tensor | list[torch.Tensor] | None],
        scalar_fields: dict | None = None,
        stream: torch.cuda.Stream | None = None,
    ) -> tuple[StagedTransfer | None, torch.cuda.Event | None]:
        """Stage GPU tensors, returning a CUDA event instead of blocking.

        Caller MUST wait on the event before reading buffer data.
        """
        total_size = 0
        for name, t in tensor_fields.items():
            if t is None:
                continue
            if isinstance(t, list):
                for ti in t:
                    total_size += ti.nelement() * ti.element_size()
            else:
                total_size += t.nelement() * t.element_size()

        if total_size == 0:
            staged = StagedTransfer(
                request_id=request_id,
                slot=None,
                manifest={},
                scalar_fields=scalar_fields or {},
            )
            with self._lock:
                self._staged[request_id] = staged
            return staged, None

        slot = self._buffer.allocate(total_size, request_id)
        if slot is None:
            logger.warning(
                "TransferManager: failed to allocate %d bytes for %s",
                total_size,
                request_id,
            )
            return None, None

        manifest = self._buffer.write_tensors_from_gpu(slot, tensor_fields, stream)

        d2h_event = None
        if stream is not None:
            d2h_event = torch.cuda.Event()
            d2h_event.record(stream)
        elif torch.cuda.is_available():
            d2h_event = torch.cuda.Event()
            d2h_event.record(torch.cuda.current_stream())

        staged = StagedTransfer(
            request_id=request_id,
            slot=slot,
            manifest=manifest,
            scalar_fields=scalar_fields or {},
        )
        with self._lock:
            self._staged[request_id] = staged

        logger.debug(
            "TransferManager: staged_async %s (%d bytes, offset=%d)",
            request_id,
            total_size,
            slot.offset,
        )
        return staged, d2h_event
```
**EN:** This block defines method `stage_tensors_async` on `DiffusionTransferManager`. Stage GPU tensors, returning a CUDA event instead of blocking. Caller MUST wait on the event before reading buffer data. Key calls include `tensor_fields.items`, `self._buffer.allocate`, `self._buffer.write_tensors_from_gpu`, `StagedTransfer`, and `logger.debug`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `request_id`, `tensor_fields`, `scalar_fields`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `stage_tensors_async`。 它用于处理 stage tensors async 相关逻辑。 关键调用包括 `tensor_fields.items`、`self._buffer.allocate`、`self._buffer.write_tensors_from_gpu`、`StagedTransfer` 和 `logger.debug`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `request_id`、`tensor_fields`、`scalar_fields` 和 `stream` 等参数驱动。

### Lines 203-240: `load_tensors_async` implementation / `load_tensors_async` 实现
```python
    def load_tensors_async(
        self,
        request_id: str,
        manifest: dict,
        device: torch.device | str = "cuda",
        stream: torch.cuda.Stream | None = None,
    ) -> tuple[dict[str, torch.Tensor | list[torch.Tensor]], torch.cuda.Event | None]:
        """Load tensors from receive slot to GPU, returning a CUDA event.

        Caller MUST wait on the event before using the returned tensors.
        """
        with self._lock:
            pending = self._pending_receives.get(request_id)

        if pending is None:
            raise ValueError(
                f"TransferManager: no pending receive slot for {request_id}"
            )

        tensors = self._buffer.read_tensors_from_manifest(
            pending.slot, manifest, device=device, stream=stream
        )

        load_event = None
        if stream is not None:
            load_event = torch.cuda.Event()
            load_event.record(stream)
        elif torch.cuda.is_available():
            load_event = torch.cuda.Event()
            load_event.record(torch.cuda.current_stream())

        logger.debug(
            "TransferManager: loaded_async %d tensor fields for %s to %s",
            len(tensors),
            request_id,
            device,
        )
        return tensors, load_event
```
**EN:** This block defines method `load_tensors_async` on `DiffusionTransferManager`. Load tensors from receive slot to GPU, returning a CUDA event. Caller MUST wait on the event before using the returned tensors. Key calls include `self._buffer.read_tensors_from_manifest`, `logger.debug`, `self._pending_receives.get`, `ValueError`, and `torch.cuda.Event`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request_id`, `manifest`, `device`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `load_tensors_async`。 它用于加载tensors async。 关键调用包括 `self._buffer.read_tensors_from_manifest`、`logger.debug`、`self._pending_receives.get`、`ValueError` 和 `torch.cuda.Event`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request_id`、`manifest`、`device` 和 `stream` 等参数驱动。

### Lines 242-279: `push_to_peer` implementation / `push_to_peer` 实现
```python
    def push_to_peer(
        self,
        request_id: str,
        dest_session_id: str,
        dest_addr: int,
        transfer_size: int,
    ) -> bool:
        """Push staged data to a remote peer's buffer via RDMA. Returns True on success."""
        with self._lock:
            staged = self._staged.get(request_id)

        if staged is None:
            logger.error("TransferManager: no staged transfer for %s", request_id)
            return False

        if staged.slot is None:
            return True

        src_addr = self._buffer.pool_data_ptr + staged.slot.offset
        ret = self._engine.transfer_sync(
            dest_session_id, src_addr, dest_addr, transfer_size
        )

        if ret == 0:
            logger.debug(
                "TransferManager: pushed %s (%d bytes) to %s",
                request_id,
                transfer_size,
                dest_session_id,
            )
        else:
            logger.error(
                "TransferManager: RDMA push failed for %s (ret=%d)",
                request_id,
                ret,
            )

        return ret == 0
```
**EN:** This block defines method `push_to_peer` on `DiffusionTransferManager`. Push staged data to a remote peer's buffer via RDMA. Returns True on success. Key calls include `self._engine.transfer_sync`, `self._staged.get`, `logger.error`, and `logger.debug`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request_id`, `dest_session_id`, `dest_addr`, and `transfer_size` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `push_to_peer`。 它用于处理 push to peer 相关逻辑。 关键调用包括 `self._engine.transfer_sync`、`self._staged.get`、`logger.error` 和 `logger.debug`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request_id`、`dest_session_id`、`dest_addr` 和 `transfer_size` 等参数驱动。

### Lines 281-287: `free_staged` implementation / `free_staged` 实现
```python
    def free_staged(self, request_id: str) -> None:
        with self._lock:
            staged = self._staged.pop(request_id, None)

        if staged and staged.slot is not None:
            self._buffer.free(staged.slot)
            logger.debug("TransferManager: freed staged slot for %s", request_id)
```
**EN:** This block defines method `free_staged` on `DiffusionTransferManager`. It handles free staged logic. Key calls include `self._staged.pop`, `self._buffer.free`, and `logger.debug`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `free_staged`。 它用于处理 free staged 相关逻辑。 关键调用包括 `self._staged.pop`、`self._buffer.free` 和 `logger.debug`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 289-312: `allocate_receive_slot` implementation / `allocate_receive_slot` 实现
```python
    def allocate_receive_slot(
        self, request_id: str, size: int
    ) -> PendingReceive | None:
        """Allocate a local buffer slot to receive incoming data."""
        slot = self._buffer.allocate(size, request_id)
        if slot is None:
            logger.warning(
                "TransferManager: failed to allocate receive slot (%d bytes) for %s",
                size,
                request_id,
            )
            return None

        pending = PendingReceive(request_id=request_id, slot=slot)
        with self._lock:
            self._pending_receives[request_id] = pending

        logger.debug(
            "TransferManager: allocated receive slot for %s (offset=%d, size=%d)",
            request_id,
            slot.offset,
            slot.size,
        )
        return pending
```
**EN:** This block defines method `allocate_receive_slot` on `DiffusionTransferManager`. Allocate a local buffer slot to receive incoming data. Key calls include `self._buffer.allocate`, `PendingReceive`, `logger.debug`, and `logger.warning`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request_id`, and `size` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `allocate_receive_slot`。 它用于处理 allocate receive slot 相关逻辑。 关键调用包括 `self._buffer.allocate`、`PendingReceive`、`logger.debug` 和 `logger.warning`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request_id` 和 `size` 等参数驱动。

### Lines 314-345: `load_tensors` implementation / `load_tensors` 实现
```python
    def load_tensors(
        self,
        request_id: str,
        manifest: dict,
        device: torch.device | str = "cuda",
        stream: torch.cuda.Stream | None = None,
    ) -> dict[str, torch.Tensor | list[torch.Tensor]]:
        """Load tensors from a receive slot into GPU memory."""
        with self._lock:
            pending = self._pending_receives.get(request_id)

        if pending is None:
            raise ValueError(
                f"TransferManager: no pending receive slot for {request_id}"
            )

        tensors = self._buffer.read_tensors_from_manifest(
            pending.slot, manifest, device=device, stream=stream
        )

        if stream is not None:
            stream.synchronize()
        elif torch.cuda.is_available():
            torch.cuda.synchronize()

        logger.debug(
            "TransferManager: loaded %d tensor fields for %s to %s",
            len(tensors),
            request_id,
            device,
        )
        return tensors
```
**EN:** This block defines method `load_tensors` on `DiffusionTransferManager`. Load tensors from a receive slot into GPU memory. Key calls include `self._buffer.read_tensors_from_manifest`, `logger.debug`, `self._pending_receives.get`, `ValueError`, and `stream.synchronize`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request_id`, `manifest`, `device`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `load_tensors`。 它用于加载tensors。 关键调用包括 `self._buffer.read_tensors_from_manifest`、`logger.debug`、`self._pending_receives.get`、`ValueError` 和 `stream.synchronize`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request_id`、`manifest`、`device` 和 `stream` 等参数驱动。

### Lines 347-354: `register_prealloc_as_receive` implementation / `register_prealloc_as_receive` 实现
```python
    def register_prealloc_as_receive(
        self, request_id: str, slot: "SlotHandle"
    ) -> "PendingReceive":
        """Register a pre-allocated slot as a pending receive (fast path)."""
        pending = PendingReceive(request_id=request_id, slot=slot)
        with self._lock:
            self._pending_receives[request_id] = pending
        return pending
```
**EN:** This block defines method `register_prealloc_as_receive` on `DiffusionTransferManager`. Register a pre-allocated slot as a pending receive (fast path). Key calls include `PendingReceive`. The implementation uses context-managed resources. Parameters such as `request_id`, and `slot` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `register_prealloc_as_receive`。 它用于注册prealloc as receive。 关键调用包括 `PendingReceive`。 实现中使用上下文管理资源。 本段逻辑主要由 `request_id` 和 `slot` 等参数驱动。

### Lines 356-362: `free_receive_slot` implementation / `free_receive_slot` 实现
```python
    def free_receive_slot(self, request_id: str) -> None:
        with self._lock:
            pending = self._pending_receives.pop(request_id, None)

        if pending:
            self._buffer.free(pending.slot)
            logger.debug("TransferManager: freed receive slot for %s", request_id)
```
**EN:** This block defines method `free_receive_slot` on `DiffusionTransferManager`. It handles free receive slot logic. Key calls include `self._pending_receives.pop`, `self._buffer.free`, and `logger.debug`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `free_receive_slot`。 它用于处理 free receive slot 相关逻辑。 关键调用包括 `self._pending_receives.pop`、`self._buffer.free` 和 `logger.debug`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 364-369: `get_receive_slot_addr` implementation / `get_receive_slot_addr` 实现
```python
    def get_receive_slot_addr(self, request_id: str) -> int | None:
        with self._lock:
            pending = self._pending_receives.get(request_id)
        if pending is None:
            return None
        return self._buffer.pool_data_ptr + pending.slot.offset
```
**EN:** This block defines method `get_receive_slot_addr` on `DiffusionTransferManager`. It retrieves receive slot addr. Key calls include `self._pending_receives.get`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `get_receive_slot_addr`。 它用于获取receive slot addr。 关键调用包括 `self._pending_receives.get`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 371-376: `get_receive_slot_offset` implementation / `get_receive_slot_offset` 实现
```python
    def get_receive_slot_offset(self, request_id: str) -> int | None:
        with self._lock:
            pending = self._pending_receives.get(request_id)
        if pending is None:
            return None
        return pending.slot.offset
```
**EN:** This block defines method `get_receive_slot_offset` on `DiffusionTransferManager`. It retrieves receive slot offset. Key calls include `self._pending_receives.get`. The implementation branches on conditions, uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `get_receive_slot_offset`。 它用于获取receive slot offset。 关键调用包括 `self._pending_receives.get`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 378-380: `get_staged_info` implementation / `get_staged_info` 实现
```python
    def get_staged_info(self, request_id: str) -> StagedTransfer | None:
        with self._lock:
            return self._staged.get(request_id)
```
**EN:** This block defines method `get_staged_info` on `DiffusionTransferManager`. It retrieves staged info. Key calls include `self._staged.get`. The implementation uses context-managed resources. Parameters such as `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `get_staged_info`。 它用于获取staged info。 关键调用包括 `self._staged.get`。 实现中使用上下文管理资源。 本段逻辑主要由 `request_id` 等参数驱动。

### Lines 382-383: `free_slots_count` implementation / `free_slots_count` 实现
```python
    def free_slots_count(self, typical_size: int = 64 * 1024 * 1024) -> int:
        return self._buffer.free_slots_count(typical_size)
```
**EN:** This block defines method `free_slots_count` on `DiffusionTransferManager`. It handles free slots count logic. Key calls include `self._buffer.free_slots_count`. Parameters such as `typical_size` drive the behavior in this section.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `free_slots_count`。 它用于处理 free slots count 相关逻辑。 关键调用包括 `self._buffer.free_slots_count`。 本段逻辑主要由 `typical_size` 等参数驱动。

### Lines 385-387: `cleanup` implementation / `cleanup` 实现
```python
    def cleanup(self) -> None:
        self._engine.deregister_buffer(self._buffer.pool_data_ptr)
        logger.info("DiffusionTransferManager cleaned up")
```
**EN:** This block defines method `cleanup` on `DiffusionTransferManager`. It handles cleanup logic. Key calls include `self._engine.deregister_buffer`, and `logger.info`.
**CN:** 该代码块定义了 `DiffusionTransferManager` 的方法 `cleanup`。 它用于处理 cleanup 相关逻辑。 关键调用包括 `self._engine.deregister_buffer` 和 `logger.info`。

## Key Concepts / 关键概念
- `StagedTransfer`: Primary class that encapsulates staged transfer behavior. / 核心类，用于封装 staged transfer 相关行为。
- `PendingReceive`: Primary class that encapsulates pending receive behavior. / 核心类，用于封装 pending receive 相关行为。
- `DiffusionTransferManager`: Manages tensor transfers for a single role instance. / 核心类，用于封装 diffusion transfer manager 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `threading`, `dataclasses`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.disaggregation.transport.buffer`, `sglang.multimodal_gen.runtime.disaggregation.transport.engine`

- **Total lines / 总行数**: 387
