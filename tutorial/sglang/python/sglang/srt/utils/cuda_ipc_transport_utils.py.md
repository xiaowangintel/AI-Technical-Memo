# cuda_ipc_transport_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/cuda_ipc_transport_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for port allocation helpers. / 为 SGLang 运行时提供面向端口分配辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Module setup and shared state / 模块设置与共享状态
```python
import fcntl
import logging
import threading
import time
from multiprocessing import shared_memory
from typing import Any, Tuple

import numpy as np
import torch

from sglang.srt.environ import envs
from sglang.srt.server_args import get_global_server_args

logger = logging.getLogger(__name__)

MM_FEATURE_CACHE_SIZE = envs.SGLANG_MM_FEATURE_CACHE_MB.get() * 1024 * 1024

MM_ITEM_MEMORY_POOL_RECYCLE_INTERVAL = (
    envs.SGLANG_MM_ITEM_MEM_POOL_RECYCLE_INTERVAL_SEC.get()
)

SHM_LOCK_FILE = "/tmp/shm_wr_lock.lock"


# Cache for pool-level IPC handles on the consumer side.
# Key: the pool CUDA IPC handle tuple. Value: opened UntypedStorage.
_pool_storage_cache: dict = {}
_pool_cache_lock = threading.Lock()
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `fcntl`, `logging`, `threading`, `time`, `multiprocessing`, `typing`. It also defines symbols such as `logger`, `MM_FEATURE_CACHE_SIZE`, `MM_ITEM_MEMORY_POOL_RECYCLE_INTERVAL`, `SHM_LOCK_FILE`, `_pool_storage_cache`, `_pool_cache_lock` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `fcntl`, `logging`, `threading`, `time`, `multiprocessing`, `typing`。 同时定义了 `logger`, `MM_FEATURE_CACHE_SIZE`, `MM_ITEM_MEMORY_POOL_RECYCLE_INTERVAL`, `SHM_LOCK_FILE`, `_pool_storage_cache`, `_pool_cache_lock` 等符号，供后续逻辑使用。

### Lines 31-35: Function `_normalize_pool_cache_key` / 函数 `_normalize_pool_cache_key`
```python
def _normalize_pool_cache_key(pool_handle, pool_device_index: int) -> tuple[Any, ...]:
    normalized_handle = (
        pool_handle if isinstance(pool_handle, tuple) else tuple(pool_handle)
    )
    return (pool_device_index, normalized_handle)
```
**EN:** This function implements `_normalize_pool_cache_key`. It primarily calls `isinstance`, `tuple` to complete its work. State updates are written into `normalized_handle`.
**CN:** 该函数实现了 `_normalize_pool_cache_key`。 它主要通过调用 `isinstance`, `tuple` 来完成任务。 状态更新主要写入 `normalized_handle`。

### Lines 38-39: Function `_open_pooled_storage_uncached` / 函数 `_open_pooled_storage_uncached`
```python
def _open_pooled_storage_uncached(pool_handle):
    return torch.UntypedStorage._new_shared_cuda(*pool_handle)
```
**EN:** This function implements `_open_pooled_storage_uncached`. It primarily calls `torch.UntypedStorage._new_shared_cuda` to complete its work.
**CN:** 该函数实现了 `_open_pooled_storage_uncached`。 它主要通过调用 `torch.UntypedStorage._new_shared_cuda` 来完成任务。

### Lines 42-50: Function `_pool_handle_cache_get_or_open` / 函数 `_pool_handle_cache_get_or_open`
```python
def _pool_handle_cache_get_or_open(cache_key, pool_handle):
    storage = _pool_storage_cache.get(cache_key)
    if storage is None:
        with _pool_cache_lock:
            storage = _pool_storage_cache.get(cache_key)
            if storage is None:
                storage = _open_pooled_storage_uncached(pool_handle)
                _pool_storage_cache[cache_key] = storage
    return storage
```
**EN:** This function implements `_pool_handle_cache_get_or_open`. It primarily calls `_pool_storage_cache.get`, `_open_pooled_storage_uncached` to complete its work. State updates are written into `storage`, `_pool_storage_cache`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `_pool_handle_cache_get_or_open`。 它主要通过调用 `_pool_storage_cache.get`, `_open_pooled_storage_uncached` 来完成任务。 状态更新主要写入 `storage`, `_pool_storage_cache`。 实现中使用了条件分支、上下文管理资源。

### Lines 53-55: Function `_pool_handle_cache_set` / 函数 `_pool_handle_cache_set`
```python
def _pool_handle_cache_set(cache_key, storage):
    with _pool_cache_lock:
        _pool_storage_cache[cache_key] = storage
```
**EN:** This function implements `_pool_handle_cache_set`. State updates are written into `_pool_storage_cache`. The implementation relies on context-managed resources.
**CN:** 该函数实现了 `_pool_handle_cache_set`。 状态更新主要写入 `_pool_storage_cache`。 实现中使用了上下文管理资源。

### Lines 58-60: Function `_pool_handle_cache_invalidate` / 函数 `_pool_handle_cache_invalidate`
```python
def _pool_handle_cache_invalidate(cache_key):
    with _pool_cache_lock:
        _pool_storage_cache.pop(cache_key, None)
