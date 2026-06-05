# test_structured_output.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/responses/test_structured_output.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises structured output behavior in the responses API end-to-end suite. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 Responses API 端到端测试 中与 structured output 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module docstring
```python
"""Structured output tests for Response API.

Tests for text.format field with json_object and json_schema formats.

Source: Migrated from e2e_response_api/features/test_structured_output.py
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 8-13: Imports and dependencies
```python
from __future__ import annotations

import json
import logging

import pytest
```
**EN:** This block imports `__future__`, `json`, `logging`, `pytest`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 15-15: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 16-109: Test class `TestStructuredOutputCloud`
```python


# =============================================================================
# Cloud Backend Tests (OpenAI)
# =============================================================================


@pytest.mark.parametrize("setup_backend", ["openai"], indirect=True)
class TestStructuredOutputCloud:
    """Structured output tests against cloud APIs."""

    def test_structured_output_json_schema(self, setup_backend):
        """Test structured output with json_schema format."""
        _, model, client, gateway = setup_backend

        params = {
            "model": model,
            "input": [
                {
                    "role": "system",
                    "content": "You are a helpful math tutor. Guide the user through the solution step by step.",
                },
                {"role": "user", "content": "how can I solve 8x + 7 = -23"},
            ],
            "text": {
                "format": {
                    "type": "json_schema",
                    "name": "math_reasoning",
                    "schema": {
                        "type": "object",
                        "properties": {
                            "steps": {
                                "type": "array",
                                "items": {
                                    "type": "object",
                                    "properties": {
                                        "explanation": {"type": "string"},
                                        "output": {"type": "string"},
                                    },
                                    "required": ["explanation", "output"],
                                    "additionalProperties": False,
                                },
                            },
                            "final_answer": {"type": "string"},
                        },
                        "required": ["steps", "final_answer"],
                        "additionalProperties": False,
                    },
                    "strict": True,
                }
            },
        }

        create_resp = client.responses.create(**params)
        assert create_resp.error is None
        assert create_resp.id is not None
        assert create_resp.output is not None
        assert create_resp.text is not None

        # Verify text format was echoed back correctly
        assert create_resp.text.format is not None
        assert create_resp.text.format.type == "json_schema"
        assert create_resp.text.format.name == "math_reasoning"
        assert create_resp.text.format.schema_ is not None
        assert create_resp.text.format.strict

        # Find the message output
        output_text = next(
            (
                content.text
                for item in create_resp.output
                if item.type == "message"
                for content in item.content
                if content.type == "output_text"
            ),
            None,
        )

        assert output_text is not None, "No output_text found in response"
        assert output_text.strip(), "output_text is empty"

        # Parse JSON output
        output_json = json.loads(output_text)

        # Verify schema structure
        assert "steps" in output_json
        assert "final_answer" in output_json
        assert isinstance(output_json["steps"], list)
        assert len(output_json["steps"]) > 0

        # Verify each step has required fields
        for step in output_json["steps"]:
            assert "explanation" in step
            assert "output" in step
```
**EN:** This test class groups related scenarios around `TestStructuredOutputCloud` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestStructuredOutputCloud` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

