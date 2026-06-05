# test_phi4mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_phi4mm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 3 test(s), 0 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 3 个测试、0 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L26)
```python
import os
from collections.abc import Sequence

import pytest
import regex as re
from huggingface_hub import snapshot_download
from transformers import AutoTokenizer

from vllm.assets.image import ImageAsset
from vllm.logprobs import SampleLogprobs
from vllm.lora.request import LoRARequest
from vllm.multimodal.image import convert_image_mode, rescale_image_size
from vllm.multimodal.media.audio import load_audio

from ....conftest import (
    IMAGE_ASSETS,
    HfRunner,
    PromptAudioInput,
    PromptImageInput,
    VllmRunner,
)
from ....utils import large_gpu_test
from ...utils import check_logprobs_close
```
**EN:** Imports standard-library modules such as `collections.abc.Sequence`, `os`, third-party packages like `huggingface_hub.snapshot_download`, `pytest`, `regex`, project helpers such as `vllm.assets.image.ImageAsset`, `vllm.logprobs.SampleLogprobs`, `vllm.lora.request.LoRARequest`.
**CN:** 导入标准库模块（如 `collections.abc.Sequence`、`os`）、第三方包（如 `huggingface_hub.snapshot_download`、`pytest`、`regex`）、项目内辅助模块（如 `vllm.assets.image.ImageAsset`、`vllm.logprobs.SampleLogprobs`、`vllm.lora.request.LoRARequest`）。

### Module setup / 模块级配置: HF_IMAGE_PROMPTS, HF_MULTIIMAGE_IMAGE_PROMPT, model_path (L28-L45)
```python
HF_IMAGE_PROMPTS = IMAGE_ASSETS.prompts(
    {
        "stop_sign": "<|user|>\n<|image_1|>\nWhat's the content of the image?<|end|>\n<|assistant|>\n",  # noqa: E501
        "cherry_blossom": "<|user|>\n<|image_1|>\nPlease infer the season with reason in details.<|end|>\n<|assistant|>\n",  # noqa: E501
    }
)
HF_MULTIIMAGE_IMAGE_PROMPT = (
    "<|user|>\n<|image_1|>\n<|image_2|>\nDescribe these images.<|end|>\n<|assistant|>\n"  # noqa: E501
)

model_path = snapshot_download("microsoft/Phi-4-multimodal-instruct")
# Since the vision-lora and speech-lora co-exist with the base model,
# we have to manually specify the path of the lora weights.
vision_lora_path = os.path.join(model_path, "vision-lora")
speech_question = os.path.join(
    model_path, "examples", "what_is_shown_in_this_image.wav"
)
models = [model_path]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `HF_IMAGE_PROMPTS`, `HF_MULTIIMAGE_IMAGE_PROMPT`, `model_path`, `vision_lora_path`, `speech_question`, `models`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `HF_IMAGE_PROMPTS`、`HF_MULTIIMAGE_IMAGE_PROMPT`、`model_path`、`vision_lora_path`、`speech_question`、`models`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: vllm_to_hf_output (L48-L65)
```python
def vllm_to_hf_output(
    vllm_output: tuple[list[int], str, SampleLogprobs | None], model: str
):
    """Sanitize vllm output to be comparable with hf output."""
    _, output_str, out_logprobs = vllm_output

    output_str_without_image = re.sub(r"(<\|image_\d+\|>)+", "", output_str)
    assert output_str_without_image[0] == " "
    output_str_without_image = output_str_without_image[1:]

    hf_output_str = output_str_without_image + "<|end|><|endoftext|>"

    tokenizer = AutoTokenizer.from_pretrained(model)
    hf_output_ids = tokenizer.encode(output_str_without_image)
    assert hf_output_ids[0] == 1
    hf_output_ids = hf_output_ids[1:]

    return hf_output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller. The main assertion is `output_str_without_image[0] == ' '` and `hf_output_ids[0] == 1`.
