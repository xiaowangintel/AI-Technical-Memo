# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines multimodal parsing, data structures, or helpers used by the input pipeline. / 定义输入流水线使用的多模态解析逻辑、数据结构或辅助函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-30)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import mimetypes
from collections import defaultdict
from collections.abc import Generator, Sequence
from itertools import groupby
from typing import TYPE_CHECKING, Any

import numpy as np
import numpy.typing as npt
from PIL import Image
from typing_extensions import deprecated

from vllm.inputs import MultiModalPlaceholders
from vllm.utils.import_utils import LazyLoader

from .hasher import MultiModalHasher
from .inputs import (
    BatchedTensorInputs,
    MultiModalFieldElem,
    MultiModalKwargsItem,
    MultiModalSharedField,
)
from .media import AudioMediaIO, ImageMediaIO, MediaConnector, VideoMediaIO

if TYPE_CHECKING:
    import torch.types
else:
    torch = LazyLoader("torch", globals(), "torch")
```
**EN:** Sets up the module with standard-library support such as `mimetypes`, `collections`, `collections.abc`, external packages such as `numpy`, `numpy.typing`, `PIL`, vLLM modules such as `vllm.inputs`, `vllm.utils.import_utils`, `.hasher`. It prepares the symbols later used by `encode_audio_base64`, `encode_audio_url`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.inputs`, `vllm.utils.import_utils`, `.hasher` 等 vLLM 内部依赖。 这些准备工作为后续的 `encode_audio_base64`, `encode_audio_url` 提供上下文。

### encode_audio_base64 (lines 33-41)
```python
def encode_audio_base64(
    audio: np.ndarray,
    sampling_rate: int,
    *,
    format: str = "WAV",
) -> str:
    """Encode audio as base64."""
    audio_io = AudioMediaIO()
    return audio_io.encode_base64((audio, sampling_rate), audio_format=format)
```
**EN:** `encode_audio_base64`: Encode audio as base64. It mainly works with `audio`, `sampling_rate`, `format`. Inside the body, it relies on `AudioMediaIO`, `audio_io.encode_base64` to complete the main steps.
**CN:** `encode_audio_base64` 负责把数据转换为编码后的表示。 它主要处理 `audio`, `sampling_rate`, `format` 等参数。 实现过程中会调用 `AudioMediaIO`, `audio_io.encode_base64` 等函数完成关键步骤。

### encode_audio_url (lines 44-53)
```python
def encode_audio_url(
    audio: np.ndarray,
    sampling_rate: int,
    *,
    format: str = "WAV",
) -> str:
    """Encode audio as a data URL."""
    audio_b64 = encode_audio_base64(audio, sampling_rate, format=format)
    mimetype = mimetypes.types_map.get("." + format.lower(), "audio")
    return f"data:{mimetype};base64,{audio_b64}"
```
**EN:** `encode_audio_url`: Encode audio as a data URL. It mainly works with `audio`, `sampling_rate`, `format`. Inside the body, it relies on `encode_audio_base64`, `mimetypes.types_map.get`, `format.lower` to complete the main steps.
**CN:** `encode_audio_url` 负责把数据转换为编码后的表示。 它主要处理 `audio`, `sampling_rate`, `format` 等参数。 实现过程中会调用 `encode_audio_base64`, `mimetypes.types_map.get`, `format.lower` 等函数完成关键步骤。

### encode_image_base64 (lines 56-68)
```python
def encode_image_base64(
    image: Image.Image,
    *,
    image_mode: str = "RGB",
    format: str = "PNG",
) -> str:
    """
    Encode a pillow image to base64 format.

    By default, the image is converted into RGB format before being encoded.
    """
    image_io = ImageMediaIO(image_mode=image_mode)
    return image_io.encode_base64(image, image_format=format)
```
**EN:** `encode_image_base64`: Encode a pillow image to base64 format. It mainly works with `image`, `image_mode`, `format`. Inside the body, it relies on `ImageMediaIO`, `image_io.encode_base64` to complete the main steps.
**CN:** `encode_image_base64` 负责把数据转换为编码后的表示。 它主要处理 `image`, `image_mode`, `format` 等参数。 实现过程中会调用 `ImageMediaIO`, `image_io.encode_base64` 等函数完成关键步骤。

