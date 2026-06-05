# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/kv_offload/tiering/base.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Abstract interfaces and data types for the secondary tiering layer. / 该模块位于 `kv_offload/tiering` 子系统，主要围绕 `JobMetadata`, `JobResult`, `SecondaryTierManager` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""
Abstract interfaces and data types for the secondary tiering layer.
"""

from abc import ABC, abstractmethod
from collections.abc import Collection, Iterable
from dataclasses import dataclass
from typing import TYPE_CHECKING

import numpy as np

from vllm.v1.kv_offload.base import OffloadKey, ReqContext

if TYPE_CHECKING:
    from vllm.config import VllmConfig

# Type alias for job IDs used in async transfer tracking
JobId = int
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `JobId`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `JobId`。

### `JobMetadata` class / `JobMetadata` 类
```python
@dataclass
class JobMetadata:
    """Metadata for an in-flight async transfer job."""

    job_id: JobId
    keys: Collection[OffloadKey]
    block_ids: np.ndarray
    is_promotion: bool
    req_context: ReqContext
```
**EN:** Uses `@dataclass` to package related state for `JobMetadata`. Typical fields include `job_id`, `keys`, `block_ids`, `is_promotion`, `req_context`.
**CN:** `JobMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `job_id`, `keys`, `block_ids`, `is_promotion`, `req_context`。

### `JobResult` class / `JobResult` 类
```python
@dataclass
class JobResult:
    """Result of an async transfer job (successful or failed)."""

    job_id: JobId
    success: bool
```
**EN:** Uses `@dataclass` to package related state for `JobResult`. Typical fields include `job_id`, `success`.
**CN:** `JobResult` 使用 `@dataclass` 打包相关状态。典型字段包括 `job_id`, `success`。

### `SecondaryTierManager` class / `SecondaryTierManager` 类
```python
class SecondaryTierManager(ABC):
    """
    Abstract interface for managing a single non-primary offloading tier.

    Secondary tiers cannot directly access GPU memory. All data transfers
    must go through the CPU (primary) tier:
      - Store: GPU → CPU (primary) → secondary  (cascade)
      - Load:  secondary → CPU (primary) → GPU  (promotion)

    IMPORTANT: All methods run in the Scheduler process and must be
    lightweight and non-blocking. submit_load() and submit_store() submit
    async jobs; get_finished() polls for completion.
    """
```
**EN:** Declares the `SecondaryTierManager` interface. Downstream implementations are expected to provide methods such as `__init__`, `lookup`, `submit_store`, `submit_load`, `get_finished`, `touch`.
**CN:** `SecondaryTierManager` 声明了一组接口约定。下游实现需要提供 `__init__`, `lookup`, `submit_store`, `submit_load`, `get_finished`, `touch` 等方法。

### `SecondaryTierManager.__init__` method / `SecondaryTierManager.__init__` 方法
```python
    def __init__(self, vllm_config: "VllmConfig", primary_kv_view: memoryview) -> None:
        self._vllm_config = vllm_config
        self._primary_kv_view: memoryview = primary_kv_view
```
**EN:** This method initializes the object state within `SecondaryTierManager`. It touches state such as `_vllm_config`, `_primary_kv_view`.
**CN:** 该方法会初始化对象状态，其作用域位于`SecondaryTierManager`。 它会读写 `_vllm_config`, `_primary_kv_view` 等状态。

### `SecondaryTierManager.lookup` method / `SecondaryTierManager.lookup` 方法
```python
    @abstractmethod
    def lookup(self, key: OffloadKey, req_context: ReqContext) -> bool | None:
        """
        Check whether a block exists in this secondary tier.

        Args:
            key: Offload key to look up.
            req_context: per-request context (e.g. kv_transfer_params).

        Returns:
            True if the block is present and ready,
            False if not found,
            or None if the block is being transferred (retry later).
        """
        pass
```
**EN:** This method implements `lookup` within `SecondaryTierManager`. The docstring frames it as: Check whether a block exists in this secondary tier.
**CN:** 该方法会实现 `lookup`，其作用域位于`SecondaryTierManager`。

### `SecondaryTierManager.submit_store` method / `SecondaryTierManager.submit_store` 方法
```python
    @abstractmethod
    def submit_store(self, job_metadata: JobMetadata) -> None:
        """
        Submit an async job to store blocks from the primary tier to this
        secondary tier.

        This method must be lightweight and non-blocking: allocate metadata
        and submit the transfer, but do NOT perform the data copy on the
        calling thread.

        Preconditions (guaranteed by the framework):
          - ``job_metadata.block_ids`` are valid primary-tier slots, pinned
            (ref-counted) for the duration of the transfer.

        The implementation is responsible for:
          1. Filtering out blocks already present in this tier
          2. Evicting blocks if capacity is needed
          3. Allocating space in this tier
          4. Submitting the async transfer (read from primary via block_ids)

        Report completion via ``get_finished()``.

        Args:
            job_metadata: Job metadata including job_id, keys, and block_ids
                          identifying the primary-tier slots to read from.
        """
        pass
```
**EN:** This method implements `submit_store` within `SecondaryTierManager`. The docstring frames it as: Submit an async job to store blocks from the primary tier to this secondary tier.
**CN:** 该方法会实现 `submit_store`，其作用域位于`SecondaryTierManager`。

### `SecondaryTierManager.submit_load` method / `SecondaryTierManager.submit_load` 方法
```python
    @abstractmethod
    def submit_load(self, job_metadata: JobMetadata) -> None:
        """
        Submit an async job to load blocks from this secondary tier to the
        primary tier.

        This method must be lightweight and non-blocking: mark blocks as
        in-flight and submit the transfer, but do NOT perform the data copy
        on the calling thread.

        Preconditions (guaranteed by the framework):
          - ``job_metadata.block_ids`` are allocated primary-tier slots
            ready to receive data.

        The implementation must copy data from this tier into the
        primary-tier slots identified by ``block_ids``.

        Report completion via ``get_finished()``.

        Args:
            job_metadata: Job metadata including job_id, keys, and block_ids
                          identifying the primary-tier slots to write into.
        """
        pass
```
**EN:** This method implements `submit_load` within `SecondaryTierManager`. The docstring frames it as: Submit an async job to load blocks from this secondary tier to the primary tier.
**CN:** 该方法会实现 `submit_load`，其作用域位于`SecondaryTierManager`。

### `SecondaryTierManager.get_finished` method / `SecondaryTierManager.get_finished` 方法
```python
    @abstractmethod
    def get_finished(self) -> Iterable[JobResult]:
        """
        Return all jobs (loads and stores) that completed since the last call.

        The framework uses these results to release resources and finalize
        transfers.

        Returns:
            Iterable of JobResult objects for jobs finished since the
            last call.
        """
        pass
```
**EN:** This method returns or derives a value within `SecondaryTierManager`. The docstring frames it as: Return all jobs (loads and stores) that completed since the last call.
**CN:** 该方法会返回或推导一个值，其作用域位于`SecondaryTierManager`。

### `SecondaryTierManager.touch` method / `SecondaryTierManager.touch` 方法
```python
    def touch(self, keys: Collection[OffloadKey], req_context: ReqContext):
        """
        Mark blocks as recently used for eviction policy.

        Args:
            keys: Offload keys to mark as recently used.
            req_context: Per-request context.
        """
        return
```
**EN:** This method implements `touch` within `SecondaryTierManager`. The docstring frames it as: Mark blocks as recently used for eviction policy.
**CN:** 该方法会实现 `touch`，其作用域位于`SecondaryTierManager`。

### `SecondaryTierManager.shutdown` method / `SecondaryTierManager.shutdown` 方法
```python
    def shutdown(self) -> None:
        """Release resources held by this tier (threads, connections, etc.)."""
        return
```
**EN:** This method implements `shutdown` within `SecondaryTierManager`. The docstring frames it as: Release resources held by this tier (threads, connections, etc.).
**CN:** 该方法会实现 `shutdown`，其作用域位于`SecondaryTierManager`。

### `SecondaryTierManager.get_tier_type` method / `SecondaryTierManager.get_tier_type` 方法
```python
    @staticmethod
    @abstractmethod
    def get_tier_type() -> str:
        """
        Get the type identifier of this tier (e.g., "example", "storage").

        Must match the "type" field in the tier config dict.

        Returns:
            Tier type string.
        """
        pass
```
**EN:** This method returns or derives a value within `SecondaryTierManager`. The docstring frames it as: Get the type identifier of this tier (e.g., "example", "storage").
**CN:** 该方法会返回或推导一个值，其作用域位于`SecondaryTierManager`。

## Key Concepts / 关键概念
- `JobMetadata`: central class or interface in this module. / `JobMetadata`：本模块中的核心类或接口。
- `JobResult`: central class or interface in this module. / `JobResult`：本模块中的核心类或接口。
- `SecondaryTierManager`: central class or interface in this module. / `SecondaryTierManager`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `abc`, `collections`, `dataclasses`, `typing`
- External / 外部依赖: `numpy`
- Internal vLLM / 内部依赖: `vllm.v1.kv_offload.base`, `vllm.config`
