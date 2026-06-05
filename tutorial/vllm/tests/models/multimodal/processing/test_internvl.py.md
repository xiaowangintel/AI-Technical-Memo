# test_internvl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/processing/test_internvl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 1 test(s), 0 fixture(s), and 2 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 1 个测试、0 个 fixture，以及 2 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L16)
```python
from collections.abc import Mapping

import pytest
from PIL import Image
from transformers import PretrainedConfig

from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.image import rescale_image_size
from vllm.multimodal.processing import BaseMultiModalProcessor

from ....conftest import ImageTestAssets
from ...utils import build_model_context
```
**EN:** Imports standard-library modules such as `collections.abc.Mapping`, third-party packages like `PIL.Image`, `pytest`, `transformers.PretrainedConfig`, project helpers such as `vllm.multimodal.MULTIMODAL_REGISTRY`, `vllm.multimodal.image.rescale_image_size`, `vllm.multimodal.processing.BaseMultiModalProcessor`.
**CN:** 导入标准库模块（如 `collections.abc.Mapping`）、第三方包（如 `PIL.Image`、`pytest`、`transformers.PretrainedConfig`）、项目内辅助模块（如 `vllm.multimodal.MULTIMODAL_REGISTRY`、`vllm.multimodal.image.rescale_image_size`、`vllm.multimodal.processing.BaseMultiModalProcessor`）。

### Helper / 辅助函数: _get_expected_num_patches (L19-L48)
```python
def _get_expected_num_patches(
    config: PretrainedConfig,
    image: Image.Image,
    num_imgs: int,
    min_num: int,
    max_num: int,
):
    from vllm.transformers_utils.processors.internvl import (
        calculate_internvl_targets,
        get_internvl_target_ratios,
    )

    width, height = image.size

    blocks, _, _ = calculate_internvl_targets(
        orig_width=width,
        orig_height=height,
        target_ratios=get_internvl_target_ratios(
            min_num,
            max_num,
        ),
        image_size=config.vision_config.image_size,
        use_thumbnail=False,
    )
    expected_num_patches = blocks

    if config.use_thumbnail and expected_num_patches > 1:
        expected_num_patches += 1

    return expected_num_patches
```
**EN:** This helper encapsulates reusable logic in `_get_expected_num_patches`. Key inputs are `config`, `image`, `num_imgs`, `min_num`, `max_num`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_expected_num_patches` 中。 关键输入包括 `config`、`image`、`num_imgs`、`min_num`、`max_num`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _run_check (L51-L81)
```python
def _run_check(
    processor: BaseMultiModalProcessor,
    images: list[Image.Image],
    min_num: int,
    max_num: int,
    mm_processor_kwargs: Mapping[str, object],
):
    tokenizer = processor.info.get_tokenizer()
    config = processor.info.get_hf_config()

    prompt = "<image>" * len(images)
    mm_data = {"image": images}

    total_expected_num_patches = sum(
        _get_expected_num_patches(config, image, len(images), min_num, max_num)
        for image in images
    )

# ... 5 lines omitted for brevity ...

    # Ensure we have the right number of placeholders per num_crops size
    image_token_id = tokenizer.convert_tokens_to_ids("<IMG_CONTEXT>")
    img_tok_count = processed_inputs["prompt_token_ids"].count(image_token_id)
    pixel_shape = processed_inputs["mm_kwargs"].get_data()["pixel_values_flat"].shape

    assert img_tok_count == 256 * total_expected_num_patches
    assert pixel_shape[0] == total_expected_num_patches
```
**EN:** This helper encapsulates reusable logic in `_run_check`. Key inputs are `processor`, `images`, `min_num`, `max_num`, `mm_processor_kwargs`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The main assertion is `img_tok_count == 256 * total_expected_num_patches` and `pixel_shape[0] == total_expected_num_patches`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_check` 中。 关键输入包括 `processor`、`images`、`min_num`、`max_num`、`mm_processor_kwargs`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 核心断言是 `img_tok_count == 256 * total_expected_num_patches` and `pixel_shape[0] == total_expected_num_patches`。

### Test / 测试: test_processor_override (L84-L135)
```python
@pytest.mark.parametrize("model_id", ["OpenGVLab/InternVL2-2B"])
@pytest.mark.parametrize(
    "size_factors",
    [
        # Single-scale
        [1.0],
        # Single-scale, batched
        [1.0, 1.0, 1.0],
        # Multi-scale
        [0.25, 0.5, 1.0],
        [4.0, 2.0, 1.0],
    ],
)
@pytest.mark.parametrize(
    ("min_dynamic_patch", "max_dynamic_patch"),
    [(1, 1), (1, 2), (1, 4), (1, 8), (2, 4), (4, 8)],
)
@pytest.mark.parametrize("dynamic_image_size", [True, False])
# ... 26 lines omitted for brevity ...

    _run_check(
        processor,
        [rescale_image_size(image_assets[0].pil_image, f) for f in size_factors],
        min_num,
        max_num,
        hf_processor_mm_kwargs,
    )
```
**EN:** This test validates `test_processor_override`. It uses parameterization over `model_id`. Key inputs are `model_id`, `image_assets`, `size_factors`, `min_dynamic_patch`, `max_dynamic_patch`, `dynamic_image_size`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It drives client-facing request creation through the API surface under test.
**CN:** 这个测试验证 `test_processor_override`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`、`image_assets`、`size_factors`、`min_dynamic_patch`、`max_dynamic_patch`、`dynamic_image_size`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它通过被测 API 表面触发面向客户端的请求创建流程。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Mapping`
- **Third-party / 第三方**: `PIL.Image`, `pytest`, `transformers.PretrainedConfig`
- **Project / 项目内**: `vllm.multimodal.MULTIMODAL_REGISTRY`, `vllm.multimodal.image.rescale_image_size`, `vllm.multimodal.processing.BaseMultiModalProcessor`, `vllm.transformers_utils.processors.internvl.calculate_internvl_targets`, `vllm.transformers_utils.processors.internvl.get_internvl_target_ratios`
- **Local relative imports / 本地相对导入**: `....conftest.ImageTestAssets`, `...utils.build_model_context`
