# evs_module.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/evs/evs_module.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements EVS-related multimodal components for evs module, covering model glue code and runtime integration. / 该模块实现与 evs module 相关的 EVS 多模态组件，涵盖模型衔接代码与运行时集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15: Comments and module notes
```python
# Copyright 2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================


```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 16-29: Imports dependencies
```python
import dataclasses
import typing
from abc import ABC, abstractmethod
from dataclasses import dataclass

import torch
from transformers import PretrainedConfig

from sglang.srt.managers.schedule_batch import MultimodalDataItem
from sglang.srt.mem_cache.multimodal_cache import EmbeddingResult
from sglang.srt.multimodal.processors.base_processor import BaseMultimodalProcessor
from sglang.utils import logger

from .evs_core import compute_retention_mask, replace_offsets_with_tokens_per_frame
```
**EN:** This block groups related imports for the module, including dataclasses, typing, abc.ABC, abc.abstractmethod, dataclasses.dataclass, and 8 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 dataclasses, typing, abc.ABC, abc.abstractmethod, dataclasses.dataclass 等 8 项，为后续代码准备所需名称。

### Lines 30-31: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 32-33: Declares class EVSDataItem
```python
@dataclasses.dataclass(kw_only=True)
class EVSDataItem(MultimodalDataItem):
```
**EN:** This block introduces class `EVSDataItem` as a reusable abstraction inside the module. It inherits from MultimodalDataItem.
**CN:** 该代码块声明类 `EVSDataItem`，作为模块中的可复用抽象。 它继承自 MultimodalDataItem。

### Lines 34-34: Declares thw_grids
```python
    thw_grids: list[tuple[int, int, int]]
```
**EN:** This statement initializes thw_grids in the EVSDataItem. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 EVSDataItem 中初始化 thw_grids。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 35-36: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 37-38: Declares class VideoEVSDataItem
```python
@dataclasses.dataclass(kw_only=True)
class VideoEVSDataItem(EVSDataItem):
```
**EN:** This block introduces class `VideoEVSDataItem` as a reusable abstraction inside the module. It inherits from EVSDataItem.
**CN:** 该代码块声明类 `VideoEVSDataItem`，作为模块中的可复用抽象。 它继承自 EVSDataItem。

### Lines 39-39: Declares pre_chunked_input_ids
```python
    pre_chunked_input_ids: torch.Tensor
```
**EN:** This statement initializes pre_chunked_input_ids in the VideoEVSDataItem. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 VideoEVSDataItem 中初始化 pre_chunked_input_ids。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 40-40: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the VideoEVSDataItem, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 VideoEVSDataItem 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 41-42: Defines function VideoEVSDataItem.__post_init__
```python
    def __post_init__(self):
        assert self.is_video()
```
**EN:** This block defines function `VideoEVSDataItem.__post_init__`. Parameters: self.
**CN:** 该代码块定义函数 `VideoEVSDataItem.__post_init__`。 参数包括 self。

### Lines 43-44: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 45-46: Declares class EVSEmbeddingResult
```python
@dataclass(kw_only=True)
class EVSEmbeddingResult(EmbeddingResult):
```
**EN:** This block introduces class `EVSEmbeddingResult` as a reusable abstraction inside the module. It inherits from EmbeddingResult. Embedding result that includes per-frame token counts after EVS pruning. After pruning, each frame retains a different number of tokens based on its dissimilarity to the previous frame.
**CN:** 该代码块声明类 `EVSEmbeddingResult`，作为模块中的可复用抽象。 它继承自 EmbeddingResult。 文档字符串摘要：Embedding result that includes per-frame token counts after EVS pruning. After pruning, each frame retains a different number of tokens based on its dissimilarity to the previous frame.

