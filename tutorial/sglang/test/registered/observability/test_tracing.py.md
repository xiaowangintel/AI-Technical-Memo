# test_tracing.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/observability/test_tracing.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates tracing behavior in SGLang's observability area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 observability 领域中与 tracing 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""Integration tests for tracing with a lightweight in-process OTLP collector.

This module implements a minimal OTLP collector that receives traces via gRPC
and stores them in memory for test assertions, eliminating the need for
Docker-based opentelemetry-collector and file I/O.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-8: module imports and dependencies / 模块导入与依赖
```python
import os
```
**EN:** This block imports the modules needed by the rest of the file, including `os`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`。

### Lines 9-11: supporting source context / 辅助源码上下文
```python

# Configure OTLP exporter for faster test execution
# Must be set before importing sglang trace module
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 12-13: supporting statements / 辅助语句
```python
os.environ.setdefault("SGLANG_OTLP_EXPORTER_SCHEDULE_DELAY_MILLIS", "50")
os.environ.setdefault("SGLANG_OTLP_EXPORTER_MAX_EXPORT_BATCH_SIZE", "4")
```
**EN:** This block performs supporting work through calls such as `setdefault`, preparing state for nearby definitions.
**CN:** 该代码块通过 `setdefault` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 15-44: module imports and dependencies / 模块导入与依赖
```python
import logging
import multiprocessing as mp
import time
import unittest
from dataclasses import dataclass
from typing import List, Optional, Union

import requests
import zmq

from sglang import Engine
from sglang.srt.observability.req_time_stats import RequestStage
from sglang.srt.observability.trace import (
    TraceReqContext,
    TraceSliceContext,
    get_cur_time_ns,
    process_tracing_init,
    set_global_trace_level,
    trace_set_thread_info,
)
from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.network import get_zmq_socket
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `logging`, `multiprocessing`, `time`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `logging`, `multiprocessing`, `time`, `unittest`。

### Lines 46-50: CI registration and metadata / CI 注册与元数据
```python
logger = logging.getLogger(__name__)

