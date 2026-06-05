# granite4_vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/granite4_vision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for granite4_vision, including encoder/decoder glue and vLLM runtime adaptation. / 面向 granite4_vision 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 11-60)
```python
import math
from collections.abc import Iterable, Mapping
from fractions import Fraction
from itertools import islice

import torch
import torch.nn as nn
from transformers import BatchFeature
from transformers.models.blip_2.configuration_blip_2 import Blip2QFormerConfig
from transformers.models.llava_next.modeling_llava_next import (
    get_anyres_image_grid_shape,
    image_size_to_num_patches,
    unpad_image,
)

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed.parallel_state import get_pp_group
from vllm.logger import init_logger
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead
from vllm.model_executor.models.granite import GraniteForCausalLM, GraniteModel
from vllm.model_executor.models.interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
)
from vllm.model_executor.models.llava import LlavaDummyInputsBuilder
from vllm.model_executor.models.llava_next import (
    BaseLlavaNextMultiModalProcessor,
    LlavaNextImageEmbeddingInputs,
    LlavaNextImageInputs,
    LlavaNextImagePixelInputs,
    LlavaNextProcessingInfo,
)
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.model_executor.models.siglip import SiglipVisionModel
from vllm.model_executor.models.utils import (
    AutoWeightsLoader,
    PPMissingLayer,
    WeightsMapper,
    maybe_prefix,
)
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import MultiModalFieldConfig
from vllm.sequence import IntermediateTensors

from .blip2 import Blip2QFormerModel
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `InterpolateDownsampler` (lines 70-91)
```python
class InterpolateDownsampler:
    """Spatial downsampling via area interpolation."""

    def __init__(self, config, mode="area"):
        self.orig_image_side = (
            config.vision_config.image_size // config.vision_config.patch_size
        )
        self.new_image_side = int(
            self.orig_image_side * Fraction(config.downsample_rate)
        )
        self.mode = mode

    def __call__(self, image_features: torch.Tensor) -> torch.Tensor:
        batch_size, _, dim = image_features.size()
        up_shape = [batch_size, self.orig_image_side, self.orig_image_side, dim]
        large = image_features.view(up_shape).permute(0, 3, 1, 2)
        small = torch.nn.functional.interpolate(
            large,
            size=(self.new_image_side, self.new_image_side),
            mode=self.mode,
        )
        return small.permute(0, 2, 3, 1).flatten(1, 2)
```
**EN:** Class `InterpolateDownsampler` organizes related behavior for this model family or helper component. Key methods include __init__, __call__.
**CN:** 类 `InterpolateDownsampler` 用于组织该模型族或辅助组件的相关行为。 关键方法包括 __init__, __call__。

### Method `InterpolateDownsampler.__init__` (lines 73-80)
```python
    def __init__(self, config, mode="area"):
        self.orig_image_side = (
            config.vision_config.image_size // config.vision_config.patch_size
        )
        self.new_image_side = int(
            self.orig_image_side * Fraction(config.downsample_rate)
        )
        self.mode = mode
```
**EN:** Method `InterpolateDownsampler.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InterpolateDownsampler.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `SpatialOffsetDownsampler` (lines 94-113)
```python
class SpatialOffsetDownsampler:
    """Sample one position from each 2x2 block (offset 0-3 = TL/TR/BL/BR)."""

    def __init__(self, config, offset: int = 0):
        self.orig_image_side = (
            config.vision_config.image_size // config.vision_config.patch_size
        )
        self.new_image_side = self.orig_image_side // 2
        offsets = [(0, 0), (0, 1), (1, 0), (1, 1)]
        self.offset_h, self.offset_w = offsets[offset]

    def __call__(self, image_features: torch.Tensor) -> torch.Tensor:
        B, _, C = image_features.shape
        features_2d = image_features.reshape(
            B, self.orig_image_side, self.orig_image_side, C
        )
        n = self.new_image_side
        blocks = features_2d.reshape(B, n, 2, n, 2, C)
        sampled = blocks[:, :, self.offset_h, :, self.offset_w, :]
        return sampled.reshape(B, -1, C)
```
**EN:** Class `SpatialOffsetDownsampler` organizes related behavior for this model family or helper component. Key methods include __init__, __call__.
**CN:** 类 `SpatialOffsetDownsampler` 用于组织该模型族或辅助组件的相关行为。 关键方法包括 __init__, __call__。

