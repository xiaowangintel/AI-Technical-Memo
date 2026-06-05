# test_scheduler_control.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/scheduler/test_scheduler_control.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates scheduler control behavior in SGLang's scheduler area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 scheduler 领域中与 scheduler control 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing
import threading
import time
import unittest
from concurrent.futures import ThreadPoolExecutor, as_completed

import requests

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.kits.abort_timeout_kit import AbortAllMixin, WaitingTimeoutMixin
from sglang.test.kits.pause_generation_kit import PauseResumeInPlaceMixin
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
    run_and_check_memory_leak,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `threading`, `time`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `threading`, `time`, `unittest`。

### Lines 23-24: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=367, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=300, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 27-27: class TestAbort declaration / 类 TestAbort 声明
```python
class TestAbort(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 28-56: method workload func / 方法 workload func
```python
    def workload_func(self, base_url, model):
        def process_func():
            def run_one(_):
                prompt = """
                System: You are a helpful assistant.
                User: What is the capital of France?
                Assistant: The capital of France is
                """

                response = requests.post(
                    f"{base_url}/generate",
                    json={
                        "text": prompt,
                        "sampling_params": {
                            "temperature": 0,
                            "max_new_tokens": 2048,
                        },
                    },
                )
                ret = response.json()

            with ThreadPoolExecutor(16) as executor:
                list(executor.map(run_one, list(range(16))))

        p = multiprocessing.Process(target=process_func)
        p.start()
        time.sleep(0.5)
        p.terminate()
        time.sleep(10)
```
**EN:** This block implements `workload_func` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `workload_func`，承担模块行为中的一个聚焦逻辑片段。

### Lines 58-66: test case memory leak / 测试用例 memory leak
```python
    def test_memory_leak(self):
        run_and_check_memory_leak(
            self.workload_func,
            disable_radix_cache=False,
            enable_mixed_chunk=False,
            disable_overlap=False,
            chunked_prefill_size=8192,
            assert_has_abort=True,
        )
```
**EN:** This test exercises `test_memory_leak` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_memory_leak`。

### Lines 69-69: class TestAbortWithApiKey declaration / 类 TestAbortWithApiKey 声明
```python
class TestAbortWithApiKey(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 70-99: method workload func / 方法 workload func
```python
    def workload_func(self, base_url, model, api_key: str):
        def process_func():
            def run_one(_):
                prompt = """
                System: You are a helpful assistant.
                User: What is the capital of France?
                Assistant: The capital of France is
                """

                response = requests.post(
                    f"{base_url}/generate",
                    json={
                        "text": prompt,
                        "sampling_params": {
                            "temperature": 0,
                            "max_new_tokens": 2048,
                        },
                    },
                    headers={"Authorization": f"Bearer {api_key}"},
                )
                response.json()

            with ThreadPoolExecutor(16) as executor:
                list(executor.map(run_one, list(range(16))))

        p = multiprocessing.Process(target=process_func)
        p.start()
        time.sleep(0.5)
        p.terminate()
        time.sleep(10)
```
**EN:** This block implements `workload_func` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `workload_func`，承担模块行为中的一个聚焦逻辑片段。

### Lines 101-111: test case memory leak with api key / 测试用例 memory leak with api key
```python
    def test_memory_leak_with_api_key(self):
        api_key = "test-api-key"
        run_and_check_memory_leak(
            lambda base_url, model: self.workload_func(base_url, model, api_key),
            disable_radix_cache=False,
            enable_mixed_chunk=False,
            disable_overlap=False,
            chunked_prefill_size=8192,
            assert_has_abort=True,
            api_key=api_key,
        )
```
**EN:** This test exercises `test_memory_leak_with_api_key` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_memory_leak_with_api_key`。

### Lines 114-114: class TestSchedulerControl declaration / 类 TestSchedulerControl 声明
```python
class TestSchedulerControl(AbortAllMixin, PauseResumeInPlaceMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `AbortAllMixin`, `PauseResumeInPlaceMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `AbortAllMixin`, `PauseResumeInPlaceMixin`, `CustomTestCase`。

