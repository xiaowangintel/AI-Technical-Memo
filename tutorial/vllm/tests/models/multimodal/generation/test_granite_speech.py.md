# test_granite_speech.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_granite_speech.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 1 test(s), 1 fixture(s), and 2 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 1 个测试、1 个 fixture，以及 2 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L15)
```python
from collections.abc import Sequence

import pytest
from transformers import AutoModelForSpeechSeq2Seq

from vllm.logprobs import SampleLogprobs
from vllm.lora.request import LoRARequest
from vllm.platforms import current_platform

from ....conftest import AudioTestAssets, HfRunner, PromptAudioInput, VllmRunner
from ...registry import HF_EXAMPLE_MODELS
from ...utils import check_logprobs_close
```
**EN:** Imports standard-library modules such as `collections.abc.Sequence`, third-party packages like `pytest`, `transformers.AutoModelForSpeechSeq2Seq`, project helpers such as `vllm.logprobs.SampleLogprobs`, `vllm.lora.request.LoRARequest`, `vllm.platforms.current_platform`.
**CN:** 导入标准库模块（如 `collections.abc.Sequence`）、第三方包（如 `pytest`、`transformers.AutoModelForSpeechSeq2Seq`）、项目内辅助模块（如 `vllm.logprobs.SampleLogprobs`、`vllm.lora.request.LoRARequest`、`vllm.platforms.current_platform`）。

### Module setup / 模块级配置: HF_AUDIO_PROMPT (L17-L17)
```python
HF_AUDIO_PROMPT = "<|start_of_role|>system<|end_of_role|>Knowledge Cutoff Date: April 2024.\nToday's Date: December 19, 2024.\nYou are Granite, developed by IBM. You are a helpful AI assistant<|end_of_text|>\n<|start_of_role|>user<|end_of_role|><|audio|>can you transcribe the speech into a written format?<|end_of_text|>\n<|start_of_role|>assistant<|end_of_role|>"  # noqa: E501
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `HF_AUDIO_PROMPT`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `HF_AUDIO_PROMPT`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: vllm_to_hf_output (L20-L28)
```python
def vllm_to_hf_output(
    vllm_output: tuple[list[int], str, SampleLogprobs | None],
) -> tuple[list[int], str, SampleLogprobs | None]:
    """Sanitize hf output to be comparable with vllm output."""
    output_ids, output_str, out_logprobs = vllm_output

    hf_output_str = output_str + "<|end_of_text|>"

    return output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `vllm_to_hf_output`. Key inputs are `vllm_output`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `vllm_to_hf_output` 中。 关键输入包括 `vllm_output`。 它把计算得到的状态或辅助对象返回给调用方。

### Module setup / 模块级配置: MODEL_NAME, MODEL_NAME_4_0, models (L31-L38)
```python
MODEL_NAME = "ibm-granite/granite-speech-3.3-2b"
MODEL_NAME_4_0 = "ibm-granite/granite-4.0-1b-speech"
# Audio lora co-exists directly in the 3.3 model directory,
# the 4.0 model has adapters merged into the weights.
models: dict[str, str | None] = {
    MODEL_NAME: MODEL_NAME,
    MODEL_NAME_4_0: None,
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `MODEL_NAME_4_0`, `models`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`MODEL_NAME_4_0`、`models`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: granite_speech_attention_config (L41-L50)
```python
@pytest.fixture
def granite_speech_attention_config():
    """Return attention config for Granite Speech tests on ROCm."""
    if current_platform.is_rocm():
        from vllm.platforms.rocm import on_mi3xx

        if on_mi3xx():
            return {"backend": "ROCM_AITER_FA"}
        return {"backend": "TRITON_ATTN"}
    return None
```
**EN:** This fixture prepares `granite_speech_attention_config` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `granite_speech_attention_config`。

### Helper / 辅助函数: run_test (L53-L131)
```python
def run_test(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    inputs: Sequence[tuple[list[str], PromptAudioInput]],
    model: str,
    *,
    max_model_len: int,
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
    tensor_parallel_size: int,
    distributed_executor_backend: str | None = None,
    attention_config: dict | None = None,
    audio_lora_path: str | None = None,
):
    """Inference result should be the same between hf and vllm.

    All the audio fixtures for the test are from AUDIO_ASSETS.
# ... 53 lines omitted for brevity ...

    for hf_outputs, vllm_outputs in zip(hf_outputs_per_case, vllm_outputs_per_case):
        check_logprobs_close(
            outputs_0_lst=hf_outputs,
            outputs_1_lst=[vllm_to_hf_output(output) for output in vllm_outputs],
            name_0="hf",
            name_1="vllm",
        )
```
**EN:** This helper encapsulates reusable logic in `run_test`. Key inputs are `hf_runner`, `vllm_runner`, `inputs`, `model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_test` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`inputs`、`model`。

### Test / 测试: test_models (L134-L178)
```python
@pytest.mark.parametrize("model,audio_lora_path", models.items())
@pytest.mark.parametrize(
    "dtype", ["float16"] if current_platform.is_rocm() else ["bfloat16"]
)
@pytest.mark.parametrize(
    "max_model_len", [512] if current_platform.is_rocm() else [2048]
)
@pytest.mark.parametrize("max_tokens", [128])
@pytest.mark.parametrize("num_logprobs", [10])
def test_models(
    hf_runner,
    vllm_runner,
    model: str,
    audio_lora_path: str | None,
    audio_assets: AudioTestAssets,
    granite_speech_attention_config,
    dtype: str,
    max_model_len: int,
# ... 19 lines omitted for brevity ...
        dtype=dtype,
        max_model_len=max_model_len,
        max_tokens=max_tokens,
        num_logprobs=num_logprobs,
        tensor_parallel_size=1,
        attention_config=granite_speech_attention_config,
        audio_lora_path=audio_lora_path,
    )
```
**EN:** This test validates `test_models`. It uses parameterization over `model`, `audio_lora_path`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `audio_lora_path`, `audio_assets`, `granite_speech_attention_config`. The main assertion is `sr == 16000`.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`、`audio_lora_path`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`audio_lora_path`、`audio_assets`、`granite_speech_attention_config`。 核心断言是 `sr == 16000`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Sequence`
- **Third-party / 第三方**: `pytest`, `transformers.AutoModelForSpeechSeq2Seq`
- **Project / 项目内**: `vllm.logprobs.SampleLogprobs`, `vllm.lora.request.LoRARequest`, `vllm.platforms.current_platform`, `vllm.platforms.rocm.on_mi3xx`
- **Local relative imports / 本地相对导入**: `....conftest.AudioTestAssets`, `....conftest.HfRunner`, `....conftest.PromptAudioInput`, `....conftest.VllmRunner`, `...registry.HF_EXAMPLE_MODELS`, `...utils.check_logprobs_close`
