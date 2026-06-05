# test_eagle_infer_b.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/spec/eagle/test_eagle_infer_b.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates eagle infer b behavior in SGLang's spec / eagle area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推测解码 / eagle 领域中与 eagle infer b 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: module imports and dependencies / 模块导入与依赖
```python
import json
import random
import threading
import time
import unittest
from concurrent.futures import ThreadPoolExecutor
from functools import partial
from types import SimpleNamespace

import numpy as np
import requests

from sglang.srt.environ import envs
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kits.abort_timeout_kit import (
    AbortAllMixin,
    RunningTimeoutTwoWaveMixin,
    WaitingTimeoutMixin,
)
from sglang.test.kits.radix_cache_server_kit import run_radix_attention_test
from sglang.test.run_eval import run_eval
from sglang.test.server_fixtures.eagle_fixture import EagleServerBase
from sglang.test.test_utils import DEFAULT_TARGET_MODEL_EAGLE, run_logprob_check
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `random`, `threading`, `time`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `random`, `threading`, `time`。

### Lines 25-25: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=847, stage="base-b", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 28-28: class TestEAGLEServerBasic declaration / 类 TestEAGLEServerBasic 声明
```python
class TestEAGLEServerBasic(EagleServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `EagleServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `EagleServerBase`。

### Lines 29-31: class-level constants and configuration for `TestEAGLEServerBasic` / 类级常量与配置
```python
    """Core tests that run on every server config variant."""

    extra_args = ["--chunked-prefill-size", 128, "--max-running-requests", 8]
```
**EN:** This block defines shared names such as `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 33-36: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        with envs.SGLANG_ENABLE_SPEC_V2.override(False):
            super().setUpClass()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 37-38: supporting source context / 辅助源码上下文
```python

    # FIXME(lsyin): move the test methods to kits
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 39-50: test case request abort / 测试用例 request abort
```python
    def test_request_abort(self):
        concurrency = 4
        threads = [
            threading.Thread(target=self.send_request) for _ in range(concurrency)
        ] + [
            threading.Thread(target=self.send_requests_abort)
            for _ in range(concurrency)
        ]
        for worker in threads:
            worker.start()
        for p in threads:
            p.join()
```
**EN:** This test exercises `test_request_abort` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_request_abort`。

### Lines 52-83: test case gsm8k / 测试用例 gsm8k
```python
    def test_gsm8k(self):
        requests.get(self.base_url + "/flush_cache")

        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.target_model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=200,
            num_threads=128,
        )

        metrics = run_eval(args)
        print(f"{metrics=}")
        self.assertGreater(metrics["score"], 0.20)

        server_info = requests.get(self.base_url + "/server_info").json()
        avg_spec_accept_length = server_info["internal_states"][0][
            "avg_spec_accept_length"
        ]
        print(f"{avg_spec_accept_length=}")

        speculative_eagle_topk = server_info["speculative_eagle_topk"]

        if speculative_eagle_topk == 1:
            self.assertGreater(avg_spec_accept_length, 2.5)
        else:
            self.assertGreater(avg_spec_accept_length, 3.47)

        # Wait a little bit so that the memory check happens.
        time.sleep(4)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 86-86: class TestEAGLEServerAdditional declaration / 类 TestEAGLEServerAdditional 声明
```python
class TestEAGLEServerAdditional(TestEAGLEServerBasic):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestEAGLEServerBasic`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestEAGLEServerBasic`。

### Lines 87-101: class-level constants and configuration for `TestEAGLEServerAdditional` / 类级常量与配置
```python
    spec_topk = 5
    spec_steps = 8
    spec_tokens = 64
    extra_args = [
        "--max-running-requests",
        8,
        "--cuda-graph-max-bs",
        5,
        "--attention-backend",
        "fa3",
        "--page-size",
        256,
        "--dtype",
        "float16",
    ]
```
**EN:** This block defines shared names such as `spec_topk`, `spec_steps`, `spec_tokens`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `spec_topk`, `spec_steps`, `spec_tokens`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 103-105: test case radix attention / 测试用例 radix attention
```python
    def test_radix_attention(self):
        run_radix_attention_test(self.base_url)
        self.assertIsNone(self.process.poll())
```
**EN:** This test exercises `test_radix_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_radix_attention`。

### Lines 107-121: test case max token one / 测试用例 max token one
```python
    def test_max_token_one(self):
        requests.get(self.base_url + "/flush_cache")

        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.target_model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=1,
            num_examples=200,
            num_threads=128,
        )

        metrics = run_eval(args)
        self.assertGreater(metrics["output_throughput"], 50)
```
**EN:** This test exercises `test_max_token_one` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_max_token_one`。

### Lines 123-154: test case logprob start len / 测试用例 logprob start len
```python
    def test_logprob_start_len(self):
        logprob_start_len = 4
        new_tokens = 4
        prompts = [
            "I have a very good idea on",
            "Today is a sunndy day and",
        ]

        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": prompts,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": new_tokens,
                },
                "return_logprob": True,
                "top_logprobs_num": 5,
                "logprob_start_len": logprob_start_len,
            },
        )
        response_json = response.json()
        print(json.dumps(response_json, indent=2))

        for res in response_json:
            self.assertEqual(
                res["meta_info"]["prompt_tokens"],
                logprob_start_len + len(res["meta_info"]["input_token_logprobs"]),
            )

            self.assertEqual(res["meta_info"]["completion_tokens"], new_tokens)
            self.assertEqual(len(res["meta_info"]["output_token_logprobs"]), new_tokens)