# CI registration
register_cuda_ci(est_time=113, stage="extra-a", runner_config="1-gpu-small")
register_amd_ci(est_time=113, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through getLogger, register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 getLogger, register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 51-56: supporting source context / 辅助源码上下文
```python


# ============================================================================
# Lightweight OTLP Collector (shared across tracing tests)
# ============================================================================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 57-57: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.otel_collector import LightweightOtlpCollector, Span  # noqa: F401
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.otel_collector`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.otel_collector`。

### Lines 58-63: supporting source context / 辅助源码上下文
```python

# ============================================================================
# Test Helper Functions
# ============================================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 64-81: function get span names by level / 函数 get span names by level
```python
def _get_span_names_by_level(level: int) -> List[str]:
    """Get expected span names for a given trace level.

    Based on RequestStage definitions in req_time_stats.py:
    - Each RequestStage has a level attribute indicating minimum trace level required
    - Spans with level <= current trace level will be exported
    """
    span_names = []
    # RequestStage is a class with class attributes that are RequestStageConfig instances
    for attr_name in dir(RequestStage):
        if attr_name.startswith("_"):
            continue
        attr = getattr(RequestStage, attr_name)
        # Check if it's a RequestStageConfig (has stage_name and level attributes)
        if hasattr(attr, "stage_name") and hasattr(attr, "level"):
            if attr.level <= level and attr.stage_name:
                span_names.append(attr.stage_name)
    return span_names
```
**EN:** Get expected span names for a given trace level. This block implements `_get_span_names_by_level` and captures one focused piece of the module's behavior.
**CN:** Get expected span names for a given trace level. 该代码块实现 `_get_span_names_by_level`，承担模块行为中的一个聚焦逻辑片段。

### Lines 82-84: supporting source context / 辅助源码上下文
```python


# Pre-computed span names by level for efficiency
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 85-104: module-level constants and configuration / 模块级常量与配置
```python
SPAN_NAMES_LEVEL_1 = _get_span_names_by_level(1)
SPAN_NAMES_LEVEL_2 = _get_span_names_by_level(2)
SPAN_NAMES_LEVEL_3 = _get_span_names_by_level(3)

# Common span names expected in typical inference requests
# Level 1: Basic request lifecycle
EXPECTED_SPANS_LEVEL_1 = [
    RequestStage.PREFILL_FORWARD.stage_name,
    RequestStage.DECODE_FORWARD.stage_name,
]

# Level 2: More detailed including dispatch
EXPECTED_SPANS_LEVEL_2 = EXPECTED_SPANS_LEVEL_1 + [
    RequestStage.REQUEST_PROCESS.stage_name,
]

# Level 3: Most detailed including internal operations
EXPECTED_SPANS_LEVEL_3 = EXPECTED_SPANS_LEVEL_2 + [
    RequestStage.DECODE_LOOP.stage_name,
]
```
**EN:** This block defines shared names such as `SPAN_NAMES_LEVEL_1`, `SPAN_NAMES_LEVEL_2`, `SPAN_NAMES_LEVEL_3`, `EXPECTED_SPANS_LEVEL_1`, `EXPECTED_SPANS_LEVEL_2`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `SPAN_NAMES_LEVEL_1`, `SPAN_NAMES_LEVEL_2`, `SPAN_NAMES_LEVEL_3`, `EXPECTED_SPANS_LEVEL_1`, `EXPECTED_SPANS_LEVEL_2` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 107-108: class Req declaration / 类 Req 声明
```python
@dataclass
class Req:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 109-110: class-level constants and configuration for `Req` / 类级常量与配置
```python
    rid: int
    req_context: Optional[Union[TraceReqContext]] = None
```
**EN:** This block defines shared names such as `rid`, `req_context`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `rid`, `req_context` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 113-131: function subprocess worker / 函数 subprocess worker
```python
def _subprocess_worker():
    """Worker function for subprocess trace context propagation test.
    Must be at module level for pickle compatibility with spawn.
    """
    process_tracing_init("127.0.0.1:4317", "test")
    trace_set_thread_info("Sub Process")

    context = zmq.Context(2)
    recv_from_main = get_zmq_socket(context, zmq.PULL, "ipc:///tmp/zmq_test.ipc", True)

    try:
        req = recv_from_main.recv_pyobj()
        req.req_context.rebuild_thread_context()
        req.req_context.trace_slice_start("work", level=1)
        time.sleep(0.2)
        req.req_context.trace_slice_end("work", level=1, thread_finish_flag=True)
    finally:
        recv_from_main.close()
        context.term()
```
**EN:** Worker function for subprocess trace context propagation test. This block implements `_subprocess_worker` and captures one focused piece of the module's behavior.
**CN:** Worker function for subprocess trace context propagation test. 该代码块实现 `_subprocess_worker`，承担模块行为中的一个聚焦逻辑片段。

### Lines 132-138: supporting source context / 辅助源码上下文
```python


# ============================================================================
# Test Cases
# ============================================================================


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 139-139: class TestTracePackage declaration / 类 TestTracePackage 声明
```python
class TestTracePackage(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 140-140: supporting statements / 辅助语句
```python
    """Unit tests for tracing package API without server/engine."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 142-143: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.collector = None
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 145-148: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        if self.collector:
            self.collector.stop()
            self.collector = None
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 150-154: method start collector / 方法 start collector
```python
    def _start_collector(self):
        """Start the lightweight OTLP collector."""
        self.collector = LightweightOtlpCollector()
        self.collector.start()
        time.sleep(0.2)
```
**EN:** Start the lightweight OTLP collector. This block implements `_start_collector` and captures one focused piece of the module's behavior.
**CN:** Start the lightweight OTLP collector. 该代码块实现 `_start_collector`，承担模块行为中的一个聚焦逻辑片段。

### Lines 156-178: test case slice simple / 测试用例 slice simple
```python
    def test_slice_simple(self):
        """Unit test: simple slice trace API."""
        self._start_collector()

        try:
            process_tracing_init("127.0.0.1:4317", "test")
            trace_set_thread_info("Test")
            set_global_trace_level(3)
            req_context = TraceReqContext(0)
            req_context.trace_req_start()
            req_context.trace_slice_start("test slice", level=1)
            time.sleep(0.1)
            req_context.trace_slice_end("test slice", level=1)
            req_context.trace_req_finish()

            time.sleep(0.3)

            self.assertTrue(
                self.collector.has_span("test slice"),
                f"Expected span 'test slice', got {self.collector.get_span_names()}",
            )
        finally:
            pass
```
**EN:** Unit test: simple slice trace API. This test exercises `test_slice_simple` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Unit test: simple slice trace API. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_slice_simple`。

### Lines 180-211: test case slice complex / 测试用例 slice complex
```python
    def test_slice_complex(self):
        """Unit test: complex slice trace with events."""
        self._start_collector()

        try:
            process_tracing_init("127.0.0.1:4317", "test")
            trace_set_thread_info("Test")
            set_global_trace_level(3)
            req_context = TraceReqContext(0)
            req_context.trace_req_start()

            t1 = get_cur_time_ns()
            time.sleep(0.1)
            req_context.trace_event("event test", 1)
            t2 = get_cur_time_ns()
            time.sleep(0.1)
            t3 = get_cur_time_ns()

            slice1 = TraceSliceContext("slice A", t1, t2)
            slice2 = TraceSliceContext("slice B", t2, t3)
            req_context.trace_slice(slice1)
            req_context.trace_slice(slice2, thread_finish_flag=True)
            req_context.trace_req_finish()

            time.sleep(0.3)

            self.assertTrue(
                self.collector.has_all_spans(["slice A", "slice B"]),
                f"Expected spans 'slice A' and 'slice B', got {self.collector.get_span_names()}",
            )
        finally:
            pass
```
**EN:** Unit test: complex slice trace with events. This test exercises `test_slice_complex` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Unit test: complex slice trace with events. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_slice_complex`。

### Lines 213-252: test case context propagate / 测试用例 context propagate
```python
    def test_context_propagate(self):
        """Unit test: trace context propagation across processes via ZMQ."""
        self._start_collector()

        ctx = mp.get_context("spawn")

        context = zmq.Context(2)
        send_to_subproc = get_zmq_socket(
            context, zmq.PUSH, "ipc:///tmp/zmq_test.ipc", False
        )

        try:
            process_tracing_init("127.0.0.1:4317", "test")
            trace_set_thread_info("Main Process")

            subproc = ctx.Process(target=_subprocess_worker)
            subproc.start()

            time.sleep(0.3)

            req = Req(rid=0)
            req.req_context = TraceReqContext(0)
            req.req_context.trace_req_start()
            req.req_context.trace_slice_start("dispatch", level=1)
            time.sleep(0.2)
            send_to_subproc.send_pyobj(req)
            req.req_context.trace_slice_end("dispatch", level=1)

            subproc.join()
            req.req_context.trace_req_finish()

            time.sleep(0.5)

            self.assertTrue(
                self.collector.has_all_spans(["dispatch", "work"]),
                f"Expected spans 'dispatch' and 'work', got {self.collector.get_span_names()}",
            )
        finally:
            send_to_subproc.close()
            context.term()
```
**EN:** Unit test: trace context propagation across processes via ZMQ. This test exercises `test_context_propagate` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Unit test: trace context propagation across processes via ZMQ. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_context_propagate`。

### Lines 255-255: class TestTraceServer declaration / 类 TestTraceServer 声明
```python
class TestTraceServer(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 256-256: supporting statements / 辅助语句
```python
    """Integration tests for tracing with server - starts server once for all tests."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 258-280: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        """Start collector and server once for all tests."""
        cls.collector = LightweightOtlpCollector()
        cls.collector.start()
        time.sleep(0.2)

        cls.process = popen_launch_server(
            DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
            DEFAULT_URL_FOR_TEST,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--enable-trace",
                "--otlp-traces-endpoint",
                "127.0.0.1:4317",
            ],
        )

        response = requests.get(f"{DEFAULT_URL_FOR_TEST}/health_generate")
        assert response.status_code == 200

        # Wait for warmup spans to be exported
        cls.collector.clear()
```
**EN:** Start collector and server once for all tests. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Start collector and server once for all tests. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 282-287: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if cls.process:
            kill_process_tree(cls.process.pid)
        if cls.collector:
            cls.collector.stop()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 289-312: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Wait for spans to be drained before each test."""
        max_wait_seconds = 10
        check_interval = 0.2
        elapsed = 0
        consecutive_zero_count = 0
        required_consecutive_zeros = 3

        while elapsed < max_wait_seconds:
            span_count = self.collector.count_spans()
            if span_count == 0:
                consecutive_zero_count += 1
                if consecutive_zero_count >= required_consecutive_zeros:
                    break
            else:
                consecutive_zero_count = 0
                self.collector.clear()
            time.sleep(check_interval)
            elapsed += check_interval
        else:
            raise RuntimeError(
                f"Timeout waiting for spans to drain after {max_wait_seconds}s. "
                f"Remaining spans: {self.collector.count_spans()}"
            )
```
**EN:** Wait for spans to be drained before each test. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Wait for spans to be drained before each test. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 314-343: method send request and wait / 方法 send request and wait
```python
    def _send_request_and_wait(
        self, text, max_new_tokens=32, stream=True, trace_level=None
    ):
        """Helper to send a request and wait for spans."""
        if trace_level is not None:
            response = requests.get(
                f"{DEFAULT_URL_FOR_TEST}/set_trace_level?level={trace_level}"
            )
            self.assertEqual(response.status_code, 200)
            self.collector.clear()

        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/generate",
            json={
                "text": text,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": max_new_tokens,
                },
                "stream": stream,
            },
            stream=stream,
        )
        if stream:
            for _ in response.iter_lines(decode_unicode=False):
                pass
        else:
            self.assertEqual(response.status_code, 200)

        time.sleep(1)