### Method `SpatialOffsetDownsampler.__init__` (lines 97-103)
```python
    def __init__(self, config, offset: int = 0):
        self.orig_image_side = (
            config.vision_config.image_size // config.vision_config.patch_size
        )
        self.new_image_side = self.orig_image_side // 2
        offsets = [(0, 0), (0, 1), (1, 0), (1, 1)]
        self.offset_h, self.offset_w = offsets[offset]
```
**EN:** Method `SpatialOffsetDownsampler.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `SpatialOffsetDownsampler.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `WindowQFormerDownsampler` (lines 116-218)
```python
class WindowQFormerDownsampler(nn.Module):
    """Window-based QFormer downsampler (matches HF downsampling.py exactly)."""

    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        cache_config: CacheConfig | None = None,
        spatial_offset: int | None = None,
        prefix: str = "",
    ):
        super().__init__()
        llm_hidden_size = config.text_config.hidden_size
        vision_hidden_size = config.vision_config.hidden_size

        self.dropout = nn.Dropout(config.projector_dropout)

        if spatial_offset is not None:
            self.downsampler = SpatialOffsetDownsampler(config, offset=spatial_offset)
        else:
            self.downsampler = InterpolateDownsampler(config)

        qformer_config = Blip2QFormerConfig(
            hidden_size=vision_hidden_size,
            num_attention_heads=vision_hidden_size // 64,
```
**EN:** Class `WindowQFormerDownsampler` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, _win, _unwin, forward.
**CN:** 类 `WindowQFormerDownsampler` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, _win, _unwin, forward。

### Method `WindowQFormerDownsampler.__init__` (lines 119-170)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        cache_config: CacheConfig | None = None,
        spatial_offset: int | None = None,
        prefix: str = "",
    ):
        super().__init__()
        llm_hidden_size = config.text_config.hidden_size
        vision_hidden_size = config.vision_config.hidden_size

        self.dropout = nn.Dropout(config.projector_dropout)

        if spatial_offset is not None:
            self.downsampler = SpatialOffsetDownsampler(config, offset=spatial_offset)
        else:
            self.downsampler = InterpolateDownsampler(config)

        qformer_config = Blip2QFormerConfig(
            hidden_size=vision_hidden_size,
            num_attention_heads=vision_hidden_size // 64,
            intermediate_size=3072,
            num_hidden_layers=1,
            encoder_hidden_size=vision_hidden_size,
            cross_attention_frequency=1,
            max_position_embeddings=2048,
            use_qformer_text_input=False,
        )
        self.qformer = Blip2QFormerModel(
            qformer_config,
            quant_config=quant_config,
            cache_config=cache_config,
            prefix=maybe_prefix(prefix, "qformer"),
        )

        self.image_side = (
            config.vision_config.image_size // config.vision_config.patch_size
        )
        q, w = config.downsample_rate.split("/")
        self.query_side, self.window_side = int(q), int(w)
        self.query_length = self.query_side**2

        embed_std = 1 / math.sqrt(vision_hidden_size)
        self.norm = nn.LayerNorm(vision_hidden_size, eps=1e-6)
        self.query = nn.Parameter(
            torch.randn(1, self.query_length, vision_hidden_size) * embed_std
        )
        self.image_positions = nn.Parameter(
            torch.randn(1, self.window_side**2, vision_hidden_size) * embed_std
        )
        self.out_linear = nn.Linear(vision_hidden_size, llm_hidden_size, bias=True)
```
**EN:** Method `WindowQFormerDownsampler.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `WindowQFormerDownsampler.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `WindowQFormerDownsampler.forward` (lines 197-218)
```python
    def forward(self, image_features: torch.Tensor) -> torch.Tensor:
        B, HW, C = image_features.shape
        assert self.image_side * self.image_side == HW
        n = self.image_side // self.window_side

        image_features = self.norm(image_features)
        enc = self._win(image_features, self.image_side, self.window_side)

        downsampled = self.downsampler(image_features)
        new_side = n * self.query_side
        downsampled_w = self._win(downsampled, new_side, self.query_side)

        query_embeds = self.query + downsampled_w
        encoder_embeds = self.dropout(enc + self.image_positions)
        out_w = self.qformer(
            query_embeds=query_embeds,
            encoder_hidden_states=encoder_embeds,
        )

        out = self._unwin(out_w, n=n, win=self.query_side)
        out = self.dropout(out)
        return self.out_linear(out)