```
**EN:** This test exercises `test_logprob_start_len` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logprob_start_len`。

### Lines 156-228: test case logprob match / 测试用例 logprob match
```python
    def test_logprob_match(self):
        """Test the output logprobs are close to the input logprobs if we run a prefill again."""

        def run_generate(
            prompt,
            return_logprob=False,
            max_new_tokens=512,
            logprob_start_len=-1,
            temperature=1.0,
        ):

            if isinstance(prompt, str):
                prompt_kwargs = {"text": prompt}
            else:
                prompt_kwargs = {"input_ids": prompt}

            response = requests.post(
                self.base_url + "/generate",
                json={
                    **prompt_kwargs,
                    "sampling_params": {
                        "temperature": temperature,
                        "max_new_tokens": max_new_tokens,
                        "ignore_eos": True,
                    },
                    "return_logprob": return_logprob,
                    "return_text_in_logprobs": True,
                    "logprob_start_len": logprob_start_len,
                    "temp_scaled_logprobs": True,
                },
            )
            return response.json()

        prompt = "I have a very good idea on how to"

        for temperature in [1.0]:
            gen = run_generate(
                prompt,
                return_logprob=True,
                logprob_start_len=0,
                temperature=temperature,
            )
            output_logprobs = np.array(
                [x[0] for x in gen["meta_info"]["output_token_logprobs"]]
            )
            num_prompts_tokens = gen["meta_info"]["prompt_tokens"]

            input_tokens = [x[1] for x in gen["meta_info"]["input_token_logprobs"]]
            output_tokens = [x[1] for x in gen["meta_info"]["output_token_logprobs"]]

            new_prompt = input_tokens + output_tokens
            score = run_generate(
                new_prompt,
                return_logprob=True,
                logprob_start_len=0,
                max_new_tokens=0,
                temperature=temperature,
            )
            output_logprobs_score = np.array(
                [
                    x[0]
                    for x in score["meta_info"]["input_token_logprobs"][
                        num_prompts_tokens:
                    ]
                ]
            )

            print(f"{output_logprobs[-10:]=}")
            print(f"{output_logprobs_score[-10:]=}")

            diff = np.abs(output_logprobs - output_logprobs_score)
            max_diff = np.max(diff)
            self.assertLess(max_diff, 0.255)
```
**EN:** Test the output logprobs are close to the input logprobs if we run a prefill again. This test exercises `test_logprob_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test the output logprobs are close to the input logprobs if we run a prefill again. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logprob_match`。

### Lines 230-259: test case logprob mixed / 测试用例 logprob mixed
```python
    def test_logprob_mixed(self):
        args = []
        temperature = 0
        # input_len, output_len, temperature, logprob_start_len, return_logprob, top_logprobs_num
        # Llama 2 context length seems to be only 2k, so we can only test small length.
        for input_len in [200, 500, 1000, 2000]:
            for output_len in [4, 8]:
                for logprob_start_len in [0, 100, 300, 800, 1998]:
                    for return_logprob in [True, False]:
                        for top_logprobs_num in [0, 5]:

                            if logprob_start_len >= input_len:
                                continue

                            args.append(
                                (
                                    input_len,
                                    output_len,
                                    temperature,
                                    logprob_start_len,
                                    return_logprob,
                                    top_logprobs_num,
                                )
                            )

        random.shuffle(args)

        func = partial(run_logprob_check, self)
        with ThreadPoolExecutor(8) as executor:
            list(executor.map(func, args))
```
**EN:** This test exercises `test_logprob_mixed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logprob_mixed`。

