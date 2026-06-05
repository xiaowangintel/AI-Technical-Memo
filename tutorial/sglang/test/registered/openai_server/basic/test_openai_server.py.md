# test_openai_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/openai_server/basic/test_openai_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates openai server behavior in SGLang's openai server / basic area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 openai server / basic 领域中与 openai server 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""
python3 -m unittest openai_server.basic.test_openai_server.TestOpenAIServer.test_completion
python3 -m unittest openai_server.basic.test_openai_server.TestOpenAIServer.test_completion_stream
python3 -m unittest openai_server.basic.test_openai_server.TestOpenAIServer.test_chat_completion
python3 -m unittest openai_server.basic.test_openai_server.TestOpenAIServer.test_chat_completion_stream
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-29: module imports and dependencies / 模块导入与依赖
```python
import json
import random
import unittest
from concurrent.futures import ThreadPoolExecutor
from typing import Optional

import openai
import requests

from sglang.srt.sampling.custom_logit_processor import CustomLogitProcessor
from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.runners import TEST_RERANK_QUERY_DOCS
from sglang.test.test_utils import (
    DEFAULT_SMALL_CROSS_ENCODER_MODEL_NAME_FOR_TEST,
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `random`, `unittest`, `concurrent.futures`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `random`, `unittest`, `concurrent.futures`。

### Lines 31-32: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=182, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=200, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 35-35: class TestOpenAIServer declaration / 类 TestOpenAIServer 声明
```python
class TestOpenAIServer(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 36-48: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(DEFAULT_SMALL_MODEL_NAME_FOR_TEST)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 50-52: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 54-112: helper routine run completion / 辅助流程 run completion
```python
    def run_completion(
        self, echo, logprobs, use_list_input, parallel_sample_num, token_input
    ):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        prompt = "The capital of France is"
        if token_input:
            prompt_input = self.tokenizer.encode(prompt)
            num_prompt_tokens = len(prompt_input)
        else:
            prompt_input = prompt
            num_prompt_tokens = len(self.tokenizer.encode(prompt))

        if use_list_input:
            prompt_arg = [prompt_input, prompt_input]
            num_choices = len(prompt_arg)
            num_prompt_tokens *= 2
        else:
            prompt_arg = prompt_input
            num_choices = 1

        response = client.completions.create(
            model=self.model,
            prompt=prompt_arg,
            temperature=0,
            max_tokens=32,
            echo=echo,
            logprobs=logprobs,
            n=parallel_sample_num,
        )

        assert len(response.choices) == num_choices * parallel_sample_num

        if echo:
            text = response.choices[0].text
            assert text.startswith(prompt)

        if logprobs:
            assert response.choices[0].logprobs
            assert isinstance(
                response.choices[0].logprobs.tokens[0], str
            ), f"{response=}"
            assert isinstance(response.choices[0].logprobs.top_logprobs[1], dict)
            ret_num_top_logprobs = len(response.choices[0].logprobs.top_logprobs[1])

            # FIXME: Sometimes, some top_logprobs are missing in the return value. The reason is that some output id maps to the same output token and duplicate in the map
            # assert ret_num_top_logprobs == logprobs, f"{ret_num_top_logprobs} vs {logprobs}"
            assert ret_num_top_logprobs > 0

            # when echo=True and request.logprobs>0, logprob_start_len is 0, so the first token's logprob would be None.
            if not echo:
                assert response.choices[0].logprobs.token_logprobs[0]

        assert response.id
        assert response.created
        assert (
            response.usage.prompt_tokens == num_prompt_tokens
        ), f"{response.usage.prompt_tokens} vs {num_prompt_tokens}"
        assert response.usage.completion_tokens > 0
        assert response.usage.total_tokens > 0
```
**EN:** This helper encapsulates `run_completion` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_completion`，以便周围测试复用准备、执行或校验逻辑。

### Lines 114-190: helper routine run completion stream / 辅助流程 run completion stream
```python
    def run_completion_stream(
        self, echo, logprobs, use_list_input, parallel_sample_num, token_input
    ):
        print(
            f"run_completion_stream: {echo=}, {logprobs=}, {use_list_input=}, {parallel_sample_num=}, {token_input=}"
        )
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        prompt = "The capital of France is"
        if token_input:
            prompt_input = self.tokenizer.encode(prompt)
            num_prompt_tokens = len(prompt_input)
        else:
            prompt_input = prompt
            num_prompt_tokens = len(self.tokenizer.encode(prompt))

        if use_list_input:
            prompt_arg = [prompt_input, prompt_input]
            num_choices = len(prompt_arg)
            num_prompt_tokens *= 2
        else:
            prompt_arg = prompt_input
            num_choices = 1

        generator = client.completions.create(
            model=self.model,
            prompt=prompt_arg,
            temperature=0,
            max_tokens=32,
            echo=echo,
            logprobs=logprobs,
            stream=True,
            stream_options={"include_usage": True},
            n=parallel_sample_num,
        )

        is_firsts = {}
        for response in generator:
            print(f"{response=}")
            usage = response.usage
            if usage is not None:
                assert usage.prompt_tokens > 0, f"usage.prompt_tokens was zero"
                assert usage.completion_tokens > 0, f"usage.completion_tokens was zero"
                assert usage.total_tokens > 0, f"usage.total_tokens was zero"
                continue

            index = response.choices[0].index
            is_first = is_firsts.get(index, True)

            if logprobs:
                assert response.choices[0].logprobs, f"no logprobs in response"
                assert isinstance(
                    response.choices[0].logprobs.tokens[0], str
                ), f"{response.choices[0].logprobs.tokens[0]} is not a string"
                if not (is_first and echo):
                    assert isinstance(
                        response.choices[0].logprobs.top_logprobs[0], dict
                    ), f"top_logprobs was not a dictionary"
                    ret_num_top_logprobs = len(
                        response.choices[0].logprobs.top_logprobs[0]
                    )
                    # FIXME: Sometimes, some top_logprobs are missing in the return value. The reason is that some output id maps to the same output token and duplicate in the map
                    # assert ret_num_top_logprobs == logprobs, f"{ret_num_top_logprobs} vs {logprobs}"
                    assert ret_num_top_logprobs > 0, f"ret_num_top_logprobs was 0"

            if is_first:
                if echo:
                    assert response.choices[0].text.startswith(
                        prompt
                    ), f"{response.choices[0].text} and all args {echo} {logprobs} {token_input} {is_first}"
                is_firsts[index] = False
            assert response.id, f"no id in response"
            assert response.created, f"no created in response"

        for index in [i for i in range(parallel_sample_num * num_choices)]:
            assert not is_firsts.get(
                index, True
            ), f"index {index} is not found in the response"
```
**EN:** This helper encapsulates `run_completion_stream` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_completion_stream`，以便周围测试复用准备、执行或校验逻辑。

### Lines 192-228: helper routine run chat completion / 辅助流程 run chat completion
```python
    def run_chat_completion(self, logprobs, parallel_sample_num):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        response = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {
                    "role": "user",
                    "content": "What is the capital of France? Answer in a few words.",
                },
            ],
            temperature=0,
            logprobs=logprobs is not None and logprobs > 0,
            top_logprobs=logprobs,
            n=parallel_sample_num,
        )

        if logprobs:
            assert isinstance(
                response.choices[0].logprobs.content[0].top_logprobs[0].token, str
            )

            ret_num_top_logprobs = len(
                response.choices[0].logprobs.content[0].top_logprobs
            )
            assert (
                ret_num_top_logprobs == logprobs
            ), f"{ret_num_top_logprobs} vs {logprobs}"

        assert len(response.choices) == parallel_sample_num
        assert response.choices[0].message.role == "assistant"
        assert isinstance(response.choices[0].message.content, str)
        assert response.id
        assert response.created
        assert response.usage.prompt_tokens > 0
        assert response.usage.completion_tokens > 0
        assert response.usage.total_tokens > 0
```
**EN:** This helper encapsulates `run_chat_completion` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_chat_completion`，以便周围测试复用准备、执行或校验逻辑。

### Lines 230-308: helper routine run chat completion stream / 辅助流程 run chat completion stream
```python
    def run_chat_completion_stream(self, logprobs, parallel_sample_num=1):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        generator = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {"role": "user", "content": "What is the capital of France?"},
            ],
            temperature=0,
            logprobs=logprobs is not None and logprobs > 0,
            top_logprobs=logprobs,
            stream=True,
            stream_options={"include_usage": True},
            n=parallel_sample_num,
        )

        is_firsts = {}
        is_finished = {}
        finish_reason_counts = {}
        for response in generator:
            usage = response.usage
            if usage is not None:
                assert usage.prompt_tokens > 0, f"usage.prompt_tokens was zero"
                assert usage.completion_tokens > 0, f"usage.completion_tokens was zero"
                assert usage.total_tokens > 0, f"usage.total_tokens was zero"
                continue

            index = response.choices[0].index
            finish_reason = response.choices[0].finish_reason
            if finish_reason is not None:
                is_finished[index] = True
                finish_reason_counts[index] = finish_reason_counts.get(index, 0) + 1

            data = response.choices[0].delta

            if is_firsts.get(index, True):
                assert (
                    data.role == "assistant"
                ), f"data.role was not 'assistant' for first chunk"
                is_firsts[index] = False
                continue

            if logprobs and not is_finished.get(index, False):
                assert response.choices[0].logprobs, f"logprobs was not returned"
                assert isinstance(
                    response.choices[0].logprobs.content[0].top_logprobs[0].token, str
                ), f"top_logprobs token was not a string"
                assert isinstance(
                    response.choices[0].logprobs.content[0].top_logprobs, list
                ), f"top_logprobs was not a list"
                ret_num_top_logprobs = len(
                    response.choices[0].logprobs.content[0].top_logprobs
                )
                assert (
                    ret_num_top_logprobs == logprobs
                ), f"{ret_num_top_logprobs} vs {logprobs}"

            assert (
                isinstance(data.content, str)
                or isinstance(data.reasoning_content, str)
                or (isinstance(data.tool_calls, list) and len(data.tool_calls) > 0)
                or response.choices[0].finish_reason
            )
            assert response.id
            assert response.created

        for index in [i for i in range(parallel_sample_num)]:
            assert not is_firsts.get(
                index, True
            ), f"index {index} is not found in the response"

        # Verify that each choice gets exactly one finish_reason chunk
        for index in range(parallel_sample_num):
            assert (
                index in finish_reason_counts
            ), f"No finish_reason found for index {index}"
            assert (
                finish_reason_counts[index] == 1
            ), f"Expected 1 finish_reason chunk for index {index}, got {finish_reason_counts[index]}"
