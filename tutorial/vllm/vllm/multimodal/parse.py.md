# parse.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/parse.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines multimodal parsing, data structures, or helpers used by the input pipeline. / 定义输入流水线使用的多模态解析逻辑、数据结构或辅助函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-45)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from abc import ABC, abstractmethod
from collections import UserDict
from collections.abc import Callable, Iterator, Mapping, Sequence, Set
from typing import (
    TYPE_CHECKING,
    Any,
    Generic,
    Literal,
    NamedTuple,
    TypeAlias,
    TypeGuard,
    TypeVar,
)

import numpy as np
import torch
from typing_extensions import assert_never

from vllm.inputs import ModalityData, MultiModalDataDict, MultiModalUUIDDict
from vllm.utils.collection_utils import is_list_of
from vllm.utils.import_utils import LazyLoader
    # ...
_I = TypeVar("_I")

if TYPE_CHECKING:
    import PIL.Image as PILImage
else:
    PILImage = LazyLoader("PILImage", globals(), "PIL.Image")
```
**EN:** Sets up the module with standard-library support such as `abc`, `collections`, `collections.abc`, external packages such as `numpy`, `torch`, `typing_extensions`, vLLM modules such as `vllm.inputs`, `vllm.utils.collection_utils`, `vllm.utils.import_utils`. It prepares the symbols later used by `ModalityDataItems`, `ProcessorBatchItems`, `validate_embedding_ndim`, `parse_mm_uuids`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.inputs`, `vllm.utils.collection_utils`, `vllm.utils.import_utils` 等 vLLM 内部依赖。 这些准备工作为后续的 `ModalityDataItems`, `ProcessorBatchItems`, `validate_embedding_ndim`, `parse_mm_uuids` 提供上下文。

### ModalityDataItems (lines 48-101)
```python
class ModalityDataItems(ABC, Generic[_T, _I]):
    """
    Represents data items for a modality in
    [`MultiModalDataItems`][vllm.multimodal.parse.MultiModalDataItems].
    """

    def __init__(self, data: _T, modality: str) -> None:
        super().__init__()

        self.data: _T = data
        self.modality = modality

    def __repr__(self) -> str:
        return f"{type(self).__name__}(modality={self.modality!r}, len={len(self)})"

    def __len__(self) -> int:
        return self.get_count()

    def __getitem__(self, index: int) -> _I:
        return self.get(index)

    if TYPE_CHECKING:
        # Auto-generated
        def __iter__(self) -> Iterator[_I]: ...

    @abstractmethod
    def get_count(self) -> int:
        """Get the number of data items."""
    # ...
        raise NotImplementedError

    @abstractmethod
    def get_passthrough_data(self) -> Mapping[str, object]:
        """Get the data to pass directly to the model."""
        raise NotImplementedError
```
**EN:** `ModalityDataItems`: Represents data items for a modality in [`MultiModalDataItems`][vllm.multimodal.parse.MultiModalDataItems]. It extends `ABC`, `Generic`. Key methods include `__init__`, `get_count`, `get`, `get_all`.
**CN:** `ModalityDataItems` 是该文件中的核心类，用于封装与 `ModalityDataItems` 相关的状态和行为。 它继承自 `ABC`, `Generic`。 关键方法包括 `__init__`, `get_count`, `get`, `get_all`。

### ProcessorBatchItems (lines 104-125)
```python
class ProcessorBatchItems(ModalityDataItems[Sequence[_T], _T]):
    """Base class for data items that are arranged in a list."""

    def _unwrap(self, item: _T | MediaWithBytes[_T]) -> _T:
        """Extract media from wrapper if present."""
        return item.media if isinstance(item, MediaWithBytes) else item

    def get_count(self) -> int:
        return len(self.data)

    def get(self, index: int) -> _T:
        return self._unwrap(self.data[index])

    def get_item_for_hash(self, index: int) -> _T | MediaWithBytes[_T]:
        # Return raw item for hashing (preserves original_bytes if present)
        return self.data[index]

    def get_processor_data(self) -> Mapping[str, object]:
        return {f"{self.modality}s": self.get_all()}

    def get_passthrough_data(self) -> Mapping[str, object]:
        return {}
```
**EN:** `ProcessorBatchItems`: Base class for data items that are arranged in a list. It extends `ModalityDataItems`. Key methods include `get_count`, `get`, `get_item_for_hash`, `get_processor_data`.
**CN:** `ProcessorBatchItems` 是该文件中的核心类，用于封装与 `ProcessorBatchItems` 相关的状态和行为。 它继承自 `ModalityDataItems`。 关键方法包括 `get_count`, `get`, `get_item_for_hash`, `get_processor_data`。

