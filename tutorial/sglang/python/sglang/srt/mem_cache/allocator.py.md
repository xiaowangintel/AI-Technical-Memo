# allocator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/allocator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the allocator logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的分配器相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: Imports and setup / 导入与初始化
```python
from __future__ import annotations

"""
Copyright 2025 SGLang Team
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
# ... omitted for brevity ...
import triton
import triton.language as tl

from sglang.srt.utils import get_bool_env_var, get_num_new_pages, next_power_of_2
```
**EN:** Imports `__future__`, `abc`, `typing`, `torch`, `triton`, `triton.language` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `abc`, `typing`, `torch`, `triton`, `triton.language` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 31-32: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.mem_cache.memory_pool import KVCache
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 35-35: BaseTokenToKVPoolAllocator declaration / BaseTokenToKVPoolAllocator 声明
```python
class BaseTokenToKVPoolAllocator(abc.ABC):
```
**EN:** Declares the `BaseTokenToKVPoolAllocator` class and connects it to `abc.ABC`.
**CN:** 声明 `BaseTokenToKVPoolAllocator` 类，并将其关联到 `abc.ABC`。

### Lines 36-56: __init__ implementation / __init__ 实现
```python
    @abc.abstractmethod
    def __init__(
        self,
        size: int,
        page_size: int,
        dtype: torch.dtype,
        device: str,
        kvcache: KVCache,
        need_sort: bool,
    ):
        self.size = size
        self.page_size = page_size
# ... omitted for brevity ...
        self.free_pages = None
        self.release_pages = None
        self.is_not_in_free_group = True
        self.free_group = []
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `BaseTokenToKVPoolAllocator`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `BaseTokenToKVPoolAllocator`。

### Lines 58-60: size_full implementation / size_full 实现
```python
    @property
    def size_full(self):
        return self.size
```
**EN:** Implements the size full routine for this scope. It belongs to `BaseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的size完整例程。 该方法属于 `BaseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 62-63: debug_print implementation / debug_print 实现
```python
    def debug_print(self) -> str:
        return ""
```
**EN:** Implements the debug print routine for this scope. It belongs to `BaseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的debug print例程。 该方法属于 `BaseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 65-66: available_size implementation / available_size 实现
```python
    def available_size(self):
        return (len(self.free_pages) + len(self.release_pages)) * self.page_size
```
**EN:** Implements the available size routine for this scope. It belongs to `BaseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available size例程。 该方法属于 `BaseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 68-69: get_kvcache implementation / get_kvcache 实现
```python
    def get_kvcache(self):
        return self._kvcache
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `BaseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `BaseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 71-72: restore_state implementation / restore_state 实现
```python
    def restore_state(self, state):
        self.free_pages, self.release_pages = state
```
**EN:** Implements the restore state routine for this scope. It belongs to `BaseTokenToKVPoolAllocator`.
**CN:** 实现当前作用域中的restore状态例程。 该方法属于 `BaseTokenToKVPoolAllocator`。

### Lines 74-75: backup_state implementation / backup_state 实现
```python
    def backup_state(self):
        return (self.free_pages, self.release_pages)
```
**EN:** Implements the backup state routine for this scope. It belongs to `BaseTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的backup状态例程。 该方法属于 `BaseTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 77-79: free_group_begin implementation / free_group_begin 实现
```python
    def free_group_begin(self):
        self.is_not_in_free_group = False
        self.free_group = []
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `BaseTokenToKVPoolAllocator`.
**CN:** 将已分配资源释放回资源池。 该方法属于 `BaseTokenToKVPoolAllocator`。

### Lines 81-84: free_group_end implementation / free_group_end 实现
```python
    def free_group_end(self):
        self.is_not_in_free_group = True
        if self.free_group:
            self.free(torch.cat(self.free_group))
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `BaseTokenToKVPoolAllocator`.
**CN:** 将已分配资源释放回资源池。 该方法属于 `BaseTokenToKVPoolAllocator`。

### Lines 86-92: merge_and_sort_free implementation / merge_and_sort_free 实现
```python
    def merge_and_sort_free(self):
        if len(self.release_pages) > 0:
            self.free_pages = torch.cat((self.free_pages, self.release_pages))
            self.free_pages, _ = torch.sort(self.free_pages)
            self.release_pages = torch.empty(
                (0,), dtype=self.release_pages.dtype, device=self.device
            )
```
**EN:** Implements the merge AND sort free routine for this scope. It belongs to `BaseTokenToKVPoolAllocator`.
**CN:** 实现当前作用域中的merge and sort free例程。 该方法属于 `BaseTokenToKVPoolAllocator`。

