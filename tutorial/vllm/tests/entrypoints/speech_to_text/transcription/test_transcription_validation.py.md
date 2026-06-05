# test_transcription_validation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/speech_to_text/transcription/test_transcription_validation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers speech-to-text serving. The file defines 3 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖语音转文本服务。它定义了 3 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L10)
```python
import json

import pytest

from tests.entrypoints.speech_to_text.conftest import add_attention_backend
from tests.utils import ROCM_ENV_OVERRIDES, ROCM_EXTRA_ARGS, RemoteOpenAIServer
```
**EN:** Imports standard-library modules such as `json`, third-party packages like `pytest`, project helpers such as `tests.entrypoints.speech_to_text.conftest.add_attention_backend`, `tests.utils.ROCM_ENV_OVERRIDES`, `tests.utils.ROCM_EXTRA_ARGS`.
**CN:** 导入标准库模块（如 `json`）、第三方包（如 `pytest`）、项目内辅助模块（如 `tests.entrypoints.speech_to_text.conftest.add_attention_backend`、`tests.utils.ROCM_ENV_OVERRIDES`、`tests.utils.ROCM_EXTRA_ARGS`）。

### Module setup / 模块级配置: MISTRAL_FORMAT_ARGS (L12-L19)
```python
MISTRAL_FORMAT_ARGS = [
    "--tokenizer_mode",
    "mistral",
    "--config_format",
    "mistral",
    "--load_format",
    "mistral",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MISTRAL_FORMAT_ARGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MISTRAL_FORMAT_ARGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: transcribe_and_check (L22-L62)
```python
async def transcribe_and_check(
    client,
    model_name: str,
    file,
    *,
    language: str,
    expected_text: str,
    expected_seconds: int | None = None,
    case_sensitive: bool = False,
):
    """Run a transcription request and assert the output contains
    *expected_text* and optionally that usage reports *expected_seconds*.

    Provides detailed failure messages with the actual transcription output.
    """
    transcription = await client.audio.transcriptions.create(
        model=model_name,
        file=file,
# ... 15 lines omitted for brevity ...
            f"output, got: {out_text!r}"
        )

    if expected_seconds is not None:
        assert out_usage["seconds"] == expected_seconds, (
            f"Expected {expected_seconds}s of audio, "
            f"got {out_usage['seconds']}s. Full usage: {out_usage!r}"
        )
```
**EN:** This async helper encapsulates reusable logic in `transcribe_and_check`. Key inputs are `client`, `model_name`, `file`. It drives client-facing request creation through the API surface under test. The main assertion is `expected_text in out_text` and `expected_text.lower() in out_text.lower()`.
**CN:** 这个异步辅助函数将可复用逻辑封装在 `transcribe_and_check` 中。 关键输入包括 `client`、`model_name`、`file`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `expected_text in out_text` and `expected_text.lower() in out_text.lower()`。

### Test / 测试: test_basic_audio (L65-L89)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name", ["mistralai/Voxtral-Mini-3B-2507", "Qwen/Qwen3-ASR-0.6B"]
)
async def test_basic_audio(mary_had_lamb, model_name, rocm_aiter_fa_attention):
    server_args = ["--enforce-eager", *ROCM_EXTRA_ARGS]

    if model_name.startswith("mistralai"):
        server_args += MISTRAL_FORMAT_ARGS

    add_attention_backend(server_args, rocm_aiter_fa_attention)

    # Based on https://github.com/openai/openai-cookbook/blob/main/examples/Whisper_prompting_guide.ipynb.
    with RemoteOpenAIServer(
        model_name, server_args, env_dict=ROCM_ENV_OVERRIDES
    ) as remote_server:
        client = remote_server.get_async_client()
        await transcribe_and_check(
            client,
            model_name,
            mary_had_lamb,
            language="en",
            expected_text="Mary had a little lamb",
            expected_seconds=16,
        )
```
**EN:** This async test validates `test_basic_audio`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`, `model_name`, `rocm_aiter_fa_attention`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint.
**CN:** 这个异步测试验证 `test_basic_audio`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`、`model_name`、`rocm_aiter_fa_attention`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。

### Test / 测试: test_basic_audio_with_lora (L92-L129)
```python
@pytest.mark.asyncio
async def test_basic_audio_with_lora(mary_had_lamb, rocm_aiter_fa_attention):
    """Ensure STT (transcribe) requests can pass LoRA through to generate."""
    # ROCm SPECIFIC CONFIGURATION:
    # To ensure the test passes on ROCm, we modify the max model length to 512.
    # We DO NOT apply this to other platforms to maintain strict upstream parity.
    from vllm.platforms import current_platform

    model_name = "ibm-granite/granite-speech-3.3-2b"
    lora_model_name = "speech"
    server_args = [
        "--enforce-eager",
        "--enable-lora",
        "--max-lora-rank",
        "64",
        "--lora-modules",
        f"{lora_model_name}={model_name}",
        "--max-model-len",
# ... 12 lines omitted for brevity ...
        await transcribe_and_check(
            client,
            lora_model_name,
            mary_had_lamb,
            language="en",
            expected_text="mary had a little lamb",
            expected_seconds=16,
        )
```
**EN:** This async test validates `test_basic_audio_with_lora`. Relevant pytest markers include `asyncio`. Key inputs are `mary_had_lamb`, `rocm_aiter_fa_attention`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint.
**CN:** 这个异步测试验证 `test_basic_audio_with_lora`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `mary_had_lamb`、`rocm_aiter_fa_attention`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。

### Test / 测试: test_basic_audio_foscolo (L132-L156)
```python
@pytest.mark.asyncio
@pytest.mark.parametrize(
    "model_name", ["google/gemma-3n-E2B-it", "Qwen/Qwen3-ASR-0.6B"]
)
async def test_basic_audio_foscolo(foscolo, rocm_aiter_fa_attention, model_name):
    # Gemma accuracy on some of the audio samples we use is particularly bad,
    # hence we use a different one here. WER is evaluated separately.
    server_args = ["--enforce-eager", *ROCM_EXTRA_ARGS]

    add_attention_backend(server_args, rocm_aiter_fa_attention)

    with RemoteOpenAIServer(
        model_name,
        server_args,
        max_wait_seconds=480,
        env_dict=ROCM_ENV_OVERRIDES,
    ) as remote_server:
        client = remote_server.get_async_client()
        await transcribe_and_check(
            client,
            model_name,
            foscolo,
            language="it",
            expected_text="ove il mio corpo fanciulletto",
        )
```
**EN:** This async test validates `test_basic_audio_foscolo`. It uses parameterization over `model_name`. Relevant pytest markers include `asyncio`. Key inputs are `foscolo`, `rocm_aiter_fa_attention`, `model_name`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint.
**CN:** 这个异步测试验证 `test_basic_audio_foscolo`。 它通过参数化组合 `model_name`。 相关的 pytest 标记包括 `asyncio`。 关键输入包括 `foscolo`、`rocm_aiter_fa_attention`、`model_name`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The scenarios exercise speech recognition or transcription endpoints.
  **CN:** 这些场景覆盖语音识别或转录端点。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `tests.entrypoints.speech_to_text.conftest.add_attention_backend`, `tests.utils.ROCM_ENV_OVERRIDES`, `tests.utils.ROCM_EXTRA_ARGS`, `tests.utils.RemoteOpenAIServer`, `vllm.platforms.current_platform`
