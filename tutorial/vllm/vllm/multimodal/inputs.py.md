# inputs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/inputs.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements inputs support for the `multimodal` portion of vLLM. / 为 vLLM 的 `multimodal` 子目录实现与 输入 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-28)
```python
from abc import ABC, abstractmethod

from collections import UserDict, defaultdict

from collections.abc import Mapping, Sequence

from dataclasses import dataclass

from functools import cached_property, partial

from itertools import accumulate

from typing import (
    TYPE_CHECKING,
    Any,
    Literal,
    TypeAlias,
    TypedDict,
    Union,
    cast,
)

import numpy as np

from PIL.Image import Image

from typing_extensions import TypeVar

from vllm.utils.collection_utils import is_list_of

from vllm.utils.import_utils import LazyLoader

from vllm.utils.jsontree import json_map_leaves

from .media import MediaWithBytes
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 38-1015)
```python
HfImageItem: TypeAlias = Union["Image", np.ndarray, "torch.Tensor"]

HfVideoItem: TypeAlias = Union[
    list["Image"], np.ndarray, "torch.Tensor", list[np.ndarray], list["torch.Tensor"]
]

HfAudioItem: TypeAlias = Union[list[float], np.ndarray, "torch.Tensor"]

ImageItem: TypeAlias = Union[HfImageItem, "torch.Tensor", MediaWithBytes[HfImageItem]]

VideoItem: TypeAlias = Union[
    HfVideoItem, "torch.Tensor", tuple[HfVideoItem, dict[str, Any]]
]

AudioItem: TypeAlias = Union[HfAudioItem, tuple[np.ndarray, float], "torch.Tensor"]

VisionChunk: TypeAlias = VisionChunkImage | VisionChunkVideo

NestedTensors: TypeAlias = Union[
    list["NestedTensors"],
    list["torch.Tensor"],
    "torch.Tensor",
    tuple["torch.Tensor", ...],
]

BatchedTensorInputs: TypeAlias = dict[str, NestedTensors]

