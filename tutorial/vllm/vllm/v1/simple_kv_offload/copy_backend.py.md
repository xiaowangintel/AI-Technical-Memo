# copy_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/simple_kv_offload/copy_backend.py`
- **Repository**: vllm-project/vllm
- **Purpose**: DMA copy backend for GPU<->CPU block transfers. / 该模块位于 `simple_kv_offload` 子系统，主要围绕 `DmaCopyBackend` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""DMA copy backend for GPU<->CPU block transfers."""

from __future__ import annotations

import queue
import threading

import torch

from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.v1.simple_kv_offload.cuda_mem_ops import (
    BatchMemcpyParams,
    build_params,
    copy_blocks,
)

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `DmaCopyBackend` class / `DmaCopyBackend` 类
```python
class DmaCopyBackend:
    """cuMemcpyBatchAsync copy backend (background thread)."""
```
**EN:** Introduces the `DmaCopyBackend` class. Core methods include `__init__`, `init`, `launch_copy`, `shutdown`, `_copy_loop`. Docstring signal: cuMemcpyBatchAsync copy backend (background thread).
**CN:** 这里定义 `DmaCopyBackend` 类。核心方法包括 `__init__`, `init`, `launch_copy`, `shutdown`, `_copy_loop`。

### `DmaCopyBackend.__init__` method / `DmaCopyBackend.__init__` 方法
```python
    def __init__(self) -> None:
        self._store_params: BatchMemcpyParams | None = None
        self._load_params: BatchMemcpyParams | None = None
        self._load_stream: torch.cuda.Stream | None = None
        self._store_stream: torch.cuda.Stream | None = None
        self._queue: queue.SimpleQueue | None = None
        self._thread: threading.Thread | None = None
        self._shutdown: bool = False
```
**EN:** This method initializes the object state within `DmaCopyBackend`. It touches state such as `_store_params`, `_load_params`, `_load_stream`, `_store_stream`, `_queue`, `_thread`, `_shutdown`.
**CN:** 该方法会初始化对象状态，其作用域位于`DmaCopyBackend`。 它会读写 `_store_params`, `_load_params`, `_load_stream`, `_store_stream`, `_queue`, `_thread`, `_shutdown` 等状态。

### `DmaCopyBackend.init` method / `DmaCopyBackend.init` 方法
```python
    def init(
        self,
        gpu_caches: dict[str, torch.Tensor],
        cpu_caches: dict[str, torch.Tensor],
        device: torch.device,
        load_stream: torch.cuda.Stream,
        store_stream: torch.cuda.Stream,
    ) -> None:
        self._load_stream = load_stream
        self._store_stream = store_stream

        self._store_params = build_params(gpu_caches, cpu_caches, store_stream)
        self._load_params = build_params(cpu_caches, gpu_caches, load_stream)

        self._queue = queue.SimpleQueue()
        self._thread = threading.Thread(
            target=self._copy_loop,
            args=(self._queue, device, load_stream, store_stream),
            daemon=True,
        )
        self._thread.start()
```
**EN:** This method implements `init` within `DmaCopyBackend`. Key calls include `build_params`, `SimpleQueue`, `Thread`, `start`. It touches state such as `_load_stream`, `_store_stream`, `_store_params`, `_load_params`, `_queue`, `_thread`.
**CN:** 该方法会实现 `init`，其作用域位于`DmaCopyBackend`。 关键调用包括 `build_params`, `SimpleQueue`, `Thread`, `start`。 它会读写 `_load_stream`, `_store_stream`, `_store_params`, `_load_params`, `_queue`, `_thread` 等状态。

### `DmaCopyBackend.launch_copy` method / `DmaCopyBackend.launch_copy` 方法
```python
    def launch_copy(
        self,
        src_blocks: list[int],
        dst_blocks: list[int],
        is_store: bool,
        event_idx: int,
        events_list: list[tuple[int, torch.Event]],
    ) -> None:
        params = self._store_params if is_store else self._load_params
        assert params is not None and self._queue is not None
        self._queue.put(
            (src_blocks, dst_blocks, params, is_store, event_idx, events_list)
        )
```
**EN:** This method implements `launch_copy` within `DmaCopyBackend`. Key calls include `put`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `launch_copy`，其作用域位于`DmaCopyBackend`。 关键调用包括 `put`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DmaCopyBackend.shutdown` method / `DmaCopyBackend.shutdown` 方法
```python
    def shutdown(self) -> None:
        if self._shutdown:
            return
        self._shutdown = True
        if self._queue is not None:
            self._queue.put(None)
        if self._thread is not None:
            self._thread.join(timeout=5.0)
```
**EN:** This method implements `shutdown` within `DmaCopyBackend`. Key calls include `put`, `join`. It touches state such as `_shutdown`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `shutdown`，其作用域位于`DmaCopyBackend`。 关键调用包括 `put`, `join`。 它会读写 `_shutdown` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `DmaCopyBackend._copy_loop` method / `DmaCopyBackend._copy_loop` 方法
```python
    @staticmethod
    def _copy_loop(
        q: queue.SimpleQueue,
        device: torch.device,
        load_stream: torch.cuda.Stream,
        store_stream: torch.cuda.Stream,
    ) -> None:
        current_platform.set_device(device)
        while True:
            item = q.get()
            if item is None:
                return
            src_blocks, dst_blocks, params, is_store, event_idx, events_list = item
            copy_blocks(src_blocks, dst_blocks, params)
            stream = store_stream if is_store else load_stream
            event = torch.Event()
            event.record(stream)
            events_list.append((event_idx, event))
```
**EN:** This method implements `_copy_loop` within `DmaCopyBackend`. Key calls include `set_device`, `get`, `copy_blocks`, `Event`, `record`, `append`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_copy_loop`，其作用域位于`DmaCopyBackend`。 关键调用包括 `set_device`, `get`, `copy_blocks`, `Event`, `record`, `append`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `DmaCopyBackend`: central class or interface in this module. / `DmaCopyBackend`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `__future__`, `queue`, `threading`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.platforms`, `vllm.v1.simple_kv_offload.cuda_mem_ops`
