# test_tool_choice.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/openai_server/function_call/test_tool_choice.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `tool choice` scenario in `test/manual/openai_server/function_call`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual/openai_server/function_call` 中的 `tool choice` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Process control logic / 进程控制逻辑
```python
"""
Test script for tool_choice functionality in SGLang
Tests: required, auto, and specific function choices in both streaming and non-streaming modes

# To run the tests, use the following command:
#
# python3 -m unittest openai_server.function_call.test_tool_choice
"""

import json
import unittest

import openai

from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。

### Lines 25-27: Class definition for TestToolChoiceLlama32 / 类定义
```python
class TestToolChoiceLlama32(CustomTestCase):

    @classmethod
```
**EN:** This range declares `TestToolChoiceLlama32`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 28-49: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        # Mark flaky tests for this model
        cls.flaky_tests = {
            "test_multi_tool_scenario_auto",
            "test_multi_tool_scenario_required",
        }

        # Use a model that supports function calling
        cls.model = "meta-llama/Llama-3.2-1B-Instruct"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"

        # Start the local OpenAI Server with tool calling support
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=[
                "--tool-call-parser",
                "llama3",  # Default parser for the test model
            ],
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 50-52: Request and response handling / 请求与响应处理
```python
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(cls.model)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get_tokenizer`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 53-54: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 55-56: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 58-60: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUp(self):
        self.client = openai.Client(base_url=self.base_url, api_key=self.api_key)
        self.model_name = self.client.models.list().data[0].id
```
**EN:** This range implements lifecycle helper(s) `setUp` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior. Representative call sites include `Client`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 62-67: Helper routines around _is_flaky_test / 辅助例程
```python
    def _is_flaky_test(self):
        """Check if the current test is marked as flaky for this class"""
        return (
            hasattr(self.__class__, "flaky_tests")
            and self._testMethodName in self.__class__.flaky_tests
        )
```
**EN:** This range implements helper routine(s) `_is_flaky_test` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `hasattr`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 69-74: Helper routines around get_test_tools / 辅助例程
```python
    def get_test_tools(self):
        """Get the test tools for function calling"""
        return [
            {
                "type": "function",
                "function": {
```
**EN:** This range implements helper routine(s) `get_test_tools` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 75-93: Scenario logic / 场景逻辑
```python
                    "name": "get_weather",
                    "description": "use this to get latest weather information for a city given its name",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "city": {
                                "type": "string",
                                "description": "name of the city to get weather for",
                            },
                            "unit": {
                                "type": "string",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["city"],
                    },
                },
            },
            {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 94-99: Scenario logic / 场景逻辑
```python
                "type": "function",
                "function": {
                    "name": "get_pokemon_info",
                    "description": "get detailed information about a pokemon given its name",
                    "parameters": {
                        "type": "object",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 100-118: Scenario logic / 场景逻辑
```python
                        "properties": {
                            "name": {
                                "type": "string",
                                "description": "name of the pokemon to get info for",
                            }
                        },
                        "required": ["name"],
                    },
                },
            },
            {
                "type": "function",
                "function": {
                    "name": "make_next_step_decision",
                    "description": "You will be given a trace of thinking process in the following format.\n\nQuestion: the input question you must answer\nTOOL: think about what to do, and choose a tool to use ONLY IF there are defined tools. \n  You should never call the same tool with the same input twice in a row.\n  If the previous conversation history already contains the information that can be retrieved from the tool, you should not call the tool again.\nOBSERVATION: the result of the tool call, NEVER include this in your response, this information will be provided\n... (this TOOL/OBSERVATION can repeat N times)\nANSWER: If you know the answer to the original question, require for more information,\n  or you don't know the answer and there are no defined tools or all available tools are not helpful, respond with the answer without mentioning anything else.\n  If the previous conversation history already contains the answer, respond with the answer right away.\n\n  If no tools are configured, naturally mention this limitation while still being helpful. Briefly note that adding tools in the agent configuration would expand capabilities.\n\nYour task is to respond with the next step to take, based on the traces, \nor answer the question if you have enough information.",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "decision": {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 119-124: Scenario logic / 场景逻辑
```python
                                "type": "string",
                                "description": 'The next step to take, it must be either "TOOL" or "ANSWER". If the previous conversation history already contains the information that can be retrieved from the tool, you should not call the tool again. If there are no defined tools, you should not return "TOOL" in your response.',
                            },
                            "content": {
                                "type": "string",
                                "description": 'The content of the next step. If the decision is "TOOL", this should be a short and concise reasoning of why you chose the tool, MUST include the tool name. If the decision is "ANSWER", this should be the answer to the question. If no tools are available, integrate this limitation conversationally without sounding scripted.',
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 125-131: Scenario logic / 场景逻辑
```python
                            },
                        },
                        "required": ["decision", "content"],
                    },
                },
            },
        ]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 133-140: Helper routines around get_test_messages / 辅助例程
```python
    def get_test_messages(self):
        """Get test messages that should trigger tool usage"""
        return [
            {
                "role": "user",
                "content": "Answer the following questions as best you can:\n\nYou will be given a trace of thinking process in the following format.\n\nQuestion: the input question you must answer\nTOOL: think about what to do, and choose a tool to use ONLY IF there are defined tools\nOBSERVATION: the result of the tool call or the observation of the current task, NEVER include this in your response, this information will be provided\n... (this TOOL/OBSERVATION can repeat N times)\nANSWER: If you know the answer to the original question, require for more information, \nif the previous conversation history already contains the answer, \nor you don't know the answer and there are no defined tools or all available tools are not helpful, respond with the answer without mentioning anything else.\nYou may use light Markdown formatting to improve clarity (e.g. lists, **bold**, *italics*), but keep it minimal and unobtrusive.\n\nYour task is to respond with the next step to take, based on the traces, \nor answer the question if you have enough information.\n\nQuestion: what is the weather in top 5 populated cities in the US in celsius?\n\nTraces:\n\n\nThese are some additional instructions that you should follow:",
            }
        ]
```
**EN:** This range implements helper routine(s) `get_test_messages` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `clarity`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 142-149: Helper routines around get_travel_tools / 辅助例程
```python
    def get_travel_tools(self):
        """Get tools for travel assistant scenario that should trigger multiple tool calls"""
        return [
            {
                "type": "function",
                "function": {
                    "name": "get_weather",
                    "description": "Get the current weather for a given location.",
```
**EN:** This range implements helper routine(s) `get_travel_tools` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 150-166: Scenario logic / 场景逻辑
```python
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "location": {
                                "type": "string",
                                "description": "The name of the city or location.",
                            },
                            "unit": {
                                "type": "string",
                                "enum": ["celsius", "fahrenheit"],
                            },
                        },
                        "required": ["location"],
                    },
                },
            },
            {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 167-174: Scenario logic / 场景逻辑
```python
                "type": "function",
                "function": {
                    "name": "get_tourist_attractions",
                    "description": "Get a list of top tourist attractions for a given city.",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "city": {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 175-183: Scenario logic / 场景逻辑
```python
                                "type": "string",
                                "description": "The name of the city to find attractions for.",
                            }
                        },
                        "required": ["city"],
                    },
                },
            },
        ]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 185-196: Helper routines around get_travel_messages / 辅助例程
