# test_tracing_disaggregation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/observability/test_tracing_disaggregation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates tracing disaggregation behavior in SGLang's observability area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 observability 领域中与 tracing disaggregation 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Test tracing in PD disaggregation mode."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-3: module imports and dependencies / 模块导入与依赖
```python
import os
```
**EN:** This block imports the modules needed by the rest of the file, including `os`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`。

### Lines 4-6: supporting source context / 辅助源码上下文
```python

# Configure OTLP exporter for faster test execution
# Must be set before importing sglang trace module
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 7-8: supporting statements / 辅助语句
```python
os.environ.setdefault("SGLANG_OTLP_EXPORTER_SCHEDULE_DELAY_MILLIS", "50")
os.environ.setdefault("SGLANG_OTLP_EXPORTER_MAX_EXPORT_BATCH_SIZE", "4")
```
**EN:** This block performs supporting work through calls such as `setdefault`, preparing state for nearby definitions.
**CN:** 该代码块通过 `setdefault` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 10-31: module imports and dependencies / 模块导入与依赖
```python
import logging
import shlex
import time
import unittest
from urllib.parse import urlparse

import requests

from sglang.srt.observability.req_time_stats import RequestStage
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.otel_collector import LightweightOtlpCollector
from sglang.test.server_fixtures.disaggregation_fixture import get_rdma_devices_args
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_pd_server,
    popen_with_error_check,
)
from sglang.utils import wait_for_http_ready
```
**EN:** This block imports the modules needed by the rest of the file, including `logging`, `shlex`, `time`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `logging`, `shlex`, `time`, `unittest`。