### Lines 94-96: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices, mamba_indices=None):
        # FIXME: reuse the get_cpu_copy after paged allocator is implemented
        raise NotImplementedError()
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `BaseTokenToKVPoolAllocator`. It validates error cases explicitly.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `BaseTokenToKVPoolAllocator`。它会显式处理错误场景。

### Lines 98-100: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, kv_cache_cpu, indices, mamba_indices=None):
        # FIXME: reuse the load_cpu_copy after paged allocator is implemented
        raise NotImplementedError()
```
**EN:** Loads state from an external or serialized representation. It belongs to `BaseTokenToKVPoolAllocator`. It validates error cases explicitly.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `BaseTokenToKVPoolAllocator`。它会显式处理错误场景。

### Lines 102-103: alloc_extend implementation / alloc_extend 实现
```python
    def alloc_extend(self, *args, **kwargs):
        raise NotImplementedError("alloc_extend is only for paged allocator")
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `BaseTokenToKVPoolAllocator`. It validates error cases explicitly.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `BaseTokenToKVPoolAllocator`。它会显式处理错误场景。

### Lines 105-106: alloc_decode implementation / alloc_decode 实现
```python
    def alloc_decode(self, *args, **kwargs):
        raise NotImplementedError("alloc_decode is only for paged allocator")
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `BaseTokenToKVPoolAllocator`. It validates error cases explicitly.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `BaseTokenToKVPoolAllocator`。它会显式处理错误场景。

### Lines 108-110: clear implementation / clear 实现
```python
    @abc.abstractmethod
    def clear(self):
        raise NotImplementedError()
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `BaseTokenToKVPoolAllocator`. It validates error cases explicitly.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `BaseTokenToKVPoolAllocator`。它会显式处理错误场景。

### Lines 112-114: alloc implementation / alloc 实现
```python
    @abc.abstractmethod
    def alloc(self, need_size: int):
        raise NotImplementedError()
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `BaseTokenToKVPoolAllocator`. It validates error cases explicitly.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `BaseTokenToKVPoolAllocator`。它会显式处理错误场景。

### Lines 116-118: free implementation / free 实现
```python
    @abc.abstractmethod
    def free(self, free_index: torch.Tensor):
        raise NotImplementedError()
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `BaseTokenToKVPoolAllocator`. It validates error cases explicitly.
**CN:** 将已分配资源释放回资源池。 该方法属于 `BaseTokenToKVPoolAllocator`。它会显式处理错误场景。

### Lines 121-123: TokenToKVPoolAllocator declaration / TokenToKVPoolAllocator 声明
```python
class TokenToKVPoolAllocator(BaseTokenToKVPoolAllocator):
    """An allocator managing the indices to kv cache data."""

```
**EN:** An allocator managing the indices to kv cache data. Declares the `TokenToKVPoolAllocator` class and connects it to `BaseTokenToKVPoolAllocator`.
**CN:** 声明 `TokenToKVPoolAllocator` 类，并将其关联到 `BaseTokenToKVPoolAllocator`。

### Lines 124-133: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        dtype: torch.dtype,
        device: str,
        kvcache: KVCache,
        need_sort: bool,
    ):
        super().__init__(size, 1, dtype, device, kvcache, need_sort)
        self.clear()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `TokenToKVPoolAllocator`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `TokenToKVPoolAllocator`。

### Lines 135-142: clear implementation / clear 实现
```python
    def clear(self):
        # The padded slot 0 is used for writing dummy outputs from padded tokens.
        self.free_pages = torch.arange(
            1, self.size + 1, dtype=torch.int64, device=self.device
        )
        self.is_not_in_free_group = True
        self.free_group = []
        self.release_pages = torch.empty((0,), dtype=torch.int64, device=self.device)
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `TokenToKVPoolAllocator`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `TokenToKVPoolAllocator`。

### Lines 144-146: available_size implementation / available_size 实现
```python
    def available_size(self):
        # To avoid minor "len(free_pages) * 1" overhead
        return len(self.free_pages) + len(self.release_pages)
