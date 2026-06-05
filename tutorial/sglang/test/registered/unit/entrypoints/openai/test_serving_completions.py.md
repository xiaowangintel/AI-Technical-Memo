# test_serving_completions.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/entrypoints/openai/test_serving_completions.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates serving completions behavior in SGLang's unit / entrypoints / openai area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 入口 / OpenAI 领域中与 serving completions 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: supporting statements / 辅助语句
```python
"""
Unit-tests for the refactored completions-serving handler (no pytest).
Run with:
    python -m unittest tests.test_serving_completions_unit -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 7-7: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.test_utils import maybe_stub_sgl_kernel
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.test_utils`。

### Lines 9-9: supporting statements / 辅助语句
```python
maybe_stub_sgl_kernel()  # must precede any import that pulls in sgl_kernel
```
**EN:** This block performs supporting work through calls such as `maybe_stub_sgl_kernel`, preparing state for nearby definitions.
**CN:** 该代码块通过 `maybe_stub_sgl_kernel` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 11-23: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest
from http import HTTPStatus
from typing import Optional
from unittest.mock import AsyncMock, Mock

from fastapi import Request

from sglang.srt.entrypoints.openai.protocol import CompletionRequest
from sglang.srt.entrypoints.openai.serving_completions import OpenAIServingCompletion
from sglang.srt.managers.tokenizer_manager import TokenizerManager
from sglang.srt.utils import get_or_create_event_loop
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `http`, `typing`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `http`, `typing`。

### Lines 25-25: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=11, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 28-28: class _MockTemplateManager declaration / 类 _MockTemplateManager 声明
```python
class _MockTemplateManager:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 29-29: supporting statements / 辅助语句
```python
    """Minimal mock for TemplateManager."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 31-36: method init / 方法 init
```python
    def __init__(self):
        self.chat_template_name: Optional[str] = None
        self.jinja_template_content_format: Optional[str] = None
        self.completion_template_name: Optional[str] = (
            None  # Set to None to avoid template processing
        )
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 39-39: class ServingCompletionTestCase declaration / 类 ServingCompletionTestCase 声明
```python
class ServingCompletionTestCase(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 40-40: supporting statements / 辅助语句
```python
    """Bundle all prompt/echo tests in one TestCase."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 41-42: supporting source context / 辅助源码上下文
```python

    # ---------- shared test fixtures ----------
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 43-60: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        # build the mock TokenizerManager once for every test
        tm = Mock(spec=TokenizerManager)

        tm.tokenizer = Mock()
        tm.tokenizer.encode.return_value = [1, 2, 3, 4]
        tm.tokenizer.decode.return_value = "decoded text"
        tm.tokenizer.bos_token_id = 1

        tm.model_config = Mock(is_multimodal=False)
        tm.server_args = Mock(enable_cache_report=False)

        tm.generate_request = AsyncMock()
        tm.create_abort_task = Mock()

        self.template_manager = _MockTemplateManager()
        self.sc = OpenAIServingCompletion(tm, self.template_manager)
        self.fastapi_request = Mock(spec=Request)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 61-62: supporting source context / 辅助源码上下文
```python

    # ---------- prompt-handling ----------
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 63-66: test case single string prompt / 测试用例 single string prompt
```python
    def test_single_string_prompt(self):
        req = CompletionRequest(model="x", prompt="Hello world", max_tokens=100)
        internal, _ = self.sc._convert_to_internal_request(req)
        self.assertEqual(internal.text, "Hello world")
```
**EN:** This test exercises `test_single_string_prompt` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_string_prompt`。

### Lines 68-71: test case single token ids prompt / 测试用例 single token ids prompt
```python
    def test_single_token_ids_prompt(self):
        req = CompletionRequest(model="x", prompt=[1, 2, 3, 4], max_tokens=100)
        internal, _ = self.sc._convert_to_internal_request(req)
        self.assertEqual(internal.input_ids, [1, 2, 3, 4])
