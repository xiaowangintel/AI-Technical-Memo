# test_input_embeddings.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/input_embedding/test_input_embeddings.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates input embeddings behavior in SGLang's input embedding area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 input embedding 领域中与 input embeddings 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: module imports and dependencies / 模块导入与依赖
```python
import json
import os
import tempfile
import unittest

import requests
from transformers import AutoModelForCausalLM, AutoTokenizer

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `os`, `tempfile`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `os`, `tempfile`, `unittest`。

### Lines 19-20: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=42, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=38, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 23-23: class TestInputEmbeds declaration / 类 TestInputEmbeds 声明
```python
class TestInputEmbeds(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 24-39: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.tokenizer = AutoTokenizer.from_pretrained(cls.model)
        cls.ref_model = AutoModelForCausalLM.from_pretrained(cls.model)
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--disable-radix", "--cuda-graph-max-bs", 4],
        )
        cls.texts = [
            "The capital of France is",
            "What is the best time of year to visit Japan for cherry blossoms?",
        ]
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 41-45: method generate input embeddings / 方法 generate input embeddings
```python
    def generate_input_embeddings(self, text):
        """Generate input embeddings for a given text."""
        input_ids = self.tokenizer(text, return_tensors="pt")["input_ids"]
        embeddings = self.ref_model.get_input_embeddings()(input_ids)
        return embeddings.squeeze().tolist()  # Convert tensor to a list for API use
```
**EN:** Generate input embeddings for a given text. This block implements `generate_input_embeddings` and captures one focused piece of the module's behavior.
**CN:** Generate input embeddings for a given text. 该代码块实现 `generate_input_embeddings`，承担模块行为中的一个聚焦逻辑片段。

### Lines 47-58: method send request / 方法 send request
```python
    def send_request(self, payload):
        """Send a POST request to the /generate endpoint and return the response."""
        response = requests.post(
            self.base_url + "/generate",
            json=payload,
            timeout=30,  # Set a reasonable timeout for the API request
        )
        if response.status_code == 200:
            return response.json()
        return {
            "error": f"Request failed with status {response.status_code}: {response.text}"
        }
```
**EN:** Send a POST request to the /generate endpoint and return the response. This block implements `send_request` and captures one focused piece of the module's behavior.
**CN:** Send a POST request to the /generate endpoint and return the response. 该代码块实现 `send_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 60-72: method send file request / 方法 send file request
```python
    def send_file_request(self, file_path):
        """Send a POST request to the /generate_from_file endpoint with a file."""
        with open(file_path, "rb") as f:
            response = requests.post(
                self.base_url + "/generate_from_file",
                files={"file": f},
                timeout=30,  # Set a reasonable timeout for the API request
            )
        if response.status_code == 200:
            return response.json()
        return {
            "error": f"Request failed with status {response.status_code}: {response.text}"
        }
```
**EN:** Send a POST request to the /generate_from_file endpoint with a file. This block implements `send_file_request` and captures one focused piece of the module's behavior.
**CN:** Send a POST request to the /generate_from_file endpoint with a file. 该代码块实现 `send_file_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 74-85: test case text based response / 测试用例 text based response
```python
    def test_text_based_response(self):
        """Test and print API responses using text-based input."""
        for text in self.texts:
            payload = {
                "model": self.model,
                "text": text,
                "sampling_params": {"temperature": 0, "max_new_tokens": 50},
            }
            response = self.send_request(payload)
            print(
                f"Text Input: {text}\nResponse: {json.dumps(response, indent=2)}\n{'-' * 80}"
            )
```
**EN:** Test and print API responses using text-based input. This test exercises `test_text_based_response` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test and print API responses using text-based input. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_text_based_response`。

### Lines 87-99: test case embedding based response / 测试用例 embedding based response
```python
    def test_embedding_based_response(self):
        """Test and print API responses using input embeddings."""
        for text in self.texts:
            embeddings = self.generate_input_embeddings(text)
            payload = {
                "model": self.model,
                "input_embeds": embeddings,
                "sampling_params": {"temperature": 0, "max_new_tokens": 50},
            }
            response = self.send_request(payload)
            print(
                f"Embeddings Input (for text '{text}'):\nResponse: {json.dumps(response, indent=2)}\n{'-' * 80}"
            )
