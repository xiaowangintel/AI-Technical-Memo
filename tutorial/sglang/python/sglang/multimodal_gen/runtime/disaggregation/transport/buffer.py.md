# buffer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/transport/buffer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `SlotHandle`, and `TransferTensorBuffer`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: TransferTensorBuffer: memory staging area for disaggregated tensor transfer. / 该文件属于解耦运行时层。它围绕 `SlotHandle` 和 `TransferTensorBuffer` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-16: module setup and imports / 模块初始化与导入
```python
"""TransferTensorBuffer: memory staging area for disaggregated tensor transfer."""

from __future__ import annotations

import logging
from dataclasses import dataclass, field

import torch

from sglang.multimodal_gen.runtime.disaggregation.transport.allocator import (
    BuddyAllocator,
)
from sglang.multimodal_gen.runtime.disaggregation.transport.codec import (
    str_to_dtype,
)
```
**EN:** This block establishes the module context and imports `__future__`, `logging`, `dataclasses`, `torch`, `sglang.multimodal_gen.runtime.disaggregation.transport.allocator`, and `sglang.multimodal_gen.runtime.disaggregation.transport.codec`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`logging`、`dataclasses`、`torch`、`sglang.multimodal_gen.runtime.disaggregation.transport.allocator` 和 `sglang.multimodal_gen.runtime.disaggregation.transport.codec`。这些依赖为后续实现提供所需符号。

### Lines 18-18: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 协同工作。

### Lines 22-22: `SlotHandle` class overview / `SlotHandle` 类概览
```python
class SlotHandle:
```
**EN:** This block defines class `SlotHandle`. It encapsulates slot handle behavior.
**CN:** 该代码块定义了类 `SlotHandle`。 它用于封装 slot handle 相关行为。

### Lines 23-28: supporting statements / 辅助语句
```python
    request_id: str
    offset: int  # byte offset in the pool
    size: int  # allocated size in bytes
    tensor_views: dict[str, torch.Tensor | list[torch.Tensor]] = field(
        default_factory=dict
    )
```
**EN:** This block gathers supporting statements inside `SlotHandle`. It updates names such as `request_id`, `offset`, `size`, and `tensor_views`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `SlotHandle` 内部的辅助语句。 它会更新 `request_id`、`offset`、`size` 和 `tensor_views` 等名称。 代码会与 `field` 协同工作。

### Lines 31-36: `TransferTensorBuffer` class overview / `TransferTensorBuffer` 类概览
```python
class TransferTensorBuffer:
    """Memory pool for staging tensor payloads between roles.

    Wraps a contiguous block of memory (CPU pinned or GPU) with a BuddyAllocator.
    """
```
**EN:** This block defines class `TransferTensorBuffer`. Memory pool for staging tensor payloads between roles. Wraps a contiguous block of memory (CPU pinned or GPU) with a BuddyAllocator.
**CN:** 该代码块定义了类 `TransferTensorBuffer`。 它用于封装 transfer tensor buffer 相关行为。

### Lines 37-63: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        pool_size: int,
        min_block_size: int = 1 << 20,
        role_name: str = "unknown",
        device: str = "cpu",
    ):
        self._role_name = role_name
        self._device = device
        self._allocator = BuddyAllocator(pool_size, min_block_size)
        actual_size = self._allocator.pool_size

        if device == "cpu":
            self._pool = torch.empty(actual_size, dtype=torch.uint8, pin_memory=True)
        else:
            self._pool = torch.empty(actual_size, dtype=torch.uint8, device=device)
        self._pool_ptr = self._pool.data_ptr()

        pool_location = "pinned CPU" if device == "cpu" else f"GPU ({device})"
        logger.info(
            "TransferTensorBuffer[%s]: allocated %d MiB %s memory "
            "(min_block=%d KiB)",
            role_name,
            actual_size >> 20,
            pool_location,
            min_block_size >> 10,
        )
