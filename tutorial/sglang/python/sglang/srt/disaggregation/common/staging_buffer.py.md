# staging_buffer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/common/staging_buffer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements staging-buffer abstractions used while moving KV data and metadata between devices or processes. It focuses on temporary storage layout, allocation, and copy behavior. / 该文件实现了在设备或进程之间搬运 KV 数据与元数据时使用的暂存缓冲区抽象，重点关注临时存储布局、分配与拷贝行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: Module-level constants and helper logic
```python
"""
GPU Staging Buffer for heterogeneous TP KV cache transfer.

When prefill attn_tp_size != decode attn_tp_size, the per-token RDMA approach
generates O(tokens * layers) small RDMA requests. This module provides a staging
buffer mechanism that gathers scattered head slices into contiguous GPU memory,
enabling bulk RDMA transfers that reduce request count to O(layers) or O(1).

Usage:
    Activated by setting SGLANG_DISAGG_STAGING_BUFFER=1.
"""

from __future__ import annotations

import logging
import os
import threading
from typing import List, Optional, Tuple

import torch
import triton
import triton.language as tl

logger = logging.getLogger(__name__)

# TODO(yangminl): remove torch fallback implementations once the Triton kernels
# have been validated in production across all configurations.
_USE_TRITON_STAGING = not bool(os.environ.get("SGLANG_STAGING_USE_TORCH", ""))


```
**EN:** This block contains module-level constants, helpers, or documentation for staging buffer allocation and movement. It prepares shared state that later classes and functions build on. Notable operations include `O`, `getLogger`, `TODO`, `get`.
**CN:** 这一段包含与暂存缓冲区分配与移动相关的模块级常量、辅助逻辑或说明文本，为后续类和函数提供共享基础。 值得注意的操作包括 `O`、`getLogger`、`TODO`、`get`。

### Lines 31-33: Function `_fused_gather_to_staging_kernel` signature and setup
```python
@triton.jit
def _fused_gather_to_staging_kernel(
    layer_ptrs,
```
**EN:** This block defines the function `_fused_gather_to_staging_kernel`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `_fused_gather_to_staging_kernel`.
**CN:** 这一段定义了function `_fused_gather_to_staging_kernel`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `_fused_gather_to_staging_kernel`。

### Lines 34-58: Function `_fused_gather_to_staging_kernel` logic (part 1)
```python
    page_indices,
    staging,
    num_tokens,
    stride_pool_token,
    head_offset,
    per_layer_elems,
    ELEMS_PER_TOKEN: tl.constexpr,
    PAGE_SIZE: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    layer_id = tl.program_id(0)
    block_id = tl.program_id(1)

    layer_ptr = tl.load(layer_ptrs + layer_id).to(staging.dtype)

    offsets = block_id * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = offsets < per_layer_elems

    t_idx = offsets // ELEMS_PER_TOKEN
    e_idx = offsets % ELEMS_PER_TOKEN

    page_id = t_idx // PAGE_SIZE
    intra_page = t_idx % PAGE_SIZE
    page_val = tl.load(page_indices + page_id, mask=mask, other=0)
    pool_token = page_val * PAGE_SIZE + intra_page
```
**EN:** This block continues `_fused_gather_to_staging_kernel` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `program_id`, `load`, `to`, `arange`.
**CN:** 这一段延续了 `_fused_gather_to_staging_kernel` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `program_id`、`load`、`to`、`arange`。

### Lines 59-66: Function `_fused_gather_to_staging_kernel` logic (part 2)
```python

    src_offsets = (
        pool_token * stride_pool_token.to(tl.int64) + head_offset.to(tl.int64) + e_idx
    )
    vals = tl.load(layer_ptr + src_offsets, mask=mask)

    dst_offsets = tl.program_id(0).to(tl.int64) * per_layer_elems.to(tl.int64) + offsets
    tl.store(staging + dst_offsets, vals, mask=mask)
```
**EN:** This block continues `_fused_gather_to_staging_kernel` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `to`, `load`, `program_id`, `store`.
**CN:** 这一段延续了 `_fused_gather_to_staging_kernel` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `to`、`load`、`program_id`、`store`。

### Lines 69-71: Function `_fused_scatter_from_staging_kernel` signature and setup
```python
@triton.jit
def _fused_scatter_from_staging_kernel(
    layer_ptrs,
```
**EN:** This block defines the function `_fused_scatter_from_staging_kernel`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `_fused_scatter_from_staging_kernel`.
**CN:** 这一段定义了function `_fused_scatter_from_staging_kernel`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `_fused_scatter_from_staging_kernel`。

### Lines 72-93: Function `_fused_scatter_from_staging_kernel` logic (part 1)
```python
    page_indices,
    staging,
    writer_head_offsets,
    num_tokens,
    stride_pool_token,
    per_layer_elems,
    ELEMS_PER_TOKEN: tl.constexpr,
    PAGE_SIZE: tl.constexpr,
    NUM_LAYERS_X2: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    prog_id = tl.program_id(0)
    block_id = tl.program_id(1)

    writer_id = prog_id // NUM_LAYERS_X2
    layer_kv_id = prog_id % NUM_LAYERS_X2

    layer_ptr = tl.load(layer_ptrs + layer_kv_id).to(staging.dtype)
    head_offset = tl.load(writer_head_offsets + writer_id)

    offsets = block_id * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    mask = offsets < per_layer_elems
```
**EN:** This block continues `_fused_scatter_from_staging_kernel` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `program_id`, `load`, `to`, `arange`.
**CN:** 这一段延续了 `_fused_scatter_from_staging_kernel` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `program_id`、`load`、`to`、`arange`。

