# gemma4_mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/gemma4_mm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for gemma4_mm, including architecture wrappers and weight loading logic. / 面向推理的 gemma4_mm vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 17-79)
```python
import math
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Any, Literal

import numpy as np
import torch
from PIL import Image as PILImage
from torch import nn
from transformers import AutoModel, BatchFeature
from transformers.models.gemma4 import (
    Gemma4Config,
    Gemma4Processor,
    Gemma4VisionConfig,
)
from transformers.models.gemma4.configuration_gemma4 import (
    Gemma4AudioConfig,
    Gemma4TextConfig,
)

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions, VideoDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.logger import init_logger
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import ReplicatedLinear
from vllm.model_executor.models.gemma4 import Gemma4ForCausalLM
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
    VideoItem,
)
from vllm.multimodal.parse import (
    AudioProcessorItems,
    ImageProcessorItems,
    MultiModalDataItems,
    MultiModalDataParser,
)
from vllm.multimodal.processing import BaseDummyInputsBuilder
from vllm.multimodal.processing.processor import (
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
    PromptUpdateDetails,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import (
    MultiModalEmbeddings,
    SupportsEagle3,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
)
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 84-84)
```python
_SUPPORTED_SOFT_TOKENS = (70, 140, 280, 560, 1120)
```
**EN:** This block defines _SUPPORTED_SOFT_TOKENS, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _SUPPORTED_SOFT_TOKENS，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 85-85)
```python
_VIDEO_MAX_SOFT_TOKENS = 70  # soft tokens per video frame (vs 280 for images)
```
**EN:** This block defines _VIDEO_MAX_SOFT_TOKENS, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _VIDEO_MAX_SOFT_TOKENS，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 86-86)
```python
_VIDEO_MAX_FRAMES = 32  # max sampled frames per video
```
**EN:** This block defines _VIDEO_MAX_FRAMES, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _VIDEO_MAX_FRAMES，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Function `_get_max_soft_tokens` (lines 89-101)
```python
def _get_max_soft_tokens(
    merged_kwargs: Mapping[str, object],
) -> tuple[object | None, bool]:
    """Return configured image max_soft_tokens and whether it is top-level."""
    val = merged_kwargs.get("max_soft_tokens")
    if val is not None:
        return val, True

    images_kwargs = merged_kwargs.get("images_kwargs")
    if isinstance(images_kwargs, Mapping):
        return images_kwargs.get("max_soft_tokens"), False

    return None, False
```
**EN:** Function `_get_max_soft_tokens` encapsulates a focused piece of reusable logic inside this module. The docstring says: Return configured image max_soft_tokens and whether it is top-level.
**CN:** Function `_get_max_soft_tokens` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Return configured image max_soft_tokens and whether it is top-level。

### Class `Gemma4ImagePixelInputs` (lines 109-133)
```python
class Gemma4ImagePixelInputs(TensorSchema):
    """
    Pre-patchified image inputs from the Gemma4 image processor.

    Dimensions:
        - bn: Batch size * number of images
        - np: Number of patches (max_patches = max_soft_tokens * pooling_kernel_size²)
        - pp: Patch pixels (patch_size² * 3)

    The HF Gemma4ImageProcessor outputs pixel_values as
    (batch, max_patches, patch_pixels) — already patchified with
    zero-padding for patches beyond the real image content.
    pixel_position_ids provides (x, y) coordinates per patch,
    with (-1, -1) for padding patches.
    """

    type: Literal["pixel_values"] = "pixel_values"
    pixel_values: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("bn", "np", "pp", dynamic_dims={"np"}),
    ]
    pixel_position_ids: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("bn", "np", 2, dynamic_dims={"np"}),
    ]
