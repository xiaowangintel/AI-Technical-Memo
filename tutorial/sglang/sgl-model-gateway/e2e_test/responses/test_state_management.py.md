# test_state_management.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/responses/test_state_management.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises state management behavior in the responses API end-to-end suite. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 Responses API 端到端测试 中与 state management 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Module docstring
```python
"""State management tests for Response API.

Tests both previous_response_id and conversation-based state management.
These tests work across local (gRPC) and cloud (OpenAI, xAI) backends.

Source: Migrated from e2e_response_api/features/test_state_management.py
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 9-14: Imports and dependencies
```python
from __future__ import annotations

import logging

import openai
import pytest
```
**EN:** This block imports `__future__`, `logging`, `openai`, `pytest`, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 16-16: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 17-151: Test class `TestStateManagementCloud`
```python


# =============================================================================
# Cloud Backend Tests (OpenAI, xAI)
# =============================================================================


@pytest.mark.parametrize("setup_backend", ["openai", "xai"], indirect=True)
class TestStateManagementCloud:
    """State management tests against cloud APIs."""

    def test_basic_response_creation(self, setup_backend):
        """Test basic response creation without state."""
        _, model, client, gateway = setup_backend

        resp = client.responses.create(model=model, input="What is 2+2?")

        assert resp.id is not None
        assert resp.error is None
        assert resp.status == "completed"
        assert len(resp.output_text) > 0
        assert resp.usage is not None

    def test_streaming_response(self, setup_backend):
        """Test streaming response."""
        _, model, client, gateway = setup_backend

        resp = client.responses.create(
            model=model, input="Count to 5", stream=True, max_output_tokens=50
        )

        events = list(resp)
        created_events = [e for e in events if e.type == "response.created"]
        assert len(created_events) > 0

        assert any(
            e.type in ["response.completed", "response.in_progress"] for e in events
        )

    def test_previous_response_id_chaining(self, setup_backend):
        """Test chaining responses using previous_response_id."""
        _, model, client, gateway = setup_backend

        # First response
        resp1 = client.responses.create(
            model=model, input="My name is Alice and my friend is Bob. Remember it."
        )
        assert resp1.error is None
        assert resp1.status == "completed"

        # Second response referencing first
        resp2 = client.responses.create(
            model=model, input="What is my name", previous_response_id=resp1.id
        )
        assert resp2.error is None
        assert resp2.status == "completed"
        assert "Alice" in resp2.output_text

        # Third response referencing second
        resp3 = client.responses.create(
            model=model,
            input="What is my friend name?",
            previous_response_id=resp2.id,
        )
        assert resp3.error is None
        assert resp3.status == "completed"
        assert "Bob" in resp3.output_text

    def test_conversation_with_multiple_turns(self, setup_backend):
        """Test state management using conversation ID."""
        _, model, client, gateway = setup_backend

        # Create conversation
        conv_resp = client.conversations.create(metadata={"topic": "math"})
        assert conv_resp.id is not None
        assert conv_resp.created_at is not None
        conversation_id = conv_resp.id

        # First response in conversation
        resp1 = client.responses.create(
            model=model, input="I have 5 apples.", conversation=conversation_id
        )
        assert resp1.error is None
        assert resp1.status == "completed"

        # Second response in same conversation
        resp2 = client.responses.create(
            model=model,
            input="How many apples do I have?",
            conversation=conversation_id,
        )
        assert resp2.error is None
        assert resp2.status == "completed"
        assert "5" in resp2.output_text or "five" in resp2.output_text.lower()

        # Third response in same conversation
        resp3 = client.responses.create(
            model=model,
            input="If I get 3 more, how many total?",
            conversation=conversation_id,
        )
        assert resp3.error is None
        assert resp3.status == "completed"
        assert "8" in resp3.output_text or "eight" in resp3.output_text.lower()

        items = client.conversations.items.list(conversation_id)
        assert items.data is not None
        assert len(items.data) >= 6  # 3 inputs + 3 outputs

    @pytest.mark.skip(reason="TODO: Add the invalid previous_response_id check")
    def test_previous_response_id_invalid(self, setup_backend):
        """Test using invalid previous_response_id."""
        _, model, client, gateway = setup_backend
        with pytest.raises(openai.BadRequestError):
            client.responses.create(
                model=model,
                input="Test",
                previous_response_id="resp_invalid123",
                max_output_tokens=50,
            )

    def test_mutually_exclusive_parameters(self, setup_backend):
        """Test that previous_response_id and conversation are mutually exclusive."""
        _, model, client, gateway = setup_backend

        conversation_id = "conv_123"
        resp1 = client.responses.create(model=model, input="Test")

        with pytest.raises(openai.BadRequestError):
            client.responses.create(
                model=model,
                input="This should fail",
                previous_response_id=resp1.id,
                conversation=conversation_id,
            )
