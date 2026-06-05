# swa_memory_pool.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/swa_memory_pool.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the SWA memory pool logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的SWA内存池相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and setup / 导入与初始化
```python
import logging
from typing import Dict, List, Optional, Tuple

import torch

from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.mem_cache.allocator import (
    BaseTokenToKVPoolAllocator,
    PagedTokenToKVPoolAllocator,
    TokenToKVPoolAllocator,
)
from sglang.srt.mem_cache.base_swa_memory_pool import BaseSWAKVPool
from sglang.srt.mem_cache.memory_pool import KVCache, MHATokenToKVPool
from sglang.srt.mem_cache.utils import maybe_init_custom_mem_pool
from sglang.srt.utils import is_npu
from sglang.srt.utils.common import get_num_new_pages
```
**EN:** Imports `logging`, `typing`, `torch`, `sglang.srt.layers.radix_attention`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.base_swa_memory_pool` and other helpers used by the surrounding scope.
**CN:** 导入 `logging`, `typing`, `torch`, `sglang.srt.layers.radix_attention`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.base_swa_memory_pool` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 18-18: Shared state definitions / 共享状态定义
```python
_is_npu = is_npu()
```
**EN:** Defines module-level variables such as `_is_npu`.
**CN:** 定义模块级变量，例如 `_is_npu`。

### Lines 20-23: Conditional logic / 条件逻辑
```python
if _is_npu:
    from sglang.srt.hardware_backend.npu.allocator_npu import (
        NPUPagedTokenToKVPoolAllocator,
    )
```
**EN:** Handles branching controlled by `_is_npu`.
**CN:** 处理由 `_is_npu` 控制的分支逻辑。

### Lines 25-26: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
GB = 1024 * 1024 * 1024
```
**EN:** Defines module-level variables such as `logger`, `GB`.
**CN:** 定义模块级变量，例如 `logger`, `GB`。

### Lines 29-31: SWAKVPool declaration / SWAKVPool 声明
```python
class SWAKVPool(BaseSWAKVPool):
    """KV cache with separate pools for full and SWA attention layers."""

```
**EN:** KV cache with separate pools for full and SWA attention layers. Declares the `SWAKVPool` class and connects it to `BaseSWAKVPool`.
**CN:** 声明 `SWAKVPool` 类，并将其关联到 `BaseSWAKVPool`。

### Lines 32-101: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        size_swa: int,
        page_size: int,
        dtype: torch.dtype,
        head_num: int,
        head_dim: int,
        swa_attention_layer_ids: List[int],
        full_attention_layer_ids: List[int],
        enable_kvcache_transpose: bool,
        device: str,
# ... omitted for brevity ...
        self.mem_usage = (k_size + v_size) / GB
        logger.info(
            f"SWAKVPool mem usage: {self.mem_usage:.2f} GB, swa size: {self.size_swa}, full size: {self.size}"
        )
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `SWAKVPool`. The implementation iterates over inputs or managed entries.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `SWAKVPool`。实现过程中会遍历输入或受管条目。

### Lines 103-104: register_mapping implementation / register_mapping 实现
```python
    def register_mapping(self, full_to_swa_index_mapping: torch.Tensor):
        self.full_to_swa_index_mapping = full_to_swa_index_mapping
```
**EN:** Registers metadata so other components can discover this object. It belongs to `SWAKVPool`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `SWAKVPool`。

### Lines 106-110: register_layer_transfer_counter implementation / register_layer_transfer_counter 实现
```python
    def register_layer_transfer_counter(self, layer_transfer_counter):
        # Wait happens at this wrapper. Inner pools must not wait again.
        self.layer_transfer_counter = layer_transfer_counter
        self.full_kv_pool.register_layer_transfer_counter(None)
        self.swa_kv_pool.register_layer_transfer_counter(None)
```
**EN:** Registers metadata so other components can discover this object. It belongs to `SWAKVPool`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `SWAKVPool`。

### Lines 112-114: _wait_for_layer implementation / _wait_for_layer 实现
```python
    def _wait_for_layer(self, layer_id: int):
        if self.layer_transfer_counter is not None:
            self.layer_transfer_counter.wait_until(layer_id - self.start_layer)
