# gpu_worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/kv_offload/cpu/gpu_worker.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `Transfer`, `compute_sub_block_ptrs`, `pin_mmap_region` for the V1 `kv_offload/cpu` subsystem. / 为 V1 的 `kv_offload/cpu` 子系统实现 `Transfer`, `compute_sub_block_ptrs`, `pin_mmap_region`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import time
from collections import deque
from dataclasses import dataclass

import numpy as np
import torch

from vllm import _custom_ops as ops
from vllm.logger import init_logger
from vllm.utils.math_utils import cdiv
from vllm.utils.platform_utils import is_pin_memory_available
from vllm.v1.kv_offload.base import (
    BlockIDsLoadStoreSpec,
    CanonicalKVCacheRef,
    CanonicalKVCaches,
    GPULoadStoreSpec,
)
from vllm.v1.kv_offload.cpu.shared_offload_region import SharedOffloadRegion
from vllm.v1.kv_offload.worker.worker import (
    OffloadingHandler,
    TransferResult,
    TransferSpec,
)

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `Transfer` class / `Transfer` 类
```python
@dataclass
class Transfer:
    job_id: int
    stream: torch.cuda.Stream
    start_event: torch.Event
    end_event: torch.Event
    num_bytes: int
```
**EN:** Uses `@dataclass` to package related state for `Transfer`. Typical fields include `job_id`, `stream`, `start_event`, `end_event`, `num_bytes`.
**CN:** `Transfer` 使用 `@dataclass` 打包相关状态。典型字段包括 `job_id`, `stream`, `start_event`, `end_event`, `num_bytes`。

### `compute_sub_block_ptrs` function / `compute_sub_block_ptrs` 函数
```python
def compute_sub_block_ptrs(
    block_ids: np.ndarray,
    block_size_factor: int,
    output: np.ndarray,
    tensor: torch.Tensor,
    skip_count: int = 0,
):
    """
    Compute byte pointers for sub-blocks of the given block IDs.

    Each block in block_ids contains block_size_factor sub-blocks.
    The pointer for sub-block j of block b is:
        base_ptr + b * row_stride + j * sub_block_size

    where sub_block_size = tensor.shape[1] // block_size_factor (gpu page size).

    This handles tensors where row_stride != block_size_factor * sub_block_size
    (e.g. non-contiguous CPU tensors).

    Args:
        block_ids: array of block IDs at the tensor's native granularity.
        block_size_factor: number of sub-blocks per block.
        output: pre-allocated int64 array to write pointers into.
        tensor: the source or destination tensor.
        skip_count: sub-blocks to skip in the first block.
    """
    assert skip_count < block_size_factor

    num_sub_blocks = len(output)
    base_ptr = tensor.data_ptr()
    row_stride = tensor.stride(0)

    if block_size_factor == 1:
        # Fast path: 1:1 mapping, no sub-block expansion needed.
        output[:] = base_ptr + block_ids[:num_sub_blocks] * row_stride
        return

    # Vectorized expansion for block_size_factor > 1.
    assert tensor.shape[1] % block_size_factor == 0
    sub_block_size = tensor.shape[1] // block_size_factor
    sub_offsets = np.arange(block_size_factor, dtype=np.int64) * sub_block_size
    # (num_blocks, 1) + (1, block_size_factor) -> (num_blocks, block_size_factor)
    all_ptrs = (
        base_ptr + block_ids.astype(np.int64)[:, np.newaxis] * row_stride
    ) + sub_offsets[np.newaxis, :]
    # Flatten and apply skip_count / truncation
    flat = all_ptrs.ravel()
    output[:] = flat[skip_count : skip_count + num_sub_blocks]
```
**EN:** This function computes derived values within the module. The docstring frames it as: Compute byte pointers for sub-blocks of the given block IDs. Key calls include `len`, `data_ptr`, `stride`, `ravel`, `arange`, `astype`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `len`, `data_ptr`, `stride`, `ravel`, `arange`, `astype`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `pin_mmap_region` function / `pin_mmap_region` 函数
```python
def pin_mmap_region(region: SharedOffloadRegion) -> None:
    """Register the entire mmap as CUDA pinned memory via cudaHostRegister."""
    rank = region.rank

    base_ptr = region._base.data_ptr()
    result = torch.cuda.cudart().cudaHostRegister(base_ptr, region.total_size_bytes, 0)
    if result.value != 0:
        logger.warning(
            "cudaHostRegister failed for rank=%d (code=%d) — "
            "transfers will still work but may be slower (unpinned DMA)",
            rank,
            result,
        )
    else:
        logger.debug(
            "cudaHostRegister rank=%d %.2f GB",
            rank,
            region.total_size_bytes / 1e9,
        )
        region.is_pinned = True
```
**EN:** This function implements `pin_mmap_region` within the module. The docstring frames it as: Register the entire mmap as CUDA pinned memory via cudaHostRegister. Key calls include `data_ptr`, `cudaHostRegister`, `warning`, `debug`, `cudart`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `pin_mmap_region`，其作用域位于the module。 关键调用包括 `data_ptr`, `cudaHostRegister`, `warning`, `debug`, `cudart`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SingleDirectionOffloadingHandler` class / `SingleDirectionOffloadingHandler` 类
```python
class SingleDirectionOffloadingHandler(OffloadingHandler):
    """
    SingleDirectionOffloadingHandler handles transfers for a single direction,
    either CPU->GPU or GPU->CPU.
    Transfers are guaranteed to be executed in order of their submission.
    Each transfer uses a unique CUDA stream, and its stream will start
    executing only after the streams of previous transfers have finished.
    """