```
**EN:** This function implements `_pool_handle_cache_invalidate`. It primarily calls `_pool_storage_cache.pop` to complete its work. The implementation relies on context-managed resources.
**CN:** 该函数实现了 `_pool_handle_cache_invalidate`。 它主要通过调用 `_pool_storage_cache.pop` 来完成任务。 实现中使用了上下文管理资源。

### Lines 63-65: Function `_pool_handle_cache_clear` / 函数 `_pool_handle_cache_clear`
```python
def _pool_handle_cache_clear():
    with _pool_cache_lock:
        _pool_storage_cache.clear()
```
**EN:** This function implements `_pool_handle_cache_clear`. It primarily calls `_pool_storage_cache.clear` to complete its work. The implementation relies on context-managed resources.
**CN:** 该函数实现了 `_pool_handle_cache_clear`。 它主要通过调用 `_pool_storage_cache.clear` 来完成任务。 实现中使用了上下文管理资源。

### Lines 68-68: Class `ShmSyncBuffer` declaration / 类 `ShmSyncBuffer` 声明
```python
class ShmSyncBuffer:
```
**EN:** This class establishes `ShmSyncBuffer` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `__del__`.
**CN:** 该类将 `ShmSyncBuffer` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `__del__` 等方法。

### Lines 69-77: Method `ShmSyncBuffer.__init__` / 方法 `ShmSyncBuffer.__init__`
```python
    def __init__(self, byte_size: int = 4):
        self.buffer = shared_memory.SharedMemory(create=True, size=byte_size)
        self.buffer_wrapper = np.ndarray(1, dtype=np.float32, buffer=self.buffer.buf)
        self.buffer_wrapper *= 0
        self.meta_data = {
            "handle": self.buffer.name,
            "shape": self.buffer_wrapper.shape,
            "dtype": str(self.buffer_wrapper.dtype),
        }
```
**EN:** This method implements `__init__` on `ShmSyncBuffer`. It primarily calls `shared_memory.SharedMemory`, `np.ndarray`, `str` to complete its work. State updates are written into `self.buffer`, `self.buffer_wrapper`, `self.meta_data`.
**CN:** 该方法（属于 `ShmSyncBuffer`）实现了 `__init__`。 它主要通过调用 `shared_memory.SharedMemory`, `np.ndarray`, `str` 来完成任务。 状态更新主要写入 `self.buffer`, `self.buffer_wrapper`, `self.meta_data`。

### Lines 79-82: Method `ShmSyncBuffer.__del__` / 方法 `ShmSyncBuffer.__del__`
```python
    def __del__(self):
        if isinstance(self.buffer, shared_memory.SharedMemory):
            self.buffer.close()
            self.buffer.unlink()
```
**EN:** This method implements `__del__` on `ShmSyncBuffer`. It primarily calls `isinstance`, `self.buffer.close`, `self.buffer.unlink` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `ShmSyncBuffer`）实现了 `__del__`。 它主要通过调用 `isinstance`, `self.buffer.close`, `self.buffer.unlink` 来完成任务。 实现中使用了条件分支。

### Lines 85-85: Class `MmItemMemoryChunk` declaration / 类 `MmItemMemoryChunk` 声明
```python
class MmItemMemoryChunk:
```
**EN:** This class establishes `MmItemMemoryChunk` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `mem_size`, `start`, `end`, `try_to_recycle`.
**CN:** 该类将 `MmItemMemoryChunk` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `mem_size`, `start`, `end`, `try_to_recycle` 等方法。

### Lines 86-88: Method `MmItemMemoryChunk.__init__` / 方法 `MmItemMemoryChunk.__init__`
```python
    def __init__(self, area: Tuple, sync_buffer: ShmSyncBuffer):
        self.area = area
        self.sync_flag = sync_buffer
```
**EN:** This method implements `__init__` on `MmItemMemoryChunk`. State updates are written into `self.area`, `self.sync_flag`.
**CN:** 该方法（属于 `MmItemMemoryChunk`）实现了 `__init__`。 状态更新主要写入 `self.area`, `self.sync_flag`。

### Lines 90-92: Method `MmItemMemoryChunk.mem_size` / 方法 `MmItemMemoryChunk.mem_size`
```python
    @property
    def mem_size(self):
        return self.area[1] - self.area[0]
```
**EN:** This method implements `mem_size` on `MmItemMemoryChunk`.
**CN:** 该方法（属于 `MmItemMemoryChunk`）实现了 `mem_size`。

### Lines 94-96: Method `MmItemMemoryChunk.start` / 方法 `MmItemMemoryChunk.start`
```python
    @property
    def start(self):
        return self.area[0]
```
**EN:** This method implements `start` on `MmItemMemoryChunk`.
**CN:** 该方法（属于 `MmItemMemoryChunk`）实现了 `start`。

### Lines 98-100: Method `MmItemMemoryChunk.end` / 方法 `MmItemMemoryChunk.end`
```python
    @property
    def end(self):
        return self.area[1]
