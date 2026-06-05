# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/hf3fs/utils/common.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements HF3FS-backed KV-transfer helpers or connectors. / 实现基于 HF3FS 的 KV 传输辅助逻辑或连接器。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化
```python
import threading
from dataclasses import dataclass, field
from typing import Optional

from vllm.distributed.kv_transfer.kv_connector.v1.base import KVConnectorMetadata
from vllm.v1.request import Request
```
**EN:** This block imports `threading`, `dataclasses`, `typing`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.v1.request` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `threading`, `dataclasses`, `typing`, `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.v1.request`，为后续实现准备运行时、类型与辅助 API。

### Class `AtomicCounter` / 类 `AtomicCounter`
```python
class AtomicCounter:
    """Thread-safe atomic counter for round-robin operations."""

    def __init__(self, n: int):
        assert n > 0, "Counter size must be positive"
        self._n = n
        self._value = 0
        self._lock = threading.Lock()

    def next(self) -> int:
        """Get next value in round-robin fashion."""
        with self._lock:
            current = self._value
            self._value = (current + 1) % self._n
            return current
```
**EN:** Declares `AtomicCounter`, a class. Key methods include `__init__`, `next`. The docstring summarizes its role as: Thread-safe atomic counter for round-robin operations.
**CN:** 声明 `AtomicCounter`，它是一个类。 关键方法包括 `__init__`, `next`。 文档字符串概括了它在整体流程中的职责。

### Class `LoadBlockInfo` / 类 `LoadBlockInfo`
```python
@dataclass
class LoadBlockInfo:
    """Operation for loading blocks from external storage."""

    num_computed_blocks: int
    num_blocks_to_load: int
    need_fetch_block_ids: list[int]
```
**EN:** Declares `LoadBlockInfo`, a dataclass. It packages structured data fields such as `num_computed_blocks`, `num_blocks_to_load`, `need_fetch_block_ids`. The docstring summarizes its role as: Operation for loading blocks from external storage.
**CN:** 声明 `LoadBlockInfo`，它是一个数据类。 它封装了 `num_computed_blocks`, `num_blocks_to_load`, `need_fetch_block_ids` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `SaveBlockInfo` / 类 `SaveBlockInfo`
```python
@dataclass
class SaveBlockInfo:
    """Operation for saving blocks to external storage."""

    skip_leading_blocks: int
```
**EN:** Declares `SaveBlockInfo`, a dataclass. It packages structured data fields such as `skip_leading_blocks`. The docstring summarizes its role as: Operation for saving blocks to external storage.
**CN:** 声明 `SaveBlockInfo`，它是一个数据类。 它封装了 `skip_leading_blocks` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `RequestSchedulingState` / 类 `RequestSchedulingState`
```python
@dataclass
class RequestSchedulingState:
    """Unified request scheduling state management."""

    request_id: str
    request: Request | None = None

    # Token and block tracking
    token_ids: list[int] = field(default_factory=list)
    allocated_block_ids: list[int] = field(default_factory=list)
    num_saved_blocks: int = 0

    # Load operation info
    load_op: LoadBlockInfo | None = None

    # Scheduling phase
    phase: str = "NEW"  # NEW -> WAITING_TO_LOAD -> ACTIVE -> FINISHED

    def needs_loading(self) -> bool:
        """Check if request needs loading."""
        return self.load_op is not None and self.load_op.num_blocks_to_load > 0

    def is_ready_to_load(self) -> bool:
        """Check if request is ready for loading."""
        return self.phase == "WAITING_TO_LOAD" and self.needs_loading()

    def update_tokens_and_blocks(self, new_token_ids: list[int], new_block_ids) -> None:
        """Update with new tokens and blocks."""
        if new_token_ids:
            self.token_ids.extend(new_token_ids)

        if new_block_ids is not None:
            normalized_block_ids = self._normalize_block_ids(new_block_ids)
            self.allocated_block_ids.extend(normalized_block_ids)

    def _normalize_block_ids(self, block_ids) -> list[int]:
        """Normalize block_ids to list format."""
        if not block_ids:
            return []
        if isinstance(block_ids, tuple):
            return block_ids[0] if block_ids else []
        if isinstance(block_ids, list):
            return block_ids
        return []
