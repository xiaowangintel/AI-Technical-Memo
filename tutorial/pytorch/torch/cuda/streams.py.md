# streams.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/cuda/streams.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-facing Python wrappers for streams, memory, random state, and runtime management.
- **Purpose (CN)**: 实现面向 CUDA 的 Python 包装层，用于流、内存、随机状态以及运行时管理。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
````python
# mypy: allow-untyped-defs
# pylint: disable=useless-parent-delegation
from __future__ import annotations

import ctypes

import torch
from torch._utils import _dummy_type


if not hasattr(torch._C, "_CudaStreamBase"):
    # Define dummy base classes
    torch._C.__dict__["_CudaStreamBase"] = _dummy_type("_CudaStreamBase")
    torch._C.__dict__["_CudaEventBase"] = _dummy_type("_CudaEventBase")


class Stream(torch._C._CudaStreamBase):
    r"""Wrapper around a CUDA stream.
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._utils; standard-library helpers such as __future__, ctypes. The future import postpones annotation evaluation, keeping type hints lightweight at import time. It introduces or extends `Stream`, which hold the main object-oriented state for this portion of the file. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._utils；标准库辅助模块，如 __future__、ctypes。 `__future__` 导入会推迟注解求值，从而在导入阶段降低类型提示的开销。 它引入或扩展了 `Stream`，这些类承载了本段涉及的主要面向对象状态。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 20-35 / 第 20-35 行
````python
    A CUDA stream is a linear sequence of execution that belongs to a specific
    device, independent from other streams. It supports with statement as a
    context manager to ensure the operators within the with block are running
    on the corresponding stream.  See :ref:`cuda-semantics` for details.

    Args:
        device(torch.device or int, optional): a device on which to allocate
            the stream. If :attr:`device` is ``None`` (default) or a negative
            integer, this will use the current device.
        priority(int, optional): priority of the stream, which can be positive, 0, or negative.
            A lower number indicates a higher priority. By default, the priority is set to 0.
            If the value falls outside of the allowed priority range, it will automatically be
            mapped to the nearest valid priority (lowest for large positive numbers or
            highest for large negative numbers).

    """
````
- **EN**: This chunk continues `Stream` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `Stream`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 37-55 / 第 37-55 行
````python
    def __new__(cls, device=None, priority=0, **kwargs):
        # Check CUDA availability
        if not torch.backends.cuda.is_built():
            raise RuntimeError("torch.cuda.Stream requires CUDA support")
        # setting device manager is expensive, so we avoid it unless necessary
        if device is None or ("stream_id" in kwargs and "device_index" in kwargs):
            return super().__new__(cls, priority=priority, **kwargs)
        else:
            with torch.cuda.device(device):
                return super().__new__(cls, priority=priority, **kwargs)

    def wait_event(self, event: Event | torch.Event) -> None:
        r"""Make all future work submitted to the stream wait for an event.

        Args:
            event (Event, torch.Event): an event to wait for.

        .. note:: This is a wrapper around ``cudaStreamWaitEvent()``: see
           `CUDA Stream documentation`_ for more info.
````
- **EN**: This chunk defines `wait_event`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `wait_event`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 57-72 / 第 57-72 行
````python
           This function returns without waiting for :attr:`event`: only future
           operations are affected.

        .. _CUDA Stream documentation:
           https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__STREAM.html
        """
        event.wait(self)

    def wait_stream(self, stream: Stream | torch.Stream) -> None:
        r"""Synchronize with another stream.

        All future work submitted to this stream will wait until all kernels
        submitted to a given stream at the time of call complete.

        Args:
            stream (Stream, torch.Stream): a stream to synchronize.
````
- **EN**: This chunk defines `wait_stream`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `wait_stream`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 74-92 / 第 74-92 行
````python
        .. note:: This function returns without waiting for currently enqueued
           kernels in :attr:`stream`: only future operations are affected.
        """
        self.wait_event(stream.record_event())

    def record_event(self, event: Event | torch.Event | None = None):
        r"""Record an event.

        Args:
            event (Event, torch.Event, optional): event to record. If not given, a new one
                will be allocated.

        Returns:
            Recorded event.
        """
        if event is None:
            event = Event()
        event.record(self)
        return event
