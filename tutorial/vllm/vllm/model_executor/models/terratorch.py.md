# terratorch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/terratorch.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Terratorch multimodal model adapter used for inference in vLLM. The module docstring summarizes it as: "Wrapper around `Terratorch` models." / 实现 Terratorch 在 vLLM 中用于推理的多模态模型适配器。 模块文档字符串还将其概括为：“Wrapper around `Terratorch` models。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-67)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

# Copyright 2025 The vLLM team.
# Copyright 2025 IBM.
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
"""Wrapper around `Terratorch` models"""
# ... omitted for brevity ...
    MultiModalDataItems,
    MultiModalDataParser,
)
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    ProcessorInputs,
    PromptUpdate,
    TimingContext,
)
from vllm.sequence import IntermediateTensors

from .interfaces import IsAttentionFree, MultiModalEmbeddings, SupportsMultiModal
from .interfaces_base import attn_type
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, terratorch.vllm, transformers supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.logger connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, terratorch.vllm, transformers 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.logger 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger` (lines 68-68)
```python
logger = init_logger(__name__)
```
**EN:** This assignment initializes the module-level logger used for diagnostics, backend selection messages, and compatibility warnings.
**CN:** 这个赋值语句初始化了模块级日志记录器，用于输出诊断信息、后端选择消息和兼容性警告。

### Function `_terratorch_field_names` (lines 71-72)
```python
def _terratorch_field_names(input_definition: InputDefinition):
    return set(input_definition.data.keys())
```
**EN:** The function `_terratorch_field_names` helps provide a reusable helper for the surrounding model code. Its main inputs are `input_definition`.
**CN:** 函数 `_terratorch_field_names` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `input_definition`。

### Function `_terratorch_field_factory` (lines 75-95)
```python
def _terratorch_field_factory(
    input_definition: InputDefinition,
    *,
    is_shared: bool = True,  # True for unprocessed data, False for processed data
):
    def _terratorch_field_config(
        hf_inputs: Mapping[str, torch.Tensor],
    ) -> Mapping[str, MultiModalFieldConfig]:
        fields = dict[str, MultiModalFieldConfig]()
        for name, input in input_definition.data.items():
            modality = "image"
            if input.type == InputTypeEnum.tensor:
                fields[name] = (
                    MultiModalFieldConfig.shared(modality, batch_size=1)
                    if is_shared
                    else MultiModalFieldConfig.batched(modality)
                )

        return fields

    return _terratorch_field_config
```
**EN:** The function `_terratorch_field_factory` helps provide a reusable helper for the surrounding model code. Its main inputs are `input_definition`.
**CN:** 函数 `_terratorch_field_factory` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `input_definition`。

### Class `TerratorchMultiModalDataParser` (lines 98-122)
```python
class TerratorchMultiModalDataParser(MultiModalDataParser):
    def __init__(self, input_definition: InputDefinition, *args, **kwargs):
        super().__init__(*args, **kwargs)

        self.input_definition = input_definition

    def _parse_image_data(
        self,
        data: dict[str, torch.Tensor] | ModalityData[ImageItem],
    ) -> ModalityDataItems[Any, Any] | None:
        if isinstance(data, dict):
            return DictEmbeddingItems(
                data,
                modality="image",
                required_fields=_terratorch_field_names(self.input_definition),
                fields_factory=_terratorch_field_factory(self.input_definition),
            )

        return super()._parse_image_data(data)

    def parse_mm_data(self, mm_data: MultiModalDataDict) -> MultiModalDataItems:
        if "image" not in mm_data:
            mm_data = {"image": mm_data}

        return super().parse_mm_data(mm_data)
```
**EN:** Defines `TerratorchMultiModalDataParser`, a supporting module used by the surrounding model implementation. It inherits from MultiModalDataParser. Key methods such as `__init__`, `_parse_image_data`, `parse_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `TerratorchMultiModalDataParser`，它是一个被周边模型实现复用的支撑模块。 它继承自 MultiModalDataParser。 `__init__`, `_parse_image_data`, `parse_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `TerratorchProcessingInfo` (lines 125-138)
```python
class TerratorchProcessingInfo(BaseProcessingInfo):
    @cached_property
    def input_definition(self) -> InputDefinition:
        pretrained_cfg = self.get_hf_config().to_dict()["pretrained_cfg"]
        return InputDefinition(**pretrained_cfg["input"])

    def get_data_parser(self):
        return TerratorchMultiModalDataParser(
            self.input_definition,
            expected_hidden_size=self._get_expected_hidden_size(),
        )

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": None}
```
**EN:** Defines `TerratorchProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseProcessingInfo. Key methods such as `input_definition`, `get_data_parser`, `get_supported_mm_limits` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `TerratorchProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseProcessingInfo。 `input_definition`, `get_data_parser`, `get_supported_mm_limits` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `TerratorchInputBuilder` (lines 141-167)
```python
class TerratorchInputBuilder(BaseDummyInputsBuilder[TerratorchProcessingInfo]):
    def __init__(self, info: TerratorchProcessingInfo):
        super().__init__(info)
        self.dummy_data_generator = DummyDataGenerator(
            self.info.get_hf_config().to_dict()["pretrained_cfg"]
        )

    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        return ""

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        # Dummy data is generated based on the 'input' section
        # defined in the HF configuration file

        if mm_options:
            logger.warning(
                "Configurable multimodal profiling "
                "options are not supported for Terratorch. "
                "They are ignored for now."
            )

        return self.dummy_data_generator.get_dummy_mm_data()
