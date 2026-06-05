# memory_manager.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/memory_manager.py`
- **EN:** Defines classes `PoolMemoryManager`, `DevicePtrWrapper`, `SYCLPtrWrapper` and functions `_todevice`, `todevice`, `device_mem_alloc`, `align_size`, `create_memory_pool` for the cutlass_cppgen package.
- **CN:** 为 cutlass_cppgen 包定义类 `PoolMemoryManager`, `DevicePtrWrapper`, `SYCLPtrWrapper`和函数 `_todevice`, `todevice`, `device_mem_alloc`, `align_size`, `create_memory_pool`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32 — File header

```python
#################################################################################################
#
# Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
# SPDX-License-Identifier: BSD-3-Clause
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# 1. Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.
#
# 2. Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following disclaimer in the documentation
# and/or other materials provided with the distribution.
#
# 3. Neither the name of the copyright holder nor the names of its
# contributors may be used to endorse or promote products derived from
# this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
# CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
#################################################################################################
```
**EN:** Contains the file header, license notice, and opening comments for the module.
**CN:** 包含文件头、许可证声明以及模块开头的注释。

### Line 33 — Import `numpy as np`

```python
import numpy as np
```
**EN:** Imports `numpy as np` so later code can use these APIs at module scope.
**CN:** 导入 `numpy as np`，供后续代码在模块级使用这些 API。

### Line 35 — Import `cutlass_cppgen`

```python
import cutlass_cppgen
```
**EN:** Imports `cutlass_cppgen` so later code can use these APIs at module scope.
**CN:** 导入 `cutlass_cppgen`，供后续代码在模块级使用这些 API。

### Line 36 — From `cutlass_cppgen.utils.datatypes` import

```python
from cutlass_cppgen.utils.datatypes import is_numpy_tensor
```
**EN:** Imports `is_numpy_tensor` from `cutlass_cppgen.utils.datatypes` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.utils.datatypes` 导入 `is_numpy_tensor`，以便后续代码在模块级复用共享定义。

### Line 37 — From `cutlass_cppgen.utils.lazy_import` import

```python
from cutlass_cppgen.utils.lazy_import import lazy_import
```
**EN:** Imports `lazy_import` from `cutlass_cppgen.utils.lazy_import` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.utils.lazy_import` 导入 `lazy_import`，以便后续代码在模块级复用共享定义。

### Lines 39-42 — Conditional `cutlass_cppgen.use_rmm`

```python
if cutlass_cppgen.use_rmm:
    import rmm
else:
    cudart = lazy_import("cuda.cudart")
```
**EN:** Checks `cutlass_cppgen.use_rmm` and selects the matching branch at module scope.
**CN:** 检查 `cutlass_cppgen.use_rmm`，并在模块级选择匹配的分支。

### Line 44 — Assign `dpctl`

```python
dpctl = lazy_import("dpctl")
```
**EN:** Assigns `dpctl` from `lazy_import('dpctl')`, establishing state at module scope.
**CN:** 将 `dpctl` 赋值为 `lazy_import('dpctl')`，用于在模块级建立状态。

### Line 47 — Class `PoolMemoryManager`

```python
class PoolMemoryManager:
```
**EN:** Declares class `PoolMemoryManager` deriving from `object`.
**CN:** 声明类 `PoolMemoryManager`，其基类为 `object`。

#### Line 48 — Function `__init__`

```python
    def __init__(self, init_pool_size: int, max_pool_size: int) -> None:
```
**EN:** Defines function `__init__` with parameters `self, init_pool_size, max_pool_size`.
**CN:** 定义函数 `__init__`，参数为 `self, init_pool_size, max_pool_size`。

##### Lines 49-53 — Assign `self.pool`

