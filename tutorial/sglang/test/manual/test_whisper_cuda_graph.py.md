# test_whisper_cuda_graph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_whisper_cuda_graph.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `whisper cuda graph` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `whisper cuda graph` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Constants and scenario settings / 常量与场景配置
```python
"""
Test Whisper model with CUDA graph support.

This test verifies that:
1. Whisper model works correctly with CUDA graph enabled (default)
2. Cross-attention KV cache is properly managed through RadixAttention
3. Output is consistent between CUDA graph and non-CUDA-graph modes

Usage:
    python test_whisper_cuda_graph.py

Requires:
    - A GPU with sufficient memory
    - openai-whisper model (e.g., openai/whisper-large-v3)
    - An audio file or URL for testing
"""

import io
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)

WHISPER_MODEL = "openai/whisper-large-v3"
TEST_AUDIO_URL = "https://huggingface.co/datasets/Narsil/asr_dummy/resolve/main/1.flac"
TEST_AUDIO_LOCAL = "/tmp/test_whisper_audio.flac"
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `enabled` and `model`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 36-47: Helper routines around get_audio_bytes / 辅助例程
```python
def get_audio_bytes():
    """Get audio bytes, downloading if necessary."""
    import os

    if os.path.exists(TEST_AUDIO_LOCAL):
        with open(TEST_AUDIO_LOCAL, "rb") as f:
            return f.read()
    resp = requests.get(TEST_AUDIO_URL, timeout=30)
    resp.raise_for_status()
    with open(TEST_AUDIO_LOCAL, "wb") as f:
        f.write(resp.content)
    return resp.content
```
**EN:** This range implements helper routine(s) `get_audio_bytes` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `exists`, `read`, `get` and `raise_for_status`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 50-53: Class definition for TestWhisperCudaGraph / 类定义
```python
class TestWhisperCudaGraph(CustomTestCase):
    """Test Whisper with CUDA graph enabled (default behavior)."""

    @classmethod
```
**EN:** This range declares `TestWhisperCudaGraph`, which organizes the scenario as a reusable test-oriented class. Representative call sites include `enabled`.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 54-65: Lifecycle helpers / 生命周期辅助逻辑
```python
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
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 66-67: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 68-69: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 71-74: Helper routines around _transcribe / 辅助例程
```python
    def _transcribe(self, language="en"):
        """Send a transcription request via OpenAI-compatible audio endpoint."""
        audio_bytes = get_audio_bytes()
        response = requests.post(
```
**EN:** This range implements helper routine(s) `_transcribe` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `get_audio_bytes` and `post`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 75-83: Assertions and result checks / 断言与结果检查
```python
            self.base_url + "/v1/audio/transcriptions",
            files={"file": ("audio.ogg", io.BytesIO(audio_bytes), "audio/ogg")},
            data={
                "model": "whisper",
                "language": language,
            },
        )
        self.assertEqual(response.status_code, 200, response.text)
        return response.json()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `BytesIO`, `assertEqual` and `json`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 85-91: Test routines around test_basic_transcription / 测试例程
```python
    def test_basic_transcription(self):
        """Test that basic transcription works with CUDA graph."""
        result = self._transcribe()
        self.assertIn("text", result)
        text = result["text"]
        self.assertTrue(len(text) > 0, "Transcription should not be empty")
        print(f"Transcription: {text}")
```
**EN:** This range defines concrete test routine(s) `test_basic_transcription`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `_transcribe`, `assertIn` and `assertTrue`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 93-99: Test routines around test_multiple_sequential_requests / 测试例程
```python
    def test_multiple_sequential_requests(self):
        """Test multiple sequential requests to verify CUDA graph replay consistency."""
        results = []
        for i in range(3):
            result = self._transcribe()
            self.assertIn("text", result)
            results.append(result["text"])
```
**EN:** This range defines concrete test routine(s) `test_multiple_sequential_requests`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `_transcribe`, `assertIn` and `append`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 100-108: Assertions and result checks / 断言与结果检查
```python
            print(f"Request {i+1}: {result['text'][:80]}...")

        # All transcriptions of the same audio should be identical
        for i in range(1, len(results)):
            self.assertEqual(
                results[0],
                results[i],
                f"Transcription {i+1} differs from first transcription",
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 110-117: Test routines around test_transcription_quality / 测试例程
```python
    def test_transcription_quality(self):
        """Test that transcription quality is reasonable (contains expected words)."""
        result = self._transcribe()
        text = result["text"].lower()
        # The test audio is a LibriSpeech sample about stew for dinner
        self.assertIn("stew", text, f"Expected 'stew' in transcription: {text}")
        self.assertIn("dinner", text, f"Expected 'dinner' in transcription: {text}")
        print(f"Quality check passed: {result['text'][:80]}...")
```
**EN:** This range defines concrete test routine(s) `test_transcription_quality`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `reasonable`, `_transcribe`, `lower` and `assertIn`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 120-123: Class definition for TestWhisperNoCudaGraph / 类定义
```python
class TestWhisperNoCudaGraph(CustomTestCase):
    """Test Whisper with CUDA graph explicitly disabled for comparison."""

    @classmethod
```
**EN:** This range declares `TestWhisperNoCudaGraph`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 124-136: Lifecycle helpers / 生命周期辅助逻辑
```python
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
                "--disable-cuda-graph",
            ],
        )
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 137-138: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 139-140: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 142-144: Test routines around test_basic_transcription_no_cuda_graph / 测试例程
```python
    def test_basic_transcription_no_cuda_graph(self):
        """Test that transcription works without CUDA graph (baseline)."""
        audio_bytes = get_audio_bytes()
```
**EN:** This range defines concrete test routine(s) `test_basic_transcription_no_cuda_graph`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `graph` and `get_audio_bytes`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 145-157: Assertions and result checks / 断言与结果检查
```python
        response = requests.post(
            self.base_url + "/v1/audio/transcriptions",
            files={"file": ("audio.ogg", io.BytesIO(audio_bytes), "audio/ogg")},
            data={
                "model": "whisper",
                "language": "en",
            },
        )
        self.assertEqual(response.status_code, 200, response.text)
        result = response.json()
        self.assertIn("text", result)
        self.assertTrue(len(result["text"]) > 0)
        print(f"No CUDA graph transcription: {result['text'][:80]}...")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. It interacts with request/response APIs to observe live model behavior. Assertions in this block enforce the intended outcome. Representative call sites include `post`, `BytesIO`, `assertEqual` and `json`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 158-161: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main(verbosity=3)
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Prefix caching / 前缀缓存
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `io`, `os`, `unittest`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.test_utils`