```
**EN:** This helper encapsulates `run_chat_completion_stream` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_chat_completion_stream`，以便周围测试复用准备、执行或校验逻辑。

### Lines 310-322: test case completion / 测试用例 completion
```python
    def test_completion(self):
        for echo in [False, True]:
            for logprobs in [None, 5]:
                for use_list_input in [True, False]:
                    for parallel_sample_num in [1, 2]:
                        for token_input in [False, True]:
                            self.run_completion(
                                echo,
                                logprobs,
                                use_list_input,
                                parallel_sample_num,
                                token_input,
                            )
```
**EN:** This test exercises `test_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion`。

### Lines 324-337: test case completion stream / 测试用例 completion stream
```python
    def test_completion_stream(self):
        # parallel sampling and list input are not supported in streaming mode
        for echo in [False, True]:
            for logprobs in [None, 5]:
                for use_list_input in [True, False]:
                    for parallel_sample_num in [1, 2]:
                        for token_input in [False, True]:
                            self.run_completion_stream(
                                echo,
                                logprobs,
                                use_list_input,
                                parallel_sample_num,
                                token_input,
                            )
```
**EN:** This test exercises `test_completion_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion_stream`。

### Lines 339-342: test case chat completion / 测试用例 chat completion
```python
    def test_chat_completion(self):
        for logprobs in [None, 5]:
            for parallel_sample_num in [1, 2]:
                self.run_chat_completion(logprobs, parallel_sample_num)
```
**EN:** This test exercises `test_chat_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion`。

