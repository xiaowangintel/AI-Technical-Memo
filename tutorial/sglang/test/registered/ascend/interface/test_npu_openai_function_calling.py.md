# test_npu_openai_function_calling.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/interface/test_npu_openai_function_calling.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on interface npu openai function calling in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 interface npu openai function calling 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Import dependencies
```python
import json
import unittest

import openai

from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
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

### Lines 17-22: Register CI metadata
```python
register_npu_ci(
    est_time=400,
    suite="nightly-1-npu-a3",
    nightly=True,
    disabled="https://github.com/Ascend/sglang/issues/39",
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 25-25: Define class TestOpenAIServerFunctionCalling
```python
class TestOpenAIServerFunctionCalling(CustomTestCase):
```
**EN:** This declaration introduces the `TestOpenAIServerFunctionCalling` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestOpenAIServerFunctionCalling` 测试类，并说明它通过继承承担的职责。

### Lines 26-31: Document the class `TestOpenAIServerFunctionCalling`
```python
    """Testcase：Verify the correctness of full-scenario OpenAI-style function calling with llama3 parser for Llama-3.2-1B-Instruct model.
        Cover: Single/multi-turn calls, streaming/non-streaming returns, multi-parameter verification of tool_choice, and JSON parsing validity of function parameters.

    [Test Category] Interface
    [Test Target] /v1/chat/completions
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestOpenAIServerFunctionCalling`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestOpenAIServerFunctionCalling`的设计意图。

### Lines 36-45: Declare TestOpenAIServerFunctionCalling configuration
```python
    SYSTEM_MESSAGE = (
        "You are a helpful assistant with tool calling capabilities. "
        "Only reply with a tool call if the function exists in the library provided by the user. "
        "If it doesn't exist, just reply directly in natural language. "
        "When you receive a tool call response, use the output to format an answer to the original user question. "
        "You have access to the following functions. "
        "To call a function, please respond with JSON for a function call. "
        'Respond in the format {"name": function name, "parameters": dictionary of argument name and its value}. '
        "Do not use variables.\n\n"
    )
```
**EN:** This block defines class-level settings that are shared across the `TestOpenAIServerFunctionCalling` test methods.
**CN:** 该代码块定义了 `TestOpenAIServerFunctionCalling` 各测试方法共享的类级配置。

### Lines 47-70: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        # Replace with the model name needed for testing
        cls.model = LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"

        # Start the local OpenAI Server. If necessary, you can add other parameters such as --enable-tools.
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=[
                # If your server needs extra parameters to test function calling, please add them here.
                "--attention-backend",
                "ascend",
                "--disable-cuda-graph",
                "--tool-call-parser",
                "llama3",
            ],
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(cls.model)
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 72-74: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 76-128: Run test: function calling format
```python
    def test_function_calling_format(self):
        """
        Test: Whether the function call format returned by the AI is correct.
        When returning a tool call, message.content should be None, and tool_calls should be a list.
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "add",
                    "description": "Compute the sum of two numbers",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "a": {
                                "type": "integer",
                                "description": "A number",
                            },
                            "b": {
                                "type": "integer",
                                "description": "A number",
                            },
                        },
                        "required": ["a", "b"],
                    },
                },
            }
        ]

        messages = [
            {"role": "system", "content": self.SYSTEM_MESSAGE},
            {"role": "user", "content": "Compute (3+5)"},
        ]
        response = client.chat.completions.create(
            model=self.model,
            max_tokens=2048,
            messages=messages,
            temperature=0.8,
            top_p=0.8,
            stream=False,
            tools=tools,
        )

        tool_calls = response.choices[0].message.tool_calls

        assert (
            isinstance(tool_calls, list) and len(tool_calls) > 0
        ), "tool_calls should be a non-empty list"

        function_name = tool_calls[0].function.name
        assert function_name == "add", "Function name should be 'add'"
