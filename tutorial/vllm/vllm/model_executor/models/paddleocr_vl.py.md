# paddleocr_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/paddleocr_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Paddleocr VL multimodal model adapter used for inference in vLLM. / 实现 Paddleocr VL 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-89)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Copyright (c) 2025 PaddlePaddle Authors. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import math
from collections.abc import Iterable, Iterator, Mapping, Sequence
# ... omitted for brevity ...
from vllm.utils.tensor_schema import TensorSchema, TensorShape
from vllm.v1.attention.backends.registry import AttentionBackendEnum

from .ernie45 import Ernie4_5ForCausalLM
from .interfaces import MultiModalEmbeddings, SupportsMRoPE, SupportsMultiModal
from .siglip import SiglipMLP
from .utils import (
    AutoWeightsLoader,
    PPMissingLayer,
    WeightsMapper,
    is_pp_missing_parameter,
    maybe_prefix,
)
from .vision import get_vit_attn_backend
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as numpy, torch, torch.nn, einops supply framework primitives, while internal modules like vllm.config, vllm.config.multimodal, vllm.distributed, vllm.inputs connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 numpy, torch, torch.nn, einops 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.config.multimodal, vllm.distributed, vllm.inputs 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Function `smart_resize` (lines 90-130)
```python
def smart_resize(
    height: int,
    width: int,
    factor: int = 28,
    min_pixels: int = 28 * 28 * 130,
    max_pixels: int = 28 * 28 * 1280,
):
    """Rescales the image so that the following conditions are met:

    1. Both dimensions (height and width) are divisible by 'factor'.

    2. The total number of pixels is within the range ['min_pixels', 'max_pixels'].

    3. The aspect ratio of the image is maintained as closely as possible.

    """

    if height < factor:
        width = round((width * factor) / height)
        height = factor

    if width < factor:
        height = round((height * factor) / width)
        width = factor

    if max(height, width) / min(height, width) > 200:
        raise ValueError(
            f"absolute aspect ratio must be smaller than 200, "
            f"got {max(height, width) / min(height, width)}"
        )
    h_bar = round(height / factor) * factor
    w_bar = round(width / factor) * factor
    if h_bar * w_bar > max_pixels:
        beta = math.sqrt((height * width) / max_pixels)
        h_bar = math.floor(height / beta / factor) * factor
        w_bar = math.floor(width / beta / factor) * factor
    elif h_bar * w_bar < min_pixels:
        beta = math.sqrt(min_pixels / (height * width))
        h_bar = math.ceil(height * beta / factor) * factor
        w_bar = math.ceil(width * beta / factor) * factor
    return h_bar, w_bar
```
**EN:** The function `smart_resize` helps provide a reusable helper for the surrounding model code. Its main inputs are `height`, `width`, `factor`, `min_pixels`, `max_pixels`. Docstring hint: "Rescales the image so that the following conditions are met: 1."
**CN:** 函数 `smart_resize` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `height`、`width`、`factor`、`min_pixels`、`max_pixels`。 文档提示：“Rescales the image so that the following conditions are met: 1。”

