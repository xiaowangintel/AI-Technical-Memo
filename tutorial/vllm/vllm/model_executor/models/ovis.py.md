# ovis.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/ovis.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Ovis multimodal model adapter used for inference in vLLM. The module docstring summarizes it as: "PyTorch Ovis model." / 实现 Ovis 在 vLLM 中用于推理的多模态模型适配器。 模块文档字符串还将其概括为：“PyTorch Ovis model。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-63)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

# adapted from https://github.com/huggingface/transformers/blob/v4.39.3/src/transformers/models/ovis/modeling_ovis.py
# Copyright 2023 The vLLM team.
# Copyright 2023 HuggingFace Inc. team. All rights reserved.
#
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
# ... omitted for brevity ...
)
from vllm.multimodal.parse import ImageSize, MultiModalDataItems
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
)
from vllm.renderers import TokenizeParams
from vllm.sequence import IntermediateTensors
from vllm.transformers_utils.processors.ovis import OvisProcessor
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP

# Cannot find the following number from hf config.
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, torch.nn.functional, transformers supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.linear connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, torch.nn.functional, transformers 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.linear 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `IMAGE_TOKEN, IMAGE_INDICATOR_IDS, IMAGE_PAD_TOKEN_MAP ...` (lines 64-76)
```python
IMAGE_TOKEN = "<image>"
IMAGE_INDICATOR_IDS = [-301, -302, -303, -304, -305]

IMAGE_PAD_TOKEN_MAP = {
    "gemma2": "<unused0>",
    "llama": "<|reserved_special_token_0|>",
    "qwen2": "<|image_pad|>",
}
IMAGE_PAD_TOKEN_ID_MAP = {
    "gemma2": 7,
    "llama": 128002,
    "qwen2": 151655,
}
```
**EN:** This assignment block centers on `IMAGE_TOKEN, IMAGE_INDICATOR_IDS, IMAGE_PAD_TOKEN_MAP ...` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `IMAGE_TOKEN, IMAGE_INDICATOR_IDS, IMAGE_PAD_TOKEN_MAP ...` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Function `st_argmax` (lines 79-84)
```python
def st_argmax(y_soft: torch.Tensor, dim: int):  # straight-through softmax
    index = y_soft.argmax(dim, keepdim=True)
    return torch.zeros_like(
        y_soft,
        memory_format=torch.legacy_contiguous_format,
    ).scatter_(dim, index, 1.0)
```
**EN:** The function `st_argmax` helps provide a reusable helper for the surrounding model code. Its main inputs are `y_soft`, `dim`.
**CN:** 函数 `st_argmax` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `y_soft`、`dim`。

### Class `VisualTokenizer` (lines 87-215)
```python
class VisualTokenizer(torch.nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.backbone = self._init_backbone(
            config=config,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "backbone"),
        )
        # reserved tokens for IMAGE_INDICATORS
        head_dim = config.vocab_size - len(IMAGE_INDICATOR_IDS)
        self.head = torch.nn.Sequential(
            ReplicatedLinear(
                config.backbone_config.hidden_size
                * config.hidden_stride
# ... omitted for brevity ...
    def _init_backbone(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> nn.Module:
        model_type = config.backbone_config.model_type
        if model_type == "aimv2":
            # No post rms_norm in Ovis2's AIMv2 ViT.
            return AIMv2Model(
                config=config.backbone_config,
                quant_config=quant_config,
                require_post_norm=False,
                prefix=prefix,
# ... omitted for brevity ...
        features = self.backbone(pixel_values)
        if self.config.drop_cls_token:
            features = features[:, 1:, :]

        # merge number of `hidden_stride * hidden_stride` hidden states together
        # to reduce token sequence length
        # e.g., for hidden_stride=2, this leads to a token length reduction:
        # 1024 -> 256 for aimv2
        if self.config.hidden_stride > 1:
            # this `d` maybe different from the above `d`
            n, L, d = features.shape
            sqrt_l = int(L**0.5)
            assert sqrt_l**2 == L, (
                "The token sequence length should be a perfect square."
            )
            features = features.reshape(n, sqrt_l, sqrt_l, d)
            pl = (
                self.config.hidden_stride - (sqrt_l % self.config.hidden_stride)
# ... omitted for brevity ...
    def forward(self, pixel_values: torch.Tensor) -> torch.Tensor:
        """[BatchSize, ImageShape] -> [BatchSize, Token, VocabSize]"""
        features = self.encode(pixel_values)
        logits = self.head(features)
        tokens = self.tokenize(logits)
        # tokens' shape is [BatchSize, #Token, VocabSize-5], so padding with
        # [BatchSize, #Token, 5], after which, tokens' shape should become
        # [BatchSize, #Token, VocabSize]
        tokens = torch.nn.functional.pad(
            tokens,
            (0, len(IMAGE_INDICATOR_IDS)),
            mode="constant",
            value=0,
        )
        return tokens
```
**EN:** Defines `VisualTokenizer`, a supporting module used by the surrounding model implementation. It inherits from torch.nn.Module. Key methods such as `__init__`, `_init_backbone`, `dtype`, `device`, `tokenize` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `VisualTokenizer`，它是一个被周边模型实现复用的支撑模块。 它继承自 torch.nn.Module。 `__init__`, `_init_backbone`, `dtype`, `device`, `tokenize` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OvisImagePatchInputs` (lines 218-231)
```python
class OvisImagePatchInputs(TensorSchema):
    """
    Dimensions:
        - bnp: Batch size * number of images * number of patches
        - h: Height of each patch
        - w: Width of each patch
        - patch_indicators: Batch size * (number of patches + 1)
        - bn: Batch size * number of images
    """

    type: Literal["image_patches"]
    flat_data: Annotated[torch.Tensor, TensorShape("bnp", 3, "h", "w")]
    indicator_tokens: Annotated[torch.Tensor, TensorShape("patch_indicators")]
    patches_per_image: Annotated[list[int], TensorShape("bn")]
