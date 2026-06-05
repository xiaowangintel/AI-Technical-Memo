# hyperclovax_vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/hyperclovax_vision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for hyperclovax_vision, including encoder/decoder glue and vLLM runtime adaptation. / 面向 hyperclovax_vision 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-48)
```python
import ast
from collections import defaultdict
from collections.abc import Iterable, Mapping, Sequence
from functools import partial
from itertools import accumulate
from typing import Annotated, Literal

import numpy as np
import torch
import torch.nn as nn
from einops import rearrange
from timm.layers import LayerNorm, LayerNorm2d
from timm.models.regnet import RegStage
from transformers import BatchFeature, CLIPVisionConfig, SiglipVisionConfig

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import ImageSize, MultiModalDataItems
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .clip import CLIPVisionModel
from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .siglip import SiglipVisionModel
from .utils import (
    AutoWeightsLoader,
    flatten_bn,
    init_vllm_registered_model,
    maybe_prefix,
)
from .vision import get_vision_encoder_info
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 50-50)
```python
IMAGE_TOKEN: str = "<|dummy3|>"
```
**EN:** This block defines IMAGE_TOKEN, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 IMAGE_TOKEN，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 51-51)
```python
VIDEO_TOKEN: str = "<|_unuse_missing_100270|>"
```
**EN:** This block defines VIDEO_TOKEN, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 VIDEO_TOKEN，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Function `get_num_combined_frames` (lines 56-66)
```python
def get_num_combined_frames(
    num_frames: int,
    max_grid_shape: tuple[int, int] = (3, 3),
) -> int:
    max_num_grids = max_grid_shape[0] * max_grid_shape[1]

    # Calculate the number of canvases needed.
    num_canvases = num_frames // max_num_grids
    leftover_frames = num_frames % max_num_grids

    return num_canvases + (leftover_frames > 0)
```
**EN:** Function `get_num_combined_frames` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `get_num_combined_frames` 封装了该模块中的一段可复用核心逻辑。

### Function `init_vision_tower_for_hcxvision` (lines 337-371)
```python
def init_vision_tower_for_hcxvision(
    vision_config,
    quant_config: QuantizationConfig | None,
    *,
    use_nth_layer: int | None = None,
    require_post_norm: bool | None = None,
    prefix: str = "",
) -> CLIPVisionModel | SiglipVisionModel:
    num_hidden_layers = vision_config.num_hidden_layers
    if not isinstance(use_nth_layer, int):
        pass
    elif use_nth_layer >= 0:
        num_hidden_layers = use_nth_layer + 1
    else:
        num_hidden_layers = num_hidden_layers + use_nth_layer + 1

    if isinstance(vision_config, CLIPVisionConfig):
        return CLIPVisionModel(
            vision_config,
            quant_config=quant_config,
            num_hidden_layers_override=num_hidden_layers,
            require_post_norm=require_post_norm,
            prefix=prefix,
        )
    elif isinstance(vision_config, SiglipVisionConfig):
        return SiglipVisionModel(
            vision_config,
            quant_config=quant_config,
            num_hidden_layers_override=num_hidden_layers,
            require_post_norm=require_post_norm,
            prefix=prefix,
        )

    msg = f"Unsupported vision config: {type(vision_config)}"
    raise NotImplementedError(msg)
```
**EN:** Function `init_vision_tower_for_hcxvision` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `init_vision_tower_for_hcxvision` 封装了该模块中的一段可复用核心逻辑。

### Function `unpad_image` (lines 993-1011)
```python
def unpad_image(tensor: torch.Tensor, original_size: tuple[int, int]) -> torch.Tensor:
    original_width, original_height = original_size
    current_height, current_width = tensor.shape[1:]

    original_aspect_ratio = original_width / original_height
    current_aspect_ratio = current_width / current_height

    if original_aspect_ratio > current_aspect_ratio:
        scale_factor = current_width / original_width
        new_height = int(original_height * scale_factor)
        padding = (current_height - new_height) // 2
        unpadded_tensor = tensor[:, padding : current_height - padding, :]
    else:
        scale_factor = current_height / original_height
        new_width = int(original_width * scale_factor)
        padding = (current_width - new_width) // 2
        unpadded_tensor = tensor[:, :, padding : current_width - padding]

    return unpadded_tensor
