# test_npu_api.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/interface/test_npu_api.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on interface npu api in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 interface npu api 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Import dependencies
```python
import json
import logging
import os
import shutil
import unittest

import requests
from transformers import AutoTokenizer

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ascend.test_ascend_utils import LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 22-24: Define module constants
```python
GLOBAL_SERVER_PROCESS = None
GLOBAL_SERVER_INITIALIZED = False
OUTPUT_DIR = "./profiler_dir"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 26-26: Register CI metadata
```python
register_npu_ci(est_time=1600, suite="nightly-npu-a3-merged", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 29-29: Define class TestNpuApi
```python
class TestNpuApi(CustomTestCase):
```
**EN:** This declaration introduces the `TestNpuApi` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNpuApi` 测试类，并说明它通过继承承担的职责。

### Lines 30-34: Document the class `TestNpuApi`
```python
    """Testcase: Verify that the basic functions of the API interfaces work properly and the returned parameters are consistent with the configurations.

    [Test Category] Interface
    [Test Target] /health; /health_generate; /ping; /model_info; /server_info; /v1/loads; /v1/models; /v1/models/{model:path}; /generate
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNpuApi`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNpuApi`的设计意图。

### Lines 36-55: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        global GLOBAL_SERVER_PROCESS, GLOBAL_SERVER_INITIALIZED
        # Start server only if not initialized
        if not GLOBAL_SERVER_INITIALIZED:
            cls.model = LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
            other_args = [
                "--attention-backend",
                "ascend",
                "--enable-return-hidden-states",
            ]
            # Start server and save to global variable
            GLOBAL_SERVER_PROCESS = popen_launch_server(
                cls.model,
                DEFAULT_URL_FOR_TEST,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=other_args,
            )
            GLOBAL_SERVER_INITIALIZED = True
            cls.base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 57-60: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        # First class does not terminate server
        pass
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。

### Lines 62-64: Run test: api health
```python
    def test_api_health(self):
        response = requests.get(f"{self.base_url}/health")
        self.assertEqual(response.status_code, 200)
```
**EN:** This test method exercises api health and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api health 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 66-68: Run test: api health generate
```python
    def test_api_health_generate(self):
        response = requests.get(f"{self.base_url}/health_generate")
        self.assertEqual(response.status_code, 200)
```
**EN:** This test method exercises api health generate and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api health generate 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 70-72: Run test: api ping
```python
    def test_api_ping(self):
        response = requests.get(f"{self.base_url}/ping")
        self.assertEqual(response.status_code, 200)
```
**EN:** This test method exercises api ping and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api ping 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 74-85: Run test: api model info
```python
    def test_api_model_info(self):
        response = requests.get(f"{self.base_url}/model_info")
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.json()["model_path"], self.model)
        self.assertEqual(response.json()["tokenizer_path"], self.model)
        self.assertTrue(response.json()["is_generation"])
        self.assertIsNone(response.json()["preferred_sampling_params"])
        self.assertEqual(response.json()["weight_version"], "default")
        self.assertFalse(response.json()["has_image_understanding"])
        self.assertFalse(response.json()["has_audio_understanding"])
        self.assertEqual(response.json()["model_type"], "llama")
        self.assertEqual(response.json()["architectures"][0], "LlamaForCausalLM")
```
**EN:** This test method exercises api model info and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api model info 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 87-91: Run test: api server info
```python
    def test_api_server_info(self):
        response = requests.get(f"{self.base_url}/server_info")
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.json()["model_path"], self.model)
        self.assertEqual(response.json()["tokenizer_path"], self.model)
```
**EN:** This test method exercises api server info and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api server info 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 93-104: Run test: api v1 loads
```python
    def test_api_v1_loads(self):
        response = requests.get(f"{self.base_url}/v1/loads")
        self.assertEqual(response.status_code, 200)
        body = response.json()
        self.assertIn("loads", body)
        self.assertIn("aggregate", body)
        self.assertGreaterEqual(len(body["loads"]), 1)
        load = body["loads"][0]
        self.assertGreaterEqual(load["num_running_reqs"], 0)
        self.assertGreaterEqual(load["num_waiting_reqs"], 0)
        self.assertGreaterEqual(load["num_used_tokens"], 0)
        self.assertGreaterEqual(load["num_total_tokens"], 0)
