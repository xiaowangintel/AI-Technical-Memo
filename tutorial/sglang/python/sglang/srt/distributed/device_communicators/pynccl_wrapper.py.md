# pynccl_wrapper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/pynccl_wrapper.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `pynccl_wrapper`. It exposes primary entry points such as `find_nccl_library`, `ncclUniqueId`, `ncclDataTypeEnum`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `pynccl_wrapper` 的逻辑。 它对外提供的主要入口包括 `find_nccl_library`, `ncclUniqueId`, `ncclDataTypeEnum`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-38: Module imports, constants, and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.4.post1/vllm/distributed/device_communicators/pynccl.py

# This file is a pure Python wrapper for the NCCL library.
# The main purpose is to use NCCL combined with CUDA graph.
# Before writing this script, we tried the following approach:
# 1. We tried to use `cupy`, it calls NCCL correctly, but `cupy` itself
#  often gets stuck when initializing the NCCL communicator.
# 2. We tried to use `torch.distributed`, but `torch.distributed.all_reduce`
#  contains many other potential cuda APIs, that are not allowed during
#  capturing the CUDA graph. For further details, please check
# https://discuss.pytorch.org/t/pytorch-cudagraph-with-nccl-operation-failed/ .
#
# Another rejected idea is to write a C/C++ binding for NCCL. It is usually
# doable, but we often encounter issues related with nccl versions, and need
# to switch between different versions of NCCL. See
# https://github.com/NVIDIA/nccl/issues/1234 for more details.
# A C/C++ binding is not flexible enough to handle this. It requires
# recompilation of the code every time we want to switch between different
# versions. This current implementation, with a **pure** Python wrapper, is
# more flexible. We can easily switch between different versions of NCCL by
# changing the environment variable `SGLANG_NCCL_SO_PATH`, or the `so_file`
# variable in the code.

import ctypes
import logging
import os
import platform
from dataclasses import dataclass
from typing import Any, Dict, List, Optional

import torch
from torch.distributed import ReduceOp

logger = logging.getLogger(__name__)


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 39-65: Function find_nccl_library
```python
def find_nccl_library() -> str:
    """
    We either use the library file specified by the `SGLANG_NCCL_SO_PATH`
    environment variable, or we find the library file brought by PyTorch.
    After importing `torch`, `libnccl.so.2`, `librccl.so.1` or `libmccl.so.2`
    can be found by `ctypes` automatically.
    """

    # so_file can be set to None in sglang
    so_file = os.environ.get("SGLANG_NCCL_SO_PATH", None)

    # manually load the nccl library
    if so_file:
        logger.info(
            "Found nccl from environment variable SGLANG_NCCL_SO_PATH=%s", so_file
        )
    else:
        if torch.version.cuda is not None:
            so_file = "libnccl.so.2"
        elif torch.version.hip is not None:
            so_file = "librccl.so.1"
        elif hasattr(torch.version, "musa") and torch.version.musa is not None:
            so_file = "libmccl.so.2"
        else:
            raise ValueError("NCCL only supports CUDA, ROCm and MUSA backends.")
        logger.debug("Found nccl from library %s", so_file)
    return so_file
```
**EN:** This callable implements `find_nccl_library` and mainly implements find nccl library. The docstring states: "We either use the library file specified by the `SGLANG_NCCL_SO_PATH` environment variable, or we find the library file brought by PyTorch." In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `find_nccl_library`，主要用于实现 find nccl library 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 66-76: Module imports, constants, and setup
```python


# === export types and functions from nccl to Python ===
# for the original nccl definition, please check
# https://github.com/NVIDIA/nccl/blob/master/src/nccl.h.in

ncclResult_t = ctypes.c_int
ncclComm_t = ctypes.c_void_p
ncclWindow_t = ctypes.c_void_p


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 77-78: Class ncclUniqueId
```python
class ncclUniqueId(ctypes.Structure):
    _fields_ = [("internal", ctypes.c_byte * 128)]
```
**EN:** This range introduces `ncclUniqueId` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ncclUniqueId`，并定义其后续方法依赖的结构或元数据。

