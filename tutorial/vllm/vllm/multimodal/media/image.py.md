# image.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/media/image.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements media loading, validation, and conversion utilities for multimodal inputs. / 实现多模态输入所需的媒体加载、校验与转换工具。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-17)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from io import BytesIO
from pathlib import Path

import numpy as np
import pybase64
import torch
from PIL import Image

from vllm.utils.serial_utils import tensor2base64

from ..image import convert_image_mode, rgba_to_rgb
from .base import MediaIO, MediaWithBytes

MAGIC_NUMPY_PREFIX = b"\x93NUMPY"  # https://numpy.org/devdocs/reference/generated/numpy.lib.format.html#format-version-1-0
```
**EN:** Sets up the module with standard-library support such as `io`, `pathlib`, external packages such as `numpy`, `pybase64`, `torch`, vLLM modules such as `vllm.utils.serial_utils`, `..image`, `.base`. It prepares the symbols later used by `ImageMediaIO`, `ImageEmbeddingMediaIO`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.utils.serial_utils`, `..image`, `.base` 等 vLLM 内部依赖。 这些准备工作为后续的 `ImageMediaIO`, `ImageEmbeddingMediaIO` 提供上下文。

### ImageMediaIO (lines 20-98)
```python
class ImageMediaIO(MediaIO[Image.Image]):
    """Configuration values can be user-provided either by --media-io-kwargs or
    by the runtime API field "media_io_kwargs". Ensure proper validation and
    error handling.
    """

    def __init__(self, image_mode: str = "RGB", **kwargs) -> None:
        super().__init__()

        self.image_mode = image_mode
        # `kwargs` contains custom arguments from
        # --media-io-kwargs for this modality, merged with
        # per-request runtime media_io_kwargs via merge_kwargs().
        # They can be passed to the underlying
        # media loaders (e.g. custom implementations)
        # for flexible control.
        self.kwargs = kwargs

        # Extract RGBA background color from kwargs if provided
        # Default to white background for backward compatibility
        rgba_bg = kwargs.get("rgba_background_color", (255, 255, 255))
        # Convert list to tuple for consistency
        if isinstance(rgba_bg, list):
            rgba_bg = tuple(rgba_bg)

        # Validate rgba_background_color format
        if not (
            isinstance(rgba_bg, tuple)
    # ...
        with BytesIO() as buffer:
            image = self._convert_image_mode(image)
            image.save(buffer, image_format)
            data = buffer.getvalue()

        return pybase64.b64encode(data).decode("utf-8")
```
**EN:** `ImageMediaIO`: Configuration values can be user-provided either by --media-io-kwargs or by the runtime API field "media_io_kwargs". It extends `MediaIO`. Key methods include `__init__`, `load_bytes`, `load_base64`, `load_file`.
**CN:** `ImageMediaIO` 是该文件中的核心类，用于封装与 `ImageMediaIO` 相关的状态和行为。 它继承自 `MediaIO`。 关键方法包括 `__init__`, `load_bytes`, `load_base64`, `load_file`。

### ImageEmbeddingMediaIO (lines 101-142)
```python
class ImageEmbeddingMediaIO(MediaIO[torch.Tensor]):
    """Image embedding MediaIO implementation.

    Configuration values can be user-provided either by --media-io-kwargs or
    by the runtime API field "media_io_kwargs". Ensure proper validation and
    error handling.
    """

    def __init__(self) -> None:
        super().__init__()

    def _load_pickled_torch(self, data: bytes) -> torch.Tensor:
        buffer = BytesIO(data)
        # Enable sparse tensor integrity checks to prevent out-of-bounds
        # writes from maliciously crafted tensors
        with torch.sparse.check_sparse_tensor_invariants():
            tensor = torch.load(buffer, weights_only=True)
            return tensor.to_dense()

    def _load_numpy(self, data: bytes) -> torch.Tensor:
        with BytesIO(data) as buffer:
            return torch.from_numpy(np.load(buffer))

    def load_bytes(self, data: bytes) -> torch.Tensor:
        if data[:6] == MAGIC_NUMPY_PREFIX:
            return self._load_numpy(data)

        return self._load_pickled_torch(data)
    # ...
        with torch.sparse.check_sparse_tensor_invariants():
            tensor = torch.load(filepath, weights_only=True)
            return tensor.to_dense()

    def encode_base64(self, media: torch.Tensor) -> str:
        return tensor2base64(media)
```
**EN:** `ImageEmbeddingMediaIO`: Image embedding MediaIO implementation. It extends `MediaIO`. Key methods include `__init__`, `load_bytes`, `load_base64`, `load_file`.
**CN:** `ImageEmbeddingMediaIO` 是该文件中的核心类，用于封装与 `ImageEmbeddingMediaIO` 相关的状态和行为。 它继承自 `MediaIO`。 关键方法包括 `__init__`, `load_bytes`, `load_base64`, `load_file`。

## Key Concepts / 关键概念
- **`ImageMediaIO`**: Core class that organizes module behavior. / **`ImageMediaIO`**：组织模块行为的核心类。
- **`ImageEmbeddingMediaIO`**: Core class that organizes module behavior. / **`ImageEmbeddingMediaIO`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: io, pathlib
- **Third-party / 第三方**: numpy, pybase64, torch, PIL
- **Internal vLLM / vLLM 内部依赖**: vllm.utils.serial_utils, ..image, .base
