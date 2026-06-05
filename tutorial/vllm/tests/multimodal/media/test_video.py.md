# test_video.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/multimodal/media/test_video.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Video behavior in the Multimodal test area through focused pytest scenarios. It focuses on scenarios such as Assert10frames1fpsvideoloader, Video Media Io Kwargs, Opencv Video Io Colorspace. / 该文件在 Multimodal 测试域中，通过有针对性的 pytest 场景验证 Video 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-21)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import io
from pathlib import Path

import numpy as np
import numpy.typing as npt
import pybase64
import pytest
from PIL import Image

from vllm.assets.base import get_vllm_public_assets
from vllm.assets.video import video_to_ndarrays, video_to_pil_images_list
from vllm.multimodal.media import ImageMediaIO, VideoMediaIO
from vllm.multimodal.video import VIDEO_LOADER_REGISTRY, VideoLoader

from ..utils import cosine_similarity, create_video_from_image, normalize_image

pytestmark = pytest.mark.cpu_test

ASSETS_DIR = Path(__file__).parent.parent / "assets"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `io`, `pathlib`, `numpy`, `numpy.typing`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: Assert10Frames1FPSVideoLoader (lines 25-33)
```python
@VIDEO_LOADER_REGISTRY.register("assert_10_frames_1_fps")
class Assert10Frames1FPSVideoLoader(VideoLoader):
    @classmethod
    def load_bytes(
        cls, data: bytes, num_frames: int = -1, fps: float = -1.0, **kwargs
    ) -> npt.NDArray:
        assert num_frames == 10, "bad num_frames"
        assert fps == 1.0, "bad fps"
        return FAKE_OUTPUT_2
```
**EN:** Groups related scenarios for Assert10frames1fpsvideoloader.
**CN:** 该类把与 Assert10frames1fpsvideoloader 相关的场景组织在一起。

### Test: test_video_media_io_kwargs (lines 36-60)
```python
def test_video_media_io_kwargs(monkeypatch: pytest.MonkeyPatch):
    with monkeypatch.context() as m:
        m.setenv("VLLM_VIDEO_LOADER_BACKEND", "assert_10_frames_1_fps")
        imageio = ImageMediaIO()

        # Verify that different args pass/fail assertions as expected.
        videoio = VideoMediaIO(imageio, **{"num_frames": 10, "fps": 1.0})
        _ = videoio.load_bytes(b"test")

        videoio = VideoMediaIO(
            imageio, **{"num_frames": 10, "fps": 1.0, "not_used": "not_used"}
        )
        _ = videoio.load_bytes(b"test")

        with pytest.raises(AssertionError, match="bad num_frames"):
            videoio = VideoMediaIO(imageio, **{})
            _ = videoio.load_bytes(b"test")

        with pytest.raises(AssertionError, match="bad num_frames"):
            videoio = VideoMediaIO(imageio, **{"num_frames": 9, "fps": 1.0})
            _ = videoio.load_bytes(b"test")

        with pytest.raises(AssertionError, match="bad fps"):
            videoio = VideoMediaIO(imageio, **{"num_frames": 10, "fps": 2.0})
            _ = videoio.load_bytes(b"test")
```
**EN:** Checks Video Media Io Kwargs under a focused test scenario. The body exercises logic via `monkeypatch.context`, `m.setenv`, `ImageMediaIO` before asserting the expected outcome.
**CN:** 该测试用例验证 Video Media Io Kwargs 在特定场景下的行为。 函数体会先通过 `monkeypatch.context`, `m.setenv`, `ImageMediaIO` 驱动目标逻辑，再断言预期结果。

### Test: test_opencv_video_io_colorspace (lines 63-112)
```python
@pytest.mark.parametrize("is_color", [True, False])
@pytest.mark.parametrize("fourcc, ext", [("mp4v", "mp4"), ("XVID", "avi")])
def test_opencv_video_io_colorspace(tmp_path, is_color: bool, fourcc: str, ext: str):
    """
    Test all functions that use OpenCV for video I/O return RGB format.
    Both RGB and grayscale videos are tested.
    """
    image_path = get_vllm_public_assets(
        filename="stop_sign.jpg", s3_prefix="vision_model_images"
    )
    image = Image.open(image_path)

    if not is_color:
        image_path = f"{tmp_path}/test_grayscale_image.png"
        image = image.convert("L")
        image.save(image_path)
        # Convert to gray RGB for comparison
        image = image.convert("RGB")
    video_path = f"{tmp_path}/test_RGB_video.{ext}"
# ... omitted for brevity ...
    for frame in pil_frames:
        sim = cosine_similarity(
            normalize_image(np.array(frame)), normalize_image(np.array(image))
        )
        assert np.sum(np.isnan(sim)) / sim.size < 0.001
        assert np.nanmean(sim) > 0.99

    io_frames, _ = VideoMediaIO(ImageMediaIO()).load_file(Path(video_path))
    for frame in io_frames:
        sim = cosine_similarity(
            normalize_image(np.array(frame)), normalize_image(np.array(image))
        )
        assert np.sum(np.isnan(sim)) / sim.size < 0.001
        assert np.nanmean(sim) > 0.99
```
**EN:** Test all functions that use OpenCV for video I/O return RGB format. Decorators expand the same assertion pattern across multiple inputs. The body exercises logic via `pytest.mark.parametrize`, `get_vllm_public_assets`, `Image.open` before asserting the expected outcome.
**CN:** 该测试用例验证 Opencv Video Io Colorspace 在特定场景下的行为。 装饰器会把同一断言模式扩展到多组输入。 函数体会先通过 `pytest.mark.parametrize`, `get_vllm_public_assets`, `Image.open` 驱动目标逻辑，再断言预期结果。