```
**EN:** Introduces the `SingleDirectionOffloadingHandler` class on top of `OffloadingHandler`. Core methods include `__init__`, `transfer_async`, `get_finished`, `wait`, `shutdown`. Docstring signal: SingleDirectionOffloadingHandler handles transfers for a single direction, either CPU->GPU or GPU->CPU.
**CN:** 这里定义 `SingleDirectionOffloadingHandler` 类，其基类包括 `OffloadingHandler`。核心方法包括 `__init__`, `transfer_async`, `get_finished`, `wait`, `shutdown`。

### `SingleDirectionOffloadingHandler.__init__` method / `SingleDirectionOffloadingHandler.__init__` 方法
```python
    def __init__(
        self,
        gpu_tensors: list[torch.Tensor],
        cpu_tensors: list[torch.Tensor],
        block_size_factor: int,
        kv_cache_groups_data_refs: list[list[CanonicalKVCacheRef]],
        gpu_to_cpu: bool,
        mmap_region: SharedOffloadRegion | None = None,
    ):
        """
        Initialize a SingleDirectionOffloadingHandler.

        Args:
            gpu_tensors: list of GPU KV cache tensors.
                Each of shape (num_gpu_blocks, gpu_page_size_bytes) with dtype int8.
            cpu_tensors: list of CPU KV cache tensors.
                Each of shape (num_cpu_blocks, cpu_page_size_bytes) with dtype int8.
                Order should match gpu_tensors.
            kv_cache_groups_data_refs: list of CanonicalKVCacheRef per group.
            gpu_to_cpu: if True, transfer from GPU to CPU; otherwise CPU to GPU.
        """
        assert len(gpu_tensors) == len(cpu_tensors)
        assert len(gpu_tensors) > 0

        # assert input tensors are as expected
        for gpu_tensor, cpu_tensor in zip(gpu_tensors, cpu_tensors):
            assert gpu_tensor.dtype == torch.int8
            assert gpu_tensor.ndim == 2
            assert gpu_tensor.is_cuda
            assert cpu_tensor.dtype == torch.int8
            assert cpu_tensor.ndim == 2
            assert cpu_tensor.device.type == "cpu"
            _, gpu_page_size = gpu_tensor.shape
            _, cpu_page_size = cpu_tensor.shape
            assert cpu_page_size == gpu_page_size * block_size_factor

        self.src_tensors: list[torch.Tensor] = (
            gpu_tensors if gpu_to_cpu else cpu_tensors
        )
        self.dst_tensors: list[torch.Tensor] = (
            cpu_tensors if gpu_to_cpu else gpu_tensors
        )
        self.gpu_to_cpu: bool = gpu_to_cpu
        self.kv_cache_groups_data_refs = kv_cache_groups_data_refs

        # GPU blocks may be smaller
        # cpu_page_size = gpu_page_size * block_size_factor.
        self.src_block_size_factor = 1 if self.gpu_to_cpu else block_size_factor
        self.dst_block_size_factor = block_size_factor if self.gpu_to_cpu else 1

        self.transfer_type = ("GPU", "CPU") if self.gpu_to_cpu else ("CPU", "GPU")
        # mmap_region to clean up on shutdown (gpu_to_cpu handler owns it)
        self._mmap_region = mmap_region
        # job_id -> event
        self._transfer_events: dict[int, torch.Event] = {}
        # queue of transfers (job_id, stream, event)
        self._transfers: deque[Transfer] = deque()
        # list of CUDA streams available for re-use
        self._stream_pool: list[torch.cuda.Stream] = []
        # list of CUDA events available for re-use
        self._event_pool: list[torch.Event] = []