```python
        self.pool = rmm.mr.PoolMemoryResource(
            rmm.mr.CudaMemoryResource(),
            initial_pool_size=init_pool_size,
            maximum_pool_size=max_pool_size
        )
```
**EN:** Assigns `self.pool` from `rmm.mr.PoolMemoryResource(rmm.mr.CudaMemoryResource(), initial_pool_size=init_pool_size, maximum_pool_size=max_pool_size)`, establishing state in function `__init__`.
**CN:** 将 `self.pool` 赋值为 `rmm.mr.PoolMemoryResource(rmm.mr.CudaMemoryResource(), initial_pool_size=init_pool_size, maximum_pool_size=max_pool_size)`，用于在函数 `__init__` 中建立状态。

##### Line 54 — Assign `self.mr`

```python
        self.mr = rmm.mr.TrackingResourceAdaptor(self.pool)
```
**EN:** Assigns `self.mr` from `rmm.mr.TrackingResourceAdaptor(self.pool)`, establishing state in function `__init__`.
**CN:** 将 `self.mr` 赋值为 `rmm.mr.TrackingResourceAdaptor(self.pool)`，用于在函数 `__init__` 中建立状态。

##### Line 55 — Call `rmm.mr.set_current_device_resource`

```python
        rmm.mr.set_current_device_resource(self.mr)
```
**EN:** Calls `rmm.mr.set_current_device_resource` for side effects or initialization work in function `__init__`.
**CN:** 调用 `rmm.mr.set_current_device_resource` 执行副作用或初始化逻辑；该语句位于在函数 `__init__` 中。

#### Line 57 — Function `pool_size`

```python
    def pool_size(self):
```
**EN:** Defines function `pool_size` with parameters `self`.
**CN:** 定义函数 `pool_size`，参数为 `self`。

##### Line 58 — Return

```python
        return self.pool.pool_size()
```
**EN:** Returns `self.pool.pool_size()` to the caller.
**CN:** 向调用方返回 `self.pool.pool_size()`。

### Lines 61-65 — Class `DevicePtrWrapper`

```python
class DevicePtrWrapper:
    """
    Wrapper around a pointer to device memory to provide a uniform interface with the RMM DeviceBuffer
    (at least in terms of the interface used by the CUTLASS Python interface)
    """
```
**EN:** Declares class `DevicePtrWrapper` deriving from `object`. Purpose: Wrapper around a pointer to device memory to provide a uniform interface with the RMM DeviceBuffer
**CN:** 声明类 `DevicePtrWrapper`，其基类为 `object`。 其用途：Wrapper around a pointer to device memory to provide a uniform interface with the RMM DeviceBuffer

#### Line 66 — Function `__init__`

```python
    def __init__(self, dev_ptr):
```
**EN:** Defines function `__init__` with parameters `self, dev_ptr`.
**CN:** 定义函数 `__init__`，参数为 `self, dev_ptr`。

##### Line 67 — Assign `self.dev_ptr`

```python
        self.dev_ptr = dev_ptr
```
**EN:** Assigns `self.dev_ptr` from `dev_ptr`, establishing state in function `__init__`.
**CN:** 将 `self.dev_ptr` 赋值为 `dev_ptr`，用于在函数 `__init__` 中建立状态。

#### Lines 69-70 — Function `ptr`

```python
    @property
    def ptr(self):
```
**EN:** Defines function `ptr` with parameters `self`.
**CN:** 定义函数 `ptr`，参数为 `self`。

##### Line 71 — Return

```python
        return self.dev_ptr
```
**EN:** Returns `self.dev_ptr` to the caller.
**CN:** 向调用方返回 `self.dev_ptr`。

### Lines 73-76 — Class `SYCLPtrWrapper`

```python
class SYCLPtrWrapper:
    """
    Wrapper around a pointer to USM device memory to provide a uniform interface.
    """
```
**EN:** Declares class `SYCLPtrWrapper` deriving from `object`. Purpose: Wrapper around a pointer to USM device memory to provide a uniform interface.
**CN:** 声明类 `SYCLPtrWrapper`，其基类为 `object`。 其用途：Wrapper around a pointer to USM device memory to provide a uniform interface.

