# test_whisper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_whisper.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 5 test(s), 3 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 5 个测试、3 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L17)
```python
from collections.abc import Sequence
from typing import Any

import pytest
from transformers import AutoModelForSpeechSeq2Seq

from vllm.assets.audio import AudioAsset
from vllm.multimodal.audio import AudioResampler
from vllm.platforms import current_platform

from ....conftest import HfRunner, PromptAudioInput, VllmRunner
from ....utils import create_new_process_for_each_test, multi_gpu_test
from ...registry import HF_EXAMPLE_MODELS
from ...utils import check_logprobs_close
```
**EN:** Imports standard-library modules such as `collections.abc.Sequence`, `typing.Any`, `unittest.mock.MagicMock`, third-party packages like `pytest`, `transformers.AutoModelForSpeechSeq2Seq`, project helpers such as `vllm.assets.audio.AudioAsset`, `vllm.model_executor.models.whisper.WhisperForConditionalGeneration`, `vllm.multimodal.audio.AudioResampler`.
**CN:** 导入标准库模块（如 `collections.abc.Sequence`、`typing.Any`、`unittest.mock.MagicMock`）、第三方包（如 `pytest`、`transformers.AutoModelForSpeechSeq2Seq`）、项目内辅助模块（如 `vllm.assets.audio.AudioAsset`、`vllm.model_executor.models.whisper.WhisperForConditionalGeneration`、`vllm.multimodal.audio.AudioResampler`）。

### Module setup / 模块级配置: VLLM_PROMPT, HF_PROMPT, WHISPER_SAMPLE_RATE (L19-L22)
```python
VLLM_PROMPT = "<|startoftranscript|><|en|><|transcribe|><|notimestamps|>"
HF_PROMPT = ""
# Whisper expects 16kHz audio
WHISPER_SAMPLE_RATE = 16000
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `VLLM_PROMPT`, `HF_PROMPT`, `WHISPER_SAMPLE_RATE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `VLLM_PROMPT`、`HF_PROMPT`、`WHISPER_SAMPLE_RATE`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: use_spawn_for_whisper (L25-L28)
```python
@pytest.fixture(autouse=True)
def use_spawn_for_whisper(monkeypatch):
    """Whisper has issues with forked workers, use spawn instead."""
    monkeypatch.setenv("VLLM_WORKER_MULTIPROC_METHOD", "spawn")
```
**EN:** This fixture prepares `use_spawn_for_whisper` for dependent tests. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个 fixture 为依赖它的测试准备 `use_spawn_for_whisper`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Helper / 辅助函数: run_test (L31-L91)
```python
def run_test(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    inputs: Sequence[tuple[list[str], list[str], PromptAudioInput]],
    model: str,
    *,
    max_model_len: int,
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
    tensor_parallel_size: int,
    distributed_executor_backend: str | None = None,
    enforce_eager: bool = True,
    gpu_memory_utilization: float = 0.9,
) -> None:
    """Inference result should be the same between hf and vllm.

    All the audio fixtures for the test are from AudioAsset.
# ... 35 lines omitted for brevity ...

    for hf_outputs, vllm_outputs in zip(hf_outputs_per_case, vllm_outputs_per_case):
        check_logprobs_close(
            outputs_0_lst=hf_outputs,
            outputs_1_lst=vllm_outputs,
            name_0="hf",
            name_1="vllm",
        )
```
**EN:** This helper encapsulates reusable logic in `run_test`. Key inputs are `hf_runner`, `vllm_runner`, `inputs`, `model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_test` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`inputs`、`model`。

### Fixture / 夹具: resampled_assets (L94-L107)
```python
@pytest.fixture
def resampled_assets() -> list[tuple[Any, int]]:
    audio_assets = [AudioAsset("mary_had_lamb"), AudioAsset("winning_call")]
    sampled_assets = []
    resampler = AudioResampler(target_sr=WHISPER_SAMPLE_RATE)
    for asset in audio_assets:
        audio, orig_sr = asset.audio_and_sample_rate
        # Resample to Whisper's expected sample rate (16kHz)
        if orig_sr != WHISPER_SAMPLE_RATE:
            audio = resampler.resample(audio, orig_sr=orig_sr)
        sampled_assets.append(
            (audio, WHISPER_SAMPLE_RATE),
        )
    return sampled_assets
```
**EN:** This fixture prepares `resampled_assets` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `resampled_assets`。

