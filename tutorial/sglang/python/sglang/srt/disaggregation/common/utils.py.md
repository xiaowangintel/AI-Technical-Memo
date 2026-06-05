# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/disaggregation/common/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file provides shared enums, constants, and helper routines used across the disaggregation stack. It centralizes configuration-sensitive logic so multiple components can reuse it. / 该文件提供解耦栈共用的枚举、常量与辅助函数，把依赖配置的逻辑集中起来供多个组件复用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Imports and module setup
```python
import struct
import threading
from collections import deque
from typing import List, Tuple

import numpy as np
import numpy.typing as npt


```
**EN:** This block gathers the imports and module-level setup for shared disaggregation helpers and enums. The imported modules show which runtime services, schemas, or backend components this file depends on.
**CN:** 这一段汇集了与共享解耦部署辅助函数与枚举相关的导入和模块级初始化。导入的模块展示了该文件所依赖的运行时服务、模式定义或后端组件。

### Lines 10-15: Function `pack_list_of_buffers`
```python
def pack_list_of_buffers(buffers: List[bytes]) -> bytes:
    if not buffers:
        return b""
    n = len(buffers)
    header = struct.pack(f"<{n+1}I", n, *(len(b) for b in buffers))
    return header + b"".join(buffers)
```
**EN:** This block defines the function `pack_list_of_buffers`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `pack_list_of_buffers`. Notable operations include `pack`, `join`.
**CN:** 这一段定义了function `pack_list_of_buffers`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `pack_list_of_buffers`。 值得注意的操作包括 `pack`、`join`。

### Lines 18-28: Function `unpack_list_of_buffers`
```python
def unpack_list_of_buffers(buf: bytes) -> List[bytes]:
    if buf == b"":
        return []
    (n,) = struct.unpack("<I", buf[:4])
    lens = struct.unpack(f"<{n}I", buf[4 : 4 + 4 * n])
    out = []
    offset = 4 + 4 * n
    for length in lens:
        out.append(buf[offset : offset + length])
        offset += length
    return out
```
**EN:** This block defines the function `unpack_list_of_buffers`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `unpack_list_of_buffers`. Notable operations include `unpack`, `append`.
**CN:** 这一段定义了function `unpack_list_of_buffers`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `unpack_list_of_buffers`。 值得注意的操作包括 `unpack`、`append`。

### Lines 31-32: Function `pack_int_lists`
```python
def pack_int_lists(lists, fmt: str) -> bytes:
    return pack_list_of_buffers([struct.pack(f"<{len(a)}{fmt}", *a) for a in lists])
```
**EN:** This block defines the function `pack_int_lists`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `pack_int_lists`. Notable operations include `pack_list_of_buffers`, `pack`.
**CN:** 这一段定义了function `pack_int_lists`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `pack_int_lists`。 值得注意的操作包括 `pack_list_of_buffers`、`pack`。

### Lines 35-40: Function `unpack_int_lists`
```python
def unpack_int_lists(buf: bytes, fmt: str) -> List[List[int]]:
    width = struct.calcsize(fmt)
    return [
        list(struct.unpack(f"<{len(b)//width}{fmt}", b))
        for b in unpack_list_of_buffers(buf)
    ]
```
**EN:** This block defines the function `unpack_int_lists`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `unpack_int_lists`. Notable operations include `calcsize`, `unpack`, `unpack_list_of_buffers`.
**CN:** 这一段定义了function `unpack_int_lists`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `unpack_int_lists`。 值得注意的操作包括 `calcsize`、`unpack`、`unpack_list_of_buffers`。

### Lines 43-43: Class `FastQueue` declaration
```python
class FastQueue:
```
**EN:** This block declares the class `FastQueue` and establishes its responsibility inside shared disaggregation helpers and enums. The surrounding comments and attributes frame the main state handled by the class. Definitions introduced here include `FastQueue`.
**CN:** 这一段声明了类 `FastQueue`，并说明它在共享解耦部署辅助函数与枚举中的职责。附近的注释与属性定义勾勒出该类需要维护的核心状态。 此处引入的定义包括 `FastQueue`。