```
**EN:** Helper to send a request and wait for spans. This block implements `_send_request_and_wait` and captures one focused piece of the module's behavior.
**CN:** Helper to send a request and wait for spans. 该代码块实现 `_send_request_and_wait`，承担模块行为中的一个聚焦逻辑片段。

### Lines 345-352: test case trace level 0 / 测试用例 trace level 0
```python
    def test_trace_level_0(self):
        """Test trace level 0 does not export any spans."""
        self._send_request_and_wait("Hello world", max_new_tokens=5, trace_level=0)
        self.assertEqual(
            self.collector.count_spans(),
            0,
            f"Spans collected but expected none: {sorted(self.collector.get_span_names())}",
        )
```
**EN:** Test trace level 0 does not export any spans. This test exercises `test_trace_level_0` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test trace level 0 does not export any spans. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_level_0`。

### Lines 354-371: test case trace level 1 / 测试用例 trace level 1
```python
    def test_trace_level_1(self):
        """Test trace level 1 exports basic request lifecycle spans."""
        self._send_request_and_wait("The capital of France is", trace_level=1)

        self.assertGreater(
            self.collector.count_spans(),
            0,
            "No spans collected but expected some",
        )

        span_names = self.collector.get_span_names()
        matched = [name for name in EXPECTED_SPANS_LEVEL_1 if name in span_names]
        self.assertGreater(
            len(matched),
            0,
            f"No expected spans found. Expected any of {EXPECTED_SPANS_LEVEL_1}, "
            f"got {sorted(span_names)}",
        )
```
**EN:** Test trace level 1 exports basic request lifecycle spans. This test exercises `test_trace_level_1` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test trace level 1 exports basic request lifecycle spans. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_level_1`。

### Lines 373-384: test case trace level 2 / 测试用例 trace level 2
```python
    def test_trace_level_2(self):
        """Test trace level 2 exports more detailed spans."""
        self._send_request_and_wait("What is AI?", trace_level=2)

        span_names = self.collector.get_span_names()
        matched = [name for name in EXPECTED_SPANS_LEVEL_2 if name in span_names]
        self.assertGreater(
            len(matched),
            0,
            f"No expected spans found. Expected any of {EXPECTED_SPANS_LEVEL_2}, "
            f"got {sorted(span_names)}",
        )
