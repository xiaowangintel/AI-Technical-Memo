# test_paddleocr_vl_mrope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/model_executor/test_paddleocr_vl_mrope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Paddleocr Vl Mrope behavior in the Model Executor test area through focused pytest scenarios. It focuses on scenarios such as Force CPU Default Device, Dummyvisionconfig, Dummyconfig. / 该文件在 Model Executor 测试域中，通过有针对性的 pytest 场景验证 Paddleocr Vl Mrope 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-18)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from dataclasses import dataclass, field

import pytest
import torch

from vllm.model_executor.models.paddleocr_vl import (
    PaddleOCRVLForConditionalGeneration,
)
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalFieldElem,
    MultiModalKwargsItem,
    PlaceholderRange,
)

pytestmark = pytest.mark.skip_global_cleanup
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `dataclasses`, `pytest`, `torch`, `vllm.model_executor.models.paddleocr_vl`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: _force_cpu_default_device (lines 21-26)
```python
@pytest.fixture(autouse=True, scope="module")
def _force_cpu_default_device():
    original = torch.get_default_device()
    torch.set_default_device("cpu")
    yield
    torch.set_default_device(original)
```
**EN:** Provides a pytest fixture for Force CPU Default Device. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `torch.get_default_device`, `torch.set_default_device`.
**CN:** 该代码块定义 pytest 夹具 `_force_cpu_default_device`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `torch.get_default_device`, `torch.set_default_device` 构造或返回测试所需的值。

### Class: DummyVisionConfig (lines 29-32)
```python
@dataclass
class DummyVisionConfig:
    spatial_merge_size: int = 2
    patch_size: int = 14
```
**EN:** Groups related scenarios for Dummyvisionconfig.
**CN:** 该类把与 Dummyvisionconfig 相关的场景组织在一起。

### Class: DummyConfig (lines 35-41)
```python
@dataclass
class DummyConfig:
    image_token_id: int = 151655
    video_token_id: int = 151654
    vision_start_token_id: int = 151652
    vision_end_token_id: int = 151653
    vision_config: DummyVisionConfig = field(default_factory=DummyVisionConfig)
```
**EN:** Groups related scenarios for Dummyconfig.
**CN:** 该类把与 Dummyconfig 相关的场景组织在一起。

### Helper: make_model (lines 44-47)
```python
def make_model(config: DummyConfig) -> PaddleOCRVLForConditionalGeneration:
    model = object.__new__(PaddleOCRVLForConditionalGeneration)
    model.config = config
    return model
```
**EN:** Implements a reusable helper for Make Model, reducing duplication across related tests. It coordinates operations such as `object.__new__`.
**CN:** 该辅助函数为 Make Model 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `object.__new__` 等操作。

### Helper: make_mm_feature (lines 50-68)
```python
def make_mm_feature(
    *,
    offset: int,
    length: int,
    image_grid_thw: tuple[int, int, int],
) -> MultiModalFeatureSpec:
    return MultiModalFeatureSpec(
        data=MultiModalKwargsItem(
            {
                "image_grid_thw": MultiModalFieldElem(
                    data=torch.tensor(image_grid_thw),
                    field=None,
                ),
            }
        ),
        modality="image",
        identifier="DUMMY",
        mm_position=PlaceholderRange(offset=offset, length=length),
    )
```
**EN:** Implements a reusable helper for Make Mm Feature, reducing duplication across related tests. It coordinates operations such as `MultiModalFeatureSpec`, `MultiModalKwargsItem`, `PlaceholderRange`.
**CN:** 该辅助函数为 Make Mm Feature 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `MultiModalFeatureSpec`, `MultiModalKwargsItem`, `PlaceholderRange` 等操作。

### Test: test_get_mrope_input_positions_text_only (lines 71-86)
```python
def test_get_mrope_input_positions_text_only():
    model = make_model(DummyConfig())
    input_tokens = [11, 12, 13, 14, 15]
    positions, delta = model.get_mrope_input_positions(
        input_tokens=input_tokens,
        mm_features=[],
    )
    expected = torch.tensor(
        [
            [0, 1, 2, 3, 4],
            [0, 1, 2, 3, 4],
            [0, 1, 2, 3, 4],
        ]
    )
    assert torch.equal(positions, expected)
    assert delta == 0
```
**EN:** Checks Get Mrope Input Positions Text Only under a focused test scenario. The body exercises logic via `make_model`, `model.get_mrope_input_positions`, `torch.tensor` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Mrope Input Positions Text Only 在特定场景下的行为。 函数体会先通过 `make_model`, `model.get_mrope_input_positions`, `torch.tensor` 驱动目标逻辑，再断言预期结果。

