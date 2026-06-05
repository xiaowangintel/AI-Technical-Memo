# decode_kvcache_offload_manager.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/decode_kvcache_offload_manager.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file manages decode-side KV cache offload and retrieval behavior. It focuses on where KV data should live and how transfer-related memory is reclaimed or restored. / 该文件管理解码侧 KV 缓存的卸载与取回行为，重点处理 KV 数据的驻留位置以及传输相关内存的回收与恢复。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: Imports and module setup
```python
from __future__ import annotations

import json
import logging
import threading
import time
from typing import TYPE_CHECKING

import torch

from sglang.srt.disaggregation.kv_events import OffloadedState
from sglang.srt.environ import envs
from sglang.srt.managers.cache_controller import HiCacheController
from sglang.srt.mem_cache.allocator import BaseTokenToKVPoolAllocator
from sglang.srt.mem_cache.base_prefix_cache import BasePrefixCache
from sglang.srt.mem_cache.memory_pool import (
    MHATokenToKVPool,
    MLATokenToKVPool,
    ReqToTokenPool,
)
from sglang.srt.mem_cache.memory_pool_host import (
    MHATokenToKVPoolHost,
    MLATokenToKVPoolHost,
)
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils.common import ceil_align

if TYPE_CHECKING:
    from sglang.srt.managers.schedule_batch import Req

logger = logging.getLogger(__name__)


```
**EN:** This block gathers the imports and module-level setup for decode KV-cache offload management. The imported modules show which runtime services, schemas, or backend components this file depends on. Notable operations include `import`, `getLogger`.
**CN:** 这一段汇集了与解码 KV 缓存卸载管理相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。 值得注意的操作包括 `import`、`getLogger`。

### Lines 34-34: Class `DecodeKVCacheOffloadManager` declaration
```python
class DecodeKVCacheOffloadManager:
```
**EN:** This block declares the class `DecodeKVCacheOffloadManager` and establishes its responsibility inside decode KV-cache offload management. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `DecodeKVCacheOffloadManager`.
**CN:** 这一段声明了类 `DecodeKVCacheOffloadManager`，并说明它在解码 KV 缓存卸载管理中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `DecodeKVCacheOffloadManager`。

### Lines 35-36: Supporting state inside `DecodeKVCacheOffloadManager`
```python
    """Manage decode-side KV cache offloading lifecycle and operations."""

```
**EN:** This block adds supporting state or helper logic inside `DecodeKVCacheOffloadManager`. It complements the class contract with concrete fields, constants, or internal glue code.
**CN:** 这一段为 `DecodeKVCacheOffloadManager` 补充了支撑性的状态或辅助逻辑，通过具体字段、常量或内部胶水代码来落实该类的设计意图。

### Lines 37-39: Method `__init__` signature and setup
```python
    def __init__(
        self,
        req_to_token_pool: ReqToTokenPool,
```
**EN:** This block defines the method `__init__` on `DecodeKVCacheOffloadManager`. It introduces the parameters, setup steps, and the main entry point for this piece of decode KV-cache offload management. Definitions introduced here include `__init__`.
**CN:** 这一段定义了method `__init__`（属于 `DecodeKVCacheOffloadManager`），介绍了参数、初始化步骤，以及这部分解码 KV 缓存卸载管理逻辑的主要入口。 此处引入的定义包括 `__init__`。