### Lines 79-86: Module-level constants and helpers
```python


cudaStream_t = ctypes.c_void_p
buffer_type = ctypes.c_void_p

ncclDataType_t = ctypes.c_int


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 87-104: Class ncclDataTypeEnum
```python
class ncclDataTypeEnum:
    ncclInt8 = 0
    ncclChar = 0
    ncclUint8 = 1
    ncclInt32 = 2
    ncclInt = 2
    ncclUint32 = 3
    ncclInt64 = 4
    ncclUint64 = 5
    ncclFloat16 = 6
    ncclHalf = 6
    ncclFloat32 = 7
    ncclFloat = 7
    ncclFloat64 = 8
    ncclDouble = 8
    ncclBfloat16 = 9
    ncclNumTypes = 10

```
**EN:** This range introduces `ncclDataTypeEnum` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ncclDataTypeEnum`，并定义其后续方法依赖的结构或元数据。

### Lines 105-123: Method ncclDataTypeEnum.from_torch
```python
    @classmethod
    def from_torch(cls, dtype: torch.dtype) -> int:
        if dtype == torch.int8:
            return cls.ncclInt8
        if dtype == torch.uint8:
            return cls.ncclUint8
        if dtype == torch.int32:
            return cls.ncclInt32
        if dtype == torch.int64:
            return cls.ncclInt64
        if dtype == torch.float16:
            return cls.ncclFloat16
        if dtype == torch.float32:
            return cls.ncclFloat32
        if dtype == torch.float64:
            return cls.ncclFloat64
        if dtype == torch.bfloat16:
            return cls.ncclBfloat16
        raise ValueError(f"Unsupported dtype: {dtype}")
```
**EN:** This callable implements `ncclDataTypeEnum.from_torch`. It takes `cls`, `dtype` and mainly constructs data from an external representation. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ncclDataTypeEnum.from_torch`。它接收 `cls`, `dtype`，主要用于从外部表示构造数据。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 124-128: Module-level constants and helpers
```python


ncclRedOp_t = ctypes.c_int


```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 129-136: Class ncclRedOpTypeEnum
```python
class ncclRedOpTypeEnum:
    ncclSum = 0
    ncclProd = 1
    ncclMax = 2
    ncclMin = 3
    ncclAvg = 4
    ncclNumOps = 5

```
**EN:** This range introduces `ncclRedOpTypeEnum` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ncclRedOpTypeEnum`，并定义其后续方法依赖的结构或元数据。

### Lines 137-149: Method ncclRedOpTypeEnum.from_torch
```python
    @classmethod
    def from_torch(cls, op: ReduceOp) -> int:
        if op == ReduceOp.SUM:
            return cls.ncclSum
        if op == ReduceOp.PRODUCT:
            return cls.ncclProd
        if op == ReduceOp.MAX:
            return cls.ncclMax
        if op == ReduceOp.MIN:
            return cls.ncclMin
        if op == ReduceOp.AVG:
            return cls.ncclAvg
        raise ValueError(f"Unsupported op: {op}")
```
**EN:** This callable implements `ncclRedOpTypeEnum.from_torch`. It takes `cls`, `op` and mainly constructs data from an external representation. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ncclRedOpTypeEnum.from_torch`。它接收 `cls`, `op`，主要用于从外部表示构造数据。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 152-156: Class Function
```python
@dataclass
class Function:
    name: str
    restype: Any
    argtypes: List[Any]
