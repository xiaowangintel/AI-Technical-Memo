# _sanitizer.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/cuda/_sanitizer.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-facing Python wrappers for streams, memory, random state, and runtime management.
- **Purpose (CN)**: 实现面向 CUDA 的 Python 包装层，用于流、内存、随机状态以及运行时管理。
## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行
````python
# mypy: allow-untyped-defs
r"""
This module introduces CUDA Sanitizer, a tool for detecting synchronization errors between kernels ran on different streams.

It stores information on accesses to tensors to determine if they are synchronized
or not. When enabled in a python program and a possible data race is detected, a
detailed warning will be printed and the program will exit.

It can be enabled either by importing this module and calling
:func:`enable_cuda_sanitizer()` or by exporting the ``TORCH_CUDA_SANITIZER``
environment variable.
"""

import enum
import functools
import inspect
import io
import logging
import re
import sys
import textwrap
import traceback
from collections.abc import Iterator
from dataclasses import dataclass, field
from typing import Any, TypeVar
````
- **EN**: This block assembles module dependencies, pulling in standard-library helpers such as enum, functools, inspect, .... The docstring documents user-facing behavior, caveats, and examples for the surrounding API surface.
- **CN**: 这一段组织模块依赖，引入了标准库辅助模块，如 enum、functools、inspect、...。 文档字符串说明了周边 API 的用户可见行为、注意事项以及示例。

### Lines 27-50 / 第 27-50 行
````python
import torch
import torch.cuda._gpu_trace as gpu_trace
from torch.utils import _pytree as pytree
from torch.utils._python_dispatch import TorchDispatchMode


aten = torch.ops.aten

DEFAULT_STREAM_ID = 0

TK = TypeVar("TK")
TVa = TypeVar("TVa")
TVb = TypeVar("TVb")

DataPtr = int
StreamId = int
EventId = int
SeqNum = int

logger = logging.getLogger(__name__)

# Note that this is only factories that take Tensor as input as they are
# the ones we care about.
FACTORY_FUNCTION_REGEX = re.compile("(new_.*|.*_like)")
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch.cuda._gpu_trace, torch.utils, ....
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch.cuda._gpu_trace、torch.utils、...。

### Lines 53-74 / 第 53-74 行
````python
class AccessType(enum.Enum):
    READ = enum.auto()
    WRITE = enum.auto()

    def __str__(self):
        return "reading from" if self is AccessType.READ else "writing to"


@dataclass
class Access:
    r"""Stores information about a single access to a tensor by a kernel.

    Args:
        type: either AccessType.READ or AccessType.Write.
        seq_num: the sequential number of the kernel performing the access.
        stream: the stream id of the stream executing the kernel.
        operator: the schema of the launched kernel, which lists the
            arguments and return type.
        aliases: the arguments in the schema this access corresponds to.
        is_output: Whether the tensor was an output of the kernel.
        stack_trace: the stack summary object captured during access.
    """
