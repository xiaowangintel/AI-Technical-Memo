# hicache_storage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/hicache_storage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the hicache storage logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的hicache存储相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and setup / 导入与初始化
```python
from __future__ import annotations

import logging
import os
from abc import ABC, abstractmethod
from dataclasses import dataclass
from enum import Enum
from typing import TYPE_CHECKING, Any, List, Optional, Set

import torch

from sglang.srt.environ import envs
```
**EN:** Imports `__future__`, `logging`, `os`, `abc`, `dataclasses`, `enum` and other helpers used by the surrounding scope.
**CN:** 导入 `__future__`, `logging`, `os`, `abc`, `dataclasses`, `enum` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 14-15: Type-checking guard / 类型检查保护
```python
if TYPE_CHECKING:
    from sglang.srt.mem_cache.memory_pool_host import HostKVCache
```
**EN:** Keeps type-only imports or declarations out of runtime execution.
**CN:** 将仅用于类型检查的导入或声明隔离到运行时之外。

### Lines 17-17: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `logger`.
**CN:** 定义模块级变量，例如 `logger`。

### Lines 20-21: HiCacheStorageConfig declaration / HiCacheStorageConfig 声明
```python
@dataclass
class HiCacheStorageConfig:
```
**EN:** Declares the `HiCacheStorageConfig` class.
**CN:** 声明 `HiCacheStorageConfig` 类。

### Lines 22-34: Shared state definitions / 共享状态定义
```python
    tp_rank: int
    tp_size: int
    pp_rank: int
    pp_size: int
    attn_cp_rank: int
    attn_cp_size: int
    is_mla_model: bool
    enable_storage_metrics: bool
    is_page_first_layout: bool
    model_name: Optional[str]
    tp_lcm_size: Optional[int] = None
    should_split_heads: bool = False
    extra_config: Optional[dict] = None
```
**EN:** Defines class-level variables such as `tp_rank`, `tp_size`, `pp_rank`, `pp_size`, `attn_cp_rank`.
**CN:** 定义类级变量，例如 `tp_rank`, `tp_size`, `pp_rank`, `pp_size`, `attn_cp_rank`。

### Lines 37-38: HiCacheStorageExtraInfo declaration / HiCacheStorageExtraInfo 声明
```python
@dataclass
class HiCacheStorageExtraInfo:
```
**EN:** Declares the `HiCacheStorageExtraInfo` class.
**CN:** 声明 `HiCacheStorageExtraInfo` 类。

### Lines 39-40: Shared state definitions / 共享状态定义
```python
    prefix_keys: Optional[List[str]] = None
    extra_info: Optional[dict] = None
```
**EN:** Defines class-level variables such as `prefix_keys`, `extra_info`.
**CN:** 定义类级变量，例如 `prefix_keys`, `extra_info`。

### Lines 43-46: PrefetchTimeoutConfig declaration / PrefetchTimeoutConfig 声明
```python
@dataclass(frozen=True)
class PrefetchTimeoutConfig:
    """Knobs for the linear prefetch-timeout policy used by HiCache."""

```
**EN:** Knobs for the linear prefetch-timeout policy used by HiCache. Declares the `PrefetchTimeoutConfig` class.
**CN:** 声明 `PrefetchTimeoutConfig` 类。

### Lines 47-49: Shared state definitions / 共享状态定义
```python
    base: float = 2.0  # seconds, fixed overhead unrelated to token count
    per_ki_token: float = 0.1  # seconds per 1024 tokens
    max: float = 30.0  # seconds, upper bound for the linear timeout
```
**EN:** Defines class-level variables such as `base`, `per_ki_token`, `max`.
**CN:** 定义类级变量，例如 `base`, `per_ki_token`, `max`。

### Lines 52-54: PoolName declaration / PoolName 声明
```python
class PoolName(str, Enum):
    """Well-known pool names used as PoolTransfer/PoolEntry identifiers."""

```
**EN:** Well-known pool names used as PoolTransfer/PoolEntry identifiers. Declares the `PoolName` class and connects it to `str`, `Enum`.
**CN:** 声明 `PoolName` 类，并将其关联到 `str`, `Enum`。

