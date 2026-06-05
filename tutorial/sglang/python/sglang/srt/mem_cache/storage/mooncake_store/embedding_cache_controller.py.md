# embedding_cache_controller.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/storage/mooncake_store/embedding_cache_controller.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the embedding cache controller logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的嵌入缓存控制器相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and setup / 导入与初始化
```python
import asyncio
import logging
import threading
import time
from queue import Empty, Queue
from typing import List, Optional

import torch

from sglang.srt.mem_cache.storage.mooncake_store.mooncake_embedding_store import (
    MooncakeEmbeddingStore,
)
```
**EN:** Imports `asyncio`, `logging`, `threading`, `time`, `queue`, `typing` and other helpers used by the surrounding scope.
**CN:** 导入 `asyncio`, `logging`, `threading`, `time`, `queue`, `typing` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 14-14: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 17-22: ContiguousMemoryAllocator declaration / ContiguousMemoryAllocator 声明
```python
class ContiguousMemoryAllocator:
    """
    A simple allocator to manage variable-sized contiguous blocks
    within a large pre-allocated flat buffer.
    """

```
**EN:** A simple allocator to manage variable-sized contiguous blocks within a large pre-allocated flat buffer. Declares the `ContiguousMemoryAllocator` class.
**CN:** 声明 `ContiguousMemoryAllocator` 类。

### Lines 23-28: __init__ implementation / __init__ 实现
```python
    def __init__(self, total_size_bytes: int):
        self.total_size = total_size_bytes
        # List of (offset, size) for free blocks
        self.free_blocks = [(0, total_size_bytes)]
        self.allocated_map = {}  # {handle: (offset, size)}
        self.lock = threading.Lock()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `ContiguousMemoryAllocator`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `ContiguousMemoryAllocator`。

### Lines 30-42: allocate implementation / allocate 实现
```python
    def allocate(self, size_bytes: int) -> Optional[int]:
        with self.lock:
            # Simple First-Fit allocation
            for i, (offset, block_size) in enumerate(self.free_blocks):
                if block_size >= size_bytes:
                    # Allocate from this block
                    remaining_size = block_size - size_bytes
                    if remaining_size > 0:
                        self.free_blocks[i] = (offset + size_bytes, remaining_size)
                    else:
                        self.free_blocks.pop(i)
                    return offset
            return None
```
**EN:** Implements the allocate routine for this scope. It belongs to `ContiguousMemoryAllocator`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的allocate例程。 该方法属于 `ContiguousMemoryAllocator`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 44-62: free implementation / free 实现
```python
    def free(self, offset: int, size_bytes: int):
        with self.lock:
            # Return block and merge adjacent free blocks
            self.free_blocks.append((offset, size_bytes))
            self.free_blocks.sort()

            merged = []
            if not self.free_blocks:
                return

            curr_offset, curr_size = self.free_blocks[0]
            for next_offset, next_size in self.free_blocks[1:]:
# ... omitted for brevity ...
                    merged.append((curr_offset, curr_size))
                    curr_offset, curr_size = next_offset, next_size
            merged.append((curr_offset, curr_size))
            self.free_blocks = merged
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `ContiguousMemoryAllocator`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 将已分配资源释放回资源池。 该方法属于 `ContiguousMemoryAllocator`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 65-67: EmbeddingPrefetchOperation declaration / EmbeddingPrefetchOperation 声明
```python
class EmbeddingPrefetchOperation:
    """Groups all missing images of a request for a single batch GET."""

```
**EN:** Groups all missing images of a request for a single batch GET. Declares the `EmbeddingPrefetchOperation` class.
**CN:** 声明 `EmbeddingPrefetchOperation` 类。

### Lines 68-75: __init__ implementation / __init__ 实现
```python
    def __init__(self, req_id: str, keys: List[str], ptrs: List[int], sizes: List[int]):
        self.req_id = req_id
        self.keys = keys
        self.ptrs = ptrs
        self.sizes = sizes
        self.is_finished = False
        self.success = False
        self._lock = threading.Lock()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `EmbeddingPrefetchOperation`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `EmbeddingPrefetchOperation`。

### Lines 77-80: mark_done implementation / mark_done 实现
```python
    def mark_done(self, success: bool):
        with self._lock:
            self.success = success
            self.is_finished = True