_I = TypeVar(
# ... omitted for brevity ...
```
**EN:** This constant/configuration block defines `HfImageItem`, `HfVideoItem`, `HfAudioItem`, `ImageItem`, `VideoItem`, `AudioItem`, `VisionChunk`, `NestedTensors`, `BatchedTensorInputs`, `_I`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `HfImageItem`, `HfVideoItem`, `HfAudioItem`, `ImageItem`, `VideoItem`, `AudioItem`, `VisionChunk`, `NestedTensors`, `BatchedTensorInputs`, `_I`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Large-file note
```python
# Focused on representative top-level definitions for a large module.
# The full file contains additional helpers following the same patterns.
```
**EN:** Because the file is large, the analysis below focuses on the most important top-level definitions and leaves repetitive helpers summarized at a higher level.
**CN:** 由于文件较大，下面的分析聚焦最重要的顶层定义；其余重复性辅助逻辑将以更高层次方式概括。

### Class `VisionChunkImage` (lines 96-101)
```python
class VisionChunkImage(TypedDict):
    """Represents an image wrapped as a vision chunk."""

    type: Literal["image"]
    image: Image
    uuid: str | None
```
**EN:** Class `VisionChunkImage` is a structured building block in this module. It inherits from `TypedDict`. The class docstring says: Represents an image wrapped as a vision chunk.
**CN:** 类 `VisionChunkImage` 是该模块中的结构化构件，继承自 `TypedDict`。 类文档说明：Represents an image wrapped as a vision chunk.

### Class `VisionChunkVideo` (lines 104-111)
```python
class VisionChunkVideo(TypedDict):
    """Represents a video chunk with metadata."""

    type: Literal["video_chunk"]
    video_chunk: list[Image]
    uuid: str | None
    prompt: str
    video_idx: int
```
**EN:** Class `VisionChunkVideo` is a structured building block in this module. It inherits from `TypedDict`. The class docstring says: Represents a video chunk with metadata.
**CN:** 类 `VisionChunkVideo` 是该模块中的结构化构件，继承自 `TypedDict`。 类文档说明：Represents a video chunk with metadata.

### Class `PlaceholderRange` (lines 119-215)
```python
class PlaceholderRange:
    """
    Placeholder location information for multi-modal data.

    Example:

    Prompt: `AAAA BBBB What is in these images?`

    Images A and B will have:

    ` ` `
    A: PlaceholderRange(offset=0, length=4)
    B: PlaceholderRange(offset=5, length=4)
    ` ` `
    """

    offset: int
    """The start index of the placeholder in the prompt."""

    length: int
    """The length of the placeholder."""

    is_embed: "torch.Tensor | None" = None
    # ... omitted for brevity ...

        return nested_tensors_equal(self.is_embed, other.is_embed)
```
**EN:** Class `PlaceholderRange` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `embeds_cumsum`, `get_num_embeds`, `get_embeds_indices_in_range`, `extract_embeds_range`, `__eq__`, which define initialization, validation, transformation, or access patterns. The class docstring says: Placeholder location information for multi-modal data.
**CN:** 类 `PlaceholderRange` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `embeds_cumsum`, `get_num_embeds`, `get_embeds_indices_in_range`, `extract_embeds_range`, `__eq__`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Placeholder location information for multi-modal data.

### Method `PlaceholderRange.embeds_cumsum` (lines 148-150)
```python
    def embeds_cumsum(self) -> list[int] | None:
        # python list so python indexing avoids torch C++ overhead/conversions/deallocs
        return None if self.is_embed is None else self.is_embed.cumsum(dim=0).tolist()
```
**EN:** Method `PlaceholderRange.embeds_cumsum` provides a reusable helper around the module's main workflow. Key calls such as `self.is_embed.cumsum(dim=0).tolist`, `self.is_embed.cumsum` show the concrete execution path.
**CN:** Method `PlaceholderRange.embeds_cumsum` 为模块主流程提供可复用的辅助逻辑。 像 `self.is_embed.cumsum(dim=0).tolist`, `self.is_embed.cumsum` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PlaceholderRange.get_num_embeds` (lines 152-156)
```python
    def get_num_embeds(self) -> int:
        if self.embeds_cumsum is None:
            return self.length

        return self.embeds_cumsum[-1] if self.embeds_cumsum else 0
```
**EN:** Method `PlaceholderRange.get_num_embeds` provides a reusable helper around the module's main workflow.
**CN:** Method `PlaceholderRange.get_num_embeds` 为模块主流程提供可复用的辅助逻辑。

### Method `PlaceholderRange.get_embeds_indices_in_range` (lines 158-177)
```python
    def get_embeds_indices_in_range(
        self, start_idx: int, end_idx: int
    ) -> tuple[int, int]:
        """
        Returns the starting and ending indices of the embeddings of encoder outputs
        in the range of [start_idx, end_idx) in the placeholders.

        For example, given:
        PlaceholderRange(offset=2, length=5, is_embed=[False, True, False, True, True])

        If start_idx=3 and end_idx=5, the output is (1, 3) because we want to get
        the second and the third embeddings from the encoder output.
        """
        if self.embeds_cumsum is None:
            return start_idx, end_idx

        embeds_start_idx = self.embeds_cumsum[start_idx - 1] if start_idx > 0 else 0
        embeds_end_idx = self.embeds_cumsum[end_idx - 1] if end_idx > 0 else 0

        return embeds_start_idx, embeds_end_idx
```
**EN:** Method `PlaceholderRange.get_embeds_indices_in_range` provides a reusable helper around the module's main workflow. The docstring highlights: Returns the starting and ending indices of the embeddings of encoder outputs in the range of [start_idx, end_idx) in the placeholders.
**CN:** Method `PlaceholderRange.get_embeds_indices_in_range` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Returns the starting and ending indices of the embeddings of encoder outputs in the range of [start_idx, end_idx) in the placeholders.

### Method `PlaceholderRange.extract_embeds_range` (lines 179-202)
```python
    def extract_embeds_range(self) -> list[tuple[int, int]]:
        """Extract the start and end indices of the embedded region in prompt.

        For example, given `PlaceholderRange(offset=2, length=5)` and
        `is_embed = [False, True, False, True, True]`, the output is
        `[(1 + offset, 1 + offset), (3 + offset, 4 + offset)]`.

        Returns:
            A tuple `(start, end)` representing the start and end
            indices (inclusive) of the embedded region.
            Returns full placeholder range if `is_embed` is `None`.
        """
        if self.is_embed is None:
            return [(self.offset, self.offset + self.length - 1)]

        mask_i = self.is_embed.int()
        starts = torch.nonzero(
            torch.diff(mask_i, prepend=mask_i.new_zeros(1)) == 1
        ).flatten()
    # ... omitted for brevity ...
        ranges = torch.stack((starts, ends), dim=1) + self.offset
        return [tuple(x) for x in ranges.tolist()]
```
**EN:** Method `PlaceholderRange.extract_embeds_range` provides a reusable helper around the module's main workflow. The docstring highlights: Extract the start and end indices of the embedded region in prompt. Key calls such as `self.is_embed.int`, `torch.nonzero(torch.diff(mask_i, prepend=mask_i.new_zeros(1)) == 1).flatten`, `torch.nonzero`, `torch.diff`, `mask_i.new_zeros` show the concrete execution path.
**CN:** Method `PlaceholderRange.extract_embeds_range` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Extract the start and end indices of the embedded region in prompt. 像 `self.is_embed.int`, `torch.nonzero(torch.diff(mask_i, prepend=mask_i.new_zeros(1)) == 1).flatten`, `torch.nonzero`, `torch.diff`, `mask_i.new_zeros` 这样的关键调用展示了该代码块的具体执行路径。

### Method `PlaceholderRange.__eq__` (lines 204-215)
```python
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, self.__class__):
            return False
        if not (self.offset, self.length) == (other.offset, other.length):
            return False

        if self.is_embed is None:
            return other.is_embed is None
        if other.is_embed is None:
            return self.is_embed is None

        return nested_tensors_equal(self.is_embed, other.is_embed)
```
**EN:** Method `PlaceholderRange.__eq__` provides a reusable helper around the module's main workflow. Key calls such as `isinstance`, `nested_tensors_equal` show the concrete execution path.
**CN:** Method `PlaceholderRange.__eq__` 为模块主流程提供可复用的辅助逻辑。 像 `isinstance`, `nested_tensors_equal` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalFeatureSpec` (lines 302-345)
```python
class MultiModalFeatureSpec:
    """
    Represents a single multimodal input with its processed data and metadata.

    Used to track multimodal data through processing and caching.
    A request containing multiple multimodal items will have one
    `MultiModalFeatureSpec` per item.
    """

    data: "MultiModalKwargsItem | None"
    """
    Represents multimodal data for this feature.

    Can be `None` if the item is cached, to skip IPC between API server
    and engine core processes.
    """

    modality: str
    """The input modality, e.g., `"image"`, `"audio"`, `"video"`."""

    identifier: str
    """The hash for caching encoder outputs (with LoRA prefix if applicable)."""

    # ... omitted for brevity ...

        return dict(kwargs)
```
**EN:** Class `MultiModalFeatureSpec` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `gather_kwargs`, which define initialization, validation, transformation, or access patterns. The class docstring says: Represents a single multimodal input with its processed data and metadata.
**CN:** 类 `MultiModalFeatureSpec` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `gather_kwargs`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Represents a single multimodal input with its processed data and metadata.

### Method `MultiModalFeatureSpec.gather_kwargs` (lines 335-345)
```python
    def gather_kwargs(features: list["MultiModalFeatureSpec"], keys: set[str]):
        kwargs = defaultdict[str, list[NestedTensors]](list)

        for f in features:
            item = f.data
            if item is not None:
                for k in keys:
                    if k in item:
                        kwargs[k].append(item[k].data)

        return dict(kwargs)
```
**EN:** Method `MultiModalFeatureSpec.gather_kwargs` parses configuration, arguments, or structured metadata. Key calls such as `kwargs[k].append`, `dict` show the concrete execution path.
**CN:** Method `MultiModalFeatureSpec.gather_kwargs` 负责解析配置、参数或结构化元数据。 像 `kwargs[k].append`, `dict` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalFieldElem` (lines 349-382)
```python
class MultiModalFieldElem:
    """
    Represents a processed keyword argument to pass to a model for a
    [`MultiModalKwargsItem`][vllm.multimodal.inputs.MultiModalKwargsItem].
    """

    data: NestedTensors
    """
    The tensor data of this field in
    [`MultiModalKwargsItem`][vllm.multimodal.inputs.MultiModalKwargsItem],
    i.e. the value of the keyword argument to be passed to the model.

    It may be set to `None` if it is determined that the item is cached
    in `EngineCore`.
    """

    field: "BaseMultiModalField"
    """
    Defines how to combine the tensor data of this field with others
    in order to batch multi-modal items together for model inference.
    """

    def __eq__(self, other: object) -> bool:
    # ... omitted for brevity ...

        return data_equal and type(self.field) is type(other.field)  # noqa: E721
```
**EN:** Class `MultiModalFieldElem` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `__eq__`, which define initialization, validation, transformation, or access patterns. The class docstring says: Represents a processed keyword argument to pass to a model for a [`MultiModalKwargsItem`][vllm.multimodal.inputs.MultiModalKwargsItem].
**CN:** 类 `MultiModalFieldElem` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `__eq__`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Represents a processed keyword argument to pass to a model for a [`MultiModalKwargsItem`][vllm.multimodal.inputs.MultiModalKwargsItem].

### Method `MultiModalFieldElem.__eq__` (lines 371-382)
```python
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, self.__class__):
            return False

        if self.data is None:
            data_equal = other.data is None
        elif other.data is None:
            data_equal = self.data is None
        else:
            data_equal = nested_tensors_equal(self.data, other.data)

        return data_equal and type(self.field) is type(other.field)  # noqa: E721
```
**EN:** Method `MultiModalFieldElem.__eq__` provides a reusable helper around the module's main workflow. Key calls such as `isinstance`, `nested_tensors_equal`, `type` show the concrete execution path.
**CN:** Method `MultiModalFieldElem.__eq__` 为模块主流程提供可复用的辅助逻辑。 像 `isinstance`, `nested_tensors_equal`, `type` 这样的关键调用展示了该代码块的具体执行路径。

### Class `BaseMultiModalField` (lines 386-460)
```python
class BaseMultiModalField(ABC):
    """
    Defines how to interpret tensor data belonging to a keyword argument for
    [`MultiModalKwargsItems`][vllm.multimodal.inputs.MultiModalKwargsItems],
    and vice versa.
    """

    keep_on_cpu: bool = False
    """
    If `True`, then this field is excluded from being moved to the accelerator when
    [`group_and_batch_mm_items`][vllm.multimodal.utils.group_and_batch_mm_items]
    is called to batch the data.
    """

    def _field_factory(self):
        f = partial(MultiModalFieldElem, field=self)

        # Allow passing data as positional argument
        def factory(data: NestedTensors) -> MultiModalFieldElem:
            return f(data=data)

        return factory

    # ... omitted for brevity ...
        out = self._reduce_data(batch, pin_memory=pin_memory)
        return _nested_tensors_h2d(out, device=device)
```
**EN:** Class `BaseMultiModalField` is a structured building block in this module. It inherits from `ABC` and uses a dataclass-style declaration to store explicit state. Key methods include `_field_factory`, `build_elems`, `_reduce_data`, `reduce_data`, which define initialization, validation, transformation, or access patterns. The class docstring says: Defines how to interpret tensor data belonging to a keyword argument for [`MultiModalKwargsItems`][vllm.multimodal.inputs.MultiModalKwargsItems], and vice versa.
**CN:** 类 `BaseMultiModalField` 是该模块中的结构化构件，继承自 `ABC`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `_field_factory`, `build_elems`, `_reduce_data`, `reduce_data`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Defines how to interpret tensor data belonging to a keyword argument for [`MultiModalKwargsItems`][vllm.multimodal.inputs.MultiModalKwargsItems], and vice versa.

### Method `BaseMultiModalField._field_factory` (lines 400-407)
```python
    def _field_factory(self):
        f = partial(MultiModalFieldElem, field=self)

        # Allow passing data as positional argument
        def factory(data: NestedTensors) -> MultiModalFieldElem:
            return f(data=data)

        return factory
```
**EN:** Method `BaseMultiModalField._field_factory` provides a reusable helper around the module's main workflow. Key calls such as `partial`, `f` show the concrete execution path.
**CN:** Method `BaseMultiModalField._field_factory` 为模块主流程提供可复用的辅助逻辑。 像 `partial`, `f` 这样的关键调用展示了该代码块的具体执行路径。

### Method `BaseMultiModalField.build_elems` (lines 410-424)
```python
    def build_elems(
        self,
        modality: str,
        key: str,
        data: NestedTensors,
    ) -> Sequence[MultiModalFieldElem]:
        """
        Construct
        [`MultiModalFieldElem`][vllm.multimodal.inputs.MultiModalFieldElem]
        instances to represent the provided data.

        This is the inverse of
        [`reduce_data`][vllm.multimodal.inputs.BaseMultiModalField.reduce_data].
        """
        raise NotImplementedError
```
**EN:** Method `BaseMultiModalField.build_elems` constructs derived objects, runtime state, or helper structures. The docstring highlights: Construct [`MultiModalFieldElem`][vllm.multimodal.inputs.MultiModalFieldElem] instances to represent the provided data.
**CN:** Method `BaseMultiModalField.build_elems` 负责构造派生对象、运行时状态或辅助结构。 文档字符串强调：Construct [`MultiModalFieldElem`][vllm.multimodal.inputs.MultiModalFieldElem] instances to represent the provided data.

### Method `BaseMultiModalField._reduce_data` (lines 427-433)
```python
    def _reduce_data(
        self,
        batch: list[NestedTensors],
        *,
        pin_memory: bool,
    ) -> NestedTensors:
        raise NotImplementedError
```
**EN:** Method `BaseMultiModalField._reduce_data` provides a reusable helper around the module's main workflow.
**CN:** Method `BaseMultiModalField._reduce_data` 为模块主流程提供可复用的辅助逻辑。

### Method `BaseMultiModalField.reduce_data` (lines 435-460)
```python
    def reduce_data(
        self,
        elems: list[MultiModalFieldElem],
        *,
        device: torch.types.Device = None,
        pin_memory: bool = False,
    ) -> NestedTensors:
        """
        Merge the data from multiple instances of
        [`MultiModalFieldElem`][vllm.multimodal.inputs.MultiModalFieldElem].

        This is the inverse of
        [`build_elems`][vllm.multimodal.inputs.BaseMultiModalField.build_elems].
        """
        field_types = [type(item.field) for item in elems]
        if len(set(field_types)) > 1:
            raise ValueError(f"Cannot merge different {field_types=}")

        if device is not None and self.keep_on_cpu:
    # ... omitted for brevity ...
        out = self._reduce_data(batch, pin_memory=pin_memory)
        return _nested_tensors_h2d(out, device=device)
```
**EN:** Method `BaseMultiModalField.reduce_data` provides a reusable helper around the module's main workflow. The docstring highlights: Merge the data from multiple instances of [`MultiModalFieldElem`][vllm.multimodal.inputs.MultiModalFieldElem]. Key calls such as `type`, `len`, `set`, `ValueError`, `self._reduce_data` show the concrete execution path.
**CN:** Method `BaseMultiModalField.reduce_data` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Merge the data from multiple instances of [`MultiModalFieldElem`][vllm.multimodal.inputs.MultiModalFieldElem]. 像 `type`, `len`, `set`, `ValueError`, `self._reduce_data` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalBatchedField` (lines 464-502)
```python
class MultiModalBatchedField(BaseMultiModalField):
    """
    Info:
        [`MultiModalFieldConfig.batched`][vllm.multimodal.inputs.MultiModalFieldConfig.batched]
    """

    def build_elems(
        self,
        modality: str,
        key: str,
        data: NestedTensors,
    ) -> Sequence[MultiModalFieldElem]:
        field_factory = self._field_factory()
        return [field_factory(item) for item in data]

    def _reduce_data(
        self,
        batch: list[NestedTensors],
        *,
        pin_memory: bool,
    ) -> NestedTensors:
        if len(batch) > 0 and is_list_of(batch, torch.Tensor, check="all"):
            batch = cast(list[torch.Tensor], batch)
    # ... omitted for brevity ...

        return batch
```
**EN:** Class `MultiModalBatchedField` is a structured building block in this module. It inherits from `BaseMultiModalField` and uses a dataclass-style declaration to store explicit state. Key methods include `build_elems`, `_reduce_data`, which define initialization, validation, transformation, or access patterns. The class docstring says: Info: [`MultiModalFieldConfig.batched`][vllm.multimodal.inputs.MultiModalFieldConfig.batched]
**CN:** 类 `MultiModalBatchedField` 是该模块中的结构化构件，继承自 `BaseMultiModalField`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `build_elems`, `_reduce_data`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Info: [`MultiModalFieldConfig.batched`][vllm.multimodal.inputs.MultiModalFieldConfig.batched]

### Method `MultiModalBatchedField.build_elems` (lines 470-477)
```python
    def build_elems(
        self,
        modality: str,
        key: str,
        data: NestedTensors,
    ) -> Sequence[MultiModalFieldElem]:
        field_factory = self._field_factory()
        return [field_factory(item) for item in data]
```
**EN:** Method `MultiModalBatchedField.build_elems` constructs derived objects, runtime state, or helper structures. Key calls such as `self._field_factory`, `field_factory` show the concrete execution path.
**CN:** Method `MultiModalBatchedField.build_elems` 负责构造派生对象、运行时状态或辅助结构。 像 `self._field_factory`, `field_factory` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalBatchedField._reduce_data` (lines 479-502)
```python
    def _reduce_data(
        self,
        batch: list[NestedTensors],
        *,
        pin_memory: bool,
    ) -> NestedTensors:
        if len(batch) > 0 and is_list_of(batch, torch.Tensor, check="all"):
            batch = cast(list[torch.Tensor], batch)
            if len(batch) == 1:
                # An optimization when `batch` contains only one tensor:
                # - produce exactly same result as `torch.stack(batch)`
                # - will achieve zero-copy if the tensor is contiguous
                return batch[0].unsqueeze(0).contiguous()
            first_shape = batch[0].shape
            if all(elem.shape == first_shape for elem in batch):
                out = torch.empty(
                    (len(batch), *batch[0].shape),
                    dtype=batch[0].dtype,
                    device=batch[0].device,
    # ... omitted for brevity ...

        return batch
```
**EN:** Method `MultiModalBatchedField._reduce_data` provides a reusable helper around the module's main workflow. Key calls such as `len`, `is_list_of`, `cast`, `batch[0].unsqueeze(0).contiguous`, `batch[0].unsqueeze` show the concrete execution path.
**CN:** Method `MultiModalBatchedField._reduce_data` 为模块主流程提供可复用的辅助逻辑。 像 `len`, `is_list_of`, `cast`, `batch[0].unsqueeze(0).contiguous`, `batch[0].unsqueeze` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalFlatField` (lines 506-603)
```python
class MultiModalFlatField(BaseMultiModalField):
    """
    Info:
        [`MultiModalFieldConfig.flat`][vllm.multimodal.inputs.MultiModalFieldConfig.flat]
        [`MultiModalFieldConfig.flat_from_sizes`][vllm.multimodal.inputs.MultiModalFieldConfig.flat_from_sizes]
    """

    slices: Sequence[slice] | Sequence[Sequence[slice]]
    dim: int = 0

    def build_elems(
        self,
        modality: str,
        key: str,
        data: NestedTensors,
    ) -> Sequence[MultiModalFieldElem]:
        field_factory = self._field_factory()
        if not is_list_of(self.slices, slice, check="all"):
            assert isinstance(data, torch.Tensor), (
                "torch.Tensor is required for multiple slices"
            )
        return [field_factory(data[cast(slice, s)]) for s in self.slices]

    # ... omitted for brevity ...
        assert self.dim == 0, "dim == 0 is required for nested list"
        return [e for elem in batch for e in elem]
```
**EN:** Class `MultiModalFlatField` is a structured building block in this module. It inherits from `BaseMultiModalField` and uses a dataclass-style declaration to store explicit state. Key methods include `build_elems`, `_reduce_data`, which define initialization, validation, transformation, or access patterns. The class docstring says: Info: [`MultiModalFieldConfig.flat`][vllm.multimodal.inputs.MultiModalFieldConfig.flat] [`MultiModalFieldConfig.flat_from_sizes`][vllm.multimodal.inputs.MultiModalFieldConfig.fl...
**CN:** 类 `MultiModalFlatField` 是该模块中的结构化构件，继承自 `BaseMultiModalField`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `build_elems`, `_reduce_data`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Info: [`MultiModalFieldConfig.flat`][vllm.multimodal.inputs.MultiModalFieldConfig.flat] [`MultiModalFieldConfig.flat_from_sizes`][vllm.multimodal.inputs.MultiModalFieldConfig.fl...

### Method `MultiModalFlatField.build_elems` (lines 516-527)
```python
    def build_elems(
        self,
        modality: str,
        key: str,
        data: NestedTensors,
    ) -> Sequence[MultiModalFieldElem]:
        field_factory = self._field_factory()
        if not is_list_of(self.slices, slice, check="all"):
            assert isinstance(data, torch.Tensor), (
                "torch.Tensor is required for multiple slices"
            )
        return [field_factory(data[cast(slice, s)]) for s in self.slices]
```
**EN:** Method `MultiModalFlatField.build_elems` constructs derived objects, runtime state, or helper structures. Key calls such as `self._field_factory`, `is_list_of`, `isinstance`, `field_factory`, `cast` show the concrete execution path.
**CN:** Method `MultiModalFlatField.build_elems` 负责构造派生对象、运行时状态或辅助结构。 像 `self._field_factory`, `is_list_of`, `isinstance`, `field_factory`, `cast` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalFlatField._reduce_data` (lines 529-603)
```python
    def _reduce_data(
        self,
        batch: list[NestedTensors],
        *,
        pin_memory: bool,
    ) -> NestedTensors:
        if len(batch) > 0 and is_list_of(batch, torch.Tensor, check="all"):
            batch = cast(list[torch.Tensor], batch)
            if len(batch) == 1:
                # An optimization when `batch` contains only one tensor:
                # - produce exactly same result as `torch.concat(batch)`
                # - will achieve zero-copy if the tensor is contiguous
                return batch[0].contiguous()

            dim = self.dim + (self.dim < 0) * len(batch[0].shape)

            def _shape_before_after(tensor: torch.Tensor):
                return tensor.shape[:dim], tensor.shape[dim + 1 :]

    # ... omitted for brevity ...
        assert self.dim == 0, "dim == 0 is required for nested list"
        return [e for elem in batch for e in elem]
```
**EN:** Method `MultiModalFlatField._reduce_data` provides a reusable helper around the module's main workflow. Key calls such as `len`, `is_list_of`, `cast`, `batch[0].contiguous`, `_shape_before_after` show the concrete execution path.
**CN:** Method `MultiModalFlatField._reduce_data` 为模块主流程提供可复用的辅助逻辑。 像 `len`, `is_list_of`, `cast`, `batch[0].contiguous`, `_shape_before_after` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalSharedField` (lines 607-630)
```python
class MultiModalSharedField(BaseMultiModalField):
    """
    Info:
        [`MultiModalFieldConfig.shared`][vllm.multimodal.inputs.MultiModalFieldConfig.shared]
    """

    batch_size: int

    def build_elems(
        self,
        modality: str,
        key: str,
        data: NestedTensors,
    ) -> Sequence[MultiModalFieldElem]:
        field_factory = self._field_factory()
        return [field_factory(data)] * self.batch_size

    def _reduce_data(
        self,
        batch: list[NestedTensors],
        *,
        pin_memory: bool,
    ) -> NestedTensors:
        return batch[0]
```
**EN:** Class `MultiModalSharedField` is a structured building block in this module. It inherits from `BaseMultiModalField` and uses a dataclass-style declaration to store explicit state. Key methods include `build_elems`, `_reduce_data`, which define initialization, validation, transformation, or access patterns. The class docstring says: Info: [`MultiModalFieldConfig.shared`][vllm.multimodal.inputs.MultiModalFieldConfig.shared]
**CN:** 类 `MultiModalSharedField` 是该模块中的结构化构件，继承自 `BaseMultiModalField`，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `build_elems`, `_reduce_data`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Info: [`MultiModalFieldConfig.shared`][vllm.multimodal.inputs.MultiModalFieldConfig.shared]

### Method `MultiModalSharedField.build_elems` (lines 615-622)
```python
    def build_elems(
        self,
        modality: str,
        key: str,
        data: NestedTensors,
    ) -> Sequence[MultiModalFieldElem]:
        field_factory = self._field_factory()
        return [field_factory(data)] * self.batch_size
```
**EN:** Method `MultiModalSharedField.build_elems` constructs derived objects, runtime state, or helper structures. Key calls such as `self._field_factory`, `field_factory` show the concrete execution path.
**CN:** Method `MultiModalSharedField.build_elems` 负责构造派生对象、运行时状态或辅助结构。 像 `self._field_factory`, `field_factory` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalSharedField._reduce_data` (lines 624-630)
```python
    def _reduce_data(
        self,
        batch: list[NestedTensors],
        *,
        pin_memory: bool,
    ) -> NestedTensors:
        return batch[0]
```
**EN:** Method `MultiModalSharedField._reduce_data` provides a reusable helper around the module's main workflow.
**CN:** Method `MultiModalSharedField._reduce_data` 为模块主流程提供可复用的辅助逻辑。

### Class `MultiModalFieldConfig` (lines 634-851)
```python
class MultiModalFieldConfig:
    @staticmethod
    def batched(modality: str, *, keep_on_cpu: bool = False):
        """
        Defines a field where an element in the batch is obtained by
        indexing into the first dimension of the underlying data.

        Args:
            modality: The modality of the multi-modal item that uses this
                keyword argument.
            keep_on_cpu: Whether to keep this field on the CPU for the model inputs.

        Example:

        ` ` `
        Input:
            Data: [[AAAA]
                [BBBB]
                [CCCC]]

        Output:
            Element 1: [AAAA]
            Element 2: [BBBB]
    # ... omitted for brevity ...
    ) -> Sequence[MultiModalFieldElem]:
        return self.field.build_elems(self.modality, key, batch)