```
**EN:** Test trace level 2 exports more detailed spans. This test exercises `test_trace_level_2` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test trace level 2 exports more detailed spans. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_level_2`。

### Lines 386-397: test case trace level 3 / 测试用例 trace level 3
```python
    def test_trace_level_3(self):
        """Test trace level 3 exports most detailed spans."""
        self._send_request_and_wait("Explain quantum computing", trace_level=3)

        span_names = self.collector.get_span_names()
        matched = [name for name in EXPECTED_SPANS_LEVEL_3 if name in span_names]
        self.assertGreater(
            len(matched),
            0,
            f"No expected spans found. Expected any of {EXPECTED_SPANS_LEVEL_3}, "
            f"got {sorted(span_names)}",
        )
```
**EN:** Test trace level 3 exports most detailed spans. This test exercises `test_trace_level_3` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test trace level 3 exports most detailed spans. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_level_3`。

### Lines 399-436: test case batch request / 测试用例 batch request
```python
    def test_batch_request(self):
        """Test tracing with batch requests (multiple prompts in one request)."""
        response = requests.get(f"{DEFAULT_URL_FOR_TEST}/set_trace_level?level=1")
        self.assertEqual(response.status_code, 200)
        self.collector.clear()

        batch_size = 4
        prompts = ["The capital of France is"] * batch_size
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/generate",
            json={
                "text": prompts,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 10,
                },
                "stream": False,
            },
        )
        self.assertEqual(response.status_code, 200)

        time.sleep(0.5)

        self.assertGreater(
            self.collector.count_spans(),
            0,
            "No spans collected from batch request",
        )

        all_spans = self.collector.get_spans()
        request_spans = [
            s for s in all_spans if s.name == RequestStage.PREFILL_FORWARD.stage_name
        ]
        self.assertEqual(
            len(request_spans),
            batch_size,
            f"Expected {batch_size} prefill_forward spans, got {len(request_spans)}",
        )
