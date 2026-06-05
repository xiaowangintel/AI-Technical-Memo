# ovis2_5.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/ovis2_5.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Ovis2 5 multimodal model adapter used for inference in vLLM. The module docstring summarizes it as: "PyTorch Ovis model." / 实现 Ovis2 5 在 vLLM 中用于推理的多模态模型适配器。 模块文档字符串还将其概括为：“PyTorch Ovis model。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-44)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""PyTorch Ovis model."""

from collections.abc import Iterable, Mapping
from functools import partial
from typing import Annotated, Literal

import torch
import torch.nn as nn
from transformers import BaseImageProcessor, BatchFeature, PretrainedConfig

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.linear import ReplicatedLinear
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.models.ovis import VisualEmbedding
# ... omitted for brevity ...
    MultiModalKwargsItems,
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
from vllm.transformers_utils.processors.ovis2_5 import Ovis2_5Processor
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsPP
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, transformers supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.linear connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, transformers 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.linear 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `IMAGE_TOKEN, VIDEO_TOKEN, INDICATOR_IDS ...` (lines 45-48)
```python
IMAGE_TOKEN = "<image>"
VIDEO_TOKEN = "<video>"
INDICATOR_IDS = [151672, 151673, 151674, 151675]
IMAGE_PAD_TOKEN_ID = 151655
```
**EN:** This assignment block centers on `IMAGE_TOKEN, VIDEO_TOKEN, INDICATOR_IDS ...` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `IMAGE_TOKEN, VIDEO_TOKEN, INDICATOR_IDS ...` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Class `Ovis2_5ImagePatchInputs` (lines 51-64)
```python
class Ovis2_5ImagePatchInputs(TensorSchema):
    """
    Dimensions:
        - bnp: Batch size * number of images * number of patches
        - patch_size: patch_size_x * patch_size_y * num_channels
        - patch_indicators: Batch size * (number of patches + 1)
        - bn: Batch size * number of images
    """

    type: Literal["image_patches"]
    flat_data: Annotated[torch.Tensor, TensorShape("bnp", "patch_size")]
    indicator_tokens: Annotated[torch.Tensor, TensorShape("patch_indicators")]
    patches_per_item: Annotated[list[int], TensorShape("bn")]
    grids: Annotated[torch.Tensor, TensorShape("bn", 3)]
```
**EN:** Defines `Ovis2_5ImagePatchInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bnp: Batch size * number of images * number of patches - patch_size: patch_size_x * patch_size_y * num_channels - patch_indicators: Batch size * (number of patches + 1) - bn: Batch size * number of images."
**CN:** 定义 `Ovis2_5ImagePatchInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bnp: Batch size * number of images * number of patches - patch_size: patch_size_x * patch_size_y * num_channels - patch_indicators: Batch size * (number of patches + 1) - bn: Batch size * number of images。”

### Class `Ovis2_5VideoPatchInputs` (lines 68-81)
```python
class Ovis2_5VideoPatchInputs(TensorSchema):
    """
    Dimensions:
        - bnp: Batch size * number of videos * number of patches
        - patch_size: patch_size_x * patch_size_y * num_channels
        - patch_indicators: Batch size * (number of patches + 1)
        - bn: Batch size * number of videos
    """

    type: Literal["video_patches"]
    flat_data: Annotated[torch.Tensor, TensorShape("bnp", "patch_size")]
    indicator_tokens: Annotated[torch.Tensor, TensorShape("patch_indicators")]
    patches_per_item: Annotated[list[int], TensorShape("bn")]
    grids: Annotated[torch.Tensor, TensorShape("bn", 3)]
```
**EN:** Defines `Ovis2_5VideoPatchInputs`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from TensorSchema. Docstring hint: "Dimensions: - bnp: Batch size * number of videos * number of patches - patch_size: patch_size_x * patch_size_y * num_channels - patch_indicators: Batch size * (number of patches + 1) - bn: Batch size * number of videos."
**CN:** 定义 `Ovis2_5VideoPatchInputs`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 TensorSchema。 文档提示：“Dimensions: - bnp: Batch size * number of videos * number of patches - patch_size: patch_size_x * patch_size_y * num_channels - patch_indicators: Batch size * (number of patches + 1) - bn: Batch size * number of videos。”