### Lines 94-114: Function `_fused_scatter_from_staging_kernel` logic (part 2)
```python

    t_idx = offsets // ELEMS_PER_TOKEN
    e_idx = offsets % ELEMS_PER_TOKEN

    page_id = t_idx // PAGE_SIZE
    intra_page = t_idx % PAGE_SIZE
    page_val = tl.load(page_indices + page_id, mask=mask, other=0)
    pool_token = page_val * PAGE_SIZE + intra_page

    per_rank_elems = per_layer_elems.to(tl.int64) * NUM_LAYERS_X2
    src_offsets = (
        writer_id.to(tl.int64) * per_rank_elems
        + layer_kv_id.to(tl.int64) * per_layer_elems.to(tl.int64)
        + offsets
    )
    vals = tl.load(staging + src_offsets, mask=mask)

    dst_offsets = (
        pool_token * stride_pool_token.to(tl.int64) + head_offset.to(tl.int64) + e_idx
    )
    tl.store(layer_ptr + dst_offsets, vals, mask=mask)
```
**EN:** This block continues `_fused_scatter_from_staging_kernel` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `load`, `to`, `store`.
**CN:** 这一段延续了 `_fused_scatter_from_staging_kernel` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `load`、`to`、`store`。

### Lines 117-117: Class `StagingBuffer` declaration
```python
class StagingBuffer:
```
**EN:** This block declares the class `StagingBuffer` and establishes its responsibility inside staging buffer allocation and movement. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `StagingBuffer`.
**CN:** 这一段声明了类 `StagingBuffer`，并说明它在暂存缓冲区分配与移动中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `StagingBuffer`。

### Lines 118-124: Supporting state inside `StagingBuffer`
```python
    """Pre-allocated GPU staging buffer for bulk KV transfer.

    When a custom_mem_pool is provided (e.g., mooncake NVLink allocator),
    the buffer is allocated within that pool so it's compatible with
    NVLink/MNNVL transport (requires cuMemCreate-backed memory).
    """

```
**EN:** This block adds supporting state or helper logic inside `StagingBuffer`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `provided`, `transport`.
**CN:** 这一段为 `StagingBuffer` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `provided`、`transport`。

### Lines 125-127: Method `__init__` signature and setup
```python
    def __init__(
        self,
        size_bytes: int,
```
**EN:** This block defines the method `__init__` on `StagingBuffer`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `StagingBuffer`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 128-149: Method `__init__` logic (part 1)
```python
        device: str,
        gpu_id: int,
        custom_mem_pool=None,
    ):
        self.size_bytes = size_bytes
        self.device = device
        self.gpu_id = gpu_id

        torch.cuda.set_device(gpu_id)
        if custom_mem_pool is not None:
            with torch.cuda.use_mem_pool(custom_mem_pool):
                self.buffer = torch.empty(size_bytes, dtype=torch.uint8, device=device)
            alloc_method = "custom_mem_pool (cuMemCreate)"
        else:
            self.buffer = torch.empty(size_bytes, dtype=torch.uint8, device=device)
            alloc_method = "cudaMalloc"
        self.data_ptr = self.buffer.data_ptr()

        logger.info(
            f"StagingBuffer allocated: {size_bytes / (1024*1024):.1f} MB "
            f"on {device}, method={alloc_method}, ptr=0x{self.data_ptr:x}"
        )
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `set_device`, `use_mem_pool`, `empty`, `custom_mem_pool`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `set_device`、`use_mem_pool`、`empty`、`custom_mem_pool`。

### Lines 151-152: Method `get_ptr`
```python
    def get_ptr(self) -> int:
        return self.data_ptr
```
**EN:** This block defines the method `get_ptr` on `StagingBuffer`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `get_ptr`.
**CN:** 这一段定义了method `get_ptr`（属于 `StagingBuffer`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `get_ptr`。

### Lines 154-155: Method `get_size`
```python
    def get_size(self) -> int:
        return self.size_bytes
```
**EN:** This block defines the method `get_size` on `StagingBuffer`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `get_size`.
**CN:** 这一段定义了method `get_size`（属于 `StagingBuffer`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `get_size`。

### Lines 157-158: Method `fits`
```python
    def fits(self, required_bytes: int) -> bool:
        return required_bytes <= self.size_bytes
```
**EN:** This block defines the method `fits` on `StagingBuffer`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `fits`.
**CN:** 这一段定义了method `fits`（属于 `StagingBuffer`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `fits`。

### Lines 161-161: Class `StagingAllocator` declaration
```python
class StagingAllocator:
```
**EN:** This block declares the class `StagingAllocator` and establishes its responsibility inside staging buffer allocation and movement. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `StagingAllocator`.
**CN:** 这一段声明了类 `StagingAllocator`，并说明它在暂存缓冲区分配与移动中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `StagingAllocator`。

### Lines 162-178: Supporting state inside `StagingAllocator`
```python
    """Decode-side dynamic staging ring buffer allocator with overcommit.

    One large pre-allocated GPU buffer used as a ring buffer. Each request
    gets a (alloc_id, offset, round) triple based on its actual byte
    requirement. Allocation (assign) is overcommit — it always succeeds
    as long as the request fits in the buffer. Overlap safety is enforced
    on the prefill side before RDMA, using a watermark that tracks the
    oldest un-freed allocation.

    The watermark (round, tail_offset) is periodically sent to prefill.
    Prefill transfer workers wait before writing if their target region
    overlaps with not-yet-freed data from a previous round.
    """

    # Permanent alloc failure: chunk exceeds ring buffer total size.
    ALLOC_OVERSIZED = -2