```
**EN:** Implements the wait for layer routine for this scope. It belongs to `SWAKVPool`.
**CN:** 实现当前作用域中的wait for layer例程。 该方法属于 `SWAKVPool`。

### Lines 116-119: get_kv_size_bytes implementation / get_kv_size_bytes 实现
```python
    def get_kv_size_bytes(self):
        k_size, v_size = self.full_kv_pool.get_kv_size_bytes()
        k_size_swa, v_size_swa = self.swa_kv_pool.get_kv_size_bytes()
        return k_size + k_size_swa, v_size + v_size_swa
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `SWAKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `SWAKVPool`。它会向调用方返回计算结果。

### Lines 121-129: get_contiguous_buf_infos implementation / get_contiguous_buf_infos 实现
```python
    def get_contiguous_buf_infos(self):
        full_kv_data_ptrs, full_kv_data_lens, full_kv_item_lens = (
            self.full_kv_pool.get_contiguous_buf_infos()
        )
        return (
            full_kv_data_ptrs,
            full_kv_data_lens,
            full_kv_item_lens,
        )
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `SWAKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `SWAKVPool`。它会向调用方返回计算结果。

### Lines 131-136: get_state_buf_infos implementation / get_state_buf_infos 实现
```python
    def get_state_buf_infos(self):
        swa_kv_data_ptrs, swa_kv_data_lens, swa_kv_item_lens = (
            self.swa_kv_pool.get_contiguous_buf_infos()
        )

        return swa_kv_data_ptrs, swa_kv_data_lens, swa_kv_item_lens
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `SWAKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `SWAKVPool`。它会向调用方返回计算结果。

### Lines 138-144: get_key_buffer implementation / get_key_buffer 实现
```python
    def get_key_buffer(self, layer_id: int):
        self._wait_for_layer(layer_id)
        layer_id_pool, is_swa_layer = self.layers_mapping[layer_id]
        if is_swa_layer:
            return self.swa_kv_pool.get_key_buffer(layer_id_pool)
        else:
            return self.full_kv_pool.get_key_buffer(layer_id_pool)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `SWAKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `SWAKVPool`。它会向调用方返回计算结果。

### Lines 146-152: get_value_buffer implementation / get_value_buffer 实现
```python
    def get_value_buffer(self, layer_id: int):
        self._wait_for_layer(layer_id)
        layer_id_pool, is_swa_layer = self.layers_mapping[layer_id]
        if is_swa_layer:
            return self.swa_kv_pool.get_value_buffer(layer_id_pool)
        else:
            return self.full_kv_pool.get_value_buffer(layer_id_pool)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `SWAKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `SWAKVPool`。它会向调用方返回计算结果。

### Lines 154-160: get_kv_buffer implementation / get_kv_buffer 实现
```python
    def get_kv_buffer(self, layer_id: int):
        self._wait_for_layer(layer_id)
        layer_id_pool, is_swa_layer = self.layers_mapping[layer_id]
        if is_swa_layer:
            return self.swa_kv_pool.get_kv_buffer(layer_id_pool)
        else:
            return self.full_kv_pool.get_kv_buffer(layer_id_pool)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `SWAKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `SWAKVPool`。它会向调用方返回计算结果。

### Lines 162-163: set_swa_loc implementation / set_swa_loc 实现
```python
    def set_swa_loc(self, loc: torch.Tensor):
        self.swa_loc = loc
```
**EN:** Updates internal state with the provided value. It belongs to `SWAKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `SWAKVPool`。

### Lines 165-170: translate_loc_from_full_to_swa implementation / translate_loc_from_full_to_swa 实现
```python
    def translate_loc_from_full_to_swa(self, kv_indices: torch.Tensor):
        assert self.full_to_swa_index_mapping is not None

        # Note: kv_indices could have -1 values (from alloc_extend), which will be mapped to -1
        # since the last item of full_to_swa_index_mapping is -1.
        return self.full_to_swa_index_mapping[kv_indices].to(torch.int32)
```
**EN:** Implements the translate LOC from full TO SWA routine for this scope. It belongs to `SWAKVPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc from完整to SWA例程。 该方法属于 `SWAKVPool`。它会向调用方返回计算结果。

### Lines 172-209: set_kv_buffer implementation / set_kv_buffer 实现
```python
    def set_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
        cache_v: torch.Tensor,
        k_scale: float = 1.0,
        v_scale: float = 1.0,
    ):

        layer_id = layer.layer_id
        layer_id_pool, is_swa_layer = self.layers_mapping[layer_id]
# ... omitted for brevity ...
                k_scale,
                v_scale,
                layer_id_override=layer_id_pool,
            )
```
**EN:** Updates internal state with the provided value. It belongs to `SWAKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `SWAKVPool`。

