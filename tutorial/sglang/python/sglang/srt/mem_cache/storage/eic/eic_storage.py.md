# eic_storage.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/storage/eic/eic_storage.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the EIC storage logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的EIC存储相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and setup / 导入与初始化
```python
import json
import logging
import os
import time
from typing import Any, List, Optional, Tuple

import eic
import torch
import yaml

from sglang.srt.mem_cache.hicache_storage import (
    HiCacheStorage,
    HiCacheStorageConfig,
    HiCacheStorageExtraInfo,
)
from sglang.srt.mem_cache.memory_pool_host import HostKVCache
```
**EN:** Imports `json`, `logging`, `os`, `time`, `typing`, `eic` and other helpers used by the surrounding scope.
**CN:** 导入 `json`, `logging`, `os`, `time`, `typing`, `eic` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 18-56: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)


TensorPoolSize = 2048

REMOTE_EIC_YAML_ENV_VAR = "REMOTE_EIC_YAML"

# gpu direct rdma for kv set
G_EnableKVSetGPUDirect = False

# gpu direct rdma for kv get
G_EnableKVGetGPUDirect = False
# ... omitted for brevity ...
    "cuda:5": "cpu",
    "cuda:6": "cpu",
    "cuda:7": "cpu",
}
```
**EN:** Defines module-level variables such as `logger`, `TensorPoolSize`, `REMOTE_EIC_YAML_ENV_VAR`, `G_EnableKVSetGPUDirect`, `G_EnableKVGetGPUDirect`.
**CN:** 定义模块级变量，例如 `logger`, `TensorPoolSize`, `REMOTE_EIC_YAML_ENV_VAR`, `G_EnableKVSetGPUDirect`, `G_EnableKVGetGPUDirect`。

### Lines 59-66: get_eic_config_file_path implementation / get_eic_config_file_path 实现
```python
def get_eic_config_file_path():
    if os.environ.get(REMOTE_EIC_YAML_ENV_VAR) is not None:
        logger.info(f"eic init with env var {REMOTE_EIC_YAML_ENV_VAR}")
        config_file = os.environ.get(REMOTE_EIC_YAML_ENV_VAR)
    else:
        config_file = "/sgl-workspace/config/remote-eic.yaml"
        logger.info(f"eic init with default config, config_file {config_file}")
    return config_file
```
**EN:** Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

### Lines 69-69: FlexibleKVCacheMemoryPool declaration / FlexibleKVCacheMemoryPool 声明
```python
class FlexibleKVCacheMemoryPool:
```
**EN:** Declares the `FlexibleKVCacheMemoryPool` class.
**CN:** 声明 `FlexibleKVCacheMemoryPool` 类。

### Lines 70-120: __init__ implementation / __init__ 实现
```python
    def __init__(self, conn, kvcache_shape, kvcache_dtype, device):
        self.connection = conn

        if device.startswith("cpu") and G_EnableGPUNicAffinity:
            gpu_id = torch.cuda.current_device()
            self.device = CPUNicAffinity["cuda:" + str(gpu_id)]
            # current memory pool size is 5 times of CPU TensorPoolSize
            mempool_size = TensorPoolSize * 5
        else:
            self.device = device
            mempool_size = TensorPoolSize

# ... omitted for brevity ...
        self.connection.register_memory(vals, meminfo)
        logger.info(
            f"allocate memory pool, size {self.kvcache_mempool.numel() * self.kvcache_mempool.element_size()}, device {self.device}"
        )
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `FlexibleKVCacheMemoryPool`. The implementation iterates over inputs or managed entries.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `FlexibleKVCacheMemoryPool`。实现过程中会遍历输入或受管条目。

