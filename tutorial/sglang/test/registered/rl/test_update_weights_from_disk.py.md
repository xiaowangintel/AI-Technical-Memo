# test_update_weights_from_disk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/rl/test_update_weights_from_disk.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates update weights from disk behavior in SGLang's rl area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 rl 领域中与 update weights from disk 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: module imports and dependencies / 模块导入与依赖
```python
import json
import random
import time
import unittest
from concurrent.futures import ThreadPoolExecutor, as_completed

import requests

import sglang as sgl
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `random`, `time`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `random`, `time`, `unittest`。

### Lines 21-26: CI registration and metadata / CI 注册与元数据
```python
register_amd_ci(
    est_time=210, suite="stage-b-test-1-gpu-small-amd", disabled="see #14021"
)
register_cuda_ci(
    est_time=210, stage="base-b", runner_config="1-gpu-large", disabled="see #14021"
)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_amd_ci, register_cuda_ci.
**CN:** 该代码块通过 register_amd_ci, register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 27-31: supporting source context / 辅助源码上下文
```python


###############################################################################
# Engine Mode Tests (Single-configuration)
###############################################################################
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 32-32: class TestEngineUpdateWeightsFromDisk declaration / 类 TestEngineUpdateWeightsFromDisk 声明
```python
class TestEngineUpdateWeightsFromDisk(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 33-36: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        # Initialize the engine in offline (direct) mode.
        self.engine = sgl.Engine(model_path=self.model)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 38-39: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        self.engine.shutdown()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 41-49: helper routine run decode / 辅助流程 run decode
```python
    def run_decode(self):
        prompts = ["The capital of France is"]
        sampling_params = {"temperature": 0, "max_new_tokens": 32}
        outputs = self.engine.generate(prompts, sampling_params)
        print("=" * 100)
        print(
            f"[Engine Mode] Prompt: {prompts[0]}\nGenerated text: {outputs[0]['text']}"
        )
        return outputs[0]["text"]
```
**EN:** This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 51-54: helper routine run update weights / 辅助流程 run update weights
```python
    def run_update_weights(self, model_path):
        ret = self.engine.update_weights_from_disk(model_path)
        print(json.dumps(ret))
        return ret
```
**EN:** This helper encapsulates `run_update_weights` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_update_weights`，以便周围测试复用准备、执行或校验逻辑。

### Lines 56-70: test case update weights / 测试用例 update weights
```python
    def test_update_weights(self):
        origin_response = self.run_decode()
        # Update weights: use new model (remove "-Instruct")
        new_model_path = self.model.replace("-Instruct", "")
        ret = self.run_update_weights(new_model_path)
        self.assertTrue(ret[0])  # ret is a tuple; index 0 holds the success flag

        updated_response = self.run_decode()
        self.assertNotEqual(origin_response[:32], updated_response[:32])

        # Revert back to original weights
        ret = self.run_update_weights(self.model)
        self.assertTrue(ret[0])
        reverted_response = self.run_decode()
        self.assertEqual(origin_response[:32], reverted_response[:32])
```
**EN:** This test exercises `test_update_weights` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights`。

### Lines 72-78: test case update weights unexist model / 测试用例 update weights unexist model
```python
    def test_update_weights_unexist_model(self):
        origin_response = self.run_decode()
        new_model_path = self.model.replace("-Instruct", "wrong")
        ret = self.run_update_weights(new_model_path)
        self.assertFalse(ret[0])
        updated_response = self.run_decode()
        self.assertEqual(origin_response[:32], updated_response[:32])
```
**EN:** This test exercises `test_update_weights_unexist_model` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_unexist_model`。

### Lines 79-83: supporting source context / 辅助源码上下文
```python


###############################################################################
# HTTP Server Mode Tests (Single-configuration)
###############################################################################
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 84-84: class TestServerUpdateWeightsFromDisk declaration / 类 TestServerUpdateWeightsFromDisk 声明
```python
class TestServerUpdateWeightsFromDisk(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 85-91: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model, cls.base_url, timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 93-95: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 97-107: helper routine run decode / 辅助流程 run decode
```python
    def run_decode(self):
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {"temperature": 0, "max_new_tokens": 32},
            },
        )
        print("=" * 100)
        print(f"[Server Mode] Generated text: {response.json()['text']}")
        return response.json()["text"]
```
**EN:** This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 109-121: helper routine run decode random / 辅助流程 run decode random
```python
    def run_decode_random(self, max_new_tokens=32):
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": f"Question: {random.randint(0, 100)},The capital of France is",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": max_new_tokens,
                    "ignore_eos": True,
                },
            },
        )
        return response.json()
```
**EN:** This helper encapsulates `run_decode_random` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_decode_random`，以便周围测试复用准备、执行或校验逻辑。

