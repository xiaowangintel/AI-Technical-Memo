# multimodal.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/transformers/multimodal.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements Multimodal support code for the Transformers-based backend inside vLLM. The module docstring summarizes it as: "Transformers modeling backend mixin for multi-modal models." / 为 vLLM 内部基于 Transformers 的后端实现 Multimodal 相关支持代码。 模块文档字符串还将其概括为：“Transformers modeling backend mixin for multi-modal models。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-49)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

# Copyright 2024 The vLLM team.
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
"""Transformers modeling backend mixin for multi-modal models."""

# ... omitted for brevity ...
    PlaceholderRange,
)
from vllm.multimodal.parse import (
    ImageProcessorItems,
    MultiModalDataItems,
)
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    ProcessorInputs,
    TimingContext,
)
from vllm.platforms import current_platform
from vllm.sequence import IntermediateTensors
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, transformers supply framework primitives, while internal modules like vllm.compilation.decorators, vllm.config.utils, vllm.inputs, vllm.logger connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, transformers 这样的外部依赖提供基础框架能力，而 vllm.compilation.decorators, vllm.config.utils, vllm.inputs, vllm.logger 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### TYPE_CHECKING branch (lines 50-54)
```python
if TYPE_CHECKING:
    from transformers import BatchFeature, PreTrainedModel

    from vllm.config import VllmConfig
    from vllm.config.multimodal import BaseDummyOptions
```
**EN:** This conditional block keeps optional imports, type-only definitions, or backend-specific branches isolated from the hot runtime path.
**CN:** 这个条件分支把可选导入、仅类型定义或特定后端分支与主要运行路径隔离开。

### Top-level mapping `logger` (lines 56-56)
```python
logger = init_logger(__name__)
```
**EN:** This assignment initializes the module-level logger used for diagnostics, backend selection messages, and compatibility warnings.
**CN:** 这个赋值语句初始化了模块级日志记录器，用于输出诊断信息、后端选择消息和兼容性警告。