```
**EN:** Test tracing with batch requests (multiple prompts in one request). This test exercises `test_batch_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test tracing with batch requests (multiple prompts in one request). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_request`。

### Lines 438-477: test case parallel sample / 测试用例 parallel sample
```python
    def test_parallel_sample(self):
        """Test tracing with parallel sampling (n > 1 in sampling_params)."""
        response = requests.get(f"{DEFAULT_URL_FOR_TEST}/set_trace_level?level=1")
        self.assertEqual(response.status_code, 200)
        self.collector.clear()

        # parallel_sample_num is controlled by 'n' in sampling_params
        parallel_num = 4
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 0.5,  # Need non-zero temp for parallel sampling
                    "max_new_tokens": 10,
                    "n": parallel_num,
                },
                "stream": False,
            },
        )
        self.assertEqual(response.status_code, 200)

        time.sleep(0.5)

        self.assertGreater(
            self.collector.count_spans(),
            0,
            "No spans collected from parallel sample request",
        )

        # With parallel sampling, we expect prefill spans for each parallel sample
        all_spans = self.collector.get_spans()
        request_spans = [
            s for s in all_spans if s.name == RequestStage.PREFILL_FORWARD.stage_name
        ]
        self.assertGreaterEqual(
            len(request_spans),
            1,
            f"Expected at least 1 prefill_forward span, got {len(request_spans)}",
        )
```
**EN:** Test tracing with parallel sampling (n > 1 in sampling_params). This test exercises `test_parallel_sample` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test tracing with parallel sampling (n > 1 in sampling_params). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parallel_sample`。