#### Line 77 — Function `__init__`

```python
    def __init__(self, usm):
```
**EN:** Defines function `__init__` with parameters `self, usm`.
**CN:** 定义函数 `__init__`，参数为 `self, usm`。

##### Line 78 — Assign `self.usm`

```python
        self.usm = usm
```
**EN:** Assigns `self.usm` from `usm`, establishing state in function `__init__`.
**CN:** 将 `self.usm` 赋值为 `usm`，用于在函数 `__init__` 中建立状态。

#### Lines 80-81 — Function `ptr`

```python
    @property
    def ptr(self):
```
**EN:** Defines function `ptr` with parameters `self`.
**CN:** 定义函数 `ptr`，参数为 `self`。

##### Line 82 — Return

```python
        return self.usm.__sycl_usm_array_interface__["data"][0]
```
**EN:** Returns `self.usm.__sycl_usm_array_interface__['data'][0]` to the caller.
**CN:** 向调用方返回 `self.usm.__sycl_usm_array_interface__['data'][0]`。

#### Lines 84-85 — Function `usm_mem`

```python
    @property
    def usm_mem(self):
```
**EN:** Defines function `usm_mem` with parameters `self`.
**CN:** 定义函数 `usm_mem`，参数为 `self`。

##### Line 86 — Return

```python
        return self.usm
```
**EN:** Returns `self.usm` to the caller.
**CN:** 向调用方返回 `self.usm`。

### Lines 89-92 — Function `_todevice`

```python
def _todevice(host_data, stream):
    """
    Helper for transferring host data to device memory
    """
```
**EN:** Defines function `_todevice` with parameters `host_data, stream`. Purpose: Helper for transferring host data to device memory
**CN:** 定义函数 `_todevice`，参数为 `host_data, stream`。 其用途：Helper for transferring host data to device memory

#### Lines 93-94 — Conditional `cutlass_cppgen.use_rmm`

```python
    if cutlass_cppgen.use_rmm:
        return rmm.DeviceBuffer.to_device(host_data.tobytes())
```
**EN:** Checks `cutlass_cppgen.use_rmm` and selects the matching branch in function `_todevice`.
**CN:** 检查 `cutlass_cppgen.use_rmm`，并在函数 `_todevice` 中选择匹配的分支。

#### Lines 95-111 — Conditional `cutlass_cppgen._use_sycl`

```python
    if cutlass_cppgen._use_sycl:
        nbytes = len(host_data.tobytes())
        usm_device_ptr = device_mem_alloc(nbytes, stream)
        stream.memcpy(usm_device_ptr.usm_mem, host_data.tobytes(), nbytes)
        return usm_device_ptr
    else:
        nbytes = len(host_data.tobytes())
        dev_ptr_wrapper = device_mem_alloc(nbytes)
        err, = cudart.cudaMemcpy(
            dev_ptr_wrapper.ptr,
            host_data.__array_interface__['data'][0],
            nbytes,
            cudart.cudaMemcpyKind.cudaMemcpyHostToDevice
        )
        if err != cudart.cudaError_t.cudaSuccess:
            raise Exception(f"cudaMemcpy failed with error {err}")
        return dev_ptr_wrapper
```
**EN:** Checks `cutlass_cppgen._use_sycl` and selects the matching branch in function `_todevice`.
**CN:** 检查 `cutlass_cppgen._use_sycl`，并在函数 `_todevice` 中选择匹配的分支。

### Lines 114-117 — Function `todevice`

```python
def todevice(host_data, dtype=np.float32, stream = None):
    """
    Pass the host_data to device memory
    """
```
**EN:** Defines function `todevice` with parameters `host_data, dtype, stream`. Purpose: Pass the host_data to device memory
**CN:** 定义函数 `todevice`，参数为 `host_data, dtype, stream`。 其用途：Pass the host_data to device memory

#### Lines 118-121 — Conditional `isinstance(host_data, list)`

