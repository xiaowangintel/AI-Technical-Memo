# mooncake_store.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/storage/mooncake_store/mooncake_store.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the mooncake store logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的Mooncake store相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Imports and setup / 导入与初始化
```python
import ctypes
import json
import logging
import os
import time
import uuid
from dataclasses import dataclass
from typing import Any, List, Optional, Tuple

import requests
import torch

# ... omitted for brevity ...
    PoolTransferResult,
)
from sglang.srt.mem_cache.memory_pool_host import HostKVCache, HostTensorAllocator
from sglang.srt.observability.metrics_collector import StorageMetrics
```
**EN:** Imports `ctypes`, `json`, `logging`, `os`, `time`, `uuid` and other helpers used by the surrounding scope.
**CN:** 导入 `ctypes`, `json`, `logging`, `os`, `time`, `uuid` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 26-29: Shared state definitions / 共享状态定义
```python
DEFAULT_LOCAL_BUFFER_SIZE = 16 * 1024 * 1024  # 16 MB
SETUP_TIMEOUT = 600  # 10min

logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `DEFAULT_LOCAL_BUFFER_SIZE`, `SETUP_TIMEOUT`, `logger`.
**CN:** 定义模块级变量，例如 `DEFAULT_LOCAL_BUFFER_SIZE`, `SETUP_TIMEOUT`, `logger`。

### Lines 32-32: MooncakeHostTensorAllocator declaration / MooncakeHostTensorAllocator 声明
```python
class MooncakeHostTensorAllocator(HostTensorAllocator):
```
**EN:** Declares the `MooncakeHostTensorAllocator` class and connects it to `HostTensorAllocator`.
**CN:** 声明 `MooncakeHostTensorAllocator` 类，并将其关联到 `HostTensorAllocator`。

### Lines 33-38: __init__ implementation / __init__ 实现
```python
    def __init__(self):
        super().__init__()
        from mooncake.store import MooncakeHostMemAllocator

        self.allocator = MooncakeHostMemAllocator()
        self.ptr = None
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `MooncakeHostTensorAllocator`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `MooncakeHostTensorAllocator`。

### Lines 40-64: allocate implementation / allocate 实现
```python
    def allocate(
        self, dims: tuple, dtype: torch.dtype, device: str = "cpu"
    ) -> torch.Tensor:
        """
        Allocates memory using MooncakeHostMemAllocator and wraps it in a PyTorch tensor.
        """
        self.dims = dims
        self.dtype = dtype
        size = 1
        for d in dims:
            size *= d
        size *= torch.tensor([], dtype=self.dtype).element_size()
# ... omitted for brevity ...
            assert size % element_size == 0, "Size must be divisible by element size"
            tensor = tensor.view(dtype)

        return tensor.view(dims)
```
**EN:** Allocates memory using MooncakeHostMemAllocator and wraps it in a PyTorch tensor. Implements the allocate routine for this scope. It belongs to `MooncakeHostTensorAllocator`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的allocate例程。 该方法属于 `MooncakeHostTensorAllocator`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 67-80: _parse_global_segment_size implementation / _parse_global_segment_size 实现
```python
def _parse_global_segment_size(value) -> int:
    if isinstance(value, int):
        return value
    if isinstance(value, str):
        s = value.strip().lower()
        if s.endswith("gb"):
            num = s[:-2].strip()
            if not num:
                raise ValueError(
                    "Invalid global_segment_size: missing number before 'gb'"
                )
            return int(num) * 1024 * 1024 * 1024
        return int(s)
    return int(value)
```
**EN:** Implements the parse global segment size routine for this scope. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的parse global segment size例程。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 83-84: MooncakeStoreConfig declaration / MooncakeStoreConfig 声明
```python
@dataclass
class MooncakeStoreConfig:
```
**EN:** Declares the `MooncakeStoreConfig` class.
**CN:** 声明 `MooncakeStoreConfig` 类。

