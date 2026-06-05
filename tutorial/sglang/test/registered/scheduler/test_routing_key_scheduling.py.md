# test_routing_key_scheduling.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/scheduler/test_routing_key_scheduling.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates routing key scheduling behavior in SGLang's scheduler area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 scheduler 领域中与 routing key scheduling 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: module imports and dependencies / 模块导入与依赖
```python
import asyncio
import os
import time
import unittest

import aiohttp

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    STDERR_FILENAME,
    STDOUT_FILENAME,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `asyncio`, `os`, `time`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `asyncio`, `os`, `time`, `unittest`。

### Lines 19-20: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=120, suite="nightly-1-gpu", nightly=True)
register_amd_ci(est_time=120, suite="nightly-amd-1-gpu", nightly=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 23-23: class TestRoutingKeyScheduling declaration / 类 TestRoutingKeyScheduling 声明
```python
class TestRoutingKeyScheduling(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 24-45: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        os.environ["SGLANG_ROUTING_KEY_POLICY_DEBUG_LOG"] = "1"

        cls.model = "Qwen/Qwen3-0.6B"
        cls.base_url = DEFAULT_URL_FOR_TEST

        cls.stdout = open(STDOUT_FILENAME, "w")
        cls.stderr = open(STDERR_FILENAME, "w")

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=(
                "--max-running-requests",
                "3",
                "--schedule-policy",
                "routing-key",
            ),
            return_stdout_stderr=(cls.stdout, cls.stderr),
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 47-53: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        cls.stdout.close()
        cls.stderr.close()
        os.remove(STDOUT_FILENAME)
        os.remove(STDERR_FILENAME)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 55-65: test case routing key scheduling order / 测试用例 routing key scheduling order
```python
    def test_routing_key_scheduling_order(self):
        """Verify requests with matching routing keys are prioritized.

        Test strategy:
        1. First send 2 long-running key_a requests to occupy running batch
        2. Then send 10 key_a and 10 key_b short requests concurrently
        3. With max_running_requests=3, key_a requests should be prioritized
           because running batch has 2 key_a requests
        4. Verify key_a requests finish before key_b requests on average
        """
        asyncio.run(self._test_routing_key_scheduling_order())
```
**EN:** Verify requests with matching routing keys are prioritized. This test exercises `test_routing_key_scheduling_order` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify requests with matching routing keys are prioritized. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_routing_key_scheduling_order`。

### Lines 67-100: method test routing key scheduling order / 方法 test routing key scheduling order
```python
    async def _test_routing_key_scheduling_order(self):
        long_running_tasks = [
            asyncio.create_task(self._send_chat_request("key_a", 20000)),
            asyncio.create_task(self._send_chat_request("key_a", 20000)),
        ]

        await asyncio.sleep(2.0)

        short_tasks = []
        for _ in range(10):
            short_tasks.append(
                asyncio.create_task(self._send_chat_request("key_a", 10))
            )
            short_tasks.append(
                asyncio.create_task(self._send_chat_request("key_b", 10))
            )

        all_short_results = await asyncio.gather(*short_tasks)
        await asyncio.gather(*long_running_tasks)

        key_a_latencies = [lat for key, lat in all_short_results if key == "key_a"]
        key_b_latencies = [lat for key, lat in all_short_results if key == "key_b"]

        avg_key_a = sum(key_a_latencies) / len(key_a_latencies)
        avg_key_b = sum(key_b_latencies) / len(key_b_latencies)

        print(f"Average key_a latency: {avg_key_a:.3f}s")
        print(f"Average key_b latency: {avg_key_b:.3f}s")

        self.assertLess(
            avg_key_a,
            avg_key_b,
            f"key_a requests (avg={avg_key_a:.3f}s) should finish before key_b (avg={avg_key_b:.3f}s)",
        )
```
**EN:** This block implements `_test_routing_key_scheduling_order` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_test_routing_key_scheduling_order`，承担模块行为中的一个聚焦逻辑片段。

### Lines 102-119: method send chat request / 方法 send chat request
```python
    async def _send_chat_request(self, routing_key: str, max_tokens: int):
        payload = {
            "model": self.model,
            "messages": [{"role": "user", "content": "What is 1+1?"}],
            "max_tokens": max_tokens,
            "temperature": 0,
        }
        headers = {"x-smg-routing-key": routing_key}
        start_time = time.perf_counter()
        async with aiohttp.ClientSession() as session:
            async with session.post(
                f"{self.base_url}/v1/chat/completions",
                json=payload,
                headers=headers,
            ) as resp:
                await resp.json()
        latency = time.perf_counter() - start_time
        return routing_key, latency
```
**EN:** This block implements `_send_chat_request` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_send_chat_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 122-123: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestRoutingKeyScheduling`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRoutingKeyScheduling.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestRoutingKeyScheduling.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestRoutingKeyScheduling.test_routing_key_scheduling_order`: Verify requests with matching routing keys are prioritized. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_routing_key_scheduling_order`。
- `TestRoutingKeyScheduling._test_routing_key_scheduling_order`: This block implements `_test_routing_key_scheduling_order` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_routing_key_scheduling_order`，承担模块行为中的一个聚焦逻辑片段。
- `TestRoutingKeyScheduling._send_chat_request`: This block implements `_send_chat_request` and captures one focused piece of the module's behavior. / 该代码块实现 `_send_chat_request`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `os`, `time`, `unittest`
- **Third-party modules / 第三方模块**: `aiohttp`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 123