```
**EN:** Class `MultiModalFieldConfig` is a structured building block in this module and uses a dataclass-style declaration to store explicit state. Key methods include `batched`, `flat`, `flat_from_sizes`, `shared`, `build_elems`, which define initialization, validation, transformation, or access patterns.
**CN:** 类 `MultiModalFieldConfig` 是该模块中的结构化构件，并使用 dataclass 风格声明来保存显式状态。 关键方法包括 `batched`, `flat`, `flat_from_sizes`, `shared`, `build_elems`，它们共同定义初始化、校验、变换或访问模式。

### Method `MultiModalFieldConfig.batched` (lines 636-663)
```python
    def batched(modality: str, *, keep_on_cpu: bool = False):
        """
        Defines a field where an element in the batch is obtained by
        indexing into the first dimension of the underlying data.

        Args:
            modality: The modality of the multi-modal item that uses this
                keyword argument.
            keep_on_cpu: Whether to keep this field on the CPU for the model inputs.

        Example:

        ` ` `
        Input:
            Data: [[AAAA]
                [BBBB]
                [CCCC]]

        Output:
    # ... omitted for brevity ...
            modality=modality,
        )
```
**EN:** Method `MultiModalFieldConfig.batched` provides a reusable helper around the module's main workflow. The docstring highlights: Defines a field where an element in the batch is obtained by indexing into the first dimension of the underlying data. Key calls such as `MultiModalFieldConfig`, `MultiModalBatchedField` show the concrete execution path.
**CN:** Method `MultiModalFieldConfig.batched` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Defines a field where an element in the batch is obtained by indexing into the first dimension of the underlying data. 像 `MultiModalFieldConfig`, `MultiModalBatchedField` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalFieldConfig.flat` (lines 666-725)
```python
    def flat(
        modality: str,
        slices: Sequence[slice] | Sequence[Sequence[slice]],
        dim: int = 0,
        *,
        keep_on_cpu: bool = False,
    ):
        """
        Defines a field where an element in the batch is obtained by
        slicing along the first dimension of the underlying data.

        Args:
            modality: The modality of the multi-modal item that uses this
                keyword argument.
            slices: For each multi-modal item, a slice (dim=0) or a tuple of
                slices (dim>0) that is used to extract the data corresponding
                to it.
            dim: The dimension to extract data, default to 0.
            keep_on_cpu: Whether to keep this field on the CPU for the model inputs.
    # ... omitted for brevity ...
            modality=modality,
        )