```
**EN:** This test method exercises function calling format and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function calling format 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 131-181: Define helper: test function calling multiturn (part 1)
```python
    def _test_function_calling_multiturn(self):
        """
        Test: Whether the function call format returned by the AI is correct.
        When returning a tool call, message.content should be None, and tool_calls should be a list.
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "add",
                    "description": "Compute the sum of two numbers",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "a": {
                                "type": "integer",
                                "description": "A number",
                            },
                            "b": {
                                "type": "integer",
                                "description": "A number",
                            },
                        },
                        "required": ["a", "b"],
                    },
                },
            }
        ]

        messages = [{"role": "user", "content": "Compute (3+5)"}]

        response = client.chat.completions.create(
            model=self.model,
            max_tokens=2048,
            messages=messages,
            temperature=0.8,
            top_p=0.8,
            stream=False,
            tools=tools,
        )

        tool_call = response.choices[0].message.tool_calls[0]
        function_name = tool_call.function.name
        assert function_name == "add", "Function name should be 'add'"
        function_arguments = json.loads(tool_call.function.arguments)
        assert function_arguments in [
            {"a": 3, "b": 5},
            {"a": "3", "b": "5"},
        ], f"Unexpected function arguments: {function_arguments}"
```
**EN:** This helper function encapsulates reusable logic inside `TestOpenAIServerFunctionCalling` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestOpenAIServerFunctionCalling` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 183-201: Define helper: test function calling multiturn (part 2)
```python
        messages.append(response.choices[0].message)
        messages.append(
            {
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": "8",
                "name": function_name,
            }
        )

        final_response = client.chat.completions.create(
            model=self.model,
            max_tokens=2048,
            messages=messages,
            temperature=0.8,
            top_p=0.8,
            stream=False,
            tools=tools,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestOpenAIServerFunctionCalling` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestOpenAIServerFunctionCalling` 内部调用，从而让场景结构更清晰。

### Lines 203-205: Define helper: test function calling multiturn (part 3)
```python
        assert (
            "8" in final_response.choices[0].message.content
        ), "tool_call response should have the sum 8 in the content"
```
**EN:** This helper function encapsulates reusable logic inside `TestOpenAIServerFunctionCalling` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestOpenAIServerFunctionCalling` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 207-259: Run test: function calling streaming simple (part 1)
```python
    def test_function_calling_streaming_simple(self):
        """
        Test: Whether the function name can be correctly recognized in streaming mode.
        - Expect a function call to be found, and the function name to be correct.
        - Verify that streaming mode returns at least multiple chunks.
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "get_current_weather",
                    "description": "Get the current weather in a given location",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "The city to find the weather for",
                            },
                            "unit": {
                                "type": "string",
                                "description": "Weather unit (celsius or fahrenheit)",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["city", "unit"],
                    },
                },
            }
        ]

        messages = [
            {"role": "system", "content": self.SYSTEM_MESSAGE},
            {
                "role": "user",
                "content": "What is the temperature in Paris in celsius??",
            },
        ]

        response_stream = client.chat.completions.create(
            model=self.model,
            max_tokens=2048,
            messages=messages,
            temperature=0.8,
            top_p=0.8,
            stream=True,
            tools=tools,
        )

        chunks = list(response_stream)
        self.assertTrue(len(chunks) > 0, "Streaming should return at least one chunk")
```
**EN:** This test method exercises function calling streaming simple and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function calling streaming simple 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 261-279: Run test: function calling streaming simple (part 2)
```python
        found_function_name = False
        for chunk in chunks:
            choice = chunk.choices[0]
            # Check whether the current chunk contains tool_calls
            if choice.delta.tool_calls:
                tool_call = choice.delta.tool_calls[0]
                if tool_call.function.name:
                    self.assertEqual(
                        tool_call.function.name,
                        "get_current_weather",
                        "Function name should be 'get_current_weather'",
                    )
                    found_function_name = True
                    break

        self.assertTrue(
            found_function_name,
            "Target function name 'get_current_weather' was not found in the streaming chunks",
        )
```
**EN:** This test method exercises function calling streaming simple and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function calling streaming simple 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 281-286: Run test: function calling streaming simple (part 3)
```python
        finish_reason = chunks[-1].choices[0].finish_reason
        self.assertEqual(
            finish_reason,
            "tool_calls",
            "Final response of function calling should have finish_reason 'tool_calls'",
        )
