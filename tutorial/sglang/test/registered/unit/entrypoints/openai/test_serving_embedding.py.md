# test_serving_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/entrypoints/openai/test_serving_embedding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates serving embedding behavior in SGLang's unit / entrypoints / openai area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 入口 / OpenAI 领域中与 serving embedding 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: supporting statements / 辅助语句
```python
"""
Unit tests for the OpenAIServingEmbedding class from serving_embedding.py.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 5-14: module imports and dependencies / 模块导入与依赖
```python
import importlib
import importlib.abc
import importlib.machinery
import sys
import types
import unittest
import uuid
from unittest.mock import MagicMock, Mock

import jinja2
```
**EN:** This block imports the modules needed by the rest of the file, including `importlib`, `importlib.abc`, `importlib.machinery`, `sys`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `importlib`, `importlib.abc`, `importlib.machinery`, `sys`。

### Lines 15-18: supporting source context / 辅助源码上下文
```python


# Stub out sgl_kernel (and all submodules) before any sglang import so
# the test runs on CPU-only runners without the real CUDA library.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 19-19: class _SglKernelMockLoader declaration / 类 _SglKernelMockLoader 声明
```python
class _SglKernelMockLoader(importlib.abc.Loader):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `importlib.abc.Loader`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `importlib.abc.Loader`。

### Lines 20-26: helper routine create module / 辅助流程 create module
```python
    def create_module(self, spec):
        mod = types.ModuleType(spec.name)
        mod.__path__ = []
        mod.__package__ = spec.name
        mod.__loader__ = self
        mod.__getattr__ = lambda name: MagicMock()
        return mod
```
**EN:** This helper encapsulates `create_module` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `create_module`，以便周围测试复用准备、执行或校验逻辑。

### Lines 28-29: method exec module / 方法 exec module
```python
    def exec_module(self, module):
        pass
```
**EN:** This block implements `exec_module` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `exec_module`，承担模块行为中的一个聚焦逻辑片段。

### Lines 32-32: class _SglKernelMockFinder declaration / 类 _SglKernelMockFinder 声明
```python
class _SglKernelMockFinder(importlib.abc.MetaPathFinder):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `importlib.abc.MetaPathFinder`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `importlib.abc.MetaPathFinder`。

### Lines 33-36: class-level constants and configuration for `_SglKernelMockFinder` / 类级常量与配置
```python
    """Import hook that intercepts all sgl_kernel.* imports and returns mocks."""

    _PREFIX = "sgl_kernel"
    _loader = _SglKernelMockLoader()
```
**EN:** This block defines shared names such as `_PREFIX`, `_loader`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_PREFIX`, `_loader` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 38-43: method find spec / 方法 find spec
```python
    def find_spec(self, fullname, path, target=None):
        if fullname == self._PREFIX or fullname.startswith(self._PREFIX + "."):
            return importlib.machinery.ModuleSpec(
                fullname, self._loader, is_package=True
            )
        return None
```
**EN:** This block implements `find_spec` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `find_spec`，承担模块行为中的一个聚焦逻辑片段。

### Lines 46-47: supporting statements / 辅助语句
```python
if "sgl_kernel" not in sys.modules:
    sys.meta_path.insert(0, _SglKernelMockFinder())
```
**EN:** This block performs supporting work through calls such as `insert`, `_SglKernelMockFinder`, preparing state for nearby definitions.
**CN:** 该代码块通过 `insert`, `_SglKernelMockFinder` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 49-57: module imports and dependencies / 模块导入与依赖
```python
from fastapi import Request

from sglang.srt.entrypoints.openai.protocol import (
    EmbeddingRequest,
    MultimodalEmbeddingInput,
)
from sglang.srt.entrypoints.openai.serving_embedding import OpenAIServingEmbedding
from sglang.srt.managers.io_struct import EmbeddingReqInput
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `fastapi`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.entrypoints.openai.serving_embedding`, `sglang.srt.managers.io_struct`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `fastapi`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.entrypoints.openai.serving_embedding`, `sglang.srt.managers.io_struct`。