```
**EN:** Implements the mark done routine for this scope. It belongs to `EmbeddingPrefetchOperation`.
**CN:** 实现当前作用域中的mark done例程。 该方法属于 `EmbeddingPrefetchOperation`。

### Lines 83-85: EmbeddingInsertOperation declaration / EmbeddingInsertOperation 声明
```python
class EmbeddingInsertOperation:
    """Groups all newly computed images of a request for a single batch PUT."""

```
**EN:** Groups all newly computed images of a request for a single batch PUT. Declares the `EmbeddingInsertOperation` class.
**CN:** 声明 `EmbeddingInsertOperation` 类。

### Lines 86-89: __init__ implementation / __init__ 实现
```python
    def __init__(self, keys: List[str], ptrs: List[int], sizes: List[int]):
        self.keys = keys
        self.ptrs = ptrs
        self.sizes = sizes
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `EmbeddingInsertOperation`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `EmbeddingInsertOperation`。

### Lines 92-92: EmbeddingCacheController declaration / EmbeddingCacheController 声明
```python
class EmbeddingCacheController:
```
**EN:** Declares the `EmbeddingCacheController` class.
**CN:** 声明 `EmbeddingCacheController` 类。

### Lines 93-143: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        tp_rank,
        tp_size,
        max_pool_size_gb=4.0,
        hidden_dims: dict = None,
        tp_group=None,
        all_rank_get=False,
    ):
        self.tp_world_size = tp_size
        self.tp_group = tp_group
        self.all_rank_get = all_rank_get
# ... omitted for brevity ...
                group_ranks=group_ranks, backend="gloo"
            )
        else:
            self.prefetch_tp_group = None
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `EmbeddingCacheController`. It validates error cases explicitly.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `EmbeddingCacheController`。它会显式处理错误场景。

### Lines 145-188: prefetch implementation / prefetch 实现
```python
    def prefetch(
        self,
        req_id: str,
        image_hashes: List[str],
        expected_tokens: List[int],
        modality=None,
    ):
        """Issues ONE batch GET for all missing images in the request."""
        dim = self.hidden_dims.get(modality) if modality is not None else None
        if not dim:
            logger.warning(
                f"Req {req_id}: Unknown dim for modality={modality}, skipping prefetch (will fallback to ViT)."
# ... omitted for brevity ...

            op = EmbeddingPrefetchOperation(req_id, keys, ptrs, sizes)
            self.ongoing_prefetch[req_id] = op
            self.prefetch_queue.put(op)
```
**EN:** Issues ONE batch GET for all missing images in the request. Implements the prefetch routine for this scope. It belongs to `EmbeddingCacheController`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的prefetch例程。 该方法属于 `EmbeddingCacheController`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 190-224: insert_batch implementation / insert_batch 实现
```python
    def insert_batch(
        self, image_hashes: List[str], embedding_tensors: List[torch.Tensor]
    ):
        """Issues ONE batch PUT for all embeddings computed by this request."""
        keys, ptrs, sizes = [], [], []

        with self.lock:
            for h, tensor in zip(image_hashes, embedding_tensors):
                if h in self.hash_to_metadata:
                    continue

                num_tokens, dim = tensor.shape[0], tensor.shape[1]
# ... omitted for brevity ...
                logger.info(
                    f"Global Cache: Inserting {len(keys)} new embeddings into Mooncake cluster."
                )
                self.insert_queue.put(EmbeddingInsertOperation(keys, ptrs, sizes))
```
**EN:** Issues ONE batch PUT for all embeddings computed by this request. Inserts new data into the managed structure. It belongs to `EmbeddingCacheController`. The implementation iterates over inputs or managed entries.
**CN:** 向受管结构中插入新数据。 该方法属于 `EmbeddingCacheController`。实现过程中会遍历输入或受管条目。