### Class `MultiModalProcessingInfo` (lines 59-78)
```python
class MultiModalProcessingInfo(BaseProcessingInfo):
    def get_supported_mm_limits(self):
        return {"image": None}

    def get_mm_max_tokens_per_item(self, seq_len, mm_counts):
        return {"image": self.get_max_image_tokens()}

    def get_max_image_tokens(self) -> int:
        width, height = self.get_max_image_size()
        processor = self.get_hf_processor()
        multimodal_config = self.ctx.model_config.multimodal_config
        mm_processor_kwargs = multimodal_config.mm_processor_kwargs or {}
        mm_tokens = processor._get_num_multimodal_tokens(
            image_sizes=([height, width],), **mm_processor_kwargs
        )
        image_tokens = mm_tokens["num_image_tokens"][0]
        return image_tokens

    def get_max_image_size(self):
        return 10_000, 10_000  # hardcode for arbitrary very large size
```
**EN:** Defines `MultiModalProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseProcessingInfo. Key methods such as `get_supported_mm_limits`, `get_mm_max_tokens_per_item`, `get_max_image_tokens`, `get_max_image_size` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MultiModalProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseProcessingInfo。 `get_supported_mm_limits`, `get_mm_max_tokens_per_item`, `get_max_image_tokens`, `get_max_image_size` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MultiModalDummyInputsBuilder` (lines 81-111)
```python
class MultiModalDummyInputsBuilder(BaseDummyInputsBuilder[MultiModalProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        processor = self.info.get_hf_processor()
        if "gemma3" in processor.__class__.__name__.lower():
            image_token = processor.boi_token
        else:
            image_token = getattr(processor, "image_token", "")
        return image_token * num_images

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, "BaseDummyOptions"],
    ) -> MultiModalDataDict:
        num_images = mm_counts.get("image", 0)

        target_width, target_height = self.info.get_max_image_size()

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=target_width,
                height=target_height,
                num_images=num_images,
                overrides=image_overrides,
            ),
        }
```
**EN:** Defines `MultiModalDummyInputsBuilder`, a supporting module used by the surrounding model implementation. It inherits from BaseDummyInputsBuilder[MultiModalProcessingInfo]. Key methods such as `get_dummy_text`, `get_dummy_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MultiModalDummyInputsBuilder`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseDummyInputsBuilder[MultiModalProcessingInfo]。 `get_dummy_text`, `get_dummy_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MultiModalProcessor` (lines 114-261)
```python
class MultiModalProcessor(BaseMultiModalProcessor[MultiModalProcessingInfo]):
    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        out_mm_kwargs: MultiModalKwargsItems,
    ):
        """
        Given the original multi-modal items for this modality
        and HF-processed data, output the updates to perform.

        The information returned by this method is used to update token inputs
        which bypass the HF processor. It is also used to update the output of
        HF processor if the HF process does not apply prompt updates to text
        inputs.

        Moreover, this information is critical to determine the token positions
        in order to construct  :class:`~vllm-multimodal.input.PlaceholderRange`
        for each multi-modal item.
        """
# ... omitted for brevity ...
    def _get_mm_fields_config(
        self,
        hf_inputs: "BatchFeature",
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        # HF Processors always return a mask but vLLM doesn't need it
        hf_inputs.pop("attention_mask", None)
        num_image_patches = hf_inputs.get("num_image_patches")
        mm_fields = {
            key: MultiModalFieldConfig.flat_from_sizes("image", num_image_patches)
            for key in hf_inputs
        }
        mm_fields["image_embeds"] = MultiModalFieldConfig.flat_from_sizes(
            "image", num_image_patches
# ... omitted for brevity ...
        mm_fields["image_grid_thw"] = MultiModalFieldConfig.batched("image")
        mm_fields["video_grid_thw"] = MultiModalFieldConfig.batched("image")
# ... omitted for brevity ...
    def _get_hf_mm_data(
        self,
        mm_items: MultiModalDataItems,
    ) -> tuple[Mapping[str, object], Mapping[str, object]]:
        """
        In contrast to the base class, this method always adds
        `return_mm_token_type_ids` to the processor data
        """
        processor_data, passthrough_data = super()._get_hf_mm_data(mm_items)
        processor_data["return_mm_token_type_ids"] = True
        return processor_data, passthrough_data
# ... omitted for brevity ...
    def apply(
        self,
        inputs: ProcessorInputs,
        timing_ctx: TimingContext,
    ) -> MultiModalInput:
        """
        Process multi-modal inputs to be used in vLLM.

        Apply HF Processor on prompt text and multi-modal data together,
        outputting token IDs and processed tensors.
        """
        prompt = inputs.prompt
        mm_items = inputs.mm_data_items
        hf_processor_mm_kwargs = inputs.hf_processor_mm_kwargs
        tokenization_kwargs = inputs.tokenization_kwargs

        with timing_ctx.record("apply_hf_processor"):
            hf_processor = self.info.get_hf_processor(**hf_processor_mm_kwargs)
            if not isinstance(prompt, str):
```
**EN:** Defines `MultiModalProcessor`, a supporting module used by the surrounding model implementation. It inherits from BaseMultiModalProcessor[MultiModalProcessingInfo]. Key methods such as `_get_prompt_updates`, `_get_mm_fields_config`, `_get_hf_mm_data`, `apply` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MultiModalProcessor`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseMultiModalProcessor[MultiModalProcessingInfo]。 `_get_prompt_updates`, `_get_mm_fields_config`, `_get_hf_mm_data`, `apply` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MultiModalMixin` (lines 264-533)
```python
class MultiModalMixin(SupportsMultiModal, SupportsMRoPE):
    supports_multimodal_raw_input_only = True

    def __init__(self, *, vllm_config: "VllmConfig", prefix: str = ""):
        # Skip SupportsMRoPE.__init__ and call the next class in MRO
        super(SupportsMRoPE, self).__init__(vllm_config=vllm_config, prefix=prefix)

    def _get_encoder_cls(
        self, modality: str = "image", **kwargs: dict
    ) -> type["PreTrainedModel"]:
        """
        Get the encoder class from the model.

        Args:
            kwargs: The kwargs to create the model.

        Returns:
            The encoder class.
        """
        with torch.device("meta"):
            model: PreTrainedModel = AutoModel.from_config(**kwargs)
        encoder_cls = type(model.get_encoder(modality=modality))
        logger.debug("Identified encoder class as: %s", encoder_cls)
        if type(model) is encoder_cls:
            raise ValueError(
                "Unable to infer vision encoder class from the model. "
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        # Gemma3 and PaliGemma needs `token_type_ids` to work correctly
        # Other models will not have `token_type_ids` in kwargs
        kwargs = {k: v for k, v in kwargs.items() if k == "token_type_ids"}
        # Positions shape handling for MRoPE models
        if self.model_config.uses_mrope:
            # [3, seq_len] -> [3, 1, seq_len]
            positions = positions[:, None]
        model_output = super().forward(
            input_ids, positions, intermediate_tensors, inputs_embeds, **kwargs
        )
        return model_output
# ... omitted for brevity ...
    def get_language_model(self) -> torch.nn.Module:
        """Transformers modeling backend multimodal classes do not contain a separate
        vLLM language model class. Therefore, in order to return a language model vLLM
        class, we use a wrapper to give `self` the same interface as a text model."""

        # Exclude self and object
        bases = self.__class__.mro()[1:-1]
        # Keep only classes defined in `vllm.model_executor.models.transformers`
        bases = [b for b in bases if ".transformers." in b.__module__]
        # Exclude MultiModalMixin itself
        bases = [b for b in bases if b is not MultiModalMixin]

        class LanguageModel(*bases):
            def __init__(self, multimodal_model):
                # Don't call super().__init__() to avoid re-initialization
                self.__dict__.update(multimodal_model.__dict__)

            model = getattr_iter(self.model, ("language_model", "text_model"), None)
```
**EN:** Defines `MultiModalMixin`, a supporting module used by the surrounding model implementation. It inherits from SupportsMultiModal, SupportsMRoPE. Key methods such as `__init__`, `_get_encoder_cls`, `_decorate_for_torch_compile`, `forward`, `get_language_model` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MultiModalMixin`，它是一个被周边模型实现复用的支撑模块。 它继承自 SupportsMultiModal、SupportsMRoPE。 `__init__`, `_get_encoder_cls`, `_decorate_for_torch_compile`, `forward`, `get_language_model` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Pooling and encoding**
  - **EN:** This file exposes encoder-style behavior, pooled outputs, or sequence-level representations beyond causal decoding.
  - **CN:** 该文件提供编码器式行为、池化输出，或超出因果解码的序列表征。
- **Position encoding**
  - **EN:** Rotary or related positional logic appears in the file to preserve token order information efficiently.
  - **CN:** 文件中出现了 Rotary 等位置编码逻辑，用于高效保留 token 顺序信息。

## Dependencies / 依赖关系
- **Standard library**: collections.abc, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.compilation.decorators, vllm.config.utils, vllm.inputs, vllm.logger, vllm.model_executor.models.interfaces, vllm.multimodal, vllm.multimodal.inputs, vllm.multimodal.parse
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