### Lines 211-215: move_kv_cache implementation / move_kv_cache 实现
```python
    def move_kv_cache(self, tgt_loc: torch.Tensor, src_loc: torch.Tensor):
        self.full_kv_pool.move_kv_cache(tgt_loc, src_loc)
        tgt_loc_swa = self.translate_loc_from_full_to_swa(tgt_loc)
        src_loc_swa = self.translate_loc_from_full_to_swa(src_loc)
        self.swa_kv_pool.move_kv_cache(tgt_loc_swa, src_loc_swa)
```
**EN:** Implements the move KV cache routine for this scope. It belongs to `SWAKVPool`.
**CN:** 实现当前作用域中的move KV缓存例程。 该方法属于 `SWAKVPool`。

### Lines 217-243: _filter_swa_cpu_copy implementation / _filter_swa_cpu_copy 实现
```python
    def _filter_swa_cpu_copy(self, swa_kv_cpu, row_mask: torch.Tensor):
        if swa_kv_cpu is None:
            return None
        if row_mask is None or bool(torch.all(row_mask).item()):
            return swa_kv_cpu

        chunk_size = getattr(
            self.swa_kv_pool, "cpu_offloading_chunk_size", len(row_mask)
        )
        filtered = []
        for layer_chunks in swa_kv_cpu:
            if len(layer_chunks) == 0:
# ... omitted for brevity ...
                    [k_cpu[i : i + chunk_size], v_cpu[i : i + chunk_size]]
                )
            filtered.append(filtered_layer)
        return filtered
```
**EN:** Implements the filter SWA CPU copy routine for this scope. It belongs to `SWAKVPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的filter SWA CPU copy例程。 该方法属于 `SWAKVPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 245-265: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices, mamba_indices=None):
        # For SWA, we need to copy KV cache from both full and SWA pools
        # The indices are for the full pool, and we use mapping to get SWA indices
        full_kv_cpu = self.full_kv_pool.get_cpu_copy(indices)

        swa_mask = None
        if self.full_to_swa_index_mapping is not None:
            swa_indices = self.full_to_swa_index_mapping[indices]
            # Slot 0 is reserved as a dummy slot. Tail-only SWA allocations leave
            # the out-of-window full KV indices unmapped, so only copy mapped SWA
            # tokens and keep their positions for load_cpu_copy().
            swa_mask = swa_indices > 0
# ... omitted for brevity ...
        else:
            swa_kv_cpu = None

        return {"full": full_kv_cpu, "swa": swa_kv_cpu, "swa_mask": swa_mask}
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `SWAKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `SWAKVPool`。它会向调用方返回计算结果。

### Lines 267-293: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, kv_cache_cpu, indices, mamba_indices=None):
        # Load KV cache back from CPU to both full and SWA pools
        # Note: indices here are NEW indices (newly allocated), different from get_cpu_copy indices
        full_kv_cpu = kv_cache_cpu["full"]
        swa_kv_cpu = kv_cache_cpu["swa"]

        # Load full KV cache to the new indices
        self.full_kv_pool.load_cpu_copy(full_kv_cpu, indices)

        # Load SWA KV cache if it exists
        if swa_kv_cpu is not None and self.full_to_swa_index_mapping is not None:
            swa_indices = self.full_to_swa_index_mapping[indices]
# ... omitted for brevity ...
                return

            swa_kv_cpu = self._filter_swa_cpu_copy(swa_kv_cpu, row_mask)
            self.swa_kv_pool.load_cpu_copy(swa_kv_cpu, swa_indices)
```
**EN:** Loads state from an external or serialized representation. It belongs to `SWAKVPool`. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `SWAKVPool`。它会向调用方返回计算结果。

### Lines 296-298: SWATokenToKVPoolAllocator declaration / SWATokenToKVPoolAllocator 声明
```python
class SWATokenToKVPoolAllocator(BaseTokenToKVPoolAllocator):
    """Allocator for SWA hybrid KV cache."""

```
**EN:** Allocator for SWA hybrid KV cache. Declares the `SWATokenToKVPoolAllocator` class and connects it to `BaseTokenToKVPoolAllocator`.
**CN:** 声明 `SWATokenToKVPoolAllocator` 类，并将其关联到 `BaseTokenToKVPoolAllocator`。

### Lines 299-377: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        size_swa: int,
        page_size: int,
        dtype: torch.dtype,
        device: str,
        kvcache: BaseSWAKVPool,
        need_sort: bool,
    ):
        assert isinstance(kvcache, BaseSWAKVPool)
        self._size_full = size
# ... omitted for brevity ...

        self.clear()
        self._kvcache = kvcache
        self._kvcache.register_mapping(self.full_to_swa_index_mapping)
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `SWATokenToKVPoolAllocator`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `SWATokenToKVPoolAllocator`。

