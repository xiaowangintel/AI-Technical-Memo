# test_phi4siglip.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_phi4siglip.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 2 test(s), 0 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 2 个测试、0 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L22)
```python
from collections.abc import Sequence
from importlib.metadata import version

import pytest
import regex as re
from packaging.version import Version
from transformers import AutoModelForCausalLM, AutoTokenizer

from vllm.logprobs import SampleLogprobs
from vllm.multimodal.image import rescale_image_size

from ....conftest import (
    IMAGE_ASSETS,
    HfRunner,
    PromptImageInput,
    VllmRunner,
)
from ....utils import multi_gpu_test
from ...utils import check_logprobs_close
```
**EN:** Imports standard-library modules such as `collections.abc.Sequence`, `importlib.metadata.version`, third-party packages like `packaging.version.Version`, `pytest`, `regex`, project helpers such as `vllm.logprobs.SampleLogprobs`, `vllm.multimodal.image.rescale_image_size`, `....conftest.HfRunner`.
**CN:** 导入标准库模块（如 `collections.abc.Sequence`、`importlib.metadata.version`）、第三方包（如 `packaging.version.Version`、`pytest`、`regex`）、项目内辅助模块（如 `vllm.logprobs.SampleLogprobs`、`vllm.multimodal.image.rescale_image_size`、`....conftest.HfRunner`）。

### Module setup / 模块级配置: pytestmark, MODEL_ID, HF_IMAGE_PROMPTS (L24-L47)
```python
pytestmark = pytest.mark.skipif(
    Version("5.0") <= Version(version("transformers")),
    reason=(
        "vllm upgraded transformers above v5.4 where HF model custom code uses siglip2 "
        "internals (filter_out_non_signature_kwargs) removed by "
        "huggingface/transformers#43514"
    ),
)

MODEL_ID = "microsoft/Phi-4-reasoning-vision-15B"

HF_IMAGE_PROMPTS = IMAGE_ASSETS.prompts(
    {
        "stop_sign": "<|user|>\n<image>\nWhat's the content of the image?<|end|>\n<|assistant|>\n",  # noqa: E501
        "cherry_blossom": "<|user|>\n<image>\nPlease infer the season with reason in details.<|end|>\n<|assistant|>\n",  # noqa: E501
    }
)
HF_MULTIIMAGE_IMAGE_PROMPT = (
    "<|user|>\n<image>\n<image>\nDescribe these images.<|end|>\n<|assistant|>\n"  # noqa: E501
)

DTYPE = "half"
MAX_TOKENS = 128
NUM_LOGPROBS = 10
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `pytestmark`, `MODEL_ID`, `HF_IMAGE_PROMPTS`, `HF_MULTIIMAGE_IMAGE_PROMPT`, `DTYPE`, `MAX_TOKENS`, `NUM_LOGPROBS`. Later helpers and tests reuse these values to keep scenarios concise. It also applies module-wide pytest markers.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `pytestmark`、`MODEL_ID`、`HF_IMAGE_PROMPTS`、`HF_MULTIIMAGE_IMAGE_PROMPT`、`DTYPE`、`MAX_TOKENS`、`NUM_LOGPROBS`。后续辅助函数和测试会复用这些值，以减少重复。 它还会应用模块级的 pytest 标记。

### Helper / 辅助函数: vllm_to_hf_output (L50-L67)
```python
def vllm_to_hf_output(
    vllm_output: tuple[list[int], str, SampleLogprobs | None], model: str
):
    """Sanitize vllm output to be comparable with hf output."""
    _, output_str, out_logprobs = vllm_output

    output_str_without_image = re.sub(r"(<image>)+", "", output_str)
    if output_str_without_image and output_str_without_image[0] == " ":
        output_str_without_image = output_str_without_image[1:]

    hf_output_str = output_str_without_image + "<|end|><|endoftext|>"

    tokenizer = AutoTokenizer.from_pretrained(model, trust_remote_code=True)
    hf_output_ids = tokenizer.encode(output_str_without_image)
    if hf_output_ids and hf_output_ids[0] == tokenizer.bos_token_id:
        hf_output_ids = hf_output_ids[1:]

    return hf_output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _build_single_image_inputs (L70-L84)
