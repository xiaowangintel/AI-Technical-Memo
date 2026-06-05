# data.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/mooncake/store/data.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Data classes for MooncakeStoreConnector / 实现基于 Mooncake 的 KV 传输连接器、存储层或运行时辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Module overview / 模块概览
```python
"""Data classes for MooncakeStoreConnector."""
```
**EN:** The opening docstring defines the module scope and high-level contract. It highlights: Data classes for MooncakeStoreConnector.
**CN:** 开头的文档字符串给出了模块范围与高层契约。 它重点概括了文件的职责、参与方以及主要接口。

### Imports and setup / 导入与初始化
```python
from collections.abc import Iterable
from dataclasses import dataclass

import torch

from vllm.distributed.kv_transfer.kv_connector.v1.base import (
    KVConnectorMetadata,
)
from vllm.logger import init_logger
from vllm.utils.math_utils import cdiv
from vllm.v1.core.kv_cache_utils import BlockHash
```
**EN:** This block imports `collections.abc`, `dataclasses`, `torch`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.logger`, `vllm.utils.math_utils` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `collections.abc`, `dataclasses`, `torch`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.logger`, `vllm.utils.math_utils`，为后续实现准备运行时、类型与辅助 API。

### Module constants / 模块常量
```python
logger = init_logger(__name__)
```
**EN:** This section defines module-level aliases, constants, or shared state such as `logger`, which later code reuses.
**CN:** 该部分定义模块级别的别名、常量或共享状态，例如 `logger`，供后续代码复用。

### Class `KeyMetadata` / 类 `KeyMetadata`
```python
@dataclass
class KeyMetadata:
    """Metadata for constructing pool keys."""

    model_name: str
    tp_rank: int
    pcp_rank: int
    dcp_rank: int
    pp_rank: int
```
**EN:** Declares `KeyMetadata`, a dataclass. It packages structured data fields such as `model_name`, `tp_rank`, `pcp_rank`, `dcp_rank`, `pp_rank`. The docstring summarizes its role as: Metadata for constructing pool keys.
**CN:** 声明 `KeyMetadata`，它是一个数据类。 它封装了 `model_name`, `tp_rank`, `pcp_rank`, `dcp_rank`, `pp_rank` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `PoolKey` / 类 `PoolKey`
```python
@dataclass(order=True)
class PoolKey:
    """Key for addressing KV cache blocks in the distributed store."""

    key_metadata: KeyMetadata
    chunk_hash: str

    def __hash__(self):
        return hash(
            (
                self.key_metadata.model_name,
                self.key_metadata.tp_rank,
                self.key_metadata.pcp_rank,
                self.key_metadata.dcp_rank,
                self.key_metadata.pp_rank,
                self.chunk_hash,
            )
        )

    def to_string(self) -> str:
        return (
            f"{self.key_metadata.model_name}"
            f"@tp_rank:{self.key_metadata.tp_rank}"
            f"@pcp{self.key_metadata.pcp_rank}"
            f"@dcp{self.key_metadata.dcp_rank}"
            f"@pp_rank:{self.key_metadata.pp_rank}"
            f"@{self.chunk_hash}"
        )
```
**EN:** Declares `PoolKey`, a class. Key methods include `__hash__`, `to_string`. The docstring summarizes its role as: Key for addressing KV cache blocks in the distributed store.
**CN:** 声明 `PoolKey`，它是一个类。 关键方法包括 `__hash__`, `to_string`。 文档字符串概括了它在整体流程中的职责。