```
**EN:** Test and print API responses using input embeddings. This test exercises `test_embedding_based_response` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test and print API responses using input embeddings. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_based_response`。

### Lines 101-126: test case compare text vs embedding / 测试用例 compare text vs embedding
```python
    def test_compare_text_vs_embedding(self):
        """Test and compare responses for text-based and embedding-based inputs."""
        for text in self.texts:
            # Text-based payload
            text_payload = {
                "model": self.model,
                "text": text,
                "sampling_params": {"temperature": 0, "max_new_tokens": 50},
            }
            # Embedding-based payload
            embeddings = self.generate_input_embeddings(text)
            embed_payload = {
                "model": self.model,
                "input_embeds": embeddings,
                "sampling_params": {"temperature": 0, "max_new_tokens": 50},
            }
            # Get responses
            text_response = self.send_request(text_payload)
            embed_response = self.send_request(embed_payload)
            # Print responses
            print(
                f"Text Input: {text}\nText-Based Response: {json.dumps(text_response, indent=2)}\n"
            )
            print(
                f"Embeddings Input (for text '{text}'):\nEmbedding-Based Response: {json.dumps(embed_response, indent=2)}\n{'-' * 80}"
            )
```
**EN:** Test and compare responses for text-based and embedding-based inputs. This test exercises `test_compare_text_vs_embedding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test and compare responses for text-based and embedding-based inputs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_compare_text_vs_embedding`。

### Lines 127-129: supporting source context / 辅助源码上下文
```python
            # This is flaky, so we skip this temporarily
            # self.assertEqual(text_response["text"], embed_response["text"])

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 130-147: test case generate from file / 测试用例 generate from file
```python
    def test_generate_from_file(self):
        """Test the /generate_from_file endpoint using tokenized embeddings."""
        for text in self.texts:
            embeddings = self.generate_input_embeddings(text)
            with tempfile.NamedTemporaryFile(
                mode="w", suffix=".json", delete=False
            ) as tmp_file:
                json.dump(embeddings, tmp_file)
                tmp_file_path = tmp_file.name

            try:
                response = self.send_file_request(tmp_file_path)
                print(
                    f"Text Input: {text}\nResponse from /generate_from_file: {json.dumps(response, indent=2)}\n{'-' * 80}"
                )
            finally:
                # Ensure the temporary file is deleted
                os.remove(tmp_file_path)
```
**EN:** Test the /generate_from_file endpoint using tokenized embeddings. This test exercises `test_generate_from_file` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test the /generate_from_file endpoint using tokenized embeddings. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_from_file`。

### Lines 149-151: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 154-155: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestInputEmbeds`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestInputEmbeds.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestInputEmbeds.generate_input_embeddings`: Generate input embeddings for a given text. / 该代码块实现 `generate_input_embeddings`，承担模块行为中的一个聚焦逻辑片段。
- `TestInputEmbeds.send_request`: Send a POST request to the /generate endpoint and return the response. / 该代码块实现 `send_request`，承担模块行为中的一个聚焦逻辑片段。
- `TestInputEmbeds.send_file_request`: Send a POST request to the /generate_from_file endpoint with a file. / 该代码块实现 `send_file_request`，承担模块行为中的一个聚焦逻辑片段。
- `TestInputEmbeds.test_text_based_response`: Test and print API responses using text-based input. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_text_based_response`。
- `TestInputEmbeds.test_embedding_based_response`: Test and print API responses using input embeddings. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding_based_response`。
- `TestInputEmbeds.test_compare_text_vs_embedding`: Test and compare responses for text-based and embedding-based inputs. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_compare_text_vs_embedding`。
- `TestInputEmbeds.test_generate_from_file`: Test the /generate_from_file endpoint using tokenized embeddings. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_generate_from_file`。
- `TestInputEmbeds.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `tempfile`, `unittest`
- **Third-party modules / 第三方模块**: `requests`, `transformers`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 155