### Lines 40-63: Method `__init__` logic (part 1)
```python
        token_to_kv_pool_allocator: BaseTokenToKVPoolAllocator,
        tp_group: torch.distributed.ProcessGroup,
        tree_cache: BasePrefixCache,
        server_args: ServerArgs,
    ) -> None:
        self.req_to_token_pool = req_to_token_pool
        self.token_to_kv_pool_allocator = token_to_kv_pool_allocator
        self.page_size = server_args.page_size
        self.server_args = server_args
        self.request_counter = 0
        self.tree_cache = tree_cache
        env_stride = envs.SGLANG_HICACHE_DECODE_OFFLOAD_STRIDE.get()
        if env_stride is None or env_stride <= 0:
            self.offload_stride = self.page_size
        else:
            self.offload_stride = max(
                self.page_size, (env_stride // self.page_size) * self.page_size
            )
        kv_cache = self.token_to_kv_pool_allocator.get_kvcache()
        if isinstance(kv_cache, MHATokenToKVPool):
            self.decode_host_mem_pool = MHATokenToKVPoolHost(
                kv_cache,
                server_args.hicache_ratio,
                server_args.hicache_size,
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode KV-cache offload management workflow. Notable operations include `get`, `get_kvcache`, `MHATokenToKVPoolHost`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码 KV 缓存卸载管理工作流。 值得注意的操作包括 `get`、`get_kvcache`、`MHATokenToKVPoolHost`。

### Lines 64-90: Method `__init__` logic (part 2)
```python
                self.page_size,
                server_args.hicache_mem_layout,
            )
        elif isinstance(kv_cache, MLATokenToKVPool):
            self.decode_host_mem_pool = MLATokenToKVPoolHost(
                kv_cache,
                server_args.hicache_ratio,
                server_args.hicache_size,
                self.page_size,
                server_args.hicache_mem_layout,
            )
        else:
            raise ValueError("Unsupported KV cache type for decode offload")

        self.tp_group = tp_group
        self.tp_world_size = torch.distributed.get_world_size(group=self.tp_group)

        hicache_storage_backend_extra_config = {}
        if server_args.hicache_storage_backend_extra_config:
            try:
                hicache_storage_backend_extra_config = json.loads(
                    server_args.hicache_storage_backend_extra_config
                )
            except json.JSONDecodeError as e:
                raise ValueError(
                    f"Invalid hicache storage backend extra config JSON: {e}"
                )
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode KV-cache offload management workflow. Notable operations include `MLATokenToKVPoolHost`, `ValueError`, `get_world_size`, `loads`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码 KV 缓存卸载管理工作流。 值得注意的操作包括 `MLATokenToKVPoolHost`、`ValueError`、`get_world_size`、`loads`。

### Lines 91-107: Method `__init__` logic (part 3)
```python

        self.cache_controller = HiCacheController(
            token_to_kv_pool_allocator=self.token_to_kv_pool_allocator,
            mem_pool_host=self.decode_host_mem_pool,
            page_size=self.page_size,
            tp_group=tp_group,
            io_backend=server_args.hicache_io_backend,
            load_cache_event=threading.Event(),
            storage_backend=server_args.hicache_storage_backend,
            model_name=server_args.served_model_name,
            storage_backend_extra_config=hicache_storage_backend_extra_config,
        )

        self.ongoing_offload = {}
        self.ongoing_backup = {}
        self.offloaded_state = {}
        logger.info("Enable offload kv cache for decode side")
```
**EN:** This block continues `__init__` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode KV-cache offload management workflow. Notable operations include `HiCacheController`, `Event`, `info`.
**CN:** 这一段延续了 `__init__` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码 KV 缓存卸载管理工作流。 值得注意的操作包括 `HiCacheController`、`Event`、`info`。

### Lines 109-110: Method `offload_kv_cache` signature and setup
```python
    def offload_kv_cache(self, req) -> bool:
        """Offload incremental KV cache for decode side."""
```
**EN:** This block defines the method `offload_kv_cache` on `DecodeKVCacheOffloadManager`. It introduces the parameters, setup steps, and the main entry point for this piece of decode KV-cache offload management. Definitions introduced here include `offload_kv_cache`.
**CN:** 这一段定义了method `offload_kv_cache`（属于 `DecodeKVCacheOffloadManager`），介绍了参数、初始化步骤，以及这部分解码 KV 缓存卸载管理逻辑的主要入口。 此处引入的定义包括 `offload_kv_cache`。