### Lines 85-96: Shared state definitions / 共享状态定义
```python
    local_hostname: str
    metadata_server: str
    global_segment_size: int
    protocol: str
    device_name: str
    master_server_address: str
    master_metrics_port: int
    check_server: bool
    standalone_storage: bool
    client_server_address: str
    enable_ssd_offload: bool = False
    ssd_offload_path: Optional[str] = None
```
**EN:** Defines class-level variables such as `local_hostname`, `metadata_server`, `global_segment_size`, `protocol`, `device_name`.
**CN:** 定义类级变量，例如 `local_hostname`, `metadata_server`, `global_segment_size`, `protocol`, `device_name`。

### Lines 98-153: from_file implementation / from_file 实现
```python
    @staticmethod
    def from_file() -> "MooncakeStoreConfig":
        """Load the config from a JSON file."""
        if not envs.SGLANG_HICACHE_MOONCAKE_CONFIG_PATH.is_set():
            raise RuntimeError(
                f"Config file path not set. Please set {envs.SGLANG_HICACHE_MOONCAKE_CONFIG_PATH.name}"
            )
        file_path = envs.SGLANG_HICACHE_MOONCAKE_CONFIG_PATH.get()
        try:
            with open(file_path) as fin:
                config = json.load(fin)
        except Exception as e:
# ... omitted for brevity ...
            ssd_offload_path=config.get(
                "ssd_offload_path", envs.MOONCAKE_OFFLOAD_FILE_STORAGE_PATH.default
            ),
        )
```
**EN:** Load the config from a JSON file. Provides an alternative constructor from existing inputs. It belongs to `MooncakeStoreConfig`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 提供基于现有输入的备用构造方式。 该方法属于 `MooncakeStoreConfig`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 155-194: load_from_env implementation / load_from_env 实现
```python
    @staticmethod
    def load_from_env() -> "MooncakeStoreConfig":
        """Load config from a file specified in the environment variable.
        export MOONCAKE_MASTER=10.13.3.232:50051
        export MOONCAKE_PROTOCOL="rdma"
        export MOONCAKE_DEVICE=""
        export MOONCAKE_TE_META_DATA_SERVER="P2PHANDSHAKE"
        """
        # other required environment variables...
        if not envs.MOONCAKE_MASTER.is_set() and not envs.MOONCAKE_CLIENT.is_set():
            raise ValueError(
                "Either the environment variable 'MOONCAKE_MASTER' or 'MOONCAKE_CLIENT' is not set."
# ... omitted for brevity ...
            client_server_address=envs.MOONCAKE_CLIENT.get(),
            enable_ssd_offload=envs.MOONCAKE_ENABLE_SSD_OFFLOAD.get(),
            ssd_offload_path=envs.MOONCAKE_OFFLOAD_FILE_STORAGE_PATH.get(),
        )
```
**EN:** Load config from a file specified in the environment variable. Loads state from an external or serialized representation. It belongs to `MooncakeStoreConfig`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `MooncakeStoreConfig`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 196-242: load_from_extra_config implementation / load_from_extra_config 实现
```python
    @staticmethod
    def load_from_extra_config(extra_config: dict) -> "MooncakeStoreConfig":
        """Load config from extra_config dictionary."""
        if (
            "master_server_address" not in extra_config
            and "client_server_address" not in extra_config
        ):
            raise ValueError(
                "Either master_server_address or client_server_address is required in extra_config"
            )

        return MooncakeStoreConfig(
# ... omitted for brevity ...
            ssd_offload_path=extra_config.get(
                "ssd_offload_path", envs.MOONCAKE_OFFLOAD_FILE_STORAGE_PATH.default
            ),
        )
```
**EN:** Load config from extra_config dictionary. Loads state from an external or serialized representation. It belongs to `MooncakeStoreConfig`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `MooncakeStoreConfig`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 245-245: MooncakeBaseStore declaration / MooncakeBaseStore 声明
```python
class MooncakeBaseStore:
```
**EN:** Declares the `MooncakeBaseStore` class.
**CN:** 声明 `MooncakeBaseStore` 类。