### Lines 115-124: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--max-running-requests", 8],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 126-128: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 130-142: method generate with rid / 方法 generate with rid
```python
    def _generate_with_rid(self, rid, max_new_tokens=8):
        return requests.post(
            f"{self.base_url}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": max_new_tokens,
                },
                "rid": rid,
            },
            timeout=30,
        )
```
**EN:** This block implements `_generate_with_rid` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_generate_with_rid`，承担模块行为中的一个聚焦逻辑片段。

### Lines 144-152: test case duplicate rid sequential ok / 测试用例 duplicate rid sequential ok
```python
    def test_duplicate_rid_sequential_ok(self):
        rid = "dup-rid-test-sequential"
        resp1 = self._generate_with_rid(rid)
        self.assertEqual(resp1.status_code, 200)
        self.assertNotIn("error", resp1.json())

        resp2 = self._generate_with_rid(rid)
        self.assertEqual(resp2.status_code, 200)
        self.assertNotIn("error", resp2.json())
```
**EN:** This test exercises `test_duplicate_rid_sequential_ok` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_duplicate_rid_sequential_ok`。

### Lines 154-176: test case duplicate rid concurrent rejected / 测试用例 duplicate rid concurrent rejected
```python
    def test_duplicate_rid_concurrent_rejected(self):
        rid = "dup-rid-test-concurrent"
        results = {}

        def send(key, max_tokens):
            results[key] = self._generate_with_rid(rid, max_new_tokens=max_tokens)

        t1 = threading.Thread(target=send, args=("first", 512))
        t2 = threading.Thread(target=send, args=("second", 8))
        t1.start()
        time.sleep(0.1)
        t2.start()
        t1.join(timeout=30)
        t2.join(timeout=30)

        r1, r2 = results["first"], results["second"]
        self.assertTrue(
            r1.status_code == 400 or r2.status_code == 400,
            "One of the concurrent duplicate-rid requests should be rejected",
        )

        rejected = r2 if r2.status_code == 400 else r1
        self.assertIn("Duplicate request ID", rejected.json()["error"]["message"])
```
**EN:** This test exercises `test_duplicate_rid_concurrent_rejected` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_duplicate_rid_concurrent_rejected`。

### Lines 178-190: test case duplicate rid in batch / 测试用例 duplicate rid in batch
```python
    def test_duplicate_rid_in_batch(self):
        rid = "dup-rid-batch"
        response = requests.post(
            f"{self.base_url}/generate",
            json={
                "text": ["Hello", "World"],
                "sampling_params": {"temperature": 0, "max_new_tokens": 8},
                "rid": [rid, rid],
            },
            timeout=30,
        )
        self.assertEqual(response.status_code, 400)
        self.assertIn("Duplicate request ID", response.json()["error"]["message"])
```
**EN:** This test exercises `test_duplicate_rid_in_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_duplicate_rid_in_batch`。

### Lines 192-208: test case server healthy after duplicate rid / 测试用例 server healthy after duplicate rid
```python
    def test_server_healthy_after_duplicate_rid(self):
        requests.post(
            f"{self.base_url}/generate",
            json={
                "text": ["Hello", "World"],
                "sampling_params": {"temperature": 0, "max_new_tokens": 8},
                "rid": ["dup-health", "dup-health"],
            },
            timeout=30,
        )

        resp = requests.get(f"{self.base_url}/health", timeout=5)
        self.assertEqual(resp.status_code, 200)

        resp = self._generate_with_rid("after-dup-health")
        self.assertEqual(resp.status_code, 200)
        self.assertIn("text", resp.json())
```
**EN:** This test exercises `test_server_healthy_after_duplicate_rid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_server_healthy_after_duplicate_rid`。

### Lines 211-211: class TestAbortAllWithRetraction declaration / 类 TestAbortAllWithRetraction 声明
```python
class TestAbortAllWithRetraction(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 212-234: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        # Here's a small trick: in scheduler.py, when SGLANG_TEST_RETRACT is enabled,
        # retraction is triggered when the batch size reaches 10.
        # However, since SGLANG_TEST_RETRACT_NO_PREFILL_BS is set to 6, the remaining 4
        # requests will stay in the waiting queue.
        with (
            envs.SGLANG_TEST_RETRACT.override(True),
            envs.SGLANG_TEST_RETRACT_NO_PREFILL_BS.override(6),
        ):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=[
                    "--max-running-requests",
                    16,
                    "--schedule-policy",
                    "random",
                ],
            )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 236-238: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 240-254: method run decode / 方法 run decode