### Lines 111-134: Method `offload_kv_cache` logic (part 1)
```python

        if self.cache_controller is None or self.decode_host_mem_pool is None:
            return False

        if req.req_pool_idx == -1 or len(req.output_ids) == 0:
            return False

        token_indices = self.req_to_token_pool.req_to_token[req.req_pool_idx]
        if token_indices.dim() == 0 or token_indices.numel() == 0:
            return False

        # Prefill side offloads page-aligned origin_input_ids, decode side offloads the incremental part
        all_tokens = req.origin_input_ids + req.output_ids[:-1]
        prefill_offloaded_len = (
            len(req.origin_input_ids) // self.page_size * self.page_size
        )
        state = self.offloaded_state.get(req.rid)
        if state is None:
            prefill_hashes = self._compute_prefix_hash(
                req.origin_input_ids[:prefill_offloaded_len]
            )
            last_prefill_hash = (
                prefill_hashes[-1] if prefill_offloaded_len > 0 else None
            )
```
**EN:** This block continues `offload_kv_cache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode KV-cache offload management workflow. Notable operations include `dim`, `numel`, `get`, `_compute_prefix_hash`.
**CN:** 这一段延续了 `offload_kv_cache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码 KV 缓存卸载管理工作流。 值得注意的操作包括 `dim`、`numel`、`get`、`_compute_prefix_hash`。

### Lines 135-158: Method `offload_kv_cache` logic (part 2)
```python
            state = OffloadedState(
                prefill_len=prefill_offloaded_len,
                inc_len=0,
                last_hash=last_prefill_hash,
            )
            self.offloaded_state[req.rid] = state
        incremental_total = len(all_tokens) - state.prefill_len
        incremental_new = incremental_total - state.inc_len
        incremental_aligned_len = (
            incremental_new // self.offload_stride * self.offload_stride
        )

        if incremental_aligned_len == 0:
            return False

        # Extract incremental tokens and indices for the newly available chunk
        start = state.prefill_len + state.inc_len
        end = start + incremental_aligned_len
        incremental_tokens = all_tokens[start:end]
        incremental_indices = token_indices[start:end]

        # Early free prefill-offloaded GPU memory
        if state.prefill_len > 0 and state.inc_len == 0:
            self.token_to_kv_pool_allocator.free(token_indices[: state.prefill_len])
```
**EN:** This block continues `offload_kv_cache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode KV-cache offload management workflow. Notable operations include `OffloadedState`, `free`.
**CN:** 这一段延续了 `offload_kv_cache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码 KV 缓存卸载管理工作流。 值得注意的操作包括 `OffloadedState`、`free`。

### Lines 159-180: Method `offload_kv_cache` logic (part 3)
```python

        # Asynchronously offload incremental KV cache from device to host
        self.request_counter += 1
        ack_id = self.request_counter
        host_indices = self.cache_controller.write(
            device_indices=incremental_indices.long(),
            node_id=ack_id,
        )
        if host_indices is None:
            logger.error(f"Not enough host memory for request {req.rid}")
            return False

        self.ongoing_offload[ack_id] = (
            req,
            host_indices,
            incremental_tokens,
            time.time(),
            start,
            end,
        )
        state.inc_len += incremental_aligned_len
        return True
```
**EN:** This block continues `offload_kv_cache` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode KV-cache offload management workflow. Notable operations include `write`, `long`, `error`, `time`.
**CN:** 这一段延续了 `offload_kv_cache` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码 KV 缓存卸载管理工作流。 值得注意的操作包括 `write`、`long`、`error`、`time`。