### Lines 246-248: __init__ implementation / __init__ 实现
```python
    def __init__(self):
        self.store = None
        self.config = None
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `MooncakeBaseStore`.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `MooncakeBaseStore`。

### Lines 250-260: _import_mooncake_store implementation / _import_mooncake_store 实现
```python
    def _import_mooncake_store(self):
        try:
            from mooncake.store import MooncakeDistributedStore

            return MooncakeDistributedStore
        except ImportError as e:
            raise ImportError(
                "Please install mooncake by following the instructions at "
                "https://kvcache-ai.github.io/Mooncake/getting_started/build.html "
                "to run SGLang with MooncakeConnector."
            ) from e
```
**EN:** Implements the import mooncake store routine for this scope. It belongs to `MooncakeBaseStore`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的import Mooncake store例程。 该方法属于 `MooncakeBaseStore`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 262-282: _load_config implementation / _load_config 实现
```python
    def _load_config(self, storage_config: Any = None):
        extra_config = (
            getattr(storage_config, "extra_config", None) if storage_config else None
        )

        if extra_config and (
            extra_config.get("master_server_address") is not None
            or extra_config.get("client_server_address") is not None
        ):
            config = MooncakeStoreConfig.load_from_extra_config(extra_config)
            logger.info("Mooncake Configuration loaded from extra_config successfully.")

# ... omitted for brevity ...
            config = MooncakeStoreConfig.load_from_env()
            logger.info("Mooncake Configuration loaded from env successfully.")

        return config
```
**EN:** Implements the load config routine for this scope. It belongs to `MooncakeBaseStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的load config例程。 该方法属于 `MooncakeBaseStore`。它会向调用方返回计算结果。

### Lines 284-294: register_buffer implementation / register_buffer 实现
```python
    def register_buffer(self, tensor: torch.Tensor):
        if self.store is None:
            raise RuntimeError("Mooncake store is not initialized.")
        ptr = tensor.data_ptr()
        size = tensor.numel() * tensor.element_size()
        ret_code = self.store.register_buffer(ptr, size)
        if ret_code != 0:
            logger.error(f"Failed to register buffer, error code: {ret_code}")
            raise RuntimeError(
                f"Failed to register buffer to Mooncake Store, error code: {ret_code}"
            )
```
**EN:** Registers metadata so other components can discover this object. It belongs to `MooncakeBaseStore`. It validates error cases explicitly.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `MooncakeBaseStore`。它会显式处理错误场景。

### Lines 297-298: MooncakeStore declaration / MooncakeStore 声明
```python
class MooncakeStore(HiCacheStorage, MooncakeBaseStore):

```
**EN:** Declares the `MooncakeStore` class and connects it to `HiCacheStorage`, `MooncakeBaseStore`.
**CN:** 声明 `MooncakeStore` 类，并将其关联到 `HiCacheStorage`, `MooncakeBaseStore`。