````
- **EN**: It introduces or extends `AccessType`, `Access`, which hold the main object-oriented state for this portion of the file. This chunk defines `__str__`, which implements a focused helper used by the surrounding module. Decorators such as `dataclass` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `AccessType`、`Access`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__str__`，其作用是实现周边模块使用的关键辅助逻辑。 像 `dataclass` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 76-102 / 第 76-102 行
````python
    type: AccessType
    seq_num: SeqNum
    stream: StreamId
    operator: str
    aliases: list[str]
    is_output: bool
    stack_trace: traceback.StackSummary


class SynchronizationError(Exception):
    """Base class for errors detected by CUDA Sanitizer."""


class UnsynchronizedAccessError(SynchronizationError):
    """Stores information about two unsynchronized accesses to one data pointer."""

    def __init__(
        self,
        data_ptr: DataPtr,
        allocation_stack_trace: traceback.StackSummary | None,
        current_access: Access,
        previous_access: Access,
    ):
        self.data_ptr = data_ptr
        self.allocation_stack_trace = allocation_stack_trace
        self.current_access = current_access
        self.previous_access = previous_access
````
- **EN**: It introduces or extends `SynchronizationError`, `UnsynchronizedAccessError`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration.
- **CN**: 它引入或扩展了 `SynchronizationError`、`UnsynchronizedAccessError`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。

### Lines 104-127 / 第 104-127 行
````python
    def __str__(self):
        def format_access(access: Access):
            message.write(f"{access.operator}\n{access.type}")
            if access.aliases:
                message.write(" argument(s) " + ", ".join(access.aliases))
                if access.is_output:
                    message.write(", and to")
            if access.is_output:
                message.write(" the output")
            message.write(
                f"\nWith stack trace:\n{''.join(access.stack_trace.format())}\n"
            )

        with io.StringIO() as message:
            message.write(
                textwrap.dedent(
                    f"""\
                    ============================
                    CSAN detected a possible data race on tensor with data pointer {self.data_ptr}
                    Access by stream {self.current_access.stream} during kernel:
                    """
                )
            )
            format_access(self.current_access)
````
- **EN**: This chunk defines `format_access`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `format_access`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 129-151 / 第 129-151 行
````python
            message.write(
                f"Previous access by stream {self.previous_access.stream} during kernel:\n"
            )
            format_access(self.previous_access)

            if self.allocation_stack_trace:
                message.write(
                    "Tensor was allocated with stack trace:\n"
                    f"{''.join(self.allocation_stack_trace.format())}"
                )
            else:
                message.write("Trace for tensor allocation not found.")
            return message.getvalue()


class CUDASanitizerErrors(Exception):
    """Wrapper class for errors reported by CUDA Sanitizer."""

    def __init__(self, errors: list[SynchronizationError]):
        self.errors = errors

    def __str__(self):
        return f"detected {len(self.errors)} errors"
````
- **EN**: It introduces or extends `CUDASanitizerErrors`, which hold the main object-oriented state for this portion of the file. This chunk defines `__str__`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `CUDASanitizerErrors`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__str__`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 154-173 / 第 154-173 行
````python
@dataclass
class TensorInfo:
    r"""Stores information about a single tensor and recent accesses to it.

    Args:
        allocation_stack_trace: the stack summary object captured during tensor
            allocation. Can be ``None`` if the allocation wasn't caught by CSAN.
        reads: list of read accesses to the tensor that were performed since
            the last write.
        write: the last write access to the tensor.
    """

    allocation_stack_trace: traceback.StackSummary | None
    reads: list[Access] = field(default_factory=list)
    write: Access | None = None


class _TensorsAccessed:
    def __init__(self) -> None:
        self.accesses: dict[DataPtr, TensorInfo] = {}
````
- **EN**: It introduces or extends `TensorInfo`, `_TensorsAccessed`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. Decorators such as `dataclass` adjust how the following definitions participate in typing, warnings, or runtime protocol hooks.
- **CN**: 它引入或扩展了 `TensorInfo`、`_TensorsAccessed`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 像 `dataclass` 这样的装饰器会调整后续定义在类型系统、告警或运行时协议中的参与方式。

### Lines 175-199 / 第 175-199 行
````python
    def ensure_tensor_exists(self, data_ptr: DataPtr) -> None:
        if data_ptr not in self.accesses:
            logger.info(
                "Found tensor with pointer: %s, but no matching tensor "
                "allocation in the trace. Backfilling the trace now. "
                "Perhaps the sanitizer was enabled after some torch operations?",
                data_ptr,
            )
            self.create_tensor(data_ptr, None)

    def ensure_tensor_does_not_exist(self, data_ptr: DataPtr) -> None:
        if data_ptr in self.accesses:
            logger.info(
                "Found duplicate tensor allocation in the trace for tensor with "
                "pointer: %s. Assuming the trace for tensor deallocation "
                "wasn't caught and backfilling it now. "
                "Perhaps the sanitizer was enabled after some torch operations?",
                data_ptr,
            )
            self.delete_tensor(data_ptr)

    def create_tensor(
        self, data_ptr: DataPtr, stack_trace: traceback.StackSummary | None
    ) -> None:
        self.accesses[data_ptr] = TensorInfo(stack_trace)
````
- **EN**: This chunk defines `create_tensor`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `create_tensor`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 201-223 / 第 201-223 行
````python
    def delete_tensor(self, data_ptr: DataPtr) -> None:
        del self.accesses[data_ptr]

    def were_there_reads_since_last_write(self, data_ptr: DataPtr) -> bool:
        return bool(self.accesses[data_ptr].reads)

    def get_allocation_stack_trace(
        self, data_ptr: DataPtr
    ) -> traceback.StackSummary | None:
        return self.accesses[data_ptr].allocation_stack_trace

    def get_write(self, data_ptr: DataPtr) -> Access | None:
        return self.accesses[data_ptr].write

    def get_reads(self, data_ptr: DataPtr) -> list[Access]:
        return self.accesses[data_ptr].reads

    def add_read(self, data_ptr: DataPtr, access: Access) -> None:
        self.accesses[data_ptr].reads.append(access)

    def set_write(self, data_ptr: DataPtr, access: Access) -> None:
        self.accesses[data_ptr].write = access
        self.accesses[data_ptr].reads = []
