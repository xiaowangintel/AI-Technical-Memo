# communicator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/managers/communicator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements communicator logic for runtime managers and coordination components. It exposes the classes, functions, and helpers that keep this part of the serving stack working. / 该模块实现与 communicator 相关的逻辑，并服务于 运行时管理与协调组件。它提供支撑这一服务链路所需的类、函数与辅助流程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Import runtime dependencies / 导入运行时依赖
```python
from __future__ import annotations
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 3-6: Import runtime dependencies / 导入运行时依赖
```python
import asyncio
import copy
from collections import deque
from typing import Deque, Generic, List, Optional, TypeVar
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 8-8: Import runtime dependencies / 导入运行时依赖
```python
import zmq
```
**EN:** This block gathers the standard-library, third-party, and local runtime modules that the rest of the file relies on.
**CN:** 该代码块汇集标准库、第三方库以及本地运行时模块，供后续实现复用。

### Lines 10-10: Provide supporting module logic / 提供辅助模块逻辑
```python
T = TypeVar("T")
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 13-23: Provide supporting module logic / 提供辅助模块逻辑
```python
class FanOutCommunicator(Generic[T]):
    """Fan-out request + collect response primitive over zmq.

    One send is fanned out to `fan_out` recipients; the caller awaits until
    all `fan_out` responses are collected. Supports two modes:
    - "queueing": requests are serialized; concurrent callers wait in a FIFO queue.
    - "watching": concurrent callers share a single in-flight request and all
      receive the same result when it completes.

    Only one request is in-flight at any time in either mode.
    """
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 25-33: Initialize FanOutCommunicator / 初始化 FanOutCommunicator
```python
    def __init__(self, sender: zmq.Socket, fan_out: int, mode="queueing"):
        self._sender = sender
        self._fan_out = fan_out
        self._mode = mode
        self._result_event: Optional[asyncio.Event] = None
        self._result_values: Optional[List[T]] = None
        self._ready_queue: Deque[asyncio.Event] = deque()

        assert mode in ["queueing", "watching"]
```
**EN:** This block implements the initializer `__init__(sender, fan_out, mode)` for `FanOutCommunicator`. It prepares the object state and connects the instance to the surrounding communicator workflow.
**CN:** 该代码块实现 `FanOutCommunicator` 的初始化方法 `__init__(sender, fan_out, mode)`。它负责准备对象状态，并把实例接入 communicator 相关的运行流程。

### Lines 35-55: Implement async queueing call / 实现异步queueing call
```python
    async def queueing_call(self, obj: T):
        ready_event = asyncio.Event()
        if self._result_event is not None or len(self._ready_queue) > 0:
            self._ready_queue.append(ready_event)
            await ready_event.wait()
            assert self._result_event is None
            assert self._result_values is None

        if obj is not None:
            self._sender.send_pyobj(obj)

        self._result_event = asyncio.Event()
        self._result_values = []
        await self._result_event.wait()
        result_values = self._result_values
        self._result_event = self._result_values = None

        if len(self._ready_queue) > 0:
            self._ready_queue.popleft().set()

        return result_values
```
**EN:** This block implements the async method `queueing_call(obj)` on `FanOutCommunicator`. It focuses on handling the communicator responsibilities represented by `queueing_call`, so the class can advance the communicator workflow in a self-contained way.
**CN:** 该代码块实现 `FanOutCommunicator` 上的异步方法 `queueing_call(obj)`。它围绕 `queueing_call` 所承担的 communicator 相关职责展开，使该类能够独立推进相应流程。

### Lines 57-75: Implement async watching call / 实现异步watching call
```python
    async def watching_call(self, obj):
        if self._result_event is None:
            assert self._result_values is None
            self._result_values = []
            self._result_event = asyncio.Event()

            if obj is not None:
                self._sender.send_pyobj(obj)

        # Capture local refs before await -- after event fires, the first
        # awakened coroutine clears shared state; later awaiters use local refs.
        values = self._result_values
        event = self._result_event
        await event.wait()

        result_values = copy.deepcopy(values)
        if self._result_event is event:
            self._result_event = self._result_values = None
        return result_values
```
**EN:** This block implements the async method `watching_call(obj)` on `FanOutCommunicator`. It focuses on handling the communicator responsibilities represented by `watching_call`, so the class can advance the communicator workflow in a self-contained way.
**CN:** 该代码块实现 `FanOutCommunicator` 上的异步方法 `watching_call(obj)`。它围绕 `watching_call` 所承担的 communicator 相关职责展开，使该类能够独立推进相应流程。

### Lines 77-81: Implement async call / 实现异步call
```python
    async def __call__(self, obj):
        if self._mode == "queueing":
            return await self.queueing_call(obj)
        else:
            return await self.watching_call(obj)
```
**EN:** This block implements the async method `__call__(obj)` on `FanOutCommunicator`. It focuses on handling the communicator responsibilities represented by `__call__`, so the class can advance the communicator workflow in a self-contained way.
**CN:** 该代码块实现 `FanOutCommunicator` 上的异步方法 `__call__(obj)`。它围绕 `__call__` 所承担的 communicator 相关职责展开，使该类能够独立推进相应流程。

### Lines 83-86: Implement handle recv / 实现handle recv
```python
    def handle_recv(self, recv_obj: T):
        self._result_values.append(recv_obj)
        if len(self._result_values) == self._fan_out:
            self._result_event.set()
```
**EN:** This block implements the method `handle_recv(recv_obj)` on `FanOutCommunicator`. It focuses on handling the communicator responsibilities represented by `handle_recv`, so the class can advance the communicator workflow in a self-contained way.
**CN:** 该代码块实现 `FanOutCommunicator` 上的方法 `handle_recv(recv_obj)`。它围绕 `handle_recv` 所承担的 communicator 相关职责展开，使该类能够独立推进相应流程。

### Lines 87-88: Provide supporting module logic / 提供辅助模块逻辑
```python

    @staticmethod
```
**EN:** This block contains supporting statements such as constants, small helpers, or configuration glue.
**CN:** 该代码块包含常量、小型辅助逻辑或配置衔接代码。

### Lines 89-93: Implement merge results / 实现merge results
```python
    def merge_results(results):
        all_success = all([r.success for r in results])
        all_message = [r.message for r in results]
        all_message = " | ".join(all_message)
        return all_success, all_message
```
**EN:** This block implements the method `merge_results(results)` on `FanOutCommunicator`. It focuses on handling the communicator responsibilities represented by `merge_results`, so the class can advance the communicator workflow in a self-contained way.
**CN:** 该代码块实现 `FanOutCommunicator` 上的方法 `merge_results(results)`。它围绕 `merge_results` 所承担的 communicator 相关职责展开，使该类能够独立推进相应流程。

## Key Concepts / 关键概念
- **Core types / 核心类型**: FanOutCommunicator
- **Domain focus / 领域焦点**: communicator / communicator
- **Concurrency / 并发特征**: async/await appears throughout the module / 模块中多处使用 async/await 协调并发流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: asyncio, collections, copy, typing
- **Third-party / 第三方库**: __future__, zmq
- **Local Modules / 本地模块**: None / 无