### Class `ChunkedTokenDatabase` / 类 `ChunkedTokenDatabase`
```python
class ChunkedTokenDatabase:
    """Maps token positions to store keys and GPU memory addresses."""

    def __init__(self, metadata: KeyMetadata, block_size: int):
        self.metadata = metadata
        self.block_size = block_size
        self.kv_caches_base_addr: list[int] = []
        self.block_len: list[int] = []

    def _make_key_by_hash(self, chunk_hash: str) -> PoolKey:
        return PoolKey(self.metadata, chunk_hash)

    def set_kv_caches_base_addr(self, kv_caches_base_addr: list[int]):
        self.kv_caches_base_addr = kv_caches_base_addr

    def set_block_len(self, block_len: list[int]):
        for length in block_len:
            if length % self.block_size != 0:
                raise ValueError(f"block_len {length} % {self.block_size} != 0")
        self.block_len = block_len

    def prepare_value(
        self, start: int, end: int, block_ids: list[int]
    ) -> tuple[list[int], list[int], int]:
        """Compute memory addresses and sizes for a token range.

        Returns:
            (addr_list, size_list, block_id)
        """
        addr_list = []
        size_list = []
        block_id = block_ids[start // self.block_size]
        length = len(self.block_len)
        for index, base_addr in enumerate(self.kv_caches_base_addr):
            addr = base_addr + block_id * self.block_len[index % length]
            size = self.block_len[index % length] // self.block_size * (end - start)
            addr_list.append(addr)
            size_list.append(size)
        return addr_list, size_list, block_id

    def process_tokens(
        self,
        token_len: int,
        block_hashes: list[BlockHash] | list[str],
        mask_num: int = 0,
# ... truncated for analysis ...
            if start_idx < mask_num:
                continue
            else:
                yield (
                    start_idx,
                    end_idx,
                    self._make_key_by_hash(
                        hash_val  # type: ignore[arg-type]
                    ),
                )
```
**EN:** Declares `ChunkedTokenDatabase`, a class. Key methods include `__init__`, `_make_key_by_hash`, `set_kv_caches_base_addr`, `set_block_len`, `prepare_value`. The docstring summarizes its role as: Maps token positions to store keys and GPU memory addresses.
**CN:** 声明 `ChunkedTokenDatabase`，它是一个类。 关键方法包括 `__init__`, `_make_key_by_hash`, `set_kv_caches_base_addr`, `set_block_len`, `prepare_value`。 文档字符串概括了它在整体流程中的职责。

