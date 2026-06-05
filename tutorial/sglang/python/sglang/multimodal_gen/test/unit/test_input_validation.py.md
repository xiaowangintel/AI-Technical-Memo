# test_input_validation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/test_input_validation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates input validation with focused assertions and fixtures. Key symbols include `_make_batch`, `_make_server_args`, `_DummyTI2IConfig`. / 该测试模块通过有针对性的断言与夹具，验证 input validation 的实现。 关键符号包括 `_make_batch`, `_make_server_args`, `_DummyTI2IConfig`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Imports and module setup / 导入与模块初始化
```python
"""Unit tests for InputValidationStage.preprocess_condition_image resolution logic."""

import unittest
from unittest.mock import MagicMock, patch

import numpy as np
import torch
from diffusers.pipelines.flux2.image_processor import Flux2ImageProcessor
from PIL import Image

from sglang.multimodal_gen.configs.pipeline_configs.base import ModelTaskType
from sglang.multimodal_gen.configs.pipeline_configs.flux import Flux2PipelineConfig
from sglang.multimodal_gen.configs.pipeline_configs.wan import (
    WanI2V480PConfig,
# ...
# Patch path for get_global_server_args used by Stage.__init__
_GLOBAL_ARGS_PATCH = (
    "sglang.multimodal_gen.runtime.pipelines_core.stages.base.get_global_server_args"
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 30-39: Function `_make_batch` / 函数 `_make_batch`
```python
def _make_batch(condition_image: Image.Image, width=None, height=None) -> Req:
    """Create a minimal Req with a condition image and optional user dimensions."""
    sp = SamplingParams(
        seed=42,
        num_outputs_per_prompt=1,
        width=width,
        height=height,
    )
    batch = Req(sampling_params=sp, condition_image=condition_image)
    return batch
```
**EN:** This function drives `_make_batch` with inputs such as `condition_image`, `width`, `height`. Create a minimal Req with a condition image and optional user dimensions.
**CN:** 这个函数负责 `_make_batch`，主要处理 `condition_image`, `width`, `height` 等输入。 文档字符串说明：Create a minimal Req with a condition image and optional user dimensions.

### Lines 42-46: Function `_make_server_args` / 函数 `_make_server_args`
```python
def _make_server_args(pipeline_config):
    """Create a mock ServerArgs with the given pipeline config."""
    sa = MagicMock()
    sa.pipeline_config = pipeline_config
    return sa