```
**EN:** This test exercises `test_single_token_ids_prompt` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_token_ids_prompt`。

### Lines 72-73: supporting source context / 辅助源码上下文
```python

    # ---------- echo-handling ----------
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 74-76: test case echo with string prompt streaming / 测试用例 echo with string prompt streaming
```python
    def test_echo_with_string_prompt_streaming(self):
        req = CompletionRequest(model="x", prompt="Hello", max_tokens=1, echo=True)
        self.assertEqual(self.sc._get_echo_text(req, 0), "Hello")
```
**EN:** This test exercises `test_echo_with_string_prompt_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_echo_with_string_prompt_streaming`。

### Lines 78-83: test case echo with list of strings streaming / 测试用例 echo with list of strings streaming
```python
    def test_echo_with_list_of_strings_streaming(self):
        req = CompletionRequest(
            model="x", prompt=["A", "B"], max_tokens=1, echo=True, n=1
        )
        self.assertEqual(self.sc._get_echo_text(req, 0), "A")
        self.assertEqual(self.sc._get_echo_text(req, 1), "B")
```
**EN:** This test exercises `test_echo_with_list_of_strings_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_echo_with_list_of_strings_streaming`。

### Lines 85-88: test case echo with token ids streaming / 测试用例 echo with token ids streaming
```python
    def test_echo_with_token_ids_streaming(self):
        req = CompletionRequest(model="x", prompt=[1, 2, 3], max_tokens=1, echo=True)
        self.sc.tokenizer_manager.tokenizer.decode.return_value = "decoded_prompt"
        self.assertEqual(self.sc._get_echo_text(req, 0), "decoded_prompt")
```
**EN:** This test exercises `test_echo_with_token_ids_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_echo_with_token_ids_streaming`。

### Lines 90-95: test case echo with multiple token ids streaming / 测试用例 echo with multiple token ids streaming
```python
    def test_echo_with_multiple_token_ids_streaming(self):
        req = CompletionRequest(
            model="x", prompt=[[1, 2], [3, 4]], max_tokens=1, echo=True, n=1
        )
        self.sc.tokenizer_manager.tokenizer.decode.return_value = "decoded"
        self.assertEqual(self.sc._get_echo_text(req, 0), "decoded")
```
**EN:** This test exercises `test_echo_with_multiple_token_ids_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_echo_with_multiple_token_ids_streaming`。

### Lines 97-109: test case prepare echo prompts non streaming / 测试用例 prepare echo prompts non streaming
```python
    def test_prepare_echo_prompts_non_streaming(self):
        # single string
        req = CompletionRequest(model="x", prompt="Hi", echo=True)
        self.assertEqual(self.sc._prepare_echo_prompts(req), ["Hi"])

        # list of strings
        req = CompletionRequest(model="x", prompt=["Hi", "Yo"], echo=True)
        self.assertEqual(self.sc._prepare_echo_prompts(req), ["Hi", "Yo"])

        # token IDs
        req = CompletionRequest(model="x", prompt=[1, 2, 3], echo=True)
        self.sc.tokenizer_manager.tokenizer.decode.return_value = "decoded"
        self.assertEqual(self.sc._prepare_echo_prompts(req), ["decoded"])