### Lines 480-480: class TestTraceEngine declaration / 类 TestTraceEngine 声明
```python
class TestTraceEngine(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 481-481: supporting statements / 辅助语句
```python
    """Integration tests for tracing with Engine API - each test creates its own engine."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 483-484: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.collector = None
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 486-489: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        if self.collector:
            self.collector.stop()
            self.collector = None
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 491-495: method start collector / 方法 start collector
```python
    def _start_collector(self):
        """Start the lightweight OTLP collector."""
        self.collector = LightweightOtlpCollector()
        self.collector.start()
        time.sleep(0.2)
```
**EN:** Start the lightweight OTLP collector. This block implements `_start_collector` and captures one focused piece of the module's behavior.
**CN:** Start the lightweight OTLP collector. 该代码块实现 `_start_collector`，承担模块行为中的一个聚焦逻辑片段。

### Lines 497-526: test case trace engine enable / 测试用例 trace engine enable
```python
    def test_trace_engine_enable(self):
        """Test tracing with Engine API."""
        self._start_collector()

        prompt = "Today is a sunny day and I like"
        model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        sampling_params = {"temperature": 0, "max_new_tokens": 8}

        engine = Engine(
            model_path=model_path,
            random_seed=42,
            enable_trace=True,
            otlp_traces_endpoint="localhost:4317",
        )

        try:
            engine.generate(prompt, sampling_params)
            time.sleep(0.5)

            self.assertGreater(
                self.collector.count_spans(),
                0,
                "No spans collected from Engine.generate",
            )
            self.assertTrue(
                self.collector.has_any_span([RequestStage.PREFILL_FORWARD.stage_name]),
                f"Expected prefill_forward span, got {self.collector.get_span_names()}",
            )
        finally:
            engine.shutdown()
```
**EN:** Test tracing with Engine API. This test exercises `test_trace_engine_enable` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test tracing with Engine API. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_engine_enable`。

### Lines 528-553: test case trace engine encode / 测试用例 trace engine encode
```python
    def test_trace_engine_encode(self):
        """Test tracing with Engine encode API."""
        self._start_collector()

        prompt = "Today is a sunny day and I like"
        model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST

        engine = Engine(
            model_path=model_path,
            random_seed=42,
            enable_trace=True,
            otlp_traces_endpoint="localhost:4317",
            is_embedding=True,
        )

        try:
            engine.encode(prompt)
            time.sleep(0.5)

            self.assertGreater(
                self.collector.count_spans(),
                0,
                "No spans collected from Engine.encode",
            )
        finally:
            engine.shutdown()
```
**EN:** Test tracing with Engine encode API. This test exercises `test_trace_engine_encode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test tracing with Engine encode API. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trace_engine_encode`。

### Lines 556-557: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_get_span_names_by_level`: Get expected span names for a given trace level. / 该代码块实现 `_get_span_names_by_level`，承担模块行为中的一个聚焦逻辑片段。
- `Req`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_subprocess_worker`: Worker function for subprocess trace context propagation test. / 该代码块实现 `_subprocess_worker`，承担模块行为中的一个聚焦逻辑片段。
- `TestTracePackage`: Unit tests for tracing package API without server/engine. / 用于组织相关测试、夹具或辅助方法。
- `TestTraceServer`: Integration tests for tracing with server - starts server once for all tests. / 用于组织相关测试、夹具或辅助方法。
- `TestTraceEngine`: Integration tests for tracing with Engine API - each test creates its own engine. / 用于组织相关测试、夹具或辅助方法。
- `TestTracePackage.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestTracePackage.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestTracePackage._start_collector`: Start the lightweight OTLP collector. / 该代码块实现 `_start_collector`，承担模块行为中的一个聚焦逻辑片段。
- `TestTracePackage.test_slice_simple`: Unit test: simple slice trace API. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_slice_simple`。
- `TestTracePackage.test_slice_complex`: Unit test: complex slice trace with events. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_slice_complex`。
- `TestTracePackage.test_context_propagate`: Unit test: trace context propagation across processes via ZMQ. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_context_propagate`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `logging`, `multiprocessing`, `time`, `unittest`, `dataclasses`, `typing`
- **Third-party modules / 第三方模块**: `requests`, `zmq`
- **Internal modules / 内部模块**: `sglang`, `sglang.srt.observability.req_time_stats`, `sglang.srt.observability.trace`, `sglang.srt.utils`, `sglang.srt.utils.network`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`, `sglang.test.otel_collector`

- **Total lines / 总行数**: 557
