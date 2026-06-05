# allocator_npu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/allocator_npu.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for memory allocation inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的内存分配支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Module setup and shared state / 模块设置与共享状态
```python
from typing import TYPE_CHECKING

import torch

from sglang.srt.mem_cache.allocator import (
    PagedTokenToKVPoolAllocator,
    alloc_extend_naive,
)
from sglang.srt.utils import get_num_new_pages, next_power_of_2

if TYPE_CHECKING:
    from sglang.srt.mem_cache.memory_pool import KVCache
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `torch`, `sglang.srt.mem_cache.allocator`, `sglang.srt.utils`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `torch`, `sglang.srt.mem_cache.allocator`, `sglang.srt.utils`。

### Lines 15-15: Class `NPUPagedTokenToKVPoolAllocator` declaration / 类 `NPUPagedTokenToKVPoolAllocator` 声明
```python
class NPUPagedTokenToKVPoolAllocator(PagedTokenToKVPoolAllocator):
```
**EN:** This class establishes `NPUPagedTokenToKVPoolAllocator` as the main container/coordinator for the surrounding logic. It inherits from `PagedTokenToKVPoolAllocator`. Its core interface includes methods such as `__init__`, `alloc_extend`, `alloc_decode`, `free`.
**CN:** 该类将 `NPUPagedTokenToKVPoolAllocator` 定义为周边逻辑的主要封装体或协调者。 它继承自 `PagedTokenToKVPoolAllocator`。 其核心接口包括 `__init__`, `alloc_extend`, `alloc_decode`, `free` 等方法。

### Lines 16-26: Method `NPUPagedTokenToKVPoolAllocator.__init__` / 方法 `NPUPagedTokenToKVPoolAllocator.__init__`
```python
    def __init__(
        self,
        size: int,
        page_size: int,
        dtype: torch.dtype,
        device: str,
        kvcache: "KVCache",
        need_sort: bool,
    ):
        super().__init__(size, page_size, dtype, device, kvcache, need_sort)
        self.roundup = page_size - 1