```
**EN:** This range introduces `Function` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `Function`，并定义其后续方法依赖的结构或元数据。

### Lines 159-218: Class NCCLLibrary (part 1/3)
```python
class NCCLLibrary:
    exported_functions = [
        # const char* ncclGetErrorString(ncclResult_t result)
        Function("ncclGetErrorString", ctypes.c_char_p, [ncclResult_t]),
        # ncclResult_t  ncclGetVersion(int *version);
        Function("ncclGetVersion", ncclResult_t, [ctypes.POINTER(ctypes.c_int)]),
        # ncclResult_t ncclGetUniqueId(ncclUniqueId* uniqueId);
        Function("ncclGetUniqueId", ncclResult_t, [ctypes.POINTER(ncclUniqueId)]),
        # ncclResult_t  ncclCommInitRank(
        #   ncclComm_t* comm, int nranks, ncclUniqueId commId, int rank);
        # note that ncclComm_t is a pointer type, so the first argument
        # is a pointer to a pointer
        Function(
            "ncclCommInitRank",
            ncclResult_t,
            [ctypes.POINTER(ncclComm_t), ctypes.c_int, ncclUniqueId, ctypes.c_int],
        ),
        # ncclResult_t  ncclAllReduce(
        #   const void* sendbuff, void* recvbuff, size_t count,
        #   ncclDataType_t datatype, ncclRedOp_t op, ncclComm_t comm,
        #   cudaStream_t stream);
        # note that cudaStream_t is a pointer type, so the last argument
        # is a pointer
        Function(
            "ncclAllReduce",
            ncclResult_t,
            [
                buffer_type,
                buffer_type,
                ctypes.c_size_t,
                ncclDataType_t,
                ncclRedOp_t,
                ncclComm_t,
                cudaStream_t,
            ],
        ),
        # ncclResult_t  ncclAllGather(
        #   const void* sendbuff, void* recvbuff, size_t count,
        #   ncclDataType_t datatype, ncclComm_t comm,
        #   cudaStream_t stream);
        # note that cudaStream_t is a pointer type, so the last argument
        # is a pointer
        Function(
            "ncclAllGather",
            ncclResult_t,
            [
                buffer_type,
                buffer_type,
                ctypes.c_size_t,
                ncclDataType_t,
                ncclComm_t,
                cudaStream_t,
            ],
        ),
        # ncclResult_t  ncclReduce(
        #   const void* sendbuff, void* recvbuff, size_t count,
        #   ncclDataType_t datatype, ncclRedOp_t op, int root,
        #   ncclComm_t comm,  cudaStream_t stream);
        # note that cudaStream_t is a pointer type, so the last argument
        # is a pointer
```
**EN:** This range introduces `NCCLLibrary` and defines the structure or metadata that its methods rely on. This chunk is part 1 of 3 for the same logical block.
**CN:** 这一段引入 `NCCLLibrary`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 1/3 部分。

### Lines 219-278: Class NCCLLibrary (part 2/3)
```python
        Function(
            "ncclReduce",
            ncclResult_t,
            [
                buffer_type,
                buffer_type,
                ctypes.c_size_t,
                ncclDataType_t,
                ncclRedOp_t,
                ctypes.c_int,
                ncclComm_t,
                cudaStream_t,
            ],
        ),
        # ncclResult_t  ncclReduceScatter(
        #   const void* sendbuff, void* recvbuff, size_t count,
        #   ncclDataType_t datatype, ncclRedOp_t op, ncclComm_t comm,
        #   cudaStream_t stream);
        # note that cudaStream_t is a pointer type, so the last argument
        # is a pointer
        Function(
            "ncclReduceScatter",
            ncclResult_t,
            [
                buffer_type,
                buffer_type,
                ctypes.c_size_t,
                ncclDataType_t,
                ncclRedOp_t,
                ncclComm_t,
                cudaStream_t,
            ],
        ),
        # ncclResult_t  ncclSend(
        #   const void* sendbuff, size_t count, ncclDataType_t datatype,
        #   int dest, ncclComm_t comm, cudaStream_t stream);
        Function(
            "ncclSend",
            ncclResult_t,
            [
                buffer_type,
                ctypes.c_size_t,
                ncclDataType_t,
                ctypes.c_int,
                ncclComm_t,
                cudaStream_t,
            ],
        ),
        # ncclResult_t  ncclRecv(
        #   void* recvbuff, size_t count, ncclDataType_t datatype,
        #   int src, ncclComm_t comm, cudaStream_t stream);
        Function(
            "ncclRecv",
            ncclResult_t,
            [
                buffer_type,
                ctypes.c_size_t,
                ncclDataType_t,
                ctypes.c_int,
                ncclComm_t,
```
**EN:** This range introduces `NCCLLibrary` and defines the structure or metadata that its methods rely on. This chunk is part 2 of 3 for the same logical block.
**CN:** 这一段引入 `NCCLLibrary`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 2/3 部分。

### Lines 279-335: Class NCCLLibrary (part 3/3)
```python
                cudaStream_t,
            ],
        ),
        # ncclResult_t ncclBroadcast(
        #   const void* sendbuff, void* recvbuff, size_t count,
        #   ncclDataType_t datatype, int root, ncclComm_t comm,
        #   cudaStream_t stream);
        Function(
            "ncclBroadcast",
            ncclResult_t,
            [
                buffer_type,
                buffer_type,
                ctypes.c_size_t,
                ncclDataType_t,
                ctypes.c_int,
                ncclComm_t,
                cudaStream_t,
            ],
        ),
        # be cautious! this is a collective call, it will block until all
        # processes in the communicator have called this function.
        # because Python object destruction can happen in random order,
        # it is better not to call it at all.
        # ncclResult_t  ncclCommDestroy(ncclComm_t comm);
        Function("ncclCommDestroy", ncclResult_t, [ncclComm_t]),
        # ncclResult_t ncclGroupStart();
        Function("ncclGroupStart", ncclResult_t, []),
        # ncclResult_t ncclGroupEnd();
        Function("ncclGroupEnd", ncclResult_t, []),
    ]

    exported_functions_symm_mem = [
        # ncclResult_t ncclCommWindowRegister(ncclComm_t comm, void* buff, size_t size, ncclWindow_t* win, int winFlags);
        Function(
            "ncclCommWindowRegister",
            ncclResult_t,
            [
                ncclComm_t,
                buffer_type,
                ctypes.c_size_t,
                ctypes.POINTER(ncclWindow_t),
                ctypes.c_int,
            ],
        ),
        # ncclResult_t ncclCommWindowDeregister(ncclComm_t comm, ncclWindow_t win);
        Function("ncclCommWindowDeregister", ncclResult_t, [ncclComm_t, ncclWindow_t]),
    ]

    # class attribute to store the mapping from the path to the library
    # to avoid loading the same library multiple times
    path_to_library_cache: Dict[str, Any] = {}

    # class attribute to store the mapping from library path
    #  to the corresponding dictionary
    path_to_dict_mapping: Dict[str, Dict[str, Any]] = {}