```
**EN:** This method initializes the object state within `SingleDirectionOffloadingHandler`. The docstring frames it as: Initialize a SingleDirectionOffloadingHandler. Key calls include `zip`, `deque`, `len`. It touches state such as `src_tensors`, `dst_tensors`, `gpu_to_cpu`, `kv_cache_groups_data_refs`, `src_block_size_factor`, `dst_block_size_factor`, `transfer_type`, `_mmap_region`. The control flow contains 5 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`SingleDirectionOffloadingHandler`。 关键调用包括 `zip`, `deque`, `len`。 它会读写 `src_tensors`, `dst_tensors`, `gpu_to_cpu`, `kv_cache_groups_data_refs`, `src_block_size_factor`, `dst_block_size_factor`, `transfer_type`, `_mmap_region` 等状态。 控制流包含 5 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `SingleDirectionOffloadingHandler.transfer_async` method / `SingleDirectionOffloadingHandler.transfer_async` 方法
```python
    def transfer_async(self, job_id: int, transfer_spec: TransferSpec) -> bool:
        src_spec, dst_spec = transfer_spec
        assert isinstance(src_spec, BlockIDsLoadStoreSpec)
        assert isinstance(dst_spec, BlockIDsLoadStoreSpec)

        src_blocks = src_spec.block_ids
        dst_blocks = dst_spec.block_ids
        assert src_blocks.ndim == 1
        assert dst_blocks.ndim == 1

        num_src_blocks = len(src_blocks)
        num_dst_blocks = len(dst_blocks)

        # There are 2 types of transfers:
        # 1. GPU -> CPU
        # 2. CPU -> GPU
        #
        # transfers are also to CPU blocks, EXCEPT MAYBE for the first and last block.
        # i.e. the first and last CPU blocks in src_blocks can match against
        # a smaller (byte-wise) set of GPU blocks in dst_blocks.
        # In such cases, we may need to skip some gpu-sized sub-blocks,
        # and start reading/writing from the middle of the first CPU block.
        # If we have multiple KV cache groups (when using HMA with hybrid models),
        # we may have a partial first/last CPU block per each group.
        # The group_sizes parameter encodes the size of each group of blocks
        # in the GPU dst_blocks.
        # If group_sizes is None, we assume all blocks belong to a single group.
        # The logical_offset parameter maps each group of blocks to its logical
        # offset inside the request, counting in GPU blocks.
        # This allows us to find the correct starting position
        # in the matching first CPU block.

        # extract group_sizes from the GPU spec
        gpu_spec = src_spec if self.gpu_to_cpu else dst_spec
        assert isinstance(gpu_spec, GPULoadStoreSpec)
        group_sizes = gpu_spec.group_sizes
        assert len(group_sizes) == len(self.kv_cache_groups_data_refs)

        # extract block indices from the GPU spec
        block_indices = gpu_spec.block_indices
        assert len(block_indices) == len(self.kv_cache_groups_data_refs)

        num_copy_ops = 0
        for group_size, group_data_refs in zip(
            group_sizes, self.kv_cache_groups_data_refs
        ):
            num_copy_ops += group_size * len(group_data_refs)

        all_src = np.empty(num_copy_ops, dtype=np.int64)
        all_dst = np.empty(num_copy_ops, dtype=np.int64)
    # ... omitted for brevity ...
        is_src_access_order_any = not self.gpu_to_cpu
        with torch.cuda.stream(stream):
            start_event.record(stream)
            if num_copy_ops > 0:
                ops.swap_blocks_batch(
                    batch_src,
                    batch_dst,
                    batch_sizes,
                    is_src_access_order_any=is_src_access_order_any,
                )
            end_event.record(stream)

        self._transfer_events[job_id] = end_event
        self._transfers.append(
            Transfer(
                job_id=job_id,
                stream=stream,
                start_event=start_event,
                end_event=end_event,
                num_bytes=num_transfer_bytes,
            )
        )

        # success
        return True