```
**EN:** Method `MultiModalFieldConfig.flat` provides a reusable helper around the module's main workflow. The docstring highlights: Defines a field where an element in the batch is obtained by slicing along the first dimension of the underlying data. Key calls such as `MultiModalFieldConfig`, `MultiModalFlatField` show the concrete execution path.
**CN:** Method `MultiModalFieldConfig.flat` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Defines a field where an element in the batch is obtained by slicing along the first dimension of the underlying data. 像 `MultiModalFieldConfig`, `MultiModalFlatField` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalFieldConfig.flat_from_sizes` (lines 728-798)
```python
    def flat_from_sizes(
        modality: str,
        size_per_item: "torch.Tensor",
        dim: int = 0,
        *,
        keep_on_cpu: bool = False,
    ):
        """
        Defines a field where an element in the batch is obtained by
        slicing along the first dimension of the underlying data.

        Args:
            modality: The modality of the multi-modal item that uses this
                keyword argument.
            size_per_item: For each multi-modal item, the size of the slice
                that is used to extract the data corresponding to it.
            dim: The dimension to slice, default to 0.
            keep_on_cpu: Whether to keep this field on the CPU for the model inputs.

    # ... omitted for brevity ...
            keep_on_cpu=keep_on_cpu,
        )
```
**EN:** Method `MultiModalFieldConfig.flat_from_sizes` provides a reusable helper around the module's main workflow. The docstring highlights: Defines a field where an element in the batch is obtained by slicing along the first dimension of the underlying data. Key calls such as `ValueError`, `accumulate`, `slice`, `range`, `len` show the concrete execution path.
**CN:** Method `MultiModalFieldConfig.flat_from_sizes` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Defines a field where an element in the batch is obtained by slicing along the first dimension of the underlying data. 像 `ValueError`, `accumulate`, `slice`, `range`, `len` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalFieldConfig.shared` (lines 801-841)
```python
    def shared(
        modality: str,
        batch_size: int,
        *,
        keep_on_cpu: bool = False,
    ):
        """
        Defines a field where an element in the batch is obtained by
        taking the entirety of the underlying data.

        This means that the data is the same for each element in the batch.

        Args:
            modality: The modality of the multi-modal item that uses this
                keyword argument.
            batch_size: The number of multi-modal items which share this data.
            keep_on_cpu: Whether to keep this field on the CPU for the model inputs.

        Example:
    # ... omitted for brevity ...
            modality=modality,
        )
```
**EN:** Method `MultiModalFieldConfig.shared` provides a reusable helper around the module's main workflow. The docstring highlights: Defines a field where an element in the batch is obtained by taking the entirety of the underlying data. Key calls such as `MultiModalFieldConfig`, `MultiModalSharedField` show the concrete execution path.
**CN:** Method `MultiModalFieldConfig.shared` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Defines a field where an element in the batch is obtained by taking the entirety of the underlying data. 像 `MultiModalFieldConfig`, `MultiModalSharedField` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalFieldConfig.build_elems` (lines 846-851)
```python
    def build_elems(
        self,
        key: str,
        batch: NestedTensors,
    ) -> Sequence[MultiModalFieldElem]:
        return self.field.build_elems(self.modality, key, batch)
```
**EN:** Method `MultiModalFieldConfig.build_elems` constructs derived objects, runtime state, or helper structures. Key calls such as `self.field.build_elems` show the concrete execution path.
**CN:** Method `MultiModalFieldConfig.build_elems` 负责构造派生对象、运行时状态或辅助结构。 像 `self.field.build_elems` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalKwargsItem` (lines 854-871)
```python
class MultiModalKwargsItem(UserDict[str, MultiModalFieldElem]):
    """
    A dictionary of processed keyword arguments to pass to the model,
    corresponding to a single item in
    [`MultiModalDataItems`][vllm.multimodal.parse.MultiModalDataItems].
    """

    @staticmethod
    def dummy(nbytes: int = 1):
        """Convenience class for testing."""
        mm_elem = MultiModalFieldElem(
            data=torch.empty(nbytes, dtype=torch.uint8),
            field=MultiModalSharedField(batch_size=1),
        )
        return MultiModalKwargsItem({"dummy": mm_elem})

    def get_data(self) -> dict[str, NestedTensors]:
        return {key: elem.data for key, elem in self.items()}
```
**EN:** Class `MultiModalKwargsItem` is a structured building block in this module. It inherits from `UserDict[str, MultiModalFieldElem]`. Key methods include `dummy`, `get_data`, which define initialization, validation, transformation, or access patterns. The class docstring says: A dictionary of processed keyword arguments to pass to the model, corresponding to a single item in [`MultiModalDataItems`][vllm.multimodal.parse.MultiModalDataItems].
**CN:** 类 `MultiModalKwargsItem` 是该模块中的结构化构件，继承自 `UserDict[str, MultiModalFieldElem]`。 关键方法包括 `dummy`, `get_data`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：A dictionary of processed keyword arguments to pass to the model, corresponding to a single item in [`MultiModalDataItems`][vllm.multimodal.parse.MultiModalDataItems].

### Method `MultiModalKwargsItem.dummy` (lines 862-868)
```python
    def dummy(nbytes: int = 1):
        """Convenience class for testing."""
        mm_elem = MultiModalFieldElem(
            data=torch.empty(nbytes, dtype=torch.uint8),
            field=MultiModalSharedField(batch_size=1),
        )
        return MultiModalKwargsItem({"dummy": mm_elem})
```
**EN:** Method `MultiModalKwargsItem.dummy` provides a reusable helper around the module's main workflow. The docstring highlights: Convenience class for testing. Key calls such as `MultiModalFieldElem`, `torch.empty`, `MultiModalSharedField`, `MultiModalKwargsItem` show the concrete execution path.
**CN:** Method `MultiModalKwargsItem.dummy` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Convenience class for testing. 像 `MultiModalFieldElem`, `torch.empty`, `MultiModalSharedField`, `MultiModalKwargsItem` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalKwargsItem.get_data` (lines 870-871)
```python
    def get_data(self) -> dict[str, NestedTensors]:
        return {key: elem.data for key, elem in self.items()}
```
**EN:** Method `MultiModalKwargsItem.get_data` provides a reusable helper around the module's main workflow. Key calls such as `self.items` show the concrete execution path.
**CN:** Method `MultiModalKwargsItem.get_data` 为模块主流程提供可复用的辅助逻辑。 像 `self.items` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalKwargsItems` (lines 882-1009)
```python
class MultiModalKwargsItems(UserDict[str, Sequence[_I]]):
    """
    A dictionary of processed multi-modal inputs by modality.

    For example, given a processor that processes
    images into `pixel_values` and `image_grid_thw`,
    and audios into `input_audio_features`,
    a prompt with 2 images and 1 audio will be processed
    into a `MultiModalKwargsItems` with the following structure:

    ` ` `python
    MultiModalKwargsItems(
        {
            "image": [
                # For the first image
                MultiModalKwargsItem({"pixel_values": ..., "image_grid_thw": ...}),
                # For the second imgae
                MultiModalKwargsItem({"pixel_values": ..., "image_grid_thw": ...}),
            ],
            "audio": [
                # For the first audio
                MultiModalKwargsItem({"input_audio_features": ...}),
            ],
    # ... omitted for brevity ...

        return out_data
```
**EN:** Class `MultiModalKwargsItems` is a structured building block in this module. It inherits from `UserDict[str, Sequence[_I]]`. Key methods include `from_hf_inputs`, `__getitem__`, `require_data`, `get_data`, which define initialization, validation, transformation, or access patterns. The class docstring says: A dictionary of processed multi-modal inputs by modality.
**CN:** 类 `MultiModalKwargsItems` 是该模块中的结构化构件，继承自 `UserDict[str, Sequence[_I]]`。 关键方法包括 `from_hf_inputs`, `__getitem__`, `require_data`, `get_data`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：A dictionary of processed multi-modal inputs by modality.