```
**EN:** This test method exercises api v1 loads and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api v1 loads 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 106-113: Run test: api v1 models
```python
    def test_api_v1_models(self):
        response = requests.get(f"{self.base_url}/v1/models")
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.json()["data"][0]["id"], self.model)
        self.assertEqual(response.json()["data"][0]["object"], "model")
        self.assertEqual(response.json()["data"][0]["owned_by"], "sglang")
        self.assertEqual(response.json()["data"][0]["root"], self.model)
        self.assertEqual(response.json()["data"][0]["max_model_len"], 131072)
```
**EN:** This test method exercises api v1 models and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api v1 models 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 115-122: Run test: api v1 models path
```python
    def test_api_v1_models_path(self):
        response = requests.get(f"{self.base_url}/v1/models/{self.model}")
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.json()["id"], self.model)
        self.assertEqual(response.json()["object"], "model")
        self.assertEqual(response.json()["owned_by"], "sglang")
        self.assertEqual(response.json()["root"], self.model)
        self.assertEqual(response.json()["max_model_len"], 131072)
```
**EN:** This test method exercises api v1 models path and verifies that the observed behavior matches the expected contract. It also issues HTTP GET requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api v1 models path 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP GET 请求，并通过 HTTP 接口与服务交互。

### Lines 124-146: Run test: api generate single text
```python
    def test_api_generate_single_text(self):
        response = requests.post(
            f"{self.base_url}/generate",
            json={
                "rid": "req_001",
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 20,
                },
                "return_logprob": True,
                "stream": False,
                "return_hidden_states": True,
            },
        )
        self.assertEqual(response.status_code, 200)
        meta_info_keys = response.json()["meta_info"].keys()
        self.assertEqual("req_001", response.json()["meta_info"]["id"])
        self.assertIn("Paris", response.json()["text"])
        self.assertEqual(20, response.json()["meta_info"]["completion_tokens"])
        self.assertIn("input_token_logprobs", meta_info_keys)
        self.assertIn("output_token_logprobs", meta_info_keys)
        self.assertIn("hidden_states", meta_info_keys)
```
**EN:** This test method exercises api generate single text and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api generate single text 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 148-172: Run test: api generate batch texts
```python
    def test_api_generate_batch_texts(self):
        rids = ["req_1", "req_2"]
        texts = [
            "The capital of France is",
            "What is the best time of year to visit Japan for cherry blossoms?",
        ]
        response = requests.post(
            f"{self.base_url}/generate",
            json={
                "rid": rids,
                "text": texts,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 20,
                },
                "return_logprob": False,
                "stream": False,
                "return_hidden_states": False,
            },
        )
        self.assertEqual(response.status_code, 200)
        self.assertEqual("req_1", response.json()[0]["meta_info"]["id"])
        self.assertIn("Paris", response.json()[0]["text"])
        self.assertEqual("req_2", response.json()[1]["meta_info"]["id"])
        self.assertIn("Japan", response.json()[1]["text"])
```
**EN:** This test method exercises api generate batch texts and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api generate batch texts 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 174-199: Run test: api generate temperature
```python
    def test_api_generate_temperature(self):
        response = requests.post(
            f"{self.base_url}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 5,
                    "max_new_tokens": 20,
                },
            },
        )
        self.assertEqual(response.status_code, 200)
        text1 = response.json()["text"]
        response = requests.post(
            f"{self.base_url}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 5,
                    "max_new_tokens": 20,
                },
            },
        )
        self.assertEqual(response.status_code, 200)
        text2 = response.json()["text"]
        self.assertNotEqual(text2, text1)
```
**EN:** This test method exercises api generate temperature and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api generate temperature 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 201-230: Run test: api generate input ids
```python
    def test_api_generate_input_ids(self):
        text = "The capital of France is"
        tokenizer = AutoTokenizer.from_pretrained(self.model)
        input_ids = tokenizer(text, return_tensors="pt")["input_ids"][0].tolist()
        response = requests.post(
            f"{self.base_url}/generate",
            json={
                "rid": "req_002",
                "input_ids": input_ids,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 10,
                },
                "return_logprob": False,
                "stream": True,
                "return_hidden_states": False,
            },
        )
        self.assertEqual(response.status_code, 200)
        lines = response.text.strip().split("\n")
        self.assertGreaterEqual(len(lines), 10)
        json_data = lines[-3][6:]
        data = json.loads(json_data)
        meta_info_keys = data["meta_info"].keys()
        self.assertEqual("req_002", data["meta_info"]["id"])
        self.assertIn("Paris", data["text"])
        self.assertEqual(10, data["meta_info"]["completion_tokens"])
        self.assertNotIn("input_token_logprobs", meta_info_keys)
        self.assertNotIn("output_token_logprobs", meta_info_keys)
        self.assertNotIn("hidden_states", meta_info_keys)
