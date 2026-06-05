# mllama4.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/mllama4.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the M-Llama4 multimodal model adapter used for inference in vLLM. / 实现 M-Llama4 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-86)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
#
# Copyright 2025 the LLAMA4, Meta Inc., vLLM, and HuggingFace Inc. team.
# All rights reserved.
#
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
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import (
    MixtureOfExperts,
    MultiModalEmbeddings,
    SupportsEagle3,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
)
from .llama4 import Llama4ForCausalLM
from .utils import AutoWeightsLoader, StageMissingLayer, maybe_prefix
from .vision import is_vit_use_data_parallel, run_dp_sharded_vision_model
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, transformers, transformers.image_utils, transformers.models.llama4 supply framework primitives, while internal modules like vllm.compilation.decorators, vllm.config, vllm.config.multimodal, vllm.distributed connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, transformers, transformers.image_utils, transformers.models.llama4 这样的外部依赖提供基础框架能力，而 vllm.compilation.decorators, vllm.config, vllm.config.multimodal, vllm.distributed 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Class `Llama4ImagePatchInputs` (lines 87-116)
```python
class Llama4ImagePatchInputs(TensorSchema):
    """
    Dimensions:
        - batch_size: Batch size
        - total_num_chunks: Batch size * number of chunks
        - num_channels: Number of channels
        - image_size: Size of each image
    """

    type: Literal["pixel_values"] = "pixel_values"

    pixel_values: Annotated[
        torch.Tensor,
        TensorShape("total_num_chunks", "num_channels", "image_size", "image_size"),
    ]

    patches_per_image: Annotated[torch.Tensor, TensorShape("batch_size")]
    """
    The number of total patches for each image in the batch.
    
    This is used to split the embeddings which has the first two dimensions
    flattened just like `pixel_values`.
    """

    aspect_ratios: Annotated[torch.Tensor, TensorShape("batch_size", 2)]
    """
    A list of aspect ratios corresponding to the number of tiles
    in each dimension that each image in the batch corresponds to.
    Each aspect ratio is a pair (ratio_h, ratio_w).
    """
```
**EN:** Defines `Llama4ImagePatchInputs`, a supporting module used by the surrounding model implementation. It inherits from TensorSchema. Docstring hint: "Dimensions: - batch_size: Batch size - total_num_chunks: Batch size * number of chunks - num_channels: Number of channels - image_size: Size of each image."
**CN:** 定义 `Llama4ImagePatchInputs`，它是一个被周边模型实现复用的支撑模块。 它继承自 TensorSchema。 文档提示：“Dimensions: - batch_size: Batch size - total_num_chunks: Batch size * number of chunks - num_channels: Number of channels - image_size: Size of each image。”