### Method `MultiModalKwargsItems.from_hf_inputs` (lines 919-952)
```python
    def from_hf_inputs(
        hf_inputs: "BatchFeature",
        config_by_key: Mapping[str, MultiModalFieldConfig],
    ):
        # NOTE: This skips fields in `hf_inputs` that are not in `config_by_key`
        # We assume that those fields are not used in vLLM
        elems_by_key = dict[str, Sequence[MultiModalFieldElem]]()
        keys_by_modality = defaultdict[str, set[str]](set)
        for key, config in config_by_key.items():
            batch = hf_inputs.get(key)
            if batch is not None:
                elems = config.build_elems(key, batch)
                if len(elems) > 0:
                    elems_by_key[key] = elems
                    keys_by_modality[config.modality].add(key)

        items_by_modality = dict[str, list[MultiModalKwargsItem]]()
        for modality, keys in keys_by_modality.items():
            elems_in_modality = {k: elems_by_key[k] for k in keys}
    # ... omitted for brevity ...

        return MultiModalKwargsItems(items_by_modality)
```
**EN:** Method `MultiModalKwargsItems.from_hf_inputs` provides a reusable helper around the module's main workflow. Key calls such as `config_by_key.items`, `hf_inputs.get`, `config.build_elems`, `len`, `keys_by_modality[config.modality].add` show the concrete execution path.
**CN:** Method `MultiModalKwargsItems.from_hf_inputs` 为模块主流程提供可复用的辅助逻辑。 像 `config_by_key.items`, `hf_inputs.get`, `config.build_elems`, `len`, `keys_by_modality[config.modality].add` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalKwargsItems.__getitem__` (lines 954-961)
```python
    def __getitem__(self, modality: str) -> Sequence[_I]:
        if modality not in self:
            raise KeyError(
                f"Modality {modality!r} not found. "
                f"Available modalities: {set(self.keys())}"
            )

        return super().__getitem__(modality)  # type: ignore[return-value]