```
**EN:** Function `unpad_image` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `unpad_image` 封装了该模块中的一段可复用核心逻辑。

### Function `select_best_resolution` (lines 1014-1039)
```python
def select_best_resolution(original_size: tuple, possible_resolutions: list) -> tuple:
    original_height, original_width = original_size
    best_fit = None
    max_effective_resolution = 0
    min_wasted_resolution = float("inf")

    for height, width in possible_resolutions:
        scale = min(width / original_width, height / original_height)
        downscaled_width, downscaled_height = (
            int(original_width * scale),
            int(original_height * scale),
        )
        effective_resolution = min(
            downscaled_width * downscaled_height, original_width * original_height
        )
        wasted_resolution = (width * height) - effective_resolution

        if effective_resolution > max_effective_resolution or (
            effective_resolution == max_effective_resolution
            and wasted_resolution < min_wasted_resolution
        ):
            max_effective_resolution = effective_resolution
            min_wasted_resolution = wasted_resolution
            best_fit = (height, width)

    return best_fit
```
**EN:** Function `select_best_resolution` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `select_best_resolution` 封装了该模块中的一段可复用核心逻辑。

### Function `get_anyres_image_grid_shape` (lines 1042-1057)
```python
def get_anyres_image_grid_shape(
    image_size: tuple[int, int],
    grid_pinpoints: str | list[tuple[int, int]],
    patch_size: int,
) -> tuple[int, int]:
    possible_resolutions = (
        grid_pinpoints
        if isinstance(grid_pinpoints, list)
        else ast.literal_eval(grid_pinpoints)
    )

    original_width, original_height = image_size
    height, width = select_best_resolution(
        (original_height, original_width), possible_resolutions
    )
    return width // patch_size, height // patch_size
```
**EN:** Function `get_anyres_image_grid_shape` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `get_anyres_image_grid_shape` 封装了该模块中的一段可复用核心逻辑。

### Function `reshape_and_unpad_image_features` (lines 1060-1103)
```python
def reshape_and_unpad_image_features(
    image_feature: torch.Tensor,
    height: int,
    width: int,
    image_size: tuple[int, int],
    possible_resolutions: list[tuple[int, int]],
    grid_size: int,
    unpad: bool,
    image_newline: torch.Tensor,
) -> torch.Tensor:
    base_image_feature = image_feature[0]
    image_feature = image_feature[1:]

    assert height * width == base_image_feature.shape[0], (
        f"{height=} * {width=} != {base_image_feature.shape[0]=}"
    )

    num_patch_width, num_patch_height = get_anyres_image_grid_shape(
        image_size, possible_resolutions, grid_size
    )
    image_feature = image_feature.view(
        num_patch_height, num_patch_width, height, width, -1
    )

    if unpad:
        image_feature = image_feature.permute(4, 0, 2, 1, 3).contiguous()
        image_feature = image_feature.flatten(1, 2).flatten(2, 3)
        image_feature = unpad_image(image_feature, image_size)
        image_feature = torch.cat(
            (
                image_feature,
                image_newline[:, None, None]
                .expand(*image_feature.shape[:-1], 1)
                .to(image_feature.device),
            ),
            dim=-1,
        )
        image_feature = image_feature.flatten(1, 2).transpose(0, 1)
    else:
        image_feature = image_feature.permute(0, 2, 1, 3, 4).contiguous()
        image_feature = image_feature.flatten(0, 3)
    image_feature = torch.cat((base_image_feature, image_feature), dim=0)

    return image_feature
