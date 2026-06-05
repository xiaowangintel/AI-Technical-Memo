# test_bench_serving_reasoning_stream.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/bench_fn/test_bench_serving_reasoning_stream.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on bench fn bench serving reasoning stream in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 bench fn bench serving reasoning stream 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Document the module
```python
"""Unit tests for bench_serving streaming with reasoning_content chunks.

Reasoning models (DeepSeek-R1, MiMo, Qwen3 reasoning, Kimi-K2, ...) stream their
chain-of-thought via OpenAI's `delta.reasoning_content` field. Without explicit
support, bench_serving only inspects `delta.content` and silently reports zero
TTFT / ITL and an empty `generated_text`, which then retokenizes to 0 tokens
even though the backend completed real work.
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 10-25: Import dependencies
```python
import asyncio
import json
import socket
import threading
import time
import unittest
from argparse import Namespace
from http.server import BaseHTTPRequestHandler, HTTPServer

from sglang.bench_serving import (
    RequestFuncInput,
    async_request_openai_chat_completions,
    set_global_args,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 27-27: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 30-33: Define helper: free port
```python
def _free_port() -> int:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind(("127.0.0.1", 0))
        return s.getsockname()[1]
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 36-36: Define class _SSEHandler
```python
class _SSEHandler(BaseHTTPRequestHandler):
```
**EN:** This declaration introduces the `_SSEHandler` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `_SSEHandler` 测试类，并说明它通过继承承担的职责。

### Lines 37-37: Document the class `_SSEHandler`
```python
    """Streams a fixed sequence of OpenAI-compatible SSE chunks per test."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `_SSEHandler`.
**CN:** 该代码块保存说明性文本，用于解释周围class `_SSEHandler`的设计意图。

### Lines 39-40: Declare _SSEHandler configuration
```python
    chunks: list = []
    chunk_delay_s: float = 0.02
```
**EN:** This block defines class-level settings that are shared across the `_SSEHandler` test methods.
**CN:** 该代码块定义了 `_SSEHandler` 各测试方法共享的类级配置。

### Lines 42-55: Define helper: do POST
```python
    def do_POST(self):  # noqa: N802 (BaseHTTPRequestHandler interface)
        length = int(self.headers.get("Content-Length", "0"))
        if length:
            self.rfile.read(length)
        self.send_response(200)
        self.send_header("Content-Type", "text/event-stream")
        self.send_header("Cache-Control", "no-cache")
        self.end_headers()
        for chunk in self.chunks:
            self.wfile.write(b"data: " + json.dumps(chunk).encode() + b"\n\n")
            self.wfile.flush()
            time.sleep(self.chunk_delay_s)
        self.wfile.write(b"data: [DONE]\n\n")
        self.wfile.flush()
```
**EN:** This helper function encapsulates reusable logic inside `_SSEHandler` so the scenario stays organized. It also waits for asynchronous state transitions before rechecking results.
**CN:** 该辅助函数封装了可复用逻辑，供 `_SSEHandler` 内部调用，从而让场景结构更清晰。 其中还会在再次检查结果前等待异步状态转换完成。

### Lines 57-58: Define helper: log message
```python
    def log_message(self, fmt, *args):  # silence access logs
        return
```
**EN:** This helper function encapsulates reusable logic inside `_SSEHandler` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `_SSEHandler` 内部调用，从而让场景结构更清晰。

### Lines 61-70: Define helper: make chunk
```python
def _make_chunk(content=None, reasoning_content=None, completion_tokens=None):
    delta = {}
    if content is not None:
        delta["content"] = content
    if reasoning_content is not None:
        delta["reasoning_content"] = reasoning_content
    chunk = {"choices": [{"index": 0, "delta": delta}]}
    if completion_tokens is not None:
        chunk["usage"] = {"completion_tokens": completion_tokens}
    return chunk
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 73-73: Define class TestBenchServingReasoningStream
```python
class TestBenchServingReasoningStream(CustomTestCase):
```
**EN:** This declaration introduces the `TestBenchServingReasoningStream` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBenchServingReasoningStream` 测试类，并说明它通过继承承担的职责。

### Lines 74-84: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        set_global_args(
            Namespace(
                disable_stream=False,
                disable_ignore_eos=True,
                print_requests=False,
                tokenizer="",
                header=None,
            )
        )
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 86-110: Define helper: run
```python
    def _run(self, chunks):
        port = _free_port()

        class Handler(_SSEHandler):
            pass

        Handler.chunks = list(chunks)
        server = HTTPServer(("127.0.0.1", port), Handler)
        thread = threading.Thread(target=server.serve_forever, daemon=True)
        thread.start()
        try:
            req = RequestFuncInput(
                prompt="hello",
                api_url=f"http://127.0.0.1:{port}/v1/chat/completions",
                prompt_len=1,
                output_len=64,
                model="dummy-model",
                lora_name="",
                image_data=None,
                extra_request_body={},
            )
            return asyncio.run(async_request_openai_chat_completions(req))
        finally:
            server.shutdown()
            server.server_close()
```
**EN:** This helper function encapsulates reusable logic inside `TestBenchServingReasoningStream` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestBenchServingReasoningStream` 内部调用，从而让场景结构更清晰。

### Lines 112-128: Run test: reasoning only stream populates metrics
```python
    def test_reasoning_only_stream_populates_metrics(self):
        chunks = [
            _make_chunk(reasoning_content="Let "),
            _make_chunk(reasoning_content="me "),
            _make_chunk(reasoning_content="think."),
            _make_chunk(completion_tokens=3),
        ]
        out = self._run(chunks)

        self.assertTrue(out.success, msg=f"request failed: {out.error}")
        self.assertEqual(out.generated_text, "Let me think.")
        self.assertGreater(out.ttft, 0.0)
        self.assertEqual(len(out.itl), 2, msg="should record ITL for chunks 2..N")
        for v in out.itl:
            self.assertGreater(v, 0.0)
        self.assertEqual(out.text_chunks, ["me ", "think."])
        self.assertEqual(out.output_len, 3)
```
**EN:** This test method exercises reasoning only stream populates metrics and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reasoning only stream populates metrics 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 130-145: Run test: reasoning then content accounts both
```python
    def test_reasoning_then_content_accounts_both(self):
        chunks = [
            _make_chunk(reasoning_content="step1 "),
            _make_chunk(reasoning_content="step2 "),
            _make_chunk(content="answer "),
            _make_chunk(content="here"),
            _make_chunk(completion_tokens=4),
        ]
        out = self._run(chunks)

        self.assertTrue(out.success, msg=f"request failed: {out.error}")
        self.assertEqual(out.generated_text, "step1 step2 answer here")
        self.assertGreater(out.ttft, 0.0)
        self.assertEqual(len(out.itl), 3)
        self.assertEqual(out.text_chunks, ["step2 ", "answer ", "here"])
        self.assertEqual(out.output_len, 4)
```
**EN:** This test method exercises reasoning then content accounts both and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reasoning then content accounts both 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 147-157: Run test: single delta preserves reasoning before content
```python
    def test_single_delta_preserves_reasoning_before_content(self):
        chunks = [
            _make_chunk(content="answer", reasoning_content="thought "),
            _make_chunk(completion_tokens=2),
        ]
        out = self._run(chunks)

        self.assertTrue(out.success, msg=f"request failed: {out.error}")
        self.assertEqual(out.generated_text, "thought answer")
        self.assertGreater(out.ttft, 0.0)
        self.assertEqual(out.output_len, 2)
```
**EN:** This test method exercises single delta preserves reasoning before content and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single delta preserves reasoning before content 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 159-169: Run test: usage only stream chunk does not break
```python
    def test_usage_only_stream_chunk_does_not_break(self):
        chunks = [
            _make_chunk(reasoning_content="thinking"),
            {"choices": [], "usage": {"completion_tokens": 1}},
        ]
        out = self._run(chunks)

        self.assertTrue(out.success, msg=f"request failed: {out.error}")
        self.assertEqual(out.generated_text, "thinking")
        self.assertGreater(out.ttft, 0.0)
        self.assertEqual(out.output_len, 1)
```
**EN:** This test method exercises usage only stream chunk does not break and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 usage only stream chunk does not break 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 171-184: Run test: content only stream unchanged
```python
    def test_content_only_stream_unchanged(self):
        chunks = [
            _make_chunk(content="hi "),
            _make_chunk(content="there"),
            _make_chunk(completion_tokens=2),
        ]
        out = self._run(chunks)

        self.assertTrue(out.success, msg=f"request failed: {out.error}")
        self.assertEqual(out.generated_text, "hi there")
        self.assertGreater(out.ttft, 0.0)
        self.assertEqual(len(out.itl), 1)
        self.assertEqual(out.text_chunks, ["there"])
        self.assertEqual(out.output_len, 2)
```
**EN:** This test method exercises content only stream unchanged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 content only stream unchanged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 186-206: Run test: null reasoning field does not break
```python
    def test_null_reasoning_field_does_not_break(self):
        # Mirrors sglang's _StreamDelta: reasoning_content is always emitted,
        # serialized as null when only content is present.
        chunks = [
            {
                "choices": [
                    {"index": 0, "delta": {"content": "ok", "reasoning_content": None}}
                ]
            },
            {
                "choices": [
                    {"index": 0, "delta": {"content": None, "reasoning_content": None}}
                ],
                "usage": {"completion_tokens": 1},
            },
        ]
        out = self._run(chunks)

        self.assertTrue(out.success, msg=f"request failed: {out.error}")
        self.assertEqual(out.generated_text, "ok")
        self.assertGreater(out.ttft, 0.0)
```
**EN:** This test method exercises null reasoning field does not break and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 null reasoning field does not break 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 209-210: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.bench_serving`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `argparse`, `asyncio`, `http.server`, `json`, `socket`, `threading`, `time`, `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
