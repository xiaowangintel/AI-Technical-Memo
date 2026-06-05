# worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/mooncake/store/worker.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Worker-side logic for MooncakeStoreConnector / 实现基于 Mooncake 的 KV 传输连接器、存储层或运行时辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Module overview / 模块概览
```python
"""Worker-side logic for MooncakeStoreConnector.

Includes the store worker, transfer threads, lookup server,
and MooncakeDistributedStore integration.
"""
```
**EN:** The opening docstring defines the module scope and high-level contract. It highlights: Worker-side logic for MooncakeStoreConnector.
**CN:** 开头的文档字符串给出了模块范围与高层契约。 它重点概括了文件的职责、参与方以及主要接口。

### Imports and setup / 导入与初始化
```python
import json
import os
import queue
import socket
import threading
from collections import defaultdict
from dataclasses import dataclass
from typing import Any, Literal

import regex as re
import torch
import zmq

import vllm.envs as envs
from vllm.config import VllmConfig
from vllm.distributed import (
    get_dcp_group,
    get_pcp_group,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from vllm.distributed.kv_events import BlockStored
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake import rdma_utils
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_utils import (
    get_mooncake_dp_engine_index,
)
from vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.data import (
    ChunkedTokenDatabase,
    KeyMetadata,
    MooncakeStoreConnectorMetadata,
    ReqMeta,
)
from vllm.logger import init_logger
from vllm.utils.network_utils import get_ip, make_zmq_socket
from vllm.v1.core.kv_cache_utils import BlockHash, maybe_convert_block_hash
from vllm.v1.serial_utils import MsgpackDecoder, MsgpackEncoder
```
**EN:** This block imports `json`, `os`, `queue`, `socket`, `threading`, `collections` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `json`, `os`, `queue`, `socket`, `threading`, `collections`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)

DEFAULT_GLOBAL_SEGMENT_SIZE = 4 * 1024 * 1024 * 1024  # 4 GiB
DEFAULT_LOCAL_BUFFER_SIZE = 4 * 1024 * 1024 * 1024  # 4 GiB

MOONCAKE_NO_AVAILABLE_HANDLE = -200

# Mirrors FileStorageConfig::local_buffer_size in Mooncake C++.
DEFAULT_MOONCAKE_DISK_STAGING_BUFFER_BYTES = 1280 * 1024 * 1024

# Mirrors DirectIO alignment in Mooncake's AllocateBatch.
_DIRECT_IO_ALIGNMENT = 4096
_DIRECT_IO_PADDING_BYTES = 2 * _DIRECT_IO_ALIGNMENT