### Lines 123-127: helper routine get model info / 辅助流程 get model info
```python
    def get_model_info(self):
        response = requests.get(self.base_url + "/get_model_info")
        model_path = response.json()["model_path"]
        print(json.dumps(response.json()))
        return model_path
```
**EN:** This helper encapsulates `get_model_info` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_model_info`，以便周围测试复用准备、执行或校验逻辑。

### Lines 129-138: helper routine run update weights / 辅助流程 run update weights
```python
    def run_update_weights(self, model_path, flush_cache=True):
        response = requests.post(
            self.base_url + "/update_weights_from_disk",
            json={
                "model_path": model_path,
                "flush_cache": flush_cache,
            },
        )
        ret = response.json()
        return ret
```
**EN:** This helper encapsulates `run_update_weights` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_update_weights`，以便周围测试复用准备、执行或校验逻辑。

### Lines 140-146: method pause generation / 方法 pause generation
```python
    def pause_generation(self, mode):
        response = requests.post(
            self.base_url + "/pause_generation",
            json={"mode": mode},
        )
        ret = response.json()
        return ret
```
**EN:** This block implements `pause_generation` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `pause_generation`，承担模块行为中的一个聚焦逻辑片段。

### Lines 148-154: method continue generation / 方法 continue generation
```python
    def continue_generation(self):
        response = requests.post(
            self.base_url + "/continue_generation",
            json={},
        )
        ret = response.json()
        return ret
```
**EN:** This block implements `continue_generation` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `continue_generation`，承担模块行为中的一个聚焦逻辑片段。

### Lines 156-179: test case update weights / 测试用例 update weights
```python
    def test_update_weights(self):
        origin_model_path = self.get_model_info()
        print(f"[Server Mode] origin_model_path: {origin_model_path}")
        origin_response = self.run_decode()

        new_model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST.replace("-Instruct", "")
        ret = self.run_update_weights(new_model_path)
        self.assertTrue(ret["success"])

        updated_model_path = self.get_model_info()
        print(f"[Server Mode] updated_model_path: {updated_model_path}")
        self.assertEqual(updated_model_path, new_model_path)
        self.assertNotEqual(updated_model_path, origin_model_path)

        updated_response = self.run_decode()
        self.assertNotEqual(origin_response[:32], updated_response[:32])

        ret = self.run_update_weights(origin_model_path)
        self.assertTrue(ret["success"])
        updated_model_path = self.get_model_info()
        self.assertEqual(updated_model_path, origin_model_path)

        updated_response = self.run_decode()
        self.assertEqual(origin_response[:32], updated_response[:32])
```
**EN:** This test exercises `test_update_weights` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights`。

### Lines 181-215: test case update weights non blocking / 测试用例 update weights non blocking
```python
    def test_update_weights_non_blocking(self):
        origin_model_path = self.get_model_info()
        print(f"[Server Mode] origin_model_path: {origin_model_path}")

        pause_generation_modes = ["in_place", "retract"]
        for pause_generation_mode in pause_generation_modes:
            num_requests = 32
            with ThreadPoolExecutor(num_requests) as executor:
                futures = [
                    executor.submit(self.run_decode_random, 1600)
                    for _ in range(num_requests)
                ]

                # ensure the decode has been started
                time.sleep(2)

                new_model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST.replace(
                    "-Instruct", ""
                )
                ret = self.pause_generation(pause_generation_mode)
                ret = self.run_update_weights(
                    new_model_path, flush_cache=pause_generation_mode == "retract"
                )
                self.assertTrue(ret["success"])
                ret = self.continue_generation()

                for future in as_completed(futures):
                    self.assertNotEqual(
                        future.result()["meta_info"]["finish_reason"]["type"], "abort"
                    )

            updated_model_path = self.get_model_info()
            print(f"[Server Mode] updated_model_path: {updated_model_path}")
            self.assertEqual(updated_model_path, new_model_path)
            self.assertNotEqual(updated_model_path, origin_model_path)
```
**EN:** This test exercises `test_update_weights_non_blocking` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_non_blocking`。