### Fixture / 夹具: input_audios (L110-L119)
```python
@pytest.fixture
def input_audios(
    resampled_assets,
) -> list[tuple[list[str], list[str], list[tuple[Any, int]]]]:
    inputs = []
    # audio assets are resampled to WHISPER_SAMPLE_RATE
    for audio_info in resampled_assets:
        # vLLM prompts, HF prompts, audio inputs
        inputs.append(([VLLM_PROMPT], [HF_PROMPT], [audio_info]))
    return inputs
```
**EN:** This fixture prepares `input_audios` for dependent tests. Key inputs are `resampled_assets`.
**CN:** 这个 fixture 为依赖它的测试准备 `input_audios`。 关键输入包括 `resampled_assets`。

### Helper / 辅助函数: check_model_available (L122-L125)
```python
def check_model_available(model: str) -> None:
    model_info = HF_EXAMPLE_MODELS.find_hf_info(model)
    model_info.check_available_online(on_fail="skip")
    model_info.check_transformers_version(on_fail="skip")
```
**EN:** This helper encapsulates reusable logic in `check_model_available`. Key inputs are `model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `check_model_available` 中。 关键输入包括 `model`。

### Test / 测试: test_beam_search_encoder_decoder (L128-L217)
```python
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("beam_width", [1, 2])
def test_beam_search_encoder_decoder(
    monkeypatch,
    hf_runner,
    vllm_runner,
    dtype: str,
    max_tokens: int,
    beam_width: int,
    resampled_assets,
) -> None:
    """Test beam search with encoder-decoder models (Whisper)."""
    if current_platform.is_rocm():
        monkeypatch.setenv("VLLM_ROCM_USE_SKINNY_GEMM", "0")

    model = "openai/whisper-large-v3-turbo"
    check_model_available(model)
# ... 64 lines omitted for brevity ...
        # 3. Outputs are reasonable (non-empty, diverse beams)
        for j in range(len(vllm_output_ids)):
            # Check that outputs are not empty
            assert len(vllm_output_ids[j]) > 0, f"Prompt {i}, beam {j}: empty output"
            # Check that decoded text is not empty
            assert len(vllm_output_texts[j].strip()) > 0, (
                f"Prompt {i}, beam {j}: empty text output"
            )
```
**EN:** This test validates `test_beam_search_encoder_decoder`. It uses parameterization over `dtype`. Key inputs are `monkeypatch`, `hf_runner`, `vllm_runner`, `dtype`, `max_tokens`, `beam_width`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `len(hf_output_ids) == len(vllm_output_ids)` and `len(vllm_output_ids[j]) > 0`.
**CN:** 这个测试验证 `test_beam_search_encoder_decoder`。 它通过参数化组合 `dtype`。 关键输入包括 `monkeypatch`、`hf_runner`、`vllm_runner`、`dtype`、`max_tokens`、`beam_width`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `len(hf_output_ids) == len(vllm_output_ids)` and `len(vllm_output_ids[j]) > 0`。

### Test / 测试: test_parse_language_detection_output (L220-L258)
```python
def test_parse_language_detection_output():
    """Unit test for WhisperForConditionalGeneration.parse_language_detection_output.

    No GPU or model loading required.
    """
    from unittest.mock import MagicMock

    from vllm.model_executor.models.whisper import (
        WhisperForConditionalGeneration,
    )

    cls = WhisperForConditionalGeneration

    def make_tokenizer(return_value: str) -> MagicMock:
        tok = MagicMock()
        tok.decode = MagicMock(return_value=return_value)
        return tok

# ... 13 lines omitted for brevity ...

    # No special token format
    with pytest.raises(AssertionError):
        cls.parse_language_detection_output([1], make_tokenizer("hello"))

    # Empty token_ids
    with pytest.raises((AssertionError, IndexError)):
        cls.parse_language_detection_output([], make_tokenizer("anything"))
```
**EN:** This test validates `test_parse_language_detection_output`. It checks an expected failure path with `pytest.raises`. The main assertion is `cls.parse_language_detection_output([50259], make_tokenizer('<|en|>')) == 'en'` and `cls.parse_language_detection_output([50261], make_tokenizer('<|de|>')) == 'de'`.
**CN:** 这个测试验证 `test_parse_language_detection_output`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `cls.parse_language_detection_output([50259], make_tokenizer('<|en|>')) == 'en'` and `cls.parse_language_detection_output([50261], make_tokenizer('<|de|>')) == 'de'`。

### Test / 测试: test_models (L261-L290)
```python
@pytest.mark.core_model
@pytest.mark.cpu_model
@pytest.mark.parametrize("model", ["openai/whisper-large-v3-turbo"])
@pytest.mark.parametrize("dtype", ["half", "float"])
@pytest.mark.parametrize("num_logprobs", [5])
@pytest.mark.parametrize("enforce_eager", [True, False])
def test_models(
    hf_runner,
    vllm_runner,
    model: str,
    dtype: str,
    num_logprobs: int,
    input_audios,
    enforce_eager: bool,
) -> None:
    check_model_available(model)
    if current_platform.is_cpu() and not enforce_eager:
        pytest.skip("Skipping test for CPU with non-eager mode")
    run_test(
        hf_runner,
        vllm_runner,
        input_audios,
        model,
        dtype=dtype,
        max_model_len=448,
        max_tokens=200,
        num_logprobs=num_logprobs,
        tensor_parallel_size=1,
        enforce_eager=enforce_eager,
    )