### Lines 344-347: test case chat completion stream / 测试用例 chat completion stream
```python
    def test_chat_completion_stream(self):
        for logprobs in [None, 5]:
            for parallel_sample_num in [1, 2]:
                self.run_chat_completion_stream(logprobs, parallel_sample_num)
```
**EN:** This test exercises `test_chat_completion_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_stream`。

### Lines 349-377: test case regex / 测试用例 regex
```python
    def test_regex(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        regex = (
            r"""\{\n"""
            + r"""   "name": "[\w]+",\n"""
            + r"""   "population": [\d]+\n"""
            + r"""\}"""
        )

        response = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {"role": "user", "content": "Introduce the capital of France."},
            ],
            temperature=0,
            max_tokens=128,
            extra_body={"regex": regex},
        )
        text = response.choices[0].message.content

        try:
            js_obj = json.loads(text)
        except (TypeError, json.decoder.JSONDecodeError):
            print("JSONDecodeError", text)
            raise
        assert isinstance(js_obj["name"], str)
        assert isinstance(js_obj["population"], int)
```
**EN:** This test exercises `test_regex` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_regex`。

### Lines 379-393: test case penalty / 测试用例 penalty
```python
    def test_penalty(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        response = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {"role": "user", "content": "Introduce the capital of France."},
            ],
            temperature=0,
            max_tokens=32,
            frequency_penalty=1.0,
        )
        text = response.choices[0].message.content
        assert isinstance(text, str)
```
**EN:** This test exercises `test_penalty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_penalty`。

### Lines 395-425: test case response prefill / 测试用例 response prefill
```python
    def test_response_prefill(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        response = client.chat.completions.create(
            model="meta-llama/Llama-3.1-8B-Instruct",
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {
                    "role": "user",
                    "content": """
Extract the name, size, price, and color from this product description as a JSON object:

<description>
The SmartHome Mini is a compact smart home assistant available in black or white for only $49.99. At just 5 inches wide, it lets you control lights, thermostats, and other connected devices via voice or app—no matter where you place it in your home. This affordable little hub brings convenient hands-free control to your smart devices.
</description>
""",
                },
                {
                    "role": "assistant",
                    "content": "{\n",
                },
            ],
            temperature=0,
            extra_body={"continue_final_message": True},
        )

        assert (
            response.choices[0]
            .message.content.strip()
            .startswith('"name": "SmartHome Mini",')
        )
```
**EN:** This test exercises `test_response_prefill` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_prefill`。

### Lines 427-431: test case model list / 测试用例 model list
```python
    def test_model_list(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        models = list(client.models.list())
        assert len(models) == 1
        assert isinstance(getattr(models[0], "max_model_len", None), int)
```
**EN:** This test exercises `test_model_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_model_list`。

