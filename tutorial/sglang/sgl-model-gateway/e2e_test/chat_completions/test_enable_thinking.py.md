# test_enable_thinking.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/chat_completions/test_enable_thinking.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises enable thinking behavior in the chat completions end-to-end suite. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 Chat Completions 端到端测试 中与 enable thinking 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module docstring
```python
"""Enable Thinking E2E Tests.

Tests for chat completions with enable_thinking feature (Qwen3 reasoning).

Source: Migrated from e2e_grpc/features/test_enable_thinking.py
"""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 8-14: Imports and dependencies
```python
from __future__ import annotations

import json
import logging

import pytest
import requests
```
**EN:** This block imports `__future__`, `json`, `logging`, `pytest`, and 1 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 16-19: Module constants and configuration
```python
logger = logging.getLogger(__name__)

# API key is not validated by the gateway, but required for OpenAI-compatible headers
API_KEY = "not-used"
```
**EN:** This section defines module-level names such as `logger`, `API_KEY`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 20-168: Test class `TestEnableThinking`
```python


# =============================================================================
# Enable Thinking Tests (Qwen 30B)
# =============================================================================


@pytest.mark.model("qwen-30b")
@pytest.mark.gateway(
    extra_args=["--reasoning-parser", "qwen3", "--history-backend", "memory"]
)
@pytest.mark.parametrize("setup_backend", ["grpc"], indirect=True)
class TestEnableThinking:
    """Tests for enable_thinking feature with Qwen3 reasoning parser."""

    def test_chat_completion_with_reasoning(self, setup_backend):
        """Test non-streaming with enable_thinking=True, reasoning_content should not be empty."""
        _, model, client, gateway = setup_backend

        response = requests.post(
            f"{gateway.base_url}/v1/chat/completions",
            headers={"Authorization": f"Bearer {API_KEY}"},
            json={
                "model": model,
                "messages": [{"role": "user", "content": "Hello"}],
                "temperature": 0,
                "separate_reasoning": True,
                "chat_template_kwargs": {"enable_thinking": True},
            },
        )

        assert response.status_code == 200, f"Failed with: {response.text}"
        data = response.json()

        assert "choices" in data
        assert len(data["choices"]) > 0
        assert "message" in data["choices"][0]
        assert "reasoning_content" in data["choices"][0]["message"]
        assert data["choices"][0]["message"]["reasoning_content"] is not None

    def test_chat_completion_without_reasoning(self, setup_backend):
        """Test non-streaming with enable_thinking=False, reasoning_content should be empty."""
        _, model, client, gateway = setup_backend

        response = requests.post(
            f"{gateway.base_url}/v1/chat/completions",
            headers={"Authorization": f"Bearer {API_KEY}"},
            json={
                "model": model,
                "messages": [{"role": "user", "content": "Hello"}],
                "temperature": 0,
                "separate_reasoning": True,
                "chat_template_kwargs": {"enable_thinking": False},
            },
        )

        assert response.status_code == 200, f"Failed with: {response.text}"
        data = response.json()

        assert "choices" in data
        assert len(data["choices"]) > 0
        assert "message" in data["choices"][0]

        if "reasoning_content" in data["choices"][0]["message"]:
            assert data["choices"][0]["message"]["reasoning_content"] is None

    def test_stream_chat_completion_with_reasoning(self, setup_backend):
        """Test streaming with enable_thinking=True, reasoning_content should not be empty."""
        _, model, client, gateway = setup_backend

        response = requests.post(
            f"{gateway.base_url}/v1/chat/completions",
            headers={"Authorization": f"Bearer {API_KEY}"},
            json={
                "model": model,
                "messages": [{"role": "user", "content": "Hello"}],
                "temperature": 0,
                "separate_reasoning": True,
                "stream": True,
                "chat_template_kwargs": {"enable_thinking": True},
            },
            stream=True,
        )

        assert response.status_code == 200, f"Failed with: {response.text}"

        has_reasoning = False
        has_content = False

        for line in response.iter_lines():
            if line:
                line = line.decode("utf-8")
                if line.startswith("data:") and not line.startswith("data: [DONE]"):
                    data = json.loads(line[6:])
                    if "choices" in data and len(data["choices"]) > 0:
                        delta = data["choices"][0].get("delta", {})

                        if "reasoning_content" in delta and delta["reasoning_content"]:
                            has_reasoning = True

                        if "content" in delta and delta["content"]:
                            has_content = True

        assert (
            has_reasoning
        ), "The reasoning content is not included in the stream response"
        assert has_content, "The stream response does not contain normal content"

    def test_stream_chat_completion_without_reasoning(self, setup_backend):
        """Test streaming with enable_thinking=False, reasoning_content should be empty."""
        _, model, client, gateway = setup_backend

        response = requests.post(
            f"{gateway.base_url}/v1/chat/completions",
            headers={"Authorization": f"Bearer {API_KEY}"},
            json={
                "model": model,
                "messages": [{"role": "user", "content": "Hello"}],
                "temperature": 0,
                "separate_reasoning": True,
                "stream": True,
                "chat_template_kwargs": {"enable_thinking": False},
            },
            stream=True,
        )

        assert response.status_code == 200, f"Failed with: {response.text}"

        has_reasoning = False
        has_content = False

        for line in response.iter_lines():
            if line:
                line = line.decode("utf-8")
                if line.startswith("data:") and not line.startswith("data: [DONE]"):
                    data = json.loads(line[6:])
                    if "choices" in data and len(data["choices"]) > 0:
                        delta = data["choices"][0].get("delta", {})

                        if "reasoning_content" in delta and delta["reasoning_content"]:
                            has_reasoning = True

                        if "content" in delta and delta["content"]:
                            has_content = True

        assert (
            not has_reasoning
        ), "The reasoning content should not be included in the stream response"
        assert has_content, "The stream response does not contain normal content"
```
**EN:** This test class groups related scenarios around `TestEnableThinking` and organizes shared assertions or helpers in one place.
**CN:** 这个测试类围绕 `TestEnableThinking` 组织相关场景，并将共享断言或辅助逻辑集中在同一位置。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- OpenAI-compatible chat completions behavior / 兼容 OpenAI 的聊天补全行为
- Primary classes: `TestEnableThinking` / 主要类：`TestEnableThinking`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `json`, `logging`
- **Third-party / 第三方**: `pytest`, `requests`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
