# worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/simple_kv_offload/worker.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Worker-side handler for SimpleCPUOffloadConnector. / 该模块位于 `simple_kv_offload` 子系统，主要围绕 `SimpleCPUOffloadWorker` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Worker-side handler for SimpleCPUOffloadConnector."""

from typing import TYPE_CHECKING

import torch

from vllm.config import VllmConfig
from vllm.logger import init_logger
from vllm.utils.platform_utils import is_pin_memory_available
from vllm.v1.simple_kv_offload.copy_backend import DmaCopyBackend
from vllm.v1.simple_kv_offload.cuda_mem_ops import pin_tensor
from vllm.v1.simple_kv_offload.metadata import (
    SimpleCPUOffloadMetadata,
    SimpleCPUOffloadWorkerMetadata,
)

if TYPE_CHECKING:
    from vllm.v1.kv_cache_interface import KVCacheConfig

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `SimpleCPUOffloadWorker` class / `SimpleCPUOffloadWorker` 类
```python
class SimpleCPUOffloadWorker:
    """Worker-side handler for CPU offloading transfers."""
```
**EN:** Introduces the `SimpleCPUOffloadWorker` class. Core methods include `__init__`, `register_kv_caches`, `bind_connector_metadata`, `clear_connector_metadata`, `start_load_kv`, `wait_for_save`. Docstring signal: Worker-side handler for CPU offloading transfers.
**CN:** 这里定义 `SimpleCPUOffloadWorker` 类。核心方法包括 `__init__`, `register_kv_caches`, `bind_connector_metadata`, `clear_connector_metadata`, `start_load_kv`, `wait_for_save`。

### `SimpleCPUOffloadWorker.__init__` method / `SimpleCPUOffloadWorker.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        kv_cache_config: "KVCacheConfig | None",
        cpu_capacity_bytes: int,
    ):
        self.vllm_config = vllm_config
        self.kv_cache_config = kv_cache_config
        self.cpu_capacity_bytes = cpu_capacity_bytes

        self.gpu_kv_caches: dict[str, torch.Tensor] | None = None
        self.cpu_kv_caches: dict[str, torch.Tensor] | None = None
        self.device: torch.device | None = None
        self.num_cpu_blocks: int = 0

        # CUDA streams for the async transfers
        self.load_stream: torch.cuda.Stream | None = None
        self.store_stream: torch.cuda.Stream | None = None

        self._backend = DmaCopyBackend()

        # Ordered (event_idx, Event). Events pre-allocated on main thread.
        self._load_events: list[tuple[int, torch.Event]] = []
        self._store_events: list[tuple[int, torch.Event]] = []
        # High-water marks: highest event_idx completed per stream.
        # When the event list is empty, the hwm covers all prior events.
        self._load_hwm: int = -1
        self._store_hwm: int = -1

        # Metadata for the current step
        self._connector_metadata: SimpleCPUOffloadMetadata | None = None

        # Pending event index sets, populated in bind_connector_metadata
        self._pending_load_event_indices: set[int] = set()
        self._pending_store_event_indices: set[int] = set()
        # Completed store events to report via build_connector_worker_meta
        self._completed_store_events: dict[int, int] = {}