```
**EN:** This block adds supporting state or helper logic inside `StagingAllocator`. It complements the class contract with concrete fields, constants, or internal glue code. Notable operations include `a`, `Allocation`, `watermark`.
**CN:** 这一段为 `StagingAllocator` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。 值得注意的操作包括 `a`、`Allocation`、`watermark`。

### Lines 179-202: Method `__init__`
```python
    def __init__(
        self,
        total_size_bytes: int,
        device: str,
        gpu_id: int,
        custom_mem_pool=None,
    ):
        self.buffer = StagingBuffer(total_size_bytes, device, gpu_id, custom_mem_pool)
        self.total_size = total_size_bytes
        self.base_ptr = self.buffer.data_ptr
        self.head = 0
        self.round = 0
        self.allocations: dict = {}  # alloc_id -> (offset, size, round)
        self.alloc_order: List[int] = []
        self.next_alloc_id = 0
        self.watermark_round = 0
        self.watermark_tail = 0
        self.lock = threading.Lock()

        logger.info(
            f"StagingAllocator (ring+overcommit): "
            f"{total_size_bytes / (1024*1024):.1f} MB "
            f"on {device}, ptr=0x{self.base_ptr:x}"
        )
```
**EN:** This block defines the method `__init__` on `StagingAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `__init__`. Notable operations include `StagingBuffer`, `Lock`, `info`, `StagingAllocator`.
**CN:** 这一段定义了method `__init__`（属于 `StagingAllocator`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `StagingBuffer`、`Lock`、`info`、`StagingAllocator`。

### Lines 204-223: Method `assign`
```python
    def assign(self, required_bytes: int) -> Optional[Tuple[int, int, int]]:
        """Allocate a region. Returns (alloc_id, offset, round) or None."""
        with self.lock:
            if required_bytes > self.total_size:
                return None

            space_at_end = self.total_size - self.head
            if required_bytes <= space_at_end:
                offset = self.head
                self.head += required_bytes
            else:
                self.round += 1
                offset = 0
                self.head = required_bytes

            alloc_id = self.next_alloc_id
            self.next_alloc_id += 1
            self.allocations[alloc_id] = (offset, required_bytes, self.round)
            self.alloc_order.append(alloc_id)
            return (alloc_id, offset, self.round)
```
**EN:** This block defines the method `assign` on `StagingAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `assign`. Notable operations include `Returns`, `append`.
**CN:** 这一段定义了method `assign`（属于 `StagingAllocator`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `assign`。 值得注意的操作包括 `Returns`、`append`。

### Lines 225-241: Method `free`
```python
    def free(self, alloc_id: int):
        """Free an allocation and advance watermark past consecutive freed entries."""
        with self.lock:
            if alloc_id not in self.allocations:
                return
            self.allocations.pop(alloc_id)

            while self.alloc_order and self.alloc_order[0] not in self.allocations:
                self.alloc_order.pop(0)

            if not self.allocations:
                self.watermark_round = self.round
                self.watermark_tail = self.head
            elif self.alloc_order:
                off, _, rnd = self.allocations[self.alloc_order[0]]
                self.watermark_round = rnd
                self.watermark_tail = off
```
**EN:** This block defines the method `free` on `StagingAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `free`. Notable operations include `pop`.
**CN:** 这一段定义了method `free`（属于 `StagingAllocator`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `free`。 值得注意的操作包括 `pop`。

### Lines 243-246: Method `get_watermark`
```python
    def get_watermark(self) -> Tuple[int, int]:
        """Return (round, tail_offset). Everything before this is safe to write."""
        with self.lock:
            return (self.watermark_round, self.watermark_tail)
```
**EN:** This block defines the method `get_watermark` on `StagingAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `get_watermark`. Notable operations include `Return`.
**CN:** 这一段定义了method `get_watermark`（属于 `StagingAllocator`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `get_watermark`。 值得注意的操作包括 `Return`。

### Lines 248-250: Method `get_ptr`
```python
    def get_ptr(self, alloc_id: int) -> int:
        offset, _, _ = self.allocations[alloc_id]
        return self.base_ptr + offset
```
**EN:** This block defines the method `get_ptr` on `StagingAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `get_ptr`.
**CN:** 这一段定义了method `get_ptr`（属于 `StagingAllocator`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `get_ptr`。

### Lines 252-254: Method `get_offset`
```python
    def get_offset(self, alloc_id: int) -> int:
        offset, _, _ = self.allocations[alloc_id]
        return offset
```
**EN:** This block defines the method `get_offset` on `StagingAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `get_offset`.
**CN:** 这一段定义了method `get_offset`（属于 `StagingAllocator`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `get_offset`。

### Lines 256-258: Method `get_round`
```python
    def get_round(self, alloc_id: int) -> int:
        _, _, rnd = self.allocations[alloc_id]
        return rnd
```
**EN:** This block defines the method `get_round` on `StagingAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `get_round`.
**CN:** 这一段定义了method `get_round`（属于 `StagingAllocator`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `get_round`。

### Lines 260-261: Method `get_base_ptr`
```python
    def get_base_ptr(self) -> int:
        return self.base_ptr
```
**EN:** This block defines the method `get_base_ptr` on `StagingAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `get_base_ptr`.
**CN:** 这一段定义了method `get_base_ptr`（属于 `StagingAllocator`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `get_base_ptr`。

### Lines 263-264: Method `get_total_size`
```python
    def get_total_size(self) -> int:
        return self.total_size