```
**EN:** This method implements `end` on `MmItemMemoryChunk`.
**CN:** 该方法（属于 `MmItemMemoryChunk`）实现了 `end`。

### Lines 102-118: Method `MmItemMemoryChunk.try_to_recycle` / 方法 `MmItemMemoryChunk.try_to_recycle`
```python
    def try_to_recycle(self) -> bool:
        try:
            tp_num = get_global_server_args().tp_size
        except Exception:
            logger.info(
                "get_global_server_args has not been inited , skip this turn 's recycle"
            )
            return False

        val = float(self.sync_flag.buffer_wrapper.item())
        logger.debug(f"[try_to_recycle] area={self.area}, flag={val}, tp_size={tp_num}")

        if val == float(tp_num):
            self.sync_flag.buffer_wrapper *= 0.0
            return True

        return False
```
**EN:** This method implements `try_to_recycle` on `MmItemMemoryChunk`. It primarily calls `float`, `logger.debug`, `self.sync_flag.buffer_wrapper.item`, `get_global_server_args`, `logger.info` to complete its work. State updates are written into `val`, `tp_num`, `self.sync_flag.buffer_wrapper`. The implementation relies on conditional branches, error handling.
**CN:** 该方法（属于 `MmItemMemoryChunk`）实现了 `try_to_recycle`。 它主要通过调用 `float`, `logger.debug`, `self.sync_flag.buffer_wrapper.item`, `get_global_server_args`, `logger.info` 来完成任务。 状态更新主要写入 `val`, `tp_num`, `self.sync_flag.buffer_wrapper`。 实现中使用了条件分支、错误处理。

### Lines 121-121: Class `MmItemMemoryPool` declaration / 类 `MmItemMemoryPool` 声明
```python
class MmItemMemoryPool:
```
**EN:** This class establishes `MmItemMemoryPool` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `shutdown`, `_recycle_loop`, `clear_sync_flag_list`, `pop_sync_buffer`, `push_sync_buffer`.
**CN:** 该类将 `MmItemMemoryPool` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `shutdown`, `_recycle_loop`, `clear_sync_flag_list`, `pop_sync_buffer`, `push_sync_buffer` 等方法。

### Lines 122-149: Method `MmItemMemoryPool.__init__` / 方法 `MmItemMemoryPool.__init__`
```python
    def __init__(self, memory_size, recycle_interval):
        self.memory_pool = torch.empty(
            memory_size, dtype=torch.int8, device="cuda"
        ).contiguous()
        storage = self.memory_pool.untyped_storage()
        self._pool_ipc_handle = storage._share_cuda_()
        self._pool_device_index = self.memory_pool.device.index

        self.sync_flag_list = []

        init_chunk = MmItemMemoryChunk((0, memory_size), self.pop_sync_buffer())
        self.available_chunks = [init_chunk]
        self.occupied_chunks = []

        self._lock = threading.Lock()
        self._pool_full_warned = False

        self._recycle_interval = recycle_interval
        self._stop_recycler = False
        self._recycle_thread = threading.Thread(
            target=self._recycle_loop, name="MmItemMemoryPoolRecycler", daemon=True
        )
        self._recycle_thread.start()

        logger.debug(
            f"[MmItemMemoryPool] init: memory_size={memory_size}, "
            f"recycle_interval={recycle_interval}s"
        )
```
**EN:** This method implements `__init__` on `MmItemMemoryPool`. It primarily calls `torch.empty.contiguous`, `self.memory_pool.untyped_storage`, `storage._share_cuda_`, `MmItemMemoryChunk`, `threading.Lock`, `threading.Thread` to complete its work. State updates are written into `self.memory_pool`, `storage`, `self._pool_ipc_handle`, `self._pool_device_index`, `self.sync_flag_list`, `init_chunk`.
**CN:** 该方法（属于 `MmItemMemoryPool`）实现了 `__init__`。 它主要通过调用 `torch.empty.contiguous`, `self.memory_pool.untyped_storage`, `storage._share_cuda_`, `MmItemMemoryChunk`, `threading.Lock`, `threading.Thread` 来完成任务。 状态更新主要写入 `self.memory_pool`, `storage`, `self._pool_ipc_handle`, `self._pool_device_index`, `self.sync_flag_list`, `init_chunk`。

### Lines 151-154: Method `MmItemMemoryPool.shutdown` / 方法 `MmItemMemoryPool.shutdown`
```python
    def shutdown(self):
        self._stop_recycler = True
        if self._recycle_thread.is_alive():
            self._recycle_thread.join(timeout=1.0)
```
**EN:** This method implements `shutdown` on `MmItemMemoryPool`. It primarily calls `self._recycle_thread.is_alive`, `self._recycle_thread.join` to complete its work. State updates are written into `self._stop_recycler`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MmItemMemoryPool`）实现了 `shutdown`。 它主要通过调用 `self._recycle_thread.is_alive`, `self._recycle_thread.join` 来完成任务。 状态更新主要写入 `self._stop_recycler`。 实现中使用了条件分支。