```
**EN:** This block defines method `__init__` on `TransferTensorBuffer`. It initializes the instance state. Key calls include `BuddyAllocator`, `self._pool.data_ptr`, `logger.info`, and `torch.empty`. The implementation branches on conditions. Parameters such as `pool_size`, `min_block_size`, `role_name`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `BuddyAllocator`、`self._pool.data_ptr`、`logger.info` 和 `torch.empty`。 实现中包含条件分支。 本段逻辑主要由 `pool_size`、`min_block_size`、`role_name` 和 `device` 等参数驱动。

### Lines 65-67: `pool_size` implementation / `pool_size` 实现
```python
    @property
    def pool_size(self) -> int:
        return self._allocator.pool_size
```
**EN:** This block defines method `pool_size` on `TransferTensorBuffer`. It handles pool size logic.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `pool_size`。 它用于处理 pool size 相关逻辑。

### Lines 69-71: `device` implementation / `device` 实现
```python
    @property
    def device(self) -> str:
        return self._device
```
**EN:** This block defines method `device` on `TransferTensorBuffer`. It handles device logic.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `device`。 它用于处理 device 相关逻辑。

### Lines 73-75: `pool_data_ptr` implementation / `pool_data_ptr` 实现
```python
    @property
    def pool_data_ptr(self) -> int:
        return self._pool_ptr
```
**EN:** This block defines method `pool_data_ptr` on `TransferTensorBuffer`. It handles pool data ptr logic.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `pool_data_ptr`。 它用于处理 pool data ptr 相关逻辑。

### Lines 77-96: `allocate` implementation / `allocate` 实现
```python
    def allocate(self, size: int, request_id: str) -> SlotHandle | None:
        """Allocate a slot. Returns None if pool is full."""
        offset = self._allocator.allocate(size, request_id=request_id)
        if offset is None:
            logger.warning(
                "TransferTensorBuffer[%s]: allocation failed for %s (%d bytes). "
                "Pool stats: %s",
                self._role_name,
                request_id,
                size,
                self._allocator.get_stats(),
            )
            return None

        block = self._allocator.get_block_info(offset)
        return SlotHandle(
            request_id=request_id,
            offset=offset,
            size=block.size if block else size,
        )
```
**EN:** This block defines method `allocate` on `TransferTensorBuffer`. Allocate a slot. Returns None if pool is full. Key calls include `self._allocator.allocate`, `self._allocator.get_block_info`, `SlotHandle`, `logger.warning`, and `self._allocator.get_stats`. The implementation branches on conditions. Parameters such as `size`, and `request_id` drive the behavior in this section.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `allocate`。 它用于处理 allocate 相关逻辑。 关键调用包括 `self._allocator.allocate`、`self._allocator.get_block_info`、`SlotHandle`、`logger.warning` 和 `self._allocator.get_stats`。 实现中包含条件分支。 本段逻辑主要由 `size` 和 `request_id` 等参数驱动。

### Lines 98-99: `free` implementation / `free` 实现
```python
    def free(self, handle: SlotHandle) -> bool:
        return self._allocator.free(handle.offset)
```
**EN:** This block defines method `free` on `TransferTensorBuffer`. It handles free logic. Key calls include `self._allocator.free`. Parameters such as `handle` drive the behavior in this section.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `free`。 它用于处理 free 相关逻辑。 关键调用包括 `self._allocator.free`。 本段逻辑主要由 `handle` 等参数驱动。

### Lines 101-130: `write_tensor` implementation / `write_tensor` 实现
```python
    def write_tensor(
        self,
        handle: SlotHandle,
        name: str,
        tensor: torch.Tensor,
        byte_offset: int = 0,
        stream: torch.cuda.Stream | None = None,
    ) -> int:
        """Copy a tensor into the pool slot. Returns bytes written."""
        src_tensor = tensor.contiguous()
        nbytes = src_tensor.numel() * src_tensor.element_size()

        if byte_offset + nbytes > handle.size:
            raise ValueError(
                f"Write exceeds slot: offset={byte_offset}, nbytes={nbytes}, "
                f"slot_size={handle.size}"
            )

        dst = self._pool[
            handle.offset + byte_offset : handle.offset + byte_offset + nbytes
        ]
        src_bytes = src_tensor.view(torch.uint8).reshape(-1)

        if stream is not None:
            with torch.cuda.stream(stream):
                dst.copy_(src_bytes, non_blocking=True)
        else:
            dst.copy_(src_bytes, non_blocking=True)

        return nbytes
