# test_serving_transcription.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/openai_server/basic/test_serving_transcription.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates serving transcription behavior in SGLang's openai server / basic area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 openai server / basic 领域中与 serving transcription 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""
Test the OpenAI-compatible /v1/audio/transcriptions endpoint with Whisper.

Usage:
    python3 test_serving_transcription.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-22: module imports and dependencies / 模块导入与依赖
```python
import io
import json
import unittest
from typing import List, Optional

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `io`, `json`, `unittest`, `typing`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `io`, `json`, `unittest`, `typing`。

### Lines 24-27: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=60, stage="base-b", runner_config="1-gpu-small")

WHISPER_MODEL = "openai/whisper-large-v3"
AUDIO_URL = "https://raw.githubusercontent.com/sgl-project/sgl-test-files/refs/heads/main/audios/Trump_WEF_2018_10s.mp3"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 30-34: function download audio bytes / 函数 download audio bytes
```python
def download_audio_bytes(url=AUDIO_URL):
    """Download audio file and return raw bytes."""
    response = requests.get(url, timeout=30)
    response.raise_for_status()
    return response.content
```
**EN:** Download audio file and return raw bytes. This block implements `download_audio_bytes` and captures one focused piece of the module's behavior.
**CN:** Download audio file and return raw bytes. 该代码块实现 `download_audio_bytes`，承担模块行为中的一个聚焦逻辑片段。

### Lines 37-37: class TestServingTranscription declaration / 类 TestServingTranscription 声明
```python
class TestServingTranscription(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 38-38: supporting statements / 辅助语句
```python
    """Test Whisper transcription via /v1/audio/transcriptions endpoint."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 40-52: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = WHISPER_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--served-model-name",
                "whisper",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 54-57: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process") and cls.process:
            kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 59-85: method transcribe / 方法 transcribe
```python
    def _transcribe(
        self,
        language: Optional[str] = "en",
        response_format: Optional[str] = None,
        timestamp_granularities: Optional[List[str]] = None,
    ):
        """Send a non-streaming transcription request and return the JSON response.

        Passing ``language=None`` omits the field entirely, which exercises
        the fused auto-detect path.
        """
        audio_bytes = download_audio_bytes()
        data = {"model": "whisper"}
        if language is not None:
            data["language"] = language
        if response_format is not None:
            data["response_format"] = response_format
        if timestamp_granularities is not None:
            # Form-encoded list fields repeat the key
            data["timestamp_granularities[]"] = timestamp_granularities
        response = requests.post(
            self.base_url + "/v1/audio/transcriptions",
            files={"file": ("audio.mp3", io.BytesIO(audio_bytes), "audio/mpeg")},
            data=data,
        )
        self.assertEqual(response.status_code, 200, response.text)
        return response.json()
```
**EN:** Send a non-streaming transcription request and return the JSON response. This block implements `_transcribe` and captures one focused piece of the module's behavior.
**CN:** Send a non-streaming transcription request and return the JSON response. 该代码块实现 `_transcribe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 87-116: method transcribe stream / 方法 transcribe stream
```python
    def _transcribe_stream(self, language: Optional[str] = None) -> List[str]:
        """Send a streaming transcription request and return the delta strings."""
        audio_bytes = download_audio_bytes()
        data = {"model": "whisper", "stream": "true"}
        if language is not None:
            data["language"] = language
        with requests.post(
            self.base_url + "/v1/audio/transcriptions",
            files={"file": ("audio.mp3", io.BytesIO(audio_bytes), "audio/mpeg")},
            data=data,
            stream=True,
            timeout=120,
        ) as response:
            self.assertEqual(response.status_code, 200, response.text)
            deltas: List[str] = []
            for raw in response.iter_lines():
                if not raw:
                    continue
                line = raw.decode("utf-8")
                if not line.startswith("data: "):
                    continue
                payload = line[len("data: ") :].strip()
                if payload == "[DONE]":
                    break
                obj = json.loads(payload)
                for choice in obj.get("choices", []):
                    content = (choice.get("delta") or {}).get("content")
                    if content:
                        deltas.append(content)
            return deltas
```
**EN:** Send a streaming transcription request and return the delta strings. This block implements `_transcribe_stream` and captures one focused piece of the module's behavior.
**CN:** Send a streaming transcription request and return the delta strings. 该代码块实现 `_transcribe_stream`，承担模块行为中的一个聚焦逻辑片段。

### Lines 118-122: test case basic transcription / 测试用例 basic transcription
```python
    def test_basic_transcription(self):
        """Test that transcription returns a valid non-empty response."""
        result = self._transcribe()
        self.assertIn("text", result)
        self.assertTrue(len(result["text"]) > 0, "Transcription should not be empty")
```
**EN:** Test that transcription returns a valid non-empty response. This test exercises `test_basic_transcription` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that transcription returns a valid non-empty response. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_transcription`。

### Lines 124-135: test case transcription content quality / 测试用例 transcription content quality
```python
    def test_transcription_content_quality(self):
        """Test that transcription captures key content from the audio."""
        result = self._transcribe()
        text = result["text"].lower()
        keywords = ["privilege", "leader", "science", "art"]
        matches = [kw for kw in keywords if kw in text]
        self.assertGreaterEqual(
            len(matches),
            2,
            f"Expected at least 2 of {keywords} in transcription, "
            f"found {matches}. Full text: {text}",
        )
```
**EN:** Test that transcription captures key content from the audio. This test exercises `test_transcription_content_quality` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that transcription captures key content from the audio. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_transcription_content_quality`。

### Lines 137-151: test case multiple sequential requests / 测试用例 multiple sequential requests
```python
    def test_multiple_sequential_requests(self):
        """Test that sequential requests produce consistent results."""
        results = []
        for _ in range(3):
            result = self._transcribe()
            self.assertIn("text", result)
            self.assertTrue(len(result["text"]) > 0)
            results.append(result["text"])

        for i in range(1, len(results)):
            self.assertEqual(
                results[0],
                results[i],
                f"Transcription {i + 1} differs from first transcription",
            )
```
**EN:** Test that sequential requests produce consistent results. This test exercises `test_multiple_sequential_requests` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that sequential requests produce consistent results. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_sequential_requests`。

### Lines 152-157: supporting source context / 辅助源码上下文
```python

    # -- fused auto-detect (language=None) ---------------------------------
    # The clip is English, so the fused path must both produce a valid
    # transcription AND expose "en" as the detected language. None of the
    # deltas / text fields should leak Whisper special tokens.

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 158-173: test case auto detect language verbose json / 测试用例 auto detect language verbose json
```python
    def test_auto_detect_language_verbose_json(self):
        """language omitted + verbose_json returns detected language + clean text."""
        result = self._transcribe(language=None, response_format="verbose_json")
        self.assertEqual(result.get("language"), "en")
        text = result.get("text", "")
        self.assertTrue(len(text) > 0, "Transcription should not be empty")
        self.assertNotIn("<|", text, f"Special token leaked into text: {text!r}")
        # Sanity-check content against the same keywords the English test uses.
        keywords = ["privilege", "leader", "science", "art"]
        matches = [kw for kw in keywords if kw in text.lower()]
        self.assertGreaterEqual(
            len(matches),
            2,
            f"Expected at least 2 of {keywords} in auto-detected transcription, "
            f"found {matches}. Full text: {text!r}",
        )
```
**EN:** language omitted + verbose_json returns detected language + clean text. This test exercises `test_auto_detect_language_verbose_json` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** language omitted + verbose_json returns detected language + clean text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_auto_detect_language_verbose_json`。

### Lines 175-185: test case auto detect matches explicit english / 测试用例 auto detect matches explicit english
```python
    def test_auto_detect_matches_explicit_english(self):
        """Auto-detected (language=None) text should match explicit language=en."""
        auto = self._transcribe(language=None).get("text", "")
        explicit = self._transcribe(language="en").get("text", "")
        self.assertEqual(
            auto.strip(),
            explicit.strip(),
            "Auto-detect should produce the same transcription as language=en "
            "on an English clip.",
        )
        self.assertNotIn("<|", auto)
```
**EN:** Auto-detected (language=None) text should match explicit language=en. This test exercises `test_auto_detect_matches_explicit_english` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Auto-detected (language=None) text should match explicit language=en. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_auto_detect_matches_explicit_english`。

### Lines 187-204: test case auto detect with segment timestamps / 测试用例 auto detect with segment timestamps
```python
    def test_auto_detect_with_segment_timestamps(self):
        """language=None + timestamp_granularities uses the timestamps fused regex."""
        result = self._transcribe(
            language=None,
            response_format="verbose_json",
            timestamp_granularities=["segment"],
        )
        self.assertEqual(result.get("language"), "en")
        segments = result.get("segments") or []
        self.assertGreater(len(segments), 0, "Expected at least one segment")
        for seg in segments:
            self.assertIn("start", seg)
            self.assertIn("end", seg)
            self.assertIn("text", seg)
            self.assertGreaterEqual(seg["end"], seg["start"])
            self.assertNotIn(
                "<|", seg["text"], f"Special token leaked into segment: {seg!r}"
            )
```
**EN:** language=None + timestamp_granularities uses the timestamps fused regex. This test exercises `test_auto_detect_with_segment_timestamps` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** language=None + timestamp_granularities uses the timestamps fused regex. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_auto_detect_with_segment_timestamps`。

### Lines 206-226: test case auto detect streaming / 测试用例 auto detect streaming
```python
    def test_auto_detect_streaming(self):
        """language=None + stream=True: deltas scrubbed, concat matches non-streaming.

        Verified against a real server: sglang's streaming path for Whisper
        produces clean deltas (complete words, no BPE fragmentation), so the
        fused path only needs to hide the forced prefix — which this PR
        does. Asserts both the prefix-leak guard and text equivalence.
        """
        deltas = self._transcribe_stream(language=None)
        self.assertTrue(len(deltas) > 0, "Expected at least one streamed delta")
        for d in deltas:
            self.assertNotIn(
                "<|", d, f"Special token leaked into streaming delta: {d!r}"
            )
        streamed = "".join(deltas).strip()
        reference = self._transcribe(language=None).get("text", "").strip()
        self.assertEqual(
            streamed,
            reference,
            "Streamed auto-detect text should match the non-streaming result.",
        )
```
**EN:** language=None + stream=True: deltas scrubbed, concat matches non-streaming. This test exercises `test_auto_detect_streaming` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** language=None + stream=True: deltas scrubbed, concat matches non-streaming. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_auto_detect_streaming`。

### Lines 229-230: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `download_audio_bytes`: Download audio file and return raw bytes. / 该代码块实现 `download_audio_bytes`，承担模块行为中的一个聚焦逻辑片段。
- `TestServingTranscription`: Test Whisper transcription via /v1/audio/transcriptions endpoint. / 用于组织相关测试、夹具或辅助方法。
- `TestServingTranscription.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestServingTranscription.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestServingTranscription._transcribe`: Send a non-streaming transcription request and return the JSON response. / 该代码块实现 `_transcribe`，承担模块行为中的一个聚焦逻辑片段。
- `TestServingTranscription._transcribe_stream`: Send a streaming transcription request and return the delta strings. / 该代码块实现 `_transcribe_stream`，承担模块行为中的一个聚焦逻辑片段。
- `TestServingTranscription.test_basic_transcription`: Test that transcription returns a valid non-empty response. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_transcription`。
- `TestServingTranscription.test_transcription_content_quality`: Test that transcription captures key content from the audio. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_transcription_content_quality`。
- `TestServingTranscription.test_multiple_sequential_requests`: Test that sequential requests produce consistent results. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_sequential_requests`。
- `TestServingTranscription.test_auto_detect_language_verbose_json`: language omitted + verbose_json returns detected language + clean text. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_auto_detect_language_verbose_json`。
- `TestServingTranscription.test_auto_detect_matches_explicit_english`: Auto-detected (language=None) text should match explicit language=en. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_auto_detect_matches_explicit_english`。
- `TestServingTranscription.test_auto_detect_with_segment_timestamps`: language=None + timestamp_granularities uses the timestamps fused regex. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_auto_detect_with_segment_timestamps`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `io`, `json`, `unittest`, `typing`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 230