### Lines 110-208: Test class `TestStructuredOutputHarmony`
```python


# =============================================================================
# Local Backend Tests (gRPC with Harmony model - complex schema)
# =============================================================================


@pytest.mark.e2e
@pytest.mark.model("gpt-oss")
@pytest.mark.gateway(
    extra_args=["--reasoning-parser=gpt-oss", "--history-backend", "memory"]
)
@pytest.mark.parametrize("setup_backend", ["grpc"], indirect=True)
class TestStructuredOutputHarmony:
    """Structured output tests against local gRPC backend with Harmony model."""

    def test_structured_output_json_schema(self, setup_backend):
        """Test structured output with json_schema format."""
        _, model, client, gateway = setup_backend

        params = {
            "model": model,
            "input": [
                {
                    "role": "system",
                    "content": "You are a helpful math tutor. Guide the user through the solution step by step.",
                },
                {"role": "user", "content": "how can I solve 8x + 7 = -23"},
            ],
            "text": {
                "format": {
                    "type": "json_schema",
                    "name": "math_reasoning",
                    "schema": {
                        "type": "object",
                        "properties": {
                            "steps": {
                                "type": "array",
                                "items": {
                                    "type": "object",
                                    "properties": {
                                        "explanation": {"type": "string"},
                                        "output": {"type": "string"},
                                    },
                                    "required": ["explanation", "output"],
                                    "additionalProperties": False,
                                },
                            },
                            "final_answer": {"type": "string"},
                        },
                        "required": ["steps", "final_answer"],
                        "additionalProperties": False,
                    },
                    "strict": True,
                }
            },
        }

        create_resp = client.responses.create(**params)
        assert create_resp.error is None
        assert create_resp.id is not None
        assert create_resp.output is not None
        assert create_resp.text is not None

        # Verify text format was echoed back correctly
        assert create_resp.text.format is not None
        assert create_resp.text.format.type == "json_schema"
        assert create_resp.text.format.name == "math_reasoning"
        assert create_resp.text.format.schema_ is not None
        assert create_resp.text.format.strict

        # Find the message output (output[0] may be reasoning, output[1] is message)
        output_text = next(
            (
                content.text
                for item in create_resp.output
                if item.type == "message"
                for content in item.content
                if content.type == "output_text"
            ),
            None,
        )

        assert output_text is not None, "No output_text found in response"
        assert output_text.strip(), "output_text is empty"

        # Parse JSON output
        output_json = json.loads(output_text)

        # Verify schema structure
        assert "steps" in output_json
        assert "final_answer" in output_json
        assert isinstance(output_json["steps"], list)
        assert len(output_json["steps"]) > 0

        # Verify each step has required fields
        for step in output_json["steps"]:
            assert "explanation" in step
            assert "output" in step
```
**EN:** This test class groups related scenarios around `TestStructuredOutputHarmony` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestStructuredOutputHarmony` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

### Lines 209-289: Test class `TestSimpleSchemaStructuredOutput`
```python


# =============================================================================
# Local Backend Tests (gRPC with Qwen model - simple schema)
# =============================================================================


@pytest.mark.e2e
@pytest.mark.model("qwen-14b")
@pytest.mark.gateway(
    extra_args=["--tool-call-parser", "qwen", "--history-backend", "memory"]
)
@pytest.mark.parametrize("setup_backend", ["grpc"], indirect=True)
class TestSimpleSchemaStructuredOutput:
    """Structured output tests with simpler schema for models that don't
    handle complex schemas well.
    """

    def test_structured_output_json_schema(self, setup_backend):
        """Test structured output with simple json_schema format."""
        _, model, client, gateway = setup_backend

        params = {
            "model": model,
            "input": [
                {
                    "role": "system",
                    "content": "You are a math solver. Return ONLY a JSON object that matches the schema-no extra text.",
                },
                {
                    "role": "user",
                    "content": "What is 1 + 1?",
                },
            ],
            "text": {
                "format": {
                    "type": "json_schema",
                    "name": "math_answer",
                    "schema": {
                        "type": "object",
                        "properties": {"answer": {"type": "string"}},
                        "required": ["answer"],
                    },
                }
            },
        }

        create_resp = client.responses.create(**params)
        assert create_resp.error is None
        assert create_resp.id is not None
        assert create_resp.output is not None
        assert create_resp.text is not None

        # Verify text format was echoed back correctly
        assert create_resp.text.format is not None
        assert create_resp.text.format.type == "json_schema"
        assert create_resp.text.format.name == "math_answer"
        assert create_resp.text.format.schema_ is not None

        # Find the message output
        output_text = next(
            (
                content.text
                for item in create_resp.output
                if item.type == "message"
                for content in item.content
                if content.type == "output_text"
            ),
            None,
        )

        assert output_text is not None, "No output_text found in response"
        assert output_text.strip(), "output_text is empty"

        # Parse JSON output
        output_json = json.loads(output_text)

        # Verify simple schema structure (just answer field)
        assert "answer" in output_json
        assert isinstance(output_json["answer"], str)
        assert output_json["answer"], "Answer is empty"
```
**EN:** This test class groups related scenarios around `TestSimpleSchemaStructuredOutput` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestSimpleSchemaStructuredOutput` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Responses API state, tools, and streaming flows / Responses API 的状态、工具与流式流程
- Primary classes: `TestStructuredOutputCloud`, `TestStructuredOutputHarmony`, `TestSimpleSchemaStructuredOutput` / 主要类：`TestStructuredOutputCloud`, `TestStructuredOutputHarmony`, `TestSimpleSchemaStructuredOutput`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `json`, `logging`
- **Third-party / 第三方**: `pytest`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