### Lines 55-66: Shared state definitions / 共享状态定义
```python
    KV = "kv"
    MAMBA = "mamba"
    SWA = "swa"
    INDEXER = "indexer"
    # TODO(hzh0425): Current DeepSeek V4 pool naming is verbose; will be normalized to
    # 'COMPRESSED_KV / COMPRESSED_INDEXER / COMPRESSED_STATE' in the next PR.
    DEEPSEEK_V4_C4 = "deepseek_v4_c4"
    DEEPSEEK_V4_C4_INDEXER = "deepseek_v4_c4_indexer"
    DEEPSEEK_V4_C128 = "deepseek_v4_c128"
    DEEPSEEK_V4_C4_STATE = "deepseek_v4_c4_state"
    DEEPSEEK_V4_C4_INDEXER_STATE = "deepseek_v4_c4_indexer_state"
    DEEPSEEK_V4_C128_STATE = "deepseek_v4_c128_state"
```
**EN:** Defines class-level variables such as `KV`, `MAMBA`, `SWA`, `INDEXER`, `DEEPSEEK_V4_C4`.
**CN:** 定义类级变量，例如 `KV`, `MAMBA`, `SWA`, `INDEXER`, `DEEPSEEK_V4_C4`。

### Lines 68-69: __str__ implementation / __str__ 实现
```python
    def __str__(self) -> str:
        return self.value
```
**EN:** Implements the STR routine for this scope. It belongs to `PoolName`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的str例程。 该方法属于 `PoolName`。它会向调用方返回计算结果。

### Lines 72-78: PoolHitPolicy declaration / PoolHitPolicy 声明
```python
class PoolHitPolicy(str, Enum):
    """Hit policy for batch_exists_v2 per-pool prefix matching.

    ALL_PAGES      : every page in [0, kv_hit) must exist (e.g. DSA).
    TRAILING_PAGES : only the last N pages must exist (e.g. Mamba/SWA states).
    """

```
**EN:** Hit policy for batch_exists_v2 per-pool prefix matching. Declares the `PoolHitPolicy` class and connects it to `str`, `Enum`.
**CN:** 声明 `PoolHitPolicy` 类，并将其关联到 `str`, `Enum`。

### Lines 79-80: Shared state definitions / 共享状态定义
```python
    ALL_PAGES = "all_pages"
    TRAILING_PAGES = "trailing_pages"
```
**EN:** Defines class-level variables such as `ALL_PAGES`, `TRAILING_PAGES`.
**CN:** 定义类级变量，例如 `ALL_PAGES`, `TRAILING_PAGES`。

### Lines 83-91: PoolTransfer declaration / PoolTransfer 声明
```python
@dataclass
class PoolTransfer:
    """Unified per-pool transfer descriptor for batch v2 interface.

    device<->host path : host_indices + device_indices
    host<->storage path: host_indices + keys
    nodes_to_load      : evicted nodes this transfer covers
    """

```
**EN:** Unified per-pool transfer descriptor for batch v2 interface. Declares the `PoolTransfer` class.
**CN:** 声明 `PoolTransfer` 类。

### Lines 92-98: Shared state definitions / 共享状态定义
```python
    name: PoolName
    host_indices: Optional[torch.Tensor] = None
    device_indices: Optional[torch.Tensor] = None
    keys: Optional[List[str]] = None
    hit_policy: PoolHitPolicy = PoolHitPolicy.ALL_PAGES
    nodes_to_load: Optional[List[Any]] = None
    indices_from_pool: Optional[PoolName] = None
```
**EN:** Defines class-level variables such as `name`, `host_indices`, `device_indices`, `keys`, `hit_policy`.
**CN:** 定义类级变量，例如 `name`, `host_indices`, `device_indices`, `keys`, `hit_policy`。

### Lines 101-104: SidecarPoolSpec declaration / SidecarPoolSpec 声明
```python
@dataclass(frozen=True)
class SidecarPoolSpec:
    """Pool whose transfer indices are reused from one real source pool."""

```
**EN:** Pool whose transfer indices are reused from one real source pool. Declares the `SidecarPoolSpec` class.
**CN:** 声明 `SidecarPoolSpec` 类。