```python
    def get_travel_messages(self):
        """Get travel assistant messages that should trigger multiple tool calls"""
        return [
            {
                "content": "You are a travel assistant providing real-time weather updates and top tourist attractions.",
                "role": "system",
            },
            {
                "content": "I'm planning a trip to Tokyo next week. What's the weather like? What are the most amazing sights?",
                "role": "user",
            },
        ]
```
**EN:** This range implements helper routine(s) `get_travel_messages` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 198-199: Test routines around test_tool_choice_auto_non_streaming / 测试例程
```python
    def test_tool_choice_auto_non_streaming(self):
        """Test tool_choice='auto' in non-streaming mode"""
```
**EN:** This range defines concrete test routine(s) `test_tool_choice_auto_non_streaming`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 200-212: Assertions and result checks / 断言与结果检查
```python
        tools = self.get_test_tools()
        messages = self.get_test_messages()

        response = self.client.chat.completions.create(
            model=self.model_name,
            messages=messages,
            max_tokens=2048,
            tools=tools,
            tool_choice="auto",
            stream=False,
        )

        self.assertIsNotNone(response.choices[0].message)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get_test_tools`, `get_test_messages`, `create` and `assertIsNotNone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 213-214: Scenario logic / 场景逻辑
```python
        # With auto, tool calls are optional
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 215-224: Test routines around test_tool_choice_auto_streaming / 测试例程
```python
    def test_tool_choice_auto_streaming(self):
        """Test tool_choice='auto' in streaming mode"""
        tools = self.get_test_tools()
        messages = self.get_test_messages()

        response = self.client.chat.completions.create(
            model=self.model_name,
            messages=messages,
            max_tokens=2048,
            tools=tools,
```
**EN:** This range defines concrete test routine(s) `test_tool_choice_auto_streaming`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get_test_tools`, `get_test_messages` and `create`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 225-239: Scenario logic / 场景逻辑
```python
            tool_choice="auto",
            stream=True,
        )

        # Collect streaming response
        content_chunks = []
        tool_call_chunks = []

        for chunk in response:
            if chunk.choices[0].delta.content:
                content_chunks.append(chunk.choices[0].delta.content)
            elif chunk.choices[0].delta.tool_calls:
                tool_call_chunks.extend(chunk.choices[0].delta.tool_calls)

        # Should complete without errors
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `append` and `extend`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 240-241: Assertions and result checks / 断言与结果检查
```python
        self.assertIsInstance(content_chunks, list)
        self.assertIsInstance(tool_call_chunks, list)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsInstance`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 243-249: Test routines around test_tool_choice_required_non_streaming / 测试例程