### Lines 156-167: Method `MmItemMemoryPool._recycle_loop` / 方法 `MmItemMemoryPool._recycle_loop`
```python
    def _recycle_loop(self):
        while not self._stop_recycler:
            try:
                with self._lock:
                    self.recycle_chunks()
                    self.merge_chunks()
            except Exception as e:
                logger.warning(
                    f"[MmItemMemoryPool] recycle loop error: {e}", exc_info=True
                )

            time.sleep(self._recycle_interval)
```
**EN:** This method implements `_recycle_loop` on `MmItemMemoryPool`. It primarily calls `time.sleep`, `self.recycle_chunks`, `self.merge_chunks`, `logger.warning` to complete its work. The implementation relies on iteration, context-managed resources, error handling.
**CN:** 该方法（属于 `MmItemMemoryPool`）实现了 `_recycle_loop`。 它主要通过调用 `time.sleep`, `self.recycle_chunks`, `self.merge_chunks`, `logger.warning` 来完成任务。 实现中使用了迭代逻辑、上下文管理资源、错误处理。

### Lines 169-171: Method `MmItemMemoryPool.clear_sync_flag_list` / 方法 `MmItemMemoryPool.clear_sync_flag_list`
```python
    def clear_sync_flag_list(self):
        # call each chunk's __del__
        self.sync_flag_list.clear()
```
**EN:** This method implements `clear_sync_flag_list` on `MmItemMemoryPool`. It primarily calls `self.sync_flag_list.clear` to complete its work.
**CN:** 该方法（属于 `MmItemMemoryPool`）实现了 `clear_sync_flag_list`。 它主要通过调用 `self.sync_flag_list.clear` 来完成任务。

### Lines 173-182: Method `MmItemMemoryPool.pop_sync_buffer` / 方法 `MmItemMemoryPool.pop_sync_buffer`
```python
    def pop_sync_buffer(self):
        if len(self.sync_flag_list) == 0:
            try:
                new_sync_buffer = ShmSyncBuffer()
                return new_sync_buffer
            except:
                logger.info("allocate shm buffer failed")
                raise RuntimeError
        else:
            return self.sync_flag_list.pop()
```
**EN:** This method implements `pop_sync_buffer` on `MmItemMemoryPool`. It primarily calls `len`, `self.sync_flag_list.pop`, `ShmSyncBuffer`, `logger.info` to complete its work. State updates are written into `new_sync_buffer`. The implementation relies on conditional branches, error handling.
**CN:** 该方法（属于 `MmItemMemoryPool`）实现了 `pop_sync_buffer`。 它主要通过调用 `len`, `self.sync_flag_list.pop`, `ShmSyncBuffer`, `logger.info` 来完成任务。 状态更新主要写入 `new_sync_buffer`。 实现中使用了条件分支、错误处理。

### Lines 184-185: Method `MmItemMemoryPool.push_sync_buffer` / 方法 `MmItemMemoryPool.push_sync_buffer`
```python
    def push_sync_buffer(self, sync_buffer):
        self.sync_flag_list.append(sync_buffer)
```
**EN:** This method implements `push_sync_buffer` on `MmItemMemoryPool`. It primarily calls `self.sync_flag_list.append` to complete its work.
**CN:** 该方法（属于 `MmItemMemoryPool`）实现了 `push_sync_buffer`。 它主要通过调用 `self.sync_flag_list.append` 来完成任务。

### Lines 187-222: Method `MmItemMemoryPool.get_available_chunk` / 方法 `MmItemMemoryPool.get_available_chunk`
```python
    def get_available_chunk(self, src_tensor: torch.Tensor) -> MmItemMemoryChunk:
        # find currently available_chunks contain a available chunk or not
        # if not, return None
        src_tensor_size = src_tensor.numel() * src_tensor.element_size()
        min_size = self.memory_pool.numel() * self.memory_pool.element_size() + 1
        selected_chunk = None
        for chunk in self.available_chunks:
            if chunk.mem_size >= src_tensor_size:
                if chunk.mem_size < min_size:
                    min_size = chunk.mem_size
                    selected_chunk = chunk

        if selected_chunk:
            occupied_chunk_area = (
                selected_chunk.start,
                selected_chunk.start + src_tensor_size,
            )
            occupied_chunk_sync_flag = selected_chunk.sync_flag
            new_occupied_chunk = MmItemMemoryChunk(
                occupied_chunk_area, occupied_chunk_sync_flag
            )

            self.occupied_chunks.append(new_occupied_chunk)
            self.available_chunks.remove(selected_chunk)

            available_split_chunk_area = (new_occupied_chunk.end, selected_chunk.end)
            # add a new chunk
            if available_split_chunk_area[0] != available_split_chunk_area[1]:
                split_available_chunk = MmItemMemoryChunk(
                    available_split_chunk_area, self.pop_sync_buffer()
                )
                self.available_chunks.append(split_available_chunk)

            return new_occupied_chunk

        return None
```
**EN:** This method implements `get_available_chunk` on `MmItemMemoryPool`. It primarily calls `src_tensor.numel`, `src_tensor.element_size`, `MmItemMemoryChunk`, `self.occupied_chunks.append`, `self.available_chunks.remove`, `self.memory_pool.numel` to complete its work. State updates are written into `src_tensor_size`, `min_size`, `selected_chunk`, `occupied_chunk_area`, `occupied_chunk_sync_flag`, `new_occupied_chunk`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MmItemMemoryPool`）实现了 `get_available_chunk`。 它主要通过调用 `src_tensor.numel`, `src_tensor.element_size`, `MmItemMemoryChunk`, `self.occupied_chunks.append`, `self.available_chunks.remove`, `self.memory_pool.numel` 来完成任务。 状态更新主要写入 `src_tensor_size`, `min_size`, `selected_chunk`, `occupied_chunk_area`, `occupied_chunk_sync_flag`, `new_occupied_chunk`。 实现中使用了条件分支、迭代逻辑。

### Lines 224-234: Method `MmItemMemoryPool.return_a_slice_tensor_with_flag` / 方法 `MmItemMemoryPool.return_a_slice_tensor_with_flag`
```python
    def return_a_slice_tensor_with_flag(self, src_tensor: torch.Tensor):
        with self._lock:
            available_chunk = self.get_available_chunk(src_tensor)
            if available_chunk is not None:
                return (
                    available_chunk.sync_flag.meta_data,
                    self.memory_pool[available_chunk.start : available_chunk.end],
                    available_chunk.start,
                )
        self._warn_pool_full_once(src_tensor)
        return None, None, None
