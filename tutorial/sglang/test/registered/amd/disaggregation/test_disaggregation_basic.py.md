# test_disaggregation_basic.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/disaggregation/test_disaggregation_basic.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on disaggregation basic in SGLang. It interacts with externally visible endpoints and verifies the returned behavior. / 该测试模块用于分析 SGLang 中与 disaggregation basic 相关的实现或行为。 它会与对外暴露的端点交互，并验证返回结果是否正确。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Import dependencies
```python
import json
import os
import unittest
from types import SimpleNamespace

import openai
import requests
from transformers import AutoTokenizer

from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.few_shot_gsm8k import run_eval as run_eval_few_shot_gsm8k
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    popen_launch_pd_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also runs an evaluation workflow to measure model quality and registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会运行评测流程以衡量模型质量，并将该用例注册到 AMD CI 覆盖范围。

### Lines 21-21: Register CI metadata
```python
register_amd_ci(est_time=600, suite="stage-b-test-large-8-gpu-35x-disaggregation-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 24-24: Define class TestDisaggregationAccuracy
```python
class TestDisaggregationAccuracy(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationAccuracy` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationAccuracy` 测试类，并说明它通过继承承担的职责。

### Lines 25-50: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Configure ROCm RDMA environment
        os.environ["SGLANG_USE_AITER"] = "1"
        rdma_env = os.environ.get("SGLANG_TEST_RDMA_DEVICE")

        if rdma_env:
            cls.rdma_devices = ["--disaggregation-ib-device", rdma_env]
            print(f"Found RDMA devices in env: {rdma_env}")
        else:
            print("SGLANG_TEST_RDMA_DEVICE is not set! Running without RDMA.")
            cls.rdma_devices = []

        cls.model = DEFAULT_MODEL_NAME_FOR_TEST
        # DEFAULT_MODEL_NAME_FOR_TEST

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 52-73: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--attention-backend",
            "aiter",
            "--log-level",
            "debug",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationAccuracy` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationAccuracy` 内部调用，从而让场景结构更清晰。

### Lines 75-102: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
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
            "--attention-backend",
            "aiter",
            "--mem-fraction-static",
            "0.8",
            "--log-level",
            "debug",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        print("Debug")
        print(decode_args)
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationAccuracy` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationAccuracy` 内部调用，从而让场景结构更清晰。

### Lines 104-117: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host=f"http://{self.base_host}",
            port=int(self.lb_port),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"Evaluation metrics: {metrics}")

        self.assertGreater(metrics["accuracy"], 0.70)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 119-142: Run test: logprob
```python
    def test_logprob(self):
        prompt = "The capital of france is "
        response = requests.post(
            self.lb_url + "/generate",
            json={
                "text": prompt,
                "sampling_params": {"temperature": 0},
                "return_logprob": True,
                "return_input_logprob": True,
                "logprob_start_len": 0,
            },
        )

        j = response.json()
        completion_tokens = j["meta_info"]["completion_tokens"]
        input_logprobs = j["meta_info"]["input_token_logprobs"]
        output_logprobs = j["meta_info"]["output_token_logprobs"]

        assert (
            len(output_logprobs) == completion_tokens
        ), f"output_logprobs and completion_tokens should have the same length, but got {len(output_logprobs)} and {completion_tokens}"
        assert (
            len(input_logprobs) > 0
        ), f"input_logprobs should have at least one token, but got {len(input_logprobs)}"
```
**EN:** This test method exercises logprob and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 logprob 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 144-170: Run test: structured output
```python
    def test_structured_output(self):
        json_schema = json.dumps(
            {
                "type": "object",
                "properties": {
                    "name": {"type": "string", "pattern": "^[\\w]+$"},
                    "population": {"type": "integer"},
                },
                "required": ["name", "population"],
            }
        )

        # JSON
        response = requests.post(
            f"{self.lb_url}/generate",
            json={
                "text": "Here is the information of the capital of France in the JSON format.\n",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 64,
                    "json_schema": json_schema,
                },
            },
        )
        output = response.json()["text"]
        # ensure the output is a valid JSON
        json.loads(output)
```
**EN:** This test method exercises structured output and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 structured output 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 172-216: Run test: first token finish
```python
    def test_first_token_finish(self):
        client = openai.Client(api_key="empty", base_url=f"{self.lb_url}/v1")
        tokenizer = AutoTokenizer.from_pretrained(self.model)
        eos_token = tokenizer.eos_token_id
        prompt = "The best programming language for AI is"

        # First token EOS
        res = client.completions.create(
            model="dummy", prompt=prompt, logit_bias={eos_token: 42}
        ).model_dump()
        print(f"{res=}")

        assert res["usage"]["completion_tokens"] == 1, (
            "Expected completion_tokens to be 1 when first token is EOS, "
            f"but got {res['usage']['completion_tokens']}"
        )

        # First token EOS with ignore_eos
        res = client.completions.create(
            model="dummy",
            prompt=prompt,
            logit_bias={eos_token: 42},
            extra_body={"ignore_eos": True},
        ).model_dump()
        print(f"{res=}")

        assert res["usage"]["completion_tokens"] > 1, (
            "Expected completion_tokens to be greater than 1 when ignore_eos is True, "
            f"but got {res['usage']['completion_tokens']}"
        )

        # First token with specified stop token
        stop_token_id = tokenizer.encode(" hello", add_special_tokens=False)[0]
        res = client.completions.create(
            model="dummy",
            prompt=prompt,
            logit_bias={stop_token_id: 42},
            stop=[" hello"],
        ).model_dump()
        print(f"{res=}")

        assert res["usage"]["completion_tokens"] == 1, (
            "Expected completion_tokens to be 1 when first token is stop token, "
            f"but got {res['usage']['completion_tokens']}"
        )
