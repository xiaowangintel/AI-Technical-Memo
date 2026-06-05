# video.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/video.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines multimodal parsing, data structures, or helpers used by the input pipeline. / 定义输入流水线使用的多模态解析逻辑、数据结构或辅助函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-28)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import math
from abc import abstractmethod
from io import BytesIO
from typing import Any, ClassVar, Literal, NamedTuple, cast

import numpy as np
import numpy.typing as npt

from vllm.logger import init_logger
from vllm.utils.import_utils import PlaceholderModule
from vllm.utils.registry import ExtensionManager

try:
    import cv2
    import cv2.videoio_registry as vr
except ImportError:
    cv2 = PlaceholderModule("cv2")
    vr = PlaceholderModule("cv2").placeholder_attr("videoio_registry")

try:
    import av
except ImportError:
    av = PlaceholderModule("av")  # type: ignore[assignment]


logger = init_logger(__name__)
```
**EN:** Sets up the module with standard-library support such as `math`, `abc`, `io`, external packages such as `numpy`, `numpy.typing`, `cv2`, vLLM modules such as `vllm.logger`, `vllm.utils.import_utils`, `vllm.utils.registry`. It prepares the symbols later used by `VideoTargetMetadata`, `VideoSourceMetadata`, `resize_video`, `rescale_video_size`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.logger`, `vllm.utils.import_utils`, `vllm.utils.registry` 等 vLLM 内部依赖。 这些准备工作为后续的 `VideoTargetMetadata`, `VideoSourceMetadata`, `resize_video`, `rescale_video_size` 提供上下文。

### resize_video (lines 31-41)
```python
def resize_video(frames: npt.NDArray, size: tuple[int, int]) -> npt.NDArray:
    num_frames, _, _, channels = frames.shape
    new_height, new_width = size
    resized_frames = np.empty(
        (num_frames, new_height, new_width, channels), dtype=frames.dtype
    )

    for i, frame in enumerate(frames):
        resized_frame = cv2.resize(frame, (new_width, new_height))
        resized_frames[i] = resized_frame
    return resized_frames
```
**EN:** `resize_video` implements helper logic used by this module. It mainly works with `frames`, `size`. Inside the body, it relies on `np.empty`, `cv2.resize` to complete the main steps.
**CN:** `resize_video` 负责实现本模块使用的辅助逻辑。 它主要处理 `frames`, `size` 等参数。 实现过程中会调用 `np.empty`, `cv2.resize` 等函数完成关键步骤。

### rescale_video_size (lines 44-49)
```python
def rescale_video_size(frames: npt.NDArray, size_factor: float) -> npt.NDArray:
    _, height, width, _ = frames.shape
    new_height = int(height * size_factor)
    new_width = int(width * size_factor)

    return resize_video(frames, (new_height, new_width))
```
**EN:** `rescale_video_size` implements helper logic used by this module. It mainly works with `frames`, `size_factor`. Inside the body, it relies on `resize_video` to complete the main steps.
**CN:** `rescale_video_size` 负责实现本模块使用的辅助逻辑。 它主要处理 `frames`, `size_factor` 等参数。 实现过程中会调用 `resize_video` 等函数完成关键步骤。

### sample_frames_from_video (lines 52-59)
```python
def sample_frames_from_video(frames: npt.NDArray, num_frames: int) -> npt.NDArray:
    total_frames = frames.shape[0]
    if num_frames == -1:
        return frames

    frame_indices = np.linspace(0, total_frames - 1, num_frames, dtype=int)
    sampled_frames = frames[frame_indices, ...]
    return sampled_frames
```
**EN:** `sample_frames_from_video` implements helper logic used by this module. It mainly works with `frames`, `num_frames`. Inside the body, it relies on `np.linspace` to complete the main steps.
**CN:** `sample_frames_from_video` 负责实现本模块使用的辅助逻辑。 它主要处理 `frames`, `num_frames` 等参数。 实现过程中会调用 `np.linspace` 等函数完成关键步骤。