```
**EN:** This block defines the method `get_total_size` on `StagingAllocator`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `get_total_size`.
**CN:** 这一段定义了method `get_total_size`（属于 `StagingAllocator`），介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `get_total_size`。

### Lines 267-288: Function `gather_kv_head_slices`
```python
def gather_kv_head_slices(
    kv_buffer_tensor: torch.Tensor,
    gather_idx: torch.Tensor,
    head_start: int,
    num_heads: int,
    staging_tensor: torch.Tensor,
):
    """Gather KV head slices from scattered pages into contiguous staging buffer.

    Uses torch.gather(out=) to write directly into staging_tensor without
    allocating temporary tensors (avoids CUDA caching allocator stalls).

    Args:
        kv_buffer_tensor: [pool_size, head_num, head_dim], one layer.
        gather_idx: [num_tokens, num_heads, head_dim] int64, pre-computed
            token indices expanded for gather on dim=0.
        head_start: Starting head index for the slice.
        num_heads: Number of heads to gather.
        staging_tensor: Output tensor, shape [num_tokens, num_heads, head_dim].
    """
    src = kv_buffer_tensor[:, head_start : head_start + num_heads, :]
    torch.gather(src, 0, gather_idx, out=staging_tensor)
```
**EN:** This block defines the function `gather_kv_head_slices`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `gather_kv_head_slices`. Notable operations include `gather`, `tensors`.
**CN:** 这一段定义了function `gather_kv_head_slices`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `gather_kv_head_slices`。 值得注意的操作包括 `gather`、`tensors`。

### Lines 291-308: Function `scatter_kv_head_slices` signature and setup
```python
def scatter_kv_head_slices(
    staging_tensor: torch.Tensor,
    kv_buffer_tensor: torch.Tensor,
    page_indices: torch.Tensor,
    head_start: int,
    num_heads: int,
    page_size: int = 1,
):
    """Scatter KV head slices from contiguous staging buffer to KV cache.

    Args:
        staging_tensor: Input tensor from staging buffer (contiguous packed data).
        kv_buffer_tensor: The KV buffer for one layer, shape [pool_size, head_num, head_dim].
        page_indices: [num_pages] int32/int64 tensor of page indices.
        head_start: Starting head index for the slice.
        num_heads: Number of heads to scatter.
        page_size: Number of tokens per page.
    """
```
**EN:** This block defines the function `scatter_kv_head_slices`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `scatter_kv_head_slices`. Notable operations include `buffer`.
**CN:** 这一段定义了function `scatter_kv_head_slices`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `scatter_kv_head_slices`。 值得注意的操作包括 `buffer`。

### Lines 309-319: Function `scatter_kv_head_slices` logic (part 1)
```python
    head_dim = kv_buffer_tensor.shape[-1]
    if page_size == 1:
        num_tokens = page_indices.shape[0]
        data = staging_tensor.reshape(num_tokens, num_heads, head_dim)
        kv_buffer_tensor[page_indices, head_start : head_start + num_heads, :] = data
    else:
        num_tokens = page_indices.shape[0] * page_size
        offsets = torch.arange(page_size, device=page_indices.device)
        token_indices = (page_indices.unsqueeze(1) * page_size + offsets).reshape(-1)
        data = staging_tensor.reshape(num_tokens, num_heads, head_dim)
        kv_buffer_tensor[token_indices, head_start : head_start + num_heads, :] = data
```
**EN:** This block continues `scatter_kv_head_slices` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `reshape`, `arange`, `unsqueeze`.
**CN:** 这一段延续了 `scatter_kv_head_slices` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `reshape`、`arange`、`unsqueeze`。

### Lines 322-332: Function `_gather_all_layers_torch` signature and setup
```python
def _gather_all_layers_torch(
    k_buffers: list,
    v_buffers: list,
    page_indices_np,
    staging_buffer: StagingBuffer,
    src_head_start: int,
    num_heads: int,
    page_size: int,
    gpu_id: int,
) -> int:
    """torch.gather path: zero per-layer allocation, one kernel per layer."""
```
**EN:** This block defines the function `_gather_all_layers_torch`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `_gather_all_layers_torch`.
**CN:** 这一段定义了function `_gather_all_layers_torch`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `_gather_all_layers_torch`。

### Lines 333-354: Function `_gather_all_layers_torch` logic (part 1)
```python
    import numpy as np

    num_layers = len(k_buffers)
    head_dim = k_buffers[0].shape[-1]
    dtype_size = k_buffers[0].element_size()
    num_tokens = len(page_indices_np) * page_size
    per_layer_bytes = num_tokens * num_heads * head_dim * dtype_size

    device = f"cuda:{gpu_id}"
    torch.cuda.set_device(gpu_id)
    page_idx_tensor = torch.from_numpy(page_indices_np.astype(np.int64)).to(device)

    if page_size == 1:
        token_indices = page_idx_tensor
    else:
        offsets = torch.arange(page_size, device=device)
        token_indices = (page_idx_tensor.unsqueeze(1) * page_size + offsets).reshape(-1)

    gather_idx = token_indices.view(-1, 1, 1).expand(num_tokens, num_heads, head_dim)

    if not hasattr(staging_buffer, "_gather_stream"):
        staging_buffer._gather_stream = torch.cuda.Stream(device=device)