### Lines 433-443: test case retrieve model / 测试用例 retrieve model
```python
    def test_retrieve_model(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        # Test retrieving an existing model
        retrieved_model = client.models.retrieve(self.model)
        self.assertEqual(retrieved_model.id, self.model)
        self.assertEqual(retrieved_model.root, self.model)

        # Test retrieving a non-existent model
        with self.assertRaises(openai.NotFoundError):
            client.models.retrieve("non-existent-model")
```
**EN:** This test exercises `test_retrieve_model` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_retrieve_model`。

### Lines 446-446: class TestOpenAIServerv1Responses declaration / 类 TestOpenAIServerv1Responses 声明
```python
class TestOpenAIServerv1Responses(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 447-459: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(DEFAULT_SMALL_MODEL_NAME_FOR_TEST)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 461-463: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 465-499: helper routine run response / 辅助流程 run response
```python
    def run_response(
        self,
        input_text: str = "The capital of France is",
        *,
        instructions: str | None = None,
        temperature: float | None = 0.0,
        top_p: float | None = 1.0,
        max_output_tokens: int | None = 32,
        store: bool | None = True,
        parallel_tool_calls: bool | None = True,
        tool_choice: str | None = "auto",
        previous_response_id: str | None = None,
        truncation: str | None = "disabled",
        user: str | None = None,
        metadata: dict | None = None,
    ):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        payload = {
            "model": self.model,
            "input": input_text,
            "temperature": temperature,
            "top_p": top_p,
            "max_output_tokens": max_output_tokens,
            "store": store,
            "parallel_tool_calls": parallel_tool_calls,
            "tool_choice": tool_choice,
            "previous_response_id": previous_response_id,
            "truncation": truncation,
            "user": user,
            "instructions": instructions,
        }
        if metadata is not None:
            payload["metadata"] = metadata
        payload = {k: v for k, v in payload.items() if v is not None}
        return client.responses.create(**payload)
```
**EN:** This helper encapsulates `run_response` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_response`，以便周围测试复用准备、执行或校验逻辑。

### Lines 501-580: helper routine run response stream (part 1/2) / 辅助流程 run response stream（第 1/2 部分）
```python
    def run_response_stream(
        self,
        input_text: str = "The capital of France is",
        *,
        instructions: str | None = None,
        temperature: float | None = 0.0,
        top_p: float | None = 1.0,
        max_output_tokens: int | None = 32,
        store: bool | None = True,
        parallel_tool_calls: bool | None = True,
        tool_choice: str | None = "auto",
        previous_response_id: str | None = None,
        truncation: str | None = "disabled",
        user: str | None = None,
        metadata: dict | None = None,
    ):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        payload = {
            "model": self.model,
            "input": input_text,
            "temperature": temperature,
            "top_p": top_p,
            "max_output_tokens": max_output_tokens,
            "store": store,
            "parallel_tool_calls": parallel_tool_calls,
            "tool_choice": tool_choice,
            "previous_response_id": previous_response_id,
            "truncation": truncation,
            "user": user,
            "instructions": instructions,
            "stream": True,
            "stream_options": {"include_usage": True},
        }
        if metadata is not None:
            payload["metadata"] = metadata
        payload = {k: v for k, v in payload.items() if v is not None}

        aggregated_text = ""
        saw_created = False
        saw_in_progress = False
        saw_completed = False
        final_usage_ok = False

        stream_ctx = getattr(client.responses, "stream", None)
        if callable(stream_ctx):
            stream_payload = dict(payload)
            stream_payload.pop("stream", None)
            stream_payload.pop("stream_options", None)
            with client.responses.stream(**stream_payload) as stream:
                for event in stream:
                    et = getattr(event, "type", None)
                    if et == "response.created":
                        saw_created = True
                    elif et == "response.in_progress":
                        saw_in_progress = True
                    elif et == "response.output_text.delta":
                        # event.delta expected to be a string
                        delta = getattr(event, "delta", "")
                        if isinstance(delta, str):
                            aggregated_text += delta
                    elif et == "response.completed":
                        saw_completed = True
                        # Validate streaming-completed usage mapping
                        resp = getattr(event, "response", None)
                        try:
                            # resp may be dict-like already
                            usage = (
                                resp.get("usage")
                                if isinstance(resp, dict)
                                else getattr(resp, "usage", None)
                            )
                            if isinstance(usage, dict):
                                final_usage_ok = all(
                                    k in usage
                                    for k in (
                                        "input_tokens",
                                        "output_tokens",
                                        "total_tokens",
                                    )
                                )
```
**EN:** This helper encapsulates `run_response_stream` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 1 of the same logical block.
**CN:** 该辅助函数封装了 `run_response_stream`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 1 部分。

### Lines 581-605: helper routine run response stream (part 2/2) / 辅助流程 run response stream（第 2/2 部分）
```python
                        except Exception:
                            pass
                _ = stream.get_final_response()
        else:
            generator = client.responses.create(**payload)
            for event in generator:
                et = getattr(event, "type", None)
                if et == "response.created":
                    saw_created = True
                elif et == "response.in_progress":
                    saw_in_progress = True
                elif et == "response.output_text.delta":
                    delta = getattr(event, "delta", "")
                    if isinstance(delta, str):
                        aggregated_text += delta
                elif et == "response.completed":
                    saw_completed = True

        return (
            aggregated_text,
            saw_created,
            saw_in_progress,
            saw_completed,
            final_usage_ok,
        )
```
**EN:** This helper encapsulates `run_response_stream` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 2 of the same logical block.
**CN:** 该辅助函数封装了 `run_response_stream`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 2 部分。

### Lines 607-623: helper routine run chat completion stream / 辅助流程 run chat completion stream
```python
    def run_chat_completion_stream(self, logprobs=None, parallel_sample_num=1):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        generator = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {"role": "user", "content": "What is the capital of France?"},
            ],
            temperature=0,
            logprobs=logprobs is not None and logprobs > 0,
            top_logprobs=logprobs,
            stream=True,
            stream_options={"include_usage": True},
            n=parallel_sample_num,
        )
        for _ in generator:
            pass
