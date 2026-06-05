# hisparse_memory_pool.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/hisparse_memory_pool.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the hisparse memory pool logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的hisparse内存池相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Comment block / 注释块
```python
# mapping on device memory, host memory and memory allocator

```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 3-21: Imports and setup / 导入与初始化
```python
import logging
import weakref
from typing import Optional

import psutil
import torch

from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.mem_cache.allocator import (
    BaseTokenToKVPoolAllocator,
    PagedTokenToKVPoolAllocator,
)
# ... omitted for brevity ...
)
from sglang.srt.mem_cache.memory_pool import NSATokenToKVPool
from sglang.srt.utils import is_cuda, is_hip
from sglang.srt.utils.common import get_num_new_pages
```
**EN:** Imports `logging`, `weakref`, `typing`, `psutil`, `torch`, `sglang.srt.layers.radix_attention` and other helpers used by the surrounding scope.
**CN:** 导入 `logging`, `weakref`, `typing`, `psutil`, `torch`, `sglang.srt.layers.radix_attention` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 23-27: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)

# sgl_kernel.kvcacheio is only available in CUDA/ROCm sgl-kernel builds (not XPU/MPS/NPU/CPU).
_is_cuda = is_cuda()
_is_hip = is_hip()
```
**EN:** Defines module-level variables such as `logger`, `_is_cuda`, `_is_hip`.
**CN:** 定义模块级变量，例如 `logger`, `_is_cuda`, `_is_hip`。

### Lines 28-36: Conditional logic / 条件逻辑
```python
if _is_cuda or _is_hip:
    from sgl_kernel.kvcacheio import transfer_kv_all_layer_mla
else:

    def transfer_kv_all_layer_mla(*args, **kwargs):
        raise RuntimeError(
            "HiSparse device KV transfer requires sgl_kernel.kvcacheio (CUDA/ROCm). "
            "It is not available on this backend."
        )
```
**EN:** Handles branching controlled by `_is_cuda or _is_hip`.
**CN:** 处理由 `_is_cuda or _is_hip` 控制的分支逻辑。

### Lines 39-39: HiSparseNSATokenToKVPool declaration / HiSparseNSATokenToKVPool 声明
```python
class HiSparseNSATokenToKVPool(NSATokenToKVPool):
```
**EN:** Declares the `HiSparseNSATokenToKVPool` class and connects it to `NSATokenToKVPool`.
**CN:** 声明 `HiSparseNSATokenToKVPool` 类，并将其关联到 `NSATokenToKVPool`。

### Lines 40-71: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        page_size: int,
        kv_lora_rank: int,
        dtype: torch.dtype,
        qk_rope_head_dim: int,
        layer_num: int,
        device: str,
        index_head_dim: int,
        enable_memory_saver: bool,
        kv_cache_dim: int,
# ... omitted for brevity ...
            end_layer=end_layer,
            index_buf_size=size * host_to_device_ratio,
        )
        self.bytes_per_token = self.kv_cache_dim * self.dtype.itemsize
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `HiSparseNSATokenToKVPool`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `HiSparseNSATokenToKVPool`。

### Lines 73-76: register_mapping implementation / register_mapping 实现
```python
    def register_mapping(self, full_to_hisparse_device_index_mapping: torch.Tensor):
        self.full_to_hisparse_device_index_mapping = (
            full_to_hisparse_device_index_mapping
        )
```
**EN:** Registers metadata so other components can discover this object. It belongs to `HiSparseNSATokenToKVPool`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `HiSparseNSATokenToKVPool`。

### Lines 78-81: translate_loc_to_hisparse_device implementation / translate_loc_to_hisparse_device 实现
```python
    def translate_loc_to_hisparse_device(self, compressed_indices: torch.Tensor):
        return self.full_to_hisparse_device_index_mapping[compressed_indices].to(
            torch.int32
        )
