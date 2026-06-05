# trace.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/observability/trace.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the metrics, tracing, and monitoring part of the SRT runtime and implements logic centered on `trace`. The module docstring frames it as: "package for sglang requests tracing" / 该模块属于 SRT 运行时的指标、追踪与监控部分，主要实现围绕 `trace` 的逻辑。 它对外提供的主要入口包括 `extract_trace_headers`, `set_global_trace_level`, `TraceThreadInfo`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60: Module imports, constants, and setup (part 1/2)
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""package for sglang requests tracing"""

from __future__ import annotations

import logging
import os
import random
import threading
import time
import uuid
from dataclasses import dataclass
from typing import Any, Dict, List, Mapping, Optional

from sglang.srt.utils import get_int_env_var

logger = logging.getLogger(__name__)
opentelemetry_imported = False
opentelemetry_initialized = False
_trace_context_propagator = None
tracer: Optional[trace.Tracer] = None

global_trace_level = get_int_env_var("SGLANG_TRACE_LEVEL", 3)

TRACE_HEADERS = ["traceparent", "tracestate"]

try:
    from opentelemetry import context, propagate, trace
    from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import (
        OTLPSpanExporter as GRPCSpanExporter,
    )
    from opentelemetry.exporter.otlp.proto.http.trace_exporter import (
        OTLPSpanExporter as HTTPSpanExporter,
    )
    from opentelemetry.sdk.environment_variables import (
        OTEL_EXPORTER_OTLP_TRACES_PROTOCOL,
    )
    from opentelemetry.sdk.resources import SERVICE_NAME, Resource
    from opentelemetry.sdk.trace import TracerProvider, id_generator
    from opentelemetry.sdk.trace.export import BatchSpanProcessor
    from opentelemetry.trace import Status, StatusCode
    from opentelemetry.trace.propagation.tracecontext import (
        TraceContextTextMapPropagator,
    )

    _trace_context_propagator = TraceContextTextMapPropagator()

    opentelemetry_imported = True
```
**EN:** This range organizes module-level state and shared setup. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; emits logs for diagnostics; records metrics or tracing signals.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；记录指标或追踪信号。

### Lines 61-69: Module-level logic (part 2/2)
```python
except ImportError:

    class id_generator:
        class IdGenerator:
            pass

    logger.debug("opentelemetry package is not installed, tracing disabled")


```
**EN:** This range organizes module-level state and shared setup. This chunk is part 2 of 2 for the same logical block. In this range it emits logs for diagnostics.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 该片段是同一逻辑块的第 2/2 部分。 在这一范围内，它会输出日志以便诊断。

### Lines 70-71: Function extract_trace_headers
```python
def extract_trace_headers(headers: Mapping[str, str]) -> Optional[Dict]:
    return {h: headers[h] for h in TRACE_HEADERS if h in headers}
```
**EN:** This callable implements `extract_trace_headers`. It takes `headers` and mainly implements extract trace headers. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `extract_trace_headers`。它接收 `headers`，主要用于实现 extract trace headers 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 74-76: Function set_global_trace_level
```python
def set_global_trace_level(level: int):
    global global_trace_level
    global_trace_level = level
```
**EN:** This callable implements `set_global_trace_level`. It takes `level` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `set_global_trace_level`。它接收 `level`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 79-86: Class TraceThreadInfo
```python
@dataclass
class TraceThreadInfo:
    host_id: str
    pid: int
    thread_label: str
    tp_rank: int
    dp_rank: int
    pp_rank: int
```
**EN:** This range introduces `TraceThreadInfo` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `TraceThreadInfo`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 89-93: Class TraceEvent
```python
@dataclass
class TraceEvent:
    event_name: str
    ts: int
    attrs: Dict[str, Any]
```
**EN:** This range introduces `TraceEvent` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `TraceEvent`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 96-104: Class TraceSliceContext
```python
@dataclass
class TraceSliceContext:
    slice_name: str
    start_time_ns: int
    end_time_ns: Optional[int] = None
    span: Optional[trace.span.Span] = None
    level: int = 1
    attrs: Optional[Dict[str, Any]] = None
    events: Optional[List[TraceEvent]] = None
```
**EN:** This range introduces `TraceSliceContext` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `TraceSliceContext`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 107-111: Class TraceThreadContext
```python
@dataclass
class TraceThreadContext:
    thread_info: TraceThreadInfo
    cur_slice_stack: Optional[List[TraceSliceContext]] = None
    thread_span: Optional[trace.span.Span] = None
```
**EN:** This range introduces `TraceThreadContext` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `TraceThreadContext`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 114-119: Class TraceCustomIdGenerator
```python
class TraceCustomIdGenerator(id_generator.IdGenerator):
    """
    The default IdGenerator may produce duplicate trace IDs across multiple TP scheduler processes,
    hence a custom IdGenerator is implemented.
    """

```
**EN:** This range introduces `TraceCustomIdGenerator` and defines the structure or metadata that its methods rely on. Its docstring summarizes the class as: "The default IdGenerator may produce duplicate trace IDs across multiple TP scheduler processes, hence a custom IdGenerator is implemented." In this range it records metrics or tracing signals.
**CN:** 这一段引入 `TraceCustomIdGenerator`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 120-123: Method TraceCustomIdGenerator.__init__
```python
    def __init__(self):
        super().__init__()
        self.local_random = random.Random()
        self.local_random.seed(time.time())
```
**EN:** This callable implements `TraceCustomIdGenerator.__init__` and mainly initializes instance state and defaults.
**CN:** 这一可调用对象实现了 `TraceCustomIdGenerator.__init__`，主要用于初始化实例状态与默认值。

### Lines 125-126: Method TraceCustomIdGenerator.generate_trace_id
```python
    def generate_trace_id(self) -> int:
        return self.local_random.getrandbits(64)
```
**EN:** This callable implements `TraceCustomIdGenerator.generate_trace_id` and mainly implements generate trace id. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceCustomIdGenerator.generate_trace_id`，主要用于实现 generate trace id 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 128-129: Method TraceCustomIdGenerator.generate_span_id
```python
    def generate_span_id(self) -> int:
        return self.local_random.getrandbits(64)
```
**EN:** This callable implements `TraceCustomIdGenerator.generate_span_id` and mainly implements generate span id. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceCustomIdGenerator.generate_span_id`，主要用于实现 generate span id 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 130-139: Module-level constants and helpers
```python


# global variables
threads_info: Dict[int, TraceThreadInfo] = {}

get_cur_time_ns = lambda: int(time.time() * 1e9)
if hasattr(time, "time_ns"):
    get_cur_time_ns = lambda: int(time.time_ns())


```
**EN:** This range organizes module-level state and shared setup. In this range it records metrics or tracing signals.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 140-157: Function __get_host_id
```python
def __get_host_id() -> str:
    """
    In distributed tracing systems, obtain a unique node identifier
    and inject it into all subsequently generated spans
    to prevent PID conflicts between threads on different nodes.
    """
    if os.path.exists("/etc/machine-id"):
        try:
            with open("/etc/machine-id", "r") as f:
                return f.read().strip()
        except:
            pass

    mac = uuid.getnode()
    if mac != 0:
        return uuid.UUID(int=mac).hex

    return "unknown"
```
**EN:** This callable implements `__get_host_id` and mainly retrieves a value or derived view. The docstring states: "In distributed tracing systems, obtain a unique node identifier and inject it into all subsequently generated spans to prevent PID conflicts between threads on different nodes." In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `__get_host_id`，主要用于获取某个值或派生视图。 在这一范围内，它会记录指标或追踪信号。

### Lines 158-160: Module-level logic
```python


# Should be called by each tracked process.
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 161-202: Function process_tracing_init
```python
def process_tracing_init(otlp_endpoint, server_name):
    global opentelemetry_initialized
    global get_cur_time_ns
    global tracer
    if not opentelemetry_imported:
        opentelemetry_initialized = False
        raise RuntimeError(
            "opentelemetry package is not installed!!! Please not enable tracing or install opentelemetry"
        )

    try:
        resource = Resource.create(
            attributes={
                SERVICE_NAME: server_name,
            }
        )
        tracer_provider = TracerProvider(
            resource=resource, id_generator=TraceCustomIdGenerator()
        )

        schedule_delay_millis = get_int_env_var(
            "SGLANG_OTLP_EXPORTER_SCHEDULE_DELAY_MILLIS", 500
        )
        max_export_batch_size = get_int_env_var(
            "SGLANG_OTLP_EXPORTER_MAX_EXPORT_BATCH_SIZE", 64
        )

        processor = BatchSpanProcessor(
            span_exporter=get_otlp_span_exporter(otlp_endpoint),
            schedule_delay_millis=schedule_delay_millis,
            max_export_batch_size=max_export_batch_size,
        )
        tracer_provider.add_span_processor(processor)
        trace.set_tracer_provider(tracer_provider)
    except Exception as e:
        opentelemetry_initialized = False
        raise RuntimeError(
            f"initialize opentelemetry error:{e}. Please set correct otlp endpoint."
        )

    opentelemetry_initialized = True
    tracer = trace.get_tracer("sglang server")
```
**EN:** This callable implements `process_tracing_init`. It takes `otlp_endpoint`, `server_name` and mainly implements process tracing init. In this range it performs defensive checks on invalid state; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `process_tracing_init`。它接收 `otlp_endpoint`, `server_name`，主要用于实现 process tracing init 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；记录指标或追踪信号。

### Lines 205-206: Function get_global_tracing_enabled
```python
def get_global_tracing_enabled():
    return opentelemetry_initialized
```
**EN:** This callable implements `get_global_tracing_enabled` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `get_global_tracing_enabled`，主要用于获取某个值或派生视图。

### Lines 209-222: Function get_otlp_span_exporter
```python
def get_otlp_span_exporter(endpoint):
    protocol = os.environ.get(OTEL_EXPORTER_OTLP_TRACES_PROTOCOL, "grpc")
    supported_protocols = {"grpc", "http/protobuf"}

    if protocol not in supported_protocols:
        raise ValueError(
            f"Unsupported OTLP protocol '{protocol}' configured. "
            f"Supported protocols are: {', '.join(sorted(supported_protocols))}"
        )

    if protocol == "grpc":
        return GRPCSpanExporter(endpoint=endpoint, insecure=True)
    elif protocol == "http/protobuf":
        return HTTPSpanExporter(endpoint=endpoint)
```
**EN:** This callable implements `get_otlp_span_exporter`. It takes `endpoint` and mainly exports processed data. In this range it performs defensive checks on invalid state; reads environment-driven configuration; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `get_otlp_span_exporter`。它接收 `endpoint`，主要用于导出处理后的数据。 在这一范围内，它会对非法状态执行防御性检查；读取环境变量驱动的配置；记录指标或追踪信号。

### Lines 223-225: Module-level logic
```python


# Should be called by each tracked thread.
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

### Lines 226-246: Function trace_set_thread_info
```python
def trace_set_thread_info(
    thread_label: str,
    tp_rank: Optional[int] = None,
    dp_rank: Optional[int] = None,
    pp_rank: Optional[int] = None,
):
    if not opentelemetry_initialized:
        return

    pid = threading.get_native_id()
    if pid in threads_info:
        return

    threads_info[pid] = TraceThreadInfo(
        host_id=__get_host_id(),
        pid=pid,
        thread_label=thread_label,
        tp_rank=tp_rank,
        dp_rank=dp_rank,
        pp_rank=pp_rank,
    )
```
**EN:** This callable implements `trace_set_thread_info`. It takes `thread_label`, `tp_rank`, `dp_rank`, `pp_rank` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `trace_set_thread_info`。它接收 `thread_label`, `tp_rank`, `dp_rank`, `pp_rank`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 249-249: Class TraceReqContext
```python
class TraceReqContext:
```
**EN:** This range introduces `TraceReqContext` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `TraceReqContext`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 250-282: Method TraceReqContext.__init__
```python
    def __init__(
        self,
        rid,
        bootstrap_room=None,
        role="unified",
        module_name="",
        external_trace_header: Optional[Dict[str, str]] = None,
    ):
        self.rid: str = str(rid)
        self.trace_level = global_trace_level
        self.tracing_enable: bool = opentelemetry_initialized and self.trace_level > 0

        if not self.tracing_enable:
            return

        self.start_time_ns: Optional[int] = None
        self.thread_context: Optional[TraceThreadContext] = None
        self.bootstrap_room: Optional[int] = bootstrap_room
        self.role: str = role
        self.module_name = module_name

        # Indicates whether this instance is a replica from the main process.
        # When True, root_span is None and only root_span_context is preserved.
        self.is_copy: bool = False
        self.root_span: Optional[trace.span.Span] = None
        self.root_span_context: Optional[context.Context] = None
        # Record the most recently completed span as the previous span for the next span to be created.
        self.last_span_context: Optional[trace.span.SpanContext] = None
        self.external_trace_header: Optional[Dict[str, str]] = external_trace_header

        self.events_cache: List[TraceEvent] = []

        self.pid: int = threading.get_native_id()
```
**EN:** This callable implements `TraceReqContext.__init__`. It takes `rid`, `bootstrap_room`, `role`, `module_name` and mainly initializes instance state and defaults. In this range it sets up imports and shared symbols; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.__init__`。它接收 `rid`, `bootstrap_room`, `role`, `module_name`，主要用于初始化实例状态与默认值。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号。

### Lines 284-285: Method TraceReqContext.is_tracing_enabled
```python
    def is_tracing_enabled(self) -> bool:
        return self.tracing_enable
```
**EN:** This callable implements `TraceReqContext.is_tracing_enabled` and mainly implements is tracing enabled.
**CN:** 这一可调用对象实现了 `TraceReqContext.is_tracing_enabled`，主要用于实现 is tracing enabled 相关逻辑。

### Lines 287-329: Method TraceReqContext.__create_thread_context
```python
    def __create_thread_context(self, ts: int):
        if self.pid not in threads_info:
            trace_set_thread_info("unknown")

        thread_info = threads_info[self.pid]
        thread_context = TraceThreadContext(
            thread_info=thread_info,
            cur_slice_stack=[],
        )

        thread_name = f"{thread_info.thread_label}"
        if thread_info.tp_rank is not None:
            thread_name += f" [TP {thread_info.tp_rank}] "
        if thread_info.pp_rank is not None:
            thread_name += f" [PP {thread_info.pp_rank}] "
        if thread_info.dp_rank is not None:
            thread_name += f" [DP {thread_info.dp_rank}] "
        thread_name += f"(host:{thread_info.host_id[:8]} | pid:{self.pid})"
        thread_context.thread_span = tracer.start_span(
            name=thread_name,
            start_time=ts,
            context=self.root_span_context,
        )

        rank_attrs = {}
        if thread_info.tp_rank is not None:
            rank_attrs["tp_rank"] = thread_info.tp_rank
        if thread_info.pp_rank is not None:
            rank_attrs["pp_rank"] = thread_info.pp_rank
        if thread_info.dp_rank is not None:
            rank_attrs["dp_rank"] = thread_info.dp_rank
        if rank_attrs:
            thread_context.thread_span.set_attributes(rank_attrs)

        thread_context.thread_span.set_attributes(
            {
                "host_id": thread_info.host_id,
                "pid": thread_info.pid,
                "thread_label": thread_info.thread_label,
            }
        )

        return thread_context
```
**EN:** This callable implements `TraceReqContext.__create_thread_context`. It takes `ts` and mainly constructs new objects or contexts. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.__create_thread_context`。它接收 `ts`，主要用于构造新的对象或上下文。 在这一范围内，它会记录指标或追踪信号。

### Lines 331-369: Method TraceReqContext.__getstate__
```python
    def __getstate__(self) -> Optional[Dict[str, Any]]:
        if not self.tracing_enable:
            return {"tracing_enable": False}

        if not self.root_span_context:
            return {"tracing_enable": False}

        state = {
            "tracing_enable": self.tracing_enable,
            "rid": self.rid,
            "bootstrap_room": self.bootstrap_room,
            "start_time_ns": self.start_time_ns,
            "role": self.role,
            "trace_level": self.trace_level,
            "module_name": self.module_name,
            "is_copy": self.is_copy,
            "pid": self.pid,
            "thread_context": None,
            "root_span": None,
            "last_span_context": None,
        }

        carrier: dict[str, str] = {}
        propagate.inject(carrier, self.root_span_context)
        state["root_span_context"] = carrier

        prev_span_context = self.last_span_context
        if self.thread_context and self.thread_context.cur_slice_stack:
            cur_slice = self.thread_context.cur_slice_stack[0]
            if cur_slice.span:
                prev_span_context = cur_slice.span.get_span_context()

        if prev_span_context:
            state["last_span_context"] = {
                "span_id": prev_span_context.span_id,
                "trace_id": prev_span_context.trace_id,
            }

        return state
```
**EN:** This callable implements `TraceReqContext.__getstate__` and mainly retrieves a value or derived view. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.__getstate__`，主要用于获取某个值或派生视图。 在这一范围内，它会记录指标或追踪信号。

### Lines 371-387: Method TraceReqContext.__setstate__
```python
    def __setstate__(self, state: Dict[str, Any]):
        self.__dict__.update(state)
        if not opentelemetry_initialized:
            self.tracing_enable = False
        if not self.tracing_enable:
            return

        self.is_copy = True
        self.pid = threading.get_native_id()
        self.root_span_context = propagate.extract(self.root_span_context)
        if self.last_span_context:
            self.last_span_context = trace.span.SpanContext(
                trace_id=self.last_span_context["trace_id"],
                span_id=self.last_span_context["span_id"],
                is_remote=True,
            )
        self.events_cache = []
```
**EN:** This callable implements `TraceReqContext.__setstate__`. It takes `state` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.__setstate__`。它接收 `state`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 389-394: Method TraceReqContext.rebuild_thread_context
```python
    def rebuild_thread_context(self, ts: Optional[int] = None):
        if not self.tracing_enable:
            return

        ts = ts or get_cur_time_ns()
        self.thread_context = self.__create_thread_context(ts)
```
**EN:** This callable implements `TraceReqContext.rebuild_thread_context`. It takes `ts` and mainly builds derived structures.
**CN:** 这一可调用对象实现了 `TraceReqContext.rebuild_thread_context`。它接收 `ts`，主要用于构建派生结构。

### Lines 396-429: Method TraceReqContext.trace_req_start
```python
    def trace_req_start(
        self,
        ts: Optional[int] = None,
    ):
        if not self.tracing_enable:
            return

        ts = ts or get_cur_time_ns()

        # create req context and root span
        self.start_time_ns = ts

        external_trace_context = _trace_context_propagator.extract(
            self.external_trace_header or {}
        )

        # Drop the worker_id added by MultiTokenizer
        orig_rid = self.rid.split("_")[-1]
        role = "" if self.role == "unified" else self.role
        attrs = {"rid": orig_rid, "module": f"sglang::{self.module_name}"}
        if self.bootstrap_room:
            attrs["bootstrap_room"] = str(hex(self.bootstrap_room))
        root_span = tracer.start_span(
            name=f"{role} Req {orig_rid[:8]}",
            start_time=ts,
            context=external_trace_context,
            attributes=attrs,
        )

        self.root_span = root_span
        self.root_span_context = trace.set_span_in_context(root_span)

        # create thread context and thread span
        self.thread_context = self.__create_thread_context(ts)
```
**EN:** This callable implements `TraceReqContext.trace_req_start`. It takes `ts` and mainly implements trace req start. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.trace_req_start`。它接收 `ts`，主要用于实现 trace req start 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 431-449: Method TraceReqContext.trace_req_finish
```python
    def trace_req_finish(
        self, ts: Optional[int] = None, attrs: Optional[Dict[str, Any]] = None
    ):
        if not self.tracing_enable:
            return

        if not self.root_span:
            return

        ts = ts or get_cur_time_ns()

        # End all unclosed thread spans.
        self.abort()

        if attrs:
            self.root_span.set_attributes(attrs)

        self.root_span.end(end_time=ts)
        self.root_span = None
```
**EN:** This callable implements `TraceReqContext.trace_req_finish`. It takes `ts`, `attrs` and mainly implements trace req finish. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.trace_req_finish`。它接收 `ts`, `attrs`，主要用于实现 trace req finish 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 451-461: Method TraceReqContext.__check_fast_return
```python
    def __check_fast_return(self, level=None):
        if not self.tracing_enable:
            return True

        if not self.thread_context:
            return True

        if level and level > self.trace_level:
            return True

        return False
```
**EN:** This callable implements `TraceReqContext.__check_fast_return`. It takes `level` and mainly checks preconditions and compatibility. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.__check_fast_return`。它接收 `level`，主要用于检查前置条件与兼容性。 在这一范围内，它会记录指标或追踪信号。

### Lines 463-502: Method TraceReqContext.trace_slice_start
```python
    def trace_slice_start(
        self,
        name: str,
        level: int,
        ts: Optional[int] = None,
    ):
        if self.__check_fast_return(level):
            return

        ts = ts or get_cur_time_ns()

        cur_slice = TraceSliceContext(
            slice_name=name,
            start_time_ns=ts,
            level=level,
            attrs={},
            events=[],
        )

        parent_span = self.thread_context.thread_span
        prev_span_context = None
        if not self.thread_context.cur_slice_stack:
            if self.last_span_context:
                prev_span_context = self.last_span_context
        else:
            parent_span = self.thread_context.cur_slice_stack[-1].span

        parent_span_context = trace.set_span_in_context(parent_span)

        span = tracer.start_span(
            name=cur_slice.slice_name,
            start_time=cur_slice.start_time_ns,
            context=parent_span_context,
        )
        cur_slice.span = span

        if prev_span_context:
            span.add_link(prev_span_context)

        self.thread_context.cur_slice_stack.append(cur_slice)
```
**EN:** This callable implements `TraceReqContext.trace_slice_start`. It takes `name`, `level`, `ts` and mainly implements trace slice start. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.trace_slice_start`。它接收 `name`, `level`, `ts`，主要用于实现 trace slice start 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 504-559: Method TraceReqContext.trace_slice_end
```python
    def trace_slice_end(
        self,
        name: str,
        level: int,
        ts: Optional[int] = None,
        attrs: Optional[Dict[str, Any]] = None,
        thread_finish_flag: bool = False,
    ):
        if self.__check_fast_return(level):
            return

        if not self.thread_context.cur_slice_stack:
            logger.warning(
                f"No matching with the SLICE_START event {name} is required."
            )
            return

        cur_slice = self.thread_context.cur_slice_stack[-1]
        ts = ts or get_cur_time_ns()

        # check if slice_name matching and level matching
        # unlikely path, excepting error API usage
        if cur_slice.slice_name != name or cur_slice.level != level:
            logger.warning(
                f"Slice name mismatch: {name} != {cur_slice.slice_name} or level mismatch: {level} != {cur_slice.level}"
            )
            self.thread_context.cur_slice_stack.pop()
            return

        span = cur_slice.span

        if attrs:
            span.set_attributes(attrs)

        if self.events_cache:
            new_events_cache = []
            for event in self.events_cache:
                if event.ts >= cur_slice.start_time_ns and event.ts < ts:
                    span.add_event(
                        name=event.event_name,
                        timestamp=event.ts,
                        attributes=event.attrs,
                    )
                else:
                    new_events_cache.append(event)
            self.events_cache = new_events_cache

        span.end(end_time=ts)

        self.thread_context.cur_slice_stack.pop()
        # only for first level slice
        if not self.thread_context.cur_slice_stack:
            self.last_span_context = span.get_span_context()

        if thread_finish_flag:
            self.abort(ts)
```
**EN:** This callable implements `TraceReqContext.trace_slice_end`. It takes `name`, `level`, `ts`, `attrs` and mainly implements trace slice end. In this range it emits logs for diagnostics; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.trace_slice_end`。它接收 `name`, `level`, `ts`, `attrs`，主要用于实现 trace slice end 相关逻辑。 在这一范围内，它会输出日志以便诊断；记录指标或追踪信号。

### Lines 561-617: Method TraceReqContext.trace_slice
```python
    def trace_slice(
        self,
        slice: TraceSliceContext,
        thread_finish_flag: bool = False,
    ):
        if self.__check_fast_return(slice.level):
            return

        parent_span = self.thread_context.thread_span
        prev_span_context = None
        if not self.thread_context.cur_slice_stack:
            if self.last_span_context:
                prev_span_context = self.last_span_context
        else:
            parent_span = self.thread_context.cur_slice_stack[-1].span

        parent_span_context = trace.set_span_in_context(parent_span)

        span = tracer.start_span(
            name=slice.slice_name,
            start_time=slice.start_time_ns,
            context=parent_span_context,
        )

        if prev_span_context:
            span.add_link(prev_span_context)

        if slice.attrs:
            span.set_attributes(slice.attrs)

        if slice.events:
            for event in slice.events:
                span.add_event(
                    name=event.event_name, timestamp=event.ts, attributes=event.attrs
                )

        if self.events_cache:
            new_events_cache = []
            for event in self.events_cache:
                if event.ts >= slice.start_time_ns and event.ts < slice.end_time_ns:
                    span.add_event(
                        name=event.event_name,
                        timestamp=event.ts,
                        attributes=event.attrs,
                    )
                else:
                    new_events_cache.append(event)
            self.events_cache = new_events_cache

        span.end(end_time=slice.end_time_ns)

        # only for first level slice
        if not self.thread_context.cur_slice_stack:
            self.last_span_context = span.get_span_context()

        if thread_finish_flag:
            self.abort(slice.end_time_ns)
```
**EN:** This callable implements `TraceReqContext.trace_slice`. It takes `slice`, `thread_finish_flag` and mainly implements trace slice. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.trace_slice`。它接收 `slice`, `thread_finish_flag`，主要用于实现 trace slice 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 618-619: Class-level scaffolding for TraceReqContext
```python

    # Add event to the current slice on the same thread with the same rid.
```
**EN:** This callable implements `None.TraceReqContext` and mainly implements Trace Req Context.
**CN:** 这一可调用对象实现了 `None.TraceReqContext`，主要用于实现 Trace Req Context 相关逻辑。

### Lines 620-634: Method TraceReqContext.trace_event
```python
    def trace_event(
        self,
        name: str,
        level: int,
        ts: Optional[int] = None,
        attrs: Dict[str, Any] = None,
    ):
        if self.__check_fast_return(level):
            return

        ts = ts or get_cur_time_ns()

        if attrs is None:
            attrs = {}
        self.events_cache.append(TraceEvent(name, ts, attrs))
```
**EN:** This callable implements `TraceReqContext.trace_event`. It takes `name`, `level`, `ts`, `attrs` and mainly implements trace event. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.trace_event`。它接收 `name`, `level`, `ts`, `attrs`，主要用于实现 trace event 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 636-641: Method TraceReqContext.trace_set_root_attrs
```python
    def trace_set_root_attrs(self, attrs: Dict[str, Any]):
        if not self.tracing_enable:
            return

        if self.root_span:
            self.root_span.set_attributes(attrs)
```
**EN:** This callable implements `TraceReqContext.trace_set_root_attrs`. It takes `attrs` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.trace_set_root_attrs`。它接收 `attrs`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 643-648: Method TraceReqContext.trace_set_thread_attrs
```python
    def trace_set_thread_attrs(self, attrs: Dict[str, Any]):
        if self.__check_fast_return():
            return

        if self.thread_context.thread_span:
            self.thread_context.thread_span.set_attributes(attrs)
```
**EN:** This callable implements `TraceReqContext.trace_set_thread_attrs`. It takes `attrs` and mainly applies configuration to mutable state. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.trace_set_thread_attrs`。它接收 `attrs`，主要用于将配置写入可变状态。 在这一范围内，它会记录指标或追踪信号。

### Lines 650-681: Method TraceReqContext.abort
```python
    def abort(self, ts=None, abort_info: Optional[Dict] = None):
        if self.__check_fast_return():
            return

        # close all slice spans (unlikely, except error API usage)
        ts = ts or get_cur_time_ns()
        while len(self.thread_context.cur_slice_stack) > 0:
            if self.thread_context.cur_slice_stack[-1].span:
                self.thread_context.cur_slice_stack[-1].span.end(end_time=ts)
            self.thread_context.cur_slice_stack.pop()

        # set abort info into thread span
        if self.thread_context.thread_span:
            if abort_info:
                from sglang.srt.managers.schedule_batch import BaseFinishReason

                if isinstance(abort_info, BaseFinishReason):
                    abort_info = abort_info.to_json()
                self.thread_context.thread_span.set_status(Status(StatusCode.ERROR))
                self.thread_context.thread_span.set_attributes(abort_info)

            if self.events_cache:
                for event in self.events_cache:
                    self.thread_context.thread_span.add_event(
                        name=event.event_name,
                        timestamp=event.ts,
                        attributes=event.attrs,
                    )
                self.events_cache = []

            self.thread_context.thread_span.end(end_time=ts)
        self.thread_context = None
```
**EN:** This callable implements `TraceReqContext.abort`. It takes `ts`, `abort_info` and mainly implements abort. In this range it sets up imports and shared symbols; records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.abort`。它接收 `ts`, `abort_info`，主要用于实现 abort 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号。

### Lines 683-684: Method TraceReqContext.__del__
```python
    def __del__(self):
        self.abort(abort_info={"reason": "have unclosed span, auto closed"})
```
**EN:** This callable implements `TraceReqContext.__del__` and mainly implements del. In this range it records metrics or tracing signals.
**CN:** 这一可调用对象实现了 `TraceReqContext.__del__`，主要用于实现 del 相关逻辑。 在这一范围内，它会记录指标或追踪信号。

### Lines 687-690: Class TraceNullContext
```python
@dataclass
class TraceNullContext:
    tracing_enable: bool = False

```
**EN:** This range introduces `TraceNullContext` and defines the structure or metadata that its methods rely on. In this range it records metrics or tracing signals.
**CN:** 这一段引入 `TraceNullContext`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会记录指标或追踪信号。

### Lines 691-692: Method TraceNullContext.__getattr__
```python
    def __getattr__(self, name):
        return self
```
**EN:** This callable implements `TraceNullContext.__getattr__`. It takes `name` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `TraceNullContext.__getattr__`。它接收 `name`，主要用于获取某个值或派生视图。

### Lines 694-695: Method TraceNullContext.__call__
```python
    def __call__(self, *args, **kwargs):
        return self
```
**EN:** This callable implements `TraceNullContext.__call__`. It takes `*args`, `**kwargs` and mainly handles invocation-time behavior.
**CN:** 这一可调用对象实现了 `TraceNullContext.__call__`。它接收 `*args`, `**kwargs`，主要用于处理调用时的行为。

### Lines 698-717: Class SpanAttributes
```python
class SpanAttributes:
    # Attribute names copied from here to avoid version conflicts:
    # https://github.com/open-telemetry/semantic-conventions/blob/main/docs/gen-ai/gen-ai-spans.md
    GEN_AI_USAGE_COMPLETION_TOKENS = "gen_ai.usage.completion_tokens"
    GEN_AI_USAGE_PROMPT_TOKENS = "gen_ai.usage.prompt_tokens"
    GEN_AI_USAGE_CACHED_TOKENS = "gen_ai.usage.cached_tokens"
    GEN_AI_REQUEST_MAX_TOKENS = "gen_ai.request.max_tokens"
    GEN_AI_REQUEST_TOP_P = "gen_ai.request.top_p"
    GEN_AI_REQUEST_TOP_K = "gen_ai.request.top_k"
    GEN_AI_REQUEST_TEMPERATURE = "gen_ai.request.temperature"
    GEN_AI_RESPONSE_MODEL = "gen_ai.response.model"
    GEN_AI_RESPONSE_FINISH_REASONS = "gen_ai.response.finish_reasons"
    GEN_AI_REQUEST_ID = "gen_ai.request.id"
    GEN_AI_REQUEST_N = "gen_ai.request.n"
    GEN_AI_LATENCY_TIME_IN_QUEUE = "gen_ai.latency.time_in_queue"
    GEN_AI_LATENCY_TIME_TO_FIRST_TOKEN = "gen_ai.latency.time_to_first_token"
    GEN_AI_LATENCY_E2E = "gen_ai.latency.e2e"
    GEN_AI_LATENCY_TIME_IN_MODEL_PREFILL = "gen_ai.latency.time_in_model_prefill"
    GEN_AI_LATENCY_TIME_IN_MODEL_DECODE = "gen_ai.latency.time_in_model_decode"
    GEN_AI_LATENCY_TIME_IN_MODEL_INFERENCE = "gen_ai.latency.time_in_model_inference"
```
**EN:** This range introduces `SpanAttributes` and defines the structure or metadata that its methods rely on. In this range it sets up imports and shared symbols; records metrics or tracing signals; adjusts sampling behavior and decoding controls.
**CN:** 这一段引入 `SpanAttributes`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会建立导入关系并准备共享符号；记录指标或追踪信号；调整采样行为与解码控制。

## Key Concepts / 关键概念
- `TRACE_HEADERS`: module constant or capability flag / 模块常量或能力标记
- `extract_trace_headers`: implements extract trace headers / 实现 extract trace headers 相关逻辑
- `set_global_trace_level`: applies configuration to mutable state / 将配置写入可变状态
- `TraceThreadInfo`: core class or state container / 核心类或状态容器
- `TraceEvent`: core class or state container / 核心类或状态容器
- `TraceSliceContext`: core class or state container / 核心类或状态容器
- `TraceThreadContext`: core class or state container / 核心类或状态容器
- `TraceCustomIdGenerator`: core class or state container / 核心类或状态容器
- `__get_host_id`: retrieves a value or derived view / 获取某个值或派生视图
- `process_tracing_init`: implements process tracing init / 实现 process tracing init 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `os`, `random`, `threading`, `time`, `uuid`, `dataclasses`, `typing`
- **Third-party / 第三方**: `opentelemetry`, `opentelemetry.exporter.otlp.proto.grpc.trace_exporter`, `opentelemetry.exporter.otlp.proto.http.trace_exporter`, `opentelemetry.sdk.environment_variables`, `opentelemetry.sdk.resources`, `opentelemetry.sdk.trace`, `opentelemetry.sdk.trace.export`, `opentelemetry.trace`, `opentelemetry.trace.propagation.tracecontext`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.srt.managers.schedule_batch`