```
**EN:** Method `MultiModalKwargsItems.__getitem__` provides a reusable helper around the module's main workflow. Key calls such as `KeyError`, `set`, `self.keys`, `super().__getitem__`, `super` show the concrete execution path.
**CN:** Method `MultiModalKwargsItems.__getitem__` 为模块主流程提供可复用的辅助逻辑。 像 `KeyError`, `set`, `self.keys`, `super().__getitem__`, `super` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalKwargsItems.require_data` (lines 963-969)
```python
    def require_data(self) -> "MultiModalKwargsItems[MultiModalKwargsItem]":
        for modality, items in self.items():
            for i, item in enumerate(items):
                if item is None:
                    raise RuntimeError(f"Found empty mm_items[{modality}][{i}]")

        return self  # type: ignore[return-value]
```
**EN:** Method `MultiModalKwargsItems.require_data` provides a reusable helper around the module's main workflow. Key calls such as `self.items`, `enumerate`, `RuntimeError` show the concrete execution path.
**CN:** Method `MultiModalKwargsItems.require_data` 为模块主流程提供可复用的辅助逻辑。 像 `self.items`, `enumerate`, `RuntimeError` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalKwargsItems.get_data` (lines 971-1009)
```python
    def get_data(
        self,
        *,
        device: torch.types.Device = None,
        pin_memory: bool = False,
    ) -> BatchedTensorInputs:
        """Construct a dictionary of keyword arguments to pass to the model."""
        from .utils import group_and_batch_mm_items

        items_by_modality = self.require_data()
        batches_by_modality = {
            modality: [
                data
                for _, data in group_and_batch_mm_items(
                    items,
                    device=device,
                    pin_memory=pin_memory,
                )
            ]
    # ... omitted for brevity ...

        return out_data
```
**EN:** Method `MultiModalKwargsItems.get_data` provides a reusable helper around the module's main workflow. The docstring highlights: Construct a dictionary of keyword arguments to pass to the model. Key calls such as `self.require_data`, `group_and_batch_mm_items`, `items_by_modality.items`, `len`, `batches_by_modality.items` show the concrete execution path.
**CN:** Method `MultiModalKwargsItems.get_data` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Construct a dictionary of keyword arguments to pass to the model. 像 `self.require_data`, `group_and_batch_mm_items`, `items_by_modality.items`, `len`, `batches_by_modality.items` 这样的关键调用展示了该代码块的具体执行路径。

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
- **Standard library / 标准库**: `from abc import ABC, abstractmethod`, `from collections import UserDict, defaultdict`, `from collections.abc import Mapping, Sequence`, `from dataclasses import dataclass`, `from functools import cached_property, partial`, `from itertools import accumulate`, `from typing import TYPE_CHECKING, Any, Literal, TypeAlias, TypedDict, Union, cast`
- **Third-party / 第三方**: `import numpy as np`, `from PIL.Image import Image`, `from typing_extensions import TypeVar`
- **vLLM internal / vLLM 内部依赖**: `from vllm.utils.collection_utils import is_list_of`, `from vllm.utils.import_utils import LazyLoader`, `from vllm.utils.jsontree import json_map_leaves`, `from .media import MediaWithBytes`
