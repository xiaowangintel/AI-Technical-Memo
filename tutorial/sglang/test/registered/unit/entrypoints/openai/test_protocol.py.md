# test_protocol.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/entrypoints/openai/test_protocol.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates protocol behavior in SGLang's unit / entrypoints / openai area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 入口 / OpenAI 领域中与 protocol 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: supporting source context / 辅助源码上下文
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 14-14: supporting statements / 辅助语句
```python
"""Tests for OpenAI API protocol models"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 16-33: module imports and dependencies / 模块导入与依赖
```python
import unittest
from typing import List, Optional

from pydantic import BaseModel, Field, ValidationError

from sglang.srt.entrypoints.openai.protocol import (
    ChatCompletionRequest,
    ChatCompletionResponse,
    ChatCompletionResponseChoice,
    ChatMessage,
    CompletionRequest,
    Function,
    ModelCard,
    ModelList,
    Tool,
    UsageInfo,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `typing`, `pydantic`, `sglang.srt.entrypoints.openai.protocol`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `typing`, `pydantic`, `sglang.srt.entrypoints.openai.protocol`。

### Lines 35-35: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 38-38: class TestModelCard declaration / 类 TestModelCard 声明
```python
class TestModelCard(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 39-39: supporting statements / 辅助语句
```python
    """Test ModelCard protocol model"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 41-47: test case model card serialization / 测试用例 model card serialization
```python
    def test_model_card_serialization(self):
        """Test model card JSON serialization"""
        card = ModelCard(id="test-model", max_model_len=4096)
        data = card.model_dump()
        self.assertEqual(data["id"], "test-model")
        self.assertEqual(data["object"], "model")
        self.assertEqual(data["max_model_len"], 4096)
```
**EN:** Test model card JSON serialization This test exercises `test_model_card_serialization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test model card JSON serialization 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_model_card_serialization`。

### Lines 50-50: class TestModelList declaration / 类 TestModelList 声明
```python
class TestModelList(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 51-51: supporting statements / 辅助语句
```python
    """Test ModelList protocol model"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 53-57: test case empty model list / 测试用例 empty model list
```python
    def test_empty_model_list(self):
        """Test empty model list creation"""
        model_list = ModelList()
        self.assertEqual(model_list.object, "list")
        self.assertEqual(len(model_list.data), 0)
```
**EN:** Test empty model list creation This test exercises `test_empty_model_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test empty model list creation 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_model_list`。

### Lines 59-68: test case model list with cards / 测试用例 model list with cards
```python
    def test_model_list_with_cards(self):
        """Test model list with model cards"""
        cards = [
            ModelCard(id="model-1"),
            ModelCard(id="model-2", max_model_len=2048),
        ]
        model_list = ModelList(data=cards)
        self.assertEqual(len(model_list.data), 2)
        self.assertEqual(model_list.data[0].id, "model-1")
        self.assertEqual(model_list.data[1].id, "model-2")
```
**EN:** Test model list with model cards This test exercises `test_model_list_with_cards` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test model list with model cards 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_model_list_with_cards`。

### Lines 71-71: class TestCompletionRequest declaration / 类 TestCompletionRequest 声明
```python
class TestCompletionRequest(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 72-72: supporting statements / 辅助语句
```python
    """Test CompletionRequest protocol model"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 74-83: test case basic completion request / 测试用例 basic completion request
```python
    def test_basic_completion_request(self):
        """Test basic completion request"""
        request = CompletionRequest(model="test-model", prompt="Hello world")
        self.assertEqual(request.model, "test-model")
        self.assertEqual(request.prompt, "Hello world")
        self.assertEqual(request.max_tokens, 16)  # default
        self.assertEqual(request.temperature, 1.0)  # default
        self.assertEqual(request.n, 1)  # default
        self.assertFalse(request.stream)  # default
        self.assertFalse(request.echo)  # default
```
**EN:** Test basic completion request This test exercises `test_basic_completion_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic completion request 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_completion_request`。

### Lines 85-102: test case completion request sglang extensions / 测试用例 completion request sglang extensions
```python
    def test_completion_request_sglang_extensions(self):
        """Test completion request with SGLang-specific extensions"""
        request = CompletionRequest(
            model="test-model",
            prompt="Hello",
            top_k=50,
            min_p=0.1,
            repetition_penalty=1.1,
            regex=r"\d+",
            json_schema='{"type": "object"}',
            lora_path="/path/to/lora",
        )
        self.assertEqual(request.top_k, 50)
        self.assertEqual(request.min_p, 0.1)
        self.assertEqual(request.repetition_penalty, 1.1)
        self.assertEqual(request.regex, r"\d+")
        self.assertEqual(request.json_schema, '{"type": "object"}')
        self.assertEqual(request.lora_path, "/path/to/lora")
```
**EN:** Test completion request with SGLang-specific extensions This test exercises `test_completion_request_sglang_extensions` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test completion request with SGLang-specific extensions 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion_request_sglang_extensions`。

### Lines 104-110: test case completion request validation errors / 测试用例 completion request validation errors
```python
    def test_completion_request_validation_errors(self):
        """Test completion request validation errors"""
        with self.assertRaises(ValidationError):
            CompletionRequest()  # missing required fields

        with self.assertRaises(ValidationError):
            CompletionRequest(model="test-model")  # missing prompt
```
**EN:** Test completion request validation errors This test exercises `test_completion_request_validation_errors` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test completion request validation errors 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion_request_validation_errors`。

### Lines 113-113: class TestChatCompletionRequest declaration / 类 TestChatCompletionRequest 声明
```python
class TestChatCompletionRequest(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 114-114: supporting statements / 辅助语句
```python
    """Test ChatCompletionRequest protocol model"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 116-126: test case basic chat completion request / 测试用例 basic chat completion request
```python
    def test_basic_chat_completion_request(self):
        """Test basic chat completion request"""
        messages = [{"role": "user", "content": "Hello"}]
        request = ChatCompletionRequest(model="test-model", messages=messages)
        self.assertEqual(request.model, "test-model")
        self.assertEqual(len(request.messages), 1)
        self.assertEqual(request.messages[0].role, "user")
        self.assertEqual(request.messages[0].content, "Hello")
        self.assertEqual(request.temperature, None)  # default
        self.assertFalse(request.stream)  # default
        self.assertEqual(request.tool_choice, "none")  # default when no tools
```
**EN:** Test basic chat completion request This test exercises `test_basic_chat_completion_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic chat completion request 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_chat_completion_request`。

### Lines 128-142: test case sampling param build / 测试用例 sampling param build
```python
    def test_sampling_param_build(self):
        req = ChatCompletionRequest(
            model="x",
            messages=[{"role": "user", "content": "Hi"}],
            temperature=0.8,
            max_tokens=150,
            min_tokens=5,
            top_p=0.9,
            stop=["</s>"],
        )
        params = req.to_sampling_params(["</s>"], {}, None)
        self.assertEqual(params["temperature"], 0.8)
        self.assertEqual(params["max_new_tokens"], 150)
        self.assertEqual(params["min_new_tokens"], 5)
        self.assertEqual(params["stop"], ["</s>"])
```
**EN:** This test exercises `test_sampling_param_build` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_sampling_param_build`。

### Lines 144-162: test case chat completion tool choice validation / 测试用例 chat completion tool choice validation
```python
    def test_chat_completion_tool_choice_validation(self):
        """Test tool choice validation logic"""
        messages = [{"role": "user", "content": "Hello"}]

        # No tools, tool_choice should default to "none"
        request1 = ChatCompletionRequest(model="test-model", messages=messages)
        self.assertEqual(request1.tool_choice, "none")

        # With tools, tool_choice should default to "auto"
        tools = [
            {
                "type": "function",
                "function": {"name": "test_func", "description": "Test function"},
            }
        ]
        request2 = ChatCompletionRequest(
            model="test-model", messages=messages, tools=tools
        )
        self.assertEqual(request2.tool_choice, "auto")
```
**EN:** Test tool choice validation logic This test exercises `test_chat_completion_tool_choice_validation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test tool choice validation logic 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_tool_choice_validation`。

### Lines 164-180: test case chat completion sglang extensions / 测试用例 chat completion sglang extensions
```python
    def test_chat_completion_sglang_extensions(self):
        """Test chat completion with SGLang extensions"""
        messages = [{"role": "user", "content": "Hello"}]
        request = ChatCompletionRequest(
            model="test-model",
            messages=messages,
            top_k=40,
            min_p=0.05,
            separate_reasoning=False,
            stream_reasoning=False,
            chat_template_kwargs={"custom_param": "value"},
        )
        self.assertEqual(request.top_k, 40)
        self.assertEqual(request.min_p, 0.05)
        self.assertFalse(request.separate_reasoning)
        self.assertFalse(request.stream_reasoning)
        self.assertEqual(request.chat_template_kwargs, {"custom_param": "value"})
```
**EN:** Test chat completion with SGLang extensions This test exercises `test_chat_completion_sglang_extensions` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test chat completion with SGLang extensions 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_sglang_extensions`。

### Lines 182-197: test case chat completion reasoning effort / 测试用例 chat completion reasoning effort
```python
    def test_chat_completion_reasoning_effort(self):
        """Test chat completion with reasoning effort"""
        messages = [{"role": "user", "content": "Hello"}]
        request = ChatCompletionRequest(
            model="test-model",
            messages=messages,
            reasoning={
                "enabled": True,
                "reasoning_effort": "high",
            },
        )
        self.assertEqual(request.reasoning_effort, "high")
        self.assertEqual(
            request.chat_template_kwargs,
            {"thinking": True, "enable_thinking": True},
        )
```
**EN:** Test chat completion with reasoning effort This test exercises `test_chat_completion_reasoning_effort` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test chat completion with reasoning effort 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_reasoning_effort`。

### Lines 199-209: test case chat completion reasoning effort none / 测试用例 chat completion reasoning effort none
```python
    def test_chat_completion_reasoning_effort_none(self):
        """Test reasoning_effort='none' disables thinking"""
        messages = [{"role": "user", "content": "Hello"}]
        request = ChatCompletionRequest(
            model="test-model",
            messages=messages,
            reasoning_effort="none",
        )
        self.assertEqual(request.reasoning_effort, "none")
        self.assertFalse(request.chat_template_kwargs.get("thinking"))
        self.assertFalse(request.chat_template_kwargs.get("enable_thinking"))
```
**EN:** Test reasoning_effort='none' disables thinking This test exercises `test_chat_completion_reasoning_effort_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test reasoning_effort='none' disables thinking 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_reasoning_effort_none`。

### Lines 211-221: test case chat completion reasoning effort none from reasoning dict / 测试用例 chat completion reasoning effort none from reasoning dict
```python
    def test_chat_completion_reasoning_effort_none_from_reasoning_dict(self):
        """Test reasoning_effort='none' via nested reasoning dict"""
        messages = [{"role": "user", "content": "Hello"}]
        request = ChatCompletionRequest(
            model="test-model",
            messages=messages,
            reasoning={"effort": "none"},
        )
        self.assertEqual(request.reasoning_effort, "none")
        self.assertFalse(request.chat_template_kwargs.get("thinking"))
        self.assertFalse(request.chat_template_kwargs.get("enable_thinking"))
```
**EN:** Test reasoning_effort='none' via nested reasoning dict This test exercises `test_chat_completion_reasoning_effort_none_from_reasoning_dict` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test reasoning_effort='none' via nested reasoning dict 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_reasoning_effort_none_from_reasoning_dict`。

### Lines 223-252: test case chat completion reasoning effort max / 测试用例 chat completion reasoning effort max
```python
    def test_chat_completion_reasoning_effort_max(self):
        """`max` is an sglang extension on chat completion's top-level
        `reasoning_effort` only; the Responses-API-style nested
        `reasoning.effort` path stays aligned with OpenAI's three levels."""
        from pydantic import ValidationError

        messages = [{"role": "user", "content": "Hello"}]
        request = ChatCompletionRequest(
            model="test-model",
            messages=messages,
            reasoning_effort="max",
        )
        self.assertEqual(request.reasoning_effort, "max")

        # Unknown values still rejected.
        with self.assertRaises(ValidationError):
            ChatCompletionRequest(
                model="test-model",
                messages=messages,
                reasoning_effort="ultra",
            )

        # Nested reasoning.effort=max is NOT promoted by normalize_reasoning_inputs:
        # the Responses API path keeps the OpenAI low/medium/high contract.
        request = ChatCompletionRequest(
            model="test-model",
            messages=messages,
            reasoning={"effort": "max"},
        )
        self.assertNotEqual(request.reasoning_effort, "max")
```
**EN:** `max` is an sglang extension on chat completion's top-level `reasoning_effort` only; the Responses-API-style nested `reasoning.effort` path stays aligned with OpenAI's three levels. This test exercises `test_chat_completion_reasoning_effort_max` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** `max` is an sglang extension on chat completion's top-level `reasoning_effort` only; the Responses-API-style nested `reasoning.effort` path stays aligned with OpenAI's three levels. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_reasoning_effort_max`。

### Lines 254-327: test case chat completion json format / 测试用例 chat completion json format
```python
    def test_chat_completion_json_format(self):
        """Test chat completion json format"""
        transcript = "Good morning! It's 7:00 AM, and I'm just waking up. Today is going to be a busy day, "
        "so let's get started. First, I need to make a quick breakfast. I think I'll have some "
        "scrambled eggs and toast with a cup of coffee. While I'm cooking, I'll also check my "
        "emails to see if there's anything urgent."

        messages = [
            {
                "role": "system",
                "content": "The following is a voice message transcript. Only answer in JSON.",
            },
            {
                "role": "user",
                "content": transcript,
            },
        ]

        class VoiceNote(BaseModel):
            title: str = Field(description="A title for the voice note")
            summary: str = Field(
                description="A short one sentence summary of the voice note."
            )
            strict: Optional[bool] = True
            actionItems: List[str] = Field(
                description="A list of action items from the voice note"
            )

        request = ChatCompletionRequest(
            model="test-model",
            messages=messages,
            top_k=40,
            min_p=0.05,
            separate_reasoning=False,
            stream_reasoning=False,
            chat_template_kwargs={"custom_param": "value"},
            response_format={
                "type": "json_schema",
                "schema": VoiceNote.model_json_schema(),
            },
        )
        res_format = request.response_format
        json_format = res_format.json_schema
        name = json_format.name
        schema = json_format.schema_
        strict = json_format.strict
        self.assertEqual(name, "VoiceNote")
        self.assertEqual(strict, True)
        self.assertNotIn("strict", schema["properties"])

        request = ChatCompletionRequest(
            model="test-model",
            messages=messages,
            top_k=40,
            min_p=0.05,
            separate_reasoning=False,
            stream_reasoning=False,
            chat_template_kwargs={"custom_param": "value"},
            response_format={
                "type": "json_schema",
                "json_schema": {
                    "name": "VoiceNote",
                    "schema": VoiceNote.model_json_schema(),
                    "strict": True,
                },
            },
        )
        res_format = request.response_format
        json_format = res_format.json_schema
        name = json_format.name
        schema = json_format.schema_
        strict = json_format.strict
        self.assertEqual(name, "VoiceNote")
        self.assertEqual(strict, True)
```
**EN:** Test chat completion json format This test exercises `test_chat_completion_json_format` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test chat completion json format 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chat_completion_json_format`。

### Lines 330-330: class TestModelSerialization declaration / 类 TestModelSerialization 声明
```python
class TestModelSerialization(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 331-331: supporting statements / 辅助语句
```python
    """Test model serialization with hidden states"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 333-351: test case hidden states excluded when none / 测试用例 hidden states excluded when none
```python
    def test_hidden_states_excluded_when_none(self):
        """Test that None hidden_states are excluded with exclude_none=True"""
        choice = ChatCompletionResponseChoice(
            index=0,
            message=ChatMessage(role="assistant", content="Hello"),
            finish_reason="stop",
            hidden_states=None,
        )

        response = ChatCompletionResponse(
            id="test-id",
            model="test-model",
            choices=[choice],
            usage=UsageInfo(prompt_tokens=5, completion_tokens=1, total_tokens=6),
        )

        # Test exclude_none serialization (should exclude None hidden_states)
        data = response.model_dump(exclude_none=True)
        self.assertNotIn("hidden_states", data["choices"][0])
```
**EN:** Test that None hidden_states are excluded with exclude_none=True This test exercises `test_hidden_states_excluded_when_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that None hidden_states are excluded with exclude_none=True 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hidden_states_excluded_when_none`。

### Lines 353-372: test case hidden states included when not none / 测试用例 hidden states included when not none
```python
    def test_hidden_states_included_when_not_none(self):
        """Test that non-None hidden_states are included"""
        choice = ChatCompletionResponseChoice(
            index=0,
            message=ChatMessage(role="assistant", content="Hello"),
            finish_reason="stop",
            hidden_states=[0.1, 0.2, 0.3],
        )

        response = ChatCompletionResponse(
            id="test-id",
            model="test-model",
            choices=[choice],
            usage=UsageInfo(prompt_tokens=5, completion_tokens=1, total_tokens=6),
        )

        # Test exclude_none serialization (should include non-None hidden_states)
        data = response.model_dump(exclude_none=True)
        self.assertIn("hidden_states", data["choices"][0])
        self.assertEqual(data["choices"][0]["hidden_states"], [0.1, 0.2, 0.3])
```
**EN:** Test that non-None hidden_states are included This test exercises `test_hidden_states_included_when_not_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that non-None hidden_states are included 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_hidden_states_included_when_not_none`。

### Lines 375-375: class TestFunctionDeferLoading declaration / 类 TestFunctionDeferLoading 声明
```python
class TestFunctionDeferLoading(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 376-376: supporting statements / 辅助语句
```python
    """Test defer_loading field behavior on Function/Tool."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 378-385: test case function defaults preserve strict / 测试用例 function defaults preserve strict
```python
    def test_function_defaults_preserve_strict(self):
        """strict must default to False and be present in dumps so downstream
        code (function_call_parser, chat templates) sees the expected shape."""
        f = Function(name="foo")
        data = f.model_dump()
        self.assertEqual(data["name"], "foo")
        self.assertEqual(data["strict"], False)
        self.assertNotIn("defer_loading", data)
```
**EN:** strict must default to False and be present in dumps so downstream code (function_call_parser, chat templates) sees the expected shape. This test exercises `test_function_defaults_preserve_strict` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** strict must default to False and be present in dumps so downstream code (function_call_parser, chat templates) sees the expected shape. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_function_defaults_preserve_strict`。

### Lines 387-391: test case function defer loading true serialized / 测试用例 function defer loading true serialized
```python
    def test_function_defer_loading_true_serialized(self):
        f = Function(name="foo", defer_loading=True)
        data = f.model_dump()
        self.assertTrue(data["defer_loading"])
        self.assertEqual(data["strict"], False)
```
**EN:** This test exercises `test_function_defer_loading_true_serialized` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_function_defer_loading_true_serialized`。

### Lines 393-398: test case function defer loading false serialized / 测试用例 function defer loading false serialized
```python
    def test_function_defer_loading_false_serialized(self):
        """defer_loading=False is an explicit value and must be preserved."""
        f = Function(name="foo", defer_loading=False)
        data = f.model_dump()
        self.assertIn("defer_loading", data)
        self.assertFalse(data["defer_loading"])
```
**EN:** defer_loading=False is an explicit value and must be preserved. This test exercises `test_function_defer_loading_false_serialized` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** defer_loading=False is an explicit value and must be preserved. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_function_defer_loading_false_serialized`。

### Lines 400-409: test case tool level defer loading propagates to function / 测试用例 tool level defer loading propagates to function
```python
    def test_tool_level_defer_loading_propagates_to_function(self):
        """defer_loading at the Tool level should propagate to Function."""
        tool = Tool(
            type="function",
            defer_loading=True,
            function={"name": "search_db"},
        )
        self.assertTrue(tool.function.defer_loading)
        data = tool.model_dump()
        self.assertTrue(data["function"]["defer_loading"])
```
**EN:** defer_loading at the Tool level should propagate to Function. This test exercises `test_tool_level_defer_loading_propagates_to_function` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** defer_loading at the Tool level should propagate to Function. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_level_defer_loading_propagates_to_function`。

### Lines 411-418: test case function level defer loading wins over tool level / 测试用例 function level defer loading wins over tool level
```python
    def test_function_level_defer_loading_wins_over_tool_level(self):
        """Explicit function-level value is preserved when both set."""
        tool = Tool(
            type="function",
            defer_loading=True,
            function={"name": "search_db", "defer_loading": False},
        )
        self.assertFalse(tool.function.defer_loading)
```
**EN:** Explicit function-level value is preserved when both set. This test exercises `test_function_level_defer_loading_wins_over_tool_level` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Explicit function-level value is preserved when both set. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_function_level_defer_loading_wins_over_tool_level`。

### Lines 420-438: test case tool reference content part accepted / 测试用例 tool reference content part accepted
```python
    def test_tool_reference_content_part_accepted(self):
        """Chat completion should accept tool_reference content on tool-role
        messages (GLM-specific extension consumed by the chat template)."""
        messages = [
            {
                "role": "tool",
                "tool_call_id": "call_1",
                "content": [
                    {"type": "tool_reference", "name": "search_db"},
                    {"type": "text", "text": "ok"},
                ],
            },
        ]
        request = ChatCompletionRequest(model="test-model", messages=messages)
        parts = request.messages[0].content
        self.assertEqual(len(parts), 2)
        self.assertEqual(parts[0].type, "tool_reference")
        self.assertEqual(parts[0].name, "search_db")
        self.assertEqual(parts[1].type, "text")
```
**EN:** Chat completion should accept tool_reference content on tool-role messages (GLM-specific extension consumed by the chat template). This test exercises `test_tool_reference_content_part_accepted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Chat completion should accept tool_reference content on tool-role messages (GLM-specific extension consumed by the chat template). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_reference_content_part_accepted`。

### Lines 441-441: class TestValidationEdgeCases declaration / 类 TestValidationEdgeCases 声明
```python
class TestValidationEdgeCases(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 442-442: supporting statements / 辅助语句
```python
    """Test edge cases and validation scenarios"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 444-450: test case invalid tool choice type / 测试用例 invalid tool choice type
```python
    def test_invalid_tool_choice_type(self):
        """Test invalid tool choice type"""
        messages = [{"role": "user", "content": "Hello"}]
        with self.assertRaises(ValidationError):
            ChatCompletionRequest(
                model="test-model", messages=messages, tool_choice=123
            )
```
**EN:** Test invalid tool choice type This test exercises `test_invalid_tool_choice_type` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test invalid tool choice type 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_tool_choice_type`。

### Lines 452-455: test case negative token limits / 测试用例 negative token limits
```python
    def test_negative_token_limits(self):
        """Test negative token limits"""
        with self.assertRaises(ValidationError):
            CompletionRequest(model="test-model", prompt="Hello", max_tokens=-1)
```
**EN:** Test negative token limits This test exercises `test_negative_token_limits` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test negative token limits 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_token_limits`。

### Lines 457-475: test case model serialization roundtrip / 测试用例 model serialization roundtrip
```python
    def test_model_serialization_roundtrip(self):
        """Test that models can be serialized and deserialized"""
        original_request = ChatCompletionRequest(
            model="test-model",
            messages=[{"role": "user", "content": "Hello"}],
            temperature=0.7,
            max_tokens=100,
        )

        # Serialize to dict
        data = original_request.model_dump()

        # Deserialize back
        restored_request = ChatCompletionRequest(**data)

        self.assertEqual(restored_request.model, original_request.model)
        self.assertEqual(restored_request.temperature, original_request.temperature)
        self.assertEqual(restored_request.max_tokens, original_request.max_tokens)
        self.assertEqual(len(restored_request.messages), len(original_request.messages))
```
**EN:** Test that models can be serialized and deserialized This test exercises `test_model_serialization_roundtrip` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that models can be serialized and deserialized 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_model_serialization_roundtrip`。

### Lines 478-479: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestModelCard`: Test ModelCard protocol model / 用于组织相关测试、夹具或辅助方法。
- `TestModelList`: Test ModelList protocol model / 用于组织相关测试、夹具或辅助方法。
- `TestCompletionRequest`: Test CompletionRequest protocol model / 用于组织相关测试、夹具或辅助方法。
- `TestChatCompletionRequest`: Test ChatCompletionRequest protocol model / 用于组织相关测试、夹具或辅助方法。
- `TestModelSerialization`: Test model serialization with hidden states / 用于组织相关测试、夹具或辅助方法。
- `TestFunctionDeferLoading`: Test defer_loading field behavior on Function/Tool. / 用于组织相关测试、夹具或辅助方法。
- `TestValidationEdgeCases`: Test edge cases and validation scenarios / 用于组织相关测试、夹具或辅助方法。
- `TestModelCard.test_model_card_serialization`: Test model card JSON serialization / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_model_card_serialization`。
- `TestModelList.test_empty_model_list`: Test empty model list creation / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_model_list`。
- `TestModelList.test_model_list_with_cards`: Test model list with model cards / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_model_list_with_cards`。
- `TestCompletionRequest.test_basic_completion_request`: Test basic completion request / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_completion_request`。
- `TestCompletionRequest.test_completion_request_sglang_extensions`: Test completion request with SGLang-specific extensions / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_completion_request_sglang_extensions`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `typing`
- **Third-party modules / 第三方模块**: `pydantic`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 479
