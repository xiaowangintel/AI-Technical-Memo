# test_trace.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/observability/test_trace.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates trace behavior in SGLang's unit / observability area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / observability 领域中与 trace 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for trace.py — no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-5: module imports and dependencies / 模块导入与依赖
```python
import os

from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `sglang.test.ci.ci_register`。

### Lines 7-7: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=6, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 9-28: module imports and dependencies / 模块导入与依赖
```python
import threading
import unittest
from unittest.mock import patch

import sglang.srt.observability.trace as mod
from sglang.srt.observability.trace import (
    SpanAttributes,
    TraceCustomIdGenerator,
    TraceEvent,
    TraceNullContext,
    TraceReqContext,
    TraceSliceContext,
    TraceThreadContext,
    TraceThreadInfo,
    extract_trace_headers,
    get_global_tracing_enabled,
    process_tracing_init,
    set_global_trace_level,
    trace_set_thread_info,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `threading`, `unittest`, `unittest.mock`, `sglang.srt.observability.trace`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `threading`, `unittest`, `unittest.mock`, `sglang.srt.observability.trace`。

### Lines 30-41: module-level constants and configuration / 模块级常量与配置
```python
try:
    from opentelemetry import trace as otel_trace
    from opentelemetry.sdk.trace import TracerProvider

    from sglang.srt.observability.trace import get_otlp_span_exporter

    _has_otel = True
except ImportError:
    _has_otel = False

# Access the private module-level function (avoid name mangling inside classes).
_get_host_id = getattr(mod, "__get_host_id")
```
**EN:** This block defines shared names such as `_has_otel`, `_get_host_id`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_has_otel`, `_get_host_id` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 44-44: class TestTraceFunctions declaration / 类 TestTraceFunctions 声明
```python
class TestTraceFunctions(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 45-48: test case extract trace headers / 测试用例 extract trace headers
```python
    def test_extract_trace_headers(self):
        headers = {"traceparent": "abc", "tracestate": "xyz", "other": "skip"}
        result = extract_trace_headers(headers)
        self.assertEqual(result, {"traceparent": "abc", "tracestate": "xyz"})
```
**EN:** This test exercises `test_extract_trace_headers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extract_trace_headers`。

### Lines 50-51: test case extract trace headers missing / 测试用例 extract trace headers missing
```python
    def test_extract_trace_headers_missing(self):
        self.assertEqual(extract_trace_headers({}), {})
```
**EN:** This test exercises `test_extract_trace_headers_missing` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extract_trace_headers_missing`。

### Lines 53-57: test case set global trace level / 测试用例 set global trace level
```python
    def test_set_global_trace_level(self):
        orig = mod.global_trace_level
        set_global_trace_level(5)
        self.assertEqual(mod.global_trace_level, 5)
        mod.global_trace_level = orig
```
**EN:** This test exercises `test_set_global_trace_level` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_global_trace_level`。

### Lines 59-66: test case global trace level env var / 测试用例 global trace level env var
```python
    def test_global_trace_level_env_var(self):
        import importlib

        with patch.dict(os.environ, {"SGLANG_TRACE_LEVEL": "2"}):
            importlib.reload(mod)
            self.assertEqual(mod.global_trace_level, 2)
        importlib.reload(mod)  # restore default (SGLANG_TRACE_LEVEL unset → 3)
        self.assertEqual(mod.global_trace_level, 3)
```
**EN:** This test exercises `test_global_trace_level_env_var` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_global_trace_level_env_var`。

### Lines 68-69: test case get global tracing enabled / 测试用例 get global tracing enabled
```python
    def test_get_global_tracing_enabled(self):
        self.assertEqual(get_global_tracing_enabled(), mod.opentelemetry_initialized)
```
**EN:** This test exercises `test_get_global_tracing_enabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_global_tracing_enabled`。

### Lines 71-74: test case get cur time ns / 测试用例 get cur time ns
```python
    def test_get_cur_time_ns(self):
        ts = mod.get_cur_time_ns()
        self.assertIsInstance(ts, int)
        self.assertGreater(ts, 0)
```
**EN:** This test exercises `test_get_cur_time_ns` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_cur_time_ns`。

### Lines 77-77: class TestDataclasses declaration / 类 TestDataclasses 声明
```python
class TestDataclasses(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 78-80: test case trace thread info / 测试用例 trace thread info
```python
    def test_trace_thread_info(self):
        info = TraceThreadInfo("host", 123, "label", 0, 1, 0)
        self.assertEqual(info.thread_label, "label")
```
**EN:** This test exercises `test_trace_thread_info` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_thread_info`。

### Lines 82-84: test case trace event / 测试用例 trace event
```python
    def test_trace_event(self):
        evt = TraceEvent("name", 100, {"k": "v"})
        self.assertEqual(evt.event_name, "name")
```
**EN:** This test exercises `test_trace_event` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_event`。

### Lines 86-88: test case trace slice context / 测试用例 trace slice context
```python
    def test_trace_slice_context(self):
        s = TraceSliceContext("slice", 100, end_time_ns=200, level=2, attrs={"a": 1})
        self.assertEqual(s.slice_name, "slice")
```
**EN:** This test exercises `test_trace_slice_context` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_slice_context`。

### Lines 90-93: test case trace thread context / 测试用例 trace thread context
```python
    def test_trace_thread_context(self):
        info = TraceThreadInfo("h", 1, "l", 0, 0, 0)
        ctx = TraceThreadContext(thread_info=info, cur_slice_stack=[])
        self.assertEqual(len(ctx.cur_slice_stack), 0)
```
**EN:** This test exercises `test_trace_thread_context` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_thread_context`。

### Lines 96-96: class TestTraceNullContext declaration / 类 TestTraceNullContext 声明
```python
class TestTraceNullContext(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 97-105: test case null object pattern / 测试用例 null object pattern
```python
    def test_null_object_pattern(self):
        ctx = TraceNullContext()
        self.assertFalse(ctx.tracing_enable)
        # Any attribute access returns self
        self.assertIs(ctx.some_method, ctx)
        # Callable returns self
        self.assertIs(ctx("arg1", key="val"), ctx)
        # Chaining works
        self.assertIs(ctx.foo.bar.baz(1, 2, 3), ctx)
```
**EN:** This test exercises `test_null_object_pattern` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_null_object_pattern`。

### Lines 108-108: class TestSpanAttributes declaration / 类 TestSpanAttributes 声明
```python
class TestSpanAttributes(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 109-111: test case constants exist / 测试用例 constants exist
```python
    def test_constants_exist(self):
        self.assertEqual(SpanAttributes.GEN_AI_LATENCY_E2E, "gen_ai.latency.e2e")
        self.assertIsInstance(SpanAttributes.GEN_AI_USAGE_COMPLETION_TOKENS, str)
```
**EN:** This test exercises `test_constants_exist` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_constants_exist`。

### Lines 114-114: class TestTraceCustomIdGenerator declaration / 类 TestTraceCustomIdGenerator 声明
```python
class TestTraceCustomIdGenerator(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 115-120: test case generates nonzero ids / 测试用例 generates nonzero ids
```python
    def test_generates_nonzero_ids(self):
        gen = TraceCustomIdGenerator()
        trace_id = gen.generate_trace_id()
        span_id = gen.generate_span_id()
        self.assertIsInstance(trace_id, int)
        self.assertIsInstance(span_id, int)
```
**EN:** This test exercises `test_generates_nonzero_ids` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generates_nonzero_ids`。

### Lines 121-123: supporting source context / 辅助源码上下文
```python


# __get_host_id
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 124-124: class TestGetHostId declaration / 类 TestGetHostId 声明
```python
class TestGetHostId(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 125-133: test case from machine id file / 测试用例 from machine id file
```python
    def test_from_machine_id_file(self):
        with (
            patch("os.path.exists", return_value=True),
            patch(
                "builtins.open",
                unittest.mock.mock_open(read_data="abc123\n"),
            ),
        ):
            self.assertEqual(_get_host_id(), "abc123")
```
**EN:** This test exercises `test_from_machine_id_file` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_from_machine_id_file`。

### Lines 135-143: test case from machine id file error / 测试用例 from machine id file error
```python
    def test_from_machine_id_file_error(self):
        """Falls back to MAC address when file read fails."""
        with (
            patch("os.path.exists", return_value=True),
            patch("builtins.open", side_effect=IOError("read error")),
        ):
            result = _get_host_id()
            self.assertIsInstance(result, str)
            self.assertGreater(len(result), 0)
```
**EN:** Falls back to MAC address when file read fails. This test exercises `test_from_machine_id_file_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Falls back to MAC address when file read fails. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_from_machine_id_file_error`。

### Lines 145-152: test case from mac address / 测试用例 from mac address
```python
    def test_from_mac_address(self):
        with (
            patch("os.path.exists", return_value=False),
            patch("uuid.getnode", return_value=0x112233445566),
        ):
            result = _get_host_id()
            self.assertIsInstance(result, str)
            self.assertGreater(len(result), 0)
```
**EN:** This test exercises `test_from_mac_address` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_from_mac_address`。

### Lines 154-159: test case unknown fallback / 测试用例 unknown fallback
```python
    def test_unknown_fallback(self):
        with (
            patch("os.path.exists", return_value=False),
            patch("uuid.getnode", return_value=0),
        ):
            self.assertEqual(_get_host_id(), "unknown")
```
**EN:** This test exercises `test_unknown_fallback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_fallback`。

### Lines 162-163: class TestGetOtlpSpanExporter declaration / 类 TestGetOtlpSpanExporter 声明
```python
@unittest.skipUnless(_has_otel, "opentelemetry not installed")
class TestGetOtlpSpanExporter(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 164-169: test case grpc default / 测试用例 grpc default
```python
    def test_grpc_default(self):

        with patch.dict(os.environ, {}, clear=False):
            os.environ.pop("OTEL_EXPORTER_OTLP_TRACES_PROTOCOL", None)
            exporter = get_otlp_span_exporter("localhost:4317")
        self.assertIsNotNone(exporter)
```
**EN:** This test exercises `test_grpc_default` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_grpc_default`。

### Lines 171-177: test case http protobuf / 测试用例 http protobuf
```python
    def test_http_protobuf(self):

        with patch.dict(
            os.environ, {"OTEL_EXPORTER_OTLP_TRACES_PROTOCOL": "http/protobuf"}
        ):
            exporter = get_otlp_span_exporter("http://localhost:4318/v1/traces")
        self.assertIsNotNone(exporter)
```
**EN:** This test exercises `test_http_protobuf` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_http_protobuf`。

### Lines 179-183: test case invalid protocol / 测试用例 invalid protocol
```python
    def test_invalid_protocol(self):

        with patch.dict(os.environ, {"OTEL_EXPORTER_OTLP_TRACES_PROTOCOL": "invalid"}):
            with self.assertRaises(ValueError):
                get_otlp_span_exporter("localhost:4317")
```
**EN:** This test exercises `test_invalid_protocol` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_protocol`。

### Lines 186-186: class TestProcessTracingInit declaration / 类 TestProcessTracingInit 声明
```python
class TestProcessTracingInit(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 187-195: test case raises without otel / 测试用例 raises without otel
```python
    def test_raises_without_otel(self):

        orig = mod.opentelemetry_imported
        mod.opentelemetry_imported = False
        try:
            with self.assertRaises(RuntimeError):
                process_tracing_init("localhost:4317", "test")
        finally:
            mod.opentelemetry_imported = orig
```
**EN:** This test exercises `test_raises_without_otel` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_raises_without_otel`。

### Lines 198-198: class TestTraceReqContextDisabled declaration / 类 TestTraceReqContextDisabled 声明
```python
class TestTraceReqContextDisabled(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 199-201: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.orig = mod.opentelemetry_initialized
        mod.opentelemetry_initialized = False
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 203-204: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        mod.opentelemetry_initialized = self.orig
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 206-209: test case init disabled / 测试用例 init disabled
```python
    def test_init_disabled(self):
        ctx = TraceReqContext(rid="req-1")
        self.assertFalse(ctx.tracing_enable)
        self.assertFalse(ctx.is_tracing_enabled())
```
**EN:** This test exercises `test_init_disabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_init_disabled`。

### Lines 211-222: test case all methods noop / 测试用例 all methods noop
```python
    def test_all_methods_noop(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start()
        ctx.trace_req_finish()
        ctx.trace_slice_start("s", 1)
        ctx.trace_slice_end("s", 1)
        ctx.trace_slice(TraceSliceContext("s", 100))
        ctx.trace_event("e", 1)
        ctx.trace_set_root_attrs({"k": "v"})
        ctx.trace_set_thread_attrs({"k": "v"})
        ctx.abort()
        ctx.rebuild_thread_context()
```
**EN:** This test exercises `test_all_methods_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_methods_noop`。

### Lines 224-227: test case getstate disabled / 测试用例 getstate disabled
```python
    def test_getstate_disabled(self):
        ctx = TraceReqContext(rid="req-1")
        state = ctx.__getstate__()
        self.assertEqual(state, {"tracing_enable": False})
```
**EN:** This test exercises `test_getstate_disabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_getstate_disabled`。

### Lines 229-233: test case setstate disabled / 测试用例 setstate disabled
```python
    def test_setstate_disabled(self):
        ctx = TraceReqContext.__new__(TraceReqContext)
        ctx.__setstate__({"tracing_enable": True, "is_copy": False})
        # opentelemetry_initialized is False → tracing forced off
        self.assertFalse(ctx.tracing_enable)
```
**EN:** This test exercises `test_setstate_disabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_setstate_disabled`。

### Lines 235-236: test case trace set thread info disabled / 测试用例 trace set thread info disabled
```python
    def test_trace_set_thread_info_disabled(self):
        trace_set_thread_info("test_label")
```
**EN:** This test exercises `test_trace_set_thread_info_disabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_set_thread_info_disabled`。

### Lines 237-239: supporting source context / 辅助源码上下文
```python
        # Should not register anything


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 240-241: class TestTraceReqContextEnabled declaration / 类 TestTraceReqContextEnabled 声明
```python
@unittest.skipUnless(_has_otel, "opentelemetry not installed")
class TestTraceReqContextEnabled(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 242-253: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):

        self.orig_initialized = mod.opentelemetry_initialized
        self.orig_tracer = mod.tracer
        self.orig_threads = mod.threads_info.copy()
        self.orig_level = mod.global_trace_level

        self.provider = TracerProvider()
        otel_trace.set_tracer_provider(self.provider)
        mod.opentelemetry_initialized = True
        mod.tracer = otel_trace.get_tracer("test")
        mod.global_trace_level = 3
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 255-260: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        mod.opentelemetry_initialized = self.orig_initialized
        mod.tracer = self.orig_tracer
        mod.threads_info.clear()
        mod.threads_info.update(self.orig_threads)
        mod.global_trace_level = self.orig_level
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 262-271: test case trace set thread info / 测试用例 trace set thread info
```python
    def test_trace_set_thread_info(self):
        trace_set_thread_info("scheduler", tp_rank=0, dp_rank=0)

        pid = threading.get_native_id()
        self.assertIn(pid, mod.threads_info)
        self.assertEqual(mod.threads_info[pid].thread_label, "scheduler")

        # Second call for same thread is a no-op
        trace_set_thread_info("different_label")
        self.assertEqual(mod.threads_info[pid].thread_label, "scheduler")
```
**EN:** This test exercises `test_trace_set_thread_info` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_set_thread_info`。

### Lines 273-291: test case full lifecycle / 测试用例 full lifecycle
```python
    def test_full_lifecycle(self):
        """Start → slice_start → slice_end → finish."""
        ctx = TraceReqContext(rid="req-1", role="unified", module_name="test")
        self.assertTrue(ctx.tracing_enable)

        ctx.trace_req_start(ts=1000)
        self.assertEqual(ctx.start_time_ns, 1000)
        self.assertIsNotNone(ctx.root_span)
        self.assertIsNotNone(ctx.thread_context)

        ctx.trace_slice_start("prefill", level=1, ts=2000)
        self.assertEqual(len(ctx.thread_context.cur_slice_stack), 1)

        ctx.trace_slice_end("prefill", level=1, ts=3000)
        self.assertEqual(len(ctx.thread_context.cur_slice_stack), 0)
        self.assertIsNotNone(ctx.last_span_context)

        ctx.trace_req_finish(ts=4000, attrs={"tokens": 42})
        self.assertIsNone(ctx.root_span)
```
**EN:** Start → slice_start → slice_end → finish. This test exercises `test_full_lifecycle` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Start → slice_start → slice_end → finish. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_full_lifecycle`。

### Lines 293-297: test case trace req start with bootstrap room / 测试用例 trace req start with bootstrap room
```python
    def test_trace_req_start_with_bootstrap_room(self):
        ctx = TraceReqContext(rid="req-1", bootstrap_room=0xFF, role="prefill")
        ctx.trace_req_start(ts=1000)
        self.assertIsNotNone(ctx.root_span)
        ctx.trace_req_finish(ts=2000)
```
**EN:** This test exercises `test_trace_req_start_with_bootstrap_room` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_req_start_with_bootstrap_room`。

### Lines 299-304: test case trace req finish without start / 测试用例 trace req finish without start
```python
    def test_trace_req_finish_without_start(self):
        """finish without start is a no-op."""
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.root_span = None
        ctx.trace_req_finish(ts=2000)
```
**EN:** finish without start is a no-op. This test exercises `test_trace_req_finish_without_start` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** finish without start is a no-op. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_req_finish_without_start`。

### Lines 306-321: test case trace slice combined / 测试用例 trace slice combined
```python
    def test_trace_slice_combined(self):
        """trace_slice() creates and ends a span in one call."""
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)

        s = TraceSliceContext(
            "decode",
            2000,
            end_time_ns=3000,
            level=1,
            attrs={"key": "val"},
            events=[TraceEvent("evt", 2500, {"e": 1})],
        )
        ctx.trace_slice(s)
        self.assertIsNotNone(ctx.last_span_context)
        ctx.trace_req_finish(ts=4000)
```
**EN:** trace_slice() creates and ends a span in one call. This test exercises `test_trace_slice_combined` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** trace_slice() creates and ends a span in one call. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_slice_combined`。

### Lines 323-336: test case trace slice with events cache / 测试用例 trace slice with events cache
```python
    def test_trace_slice_with_events_cache(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)

        # Add events to cache
        ctx.trace_event("schedule", level=1, ts=1500, attrs={"bid": "x"})
        self.assertEqual(len(ctx.events_cache), 1)

        # trace_slice_start + trace_slice_end flushes matching events
        ctx.trace_slice_start("prefill", level=1, ts=1200)
        ctx.trace_slice_end("prefill", level=1, ts=2000)
        self.assertEqual(len(ctx.events_cache), 0)

        ctx.trace_req_finish(ts=3000)
```
**EN:** This test exercises `test_trace_slice_with_events_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_slice_with_events_cache`。

### Lines 338-346: test case trace slice combined with events cache / 测试用例 trace slice combined with events cache
```python
    def test_trace_slice_combined_with_events_cache(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)

        ctx.trace_event("evt", level=1, ts=1500)
        s = TraceSliceContext("decode", 1200, end_time_ns=2000, level=1)
        ctx.trace_slice(s)
        self.assertEqual(len(ctx.events_cache), 0)
        ctx.trace_req_finish(ts=3000)
```
**EN:** This test exercises `test_trace_slice_combined_with_events_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_slice_combined_with_events_cache`。

### Lines 348-353: test case trace event no attrs / 测试用例 trace event no attrs
```python
    def test_trace_event_no_attrs(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.trace_event("evt", level=1, ts=1500, attrs=None)
        self.assertEqual(ctx.events_cache[0].attrs, {})
        ctx.trace_req_finish(ts=2000)
```
**EN:** This test exercises `test_trace_event_no_attrs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_event_no_attrs`。

### Lines 355-360: test case trace slice end empty stack / 测试用例 trace slice end empty stack
```python
    def test_trace_slice_end_empty_stack(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        # End without start → warning, no crash
        ctx.trace_slice_end("missing", level=1, ts=2000)
        ctx.trace_req_finish(ts=3000)
```
**EN:** This test exercises `test_trace_slice_end_empty_stack` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_slice_end_empty_stack`。

### Lines 362-369: test case trace slice end name mismatch / 测试用例 trace slice end name mismatch
```python
    def test_trace_slice_end_name_mismatch(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.trace_slice_start("prefill", level=1, ts=1500)
        # Mismatched name → warning, slice popped
        ctx.trace_slice_end("wrong_name", level=1, ts=2000)
        self.assertEqual(len(ctx.thread_context.cur_slice_stack), 0)
        ctx.trace_req_finish(ts=3000)
```
**EN:** This test exercises `test_trace_slice_end_name_mismatch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_slice_end_name_mismatch`。

### Lines 371-383: test case trace slice end with attrs and thread finish / 测试用例 trace slice end with attrs and thread finish
```python
    def test_trace_slice_end_with_attrs_and_thread_finish(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.trace_slice_start("dispatch", level=2, ts=1500)
        ctx.trace_slice_end(
            "dispatch",
            level=2,
            ts=2000,
            attrs={"key": "val"},
            thread_finish_flag=True,
        )
        # thread_finish_flag triggers abort → thread_context is None
        self.assertIsNone(ctx.thread_context)
```
**EN:** This test exercises `test_trace_slice_end_with_attrs_and_thread_finish` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_slice_end_with_attrs_and_thread_finish`。

### Lines 385-390: test case trace slice combined with thread finish / 测试用例 trace slice combined with thread finish
```python
    def test_trace_slice_combined_with_thread_finish(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        s = TraceSliceContext("dispatch", 1500, end_time_ns=2000, level=2)
        ctx.trace_slice(s, thread_finish_flag=True)
        self.assertIsNone(ctx.thread_context)
```
**EN:** This test exercises `test_trace_slice_combined_with_thread_finish` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_slice_combined_with_thread_finish`。

### Lines 392-401: test case nested slices / 测试用例 nested slices
```python
    def test_nested_slices(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.trace_slice_start("outer", level=1, ts=1500)
        ctx.trace_slice_start("inner", level=2, ts=1600)
        self.assertEqual(len(ctx.thread_context.cur_slice_stack), 2)
        ctx.trace_slice_end("inner", level=2, ts=1800)
        self.assertEqual(len(ctx.thread_context.cur_slice_stack), 1)
        ctx.trace_slice_end("outer", level=1, ts=2000)
        ctx.trace_req_finish(ts=3000)
```
**EN:** This test exercises `test_nested_slices` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nested_slices`。

### Lines 403-421: test case nested slice with last span context / 测试用例 nested slice with last span context
```python
    def test_nested_slice_with_last_span_context(self):
        """trace_slice uses last_span_context when slice stack is empty."""
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)

        # First slice sets last_span_context
        ctx.trace_slice_start("s1", level=1, ts=1500)
        ctx.trace_slice_end("s1", level=1, ts=2000)
        self.assertIsNotNone(ctx.last_span_context)

        # Second slice uses last_span_context as link
        ctx.trace_slice_start("s2", level=1, ts=2500)
        ctx.trace_slice_end("s2", level=1, ts=3000)

        # trace_slice also uses last_span_context
        s = TraceSliceContext("s3", 3500, end_time_ns=4000, level=1)
        ctx.trace_slice(s)

        ctx.trace_req_finish(ts=5000)
```
**EN:** trace_slice uses last_span_context when slice stack is empty. This test exercises `test_nested_slice_with_last_span_context` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** trace_slice uses last_span_context when slice stack is empty. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nested_slice_with_last_span_context`。

### Lines 423-427: test case trace set root attrs / 测试用例 trace set root attrs
```python
    def test_trace_set_root_attrs(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.trace_set_root_attrs({"model": "llama"})
        ctx.trace_req_finish(ts=2000)
```
**EN:** This test exercises `test_trace_set_root_attrs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_set_root_attrs`。

### Lines 429-433: test case trace set root attrs no span / 测试用例 trace set root attrs no span
```python
    def test_trace_set_root_attrs_no_span(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.root_span = None
        ctx.trace_set_root_attrs({"model": "llama"})  # no crash
```
**EN:** This test exercises `test_trace_set_root_attrs_no_span` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_set_root_attrs_no_span`。

### Lines 435-439: test case trace set thread attrs / 测试用例 trace set thread attrs
```python
    def test_trace_set_thread_attrs(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.trace_set_thread_attrs({"batch_size": 32})
        ctx.trace_req_finish(ts=2000)
```
**EN:** This test exercises `test_trace_set_thread_attrs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_set_thread_attrs`。

### Lines 441-447: test case abort with unclosed slices / 测试用例 abort with unclosed slices
```python
    def test_abort_with_unclosed_slices(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.trace_slice_start("s1", level=1, ts=1500)
        ctx.trace_slice_start("s2", level=2, ts=1600)
        ctx.abort(ts=2000)
        self.assertIsNone(ctx.thread_context)
```
**EN:** This test exercises `test_abort_with_unclosed_slices` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_abort_with_unclosed_slices`。

### Lines 449-454: test case abort with events cache / 测试用例 abort with events cache
```python
    def test_abort_with_events_cache(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.trace_event("evt", level=1, ts=1500)
        ctx.abort(ts=2000)
        self.assertEqual(len(ctx.events_cache), 0)
```
**EN:** This test exercises `test_abort_with_events_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_abort_with_events_cache`。

### Lines 456-460: test case abort with abort info dict / 测试用例 abort with abort info dict
```python
    def test_abort_with_abort_info_dict(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.abort(ts=2000, abort_info={"reason": "cancelled"})
        self.assertIsNone(ctx.thread_context)
```
**EN:** This test exercises `test_abort_with_abort_info_dict` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_abort_with_abort_info_dict`。

### Lines 462-469: test case abort with base finish reason / 测试用例 abort with base finish reason
```python
    def test_abort_with_base_finish_reason(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        from sglang.srt.managers.schedule_batch import FINISH_LENGTH

        abort_obj = FINISH_LENGTH(length=10)
        ctx.abort(ts=2000, abort_info=abort_obj)
        self.assertIsNone(ctx.thread_context)
```
**EN:** This test exercises `test_abort_with_base_finish_reason` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_abort_with_base_finish_reason`。

### Lines 471-479: test case check fast return by level / 测试用例 check fast return by level
```python
    def test_check_fast_return_by_level(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.trace_level = 1  # instance-level, set at init from global
        # Level 2 > trace_level 1 → fast return
        ctx.trace_slice_start("s", level=2, ts=1500)
        self.assertEqual(len(ctx.thread_context.cur_slice_stack), 0)
        ctx.trace_level = 3
        ctx.trace_req_finish(ts=2000)
```
**EN:** This test exercises `test_check_fast_return_by_level` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_check_fast_return_by_level`。

### Lines 481-487: test case rebuild thread context / 测试用例 rebuild thread context
```python
    def test_rebuild_thread_context(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        old_tc = ctx.thread_context
        ctx.rebuild_thread_context(ts=1500)
        self.assertIsNot(ctx.thread_context, old_tc)
        ctx.trace_req_finish(ts=2000)
```
**EN:** This test exercises `test_rebuild_thread_context` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rebuild_thread_context`。

### Lines 489-496: test case getstate enabled / 测试用例 getstate enabled
```python
    def test_getstate_enabled(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        state = ctx.__getstate__()
        self.assertTrue(state["tracing_enable"])
        self.assertEqual(state["rid"], "req-1")
        self.assertIn("root_span_context", state)
        ctx.trace_req_finish(ts=2000)
```
**EN:** This test exercises `test_getstate_enabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_getstate_enabled`。

### Lines 498-505: test case getstate no root context / 测试用例 getstate no root context
```python
    def test_getstate_no_root_context(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.root_span_context = None
        state = ctx.__getstate__()
        self.assertFalse(state["tracing_enable"])
        ctx.root_span_context = True  # prevent __del__ issues
        ctx.trace_req_finish(ts=2000)
```
**EN:** This test exercises `test_getstate_no_root_context` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_getstate_no_root_context`。

### Lines 507-513: test case getstate with slice stack / 测试用例 getstate with slice stack
```python
    def test_getstate_with_slice_stack(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.trace_slice_start("s1", level=1, ts=1500)
        state = ctx.__getstate__()
        self.assertIn("last_span_context", state)
        ctx.trace_req_finish(ts=2000)
```
**EN:** This test exercises `test_getstate_with_slice_stack` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_getstate_with_slice_stack`。

### Lines 515-525: test case setstate enabled / 测试用例 setstate enabled
```python
    def test_setstate_enabled(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        state = ctx.__getstate__()
        ctx.trace_req_finish(ts=2000)

        ctx2 = TraceReqContext.__new__(TraceReqContext)
        ctx2.__setstate__(state)
        self.assertTrue(ctx2.tracing_enable)
        self.assertTrue(ctx2.is_copy)
        self.assertIsNotNone(ctx2.root_span_context)
```
**EN:** This test exercises `test_setstate_enabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_setstate_enabled`。

### Lines 527-537: test case thread context with tp rank / 测试用例 thread context with tp rank
```python
    def test_thread_context_with_tp_rank(self):
        """Covers tp_rank branch in __create_thread_context."""

        pid = threading.get_native_id()
        mod.threads_info[pid] = TraceThreadInfo(
            "host", pid, "sched", tp_rank=0, dp_rank=0, pp_rank=0
        )
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        self.assertIsNotNone(ctx.thread_context)
        ctx.trace_req_finish(ts=2000)
```
**EN:** Covers tp_rank branch in __create_thread_context. This test exercises `test_thread_context_with_tp_rank` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Covers tp_rank branch in __create_thread_context. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_thread_context_with_tp_rank`。

### Lines 539-551: test case setstate with last span context / 测试用例 setstate with last span context
```python
    def test_setstate_with_last_span_context(self):
        """Covers __setstate__ path where last_span_context is truthy."""
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        ctx.trace_slice_start("s1", level=1, ts=1500)
        ctx.trace_slice_end("s1", level=1, ts=2000)
        state = ctx.__getstate__()
        ctx.trace_req_finish(ts=3000)

        self.assertIsNotNone(state.get("last_span_context"))
        ctx2 = TraceReqContext.__new__(TraceReqContext)
        ctx2.__setstate__(state)
        self.assertIsNotNone(ctx2.last_span_context)
```
**EN:** Covers __setstate__ path where last_span_context is truthy. This test exercises `test_setstate_with_last_span_context` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Covers __setstate__ path where last_span_context is truthy. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_setstate_with_last_span_context`。

### Lines 553-566: test case events cache partial match / 测试用例 events cache partial match
```python
    def test_events_cache_partial_match(self):
        """Events outside the slice time range stay in cache."""
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)

        ctx.trace_event("early", level=1, ts=500)
        ctx.trace_event("inside", level=1, ts=1500)
        ctx.trace_event("late", level=1, ts=5000)

        ctx.trace_slice_start("s", level=1, ts=1200)
        ctx.trace_slice_end("s", level=1, ts=2000)
        # "early" (500 < 1200) and "late" (5000 >= 2000) stay in cache
        self.assertEqual(len(ctx.events_cache), 2)
        ctx.trace_req_finish(ts=6000)
```
**EN:** Events outside the slice time range stay in cache. This test exercises `test_events_cache_partial_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Events outside the slice time range stay in cache. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_events_cache_partial_match`。

### Lines 568-579: test case trace slice combined events partial match / 测试用例 trace slice combined events partial match
```python
    def test_trace_slice_combined_events_partial_match(self):
        """Events outside slice range stay in cache for trace_slice method."""
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)

        ctx.trace_event("early", level=1, ts=500)
        ctx.trace_event("inside", level=1, ts=1500)

        s = TraceSliceContext("s", 1200, end_time_ns=2000, level=1)
        ctx.trace_slice(s)
        self.assertEqual(len(ctx.events_cache), 1)  # "early" stays
        ctx.trace_req_finish(ts=3000)
```
**EN:** Events outside slice range stay in cache for trace_slice method. This test exercises `test_trace_slice_combined_events_partial_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Events outside slice range stay in cache for trace_slice method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_slice_combined_events_partial_match`。

### Lines 581-590: test case trace slice nested parent / 测试用例 trace slice nested parent
```python
    def test_trace_slice_nested_parent(self):
        """trace_slice with parent from slice stack (not thread_span)."""
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)

        ctx.trace_slice_start("outer", level=1, ts=1500)
        s = TraceSliceContext("inner", 1600, end_time_ns=1800, level=2)
        ctx.trace_slice(s)
        ctx.trace_slice_end("outer", level=1, ts=2000)
        ctx.trace_req_finish(ts=3000)
```
**EN:** trace_slice with parent from slice stack (not thread_span). This test exercises `test_trace_slice_nested_parent` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** trace_slice with parent from slice stack (not thread_span). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_slice_nested_parent`。

### Lines 592-597: test case del triggers abort / 测试用例 del triggers abort
```python
    def test_del_triggers_abort(self):
        ctx = TraceReqContext(rid="req-1")
        ctx.trace_req_start(ts=1000)
        # __del__ calls abort
        ctx.__del__()
        self.assertIsNone(ctx.thread_context)
```
**EN:** This test exercises `test_del_triggers_abort` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_del_triggers_abort`。

### Lines 600-601: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestTraceFunctions`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDataclasses`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTraceNullContext`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSpanAttributes`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTraceCustomIdGenerator`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGetHostId`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGetOtlpSpanExporter`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestProcessTracingInit`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTraceFunctions.test_extract_trace_headers`: This test exercises `test_extract_trace_headers` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extract_trace_headers`。
- `TestTraceFunctions.test_extract_trace_headers_missing`: This test exercises `test_extract_trace_headers_missing` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extract_trace_headers_missing`。
- `TestTraceFunctions.test_set_global_trace_level`: This test exercises `test_set_global_trace_level` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_global_trace_level`。
- `TestTraceFunctions.test_global_trace_level_env_var`: This test exercises `test_global_trace_level_env_var` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_global_trace_level_env_var`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `threading`, `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.observability.trace`

- **Total lines / 总行数**: 601
