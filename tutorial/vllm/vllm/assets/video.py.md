# video.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/assets/video.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements video support for the `assets` portion of vLLM. / 为 vLLM 的 `assets` 子目录实现与 视频 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-15)
```python
from dataclasses import dataclass

from functools import lru_cache

from typing import Any, ClassVar, Literal

import numpy as np

import numpy.typing as npt

from huggingface_hub import hf_hub_download

from PIL import Image

from vllm.multimodal.media.audio import load_audio_pyav

from .base import get_cache_dir
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 103-103)
```python
VideoAssetName = Literal["baby_reading"]
```
**EN:** This constant/configuration block defines `VideoAssetName`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `VideoAssetName`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Function `download_video_asset` (lines 19-36)
```python
def download_video_asset(filename: str) -> str:
    """
    Download and open an image from huggingface
    repo: raushan-testing-hf/videos-test
    """
    video_directory = get_cache_dir() / "video-example-data"
    video_directory.mkdir(parents=True, exist_ok=True)

    video_path = video_directory / filename
    video_path_str = str(video_path)
    if not video_path.exists():
        video_path_str = hf_hub_download(
            repo_id="raushan-testing-hf/videos-test",
            filename=filename,
            repo_type="dataset",
            cache_dir=video_directory,
        )
    return video_path_str
```
**EN:** Function `download_video_asset` handles loading or retrieval of external/internal data. The docstring highlights: Download and open an image from huggingface repo: raushan-testing-hf/videos-test Key calls such as `get_cache_dir`, `video_directory.mkdir`, `str`, `video_path.exists`, `hf_hub_download` show the concrete execution path.
**CN:** Function `download_video_asset` 负责加载或获取外部/内部数据。 文档字符串强调：Download and open an image from huggingface repo: raushan-testing-hf/videos-test 像 `get_cache_dir`, `video_directory.mkdir`, `str`, `video_path.exists`, `hf_hub_download` 这样的关键调用展示了该代码块的具体执行路径。

### Function `video_to_ndarrays` (lines 39-68)
```python
def video_to_ndarrays(path: str, num_frames: int = -1) -> npt.NDArray:
    import cv2

    cap = cv2.VideoCapture(path)
    if not cap.isOpened():
        raise ValueError(f"Could not open video file {path}")

    total_frames = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))
    frames = []

    num_frames = num_frames if num_frames > 0 else total_frames
    frame_indices = np.linspace(0, total_frames - 1, num_frames, dtype=int)
    for idx in range(total_frames):
        ok = cap.grab()  # next img
        if not ok:
            break
        if idx in frame_indices:  # only decompress needed
            ret, frame = cap.retrieve()
            if ret:
                # OpenCV uses BGR format, we need to convert it to RGB
                # for PIL and transformers compatibility
                frames.append(cv2.cvtColor(frame, cv2.COLOR_BGR2RGB))

    frames = np.stack(frames)
    if len(frames) < num_frames:
    # ... omitted for brevity ...
        )
    return frames
```
**EN:** Function `video_to_ndarrays` works with modality-specific preprocessing or transport logic. Key calls such as `cv2.VideoCapture`, `cap.isOpened`, `ValueError`, `int`, `cap.get` show the concrete execution path.
**CN:** Function `video_to_ndarrays` 处理模态相关的预处理或传输逻辑。 像 `cv2.VideoCapture`, `cap.isOpened`, `ValueError`, `int`, `cap.get` 这样的关键调用展示了该代码块的具体执行路径。

### Function `video_to_pil_images_list` (lines 71-73)
```python
def video_to_pil_images_list(path: str, num_frames: int = -1) -> list[Image.Image]:
    frames = video_to_ndarrays(path, num_frames)
    return [Image.fromarray(frame) for frame in frames]
```
**EN:** Function `video_to_pil_images_list` works with modality-specific preprocessing or transport logic. Key calls such as `video_to_ndarrays`, `Image.fromarray` show the concrete execution path.
**CN:** Function `video_to_pil_images_list` 处理模态相关的预处理或传输逻辑。 像 `video_to_ndarrays`, `Image.fromarray` 这样的关键调用展示了该代码块的具体执行路径。

### Function `video_get_metadata` (lines 76-100)
```python
def video_get_metadata(path: str, num_frames: int = -1) -> dict[str, Any]:
    import cv2

    cap = cv2.VideoCapture(path)
    if not cap.isOpened():
        raise ValueError(f"Could not open video file {path}")

    total_frames = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))
    fps = cap.get(cv2.CAP_PROP_FPS)
    duration = total_frames / fps if fps > 0 else 0

    if num_frames == -1 or num_frames > total_frames:
        num_frames = total_frames

    metadata = {
        "total_num_frames": num_frames,
        "fps": duration / num_frames,
        "duration": duration,
        "video_backend": "opencv",
        "frames_indices": list(range(num_frames)),
        # extra field used to control hf processor's video
        # sampling behavior
        "do_sample_frames": num_frames == total_frames,
    }
    return metadata
