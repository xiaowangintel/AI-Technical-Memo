# test_priority_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/observability/test_priority_metrics.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates priority metrics behavior in SGLang's observability area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 observability 领域中与 priority metrics 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: module imports and dependencies / 模块导入与依赖
```python
import unittest
from typing import Dict, List
from unittest.mock import Mock

import requests
from prometheus_client.parser import text_string_to_metric_families
from prometheus_client.samples import Sample

from sglang.srt.observability.metrics_collector import QueueCount
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `typing`, `unittest.mock`, `requests`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `typing`, `unittest.mock`, `requests`。

### Lines 19-26: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(
    est_time=60,
    stage="base-b",
    runner_config="1-gpu-small",
)
register_amd_ci(est_time=60, suite="stage-b-test-1-gpu-small-amd")

_MODEL_NAME = "Qwen/Qwen3-0.6B"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 29-36: function parse prometheus metrics / 函数 parse prometheus metrics
```python
def _parse_prometheus_metrics(metrics_text: str) -> Dict[str, List[Sample]]:
    result = {}
    for family in text_string_to_metric_families(metrics_text):
        for sample in family.samples:
            if sample.name not in result:
                result[sample.name] = []
            result[sample.name].append(sample)
    return result
```
**EN:** This block implements `_parse_prometheus_metrics` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_parse_prometheus_metrics`，承担模块行为中的一个聚焦逻辑片段。

### Lines 39-40: function get samples by name / 函数 get samples by name
```python
def _get_samples_by_name(metrics: Dict[str, List[Sample]], name: str) -> List[Sample]:
    return metrics.get(name, [])
```
**EN:** This block implements `_get_samples_by_name` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_samples_by_name`，承担模块行为中的一个聚焦逻辑片段。

### Lines 43-47: function get sample value by labels / 函数 get sample value by labels
```python
def _get_sample_value_by_labels(samples: List[Sample], labels: Dict[str, str]) -> float:
    for sample in samples:
        if all(sample.labels.get(k) == v for k, v in labels.items()):
            return sample.value
    raise KeyError(f"No sample found with labels {labels}")
```
**EN:** This block implements `_get_sample_value_by_labels` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_sample_value_by_labels`，承担模块行为中的一个聚焦逻辑片段。

### Lines 50-50: class TestQueueCount declaration / 类 TestQueueCount 声明
```python
class TestQueueCount(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 51-51: supporting statements / 辅助语句
```python
    """Unit tests for QueueCount (no server needed)."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 53-64: test case queue count from reqs / 测试用例 queue count from reqs
```python
    def test_queue_count_from_reqs(self):
        """QueueCount correctly counts per-priority breakdown."""
        reqs = [
            Mock(priority=1),
            Mock(priority=1),
            Mock(priority=5),
            Mock(priority=5),
            Mock(priority=10),
        ]
        qc = QueueCount.from_reqs(reqs, enable_priority_scheduling=True)
        self.assertEqual(qc.total, 5)
        self.assertEqual(qc.by_priority, {1: 2, 5: 2, 10: 1})
```
**EN:** QueueCount correctly counts per-priority breakdown. This test exercises `test_queue_count_from_reqs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** QueueCount correctly counts per-priority breakdown. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_queue_count_from_reqs`。

### Lines 66-71: test case queue count from reqs disabled / 测试用例 queue count from reqs disabled
```python
    def test_queue_count_from_reqs_disabled(self):
        """Priority scheduling disabled → no breakdown."""
        reqs = [Mock(priority=1), Mock(priority=5)]
        qc = QueueCount.from_reqs(reqs, enable_priority_scheduling=False)
        self.assertEqual(qc.total, 2)
        self.assertIsNone(qc.by_priority)
```
**EN:** Priority scheduling disabled → no breakdown. This test exercises `test_queue_count_from_reqs_disabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Priority scheduling disabled → no breakdown. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_queue_count_from_reqs_disabled`。

### Lines 73-77: test case queue count empty / 测试用例 queue count empty
```python
    def test_queue_count_empty(self):
        """Empty request list."""
        qc = QueueCount.from_reqs([], enable_priority_scheduling=True)
        self.assertEqual(qc.total, 0)
        self.assertEqual(qc.by_priority, {})
```
**EN:** Empty request list. This test exercises `test_queue_count_empty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Empty request list. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_queue_count_empty`。

### Lines 80-80: class TestPriorityMetrics declaration / 类 TestPriorityMetrics 声明
```python
class TestPriorityMetrics(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 81-82: supporting statements / 辅助语句
```python
    """Test that priority-based metrics are correctly emitted when
    --enable-priority-scheduling is enabled."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 84-96: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.process = popen_launch_server(
            _MODEL_NAME,
            DEFAULT_URL_FOR_TEST,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--enable-metrics",
                "--enable-priority-scheduling",
                "--default-priority-value",
                "0",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 98-100: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 102-136: test case priority label in gauge metrics / 测试用例 priority label in gauge metrics