```
**EN:** Method `WindowQFormerDownsampler.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `WindowQFormerDownsampler.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Granite4VisionLLMModel` (lines 235-296)
```python
@support_torch_compile(
    dynamic_arg_dims={
        "input_ids": 0,
        "positions": 0,
        "intermediate_tensors": 0,
        "inputs_embeds": 0,
        "deepstack_input_embeds": 0,
    }
)
class Granite4VisionLLMModel(GraniteModel):
    """GraniteModel with deepstack feature injection in the layer loop."""

    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        deepstack_input_embeds: IntermediateTensors | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        if get_pp_group().is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.embed_input_ids(input_ids)
```
**EN:** Class `Granite4VisionLLMModel` organizes related behavior for this model family or helper component. It inherits from GraniteModel. Key methods include forward.
**CN:** 类 `Granite4VisionLLMModel` 用于组织该模型族或辅助组件的相关行为。 它继承自 GraniteModel。 关键方法包括 forward。

### Method `Granite4VisionLLMModel.forward` (lines 238-296)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        deepstack_input_embeds: IntermediateTensors | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        if get_pp_group().is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.embed_input_ids(input_ids)
                hidden_states = hidden_states * self.config.embedding_multiplier
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]
            # Recover deepstack features forwarded from the previous PP rank.
            if deepstack_input_embeds is None:
                ds_keys = [
                    k for k in intermediate_tensors.tensors if k.startswith("ds_")
                ]
                if ds_keys:
                    deepstack_input_embeds = IntermediateTensors(
                        {k: intermediate_tensors[k] for k in ds_keys}
                    )

        for layer_idx, layer in islice(
            enumerate(self.layers), self.start_layer, self.end_layer
        ):
            if deepstack_input_embeds is not None:
                key = f"ds_{layer_idx}"
                if key in deepstack_input_embeds.tensors:
                    feat = deepstack_input_embeds[key]
                    # Resize to match hidden_states in case of CUDA graph padding
                    num_tokens = hidden_states.size(0)
                    buf_len = feat.shape[0]
                    if buf_len != num_tokens:
                        feat = torch.nn.functional.pad(
                            feat[:num_tokens],
                            (0, 0, 0, max(0, num_tokens - buf_len)),
                        )
                    hidden_states = hidden_states + feat
            hidden_states = layer(positions, hidden_states)

        if not get_pp_group().is_last_rank:
            # Forward hidden_states and any deepstack features for later ranks.
            it = {"hidden_states": hidden_states}
            if deepstack_input_embeds is not None:
                remaining = {
                    k: v
                    for k, v in deepstack_input_embeds.tensors.items()
                    if int(k.split("_")[1]) >= self.end_layer
                }
                it.update(remaining)
            return IntermediateTensors(it)

        hidden_states = self.norm(hidden_states)
        return hidden_states
