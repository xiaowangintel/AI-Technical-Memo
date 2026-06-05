# test_run_batch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/openai/test_run_batch.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers OpenAI-compatible serving. The file defines 19 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖OpenAI 兼容服务。它定义了 19 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L15)
```python
import json
import subprocess
import tempfile
from unittest.mock import AsyncMock, MagicMock, patch

import pytest

from vllm.assets.audio import AudioAsset
from vllm.entrypoints.openai.run_batch import (
    BatchRequestOutput,
    download_bytes_from_url,
)
```
**EN:** Imports standard-library modules such as `json`, `subprocess`, `tempfile`, third-party packages like `pytest`, project helpers such as `vllm.assets.audio.AudioAsset`, `vllm.entrypoints.openai.run_batch.BatchRequestOutput`, `vllm.entrypoints.openai.run_batch.download_bytes_from_url`.
**CN:** 导入标准库模块（如 `json`、`subprocess`、`tempfile`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.assets.audio.AudioAsset`、`vllm.entrypoints.openai.run_batch.BatchRequestOutput`、`vllm.entrypoints.openai.run_batch.download_bytes_from_url`）。

### Module setup / 模块级配置: CHAT_MODEL_NAME, EMBEDDING_MODEL_NAME, RERANKER_MODEL_NAME (L17-L371)
```python
CHAT_MODEL_NAME = "hmellor/tiny-random-LlamaForCausalLM"
EMBEDDING_MODEL_NAME = "intfloat/multilingual-e5-small"
RERANKER_MODEL_NAME = "BAAI/bge-reranker-v2-m3"
REASONING_MODEL_NAME = "Qwen/Qwen3-0.6B"
SPEECH_LARGE_MODEL_NAME = "openai/whisper-large-v3"
SPEECH_SMALL_MODEL_NAME = "openai/whisper-small"