```
**EN:** This block continues `_gather_all_layers_torch` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `element_size`, `set_device`, `from_numpy`, `astype`.
**CN:** 这一段延续了 `_gather_all_layers_torch` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `element_size`、`set_device`、`from_numpy`、`astype`。

### Lines 355-378: Function `_gather_all_layers_torch` logic (part 2)
```python

    staging_buffer._gather_stream.wait_stream(
        torch.cuda.default_stream(torch.device(device))
    )

    staging_view = staging_buffer.buffer
    offset = 0
    with torch.cuda.stream(staging_buffer._gather_stream):
        for layer_id in range(num_layers):
            dst = (
                staging_view[offset : offset + per_layer_bytes]
                .view(k_buffers[layer_id].dtype)
                .reshape(num_tokens, num_heads, head_dim)
            )
            gather_kv_head_slices(
                k_buffers[layer_id],
                gather_idx,
                src_head_start,
                num_heads,
                dst,
            )
            offset += per_layer_bytes
        for layer_id in range(num_layers):
            dst = (
```
**EN:** This block continues `_gather_all_layers_torch` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `wait_stream`, `default_stream`, `device`, `stream`.
**CN:** 这一段延续了 `_gather_all_layers_torch` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `wait_stream`、`default_stream`、`device`、`stream`。

### Lines 379-393: Function `_gather_all_layers_torch` logic (part 3)
```python
                staging_view[offset : offset + per_layer_bytes]
                .view(v_buffers[layer_id].dtype)
                .reshape(num_tokens, num_heads, head_dim)
            )
            gather_kv_head_slices(
                v_buffers[layer_id],
                gather_idx,
                src_head_start,
                num_heads,
                dst,
            )
            offset += per_layer_bytes

    staging_buffer._gather_stream.synchronize()
    return offset
```
**EN:** This block continues `_gather_all_layers_torch` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `view`, `reshape`, `gather_kv_head_slices`, `synchronize`.
**CN:** 这一段延续了 `_gather_all_layers_torch` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `view`、`reshape`、`gather_kv_head_slices`、`synchronize`。

### Lines 396-406: Function `_gather_all_layers_triton` signature and setup
```python
def _gather_all_layers_triton(
    k_buffers: list,
    v_buffers: list,
    page_indices_np,
    staging_buffer: StagingBuffer,
    src_head_start: int,
    num_heads: int,
    page_size: int,
    gpu_id: int,
) -> int:
    """Triton fused kernel path: single kernel launch for all layers."""
```
**EN:** This block defines the function `_gather_all_layers_triton`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `_gather_all_layers_triton`.
**CN:** 这一段定义了function `_gather_all_layers_triton`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `_gather_all_layers_triton`。

### Lines 407-431: Function `_gather_all_layers_triton` logic (part 1)
```python
    import numpy as np

    num_layers = len(k_buffers)
    head_dim = k_buffers[0].shape[-1]
    total_heads = k_buffers[0].shape[1]
    dtype_size = k_buffers[0].element_size()
    num_tokens = len(page_indices_np) * page_size
    elems_per_token = num_heads * head_dim
    per_layer_elems = num_tokens * elems_per_token
    per_layer_bytes = per_layer_elems * dtype_size
    total_bytes = per_layer_bytes * num_layers * 2

    device = f"cuda:{gpu_id}"
    torch.cuda.set_device(gpu_id)
    page_idx_tensor = torch.from_numpy(page_indices_np.astype(np.int64)).to(device)

    layer_ptrs = torch.tensor(
        [buf.data_ptr() for buf in k_buffers] + [buf.data_ptr() for buf in v_buffers],
        dtype=torch.int64,
        device=device,
    )
    # Use integer dtype matching element size for bit-preserving copy
    int_dtype_map = {1: torch.int8, 2: torch.int16, 4: torch.int32}
    int_dtype = int_dtype_map.get(dtype_size, torch.int16)
    staging_typed = staging_buffer.buffer[:total_bytes].view(int_dtype)
```
**EN:** This block continues `_gather_all_layers_triton` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `element_size`, `set_device`, `from_numpy`, `astype`.
**CN:** 这一段延续了 `_gather_all_layers_triton` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `element_size`、`set_device`、`from_numpy`、`astype`。

### Lines 432-455: Function `_gather_all_layers_triton` logic (part 2)
```python

    if not hasattr(staging_buffer, "_gather_stream"):
        staging_buffer._gather_stream = torch.cuda.Stream(device=device)

    staging_buffer._gather_stream.wait_stream(
        torch.cuda.default_stream(torch.device(device))
    )

    BLOCK_SIZE = 1024
    grid = (2 * num_layers, triton.cdiv(per_layer_elems, BLOCK_SIZE))

    with torch.cuda.stream(staging_buffer._gather_stream):
        _fused_gather_to_staging_kernel[grid](
            layer_ptrs,
            page_idx_tensor,
            staging_typed,
            num_tokens,
            total_heads * head_dim,
            src_head_start * head_dim,
            per_layer_elems,
            elems_per_token,
            page_size,
            BLOCK_SIZE,
        )
```
**EN:** This block continues `_gather_all_layers_triton` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `Stream`, `wait_stream`, `default_stream`, `device`.
**CN:** 这一段延续了 `_gather_all_layers_triton` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `Stream`、`wait_stream`、`default_stream`、`device`。

### Lines 456-458: Function `_gather_all_layers_triton` logic (part 3)
```python

    staging_buffer._gather_stream.synchronize()
    return total_bytes
```
**EN:** This block continues `_gather_all_layers_triton` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `synchronize`.
**CN:** 这一段延续了 `_gather_all_layers_triton` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `synchronize`。

### Lines 461-475: Function `gather_all_layers_to_staging` signature and setup
```python
def gather_all_layers_to_staging(
    k_buffers: list,
    v_buffers: list,
    page_indices_np,
    staging_buffer: StagingBuffer,
    src_head_start: int,
    num_heads: int,
    page_size: int,
    gpu_id: int,
) -> int:
    """Gather all layers' K and V head slices into a staging buffer.

    Returns total bytes written.
    Dispatches to Triton fused kernel when available, falls back to torch.gather.
    """
