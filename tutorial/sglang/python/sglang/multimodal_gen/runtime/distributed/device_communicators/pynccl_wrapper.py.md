# pynccl_wrapper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/device_communicators/pynccl_wrapper.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `ncclUniqueId`, `ncclDataTypeEnum`, and `ncclRedOpTypeEnum`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于分布式运行时层。它围绕 `ncclUniqueId`、`ncclDataTypeEnum` 和 `ncclRedOpTypeEnum` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 30-39: module setup and imports / 模块初始化与导入
```python
import ctypes
import platform
from dataclasses import dataclass
from typing import Any

import torch
from torch.distributed import ReduceOp

from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import find_nccl_library
```
**EN:** This block establishes the module context and imports `ctypes`, `platform`, `dataclasses`, `typing`, `torch`, and `torch.distributed`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `ctypes`、`platform`、`dataclasses`、`typing`、`torch` 和 `torch.distributed`。这些依赖为后续实现提供所需符号。

### Lines 41-48: supporting statements / 辅助语句
```python
logger = init_logger(__name__)

# === export types and functions from nccl to Python ===
# for the original nccl definition, please check
# https://github.com/NVIDIA/nccl/blob/master/src/nccl.h.in

ncclResult_t = ctypes.c_int
ncclComm_t = ctypes.c_void_p
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, `ncclResult_t`, and `ncclComm_t`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger`、`ncclResult_t` 和 `ncclComm_t` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 51-51: `ncclUniqueId` class overview / `ncclUniqueId` 类概览
```python
class ncclUniqueId(ctypes.Structure):
```
**EN:** This block defines class `ncclUniqueId`. It encapsulates nccl unique id behavior. It inherits from `ctypes.Structure`.
**CN:** 该代码块定义了类 `ncclUniqueId`。 它用于封装 nccl unique id 相关行为。 它继承自 `ctypes.Structure`。

### Lines 52-52: supporting statements / 辅助语句
```python
    _fields_ = [("internal", ctypes.c_byte * 128)]
```
**EN:** This block gathers supporting statements inside `ncclUniqueId`. It updates names such as `_fields_`.
**CN:** 该代码块汇集了位于 `ncclUniqueId` 内部的辅助语句。 它会更新 `_fields_` 等名称。

### Lines 55-58: supporting statements / 辅助语句
```python
cudaStream_t = ctypes.c_void_p
buffer_type = ctypes.c_void_p

ncclDataType_t = ctypes.c_int
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `cudaStream_t`, `buffer_type`, and `ncclDataType_t`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `cudaStream_t`、`buffer_type` 和 `ncclDataType_t` 等名称。

### Lines 61-61: `ncclDataTypeEnum` class overview / `ncclDataTypeEnum` 类概览
```python
class ncclDataTypeEnum:
```
**EN:** This block defines class `ncclDataTypeEnum`. It encapsulates nccl data type enum behavior.
**CN:** 该代码块定义了类 `ncclDataTypeEnum`。 它用于封装 nccl data type enum 相关行为。

### Lines 62-77: supporting statements / 辅助语句
```python
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
**EN:** This block gathers supporting statements inside `ncclDataTypeEnum`. It updates names such as `ncclInt8`, `ncclChar`, `ncclUint8`, `ncclInt32`, `ncclInt`, and `ncclUint32`.
**CN:** 该代码块汇集了位于 `ncclDataTypeEnum` 内部的辅助语句。 它会更新 `ncclInt8`、`ncclChar`、`ncclUint8`、`ncclInt32`、`ncclInt` 和 `ncclUint32` 等名称。

### Lines 79-97: `from_torch` implementation / `from_torch` 实现
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
**EN:** This block defines method `from_torch` on `ncclDataTypeEnum`. It constructs from torch. Key calls include `ValueError`. The implementation branches on conditions. Parameters such as `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `ncclDataTypeEnum` 的方法 `from_torch`。 它用于从…构造torch。 关键调用包括 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `dtype` 等参数驱动。

### Lines 100-100: supporting statements / 辅助语句
```python
ncclRedOp_t = ctypes.c_int
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `ncclRedOp_t`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `ncclRedOp_t` 等名称。