```
**EN:** This test validates `test_models`. It uses parameterization over `model`. Relevant pytest markers include `core_model`, `cpu_model`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `dtype`, `num_logprobs`, `input_audios`. The logic can skip unsupported environments when prerequisites are not satisfied.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `core_model`、`cpu_model`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`dtype`、`num_logprobs`、`input_audios`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。

### Test / 测试: test_models_distributed (L293-L325)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.core_model
@pytest.mark.parametrize("model", ["openai/whisper-large-v3-turbo"])
@pytest.mark.parametrize("distributed_executor_backend", ["ray", "mp"])
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.parametrize("max_tokens", [200])
@pytest.mark.parametrize("num_logprobs", [5])
@create_new_process_for_each_test("spawn")
def test_models_distributed(
    hf_runner,
    vllm_runner,
    model: str,
    distributed_executor_backend: str,
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
    input_audios,
) -> None:
# ... 7 lines omitted for brevity ...
        max_model_len=448,
        max_tokens=max_tokens,
        num_logprobs=num_logprobs,
        tensor_parallel_size=2,
        distributed_executor_backend=distributed_executor_backend,
        enforce_eager=False,
        gpu_memory_utilization=0.65,
    )
```
**EN:** This test validates `test_models_distributed`. It uses parameterization over `model`. Relevant pytest markers include `core_model`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `distributed_executor_backend`, `dtype`, `max_tokens`.
**CN:** 这个测试验证 `test_models_distributed`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `core_model`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`distributed_executor_backend`、`dtype`、`max_tokens`。

### Test / 测试: test_encoder_cache_cleanup (L328-L368)
```python
@pytest.mark.core_model
@pytest.mark.parametrize("model", ["openai/whisper-large-v3-turbo"])
def test_encoder_cache_cleanup(
    vllm_runner,
    model: str,
    input_audios,
    monkeypatch,
) -> None:
    """Test that encoder cache is properly cleaned up after requests complete.

    This is a regression test for a bug where encoder cache entries were freed
    in the same scheduling step they were allocated, before the model could use
    them.
    """
    # Set single-process mode to access the model runner's encoder cache directly
    monkeypatch.setenv("VLLM_ENABLE_V1_MULTIPROCESSING", "0")
    check_model_available(model)

# ... 15 lines omitted for brevity ...

        # After all requests complete, encoder cache should be empty
        cache_size = len(encoder_cache)
        assert cache_size == 0, (
            f"Encoder cache should be empty after all requests complete, "
            f"but has {cache_size} entries. This indicates encoder cache "
            f"entries are not being properly freed."
        )
```
**EN:** This test validates `test_encoder_cache_cleanup`. It uses parameterization over `model`. Relevant pytest markers include `core_model`. Key inputs are `vllm_runner`, `model`, `input_audios`, `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `cache_size == 0`.
**CN:** 这个测试验证 `test_encoder_cache_cleanup`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `core_model`。 关键输入包括 `vllm_runner`、`model`、`input_audios`、`monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `cache_size == 0`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Sequence`, `typing.Any`, `unittest.mock.MagicMock`
- **Third-party / 第三方**: `pytest`, `transformers.AutoModelForSpeechSeq2Seq`
- **Project / 项目内**: `vllm.assets.audio.AudioAsset`, `vllm.model_executor.models.whisper.WhisperForConditionalGeneration`, `vllm.multimodal.audio.AudioResampler`, `vllm.platforms.current_platform`
- **Local relative imports / 本地相对导入**: `....conftest.HfRunner`, `....conftest.PromptAudioInput`, `....conftest.VllmRunner`, `....utils.create_new_process_for_each_test`, `....utils.multi_gpu_test`, `...registry.HF_EXAMPLE_MODELS`, `...utils.check_logprobs_close`