```
**EN:** This test method exercises first token finish and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 first token finish 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 220-220: Define class TestDisaggregationMooncakeFailure
```python
class TestDisaggregationMooncakeFailure(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationMooncakeFailure` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationMooncakeFailure` 测试类，并说明它通过继承承担的职责。

### Lines 221-248: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Configure ROCm RDMA environment
        os.environ["SGLANG_USE_AITER"] = "1"
        rdma_env = os.environ.get("SGLANG_TEST_RDMA_DEVICE")

        if rdma_env:
            cls.rdma_devices = ["--disaggregation-ib-device", rdma_env]
            print(f"Found RDMA devices in env: {rdma_env}")
        else:
            print("SGLANG_TEST_RDMA_DEVICE is not set! Running without RDMA.")
            cls.rdma_devices = []

        # set DISAGGREGATION_TEST_FAILURE_PROB to simulate failure
        os.environ["DISAGGREGATION_TEST_FAILURE_PROB"] = "0.05"

        cls.model = DEFAULT_MODEL_NAME_FOR_TEST

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 250-253: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        os.environ.pop("DISAGGREGATION_TEST_FAILURE_PROB")
        super().tearDownClass()
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。

### Lines 255-276: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--attention-backend",
            "aiter",
            "--log-level",
            "debug",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakeFailure` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakeFailure` 内部调用，从而让场景结构更清晰。

### Lines 278-303: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
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
            "--attention-backend",
            "aiter",
            "--mem-fraction-static",
            "0.8",
            "--log-level",
            "debug",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationMooncakeFailure` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationMooncakeFailure` 内部调用，从而让场景结构更清晰。

### Lines 305-330: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host=f"http://{self.base_host}",
            port=int(self.lb_port),
        )

        # Expect lots of failure but the server cannot crash
        try:
            metrics = run_eval_few_shot_gsm8k(args)
            print(f"Evaluation metrics: {metrics}")
        except Exception as e:
            print(f"Test encountered expected errors: {e}")
            # Check if servers are still healthy
            try:
                response = requests.get(self.prefill_url + "/health_generate")
                assert response.status_code == 200
                response = requests.get(self.decode_url + "/health_generate")
                assert response.status_code == 200
            except Exception as health_check_error:
                # If health check fails, re-raise the original exception
                raise e from health_check_error
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 334-334: Define class TestDisaggregationSimulatedRetract
```python
class TestDisaggregationSimulatedRetract(PDDisaggregationServerBase):
```
**EN:** This declaration introduces the `TestDisaggregationSimulatedRetract` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDisaggregationSimulatedRetract` 测试类，并说明它通过继承承担的职责。

### Lines 335-360: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        # Configure ROCm RDMA environment
        os.environ["SGLANG_USE_AITER"] = "1"
        rdma_env = os.environ.get("SGLANG_TEST_RDMA_DEVICE")

        if rdma_env:
            cls.rdma_devices = ["--disaggregation-ib-device", rdma_env]
            print(f"Found RDMA devices in env: {rdma_env}")
        else:
            print("SGLANG_TEST_RDMA_DEVICE is not set! Running without RDMA.")
            cls.rdma_devices = []

        os.environ["SGLANG_TEST_RETRACT"] = "true"
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST

        # Non blocking start servers
        cls.start_prefill()
        cls.start_decode()

        # Block until both
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 362-365: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        os.environ.pop("SGLANG_TEST_RETRACT")
        super().tearDownClass()
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。

### Lines 367-388: Define helper: start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--attention-backend",
            "aiter",
            "--log-level",
            "debug",
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_pd_server(
            cls.model,
            cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationSimulatedRetract` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationSimulatedRetract` 内部调用，从而让场景结构更清晰。

### Lines 390-415: Define helper: start decode
```python
    @classmethod
    def start_decode(cls):
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
            "--attention-backend",
            "aiter",
            "--mem-fraction-static",
            "0.8",
            "--log-level",
            "debug",
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_pd_server(
            cls.model,
            cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDisaggregationSimulatedRetract` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDisaggregationSimulatedRetract` 内部调用，从而让场景结构更清晰。

### Lines 417-430: Run test: gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            num_shots=5,
            data_path=None,
            num_questions=200,
            max_new_tokens=512,
            parallel=128,
            host=f"http://{self.base_host}",
            port=int(self.lb_port),
        )
        metrics = run_eval_few_shot_gsm8k(args)
        print(f"Evaluation metrics: {metrics}")

        self.assertGreater(metrics["accuracy"], 0.70)
```
**EN:** This test method exercises gsm8k and verifies that the observed behavior matches the expected contract. It also runs an evaluation workflow to measure model quality and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 gsm8k 场景，并验证观测到的行为是否符合预期契约。 其中还会运行评测流程以衡量模型质量，并通过断言检查明确的预期。

### Lines 433-434: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- HTTP requests validate the externally visible API contract rather than only internal helpers. / HTTP 请求验证的是对外暴露的 API 契约，而不只是内部辅助函数。
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `json`, `openai`, `os`, `requests`, `transformers`, `types`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `run_eval`, `SimpleNamespace`, `requests.get`, `requests.post`, `unittest.main`, `DEFAULT_MODEL_NAME_FOR_TEST`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
