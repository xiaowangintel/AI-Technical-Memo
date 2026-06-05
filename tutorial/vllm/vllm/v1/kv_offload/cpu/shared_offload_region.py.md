# shared_offload_region.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/kv_offload/cpu/shared_offload_region.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_wait_for_file_size`, `SharedOffloadRegion` for the V1 `kv_offload/cpu` subsystem. / 为 V1 的 `kv_offload/cpu` 子系统实现 `_wait_for_file_size`, `SharedOffloadRegion`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import mmap
import os
import time

import torch

from vllm.logger import init_logger

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `_wait_for_file_size` function / `_wait_for_file_size` 函数
```python
def _wait_for_file_size(fd: int, expected_size: int, timeout: float = 30.0) -> None:
    """Spin-wait until the file reaches expected_size (creator truncated it)."""
    deadline = time.monotonic() + timeout
    while True:
        if os.fstat(fd).st_size >= expected_size:
            return
        if time.monotonic() > deadline:
            raise TimeoutError(
                f"Timed out waiting for mmap file to reach {expected_size} bytes"
            )
        time.sleep(0.005)
```
**EN:** This function implements `_wait_for_file_size` within the module. The docstring frames it as: Spin-wait until the file reaches expected_size (creator truncated it). Key calls include `monotonic`, `sleep`, `TimeoutError`, `fstat`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_wait_for_file_size`，其作用域位于the module。 关键调用包括 `monotonic`, `sleep`, `TimeoutError`, `fstat`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `SharedOffloadRegion` class / `SharedOffloadRegion` 类
```python
class SharedOffloadRegion:
    """
    Single mmap-backed memory region shared across all workers for a
    vLLM instance.  Workers coordinate via the filesystem: the first worker
    to open the file with O_EXCL becomes the creator and calls ftruncate;
    the rest open the existing file and wait until it reaches the expected
    size.  Each worker then mmap()s the full file.

    File path: /dev/shm/vllm_offload_{instance_id}.mmap
    """
