# test_anthropic_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/openai_server/basic/test_anthropic_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates anthropic server behavior in SGLang's openai server / basic area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 openai server / basic 领域中与 anthropic server 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: supporting statements / 辅助语句
```python
"""
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_simple_messages
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_simple_messages_stream
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_multi_turn_messages
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_system_message_string
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_system_message_blocks
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_max_tokens
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_temperature
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_stop_sequences
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_error_invalid_max_tokens
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_error_empty_messages
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_raw_http_non_streaming
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_raw_http_streaming
python3 -m unittest openai_server.basic.test_anthropic_server.TestAnthropicServer.test_tool_result_image_content_conversion
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 17-32: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest

import requests

from sglang.srt.entrypoints.anthropic.protocol import AnthropicMessagesRequest
from sglang.srt.entrypoints.anthropic.serving import AnthropicServing
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `requests`, `sglang.srt.entrypoints.anthropic.protocol`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `requests`, `sglang.srt.entrypoints.anthropic.protocol`。

### Lines 34-35: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=40, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=140, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 38-38: class TestAnthropicServer declaration / 类 TestAnthropicServer 声明
```python
class TestAnthropicServer(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 39-50: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.api_key = "sk-123456"
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            api_key=cls.api_key,
        )
        cls.messages_url = cls.base_url + "/v1/messages"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 52-54: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 56-67: method make request / 方法 make request
```python
    def _make_request(self, payload, stream=False):
        """Send a request to the /v1/messages endpoint."""
        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {self.api_key}",
        }
        return requests.post(
            self.messages_url,
            headers=headers,
            json=payload,
            stream=stream,
        )
```
**EN:** Send a request to the /v1/messages endpoint. This block implements `_make_request` and captures one focused piece of the module's behavior.
**CN:** Send a request to the /v1/messages endpoint. 该代码块实现 `_make_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 69-82: method default payload / 方法 default payload
```python
    def _default_payload(self, **overrides):
        """Build a default Anthropic Messages request payload."""
        payload = {
            "model": self.model,
            "max_tokens": 64,
            "messages": [
                {
                    "role": "user",
                    "content": "What is the capital of France? Answer in a few words.",
                }
            ],
        }
        payload.update(overrides)
        return payload
```
**EN:** Build a default Anthropic Messages request payload. This block implements `_default_payload` and captures one focused piece of the module's behavior.
**CN:** Build a default Anthropic Messages request payload. 该代码块实现 `_default_payload`，承担模块行为中的一个聚焦逻辑片段。

### Lines 83-85: supporting source context / 辅助源码上下文
```python

    # ---- Non-streaming tests ----

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 86-148: test case tool result image content conversion / 测试用例 tool result image content conversion
```python
    def test_tool_result_image_content_conversion(self):
        """Tool-result image blocks should be preserved as OpenAI image_url content."""
        anthropic_request = AnthropicMessagesRequest(
            model=self.model,
            max_tokens=64,
            messages=[
                {
                    "role": "user",
                    "content": "I have called read_file to get an image. What color is it?",
                },
                {
                    "role": "assistant",
                    "content": [
                        {
                            "type": "tool_use",
                            "id": "call_123",
                            "name": "read_file",
                            "input": {"file_path": "/test.png"},
                        }
                    ],
                },
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "tool_result",
                            "tool_use_id": "call_123",
                            "content": [
                                {
                                    "type": "image",
                                    "source": {
                                        "type": "base64",
                                        "media_type": "image/png",
                                        "data": "abcd",
                                    },
                                }
                            ],
                        }
                    ],
                },
            ],
        )

        serving = AnthropicServing(openai_serving_chat=object())
        chat_request = serving._convert_to_chat_completion_request(anthropic_request)
        converted = chat_request.model_dump()

        tool_messages = [m for m in converted["messages"] if m.get("role") == "tool"]
        self.assertEqual(
            len(tool_messages),
            1,
            f"Expected one tool message, got: {converted['messages']}",
        )

        tool_message = tool_messages[0]
        self.assertEqual(tool_message["tool_call_id"], "call_123")
        self.assertIsInstance(tool_message["content"], list)
        self.assertEqual(len(tool_message["content"]), 1)
        self.assertEqual(tool_message["content"][0]["type"], "image_url")
        self.assertEqual(
            tool_message["content"][0]["image_url"]["url"],
            "data:image/png;base64,abcd",
        )
```
**EN:** Tool-result image blocks should be preserved as OpenAI image_url content. This test exercises `test_tool_result_image_content_conversion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Tool-result image blocks should be preserved as OpenAI image_url content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_result_image_content_conversion`。