### Lines 182-200: Method `check_offload_progress`
```python
    def check_offload_progress(self):
        """Check the progress of offload from device to host and backup from host to storage."""
        cc = self.cache_controller

        qsizes = torch.tensor(
            [
                len(cc.ack_write_queue),
                cc.ack_backup_queue.qsize(),
            ],
            dtype=torch.int,
        )
        if self.tp_world_size > 1:
            torch.distributed.all_reduce(
                qsizes, op=torch.distributed.ReduceOp.MIN, group=self.tp_group
            )

        n_write, n_backup = map(int, qsizes.tolist())
        self._check_offload_progress(n_write)
        self._check_backup_progress(n_backup)
```
**EN:** This block defines the method `check_offload_progress` on `DecodeKVCacheOffloadManager`. It introduces the parameters, setup steps, and the main entry point for this piece of decode KV-cache offload management. Definitions introduced here include `check_offload_progress`. Notable operations include `tensor`, `qsize`, `all_reduce`, `tolist`.
**CN:** 这一段定义了method `check_offload_progress`（属于 `DecodeKVCacheOffloadManager`），介绍了参数、初始化步骤，以及这部分解码 KV 缓存卸载管理逻辑的主要入口。 此处引入的定义包括 `check_offload_progress`。 值得注意的操作包括 `tensor`、`qsize`、`all_reduce`、`tolist`。

### Lines 202-203: Method `_check_offload_progress` signature and setup
```python
    def _check_offload_progress(self, finish_count):
        """Check the progress of offload from device to host."""
```
**EN:** This block defines the method `_check_offload_progress` on `DecodeKVCacheOffloadManager`. It introduces the parameters, setup steps, and the main entry point for this piece of decode KV-cache offload management. Definitions introduced here include `_check_offload_progress`.
**CN:** 这一段定义了method `_check_offload_progress`（属于 `DecodeKVCacheOffloadManager`），介绍了参数、初始化步骤，以及这部分解码 KV 缓存卸载管理逻辑的主要入口。 此处引入的定义包括 `_check_offload_progress`。

### Lines 204-223: Method `_check_offload_progress` logic (part 1)
```python
        while finish_count > 0:
            _, finish_event, ack_list = self.cache_controller.ack_write_queue.pop(0)
            finish_event.synchronize()
            for ack_id in ack_list:
                (
                    req,
                    host_indices,
                    incremental_tokens,
                    start_time,
                    start,
                    end,
                ) = self.ongoing_offload.pop(ack_id)

                if req.finished():
                    self._release_finished_req(req, start)
                else:
                    kv_indices = self.req_to_token_pool.req_to_token[
                        req.req_pool_idx, start:end
                    ]
                    self.token_to_kv_pool_allocator.free(kv_indices)
```
**EN:** This block continues `_check_offload_progress` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode KV-cache offload management workflow. Notable operations include `pop`, `synchronize`, `finished`, `_release_finished_req`.
**CN:** 这一段延续了 `_check_offload_progress` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码 KV 缓存卸载管理工作流。 值得注意的操作包括 `pop`、`synchronize`、`finished`、`_release_finished_req`。

### Lines 224-235: Method `_check_offload_progress` logic (part 2)
```python

                prior_hash = (
                    self.offloaded_state[req.rid].last_hash
                    if req.rid in self.offloaded_state
                    else None
                )
                last_hash = self._trigger_backup(
                    req, host_indices, incremental_tokens, start_time, prior_hash
                )
                if req.rid in self.offloaded_state:
                    self.offloaded_state[req.rid].last_hash = last_hash
            finish_count -= 1
```
**EN:** This block continues `_check_offload_progress` and carries out the operational logic of the routine. It updates local state and connects intermediate results to the surrounding decode KV-cache offload management workflow. Notable operations include `_trigger_backup`.
**CN:** 这一段延续了 `_check_offload_progress` 的实现，执行该过程的具体运行逻辑。它会更新局部状态，并把中间结果接入周围的解码 KV 缓存卸载管理工作流。 值得注意的操作包括 `_trigger_backup`。