### Lines 105-107: Shared state definitions / 共享状态定义
```python
    pool_name: PoolName
    indices_from_pool: PoolName
    hit_policy: PoolHitPolicy = PoolHitPolicy.ALL_PAGES
```
**EN:** Defines class-level variables such as `pool_name`, `indices_from_pool`, `hit_policy`.
**CN:** 定义类级变量，例如 `pool_name`, `indices_from_pool`, `hit_policy`。

### Lines 110-113: PoolTransferResult declaration / PoolTransferResult 声明
```python
@dataclass
class PoolTransferResult:
    """Tracks how many pages were successfully processed per pool."""

```
**EN:** Tracks how many pages were successfully processed per pool. Declares the `PoolTransferResult` class.
**CN:** 声明 `PoolTransferResult` 类。

### Lines 114-115: Shared state definitions / 共享状态定义
```python
    kv_hit_pages: int
    extra_pool_hit_pages: dict[str, int]
```
**EN:** Defines class-level variables such as `kv_hit_pages`, `extra_pool_hit_pages`.
**CN:** 定义类级变量，例如 `kv_hit_pages`, `extra_pool_hit_pages`。

### Lines 117-119: empty implementation / empty 实现
```python
    @classmethod
    def empty(cls) -> "PoolTransferResult":
        return cls(0, {})
```
**EN:** Implements the empty routine for this scope. It belongs to `PoolTransferResult`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的empty例程。 该方法属于 `PoolTransferResult`。它会向调用方返回计算结果。

### Lines 121-123: update_kv_hit_pages implementation / update_kv_hit_pages 实现
```python
    def update_kv_hit_pages(self, kv_hit_pages: int) -> None:
        """Accumulate kv_hit_pages across batches (max = last successful batch)."""
        self.kv_hit_pages = max(self.kv_hit_pages, kv_hit_pages)
```
**EN:** Accumulate kv_hit_pages across batches (max = last successful batch). Updates existing state to reflect new inputs. It belongs to `PoolTransferResult`.
**CN:** 根据新输入更新已有状态。 该方法属于 `PoolTransferResult`。

### Lines 125-129: update_extra_pool_hit_pages implementation / update_extra_pool_hit_pages 实现
```python
    def update_extra_pool_hit_pages(self, results: dict[str, List[bool]]) -> None:
        """Record actual load/write success counts per extra pool."""
        self.extra_pool_hit_pages.update(
            {name: sum(rs) for name, rs in results.items()}
        )
```
**EN:** Record actual load/write success counts per extra pool. Updates existing state to reflect new inputs. It belongs to `PoolTransferResult`.
**CN:** 根据新输入更新已有状态。 该方法属于 `PoolTransferResult`。

### Lines 132-138: HiCacheStorage declaration / HiCacheStorage 声明
```python
class HiCacheStorage(ABC):
    """
    HiCacheStorage is a class that provides a generic key-value interface for storing and retrieving KV cache.
    It abstracts the underlying storage mechanism, allowing different implementations to be used.
    """

    # todo, the page size of storage backend does not have to be the same as the same as host memory pool
```
**EN:** HiCacheStorage is a class that provides a generic key-value interface for storing and retrieving KV cache. Declares the `HiCacheStorage` class and connects it to `ABC`.
**CN:** 声明 `HiCacheStorage` 类，并将其关联到 `ABC`。

### Lines 139-140: register_mem_pool_host implementation / register_mem_pool_host 实现
```python
    def register_mem_pool_host(self, mem_pool_host: HostKVCache):
        self.mem_pool_host = mem_pool_host
```
**EN:** Registers metadata so other components can discover this object. It belongs to `HiCacheStorage`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `HiCacheStorage`。

### Lines 142-145: register_mem_host_pool_v2 implementation / register_mem_host_pool_v2 实现
```python
    def register_mem_host_pool_v2(self, host_pool: HostKVCache, host_pool_name):
        if not hasattr(self, "registered_pools"):
            self.registered_pools = {}
        self.registered_pools[host_pool_name] = host_pool
```
**EN:** Registers metadata so other components can discover this object. It belongs to `HiCacheStorage`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `HiCacheStorage`。