### Lines 150-183: test case simple messages / 测试用例 simple messages
```python
    def test_simple_messages(self):
        """Test basic non-streaming message request."""
        payload = self._default_payload()
        resp = self._make_request(payload)
        self.assertEqual(resp.status_code, 200, f"Response: {resp.text}")

        body = resp.json()
        self.assertEqual(body["type"], "message")
        self.assertEqual(body["role"], "assistant")
        self.assertIn("content", body)
        self.assertIsInstance(body["content"], list)
        self.assertTrue(len(body["content"]) > 0)
        self.assertEqual(body["content"][0]["type"], "text")
        self.assertIsInstance(body["content"][0]["text"], str)
        self.assertTrue(len(body["content"][0]["text"]) > 0)

        # Verify stop reason
        self.assertIn(body["stop_reason"], ["end_turn", "max_tokens", "stop_sequence"])

        # Verify usage
        self.assertIn("usage", body)
        self.assertIsInstance(body["usage"]["input_tokens"], int)
        self.assertIsInstance(body["usage"]["output_tokens"], int)
        self.assertGreater(body["usage"]["input_tokens"], 0)
        self.assertGreater(body["usage"]["output_tokens"], 0)

        # Verify id format (must be msg_*) and model
        self.assertIn("id", body)
        self.assertIsInstance(body["id"], str)
        self.assertTrue(
            body["id"].startswith("msg_"),
            f"ID should start with 'msg_', got: {body['id']}",
        )
        self.assertIn("model", body)
```
**EN:** Test basic non-streaming message request. This test exercises `test_simple_messages` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic non-streaming message request. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_messages`。

### Lines 185-201: test case multi turn messages / 测试用例 multi turn messages
```python
    def test_multi_turn_messages(self):
        """Test multi-turn conversation."""
        payload = self._default_payload(
            messages=[
                {"role": "user", "content": "My name is Alice."},
                {"role": "assistant", "content": "Hello Alice! Nice to meet you."},
                {"role": "user", "content": "What is my name?"},
            ]
        )
        resp = self._make_request(payload)
        self.assertEqual(resp.status_code, 200, f"Response: {resp.text}")

        body = resp.json()
        self.assertEqual(body["type"], "message")
        self.assertTrue(len(body["content"]) > 0)
        self.assertEqual(body["content"][0]["type"], "text")
        self.assertIsInstance(body["content"][0]["text"], str)
```
**EN:** Test multi-turn conversation. This test exercises `test_multi_turn_messages` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test multi-turn conversation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_turn_messages`。

### Lines 203-213: test case system message string / 测试用例 system message string
```python
    def test_system_message_string(self):
        """Test system message as a string."""
        payload = self._default_payload(
            system="You are a helpful assistant. Always respond in French.",
        )
        resp = self._make_request(payload)
        self.assertEqual(resp.status_code, 200, f"Response: {resp.text}")

        body = resp.json()
        self.assertEqual(body["type"], "message")
        self.assertTrue(len(body["content"]) > 0)
```
**EN:** Test system message as a string. This test exercises `test_system_message_string` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test system message as a string. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_system_message_string`。

### Lines 215-228: test case system message blocks / 测试用例 system message blocks
```python
    def test_system_message_blocks(self):
        """Test system message as content blocks."""
        payload = self._default_payload(
            system=[
                {"type": "text", "text": "You are a helpful assistant."},
                {"type": "text", "text": "Always be concise."},
            ],
        )
        resp = self._make_request(payload)
        self.assertEqual(resp.status_code, 200, f"Response: {resp.text}")

        body = resp.json()
        self.assertEqual(body["type"], "message")
        self.assertTrue(len(body["content"]) > 0)