```
**EN:** This helper encapsulates `run_chat_completion_stream` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_chat_completion_stream`，以便周围测试复用准备、执行或校验逻辑。

### Lines 624-625: supporting source context / 辅助源码上下文
```python

    # ---- tests ----
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 626-652: test case response / 测试用例 response
```python
    def test_response(self):
        resp = self.run_response(temperature=0, max_output_tokens=32)
        assert resp.id
        assert resp.object == "response"
        assert resp.created_at
        assert isinstance(resp.model, str)
        assert isinstance(resp.output, list)
        assert resp.status in (
            "completed",
            "in_progress",
            "queued",
            "failed",
            "cancelled",
        )
        if resp.status == "completed":
            assert resp.usage is not None
            assert resp.usage.prompt_tokens >= 0
            assert resp.usage.completion_tokens >= 0
            assert resp.usage.total_tokens >= 0
        if hasattr(resp, "error"):
            assert resp.error is None
        if hasattr(resp, "incomplete_details"):
            assert resp.incomplete_details is None
        if getattr(resp, "text", None):
            fmt = resp.text.get("format") if isinstance(resp.text, dict) else None
            if fmt:
                assert fmt.get("type") == "text"
```
**EN:** This test exercises `test_response` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response`。

### Lines 654-663: test case response stream / 测试用例 response stream
```python
    def test_response_stream(self):
        aggregated_text, saw_created, saw_in_progress, saw_completed, final_usage_ok = (
            self.run_response_stream(temperature=0, max_output_tokens=32)
        )
        assert saw_created, "Did not observe response.created"
        assert saw_in_progress, "Did not observe response.in_progress"
        assert saw_completed, "Did not observe response.completed"
        assert isinstance(aggregated_text, str)
        assert len(aggregated_text) >= 0
        assert final_usage_ok or True  # final_usage's stats are not done for now
```
**EN:** This test exercises `test_response_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_stream`。

### Lines 665-670: test case response completion / 测试用例 response completion
```python
    def test_response_completion(self):
        resp = self.run_response(temperature=0, max_output_tokens=16)
        assert resp.status in ("completed", "in_progress", "queued")
        if resp.status == "completed":
            assert resp.usage is not None
            assert resp.usage.total_tokens >= 0
```
**EN:** This test exercises `test_response_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_completion`。

### Lines 672-679: test case response completion stream / 测试用例 response completion stream
```python
    def test_response_completion_stream(self):
        _, saw_created, saw_in_progress, saw_completed, final_usage_ok = (
            self.run_response_stream(temperature=0, max_output_tokens=16)
        )
        assert saw_created
        assert saw_in_progress
        assert saw_completed
        assert final_usage_ok or True  # final_usage's stats are not done for now
```
**EN:** This test exercises `test_response_completion_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_completion_stream`。

### Lines 681-709: test case regex / 测试用例 regex
```python
    def test_regex(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        regex = (
            r"""\{\n"""
            + r"""   "name": "[\w]+",\n"""
            + r"""   "population": [\d]+\n"""
            + r"""\}"""
        )

        response = client.chat.completions.create(
            model=self.model,
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {"role": "user", "content": "Introduce the capital of France."},
            ],
            temperature=0,
            max_tokens=128,
            extra_body={"regex": regex},
        )
        text = response.choices[0].message.content

        try:
            js_obj = json.loads(text)
        except (TypeError, json.decoder.JSONDecodeError):
            print("JSONDecodeError", text)
            raise
        assert isinstance(js_obj["name"], str)
        assert isinstance(js_obj["population"], int)
```
**EN:** This test exercises `test_regex` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_regex`。

### Lines 711-728: test case error / 测试用例 error
```python
    def test_error(self):
        url = f"{self.base_url}/responses"
        headers = {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json",
        }
        payload = {
            "model": self.model,
            "input": "Hi",
            "previous_response_id": "bad",  # invalid prefix
        }
        r = requests.post(url, headers=headers, json=payload)
        self.assertEqual(r.status_code, 400)
        body = r.json()
        self.assertIn("error", body)
        self.assertIn("message", body["error"])
        self.assertIn("type", body["error"])
        self.assertIn("code", body["error"])
