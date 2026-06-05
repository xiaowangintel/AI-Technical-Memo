# test_siglip.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/pooling/test_siglip.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and multimodal processing. The file defines 3 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与多模态处理。它定义了 3 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L11)
```python
from typing import Any

import pytest
import torch
from transformers import SiglipModel

from ....conftest import IMAGE_ASSETS, HfRunner, PromptImageInput, VllmRunner
from ...utils import check_embeddings_close
```
**EN:** Imports standard-library modules such as `typing.Any`, third-party packages like `pytest`, `torch`, `transformers.SiglipModel`, project helpers such as `....conftest.HfRunner`, `....conftest.IMAGE_ASSETS`, `....conftest.PromptImageInput`.
**CN:** 导入标准库模块（如 `typing.Any`）、第三方包（如 `pytest`、`torch`、`transformers.SiglipModel`）、项目内辅助模块（如 `....conftest.HfRunner`、`....conftest.IMAGE_ASSETS`、`....conftest.PromptImageInput`）。

### Module setup / 模块级配置: HF_TEXT_PROMPTS, HF_IMAGE_PROMPTS, MODELS (L13-L30)
```python
HF_TEXT_PROMPTS = [
    "a photo of a stop sign",
    "a photo of a cherry blossom",
]

HF_IMAGE_PROMPTS = IMAGE_ASSETS.prompts(
    {
        "stop_sign": "",
        "cherry_blossom": "",
    }
)

MODELS = [
    "google/siglip-base-patch16-224",
    "google/siglip2-base-patch16-224",
    # Different image embedding dim than text_config.hidden_size
    "google/siglip2-giant-opt-patch16-384",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `HF_TEXT_PROMPTS`, `HF_IMAGE_PROMPTS`, `MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `HF_TEXT_PROMPTS`、`HF_IMAGE_PROMPTS`、`MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _run_test (L33-L90)
```python
def _run_test(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    input_texts: list[str],
    input_images: PromptImageInput,
    model: str,
    *,
    dtype: str,
    tokenization_kwargs: dict[str, Any] | None = None,
    attention_config: dict[str, Any] | None = None,
) -> None:
    if tokenization_kwargs is None:
        tokenization_kwargs = {}

    with vllm_runner(
        model,
        runner="pooling",
        dtype=dtype,
# ... 32 lines omitted for brevity ...
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

### Test / 测试: test_models_text (L93-L119)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["float"])
def test_models_text(
    hf_runner,
    vllm_runner,
    image_assets,
    siglip_attention_config,
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
        tokenization_kwargs={
            "padding": "max_length",
            "max_length": 64,
        },  # siglip2 was trained with this padding setting.
        attention_config=siglip_attention_config,
    )
```
**EN:** This test validates `test_models_text`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `siglip_attention_config`, `model`, `dtype`.
**CN:** 这个测试验证 `test_models_text`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`siglip_attention_config`、`model`、`dtype`。

### Test / 测试: test_models_image (L122-L146)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["float"])
def test_models_image(
    hf_runner,
    vllm_runner,
    image_assets,
    siglip_attention_config,
    model: str,
    dtype: str,
) -> None:
    input_texts_images = [
        (text, asset.pil_image) for text, asset in zip(HF_IMAGE_PROMPTS, image_assets)
    ]
    input_texts = [text for text, _ in input_texts_images]
    input_images = [image for _, image in input_texts_images]

    _run_test(
        hf_runner,
        vllm_runner,
        input_texts,
        input_images,
        model,
        dtype=dtype,
        attention_config=siglip_attention_config,
    )
```
**EN:** This test validates `test_models_image`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `siglip_attention_config`, `model`, `dtype`.
**CN:** 这个测试验证 `test_models_image`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`siglip_attention_config`、`model`、`dtype`。

### Test / 测试: test_models_text_image_no_crash (L149-L174)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["float"])
def test_models_text_image_no_crash(
    vllm_runner,
    image_assets,
    siglip_attention_config,
    model: str,
    dtype: str,
) -> None:
    texts = [HF_TEXT_PROMPTS[0]]
    images = [image_assets[0].pil_image]

    with vllm_runner(
        model,
        runner="pooling",
        dtype=dtype,
        enforce_eager=True,
        max_model_len=64,
        gpu_memory_utilization=0.7,
        attention_config=siglip_attention_config,
    ) as vllm_model:
        with pytest.raises(ValueError, match="not both"):
            vllm_model.embed(texts, images=images)

        vllm_model.embed(texts)
        vllm_model.embed([""], images=images)
```
**EN:** This test validates `test_models_text_image_no_crash`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `image_assets`, `siglip_attention_config`, `model`, `dtype`. It checks an expected failure path with `pytest.raises`.
**CN:** 这个测试验证 `test_models_text_image_no_crash`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`image_assets`、`siglip_attention_config`、`model`、`dtype`。 它使用 `pytest.raises` 检查预期失败路径。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing.Any`
- **Third-party / 第三方**: `pytest`, `torch`, `transformers.SiglipModel`
- **Local relative imports / 本地相对导入**: `....conftest.HfRunner`, `....conftest.IMAGE_ASSETS`, `....conftest.PromptImageInput`, `....conftest.VllmRunner`, `...utils.check_embeddings_close`