```
**EN:** Test system message as content blocks. This test exercises `test_system_message_blocks` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test system message as content blocks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_system_message_blocks`。

### Lines 230-245: test case max tokens / 测试用例 max tokens
```python
    def test_max_tokens(self):
        """Test max_tokens limits output length."""
        payload = self._default_payload(
            max_tokens=5,
            messages=[
                {"role": "user", "content": "Tell me a long story about a dragon."}
            ],
        )
        resp = self._make_request(payload)
        self.assertEqual(resp.status_code, 200, f"Response: {resp.text}")

        body = resp.json()
        self.assertEqual(body["type"], "message")
        # With very small max_tokens the model should hit the limit
        self.assertIn(body["stop_reason"], ["max_tokens", "end_turn"])
        self.assertGreater(body["usage"]["output_tokens"], 0)
```
**EN:** Test max_tokens limits output length. This test exercises `test_max_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test max_tokens limits output length. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_max_tokens`。

### Lines 247-255: test case temperature / 测试用例 temperature
```python
    def test_temperature(self):
        """Test temperature parameter is accepted."""
        payload = self._default_payload(temperature=0.0)
        resp = self._make_request(payload)
        self.assertEqual(resp.status_code, 200, f"Response: {resp.text}")

        body = resp.json()
        self.assertEqual(body["type"], "message")
        self.assertTrue(len(body["content"]) > 0)
```
**EN:** Test temperature parameter is accepted. This test exercises `test_temperature` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test temperature parameter is accepted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_temperature`。

### Lines 257-267: test case stop sequences / 测试用例 stop sequences
```python
    def test_stop_sequences(self):
        """Test stop_sequences parameter is accepted."""
        payload = self._default_payload(
            stop_sequences=["\n"],
            max_tokens=128,
        )
        resp = self._make_request(payload)
        self.assertEqual(resp.status_code, 200, f"Response: {resp.text}")

        body = resp.json()
        self.assertEqual(body["type"], "message")
```
**EN:** Test stop_sequences parameter is accepted. This test exercises `test_stop_sequences` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test stop_sequences parameter is accepted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stop_sequences`。

### Lines 269-277: test case top p and top k / 测试用例 top p and top k
```python
    def test_top_p_and_top_k(self):
        """Test top_p and top_k parameters."""
        payload = self._default_payload(top_p=0.9, top_k=40)
        resp = self._make_request(payload)
        self.assertEqual(resp.status_code, 200, f"Response: {resp.text}")

        body = resp.json()
        self.assertEqual(body["type"], "message")
        self.assertTrue(len(body["content"]) > 0)
```
**EN:** Test top_p and top_k parameters. This test exercises `test_top_p_and_top_k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test top_p and top_k parameters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_top_p_and_top_k`。