```
**EN:** This method implements `return_a_slice_tensor_with_flag` on `MmItemMemoryPool`. It primarily calls `self._warn_pool_full_once`, `self.get_available_chunk` to complete its work. State updates are written into `available_chunk`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该方法（属于 `MmItemMemoryPool`）实现了 `return_a_slice_tensor_with_flag`。 它主要通过调用 `self._warn_pool_full_once`, `self.get_available_chunk` 来完成任务。 状态更新主要写入 `available_chunk`。 实现中使用了条件分支、上下文管理资源。

### Lines 236-250: Method `MmItemMemoryPool._warn_pool_full_once` / 方法 `MmItemMemoryPool._warn_pool_full_once`
```python
    def _warn_pool_full_once(self, src_tensor: torch.Tensor):
        if self._pool_full_warned:
            return
        self._pool_full_warned = True
        pool_mb = (
            self.memory_pool.numel() * self.memory_pool.element_size() / (1024 * 1024)
        )
        need_mb = src_tensor.numel() * src_tensor.element_size() / (1024 * 1024)
        logger.warning(
            "MmItemMemoryPool has no free chunk large enough for a %.2f MiB tensor "
            "(pool size: %.2f MiB); falling back to non-IPC transport. "
            "Consider increasing SGLANG_MM_FEATURE_CACHE_MB.",
            need_mb,
            pool_mb,
        )
```
**EN:** This method implements `_warn_pool_full_once` on `MmItemMemoryPool`. It primarily calls `logger.warning`, `self.memory_pool.numel`, `self.memory_pool.element_size`, `src_tensor.numel`, `src_tensor.element_size` to complete its work. State updates are written into `self._pool_full_warned`, `pool_mb`, `need_mb`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MmItemMemoryPool`）实现了 `_warn_pool_full_once`。 它主要通过调用 `logger.warning`, `self.memory_pool.numel`, `self.memory_pool.element_size`, `src_tensor.numel`, `src_tensor.element_size` 来完成任务。 状态更新主要写入 `self._pool_full_warned`, `pool_mb`, `need_mb`。 实现中使用了条件分支。

### Lines 252-260: Method `MmItemMemoryPool.recycle_chunks` / 方法 `MmItemMemoryPool.recycle_chunks`
```python
    def recycle_chunks(self):

        new_occupied_chunks = []
        for chunk in self.occupied_chunks:
            if chunk.try_to_recycle():
                self.available_chunks.append(chunk)
            else:
                new_occupied_chunks.append(chunk)
        self.occupied_chunks = new_occupied_chunks
```
**EN:** This method implements `recycle_chunks` on `MmItemMemoryPool`. It primarily calls `chunk.try_to_recycle`, `self.available_chunks.append`, `new_occupied_chunks.append` to complete its work. State updates are written into `new_occupied_chunks`, `self.occupied_chunks`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MmItemMemoryPool`）实现了 `recycle_chunks`。 它主要通过调用 `chunk.try_to_recycle`, `self.available_chunks.append`, `new_occupied_chunks.append` 来完成任务。 状态更新主要写入 `new_occupied_chunks`, `self.occupied_chunks`。 实现中使用了条件分支、迭代逻辑。

### Lines 262-280: Method `MmItemMemoryPool.merge_chunks` / 方法 `MmItemMemoryPool.merge_chunks`
```python
    def merge_chunks(self):
        # merge_all_available_chunks
        merged_chunks = []
        for chunk in sorted(self.available_chunks, key=lambda x: x.start):
            if len(merged_chunks) == 0:
                merged_chunks.append(chunk)
            else:
                if chunk.start == merged_chunks[-1].end:
                    to_merge_chunk = merged_chunks.pop()
                    to_merge_chunk_sync = to_merge_chunk.sync_flag
                    merged_chunk_area = (to_merge_chunk.start, chunk.end)
                    merged_chunks.append(
                        MmItemMemoryChunk(merged_chunk_area, to_merge_chunk_sync)
                    )
                    self.push_sync_buffer(chunk.sync_flag)
                else:
                    merged_chunks.append(chunk)

        self.available_chunks = merged_chunks
