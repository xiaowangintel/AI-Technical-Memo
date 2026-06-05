# hicache_simm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/storage/simm/hicache_simm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the hicache simm logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的hicache SIMM相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Imports and setup / 导入与初始化
```python
import json
import logging
import os
import re
import time
import uuid
from collections import defaultdict
from dataclasses import dataclass
from datetime import datetime
from typing import Any, Dict, List, Optional

import torch
# ... omitted for brevity ...
    HiCacheStorageConfig,
    HiCacheStorageExtraInfo,
)
from sglang.srt.mem_cache.memory_pool_host import HostKVCache
```
**EN:** Imports `json`, `logging`, `os`, `re`, `time`, `uuid` and other helpers used by the surrounding scope.
**CN:** 导入 `json`, `logging`, `os`, `re`, `time`, `uuid` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 20-21: Comment block / 注释块
```python

# Third Party
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 22-28: Control flow block / 控制流代码块
```python
try:
    from simm.kv import BlockView, Store, register_mr, set_flag
except ImportError as e:
    raise ImportError(
        "Please install simm by following the instructions at https://github.com/scitix/SiMM "
        "to run SGLang with SimmConnector."
    ) from e
```
**EN:** Implements top-level control flow needed for setup or execution.
**CN:** 实现顶层初始化或执行所需的控制流。

### Lines 30-32: Shared state definitions / 共享状态定义
```python
SGLANG_HICACHE_SIMM_JSON_ENV_VAR = "SGLANG_HICACHE_SIMM_CONFIG_PATH"

logger = logging.getLogger(__name__)
```
**EN:** Defines module-level variables such as `SGLANG_HICACHE_SIMM_JSON_ENV_VAR`, `logger`.
**CN:** 定义模块级变量，例如 `SGLANG_HICACHE_SIMM_JSON_ENV_VAR`, `logger`。

### Lines 35-36: SiMMConfig declaration / SiMMConfig 声明
```python
@dataclass
class SiMMConfig:
```
**EN:** Declares the `SiMMConfig` class.
**CN:** 声明 `SiMMConfig` 类。

### Lines 37-39: Shared state definitions / 共享状态定义
```python
    manager_address: str
    clnt_threadpool_size: int
    enable_profile: bool
```
**EN:** Defines class-level variables such as `manager_address`, `clnt_threadpool_size`, `enable_profile`.
**CN:** 定义类级变量，例如 `manager_address`, `clnt_threadpool_size`, `enable_profile`。

### Lines 41-62: from_file implementation / from_file 实现
```python
    @staticmethod
    def from_file() -> "SiMMConfig":
        """Load the config from a JSON file."""
        if os.environ.get(SGLANG_HICACHE_SIMM_JSON_ENV_VAR) is None:
            raise RuntimeError(
                f"Config file path not set. Please set {SGLANG_HICACHE_SIMM_JSON_ENV_VAR}"
            )
        file_path = os.environ.get(SGLANG_HICACHE_SIMM_JSON_ENV_VAR)
        try:
            with open(file_path) as fin:
                config = json.load(fin)
        except Exception as e:
# ... omitted for brevity ...
            manager_address=config.get("manager_address"),
            clnt_threadpool_size=config.get("clnt_threadpool_size", 10),
            enable_profile=config.get("enable_profile", False),
        )
```
**EN:** Load the config from a JSON file. Provides an alternative constructor from existing inputs. It belongs to `SiMMConfig`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 提供基于现有输入的备用构造方式。 该方法属于 `SiMMConfig`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 64-74: load_from_extra_config implementation / load_from_extra_config 实现
```python
    @staticmethod
    def load_from_extra_config(extra_config: dict) -> "SiMMConfig":
        """Load config from extra_config dictionary."""
        if "manager_address" not in extra_config:
            raise ValueError("manager_address is required in extra_config")

        return SiMMConfig(
            manager_address=extra_config.get("manager_address"),
            clnt_threadpool_size=extra_config.get("clnt_threadpool_size", 10),
            enable_profile=extra_config.get("enable_profile", False),
        )