INPUT_BATCH = "\n".join(
    json.dumps(req)
    for req in [
        {
            "custom_id": "request-1",
            "method": "POST",
            "url": "/v1/chat/completions",
            "body": {
                "model": CHAT_MODEL_NAME,
# ... 333 lines omitted for brevity ...
            "tools": [WEATHER_TOOL],
            "tool_choice": "required",
            "max_tokens": 1000,
        },
    }
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `CHAT_MODEL_NAME`, `EMBEDDING_MODEL_NAME`, `RERANKER_MODEL_NAME`, `REASONING_MODEL_NAME`, `SPEECH_LARGE_MODEL_NAME`, `SPEECH_SMALL_MODEL_NAME`, `INPUT_BATCH`, `INVALID_INPUT_BATCH`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `CHAT_MODEL_NAME`、`EMBEDDING_MODEL_NAME`、`RERANKER_MODEL_NAME`、`REASONING_MODEL_NAME`、`SPEECH_LARGE_MODEL_NAME`、`SPEECH_SMALL_MODEL_NAME`、`INPUT_BATCH`、`INVALID_INPUT_BATCH`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_empty_file (L374-L398)
```python
def test_empty_file():
    with (
        tempfile.NamedTemporaryFile("w") as input_file,
        tempfile.NamedTemporaryFile("r") as output_file,
    ):
        input_file.write("")
        input_file.flush()
        proc = subprocess.Popen(
            [
                "vllm",
                "run-batch",
                "-i",
                input_file.name,
                "-o",
                output_file.name,
                "--model",
                EMBEDDING_MODEL_NAME,
            ],
        )
        proc.communicate()
        proc.wait()
        assert proc.returncode == 0, f"{proc=}"

        contents = output_file.read()
        assert contents.strip() == ""
```
**EN:** This test validates `test_empty_file`. The main assertion is `proc.returncode == 0` and `contents.strip() == ''`.
**CN:** 这个测试验证 `test_empty_file`。 核心断言是 `proc.returncode == 0` and `contents.strip() == ''`。

### Test / 测试: test_completions (L401-L428)
```python
def test_completions():
    with (
        tempfile.NamedTemporaryFile("w") as input_file,
        tempfile.NamedTemporaryFile("r") as output_file,
    ):
        input_file.write(INPUT_BATCH)
        input_file.flush()
        proc = subprocess.Popen(
            [
                "vllm",
                "run-batch",
                "-i",
                input_file.name,
                "-o",
                output_file.name,
                "--model",
                CHAT_MODEL_NAME,
            ],
        )
        proc.communicate()
        proc.wait()
        assert proc.returncode == 0, f"{proc=}"

        contents = output_file.read()
        for line in contents.strip().split("\n"):
            # Ensure that the output format conforms to the openai api.
            # Validation should throw if the schema is wrong.
            BatchRequestOutput.model_validate_json(line)
```
**EN:** This test validates `test_completions`. The main assertion is `proc.returncode == 0`.
**CN:** 这个测试验证 `test_completions`。 核心断言是 `proc.returncode == 0`。

### Test / 测试: test_completions_invalid_input (L431-L455)
```python
def test_completions_invalid_input():
    """
    Ensure that we fail when the input doesn't conform to the openai api.
    """
    with (
        tempfile.NamedTemporaryFile("w") as input_file,
        tempfile.NamedTemporaryFile("r") as output_file,
    ):
        input_file.write(INVALID_INPUT_BATCH)
        input_file.flush()
        proc = subprocess.Popen(
            [
                "vllm",
                "run-batch",
                "-i",
                input_file.name,
                "-o",
                output_file.name,
                "--model",
                CHAT_MODEL_NAME,
            ],
        )
        proc.communicate()
        proc.wait()
        assert proc.returncode != 0, f"{proc=}"
```
**EN:** This test validates `test_completions_invalid_input`. The main assertion is `proc.returncode != 0`.
**CN:** 这个测试验证 `test_completions_invalid_input`。 核心断言是 `proc.returncode != 0`。

### Test / 测试: test_embeddings (L458-L485)
```python
def test_embeddings():
    with (
        tempfile.NamedTemporaryFile("w") as input_file,
        tempfile.NamedTemporaryFile("r") as output_file,
    ):
        input_file.write(INPUT_EMBEDDING_BATCH)
        input_file.flush()
        proc = subprocess.Popen(
            [
                "vllm",
                "run-batch",
                "-i",
                input_file.name,
                "-o",
                output_file.name,
                "--model",
                EMBEDDING_MODEL_NAME,
            ],
        )
        proc.communicate()
        proc.wait()
        assert proc.returncode == 0, f"{proc=}"

        contents = output_file.read()
        for line in contents.strip().split("\n"):
            # Ensure that the output format conforms to the openai api.
            # Validation should throw if the schema is wrong.
            BatchRequestOutput.model_validate_json(line)
```
**EN:** This test validates `test_embeddings`. The main assertion is `proc.returncode == 0`.
**CN:** 这个测试验证 `test_embeddings`。 核心断言是 `proc.returncode == 0`。

### Test / 测试: test_score (L488-L521)
```python
@pytest.mark.parametrize("input_batch", [INPUT_SCORE_BATCH, INPUT_RERANK_BATCH])
def test_score(input_batch):
    with (
        tempfile.NamedTemporaryFile("w") as input_file,
        tempfile.NamedTemporaryFile("r") as output_file,
    ):
        input_file.write(input_batch)
        input_file.flush()
        proc = subprocess.Popen(
            [
                "vllm",
                "run-batch",
                "-i",
                input_file.name,
                "-o",
                output_file.name,
                "--model",
                RERANKER_MODEL_NAME,
# ... 8 lines omitted for brevity ...
            # Ensure that the output format conforms to the openai api.
            # Validation should throw if the schema is wrong.
            BatchRequestOutput.model_validate_json(line)

            # Ensure that there is no error in the response.
            line_dict = json.loads(line)
            assert isinstance(line_dict, dict)
            assert line_dict["error"] is None
```
**EN:** This test validates `test_score`. It uses parameterization over `input_batch`. Key inputs are `input_batch`. The main assertion is `proc.returncode == 0` and `isinstance(line_dict, dict)`.
**CN:** 这个测试验证 `test_score`。 它通过参数化组合 `input_batch`。 关键输入包括 `input_batch`。 核心断言是 `proc.returncode == 0` and `isinstance(line_dict, dict)`。

### Test / 测试: test_reasoning_parser (L524-L568)
```python
def test_reasoning_parser():
    """
    Test that reasoning_parser parameter works correctly in run_batch.
    """
    with (
        tempfile.NamedTemporaryFile("w") as input_file,
        tempfile.NamedTemporaryFile("r") as output_file,
    ):
        input_file.write(INPUT_REASONING_BATCH)
        input_file.flush()
        proc = subprocess.Popen(
            [
                "vllm",
                "run-batch",
                "-i",
                input_file.name,
                "-o",
                output_file.name,
# ... 19 lines omitted for brevity ...
            assert line_dict["error"] is None

            # Check that reasoning is present and not empty
            reasoning = line_dict["response"]["body"]["choices"][0]["message"][
                "reasoning"
            ]
            assert reasoning is not None
            assert len(reasoning) > 0
```
**EN:** This test validates `test_reasoning_parser`. The main assertion is `proc.returncode == 0` and `isinstance(line_dict, dict)`.
**CN:** 这个测试验证 `test_reasoning_parser`。 核心断言是 `proc.returncode == 0` and `isinstance(line_dict, dict)`。

### Test / 测试: test_transcription (L571-L606)
```python
def test_transcription():
    with (
        tempfile.NamedTemporaryFile("w") as input_file,
        tempfile.NamedTemporaryFile("r") as output_file,
    ):
        input_file.write(INPUT_TRANSCRIPTION_BATCH)
        input_file.flush()
        proc = subprocess.Popen(
            [
                "vllm",
                "run-batch",
                "-i",
                input_file.name,
                "-o",
                output_file.name,
                "--model",
                SPEECH_LARGE_MODEL_NAME,
            ],
# ... 10 lines omitted for brevity ...
            line_dict = json.loads(line)
            assert isinstance(line_dict, dict)
            assert line_dict["error"] is None

            response_body = line_dict["response"]["body"]
            assert response_body is not None
            assert "text" in response_body
            assert "usage" in response_body
```
**EN:** This test validates `test_transcription`. The main assertion is `proc.returncode == 0` and `isinstance(line_dict, dict)`.
**CN:** 这个测试验证 `test_transcription`。 核心断言是 `proc.returncode == 0` and `isinstance(line_dict, dict)`。

### Test / 测试: test_transcription_http_url (L609-L646)
```python
def test_transcription_http_url():
    with (
        tempfile.NamedTemporaryFile("w") as input_file,
        tempfile.NamedTemporaryFile("r") as output_file,
    ):
        input_file.write(INPUT_TRANSCRIPTION_HTTP_BATCH)
        input_file.flush()
        proc = subprocess.Popen(
            [
                "vllm",
                "run-batch",
                "-i",
                input_file.name,
                "-o",
                output_file.name,
                "--model",
                SPEECH_LARGE_MODEL_NAME,
            ],
# ... 12 lines omitted for brevity ...

            response_body = line_dict["response"]["body"]
            assert response_body is not None
            assert "text" in response_body
            assert "usage" in response_body

            transcription_text = response_body["text"]
            assert "Mary had a little lamb" in transcription_text
```
**EN:** This test validates `test_transcription_http_url`. The main assertion is `proc.returncode == 0` and `isinstance(line_dict, dict)`.
**CN:** 这个测试验证 `test_transcription_http_url`。 核心断言是 `proc.returncode == 0` and `isinstance(line_dict, dict)`。

### Test / 测试: test_translation (L649-L686)
```python
def test_translation():
    with (
        tempfile.NamedTemporaryFile("w") as input_file,
        tempfile.NamedTemporaryFile("r") as output_file,
    ):
        input_file.write(INPUT_TRANSLATION_BATCH)
        input_file.flush()
        proc = subprocess.Popen(
            [
                "vllm",
                "run-batch",
                "-i",
                input_file.name,
                "-o",
                output_file.name,
                "--model",
                SPEECH_SMALL_MODEL_NAME,
            ],
# ... 12 lines omitted for brevity ...

            response_body = line_dict["response"]["body"]
            assert response_body is not None
            assert "text" in response_body

            translation_text = response_body["text"]
            translation_text_lower = str(translation_text).strip().lower()
            assert "mary" in translation_text_lower or "lamb" in translation_text_lower
```
**EN:** This test validates `test_translation`. The main assertion is `proc.returncode == 0` and `isinstance(line_dict, dict)`.
**CN:** 这个测试验证 `test_translation`。 核心断言是 `proc.returncode == 0` and `isinstance(line_dict, dict)`。

### Test / 测试: test_tool_calling (L689-L752)
```python
def test_tool_calling():
    """
    Test that tool calling works correctly in run_batch.
    Verifies that requests with tools return tool_calls in the response.
    """
    with (
        tempfile.NamedTemporaryFile("w") as input_file,
        tempfile.NamedTemporaryFile("r") as output_file,
    ):
        input_file.write(INPUT_TOOL_CALLING_BATCH)
        input_file.flush()
        proc = subprocess.Popen(
            [
                "vllm",
                "run-batch",
                "-i",
                input_file.name,
                "-o",
# ... 38 lines omitted for brevity ...
                assert "id" in tool_call
                assert "type" in tool_call
                assert tool_call["type"] == "function"
                assert "function" in tool_call
                assert "name" in tool_call["function"]
                assert "arguments" in tool_call["function"]
                # Verify the tool name matches our tool definition
                assert tool_call["function"]["name"] == "get_current_weather"
```
**EN:** This test validates `test_tool_calling`. The main assertion is `proc.returncode == 0` and `isinstance(line_dict, dict)`.
**CN:** 这个测试验证 `test_tool_calling`。 核心断言是 `proc.returncode == 0` and `isinstance(line_dict, dict)`。

### Helper / 辅助函数: _make_aiohttp_mocks (L760-L772)
```python
def _make_aiohttp_mocks(response_data: bytes = b"fake-data", status: int = 200):
    """Create mock objects that simulate aiohttp.ClientSession context managers."""
    mock_resp = MagicMock()
    mock_resp.status = status
    mock_resp.read = AsyncMock(return_value=response_data)
    mock_resp.__aenter__ = AsyncMock(return_value=mock_resp)
    mock_resp.__aexit__ = AsyncMock(return_value=False)

    mock_session = MagicMock()
    mock_session.get = MagicMock(return_value=mock_resp)
    mock_session.__aenter__ = AsyncMock(return_value=mock_session)
    mock_session.__aexit__ = AsyncMock(return_value=False)
    return mock_session
```
**EN:** This helper encapsulates reusable logic in `_make_aiohttp_mocks`. Key inputs are `response_data`, `status`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_aiohttp_mocks` 中。 关键输入包括 `response_data`、`status`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_download_bytes_data_url_bypasses_domain_check (L775-L783)
```python
@pytest.mark.asyncio
async def test_download_bytes_data_url_bypasses_domain_check():
    """data: URLs must work regardless of the domain allowlist."""
    data_url = f"data:audio/wav;base64,{MINIMAL_WAV_BASE64}"
    result = await download_bytes_from_url(
        data_url, allowed_media_domains=["example.com"]
    )
    assert isinstance(result, bytes)
    assert len(result) > 0
```
**EN:** This async test validates `test_download_bytes_data_url_bypasses_domain_check`. Relevant pytest markers include `asyncio`. The main assertion is `isinstance(result, bytes)` and `len(result) > 0`.
**CN:** 这个异步测试验证 `test_download_bytes_data_url_bypasses_domain_check`。 相关的 pytest 标记包括 `asyncio`。 核心断言是 `isinstance(result, bytes)` and `len(result) > 0`。

### Test / 测试: test_download_bytes_rejects_disallowed_domain (L786-L791)
```python
@pytest.mark.asyncio
async def test_download_bytes_rejects_disallowed_domain():
    """HTTP URLs whose hostname is not in the allowlist must be rejected."""
    url = "https://evil.internal/secret"
    with pytest.raises(ValueError, match="allowed domains"):
        await download_bytes_from_url(url, allowed_media_domains=["example.com"])
```
**EN:** This async test validates `test_download_bytes_rejects_disallowed_domain`. Relevant pytest markers include `asyncio`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个异步测试验证 `test_download_bytes_rejects_disallowed_domain`。 相关的 pytest 标记包括 `asyncio`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_download_bytes_rejects_cloud_metadata_ip (L794-L799)
```python
@pytest.mark.asyncio
async def test_download_bytes_rejects_cloud_metadata_ip():
    """Cloud metadata endpoints must be blocked when an allowlist is set."""
    url = "http://169.254.169.254/latest/meta-data/"
    with pytest.raises(ValueError, match="allowed domains"):
        await download_bytes_from_url(url, allowed_media_domains=["example.com"])
```
**EN:** This async test validates `test_download_bytes_rejects_cloud_metadata_ip`. Relevant pytest markers include `asyncio`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个异步测试验证 `test_download_bytes_rejects_cloud_metadata_ip`。 相关的 pytest 标记包括 `asyncio`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_download_bytes_rejects_internal_ip (L802-L813)
```python
@pytest.mark.asyncio
async def test_download_bytes_rejects_internal_ip():
    """Private-range IPs must be blocked when an allowlist is set."""
    for internal_url in [
        "http://10.0.0.1/secret",
        "http://192.168.1.1/admin",
        "http://127.0.0.1:8080/internal",
    ]:
        with pytest.raises(ValueError, match="allowed domains"):
            await download_bytes_from_url(
                internal_url, allowed_media_domains=["example.com"]
            )
```
**EN:** This async test validates `test_download_bytes_rejects_internal_ip`. Relevant pytest markers include `asyncio`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个异步测试验证 `test_download_bytes_rejects_internal_ip`。 相关的 pytest 标记包括 `asyncio`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_download_bytes_allows_permitted_domain (L816-L830)
```python
@pytest.mark.asyncio
async def test_download_bytes_allows_permitted_domain():
    """HTTP URLs whose hostname IS in the allowlist must be fetched."""
    url = "https://example.com/audio.wav"
    expected = b"audio-bytes"
    mock_session = _make_aiohttp_mocks(expected)

    with patch(
        "vllm.entrypoints.openai.run_batch.aiohttp.ClientSession",
        return_value=mock_session,
    ):
        result = await download_bytes_from_url(
            url, allowed_media_domains=["example.com"]
        )
    assert result == expected
```
**EN:** This async test validates `test_download_bytes_allows_permitted_domain`. Relevant pytest markers include `asyncio`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `result == expected`.
**CN:** 这个异步测试验证 `test_download_bytes_allows_permitted_domain`。 相关的 pytest 标记包括 `asyncio`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `result == expected`。

### Test / 测试: test_download_bytes_no_allowlist_permits_any_domain (L833-L845)
```python
@pytest.mark.asyncio
async def test_download_bytes_no_allowlist_permits_any_domain():
    """Without an allowlist all HTTP URLs must be attempted (backward compat)."""
    url = "https://any-domain.example.org/file.wav"
    expected = b"some-data"
    mock_session = _make_aiohttp_mocks(expected)

    with patch(
        "vllm.entrypoints.openai.run_batch.aiohttp.ClientSession",
        return_value=mock_session,
    ):
        result = await download_bytes_from_url(url, allowed_media_domains=None)
    assert result == expected
```
**EN:** This async test validates `test_download_bytes_no_allowlist_permits_any_domain`. Relevant pytest markers include `asyncio`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `result == expected`.
**CN:** 这个异步测试验证 `test_download_bytes_no_allowlist_permits_any_domain`。 相关的 pytest 标记包括 `asyncio`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `result == expected`。

### Test / 测试: test_download_bytes_empty_allowlist_denies_all (L848-L853)
```python
@pytest.mark.asyncio
async def test_download_bytes_empty_allowlist_denies_all():
    """An empty allowlist must deny all HTTP URLs (least privilege)."""
    url = "https://any-domain.example.org/file.wav"
    with pytest.raises(ValueError, match="allowed domains"):
        await download_bytes_from_url(url, allowed_media_domains=[])
```
**EN:** This async test validates `test_download_bytes_empty_allowlist_denies_all`. Relevant pytest markers include `asyncio`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个异步测试验证 `test_download_bytes_empty_allowlist_denies_all`。 相关的 pytest 标记包括 `asyncio`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_download_bytes_unsupported_scheme (L856-L866)
```python
@pytest.mark.asyncio
async def test_download_bytes_unsupported_scheme():
    """Unsupported URL schemes must be rejected regardless of allowlist."""
    with pytest.raises(ValueError, match="Unsupported URL scheme"):
        await download_bytes_from_url("ftp://example.com/file.wav")

    with pytest.raises(ValueError, match="Unsupported URL scheme"):
        await download_bytes_from_url(
            "ftp://example.com/file.wav",
            allowed_media_domains=["example.com"],
        )
```
**EN:** This async test validates `test_download_bytes_unsupported_scheme`. Relevant pytest markers include `asyncio`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个异步测试验证 `test_download_bytes_unsupported_scheme`。 相关的 pytest 标记包括 `asyncio`。 它使用 `pytest.raises` 检查预期失败路径。

### Test / 测试: test_download_bytes_backslash_bypass (L869-L880)
```python
@pytest.mark.asyncio
async def test_download_bytes_backslash_bypass():
    """Backslash-@ URL confusion must not bypass the allowlist.

    urllib3.parse_url() and aiohttp/yarl disagree on backslash-before-@.
    The fix normalizes through urllib3 before handing to aiohttp.
    """
    bypass_url = "http://allowed.example.com\\@evil.internal/secret"
    with pytest.raises(ValueError, match="allowed domains"):
        await download_bytes_from_url(
            bypass_url, allowed_media_domains=["evil.internal"]
        )
```
**EN:** This async test validates `test_download_bytes_backslash_bypass`. Relevant pytest markers include `asyncio`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个异步测试验证 `test_download_bytes_backslash_bypass`。 相关的 pytest 标记包括 `asyncio`。 它使用 `pytest.raises` 检查预期失败路径。

## Key Concepts / 关键概念
- **EN:** Async functions verify coroutine-based control flow and concurrency.
  **CN:** 异步函数用于验证基于协程的控制流与并发行为。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The file focuses on OpenAI-compatible request and response behavior.
  **CN:** 该文件重点关注 OpenAI 兼容的请求与响应行为。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `subprocess`, `tempfile`, `unittest.mock.AsyncMock`, `unittest.mock.MagicMock`, `unittest.mock.patch`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.assets.audio.AudioAsset`, `vllm.entrypoints.openai.run_batch.BatchRequestOutput`, `vllm.entrypoints.openai.run_batch.download_bytes_from_url`