### validate_embedding_ndim (lines 128-149)
```python
def validate_embedding_ndim(
    tensor: torch.Tensor,
    modality: str,
    index: int | None = None,
) -> None:
    """Validate tensor ndim for multimodal embeddings.

    Single embeddings should be 2D (seq_len, hidden_size).
    Batched embeddings should be 3D (batch, seq_len, hidden_size).

    Args:
        tensor: The tensor to validate.
        modality: The modality name for error messages (e.g., "image", "audio").
        index: Optional index for list items, included in error messages.
    """
    if tensor.ndim < 2 or tensor.ndim > 3:
        idx_str = f" [{index}]" if index is not None else ""
        raise ValueError(
            f"{modality.capitalize()} embedding{idx_str} must be 2D "
            f"(seq_len, hidden_size) or 3D (batch, seq_len, hidden_size), "
            f"got {tensor.ndim}D tensor with shape {tuple(tensor.shape)}"
        )
```
**EN:** `validate_embedding_ndim`: Validate tensor ndim for multimodal embeddings. It mainly works with `tensor`, `modality`, `index`. Inside the body, it relies on `ValueError`, `modality.capitalize` to complete the main steps.
**CN:** `validate_embedding_ndim` 负责校验取值并在约束不满足时抛出清晰错误。 它主要处理 `tensor`, `modality`, `index` 等参数。 实现过程中会调用 `ValueError`, `modality.capitalize` 等函数完成关键步骤。

### EmbeddingItems overview (lines 152-236)
```python
class EmbeddingItems(
    ModalityDataItems[torch.Tensor | list[torch.Tensor], torch.Tensor]
):
    """
    Base class for data items that are expressed as a batched embedding tensor,
    or a list of embedding tensors (one per item).
    """

    def __init__(
        self,
        data: torch.Tensor | list[torch.Tensor],
        modality: str,
        expected_hidden_size: int | None = None,
    ) -> None:
        super().__init__(data, modality)

        # Validate ndim first (before hidden_size which depends on correct ndim)
        self._validate_ndim()

        # Validate hidden dimension if expected size is provided
        if expected_hidden_size is not None:
            self._validate_hidden_size(expected_hidden_size)

    def _validate_ndim(self) -> None:
        """Validate that embedding tensors have correct ndim (2D or 3D)."""
        if isinstance(self.data, torch.Tensor):
            validate_embedding_ndim(self.data, self.modality)
    # ...
```
**EN:** `EmbeddingItems`: Base class for data items that are expressed as a batched embedding tensor, or a list of embedding tensors (one per item). It extends `ModalityDataItems`. Key methods include `__init__`, `get_count`, `get`, `get_processor_data`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `EmbeddingItems` 是该文件中的核心类，用于封装与 `EmbeddingItems` 相关的状态和行为。 它继承自 `ModalityDataItems`。 关键方法包括 `__init__`, `get_count`, `get`, `get_processor_data`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### EmbeddingItems.__init__ (lines 160-173)
```python
    def __init__(
        self,
        data: torch.Tensor | list[torch.Tensor],
        modality: str,
        expected_hidden_size: int | None = None,
    ) -> None:
        super().__init__(data, modality)

        # Validate ndim first (before hidden_size which depends on correct ndim)
        self._validate_ndim()

        # Validate hidden dimension if expected size is provided
        if expected_hidden_size is not None:
            self._validate_hidden_size(expected_hidden_size)
```
**EN:** `__init__` initializes state required by the module. It mainly works with `data`, `modality`, `expected_hidden_size`. Inside the body, it relies on `super.__init__`, `self._validate_ndim`, `self._validate_hidden_size` to complete the main steps.
**CN:** `__init__` 负责初始化模块所需的状态。 它主要处理 `data`, `modality`, `expected_hidden_size` 等参数。 实现过程中会调用 `super.__init__`, `self._validate_ndim`, `self._validate_hidden_size` 等函数完成关键步骤。