```
**EN:** Class `Gemma4ImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Gemma4ImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Gemma4AudioInputs` (lines 136-150)
```python
class Gemma4AudioInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of audios
        - s: Sequence length (MEL spectrogram frames)
        - f: Number of features (MEL bins)
    """

    type: Literal["audio"] = "audio"
    input_features_padded: Annotated[
        torch.Tensor, TensorShape("bn", "s", "f", dynamic_dims={"s"})
    ]
    input_features_mask: Annotated[
        torch.Tensor, TensorShape("bn", "s", dynamic_dims={"s"})
    ]
```
**EN:** Class `Gemma4AudioInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Gemma4AudioInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Gemma4VideoInputs` (lines 156-171)
```python
class Gemma4VideoInputs(TensorSchema):
    """Video frame inputs — same tensor format as image inputs.

    Gemma4 has no separate video tower; video frames are processed
    through the vision tower at lower resolution (max_soft_tokens=70).
    """

    type: Literal["pixel_values_videos"] = "pixel_values_videos"
    pixel_values_videos: Annotated[
        torch.Tensor,
        TensorShape("bn", "np", "pp"),
    ]
    pixel_position_ids_videos: Annotated[
        torch.Tensor,
        TensorShape("bn", "np", 2),
    ]
```
**EN:** Class `Gemma4VideoInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `Gemma4VideoInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `Gemma4ProcessingInfo` (lines 179-399)
```python
class Gemma4ProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(Gemma4Config)

    def get_default_tok_params(self):
        """Gemma4's chat template already embeds a literal ``<bos>`` token in
        the rendered text.  If ``add_special_tokens=True`` (the base-class
        default), the tokenizer prepends *another* BOS, producing a
        ``[2, 2, ...]`` double-BOS sequence that the model was not trained on.

        Setting ``add_special_tokens=False`` here prevents the duplicate and
        ensures both ``llm.generate()`` and the chat/completions API behave
        correctly for IT models. For PT models (without chat template), we
        keep the default (True) to ensure BOS is added for raw prompts.
        """
        tokenizer = self.ctx.get_tokenizer()
        has_chat_template = getattr(tokenizer, "chat_template", None) is not None

        params = super().get_default_tok_params()
        if has_chat_template:
            params = params.with_kwargs(add_special_tokens=False)
        return params

    def get_hf_processor(self, **kwargs: object) -> Gemma4Processor:
        return self.ctx.get_hf_processor(
```
**EN:** Class `Gemma4ProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_default_tok_params, get_hf_processor, validate_num_items, get_supported_mm_limits, get_mm_max_tokens_per_item.
**CN:** 类 `Gemma4ProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_default_tok_params, get_hf_processor, validate_num_items, get_supported_mm_limits, get_mm_max_tokens_per_item。

### Method `Gemma4ProcessingInfo.get_hf_config` (lines 180-181)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config(Gemma4Config)
```
**EN:** Method `Gemma4ProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma4ProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `Gemma4ProcessingInfo.get_default_tok_params` (lines 183-200)
```python
    def get_default_tok_params(self):
        """Gemma4's chat template already embeds a literal ``<bos>`` token in
        the rendered text.  If ``add_special_tokens=True`` (the base-class
        default), the tokenizer prepends *another* BOS, producing a
        ``[2, 2, ...]`` double-BOS sequence that the model was not trained on.

        Setting ``add_special_tokens=False`` here prevents the duplicate and
        ensures both ``llm.generate()`` and the chat/completions API behave
        correctly for IT models. For PT models (without chat template), we
        keep the default (True) to ensure BOS is added for raw prompts.
        """
        tokenizer = self.ctx.get_tokenizer()
        has_chat_template = getattr(tokenizer, "chat_template", None) is not None

        params = super().get_default_tok_params()
        if has_chat_template:
            params = params.with_kwargs(add_special_tokens=False)
        return params
```
**EN:** Method `Gemma4ProcessingInfo.get_default_tok_params` encapsulates a focused piece of reusable logic inside this module. The docstring says: Gemma4's chat template already embeds a literal ``<bos>`` token in the rendered text.
**CN:** Method `Gemma4ProcessingInfo.get_default_tok_params` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Gemma4's chat template already embeds a literal ``<bos>`` token in the rendered text。

### Class `Gemma4DummyInputsBuilder` (lines 407-507)
```python
class Gemma4DummyInputsBuilder(BaseDummyInputsBuilder[Gemma4ProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        num_audios = mm_counts.get("audio", 0)
        num_videos = mm_counts.get("video", 0)
        processor = self.info.get_hf_processor()
        # Use image_token (<|image|>) with tab prefix — this is what the
        # Gemma4 chat template inserts per image (\t<|image|>).
        # _get_prompt_updates targets image_token and expands it to the
        # full_image_sequence.
        text = ("\t" + processor.image_token) * num_images
        if num_audios > 0 and processor.audio_token:
            text += processor.audio_token * num_audios
        if num_videos > 0:
            text += processor.video_token * num_videos
        return text

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions] | None = None,
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        num_audios = mm_counts.get("audio", 0)
```
**EN:** Class `Gemma4DummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[Gemma4ProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data, _get_dummy_videos.
**CN:** 类 `Gemma4DummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[Gemma4ProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data, _get_dummy_videos。

### Method `Gemma4DummyInputsBuilder.get_dummy_text` (lines 408-422)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        num_audios = mm_counts.get("audio", 0)
        num_videos = mm_counts.get("video", 0)
        processor = self.info.get_hf_processor()
        # Use image_token (<|image|>) with tab prefix — this is what the
        # Gemma4 chat template inserts per image (\t<|image|>).
        # _get_prompt_updates targets image_token and expands it to the
        # full_image_sequence.
        text = ("\t" + processor.image_token) * num_images
        if num_audios > 0 and processor.audio_token:
            text += processor.audio_token * num_audios
        if num_videos > 0:
            text += processor.video_token * num_videos
        return text
```
**EN:** Method `Gemma4DummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma4DummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `Gemma4DummyInputsBuilder.get_dummy_mm_data` (lines 424-473)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions] | None = None,
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        num_audios = mm_counts.get("audio", 0)
        num_videos = mm_counts.get("video", 0)
        processor = self.info.get_hf_processor()
        image_processor = processor.image_processor
        # Use processor's configured image size for dummies.
        # Gemma4ImageProcessor sets size=None (it uses patch_size /
        # max_soft_tokens instead of the standard size dict), so we
        # guard against None with `or {}`.
        size = getattr(image_processor, "size", None) or {}
        img_width = size.get("width", 224)
        img_height = size.get("height", 224)

        image_overrides = mm_options.get("image") if mm_options else None
        audio_overrides = mm_options.get("audio") if mm_options else None
        video_overrides = mm_options.get("video") if mm_options else None

        data: MultiModalDataDict = {
            "image": self._get_dummy_images(
                width=img_width,
                height=img_height,
                num_images=num_images,
                overrides=image_overrides,
            ),
        }

        if num_audios > 0:
            audio_len = processor.feature_extractor.fft_length
            data["audio"] = self._get_dummy_audios(
                length=audio_len,
                num_audios=num_audios,
                overrides=audio_overrides,
            )

        if num_videos > 0:
            data["video"] = self._get_dummy_videos(
                width=img_width,
                height=img_height,
                num_frames=_VIDEO_MAX_FRAMES,
                num_videos=num_videos,
                overrides=video_overrides,
            )

        return data
```
**EN:** Method `Gemma4DummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Gemma4DummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `Gemma4MultiModalProcessor` (lines 515-842)
```python
class Gemma4MultiModalProcessor(BaseMultiModalProcessor[Gemma4ProcessingInfo]):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        merged_kwargs = self.info.ctx.get_merged_mm_kwargs(mm_kwargs)
        val, is_top_level_max_soft_tokens = _get_max_soft_tokens(merged_kwargs)

        if val is not None and val not in _SUPPORTED_SOFT_TOKENS:
            raise ValueError(
                f"Unsupported max_soft_tokens value: {val}. "
                f"Valid values are {_SUPPORTED_SOFT_TOKENS}."
            )

        mm_data = dict(mm_data)

        # ---- VIDEO HANDLING ----
        # Gemma4 decomposes video into timestamped image frames.
        # Each frame is processed with max_soft_tokens=70 through the
        # same vision tower, matching transformers processing_gemma4.py.
        video_outputs: dict[str, Any] = {}
        if videos := mm_data.pop("videos", []):
```
**EN:** Class `Gemma4MultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[Gemma4ProcessingInfo]. Key methods include _call_hf_processor, _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `Gemma4MultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[Gemma4ProcessingInfo]。 关键方法包括 _call_hf_processor, _get_mm_fields_config, _get_prompt_updates。

### Class `Gemma4MultimodalEmbedder` (lines 856-903)
```python
class Gemma4MultimodalEmbedder(nn.Module):
    """Projects vision/audio soft tokens into LM embedding space.

    Architecture:
        inputs_embeds → embedding_projection → embedding_post_projection_norm

    Unlike Gemma3n which has separate hard/soft embedding paths with
    per-path normalization and a learned embedding table, Gemma4 uses a
    simplified 2-layer design: a linear projection followed by RMSNorm
    (without learnable scale).  The checkpoint confirms this — only
    ``embedding_projection.weight`` exists; there is no embedding table
    or pre-projection norm weights.
    """

    def __init__(
        self,
        multimodal_config: Gemma4VisionConfig | Gemma4AudioConfig,
        text_config: Gemma4TextConfig,
    ):
        super().__init__()

        self.eps = multimodal_config.rms_norm_eps
        self.text_hidden_size = text_config.hidden_size

        # Audio tower uses output_proj_dims (1536) rather than hidden_size
```
**EN:** Class `Gemma4MultimodalEmbedder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma4MultimodalEmbedder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma4MultimodalEmbedder.__init__` (lines 870-897)
```python
    def __init__(
        self,
        multimodal_config: Gemma4VisionConfig | Gemma4AudioConfig,
        text_config: Gemma4TextConfig,
    ):
        super().__init__()

        self.eps = multimodal_config.rms_norm_eps
        self.text_hidden_size = text_config.hidden_size

        # Audio tower uses output_proj_dims (1536) rather than hidden_size
        # (1024); vision uses hidden_size (768) directly.
        embedding_dim = (
            getattr(multimodal_config, "output_proj_dims", None)
            or multimodal_config.hidden_size
        )

        self.embedding_pre_projection_norm = RMSNorm(
            embedding_dim,
            eps=self.eps,
            has_weight=False,
        )

        self.embedding_projection = ReplicatedLinear(
            embedding_dim,
            self.text_hidden_size,
            bias=False,
        )
```
**EN:** Method `Gemma4MultimodalEmbedder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4MultimodalEmbedder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4MultimodalEmbedder.forward` (lines 899-903)
```python
    def forward(self, inputs_embeds: torch.Tensor) -> torch.Tensor:
        """Project soft tokens from a multimodal tower into LM space."""
        embs_normed = self.embedding_pre_projection_norm(inputs_embeds)
        embs_proj, _ = self.embedding_projection(embs_normed)
        return embs_proj
```
**EN:** Method `Gemma4MultimodalEmbedder.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Project soft tokens from a multimodal tower into LM space.
**CN:** Method `Gemma4MultimodalEmbedder.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Project soft tokens from a multimodal tower into LM space。

### Class `Gemma4ForConditionalGeneration` (lines 916-1483)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Gemma4MultiModalProcessor,
    info=Gemma4ProcessingInfo,
    dummy_inputs=Gemma4DummyInputsBuilder,
)
class Gemma4ForConditionalGeneration(
    nn.Module,
    SupportsMultiModal,
    SupportsPP,
    SupportsLoRA,
    SupportsEagle3,
):
    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
        "gate_up_proj": [
            "gate_proj",
            "up_proj",
        ],
    }

    # Maps checkpoint prefixes to vLLM module paths.
```
**EN:** Class `Gemma4ForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP, SupportsLoRA, SupportsEagle3. Key methods include __init__, _parse_and_validate_image_input, _parse_and_validate_audio_input, _parse_and_validate_video_input, _parse_and_validate_multimodal_inputs, _process_image_input.
**CN:** 类 `Gemma4ForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP、SupportsLoRA、SupportsEagle3。 关键方法包括 __init__, _parse_and_validate_image_input, _parse_and_validate_audio_input, _parse_and_validate_video_input, _parse_and_validate_multimodal_inputs, _process_image_input。

### Method `Gemma4ForConditionalGeneration.__init__` (lines 948-1027)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
        self.config = config
        self.quant_config = quant_config
        self.multimodal_config = multimodal_config

        # ---- Vision tower (shared by image and video) ----
        with self._mark_tower_model(vllm_config, {"image", "video"}):
            self.vision_tower = AutoModel.from_config(config=config.vision_config)
            self.embed_vision = Gemma4MultimodalEmbedder(
                config.vision_config, config.text_config
            )

        # ---- Audio tower (variants with audio_config) ----
        if config.audio_config is not None:
            with self._mark_tower_model(vllm_config, "audio"):
                self.audio_tower = AutoModel.from_config(config=config.audio_config)
                # AutoModel.from_config does NOT call post_init(),
                # which is needed to initialize buffers that are absent
                # from the checkpoint (e.g. inv_timescales for relative
                # position embeddings, softcap, gradient_clipping).
                self.audio_tower.post_init()
                self.embed_audio = Gemma4MultimodalEmbedder(
                    config.audio_config, config.text_config
                )
        else:
            self.audio_tower = None
            self.embed_audio = None

        # ---- Language model (vLLM optimised) ----
        with self._mark_language_model(vllm_config):
            self.language_model: Gemma4ForCausalLM = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=config.text_config,
                prefix=maybe_prefix(prefix, "language_model"),
                architectures=["Gemma4ForCausalLM"],
            )

            # Pre-allocate PLE buffer for CUDA graph compatibility.
            # Some variants have hidden_size_per_layer_input=None (no PLE).
            ple_dim = config.text_config.hidden_size_per_layer_input
            if ple_dim is not None:
                self.per_layer_embeddings = torch.zeros(
                    vllm_config.scheduler_config.max_num_batched_tokens,
                    config.text_config.num_hidden_layers,
                    ple_dim,
                    device=(self.language_model.model.embed_tokens.weight.device),
                    dtype=(self.language_model.model.embed_tokens.weight.dtype),
                )
            else:
                self.per_layer_embeddings = None

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )

        # --- Precompute full-attention layer indices for bidi clearing ---
# ... truncated for analysis ...
```
**EN:** Method `Gemma4ForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4ForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4ForConditionalGeneration.embed_multimodal` (lines 1273-1293)
```python
    def embed_multimodal(self, **kwargs: object) -> MultiModalEmbeddings:
        mm_input_by_modality = self._parse_and_validate_multimodal_inputs(**kwargs)
        multimodal_embeddings: list[torch.Tensor] = []

        for modality, multimodal_input in mm_input_by_modality.items():
            if multimodal_input is None:
                continue
            if modality == "image":
                multimodal_embeddings.extend(
                    self._process_image_input(multimodal_input)
                )
            elif modality == "video":
                multimodal_embeddings.extend(
                    self._process_video_input(multimodal_input)
                )
            elif modality == "audio":
                multimodal_embeddings.extend(
                    self._process_audio_input(multimodal_input)
                )

        return multimodal_embeddings
```
**EN:** Method `Gemma4ForConditionalGeneration.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Gemma4ForConditionalGeneration.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **LoRA support / LoRA 支持**
  - **EN:** The file is aware of low-rank adaptation interfaces or compatible parameter layouts.
  - **CN:** 该文件考虑了 LoRA 接口或兼容的低秩参数布局。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, Any, Literal`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `from PIL import Image as PILImage`, `from torch import nn`, `from transformers import AutoModel, BatchFeature`, `from transformers.models.gemma4 import (`, `from transformers.models.gemma4.configuration_gemma4 import (`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions, VideoDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import ReplicatedLinear`, `from vllm.model_executor.models.gemma4 import Gemma4ForCausalLM`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import (`, `from vllm.multimodal.processing import BaseDummyInputsBuilder`
- **Module note / 模块说明**: **EN:** Gemma 4 multimodal model (image + audio + video support).  Adds vision tower, audio tower, and multimodal embedders on top of the text-only Gemma4ForCausalLM.  The vision/audio encoders are loaded via AutoModel.from_config and run in eager mode while the language model uses the vLLM-optimized path.  Video support:  Gemma4 does **not** have a native video tower.  Videos are decomposed into timestamped image frames (up to 32 frames at 70 soft tokens each) and fed through the same vision tower as regular images.  The processor inserts ``mm:ss`` timestamps between frames so the model can reason about temporal order. **CN:** 模块文档字符串给出的原始说明是：Gemma 4 multimodal model (image + audio + video support).  Adds vision tower, audio tower, and multimodal embedders on top of the text-only Gemma4ForCausalLM.  The vision/audio encoders are loaded via AutoModel.from_config and run in eager mode while the language model uses the vLLM-optimized path.  Video support:  Gemma4 does **not** have a native video tower.  Videos are decomposed into timestamped image frames (up to 32 frames at 70 soft tokens each) and fed through the same vision tower as regular images.  The processor inserts ``mm:ss`` timestamps between frames so the model can reason about temporal order.。