```
**EN:** This test method exercises function calling streaming simple and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function calling streaming simple 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 288-334: Run test: function calling streaming args parsing (part 1)
```python
    def test_function_calling_streaming_args_parsing(self):
        """
        Test: Whether the function call arguments returned in streaming mode can be correctly concatenated into valid JSON.
        - The user request requires multiple parameters.
        - AI may return the arguments in chunks that need to be concatenated.
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "add",
                    "description": "Compute the sum of two integers",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "a": {
                                "type": "integer",
                                "description": "First integer",
                            },
                            "b": {
                                "type": "integer",
                                "description": "Second integer",
                            },
                        },
                        "required": ["a", "b"],
                    },
                    "strict": True,  # Llama-3.2-1B is flaky in tool call. It won't always respond with parameters unless we set strict.
                },
            }
        ]

        messages = [
            {"role": "system", "content": self.SYSTEM_MESSAGE},
            {"role": "user", "content": "Please sum 5 and 7, just call the function."},
        ]

        response_stream = client.chat.completions.create(
            model=self.model,
            max_tokens=2048,
            messages=messages,
            temperature=0.9,
            top_p=0.9,
            stream=True,
            tools=tools,
        )
```
**EN:** This test method exercises function calling streaming args parsing and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 function calling streaming args parsing 场景，并验证观测到的行为是否符合预期契约。

### Lines 336-369: Run test: function calling streaming args parsing (part 2)
```python
        argument_fragments = []
        chunks = list(response_stream)
        function_name = None
        for chunk in chunks:
            choice = chunk.choices[0]
            if choice.delta.tool_calls:
                tool_call = choice.delta.tool_calls[0]
                # Record the function name on first occurrence
                function_name = tool_call.function.name or function_name
                # In case of multiple chunks, JSON fragments may need to be concatenated
                if tool_call.function.arguments is not None:
                    argument_fragments.append(tool_call.function.arguments)

        self.assertEqual(function_name, "add", "Function name should be 'add'")
        joined_args = "".join(argument_fragments)
        self.assertTrue(
            len(joined_args) > 0,
            "No parameter fragments were returned in the function call",
        )

        finish_reason = chunks[-1].choices[0].finish_reason
        self.assertEqual(
            finish_reason,
            "tool_calls",
            "Final response of function calling should have finish_reason 'tool_calls'",
        )

        # Check whether the concatenated JSON is valid
        try:
            args_obj = json.loads(joined_args)
        except json.JSONDecodeError:
            self.fail(
                "The concatenated tool call arguments are not valid JSON, parsing failed"
            )
```
**EN:** This test method exercises function calling streaming args parsing and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function calling streaming args parsing 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 371-374: Run test: function calling streaming args parsing (part 3)
```python
        self.assertIn("a", args_obj, "Missing parameter 'a'")
        self.assertIn("b", args_obj, "Missing parameter 'b'")
        self.assertEqual(str(args_obj["a"]), "5", "Parameter a should be 5")
        self.assertEqual(str(args_obj["b"]), "7", "Parameter b should be 7")
```
**EN:** This test method exercises function calling streaming args parsing and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function calling streaming args parsing 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 376-428: Run test: function call strict
```python
    def test_function_call_strict(self):
        """
        Test: Whether the strict mode of function calling works as expected.
        - When strict mode is enabled, the AI should not return a function call if the function name is not recognized.
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "sub",
                    "description": "Compute the difference of two integers",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "int_a": {
                                "type": "integer",
                                "description": "First integer",
                            },
                            "int_b": {
                                "type": "integer",
                                "description": "Second integer",
                            },
                        },
                        "required": ["int_a", "int_b"],
                    },
                    "strict": True,
                },
            }
        ]

        messages = [
            {"role": "user", "content": "Please compute 5 - 7, using your tool."}
        ]
        response = client.chat.completions.create(
            model=self.model,
            max_tokens=2048,
            messages=messages,
            temperature=0.8,
            top_p=0.8,
            stream=False,
            tools=tools,
        )

        tool_calls = response.choices[0].message.tool_calls
        function_name = tool_calls[0].function.name
        arguments = tool_calls[0].function.arguments
        args_obj = json.loads(arguments)

        self.assertEqual(function_name, "sub", "Function name should be 'sub'")
        self.assertEqual(str(args_obj["int_a"]), "5", "Parameter int_a should be 5")
        self.assertEqual(str(args_obj["int_b"]), "7", "Parameter int_b should be 7")
