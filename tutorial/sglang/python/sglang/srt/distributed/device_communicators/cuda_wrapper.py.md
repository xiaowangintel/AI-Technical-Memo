# cuda_wrapper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/cuda_wrapper.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `cuda_wrapper`. The module docstring frames it as: "This file is a pure Python wrapper for the cudart library." / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `cuda_wrapper` 的逻辑。 它对外提供的主要入口包括 `cudaIpcMemHandle_t`, `Function`, `find_loaded_library`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/distributed/device_communicators/cuda_wrapper.py

"""This file is a pure Python wrapper for the cudart library.
It avoids the need to compile a separate shared library, and is
convenient for use when we just need to call a few functions.
"""

import ctypes
import logging
from dataclasses import dataclass
from typing import Any, Dict, List, Optional

# this line makes it possible to directly load `libcudart.so` using `ctypes`
import torch  # noqa

from sglang.srt.utils import is_musa

_is_musa = is_musa()

logger = logging.getLogger(__name__)

# === export types and functions from cudart to Python ===
# for the original cudart definition, please check
# https://docs.nvidia.com/cuda/cuda-runtime-api/index.html

cudaError_t = ctypes.c_int
cudaMemcpyKind = ctypes.c_int


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; prepares compilation-related behavior.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；处理与编译相关的行为。

### Lines 32-33: Class cudaIpcMemHandle_t
```python
class cudaIpcMemHandle_t(ctypes.Structure):
    _fields_ = [("internal", ctypes.c_byte * 128)]
```
**EN:** This range introduces `cudaIpcMemHandle_t` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `cudaIpcMemHandle_t`，并定义其后续方法依赖的结构或元数据。

### Lines 36-40: Class Function
```python
@dataclass
class Function:
    name: str
    restype: Any
    argtypes: List[Any]
```
**EN:** This range introduces `Function` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `Function`，并定义其后续方法依赖的结构或元数据。