### Class `Projector` (lines 321-382)
```python
class Projector(nn.Module):
    def __init__(
        self,
        text_config: PretrainedConfig,
        vision_config: PretrainedConfig,
        prefix: str = "",
    ):
        super().__init__()
        self.text_config = text_config
        self.vision_config = vision_config
        self.merge_kernel_size = (2, 2)

        self.hidden_size = (
            self.vision_config.hidden_size
            * self.merge_kernel_size[0]
            * self.merge_kernel_size[1]
        )

        self.pre_norm = torch.nn.LayerNorm(self.vision_config.hidden_size, eps=1e-05)
        self.linear_1 = nn.Linear(self.hidden_size, self.hidden_size, bias=True)
        self.act = GELUActivation()
        self.linear_2 = nn.Linear(
            self.hidden_size, self.text_config.hidden_size, bias=True
        )

    def forward(
        self,
        image_features: torch.Tensor,
        image_grid_thw: torch.Tensor,
    ) -> torch.Tensor:
# ... omitted for brevity ...
            for image_feature, image_grid in zip(image_features, image_grid_thw):
                image_feature = self.pre_norm(image_feature)
                t, h, w = image_grid

                image_feature = rearrange(
                    image_feature,
                    "(t h p1 w p2) d -> (t h w) (p1 p2 d)",
                    t=t,
                    h=h // m1,
                    p1=m1,
                    w=w // m2,
                    p2=m2,
                )
                hidden_states = self.linear_1(image_feature)
                hidden_states = self.act(hidden_states)
                hidden_states = self.linear_2(hidden_states)
                processed_features.append(hidden_states)

            return processed_features

        dims = image_features.shape[:-1]
        dim = image_features.shape[-1]
        image_features = image_features.view(np.prod(dims), dim)
        hidden_states = self.pre_norm(image_features).view(-1, self.hidden_size)
        hidden_states = self.linear_1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states = self.linear_2(hidden_states)

        return hidden_states.view(*dims, -1)
```
**EN:** Defines `Projector`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Projector`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SiglipVisionEmbeddings` (lines 397-530)
```python
class SiglipVisionEmbeddings(nn.Module):
    def __init__(self, config: PretrainedConfig):
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
        self.cache_position_embedding = dict()
        self.cache_position_count = dict()
# ... omitted for brevity ...
    def interpolate_pos_encoding(
        self,
        embeddings: torch.Tensor,
        height: int,
        width: int,
        is_after_patchify: bool = False,
    ) -> torch.Tensor:
        num_positions = self.position_embedding.weight.shape[0]

        patch_pos_embed = self.position_embedding.weight.unsqueeze(0)

        dim = embeddings.shape[-1]

        if is_after_patchify:
# ... omitted for brevity ...
            self.cache_position_count[grid] += 1
            return self.cache_position_embedding[grid]

        if len(self.cache_position_embedding) >= max_cache:
            min_hit_grid = min(
                self.cache_position_count,
                key=self.cache_position_count.get,
            )
            self.cache_position_count.pop(min_hit_grid)
            self.cache_position_embedding.pop(min_hit_grid)

        position_embedding = self.interpolate_pos_encoding(embeddings, h, w, True)
        self.cache_position_count[grid] = 1
        self.cache_position_embedding[grid] = position_embedding
# ... omitted for brevity ...
    def forward(
        self,
        pixel_values: torch.FloatTensor,
        position_ids: torch.Tensor | None = None,
        image_grid_thw: list[tuple[int, int, int] | list[tuple[int, int, int]]]
        | None = None,
        interpolate_pos_encoding=False,
    ) -> torch.Tensor:
        if pixel_values.dim() == 4:
            pixel_values = pixel_values.unsqueeze(0)
        if pixel_values.dim() == 5:
            if position_ids is None:
                raise ValueError(
                    "position_ids cannot be None when pixel_values.dim() is 5."
                )
            (
                batch_size,
                sequence_len,
                channel,
```
**EN:** Defines `SiglipVisionEmbeddings`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `interpolate_pos_encoding`, `fetch_position_embedding_lfu_cache`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SiglipVisionEmbeddings`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `interpolate_pos_encoding`, `fetch_position_embedding_lfu_cache`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SiglipAttention` (lines 552-659)
```python
class SiglipAttention(nn.Module):
    """SigLIP vision attention adapted from Qwen2.5-VisionAttention."""

    def __init__(
        self,
        *,
        embed_dim: int,
        num_heads: int,
        projection_size: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.tp_size = parallel_state.get_tensor_model_parallel_world_size()
        self.tp_rank = parallel_state.get_tensor_model_parallel_rank()
        self.hidden_size_per_attention_head = dist_utils.divide(
            projection_size, num_heads
        )
        self.num_attention_heads_per_partition = dist_utils.divide(
            num_heads, self.tp_size
        )
# ... omitted for brevity ...
    def split_qkv(self, qkv: torch.Tensor) -> tuple[torch.Tensor, ...]:
        seq_len, bs, _ = qkv.shape
        if self.tp_size > 1:
            qkv = all_gather_interleave(qkv, self.qkv_proj.hidden_size, self.tp_size)

        q, k, v = qkv.chunk(3, dim=2)

        if self.tp_size > 1:
            splitter = partial(
                dist_utils.split_tensor_along_last_dim, num_partitions=self.tp_size
            )
            q = splitter(q)[self.tp_rank]
            k = splitter(k)[self.tp_rank]
            v = splitter(v)[self.tp_rank]

        new_shape = (
            seq_len,
            bs,
            self.num_attention_heads_per_partition,
# ... omitted for brevity ...
    def forward(
        self,
        hidden_states: torch.Tensor,
        *,
        cu_seqlens: torch.Tensor,
        rotary_pos_emb: torch.Tensor | None,
        max_seqlen: torch.Tensor | None,
    ) -> torch.Tensor:
        batch_size, _, _ = hidden_states.shape

        x = rearrange(hidden_states, "b s d -> s b d")
        x, _ = self.qkv_proj(x)
        q, k, v = self.split_qkv(x)
        q, k, v = (rearrange(t, "s b h d -> b s h d") for t in (q, k, v))

        if rotary_pos_emb is not None:
            qk_concat = torch.cat([q, k], dim=0)
            qk_rotated = self.apply_rotary_emb(
                qk_concat,
```
**EN:** Defines `SiglipAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `split_qkv`, `forward` show where construction, forward execution, or weight adaptation happens. Docstring hint: "SigLIP vision attention adapted from Qwen2.5-VisionAttention."
**CN:** 定义 `SiglipAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `split_qkv`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“SigLIP vision attention adapted from Qwen2.5-VisionAttention。”