### encode_image_url (lines 71-84)
```python
def encode_image_url(
    image: Image.Image,
    *,
    image_mode: str = "RGB",
    format: str = "PNG",
) -> str:
    """
    Encode a pillow image as a data URL.

    By default, the image is converted into RGB format before being encoded.
    """
    image_b64 = encode_image_base64(image, image_mode=image_mode, format=format)
    mimetype = mimetypes.types_map.get("." + format.lower(), "image")
    return f"data:{mimetype};base64,{image_b64}"
```
**EN:** `encode_image_url`: Encode a pillow image as a data URL. It mainly works with `image`, `image_mode`, `format`. Inside the body, it relies on `encode_image_base64`, `mimetypes.types_map.get`, `format.lower` to complete the main steps.
**CN:** `encode_image_url` 负责把数据转换为编码后的表示。 它主要处理 `image`, `image_mode`, `format` 等参数。 实现过程中会调用 `encode_image_base64`, `mimetypes.types_map.get`, `format.lower` 等函数完成关键步骤。

### encode_video_base64 (lines 87-94)
```python
def encode_video_base64(
    frames: npt.NDArray,
    *,
    format: str = "JPEG",
) -> str:
    image_io = ImageMediaIO()
    video_io = VideoMediaIO(image_io)
    return video_io.encode_base64(frames, video_format=format)
```
**EN:** `encode_video_base64` converts data into an encoded representation. It mainly works with `frames`, `format`. Inside the body, it relies on `ImageMediaIO`, `VideoMediaIO`, `video_io.encode_base64` to complete the main steps.
**CN:** `encode_video_base64` 负责把数据转换为编码后的表示。 它主要处理 `frames`, `format` 等参数。 实现过程中会调用 `ImageMediaIO`, `VideoMediaIO`, `video_io.encode_base64` 等函数完成关键步骤。

### encode_video_url (lines 97-109)
```python
def encode_video_url(
    frames: npt.NDArray,
    *,
    format: str = "JPEG",
) -> str:
    video_b64 = encode_video_base64(frames, format=format)

    if format.lower() == "jpeg":
        mimetype = "video/jpeg"
    else:
        mimetype = mimetypes.types_map.get("." + format.lower(), "video")

    return f"data:{mimetype};base64,{video_b64}"
```
**EN:** `encode_video_url` converts data into an encoded representation. It mainly works with `frames`, `format`. Inside the body, it relies on `encode_video_base64`, `format.lower`, `mimetypes.types_map.get` to complete the main steps.
**CN:** `encode_video_url` 负责把数据转换为编码后的表示。 它主要处理 `frames`, `format` 等参数。 实现过程中会调用 `encode_video_base64`, `format.lower`, `mimetypes.types_map.get` 等函数完成关键步骤。

### argsort_mm_positions (lines 112-132)
```python
def argsort_mm_positions(
    mm_positions: MultiModalPlaceholders,
) -> list[tuple[str, int]]:
    """
    Given a `MultiModalPlaceholders`, output a sequence of keys to
    sort the dictionary by `offset` (starting index in the input sequence)
    in ascending order.

    Returns:
        A list of `(modality, idx)`, which can be used to access an item
        by `mm_positions[modality][idx]`.
    """
    flat_items = (
        (modality, idx, item)
        for modality, items in mm_positions.items()
        for idx, item in enumerate(items)
    )

    sorted_flat_items = sorted(flat_items, key=lambda x: x[2].offset)

    return [(modality, idx) for modality, idx, _ in sorted_flat_items]
```
**EN:** `argsort_mm_positions`: Given a `MultiModalPlaceholders`, output a sequence of keys to sort the dictionary by `offset` (starting index in the input sequence) in ascending order. It mainly works with `mm_positions`. Inside the body, it relies on `sorted`, `mm_positions.items` to complete the main steps.
**CN:** `argsort_mm_positions` 负责实现本模块使用的辅助逻辑。 它主要处理 `mm_positions` 等参数。 实现过程中会调用 `sorted`, `mm_positions.items` 等函数完成关键步骤。

### _get_group_hash (lines 135-139)
```python
def _get_group_hash(elem: MultiModalFieldElem):
    if not isinstance(elem.field, MultiModalSharedField):
        return None

    return MultiModalHasher.hash_kwargs(data=elem.data)
```
**EN:** `_get_group_hash` retrieves data or state needed by the pipeline. It mainly works with `elem`. Inside the body, it relies on `MultiModalHasher.hash_kwargs` to complete the main steps.
**CN:** `_get_group_hash` 负责获取流水线所需的数据或状态。 它主要处理 `elem` 等参数。 实现过程中会调用 `MultiModalHasher.hash_kwargs` 等函数完成关键步骤。