### Lines 379-383: available_size implementation / available_size 实现
```python
    def available_size(self):
        return min(
            self.full_attn_allocator.available_size(),
            self.swa_attn_allocator.available_size(),
        )
```
**EN:** Implements the available size routine for this scope. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available size例程。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 385-386: full_available_size implementation / full_available_size 实现
```python
    def full_available_size(self):
        return self.full_attn_allocator.available_size()
```
**EN:** Implements the full available size routine for this scope. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的完整available size例程。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 388-389: swa_available_size implementation / swa_available_size 实现
```python
    def swa_available_size(self):
        return self.swa_attn_allocator.available_size()
```
**EN:** Implements the SWA available size routine for this scope. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的SWA available size例程。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 391-393: size implementation / size 实现
```python
    @property
    def size(self):
        return min(self._size_full, self._size_swa)
```
**EN:** Implements the size routine for this scope. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的size例程。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 395-397: size_swa implementation / size_swa 实现
```python
    @property
    def size_swa(self):
        return self._size_swa
```
**EN:** Implements the size SWA routine for this scope. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的size SWA例程。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 399-401: size_full implementation / size_full 实现
```python
    @property
    def size_full(self):
        return self._size_full
```
**EN:** Implements the size full routine for this scope. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的size完整例程。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 403-409: debug_print implementation / debug_print 实现
```python
    def debug_print(self) -> str:
        msg = ""
        msg += f"#swa-available-size: {self.swa_attn_allocator.available_size()}, "
        msg += (
            f"#full-attn-available-size: {self.full_attn_allocator.available_size()}, "
        )
        return msg
```
**EN:** Implements the debug print routine for this scope. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的debug print例程。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 411-412: get_kvcache implementation / get_kvcache 实现
```python
    def get_kvcache(self):
        return self._kvcache
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 414-416: translate_loc_from_full_to_swa implementation / translate_loc_from_full_to_swa 实现
```python
    def translate_loc_from_full_to_swa(self, kv_indices: torch.Tensor):
        assert self._kvcache.full_to_swa_index_mapping is not None
        return self._kvcache.translate_loc_from_full_to_swa(kv_indices)
```
**EN:** Implements the translate LOC from full TO SWA routine for this scope. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc from完整to SWA例程。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 418-436: alloc implementation / alloc 实现
```python
    def alloc(self, need_size: int):
        assert self.page_size == 1
        if need_size > self.full_attn_allocator.available_size():
            return None
        if need_size > self.swa_attn_allocator.available_size():
            return None

        alloc_full_indices = self.full_attn_allocator.alloc(need_size)
        alloc_swa_indices = self.swa_attn_allocator.alloc(need_size)
        assert alloc_full_indices is not None
        assert alloc_swa_indices is not None

