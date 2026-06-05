# connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/media/connector.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements media loading, validation, and conversion utilities for multimodal inputs. / 实现多模态输入所需的媒体加载、校验与转换工具。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-47)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import asyncio
import atexit
import contextlib
import hashlib
import os
import tempfile
import time
from concurrent.futures import ThreadPoolExecutor
from pathlib import Path
from typing import Any, TypeVar
from urllib.request import url2pathname

import numpy as np
import numpy.typing as npt
import torch
from PIL import Image, UnidentifiedImageError
from urllib3.util import Url, parse_url

import vllm.envs as envs
from vllm.connections import HTTPConnection, global_http_connection
from vllm.logger import init_logger
    # ...

MODALITY_IO_MAP: dict[str, type[MediaIO]] = {
    "audio": AudioMediaIO,
    "image": ImageMediaIO,
    "video": VideoMediaIO,
}
```
**EN:** Sets up the module with standard-library support such as `asyncio`, `atexit`, `contextlib`, external packages such as `numpy`, `numpy.typing`, `torch`, vLLM modules such as `vllm.envs`, `vllm.connections`, `vllm.logger`. It prepares the symbols later used by `MediaConnector`, `merge_media_io_kwargs`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.envs`, `vllm.connections`, `vllm.logger` 等 vLLM 内部依赖。 这些准备工作为后续的 `MediaConnector`, `merge_media_io_kwargs` 提供上下文。

### merge_media_io_kwargs (lines 50-70)
```python
def merge_media_io_kwargs(
    defaults: dict[str, dict[str, Any]] | None,
    overrides: dict[str, dict[str, Any]] | None,
) -> dict[str, dict[str, Any]] | None:
    """Merge config-level and per-request media_io_kwargs per modality.

    Each modality key is merged using the corresponding MediaIO subclass's
    ``merge_kwargs``, which may apply modality-specific logic (e.g.
    VideoMediaIO clears cross-dependent fps/num_frames fields).
    """
    if not defaults and not overrides:
        return None
    all_keys = set(defaults or {}) | set(overrides or {})
    merged = {}
    for key in all_keys:
        io_cls = MODALITY_IO_MAP.get(key, MediaIO)
        merged[key] = io_cls.merge_kwargs(
            (defaults or {}).get(key),
            (overrides or {}).get(key),
        )
    return merged or None
```
**EN:** `merge_media_io_kwargs`: Merge config-level and per-request media_io_kwargs per modality. It mainly works with `defaults`, `overrides`. Inside the body, it relies on `MODALITY_IO_MAP.get`, `io_cls.merge_kwargs`, `get` to complete the main steps.
**CN:** `merge_media_io_kwargs` 负责合并相关配置或数据并生成规范化结果。 它主要处理 `defaults`, `overrides` 等参数。 实现过程中会调用 `MODALITY_IO_MAP.get`, `io_cls.merge_kwargs`, `get` 等函数完成关键步骤。

### MediaConnector overview (lines 74-512)
```python
class MediaConnector:
    """Configuration values can be user-provided either by --media-io-kwargs or
    by the runtime API field "media_io_kwargs". Ensure proper validation and
    error handling.
    """

    def __init__(
        self,
        media_io_kwargs: dict[str, dict[str, Any]] | None = None,
        connection: HTTPConnection = global_http_connection,
        *,
        allowed_local_media_path: str = "",
        allowed_media_domains: list[str] | None = None,
    ) -> None:
        """
        Args:
            media_io_kwargs: Additional args passed to process media
                             inputs, keyed by modalities. For example,
                             to set num_frames for video, set
                             `--media-io-kwargs '{"video":{"num_frames":40}}'`
            connection: HTTP connection client to download media contents.
            allowed_local_media_path: A local directory to load media files from.
            allowed_media_domains: If set, only media URLs that belong to this
                                   domain can be used for multi-modal inputs.
        """
        super().__init__()

    # ...
```
**EN:** `MediaConnector`: Configuration values can be user-provided either by --media-io-kwargs or by the runtime API field "media_io_kwargs". Key methods include `__init__`, `load_from_url`, `load_from_url_async`, `fetch_audio`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `MediaConnector` 是该文件中的核心类，用于封装与 `MediaConnector` 相关的状态和行为。 关键方法包括 `__init__`, `load_from_url`, `load_from_url_async`, `fetch_audio`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### MediaConnector.__init__ (lines 80-153)
```python
    def __init__(
        self,
        media_io_kwargs: dict[str, dict[str, Any]] | None = None,
        connection: HTTPConnection = global_http_connection,
        *,
        allowed_local_media_path: str = "",
        allowed_media_domains: list[str] | None = None,
    ) -> None:
        """
        Args:
            media_io_kwargs: Additional args passed to process media
                             inputs, keyed by modalities. For example,
                             to set num_frames for video, set
                             `--media-io-kwargs '{"video":{"num_frames":40}}'`
            connection: HTTP connection client to download media contents.
            allowed_local_media_path: A local directory to load media files from.
            allowed_media_domains: If set, only media URLs that belong to this
                                   domain can be used for multi-modal inputs.
        """
        super().__init__()
    # ...
                )
            except OSError:
                logger.warning(
                    "VLLM_MEDIA_CACHE path %s is not writable, media caching disabled",
                    media_cache,
                )
```
**EN:** `__init__`: Args: media_io_kwargs: Additional args passed to process media inputs, keyed by modalities. It mainly works with `media_io_kwargs`, `connection`, `allowed_local_media_path`, `allowed_media_domains`. Inside the body, it relies on `super.__init__`, `Path`, `allowed_local_media_path_.exists` to complete the main steps.
**CN:** `__init__` 负责初始化模块所需的状态。 它主要处理 `media_io_kwargs`, `connection`, `allowed_local_media_path`, `allowed_media_domains` 等参数。 实现过程中会调用 `super.__init__`, `Path`, `allowed_local_media_path_.exists` 等函数完成关键步骤。