### Class `Llama4VisionMLP` (lines 119-157)
```python
class Llama4VisionMLP(nn.Module):
    def __init__(
        self,
        input_size: int,
        intermediate_size: int,
        output_size: int,
        bias: bool,
        output_activation: bool,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        use_data_parallel = is_vit_use_data_parallel()
        self.fc1 = ColumnParallelLinear(
            input_size=input_size,
            output_size=intermediate_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.fc1",
            disable_tp=use_data_parallel,
        )
        self.fc2 = RowParallelLinear(
            input_size=intermediate_size,
            output_size=output_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.fc2",
            disable_tp=use_data_parallel,
        )
        self.activation_fn = nn.GELU()
        self.output_activation = output_activation

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.fc1(hidden_states)
        hidden_states = self.activation_fn(hidden_states)
        hidden_states, _ = self.fc2(hidden_states)
        if self.output_activation:
            return self.activation_fn(hidden_states)
        return hidden_states
```
**EN:** Defines `Llama4VisionMLP`, a feed-forward block that expands hidden states and projects them back. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4VisionMLP`，它是一个扩展隐藏状态并再投影回主维度的前馈模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4MultiModalProjector` (lines 160-179)
```python
class Llama4MultiModalProjector(nn.Module):
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.linear_1 = ColumnParallelLinear(
            input_size=config.vision_config.vision_output_dim,
            output_size=config.text_config.hidden_size,
            bias=False,
            quant_config=quant_config,
            gather_output=True,
            prefix=f"{prefix}.linear_1",
        )

    def forward(self, image_features):
        hidden_states, _ = self.linear_1(image_features)
        return hidden_states
```
**EN:** Defines `Llama4MultiModalProjector`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4MultiModalProjector`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4VisionPixelShuffleMLP` (lines 207-232)
```python
class Llama4VisionPixelShuffleMLP(nn.Module):
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.pixel_shuffle_ratio = config.pixel_shuffle_ratio
        self.inner_dim = int(
            config.projector_input_dim // (self.pixel_shuffle_ratio**2)
        )
        self.output_dim = config.projector_output_dim
        self.mlp = Llama4VisionMLP(
            input_size=config.intermediate_size,
            intermediate_size=config.projector_input_dim,
            output_size=config.projector_output_dim,
            bias=config.multi_modal_projector_bias,
            output_activation=True,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )

    def forward(self, encoded_patches: torch.Tensor) -> torch.Tensor:
        encoded_patches = pixel_shuffle(encoded_patches, self.pixel_shuffle_ratio)
        return self.mlp(encoded_patches)
```
**EN:** Defines `Llama4VisionPixelShuffleMLP`, a feed-forward block that expands hidden states and projects them back. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4VisionPixelShuffleMLP`，它是一个扩展隐藏状态并再投影回主维度的前馈模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4VisionAttention` (lines 235-333)
```python
class Llama4VisionAttention(nn.Module):
    def __init__(
        self,
        config: Llama4VisionConfig,
        quant_config: QuantizationConfig | None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        use_data_parallel = is_vit_use_data_parallel()
        self.tp_size = (
            1 if use_data_parallel else get_tensor_model_parallel_world_size()
        )
        self.embed_dim = config.hidden_size
        self.num_heads = config.num_attention_heads
        self.head_dim = config.hidden_size // self.num_heads
        assert self.num_heads % self.tp_size == 0
        self.num_local_heads = self.num_heads // self.tp_size
        self.q_size = self.num_local_heads * self.head_dim
        self.kv_size = self.num_local_heads * self.head_dim
# ... omitted for brevity ...
    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        input_shape = hidden_states.shape[:-1]

        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)

        q = q.view(q.shape[0], q.shape[1], self.num_local_heads, self.head_dim)
        k = k.view(k.shape[0], k.shape[1], self.num_local_heads, self.head_dim)
        q, k = self.rotary_emb(q, k)

        q = q.view(q.shape[0], q.shape[1], -1)
        k = k.view(k.shape[0], k.shape[1], -1)

        attn_output = self.attn(q, k, v)
        attn_output = attn_output.reshape(*input_shape, -1).contiguous()
        attn_output, _ = self.o_proj(attn_output)
```
**EN:** Defines `Llama4VisionAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4VisionAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4VisionEncoderLayer` (lines 336-383)
```python
class Llama4VisionEncoderLayer(nn.Module):
    def __init__(
        self,
        config: Llama4VisionConfig,
        quant_config: QuantizationConfig | None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.num_attention_heads = config.num_attention_heads
        self.intermediate_size = config.intermediate_size

        self.self_attn = Llama4VisionAttention(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.mlp = Llama4VisionMLP(
            input_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            output_size=config.hidden_size,
            bias=True,
            output_activation=False,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )

        self.input_layernorm = nn.LayerNorm(config.hidden_size)
        self.post_attention_layernorm = nn.LayerNorm(config.hidden_size)

    def forward(
        self,
        hidden_state: torch.Tensor,
    ):
        # Self Attention
        residual = hidden_state
        hidden_state = self.input_layernorm(hidden_state)
        hidden_state = self.self_attn(hidden_state)
        hidden_state = residual + hidden_state

        # Feed forward
        residual = hidden_state
        hidden_state = self.post_attention_layernorm(hidden_state)
        hidden_state = self.mlp(hidden_state)
        hidden_state = residual + hidden_state

        outputs = (hidden_state,)
        return outputs
```
**EN:** Defines `Llama4VisionEncoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4VisionEncoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4VisionEncoder` (lines 386-424)
```python
class Llama4VisionEncoder(nn.Module):
    def __init__(
        self,
        config: Llama4VisionConfig,
        quant_config: QuantizationConfig | None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.layers = nn.ModuleList(
            [
                Llama4VisionEncoderLayer(
                    config,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layers.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )

    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        r"""
        Args:
            hidden_states: Input tensor of shape
                (batch_size, sequence_length, hidden_size).
                Hidden states from the model embeddings, representing
                the input tokens.
                associated vectors than the model's internal embedding
                lookup matrix.
        """

        for encoder_layer in self.layers:
            layer_outputs = encoder_layer(hidden_states)
            hidden_states = layer_outputs[0]

        return hidden_states
```
**EN:** Defines `Llama4VisionEncoder`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4VisionEncoder`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4UnfoldConvolution` (lines 427-454)
```python
class Llama4UnfoldConvolution(nn.Module):
    def __init__(
        self,
        config: Llama4VisionConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        kernel_size = config.patch_size
        if isinstance(kernel_size, int):
            kernel_size = (kernel_size, kernel_size)
        self.unfold = torch.nn.Unfold(kernel_size=kernel_size, stride=config.patch_size)
        use_data_parallel = is_vit_use_data_parallel()
        self.linear = ColumnParallelLinear(
            input_size=config.num_channels * kernel_size[0] * kernel_size[1],
            output_size=config.hidden_size,
            bias=False,
            gather_output=True,
            quant_config=quant_config,
            prefix=f"{prefix}.linear",
            disable_tp=use_data_parallel,
        )

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states = self.unfold(hidden_states)
        hidden_states = hidden_states.permute(0, 2, 1)
        hidden_states, _ = self.linear(hidden_states)
        return hidden_states
```
**EN:** Defines `Llama4UnfoldConvolution`, a supporting module used by the surrounding model implementation. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4UnfoldConvolution`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4VisionModel` (lines 457-546)
```python
@support_torch_compile(
    dynamic_arg_dims={"images_flattened": 0},
    enable_if=should_torch_compile_mm_encoder,
    is_encoder=True,
)
class Llama4VisionModel(nn.Module):
    def __init__(
        self,
        config: Llama4VisionConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.image_size = config.image_size
        self.patch_size = config.patch_size
        self.hidden_size = config.hidden_size
        self.num_channels = config.num_channels

        self.num_patches = (self.image_size // self.patch_size) ** 2 + 1
        self.scale = config.hidden_size**-0.5

        self.patch_embedding = Llama4UnfoldConvolution(
            config,
            quant_config=quant_config,
# ... omitted for brevity ...
    def forward(
        self,
        images_flattened: torch.Tensor,
    ) -> torch.Tensor:
        # Patch embedding
        hidden_state = self.patch_embedding(images_flattened)
        num_tiles, num_patches, hidden_dim = hidden_state.shape

        # Add cls token
        class_embedding = self.class_embedding.expand(
            hidden_state.shape[0], 1, hidden_state.shape[-1]
        )
        hidden_state = torch.cat([hidden_state, class_embedding], dim=1)
        num_patches += 1

        # Position embeddings
        hidden_state = hidden_state.reshape(
            num_tiles,
            1,
```
**EN:** Defines `Llama4VisionModel`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4VisionModel`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Llama4ForConditionalGeneration` (lines 721-1159)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Mllama4MultiModalProcessor,
    info=Mllama4ProcessingInfo,
    dummy_inputs=Mllama4DummyInputsBuilder,
)
class Llama4ForConditionalGeneration(
    nn.Module,
    SupportsMultiModal,
    SupportsPP,
    MixtureOfExperts,
    SupportsEagle3,
    SupportsLoRA,
):
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config
        self.use_data_parallel = multimodal_config.mm_encoder_tp_mode == "data"

        self.vllm_config = vllm_config
        self.config = config
        self.quant_config = quant_config
        self.multimodal_config = multimodal_config

        with self._mark_tower_model(vllm_config, "image"):
            with set_current_vllm_config(vllm_config):
                self.vision_model = Llama4VisionModel(
                    config=config.vision_config,
                    quant_config=None,
                    prefix=maybe_prefix(prefix, "vision_model"),
                )