```
**EN:** This range introduces `NCCLLibrary` and defines the structure or metadata that its methods rely on. This chunk is part 3 of 3 for the same logical block. In this range it sets up imports and shared symbols; coordinates distributed communication.
**CN:** 这一段引入 `NCCLLibrary`，并定义其后续方法依赖的结构或元数据。 该片段是同一逻辑块的第 3/3 部分。 在这一范围内，它会建立导入关系并准备共享符号；协调分布式通信。

### Lines 336-370: Method NCCLLibrary.__init__
```python
    def __init__(self, so_file: Optional[str] = None):

        so_file = so_file or find_nccl_library()

        try:
            if so_file not in NCCLLibrary.path_to_dict_mapping:
                lib = ctypes.CDLL(so_file)
                NCCLLibrary.path_to_library_cache[so_file] = lib
            self.lib = NCCLLibrary.path_to_library_cache[so_file]
        except Exception as e:
            logger.error(
                "Failed to load NCCL library from %s . "
                "It is expected if you are not running on NVIDIA/AMD/MTHREADS GPUs. "
                "Otherwise, the nccl library might not exist, be corrupted "
                "or it does not support the current platform %s. "
                "If you already have the library, please set the "
                "environment variable SGLANG_NCCL_SO_PATH"
                " to point to the correct nccl library path.",
                so_file,
                platform.platform(),
            )
            raise e

        if so_file not in NCCLLibrary.path_to_dict_mapping:
            _funcs: Dict[str, Any] = {}
            exported_functions = NCCLLibrary.exported_functions
            if hasattr(self.lib, "ncclCommWindowRegister"):
                exported_functions.extend(NCCLLibrary.exported_functions_symm_mem)
            for func in exported_functions:
                f = getattr(self.lib, func.name)
                f.restype = func.restype
                f.argtypes = func.argtypes
                _funcs[func.name] = f
            NCCLLibrary.path_to_dict_mapping[so_file] = _funcs
        self._funcs = NCCLLibrary.path_to_dict_mapping[so_file]
```
**EN:** This callable implements `NCCLLibrary.__init__`. It takes `so_file` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols; performs defensive checks on invalid state; emits logs for diagnostics.
**CN:** 这一可调用对象实现了 `NCCLLibrary.__init__`。它接收 `so_file`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查；输出日志以便诊断。

### Lines 372-373: Method NCCLLibrary.ncclGetErrorString
```python
    def ncclGetErrorString(self, result: ncclResult_t) -> str:
        return self._funcs["ncclGetErrorString"](result).decode("utf-8")
```
**EN:** This callable implements `NCCLLibrary.ncclGetErrorString`. It takes `result` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclGetErrorString`。它接收 `result`，主要用于获取某个值或派生视图。