### Lines 122-139: try_allocate_kv_cache implementation / try_allocate_kv_cache 实现
```python
    def try_allocate_kv_cache(self, shape, dtype, count=1):
        if len(self.free_data_addr) < count:
            return None

        numel = 1
        for i in shape:
            numel *= i
        if numel != self.kv_cache_numel or dtype != self.kvcache_dtype:
            logger.error(
                f"allocate from mempool failed, self.kvcache_shape {self.kvcache_shape}, dtype {self.kvcache_dtype}, require shape {shape}, dtype {dtype}"
            )
            return None

        ret = []
        for _ in range(count):
            free_index = self.free_data_addr.pop()
            ret.append(self.kvcache_mempool[free_index])
        return ret
```
**EN:** Implements the TRY allocate KV cache routine for this scope. It belongs to `FlexibleKVCacheMemoryPool`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的try allocate KV缓存例程。 该方法属于 `FlexibleKVCacheMemoryPool`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 141-147: free_to_mempool implementation / free_to_mempool 实现
```python
    def free_to_mempool(self, data_ptr):
        if data_ptr not in self.data_ptr_to_index:
            logger.error(
                f"free_to_mempool failed, data_ptr {data_ptr} not in allocated_data_addr"
            )
            return
        self.free_data_addr.add(self.data_ptr_to_index[data_ptr])
```
**EN:** Releases previously allocated resources back to the pool. It belongs to `FlexibleKVCacheMemoryPool`. It returns a computed result to its caller.
**CN:** 将已分配资源释放回资源池。 该方法属于 `FlexibleKVCacheMemoryPool`。它会向调用方返回计算结果。

### Lines 149-150: check_data_ptr_allocated implementation / check_data_ptr_allocated 实现
```python
    def check_data_ptr_allocated(self, data_ptr):
        return data_ptr in self.data_ptr_to_index
```
**EN:** Implements the check data PTR allocated routine for this scope. It belongs to `FlexibleKVCacheMemoryPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的check data ptr allocated例程。 该方法属于 `FlexibleKVCacheMemoryPool`。它会向调用方返回计算结果。

### Lines 152-153: left_count implementation / left_count 实现
```python
    def left_count(self):
        return len(self.free_data_addr)
```
**EN:** Implements the left count routine for this scope. It belongs to `FlexibleKVCacheMemoryPool`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的left count例程。 该方法属于 `FlexibleKVCacheMemoryPool`。它会向调用方返回计算结果。

### Lines 156-156: EICStorage declaration / EICStorage 声明
```python
class EICStorage(HiCacheStorage):
```
**EN:** Declares the `EICStorage` class and connects it to `HiCacheStorage`.
**CN:** 声明 `EICStorage` 类，并将其关联到 `HiCacheStorage`。

### Lines 157-282: __init__ implementation / __init__ 实现
```python
    def __init__(
        self, hicache_config: HiCacheStorageConfig, memory_pool_host: HostKVCache
    ):
        global G_EnableKVSetGPUDirect, G_EnableKVGetGPUDirect
        global GPUNicAffinity, CPUNicAffinity, G_EnableGPUNicAffinity

        config_file = get_eic_config_file_path()
        if os.path.exists(config_file) is False:
            logger.error(f"config file {config_file} not exists")
            raise RuntimeError(f"eic config file {config_file} not exists")

        with open(config_file, "r") as fin:
# ... omitted for brevity ...
                self.kv_cache_get_mem_pool = FlexibleKVCacheMemoryPool(
                    self.connection, self.kv_cache_shape, self.kv_cache_dtype, "cpu"
                )
        self._init_eic_prefix()
```
**EN:** Initializes the instance and stores construction-time state. It belongs to `EICStorage`. It validates error cases explicitly.
**CN:** 初始化实例并保存构造阶段需要的状态。 该方法属于 `EICStorage`。它会显式处理错误场景。

### Lines 284-298: warmup implementation / warmup 实现
```python
    def warmup(self):
        logger.info("begin warm up eic client")
        start_time = time.perf_counter()
        num_warmup = 1024
        preheat_keys = ["warmup_key_" + str(i) for i in range(num_warmup)]
        batch_size = 32
        for i in range(0, num_warmup, batch_size):
            keys_vec = eic.StringVector()
            for key in preheat_keys[i : i + batch_size]:
                keys_vec.append(key)
            exist_option = eic.ExistOption()
            _, _ = self.connection.mexist(keys_vec, exist_option)
        logger.info(
            f"finish eic client warm up, warm up cost {time.perf_counter() - start_time:.2f} seconds"
        )
