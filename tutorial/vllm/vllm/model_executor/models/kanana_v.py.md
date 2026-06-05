# kanana_v.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/kanana_v.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for kanana_v, including architecture wrappers and weight loading logic. / 面向推理的 kanana_v vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-43)
```python
from collections.abc import Iterable, Mapping, Sequence
from functools import partial
from typing import Annotated, Literal, TypeAlias

import numpy as np
import regex as re
import torch
from einops import rearrange
from PIL import Image
from timm.layers import LayerNorm2d
from timm.layers.pos_embed import resample_abs_pos_embed
from timm.models.regnet import RegStage
from torch import nn
from transformers import BatchFeature
from transformers.modeling_outputs import BaseModelOutput
from transformers.models.qwen2_vl.configuration_qwen2_vl import Qwen2VLVisionConfig

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.logger import init_logger
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
from vllm.utils.import_utils import resolve_obj_by_qualname
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .qwen2_vl import Qwen2VisionTransformer
from .utils import AutoWeightsLoader, init_vllm_registered_model, maybe_prefix
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `build_pos_embeds` (lines 72-84)
```python
def build_pos_embeds(
    config: Qwen2VLVisionConfig,
    num_input_tokens: int,
    vision_hidden_size: int,
) -> nn.Parameter | None:
    """Build positional embeddings for the visual encoder output."""
    if config.pos_emb:
        pos_emb = nn.Parameter(torch.zeros(1, num_input_tokens, vision_hidden_size))
        nn.init.trunc_normal_(pos_emb, mean=0.0, std=0.02)
    else:
        pos_emb = None

    return pos_emb
```
**EN:** Function `build_pos_embeds` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline. The docstring says: Build positional embeddings for the visual encoder output.
**CN:** Function `build_pos_embeds` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。 文档字符串表达的核心意思是：Build positional embeddings for the visual encoder output。

### Function `build_mlp` (lines 87-97)
```python
def build_mlp(
    depth: int,
    hidden_size: int,
    output_hidden_size: int,
) -> nn.Sequential:
    """Simple SiLU-activated MLP used as a projector readout."""
    layers = [nn.Linear(hidden_size, output_hidden_size)]
    for _ in range(1, depth):
        layers.append(nn.SiLU())
        layers.append(nn.Linear(output_hidden_size, output_hidden_size))
    return nn.Sequential(*layers)