### Lines 47-59: Documents the scope
```python
    """
    Embedding result that includes per-frame token counts after EVS pruning.

    After pruning, each frame retains a different number of tokens based on its
    dissimilarity to the previous frame. This metadata is needed downstream to
    adjust the input_ids placeholder spans to match the actual embedding sizes.

    Attributes:
        embedding: The pruned video embeddings tensor.
        num_tokens_per_frame: Actual retained token count for each frame.
            For example, [256, 180, 195, 256] means frame 0 kept all 256 tokens
            (first frame is never pruned), while frames 1-2 were pruned.
    """
```
**EN:** This string literal serves as documentation for the EVSEmbeddingResult, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 EVSEmbeddingResult 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 60-60: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the EVSEmbeddingResult, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 EVSEmbeddingResult 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 61-61: Declares num_tokens_per_frame
```python
    num_tokens_per_frame: list[int]
```
**EN:** This statement initializes num_tokens_per_frame in the EVSEmbeddingResult. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 EVSEmbeddingResult 中初始化 num_tokens_per_frame。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 62-62: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the EVSEmbeddingResult, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 EVSEmbeddingResult 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 63-94: Defines function EVSEmbeddingResult.redistribute_pruned_frames_placeholders
```python
    def redistribute_pruned_frames_placeholders(
        self,
        input_ids: torch.Tensor,
        offsets: list[tuple[int, int]],
        *,
        item: VideoEVSDataItem,
        extend_prefix_len: int,
        extend_seq_len: int,
    ) -> tuple[torch.Tensor, list[tuple[int, int]]]:
        assert len(input_ids) == extend_seq_len
        assert isinstance(
            item, VideoEVSDataItem
        ), f"Expected VideoEVSDataItem, got {type(item)}"
        pre_chunked_input_ids = item.pre_chunked_input_ids
        filler_token_id = item.pad_value
        input_ids_list = replace_offsets_with_tokens_per_frame(
            pre_chunked_input_ids=pre_chunked_input_ids,
            num_tokens_per_frame=self.num_tokens_per_frame,
            frame_offsets_inclusive=offsets,
            filler_token_id=filler_token_id,
        )
        input_ids = torch.tensor(
            input_ids_list, dtype=input_ids.dtype, device=input_ids.device
        )
        offsets = BaseMultimodalProcessor.get_mm_items_offset(
            input_ids, filler_token_id
        )
        input_ids = input_ids[extend_prefix_len : extend_prefix_len + extend_seq_len]
        assert (
            len(input_ids) == extend_seq_len
        ), f"Input ids length changed after redistribution, got {len(input_ids)} != {extend_seq_len}"
        return input_ids, offsets
```
**EN:** This block defines function `EVSEmbeddingResult.redistribute_pruned_frames_placeholders`. Parameters: self, input_ids, offsets.
**CN:** 该代码块定义函数 `EVSEmbeddingResult.redistribute_pruned_frames_placeholders`。 参数包括 self、input_ids、offsets。

### Lines 95-96: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 97-98: Declares class EVSConfig
```python
@dataclass(frozen=True, kw_only=True)
class EVSConfig:
```
**EN:** This block introduces class `EVSConfig` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `EVSConfig`，作为模块中的可复用抽象。

### Lines 99-100: Declares video_pruning_rate, spatial_merge_size
```python
    video_pruning_rate: float
    spatial_merge_size: int = 1
```
**EN:** This block initializes a related set of values in the EVSConfig, including video_pruning_rate, spatial_merge_size. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 EVSConfig 中初始化一组相关值，包括 video_pruning_rate, spatial_merge_size。将这些赋值集中在一起有助于理解周边配置。

### Lines 101-101: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the EVSConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 EVSConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 102-105: Defines function EVSConfig.__post_init__
```python
    def __post_init__(self):
        assert (
            self.video_pruning_rate >= 0.0 and self.video_pruning_rate < 1.0
        ), f"Video pruning rate must be between 0.0 and 1.0, got {self.video_pruning_rate=}"
```
**EN:** This block defines function `EVSConfig.__post_init__`. Parameters: self.
**CN:** 该代码块定义函数 `EVSConfig.__post_init__`。 参数包括 self。

### Lines 106-107: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 108-108: Declares class EVS
```python
class EVS(torch.nn.Module, ABC):
```
**EN:** This block introduces class `EVS` as a reusable abstraction inside the module. It inherits from torch.nn.Module, ABC. Base class for video models that support EVS pruning. Subclass this alongside your model class and implement the static `create_evs_config`.
**CN:** 该代码块声明类 `EVS`，作为模块中的可复用抽象。 它继承自 torch.nn.Module、ABC。 文档字符串摘要：Base class for video models that support EVS pruning. Subclass this alongside your model class and implement the static `create_evs_config`.

### Lines 109-117: Documents the scope
```python
    """
    Base class for video models that support EVS pruning.

    Subclass this alongside your model class and implement the static `create_evs_config`.
    On initialization, if video_pruning_rate > 0, this mixin replaces the model's
    get_video_feature() method with a wrapper that applies EVS pruning.

    Example: See `NemotronH_Nano_VL_V2`
    """
```
**EN:** This string literal serves as documentation for the EVS, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 EVS 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 118-118: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the EVS, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 EVS 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 119-123: Defines function EVS.create_evs_config
```python
    @staticmethod
    @abstractmethod
    def create_evs_config(config: PretrainedConfig) -> EVSConfig:
        """Extract EVS parameters from model config. Must be implemented by subclass."""
        raise NotImplementedError
```
**EN:** This block defines function `EVS.create_evs_config`. Parameters: config. Decorators: staticmethod, abstractmethod. Extract EVS parameters from model config. Must be implemented by subclass.
**CN:** 该代码块定义函数 `EVS.create_evs_config`。 参数包括 config。 装饰器包括 staticmethod、abstractmethod。 文档字符串摘要：Extract EVS parameters from model config. Must be implemented by subclass.

