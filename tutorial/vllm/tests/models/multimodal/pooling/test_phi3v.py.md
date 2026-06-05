# test_phi3v.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/pooling/test_phi3v.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and multimodal processing. The file defines 2 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与多模态处理。它定义了 2 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L14)
```python
import pytest
import torch.nn.functional as F
import transformers.utils
from PIL import Image

from vllm.assets.base import get_vllm_public_assets
from vllm.assets.image import VLM_IMAGES_DIR

from ....conftest import IMAGE_ASSETS, HfRunner, PromptImageInput, VllmRunner
from ....utils import large_gpu_test
from ...utils import check_embeddings_close
```
**EN:** Imports third-party packages like `PIL.Image`, `pytest`, `torch.nn.functional`, project helpers such as `vllm.assets.base.get_vllm_public_assets`, `vllm.assets.image.VLM_IMAGES_DIR`, `....conftest.HfRunner`.
**CN:** 导入第三方包（如 `PIL.Image`、`pytest`、`torch.nn.functional`）、项目内辅助模块（如 `vllm.assets.base.get_vllm_public_assets`、`vllm.assets.image.VLM_IMAGES_DIR`、`....conftest.HfRunner`）。

### Module setup / 模块级配置: HF_TEXT_PROMPTS, HF_IMAGE_PROMPTS, MODELS (L18-L38)
```python
transformers.utils.is_flash_attn_greater_or_equal_2_10 = (
    lambda: transformers.utils.is_flash_attn_greater_or_equal("2.1.0")
)

HF_TEXT_PROMPTS = [
    # T -> X
    "Find me an everyday image that matches the given caption: The label of the object is stop sign",  # noqa: E501
    # T -> X
    "Retrieve an image of this caption: cherry blossom",
]

HF_IMAGE_PROMPTS = IMAGE_ASSETS.prompts(
    {
        # T + I -> X
        "stop_sign": "<|image_1|> Select the portion of the image that isolates the object of the given label: The label of the object is stop sign",  # noqa: E501
        # I -> X
        "cherry_blossom": "<|image_1|> Represent the given image for classification",  # noqa: E501
    }
)

MODELS = ["TIGER-Lab/VLM2Vec-Full"]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `HF_TEXT_PROMPTS`, `HF_IMAGE_PROMPTS`, `MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `HF_TEXT_PROMPTS`、`HF_IMAGE_PROMPTS`、`MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _run_test (L41-L85)
```python
def _run_test(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    input_texts: list[str],
    input_images: PromptImageInput,
    model: str,
    *,
    dtype: str,
) -> None:
    # NOTE: take care of the order. run vLLM first, and then run HF.
    # vLLM needs a fresh new process without cuda initialization.
    # if we run HF first, the cuda initialization will be done and it
    # will hurt multiprocessing backend with fork method (the default method).
    with vllm_runner(
        model, runner="pooling", dtype=dtype, enforce_eager=True
    ) as vllm_model:
        vllm_outputs = vllm_model.embed(input_texts, images=input_images)

# ... 19 lines omitted for brevity ...
        hf_outputs = all_outputs

    check_embeddings_close(
        embeddings_0_lst=hf_outputs,
        embeddings_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** This helper encapsulates reusable logic in `_run_test`. Key inputs are `hf_runner`, `vllm_runner`, `input_texts`, `input_images`, `model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_test` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`input_texts`、`input_images`、`model`。

### Test / 测试: test_models_text (L88-L109)
```python
@pytest.mark.core_model
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half"])
def test_models_text(
    hf_runner,
    vllm_runner,
    image_assets,
    model: str,
    dtype: str,
) -> None:
    input_texts_images = [(text, None) for text in HF_TEXT_PROMPTS]
    input_texts = [text for text, _ in input_texts_images]
    input_images = [image for _, image in input_texts_images]

    _run_test(
        hf_runner,
        vllm_runner,
        input_texts,
        input_images,  # type: ignore
        model,
        dtype=dtype,
    )
```
**EN:** This test validates `test_models_text`. It uses parameterization over `model`. Relevant pytest markers include `core_model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `model`, `dtype`.
**CN:** 这个测试验证 `test_models_text`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `core_model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`model`、`dtype`。

### Test / 测试: test_models_image (L112-L149)
```python
@large_gpu_test(min_gb=48)
@pytest.mark.core_model
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["half"])
def test_models_image(
    hf_runner,
    vllm_runner,
    image_assets,
    model: str,
    dtype: str,
) -> None:
    input_texts_images = [
        (text, asset.pil_image) for text, asset in zip(HF_IMAGE_PROMPTS, image_assets)
    ]
    # add cases for special_tokens
    input_texts_images.append(
        (
            "\n<s><|user|>\n <|image_1|>\n\t <s>"
# ... 12 lines omitted for brevity ...
    _run_test(
        hf_runner,
        vllm_runner,
        input_texts,
        input_images,
        model,
        dtype=dtype,
    )
```
**EN:** This test validates `test_models_image`. It uses parameterization over `model`. Relevant pytest markers include `core_model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `model`, `dtype`.
**CN:** 这个测试验证 `test_models_image`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `core_model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`model`、`dtype`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `PIL.Image`, `pytest`, `torch.nn.functional`, `transformers.utils`
- **Project / 项目内**: `vllm.assets.base.get_vllm_public_assets`, `vllm.assets.image.VLM_IMAGES_DIR`
- **Local relative imports / 本地相对导入**: `....conftest.HfRunner`, `....conftest.IMAGE_ASSETS`, `....conftest.PromptImageInput`, `....conftest.VllmRunner`, `....utils.large_gpu_test`, `...utils.check_embeddings_close`
