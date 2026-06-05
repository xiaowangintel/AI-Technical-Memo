# test_basic_crud.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/responses/test_basic_crud.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises basic crud behavior in the responses API end-to-end suite. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 Responses API 端到端测试 中与 basic crud 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module docstring
```python
"""Basic CRUD tests for Response API.

Tests for Response and Conversation CRUD operations against cloud backends.

Source: Migrated from e2e_response_api/features/test_basic_crud.py
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 8-16: Imports and dependencies
```python
from __future__ import annotations

import logging
import time

import openai
import pytest
from openai import OpenAI
from openai.types import responses
```
**EN:** This block imports `__future__`, `logging`, `time`, `openai`, and 3 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 18-18: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 21-57: Helper function `wait_for_background_task`
```python
def wait_for_background_task(
    client: OpenAI, response_id: str, timeout: int = 30, poll_interval: float = 0.5
) -> responses.Response:
    """Wait for background task to complete.

    Args:
        client: OpenAI client
        response_id: Response ID to poll
        timeout: Max seconds to wait
        poll_interval: Seconds between polls

    Returns:
        Final response data

    Raises:
        TimeoutError: If task doesn't complete in time
        AssertionError: If task fails
    """
    start_time = time.time()

    while time.time() - start_time < timeout:
        resp = client.responses.retrieve(response_id=response_id)
        assert resp.error is None
        assert resp.id == response_id

        if resp.status == "completed":
            return resp
        elif resp.status == "failed":
            raise AssertionError(f"Background task failed: {resp.error}")
        elif resp.status == "cancelled":
            raise AssertionError("Background task was cancelled")

        time.sleep(poll_interval)

    raise TimeoutError(
        f"Background task {response_id} did not complete within {timeout}s"
    )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 58-132: Test class `TestResponseCRUD`
```python


# =============================================================================
# Response CRUD Tests (Memory Storage)
# =============================================================================


@pytest.mark.parametrize("setup_backend", ["openai"], indirect=True)
class TestResponseCRUD:
    """Tests for Response API CRUD operations."""

    def test_create_and_get_response(self, setup_backend):
        """Test creating response and retrieving it."""
        _, model, client, gateway = setup_backend

        # Create response
        create_resp = client.responses.create(model=model, input="Hello, world!")
        assert create_resp.id is not None
        assert create_resp.error is None
        assert create_resp.status == "completed"
        assert len(create_resp.output_text) > 0
        response_id = create_resp.id

        # Get response
        get_resp = client.responses.retrieve(response_id=response_id)
        assert get_resp.error is None
        assert get_resp.id == response_id
        assert get_resp.status == "completed"

        input_resp = client.responses.input_items.list(response_id=get_resp.id)
        assert input_resp.data is not None
        assert len(input_resp.data) > 0

    @pytest.mark.skip(reason="TODO: Add delete response feature")
    def test_delete_response(self, setup_backend):
        """Test deleting response."""
        _, model, client, gateway = setup_backend

        # Create response
        create_resp = client.responses.create(model=model, input="Test deletion")
        assert create_resp.id is not None
        assert create_resp.error is None
        assert create_resp.status == "completed"
        assert len(create_resp.output_text) > 0

        response_id = create_resp.id

        # Delete response
        client.responses.delete(response_id=response_id)

        # Verify it's deleted (should return 404)
        with pytest.raises(openai.NotFoundError):
            client.responses.retrieve(response_id=response_id)

    @pytest.mark.skip(reason="TODO: Add background response feature")
    def test_background_response(self, setup_backend):
        """Test background response execution."""
        _, model, client, gateway = setup_backend

        # Create background response
        create_resp = client.responses.create(
            model=model,
            input="Write a short story",
            background=True,
            max_output_tokens=100,
        )
        assert create_resp.id is not None
        assert create_resp.error is None
        assert create_resp.status in ["in_progress", "queued"]

        response_id = create_resp.id

        # Wait for completion
        final_data = wait_for_background_task(client, response_id, timeout=60)
        assert final_data.status == "completed"
```
**EN:** This test class groups related scenarios around `TestResponseCRUD` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestResponseCRUD` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

### Lines 133-208: Test class `TestResponseCRUDOracleStorage`
```python


# =============================================================================
# Response CRUD Tests (Oracle Storage)
# =============================================================================