```
**EN:** Implements the warmup routine for this scope. It belongs to `EICStorage`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的warmup例程。 该方法属于 `EICStorage`。实现过程中会遍历输入或受管条目。

### Lines 300-312: register_mem_pool_host implementation / register_mem_pool_host 实现
```python
    def register_mem_pool_host(self, memory_pool_host: HostKVCache) -> None:
        # no need judge meminfo type, cuda_id, etc.
        meminfo = eic.MemoryInfo()
        meminfo.type = eic.MemoryType.MEMORY_CUDA
        meminfo.cuda_id = 0
        vals = eic.IOBuffers()
        buffer = memory_pool_host.kv_buffer
        vals.append(
            buffer.data_ptr(),
            buffer.numel() * buffer.element_size(),
            True,
        )
        self.connection.register_memory(vals, meminfo)
```
**EN:** Registers metadata so other components can discover this object. It belongs to `EICStorage`.
**CN:** 注册元数据，使其他组件能够发现该对象。 该方法属于 `EICStorage`。

### Lines 314-320: _init_eic_prefix implementation / _init_eic_prefix 实现
```python
    def _init_eic_prefix(self):
        if self.is_mla_model:
            self.eic_prefix = (
                f"{self.model_name}_mla_att_{self.host_kvcache_layout}@sglang"
            )
        else:
            self.eic_prefix = f"{self.model_name}_mha_attn_{self.host_kvcache_layout}_{self.rank}_{self.world_size}_@sglang"
```
**EN:** Implements the init EIC prefix routine for this scope. It belongs to `EICStorage`.
**CN:** 实现当前作用域中的初始化EIC前缀例程。 该方法属于 `EICStorage`。

### Lines 322-323: _get_eic_key implementation / _get_eic_key 实现
```python
    def _get_eic_key(self, keys: List[str]) -> str:
        return [f"{self.eic_prefix}_{key}" for key in keys]
```
**EN:** Implements the get EIC KEY routine for this scope. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get EIC key例程。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

### Lines 325-336: set implementation / set 实现
```python
    def set(
        self,
        key: str,
        value: Optional[Any] = None,
        target_location: Optional[Any] = None,
        target_size: Optional[Any] = None,
    ) -> bool:
        # now is not used
        if self.use_zero_copy:
            return self.zero_copy_batch_set([key], [target_location])
        else:
            return self.generic_batch_set([key], [value])
```
**EN:** Updates internal state with the provided value. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 使用给定值更新内部状态。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

### Lines 337-338: Comment block / 注释块
```python

    # target_locations and target_sizes are not used for now
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 339-351: batch_set implementation / batch_set 实现
```python
    def batch_set(
        self,
        keys: List[str],
        values: Optional[Any] = None,
        target_locations: Optional[Any] = None,
        target_sizes: Optional[Any] = None,
    ) -> bool:
        if len(keys) == 0:
            return True
        if self.use_zero_copy:
            return self.zero_copy_batch_set(keys, values)
        else:
            return self.generic_batch_set(keys, values)
```
**EN:** Implements the batch set routine for this scope. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch set例程。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

### Lines 353-363: get implementation / get 实现
```python
    def get(
        self,
        key,
        target_location: Optional[Any] = None,
        target_size: Optional[Any] = None,
    ) -> torch.Tensor | None:
        # now is not used
        if self.use_zero_copy:
            return self.zero_copy_batch_get([key], [target_location])
        else:
            return self.generic_batch_get([key], [target_location])
```
**EN:** Retrieves the requested data or state from the current object. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

### Lines 364-365: Comment block / 注释块
```python

    # use for v1 interface, and shound not be called directly
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 366-378: batch_get implementation / batch_get 实现
```python
    def batch_get(
        self,
        keys: List[str],
        target_locations: Optional[Any] = None,
        target_sizes: Optional[Any] = None,
    ) -> List[torch.Tensor | None]:
        assert len(keys) == len(target_locations)
        if len(keys) == 0:
            return None
        if self.use_zero_copy:
            return self.zero_copy_batch_get(keys, target_locations)
        else:
            return self.generic_batch_get(keys, target_locations)