```
**EN:** This test method exercises function call strict and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function call strict 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 430-477: Run test: function call required (part 1)
```python
    def test_function_call_required(self):
        """
        Test: Whether tool_choice: "required" works as expected
        - When tool_choice == "required", the model should return one or more tool_calls.
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "sub",
                    "description": "Compute the difference of two integers",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "int_a": {
                                "type": "integer",
                                "description": "First integer",
                            },
                            "int_b": {
                                "type": "integer",
                                "description": "Second integer",
                            },
                        },
                        "required": ["int_a", "int_b"],
                    },
                    "strict": True,
                },
            },
            {
                "type": "function",
                "function": {
                    "name": "get_weather",
                    "description": "use this to get latest weather information for a city given its name",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "name of the city to get weather for",
                            }
                        },
                        "required": ["city"],
                    },
                },
            },
        ]
```
**EN:** This test method exercises function call required and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 function call required 场景，并验证观测到的行为是否符合预期契约。

### Lines 479-504: Run test: function call required (part 2)
```python
        messages = [{"role": "user", "content": "What is the capital of France?"}]
        response = client.chat.completions.create(
            model=self.model,
            max_tokens=2048,
            messages=messages,
            temperature=0.8,
            top_p=0.8,
            stream=False,
            tools=tools,
            tool_choice="required",
        )

        tool_calls = response.choices[0].message.tool_calls
        self.assertIsNotNone(tool_calls, "No tool_calls in the response")
        function_name = tool_calls[0].function.name
        arguments = tool_calls[0].function.arguments
        args_obj = json.loads(arguments)

        self.assertEqual(
            function_name,
            "get_weather",
            f"Function name should be 'get_weather', got: {function_name}",
        )
        self.assertIn(
            "city", args_obj, f"Function arguments should have 'city', got: {args_obj}"
        )
```
**EN:** This test method exercises function call required and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function call required 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 506-516: Run test: function call required (part 3)
```python
        # Make the test more robust by checking type and accepting valid responses
        city_value = args_obj["city"]
        self.assertIsInstance(
            city_value,
            str,
            f"Parameter city should be a string, got: {type(city_value)}",
        )
        self.assertTrue(
            "Paris" in city_value or "France" in city_value,
            f"Parameter city should contain either 'Paris' or 'France', got: {city_value}",
        )
```
**EN:** This test method exercises function call required and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function call required 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 518-565: Run test: function call specific (part 1)
```python
    def test_function_call_specific(self):
        """
        Test: Whether tool_choice: ToolChoice works as expected
        - When tool_choice is a specific ToolChoice, the model should return one or more tool_calls.
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "sub",
                    "description": "Compute the difference of two integers",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "int_a": {
                                "type": "integer",
                                "description": "First integer",
                            },
                            "int_b": {
                                "type": "integer",
                                "description": "Second integer",
                            },
                        },
                        "required": ["int_a", "int_b"],
                    },
                    "strict": True,
                },
            },
            {
                "type": "function",
                "function": {
                    "name": "get_weather",
                    "description": "use this to get latest weather information for a city given its name",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "name of the city to get weather for",
                            }
                        },
                        "required": ["city"],
                    },
                },
            },
        ]
```
**EN:** This test method exercises function call specific and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 function call specific 场景，并验证观测到的行为是否符合预期契约。

### Lines 567-583: Run test: function call specific (part 2)
```python
        messages = [{"role": "user", "content": "What is the capital of France?"}]
        response = client.chat.completions.create(
            model=self.model,
            max_tokens=2048,
            messages=messages,
            temperature=0.8,
            top_p=0.8,
            stream=False,
            tools=tools,
            tool_choice={"type": "function", "function": {"name": "get_weather"}},
        )

        tool_calls = response.choices[0].message.tool_calls
        self.assertIsNotNone(tool_calls, "No tool_calls in the response")
        function_name = tool_calls[0].function.name
        arguments = tool_calls[0].function.arguments
        args_obj = json.loads(arguments)
