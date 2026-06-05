# test_srt_endpoint.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/core/test_srt_endpoint.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on core srt endpoint in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 core srt endpoint 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Document the module
```python
"""
python3 -m unittest test_srt_endpoint.TestSRTEndpoint.test_simple_decode
python3 -m unittest test_srt_endpoint.TestSRTEndpoint.test_logprob_with_chunked_prefill
python3 -m unittest test_srt_endpoint.TestTokenizeDetokenize
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 7-29: Import dependencies
```python
import json
import random
import time
import unittest
from concurrent.futures import ThreadPoolExecutor
from functools import partial
from typing import Optional

import numpy as np
import requests

from sglang.srt.sampling.custom_logit_processor import CustomLogitProcessor
from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
    run_logprob_check,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 31-32: Register CI metadata
```python
register_cuda_ci(est_time=134, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=130, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 35-35: Define class TestSRTEndpoint
```python
class TestSRTEndpoint(CustomTestCase):
```
**EN:** This declaration introduces the `TestSRTEndpoint` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSRTEndpoint` 测试类，并说明它通过继承承担的职责。

### Lines 36-51: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=(
                "--enable-custom-logit-processor",
                "--mem-fraction-static",
                "0.7",
                "--cuda-graph-max-bs",
                "8",
            ),
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 53-55: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 57-96: Define helper: run decode
```python
    def run_decode(
        self,
        return_logprob=False,
        top_logprobs_num=0,
        return_text=False,
        n=1,
        stream=False,
        batch=False,
    ):
        if batch:
            text = ["The capital of France is"]
        else:
            text = "The capital of France is"

        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": text,
                "sampling_params": {
                    "temperature": 0 if n == 1 else 0.5,
                    "max_new_tokens": 16,
                    "n": n,
                },
                "stream": stream,
                "return_logprob": return_logprob,
                "top_logprobs_num": top_logprobs_num,
                "return_text_in_logprobs": return_text,
                "logprob_start_len": 0,
            },
        )
        if not stream:
            response_json = response.json()
        else:
            response_json = []
            for line in response.iter_lines():
                if line.startswith(b"data: ") and line[6:] != b"[DONE]":
                    response_json.append(json.loads(line[6:]))

        print(json.dumps(response_json, indent=2))
        print("=" * 100)
```
**EN:** This helper function encapsulates reusable logic inside `TestSRTEndpoint` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestSRTEndpoint` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 98-99: Run test: simple decode
```python
    def test_simple_decode(self):
        self.run_decode()
```
**EN:** This test method exercises simple decode and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 simple decode 场景，并验证观测到的行为是否符合预期契约。

### Lines 101-102: Run test: simple decode batch
```python
    def test_simple_decode_batch(self):
        self.run_decode(batch=True)
```
**EN:** This test method exercises simple decode batch and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 simple decode batch 场景，并验证观测到的行为是否符合预期契约。

### Lines 104-105: Run test: parallel sample
```python
    def test_parallel_sample(self):
        self.run_decode(n=3)
```
**EN:** This test method exercises parallel sample and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 parallel sample 场景，并验证观测到的行为是否符合预期契约。

### Lines 107-108: Run test: parallel sample stream
```python
    def test_parallel_sample_stream(self):
        self.run_decode(n=3, stream=True)
```
**EN:** This test method exercises parallel sample stream and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 parallel sample stream 场景，并验证观测到的行为是否符合预期契约。

### Lines 110-115: Run test: logprob
```python
    def test_logprob(self):
        self.run_decode(
            return_logprob=True,
            top_logprobs_num=5,
            return_text=True,
        )