```
**EN:** This block defines the function `gather_all_layers_to_staging`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `gather_all_layers_to_staging`.
**CN:** 这一段定义了function `gather_all_layers_to_staging`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `gather_all_layers_to_staging`。

### Lines 476-496: Function `gather_all_layers_to_staging` logic (part 1)
```python
    if _USE_TRITON_STAGING:
        return _gather_all_layers_triton(
            k_buffers,
            v_buffers,
            page_indices_np,
            staging_buffer,
            src_head_start,
            num_heads,
            page_size,
            gpu_id,
        )
    return _gather_all_layers_torch(
        k_buffers,
        v_buffers,
        page_indices_np,
        staging_buffer,
        src_head_start,
        num_heads,
        page_size,
        gpu_id,
    )
```
**EN:** This block continues `gather_all_layers_to_staging` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `_gather_all_layers_triton`, `_gather_all_layers_torch`.
**CN:** 这一段延续了 `gather_all_layers_to_staging` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `_gather_all_layers_triton`、`_gather_all_layers_torch`。

### Lines 499-510: Function `_scatter_staging_to_kv_torch` signature and setup
```python
def _scatter_staging_to_kv_torch(
    staging_buffer_view: torch.Tensor,
    k_buffers: list,
    v_buffers: list,
    page_idx_tensor: torch.Tensor,
    page_size: int,
    prefill_attn_tp_size: int,
    decode_attn_tp_size: int,
    dst_tp_rank: int,
    total_kv_heads: int,
) -> None:
    """torch path for scatter."""
```
**EN:** This block defines the function `_scatter_staging_to_kv_torch`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `_scatter_staging_to_kv_torch`.
**CN:** 这一段定义了function `_scatter_staging_to_kv_torch`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `_scatter_staging_to_kv_torch`。

### Lines 511-531: Function `_scatter_staging_to_kv_torch` logic (part 1)
```python
    num_layers = len(k_buffers)
    head_dim = k_buffers[0].shape[-1]
    dtype_size = k_buffers[0].element_size()
    num_tokens = page_idx_tensor.shape[0] * page_size

    if prefill_attn_tp_size > decode_attn_tp_size:
        num_writers = prefill_attn_tp_size // max(1, decode_attn_tp_size)
    else:
        num_writers = 1

    for writer_rank in range(num_writers):
        _, num_heads, dst_head_start, _ = compute_head_slice_params(
            prefill_attn_tp_size,
            decode_attn_tp_size,
            writer_rank,
            dst_tp_rank,
            total_kv_heads,
        )
        per_layer_bytes = num_tokens * num_heads * head_dim * dtype_size
        per_rank_bytes = per_layer_bytes * num_layers * 2
        rank_base = writer_rank * per_rank_bytes
```
**EN:** This block continues `_scatter_staging_to_kv_torch` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `element_size`, `compute_head_slice_params`.
**CN:** 这一段延续了 `_scatter_staging_to_kv_torch` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `element_size`、`compute_head_slice_params`。

### Lines 532-555: Function `_scatter_staging_to_kv_torch` logic (part 2)
```python

        offset = rank_base
        for layer_id in range(num_layers):
            layer_data = (
                staging_buffer_view[offset : offset + per_layer_bytes]
                .view(k_buffers[layer_id].dtype)
                .reshape(num_tokens, num_heads, head_dim)
            )
            scatter_kv_head_slices(
                layer_data,
                k_buffers[layer_id],
                page_idx_tensor,
                dst_head_start,
                num_heads,
                page_size,
            )
            offset += per_layer_bytes
        for layer_id in range(num_layers):
            layer_data = (
                staging_buffer_view[offset : offset + per_layer_bytes]
                .view(v_buffers[layer_id].dtype)
                .reshape(num_tokens, num_heads, head_dim)
            )
            scatter_kv_head_slices(
```
**EN:** This block continues `_scatter_staging_to_kv_torch` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `view`, `reshape`, `scatter_kv_head_slices`.
**CN:** 这一段延续了 `_scatter_staging_to_kv_torch` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `view`、`reshape`、`scatter_kv_head_slices`。

### Lines 556-563: Function `_scatter_staging_to_kv_torch` logic (part 3)
```python
                layer_data,
                v_buffers[layer_id],
                page_idx_tensor,
                dst_head_start,
                num_heads,
                page_size,
            )
            offset += per_layer_bytes
```
**EN:** This block continues `_scatter_staging_to_kv_torch` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow.
**CN:** 这一段延续了 `_scatter_staging_to_kv_torch` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。

### Lines 566-577: Function `_scatter_staging_to_kv_triton` signature and setup
```python
def _scatter_staging_to_kv_triton(
    staging_buffer_view: torch.Tensor,
    k_buffers: list,
    v_buffers: list,
    page_idx_tensor: torch.Tensor,
    page_size: int,
    prefill_attn_tp_size: int,
    decode_attn_tp_size: int,
    dst_tp_rank: int,
    total_kv_heads: int,
) -> None:
    """Triton fused kernel path for scatter."""
```
**EN:** This block defines the function `_scatter_staging_to_kv_triton`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `_scatter_staging_to_kv_triton`.
**CN:** 这一段定义了function `_scatter_staging_to_kv_triton`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `_scatter_staging_to_kv_triton`。

### Lines 578-605: Function `_scatter_staging_to_kv_triton` logic (part 1)
```python
    num_layers = len(k_buffers)
    head_dim = k_buffers[0].shape[-1]
    total_heads = k_buffers[0].shape[1]
    dtype_size = k_buffers[0].element_size()
    num_tokens = page_idx_tensor.shape[0] * page_size
    device = page_idx_tensor.device

    if prefill_attn_tp_size > decode_attn_tp_size:
        num_writers = prefill_attn_tp_size // max(1, decode_attn_tp_size)
    else:
        num_writers = 1

    # All writers share the same num_heads; only dst_head_start differs
    _, num_heads, _, _ = compute_head_slice_params(
        prefill_attn_tp_size,
        decode_attn_tp_size,
        0,
        dst_tp_rank,
        total_kv_heads,
    )
    elems_per_token = num_heads * head_dim
    per_layer_elems = num_tokens * elems_per_token

    layer_ptrs = torch.tensor(
        [buf.data_ptr() for buf in k_buffers] + [buf.data_ptr() for buf in v_buffers],
        dtype=torch.int64,
        device=device,
    )