### Lines 237-259: Method `_release_finished_req`
```python
    def _release_finished_req(self, req: Req, start_offset: int):
        kv_committed_len = req.pop_committed_kv_cache()
        start = start_offset
        end = kv_committed_len
        # Free the incremental part of the request (NSA-aware)
        kv_indices = self.req_to_token_pool.req_to_token[req.req_pool_idx, start:end]
        self.token_to_kv_pool_allocator.free(kv_indices)

        # Free over-allocated KV cache slots (e.g. from speculative decoding v2).
        # Without spec v2, start_p == end_p so this is a no-op.
        start_p, end_p = req.pop_overallocated_kv_cache()
        if self.page_size > 1:
            start_p = ceil_align(start_p, self.page_size)
        if start_p < end_p:
            overalloc_indices = self.req_to_token_pool.req_to_token[
                req.req_pool_idx, start_p:end_p
            ]
            self.token_to_kv_pool_allocator.free(overalloc_indices)

        self.req_to_token_pool.free(req)
        self.tree_cache.protected_size_ -= len(req.prefix_indices)
        if req.rid in self.offloaded_state:
            del self.offloaded_state[req.rid]
```
**EN:** This block defines the method `_release_finished_req` on `DecodeKVCacheOffloadManager`. It introduces the parameters, setup steps, and the main entry point for this piece of decode KV-cache offload management. Definitions introduced here include `_release_finished_req`. Notable operations include `pop_committed_kv_cache`, `request`, `free`, `slots`.
**CN:** 这一段定义了method `_release_finished_req`（属于 `DecodeKVCacheOffloadManager`），介绍了参数、初始化步骤，以及这部分解码 KV 缓存卸载管理逻辑的主要入口。 此处引入的定义包括 `_release_finished_req`。 值得注意的操作包括 `pop_committed_kv_cache`、`request`、`free`、`slots`。

### Lines 261-273: Method `_check_backup_progress`
```python
    def _check_backup_progress(self, finish_count):
        """Check the progress of backup from host to storage."""
        for _ in range(finish_count):
            storage_operation = self.cache_controller.ack_backup_queue.get()
            ack_id = storage_operation.id
            req_id, host_indices, start_time = self.ongoing_backup.pop(ack_id)

            # Release host memory
            self.decode_host_mem_pool.free(host_indices)

            logger.debug(
                f"Finished backup request {req_id}, free host memory, len:{len(host_indices)}, cost time:{time.time() - start_time:.2f} seconds."
            )
```
**EN:** This block defines the method `_check_backup_progress` on `DecodeKVCacheOffloadManager`. It introduces the parameters, setup steps, and the main entry point for this piece of decode KV-cache offload management. Definitions introduced here include `_check_backup_progress`. Notable operations include `get`, `pop`, `free`, `debug`.
**CN:** 这一段定义了method `_check_backup_progress`（属于 `DecodeKVCacheOffloadManager`），介绍了参数、初始化步骤，以及这部分解码 KV 缓存卸载管理逻辑的主要入口。 此处引入的定义包括 `_check_backup_progress`。 值得注意的操作包括 `get`、`pop`、`free`、`debug`。

### Lines 275-286: Method `_trigger_backup`
```python
    def _trigger_backup(
        self, req, host_indices, incremental_tokens, start_time, prior_hash
    ):
        """Trigger async backup from host to storage."""
        page_hashes = self._compute_prefix_hash(incremental_tokens, prior_hash)
        ack_id = self.cache_controller.write_storage(
            host_indices,
            incremental_tokens,
            hash_value=page_hashes,
        )
        self.ongoing_backup[ack_id] = (req.rid, host_indices, start_time)
        return page_hashes[-1] if len(page_hashes) > 0 else prior_hash
```
**EN:** This block defines the method `_trigger_backup` on `DecodeKVCacheOffloadManager`. It introduces the parameters, setup steps, and the main entry point for this piece of decode KV-cache offload management. Definitions introduced here include `_trigger_backup`. Notable operations include `_compute_prefix_hash`, `write_storage`.
**CN:** 这一段定义了method `_trigger_backup`（属于 `DecodeKVCacheOffloadManager`），介绍了参数、初始化步骤，以及这部分解码 KV 缓存卸载管理逻辑的主要入口。 此处引入的定义包括 `_trigger_backup`。 值得注意的操作包括 `_compute_prefix_hash`、`write_storage`。

