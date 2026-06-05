# host_shared_memory.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/host_shared_memory.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `host_shared_memory` and the surrounding SGLang serving stack. / 提供围绕 `host_shared_memory` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Module setup and shared state / 模块设置与共享状态
```python
import logging
from dataclasses import dataclass
from multiprocessing import shared_memory
from pathlib import Path
from typing import List, Optional

import numpy as np
import torch

from sglang.srt.distributed.naive_distributed import get_naive_distributed
from sglang.srt.utils import check_cuda_result

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `dataclasses`, `multiprocessing`, `pathlib`, `typing`, `numpy`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `dataclasses`, `multiprocessing`, `pathlib`, `typing`, `numpy`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 16-16: Class `HostSharedMemoryManager` declaration / 类 `HostSharedMemoryManager` 声明
```python
class HostSharedMemoryManager:
```
**EN:** This class establishes `HostSharedMemoryManager` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `malloc`, `_malloc_raw`.
**CN:** 该类将 `HostSharedMemoryManager` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `malloc`, `_malloc_raw` 等方法。

### Lines 17-20: Method `HostSharedMemoryManager.__init__` / 方法 `HostSharedMemoryManager.__init__`
```python
    def __init__(self, base_name: str):
        self._base_name = Path(base_name)
        self._operation_index = 0
        self._records: List[_Record] = []
```
**EN:** This method implements `__init__` on `HostSharedMemoryManager`. It primarily calls `Path` to complete its work. State updates are written into `self._base_name`, `self._operation_index`, `self._records`.
**CN:** 该方法（属于 `HostSharedMemoryManager`）实现了 `__init__`。 它主要通过调用 `Path` 来完成任务。 状态更新主要写入 `self._base_name`, `self._operation_index`, `self._records`。

### Lines 22-25: Method `HostSharedMemoryManager.malloc` / 方法 `HostSharedMemoryManager.malloc`
```python
    def malloc(self, *, shape, dtype):
        meta_tensor = torch.empty(size=shape, dtype=dtype, device="meta")
        raw = self._malloc_raw(num_bytes=meta_tensor.nbytes)
        return raw.view(dtype).view(*shape)
```
**EN:** This method implements `malloc` on `HostSharedMemoryManager`. It primarily calls `torch.empty`, `self._malloc_raw`, `raw.view.view`, `raw.view` to complete its work. State updates are written into `meta_tensor`, `raw`.
**CN:** 该方法（属于 `HostSharedMemoryManager`）实现了 `malloc`。 它主要通过调用 `torch.empty`, `self._malloc_raw`, `raw.view.view`, `raw.view` 来完成任务。 状态更新主要写入 `meta_tensor`, `raw`。

### Lines 27-60: Method `HostSharedMemoryManager._malloc_raw` / 方法 `HostSharedMemoryManager._malloc_raw`
```python
    def _malloc_raw(self, *, num_bytes: int) -> torch.Tensor:
        import cuda.bindings.runtime as cuda_rt

        self._operation_index += 1
        shm_name = f"{self._base_name}_op{self._operation_index}"

        # TODO handle dispose
        if get_naive_distributed().get_rank() == 0:
            shm = shared_memory.SharedMemory(name=shm_name, create=True, size=num_bytes)

        get_naive_distributed().barrier()

        if get_naive_distributed().get_rank() != 0:
            shm = shared_memory.SharedMemory(name=shm_name)

        np_array = np.ndarray((num_bytes,), dtype=np.uint8, buffer=shm.buf)
        tensor = torch.from_numpy(np_array)

        check_cuda_result(
            cuda_rt.cudaHostRegister(
                tensor.data_ptr(), num_bytes, cuda_rt.cudaHostRegisterPortable
            )
        )

        get_naive_distributed().barrier()

        self._records.append(
            _Record(
                shm=shm,
                np_array=np_array,
                tensor=tensor,
            )
        )
        return tensor
```
**EN:** This method implements `_malloc_raw` on `HostSharedMemoryManager`. It primarily calls `get_naive_distributed.barrier`, `np.ndarray`, `torch.from_numpy`, `check_cuda_result`, `self._records.append`, `get_naive_distributed.get_rank` to complete its work. State updates are written into `self._operation_index`, `shm_name`, `np_array`, `tensor`, `shm`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `HostSharedMemoryManager`）实现了 `_malloc_raw`。 它主要通过调用 `get_naive_distributed.barrier`, `np.ndarray`, `torch.from_numpy`, `check_cuda_result`, `self._records.append`, `get_naive_distributed.get_rank` 来完成任务。 状态更新主要写入 `self._operation_index`, `shm_name`, `np_array`, `tensor`, `shm`。 实现中使用了条件分支。

### Lines 63-67: Class `_Record` declaration / 类 `_Record` 声明
```python
@dataclass
class _Record:
    shm: shared_memory.SharedMemory
    np_array: np.ndarray
    tensor: torch.Tensor
```
**EN:** This class establishes `_Record` as a compact data container for the surrounding logic. The main stored fields include `shm`, `np_array`, `tensor`.
**CN:** 该类将 `_Record` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `shm`, `np_array`, `tensor`。

### Lines 71-71: Constants and shared state / 常量与共享状态
```python
_instance: Optional[HostSharedMemoryManager] = None
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_instance`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_instance`。

### Lines 74-76: Function `get_host_shared_memory_manager` / 函数 `get_host_shared_memory_manager`
```python
def get_host_shared_memory_manager():
    assert _instance is not None
    return _instance
```
**EN:** This function implements `get_host_shared_memory_manager`.
**CN:** 该函数实现了 `get_host_shared_memory_manager`。

### Lines 79-82: Function `set_host_shared_memory_manager` / 函数 `set_host_shared_memory_manager`
```python
def set_host_shared_memory_manager(instance: HostSharedMemoryManager):
    global _instance
    assert _instance is None
    _instance = instance
```
**EN:** This function implements `set_host_shared_memory_manager`. State updates are written into `_instance`.
**CN:** 该函数实现了 `set_host_shared_memory_manager`。 状态更新主要写入 `_instance`。

## Key Concepts / 关键概念
- **Classes / 类**: `HostSharedMemoryManager`, `_Record`
- **Functions / 函数**: `get_host_shared_memory_manager`, `set_host_shared_memory_manager`, `__init__`, `malloc`, `_malloc_raw`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.distributed.naive_distributed`, `sglang.srt.utils`
- **External / 外部依赖**: `numpy`, `torch`, `cuda.bindings.runtime`
- **Standard library / 标准库**: `logging`, `dataclasses`, `multiprocessing`, `pathlib`, `typing`