```
**EN:** Implements the translate LOC TO hisparse device routine for this scope. It belongs to `HiSparseNSATokenToKVPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc to hisparse device例程。 该方法属于 `HiSparseNSATokenToKVPool`。它会向调用方返回计算结果。

### Lines 83-84: _translate_loc_to_hisparse_device implementation / _translate_loc_to_hisparse_device 实现
```python
    def _translate_loc_to_hisparse_device(self, compressed_indices: torch.Tensor):
        return self.full_to_hisparse_device_index_mapping[compressed_indices]
```
**EN:** Implements the translate LOC TO hisparse device routine for this scope. It belongs to `HiSparseNSATokenToKVPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc to hisparse device例程。 该方法属于 `HiSparseNSATokenToKVPool`。它会向调用方返回计算结果。

### Lines 86-87: translate_loc_from_full_to_hisparse_device implementation / translate_loc_from_full_to_hisparse_device 实现
```python
    def translate_loc_from_full_to_hisparse_device(self, full_indices: torch.Tensor):
        return self._translate_loc_to_hisparse_device(full_indices)
```
**EN:** Implements the translate LOC from full TO hisparse device routine for this scope. It belongs to `HiSparseNSATokenToKVPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc from完整to hisparse device例程。 该方法属于 `HiSparseNSATokenToKVPool`。它会向调用方返回计算结果。

### Lines 89-90: translate_loc_from_full_to_compressed implementation / translate_loc_from_full_to_compressed 实现
```python
    def translate_loc_from_full_to_compressed(self, full_indices: torch.Tensor):
        return full_indices
```
**EN:** Implements the translate LOC from full TO compressed routine for this scope. It belongs to `HiSparseNSATokenToKVPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc from完整to compressed例程。 该方法属于 `HiSparseNSATokenToKVPool`。它会向调用方返回计算结果。

### Lines 92-100: set_kv_buffer implementation / set_kv_buffer 实现
```python
    def set_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k: torch.Tensor,
        cache_v: torch.Tensor,
    ):
        loc = self.translate_loc_to_hisparse_device(loc)
        super().set_kv_buffer(layer, loc, cache_k, cache_v)
```
**EN:** Updates internal state with the provided value. It belongs to `HiSparseNSATokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `HiSparseNSATokenToKVPool`。

### Lines 102-110: set_mla_kv_buffer implementation / set_mla_kv_buffer 实现
```python
    def set_mla_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        cache_k_nope: torch.Tensor,
        cache_k_rope: torch.Tensor,
    ):
        loc = self.translate_loc_to_hisparse_device(loc)
        super().set_mla_kv_buffer(layer, loc, cache_k_nope, cache_k_rope)
```
**EN:** Updates internal state with the provided value. It belongs to `HiSparseNSATokenToKVPool`.
**CN:** 使用给定值更新内部状态。 该方法属于 `HiSparseNSATokenToKVPool`。

### Lines 112-119: get_mla_kv_buffer implementation / get_mla_kv_buffer 实现
```python
    def get_mla_kv_buffer(
        self,
        layer: RadixAttention,
        loc: torch.Tensor,
        dst_dtype: Optional[torch.dtype] = None,
    ):
        loc = self.translate_loc_to_hisparse_device(loc)
        return super().get_mla_kv_buffer(layer, loc, dst_dtype)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HiSparseNSATokenToKVPool`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HiSparseNSATokenToKVPool`。它会向调用方返回计算结果。

### Lines 121-129: transfer_values_on_device implementation / transfer_values_on_device 实现
```python
    def transfer_values_on_device(self, dst_indices, src_indices):
        transfer_kv_all_layer_mla(
            src_layers=self.data_ptrs,
            dst_layers=self.data_ptrs,
            src_indices=src_indices,
            dst_indices=dst_indices,
            item_size=self.bytes_per_token,
            num_layers=self.layer_num,
        )
