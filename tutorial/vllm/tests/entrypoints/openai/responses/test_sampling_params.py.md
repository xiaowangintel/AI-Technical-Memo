# test_sampling_params.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/responses/test_sampling_params.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers Responses API behavior and OpenAI-compatible serving. The file defines 7 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖Responses API 行为与OpenAI 兼容服务。它定义了 7 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L6-L17)
```python
import pytest
import torch
from openai.types.responses.response_format_text_json_schema_config import (
    ResponseFormatTextJSONSchemaConfig,
)
from pydantic import ValidationError

from vllm.entrypoints.openai.responses.protocol import (
    ResponsesRequest,
    ResponseTextConfig,
)
from vllm.sampling_params import StructuredOutputsParams
```
**EN:** Imports third-party packages like `openai.types.responses.response_format_text_json_schema_config.ResponseFormatTextJSONSchemaConfig`, `pydantic.ValidationError`, `pytest`, project helpers such as `vllm.entrypoints.openai.responses.protocol.ResponseTextConfig`, `vllm.entrypoints.openai.responses.protocol.ResponsesRequest`, `vllm.sampling_params.StructuredOutputsParams`.
**CN:** 导入第三方包（如 `openai.types.responses.response_format_text_json_schema_config.ResponseFormatTextJSONSchemaConfig`、`pydantic.ValidationError`、`pytest`）、项目内辅助模块（如 `vllm.entrypoints.openai.responses.protocol.ResponseTextConfig`、`vllm.entrypoints.openai.responses.protocol.ResponsesRequest`、`vllm.sampling_params.StructuredOutputsParams`）。

### Class / 类: TestResponsesRequestSamplingParams (L20-L156)
```python
class TestResponsesRequestSamplingParams:
    """Test that ResponsesRequest correctly maps parameters to SamplingParams."""

    def test_basic_sampling_params(self):
        """Test basic sampling parameters are correctly mapped."""
        request = ResponsesRequest(
            model="test-model",
            input="test input",
            temperature=0.8,
            top_p=0.95,
            top_k=50,
            max_output_tokens=100,
        )

# ... 115 lines omitted for brevity ...
        )

        with pytest.raises(ValueError) as exc_info:
            request.to_sampling_params(default_max_tokens=1000)

        assert "Cannot specify both structured_outputs and text.format" in str(
            exc_info.value
        )
```
**EN:** This class groups related scenarios in `TestResponsesRequestSamplingParams`. It contains 7 test method(s) and 0 supporting method(s). Representative methods include `test_basic_sampling_params`, `test_extra_sampling_params`, `test_stop_string_conversion`.
**CN:** 该类将与 `TestResponsesRequestSamplingParams` 相关的场景组织在一起。 它包含 7 个测试方法和 0 个辅助方法。 代表性方法包括 `test_basic_sampling_params`、`test_extra_sampling_params`、`test_stop_string_conversion`。

### Test method / 测试方法: TestResponsesRequestSamplingParams.test_basic_sampling_params (L23-L39)
```python
    def test_basic_sampling_params(self):
        """Test basic sampling parameters are correctly mapped."""
        request = ResponsesRequest(
            model="test-model",
            input="test input",
            temperature=0.8,
            top_p=0.95,
            top_k=50,
            max_output_tokens=100,
        )

        sampling_params = request.to_sampling_params(default_max_tokens=1000)

        assert sampling_params.temperature == 0.8
        assert sampling_params.top_p == 0.95
        assert sampling_params.top_k == 50
        assert sampling_params.max_tokens == 100
```
**EN:** This test validates `TestResponsesRequestSamplingParams.test_basic_sampling_params`. The main assertion is `sampling_params.temperature == 0.8` and `sampling_params.top_p == 0.95`.
**CN:** 这个测试验证 `TestResponsesRequestSamplingParams.test_basic_sampling_params`。 核心断言是 `sampling_params.temperature == 0.8` and `sampling_params.top_p == 0.95`。

### Test method / 测试方法: TestResponsesRequestSamplingParams.test_extra_sampling_params (L41-L59)
```python
    def test_extra_sampling_params(self):
        """Test extra sampling parameters are correctly mapped."""
        request = ResponsesRequest(
            model="test-model",
            input="test input",
            repetition_penalty=1.2,
            seed=42,
            stop=["END", "STOP"],
            ignore_eos=True,
            vllm_xargs={"custom": "value"},
        )

        sampling_params = request.to_sampling_params(default_max_tokens=1000)

        assert sampling_params.repetition_penalty == 1.2
        assert sampling_params.seed == 42
        assert sampling_params.stop == ["END", "STOP"]
        assert sampling_params.ignore_eos is True
        assert sampling_params.extra_args == {"custom": "value"}
```
**EN:** This test validates `TestResponsesRequestSamplingParams.test_extra_sampling_params`. The main assertion is `sampling_params.repetition_penalty == 1.2` and `sampling_params.seed == 42`.
**CN:** 这个测试验证 `TestResponsesRequestSamplingParams.test_extra_sampling_params`。 核心断言是 `sampling_params.repetition_penalty == 1.2` and `sampling_params.seed == 42`。

### Test method / 测试方法: TestResponsesRequestSamplingParams.test_stop_string_conversion (L61-L71)
```python
    def test_stop_string_conversion(self):
        """Test that single stop string is converted to list."""
        request = ResponsesRequest(
            model="test-model",
            input="test input",
            stop="STOP",
        )

        sampling_params = request.to_sampling_params(default_max_tokens=1000)

        assert sampling_params.stop == ["STOP"]
```
**EN:** This test validates `TestResponsesRequestSamplingParams.test_stop_string_conversion`. The main assertion is `sampling_params.stop == ['STOP']`.
**CN:** 这个测试验证 `TestResponsesRequestSamplingParams.test_stop_string_conversion`。 核心断言是 `sampling_params.stop == ['STOP']`。