### _batch_mm_items (lines 142-160)
```python
def _batch_mm_items(
    items: Sequence[MultiModalKwargsItem],
    *,
    device: torch.types.Device = None,
    pin_memory: bool = False,
):
    elems = defaultdict[str, list[MultiModalFieldElem]](list)
    for item in items:
        for key, elem in item.items():
            elems[key].append(elem)

    return {
        key: elems[0].field.reduce_data(
            elems,
            device=device,
            pin_memory=pin_memory,
        )
        for key, elems in elems.items()
    }
```
**EN:** `_batch_mm_items` implements helper logic used by this module. It mainly works with `items`, `device`, `pin_memory`. Inside the body, it relies on `defaultdict`, `item.items`, `elems.field.reduce_data` to complete the main steps.
**CN:** `_batch_mm_items` 负责实现本模块使用的辅助逻辑。 它主要处理 `items`, `device`, `pin_memory` 等参数。 实现过程中会调用 `defaultdict`, `item.items`, `elems.field.reduce_data` 等函数完成关键步骤。

### group_and_batch_mm_items (lines 163-208)
```python
def group_and_batch_mm_items(
    items: Sequence[MultiModalKwargsItem],
    *,
    device: torch.types.Device = None,
    pin_memory: bool = False,
) -> Generator[tuple[int, BatchedTensorInputs]]:
    """
    Group consecutive items (possibly from different requests) into batches.

    Items must be split across groups if any of the following occurs,
    as the batch would otherwise be invalid:
    - They have different fields (e.g. mixed image and embedding inputs).
    - They have different values in `MultiModalSharedField`.

    Args:
        items: List of `MultiModalKwargsItem`.
        device: The device to place the grouped tensors on.
        pin_memory: Whether to pin memory for faster host-to-device transfer.

    Yields:
        A tuple `(num_items, grouped_kwargs)`, where:
        - `kwargs` is a dictionary of keyword arguments to pass to the model;
        - `num_items` is the corresponding number of items.
    """
    # ...

        yield group_size, group_data

        start_idx += group_size

    assert start_idx == len(items)
```
**EN:** `group_and_batch_mm_items`: Group consecutive items (possibly from different requests) into batches. It mainly works with `items`, `device`, `pin_memory`. Inside the body, it relies on `sum`, `_batch_mm_items`, `groupby` to complete the main steps.
**CN:** `group_and_batch_mm_items` 负责实现本模块使用的辅助逻辑。 它主要处理 `items`, `device`, `pin_memory` 等参数。 实现过程中会调用 `sum`, `_batch_mm_items`, `groupby` 等函数完成关键步骤。

### group_and_batch_mm_kwargs (lines 211-247)
```python
def group_and_batch_mm_kwargs(
    mm_kwargs: list[tuple[str, MultiModalKwargsItem]],
    *,
    device: torch.types.Device = None,
    pin_memory: bool = False,
) -> Generator[tuple[str, int, BatchedTensorInputs], None, None]:
    """
    Group consecutive items (possibly from different requests) into batches.

    Items must be split across groups if any of the following occurs,
    as the batch would otherwise be invalid:
    - They have different fields (e.g. mixed image and embedding inputs).
    - They have different values in `MultiModalSharedField`.

    To simplify the implementation of `embed_multimodal`, we add another
    restriction that the items in a batch must belong to the same modality.

    Args:
        mm_kwargs: List of `(modality, item)`.
        device: The device to place the grouped tensors on.
        pin_memory: Whether to pin memory for faster host-to-device transfer.

    Yields:
        A tuple `(modality, num_items, grouped_kwargs)`, where:
    # ...
        for num_items, mm_kwargs_batch in group_and_batch_mm_items(
            items_lst,
            device=device,
            pin_memory=pin_memory,
        ):
            yield modality, num_items, mm_kwargs_batch
```
**EN:** `group_and_batch_mm_kwargs`: Group consecutive items (possibly from different requests) into batches. It mainly works with `mm_kwargs`, `device`, `pin_memory`. Inside the body, it relies on `groupby`, `group_and_batch_mm_items` to complete the main steps.
**CN:** `group_and_batch_mm_kwargs` 负责实现本模块使用的辅助逻辑。 它主要处理 `mm_kwargs`, `device`, `pin_memory` 等参数。 实现过程中会调用 `groupby`, `group_and_batch_mm_items` 等函数完成关键步骤。

### group_mm_kwargs_by_modality (lines 254-260)
```python
def group_mm_kwargs_by_modality(
    mm_kwargs: list[tuple[str, MultiModalKwargsItem]],
    *,
    device: torch.types.Device = None,
    pin_memory: bool = False,
) -> Generator[tuple[str, int, BatchedTensorInputs], None, None]:
    return group_and_batch_mm_kwargs(mm_kwargs, device=device, pin_memory=pin_memory)
```
**EN:** `group_mm_kwargs_by_modality` implements helper logic used by this module. It mainly works with `mm_kwargs`, `device`, `pin_memory`. Inside the body, it relies on `deprecated`, `group_and_batch_mm_kwargs` to complete the main steps.
**CN:** `group_mm_kwargs_by_modality` 负责实现本模块使用的辅助逻辑。 它主要处理 `mm_kwargs`, `device`, `pin_memory` 等参数。 实现过程中会调用 `deprecated`, `group_and_batch_mm_kwargs` 等函数完成关键步骤。