```python
    def _run_decode(self):
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 4000,
                    "ignore_eos": True,
                },
                "return_logprob": True,
                "top_logprobs_num": 3,
            },
        )
        return response.json()
```
**EN:** This block implements `_run_decode` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_decode`，承担模块行为中的一个聚焦逻辑片段。

### Lines 256-306: test case abort all with retraction / 测试用例 abort all with retraction
```python
    def test_abort_all_with_retraction(self):
        num_requests = 32
        with ThreadPoolExecutor(num_requests) as executor:
            futures = [executor.submit(self._run_decode) for _ in range(num_requests)]

            # ensure the decode has been started and retractions happen.
            time.sleep(8)

            requests.post(
                self.base_url + "/abort_request",
                json={
                    "abort_all": True,
                },
            )

            abort_in_queue_count = 0
            abort_in_queue_with_partial_gen = 0

            for future in as_completed(futures):
                result = future.result()
                meta_info = result["meta_info"]
                finish_reason = meta_info.get("finish_reason", {})

                self.assertEqual(finish_reason.get("type"), "abort")

                if finish_reason.get("message") == "Abort in waiting queue":
                    abort_in_queue_count += 1
                    output_ids = result.get("output_ids", [])

                    if len(output_ids) > 0:
                        abort_in_queue_with_partial_gen += 1

                        self.assertEqual(
                            meta_info.get("completion_tokens"), len(output_ids)
                        )
                        self.assertGreater(len(result.get("text", "")), 0)
                        self.assertIsNotNone(meta_info.get("weight_version"))
                        self.assertGreater(meta_info.get("e2e_latency"), 0)
                        for logprob_key in [
                            "output_token_logprobs",
                            "output_top_logprobs",
                        ]:
                            self.assertEqual(
                                len(meta_info.get(logprob_key, [])),
                                len(output_ids),
                                f"Length of '{logprob_key}' should match output_ids length",
                            )

            self.assertGreater(abort_in_queue_count, 0)
            self.assertGreater(abort_in_queue_with_partial_gen, 0)
            print("Finished test_abort_all_with_retraction")
```
**EN:** This test exercises `test_abort_all_with_retraction` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_abort_all_with_retraction`。

### Lines 309-309: class TestAbortWithWaitingTimeout declaration / 类 TestAbortWithWaitingTimeout 声明
```python
class TestAbortWithWaitingTimeout(WaitingTimeoutMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `WaitingTimeoutMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `WaitingTimeoutMixin`, `CustomTestCase`。

### Lines 310-322: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        with envs.SGLANG_REQ_WAITING_TIMEOUT.override(0.001):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=[
                    "--max-running-requests=1",
                ],
            )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 324-326: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 329-329: class TestAbortWithRunningTimeout declaration / 类 TestAbortWithRunningTimeout 声明
```python
class TestAbortWithRunningTimeout(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 330-343: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        with (
            envs.SGLANG_REQ_RUNNING_TIMEOUT.override(0.001),
            envs.SGLANG_ENABLE_HEALTH_ENDPOINT_GENERATION.override(False),
        ):
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=["--skip-server-warmup"],
            )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 345-347: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 349-363: test case running timeout / 测试用例 running timeout
```python
    def test_running_timeout(self):
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": "Today is ",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 512,
                    "ignore_eos": True,
                },
            },
        )
        result = response.json()
        self.assertEqual(result["object"], "error")
        self.assertEqual(result["code"], 503)
```
**EN:** This test exercises `test_running_timeout` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_running_timeout`。

### Lines 366-367: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestAbort`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAbortWithApiKey`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSchedulerControl`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAbortAllWithRetraction`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAbortWithWaitingTimeout`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAbortWithRunningTimeout`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAbort.workload_func`: This block implements `workload_func` and captures one focused piece of the module's behavior. / 该代码块实现 `workload_func`，承担模块行为中的一个聚焦逻辑片段。
- `TestAbort.test_memory_leak`: This test exercises `test_memory_leak` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_memory_leak`。
- `TestAbortWithApiKey.workload_func`: This block implements `workload_func` and captures one focused piece of the module's behavior. / 该代码块实现 `workload_func`，承担模块行为中的一个聚焦逻辑片段。
- `TestAbortWithApiKey.test_memory_leak_with_api_key`: This test exercises `test_memory_leak_with_api_key` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_memory_leak_with_api_key`。
- `TestSchedulerControl.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestSchedulerControl.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `threading`, `time`, `unittest`, `concurrent.futures`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.abort_timeout_kit`, `sglang.test.kits.pause_generation_kit`, `sglang.test.test_utils`

- **Total lines / 总行数**: 367
