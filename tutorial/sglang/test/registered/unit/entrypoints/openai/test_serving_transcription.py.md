# test_serving_transcription.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/entrypoints/openai/test_serving_transcription.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates serving transcription behavior in SGLang's unit / entrypoints / openai area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 入口 / OpenAI 领域中与 serving transcription 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: supporting statements / 辅助语句
```python
"""Unit tests for OpenAIServingTranscription's streaming fused-autodetect path.

Exercises the streaming handler: buffer deltas until the forced-prefix
sentinel lands, emit the scrubbed user-visible text, and never leak
Whisper special tokens. Covers both streaming modes — cumulative
(``incremental_streaming_output=False``, the default) and incremental
(``incremental_streaming_output=True``).

The tests mock ``TokenizerManager.generate_request`` to yield synthetic
``text`` chunks for each of the happy, abort, and boundary cases.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 13-13: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.test_utils import maybe_stub_sgl_kernel
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.test_utils`。

### Lines 15-15: supporting statements / 辅助语句
```python
maybe_stub_sgl_kernel()  # must precede any import that pulls in sgl_kernel
```
**EN:** This block performs supporting work through calls such as `maybe_stub_sgl_kernel`, preparing state for nearby definitions.
**CN:** 该代码块通过 `maybe_stub_sgl_kernel` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 17-29: module imports and dependencies / 模块导入与依赖
```python
import json
import unittest
from typing import List
from unittest.mock import Mock

from sglang.srt.entrypoints.openai.protocol import TranscriptionRequest
from sglang.srt.entrypoints.openai.serving_transcription import (
    OpenAIServingTranscription,
)
from sglang.srt.managers.io_struct import GenerateReqInput
from sglang.srt.utils import get_or_create_event_loop
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `unittest`, `typing`, `unittest.mock`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `unittest`, `typing`, `unittest.mock`。

### Lines 31-31: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=4, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 34-41: function chunk / 函数 chunk
```python
def _chunk(text: str, finish: str = None) -> dict:
    """Shape of what TokenizerManager.generate_request yields per step."""
    return {
        "text": text,
        "meta_info": {
            "finish_reason": {"type": finish} if finish else None,
        },
    }
```
**EN:** Shape of what TokenizerManager.generate_request yields per step. This block implements `_chunk` and captures one focused piece of the module's behavior.
**CN:** Shape of what TokenizerManager.generate_request yields per step. 该代码块实现 `_chunk`，承担模块行为中的一个聚焦逻辑片段。

### Lines 44-44: class _MockTokenizerManager declaration / 类 _MockTokenizerManager 声明
```python
class _MockTokenizerManager:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 45-45: supporting statements / 辅助语句
```python
    """Minimal mock satisfying OpenAIServingTranscription.__init__ and stream loop."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 47-56: method init / 方法 init
```python
    def __init__(self, stream_chunks: List[dict]):
        self.model_config = Mock()
        self.model_config.hf_config = Mock()
        self.model_config.hf_config.architectures = ["WhisperForConditionalGeneration"]
        # Not a real ServerArgs, so base class sets allowed_custom_labels=None.
        # Default tests assume cumulative-text streaming (the sglang upstream
        # default); tests for incremental_streaming_output=True override this.
        self.server_args = Mock(incremental_streaming_output=False)
        self.tokenizer = Mock()
        self._stream_chunks = stream_chunks
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 58-65: method generate request / 方法 generate request
```python
    def generate_request(self, adapted_request, raw_request):
        chunks = self._stream_chunks

        async def gen():
            for c in chunks:
                yield c

        return gen()
```
**EN:** This block implements `generate_request` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `generate_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 67-68: helper routine create abort task / 辅助流程 create abort task
```python
    def create_abort_task(self, adapted_request):
        return None
