# otel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/tracing/otel.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `otel`-related logic centered around `is_otel_available`, `init_otel_tracer`. / 实现与 `otel` 相关的逻辑，核心符号包括 `is_otel_available`, `init_otel_tracer`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-53)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import atexit
import functools
import inspect
import os
import traceback
from collections.abc import Mapping
from contextlib import contextmanager
from typing import Any

from vllm.logger import init_logger
from vllm.tracing.utils import TRACE_HEADERS, LoadingSpanAttributes

logger = init_logger(__name__)

try:
    from opentelemetry import trace
    from opentelemetry.context.context import Context
    from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import (
        OTLPSpanExporter as OTLPGrpcExporter,
    )
    from opentelemetry.exporter.otlp.proto.http.trace_exporter import (
    # ...
    trace = None  # type: ignore
    Context = Any  # type: ignore
    Tracer = Any  # type: ignore
    inject = None  # type: ignore
    Resource = None  # type: ignore
    SpanKind = Any  # type: ignore
```
**EN:** Sets up the module with standard-library support such as `atexit`, `functools`, `inspect`, external packages such as `opentelemetry`, `opentelemetry.context.context`, `opentelemetry.exporter.otlp.proto.grpc.trace_exporter`, vLLM modules such as `vllm.logger`, `vllm.tracing.utils`. It prepares the symbols later used by `is_otel_available`, `init_otel_tracer`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.logger`, `vllm.tracing.utils` 等 vLLM 内部依赖。 这些准备工作为后续的 `is_otel_available`, `init_otel_tracer` 提供上下文。

### is_otel_available (lines 56-57)
```python
def is_otel_available() -> bool:
    return _IS_OTEL_AVAILABLE
```
**EN:** `is_otel_available` checks a condition and returns a boolean-style result.
**CN:** `is_otel_available` 负责检查条件并返回布尔结果。

### init_otel_tracer (lines 60-91)
```python
def init_otel_tracer(
    instrumenting_module_name: str,
    otlp_traces_endpoint: str,
    extra_attributes: dict[str, str] | None = None,
) -> Tracer:
    """Initializes the OpenTelemetry tracer provider."""
    if not _IS_OTEL_AVAILABLE:
        raise ValueError(
            "OpenTelemetry is not available. Unable to initialize "
            "a tracer. Ensure OpenTelemetry packages are installed. "
            f"Original error:\n{otel_import_error_traceback}"
        )

    # Store the endpoint in environment so child processes can inherit it
    os.environ["OTEL_EXPORTER_OTLP_TRACES_ENDPOINT"] = otlp_traces_endpoint

    resource_attrs = {}
    resource_attrs["vllm.instrumenting_module_name"] = instrumenting_module_name
    resource_attrs["vllm.process_id"] = str(os.getpid())
    if extra_attributes:
        resource_attrs.update(extra_attributes)
    resource = Resource.create(resource_attrs)

    trace_provider = TracerProvider(resource=resource)
    span_exporter = get_span_exporter(otlp_traces_endpoint)
    trace_provider.add_span_processor(BatchSpanProcessor(span_exporter))
    set_tracer_provider(trace_provider)

    atexit.register(trace_provider.shutdown)

    tracer = trace_provider.get_tracer(instrumenting_module_name)
    return tracer
```
**EN:** `init_otel_tracer`: Initializes the OpenTelemetry tracer provider. It mainly works with `instrumenting_module_name`, `otlp_traces_endpoint`, `extra_attributes`. Inside the body, it relies on `Resource.create`, `TracerProvider`, `get_span_exporter` to complete the main steps.
**CN:** `init_otel_tracer` 负责初始化模块所需的状态。 它主要处理 `instrumenting_module_name`, `otlp_traces_endpoint`, `extra_attributes` 等参数。 实现过程中会调用 `Resource.create`, `TracerProvider`, `get_span_exporter` 等函数完成关键步骤。

### get_span_exporter (lines 94-102)
```python
def get_span_exporter(endpoint):
    protocol = os.environ.get(OTEL_EXPORTER_OTLP_TRACES_PROTOCOL, "grpc")
    if protocol == "grpc":
        exporter = OTLPGrpcExporter(endpoint=endpoint, insecure=True)
    elif protocol == "http/protobuf":
        exporter = OTLPHttpExporter(endpoint=endpoint)
    else:
        raise ValueError(f"Unsupported OTLP protocol '{protocol}' is configured")
    return exporter
```
**EN:** `get_span_exporter` retrieves data or state needed by the pipeline. It mainly works with `endpoint`. Inside the body, it relies on `os.environ.get`, `OTLPGrpcExporter`, `OTLPHttpExporter` to complete the main steps.
**CN:** `get_span_exporter` 负责获取流水线所需的数据或状态。 它主要处理 `endpoint` 等参数。 实现过程中会调用 `os.environ.get`, `OTLPGrpcExporter`, `OTLPHttpExporter` 等函数完成关键步骤。

### init_otel_worker_tracer (lines 105-124)
```python
def init_otel_worker_tracer(
    instrumenting_module_name: str,
    process_kind: str,
    process_name: str,
) -> Tracer:
    """
    Backend-specific initialization for OpenTelemetry in a worker process.
    """
    # Initialize the tracer if an OTLP endpoint is configured.
    # The endpoint is propagated via environment variable from the main process.
    otlp_endpoint = os.environ.get("OTEL_EXPORTER_OTLP_TRACES_ENDPOINT")
    if not otlp_endpoint:
        return None

    extra_attrs = {
        "vllm.process_kind": process_kind,
        "vllm.process_name": process_name,
    }

    return init_otel_tracer(instrumenting_module_name, otlp_endpoint, extra_attrs)
```
**EN:** `init_otel_worker_tracer`: Backend-specific initialization for OpenTelemetry in a worker process. It mainly works with `instrumenting_module_name`, `process_kind`, `process_name`. Inside the body, it relies on `os.environ.get`, `init_otel_tracer` to complete the main steps.
**CN:** `init_otel_worker_tracer` 负责初始化模块所需的状态。 它主要处理 `instrumenting_module_name`, `process_kind`, `process_name` 等参数。 实现过程中会调用 `os.environ.get`, `init_otel_tracer` 等函数完成关键步骤。

### extract_trace_context (lines 127-131)
```python
def extract_trace_context(headers: Mapping[str, str] | None) -> Context | None:
    """Extracts context from HTTP headers."""
    if _IS_OTEL_AVAILABLE and headers:
        return TraceContextTextMapPropagator().extract(headers)
    return None
```
**EN:** `extract_trace_context`: Extracts context from HTTP headers. It mainly works with `headers`. Inside the body, it relies on `TraceContextTextMapPropagator.extract`, `TraceContextTextMapPropagator` to complete the main steps.
**CN:** `extract_trace_context` 负责实现本模块使用的辅助逻辑。 它主要处理 `headers` 等参数。 实现过程中会调用 `TraceContextTextMapPropagator.extract`, `TraceContextTextMapPropagator` 等函数完成关键步骤。

### instrument_otel (lines 134-180)
```python
def instrument_otel(func, span_name, attributes, record_exception):
    """Internal wrapper logic for sync and async functions."""

    # Pre-calculate static code attributes once (these don't change)
    code_attrs = {
        LoadingSpanAttributes.CODE_FUNCTION: func.__qualname__,
        LoadingSpanAttributes.CODE_NAMESPACE: func.__module__,
        LoadingSpanAttributes.CODE_FILEPATH: func.__code__.co_filename,
        LoadingSpanAttributes.CODE_LINENO: str(func.__code__.co_firstlineno),
    }
    if attributes:
        code_attrs.update(attributes)

    final_span_name = span_name or func.__qualname__
    module_name = func.__module__

    @functools.wraps(func)
    async def async_wrapper(*args, **kwargs):
        tracer = trace.get_tracer(module_name)
        ctx = _get_smart_context()
        with (
            tracer.start_as_current_span(
                final_span_name,
                context=ctx,
    # ...
            ),
            propagate_trace_to_env(),
        ):
            return func(*args, **kwargs)

    return async_wrapper if inspect.iscoroutinefunction(func) else sync_wrapper
```
**EN:** `instrument_otel`: Internal wrapper logic for sync and async functions. It mainly works with `func`, `span_name`, `attributes`, `record_exception`. Inside the body, it relies on `functools.wraps`, `code_attrs.update`, `trace.get_tracer` to complete the main steps.
**CN:** `instrument_otel` 负责实现本模块使用的辅助逻辑。 它主要处理 `func`, `span_name`, `attributes`, `record_exception` 等参数。 实现过程中会调用 `functools.wraps`, `code_attrs.update`, `trace.get_tracer` 等函数完成关键步骤。

### manual_instrument_otel (lines 183-213)
```python
def manual_instrument_otel(
    span_name: str,
    start_time: int,
    end_time: int | None = None,
    attributes: dict[str, Any] | None = None,
    context: Context | None = None,
    kind: Any = None,  # SpanKind, but typed as Any for when OTEL unavailable
):
    """Manually create and end a span with explicit timestamps."""
    if not _IS_OTEL_AVAILABLE:
        return

    tracer = trace.get_tracer(__name__)
    # Use provided context, or fall back to smart context detection
    ctx = context if context is not None else _get_smart_context()

    span_kwargs: dict[str, Any] = {
        "name": span_name,
        "context": ctx,
        "start_time": start_time,
    }
    if kind is not None:
        span_kwargs["kind"] = kind

    span = tracer.start_span(**span_kwargs)
    if attributes:
        span.set_attributes(attributes)
    if end_time is not None:
        span.end(end_time=end_time)
    else:
        span.end()
```
**EN:** `manual_instrument_otel`: Manually create and end a span with explicit timestamps. It mainly works with `span_name`, `start_time`, `end_time`, `attributes`. Inside the body, it relies on `trace.get_tracer`, `tracer.start_span`, `_get_smart_context` to complete the main steps.
**CN:** `manual_instrument_otel` 负责实现本模块使用的辅助逻辑。 它主要处理 `span_name`, `start_time`, `end_time`, `attributes` 等参数。 实现过程中会调用 `trace.get_tracer`, `tracer.start_span`, `_get_smart_context` 等函数完成关键步骤。

### _get_smart_context (lines 216-237)
```python
def _get_smart_context() -> Context | None:
    """
    Determines the parent context.
    1. If a Span is already active in this process, use it.
    2. If not, extract from os.environ, handling the case-sensitivity mismatch.
    """
    current_span = trace.get_current_span()
    if current_span.get_span_context().is_valid:
        return None

    carrier = {}

    if tp := os.environ.get("traceparent", os.environ.get("TRACEPARENT")):  # noqa: SIM112
        carrier["traceparent"] = tp

    if ts := os.environ.get("tracestate", os.environ.get("TRACESTATE")):  # noqa: SIM112
        carrier["tracestate"] = ts

    if not carrier:
        carrier = dict(os.environ)

    return TraceContextTextMapPropagator().extract(carrier)
```
**EN:** `_get_smart_context`: Determines the parent context. Inside the body, it relies on `trace.get_current_span`, `TraceContextTextMapPropagator.extract`, `current_span.get_span_context` to complete the main steps.
**CN:** `_get_smart_context` 负责获取流水线所需的数据或状态。 实现过程中会调用 `trace.get_current_span`, `TraceContextTextMapPropagator.extract`, `current_span.get_span_context` 等函数完成关键步骤。

### propagate_trace_to_env (lines 241-265)
```python
def propagate_trace_to_env():
    """
    Temporarily injects the current OTel context into os.environ.
    This ensures that any subprocesses (like vLLM workers) spawned
    within this context inherit the correct traceparent.
    """
    if not _IS_OTEL_AVAILABLE:
        yield
        return

    # Capture original state of relevant keys
    original_state = {k: os.environ.get(k) for k in TRACE_HEADERS}

    try:
        # inject() writes 'traceparent' and 'tracestate' to os.environ
        inject(os.environ)
        yield

    finally:
        # Restore original environment
        for key, original_value in original_state.items():
            if original_value is None:
                os.environ.pop(key, None)
            else:
                os.environ[key] = original_value
```
**EN:** `propagate_trace_to_env`: Temporarily injects the current OTel context into os.environ. Inside the body, it relies on `os.environ.get`, `inject`, `original_state.items` to complete the main steps.
**CN:** `propagate_trace_to_env` 负责实现本模块使用的辅助逻辑。 实现过程中会调用 `os.environ.get`, `inject`, `original_state.items` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`is_otel_available`**: Key helper or entry point in this file. / **`is_otel_available`**：本文件中的关键辅助函数或入口。
- **`init_otel_tracer`**: Key helper or entry point in this file. / **`init_otel_tracer`**：本文件中的关键辅助函数或入口。
- **`get_span_exporter`**: Key helper or entry point in this file. / **`get_span_exporter`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: atexit, functools, inspect, os, traceback, collections.abc, contextlib, typing
- **Third-party / 第三方**: opentelemetry, opentelemetry.context.context, opentelemetry.exporter.otlp.proto.grpc.trace_exporter, opentelemetry.exporter.otlp.proto.http.trace_exporter, opentelemetry.propagate, opentelemetry.sdk.environment_variables, opentelemetry.sdk.resources, opentelemetry.sdk.trace, opentelemetry.sdk.trace.export, opentelemetry.trace, opentelemetry.trace.propagation.tracecontext
- **Internal vLLM / vLLM 内部依赖**: vllm.logger, vllm.tracing.utils
