# minicpmv.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/minicpmv.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Minicpmv multimodal model adapter used for inference in vLLM. The module docstring summarizes it as: "Inference-only MiniCPM-V model compatible with HuggingFace weights." / 实现 Minicpmv 在 vLLM 中用于推理的多模态模型适配器。 模块文档字符串还将其概括为：“Inference-only MiniCPM-V model compatible with HuggingFace weights。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-98)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

# Adapted from
# https://github.com/huggingface/transformers/blob/v4.28.0/src/transformers/models/llama/modeling_llama.py
# Copyright 2023 The vLLM team.
# Copyright 2022 EleutherAI and the HuggingFace Inc. team. All rights reserved.
#
# This code is based on EleutherAI's GPT-NeoX library and the GPT-NeoX
# and OPT implementations in this library. It has been modified from its
# original forms to accommodate minor architectural differences compared
# to GPT-NeoX and OPT used by the Meta AI team that trained the model.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
# ... omitted for brevity ...
from vllm.platforms import current_platform
from vllm.sequence import IntermediateTensors
from vllm.utils.collection_utils import flatten_2d_lists
from vllm.utils.tensor_schema import TensorSchema, TensorShape
from vllm.utils.torch_utils import set_default_torch_dtype

from .idefics2_vision_model import Idefics2VisionTransformer
from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
)
from .utils import AutoWeightsLoader, flatten_bn, maybe_prefix

# For profile run
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as numpy, torch, torch.types, torch.nn.init supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.quantization connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 numpy, torch, torch.types, torch.nn.init 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.quantization 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `_MAX_FRAMES_PER_VIDEO` (lines 99-99)
```python
_MAX_FRAMES_PER_VIDEO = 16
```
**EN:** This assignment block centers on `_MAX_FRAMES_PER_VIDEO` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `_MAX_FRAMES_PER_VIDEO` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Class `MiniCPMVImagePixelInputs` (lines 102-127)
```python
class MiniCPMVImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bns: Batch size * number of images * number of slices
        - bn: Batch size * number of images
        - c: Number of channels
        - h: Height
        - w: Width
    """

    type: Literal["pixel_values"] = "pixel_values"

    # Note that the patch size may vary, so we pass it as a list instead of a
    # batched tensor.
    pixel_values: Annotated[
        list[torch.Tensor],
        TensorShape("bns", "c", "h", "w", dynamic_dims={"h", "w"}),
    ]
    tgt_sizes: Annotated[
        torch.Tensor,
        TensorShape("bns", 2),  # This should be in `(height, width)` format.
    ]
    num_slices: Annotated[
        torch.Tensor,
        TensorShape("bn"),
    ]
```
**EN:** Defines `MiniCPMVImagePixelInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bns: Batch size * number of images * number of slices - bn: Batch size * number of images - c: Number of channels - h: Height - w: Width."
**CN:** 定义 `MiniCPMVImagePixelInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bns: Batch size * number of images * number of slices - bn: Batch size * number of images - c: Number of channels - h: Height - w: Width。”

### Class `MiniCPMVImageEmbeddingInputs` (lines 130-142)
```python
class MiniCPMVImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - ns: Number of slices
        - hs: Hidden size (must match language model backbone)
    """

    type: Literal["image_embeds"]
    image_embeds: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("bn", "ns", "hs", dynamic_dims={"ns"}),
    ]
```
**EN:** Defines `MiniCPMVImageEmbeddingInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - ns: Number of slices - hs: Hidden size (must match language model backbone)."
**CN:** 定义 `MiniCPMVImageEmbeddingInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - ns: Number of slices - hs: Hidden size (must match language model backbone)。”