### Lines 299-490: __init__ implementation / __init__ 实现
```python
    def __init__(
        self, storage_config: HiCacheStorageConfig = None, mem_pool: HostKVCache = None
    ):
        MooncakeBaseStore.__init__(self)
        MooncakeDistributedStore = self._import_mooncake_store()
        try:
            self.store = MooncakeDistributedStore()

            self.config = self._load_config(storage_config)
            extra_config = (
                getattr(storage_config, "extra_config", None)
                if storage_config
# ... omitted for brevity ...
            raise
        except Exception as exc:
            logger.error("An error occurred while loading the configuration: %s", exc)
            raise
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `MooncakeStore`. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `MooncakeStore`。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 492-523: check_server implementation / check_server 实现
```python
    def check_server(self):
        master_server_ip = self.config.master_server_address.split(":")[0]
        segments_url = f"http://{master_server_ip}:{self.config.master_metrics_port}/get_all_segments"
        start_time = time.perf_counter()

        check_result = False
        while time.perf_counter() - start_time < SETUP_TIMEOUT:
            try:
                check_segments_resp = requests.get(segments_url, timeout=3)
            except Exception:
                logger.info(
                    "waiting mooncake store server started, cost_time: %.2f seconds.",
# ... omitted for brevity ...

        if not check_result:
            logger.error("Launch mooncake store server timeout")
            raise ValueError("Launch mooncake store server timeout")
```
**EN:** Implements the check server routine for this scope. It belongs to `MooncakeStore`. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的check服务器例程。 该方法属于 `MooncakeStore`。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 525-549: warmup implementation / warmup 实现
```python
    def warmup(self):
        warmup_key = "sglang_mooncake_store_warmup_key" + uuid.uuid4().hex
        warmup_value = bytes(4 * 1024)  # 4 KB

        # Retry logic to handle Transfer Engine startup race condition
        max_retries = 10
        retry_delay = 1.0  # seconds

        for attempt in range(max_retries):
            ret = self.store.put(warmup_key, warmup_value)
            if ret == 0:
                break
# ... omitted for brevity ...
            )

        assert self.store.is_exist(warmup_key) == 1
        assert self.store.get(warmup_key) == warmup_value
```
**EN:** Implements the warmup routine for this scope. It belongs to `MooncakeStore`. It validates error cases explicitly. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的warmup例程。 该方法属于 `MooncakeStore`。它会显式处理错误场景。实现过程中会遍历输入或受管条目。

### Lines 551-567: register_mem_pool_host implementation / register_mem_pool_host 实现
```python
    def register_mem_pool_host(self, mem_pool_host: HostKVCache):
        super().register_mem_pool_host(mem_pool_host)
        assert self.mem_pool_host.layout in [
            "page_first",
            "page_first_direct",
            "page_head",
            "page_first_kv_split",
        ], "mooncake store storage backend only support page first, page first direct, page head and  page_first_kv_split layout"
        buffer = self.mem_pool_host.kv_buffer
        try:
            super().register_buffer(buffer)
        except TypeError as err:
            logger.error("Failed to register buffer to Mooncake Store: %s", err)
            raise TypeError("Mooncake Store Register Buffer Error.") from err

        bytes_per_page = mem_pool_host.get_ksize_per_token() * mem_pool_host.page_size
        self.gb_per_page = bytes_per_page / (1 << 30)
```
**EN:** Registers metadata so other components can discover this object. It belongs to `MooncakeStore`. It validates error cases explicitly.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `MooncakeStore`。它会显式处理错误场景。

### Lines 569-582: register_mem_host_pool_v2 implementation / register_mem_host_pool_v2 实现
```python
    def register_mem_host_pool_v2(self, host_pool: HostKVCache, host_pool_name):
        # KV anchor memory is already registered via register_mem_pool_host().
        # v2 here only registers additional hybrid pools.
        if host_pool_name == PoolName.KV:
            return
        # Keep a name->pool mapping so batch v2 can resolve PoolTransfer.name to
        # the corresponding host pool implementation at runtime.
        self.registered_pools[host_pool_name] = host_pool

        # Hybrid pools expose the tensors that Mooncake needs for zero-copy I/O.
        # The storage backend only depends on this accessor, not concrete fields.
        buf_list = host_pool.get_hybrid_pool_buffer()
        for buf in buf_list:
            super().register_buffer(buf)
```
**EN:** Registers metadata so other components can discover this object. It belongs to `MooncakeStore`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `MooncakeStore`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 584-587: _tag_keys implementation / _tag_keys 实现
```python
    def _tag_keys(self, keys: List[str]) -> List[str]:
        if self.extra_backend_tag is None:
            return keys
        return [f"{ self.extra_backend_tag}_{key}" for key in keys]