### fetch_audio (lines 263-281)
```python
def fetch_audio(
    audio_url: str,
    audio_io_kwargs: dict[str, Any] | None = None,
) -> tuple[np.ndarray, int | float]:
    """
    Args:
        audio_url: URL of the audio file to fetch.
        audio_io_kwargs: Additional kwargs passed to handle audio IO.

    Warning:
        This method has direct access to local files and is only intended
        to be called by user code. Never call this from the online server!
    """
    media_io_kwargs = None if not audio_io_kwargs else {"audio": audio_io_kwargs}
    media_connector = MediaConnector(
        media_io_kwargs=media_io_kwargs,
        allowed_local_media_path="/",
    )
    return media_connector.fetch_audio(audio_url)
```
**EN:** `fetch_audio`: Args: audio_url: URL of the audio file to fetch. It mainly works with `audio_url`, `audio_io_kwargs`. Inside the body, it relies on `MediaConnector`, `media_connector.fetch_audio` to complete the main steps.
**CN:** `fetch_audio` 负责获取远端或缓存数据供后续步骤使用。 它主要处理 `audio_url`, `audio_io_kwargs` 等参数。 实现过程中会调用 `MediaConnector`, `media_connector.fetch_audio` 等函数完成关键步骤。

### fetch_image (lines 284-302)
```python
def fetch_image(
    image_url: str,
    image_io_kwargs: dict[str, Any] | None = None,
) -> Image.Image:
    """
    Args:
        image_url: URL of the image file to fetch.
        image_io_kwargs: Additional kwargs passed to handle image IO.

    Warning:
        This method has direct access to local files and is only intended
        to be called by user code. Never call this from the online server!
    """
    media_io_kwargs = None if not image_io_kwargs else {"image": image_io_kwargs}
    media_connector = MediaConnector(
        media_io_kwargs=media_io_kwargs,
        allowed_local_media_path="/",
    )
    return media_connector.fetch_image(image_url)
```
**EN:** `fetch_image`: Args: image_url: URL of the image file to fetch. It mainly works with `image_url`, `image_io_kwargs`. Inside the body, it relies on `MediaConnector`, `media_connector.fetch_image` to complete the main steps.
**CN:** `fetch_image` 负责获取远端或缓存数据供后续步骤使用。 它主要处理 `image_url`, `image_io_kwargs` 等参数。 实现过程中会调用 `MediaConnector`, `media_connector.fetch_image` 等函数完成关键步骤。

### fetch_video (lines 305-323)
```python
def fetch_video(
    video_url: str,
    video_io_kwargs: dict[str, Any] | None = None,
) -> tuple[npt.NDArray, dict[str, Any]]:
    """
    Args:
        video_url: URL of the video file to fetch.
        video_io_kwargs: Additional kwargs passed to handle video IO.

    Warning:
        This method has direct access to local files and is only intended
        to be called by user code. Never call this from the online server!
    """
    media_io_kwargs = None if not video_io_kwargs else {"video": video_io_kwargs}
    media_connector = MediaConnector(
        media_io_kwargs=media_io_kwargs,
        allowed_local_media_path="/",
    )
    return media_connector.fetch_video(video_url)
```
**EN:** `fetch_video`: Args: video_url: URL of the video file to fetch. It mainly works with `video_url`, `video_io_kwargs`. Inside the body, it relies on `MediaConnector`, `media_connector.fetch_video` to complete the main steps.
**CN:** `fetch_video` 负责获取远端或缓存数据供后续步骤使用。 它主要处理 `video_url`, `video_io_kwargs` 等参数。 实现过程中会调用 `MediaConnector`, `media_connector.fetch_video` 等函数完成关键步骤。

## Key Concepts / 关键概念
- **`encode_audio_base64`**: Key helper or entry point in this file. / **`encode_audio_base64`**：本文件中的关键辅助函数或入口。
- **`encode_audio_url`**: Key helper or entry point in this file. / **`encode_audio_url`**：本文件中的关键辅助函数或入口。
- **`encode_image_base64`**: Key helper or entry point in this file. / **`encode_image_base64`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: mimetypes, collections, collections.abc, itertools, typing
- **Third-party / 第三方**: numpy, numpy.typing, PIL, typing_extensions, torch.types
- **Internal vLLM / vLLM 内部依赖**: vllm.inputs, vllm.utils.import_utils, .hasher, .inputs, .media