### Lines 147-178: batch_exists_v2 implementation / batch_exists_v2 实现
```python
    def batch_exists_v2(
        self,
        keys: List[str],
        pool_transfers: Optional[List[PoolTransfer]] = None,
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> PoolTransferResult:
        """Check which cache pages exist in storage, respecting per-pool hit policies.

        Longest-prefix semantics
        Extra-pool hit policies (``PoolTransfer.hit_policy``)
        ------------------------------------------------------
        Each ``PoolTransfer`` in ``pool_transfers`` describes a secondary
# ... omitted for brevity ...
            ``extra_pool_hit_pages`` maps each pool name to the number of pages
            that were found.
        """
        raise NotImplementedError()
```
**EN:** Check which cache pages exist in storage, respecting per-pool hit policies. Implements the batch exists V 2 routine for this scope. It belongs to `HiCacheStorage`. It validates error cases explicitly.
**CN:** 实现当前作用域中的batch exists v 2例程。 该方法属于 `HiCacheStorage`。它会显式处理错误场景。

### Lines 180-189: batch_get_v2 implementation / batch_get_v2 实现
```python
    def batch_get_v2(
        self,
        transfers: List[PoolTransfer],
        extra_info: Optional["HiCacheStorageExtraInfo"] = None,
    ) -> dict[str, List[bool]]:
        """Read data from storage into host memory for each PoolTransfer.

        Returns a dict mapping pool name to a per-entry success list.
        """
        raise NotImplementedError()
```
**EN:** Read data from storage into host memory for each PoolTransfer. Implements the batch get V 2 routine for this scope. It belongs to `HiCacheStorage`. It validates error cases explicitly.
**CN:** 实现当前作用域中的batch get v 2例程。 该方法属于 `HiCacheStorage`。它会显式处理错误场景。

### Lines 191-200: batch_set_v2 implementation / batch_set_v2 实现
```python
    def batch_set_v2(
        self,
        transfers: List[PoolTransfer],
        extra_info: Optional["HiCacheStorageExtraInfo"] = None,
    ) -> dict[str, List[bool]]:
        """Write data from host memory to storage for each PoolTransfer.

        Returns a dict mapping pool name to a per-entry success list.
        """
        raise NotImplementedError()
```
**EN:** Write data from host memory to storage for each PoolTransfer. Implements the batch set V 2 routine for this scope. It belongs to `HiCacheStorage`. It validates error cases explicitly.
**CN:** 实现当前作用域中的batch set v 2例程。 该方法属于 `HiCacheStorage`。它会显式处理错误场景。

### Lines 202-212: batch_get_v1 implementation / batch_get_v1 实现
```python
    def batch_get_v1(
        self,
        keys: List[str],
        host_indices: torch.Tensor,
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> List[bool]:
        """
        Retrieve values for multiple keys.
        Returns a list of booleans indicating success for each key.
        """
        pass
```
**EN:** Retrieve values for multiple keys. Implements the batch get V 1 routine for this scope. It belongs to `HiCacheStorage`.
**CN:** 实现当前作用域中的batch get v 1例程。 该方法属于 `HiCacheStorage`。

### Lines 214-224: batch_set_v1 implementation / batch_set_v1 实现
```python
    def batch_set_v1(
        self,
        keys: List[str],
        host_indices: torch.Tensor,
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> List[bool]:
        """
        Store multiple key-value pairs.
        Returns a list of booleans indicating success for each key.
        """
        pass
```
**EN:** Store multiple key-value pairs. Implements the batch set V 1 routine for this scope. It belongs to `HiCacheStorage`.
**CN:** 实现当前作用域中的batch set v 1例程。 该方法属于 `HiCacheStorage`。