```
**EN:** Method `Granite4VisionLLMModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Granite4VisionLLMModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Granite4VisionLLMForCausalLM` (lines 299-339)
```python
class Granite4VisionLLMForCausalLM(GraniteForCausalLM):
    """GraniteForCausalLM backed by Granite4VisionLLMModel."""

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        nn.Module.__init__(self)
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.quant_config = quant_config
        self.model = Granite4VisionLLMModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        if get_pp_group().is_last_rank:
            self.lm_head = ParallelLMHead(
                config.vocab_size,
                config.hidden_size,
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "lm_head"),
            )
            if config.tie_word_embeddings:
                self.lm_head.weight = self.model.embed_tokens.weight
            logit_scale = getattr(config, "logit_scale", 1.0)
            if hasattr(config, "logits_scaling"):
                logit_scale /= config.logits_scaling
            self.logits_processor = LogitsProcessor(
```
**EN:** Class `Granite4VisionLLMForCausalLM` is a structural model block in the vLLM execution graph. It inherits from GraniteForCausalLM. Key methods include __init__, make_empty_intermediate_tensors.
**CN:** 类 `Granite4VisionLLMForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 GraniteForCausalLM。 关键方法包括 __init__, make_empty_intermediate_tensors。

### Method `Granite4VisionLLMForCausalLM.__init__` (lines 302-327)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        nn.Module.__init__(self)
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.quant_config = quant_config
        self.model = Granite4VisionLLMModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        if get_pp_group().is_last_rank:
            self.lm_head = ParallelLMHead(
                config.vocab_size,
                config.hidden_size,
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "lm_head"),
            )
            if config.tie_word_embeddings:
                self.lm_head.weight = self.model.embed_tokens.weight
            logit_scale = getattr(config, "logit_scale", 1.0)
            if hasattr(config, "logits_scaling"):
                logit_scale /= config.logits_scaling
            self.logits_processor = LogitsProcessor(
                config.vocab_size, scale=logit_scale
            )
        else:
            self.lm_head = PPMissingLayer()
```
**EN:** Method `Granite4VisionLLMForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Granite4VisionLLMForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Granite4VisionLLMForCausalLM.make_empty_intermediate_tensors` (lines 329-339)
```python
    def make_empty_intermediate_tensors(
        self, batch_size: int, dtype: torch.dtype, device: torch.device
    ) -> IntermediateTensors:
        tensors = super().make_empty_intermediate_tensors(batch_size, dtype, device)
        # Include deepstack buffers so non-first PP ranks receive them.
        # _ds_layer_indices is set directly on this instance by the outer model.
        for llm_layer in getattr(self, "_ds_layer_indices", []):
            tensors.tensors[f"ds_{llm_layer}"] = torch.zeros(
                (batch_size, self.config.hidden_size), dtype=dtype, device=device
            )
        return tensors
```
**EN:** Method `Granite4VisionLLMForCausalLM.make_empty_intermediate_tensors` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Granite4VisionLLMForCausalLM.make_empty_intermediate_tensors` 封装了该模块中的一段可复用核心逻辑。

### Class `Granite4VisionProcessingInfo` (lines 347-388)
```python
class Granite4VisionProcessingInfo(LlavaNextProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config()

    def get_hf_processor(self, **kwargs):
        return self.ctx.get_hf_processor(**kwargs)

    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
        hf_config = self.get_hf_config()
        vision_encoder_info = self.get_vision_encoder_info()

        # After QFormer downsampling, patch grid is scaled by downsample_rate
        ds_rate = Fraction(hf_config.downsample_rate)
        patch_grid = vision_encoder_info.get_patch_grid_length()  # 24 for 384/16
        downsampled_grid = int(patch_grid * ds_rate)  # 12 for rate 4/8

        # Base feature: downsampled_grid^2
        base_feature_size = downsampled_grid * downsampled_grid

        num_patch_height, num_patch_width = get_anyres_image_grid_shape(
```
**EN:** Class `Granite4VisionProcessingInfo` organizes related behavior for this model family or helper component. It inherits from LlavaNextProcessingInfo. Key methods include get_hf_config, get_hf_processor, get_num_image_tokens.
**CN:** 类 `Granite4VisionProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 LlavaNextProcessingInfo。 关键方法包括 get_hf_config, get_hf_processor, get_num_image_tokens。

### Method `Granite4VisionProcessingInfo.get_hf_config` (lines 348-349)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config()
```
**EN:** Method `Granite4VisionProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Granite4VisionProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `Granite4VisionProcessingInfo.get_hf_processor` (lines 351-352)
```python
    def get_hf_processor(self, **kwargs):
        return self.ctx.get_hf_processor(**kwargs)
```
**EN:** Method `Granite4VisionProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Granite4VisionProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Class `Granite4VisionMultiModalProcessor` (lines 391-402)
```python
class Granite4VisionMultiModalProcessor(
    BaseLlavaNextMultiModalProcessor[Granite4VisionProcessingInfo]
):
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        return dict(
            pixel_values=MultiModalFieldConfig.batched("image"),
            image_sizes=MultiModalFieldConfig.batched("image"),
        )
```
**EN:** Class `Granite4VisionMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseLlavaNextMultiModalProcessor[Granite4VisionProcessingInfo]. Key methods include _get_mm_fields_config.
**CN:** 类 `Granite4VisionMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseLlavaNextMultiModalProcessor[Granite4VisionProcessingInfo]。 关键方法包括 _get_mm_fields_config。

