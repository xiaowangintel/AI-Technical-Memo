# nemotron_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/nemotron_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Nemotron VL multimodal model adapter used for inference in vLLM. / 实现 Nemotron VL 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-51)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import math
from collections.abc import Iterable

import torch
import torch.nn as nn
from transformers import AutoModel, PretrainedConfig

from vllm.config import VllmConfig
from vllm.model_executor.layers.linear import ReplicatedLinear
from vllm.model_executor.layers.pooler import DispatchPooler
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.quantization.awq import AWQConfig
from vllm.model_executor.models.internvl import (
    BaseInternVLDummyInputsBuilder,
    BaseInternVLMultiModalProcessor,
    BaseInternVLProcessingInfo,
# ... omitted for brevity ...
from .interfaces import (
    MultiModalEmbeddings,
    SupportsCrossEncoding,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
)
from .interfaces_base import VllmModelForPooling
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    init_vllm_registered_model,
    maybe_prefix,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, transformers supply framework primitives, while internal modules like vllm.config, vllm.model_executor.layers.linear, vllm.model_executor.layers.pooler, vllm.model_executor.layers.quantization connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, transformers 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.model_executor.layers.linear, vllm.model_executor.layers.pooler, vllm.model_executor.layers.quantization 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Class `NemotronVLProcessingInfo` (lines 52-83)
```python
class NemotronVLProcessingInfo(BaseInternVLProcessingInfo):
    """Processing info for Nemotron VL models."""

    def get_image_processor(self, **kwargs: object):
        kwargs = self.ctx.get_merged_mm_kwargs(kwargs)
        orig_processor = cached_image_processor_from_config(
            self.ctx.model_config, **kwargs
        )

        return LlamaNemotronNanoVLImageProcessor(
            image_size=orig_processor.image_size,
            min_dynamic_patch=1,
            max_dynamic_patch=orig_processor.max_num_tiles,
            dynamic_image_size=True,
            use_thumbnail=orig_processor.use_thumbnail,
        )

    def get_hf_processor(self, **kwargs: object) -> LlamaNemotronNanoVLProcessor:
        config = self.get_hf_config()
        vision_config = config.vision_config

        image_processor = self.get_image_processor(**kwargs)
        image_size = image_processor.image_size
        patch_size = vision_config.patch_size
        downsample_ratio = config.downsample_ratio
        image_seq_length = int((image_size // patch_size) ** 2 * (downsample_ratio**2))

        return LlamaNemotronNanoVLProcessor(
            tokenizer=self.get_tokenizer(),
            image_processor=image_processor,
            image_seq_length=image_seq_length,
        )
```
**EN:** Defines `NemotronVLProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseInternVLProcessingInfo. Key methods such as `get_image_processor`, `get_hf_processor` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Processing info for Nemotron VL models."
**CN:** 定义 `NemotronVLProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseInternVLProcessingInfo。 `get_image_processor`, `get_hf_processor` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Processing info for Nemotron VL models。”

### Class `LlamaNemotronVLChatModel` (lines 86-394)
```python
@MULTIMODAL_REGISTRY.register_processor(
    BaseInternVLMultiModalProcessor[NemotronVLProcessingInfo],
    info=NemotronVLProcessingInfo,
    dummy_inputs=BaseInternVLDummyInputsBuilder[NemotronVLProcessingInfo],
)
class LlamaNemotronVLChatModel(nn.Module, SupportsMultiModal, SupportsPP, SupportsLoRA):
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config

        self.config = config
        self.model_config = vllm_config.model_config
        self.multimodal_config = multimodal_config
        self._patch_quant_config(config, quant_config)

        image_size = config.force_image_size or config.vision_config.image_size
        patch_size = config.vision_config.patch_size
        self.patch_size = patch_size
        self.num_image_token = int(
            (image_size // patch_size) ** 2 * (config.downsample_ratio**2)
        )
        self.downsample_ratio = config.downsample_ratio
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> IntermediateTensors:
        if intermediate_tensors is not None:
            inputs_embeds = None

        forward_kwargs = {
            "input_ids": input_ids,
            "positions": positions,
            "intermediate_tensors": intermediate_tensors,
            "inputs_embeds": inputs_embeds,
        }

        # Only required if the model is mono-architecture
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        return self.language_model.compute_logits(hidden_states)
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        ## Ignore registered_buffers
        ## see https://huggingface.co/nvidia/C-RADIOv2-H/blob/main/input_conditioner.py#L28 # noqa: E501
        skip_substrs = ["norm_mean", "norm_std"]
        loader = AutoWeightsLoader(self, skip_substrs=skip_substrs)
        return loader.load_weights(weights)
```
**EN:** Defines `LlamaNemotronVLChatModel`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods such as `get_placeholder_str`, `__init__`, `_patch_quant_config`, `_init_vision_model`, `_init_mlp1` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `LlamaNemotronVLChatModel`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 `get_placeholder_str`, `__init__`, `_patch_quant_config`, `_init_vision_model`, `_init_mlp1` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `LlamaNemotronVLEmbedProcessingInfo` (lines 406-458)
```python
class LlamaNemotronVLEmbedProcessingInfo(BaseInternVLProcessingInfo):
    """Processing info for LlamaNemotronVL embedding model."""

    def get_image_processor(self, **kwargs):
        model_config = self.ctx.model_config

        config = self.get_hf_config()
        processor_config = (
            get_hf_file_to_dict(
                "processor_config.json",
                model_config.model,
                model_config.revision,
            )
            or {}
        )

        min_dynamic_patch = processor_config.get(
            "min_input_tiles",
            getattr(config, "min_dynamic_patch", 1),
        )
        max_dynamic_patch = processor_config.get(
            "max_input_tiles",
            getattr(config, "max_dynamic_patch", 1),
        )
        dynamic_image_size = processor_config.get(
            "dynamic_image_size",
            getattr(config, "dynamic_image_size", True),
        )

        kwargs = self.ctx.get_merged_mm_kwargs(kwargs)
        kwargs.setdefault("image_size", config.force_image_size)
        kwargs.setdefault("min_dynamic_patch", min_dynamic_patch)
        kwargs.setdefault("max_dynamic_patch", max_dynamic_patch)
        kwargs.setdefault("dynamic_image_size", dynamic_image_size)
        kwargs.setdefault("use_thumbnail", True)

        return LlamaNemotronVLEmbedImageProcessor(**kwargs)

    def get_hf_processor(self, **kwargs: object) -> LlamaNemotronVLEmbedProcessor:
        config = self.get_hf_config()
        vision_config = config.vision_config

        image_processor = self.get_image_processor(**kwargs)
        image_size = image_processor.image_size
        patch_size = vision_config.patch_size
        downsample_ratio = config.downsample_ratio
        image_seq_length = int((image_size // patch_size) ** 2 * (downsample_ratio**2))

        return LlamaNemotronVLEmbedProcessor(
            tokenizer=self.get_tokenizer(),
            image_processor=image_processor,
            image_seq_length=image_seq_length,
        )
```
**EN:** Defines `LlamaNemotronVLEmbedProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from BaseInternVLProcessingInfo. Key methods such as `get_image_processor`, `get_hf_processor` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Processing info for LlamaNemotronVL embedding model."
**CN:** 定义 `LlamaNemotronVLEmbedProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseInternVLProcessingInfo。 `get_image_processor`, `get_hf_processor` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Processing info for LlamaNemotronVL embedding model。”

### Class `LlamaNemotronVLForEmbedding` (lines 461-536)
```python
@MULTIMODAL_REGISTRY.register_processor(
    BaseInternVLMultiModalProcessor[LlamaNemotronVLEmbedProcessingInfo],
    info=LlamaNemotronVLEmbedProcessingInfo,
    dummy_inputs=BaseInternVLDummyInputsBuilder[LlamaNemotronVLEmbedProcessingInfo],
)
class LlamaNemotronVLForEmbedding(LlamaNemotronVLChatModel, VllmModelForPooling):
    """
    LlamaNemotronVL model for embeddings.

# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__(vllm_config=vllm_config, prefix=prefix)

        config = vllm_config.model_config.hf_config

        # Override: get img_context_token_id from config (parent sets None)
        self.img_context_token_id = getattr(config, "img_context_token_id", None)

        # Initialize pooler for embedding output
        pooler_config = vllm_config.model_config.pooler_config
        assert pooler_config is not None
        self.pooler = DispatchPooler.for_embedding(pooler_config)
# ... omitted for brevity ...
    def _init_vision_model(
        self,
        config: PretrainedConfig,
        quant_config,
        *,
        prefix: str,
    ) -> nn.Module:
        """Override to use SigLIP instead of C-RADIO."""
        return SiglipVisionModel(
            config.vision_config,
            quant_config=quant_config,
            prefix=prefix,
            use_head=False,
        )
# ... omitted for brevity ...
    def _init_mlp1(self, config: PretrainedConfig) -> nn.Module:
        """Override to use different MLP structure for embedding model."""
        return super()._init_mlp1(
            config,
            vit_hidden_size=config.vision_config.hidden_size,
            vision_projection_hidden_size=config.get_text_config().hidden_size,
        )
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        """Override to use different weight mapping for SigLIP."""
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights, mapper=self.weight_mapper)
```
**EN:** Defines `LlamaNemotronVLForEmbedding`, a supporting module used by the surrounding model implementation. It inherits from LlamaNemotronVLChatModel, VllmModelForPooling. Key methods such as `__init__`, `_init_vision_model`, `_init_mlp1`, `_call_vision_model`, `load_weights` show where construction, forward execution, or weight adaptation happens. Docstring hint: "LlamaNemotronVL model for embeddings."
**CN:** 定义 `LlamaNemotronVLForEmbedding`，它是一个被周边模型实现复用的支撑模块。 它继承自 LlamaNemotronVLChatModel、VllmModelForPooling。 `__init__`, `_init_vision_model`, `_init_mlp1`, `_call_vision_model`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“LlamaNemotronVL model for embeddings。”

### Class `LlamaNemotronVLForSequenceClassification` (lines 539-590)
```python
class LlamaNemotronVLForSequenceClassification(
    LlamaNemotronVLForEmbedding, SupportsCrossEncoding
):
    """LlamaNemotronVL model variant for sequence classification / reranking."""

    # Reranker checkpoint places base model weights under `model.*`,
    # while `score.*` remains at the top level.
    weight_mapper = WeightsMapper(orig_to_new_prefix={"model.": ""}) | (
        LlamaNemotronVLForEmbedding.weight_mapper
    )

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__(vllm_config=vllm_config, prefix=prefix)

        text_config = vllm_config.model_config.hf_config.get_text_config()
        model_config = vllm_config.model_config
        quant_config = vllm_config.quant_config

        self.score = ReplicatedLinear(
            model_config.get_hidden_size(),
            text_config.num_labels,
            bias=False,
            params_dtype=model_config.head_dtype,
            quant_config=quant_config,
            return_bias=False,
            prefix=maybe_prefix(prefix, "score"),
        )

        pooler_config = model_config.pooler_config
        assert pooler_config is not None
        self.pooler = DispatchPooler.for_seq_cls(pooler_config, classifier=self.score)

    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loaded_weights = super().load_weights(weights)

        # reranker checkpoint omits the inner LM seq-cls head
        # (`language_model.score.*`). It is unused by this outer model, but
        # the default loader expects all parameters to be initialized.
        for name, param in self.named_parameters():
            if not name.startswith("language_model.score.") or name in loaded_weights:
                continue

            if name.endswith(".weight"):
                torch.nn.init.kaiming_uniform_(param, a=math.sqrt(5))
            elif name.endswith(".bias"):
                torch.nn.init.zeros_(param)
            else:
                torch.nn.init.normal_(param, mean=0.0, std=0.02)

            loaded_weights.add(name)

        return loaded_weights
```
**EN:** Defines `LlamaNemotronVLForSequenceClassification`, a supporting module used by the surrounding model implementation. It inherits from LlamaNemotronVLForEmbedding, SupportsCrossEncoding. Key methods such as `__init__`, `load_weights` show where construction, forward execution, or weight adaptation happens. Docstring hint: "LlamaNemotronVL model variant for sequence classification / reranking."
**CN:** 定义 `LlamaNemotronVLForSequenceClassification`，它是一个被周边模型实现复用的支撑模块。 它继承自 LlamaNemotronVLForEmbedding、SupportsCrossEncoding。 `__init__`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“LlamaNemotronVL model variant for sequence classification / reranking。”

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
- **Pooling and encoding**
  - **EN:** This file exposes encoder-style behavior, pooled outputs, or sequence-level representations beyond causal decoding.
  - **CN:** 该文件提供编码器式行为、池化输出，或超出因果解码的序列表征。

## Dependencies / 依赖关系
- **Standard library**: math, collections.abc
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.model_executor.layers.linear, vllm.model_executor.layers.pooler, vllm.model_executor.layers.quantization, vllm.model_executor.layers.quantization.awq, vllm.model_executor.models.internvl, vllm.model_executor.models.module_mapping, vllm.model_executor.models.siglip
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .interfaces_base, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