### Constants / assignments (lines 115-115)
```python
NUM_FRAMES = 10
```
**EN:** Defines shared constants or configuration objects like `NUM_FRAMES`, which are reused by later tests.
**CN:** 这里定义了共享常量或配置对象（如 `NUM_FRAMES`），供后续测试重复使用。

### Class: TestVideoBackendOverride1 (lines 120-128)
```python
@VIDEO_LOADER_REGISTRY.register("test_video_backend_override_1")
class TestVideoBackendOverride1(VideoLoader):
    """Test loader that returns FAKE_OUTPUT_1 to verify backend selection."""

    @classmethod
    def load_bytes(
        cls, data: bytes, num_frames: int = -1, **kwargs
    ) -> tuple[npt.NDArray, dict]:
        return FAKE_OUTPUT_1, {"video_backend": "test_video_backend_override_1"}
```
**EN:** Groups related scenarios for Testvideobackendoverride1.
**CN:** 该类把与 Testvideobackendoverride1 相关的场景组织在一起。

### Class: TestVideoBackendOverride2 (lines 131-139)
```python
@VIDEO_LOADER_REGISTRY.register("test_video_backend_override_2")
class TestVideoBackendOverride2(VideoLoader):
    """Test loader that returns FAKE_OUTPUT_2 to verify backend selection."""

    @classmethod
    def load_bytes(
        cls, data: bytes, num_frames: int = -1, **kwargs
    ) -> tuple[npt.NDArray, dict]:
        return FAKE_OUTPUT_2, {"video_backend": "test_video_backend_override_2"}
```
**EN:** Groups related scenarios for Testvideobackendoverride2.
**CN:** 该类把与 Testvideobackendoverride2 相关的场景组织在一起。

### Test: test_video_media_io_backend_kwarg_override (lines 142-170)
```python
def test_video_media_io_backend_kwarg_override(monkeypatch: pytest.MonkeyPatch):
    """
    Test that video_backend kwarg can override the VLLM_VIDEO_LOADER_BACKEND
    environment variable.

    This allows users to dynamically select a different video backend
    via --media-io-kwargs without changing the global env var, which is
    useful when plugins set a default backend but a specific request
    needs a different one.
    """
    with monkeypatch.context() as m:
        # Set the env var to one backend
        m.setenv("VLLM_VIDEO_LOADER_BACKEND", "test_video_backend_override_1")

        imageio = ImageMediaIO()

        # Without video_backend kwarg, should use env var backend
        videoio_default = VideoMediaIO(imageio, num_frames=10)
        frames_default, metadata_default = videoio_default.load_bytes(b"test")
        np.testing.assert_array_equal(frames_default, FAKE_OUTPUT_1)
        assert metadata_default["video_backend"] == "test_video_backend_override_1"

        # With video_backend kwarg, should override env var
        videoio_override = VideoMediaIO(
            imageio, num_frames=10, video_backend="test_video_backend_override_2"
        )
        frames_override, metadata_override = videoio_override.load_bytes(b"test")
        np.testing.assert_array_equal(frames_override, FAKE_OUTPUT_2)
        assert metadata_override["video_backend"] == "test_video_backend_override_2"
```
**EN:** Test that video_backend kwarg can override the VLLM_VIDEO_LOADER_BACKEND environment variable. The body exercises logic via `monkeypatch.context`, `m.setenv`, `ImageMediaIO` before asserting the expected outcome.
**CN:** 该测试用例验证 Video Media Io Backend Kwarg Override 在特定场景下的行为。 函数体会先通过 `monkeypatch.context`, `m.setenv`, `ImageMediaIO` 驱动目标逻辑，再断言预期结果。

### Helper: _make_jpeg_b64_frames (lines 242-250)
```python
def _make_jpeg_b64_frames(n: int, width: int = 8, height: int = 8) -> list[str]:
    """Return *n* tiny base64-encoded JPEG frames."""
    frames: list[str] = []
    for i in range(n):
        img = Image.new("RGB", (width, height), color=(i % 256, 0, 0))
        buf = io.BytesIO()
        img.save(buf, format="JPEG")
        frames.append(pybase64.b64encode(buf.getvalue()).decode("ascii"))
    return frames
```
**EN:** Return *n* tiny base64-encoded JPEG frames. It coordinates operations such as `range`, `Image.new`, `io.BytesIO`.
**CN:** 该辅助函数为 Make Jpeg B64 Frames 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `range`, `Image.new`, `io.BytesIO` 等操作。

### Additional scenarios (summary)
```python
Block
Constants / assignments
Constants / assignments
test_video_media_io_backend_kwarg_not_passed_to_loader
test_video_media_io_backend_env_var_fallback
test_load_base64_jpeg_returns_metadata
test_load_base64_jpeg_enforces_num_frames_limit
test_load_base64_jpeg_no_limit_when_num_frames_negative
test_load_base64_jpeg_raises_on_zero_num_frames
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
- **Standard library / 标准库**: `io`, `pathlib`
- **Third-party / 第三方依赖**: `numpy`, `numpy.typing`, `pybase64`, `pytest`, `PIL`
- **vLLM internal / vLLM 内部依赖**: `vllm.assets.base`, `vllm.assets.video`, `vllm.multimodal.media`, `vllm.multimodal.video`
- **Local test utilities / 本地测试辅助**: `..utils`
