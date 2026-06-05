# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tracing/__init__.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Initializes the `tracing` package and exposes package-level entry points. It also re-exports `init_tracer`, `maybe_init_worker_tracer`. / 初始化 `tracing` 包，并暴露包级入口。 同时重新导出 `init_tracer`, `maybe_init_worker_tracer`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-63)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import functools
from collections.abc import Callable
from typing import Any, TypeAlias

# Import the implementation details
from .otel import (
    SpanKind,
    extract_trace_context,
    init_otel_tracer,
    init_otel_worker_tracer,
    instrument_otel,
    is_otel_available,
    manual_instrument_otel,
    otel_import_error_traceback,
)
from .utils import (
    SpanAttributes,
    contains_trace_headers,
    extract_trace_headers,
    log_tracing_disabled_warning,
)
    # ...
        init_otel_tracer,
        init_otel_worker_tracer,
        instrument_otel,
        manual_instrument_otel,
    ),
}
```
**EN:** Sets up the module with standard-library support such as `functools`, `collections.abc`, `typing`, vLLM modules such as `.otel`, `.utils`. It prepares the symbols later used by `init_tracer`, `maybe_init_worker_tracer`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `.otel`, `.utils` 等 vLLM 内部依赖。 这些准备工作为后续的 `init_tracer`, `maybe_init_worker_tracer` 提供上下文。

### init_tracer (lines 66-75)
```python
def init_tracer(
    instrumenting_module_name: str,
    otlp_traces_endpoint: str,
    extra_attributes: dict[str, str] | None = None,
):
    is_available, init_tracer_fn, _, _, _ = _REGISTERED_TRACING_BACKENDS["otel"]
    if is_available():
        return init_tracer_fn(
            instrumenting_module_name, otlp_traces_endpoint, extra_attributes
        )
```
**EN:** `init_tracer` initializes state required by the module. It mainly works with `instrumenting_module_name`, `otlp_traces_endpoint`, `extra_attributes`. Inside the body, it relies on `is_available`, `init_tracer_fn` to complete the main steps.
**CN:** `init_tracer` 负责初始化模块所需的状态。 它主要处理 `instrumenting_module_name`, `otlp_traces_endpoint`, `extra_attributes` 等参数。 实现过程中会调用 `is_available`, `init_tracer_fn` 等函数完成关键步骤。

### maybe_init_worker_tracer (lines 78-87)
```python
def maybe_init_worker_tracer(
    instrumenting_module_name: str,
    process_kind: str,
    process_name: str,
):
    is_available, _, init_worker_tracer_fn, _, _ = _REGISTERED_TRACING_BACKENDS["otel"]
    if is_available():
        return init_worker_tracer_fn(
            instrumenting_module_name, process_kind, process_name
        )
```
**EN:** `maybe_init_worker_tracer` implements helper logic used by this module. It mainly works with `instrumenting_module_name`, `process_kind`, `process_name`. Inside the body, it relies on `is_available`, `init_worker_tracer_fn` to complete the main steps.
**CN:** `maybe_init_worker_tracer` 负责实现本模块使用的辅助逻辑。 它主要处理 `instrumenting_module_name`, `process_kind`, `process_name` 等参数。 实现过程中会调用 `is_available`, `init_worker_tracer_fn` 等函数完成关键步骤。

### instrument (lines 90-118)
```python
def instrument(
    obj: Callable | None = None,
    *,
    span_name: str = "",
    attributes: dict[str, str] | None = None,
    record_exception: bool = True,
):
    """
    Generic decorator to instrument functions.
    """
    if obj is None:
        return functools.partial(
            instrument,
            span_name=span_name,
            attributes=attributes,
            record_exception=record_exception,
        )

    # Dispatch to OTel (and potentially others later)
    is_available, _, _, otel_instrument, _ = _REGISTERED_TRACING_BACKENDS["otel"]
    if is_available():
        return otel_instrument(
            func=obj,
            span_name=span_name,
            attributes=attributes,
            record_exception=record_exception,
        )
    else:
        return obj
```
**EN:** `instrument`: Generic decorator to instrument functions. It mainly works with `obj`, `span_name`, `attributes`, `record_exception`. Inside the body, it relies on `is_available`, `functools.partial`, `otel_instrument` to complete the main steps.
**CN:** `instrument` 负责实现本模块使用的辅助逻辑。 它主要处理 `obj`, `span_name`, `attributes`, `record_exception` 等参数。 实现过程中会调用 `is_available`, `functools.partial`, `otel_instrument` 等函数完成关键步骤。

### instrument_manual (lines 121-145)
```python
def instrument_manual(
    span_name: str,
    start_time: int,
    end_time: int | None = None,
    attributes: dict[str, Any] | None = None,
    context: Any = None,
    kind: Any = None,
):
    """Manually create a span with explicit timestamps.

    Args:
        span_name: Name of the span to create.
        start_time: Start time in nanoseconds since epoch.
        end_time: Optional end time in nanoseconds. If None, ends immediately.
        attributes: Optional dict of span attributes.
        context: Optional trace context (e.g., from extract_trace_context).
        kind: Optional SpanKind (e.g., SpanKind.SERVER).
    """
    is_available, _, _, _, manual_instrument_fn = _REGISTERED_TRACING_BACKENDS["otel"]
    if is_available():
        return manual_instrument_fn(
            span_name, start_time, end_time, attributes, context, kind
        )
    else:
        return None
```
**EN:** `instrument_manual`: Manually create a span with explicit timestamps. It mainly works with `span_name`, `start_time`, `end_time`, `attributes`. Inside the body, it relies on `is_available`, `manual_instrument_fn` to complete the main steps.
**CN:** `instrument_manual` 负责实现本模块使用的辅助逻辑。 它主要处理 `span_name`, `start_time`, `end_time`, `attributes` 等参数。 实现过程中会调用 `is_available`, `manual_instrument_fn` 等函数完成关键步骤。

### is_tracing_available (lines 148-157)
```python
def is_tracing_available() -> bool:
    """
    Returns True if any tracing backend (OTel, Profiler, etc.) is available.
    Use this to guard expensive tracing logic in the main code.
    """
    check_available = [
        is_available
        for is_available, _, _, _, _ in _REGISTERED_TRACING_BACKENDS.values()
    ]
    return any(check_available)
```
**EN:** `is_tracing_available`: Returns True if any tracing backend (OTel, Profiler, etc.) is available. Inside the body, it relies on `any`, `_REGISTERED_TRACING_BACKENDS.values` to complete the main steps.
**CN:** `is_tracing_available` 负责检查条件并返回布尔结果。 实现过程中会调用 `any`, `_REGISTERED_TRACING_BACKENDS.values` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`init_tracer`**: Key helper or entry point in this file. / **`init_tracer`**：本文件中的关键辅助函数或入口。
- **`maybe_init_worker_tracer`**: Key helper or entry point in this file. / **`maybe_init_worker_tracer`**：本文件中的关键辅助函数或入口。
- **`instrument`**: Key helper or entry point in this file. / **`instrument`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: functools, collections.abc, typing
- **Third-party / 第三方**: None / 无
- **Internal vLLM / vLLM 内部依赖**: .otel, .utils