```
**EN:** Declares `RequestSchedulingState`, a dataclass. It packages structured data fields such as `request_id`, `request`, `token_ids`, `allocated_block_ids`, `num_saved_blocks`. The docstring summarizes its role as: Unified request scheduling state management.
**CN:** 声明 `RequestSchedulingState`，它是一个数据类。 它封装了 `request_id`, `request`, `token_ids`, `allocated_block_ids`, `num_saved_blocks` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `HF3FSRequestMetadata` / 类 `HF3FSRequestMetadata`
```python
@dataclass
class HF3FSRequestMetadata:
    """Metadata for a single request in HF3FS connector."""

    request_id: str
    token_ids: list[int]
    block_ids: list[int]
    load_block_op: LoadBlockInfo | None = None
    save_block_op: SaveBlockInfo | None = None

    @staticmethod
    def from_scheduling_state(
        state: "RequestSchedulingState",
        block_size: int,
        load_op: LoadBlockInfo | None = None,
        skip_leading_blocks: int | None = None,
    ) -> Optional["HF3FSRequestMetadata"]:
        """Create request metadata from scheduling state."""
        token_count = len(state.token_ids)
        total_blocks = token_count // block_size

        skip_blocks = (
            state.num_saved_blocks
            if skip_leading_blocks is None
            else skip_leading_blocks
        )

        new_blocks_to_save = total_blocks - state.num_saved_blocks
        if new_blocks_to_save <= 0 and load_op is None:
            return None

        state.num_saved_blocks = total_blocks
        return HF3FSRequestMetadata(
            request_id=state.request_id,
            token_ids=state.token_ids,
            block_ids=state.allocated_block_ids,
            load_block_op=load_op,
            save_block_op=SaveBlockInfo(skip_leading_blocks=skip_blocks),
        )
```
**EN:** Declares `HF3FSRequestMetadata`, a dataclass. It packages structured data fields such as `request_id`, `token_ids`, `block_ids`, `load_block_op`, `save_block_op`. The docstring summarizes its role as: Metadata for a single request in HF3FS connector.
**CN:** 声明 `HF3FSRequestMetadata`，它是一个数据类。 它封装了 `request_id`, `token_ids`, `block_ids`, `load_block_op`, `save_block_op` 等结构化字段。 文档字符串概括了它在整体流程中的职责。

### Class `HF3FSConnectorMetadata` / 类 `HF3FSConnectorMetadata`
```python
class HF3FSConnectorMetadata(KVConnectorMetadata):
    """Container for HF3FS connector metadata."""

    def __init__(self):
        self.requests: list[HF3FSRequestMetadata] = []

    def add_request(self, request_metadata: HF3FSRequestMetadata) -> None:
        """Add request to metadata."""
        self.requests.append(request_metadata)
```
**EN:** Declares `HF3FSConnectorMetadata`, a class derived from `KVConnectorMetadata`. Key methods include `__init__`, `add_request`. The docstring summarizes its role as: Container for HF3FS connector metadata.
**CN:** 声明 `HF3FSConnectorMetadata`，它是一个类，继承自 `KVConnectorMetadata`。 关键方法包括 `__init__`, `add_request`。 文档字符串概括了它在整体流程中的职责。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `AtomicCounter`: class interface or data carrier / `AtomicCounter`：类接口或数据载体
- `LoadBlockInfo`: dataclass interface or data carrier / `LoadBlockInfo`：数据类接口或数据载体
- `SaveBlockInfo`: dataclass interface or data carrier / `SaveBlockInfo`：数据类接口或数据载体
- `RequestSchedulingState`: dataclass interface or data carrier / `RequestSchedulingState`：数据类接口或数据载体
- `HF3FSRequestMetadata`: dataclass interface or data carrier / `HF3FSRequestMetadata`：数据类接口或数据载体

## Dependencies / 依赖关系
- **Standard library / 标准库**: `threading`, `dataclasses`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal modules / 内部模块**: `vllm.distributed.kv_transfer.kv_connector.v1.base`, `vllm.v1.request`