```
**EN:** This test method exercises api generate input ids and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 api generate input ids 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 233-233: Define class TestChatCompletionsInterface
```python
class TestChatCompletionsInterface(CustomTestCase):
```
**EN:** This declaration introduces the `TestChatCompletionsInterface` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestChatCompletionsInterface` 测试类，并说明它通过继承承担的职责。

### Lines 234-238: Document the class `TestChatCompletionsInterface`
```python
    """Testcase: The test is to verify whether the functions of each parameter of the v1/chat/completions interface are normal.

    [Test Category] Interface
    [Test Target] v1/chat/completions
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestChatCompletionsInterface`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestChatCompletionsInterface`的设计意图。

### Lines 240-245: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        # Skip initialization, directly reuse global server
        cls.model = LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.additional_chat_kwargs = {}
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 247-250: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        # Do not terminate server
        pass
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。

### Lines 252-274: Run test: model and messages
```python
    def test_model_and_messages(self):
        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "Hello"}],
            },
        )
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        data = response.json()
        self.assertEqual(data["model"], self.model)
        self.assertIsNotNone(data["choices"][0]["message"]["reasoning_content"])

        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "messages": [{"role": "user", "content": "Hello"}],
            },
        )
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        data = response.json()
        self.assertEqual(data["model"], "default")
        self.assertIsNotNone(data["choices"][0]["message"]["reasoning_content"])
```
**EN:** This test method exercises model and messages and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 model and messages 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 276-285: Run test: max completion tokens
```python
    def test_max_completion_tokens(self):
        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "messages": [{"role": "user", "content": "Hello"}],
                "max_completion_tokens": 1,
            },
        )
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        self.assertEqual(response.json()["choices"][0]["finish_reason"], "length")
```
**EN:** This test method exercises max completion tokens and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 max completion tokens 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 287-315: Run test: stream
```python
    def test_stream(self):
        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "Hello"}],
                "stream": True,
            },
        )
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        has_reasoning = False
        has_content = False

        for line in response.iter_lines():
            if line:
                line = line.decode("utf-8")
                if line.startswith("data:") and not line.startswith("data: [DONE]"):
                    data = json.loads(line[6:])
                    if "choices" in data and len(data["choices"]) > 0:
                        delta = data["choices"][0].get("delta", {})
                        if "reasoning_content" in delta and delta["reasoning_content"]:
                            has_reasoning = True
                        if "content" in delta and delta["content"]:
                            has_content = True

        self.assertTrue(
            has_reasoning, "Reasoning content not included in stream response"
        )
        self.assertTrue(has_content, "Normal content not included in stream response")
```
**EN:** This test method exercises stream and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 stream 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 317-365: Run test: temperature (part 1)
```python
    def test_temperature(self):
        response1 = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [
                    {
                        "role": "user",
                        "content": "Please write a five-character quatrain for me.",
                    }
                ],
                "temperature": 0,
            },
        )
        self.assertEqual(response1.status_code, 200, f"Failed with: {response1.text}")
        content1 = response1.json()["choices"][0]["message"]["content"]

        response2 = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [
                    {
                        "role": "user",
                        "content": "Please write a five-character quatrain for me.",
                    }
                ],
                "temperature": 0,
            },
        )
        self.assertEqual(response2.status_code, 200, f"Failed with: {response2.text}")
        content2 = response2.json()["choices"][0]["message"]["content"]
        self.assertEqual(content1, content2)

        response3 = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [
                    {
                        "role": "user",
                        "content": "Please write a five-character quatrain for me.",
                    }
                ],
                "temperature": 2,
            },
        )
        self.assertEqual(response3.status_code, 200, f"Failed with: {response3.text}")
        content3 = response3.json()["choices"][0]["message"]["content"]
```
**EN:** This test method exercises temperature and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 temperature 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 367-382: Run test: temperature (part 2)
```python
        response4 = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [
                    {
                        "role": "user",
                        "content": "Please write a five-character quatrain for me.",
                    }
                ],
                "temperature": 2,
            },
        )
        self.assertEqual(response4.status_code, 200, f"Failed with: {response4.text}")
        content4 = response4.json()["choices"][0]["message"]["content"]
        self.assertNotEqual(content3, content4)
