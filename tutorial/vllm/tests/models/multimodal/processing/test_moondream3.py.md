# test_moondream3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/processing/test_moondream3.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 23 test(s), 0 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 23 个测试、0 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L12-L19)
```python
import numpy as np
import pytest
import torch

from vllm.multimodal import MULTIMODAL_REGISTRY

from ....conftest import ImageTestAssets
from ...utils import build_model_context
```
**EN:** Imports third-party packages like `PIL.Image`, `numpy`, `pytest`, project helpers such as `vllm.multimodal.MULTIMODAL_REGISTRY`, `vllm.transformers_utils.processors.moondream3.Moondream3Processor`, `vllm.transformers_utils.processors.moondream3.select_tiling`.
**CN:** 导入第三方包（如 `PIL.Image`、`numpy`、`pytest`）、项目内辅助模块（如 `vllm.multimodal.MULTIMODAL_REGISTRY`、`vllm.transformers_utils.processors.moondream3.Moondream3Processor`、`vllm.transformers_utils.processors.moondream3.select_tiling`）。

### Module setup / 模块级配置: MOONDREAM3_MODEL_ID, EXPECTED_IMAGE_TOKENS, CROP_SIZE (L21-L27)
```python
MOONDREAM3_MODEL_ID = "moondream/moondream3-preview"
# Expected multimodal prefix: BOS + 729 image tokens.
EXPECTED_IMAGE_TOKENS = 730
# Vision encoder constants
CROP_SIZE = 378
PATCH_SIZE = 14
MAX_CROPS = 12
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MOONDREAM3_MODEL_ID`, `EXPECTED_IMAGE_TOKENS`, `CROP_SIZE`, `PATCH_SIZE`, `MAX_CROPS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MOONDREAM3_MODEL_ID`、`EXPECTED_IMAGE_TOKENS`、`CROP_SIZE`、`PATCH_SIZE`、`MAX_CROPS`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_processor_creation (L30-L38)
```python
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_processor_creation(model_id: str):
    """Test that Moondream3 processor can be created."""
    ctx = build_model_context(
        model_id,
        limit_mm_per_prompt={"image": 1},
    )
    processor = MULTIMODAL_REGISTRY.create_processor(ctx.model_config)
    assert processor is not None
```
**EN:** This test validates `test_processor_creation`. It uses parameterization over `model_id`. Key inputs are `model_id`. It drives client-facing request creation through the API surface under test. The main assertion is `processor is not None`.
**CN:** 这个测试验证 `test_processor_creation`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `processor is not None`。

### Test / 测试: test_processor_apply (L41-L69)
```python
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_processor_apply(
    image_assets: ImageTestAssets,
    model_id: str,
):
    """Test that Moondream3 processor can process inputs.

    NOTE: The prompt includes the leading BOS token because Moondream3
    pre-fills BOS and image embeddings together.
    """
    ctx = build_model_context(
        model_id,
        limit_mm_per_prompt={"image": 1},
    )
    processor = MULTIMODAL_REGISTRY.create_processor(ctx.model_config)

    prompt = "<|endoftext|><image><|md_reserved_0|>query<|md_reserved_1|>What is this?<|md_reserved_2|>"  # noqa: E501
    mm_data = {"image": [image_assets[0].pil_image]}

    processed_inputs = processor(
        prompt,
        mm_items=processor.info.parse_mm_data(mm_data),
        hf_processor_mm_kwargs={},
    )

    assert "prompt_token_ids" in processed_inputs
    image_placeholders = processed_inputs["mm_placeholders"]["image"]
    assert len(image_placeholders) == 1
    assert image_placeholders[0].length == EXPECTED_IMAGE_TOKENS
```
**EN:** This test validates `test_processor_apply`. It uses parameterization over `model_id`. Key inputs are `image_assets`, `model_id`. It drives client-facing request creation through the API surface under test. The main assertion is `'prompt_token_ids' in processed_inputs` and `len(image_placeholders) == 1`.
**CN:** 这个测试验证 `test_processor_apply`。 它通过参数化组合 `model_id`。 关键输入包括 `image_assets`、`model_id`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `'prompt_token_ids' in processed_inputs` and `len(image_placeholders) == 1`。