```
**EN:** Implements the transfer values ON device routine for this scope. It belongs to `HiSparseNSATokenToKVPool`.
**CN:** 实现当前作用域中的transfer values on device例程。 该方法属于 `HiSparseNSATokenToKVPool`。

### Lines 131-132: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices, mamba_indices=None):
        raise NotImplementedError("HiSparseDevicePool does not support get_cpu_copy")
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HiSparseNSATokenToKVPool`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HiSparseNSATokenToKVPool`。它会显式处理错误场景。

### Lines 134-135: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, kv_cache_cpu, indices, mamba_indices=None):
        raise NotImplementedError("HiSparseDevicePool does not support load_cpu_copy")
```
**EN:** Loads state from an external or serialized representation. It belongs to `HiSparseNSATokenToKVPool`. It validates error cases explicitly.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `HiSparseNSATokenToKVPool`。它会显式处理错误场景。

### Lines 138-138: HiSparseTokenToKVPoolAllocator declaration / HiSparseTokenToKVPoolAllocator 声明
```python
class HiSparseTokenToKVPoolAllocator(BaseTokenToKVPoolAllocator):
```
**EN:** Declares the `HiSparseTokenToKVPoolAllocator` class and connects it to `BaseTokenToKVPoolAllocator`.
**CN:** 声明 `HiSparseTokenToKVPoolAllocator` 类，并将其关联到 `BaseTokenToKVPoolAllocator`。

### Lines 139-192: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        page_size: int,
        dtype: torch.dtype,
        device: torch.device,
        kvcache: HiSparseNSATokenToKVPool,
        need_sort: bool,
        host_to_device_ratio: int = 2,
    ):
        self._kvcache = kvcache
        self._size_full = size * host_to_device_ratio
# ... omitted for brevity ...
        self.clear()
        self._kvcache.register_mapping(
            weakref.proxy(self.full_to_hisparse_device_index_mapping)
        )
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `HiSparseTokenToKVPoolAllocator`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `HiSparseTokenToKVPoolAllocator`。

### Lines 194-196: size_full implementation / size_full 实现
```python
    @property
    def size_full(self) -> int:
        return self._size_full
```
**EN:** Implements the size full routine for this scope. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的size完整例程。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 198-200: size implementation / size 实现
```python
    @property
    def size(self) -> int:
        return self._size_full
```
**EN:** Implements the size routine for this scope. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的size例程。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 202-206: available_size implementation / available_size 实现
```python
    def available_size(self) -> int:
        return min(
            self.logical_attn_allocator.available_size(),
            self.hisparse_attn_allocator.available_size(),
        )
```
**EN:** Implements the available size routine for this scope. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available size例程。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 208-209: get_kvcache implementation / get_kvcache 实现
```python
    def get_kvcache(self):
        return self._kvcache
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 211-215: alloc implementation / alloc 实现
```python
    def alloc(self, need_size: int):
        raise NotImplementedError(
            "HiSparse allocator does not support direct token allocation; "
            "use alloc_extend or alloc_decode instead."
        )
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `HiSparseTokenToKVPoolAllocator`. It validates error cases explicitly.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会显式处理错误场景。