```
**EN:** This test class groups related scenarios around `TestStateManagementCloud` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestStateManagementCloud` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

### Lines 152-250: Test class `TestStateManagementLocal`
```python


# =============================================================================
# Local Backend Tests (gRPC with Qwen model)
# =============================================================================


@pytest.mark.e2e
@pytest.mark.model("qwen-14b")
@pytest.mark.gateway(
    extra_args=["--tool-call-parser", "qwen", "--history-backend", "memory"]
)
@pytest.mark.parametrize("setup_backend", ["grpc"], indirect=True)
class TestStateManagementLocal:
    """State management tests against local gRPC backend."""

    @pytest.mark.skip(reason="TODO: Add the invalid previous_response_id check")
    def test_previous_response_id_invalid(self, setup_backend):
        """Test using invalid previous_response_id."""
        _, model, client, gateway = setup_backend
        with pytest.raises(openai.BadRequestError):
            client.responses.create(
                model=model,
                input="Test",
                previous_response_id="resp_invalid123",
                max_output_tokens=50,
            )

    def test_basic_response_creation(self, setup_backend):
        """Test basic response creation without state."""
        _, model, client, gateway = setup_backend

        resp = client.responses.create(model=model, input="What is 2+2?")

        assert resp.id is not None
        assert resp.error is None
        assert resp.status == "completed"
        assert len(resp.output_text) > 0
        assert resp.usage is not None

    def test_streaming_response(self, setup_backend):
        """Test streaming response."""
        _, model, client, gateway = setup_backend

        resp = client.responses.create(
            model=model, input="Count to 5", stream=True, max_output_tokens=50
        )

        events = list(resp)
        created_events = [e for e in events if e.type == "response.created"]
        assert len(created_events) > 0

        assert any(
            e.type in ["response.completed", "response.in_progress"] for e in events
        )

    def test_previous_response_id_chaining(self, setup_backend):
        """Test chaining responses using previous_response_id."""
        _, model, client, gateway = setup_backend

        # First response
        resp1 = client.responses.create(
            model=model, input="My name is Alice and my friend is Bob. Remember it."
        )
        assert resp1.error is None
        assert resp1.status == "completed"

        # Second response referencing first
        resp2 = client.responses.create(
            model=model, input="What is my name", previous_response_id=resp1.id
        )
        assert resp2.error is None
        assert resp2.status == "completed"
        assert "Alice" in resp2.output_text

        # Third response referencing second
        resp3 = client.responses.create(
            model=model,
            input="What is my friend name?",
            previous_response_id=resp2.id,
        )
        assert resp3.error is None
        assert resp3.status == "completed"
        assert "Bob" in resp3.output_text

    def test_mutually_exclusive_parameters(self, setup_backend):
        """Test that previous_response_id and conversation are mutually exclusive."""
        _, model, client, gateway = setup_backend

        conversation_id = "conv_123"
        resp1 = client.responses.create(model=model, input="Test")

        with pytest.raises(openai.BadRequestError):
            client.responses.create(
                model=model,
                input="This should fail",
                previous_response_id=resp1.id,
                conversation=conversation_id,
            )
```
**EN:** This test class groups related scenarios around `TestStateManagementLocal` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestStateManagementLocal` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

### Lines 251-349: Test class `TestStateManagementHarmony`
```python


