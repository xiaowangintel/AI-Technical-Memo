# test_qwen3_omni.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/processing/test_qwen3_omni.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 2 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 2 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L12)
```python
from typing import Any

import numpy as np
import pytest

from vllm.multimodal import MULTIMODAL_REGISTRY

from ...utils import build_model_context
```
**EN:** Imports standard-library modules such as `typing.Any`, third-party packages like `numpy`, `pytest`, project helpers such as `vllm.multimodal.MULTIMODAL_REGISTRY`, `...utils.build_model_context`.
**CN:** 导入标准库模块（如 `typing.Any`）、第三方包（如 `numpy`、`pytest`）、项目内辅助模块（如 `vllm.multimodal.MULTIMODAL_REGISTRY`、`...utils.build_model_context`）。

### Test / 测试: test_processor_with_audio_sample_rate (L15-L68)
```python
@pytest.mark.parametrize("model_id", ["Qwen/Qwen3-Omni-30B-A3B-Instruct"])
@pytest.mark.parametrize(
    ("audio_sample_rate", "audio_duration_sec"),
    [
        (16000, 1.0),  # Native Whisper sample rate, 1 second
        (16000, 2.0),  # Native Whisper sample rate, 2 seconds
    ],
)
def test_processor_with_audio_sample_rate(
    model_id: str,
    audio_sample_rate: int,
    audio_duration_sec: float,
) -> None:
    """
    Test that vLLM's processor generates expected outputs with audio_sample_rate.

    This validates that the processor correctly handles audio_sample_rate
    passed via hf_processor_mm_kwargs and generates audio tokens.
# ... 28 lines omitted for brevity ...
    hf_processor = processor.info.get_hf_processor(**hf_processor_mm_kwargs)
    audio_token_id = tokenizer.convert_tokens_to_ids(hf_processor.audio_token)
    aud_tok_count = processed_inputs["prompt_token_ids"].count(audio_token_id)

    assert aud_tok_count >= 1, (
        f"Expected at least 1 audio token but got {aud_tok_count}. "
        f"sample_rate: {audio_sample_rate}Hz, duration: {audio_duration_sec}s"
    )
```
**EN:** This test validates `test_processor_with_audio_sample_rate`. It uses parameterization over `model_id`. Key inputs are `model_id`, `audio_sample_rate`, `audio_duration_sec`. It drives client-facing request creation through the API surface under test. The main assertion is `aud_tok_count >= 1`.
**CN:** 这个测试验证 `test_processor_with_audio_sample_rate`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`、`audio_sample_rate`、`audio_duration_sec`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `aud_tok_count >= 1`。

### Test / 测试: test_longer_audio_generates_more_tokens (L71-L112)
```python
@pytest.mark.parametrize("model_id", ["Qwen/Qwen3-Omni-30B-A3B-Instruct"])
def test_longer_audio_generates_more_tokens(model_id: str) -> None:
    """
    Test that longer audio generates more tokens than shorter audio.

    This validates that audio_sample_rate is being used correctly by checking
    that audio duration affects token count as expected.
    """
    ctx = build_model_context(
        model_id,
        limit_mm_per_prompt={"audio": 1, "image": 0, "video": 0},
    )
    processor = MULTIMODAL_REGISTRY.create_processor(ctx.model_config)
    tokenizer = processor.info.get_tokenizer()

    audio_sample_rate = 16000
    rng = np.random.RandomState(42)

# ... 16 lines omitted for brevity ...

    short_tokens = get_token_count(1.0)
    long_tokens = get_token_count(2.0)

    assert long_tokens > short_tokens, (
        f"Expected longer audio (2s) to have more tokens than shorter (1s). "
        f"Got short={short_tokens}, long={long_tokens}"
    )
```
**EN:** This test validates `test_longer_audio_generates_more_tokens`. It uses parameterization over `model_id`. Key inputs are `model_id`. It drives client-facing request creation through the API surface under test. The main assertion is `long_tokens > short_tokens`.
**CN:** 这个测试验证 `test_longer_audio_generates_more_tokens`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `long_tokens > short_tokens`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing.Any`
- **Third-party / 第三方**: `numpy`, `pytest`
- **Project / 项目内**: `vllm.multimodal.MULTIMODAL_REGISTRY`
- **Local relative imports / 本地相对导入**: `...utils.build_model_context`