```
**EN:** This test exercises `test_prepare_echo_prompts_non_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_echo_prompts_non_streaming`。

### Lines 110-111: supporting source context / 辅助源码上下文
```python

    # ---------- response_format handling ----------
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 112-121: test case response format json object / 测试用例 response format json object
```python
    def test_response_format_json_object(self):
        """Test that response_format json_object is correctly processed in sampling params."""
        req = CompletionRequest(
            model="x",
            prompt="Generate a JSON object:",
            max_tokens=100,
            response_format={"type": "json_object"},
        )
        sampling_params = self.sc._build_sampling_params(req)
        self.assertEqual(sampling_params["json_schema"], '{"type": "object"}')
```
**EN:** Test that response_format json_object is correctly processed in sampling params. This test exercises `test_response_format_json_object` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that response_format json_object is correctly processed in sampling params. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_format_json_object`。

### Lines 123-141: test case response format json schema / 测试用例 response format json schema
```python
    def test_response_format_json_schema(self):
        """Test that response_format json_schema is correctly processed in sampling params."""
        schema = {
            "type": "object",
            "properties": {"name": {"type": "string"}, "age": {"type": "integer"}},
        }
        req = CompletionRequest(
            model="x",
            prompt="Generate a JSON object:",
            max_tokens=100,
            response_format={
                "type": "json_schema",
                "json_schema": {"name": "person", "schema": schema},
            },
        )
        sampling_params = self.sc._build_sampling_params(req)
        # The schema should be converted to string by convert_json_schema_to_str
        self.assertIn("json_schema", sampling_params)
        self.assertIsInstance(sampling_params["json_schema"], str)
```
**EN:** Test that response_format json_schema is correctly processed in sampling params. This test exercises `test_response_format_json_schema` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that response_format json_schema is correctly processed in sampling params. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_format_json_schema`。

### Lines 143-158: test case response format structural tag / 测试用例 response format structural tag
```python
    def test_response_format_structural_tag(self):
        """Test that response_format structural_tag is correctly processed in sampling params."""
        req = CompletionRequest(
            model="x",
            prompt="Generate structured output:",
            max_tokens=100,
            response_format={
                "type": "structural_tag",
                "structures": [{"begin": "<data>", "end": "</data>"}],
                "triggers": ["<data>"],
            },
        )
        sampling_params = self.sc._build_sampling_params(req)
        # The structural_tag should be processed
        self.assertIn("structural_tag", sampling_params)
        self.assertIsInstance(sampling_params["structural_tag"], str)
```
**EN:** Test that response_format structural_tag is correctly processed in sampling params. This test exercises `test_response_format_structural_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that response_format structural_tag is correctly processed in sampling params. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_format_structural_tag`。

### Lines 160-166: test case response format none / 测试用例 response format none
```python
    def test_response_format_none(self):
        """Test that no response_format doesn't add extra constraints."""
        req = CompletionRequest(model="x", prompt="Generate text:", max_tokens=100)
        sampling_params = self.sc._build_sampling_params(req)
        # Should not have json_schema or structural_tag from response_format
        # (but might have json_schema from the legacy json_schema field)
        self.assertIsNone(sampling_params.get("structural_tag"))
```
**EN:** Test that no response_format doesn't add extra constraints. This test exercises `test_response_format_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that no response_format doesn't add extra constraints. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_format_none`。

### Lines 168-191: test case logprobs false non streaming / 测试用例 logprobs false non streaming
```python
    def test_logprobs_false_non_streaming(self):
        """Test that logprobs=False doesn't cause KeyError in non-streaming response."""
        req = CompletionRequest(
            model="x", prompt="Hello", max_tokens=10, logprobs=False
        )

        mock_ret = [
            {
                "text": " world",
                "meta_info": {
                    "id": "test-id",
                    "prompt_tokens": 1,
                    "completion_tokens": 2,
                    "finish_reason": {"type": "stop"},
                    "weight_version": "v1",
                },
            }
        ]

        response = self.sc._build_completion_response(req, mock_ret, 1234567890)

        self.assertEqual(len(response.choices), 1)
        self.assertEqual(response.choices[0].text, " world")
        self.assertEqual(len(response.choices[0].logprobs.top_logprobs), 0)
```
**EN:** Test that logprobs=False doesn't cause KeyError in non-streaming response. This test exercises `test_logprobs_false_non_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that logprobs=False doesn't cause KeyError in non-streaming response. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logprobs_false_non_streaming`。