### Lines 217-231: test case update weights unexist model / 测试用例 update weights unexist model
```python
    def test_update_weights_unexist_model(self):
        origin_model_path = self.get_model_info()
        print(f"[Server Mode] origin_model_path: {origin_model_path}")
        origin_response = self.run_decode()

        new_model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST.replace("-Instruct", "wrong")
        ret = self.run_update_weights(new_model_path)
        self.assertFalse(ret["success"])

        updated_model_path = self.get_model_info()
        print(f"[Server Mode] updated_model_path: {updated_model_path}")
        self.assertEqual(updated_model_path, origin_model_path)

        updated_response = self.run_decode()
        self.assertEqual(origin_response[:32], updated_response[:32])
```
**EN:** This test exercises `test_update_weights_unexist_model` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_unexist_model`。

### Lines 234-234: class TestServerUpdateWeightsFromDiskAbortAllRequests declaration / 类 TestServerUpdateWeightsFromDiskAbortAllRequests 声明
```python
class TestServerUpdateWeightsFromDiskAbortAllRequests(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 235-244: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
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

### Lines 246-248: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 250-262: helper routine run decode / 辅助流程 run decode
```python
    def run_decode(self, max_new_tokens=32):
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": max_new_tokens,
                    "ignore_eos": True,
                },
            },
        )
        return response.json()
```
**EN:** This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 264-268: helper routine get model info / 辅助流程 get model info
```python
    def get_model_info(self):
        response = requests.get(self.base_url + "/get_model_info")
        model_path = response.json()["model_path"]
        print(json.dumps(response.json()))
        return model_path
```
**EN:** This helper encapsulates `get_model_info` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_model_info`，以便周围测试复用准备、执行或校验逻辑。

### Lines 270-280: helper routine run update weights / 辅助流程 run update weights
```python
    def run_update_weights(self, model_path, abort_all_requests=False):
        response = requests.post(
            self.base_url + "/update_weights_from_disk",
            json={
                "model_path": model_path,
                "abort_all_requests": abort_all_requests,
            },
        )
        ret = response.json()
        print(json.dumps(ret))
        return ret
```
**EN:** This helper encapsulates `run_update_weights` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_update_weights`，以便周围测试复用准备、执行或校验逻辑。

### Lines 282-307: test case update weights abort all requests / 测试用例 update weights abort all requests
```python
    def test_update_weights_abort_all_requests(self):
        origin_model_path = self.get_model_info()
        print(f"[Server Mode] origin_model_path: {origin_model_path}")

        num_requests = 32
        with ThreadPoolExecutor(num_requests) as executor:
            futures = [
                executor.submit(self.run_decode, 16000) for _ in range(num_requests)
            ]

            # ensure the decode has been started
            time.sleep(2)

            new_model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST.replace("-Instruct", "")
            ret = self.run_update_weights(new_model_path, abort_all_requests=True)
            self.assertTrue(ret["success"])

            for future in as_completed(futures):
                self.assertEqual(
                    future.result()["meta_info"]["finish_reason"]["type"], "abort"
                )

        updated_model_path = self.get_model_info()
        print(f"[Server Mode] updated_model_path: {updated_model_path}")
        self.assertEqual(updated_model_path, new_model_path)
        self.assertNotEqual(updated_model_path, origin_model_path)
```
**EN:** This test exercises `test_update_weights_abort_all_requests` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_abort_all_requests`。

### Lines 308-316: supporting source context / 辅助源码上下文
```python


###############################################################################
# Parameterized Tests for update_weights_from_disk
# Test coverage is determined based on the value of is_in_ci:
# - In a CI environment: randomly select one mode (Engine or Server) and test only with tp=1, dp=1.
# - In a non-CI environment: test both Engine and Server modes, and enumerate all combinations
#   with tp and dp ranging from 1 to 2.
###############################################################################
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 317-317: class TestUpdateWeightsFromDiskParameterized declaration / 类 TestUpdateWeightsFromDiskParameterized 声明
```python
class TestUpdateWeightsFromDiskParameterized(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 318-352: helper routine run common test / 辅助流程 run common test
```python
    def run_common_test(self, mode, tp, dp):
        """
        Common test procedure for update_weights_from_disk.
        For Engine mode, we instantiate the engine with tp_size=tp.
        For Server mode, we launch the server with additional arguments for tp (dp is not used in server launch here).
        """
        if mode == "Engine":
            # Instantiate engine with additional parameter tp_size.
            print(f"[Parameterized Engine] Testing with tp={tp}, dp={dp}")
            engine = sgl.Engine(
                model_path=DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
                random_seed=42,
                tp_size=tp,
                # dp parameter is not explicitly used in this API.
            )
            try:
                origin_response = self._engine_update_weights_test(engine)
            finally:
                engine.shutdown()
        elif mode == "Server":
            print(f"[Parameterized Server] Testing with tp={tp}, dp={dp}")
            # Pass additional arguments to launch the server.
            base_args = ["--tp-size", str(tp)]
            process = popen_launch_server(
                DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
                DEFAULT_URL_FOR_TEST,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=base_args,
            )
            try:
                origin_response = self._server_update_weights_test(DEFAULT_URL_FOR_TEST)
            finally:
                kill_process_tree(process.pid)
        else:
            raise ValueError(f"Unknown mode: {mode}")
```
**EN:** Common test procedure for update_weights_from_disk. This helper encapsulates `run_common_test` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Common test procedure for update_weights_from_disk. 该辅助函数封装了 `run_common_test`，以便周围测试复用准备、执行或校验逻辑。

### Lines 354-381: method engine update weights test / 方法 engine update weights test
```python
    def _engine_update_weights_test(self, engine):
        # Run the update weights test on the given engine instance.
        def run_decode():
            prompts = ["The capital of France is"]
            sampling_params = {"temperature": 0, "max_new_tokens": 32}
            outputs = engine.generate(prompts, sampling_params)
            print("=" * 100)
            print(
                f"[Parameterized Engine] Prompt: {prompts[0]}\nGenerated text: {outputs[0]['text']}"
            )
            return outputs[0]["text"]

        def run_update_weights(model_path):
            ret = engine.update_weights_from_disk(model_path)
            print(json.dumps(ret))
            return ret

        origin_response = run_decode()
        new_model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST.replace("-Instruct", "")
        ret = run_update_weights(new_model_path)
        self.assertTrue(ret[0])
        updated_response = run_decode()
        self.assertNotEqual(origin_response[:32], updated_response[:32])
        ret = run_update_weights(DEFAULT_SMALL_MODEL_NAME_FOR_TEST)
        self.assertTrue(ret[0])
        reverted_response = run_decode()
        self.assertEqual(origin_response[:32], reverted_response[:32])
        return origin_response
```
**EN:** This block implements `_engine_update_weights_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_engine_update_weights_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 383-427: method server update weights test / 方法 server update weights test
```python
    def _server_update_weights_test(self, base_url):
        def run_decode():
            response = requests.post(
                base_url + "/generate",
                json={
                    "text": "The capital of France is",
                    "sampling_params": {"temperature": 0, "max_new_tokens": 32},
                },
            )
            print("=" * 100)
            print(f"[Parameterized Server] Generated text: {response.json()['text']}")
            return response.json()["text"]

        def get_model_info():
            response = requests.get(base_url + "/get_model_info")
            model_path = response.json()["model_path"]
            print(json.dumps(response.json()))
            return model_path

        def run_update_weights(model_path):
            response = requests.post(
                base_url + "/update_weights_from_disk",
                json={"model_path": model_path},
            )
            ret = response.json()
            print(json.dumps(ret))
            return ret

        origin_model_path = get_model_info()
        origin_response = run_decode()
        new_model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST.replace("-Instruct", "")
        ret = run_update_weights(new_model_path)
        self.assertTrue(ret["success"])
        updated_model_path = get_model_info()
        self.assertEqual(updated_model_path, new_model_path)
        self.assertNotEqual(updated_model_path, origin_model_path)
        updated_response = run_decode()
        self.assertNotEqual(origin_response[:32], updated_response[:32])
        ret = run_update_weights(origin_model_path)
        self.assertTrue(ret["success"])
        updated_model_path = get_model_info()
        self.assertEqual(updated_model_path, origin_model_path)
        reverted_response = run_decode()
        self.assertEqual(origin_response[:32], reverted_response[:32])
        return origin_response
```
**EN:** This block implements `_server_update_weights_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_server_update_weights_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 429-443: test case parameterized update weights / 测试用例 parameterized update weights
```python
    def test_parameterized_update_weights(self):
        if is_in_ci():
            # In CI, choose one random mode (Engine or Server) with tp=1, dp=1.
            mode = random.choice(["Engine", "Server"])
            test_suits = [(1, 1, mode)]
        else:
            # Otherwise, test both modes and enumerate tp,dp combinations from 1 to 2.
            test_suits = []
            for mode in ["Engine", "Server"]:
                for tp in [1, 2]:
                    for dp in [1, 2]:
                        test_suits.append((tp, dp, mode))
        for tp, dp, mode in test_suits:
            with self.subTest(mode=mode, tp=tp, dp=dp):
                self.run_common_test(mode, tp, dp)
```
**EN:** This test exercises `test_parameterized_update_weights` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parameterized_update_weights`。

### Lines 446-447: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestEngineUpdateWeightsFromDisk`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestServerUpdateWeightsFromDisk`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestServerUpdateWeightsFromDiskAbortAllRequests`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestUpdateWeightsFromDiskParameterized`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEngineUpdateWeightsFromDisk.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestEngineUpdateWeightsFromDisk.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestEngineUpdateWeightsFromDisk.run_decode`: This helper encapsulates `run_decode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestEngineUpdateWeightsFromDisk.run_update_weights`: This helper encapsulates `run_update_weights` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_update_weights`，以便周围测试复用准备、执行或校验逻辑。
- `TestEngineUpdateWeightsFromDisk.test_update_weights`: This test exercises `test_update_weights` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights`。
- `TestEngineUpdateWeightsFromDisk.test_update_weights_unexist_model`: This test exercises `test_update_weights_unexist_model` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_weights_unexist_model`。
- `TestServerUpdateWeightsFromDisk.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestServerUpdateWeightsFromDisk.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `random`, `time`, `unittest`, `concurrent.futures`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 447