### Lines 103-103: `ncclRedOpTypeEnum` class overview / `ncclRedOpTypeEnum` 类概览
```python
class ncclRedOpTypeEnum:
```
**EN:** This block defines class `ncclRedOpTypeEnum`. It encapsulates nccl red op type enum behavior.
**CN:** 该代码块定义了类 `ncclRedOpTypeEnum`。 它用于封装 nccl red op type enum 相关行为。

### Lines 104-109: supporting statements / 辅助语句
```python
    ncclSum = 0
    ncclProd = 1
    ncclMax = 2
    ncclMin = 3
    ncclAvg = 4
    ncclNumOps = 5
```
**EN:** This block gathers supporting statements inside `ncclRedOpTypeEnum`. It updates names such as `ncclSum`, `ncclProd`, `ncclMax`, `ncclMin`, `ncclAvg`, and `ncclNumOps`.
**CN:** 该代码块汇集了位于 `ncclRedOpTypeEnum` 内部的辅助语句。 它会更新 `ncclSum`、`ncclProd`、`ncclMax`、`ncclMin`、`ncclAvg` 和 `ncclNumOps` 等名称。

### Lines 111-123: `from_torch` implementation / `from_torch` 实现
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
**EN:** This block defines method `from_torch` on `ncclRedOpTypeEnum`. It constructs from torch. Key calls include `ValueError`. The implementation branches on conditions. Parameters such as `op` drive the behavior in this section.
**CN:** 该代码块定义了 `ncclRedOpTypeEnum` 的方法 `from_torch`。 它用于从…构造torch。 关键调用包括 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `op` 等参数驱动。

### Lines 127-127: `Function` class overview / `Function` 类概览
```python
class Function:
```
**EN:** This block defines class `Function`. It encapsulates function behavior.
**CN:** 该代码块定义了类 `Function`。 它用于封装 function 相关行为。

### Lines 128-130: supporting statements / 辅助语句
```python
    name: str
    restype: Any
    argtypes: list[Any]
```
**EN:** This block gathers supporting statements inside `Function`. It updates names such as `name`, `restype`, and `argtypes`.
**CN:** 该代码块汇集了位于 `Function` 内部的辅助语句。 它会更新 `name`、`restype` 和 `argtypes` 等名称。

### Lines 133-133: `NCCLLibrary` class overview / `NCCLLibrary` 类概览
```python
class NCCLLibrary:
```
**EN:** This block defines class `NCCLLibrary`. It encapsulates nccllibrary behavior.
**CN:** 该代码块定义了类 `NCCLLibrary`。 它用于封装 nccllibrary 相关行为。

### Lines 134-267: supporting statements / 辅助语句
```python
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
    ]

    # class attribute to store the mapping from the path to the library
    # to avoid loading the same library multiple times
    path_to_library_cache: dict[str, Any] = {}

    # class attribute to store the mapping from library path
    #  to the corresponding dictionary
    path_to_dict_mapping: dict[str, dict[str, Any]] = {}
```
**EN:** This block gathers supporting statements inside `NCCLLibrary`. It updates names such as `exported_functions`, `path_to_library_cache`, and `path_to_dict_mapping`. The code collaborates with `Function`, and `ctypes.POINTER`.
**CN:** 该代码块汇集了位于 `NCCLLibrary` 内部的辅助语句。 它会更新 `exported_functions`、`path_to_library_cache` 和 `path_to_dict_mapping` 等名称。 代码会与 `Function` 和 `ctypes.POINTER` 协同工作。

