# test_keye_vl1_5_mrope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/model_executor/test_keye_vl1_5_mrope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Keye Vl1 5 Mrope behavior in the Model Executor test area through focused pytest scenarios. It focuses on scenarios such as Force CPU Default Device, Dummyvisionconfig, Dummyconfig. / 该文件在 Model Executor 测试域中，通过有针对性的 pytest 场景验证 Keye Vl1 5 Mrope 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-16)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from dataclasses import dataclass, field

import pytest
import torch

from vllm.model_executor.models.keye_vl1_5 import KeyeVL1_5ForConditionalGeneration
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalFieldElem,
    MultiModalKwargsItem,
    PlaceholderRange,
)

pytestmark = pytest.mark.skip_global_cleanup
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `dataclasses`, `pytest`, `torch`, `vllm.model_executor.models.keye_vl1_5`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: _force_cpu_default_device (lines 19-24)
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

### Class: DummyVisionConfig (lines 27-29)
```python
@dataclass
class DummyVisionConfig:
    spatial_merge_size: int = 2
```
**EN:** Groups related scenarios for Dummyvisionconfig.
**CN:** 该类把与 Dummyvisionconfig 相关的场景组织在一起。

### Class: DummyConfig (lines 32-34)
```python
@dataclass
class DummyConfig:
    vision_config: DummyVisionConfig = field(default_factory=DummyVisionConfig)
```
**EN:** Groups related scenarios for Dummyconfig.
**CN:** 该类把与 Dummyconfig 相关的场景组织在一起。

### Helper: make_model (lines 37-40)
```python
def make_model(config: DummyConfig) -> KeyeVL1_5ForConditionalGeneration:
    model = object.__new__(KeyeVL1_5ForConditionalGeneration)
    model.config = config
    return model
```
**EN:** Implements a reusable helper for Make Model, reducing duplication across related tests. It coordinates operations such as `object.__new__`.
**CN:** 该辅助函数为 Make Model 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `object.__new__` 等操作。

### Helper: make_mm_feature (lines 43-68)
```python
def make_mm_feature(
    *,
    modality: str,
    offset: int,
    length: int,
    grid_thw: tuple[int, int, int] | list[tuple[int, int, int]],
    is_embed: list[bool] | None = None,
) -> MultiModalFeatureSpec:
    field_name = "image_grid_thw" if modality == "image" else "video_grid_thw"
    return MultiModalFeatureSpec(
        data=MultiModalKwargsItem(
            {
                field_name: MultiModalFieldElem(
                    data=torch.tensor(grid_thw),
                    field=None,  # HACK.
                ),
            }
        ),
        modality=modality,
        identifier="DUMMY",
        mm_position=PlaceholderRange(
            offset=offset,
            length=length,
            is_embed=None if is_embed is None else torch.tensor(is_embed),
        ),
    )
```
**EN:** Implements a reusable helper for Make Mm Feature, reducing duplication across related tests. It coordinates operations such as `MultiModalFeatureSpec`, `MultiModalKwargsItem`, `PlaceholderRange`.
**CN:** 该辅助函数为 Make Mm Feature 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `MultiModalFeatureSpec`, `MultiModalKwargsItem`, `PlaceholderRange` 等操作。

### Test: test_get_mrope_input_positions_text_only (lines 71-88)
```python
def test_get_mrope_input_positions_text_only():
    model = make_model(DummyConfig())

    positions, delta = model.get_mrope_input_positions(
        input_tokens=[11, 12, 13, 14, 15],
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

### Test: test_get_mrope_input_positions_single_image (lines 91-116)
```python
def test_get_mrope_input_positions_single_image():
    model = make_model(DummyConfig())
    mm_features = [
        make_mm_feature(
            modality="image",
            offset=1,
            length=4,
            grid_thw=(1, 4, 4),
        )
    ]

    positions, delta = model.get_mrope_input_positions(
        input_tokens=[10, 20, 21, 22, 23, 30, 31],
        mm_features=mm_features,
    )

    expected = torch.tensor(
        [
            [0, 1, 1, 1, 1, 3, 4],
            [0, 1, 1, 2, 2, 3, 4],
            [0, 1, 2, 1, 2, 3, 4],
        ]
    )

    assert torch.equal(positions, expected)
    assert delta == -2
```
**EN:** Checks Get Mrope Input Positions Single Image under a focused test scenario. The body exercises logic via `make_model`, `model.get_mrope_input_positions`, `torch.tensor` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Mrope Input Positions Single Image 在特定场景下的行为。 函数体会先通过 `make_model`, `model.get_mrope_input_positions`, `torch.tensor` 驱动目标逻辑，再断言预期结果。

### Test: test_get_mrope_input_positions_video_uses_embed_ranges (lines 119-145)
```python
def test_get_mrope_input_positions_video_uses_embed_ranges():
    model = make_model(DummyConfig())
    mm_features = [
        make_mm_feature(
            modality="video",
            offset=1,
            length=8,
            grid_thw=[(2, 4, 2)],
            is_embed=[False, False, True, True, False, False, True, True],
        )
    ]

    positions, delta = model.get_mrope_input_positions(
        input_tokens=[10, 101, 102, 20, 21, 103, 104, 30, 31, 40, 41],
        mm_features=mm_features,
    )

    expected = torch.tensor(
        [
            [0, 1, 2, 3, 3, 5, 6, 7, 7, 9, 10],
            [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
            [0, 1, 2, 3, 3, 5, 6, 7, 7, 9, 10],
        ]
    )

    assert torch.equal(positions, expected)
    assert delta == 0
```
**EN:** Checks Get Mrope Input Positions Video Uses Embed Ranges under a focused test scenario. The body exercises logic via `make_model`, `model.get_mrope_input_positions`, `torch.tensor` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Mrope Input Positions Video Uses Embed Ranges 在特定场景下的行为。 函数体会先通过 `make_model`, `model.get_mrope_input_positions`, `torch.tensor` 驱动目标逻辑，再断言预期结果。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Multimodal inputs / 多模态输入**
  - **EN:** The tests combine text with image, audio, or video data and verify the modality-specific pipeline.
  - **CN:** 这些测试把文本与图像、音频或视频数据结合起来，验证模态专用处理流程。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.models.keye_vl1_5`, `vllm.multimodal.inputs`