```
**EN:** Load config from extra_config dictionary. Loads state from an external or serialized representation. It belongs to `SiMMConfig`. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 从外部或序列化表示中加载状态。 该方法属于 `SiMMConfig`。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 77-101: get_current_process_numa implementation / get_current_process_numa 实现
```python
def get_current_process_numa() -> int:
    """
    Return value: numa_node of current process, failed return -1
    """
    try:
        # get current cpu
        with open("/proc/self/stat", "r") as f:
            stat_data = f.read()

        # the 39th field is processor
        fields = stat_data.split()
        if len(fields) < 39:
# ... omitted for brevity ...

        return -1
    except Exception:
        return -1
```
**EN:** Return value: numa_node of current process, failed return -1 Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

### Lines 104-128: get_numa_nic_mapping implementation / get_numa_nic_mapping 实现
```python
def get_numa_nic_mapping() -> Dict[int, List[str]]:
    """
    Return value: Dict[numa_node, List(rdma_device_name)]
    """
    ib_root = "/sys/class/infiniband"
    device_map = defaultdict(list)

    if not os.path.exists(ib_root):
        logger.error(f"SiMM ERROR: {ib_root} not found. Are RDMA drivers loaded?")
        return []

    for device_name in os.listdir(ib_root):
# ... omitted for brevity ...
            pass
        device_map[numa_node].append(device_name)

    return device_map
```
**EN:** Return value: Dict[numa_node, List(rdma_device_name)] Retrieves the requested data or state from the current object. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 131-132: HiCacheSiMM declaration / HiCacheSiMM 声明
```python
class HiCacheSiMM(HiCacheStorage):

