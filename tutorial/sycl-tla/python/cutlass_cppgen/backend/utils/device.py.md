# device.py — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `python/cutlass_cppgen/backend/utils/device.py`
- **EN:** Utility functions for interacting with the device
- **CN:** 模块文档说明：Utility functions for interacting with the device

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

### Lines 33-35 — Docstring

```python
"""
Utility functions for interacting with the device
"""
```
**EN:** Docstring explains this scope: Utility functions for interacting with the device
**CN:** 文档字符串说明了该作用域的用途：Utility functions for interacting with the device

### Line 36 — From `__future__` import

```python
from __future__ import annotations
```
**EN:** Imports `annotations` from `__future__` to reuse shared definitions at module scope.
**CN:** 从 `__future__` 导入 `annotations`，以便后续代码在模块级复用共享定义。

### Line 38 — From `cutlass_cppgen.utils.lazy_import` import

```python
from cutlass_cppgen.utils.lazy_import import lazy_import
```
**EN:** Imports `lazy_import` from `cutlass_cppgen.utils.lazy_import` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.utils.lazy_import` 导入 `lazy_import`，以便后续代码在模块级复用共享定义。

### Line 39 — Assign `cuda`

```python
cuda = lazy_import("cuda.cuda")
```
**EN:** Assigns `cuda` from `lazy_import('cuda.cuda')`, establishing state at module scope.
**CN:** 将 `cuda` 赋值为 `lazy_import('cuda.cuda')`，用于在模块级建立状态。

### Line 40 — Assign `cudart`

```python
cudart =  lazy_import("cuda.cudart")
```
**EN:** Assigns `cudart` from `lazy_import('cuda.cudart')`, establishing state at module scope.
**CN:** 将 `cudart` 赋值为 `lazy_import('cuda.cudart')`，用于在模块级建立状态。

### Line 41 — Assign `dpctl`

```python
dpctl = lazy_import("dpctl")
```
**EN:** Assigns `dpctl` from `lazy_import('dpctl')`, establishing state at module scope.
**CN:** 将 `dpctl` 赋值为 `lazy_import('dpctl')`，用于在模块级建立状态。

### Line 43 — Comment or spacing block

```python
# These are checked against the dpctl API equivalent of sycl-ls | grep level_zero:gpu
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

### Line 44 — Assign `bmg_indicators`

```python
bmg_indicators = ["bmg", "battlemage", "xe20", "g21", "b580", "b60", "b70", "g31","20.2.0"]
```
**EN:** Assigns `bmg_indicators` from `['bmg', 'battlemage', 'xe20', 'g21', 'b580', 'b60', 'b70', 'g31', '20.2.0']`, establishing state at module scope.
**CN:** 将 `bmg_indicators` 赋值为 `['bmg', 'battlemage', 'xe20', 'g21', 'b580', 'b60', 'b70', 'g31', '20.2.0']`，用于在模块级建立状态。

### Line 46 — Import `cutlass_cppgen`

```python
import cutlass_cppgen
```
**EN:** Imports `cutlass_cppgen` so later code can use these APIs at module scope.
**CN:** 导入 `cutlass_cppgen`，供后续代码在模块级使用这些 API。

### Line 47 — From `cutlass_cppgen.utils.datatypes` import

```python
from cutlass_cppgen.utils.datatypes import is_cupy_tensor, is_numpy_tensor, is_torch_tensor
```
**EN:** Imports `is_cupy_tensor, is_numpy_tensor, is_torch_tensor` from `cutlass_cppgen.utils.datatypes` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_cppgen.utils.datatypes` 导入 `is_cupy_tensor, is_numpy_tensor, is_torch_tensor`，以便后续代码在模块级复用共享定义。

### Line 48 — From `cutlass_library.arch_constants` import

```python
from cutlass_library.arch_constants import ( INTEL_XE12, INTEL_XE20)
```
**EN:** Imports `INTEL_XE12, INTEL_XE20` from `cutlass_library.arch_constants` to reuse shared definitions at module scope.
**CN:** 从 `cutlass_library.arch_constants` 导入 `INTEL_XE12, INTEL_XE20`，以便后续代码在模块级复用共享定义。

### Lines 51-60 — Function `check_cuda_errors`

```python
def check_cuda_errors(result: list):
    """
    Checks whether `result` contains a CUDA error raises the error as an exception, if so. Otherwise,
    returns the result contained in the remaining fields of `result`.

    :param result: the results of the `cudart` method, consisting of an error code and any method results
    :type result: list

    :return: non-error-code results from the `results` parameter
    """