### Lines 226-237: get implementation / get 实现
```python
    @abstractmethod
    def get(
        self,
        key: str,
        target_location: Optional[Any] = None,
        target_sizes: Optional[Any] = None,
    ) -> torch.Tensor | None:
        """
        Retrieve the value associated with the given key.
        Returns None if the key does not exist.
        """
        pass
```
**EN:** Retrieve the value associated with the given key. Retrieves the requested data or state from the current object. It belongs to `HiCacheStorage`.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HiCacheStorage`。

### Lines 238-239: Comment block / 注释块
```python

    # TODO: Deprecate
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 240-251: batch_get implementation / batch_get 实现
```python
    @abstractmethod
    def batch_get(
        self,
        keys: List[str],
        target_locations: Optional[Any] = None,
        target_sizes: Optional[Any] = None,
    ) -> List[torch.Tensor | None] | int:
        """
        Retrieve values for multiple keys.
        Returns a list of tensors or None for each key.
        """
        pass
```
**EN:** Retrieve values for multiple keys. Implements the batch get routine for this scope. It belongs to `HiCacheStorage`.
**CN:** 实现当前作用域中的batch get例程。 该方法属于 `HiCacheStorage`。

### Lines 253-265: set implementation / set 实现
```python
    @abstractmethod
    def set(
        self,
        key: str,
        value: Optional[Any] = None,
        target_location: Optional[Any] = None,
        target_sizes: Optional[Any] = None,
    ) -> bool:
        """
        Store the value associated with the given key.
        Returns True if the operation was successful, False otherwise.
        """
        pass
```
**EN:** Store the value associated with the given key. Updates internal state with the provided value. It belongs to `HiCacheStorage`.
**CN:** 使用给定值更新内部状态。 该方法属于 `HiCacheStorage`。

### Lines 266-267: Comment block / 注释块
```python

    # TODO: Deprecate
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 268-280: batch_set implementation / batch_set 实现
```python
    @abstractmethod
    def batch_set(
        self,
        keys: List[str],
        values: Optional[Any] = None,
        target_locations: Optional[Any] = None,
        target_sizes: Optional[Any] = None,
    ) -> bool:
        """
        Store multiple key-value pairs.
        Returns True if all operations were successful, False otherwise.
        """
        pass
```
**EN:** Store multiple key-value pairs. Implements the batch set routine for this scope. It belongs to `HiCacheStorage`.
**CN:** 实现当前作用域中的batch set例程。 该方法属于 `HiCacheStorage`。

### Lines 282-288: exists implementation / exists 实现
```python
    @abstractmethod
    def exists(self, key: str) -> bool:
        """
        Check if the key exists in the storage.
        Returns True if the key exists, False otherwise.
        """
        pass
```
**EN:** Check if the key exists in the storage. Implements the exists routine for this scope. It belongs to `HiCacheStorage`.
**CN:** 实现当前作用域中的exists例程。 该方法属于 `HiCacheStorage`。

### Lines 289-290: Comment block / 注释块
```python

    # TODO: Use a finer-grained return type (e.g., List[bool])
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 291-302: batch_exists implementation / batch_exists 实现
```python
    def batch_exists(
        self, keys: List[str], extra_info: Optional[HiCacheStorageExtraInfo] = None
    ) -> int:
        """
        Check if the keys exist in the storage.
        return the number of consecutive existing keys from the start.
        Can be overridden by subclasses for more efficient implementation.
        """
        for i in range(len(keys)):
            if not self.exists(keys[i]):
                return i
        return len(keys)
```
**EN:** Check if the keys exist in the storage. Implements the batch exists routine for this scope. It belongs to `HiCacheStorage`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch exists例程。 该方法属于 `HiCacheStorage`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 304-305: clear implementation / clear 实现
```python
    def clear(self) -> None:
        pass
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `HiCacheStorage`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `HiCacheStorage`。

### Lines 307-308: get_stats implementation / get_stats 实现
```python
    def get_stats(self):
        return None
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HiCacheStorage`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HiCacheStorage`。它会向调用方返回计算结果。

### Lines 311-312: HiCacheFile declaration / HiCacheFile 声明
```python
class HiCacheFile(HiCacheStorage):

```
**EN:** Declares the `HiCacheFile` class and connects it to `HiCacheStorage`.
**CN:** 声明 `HiCacheFile` 类，并将其关联到 `HiCacheStorage`。

