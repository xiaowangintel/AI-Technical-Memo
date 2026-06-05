# conftest.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/speech_to_text/conftest.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared pytest fixtures and hooks for speech-to-text serving. / [CN] 为语音转文本服务提供共享的 pytest fixture 与钩子。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L5)
```python
import pytest

from vllm.assets.audio import AudioAsset
```
**EN:** Imports third-party packages like `pytest`, project helpers such as `vllm.assets.audio.AudioAsset`, `vllm.platforms.current_platform`.
**CN:** 导入第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.assets.audio.AudioAsset`、`vllm.platforms.current_platform`）。

### Helper / 辅助函数: add_attention_backend (L8-L16)
```python
def add_attention_backend(server_args, attention_config):
    """Append attention backend CLI arg if specified.

    Args:
        server_args: List of server arguments to extend in-place.
        attention_config: Dict with 'backend' key, or None.
    """
    if attention_config and "backend" in attention_config:
        server_args.extend(["--attention-backend", attention_config["backend"]])
```
**EN:** This helper encapsulates reusable logic in `add_attention_backend`. Key inputs are `server_args`, `attention_config`.
**CN:** 这个辅助函数将可复用逻辑封装在 `add_attention_backend` 中。 关键输入包括 `server_args`、`attention_config`。

### Fixture / 夹具: rocm_aiter_fa_attention (L19-L29)
```python
@pytest.fixture(scope="module")
def rocm_aiter_fa_attention():
    """Return attention config for transcription/translation tests on ROCm.

    On ROCm, audio tests require ROCM_AITER_FA attention backend.
    """
    from vllm.platforms import current_platform

    if current_platform.is_rocm():
        return {"backend": "ROCM_AITER_FA"}
    return None
```
**EN:** This fixture prepares `rocm_aiter_fa_attention` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `rocm_aiter_fa_attention`。

### Fixture / 夹具: mary_had_lamb (L32-L36)
```python
@pytest.fixture
def mary_had_lamb():
    path = AudioAsset("mary_had_lamb").get_local_path()
    with open(str(path), "rb") as f:
        yield f
```
**EN:** This fixture prepares `mary_had_lamb` for dependent tests. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `mary_had_lamb`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: winning_call (L39-L43)
```python
@pytest.fixture
def winning_call():
    path = AudioAsset("winning_call").get_local_path()
    with open(str(path), "rb") as f:
        yield f
```
**EN:** This fixture prepares `winning_call` for dependent tests. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `winning_call`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Fixture / 夹具: foscolo (L46-L51)
```python
@pytest.fixture
def foscolo():
    # Test translation it->en
    path = AudioAsset("azacinto_foscolo").get_local_path()
    with open(str(path), "rb") as f:
        yield f
```
**EN:** This fixture prepares `foscolo` for dependent tests. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `foscolo`。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** The scenarios exercise speech recognition or transcription endpoints.
  **CN:** 这些场景覆盖语音识别或转录端点。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.assets.audio.AudioAsset`, `vllm.platforms.current_platform`