### Lines 59-59: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 60-62: supporting source context / 辅助源码上下文
```python


# Mock TokenizerManager for embedding tests
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 63-63: class _MockTokenizerManager declaration / 类 _MockTokenizerManager 声明
```python
class _MockTokenizerManager:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 64-88: method init / 方法 init
```python
    def __init__(self):
        self.model_config = Mock()
        self.model_config.is_multimodal = False
        self.server_args = Mock()
        self.server_args.enable_cache_report = False
        self.model_path = "test-model"

        # Mock tokenizer
        self.tokenizer = Mock()
        self.tokenizer.encode = Mock(return_value=[1, 2, 3, 4, 5])
        self.tokenizer.decode = Mock(return_value="Test embedding input")
        self.tokenizer.chat_template = None
        self.tokenizer.bos_token_id = 1

        # Mock generate_request method for embeddings
        async def mock_generate_embedding():
            yield {
                "embedding": [0.1, 0.2, 0.3, 0.4, 0.5] * 20,  # 100-dim embedding
                "meta_info": {
                    "id": f"embd-{uuid.uuid4()}",
                    "prompt_tokens": 5,
                },
            }

        self.generate_request = Mock(return_value=mock_generate_embedding())
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 89-91: supporting source context / 辅助源码上下文
```python


# Mock TemplateManager for embedding tests
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 92-92: class _MockTemplateManager declaration / 类 _MockTemplateManager 声明
```python
class _MockTemplateManager:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 93-96: method init / 方法 init
```python
    def __init__(self):
        self.chat_template_name = None  # None for embeddings usually
        self.jinja_template_content_format = "openai"
        self.completion_template_name = None
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 99-99: class ServingEmbeddingTestCase declaration / 类 ServingEmbeddingTestCase 声明
```python
class ServingEmbeddingTestCase(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 100-149: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Set up test fixtures."""
        self.tokenizer_manager = _MockTokenizerManager()
        self.template_manager = _MockTemplateManager()
        self.serving_embedding = OpenAIServingEmbedding(
            self.tokenizer_manager, self.template_manager
        )

        self.request = Mock(spec=Request)
        self.request.headers = {}

        self.basic_req = EmbeddingRequest(
            model="test-model",
            input="Hello, how are you?",
            encoding_format="float",
        )
        self.list_req = EmbeddingRequest(
            model="test-model",
            input=["Hello, how are you?", "I am fine, thank you!"],
            encoding_format="float",
        )
        self.multimodal_req = EmbeddingRequest(
            model="test-model",
            input=[
                MultimodalEmbeddingInput(text="Hello", image="base64_image_data"),
                MultimodalEmbeddingInput(text="World", image=None),
            ],
            encoding_format="float",
        )
        self.image_only_multimodal_req = EmbeddingRequest(
            model="test-model",
            input=[
                MultimodalEmbeddingInput(text=None, image="base64_image_data"),
            ],
            encoding_format="float",
        )
        self.video_multimodal_req = EmbeddingRequest(
            model="test-model",
            input=[
                MultimodalEmbeddingInput(
                    text="Describe", image=None, video="base64_video_data"
                ),
            ],
            encoding_format="float",
        )
        self.token_ids_req = EmbeddingRequest(
            model="test-model",
            input=[1, 2, 3, 4, 5],
            encoding_format="float",
        )
```
**EN:** Set up test fixtures. This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Set up test fixtures. 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 151-160: test case convert single string request / 测试用例 convert single string request
```python
    def test_convert_single_string_request(self):
        """Test converting single string request to internal format."""
        adapted_request, processed_request = (
            self.serving_embedding._convert_to_internal_request(self.basic_req)
        )

        self.assertIsInstance(adapted_request, EmbeddingReqInput)
        self.assertEqual(adapted_request.text, "Hello, how are you?")
        # self.assertEqual(adapted_request.rid, "test-id")
        self.assertEqual(processed_request, self.basic_req)
```
**EN:** Test converting single string request to internal format. This test exercises `test_convert_single_string_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test converting single string request to internal format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_single_string_request`。

### Lines 162-173: test case convert list string request / 测试用例 convert list string request
```python
    def test_convert_list_string_request(self):
        """Test converting list of strings request to internal format."""
        adapted_request, processed_request = (
            self.serving_embedding._convert_to_internal_request(self.list_req)
        )

        self.assertIsInstance(adapted_request, EmbeddingReqInput)
        self.assertEqual(
            adapted_request.text, ["Hello, how are you?", "I am fine, thank you!"]
        )
        # self.assertEqual(adapted_request.rid, "test-id")
        self.assertEqual(processed_request, self.list_req)
```
**EN:** Test converting list of strings request to internal format. This test exercises `test_convert_list_string_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test converting list of strings request to internal format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_list_string_request`。