```
**EN:** This test method exercises logprob and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 logprob 场景，并验证观测到的行为是否符合预期契约。

### Lines 117-156: Run test: logprob start len
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
                "return_text_in_logprobs": True,
                "logprob_start_len": logprob_start_len,
            },
        )
        response_json = response.json()
        print(json.dumps(response_json, indent=2))

        for i, res in enumerate(response_json):
            self.assertEqual(
                res["meta_info"]["prompt_tokens"],
                logprob_start_len + len(res["meta_info"]["input_token_logprobs"]),
            )
            assert prompts[i].endswith(
                "".join([x[-1] for x in res["meta_info"]["input_token_logprobs"]])
            )

            self.assertEqual(res["meta_info"]["completion_tokens"], new_tokens)
            self.assertEqual(len(res["meta_info"]["output_token_logprobs"]), new_tokens)
            self.assertEqual(
                res["text"],
                "".join([x[-1] for x in res["meta_info"]["output_token_logprobs"]]),
            )
```
**EN:** This test method exercises logprob start len and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 logprob start len 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 158-192: Run test: logprob with chunked prefill
```python
    def test_logprob_with_chunked_prefill(self):
        """Test a long prompt that requests output logprobs will not hit OOM."""
        new_tokens = 4
        prompts = "I have a very good idea on this. " * 8000

        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": prompts,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": new_tokens,
                },
                "return_logprob": True,
                "logprob_start_len": -1,
                "top_logprobs_num": 5,
            },
        )
        response_json = response.json()
        # print(json.dumps(response_json, indent=2))

        res = response_json
        self.assertEqual(res["meta_info"]["completion_tokens"], new_tokens)

        # Test the number of tokens are correct
        self.assertEqual(len(res["meta_info"]["output_token_logprobs"]), new_tokens)
        self.assertEqual(len(res["meta_info"]["output_top_logprobs"]), new_tokens)

        # Test the top-1 tokens are the same as output tokens (because temp = 0.0)
        for i in range(new_tokens):
            self.assertListEqual(
                res["meta_info"]["output_token_logprobs"][i],
                res["meta_info"]["output_top_logprobs"][i][0],
            )
            self.assertEqual(len(res["meta_info"]["output_top_logprobs"][i]), 5)
```
**EN:** This test method exercises logprob with chunked prefill and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 logprob with chunked prefill 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 194-249: Run test: logprob match
```python
    def test_logprob_match(self):
        """Test the output logprobs are close to the input logprobs if we run a prefill again."""

        def run_generate(
            prompt, return_logprob=False, max_new_tokens=512, logprob_start_len=-1
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
                        "temperature": 1.0,
                        "max_new_tokens": max_new_tokens,
                        "ignore_eos": True,
                    },
                    "return_logprob": return_logprob,
                    "return_text_in_logprobs": True,
                    "logprob_start_len": logprob_start_len,
                },
            )
            return response.json()

        prompt = "I have a very good idea on how to"

        gen = run_generate(prompt, return_logprob=True, logprob_start_len=0)
        output_logprobs = np.array(
            [x[0] for x in gen["meta_info"]["output_token_logprobs"]]
        )
        num_prompts_tokens = gen["meta_info"]["prompt_tokens"]

        input_tokens = [x[1] for x in gen["meta_info"]["input_token_logprobs"]]
        output_tokens = [x[1] for x in gen["meta_info"]["output_token_logprobs"]]

        new_prompt = input_tokens + output_tokens
        score = run_generate(
            new_prompt, return_logprob=True, logprob_start_len=0, max_new_tokens=0
        )
        output_logprobs_score = np.array(
            [
                x[0]
                for x in score["meta_info"]["input_token_logprobs"][num_prompts_tokens:]
            ]
        )

        print(f"{output_logprobs[-10:]=}")
        print(f"{output_logprobs_score[-10:]=}")

        diff = np.abs(output_logprobs - output_logprobs_score)
        max_diff = np.max(diff)
        self.assertLess(max_diff, 0.35)
```
**EN:** This test method exercises logprob match and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 logprob match 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 251-279: Run test: logprob mixed
```python
    def test_logprob_mixed(self):
        args = []
        temperature = 0
        # input_len, output_len, temperature, logprob_start_len, return_logprob, top_logprobs_num
        for input_len in [1000, 5000, 10000, 50000]:
            for output_len in [4, 8]:
                for logprob_start_len in [0, 500, 2500, 5000, 25000]:
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
**EN:** This test method exercises logprob mixed and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 logprob mixed 场景，并验证观测到的行为是否符合预期契约。

### Lines 281-314: Run test: logprob grammar
```python
    def test_logprob_grammar(self):
        prompts = "Question: Is Paris the Capital of France? Answer:"
        allowed_tokens = [" Yes", " No"]

        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": prompts,
                "sampling_params": {
                    "temperature": 1.0,
                    "max_new_tokens": 1,
                    "regex": "( Yes| No)",
                },
                "return_logprob": True,
                "top_logprobs_num": 5,  # The grammar constraint allows all prefix tokens so we need to use a larger top_k.
                "return_text_in_logprobs": True,
            },
        )
        response_json = response.json()
        output_top_logprobs = response_json["meta_info"]["output_top_logprobs"][0]
        print(f"{output_top_logprobs=}")

        # Parse results
        # This is because the grammar constraint allows all prefix tokens
        logprobs = [None] * 2
        for i in range(len(output_top_logprobs)):
            try:
                idx = allowed_tokens.index(output_top_logprobs[i][2])
            except ValueError:
                # Not found
                continue
            logprobs[idx] = output_top_logprobs[i][0]

        self.assertTrue(all(x is not None for x in logprobs))