### Test: test_get_mrope_input_positions_single_image (lines 89-127)
```python
def test_get_mrope_input_positions_single_image():
    model = make_model(DummyConfig())
    spatial_merge_size = model.config.vision_config.spatial_merge_size

    t, h, w = 1, 2, 2
    num_image_tokens = t * h * w

    input_tokens = (
        [10]
        + [model.config.vision_start_token_id]
        + [model.config.image_token_id] * num_image_tokens
        + [model.config.vision_end_token_id]
        + [30, 31]
    )

    mm_features = [
        make_mm_feature(
            offset=2,  # 1 (text) + 1 (vision_start)
            length=num_image_tokens,
# ... omitted for brevity ...
        mm_features=mm_features,
    )

    expected = torch.tensor(
        [
            [0, 1, 2, 2, 2, 2, 4, 5, 6],
            [0, 1, 2, 2, 3, 3, 4, 5, 6],
            [0, 1, 2, 3, 2, 3, 4, 5, 6],
        ]
    )

    assert torch.equal(positions, expected)
    expected_delta = (positions.max().item() + 1) - len(input_tokens)
    assert delta == expected_delta
```
**EN:** Checks Get Mrope Input Positions Single Image under a focused test scenario. The body exercises logic via `make_model`, `model.get_mrope_input_positions`, `torch.tensor` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Mrope Input Positions Single Image 在特定场景下的行为。 函数体会先通过 `make_model`, `model.get_mrope_input_positions`, `torch.tensor` 驱动目标逻辑，再断言预期结果。

### Test: test_get_mrope_input_positions_multiple_images (lines 130-172)
```python
def test_get_mrope_input_positions_multiple_images():
    model = make_model(DummyConfig())
    spatial_merge_size = model.config.vision_config.spatial_merge_size

    t1, h1, w1 = 1, 2, 2
    num1 = t1 * h1 * w1

    t2, h2, w2 = 1, 1, 3
    num2 = t2 * h2 * w2

    input_tokens = (
        [10]
        + [model.config.vision_start_token_id]
        + [model.config.image_token_id] * num1
        + [model.config.vision_end_token_id]
        + [20, 21]
        + [model.config.vision_start_token_id]
        + [model.config.image_token_id] * num2
        + [model.config.vision_end_token_id]
# ... omitted for brevity ...
            offset=2 + num1 + 1 + 2 + 1,
            length=num2,
            image_grid_thw=(t2, h2 * spatial_merge_size, w2 * spatial_merge_size),
        ),
    ]

    positions, delta = model.get_mrope_input_positions(
        input_tokens=input_tokens,
        mm_features=mm_features,
    )

    assert positions.shape == (3, 15)
    assert not torch.equal(positions[:, 2:6], torch.arange(4).expand(3, 4) + 2)
    assert not torch.equal(positions[:, 10:13], torch.arange(3).expand(3, 3) + 10)
```
**EN:** Checks Get Mrope Input Positions Multiple Images under a focused test scenario. The body exercises logic via `make_model`, `model.get_mrope_input_positions`, `DummyConfig` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Mrope Input Positions Multiple Images 在特定场景下的行为。 函数体会先通过 `make_model`, `model.get_mrope_input_positions`, `DummyConfig` 驱动目标逻辑，再断言预期结果。

### Test: test_get_mrope_input_positions_image_at_start (lines 175-210)
```python
def test_get_mrope_input_positions_image_at_start():
    model = make_model(DummyConfig())
    spatial_merge_size = model.config.vision_config.spatial_merge_size

    t, h, w = 1, 2, 2
    num_tokens = t * h * w

    input_tokens = (
        [model.config.vision_start_token_id]
        + [model.config.image_token_id] * num_tokens
        + [model.config.vision_end_token_id]
        + [10, 11]
    )

    mm_features = [
        make_mm_feature(
            offset=1,  # start token at index 0
            length=num_tokens,
            image_grid_thw=(t, h * spatial_merge_size, w * spatial_merge_size),
# ... omitted for brevity ...
    positions, delta = model.get_mrope_input_positions(
        input_tokens=input_tokens,
        mm_features=mm_features,
    )

    expected = torch.tensor(
        [
            [0, 1, 1, 1, 1, 3, 4, 5],
            [0, 1, 1, 2, 2, 3, 4, 5],
            [0, 1, 2, 1, 2, 3, 4, 5],
        ]
    )

    assert torch.equal(positions, expected)
```
**EN:** Checks Get Mrope Input Positions Image At Start under a focused test scenario. The body exercises logic via `make_model`, `model.get_mrope_input_positions`, `torch.tensor` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Mrope Input Positions Image At Start 在特定场景下的行为。 函数体会先通过 `make_model`, `model.get_mrope_input_positions`, `torch.tensor` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Multimodal inputs / 多模态输入**
  - **EN:** The tests combine text with image, audio, or video data and verify the modality-specific pipeline.
  - **CN:** 这些测试把文本与图像、音频或视频数据结合起来，验证模态专用处理流程。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.models.paddleocr_vl`, `vllm.multimodal.inputs`
