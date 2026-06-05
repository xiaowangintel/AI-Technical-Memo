# video.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/media/video.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements media loading, validation, and conversion utilities for multimodal inputs. / 实现多模态输入所需的媒体加载、校验与转换工具。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-16)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
from functools import partial
from pathlib import Path
from typing import Any

import numpy as np
import numpy.typing as npt
import pybase64
from PIL import Image

from vllm import envs

from ..video import VIDEO_LOADER_REGISTRY
from .base import MediaIO
from .image import ImageMediaIO
```
**EN:** Sets up the module with standard-library support such as `functools`, `pathlib`, `typing`, external packages such as `numpy`, `numpy.typing`, `pybase64`, vLLM modules such as `vllm`, `..video`, `.base`. It prepares the symbols later used by `VideoMediaIO`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm`, `..video`, `.base` 等 vLLM 内部依赖。 这些准备工作为后续的 `VideoMediaIO` 提供上下文。

### VideoMediaIO overview (lines 19-165)
```python
class VideoMediaIO(MediaIO[tuple[npt.NDArray, dict[str, Any]]]):
    """Configuration values can be user-provided either by --media-io-kwargs or
    by the runtime API field "media_io_kwargs". Ensure proper validation and
    error handling.
    """

    @classmethod
    def merge_kwargs(
        cls,
        default_kwargs: dict[str, Any] | None,
        runtime_kwargs: dict[str, Any] | None,
    ) -> dict[str, Any]:
        merged = super().merge_kwargs(default_kwargs, runtime_kwargs)
        # fps and num_frames interact with each other, so if either is
        # overridden at request time, wipe the other from defaults to
        # avoid unintuitive cross-field interactions.
        if runtime_kwargs:
            if "num_frames" in runtime_kwargs and "fps" not in runtime_kwargs:
                merged.pop("fps", None)
            elif "fps" in runtime_kwargs and "num_frames" not in runtime_kwargs:
                merged.pop("num_frames", None)
        return merged

    def __init__(
        self,
        image_io: ImageMediaIO,
        num_frames: int = 32,
    # ...
```
**EN:** `VideoMediaIO`: Configuration values can be user-provided either by --media-io-kwargs or by the runtime API field "media_io_kwargs". It extends `MediaIO`. Key methods include `merge_kwargs`, `__init__`, `load_bytes`, `load_base64`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `VideoMediaIO` 是该文件中的核心类，用于封装与 `VideoMediaIO` 相关的状态和行为。 它继承自 `MediaIO`。 关键方法包括 `merge_kwargs`, `__init__`, `load_bytes`, `load_base64`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### VideoMediaIO.__init__ (lines 42-67)
```python
    def __init__(
        self,
        image_io: ImageMediaIO,
        num_frames: int = 32,
        **kwargs,
    ) -> None:
        super().__init__()

        self.image_io = image_io
        self.num_frames = num_frames
        # `kwargs` contains custom arguments from
        # --media-io-kwargs for this modality, merged with
        # per-request runtime media_io_kwargs via merge_kwargs().
        # They can be passed to the underlying
        # media loaders (e.g. custom implementations)
        # for flexible control.

        # Allow per-request override of video backend via kwargs.
        # This enables users to specify a different backend than the
        # global VLLM_VIDEO_LOADER_BACKEND env var, e.g.:
        #   --media-io-kwargs '{"video": {"video_backend": "torchcodec"}}'
        video_loader_backend = (
            kwargs.pop("video_backend", None) or envs.VLLM_VIDEO_LOADER_BACKEND
        )
        self.kwargs = kwargs
        self.video_loader = VIDEO_LOADER_REGISTRY.load(video_loader_backend)
```
**EN:** `__init__` initializes state required by the module. It mainly works with `image_io`, `num_frames`, `**kwargs`. Inside the body, it relies on `super.__init__`, `VIDEO_LOADER_REGISTRY.load`, `kwargs.pop` to complete the main steps.
**CN:** `__init__` 负责初始化模块所需的状态。 它主要处理 `image_io`, `num_frames`, `**kwargs` 等参数。 实现过程中会调用 `super.__init__`, `VIDEO_LOADER_REGISTRY.load`, `kwargs.pop` 等函数完成关键步骤。

### VideoMediaIO.load_base64 (lines 74-140)
```python
    def load_base64(
        self, media_type: str, data: str
    ) -> tuple[npt.NDArray, dict[str, Any]]:
        if media_type.lower() == "video/jpeg":
            load_frame = partial(
                self.image_io.load_base64,
                "image/jpeg",
            )

            if self.num_frames > 0:
                frame_parts = data.split(",", self.num_frames)[: self.num_frames]
            elif self.num_frames == 0:
                raise ValueError("num_frames must be greater than 0 or -1")
            else:
                frame_parts = data.split(",")

            frames = np.stack(
                [np.asarray(load_frame(frame_data)) for frame_data in frame_parts]
            )
            total = int(frames.shape[0])
    # ...
                "frames_indices": frames_indices,
                "do_sample_frames": self.kwargs.get("do_sample_frames", False),
            }
            return frames, metadata

        return self.load_bytes(pybase64.b64decode(data))
```
**EN:** `load_base64` loads resources for downstream use. It mainly works with `media_type`, `data`. Inside the body, it relies on `self.load_bytes`, `media_type.lower`, `partial` to complete the main steps.
**CN:** `load_base64` 负责加载下游使用的资源。 它主要处理 `media_type`, `data` 等参数。 实现过程中会调用 `self.load_bytes`, `media_type.lower`, `partial` 等函数完成关键步骤。

### VideoMediaIO.encode_base64 (lines 148-165)
```python
    def encode_base64(
        self,
        media: npt.NDArray,
        *,
        video_format: str = "JPEG",
    ) -> str:
        video = media

        if video_format == "JPEG":
            encode_frame = partial(
                self.image_io.encode_base64,
                image_format=video_format,
            )

            return ",".join(encode_frame(Image.fromarray(frame)) for frame in video)

        msg = "Only JPEG format is supported for now."
        raise NotImplementedError(msg)
```
**EN:** `encode_base64` converts data into an encoded representation. It mainly works with `media`, `video_format`. Inside the body, it relies on `NotImplementedError`, `partial`, `join` to complete the main steps.
**CN:** `encode_base64` 负责把数据转换为编码后的表示。 它主要处理 `media`, `video_format` 等参数。 实现过程中会调用 `NotImplementedError`, `partial`, `join` 等函数完成关键步骤。

### VideoMediaIO.load_file (lines 142-146)
```python
    def load_file(self, filepath: Path) -> tuple[npt.NDArray, dict[str, Any]]:
        with filepath.open("rb") as f:
            data = f.read()

        return self.load_bytes(data)
```
**EN:** `load_file` loads resources for downstream use. It mainly works with `filepath`. Inside the body, it relies on `self.load_bytes`, `filepath.open`, `f.read` to complete the main steps.
**CN:** `load_file` 负责加载下游使用的资源。 它主要处理 `filepath` 等参数。 实现过程中会调用 `self.load_bytes`, `filepath.open`, `f.read` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`VideoMediaIO`**: Core class that organizes module behavior. / **`VideoMediaIO`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: functools, pathlib, typing
- **Third-party / 第三方**: numpy, numpy.typing, pybase64, PIL
- **Internal vLLM / vLLM 内部依赖**: vllm, ..video, .base, .image
