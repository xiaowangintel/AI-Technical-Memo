# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/apps/ComfyUI_SGLDiffusion/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the ComfyUI integration layer. It centers on `SGLDVideoInput`, `_ensure_dir`, and `_to_numpy_image`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于ComfyUI 集成层。它围绕 `SGLDVideoInput`、`_ensure_dir` 和 `_to_numpy_image` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: module setup and imports / 模块初始化与导入
```python
import base64
import io
import os
import shutil
import time
import uuid

import folder_paths
import numpy as np
import torch
from comfy_api.input import VideoInput
from PIL import Image
```
**EN:** This block establishes the module context and imports `base64`, `io`, `os`, `shutil`, `time`, and `uuid`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `base64`、`io`、`os`、`shutil`、`time` 和 `uuid`。这些依赖为后续实现提供所需符号。

### Lines 15-16: `_ensure_dir` implementation / `_ensure_dir` 实现
```python
def _ensure_dir(path: str) -> None:
    os.makedirs(path, exist_ok=True)
```
**EN:** This block defines function `_ensure_dir`. It handles ensure dir logic. Key calls include `os.makedirs`. Parameters such as `path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_ensure_dir`。 它用于处理 ensure dir 相关逻辑。 关键调用包括 `os.makedirs`。 本段逻辑主要由 `path` 等参数驱动。

### Lines 19-32: `_to_numpy_image` implementation / `_to_numpy_image` 实现
```python
def _to_numpy_image(image: torch.Tensor) -> np.ndarray:
    """Convert ComfyUI image tensor to uint8 numpy array (H, W, C)."""
    if image.dim() == 4:
        image = image[0]
    if image.dim() == 3 and image.shape[0] in (1, 3, 4):
        image = image.permute(1, 2, 0)
    elif image.dim() == 2:
        image = image.unsqueeze(-1)
    np_img = image.detach().cpu().numpy()
    np_img = np.clip(np_img, 0.0, 1.0)
    np_img = (np_img * 255).astype(np.uint8)
    if np_img.shape[-1] == 1:
        np_img = np.repeat(np_img, 3, axis=-1)
    return np_img
```
**EN:** This block defines function `_to_numpy_image`. Convert ComfyUI image tensor to uint8 numpy array (H, W, C). Key calls include `image.detach.cpu.numpy`, `np.clip`, `astype`, `image.dim`, and `image.permute`. The implementation branches on conditions. Parameters such as `image` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_to_numpy_image`。 它用于转换为numpy image。 关键调用包括 `image.detach.cpu.numpy`、`np.clip`、`astype`、`image.dim` 和 `image.permute`。 实现中包含条件分支。 本段逻辑主要由 `image` 等参数驱动。

### Lines 35-49: `_to_hwc_tensor` implementation / `_to_hwc_tensor` 实现
```python
def _to_hwc_tensor(image: torch.Tensor) -> torch.Tensor:
    """Convert ComfyUI image tensor to HWC format (normalized [0, 1])."""
    img = image.clone()
    if img.dim() == 4:
        img = img[0]
    if img.dim() == 3 and img.shape[0] in (1, 3, 4):
        img = img.permute(1, 2, 0)
    elif img.dim() == 2:
        img = img.unsqueeze(-1)

    img = torch.clamp(img, 0.0, 1.0)
    if img.shape[-1] == 1:
        img = img.repeat(1, 1, 3)

    return img
```
**EN:** This block defines function `_to_hwc_tensor`. Convert ComfyUI image tensor to HWC format (normalized [0, 1]). Key calls include `image.clone`, `torch.clamp`, `img.dim`, `img.permute`, and `img.repeat`. The implementation branches on conditions. Parameters such as `image` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_to_hwc_tensor`。 它用于转换为hwc tensor。 关键调用包括 `image.clone`、`torch.clamp`、`img.dim`、`img.permute` 和 `img.repeat`。 实现中包含条件分支。 本段逻辑主要由 `image` 等参数驱动。