### Class `SiglipEncoderLayer` (lines 685-735)
```python
class SiglipEncoderLayer(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.embed_dim = config.hidden_size
        self.layer_norm1 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)
        self.self_attn = SiglipAttention(
            embed_dim=config.hidden_size,
            num_heads=config.num_attention_heads,
            projection_size=config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.layer_norm2 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)
        self.mlp = SiglipMLP(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )

    def forward(
        self,
        hidden_states: torch.Tensor,
        *,
        cu_seqlens: torch.Tensor,
        rotary_pos_emb: torch.Tensor | None,
        max_seqlen: torch.Tensor | None,
    ) -> torch.Tensor:
        residual = hidden_states

        hidden_states = self.layer_norm1(hidden_states)
        hidden_states = self.self_attn(
            hidden_states=hidden_states,
            cu_seqlens=cu_seqlens,
            rotary_pos_emb=rotary_pos_emb,
            max_seqlen=max_seqlen,
        )

        hidden_states = residual + hidden_states

        residual = hidden_states
        hidden_states = self.layer_norm2(hidden_states)
        hidden_states = self.mlp(hidden_states)

        hidden_states = residual + hidden_states

        return hidden_states
```
**EN:** Defines `SiglipEncoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SiglipEncoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SiglipEncoder` (lines 738-835)
```python
class SiglipEncoder(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        embed_dim = config.hidden_size
        num_heads = config.num_attention_heads
        head_dim = embed_dim // num_heads

        self.attn_backend = get_vit_attn_backend(
            head_size=head_dim,
            dtype=torch.get_default_dtype(),
        )

        self.layers = nn.ModuleList(
            [
# ... omitted for brevity ...
    @staticmethod
    def flatten_list(image_grid_thw):
        tmp_image_grid_thw = list()
        for image_grid in image_grid_thw:
            if isinstance(image_grid, list):
                tmp_image_grid_thw.extend(image_grid)
            else:
                tmp_image_grid_thw.append(image_grid)
        return tmp_image_grid_thw
# ... omitted for brevity ...
    def forward(
        self,
        inputs_embeds,
        cu_seqlens: torch.Tensor | None = None,
        image_grid_thw: list[tuple[int, int, int] | list[tuple[int, int, int]]]
        | None = None,
        height_position_ids: torch.Tensor | None = None,
        width_position_ids: torch.Tensor | None = None,
    ) -> torch.Tensor:
        device = inputs_embeds.device
        hidden_states = inputs_embeds

        flatten_image_grid_thw = self.flatten_list(image_grid_thw)

        if width_position_ids is None or height_position_ids is None:
            split_hids = list()
            split_wids = list()
            for t, h, w in flatten_image_grid_thw:
                image_pids = torch.arange(t * h * w, device=device) % (h * w)
```
**EN:** Defines `SiglipEncoder`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `flatten_list`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SiglipEncoder`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `flatten_list`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `SiglipVisionModel` (lines 886-994)
```python
class SiglipVisionModel(nn.Module):
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.vision_model = SiglipVisionTransformer(
            config,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "vision_model"),
        )
        self.quant_config = quant_config