```
**EN:** This block continues `_scatter_staging_to_kv_triton` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `element_size`, `compute_head_slice_params`, `tensor`, `data_ptr`.
**CN:** 这一段延续了 `_scatter_staging_to_kv_triton` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `element_size`、`compute_head_slice_params`、`tensor`、`data_ptr`。

### Lines 606-632: Function `_scatter_staging_to_kv_triton` logic (part 2)
```python

    writer_head_offsets = torch.tensor(
        [
            compute_head_slice_params(
                prefill_attn_tp_size,
                decode_attn_tp_size,
                wr,
                dst_tp_rank,
                total_kv_heads,
            )[2]
            * head_dim
            for wr in range(num_writers)
        ],
        dtype=torch.int64,
        device=device,
    )

    int_dtype_map = {1: torch.int8, 2: torch.int16, 4: torch.int32}
    int_dtype = int_dtype_map.get(dtype_size, torch.int16)
    total_staging_bytes = (
        num_tokens * elems_per_token * dtype_size * num_layers * 2 * num_writers
    )
    staging_typed = staging_buffer_view[:total_staging_bytes].view(int_dtype)

    BLOCK_SIZE = 1024
    num_layers_x2 = 2 * num_layers
    grid = (num_writers * num_layers_x2, triton.cdiv(per_layer_elems, BLOCK_SIZE))
```
**EN:** This block continues `_scatter_staging_to_kv_triton` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `tensor`, `compute_head_slice_params`, `get`, `view`.
**CN:** 这一段延续了 `_scatter_staging_to_kv_triton` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `tensor`、`compute_head_slice_params`、`get`、`view`。

### Lines 633-646: Function `_scatter_staging_to_kv_triton` logic (part 3)
```python

    _fused_scatter_from_staging_kernel[grid](
        layer_ptrs,
        page_idx_tensor,
        staging_typed,
        writer_head_offsets,
        num_tokens,
        total_heads * head_dim,
        per_layer_elems,
        elems_per_token,
        page_size,
        num_layers_x2,
        BLOCK_SIZE,
    )
```
**EN:** This block continues `_scatter_staging_to_kv_triton` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow.
**CN:** 这一段延续了 `_scatter_staging_to_kv_triton` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。

### Lines 649-660: Function `scatter_staging_to_kv` signature and setup
```python
def scatter_staging_to_kv(
    staging_buffer_view: torch.Tensor,
    k_buffers: list,
    v_buffers: list,
    page_idx_tensor: torch.Tensor,
    page_size: int,
    prefill_attn_tp_size: int,
    decode_attn_tp_size: int,
    dst_tp_rank: int,
    total_kv_heads: int,
) -> None:
    """Scatter data from a contiguous staging region into KV cache buffers."""
```
**EN:** This block defines the function `scatter_staging_to_kv`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `scatter_staging_to_kv`.
**CN:** 这一段定义了function `scatter_staging_to_kv`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `scatter_staging_to_kv`。

### Lines 661-683: Function `scatter_staging_to_kv` logic (part 1)
```python
    if _USE_TRITON_STAGING:
        return _scatter_staging_to_kv_triton(
            staging_buffer_view,
            k_buffers,
            v_buffers,
            page_idx_tensor,
            page_size,
            prefill_attn_tp_size,
            decode_attn_tp_size,
            dst_tp_rank,
            total_kv_heads,
        )
    return _scatter_staging_to_kv_torch(
        staging_buffer_view,
        k_buffers,
        v_buffers,
        page_idx_tensor,
        page_size,
        prefill_attn_tp_size,
        decode_attn_tp_size,
        dst_tp_rank,
        total_kv_heads,
    )
```
**EN:** This block continues `scatter_staging_to_kv` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `_scatter_staging_to_kv_triton`, `_scatter_staging_to_kv_torch`.
**CN:** 这一段延续了 `scatter_staging_to_kv` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `_scatter_staging_to_kv_triton`、`_scatter_staging_to_kv_torch`。

### Lines 686-697: Function `compute_head_slice_params` signature and setup
```python
def compute_head_slice_params(
    src_attn_tp_size: int,
    dst_attn_tp_size: int,
    src_tp_rank: int,
    dst_tp_rank: int,
    total_kv_heads: int,
) -> Tuple[int, int, int, int]:
    """Compute head slicing parameters for heterogeneous TP transfer.

    Returns:
        (src_head_start, num_heads_to_send, dst_head_start, num_heads_to_send)
    """
