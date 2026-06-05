# deepseek_vl2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/deepseek_vl2.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for deepseek_vl2, including encoder/decoder glue and vLLM runtime adaptation. / 面向 deepseek_vl2 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 7-60)
```python
import math
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Literal, TypeAlias

import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange, repeat
from transformers import BatchFeature

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.models.transformers.utils import replace_linear_class
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import (
    ImageEmbeddingItems,
    ImageProcessorItems,
    ImageSize,
    MultiModalDataItems,
)
from vllm.multimodal.processing import BaseDummyInputsBuilder
from vllm.multimodal.processing.processor import (
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    MultiModalProcessingInfo,
    ProcessorInputs,
    PromptReplacement,
    PromptUpdate,
    TimingContext,
)
from vllm.sequence import IntermediateTensors
from vllm.tokenizers import cached_tokenizer_from_config
from vllm.transformers_utils.configs.deepseek_vl2 import (
    DeepseekVLV2Config,
    MlpProjectorConfig,
    VisionEncoderConfig,
)
from vllm.utils.tensor_schema import TensorSchema, TensorShape
from vllm.utils.torch_utils import set_default_torch_dtype

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 63-63)
```python
_IMAGE_TOKEN = "<image>"
```
**EN:** This block defines _IMAGE_TOKEN, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _IMAGE_TOKEN，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Class `DeepseekVL2ImagePixelInputs` (lines 66-78)
```python
class DeepseekVL2ImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bnp: Batch size * number of images * number of patches
        - p: Number of patches
        - c: Number of channels (3)
        - h: Height of each image
        - w: Width of each image
    """

    type: Literal["pixel_values"]
    data: Annotated[torch.Tensor, TensorShape("bnp", 3, "h", "w", dynamic_dims={"bnp"})]
    images_spatial_crop: Annotated[torch.Tensor, TensorShape("bn", 2)]
```
**EN:** Class `DeepseekVL2ImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `DeepseekVL2ImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `DeepseekVL2VImageEmbeddingInputs` (lines 81-90)
```python
class DeepseekVL2VImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - f: Image feature size
        - h: Hidden size (must match language model backbone)
    """

    type: Literal["image_embeds"]
    data: Annotated[torch.Tensor | list[torch.Tensor], TensorShape("bn", "f", "h")]
```
**EN:** Class `DeepseekVL2VImageEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `DeepseekVL2VImageEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `MlpProjector` (lines 98-154)
```python
class MlpProjector(nn.Module):
    def __init__(self, cfg: MlpProjectorConfig):
        super().__init__()

        self.cfg = cfg
        self.projector_type = cfg.projector_type
        assert not cfg.token_pooling, "Token pooling is not supported currently."

        if self.projector_type == "downsample_mlp_gelu":
            mlp_depth = cfg.depth
            mlp_ratio = cfg.mlp_ratio
            modules = [
                nn.Linear(
                    cfg.input_dim * cfg.downsample_ratio * cfg.downsample_ratio,
                    cfg.n_embed * mlp_ratio,
                )
            ]
            for _ in range(1, mlp_depth - 1):
                modules.append(nn.GELU())
                modules.append(
                    nn.Linear(cfg.n_embed * mlp_ratio, cfg.n_embed * mlp_ratio)
                )
            modules.append(nn.GELU())
            modules.append(nn.Linear(cfg.n_embed * mlp_ratio, cfg.n_embed))
            modules = nn.Sequential(*modules)