### Lines 43-67: Function find_loaded_library
```python
def find_loaded_library(lib_name) -> Optional[str]:
    """
    According to according to https://man7.org/linux/man-pages/man5/proc_pid_maps.5.html,
    the file `/proc/self/maps` contains the memory maps of the process, which includes the
    shared libraries loaded by the process. We can use this file to find the path of the
    a loaded library.
    """  # noqa
    found = False
    with open("/proc/self/maps") as f:
        for line in f:
            if lib_name in line:
                found = True
                break
    if not found:
        # the library is not loaded in the current process
        return None
    # if lib_name is libcudart, we need to match a line with:
    # address /path/to/libcudart-hash.so.11.0
    start = line.index("/")
    path = line[start:].strip()
    filename = path.split("/")[-1]
    assert filename.rpartition(".so")[0].startswith(
        lib_name
    ), f"Unexpected filename: {filename} for library {lib_name}"
    return path
```
**EN:** This callable implements `find_loaded_library`. It takes `lib_name` and mainly loads external data or weights. The docstring states: "According to according to https://man7.org/linux/man-pages/man5/proc_pid_maps.5.html, the file `/proc/self/maps` contains the memory maps of the process, which includes the shared libraries loaded by the process." In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `find_loaded_library`。它接收 `lib_name`，主要用于加载外部数据或权重。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 70-119: Class CudaRTLibrary
```python
class CudaRTLibrary:
    exported_functions = [
        # ​cudaError_t cudaSetDevice ( int  device )
        Function("cudaSetDevice", cudaError_t, [ctypes.c_int]),
        # cudaError_t 	cudaDeviceSynchronize ( void )
        Function("cudaDeviceSynchronize", cudaError_t, []),
        # ​cudaError_t cudaDeviceReset ( void )
        Function("cudaDeviceReset", cudaError_t, []),
        # const char* 	cudaGetErrorString ( cudaError_t error )
        Function("cudaGetErrorString", ctypes.c_char_p, [cudaError_t]),
        # ​cudaError_t 	cudaMalloc ( void** devPtr, size_t size )
        Function(
            "cudaMalloc",
            cudaError_t,
            [ctypes.POINTER(ctypes.c_void_p), ctypes.c_size_t],
        ),
        # ​cudaError_t 	cudaFree ( void* devPtr )
        Function("cudaFree", cudaError_t, [ctypes.c_void_p]),
        # ​cudaError_t cudaMemset ( void* devPtr, int  value, size_t count )
        Function(
            "cudaMemset", cudaError_t, [ctypes.c_void_p, ctypes.c_int, ctypes.c_size_t]
        ),
        # ​cudaError_t cudaMemcpy ( void* dst, const void* src, size_t count, cudaMemcpyKind kind ) # noqa
        Function(
            "cudaMemcpy",
            cudaError_t,
            [ctypes.c_void_p, ctypes.c_void_p, ctypes.c_size_t, cudaMemcpyKind],
        ),
        # cudaError_t cudaIpcGetMemHandle ( cudaIpcMemHandle_t* handle, void* devPtr ) # noqa
        Function(
            "cudaIpcGetMemHandle",
            cudaError_t,
            [ctypes.POINTER(cudaIpcMemHandle_t), ctypes.c_void_p],
        ),
        # ​cudaError_t cudaIpcOpenMemHandle ( void** devPtr, cudaIpcMemHandle_t handle, unsigned int  flags ) # noqa
        Function(
            "cudaIpcOpenMemHandle",
            cudaError_t,
            [ctypes.POINTER(ctypes.c_void_p), cudaIpcMemHandle_t, ctypes.c_uint],
        ),
    ]

    # class attribute to store the mapping from the path to the library
    # to avoid loading the same library multiple times
    path_to_library_cache: Dict[str, Any] = {}

    # class attribute to store the mapping from library path
    #  to the corresponding dictionary
    path_to_dict_mapping: Dict[str, Dict[str, Any]] = {}

```
**EN:** This range introduces `CudaRTLibrary` and defines the structure or metadata that its methods rely on. In this range it sets up imports and shared symbols.
**CN:** 这一段引入 `CudaRTLibrary`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 120-137: Method CudaRTLibrary.__init__
```python
    def __init__(self, so_file: Optional[str] = None):
        if so_file is None:
            so_file = find_loaded_library("libcudart" if not _is_musa else "libmusart")
            assert so_file is not None, "libcudart is not loaded in the current process"
        if so_file not in CudaRTLibrary.path_to_library_cache:
            lib = ctypes.CDLL(so_file)
            CudaRTLibrary.path_to_library_cache[so_file] = lib
        self.lib = CudaRTLibrary.path_to_library_cache[so_file]

        if so_file not in CudaRTLibrary.path_to_dict_mapping:
            _funcs = {}
            for func in CudaRTLibrary.exported_functions:
                f = getattr(self.lib, func.name)
                f.restype = func.restype
                f.argtypes = func.argtypes
                _funcs[func.name] = f
            CudaRTLibrary.path_to_dict_mapping[so_file] = _funcs
        self.funcs = CudaRTLibrary.path_to_dict_mapping[so_file]
```
**EN:** This callable implements `CudaRTLibrary.__init__`. It takes `so_file` and mainly initializes instance state and defaults. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.__init__`。它接收 `so_file`，主要用于初始化实例状态与默认值。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 139-142: Method CudaRTLibrary.CUDART_CHECK
```python
    def CUDART_CHECK(self, result: cudaError_t) -> None:
        if result != 0:
            error_str = self.cudaGetErrorString(result)
            raise RuntimeError(f"CUDART error: {error_str}")
```
**EN:** This callable implements `CudaRTLibrary.CUDART_CHECK`. It takes `result` and mainly checks preconditions and compatibility. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.CUDART_CHECK`。它接收 `result`，主要用于检查前置条件与兼容性。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 144-145: Method CudaRTLibrary.cudaGetErrorString
```python
    def cudaGetErrorString(self, error: cudaError_t) -> str:
        return self.funcs["cudaGetErrorString"](error).decode("utf-8")
```
**EN:** This callable implements `CudaRTLibrary.cudaGetErrorString`. It takes `error` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.cudaGetErrorString`。它接收 `error`，主要用于获取某个值或派生视图。

### Lines 147-148: Method CudaRTLibrary.cudaSetDevice
```python
    def cudaSetDevice(self, device: int) -> None:
        self.CUDART_CHECK(self.funcs["cudaSetDevice"](device))
```
**EN:** This callable implements `CudaRTLibrary.cudaSetDevice`. It takes `device` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.cudaSetDevice`。它接收 `device`，主要用于将配置写入可变状态。

### Lines 150-151: Method CudaRTLibrary.cudaDeviceSynchronize
```python
    def cudaDeviceSynchronize(self) -> None:
        self.CUDART_CHECK(self.funcs["cudaDeviceSynchronize"]())
```
**EN:** This callable implements `CudaRTLibrary.cudaDeviceSynchronize` and mainly implements cuda Device Synchronize.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.cudaDeviceSynchronize`，主要用于实现 cuda Device Synchronize 相关逻辑。

### Lines 153-154: Method CudaRTLibrary.cudaDeviceReset
```python
    def cudaDeviceReset(self) -> None:
        self.CUDART_CHECK(self.funcs["cudaDeviceReset"]())