# ... omitted for brevity ...
            )
        else:
            self.full_to_swa_index_mapping[alloc_full_indices] = alloc_swa_indices
        return alloc_full_indices
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 438-487: alloc_extend implementation / alloc_extend 实现
```python
    def alloc_extend(
        self,
        prefix_lens: torch.Tensor,
        prefix_lens_cpu: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
        last_loc: torch.Tensor,  # last_loc for full layers
        extend_num_tokens: int,
    ):
        assert self.page_size > 1

        num_new_pages = get_num_new_pages(
# ... omitted for brevity ...
        else:
            self.full_to_swa_index_mapping[alloc_full_indices] = alloc_swa_indices

        return alloc_full_indices
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 489-552: alloc_extend_swa_tail implementation / alloc_extend_swa_tail 实现
```python
    def alloc_extend_swa_tail(
        self,
        prefix_lens: torch.Tensor,
        prefix_lens_cpu: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
        last_loc: torch.Tensor,  # last_loc for full layers
        extend_num_tokens: int,
        swa_tail_len: int,
    ):
        """Allocate full KV for the whole extend and SWA KV only for the tail.

# ... omitted for brevity ...
        self.full_to_swa_index_mapping[alloc_full_indices[-swa_tail_len:]] = (
            alloc_swa_indices
        )
        return alloc_full_indices
```
**EN:** Allocate full KV for the whole extend and SWA KV only for the tail. Allocates cache or memory resources for the requested workload. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 554-580: alloc_decode implementation / alloc_decode 实现
```python
    def alloc_decode(
        self,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
        last_loc: torch.Tensor,  # last_loc for full layers
    ):
        assert self.page_size > 1
        swa_last_loc = self.translate_loc_from_full_to_swa(last_loc)

        alloc_full_indices = self.full_attn_allocator.alloc_decode(
            seq_lens, seq_lens_cpu, last_loc
        )
# ... omitted for brevity ...
        else:
            self.full_to_swa_index_mapping[alloc_full_indices] = alloc_swa_indices

        return alloc_full_indices
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 582-595: free implementation / free 实现
```python
    def free(self, free_index: torch.Tensor):
        if free_index.numel() == 0:
            return

        # NOTE: the API is not idempotent.
        if self.is_not_in_free_group:
            self.full_attn_allocator.free(free_index)
            self.free_swa(free_index)
        else:
            self.free_group.append(free_index)
        assert (
            self.full_attn_allocator.available_size() <= self.full_attn_allocator.size
        )
        assert self.swa_attn_allocator.available_size() <= self.swa_attn_allocator.size
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 将已分配资源释放回资源池。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 597-612: set_full_to_swa_mapping implementation / set_full_to_swa_mapping 实现
```python
    def set_full_to_swa_mapping(
        self, full_indices: torch.Tensor, swa_indices: torch.Tensor
    ) -> None:
        """Write full_to_swa_index_mapping[full_indices[i]] = swa_indices[i].

        Used by HiCache load-back path to rebuild the mapping after FULL and SWA device alloc.
        """
        if full_indices.numel() == 0:
            return
        assert full_indices.numel() == swa_indices.numel()
        if _is_npu:
            self.full_to_swa_index_mapping[full_indices.to(torch.int64)] = (
                swa_indices.to(torch.int64)
            )
        else:
            self.full_to_swa_index_mapping[full_indices] = swa_indices
```
**EN:** Write full_to_swa_index_mapping[full_indices[i]] = swa_indices[i]. Updates internal state with the provided value. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 614-618: free_swa implementation / free_swa 实现
```python
    def free_swa(self, free_index: torch.Tensor):
        swa_indices = self.full_to_swa_index_mapping[free_index]
        swa_indices = swa_indices[swa_indices > 0]
        self.swa_attn_allocator.free(swa_indices)
        self.full_to_swa_index_mapping[free_index] = 0
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `SWATokenToKVPoolAllocator`.
**CN:** 将已分配资源释放回资源池。 该方法属于 `SWATokenToKVPoolAllocator`。

### Lines 620-624: backup_state implementation / backup_state 实现
```python
    def backup_state(self):
        return [
            self.full_attn_allocator.backup_state(),
            self.swa_attn_allocator.backup_state(),
        ]
```
**EN:** Implements the backup state routine for this scope. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的backup状态例程。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 626-629: restore_state implementation / restore_state 实现
```python
    def restore_state(self, state):
        assert len(state) == 2
        self.full_attn_allocator.restore_state(state[0])
        self.swa_attn_allocator.restore_state(state[1])
```
**EN:** Implements the restore state routine for this scope. It belongs to `SWATokenToKVPoolAllocator`.
**CN:** 实现当前作用域中的restore状态例程。 该方法属于 `SWATokenToKVPoolAllocator`。

### Lines 631-637: clear implementation / clear 实现
```python
    def clear(self):
        self.swa_attn_allocator.clear()
        self.full_attn_allocator.clear()
        # Note: the last item is -1, we don't clear it, see the comment in __init__
        self.full_to_swa_index_mapping[:-1].fill_(0)
        self.is_not_in_free_group = True
        self.free_group = []
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `SWATokenToKVPoolAllocator`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `SWATokenToKVPoolAllocator`。

### Lines 639-640: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices, mamba_indices=None):
        return self._kvcache.get_cpu_copy(indices, mamba_indices=mamba_indices)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 642-645: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, kv_cache_cpu, indices, mamba_indices=None):
        return self._kvcache.load_cpu_copy(
            kv_cache_cpu, indices, mamba_indices=mamba_indices
        )
```
**EN:** Loads state from an external or serialized representation. It belongs to `SWATokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `SWATokenToKVPoolAllocator`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`SWAKVPool`**: Defines the `SWAKVPool` type and its core responsibilities. / 定义 `SWAKVPool` 类型及其核心职责。
- **`SWATokenToKVPoolAllocator`**: Defines the `SWATokenToKVPoolAllocator` type and its core responsibilities. / 定义 `SWATokenToKVPoolAllocator` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `logging`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.layers.radix_attention`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.base_swa_memory_pool`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.utils`, `sglang.srt.utils`, `sglang.srt.utils.common`, `sglang.srt.hardware_backend.npu.allocator_npu`