### Class `LoadSpec` / 类 `LoadSpec`
```python
@dataclass
class LoadSpec:
    """Specification for loading KV cache from external store."""

    vllm_cached_tokens: int
    kvpool_cached_tokens: int
    can_load: bool
    token_len: int = 0
```
**EN:** Declares `LoadSpec`, a dataclass. It packages structured data fields such as `vllm_cached_tokens`, `kvpool_cached_tokens`, `can_load`, `token_len`. The docstring summarizes its role as: Specification for loading KV cache from external store.
**CN:** 声明 `LoadSpec`，它是一个数据类。 它封装了 `vllm_cached_tokens`, `kvpool_cached_tokens`, `can_load`, `token_len` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `RequestTracker` / 类 `RequestTracker`
```python
@dataclass
class RequestTracker:
    """Tracks per-request state across scheduler ticks."""

    req_id: str
    token_len: int
    allocated_block_ids: list[int]
    num_saved_tokens: int = 0
    token_ids: list[int] | None = None
    # Snapshot of the prefill range length at tracker creation time.
    # For a fresh request this is len(prompt). For a resumed-from-preemption
    # request it includes previously-generated tokens, which are re-prefilled.
    prefill_end_tokens: int = 0

    def update(
        self,
        new_block_ids: tuple[list[int], ...] | list[int],
    ) -> None:
        if len(new_block_ids) == 0:
            new_block_ids = []
        elif isinstance(new_block_ids, tuple):
            new_block_ids = new_block_ids[0]
        elif isinstance(new_block_ids, list):
            pass
        else:
            raise ValueError(f"Unsupported new_block_ids type {type(new_block_ids)}")
        self.allocated_block_ids.extend(new_block_ids)
```
**EN:** Declares `RequestTracker`, a dataclass. It packages structured data fields such as `req_id`, `token_len`, `allocated_block_ids`, `num_saved_tokens`, `token_ids`. The docstring summarizes its role as: Tracks per-request state across scheduler ticks.
**CN:** 声明 `RequestTracker`，它是一个数据类。 它封装了 `req_id`, `token_len`, `allocated_block_ids`, `num_saved_tokens`, `token_ids` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `ReqMeta` / 类 `ReqMeta`
```python
@dataclass
class ReqMeta:
    """Per-request metadata for store put/get operations."""

    req_id: str
    token_len_chunk: int
    block_ids: list[int]
    block_hashes: list[BlockHash]

    can_save: bool | None = None
    load_spec: LoadSpec | None = None
    is_last_chunk: bool | None = None
    current_event: torch.cuda.Event | None = None

    token_ids: list[int] | None = None
    original_block_size: int | None = None

    @staticmethod
    def from_request_tracker(
        tracker: RequestTracker,
        block_size: int,
        load_spec: LoadSpec | None = None,
        skip_save: bool | None = False,
        block_hashes: list[BlockHash] | None = None,
        is_last_chunk: bool | None = None,
        discard_partial_chunks: bool = True,
        original_block_size: int | None = None,
    ) -> "ReqMeta | None":
        """Create ReqMeta from a RequestTracker."""
        if block_hashes is None:
            block_hashes = []
        input_token_len = tracker.token_len

        chunk_boundary = (
            cdiv(tracker.num_saved_tokens + 1, block_size) * block_size
            if discard_partial_chunks
            else 0
        )
        num_tokens_to_save = (
            (input_token_len // block_size * block_size)
            if discard_partial_chunks
            else input_token_len
        )

        skip_save = skip_save or num_tokens_to_save < chunk_boundary
# ... truncated for analysis ...
            req_id=tracker.req_id,
            token_len_chunk=num_tokens_to_save,
            block_ids=tracker.allocated_block_ids,
            can_save=not skip_save,
            load_spec=load_spec,
            block_hashes=block_hashes,
            is_last_chunk=is_last_chunk,
            token_ids=token_ids,
            original_block_size=original_block_size,
        )
```
**EN:** Declares `ReqMeta`, a dataclass. It packages structured data fields such as `req_id`, `token_len_chunk`, `block_ids`, `block_hashes`, `can_save`. The docstring summarizes its role as: Per-request metadata for store put/get operations.
**CN:** 声明 `ReqMeta`，它是一个数据类。 它封装了 `req_id`, `token_len_chunk`, `block_ids`, `block_hashes`, `can_save` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `MooncakeStoreConnectorMetadata` / 类 `MooncakeStoreConnectorMetadata`
```python
class MooncakeStoreConnectorMetadata(KVConnectorMetadata):
    """Metadata passed from scheduler to worker."""

    def __init__(
        self,
        unfinished_request_ids: set[str],
        preempted_req_ids: set[str],
    ):
        self.requests: list[ReqMeta] = []
        self.unfinished_request_ids = unfinished_request_ids
        self.preempted_req_ids = preempted_req_ids

    def add_request(self, req_meta: ReqMeta) -> None:
        self.requests.append(req_meta)
```
**EN:** Declares `MooncakeStoreConnectorMetadata`, a class derived from `KVConnectorMetadata`. Key methods include `__init__`, `add_request`. The docstring summarizes its role as: Metadata passed from scheduler to worker.
**CN:** 声明 `MooncakeStoreConnectorMetadata`，它是一个类，继承自 `KVConnectorMetadata`。 关键方法包括 `__init__`, `add_request`。 文档字符串概括了它在整体流程中的职责。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `KeyMetadata`: dataclass interface or data carrier / `KeyMetadata`：数据类接口或数据载体
- `PoolKey`: class interface or data carrier / `PoolKey`：类接口或数据载体
- `ChunkedTokenDatabase`: class interface or data carrier / `ChunkedTokenDatabase`：类接口或数据载体
- `LoadSpec`: dataclass interface or data carrier / `LoadSpec`：数据类接口或数据载体
- `RequestTracker`: dataclass interface or data carrier / `RequestTracker`：数据类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `dataclasses`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.logger`, `vllm.utils.math_utils`, `vllm.v1.core.kv_cache_utils`