### Lines 217-238: alloc_logical_only implementation / alloc_logical_only 实现
```python
    def alloc_logical_only(
        self,
        prefix_lens: torch.Tensor,
        prefix_lens_cpu: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
        last_loc: torch.Tensor,
        extend_num_tokens: int,
    ):
        """Allocate only logical indices without hisparse device indices.

        Used in the direct-to-host transfer path where KV data is written
# ... omitted for brevity ...
            seq_lens_cpu,
            last_loc,
            extend_num_tokens,
        )
```
**EN:** Allocate only logical indices without hisparse device indices. Allocates cache or memory resources for the requested workload. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 240-276: alloc_device_buffer implementation / alloc_device_buffer 实现
```python
    def alloc_device_buffer(self, allocated_indices, need_size: int):
        assert need_size % self.page_size == 0
        # clear original reference and isolate the buffer from outside addressing, allocate new buffer if needed
        hisparse_indices = self.full_to_hisparse_device_index_mapping[allocated_indices]
        self.full_to_hisparse_device_index_mapping[allocated_indices] = 0
        # Filter valid (non-zero) hisparse indices.
        # In the direct-to-host path, mapping is all zeros since no hisparse
        # device indices were pre-allocated.
        hisparse_indices = hisparse_indices[hisparse_indices > 0]
        if len(hisparse_indices) >= need_size:
            buffer_indices = hisparse_indices[:need_size]
            self.free_hisparse_indices(hisparse_indices[need_size:])
# ... omitted for brevity ...
                extra_indices is not None
            ), "Hisparse allocation failed in alloc_device_buffer"
            buffer_indices = torch.cat([hisparse_indices, extra_indices])
        return buffer_indices
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 278-281: free_hisparse_indices implementation / free_hisparse_indices 实现
```python
    def free_hisparse_indices(self, buffer_indices: torch.Tensor):
        # disable free group mechanism for device buffer free
        self.hisparse_attn_allocator.is_not_in_free_group = True
        self.hisparse_attn_allocator.free(buffer_indices[buffer_indices > 0])
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `HiSparseTokenToKVPoolAllocator`.
**CN:** 将已分配资源释放回资源池。 该方法属于 `HiSparseTokenToKVPoolAllocator`。

### Lines 283-284: get_last_loc_compressed implementation / get_last_loc_compressed 实现
```python
    def get_last_loc_compressed(self, last_locs: torch.Tensor):
        return last_locs
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 286-287: get_last_loc_hisparse_device implementation / get_last_loc_hisparse_device 实现
```python
    def get_last_loc_hisparse_device(self, last_locs: torch.Tensor):
        return self._kvcache._translate_loc_to_hisparse_device(last_locs)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 289-338: alloc_extend implementation / alloc_extend 实现
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
            hisparse_indices is not None
        ), "Hisparse allocation failed in alloc_extend"
        self.full_to_hisparse_device_index_mapping[logical_indices] = hisparse_indices
        return logical_indices
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 340-348: alloc_decode implementation / alloc_decode 实现
```python
    def alloc_decode(
        self,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
        last_loc: torch.Tensor,  # last_loc for full layers
    ):
        return self.logical_attn_allocator.alloc_decode(
            seq_lens, seq_lens_cpu, last_loc
        )
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 350-354: free_hisparse implementation / free_hisparse 实现
```python
    def free_hisparse(self, free_indices: torch.Tensor):
        hisparse_indices = self._kvcache._translate_loc_to_hisparse_device(free_indices)
        hisparse_indices = hisparse_indices[hisparse_indices > 0]
        self.free_hisparse_indices(hisparse_indices)
        self.full_to_hisparse_device_index_mapping[free_indices] = 0
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `HiSparseTokenToKVPoolAllocator`.
**CN:** 将已分配资源释放回资源池。 该方法属于 `HiSparseTokenToKVPoolAllocator`。

### Lines 356-362: clear implementation / clear 实现
```python
    def clear(self):
        self.logical_attn_allocator.clear()
        self.hisparse_attn_allocator.clear()
        # Note: the last item is -1, we don't clear it, see the comment in __init__
        self.full_to_hisparse_device_index_mapping[:-1].fill_(0)
        self.is_not_in_free_group = True
        self.free_group = []
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `HiSparseTokenToKVPoolAllocator`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `HiSparseTokenToKVPoolAllocator`。

### Lines 364-365: free_group_begin implementation / free_group_begin 实现
```python
    def free_group_begin(self):
        return
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 将已分配资源释放回资源池。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 367-368: free_group_end implementation / free_group_end 实现
```python
    def free_group_end(self):
        return
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 将已分配资源释放回资源池。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 370-385: free implementation / free 实现
```python
    def free(self, free_index: torch.Tensor):
        if free_index.numel() == 0:
            return
        if self.is_not_in_free_group:
            self.logical_attn_allocator.free(free_index)
            self.free_hisparse(free_index)
        else:
            self.free_group.append(free_index)
        assert (
            self.logical_attn_allocator.available_size()
            <= self.logical_attn_allocator.size
        )
        assert (
            self.hisparse_attn_allocator.available_size()
            <= self.hisparse_attn_allocator.size
        )
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 将已分配资源释放回资源池。 该方法属于 `HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 388-389: DeepSeekV4SingleKVPoolHost declaration / DeepSeekV4SingleKVPoolHost 声明
```python
class DeepSeekV4SingleKVPoolHost:

