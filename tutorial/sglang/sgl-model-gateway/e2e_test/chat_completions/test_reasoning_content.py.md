# test_reasoning_content.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/chat_completions/test_reasoning_content.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises reasoning content behavior in the chat completions end-to-end suite. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 Chat Completions 端到端测试 中与 reasoning content 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module docstring
```python
"""Reasoning Content E2E Tests.

Tests for chat completions with reasoning content (DeepSeek R1 reasoning parser).

Source: Migrated from e2e_grpc/features/test_reasoning_content.py
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 8-12: Imports and dependencies
```python
from __future__ import annotations

import logging

import pytest
```
**EN:** This block imports `__future__`, `logging`, `pytest`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 14-14: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 15-165: Test class `TestReasoningContentAPI`
```python


# =============================================================================
# Reasoning Content API Tests (DeepSeek 7B)
# =============================================================================


@pytest.mark.model("deepseek-7b")
@pytest.mark.gateway(
    extra_args=["--reasoning-parser", "deepseek_r1", "--history-backend", "memory"]
)
@pytest.mark.parametrize("setup_backend", ["grpc"], indirect=True)
class TestReasoningContentAPI:
    """Tests for reasoning content API with DeepSeek R1 reasoning parser."""

    def test_streaming_separate_reasoning_false(self, setup_backend):
        """Test streaming with separate_reasoning=False, reasoning_content should be empty."""
        _, model, client, gateway = setup_backend

        response = client.chat.completions.create(
            model=model,
            messages=[
                {
                    "role": "user",
                    "content": "What is 1+3?",
                }
            ],
            max_tokens=100,
            stream=True,
            extra_body={"separate_reasoning": False},
        )

        reasoning_content = ""
        content = ""
        for chunk in response:
            if chunk.choices[0].delta.content:
                content += chunk.choices[0].delta.content
            elif chunk.choices[0].delta.reasoning_content:
                reasoning_content += chunk.choices[0].delta.reasoning_content

        assert len(reasoning_content) == 0
        assert len(content) > 0

    def test_streaming_separate_reasoning_true(self, setup_backend):
        """Test streaming with separate_reasoning=True, reasoning_content should not be empty."""
        _, model, client, gateway = setup_backend

        response = client.chat.completions.create(
            model=model,
            messages=[
                {
                    "role": "user",
                    "content": "What is 1+3?",
                }
            ],
            max_tokens=100,
            stream=True,
            extra_body={"separate_reasoning": True},
        )

        reasoning_content = ""
        content = ""
        for chunk in response:
            if chunk.choices[0].delta.content:
                content += chunk.choices[0].delta.content
            elif chunk.choices[0].delta.reasoning_content:
                reasoning_content += chunk.choices[0].delta.reasoning_content

        assert len(reasoning_content) > 0
        assert len(content) > 0

    def test_streaming_separate_reasoning_true_stream_reasoning_false(
        self, setup_backend
    ):
        """Test streaming with separate_reasoning=True and stream_reasoning=False."""
        _, model, client, gateway = setup_backend

        response = client.chat.completions.create(
            model=model,
            messages=[
                {
                    "role": "user",
                    "content": "What is 1+3?",
                }
            ],
            max_tokens=100,
            stream=True,
            extra_body={"separate_reasoning": True, "stream_reasoning": False},
        )

        reasoning_content = ""
        content = ""
        first_chunk = False
        for chunk in response:
            if chunk.choices[0].delta.reasoning_content:
                reasoning_content = chunk.choices[0].delta.reasoning_content
                first_chunk = True
            if chunk.choices[0].delta.content:
                content += chunk.choices[0].delta.content
                if not first_chunk:
                    reasoning_content = chunk.choices[0].delta.reasoning_content
                first_chunk = True
            if not first_chunk:
                assert (
                    not chunk.choices[0].delta.reasoning_content
                    or len(chunk.choices[0].delta.reasoning_content) == 0
                )

        assert len(reasoning_content) > 0
        assert len(content) > 0

    def test_nonstreaming_separate_reasoning_false(self, setup_backend):
        """Test non-streaming with separate_reasoning=False, reasoning_content should be empty."""
        _, model, client, gateway = setup_backend

        response = client.chat.completions.create(
            model=model,
            messages=[
                {
                    "role": "user",
                    "content": "What is 1+3?",
                }
            ],
            max_tokens=100,
            extra_body={"separate_reasoning": False},
        )

        assert (
            not response.choices[0].message.reasoning_content
            or len(response.choices[0].message.reasoning_content) == 0
        )
        assert len(response.choices[0].message.content) > 0

    def test_nonstreaming_separate_reasoning_true(self, setup_backend):
        """Test non-streaming with separate_reasoning=True, reasoning_content should not be empty."""
        _, model, client, gateway = setup_backend

        response = client.chat.completions.create(
            model=model,
            messages=[
                {
                    "role": "user",
                    "content": "What is 1+3?",
                }
            ],
            max_tokens=100,
            extra_body={"separate_reasoning": True},
        )

        assert len(response.choices[0].message.reasoning_content) > 0
        assert len(response.choices[0].message.content) > 0
```
**EN:** This test class groups related scenarios around `TestReasoningContentAPI` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestReasoningContentAPI` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- OpenAI-compatible chat completions behavior / 兼容 OpenAI 的聊天补全行为
- Primary classes: `TestReasoningContentAPI` / 主要类：`TestReasoningContentAPI`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`
- **Third-party / 第三方**: `pytest`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
