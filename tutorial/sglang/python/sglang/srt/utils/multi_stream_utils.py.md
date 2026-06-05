# multi_stream_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/multi_stream_utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `multi_stream_utils` and the surrounding SGLang serving stack. / 提供围绕 `multi_stream_utils` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 3-7: Module setup and shared state / 模块设置与共享状态
```python
import threading
from contextlib import contextmanager
from typing import Any, Callable, Optional

import torch
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `threading`, `contextlib`, `typing`, `torch`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `threading`, `contextlib`, `typing`, `torch`。

### Lines 10-10: Class `do_multi_stream_local` declaration / 类 `do_multi_stream_local` 声明
```python
class do_multi_stream_local(threading.local):
```
**EN:** This class establishes `do_multi_stream_local` as the main container/coordinator for the surrounding logic. It inherits from `threading.local`. Its core interface includes methods such as `__init__`.
**CN:** 该类将 `do_multi_stream_local` 定义为周边逻辑的主要封装体或协调者。 它继承自 `threading.local`。 其核心接口包括 `__init__` 等方法。

### Lines 12-13: Method `do_multi_stream_local.__init__` / 方法 `do_multi_stream_local.__init__`
```python
    def __init__(self):
        self.do_multi_stream = False
```
**EN:** This method implements `__init__` on `do_multi_stream_local`. State updates are written into `self.do_multi_stream`.
**CN:** 该方法（属于 `do_multi_stream_local`）实现了 `__init__`。 状态更新主要写入 `self.do_multi_stream`。

### Lines 16-16: Constants and shared state / 常量与共享状态
```python
_local = do_multi_stream_local()
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_local`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_local`。

### Lines 19-20: Function `set_do_multi_stream` / 函数 `set_do_multi_stream`
```python
def set_do_multi_stream(enable: bool):
    _local.do_multi_stream = enable
```
**EN:** This function implements `set_do_multi_stream`. State updates are written into `_local.do_multi_stream`.
**CN:** 该函数实现了 `set_do_multi_stream`。 状态更新主要写入 `_local.do_multi_stream`。

### Lines 23-24: Function `do_multi_stream` / 函数 `do_multi_stream`
```python
def do_multi_stream() -> bool:
    return _local.do_multi_stream
```
**EN:** This function implements `do_multi_stream`.
**CN:** 该函数实现了 `do_multi_stream`。

### Lines 27-34: Function `with_multi_stream` / 函数 `with_multi_stream`
```python
@contextmanager
def with_multi_stream(enable: bool):
    prev_do_multi_stream = _local.do_multi_stream
    set_do_multi_stream(enable)
    try:
        yield
    finally:
        set_do_multi_stream(prev_do_multi_stream)
```
**EN:** This function implements `with_multi_stream`. It primarily calls `set_do_multi_stream` to complete its work. State updates are written into `prev_do_multi_stream`. The implementation relies on error handling.
**CN:** 该函数实现了 `with_multi_stream`。 它主要通过调用 `set_do_multi_stream` 来完成任务。 状态更新主要写入 `prev_do_multi_stream`。 实现中使用了错误处理。

### Lines 37-75: Function `maybe_execute_in_parallel` / 函数 `maybe_execute_in_parallel`
```python
def maybe_execute_in_parallel(
    fn0: Callable,
    fn1: Callable,
    events: list[torch.cuda.Event],
    aux_stream: Optional[torch.cuda.Stream] = None,
) -> tuple[Any, Any]:
    """Utility function to run two functions in two cuda streams in parallel. Multi-stream is
    only enabled when cuda graph is turned on because switch stream has extra host overhead.

    This design is mainly for low latency use case. It needs to be improved for max throughput
    use case.
    For simplicity, fn0 and fn1 do not support inputs.

    Args:
        fn0 (Callable): callable for the default stream
        fn1 (Callable): callable for the second stream, aux_stream
        events (list[torch.cuda.Event]): cuda events for callables
        aux_stream (Optional[torch.cuda.Stream]): the second cuda stream for fn1.
            Multi-stream is disabled when aux_stream is None.

    Returns:
        tuple[Any, Any]: the return values of fn0() and fn1()
    """

    multi_stream = do_multi_stream() and aux_stream is not None

    if multi_stream:
        events[0].record()
# ... omitted for brevity ...
            events[1].record()
        events[1].wait()
    else:
        result0 = fn0()
        result1 = fn1()
    return (result0, result1)
```
**EN:** This function implements `maybe_execute_in_parallel`. It primarily calls `do_multi_stream`, `events.record`, `fn0`, `events.wait`, `fn1`, `torch.cuda.stream` to complete its work. State updates are written into `multi_stream`, `result0`, `result1`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `maybe_execute_in_parallel`。 它主要通过调用 `do_multi_stream`, `events.record`, `fn0`, `events.wait`, `fn1`, `torch.cuda.stream` 来完成任务。 状态更新主要写入 `multi_stream`, `result0`, `result1`。 实现中使用了条件分支、上下文管理资源。

## Key Concepts / 关键概念
- **Classes / 类**: `do_multi_stream_local`
- **Functions / 函数**: `set_do_multi_stream`, `do_multi_stream`, `with_multi_stream`, `maybe_execute_in_parallel`, `__init__`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `torch`
- **Standard library / 标准库**: `threading`, `contextlib`, `typing`
