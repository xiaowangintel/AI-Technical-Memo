# test_metrics.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/observability/test_metrics.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates metrics behavior in SGLang's observability area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 observability 领域中与 metrics 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: module imports and dependencies / 模块导入与依赖
```python
import unittest
from typing import Dict, List

import requests
from prometheus_client.parser import text_string_to_metric_families
from prometheus_client.samples import Sample

from sglang.srt.environ import envs
from sglang.srt.observability.metrics_collector import (
    ROUTING_KEY_REQ_COUNT_BUCKET_BOUNDS,
    compute_routing_key_stats,
)
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `typing`, `requests`, `prometheus_client.parser`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `typing`, `requests`, `prometheus_client.parser`。

### Lines 23-26: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=74, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=32, suite="stage-b-test-1-gpu-small-amd")

_MODEL_NAME = "Qwen/Qwen3-0.6B"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 29-29: class TestEnableMetrics declaration / 类 TestEnableMetrics 声明
```python
class TestEnableMetrics(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 30-37: test case metrics 1gpu / 测试用例 metrics 1gpu
```python
    def test_metrics_1gpu(self):
        """Test that metrics endpoint returns data when enabled"""
        self._execute_core(
            other_args=[],
            verify_metrics_extra=None,
            expect_mfu_metrics=True,
            enable_mfu_metrics=True,
        )
```
**EN:** Test that metrics endpoint returns data when enabled This test exercises `test_metrics_1gpu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that metrics endpoint returns data when enabled 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_metrics_1gpu`。

### Lines 39-46: test case mfu metrics gate disabled / 测试用例 mfu metrics gate disabled
```python
    def test_mfu_metrics_gate_disabled(self):
        """MFU metrics should not be emitted when the gate is disabled."""
        self._execute_core(
            other_args=[],
            verify_metrics_extra=None,
            expect_mfu_metrics=False,
            enable_mfu_metrics=False,
        )
```
**EN:** MFU metrics should not be emitted when the gate is disabled. This test exercises `test_mfu_metrics_gate_disabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** MFU metrics should not be emitted when the gate is disabled. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mfu_metrics_gate_disabled`。

### Lines 48-87: test case metrics 2gpu / 测试用例 metrics 2gpu
```python
    def test_metrics_2gpu(self):
        # TODO enable when we have 2-gpu runner in nightly CI
        if is_in_ci():
            print("Skip test_metrics_2gpu since in 1-gpu CI")
            return

        def _verify_metrics_extra(metrics):
            metrics_to_check = [
                (
                    "sglang:dp_cooperation_realtime_tokens_total",
                    {"mode": "prefill_compute"},
                ),
                (
                    "sglang:dp_cooperation_realtime_tokens_total",
                    {"mode": "decode"},
                ),
                (
                    "sglang:dp_cooperation_forward_execution_seconds_total",
                    {"category": "extend"},
                ),
                (
                    "sglang:dp_cooperation_forward_execution_seconds_total",
                    {"category": "decode"},
                ),
            ]
            _check_metrics_positive(self, metrics, metrics_to_check)

            num_prefill_ranks_values = {
                s.labels["num_prefill_ranks"]
                for s in metrics["sglang:dp_cooperation_realtime_tokens_total"]
            }
            self.assertIn("0", num_prefill_ranks_values)
            self.assertIn("1", num_prefill_ranks_values)

        self._execute_core(
            other_args=["--tp", "2", "--dp", "2", "--enable-dp-attention"],
            verify_metrics_extra=_verify_metrics_extra,
            expect_mfu_metrics=True,
            enable_mfu_metrics=True,
        )
```
**EN:** This test exercises `test_metrics_2gpu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_metrics_2gpu`。

### Lines 89-156: method execute core / 方法 execute core
```python
    def _execute_core(
        self,
        other_args,
        verify_metrics_extra,
        expect_mfu_metrics: bool,
        enable_mfu_metrics: bool,
    ):
        with (
            envs.SGLANG_ENABLE_METRICS_DP_ATTENTION.override(True),
            envs.SGLANG_ENABLE_METRICS_DEVICE_TIMER.override(True),
            envs.SGLANG_TEST_RETRACT.override(True),
        ):
            launch_args = ["--enable-metrics", "--cuda-graph-max-bs", 2, *other_args]
            if enable_mfu_metrics:
                launch_args.insert(1, "--enable-mfu-metrics")
            process = popen_launch_server(
                _MODEL_NAME,
                DEFAULT_URL_FOR_TEST,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=launch_args,
            )

        try:
            # Make some requests to generate some metrics
            response = requests.get(f"{DEFAULT_URL_FOR_TEST}/health_generate")
            self.assertEqual(response.status_code, 200)

            response = requests.post(
                f"{DEFAULT_URL_FOR_TEST}/generate",
                json={
                    "text": ["The capital of France is"] * 20,
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": 50,
                    },
                    "stream": True,
                    "ignore_eos": True,
                },
                stream=True,
            )
            for _ in response.iter_lines(decode_unicode=False):
                pass

            for i in range(2):
                # Send the request twice to trigger cached token metrics
                response = requests.post(
                    f"{DEFAULT_URL_FOR_TEST}/generate",
                    json={
                        "text": "Hello, " * 100,
                        "sampling_params": {"temperature": 0, "max_new_tokens": 5},
                    },
                    headers={"x-smg-routing-key": "test-key"},
                )
                self.assertEqual(response.status_code, 200)

            # Get metrics
            metrics_response = requests.get(f"{DEFAULT_URL_FOR_TEST}/metrics")
            self.assertEqual(metrics_response.status_code, 200)
            metrics_text = metrics_response.text

            print(f"metrics_text=\n{metrics_text}")

            metrics = _parse_prometheus_metrics(metrics_text)
            self._verify_metrics_common(metrics_text, metrics, expect_mfu_metrics)
            if verify_metrics_extra is not None:
                verify_metrics_extra(metrics)
        finally:
            kill_process_tree(process.pid)
```
**EN:** This block implements `_execute_core` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_execute_core`，承担模块行为中的一个聚焦逻辑片段。

### Lines 158-237: method verify metrics common (part 1/2) / 方法 verify metrics common（第 1/2 部分）
```python
    def _verify_metrics_common(self, metrics_text, metrics, expect_mfu_metrics: bool):
        essential_metrics = [
            "sglang:num_running_reqs",
            "sglang:num_used_tokens",
            "sglang:token_usage",
            "sglang:gen_throughput",
            "sglang:num_queue_reqs",
            "sglang:num_grammar_queue_reqs",
            "sglang:cache_hit_rate",
            "sglang:spec_accept_length",
            "sglang:prompt_tokens_total",
            "sglang:generation_tokens_total",
            "sglang:cached_tokens_total",
            "sglang:num_requests_total",
            "sglang:time_to_first_token_seconds",
            "sglang:inter_token_latency_seconds",
            "sglang:e2e_request_latency_seconds",
            "sglang:http_requests_active",
            "sglang:routing_keys_active",
            "sglang:num_unique_running_routing_keys",
            "sglang:routing_key_running_req_count",
            "sglang:routing_key_all_req_count",
        ]
        mfu_metrics = [
            "sglang:estimated_flops_per_gpu_total",
            "sglang:estimated_read_bytes_per_gpu_total",
            "sglang:estimated_write_bytes_per_gpu_total",
        ]
        if expect_mfu_metrics:
            essential_metrics.extend(mfu_metrics)
        for metric in essential_metrics:
            self.assertIn(metric, metrics_text, f"Missing metric: {metric}")

        # Verify routing key GaugeHistogram buckets
        expected_buckets = len(ROUTING_KEY_REQ_COUNT_BUCKET_BOUNDS) + 1
        for metric_name in [
            "sglang:routing_key_running_req_count",
            "sglang:routing_key_all_req_count",
        ]:
            gt_le_pairs = set()
            for sample in metrics.get(metric_name, []):
                gt_le_pairs.add((sample.labels.get("gt"), sample.labels.get("le")))
            self.assertEqual(
                len(gt_le_pairs),
                expected_buckets,
                f"{metric_name}: Expected {expected_buckets} buckets, got {len(gt_le_pairs)}",
            )

        self.assertIn(f'model_name="{_MODEL_NAME}"', metrics_text)
        self.assertIn("_sum{", metrics_text)
        self.assertIn("_count{", metrics_text)
        self.assertIn("_bucket{", metrics_text)

        metrics_to_check = [
            ("sglang:realtime_tokens_total", {"mode": "prefill_compute"}),
            ("sglang:realtime_tokens_total", {"mode": "decode"}),
            ("sglang:forward_execution_seconds_total", {"category": "extend"}),
            ("sglang:forward_execution_seconds_total", {"category": "decode"}),
            ("sglang:process_cpu_seconds_total", {"component": "tokenizer"}),
        ]
        _check_metrics_positive(self, metrics, metrics_to_check)

        if expect_mfu_metrics:
            # Estimated perf metrics may have multiple series (e.g., by rank). Ensure
            # that at least one series for this model has a positive accumulated value.
            for metric_name in mfu_metrics:
                values = [
                    sample.value
                    for sample in metrics.get(metric_name, [])
                    if sample.labels.get("model_name") == _MODEL_NAME
                ]
                self.assertTrue(
                    values, f"{metric_name}: no samples for model {_MODEL_NAME}"
                )
                self.assertGreater(
                    sum(values),
                    0,
                    f"{metric_name}: expected positive total for model {_MODEL_NAME}",
                )
        else:
```
**EN:** This block implements `_verify_metrics_common` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `_verify_metrics_common`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 238-251: method verify metrics common (part 2/2) / 方法 verify metrics common（第 2/2 部分）
```python
            # With only --enable-metrics (without --enable-mfu-metrics), MFU
            # counters should not emit positive values.
            for metric_name in mfu_metrics:
                values = [
                    sample.value
                    for sample in metrics.get(metric_name, [])
                    if sample.labels.get("model_name") == _MODEL_NAME
                ]
                if values:
                    self.assertEqual(
                        sum(values),
                        0,
                        f"{metric_name}: expected no positive samples with MFU metrics gate disabled",
                    )
```
**EN:** This block implements `_verify_metrics_common` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `_verify_metrics_common`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 254-261: function parse prometheus metrics / 函数 parse prometheus metrics
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

### Lines 264-268: function get sample value by labels / 函数 get sample value by labels
```python
def _get_sample_value_by_labels(samples: List[Sample], labels: Dict[str, str]) -> float:
    for sample in samples:
        if all(sample.labels.get(k) == v for k, v in labels.items()):
            return sample.value
    raise KeyError(f"No sample found with labels {labels}")
```
**EN:** This block implements `_get_sample_value_by_labels` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_sample_value_by_labels`，承担模块行为中的一个聚焦逻辑片段。

### Lines 271-274: function check metrics positive / 函数 check metrics positive
```python
def _check_metrics_positive(test_case, metrics, metrics_to_check):
    for metric_name, labels in metrics_to_check:
        value = _get_sample_value_by_labels(metrics[metric_name], labels)
        test_case.assertGreater(value, 0, f"{metric_name} {labels}")
```
**EN:** This block implements `_check_metrics_positive` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_check_metrics_positive`，承担模块行为中的一个聚焦逻辑片段。

### Lines 277-277: class TestComputeRoutingKeyStats declaration / 类 TestComputeRoutingKeyStats 声明
```python
class TestComputeRoutingKeyStats(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 278-281: test case empty / 测试用例 empty
```python
    def test_empty(self):
        num_unique, req_counts = compute_routing_key_stats([])
        self.assertEqual(num_unique, 0)
        self.assertEqual(req_counts, [])
```
**EN:** This test exercises `test_empty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty`。

### Lines 283-286: test case all none / 测试用例 all none
```python
    def test_all_none(self):
        num_unique, req_counts = compute_routing_key_stats([None, None, None])
        self.assertEqual(num_unique, 0)
        self.assertEqual(req_counts, [])
```
**EN:** This test exercises `test_all_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_none`。

### Lines 288-291: test case with none / 测试用例 with none
```python
    def test_with_none(self):
        num_unique, req_counts = compute_routing_key_stats([None, "key1", None])
        self.assertEqual(num_unique, 1)
        self.assertEqual(req_counts, [1])
```
**EN:** This test exercises `test_with_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_with_none`。

### Lines 293-296: test case single key multiple reqs / 测试用例 single key multiple reqs
```python
    def test_single_key_multiple_reqs(self):
        num_unique, req_counts = compute_routing_key_stats(["key1"] * 5)
        self.assertEqual(num_unique, 1)
        self.assertEqual(req_counts, [5])
```
**EN:** This test exercises `test_single_key_multiple_reqs` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_key_multiple_reqs`。

### Lines 298-302: test case distribution / 测试用例 distribution
```python
    def test_distribution(self):
        routing_keys = ["key1"] * 5 + ["key2"] * 1 + ["key3"] * 15 + ["key4"] * 250
        num_unique, req_counts = compute_routing_key_stats(routing_keys)
        self.assertEqual(num_unique, 4)
        self.assertEqual(sorted(req_counts), [1, 5, 15, 250])
```
**EN:** This test exercises `test_distribution` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_distribution`。

### Lines 305-306: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestEnableMetrics`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_parse_prometheus_metrics`: This block implements `_parse_prometheus_metrics` and captures one focused piece of the module's behavior. / 该代码块实现 `_parse_prometheus_metrics`，承担模块行为中的一个聚焦逻辑片段。
- `_get_sample_value_by_labels`: This block implements `_get_sample_value_by_labels` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_sample_value_by_labels`，承担模块行为中的一个聚焦逻辑片段。
- `_check_metrics_positive`: This block implements `_check_metrics_positive` and captures one focused piece of the module's behavior. / 该代码块实现 `_check_metrics_positive`，承担模块行为中的一个聚焦逻辑片段。
- `TestComputeRoutingKeyStats`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEnableMetrics.test_metrics_1gpu`: Test that metrics endpoint returns data when enabled / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_metrics_1gpu`。
- `TestEnableMetrics.test_mfu_metrics_gate_disabled`: MFU metrics should not be emitted when the gate is disabled. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mfu_metrics_gate_disabled`。
- `TestEnableMetrics.test_metrics_2gpu`: This test exercises `test_metrics_2gpu` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_metrics_2gpu`。
- `TestEnableMetrics._execute_core`: This block implements `_execute_core` and captures one focused piece of the module's behavior. / 该代码块实现 `_execute_core`，承担模块行为中的一个聚焦逻辑片段。
- `TestEnableMetrics._verify_metrics_common`: This block implements `_verify_metrics_common` and captures one focused piece of the module's behavior. / 该代码块实现 `_verify_metrics_common`，承担模块行为中的一个聚焦逻辑片段。
- `TestComputeRoutingKeyStats.test_empty`: This test exercises `test_empty` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty`。
- `TestComputeRoutingKeyStats.test_all_none`: This test exercises `test_all_none` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_none`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `typing`
- **Third-party modules / 第三方模块**: `requests`, `prometheus_client.parser`, `prometheus_client.samples`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.observability.metrics_collector`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 306