### Lines 124-124: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the EVS, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 EVS 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 125-128: Defines function EVS.get_video_feature
```python
    @abstractmethod
    def get_video_feature(self, items: list[MultimodalDataItem]) -> torch.Tensor:
        """Extract EVS parameters from model config. Must be implemented by subclass."""
        raise NotImplementedError
```
**EN:** This block defines function `EVS.get_video_feature`. Parameters: self, items. Decorators: abstractmethod. Extract EVS parameters from model config. Must be implemented by subclass.
**CN:** 该代码块定义函数 `EVS.get_video_feature`。 参数包括 self、items。 装饰器包括 abstractmethod。 文档字符串摘要：Extract EVS parameters from model config. Must be implemented by subclass.

### Lines 129-129: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the EVS, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 EVS 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 130-147: Defines function EVS.__init__
```python
    def __init__(
        self,
        config: PretrainedConfig,
        *args: typing.Any,
        **kwargs: typing.Any,
    ) -> None:
        super().__init__()
        model_name = self.__class__.__name__
        self.original_get_video_feature = self.get_video_feature
        self.evs_config = self.create_evs_config(config)
        self.evs_enabled = self.evs_config.video_pruning_rate > 0.0
        if self.evs_enabled:
            logger.info(f"[EVS] enabled for {model_name} [{self.evs_config}]")
            self.get_video_feature = self.evs_video
        else:
            logger.info(
                f"[EVS] requested on model {model_name} but is disabled for pruning_rate == 0.0."
            )
```
**EN:** This block defines function `EVS.__init__`. Parameters: self, config.
**CN:** 该代码块定义函数 `EVS.__init__`。 参数包括 self、config。

### Lines 148-148: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the EVS, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 EVS 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 149-201: Defines function EVS.evs_video
```python
    def evs_video(self, items: list[MultimodalDataItem]) -> EVSEmbeddingResult:
        """
        Apply EVS pruning to video embeddings.

        Args:
            items: List containing a single VideoEVSDataItem with video features.

        Returns:
            EVSEmbeddingResult with pruned embeddings and actual token counts per frame.
        """
        logger.debug(
            f"[EVS] beginning for model {self.__class__.__name__} [evs_config={self.evs_config=}]"
        )
        assert len(items) == 1, f"Expected 1 item, got {len(items)}"
        item = items[0]
        assert isinstance(
            item, VideoEVSDataItem
        ), f"Expected VideoEVSDataItem with modality VIDEO, got {item}"

        q = self.evs_config.video_pruning_rate
        merge = self.evs_config.spatial_merge_size
        videos_features = self.original_get_video_feature([item])
        if videos_features.ndim == 3:
            videos_features = videos_features.flatten(0, 1)
        assert videos_features.ndim == 2, videos_features.ndim

        final_embeddings: list[torch.Tensor] = []
        num_tokens_per_frame: list[int] = []

        sizes = [(t * h * w // merge**2) for t, h, w in item.thw_grids]
        for single_video, video_size_thw in zip(
            videos_features.split(sizes),
            item.thw_grids,
            strict=True,
        ):
            retention_mask = compute_retention_mask(
                single_video,
                video_size_thw=video_size_thw,
                spatial_merge_size=merge,
                q=q,
            )
            preserved = single_video[retention_mask]
            final_embeddings.append(preserved)
            num_frames = video_size_thw[0]
            tokens_per_frame = (
                retention_mask.reshape(num_frames, -1).sum(dim=-1).tolist()
            )
            num_tokens_per_frame.extend(tokens_per_frame)
        final_embeddings_tensor = torch.cat(final_embeddings)
        return EVSEmbeddingResult(
            embedding=final_embeddings_tensor,
            num_tokens_per_frame=num_tokens_per_frame,
        )
```
**EN:** This block defines function `EVS.evs_video`. Parameters: self, items. Apply EVS pruning to video embeddings. Args: items: List containing a single VideoEVSDataItem with video features.
**CN:** 该代码块定义函数 `EVS.evs_video`。 参数包括 self、items。 文档字符串摘要：Apply EVS pruning to video embeddings. Args: items: List containing a single VideoEVSDataItem with video features.

## Key Concepts / 关键概念
- **Classes / 类**: `EVSDataItem`, `VideoEVSDataItem`, `EVSEmbeddingResult`, `EVSConfig`, `EVS`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `abc`, `dataclasses`, `typing`
- **Third-Party / 第三方**: `evs_core`, `torch`, `transformers`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.mem_cache.multimodal_cache`, `sglang.srt.multimodal.processors.base_processor`, `sglang.utils`