### Test / 测试: test_processor_pixel_values (L72-L104)
```python
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_processor_pixel_values(
    image_assets: ImageTestAssets,
    model_id: str,
):
    """Test that pixel values are correctly produced."""
    ctx = build_model_context(
        model_id,
        limit_mm_per_prompt={"image": 1},
    )
    processor = MULTIMODAL_REGISTRY.create_processor(ctx.model_config)

    prompt = "<|endoftext|><image><|md_reserved_0|>query<|md_reserved_1|>What is this?<|md_reserved_2|>"  # noqa: E501
    mm_data = {"image": [image_assets[0].pil_image]}

    processed_inputs = processor(
        prompt,
        mm_items=processor.info.parse_mm_data(mm_data),
# ... 7 lines omitted for brevity ...
    assert "pixel_values" in mm_data_result

    # Verify pixel_values shape
    pixel_values = mm_data_result["pixel_values"]
    assert pixel_values.dim() == 5  # [batch, num_crops, C, H, W]
    assert pixel_values.shape[2] == 3  # RGB channels
    assert pixel_values.shape[3] == 378  # crop height
    assert pixel_values.shape[4] == 378  # crop width
```
**EN:** This test validates `test_processor_pixel_values`. It uses parameterization over `model_id`. Key inputs are `image_assets`, `model_id`. It drives client-facing request creation through the API surface under test. The main assertion is `mm_kwargs is not None` and `'pixel_values' in mm_data_result`.
**CN:** 这个测试验证 `test_processor_pixel_values`。 它通过参数化组合 `model_id`。 关键输入包括 `image_assets`、`model_id`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `mm_kwargs is not None` and `'pixel_values' in mm_data_result`。

### Test / 测试: test_processor_image_token_expansion (L107-L129)
```python
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_processor_image_token_expansion(
    image_assets: ImageTestAssets,
    model_id: str,
):
    """Test that <image> placeholder is expanded to correct number of tokens."""
    ctx = build_model_context(
        model_id,
        limit_mm_per_prompt={"image": 1},
    )
    processor = MULTIMODAL_REGISTRY.create_processor(ctx.model_config)

    prompt = "<|endoftext|><image><|md_reserved_0|>query<|md_reserved_1|>Describe.<|md_reserved_2|>"  # noqa: E501
    mm_data = {"image": [image_assets[0].pil_image]}

    processed_inputs = processor(
        prompt,
        mm_items=processor.info.parse_mm_data(mm_data),
        hf_processor_mm_kwargs={},
    )
    image_placeholders = processed_inputs["mm_placeholders"]["image"]
    assert len(image_placeholders) == 1
    assert image_placeholders[0].length == EXPECTED_IMAGE_TOKENS
```
**EN:** This test validates `test_processor_image_token_expansion`. It uses parameterization over `model_id`. Key inputs are `image_assets`, `model_id`. It drives client-facing request creation through the API surface under test. The main assertion is `len(image_placeholders) == 1` and `image_placeholders[0].length == EXPECTED_IMAGE_TOKENS`.
**CN:** 这个测试验证 `test_processor_image_token_expansion`。 它通过参数化组合 `model_id`。 关键输入包括 `image_assets`、`model_id`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(image_placeholders) == 1` and `image_placeholders[0].length == EXPECTED_IMAGE_TOKENS`。

### Test / 测试: test_multi_crop_tiling (L132-L151)
```python
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_multi_crop_tiling(
    model_id: str,
):
    """Test that large images produce correct multi-crop tiling."""
    from PIL import Image

    from vllm.transformers_utils.processors.moondream3 import Moondream3Processor

    processor = Moondream3Processor.from_pretrained(model_id, trust_remote_code=True)

    # Create a large image that requires multiple crops
    large_image = Image.new("RGB", (1000, 1000), color="blue")
    pixel_values, tiling = processor.preprocess_image(large_image)

    # Large images should produce more than 1x1 tiling
    assert tiling[0] >= 1 and tiling[1] >= 1
    # Check that we have global crop + local crops
    expected_crops = tiling[0] * tiling[1] + 1
    assert pixel_values.shape[0] == expected_crops
```
**EN:** This test validates `test_multi_crop_tiling`. It uses parameterization over `model_id`. Key inputs are `model_id`. The main assertion is `tiling[0] >= 1 and tiling[1] >= 1` and `pixel_values.shape[0] == expected_crops`.
**CN:** 这个测试验证 `test_multi_crop_tiling`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`。 核心断言是 `tiling[0] >= 1 and tiling[1] >= 1` and `pixel_values.shape[0] == expected_crops`。