````
- **EN**: This chunk defines `set_write`, which mutates configuration or backend state that affects later execution. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `set_write`，其作用是修改会影响后续执行的配置或后端状态。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 226-251 / 第 226-251 行
````python
class StreamSynchronizations:
    def __init__(self) -> None:
        self.current_sync_states: dict[StreamId, dict[StreamId, SeqNum]] = {}
        self.recorded_sync_states: dict[EventId, dict[StreamId, SeqNum]] = {}
        self.host_sync_state: dict[StreamId, SeqNum] = {}
        self.create_stream(DEFAULT_STREAM_ID)

    def _ensure_stream_exists(self, stream: StreamId) -> None:
        if stream not in self.current_sync_states:
            logger.info(
                "Found Stream with id: %s, but no matching stream "
                "creation in the trace. Backfilling the trace now. "
                "Perhaps the sanitizer was enabled after some torch operations?",
                stream,
            )
            self.create_stream(stream)

    def _ensure_event_exists(self, event: EventId) -> None:
        if event not in self.recorded_sync_states:
            logger.info(
                "Found Event with id: %s, but no matching event "
                "creation in the trace. Backfilling the trace now. "
                "Perhaps the sanitizer was enabled after some torch operations?",
                event,
            )
            self.create_event(event)
````
- **EN**: It introduces or extends `StreamSynchronizations`, which hold the main object-oriented state for this portion of the file. This chunk defines `_ensure_event_exists`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 它引入或扩展了 `StreamSynchronizations`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `_ensure_event_exists`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 253-278 / 第 253-278 行
````python
    def _ensure_event_does_not_exist(self, event: EventId) -> None:
        if event in self.recorded_sync_states:
            logger.info(
                "Found duplicate event creation in the trace for event with "
                "id: %s. Assuming the trace for event deletion wasn't caught "
                "and backfilling it now. "
                "Perhaps the sanitizer was enabled after some torch operations?",
                event,
            )
            self.delete_event(event)

    def create_stream(self, stream: StreamId) -> None:
        if stream in self.current_sync_states:
            logger.info(
                "Found duplicate Stream creation in the trace for Stream with "
                "id: %s. PyTorch Streams are only created once, so this "
                "trace entry is ignored.",
                stream,
            )
        else:
            self.host_sync_state[stream] = 0
            self.current_sync_states[stream] = self.host_sync_state.copy()

    def create_event(self, event: EventId) -> None:
        self._ensure_event_does_not_exist(event)
        self.recorded_sync_states[event] = {}
````
- **EN**: This chunk defines `create_event`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `create_event`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 280-304 / 第 280-304 行
````python
    def delete_event(self, event: EventId) -> None:
        self._ensure_event_exists(event)
        del self.recorded_sync_states[event]

    def update_seq_num(self, stream: StreamId, seq_num: SeqNum) -> None:
        self._ensure_stream_exists(stream)
        self.current_sync_states[stream][stream] = seq_num

    def record_state(self, event: EventId, stream: StreamId) -> None:
        self._ensure_event_exists(event)
        self._ensure_stream_exists(stream)
        self.recorded_sync_states[event] = self.current_sync_states[stream].copy()

    def _state_wait_for_other(
        self, state: dict[StreamId, SeqNum], other: dict[StreamId, SeqNum]
    ) -> None:
        for stream, seq_num in other.items():
            state[stream] = max(state.get(stream, -1), seq_num)

    def stream_wait_for_event(self, stream: StreamId, event: EventId) -> None:
        self._ensure_stream_exists(stream)
        self._ensure_event_exists(event)
        self._state_wait_for_other(
            self.current_sync_states[stream], self.recorded_sync_states[event]
        )