### Lines 313-335: __init__ implementation / __init__ 实现
```python
    def __init__(
        self, storage_config: HiCacheStorageConfig, file_path: str = "/tmp/hicache"
    ):
        self.file_path = envs.SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR.get() or file_path

        tp_rank, tp_size, pp_rank, pp_size, model_name, is_mla_model = (
            storage_config.tp_rank,
            storage_config.tp_size,
            storage_config.pp_rank,
            storage_config.pp_size,
            storage_config.model_name,
            storage_config.is_mla_model,
# ... omitted for brevity ...
            self.config_suffix += f"_{pp_size}_{pp_rank}"
        if not os.path.exists(self.file_path) and tp_rank == 0:
            os.makedirs(self.file_path)
            logger.info(f"Created HiCacheFile storage directory at {self.file_path}")
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `HiCacheFile`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `HiCacheFile`。

### Lines 337-338: _get_suffixed_key implementation / _get_suffixed_key 实现
```python
    def _get_suffixed_key(self, key: str) -> str:
        return key + self.config_suffix
```
**EN:** Implements the get suffixed KEY routine for this scope. It belongs to `HiCacheFile`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get suffixed key例程。 该方法属于 `HiCacheFile`。它会向调用方返回计算结果。

### Lines 340-343: _get_component_key implementation / _get_component_key 实现
```python
    def _get_component_key(self, key: str, component_name: Optional[str] = None) -> str:
        if component_name is None or component_name in ("__default__", PoolName.KV):
            return self._get_suffixed_key(key)
        return self._get_suffixed_key(f"{key}.{component_name}")
```
**EN:** Implements the get component KEY routine for this scope. It belongs to `HiCacheFile`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get组件key例程。 该方法属于 `HiCacheFile`。它会向调用方返回计算结果。

### Lines 345-350: _get_component_path implementation / _get_component_path 实现
```python
    def _get_component_path(
        self, key: str, component_name: Optional[str] = None
    ) -> str:
        return os.path.join(
            self.file_path, f"{self._get_component_key(key, component_name)}.bin"
        )
```
**EN:** Implements the get component path routine for this scope. It belongs to `HiCacheFile`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get组件path例程。 该方法属于 `HiCacheFile`。它会向调用方返回计算结果。

### Lines 352-369: get implementation / get 实现
```python
    def get(
        self,
        key: str,
        target_location: torch.Tensor,
        target_sizes: Optional[Any] = None,
    ) -> torch.Tensor | None:
        key = self._get_suffixed_key(key)
        tensor_path = os.path.join(self.file_path, f"{key}.bin")
        try:
            expected = target_location.numel() * target_location.element_size()
            with open(tensor_path, "rb", buffering=0) as f:
                buf = memoryview(target_location.view(torch.uint8).contiguous().numpy())
                if f.readinto(buf) != expected:
                    raise IOError(f"Short read for {key}")
            return target_location
        except FileNotFoundError:
            logger.warning(f"Failed to fetch {key} from HiCacheFile storage.")
            return None
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `HiCacheFile`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HiCacheFile`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 371-382: batch_get implementation / batch_get 实现
```python
    def batch_get(
        self,
        keys: List[str],
        target_locations: List[torch.Tensor],
        target_sizes: Optional[Any] = None,
    ) -> List[torch.Tensor | None]:
        return [
            self.get(key, target_location)
            for key, target_location in zip(
                keys, target_locations or [None] * len(keys)
            )
        ]
```
**EN:** Implements the batch get routine for this scope. It belongs to `HiCacheFile`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get例程。 该方法属于 `HiCacheFile`。它会向调用方返回计算结果。

### Lines 384-402: set implementation / set 实现
```python
    def set(
        self,
        key: str,
        value: Optional[Any] = None,
        target_location: Optional[Any] = None,
        target_sizes: Optional[Any] = None,
    ) -> bool:
        if self.exists(key):
            logger.debug(f"Key {key} already exists. Skipped.")
            return True

        key = self._get_suffixed_key(key)
# ... omitted for brevity ...
            return True
        except Exception as e:
            logger.error(f"Failed to save tensor {key}: {e}")
            return False