```
**EN:** Class `MlpProjector` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `MlpProjector` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `MlpProjector.__init__` (lines 99-130)
```python
    def __init__(self, cfg: MlpProjectorConfig):
        super().__init__()

        self.cfg = cfg
        self.projector_type = cfg.projector_type
        assert not cfg.token_pooling, "Token pooling is not supported currently."

        if self.projector_type == "downsample_mlp_gelu":
            mlp_depth = cfg.depth
            mlp_ratio = cfg.mlp_ratio
            modules = [
                nn.Linear(
                    cfg.input_dim * cfg.downsample_ratio * cfg.downsample_ratio,
                    cfg.n_embed * mlp_ratio,
                )
            ]
            for _ in range(1, mlp_depth - 1):
                modules.append(nn.GELU())
                modules.append(
                    nn.Linear(cfg.n_embed * mlp_ratio, cfg.n_embed * mlp_ratio)
                )
            modules.append(nn.GELU())
            modules.append(nn.Linear(cfg.n_embed * mlp_ratio, cfg.n_embed))
            modules = nn.Sequential(*modules)
        elif self.projector_type == "linear":
            modules = nn.Linear(cfg.input_dim, cfg.n_embed)
        else:
            raise NotImplementedError(
                f"Unsupported projector type: {cfg.projector_type}"
            )

        self.layers = modules
```
**EN:** Method `MlpProjector.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `MlpProjector.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `MlpProjector.forward` (lines 132-154)
```python
    def forward(self, x):
        bs, hw, input_dim = x.shape
        if self.projector_type == "downsample_mlp_gelu":
            h = w = int((hw) ** 0.5)
            """compute padding"""
            if h % self.cfg.downsample_ratio:
                pad = self.cfg.downsample_ratio - h % self.cfg.downsample_ratio
            else:
                pad = 0
            x = x.reshape(bs, h, w, input_dim)
            if pad > 0:
                x = F.pad(x, (0, 0, 0, pad, 0, pad), "constant", 0)
            """4 to 1 concat"""
            x = x.permute(0, 3, 1, 2)  # B, C, H, W
            x = F.unfold(
                x,
                kernel_size=self.cfg.downsample_ratio,
                stride=self.cfg.downsample_ratio,
                padding=0,
            )  # B, C*4, HW // 4
            x = x.permute(0, 2, 1)

        return self.layers(x)
```
**EN:** Method `MlpProjector.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `MlpProjector.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `DeepseekVL2ProcessingInfo` (lines 157-201)
```python
class DeepseekVL2ProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(DeepseekVLV2Config)

    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(**kwargs)

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def get_num_image_tokens(
        self, *, image_width: int, image_height: int, cropping: bool = True
    ) -> int:
        hf_processor = self.get_hf_processor()
        image_size = hf_processor.image_size
        patch_size = hf_processor.patch_size
        downsample_ratio = hf_processor.downsample_ratio

        if cropping:
            best_width, best_height = hf_processor.select_best_resolution(
                (image_width, image_height)
            )
            num_width_tiles, num_height_tiles = (
                best_width // image_size,
                best_height // image_size,
```
**EN:** Class `DeepseekVL2ProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_hf_processor, get_supported_mm_limits, get_num_image_tokens, get_image_size_with_most_features.
**CN:** 类 `DeepseekVL2ProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_hf_processor, get_supported_mm_limits, get_num_image_tokens, get_image_size_with_most_features。

### Method `DeepseekVL2ProcessingInfo.get_hf_config` (lines 158-159)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config(DeepseekVLV2Config)
```
**EN:** Method `DeepseekVL2ProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekVL2ProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `DeepseekVL2ProcessingInfo.get_hf_processor` (lines 161-162)
```python
    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(**kwargs)
```
**EN:** Method `DeepseekVL2ProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekVL2ProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `DeepseekVL2ProcessingInfo.get_supported_mm_limits` (lines 164-165)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}
```
**EN:** Method `DeepseekVL2ProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekVL2ProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Method `DeepseekVL2ProcessingInfo.get_num_image_tokens` (lines 167-190)
```python
    def get_num_image_tokens(
        self, *, image_width: int, image_height: int, cropping: bool = True
    ) -> int:
        hf_processor = self.get_hf_processor()
        image_size = hf_processor.image_size
        patch_size = hf_processor.patch_size
        downsample_ratio = hf_processor.downsample_ratio

        if cropping:
            best_width, best_height = hf_processor.select_best_resolution(
                (image_width, image_height)
            )
            num_width_tiles, num_height_tiles = (
                best_width // image_size,
                best_height // image_size,
            )
        else:
            num_width_tiles = num_height_tiles = 1

        h = w = math.ceil((image_size // patch_size) / downsample_ratio)

        global_views_tokens = h * (w + 1)
        local_views_tokens = (num_height_tiles * h) * (num_width_tiles * w + 1)
        return global_views_tokens + local_views_tokens + 1
```
**EN:** Method `DeepseekVL2ProcessingInfo.get_num_image_tokens` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekVL2ProcessingInfo.get_num_image_tokens` 封装了该模块中的一段可复用核心逻辑。

### Class `DeepseekVL2DummyInputsBuilder` (lines 204-232)
```python
class DeepseekVL2DummyInputsBuilder(BaseDummyInputsBuilder[DeepseekVL2ProcessingInfo]):
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

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=max_image_size.width,
                height=max_image_size.height,