### Lines 278-280: supporting source context / 辅助源码上下文
```python

    # ---- Streaming tests ----

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 281-341: test case simple messages stream / 测试用例 simple messages stream
```python
    def test_simple_messages_stream(self):
        """Test basic streaming message request."""
        payload = self._default_payload(stream=True)
        resp = self._make_request(payload, stream=True)
        self.assertEqual(resp.status_code, 200, f"Status: {resp.status_code}")

        events = self._parse_sse_events(resp)

        # Verify event sequence
        event_types = [e["type"] for e in events]
        self.assertIn("message_start", event_types)
        self.assertIn("message_stop", event_types)

        # Verify message_start
        message_start = next(e for e in events if e["type"] == "message_start")
        self.assertIn("message", message_start)
        self.assertEqual(message_start["message"]["type"], "message")
        self.assertEqual(message_start["message"]["role"], "assistant")
        self.assertIn("usage", message_start["message"])

        # Verify we got content deltas
        content_deltas = [e for e in events if e["type"] == "content_block_delta"]
        self.assertTrue(
            len(content_deltas) > 0, "Expected at least one content_block_delta event"
        )

        # Verify all text deltas have correct structure
        for delta_event in content_deltas:
            self.assertIn("delta", delta_event)
            self.assertEqual(delta_event["delta"]["type"], "text_delta")
            self.assertIn("text", delta_event["delta"])

        # Reconstruct the full text
        full_text = "".join(
            e["delta"]["text"]
            for e in content_deltas
            if e["delta"].get("type") == "text_delta"
        )
        self.assertTrue(len(full_text) > 0, "Reconstructed text should not be empty")

        # Verify content_block_start/stop
        block_starts = [e for e in events if e["type"] == "content_block_start"]
        block_stops = [e for e in events if e["type"] == "content_block_stop"]
        self.assertTrue(len(block_starts) > 0, "Expected content_block_start")
        self.assertTrue(len(block_stops) > 0, "Expected content_block_stop")
        self.assertEqual(block_starts[0]["content_block"]["type"], "text")

        # Verify message_delta with stop_reason
        message_deltas = [e for e in events if e["type"] == "message_delta"]
        self.assertTrue(len(message_deltas) > 0, "Expected message_delta event")
        last_delta = message_deltas[-1]
        self.assertIn("delta", last_delta)
        self.assertIn("stop_reason", last_delta["delta"])
        self.assertIn(
            last_delta["delta"]["stop_reason"],
            ["end_turn", "max_tokens", "stop_sequence", "tool_use"],
        )

        # Verify usage in message_delta
        self.assertIn("usage", last_delta)
        self.assertIsInstance(last_delta["usage"]["output_tokens"], int)
```
**EN:** Test basic streaming message request. This test exercises `test_simple_messages_stream` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic streaming message request. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_messages_stream`。

### Lines 343-359: test case stream multi turn / 测试用例 stream multi turn
```python
    def test_stream_multi_turn(self):
        """Test streaming with multi-turn conversation."""
        payload = self._default_payload(
            stream=True,
            messages=[
                {"role": "user", "content": "Say hello."},
                {"role": "assistant", "content": "Hello!"},
                {"role": "user", "content": "Say goodbye."},
            ],
        )
        resp = self._make_request(payload, stream=True)
        self.assertEqual(resp.status_code, 200)

        events = self._parse_sse_events(resp)
        event_types = [e["type"] for e in events]
        self.assertIn("message_start", event_types)
        self.assertIn("message_stop", event_types)
```
**EN:** Test streaming with multi-turn conversation. This test exercises `test_stream_multi_turn` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with multi-turn conversation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stream_multi_turn`。

### Lines 361-373: test case stream with system / 测试用例 stream with system
```python
    def test_stream_with_system(self):
        """Test streaming with system message."""
        payload = self._default_payload(
            stream=True,
            system="You are a pirate. Respond in pirate speak.",
        )
        resp = self._make_request(payload, stream=True)
        self.assertEqual(resp.status_code, 200)

        events = self._parse_sse_events(resp)
        event_types = [e["type"] for e in events]
        self.assertIn("message_start", event_types)
        self.assertIn("message_stop", event_types)
```
**EN:** Test streaming with system message. This test exercises `test_stream_with_system` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test streaming with system message. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stream_with_system`。

### Lines 374-376: supporting source context / 辅助源码上下文
```python

    # ---- Error handling tests ----

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 377-381: test case error invalid max tokens / 测试用例 error invalid max tokens
```python
    def test_error_invalid_max_tokens(self):
        """Test error response for invalid max_tokens."""
        payload = self._default_payload(max_tokens=-1)
        resp = self._make_request(payload)
        self.assertIn(resp.status_code, [400, 422])
```
**EN:** Test error response for invalid max_tokens. This test exercises `test_error_invalid_max_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test error response for invalid max_tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_error_invalid_max_tokens`。

### Lines 383-387: test case error empty messages / 测试用例 error empty messages
```python
    def test_error_empty_messages(self):
        """Test error response for empty messages list."""
        payload = self._default_payload(messages=[])
        resp = self._make_request(payload)
        self.assertIn(resp.status_code, [400, 422])
```
**EN:** Test error response for empty messages list. This test exercises `test_error_empty_messages` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test error response for empty messages list. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_error_empty_messages`。