```
**EN:** This test method exercises temperature and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 temperature 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 384-404: Run test: return hidden states
```python
    def test_return_hidden_states(self):
        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "Hello"}],
                "return_hidden_states": True,
            },
        )
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        self.assertIn("hidden_states", response.json()["choices"][0])

        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "Hello"}],
            },
        )
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        self.assertNotIn("hidden_states", response.json()["choices"][0])
```
**EN:** This test method exercises return hidden states and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 return hidden states 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 406-438: Run test: top k
```python
    def test_top_k(self):
        response1 = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [
                    {
                        "role": "user",
                        "content": "Please write a five-character quatrain for me.",
                    }
                ],
                "top_k": 20,
            },
        )
        self.assertEqual(response1.status_code, 200, f"Failed with: {response1.text}")
        content1 = response1.json()["choices"][0]["message"]["content"]

        response2 = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [
                    {
                        "role": "user",
                        "content": "Please write a five-character quatrain for me.",
                    }
                ],
                "top_k": 20,
            },
        )
        self.assertEqual(response2.status_code, 200, f"Failed with: {response2.text}")
        content2 = response2.json()["choices"][0]["message"]["content"]
        self.assertNotEqual(content1, content2)
```
**EN:** This test method exercises top k and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 top k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 440-450: Run test: stop token ids
```python
    def test_stop_token_ids(self):
        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "Hello"}],
                "stop_token_ids": [1, 13],
            },
        )
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        self.assertEqual(response.json()["choices"][0]["matched_stop"], 13)
```
**EN:** This test method exercises stop token ids and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 stop token ids 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 452-462: Run test: rid
```python
    def test_rid(self):
        response = requests.post(
            f"{self.base_url}/v1/chat/completions",
            json={
                "model": self.model,
                "messages": [{"role": "user", "content": "Hello"}],
                "rid": "sssss",
            },
        )
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        self.assertEqual(response.json()["id"], "sssss")
```
**EN:** This test method exercises rid and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 rid 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 465-465: Define class TestEnableThinking
```python
class TestEnableThinking(CustomTestCase):
```
**EN:** This declaration introduces the `TestEnableThinking` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestEnableThinking` 测试类，并说明它通过继承承担的职责。

### Lines 466-470: Document the class `TestEnableThinking`
```python
    """Testcase: The test is to verify whether the functions of each parameter of the v1/completions interface are normal.

    [Test Category] Interface
    [Test Target] v1/completions
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestEnableThinking`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestEnableThinking`的设计意图。

### Lines 472-478: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        # Skip initialization, directly reuse global server
        cls.model = LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.additional_chat_kwargs = {}
        logging.basicConfig(level=logging.INFO)  # Initialize logging
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 480-483: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        # Do not terminate server
        pass
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。

### Lines 485-493: Run test: model parameters model
```python
    def test_model_parameters_model(self):
        response = requests.post(
            f"{self.base_url}/v1/completions",
            json={"model": self.model, "prompt": "who are you?"},
        )
        logging.info(f"response.json:{response.json()}")
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        data = response.json()
        self.assertEqual(data["model"], self.model)
```
**EN:** This test method exercises model parameters model and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 model parameters model 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 495-529: Run test: model parameters prompt
```python
    def test_model_parameters_prompt(self):
        # str format
        response = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": "who are you?"},
        )
        logging.info(f"response.json:{response.json()}")
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")

        # list[int] format
        list_int = [1, 2, 3, 4]
        response1 = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": list_int},
        )
        logging.info(f"response1.json:{response1.json()}")
        self.assertEqual(response1.status_code, 200, f"Failed with: {response1.text}")

        # list[str] format
        list_str = ["who is you", "hello world", "ABChello"]
        response2 = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": list_str},
        )
        logging.info(f"response2.json:{response2.json()}")
        self.assertEqual(response2.status_code, 200, f"Failed with: {response2.text}")

        # list[list[int]] format
        list_list_int = [[14990], [1350, 445, 14990, 1879, 899], [14623, 525, 498, 30]]
        response3 = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": list_list_int},
        )
        logging.info(f"response3.json:{response3.json()}")
        self.assertEqual(response3.status_code, 200, f"Failed with: {response3.text}")