MooncakeMode = Literal["embedded", "standalone-store"]
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, `DEFAULT_GLOBAL_SEGMENT_SIZE`, `DEFAULT_LOCAL_BUFFER_SIZE`, `MOONCAKE_NO_AVAILABLE_HANDLE`, `DEFAULT_MOONCAKE_DISK_STAGING_BUFFER_BYTES`, `_DIRECT_IO_ALIGNMENT`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`, `DEFAULT_GLOBAL_SEGMENT_SIZE`, `DEFAULT_LOCAL_BUFFER_SIZE`, `MOONCAKE_NO_AVAILABLE_HANDLE`, `DEFAULT_MOONCAKE_DISK_STAGING_BUFFER_BYTES`, `_DIRECT_IO_ALIGNMENT`，供后续代码复用。

### Class `MooncakeStoreConfig` / 类 `MooncakeStoreConfig`
```python
@dataclass
class MooncakeStoreConfig:
    """Configuration for MooncakeDistributedStore.

    ``mode`` selects the topology: ``embedded`` (each rank contributes
    ``global_segment_size`` in-process) or ``standalone-store`` (rank
    contributes 0; an external ``mooncake_client`` process owns the pool
    and the SSD tier).
    """

    metadata_server: str
    master_server_address: str
    protocol: str
    device_name: str
    mode: MooncakeMode = "embedded"
    global_segment_size: int = DEFAULT_GLOBAL_SEGMENT_SIZE
    local_buffer_size: int = DEFAULT_LOCAL_BUFFER_SIZE
    enable_offload: bool = False

    def __post_init__(self) -> None:
        if self.mode not in ("embedded", "standalone-store"):
            raise ValueError(f"unknown Mooncake mode: {self.mode!r}")
        if self.local_buffer_size <= 0:
            raise ValueError("local_buffer_size must be > 0")
        if self.mode == "embedded" and self.global_segment_size == 0:
            raise ValueError("embedded mode requires global_segment_size > 0")
        if self.mode == "standalone-store" and self.global_segment_size != 0:
            raise ValueError("standalone-store mode requires global_segment_size == 0")

    @staticmethod
    def from_file(file_path: str) -> "MooncakeStoreConfig":
        with open(file_path) as file:
            config = json.load(file)
        return MooncakeStoreConfig(
            metadata_server=config.get("metadata_server", ""),
            master_server_address=config.get("master_server_address", ""),
            protocol=config.get("protocol", "rdma"),
            device_name=config.get("device_name", ""),
            mode=config.get("mode", "embedded"),
            global_segment_size=_parse_size(
                config.get("global_segment_size", DEFAULT_GLOBAL_SEGMENT_SIZE)
            ),
            local_buffer_size=_parse_size(
                config.get("local_buffer_size", DEFAULT_LOCAL_BUFFER_SIZE)
            ),
            enable_offload=bool(config.get("enable_offload", False)),
        )

    @staticmethod
    def load_from_env() -> "MooncakeStoreConfig":
        config_path = os.getenv("MOONCAKE_CONFIG_PATH")
        if not config_path:
            raise ValueError(
                "The environment variable 'MOONCAKE_CONFIG_PATH' is not set."
            )
        return MooncakeStoreConfig.from_file(config_path)
```
**EN:** Declares `MooncakeStoreConfig`, a dataclass. It packages structured data fields such as `metadata_server`, `master_server_address`, `protocol`, `device_name`, `mode`. The docstring summarizes its role as: Configuration for MooncakeDistributedStore.
**CN:** 声明 `MooncakeStoreConfig`，它是一个数据类。 它封装了 `metadata_server`, `master_server_address`, `protocol`, `device_name`, `mode` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Function `_parse_size` / 函数 `_parse_size`
```python
def _parse_size(value: Any) -> int:
    """Parse storage size strings with units: GB, MB, KB, B."""
    if isinstance(value, int):
        return value
    if not isinstance(value, str):
        try:
            return int(value)
        except (TypeError, ValueError) as e:
            raise TypeError(f"Unsupported type for size: {type(value)}") from e

    cleaned = value.strip().lower()
    if not cleaned:
        raise ValueError("Size cannot be empty.")

    unit_multipliers = {
        "gb": 1024**3,
        "mb": 1024**2,
        "kb": 1024,
        "b": 1,
    }
    match = re.match(r"^\s*([\d.]+)\s*(gb|mb|kb|b)?\s*$", cleaned)
    if not match:
        raise ValueError(f"Invalid format: '{value}'")

    number_str = match.group(1)
    unit = match.group(2) or "b"
    multiplier = unit_multipliers[unit]

    try:
        numeric_value = float(number_str)
    except ValueError as exc:
        raise ValueError(f"Invalid numeric value '{number_str}' in: '{value}'") from exc
    return int(numeric_value * multiplier)