```
**EN:** Function `build_mlp` encapsulates a focused piece of reusable logic inside this module. The docstring says: Simple SiLU-activated MLP used as a projector readout.
**CN:** Function `build_mlp` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Simple SiLU-activated MLP used as a projector readout。

### Class `KananaVImagePixelInputs` (lines 48-66)
```python
class KananaVImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - np: The total number of patches over all images in the batch
        - cps: Number of channels * patch_size * patch_size
        - ni: Number of images
    """

    type: Literal["pixel_values"]

    pixel_values: Annotated[
        torch.Tensor,
        TensorShape("np", "cps"),
    ]

    vision_grid_thw: Annotated[
        torch.Tensor,
        TensorShape("ni", 3),
    ]
```
**EN:** Class `KananaVImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `KananaVImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `PatchMerge` (lines 100-122)
```python
class PatchMerge(nn.Module):
    """Merge neighboring patches spatially to reduce resolution."""

    def __init__(self, merge_size: int) -> None:
        super().__init__()
        self.merge_size = merge_size

    def forward(
        self,
        x: torch.Tensor,
        channel_last: bool = False,
    ) -> torch.Tensor:
        """Merge patches by `merge_size x merge_size`."""
        if channel_last:
            x = rearrange(x, "B H W D -> B D H W")
        _, _, H, W = x.shape
        merged_x = rearrange(
            x,
            "B D (H h2) (W w2) -> B (D h2 w2) H W",
            h2=self.merge_size,
            w2=self.merge_size,
        )
        return merged_x
```
**EN:** Class `PatchMerge` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `PatchMerge` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `PatchMerge.__init__` (lines 103-105)
```python
    def __init__(self, merge_size: int) -> None:
        super().__init__()
        self.merge_size = merge_size
```
**EN:** Method `PatchMerge.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `PatchMerge.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `PatchMerge.forward` (lines 107-122)
```python
    def forward(
        self,
        x: torch.Tensor,
        channel_last: bool = False,
    ) -> torch.Tensor:
        """Merge patches by `merge_size x merge_size`."""
        if channel_last:
            x = rearrange(x, "B H W D -> B D H W")
        _, _, H, W = x.shape
        merged_x = rearrange(
            x,
            "B D (H h2) (W w2) -> B (D h2 w2) H W",
            h2=self.merge_size,
            w2=self.merge_size,
        )
        return merged_x
```
**EN:** Method `PatchMerge.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Merge patches by `merge_size x merge_size`.
**CN:** Method `PatchMerge.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Merge patches by `merge_size x merge_size`。

### Class `DynamicCAbstractor` (lines 125-261)
```python
class DynamicCAbstractor(nn.Module):
    """Dynamic C-Abstractor based on RegNet blocks."""

    def __init__(
        self,
        config: Qwen2VLVisionConfig,
        num_input_tokens: int,
    ) -> None:
        super().__init__()
        assert hasattr(config, "merge_size"), "merge_size must be provided."
        self.config = config
        self.merge_size = config.merge_size
        self.pos_emb_size = config.pos_emb_size
        if num_input_tokens == -1:
            num_input_tokens = config.pos_emb_size
        self.num_input_tokens = num_input_tokens
        self.pos_emb = build_pos_embeds(
            config, num_input_tokens, config.encoder_hidden_size
        )
        self.build_net()

    def _load_from_state_dict(self, state_dict, *args, **kwargs) -> None:
        if not state_dict:
            return
```
**EN:** Class `DynamicCAbstractor` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, _load_from_state_dict, build_net, forward, _forward.
**CN:** 类 `DynamicCAbstractor` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, _load_from_state_dict, build_net, forward, _forward。

### Method `DynamicCAbstractor.__init__` (lines 128-144)
```python
    def __init__(
        self,
        config: Qwen2VLVisionConfig,
        num_input_tokens: int,
    ) -> None:
        super().__init__()
        assert hasattr(config, "merge_size"), "merge_size must be provided."
        self.config = config
        self.merge_size = config.merge_size
        self.pos_emb_size = config.pos_emb_size
        if num_input_tokens == -1:
            num_input_tokens = config.pos_emb_size
        self.num_input_tokens = num_input_tokens
        self.pos_emb = build_pos_embeds(
            config, num_input_tokens, config.encoder_hidden_size
        )
        self.build_net()
```
**EN:** Method `DynamicCAbstractor.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `DynamicCAbstractor.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `DynamicCAbstractor.build_net` (lines 167-199)
```python
    def build_net(self) -> None:
        encoder_hidden_size = self.config.encoder_hidden_size
        hidden_size = self.config.hidden_size
        output_hidden_size = self.config.output_hidden_size
        depth = self.config.depth
        mlp_depth = self.config.mlp_depth

        RegBlock = partial(
            RegStage,
            stride=1,
            dilation=1,
            act_layer=nn.SiLU,
            norm_layer=LayerNorm2d,
        )

        s1 = RegBlock(
            depth,
            encoder_hidden_size,
            hidden_size,
        )
        sampler = PatchMerge(merge_size=self.merge_size)
        s2 = RegBlock(
            depth,
            self.merge_size**2 * hidden_size,
            hidden_size,
        )

        if depth:
            self.net = nn.ModuleList([s1, sampler, s2])
            self.readout = build_mlp(mlp_depth, hidden_size, output_hidden_size)
        else:
            self.net = sampler
            self.readout = build_mlp(mlp_depth, encoder_hidden_size, output_hidden_size)
```
**EN:** Method `DynamicCAbstractor.build_net` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DynamicCAbstractor.build_net` 封装了该模块中的一段可复用核心逻辑。

### Class `CustomQwen2VLVE` (lines 264-365)
```python
class CustomQwen2VLVE(Qwen2VisionTransformer):
    """Thin wrapper around the Qwen2-VL used as a vision encoder.

    This mirrors the original HF-based vision encoder used in Kanana-V, but
    reuses vLLM's optimized `Qwen2VisionTransformer` building blocks.
    """

    def __init__(self, config: Qwen2VLVisionConfig) -> None:
        super().__init__(
            vision_config=config,
            norm_eps=getattr(config, "rms_norm_eps", 1e-6),
            quant_config=None,
            prefix="",
        )

        # Kanana-V uses its own projector/abstractor instead of the Qwen2
        # built-in patch merger, so we drop the merger module to keep the
        # parameter set compatible with the original checkpoint.
        if hasattr(self, "merger"):
            del self.merger

    @classmethod
    def _from_config(cls, config: Qwen2VLVisionConfig) -> "CustomQwen2VLVE":
        """Drop-in replacement for the HF `_from_config` constructor."""
        return cls(config)
```
**EN:** Class `CustomQwen2VLVE` organizes related behavior for this model family or helper component. It inherits from Qwen2VisionTransformer. Key methods include __init__, _from_config, forward, get_num_tokens.
**CN:** 类 `CustomQwen2VLVE` 用于组织该模型族或辅助组件的相关行为。 它继承自 Qwen2VisionTransformer。 关键方法包括 __init__, _from_config, forward, get_num_tokens。

### Method `CustomQwen2VLVE.__init__` (lines 271-283)
```python
    def __init__(self, config: Qwen2VLVisionConfig) -> None:
        super().__init__(
            vision_config=config,
            norm_eps=getattr(config, "rms_norm_eps", 1e-6),
            quant_config=None,
            prefix="",
        )

        # Kanana-V uses its own projector/abstractor instead of the Qwen2
        # built-in patch merger, so we drop the merger module to keep the
        # parameter set compatible with the original checkpoint.
        if hasattr(self, "merger"):
            del self.merger
```
**EN:** Method `CustomQwen2VLVE.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `CustomQwen2VLVE.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `CustomQwen2VLVE.forward` (lines 290-361)
```python
    def forward(
        self,
        pixel_values: torch.Tensor,
        grid_thw: torch.Tensor,
        output_hidden_states: bool | None = None,
        return_dict: bool | None = None,
    ) -> tuple | BaseModelOutput:
        """Run the vision transformer and optionally return intermediate states.

        Unlike the base `Qwen2VisionTransformer`, this wrapper exposes the
        pre-merger patch-level representations and a HF-style `BaseModelOutput`
        so that the existing projector / abstractor code can be reused.
        """
        assert return_dict, "Only return_dict=True is supported."

        # Patchify
        x = pixel_values.to(device=self.device, dtype=self.dtype)
        x = self.patch_embed(x)  # (num_patches, embed_dim)

        # Prepare grid and rotary embeddings – mirror base implementation.
        if isinstance(grid_thw, list):
            grid_thw_list = grid_thw
            grid_thw_np = np.array(grid_thw, dtype=np.int32)
        else:
            grid_thw_list = grid_thw.tolist()
            grid_thw_np = grid_thw.cpu().numpy()

        rotary_pos_emb_cos, rotary_pos_emb_sin = self.rot_pos_emb(grid_thw_list)

        # Compute cu_seqlens in numpy then move to device, same as base model.
        cu_seqlens = np.repeat(
            grid_thw_np[:, 1] * grid_thw_np[:, 2],
            grid_thw_np[:, 0],
        ).cumsum(axis=0, dtype=np.int32)
        cu_seqlens = np.concatenate([np.zeros(1, dtype=np.int32), cu_seqlens])
        cu_seqlens = torch.from_numpy(cu_seqlens).to(
            self.device,
            non_blocking=True,
        )

        # Shape to (S, B, D) with batch dimension 1 as expected by the blocks.
        x = x.unsqueeze(1)

        # Pre-compute seqlens for attention backend.
        max_seqlen = self.compute_attn_mask_seqlen(cu_seqlens)

        encoder_states = () if output_hidden_states else None

        for blk in self.blocks:
            if output_hidden_states:
                # Store patch-level states (S, D).
                encoder_states = encoder_states + (x.squeeze(1),)

            x = blk(
                x,
                cu_seqlens=cu_seqlens,
                rotary_pos_emb_cos=rotary_pos_emb_cos,
                rotary_pos_emb_sin=rotary_pos_emb_sin,
                max_seqlen=max_seqlen,
            )
# ... truncated for analysis ...
```
**EN:** Method `CustomQwen2VLVE.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Run the vision transformer and optionally return intermediate states.
**CN:** Method `CustomQwen2VLVE.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Run the vision transformer and optionally return intermediate states。

### Class `KananaVProcessingInfo` (lines 368-435)
```python
class KananaVProcessingInfo(BaseProcessingInfo):
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def get_image_size_with_most_features(self) -> ImageSize:
        max_image_size, _ = self._get_vision_info(
            image_width=9999,
            image_height=9999,
            num_frames=1,
        )
        return max_image_size

    def _get_vision_info(
        self,
        *,
        image_width: int,
        image_height: int,
        num_frames: int = 1,
        do_resize: bool = True,
    ) -> tuple[ImageSize, int]:
        image_processor = self.ctx.get_hf_processor().image_processor
        smart_resize = resolve_obj_by_qualname(
            f"{type(image_processor).__module__}.smart_resize"
        )
```
**EN:** Class `KananaVProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_supported_mm_limits, get_image_size_with_most_features, _get_vision_info, get_mm_max_tokens_per_item.
**CN:** 类 `KananaVProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_supported_mm_limits, get_image_size_with_most_features, _get_vision_info, get_mm_max_tokens_per_item。

### Method `KananaVProcessingInfo.get_supported_mm_limits` (lines 369-370)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}
```
**EN:** Method `KananaVProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KananaVProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Method `KananaVProcessingInfo.get_image_size_with_most_features` (lines 372-378)
```python
    def get_image_size_with_most_features(self) -> ImageSize:
        max_image_size, _ = self._get_vision_info(
            image_width=9999,
            image_height=9999,
            num_frames=1,
        )
        return max_image_size
```
**EN:** Method `KananaVProcessingInfo.get_image_size_with_most_features` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KananaVProcessingInfo.get_image_size_with_most_features` 封装了该模块中的一段可复用核心逻辑。

### Class `KananaVDummyInputsBuilder` (lines 438-454)
```python
class KananaVDummyInputsBuilder(BaseDummyInputsBuilder[KananaVProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        return "<image>" * num_images

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        return {
            "image": self._get_dummy_images(
                width=9999, height=9999, num_images=num_images
            ),
        }
```
**EN:** Class `KananaVDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[KananaVProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `KananaVDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[KananaVProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `KananaVDummyInputsBuilder.get_dummy_text` (lines 439-441)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        return "<image>" * num_images
```
**EN:** Method `KananaVDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KananaVDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `KananaVDummyInputsBuilder.get_dummy_mm_data` (lines 443-454)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        return {
            "image": self._get_dummy_images(
                width=9999, height=9999, num_images=num_images
            ),
        }
```
**EN:** Method `KananaVDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KananaVDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `KananaVMultiModalProcessor` (lines 457-567)
```python
class KananaVMultiModalProcessor(BaseMultiModalProcessor[KananaVProcessingInfo]):
    """vLLM multimodal processor for Kanana-V (text + image)."""

    @property
    def media_token_id(self) -> int:
        return self.info.get_hf_config().text_config.eos_token_id + 1

    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        """Run the underlying HF processor on text and image data."""
        # Text-only input is handled as a special case here.
        if not mm_data or not mm_data.get("images", []):
            prompt_ids = self.info.get_tokenizer().encode(prompt)
            return BatchFeature(dict(input_ids=[prompt_ids]), tensor_type="pt")

        # Images
        image_inputs = mm_data.get("images", [])
        pixel_sizes = []
        if not isinstance(image_inputs[0], Image.Image):
            image_inputs = [Image.fromarray(image) for image in image_inputs]
```
**EN:** Class `KananaVMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[KananaVProcessingInfo]. Key methods include media_token_id, _call_hf_processor, _get_prompt_updates, _get_mm_fields_config.
**CN:** 类 `KananaVMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[KananaVProcessingInfo]。 关键方法包括 media_token_id, _call_hf_processor, _get_prompt_updates, _get_mm_fields_config。

### Method `KananaVMultiModalProcessor.media_token_id` (lines 461-462)
```python
    @property
    def media_token_id(self) -> int:
        return self.info.get_hf_config().text_config.eos_token_id + 1
```
**EN:** Method `KananaVMultiModalProcessor.media_token_id` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KananaVMultiModalProcessor.media_token_id` 封装了该模块中的一段可复用核心逻辑。

### Class `KananaVForConditionalGeneration` (lines 575-758)
```python
@MULTIMODAL_REGISTRY.register_processor(
    KananaVMultiModalProcessor,
    info=KananaVProcessingInfo,
    dummy_inputs=KananaVDummyInputsBuilder,
)
class KananaVForConditionalGeneration(nn.Module, SupportsMultiModal, SupportsPP):
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<image>"
        else:
            raise ValueError(f"Unsupported modality: {modality}")

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        self.config = config

        with self._mark_tower_model(vllm_config, "image"):
            self.vision_model = CustomQwen2VLVE._from_config(config.vision_config)
            self.abstractor = DynamicCAbstractor(
                config.projector_config,
                num_input_tokens=self.vision_model.get_num_tokens(),
            )
```
**EN:** Class `KananaVForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods include get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_input, _get_visual_feature_at, forward_vision.
**CN:** 类 `KananaVForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 关键方法包括 get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_input, _get_visual_feature_at, forward_vision。

### Method `KananaVForConditionalGeneration.get_placeholder_str` (lines 577-581)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<image>"
        else:
            raise ValueError(f"Unsupported modality: {modality}")
```
**EN:** Method `KananaVForConditionalGeneration.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KananaVForConditionalGeneration.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `KananaVForConditionalGeneration.__init__` (lines 583-606)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        self.config = config

        with self._mark_tower_model(vllm_config, "image"):
            self.vision_model = CustomQwen2VLVE._from_config(config.vision_config)
            self.abstractor = DynamicCAbstractor(
                config.projector_config,
                num_input_tokens=self.vision_model.get_num_tokens(),
            )

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=config.text_config,
                prefix=maybe_prefix(prefix, "model"),
                architectures=["LlamaForCausalLM"],
            )

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )
```
**EN:** Method `KananaVForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KananaVForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping, Sequence`, `from functools import partial`, `from typing import Annotated, Literal, TypeAlias`, `import regex as re`, `from einops import rearrange`, `from timm.layers import LayerNorm2d`, `from timm.layers.pos_embed import resample_abs_pos_embed`, `from timm.models.regnet import RegStage`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `from PIL import Image`, `from torch import nn`, `from transformers import BatchFeature`, `from transformers.modeling_outputs import BaseModelOutput`, `from transformers.models.qwen2_vl.configuration_qwen2_vl import Qwen2VLVisionConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.logger import init_logger`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import ImageSize, MultiModalDataItems`, `from vllm.multimodal.processing import (`, `from vllm.sequence import IntermediateTensors`, `from vllm.utils.import_utils import resolve_obj_by_qualname`, `from vllm.utils.tensor_schema import TensorSchema, TensorShape`, `from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP`