### Lines 288-295: Method `_compute_prefix_hash`
```python
    def _compute_prefix_hash(self, tokens, prior_hash=""):
        page_hashes = []
        last_hash = prior_hash
        for offset in range(0, len(tokens), self.page_size):
            page_tokens = tokens[offset : offset + self.page_size]
            last_hash = self.cache_controller.get_hash_str(page_tokens, last_hash)
            page_hashes.append(last_hash)
        return page_hashes
```
**EN:** This block defines the method `_compute_prefix_hash` on `DecodeKVCacheOffloadManager`. It introduces the parameters, setup steps, and the main entry point for this piece of decode KV-cache offload management. Definitions introduced here include `_compute_prefix_hash`. Notable operations include `get_hash_str`, `append`.
**CN:** 这一段定义了method `_compute_prefix_hash`（属于 `DecodeKVCacheOffloadManager`），介绍了参数、初始化步骤，以及这部分解码 KV 缓存卸载管理逻辑的主要入口。 此处引入的定义包括 `_compute_prefix_hash`。 值得注意的操作包括 `get_hash_str`、`append`。

### Lines 297-317: Method `finalize_release_on_finish`
```python
    def finalize_release_on_finish(self, req: Req):
        """Free any remaining tail KV that was not offloaded due to non-aligned length."""
        if req.req_pool_idx == -1:
            return
        state = self.offloaded_state.get(req.rid)
        if state is None:
            prefill_len = len(req.origin_input_ids) // self.page_size * self.page_size
            inc_len = 0
        else:
            prefill_len = state.prefill_len
            inc_len = state.inc_len
        # If no incremental offload ever happened, the prefill-aligned part was never freed.
        # Free the prefill portion on request finish to avoid leaks.
        if prefill_len > 0 and inc_len == 0:
            token_indices = self.req_to_token_pool.req_to_token[req.req_pool_idx]
            self.token_to_kv_pool_allocator.free(token_indices[:prefill_len])
            logger.info(
                f"Finalize release: freed prefill-aligned KV for req {req.rid}, len:{prefill_len}"
            )
        start_offset = prefill_len + inc_len
        self._release_finished_req(req, start_offset)
```
**EN:** This block defines the method `finalize_release_on_finish` on `DecodeKVCacheOffloadManager`. It introduces the parameters, setup steps, and the main entry point for this piece of decode KV-cache offload management. Definitions introduced here include `finalize_release_on_finish`. Notable operations include `get`, `free`, `info`, `_release_finished_req`.
**CN:** 这一段定义了method `finalize_release_on_finish`（属于 `DecodeKVCacheOffloadManager`），介绍了参数、初始化步骤，以及这部分解码 KV 缓存卸载管理逻辑的主要入口。 此处引入的定义包括 `finalize_release_on_finish`。 值得注意的操作包括 `get`、`free`、`info`、`_release_finished_req`。

## Key Concepts / 关键概念
- `DecodeKVCacheOffloadManager`: Class that encapsulates decode kvcache offload manager behavior in this module. / `DecodeKVCacheOffloadManager`：封装与“解码kvcache卸载管理器”相关行为的类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `json`, `logging`, `threading`, `time`, `typing`
- **External packages / 外部依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.disaggregation.kv_events`, `sglang.srt.environ`, `sglang.srt.managers.cache_controller`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.base_prefix_cache`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.mem_cache.memory_pool_host`, `sglang.srt.server_args`, `sglang.srt.utils.common`, `sglang.srt.managers.schedule_batch`