```
**EN:** Function `video_get_metadata` works with modality-specific preprocessing or transport logic. Key calls such as `cv2.VideoCapture`, `cap.isOpened`, `ValueError`, `int`, `cap.get` show the concrete execution path.
**CN:** Function `video_get_metadata` 处理模态相关的预处理或传输逻辑。 像 `cv2.VideoCapture`, `cap.isOpened`, `ValueError`, `int`, `cap.get` 这样的关键调用展示了该代码块的具体执行路径。

### Class `VideoAsset` (lines 107-144)
```python
class VideoAsset:
    name: VideoAssetName
    num_frames: int = -1

    _NAME_TO_FILE: ClassVar[dict[VideoAssetName, str]] = {
        "baby_reading": "sample_demo_1.mp4",
    }

    @property
    def filename(self) -> str:
        return self._NAME_TO_FILE[self.name]

    @property
    def video_path(self) -> str:
        return download_video_asset(self.filename)

    @property
    def pil_images(self) -> list[Image.Image]:
        ret = video_to_pil_images_list(self.video_path, self.num_frames)
        return ret

    @property
    def np_ndarrays(self) -> npt.NDArray:
    # ... omitted for brevity ...
        """
        return load_audio_pyav(self.video_path, sr=sampling_rate)[0]
```
**EN:** Class `VideoAsset` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `filename`, `video_path`, `pil_images`, `np_ndarrays`, `metadata`, `get_audio`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `VideoAsset` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `filename`, `video_path`, `pil_images`, `np_ndarrays`, `metadata`, `get_audio`，它们共同定义初始化、校验、变换或访问模式。

### Method `VideoAsset.filename` (lines 116-117)
```python
    def filename(self) -> str:
        return self._NAME_TO_FILE[self.name]
```
**EN:** Method `VideoAsset.filename` exposes a computed property so callers can access derived state without duplicating logic.
**CN:** Method `VideoAsset.filename` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。

### Method `VideoAsset.video_path` (lines 120-121)
```python
    def video_path(self) -> str:
        return download_video_asset(self.filename)
```
**EN:** Method `VideoAsset.video_path` exposes a computed property so callers can access derived state without duplicating logic. Key calls such as `download_video_asset` show the concrete execution path.
**CN:** Method `VideoAsset.video_path` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。 像 `download_video_asset` 这样的关键调用展示了该代码块的具体执行路径。

### Method `VideoAsset.pil_images` (lines 124-126)
```python
    def pil_images(self) -> list[Image.Image]:
        ret = video_to_pil_images_list(self.video_path, self.num_frames)
        return ret
```
**EN:** Method `VideoAsset.pil_images` exposes a computed property so callers can access derived state without duplicating logic. Key calls such as `video_to_pil_images_list` show the concrete execution path.
**CN:** Method `VideoAsset.pil_images` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。 像 `video_to_pil_images_list` 这样的关键调用展示了该代码块的具体执行路径。

### Method `VideoAsset.np_ndarrays` (lines 129-131)
```python
    def np_ndarrays(self) -> npt.NDArray:
        ret = video_to_ndarrays(self.video_path, self.num_frames)
        return ret
```
**EN:** Method `VideoAsset.np_ndarrays` exposes a computed property so callers can access derived state without duplicating logic. Key calls such as `video_to_ndarrays` show the concrete execution path.
**CN:** Method `VideoAsset.np_ndarrays` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。 像 `video_to_ndarrays` 这样的关键调用展示了该代码块的具体执行路径。

### Method `VideoAsset.metadata` (lines 134-136)
```python
    def metadata(self) -> dict[str, Any]:
        ret = video_get_metadata(self.video_path, self.num_frames)
        return ret
```
**EN:** Method `VideoAsset.metadata` exposes a computed property so callers can access derived state without duplicating logic. Key calls such as `video_get_metadata` show the concrete execution path.
**CN:** Method `VideoAsset.metadata` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。 像 `video_get_metadata` 这样的关键调用展示了该代码块的具体执行路径。

### Method `VideoAsset.get_audio` (lines 138-144)
```python
    def get_audio(self, sampling_rate: float | None = None) -> npt.NDArray:
        """
        Read audio data from the video asset, used in Qwen2.5-Omni examples.

        See also: examples/generate/multimodal/qwen2_5_omni/only_thinker.py
        """
        return load_audio_pyav(self.video_path, sr=sampling_rate)[0]
```
**EN:** Method `VideoAsset.get_audio` works with modality-specific preprocessing or transport logic. The docstring highlights: Read audio data from the video asset, used in Qwen2.5-Omni examples. Key calls such as `load_audio_pyav` show the concrete execution path.
**CN:** Method `VideoAsset.get_audio` 处理模态相关的预处理或传输逻辑。 文档字符串强调：Read audio data from the video asset, used in Qwen2.5-Omni examples. 像 `load_audio_pyav` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。
- **Structured types / 结构化类型**
  - **EN:** Dataclasses, typed dictionaries, or aliases are used to make runtime contracts explicit.
  - **CN:** 通过 dataclass、类型化字典或别名来显式表达运行时契约。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from dataclasses import dataclass`, `from functools import lru_cache`, `from typing import Any, ClassVar, Literal`
- **Third-party / 第三方**: `import numpy as np`, `import numpy.typing as npt`, `from huggingface_hub import hf_hub_download`, `from PIL import Image`
- **vLLM internal / vLLM 内部依赖**: `from vllm.multimodal.media.audio import load_audio_pyav`, `from .base import get_cache_dir`