```
**EN:** Implements the available size routine for this scope. It belongs to `TokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的available size例程。 该方法属于 `TokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 148-157: alloc implementation / alloc 实现
```python
    def alloc(self, need_size: int):
        if self.need_sort and need_size > len(self.free_pages):
            self.merge_and_sort_free()

        if need_size > len(self.free_pages):
            return None

        select_index = self.free_pages[:need_size]
        self.free_pages = self.free_pages[need_size:]
        return select_index
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `TokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `TokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 159-169: free implementation / free 实现
```python
    def free(self, free_index: torch.Tensor):
        if free_index.numel() == 0:
            return

        if self.is_not_in_free_group:
            if self.need_sort:
                self.release_pages = torch.cat((self.release_pages, free_index))
            else:
                self.free_pages = torch.cat((self.free_pages, free_index))
        else:
            self.free_group.append(free_index)
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `TokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 将已分配资源释放回资源池。 该方法属于 `TokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 171-172: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices, mamba_indices=None):
        return self._kvcache.get_cpu_copy(indices, mamba_indices=mamba_indices)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `TokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `TokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 174-177: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, kv_cache_cpu, indices, mamba_indices=None):
        return self._kvcache.load_cpu_copy(
            kv_cache_cpu, indices, mamba_indices=mamba_indices
        )
```
**EN:** Loads state from an external or serialized representation. It belongs to `TokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `TokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 180-237: alloc_extend_naive implementation / alloc_extend_naive 实现
```python
def alloc_extend_naive(
    prefix_lens,
    seq_lens,
    last_loc,
    free_pages,
    out_indices,
    page_size,
    device,
):
    extend_lens = seq_lens - prefix_lens
    end_pos = torch.cumsum(extend_lens, 0)
    start_pos = end_pos - extend_lens
# ... omitted for brevity ...
        if num3:
            out_indices[end_pos[i] - num3 : end_pos[i]] = (
                free_pages[end_new_pages[i] - 1] * page_size + pos_in_page[:num3]
            ).view(-1)
```
**EN:** Allocates cache or memory resources for the requested workload. The implementation iterates over inputs or managed entries.
**CN:** 为请求的工作负载分配缓存或内存资源。实现过程中会遍历输入或受管条目。

### Lines 240-323: alloc_extend_kernel implementation / alloc_extend_kernel 实现
```python
@triton.jit
def alloc_extend_kernel(
    pre_lens_ptr,
    seq_lens_ptr,
    last_loc_ptr,
    free_page_ptr,
    out_indices,
    bs_upper: tl.constexpr,
    page_size: tl.constexpr,
):
    pid = tl.program_id(0)

# ... omitted for brevity ...
        out_indices + output_start_loc + num_part1 + num_part2 + offset_one_page,
        start_loc * page_size + offset_one_page,
        mask=offset_one_page < num_part3,
    )
```
**EN:** Allocates cache or memory resources for the requested workload. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 326-359: alloc_decode_kernel implementation / alloc_decode_kernel 实现
```python
@triton.jit
def alloc_decode_kernel(
    seq_lens_ptr,
    last_loc_ptr,
    free_page_ptr,
    out_indices,
    bs_upper: tl.constexpr,
    page_size: tl.constexpr,
):
    pid = tl.program_id(0)

    load_offset = tl.arange(0, bs_upper)
# ... omitted for brevity ...
        tl.store(out_indices + pid, last_loc + 1)
    else:
        page = tl.load(free_page_ptr + new_page_start_loc)
        tl.store(out_indices + pid, page * page_size)
```
**EN:** Allocates cache or memory resources for the requested workload.
**CN:** 为请求的工作负载分配缓存或内存资源。

### Lines 362-371: PagedTokenToKVPoolAllocator declaration / PagedTokenToKVPoolAllocator 声明
```python
class PagedTokenToKVPoolAllocator(BaseTokenToKVPoolAllocator):
    """
    An allocator managing the indices to kv cache data.

    This class has the same interface as `TokenToKVPoolAllocator` but the output
    of one request is always page-aligned.

    TODO: fuse last_loc into the kernel.
    """

```
**EN:** An allocator managing the indices to kv cache data. Declares the `PagedTokenToKVPoolAllocator` class and connects it to `BaseTokenToKVPoolAllocator`.
**CN:** 声明 `PagedTokenToKVPoolAllocator` 类，并将其关联到 `BaseTokenToKVPoolAllocator`。

### Lines 372-384: __init__ implementation / __init__ 实现
```python
    def __init__(
        self,
        size: int,
        page_size: int,
        dtype: torch.dtype,
        device: str,
        kvcache: KVCache,
        need_sort: bool,
    ):
        super().__init__(size, page_size, dtype, device, kvcache, need_sort)
        self.num_pages = size // page_size
        self.debug_mode = get_bool_env_var("SGLANG_DEBUG_MEMORY_POOL")
        self.clear()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `PagedTokenToKVPoolAllocator`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `PagedTokenToKVPoolAllocator`。

### Lines 386-407: alloc implementation / alloc 实现
```python
    def alloc(self, need_size: int):
        # page-aligned allocation, returning contiguous indices of pages
        if self.debug_mode:
            assert (
                need_size % self.page_size == 0
            ), "The allocation size should be page-aligned"

        num_pages = need_size // self.page_size
        if self.need_sort and num_pages > len(self.free_pages):
            self.merge_and_sort_free()
        if num_pages > len(self.free_pages):
            return None
