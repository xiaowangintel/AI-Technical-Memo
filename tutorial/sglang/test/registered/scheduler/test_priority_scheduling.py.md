# test_priority_scheduling.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/scheduler/test_priority_scheduling.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates priority scheduling behavior in SGLang's scheduler area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 scheduler 领域中与 priority scheduling 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and dependencies / 模块导入与依赖
```python
import asyncio
import os
import re
import unittest
from typing import Any, List, Optional, Tuple

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    STDERR_FILENAME,
    STDOUT_FILENAME,
    CustomTestCase,
    popen_launch_server,
    send_concurrent_generate_requests_with_custom_params,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `asyncio`, `os`, `re`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `asyncio`, `os`, `re`, `unittest`。

### Lines 20-21: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=149, stage="extra-a", runner_config="1-gpu-small")
register_amd_ci(est_time=195, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 24-24: class TestPriorityScheduling declaration / 类 TestPriorityScheduling 声明
```python
class TestPriorityScheduling(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 25-46: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST

        cls.stdout = open(STDOUT_FILENAME, "w")
        cls.stderr = open(STDERR_FILENAME, "w")

        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=(
                "--max-running-requests",  # Enforce max request concurrency is 1
                "1",
                "--max-queued-requests",  # Enforce max queued request number is 3
                "3",
                "--enable-priority-scheduling",  # Enable priority scheduling
            ),
            return_stdout_stderr=(cls.stdout, cls.stderr),
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 48-55: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        _verify_max_running_requests_and_max_queued_request_validation(1, 3)
        cls.stdout.close()
        cls.stderr.close()
        os.remove(STDOUT_FILENAME)
        os.remove(STDERR_FILENAME)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 57-86: test case priority scheduling request ordering validation / 测试用例 priority scheduling request ordering validation
```python
    def test_priority_scheduling_request_ordering_validation(self):
        """Verify pending requests are ordered by priority and received timestamp."""

        responses = asyncio.run(
            send_concurrent_generate_requests_with_custom_params(
                self.base_url,
                [
                    {
                        "priority": 0,
                        "sampling_params": {"max_new_tokens": 10000},
                    },  # starts being processed first
                    {"priority": 1},  # third
                    {"priority": 1},  # fourth
                    {"priority": 2},  # second
                ],
            )
        )

        expected_status_and_error_messages = [
            (200, None),
            (200, None),
            (200, None),
            (200, None),
        ]

        e2e_latencies = []
        _verify_genereate_responses(
            responses, expected_status_and_error_messages, e2e_latencies
        )
        assert e2e_latencies[0] < e2e_latencies[3] < e2e_latencies[1] < e2e_latencies[2]
```
**EN:** Verify pending requests are ordered by priority and received timestamp. This test exercises `test_priority_scheduling_request_ordering_validation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify pending requests are ordered by priority and received timestamp. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_request_ordering_validation`。

### Lines 88-123: test case priority scheduling existing requests abortion validation / 测试用例 priority scheduling existing requests abortion validation
```python
    def test_priority_scheduling_existing_requests_abortion_validation(self):
        """Verify lower priority requests are aborted when incoming requests have higher priority"""

        responses = asyncio.run(
            send_concurrent_generate_requests_with_custom_params(
                self.base_url,
                [
                    {
                        "priority": 1,
                        "sampling_params": {"max_new_tokens": 10000},
                    },  # starts being processed first and holds the running queue capacity
                    {"priority": 2},  # aborted by request 5
                    {"priority": 3},  # aborted by request 6
                    {"priority": 4},  # aborted by request 7
                    {"priority": 5},  # fourth
                    {"priority": 6},  # third
                    {"priority": 7},  # second
                ],
            )
        )

        expected_status_and_error_messages = [
            (200, None),
            (503, "The request is aborted by a higher priority request."),
            (503, "The request is aborted by a higher priority request."),
            (503, "The request is aborted by a higher priority request."),
            (200, None),
            (200, None),
            (200, None),
        ]

        e2e_latencies = []
        _verify_genereate_responses(
            responses, expected_status_and_error_messages, e2e_latencies
        )
        assert e2e_latencies[0] < e2e_latencies[6] < e2e_latencies[5] < e2e_latencies[4]
```
**EN:** Verify lower priority requests are aborted when incoming requests have higher priority This test exercises `test_priority_scheduling_existing_requests_abortion_validation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify lower priority requests are aborted when incoming requests have higher priority 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_existing_requests_abortion_validation`。

### Lines 125-160: test case priority scheduling incoming request rejection validation / 测试用例 priority scheduling incoming request rejection validation
```python
    def test_priority_scheduling_incoming_request_rejection_validation(self):
        """Verify incoming requests are rejected when existing requests have higher priority"""

        responses = asyncio.run(
            send_concurrent_generate_requests_with_custom_params(
                self.base_url,
                [
                    {
                        "priority": 7,
                        "sampling_params": {"max_new_tokens": 10000},
                    },  # starts being processed first and holds the running queue capacity
                    {"priority": 6},  # second
                    {"priority": 5},  # third
                    {"priority": 4},  # fourth
                    {"priority": 3},  # rejected
                    {"priority": 2},  # rejected
                    {"priority": 1},  # rejected
                ],
            )
        )

        expected_status_and_error_messages = [
            (200, None),
            (200, None),
            (200, None),
            (200, None),
            (503, "The request queue is full."),
            (503, "The request queue is full."),
            (503, "The request queue is full."),
        ]

        e2e_latencies = []
        _verify_genereate_responses(
            responses, expected_status_and_error_messages, e2e_latencies
        )
        assert e2e_latencies[0] < e2e_latencies[1] < e2e_latencies[2] < e2e_latencies[3]
```
**EN:** Verify incoming requests are rejected when existing requests have higher priority This test exercises `test_priority_scheduling_incoming_request_rejection_validation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify incoming requests are rejected when existing requests have higher priority 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_incoming_request_rejection_validation`。

### Lines 162-196: test case priority scheduling preemption meeting threshold validation / 测试用例 priority scheduling preemption meeting threshold validation
```python
    def test_priority_scheduling_preemption_meeting_threshold_validation(self):
        """Verify running requests are preempted by requests with priorities meeting the preemption threshold"""

        responses = asyncio.run(
            send_concurrent_generate_requests_with_custom_params(
                self.base_url,
                [
                    {
                        "priority": 0,
                        "sampling_params": {"max_new_tokens": 10000},
                    },  # starts being processed first then preempted or pushed by later requests, and finishes last.
                    {
                        "priority": 10,
                        "sampling_params": {"max_new_tokens": 10000},
                    },  # scheduled after the third request, and finishes second.
                    {
                        "priority": 20,
                        "sampling_params": {"max_new_tokens": 10000},
                    },  # finishes first.
                ],
            )
        )

        expected_status_and_error_messages = [
            (200, None),
            (200, None),
            (200, None),
        ]

        e2e_latencies = []
        _verify_genereate_responses(
            responses, expected_status_and_error_messages, e2e_latencies
        )

        assert e2e_latencies[2] < e2e_latencies[1] < e2e_latencies[0]
```
**EN:** Verify running requests are preempted by requests with priorities meeting the preemption threshold This test exercises `test_priority_scheduling_preemption_meeting_threshold_validation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify running requests are preempted by requests with priorities meeting the preemption threshold 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_preemption_meeting_threshold_validation`。

### Lines 198-227: test case priority scheduling preemption below threshold validation / 测试用例 priority scheduling preemption below threshold validation
```python
    def test_priority_scheduling_preemption_below_threshold_validation(self):
        """Verify running requests are not preempted by requests with priorities below preemption threshold"""

        responses = asyncio.run(
            send_concurrent_generate_requests_with_custom_params(
                self.base_url,
                [
                    {
                        "priority": 0,
                        "sampling_params": {"max_new_tokens": 10000},
                    },
                    {
                        "priority": 5,
                        "sampling_params": {"max_new_tokens": 10000},
                    },
                ],
            )
        )

        expected_status_and_error_messages = [
            (200, None),
            (200, None),
        ]

        e2e_latencies = []
        _verify_genereate_responses(
            responses, expected_status_and_error_messages, e2e_latencies
        )

        assert e2e_latencies[0] < e2e_latencies[1]
```
**EN:** Verify running requests are not preempted by requests with priorities below preemption threshold This test exercises `test_priority_scheduling_preemption_below_threshold_validation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify running requests are not preempted by requests with priorities below preemption threshold 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_preemption_below_threshold_validation`。

### Lines 230-230: class TestPrioritySchedulingMultipleRunningRequests declaration / 类 TestPrioritySchedulingMultipleRunningRequests 声明
```python
class TestPrioritySchedulingMultipleRunningRequests(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 231-252: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST

        cls.stdout = open(STDOUT_FILENAME, "w")
        cls.stderr = open(STDERR_FILENAME, "w")

        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=(
                "--max-running-requests",  # Enforce max request concurrency is 2
                "2",
                "--max-queued-requests",  # Enforce max queued request number is 3
                "3",
                "--enable-priority-scheduling",  # Enable priority scheduling
            ),
            return_stdout_stderr=(cls.stdout, cls.stderr),
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 254-261: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        _verify_max_running_requests_and_max_queued_request_validation(2, 3)
        cls.stdout.close()
        cls.stderr.close()
        os.remove(STDOUT_FILENAME)
        os.remove(STDERR_FILENAME)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 263-293: test case priority scheduling with multiple running requests preemption / 测试用例 priority scheduling with multiple running requests preemption
```python
    def test_priority_scheduling_with_multiple_running_requests_preemption(self):
        """Verify preempting a subset of running requests is safe."""

        responses = asyncio.run(
            send_concurrent_generate_requests_with_custom_params(
                self.base_url,
                [
                    {
                        "priority": 10,
                        "sampling_params": {"max_new_tokens": 10000},
                    },  # finishes first
                    {
                        "priority": 5,
                        "sampling_params": {"max_new_tokens": 10000},
                    },  # preempted by fourth request, then finishes third
                    {
                        "priority": 15,
                        "sampling_params": {"max_new_tokens": 10000},
                    },  # preempt the first request
                ],
            )
        )

        expected_status_and_error_messages = [
            (200, None),
            (200, None),
            (200, None),
            (200, None),
        ]

        _verify_genereate_responses(responses, expected_status_and_error_messages, [])
```
**EN:** Verify preempting a subset of running requests is safe. This test exercises `test_priority_scheduling_with_multiple_running_requests_preemption` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify preempting a subset of running requests is safe. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_with_multiple_running_requests_preemption`。

### Lines 295-366: test case priority scheduling preemption token offset calculation / 测试用例 priority scheduling preemption token offset calculation
```python
    def test_priority_scheduling_preemption_token_offset_calculation(self):
        """
        Verify correct token offset calculation during preemption.

        This test specifically targets the bug where rem_total_token_offset was incorrectly
        calculated using the incoming request's tokens instead of the preempted request's tokens
        (related to issue #13111 and PR #13201).

        THE BUG:
        In schedule_policy.py line 700, the code was using:
            self.rem_total_token_offset -= self._get_running_request_total_token_offset(req)
        Instead of:
            self.rem_total_token_offset -= self._get_running_request_total_token_offset(running_req)

        WHY THIS TEST CATCHES THE BUG:
        - Request 1 (preempted): 8000 tokens - This is what SHOULD be freed
        - Request 3 (incoming):  1000 tokens - This is what WAS freed (bug)
        - Token difference: 8000 - 1000 = 7000 tokens incorrectly accounted

        With the bug, the system thinks it only freed 1000 tokens instead of 8000 tokens.
        This causes incorrect memory accounting and can lead to:
        1. Scheduler believes less memory is available than actually is
        2. Subsequent requests (like Request 4) may fail to schedule or cause issues
        3. Memory calculations become increasingly inaccurate with each preemption

        The test creates a scenario where:
        1. A low-priority request with many tokens (8000) starts running
        2. A high-priority request with few tokens (1000) arrives and triggers preemption
        3. The system must correctly free 8000 tokens from the preempted request
        4. Additional requests can be scheduled only if tokens were correctly freed
        5. Execution order validates priority-based scheduling works correctly

        The large token difference (8x) makes the bug's impact obvious and testable.
        """
        responses = asyncio.run(
            send_concurrent_generate_requests_with_custom_params(
                self.base_url,
                [
                    {
                        "priority": 0,
                        "sampling_params": {"max_new_tokens": 8000},
                    },  # Low priority, large token count - will be preempted
                    {
                        "priority": 1,
                        "sampling_params": {"max_new_tokens": 5000},
                    },  # Medium priority, medium token count - queued initially
                    {
                        "priority": 100,
                        "sampling_params": {"max_new_tokens": 1000},
                    },  # High priority, small token count - triggers preemption
                    {
                        "priority": 50,
                        "sampling_params": {"max_new_tokens": 2000},
                    },  # Should be schedulable after correct token accounting
                ],
            )
        )

        # All requests should complete successfully
        # The key is that the fourth request should be schedulable because
        # the system correctly freed tokens from the first (preempted) request
        expected_status_and_error_messages = [
            (200, None),
            (200, None),
            (200, None),
            (200, None),
        ]

        e2e_latencies = []
        _verify_genereate_responses(
            responses, expected_status_and_error_messages, e2e_latencies
        )
```
**EN:** Verify correct token offset calculation during preemption. This test exercises `test_priority_scheduling_preemption_token_offset_calculation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify correct token offset calculation during preemption. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_preemption_token_offset_calculation`。

### Lines 367-377: supporting source context / 辅助源码上下文
```python

        # Verify execution order: high priority requests finish before low priority ones
        # Request 3 (priority 100) should finish first
        # Request 4 (priority 50) should finish second
        # Request 2 (priority 1) should finish third
        # Request 1 (priority 0) should finish last (after being preempted)

        # FIXME(harrison lim)
        # assert e2e_latencies[2] < e2e_latencies[3] < e2e_latencies[1] < e2e_latencies[0]


```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 378-405: function verify genereate responses / 函数 verify genereate responses
```python
def _verify_genereate_responses(
    responses: Tuple[int, Any, float],
    expected_code_and_error_message: Tuple[int, Any],
    e2e_latencies: List[Optional[float]],
):
    """
    Verify generate response results are as expected based on status code and response json object content.
    In addition, collects e2e latency info to verify scheduling and processing ordering.
    """
    for got, expected in zip(responses, expected_code_and_error_message):
        got_status, got_json = got
        expected_status, expected_err_msg = expected

        # Check status code is as expected
        assert got_status == expected_status

        # Check error message content or fields' existence based on status code
        if got_status != 200:
            assert got_json["object"] == "error"
            assert got_json["message"] == expected_err_msg
        else:
            assert "object" not in got_json
            assert "message" not in got_json

        # Collect e2e latencies for scheduling validation
        e2e_latencies.append(
            got_json["meta_info"]["e2e_latency"] if got_status == 200 else None
        )
```
**EN:** Verify generate response results are as expected based on status code and response json object content. This block implements `_verify_genereate_responses` and captures one focused piece of the module's behavior.
**CN:** Verify generate response results are as expected based on status code and response json object content. 该代码块实现 `_verify_genereate_responses`，承担模块行为中的一个聚焦逻辑片段。

### Lines 408-421: function verify max running requests and max queued request validation / 函数 verify max running requests and max queued request validation
```python
def _verify_max_running_requests_and_max_queued_request_validation(
    max_running_requests: int, max_queued_requests: int
):
    """Verify running request and queued request numbers based on server logs."""
    rr_pattern = re.compile(r"#running-req:\s*(\d+)")
    qr_pattern = re.compile(r"#queue-req:\s*(\d+)")

    with open(STDERR_FILENAME) as lines:
        for line in lines:
            rr_match, qr_match = rr_pattern.search(line), qr_pattern.search(line)
            if rr_match:
                assert int(rr_match.group(1)) <= max_running_requests
            if qr_match:
                assert int(qr_match.group(1)) <= max_queued_requests
```
**EN:** Verify running request and queued request numbers based on server logs. This block implements `_verify_max_running_requests_and_max_queued_request_validation` and captures one focused piece of the module's behavior.
**CN:** Verify running request and queued request numbers based on server logs. 该代码块实现 `_verify_max_running_requests_and_max_queued_request_validation`，承担模块行为中的一个聚焦逻辑片段。

### Lines 424-425: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPriorityScheduling`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPrioritySchedulingMultipleRunningRequests`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_verify_genereate_responses`: Verify generate response results are as expected based on status code and response json object content. / 该代码块实现 `_verify_genereate_responses`，承担模块行为中的一个聚焦逻辑片段。
- `_verify_max_running_requests_and_max_queued_request_validation`: Verify running request and queued request numbers based on server logs. / 该代码块实现 `_verify_max_running_requests_and_max_queued_request_validation`，承担模块行为中的一个聚焦逻辑片段。
- `TestPriorityScheduling.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestPriorityScheduling.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestPriorityScheduling.test_priority_scheduling_request_ordering_validation`: Verify pending requests are ordered by priority and received timestamp. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_request_ordering_validation`。
- `TestPriorityScheduling.test_priority_scheduling_existing_requests_abortion_validation`: Verify lower priority requests are aborted when incoming requests have higher priority / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_existing_requests_abortion_validation`。
- `TestPriorityScheduling.test_priority_scheduling_incoming_request_rejection_validation`: Verify incoming requests are rejected when existing requests have higher priority / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_incoming_request_rejection_validation`。
- `TestPriorityScheduling.test_priority_scheduling_preemption_meeting_threshold_validation`: Verify running requests are preempted by requests with priorities meeting the preemption threshold / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_preemption_meeting_threshold_validation`。
- `TestPriorityScheduling.test_priority_scheduling_preemption_below_threshold_validation`: Verify running requests are not preempted by requests with priorities below preemption threshold / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_scheduling_preemption_below_threshold_validation`。
- `TestPrioritySchedulingMultipleRunningRequests.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `os`, `re`, `unittest`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 425