````
- **EN**: This chunk defines `record_event`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `record_event`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 94-112 / 第 94-112 行
````python
    def query(self) -> bool:
        r"""Check if all the work submitted has been completed.

        Returns:
            A boolean indicating if all kernels in this stream are completed.
        """
        return super().query()

    def synchronize(self) -> None:
        r"""Wait for all the kernels in this stream to complete.

        .. note:: This is a wrapper around ``cudaStreamSynchronize()``: see
           `CUDA Stream documentation`_ for more info.
        """
        super().synchronize()

    @property
    def _as_parameter_(self):
        return ctypes.c_void_p(self.cuda_stream)
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `_as_parameter_`, which implements a focused helper used by the surrounding module. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `_as_parameter_`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 114-127 / 第 114-127 行
````python
    def __eq__(self, o) -> bool:
        if isinstance(o, Stream):
            return super().__eq__(o)
        return False

    def __hash__(self):
        return hash((self.cuda_stream, self.device))

    def __repr__(self):
        return f"<torch.cuda.Stream device={self.device} cuda_stream={self.cuda_stream:#x}>"

    def __cuda_stream__(self):
        """Implements the CUDA Stream Protocol:
        https://nvidia.github.io/cuda-python/cuda-core/latest/interoperability.html#cuda-stream-protocol
````
- **EN**: This chunk defines `__cuda_stream__`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__cuda_stream__`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 129-144 / 第 129-144 行
````python
        Returns:
            tuple: A 2-tuple of (version, handle) where version is the protocol version
                   and handle is the address of cudaStream_t (CUDA) or hipStream_t (ROCm) as a Python int.
        """
        return (0, self.cuda_stream)


class ExternalStream(Stream):
    r"""Wrapper around an externally allocated CUDA stream.

    This class is used to wrap streams allocated in other libraries in order
    to facilitate data exchange and multi-library interactions.

    .. note:: This class doesn't manage the stream life-cycle, it is the user
       responsibility to keep the referenced stream alive while this class is
       being used.
````
- **EN**: It introduces or extends `ExternalStream`, which hold the main object-oriented state for this portion of the file. This chunk continues `ExternalStream` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 它引入或扩展了 `ExternalStream`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `ExternalStream`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 146-164 / 第 146-164 行
````python
    Args:
        stream_ptr(int): Integer representation of the `cudaStream_t` value.
            allocated externally.
        device(torch.device or int, optional): the device where the stream
            was originally allocated. If device is specified incorrectly,
            subsequent launches using this stream may fail.
    """

    def __new__(cls, stream_ptr, device=None, **kwargs):
        with torch.cuda.device(device):
            return super().__new__(cls, stream_ptr=stream_ptr, **kwargs)


class Event(torch._C._CudaEventBase):
    r"""Wrapper around a CUDA event.

    CUDA events are synchronization markers that can be used to monitor the
    device's progress, to accurately measure timing, and to synchronize CUDA
    streams.
````
- **EN**: It introduces or extends `Event`, which hold the main object-oriented state for this portion of the file. This chunk defines `__new__`, which initializes object state and binds construction-time configuration. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics.
- **CN**: 它引入或扩展了 `Event`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__new__`，其作用是初始化对象状态并绑定构造期配置。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。

### Lines 166-181 / 第 166-181 行
````python
    The underlying CUDA events are lazily initialized when the event is first
    recorded or exported to another process. After creation, only streams on the
    same device may record the event. However, streams on any device can wait on
    the event.

    Args:
        enable_timing (bool, optional): indicates if the event should measure time
            (default: ``False``)
        blocking (bool, optional): if ``True``, :meth:`wait` will be blocking (default: ``False``)
        interprocess (bool): if ``True``, the event can be shared between processes
            (default: ``False``)
        external (bool, optional): indicates whether this event should create event record and event wait nodes, or create an internal cross-stream dependency, when captured in a cuda graph. See `cross-stream dependencies <https://docs.nvidia.com/cuda/archive/12.9.0/cuda-c-programming-guide/index.html#cross-stream-dependencies-and-events>`_, `cudaEventRecordExternal <https://docs.nvidia.com/cuda/archive/12.9.0/cuda-runtime-api/group__CUDART__TYPES.html#group__CUDART__TYPES_1g3457b81d1d32c6a00f6132fbc2693d47>`_, and `cudaEventWaitExternal <https://docs.nvidia.com/cuda/archive/12.9.0/cuda-runtime-api/group__CUDART__TYPES.html#group__CUDART__TYPES_1g0c23426b7252eaa9cef695859991304e>`_ for more information about internal vs. external events. (default: ``False``)

    .. _CUDA Event Documentation:
       https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__EVENT.html
    """