```
**EN:** Declares the `DeepSeekV4SingleKVPoolHost` class.
**CN:** 声明 `DeepSeekV4SingleKVPoolHost` 类。

### Lines 390-420: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        device_pool: HiSparseC4DevicePool,
        host_size: int,
        page_size: int,
        pin_memory: bool = True,
        device: str = "cpu",
    ):

        assert host_size > 0, "Host size must be specified and greater than 0"
        assert page_size == 1, "Host page size must be 1 for DeepSeekV4SingleKVPoolHost"

# ... omitted for brevity ...
            dtype=torch.uint64,
            device=self.device_pool.device,
        )
        self.clear()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `DeepSeekV4SingleKVPoolHost`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `DeepSeekV4SingleKVPoolHost`。

### Lines 422-425: clear implementation / clear 实现
```python
    def clear(self):
        self.free_slots = torch.arange(
            1, self.num_pages + 1, dtype=torch.int64, device="cpu"
        )
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `DeepSeekV4SingleKVPoolHost`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `DeepSeekV4SingleKVPoolHost`。

### Lines 427-457: init_kv_buffer implementation / init_kv_buffer 实现
```python
    def init_kv_buffer(self):
        dims = (self.layer_num, self.size + self.page_size, self.kv_cache_total_dim)
        requested_bytes = (
            self.layer_num
            * (self.size + self.page_size)
            * self.kv_cache_total_dim
            * self.dtype.itemsize
        )
        host_mem = psutil.virtual_memory()
        # preserve at least 10GB for other usage
        ten_gb = 10 * (1024**3)
        available_bytes = host_mem.available - ten_gb
# ... omitted for brevity ...
            torch.cuda.cudart().cudaHostRegister(
                host_pool.data_ptr(), host_pool.numel() * host_pool.element_size(), 0
            )
        return host_pool
```
**EN:** Initializes supporting state for later operations. It belongs to `DeepSeekV4SingleKVPoolHost`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 初始化后续操作所需的辅助状态。 该方法属于 `DeepSeekV4SingleKVPoolHost`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 459-484: backup_from_device_all_layer implementation / backup_from_device_all_layer 实现
```python
    def backup_from_device_all_layer(
        self, device_pool, host_indices, device_indices, io_backend="kernel"
    ):
        if io_backend != "kernel":
            raise ValueError(f"Unsupported IO backend: {io_backend}")

        from sglang.jit_kernel.deepseek_v4 import hisparse_offload_to_host

        if host_indices.device != device_indices.device:
            host_indices = host_indices.to(device=device_indices.device)
        host_indices_i64 = (
            host_indices.to(torch.int64)
# ... omitted for brevity ...
            cpu_ptrs=self.data_ptrs,
            gpu_indices=device_indices_i64,
            cpu_indices=host_indices_i64,
        )
```
**EN:** Implements the backup from device ALL layer routine for this scope. It belongs to `DeepSeekV4SingleKVPoolHost`. It validates error cases explicitly.
**CN:** 实现当前作用域中的backup from device all layer例程。 该方法属于 `DeepSeekV4SingleKVPoolHost`。它会显式处理错误场景。