```
**EN:** This test method exercises logprob grammar and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 logprob grammar 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 316-370: Define helper: run custom logit processor
```python
    def run_custom_logit_processor(self, target_token_id: Optional[int] = None):
        """Test custom logit processor with custom params.

        If target_token_id is None, the custom logit processor won't be passed in.
        """

        custom_params = {"token_id": target_token_id}

        class DeterministicLogitProcessor(CustomLogitProcessor):
            """A dummy logit processor that changes the logits to always
            sample the given token id.
            """

            def __call__(self, logits, custom_param_list):
                assert logits.shape[0] == len(custom_param_list)
                key = "token_id"

                for i, param_dict in enumerate(custom_param_list):
                    # Mask all other tokens
                    logits[i, :] = -float("inf")
                    # Assign highest probability to the specified token
                    logits[i, param_dict[key]] = 0.0
                return logits

        prompts = "Question: Is Paris the Capital of France? Answer:"

        # Base case json data to be posted to the server.
        base_json = {
            "text": prompts,
            "sampling_params": {"temperature": 0.0},
            "return_logprob": True,
        }

        # Custom json data with custom logit processor and params.
        custom_json = base_json.copy()
        # Only set the custom logit processor if target_token_id is not None.
        if target_token_id is not None:
            custom_json["custom_logit_processor"] = DeterministicLogitProcessor.to_str()
            custom_json["sampling_params"]["custom_params"] = custom_params

        custom_response = requests.post(
            self.base_url + "/generate",
            json=custom_json,
        ).json()

        output_token_logprobs = custom_response["meta_info"]["output_token_logprobs"]
        sampled_tokens = [x[1] for x in output_token_logprobs]

        # The logit processor should always sample the given token as the logits is deterministic.
        if target_token_id is not None:
            self.assertTrue(
                all(x == custom_params["token_id"] for x in sampled_tokens),
                # Print the detailed test case info if the test fails.
                f"{target_token_id=}\n{sampled_tokens=}\n{custom_response=}",
            )
```
**EN:** This helper function encapsulates reusable logic inside `TestSRTEndpoint` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestSRTEndpoint` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 372-427: Define helper: run stateful custom logit processor (part 1)
```python
    def run_stateful_custom_logit_processor(
        self, first_token_id: int | None, delay: int = 2
    ):
        """Test custom logit processor with custom params and state.

        Should sample the first `delay` tokens normally, then output first_token_id and consecutive tokens after that.
        If first_token_id is None, the custom logit processor won't be passed in.
        """
        custom_params = {"token_id": first_token_id, "delay": 2}

        class DeterministicStatefulLogitProcessor(CustomLogitProcessor):
            """A dummy logit processor that changes the logits to always
            sample the given token id.
            """

            def __call__(self, logits, custom_param_list):
                assert logits.shape[0] == len(custom_param_list)

                for i, param_dict in enumerate(custom_param_list):
                    if param_dict["delay"] > 0:
                        param_dict["delay"] -= 1
                        continue
                    if param_dict["delay"] == 0:
                        param_dict["delay"] -= 1
                        force_token = param_dict["token_id"]
                    else:
                        output_ids = param_dict["__req__"].output_ids
                        force_token = output_ids[-1] + 1
                    # Mask all other tokens
                    logits[i, :] = -float("inf")
                    # Assign highest probability to the specified token
                    logits[i, force_token] = 0.0
                return logits

        prompts = "Question: Is Paris the Capital of France? Answer:"

        # Base case json data to be posted to the server.
        base_json = {
            "text": prompts,
            "sampling_params": {"temperature": 0.0},
            "return_logprob": True,
        }

        # Custom json data with custom logit processor and params.
        custom_json = base_json.copy()
        # Only set the custom logit processor if target_token_id is not None.
        if first_token_id is not None:
            custom_json["custom_logit_processor"] = (
                DeterministicStatefulLogitProcessor().to_str()
            )
            custom_json["sampling_params"]["custom_params"] = custom_params

        custom_response = requests.post(
            self.base_url + "/generate",
            json=custom_json,
        ).json()
```
**EN:** This helper function encapsulates reusable logic inside `TestSRTEndpoint` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestSRTEndpoint` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 429-440: Define helper: run stateful custom logit processor (part 2)
```python
        output_token_logprobs = custom_response["meta_info"]["output_token_logprobs"]
        sampled_tokens = [x[1] for x in output_token_logprobs]
        # The logit processor should always sample the given token as the logits is deterministic.
        if first_token_id is not None:
            self.assertTrue(
                all(
                    x == custom_params["token_id"] + k
                    for k, x in enumerate(sampled_tokens[custom_params["delay"] :])
                ),
                # Print the detailed test case info if the test fails.
                f"{first_token_id=}\n{sampled_tokens=}\n{custom_response=}",
            )