````
- **EN**: This chunk defines `stream_wait_for_event`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `stream_wait_for_event`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 306-329 / 第 306-329 行
````python
    def all_streams_wait_for_event(self, event: EventId) -> None:
        self._ensure_event_exists(event)
        for stream in self.current_sync_states:
            self.stream_wait_for_event(stream, event)

        self._state_wait_for_other(
            self.host_sync_state, self.recorded_sync_states[event]
        )

    def all_streams_wait_for_stream(self, stream: StreamId) -> None:
        self._ensure_stream_exists(stream)
        for state in self.current_sync_states.values():
            self._state_wait_for_other(state, self.current_sync_states[stream])

        self._state_wait_for_other(
            self.host_sync_state, self.current_sync_states[stream]
        )

    def sync_all_streams(self) -> None:
        for stream, state in self.current_sync_states.items():
            self.host_sync_state[stream] = state[stream]

        for state in self.current_sync_states.values():
            self._state_wait_for_other(state, self.host_sync_state)
````
- **EN**: This chunk defines `sync_all_streams`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `sync_all_streams`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 331-350 / 第 331-350 行
````python
    def is_ordered_after(
        self, current_stream: StreamId, seq_num: SeqNum, other_stream: StreamId
    ) -> bool:
        self._ensure_stream_exists(current_stream)
        self._ensure_stream_exists(other_stream)
        return seq_num <= self.current_sync_states[current_stream].get(other_stream, -1)


class EventHandler:
    """Analyzes CSAN trace for synchronization errors.

    Stores information on each stream's synchronizations with other streams as well
    as tensor accesses to determine whether a given kernel launch might cause a
    data race.
    """

    def __init__(self) -> None:
        self.tensors_accessed = _TensorsAccessed()
        self.syncs = StreamSynchronizations()
        self.seq_num: SeqNum = 0
````
- **EN**: It introduces or extends `EventHandler`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `EventHandler`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 352-376 / 第 352-376 行
````python
    def _handle_kernel_launch(
        self,
        stream: StreamId,
        read_only: set[DataPtr],
        read_write: set[DataPtr],
        outputs: set[DataPtr],
        operator: str,
        tensor_aliases: dict[int, list[str]],
    ) -> list[SynchronizationError]:
        def check_conflict(
            data_ptr: DataPtr, current_access: Access, previous_access: Access | None
        ) -> None:
            if previous_access is None:
                return
            if not self.syncs.is_ordered_after(
                current_access.stream, previous_access.seq_num, previous_access.stream
            ):
                error_list.append(
                    UnsynchronizedAccessError(
                        data_ptr,
                        self.tensors_accessed.get_allocation_stack_trace(data_ptr),
                        current_access,
                        previous_access,
                    )
                )
````
- **EN**: This chunk defines `check_conflict`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `check_conflict`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 378-402 / 第 378-402 行
````python
        error_list: list[SynchronizationError] = []
        self.seq_num += 1
        self.syncs.update_seq_num(stream, self.seq_num)
        stack_trace = traceback.StackSummary.extract(
            traceback.walk_stack(inspect.currentframe()), lookup_lines=False
        )
        # The stack trace generated in this way is in the inverse order, so it must be
        # reversed.
        stack_trace.reverse()

        for data_ptr in read_only:
            self.tensors_accessed.ensure_tensor_exists(data_ptr)
            current_access = Access(
                AccessType.READ,
                self.seq_num,
                stream,
                operator,
                tensor_aliases[data_ptr],
                data_ptr in outputs,
                stack_trace,
            )
            check_conflict(
                data_ptr, current_access, self.tensors_accessed.get_write(data_ptr)
            )
            self.tensors_accessed.add_read(data_ptr, current_access)
````
- **EN**: This chunk continues `check_conflict` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `check_conflict`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 404-430 / 第 404-430 行
````python
        for data_ptr in read_write:
            self.tensors_accessed.ensure_tensor_exists(data_ptr)
            current_access = Access(
                AccessType.WRITE,
                self.seq_num,
                stream,
                operator,
                tensor_aliases[data_ptr],
                data_ptr in outputs,
                stack_trace,
            )
            if self.tensors_accessed.were_there_reads_since_last_write(data_ptr):
                for previous_access in self.tensors_accessed.get_reads(data_ptr):
                    check_conflict(data_ptr, current_access, previous_access)
            else:
                check_conflict(
                    data_ptr, current_access, self.tensors_accessed.get_write(data_ptr)
                )
            self.tensors_accessed.set_write(data_ptr, current_access)

        return error_list

    def _handle_event_creation(self, event: EventId) -> None:
        self.syncs.create_event(event)

    def _handle_event_deletion(self, event: EventId) -> None:
        self.syncs.delete_event(event)
