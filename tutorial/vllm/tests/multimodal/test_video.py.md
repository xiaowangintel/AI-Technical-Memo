# test_video.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/multimodal/test_video.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Video behavior in the Multimodal test area through focused pytest scenarios. It focuses on scenarios such as Testvideoloader1, Testvideoloader2, Video Loader Registry. / 该文件在 Multimodal 测试域中，通过有针对性的 pytest 场景验证 Video 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-20)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from pathlib import Path

import numpy as np
import numpy.typing as npt
import pytest

from vllm.assets.base import get_vllm_public_assets
from vllm.multimodal.video import (
    VIDEO_LOADER_REGISTRY,
    VideoLoader,
)

from .utils import create_long_gop_video, create_video_from_image

pytestmark = pytest.mark.cpu_test

ASSETS_DIR = Path(__file__).parent / "assets"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `pathlib`, `numpy`, `numpy.typing`, `vllm.assets.base`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Constants / assignments (lines 23-23)
```python
NUM_FRAMES = 10
```
**EN:** Defines shared constants or configuration objects like `NUM_FRAMES`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `NUM_FRAMES`），供后续测试重复使用。

### Class: TestVideoLoader1 (lines 28-32)
```python
@VIDEO_LOADER_REGISTRY.register("test_video_loader_1")
class TestVideoLoader1(VideoLoader):
    @classmethod
    def load_bytes(cls, data: bytes, num_frames: int = -1) -> npt.NDArray:
        return FAKE_OUTPUT_1
```
**EN:** Groups related scenarios for Testvideoloader1.
**CN:** 该类把与 Testvideoloader1 相关的场景组织在一起。

### Class: TestVideoLoader2 (lines 35-39)
```python
@VIDEO_LOADER_REGISTRY.register("test_video_loader_2")
class TestVideoLoader2(VideoLoader):
    @classmethod
    def load_bytes(cls, data: bytes, num_frames: int = -1) -> npt.NDArray:
        return FAKE_OUTPUT_2
```
**EN:** Groups related scenarios for Testvideoloader2.
**CN:** 该类把与 Testvideoloader2 相关的场景组织在一起。

### Test: test_video_loader_registry (lines 42-49)
```python
def test_video_loader_registry():
    custom_loader_1 = VIDEO_LOADER_REGISTRY.load("test_video_loader_1")
    output_1 = custom_loader_1.load_bytes(b"test")
    np.testing.assert_array_equal(output_1, FAKE_OUTPUT_1)

    custom_loader_2 = VIDEO_LOADER_REGISTRY.load("test_video_loader_2")
    output_2 = custom_loader_2.load_bytes(b"test")
    np.testing.assert_array_equal(output_2, FAKE_OUTPUT_2)
```
**EN:** Checks Video Loader Registry under a focused test scenario. The body exercises logic via `VIDEO_LOADER_REGISTRY.load`, `custom_loader_1.load_bytes`, `np.testing.assert_array_equal` before asserting the expected outcome.
**CN:** 该测试用例验证 Video Loader Registry 在特定场景下的行为。 函数体会先通过 `VIDEO_LOADER_REGISTRY.load`, `custom_loader_1.load_bytes`, `np.testing.assert_array_equal` 驱动目标逻辑，再断言预期结果。

### Test: test_video_loader_type_doesnt_exist (lines 52-54)
```python
def test_video_loader_type_doesnt_exist():
    with pytest.raises(AssertionError):
        VIDEO_LOADER_REGISTRY.load("non_existing_video_loader")
```
**EN:** Checks Video Loader Type Doesnt Exist under a focused test scenario. The body exercises logic via `pytest.raises`, `VIDEO_LOADER_REGISTRY.load` before asserting the expected outcome.
**CN:** 该测试用例验证 Video Loader Type Doesnt Exist 在特定场景下的行为。 函数体会先通过 `pytest.raises`, `VIDEO_LOADER_REGISTRY.load` 驱动目标逻辑，再断言预期结果。