```
**EN:** This helper function encapsulates reusable logic inside `TestSRTEndpoint` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestSRTEndpoint` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 442-444: Run test: custom logit processor
```python
    def test_custom_logit_processor(self):
        """Test custom logit processor with a single request."""
        self.run_custom_logit_processor(target_token_id=5)
```
**EN:** This test method exercises custom logit processor and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 custom logit processor 场景，并验证观测到的行为是否符合预期契约。

### Lines 446-451: Run test: custom logit processor batch mixed
```python
    def test_custom_logit_processor_batch_mixed(self):
        """Test a batch of requests mixed of requests with and without custom logit processor."""
        target_token_ids = list(range(32)) + [None] * 16
        random.shuffle(target_token_ids)
        with ThreadPoolExecutor(len(target_token_ids)) as executor:
            list(executor.map(self.run_custom_logit_processor, target_token_ids))
```
**EN:** This test method exercises custom logit processor batch mixed and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 custom logit processor batch mixed 场景，并验证观测到的行为是否符合预期契约。

### Lines 453-466: Run test: stateful custom logit processor
```python
    @unittest.skip("Skip this test because this feature has a bug. See comments below.")
    def test_stateful_custom_logit_processor(self):
        """Test custom logit processor with a single request."""

        """
        NOTE: This feature has a race condition bug.
        This line https://github.com/sgl-project/sglang/blob/ef8ec07b2ce4c70c2a33ec5acda4ce529bc3cda4/test/srt/test_srt_endpoint.py#L395-L396 can be accessed by two concurrent threads at the same time. The access order is not guaranteed.
        In sglang, we use two python threads to overlap the GPU computation and CPU scheduling.
        Thread 1 (the CPU scheduling thread) will update the `param_dict["__req__"].output_ids`.
        Thread 2 (the GPU computation thread) will call `DeterministicStatefulLogitProcessor` because sampling is considered as GPU computation.
        We can fix this by moving the call of DeterministicStatefulLogitProcessor to the CPU scheduling thread.
        """

        self.run_stateful_custom_logit_processor(first_token_id=5)
```
**EN:** This test method exercises stateful custom logit processor and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 stateful custom logit processor 场景，并验证观测到的行为是否符合预期契约。

### Lines 468-476: Run test: stateful custom logit processor batch mixed
```python
    @unittest.skip("Skip this test because this feature has a bug. See comments above.")
    def test_stateful_custom_logit_processor_batch_mixed(self):
        """Test a batch of requests mixed of requests with and without custom logit processor."""
        target_token_ids = list(range(32)) + [None] * 16
        random.shuffle(target_token_ids)
        with ThreadPoolExecutor(len(target_token_ids)) as executor:
            list(
                executor.map(self.run_stateful_custom_logit_processor, target_token_ids)
            )
```
**EN:** This test method exercises stateful custom logit processor batch mixed and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 stateful custom logit processor batch mixed 场景，并验证观测到的行为是否符合预期契约。