```
**EN:** Implements the TAG keys routine for this scope. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的tag keys例程。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 589-612: _get_hybrid_page_component_keys implementation / _get_hybrid_page_component_keys 实现
```python
    def _get_hybrid_page_component_keys(
        self, page_keys: List[str], transfer: PoolTransfer
    ) -> Tuple[List[str], int]:
        # A logical "page" may map to multiple physical objects in storage.
        # - INDEXER: one key per page
        # - MAMBA  : one temporal key + N conv keys per page
        # key_multiplier records how many component keys are generated per page.
        name = transfer.name
        suffixes = []
        if name == PoolName.INDEXER:
            suffixes = [f"_{self.mla_suffix}_{PoolName.INDEXER}"]
        elif name == PoolName.MAMBA:
# ... omitted for brevity ...
        component_keys = [
            f"{page_key}{suffix}" for page_key in page_keys for suffix in suffixes
        ]
        return component_keys, key_multiplier
```
**EN:** Implements the get hybrid page component keys routine for this scope. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get混合页组件keys例程。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 614-662: batch_exists_v2 implementation / batch_exists_v2 实现
```python
    def batch_exists_v2(
        self,
        keys: List[str],
        pool_transfers: Optional[List[PoolTransfer]] = None,
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> PoolTransferResult:
        qkeys = self._tag_keys(keys)
        kv_pages = self.batch_exists(qkeys, extra_info)

        hit_count: dict = {PoolName.KV: kv_pages} if kv_pages else {}
        final_pages = kv_pages

# ... omitted for brevity ...
                hit_count[transfer.name] = boundary
            final_pages = min(final_pages, boundary)

        return PoolTransferResult(final_pages, hit_count)
```
**EN:** Implements the batch exists V 2 routine for this scope. It belongs to `MooncakeStore`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch exists v 2例程。 该方法属于 `MooncakeStore`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 664-701: _batch_io_v2 implementation / _batch_io_v2 实现
```python
    def _batch_io_v2(self, transfers: List[PoolTransfer], is_set: bool):
        # Unified v2 I/O path: each PoolTransfer can expand to one or more
        # storage objects per logical page, but API still reports page-level result.
        results: dict = {}
        for transfer in transfers:
            host_pool = getattr(self, "registered_pools", {}).get(transfer.name)
            keys = transfer.keys
            page_size = getattr(host_pool, "page_size", 1) or 1
            host_indices = transfer.host_indices
            assert len(keys) > 0
            assert len(keys) == len(host_indices) // page_size

# ... omitted for brevity ...
            results[transfer.name] = self._batch_postprocess(
                io_results, is_set_operate=is_set, key_multiplier=key_multiplier
            )
        return results
```
**EN:** Implements the batch IO V 2 routine for this scope. It belongs to `MooncakeStore`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch IO v 2例程。 该方法属于 `MooncakeStore`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 703-708: batch_get_v2 implementation / batch_get_v2 实现
```python
    def batch_get_v2(
        self,
        transfers: List[PoolTransfer],
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> dict:
        return self._batch_io_v2(transfers, is_set=False)
```
**EN:** Implements the batch get V 2 routine for this scope. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get v 2例程。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 710-715: batch_set_v2 implementation / batch_set_v2 实现
```python
    def batch_set_v2(
        self,
        transfers: List[PoolTransfer],
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> dict:
        return self._batch_io_v2(transfers, is_set=True)
```
**EN:** Implements the batch set V 2 routine for this scope. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch set v 2例程。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 717-729: _get_mha_split_heads_buffer_meta implementation / _get_mha_split_heads_buffer_meta 实现
```python
    def _get_mha_split_heads_buffer_meta(self, keys, indices):
        ptr_list, element_size_list = (
            self.mem_pool_host.get_split_heads_page_buffer_meta(
                indices, self.split_factor
            )
        )
        key_list = []
        for key_ in keys:
            for suffix in self.mha_suffix:
                key_list.append(f"{key_}_{suffix}_k")
                key_list.append(f"{key_}_{suffix}_v")
        assert len(key_list) == len(ptr_list)
        return key_list, ptr_list, element_size_list
```
**EN:** Implements the get MHA split heads buffer meta routine for this scope. It belongs to `MooncakeStore`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get mha split heads buffer meta例程。 该方法属于 `MooncakeStore`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 731-738: _get_mha_buffer_meta implementation / _get_mha_buffer_meta 实现
```python
    def _get_mha_buffer_meta(self, keys, indices):
        ptr_list, element_size_list = self.mem_pool_host.get_page_buffer_meta(indices)
        key_list = []
        for key_ in keys:
            key_list.append(f"{key_}_{self.mha_suffix}_k")
            key_list.append(f"{key_}_{self.mha_suffix}_v")
        assert len(key_list) == len(ptr_list)
        return key_list, ptr_list, element_size_list
```
**EN:** Implements the get MHA buffer meta routine for this scope. It belongs to `MooncakeStore`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get mha buffer meta例程。 该方法属于 `MooncakeStore`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 740-746: _get_mla_buffer_meta implementation / _get_mla_buffer_meta 实现
```python
    def _get_mla_buffer_meta(self, keys, indices):
        ptr_list, element_size_list = self.mem_pool_host.get_page_buffer_meta(indices)
        key_list = []
        for key_ in keys:
            key_list.append(f"{key_}_{self.mla_suffix}_k")
        assert len(key_list) == len(ptr_list)
        return key_list, ptr_list, element_size_list
```
**EN:** Implements the get MLA buffer meta routine for this scope. It belongs to `MooncakeStore`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get mla buffer meta例程。 该方法属于 `MooncakeStore`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 748-757: _batch_preprocess implementation / _batch_preprocess 实现
```python
    def _batch_preprocess(self, keys, host_indices):
        assert len(keys) > 0
        assert len(keys) == len(host_indices) // self.mem_pool_host.page_size
        if self.is_mla_backend:
            return self._get_mla_buffer_meta(keys, host_indices)
        else:
            if self.storage_config.should_split_heads:
                return self._get_mha_split_heads_buffer_meta(keys, host_indices)
            else:
                return self._get_mha_buffer_meta(keys, host_indices)
```
**EN:** Implements the batch preprocess routine for this scope. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch preprocess例程。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 759-788: _batch_postprocess implementation / _batch_postprocess 实现
```python
    def _batch_postprocess(
        self, results: List[int], is_set_operate=False, key_multiplier=None
    ):
        """
        refer to https://github.com/kvcache-ai/Mooncake/blob/main/mooncake-store/include/pybind_client.h
        for batch_get_into, results is Vector of integers,
            where each element is the number of bytes read on success, or a negative value on error
        for batch_put_from, results is Vector of integers,
            where each element is 0 on success, or a negative value on error
        """
        if key_multiplier is None:
            if self.is_mla_backend:
# ... omitted for brevity ...
                else all(res > 0 for res in group)
            )
            for group in result_groups
        ]
```
**EN:** refer to https://github.com/kvcache-ai/Mooncake/blob/main/mooncake-store/include/pybind_client.h for batch_get_into, results is Vector of integers, where each element is the... Implements the batch postprocess routine for this scope. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch postprocess例程。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 790-813: batch_get_v1 implementation / batch_get_v1 实现
```python
    def batch_get_v1(
        self,
        keys: List[str],
        host_indices: torch.Tensor,
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> List[bool]:
        # Apply extra_backend_tag prefix if available
        keys = self._tag_keys(keys)

        key_strs, buffer_ptrs, buffer_sizes = self._batch_preprocess(keys, host_indices)

        start_time = time.perf_counter()
# ... omitted for brevity ...
                len(keys) / (end_time - start_time) * self.gb_per_page
            )

        return self._batch_postprocess(get_results, is_set_operate=False)
```
**EN:** Implements the batch get V 1 routine for this scope. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get v 1例程。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 815-858: batch_set_v1 implementation / batch_set_v1 实现
```python
    def batch_set_v1(
        self,
        keys: List[str],
        host_indices: torch.Tensor,
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> List[bool]:
        # Apply extra_backend_tag prefix if available
        keys = self._tag_keys(keys)

        key_strs, buffer_ptrs, buffer_sizes = self._batch_preprocess(keys, host_indices)
        exist_result = self._batch_exist(key_strs)

# ... omitted for brevity ...
            for i in range(len(set_indices)):
                set_results[set_indices[i]] = put_results[i]

        return self._batch_postprocess(set_results, is_set_operate=True)
```
**EN:** Implements the batch set V 1 routine for this scope. It belongs to `MooncakeStore`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch set v 1例程。 该方法属于 `MooncakeStore`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 860-875: set implementation / set 实现
```python
    def set(
        self,
        key,
        value: Optional[Any] = None,
        target_location: Optional[List[int]] = None,
        target_sizes: Optional[List[int]] = None,
    ) -> bool:
        # Only support zero copy set for now
        assert target_location is not None and target_sizes is not None
        exist_result = self._batch_exist([key])
        if exist_result[0] == 1:
            return True
        put_result = self._put_batch_zero_copy_impl(
            [key], [target_location], [target_sizes]
        )
        return put_result[0] == 0
```
**EN:** Updates internal state with the provided value. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 877-933: batch_set implementation / batch_set 实现
```python
    def batch_set(
        self,
        keys: List[str],
        values: Optional[List[torch.Tensor]] = None,
        target_locations: Optional[List[int]] = None,
        target_sizes: Optional[List[int]] = None,
    ) -> bool:
        # Only support zero copy set for now
        assert target_locations is not None and target_sizes is not None
        assert len(keys) == len(target_locations) == len(target_sizes)

        if len(keys) == 0:
# ... omitted for brevity ...
                break
            success_count += 1
        # TODO: return the number of consecutive successful operations from the start.
        return success_count == len(keys)
```
**EN:** Implements the batch set routine for this scope. It belongs to `MooncakeStore`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch set例程。 该方法属于 `MooncakeStore`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 935-945: get implementation / get 实现
```python
    def get(
        self,
        key,
        target_location: Optional[Any] = None,
        target_sizes: Optional[Any] = None,
    ) -> bool:
        assert target_location is not None and target_sizes is not None
        get_result = self._get_batch_zero_copy_impl(
            [key], [target_location], [target_sizes]
        )
        return get_result[0] >= 0
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 947-977: batch_get implementation / batch_get 实现
```python
    def batch_get(
        self,
        keys: List[str],
        target_locations: Optional[Any] = None,
        target_sizes: Optional[Any] = None,
    ) -> int:
        assert len(keys) == len(target_locations) == len(target_sizes)
        if len(keys) == 0:
            return 0

        start_time = time.perf_counter()
        get_result = self._get_batch_zero_copy_impl(
# ... omitted for brevity ...
        for i in range(len(keys)):
            if get_result[i] < 0:
                return i // key_multiplier
        return len(keys) // key_multiplier
```
**EN:** Implements the batch get routine for this scope. It belongs to `MooncakeStore`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get例程。 该方法属于 `MooncakeStore`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 979-981: exists implementation / exists 实现
```python
    def exists(self, key) -> bool:
        exist_result = self._batch_exist([key])
        return exist_result[0] == 1
```
**EN:** Implements the exists routine for this scope. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的exists例程。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 983-1010: batch_exists implementation / batch_exists 实现
```python
    def batch_exists(
        self, keys, extra_info: Optional[HiCacheStorageExtraInfo] = None
    ) -> int:
        # Apply extra_backend_tag prefix if available
        keys = self._tag_keys(keys)

        if self.is_mla_backend:
            query_keys = [f"{key}_{self.mla_suffix}_k" for key in keys]
            key_multiplier = 1
        else:
            query_keys = []
            if self.storage_config.should_split_heads:
# ... omitted for brevity ...
        for i in range(len(query_keys)):
            if exist_result[i] != 1:
                return i // key_multiplier
        return len(query_keys) // key_multiplier
```
**EN:** Implements the batch exists routine for this scope. It belongs to `MooncakeStore`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch exists例程。 该方法属于 `MooncakeStore`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 1012-1015: close implementation / close 实现
```python
    def close(self):
        # MooncakeDistributedStore will automatically call the destructor, so
        # it is unnecessary to close it manually.
        pass
```
**EN:** Closes resources owned by this component. It belongs to `MooncakeStore`.
**CN:** 关闭该组件持有的资源。 该方法属于 `MooncakeStore`。

### Lines 1017-1018: clear implementation / clear 实现
```python
    def clear(self) -> None:
        self.store.remove_all()
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `MooncakeStore`.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `MooncakeStore`。

### Lines 1020-1023: _put_batch_zero_copy_impl implementation / _put_batch_zero_copy_impl 实现
```python
    def _put_batch_zero_copy_impl(
        self, key_strs: List[str], buffer_ptrs: List[int], buffer_sizes: List[int]
    ) -> List[int]:
        return self.store.batch_put_from(key_strs, buffer_ptrs, buffer_sizes)
```
**EN:** Implements the PUT batch zero copy impl routine for this scope. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的put batch zero copy impl例程。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 1025-1028: _get_batch_zero_copy_impl implementation / _get_batch_zero_copy_impl 实现
```python
    def _get_batch_zero_copy_impl(
        self, key_strs: List[str], buffer_ptrs: List[int], buffer_sizes: List[int]
    ) -> List[int]:
        return self.store.batch_get_into(key_strs, buffer_ptrs, buffer_sizes)
```
**EN:** Implements the get batch zero copy impl routine for this scope. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get batch zero copy impl例程。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 1030-1031: _batch_exist implementation / _batch_exist 实现
```python
    def _batch_exist(self, key_strs: List[str]) -> List[int]:
        return self.store.batch_is_exist(key_strs)
```
**EN:** Implements the batch exist routine for this scope. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch exist例程。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

### Lines 1033-1043: get_stats implementation / get_stats 实现
```python
    def get_stats(self):
        storage_metrics = StorageMetrics()
        storage_metrics.prefetch_pgs.extend(self.prefetch_pgs)
        storage_metrics.backup_pgs.extend(self.backup_pgs)
        storage_metrics.prefetch_bandwidth.extend(self.prefetch_bandwidth)
        storage_metrics.backup_bandwidth.extend(self.backup_bandwidth)
        self.prefetch_pgs.clear()
        self.backup_pgs.clear()
        self.prefetch_bandwidth.clear()
        self.backup_bandwidth.clear()
        return storage_metrics
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `MooncakeStore`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `MooncakeStore`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`MooncakeHostTensorAllocator`**: Defines the `MooncakeHostTensorAllocator` type and its core responsibilities. / 定义 `MooncakeHostTensorAllocator` 类型及其核心职责。
- **`_parse_global_segment_size`**: Provides the `_parse_global_segment_size` entry point for module-level behavior. / 提供模块级行为的 `_parse_global_segment_size` 入口。
- **`MooncakeStoreConfig`**: Defines the `MooncakeStoreConfig` type and its core responsibilities. / 定义 `MooncakeStoreConfig` 类型及其核心职责。
- **`MooncakeBaseStore`**: Defines the `MooncakeBaseStore` type and its core responsibilities. / 定义 `MooncakeBaseStore` 类型及其核心职责。
- **`MooncakeStore`**: Defines the `MooncakeStore` type and its core responsibilities. / 定义 `MooncakeStore` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `ctypes`, `json`, `logging`, `os`, `time`, `uuid`, `dataclasses`, `typing`, `requests`, `torch`, `mooncake.store`
- **Internal / 内部**: `sglang.srt.environ`, `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.memory_pool_host`, `sglang.srt.observability.metrics_collector`, `sglang.srt.distributed.parallel_state`