```
**EN:** Implements the batch get routine for this scope. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get例程。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

### Lines 380-402: _batch_exists_impl implementation / _batch_exists_impl 实现
```python
    def _batch_exists_impl(self, keys) -> List[bool]:
        if len(keys) == 0:
            return 0
        eic_keys = self._get_eic_key(keys)
        logger.debug(f"eic exists {len(keys)}")
        result = []
        exist_bs = 1024
        for i in range(0, len(eic_keys), exist_bs):
            batch_keys = eic_keys[i : i + exist_bs]
            keys_vec = eic.StringVector()
            for key in batch_keys:
                keys_vec.append(key)
# ... omitted for brevity ...
                result.extend([False] * len(batch_keys))
            for err_code in exist_outcome.status_codes:
                result.append(err_code == eic.StatusCode.SUCCESS)
        return result
```
**EN:** Implements the batch exists impl routine for this scope. It belongs to `EICStorage`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch exists impl例程。 该方法属于 `EICStorage`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 404-406: exists implementation / exists 实现
```python
    def exists(self, key) -> bool:
        exist_num = self.batch_exists([key])
        return exist_num == 1
```
**EN:** Implements the exists routine for this scope. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的exists例程。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

### Lines 408-424: batch_exists implementation / batch_exists 实现
```python
    def batch_exists(
        self, keys, extra_info: Optional[HiCacheStorageExtraInfo] = None
    ) -> int:
        if len(keys) == 0:
            return 0
        if self.use_zero_copy and not self.is_mla_model:
            keys = self._get_mha_zero_copy_keys(keys)
        exist_mask = self._batch_exists_impl(keys)
        prefix_success = 0
        for exist in exist_mask:
            if exist:
                prefix_success += 1
            else:
                break
        if not self.is_mla_model and self.use_zero_copy:
            prefix_success = prefix_success // 2
        return prefix_success
```
**EN:** Implements the batch exists routine for this scope. It belongs to `EICStorage`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch exists例程。 该方法属于 `EICStorage`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 426-432: delete implementation / delete 实现
```python
    def delete(self, key) -> None:
        eic_keys = self._get_eic_key([key])
        keys_vec = eic.StringVector()
        for eic_key in eic_keys:
            keys_vec.append(eic_key)
        del_option = eic.DelOption()
        self.connection.mdel(keys_vec, del_option)
```
**EN:** Implements the delete routine for this scope. It belongs to `EICStorage`. The implementation iterates over inputs or managed entries.
**CN:** 实现当前作用域中的delete例程。 该方法属于 `EICStorage`。实现过程中会遍历输入或受管条目。

### Lines 434-435: clear implementation / clear 实现
```python
    def clear(self) -> None:
        return
```
**EN:** Resets internal state and returns the object to a clean baseline. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 重置内部状态，使对象回到干净的初始基线。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

### Lines 436-437: Comment block / 注释块
```python

    # Not used for now
```
**EN:** Adds comments that clarify configuration or implementation details.
**CN:** 通过注释补充配置或实现细节。

### Lines 438-445: _filter_kv_cache implementation / _filter_kv_cache 实现
```python
    def _filter_kv_cache(self, total_len) -> Tuple[int, int]:
        mean_len = total_len // self.world_size
        remainder = total_len % self.world_size
        tp_keys_len = mean_len + (1 if self.rank < remainder else 0)
        start = self.rank * mean_len + min(self.rank, remainder)
        end = start + tp_keys_len
        logger.debug(f"start: {start}, end: {end}, tp_keys_len: {tp_keys_len}")
        return start, end