````
- **EN**: This chunk defines `_handle_event_deletion`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `_handle_event_deletion`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 432-456 / 第 432-456 行
````python
    def _handle_event_record(self, event: EventId, stream: StreamId) -> None:
        self.syncs.record_state(event, stream)

    def _handle_event_wait(self, event: EventId, stream: StreamId) -> None:
        self.syncs.stream_wait_for_event(stream, event)

    def _handle_memory_allocation(self, data_ptr: DataPtr) -> None:
        self.tensors_accessed.ensure_tensor_does_not_exist(data_ptr)
        stack_trace = traceback.StackSummary.extract(
            traceback.walk_stack(inspect.currentframe()), lookup_lines=False
        )
        # The stack trace generated in this way is in the inverse order, so it must be
        # reversed.
        stack_trace.reverse()
        self.tensors_accessed.create_tensor(
            data_ptr,
            stack_trace,
        )

    def _handle_memory_deallocation(self, data_ptr: DataPtr) -> None:
        self.tensors_accessed.ensure_tensor_exists(data_ptr)
        self.tensors_accessed.delete_tensor(data_ptr)

    def _handle_stream_creation(self, stream: StreamId) -> None:
        self.syncs.create_stream(stream)
````
- **EN**: This chunk defines `_handle_stream_creation`, which implements a focused helper used by the surrounding module.
- **CN**: 这一段定义了 `_handle_stream_creation`，其作用是实现周边模块使用的关键辅助逻辑。

### Lines 458-483 / 第 458-483 行
````python
    def _handle_device_synchronization(self) -> None:
        self.syncs.sync_all_streams()

    def _handle_stream_synchronization(self, stream: StreamId) -> None:
        self.syncs.all_streams_wait_for_stream(stream)

    def _handle_event_synchronization(self, event: EventId) -> None:
        self.syncs.all_streams_wait_for_event(event)


def zip_by_key(a: dict[TK, TVa], b: dict[TK, TVb]) -> Iterator[tuple[TK, TVa, TVb]]:
    for arg, value in a.items():
        if arg in b:
            yield arg, value, b[arg]


def zip_arguments(
    schema: torch.FunctionSchema, args: tuple[Any, ...], kwargs: dict[str, Any]
) -> Iterator[tuple[torch.Argument, Any]]:
    schema_args = schema.arguments[: len(args)]
    schema_kwargs = {arg.name: arg for arg in schema.arguments[len(args) :]}

    yield from zip(schema_args, args)

    for _, argument, value in zip_by_key(schema_kwargs, kwargs):
        yield (argument, value)
````
- **EN**: This chunk defines `zip_arguments`, which implements a focused helper used by the surrounding module. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `zip_arguments`，其作用是实现周边模块使用的关键辅助逻辑。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 486-508 / 第 486-508 行
````python
class ArgumentHandler:
    def __init__(self) -> None:
        self.dataptrs_read: set[DataPtr] = set()
        self.dataptrs_written: set[DataPtr] = set()
        self.tensor_aliases: dict[DataPtr, list[str]] = {}
        self.outputs: set[DataPtr] = set()

    def _handle_argument(
        self,
        value: Any,
        is_write: bool,
        metadata_only: bool,
        name: str | None = None,
        is_output: bool = False,
    ) -> None:
        if isinstance(value, torch.Tensor) and value.is_cuda:
            # data_ptr() is preferred, but distinguish Tensors with null data_ptr()
            # otherwise two empty Tensors could incorrectly match as a conflict
            data_ptr = value.data_ptr() if value.data_ptr() else id(value)
            if is_write:
                self.dataptrs_written.add(data_ptr)
            elif not metadata_only:
                self.dataptrs_read.add(data_ptr)
