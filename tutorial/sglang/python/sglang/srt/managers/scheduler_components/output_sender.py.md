# output_sender.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/scheduler_components/output_sender.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements output sender logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 output sender 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from typing import Optional, Union
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-3: Import runtime dependencies / 导入运行时依赖
```python
import zmq
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 5-5: Import runtime dependencies / 导入运行时依赖
```python
from sglang.srt.managers.io_struct import BaseBatchReq, BaseReq
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-28: Define class SenderWrapper / 定义类 SenderWrapper
```python
class SenderWrapper:
    def __init__(self, socket: zmq.Socket):
        self.socket = socket

    def send_output(
        self,
        output: Union[BaseReq, BaseBatchReq],
        recv_obj: Optional[Union[BaseReq, BaseBatchReq]] = None,
    ):
        if self.socket is None:
            return

        if (
            isinstance(recv_obj, BaseReq)
            and recv_obj.http_worker_ipc is not None
            and output.http_worker_ipc is None
        ):
            # handle communicator reqs for multi-http worker case
            output.http_worker_ipc = recv_obj.http_worker_ipc

        self.socket.send_pyobj(output)
```
**EN:** This block declares the class `SenderWrapper`. It centers on coordinating output sender behavior, with methods such as __init__, send_output.
**CN:** 该代码块声明类 `SenderWrapper`。它负责承载与 output sender 相关的核心状态与行为，并通过 __init__, send_output 等方法组织实现。

## Key Concepts / 关键概念
- **Core types / 核心类型**: SenderWrapper
- **Domain focus / 领域焦点**: output sender / output sender
- **Control style / 控制方式**: mostly synchronous orchestration and helper composition / 以同步编排与辅助逻辑组合为主

## Dependencies / 依赖关系
- **Standard Library / 标准库**: typing
- **Third-party / 第三方库**: zmq
- **Local Modules / 本地模块**: sglang.srt.managers.io_struct