### Test / 测试: test_tiling_various_sizes (L154-L185)
```python
@pytest.mark.parametrize(
    "image_size",
    [
        (500, 500),
        (800, 600),
        (1920, 1080),
    ],
)
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_tiling_various_sizes(
    image_size: tuple[int, int],
    model_id: str,
):
    """Test tiling with various image sizes."""
    from PIL import Image

    from vllm.transformers_utils.processors.moondream3 import Moondream3Processor

# ... 6 lines omitted for brevity ...
    # Basic shape checks
    assert pixel_values.dim() == 4  # [num_crops, C, H, W]
    assert pixel_values.shape[1] == 3  # RGB
    assert pixel_values.shape[2] == 378  # crop height
    assert pixel_values.shape[3] == 378  # crop width

    # Tiling should respect max_crops (12)
    assert tiling[0] * tiling[1] <= 12
```
**EN:** This test validates `test_tiling_various_sizes`. It uses parameterization over `image_size`. Key inputs are `image_size`, `model_id`. The main assertion is `pixel_values.dim() == 4` and `pixel_values.shape[1] == 3`.
**CN:** 这个测试验证 `test_tiling_various_sizes`。 它通过参数化组合 `image_size`。 关键输入包括 `image_size`、`model_id`。 核心断言是 `pixel_values.dim() == 4` and `pixel_values.shape[1] == 3`。

### Test / 测试: test_pixel_normalization (L188-L206)
```python
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_pixel_normalization(
    model_id: str,
):
    """Test that pixel values are normalized to [-1, 1] range."""
    from PIL import Image

    from vllm.transformers_utils.processors.moondream3 import Moondream3Processor

    processor = Moondream3Processor.from_pretrained(model_id, trust_remote_code=True)

    # Create test image
    image = Image.new("RGB", (378, 378), color="green")
    pixel_values, _ = processor.preprocess_image(image)

    # Normalization: (x - 0.5) / 0.5 = 2*x - 1
    # For input [0, 1], output should be [-1, 1]
    assert pixel_values.min() >= -1.0
    assert pixel_values.max() <= 1.0
```
**EN:** This test validates `test_pixel_normalization`. It uses parameterization over `model_id`. Key inputs are `model_id`. The main assertion is `pixel_values.min() >= -1.0` and `pixel_values.max() <= 1.0`.
**CN:** 这个测试验证 `test_pixel_normalization`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`。 核心断言是 `pixel_values.min() >= -1.0` and `pixel_values.max() <= 1.0`。

### Test / 测试: test_chat_template_with_image (L209-L238)
```python
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_chat_template_with_image(
    image_assets: ImageTestAssets,
    model_id: str,
):
    """Test that chat template correctly formats BOS + image + prompt."""
    ctx = build_model_context(
        model_id,
        limit_mm_per_prompt={"image": 1},
    )
    processor = MULTIMODAL_REGISTRY.create_processor(ctx.model_config)
    tokenizer = ctx.tokenizer

    # Use the chat template format
    prompt = "<|endoftext|><image><|md_reserved_0|>query<|md_reserved_1|>What is this?<|md_reserved_2|>"  # noqa: E501
    mm_data = {"image": [image_assets[0].pil_image]}

    processed_inputs = processor(
        prompt,
        mm_items=processor.info.parse_mm_data(mm_data),
        hf_processor_mm_kwargs={},
    )
    token_ids = processed_inputs["prompt_token_ids"]

    # BOS token (<|endoftext|>) should be token ID 0
    bos_token_id = tokenizer.encode("<|endoftext|>", add_special_tokens=False)[0]
    assert bos_token_id == 0

    # First token should be BOS
    assert token_ids[0] == bos_token_id