### Class `VisualTokenizer` (lines 85-168)
```python
class VisualTokenizer(torch.nn.Module):
    """
    VIT
    """

    def __init__(
        self,
        config: PretrainedConfig,
        visual_vocab_size: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.vit = self._init_backbone(
            config=config,
            quant_config=quant_config,
            prefix=f"{prefix}.vit",
        )
        # reserved tokens for INDICATOR_IDS
        head_dim = visual_vocab_size - len(INDICATOR_IDS)
        self.head = torch.nn.Sequential(
            ReplicatedLinear(
                self.config.hidden_size * self.config.hidden_stride**2,
# ... omitted for brevity ...
    def _init_backbone(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        model_type = config.model_type
        if model_type == "siglip2_navit":
            return Siglip2NavitModel(
                config=config,
                quant_config=quant_config,
                prefix=prefix,
            )
        raise ValueError(f"Unsupported visual tokenizer model_type: {model_type}")
# ... omitted for brevity ...
    def encode(
        self, pixel_values: torch.Tensor, grid_thws: torch.Tensor
    ) -> torch.Tensor:
        features = self.vit(pixel_values, grid_thws)
        # refer to qwen2.5-vl patchmerger
        seq_len, _ = features.shape
        features = features.reshape(seq_len // (self.config.hidden_stride**2), -1)

        return features
# ... omitted for brevity ...
    def forward(
        self, pixel_values: torch.Tensor, grid_thws: torch.Tensor
    ) -> torch.Tensor:
        features = self.encode(pixel_values, grid_thws)
        logits = self.head(features)
        tokens = self.tokenize(logits)
        # tokens' shape is [#Token, VocabSize-4],
        # so padding with [#Token, 4], after which,
        # tokens' shape should become [#Token, VocabSize];
        tokens = torch.nn.functional.pad(
            tokens,
            (0, len(INDICATOR_IDS)),
            mode="constant",
            value=0,
        )
        return tokens
```
**EN:** Defines `VisualTokenizer`, a supporting module used by the surrounding model implementation. It inherits from torch.nn.Module. Key methods such as `__init__`, `_init_backbone`, `dtype`, `device`, `tokenize` show where construction, forward execution, or weight adaptation happens. Docstring hint: "VIT."
**CN:** 定义 `VisualTokenizer`，它是一个被周边模型实现复用的支撑模块。 它继承自 torch.nn.Module。 `__init__`, `_init_backbone`, `dtype`, `device`, `tokenize` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“VIT。”