```
**EN:** This method implements `__init__` on `NPUPagedTokenToKVPoolAllocator`. It primarily calls `super.__init__`, `super` to complete its work. State updates are written into `self.roundup`.
**CN:** 该方法（属于 `NPUPagedTokenToKVPoolAllocator`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `super` 来完成任务。 状态更新主要写入 `self.roundup`。

### Lines 28-98: Method `NPUPagedTokenToKVPoolAllocator.alloc_extend` / 方法 `NPUPagedTokenToKVPoolAllocator.alloc_extend`
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
                (last_loc + 1) % self.page_size == prefix_lens % self.page_size
            )

        if num_new_pages is None:
            num_new_pages_tensor = (
                (seq_lens + self.roundup) // self.page_size
                - (prefix_lens + self.roundup) // self.page_size
            ).sum()
            num_new_pages_item = num_new_pages_tensor.item()
        else:
            num_new_pages_item = num_new_pages
        if self.need_sort and num_new_pages_item > len(self.free_pages):
            self.merge_and_sort_free()

        if num_new_pages_item > len(self.free_pages):
            return None
# ... omitted for brevity ...

        if self.debug_mode:
            assert len(torch.unique(out_indices)) == len(out_indices)

        self.free_pages = self.free_pages[num_new_pages_item:]
        return out_indices.int()
```
**EN:** This method implements `alloc_extend` on `NPUPagedTokenToKVPoolAllocator`. It primarily calls `out_indices.int`, `torch.all`, `sum`, `num_new_pages_tensor.item`, `self.merge_and_sort_free`, `len` to complete its work. State updates are written into `self.free_pages`, `num_new_pages_tensor`, `num_new_pages_item`, `out_indices`, `max_num_extend_tokens`, `bs`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUPagedTokenToKVPoolAllocator`）实现了 `alloc_extend`。 它主要通过调用 `out_indices.int`, `torch.all`, `sum`, `num_new_pages_tensor.item`, `self.merge_and_sort_free`, `len` 来完成任务。 状态更新主要写入 `self.free_pages`, `num_new_pages_tensor`, `num_new_pages_item`, `out_indices`, `max_num_extend_tokens`, `bs`。 实现中使用了条件分支。

### Lines 100-137: Method `NPUPagedTokenToKVPoolAllocator.alloc_decode` / 方法 `NPUPagedTokenToKVPoolAllocator.alloc_decode`
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

        num_new_pages = get_num_new_pages(
            seq_lens=seq_lens_cpu,
            page_size=self.page_size,
            decode=True,
        )

        if num_new_pages > len(self.free_pages):
            self.merge_and_sort_free()

        if num_new_pages > len(self.free_pages):
            return None

        need_new_pages = (seq_lens % self.page_size == 1).int()
        end_new_pages = torch.cumsum(need_new_pages, 0)
        start_new_pages = end_new_pages - need_new_pages
        if num_new_pages == 0:
            out_indices = last_loc + 1
# ... omitted for brevity ...

        if self.debug_mode:
            assert len(torch.unique(out_indices)) == len(out_indices)

        self.free_pages = self.free_pages[num_new_pages:]
        return out_indices.int()
```
**EN:** This method implements `alloc_decode` on `NPUPagedTokenToKVPoolAllocator`. It primarily calls `get_num_new_pages`, `int`, `torch.cumsum`, `out_indices.int`, `torch.all`, `len` to complete its work. State updates are written into `num_new_pages`, `need_new_pages`, `end_new_pages`, `start_new_pages`, `self.free_pages`, `out_indices`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUPagedTokenToKVPoolAllocator`）实现了 `alloc_decode`。 它主要通过调用 `get_num_new_pages`, `int`, `torch.cumsum`, `out_indices.int`, `torch.all`, `len` 来完成任务。 状态更新主要写入 `num_new_pages`, `need_new_pages`, `end_new_pages`, `start_new_pages`, `self.free_pages`, `out_indices`。 实现中使用了条件分支。

### Lines 139-155: Method `NPUPagedTokenToKVPoolAllocator.free` / 方法 `NPUPagedTokenToKVPoolAllocator.free`
```python
    def free(self, free_index: torch.Tensor):
        if free_index.numel() == 0:
            return

        if self.is_not_in_free_group:
            device = free_index.device
            free_page_indices = torch.unique(free_index.cpu() // self.page_size)
            free_page_indices = free_page_indices.to(device)
            if self.need_sort:
                self.release_pages = torch.cat((free_page_indices, self.release_pages))
            else:
                self.free_pages = torch.cat((free_page_indices, self.free_pages))
        else:
            self.free_group.append(free_index)

        if self.debug_mode:
            assert len(torch.unique(self.free_pages)) == len(self.free_pages)
```
**EN:** This method implements `free` on `NPUPagedTokenToKVPoolAllocator`. It primarily calls `free_index.numel`, `torch.unique`, `free_page_indices.to`, `self.free_group.append`, `torch.cat`, `len` to complete its work. State updates are written into `device`, `free_page_indices`, `self.release_pages`, `self.free_pages`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `NPUPagedTokenToKVPoolAllocator`）实现了 `free`。 它主要通过调用 `free_index.numel`, `torch.unique`, `free_page_indices.to`, `self.free_group.append`, `torch.cat`, `len` 来完成任务。 状态更新主要写入 `device`, `free_page_indices`, `self.release_pages`, `self.free_pages`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `NPUPagedTokenToKVPoolAllocator`
- **Functions / 函数**: `__init__`, `alloc_extend`, `alloc_decode`, `free`
- **Themes / 主题**: `allocator`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.mem_cache.allocator`, `sglang.srt.utils`, `sglang.srt.mem_cache.memory_pool`
- **External / 外部依赖**: `torch`, `sgl_kernel_npu.mem_cache.allocator`
- **Standard library / 标准库**: `typing`