### Lines 389-399: test case error missing content type / 测试用例 error missing content type
```python
    def test_error_missing_content_type(self):
        """Test error when Content-Type is not application/json."""
        headers = {
            "Authorization": f"Bearer {self.api_key}",
        }
        resp = requests.post(
            self.messages_url,
            headers=headers,
            data="not json",
        )
        self.assertIn(resp.status_code, [400, 415, 422])
```
**EN:** Test error when Content-Type is not application/json. This test exercises `test_error_missing_content_type` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test error when Content-Type is not application/json. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_error_missing_content_type`。

### Lines 400-402: supporting source context / 辅助源码上下文
```python

    # ---- Raw HTTP tests ----

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 403-419: test case raw http non streaming / 测试用例 raw http non streaming
```python
    def test_raw_http_non_streaming(self):
        """Test raw HTTP request/response format for non-streaming."""
        payload = self._default_payload(temperature=0)
        resp = self._make_request(payload)
        self.assertEqual(resp.status_code, 200)

        # Verify response content type
        self.assertIn("application/json", resp.headers.get("content-type", ""))

        body = resp.json()
        # Verify all required fields per Anthropic spec
        required_fields = ["id", "type", "role", "content", "model", "usage"]
        for field in required_fields:
            self.assertIn(field, body, f"Missing required field: {field}")

        self.assertEqual(body["type"], "message")
        self.assertEqual(body["role"], "assistant")
```
**EN:** Test raw HTTP request/response format for non-streaming. This test exercises `test_raw_http_non_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test raw HTTP request/response format for non-streaming. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_raw_http_non_streaming`。

### Lines 421-445: test case raw http streaming / 测试用例 raw http streaming
```python
    def test_raw_http_streaming(self):
        """Test raw HTTP request/response format for streaming."""
        payload = self._default_payload(stream=True, temperature=0)
        resp = self._make_request(payload, stream=True)
        self.assertEqual(resp.status_code, 200)

        # Verify streaming content type
        self.assertIn("text/event-stream", resp.headers.get("content-type", ""))

        # Verify we get proper SSE events
        events = self._parse_sse_events(resp)
        self.assertTrue(len(events) > 0, "Expected at least some SSE events")

        # Verify event ordering: message_start should be first
        self.assertEqual(
            events[0]["type"], "message_start", "First event should be message_start"
        )

        # Verify message_stop is last data event
        data_events = [e for e in events if e["type"] != "ping"]
        self.assertEqual(
            data_events[-1]["type"],
            "message_stop",
            "Last data event should be message_stop",
        )
```
**EN:** Test raw HTTP request/response format for streaming. This test exercises `test_raw_http_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test raw HTTP request/response format for streaming. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_raw_http_streaming`。

### Lines 446-448: supporting source context / 辅助源码上下文
```python

    # ---- Content block tests ----

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 449-467: test case content blocks message / 测试用例 content blocks message
```python
    def test_content_blocks_message(self):
        """Test sending messages with explicit content blocks."""
        payload = self._default_payload(
            messages=[
                {
                    "role": "user",
                    "content": [
                        {"type": "text", "text": "What is 2+2?"},
                    ],
                }
            ],
        )
        resp = self._make_request(payload)
        self.assertEqual(resp.status_code, 200, f"Response: {resp.text}")

        body = resp.json()
        self.assertEqual(body["type"], "message")
        self.assertTrue(len(body["content"]) > 0)
        self.assertEqual(body["content"][0]["type"], "text")
```
**EN:** Test sending messages with explicit content blocks. This test exercises `test_content_blocks_message` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test sending messages with explicit content blocks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_content_blocks_message`。

### Lines 468-470: supporting source context / 辅助源码上下文
```python

    # ---- Count tokens tests ----

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 471-493: test case count tokens / 测试用例 count tokens
```python
    def test_count_tokens(self):
        """Test /v1/messages/count_tokens endpoint."""
        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {self.api_key}",
        }
        payload = {
            "model": self.model,
            "messages": [
                {"role": "user", "content": "Hello, how are you?"},
            ],
        }
        resp = requests.post(
            self.base_url + "/v1/messages/count_tokens",
            headers=headers,
            json=payload,
        )
        self.assertEqual(resp.status_code, 200, f"Response: {resp.text}")

        body = resp.json()
        self.assertIn("input_tokens", body)
        self.assertIsInstance(body["input_tokens"], int)
        self.assertGreater(body["input_tokens"], 0)