# ... omitted for brevity ...
    def get_input_embeddings(self) -> nn.Module:
        return self.vision_model.embeddings.patch_embedding
# ... omitted for brevity ...
    def forward(
        self,
        pixel_values,
        interpolate_pos_encoding: bool = False,
        position_ids: torch.Tensor | None = None,
        image_grid_thw: list[tuple[int, int, int] | list[tuple[int, int, int]]]
        | None = None,
        cu_seqlens: torch.Tensor | None = None,
    ) -> BaseModelOutputWithPooling:
        return self.vision_model(
            pixel_values=pixel_values,
            interpolate_pos_encoding=interpolate_pos_encoding,
            position_ids=position_ids,
            image_grid_thw=image_grid_thw,
            cu_seqlens=cu_seqlens,
        )
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]
        params_dict = dict(self.named_parameters(remove_duplicate=False))
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            if "head.attention" in name or "head.layernorm" in name:
                continue
            if "head.mlp" in name or "head.probe" in name:
                continue
            if "packing_position_embedding" in name:
                continue
            if self.quant_config is not None and (
                scale_name := self.quant_config.get_cache_scale(name)
```
**EN:** Defines `SiglipVisionModel`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `dtype`, `device`, `get_input_embeddings`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `SiglipVisionModel`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `dtype`, `device`, `get_input_embeddings`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `PaddleOCRVLForConditionalGeneration` (lines 997-1222)
```python
@MULTIMODAL_REGISTRY.register_processor(
    PaddleOCRVLMultiModalProcessor,
    info=PaddleOCRVLProcessingInfo,
    dummy_inputs=PaddleOCRVLDummyInputsBuilder,
)
class PaddleOCRVLForConditionalGeneration(nn.Module, SupportsMultiModal, SupportsMRoPE):
    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            "model.": "language_model.model.",
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        if hasattr(config, "text_config"):
            text_config = config.text_config.to_dict()
            unsafe_keys = ["model_type", "architectures", "tie_word_embeddings"]
            for key in unsafe_keys:
                text_config.pop(key, None)
            config.update(text_config)

        quant_config = vllm_config.quant_config

        self.config = config

        with self._mark_tower_model(vllm_config, "image"):
            self.visual = SiglipVisionModel(
                config=config.vision_config,
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "visual"),
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        return self.language_model.compute_logits(hidden_states)
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs,
    ):
        if intermediate_tensors is not None:
            inputs_embeds = None

        return self.language_model(
            input_ids, positions, intermediate_tensors, inputs_embeds
        )
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(self)
        autoloaded_weights = loader.load_weights(weights, mapper=self.hf_to_vllm_mapper)
        return autoloaded_weights
```
**EN:** Defines `PaddleOCRVLForConditionalGeneration`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module, SupportsMultiModal, SupportsMRoPE. Key methods such as `get_placeholder_str`, `__init__`, `compute_logits`, `iter_mm_grid_thw`, `get_mrope_input_positions` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `PaddleOCRVLForConditionalGeneration`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module、SupportsMultiModal、SupportsMRoPE。 `get_placeholder_str`, `__init__`, `compute_logits`, `iter_mm_grid_thw`, `get_mrope_input_positions` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **Standard library**: math, collections.abc, functools, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: numpy, torch, torch.nn, einops, transformers, transformers.activations, transformers.image_utils, transformers.modeling_outputs
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.config.multimodal, vllm.distributed, vllm.inputs, vllm.model_executor.layers.attention, vllm.model_executor.layers.conv, vllm.model_executor.layers.linear, vllm.model_executor.layers.quantization
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .ernie45, .interfaces, .siglip, .utils, .vision
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