````
- **EN**: This chunk continues `Event` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `Event`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 183-200 / 第 183-200 行
````python
    def __new__(
        cls, enable_timing=False, blocking=False, interprocess=False, external=False
    ):
        return super().__new__(
            cls,
            enable_timing=enable_timing,
            blocking=blocking,
            interprocess=interprocess,
            external=external,
        )

    @classmethod
    def from_ipc_handle(cls, device, handle):
        r"""Reconstruct an event from an IPC handle on the given device."""
        return super().from_ipc_handle(device, handle)

    def record(self, stream: Stream | torch.Stream | None = None):
        r"""Record the event in a given stream.
````
- **EN**: This chunk defines `record`, which implements a focused helper used by the surrounding module. Decorators such as `classmethod` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `record`，其作用是实现周边模块使用的关键辅助逻辑。 像 `classmethod` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 202-215 / 第 202-215 行
````python
        Args:
            stream (Stream, torch.Stream, optional): Uses ``torch.cuda.current_stream()`` if no stream is specified.
                The stream's device must match the event's device.
        """
        if stream is None:
            stream = torch.cuda.current_stream()
        # pyrefly: ignore [bad-argument-type]
        super().record(stream)

    def wait(self, stream: Stream | torch.Stream | None = None) -> None:
        r"""Make all future work submitted to the given stream wait for this event.

        Args:
            stream (Stream, torch.Stream, optional): Uses ``torch.cuda.current_stream()`` if no stream is specified.
````
- **EN**: This chunk defines `wait`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `wait`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 217-235 / 第 217-235 行
````python
        .. note:: This is a wrapper around ``cudaStreamWaitEvent()``: see
            `CUDA Event documentation`_ for more info.
        """
        if stream is None:
            stream = torch.cuda.current_stream()
        # pyrefly: ignore [bad-argument-type]
        super().wait(stream)

    def query(self):
        r"""Check if all work currently captured by event has completed.

        Returns:
            A boolean indicating if all work currently captured by event has
            completed.
        """
        return super().query()

    def elapsed_time(self, end_event: Event):
        r"""Return the time elapsed.
````
- **EN**: This chunk defines `elapsed_time`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `elapsed_time`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 237-254 / 第 237-254 行
````python
        Time reported in milliseconds after the event was recorded and
        before the end_event was recorded.

        Args:
            end_event (Event): the end event.
        """
        return super().elapsed_time(end_event)

    def synchronize(self) -> None:
        r"""Wait for the event to complete.

        Waits until the completion of all work currently captured in this event.
        This prevents the CPU thread from proceeding until the event completes.

         .. note:: This is a wrapper around ``cudaEventSynchronize()``: see
            `CUDA Event documentation`_ for more info.
        """
        super().synchronize()
````
- **EN**: This chunk defines `synchronize`, which implements a focused helper used by the surrounding module. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `synchronize`，其作用是实现周边模块使用的关键辅助逻辑。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 256-271 / 第 256-271 行
````python
    def ipc_handle(self):
        r"""Return an IPC handle of this event.

        If not recorded yet, the event will use the current device.
        """
        return super().ipc_handle()

    @property
    def _as_parameter_(self):
        return ctypes.c_void_p(self.cuda_event)

    def __repr__(self) -> str:
        if self.cuda_event:
            return f"<torch.cuda.Event {self._as_parameter_.value:#x}>"
        else:
            return "<torch.cuda.Event uninitialized>"
````
- **EN**: The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface. This chunk defines `__repr__`, which implements a focused helper used by the surrounding module. Decorators such as `property` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Calls to `super()` delegate the low-level work to a base class or C++-backed implementation while the Python layer adds policy or ergonomics. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。 这一段定义了 `__repr__`，其作用是实现周边模块使用的关键辅助逻辑。 像 `property` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 对 `super()` 的调用会把底层工作委托给基类或 C++ 支撑的实现，而 Python 层主要补充策略或易用性。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **CUDA runtime**
  - EN: Wraps streams, devices, memory state, and CUDA-specific coordination points.
  - CN: 封装流、设备、内存状态以及 CUDA 专用协同点。
- **Stream**
  - EN: `Stream` is one of the main symbols declared or implemented in this file.
  - CN: `Stream` 是本文件声明或实现的主要符号之一。
- **ExternalStream**
  - EN: `ExternalStream` is one of the main symbols declared or implemented in this file.
  - CN: `ExternalStream` 是本文件声明或实现的主要符号之一。
- **Graph handling**
  - EN: The implementation manipulates captured graphs, graph metadata, or graph-scoped execution state.
  - CN: 实现会处理捕获图、图元数据或图作用域的执行状态。
- **Stream coordination**
  - EN: The code exposes stream-aware state so asynchronous execution can be controlled from Python.
  - CN: 代码暴露与流相关的状态，使异步执行可从 Python 侧进行控制。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._utils`
- **Standard library / 标准库**: `__future__`, `ctypes`
- **Primary symbols in this file / 本文件核心符号**: `Stream`, `ExternalStream`, `Event`