```
**EN:** This block defines method `write_tensor` on `TransferTensorBuffer`. Copy a tensor into the pool slot. Returns bytes written. Key calls include `tensor.contiguous`, `src_tensor.view.reshape`, `src_tensor.numel`, `src_tensor.element_size`, and `ValueError`. The implementation branches on conditions, uses context-managed resources. Parameters such as `handle`, `name`, `tensor`, `byte_offset`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `write_tensor`。 它用于处理 write tensor 相关逻辑。 关键调用包括 `tensor.contiguous`、`src_tensor.view.reshape`、`src_tensor.numel`、`src_tensor.element_size` 和 `ValueError`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `handle`、`name`、`tensor`、`byte_offset` 和 `stream` 等参数驱动。

### Lines 132-167: `read_tensor` implementation / `read_tensor` 实现
```python
    def read_tensor(
        self,
        handle: SlotHandle,
        shape: list[int],
        dtype: torch.dtype,
        byte_offset: int = 0,
        device: torch.device | str = "cpu",
        stream: torch.cuda.Stream | None = None,
    ) -> torch.Tensor:
        """Read a tensor from the pool slot. Returns a clone on target device."""
        nbytes = 1
        for s in shape:
            nbytes *= s
        nbytes *= torch.tensor([], dtype=dtype).element_size()

        raw = self._pool[
            handle.offset + byte_offset : handle.offset + byte_offset + nbytes
        ]
        src = raw.view(dtype).reshape(shape)

        pool_dev = str(self._pool.device)
        target_dev = str(device)

        same_device = pool_dev == target_dev

        if same_device:
            # Clone to decouple tensor lifetime from pool slot
            if stream is not None:
                with torch.cuda.stream(stream):
                    return src.clone()
            return src.clone()

        if stream is not None:
            with torch.cuda.stream(stream):
                return src.to(device, non_blocking=True)
        return src.to(device, non_blocking=True)
```
**EN:** This block defines method `read_tensor` on `TransferTensorBuffer`. Read a tensor from the pool slot. Returns a clone on target device. Key calls include `torch.tensor.element_size`, `raw.view.reshape`, `str`, `src.to`, and `src.clone`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `handle`, `shape`, `dtype`, `byte_offset`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `read_tensor`。 它用于处理 read tensor 相关逻辑。 关键调用包括 `torch.tensor.element_size`、`raw.view.reshape`、`str`、`src.to` 和 `src.clone`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `handle`、`shape`、`dtype`、`byte_offset` 和 `device` 等参数驱动。

### Lines 169-221: `write_tensors_from_gpu` implementation / `write_tensors_from_gpu` 实现
```python
    def write_tensors_from_gpu(
        self,
        handle: SlotHandle,
        tensors: dict[str, torch.Tensor | list[torch.Tensor] | None],
        stream: torch.cuda.Stream | None = None,
    ) -> dict[str, list[dict]]:
        """Batch-write GPU tensors into a slot. Returns a manifest for later reads."""
        manifest: dict[str, list[dict]] = {}
        byte_offset = 0

        # Ensure copy stream sees all prior compute kernels
        if stream is not None:
            stream.wait_stream(torch.cuda.current_stream())

        for name, value in tensors.items():
            if value is None:
                continue

            entries = []
            if isinstance(value, torch.Tensor):
                nbytes = self.write_tensor(handle, name, value, byte_offset, stream)
                entries.append(
                    {
                        "offset": byte_offset,
                        "shape": list(value.shape),
                        "dtype": str(value.dtype).replace("torch.", ""),
                    }
                )
                byte_offset += nbytes
                byte_offset = (byte_offset + 511) & ~511  # align to 512B

            elif isinstance(value, list):
                for i, t in enumerate(value):
                    if t is None:
                        continue
                    nbytes = self.write_tensor(
                        handle, f"{name}[{i}]", t, byte_offset, stream
                    )
                    entries.append(
                        {
                            "offset": byte_offset,
                            "shape": list(t.shape),
                            "dtype": str(t.dtype).replace("torch.", ""),
                            "list_index": i,
                        }
                    )
                    byte_offset += nbytes
                    byte_offset = (byte_offset + 511) & ~511

            if entries:
                manifest[name] = entries

        return manifest