```
**EN:** `_parse_size` implements a focused helper routine for this module. The docstring frames it as: Parse storage size strings with units: GB, MB, KB, B. It primarily works with arguments like `value`. Key calls include `isinstance`, `value.strip().lower`, `re.match`.
**CN:** `_parse_size` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `value` 这样的参数。 关键调用包括 `isinstance`, `value.strip().lower`, `re.match`。

### Function `_align_up` / 函数 `_align_up`
```python
def _align_up(value: int, alignment: int) -> int:
    return ((value + alignment - 1) // alignment) * alignment
```
**EN:** `_align_up` implements a focused helper routine for this module. It primarily works with arguments like `value`, `alignment`.
**CN:** `_align_up` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `value`, `alignment` 这样的参数。

### Function `_estimate_disk_offload_staging_bytes` / 函数 `_estimate_disk_offload_staging_bytes`
```python
def _estimate_disk_offload_staging_bytes(size_list: list[int]) -> int:
    data_size = sum(size_list)
    return _align_up(data_size, _DIRECT_IO_ALIGNMENT) + _DIRECT_IO_PADDING_BYTES
```
**EN:** `_estimate_disk_offload_staging_bytes` implements a focused helper routine for this module. It primarily works with arguments like `size_list`. Key calls include `sum`, `_align_up`.
**CN:** `_estimate_disk_offload_staging_bytes` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `size_list` 这样的参数。 关键调用包括 `sum`, `_align_up`。

### Function `_get_usable_disk_offload_buffer_budget_bytes` / 函数 `_get_usable_disk_offload_buffer_budget_bytes`
```python
def _get_usable_disk_offload_buffer_budget_bytes(raw_budget_bytes: int) -> int:
    return max(1, int(raw_budget_bytes * envs.VLLM_MOONCAKE_DISK_STAGING_USABLE_RATIO))
```
**EN:** `_get_usable_disk_offload_buffer_budget_bytes` is a thin wrapper around `max`, exposing that operation through a module-level helper. It primarily works with arguments like `raw_budget_bytes`. Key calls include `max`, `int`.
**CN:** `_get_usable_disk_offload_buffer_budget_bytes` 是对 `max` 的轻量封装，通过模块级辅助函数暴露该操作。 它主要处理诸如 `raw_budget_bytes` 这样的参数。 关键调用包括 `max`, `int`。

### Function `_split_disk_offload_load_batches` / 函数 `_split_disk_offload_load_batches`
```python
def _split_disk_offload_load_batches(
    keys: list[str],
    addrs: list[list[int]],
    sizes: list[list[int]],
    usable_budget_bytes: int,
    raw_budget_bytes: int,
) -> tuple[list[tuple[list[str], list[list[int]], list[list[int]]]], str | None]:
    """Split a GET into sub-batches that fit the owner's staging buffer.

    ``addrs[i]`` / ``sizes[i]`` are scatter-gather lists (K/V or multi-layer
    segments) for key ``i``. ``usable_budget_bytes`` caps a multi-key batch;
    ``raw_budget_bytes`` is the hard per-key cap.

    Returns ``(batches, oversize_key)``. Aborts with ``([], key)`` if any
    single key exceeds ``raw_budget_bytes``; otherwise ``oversize_key`` is
    ``None``.
    """
    batches: list[tuple[list[str], list[list[int]], list[list[int]]]] = []
    batch_keys: list[str] = []
    batch_addrs: list[list[int]] = []
    batch_sizes: list[list[int]] = []
    batch_bytes = 0

    for key, addr, size in zip(keys, addrs, sizes, strict=True):
        key_bytes = _estimate_disk_offload_staging_bytes(size)
        if key_bytes > raw_budget_bytes:
            return [], key
        if key_bytes > usable_budget_bytes:
            if batch_keys:
                batches.append((batch_keys, batch_addrs, batch_sizes))
                batch_keys, batch_addrs, batch_sizes = [], [], []
                batch_bytes = 0
            batches.append(([key], [addr], [size]))
            continue
        if batch_keys and batch_bytes + key_bytes > usable_budget_bytes:
            batches.append((batch_keys, batch_addrs, batch_sizes))
            batch_keys, batch_addrs, batch_sizes = [], [], []
            batch_bytes = 0
        batch_keys.append(key)
        batch_addrs.append(addr)
        batch_sizes.append(size)
        batch_bytes += key_bytes

    if batch_keys:
        batches.append((batch_keys, batch_addrs, batch_sizes))
    return batches, None
```
**EN:** `_split_disk_offload_load_batches` implements a focused helper routine for this module. The docstring frames it as: Split a GET into sub-batches that fit the owner's staging buffer. It primarily works with arguments like `keys`, `addrs`, `sizes`, `usable_budget_bytes`. Key calls include `zip`, `_estimate_disk_offload_staging_bytes`, `batch_keys.append`.
**CN:** `_split_disk_offload_load_batches` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `keys`, `addrs`, `sizes`, `usable_budget_bytes` 这样的参数。 关键调用包括 `zip`, `_estimate_disk_offload_staging_bytes`, `batch_keys.append`。

### Function `_call_replica_predicate` / 函数 `_call_replica_predicate`
```python
def _call_replica_predicate(replica_desc: Any, method_name: str) -> bool:
    method = getattr(replica_desc, method_name, None)
    if method is None:
        return False
    try:
        return bool(method())
    except Exception:
        return False
```
**EN:** `_call_replica_predicate` implements a focused helper routine for this module. It primarily works with arguments like `replica_desc`, `method_name`. Key calls include `getattr`, `bool`, `method`.
**CN:** `_call_replica_predicate` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `replica_desc`, `method_name` 这样的参数。 关键调用包括 `getattr`, `bool`, `method`。

### Function `_classify_replica_tier` / 函数 `_classify_replica_tier`
```python
def _classify_replica_tier(replica_descs: Any) -> str:
    if not replica_descs:
        return "unknown"
    try:
        replica_desc = replica_descs[0]
    except (IndexError, KeyError, TypeError):
        return "unknown"

    if _call_replica_predicate(replica_desc, "is_memory_replica"):
        return "memory"
    if _call_replica_predicate(
        replica_desc, "is_disk_replica"
    ) or _call_replica_predicate(replica_desc, "is_local_disk_replica"):
        return "disk"
    return "unknown"
```
**EN:** `_classify_replica_tier` implements a focused helper routine for this module. It primarily works with arguments like `replica_descs`. Key calls include `_call_replica_predicate`.
**CN:** `_classify_replica_tier` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `replica_descs` 这样的参数。 关键调用包括 `_call_replica_predicate`。

### Function `_get_replica_tiers_by_key` / 函数 `_get_replica_tiers_by_key`
```python
def _get_replica_tiers_by_key(store: Any, keys: list[str]) -> dict[str, str]:
    tiers_by_key = {key: "unknown" for key in keys}
    try:
        replica_descs_by_key = store.batch_get_replica_desc(keys)
    except Exception as e:
        logger.warning(
            "Failed to get Mooncake replica descriptors for tier logging "
            "(batch_keys=%d, error=%s); marking tiers unknown",
            len(keys),
            e,
        )
        return tiers_by_key

    for key in keys:
        if hasattr(replica_descs_by_key, "get"):
            replica_descs = replica_descs_by_key.get(key)
        else:
            try:
                replica_descs = replica_descs_by_key[key]
            except (KeyError, TypeError):
                replica_descs = None
        tiers_by_key[key] = _classify_replica_tier(replica_descs)
    return tiers_by_key
```
**EN:** `_get_replica_tiers_by_key` implements a focused helper routine for this module. It primarily works with arguments like `store`, `keys`. Key calls include `store.batch_get_replica_desc`, `hasattr`, `_classify_replica_tier`.
**CN:** `_get_replica_tiers_by_key` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `store`, `keys` 这样的参数。 关键调用包括 `store.batch_get_replica_desc`, `hasattr`, `_classify_replica_tier`。

### Function `_log_mooncake_load_tier_summary` / 函数 `_log_mooncake_load_tier_summary`
```python
def _log_mooncake_load_tier_summary(
    req_id: str,
    batch_keys: list[str],
    load_results: list[int],
    tiers_by_key: dict[str, str],
) -> None:
    tier_counts = {"memory": 0, "disk": 0, "unknown": 0}
    bytes_by_tier = {"memory": 0, "disk": 0, "unknown": 0}
    success_keys = 0
    failed_keys = 0

    for index, key in enumerate(batch_keys):
        tier = tiers_by_key.get(key, "unknown")
        if tier not in tier_counts:
            tier = "unknown"
        tier_counts[tier] += 1

        value = load_results[index] if index < len(load_results) else -1
        if value >= 0:
            success_keys += 1
            bytes_by_tier[tier] += int(value)
        else:
            failed_keys += 1

    logger.info(
        "Mooncake load tier summary: req_id=%s batch_keys=%d "
        "memory_keys=%d disk_keys=%d unknown_keys=%d "
        "success_keys=%d failed_keys=%d bytes_by_tier=%s",
        req_id,
        len(batch_keys),
        tier_counts["memory"],
        tier_counts["disk"],
        tier_counts["unknown"],
        success_keys,
        failed_keys,
        bytes_by_tier,
    )
```
**EN:** `_log_mooncake_load_tier_summary` implements a focused helper routine for this module. It primarily works with arguments like `req_id`, `batch_keys`, `load_results`, `tiers_by_key`. Key calls include `enumerate`, `logger.info`, `tiers_by_key.get`.
**CN:** `_log_mooncake_load_tier_summary` 实现了一个面向当前模块的辅助例程。 它主要处理诸如 `req_id`, `batch_keys`, `load_results`, `tiers_by_key` 这样的参数。 关键调用包括 `enumerate`, `logger.info`, `tiers_by_key.get`。

### Class `KVTransferThread` / 类 `KVTransferThread`
```python
class KVTransferThread(threading.Thread):
    """Base class for async KV cache transfer threads."""

    def __init__(
        self,
        store: Any,
        token_database: ChunkedTokenDatabase,
        block_size: int,
        tp_rank: int,
        ready_event: threading.Event,
        name: str,
    ):
        super().__init__(daemon=True, name=name)
        self.store = store
        self.ready_event = ready_event
        self.block_size = block_size
        self.tp_rank = tp_rank
        self.token_database = token_database
        self.done_task_lock = threading.Lock()
        self.request_queue: queue.Queue[Any] = queue.Queue()
        self.finished_requests: set[str] = set()
        self.kv_event_lock = threading.Lock()
        self.kv_events: list[BlockStored] = []

    def add_request(self, request: ReqMeta) -> None:
        self.request_queue.put(request)

    def get_and_clear_finished_requests(self) -> set[str]:
        with self.done_task_lock:
            finished = self.finished_requests.copy()
            self.finished_requests.clear()
        return finished

    def set_finished_request(self, req_id: str):
        with self.done_task_lock:
            self.finished_requests.add(req_id)

    def run(self):
        self.ready_event.set()
        while True:
            try:
                request_data = self.request_queue.get()
                if request_data is None:
                    logger.warning("Received a None request!")
                    self.request_queue.task_done()
# ... truncated for analysis ...

    def update_kv_event(self, events: list[BlockStored]):
        with self.kv_event_lock:
            self.kv_events.extend(events)

    def get_kv_events(self) -> list[BlockStored]:
        with self.kv_event_lock:
            events = self.kv_events.copy()
            self.kv_events.clear()
        return events
```
**EN:** Declares `KVTransferThread`, a class derived from `threading.Thread`. Key methods include `__init__`, `add_request`, `get_and_clear_finished_requests`, `set_finished_request`, `run`. The docstring summarizes its role as: Base class for async KV cache transfer threads.
**CN:** 声明 `KVTransferThread`，它是一个类，继承自 `threading.Thread`。 关键方法包括 `__init__`, `add_request`, `get_and_clear_finished_requests`, `set_finished_request`, `run`。 文档字符串概括了它在整体流程中的职责。

### Class `KVCacheStoreSendingThread` / 类 `KVCacheStoreSendingThread`
```python
class KVCacheStoreSendingThread(KVTransferThread):
    """Background thread for storing KV cache blocks to the store."""

    def __init__(
        self,
        store: Any,
        token_database: ChunkedTokenDatabase,
        block_size: int,
        tp_rank: int,
        put_step: int,
        kv_role: str,
        ready_event: threading.Event,
        enable_kv_event: bool = False,
        replicate_config: Any = None,
    ):
        super().__init__(
            store,
            token_database,
            block_size,
            tp_rank,
            ready_event,
            name="KVCacheStoreSendingThread",
        )
        self.put_step = put_step
        self.kv_role = kv_role
        self.stored_requests: defaultdict[str, int] = defaultdict(int)
        self.enable_kv_event = enable_kv_event
        # Caller always passes a non-None ReplicateConfig — see
        # MooncakeStoreWorker.__init__ where store_replicate_config is built.
        self.replicate_config = replicate_config

        # Pause store requests when CPU/disk offloading is under pressure.
        self._store_pressure_active = False
        self._skip_store_requests: set[str] = set()

    def add_stored_request(self, req_id: str):
        with self.done_task_lock:
            self.stored_requests[req_id] += 1

    def dec_stored_request(self, req_id: str):
        with self.done_task_lock:
            if req_id in self.stored_requests:
                self.stored_requests[req_id] -= 1

    def delete_finished_stored_request(self, req_id: str):
# ... truncated for analysis ...
                    "successful store batch"
                )
        except Exception as e:
            logger.error("Failed to put key %s, error: %s", keys, e)

        if self.enable_kv_event and stored_events:
            self.update_kv_event(stored_events)

        self.dec_stored_request(req_id)
        self.request_queue.task_done()