### Lines 486-487: available_size implementation / available_size 实现
```python
    def available_size(self):
        return len(self.free_slots)
```
**EN:** Implements the available size routine for this scope. It belongs to `DeepSeekV4SingleKVPoolHost`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available size例程。 该方法属于 `DeepSeekV4SingleKVPoolHost`。它会向调用方返回计算结果。

### Lines 489-496: alloc implementation / alloc 实现
```python
    def alloc(self, need_size: int) -> Optional[torch.Tensor]:
        if need_size > self.available_size():
            return None

        select_index = self.free_slots[:need_size]
        self.free_slots = self.free_slots[need_size:]

        return select_index
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `DeepSeekV4SingleKVPoolHost`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `DeepSeekV4SingleKVPoolHost`。它会向调用方返回计算结果。

### Lines 498-500: free implementation / free 实现
```python
    def free(self, indices: torch.Tensor) -> int:
        self.free_slots = torch.cat([self.free_slots, indices.cpu()])
        return len(indices)
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `DeepSeekV4SingleKVPoolHost`. It returns a computed result to its caller.
**CN:** 将已分配资源释放回资源池。 该方法属于 `DeepSeekV4SingleKVPoolHost`。它会向调用方返回计算结果。

### Lines 503-504: DeepSeekV4HiSparseTokenToKVPoolAllocator declaration / DeepSeekV4HiSparseTokenToKVPoolAllocator 声明
```python
class DeepSeekV4HiSparseTokenToKVPoolAllocator(BaseTokenToKVPoolAllocator):

```
**EN:** Declares the `DeepSeekV4HiSparseTokenToKVPoolAllocator` class and connects it to `BaseTokenToKVPoolAllocator`.
**CN:** 声明 `DeepSeekV4HiSparseTokenToKVPoolAllocator` 类，并将其关联到 `BaseTokenToKVPoolAllocator`。

### Lines 505-554: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        logical_attn_allocator: BaseTokenToKVPoolAllocator,
    ):
        assert isinstance(logical_attn_allocator._kvcache, DeepSeekV4TokenToKVPool)
        assert isinstance(
            logical_attn_allocator._kvcache.c4_kv_pool, HiSparseC4DevicePool
        )
        self.compress_ratio = 4

        self.hisparse_kvcache = logical_attn_allocator._kvcache.c4_kv_pool
        self._size_full = logical_attn_allocator.size_full
# ... omitted for brevity ...

        self.hisparse_kvcache.register_mapping(
            weakref.proxy(self.full_to_hisparse_device_index_mapping)
        )
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。

### Lines 556-558: size_full implementation / size_full 实现
```python
    @property
    def size_full(self) -> int:
        return self._size_full
```
**EN:** Implements the size full routine for this scope. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的size完整例程。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 560-562: size implementation / size 实现
```python
    @property
    def size(self) -> int:
        return self.logical_attn_allocator.size
```
**EN:** Implements the size routine for this scope. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的size例程。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 564-566: size_swa implementation / size_swa 实现
```python
    @property
    def size_swa(self) -> int:
        return self.logical_attn_allocator.size_swa
```
**EN:** Implements the size SWA routine for this scope. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的size SWA例程。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 568-570: full_to_swa_index_mapping implementation / full_to_swa_index_mapping 实现
```python
    @property
    def full_to_swa_index_mapping(self):
        return self.logical_attn_allocator.full_to_swa_index_mapping
```
**EN:** Implements the full TO SWA index mapping routine for this scope. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的完整to SWA index mapping例程。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 572-578: debug_print implementation / debug_print 实现
```python
    def debug_print(self) -> str:
        msg = self.logical_attn_allocator.debug_print()
        msg += (
            f"#hisparse-available-size: "
            f"{self.hisparse_attn_allocator.available_size()}, "
        )
        return msg
```
**EN:** Implements the debug print routine for this scope. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的debug print例程。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 580-581: get_kvcache implementation / get_kvcache 实现
```python
    def get_kvcache(self):
        return self._kvcache
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 583-584: translate_loc_from_full_to_swa implementation / translate_loc_from_full_to_swa 实现
```python
    def translate_loc_from_full_to_swa(self, kv_indices: torch.Tensor):
        return self.logical_attn_allocator.translate_loc_from_full_to_swa(kv_indices)