### Lines 175-184: test case convert token ids request / 测试用例 convert token ids request
```python
    def test_convert_token_ids_request(self):
        """Test converting token IDs request to internal format."""
        adapted_request, processed_request = (
            self.serving_embedding._convert_to_internal_request(self.token_ids_req)
        )

        self.assertIsInstance(adapted_request, EmbeddingReqInput)
        self.assertEqual(adapted_request.input_ids, [1, 2, 3, 4, 5])
        # self.assertEqual(adapted_request.rid, "test-id")
        self.assertEqual(processed_request, self.token_ids_req)
```
**EN:** Test converting token IDs request to internal format. This test exercises `test_convert_token_ids_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test converting token IDs request to internal format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_token_ids_request`。

### Lines 186-198: test case convert multimodal request / 测试用例 convert multimodal request
```python
    def test_convert_multimodal_request(self):
        """Test converting multimodal request to internal format."""
        adapted_request, processed_request = (
            self.serving_embedding._convert_to_internal_request(self.multimodal_req)
        )

        self.assertIsInstance(adapted_request, EmbeddingReqInput)
        # Should extract text and images separately
        self.assertEqual(len(adapted_request.text), 2)
        self.assertIn("Hello", adapted_request.text)
        self.assertIn("World", adapted_request.text)
        self.assertEqual(adapted_request.image_data[0], "base64_image_data")
        self.assertIsNone(adapted_request.image_data[1])
```
**EN:** Test converting multimodal request to internal format. This test exercises `test_convert_multimodal_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test converting multimodal request to internal format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_multimodal_request`。

### Lines 199-200: supporting source context / 辅助源码上下文
```python
        # self.assertEqual(adapted_request.rid, "test-id")

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 201-241: test case convert multimodal request with jinja chat template / 测试用例 convert multimodal request with jinja chat template
```python
    def test_convert_multimodal_request_with_jinja_chat_template(self):
        """Multimodal embeddings should apply explicit/HF Jinja chat templates."""
        self.tokenizer_manager.tokenizer.chat_template = "mock-template"
        self.tokenizer_manager.tokenizer.apply_chat_template = Mock(
            side_effect=[
                "<prompt>Hello<image></prompt>",
                "<prompt>World</prompt>",
            ]
        )

        adapted_request, _ = self.serving_embedding._convert_to_internal_request(
            self.multimodal_req
        )

        self.assertEqual(
            adapted_request.text,
            ["<prompt>Hello<image></prompt>", "<prompt>World</prompt>"],
        )
        self.assertEqual(adapted_request.image_data[0], "base64_image_data")
        self.assertIsNone(adapted_request.image_data[1])
        self.assertEqual(
            self.tokenizer_manager.tokenizer.apply_chat_template.call_count, 2
        )
        first_call = (
            self.tokenizer_manager.tokenizer.apply_chat_template.call_args_list[0]
        )
        first_messages = first_call.args[0]
        self.assertEqual(first_messages[0]["role"], "user")
        self.assertEqual(first_messages[0]["content"][0]["type"], "image")
        self.assertEqual(first_messages[0]["content"][1]["type"], "text")
        self.assertEqual(first_messages[0]["content"][1]["text"], "Hello")
        self.assertEqual(first_call.kwargs["tokenize"], False)
        self.assertEqual(first_call.kwargs["add_generation_prompt"], True)

        second_call = (
            self.tokenizer_manager.tokenizer.apply_chat_template.call_args_list[1]
        )
        second_messages = second_call.args[0]
        self.assertEqual(len(second_messages[0]["content"]), 1)
        self.assertEqual(second_messages[0]["content"][0]["type"], "text")
        self.assertEqual(second_messages[0]["content"][0]["text"], "World")
```
**EN:** Multimodal embeddings should apply explicit/HF Jinja chat templates. This test exercises `test_convert_multimodal_request_with_jinja_chat_template` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Multimodal embeddings should apply explicit/HF Jinja chat templates. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_multimodal_request_with_jinja_chat_template`。