````
- **EN**: It introduces or extends `ArgumentHandler`, which hold the main object-oriented state for this portion of the file. This chunk defines `_handle_argument`, which implements a focused helper used by the surrounding module. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 它引入或扩展了 `ArgumentHandler`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `_handle_argument`，其作用是实现周边模块使用的关键辅助逻辑。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 510-537 / 第 510-537 行
````python
            self.tensor_aliases.setdefault(data_ptr, [])
            if name is not None:
                self.tensor_aliases[data_ptr].append(name)
            if is_output:
                self.outputs.add(data_ptr)

    def parse_inputs(
        self,
        schema: torch.FunctionSchema,
        args: tuple[Any, ...],
        kwargs: dict[str, Any],
        *,
        is_factory: bool,
    ) -> None:
        for argument, value in zip_arguments(schema, args, kwargs):
            is_write = argument.alias_info is not None and argument.alias_info.is_write
            # A change is metadata only if it is a view or a factory function that
            # reads only metadata
            metadata_only = is_factory or (
                argument.alias_info is not None and not argument.alias_info.is_write
            )
            pytree.tree_map_(
                functools.partial(
                    self._handle_argument,
                    is_write=is_write,
                    name=argument.name,
                    metadata_only=metadata_only,
                ),
````
- **EN**: This chunk defines `parse_inputs`, which parses structured input into internal objects or validated metadata. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `parse_inputs`，其作用是把结构化输入解析为内部对象或经过验证的元数据。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 538-556 / 第 538-556 行
````python
                value,
            )

    def parse_outputs(
        self, schema: torch.FunctionSchema, outputs: Any, *, is_factory: bool
    ) -> None:
        for res, value in zip(schema.returns, (outputs,)):
            metadata_only = is_factory or (
                res.alias_info is not None and not res.alias_info.is_write
            )
            pytree.tree_map_(
                functools.partial(
                    self._handle_argument,
                    is_write=not metadata_only,
                    is_output=True,
                    metadata_only=metadata_only,
                ),
                value,
            )
````
- **EN**: This chunk defines `parse_outputs`, which parses structured input into internal objects or validated metadata. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `parse_outputs`，其作用是把结构化输入解析为内部对象或经过验证的元数据。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 559-586 / 第 559-586 行
````python
class CUDASanitizerDispatchMode(TorchDispatchMode):
    def __init__(self) -> None:
        self.event_handler = EventHandler()
        torch._C._activate_gpu_trace()
        gpu_trace.register_callback_for_event_creation(
            self.event_handler._handle_event_creation
        )
        gpu_trace.register_callback_for_event_deletion(
            self.event_handler._handle_event_deletion
        )
        gpu_trace.register_callback_for_event_record(
            self.event_handler._handle_event_record
        )
        gpu_trace.register_callback_for_event_wait(
            self.event_handler._handle_event_wait
        )
        gpu_trace.register_callback_for_memory_allocation(
            self.event_handler._handle_memory_allocation
        )
        gpu_trace.register_callback_for_memory_deallocation(
            self.event_handler._handle_memory_deallocation
        )
        gpu_trace.register_callback_for_stream_creation(
            self.event_handler._handle_stream_creation
        )
        gpu_trace.register_callback_for_device_synchronization(
            self.event_handler._handle_device_synchronization
        )
````
- **EN**: It introduces or extends `CUDASanitizerDispatchMode`, which hold the main object-oriented state for this portion of the file. This chunk defines `__init__`, which initializes object state and binds construction-time configuration.
- **CN**: 它引入或扩展了 `CUDASanitizerDispatchMode`，这些类承载了本段涉及的主要面向对象状态。 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。

### Lines 587-607 / 第 587-607 行
````python
        gpu_trace.register_callback_for_stream_synchronization(
            self.event_handler._handle_stream_synchronization
        )
        gpu_trace.register_callback_for_event_synchronization(
            self.event_handler._handle_event_synchronization
        )

    def __torch_dispatch__(self, func, types, args=(), kwargs=None):
        if kwargs is None:
            kwargs = {}

        # record_stream is not a kernel dispatch, skip it
        if func is aten.record_stream.default:
            return func(*args, **kwargs)

        is_factory = bool(FACTORY_FUNCTION_REGEX.match(func._schema.name))

        argument_handler = ArgumentHandler()
        argument_handler.parse_inputs(func._schema, args, kwargs, is_factory=is_factory)

        outputs = func(*args, **kwargs)
````
- **EN**: This chunk defines `__torch_dispatch__`, which implements a focused helper used by the surrounding module. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `__torch_dispatch__`，其作用是实现周边模块使用的关键辅助逻辑。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 609-633 / 第 609-633 行
````python
        argument_handler.parse_outputs(func._schema, outputs, is_factory=is_factory)
        errors = self.event_handler._handle_kernel_launch(
            torch.cuda.current_stream().cuda_stream,
            argument_handler.dataptrs_read - argument_handler.dataptrs_written,
            argument_handler.dataptrs_written,
            argument_handler.outputs,
            func._schema,
            argument_handler.tensor_aliases,
        )
        if errors:
            for error in errors:
                print(error, file=sys.stderr)
            raise CUDASanitizerErrors(errors)

        return outputs


