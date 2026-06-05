# test_separate_reasoning_execution.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/lang_frontend/test_separate_reasoning_execution.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `separate reasoning execution` scenario in `test/manual/lang_frontend`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual/lang_frontend` 中的 `separate reasoning execution` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and shared helpers / 导入与共享辅助项
```python
"""
Tests for the execution of separate_reasoning functionality in sglang.

Usage:
python3 -m unittest test/lang/test_separate_reasoning_execution.py
"""

import threading
import unittest
from unittest.mock import MagicMock, patch

from sglang.lang.interpreter import StreamExecutor
from sglang.lang.ir import SglGen, SglSeparateReasoning
from sglang.test.test_utils import CustomTestCase


# Helper function to create events that won't block program exit
```
**EN:** This range imports `threading`, `unittest`, `unittest.mock` and `sglang.lang.interpreter`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 18-20: Helper routines around create_daemon_event / 辅助例程
```python
def create_daemon_event():
    event = threading.Event()
    return event
```
**EN:** This range implements helper routine(s) `create_daemon_event` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `Event`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 23-23: Class definition for MockReasoningParser / 类定义
```python
class MockReasoningParser:
```
**EN:** This range declares `MockReasoningParser`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 24-27: Helper routines around __init__ / 辅助例程
```python
    def __init__(self, model_type):
        self.model_type = model_type
        self.parse_non_stream_called = False
        self.parse_stream_chunk_called = False
```
**EN:** This range implements helper routine(s) `__init__` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 29-34: Helper routines around parse_non_stream / 辅助例程
```python
    def parse_non_stream(self, full_text):
        self.parse_non_stream_called = True
        # Simulate parsing by adding a prefix to indicate reasoning
        reasoning = f"[REASONING from {self.model_type}]: {full_text}"
        normal_text = f"[NORMAL from {self.model_type}]: {full_text}"
        return reasoning, normal_text
```
**EN:** This range implements helper routine(s) `parse_non_stream` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 36-41: Helper routines around parse_stream_chunk / 辅助例程
```python
    def parse_stream_chunk(self, chunk_text):
        self.parse_stream_chunk_called = True
        # Simulate parsing by adding a prefix to indicate reasoning
        reasoning = f"[REASONING from {self.model_type}]: {chunk_text}"
        normal_text = f"[NORMAL from {self.model_type}]: {chunk_text}"
        return reasoning, normal_text
```
**EN:** This range implements helper routine(s) `parse_stream_chunk` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 44-44: Class definition for TestSeparateReasoningExecution / 类定义
```python
class TestSeparateReasoningExecution(CustomTestCase):
```
**EN:** This range declares `TestSeparateReasoningExecution`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 45-49: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUp(self):
        """Set up for the test."""
        super().setUp()
        # Store any events created during the test
        self.events = []
```
**EN:** This range implements lifecycle helper(s) `setUp` to prepare or clean up shared resources across the test flow.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。

### Lines 51-57: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDown(self):
        """Clean up any threads that might have been created during the test."""
        super().tearDown()

        # Set all events to ensure any waiting threads are released
        for event in self.events:
            event.set()
```
**EN:** This range implements lifecycle helper(s) `tearDown` to prepare or clean up shared resources across the test flow.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。

### Lines 59-63: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDown(self):
        super().tearDown()
        # wake up all threads
        for ev in self.events:
            ev.set()
```
**EN:** This range implements lifecycle helper(s) `tearDown` to prepare or clean up shared resources across the test flow.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。

### Lines 64-65: Scenario logic / 场景逻辑
```python

    @patch("sglang.srt.parser.reasoning_parser.ReasoningParser")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `patch`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 66-68: Test routines around test_execute_separate_reasoning / 测试例程
```python
    def test_execute_separate_reasoning(self, mock_parser_class):
        """Test that _execute_separate_reasoning correctly calls the ReasoningParser."""
        # Setup mock parser
```
**EN:** This range defines concrete test routine(s) `test_execute_separate_reasoning`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 69-90: Scenario logic / 场景逻辑
```python
        mock_parser = MockReasoningParser("deepseek-r1")
        mock_parser_class.return_value = mock_parser

        # Create a mock backend to avoid AttributeError in __del__
        mock_backend = MagicMock()

        # Create a StreamExecutor with necessary setup
        executor = StreamExecutor(
            backend=mock_backend,
            arguments={},
            default_sampling_para={},
            chat_template={
                "role_map": {"user": "user", "assistant": "assistant"}
            },  # Simple chat template
            stream=False,
            use_thread=False,
        )

        # Set up the executor with a variable and its value
        var_name = "test_var"
        reasoning_name = f"{var_name}_reasoning_content"
        var_value = "Test content"
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `MockReasoningParser`, `MagicMock` and `StreamExecutor`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 91-93: Scenario logic / 场景逻辑
```python
        executor.variables = {var_name: var_value}

        # Create events and track them for cleanup
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 94-115: Assertions and result checks / 断言与结果检查
```python
        var_event = create_daemon_event()
        reasoning_event = create_daemon_event()
        self.events.extend([var_event, reasoning_event])

        executor.variable_event = {var_name: var_event, reasoning_name: reasoning_event}
        executor.variable_event[var_name].set()  # Mark as ready

        # Set up the current role
        executor.cur_role = "assistant"
        executor.cur_role_begin_pos = 0
        executor.text_ = var_value

        # Create a gen expression and a separate_reasoning expression
        gen_expr = SglGen(var_name)
        expr = SglSeparateReasoning("deepseek-r1", expr=gen_expr)

        # Execute separate_reasoning
        executor._execute_separate_reasoning(expr)

        # Verify that the parser was created with the correct model type
        mock_parser_class.assert_called_once_with("deepseek-r1")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `create_daemon_event`, `extend`, `SglGen` and `SglSeparateReasoning`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 116-118: Assertions and result checks / 断言与结果检查
```python
        # Verify that parse_non_stream was called
        self.assertTrue(mock_parser.parse_non_stream_called)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 119-135: Assertions and result checks / 断言与结果检查