```
**EN:** This test method exercises model parameters prompt and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 model parameters prompt 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 531-539: Run test: model parameters max tokens
```python
    def test_model_parameters_max_tokens(self):
        response = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": "who are you?", "max_tokens": 1},
        )
        logging.info(f"response.json:{response.json()}")
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        logging.info(f"finish_reason:{response.json()['choices'][0]['finish_reason']}")
        self.assertEqual(response.json()["choices"][0]["finish_reason"], "length")
```
**EN:** This test method exercises model parameters max tokens and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 model parameters max tokens 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 541-559: Run test: model parameters stream
```python
    def test_model_parameters_stream(self):
        response = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": "who are you?", "stream": True},
        )
        logging.info(f"response.text:{response.text}")
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")

        has_text = False
        logging.info("\n=== Stream With Reasoning ===")
        for line in response.iter_lines():
            if line:
                line = line.decode("utf-8")
                if line.startswith("data:") and not line.startswith("data: [DONE]"):
                    data = json.loads(line[6:])
                    if "choices" in data and len(data["choices"]) > 0:
                        if "text" in data["choices"][0]:
                            has_text = True
        self.assertTrue(has_text, "Text content not included in stream response")
```
**EN:** This test method exercises model parameters stream and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 model parameters stream 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 561-596: Run test: model parameters temperature
```python
    def test_model_parameters_temperature(self):
        response = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": "who are you?", "temperature": 0},
        )
        logging.info(f"response.json:{response.json()}")
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")

        response1 = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": "who are you?", "temperature": 0},
        )
        logging.info(f"response1.json:{response1.json()}")
        self.assertEqual(response1.status_code, 200, f"Failed with: {response1.text}")
        self.assertEqual(
            response.json()["choices"][0]["text"],
            response1.json()["choices"][0]["text"],
        )

        response2 = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": "who are you?", "temperature": 2},
        )
        logging.info(f"response2.json:{response2.json()}")
        self.assertEqual(response2.status_code, 200, f"Failed with: {response2.text}")

        response3 = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": "who are you?", "temperature": 2},
        )
        logging.info(f"response3.json:{response3.json()}")
        self.assertEqual(response3.status_code, 200, f"Failed with: {response3.text}")
        self.assertNotEqual(
            response2.json()["choices"][0]["text"],
            response3.json()["choices"][0]["text"],
        )
```
**EN:** This test method exercises model parameters temperature and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 model parameters temperature 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 598-605: Run test: model parameters hidden states
```python
    def test_model_parameters_hidden_states(self):
        response = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": "who are you?", "return_hidden_states": True},
        )
        logging.info(f"response.json:{response.json()}")
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        self.assertIn("hidden_states", response.json()["choices"][0])
```
**EN:** This test method exercises model parameters hidden states and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 model parameters hidden states 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 607-626: Run test: model parameters top k
```python
    def test_model_parameters_top_k(self):
        response = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": "who are you?", "top_k": 20},
        )
        logging.info(f"response.json:{response.json()}")
        logging.info(f"response.text:{response.text}")
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")

        response1 = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": "who are you?", "top_k": 20},
        )
        logging.info(f"response1.json:{response1.json()}")
        logging.info(f"response1.text:{response1.text}")
        self.assertEqual(response1.status_code, 200, f"Failed with: {response1.text}")
        self.assertNotEqual(
            response.json()["choices"][0]["text"],
            response1.json()["choices"][0]["text"],
        )
```
**EN:** This test method exercises model parameters top k and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 model parameters top k 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 628-640: Run test: model parameters stop token ids
```python
    def test_model_parameters_stop_token_ids(self):
        list_ids = [13]
        response = requests.post(
            f"{self.base_url}/v1/completions",
            json={
                "prompt": "who are you?",
                "stop_token_ids": list_ids,
                "max_tokens": 1024,
            },
        )
        logging.info(f"response.json:{response.json()}")
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        self.assertEqual(response.json()["choices"][0]["matched_stop"], 13)
```
**EN:** This test method exercises model parameters stop token ids and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 model parameters stop token ids 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 642-649: Run test: model parameters rid
```python
    def test_model_parameters_rid(self):
        response = requests.post(
            f"{self.base_url}/v1/completions",
            json={"prompt": "who are you?", "rid": "10086"},
        )
        logging.info(f"response.json:{response.json()}")
        self.assertEqual(response.status_code, 200, f"Failed with: {response.text}")
        self.assertEqual(response.json()["id"], "10086")