```
**EN:** Declares the `HiCacheSiMM` class and connects it to `HiCacheStorage`.
**CN:** 声明 `HiCacheSiMM` 类，并将其关联到 `HiCacheStorage`。

### Lines 133-218: __init__ implementation / __init__ 实现
```python
    def __init__(
        self, storage_config: HiCacheStorageConfig = None, mem_pool: HostKVCache = None
    ):
        try:
            extra_config = (
                getattr(storage_config, "extra_config", None)
                if storage_config
                else None
            )
            # Load configuration with manager_address prioritized from extra_config if available
            if (
                extra_config is not None
# ... omitted for brevity ...
            raise
        except Exception as exc:
            logger.error("An error occurred while loading the configuration: %s", exc)
            raise
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `HiCacheSiMM`. It validates error cases explicitly.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `HiCacheSiMM`。它会显式处理错误场景。

### Lines 220-243: warmup implementation / warmup 实现
```python
    def warmup(self):
        """Dryrun a key to warmup SiMM client"""
        logger.info("begin warm up SiMM client")
        start_time = time.perf_counter_ns()
        warmup_key = "sglang_simm_warmup_key" + uuid.uuid4().hex
        warmup_tensor = torch.frombuffer(
            bytearray(warmup_key.encode()), dtype=torch.uint8
        )
        warmup_size = 4 * 1024  # 4 KB
        block = self.store.allocate(warmup_size)
        block_ = block.as_ref()
        block_[: len(warmup_key)] = warmup_tensor
# ... omitted for brevity ...
            logger.warning(f"SiMM client warmup key {warmup_key} data wrong")
        logger.info(
            f"finish SiMM client warm up, cost {(time.perf_counter_ns() - start_time)/1000:.2f} us"
        )
```
**EN:** Dryrun a key to warmup SiMM client Implements the warmup routine for this scope. It belongs to `HiCacheSiMM`.
**CN:** 实现当前作用域中的warmup例程。 该方法属于 `HiCacheSiMM`。

### Lines 245-261: register_mem_pool_host implementation / register_mem_pool_host 实现
```python
    def register_mem_pool_host(self, mem_pool_host: HostKVCache):
        super().register_mem_pool_host(mem_pool_host)
        assert self.mem_pool_host.layout in [
            "page_first",
            "page_first_direct",
        ], "simm storage backend only support page first or page first direct layout"
        buffer = self.mem_pool_host.kv_buffer
        try:
            self.mr_ext = register_mr(buffer)
            if self.mr_ext is None:
                logger.error(
                    f"Failed to register buffer, {buffer=}, please check buffer and RDMA network"
                )
                raise RuntimeError(f"Failed to register buffer to SiMM")
        except TypeError as err:
            logger.error("Failed to register buffer to SiMM: %s", err)
            raise TypeError("SiMM Register Buffer Error.") from err
```
**EN:** Registers metadata so other components can discover this object. It belongs to `HiCacheSiMM`. It validates error cases explicitly.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `HiCacheSiMM`。它会显式处理错误场景。

### Lines 263-274: _get_mha_buffer_meta implementation / _get_mha_buffer_meta 实现
```python
    def _get_mha_buffer_meta(self, keys, indices):
        ptr_list, element_size_list = self.mem_pool_host.get_page_buffer_meta(indices)
        key_list = []
        for key_ in keys:
            key_list.append(f"{key_}_{self.mha_suffix}_k")
            key_list.append(f"{key_}_{self.mha_suffix}_v")
        if len(key_list) != len(ptr_list):
            logger.error(
                f"key size {len(key_list)} not equal with incides ptr size {len(ptr_list)}"
            )
        assert len(key_list) == len(ptr_list)
        return key_list, ptr_list, element_size_list
```
**EN:** Implements the get MHA buffer meta routine for this scope. It belongs to `HiCacheSiMM`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get mha buffer meta例程。 该方法属于 `HiCacheSiMM`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 276-286: _get_mla_buffer_meta implementation / _get_mla_buffer_meta 实现
```python
    def _get_mla_buffer_meta(self, keys, indices):
        ptr_list, element_size_list = self.mem_pool_host.get_page_buffer_meta(indices)
        key_list = []
        for key_ in keys:
            key_list.append(f"{key_}_{self.mla_suffix}_k")
        if len(key_list) != len(ptr_list):
            logger.error(
                f"key size {len(key_list)} not equal with incides ptr size {len(ptr_list)}"
            )
        assert len(key_list) == len(ptr_list)
        return key_list, ptr_list, element_size_list
```
**EN:** Implements the get MLA buffer meta routine for this scope. It belongs to `HiCacheSiMM`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get mla buffer meta例程。 该方法属于 `HiCacheSiMM`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 288-294: _batch_preprocess implementation / _batch_preprocess 实现
```python
    def _batch_preprocess(self, keys, host_indices):
        assert len(keys) > 0
        assert len(keys) == len(host_indices) // self.mem_pool_host.page_size
        if self.is_mla_backend:
            return self._get_mla_buffer_meta(keys, host_indices)
        else:
            return self._get_mha_buffer_meta(keys, host_indices)
```
**EN:** Implements the batch preprocess routine for this scope. It belongs to `HiCacheSiMM`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch preprocess例程。 该方法属于 `HiCacheSiMM`。它会向调用方返回计算结果。

### Lines 296-314: _batch_postprocess implementation / _batch_postprocess 实现
```python
    def _batch_postprocess(self, results: List[int], is_set_operate=False):
        """
        for batch_get_into, results is Vector of integers,
            where each element is the number of bytes read on success, or a negative value on error
        for batch_put_from, results is Vector of integers,
            where each element is 0 on success, or a negative value on error
        """
        if self.is_mla_backend:
            return [k_res == 0 if is_set_operate else k_res > 0 for k_res in results]
        else:
            kv_pairs = zip(results[::2], results[1::2])
            return [
# ... omitted for brevity ...
                    else (k_res > 0 and v_res > 0)
                )
                for k_res, v_res in kv_pairs
            ]
```
**EN:** for batch_get_into, results is Vector of integers, where each element is the number of bytes read on success, or a negative value on error for batch_put_from, results is Vector... Implements the batch postprocess routine for this scope. It belongs to `HiCacheSiMM`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch postprocess例程。 该方法属于 `HiCacheSiMM`。它会向调用方返回计算结果。

### Lines 316-339: batch_get_v1 implementation / batch_get_v1 实现
```python
    def batch_get_v1(
        self,
        keys: List[str],
        host_indices: torch.Tensor,
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> List[bool]:
        # Apply extra_backend_tag prefix if available
        if self.extra_backend_tag is not None:
            prefix = self.extra_backend_tag
            keys = [f"{prefix}_{key}" for key in keys]

        t1 = time.perf_counter_ns()
# ... omitted for brevity ...
                f"SiMM batch_get_v1 {len(keys)} keys, total size: {total_size / 1024**2} MiB, \
                    using {(t2 - t1)/1000} us, Throughput: {total_size / 1024**3 / ((t2 - t1) / 1000**3):.2f} GiB/s"
            )
        return self._batch_postprocess(get_results, is_set_operate=False)
```
**EN:** Implements the batch get V 1 routine for this scope. It belongs to `HiCacheSiMM`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get v 1例程。 该方法属于 `HiCacheSiMM`。它会向调用方返回计算结果。

### Lines 341-391: batch_set_v1 implementation / batch_set_v1 实现
```python
    def batch_set_v1(
        self,
        keys: List[str],
        host_indices: torch.Tensor,
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> List[bool]:
        # Apply extra_backend_tag prefix if available
        if self.extra_backend_tag is not None:
            prefix = self.extra_backend_tag
            keys = [f"{prefix}_{key}" for key in keys]

        t1 = time.perf_counter_ns()
# ... omitted for brevity ...
                    using {(t3 - t2)/1000} us, Throughput: {total_size / 1024**3 / ((t3 - t2) / 1000**3):.2f} GiB/s"
            )

        return self._batch_postprocess(set_results, is_set_operate=True)
```
**EN:** Implements the batch set V 1 routine for this scope. It belongs to `HiCacheSiMM`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch set v 1例程。 该方法属于 `HiCacheSiMM`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 393-408: set implementation / set 实现
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
        exist_result = self._batch_exist_impl([key])
        if exist_result[0]:
            return True
        put_result = self._put_batch_zero_copy_impl(
            [key], [target_location], [target_sizes]
        )
        return put_result[0] == 0
```
**EN:** Updates internal state with the provided value. It belongs to `HiCacheSiMM`. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。 该方法属于 `HiCacheSiMM`。它会向调用方返回计算结果。

### Lines 410-457: batch_set implementation / batch_set 实现
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
            if exist_result[i] == 0:
                break
            success_count += 1
        return success_count == len(keys)
```
**EN:** Implements the batch set routine for this scope. It belongs to `HiCacheSiMM`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch set例程。 该方法属于 `HiCacheSiMM`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 459-469: get implementation / get 实现
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
**EN:** Retrieves the requested data or state from the current object. It belongs to `HiCacheSiMM`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `HiCacheSiMM`。它会向调用方返回计算结果。

### Lines 471-490: batch_get implementation / batch_get 实现
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
        get_result = self._get_batch_zero_copy_impl(
            keys, target_locations, target_sizes
        )
# ... omitted for brevity ...
        for i in range(len(keys)):
            if get_result[i] < 0:
                return i // key_multiplier
        return len(keys) // key_multiplier
```
**EN:** Implements the batch get routine for this scope. It belongs to `HiCacheSiMM`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get例程。 该方法属于 `HiCacheSiMM`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 492-494: exists implementation / exists 实现
```python
    def exists(self, key) -> bool:
        exist_result = self._batch_exist_impl([key])
        return exist_result[0]
```
**EN:** Implements the exists routine for this scope. It belongs to `HiCacheSiMM`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的exists例程。 该方法属于 `HiCacheSiMM`。它会向调用方返回计算结果。

### Lines 496-519: batch_exists implementation / batch_exists 实现
```python
    def batch_exists(
        self, keys, extra_info: Optional[HiCacheStorageExtraInfo] = None
    ) -> int:
        if self.is_mla_backend:
            query_keys = [f"{key}_{self.mla_suffix}_k" for key in keys]
            key_multiplier = 1
        else:
            query_keys = []
            for key in keys:
                query_keys.append(f"{key}_{self.mha_suffix}_k")
                query_keys.append(f"{key}_{self.mha_suffix}_v")
            key_multiplier = 2
# ... omitted for brevity ...
        for i in range(len(query_keys)):
            if not exist_result[i]:
                return i // key_multiplier
        return len(query_keys) // key_multiplier
```
**EN:** Implements the batch exists routine for this scope. It belongs to `HiCacheSiMM`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch exists例程。 该方法属于 `HiCacheSiMM`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 521-530: _put_batch_zero_copy_impl implementation / _put_batch_zero_copy_impl 实现
```python
    def _put_batch_zero_copy_impl(
        self, key_strs: List[str], buffer_ptrs: List[int], buffer_sizes: List[int]
    ) -> List[int]:
        block_views = []
        for i in range(len(buffer_ptrs)):
            block_view = BlockView.from_buffer(
                buffer_ptrs[i], buffer_sizes[i], self.mr_ext
            )
            block_views.append(block_view)
        return self.store.mput(key_strs, block_views)
```
**EN:** Implements the PUT batch zero copy impl routine for this scope. It belongs to `HiCacheSiMM`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的put batch zero copy impl例程。 该方法属于 `HiCacheSiMM`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 532-541: _get_batch_zero_copy_impl implementation / _get_batch_zero_copy_impl 实现
```python
    def _get_batch_zero_copy_impl(
        self, key_strs: List[str], buffer_ptrs: List[int], buffer_sizes: List[int]
    ) -> List[int]:
        block_views = []
        for i in range(len(buffer_ptrs)):
            block_view = BlockView.from_buffer(
                buffer_ptrs[i], buffer_sizes[i], self.mr_ext
            )
            block_views.append(block_view)
        return self.store.mget(key_strs, block_views)
```
**EN:** Implements the get batch zero copy impl routine for this scope. It belongs to `HiCacheSiMM`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get batch zero copy impl例程。 该方法属于 `HiCacheSiMM`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 543-544: _batch_exist_impl implementation / _batch_exist_impl 实现
```python
    def _batch_exist_impl(self, key_strs: List[str]) -> List[bool]:
        return self.store.mexists(key_strs)
```
**EN:** Implements the batch exist impl routine for this scope. It belongs to `HiCacheSiMM`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch exist impl例程。 该方法属于 `HiCacheSiMM`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`SiMMConfig`**: Defines the `SiMMConfig` type and its core responsibilities. / 定义 `SiMMConfig` 类型及其核心职责。
- **`get_current_process_numa`**: Provides the `get_current_process_numa` entry point for module-level behavior. / 提供模块级行为的 `get_current_process_numa` 入口。
- **`get_numa_nic_mapping`**: Provides the `get_numa_nic_mapping` entry point for module-level behavior. / 提供模块级行为的 `get_numa_nic_mapping` 入口。
- **`HiCacheSiMM`**: Defines the `HiCacheSiMM` type and its core responsibilities. / 定义 `HiCacheSiMM` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `json`, `logging`, `os`, `re`, `time`, `uuid`, `collections`, `dataclasses`, `datetime`, `typing`, `torch`, `simm.kv`
- **Internal / 内部**: `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.memory_pool_host`