### Lines 193-257: test case streaming abort yields error / 测试用例 streaming abort yields error
```python
    def test_streaming_abort_yields_error(self):
        """Test that an abort finish reason during streaming correctly yields an error and stops."""
        err_msg = "Aborted by scheduler"
        err_code = HTTPStatus.INTERNAL_SERVER_ERROR

        async def _mock_generate_abort(*args, **kwargs):
            yield {
                "text": "Partial ",
                "meta_info": {
                    "id": "cmpl-test",
                    "prompt_tokens": 10,
                    "completion_tokens": 2,
                    "cached_tokens": 0,
                    "finish_reason": {
                        "type": "abort",
                        "status_code": err_code,
                        "message": err_msg,
                    },
                    "output_token_logprobs": None,
                    "output_top_logprobs": None,
                },
                "index": 0,
            }

        self.sc.tokenizer_manager.generate_request = _mock_generate_abort

        req = CompletionRequest(
            model="x",
            prompt="Hello world",
            max_tokens=100,
            stream=True,
        )

        adapted_request, _ = self.sc._convert_to_internal_request(req)

        async def run_stream():
            chunks = []
            try:
                async for chunk in self.sc._generate_completion_stream(
                    adapted_request, req, self.fastapi_request
                ):
                    chunks.append(chunk)
            except Exception as e:
                print(f"Error during stream iteration: {e}")
            return chunks

        loop = get_or_create_event_loop()
        chunks = loop.run_until_complete(run_stream())

        error_chunk_data = None
        for c in chunks:
            if "error" in c:
                error_chunk_data = json.loads(c[len("data: ") :])
                break
        self.assertIsNotNone(error_chunk_data, "Error chunk not found in stream")
        self.assertEqual(error_chunk_data["error"]["message"], err_msg)
        self.assertEqual(error_chunk_data["error"]["code"], err_code.value)

        # Ensure the stream stops after the abort error
        # The last chunk should be "data: [DONE]\n\n"
        self.assertEqual(chunks[-1], "data: [DONE]\n\n")

        # Check that there is an error chunk and a DONE chunk, and possibly a role chunk
        self.assertGreaterEqual(len(chunks), 2)
        self.assertIn("error", chunks[0])
```
**EN:** Test that an abort finish reason during streaming correctly yields an error and stops. This test exercises `test_streaming_abort_yields_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that an abort finish reason during streaming correctly yields an error and stops. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_abort_yields_error`。

### Lines 259-299: test case non streaming cached tokens details emits sglext / 测试用例 non streaming cached tokens details emits sglext
```python
    def test_non_streaming_cached_tokens_details_emits_sglext(self):
        """Test that non-streaming completion responses emit cached token details in sglext."""

        req = CompletionRequest(
            model="x",
            prompt="Hello world",
            max_tokens=100,
            return_cached_tokens_details=True,
        )
        ret = [
            {
                "text": "Cached response",
                "meta_info": {
                    "id": "cmpl-cache-test",
                    "prompt_tokens": 10,
                    "completion_tokens": 2,
                    "cached_tokens": 6,
                    "cached_tokens_details": {
                        "device": 4,
                        "host": 1,
                        "storage": 1,
                        "storage_backend": "file",
                    },
                    "finish_reason": {"type": "stop", "matched": None},
                    "weight_version": "default",
                },
            }
        ]

        response = self.sc._build_completion_response(req, ret, 1234567890)

        self.assertIsNotNone(response.sglext)
        self.assertEqual(
            response.sglext.cached_tokens_details.model_dump(exclude_none=True),
            {
                "device": 4,
                "host": 1,
                "storage": 1,
                "storage_backend": "file",
            },
        )
```
**EN:** Test that non-streaming completion responses emit cached token details in sglext. This test exercises `test_non_streaming_cached_tokens_details_emits_sglext` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that non-streaming completion responses emit cached token details in sglext. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_streaming_cached_tokens_details_emits_sglext`。

### Lines 301-368: test case streaming cached tokens details emits sglext / 测试用例 streaming cached tokens details emits sglext
```python
    def test_streaming_cached_tokens_details_emits_sglext(self):
        """Test that streaming completion responses emit cached token details in sglext."""

        async def _mock_generate_with_cached_tokens_details(*args, **kwargs):
            yield {
                "text": "Cached response",
                "meta_info": {
                    "id": "cmpl-cache-test",
                    "prompt_tokens": 10,
                    "completion_tokens": 2,
                    "cached_tokens": 6,
                    "cached_tokens_details": {
                        "device": 4,
                        "host": 1,
                        "storage": 1,
                        "storage_backend": "file",
                    },
                    "finish_reason": {"type": "stop", "matched": None},
                    "output_token_logprobs": None,
                    "output_top_logprobs": None,
                },
                "index": 0,
            }

        self.sc.tokenizer_manager.generate_request = (
            _mock_generate_with_cached_tokens_details
        )

        req = CompletionRequest(
            model="x",
            prompt="Hello world",
            max_tokens=100,
            stream=True,
            return_cached_tokens_details=True,
        )

        adapted_request, _ = self.sc._convert_to_internal_request(req)

        async def run_stream():
            chunks = []
            async for chunk in self.sc._generate_completion_stream(
                adapted_request, req, self.fastapi_request
            ):
                chunks.append(chunk)
            return chunks

        loop = get_or_create_event_loop()
        chunks = loop.run_until_complete(run_stream())

        sglext_chunks = []
        for chunk in chunks:
            if not chunk.startswith("data: ") or chunk.strip() == "data: [DONE]":
                continue
            data = json.loads(chunk[len("data: ") :])
            if "sglext" in data:
                sglext_chunks.append(data)

        self.assertEqual(len(sglext_chunks), 1)
        self.assertEqual(sglext_chunks[0]["choices"], [])
        self.assertEqual(
            sglext_chunks[0]["sglext"]["cached_tokens_details"],
            {
                "device": 4,
                "host": 1,
                "storage": 1,
                "storage_backend": "file",
            },
        )
