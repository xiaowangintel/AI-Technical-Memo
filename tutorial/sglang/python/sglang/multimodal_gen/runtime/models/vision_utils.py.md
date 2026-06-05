# vision_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/vision_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for vision utils within the multimodal runtime. Key symbols include `pil_to_numpy`, `numpy_to_pt`, `normalize`. / 该模块实现多模态运行时中与 vision utils 相关的模型构件。 关键符号包括 `pil_to_numpy`, `numpy_to_pt`, `normalize`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-36: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0

import os
import tempfile
from collections.abc import Callable
from io import BytesIO
from urllib.parse import unquote, urlparse

import imageio
import numpy as np
import PIL.Image
import PIL.ImageOps
# ...
        "bicubic": PIL.Image.BICUBIC,
        "lanczos": PIL.Image.LANCZOS,
        "nearest": PIL.Image.NEAREST,
    }
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 39-56: Function `pil_to_numpy` / 函数 `pil_to_numpy`
```python
def pil_to_numpy(images: list[PIL.Image.Image] | PIL.Image.Image) -> np.ndarray:
    r"""
    Convert a PIL image or a list of PIL images to NumPy arrays.

    Args:
        images (`PIL.Image.Image` or `List[PIL.Image.Image]`):
            The PIL image or list of images to convert to NumPy format.

    Returns:
        `np.ndarray`:
            A NumPy array representation of the images.
    """
    if not isinstance(images, list):
        images = [images]
    images = [np.array(image).astype(np.float32) / 255.0 for image in images]
    images_arr: np.ndarray = np.stack(images, axis=0)

    return images_arr
```
**EN:** This function drives `pil_to_numpy` with inputs such as `images`. Convert a PIL image or a list of PIL images to NumPy arrays.
**CN:** 这个函数负责 `pil_to_numpy`，主要处理 `images` 等输入。 文档字符串说明：Convert a PIL image or a list of PIL images to NumPy arrays.

### Lines 59-75: Function `numpy_to_pt` / 函数 `numpy_to_pt`
```python
def numpy_to_pt(images: np.ndarray) -> torch.Tensor:
    r"""
    Convert a NumPy image to a PyTorch tensor.

    Args:
        images (`np.ndarray`):
            The NumPy image array to convert to PyTorch format.

    Returns:
        `torch.Tensor`:
            A PyTorch tensor representation of the images.
    """
    if images.ndim == 3:
        images = images[..., None]

    images = torch.from_numpy(images.transpose(0, 3, 1, 2))
    return images
```
**EN:** This function drives `numpy_to_pt` with inputs such as `images`. Convert a NumPy image to a PyTorch tensor.
**CN:** 这个函数负责 `numpy_to_pt`，主要处理 `images` 等输入。 文档字符串说明：Convert a NumPy image to a PyTorch tensor.

### Lines 78-90: Function `normalize` / 函数 `normalize`
```python
def normalize(images: np.ndarray | torch.Tensor) -> np.ndarray | torch.Tensor:
    r"""
    Normalize an image array to [-1,1].

    Args:
        images (`np.ndarray` or `torch.Tensor`):
            The image array to normalize.

    Returns:
        `np.ndarray` or `torch.Tensor`:
            The normalized image array.
    """
    return 2.0 * images - 1.0
```
**EN:** This function drives `normalize` with inputs such as `images`. Normalize an image array to [-1,1].
**CN:** 这个函数负责 `normalize`，主要处理 `images` 等输入。 文档字符串说明：Normalize an image array to [-1,1].

### Lines 94-128: Function `load_image` / 函数 `load_image`
```python
def load_image(
    image: str | bytes | PIL.Image.Image,
    convert_method: Callable[[PIL.Image.Image], PIL.Image.Image] | None = None,
) -> PIL.Image.Image:
    """
    Loads `image` to a PIL Image.

    Args:
        image (`str` or `PIL.Image.Image`):
            The image to convert to the PIL Image format.
        convert_method (Callable[[PIL.Image.Image], PIL.Image.Image], *optional*):
            A conversion method to apply to the image after loading it. When set to `None` the image will be converted
            "RGB".
    """
# ...
    else:
        image = image.convert("RGB")

    return image
```
**EN:** This function drives `load_image` with inputs such as `image`, `convert_method`. Loads `image` to a PIL Image.
**CN:** 这个函数负责 `load_image`，主要处理 `image`, `convert_method` 等输入。 文档字符串说明：Loads `image` to a PIL Image.

