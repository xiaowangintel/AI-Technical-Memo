# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/kv_offload/base.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Core abstractions for KV cache offloading in vLLM v1. / 该模块位于 `kv_offload` 子系统，主要围绕 `make_offload_key`, `get_offload_block_hash`, `get_offload_group_idx` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""
Core abstractions for KV cache offloading in vLLM v1.
"""

from abc import ABC, abstractmethod
from collections.abc import Collection, Iterable, Iterator, Sequence
from dataclasses import dataclass
from typing import TYPE_CHECKING, Any, NewType

import numpy as np
import torch

from vllm.logger import init_logger

if TYPE_CHECKING:
    from vllm.config import VllmConfig
    from vllm.v1.kv_cache_interface import KVCacheConfig
    from vllm.v1.kv_offload.worker.worker import OffloadingHandler

# `OffloadKey` identifies an offloaded block. It combines a block hash with
# its KV cache group index, encoded as raw bytes to avoid tuple GC overhead.
# Use the helper functions below to construct / decompose keys.
OffloadKey = NewType("OffloadKey", bytes)

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `OffloadKey`, `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `OffloadKey`, `logger`。

### `make_offload_key` function / `make_offload_key` 函数
```python
def make_offload_key(block_hash: bytes, group_idx: int) -> OffloadKey:
    """Pack a block hash and group index into an `OffloadKey`."""
    return OffloadKey(block_hash + group_idx.to_bytes(4, "big", signed=False))
```
**EN:** This function implements `make_offload_key` within the module. The docstring frames it as: Pack a block hash and group index into an `OffloadKey`. Key calls include `OffloadKey`, `to_bytes`.
**CN:** 该函数会实现 `make_offload_key`，其作用域位于the module。 关键调用包括 `OffloadKey`, `to_bytes`。

### `get_offload_block_hash` function / `get_offload_block_hash` 函数
```python
def get_offload_block_hash(key: OffloadKey) -> bytes:
    """Extract the block hash from an `OffloadKey`."""
    return key[:-4]
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Extract the block hash from an `OffloadKey`.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。

### `get_offload_group_idx` function / `get_offload_group_idx` 函数
```python
def get_offload_group_idx(key: OffloadKey) -> int:
    """Extract the group index from an `OffloadKey`."""
    return int.from_bytes(key[-4:], "big", signed=False)
```
**EN:** This function returns or derives a value within the module. The docstring frames it as: Extract the group index from an `OffloadKey`. Key calls include `from_bytes`.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `from_bytes`。

### `ReqContext` class / `ReqContext` 类
```python
@dataclass
class ReqContext:
    req_id: str
    kv_transfer_params: dict[str, Any] | None = None
```
**EN:** Uses `@dataclass` to package related state for `ReqContext`. Typical fields include `req_id`, `kv_transfer_params`.
**CN:** `ReqContext` 使用 `@dataclass` 打包相关状态。典型字段包括 `req_id`, `kv_transfer_params`。

### `LoadStoreSpec` class / `LoadStoreSpec` 类
```python
class LoadStoreSpec(ABC):
    """
    Abstract metadata that encapsulates information allowing a worker
    to load, and optionally also to store, blocks of KV data.
    """
```
**EN:** Declares the `LoadStoreSpec` interface. Downstream implementations are expected to provide methods such as `medium`.
**CN:** `LoadStoreSpec` 声明了一组接口约定。下游实现需要提供 `medium` 等方法。

### `LoadStoreSpec.medium` method / `LoadStoreSpec.medium` 方法
```python
    @staticmethod
    @abstractmethod
    def medium() -> str:
        """
        Returns a string representation of the medium type
        this store/load targets.
        """
        pass
```
**EN:** This method implements `medium` within `LoadStoreSpec`. The docstring frames it as: Returns a string representation of the medium type this store/load targets.
**CN:** 该方法会实现 `medium`，其作用域位于`LoadStoreSpec`。

### `PrepareStoreOutput` class / `PrepareStoreOutput` 类
```python
@dataclass
class PrepareStoreOutput:
    keys_to_store: list[OffloadKey]
    store_spec: LoadStoreSpec
    evicted_keys: list[OffloadKey]
```
**EN:** Uses `@dataclass` to package related state for `PrepareStoreOutput`. Typical fields include `keys_to_store`, `store_spec`, `evicted_keys`.
**CN:** `PrepareStoreOutput` 使用 `@dataclass` 打包相关状态。典型字段包括 `keys_to_store`, `store_spec`, `evicted_keys`。

### `OffloadingEvent` class / `OffloadingEvent` 类
```python
@dataclass
class OffloadingEvent:
    keys: list[OffloadKey]
    medium: str
    # True if blocks are removed, False if stored
    removed: bool
```
**EN:** Uses `@dataclass` to package related state for `OffloadingEvent`. Typical fields include `keys`, `medium`, `removed`.
**CN:** `OffloadingEvent` 使用 `@dataclass` 打包相关状态。典型字段包括 `keys`, `medium`, `removed`。

### Module setup / 模块初始化
```python
"""
OffloadingManager class for managing KV data offloading in vLLM v1

