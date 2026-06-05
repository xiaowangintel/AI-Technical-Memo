# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/multimodal/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Utils behavior in the Multimodal test area through focused pytest scenarios. It focuses on scenarios such as Random Image, Random Video, Random Audio. / 该文件在 Multimodal 测试域中，通过有针对性的 pytest 场景验证 Utils 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-7)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import cv2
import numpy as np
import numpy.typing as npt
from PIL import Image
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `io`, `cv2`, `numpy`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: random_image (lines 10-13)
```python
def random_image(rng: np.random.RandomState, min_wh: int, max_wh: int):
    w, h = rng.randint(min_wh, max_wh, size=(2,))
    arr = rng.randint(0, 255, size=(w, h, 3), dtype=np.uint8)
    return Image.fromarray(arr)
```
**EN:** Implements a reusable helper for Random Image, reducing duplication across related tests. It coordinates operations such as `rng.randint`, `Image.fromarray`.
**CN:** 该辅助函数为 Random Image 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `rng.randint`, `Image.fromarray` 等操作。

### Helper: random_video (lines 16-25)
```python
def random_video(
    rng: np.random.RandomState,
    min_frames: int,
    max_frames: int,
    min_wh: int,
    max_wh: int,
):
    num_frames = rng.randint(min_frames, max_frames)
    w, h = rng.randint(min_wh, max_wh, size=(2,))
    return rng.randint(0, 255, size=(num_frames, w, h, 3), dtype=np.uint8)
```
**EN:** Implements a reusable helper for Random Video, reducing duplication across related tests. It coordinates operations such as `rng.randint`.
**CN:** 该辅助函数为 Random Video 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `rng.randint` 等操作。

### Helper: random_audio (lines 28-35)
```python
def random_audio(
    rng: np.random.RandomState,
    min_len: int,
    max_len: int,
    sr: int,
):
    audio_len = rng.randint(min_len, max_len)
    return rng.rand(audio_len), sr
```
**EN:** Implements a reusable helper for Random Audio, reducing duplication across related tests. It coordinates operations such as `rng.randint`, `rng.rand`.
**CN:** 该辅助函数为 Random Audio 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `rng.randint`, `rng.rand` 等操作。

### Helper: create_video_from_image (lines 38-66)
```python
def create_video_from_image(
    image_path: str,
    video_path: str,
    num_frames: int = 10,
    fps: float = 1.0,
    is_color: bool = True,
    fourcc: str = "mp4v",
):
    image = cv2.imread(image_path)
    if not is_color:
        # Convert to grayscale if is_color is False
        image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
        height, width = image.shape
    else:
        height, width, _ = image.shape

    video_writer = cv2.VideoWriter(
        video_path,
        cv2.VideoWriter_fourcc(*fourcc),
        fps,
        (width, height),
        isColor=is_color,
    )

    for _ in range(num_frames):
        video_writer.write(image)

    video_writer.release()
    return video_path
```
**EN:** Implements a reusable helper for Create Video From Image, reducing duplication across related tests. It coordinates operations such as `cv2.imread`, `cv2.VideoWriter`, `range`.
**CN:** 该辅助函数为 Create Video From Image 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `cv2.imread`, `cv2.VideoWriter`, `range` 等操作。

### Helper: create_long_gop_video (lines 69-103)
```python
def create_long_gop_video(
    num_frames: int = 50,
    fps: int = 30,
    width: int = 64,
    height: int = 64,
) -> bytes:
    """Encode an H.264 clip with one keyframe and green-channel = frame index.

    The marker lets a test recover which frame the decoder actually returned,
    independent of any metadata label.
    """
    import io

    import av

    buf = io.BytesIO()
    with av.open(buf, mode="w", format="mp4") as container:
        stream = container.add_stream("h264", rate=fps)
        stream.width = width
# ... omitted for brevity ...
        stream.codec_context.gop_size = num_frames
        stream.codec_context.max_b_frames = 0
        stream.codec_context.options = {
            "x264-params": (f"scenecut=0:keyint={num_frames}:min-keyint={num_frames}")
        }
        for i in range(num_frames):
            img = np.zeros((height, width, 3), dtype=np.uint8)
            img[:, :, 1] = i
            frame = av.VideoFrame.from_ndarray(img, format="rgb24")
            for packet in stream.encode(frame):
                container.mux(packet)
        for packet in stream.encode():
            container.mux(packet)
    return buf.getvalue()
```
**EN:** Encode an H.264 clip with one keyframe and green-channel = frame index. It coordinates operations such as `io.BytesIO`, `buf.getvalue`, `av.open`.
**CN:** 该辅助函数为 Create Long Gop Video 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `io.BytesIO`, `buf.getvalue`, `av.open` 等操作。

### Helper: cosine_similarity (lines 106-110)
```python
def cosine_similarity(A: npt.NDArray, B: npt.NDArray, axis: int = -1) -> npt.NDArray:
    """Compute cosine similarity between two vectors."""
    return np.sum(A * B, axis=axis) / (
        np.linalg.norm(A, axis=axis) * np.linalg.norm(B, axis=axis)
    )
```
**EN:** Compute cosine similarity between two vectors. It coordinates operations such as `np.sum`, `np.linalg.norm`.
**CN:** 该辅助函数为 Cosine Similarity 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `np.sum`, `np.linalg.norm` 等操作。

### Helper: normalize_image (lines 113-115)
```python
def normalize_image(image: npt.NDArray) -> npt.NDArray:
    """Normalize image to [0, 1] range."""
    return image.astype(np.float32) / 255.0
```
**EN:** Normalize image to [0, 1] range. It coordinates operations such as `image.astype`.
**CN:** 该辅助函数为 Normalize Image 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `image.astype` 等操作。

## Key Concepts / 关键概念
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Multimodal inputs / 多模态输入**
  - **EN:** The tests combine text with image, audio, or video data and verify the modality-specific pipeline.
  - **CN:** 这些测试把文本与图像、音频或视频数据结合起来，验证模态专用处理流程。
- **Quantization / 量化**
  - **EN:** The file covers low-precision model paths, kernels, or configuration rules for quantized inference.
  - **CN:** 该文件覆盖低精度模型路径、相关内核或量化推理配置规则。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `io`
- **Third-party / 第三方依赖**: `cv2`, `numpy`, `numpy.typing`, `PIL`, `av`