### Lines 132-207: Function `load_video` / 函数 `load_video`
```python
def load_video(
    video: str,
    convert_method: (
        Callable[[list[PIL.Image.Image]], list[PIL.Image.Image]] | None
    ) = None,
) -> list[PIL.Image.Image]:
    """
    Loads `video` to a list of PIL Image.
    Args:
        video (`str`):
            A URL or Path to a video to convert to a list of PIL Image format.
        convert_method (Callable[[List[PIL.Image.Image]], List[PIL.Image.Image]], *optional*):
            A conversion method to apply to the video after loading it. When set to `None` the images will be converted
            to "RGB".
# ...
    if convert_method is not None:
        pil_images = convert_method(pil_images)

    return pil_images
```
**EN:** This function drives `load_video` with inputs such as `video`, `convert_method`. Loads `video` to a list of PIL Image.
**CN:** 这个函数负责 `load_video`，主要处理 `video`, `convert_method` 等输入。 文档字符串说明：Loads `video` to a list of PIL Image.

### Lines 210-255: Function `get_default_height_width` / 函数 `get_default_height_width`
```python
def get_default_height_width(
    image: PIL.Image.Image | np.ndarray | torch.Tensor,
    vae_scale_factor: int,
    height: int | None = None,
    width: int | None = None,
) -> tuple[int, int]:
    r"""
    Returns the height and width of the image, downscaled to the next integer multiple of `vae_scale_factor`.

    Args:
        image (`Union[PIL.Image.Image, np.ndarray, torch.Tensor]`):
            The image input, which can be a PIL image, NumPy array, or PyTorch tensor. If it is a NumPy array, it
            should have shape `[batch, height, width]` or `[batch, height, width, channels]`. If it is a PyTorch
            tensor, it should have shape `[batch, channels, height, width]`.
# ...
        x - x % vae_scale_factor for x in (width, height)
    )  # resize to integer multiple of vae_scale_factor

    return height, width
```
**EN:** This function drives `get_default_height_width` with inputs such as `image`, `vae_scale_factor`, `height`, `width`. Returns the height and width of the image, downscaled to the next integer multiple of `vae_scale_factor`.
**CN:** 这个函数负责 `get_default_height_width`，主要处理 `image`, `vae_scale_factor`, `height`, `width` 等输入。 文档字符串说明：Returns the height and width of the image, downscaled to the next integer multiple of `vae_scale_factor`.

### Lines 258-297: Function `resize` / 函数 `resize`
```python
def resize(
    image: PIL.Image.Image | np.ndarray | torch.Tensor,
    height: int,
    width: int,
    resize_mode: str = "default",  # "default", "fill", "crop"
    resample: str = "lanczos",
) -> PIL.Image.Image | np.ndarray | torch.Tensor:
    """
    Resize image.

    Args:
        image (`PIL.Image.Image`, `np.ndarray` or `torch.Tensor`):
            The image input, can be a PIL image, numpy array or pytorch tensor.
        height (`int`):
# ...
        image = image.resize((width, height), resample=PIL_INTERPOLATION[resample])
    else:
        raise ValueError(f"resize_mode {resize_mode} is not supported")
    return image
```
**EN:** This function drives `resize` with inputs such as `image`, `height`, `width`, `resize_mode`. Resize image.
**CN:** 这个函数负责 `resize`，主要处理 `image`, `height`, `width`, `resize_mode` 等输入。 文档字符串说明：Resize image.

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Sampling parameter control / 采样参数控制
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程
- Symbol `pil_to_numpy` anchors the module API / 符号 `pil_to_numpy` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.srt.utils.common`
- **External / 外部**: `urllib.parse`, `imageio`, `numpy`, `PIL.Image`, `PIL.ImageOps`, `requests`, `torch`, `packaging`
- **Stdlib / 标准库**: `os`, `tempfile`, `collections.abc`, `io`