# ... omitted for brevity ...
    def forward(
# ... omitted for brevity ...
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        if intermediate_tensors is not None:
            inputs_embeds = None

        return self.language_model(
            input_ids, positions, intermediate_tensors, inputs_embeds
        )
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        return self.language_model.compute_logits(hidden_states)
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".self_attn.qkv_proj", ".self_attn.q_proj", "q"),
            (".self_attn.qkv_proj", ".self_attn.k_proj", "k"),
            (".self_attn.qkv_proj", ".self_attn.v_proj", "v"),
            # Shared expert gate_up_proj stacking
            (".shared_expert.gate_up_proj", ".shared_expert.gate_proj", 0),
            (".shared_expert.gate_up_proj", ".shared_expert.up_proj", 1),
            # Feed forward gate_up_proj stacking (for non-MoE layers if any)
            (".feed_forward.gate_up_proj", ".feed_forward.gate_proj", 0),
            (".feed_forward.gate_up_proj", ".feed_forward.up_proj", 1),
        ]
        params_dict = dict(self.named_parameters())
        updated_params: set[str] = set()

        # Separate and rename weights
        language_model_weights, other_weights = self._separate_and_rename_weights(
            weights
```
**EN:** Defines `Llama4ForConditionalGeneration`, a supporting module used by the surrounding model implementation. It inherits from nn.Module, SupportsMultiModal, SupportsPP. Key methods such as `get_placeholder_str`, `__init__`, `set_aux_hidden_state_layers`, `get_eagle3_default_aux_hidden_state_layers`, `set_eplb_state` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Llama4ForConditionalGeneration`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module、SupportsMultiModal、SupportsPP。 `get_placeholder_str`, `__init__`, `set_aux_hidden_state_layers`, `get_eagle3_default_aux_hidden_state_layers`, `set_eplb_state` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **Standard library**: math, collections.abc, itertools, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, transformers, transformers.image_utils, transformers.models.llama4, transformers.models.llama4.image_processing_llama4_fast
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.compilation.decorators, vllm.config, vllm.config.multimodal, vllm.distributed, vllm.inputs, vllm.model_executor.layers.attention, vllm.model_executor.layers.fused_moe, vllm.model_executor.layers.linear
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .llama4, .utils, .vision
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