### Lines 261-280: test case penalty mixed / 测试用例 penalty mixed
```python
    def test_penalty_mixed(self):
        args = [
            {},
            {},
            {},
            {"frequency_penalty": 2},
            {"presence_penalty": 1},
            {"min_new_tokens": 16},
            {"frequency_penalty": 0.2},
            {"presence_penalty": 0.4},
            {"min_new_tokens": 8},
            {"frequency_penalty": 0.4, "presence_penalty": 0.8},
            {"frequency_penalty": 0.4, "min_new_tokens": 12},
            {"presence_penalty": 0.8, "min_new_tokens": 12},
            {"presence_penalty": -0.3, "frequency_penalty": 1.3, "min_new_tokens": 32},
            {"presence_penalty": 0.3, "frequency_penalty": -1.3, "min_new_tokens": 32},
        ]
        random.shuffle(args * 5)
        with ThreadPoolExecutor(8) as executor:
            list(executor.map(self.run_decode, args))
```
**EN:** This test exercises `test_penalty_mixed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_penalty_mixed`。

### Lines 282-315: test case constrained decoding / 测试用例 constrained decoding
```python
    def test_constrained_decoding(self):
        messages = [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Give me a json"},
        ]

        response = requests.post(
            self.base_url + "/v1/chat/completions",
            json={
                "model": DEFAULT_TARGET_MODEL_EAGLE,
                "messages": messages,
                "temperature": 0,
                "response_format": {"type": "json_object"},
            },
        )
        self.assertEqual(response.status_code, 200)
        res = response.json()

        # Validate response structure
        self.assertIn("choices", res)
        self.assertEqual(len(res["choices"]), 1)
        self.assertIn("message", res["choices"][0])
        self.assertIn("content", res["choices"][0]["message"])

        # Validate JSON content
        content_json = res["choices"][0]["message"]["content"]
        is_valid_json = True
        try:
            content = json.loads(content_json)
            self.assertIsInstance(content, dict)
        except Exception:
            print(f"parse JSON failed: {content_json}")
            is_valid_json = False
        self.assertTrue(is_valid_json)
```
**EN:** This test exercises `test_constrained_decoding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_constrained_decoding`。

### Lines 318-318: class TestEAGLERetract declaration / 类 TestEAGLERetract 声明
```python
class TestEAGLERetract(TestEAGLEServerBasic):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestEAGLEServerBasic`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestEAGLEServerBasic`。

### Lines 319-323: class-level constants and configuration for `TestEAGLERetract` / 类级常量与配置
```python
    extra_args = [
        "--chunked-prefill-size=128",
        "--max-running-requests=64",
        "--max-total-tokens=4500",  # Set a smaller KV cache to trigger retract more easily
    ]
```
**EN:** This block defines shared names such as `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 325-329: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        # These config helps find a leak.
        with envs.SGLANG_TEST_RETRACT.override(True):
            super().setUpClass()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 332-332: class TestEAGLEServerTriton declaration / 类 TestEAGLEServerTriton 声明
```python
class TestEAGLEServerTriton(TestEAGLEServerBasic):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestEAGLEServerBasic`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestEAGLEServerBasic`。

### Lines 333-333: class-level constants and configuration for `TestEAGLEServerTriton` / 类级常量与配置
```python
    extra_args = ["--attention-backend=triton", "--max-running-requests=8"]
```
**EN:** This block defines shared names such as `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 336-336: class TestEAGLEServerPageSize declaration / 类 TestEAGLEServerPageSize 声明
```python
class TestEAGLEServerPageSize(TestEAGLEServerBasic):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestEAGLEServerBasic`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestEAGLEServerBasic`。

### Lines 337-345: class-level constants and configuration for `TestEAGLEServerPageSize` / 类级常量与配置
```python
    spec_steps = 5
    spec_topk = 1
    spec_tokens = 6
    extra_args = [
        "--chunked-prefill-size=128",
        "--max-running-requests=8",
        "--page-size=4",
        "--attention-backend=flashinfer",
    ]