```
**EN:** Class `DeepseekVL2DummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[DeepseekVL2ProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `DeepseekVL2DummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[DeepseekVL2ProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `DeepseekVL2DummyInputsBuilder.get_dummy_text` (lines 205-211)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        image_token = processor.image_token

        return image_token * num_images
```
**EN:** Method `DeepseekVL2DummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekVL2DummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `DeepseekVL2DummyInputsBuilder.get_dummy_mm_data` (lines 213-232)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)

        max_image_size = self.info.get_image_size_with_most_features()

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=max_image_size.width,
                height=max_image_size.height,
                num_images=num_images,
                overrides=image_overrides,
            )
        }
```
**EN:** Method `DeepseekVL2DummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekVL2DummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `DeepseekVL2MultiModalProcessor` (lines 235-323)
```python
class DeepseekVL2MultiModalProcessor(
    BaseMultiModalProcessor[DeepseekVL2ProcessingInfo]
):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        if not mm_data:
            tokenizer = self.info.get_tokenizer()
            return tokenizer(prompt, add_special_tokens=True, return_tensors="pt")

        processed_outputs = super()._call_hf_processor(
            prompt=prompt,
            mm_data=mm_data,
            mm_kwargs=mm_kwargs,
            tok_kwargs=tok_kwargs,
        )

        processed_outputs["num_patches"] = (
            processed_outputs["images_spatial_crop"].prod(-1) + 1
        )
```
**EN:** Class `DeepseekVL2MultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[DeepseekVL2ProcessingInfo]. Key methods include _call_hf_processor, _get_mm_fields_config, _get_prompt_updates, _cached_apply_hf_processor.
**CN:** 类 `DeepseekVL2MultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[DeepseekVL2ProcessingInfo]。 关键方法包括 _call_hf_processor, _get_mm_fields_config, _get_prompt_updates, _cached_apply_hf_processor。

### Class `DeepseekVLV2ForCausalLM` (lines 331-626)
```python
@MULTIMODAL_REGISTRY.register_processor(
    DeepseekVL2MultiModalProcessor,
    info=DeepseekVL2ProcessingInfo,
    dummy_inputs=DeepseekVL2DummyInputsBuilder,
)
class DeepseekVLV2ForCausalLM(nn.Module, SupportsMultiModal, SupportsPP):
    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            "language.": "language_model.",
        }
    )

    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<image>"

        raise ValueError("Only image modality is supported")

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config: DeepseekVLV2Config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
```
**EN:** Class `DeepseekVLV2ForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods include get_placeholder_str, __init__, _get_parent_and_attr, patch_vit_for_tp, _init_vision_module, _parse_and_validate_image_input.
**CN:** 类 `DeepseekVLV2ForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 关键方法包括 get_placeholder_str, __init__, _get_parent_and_attr, patch_vit_for_tp, _init_vision_module, _parse_and_validate_image_input。

### Method `DeepseekVLV2ForCausalLM.get_placeholder_str` (lines 339-343)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<image>"

        raise ValueError("Only image modality is supported")
```
**EN:** Method `DeepseekVLV2ForCausalLM.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekVLV2ForCausalLM.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `DeepseekVLV2ForCausalLM.__init__` (lines 345-398)
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
        self.image_token_id: int = tokenizer.vocab[_IMAGE_TOKEN]

        with self._mark_tower_model(vllm_config, "image"):
            self.vision = self._init_vision_module(
                self.vision_config, quant_config, maybe_prefix(prefix, "vision")
            )

            self.projector = MlpProjector(self.projector_config)
            self.tile_tag = config.tile_tag
            self.global_view_pos = config.global_view_pos

            # special token for image token sequence format
            embed_std = 1 / torch.sqrt(
                torch.tensor(self.projector_config.n_embed, dtype=torch.float32)
            )
            if self.tile_tag == "2D":
                # <|view_seperator|>, <|\n|>
                self.image_newline = nn.Parameter(
                    torch.randn(self.projector_config.n_embed) * embed_std
                )
                # This is a typo in original implementation
                self.view_seperator = nn.Parameter(
                    torch.randn(self.projector_config.n_embed) * embed_std
                )
            else:
                raise ValueError(
                    f"Only 2D tile_tag is supported currently, got: {self.tile_tag}"
                )

        with self._mark_language_model(vllm_config):
            self.language_model = init_vllm_registered_model(
                vllm_config=vllm_config,
                hf_config=self.text_config,
                prefix=maybe_prefix(prefix, "language"),
            )

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )
```
**EN:** Method `DeepseekVLV2ForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `DeepseekVLV2ForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `DeepseekVLV2ForCausalLM.patch_vit_for_tp` (lines 409-429)
```python
    def patch_vit_for_tp(self, vit: torch.nn.Module, quant_config: QuantizationConfig):
        try:
            import timm
        except ImportError as e:
            raise ImportError("Please install timm") from e

        for name, module in vit.named_modules():
            if isinstance(module, nn.Linear):
                parent, attr_name = self._get_parent_and_attr(vit, name)
                if isinstance(parent, timm.layers.Mlp) and attr_name == "fc1":
                    new_linear = replace_linear_class(
                        module, "colwise", quant_config, prefix=name
                    )
                    setattr(parent, attr_name, new_linear)
                elif isinstance(parent, timm.layers.Mlp) and attr_name == "fc2":
                    new_linear = replace_linear_class(
                        module, "rowwise", quant_config, prefix=name
                    )
                    setattr(parent, attr_name, new_linear)

        return vit
```
**EN:** Method `DeepseekVLV2ForCausalLM.patch_vit_for_tp` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `DeepseekVLV2ForCausalLM.patch_vit_for_tp` 封装了该模块中的一段可复用核心逻辑。