```
**EN:** Updates internal state with the provided value. It belongs to `HiCacheFile`. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。 该方法属于 `HiCacheFile`。它会向调用方返回计算结果。

### Lines 404-414: batch_set implementation / batch_set 实现
```python
    def batch_set(
        self,
        keys: List[str],
        values: Optional[Any] = None,
        target_locations: Optional[Any] = None,
        target_sizes: Optional[Any] = None,
    ) -> bool:
        for key, value in zip(keys, values):
            if not self.set(key, value):
                return False
        return True
```
**EN:** Implements the batch set routine for this scope. It belongs to `HiCacheFile`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch set例程。 该方法属于 `HiCacheFile`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 416-419: exists implementation / exists 实现
```python
    def exists(self, key: str) -> bool:
        key = self._get_suffixed_key(key)
        tensor_path = os.path.join(self.file_path, f"{key}.bin")
        return os.path.exists(tensor_path)
```
**EN:** Implements the exists routine for this scope. It belongs to `HiCacheFile`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的exists例程。 该方法属于 `HiCacheFile`。它会向调用方返回计算结果。

### Lines 421-436: _collect_existing_component_keys implementation / _collect_existing_component_keys 实现
```python
    def _collect_existing_component_keys(
        self,
        keys: List[str],
        pool_transfers: Optional[List[PoolTransfer]] = None,
    ) -> Set[str]:
        target_files = {f"{self._get_component_key(key)}.bin" for key in keys}
        for transfer in pool_transfers or []:
            for key in keys:
                target_files.add(f"{self._get_component_key(key, transfer.name)}.bin")

        existing_files = set()
        with os.scandir(self.file_path) as entries:
            for entry in entries:
                if entry.is_file() and entry.name in target_files:
                    existing_files.add(entry.name)
        return existing_files