```python
    def test_tool_choice_required_non_streaming(self):
        """Test tool_choice='required' in non-streaming mode"""
        tools = self.get_test_tools()
        messages = self.get_test_messages()

        response = self.client.chat.completions.create(
            model=self.model_name,
```
**EN:** This range defines concrete test routine(s) `test_tool_choice_required_non_streaming`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get_test_tools`, `get_test_messages` and `create`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 250-261: Assertions and result checks / 断言与结果检查
```python
            messages=messages,
            max_tokens=2048,
            temperature=0.2,
            tools=tools,
            tool_choice="required",
            stream=False,
        )

        # With required, we should get tool calls
        tool_calls = response.choices[0].message.tool_calls
        self.assertIsNotNone(tool_calls)
        self.assertGreater(len(tool_calls), 0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsNotNone` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 263-274: Test routines around test_tool_choice_required_streaming / 测试例程
```python
    def test_tool_choice_required_streaming(self):
        """Test tool_choice='required' in streaming mode"""
        tools = self.get_test_tools()
        messages = self.get_test_messages()

        response = self.client.chat.completions.create(
            model=self.model_name,
            messages=messages,
            max_tokens=2048,
            tools=tools,
            tool_choice="required",
            stream=True,
```
**EN:** This range defines concrete test routine(s) `test_tool_choice_required_streaming`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get_test_tools`, `get_test_messages` and `create`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 275-285: Assertions and result checks / 断言与结果检查
```python
        )

        # Collect streaming response
        tool_call_chunks = []

        for chunk in response:
            if chunk.choices[0].delta.tool_calls:
                tool_call_chunks.extend(chunk.choices[0].delta.tool_calls)

        # With required, we should get tool call chunks
        self.assertGreater(len(tool_call_chunks), 0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `extend` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 287-299: Test routines around test_tool_choice_specific_function_non_streaming / 测试例程
```python
    def test_tool_choice_specific_function_non_streaming(self):
        """Test tool_choice with specific function in non-streaming mode"""
        tools = self.get_test_tools()
        messages = self.get_test_messages()

        tool_choice = {"type": "function", "function": {"name": "get_weather"}}

        response = self.client.chat.completions.create(
            model=self.model_name,
            messages=messages,
            max_tokens=2048,
            tools=tools,
            tool_choice=tool_choice,
```
**EN:** This range defines concrete test routine(s) `test_tool_choice_specific_function_non_streaming`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get_test_tools`, `get_test_messages` and `create`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 300-309: Assertions and result checks / 断言与结果检查
```python
            stream=False,
        )

        # Should call the specific function
        tool_calls = response.choices[0].message.tool_calls
        self.assertIsNotNone(tool_calls)
        # Our messages ask the top 5 populated cities in the US, so the model could get 5 tool calls
        self.assertGreaterEqual(len(tool_calls), 1)
        for tool_call in tool_calls:
            self.assertEqual(tool_call.function.name, "get_weather")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsNotNone`, `assertGreaterEqual` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 311-324: Test routines around test_tool_choice_specific_function_streaming / 测试例程
```python
    def test_tool_choice_specific_function_streaming(self):
        """Test tool_choice with specific function in streaming mode"""
        tools = self.get_test_tools()
        messages = self.get_test_messages()

        tool_choice = {"type": "function", "function": {"name": "get_weather"}}

        response = self.client.chat.completions.create(
            model=self.model_name,
            messages=messages,
            max_tokens=2048,
            tools=tools,
            tool_choice=tool_choice,
            stream=True,
```
**EN:** This range defines concrete test routine(s) `test_tool_choice_specific_function_streaming`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get_test_tools`, `get_test_messages` and `create`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 325-335: Assertions and result checks / 断言与结果检查
```python
        )

        # Collect streaming response
        tool_call_chunks = []

        for chunk in response:
            if chunk.choices[0].delta.tool_calls:
                tool_call_chunks.extend(chunk.choices[0].delta.tool_calls)

        # Should get tool call chunks for the specific function
        self.assertGreater(len(tool_call_chunks), 0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `extend` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 336-344: Assertions and result checks / 断言与结果检查
```python

        # Find function name in chunks
        found_name = None
        for chunk in tool_call_chunks:
            if chunk.function and chunk.function.name:
                found_name = chunk.function.name
                break

        self.assertEqual(found_name, "get_weather")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 346-349: Test routines around test_required_streaming_arguments_chunks_json / 测试例程
```python
    def test_required_streaming_arguments_chunks_json(self):
        """In streaming required mode, complete tool call arguments should be valid JSON when all chunks are combined.
        Uses strict=True so the grammar enforces the parameter schema."""
        tools = self.get_test_tools()
```
**EN:** This range defines concrete test routine(s) `test_required_streaming_arguments_chunks_json`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `get_test_tools`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 350-370: Request and response handling / 请求与响应处理
```python
        # Add strict=True so arguments are schema-constrained
        for tool in tools:
            tool["function"]["strict"] = True
        messages = self.get_test_messages()

        response = self.client.chat.completions.create(
            model=self.model_name,
            messages=messages,
            max_tokens=1024,
            temperature=0.1,
            tools=tools,
            tool_choice="required",
            stream=True,
        )

        # Collect all tool call chunks and reconstruct complete tool calls
        tool_calls_by_index = {}
        for chunk in response:
            if chunk.choices[0].delta.tool_calls:
                for tool_call_delta in chunk.choices[0].delta.tool_calls:
                    tool_index = tool_call_delta.index
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get_test_messages` and `create`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 371-374: Scenario logic / 场景逻辑
```python

                    # Initialize tool call if not seen before
                    if tool_index not in tool_calls_by_index:
                        tool_calls_by_index[tool_index] = {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 375-395: Assertions and result checks / 断言与结果检查
```python
                            "id": tool_call_delta.id,
                            "type": "function",
                            "function": {"name": "", "arguments": ""},
                        }

                    # Update function name if present (first chunk)
                    if tool_call_delta.function and tool_call_delta.function.name:
                        tool_calls_by_index[tool_index]["function"][
                            "name"
                        ] = tool_call_delta.function.name

                    # Accumulate arguments (all chunks)
                    if tool_call_delta.function and tool_call_delta.function.arguments:
                        tool_calls_by_index[tool_index]["function"][
                            "arguments"
                        ] += tool_call_delta.function.arguments

        self.assertGreater(len(tool_calls_by_index), 0)

        # Validate that complete tool calls have valid JSON arguments
        for tool_call in tool_calls_by_index.values():
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `present`, `arguments`, `assertGreater` and `values`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 396-399: Assertions and result checks / 断言与结果检查
```python
            self.assertIsNotNone(tool_call["function"]["name"])
            self.assertIsNotNone(tool_call["function"]["arguments"])

            # The complete arguments should be valid JSON
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsNotNone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 400-406: Assertions and result checks / 断言与结果检查
```python
            try:
                args = json.loads(tool_call["function"]["arguments"])
                self.assertIsInstance(args, dict)
            except json.JSONDecodeError:
                self.fail(
                    f"Invalid JSON in complete tool call arguments: {tool_call['function']['arguments']}"
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `loads`, `assertIsInstance` and `fail`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 408-424: Test routines around test_complex_parameters_required_non_streaming / 测试例程
```python
    def test_complex_parameters_required_non_streaming(self):
        """Validate complex nested parameter schemas in non-streaming required mode.
        Uses strict=True so the grammar enforces the parameter schema."""
        complex_tools = [
            {
                "type": "function",
                "function": {
                    "name": "analyze_data",
                    "description": "Analyze complex data structures",
                    "strict": True,
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "data": {
                                "type": "object",
                                "properties": {
                                    "metrics": {
```
**EN:** This range defines concrete test routine(s) `test_complex_parameters_required_non_streaming`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 425-432: Scenario logic / 场景逻辑
```python
                                        "type": "array",
                                        "items": {"type": "string"},
                                    },
                                    "config": {
                                        "type": "object",
                                        "properties": {
                                            "threshold": {"type": "number"},
                                            "enabled": {"type": "boolean"},
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 433-449: Scenario logic / 场景逻辑
```python
                                        },
                                    },
                                },
                                "required": ["metrics"],
                            },
                            "options": {
                                "type": "array",
                                "items": {
                                    "type": "object",
                                    "properties": {
                                        "name": {"type": "string"},
                                        "value": {"type": "string"},
                                    },
                                },
                            },
                        },
                        "required": ["data"],
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 450-457: Scenario logic / 场景逻辑
```python
                    },
                },
            }
        ]

        messages = [
            {
                "role": "user",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 458-474: Assertions and result checks / 断言与结果检查
```python
                "content": "Analyze some data with metrics and configuration",
            }
        ]

        response = self.client.chat.completions.create(
            model=self.model_name,
            messages=messages,
            max_tokens=1024,
            temperature=0.1,
            tools=complex_tools,
            tool_choice="required",
            stream=False,
        )

        tool_calls = response.choices[0].message.tool_calls
        self.assertIsNotNone(tool_calls)
        self.assertGreater(len(tool_calls), 0)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `create`, `assertIsNotNone` and `assertGreater`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 475-482: Assertions and result checks / 断言与结果检查
```python

        for tool_call in tool_calls:
            self.assertEqual(tool_call.function.name, "analyze_data")
            try:
                args = json.loads(tool_call.function.arguments)
                self.assertIsInstance(args, dict)
                self.assertIn("data", args)
                self.assertIsInstance(args["data"], dict)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`, `loads`, `assertIsInstance` and `assertIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 483-486: Scenario logic / 场景逻辑
```python
            except json.JSONDecodeError:
                self.fail(
                    f"Invalid JSON in complex tool call arguments: {tool_call.function.arguments}"
                )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `fail`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 488-499: Test routines around test_multi_tool_scenario_auto / 测试例程
```python
    def test_multi_tool_scenario_auto(self):
        """Test multi-tool scenario with tool_choice='auto'"""
        tools = self.get_travel_tools()
        messages = self.get_travel_messages()

        response = self.client.chat.completions.create(
            model=self.model_name,
            messages=messages,
            max_tokens=2048,
            temperature=0.2,
            tools=tools,
            tool_choice="auto",
```
**EN:** This range defines concrete test routine(s) `test_multi_tool_scenario_auto`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get_travel_tools`, `get_travel_messages` and `create`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 500-512: Assertions and result checks / 断言与结果检查
```python
            stream=False,
        )

        # Should complete without errors
        self.assertIsNotNone(response.choices[0].message)

        tool_calls = response.choices[0].message.tool_calls
        expected_functions = {"get_weather", "get_tourist_attractions"}

        if self._is_flaky_test():
            # For flaky tests, just verify all called functions are available tools
            if tool_calls:
                available_names = [tool["function"]["name"] for tool in tools]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsNotNone` and `_is_flaky_test`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 513-524: Assertions and result checks / 断言与结果检查
```python
                for call in tool_calls:
                    self.assertIn(call.function.name, available_names)
        else:
            # For non-flaky tests, enforce strict requirements
            self.assertIsNotNone(tool_calls, "Expected tool calls but got none")
            self.assertEqual(
                len(tool_calls), 2, f"Expected 2 tool calls, got {len(tool_calls)}"
            )

            called_functions = {call.function.name for call in tool_calls}
            self.assertEqual(
                called_functions,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIn`, `assertIsNotNone` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 525-527: Scenario logic / 场景逻辑
```python
                expected_functions,
                f"Expected functions {expected_functions}, got {called_functions}",
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 529-549: Test routines around test_multi_tool_scenario_required / 测试例程
```python
    def test_multi_tool_scenario_required(self):
        """Test multi-tool scenario with tool_choice='required'"""
        tools = self.get_travel_tools()
        messages = self.get_travel_messages()

        response = self.client.chat.completions.create(
            model=self.model_name,
            messages=messages,
            max_tokens=2048,
            temperature=0.2,
            tools=tools,
            tool_choice="required",
            stream=False,
        )

        # With required, we should get at least one tool call
        tool_calls = response.choices[0].message.tool_calls
        self.assertIsNotNone(tool_calls)
        self.assertGreater(len(tool_calls), 0)

        # Verify all called functions are available tools
```
**EN:** This range defines concrete test routine(s) `test_multi_tool_scenario_required`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get_travel_tools`, `get_travel_messages`, `create` and `assertIsNotNone`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 550-553: Scenario logic / 场景逻辑
```python
        available_names = [tool["function"]["name"] for tool in tools]
        expected_functions = {"get_weather", "get_tourist_attractions"}

        for tool_call in tool_calls:
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 554-574: Assertions and result checks / 断言与结果检查
```python
            self.assertIsNotNone(tool_call.function.name)
            self.assertIsNotNone(tool_call.function.arguments)

        if self._is_flaky_test():
            # For flaky tests, just ensure basic functionality works
            self.assertGreater(
                len(tool_calls),
                0,
                f"Expected at least 1 tool call, got {len(tool_calls)}",
            )
            for call in tool_calls:
                self.assertIn(call.function.name, available_names)
        else:
            # For non-flaky tests, enforce strict requirements
            self.assertEqual(
                len(tool_calls), 2, f"Expected 2 tool calls, got {len(tool_calls)}"
            )

            called_functions = {call.function.name for call in tool_calls}
            self.assertEqual(
                called_functions,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIsNotNone`, `_is_flaky_test`, `assertGreater` and `assertIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 575-577: Scenario logic / 场景逻辑
```python
                expected_functions,
                f"Expected functions {expected_functions}, got {called_functions}",
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 579-599: Test routines around test_error_handling_invalid_tool_choice / 测试例程
```python
    def test_error_handling_invalid_tool_choice(self):
        """Test error handling for invalid tool_choice"""
        tools = self.get_test_tools()
        messages = self.get_test_messages()

        # Test with invalid function name
        tool_choice = {"type": "function", "function": {"name": "nonexistent_function"}}

        # Expect a 400 BadRequestError to be raised for invalid tool_choice
        with self.assertRaises(openai.BadRequestError) as context:
            self.client.chat.completions.create(
                model=self.model_name,
                messages=messages,
                max_tokens=2048,
                tools=tools,
                tool_choice=tool_choice,
                stream=False,
            )

        # Verify the error message contains the expected text
        self.assertIn(
```
**EN:** This range defines concrete test routine(s) `test_error_handling_invalid_tool_choice`. The logic drives the target scenario and encodes the expected acceptance criteria. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `get_test_tools`, `get_test_messages`, `assertRaises` and `create`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 600-602: Scenario logic / 场景逻辑
```python
            "Tool 'nonexistent_function' not found in tools list",
            str(context.exception),
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 604-624: Test routines around test_invalid_tool_missing_name / 测试例程
```python
    def test_invalid_tool_missing_name(self):
        """Test what happens when user doesn't provide a tool name in request"""
        # Test with malformed JSON in tool parameters - missing required "name" field
        invalid_tools = [
            {
                "type": "function",
                "function": {
                    # Missing required "name" field
                    "description": "Test function with invalid schema",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "test_field": {
                                "type": "string",
                                "description": "Test field",
                            }
                        },
                        "required": ["test_field"],
                    },
                },
            }
```
**EN:** This range defines concrete test routine(s) `test_invalid_tool_missing_name`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 625-628: Scenario logic / 场景逻辑
```python
        ]

        messages = [
            {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 629-648: Assertions and result checks / 断言与结果检查
```python
                "role": "user",
                "content": "Test the function",
            }
        ]

        # Should raise BadRequestError due to missing required 'name' field
        with self.assertRaises(openai.BadRequestError) as context:
            self.client.chat.completions.create(
                model=self.model_name,
                messages=messages,
                max_tokens=100,
                temperature=0.1,
                tools=invalid_tools,
                tool_choice="required",
                stream=False,
            )

        # Verify the error message indicates missing name field
        error_msg = str(context.exception).lower()
        self.assertIn("name", error_msg)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `assertRaises`, `create`, `lower` and `assertIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 650-674: Test routines around test_invalid_json_schema_in_tool / 测试例程
```python
    def test_invalid_json_schema_in_tool(self):
        """Test what happens when tool function has invalid JSON schema"""
        invalid_tools = [
            {
                "type": "function",
                "function": {
                    "name": "test_function",
                    "description": "Test function with invalid JSON schema",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "invalid_field": {
                                "type": "unknown_type",  # Invalid type
                                "description": "This field has an invalid type",
                            }
                        },
                        "required": ["invalid_field"],
                    },
                },
            }
        ]

        messages = [
            {
                "role": "user",
```
**EN:** This range defines concrete test routine(s) `test_invalid_json_schema_in_tool`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 675-693: Assertions and result checks / 断言与结果检查
```python
                "content": "Test the function",
            }
        ]

        # Should raise BadRequestError due to invalid JSON schema in tool parameters
        with self.assertRaises(openai.BadRequestError) as context:
            self.client.chat.completions.create(
                model=self.model_name,
                messages=messages,
                max_tokens=100,
                temperature=0.1,
                tools=invalid_tools,
                tool_choice="required",
                stream=False,
            )

        # Verify the error message indicates invalid JSON schema for parameters field
        error_msg = str(context.exception).lower()
        self.assertIn("invalid 'parameters' schema", error_msg)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `assertRaises`, `create`, `lower` and `assertIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 695-699: Test routines around test_conflicting_defs_required_tool_choice / 测试例程
```python
    def test_conflicting_defs_required_tool_choice(self):
        """Test that conflicting $defs with required tool_choice returns 400 error"""
        conflicting_tools = [
            {
                "type": "function",
```
**EN:** This range defines concrete test routine(s) `test_conflicting_defs_required_tool_choice`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 700-719: Scenario logic / 场景逻辑
```python
                "function": {
                    "name": "tool1",
                    "description": "Tool 1 with conflicting $defs",
                    "parameters": {
                        "type": "object",
                        "properties": {
                            "data": {"$ref": "#/$defs/DataType"},
                        },
                        "required": ["data"],
                        "$defs": {
                            "DataType": {
                                "type": "object",
                                "properties": {"value": {"type": "string"}},
                                "required": ["value"],
                            },
                        },
                    },
                },
            },
            {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 720-724: Scenario logic / 场景逻辑
```python
                "type": "function",
                "function": {
                    "name": "tool2",
                    "description": "Tool 2 with conflicting $defs",
                    "parameters": {
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 725-744: Scenario logic / 场景逻辑
```python
                        "type": "object",
                        "properties": {
                            "data": {"$ref": "#/$defs/DataType"},
                        },
                        "required": ["data"],
                        "$defs": {
                            "DataType": {  # Different definition for DataType
                                "type": "object",
                                "properties": {"value": {"type": "number"}},
                                "required": ["value"],
                            },
                        },
                    },
                },
            },
        ]

        messages = [
            {
                "role": "user",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 745-749: Scenario logic / 场景逻辑
```python
                "content": "Test the conflicting tools",
            }
        ]

        # Should raise BadRequestError due to conflicting $defs
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 750-764: Assertions and result checks / 断言与结果检查
```python
        with self.assertRaises(openai.BadRequestError) as context:
            self.client.chat.completions.create(
                model=self.model_name,
                messages=messages,
                max_tokens=100,
                temperature=0.1,
                tools=conflicting_tools,
                tool_choice="required",
                stream=False,
            )

        # Verify the error message indicates conflicting tool definitions
        error_msg = str(context.exception).lower()
        self.assertIn("multiple schemas", error_msg)
        self.assertIn("not supported", error_msg)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `assertRaises`, `create`, `lower` and `assertIn`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 767-770: Class definition for TestToolChoiceQwen25 / 类定义
```python
class TestToolChoiceQwen25(TestToolChoiceLlama32):
    """Test tool_choice functionality with Qwen2.5 model"""

    @classmethod
```
**EN:** This range declares `TestToolChoiceQwen25`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 771-789: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.flaky_tests = {}

        cls.model = "Qwen/Qwen2.5-7B-Instruct"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=[
                "--tool-call-parser",
                "qwen25",
            ],
        )
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(cls.model)
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server` and `get_tokenizer`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 792-795: Class definition for TestToolChoiceMistral / 类定义
```python
class TestToolChoiceMistral(TestToolChoiceLlama32):
    """Test tool_choice functionality with Mistral model"""

    @classmethod
```
**EN:** This range declares `TestToolChoiceMistral`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 796-816: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        # Mark flaky tests for this model
        cls.flaky_tests = {
            "test_multi_tool_scenario_auto",
            "test_multi_tool_scenario_required",
        }

        cls.model = "mistralai/Mistral-7B-Instruct-v0.3"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=[
                "--tool-call-parser",
                "mistral",
            ],
        )
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 817-818: Request and response handling / 请求与响应处理
```python
        cls.base_url += "/v1"
        cls.tokenizer = get_tokenizer(cls.model)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get_tokenizer`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 819-820: Scenario logic / 场景逻辑
```python

    @unittest.skip("Fails due to whitespace issue with Mistral - skipping")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skip`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 821-823: Test routines around test_tool_choice_required_non_streaming / 测试例程
```python
    def test_tool_choice_required_non_streaming(self):
        """Test tool_choice='required' in non-streaming mode"""
        super().test_tool_choice_required_non_streaming()
```
**EN:** This range defines concrete test routine(s) `test_tool_choice_required_non_streaming`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 824-825: Scenario logic / 场景逻辑
```python

    @unittest.skip("Fails due to whitespace issue with Mistral - skipping")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skip`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 826-828: Test routines around test_multi_tool_scenario_required / 测试例程
```python
    def test_multi_tool_scenario_required(self):
        """Test multi-tool scenario with tool_choice='required'"""
        super().test_multi_tool_scenario_required()
```
**EN:** This range defines concrete test routine(s) `test_multi_tool_scenario_required`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 829-830: Scenario logic / 场景逻辑
```python

    @unittest.skip("Fails due to whitespace issue with Mistral - skipping")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skip`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 831-833: Test routines around test_complex_parameters_required_non_streaming / 测试例程
```python
    def test_complex_parameters_required_non_streaming(self):
        """Validate complex nested parameter schemas in non-streaming required mode"""
        super().test_complex_parameters_required_non_streaming()
```
**EN:** This range defines concrete test routine(s) `test_complex_parameters_required_non_streaming`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 834-864: Process control logic / 进程控制逻辑
```python


# Skip for ci test
# class TestToolChoiceGLM45(TestToolChoiceLlama32):
#     @classmethod
#     def setUpClass(cls):
#         # Replace with the model name needed for testing; if not required, reuse DEFAULT_SMALL_MODEL_NAME_FOR_TEST
#         cls.model = "THUDM/GLM-4.5"
#         cls.base_url = DEFAULT_URL_FOR_TEST
#         cls.api_key = "sk-123456"

#         # Start the local OpenAI Server. If necessary, you can add other parameters such as --enable-tools.
#         cls.process = popen_launch_server(
#             cls.model,
#             cls.base_url,
#             timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
#             api_key=cls.api_key,
#             other_args=[
#                 # If your server needs extra parameters to test function calling, please add them here.
#                 "--tool-call-parser",
#                 "glm45",
#                 "--reasoning-parser",
#                 "glm45",
#                 "--tp-size",
#                 "8"
#             ],
#         )
#         cls.base_url += "/v1"
#         cls.tokenizer = get_tokenizer(cls.model)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `TestToolChoiceGLM45`, `setUpClass`, `popen_launch_server` and `get_tokenizer`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 865-868: Class definition for TestToolChoiceLfm2 / 类定义
```python
class TestToolChoiceLfm2(TestToolChoiceLlama32):
    """Test tool_choice functionality with LiquidAI LFM2 model"""

    @classmethod
```
**EN:** This range declares `TestToolChoiceLfm2`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 869-889: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.flaky_tests = {
            "test_multi_tool_scenario_auto",
            "test_multi_tool_scenario_required",
        }

        cls.model = "LiquidAI/LFM2.5-1.2B-Instruct"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
            other_args=[
                "--tool-call-parser",
                "lfm2",
            ],
        )
        cls.base_url += "/v1"
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 890-890: Scenario logic / 场景逻辑
```python
        cls.tokenizer = get_tokenizer(cls.model)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `get_tokenizer`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 891-894: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Streaming responses / 流式响应
- Token-level inspection / Token 级分析
- Tool calling workflow / 工具调用流程

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `unittest`
- **Third-party / 第三方库**: `openai`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.test_utils`