```
**EN:** Declares `KVCacheStoreSendingThread`, a class derived from `KVTransferThread`. Key methods include `__init__`, `add_stored_request`, `dec_stored_request`, `delete_finished_stored_request`, `_should_skip_request`. The docstring summarizes its role as: Background thread for storing KV cache blocks to the store.
**CN:** 声明 `KVCacheStoreSendingThread`，它是一个类，继承自 `KVTransferThread`。 关键方法包括 `__init__`, `add_stored_request`, `dec_stored_request`, `delete_finished_stored_request`, `_should_skip_request`。 文档字符串概括了它在整体流程中的职责。

### Class `KVCacheStoreRecvingThread` / 类 `KVCacheStoreRecvingThread`
```python
class KVCacheStoreRecvingThread(KVTransferThread):
    """Background thread for loading KV cache blocks from the store."""

    def __init__(
        self,
        store: Any,
        token_database: ChunkedTokenDatabase,
        block_size: int,
        tp_rank: int,
        ready_event: threading.Event,
        disk_offload_buffer_budget_bytes: int | None = None,
    ):
        super().__init__(
            store,
            token_database,
            block_size,
            tp_rank,
            ready_event,
            name="KVCacheStoreRecvingThread",
        )
        self.disk_offload_buffer_budget_bytes = disk_offload_buffer_budget_bytes
        self.usable_disk_offload_buffer_budget_bytes = (
            None
            if disk_offload_buffer_budget_bytes is None
            else _get_usable_disk_offload_buffer_budget_bytes(
                disk_offload_buffer_budget_bytes
            )
        )

    def _handle_request(self, req_meta: ReqMeta):
        token_len = req_meta.load_spec.token_len  # type: ignore[union-attr]
        req_id = req_meta.req_id
        mask_num = (
            req_meta.load_spec.vllm_cached_tokens  # type: ignore[union-attr]
            // self.block_size
            * self.block_size
        )

        addr_list = []
        size_list = []
        key_list = []
        for start, end, key in self.token_database.process_tokens(
            token_len, req_meta.block_hashes, mask_num
        ):
            addr, size, _ = self.token_database.prepare_value(
# ... truncated for analysis ...
                    break
        except Exception as e:
            logger.warning(
                "Failed to get Mooncake sub-batch %s, error: %s",
                current_batch_keys[:3],
                e,
            )

        self.set_finished_request(req_id)
        self.request_queue.task_done()
```
**EN:** Declares `KVCacheStoreRecvingThread`, a class derived from `KVTransferThread`. Key methods include `__init__`, `_handle_request`. The docstring summarizes its role as: Background thread for loading KV cache blocks from the store.
**CN:** 声明 `KVCacheStoreRecvingThread`，它是一个类，继承自 `KVTransferThread`。 关键方法包括 `__init__`, `_handle_request`。 文档字符串概括了它在整体流程中的职责。

### Class `MooncakeStoreWorker` / 类 `MooncakeStoreWorker`
```python
class MooncakeStoreWorker:
    """Worker-side component for MooncakeStoreConnector."""

    def __init__(self, vllm_config: VllmConfig):
        try:
            from mooncake.store import (  # type: ignore
                MooncakeDistributedStore,
                ReplicateConfig,
            )
        except ImportError as e:
            raise ImportError(
                "Please install mooncake by following the instructions at "
                "https://github.com/kvcache-ai/Mooncake/blob/main/doc/"
                "en/build.md to run vLLM with MooncakeStoreConnector."
            ) from e

        model_config = vllm_config.model_config
        parallel_config = vllm_config.parallel_config

        self.dp_rank = get_mooncake_dp_engine_index(parallel_config)
        self.tp_rank = get_tensor_model_parallel_rank()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.pp_size = parallel_config.pipeline_parallel_size
        self.pp_rank = (parallel_config.rank // self.tp_size) % self.pp_size

        self.pcp_size = get_pcp_group().world_size
        self.pcp_rank = get_pcp_group().rank_in_group if self.pcp_size > 1 else 0
        self.dcp_size = get_dcp_group().world_size
        self.dcp_rank = get_dcp_group().rank_in_group if self.dcp_size > 1 else 0

        assert vllm_config.kv_transfer_config is not None
        self.kv_role = vllm_config.kv_transfer_config.kv_role
        self.load_async = vllm_config.kv_transfer_config.kv_connector_extra_config.get(
            "load_async", True
        )
        self.cache_config = vllm_config.cache_config
        self.original_block_size = self.cache_config.block_size
        self.block_size = self.cache_config.block_size
        if self.pcp_size > 1:
            self.block_size *= self.pcp_size
        if self.dcp_size > 1:
            self.block_size *= self.dcp_size
        self.num_layers = model_config.get_num_layers(parallel_config)

        self.use_mla = False
# ... truncated for analysis ...
    ) -> int:
        try:
            return min(idx for row in arr for idx, val in enumerate(row) if val != 1)
        except ValueError:
            return -1

    def get_kv_events(self) -> list[BlockStored]:
        if self.enable_kv_events and self.kv_send_thread is not None:
            return self.kv_send_thread.get_kv_events()
        return []
```
**EN:** Declares `MooncakeStoreWorker`, a class. Key methods include `__init__`, `register_cross_layers_kv_caches`, `register_kv_caches`, `start_load_kv`, `wait_for_save`. The docstring summarizes its role as: Worker-side component for MooncakeStoreConnector.
**CN:** 声明 `MooncakeStoreWorker`，它是一个类。 关键方法包括 `__init__`, `register_cross_layers_kv_caches`, `register_kv_caches`, `start_load_kv`, `wait_for_save`。 文档字符串概括了它在整体流程中的职责。

### Class `LookupKeyServer` / 类 `LookupKeyServer`
```python
class LookupKeyServer:
    """ZMQ server on worker rank 0 for handling prefix lookup queries."""

    def __init__(
        self,
        store_worker: MooncakeStoreWorker,
        vllm_config: VllmConfig,
    ):
        self.decoder = MsgpackDecoder()
        self.ctx = zmq.Context()  # type: ignore[attr-defined]
        socket_path = get_zmq_rpc_path_lookup(vllm_config)
        self._ipc_path = socket_path.removeprefix("ipc://")
        if os.path.exists(self._ipc_path):
            os.unlink(self._ipc_path)
        self.socket = make_zmq_socket(
            self.ctx,
            socket_path,
            zmq.REP,  # type: ignore[attr-defined]
            bind=True,
        )

        self.store_worker = store_worker
        self.running = True

        def process_request():
            while self.running:
                all_frames = self.socket.recv_multipart(copy=False)
                token_len = int.from_bytes(all_frames[0], byteorder="big")
                hash_frames = all_frames[1:]
                hashes_str = self.decoder.decode(hash_frames)
                result = self.store_worker.lookup(token_len, hashes_str)
                response = result.to_bytes(4, "big")
                self.socket.send(response)

        self.thread = threading.Thread(target=process_request, daemon=True)
        self.thread.start()

    def close(self):
        self.socket.close(linger=0)
        if os.path.exists(self._ipc_path):
            os.unlink(self._ipc_path)
```
**EN:** Declares `LookupKeyServer`, a class. Key methods include `__init__`, `close`. The docstring summarizes its role as: ZMQ server on worker rank 0 for handling prefix lookup queries.
**CN:** 声明 `LookupKeyServer`，它是一个类。 关键方法包括 `__init__`, `close`。 文档字符串概括了它在整体流程中的职责。

### Class `LookupKeyClient` / 类 `LookupKeyClient`
```python
class LookupKeyClient:
    """ZMQ client for querying prefix cache hits from worker."""

    def __init__(self, vllm_config: VllmConfig):
        self.encoder = MsgpackEncoder()
        self.ctx = zmq.Context()  # type: ignore[attr-defined]
        socket_path = get_zmq_rpc_path_lookup(vllm_config)
        self.socket = make_zmq_socket(
            self.ctx,
            socket_path,
            zmq.REQ,  # type: ignore[attr-defined]
            bind=False,
        )

    def lookup(self, token_len: int, block_hashes: list[BlockHash]) -> int:
        hash_strs = [h.hex() for h in block_hashes]
        hash_frames = self.encoder.encode(hash_strs)
        token_len_bytes = token_len.to_bytes(4, byteorder="big")
        all_frames = [token_len_bytes] + list(hash_frames)
        self.socket.send_multipart(all_frames, copy=False)
        resp = self.socket.recv()
        result = int.from_bytes(resp, "big")
        return result

    def close(self):
        self.socket.close(linger=0)
```
**EN:** Declares `LookupKeyClient`, a class. Key methods include `__init__`, `lookup`, `close`. The docstring summarizes its role as: ZMQ client for querying prefix cache hits from worker.
**CN:** 声明 `LookupKeyClient`，它是一个类。 关键方法包括 `__init__`, `lookup`, `close`。 文档字符串概括了它在整体流程中的职责。

### Function `get_zmq_rpc_path_lookup` / 函数 `get_zmq_rpc_path_lookup`
```python
def get_zmq_rpc_path_lookup(vllm_config: VllmConfig) -> str:
    """Construct IPC path for ZMQ lookup socket."""
    assert vllm_config.kv_transfer_config is not None
    dp_rank = get_mooncake_dp_engine_index(vllm_config.parallel_config)
    base_url = envs.VLLM_RPC_BASE_PATH
    rpc_port = 0
    hostname = socket.gethostname()
    extra_config = vllm_config.kv_transfer_config.kv_connector_extra_config
    if "lookup_rpc_port" in extra_config:
        rpc_port = extra_config["lookup_rpc_port"]
    logger.debug("Base URL: %s, RPC Port: %s", base_url, rpc_port)
    return (
        f"ipc://{base_url}/lookup_rpc_port_{rpc_port}_host_{hostname}_dp_rank{dp_rank}"
    )
```
**EN:** `get_zmq_rpc_path_lookup` retrieves state or computed results for this module. The docstring frames it as: Construct IPC path for ZMQ lookup socket. It primarily works with arguments like `vllm_config`. Key calls include `get_mooncake_dp_engine_index`, `socket.gethostname`, `logger.debug`.
**CN:** `get_zmq_rpc_path_lookup` 负责获取状态或计算结果。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `vllm_config` 这样的参数。 关键调用包括 `get_mooncake_dp_engine_index`, `socket.gethostname`, `logger.debug`。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `MooncakeStoreConfig`: dataclass interface or data carrier / `MooncakeStoreConfig`：数据类接口或数据载体
- `_parse_size`: module-level helper or API entry / `_parse_size`：模块级辅助函数或 API 入口
- `_align_up`: module-level helper or API entry / `_align_up`：模块级辅助函数或 API 入口
- `_estimate_disk_offload_staging_bytes`: module-level helper or API entry / `_estimate_disk_offload_staging_bytes`：模块级辅助函数或 API 入口
- `_get_usable_disk_offload_buffer_budget_bytes`: module-level helper or API entry / `_get_usable_disk_offload_buffer_budget_bytes`：模块级辅助函数或 API 入口

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `queue`, `socket`, `threading`, `collections`, `dataclasses`, `typing`
- **Third-party / 第三方**: `regex`, `torch`, `zmq`, `mooncake.store`
- **Internal modules / 内部模块**: `vllm.envs`, `vllm.config`, `vllm.distributed`, `vllm.distributed.kv_events`, `vllm.distributed.kv_transfer.kv_connector.v1.mooncake`, `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.mooncake_utils`, `vllm.distributed.kv_transfer.kv_connector.v1.mooncake.store.data`, `vllm.logger`, `vllm.utils.network_utils`, `vllm.v1.core.kv_cache_utils`, `vllm.v1.serial_utils`
