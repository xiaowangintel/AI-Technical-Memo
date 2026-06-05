# evs_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/evs/evs_processor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements EVS-related multimodal components for evs processor, covering model glue code and runtime integration. / 该模块实现与 evs processor 相关的 EVS 多模态组件，涵盖模型衔接代码与运行时集成。

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

### Lines 16-23: Imports dependencies
```python
import torch
from transformers import PretrainedConfig

from sglang.srt.managers.schedule_batch import Modality, MultimodalDataItem
from sglang.utils import logger

from .evs_core import tokens_per_frame
from .evs_module import EVS, EVSConfig, EVSDataItem, VideoEVSDataItem
```
**EN:** This block groups related imports for the module, including torch, transformers.PretrainedConfig, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem, sglang.utils.logger, and 5 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 torch, transformers.PretrainedConfig, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem, sglang.utils.logger 等 5 项，为后续代码准备所需名称。

### Lines 24-25: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 26-45: Defines function _non_evs_data_items
```python
def _non_evs_data_items(
    *,
    image: torch.Tensor | None,
    image_offsets: list[tuple[int, int]],
    video: torch.Tensor | None,
    video_offsets: list[tuple[int, int]],
    input_ids_list: list[int],
):
    items: list[MultimodalDataItem] = []
    if image is not None:
        item = MultimodalDataItem(
            modality=Modality.IMAGE, feature=image, offsets=image_offsets
        )
        items.append(item)
    if video is not None:
        item = MultimodalDataItem(
            modality=Modality.VIDEO, feature=video, offsets=video_offsets
        )
        items.append(item)
    return items
```
**EN:** This block defines function `_non_evs_data_items`. It takes no explicit parameters.
**CN:** 该代码块定义函数 `_non_evs_data_items`。 它没有显式参数。

### Lines 46-47: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 48-48: Declares class EVSProcessor
```python
class EVSProcessor:
```
**EN:** This block introduces class `EVSProcessor` as a reusable abstraction inside the module. This processor handles prompt construction with the correct number of placeholder tokens per frame. When EVS is active, it allocates fewer placeholders based on the pruning rate.
**CN:** 该代码块声明类 `EVSProcessor`，作为模块中的可复用抽象。 文档字符串摘要：This processor handles prompt construction with the correct number of placeholder tokens per frame. When EVS is active, it allocates fewer placeholders based on the pruning rate.

### Lines 49-54: Documents the scope
```python
    """
    This processor handles prompt construction with the correct number of
    placeholder tokens per frame. When EVS is active, it allocates fewer
    placeholders based on the pruning rate. When inactive, it uses the full
    token count.
    """
```
**EN:** This string literal serves as documentation for the EVSProcessor, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 EVSProcessor 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 55-55: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the EVSProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 EVSProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 56-78: Defines function EVSProcessor.__init__
```python
    def __init__(
        self,
        hf_config: PretrainedConfig,
        config_to_evs_model: dict[type[PretrainedConfig], type[EVS]],
    ):
        assert len(config_to_evs_model) > 0
        assert all(issubclass(model, EVS) for model in config_to_evs_model.values())

        self.evs_config: EVSConfig | None = None

        config_name = hf_config.__class__.__name__
        evs_model = config_to_evs_model.get(hf_config.__class__)
        if evs_model is None:
            logger.info(
                f"[EVS] no model matches {config_name} in {config_to_evs_model}"
            )
            return
        evs_config = evs_model.create_evs_config(hf_config)
        logger.info(
            f"""[EVS] {evs_config} {'enabled' if evs_config.video_pruning_rate > 0.0 else 'disabled'} for model={evs_model.__name__}; model_config={config_name}"""
        )
        if evs_config.video_pruning_rate > 0.0:
            self.evs_config = evs_config
```
**EN:** This block defines function `EVSProcessor.__init__`. Parameters: self, hf_config, config_to_evs_model.
**CN:** 该代码块定义函数 `EVSProcessor.__init__`。 参数包括 self、hf_config、config_to_evs_model。

### Lines 79-79: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the EVSProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 EVSProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 80-132: Defines function EVSProcessor.static_size_data_items
```python
    def static_size_data_items(
        self, *, frames_per_video: list[int], num_images: int, rows: int, cols: int
    ):
        """helper function to create data items for models with static image and video tokens per frame"""

        frame_num_tokens = rows * cols

        if self.evs_config is None:
            tpf = [[frame_num_tokens] * num_frames for num_frames in frames_per_video]
            return _non_evs_data_items, tpf

        def create_evs_data_items(
            *,
            input_ids_list: list[int],
            image: torch.Tensor | None,
            image_offsets: list[tuple[int, int]],
            video: torch.Tensor | None,
            video_offsets: list[tuple[int, int]],
        ) -> list[MultimodalDataItem]:
            items = []
            if image is not None:
                image_thw_grids = [(1, rows, cols)] * num_images
                item = EVSDataItem(
                    modality=Modality.IMAGE,
                    feature=image,
                    offsets=image_offsets,
                    thw_grids=image_thw_grids,
                )
                items.append(item)
            if video is not None:
                video_thw_grids = [
                    (num_frames, rows, cols) for num_frames in frames_per_video
                ]
                item = VideoEVSDataItem(
                    modality=Modality.VIDEO,
                    feature=video,
                    offsets=video_offsets,
                    thw_grids=video_thw_grids,
                    pre_chunked_input_ids=input_ids_list,
                )
                items.append(item)
            return items

        tpf = [
            tokens_per_frame(
                q=self.evs_config.video_pruning_rate,
                num_frames=num_frames,
                frame_num_tokens=frame_num_tokens,
            )
            for num_frames in frames_per_video
        ]

        return create_evs_data_items, tpf
```
**EN:** This block defines function `EVSProcessor.static_size_data_items`. Parameters: self. helper function to create data items for models with static image and video tokens per frame
**CN:** 该代码块定义函数 `EVSProcessor.static_size_data_items`。 参数包括 self。 文档字符串摘要：helper function to create data items for models with static image and video tokens per frame

## Key Concepts / 关键概念
- **Classes / 类**: `EVSProcessor`
- **Functions / 函数**: `_non_evs_data_items`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `evs_core`, `evs_module`, `torch`, `transformers`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.utils`