### Lines 478-501: Run test: cache tokens
```python
    def test_cache_tokens(self):
        for _ in range(2):
            time.sleep(1)
            response = requests.post(self.base_url + "/flush_cache")
            assert response.status_code == 200

        def send_and_check_cached_tokens(input_ids):
            response = requests.post(
                self.base_url + "/generate",
                json={
                    "input_ids": list(input_ids),
                    "sampling_params": {
                        "max_new_tokens": 1,
                    },
                },
            )
            response_json = response.json()
            return response_json["meta_info"]["cached_tokens"]

        self.assertEqual(send_and_check_cached_tokens(range(0, 100)), 0)
        self.assertEqual(send_and_check_cached_tokens(range(0, 10000)), 100)
        self.assertEqual(send_and_check_cached_tokens(range(0, 10000)), 9999)
        self.assertEqual(send_and_check_cached_tokens(range(0, 1000)), 999)
        self.assertEqual(send_and_check_cached_tokens(range(0, 11000)), 10000)
```
**EN:** This test method exercises cache tokens and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 cache tokens 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 503-511: Run test: get server info
```python
    def test_get_server_info(self):
        response = requests.get(self.base_url + "/server_info")
        response_json = response.json()

        max_total_num_tokens = response_json["max_total_num_tokens"]
        self.assertIsInstance(max_total_num_tokens, int)

        version = response_json["version"]
        self.assertIsInstance(version, str)
```
**EN:** This test method exercises get server info and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 get server info 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 513-541: Run test: logit bias
```python
    def test_logit_bias(self):
        """Test that a very high logit bias forces sampling of a specific token."""
        # Choose a token ID to bias (using 5 as an example)
        target_token_id = 60704  # Paris for meta-llama/Llama-3.2-1B-Instruct, DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        logit_bias = {str(target_token_id): 100.0}  # Very high positive bias

        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 1.0,  # Use high temperature to encourage exploration
                    "max_new_tokens": 4,
                    "logit_bias": logit_bias,
                },
                "return_logprob": True,
            },
        )
        response_json = response.json()

        # Extract the sampled token IDs from the output
        output_token_logprobs = response_json["meta_info"]["output_token_logprobs"]
        sampled_tokens = [x[1] for x in output_token_logprobs]

        # Verify that all sampled tokens are the target token
        self.assertTrue(
            all(x == target_token_id for x in sampled_tokens),
            f"Expected all tokens to be {target_token_id}, but got {sampled_tokens}",
        )
```
**EN:** This test method exercises logit bias and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 logit bias 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 543-574: Run test: forbidden token
```python
    def test_forbidden_token(self):
        """Test that a forbidden token (very negative logit bias) doesn't appear in the output."""
        # Choose a token ID to forbid (using 10 as an example)
        forbidden_token_id = 23994  # rice for meta-llama/Llama-3.2-1B-Instruct, DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        logit_bias = {
            str(forbidden_token_id): -100.0
        }  # Very negative bias to forbid the token

        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": "Only output 'rice' exactly like this, in lowercase ONLY: rice",
                "sampling_params": {
                    "temperature": 1.0,  # Use high temperature to encourage diverse output
                    "max_new_tokens": 50,  # Generate enough tokens to likely include numbers
                    "logit_bias": logit_bias,
                },
                "return_logprob": True,
            },
        )
        response_json = response.json()

        # Extract the sampled token IDs from the output
        output_token_logprobs = response_json["meta_info"]["output_token_logprobs"]
        sampled_tokens = [x[1] for x in output_token_logprobs]

        # Verify that the forbidden token doesn't appear in the output
        self.assertNotIn(
            forbidden_token_id,
            sampled_tokens,
            f"Expected forbidden token {forbidden_token_id} not to be present, but it was found",
        )
```
**EN:** This test method exercises forbidden token and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 forbidden token 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 576-627: Run test: logit bias isolation
```python
    def test_logit_bias_isolation(self):
        """Test that logit_bias applied to one request doesn't affect other requests in batch."""
        # Choose a token ID to bias in first request only
        biased_token_id = 60704  # Paris for meta-llama/Llama-3.2-1B-Instruct, DEFAULT_SMALL_MODEL_NAME_FOR_TEST

        # Prepare batch requests - one with logit_bias and one without
        requests_data = [
            {
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 1.0,
                    "max_new_tokens": 4,
                    "logit_bias": {str(biased_token_id): 100.0},  # Strong bias
                },
                "return_logprob": True,
            },
            {
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 1.0,
                    "max_new_tokens": 4,
                },
                "return_logprob": True,
            },
        ]

        # Send both requests
        responses = []
        for req in requests_data:
            response = requests.post(self.base_url + "/generate", json=req)
            responses.append(response.json())

        # Extract token IDs from each response
        biased_tokens = [
            x[1] for x in responses[0]["meta_info"]["output_token_logprobs"]
        ]
        unbiased_tokens = [
            x[1] for x in responses[1]["meta_info"]["output_token_logprobs"]
        ]

        # Verify first response contains only biased tokens
        self.assertTrue(
            all(x == biased_token_id for x in biased_tokens),
            f"Expected all tokens to be {biased_token_id} in first response, but got {biased_tokens}",
        )

        # Verify second response contains at least some different tokens
        # (We can't guarantee exactly what tokens will be generated, but they shouldn't all be the biased token)
        self.assertTrue(
            any(x != biased_token_id for x in unbiased_tokens),
            f"Expected some tokens to be different from {biased_token_id} in second response, but got {unbiased_tokens}",
        )
```
**EN:** This test method exercises logit bias isolation and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 logit bias isolation 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 629-642: Run test: get server info concurrent
```python
    def test_get_server_info_concurrent(self):
        """Make sure the concurrent get_server_info doesn't crash the server."""
        tp = ThreadPoolExecutor(max_workers=30)

        def s():
            server_info = requests.get(self.base_url + "/server_info")
            server_info.json()

        futures = []
        for _ in range(4):
            futures.append(tp.submit(s))

        for f in futures:
            f.result()
```
**EN:** This test method exercises get server info concurrent and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 get server info concurrent 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 650-650: Define class TestTokenizeDetokenize
```python
class TestTokenizeDetokenize(CustomTestCase):
```
**EN:** This declaration introduces the `TestTokenizeDetokenize` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestTokenizeDetokenize` 测试类，并说明它通过继承承担的职责。

### Lines 651-664: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.tokenize_url = f"{cls.base_url}/tokenize"
        cls.openai_tokenize_url = f"{cls.base_url}/v1/tokenize"
        cls.detokenize_url = f"{cls.base_url}/detokenize"
        cls.session = requests.Session()
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        )
        cls.tokenizer = get_tokenizer(cls.model)
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario and talks to the server through its HTTP interface.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务，并通过 HTTP 接口与服务交互。

### Lines 666-669: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        cls.session.close()
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 671-674: Define helper: post json
```python
    def _post_json(self, url, payload):
        r = self.session.post(url, json=payload)
        r.raise_for_status()
        return r.json()