```
**EN:** This method implements `transfer_async` within `SingleDirectionOffloadingHandler`. Key calls include `isinstance`, `len`, `zip`, `empty`, `from_numpy`, `append`. It touches state such as `_transfer_events`. The control flow contains 8 branch(es) and 3 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `transfer_async`，其作用域位于`SingleDirectionOffloadingHandler`。 关键调用包括 `isinstance`, `len`, `zip`, `empty`, `from_numpy`, `append`。 它会读写 `_transfer_events` 等状态。 控制流包含 8 个分支和 3 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `SingleDirectionOffloadingHandler.get_finished` method / `SingleDirectionOffloadingHandler.get_finished` 方法
```python
    def get_finished(self) -> list[TransferResult]:
        results: list[TransferResult] = []
        while self._transfers and self._transfers[0].end_event.query():
            transfer = self._transfers.popleft()
            transfer_time = (
                transfer.start_event.elapsed_time(transfer.end_event) * 1e-3
            )  # elapsed_time is in milliseconds
            result = TransferResult(
                job_id=transfer.job_id,
                success=True,
                transfer_size=transfer.num_bytes,
                transfer_time=transfer_time,
                transfer_type=self.transfer_type,
            )

            results.append(result)
            self._stream_pool.append(transfer.stream)
            self._event_pool.append(transfer.end_event)
            self._event_pool.append(transfer.start_event)
            del self._transfer_events[transfer.job_id]
        return results