### Method `DeepseekVLV2ForCausalLM.embed_multimodal` (lines 593-598)
```python
    def embed_multimodal(self, **kwargs: object) -> MultiModalEmbeddings:
        image_input = self._parse_and_validate_image_input(**kwargs)
        if image_input is None:
            return []
        vision_embeddings = self._process_image_input(image_input)
        return vision_embeddings
```
**EN:** Method `DeepseekVLV2ForCausalLM.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `DeepseekVLV2ForCausalLM.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。
- **KV cache / KV 缓存**
  - **EN:** The runtime stores key/value state so decoding can reuse prior computation efficiently.
  - **CN:** 运行时会保存 key/value 状态，以便解码阶段高效复用历史计算。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, Literal, TypeAlias`, `from einops import rearrange, repeat`, `import timm`, `import timm`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `import torch.nn.functional as F`, `from transformers import BatchFeature`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.distributed import get_tensor_model_parallel_world_size`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.models.transformers.utils import replace_linear_class`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import (`, `from vllm.multimodal.processing import BaseDummyInputsBuilder`, `from vllm.multimodal.processing.processor import (`, `from vllm.sequence import IntermediateTensors`
- **Module note / 模块说明**: **EN:** Inference-only Deepseek-VL2 model compatible with HuggingFace weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only Deepseek-VL2 model compatible with HuggingFace weights.。