```python
def _build_single_image_inputs(
    image_assets,
) -> list[tuple[list[str], PromptImageInput]]:
    """Build single-image inputs for all size_factors at once."""
    images = [asset.pil_image for asset in image_assets]
    all_inputs: list[tuple[list[str], PromptImageInput]] = []
    for size_factors in [[1.0], [0.25, 0.5, 1.0]]:
        for image, prompt in zip(images, HF_IMAGE_PROMPTS):
            all_inputs.append(
                (
                    [prompt for _ in size_factors],
                    [rescale_image_size(image, f) for f in size_factors],
                )
            )
    return all_inputs
```
**EN:** This helper encapsulates reusable logic in `_build_single_image_inputs`. Key inputs are `image_assets`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_single_image_inputs` 中。 关键输入包括 `image_assets`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _build_multi_image_inputs (L87-L103)
```python
def _build_multi_image_inputs(
    image_assets,
) -> list[tuple[list[str], PromptImageInput]]:
    """Build multi-image inputs for all size_factors at once."""
    images = [asset.pil_image for asset in image_assets]
    all_inputs: list[tuple[list[str], PromptImageInput]] = []
    for size_factors in [[0.5], [0.15, 0.30]]:
        all_inputs.append(
            (
                [HF_MULTIIMAGE_IMAGE_PROMPT for _ in size_factors],
                [
                    [rescale_image_size(image, factor) for image in images]
                    for factor in size_factors
                ],
            )
        )
    return all_inputs
```
**EN:** This helper encapsulates reusable logic in `_build_multi_image_inputs`. Key inputs are `image_assets`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_build_multi_image_inputs` 中。 关键输入包括 `image_assets`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _run_and_compare (L106-L166)
```python
def _run_and_compare(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    all_inputs: Sequence[tuple[list[str], PromptImageInput]],
    model: str,
    max_model_len: int,
    max_num_seqs: int,
    mm_limit: int,
    gpu_memory_utilization: float,
):
    """Load each runner once, run all inputs, then compare."""
    # NOTE: run vLLM first, then HF.  vLLM needs a fresh process without
    # cuda initialization; running HF first would break the multiprocessing
    # backend with fork method.
    with vllm_runner(
        model,
        runner="generate",
        max_model_len=max_model_len,
# ... 35 lines omitted for brevity ...

    for hf_outputs, vllm_outputs in zip(hf_outputs_per_case, vllm_outputs_per_case):
        check_logprobs_close(
            outputs_0_lst=hf_outputs,
            outputs_1_lst=vllm_outputs,
            name_0="hf",
            name_1="vllm",
        )
```
**EN:** This helper encapsulates reusable logic in `_run_and_compare`. Key inputs are `hf_runner`, `vllm_runner`, `all_inputs`, `model`, `max_model_len`, `max_num_seqs`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_and_compare` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`all_inputs`、`model`、`max_model_len`、`max_num_seqs`。

### Test / 测试: test_models (L169-L182)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize("model", [MODEL_ID])
def test_models(hf_runner, vllm_runner, image_assets, model) -> None:
    all_inputs = _build_single_image_inputs(image_assets)
    _run_and_compare(
        hf_runner,
        vllm_runner,
        all_inputs,
        model,
        max_model_len=8192,
        max_num_seqs=2,
        mm_limit=1,
        gpu_memory_utilization=0.80,
    )
```
**EN:** This test validates `test_models`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `model`.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`model`。

### Test / 测试: test_multi_images_models (L185-L198)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize("model", [MODEL_ID])
def test_multi_images_models(hf_runner, vllm_runner, image_assets, model) -> None:
    all_inputs = _build_multi_image_inputs(image_assets)
    _run_and_compare(
        hf_runner,
        vllm_runner,
        all_inputs,
        model,
        max_model_len=8192,
        max_num_seqs=2,
        mm_limit=2,
        gpu_memory_utilization=0.80,
    )
```
**EN:** This test validates `test_multi_images_models`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `model`.
**CN:** 这个测试验证 `test_multi_images_models`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`model`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Sequence`, `importlib.metadata.version`
- **Third-party / 第三方**: `packaging.version.Version`, `pytest`, `regex`, `transformers.AutoModelForCausalLM`, `transformers.AutoTokenizer`
- **Project / 项目内**: `vllm.logprobs.SampleLogprobs`, `vllm.multimodal.image.rescale_image_size`
- **Local relative imports / 本地相对导入**: `....conftest.HfRunner`, `....conftest.IMAGE_ASSETS`, `....conftest.PromptImageInput`, `....conftest.VllmRunner`, `....utils.multi_gpu_test`, `...utils.check_logprobs_close`