```
**EN:** Defines `TerratorchInputBuilder`, a supporting module used by the surrounding model implementation. It inherits from BaseDummyInputsBuilder[TerratorchProcessingInfo]. Key methods such as `__init__`, `get_dummy_text`, `get_dummy_mm_data` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `TerratorchInputBuilder`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseDummyInputsBuilder[TerratorchProcessingInfo]。 `__init__`, `get_dummy_text`, `get_dummy_mm_data` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `TerratorchMultiModalProcessor` (lines 170-229)
```python
class TerratorchMultiModalProcessor(BaseMultiModalProcessor[TerratorchProcessingInfo]):
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
        *,
        is_shared: bool = True,
    ) -> Mapping[str, MultiModalFieldConfig]:
        factory = _terratorch_field_factory(
            self.info.input_definition,
            is_shared=is_shared,
        )
        return factory(hf_inputs)

    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        return []

    def apply(
        self,
        inputs: ProcessorInputs,
        timing_ctx: TimingContext,
    ) -> MultiModalInput:
        mm_items = inputs.mm_data_items
        hf_processor_mm_kwargs = inputs.hf_processor_mm_kwargs

        with timing_ctx.record("apply_hf_processor"):
            _, passthrough_data = self._get_hf_mm_data(mm_items)
            mm_processed_data = BatchFeature(
                {
                    k: torch.as_tensor(v).unsqueeze(0)
                    for k, v in passthrough_data.items()
                },
                tensor_type="pt",
            )

        mm_kwargs = MultiModalKwargsItems.from_hf_inputs(
            mm_processed_data,
            self._get_mm_fields_config(
                mm_processed_data,
                hf_processor_mm_kwargs,
                is_shared=False,
            ),
        )

        with timing_ctx.record("get_mm_hashes"):
            mm_hashes = inputs.get_mm_hashes(self.info.model_id)

        mm_placeholders = {"image": [PlaceholderRange(offset=0, length=0)]}

        return mm_input(
            prompt_token_ids=[1],
            mm_kwargs=mm_kwargs,
            mm_hashes=mm_hashes,
            mm_placeholders=mm_placeholders,
        )
```
**EN:** Defines `TerratorchMultiModalProcessor`, a supporting module used by the surrounding model implementation. It inherits from BaseMultiModalProcessor[TerratorchProcessingInfo]. Key methods such as `_get_mm_fields_config`, `_get_prompt_updates`, `apply` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `TerratorchMultiModalProcessor`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseMultiModalProcessor[TerratorchProcessingInfo]。 `_get_mm_fields_config`, `_get_prompt_updates`, `apply` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Terratorch` (lines 232-322)
```python
@attn_type("attention_free")
@MULTIMODAL_REGISTRY.register_processor(
    TerratorchMultiModalProcessor,
    info=TerratorchProcessingInfo,
    dummy_inputs=TerratorchInputBuilder,
)
class Terratorch(nn.Module, IsAttentionFree, SupportsMultiModal):
    supports_multimodal_raw_input_only = True
    is_pooling_model = True
# ... omitted for brevity ...
    def __init__(self, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config.to_dict()["pretrained_cfg"]

        self.inference_runner = InferenceRunner(config)
        self.model = self.inference_runner.model

        self.pooler = IdentityPooler()
# ... omitted for brevity ...
    def embed_input_ids(
        self,
        input_ids: torch.Tensor,
        multimodal_embeddings: MultiModalEmbeddings | None = None,
        *,
        is_multimodal: torch.Tensor | None = None,
    ) -> torch.Tensor:
        # We do not really use any input tokens and therefore no embeddings
        # to be calculated. However, due to the mandatory token ids in
        # the input prompt we pass one token and the size of the dummy
        # embedding tensors must reflect that.
        return torch.empty((input_ids.shape[0], 0))
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ):
        model_output = self.inference_runner.forward(**kwargs)
        return model_output.output
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        params_list = []
        model_buffers = dict(self.named_buffers())
        loaded_buffers = []
        for key, value in weights:
            if isinstance(value, (dict, OrderedDict)):
                if key == "state_dict":
                    weights_to_parse = value
                    for name, weight in weights_to_parse.items():
                        name = f"inference_runner.{name}"

                        if "pos_embed" in name:
                            continue

                        if "_timm_module." in name:
                            name = name.replace("_timm_module.", "")

                        # this model requires a couple of buffers to be loaded
                        # that are not loadable with the AutoWeightsLoader
```
**EN:** Defines `Terratorch`, a supporting module used by the surrounding model implementation. It inherits from nn.Module, IsAttentionFree, SupportsMultiModal. Key methods such as `get_placeholder_str`, `__init__`, `embed_input_ids`, `forward`, `load_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Terratorch`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module、IsAttentionFree、SupportsMultiModal。 `get_placeholder_str`, `__init__`, `embed_input_ids`, `forward`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。
- **Pooling and encoding**
  - **EN:** This file exposes encoder-style behavior, pooled outputs, or sequence-level representations beyond causal decoding.
  - **CN:** 该文件提供编码器式行为、池化输出，或超出因果解码的序列表征。
- **Position encoding**
  - **EN:** Rotary or related positional logic appears in the file to preserve token order information efficiently.
  - **CN:** 文件中出现了 Rotary 等位置编码逻辑，用于高效保留 token 顺序信息。

## Dependencies / 依赖关系
- **Standard library**: collections, collections.abc, functools, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, terratorch.vllm, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.logger, vllm.model_executor.layers.pooler, vllm.model_executor.model_loader.weight_utils, vllm.model_executor.models.utils, vllm.multimodal
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .interfaces_base
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