### Lines 375-378: Method NCCLLibrary.NCCL_CHECK
```python
    def NCCL_CHECK(self, result: ncclResult_t) -> None:
        if result != 0:
            error_str = self.ncclGetErrorString(result)
            raise RuntimeError(f"NCCL error: {error_str}")
```
**EN:** This callable implements `NCCLLibrary.NCCL_CHECK`. It takes `result` and mainly checks preconditions and compatibility. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `NCCLLibrary.NCCL_CHECK`。它接收 `result`，主要用于检查前置条件与兼容性。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 380-384: Method NCCLLibrary.ncclGetRawVersion
```python
    def ncclGetRawVersion(self) -> int:
        version = ctypes.c_int()
        self.NCCL_CHECK(self._funcs["ncclGetVersion"](ctypes.byref(version)))
        # something like 21903
        return version.value
```
**EN:** This callable implements `NCCLLibrary.ncclGetRawVersion` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclGetRawVersion`，主要用于获取某个值或派生视图。

### Lines 386-392: Method NCCLLibrary.ncclGetVersion
```python
    def ncclGetVersion(self) -> str:
        version_str = str(self.ncclGetRawVersion())
        # something like 21903 --> "2.19.3"
        major = version_str[0].lstrip("0")
        minor = version_str[1:3].lstrip("0")
        patch = version_str[3:].lstrip("0")
        return f"{major}.{minor}.{patch}"
```
**EN:** This callable implements `NCCLLibrary.ncclGetVersion` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclGetVersion`，主要用于获取某个值或派生视图。

### Lines 394-397: Method NCCLLibrary.ncclGetUniqueId
```python
    def ncclGetUniqueId(self) -> ncclUniqueId:
        unique_id = ncclUniqueId()
        self.NCCL_CHECK(self._funcs["ncclGetUniqueId"](ctypes.byref(unique_id)))
        return unique_id
```
**EN:** This callable implements `NCCLLibrary.ncclGetUniqueId` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclGetUniqueId`，主要用于获取某个值或派生视图。

### Lines 399-408: Method NCCLLibrary.ncclCommInitRank
```python
    def ncclCommInitRank(
        self, world_size: int, unique_id: ncclUniqueId, rank: int
    ) -> ncclComm_t:
        comm = ncclComm_t()
        self.NCCL_CHECK(
            self._funcs["ncclCommInitRank"](
                ctypes.byref(comm), world_size, unique_id, rank
            )
        )
        return comm
```
**EN:** This callable implements `NCCLLibrary.ncclCommInitRank`. It takes `world_size`, `unique_id`, `rank` and mainly implements nccl Comm Init Rank.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclCommInitRank`。它接收 `world_size`, `unique_id`, `rank`，主要用于实现 nccl Comm Init Rank 相关逻辑。

### Lines 410-429: Method NCCLLibrary.ncclAllReduce
```python
    def ncclAllReduce(
        self,
        sendbuff: buffer_type,
        recvbuff: buffer_type,
        count: int,
        datatype: int,
        op: int,
        comm: ncclComm_t,
        stream: cudaStream_t,
    ) -> None:
        # `datatype` actually should be `ncclDataType_t`
        # and `op` should be `ncclRedOp_t`
        # both are aliases of `ctypes.c_int`
        # when we pass int to a function, it will be converted to `ctypes.c_int`
        # by ctypes automatically
        self.NCCL_CHECK(
            self._funcs["ncclAllReduce"](
                sendbuff, recvbuff, count, datatype, op, comm, stream
            )
        )
```
**EN:** This callable implements `NCCLLibrary.ncclAllReduce`. It takes `sendbuff`, `recvbuff`, `count`, `datatype` and mainly reduces or aggregates values.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclAllReduce`。它接收 `sendbuff`, `recvbuff`, `count`, `datatype`，主要用于归约或聚合数值。

### Lines 431-451: Method NCCLLibrary.ncclReduce
```python
    def ncclReduce(
        self,
        sendbuff: buffer_type,
        recvbuff: buffer_type,
        count: int,
        datatype: int,
        op: int,
        root: int,
        comm: ncclComm_t,
        stream: cudaStream_t,
    ) -> None:
        # `datatype` actually should be `ncclDataType_t`
        # and `op` should be `ncclRedOp_t`
        # both are aliases of `ctypes.c_int`
        # when we pass int to a function, it will be converted to `ctypes.c_int`
        # by ctypes automatically
        self.NCCL_CHECK(
            self._funcs["ncclReduce"](
                sendbuff, recvbuff, count, datatype, op, root, comm, stream
            )
        )