```
**EN:** This helper encapsulates `create_abort_task` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `create_abort_task`，以便周围测试复用准备、执行或校验逻辑。

### Lines 71-88: function deltas from sse / 函数 deltas from sse
```python
def _deltas_from_sse(sse_lines: List[str]) -> List[str]:
    """Extract ``choices[0].delta.content`` strings from a list of SSE frames."""
    out = []
    for line in sse_lines:
        if not line.startswith("data: "):
            continue
        payload = line[len("data: ") :].strip()
        if payload == "[DONE]":
            continue
        try:
            obj = json.loads(payload)
        except json.JSONDecodeError:
            continue
        for choice in obj.get("choices", []):
            content = (choice.get("delta") or {}).get("content")
            if content:
                out.append(content)
    return out
```
**EN:** Extract ``choices[0].delta.content`` strings from a list of SSE frames. This block implements `_deltas_from_sse` and captures one focused piece of the module's behavior.
**CN:** Extract ``choices[0].delta.content`` strings from a list of SSE frames. 该代码块实现 `_deltas_from_sse`，承担模块行为中的一个聚焦逻辑片段。

### Lines 91-91: class TestStreamingFusedAutodetect declaration / 类 TestStreamingFusedAutodetect 声明
```python
class TestStreamingFusedAutodetect(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 92-92: supporting statements / 辅助语句
```python
    """_generate_transcription_stream with _fused_autodetect=True."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 94-120: method run stream / 方法 run stream
```python
    def _run_stream(
        self, chunks: List[dict], fused: bool = True, ts_variant: bool = False
    ):
        tm = _MockTokenizerManager(chunks)
        serving = OpenAIServingTranscription(tm)

        kwargs = {"model": "whisper", "stream": True}
        if ts_variant:
            kwargs["timestamp_granularities"] = ["segment"]
        request = TranscriptionRequest(**kwargs)
        if fused:
            request._fused_autodetect = True
            request._fused_ts_variant = ts_variant
        adapted = GenerateReqInput(text="", modalities=["audio"])
        raw_request = Mock()

        async def drive():
            frames = []
            async for frame in serving._generate_transcription_stream(
                adapted, request, raw_request
            ):
                frames.append(frame)
            return frames

        loop = get_or_create_event_loop()
        frames = loop.run_until_complete(drive())
        return request, frames
```
**EN:** This block implements `_run_stream` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_stream`，承担模块行为中的一个聚焦逻辑片段。

### Lines 122-136: test case prefix stripped and language extracted / 测试用例 prefix stripped and language extracted
```python
    def test_prefix_stripped_and_language_extracted(self):
        chunks = [
            _chunk("<|en|>"),
            _chunk("<|en|><|transcribe|>"),
            _chunk("<|en|><|transcribe|><|notimestamps|>"),
            _chunk("<|en|><|transcribe|><|notimestamps|> Hello"),
            _chunk("<|en|><|transcribe|><|notimestamps|> Hello world", finish="stop"),
        ]
        request, frames = self._run_stream(chunks)
        deltas = _deltas_from_sse(frames)
        self.assertEqual(deltas, ["Hello", " world"])
        self.assertEqual(request.language, "en")
        # No delta ever starts with the forced prefix or leading whitespace.
        self.assertFalse(any("<|" in d for d in deltas))
        self.assertFalse(deltas[0].startswith(" "))
```
**EN:** This test exercises `test_prefix_stripped_and_language_extracted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prefix_stripped_and_language_extracted`。

### Lines 138-145: test case non english language extracted / 测试用例 non english language extracted
```python
    def test_non_english_language_extracted(self):
        chunks = [
            _chunk("<|zh|><|transcribe|><|notimestamps|>你好"),
            _chunk("<|zh|><|transcribe|><|notimestamps|>你好世界", finish="stop"),
        ]
        request, frames = self._run_stream(chunks)
        self.assertEqual(request.language, "zh")
        self.assertEqual(_deltas_from_sse(frames), ["你好", "世界"])
```
**EN:** This test exercises `test_non_english_language_extracted` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_english_language_extracted`。

### Lines 147-163: test case fsm abort before sentinel emits error frame / 测试用例 fsm abort before sentinel emits error frame
```python
    def test_fsm_abort_before_sentinel_emits_error_frame(self):
        # Sentinel never arrives; stream terminates on finish_reason. The
        # handler must surface this as a real SSE error frame so the client
        # can distinguish "detection failed" from "silent audio with zero
        # transcription". language stays unset.
        chunks = [
            _chunk("<|en|>"),
            _chunk("<|en|><|transcribe|>", finish="length"),
        ]
        request, frames = self._run_stream(chunks)
        self.assertEqual(_deltas_from_sse(frames), [])
        error_frames = [f for f in frames if f.startswith("data: ") and '"error"' in f]
        self.assertTrue(
            error_frames, f"expected an SSE error frame, got frames={frames!r}"
        )
        self.assertIn("language auto-detect failed", error_frames[0])
        self.assertIsNone(request.language)
```
**EN:** This test exercises `test_fsm_abort_before_sentinel_emits_error_frame` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fsm_abort_before_sentinel_emits_error_frame`。

### Lines 165-172: test case non fused stream passes through / 测试用例 non fused stream passes through
```python
    def test_non_fused_stream_passes_through(self):
        # When _fused_autodetect is False, no buffering or anchoring happens.
        chunks = [
            _chunk("Hello"),
            _chunk("Hello world", finish="stop"),
        ]
        request, frames = self._run_stream(chunks, fused=False)
        self.assertEqual(_deltas_from_sse(frames), ["Hello", " world"])
```
**EN:** This test exercises `test_non_fused_stream_passes_through` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_fused_stream_passes_through`。

### Lines 174-200: test case streaming ts variant sentinel at chunk boundary / 测试用例 streaming ts variant sentinel at chunk boundary
```python
    def test_streaming_ts_variant_sentinel_at_chunk_boundary(self):
        # The <|0.00|> sentinel can land in its own chunk ahead of any
        # transcription text, and the trailing-space arrives later. The
        # handler must buffer silently until a non-whitespace char shows
        # up (so the first delta doesn't leak a leading space) and then
        # scrub subsequent embedded timestamp tokens.
        chunks = [
            _chunk("<|en|>"),
            _chunk("<|en|><|transcribe|>"),
            _chunk("<|en|><|transcribe|><|0.00|>"),  # sentinel alone
            _chunk("<|en|><|transcribe|><|0.00|> "),  # + whitespace only
            _chunk("<|en|><|transcribe|><|0.00|> Hello"),  # first word
            _chunk("<|en|><|transcribe|><|0.00|> Hello<|5.00|> World"),
            _chunk(
                "<|en|><|transcribe|><|0.00|> Hello<|5.00|> World<|endoftext|>",
                finish="stop",
            ),
        ]
        request, frames = self._run_stream(chunks, ts_variant=True)
        deltas = _deltas_from_sse(frames)
        self.assertEqual(request.language, "en")
        self.assertFalse(any("<|" in d for d in deltas))
        # No delta starts with a leading space (the one Whisper emits
        # between <|0.00|> and "Hello" was consumed by the defer-on-
        # whitespace path).
        self.assertFalse(deltas[0].startswith(" "))
        self.assertEqual("".join(deltas), "Hello World")
```
**EN:** This test exercises `test_streaming_ts_variant_sentinel_at_chunk_boundary` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_ts_variant_sentinel_at_chunk_boundary`。

### Lines 202-222: test case streaming timestamps variant scrubs embedded segment tokens / 测试用例 streaming timestamps variant scrubs embedded segment tokens
```python
    def test_streaming_timestamps_variant_scrubs_embedded_segment_tokens(self):
        # Streaming + timestamp_granularities + language=None uses the fused
        # timestamps variant (<|0.00|> sentinel). Segment-boundary tokens
        # <|5.00|>, <|10.00|> land mid-stream; each delta must have them
        # scrubbed before reaching the client. Auto-detection still works
        # — the SSE stream carries clean text, and callers who want
        # segment timing can use response_format=verbose_json which builds
        # segments from output_ids on a separate path.
        chunks = [
            _chunk("<|en|><|transcribe|><|0.00|> Hello"),
            _chunk("<|en|><|transcribe|><|0.00|> Hello<|5.00|> World"),
            _chunk(
                "<|en|><|transcribe|><|0.00|> Hello<|5.00|> World<|10.00|><|endoftext|>",
                finish="stop",
            ),
        ]
        request, frames = self._run_stream(chunks, ts_variant=True)
        deltas = _deltas_from_sse(frames)
        self.assertEqual(request.language, "en")
        self.assertFalse(any("<|" in d for d in deltas))
        self.assertEqual("".join(deltas), "Hello World")
```
**EN:** This test exercises `test_streaming_timestamps_variant_scrubs_embedded_segment_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_streaming_timestamps_variant_scrubs_embedded_segment_tokens`。

### Lines 224-238: test case trailing endoftext scrubbed from last delta / 测试用例 trailing endoftext scrubbed from last delta
```python
    def test_trailing_endoftext_scrubbed_from_last_delta(self):
        # skip_special_tokens=False means the detokenizer may emit
        # <|endoftext|> at the tail. The fused streaming path must scrub it
        # per-delta so clients never see special tokens in SSE chunks.
        chunks = [
            _chunk("<|en|><|transcribe|><|notimestamps|> Hello"),
            _chunk(
                "<|en|><|transcribe|><|notimestamps|> Hello world<|endoftext|>",
                finish="stop",
            ),
        ]
        _, frames = self._run_stream(chunks)
        deltas = _deltas_from_sse(frames)
        self.assertEqual(deltas, ["Hello", " world"])
        self.assertFalse(any("<|" in d for d in deltas))
```
**EN:** This test exercises `test_trailing_endoftext_scrubbed_from_last_delta` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trailing_endoftext_scrubbed_from_last_delta`。

### Lines 241-241: class TestStreamingIncrementalOutputMode declaration / 类 TestStreamingIncrementalOutputMode 声明
```python
class TestStreamingIncrementalOutputMode(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 242-249: supporting statements / 辅助语句
```python
    """Server runs with ``incremental_streaming_output=True``.

    In that mode each chunk's ``content["text"]`` is the new delta from the
    detokenizer, not the cumulative text. The handler must accumulate
    locally into ``cumulative_text`` — otherwise the subsequent
    ``visible[len(visible_buffer):]`` slice would strip characters the
    server already sent as a delta.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 251-274: method run incremental stream / 方法 run incremental stream
```python
    def _run_incremental_stream(self, chunk_deltas, fused=False):
        """Server in incremental mode: yield per-chunk delta, not cumulative."""
        chunks = [
            _chunk(d, finish=("stop" if i == len(chunk_deltas) - 1 else None))
            for i, d in enumerate(chunk_deltas)
        ]
        tm = _MockTokenizerManager(chunks)
        tm.server_args = Mock(incremental_streaming_output=True)
        serving = OpenAIServingTranscription(tm)

        request = TranscriptionRequest(model="whisper", stream=True)
        if fused:
            request._fused_autodetect = True
        adapted = GenerateReqInput(text="", modalities=["audio"])

        async def drive():
            frames = []
            async for f in serving._generate_transcription_stream(
                adapted, request, Mock()
            ):
                frames.append(f)
            return frames

        return request, get_or_create_event_loop().run_until_complete(drive())
```
**EN:** Server in incremental mode: yield per-chunk delta, not cumulative. This block implements `_run_incremental_stream` and captures one focused piece of the module's behavior.
**CN:** Server in incremental mode: yield per-chunk delta, not cumulative. 该代码块实现 `_run_incremental_stream`，承担模块行为中的一个聚焦逻辑片段。

### Lines 276-282: test case incremental non fused emits each delta verbatim / 测试用例 incremental non fused emits each delta verbatim
```python
    def test_incremental_non_fused_emits_each_delta_verbatim(self):
        # sglang.private default: each content["text"] IS the new delta, so
        # the handler should NOT slice it. Client should see exactly what
        # the detokenizer emitted.
        deltas_in = [" The", " President", ":", " Thank", " you"]
        _, frames = self._run_incremental_stream(deltas_in, fused=False)
        self.assertEqual(_deltas_from_sse(frames), deltas_in)
```
**EN:** This test exercises `test_incremental_non_fused_emits_each_delta_verbatim` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_incremental_non_fused_emits_each_delta_verbatim`。

### Lines 284-299: test case incremental fused autodetect still strips prefix / 测试用例 incremental fused autodetect still strips prefix
```python
    def test_incremental_fused_autodetect_still_strips_prefix(self):
        # Incremental + fused: the handler must accumulate to find the
        # sentinel, then emit only the post-prefix portion per chunk.
        deltas_in = [
            "<|en|>",
            "<|transcribe|>",
            "<|notimestamps|>",
            " Hello",
            " world",
        ]
        request, frames = self._run_incremental_stream(deltas_in, fused=True)
        emitted = _deltas_from_sse(frames)
        # Prefix never leaks, and concat matches the expected transcription.
        self.assertFalse(any("<|" in d for d in emitted))
        self.assertEqual("".join(emitted), "Hello world")
        self.assertEqual(request.language, "en")
```
**EN:** This test exercises `test_incremental_fused_autodetect_still_strips_prefix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_incremental_fused_autodetect_still_strips_prefix`。

### Lines 302-303: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_chunk`: Shape of what TokenizerManager.generate_request yields per step. / 该代码块实现 `_chunk`，承担模块行为中的一个聚焦逻辑片段。
- `_MockTokenizerManager`: Minimal mock satisfying OpenAIServingTranscription.__init__ and stream loop. / 用于组织相关测试、夹具或辅助方法。
- `_deltas_from_sse`: Extract ``choices[0].delta.content`` strings from a list of SSE frames. / 该代码块实现 `_deltas_from_sse`，承担模块行为中的一个聚焦逻辑片段。
- `TestStreamingFusedAutodetect`: _generate_transcription_stream with _fused_autodetect=True. / 用于组织相关测试、夹具或辅助方法。
- `TestStreamingIncrementalOutputMode`: Server runs with ``incremental_streaming_output=True``. / 用于组织相关测试、夹具或辅助方法。
- `_MockTokenizerManager.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `_MockTokenizerManager.generate_request`: This block implements `generate_request` and captures one focused piece of the module's behavior. / 该代码块实现 `generate_request`，承担模块行为中的一个聚焦逻辑片段。
- `_MockTokenizerManager.create_abort_task`: This helper encapsulates `create_abort_task` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `create_abort_task`，以便周围测试复用准备、执行或校验逻辑。
- `TestStreamingFusedAutodetect._run_stream`: This block implements `_run_stream` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_stream`，承担模块行为中的一个聚焦逻辑片段。
- `TestStreamingFusedAutodetect.test_prefix_stripped_and_language_extracted`: This test exercises `test_prefix_stripped_and_language_extracted` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_prefix_stripped_and_language_extracted`。
- `TestStreamingFusedAutodetect.test_non_english_language_extracted`: This test exercises `test_non_english_language_extracted` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_english_language_extracted`。
- `TestStreamingFusedAutodetect.test_fsm_abort_before_sentinel_emits_error_frame`: This test exercises `test_fsm_abort_before_sentinel_emits_error_frame` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fsm_abort_before_sentinel_emits_error_frame`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `unittest`, `typing`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.test.test_utils`, `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.entrypoints.openai.serving_transcription`, `sglang.srt.managers.io_struct`, `sglang.srt.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 303