```
**EN:** Function `reshape_and_unpad_image_features` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `reshape_and_unpad_image_features` 封装了该模块中的一段可复用核心逻辑。

### Class `HCXVisionImagePixelInputs` (lines 69-83)
```python
class HCXVisionImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - n: Number of images
        - g: Number of grids
        - c: Number of channels (3)
        - h: Height
        - w: Width
    """

    type: Literal["pixel_values"] = "pixel_values"
    pixel_values_images: Annotated[
        list[torch.Tensor], TensorShape("n", "g", 3, "h", "w", dynamic_dims={"g"})
    ]
    image_sizes_images: Annotated[torch.Tensor, TensorShape("n", 2)]
```
**EN:** Class `HCXVisionImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `HCXVisionImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `HCXVisionVideoPixelInputs` (lines 89-104)
```python
class HCXVisionVideoPixelInputs(TensorSchema):
    """
    Dimensions:
        - n: Number of videos
        - f: Number of frames
        - g: Number of grids
        - c: Number of channels (3)
        - h: Height
        - w: Width
    """

    type: Literal["pixel_values_videos"] = "pixel_values_videos"
    pixel_values_videos: Annotated[
        list[list[torch.Tensor]],
        TensorShape("n", "f", "g", 3, "h", "w", dynamic_dims={"f", "g"}),
    ]
```
**EN:** Class `HCXVisionVideoPixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `HCXVisionVideoPixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `HCXVisionProcessingInfo` (lines 110-148)
```python
class HCXVisionProcessingInfo(BaseProcessingInfo):
    def get_vision_encoder_info(self):
        return get_vision_encoder_info(self.get_hf_config())

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None, "video": None}

    def get_num_image_tokens(
        self,
        *,
        vision_query_length: int | list[int],
    ) -> int:
        if isinstance(vision_query_length, int):
            return vision_query_length
        else:
            return sum(vision_query_length)

    def get_num_video_tokens(
        self,
        *,
        vision_query_length: int | list[int],
    ) -> int:
        if isinstance(vision_query_length, int):
            return vision_query_length
        else:
```
**EN:** Class `HCXVisionProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_vision_encoder_info, get_supported_mm_limits, get_num_image_tokens, get_num_video_tokens, get_image_size_with_most_features, get_max_image_tokens.
**CN:** 类 `HCXVisionProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_vision_encoder_info, get_supported_mm_limits, get_num_image_tokens, get_num_video_tokens, get_image_size_with_most_features, get_max_image_tokens。

### Method `HCXVisionProcessingInfo.get_vision_encoder_info` (lines 111-112)
```python
    def get_vision_encoder_info(self):
        return get_vision_encoder_info(self.get_hf_config())
```
**EN:** Method `HCXVisionProcessingInfo.get_vision_encoder_info` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionProcessingInfo.get_vision_encoder_info` 封装了该模块中的一段可复用核心逻辑。

### Method `HCXVisionProcessingInfo.get_supported_mm_limits` (lines 114-115)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None, "video": None}
```
**EN:** Method `HCXVisionProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Class `HCXVisionDummyInputsBuilder` (lines 151-190)
```python
class HCXVisionDummyInputsBuilder(BaseDummyInputsBuilder[HCXVisionProcessingInfo]):
    def get_dummy_text(
        self,
        mm_counts: Mapping[str, int],
    ) -> str:
        dummy_text = IMAGE_TOKEN * mm_counts.get(
            "image", 0
        ) + VIDEO_TOKEN * mm_counts.get("video", 0)
        return dummy_text

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)

        target_width, target_height = self.info.get_image_size_with_most_features()
        target_num_frames = 32

        image_overrides = mm_options.get("image")
        video_overrides = mm_options.get("video")
```
**EN:** Class `HCXVisionDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[HCXVisionProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `HCXVisionDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[HCXVisionProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `HCXVisionDummyInputsBuilder.get_dummy_text` (lines 152-159)
```python
    def get_dummy_text(
        self,
        mm_counts: Mapping[str, int],
    ) -> str:
        dummy_text = IMAGE_TOKEN * mm_counts.get(
            "image", 0
        ) + VIDEO_TOKEN * mm_counts.get("video", 0)
        return dummy_text
```
**EN:** Method `HCXVisionDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `HCXVisionDummyInputsBuilder.get_dummy_mm_data` (lines 161-190)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)

        target_width, target_height = self.info.get_image_size_with_most_features()
        target_num_frames = 32

        image_overrides = mm_options.get("image")
        video_overrides = mm_options.get("video")

        return {
            "image": self._get_dummy_images(
                width=target_width,
                height=target_height,
                num_images=num_images,
                overrides=image_overrides,
            ),
            "video": self._get_dummy_videos(
                width=target_width - 1,
                height=target_height - 1,
                num_frames=target_num_frames,
                num_videos=num_videos,
                overrides=video_overrides,
            ),
        }
```
**EN:** Method `HCXVisionDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `HCXVisionMultiModalProcessor` (lines 193-334)
```python
class HCXVisionMultiModalProcessor(BaseMultiModalProcessor[HCXVisionProcessingInfo]):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        for video_idx, video_arr in enumerate(mm_data.get("videos", [])):
            if video_arr.dtype != np.uint8:
                mm_data["videos"][video_idx] = video_arr.astype(np.uint8)

        processed_outputs = self.info.ctx.call_hf_processor(
            hf_processor=self.info.get_hf_processor(**mm_kwargs),
            data=dict(
                text=prompt,
                images=None,
                videos=None,
            ),
        )  # text-only

        if len(mm_data) > 0:
            images = mm_data.get("images")
            videos = mm_data.get("videos")
```
**EN:** Class `HCXVisionMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[HCXVisionProcessingInfo]. Key methods include _call_hf_processor, _hf_processor_applies_updates, _get_prompt_updates, _get_mm_fields_config.
**CN:** 类 `HCXVisionMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[HCXVisionProcessingInfo]。 关键方法包括 _call_hf_processor, _hf_processor_applies_updates, _get_prompt_updates, _get_mm_fields_config。

### Class `HCXVisionMlp` (lines 374-404)
```python
class HCXVisionMlp(nn.Module):
    def __init__(
        self,
        mm_projector_type,
        in_features,
        hidden_features=None,
        out_features=None,
        act_layer=nn.GELU,
    ):
        super().__init__()
        out_features = out_features or in_features
        hidden_features = hidden_features or in_features
        self.mm_projector_type = mm_projector_type
        if self.mm_projector_type == "mlp":
            self.fc1 = nn.Linear(in_features, hidden_features)
            self.act = act_layer()
            self.fc2 = nn.Linear(hidden_features, out_features)
        elif self.mm_projector_type == "inverted_mlp":
            self.fc1 = nn.Linear(in_features, 2 * hidden_features)
            self.act = act_layer()
            self.fc2 = nn.Linear(2 * hidden_features, out_features)
        else:
            raise NotImplementedError(
                "{} is not implemented".format(self.mm_projector_type)
            )
```
**EN:** Class `HCXVisionMlp` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `HCXVisionMlp` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `HCXVisionMlp.__init__` (lines 375-398)
```python
    def __init__(
        self,
        mm_projector_type,
        in_features,
        hidden_features=None,
        out_features=None,
        act_layer=nn.GELU,
    ):
        super().__init__()
        out_features = out_features or in_features
        hidden_features = hidden_features or in_features
        self.mm_projector_type = mm_projector_type
        if self.mm_projector_type == "mlp":
            self.fc1 = nn.Linear(in_features, hidden_features)
            self.act = act_layer()
            self.fc2 = nn.Linear(hidden_features, out_features)
        elif self.mm_projector_type == "inverted_mlp":
            self.fc1 = nn.Linear(in_features, 2 * hidden_features)
            self.act = act_layer()
            self.fc2 = nn.Linear(2 * hidden_features, out_features)
        else:
            raise NotImplementedError(
                "{} is not implemented".format(self.mm_projector_type)
            )
```
**EN:** Method `HCXVisionMlp.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `HCXVisionMlp.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `HCXVisionMlp.forward` (lines 400-404)
```python
    def forward(self, x):
        x = self.fc1(x)
        x = self.act(x)
        x = self.fc2(x)
        return x
```
**EN:** Method `HCXVisionMlp.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `HCXVisionMlp.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `HCXVisionCAbstractor` (lines 407-562)
```python
class HCXVisionCAbstractor(nn.Module):
    """
    This module is based on C-Abstractor, whose license is under apache-2.0.
    You can check the original code at
    https://github.com/khanrc/honeybee/blob/main/honeybee/projectors/projectors.py
    and we made necessary modifications.
    """

    def __init__(
        self,
        num_queries: int,
        num_input_tokens: int,
        encoder_hidden_size: int,
        hidden_size: int,
        output_hidden_size: int,
        pos_emb: bool = True,
        prenorm: bool = False,
    ):
        super().__init__()
        self.num_input_tokens = num_input_tokens
        self.output_hidden_size = output_hidden_size

        # Positional embedding
        if pos_emb:
            self.pos_emb = torch.nn.Parameter(
```
**EN:** Class `HCXVisionCAbstractor` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward, _forward, _forward_adaptive_num_query, build_net, build_mlp.
**CN:** 类 `HCXVisionCAbstractor` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward, _forward, _forward_adaptive_num_query, build_net, build_mlp。

### Method `HCXVisionCAbstractor.__init__` (lines 415-447)
```python
    def __init__(
        self,
        num_queries: int,
        num_input_tokens: int,
        encoder_hidden_size: int,
        hidden_size: int,
        output_hidden_size: int,
        pos_emb: bool = True,
        prenorm: bool = False,
    ):
        super().__init__()
        self.num_input_tokens = num_input_tokens
        self.output_hidden_size = output_hidden_size

        # Positional embedding
        if pos_emb:
            self.pos_emb = torch.nn.Parameter(
                torch.zeros(1, num_input_tokens, encoder_hidden_size)
            )
            self.pos_emb.data.normal_(mean=0.0, std=0.02)
        else:
            self.pos_emb = None

        # (Optional) Pre-normalization layer
        if prenorm:
            self.prenorm = LayerNorm(encoder_hidden_size)
        else:
            self.prenorm = None

        self.build_net(
            num_queries, encoder_hidden_size, hidden_size, output_hidden_size
        )
        self.dtype = next(self.parameters()).dtype
```
**EN:** Method `HCXVisionCAbstractor.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `HCXVisionCAbstractor.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `HCXVisionCAbstractor.forward` (lines 449-467)
```python
    def forward(
        self,
        x: torch.Tensor,
        num_queries_vis_abstractors: list[list[int]] | None = None,
        num_grids: list[int] | None = None,
    ) -> torch.Tensor:
        if self.prenorm is not None:
            x = self.prenorm(x)

        if self.pos_emb is not None:
            x = x + self.pos_emb

        x = self._forward(
            x,
            num_queries_vis_abstractors=num_queries_vis_abstractors,
            num_grids=num_grids,
        )  # (B, L, output_hidden_size)

        return x
```
**EN:** Method `HCXVisionCAbstractor.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `HCXVisionCAbstractor.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `HCXVisionForCausalLM` (lines 570-990)
```python
@MULTIMODAL_REGISTRY.register_processor(
    HCXVisionMultiModalProcessor,
    info=HCXVisionProcessingInfo,
    dummy_inputs=HCXVisionDummyInputsBuilder,
)
class HCXVisionForCausalLM(nn.Module, SupportsMultiModal, SupportsPP):
    """
    HyperCLOVAX-SEED Vision-Language Model (V1 architecture).

    Supports:
    - HyperCLOVAX-SEED-Vision-Instruct-3B

    Uses CLIP/SigLIP as the vision encoder with C-Abstractor projector.
    """

    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }

    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
```
**EN:** Class `HCXVisionForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods include __init__, get_placeholder_str, _parse_and_validate_image_input, _parse_and_validate_video_input, _process_image_input, _process_video_input.
**CN:** 类 `HCXVisionForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 关键方法包括 __init__, get_placeholder_str, _parse_and_validate_image_input, _parse_and_validate_video_input, _process_image_input, _process_video_input。

### Method `HCXVisionForCausalLM.__init__` (lines 585-644)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
    ) -> None:
        super().__init__()

        # init configs
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        # text_config
        text_config = config.text_config
        if text_config.model_type in ["gpt2", "hyperclovax", "llama"]:
            text_config._attn_implementation = "sdpa"
        if text_config.model_type != "hyperclovax":
            text_config.logits_scaling = 1.0
        # vision_config
        vision_config = config.vision_config
        vision_config.auto_map = {}
        vision_config.anyres = config.anyres
        vision_config.max_num_grids = config.max_num_grids
        self.dtype = vllm_config.model_config.dtype

        ## possible_resolution should be matched with preprocessor_config.json
        config.possible_resolutions = self._init_possible_resolutions(
            config, vision_config
        )

        with self._mark_tower_model(vllm_config, {"image", "video"}):
            self.vision_model = init_vision_tower_for_hcxvision(
                vision_config,
                quant_config=quant_config,
                use_nth_layer=getattr(config, "use_nth_layer", -1),
                require_post_norm=False,
                prefix=maybe_prefix(prefix, "vision_model"),
            )
            self.mm_projector = self._init_mm_projector(
                config, text_config, vision_config
            )

            if config.anyres:
                self.image_newline = nn.Parameter(
                    torch.empty(text_config.hidden_size, dtype=self.dtype)
                )

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=text_config,
                prefix=maybe_prefix(prefix, "language_model"),
            )

        self.config = config
        self.vision_config = vision_config
        self.text_config = text_config

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )
```
**EN:** Method `HCXVisionForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `HCXVisionForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `HCXVisionForCausalLM.get_placeholder_str` (lines 647-653)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return IMAGE_TOKEN
        if modality.startswith("video"):
            return VIDEO_TOKEN

        raise ValueError("Only image or video modality is supported")
```
**EN:** Method `HCXVisionForCausalLM.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `HCXVisionForCausalLM.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import ast`, `from collections import defaultdict`, `from collections.abc import Iterable, Mapping, Sequence`, `from functools import partial`, `from itertools import accumulate`, `from typing import Annotated, Literal`, `from einops import rearrange`, `from timm.layers import LayerNorm, LayerNorm2d`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `import torch.nn as nn`, `from transformers import BatchFeature, CLIPVisionConfig, SiglipVisionConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import ImageSize, MultiModalDataItems`, `from vllm.multimodal.processing import (`, `from vllm.sequence import IntermediateTensors`, `from vllm.utils.tensor_schema import TensorSchema, TensorShape`, `from .clip import CLIPVisionModel`, `from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP`