### Class `Resampler2_5` (lines 150-242)
```python
class Resampler2_5(BaseResampler):
    def __init__(
        self,
        num_queries: int,
        embed_dim: int,
        num_heads: int,
        kv_dim: int | None = None,
        norm_layer: Callable[[int], nn.LayerNorm] = DEFAULT_LN,
        max_size: tuple[int, int] = (70, 70),
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__(
            num_queries,
            embed_dim,
            num_heads,
            kv_dim,
            norm_layer,
            quant_config=quant_config,
            prefix=prefix,
# ... omitted for brevity ...
    def _set_2d_pos_cache(
        self, max_size: tuple[int, int], device: torch.types.Device = "cpu"
    ) -> None:
        pos_embed_arr = get_2d_sincos_pos_embed(
            self.embed_dim, max_size, version=(2, 5)
        )
        pos_embed = torch.from_numpy(pos_embed_arr).float().to(device)
        self.register_buffer("pos_embed", pos_embed, persistent=False)
# ... omitted for brevity ...
    def _adjust_pos_cache(
        self, tgt_sizes: torch.Tensor, device: torch.types.Device
    ) -> None:
        max_h = tgt_sizes[:, 0].max().item()
        max_w = tgt_sizes[:, 1].max().item()
        assert isinstance(max_h, int) and isinstance(max_w, int)

        if max_h > self.max_size[0] or max_w > self.max_size[1]:
            self.max_size = (
                max(max_h, self.max_size[0]),
                max(max_w, self.max_size[1]),
            )
            self._set_2d_pos_cache(self.max_size, device)
# ... omitted for brevity ...
    def forward(self, x: torch.Tensor, tgt_sizes: torch.Tensor) -> torch.Tensor:
        assert x.shape[0] == tgt_sizes.shape[0]
        bs = x.shape[0]

        device = x.device
        dtype = x.dtype

        patch_len = tgt_sizes[:, 0] * tgt_sizes[:, 1]

        self._adjust_pos_cache(tgt_sizes, device=device)

        max_patch_len = patch_len.max().item()
        assert isinstance(max_patch_len, int)

        key_padding_mask = torch.zeros(
            (bs, max_patch_len), dtype=torch.bool, device=device
        )

        pos_embed = []
```
**EN:** Defines `Resampler2_5`, a supporting module used by the surrounding model implementation. It inherits from BaseResampler. Key methods such as `__init__`, `_set_2d_pos_cache`, `_adjust_pos_cache`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Resampler2_5`，它是一个被周边模型实现复用的支撑模块。 它继承自 BaseResampler。 `__init__`, `_set_2d_pos_cache`, `_adjust_pos_cache`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Resampler4_5` (lines 245-436)
```python
class Resampler4_5(Resampler2_5):
    def __init__(
        self,
        num_queries: int,
        embed_dim: int,
        num_heads: int,
        kv_dim: int | None = None,
        norm_layer: Callable[[int], nn.LayerNorm] = DEFAULT_LN,
        max_size: tuple[int, int] = (70, 70),
        max_temporal_size: int = 36000,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__(
            num_queries,
            embed_dim,
            num_heads,
            kv_dim,
            norm_layer,
            max_size,
# ... omitted for brevity ...
    def get_1d_sincos_pos_embed_from_temporal_size(
        self, embed_dim: int, pos: np.ndarray
    ):
        """
        embed_dim: output dimension for each position
        pos: a list of positions to be encoded: size (M,)
        out: (M, D)
        """
        assert embed_dim % 2 == 0
        omega = np.arange(embed_dim // 2, dtype=np.float32)
        omega /= embed_dim / 2.0
        omega = 1.0 / 10000**omega  # (D/2,)

        pos = pos.reshape(-1)  # (M,)
# ... omitted for brevity ...
    def _set_temporal_pos_cache(
        self, max_temporal_size: int, device: torch.types.Device = "cpu"
    ) -> None:
        temporal_size = np.arange(max_temporal_size, dtype=np.float32)
        pos_embed = (
            torch.from_numpy(
                self.get_1d_sincos_pos_embed_from_temporal_size(
                    self.embed_dim, temporal_size
                )
            )
            .float()
            .to(device)
        )
        self.register_buffer("temporal_pos_embed", pos_embed, persistent=False)
# ... omitted for brevity ...
    def forward(
        self,
        x: torch.Tensor,
        tgt_sizes: torch.Tensor,
        # temporal_ids for high refresh rate videos
        temporal_ids=None,
    ) -> torch.Tensor:
        assert x.shape[0] == tgt_sizes.shape[0]
        bs = x.shape[0]

        device = x.device
        dtype = x.dtype

        patch_len = tgt_sizes[:, 0] * tgt_sizes[:, 1]

        self._adjust_pos_cache(tgt_sizes, device=device)

        temporal_pos_emb = False
        temporal_ids_flatten = None
```
**EN:** Defines `Resampler4_5`, a supporting module used by the surrounding model implementation. It inherits from Resampler2_5. Key methods such as `__init__`, `get_1d_sincos_pos_embed_from_temporal_size`, `_set_temporal_pos_cache`, `_adjust_temporal_pos_cache`, `_init_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Resampler4_5`，它是一个被周边模型实现复用的支撑模块。 它继承自 Resampler2_5。 `__init__`, `get_1d_sincos_pos_embed_from_temporal_size`, `_set_temporal_pos_cache`, `_adjust_temporal_pos_cache`, `_init_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MiniCPMVImageEmbeddingItems` (lines 465-483)
```python
class MiniCPMVImageEmbeddingItems(DictEmbeddingItems):
    def __init__(
        self,
        data: Mapping[str, torch.Tensor],
        fields_factory: Callable[
            [Mapping[str, torch.Tensor]],
            Mapping[str, MultiModalFieldConfig],
        ],
    ) -> None:
        super().__init__(
            data,
            modality="image",
            required_fields={"image_embeds", "image_sizes"},
            fields_factory=fields_factory,
        )

    def get_image_size(self, index: int) -> ImageSize:
        image_size = self.get(index)["image_sizes"].tolist()
        return ImageSize(width=image_size[0], height=image_size[1])
```
**EN:** Defines `MiniCPMVImageEmbeddingItems`, a supporting module used by the surrounding model implementation. It inherits from DictEmbeddingItems. Key methods such as `__init__`, `get_image_size` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MiniCPMVImageEmbeddingItems`，它是一个被周边模型实现复用的支撑模块。 它继承自 DictEmbeddingItems。 `__init__`, `get_image_size` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MiniCPMVBaseModel` (lines 1082-1302)
```python
class MiniCPMVBaseModel(nn.Module, SupportsMultiModal, SupportsPP):
    """
    The abstract class of MiniCPMV can only be inherited, but cannot be
    instantiated.
    """

    supports_encoder_tp_data = True

    @classmethod
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_config
        multimodal_config = vllm_config.model_config.multimodal_config
        quant_config = vllm_config.quant_config
        self.use_data_parallel = multimodal_config.mm_encoder_tp_mode == "data"
        super().__init__()
        # All MiniCPM-V models disable `tie_word_embeddings` but
        # `PretrainedConfig.tie_word_embeddings` defaults to True; we cannot
        # check `tie_word_embeddings` until vLLM integrate MiniCPM-V model
        # and config class
        self.config = config
        self.multimodal_config = multimodal_config

        self.version = get_version_by_config(self.config)

        with self._mark_language_model(vllm_config):
            self.llm = self.init_llm(
                vllm_config=vllm_config, prefix=maybe_prefix(prefix, "llm")
            )
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: Any,
    ) -> torch.Tensor:
        if intermediate_tensors is not None:
            inputs_embeds = None

        hidden_states = self.llm.model(
            input_ids=input_ids,
            positions=positions,
            intermediate_tensors=intermediate_tensors,
            inputs_embeds=inputs_embeds,
        )
        return hidden_states
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        return self.llm.compute_logits(hidden_states)
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(self)
        loaded = loader.load_weights(weights)
        self._ensure_resampler_device()
        return loaded
```
**EN:** Defines `MiniCPMVBaseModel`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods such as `get_placeholder_str`, `__init__`, `_ensure_resampler_device`, `_parse_and_validate_vision_input`, `_parse_and_validate_multimodal_inputs` show where construction, forward execution, or weight adaptation happens. Docstring hint: "The abstract class of MiniCPMV can only be inherited, but cannot be instantiated."
**CN:** 定义 `MiniCPMVBaseModel`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 `get_placeholder_str`, `__init__`, `_ensure_resampler_device`, `_parse_and_validate_vision_input`, `_parse_and_validate_multimodal_inputs` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“The abstract class of MiniCPMV can only be inherited, but cannot be instantiated。”