### EmbeddingItems._validate_hidden_size (lines 189-215)
```python
    def _validate_hidden_size(self, expected_hidden_size: int) -> None:
        """Validate that embedding hidden dimension matches expected size.

        This validates hidden dimensions to prevent vulnerabilities: Embeddings
        with correct ndim but wrong hidden dimension could bypass initial
        checks and cause crashes during model inference when dimensions don't match.
        """
        if isinstance(self.data, torch.Tensor):
            # Batched tensor: shape is (batch, seq_len, hidden_size)
            actual_hidden_size = self.data.shape[-1]
            if actual_hidden_size != expected_hidden_size:
                raise ValueError(
                    f"{self.modality.capitalize()} embedding hidden dimension "
                    f"mismatch: got {actual_hidden_size}, but model expects "
                    f"{expected_hidden_size}. Embedding shape: {tuple(self.data.shape)}"
                )
        else:
            # List of tensors: each has shape (seq_len, hidden_size)
            for idx, tensor in enumerate(self.data):
                actual_hidden_size = tensor.shape[-1]
                if actual_hidden_size != expected_hidden_size:
                    raise ValueError(
                        f"{self.modality.capitalize()} embedding [{idx}] hidden "
                        f"dimension mismatch: got {actual_hidden_size}, but model "
                        f"expects {expected_hidden_size}. "
                        f"Embedding shape: {tuple(tensor.shape)}"
                    )
```
**EN:** `_validate_hidden_size`: Validate that embedding hidden dimension matches expected size. It mainly works with `expected_hidden_size`. Inside the body, it relies on `ValueError`, `self.modality.capitalize` to complete the main steps.
**CN:** `_validate_hidden_size` 负责校验取值并在约束不满足时抛出清晰错误。 它主要处理 `expected_hidden_size` 等参数。 实现过程中会调用 `ValueError`, `self.modality.capitalize` 等函数完成关键步骤。

### EmbeddingItems._validate_ndim (lines 175-187)
```python
    def _validate_ndim(self) -> None:
        """Validate that embedding tensors have correct ndim (2D or 3D)."""
        if isinstance(self.data, torch.Tensor):
            validate_embedding_ndim(self.data, self.modality)
        else:
            # List of tensors: each should be 2D (seq_len, hidden_size)
            for idx, tensor in enumerate(self.data):
                if tensor.ndim != 2:
                    raise ValueError(
                        f"{self.modality.capitalize()} embedding [{idx}] must be "
                        f"2D (seq_len, hidden_size), got {tensor.ndim}D tensor "
                        f"with shape {tuple(tensor.shape)}"
                    )
```
**EN:** `_validate_ndim`: Validate that embedding tensors have correct ndim (2D or 3D). Inside the body, it relies on `validate_embedding_ndim`, `ValueError`, `self.modality.capitalize` to complete the main steps.
**CN:** `_validate_ndim` 负责校验取值并在约束不满足时抛出清晰错误。 实现过程中会调用 `validate_embedding_ndim`, `ValueError`, `self.modality.capitalize` 等函数完成关键步骤。

### EmbeddingItems._unwrap (lines 217-221)
```python
    def _unwrap(
        self, item: torch.Tensor | MediaWithBytes[torch.Tensor]
    ) -> torch.Tensor:
        """Extract media from wrapper if present."""
        return item.media if isinstance(item, MediaWithBytes) else item
```
**EN:** `_unwrap`: Extract media from wrapper if present. It mainly works with `item`.
**CN:** `_unwrap` 负责实现本模块使用的辅助逻辑。 它主要处理 `item` 等参数。