### Lines 269-300: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, so_file: str | None = None):

        so_file = so_file or find_nccl_library()

        try:
            if so_file not in NCCLLibrary.path_to_dict_mapping:
                lib = ctypes.CDLL(so_file)
                NCCLLibrary.path_to_library_cache[so_file] = lib
            self.lib = NCCLLibrary.path_to_library_cache[so_file]
        except Exception as e:
            logger.error(
                "Failed to load NCCL library from %s ."
                "It is expected if you are not running on NVIDIA/AMD/MTHREADS GPUs."
                "Otherwise, the nccl library might not exist, be corrupted "
                "or it does not support the current platform %s."
                "If you already have the library, please set the "
                "environment variable SGLANG_DIFFUSION_NCCL_SO_PATH"
                " to point to the correct nccl library path.",
                so_file,
                platform.platform(),
            )
            raise e

        if so_file not in NCCLLibrary.path_to_dict_mapping:
            _funcs: dict[str, Any] = {}
            for func in NCCLLibrary.exported_functions:
                f = getattr(self.lib, func.name)
                f.restype = func.restype
                f.argtypes = func.argtypes
                _funcs[func.name] = f
            NCCLLibrary.path_to_dict_mapping[so_file] = _funcs
        self._funcs = NCCLLibrary.path_to_dict_mapping[so_file]
```
**EN:** This block defines method `__init__` on `NCCLLibrary`. It initializes the instance state. Key calls include `find_nccl_library`, `ctypes.CDLL`, `logger.error`, `getattr`, and `platform.platform`. The implementation branches on conditions, iterates over collections or steps, handles exceptional paths. Parameters such as `so_file` drive the behavior in this section.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `find_nccl_library`、`ctypes.CDLL`、`logger.error`、`getattr` 和 `platform.platform`。 实现中包含条件分支，会遍历集合或步骤，处理异常路径。 本段逻辑主要由 `so_file` 等参数驱动。

### Lines 302-303: `ncclGetErrorString` implementation / `ncclGetErrorString` 实现
```python
    def ncclGetErrorString(self, result: ncclResult_t) -> str:
        return str(self._funcs["ncclGetErrorString"](result).decode("utf-8"))
```
**EN:** This block defines method `ncclGetErrorString` on `NCCLLibrary`. It handles nccl get error string logic. Key calls include `str`, `self._funcs.decode`, and `self._funcs`. Parameters such as `result` drive the behavior in this section.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `ncclGetErrorString`。 它用于处理 nccl get error string 相关逻辑。 关键调用包括 `str`、`self._funcs.decode` 和 `self._funcs`。 本段逻辑主要由 `result` 等参数驱动。

### Lines 305-308: `NCCL_CHECK` implementation / `NCCL_CHECK` 实现
```python
    def NCCL_CHECK(self, result: ncclResult_t) -> None:
        if result != 0:
            error_str = self.ncclGetErrorString(result)
            raise RuntimeError(f"NCCL error: {error_str}")
```
**EN:** This block defines method `NCCL_CHECK` on `NCCLLibrary`. It handles nccl check logic. Key calls include `self.ncclGetErrorString`, and `RuntimeError`. The implementation branches on conditions. Parameters such as `result` drive the behavior in this section.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `NCCL_CHECK`。 它用于处理 nccl check 相关逻辑。 关键调用包括 `self.ncclGetErrorString` 和 `RuntimeError`。 实现中包含条件分支。 本段逻辑主要由 `result` 等参数驱动。

### Lines 310-318: `ncclGetVersion` implementation / `ncclGetVersion` 实现
```python
    def ncclGetVersion(self) -> str:
        version = ctypes.c_int()
        self.NCCL_CHECK(self._funcs["ncclGetVersion"](ctypes.byref(version)))
        version_str = str(version.value)
        # something like 21903 --> "2.19.3"
        major = version_str[0].lstrip("0")
        minor = version_str[1:3].lstrip("0")
        patch = version_str[3:].lstrip("0")
        return f"{major}.{minor}.{patch}"