### Class `Granite4VisionForConditionalGeneration` (lines 415-931)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Granite4VisionMultiModalProcessor,
    info=Granite4VisionProcessingInfo,
    dummy_inputs=LlavaDummyInputsBuilder,
)
class Granite4VisionForConditionalGeneration(
    nn.Module, SupportsLoRA, SupportsMultiModal, SupportsPP
):
    """vLLM implementation of Granite 4 Vision.

    Architecture:
    - SigLIP vision tower -> WindowQFormerDownsampler projectors
    - Deepstack: 4 vision layers projected and injected at 4 LLM layers
    - Spatial: 4 offset groups from last vision layer injected at 4 more LLM layers
    - Granite language backbone with embedding_multiplier
    - logits_scaling via LogitsProcessor

    The outer model runs the LLM layer loop directly (like HF does) to inject
    deepstack features. This avoids wrapping the inner model and keeps weight
    loading simple.

    LoRA support:
    - Full merge: --hf-overrides '{"adapter_path": "path/to/lora"}' merges
      LM-only LoRA deltas at load time (W += scaling * B @ A).
    - Native LoRA: --enable-lora --default-mm-loras '{"image": "path/to/lora"}'
```
**EN:** Class `Granite4VisionForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsLoRA, SupportsMultiModal, SupportsPP. Key methods include get_placeholder_str, get_mm_mapping, __init__, _get_vision_hidden_states, _pack_and_unpad_image_features, _get_all_layer_features.
**CN:** 类 `Granite4VisionForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsLoRA、SupportsMultiModal、SupportsPP。 关键方法包括 get_placeholder_str, get_mm_mapping, __init__, _get_vision_hidden_states, _pack_and_unpad_image_features, _get_all_layer_features。

### Method `Granite4VisionForConditionalGeneration.get_placeholder_str` (lines 462-465)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<image>"
        raise ValueError(f"Only image modality is supported, got {modality}")
```
**EN:** Method `Granite4VisionForConditionalGeneration.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Granite4VisionForConditionalGeneration.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `Granite4VisionForConditionalGeneration.get_mm_mapping` (lines 467-472)
```python
    def get_mm_mapping(self) -> MultiModelKeys:
        return MultiModelKeys.from_string_field(
            language_model="language_model",
            connector=["layerwise_projectors", "spatial_projectors"],
            tower_model="vision_tower",
        )
```
**EN:** Method `Granite4VisionForConditionalGeneration.get_mm_mapping` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `Granite4VisionForConditionalGeneration.get_mm_mapping` 封装了该模块中的一段可复用核心逻辑。

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
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable, Mapping`, `from fractions import Fraction`, `from itertools import islice`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `from transformers import BatchFeature`, `from transformers.models.blip_2.configuration_blip_2 import Blip2QFormerConfig`, `from transformers.models.llava_next.modeling_llava_next import (`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed.parallel_state import get_pp_group`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead`, `from vllm.model_executor.models.granite import GraniteForCausalLM, GraniteModel`, `from vllm.model_executor.models.interfaces import (`, `from vllm.model_executor.models.llava import LlavaDummyInputsBuilder`, `from vllm.model_executor.models.llava_next import (`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`
- **Module note / 模块说明**: **EN:** vLLM implementation of Granite 4 Vision.  Uses GraniteForCausalLM as the language backbone with SigLIP vision encoder and deepstack feature injection via WindowQFormer projectors.  LoRA support: use --enable-lora --default-mm-loras for LM-only LoRA adapters. **CN:** 模块文档字符串给出的原始说明是：vLLM implementation of Granite 4 Vision.  Uses GraniteForCausalLM as the language backbone with SigLIP vision encoder and deepstack feature injection via WindowQFormer projectors.  LoRA support: use --enable-lora --default-mm-loras for LM-only LoRA adapters.。