```python
    if isinstance(host_data, list):
        return _todevice(np.array(host_data, dtype=dtype), stream)
    elif is_numpy_tensor(host_data):
        return _todevice(host_data, stream)
```
**EN:** Checks `isinstance(host_data, list)` and selects the matching branch in function `todevice`.
**CN:** 检查 `isinstance(host_data, list)`，并在函数 `todevice` 中选择匹配的分支。

### Line 124 — Function `device_mem_alloc`

```python
def device_mem_alloc(size, stream = None):
```
**EN:** Defines function `device_mem_alloc` with parameters `size, stream`.
**CN:** 定义函数 `device_mem_alloc`，参数为 `size, stream`。

#### Lines 125-134 — Conditional `cutlass_cppgen.use_rmm`

```python
    if cutlass_cppgen.use_rmm:
        return rmm.DeviceBuffer(size=size)
    elif cutlass_cppgen._use_sycl:
        device_usm = dpctl.memory.MemoryUSMDevice(size, queue=stream)
        return SYCLPtrWrapper(device_usm)
    else:
        err, ptr = cudart.cudaMalloc(size)
        if err != cudart.cudaError_t.cudaSuccess:
            raise Exception(f"cudaMalloc failed with error {err}")
        return DevicePtrWrapper(ptr)
```
**EN:** Checks `cutlass_cppgen.use_rmm` and selects the matching branch in function `device_mem_alloc`.
**CN:** 检查 `cutlass_cppgen.use_rmm`，并在函数 `device_mem_alloc` 中选择匹配的分支。

### Line 137 — Function `align_size`

```python
def align_size(size, alignment=256):
```
**EN:** Defines function `align_size` with parameters `size, alignment`.
**CN:** 定义函数 `align_size`，参数为 `size, alignment`。

#### Line 138 — Return

```python
    return ((size + alignment - 1) // alignment) * alignment
```
**EN:** Returns `(size + alignment - 1) // alignment * alignment` to the caller.
**CN:** 向调用方返回 `(size + alignment - 1) // alignment * alignment`。

### Line 141 — Function `create_memory_pool`

```python
def create_memory_pool(init_pool_size=0, max_pool_size=2 ** 34):
```
**EN:** Defines function `create_memory_pool` with parameters `init_pool_size, max_pool_size`.
**CN:** 定义函数 `create_memory_pool`，参数为 `init_pool_size, max_pool_size`。

#### Lines 142-146 — Conditional `cutlass_cppgen.use_rmm`

```python
    if cutlass_cppgen.use_rmm:
        memory_pool = PoolMemoryManager(init_pool_size=init_pool_size, max_pool_size=max_pool_size)
        return memory_pool
    else:
        return None
```
**EN:** Checks `cutlass_cppgen.use_rmm` and selects the matching branch in function `create_memory_pool`.
**CN:** 检查 `cutlass_cppgen.use_rmm`，并在函数 `create_memory_pool` 中选择匹配的分支。

## Key Concepts / 关键概念

- **EN:** Top-level classes: `PoolMemoryManager`, `DevicePtrWrapper`, `SYCLPtrWrapper`.
- **CN:** 顶层类：`PoolMemoryManager`, `DevicePtrWrapper`, `SYCLPtrWrapper`。
- **EN:** Top-level functions: `_todevice`, `todevice`, `device_mem_alloc`, `align_size`, `create_memory_pool`.
- **CN:** 顶层函数：`_todevice`, `todevice`, `device_mem_alloc`, `align_size`, `create_memory_pool`。
- **EN:** This module organizes backend utilities, data structures, or runtime helpers used by cutlass_cppgen.
- **CN:** 该模块组织 cutlass_cppgen 所需的后端工具、数据结构或运行时辅助逻辑。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen`, `cutlass_cppgen.utils.datatypes`, `cutlass_cppgen.utils.lazy_import`
- **Standard & third-party / 标准库与第三方:** `numpy`, `rmm`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