**CN:** 这个辅助函数将可复用逻辑封装在 `vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `output_str_without_image[0] == ' '` and `hf_output_ids[0] == 1`。

### Module setup / 模块级配置: target_dtype (L68-L68)
```python
target_dtype = "half"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `target_dtype`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `target_dtype`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: run_test (L71-L166)
```python
def run_test(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    inputs: Sequence[tuple[list[str], PromptImageInput, PromptAudioInput | None]],
    model: str,
    *,
    max_model_len: int,
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
    mm_limit: int,
    tensor_parallel_size: int,
    distributed_executor_backend: str | None = None,
):
    """Inference result should be the same between hf and vllm.

    All the image fixtures for the test are from IMAGE_ASSETS.
    For huggingface runner, we provide the PIL images as input.
# ... 70 lines omitted for brevity ...

    for hf_outputs, vllm_outputs in zip(hf_outputs_per_case, vllm_outputs_per_case):
        check_logprobs_close(
            outputs_0_lst=hf_outputs,
            outputs_1_lst=vllm_outputs,
            name_0="hf",
            name_1="vllm",
        )
```
**EN:** This helper encapsulates reusable logic in `run_test`. Key inputs are `hf_runner`, `vllm_runner`, `inputs`, `model`. The logic can skip unsupported environments when prerequisites are not satisfied. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_test` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`inputs`、`model`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_models (L169-L218)
```python
@pytest.mark.parametrize("model", models)
@pytest.mark.parametrize(
    "size_factors",
    [
        # Single-scale
        [1.0],
        # Single-scale, batched
        [1.0, 1.0, 1.0],
        # Multi-scale
        [0.25, 0.5, 1.0],
    ],
)
@pytest.mark.parametrize("dtype", [target_dtype])
@pytest.mark.parametrize("max_model_len", [12800])
@pytest.mark.parametrize("max_tokens", [128])
@pytest.mark.parametrize("num_logprobs", [10])
def test_models(
    hf_runner,
# ... 24 lines omitted for brevity ...
        model,
        dtype=dtype,
        max_model_len=max_model_len,
        max_tokens=max_tokens,
        num_logprobs=num_logprobs,
        mm_limit=1,
        tensor_parallel_size=1,
    )
```
**EN:** This test validates `test_models`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `model`, `size_factors`, `dtype`.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`model`、`size_factors`、`dtype`。

### Test / 测试: test_multi_images_models (L221-L275)
```python
@large_gpu_test(min_gb=48)
@pytest.mark.parametrize("model", models)
@pytest.mark.parametrize(
    "size_factors",
    [
        # No image
        # [],
        # Single-scale
        [1.0],
        # Single-scale, batched
        [1.0, 1.0, 1.0],
        # Multi-scale
        [0.25, 0.5, 1.0],
    ],
)
@pytest.mark.parametrize("dtype", [target_dtype])
@pytest.mark.parametrize("max_model_len", [25600])
@pytest.mark.parametrize("max_tokens", [128])
# ... 29 lines omitted for brevity ...
        model,
        dtype=dtype,
        max_model_len=max_model_len,
        max_tokens=max_tokens,
        num_logprobs=num_logprobs,
        mm_limit=2,
        tensor_parallel_size=1,
    )
```
**EN:** This test validates `test_multi_images_models`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `model`, `size_factors`, `dtype`.
**CN:** 这个测试验证 `test_multi_images_models`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`model`、`size_factors`、`dtype`。

### Test / 测试: test_vision_speech_models (L278-L315)
```python
@pytest.mark.parametrize("model", models)
@pytest.mark.parametrize("dtype", [target_dtype])
@pytest.mark.parametrize("max_model_len", [12800])
@pytest.mark.parametrize("max_tokens", [128])
@pytest.mark.parametrize("num_logprobs", [10])
def test_vision_speech_models(
    hf_runner,
    vllm_runner,
    model,
    dtype: str,
    max_model_len: int,
    max_tokens: int,
    num_logprobs: int,
) -> None:
    # use the example speech question so that the model outputs are reasonable
    audio = load_audio(speech_question, sr=None)
    image = convert_image_mode(ImageAsset("cherry_blossom").pil_image, "RGB")

# ... 12 lines omitted for brevity ...
        model,
        dtype=dtype,
        max_model_len=max_model_len,
        max_tokens=max_tokens,
        num_logprobs=num_logprobs,
        mm_limit=1,
        tensor_parallel_size=1,
    )
```
**EN:** This test validates `test_vision_speech_models`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `model`, `dtype`, `max_model_len`, `max_tokens`.
**CN:** 这个测试验证 `test_vision_speech_models`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`model`、`dtype`、`max_model_len`、`max_tokens`。

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
- **Stdlib / 标准库**: `collections.abc.Sequence`, `os`
- **Third-party / 第三方**: `huggingface_hub.snapshot_download`, `pytest`, `regex`, `transformers.AutoTokenizer`
- **Project / 项目内**: `vllm.assets.image.ImageAsset`, `vllm.logprobs.SampleLogprobs`, `vllm.lora.request.LoRARequest`, `vllm.multimodal.image.convert_image_mode`, `vllm.multimodal.image.rescale_image_size`, `vllm.multimodal.media.audio.load_audio`
- **Local relative imports / 本地相对导入**: `....conftest.HfRunner`, `....conftest.IMAGE_ASSETS`, `....conftest.PromptAudioInput`, `....conftest.PromptImageInput`, `....conftest.VllmRunner`, `....utils.large_gpu_test`, `...utils.check_logprobs_close`
