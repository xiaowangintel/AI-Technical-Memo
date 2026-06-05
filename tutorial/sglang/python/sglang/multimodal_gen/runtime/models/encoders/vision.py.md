# vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/encoders/vision.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for vision within the multimodal runtime. Key symbols include `VisionEncoderInfo`, `resolve_visual_encoder_outputs`. / 该模块实现多模态运行时中与 vision 相关的模型构件。 关键符号包括 `VisionEncoderInfo`, `resolve_visual_encoder_outputs`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Adapted from vllm: https://github.com/vllm-project/vllm/blob/v0.7.3/vllm/model_executor/models/vision.py

from abc import ABC, abstractmethod
from typing import Generic, TypeVar

import torch
from transformers import PretrainedConfig

from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)

_C = TypeVar("_C", bound=PretrainedConfig)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 19-49: Class `VisionEncoderInfo` / 类 `VisionEncoderInfo`
```python
class VisionEncoderInfo(ABC, Generic[_C]):

    def __init__(self, vision_config: _C) -> None:
        super().__init__()

        self.vision_config = vision_config

    @abstractmethod
    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
# ...

    @abstractmethod
    def get_patch_grid_length(self) -> int:
        raise NotImplementedError
```
**EN:** This class models `VisionEncoderInfo` as a specialization of `ABC`, `Generic[_C]`. Important methods include `__init__`, `get_num_image_tokens`, `get_max_image_tokens`, `get_image_size`.
**CN:** 该类实现 `VisionEncoderInfo`，并继承/扩展 `ABC`, `Generic[_C]`。 其中较重要的方法包括 `__init__`, `get_num_image_tokens`, `get_max_image_tokens`, `get_image_size`。

### Lines 52-96: Function `resolve_visual_encoder_outputs` / 函数 `resolve_visual_encoder_outputs`
```python
def resolve_visual_encoder_outputs(
    encoder_outputs: torch.Tensor | list[torch.Tensor],
    feature_sample_layers: list[int] | None,
    post_layer_norm: torch.nn.LayerNorm | None,
    max_possible_layers: int,
) -> torch.Tensor:
    """Given the outputs a visual encoder module that may correspond to the
    output of the last layer, or a list of hidden states to be stacked,
    handle post normalization and resolve it into a single output tensor.

    Args:
        encoder_outputs: Output of encoder's last layer or all hidden states.
        feature_sample_layers: Optional layer indices to grab from the encoder
            outputs; if provided, encoder outputs must be a list.
# ...
    uses_last_layer = feature_sample_layers[-1] in (len(hs_pool) - 1, -1)
    if post_layer_norm is not None and uses_last_layer:
        hs_pool[-1] = post_layer_norm(encoder_outputs)
    return torch.cat(hs_pool, dim=-1)
```
**EN:** This function drives `resolve_visual_encoder_outputs` with inputs such as `encoder_outputs`, `feature_sample_layers`, `post_layer_norm`, `max_possible_layers`. Given the outputs a visual encoder module that may correspond to the
**CN:** 这个函数负责 `resolve_visual_encoder_outputs`，主要处理 `encoder_outputs`, `feature_sample_layers`, `post_layer_norm`, `max_possible_layers` 等输入。 文档字符串说明：Given the outputs a visual encoder module that may correspond to the

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Image generation flow / 图像生成流程
- Video generation flow / 视频生成流程
- Symbol `VisionEncoderInfo` anchors the module API / 符号 `VisionEncoderInfo` 构成该模块的核心 API
- Symbol `resolve_visual_encoder_outputs` anchors the module API / 符号 `resolve_visual_encoder_outputs` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.utils.logging_utils`
- **External / 外部**: `torch`, `transformers`
- **Stdlib / 标准库**: `abc`, `typing`