```
**EN:** This test validates `test_chat_template_with_image`. It uses parameterization over `model_id`. Key inputs are `image_assets`, `model_id`. It drives client-facing request creation through the API surface under test. The main assertion is `bos_token_id == 0` and `token_ids[0] == bos_token_id`.
**CN:** 这个测试验证 `test_chat_template_with_image`。 它通过参数化组合 `model_id`。 关键输入包括 `image_assets`、`model_id`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `bos_token_id == 0` and `token_ids[0] == bos_token_id`。

### Test / 测试: test_chat_template_content_list_uses_moondream_image_prefix (L241-L298)
```python
@pytest.mark.parametrize(
    "content",
    [
        pytest.param(
            [
                {
                    "type": "image_url",
                    "image_url": {"url": "https://example.invalid/image.png"},
                },
                {"type": "text", "text": "What is in this image?"},
            ],
            id="image-first",
        ),
        pytest.param(
            [
                {"type": "text", "text": "What is in this image?"},
                {
                    "type": "image_url",
# ... 32 lines omitted for brevity ...
    processed_inputs = processor(
        prompt,
        mm_items=processor.info.parse_mm_data({"image": [image_assets[0].pil_image]}),
        hf_processor_mm_kwargs={},
    )
    image_placeholders = processed_inputs["mm_placeholders"]["image"]
    assert len(image_placeholders) == 1
    assert image_placeholders[0].length == EXPECTED_IMAGE_TOKENS
```
**EN:** This test validates `test_chat_template_content_list_uses_moondream_image_prefix`. It uses parameterization over `content`. Key inputs are `image_assets`, `content`, `model_id`. It drives client-facing request creation through the API surface under test. The main assertion is `prompt == expected_prompt` and `len(image_placeholders) == 1`.
**CN:** 这个测试验证 `test_chat_template_content_list_uses_moondream_image_prefix`。 它通过参数化组合 `content`。 关键输入包括 `image_assets`、`content`、`model_id`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `prompt == expected_prompt` and `len(image_placeholders) == 1`。

### Test / 测试: test_bos_token_always_first (L301-L327)
```python
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_bos_token_always_first(
    image_assets: ImageTestAssets,
    model_id: str,
):
    """Test that BOS token (ID 0) is always at position 0."""
    ctx = build_model_context(
        model_id,
        limit_mm_per_prompt={"image": 1},
    )
    processor = MULTIMODAL_REGISTRY.create_processor(ctx.model_config)

    # Start with BOS token explicitly
    prompt = "<|endoftext|><image><|md_reserved_0|>query<|md_reserved_1|>Describe this image.<|md_reserved_2|>"  # noqa: E501
    mm_data = {"image": [image_assets[0].pil_image]}

    processed_inputs = processor(
        prompt,
        mm_items=processor.info.parse_mm_data(mm_data),
        hf_processor_mm_kwargs={},
    )
    token_ids = processed_inputs["prompt_token_ids"]

    # Token ID 0 (<|endoftext|>) should be the first token
    assert token_ids[0] == 0, (
        f"Expected BOS token (0) at position 0, got {token_ids[0]}"
    )
```
**EN:** This test validates `test_bos_token_always_first`. It uses parameterization over `model_id`. Key inputs are `image_assets`, `model_id`. It drives client-facing request creation through the API surface under test. The main assertion is `token_ids[0] == 0`.
**CN:** 这个测试验证 `test_bos_token_always_first`。 它通过参数化组合 `model_id`。 关键输入包括 `image_assets`、`model_id`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `token_ids[0] == 0`。

### Test / 测试: test_processor_with_small_image (L330-L348)
```python
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_processor_with_small_image(
    model_id: str,
):
    """Test processor with image smaller than crop size."""
    from PIL import Image

    from vllm.transformers_utils.processors.moondream3 import Moondream3Processor

    processor = Moondream3Processor.from_pretrained(model_id, trust_remote_code=True)

    # Small image (smaller than crop size)
    small_image = Image.new("RGB", (100, 100), color="yellow")
    pixel_values, tiling = processor.preprocess_image(small_image)

    # Small images should use 1x1 tiling
    assert tiling == (1, 1)
    # Should have 2 crops (global + 1 local)
    assert pixel_values.shape[0] == 2
```
**EN:** This test validates `test_processor_with_small_image`. It uses parameterization over `model_id`. Key inputs are `model_id`. The main assertion is `tiling == (1, 1)` and `pixel_values.shape[0] == 2`.
**CN:** 这个测试验证 `test_processor_with_small_image`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`。 核心断言是 `tiling == (1, 1)` and `pixel_values.shape[0] == 2`。

### Test / 测试: test_preprocess_image_accepts_non_pil_inputs (L351-L384)
```python
@pytest.mark.parametrize(
    "image_kind",
    [
        pytest.param("numpy_hwc", id="numpy-hwc"),
        pytest.param("numpy_chw", id="numpy-chw"),
        pytest.param("torch_chw", id="torch-chw"),
    ],
)
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_preprocess_image_accepts_non_pil_inputs(
    image_assets: ImageTestAssets,
    image_kind: str,
    model_id: str,
):
    from vllm.transformers_utils.processors.moondream3 import Moondream3Processor

    processor = Moondream3Processor.from_pretrained(model_id, trust_remote_code=True)
    pil_image = image_assets[0].pil_image.convert("RGB")
# ... 8 lines omitted for brevity ...
        image = torch.from_numpy(np.transpose(hwc_array, (2, 0, 1)).copy())

    pixel_values, tiling = processor.preprocess_image(image)

    assert tiling == expected_tiling
    assert pixel_values.shape == expected_pixel_values.shape
    assert pixel_values.dtype == torch.bfloat16
    assert torch.equal(pixel_values, expected_pixel_values)
```
**EN:** This test validates `test_preprocess_image_accepts_non_pil_inputs`. It uses parameterization over `image_kind`. Key inputs are `image_assets`, `image_kind`, `model_id`. The main assertion is `tiling == expected_tiling` and `pixel_values.shape == expected_pixel_values.shape`.
**CN:** 这个测试验证 `test_preprocess_image_accepts_non_pil_inputs`。 它通过参数化组合 `image_kind`。 关键输入包括 `image_assets`、`image_kind`、`model_id`。 核心断言是 `tiling == expected_tiling` and `pixel_values.shape == expected_pixel_values.shape`。

### Test / 测试: test_processor_apply_accepts_non_pil_image_inputs (L387-L416)
```python
@pytest.mark.parametrize("image_kind", ["numpy_chw", "torch_chw"])
@pytest.mark.parametrize("model_id", [MOONDREAM3_MODEL_ID])
def test_processor_apply_accepts_non_pil_image_inputs(
    image_assets: ImageTestAssets,
    image_kind: str,
    model_id: str,
):
    ctx = build_model_context(
        model_id,
        limit_mm_per_prompt={"image": 1},
    )
    processor = MULTIMODAL_REGISTRY.create_processor(ctx.model_config)

    prompt = "<|endoftext|><image><|md_reserved_0|>query<|md_reserved_1|>What is this?<|md_reserved_2|>"  # noqa: E501
    hwc_array = np.asarray(image_assets[0].pil_image.convert("RGB"))
    chw_array = np.transpose(hwc_array, (2, 0, 1)).copy()
    image = chw_array if image_kind == "numpy_chw" else torch.from_numpy(chw_array)

    processed_inputs = processor(
        prompt,
        mm_items=processor.info.parse_mm_data({"image": [image]}),
        hf_processor_mm_kwargs={},
    )

    image_placeholders = processed_inputs["mm_placeholders"]["image"]
    assert len(image_placeholders) == 1
    assert image_placeholders[0].length == EXPECTED_IMAGE_TOKENS

    mm_kwargs = processed_inputs["mm_kwargs"].get_data()
    assert mm_kwargs["pixel_values"].shape[2:] == (3, 378, 378)
```
**EN:** This test validates `test_processor_apply_accepts_non_pil_image_inputs`. It uses parameterization over `image_kind`. Key inputs are `image_assets`, `image_kind`, `model_id`. It drives client-facing request creation through the API surface under test. The main assertion is `len(image_placeholders) == 1` and `image_placeholders[0].length == EXPECTED_IMAGE_TOKENS`.
**CN:** 这个测试验证 `test_processor_apply_accepts_non_pil_image_inputs`。 它通过参数化组合 `image_kind`。 关键输入包括 `image_assets`、`image_kind`、`model_id`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(image_placeholders) == 1` and `image_placeholders[0].length == EXPECTED_IMAGE_TOKENS`。

### Class / 类: TestMoondream3TilingLogic (L419-L484)
```python
class TestMoondream3TilingLogic:
    """CPU-based tests for Moondream3 tiling selection logic.

    These tests validate the select_tiling() function which determines
    how images are divided into crops for the vision encoder.
    """

    def test_small_image_no_tiling(self):
        """Small images should use 1x1 tiling."""
        from vllm.transformers_utils.processors.moondream3 import select_tiling

        tiling = select_tiling(
            height=300, width=300, crop_size=CROP_SIZE, max_crops=MAX_CROPS
        )
# ... 44 lines omitted for brevity ...
        """Tiling should not exceed max_crops."""
        from vllm.transformers_utils.processors.moondream3 import select_tiling

        tiling = select_tiling(
            height=2000, width=2000, crop_size=CROP_SIZE, max_crops=4
        )
        h_tiles, w_tiles = tiling
        assert h_tiles * w_tiles <= 4
```
**EN:** This class groups related scenarios in `TestMoondream3TilingLogic`. It contains 6 test method(s) and 0 supporting method(s). Representative methods include `test_small_image_no_tiling`, `test_exact_crop_size`, `test_large_square_image`.
**CN:** 该类将与 `TestMoondream3TilingLogic` 相关的场景组织在一起。 它包含 6 个测试方法和 0 个辅助方法。 代表性方法包括 `test_small_image_no_tiling`、`test_exact_crop_size`、`test_large_square_image`。

### Test method / 测试方法: TestMoondream3TilingLogic.test_small_image_no_tiling (L426-L433)
```python
    def test_small_image_no_tiling(self):
        """Small images should use 1x1 tiling."""
        from vllm.transformers_utils.processors.moondream3 import select_tiling

        tiling = select_tiling(
            height=300, width=300, crop_size=CROP_SIZE, max_crops=MAX_CROPS
        )
        assert tiling == (1, 1)
```
**EN:** This test validates `TestMoondream3TilingLogic.test_small_image_no_tiling`. The main assertion is `tiling == (1, 1)`.
**CN:** 这个测试验证 `TestMoondream3TilingLogic.test_small_image_no_tiling`。 核心断言是 `tiling == (1, 1)`。

### Test method / 测试方法: TestMoondream3TilingLogic.test_exact_crop_size (L435-L442)
```python
    def test_exact_crop_size(self):
        """Image exactly at crop size should use 1x1."""
        from vllm.transformers_utils.processors.moondream3 import select_tiling

        tiling = select_tiling(
            height=CROP_SIZE, width=CROP_SIZE, crop_size=CROP_SIZE, max_crops=MAX_CROPS
        )
        assert tiling == (1, 1)
```
**EN:** This test validates `TestMoondream3TilingLogic.test_exact_crop_size`. The main assertion is `tiling == (1, 1)`.
**CN:** 这个测试验证 `TestMoondream3TilingLogic.test_exact_crop_size`。 核心断言是 `tiling == (1, 1)`。

### Test method / 测试方法: TestMoondream3TilingLogic.test_large_square_image (L444-L454)
```python
    def test_large_square_image(self):
        """Large square image should use multiple tiles."""
        from vllm.transformers_utils.processors.moondream3 import select_tiling

        tiling = select_tiling(
            height=800, width=800, crop_size=CROP_SIZE, max_crops=MAX_CROPS
        )
        h_tiles, w_tiles = tiling
        assert h_tiles >= 2
        assert w_tiles >= 2
        assert h_tiles * w_tiles <= MAX_CROPS
```
**EN:** This test validates `TestMoondream3TilingLogic.test_large_square_image`. The main assertion is `h_tiles >= 2` and `w_tiles >= 2`.
**CN:** 这个测试验证 `TestMoondream3TilingLogic.test_large_square_image`。 核心断言是 `h_tiles >= 2` and `w_tiles >= 2`。

### Test method / 测试方法: TestMoondream3TilingLogic.test_wide_image (L456-L464)
```python
    def test_wide_image(self):
        """Wide image should have more width tiles."""
        from vllm.transformers_utils.processors.moondream3 import select_tiling

        tiling = select_tiling(
            height=400, width=1200, crop_size=CROP_SIZE, max_crops=MAX_CROPS
        )
        h_tiles, w_tiles = tiling
        assert w_tiles >= h_tiles
```
**EN:** This test validates `TestMoondream3TilingLogic.test_wide_image`. The main assertion is `w_tiles >= h_tiles`.
**CN:** 这个测试验证 `TestMoondream3TilingLogic.test_wide_image`。 核心断言是 `w_tiles >= h_tiles`。

### Test method / 测试方法: TestMoondream3TilingLogic.test_tall_image (L466-L474)
```python
    def test_tall_image(self):
        """Tall image should have more height tiles."""
        from vllm.transformers_utils.processors.moondream3 import select_tiling

        tiling = select_tiling(
            height=1200, width=400, crop_size=CROP_SIZE, max_crops=MAX_CROPS
        )
        h_tiles, w_tiles = tiling
        assert h_tiles >= w_tiles
```
**EN:** This test validates `TestMoondream3TilingLogic.test_tall_image`. The main assertion is `h_tiles >= w_tiles`.
**CN:** 这个测试验证 `TestMoondream3TilingLogic.test_tall_image`。 核心断言是 `h_tiles >= w_tiles`。

### Test method / 测试方法: TestMoondream3TilingLogic.test_respects_max_crops (L476-L484)
```python
    def test_respects_max_crops(self):
        """Tiling should not exceed max_crops."""
        from vllm.transformers_utils.processors.moondream3 import select_tiling

        tiling = select_tiling(
            height=2000, width=2000, crop_size=CROP_SIZE, max_crops=4
        )
        h_tiles, w_tiles = tiling
        assert h_tiles * w_tiles <= 4
```
**EN:** This test validates `TestMoondream3TilingLogic.test_respects_max_crops`. The main assertion is `h_tiles * w_tiles <= 4`.
**CN:** 这个测试验证 `TestMoondream3TilingLogic.test_respects_max_crops`。 核心断言是 `h_tiles * w_tiles <= 4`。

### Class / 类: TestMoondream3VisionShapes (L487-L508)
```python
class TestMoondream3VisionShapes:
    """CPU-based tests for vision encoder expected shapes.

    These tests verify the mathematical relationships between
    crop size, patch size, and token counts.
    """

    def test_expected_patch_count(self):
        """Test 378/14 = 27 patches per side, 729 total."""
        patches_per_side = CROP_SIZE // PATCH_SIZE
        total_patches = patches_per_side**2

        assert patches_per_side == 27
        assert total_patches == EXPECTED_IMAGE_TOKENS - 1

    def test_patch_embedding_input_dim(self):
        """Test patch embedding input dimension."""
        channels = 3
        input_dim = PATCH_SIZE * PATCH_SIZE * channels

        assert input_dim == 14 * 14 * 3
        assert input_dim == 588
```
**EN:** This class groups related scenarios in `TestMoondream3VisionShapes`. It contains 2 test method(s) and 0 supporting method(s). Representative methods include `test_expected_patch_count`, `test_patch_embedding_input_dim`.
**CN:** 该类将与 `TestMoondream3VisionShapes` 相关的场景组织在一起。 它包含 2 个测试方法和 0 个辅助方法。 代表性方法包括 `test_expected_patch_count`、`test_patch_embedding_input_dim`。

### Test method / 测试方法: TestMoondream3VisionShapes.test_expected_patch_count (L494-L500)
```python
    def test_expected_patch_count(self):
        """Test 378/14 = 27 patches per side, 729 total."""
        patches_per_side = CROP_SIZE // PATCH_SIZE
        total_patches = patches_per_side**2

        assert patches_per_side == 27
        assert total_patches == EXPECTED_IMAGE_TOKENS - 1
```
**EN:** This test validates `TestMoondream3VisionShapes.test_expected_patch_count`. The main assertion is `patches_per_side == 27` and `total_patches == EXPECTED_IMAGE_TOKENS - 1`.
**CN:** 这个测试验证 `TestMoondream3VisionShapes.test_expected_patch_count`。 核心断言是 `patches_per_side == 27` and `total_patches == EXPECTED_IMAGE_TOKENS - 1`。

### Test method / 测试方法: TestMoondream3VisionShapes.test_patch_embedding_input_dim (L502-L508)
```python
    def test_patch_embedding_input_dim(self):
        """Test patch embedding input dimension."""
        channels = 3
        input_dim = PATCH_SIZE * PATCH_SIZE * channels

        assert input_dim == 14 * 14 * 3
        assert input_dim == 588
```
**EN:** This test validates `TestMoondream3VisionShapes.test_patch_embedding_input_dim`. The main assertion is `input_dim == 14 * 14 * 3` and `input_dim == 588`.
**CN:** 这个测试验证 `TestMoondream3VisionShapes.test_patch_embedding_input_dim`。 核心断言是 `input_dim == 14 * 14 * 3` and `input_dim == 588`。

### Class / 类: TestMoondream3TauAttention (L511-L553)
```python
class TestMoondream3TauAttention:
    """CPU-based tests for tau attention scaling components.

    These tests validate the tau attention formula used in Moondream3:
    - Token-based: tok_q = tanh(gelu(qkv) @ tau_wq.T)
    - Position-based: tau_pos = 1 + (sigmoid(alpha * log(pos+1)) - 0.5)
    """

    def test_tau_position_range(self):
        """Test tau position scaling produces values in valid range."""
        num_heads = 32
        seq_len = 100

        tau_alpha = torch.randn(num_heads)
# ... 21 lines omitted for brevity ...

        tok_feat = F.gelu(qkv)
        tok_q = torch.tanh(tok_feat @ tau_wq.t())

        assert tok_q.shape == (seq_len, num_heads)
        # tanh output is bounded by [-1, 1]
        assert tok_q.min() >= -1.0
        assert tok_q.max() <= 1.0
```
**EN:** This class groups related scenarios in `TestMoondream3TauAttention`. It contains 2 test method(s) and 0 supporting method(s). Representative methods include `test_tau_position_range`, `test_tau_token_output_range`.
**CN:** 该类将与 `TestMoondream3TauAttention` 相关的场景组织在一起。 它包含 2 个测试方法和 0 个辅助方法。 代表性方法包括 `test_tau_position_range`、`test_tau_token_output_range`。

### Test method / 测试方法: TestMoondream3TauAttention.test_tau_position_range (L519-L534)
```python
    def test_tau_position_range(self):
        """Test tau position scaling produces values in valid range."""
        num_heads = 32
        seq_len = 100

        tau_alpha = torch.randn(num_heads)
        positions = torch.arange(seq_len)

        pos_float = (positions.float() + 1.0).clamp(min=1e-6)
        pos_log = pos_float.log()
        tau_pos = 1.0 + (torch.sigmoid(tau_alpha[:, None] * pos_log[None, :]) - 0.5)

        assert tau_pos.shape == (num_heads, seq_len)
        # tau_pos should be between 0.5 and 1.5
        assert tau_pos.min() >= 0.5
        assert tau_pos.max() <= 1.5
```
**EN:** This test validates `TestMoondream3TauAttention.test_tau_position_range`. The main assertion is `tau_pos.shape == (num_heads, seq_len)` and `tau_pos.min() >= 0.5`.
**CN:** 这个测试验证 `TestMoondream3TauAttention.test_tau_position_range`。 核心断言是 `tau_pos.shape == (num_heads, seq_len)` and `tau_pos.min() >= 0.5`。

### Test method / 测试方法: TestMoondream3TauAttention.test_tau_token_output_range (L536-L553)
```python
    def test_tau_token_output_range(self):
        """Test tau token scaling output is bounded by tanh."""
        import torch.nn.functional as F

        seq_len = 100
        qkv_dim = 6144  # 2048 * 3
        num_heads = 32

        qkv = torch.randn(seq_len, qkv_dim)
        tau_wq = torch.randn(num_heads, qkv_dim)

        tok_feat = F.gelu(qkv)
        tok_q = torch.tanh(tok_feat @ tau_wq.t())

        assert tok_q.shape == (seq_len, num_heads)
        # tanh output is bounded by [-1, 1]
        assert tok_q.min() >= -1.0
        assert tok_q.max() <= 1.0
```
**EN:** This test validates `TestMoondream3TauAttention.test_tau_token_output_range`. The main assertion is `tok_q.shape == (seq_len, num_heads)` and `tok_q.min() >= -1.0`.
**CN:** 这个测试验证 `TestMoondream3TauAttention.test_tau_token_output_range`。 核心断言是 `tok_q.shape == (seq_len, num_heads)` and `tok_q.min() >= -1.0`。

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
- **Third-party / 第三方**: `PIL.Image`, `numpy`, `pytest`, `torch`, `torch.nn.functional`
- **Project / 项目内**: `vllm.multimodal.MULTIMODAL_REGISTRY`, `vllm.transformers_utils.processors.moondream3.Moondream3Processor`, `vllm.transformers_utils.processors.moondream3.select_tiling`
- **Local relative imports / 本地相对导入**: `....conftest.ImageTestAssets`, `...utils.build_model_context`