### Lines 243-259: test case convert image only multimodal request with jinja chat template / 测试用例 convert image only multimodal request with jinja chat template
```python
    def test_convert_image_only_multimodal_request_with_jinja_chat_template(self):
        """Image-only requests should not inject literal padding into Jinja prompts."""
        self.tokenizer_manager.tokenizer.chat_template = "mock-template"
        self.tokenizer_manager.tokenizer.apply_chat_template = Mock(
            return_value="<prompt><image></prompt>"
        )

        adapted_request, _ = self.serving_embedding._convert_to_internal_request(
            self.image_only_multimodal_req
        )

        self.assertEqual(adapted_request.text, "<prompt><image></prompt>")
        first_call = self.tokenizer_manager.tokenizer.apply_chat_template.call_args
        first_messages = first_call.args[0]
        self.assertEqual(first_messages[0]["role"], "user")
        self.assertEqual(len(first_messages[0]["content"]), 1)
        self.assertEqual(first_messages[0]["content"][0]["type"], "image")
```
**EN:** Image-only requests should not inject literal padding into Jinja prompts. This test exercises `test_convert_image_only_multimodal_request_with_jinja_chat_template` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Image-only requests should not inject literal padding into Jinja prompts. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_image_only_multimodal_request_with_jinja_chat_template`。

### Lines 261-279: test case convert video multimodal request with jinja chat template / 测试用例 convert video multimodal request with jinja chat template
```python
    def test_convert_video_multimodal_request_with_jinja_chat_template(self):
        """Video inputs should land in video_data and flow through the Jinja branch."""
        self.tokenizer_manager.tokenizer.chat_template = "mock-template"
        self.tokenizer_manager.tokenizer.apply_chat_template = Mock(
            return_value="<prompt>Describe<video></prompt>"
        )

        adapted_request, _ = self.serving_embedding._convert_to_internal_request(
            self.video_multimodal_req
        )

        self.assertEqual(adapted_request.text, "<prompt>Describe<video></prompt>")
        self.assertEqual(adapted_request.video_data, "base64_video_data")
        self.assertIsNone(adapted_request.image_data)
        first_messages = (
            self.tokenizer_manager.tokenizer.apply_chat_template.call_args.args[0]
        )
        content = first_messages[0]["content"]
        self.assertEqual([c["type"] for c in content], ["video", "text"])
```
**EN:** Video inputs should land in video_data and flow through the Jinja branch. This test exercises `test_convert_video_multimodal_request_with_jinja_chat_template` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Video inputs should land in video_data and flow through the Jinja branch. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_video_multimodal_request_with_jinja_chat_template`。

### Lines 281-291: test case multimodal request falls back when no chat template / 测试用例 multimodal request falls back when no chat template
```python
    def test_multimodal_request_falls_back_when_no_chat_template(self):
        """Without any chat template the raw-text fallback must run without raising."""
        self.tokenizer_manager.tokenizer.chat_template = None

        adapted_request, _ = self.serving_embedding._convert_to_internal_request(
            self.image_only_multimodal_req
        )

        # text=None on an image-only input falls back to the "padding" literal.
        self.assertEqual(adapted_request.text, "padding")
        self.assertEqual(adapted_request.image_data, "base64_image_data")
```
**EN:** Without any chat template the raw-text fallback must run without raising. This test exercises `test_multimodal_request_falls_back_when_no_chat_template` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Without any chat template the raw-text fallback must run without raising. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multimodal_request_falls_back_when_no_chat_template`。

### Lines 293-301: test case multimodal request with no tokenizer uses fallback / 测试用例 multimodal request with no tokenizer uses fallback
```python
    def test_multimodal_request_with_no_tokenizer_uses_fallback(self):
        """Missing tokenizer should not crash the Jinja branch check."""
        self.tokenizer_manager.tokenizer = None

        adapted_request, _ = self.serving_embedding._convert_to_internal_request(
            self.multimodal_req
        )

        self.assertEqual(adapted_request.text, ["Hello", "World"])