```
**EN:** This test method exercises function call specific and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function call specific 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 585-588: Run test: function call specific (part 3)
```python
        self.assertEqual(
            function_name, "get_weather", "Function name should be 'get_weather'"
        )
        self.assertIn("city", args_obj, "Function arguments should have 'city'")
```
**EN:** This test method exercises function call specific and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function call specific 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 590-648: Run test: streaming multiple choices finish reason (part 1)
```python
    def test_streaming_multiple_choices_finish_reason(self):
        """
        Test: Verify that each choice gets its own finish_reason chunk in streaming mode with n > 1.
        This tests the fix for the bug where only the last index got a finish_reason chunk.
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "get_current_weather",
                    "description": "Get the current weather in a given location",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "location": {
                                "type": "string",
                                "description": "The city and state, e.g. San Francisco, CA",
                            },
                            "unit": {
                                "type": "string",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["location"],
                    },
                },
            }
        ]

        messages = [
            {"role": "user", "content": "What is the weather like in Los Angeles?"}
        ]

        # Request with n=2 to get multiple choices
        response_stream = client.chat.completions.create(
            model=self.model,
            messages=messages,
            max_tokens=2048,
            temperature=0.8,
            stream=True,
            tools=tools,
            tool_choice="required",  # Force tool calls
            n=2,  # Multiple choices
        )

        chunks = list(response_stream)

        # Track finish_reason chunks for each index
        finish_reason_chunks = {}
        for chunk in chunks:
            if chunk.choices:
                for choice in chunk.choices:
                    if choice.finish_reason is not None:
                        index = choice.index
                        if index not in finish_reason_chunks:
                            finish_reason_chunks[index] = []
                        finish_reason_chunks[index].append(choice.finish_reason)
```
**EN:** This test method exercises streaming multiple choices finish reason and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 streaming multiple choices finish reason 场景，并验证观测到的行为是否符合预期契约。

### Lines 650-663: Run test: streaming multiple choices finish reason (part 2)
```python
        # Verify we got finish_reason chunks for both indices
        self.assertEqual(
            len(finish_reason_chunks),
            2,
            f"Expected finish_reason chunks for 2 indices, got {len(finish_reason_chunks)}",
        )

        # Verify both index 0 and 1 have finish_reason
        self.assertIn(
            0, finish_reason_chunks, "Missing finish_reason chunk for index 0"
        )
        self.assertIn(
            1, finish_reason_chunks, "Missing finish_reason chunk for index 1"
        )
```
**EN:** This test method exercises streaming multiple choices finish reason and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 streaming multiple choices finish reason 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 665-671: Run test: streaming multiple choices finish reason (part 3)
```python
        # Verify the finish_reason is "tool_calls" since we forced tool calls
        for index, reasons in finish_reason_chunks.items():
            self.assertEqual(
                reasons[-1],  # Last finish_reason for this index
                "tool_calls",
                f"Expected finish_reason 'tool_calls' for index {index}, got {reasons[-1]}",
            )
```
**EN:** This test method exercises streaming multiple choices finish reason and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 streaming multiple choices finish reason 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 673-731: Run test: function calling streaming no tool call (part 1)
```python
    def test_function_calling_streaming_no_tool_call(self):
        """
        Test: Whether the finish_reason is stop in streaming mode when no tool call is given.
        - Expect no function call to be found.
        - Verify that finish_reason is stop
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        tools = [
            {
                "type": "function",
                "function": {
                    "name": "get_current_weather",
                    "description": "Get the current weather in a given location",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "The city to find the weather for",
                            },
                            "unit": {
                                "type": "string",
                                "description": "Weather unit (celsius or fahrenheit)",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["city", "unit"],
                    },
                },
            }
        ]

        messages = [{"role": "user", "content": "Who are you?"}]

        response_stream = client.chat.completions.create(
            model=self.model,
            max_tokens=2048,
            messages=messages,
            temperature=0.8,
            top_p=0.8,
            stream=True,
            tools=tools,
            tool_choice="none",
        )

        chunks = list(response_stream)
        self.assertTrue(len(chunks) > 0, "Streaming should return at least one chunk")

        found_tool_call = False
        for chunk in chunks:
            choice = chunk.choices[0]
            # Check whether the current chunk contains tool_calls
            found_tool_call = choice.delta.tool_calls is not None

        self.assertFalse(
            found_tool_call,
            "Shouldn't have any tool_call in the streaming chunks",
        )