### DictEmbeddingItems (lines 239-296)
```python
class DictEmbeddingItems(
    ModalityDataItems[Mapping[str, torch.Tensor], Mapping[str, torch.Tensor]]
):
    """
    Base class for data items that are expressed as a dictionary of tensors.

    Usually, the dictionary keys correspond to the outputs of HF processor.
    """

    def __init__(
        self,
        data: Mapping[str, torch.Tensor],
        modality: str,
        required_fields: set[str],
        fields_factory: Callable[
            [Mapping[str, torch.Tensor]],
            Mapping[str, MultiModalFieldConfig],
        ],
    ) -> None:
        from transformers.feature_extraction_utils import BatchFeature

        super().__init__(data, modality)

        missing_required_data_keys = required_fields - data.keys()
        if missing_required_data_keys:
            data_keys = set(data.keys())
            msg = (
                f"The data should contain the fields: {required_fields}, "
    # ...

    def get_processor_data(self) -> Mapping[str, object]:
        return {}

    def get_passthrough_data(self) -> Mapping[str, object]:
        return self.data
```
**EN:** `DictEmbeddingItems`: Base class for data items that are expressed as a dictionary of tensors. It extends `ModalityDataItems`. Key methods include `__init__`, `get_count`, `get`, `get_processor_data`.
**CN:** `DictEmbeddingItems` 是该文件中的核心类，用于封装与 `DictEmbeddingItems` 相关的状态和行为。 它继承自 `ModalityDataItems`。 关键方法包括 `__init__`, `get_count`, `get`, `get_processor_data`。

### AudioProcessorItems (lines 299-308)
```python
class AudioProcessorItems(ProcessorBatchItems[HfAudioItem | None]):
    def __init__(self, data: Sequence[HfAudioItem | None]) -> None:
        super().__init__(data, "audio")

    def get_audio_length(self, item_idx: int) -> int:
        audio = self.get(item_idx)
        if audio is None:
            raise ValueError(f"Cannot get length of cached audio at {item_idx}")

        return len(audio)
```
**EN:** Defines the `AudioProcessorItems` class used by this module. It extends `ProcessorBatchItems`. Key methods include `__init__`, `get_audio_length`.
**CN:** `AudioProcessorItems` 是该文件中的核心类，用于封装与 `AudioProcessorItems` 相关的状态和行为。 它继承自 `ProcessorBatchItems`。 关键方法包括 `__init__`, `get_audio_length`。

### AudioEmbeddingItems (lines 311-317)
```python
class AudioEmbeddingItems(EmbeddingItems):
    def __init__(
        self,
        data: torch.Tensor | list[torch.Tensor],
        expected_hidden_size: int | None = None,
    ) -> None:
        super().__init__(data, "audio", expected_hidden_size)
```
**EN:** Defines the `AudioEmbeddingItems` class used by this module. It extends `EmbeddingItems`. Key methods include `__init__`.
**CN:** `AudioEmbeddingItems` 是该文件中的核心类，用于封装与 `AudioEmbeddingItems` 相关的状态和行为。 它继承自 `EmbeddingItems`。 关键方法包括 `__init__`。

### ImageSize (lines 320-322)
```python
class ImageSize(NamedTuple):
    width: int
    height: int
```
**EN:** Defines the `ImageSize` class used by this module. It extends `NamedTuple`.
**CN:** `ImageSize` 是该文件中的核心类，用于封装与 `ImageSize` 相关的状态和行为。 它继承自 `NamedTuple`。