### MediaConnector.load_from_url_async (lines 320-368)
```python
    async def load_from_url_async(
        self,
        url: str,
        media_io: MediaIO[_M],
        *,
        fetch_timeout: int | None = None,
    ) -> _M:
        loop = asyncio.get_running_loop()

        if url[:5].lower() == "data:":
            future = loop.run_in_executor(
                global_thread_pool, self._load_data_url, url, media_io
            )
            return await future

        url_spec = parse_url(url)

        if url_spec.scheme and url_spec.scheme.startswith("http"):
            self._assert_url_in_allowed_media_domains(url_spec)

    # ...
            future = loop.run_in_executor(
                global_thread_pool, self._load_file_url, url_spec, media_io
            )
            return await future
        msg = "The URL must be either a HTTP, data or file URL."
        raise ValueError(msg)
```
**EN:** `load_from_url_async` loads resources for downstream use. It mainly works with `url`, `media_io`, `fetch_timeout`. Inside the body, it relies on `asyncio.get_running_loop`, `parse_url`, `ValueError` to complete the main steps.
**CN:** `load_from_url_async` 负责加载下游使用的资源。 它主要处理 `url`, `media_io`, `fetch_timeout` 等参数。 实现过程中会调用 `asyncio.get_running_loop`, `parse_url`, `ValueError` 等函数完成关键步骤。

### MediaConnector.fetch_image (lines 400-423)
```python
    def fetch_image(
        self,
        image_url: str,
        *,
        image_mode: str = "RGB",
    ) -> Image.Image:
        """
        Load a PIL image from an HTTP or base64 data URL.

        By default, the image is converted into RGB format.
        """
        image_io = ImageMediaIO(
            image_mode=image_mode, **self.media_io_kwargs.get("image", {})
        )

        try:
            return self.load_from_url(
                image_url,
                image_io,
                fetch_timeout=envs.VLLM_IMAGE_FETCH_TIMEOUT,
            )
        except UnidentifiedImageError as e:
            # convert to ValueError to be properly caught upstream
            raise ValueError(str(e)) from e
```
**EN:** `fetch_image`: Load a PIL image from an HTTP or base64 data URL. It mainly works with `image_url`, `image_mode`. Inside the body, it relies on `ImageMediaIO`, `self.load_from_url`, `self.media_io_kwargs.get` to complete the main steps.
**CN:** `fetch_image` 负责获取远端或缓存数据供后续步骤使用。 它主要处理 `image_url`, `image_mode` 等参数。 实现过程中会调用 `ImageMediaIO`, `self.load_from_url`, `self.media_io_kwargs.get` 等函数完成关键步骤。

### MediaConnector.fetch_image_async (lines 425-448)
```python
    async def fetch_image_async(
        self,
        image_url: str,
        *,
        image_mode: str = "RGB",
    ) -> Image.Image:
        """
        Asynchronously load a PIL image from an HTTP or base64 data URL.

        By default, the image is converted into RGB format.
        """
        image_io = ImageMediaIO(
            image_mode=image_mode, **self.media_io_kwargs.get("image", {})
        )

        try:
            return await self.load_from_url_async(
                image_url,
                image_io,
                fetch_timeout=envs.VLLM_IMAGE_FETCH_TIMEOUT,
            )
        except UnidentifiedImageError as e:
            # convert to ValueError to be properly caught upstream
            raise ValueError(str(e)) from e
```
**EN:** `fetch_image_async`: Asynchronously load a PIL image from an HTTP or base64 data URL. It mainly works with `image_url`, `image_mode`. Inside the body, it relies on `ImageMediaIO`, `self.media_io_kwargs.get`, `self.load_from_url_async` to complete the main steps.
**CN:** `fetch_image_async` 负责获取远端或缓存数据供后续步骤使用。 它主要处理 `image_url`, `image_mode` 等参数。 实现过程中会调用 `ImageMediaIO`, `self.media_io_kwargs.get`, `self.load_from_url_async` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`MediaConnector`**: Core class that organizes module behavior. / **`MediaConnector`**：组织模块行为的核心类。
- **`merge_media_io_kwargs`**: Key helper or entry point in this file. / **`merge_media_io_kwargs`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: asyncio, atexit, contextlib, hashlib, os, tempfile, time, concurrent.futures, pathlib, typing, urllib.request
- **Third-party / 第三方**: numpy, numpy.typing, torch, PIL, urllib3.util
- **Internal vLLM / vLLM 内部依赖**: vllm.envs, vllm.connections, vllm.logger, vllm.utils.registry, .audio, .base, .image, .video
