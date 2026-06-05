# test_qwen3_asr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/models/test_qwen3_asr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `qwen3 asr` scenario in `test/manual/models`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual/models` 中的 `qwen3 asr` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Constants and scenario settings / 常量与场景配置
```python
"""
Test Qwen3-ASR model support in SGLang.

Tests /v1/audio/transcriptions endpoint (OpenAI-compatible).

Usage:
    python test/manual/models/test_qwen3_asr.py
"""

import io
import os
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)

MODEL = "Qwen/Qwen3-ASR-0.6B"
# MODEL = "Qwen/Qwen3-ASR-1.7B"
TEST_AUDIO_EN_URL = (
    "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen3-ASR-Repo/asr_en.wav"
)
TEST_AUDIO_ZH_URL = (
    "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen3-ASR-Repo/asr_zh.wav"
)
TEST_AUDIO_EN_LOCAL = "/tmp/test_qwen3_asr_en.wav"
TEST_AUDIO_ZH_LOCAL = "/tmp/test_qwen3_asr_zh.wav"
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `endpoint`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 36-45: Helper routines around download_audio / 辅助例程
```python
def download_audio(url, local_path):
    """Download audio file if not already cached."""
    if os.path.exists(local_path):
        with open(local_path, "rb") as f:
            return f.read()
    resp = requests.get(url, timeout=60)
    resp.raise_for_status()
    with open(local_path, "wb") as f:
        f.write(resp.content)
    return resp.content
```
**EN:** This range implements helper routine(s) `download_audio` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `exists`, `read`, `get` and `raise_for_status`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 48-51: Class definition for TestQwen3ASRTranscription / 类定义
```python
class TestQwen3ASRTranscription(CustomTestCase):
    """Test Qwen3-ASR via /v1/audio/transcriptions endpoint."""

    @classmethod
```
**EN:** This range declares `TestQwen3ASRTranscription`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 52-64: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--served-model-name",
                "qwen3-asr",
                "--trust-remote-code",
            ],
        )
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 65-66: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 67-68: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 70-72: Helper routines around _transcribe / 辅助例程
```python
    def _transcribe(self, audio_url, local_path, language=None):
        """Send a transcription request."""
        audio_bytes = download_audio(audio_url, local_path)
```
**EN:** This range implements helper routine(s) `_transcribe` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `download_audio`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 73-83: Assertions and result checks / 断言与结果检查
```python
        data = {"model": "qwen3-asr"}
        if language:
            data["language"] = language
        response = requests.post(
            self.base_url + "/v1/audio/transcriptions",
            files={"file": ("audio.wav", io.BytesIO(audio_bytes), "audio/wav")},
            data=data,
            timeout=120,
        )
        self.assertEqual(response.status_code, 200, response.text)
        return response.json()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `post`, `BytesIO`, `assertEqual` and `json`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 85-91: Test routines around test_english_transcription / 测试例程
```python
    def test_english_transcription(self):
        """Test English audio transcription."""
        result = self._transcribe(TEST_AUDIO_EN_URL, TEST_AUDIO_EN_LOCAL)
        self.assertIn("text", result)
        text = result["text"]
        self.assertTrue(len(text) > 0, "Transcription should not be empty")
        print(f"[EN Transcription] {text}")
```
**EN:** This range defines concrete test routine(s) `test_english_transcription`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `_transcribe`, `assertIn` and `assertTrue`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 93-97: Test routines around test_chinese_transcription / 测试例程
```python
    def test_chinese_transcription(self):
        """Test Chinese audio transcription."""
        result = self._transcribe(TEST_AUDIO_ZH_URL, TEST_AUDIO_ZH_LOCAL)
        self.assertIn("text", result)
        text = result["text"]
```
**EN:** This range defines concrete test routine(s) `test_chinese_transcription`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `_transcribe` and `assertIn`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 98-99: Assertions and result checks / 断言与结果检查
```python
        self.assertTrue(len(text) > 0, "Transcription should not be empty")
        print(f"[ZH Transcription] {text}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 101-114: Test routines around test_multiple_requests_consistency / 测试例程
```python
    def test_multiple_requests_consistency(self):
        """Test that repeated requests produce consistent output."""
        results = []
        for _ in range(3):
            result = self._transcribe(TEST_AUDIO_EN_URL, TEST_AUDIO_EN_LOCAL)
            results.append(result["text"])

        for i in range(1, len(results)):
            self.assertEqual(
                results[0],
                results[i],
                f"Request {i+1} differs from first request",
            )
        print(f"[Consistency] All 3 requests match: {results[0][:80]}...")
```
**EN:** This range defines concrete test routine(s) `test_multiple_requests_consistency`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `_transcribe`, `append` and `assertEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 115-118: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main(verbosity=3)
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Prefix caching / 前缀缓存
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `io`, `os`, `unittest`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.test_utils`