```
**EN:** This method implements `merge_chunks` on `MmItemMemoryPool`. It primarily calls `sorted`, `len`, `merged_chunks.append`, `merged_chunks.pop`, `self.push_sync_buffer`, `MmItemMemoryChunk` to complete its work. State updates are written into `merged_chunks`, `self.available_chunks`, `to_merge_chunk`, `to_merge_chunk_sync`, `merged_chunk_area`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MmItemMemoryPool`）实现了 `merge_chunks`。 它主要通过调用 `sorted`, `len`, `merged_chunks.append`, `merged_chunks.pop`, `self.push_sync_buffer`, `MmItemMemoryChunk` 来完成任务。 状态更新主要写入 `merged_chunks`, `self.available_chunks`, `to_merge_chunk`, `to_merge_chunk_sync`, `merged_chunk_area`。 实现中使用了条件分支、迭代逻辑。

### Lines 283-290: Class `CudaIpcTensorTransportProxy` declaration / 类 `CudaIpcTensorTransportProxy` 声明
```python
class CudaIpcTensorTransportProxy:
    """
    A torch.tensor's proxy used to do inter-process data-sharing
    including:

    torch.tensor(on gpu)'s cuda-ipc-hande infos
    a shm sync buffer's meta data which is used to sync between different process
    """
```
**EN:** This class establishes `CudaIpcTensorTransportProxy` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `get_sync_flag`, `close_shm`, `get_proxy_state`, `_reconstruct_from_ipc_extra`, `_copy_slice_tensor_to_target`.
**CN:** 该类将 `CudaIpcTensorTransportProxy` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `get_sync_flag`, `close_shm`, `get_proxy_state`, `_reconstruct_from_ipc_extra`, `_copy_slice_tensor_to_target` 等方法。

### Lines 292-329: Method `CudaIpcTensorTransportProxy.__init__` / 方法 `CudaIpcTensorTransportProxy.__init__`
```python
    def __init__(
        self,
        data: torch.Tensor,
        info_data: torch.Tensor,
        sync_buffer_meta,
        pool_ipc_handle=None,
        pool_byte_offset: int = 0,
        pool_device_index: int = 0,
    ):

        if (not isinstance(data, torch.Tensor)) or (
            not isinstance(info_data, torch.Tensor)
        ):
            raise TypeError(
                f"Input 'data' must be a torch.Tensor, but got {type(data)}"
            )

        if pool_ipc_handle is not None:
            self.proxy_state = {
                "ipc_extra": {
                    "pool_handle": pool_ipc_handle,
                    "pool_byte_offset": pool_byte_offset,
                    "pool_device_index": pool_device_index,
                    "shape": data.shape,
                    "dtype": data.dtype,
                    "stride": data.stride(),
                    "storage_offset": 0,
                    "nbytes": data.numel() * data.element_size(),
# ... omitted for brevity ...
            }
        else:
            self.proxy_state = self.get_proxy_state(data, info_data)
        self.reconstruct_tensor = None
        self.sync_data_meta = sync_buffer_meta
        self.sync_buffer = None
```
**EN:** This method implements `__init__` on `CudaIpcTensorTransportProxy`. It primarily calls `TypeError`, `self.get_proxy_state`, `isinstance`, `data.stride`, `type`, `data.numel` to complete its work. State updates are written into `self.reconstruct_tensor`, `self.sync_data_meta`, `self.sync_buffer`, `self.proxy_state`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `CudaIpcTensorTransportProxy`）实现了 `__init__`。 它主要通过调用 `TypeError`, `self.get_proxy_state`, `isinstance`, `data.stride`, `type`, `data.numel` 来完成任务。 状态更新主要写入 `self.reconstruct_tensor`, `self.sync_data_meta`, `self.sync_buffer`, `self.proxy_state`。 实现中使用了条件分支。

### Lines 331-339: Method `CudaIpcTensorTransportProxy.get_sync_flag` / 方法 `CudaIpcTensorTransportProxy.get_sync_flag`
```python
    @property
    def get_sync_flag(self):
        if not self.sync_buffer:
            shm_name = self.sync_data_meta["handle"]
            self.sync_buffer = shared_memory.SharedMemory(name=shm_name)

        shape = self.sync_data_meta["shape"]
        dtype = self.sync_data_meta["dtype"]
        return np.ndarray(shape, dtype=dtype, buffer=self.sync_buffer.buf)
```
**EN:** This method implements `get_sync_flag` on `CudaIpcTensorTransportProxy`. It primarily calls `np.ndarray`, `shared_memory.SharedMemory` to complete its work. State updates are written into `shape`, `dtype`, `shm_name`, `self.sync_buffer`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `CudaIpcTensorTransportProxy`）实现了 `get_sync_flag`。 它主要通过调用 `np.ndarray`, `shared_memory.SharedMemory` 来完成任务。 状态更新主要写入 `shape`, `dtype`, `shm_name`, `self.sync_buffer`。 实现中使用了条件分支。