@pytest.mark.storage("oracle")
@pytest.mark.parametrize("setup_backend", ["openai"], indirect=True)
class TestResponseCRUDOracleStorage:
    """Tests for Response API CRUD operations with Oracle history backend."""

    def test_create_and_get_response(self, setup_backend):
        """Test creating response and retrieving it."""
        _, model, client, gateway = setup_backend

        # Create response
        create_resp = client.responses.create(model=model, input="Hello, world!")
        assert create_resp.id is not None
        assert create_resp.error is None
        assert create_resp.status == "completed"
        assert len(create_resp.output_text) > 0
        response_id = create_resp.id

        # Get response
        get_resp = client.responses.retrieve(response_id=response_id)
        assert get_resp.error is None
        assert get_resp.id == response_id
        assert get_resp.status == "completed"

        input_resp = client.responses.input_items.list(response_id=get_resp.id)
        assert input_resp.data is not None
        assert len(input_resp.data) > 0

    @pytest.mark.skip(reason="TODO: Add delete response feature")
    def test_delete_response(self, setup_backend):
        """Test deleting response."""
        _, model, client, gateway = setup_backend

        # Create response
        create_resp = client.responses.create(model=model, input="Test deletion")
        assert create_resp.id is not None
        assert create_resp.error is None
        assert create_resp.status == "completed"
        assert len(create_resp.output_text) > 0

        response_id = create_resp.id

        # Delete response
        client.responses.delete(response_id=response_id)

        # Verify it's deleted (should return 404)
        with pytest.raises(openai.NotFoundError):
            client.responses.retrieve(response_id=response_id)

    @pytest.mark.skip(reason="TODO: Add background response feature")
    def test_background_response(self, setup_backend):
        """Test background response execution."""
        _, model, client, gateway = setup_backend

        # Create background response
        create_resp = client.responses.create(
            model=model,
            input="Write a short story",
            background=True,
            max_output_tokens=100,
        )
        assert create_resp.id is not None
        assert create_resp.error is None
        assert create_resp.status in ["in_progress", "queued"]

        response_id = create_resp.id

        # Wait for completion
        final_data = wait_for_background_task(client, response_id, timeout=60)
        assert final_data.status == "completed"
```
**EN:** This test class groups related scenarios around `TestResponseCRUDOracleStorage` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestResponseCRUDOracleStorage` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

### Lines 209-324: Test class `TestConversationCRUD`
```python


# =============================================================================
# Conversation CRUD Tests (Memory Storage)
# =============================================================================


@pytest.mark.parametrize("setup_backend", ["openai"], indirect=True)
class TestConversationCRUD:
    """Tests for Conversation API CRUD operations."""

    def test_create_and_get_conversation(self, setup_backend):
        """Test creating and retrieving conversation."""
        _, model, client, gateway = setup_backend

        # Create conversation
        create_resp = client.conversations.create(metadata={"user": "test_user"})
        assert create_resp.id is not None
        assert create_resp.created_at is not None

        create_data = create_resp.metadata
        assert create_data["user"] == "test_user"
        conversation_id = create_resp.id

        # Get conversation
        get_resp = client.conversations.retrieve(conversation_id=conversation_id)
        assert get_resp.id is not None
        assert get_resp.created_at is not None

        get_data = get_resp.metadata
        assert get_resp.id == conversation_id
        assert get_data["user"] == "test_user"

    def test_update_conversation(self, setup_backend):
        """Test updating conversation metadata."""
        _, model, client, gateway = setup_backend

        # Create conversation
        create_resp = client.conversations.create(metadata={"key1": "value1"})
        assert create_resp.id is not None
        assert create_resp.created_at is not None

        create_data = create_resp.metadata
        assert create_data["key1"] == "value1"
        assert "key2" not in create_data
        conversation_id = create_resp.id

        # Update conversation
        update_resp = client.conversations.update(
            conversation_id=conversation_id,
            metadata={"key1": "value1", "key2": "value2"},
        )
        assert update_resp.id == conversation_id
        update_data = update_resp.metadata
        assert update_data["key1"] == "value1"
        assert update_data["key2"] == "value2"

        # Verify update
        get_resp = client.conversations.retrieve(conversation_id=conversation_id)
        get_data = get_resp.metadata
        assert get_data["key1"] == "value1"
        assert get_data["key2"] == "value2"

    def test_delete_conversation(self, setup_backend):
        """Test deleting conversation."""
        _, model, client, gateway = setup_backend

        # Create conversation
        create_resp = client.conversations.create()
        assert create_resp.id is not None
        assert create_resp.created_at is not None
        conversation_id = create_resp.id

        # Delete conversation
        delete_resp = client.conversations.delete(conversation_id=conversation_id)
        assert delete_resp.id is not None
        assert delete_resp.deleted

        # Verify deletion
        with pytest.raises(openai.NotFoundError):
            client.conversations.retrieve(conversation_id=conversation_id)

    def test_list_conversation_items(self, setup_backend):
        """Test listing conversation items."""
        _, model, client, gateway = setup_backend

        # Create conversation
        conv_resp = client.conversations.create()
        assert conv_resp.id is not None
        conversation_id = conv_resp.id

        # Create response with conversation
        resp1 = client.responses.create(
            model=model,
            input="First message",
            conversation=conversation_id,
            max_output_tokens=50,
        )
        assert resp1.error is None

        resp2 = client.responses.create(
            model=model,
            input="Second message",
            conversation=conversation_id,
            max_output_tokens=50,
        )
        assert resp2.error is None

        # List items
        list_resp = client.conversations.items.list(conversation_id=conversation_id)
        assert list_resp is not None
        assert list_resp.data is not None

        list_data = list_resp.data
        # Should have at least 4 items (2 inputs + 2 outputs)
        assert len(list_data) >= 4
```
**EN:** This test class groups related scenarios around `TestConversationCRUD` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestConversationCRUD` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

### Lines 325-441: Test class `TestConversationCRUDOracleStorage`
```python