# ... omitted for brevity ...
            + torch.arange(self.page_size, device=self.device)
        ).reshape(-1)

        return out_indices
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `PagedTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `PagedTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 409-457: alloc_extend implementation / alloc_extend 实现
```python
    def alloc_extend(
        self,
        prefix_lens: torch.Tensor,
        prefix_lens_cpu: torch.Tensor,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
        last_loc: torch.Tensor,
        extend_num_tokens: int,
        num_new_pages: int = None,
    ):
        if self.debug_mode:
            assert torch.all(
# ... omitted for brevity ...
            return None

        self.free_pages = self.free_pages[num_new_pages:]
        return out_indices
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `PagedTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `PagedTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 459-496: alloc_decode implementation / alloc_decode 实现
```python
    def alloc_decode(
        self,
        seq_lens: torch.Tensor,
        seq_lens_cpu: torch.Tensor,
        last_loc: torch.Tensor,
    ):
        if self.debug_mode:
            assert torch.all(
                (last_loc + 2) % self.page_size == seq_lens % self.page_size
            )

        bs = len(seq_lens)
# ... omitted for brevity ...
            return None

        self.free_pages = self.free_pages[num_new_pages:]
        return out_indices
```
**EN:** Allocates cache or memory resources for the requested workload. It belongs to `PagedTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 为请求的工作负载分配缓存或内存资源。 该方法属于 `PagedTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 498-512: free implementation / free 实现
```python
    def free(self, free_index: torch.Tensor):
        if free_index.numel() == 0:
            return

        if self.is_not_in_free_group:
            free_page_indices = torch.unique(free_index // self.page_size)
            if self.need_sort:
                self.release_pages = torch.cat((free_page_indices, self.release_pages))
            else:
                self.free_pages = torch.cat((free_page_indices, self.free_pages))
        else:
            self.free_group.append(free_index)

        if self.debug_mode:
            assert len(torch.unique(self.free_pages)) == len(self.free_pages)
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `PagedTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 将已分配资源释放回资源池。 该方法属于 `PagedTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 514-521: clear implementation / clear 实现
```python
    def clear(self):
        # The padded slot 0 is used for writing dummy outputs from padded tokens.
        self.free_pages = torch.arange(
            1, self.num_pages + 1, dtype=torch.int64, device=self.device
        )
        self.is_not_in_free_group = True
        self.free_group = []
        self.release_pages = torch.empty((0,), dtype=torch.int64, device=self.device)
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `PagedTokenToKVPoolAllocator`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `PagedTokenToKVPoolAllocator`。

### Lines 523-524: get_cpu_copy implementation / get_cpu_copy 实现
```python
    def get_cpu_copy(self, indices, mamba_indices=None):
        return self._kvcache.get_cpu_copy(indices, mamba_indices=mamba_indices)
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `PagedTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `PagedTokenToKVPoolAllocator`。它会向调用方返回计算结果。

### Lines 526-529: load_cpu_copy implementation / load_cpu_copy 实现
```python
    def load_cpu_copy(self, kv_cache_cpu, indices, mamba_indices=None):
        return self._kvcache.load_cpu_copy(
            kv_cache_cpu, indices, mamba_indices=mamba_indices
        )
```
**EN:** Loads state from an external or serialized representation. It belongs to `PagedTokenToKVPoolAllocator`. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `PagedTokenToKVPoolAllocator`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`BaseTokenToKVPoolAllocator`**: Defines the `BaseTokenToKVPoolAllocator` type and its core responsibilities. / 定义 `BaseTokenToKVPoolAllocator` 类型及其核心职责。
- **`TokenToKVPoolAllocator`**: Defines the `TokenToKVPoolAllocator` type and its core responsibilities. / 定义 `TokenToKVPoolAllocator` 类型及其核心职责。
- **`alloc_extend_naive`**: Provides the `alloc_extend_naive` entry point for module-level behavior. / 提供模块级行为的 `alloc_extend_naive` 入口。
- **`alloc_extend_kernel`**: Provides the `alloc_extend_kernel` entry point for module-level behavior. / 提供模块级行为的 `alloc_extend_kernel` 入口。
- **`alloc_decode_kernel`**: Provides the `alloc_decode_kernel` entry point for module-level behavior. / 提供模块级行为的 `alloc_decode_kernel` 入口。
- **`PagedTokenToKVPoolAllocator`**: Defines the `PagedTokenToKVPoolAllocator` type and its core responsibilities. / 定义 `PagedTokenToKVPoolAllocator` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `abc`, `typing`, `torch`, `triton`, `triton.language`
- **Internal / 内部**: `sglang.srt.utils`, `sglang.srt.mem_cache.memory_pool`