```
**EN:** This test exercises `test_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_error`。

### Lines 730-751: test case penalty / 测试用例 penalty
```python
    def test_penalty(self):
        url = f"{self.base_url}/responses"
        headers = {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json",
        }
        payload = {
            "model": self.model,
            "input": "Introduce the capital of France.",
            "temperature": 0,
            "max_output_tokens": 32,
            "frequency_penalty": 1.0,
        }
        r = requests.post(url, headers=headers, json=payload)
        self.assertEqual(r.status_code, 200)
        body = r.json()
        self.assertEqual(body.get("object"), "response")
        self.assertIn("output", body)
        self.assertIn("status", body)
        if "usage" in body:
            self.assertIn("prompt_tokens", body["usage"])
            self.assertIn("total_tokens", body["usage"])
```
**EN:** This test exercises `test_penalty` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_penalty`。

### Lines 753-783: test case response prefill / 测试用例 response prefill
```python
    def test_response_prefill(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        response = client.chat.completions.create(
            model="meta-llama/Llama-3.1-8B-Instruct",
            messages=[
                {"role": "system", "content": "You are a helpful AI assistant"},
                {
                    "role": "user",
                    "content": """
Extract the name, size, price, and color from this product description as a JSON object:

<description>
The SmartHome Mini is a compact smart home assistant available in black or white for only $49.99. At just 5 inches wide, it lets you control lights, thermostats, and other connected devices via voice or app—no matter where you place it in your home. This affordable little hub brings convenient hands-free control to your smart devices.
</description>
""",
                },
                {
                    "role": "assistant",
                    "content": "{\n",
                },
            ],
            temperature=0,
            extra_body={"continue_final_message": True},
        )

        assert (
            response.choices[0]
            .message.content.strip()
            .startswith('"name": "SmartHome Mini",')
        )
```
**EN:** This test exercises `test_response_prefill` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_prefill`。

### Lines 785-789: test case model list / 测试用例 model list
```python
    def test_model_list(self):
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        models = list(client.models.list())
        assert len(models) == 1
        assert isinstance(getattr(models[0], "max_model_len", None), int)
```
**EN:** This test exercises `test_model_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_model_list`。

### Lines 792-792: class TestOpenAIV1Rerank declaration / 类 TestOpenAIV1Rerank 声明
```python
class TestOpenAIV1Rerank(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 793-817: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_CROSS_ENCODER_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.score_tolerance = 1e-2

        # Configure embedding-specific args
        other_args = [
            "--is-embedding",
            "--enable-metrics",
            "--disable-radix-cache",
            "--chunked-prefill-size",
            "-1",
            "--attention-backend",
            "torch_native",
        ]
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=other_args,
        )
        cls.base_url += "/v1/rerank"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 819-821: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 823-833: helper routine run rerank / 辅助流程 run rerank
```python
    def run_rerank(self, query, docs):
        response = requests.post(
            self.base_url,
            headers={
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json",
            },
            json={"query": query, "documents": docs},
        )

        return response.json()
```
**EN:** This helper encapsulates `run_rerank` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_rerank`，以便周围测试复用准备、执行或校验逻辑。

### Lines 835-845: test case rerank single / 测试用例 rerank single
```python
    def test_rerank_single(self):
        """Test single rerank request"""
        query = TEST_RERANK_QUERY_DOCS[0]["query"]
        docs = TEST_RERANK_QUERY_DOCS[0]["documents"]

        response = self.run_rerank(query, docs)

        self.assertEqual(len(response), 1)
        self.assertTrue(isinstance(response[0]["score"], float))
        self.assertTrue(isinstance(response[0]["document"], str))
        self.assertTrue(isinstance(response[0]["index"], int))
```
**EN:** Test single rerank request This test exercises `test_rerank_single` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test single rerank request 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rerank_single`。

### Lines 847-860: test case rerank batch / 测试用例 rerank batch
```python
    def test_rerank_batch(self):
        """Test batch rerank request"""
        query = TEST_RERANK_QUERY_DOCS[1]["query"]
        docs = TEST_RERANK_QUERY_DOCS[1]["documents"]

        response = self.run_rerank(query, docs)

        self.assertEqual(len(response), 2)
        self.assertTrue(isinstance(response[0]["score"], float))
        self.assertTrue(isinstance(response[1]["score"], float))
        self.assertTrue(isinstance(response[0]["document"], str))
        self.assertTrue(isinstance(response[1]["document"], str))
        self.assertTrue(isinstance(response[0]["index"], int))
        self.assertTrue(isinstance(response[1]["index"], int))
```
**EN:** Test batch rerank request This test exercises `test_rerank_batch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test batch rerank request 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rerank_batch`。