```
**EN:** This block defines method `ncclGetVersion` on `NCCLLibrary`. It handles nccl get version logic. Key calls include `ctypes.c_int`, `self.NCCL_CHECK`, `str`, `version_str.lstrip`, and `self._funcs`.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `ncclGetVersion`。 它用于处理 nccl get version 相关逻辑。 关键调用包括 `ctypes.c_int`、`self.NCCL_CHECK`、`str`、`version_str.lstrip` 和 `self._funcs`。

### Lines 320-323: `ncclGetUniqueId` implementation / `ncclGetUniqueId` 实现
```python
    def ncclGetUniqueId(self) -> ncclUniqueId:
        unique_id = ncclUniqueId()
        self.NCCL_CHECK(self._funcs["ncclGetUniqueId"](ctypes.byref(unique_id)))
        return unique_id
```
**EN:** This block defines method `ncclGetUniqueId` on `NCCLLibrary`. It handles nccl get unique id logic. Key calls include `ncclUniqueId`, `self.NCCL_CHECK`, `self._funcs`, and `ctypes.byref`.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `ncclGetUniqueId`。 它用于处理 nccl get unique id 相关逻辑。 关键调用包括 `ncclUniqueId`、`self.NCCL_CHECK`、`self._funcs` 和 `ctypes.byref`。

### Lines 325-334: `ncclCommInitRank` implementation / `ncclCommInitRank` 实现
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
**EN:** This block defines method `ncclCommInitRank` on `NCCLLibrary`. It handles nccl comm init rank logic. Key calls include `ncclComm_t`, `self.NCCL_CHECK`, `self._funcs`, and `ctypes.byref`. Parameters such as `world_size`, `unique_id`, and `rank` drive the behavior in this section.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `ncclCommInitRank`。 它用于处理 nccl comm init rank 相关逻辑。 关键调用包括 `ncclComm_t`、`self.NCCL_CHECK`、`self._funcs` 和 `ctypes.byref`。 本段逻辑主要由 `world_size`、`unique_id` 和 `rank` 等参数驱动。

### Lines 336-355: `ncclAllReduce` implementation / `ncclAllReduce` 实现
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
**EN:** This block defines method `ncclAllReduce` on `NCCLLibrary`. It handles nccl all reduce logic. Key calls include `self.NCCL_CHECK`, and `self._funcs`. Parameters such as `sendbuff`, `recvbuff`, `count`, `datatype`, and `op` drive the behavior in this section.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `ncclAllReduce`。 它用于处理 nccl all reduce 相关逻辑。 关键调用包括 `self.NCCL_CHECK` 和 `self._funcs`。 本段逻辑主要由 `sendbuff`、`recvbuff`、`count`、`datatype` 和 `op` 等参数驱动。

### Lines 357-376: `ncclReduceScatter` implementation / `ncclReduceScatter` 实现
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
**EN:** This block defines method `ncclReduceScatter` on `NCCLLibrary`. It handles nccl reduce scatter logic. Key calls include `self.NCCL_CHECK`, and `self._funcs`. Parameters such as `sendbuff`, `recvbuff`, `count`, `datatype`, and `op` drive the behavior in this section.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `ncclReduceScatter`。 它用于处理 nccl reduce scatter 相关逻辑。 关键调用包括 `self.NCCL_CHECK` 和 `self._funcs`。 本段逻辑主要由 `sendbuff`、`recvbuff`、`count`、`datatype` 和 `op` 等参数驱动。

### Lines 378-395: `ncclAllGather` implementation / `ncclAllGather` 实现
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
**EN:** This block defines method `ncclAllGather` on `NCCLLibrary`. It handles nccl all gather logic. Key calls include `self.NCCL_CHECK`, and `self._funcs`. Parameters such as `sendbuff`, `recvbuff`, `count`, `datatype`, and `comm` drive the behavior in this section.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `ncclAllGather`。 它用于处理 nccl all gather 相关逻辑。 关键调用包括 `self.NCCL_CHECK` 和 `self._funcs`。 本段逻辑主要由 `sendbuff`、`recvbuff`、`count`、`datatype` 和 `comm` 等参数驱动。

### Lines 397-408: `ncclSend` implementation / `ncclSend` 实现
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
**EN:** This block defines method `ncclSend` on `NCCLLibrary`. It handles nccl send logic. Key calls include `self.NCCL_CHECK`, and `self._funcs`. Parameters such as `sendbuff`, `count`, `datatype`, `dest`, and `comm` drive the behavior in this section.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `ncclSend`。 它用于处理 nccl send 相关逻辑。 关键调用包括 `self.NCCL_CHECK` 和 `self._funcs`。 本段逻辑主要由 `sendbuff`、`count`、`datatype`、`dest` 和 `comm` 等参数驱动。

### Lines 410-421: `ncclRecv` implementation / `ncclRecv` 实现
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
**EN:** This block defines method `ncclRecv` on `NCCLLibrary`. It handles nccl recv logic. Key calls include `self.NCCL_CHECK`, and `self._funcs`. Parameters such as `recvbuff`, `count`, `datatype`, `src`, and `comm` drive the behavior in this section.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `ncclRecv`。 它用于处理 nccl recv 相关逻辑。 关键调用包括 `self.NCCL_CHECK` 和 `self._funcs`。 本段逻辑主要由 `recvbuff`、`count`、`datatype`、`src` 和 `comm` 等参数驱动。

### Lines 423-437: `ncclBroadcast` implementation / `ncclBroadcast` 实现
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
**EN:** This block defines method `ncclBroadcast` on `NCCLLibrary`. It handles nccl broadcast logic. Key calls include `self.NCCL_CHECK`, and `self._funcs`. Parameters such as `sendbuff`, `recvbuff`, `count`, `datatype`, and `root` drive the behavior in this section.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `ncclBroadcast`。 它用于处理 nccl broadcast 相关逻辑。 关键调用包括 `self.NCCL_CHECK` 和 `self._funcs`。 本段逻辑主要由 `sendbuff`、`recvbuff`、`count`、`datatype` 和 `root` 等参数驱动。

### Lines 439-440: `ncclCommDestroy` implementation / `ncclCommDestroy` 实现
```python
    def ncclCommDestroy(self, comm: ncclComm_t) -> None:
        self.NCCL_CHECK(self._funcs["ncclCommDestroy"](comm))