# =============================================================================
# Conversation CRUD Tests (Oracle Storage)
# =============================================================================


@pytest.mark.storage("oracle")
@pytest.mark.parametrize("setup_backend", ["openai"], indirect=True)
class TestConversationCRUDOracleStorage:
    """Tests for Conversation API CRUD operations with Oracle history backend."""

    def test_create_and_get_conversation(self, setup_backend):
        """Test creating and retrieving conversation."""
        _, model, client, gateway = setup_backend

        # Create conversation
        create_resp = client.conversations.create(metadata={"user": "test_user"})
        assert create_resp.id is not None
        assert create_resp.created_at is not None

        create_data = create_resp.metadata
        assert create_data["user"] == "test_user"
        conversation_id = create_resp.id

        # Get conversation
        get_resp = client.conversations.retrieve(conversation_id=conversation_id)
        assert get_resp.id is not None
        assert get_resp.created_at is not None

        get_data = get_resp.metadata
        assert get_resp.id == conversation_id
        assert get_data["user"] == "test_user"

    def test_update_conversation(self, setup_backend):
        """Test updating conversation metadata."""
        _, model, client, gateway = setup_backend

        # Create conversation
        create_resp = client.conversations.create(metadata={"key1": "value1"})
        assert create_resp.id is not None
        assert create_resp.created_at is not None

        create_data = create_resp.metadata
        assert create_data["key1"] == "value1"
        assert "key2" not in create_data
        conversation_id = create_resp.id

        # Update conversation
        update_resp = client.conversations.update(
            conversation_id=conversation_id,
            metadata={"key1": "value1", "key2": "value2"},
        )
        assert update_resp.id == conversation_id
        update_data = update_resp.metadata
        assert update_data["key1"] == "value1"
        assert update_data["key2"] == "value2"

        # Verify update
        get_resp = client.conversations.retrieve(conversation_id=conversation_id)
        get_data = get_resp.metadata
        assert get_data["key1"] == "value1"
        assert get_data["key2"] == "value2"

    def test_delete_conversation(self, setup_backend):
        """Test deleting conversation."""
        _, model, client, gateway = setup_backend

        # Create conversation
        create_resp = client.conversations.create()
        assert create_resp.id is not None
        assert create_resp.created_at is not None
        conversation_id = create_resp.id

        # Delete conversation
        delete_resp = client.conversations.delete(conversation_id=conversation_id)
        assert delete_resp.id is not None
        assert delete_resp.deleted

        # Verify deletion
        with pytest.raises(openai.NotFoundError):
            client.conversations.retrieve(conversation_id=conversation_id)

    def test_list_conversation_items(self, setup_backend):
        """Test listing conversation items."""
        _, model, client, gateway = setup_backend

        # Create conversation
        conv_resp = client.conversations.create()
        assert conv_resp.id is not None
        conversation_id = conv_resp.id

        # Create response with conversation
        resp1 = client.responses.create(
            model=model,
            input="First message",
            conversation=conversation_id,
            max_output_tokens=50,
        )
        assert resp1.error is None

        resp2 = client.responses.create(
            model=model,
            input="Second message",
            conversation=conversation_id,
            max_output_tokens=50,
        )
        assert resp2.error is None

        # List items
        list_resp = client.conversations.items.list(conversation_id=conversation_id)
        assert list_resp is not None
        assert list_resp.data is not None

        list_data = list_resp.data
        # Should have at least 4 items (2 inputs + 2 outputs)
        assert len(list_data) >= 4
```
**EN:** This test class groups related scenarios around `TestConversationCRUDOracleStorage` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestConversationCRUDOracleStorage` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Responses API state, tools, and streaming flows / Responses API 的状态、工具与流式流程
- Primary classes: `TestResponseCRUD`, `TestResponseCRUDOracleStorage`, `TestConversationCRUD`, `TestConversationCRUDOracleStorage` / 主要类：`TestResponseCRUD`, `TestResponseCRUDOracleStorage`, `TestConversationCRUD`, `TestConversationCRUDOracleStorage`
- Reusable functions: `wait_for_background_task` / 可复用函数：`wait_for_background_task`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `time`
- **Third-party / 第三方**: `openai`, `openai.types`, `pytest`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