### Class `Ovis2_5ProcessingInfo` (lines 171-274)
```python
class Ovis2_5ProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config()

    def get_hf_processor(self, **kwargs):
        vit_config = self.get_hf_config().vit_config
        return self.ctx.get_hf_processor(
            Ovis2_5Processor,
            patch_size=vit_config.patch_size,
            hidden_stride=vit_config.hidden_stride,
            temporal_patch_size=vit_config.temporal_patch_size,
        )
# ... omitted for brevity ...
    def get_default_tok_params(self) -> TokenizeParams:
        return super().get_default_tok_params().with_kwargs(add_special_tokens=False)
# ... omitted for brevity ...
    def get_image_processor(self) -> BaseImageProcessor:
        return self.get_hf_processor().image_processor  # type: ignore
```
**EN:** Defines `Ovis2_5ProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseProcessingInfo. Key methods such as `get_hf_config`, `get_hf_processor`, `get_default_tok_params`, `get_image_processor`, `get_supported_mm_limits` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Ovis2_5ProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseProcessingInfo。 `get_hf_config`, `get_hf_processor`, `get_default_tok_params`, `get_image_processor`, `get_supported_mm_limits` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Ovis2_5DummyInputsBuilder` (lines 277-315)
```python
class Ovis2_5DummyInputsBuilder(BaseDummyInputsBuilder[Ovis2_5ProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)
        return IMAGE_TOKEN * num_images + VIDEO_TOKEN * num_videos

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)
        num_videos = mm_counts.get("video", 0)

        target_width, target_height = self.info.get_image_size_with_most_features()
        target_num_frames = self.info.get_num_frames_with_most_features(
            seq_len, mm_counts
        )

        image_overrides = mm_options.get("image")
        video_overrides = mm_options.get("video")

        mm_data = {
            "image": self._get_dummy_images(
                width=target_width,
                height=target_height,
                num_images=num_images,
                overrides=image_overrides,
            ),
            "video": self._get_dummy_videos(
                width=target_width,
                height=target_height,
                num_frames=target_num_frames,
                num_videos=num_videos,
                overrides=video_overrides,
            ),
        }
        return mm_data
```
**EN:** Defines `Ovis2_5DummyInputsBuilder`, a supporting module used by the surrounding model implementation. It inherits from BaseDummyInputsBuilder[Ovis2_5ProcessingInfo]. Key methods such as `get_dummy_text`, `get_dummy_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Ovis2_5DummyInputsBuilder`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseDummyInputsBuilder[Ovis2_5ProcessingInfo]。 `get_dummy_text`, `get_dummy_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Ovis2_5MultiModalProcessor` (lines 318-432)
```python
class Ovis2_5MultiModalProcessor(BaseMultiModalProcessor[Ovis2_5ProcessingInfo]):
    def visual_indicators_to_visual_tokens(
        self,
        visual_indicators: list[int],
    ) -> list[int]:
        """
        Filter image indicators placeholders and convert them to corresponding
        tokens in visual tokenizer.
        """
        hf_config = self.info.get_hf_config()
        vte_vocab_size = hf_config.visual_vocab_size
        return [
            vte_vocab_size - len(INDICATOR_IDS) + (x - INDICATOR_IDS[0])
            for x in visual_indicators
            if x >= INDICATOR_IDS[0]
        ]
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
            video_pixel_values=MultiModalFieldConfig.batched("video"),
            video_indicator_tokens=MultiModalFieldConfig.batched("video"),
            video_grids=MultiModalFieldConfig.batched("video"),
        )
```
**EN:** Defines `Ovis2_5MultiModalProcessor`, a supporting module used by the surrounding model implementation. It inherits from BaseMultiModalProcessor[Ovis2_5ProcessingInfo]. Key methods such as `visual_indicators_to_visual_tokens`, `_call_hf_processor`, `_apply_hf_processor_tokens_only`, `_get_mm_fields_config`, `_get_prompt_updates` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Ovis2_5MultiModalProcessor`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseMultiModalProcessor[Ovis2_5ProcessingInfo]。 `visual_indicators_to_visual_tokens`, `_call_hf_processor`, `_apply_hf_processor_tokens_only`, `_get_mm_fields_config`, `_get_prompt_updates` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Ovis2_5` (lines 435-652)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Ovis2_5MultiModalProcessor,
    info=Ovis2_5ProcessingInfo,
    dummy_inputs=Ovis2_5DummyInputsBuilder,
)
class Ovis2_5(nn.Module, SupportsMultiModal, SupportsPP):
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
                vllm_config=vllm_config.with_hf_config(config.text_config),
                prefix=maybe_prefix(prefix, "llm"),
            )

        with self._mark_tower_model(vllm_config, {"image", "video"}):
            self.visual_tokenizer = VisualTokenizer(
                config=config.vit_config,
                visual_vocab_size=config.visual_vocab_size,
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "visual_tokenizer"),
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

        # up until here we have a inputs_embeds 100% numerical identity
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
**EN:** Defines `Ovis2_5`, a supporting module used by the surrounding model implementation. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods such as `get_placeholder_str`, `__init__`, `_parse_and_validate_image_input`, `_parse_and_validate_video_input`, `_process_visual_input` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Ovis2_5`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 `get_placeholder_str`, `__init__`, `_parse_and_validate_image_input`, `_parse_and_validate_video_input`, `_process_visual_input` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **Standard library**: collections.abc, functools, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.linear, vllm.model_executor.layers.quantization, vllm.model_executor.models.ovis, vllm.model_executor.models.siglip2navit, vllm.model_executor.models.utils
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
