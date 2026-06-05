# test_gemma4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/processing/test_gemma4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 7 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 7 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L15)
```python
from collections.abc import Mapping

import pytest
import torch
from PIL import Image as PILImage

from vllm.model_executor.models.gemma4_mm import Gemma4ImagePixelInputs
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import MultiModalFieldConfig

from ....conftest import ImageTestAssets
from ...utils import build_model_context
```
**EN:** Imports standard-library modules such as `collections.abc.Mapping`, third-party packages like `PIL.Image`, `pytest`, `torch`, project helpers such as `vllm.model_executor.models.gemma4_mm.Gemma4ImagePixelInputs`, `vllm.multimodal.MULTIMODAL_REGISTRY`, `vllm.multimodal.inputs.MultiModalFieldConfig`.
**CN:** 导入标准库模块（如 `collections.abc.Mapping`）、第三方包（如 `PIL.Image`、`pytest`、`torch`）、项目内辅助模块（如 `vllm.model_executor.models.gemma4_mm.Gemma4ImagePixelInputs`、`vllm.multimodal.MULTIMODAL_REGISTRY`、`vllm.multimodal.inputs.MultiModalFieldConfig`）。

### Module setup / 模块级配置: GEMMA4_MODEL_ID (L18-L18)
```python
GEMMA4_MODEL_ID = "google/gemma-4-E2B-it"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `GEMMA4_MODEL_ID`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `GEMMA4_MODEL_ID`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_gemma4_image_schema_accepts_variable_patch_counts (L21-L31)
```python
def test_gemma4_image_schema_accepts_variable_patch_counts():
    Gemma4ImagePixelInputs(
        pixel_values=[
            torch.randn(10080, 768),
            torch.randn(2520, 768),
        ],
        pixel_position_ids=[
            torch.zeros(10080, 2, dtype=torch.long),
            torch.zeros(2520, 2, dtype=torch.long),
        ],
    )
```
**EN:** This test validates `test_gemma4_image_schema_accepts_variable_patch_counts`.
**CN:** 这个测试验证 `test_gemma4_image_schema_accepts_variable_patch_counts`。

### Test / 测试: test_gemma4_image_batching_keeps_variable_patch_counts_unstacked (L34-L48)
```python
def test_gemma4_image_batching_keeps_variable_patch_counts_unstacked():
    field = MultiModalFieldConfig.batched("image").field
    elems = field.build_elems(
        "image",
        "pixel_values",
        [torch.randn(10080, 768), torch.randn(2520, 768)],
    )

    reduced = field.reduce_data(list(elems))

    assert isinstance(reduced, list)
    assert [tensor.shape for tensor in reduced] == [
        torch.Size([10080, 768]),
        torch.Size([2520, 768]),
    ]