```
**EN:** Defines function `check_cuda_errors` with parameters `result`. Purpose: Checks whether `result` contains a CUDA error raises the error as an exception, if so.
**CN:** 定义函数 `check_cuda_errors`，参数为 `result`。 其用途：Checks whether `result` contains a CUDA error raises the error as an exception, if so.

#### Line 61 — Comment or spacing block

```python
    # `result` is of the format : (cudaError_t, result...)
```
**EN:** Preserves standalone comments, separators, or formatting that clarify the surrounding code.
**CN:** 保留独立注释、分隔内容或格式信息，以帮助理解周围代码。

#### Line 62 — Assign `err`

```python
    err = result[0]
```
**EN:** Assigns `err` from `result[0]`, establishing state in function `check_cuda_errors`.
**CN:** 将 `err` 赋值为 `result[0]`，用于在函数 `check_cuda_errors` 中建立状态。

#### Lines 63-64 — Conditional `err.value`

```python
    if err.value:
        raise RuntimeError("CUDA error: {}".format(cudart.cudaGetErrorName(err)))
```
**EN:** Checks `err.value` and selects the matching branch in function `check_cuda_errors`.
**CN:** 检查 `err.value`，并在函数 `check_cuda_errors` 中选择匹配的分支。

#### Lines 66-71 — Conditional `len(result) == 1`

```python
    if len(result) == 1:
        return None
    elif len(result) == 2:
        return result[1]
    else:
        return result[1:]
```
**EN:** Checks `len(result) == 1` and selects the matching branch in function `check_cuda_errors`.
**CN:** 检查 `len(result) == 1`，并在函数 `check_cuda_errors` 中选择匹配的分支。

### Lines 74-83 — Function `device_cc`

```python
def device_cc(device: int = -1) -> int:
    """
    Returns the compute capability of the device with ID `device`.

    :param device: ID of the device to query
    :type device: int

    :return: compute capability of the queried device (e.g., 80 for SM80)
    :rtype: int
    """
```
**EN:** Defines function `device_cc` with parameters `device`. Purpose: Returns the compute capability of the device with ID `device`.
**CN:** 定义函数 `device_cc`，参数为 `device`。 其用途：Returns the compute capability of the device with ID `device`.

#### Lines 84-85 — Conditional `device == -1`

```python
    if device == -1:
        device = cutlass_cppgen.device_id()
```
**EN:** Checks `device == -1` and selects the matching branch in function `device_cc`.
**CN:** 检查 `device == -1`，并在函数 `device_cc` 中选择匹配的分支。

#### Lines 87-93 — Conditional `cutlass_cppgen._use_sycl`

```python
    if cutlass_cppgen._use_sycl:
        # Using '12' to encode Intel PVC as an integer in the expected format.
        intel_device = cutlass_cppgen._sycl_device
        device_name = intel_device.name.lower()
        if any(indicator in device_name for indicator in bmg_indicators):
            return INTEL_XE20
        return INTEL_XE12
```
**EN:** Checks `cutlass_cppgen._use_sycl` and selects the matching branch in function `device_cc`.
**CN:** 检查 `cutlass_cppgen._use_sycl`，并在函数 `device_cc` 中选择匹配的分支。

#### Line 95 — Assign `deviceProp`

```python
    deviceProp = check_cuda_errors(cudart.cudaGetDeviceProperties(device))