```
**EN:** This method initializes the object state within `SimpleCPUOffloadWorker`. Key calls include `DmaCopyBackend`, `set`. It touches state such as `vllm_config`, `kv_cache_config`, `cpu_capacity_bytes`, `gpu_kv_caches`, `cpu_kv_caches`, `device`, `num_cpu_blocks`, `load_stream`.
**CN:** 该方法会初始化对象状态，其作用域位于`SimpleCPUOffloadWorker`。 关键调用包括 `DmaCopyBackend`, `set`。 它会读写 `vllm_config`, `kv_cache_config`, `cpu_capacity_bytes`, `gpu_kv_caches`, `cpu_kv_caches`, `device`, `num_cpu_blocks`, `load_stream` 等状态。

### `SimpleCPUOffloadWorker.register_kv_caches` method / `SimpleCPUOffloadWorker.register_kv_caches` 方法
```python
    def register_kv_caches(
        self,
        kv_caches: dict[str, torch.Tensor],
    ) -> None:
        """Register GPU KV caches and allocate pinned CPU tensors.
        The worker will infer the underlying raw storage from the kv_caches.

        Args:
            kv_caches: Per-layer GPU KV caches. Values are either a single
                tensor (attention layers) or a list of tensors (Mamba layers
                in hybrid models). All values are included for offloading
                by resolving to their underlying raw storage.
        """
        if not kv_caches:
            logger.warning("No KV caches to offload.")
            return

        # Resolve each entry to a representative tensor for storage
        # deduplication. For attention layers the value is already a tensor;
        # for Mamba layers it is a list of tensors that all share the same
        # underlying raw storage, so we take the first one.
        def _repr_tensor(v: torch.Tensor | list[torch.Tensor]) -> torch.Tensor:
            assert isinstance(v, torch.Tensor | list)
            return v if isinstance(v, torch.Tensor) else v[0]

        any_tensor = _repr_tensor(next(iter(kv_caches.values())))
        self.device = any_tensor.device

        assert self.kv_cache_config is not None
        num_blocks = self.kv_cache_config.num_blocks

        # Deduplicate: multiple layers may share the same backing storage.
        seen_ptrs: dict[int, tuple[str, torch.Tensor]] = {}
        for name, value in kv_caches.items():
            tensor = _repr_tensor(value)
            ptr = tensor.untyped_storage().data_ptr()
            if ptr not in seen_ptrs:
                seen_ptrs[ptr] = (name, tensor)

        # Build [num_blocks, block_bytes] int8 views from each unique
        # storage so that stride(0) gives block_bytes for the copy op.
        #
        # The physical layout varies across attention backends:
        #   FlashAttn/ROCm:  (2, num_blocks, ...) -> K/V outermost, 2 segments
        #   FlashInfer/MLA:  (num_blocks, ...)    -> blocks outermost, 1 segment
        # We derive page_size_bytes = storage.nbytes() // num_blocks, then
        # classify dims: any dim whose byte-stride exceeds page_size_bytes
        # must be an outer segment dim (e.g. the K/V dim of size 2). A less
        # hacky way is to update the interface with the layout.
        unique_gpu_caches: dict[str, torch.Tensor] = {}
    # ... omitted for brevity ...
        self.gpu_kv_caches = unique_gpu_caches
        self.cpu_kv_caches = {}
        for name, gpu_tensor in unique_gpu_caches.items():
            cpu_shape = (self.num_cpu_blocks,) + gpu_tensor.shape[1:]
            # Allocate non-pinned first, then pin via cudaHostRegister to
            # bypass PyTorch's CUDACachingHostAllocator which rounds up to
            # the next power of 2 (e.g. 100 GB -> 128 GB).
            tensor = torch.zeros(cpu_shape, dtype=gpu_tensor.dtype, device="cpu")
            if pin_memory:
                pin_tensor(tensor)
            self.cpu_kv_caches[name] = tensor

        # Use lowest priority so KV cache I/O yields to compute streams.
        low_pri, _ = torch.cuda.Stream.priority_range()
        self.load_stream = torch.cuda.Stream(priority=low_pri)
        self.store_stream = torch.cuda.Stream(priority=low_pri)

        # Initialize copy backend with caches and streams.
        self._backend.init(
            self.gpu_kv_caches,
            self.cpu_kv_caches,
            self.device,
            self.load_stream,
            self.store_stream,
        )
```
**EN:** This method implements `register_kv_caches` within `SimpleCPUOffloadWorker`. The docstring frames it as: Register GPU KV caches and allocate pinned CPU tensors. Key calls include `_repr_tensor`, `items`, `values`, `sum`, `max`, `info`. It touches state such as `device`, `num_cpu_blocks`, `gpu_kv_caches`, `cpu_kv_caches`, `load_stream`, `store_stream`. The control flow contains 6 branch(es) and 4 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `register_kv_caches`，其作用域位于`SimpleCPUOffloadWorker`。 关键调用包括 `_repr_tensor`, `items`, `values`, `sum`, `max`, `info`。 它会读写 `device`, `num_cpu_blocks`, `gpu_kv_caches`, `cpu_kv_caches`, `load_stream`, `store_stream` 等状态。 控制流包含 6 个分支和 4 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `SimpleCPUOffloadWorker.bind_connector_metadata` method / `SimpleCPUOffloadWorker.bind_connector_metadata` 方法
```python
    def bind_connector_metadata(self, metadata: SimpleCPUOffloadMetadata) -> None:
        self._connector_metadata = metadata
        if metadata.load_event >= 0:
            self._pending_load_event_indices.add(metadata.load_event)
        if metadata.store_event >= 0:
            self._pending_store_event_indices.add(metadata.store_event)
```
**EN:** This method implements `bind_connector_metadata` within `SimpleCPUOffloadWorker`. Key calls include `add`. It touches state such as `_connector_metadata`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `bind_connector_metadata`，其作用域位于`SimpleCPUOffloadWorker`。 关键调用包括 `add`。 它会读写 `_connector_metadata` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SimpleCPUOffloadWorker.start_load_kv` method / `SimpleCPUOffloadWorker.start_load_kv` 方法
```python
    def start_load_kv(self) -> None:
        # NOTE: we defer launching both load and store to get_finished(),
        # which runs after model execution. This hides the CPU-side
        # block copy op overhead (~5ms) behind GPU compute.
        pass
```
**EN:** This method implements `start_load_kv` within `SimpleCPUOffloadWorker`.
**CN:** 该方法会实现 `start_load_kv`，其作用域位于`SimpleCPUOffloadWorker`。