This class runs in the scheduler, tracks which blocks are offloaded
and their address.

The class provides the following primitives:
    lookup() - check whether a single block is offloaded and ready.
    prepare_load() - prepare given blocks to be read.
        The given blocks will be protected from eviction.
        This function returns a LoadSpec which encapsulates
        information required for performing the load.
    touch() - marks the give blocks as recently used. Can be used
        to track block's LRU. This function is separated from the
        prepare_load function to allow setting block recency even
        for blocks which do not need reading from the cache, such as
        blocks that are cached by the GPU prefix cache.
    complete_load() - mark blocks which were previously prepared to be
        loaded as done loading. This is to re-allow their eviction.
    prepare_store() - prepare the given blocks to be written.
        Returns a StoreSpec encapsulating offloading information,
        as well as a list of blocks that were evicted as a result.
    complete_store() - marks a previous store as completed.
        Following this call, the given blocks will become loadable.
"""
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `OffloadingManager` class / `OffloadingManager` 类
```python
class OffloadingManager(ABC):
```
**EN:** Declares the `OffloadingManager` interface. Downstream implementations are expected to provide methods such as `lookup`, `prepare_load`, `touch`, `complete_load`, `prepare_store`, `complete_store`.
**CN:** `OffloadingManager` 声明了一组接口约定。下游实现需要提供 `lookup`, `prepare_load`, `touch`, `complete_load`, `prepare_store`, `complete_store` 等方法。

### `OffloadingManager.prepare_load` method / `OffloadingManager.prepare_load` 方法
```python
    @abstractmethod
    def prepare_load(
        self,
        keys: Collection[OffloadKey],
        req_context: ReqContext,
    ) -> LoadStoreSpec:
        """
        Prepare the given blocks to be read.
        The given blocks will be protected from eviction until
        complete_load is called.
        It assumes all given blocks are offloaded.

        Args:
            keys: the keys identifying the blocks.
            req_context: per-request context (e.g. kv_transfer_params).

        Returns:
            A LoadStoreSpec that can be used by a worker to locate and load
            the actual offloaded KV data.
        """
        pass
```
**EN:** This method prepares inputs and state within `OffloadingManager`. The docstring frames it as: Prepare the given blocks to be read.
**CN:** 该方法会准备输入与状态，其作用域位于`OffloadingManager`。

### `OffloadingManager.complete_load` method / `OffloadingManager.complete_load` 方法
```python
    def complete_load(self, keys: Collection[OffloadKey], req_context: ReqContext):
        """
        Marks previous blocks that were prepared to load as done loading.

        Args:
            keys: the keys identifying the blocks.
            req_context: per-request context (e.g. kv_transfer_params).
        """
        return
```
**EN:** This method implements `complete_load` within `OffloadingManager`. The docstring frames it as: Marks previous blocks that were prepared to load as done loading.
**CN:** 该方法会实现 `complete_load`，其作用域位于`OffloadingManager`。

### `OffloadingManager.prepare_store` method / `OffloadingManager.prepare_store` 方法
```python
    @abstractmethod
    def prepare_store(
        self,
        keys: Collection[OffloadKey],
        req_context: ReqContext,
    ) -> PrepareStoreOutput | None:
        """
        Prepare the given blocks to be offloaded.
        The given blocks will be protected from eviction until
        complete_store is called.

        Args:
            keys: the keys identifying the blocks.
            req_context: per-request context (e.g. kv_transfer_params).

        Returns:
            A PrepareStoreOutput indicating which blocks need storing,
            where to store them (LoadStoreSpec), and list of blocks that
            were evicted as a result.
            None is returned if the blocks cannot be stored.
        """
        pass
```
**EN:** This method prepares inputs and state within `OffloadingManager`. The docstring frames it as: Prepare the given blocks to be offloaded.
**CN:** 该方法会准备输入与状态，其作用域位于`OffloadingManager`。

### `BlockIDsLoadStoreSpec` class / `BlockIDsLoadStoreSpec` 类
```python
class BlockIDsLoadStoreSpec(LoadStoreSpec, ABC):
    """
    Spec for loading/storing KV blocks from given block numbers.
    """
```
**EN:** Declares the `BlockIDsLoadStoreSpec` interface. Downstream implementations are expected to provide methods such as `__init__`, `__repr__`.
**CN:** `BlockIDsLoadStoreSpec` 声明了一组接口约定。下游实现需要提供 `__init__`, `__repr__` 等方法。

### `BlockIDsLoadStoreSpec.__init__` method / `BlockIDsLoadStoreSpec.__init__` 方法
```python
    def __init__(self, block_ids: list[int]):
        self.block_ids = np.array(block_ids, dtype=np.int64)
```
**EN:** This method initializes the object state within `BlockIDsLoadStoreSpec`. Key calls include `array`. It touches state such as `block_ids`.
**CN:** 该方法会初始化对象状态，其作用域位于`BlockIDsLoadStoreSpec`。 关键调用包括 `array`。 它会读写 `block_ids` 等状态。

### `BlockIDsLoadStoreSpec.__repr__` method / `BlockIDsLoadStoreSpec.__repr__` 方法
```python
    def __repr__(self) -> str:
        return repr(self.block_ids)
```
**EN:** This method implements `__repr__` within `BlockIDsLoadStoreSpec`. Key calls include `repr`.
**CN:** 该方法会实现 `__repr__`，其作用域位于`BlockIDsLoadStoreSpec`。 关键调用包括 `repr`。

### `GPULoadStoreSpec` class / `GPULoadStoreSpec` 类
```python
class GPULoadStoreSpec(BlockIDsLoadStoreSpec):
    """
    Spec for loading/storing a KV block to GPU memory.

    If there are multiple KV groups, the blocks are expected to be
    ordered by the group index.
    In that case, group_sizes[i] determines the number of blocks
    per the i-th KV group, and thus sum(group_sizes) == len(block_ids).
    group_sizes=None indicates a single KV group.

    If block_indices is given, each group (determined by group_sizes) of block IDs
    will correspond to logically contiguous blocks, e.g. blocks 5-10 of a some request.
    block_indices[i] will represent the block index of the first block in group #i.
    Thus, len(block_indices) == len(group_sizes) = number of KV cache groups.
    This information is required in order to support off/loading from offloaded blocks
    which are larger than GPU blocks.
    In such cases, the first GPU block per each group may be unaligned to the offloaded
    block size, and so knowing block_indices[i] allows the worker to correctly
    skip part of the first matching offloaded block.
    """
```
**EN:** Introduces the `GPULoadStoreSpec` class on top of `BlockIDsLoadStoreSpec`. Core methods include `__init__`, `medium`. Docstring signal: Spec for loading/storing a KV block to GPU memory.
**CN:** 这里定义 `GPULoadStoreSpec` 类，其基类包括 `BlockIDsLoadStoreSpec`。核心方法包括 `__init__`, `medium`。

### `GPULoadStoreSpec.__init__` method / `GPULoadStoreSpec.__init__` 方法
```python
    def __init__(
        self,
        block_ids: list[int],
        group_sizes: Sequence[int],
        block_indices: Sequence[int],
    ):
        super().__init__(block_ids)
        assert sum(group_sizes) == len(block_ids)
        assert len(block_indices) == len(group_sizes)
        self.group_sizes: Sequence[int] = group_sizes
        self.block_indices: Sequence[int] = block_indices
```
**EN:** This method initializes the object state within `GPULoadStoreSpec`. Key calls include `__init__`, `sum`, `len`, `super`. It touches state such as `group_sizes`, `block_indices`.
**CN:** 该方法会初始化对象状态，其作用域位于`GPULoadStoreSpec`。 关键调用包括 `__init__`, `sum`, `len`, `super`。 它会读写 `group_sizes`, `block_indices` 等状态。

### `GPULoadStoreSpec.medium` method / `GPULoadStoreSpec.medium` 方法
```python
    @staticmethod
    def medium() -> str:
        return "GPU"
```
**EN:** This method implements `medium` within `GPULoadStoreSpec`.
**CN:** 该方法会实现 `medium`，其作用域位于`GPULoadStoreSpec`。

### `CanonicalKVCacheTensor` class / `CanonicalKVCacheTensor` 类
```python
@dataclass
class CanonicalKVCacheTensor:
    """
    A canonicalized KV cache tensor whose first dimension is num_blocks.

    For attention backends where the raw tensor has num_blocks at a
    non-leading physical dimension (e.g. FlashAttention's
    (2, num_blocks, ...) layout), the tensor is split so that each
    resulting CanonicalKVCacheTensor starts with (num_blocks, ...).
    """

    # The KV cache tensor with shape (num_blocks, ...)
    tensor: torch.Tensor
    # The (possibly padded) page size per block in bytes
    page_size_bytes: int
```
**EN:** Uses `@dataclass` to package related state for `CanonicalKVCacheTensor`. Typical fields include `tensor`, `page_size_bytes`.
**CN:** `CanonicalKVCacheTensor` 使用 `@dataclass` 打包相关状态。典型字段包括 `tensor`, `page_size_bytes`。

### `CanonicalKVCacheRef` class / `CanonicalKVCacheRef` 类
```python
@dataclass
class CanonicalKVCacheRef:
    """
    Per-layer (or group of layers) reference to a specific (by index)
    CanonicalKVCacheTensor and records the un-padded page size used by that layer.
    """

    # Index into the list of CanonicalKVCacheTensor objects
    tensor_idx: int
    # The un-padded page size per block in bytes
    page_size_bytes: int
```
**EN:** Uses `@dataclass` to package related state for `CanonicalKVCacheRef`. Typical fields include `tensor_idx`, `page_size_bytes`.
**CN:** `CanonicalKVCacheRef` 使用 `@dataclass` 打包相关状态。典型字段包括 `tensor_idx`, `page_size_bytes`。

### `CanonicalKVCaches` class / `CanonicalKVCaches` 类
```python
@dataclass
class CanonicalKVCaches:
    """
    Canonicalized block-level representation of the KV caches.

    Composed of:
        - Unique list of KV cache data tensors,
          each with shape (num_blocks, page_size_in_bytes) and int8 dtype.
        - Per-group data references of the tensors.
          i.e. how each KV cache group maps to the tensors.
    """

    # Ordered list of unique block tensors, each with shape
    # (num_blocks, ...).
    tensors: list[CanonicalKVCacheTensor]
    # Per-KV-cache-group list of data references that map each layer
    # in the group to the appropriate entry in the tensors list.
    group_data_refs: list[list[CanonicalKVCacheRef]]
```
**EN:** Uses `@dataclass` to package related state for `CanonicalKVCaches`. Typical fields include `tensors`, `group_data_refs`.
**CN:** `CanonicalKVCaches` 使用 `@dataclass` 打包相关状态。典型字段包括 `tensors`, `group_data_refs`。

### `OffloadingSpec` class / `OffloadingSpec` 类
```python
class OffloadingSpec(ABC):
    """Spec for an offloading connector"""
```
**EN:** Declares the `OffloadingSpec` interface. Downstream implementations are expected to provide methods such as `__init__`, `get_manager`, `get_handlers`.
**CN:** `OffloadingSpec` 声明了一组接口约定。下游实现需要提供 `__init__`, `get_manager`, `get_handlers` 等方法。

### `OffloadingSpec.__init__` method / `OffloadingSpec.__init__` 方法
```python
    def __init__(self, vllm_config: "VllmConfig", kv_cache_config: "KVCacheConfig"):
        logger.warning(
            "Initializing OffloadingSpec. This API is experimental and "
            "subject to change in the future as we iterate the design."
        )
        self.vllm_config = vllm_config
        self.kv_cache_config = kv_cache_config

        kv_transfer_config = vllm_config.kv_transfer_config
        assert kv_transfer_config is not None
        self.extra_config = kv_transfer_config.kv_connector_extra_config

        parallel_config = vllm_config.parallel_config
        context_parallel_factor = (
            parallel_config.decode_context_parallel_size
            * parallel_config.prefill_context_parallel_size
        )

        # block size used by vLLM for hashing request tokens for the sake
        # of enabling prefix caching
        self.hash_block_size = (
            vllm_config.cache_config.block_size * context_parallel_factor
        )
        # gpu block size per group
        self.gpu_block_size: tuple[int, ...] = tuple(
            kv_cache_group.kv_cache_spec.block_size * context_parallel_factor
            for kv_cache_group in kv_cache_config.kv_cache_groups
        )

        for block_size in self.gpu_block_size:
            assert block_size % self.hash_block_size == 0, (
                f"gpu_block_size={block_size} not divisible by "
                f"hash_block_size={self.hash_block_size}. "
                f"Hybrid models (e.g. Mamba+Attention) need "
                f"--enable-prefix-caching to align block sizes."
            )

        # offloaded_block_size / gpu_block_size
        self.block_size_factor: int = 1

        offloaded_block_size = self.extra_config.get("block_size")
        if offloaded_block_size is not None:
            offloaded_block_size_int = int(offloaded_block_size)
            gpu_block_sizes = set(self.gpu_block_size)
            assert len(gpu_block_sizes) == 1, (
                "If 'block_size' is specified in kv_connector_extra_config, "
                "there must be at least one KV cache group, "
                "and all groups must have the same block size."
            )
            gpu_block_size = gpu_block_sizes.pop()

            assert offloaded_block_size_int % gpu_block_size == 0
            self.block_size_factor = offloaded_block_size_int // gpu_block_size
```
**EN:** This method initializes the object state within `OffloadingSpec`. Key calls include `warning`, `tuple`, `get`, `int`, `set`, `pop`. It touches state such as `vllm_config`, `kv_cache_config`, `extra_config`, `hash_block_size`, `gpu_block_size`, `block_size_factor`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`OffloadingSpec`。 关键调用包括 `warning`, `tuple`, `get`, `int`, `set`, `pop`。 它会读写 `vllm_config`, `kv_cache_config`, `extra_config`, `hash_block_size`, `gpu_block_size`, `block_size_factor` 等状态。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `OffloadingSpec.get_manager` method / `OffloadingSpec.get_manager` 方法
```python
    @abstractmethod
    def get_manager(self) -> OffloadingManager:
        """
        Get an OffloadingManager that will be used
        by the scheduler-side offloading connector to track
        offloaded blocks and manage evictions.
        """
        pass
```
**EN:** This method returns or derives a value within `OffloadingSpec`. The docstring frames it as: Get an OffloadingManager that will be used by the scheduler-side offloading connector to track offloaded blocks and manage evictions.
**CN:** 该方法会返回或推导一个值，其作用域位于`OffloadingSpec`。

### `OffloadingSpec.get_handlers` method / `OffloadingSpec.get_handlers` 方法
```python
    @abstractmethod
    def get_handlers(
        self, kv_caches: CanonicalKVCaches
    ) -> Iterator[tuple[type[LoadStoreSpec], type[LoadStoreSpec], "OffloadingHandler"]]:
        """
        Get offloading handlers along with their respective src and dst types.

        Args:
            kv_caches: Canonicalized KV caches.

        Yields:
            Tuples of (src_type, dst_type, offloading_handler).
        """
        pass
```
**EN:** This method returns or derives a value within `OffloadingSpec`. The docstring frames it as: Get offloading handlers along with their respective src and dst types.
**CN:** 该方法会返回或推导一个值，其作用域位于`OffloadingSpec`。

## Key Concepts / 关键概念
- `make_offload_key`: top-level helper or orchestration entry point. / `make_offload_key`：顶层辅助函数或编排入口。
- `get_offload_block_hash`: top-level helper or orchestration entry point. / `get_offload_block_hash`：顶层辅助函数或编排入口。
- `get_offload_group_idx`: top-level helper or orchestration entry point. / `get_offload_group_idx`：顶层辅助函数或编排入口。
- `ReqContext`: central class or interface in this module. / `ReqContext`：本模块中的核心类或接口。
- `LoadStoreSpec`: central class or interface in this module. / `LoadStoreSpec`：本模块中的核心类或接口。
- `PrepareStoreOutput`: central class or interface in this module. / `PrepareStoreOutput`：本模块中的核心类或接口。
- `OffloadingEvent`: central class or interface in this module. / `OffloadingEvent`：本模块中的核心类或接口。
- `OffloadingManager`: central class or interface in this module. / `OffloadingManager`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `abc`, `collections`, `dataclasses`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.config`, `vllm.v1.kv_cache_interface`, `vllm.v1.kv_offload.worker.worker`