```
**EN:** Implements the collect existing component keys routine for this scope. It belongs to `HiCacheFile`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的collect existing组件keys例程。 该方法属于 `HiCacheFile`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 438-486: batch_exists_v2 implementation / batch_exists_v2 实现
```python
    def batch_exists_v2(
        self,
        keys: List[str],
        pool_transfers: Optional[List[PoolTransfer]] = None,
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> PoolTransferResult:
        existing_files = self._collect_existing_component_keys(keys, pool_transfers)

        def has_component(page_idx: int, name: str) -> bool:
            return (
                f"{self._get_component_key(keys[page_idx], name)}.bin" in existing_files
            )
# ... omitted for brevity ...
                hit_count[name] = boundary
            final_pages = min(final_pages, boundary)

        return PoolTransferResult(final_pages, hit_count)
```
**EN:** Implements the batch exists V 2 routine for this scope. It belongs to `HiCacheFile`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch exists v 2例程。 该方法属于 `HiCacheFile`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 488-489: _log_key implementation / _log_key 实现
```python
    def _log_key(self, pool_name: str, key: str) -> str:
        return key if pool_name == PoolName.KV else f"{key}.{pool_name}"
```
**EN:** Implements the LOG KEY routine for this scope. It belongs to `HiCacheFile`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的log key例程。 该方法属于 `HiCacheFile`。它会向调用方返回计算结果。

### Lines 491-498: _read_page implementation / _read_page 实现
```python
    def _read_page(self, pool_name: str, key: str, host_pool, page_offset: int) -> bool:
        """Read one page from storage into host_pool at page_offset."""
        storage_key = self._log_key(pool_name, key)
        data_page = self.get(storage_key, host_pool.get_dummy_flat_data_page())
        if data_page is None:
            return False
        host_pool.set_from_flat_data_page(page_offset, data_page)
        return True
```
**EN:** Read one page from storage into host_pool at page_offset. Implements the read page routine for this scope. It belongs to `HiCacheFile`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的read页例程。 该方法属于 `HiCacheFile`。它会向调用方返回计算结果。

### Lines 500-506: _write_page implementation / _write_page 实现
```python
    def _write_page(
        self, pool_name: str, key: str, host_pool, page_offset: int
    ) -> bool:
        """Write one page from host_pool at page_offset to storage as raw bytes."""
        storage_key = self._log_key(pool_name, key)
        data_page = host_pool.get_data_page(page_offset, flat=True)
        return self.set(storage_key, data_page)
```
**EN:** Write one page from host_pool at page_offset to storage as raw bytes. Implements the write page routine for this scope. It belongs to `HiCacheFile`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的write页例程。 该方法属于 `HiCacheFile`。它会向调用方返回计算结果。

### Lines 508-532: _batch_io_v2 implementation / _batch_io_v2 实现
```python
    def _batch_io_v2(self, transfers: List[PoolTransfer], op_fn):
        results: dict[str, List[bool]] = {}
        for transfer in transfers:
            host_pool = self.registered_pools[transfer.name]
            keys = transfer.keys or []
            page_size = getattr(host_pool, "page_size", 1) or 1
            expected = len(keys) * page_size
            host_indices = transfer.host_indices

            if host_indices is None or host_indices.numel() != expected:
                logger.error(
                    "%s indices length mismatch for %s: expected %s, got %s",
# ... omitted for brevity ...
                op_fn(transfer.name, key, host_pool, host_indices[i * page_size].item())
                for i, key in enumerate(keys)
            ]
        return results
```
**EN:** Implements the batch IO V 2 routine for this scope. It belongs to `HiCacheFile`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch IO v 2例程。 该方法属于 `HiCacheFile`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 534-539: batch_get_v2 implementation / batch_get_v2 实现
```python
    def batch_get_v2(
        self,
        transfers: List[PoolTransfer],
        extra_info: Optional["HiCacheStorageExtraInfo"] = None,
    ) -> dict[str, List[bool]]:
        return self._batch_io_v2(transfers, self._read_page)
```
**EN:** Implements the batch get V 2 routine for this scope. It belongs to `HiCacheFile`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get v 2例程。 该方法属于 `HiCacheFile`。它会向调用方返回计算结果。

### Lines 541-546: batch_set_v2 implementation / batch_set_v2 实现
```python
    def batch_set_v2(
        self,
        transfers: List[PoolTransfer],
        extra_info: Optional["HiCacheStorageExtraInfo"] = None,
    ) -> dict[str, List[bool]]:
        return self._batch_io_v2(transfers, self._write_page)
```
**EN:** Implements the batch set V 2 routine for this scope. It belongs to `HiCacheFile`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch set v 2例程。 该方法属于 `HiCacheFile`。它会向调用方返回计算结果。

### Lines 548-558: clear implementation / clear 实现
```python
    def clear(self) -> bool:
        try:
            for filename in os.listdir(self.file_path):
                file_path = os.path.join(self.file_path, filename)
                if os.path.isfile(file_path):
                    os.remove(file_path)
            logger.info("Cleared all entries in HiCacheFile storage.")
            return True
        except Exception as e:
            logger.error(f"Failed to clear HiCacheFile storage: {e}")
            return False
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `HiCacheFile`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `HiCacheFile`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`HiCacheStorageConfig`**: Defines the `HiCacheStorageConfig` type and its core responsibilities. / 定义 `HiCacheStorageConfig` 类型及其核心职责。
- **`HiCacheStorageExtraInfo`**: Defines the `HiCacheStorageExtraInfo` type and its core responsibilities. / 定义 `HiCacheStorageExtraInfo` 类型及其核心职责。
- **`PrefetchTimeoutConfig`**: Defines the `PrefetchTimeoutConfig` type and its core responsibilities. / 定义 `PrefetchTimeoutConfig` 类型及其核心职责。
- **`PoolName`**: Defines the `PoolName` type and its core responsibilities. / 定义 `PoolName` 类型及其核心职责。
- **`PoolHitPolicy`**: Defines the `PoolHitPolicy` type and its core responsibilities. / 定义 `PoolHitPolicy` 类型及其核心职责。
- **`PoolTransfer`**: Defines the `PoolTransfer` type and its core responsibilities. / 定义 `PoolTransfer` 类型及其核心职责。
- **`SidecarPoolSpec`**: Defines the `SidecarPoolSpec` type and its core responsibilities. / 定义 `SidecarPoolSpec` 类型及其核心职责。
- **`PoolTransferResult`**: Defines the `PoolTransferResult` type and its core responsibilities. / 定义 `PoolTransferResult` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `logging`, `os`, `abc`, `dataclasses`, `enum`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.environ`, `sglang.srt.mem_cache.memory_pool_host`