### Lines 52-77: `is_empty_image` implementation / `is_empty_image` 实现
```python
def is_empty_image(image: torch.Tensor, tolerance: float = 1e-6) -> bool:
    """
    Check if the input image is an empty/solid color image (like ComfyUI's empty image).
    Args:
        image: Input tensor image in ComfyUI format (BCHW, CHW, HWC, etc.)
        tolerance: Tolerance for floating point comparison (default: 1e-6)

    Returns:
        True if the image is empty (all pixels have same color), False otherwise
    """
    if image is None:
        return True

    # Convert to HWC format
    img_hwc = _to_hwc_tensor(image)

    # Get the first pixel's RGB values
    first_pixel = img_hwc[0, 0, :]

    h, w, c = img_hwc.shape
    pixels = img_hwc.reshape(-1, c)

    diff = torch.abs(pixels - first_pixel)
    max_diff = torch.max(diff)

    return max_diff.item() <= tolerance
```
**EN:** This block defines function `is_empty_image`. Check if the input image is an empty/solid color image (like ComfyUI's empty image). Args: image: Input tensor image in ComfyUI format (BCHW, CHW, HWC, etc.) tolerance: Tolerance for floating point comparison (default: 1e-6) Returns: True if the image is empty (all pixels have same color), False otherwise Key calls include `_to_hwc_tensor`, `img_hwc.reshape`, `torch.abs`, `torch.max`, and `max_diff.item`. The implementation branches on conditions. Parameters such as `image`, and `tolerance` drive the behavior in this section.
**CN:** 该代码块定义了函数 `is_empty_image`。 它用于处理 is empty image 相关逻辑。 关键调用包括 `_to_hwc_tensor`、`img_hwc.reshape`、`torch.abs`、`torch.max` 和 `max_diff.item`。 实现中包含条件分支。 本段逻辑主要由 `image` 和 `tolerance` 等参数驱动。

### Lines 80-97: `get_image_path` implementation / `get_image_path` 实现
```python
def get_image_path(image: torch.Tensor) -> str:
    """
    Save tensor image to ComfyUI temp directory as PNG and return the path.
    """
    temp_dir = folder_paths.get_temp_directory()

    # Build file name
    ts = time.strftime("%Y%m%d-%H%M%S")
    unique = uuid.uuid4().hex[:8]
    file_name = f"sgl_output_{ts}_{unique}.png"
    file_path = os.path.join(temp_dir, file_name)

    # Save image
    np_img = _to_numpy_image(image)
    img = Image.fromarray(np_img)
    img.save(file_path, format="PNG")

    return file_path
```
**EN:** This block defines function `get_image_path`. Save tensor image to ComfyUI temp directory as PNG and return the path. Key calls include `folder_paths.get_temp_directory`, `time.strftime`, `os.path.join`, `_to_numpy_image`, and `Image.fromarray`. Parameters such as `image` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_image_path`。 它用于获取image path。 关键调用包括 `folder_paths.get_temp_directory`、`time.strftime`、`os.path.join`、`_to_numpy_image` 和 `Image.fromarray`。 本段逻辑主要由 `image` 等参数驱动。

### Lines 100-128: `convert_b64_to_tensor_image` implementation / `convert_b64_to_tensor_image` 实现
```python
def convert_b64_to_tensor_image(b64_image: str) -> torch.Tensor:
    """
    Convert base64 encoded image to ComfyUI IMAGE format (torch.Tensor).

    Args:
        b64_image: Base64 encoded image string

    Returns:
        torch.Tensor with shape [batch_size, height, width, channels] (BHWC format),
        values normalized to [0, 1] range, RGB format (3 channels)
    """
    # Decode base64
    image_bytes = base64.b64decode(b64_image)

    # Open image and convert to RGB
    pil_image = Image.open(io.BytesIO(image_bytes))
    if pil_image.mode != "RGB":
        pil_image = pil_image.convert("RGB")

    # Convert to numpy array and normalize to [0, 1]
    image_array = np.array(pil_image).astype(np.float32) / 255.0

    # Add batch dimension: [height, width, channels] -> [1, height, width, channels]
    image_array = image_array[np.newaxis, ...]

    # Convert to torch.Tensor
    tensor_image = torch.from_numpy(image_array)

    return tensor_image
```
**EN:** This block defines function `convert_b64_to_tensor_image`. Convert base64 encoded image to ComfyUI IMAGE format (torch.Tensor). Args: b64_image: Base64 encoded image string Returns: torch.Tensor with shape [batch_size, height, width, channels] (BHWC format), values normalized to [0, 1] range, RGB format (3 channels) Key calls include `base64.b64decode`, `Image.open`, `torch.from_numpy`, `io.BytesIO`, and `pil_image.convert`. The implementation branches on conditions. Parameters such as `b64_image` drive the behavior in this section.
**CN:** 该代码块定义了函数 `convert_b64_to_tensor_image`。 它用于转换b64 to tensor image。 关键调用包括 `base64.b64decode`、`Image.open`、`torch.from_numpy`、`io.BytesIO` 和 `pil_image.convert`。 实现中包含条件分支。 本段逻辑主要由 `b64_image` 等参数驱动。

### Lines 131-131: `SGLDVideoInput` class overview / `SGLDVideoInput` 类概览
```python
class SGLDVideoInput(VideoInput):
```
**EN:** This block defines class `SGLDVideoInput`. It encapsulates sgldvideo input behavior. It inherits from `VideoInput`.
**CN:** 该代码块定义了类 `SGLDVideoInput`。 它用于封装 sgldvideo input 相关行为。 它继承自 `VideoInput`。

### Lines 132-137: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, video_path: str, height: int, width: int):
        super().__init__()

        self.video_path = video_path
        self.height = height
        self.width = width
```
**EN:** This block defines method `__init__` on `SGLDVideoInput`. It initializes the instance state. Key calls include `super.__init__`, and `super`. Parameters such as `video_path`, `height`, and `width` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDVideoInput` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。 本段逻辑主要由 `video_path`、`height` 和 `width` 等参数驱动。

### Lines 139-146: `get_dimensions` implementation / `get_dimensions` 实现
```python
    def get_dimensions(self) -> tuple[int, int]:
        """
        Returns the dimensions of the video input.

        Returns:
            Tuple of (width, height)
        """
        return self.width, self.height
```
**EN:** This block defines method `get_dimensions` on `SGLDVideoInput`. Returns the dimensions of the video input. Returns: Tuple of (width, height)
**CN:** 该代码块定义了 `SGLDVideoInput` 的方法 `get_dimensions`。 它用于获取dimensions。

### Lines 148-153: `get_components` implementation / `get_components` 实现
```python
    def get_components(self):
        """
        Returns the components of the video input.
        This is required by the VideoInput abstract base class.
        """
        return [self.video_path]
```
**EN:** This block defines method `get_components` on `SGLDVideoInput`. Returns the components of the video input. This is required by the VideoInput abstract base class.
**CN:** 该代码块定义了 `SGLDVideoInput` 的方法 `get_components`。 它用于获取components。

### Lines 155-166: `save_to` implementation / `save_to` 实现
```python
    def save_to(self, path: str, format=None, codec=None, metadata=None):
        """
        Abstract method to save the video input to a file.
        """
        save_path = path
        # Copy video file from video_path to save_path
        if os.path.exists(self.video_path):
            # Ensure destination directory exists
            save_dir = os.path.dirname(save_path)
            if save_dir:
                os.makedirs(save_dir, exist_ok=True)
            shutil.copy2(self.video_path, save_path)
```
**EN:** This block defines method `save_to` on `SGLDVideoInput`. Abstract method to save the video input to a file. Key calls include `os.path.exists`, `os.path.dirname`, `shutil.copy2`, and `os.makedirs`. The implementation branches on conditions. Parameters such as `path`, `format`, `codec`, and `metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `SGLDVideoInput` 的方法 `save_to`。 它用于保存to。 关键调用包括 `os.path.exists`、`os.path.dirname`、`shutil.copy2` 和 `os.makedirs`。 实现中包含条件分支。 本段逻辑主要由 `path`、`format`、`codec` 和 `metadata` 等参数驱动。

### Lines 169-176: `convert_video_to_comfy_video` implementation / `convert_video_to_comfy_video` 实现
```python
def convert_video_to_comfy_video(
    video_path: str, height: int, width: int
) -> VideoInput:
    """
    Convert video to ComfyUI VIDEO format (VideoInput).
    """
    video_input = SGLDVideoInput(video_path, height, width)
    return video_input
```
**EN:** This block defines function `convert_video_to_comfy_video`. Convert video to ComfyUI VIDEO format (VideoInput). Key calls include `SGLDVideoInput`. Parameters such as `video_path`, `height`, and `width` drive the behavior in this section.
**CN:** 该代码块定义了函数 `convert_video_to_comfy_video`。 它用于转换video to comfy video。 关键调用包括 `SGLDVideoInput`。 本段逻辑主要由 `video_path`、`height` 和 `width` 等参数驱动。

## Key Concepts / 关键概念
- `_ensure_dir`: Top-level function that handles ensure dir logic. / 顶层函数，用于处理 ensure dir 相关逻辑。
- `_to_numpy_image`: Convert ComfyUI image tensor to uint8 numpy array (H, W, C). / 顶层函数，用于转换为numpy image。
- `_to_hwc_tensor`: Convert ComfyUI image tensor to HWC format (normalized [0, 1]). / 顶层函数，用于转换为hwc tensor。
- `is_empty_image`: Check if the input image is an empty/solid color image (like ComfyUI's empty image). / 顶层函数，用于处理 is empty image 相关逻辑。
- `get_image_path`: Save tensor image to ComfyUI temp directory as PNG and return the path. / 顶层函数，用于获取image path。
- `convert_b64_to_tensor_image`: Convert base64 encoded image to ComfyUI IMAGE format (torch.Tensor). / 顶层函数，用于转换b64 to tensor image。
- `SGLDVideoInput`: Primary class that encapsulates sgldvideo input behavior. / 核心类，用于封装 sgldvideo input 相关行为。
- `convert_video_to_comfy_video`: Convert video to ComfyUI VIDEO format (VideoInput). / 顶层函数，用于转换video to comfy video。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `base64`, `io`, `os`, `shutil`, `time`, `uuid`
- **Third-party / 第三方依赖**: `folder_paths`, `numpy`, `torch`, `comfy_api.input`, `PIL`

- **Total lines / 总行数**: 176