```
**EN:** This test method exercises model parameters rid and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 model parameters rid 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 652-652: Define class TestStartProfile
```python
class TestStartProfile(CustomTestCase):
```
**EN:** This declaration introduces the `TestStartProfile` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestStartProfile` 测试类，并说明它通过继承承担的职责。

### Lines 653-657: Document the class `TestStartProfile`
```python
    """Testcase: Verify the correctness of /start_profile API with different parameter combinations (start_step/num_steps) on Ascend NPU backend.

    [Test Category] Interface
    [Test Target] /start_profile
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestStartProfile`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestStartProfile`的设计意图。

### Lines 659-665: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        # Skip initialization, reuse global server + configure profiler directory
        envs.SGLANG_TORCH_PROFILER_DIR.set(OUTPUT_DIR)
        cls.model = LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.additional_chat_kwargs = {}
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 667-673: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        # Terminate server in last class
        global GLOBAL_SERVER_PROCESS
        if GLOBAL_SERVER_PROCESS:
            kill_process_tree(GLOBAL_SERVER_PROCESS.pid)
            GLOBAL_SERVER_PROCESS = None
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 675-676: Prepare per-test state
```python
    def setUp(self):
        self._clear_profile_dir()
```
**EN:** This setup method initializes the state required before each individual test executes.
**CN:** 该初始化方法会在每个独立测试执行前准备所需状态。

### Lines 678-681: Run test: start profile 1
```python
    def test_start_profile_1(self):
        self._start_profile(start_step="15", num_steps=5)
        self._post_request()
        self._check_non_empty_profile_dir()
```
**EN:** This test method exercises start profile 1 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 start profile 1 场景，并验证观测到的行为是否符合预期契约。

### Lines 683-689: Run test: start profile 2
```python
    def test_start_profile_2(self):
        self._clear_profile_dir()
        self._check_empty_profile_dir()
        self._start_profile()
        self._post_request()
        requests.post(f"{self.base_url}/stop_profile")
        self._check_non_empty_profile_dir()
```
**EN:** This test method exercises start profile 2 and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 start profile 2 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 691-694: Run test: start profile 3
```python
    def test_start_profile_3(self):
        self._start_profile(num_steps=5)
        self._post_request()
        self._check_non_empty_profile_dir()
```
**EN:** This test method exercises start profile 3 and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 start profile 3 场景，并验证观测到的行为是否符合预期契约。

### Lines 696-702: Define helper: start profile
```python
    def _start_profile(self, **kwargs):
        response = requests.post(
            f"{self.base_url}/start_profile",
            json=kwargs if kwargs else None,
        )
        self.assertEqual(response.status_code, 200)
        return response
```
**EN:** This helper function encapsulates reusable logic inside `TestStartProfile` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestStartProfile` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 704-715: Define helper: post request
```python
    def _post_request(self):
        response = requests.post(
            f"{self.base_url}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 32,
                },
            },
        )
        self.assertEqual(response.status_code, 200)
```
**EN:** This helper function encapsulates reusable logic inside `TestStartProfile` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestStartProfile` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 717-719: Define helper: clear profile dir
```python
    def _clear_profile_dir(self):
        if os.path.isdir(OUTPUT_DIR):
            shutil.rmtree(OUTPUT_DIR)
```
**EN:** This helper function encapsulates reusable logic inside `TestStartProfile` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestStartProfile` 内部调用，从而让场景结构更清晰。

### Lines 721-725: Define helper: check non empty profile dir
```python
    def _check_non_empty_profile_dir(self):
        self.assertTrue(os.path.isdir(OUTPUT_DIR), "Profiler directory does not exist")
        self.assertNotEqual(
            len(os.listdir(OUTPUT_DIR)), 0, "Profiler directory is empty"
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestStartProfile` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestStartProfile` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 727-731: Define helper: check empty profile dir
```python
    def _check_empty_profile_dir(self):
        if os.path.isdir(OUTPUT_DIR):
            self.assertEqual(
                len(os.listdir(OUTPUT_DIR)), 0, "Profiler directory is not empty"
            )
```
**EN:** This helper function encapsulates reusable logic inside `TestStartProfile` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestStartProfile` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 734-735: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `json`, `logging`, `os`, `requests`, `shutil`, `transformers`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.get`, `requests.post`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