### `SimpleCPUOffloadWorker.wait_for_save` method / `SimpleCPUOffloadWorker.wait_for_save` 方法
```python
    def wait_for_save(self) -> None:
        pass
```
**EN:** This method implements `wait_for_save` within `SimpleCPUOffloadWorker`.
**CN:** 该方法会实现 `wait_for_save`，其作用域位于`SimpleCPUOffloadWorker`。

### `SimpleCPUOffloadWorker.get_finished` method / `SimpleCPUOffloadWorker.get_finished` 方法
```python
    def get_finished(
        self,
        finished_req_ids: set[str],
    ) -> tuple[set[str] | None, set[str] | None]:
        """Submit transfers and report completed events to the scheduler.

        Called after model execution. The manager only schedules stores for
        blocks whose KV data is confirmed computed, so we launch both loads
        and stores immediately — no deferral or cross-stream sync needed.

        Returns:
            tuple of (finished_sending, finished_recving).
            - finished_sending: always None (stores use worker metadata).
            - finished_recving: req_ids whose loads have completed.
        """
        # (1) Submit transfers
        metadata = self._connector_metadata
        if metadata is not None:
            # Launch loads (CPU->GPU).
            if metadata.load_cpu_blocks:
                self._backend.launch_copy(
                    metadata.load_cpu_blocks,
                    metadata.load_gpu_blocks,
                    is_store=False,
                    event_idx=metadata.load_event,
                    events_list=self._load_events,
                )
            # Launch stores (GPU->CPU).
            if metadata.store_gpu_blocks:
                self._backend.launch_copy(
                    metadata.store_gpu_blocks,
                    metadata.store_cpu_blocks,
                    is_store=True,
                    event_idx=metadata.store_event,
                    events_list=self._store_events,
                )

        # (2) Track completed transfer events
        finished_recving: set[str] = set()

        if self._pending_load_event_indices:
            load_wm = self._poll_stream_events(is_store=False)
            for j in [j for j in self._pending_load_event_indices if j <= load_wm]:
                self._pending_load_event_indices.discard(j)
                req_ids = (
                    metadata.load_event_to_reqs.get(j) if metadata is not None else None
                )
                if req_ids:
                    finished_recving.update(req_ids)

        if self._pending_store_event_indices:
            store_wm = self._poll_stream_events(is_store=True)
            for j in [j for j in self._pending_store_event_indices if j <= store_wm]:
                self._pending_store_event_indices.discard(j)
                self._completed_store_events[j] = 1

        return None, finished_recving or None
```
**EN:** This method returns or derives a value within `SimpleCPUOffloadWorker`. The docstring frames it as: Submit transfers and report completed events to the scheduler. Key calls include `set`, `_poll_stream_events`, `launch_copy`, `discard`, `get`, `update`. It touches state such as `_completed_store_events`. The control flow contains 7 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`SimpleCPUOffloadWorker`。 关键调用包括 `set`, `_poll_stream_events`, `launch_copy`, `discard`, `get`, `update`。 它会读写 `_completed_store_events` 等状态。 控制流包含 7 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `SimpleCPUOffloadWorker.build_connector_worker_meta` method / `SimpleCPUOffloadWorker.build_connector_worker_meta` 方法
```python
    def build_connector_worker_meta(self) -> SimpleCPUOffloadWorkerMetadata | None:
        """Return completed store events since the last call."""
        if not self._completed_store_events:
            return None
        meta = SimpleCPUOffloadWorkerMetadata(
            completed_store_events=self._completed_store_events,
        )
        self._completed_store_events = {}
        return meta
```
**EN:** This method builds derived structures within `SimpleCPUOffloadWorker`. The docstring frames it as: Return completed store events since the last call. Key calls include `SimpleCPUOffloadWorkerMetadata`. It touches state such as `_completed_store_events`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会构建派生结构，其作用域位于`SimpleCPUOffloadWorker`。 关键调用包括 `SimpleCPUOffloadWorkerMetadata`。 它会读写 `_completed_store_events` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SimpleCPUOffloadWorker.handle_preemptions` method / `SimpleCPUOffloadWorker.handle_preemptions` 方法
```python
    def handle_preemptions(
        self, kv_connector_metadata: SimpleCPUOffloadMetadata
    ) -> None:
        """Sync all in-flight transfers before preempted blocks are reused."""
        if not kv_connector_metadata.need_flush:
            return
        self._flush_and_sync_all()
```
**EN:** This method implements `handle_preemptions` within `SimpleCPUOffloadWorker`. The docstring frames it as: Sync all in-flight transfers before preempted blocks are reused. Key calls include `_flush_and_sync_all`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `handle_preemptions`，其作用域位于`SimpleCPUOffloadWorker`。 关键调用包括 `_flush_and_sync_all`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `SimpleCPUOffloadWorker`: central class or interface in this module. / `SimpleCPUOffloadWorker`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.logger`, `vllm.utils.platform_utils`, `vllm.v1.simple_kv_offload.copy_backend`, `vllm.v1.simple_kv_offload.cuda_mem_ops`, `vllm.v1.simple_kv_offload.metadata`, `vllm.v1.kv_cache_interface`