```
**EN:** Test /v1/messages/count_tokens endpoint. This test exercises `test_count_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test /v1/messages/count_tokens endpoint. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_count_tokens`。

### Lines 495-534: test case count tokens with system / 测试用例 count tokens with system
```python
    def test_count_tokens_with_system(self):
        """Test count_tokens with system message."""
        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {self.api_key}",
        }
        payload_no_system = {
            "model": self.model,
            "messages": [
                {"role": "user", "content": "Hello"},
            ],
        }
        payload_with_system = {
            "model": self.model,
            "messages": [
                {"role": "user", "content": "Hello"},
            ],
            "system": "You are a helpful assistant with a very long system prompt that adds tokens.",
        }
        resp1 = requests.post(
            self.base_url + "/v1/messages/count_tokens",
            headers=headers,
            json=payload_no_system,
        )
        resp2 = requests.post(
            self.base_url + "/v1/messages/count_tokens",
            headers=headers,
            json=payload_with_system,
        )
        self.assertEqual(resp1.status_code, 200)
        self.assertEqual(resp2.status_code, 200)

        # System message should increase the token count
        tokens_no_system = resp1.json()["input_tokens"]
        tokens_with_system = resp2.json()["input_tokens"]
        self.assertGreater(
            tokens_with_system,
            tokens_no_system,
            "Adding system message should increase token count",
        )
```
**EN:** Test count_tokens with system message. This test exercises `test_count_tokens_with_system` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test count_tokens with system message. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_count_tokens_with_system`。

### Lines 535-537: supporting source context / 辅助源码上下文
```python

    # ---- Helpers ----

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 538-556: method parse sse events / 方法 parse sse events
```python
    def _parse_sse_events(self, response):
        """Parse SSE events from a streaming response."""
        events = []

        for line in response.iter_lines(decode_unicode=True):
            if not line:
                continue

            if line.startswith("data: "):
                data_str = line[6:].strip()
                if data_str == "[DONE]":
                    continue
                try:
                    data = json.loads(data_str)
                    events.append(data)
                except json.JSONDecodeError:
                    pass

        return events
```
**EN:** Parse SSE events from a streaming response. This block implements `_parse_sse_events` and captures one focused piece of the module's behavior.
**CN:** Parse SSE events from a streaming response. 该代码块实现 `_parse_sse_events`，承担模块行为中的一个聚焦逻辑片段。

### Lines 559-560: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestAnthropicServer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestAnthropicServer.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestAnthropicServer.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestAnthropicServer._make_request`: Send a request to the /v1/messages endpoint. / 该代码块实现 `_make_request`，承担模块行为中的一个聚焦逻辑片段。
- `TestAnthropicServer._default_payload`: Build a default Anthropic Messages request payload. / 该代码块实现 `_default_payload`，承担模块行为中的一个聚焦逻辑片段。
- `TestAnthropicServer.test_tool_result_image_content_conversion`: Tool-result image blocks should be preserved as OpenAI image_url content. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tool_result_image_content_conversion`。
- `TestAnthropicServer.test_simple_messages`: Test basic non-streaming message request. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_messages`。
- `TestAnthropicServer.test_multi_turn_messages`: Test multi-turn conversation. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_turn_messages`。
- `TestAnthropicServer.test_system_message_string`: Test system message as a string. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_system_message_string`。
- `TestAnthropicServer.test_system_message_blocks`: Test system message as content blocks. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_system_message_blocks`。
- `TestAnthropicServer.test_max_tokens`: Test max_tokens limits output length. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_max_tokens`。
- `TestAnthropicServer.test_temperature`: Test temperature parameter is accepted. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_temperature`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.anthropic.protocol`, `sglang.srt.entrypoints.anthropic.serving`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 560