```
**EN:** This block defines shared names such as `spec_steps`, `spec_topk`, `spec_tokens`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `spec_steps`, `spec_topk`, `spec_tokens`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 347-351: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        # Runtime check only supported for topk=1, and can help to find a leak.
        with envs.SGLANG_ENABLE_STRICT_MEM_CHECK_DURING_BUSY.override(1):
            super().setUpClass()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 354-355: class TestEAGLEServerPageSizeTopk declaration / 类 TestEAGLEServerPageSizeTopk 声明
```python
class TestEAGLEServerPageSizeTopk(TestEAGLEServerBasic):
    # default topk=8 and tokens=64
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestEAGLEServerBasic`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestEAGLEServerBasic`。

### Lines 356-361: class-level constants and configuration for `TestEAGLEServerPageSizeTopk` / 类级常量与配置
```python
    extra_args = [
        "--chunked-prefill-size=128",
        "--max-running-requests=8",
        "--page-size=4",
        "--attention-backend=flashinfer",
    ]
```
**EN:** This block defines shared names such as `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 364-364: class TestEAGLEAbortAll declaration / 类 TestEAGLEAbortAll 声明
```python
class TestEAGLEAbortAll(AbortAllMixin, EagleServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `AbortAllMixin`, `EagleServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `AbortAllMixin`, `EagleServerBase`。

### Lines 365-366: class-level constants and configuration for `TestEAGLEAbortAll` / 类级常量与配置
```python
    abort_all_max_new_tokens = 4000
    extra_args = ["--max-running-requests=8"]
```
**EN:** This block defines shared names such as `abort_all_max_new_tokens`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `abort_all_max_new_tokens`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 369-369: class TestEAGLEWaitingTimeout declaration / 类 TestEAGLEWaitingTimeout 声明
```python
class TestEAGLEWaitingTimeout(WaitingTimeoutMixin, EagleServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `WaitingTimeoutMixin`, `EagleServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `WaitingTimeoutMixin`, `EagleServerBase`。

### Lines 370-370: class-level constants and configuration for `TestEAGLEWaitingTimeout` / 类级常量与配置
```python
    extra_args = ["--max-running-requests=1"]
```
**EN:** This block defines shared names such as `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 372-375: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        with envs.SGLANG_REQ_WAITING_TIMEOUT.override(0.001):
            super().setUpClass()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 378-379: class TestEAGLERunningTimeout declaration / 类 TestEAGLERunningTimeout 声明
```python
class TestEAGLERunningTimeout(RunningTimeoutTwoWaveMixin, EagleServerBase):
    # Regression test for https://github.com/sgl-project/sglang/pull/18760
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `RunningTimeoutTwoWaveMixin`, `EagleServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `RunningTimeoutTwoWaveMixin`, `EagleServerBase`。

### Lines 380-380: class-level constants and configuration for `TestEAGLERunningTimeout` / 类级常量与配置
```python
    extra_args = ["--max-running-requests=16"]
```
**EN:** This block defines shared names such as `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 382-385: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        with envs.SGLANG_REQ_RUNNING_TIMEOUT.override(3):
            super().setUpClass()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 388-389: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestEAGLEServerBasic`: Core tests that run on every server config variant. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLEServerAdditional`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLERetract`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLEServerTriton`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLEServerPageSize`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLEServerPageSizeTopk`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLEAbortAll`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLEWaitingTimeout`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestEAGLEServerBasic.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestEAGLEServerBasic.test_request_abort`: This test exercises `test_request_abort` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_request_abort`。
- `TestEAGLEServerBasic.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `TestEAGLEServerAdditional.test_radix_attention`: This test exercises `test_radix_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_radix_attention`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `random`, `threading`, `time`, `unittest`, `concurrent.futures`, `functools`, `types`
- **Third-party modules / 第三方模块**: `numpy`, `requests`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.test.ci.ci_register`, `sglang.test.kits.abort_timeout_kit`, `sglang.test.kits.radix_cache_server_kit`, `sglang.test.run_eval`, `sglang.test.server_fixtures.eagle_fixture`, `sglang.test.test_utils`

- **Total lines / 总行数**: 389