```
**EN:** Test that streaming completion responses emit cached token details in sglext. This test exercises `test_streaming_cached_tokens_details_emits_sglext` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that streaming completion responses emit cached token details in sglext. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_cached_tokens_details_emits_sglext`。

### Lines 371-372: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_MockTemplateManager`: Minimal mock for TemplateManager. / 用于组织相关测试、夹具或辅助方法。
- `ServingCompletionTestCase`: Bundle all prompt/echo tests in one TestCase. / 用于组织相关测试、夹具或辅助方法。
- `_MockTemplateManager.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `ServingCompletionTestCase.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `ServingCompletionTestCase.test_single_string_prompt`: This test exercises `test_single_string_prompt` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_string_prompt`。
- `ServingCompletionTestCase.test_single_token_ids_prompt`: This test exercises `test_single_token_ids_prompt` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_token_ids_prompt`。
- `ServingCompletionTestCase.test_echo_with_string_prompt_streaming`: This test exercises `test_echo_with_string_prompt_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_echo_with_string_prompt_streaming`。
- `ServingCompletionTestCase.test_echo_with_list_of_strings_streaming`: This test exercises `test_echo_with_list_of_strings_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_echo_with_list_of_strings_streaming`。
- `ServingCompletionTestCase.test_echo_with_token_ids_streaming`: This test exercises `test_echo_with_token_ids_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_echo_with_token_ids_streaming`。
- `ServingCompletionTestCase.test_echo_with_multiple_token_ids_streaming`: This test exercises `test_echo_with_multiple_token_ids_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_echo_with_multiple_token_ids_streaming`。
- `ServingCompletionTestCase.test_prepare_echo_prompts_non_streaming`: This test exercises `test_prepare_echo_prompts_non_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prepare_echo_prompts_non_streaming`。
- `ServingCompletionTestCase.test_response_format_json_object`: Test that response_format json_object is correctly processed in sampling params. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_response_format_json_object`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`, `http`, `typing`, `unittest.mock`
- **Third-party modules / 第三方模块**: `fastapi`
- **Internal modules / 内部模块**: `sglang.test.test_utils`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.entrypoints.openai.serving_completions`, `sglang.srt.managers.tokenizer_manager`, `sglang.srt.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 372