```
**EN:** This block defines method `write_tensors_from_gpu` on `TransferTensorBuffer`. Batch-write GPU tensors into a slot. Returns a manifest for later reads. Key calls include `tensors.items`, `stream.wait_stream`, `isinstance`, `torch.cuda.current_stream`, and `self.write_tensor`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `handle`, `tensors`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `write_tensors_from_gpu`。 它用于处理 write tensors from gpu 相关逻辑。 关键调用包括 `tensors.items`、`stream.wait_stream`、`isinstance`、`torch.cuda.current_stream` 和 `self.write_tensor`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `handle`、`tensors` 和 `stream` 等参数驱动。

### Lines 223-263: `read_tensors_from_manifest` implementation / `read_tensors_from_manifest` 实现
```python
    def read_tensors_from_manifest(
        self,
        handle: SlotHandle,
        manifest: dict[str, list[dict]],
        device: torch.device | str = "cpu",
        stream: torch.cuda.Stream | None = None,
    ) -> dict[str, torch.Tensor | list[torch.Tensor]]:
        """Batch-read tensors from a slot using a manifest."""
        result: dict[str, torch.Tensor | list[torch.Tensor]] = {}

        for name, entries in manifest.items():
            if not entries:
                continue
            has_list_index = any("list_index" in e for e in entries)

            if has_list_index:
                max_idx = max(e.get("list_index", 0) for e in entries) + 1
                tensors = [None] * max_idx
                for entry in entries:
                    t = self.read_tensor(
                        handle,
                        entry["shape"],
                        str_to_dtype(entry["dtype"]),
                        entry["offset"],
                        device,
                        stream,
                    )
                    tensors[entry["list_index"]] = t
                result[name] = tensors
            else:
                entry = entries[0]
                result[name] = self.read_tensor(
                    handle,
                    entry["shape"],
                    str_to_dtype(entry["dtype"]),
                    entry["offset"],
                    device,
                    stream,
                )

        return result
```
**EN:** This block defines method `read_tensors_from_manifest` on `TransferTensorBuffer`. Batch-read tensors from a slot using a manifest. Key calls include `manifest.items`, `any`, `self.read_tensor`, `max`, and `str_to_dtype`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `handle`, `manifest`, `device`, and `stream` drive the behavior in this section.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `read_tensors_from_manifest`。 它用于处理 read tensors from manifest 相关逻辑。 关键调用包括 `manifest.items`、`any`、`self.read_tensor`、`max` 和 `str_to_dtype`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `handle`、`manifest`、`device` 和 `stream` 等参数驱动。

### Lines 265-267: `free_slots_count` implementation / `free_slots_count` 实现
```python
    def free_slots_count(self, typical_request_size: int) -> int:
        """Estimate how many requests of typical size can still be buffered."""
        return self._allocator.count_free_slots(typical_request_size)
```
**EN:** This block defines method `free_slots_count` on `TransferTensorBuffer`. Estimate how many requests of typical size can still be buffered. Key calls include `self._allocator.count_free_slots`. Parameters such as `typical_request_size` drive the behavior in this section.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `free_slots_count`。 它用于处理 free slots count 相关逻辑。 关键调用包括 `self._allocator.count_free_slots`。 本段逻辑主要由 `typical_request_size` 等参数驱动。

### Lines 269-272: `get_stats` implementation / `get_stats` 实现
```python
    def get_stats(self) -> dict:
        alloc_stats = self._allocator.get_stats()
        alloc_stats["role"] = self._role_name
        return alloc_stats
```
**EN:** This block defines method `get_stats` on `TransferTensorBuffer`. It retrieves stats. Key calls include `self._allocator.get_stats`.
**CN:** 该代码块定义了 `TransferTensorBuffer` 的方法 `get_stats`。 它用于获取stats。 关键调用包括 `self._allocator.get_stats`。

## Key Concepts / 关键概念
- `SlotHandle`: Primary class that encapsulates slot handle behavior. / 核心类，用于封装 slot handle 相关行为。
- `TransferTensorBuffer`: Memory pool for staging tensor payloads between roles. / 核心类，用于封装 transfer tensor buffer 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `dataclasses`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.disaggregation.transport.allocator`, `sglang.multimodal_gen.runtime.disaggregation.transport.codec`

- **Total lines / 总行数**: 272
