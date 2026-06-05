# deepseek_ocr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/deepseek_ocr.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for deepseek_ocr, including encoder/decoder glue and vLLM runtime adaptation. / 面向 deepseek_ocr 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-65)
```python
import math
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Literal

import torch
import torch.nn as nn
from transformers import BatchFeature, CLIPVisionConfig

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.models.interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
)
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.model_executor.models.utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
    NestedTensors,
)
from vllm.multimodal.parse import (
    ImageEmbeddingItems,
    ImageProcessorItems,
    ImageSize,
    MultiModalDataItems,
)
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
)
from vllm.sampling_params import SamplingParams
from vllm.sequence import IntermediateTensors
from vllm.tokenizers import cached_tokenizer_from_config
from vllm.transformers_utils.configs.deepseek_vl2 import DeepseekVLV2Config
from vllm.transformers_utils.processors.deepseek_ocr import (
    BASE_SIZE,
    CROP_MODE,
    DeepseekOCRProcessor,
    count_tiles,
)
from vllm.utils.tensor_schema import TensorSchema, TensorShape
from vllm.v1.sample.logits_processor import (
    AdapterLogitsProcessor,
    RequestLogitsProcessor,
)

from .deepencoder import DeepCLIPVisionTransformer, build_sam_vit_b
from .deepseek_vl2 import MlpProjector
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 68-68)
```python
IMAGE_SIZE = 640
```
**EN:** This block defines IMAGE_SIZE, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 IMAGE_SIZE，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 69-69)
```python
_IMAGE_TOKEN = "<image>"
```
**EN:** This block defines _IMAGE_TOKEN, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _IMAGE_TOKEN，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Class `DeepseekOCRImagePixelInputs` (lines 72-91)
```python
class DeepseekOCRImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - b: Batch size
        - n: Number of images
        - p: Number of patches
        - base_size: Base size of the processor
        - image_size: Image size of the processor
    """

    type: Literal["pixel_values"]
    data: Annotated[
        torch.Tensor,
        TensorShape("bn", 3, "base_size", "base_size", dynamic_dims={"bnp"}),
    ]
    images_crop: Annotated[
        torch.Tensor,
        TensorShape("bnp", 3, "image_size", "image_size", dynamic_dims={"bnp"}),
    ]
    images_spatial_crop: Annotated[torch.Tensor, TensorShape("bn", 2)]
```
**EN:** Class `DeepseekOCRImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `DeepseekOCRImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `NoRepeatNGramLogitsProcessor` (lines 94-129)
```python
class NoRepeatNGramLogitsProcessor:
    def __init__(
        self,
        ngram_size: int,
        window_size: int,
        whitelist_token_ids: set[int] | None = None,
    ):
        self.ngram_size = ngram_size
        self.window_size = window_size
        self.whitelist_token_ids = whitelist_token_ids or set()

    def __call__(
        self,
        output_ids: list[int],
        logits: torch.Tensor,
    ) -> torch.Tensor:
        if len(output_ids) < self.ngram_size:
            return logits

        current_prefix = tuple(output_ids[-(self.ngram_size - 1) :])

        search_start = max(0, len(output_ids) - self.window_size)
        search_end = len(output_ids) - self.ngram_size + 1

        banned_tokens = set()
```
**EN:** Class `NoRepeatNGramLogitsProcessor` organizes related behavior for this model family or helper component. Key methods include __init__, __call__.
**CN:** 类 `NoRepeatNGramLogitsProcessor` 用于组织该模型族或辅助组件的相关行为。 关键方法包括 __init__, __call__。

### Method `NoRepeatNGramLogitsProcessor.__init__` (lines 95-103)
```python
    def __init__(
        self,
        ngram_size: int,
        window_size: int,
        whitelist_token_ids: set[int] | None = None,
    ):
        self.ngram_size = ngram_size
        self.window_size = window_size
        self.whitelist_token_ids = whitelist_token_ids or set()
```
**EN:** Method `NoRepeatNGramLogitsProcessor.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `NoRepeatNGramLogitsProcessor.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `NGramPerReqLogitsProcessor` (lines 132-185)
```python
class NGramPerReqLogitsProcessor(AdapterLogitsProcessor):
    """Example of overriding the wrapper class `__init__()` in order to utilize
    info about the device type"""

    @classmethod
    def validate_params(cls, params: SamplingParams):
        ngram_size = params.extra_args and params.extra_args.get("ngram_size")
        window_size = params.extra_args and params.extra_args.get("window_size", 100)
        whitelist_token_ids = params.extra_args and params.extra_args.get(
            "whitelist_token_ids", None
        )
        # if ngram_size is not provided, skip validation because the processor
        # will not be used.
        if ngram_size is None:
            return None

        if not isinstance(ngram_size, int) or ngram_size <= 0:
            raise ValueError(
                f"`ngram_size` has to be a strictly positive integer, got {ngram_size}."
            )
        if not isinstance(window_size, int) or window_size <= 0:
            raise ValueError(
                "`window_size` has to be a strictly positive integer, "
                f"got {window_size}."
            )
```
**EN:** Class `NGramPerReqLogitsProcessor` organizes related behavior for this model family or helper component. It inherits from AdapterLogitsProcessor. Key methods include validate_params, is_argmax_invariant, new_req_logits_processor.
**CN:** 类 `NGramPerReqLogitsProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 AdapterLogitsProcessor。 关键方法包括 validate_params, is_argmax_invariant, new_req_logits_processor。

### Method `NGramPerReqLogitsProcessor.validate_params` (lines 137-163)
```python
    @classmethod
    def validate_params(cls, params: SamplingParams):
        ngram_size = params.extra_args and params.extra_args.get("ngram_size")
        window_size = params.extra_args and params.extra_args.get("window_size", 100)
        whitelist_token_ids = params.extra_args and params.extra_args.get(
            "whitelist_token_ids", None
        )
        # if ngram_size is not provided, skip validation because the processor
        # will not be used.
        if ngram_size is None:
            return None

        if not isinstance(ngram_size, int) or ngram_size <= 0:
            raise ValueError(
                f"`ngram_size` has to be a strictly positive integer, got {ngram_size}."
            )
        if not isinstance(window_size, int) or window_size <= 0:
            raise ValueError(
                "`window_size` has to be a strictly positive integer, "
                f"got {window_size}."
            )
        if whitelist_token_ids is not None and not isinstance(
            whitelist_token_ids, Iterable
        ):
            raise ValueError(
                "`whitelist_token_ids` has to be a sequence of integers, "
                f"got {whitelist_token_ids}."
            )
```
**EN:** Method `NGramPerReqLogitsProcessor.validate_params` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `NGramPerReqLogitsProcessor.validate_params` 封装了该模块中的一段可复用核心逻辑。

### Method `NGramPerReqLogitsProcessor.is_argmax_invariant` (lines 165-166)
```python
    def is_argmax_invariant(self) -> bool:
        return False
```
**EN:** Method `NGramPerReqLogitsProcessor.is_argmax_invariant` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `NGramPerReqLogitsProcessor.is_argmax_invariant` 封装了该模块中的一段可复用核心逻辑。

### Method `NGramPerReqLogitsProcessor.new_req_logits_processor` (lines 168-185)
```python
    def new_req_logits_processor(
        self,
        params: SamplingParams,
    ) -> RequestLogitsProcessor | None:
        ngram_size = params.extra_args and params.extra_args.get("ngram_size")
        window_size = params.extra_args and params.extra_args.get("window_size", 100)
        whitelist_token_ids = params.extra_args and params.extra_args.get(
            "whitelist_token_ids", None
        )
        if ngram_size is None:
            return None

        whitelist_token_ids = set(whitelist_token_ids) if whitelist_token_ids else None
        return NoRepeatNGramLogitsProcessor(
            ngram_size=ngram_size,
            window_size=window_size,
            whitelist_token_ids=whitelist_token_ids,
        )
```
**EN:** Method `NGramPerReqLogitsProcessor.new_req_logits_processor` converts hidden states into prediction-ready outputs, often by applying a head, post-processing, or sampling logic.
**CN:** Method `NGramPerReqLogitsProcessor.new_req_logits_processor` 将隐藏状态转成可预测输出，通常会经过输出头、后处理或采样逻辑。

### Class `DeepseekOCRProcessingInfo` (lines 188-244)
```python
class DeepseekOCRProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(DeepseekVLV2Config)

    def get_hf_processor(self, **kwargs: object):
        v1_processor_config = dict(
            image_size=IMAGE_SIZE,
            base_size=BASE_SIZE,
            crop_mode=CROP_MODE,
            strategy="v1",
        )

        return self.ctx.get_hf_processor(
            DeepseekOCRProcessor,
            **{**v1_processor_config, **kwargs},
        )

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def get_num_image_tokens(
        self, *, image_width: int, image_height: int, cropping: bool = True
    ) -> int:
        image_size = IMAGE_SIZE
        base_size = BASE_SIZE
```
**EN:** Class `DeepseekOCRProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_hf_processor, get_supported_mm_limits, get_num_image_tokens, get_image_size_with_most_features.
**CN:** 类 `DeepseekOCRProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_hf_processor, get_supported_mm_limits, get_num_image_tokens, get_image_size_with_most_features。

### Method `DeepseekOCRProcessingInfo.get_hf_config` (lines 189-190)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config(DeepseekVLV2Config)
```
**EN:** Method `DeepseekOCRProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekOCRProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `DeepseekOCRProcessingInfo.get_hf_processor` (lines 192-203)
```python
    def get_hf_processor(self, **kwargs: object):
        v1_processor_config = dict(
            image_size=IMAGE_SIZE,
            base_size=BASE_SIZE,
            crop_mode=CROP_MODE,
            strategy="v1",
        )

        return self.ctx.get_hf_processor(
            DeepseekOCRProcessor,
            **{**v1_processor_config, **kwargs},
        )
```
**EN:** Method `DeepseekOCRProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekOCRProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `DeepseekOCRProcessingInfo.get_supported_mm_limits` (lines 205-206)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}
```
**EN:** Method `DeepseekOCRProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekOCRProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Method `DeepseekOCRProcessingInfo.get_num_image_tokens` (lines 208-239)
```python
    def get_num_image_tokens(
        self, *, image_width: int, image_height: int, cropping: bool = True
    ) -> int:
        image_size = IMAGE_SIZE
        base_size = BASE_SIZE
        patch_size = 16
        downsample_ratio = 4

        if CROP_MODE:
            if image_width <= 640 and image_height <= 640:
                crop_ratio = [1, 1]
            else:
                # find the closest aspect ratio to the target
                crop_ratio = count_tiles(
                    image_width, image_height, image_size=IMAGE_SIZE
                )

            num_width_tiles, num_height_tiles = crop_ratio
        else:
            num_width_tiles = num_height_tiles = 1

        h = w = math.ceil((base_size // patch_size) / downsample_ratio)

        h2 = w2 = math.ceil((image_size // patch_size) / downsample_ratio)

        global_views_tokens = h * (w + 1)
        if num_width_tiles > 1 or num_height_tiles > 1:
            local_views_tokens = (num_height_tiles * h2) * (num_width_tiles * w2 + 1)
        else:
            local_views_tokens = 0

        return global_views_tokens + local_views_tokens + 1
```
**EN:** Method `DeepseekOCRProcessingInfo.get_num_image_tokens` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekOCRProcessingInfo.get_num_image_tokens` 封装了该模块中的一段可复用核心逻辑。

### Class `DeepseekOCRDummyInputsBuilder` (lines 247-272)
```python
class DeepseekOCRDummyInputsBuilder(BaseDummyInputsBuilder[DeepseekOCRProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token = processor.image_token

        return image_token * num_images

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)

        max_image_size = self.info.get_image_size_with_most_features()

        return {
            "image": self._get_dummy_images(
                width=max_image_size.width,
                height=max_image_size.height,
                num_images=num_images,
            )
```
**EN:** Class `DeepseekOCRDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[DeepseekOCRProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `DeepseekOCRDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[DeepseekOCRProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `DeepseekOCRDummyInputsBuilder.get_dummy_text` (lines 248-254)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token = processor.image_token

        return image_token * num_images
```
**EN:** Method `DeepseekOCRDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekOCRDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `DeepseekOCRDummyInputsBuilder.get_dummy_mm_data` (lines 256-272)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)

        max_image_size = self.info.get_image_size_with_most_features()

        return {
            "image": self._get_dummy_images(
                width=max_image_size.width,
                height=max_image_size.height,
                num_images=num_images,
            )
        }
```
**EN:** Method `DeepseekOCRDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekOCRDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `DeepseekOCRMultiModalProcessor` (lines 275-350)
```python
class DeepseekOCRMultiModalProcessor(
    BaseMultiModalProcessor[DeepseekOCRProcessingInfo]
):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        if mm_data:
            processed_outputs = self.info.ctx.call_hf_processor(
                self.info.get_hf_processor(**mm_kwargs),
                dict(prompt=prompt, **mm_data),
                mm_kwargs,
            )

        else:
            tokenizer = self.info.get_tokenizer()
            processed_outputs = tokenizer(
                prompt, add_special_tokens=True, return_tensors="pt"
            )

        return processed_outputs
```
**EN:** Class `DeepseekOCRMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[DeepseekOCRProcessingInfo]. Key methods include _call_hf_processor, _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `DeepseekOCRMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[DeepseekOCRProcessingInfo]。 关键方法包括 _call_hf_processor, _get_mm_fields_config, _get_prompt_updates。

### Class `DeepseekOCRForCausalLM` (lines 358-616)
```python
@MULTIMODAL_REGISTRY.register_processor(
    DeepseekOCRMultiModalProcessor,
    info=DeepseekOCRProcessingInfo,
    dummy_inputs=DeepseekOCRDummyInputsBuilder,
)
class DeepseekOCRForCausalLM(nn.Module, SupportsMultiModal, SupportsPP, SupportsLoRA):
    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            # map prefix for language backbone
            "model.embed_tokens.": "language_model.model.embed_tokens.",
            "model.layers.": "language_model.model.layers.",
            "model.norm.": "language_model.model.norm.",
            "lm_head.": "language_model.lm_head.",
            # remove "model." prefix for other components
            "model.": "",
        }
    )

    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<image>"

        raise ValueError("Only image modality is supported")
```
**EN:** Class `DeepseekOCRForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP, SupportsLoRA. Key methods include get_placeholder_str, __init__, _parse_and_validate_image_input, _encode_global_features, _encode_local_features, _pixel_values_to_embedding.
**CN:** 类 `DeepseekOCRForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP、SupportsLoRA。 关键方法包括 get_placeholder_str, __init__, _parse_and_validate_image_input, _encode_global_features, _encode_local_features, _pixel_values_to_embedding。

### Method `DeepseekOCRForCausalLM.get_placeholder_str` (lines 372-376)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<image>"

        raise ValueError("Only image modality is supported")
```
**EN:** Method `DeepseekOCRForCausalLM.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekOCRForCausalLM.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `DeepseekOCRForCausalLM.__init__` (lines 378-440)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config: DeepseekVLV2Config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config

        self.config = config
        self.multimodal_config = multimodal_config

        self.vision_config = config.vision_config
        self.projector_config = config.projector_config
        self.text_config = config.text_config

        model_config = vllm_config.model_config
        tokenizer = cached_tokenizer_from_config(model_config)
        self.image_token_id = tokenizer.vocab[_IMAGE_TOKEN]

        with self._mark_tower_model(vllm_config, "image"):
            self.sam_model = build_sam_vit_b()
            clip_vision_config = CLIPVisionConfig(
                hidden_size=1024,
                intermediate_size=4096,
                num_attention_heads=16,
                num_hidden_layers=24,
                image_size=224,
                patch_size=14,
                projection_dim=512,
                layer_norm_eps=1e-5,
            )
            self.vision_model = DeepCLIPVisionTransformer(
                config=clip_vision_config,
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "vision_model"),
            )

            self.projector = MlpProjector(self.projector_config)
            self.tile_tag = config.tile_tag
            self.global_view_pos = config.global_view_pos

            # special token for image token sequence format
            n_embed = self.projector_config.n_embed
            embed_std = 1 / torch.sqrt(torch.tensor(n_embed, dtype=torch.float32))
            if self.tile_tag == "2D":
                # <|view_separator|>, <|\n|>
                self.image_newline = nn.Parameter(torch.randn(n_embed) * embed_std)
                # This is a typo in original implementation
                self.view_seperator = nn.Parameter(torch.randn(n_embed) * embed_std)
            else:
                raise ValueError(
                    f"Only 2D tile_tag is supported currently, got: {self.tile_tag}"
                )

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=self.text_config,
                prefix=maybe_prefix(prefix, "language_model"),
            )

# ... truncated for analysis ...
```
**EN:** Method `DeepseekOCRForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `DeepseekOCRForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `DeepseekOCRForCausalLM.embed_multimodal` (lines 573-578)
```python
    def embed_multimodal(self, **kwargs: object) -> MultiModalEmbeddings | None:
        image_input = self._parse_and_validate_image_input(**kwargs)
        if image_input is None:
            return None
        vision_embeddings = self._process_image_input(image_input)
        return vision_embeddings
```
**EN:** Method `DeepseekOCRForCausalLM.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `DeepseekOCRForCausalLM.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `DeepseekOCRForCausalLM.forward` (lines 580-595)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ):
        if intermediate_tensors is not None:
            inputs_embeds = None

        hidden_states = self.language_model(
            input_ids, positions, intermediate_tensors, inputs_embeds=inputs_embeds
        )

        return hidden_states
```
**EN:** Method `DeepseekOCRForCausalLM.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `DeepseekOCRForCausalLM.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

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
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, Literal`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `from transformers import BatchFeature, CLIPVisionConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.models.interfaces import (`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`, `from vllm.model_executor.models.utils import (`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import (`, `from vllm.multimodal.processing import (`, `from vllm.sampling_params import SamplingParams`, `from vllm.sequence import IntermediateTensors`
- **Module note / 模块说明**: **EN:** Inference-only Deepseek-OCR model compatible with HuggingFace weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only Deepseek-OCR model compatible with HuggingFace weights.。