### Lines 863-863: class TestOpenAIServerCustomLogitProcessor declaration / 类 TestOpenAIServerCustomLogitProcessor 声明
```python
class TestOpenAIServerCustomLogitProcessor(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 864-877: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls) -> None:
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=["--enable-custom-logit-processor"],
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(cls.model)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 879-881: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls) -> None:
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 883-937: helper routine run custom logit processor / 辅助流程 run custom logit processor
```python
    def run_custom_logit_processor(self, target_token_id: Optional[int] = None) -> None:
        """
        Test custom logit processor with custom params.

        If target_token_id is None, the custom logit processor won't be passed in.
        """

        class DeterministicLogitProcessor(CustomLogitProcessor):
            """A dummy logit processor that changes the logits to always sample the given token id."""

            CUSTOM_PARAM_KEY = "token_id"

            def __call__(self, logits, custom_param_list):
                assert logits.shape[0] == len(custom_param_list)

                for i, param_dict in enumerate(custom_param_list):
                    # Mask all other tokens
                    logits[i, :] = -float("inf")
                    # Assign highest probability to the specified token
                    logits[i, param_dict[self.CUSTOM_PARAM_KEY]] = 0.0

                return logits

        extra_body = {}

        if target_token_id is not None:
            extra_body["custom_logit_processor"] = (
                DeterministicLogitProcessor().to_str()
            )
            extra_body["custom_params"] = {
                "token_id": target_token_id,
            }

        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        max_tokens = 200

        response = client.chat.completions.create(
            model=self.model,
            messages=[
                {
                    "role": "user",
                    "content": "Question: Is Paris the Capital of France?",
                },
            ],
            temperature=0.0,
            max_tokens=max_tokens,
            extra_body=extra_body,
        )

        if target_token_id is not None:
            target_text = self.tokenizer.decode([target_token_id] * max_tokens)
            self.assertTrue(
                target_text == response.choices[0].message.content,
                f"{target_token_id=}\n{target_text=}\n{response.model_dump(mode='json')}",
            )
```
**EN:** Test custom logit processor with custom params. This helper encapsulates `run_custom_logit_processor` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Test custom logit processor with custom params. 该辅助函数封装了 `run_custom_logit_processor`，以便周围测试复用准备、执行或校验逻辑。

### Lines 939-941: test case custom logit processor / 测试用例 custom logit processor
```python
    def test_custom_logit_processor(self) -> None:
        """Test custom logit processor with a single request."""
        self.run_custom_logit_processor(target_token_id=5)
```
**EN:** Test custom logit processor with a single request. This test exercises `test_custom_logit_processor` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test custom logit processor with a single request. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_logit_processor`。

### Lines 943-948: test case custom logit processor batch mixed / 测试用例 custom logit processor batch mixed
```python
    def test_custom_logit_processor_batch_mixed(self) -> None:
        """Test a batch of requests mixed of requests with and without custom logit processor."""
        target_token_ids = list(range(32)) + [None] * 16
        random.shuffle(target_token_ids)
        with ThreadPoolExecutor(len(target_token_ids)) as executor:
            list(executor.map(self.run_custom_logit_processor, target_token_ids))
```
**EN:** Test a batch of requests mixed of requests with and without custom logit processor. This test exercises `test_custom_logit_processor_batch_mixed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test a batch of requests mixed of requests with and without custom logit processor. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_logit_processor_batch_mixed`。

### Lines 951-951: class TestOpenAIV1Score declaration / 类 TestOpenAIV1Score 声明
```python
class TestOpenAIV1Score(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 952-965: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
        )
        cls.base_url += "/v1/score"
        cls.tokenizer = get_tokenizer(DEFAULT_SMALL_MODEL_NAME_FOR_TEST)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 967-969: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 971-989: helper routine run score / 辅助流程 run score
```python
    def run_score(
        self, query, items, label_token_ids, apply_softmax=False, item_first=False
    ):
        response = requests.post(
            self.base_url,
            headers={
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json",
            },
            json={
                "model": self.model,
                "query": query,
                "items": items,
                "label_token_ids": label_token_ids,
                "apply_softmax": apply_softmax,
                "item_first": item_first,
            },
        )
        return response.json()
```
**EN:** This helper encapsulates `run_score` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_score`，以便周围测试复用准备、执行或校验逻辑。