```python
        # Verify that the variables were updated correctly
        reasoning_name = f"{var_name}_reasoning_content"
        self.assertIn(reasoning_name, executor.variables)
        self.assertEqual(
            executor.variables[reasoning_name],
            f"[REASONING from deepseek-r1]: {var_value}",
        )
        self.assertEqual(
            executor.variables[var_name], f"[NORMAL from deepseek-r1]: {var_value}"
        )

        # Verify that the variable event was set
        self.assertIn(reasoning_name, executor.variable_event)
        self.assertTrue(executor.variable_event[reasoning_name].is_set())

        # Verify that the text was updated
        self.assertEqual(executor.text_, f"[NORMAL from deepseek-r1]: {var_value}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertIn`, `assertEqual`, `assertTrue` and `is_set`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 136-137: Scenario logic / 场景逻辑
```python

    @patch("sglang.srt.parser.reasoning_parser.ReasoningParser")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `patch`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 138-143: Test routines around test_reasoning_parser_integration / 测试例程
```python
    def test_reasoning_parser_integration(self, mock_parser_class):
        """Test the integration between separate_reasoning and ReasoningParser."""
        # Setup mock parsers for different model types
        deepseek_parser = MockReasoningParser("deepseek-r1")
        qwen_parser = MockReasoningParser("qwen3")
```
**EN:** This range defines concrete test routine(s) `test_reasoning_parser_integration`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `MockReasoningParser`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 144-162: Helper routines around get_parser / 辅助例程
```python
        # Configure the mock to return different parsers based on model type
        def get_parser(model_type):
            if model_type == "deepseek-r1":
                return deepseek_parser
            elif model_type == "qwen3":
                return qwen_parser
            else:
                raise ValueError(f"Unsupported model type: {model_type}")

        mock_parser_class.side_effect = get_parser

        # Test with DeepSeek-R1 model
        test_text = "This is a test"
        reasoning, normal_text = deepseek_parser.parse_non_stream(test_text)

        self.assertEqual(reasoning, f"[REASONING from deepseek-r1]: {test_text}")
        self.assertEqual(normal_text, f"[NORMAL from deepseek-r1]: {test_text}")

        # Test with Qwen3 model
```
**EN:** This range implements helper routine(s) `get_parser` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `ValueError`, `parse_non_stream` and `assertEqual`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 163-166: Assertions and result checks / 断言与结果检查
```python
        reasoning, normal_text = qwen_parser.parse_non_stream(test_text)

        self.assertEqual(reasoning, f"[REASONING from qwen3]: {test_text}")
        self.assertEqual(normal_text, f"[NORMAL from qwen3]: {test_text}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `parse_non_stream` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 167-168: Scenario logic / 场景逻辑
```python

    @patch("sglang.srt.parser.reasoning_parser.ReasoningParser")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `patch`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 169-189: Test routines around test_reasoning_parser_invalid_model / 测试例程
```python
    def test_reasoning_parser_invalid_model(self, mock_parser_class):
        """Test that ReasoningParser raises an error for invalid model types."""

        # Configure the mock to raise an error for invalid model types
        def get_parser(model_type):
            if model_type in ["deepseek-r1", "qwen3"]:
                return MockReasoningParser(model_type)
            elif model_type is None:
                raise ValueError("Model type must be specified")
            else:
                raise ValueError(f"Unsupported model type: {model_type}")

        mock_parser_class.side_effect = get_parser

        with self.assertRaises(ValueError) as context:
            mock_parser_class("invalid-model")
        self.assertIn("Unsupported model type", str(context.exception))

        with self.assertRaises(ValueError) as context:
            mock_parser_class(None)
        self.assertIn("Model type must be specified", str(context.exception))
```
**EN:** This range defines concrete test routine(s) `test_reasoning_parser_invalid_model` and `get_parser`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `MockReasoningParser`, `ValueError`, `assertRaises` and `mock_parser_class`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 190-193: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Streaming responses / 流式响应
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `threading`, `unittest`, `unittest.mock`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.lang.interpreter`, `sglang.lang.ir`, `sglang.test.test_utils`
