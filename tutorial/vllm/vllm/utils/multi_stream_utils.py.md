# multi_stream_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/utils/multi_stream_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides reusable utility helpers shared across vLLM components. / 提供 vLLM 各组件复用的通用工具函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-8)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from collections.abc import Callable
from enum import Enum
from typing import Any

import torch
```
**EN:** Sets up the module with standard-library support such as `collections.abc`, `enum`, `typing`, external packages such as `torch`. It prepares the symbols later used by `AuxStreamType`, `EventType`, `maybe_execute_in_parallel`, `execute_in_parallel`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义。 这些准备工作为后续的 `AuxStreamType`, `EventType`, `maybe_execute_in_parallel`, `execute_in_parallel` 提供上下文。

### AuxStreamType (lines 11-12)
```python
class AuxStreamType(Enum):
    Attention = 1
```
**EN:** Defines the `AuxStreamType` class used by this module. It extends `Enum`.
**CN:** `AuxStreamType` 是该文件中的核心类，用于封装与 `AuxStreamType` 相关的状态和行为。 它继承自 `Enum`。

### EventType (lines 15-17)
```python
class EventType(Enum):
    Main = 0
    Attention = 1
```
**EN:** Defines the `EventType` class used by this module. It extends `Enum`.
**CN:** `EventType` 是该文件中的核心类，用于封装与 `EventType` 相关的状态和行为。 它继承自 `Enum`。

### maybe_execute_in_parallel (lines 20-58)
```python
def maybe_execute_in_parallel(
    fn0: Callable[[], Any],
    fn1: Callable[[], Any],
    event0: torch.cuda.Event,
    event1: torch.cuda.Event,
    aux_stream: torch.cuda.Stream | None = None,
) -> tuple[Any, Any]:
    """Run two functions potentially in parallel on separate CUDA streams.

    When aux_stream is provided, fn0 runs on the current (default) stream and
    fn1 runs on aux_stream, synchronized via CUDA events.  When aux_stream is
    None, both functions execute sequentially on the current stream.

    This design follows TensorRT-LLM's maybe_execute_in_parallel pattern
    (tensorrt_llm/_torch/modules/multi_stream_utils.py).

    Args:
        fn0: Callable for the default stream.
        fn1: Callable for the auxiliary stream.
        event0: CUDA event recorded before fn0 so aux_stream can wait.
        event1: CUDA event recorded after fn1 so default stream can wait.
        aux_stream: The second CUDA stream for fn1.
            Multi-stream is disabled when aux_stream is None.

    # ...
            event1.record()
        event1.wait()
    else:
        result0 = fn0()
        result1 = fn1()
    return (result0, result1)
```
**EN:** `maybe_execute_in_parallel`: Run two functions potentially in parallel on separate CUDA streams. It mainly works with `fn0`, `fn1`, `event0`, `event1`. Inside the body, it relies on `event0.record`, `fn0`, `event1.wait` to complete the main steps.
**CN:** `maybe_execute_in_parallel` 负责实现本模块使用的辅助逻辑。 它主要处理 `fn0`, `fn1`, `event0`, `event1` 等参数。 实现过程中会调用 `event0.record`, `fn0`, `event1.wait` 等函数完成关键步骤。

### execute_in_parallel (lines 61-128)
```python
def execute_in_parallel(
    default_fn: Callable[[], Any],
    aux_fns: list[Callable[[], Any] | None],
    start_event: torch.cuda.Event,
    done_events: list[torch.cuda.Event],
    aux_streams: list[torch.cuda.Stream] | None = None,
    enable: bool = False,
) -> tuple[Any, list[Any]]:
    """Run default_fn on the current stream and aux_fns concurrently on
    aux_streams.

    Generalizes maybe_execute_in_parallel to N aux callables. Slots where
    aux_fns[i] is None are skipped (no stream switch, no event record); their
    corresponding entry in the returned aux_results list is None.

    start_event fans out from the current stream to every launched aux stream;
    done_events[i] is recorded after aux_fns[i] so the current stream joins
    before returning. Falls back to sequential execution on the current stream
    when aux_streams is None or enable is False; in that case default_fn runs
    first, then aux_fns in order.

    Args:
        default_fn: Callable for the default (current) stream.
        aux_fns: Per-aux callables; entries may be None to skip.
    # ...
    default_result = default_fn()

    for ev in pending:
        ev.wait()

    return default_result, aux_results
```
**EN:** `execute_in_parallel`: Run default_fn on the current stream and aux_fns concurrently on aux_streams. It mainly works with `default_fn`, `aux_fns`, `start_event`, `done_events`. Inside the body, it relies on `start_event.record`, `default_fn`, `pending.append` to complete the main steps.
**CN:** `execute_in_parallel` 负责实现本模块使用的辅助逻辑。 它主要处理 `default_fn`, `aux_fns`, `start_event`, `done_events` 等参数。 实现过程中会调用 `start_event.record`, `default_fn`, `pending.append` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`AuxStreamType`**: Core class that organizes module behavior. / **`AuxStreamType`**：组织模块行为的核心类。
- **`EventType`**: Core class that organizes module behavior. / **`EventType`**：组织模块行为的核心类。
- **`maybe_execute_in_parallel`**: Key helper or entry point in this file. / **`maybe_execute_in_parallel`**：本文件中的关键辅助函数或入口。
- **`execute_in_parallel`**: Key helper or entry point in this file. / **`execute_in_parallel`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: collections.abc, enum, typing
- **Third-party / 第三方**: torch
- **Internal vLLM / vLLM 内部依赖**: None / 无