### Class `MiniCPMV` (lines 1793-1830)
```python
@MULTIMODAL_REGISTRY.register_processor(
    MiniCPMVMultiModalProcessor,
    info=MiniCPMVProcessingInfo,
    dummy_inputs=MiniCPMVDummyInputsBuilder,
)
class MiniCPMV(MiniCPMVBaseModel, SupportsMultiModal, SupportsLoRA):
    """
    Different versions of MiniCPMV use different visual encoders and LLMs,
    which is not conducive to the current integration logic of LoRA and
    bitsandbytes in vLLM. Therefore, it is necessary to separate them.
    """

    def __new__(cls, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_config
        if not hasattr(config, "version"):
            if config.hidden_size == 2304 and config.query_num == 64:
                version = (2, 0)
            else:
                version = (2, 5)
        else:
            version = str(config.version).split(".")
            version = tuple([int(x) for x in version])
        # Dispatch class based on version
        instance_cls = _SUPPORT_VERSION.get(version)
        if instance_cls is None:
            supported_versions = ", ".join(
                [f"{v[0]}.{v[1]}" for v in sorted(_SUPPORT_VERSION.keys())]
            )
            raise ValueError(
                f"Currently, MiniCPMV only supports versions "
                f"{supported_versions}. Got version: {version}"
            )

        # quant_config references base class members,
        # so update values before init is called
        cls.packed_modules_mapping.update(instance_cls.packed_modules_mapping)
        cls.embedding_modules.update(instance_cls.embedding_modules)
        return instance_cls(vllm_config=vllm_config, prefix=prefix)
```
**EN:** Defines `MiniCPMV`, a supporting module used by the surrounding model implementation. It inherits from MiniCPMVBaseModel, SupportsMultiModal, SupportsLoRA. Key methods such as `__new__` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Different versions of MiniCPMV use different visual encoders and LLMs, which is not conducive to the current integration logic of LoRA and bitsandbytes in vLLM."
**CN:** 定义 `MiniCPMV`，它是一个被周边模型实现复用的支撑模块。 它继承自 MiniCPMVBaseModel、SupportsMultiModal、SupportsLoRA。 `__new__` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Different versions of MiniCPMV use different visual encoders and LLMs, which is not conducive to the current integration logic of LoRA and bitsandbytes in vLLM。”

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
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
- **Standard library**: math, collections, collections.abc, functools, itertools, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: numpy, torch, torch.types, torch.nn.init, transformers, typing_extensions
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.inputs, vllm.model_executor.layers.quantization, vllm.model_executor.layers.resampler, vllm.model_executor.models.llama, vllm.model_executor.models.minicpm, vllm.model_executor.models.module_mapping
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .idefics2_vision_model, .interfaces, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