### Test method / 测试方法: TestResponsesRequestSamplingParams.test_default_values (L73-L84)
```python
    def test_default_values(self):
        """Test default values for optional parameters."""
        request = ResponsesRequest(
            model="test-model",
            input="test input",
        )

        sampling_params = request.to_sampling_params(default_max_tokens=1000)

        assert sampling_params.repetition_penalty == 1.0  # None → 1.0
        assert sampling_params.stop == []  # Empty list
        assert sampling_params.extra_args == {}  # Empty dict
```
**EN:** This test validates `TestResponsesRequestSamplingParams.test_default_values`. The main assertion is `sampling_params.repetition_penalty == 1.0` and `sampling_params.stop == []`.
**CN:** 这个测试验证 `TestResponsesRequestSamplingParams.test_default_values`。 核心断言是 `sampling_params.repetition_penalty == 1.0` and `sampling_params.stop == []`。

### Test method / 测试方法: TestResponsesRequestSamplingParams.test_seed_bounds_validation (L86-L119)
```python
    def test_seed_bounds_validation(self):
        """Test that seed values outside torch.long bounds are rejected."""
        # Test seed below minimum
        with pytest.raises(ValidationError) as exc_info:
            ResponsesRequest(
                model="test-model",
                input="test input",
                seed=torch.iinfo(torch.long).min - 1,
            )
        assert "greater_than_equal" in str(exc_info.value).lower()

        # Test seed above maximum
        with pytest.raises(ValidationError) as exc_info:
            ResponsesRequest(
                model="test-model",
                input="test input",
                seed=torch.iinfo(torch.long).max + 1,
            )
# ... 8 lines omitted for brevity ...
        assert request_min.seed == torch.iinfo(torch.long).min

        request_max = ResponsesRequest(
            model="test-model",
            input="test input",
            seed=torch.iinfo(torch.long).max,
        )
        assert request_max.seed == torch.iinfo(torch.long).max
```
**EN:** This test validates `TestResponsesRequestSamplingParams.test_seed_bounds_validation`. It checks an expected failure path with `pytest.raises`. The main assertion is `'greater_than_equal' in str(exc_info.value).lower()` and `'less_than_equal' in str(exc_info.value).lower()`.
**CN:** 这个测试验证 `TestResponsesRequestSamplingParams.test_seed_bounds_validation`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `'greater_than_equal' in str(exc_info.value).lower()` and `'less_than_equal' in str(exc_info.value).lower()`。

### Test method / 测试方法: TestResponsesRequestSamplingParams.test_structured_outputs_passed_through (L121-L133)
```python
    def test_structured_outputs_passed_through(self):
        """Test that structured_outputs field is passed to SamplingParams."""
        structured_outputs = StructuredOutputsParams(grammar="root ::= 'hello'")
        request = ResponsesRequest(
            model="test-model",
            input="test input",
            structured_outputs=structured_outputs,
        )

        sampling_params = request.to_sampling_params(default_max_tokens=1000)

        assert sampling_params.structured_outputs is not None
        assert sampling_params.structured_outputs.grammar == "root ::= 'hello'"
```
**EN:** This test validates `TestResponsesRequestSamplingParams.test_structured_outputs_passed_through`. The main assertion is `sampling_params.structured_outputs is not None` and `sampling_params.structured_outputs.grammar == "root ::= 'hello'"`.
**CN:** 这个测试验证 `TestResponsesRequestSamplingParams.test_structured_outputs_passed_through`。 核心断言是 `sampling_params.structured_outputs is not None` and `sampling_params.structured_outputs.grammar == "root ::= 'hello'"`。

### Test method / 测试方法: TestResponsesRequestSamplingParams.test_structured_outputs_and_json_schema_conflict (L135-L156)
```python
    def test_structured_outputs_and_json_schema_conflict(self):
        """Test that specifying both structured_outputs and json_schema raises."""
        structured_outputs = StructuredOutputsParams(grammar="root ::= 'hello'")
        text_config = ResponseTextConfig()
        text_config.format = ResponseFormatTextJSONSchemaConfig(
            type="json_schema",
            name="test",
            schema={"type": "object"},
        )
        request = ResponsesRequest(
            model="test-model",
            input="test input",
            structured_outputs=structured_outputs,
            text=text_config,
        )

        with pytest.raises(ValueError) as exc_info:
            request.to_sampling_params(default_max_tokens=1000)

        assert "Cannot specify both structured_outputs and text.format" in str(
            exc_info.value
        )
```
**EN:** This test validates `TestResponsesRequestSamplingParams.test_structured_outputs_and_json_schema_conflict`. It checks an expected failure path with `pytest.raises`. The main assertion is `'Cannot specify both structured_outputs and text.format' in str(exc_info.value)`.
**CN:** 这个测试验证 `TestResponsesRequestSamplingParams.test_structured_outputs_and_json_schema_conflict`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `'Cannot specify both structured_outputs and text.format' in str(exc_info.value)`。

## Key Concepts / 关键概念
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `openai.types.responses.response_format_text_json_schema_config.ResponseFormatTextJSONSchemaConfig`, `pydantic.ValidationError`, `pytest`, `torch`
- **Project / 项目内**: `vllm.entrypoints.openai.responses.protocol.ResponseTextConfig`, `vllm.entrypoints.openai.responses.protocol.ResponsesRequest`, `vllm.sampling_params.StructuredOutputsParams`