```python
    def test_priority_label_in_gauge_metrics(self):
        """Send requests with different priorities and verify that
        gauge metrics (num_running_reqs, num_queue_reqs) contain
        the priority label dimension."""

        # Send requests with different priorities to populate metrics
        for priority in [1, 5, 10]:
            response = requests.post(
                f"{DEFAULT_URL_FOR_TEST}/generate",
                json={
                    "text": "Hello",
                    "sampling_params": {"temperature": 0, "max_new_tokens": 5},
                    "priority": priority,
                },
            )
            self.assertEqual(response.status_code, 200)

        # Fetch metrics
        metrics_response = requests.get(f"{DEFAULT_URL_FOR_TEST}/metrics")
        self.assertEqual(metrics_response.status_code, 200)
        metrics = _parse_prometheus_metrics(metrics_response.text)

        # Verify priority label exists on queue gauge metrics
        for metric_name in ["sglang:num_running_reqs", "sglang:num_queue_reqs"]:
            samples = _get_samples_by_name(metrics, metric_name)
            self.assertGreater(len(samples), 0, f"No samples found for {metric_name}")

            # Should have at least one sample with a non-empty priority label
            # (the total has priority="" and per-priority has priority="<int>")
            priority_labels = {s.labels.get("priority", "") for s in samples}
            self.assertIn(
                "",
                priority_labels,
                f"{metric_name}: missing total (priority='') sample",
            )
```
**EN:** Send requests with different priorities and verify that gauge metrics (num_running_reqs, num_queue_reqs) contain the priority label dimension. This test exercises `test_priority_label_in_gauge_metrics` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Send requests with different priorities and verify that gauge metrics (num_running_reqs, num_queue_reqs) contain the priority label dimension. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_label_in_gauge_metrics`。

### Lines 138-190: test case priority label in histogram metrics / 测试用例 priority label in histogram metrics
```python
    def test_priority_label_in_histogram_metrics(self):
        """Send requests with different priorities and verify that
        histogram metrics (TTFT, ITL, e2e latency) contain the priority label."""

        for priority in [1, 5]:
            response = requests.post(
                f"{DEFAULT_URL_FOR_TEST}/generate",
                json={
                    "text": "The capital of France is",
                    "sampling_params": {"temperature": 0, "max_new_tokens": 20},
                    "priority": priority,
                },
            )
            self.assertEqual(response.status_code, 200)

        metrics_response = requests.get(f"{DEFAULT_URL_FOR_TEST}/metrics")
        self.assertEqual(metrics_response.status_code, 200)
        metrics = _parse_prometheus_metrics(metrics_response.text)

        # Check histogram metrics have priority label with per-priority breakdown
        histogram_metrics = [
            "sglang:time_to_first_token_seconds",
            "sglang:e2e_request_latency_seconds",
        ]
        for metric_name in histogram_metrics:
            # Histogram metrics are emitted as _sum, _count, _bucket
            count_name = f"{metric_name}_count"
            samples = _get_samples_by_name(metrics, count_name)
            self.assertGreater(len(samples), 0, f"No samples found for {count_name}")
            # At least one sample should have a non-empty priority label
            priority_values = {s.labels.get("priority", "") for s in samples}
            non_empty = priority_values - {""}
            self.assertGreater(
                len(non_empty),
                0,
                f"{count_name}: expected per-priority samples, "
                f"got priority labels: {priority_values}",
            )
            # Verify that both priority="1" and priority="5" have count > 0
            for expected_priority in ["1", "5"]:
                matching = [
                    s for s in samples if s.labels.get("priority") == expected_priority
                ]
                self.assertGreater(
                    len(matching),
                    0,
                    f"{count_name}: no sample with priority='{expected_priority}'",
                )
                self.assertGreater(
                    matching[0].value,
                    0,
                    f"{count_name}: priority='{expected_priority}' count should be > 0",
                )