### Lines 341-343: Method `CudaIpcTensorTransportProxy.close_shm` / 方法 `CudaIpcTensorTransportProxy.close_shm`
```python
    def close_shm(self):
        self.sync_buffer.close()
        self.sync_buffer = None
```
**EN:** This method implements `close_shm` on `CudaIpcTensorTransportProxy`. It primarily calls `self.sync_buffer.close` to complete its work. State updates are written into `self.sync_buffer`.
**CN:** 该方法（属于 `CudaIpcTensorTransportProxy`）实现了 `close_shm`。 它主要通过调用 `self.sync_buffer.close` 来完成任务。 状态更新主要写入 `self.sync_buffer`。

### Lines 345-369: Method `CudaIpcTensorTransportProxy.get_proxy_state` / 方法 `CudaIpcTensorTransportProxy.get_proxy_state`
```python
    def get_proxy_state(self, data, info_data):
        # acquire all serialize metadata from _metadata
        state = {}

        try:
            storage = data.untyped_storage()
            handle = storage._share_cuda_()

            state["ipc_extra"] = {
                "handle": handle,
                "shape": data.shape,
                "dtype": data.dtype,
                "stride": data.stride(),
                "device_index": data.device.index,
                "storage_offset": data.storage_offset(),
                "recons_shape": info_data.shape,
                "recons_dtype": info_data.dtype,
            }
            state["tensor_data"] = None
        except Exception as e:
            # Failed to get CUDA IPC handle (possibly tp). Falling back to default transport.
            state["ipc_extra"] = None
            state["tensor_data"] = data

        return state
```
**EN:** This method implements `get_proxy_state` on `CudaIpcTensorTransportProxy`. It primarily calls `data.untyped_storage`, `storage._share_cuda_`, `data.stride`, `data.storage_offset` to complete its work. State updates are written into `state`, `storage`, `handle`. The implementation relies on error handling.
**CN:** 该方法（属于 `CudaIpcTensorTransportProxy`）实现了 `get_proxy_state`。 它主要通过调用 `data.untyped_storage`, `storage._share_cuda_`, `data.stride`, `data.storage_offset` 来完成任务。 状态更新主要写入 `state`, `storage`, `handle`。 实现中使用了错误处理。

### Lines 371-402: Method `CudaIpcTensorTransportProxy._reconstruct_from_ipc_extra` / 方法 `CudaIpcTensorTransportProxy._reconstruct_from_ipc_extra`
```python
    def _reconstruct_from_ipc_extra(
        self, ipc_extra, *, use_cache: bool, rebuild_device_idx: int
    ):
        shape = ipc_extra["shape"]
        dtype = ipc_extra["dtype"]
        stride = ipc_extra["stride"]
        # Redirect handle[0] to the consumer's device so _new_shared_cuda's
        # CUDAGuard stays there; peer access handles the cross-GPU open.
        pool_handle = ipc_extra["pool_handle"]
        redirected_handle = (rebuild_device_idx,) + tuple(pool_handle)[1:]
        target_device = torch.device(f"cuda:{rebuild_device_idx}")
        cache_key = _normalize_pool_cache_key(pool_handle, rebuild_device_idx)

        with torch.cuda.device(target_device):
            if use_cache:
                storage = _pool_handle_cache_get_or_open(cache_key, redirected_handle)
                storage_to_cache = None
            else:
                storage = _open_pooled_storage_uncached(redirected_handle)
                storage_to_cache = storage
            slice_storage = storage[
                ipc_extra["pool_byte_offset"] : ipc_extra["pool_byte_offset"]
                + ipc_extra["nbytes"]
            ]
            slice_tensor = torch.empty(0, dtype=dtype, device=target_device).set_(
                slice_storage,
                storage_offset=ipc_extra["storage_offset"],
                size=shape,
                stride=stride,
            )

        return slice_tensor, target_device, cache_key, storage_to_cache
```
**EN:** This method implements `_reconstruct_from_ipc_extra` on `CudaIpcTensorTransportProxy`. It primarily calls `torch.device`, `_normalize_pool_cache_key`, `torch.cuda.device`, `torch.empty.set_`, `tuple`, `_pool_handle_cache_get_or_open` to complete its work. State updates are written into `shape`, `dtype`, `stride`, `pool_handle`, `redirected_handle`, `target_device`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该方法（属于 `CudaIpcTensorTransportProxy`）实现了 `_reconstruct_from_ipc_extra`。 它主要通过调用 `torch.device`, `_normalize_pool_cache_key`, `torch.cuda.device`, `torch.empty.set_`, `tuple`, `_pool_handle_cache_get_or_open` 来完成任务。 状态更新主要写入 `shape`, `dtype`, `stride`, `pool_handle`, `redirected_handle`, `target_device`。 实现中使用了条件分支、上下文管理资源。