```
**EN:** This callable implements `CudaRTLibrary.cudaDeviceReset` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.cudaDeviceReset`，主要用于将配置写入可变状态。

### Lines 156-159: Method CudaRTLibrary.cudaMalloc
```python
    def cudaMalloc(self, size: int) -> ctypes.c_void_p:
        devPtr = ctypes.c_void_p()
        self.CUDART_CHECK(self.funcs["cudaMalloc"](ctypes.byref(devPtr), size))
        return devPtr
```
**EN:** This callable implements `CudaRTLibrary.cudaMalloc`. It takes `size` and mainly implements cuda Malloc.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.cudaMalloc`。它接收 `size`，主要用于实现 cuda Malloc 相关逻辑。

### Lines 161-162: Method CudaRTLibrary.cudaFree
```python
    def cudaFree(self, devPtr: ctypes.c_void_p) -> None:
        self.CUDART_CHECK(self.funcs["cudaFree"](devPtr))
```
**EN:** This callable implements `CudaRTLibrary.cudaFree`. It takes `devPtr` and mainly implements cuda Free.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.cudaFree`。它接收 `devPtr`，主要用于实现 cuda Free 相关逻辑。

### Lines 164-165: Method CudaRTLibrary.cudaMemset
```python
    def cudaMemset(self, devPtr: ctypes.c_void_p, value: int, count: int) -> None:
        self.CUDART_CHECK(self.funcs["cudaMemset"](devPtr, value, count))
```
**EN:** This callable implements `CudaRTLibrary.cudaMemset`. It takes `devPtr`, `value`, `count` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.cudaMemset`。它接收 `devPtr`, `value`, `count`，主要用于将配置写入可变状态。

### Lines 167-172: Method CudaRTLibrary.cudaMemcpy
```python
    def cudaMemcpy(
        self, dst: ctypes.c_void_p, src: ctypes.c_void_p, count: int
    ) -> None:
        cudaMemcpyDefault = 4
        kind = cudaMemcpyDefault
        self.CUDART_CHECK(self.funcs["cudaMemcpy"](dst, src, count, kind))
```
**EN:** This callable implements `CudaRTLibrary.cudaMemcpy`. It takes `dst`, `src`, `count` and mainly implements cuda Memcpy.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.cudaMemcpy`。它接收 `dst`, `src`, `count`，主要用于实现 cuda Memcpy 相关逻辑。

### Lines 174-179: Method CudaRTLibrary.cudaIpcGetMemHandle
```python
    def cudaIpcGetMemHandle(self, devPtr: ctypes.c_void_p) -> cudaIpcMemHandle_t:
        handle = cudaIpcMemHandle_t()
        self.CUDART_CHECK(
            self.funcs["cudaIpcGetMemHandle"](ctypes.byref(handle), devPtr)
        )
        return handle
```
**EN:** This callable implements `CudaRTLibrary.cudaIpcGetMemHandle`. It takes `devPtr` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.cudaIpcGetMemHandle`。它接收 `devPtr`，主要用于获取某个值或派生视图。

### Lines 181-189: Method CudaRTLibrary.cudaIpcOpenMemHandle
```python
    def cudaIpcOpenMemHandle(self, handle: cudaIpcMemHandle_t) -> ctypes.c_void_p:
        cudaIpcMemLazyEnablePeerAccess = 1
        devPtr = ctypes.c_void_p()
        self.CUDART_CHECK(
            self.funcs["cudaIpcOpenMemHandle"](
                ctypes.byref(devPtr), handle, cudaIpcMemLazyEnablePeerAccess
            )
        )
        return devPtr
```
**EN:** This callable implements `CudaRTLibrary.cudaIpcOpenMemHandle`. It takes `handle` and mainly implements cuda Ipc Open Mem Handle.
**CN:** 这一可调用对象实现了 `CudaRTLibrary.cudaIpcOpenMemHandle`。它接收 `handle`，主要用于实现 cuda Ipc Open Mem Handle 相关逻辑。

## Key Concepts / 关键概念
- `cudaIpcMemHandle_t`: core class or state container / 核心类或状态容器
- `Function`: core class or state container / 核心类或状态容器
- `find_loaded_library`: loads external data or weights / 加载外部数据或权重
- `CudaRTLibrary`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ctypes`, `logging`, `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.utils`