```
**EN:** Send requests with different priorities and verify that histogram metrics (TTFT, ITL, e2e latency) contain the priority label. This test exercises `test_priority_label_in_histogram_metrics` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Send requests with different priorities and verify that histogram metrics (TTFT, ITL, e2e latency) contain the priority label. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_label_in_histogram_metrics`。

### Lines 192-218: test case default priority value / 测试用例 default priority value
```python
    def test_default_priority_value(self):
        """Requests without explicit priority should use --default-priority-value (0)."""

        # Send request WITHOUT priority — should get default priority 0
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/generate",
            json={
                "text": "Hello world",
                "sampling_params": {"temperature": 0, "max_new_tokens": 5},
            },
        )
        self.assertEqual(response.status_code, 200)

        metrics_response = requests.get(f"{DEFAULT_URL_FOR_TEST}/metrics")
        self.assertEqual(metrics_response.status_code, 200)
        metrics = _parse_prometheus_metrics(metrics_response.text)

        # Check that e2e latency has samples with priority="0" (the default)
        e2e_count = _get_samples_by_name(
            metrics, "sglang:e2e_request_latency_seconds_count"
        )
        priority_values = {s.labels.get("priority", "") for s in e2e_count}
        self.assertIn(
            "0",
            priority_values,
            f"Expected priority='0' from default, got: {priority_values}",
        )
```
**EN:** Requests without explicit priority should use --default-priority-value (0). This test exercises `test_default_priority_value` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Requests without explicit priority should use --default-priority-value (0). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_priority_value`。

### Lines 221-222: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_parse_prometheus_metrics`: This block implements `_parse_prometheus_metrics` and captures one focused piece of the module's behavior. / 该代码块实现 `_parse_prometheus_metrics`，承担模块行为中的一个聚焦逻辑片段。
- `_get_samples_by_name`: This block implements `_get_samples_by_name` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_samples_by_name`，承担模块行为中的一个聚焦逻辑片段。
- `_get_sample_value_by_labels`: This block implements `_get_sample_value_by_labels` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_sample_value_by_labels`，承担模块行为中的一个聚焦逻辑片段。
- `TestQueueCount`: Unit tests for QueueCount (no server needed). / 用于组织相关测试、夹具或辅助方法。
- `TestPriorityMetrics`: Test that priority-based metrics are correctly emitted when --enable-priority-scheduling is enabled. / 用于组织相关测试、夹具或辅助方法。
- `TestQueueCount.test_queue_count_from_reqs`: QueueCount correctly counts per-priority breakdown. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_queue_count_from_reqs`。
- `TestQueueCount.test_queue_count_from_reqs_disabled`: Priority scheduling disabled → no breakdown. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_queue_count_from_reqs_disabled`。
- `TestQueueCount.test_queue_count_empty`: Empty request list. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_queue_count_empty`。
- `TestPriorityMetrics.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestPriorityMetrics.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestPriorityMetrics.test_priority_label_in_gauge_metrics`: Send requests with different priorities and verify that gauge metrics (num_running_reqs, num_queue_reqs) contain the priority label dimension. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_label_in_gauge_metrics`。
- `TestPriorityMetrics.test_priority_label_in_histogram_metrics`: Send requests with different priorities and verify that histogram metrics (TTFT, ITL, e2e latency) contain the priority label. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_label_in_histogram_metrics`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `typing`, `unittest.mock`
- **Third-party modules / 第三方模块**: `requests`, `prometheus_client.parser`, `prometheus_client.samples`
- **Internal modules / 内部模块**: `sglang.srt.observability.metrics_collector`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 222