```
**EN:** Implements the translate LOC from full TO SWA routine for this scope. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的translate loc from完整to SWA例程。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 586-590: full_available_size implementation / full_available_size 实现
```python
    def full_available_size(self):
        return min(
            self.logical_attn_allocator.full_available_size(),
            self.hisparse_attn_allocator.available_size() * self.compress_ratio,
        )
```
**EN:** Implements the full available size routine for this scope. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的完整available size例程。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 592-593: swa_available_size implementation / swa_available_size 实现
```python
    def swa_available_size(self):
        return self.logical_attn_allocator.swa_available_size()
```
**EN:** Implements the SWA available size routine for this scope. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的SWA available size例程。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 595-596: free_swa implementation / free_swa 实现
```python
    def free_swa(self, free_indices: torch.Tensor):
        self.logical_attn_allocator.free_swa(free_indices)
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`.
**CN:** 将已分配资源释放回资源池。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。

### Lines 598-602: available_size implementation / available_size 实现
```python
    def available_size(self) -> int:
        return min(
            self.logical_attn_allocator.available_size(),
            self.hisparse_attn_allocator.available_size() * self.compress_ratio,
        )
```
**EN:** Implements the available size routine for this scope. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available size例程。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 604-608: alloc implementation / alloc 实现
```python
    def alloc(self, need_size: int):
        raise NotImplementedError(
            "DeepSeek V4 HiSparse allocator does not support direct token allocation; "
            "use alloc_extend or alloc_decode instead."
        )
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It validates error cases explicitly.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会显式处理错误场景。