```
**EN:** This method returns or derives a value within `SingleDirectionOffloadingHandler`. Key calls include `query`, `popleft`, `TransferResult`, `append`, `elapsed_time`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`SingleDirectionOffloadingHandler`。 关键调用包括 `query`, `popleft`, `TransferResult`, `append`, `elapsed_time`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `CpuGpuOffloadingHandlers` class / `CpuGpuOffloadingHandlers` 类
```python
class CpuGpuOffloadingHandlers:
```
**EN:** Introduces the `CpuGpuOffloadingHandlers` class. Core methods include `__init__`.
**CN:** 这里定义 `CpuGpuOffloadingHandlers` 类。核心方法包括 `__init__`。

### `CpuGpuOffloadingHandlers.__init__` method / `CpuGpuOffloadingHandlers.__init__` 方法
```python
    def __init__(
        self,
        kv_caches: CanonicalKVCaches,
        block_size_factor: int,
        num_cpu_blocks: int,
        mmap_region: SharedOffloadRegion | None = None,
    ):
        pin_memory = is_pin_memory_available()
        logger.info("Allocating %d CPU tensors...", len(kv_caches.tensors))
        self._mmap_region = mmap_region
        if mmap_region is not None and pin_memory:
            pin_mmap_region(mmap_region)

        gpu_tensors: list[torch.Tensor] = []
        cpu_tensors: list[torch.Tensor] = []
        for kv_cache_tensor in kv_caches.tensors:
            gpu_page_size_bytes = kv_cache_tensor.page_size_bytes
            gpu_tensor = kv_cache_tensor.tensor.view(torch.int8).view(
                (-1, gpu_page_size_bytes)
            )
            cpu_page_size_bytes = gpu_page_size_bytes * block_size_factor

            if mmap_region is not None:
                cpu_tensor = mmap_region.create_next_view(cpu_page_size_bytes)
            else:
                t0 = time.monotonic()
                cpu_tensor = torch.zeros(
                    (num_cpu_blocks, cpu_page_size_bytes),
                    dtype=torch.int8,
                    device="cpu",
                    pin_memory=pin_memory,
                )
                logger.debug(
                    "torch.zeros pinned tensor %d×%d (%.2f GB): %.3f s",
                    num_cpu_blocks,
                    cpu_page_size_bytes,
                    num_cpu_blocks * cpu_page_size_bytes / 1e9,
                    time.monotonic() - t0,
                )

            gpu_tensors.append(gpu_tensor)
            cpu_tensors.append(cpu_tensor)

        self.gpu_to_cpu_handler = SingleDirectionOffloadingHandler(
            gpu_tensors=gpu_tensors,
            cpu_tensors=cpu_tensors,
            block_size_factor=block_size_factor,
            kv_cache_groups_data_refs=kv_caches.group_data_refs,
            gpu_to_cpu=True,
            mmap_region=mmap_region,
        )

        self.cpu_to_gpu_handler = SingleDirectionOffloadingHandler(
            gpu_tensors=gpu_tensors,
            cpu_tensors=cpu_tensors,
            block_size_factor=block_size_factor,
            kv_cache_groups_data_refs=kv_caches.group_data_refs,
            gpu_to_cpu=False,
        )
```
**EN:** This method initializes the object state within `CpuGpuOffloadingHandlers`. Key calls include `is_pin_memory_available`, `info`, `SingleDirectionOffloadingHandler`, `len`, `pin_mmap_region`, `view`. It touches state such as `_mmap_region`, `gpu_to_cpu_handler`, `cpu_to_gpu_handler`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`CpuGpuOffloadingHandlers`。 关键调用包括 `is_pin_memory_available`, `info`, `SingleDirectionOffloadingHandler`, `len`, `pin_mmap_region`, `view`。 它会读写 `_mmap_region`, `gpu_to_cpu_handler`, `cpu_to_gpu_handler` 等状态。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `Transfer`: central class or interface in this module. / `Transfer`：本模块中的核心类或接口。
- `compute_sub_block_ptrs`: top-level helper or orchestration entry point. / `compute_sub_block_ptrs`：顶层辅助函数或编排入口。
- `pin_mmap_region`: top-level helper or orchestration entry point. / `pin_mmap_region`：顶层辅助函数或编排入口。
- `SingleDirectionOffloadingHandler`: central class or interface in this module. / `SingleDirectionOffloadingHandler`：本模块中的核心类或接口。
- `CpuGpuOffloadingHandlers`: central class or interface in this module. / `CpuGpuOffloadingHandlers`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `time`, `collections`, `dataclasses`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm`, `vllm.logger`, `vllm.utils.math_utils`, `vllm.utils.platform_utils`, `vllm.v1.kv_offload.base`, `vllm.v1.kv_offload.cpu.shared_offload_region`, `vllm.v1.kv_offload.worker.worker`