### Test: test_video_backend_handles_broken_frames (lines 57-92)
```python
def test_video_backend_handles_broken_frames(monkeypatch: pytest.MonkeyPatch):
    """
    Regression test for handling videos with broken frames.
    This test uses a pre-corrupted video file (assets/corrupted.mp4) that
    contains broken frames to verify the video loader handles
    them gracefully without crashing and returns accurate metadata.
    """
    with monkeypatch.context() as m:
        m.setenv("VLLM_VIDEO_LOADER_BACKEND", "opencv")

        # Load the pre-corrupted video file that contains broken frames
        corrupted_video_path = ASSETS_DIR / "corrupted.mp4"

        with open(corrupted_video_path, "rb") as f:
            video_data = f.read()

        loader = VIDEO_LOADER_REGISTRY.load("opencv")
        frames, metadata = loader.load_bytes(
            video_data, num_frames=-1, backend="opencv"
# ... omitted for brevity ...
        # frames_indices must match actual loaded frames
        assert frames.shape[0] == len(metadata["frames_indices"]), (
            f"Frames array size must equal frames_indices length. "
            f"Got {frames.shape[0]} frames but "
            f"{len(metadata['frames_indices'])} indices"
        )

        # Verify that broken frames were skipped:
        # loaded frames should be less than total
        assert frames.shape[0] < metadata["total_num_frames"], (
            f"Should load fewer frames than total due to broken frames. "
            f"Expected fewer than {metadata['total_num_frames']} frames, "
            f"but loaded {frames.shape[0]} frames"
        )
```
**EN:** Regression test for handling videos with broken frames. The body exercises logic via `monkeypatch.context`, `m.setenv`, `VIDEO_LOADER_REGISTRY.load` before asserting the expected outcome.
**CN:** 该测试用例验证 Video Backend Handles Broken Frames 在特定场景下的行为。 函数体会先通过 `monkeypatch.context`, `m.setenv`, `VIDEO_LOADER_REGISTRY.load` 驱动目标逻辑，再断言预期结果。

### Test: test_video_recovery_simulated_failures (lines 100-185)
```python
def test_video_recovery_simulated_failures(monkeypatch: pytest.MonkeyPatch):
    """
    Test that frame recovery correctly uses the next valid frame when
    target frames fail to load.

    Uses corrupted.mp4 and mocks VideoCapture.grab() to fail on specific
    frame indices (in addition to the real corruption at frame 17), then
    verifies recovery produces more frames.
    """
    import cv2

    with monkeypatch.context() as m:
        m.setenv("VLLM_VIDEO_LOADER_BACKEND", "opencv")

        # Load corrupted.mp4 (26 frames, frame 17 is genuinely corrupted)
        video_path = ASSETS_DIR / "corrupted.mp4"
        with open(video_path, "rb") as f:
            video_data = f.read()

# ... omitted for brevity ...
        # Without: 5 frames (3, 10, 17 all fail)
        # With: 8 frames (all recovered)
        assert frames_with_recovery.shape[0] > frames_no_recovery.shape[0], (
            f"Recovery should produce more frames. "
            f"Without: {frames_no_recovery.shape[0]}, "
            f"With: {frames_with_recovery.shape[0]}"
        )

        # Verify metadata consistency
        assert frames_no_recovery.shape[0] == len(meta_no["frames_indices"])
        assert frames_with_recovery.shape[0] == len(meta_yes["frames_indices"])

        # Verify temporal order is preserved
        assert meta_yes["frames_indices"] == sorted(meta_yes["frames_indices"])
```
**EN:** Test that frame recovery correctly uses the next valid frame when target frames fail to load. The body exercises logic via `monkeypatch.context`, `m.setenv`, `m.setattr` before asserting the expected outcome.
**CN:** 该测试用例验证 Video Recovery Simulated Failures 在特定场景下的行为。 函数体会先通过 `monkeypatch.context`, `m.setenv`, `m.setattr` 驱动目标逻辑，再断言预期结果。

### Fixture: dummy_video_path (lines 308-316)
```python
@pytest.fixture
def dummy_video_path(tmp_path):
    image_path = get_vllm_public_assets(
        filename="stop_sign.jpg", s3_prefix="vision_model_images"
    )

    video_path = tmp_path / "test_RGB_video.mp4"
    create_video_from_image(str(image_path), str(video_path), num_frames=1800, fps=30)
    return video_path
```
**EN:** Provides a pytest fixture for Dummy Video Path. It prepares shared state or helper objects for downstream scenarios. The fixture mainly builds or returns values through `get_vllm_public_assets`, `create_video_from_image`, `str`.
**CN:** 该代码块定义 pytest 夹具 `dummy_video_path`，用于为后续场景准备共享状态或辅助对象。 它主要通过 `get_vllm_public_assets`, `create_video_from_image`, `str` 构造或返回测试所需的值。

### Additional scenarios (summary)
```python
Block
Constants / assignments
Constants / assignments
test_video_recovery_with_corrupted_file
test_video_recovery_dynamic_backend
test_pyav_backend_loads_frames
test_pyav_dynamic_backend_loads_frames
test_pyav_backend_returns_target_frames_not_keyframes
test_video_loader_frames_sampling
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Multimodal inputs / 多模态输入**
  - **EN:** The tests combine text with image, audio, or video data and verify the modality-specific pipeline.
  - **CN:** 这些测试把文本与图像、音频或视频数据结合起来，验证模态专用处理流程。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `pathlib`
- **Third-party / 第三方依赖**: `numpy`, `numpy.typing`, `pytest`, `cv2`
- **vLLM internal / vLLM 内部依赖**: `vllm.assets.base`, `vllm.multimodal.video`
- **Local test utilities / 本地测试辅助**: `.utils`