### VideoTargetMetadata overview (lines 62-67)
```python
class VideoTargetMetadata(NamedTuple):
    """Metadata represents target video."""

    num_frames: int
    fps: float
    max_duration: float
```
**EN:** `VideoTargetMetadata`: Metadata represents target video. It extends `NamedTuple`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `VideoTargetMetadata` 是该文件中的核心类，用于封装与 `VideoTargetMetadata` 相关的状态和行为。 它继承自 `NamedTuple`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### VideoSourceMetadata overview (lines 70-75)
```python
class VideoSourceMetadata(NamedTuple):
    """Metadata represents source video."""

    total_frames_num: int
    original_fps: float
    duration: float
```
**EN:** `VideoSourceMetadata`: Metadata represents source video. It extends `NamedTuple`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `VideoSourceMetadata` 是该文件中的核心类，用于封装与 `VideoSourceMetadata` 相关的状态和行为。 它继承自 `NamedTuple`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### VideoLoader overview (lines 78-113)
```python
class VideoLoader:
    @classmethod
    def compute_frames_index_to_sample(
        cls,
        source: VideoSourceMetadata,
        target: VideoTargetMetadata,
        **kwargs,
    ) -> list[int]:
        """Return the list of frame indices to sample from the video."""
        raise NotImplementedError

    @classmethod
    @abstractmethod
    def load_bytes(
        cls,
        data: bytes,
        **kwargs,
    ) -> tuple[npt.NDArray, dict[str, Any]]:
        """Load video frames from bytes and return (frames_array, metadata_dict)."""
        raise NotImplementedError

    @classmethod
    def create_hf_metadata(
        cls,
        source: VideoSourceMetadata,
        valid_frame_indices: list[int],
        video_backend: str,
    # ...
```
**EN:** Defines the `VideoLoader` class used by this module. Key methods include `compute_frames_index_to_sample`, `load_bytes`, `create_hf_metadata`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `VideoLoader` 是该文件中的核心类，用于封装与 `VideoLoader` 相关的状态和行为。 关键方法包括 `compute_frames_index_to_sample`, `load_bytes`, `create_hf_metadata`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### VideoLoader.load_bytes (lines 91-97)
```python
    def load_bytes(
        cls,
        data: bytes,
        **kwargs,
    ) -> tuple[npt.NDArray, dict[str, Any]]:
        """Load video frames from bytes and return (frames_array, metadata_dict)."""
        raise NotImplementedError
```
**EN:** `load_bytes`: Load video frames from bytes and return (frames_array, metadata_dict). It mainly works with `data`, `**kwargs`.
**CN:** `load_bytes` 负责加载下游使用的资源。 它主要处理 `data`, `**kwargs` 等参数。

### VideoLoader.create_hf_metadata (lines 100-113)
```python
    def create_hf_metadata(
        cls,
        source: VideoSourceMetadata,
        valid_frame_indices: list[int],
        video_backend: str,
    ):
        return {
            "total_num_frames": source.total_frames_num,
            "fps": source.original_fps,
            "duration": source.duration,
            "video_backend": video_backend,
            "frames_indices": valid_frame_indices,
            "do_sample_frames": len(valid_frame_indices) == source.total_frames_num,
        }
```
**EN:** `create_hf_metadata` constructs new objects or configuration bundles. It mainly works with `source`, `valid_frame_indices`, `video_backend`.
**CN:** `create_hf_metadata` 负责构造新的对象或配置集合。 它主要处理 `source`, `valid_frame_indices`, `video_backend` 等参数。

### VideoLoader.compute_frames_index_to_sample (lines 80-87)
```python
    def compute_frames_index_to_sample(
        cls,
        source: VideoSourceMetadata,
        target: VideoTargetMetadata,
        **kwargs,
    ) -> list[int]:
        """Return the list of frame indices to sample from the video."""
        raise NotImplementedError
```
**EN:** `compute_frames_index_to_sample`: Return the list of frame indices to sample from the video. It mainly works with `source`, `target`, `**kwargs`.
**CN:** `compute_frames_index_to_sample` 负责实现本模块使用的辅助逻辑。 它主要处理 `source`, `target`, `**kwargs` 等参数。