```
**EN:** This test method exercises function calling streaming no tool call and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function calling streaming no tool call 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 733-738: Run test: function calling streaming no tool call (part 2)
```python
        finish_reason = chunks[-1].choices[0].finish_reason
        self.assertEqual(
            finish_reason,
            "stop",
            "Final response of no function calling should have finish_reason 'stop'",
        )
```
**EN:** This test method exercises function calling streaming no tool call and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 function calling streaming no tool call 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 740-793: Run test: streaming multiple choices without tools
```python
    def test_streaming_multiple_choices_without_tools(self):
        """
        Test: Verify that each choice gets its own finish_reason chunk without tool calls.
        This tests the fix for regular content streaming with multiple choices.
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)

        messages = [{"role": "user", "content": "Say hello in one word."}]

        # Request with n=2 to get multiple choices, no tools
        response_stream = client.chat.completions.create(
            model=self.model,
            messages=messages,
            temperature=0.8,
            stream=True,
            max_tokens=10,  # Keep it short
            n=2,  # Multiple choices
        )

        chunks = list(response_stream)

        # Track finish_reason chunks for each index
        finish_reason_chunks = {}
        for chunk in chunks:
            if chunk.choices:
                for choice in chunk.choices:
                    if choice.finish_reason is not None:
                        index = choice.index
                        if index not in finish_reason_chunks:
                            finish_reason_chunks[index] = []
                        finish_reason_chunks[index].append(choice.finish_reason)

        # Verify we got finish_reason chunks for both indices
        self.assertEqual(
            len(finish_reason_chunks),
            2,
            f"Expected finish_reason chunks for 2 indices, got {len(finish_reason_chunks)}",
        )

        # Verify both index 0 and 1 have finish_reason
        self.assertIn(
            0, finish_reason_chunks, "Missing finish_reason chunk for index 0"
        )
        self.assertIn(
            1, finish_reason_chunks, "Missing finish_reason chunk for index 1"
        )

        # Verify the finish_reason is "stop" (regular completion)
        for index, reasons in finish_reason_chunks.items():
            self.assertIn(
                reasons[-1],
                ["stop", "length"],  # Could be either depending on how model responds
                f"Expected finish_reason 'stop' or 'length' for index {index}, got {reasons[-1]}",
            )
```
**EN:** This test method exercises streaming multiple choices without tools and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 streaming multiple choices without tools 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 796-796: Define class TestOpenAIPythonicFunctionCalling
```python
class TestOpenAIPythonicFunctionCalling(CustomTestCase):
```
**EN:** This declaration introduces the `TestOpenAIPythonicFunctionCalling` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestOpenAIPythonicFunctionCalling` 测试类，并说明它通过继承承担的职责。

### Lines 797-802: Document the class `TestOpenAIPythonicFunctionCalling`
```python
    """Testcase：Verify the functionality of Python-style list-format function calling with pythonic parser for Llama-3.2-1B-Instruct model on Ascend NPU backend.
    Cover: Explicit format prompt verification, streaming call index integrity, and return validity of parallel tool calls.

    [Test Category] Interface
    [Test Target] /v1/chat/completions
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestOpenAIPythonicFunctionCalling`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestOpenAIPythonicFunctionCalling`的设计意图。