### Lines 33-36: CI registration and metadata / CI 注册与元数据
```python
logger = logging.getLogger(__name__)

# CI registration - PD disaggregation requires 2 GPUs
register_cuda_ci(est_time=65, stage="base-b", runner_config="2-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through getLogger, register_cuda_ci.
**CN:** 该代码块通过 getLogger, register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 39-39: class TestTraceDisaggregation declaration / 类 TestTraceDisaggregation 声明
```python
class TestTraceDisaggregation(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 40-40: supporting statements / 辅助语句
```python
    """Test tracing in PD disaggregation mode."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 42-121: setUpClass setup routine (part 1/2) / setUpClass 初始化流程（第 1/2 部分）
```python
    @classmethod
    def setUpClass(cls):
        # Initialize collector first
        cls.collector = LightweightOtlpCollector()
        cls.collector.start()
        time.sleep(0.2)

        # Setup PD disaggregation server addresses
        parsed_url = urlparse(DEFAULT_URL_FOR_TEST)
        cls.base_host = parsed_url.hostname
        base_port = str(parsed_url.port)
        cls.lb_port = base_port
        cls.prefill_port = f"{int(base_port) + 100}"
        cls.decode_port = f"{int(base_port) + 200}"
        cls.bootstrap_port = f"{int(base_port) + 500}"
        cls.prefill_url = f"http://{cls.base_host}:{cls.prefill_port}"
        cls.decode_url = f"http://{cls.base_host}:{cls.decode_port}"
        cls.lb_url = f"http://{cls.base_host}:{cls.lb_port}"
        cls.process_lb = None
        cls.process_decode = None
        cls.process_prefill = None
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST

        # Config transfer backend
        cls.transfer_backend = ["--disaggregation-transfer-backend", "mooncake"]
        cls.rdma_devices = ["--disaggregation-ib-device", get_rdma_devices_args()]

        # Start prefill server with trace enabled
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--enable-trace",
            "--otlp-traces-endpoint",
            "localhost:4317",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )

        # Start decode server with trace enabled
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--base-gpu-id",
            "1",
            "--enable-trace",
            "--otlp-traces-endpoint",
            "localhost:4317",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )

        # Wait for servers to be ready
        wait_for_http_ready(
            url=cls.prefill_url + "/health",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            process=cls.process_prefill,
        )
        wait_for_http_ready(
            url=cls.decode_url + "/health",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute. This chunk continues part 1 of the same logical block.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。 这一段对应同一逻辑块的第 1 部分。

### Lines 122-147: setUpClass setup routine (part 2/2) / setUpClass 初始化流程（第 2/2 部分）
```python
            process=cls.process_decode,
        )

        # Start load balancer
        lb_command = [
            "python3",
            "-m",
            "sglang_router.launch_router",
            "--pd-disaggregation",
            "--mini-lb",
            "--prefill",
            cls.prefill_url,
            "--decode",
            cls.decode_url,
            "--host",
            cls.base_host,
            "--port",
            cls.lb_port,
        ]
        print("Starting load balancer:", shlex.join(lb_command))
        cls.process_lb = popen_with_error_check(lb_command)
        wait_for_http_ready(url=cls.lb_url + "/health", process=cls.process_lb)

        # Wait for warmup spans and clear
        time.sleep(1)
        cls.collector.clear()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute. This chunk continues part 2 of the same logical block.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。 这一段对应同一逻辑块的第 2 部分。

### Lines 149-159: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        for process in [cls.process_lb, cls.process_decode, cls.process_prefill]:
            if process:
                try:
                    kill_process_tree(process.pid)
                except Exception as e:
                    print(f"Error killing process {process.pid}: {e}")
        if cls.collector:
            cls.collector.stop()
        time.sleep(5)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 161-184: setUp setup routine / setUp 初始化流程
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

### Lines 186-231: test case disaggregation transfer spans / 测试用例 disaggregation transfer spans
```python
    def test_disaggregation_transfer_spans(self):
        """Test that disaggregation produces PREFILL_TRANSFER_KV_CACHE and DECODE_TRANSFERRED spans."""
        # Set trace level
        response = requests.get(f"{self.prefill_url}/set_trace_level?level=1")
        self.assertEqual(response.status_code, 200)
        response = requests.get(f"{self.decode_url}/set_trace_level?level=1")
        self.assertEqual(response.status_code, 200)
        self.collector.clear()

        # Send a request through load balancer
        response = requests.post(
            f"{self.lb_url}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 10,
                },
                "stream": False,
            },
        )
        self.assertEqual(response.status_code, 200)

        # Wait for async export
        time.sleep(1)

        # Verify spans were collected
        self.assertGreater(
            self.collector.count_spans(),
            0,
            "No spans collected from disaggregation request",
        )

        # Verify disaggregation-specific spans exist
        span_names = self.collector.get_span_names()

        # Check for transfer-related spans
        self.assertTrue(
            self.collector.has_any_span(
                [
                    RequestStage.PREFILL_TRANSFER_KV_CACHE.stage_name,
                    RequestStage.DECODE_TRANSFERRED.stage_name,
                ]
            ),
            f"Expected disaggregation transfer spans, got {sorted(span_names)}",
        )
```
**EN:** Test that disaggregation produces PREFILL_TRANSFER_KV_CACHE and DECODE_TRANSFERRED spans. This test exercises `test_disaggregation_transfer_spans` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that disaggregation produces PREFILL_TRANSFER_KV_CACHE and DECODE_TRANSFERRED spans. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_disaggregation_transfer_spans`。

### Lines 234-235: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestTraceDisaggregation`: Test tracing in PD disaggregation mode. / 用于组织相关测试、夹具或辅助方法。
- `TestTraceDisaggregation.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestTraceDisaggregation.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestTraceDisaggregation.setUp`: Wait for spans to be drained before each test. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestTraceDisaggregation.test_disaggregation_transfer_spans`: Test that disaggregation produces PREFILL_TRANSFER_KV_CACHE and DECODE_TRANSFERRED spans. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_disaggregation_transfer_spans`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `logging`, `shlex`, `time`, `unittest`, `urllib.parse`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.observability.req_time_stats`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.otel_collector`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`, `sglang.utils`

- **Total lines / 总行数**: 235
