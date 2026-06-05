# siglip.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/siglip.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the SigLIP multimodal model adapter used for inference in vLLM. / 实现 SigLIP 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-75)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

from collections.abc import Callable, Iterable, Mapping
from functools import cached_property, partial
from typing import Annotated, Literal

import torch
from torch import nn
from transformers import (
    BatchFeature,
    SiglipConfig,
    SiglipProcessor,
    SiglipTextConfig,
    SiglipVisionConfig,
)

from vllm.config import VllmConfig
# ... omitted for brevity ...
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import MultiModalEmbeddings, SupportsMultiModal, SupportsQuant
from .interfaces_base import default_pooling_type
from .utils import AutoWeightsLoader, maybe_prefix
from .vision import (
    VisionEncoderInfo,
    VisionFeatureSelectStrategy,
    VisionFeatureSelectStrategyStr,
    get_num_selected_vision_tokens,
    is_vit_use_data_parallel,
    resolve_visual_encoder_outputs,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, transformers supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.distributed, vllm.inputs connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, transformers 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.distributed, vllm.inputs 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Class `SiglipImagePixelInputs` (lines 76-86)
```python
class SiglipImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size * number of images
        - c: Number of channels (3)
        - h: Height of each image
        - w: Width of each image
    """

    type: Literal["pixel_values"]
    data: Annotated[torch.Tensor, TensorShape("bn", 3, "h", "w")]
```
**EN:** Defines `SiglipImagePixelInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - bn: Batch size * number of images - c: Number of channels (3) - h: Height of each image - w: Width of each image."
**CN:** 定义 `SiglipImagePixelInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - bn: Batch size * number of images - c: Number of channels (3) - h: Height of each image - w: Width of each image。”

### Class `SiglipEncoderInfo` (lines 264-281)
```python
class SiglipEncoderInfo(VisionEncoderInfo[SiglipVisionConfig]):
    def get_num_image_tokens(
        self,
        *,
        image_width: int,
        image_height: int,
    ) -> int:
        return self.get_patch_grid_length() ** 2

    def get_image_size(self) -> int:
        return self.vision_config.image_size

    def get_patch_size(self) -> int:
        return self.vision_config.patch_size

    def get_patch_grid_length(self) -> int:
        image_size, patch_size = self.get_image_size(), self.get_patch_size()
        return image_size // patch_size
```
**EN:** Defines `SiglipEncoderInfo`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from VisionEncoderInfo[SiglipVisionConfig]. Key methods such as `get_num_image_tokens`, `get_image_size`, `get_patch_size`, `get_patch_grid_length` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SiglipEncoderInfo`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 VisionEncoderInfo[SiglipVisionConfig]。 `get_num_image_tokens`, `get_image_size`, `get_patch_size`, `get_patch_grid_length` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SiglipVisionEmbeddings` (lines 285-355)
```python
class SiglipVisionEmbeddings(nn.Module):
    def __init__(self, config: SiglipVisionConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.image_size = config.image_size
        self.patch_size = config.patch_size

        self.patch_embedding = Conv2dLayer(
            in_channels=config.num_channels,
            out_channels=self.embed_dim,
            kernel_size=self.patch_size,
            stride=self.patch_size,
            padding="valid",
        )

        self.num_patches = (self.image_size // self.patch_size) ** 2
        self.num_positions = self.num_patches
        self.position_embedding = nn.Embedding(self.num_positions, self.embed_dim)
        self.register_buffer(
# ... omitted for brevity ...
    def interpolate_pos_encoding(
        self, embeddings: torch.Tensor, height: int, width: int
    ) -> torch.Tensor:
        num_patches = embeddings.shape[1]
        num_positions = self.position_embedding.weight.shape[1]
        if num_patches == num_positions and height == width:
            return self.position_embedding(self.position_ids)

        patch_pos_embed = self.position_embedding.weight.unsqueeze(0)

        dim = embeddings.shape[-1]

        new_height = height // self.patch_size
        new_width = width // self.patch_size

        sqrt_num_positions = int(num_positions**0.5)
        patch_pos_embed = patch_pos_embed.reshape(
            1, sqrt_num_positions, sqrt_num_positions, dim
        )
# ... omitted for brevity ...
    def forward(
        self, pixel_values: torch.Tensor, interpolate_pos_encoding: bool = False
    ) -> torch.Tensor:
        _, _, height, width = pixel_values.shape
        target_dtype = self.patch_embedding.weight.dtype
        patch_embeds = self.patch_embedding(
            pixel_values.to(dtype=target_dtype)
        )  # shape = [*, width, grid, grid]
        embeddings = patch_embeds.flatten(2).transpose(1, 2)

        if interpolate_pos_encoding:
            embeddings += self.interpolate_pos_encoding(embeddings, height, width)
        else:
            embeddings += self.position_embedding(self.position_ids)
        return embeddings
```
**EN:** Defines `SiglipVisionEmbeddings`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `interpolate_pos_encoding`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SiglipVisionEmbeddings`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `interpolate_pos_encoding`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SiglipAttention` (lines 358-430)
```python
class SiglipAttention(nn.Module):
    def __init__(
        self,
        config: SiglipVisionConfig | SiglipTextConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        prefix: str = "",
        attn_cls: type[EncoderOnlyAttention] | type[MMEncoderAttention],
    ) -> None:
        super().__init__()

        self.config = config
        self.embed_dim = config.hidden_size
        self.num_heads = config.num_attention_heads
        self.head_dim = self.embed_dim // self.num_heads
        if self.head_dim * self.num_heads != self.embed_dim:
            raise ValueError(
                f"embed_dim must be divisible by num_heads "
                f"(got `embed_dim`: {self.embed_dim} and "
                f"`num_heads`: {self.num_heads})."
# ... omitted for brevity ...
    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> tuple[torch.Tensor, None]:
        """Input shape: Batch x Time x Channel"""
        qkv_states, _ = self.qkv_proj(hidden_states)
        query_states, key_states, value_states = qkv_states.chunk(3, dim=-1)
        out = self.attn(query_states, key_states, value_states)
        attn_output, _ = self.out_proj(out)

        return attn_output, None
```
**EN:** Defines `SiglipAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SiglipAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SiglipEncoderLayer` (lines 478-520)
```python
class SiglipEncoderLayer(nn.Module):
    def __init__(
        self,
        config: SiglipVisionConfig | SiglipTextConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        prefix: str = "",
        attn_cls: type[EncoderOnlyAttention] | type[MMEncoderAttention],
    ) -> None:
        super().__init__()

        self.embed_dim = config.hidden_size

        self.self_attn = SiglipAttention(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
            attn_cls=attn_cls,
        )
        self.layer_norm1 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)
        self.mlp = SiglipMLP(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )
        self.layer_norm2 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)

    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> tuple[torch.Tensor, None]:
        residual = hidden_states

        hidden_states = self.layer_norm1(hidden_states)
        hidden_states, _ = self.self_attn(hidden_states=hidden_states)
        hidden_states += residual

        residual = hidden_states
        hidden_states = self.layer_norm2(hidden_states)
        hidden_states = self.mlp(hidden_states)
        hidden_states += residual

        return hidden_states, None
```
**EN:** Defines `SiglipEncoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SiglipEncoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SiglipEncoder` (lines 523-570)
```python
class SiglipEncoder(nn.Module):
    def __init__(
        self,
        config: SiglipVisionConfig | SiglipTextConfig,
        quant_config: QuantizationConfig | None = None,
        num_hidden_layers_override: int | None = None,
        *,
        prefix: str = "",
        attn_cls: type[EncoderOnlyAttention] | type[MMEncoderAttention],
    ) -> None:
        super().__init__()

        self.config = config

        if num_hidden_layers_override is None:
            num_hidden_layers = config.num_hidden_layers
        else:
            num_hidden_layers = num_hidden_layers_override

        self.layers = nn.ModuleList(
            [
                SiglipEncoderLayer(
                    config,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layers.{layer_idx}",
                    attn_cls=attn_cls,
                )
                for layer_idx in range(num_hidden_layers)
            ]
        )

    def forward(
        self,
        inputs_embeds: torch.Tensor,
        return_all_hidden_states: bool,
    ) -> torch.Tensor | list[torch.Tensor]:
        hidden_states_pool = [inputs_embeds]
        hidden_states = inputs_embeds

        for encoder_layer in self.layers:
            hidden_states, _ = encoder_layer(hidden_states)
            if return_all_hidden_states:
                hidden_states_pool.append(hidden_states)
        # If we have multiple feature sample layers, we return all hidden
        # states in order and grab the ones we need by index.
        if return_all_hidden_states:
            return hidden_states_pool
        return hidden_states
```
**EN:** Defines `SiglipEncoder`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SiglipEncoder`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SiglipVisionModel` (lines 851-961)
```python
class SiglipVisionModel(nn.Module):
    def __init__(
        self,
        config: SiglipVisionConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        num_hidden_layers_override: int | None = None,
        require_post_norm: bool | None = None,
        prefix: str = "",
        use_head: bool | None = False,
    ) -> None:
        super().__init__()

        self.quant_config = quant_config
        self.vision_model = SiglipVisionTransformer(
            config,
            quant_config=quant_config,
            num_hidden_layers_override=num_hidden_layers_override,
            require_post_norm=require_post_norm,
            prefix=maybe_prefix(prefix, "vision_model"),
# ... omitted for brevity ...
    def get_input_embeddings(self) -> nn.Module:
        return self.vision_model.embeddings.patch_embedding
# ... omitted for brevity ...
    def forward(
        self,
        pixel_values: torch.Tensor,
        interpolate_pos_encoding: bool = False,
        select_layers: list[int] | None = None,
        feature_select_strategy: VisionFeatureSelectStrategy | None = None,
    ) -> torch.Tensor:
        return self.vision_model(
            pixel_values=pixel_values,
            interpolate_pos_encoding=interpolate_pos_encoding,
            select_layers=select_layers,
            feature_select_strategy=feature_select_strategy,
        )
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]
        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        layer_count = len(self.vision_model.encoder.layers)

        for name, loaded_weight in weights:
            # post_layernorm is optional in SiglipVisionModel
            if (
                name.startswith("vision_model.post_layernorm")
                and self.vision_model.post_layernorm is None
            ):
                continue
```
**EN:** Defines `SiglipVisionModel`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `get_input_embeddings`, `dtype`, `device`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SiglipVisionModel`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `get_input_embeddings`, `dtype`, `device`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SiglipEmbeddingModel` (lines 1023-1274)
```python
@default_pooling_type(seq_pooling_type="CLS")
@MULTIMODAL_REGISTRY.register_processor(
    SiglipMultiModalProcessor,
    info=SiglipProcessingInfo,
    dummy_inputs=SiglipDummyInputsBuilder,
)
class SiglipEmbeddingModel(nn.Module, SupportsMultiModal, SupportsQuant):
    is_pooling_model = True

# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config: SiglipConfig = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config

        if hasattr(config, "num_labels"):
            config.num_labels = 0

        text_config = config.text_config
        vision_config = config.vision_config

        self.text_embed_dim = text_config.hidden_size
        self.vision_embed_dim = vision_config.hidden_size
        self.text_projection_size = text_config.projection_size

        with self._mark_language_model(vllm_config):
            self.text_model = SiglipTextTransformer(
# ... omitted for brevity ...
    def _parse_and_validate_image_input(
        self, **kwargs: object
    ) -> SiglipImagePixelInputs | None:
        pixel_values = kwargs.pop("pixel_values", None)
        if pixel_values is None:
# ... omitted for brevity ...
        return SiglipImagePixelInputs(
            type="pixel_values",
            data=pixel_values,
            resolve_bindings={"h": expected_h, "w": expected_w},
        )
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor:
        if intermediate_tensors is not None:
            raise RuntimeError("PP is not supported for this model")

        # Multimodal inputs (image embeddings)
        if not self._is_text_input:
            return inputs_embeds

        # NOTE: inputs_embeds in model runner has size text_config.projection_size
        # (instead of text_config.hidden_size) to accommodate image embeddings
        hidden_size = self.text_embed_dim
        if inputs_embeds.shape[1] > hidden_size:
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        loader = AutoWeightsLoader(
            self,
            skip_substrs=[".position_ids"],
            ignore_unexpected_prefixes=["logit_scale.", "logit_bias."],
        )

        return loader.load_weights(weights)
```
**EN:** Defines `SiglipEmbeddingModel`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from nn.Module, SupportsMultiModal, SupportsQuant. Key methods such as `get_placeholder_str`, `__init__`, `get_text_features`, `_flip_sequences_by_position_ids`, `get_image_features` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SiglipEmbeddingModel`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 nn.Module、SupportsMultiModal、SupportsQuant。 `get_placeholder_str`, `__init__`, `get_text_features`, `_flip_sequences_by_position_ids`, `get_image_features` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Distributed execution**
  - **EN:** The implementation is aware of tensor parallelism or pipeline parallelism, so layers and weights can be sharded across devices.
  - **CN:** 实现考虑了张量并行或流水线并行，因此层和权重可以在多设备间切分。
- **Quantized weights**
  - **EN:** Quantization hooks allow the model to load compressed checkpoints or specialized linear layers.
  - **CN:** 量化钩子使模型能够加载压缩检查点或专用线性层。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。

## Dependencies / 依赖关系
- **Standard library**: collections.abc, functools, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.distributed, vllm.inputs, vllm.model_executor.layers.activation, vllm.model_executor.layers.attention, vllm.model_executor.layers.conv, vllm.model_executor.layers.linear
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .interfaces_base, .utils, .vision
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