### Lines 404-427: Method `CudaIpcTensorTransportProxy._copy_slice_tensor_to_target` / 方法 `CudaIpcTensorTransportProxy._copy_slice_tensor_to_target`
```python
    def _copy_slice_tensor_to_target(
        self,
        slice_tensor: torch.Tensor,
        rebuild_device: torch.device,
        recons_shape,
        recons_dtype,
    ):
        with torch.cuda.device(rebuild_device):
            reconstructed_tensor = torch.empty(
                recons_shape, dtype=recons_dtype, device=rebuild_device
            ).contiguous()
            reconstructed_tensor.view(torch.int8).view(-1).copy_(slice_tensor)

            open(SHM_LOCK_FILE, "a").close()
            # write the shm_sync_buffer with a file lock
            with open(SHM_LOCK_FILE, "w+") as f:
                fcntl.flock(f, fcntl.LOCK_EX)
                sync_flag = self.get_sync_flag
                sync_flag += 1
                fcntl.flock(f, fcntl.LOCK_UN)

            self.close_shm()

        return reconstructed_tensor
```
**EN:** This method implements `_copy_slice_tensor_to_target` on `CudaIpcTensorTransportProxy`. It primarily calls `torch.cuda.device`, `torch.empty.contiguous`, `reconstructed_tensor.view.view.copy_`, `open.close`, `self.close_shm`, `open` to complete its work. State updates are written into `reconstructed_tensor`, `sync_flag`. The implementation relies on context-managed resources.
**CN:** 该方法（属于 `CudaIpcTensorTransportProxy`）实现了 `_copy_slice_tensor_to_target`。 它主要通过调用 `torch.cuda.device`, `torch.empty.contiguous`, `reconstructed_tensor.view.view.copy_`, `open.close`, `self.close_shm`, `open` 来完成任务。 状态更新主要写入 `reconstructed_tensor`, `sync_flag`。 实现中使用了上下文管理资源。

### Lines 429-511: Method `CudaIpcTensorTransportProxy.reconstruct_on_target_device` / 方法 `CudaIpcTensorTransportProxy.reconstruct_on_target_device`
```python
    def reconstruct_on_target_device(self, rebuild_device_idx):
        rebuild_device = torch.device(f"cuda:{rebuild_device_idx}")
        if (
            isinstance(self.reconstruct_tensor, torch.Tensor)
            and self.reconstruct_tensor.device == rebuild_device
        ):
            return self.reconstruct_tensor

        if self.proxy_state["ipc_extra"]:
            ipc_extra = self.proxy_state["ipc_extra"]
            recons_shape = ipc_extra["recons_shape"]
            recons_dtype = ipc_extra["recons_dtype"]

            if "pool_handle" in ipc_extra:
                try:
                    (
                        slice_tensor,
                        _target_device,
                        cache_key,
                        storage_to_cache,
                    ) = self._reconstruct_from_ipc_extra(
                        ipc_extra,
                        use_cache=True,
                        rebuild_device_idx=rebuild_device_idx,
                    )
                except Exception as e:
                    cache_key = _normalize_pool_cache_key(
                        ipc_extra["pool_handle"], rebuild_device_idx
# ... omitted for brevity ...
            )
        else:
            raise TypeError("invalid proxy_state")

        self.reconstruct_tensor = reconstructed_tensor
        return self.reconstruct_tensor
```
**EN:** This method implements `reconstruct_on_target_device` on `CudaIpcTensorTransportProxy`. It primarily calls `torch.device`, `isinstance`, `self._copy_slice_tensor_to_target`, `self.proxy_state.to`, `TypeError`, `self._reconstruct_from_ipc_extra` to complete its work. State updates are written into `rebuild_device`, `self.reconstruct_tensor`, `ipc_extra`, `recons_shape`, `recons_dtype`, `reconstructed_tensor`. The implementation relies on conditional branches, context-managed resources, error handling.
**CN:** 该方法（属于 `CudaIpcTensorTransportProxy`）实现了 `reconstruct_on_target_device`。 它主要通过调用 `torch.device`, `isinstance`, `self._copy_slice_tensor_to_target`, `self.proxy_state.to`, `TypeError`, `self._reconstruct_from_ipc_extra` 来完成任务。 状态更新主要写入 `rebuild_device`, `self.reconstruct_tensor`, `ipc_extra`, `recons_shape`, `recons_dtype`, `reconstructed_tensor`。 实现中使用了条件分支、上下文管理资源、错误处理。

## Key Concepts / 关键概念
- **Classes / 类**: `ShmSyncBuffer`, `MmItemMemoryChunk`, `MmItemMemoryPool`, `CudaIpcTensorTransportProxy`
- **Functions / 函数**: `_normalize_pool_cache_key`, `_open_pooled_storage_uncached`, `_pool_handle_cache_get_or_open`, `_pool_handle_cache_set`, `_pool_handle_cache_invalidate`, `_pool_handle_cache_clear`, `__init__`, `__del__`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.environ`, `sglang.srt.server_args`
- **External / 外部依赖**: `numpy`, `torch`
- **Standard library / 标准库**: `fcntl`, `logging`, `threading`, `time`, `multiprocessing`, `typing`