```
**EN:** Implements the filter KV cache routine for this scope. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的filter KV缓存例程。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

### Lines 447-473: zero_copy_batch_set implementation / zero_copy_batch_set 实现
```python
    def zero_copy_batch_set(self, keys: List[str], values: List[torch.Tensor]) -> bool:
        logger.debug(f"eic zero copy set {len(keys)} keys")
        if len(keys) == 0:
            return True
        eic_keys = self._get_eic_key(keys)
        keys_vec = eic.StringVector()
        vals_vec = eic.IOBuffers()
        # set data key & value
        for i, key in enumerate(eic_keys):
            # set data key & value
            keys_vec.append(key)
            vals_vec.append(
# ... omitted for brevity ...
            return [False] * len(keys)
        else:
            logger.debug(f"eic zero copy mset {len(keys)} success")
        return [True] * len(keys)
```
**EN:** Implements the zero copy batch set routine for this scope. It belongs to `EICStorage`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的zero copy batch set例程。 该方法属于 `EICStorage`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 475-522: zero_copy_batch_get implementation / zero_copy_batch_get 实现
```python
    def zero_copy_batch_get(
        self, keys: List[str], values: List[torch.Tensor]
    ) -> List[bool]:
        logger.debug(f"eic zero copy get {len(keys)} keys")
        # Get Data: generate data keys and vals
        get_data_start_time = time.perf_counter()
        eic_keys = self._get_eic_key(keys)
        data_keys = eic.StringVector()
        data_vals = eic.IOBuffers()
        success_mask = [True] * len(keys)
        count = len(keys)
        for i, key in enumerate(eic_keys):
# ... omitted for brevity ...
        get_data_end_time = time.perf_counter()
        get_data_execution_time = (get_data_end_time - get_data_start_time) * 1e6
        logger.debug(f"eic get {count} keys data cost %.2f us", get_data_execution_time)
        return success_mask
```
**EN:** Implements the zero copy batch get routine for this scope. It belongs to `EICStorage`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的zero copy batch get例程。 该方法属于 `EICStorage`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 524-601: generic_batch_set implementation / generic_batch_set 实现
```python
    def generic_batch_set(
        self,
        keys: List[str],
        values: List[torch.Tensor],
    ) -> List[bool]:
        assert len(keys) == len(values)
        logger.debug(f"eic generic set {len(keys)} keys")
        if len(keys) == 0:
            return True
        eic_keys = self._get_eic_key(keys)
        keys_vec = eic.StringVector()
        vals_vec = eic.IOBuffers()
# ... omitted for brevity ...
            return [False] * len(keys)

        logger.debug(f"set data key {len(eic_keys)} success")
        return [True] * len(keys)
```
**EN:** Implements the generic batch set routine for this scope. It belongs to `EICStorage`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的generic batch set例程。 该方法属于 `EICStorage`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 603-681: generic_batch_get implementation / generic_batch_get 实现
```python
    def generic_batch_get(
        self, keys: List[str], buffers: List[torch.Tensor]
    ) -> List[bool]:
        # all success or all fail
        logger.debug(f"eic generic get {len(keys)} keys")
        eic_keys = self._get_eic_key(keys)
        get_data_start_time = time.perf_counter()
        data_keys = eic.StringVector()
        data_vals = eic.IOBuffers()
        count = len(eic_keys)
        registered = False
        items = []
# ... omitted for brevity ...
        get_data_end_time = time.perf_counter()
        get_data_execution_time = (get_data_end_time - get_data_start_time) * 1e6
        logger.debug(f"eic get {count} keys data cost %.2f us", get_data_execution_time)
        return success_mask
```
**EN:** Implements the generic batch get routine for this scope. It belongs to `EICStorage`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的generic batch get例程。 该方法属于 `EICStorage`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 683-688: _get_mha_zero_copy_keys implementation / _get_mha_zero_copy_keys 实现
```python
    def _get_mha_zero_copy_keys(self, keys: List[str]) -> List[str]:
        new_keys = []
        for k in keys:
            new_keys.append(f"{k}_k")
            new_keys.append(f"{k}_v")
        return new_keys
```
**EN:** Implements the get MHA zero copy keys routine for this scope. It belongs to `EICStorage`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get mha zero copy keys例程。 该方法属于 `EICStorage`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 690-697: _get_mha_zero_copy_values implementation / _get_mha_zero_copy_values 实现
```python
    def _get_mha_zero_copy_values(
        self, values: List[torch.Tensor]
    ) -> List[torch.Tensor]:
        new_values = []
        for value in values:
            new_values.append(value[0])
            new_values.append(value[1])
        return new_values
```
**EN:** Implements the get MHA zero copy values routine for this scope. It belongs to `EICStorage`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的get mha zero copy values例程。 该方法属于 `EICStorage`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 699-720: _batch_get_preprocess implementation / _batch_get_preprocess 实现
```python
    def _batch_get_preprocess(self, keys, host_indices):
        page_num = len(host_indices) // self.page_size
        # use memory pool directly or dummy page
        values = (
            [
                self.memory_pool_host.get_data_page(
                    host_indices[i * self.page_size], flat=False
                )
                for i in range(page_num)
            ]
            if self.use_zero_copy
            else [
# ... omitted for brevity ...
            keys = self._get_mha_zero_copy_keys(keys)
            values = self._get_mha_zero_copy_values(values)

        return keys, values
```
**EN:** Implements the batch get preprocess routine for this scope. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get preprocess例程。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

### Lines 722-741: _batch_get_postprocess implementation / _batch_get_postprocess 实现
```python
    def _batch_get_postprocess(self, host_indices, values, results):
        page_num = len(host_indices) // self.page_size

        if self.use_zero_copy:
            if not self.is_mla_model:
                results = [
                    (results[2 * i] and results[2 * i + 1]) for i in range(page_num)
                ]
                results = results[:page_num]
            return results

        # dummy page copy to host memory pool
# ... omitted for brevity ...
                host_indices[i * self.memory_pool_host.page_size], values[i]
            )

        return results
```
**EN:** Implements the batch get postprocess routine for this scope. It belongs to `EICStorage`. The implementation iterates over inputs or managed entries. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get postprocess例程。 该方法属于 `EICStorage`。实现过程中会遍历输入或受管条目。它会向调用方返回计算结果。

### Lines 743-751: batch_get_v1 implementation / batch_get_v1 实现
```python
    def batch_get_v1(
        self,
        keys: List[str],
        host_indices: torch.Tensor,
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> List[bool]:
        keys, values = self._batch_get_preprocess(keys, host_indices)
        results = self.batch_get(keys, values)
        return self._batch_get_postprocess(host_indices, values, results)
```
**EN:** Implements the batch get V 1 routine for this scope. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch get v 1例程。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

### Lines 753-767: _batch_set_preprocess implementation / _batch_set_preprocess 实现
```python
    def _batch_set_preprocess(self, keys, host_indices):
        page_num = len(host_indices) // self.page_size
        flat = not self.use_zero_copy
        values = [
            self.memory_pool_host.get_data_page(
                host_indices[i * self.page_size], flat=flat
            )
            for i in range(page_num)
        ]

        if self.use_zero_copy and not self.is_mla_model:
            keys = self._get_mha_zero_copy_keys(keys)
            values = self._get_mha_zero_copy_values(values)

        return keys, values
```
**EN:** Implements the batch set preprocess routine for this scope. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch set preprocess例程。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

### Lines 769-777: batch_set_v1 implementation / batch_set_v1 实现
```python
    def batch_set_v1(
        self,
        keys: List[str],
        host_indices: torch.Tensor,
        extra_info: Optional[HiCacheStorageExtraInfo] = None,
    ) -> List[bool]:
        keys, values = self._batch_set_preprocess(keys, host_indices)
        results = self.batch_set(keys, values)
        return results
```
**EN:** Implements the batch set V 1 routine for this scope. It belongs to `EICStorage`. It returns a computed result to its caller.
**CN:** 实现当前作用域中的batch set v 1例程。 该方法属于 `EICStorage`。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`get_eic_config_file_path`**: Provides the `get_eic_config_file_path` entry point for module-level behavior. / 提供模块级行为的 `get_eic_config_file_path` 入口。
- **`FlexibleKVCacheMemoryPool`**: Defines the `FlexibleKVCacheMemoryPool` type and its core responsibilities. / 定义 `FlexibleKVCacheMemoryPool` 类型及其核心职责。
- **`EICStorage`**: Defines the `EICStorage` type and its core responsibilities. / 定义 `EICStorage` 类型及其核心职责。

## Dependencies / 依赖关系
- **External / 外部**: `json`, `logging`, `os`, `time`, `typing`, `eic`, `torch`, `yaml`
- **Internal / 内部**: `sglang.srt.mem_cache.hicache_storage`, `sglang.srt.mem_cache.memory_pool_host`