```
**EN:** This function drives `_make_server_args` with inputs such as `pipeline_config`. Create a mock ServerArgs with the given pipeline config.
**CN:** 这个函数负责 `_make_server_args`，主要处理 `pipeline_config` 等输入。 文档字符串说明：Create a mock ServerArgs with the given pipeline config.

### Lines 49-68: Class `_DummyTI2IConfig` / 类 `_DummyTI2IConfig`
```python
class _DummyTI2IConfig:
    task_type = ModelTaskType.TI2I

    def __init__(self):
        self.vae_config = MagicMock()
        self.vae_config.get_vae_scale_factor.return_value = 8

    def preprocess_vae_image(self, batch, vae_image_processor):
        return None

    def calculate_condition_image_size(self, image, width, height):
        return None

    def preprocess_condition_image(
# ...
        return image, (target_width, target_height)

    def prepare_calculated_size(self, image):
        return image.size
```
**EN:** This class models `_DummyTI2IConfig`. Important methods include `__init__`, `preprocess_vae_image`, `calculate_condition_image_size`, `preprocess_condition_image`.
**CN:** 该类实现 `_DummyTI2IConfig`。 其中较重要的方法包括 `__init__`, `preprocess_vae_image`, `calculate_condition_image_size`, `preprocess_condition_image`。

### Lines 71-98: Class `TestCalculateDimensionsFromArea` / 类 `TestCalculateDimensionsFromArea`
```python
class TestCalculateDimensionsFromArea(unittest.TestCase):
    """Tests for InputValidationStage._calculate_dimensions_from_area."""

    def test_square_aspect_ratio(self):
        # area=921600, aspect=1.0, mod=16 → sqrt(921600)=~960
        w, h = InputValidationStage._calculate_dimensions_from_area(921600, 1.0, 16)
        self.assertEqual(w % 16, 0)
        self.assertEqual(h % 16, 0)
        self.assertEqual((w, h), (960, 960))

    def test_16_9_aspect_ratio(self):
        # aspect = 720/1280 = 0.5625
        w, h = InputValidationStage._calculate_dimensions_from_area(921600, 9 / 16, 16)
        self.assertEqual(w % 16, 0)
# ...
        # Ensure dimensions are always multiples of mod_value
        w, h = InputValidationStage._calculate_dimensions_from_area(500000, 1.3, 16)
        self.assertEqual(w % 16, 0)
        self.assertEqual(h % 16, 0)
```
**EN:** This class models `TestCalculateDimensionsFromArea` as a specialization of `unittest.TestCase`. Tests for InputValidationStage._calculate_dimensions_from_area. Important methods include `test_square_aspect_ratio`, `test_16_9_aspect_ratio`, `test_9_16_aspect_ratio`, `test_mod_alignment`.
**CN:** 该类实现 `TestCalculateDimensionsFromArea`，并继承/扩展 `unittest.TestCase`。 文档字符串指出：Tests for InputValidationStage._calculate_dimensions_from_area. 其中较重要的方法包括 `test_square_aspect_ratio`, `test_16_9_aspect_ratio`, `test_9_16_aspect_ratio`, `test_mod_alignment`。

### Lines 101-188: Class `TestPreprocessConditionImageResolution` / 类 `TestPreprocessConditionImageResolution`
```python
class TestPreprocessConditionImageResolution(unittest.TestCase):
    """Tests for the WanI2V480PConfig branch of preprocess_condition_image.

    Verifies that:
    - Aspect ratio always comes from the condition image
    - User-specified width/height controls target area (scale)
    - Output is clamped to max_area when user dimensions exceed it
    - Dimensions are always mod-aligned
    """

    def setUp(self):
        with patch(_GLOBAL_ARGS_PATCH, return_value=MagicMock()):
            self.stage = InputValidationStage()

# ...
        server_args = _make_server_args(WanI2V720PConfig())
        self.stage.preprocess_condition_image(batch, server_args, 1920, 1080)
        self.assertIsInstance(batch.condition_image, Image.Image)
        self.assertEqual((batch.width, batch.height), (1280, 720))
```
**EN:** This class models `TestPreprocessConditionImageResolution` as a specialization of `unittest.TestCase`. Tests for the WanI2V480PConfig branch of preprocess_condition_image. Important methods include `setUp`, `_run`, `test_720p_no_user_dims_16_9_image`, `test_720p_no_user_dims_9_16_image`.
**CN:** 该类实现 `TestPreprocessConditionImageResolution`，并继承/扩展 `unittest.TestCase`。 文档字符串指出：Tests for the WanI2V480PConfig branch of preprocess_condition_image. 其中较重要的方法包括 `setUp`, `_run`, `test_720p_no_user_dims_16_9_image`, `test_720p_no_user_dims_9_16_image`。

### Lines 191-236: Class `TestFlux2ConditionImagePreprocess` / 类 `TestFlux2ConditionImagePreprocess`
```python
class TestFlux2ConditionImagePreprocess(unittest.TestCase):
    def test_matches_official_flux2_image_processor(self):
        config = Flux2PipelineConfig()
        config.vae_config.arch_config.vae_scale_factor = 8
        processor = Flux2ImageProcessor(vae_scale_factor=16)
        image = Image.fromarray(
            np.arange(1792 * 1216 * 3, dtype=np.uint8).reshape(1216, 1792, 3),
            mode="RGB",
        )

        size = config.calculate_condition_image_size(image, image.width, image.height)
        self.assertEqual(size, (1232, 832))

        processed, processed_size = config.preprocess_condition_image(
# ...
            mock_add_stages.call_args.kwargs["image_vae_stage_kwargs"][
                "vae_image_processor"
            ],
        )
```
**EN:** This class models `TestFlux2ConditionImagePreprocess` as a specialization of `unittest.TestCase`. Important methods include `test_matches_official_flux2_image_processor`, `test_runtime_pipeline_uses_flux2_image_processor`.
**CN:** 该类实现 `TestFlux2ConditionImagePreprocess`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `test_matches_official_flux2_image_processor`, `test_runtime_pipeline_uses_flux2_image_processor`。

### Lines 239-271: Class `TestFlux2TI2ISizeResolution` / 类 `TestFlux2TI2ISizeResolution`
```python
class TestFlux2TI2ISizeResolution(unittest.TestCase):
    def setUp(self):
        with patch(_GLOBAL_ARGS_PATCH, return_value=MagicMock()):
            self.stage = InputValidationStage()
        self.config = _DummyTI2IConfig()

    def test_uses_condition_image_size_when_width_height_not_explicit(self):
        image = Image.new("RGB", (1255, 833), color="red")
        batch = _make_batch(image)
        batch.extra = {}

        self.stage.preprocess_condition_image(
            batch,
            _make_server_args(self.config),
# ...
            image.height,
        )

        self.assertEqual((batch.width, batch.height), (768, 512))
```
**EN:** This class models `TestFlux2TI2ISizeResolution` as a specialization of `unittest.TestCase`. Important methods include `setUp`, `test_uses_condition_image_size_when_width_height_not_explicit`, `test_preserves_explicit_width_height_for_ti2i`.
**CN:** 该类实现 `TestFlux2TI2ISizeResolution`，并继承/扩展 `unittest.TestCase`。 其中较重要的方法包括 `setUp`, `test_uses_condition_image_size_when_width_height_not_explicit`, `test_preserves_explicit_width_height_for_ti2i`。

### Lines 272-275: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Pipeline orchestration / 流水线编排
- Sampling parameter control / 采样参数控制
- Automated verification / 自动化验证
- Image generation flow / 图像生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.configs.pipeline_configs.flux`, `sglang.multimodal_gen.configs.pipeline_configs.wan`, `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.runtime.pipelines.flux_2`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.input_validation`
- **External / 外部**: `unittest`, `unittest.mock`, `numpy`, `torch`, `diffusers.pipelines.flux2.image_processor`, `PIL`