### Lines 804-860: Declare TestOpenAIPythonicFunctionCalling configuration
```python
    PYTHONIC_TOOLS = [
        {
            "type": "function",
            "function": {
                "name": "get_weather",
                "description": "Get the current weather for a given location.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {
                            "type": "string",
                            "description": "The name of the city or location.",
                        }
                    },
                    "required": ["location"],
                },
            },
        },
        {
            "type": "function",
            "function": {
                "name": "get_tourist_attractions",
                "description": "Get a list of top tourist attractions for a given city.",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "city": {
                            "type": "string",
                            "description": "The name of the city to find attractions for.",
                        }
                    },
                    "required": ["city"],
                },
            },
        },
    ]

    PYTHONIC_MESSAGES = [
        {
            "role": "system",
            "content": (
                "You are a travel assistant. "
                "When asked to call functions, ALWAYS respond ONLY with a python list of function calls, "
                "using this format: [func_name1(param1=value1, param2=value2), func_name2(param=value)]. "
                "Do NOT use JSON, do NOT use variables, do NOT use any other format. "
                "Here is an example:\n"
                '[get_weather(location="Paris"), get_tourist_attractions(city="Paris")]'
            ),
        },
        {
            "role": "user",
            "content": (
                "I'm planning a trip to Tokyo next week. What's the weather like and what are some top tourist attractions? "
                "Propose parallel tool calls at once, using the python list of function calls format as shown above."
            ),
        },
    ]
```
**EN:** This block defines class-level settings that are shared across the `TestOpenAIPythonicFunctionCalling` test methods.
**CN:** 该代码块定义了 `TestOpenAIPythonicFunctionCalling` 各测试方法共享的类级配置。

### Lines 862-881: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = LLAMA_3_2_1B_INSTRUCT_WEIGHTS_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=[
                "--attention-backend",
                "ascend",
                "--disable-cuda-graph",
                "--tool-call-parser",
                "pythonic",
            ],
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(cls.model)
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite. It also launches a model server for the scenario.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。 其中还会启动场景所需的模型服务。

### Lines 883-885: Clean up shared test fixture
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This class-level teardown releases the resources created during setup so later runs start cleanly. It also releases spawned processes after the checks finish.
**CN:** 该类级清理会释放初始化阶段创建的资源，确保后续运行从干净状态开始。 其中还会在检查完成后释放已启动的进程。

### Lines 887-906: Run test: pythonic tool call prompt
```python
    def test_pythonic_tool_call_prompt(self):
        """
        Test: Explicit prompt for pythonic tool call format without chat template.
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        response = client.chat.completions.create(
            model=self.model,
            messages=self.PYTHONIC_MESSAGES,
            tools=self.PYTHONIC_TOOLS,
            temperature=0.1,
            stream=False,
        )
        tool_calls = response.choices[0].message.tool_calls
        self.assertIsInstance(tool_calls, list, "No tool_calls found")
        self.assertGreaterEqual(len(tool_calls), 1)
        names = [tc.function.name for tc in tool_calls]
        self.assertTrue(
            "get_weather" in names or "get_tourist_attractions" in names,
            f"Function name '{names}' should container either 'get_weather' or 'get_tourist_attractions'",
        )
```
**EN:** This test method exercises pythonic tool call prompt and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 pythonic tool call prompt 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 908-939: Run test: pythonic tool call streaming
```python
    def test_pythonic_tool_call_streaming(self):
        """
        Test: Streaming pythonic tool call format; assert tool_call index is present.
        """
        client = openai.Client(api_key=self.api_key, base_url=self.base_url)
        response_stream = client.chat.completions.create(
            model=self.model,
            messages=self.PYTHONIC_MESSAGES,
            tools=self.PYTHONIC_TOOLS,
            temperature=0.1,
            stream=True,
        )
        found_tool_calls = False
        found_index = False
        found_names = set()
        for chunk in response_stream:
            choice = chunk.choices[0]
            if getattr(choice.delta, "tool_calls", None):
                found_tool_calls = True
                tool_call = choice.delta.tool_calls[0]
                if hasattr(tool_call, "index") or (
                    isinstance(tool_call, dict) and "index" in tool_call
                ):
                    found_index = True
                found_names.add(str(tool_call.function.name))

        self.assertTrue(found_tool_calls, "No tool_calls found in streaming response")
        self.assertTrue(found_index, "No index field found in any streamed tool_call")
        self.assertTrue(
            "get_weather" in found_names or "get_tourist_attractions" in found_names,
            f"Function name '{found_names}' should container either 'get_weather' or 'get_tourist_attractions'",
        )
```
**EN:** This test method exercises pythonic tool call streaming and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 pythonic tool call streaming 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 942-943: Expose unittest entrypoint
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
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `json`, `openai`, `unittest`
- Notable symbols / 关键符号: `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