class CUDASanitizer:
    """Manages the lifetime of a CUDASanitizer dispatch mode object.

    The CUDASanitizer class wraps the entering/exiting functions of the dispatch mode
    context manager in the enable function/destructor, respectively. This is to
    explicitly set the lifetime of the dispatch mode object to that of the application.
    This approach was deemed more elegant than using the atexit module.
    """
````
- **EN**: It introduces or extends `CUDASanitizer`, which hold the main object-oriented state for this portion of the file. This chunk continues `CUDASanitizer` and expands its internal control flow or state updates. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 它引入或扩展了 `CUDASanitizer`，这些类承载了本段涉及的主要面向对象状态。 这一段延续了 `CUDASanitizer`，进一步展开其内部控制流或状态更新。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 635-660 / 第 635-660 行
````python
    def __init__(self) -> None:
        self.dispatch = CUDASanitizerDispatchMode()
        self.enabled = False

    def enable(self):
        self.dispatch.__enter__()
        self.enabled = True

    def disable(self):
        self.dispatch.__exit__(None, None, None)
        self.enabled = False

    def __del__(self):
        # Since this object lifetime is linked to the `torch.cuda._sanitizer` python
        # module, it often gets deleted as part of the overall `torch` module cleanup
        # At that time, depending on CPython version, the torch.* module might be in
        # different states of being already cleaned up.
        # Similarly other imports might already have been cleaned up so `sys` might
        # be already gone as well.
        # Skip exiting the mode if it outlived the runtime.
        if (sys is not None) and (not sys.is_finalizing()) and self.enabled:
            self.disable()


def enable_cuda_sanitizer():
    """Enable CUDA Sanitizer.
````
- **EN**: This chunk defines `enable_cuda_sanitizer`, which mutates configuration or backend state that affects later execution. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段定义了 `enable_cuda_sanitizer`，其作用是修改会影响后续执行的配置或后端状态。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 662-670 / 第 662-670 行
````python
    The sanitizer will begin to analyze low-level CUDA calls invoked by torch functions
    for synchronization errors. All data races found will be printed to the standard
    error output along with stack traces of suspected causes. For best results, the
    sanitizer should be enabled at the very beginning of the program.
    """
    cuda_sanitizer.enable()


cuda_sanitizer = CUDASanitizer()
````
- **EN**: This chunk continues `enable_cuda_sanitizer` and expands its internal control flow or state updates. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段延续了 `enable_cuda_sanitizer`，进一步展开其内部控制流或状态更新。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

## Key Concepts / 关键概念

- **CUDA runtime**
  - EN: Wraps streams, devices, memory state, and CUDA-specific coordination points.
  - CN: 封装流、设备、内存状态以及 CUDA 专用协同点。
- **DEFAULT_STREAM_ID**
  - EN: `DEFAULT_STREAM_ID` is one of the main symbols declared or implemented in this file.
  - CN: `DEFAULT_STREAM_ID` 是本文件声明或实现的主要符号之一。
- **TK**
  - EN: `TK` is one of the main symbols declared or implemented in this file.
  - CN: `TK` 是本文件声明或实现的主要符号之一。
- **Dispatch logic**
  - EN: The code routes Python-visible APIs to backend-specific kernels or registrations.
  - CN: 代码把 Python 可见 API 路由到后端特定的内核或注册项。
- **Stream coordination**
  - EN: The code exposes stream-aware state so asynchronous execution can be controlled from Python.
  - CN: 代码暴露与流相关的状态，使异步执行可从 Python 侧进行控制。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch.cuda._gpu_trace`, `torch.utils`, `torch.utils._python_dispatch`
- **Standard library / 标准库**: `enum`, `functools`, `inspect`, `io`, `logging`, `re`, `sys`, `textwrap`, `traceback`, `collections.abc`, `dataclasses`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `DEFAULT_STREAM_ID`, `TK`, `FACTORY_FUNCTION_REGEX`, `AccessType`, `Access`, `SynchronizationError`, `UnsynchronizedAccessError`, `CUDASanitizerErrors`, `TensorInfo`, `_TensorsAccessed`