### OpenCVVideoBackendMixin overview (lines 119-360)
```python
class OpenCVVideoBackendMixin:
    @staticmethod
    def get_cv2_video_api():
        api_pref = None
        for backend in vr.getStreamBufferedBackends():
            if not vr.hasBackend(backend):
                continue
            if not vr.isBackendBuiltIn(backend):
                _, abi, api = vr.getStreamBufferedBackendPluginVersion(backend)
                if abi < 1 or (abi == 1 and api < 2):
                    continue
            api_pref = backend
            break
        return api_pref

    @classmethod
    def open_video_capture(cls, data: bytes) -> "cv2.VideoCapture":
        backend = cls.get_cv2_video_api()
        cap = cv2.VideoCapture(BytesIO(data), backend, [])
        if not cap.isOpened():
            raise ValueError("Could not open video stream")
        return cap

    @staticmethod
    def get_video_metadata(cap: "cv2.VideoCapture") -> VideoSourceMetadata:
        total_frames_num = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))
        original_fps = cap.get(cv2.CAP_PROP_FPS)
    # ...
```
**EN:** Defines the `OpenCVVideoBackendMixin` class used by this module. Key methods include `get_cv2_video_api`, `open_video_capture`, `get_video_metadata`, `read_frames`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `OpenCVVideoBackendMixin` 是该文件中的核心类，用于封装与 `OpenCVVideoBackendMixin` 相关的状态和行为。 关键方法包括 `get_cv2_video_api`, `open_video_capture`, `get_video_metadata`, `read_frames`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### OpenCVVideoBackendMixin._read_frames_with_recovery (lines 169-272)
```python
    def _read_frames_with_recovery(
        cls,
        cap: "cv2.VideoCapture",
        frame_indices: list[int],
        total_frames: int,
    ) -> tuple[npt.NDArray, list[int], dict[int, int]]:
        """
        Read frames with dynamic window forward-scan recovery.

        When a target frame fails to load, the next successfully grabbed
        frame (before the next target frame) will be used to recover it.

        Args:
            cap: OpenCV VideoCapture object
            frame_indices: Sorted list of target frame indices to load
            total_frames: Total number of frames in the video

        Returns:
            Tuple of (frames_array, valid_frame_indices, recovered_map)
            - frames_array: Array of loaded frames
    # ...
        if frames_list:
            frames = np.stack(frames_list)
        else:
            frames = np.empty((0, height, width, 3), dtype=np.uint8)

        return frames, valid_frame_indices, recovered_map
```
**EN:** `_read_frames_with_recovery`: Read frames with dynamic window forward-scan recovery. It mainly works with `cap`, `frame_indices`, `total_frames`. Inside the body, it relies on `cap.get`, `cap.grab`, `cls._can_use_for_recovery` to complete the main steps.
**CN:** `_read_frames_with_recovery` 负责从输入源读取并规范化数据。 它主要处理 `cap`, `frame_indices`, `total_frames` 等参数。 实现过程中会调用 `cap.get`, `cap.grab`, `cls._can_use_for_recovery` 等函数完成关键步骤。

### Additional repeated patterns / 其他重复模式
```python
# Large file omitted for brevity
# ...
```
**EN:** The remaining code mostly expands the same abstractions with more wrappers, constants, or schema variants. The analysis above focuses on the control points that shape overall behavior.
**CN:** 剩余代码主要在相同抽象之上继续展开，补充更多包装函数、常量或模式变体。上面的分析聚焦于决定整体行为的关键控制点。

## Key Concepts / 关键概念
- **`VideoTargetMetadata`**: Core class that organizes module behavior. / **`VideoTargetMetadata`**：组织模块行为的核心类。
- **`VideoSourceMetadata`**: Core class that organizes module behavior. / **`VideoSourceMetadata`**：组织模块行为的核心类。
- **`VideoLoader`**: Core class that organizes module behavior. / **`VideoLoader`**：组织模块行为的核心类。
- **`resize_video`**: Key helper or entry point in this file. / **`resize_video`**：本文件中的关键辅助函数或入口。
- **`rescale_video_size`**: Key helper or entry point in this file. / **`rescale_video_size`**：本文件中的关键辅助函数或入口。
- **`sample_frames_from_video`**: Key helper or entry point in this file. / **`sample_frames_from_video`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: math, abc, io, typing
- **Third-party / 第三方**: numpy, numpy.typing, cv2, cv2.videoio_registry, av
- **Internal vLLM / vLLM 内部依赖**: vllm.logger, vllm.utils.import_utils, vllm.utils.registry