### Lines 991-1048: test case score text input / 测试用例 score text input
```python
    def test_score_text_input(self):
        """Test scoring with text input"""
        query = "The capital of France is"
        items = ["Paris", "London", "Berlin"]

        # Get valid token IDs from the tokenizer
        label_token_ids = []
        for item in items:
            token_ids = self.tokenizer.encode(item, add_special_tokens=False)
            if not token_ids:
                self.fail(f"Failed to encode item: {item}")
            label_token_ids.append(token_ids[0])

        response = self.run_score(query, items, label_token_ids, apply_softmax=True)

        # Handle error responses
        if response.get("type") == "BadRequestError":
            self.fail(f"Score request failed with error: {response['message']}")

        # Verify response structure
        self.assertIn("scores", response, "Response should have a 'scores' field")
        self.assertIsInstance(response["scores"], list, "scores should be a list")
        self.assertEqual(
            len(response["scores"]),
            len(items),
            "Number of scores should match number of items",
        )

        # Each score should be a list of floats in the order of label_token_ids
        for i, score_list in enumerate(response["scores"]):
            self.assertIsInstance(score_list, list, f"Score {i} should be a list")
            self.assertEqual(
                len(score_list),
                len(label_token_ids),
                f"Score {i} length should match label_token_ids",
            )
            self.assertTrue(
                all(isinstance(v, float) for v in score_list),
                f"Score {i} values should be floats",
            )
            self.assertAlmostEqual(
                sum(score_list),
                1.0,
                places=6,
                msg=f"Score {i} probabilities should sum to 1",
            )

        # Verify usage
        self.assertIn("usage", response, "Response should have a 'usage' field")
        self.assertGreater(response["usage"]["prompt_tokens"], 0)
        self.assertEqual(
            response["usage"]["prompt_tokens"], response["usage"]["total_tokens"]
        )
        self.assertEqual(
            response["usage"]["completion_tokens"],
            0,
            "completion_tokens should be 0 for /v1/score",
        )
```
**EN:** Test scoring with text input This test exercises `test_score_text_input` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test scoring with text input 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_text_input`。

### Lines 1050-1110: test case score token input / 测试用例 score token input
```python
    def test_score_token_input(self):
        """Test scoring with token IDs input"""
        query = "The capital of France is"
        items = ["Paris", "London", "Berlin"]

        # Get valid token IDs
        query_ids = self.tokenizer.encode(query, add_special_tokens=False)
        item_ids = [
            self.tokenizer.encode(item, add_special_tokens=False) for item in items
        ]
        label_token_ids = [
            ids[0] for ids in item_ids if ids
        ]  # Get first token ID of each item

        response = self.run_score(
            query_ids, item_ids, label_token_ids, apply_softmax=True
        )

        # Handle error responses
        if response.get("type") == "BadRequestError":
            self.fail(f"Score request failed with error: {response['message']}")

        # Verify response structure
        self.assertIn("scores", response, "Response should have a 'scores' field")
        self.assertIsInstance(response["scores"], list, "scores should be a list")
        self.assertEqual(
            len(response["scores"]),
            len(items),
            "Number of scores should match number of items",
        )

        # Each score should be a list of floats in the order of label_token_ids
        for i, score_list in enumerate(response["scores"]):
            self.assertIsInstance(score_list, list, f"Score {i} should be a list")
            self.assertEqual(
                len(score_list),
                len(label_token_ids),
                f"Score {i} length should match label_token_ids",
            )
            self.assertTrue(
                all(isinstance(v, float) for v in score_list),
                f"Score {i} values should be floats",
            )
            self.assertAlmostEqual(
                sum(score_list),
                1.0,
                places=6,
                msg=f"Score {i} probabilities should sum to 1",
            )

        # Verify usage
        self.assertIn("usage", response, "Response should have a 'usage' field")
        self.assertGreater(response["usage"]["prompt_tokens"], 0)
        self.assertEqual(
            response["usage"]["prompt_tokens"], response["usage"]["total_tokens"]
        )
        self.assertEqual(
            response["usage"]["completion_tokens"],
            0,
            "completion_tokens should be 0 for /v1/score",
        )
```
**EN:** Test scoring with token IDs input This test exercises `test_score_token_input` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test scoring with token IDs input 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_token_input`。

### Lines 1112-1135: test case score error handling / 测试用例 score error handling
```python
    def test_score_error_handling(self):
        """Test error handling for invalid inputs"""
        query = "The capital of France is"
        items = ["Paris", "London", "Berlin"]

        # Test with invalid token ID
        response = requests.post(
            self.base_url,
            headers={
                "Authorization": f"Bearer {self.api_key}",
                "Content-Type": "application/json",
            },
            json={
                "model": self.model,
                "query": query,
                "items": items,
                "label_token_ids": [999999],  # Invalid token ID
                "apply_softmax": True,
            },
        )
        self.assertEqual(response.status_code, 400)
        error_response = response.json()
        self.assertEqual(error_response["type"], "BadRequestError")
        self.assertIn("Token ID 999999 is out of vocabulary", error_response["message"])
```
**EN:** Test error handling for invalid inputs This test exercises `test_score_error_handling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test error handling for invalid inputs 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_score_error_handling`。

### Lines 1138-1139: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestOpenAIServer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOpenAIServerv1Responses`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOpenAIV1Rerank`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOpenAIServerCustomLogitProcessor`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOpenAIV1Score`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestOpenAIServer.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestOpenAIServer.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestOpenAIServer.run_completion`: This helper encapsulates `run_completion` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_completion`，以便周围测试复用准备、执行或校验逻辑。
- `TestOpenAIServer.run_completion_stream`: This helper encapsulates `run_completion_stream` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_completion_stream`，以便周围测试复用准备、执行或校验逻辑。
- `TestOpenAIServer.run_chat_completion`: This helper encapsulates `run_chat_completion` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_chat_completion`，以便周围测试复用准备、执行或校验逻辑。
- `TestOpenAIServer.run_chat_completion_stream`: This helper encapsulates `run_chat_completion_stream` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_chat_completion_stream`，以便周围测试复用准备、执行或校验逻辑。
- `TestOpenAIServer.test_completion`: This test exercises `test_completion` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `random`, `unittest`, `concurrent.futures`, `typing`
- **Third-party modules / 第三方模块**: `openai`, `requests`
- **Internal modules / 内部模块**: `sglang.srt.sampling.custom_logit_processor`, `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`

- **Total lines / 总行数**: 1139