### Lines 44-46: Method `__init__`
```python
    def __init__(self):
        self._buf = deque()
        self._cond = threading.Condition()
```
**EN:** This block defines the method `__init__` on `FastQueue`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `__init__`. Notable operations include `deque`, `Condition`.
**CN:** 这一段定义了method `__init__`（属于 `FastQueue`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `__init__`。 值得注意的操作包括 `deque`、`Condition`。

### Lines 48-52: Method `put`
```python
    def put(self, item):
        with self._cond:
            self._buf.append(item)
            # wake up a thread of wait()
            self._cond.notify()
```
**EN:** This block defines the method `put` on `FastQueue`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `put`. Notable operations include `append`, `wait`, `notify`.
**CN:** 这一段定义了method `put`（属于 `FastQueue`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `put`。 值得注意的操作包括 `append`、`wait`、`notify`。

### Lines 54-59: Method `get`
```python
    def get(self):
        with self._cond:
            # if queue is empty  ,block until is notified()
            while not self._buf:
                self._cond.wait()
            return self._buf.popleft()
```
**EN:** This block defines the method `get` on `FastQueue`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `get`. Notable operations include `notified`, `wait`, `popleft`.
**CN:** 这一段定义了method `get`（属于 `FastQueue`），介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `get`。 值得注意的操作包括 `notified`、`wait`、`popleft`。

### Lines 62-76: Function `group_concurrent_contiguous`
```python
def group_concurrent_contiguous(
    src_indices: npt.NDArray[np.int32], dst_indices: npt.NDArray[np.int32]
) -> Tuple[List[npt.NDArray[np.int32]], List[npt.NDArray[np.int32]]]:
    """Vectorised NumPy implementation."""
    if src_indices.size == 0:
        return [], []

    brk = np.where((np.diff(src_indices) != 1) | (np.diff(dst_indices) != 1))[0] + 1
    src_groups = np.split(src_indices, brk)
    dst_groups = np.split(dst_indices, brk)

    src_groups = [g.tolist() for g in src_groups]
    dst_groups = [g.tolist() for g in dst_groups]

    return src_groups, dst_groups
```
**EN:** This block defines the function `group_concurrent_contiguous`. It introduces the parameters, setup steps, and the main entry point for this piece of shared disaggregation helpers and enums. Definitions introduced here include `group_concurrent_contiguous`. Notable operations include `where`, `diff`, `split`, `tolist`.
**CN:** 这一段定义了function `group_concurrent_contiguous`，介绍了参数、初始化步骤，以及这部分共享解耦部署辅助函数与枚举逻辑的主要入口。 此处引入的定义包括 `group_concurrent_contiguous`。 值得注意的操作包括 `where`、`diff`、`split`、`tolist`。

## Key Concepts / 关键概念
- `pack_list_of_buffers`: Function that performs pack list of buffers for the surrounding workflow. / `pack_list_of_buffers`：在周边工作流中执行“packlistofbuffers”相关任务的函数。
- `unpack_list_of_buffers`: Function that performs unpack list of buffers for the surrounding workflow. / `unpack_list_of_buffers`：在周边工作流中执行“unpacklistofbuffers”相关任务的函数。
- `pack_int_lists`: Function that performs pack int lists for the surrounding workflow. / `pack_int_lists`：在周边工作流中执行“packintlists”相关任务的函数。
- `unpack_int_lists`: Function that performs unpack int lists for the surrounding workflow. / `unpack_int_lists`：在周边工作流中执行“unpackintlists”相关任务的函数。
- `FastQueue`: Class that encapsulates fast queue behavior in this module. / `FastQueue`：封装与“fastqueue”相关行为的类。
- `group_concurrent_contiguous`: Function that performs group concurrent contiguous for the surrounding workflow. / `group_concurrent_contiguous`：在周边工作流中执行“groupconcurrentcontiguous”相关任务的函数。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `struct`, `threading`, `collections`, `typing`
- **External packages / 外部依赖**: `numpy`