### Lines 226-256: _io_loop implementation / _io_loop 实现
```python
    def _io_loop(self):
        """Asynchronous worker handling both Batch GET and Batch PUT."""
        while not self.stop_event.is_set():
            processed_any = False

            try:
                op = self.prefetch_queue.get_nowait()
                results = self.mooncake_store.batch_get(op.keys, op.ptrs, op.sizes)
                success_count = sum(results)
                logger.info(
                    f"Mooncake GET Finished: Req {op.req_id}, Successfully fetched {success_count}/{len(op.keys)} images."
                )
# ... omitted for brevity ...
                pass

            if not processed_any:
                time.sleep(0.001)
```
**EN:** Asynchronous worker handling both Batch GET and Batch PUT. Implements the IO loop routine for this scope. It belongs to `EmbeddingCacheController`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的IO loop例程。 该方法属于 `EmbeddingCacheController`。实现过程中会遍历输入或受管条目。

### Lines 258-284: check_prefetch_progress implementation / check_prefetch_progress 实现
```python
    def check_prefetch_progress(self, req_id: str) -> bool:
        """TP-Group barrier: ensures all cards have the request batch ready."""
        local_ready = False
        with self.lock:
            if req_id not in self.ongoing_prefetch:
                local_ready = True
            else:
                op = self.ongoing_prefetch[req_id]
                if op.is_finished:
                    local_ready = op.success

        if self.all_rank_get and self.tp_world_size > 1:
# ... omitted for brevity ...
            with self.lock:
                self.ongoing_prefetch.pop(req_id, None)
            return True
        return False
```
**EN:** TP-Group barrier: ensures all cards have the request batch ready. Implements the check prefetch progress routine for this scope. It belongs to `EmbeddingCacheController`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的check prefetch progress例程。 该方法属于 `EmbeddingCacheController`。它会向调用方返回计算结果。

### Lines 286-297: get_embeddings implementation / get_embeddings 实现
```python
    def get_embeddings(self, image_hashes: List[str]) -> List[torch.Tensor]:
        """Final reconstruction for model input."""
        with self.lock:
            tensors = []
            for h in image_hashes:
                offset, num_tokens, dim, size_bytes = self.hash_to_metadata[h]
                tensors.append(
                    self.cpu_pool[offset : offset + size_bytes]
                    .view(torch.float32)
                    .view(num_tokens, dim)
                )
            return tensors
```
**EN:** Final reconstruction for model input. Retrieves the requested data or state from the current object. It belongs to `EmbeddingCacheController`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `EmbeddingCacheController`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 299-326: batch_is_exist implementation / batch_is_exist 实现
```python
    async def batch_is_exist(self, image_hashes: List[str]) -> List[bool]:
        with self.lock:
            local_results = [h in self.hash_to_metadata for h in image_hashes]
        local_hit_count = sum(local_results)

        global_hit_count = 0
        if not all(local_results):
            missing_indices = [i for i, res in enumerate(local_results) if not res]
            missing_hashes = [image_hashes[i] for i in missing_indices]

            global_exists = await asyncio.to_thread(
                self.mooncake_store.batch_is_exist, missing_hashes
# ... omitted for brevity ...
            f"Global Hits: {global_hit_count} | "
            f"Misses (GPU Work): {miss_count}"
        )
        return local_results
```
**EN:** Implements the batch IS exist routine for this scope. It belongs to `EmbeddingCacheController`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch is exist例程。 该方法属于 `EmbeddingCacheController`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`ContiguousMemoryAllocator`**: Defines the `ContiguousMemoryAllocator` type and its core responsibilities. / 定义 `ContiguousMemoryAllocator` 类型及其核心职责。
- **`EmbeddingPrefetchOperation`**: Defines the `EmbeddingPrefetchOperation` type and its core responsibilities. / 定义 `EmbeddingPrefetchOperation` 类型及其核心职责。
- **`EmbeddingInsertOperation`**: Defines the `EmbeddingInsertOperation` type and its core responsibilities. / 定义 `EmbeddingInsertOperation` 类型及其核心职责。
- **`EmbeddingCacheController`**: Defines the `EmbeddingCacheController` type and its core responsibilities. / 定义 `EmbeddingCacheController` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `asyncio`, `logging`, `threading`, `time`, `queue`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.mem_cache.storage.mooncake_store.mooncake_embedding_store`, `sglang.srt.distributed.parallel_state`