### Lines 610-656: alloc_device_buffer implementation / alloc_device_buffer 实现
```python
    def alloc_device_buffer(self, allocated_indices, need_size: int):
        assert need_size % self.page_size == 0
        hisparse_indices = self.full_to_hisparse_device_index_mapping[allocated_indices]
        self.full_to_hisparse_device_index_mapping[allocated_indices] = 0

        device_buffer_size = need_size - self.page_size
        P = len(hisparse_indices)
        if P > device_buffer_size + 1:
            newest_src = hisparse_indices[P - 1].clone()
            old_at_dbs = hisparse_indices[device_buffer_size].clone()
            hisparse_indices[device_buffer_size] = newest_src
            hisparse_indices[P - 1] = old_at_dbs
# ... omitted for brevity ...
                extra_indices is not None
            ), "Hisparse allocation failed in alloc_device_buffer"
            buffer_indices = torch.cat([hisparse_indices, extra_indices])
        return buffer_indices
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 658-660: free_hisparse_indices implementation / free_hisparse_indices 实现
```python
    def free_hisparse_indices(self, buffer_indices: torch.Tensor):
        self.hisparse_attn_allocator.is_not_in_free_group = True
        self.hisparse_attn_allocator.free(buffer_indices[buffer_indices > 0])
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`.
**CN:** 将已分配资源释放回资源池。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。

### Lines 662-663: get_last_loc_compressed implementation / get_last_loc_compressed 实现
```python
    def get_last_loc_compressed(self, last_locs: torch.Tensor):
        return (last_locs - 3) // self.compress_ratio
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 665-668: get_last_loc_hisparse_device implementation / get_last_loc_hisparse_device 实现
```python
    def get_last_loc_hisparse_device(self, last_locs: torch.Tensor):
        return self.hisparse_kvcache._translate_loc_to_hisparse_device(
            self.get_last_loc_compressed(last_locs)
        )
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 670-729: alloc_extend implementation / alloc_extend 实现
```python
    def alloc_extend(
        self,
        prefix_lens: torch.Tensor,
        prefix_lens_cpu: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
        last_loc: torch.Tensor,
        extend_num_tokens: int,
    ):
        assert self.page_size > 1

        num_new_pages_logical = get_num_new_pages(
# ... omitted for brevity ...
        self.full_to_hisparse_device_index_mapping[compressed_logical_indices] = (
            hisparse_indices.to(torch.int64)
        )
        return logical_indices
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 731-739: alloc_decode implementation / alloc_decode 实现
```python
    def alloc_decode(
        self,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
        last_loc: torch.Tensor,
    ):
        return self.logical_attn_allocator.alloc_decode(
            seq_lens, seq_lens_cpu, last_loc
        )
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 741-747: free_compressed implementation / free_compressed 实现
```python
    def free_compressed(self, compressed_indices: torch.Tensor):
        hisparse_indices = self.hisparse_kvcache.translate_loc_to_hisparse_device(
            compressed_indices
        )
        hisparse_indices = hisparse_indices[hisparse_indices > 0]
        self.free_hisparse_indices(hisparse_indices)
        self.full_to_hisparse_device_index_mapping[compressed_indices] = 0
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`.
**CN:** 将已分配资源释放回资源池。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。

### Lines 749-753: free_hisparse implementation / free_hisparse 实现
```python
    def free_hisparse(self, free_indices: torch.Tensor):
        compressed_indices = (
            self.hisparse_kvcache.translate_loc_from_full_to_compressed(free_indices)
        )
        self.free_compressed(compressed_indices)
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`.
**CN:** 将已分配资源释放回资源池。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。

### Lines 755-761: clear implementation / clear 实现
```python
    def clear(self):
        self.logical_attn_allocator.clear()
        self.hisparse_attn_allocator.clear()

        self.full_to_hisparse_device_index_mapping[:-1].fill_(0)
        self.is_not_in_free_group = True
        self.free_group = []
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。

### Lines 763-778: free implementation / free 实现
```python
    def free(self, free_index: torch.Tensor):
        if free_index.numel() == 0:
            return

        if self.is_not_in_free_group:
            self.logical_attn_allocator.free(free_index)
        else:
            self.free_group.append(free_index)
        assert (
            self.logical_attn_allocator.available_size()
            <= self.logical_attn_allocator.size
        )
        assert (
            self.hisparse_attn_allocator.available_size()
            <= self.hisparse_attn_allocator.size
        )
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `DeepSeekV4HiSparseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 将已分配资源释放回资源池。 该方法属于 `DeepSeekV4HiSparseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`HiSparseNSATokenToKVPool`**: Defines the `HiSparseNSATokenToKVPool` type and its core responsibilities. / 定义 `HiSparseNSATokenToKVPool` 类型及其核心职责。
- **`HiSparseTokenToKVPoolAllocator`**: Defines the `HiSparseTokenToKVPoolAllocator` type and its core responsibilities. / 定义 `HiSparseTokenToKVPoolAllocator` 类型及其核心职责。
- **`DeepSeekV4SingleKVPoolHost`**: Defines the `DeepSeekV4SingleKVPoolHost` type and its core responsibilities. / 定义 `DeepSeekV4SingleKVPoolHost` 类型及其核心职责。
- **`DeepSeekV4HiSparseTokenToKVPoolAllocator`**: Defines the `DeepSeekV4HiSparseTokenToKVPoolAllocator` type and its core responsibilities. / 定义 `DeepSeekV4HiSparseTokenToKVPoolAllocator` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `logging`, `weakref`, `typing`, `psutil`, `torch`, `sgl_kernel.kvcacheio`
- **Internal / 内部**: `sglang.srt.layers.radix_attention`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.deepseek_v4_memory_pool`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.utils`, `sglang.srt.utils.common`, `sglang.jit_kernel.deepseek_v4`
