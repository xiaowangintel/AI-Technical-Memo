# test_qwen3_vl_mrope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/model_executor/test_qwen3_vl_mrope.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Qwen3 Vl Mrope behavior in the Model Executor test area through focused pytest scenarios. It focuses on scenarios such as Force CPU Default Device, Dummyvisionconfig, Dummyconfig. / 该文件在 Model Executor 测试域中，通过有针对性的 pytest 场景验证 Qwen3 Vl Mrope 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-16)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import dataclasses
import random
from dataclasses import dataclass

import pytest
import torch

from vllm.model_executor.models.qwen3_vl import Qwen3VLForConditionalGeneration
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalFieldElem,
    MultiModalKwargsItem,
    PlaceholderRange,
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `dataclasses`, `random`, `pytest`, `torch`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Fixture: _force_cpu_default_device (lines 19-26)
```python
@pytest.fixture(autouse=True, scope="module")
def _force_cpu_default_device():
    # _get_mrope_input_positions returns CPU tensors (via torch.from_numpy).
    # Ensure the default device is CPU so the rest of the test tensors match.
    original = torch.get_default_device()
    torch.set_default_device("cpu")
    yield
    torch.set_default_device(original)
```
**EN:** Provides a pytest fixture for Force CPU Default Device. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `pytest.fixture`, `torch.get_default_device`, `torch.set_default_device`.
**CN:** 该代码块定义 pytest 夹具 `_force_cpu_default_device`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `pytest.fixture`, `torch.get_default_device`, `torch.set_default_device` 构造或返回测试所需的值。

### Constants / assignments (lines 29-29)
```python
IMAGE_TOKEN_ID = 999
```
**EN:** Defines shared constants or configuration objects like `IMAGE_TOKEN_ID`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `IMAGE_TOKEN_ID`），供后续测试重复使用。

### Constants / assignments (lines 30-30)
```python
VIDEO_TOKEN_ID = 888
```
**EN:** Defines shared constants or configuration objects like `VIDEO_TOKEN_ID`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `VIDEO_TOKEN_ID`），供后续测试重复使用。

### Constants / assignments (lines 31-31)
```python
VISION_START_TOKEN_ID = 777
```
**EN:** Defines shared constants or configuration objects like `VISION_START_TOKEN_ID`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `VISION_START_TOKEN_ID`），供后续测试重复使用。

### Constants / assignments (lines 32-32)
```python
VISION_END_TOKEN_ID = 778
```
**EN:** Defines shared constants or configuration objects like `VISION_END_TOKEN_ID`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `VISION_END_TOKEN_ID`），供后续测试重复使用。

### Class: DummyVisionConfig (lines 35-37)
```python
@dataclass
class DummyVisionConfig:
    spatial_merge_size: int = 1
```
**EN:** Groups related scenarios for Dummyvisionconfig.
**CN:** 该类把与 Dummyvisionconfig 相关的场景组织在一起。

### Class: DummyConfig (lines 40-48)
```python
@dataclass
class DummyConfig:
    image_token_id: int = IMAGE_TOKEN_ID
    video_token_id: int = VIDEO_TOKEN_ID
    vision_start_token_id: int = VISION_START_TOKEN_ID
    vision_end_token_id: int = VISION_END_TOKEN_ID
    vision_config: DummyVisionConfig = dataclasses.field(
        default_factory=DummyVisionConfig
    )
```
**EN:** Groups related scenarios for Dummyconfig.
**CN:** 该类把与 Dummyconfig 相关的场景组织在一起。

### Helper: make_video_embedding (lines 51-93)
```python
def make_video_embedding(
    t, h, w, interleave_text_tokens: tuple[int, int], video_pruning_rate: float = 0.0
):
    """
    Helper function to make a video embedding for a given video size and pruning rate.

    Args:
        t: Number of frames.
        h: Number of rows.
        w: Number of columns.
        interleave_text_tokens: Tuple of minimum and maximum number of text tokens to
            interleave with the video.
        video_pruning_rate: Pruning rate for the video.

    Returns:
        Tuple of (unpruned_tokens_sequence, pruned_tokens_sequence, retention_mask)
    """
    unpruned_tokens_sequence = []
    population = list(range(1, 100))
# ... omitted for brevity ...

        unpruned_tokens_sequence.extend(prefix_tokens)
        unpruned_tokens_sequence.extend(vision_tokens)

    unpruned_tokens_sequence = torch.tensor(unpruned_tokens_sequence, dtype=torch.long)
    video_token_mask = unpruned_tokens_sequence == VIDEO_TOKEN_ID

    pruning_mask = torch.bernoulli(video_token_mask.float() * video_pruning_rate).bool()  # type: ignore[attr-defined]
    # Sanity check that we don't prune what should not be pruned.
    assert not pruning_mask[~video_token_mask].any()

    retention_mask = ~pruning_mask
    pruned_tokens_sequence = unpruned_tokens_sequence[retention_mask]
    return unpruned_tokens_sequence, pruned_tokens_sequence, retention_mask
```
**EN:** Helper function to make a video embedding for a given video size and pruning rate. It coordinates operations such as `list`, `range`, `torch.tensor`.
**CN:** 该辅助函数为 Make Video Embedding 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `list`, `range`, `torch.tensor` 等操作。

### Test: test_match_qwen3vl_mrope_evs_on (lines 96-237)
```python
@pytest.mark.parametrize("spatial_merge_size", [1, 2])
@pytest.mark.parametrize("grid_thw", [[3, 8, 7], [128, 10, 12]])
@pytest.mark.parametrize("num_prefix_tokens", [1, 11])
@pytest.mark.parametrize("num_suffix_tokens", [0, 7])
@pytest.mark.parametrize("video_pruning_rate", [0, 0.25, 0.75])
@pytest.mark.parametrize("interleave_text_tokens", [(0, 0), (1, 4)])
def test_match_qwen3vl_mrope_evs_on(
    spatial_merge_size: int,
    num_prefix_tokens: int,
    grid_thw: tuple[int, int, int],
    num_suffix_tokens: int,
    video_pruning_rate: float,
    interleave_text_tokens: tuple[int, int],
):
    hf_config = DummyConfig()
    hf_config.vision_config.spatial_merge_size = spatial_merge_size

    t, h, w = grid_thw
    population = list(range(1, 100))
# ... omitted for brevity ...
    # Paranoia check that computed_mrope is wrong.
    assert not torch.equal(computed_mrope, expected_mrope_masked)

    _, actual_mrope, _ = Qwen3VLForConditionalGeneration._recompute_mrope_positions(
        input_ids=input_tokens_pruned,
        multimodal_embeddings=multimodal_embeddings,
        mrope_positions=computed_mrope,
        num_computed_tokens=len(prefix_tokens),
        vision_start_token_id=hf_config.vision_start_token_id,
        image_token_id=hf_config.image_token_id,
        video_token_id=hf_config.video_token_id,
    )

    assert torch.equal(actual_mrope, expected_mrope_masked)
```
**EN:** Checks Match Qwen3vl Mrope Evs On under a focused test scenario. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `DummyConfig`, `list` before asserting the expected outcome.
**CN:** 该测试用例验证 Match Qwen3vl Mrope Evs On 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `DummyConfig`, `list` 驱动目标逻辑，再断言预期结果。

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
- **Standard library / 标准库**: `dataclasses`, `random`
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.models.qwen3_vl`, `vllm.multimodal.inputs`