```
**EN:** Missing tokenizer should not crash the Jinja branch check. This test exercises `test_multimodal_request_with_no_tokenizer_uses_fallback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Missing tokenizer should not crash the Jinja branch check. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multimodal_request_with_no_tokenizer_uses_fallback`。

### Lines 303-313: test case jinja template errors are raised as value error / 测试用例 jinja template errors are raised as value error
```python
    def test_jinja_template_errors_are_raised_as_value_error(self):
        """Template failures should be converted to ValueError for a 400 response."""
        self.tokenizer_manager.tokenizer.chat_template = "mock-template"
        self.tokenizer_manager.tokenizer.apply_chat_template = Mock(
            side_effect=jinja2.TemplateError("bad template")
        )

        with self.assertRaisesRegex(ValueError, "bad template"):
            self.serving_embedding._convert_to_internal_request(
                self.image_only_multimodal_req
            )
```
**EN:** Template failures should be converted to ValueError for a 400 response. This test exercises `test_jinja_template_errors_are_raised_as_value_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Template failures should be converted to ValueError for a 400 response. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_jinja_template_errors_are_raised_as_value_error`。

### Lines 315-327: test case jinja template syntax error includes location / 测试用例 jinja template syntax error includes location
```python
    def test_jinja_template_syntax_error_includes_location(self):
        """TemplateSyntaxError should surface template name and line number."""
        err = jinja2.TemplateSyntaxError("unexpected end", lineno=7, name="mock.jinja")
        self.tokenizer_manager.tokenizer.chat_template = "mock-template"
        self.tokenizer_manager.tokenizer.apply_chat_template = Mock(side_effect=err)

        with self.assertRaises(ValueError) as ctx:
            self.serving_embedding._convert_to_internal_request(
                self.image_only_multimodal_req
            )
        message = str(ctx.exception)
        self.assertIn("mock.jinja", message)
        self.assertIn("line=7", message)
```
**EN:** TemplateSyntaxError should surface template name and line number. This test exercises `test_jinja_template_syntax_error_includes_location` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** TemplateSyntaxError should surface template name and line number. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_jinja_template_syntax_error_includes_location`。

### Lines 329-339: test case non jinja template errors are raised as value error / 测试用例 non jinja template errors are raised as value error
```python
    def test_non_jinja_template_errors_are_raised_as_value_error(self):
        """TypeError / KeyError from apply_chat_template should map to 400, not 500."""
        self.tokenizer_manager.tokenizer.chat_template = "mock-template"
        self.tokenizer_manager.tokenizer.apply_chat_template = Mock(
            side_effect=KeyError("missing_field")
        )

        with self.assertRaisesRegex(ValueError, "missing_field"):
            self.serving_embedding._convert_to_internal_request(
                self.image_only_multimodal_req
            )
```
**EN:** TypeError / KeyError from apply_chat_template should map to 400, not 500. This test exercises `test_non_jinja_template_errors_are_raised_as_value_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** TypeError / KeyError from apply_chat_template should map to 400, not 500. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_jinja_template_errors_are_raised_as_value_error`。

### Lines 342-343: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_SglKernelMockLoader`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_SglKernelMockFinder`: Import hook that intercepts all sgl_kernel.* imports and returns mocks. / 用于组织相关测试、夹具或辅助方法。
- `_MockTokenizerManager`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_MockTemplateManager`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `ServingEmbeddingTestCase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_SglKernelMockLoader.create_module`: This helper encapsulates `create_module` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `create_module`，以便周围测试复用准备、执行或校验逻辑。
- `_SglKernelMockLoader.exec_module`: This block implements `exec_module` and captures one focused piece of the module's behavior. / 该代码块实现 `exec_module`，承担模块行为中的一个聚焦逻辑片段。
- `_SglKernelMockFinder.find_spec`: This block implements `find_spec` and captures one focused piece of the module's behavior. / 该代码块实现 `find_spec`，承担模块行为中的一个聚焦逻辑片段。
- `_MockTokenizerManager.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `_MockTemplateManager.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `ServingEmbeddingTestCase.setUp`: Set up test fixtures. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `ServingEmbeddingTestCase.test_convert_single_string_request`: Test converting single string request to internal format. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_convert_single_string_request`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `importlib`, `importlib.abc`, `importlib.machinery`, `sys`, `types`, `unittest`, `uuid`, `unittest.mock`
- **Third-party modules / 第三方模块**: `jinja2`, `fastapi`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.entrypoints.openai.serving_embedding`, `sglang.srt.managers.io_struct`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 343