```
**EN:** This callable implements `NCCLLibrary.ncclReduce`. It takes `sendbuff`, `recvbuff`, `count`, `datatype` and mainly reduces or aggregates values.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclReduce`。它接收 `sendbuff`, `recvbuff`, `count`, `datatype`，主要用于归约或聚合数值。

### Lines 453-472: Method NCCLLibrary.ncclReduceScatter
```python
    def ncclReduceScatter(
        self,
        sendbuff: buffer_type,
        recvbuff: buffer_type,
        count: int,
        datatype: int,
        op: int,
        comm: ncclComm_t,
        stream: cudaStream_t,
    ) -> None:
        # `datatype` actually should be `ncclDataType_t`
        # and `op` should be `ncclRedOp_t`
        # both are aliases of `ctypes.c_int`
        # when we pass int to a function, it will be converted to `ctypes.c_int`
        # by ctypes automatically
        self.NCCL_CHECK(
            self._funcs["ncclReduceScatter"](
                sendbuff, recvbuff, count, datatype, op, comm, stream
            )
        )
```
**EN:** This callable implements `NCCLLibrary.ncclReduceScatter`. It takes `sendbuff`, `recvbuff`, `count`, `datatype` and mainly reduces or aggregates values.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclReduceScatter`。它接收 `sendbuff`, `recvbuff`, `count`, `datatype`，主要用于归约或聚合数值。

### Lines 474-491: Method NCCLLibrary.ncclAllGather
```python
    def ncclAllGather(
        self,
        sendbuff: buffer_type,
        recvbuff: buffer_type,
        count: int,
        datatype: int,
        comm: ncclComm_t,
        stream: cudaStream_t,
    ) -> None:
        # `datatype` actually should be `ncclDataType_t`
        # which is an aliases of `ctypes.c_int`
        # when we pass int to a function, it will be converted to `ctypes.c_int`
        # by ctypes automatically
        self.NCCL_CHECK(
            self._funcs["ncclAllGather"](
                sendbuff, recvbuff, count, datatype, comm, stream
            )
        )
```
**EN:** This callable implements `NCCLLibrary.ncclAllGather`. It takes `sendbuff`, `recvbuff`, `count`, `datatype` and mainly implements nccl All Gather.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclAllGather`。它接收 `sendbuff`, `recvbuff`, `count`, `datatype`，主要用于实现 nccl All Gather 相关逻辑。

### Lines 493-504: Method NCCLLibrary.ncclSend
```python
    def ncclSend(
        self,
        sendbuff: buffer_type,
        count: int,
        datatype: int,
        dest: int,
        comm: ncclComm_t,
        stream: cudaStream_t,
    ) -> None:
        self.NCCL_CHECK(
            self._funcs["ncclSend"](sendbuff, count, datatype, dest, comm, stream)
        )
```
**EN:** This callable implements `NCCLLibrary.ncclSend`. It takes `sendbuff`, `count`, `datatype`, `dest` and mainly sends data to another component.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclSend`。它接收 `sendbuff`, `count`, `datatype`, `dest`，主要用于向其他组件发送数据。

### Lines 506-517: Method NCCLLibrary.ncclRecv
```python
    def ncclRecv(
        self,
        recvbuff: buffer_type,
        count: int,
        datatype: int,
        src: int,
        comm: ncclComm_t,
        stream: cudaStream_t,
    ) -> None:
        self.NCCL_CHECK(
            self._funcs["ncclRecv"](recvbuff, count, datatype, src, comm, stream)
        )
```
**EN:** This callable implements `NCCLLibrary.ncclRecv`. It takes `recvbuff`, `count`, `datatype`, `src` and mainly receives data from another component.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclRecv`。它接收 `recvbuff`, `count`, `datatype`, `src`，主要用于从其他组件接收数据。