```
**EN:** Assigns `deviceProp` from `check_cuda_errors(cudart.cudaGetDeviceProperties(device))`, establishing state in function `device_cc`.
**CN:** 将 `deviceProp` 赋值为 `check_cuda_errors(cudart.cudaGetDeviceProperties(device))`，用于在函数 `device_cc` 中建立状态。

#### Line 96 — Assign `major`

```python
    major = str(deviceProp.major)
```
**EN:** Assigns `major` from `str(deviceProp.major)`, establishing state in function `device_cc`.
**CN:** 将 `major` 赋值为 `str(deviceProp.major)`，用于在函数 `device_cc` 中建立状态。

#### Line 97 — Assign `minor`

```python
    minor = str(deviceProp.minor)
```
**EN:** Assigns `minor` from `str(deviceProp.minor)`, establishing state in function `device_cc`.
**CN:** 将 `minor` 赋值为 `str(deviceProp.minor)`，用于在函数 `device_cc` 中建立状态。

#### Line 98 — Return

```python
    return int(major + minor)
```
**EN:** Returns `int(major + minor)` to the caller.
**CN:** 向调用方返回 `int(major + minor)`。

### Line 101 — Function `device_sm_count`

```python
def device_sm_count(device: int = -1):
```
**EN:** Defines function `device_sm_count` with parameters `device`.
**CN:** 定义函数 `device_sm_count`，参数为 `device`。

#### Lines 102-103 — Conditional `device == -1`

```python
    if device == -1:
        device = cutlass_cppgen.device_id()
```
**EN:** Checks `device == -1` and selects the matching branch in function `device_sm_count`.
**CN:** 检查 `device == -1`，并在函数 `device_sm_count` 中选择匹配的分支。

#### Lines 105-106 — Conditional `cutlass_cppgen._use_sycl`

```python
    if cutlass_cppgen._use_sycl:
        return cutlass_cppgen._sycl_device.max_compute_units
```
**EN:** Checks `cutlass_cppgen._use_sycl` and selects the matching branch in function `device_sm_count`.
**CN:** 检查 `cutlass_cppgen._use_sycl`，并在函数 `device_sm_count` 中选择匹配的分支。

#### Lines 108-110 — Assign `err, device_sm_count`

```python
    err, device_sm_count = cuda.cuDeviceGetAttribute(
        cuda.CUdevice_attribute.CU_DEVICE_ATTRIBUTE_MULTIPROCESSOR_COUNT, device
    )
```
**EN:** Assigns `err, device_sm_count` from `cuda.cuDeviceGetAttribute(cuda.CUdevice_attribute.CU_DEVICE_ATTRIBUTE_MULTIPROCESSOR_COUNT, device)`, establishing state in function `device_sm_count`.
**CN:** 将 `err, device_sm_count` 赋值为 `cuda.cuDeviceGetAttribute(cuda.CUdevice_attribute.CU_DEVICE_ATTRIBUTE_MULTIPROCESSOR_COUNT, device)`，用于在函数 `device_sm_count` 中建立状态。

#### Lines 111-115 — Conditional `err != cuda.CUresult.CUDA_SUCCESS`

```python
    if err != cuda.CUresult.CUDA_SUCCESS:
        raise Exception(
            "Failed to retireve SM count. "
            f"cuDeviceGetAttribute() failed with error: {cuda.cuGetErrorString(err)[1]}"
        )
```
**EN:** Checks `err != cuda.CUresult.CUDA_SUCCESS` and selects the matching branch in function `device_sm_count`.
**CN:** 检查 `err != cuda.CUresult.CUDA_SUCCESS`，并在函数 `device_sm_count` 中选择匹配的分支。

#### Line 117 — Return

```python
    return device_sm_count