```
**EN:** This block defines the function `compute_head_slice_params`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `compute_head_slice_params`.
**CN:** 这一段定义了function `compute_head_slice_params`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `compute_head_slice_params`。

### Lines 698-717: Function `compute_head_slice_params` logic (part 1)
```python
    src_heads_per_rank = max(1, total_kv_heads // src_attn_tp_size)
    dst_heads_per_rank = max(1, total_kv_heads // dst_attn_tp_size)

    local_tp_rank = src_tp_rank % src_attn_tp_size
    dst_tp_rank_in_group = dst_tp_rank % dst_attn_tp_size

    if src_attn_tp_size > dst_attn_tp_size:
        src_head_start = 0
        num_heads_to_send = src_heads_per_rank
        src_replication = max(1, src_attn_tp_size // total_kv_heads)
        unique_head_idx = local_tp_rank // src_replication
        dst_head_start = (unique_head_idx * src_heads_per_rank) % dst_heads_per_rank
    else:
        src_head_start = (
            dst_tp_rank_in_group * dst_heads_per_rank
        ) % src_heads_per_rank
        num_heads_to_send = dst_heads_per_rank
        dst_head_start = 0

    return src_head_start, num_heads_to_send, dst_head_start, num_heads_to_send
```
**EN:** This block continues `compute_head_slice_params` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow.
**CN:** 这一段延续了 `compute_head_slice_params` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。

### Lines 720-734: Function `compute_staging_layout` signature and setup
```python
def compute_staging_layout(
    src_attn_tp_size: int,
    dst_attn_tp_size: int,
    dst_tp_rank: int,
    total_kv_heads: int,
    num_tokens: int,
    bytes_per_head_token: int,
    num_layers: int,
) -> Tuple[int, List[int], int]:
    """Compute per-writer byte layout for a staging region.

    Returns:
        (num_writers, writer_bytes_list, total_bytes)
        where writer_bytes_list[i] = bytes for writer i covering all layers (K+V).
    """
```
**EN:** This block defines the function `compute_staging_layout`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `compute_staging_layout`. Notable operations include `layers`.
**CN:** 这一段定义了function `compute_staging_layout`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `compute_staging_layout`。 值得注意的操作包括 `layers`。

### Lines 735-750: Function `compute_staging_layout` logic (part 1)
```python
    if src_attn_tp_size > dst_attn_tp_size:
        num_writers = src_attn_tp_size // max(1, dst_attn_tp_size)
    else:
        num_writers = 1

    writer_bytes = []
    for wr in range(num_writers):
        _, nh, _, _ = compute_head_slice_params(
            src_attn_tp_size,
            dst_attn_tp_size,
            wr,
            dst_tp_rank,
            total_kv_heads,
        )
        writer_bytes.append(num_tokens * nh * bytes_per_head_token * num_layers * 2)
    return num_writers, writer_bytes, sum(writer_bytes)
```
**EN:** This block continues `compute_staging_layout` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding staging buffer allocation and movement workflow. Notable operations include `compute_head_slice_params`, `append`.
**CN:** 这一段延续了 `compute_staging_layout` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的暂存缓冲区分配与移动工作流。 值得注意的操作包括 `compute_head_slice_params`、`append`。

### Lines 753-768: Function `resolve_total_kv_heads`
```python
def resolve_total_kv_heads(
    kv_args,
    attn_tp_size: int,
) -> int:
    """Resolve the global total KV head count from kv_args metadata."""
    total = getattr(kv_args, "total_kv_head_num", 0)
    if total > 0:
        return total
    per_rank = getattr(kv_args, "kv_head_num", 0)
    if per_rank > 0:
        return per_rank * attn_tp_size
    raise ValueError(
        "Cannot resolve total_kv_heads: kv_args has neither total_kv_head_num "
        "nor kv_head_num. "
        "Ensure DecodePreallocQueue._init_kv_manager sets kv_args.kv_head_num."
    )
```
**EN:** This block defines the function `resolve_total_kv_heads`. It introduces the parameters, setup steps, and the main entry point for this piece of staging buffer allocation and movement. Definitions introduced here include `resolve_total_kv_heads`. Notable operations include `ValueError`.
**CN:** 这一段定义了function `resolve_total_kv_heads`，介绍了参数、初始化步骤，以及这部分暂存缓冲区分配与移动逻辑的主要入口。 此处引入的定义包括 `resolve_total_kv_heads`。 值得注意的操作包括 `ValueError`。

## Key Concepts / 关键概念
- `_fused_gather_to_staging_kernel`: Function that performs fused gather to staging kernel for the surrounding workflow. / `_fused_gather_to_staging_kernel`：在周边工作流中执行“fusedgatherto暂存kernel”相关任务的函数。
- `_fused_scatter_from_staging_kernel`: Function that performs fused scatter from staging kernel for the surrounding workflow. / `_fused_scatter_from_staging_kernel`：在周边工作流中执行“fusedscatterfrom暂存kernel”相关任务的函数。
- `StagingBuffer`: Class that encapsulates staging buffer behavior in this module. / `StagingBuffer`：封装与“暂存缓冲区”相关行为的类。
- `StagingAllocator`: Class that encapsulates staging allocator behavior in this module. / `StagingAllocator`：封装与“暂存allocator”相关行为的类。
- `gather_kv_head_slices`: Function that performs gather kv head slices for the surrounding workflow. / `gather_kv_head_slices`：在周边工作流中执行“gatherKVheadslices”相关任务的函数。
- `scatter_kv_head_slices`: Function that performs scatter kv head slices for the surrounding workflow. / `scatter_kv_head_slices`：在周边工作流中执行“scatterKVheadslices”相关任务的函数。
- `_gather_all_layers_torch`: Function that performs gather all layers torch for the surrounding workflow. / `_gather_all_layers_torch`：在周边工作流中执行“gatheralllayerstorch”相关任务的函数。
- `_gather_all_layers_triton`: Function that performs gather all layers triton for the surrounding workflow. / `_gather_all_layers_triton`：在周边工作流中执行“gatheralllayerstriton”相关任务的函数。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `os`, `threading`, `typing`
- **External packages / 外部依赖**: `torch`, `triton`, `numpy`