```
**EN:** Defines `OvisImagePatchInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bnp: Batch size * number of images * number of patches - h: Height of each patch - w: Width of each patch - patch_indicators: Batch size * (number of patches + 1) - bn: Batch size * number of images."
**CN:** 定义 `OvisImagePatchInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bnp: Batch size * number of images * number of patches - h: Height of each patch - w: Width of each patch - patch_indicators: Batch size * (number of patches + 1) - bn: Batch size * number of images。”

### Class `VisualEmbedding` (lines 235-256)
```python
class VisualEmbedding(torch.nn.Embedding):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    def forward(self, visual_tokens: Tensor) -> Tensor:
        if visual_tokens.dtype in [
            torch.int8,
            torch.int16,
            torch.int32,
            torch.int64,
            torch.long,
        ]:
            return super().forward(visual_tokens)
        return torch.matmul(visual_tokens, self.weight)

    @property
    def device(self):
        return self.weight.device

    @property
    def dtype(self):
        return self.weight.dtype
```
**EN:** Defines `VisualEmbedding`, a supporting module used by the surrounding model implementation. It inherits from torch.nn.Embedding. Key methods such as `__init__`, `forward`, `device`, `dtype` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `VisualEmbedding`，它是一个被周边模型实现复用的支撑模块。 它继承自 torch.nn.Embedding。 `__init__`, `forward`, `device`, `dtype` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OvisProcessingInfo` (lines 259-297)
```python
class OvisProcessingInfo(BaseProcessingInfo):
    def get_hf_processor(self, **kwargs: object):
        return self.ctx.get_hf_processor(
            OvisProcessor,
            image_pad_token=self.get_image_pad_token(),
            image_segment_len=self.get_image_segment_len(),
            **kwargs,
        )

    def get_default_tok_params(self) -> TokenizeParams:
        return super().get_default_tok_params().with_kwargs(add_special_tokens=False)

    def get_image_segment_len(self) -> int:
        visual_tokenizer_config = self.get_hf_config().visual_tokenizer_config
        image_size = visual_tokenizer_config.backbone_config.image_size
        patch_size = visual_tokenizer_config.backbone_config.patch_size
        hidden_stride = visual_tokenizer_config.hidden_stride
        patch_grid_length = math.ceil(image_size / patch_size)
        assert patch_grid_length % hidden_stride == 0, (
            f"patch_grid_length {patch_grid_length} is not divisible by "
            f"hidden_stride {hidden_stride}"
        )
        # minus 1 for presented image token
        return (patch_grid_length // hidden_stride) ** 2 - 1

    def get_image_pad_token(self) -> str:
        hf_text_config = self.get_hf_config().get_text_config()
        text_model_type = hf_text_config.model_type
        return IMAGE_PAD_TOKEN_MAP.get(text_model_type)

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}

    def get_image_size_with_most_features(self) -> ImageSize:
        height, width = self.get_hf_processor().get_image_size()
        hs = self.get_hf_config().visual_tokenizer_config.hidden_stride
        # NOTE(Isotr0py): 9 is `max_partition` hardcoded in original code
        # https://huggingface.co/AIDC-AI/Ovis2-1B/blob/main/modeling_ovis.py#L96
        return ImageSize(width=width * hs * 9, height=height * hs * 9)
```
**EN:** Defines `OvisProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseProcessingInfo. Key methods such as `get_hf_processor`, `get_default_tok_params`, `get_image_segment_len`, `get_image_pad_token`, `get_supported_mm_limits` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OvisProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseProcessingInfo。 `get_hf_processor`, `get_default_tok_params`, `get_image_segment_len`, `get_image_pad_token`, `get_supported_mm_limits` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OvisDummyInputsBuilder` (lines 300-325)
```python
class OvisDummyInputsBuilder(BaseDummyInputsBuilder[OvisProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        return IMAGE_TOKEN * num_images

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)

        target_width, target_height = self.info.get_image_size_with_most_features()

        image_overrides = mm_options.get("image")

        mm_data = {
            "image": self._get_dummy_images(
                width=target_width,
                height=target_height,
                num_images=num_images,
                overrides=image_overrides,
            ),
        }
        return mm_data
```
**EN:** Defines `OvisDummyInputsBuilder`, a supporting module used by the surrounding model implementation. It inherits from BaseDummyInputsBuilder[OvisProcessingInfo]. Key methods such as `get_dummy_text`, `get_dummy_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OvisDummyInputsBuilder`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseDummyInputsBuilder[OvisProcessingInfo]。 `get_dummy_text`, `get_dummy_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OvisMultiModalProcessor` (lines 328-412)
```python
class OvisMultiModalProcessor(BaseMultiModalProcessor[OvisProcessingInfo]):
    def image_indicators_to_visual_tokens(
        self,
        image_indicators: list[int],
    ) -> list[int]:
        """
        Filter image indicators placeholders and convert them to corresponding
        tokens in visual tokenizer.
        For example, [-301, -300, -302, -300, -303, -300, -304, -300, -305]
        should return [vocab_size-1, vocab_size-2, ..., vocab_size-5]
        """
        hf_config = self.info.get_hf_config()
        vte_vocab_size = hf_config.visual_tokenizer_config.vocab_size
        # -300 is image_atom token, filter them out
        return [vte_vocab_size + x + 300 for x in image_indicators if x < -300]
# ... omitted for brevity ...
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        if not mm_data:
            # Avoid warning from HF logger for text-only input
            tokenizer = self.info.get_tokenizer()
            prompt_ids = tokenizer.encode(prompt, add_special_tokens=False)
            return BatchFeature(dict(input_ids=[prompt_ids]), tensor_type="pt")

        processed_outputs = super()._call_hf_processor(
            prompt=prompt,
            mm_data=mm_data,
            mm_kwargs=mm_kwargs,
            tok_kwargs=tok_kwargs,
        )
# ... omitted for brevity ...
    def _apply_hf_processor_tokens_only(
        self,
        prompt_tokens: list[int],
    ) -> list[int]:
        return prompt_tokens
# ... omitted for brevity ...
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        return dict(
            pixel_values=MultiModalFieldConfig.batched("image"),
            grids=MultiModalFieldConfig.batched("image"),
            indicator_tokens=MultiModalFieldConfig.batched("image"),
        )
```
**EN:** Defines `OvisMultiModalProcessor`, a supporting module used by the surrounding model implementation. It inherits from BaseMultiModalProcessor[OvisProcessingInfo]. Key methods such as `image_indicators_to_visual_tokens`, `_call_hf_processor`, `_apply_hf_processor_tokens_only`, `_get_mm_fields_config`, `_get_prompt_updates` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OvisMultiModalProcessor`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseMultiModalProcessor[OvisProcessingInfo]。 `image_indicators_to_visual_tokens`, `_call_hf_processor`, `_apply_hf_processor_tokens_only`, `_get_mm_fields_config`, `_get_prompt_updates` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Ovis` (lines 415-559)
```python
@MULTIMODAL_REGISTRY.register_processor(
    OvisMultiModalProcessor,
    info=OvisProcessingInfo,
    dummy_inputs=OvisDummyInputsBuilder,
)
class Ovis(nn.Module, SupportsMultiModal, SupportsPP):
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config: PretrainedConfig = config

        with self._mark_language_model(vllm_config):
            self.llm = init_vllm_registered_model(
                vllm_config=vllm_config.with_hf_config(config.get_text_config()),
                prefix=maybe_prefix(prefix, "llm"),
            )

        with self._mark_tower_model(vllm_config, "image"):
            self.visual_tokenizer = VisualTokenizer(
                config=config.visual_tokenizer_config,
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "visual_tokenizer"),
            )
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        if intermediate_tensors is not None:
            inputs_embeds = None

        # up until here we have an inputs_embeds 100% numerical identity
        # between the OG HF Transformers implementation and ours
        hidden_states = self.llm(
            input_ids=input_ids,
            positions=positions,
            intermediate_tensors=intermediate_tensors,
            inputs_embeds=inputs_embeds,
        )
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        return self.llm.compute_logits(hidden_states)
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights)
```
**EN:** Defines `Ovis`, a supporting module used by the surrounding model implementation. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods such as `get_placeholder_str`, `__init__`, `_parse_and_validate_image_input`, `_process_image_input`, `embed_multimodal` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Ovis`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 `get_placeholder_str`, `__init__`, `_parse_and_validate_image_input`, `_process_image_input`, `embed_multimodal` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Quantized weights**
  - **EN:** Quantization hooks allow the model to load compressed checkpoints or specialized linear layers.
  - **CN:** 量化钩子使模型能够加载压缩检查点或专用线性层。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。
- **Position encoding**
  - **EN:** Rotary or related positional logic appears in the file to preserve token order information efficiently.
  - **CN:** 文件中出现了 Rotary 等位置编码逻辑，用于高效保留 token 顺序信息。

## Dependencies / 依赖关系
- **Standard library**: math, collections.abc, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, torch.nn.functional, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.linear, vllm.model_executor.layers.quantization, vllm.model_executor.models.aimv2, vllm.model_executor.models.siglip, vllm.model_executor.models.utils
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