### ImageProcessorItems (lines 325-340)
```python
class ImageProcessorItems(ProcessorBatchItems[HfImageItem | None]):
    def __init__(self, data: Sequence[HfImageItem | None]) -> None:
        super().__init__(data, "image")

    def get_image_size(self, item_idx: int) -> ImageSize:
        image = self.get(item_idx)
        if image is None:
            raise ValueError(f"Cannot get size of cached image at {item_idx}")

        if isinstance(image, PILImage.Image):
            return ImageSize(*image.size)
        if isinstance(image, (np.ndarray, torch.Tensor)):
            _, h, w = image.shape
            return ImageSize(w, h)

        assert_never(image)
```
**EN:** Defines the `ImageProcessorItems` class used by this module. It extends `ProcessorBatchItems`. Key methods include `__init__`, `get_image_size`.
**CN:** `ImageProcessorItems` 是该文件中的核心类，用于封装与 `ImageProcessorItems` 相关的状态和行为。 它继承自 `ProcessorBatchItems`。 关键方法包括 `__init__`, `get_image_size`。

### ImageEmbeddingItems (lines 343-349)
```python
class ImageEmbeddingItems(EmbeddingItems):
    def __init__(
        self,
        data: torch.Tensor | list[torch.Tensor],
        expected_hidden_size: int | None = None,
    ) -> None:
        super().__init__(data, "image", expected_hidden_size)
```
**EN:** Defines the `ImageEmbeddingItems` class used by this module. It extends `EmbeddingItems`. Key methods include `__init__`.
**CN:** `ImageEmbeddingItems` 是该文件中的核心类，用于封装与 `ImageEmbeddingItems` 相关的状态和行为。 它继承自 `EmbeddingItems`。 关键方法包括 `__init__`。

### VideoProcessorItems (lines 352-384)
```python
class VideoProcessorItems(ProcessorBatchItems[HfVideoItem | None]):
    def __init__(
        self,
        data: Sequence[HfVideoItem | None],
        metadata: dict[str, Any] | list[dict[str, Any] | None] | None = None,
    ) -> None:
        super().__init__(data, "video")

        self.metadata = metadata

    def get_num_frames(self, item_idx: int) -> int:
        video = self.get(item_idx)
        if video is None:
            raise ValueError(f"Cannot get length of cached video at {item_idx}")

        return len(video)

    def get_frame_size(self, item_idx: int) -> ImageSize:
        video = self.get(item_idx)
        if video is None:
            raise ValueError(f"Cannot get size of cached video at {item_idx}")
        if len(video) == 0:
            raise ValueError(f"Cannot get size of empty video at {item_idx}")

        image = video[0]

        if isinstance(image, PILImage.Image):
            return ImageSize(*image.size)
        if isinstance(image, (np.ndarray, torch.Tensor)):
            _, h, w = image.shape
            return ImageSize(w, h)

        assert_never(image)
```
**EN:** Defines the `VideoProcessorItems` class used by this module. It extends `ProcessorBatchItems`. Key methods include `__init__`, `get_num_frames`, `get_frame_size`.
**CN:** `VideoProcessorItems` 是该文件中的核心类，用于封装与 `VideoProcessorItems` 相关的状态和行为。 它继承自 `ProcessorBatchItems`。 关键方法包括 `__init__`, `get_num_frames`, `get_frame_size`。

## Key Concepts / 关键概念
- **`ModalityDataItems`**: Core class that organizes module behavior. / **`ModalityDataItems`**：组织模块行为的核心类。
- **`ProcessorBatchItems`**: Core class that organizes module behavior. / **`ProcessorBatchItems`**：组织模块行为的核心类。
- **`EmbeddingItems`**: Core class that organizes module behavior. / **`EmbeddingItems`**：组织模块行为的核心类。
- **`validate_embedding_ndim`**: Key helper or entry point in this file. / **`validate_embedding_ndim`**：本文件中的关键辅助函数或入口。
- **`parse_mm_uuids`**: Key helper or entry point in this file. / **`parse_mm_uuids`**：本文件中的关键辅助函数或入口。

## Dependencies / 依赖关系
- **Standard library / 标准库**: abc, collections, collections.abc, typing
- **Third-party / 第三方**: numpy, torch, typing_extensions, PIL.Image, transformers.feature_extraction_utils
- **Internal vLLM / vLLM 内部依赖**: vllm.inputs, vllm.utils.collection_utils, vllm.utils.import_utils, .audio, .inputs, .media