```
**EN:** This helper function encapsulates reusable logic inside `TestTokenizeDetokenize` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestTokenizeDetokenize` 内部调用，从而让场景结构更清晰。

### Lines 676-703: Run test: tokenize various inputs
```python
    def test_tokenize_various_inputs(self):
        single = "Hello SGLang world! 123 😊, ಪರ್ವತದ ಮೇಲೆ ಹಿಮ."
        multi = ["First sentence.", "Second, with 中文."]
        scenarios = [
            {"prompt": single, "add_special_tokens": True},
            {"prompt": single, "add_special_tokens": False},
            {"prompt": multi, "add_special_tokens": True},
            {"prompt": multi, "add_special_tokens": False},
            {"prompt": "", "add_special_tokens": False},
        ]
        for case in scenarios:
            payload = {"model": self.model, "prompt": case["prompt"]}
            if "add_special_tokens" in case:
                payload["add_special_tokens"] = case["add_special_tokens"]
            resp = self._post_json(self.tokenize_url, payload)
            tokens = resp["tokens"]
            count = resp["count"]
            self.assertIsInstance(tokens, list)
            if not tokens:
                self.assertEqual(count, 0)
            else:
                if isinstance(tokens[0], list):
                    total = sum(len(t) for t in tokens)
                    expected = sum(count) if isinstance(count, list) else count
                else:
                    total = len(tokens)
                    expected = count
                self.assertEqual(total, expected)
```
**EN:** This test method exercises tokenize various inputs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 tokenize various inputs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 705-709: Run test: tokenize invalid type
```python
    def test_tokenize_invalid_type(self):
        r = self.session.post(
            self.tokenize_url, json={"model": self.model, "prompt": 12345}
        )
        self.assertEqual(r.status_code, 400)
```
**EN:** This test method exercises tokenize invalid type and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 tokenize invalid type 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 711-761: Run test: openai tokenize chat messages
```python
    def test_openai_tokenize_chat_messages(self):
        messages = [{"role": "user", "content": "What is the weather in Paris?"}]
        resp = self._post_json(
            self.openai_tokenize_url,
            {"model": self.model, "messages": messages},
        )
        expected_tokens = self.tokenizer.apply_chat_template(
            messages,
            tokenize=True,
            add_generation_prompt=True,
        )
        if not isinstance(expected_tokens, list):
            expected_tokens = expected_tokens["input_ids"]
        if hasattr(expected_tokens, "tolist"):
            expected_tokens = expected_tokens.tolist()
        self.assertEqual(resp["tokens"], expected_tokens)
        self.assertEqual(resp["count"], len(expected_tokens))

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "get_weather",
                    "description": "Get weather for a city.",
                    "parameters": {
                        "type": "object",
                        "properties": {"city": {"type": "string"}},
                        "required": ["city"],
                    },
                },
            }
        ]
        tools_resp = self._post_json(
            self.openai_tokenize_url,
            {"model": self.model, "messages": messages, "tools": tools},
        )
        self.assertIsInstance(tools_resp["tokens"], list)
        self.assertEqual(tools_resp["count"], len(tools_resp["tokens"]))
        self.assertNotEqual(tools_resp["tokens"], resp["tokens"])

        no_tools_resp = self._post_json(
            self.openai_tokenize_url,
            {
                "model": self.model,
                "messages": messages,
                "tools": tools,
                "tool_choice": "none",
            },
        )
        self.assertEqual(no_tools_resp["tokens"], resp["tokens"])
        self.assertEqual(no_tools_resp["count"], resp["count"])
```
**EN:** This test method exercises openai tokenize chat messages and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 openai tokenize chat messages 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 763-788: Run test: detokenize roundtrip
```python
    def test_detokenize_roundtrip(self):
        text = "Verify detokenization round trip. यह डिटोकेनाइजेशन है"
        t0 = self._post_json(
            self.tokenize_url,
            {"model": self.model, "prompt": text, "add_special_tokens": False},
        )["tokens"]
        t1 = self._post_json(
            self.tokenize_url,
            {"model": self.model, "prompt": text, "add_special_tokens": True},
        )["tokens"]
        cases = [
            {"tokens": t0, "skip_special_tokens": True, "expected": text},
            {"tokens": t1, "skip_special_tokens": True, "expected": text},
            {"tokens": t1, "skip_special_tokens": False, "expected": None},
            {"tokens": [], "skip_special_tokens": True, "expected": ""},
        ]
        for case in cases:
            payload = {"model": self.model, "tokens": case["tokens"]}
            if "skip_special_tokens" in case:
                payload["skip_special_tokens"] = case["skip_special_tokens"]
            resp = self._post_json(self.detokenize_url, payload)
            text_out = resp["text"]
            if case["expected"] is not None:
                self.assertEqual(text_out, case["expected"])
            else:
                self.assertIsInstance(text_out, str)
```
**EN:** This test method exercises detokenize roundtrip and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 detokenize roundtrip 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 790-798: Run test: detokenize invalid tokens
```python
    def test_detokenize_invalid_tokens(self):
        r = self.session.post(
            self.detokenize_url, json={"model": self.model, "tokens": ["a", "b"]}
        )
        self.assertEqual(r.status_code, 400)
        r2 = self.session.post(
            self.detokenize_url, json={"model": self.model, "tokens": [1, -1, 2]}
        )
        self.assertEqual(r2.status_code, 500)
```
**EN:** This test method exercises detokenize invalid tokens and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 detokenize invalid tokens 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 801-802: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Server lifecycle helpers start the target runtime before assertions begin. / 服务生命周期辅助函数会在断言开始前启动目标运行时。
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- HTTP requests validate the externally visible API contract rather than only internal helpers. / HTTP 请求验证的是对外暴露的 API 契约，而不只是内部辅助函数。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.sampling.custom_logit_processor`, `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `concurrent.futures`, `functools`, `json`, `numpy`, `random`, `requests`, `time`, `typing`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.get`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