```
**EN:** Introduces the `SharedOffloadRegion` class. Core methods include `__init__`, `create_next_view`, `create_kv_memoryview`, `cleanup`. Docstring signal: Single mmap-backed memory region shared across all workers for a vLLM instance.
**CN:** 这里定义 `SharedOffloadRegion` 类。核心方法包括 `__init__`, `create_next_view`, `create_kv_memoryview`, `cleanup`。

### `SharedOffloadRegion.__init__` method / `SharedOffloadRegion.__init__` 方法
```python
    def __init__(
        self,
        instance_id: str,
        total_size_bytes: int,
        num_blocks: int,
        rank: int | None,
        num_workers: int,
        cpu_page_size: int,
    ) -> None:
        self.page_size = mmap.PAGESIZE

        self.total_size_bytes = total_size_bytes
        self.mmap_path = f"/dev/shm/vllm_offload_{instance_id}.mmap"
        self._creator = False  # set True only if this worker creates the file
        self.num_blocks = num_blocks
        self.rank = rank
        # interleaved-layout stride: one row = all workers' data for one block
        self._row_stride = cpu_page_size * num_workers
        if rank is not None:
            # byte offset to this worker's first slot within each block row
            self._worker_offset = rank * cpu_page_size
            # exclusive upper bound for this worker's area within each row
            self._worker_area_end = (rank + 1) * cpu_page_size
        try:
            # Exclusive create — only one worker succeeds
            self.fd: int | None = os.open(
                self.mmap_path, os.O_CREAT | os.O_EXCL | os.O_RDWR, 0o600
            )
            os.ftruncate(self.fd, self.total_size_bytes)
            self._creator = True
            logger.info(
                "Created mmap file %s (%.2f GB)",
                self.mmap_path,
                self.total_size_bytes / 1e9,
            )
        except FileExistsError:
            self.fd = os.open(self.mmap_path, os.O_RDWR)
            _wait_for_file_size(self.fd, self.total_size_bytes)
            logger.info("Opened existing mmap file %s", self.mmap_path)

        self.mmap_obj: mmap.mmap | None = mmap.mmap(
            self.fd,
            self.total_size_bytes,
            flags=mmap.MAP_SHARED,
            prot=mmap.PROT_READ | mmap.PROT_WRITE,
        )

        # MADV_POPULATE_WRITE was added in Linux 5.14 (value 23).
        _MADV_POPULATE_WRITE = getattr(mmap, "MADV_POPULATE_WRITE", 23)
        if rank is not None:
            # Populate only this worker's pages (one slot per block row).
            worker_offset = rank * cpu_page_size
            _t0 = time.perf_counter()
            page_size = self.page_size
            for block in range(num_blocks):
                raw_offset = block * self._row_stride + worker_offset
                aligned_offset = (raw_offset // page_size) * page_size
                end = raw_offset + cpu_page_size
                aligned_length = end - aligned_offset
                self.mmap_obj.madvise(
                    _MADV_POPULATE_WRITE, aligned_offset, aligned_length
                )
            logger.debug(
                "MADV_POPULATE_WRITE loop: %d blocks in %.3f s",
                num_blocks,
                time.perf_counter() - _t0,
            )
        else:
            # No rank — populate the entire shared region in one call.
            _t0 = time.perf_counter()
            self.mmap_obj.madvise(_MADV_POPULATE_WRITE, 0, self.total_size_bytes)
            logger.debug(
                "MADV_POPULATE_WRITE entire region: %.3f s", time.perf_counter() - _t0
            )

        self._base = torch.frombuffer(memoryview(self.mmap_obj), dtype=torch.int8)
        self._views: list[torch.Tensor] = []
        self.is_pinned: bool = False
```
**EN:** This method initializes the object state within `SharedOffloadRegion`. Key calls include `mmap`, `getattr`, `frombuffer`, `open`, `ftruncate`, `info`. It touches state such as `page_size`, `total_size_bytes`, `mmap_path`, `_creator`, `num_blocks`, `rank`, `_row_stride`, `mmap_obj`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`SharedOffloadRegion`。 关键调用包括 `mmap`, `getattr`, `frombuffer`, `open`, `ftruncate`, `info`。 它会读写 `page_size`, `total_size_bytes`, `mmap_path`, `_creator`, `num_blocks`, `rank`, `_row_stride`, `mmap_obj` 等状态。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `SharedOffloadRegion.create_next_view` method / `SharedOffloadRegion.create_next_view` 方法
```python
    def create_next_view(self, tensor_page_size: int) -> torch.Tensor:
        """Allocate a strided int8 view for this worker, one canonical tensor.

        Must be called once per canonical tensor. The full mmap layout is:

            worker0_block0 | worker1_block0 | ... | worker{M-1}_block0
            worker0_block1 | worker1_block1 | ... | worker{M-1}_block1
            ...

        Each worker_block cell is cpu_page_size bytes and holds all canonical
        tensors for that worker and block concatenated:
            [ tensor0_data | tensor1_data | ... | tensor{L-1}_data ]

        Consecutive rows are separated by row_stride = cpu_page_size * M.

        Returns an int8 tensor of shape (num_blocks, tensor_page_size) with stride
        (row_stride, 1).  Using int8 keeps stride == bytes, so swap_blocks
        address arithmetic works without any dtype conversion.

        Args:
            tensor_page_size: Bytes per block for this  tensor.
        """
        assert self.rank is not None
        new_offset = self._worker_offset + tensor_page_size
        assert new_offset <= self._worker_area_end, (
            f"Worker offset {new_offset} exceeds worker area end "
            f"{self._worker_area_end} (overflowed by "
            f"{new_offset - self._worker_area_end} bytes)"
        )
        worker_layer_view = torch.as_strided(
            self._base,
            size=(self.num_blocks, tensor_page_size),
            stride=(self._row_stride, 1),
            storage_offset=self._worker_offset,
        )
        self._worker_offset = new_offset
        self._views.append(worker_layer_view)
        return worker_layer_view
```
**EN:** This method creates a new object or plan within `SharedOffloadRegion`. The docstring frames it as: Allocate a strided int8 view for this worker, one canonical tensor. Key calls include `as_strided`, `append`. It touches state such as `_worker_offset`.
**CN:** 该方法会创建新的对象或计划，其作用域位于`SharedOffloadRegion`。 关键调用包括 `as_strided`, `append`。 它会读写 `_worker_offset` 等状态。

### `SharedOffloadRegion.create_kv_memoryview` method / `SharedOffloadRegion.create_kv_memoryview` 方法
```python
    def create_kv_memoryview(self) -> memoryview:
        """Return a zero-copy memoryview over the entire KV buffer.

        Shape: (num_blocks, row_stride_bytes). Secondary tiers address
        block *b* as ``view[b]``.
        """
        kv_tensor = self._base.view(self.num_blocks, self._row_stride)
        np_arr = kv_tensor.numpy()
        assert np_arr.ctypes.data == self._base.data_ptr(), (
            "view()/numpy() created a copy instead of sharing the mmap buffer; "
            "secondary tiers require zero-copy access to primary KV data"
        )
        return memoryview(np_arr)
```
**EN:** This method creates a new object or plan within `SharedOffloadRegion`. The docstring frames it as: Return a zero-copy memoryview over the entire KV buffer. Key calls include `view`, `numpy`, `memoryview`, `data_ptr`.
**CN:** 该方法会创建新的对象或计划，其作用域位于`SharedOffloadRegion`。 关键调用包括 `view`, `numpy`, `memoryview`, `data_ptr`。

### `SharedOffloadRegion.cleanup` method / `SharedOffloadRegion.cleanup` 方法
```python
    def cleanup(self) -> None:
        if self.is_pinned and self._base is not None:
            base_ptr = self._base.data_ptr()
            result = torch.cuda.cudart().cudaHostUnregister(base_ptr)
            if result.value != 0:
                logger.warning(
                    "cudaHostUnregister failed for rank=%d (code=%d)", self.rank, result
                )
            self.is_pinned = False
        # Release views before _base: each view holds a _base reference and a
        # direct StorageImpl reference.  Freeing views first lets both refcounts
        # drop so the storage (which holds the mmap_obj buffer export) is freed
        # before mmap_obj.close() is called below.
        if self._views is not None:
            self._views.clear()
        self._base = None
        if self.mmap_obj:
            try:
                self.mmap_obj.close()
            except Exception:
                logger.warning("Failed to close mmap_obj", exc_info=True)
            self.mmap_obj = None
        if self.fd is not None:
            try:
                os.close(self.fd)
            except Exception:
                logger.warning("Failed to close fd %s", self.fd, exc_info=True)
            self.fd = None
        if self._creator and getattr(self, "mmap_path", None):
            try:
                os.unlink(self.mmap_path)
                logger.info("Removed mmap file %s", self.mmap_path)
            except Exception:
                logger.warning(
                    "Failed to unlink path %s", self.mmap_path, exc_info=True
                )
            self._creator = False
```
**EN:** This method implements `cleanup` within `SharedOffloadRegion`. Key calls include `data_ptr`, `cudaHostUnregister`, `clear`, `getattr`, `warning`, `close`. It touches state such as `_base`, `is_pinned`, `mmap_obj`, `fd`, `_creator`. The control flow contains 9 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `cleanup`，其作用域位于`SharedOffloadRegion`。 关键调用包括 `data_ptr`, `cudaHostUnregister`, `clear`, `getattr`, `warning`, `close`。 它会读写 `_base`, `is_pinned`, `mmap_obj`, `fd`, `_creator` 等状态。 控制流包含 9 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_wait_for_file_size`: top-level helper or orchestration entry point. / `_wait_for_file_size`：顶层辅助函数或编排入口。
- `SharedOffloadRegion`: central class or interface in this module. / `SharedOffloadRegion`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `mmap`, `os`, `time`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.logger`