# =============================================================================
# Local Backend Tests (gRPC with Harmony/Reasoning model)
# =============================================================================


@pytest.mark.e2e
@pytest.mark.model("gpt-oss")
@pytest.mark.gateway(
    extra_args=["--reasoning-parser=gpt-oss", "--history-backend", "memory"]
)
@pytest.mark.parametrize("setup_backend", ["grpc"], indirect=True)
class TestStateManagementHarmony:
    """State management tests against local gRPC backend with Harmony model."""

    @pytest.mark.skip(reason="TODO: Add the invalid previous_response_id check")
    def test_previous_response_id_invalid(self, setup_backend):
        """Test using invalid previous_response_id."""
        _, model, client, gateway = setup_backend
        with pytest.raises(openai.BadRequestError):
            client.responses.create(
                model=model,
                input="Test",
                previous_response_id="resp_invalid123",
                max_output_tokens=50,
            )

    def test_basic_response_creation(self, setup_backend):
        """Test basic response creation without state."""
        _, model, client, gateway = setup_backend

        resp = client.responses.create(model=model, input="What is 2+2?")

        assert resp.id is not None
        assert resp.error is None
        assert resp.status == "completed"
        assert len(resp.output_text) > 0
        assert resp.usage is not None

    def test_streaming_response(self, setup_backend):
        """Test streaming response."""
        _, model, client, gateway = setup_backend

        resp = client.responses.create(
            model=model, input="Count to 5", stream=True, max_output_tokens=50
        )

        events = list(resp)
        created_events = [e for e in events if e.type == "response.created"]
        assert len(created_events) > 0

        assert any(
            e.type in ["response.completed", "response.in_progress"] for e in events
        )

    def test_previous_response_id_chaining(self, setup_backend):
        """Test chaining responses using previous_response_id."""
        _, model, client, gateway = setup_backend

        # First response
        resp1 = client.responses.create(
            model=model, input="My name is Alice and my friend is Bob. Remember it."
        )
        assert resp1.error is None
        assert resp1.status == "completed"

        # Second response referencing first
        resp2 = client.responses.create(
            model=model, input="What is my name", previous_response_id=resp1.id
        )
        assert resp2.error is None
        assert resp2.status == "completed"
        assert "Alice" in resp2.output_text

        # Third response referencing second
        resp3 = client.responses.create(
            model=model,
            input="What is my friend name?",
            previous_response_id=resp2.id,
        )
        assert resp3.error is None
        assert resp3.status == "completed"
        assert "Bob" in resp3.output_text

    def test_mutually_exclusive_parameters(self, setup_backend):
        """Test that previous_response_id and conversation are mutually exclusive."""
        _, model, client, gateway = setup_backend

        conversation_id = "conv_123"
        resp1 = client.responses.create(model=model, input="Test")

        with pytest.raises(openai.BadRequestError):
            client.responses.create(
                model=model,
                input="This should fail",
                previous_response_id=resp1.id,
                conversation=conversation_id,
            )
```
**EN:** This test class groups related scenarios around `TestStateManagementHarmony` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestStateManagementHarmony` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Responses API state, tools, and streaming flows / Responses API 的状态、工具与流式流程
- Primary classes: `TestStateManagementCloud`, `TestStateManagementLocal`, `TestStateManagementHarmony` / 主要类：`TestStateManagementCloud`, `TestStateManagementLocal`, `TestStateManagementHarmony`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`
- **Third-party / 第三方**: `openai`, `pytest`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