### Lines 519-533: Method NCCLLibrary.ncclBroadcast
```python
    def ncclBroadcast(
        self,
        sendbuff: buffer_type,
        recvbuff: buffer_type,
        count: int,
        datatype: int,
        root: int,
        comm: ncclComm_t,
        stream: cudaStream_t,
    ) -> None:
        self.NCCL_CHECK(
            self._funcs["ncclBroadcast"](
                sendbuff, recvbuff, count, datatype, root, comm, stream
            )
        )
```
**EN:** This callable implements `NCCLLibrary.ncclBroadcast`. It takes `sendbuff`, `recvbuff`, `count`, `datatype` and mainly implements nccl Broadcast. In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclBroadcast`。它接收 `sendbuff`, `recvbuff`, `count`, `datatype`，主要用于实现 nccl Broadcast 相关逻辑。 在这一范围内，它会协调分布式通信。

### Lines 535-536: Method NCCLLibrary.ncclCommDestroy
```python
    def ncclCommDestroy(self, comm: ncclComm_t) -> None:
        self.NCCL_CHECK(self._funcs["ncclCommDestroy"](comm))
```
**EN:** This callable implements `NCCLLibrary.ncclCommDestroy`. It takes `comm` and mainly implements nccl Comm Destroy.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclCommDestroy`。它接收 `comm`，主要用于实现 nccl Comm Destroy 相关逻辑。

### Lines 538-547: Method NCCLLibrary.ncclCommWindowRegister
```python
    def ncclCommWindowRegister(
        self, comm: ncclComm_t, buff: buffer_type, size: int, win_flags: int
    ) -> ncclWindow_t:
        window = ncclWindow_t()
        self.NCCL_CHECK(
            self._funcs["ncclCommWindowRegister"](
                comm, buff, size, ctypes.byref(window), win_flags
            )
        )
        return window
```
**EN:** This callable implements `NCCLLibrary.ncclCommWindowRegister`. It takes `comm`, `buff`, `size`, `win_flags` and mainly registers hooks, handlers, or operators.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclCommWindowRegister`。它接收 `comm`, `buff`, `size`, `win_flags`，主要用于注册钩子、处理器或算子。

### Lines 549-550: Method NCCLLibrary.ncclCommWindowDeregister
```python
    def ncclCommWindowDeregister(self, comm: ncclComm_t, window: ncclWindow_t) -> None:
        self.NCCL_CHECK(self._funcs["ncclCommWindowDeregister"](comm, window))
```
**EN:** This callable implements `NCCLLibrary.ncclCommWindowDeregister`. It takes `comm`, `window` and mainly registers hooks, handlers, or operators.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclCommWindowDeregister`。它接收 `comm`, `window`，主要用于注册钩子、处理器或算子。

### Lines 552-553: Method NCCLLibrary.ncclGroupStart
```python
    def ncclGroupStart(self) -> None:
        self.NCCL_CHECK(self._funcs["ncclGroupStart"]())
```
**EN:** This callable implements `NCCLLibrary.ncclGroupStart` and mainly implements nccl Group Start.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclGroupStart`，主要用于实现 nccl Group Start 相关逻辑。

### Lines 555-556: Method NCCLLibrary.ncclGroupEnd
```python
    def ncclGroupEnd(self) -> None:
        self.NCCL_CHECK(self._funcs["ncclGroupEnd"]())
```
**EN:** This callable implements `NCCLLibrary.ncclGroupEnd` and mainly implements nccl Group End.
**CN:** 这一可调用对象实现了 `NCCLLibrary.ncclGroupEnd`，主要用于实现 nccl Group End 相关逻辑。

### Lines 557-567: Module-level constants and helpers
```python


__all__ = [
    "NCCLLibrary",
    "ncclDataTypeEnum",
    "ncclRedOpTypeEnum",
    "ncclUniqueId",
    "ncclComm_t",
    "cudaStream_t",
    "buffer_type",
]
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

## Key Concepts / 关键概念
- `find_nccl_library`: implements find nccl library / 实现 find nccl library 相关逻辑
- `ncclUniqueId`: core class or state container / 核心类或状态容器
- `ncclDataTypeEnum`: core class or state container / 核心类或状态容器
- `ncclRedOpTypeEnum`: core class or state container / 核心类或状态容器
- `Function`: core class or state container / 核心类或状态容器
- `NCCLLibrary`: core class or state container / 核心类或状态容器

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ctypes`, `logging`, `os`, `platform`, `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`