```
**EN:** This test validates `test_gemma4_image_batching_keeps_variable_patch_counts_unstacked`. The main assertion is `isinstance(reduced, list)` and `[tensor.shape for tensor in reduced] == [torch.Size([10080, 768]), torch.Size([2520, 768])]`.
**CN:** 这个测试验证 `test_gemma4_image_batching_keeps_variable_patch_counts_unstacked`。 核心断言是 `isinstance(reduced, list)` and `[tensor.shape for tensor in reduced] == [torch.Size([10080, 768]), torch.Size([2520, 768])]`。

### Test / 测试: test_compute_num_soft_tokens_does_not_exceed_max_soft_tokens (L51-L102)
```python
@pytest.mark.parametrize(
    "image_width,image_height,max_soft_tokens",
    [
        # Production repro: a 3x900 image (extreme aspect ratio) made the
        # prompt-side estimator return 289 while the HF Gemma 4 image
        # processor's vision tower output capped at 280, producing the
        # "Attempted to assign 280 multimodal tokens to 289 placeholders"
        # mismatch that crashed EngineCore.
        (900, 3, 280),
        (3, 900, 280),
        # Same pathology should hold for the video-frame budget (70 tokens).
        (900, 3, 70),
        # And for any other supported budget.
        (4000, 2, 1120),
    ],
)
@pytest.mark.parametrize("model_id", [GEMMA4_MODEL_ID])
def test_compute_num_soft_tokens_does_not_exceed_max_soft_tokens(
# ... 26 lines omitted for brevity ...

    assert num_soft_tokens <= max_soft_tokens, (
        f"_compute_num_soft_tokens returned {num_soft_tokens} for "
        f"image_width={image_width}, image_height={image_height}, "
        f"max_soft_tokens={max_soft_tokens} — exceeds the cap that the HF "
        f"image processor enforces on its vision tower output. This is "
        f"the placeholder/encoder count mismatch that crashes EngineCore."
    )
```
**EN:** This test validates `test_compute_num_soft_tokens_does_not_exceed_max_soft_tokens`. It uses parameterization over `image_width`, `image_height`, `max_soft_tokens`. Key inputs are `model_id`, `image_width`, `image_height`, `max_soft_tokens`. It drives client-facing request creation through the API surface under test. The main assertion is `num_soft_tokens <= max_soft_tokens`.
**CN:** 这个测试验证 `test_compute_num_soft_tokens_does_not_exceed_max_soft_tokens`。 它通过参数化组合 `image_width`、`image_height`、`max_soft_tokens`。 关键输入包括 `model_id`、`image_width`、`image_height`、`max_soft_tokens`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `num_soft_tokens <= max_soft_tokens`。

### Test / 测试: test_get_mm_max_tokens_per_item_respects_configured_max_soft_tokens (L105-L137)
```python
@pytest.mark.parametrize(
    ("mm_processor_kwargs", "expected_image_tokens"),
    [
        ({}, 280),
        ({"max_soft_tokens": 70}, 70),
        ({"max_soft_tokens": 280}, 280),
        ({"max_soft_tokens": 1120}, 1120),
        ({"images_kwargs": {"max_soft_tokens": 560}}, 560),
        ({"images_kwargs": None}, 280),
        ({"images_kwargs": "not-a-dict"}, 280),
    ],
)
@pytest.mark.parametrize("model_id", [GEMMA4_MODEL_ID])
def test_get_mm_max_tokens_per_item_respects_configured_max_soft_tokens(
    model_id: str,
    mm_processor_kwargs: dict[str, object],
    expected_image_tokens: int,
):
# ... 7 lines omitted for brevity ...
    tokens = processor.info.get_mm_max_tokens_per_item(
        seq_len=ctx.model_config.max_model_len,
        mm_counts={"image": 1, "video": 1},
    )

    assert tokens is not None
    assert tokens["image"] == expected_image_tokens
    assert tokens["video"] == 32 * (70 + 2 + 6)
```
**EN:** This test validates `test_get_mm_max_tokens_per_item_respects_configured_max_soft_tokens`. It uses parameterization over `mm_processor_kwargs`, `expected_image_tokens`. Key inputs are `model_id`, `mm_processor_kwargs`, `expected_image_tokens`. It drives client-facing request creation through the API surface under test. The main assertion is `tokens is not None` and `tokens['image'] == expected_image_tokens`.
**CN:** 这个测试验证 `test_get_mm_max_tokens_per_item_respects_configured_max_soft_tokens`。 它通过参数化组合 `mm_processor_kwargs`、`expected_image_tokens`。 关键输入包括 `model_id`、`mm_processor_kwargs`、`expected_image_tokens`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `tokens is not None` and `tokens['image'] == expected_image_tokens`。

### Test / 测试: test_get_mm_max_tokens_per_item_respects_configured_video_num_frames (L140-L170)
```python
@pytest.mark.parametrize(
    ("limit_mm_per_prompt", "expected_video_tokens"),
    [
        ({"video": 1}, 32 * (70 + 2 + 6)),
        ({"video": {"count": 1}}, 32 * (70 + 2 + 6)),
        ({"video": {"count": 1, "num_frames": 1}}, 1 * (70 + 2 + 6)),
        ({"video": {"count": 1, "num_frames": 8}}, 8 * (70 + 2 + 6)),
        ({"video": {"count": 1, "num_frames": 32}}, 32 * (70 + 2 + 6)),
        ({"video": {"count": 1, "num_frames": 40}}, 32 * (70 + 2 + 6)),
    ],
)
@pytest.mark.parametrize("model_id", [GEMMA4_MODEL_ID])
def test_get_mm_max_tokens_per_item_respects_configured_video_num_frames(
    model_id: str,
    limit_mm_per_prompt: Mapping[str, int | Mapping[str, int]],
    expected_video_tokens: int,
):
    ctx = build_model_context(
# ... 5 lines omitted for brevity ...
    tokens = processor.info.get_mm_max_tokens_per_item(
        seq_len=ctx.model_config.max_model_len,
        mm_counts={"video": 1},
    )

    assert tokens is not None
    assert tokens["image"] == 280
    assert tokens["video"] == expected_video_tokens
```
**EN:** This test validates `test_get_mm_max_tokens_per_item_respects_configured_video_num_frames`. It uses parameterization over `limit_mm_per_prompt`, `expected_video_tokens`. Key inputs are `model_id`, `limit_mm_per_prompt`, `expected_video_tokens`. It drives client-facing request creation through the API surface under test. The main assertion is `tokens is not None` and `tokens['image'] == 280`.
**CN:** 这个测试验证 `test_get_mm_max_tokens_per_item_respects_configured_video_num_frames`。 它通过参数化组合 `limit_mm_per_prompt`、`expected_video_tokens`。 关键输入包括 `model_id`、`limit_mm_per_prompt`、`expected_video_tokens`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `tokens is not None` and `tokens['image'] == 280`。

### Test / 测试: test_get_prompt_updates_respects_nested_max_soft_tokens (L173-L194)
```python
@pytest.mark.parametrize("model_id", [GEMMA4_MODEL_ID])
def test_get_prompt_updates_respects_nested_max_soft_tokens(model_id: str):
    ctx = build_model_context(
        model_id,
        mm_processor_kwargs={"images_kwargs": {"max_soft_tokens": 560}},
        limit_mm_per_prompt={"image": 1},
    )
    processor = MULTIMODAL_REGISTRY.create_processor(ctx.model_config)
    image = PILImage.new("RGB", (1000, 1000), color="white")
    image_size = image.size
    mm_items = processor.info.parse_mm_data({"image": image})

    prompt_update = processor._get_prompt_updates(mm_items, {}, {})[0]
    replacement = prompt_update.resolve(0).content.full
    expected = processor.info.get_image_repl(
        image_width=image_size[0],
        image_height=image_size[1],
        processor=processor.info.get_hf_processor(),
        max_soft_tokens=560,
    ).full

    assert replacement == expected
```
**EN:** This test validates `test_get_prompt_updates_respects_nested_max_soft_tokens`. It uses parameterization over `model_id`. Key inputs are `model_id`. It drives client-facing request creation through the API surface under test. The main assertion is `replacement == expected`.
**CN:** 这个测试验证 `test_get_prompt_updates_respects_nested_max_soft_tokens`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `replacement == expected`。

### Test / 测试: test_limit_mm_per_prompt (L197-L226)
```python
@pytest.mark.parametrize("model_id", [GEMMA4_MODEL_ID])
def test_limit_mm_per_prompt(
    image_assets: ImageTestAssets,
    model_id: str,
):
    """Test that limit_mm_per_prompt accurately restricts multiple images."""
    # We only allow 1 image
    ctx = build_model_context(
        model_id,
        mm_processor_kwargs={},
        limit_mm_per_prompt={"image": 1},
    )
    processor = MULTIMODAL_REGISTRY.create_processor(ctx.model_config)

    # Provide 2 images in the prompt
    prompt = "<image><image>"
    # image_assets usually has multiple images
    images = [asset.pil_image for asset in image_assets][:2]
    if len(images) < 2:
        images = [images[0], images[0]]

    mm_data = {"image": images}

    # Expect ValueError when exceeding limit
    with pytest.raises(ValueError, match="At most 1 image"):
        processor(
            prompt,
            mm_items=processor.info.parse_mm_data(mm_data),
            hf_processor_mm_kwargs={},
        )
```
**EN:** This test validates `test_limit_mm_per_prompt`. It uses parameterization over `model_id`. Key inputs are `image_assets`, `model_id`. It checks an expected failure path with `pytest.raises`. It drives client-facing request creation through the API surface under test.
**CN:** 这个测试验证 `test_limit_mm_per_prompt`。 它通过参数化组合 `model_id`。 关键输入包括 `image_assets`、`model_id`。 它使用 `pytest.raises` 检查预期失败路径。 它通过被测 API 表面触发面向客户端的请求创建流程。

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
- **Stdlib / 标准库**: `collections.abc.Mapping`
- **Third-party / 第三方**: `PIL.Image`, `pytest`, `torch`
- **Project / 项目内**: `vllm.model_executor.models.gemma4_mm.Gemma4ImagePixelInputs`, `vllm.multimodal.MULTIMODAL_REGISTRY`, `vllm.multimodal.inputs.MultiModalFieldConfig`
- **Local relative imports / 本地相对导入**: `....conftest.ImageTestAssets`, `...utils.build_model_context`