```
**EN:** This block defines method `ncclCommDestroy` on `NCCLLibrary`. It handles nccl comm destroy logic. Key calls include `self.NCCL_CHECK`, and `self._funcs`. Parameters such as `comm` drive the behavior in this section.
**CN:** 该代码块定义了 `NCCLLibrary` 的方法 `ncclCommDestroy`。 它用于处理 nccl comm destroy 相关逻辑。 关键调用包括 `self.NCCL_CHECK` 和 `self._funcs`。 本段逻辑主要由 `comm` 等参数驱动。

### Lines 443-451: supporting statements / 辅助语句
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
**EN:** This block gathers supporting statements at module scope. It updates names such as `__all__`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `__all__` 等名称。

## Key Concepts / 关键概念
- `ncclUniqueId`: Primary class that encapsulates nccl unique id behavior. / 核心类，用于封装 nccl unique id 相关行为。
- `ncclDataTypeEnum`: Primary class that encapsulates nccl data type enum behavior. / 核心类，用于封装 nccl data type enum 相关行为。
- `ncclRedOpTypeEnum`: Primary class that encapsulates nccl red op type enum behavior. / 核心类，用于封装 nccl red op type enum 相关行为。
- `Function`: Primary class that encapsulates function behavior. / 核心类，用于封装 function 相关行为。
- `NCCLLibrary`: Primary class that encapsulates nccllibrary behavior. / 核心类，用于封装 nccllibrary 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `ctypes`, `platform`, `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 451