```
**EN:** Returns `device_sm_count` to the caller.
**CN:** 向调用方返回 `device_sm_count`。

### Lines 120-129 — Function `to_device_ptr`

```python
def to_device_ptr(tensor) -> cuda.CUdeviceptr:
    """
    Converts a tensor to a CUdeviceptr

    :param tensor: tensor to convert
    :type tensor: np.ndarray | torch.Tensor | cp.ndarray | int

    :return: device pointer
    :rtype: cuda.CUdeviceptr
    """
```
**EN:** Defines function `to_device_ptr` with parameters `tensor`. Purpose: Converts a tensor to a CUdeviceptr
**CN:** 定义函数 `to_device_ptr`，参数为 `tensor`。 其用途：Converts a tensor to a CUdeviceptr

#### Lines 130-141 — Conditional `is_numpy_tensor(tensor)`

```python
    if is_numpy_tensor(tensor):
        ptr = cuda.CUdeviceptr(tensor.__array_interface__["data"][0])
    elif is_torch_tensor(tensor):
        ptr = cuda.CUdeviceptr(tensor.data_ptr())
    elif is_cupy_tensor(tensor):
        ptr = cuda.CUdeviceptr(int(tensor.data.ptr))
    elif isinstance(tensor, cuda.CUdeviceptr):
        ptr = tensor
    elif isinstance(tensor, int):
        ptr = cuda.CUdeviceptr(tensor)
    else:
        raise NotImplementedError(tensor)
```
**EN:** Checks `is_numpy_tensor(tensor)` and selects the matching branch in function `to_device_ptr`.
**CN:** 检查 `is_numpy_tensor(tensor)`，并在函数 `to_device_ptr` 中选择匹配的分支。

#### Line 143 — Return

```python
    return ptr
```
**EN:** Returns `ptr` to the caller.
**CN:** 向调用方返回 `ptr`。

### Line 146 — Function `default_stream`

```python
def default_stream():
```
**EN:** Defines function `default_stream` with parameters ``.
**CN:** 定义函数 `default_stream`，参数为 ``。

#### Lines 147-148 — Conditional `cutlass_cppgen._use_sycl`

```python
    if cutlass_cppgen._use_sycl:
        return dpctl.SyclQueue(cutlass_cppgen._sycl_device)
```
**EN:** Checks `cutlass_cppgen._use_sycl` and selects the matching branch in function `default_stream`.
**CN:** 检查 `cutlass_cppgen._use_sycl`，并在函数 `default_stream` 中选择匹配的分支。

#### Line 149 — Return

```python
    return cuda.CUstream(0)
```
**EN:** Returns `cuda.CUstream(0)` to the caller.
**CN:** 向调用方返回 `cuda.CUstream(0)`。

## Key Concepts / 关键概念

- **EN:** Top-level classes: none.
- **CN:** 顶层类：无。
- **EN:** Top-level functions: `check_cuda_errors`, `device_cc`, `device_sm_count`, `to_device_ptr`, `default_stream`.
- **CN:** 顶层函数：`check_cuda_errors`, `device_cc`, `device_sm_count`, `to_device_ptr`, `default_stream`。
- **EN:** This module organizes backend utilities, data structures, or runtime helpers used by cutlass_cppgen.
- **CN:** 该模块组织 cutlass_cppgen 所需的后端工具、数据结构或运行时辅助逻辑。

## Dependencies / 依赖关系

- **Project-local / 项目内:** `cutlass_cppgen`, `cutlass_cppgen.utils.datatypes`, `cutlass_cppgen.utils.lazy_import`, `cutlass_library.arch_constants`
- **Standard & third-party / 标准库与第三方:** `__future__`
- **EN:** Project-local imports connect this file to `cutlass_cppgen` and related CUTLASS helpers; the remaining imports provide Python runtime services, numeric types, or accelerator backends.
- **CN:** 项目内导入把该文件连接到 `cutlass_cppgen` 及相关 CUTLASS 辅助模块；其余导入则提供 Python 运行时服务、数值类型或加速器后端能力。
