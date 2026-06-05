# openpangu_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/openpangu_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Openpangu VL multimodal model adapter used for inference in vLLM. / 实现 Openpangu VL 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-85)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
#
# Copyright (c) 2025 Huawei Technologies Co., Ltd. All Rights Reserved.
# Adapted from vllm/model_executor/models/qwen2_5_vl.py
# Copyright 2025 The vLLM team.
# Copyright 2025 The Qwen Team.
#
# This file is a part of the vllm-ascend project.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# ... omitted for brevity ...
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import MultiModalDataItems
from vllm.multimodal.processing import (
    PromptReplacement,
    PromptUpdate,
    PromptUpdateDetails,
)
from vllm.sequence import IntermediateTensors
from vllm.utils.tensor_schema import TensorSchema, TensorShape
from vllm.v1.attention.backends.registry import AttentionBackendEnum

from .vision import get_vit_attn_backend
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, torch.nn.functional, einops supply framework primitives, while internal modules like vllm.config, vllm.distributed, vllm.model_executor.layers.activation, vllm.model_executor.layers.attention.mm_encoder_attention connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, torch.nn.functional, einops 这样的外部依赖提供基础框架能力，而 vllm.config, vllm.distributed, vllm.model_executor.layers.activation, vllm.model_executor.layers.attention.mm_encoder_attention 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger` (lines 86-86)
```python
logger = logging.get_logger(__name__)
```
**EN:** This assignment initializes the module-level logger used for diagnostics, backend selection messages, and compatibility warnings.
**CN:** 这个赋值语句初始化了模块级日志记录器，用于输出诊断信息、后端选择消息和兼容性警告。

### Class `OpenPanguVisionAttention` (lines 89-168)
```python
class OpenPanguVisionAttention(nn.Module):
    def __init__(
        self,
        embed_dim: int,
        num_heads: int,
        projection_size: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size_per_attention_head = dist_utils.divide(
            projection_size, num_heads
        )
        self.tp_size = parallel_state.get_tensor_model_parallel_world_size()
        self.tp_rank = parallel_state.get_tensor_model_parallel_rank()
        self.num_attention_heads_per_partition = dist_utils.divide(
            num_heads, self.tp_size
        )

        self.qkv = QKVParallelLinear(
# ... omitted for brevity ...
    def forward(
        self,
        x: torch.Tensor,
        cu_seqlens: torch.Tensor,
        cos: torch.Tensor,
        sin: torch.Tensor,
    ) -> torch.Tensor:
        seq_length, _ = x.size()
        x, bias = self.qkv(x)
        if bias is not None:
            x = x + bias
        q, k, v = x.chunk(3, dim=1)

        q, k, v = (
            rearrange(
                x, "s (b n d) -> b s n d", d=self.hidden_size_per_attention_head, b=1
            ).contiguous()
            for x in (q, k, v)
        )
```
**EN:** Defines `OpenPanguVisionAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OpenPanguVisionAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OpenPanguVisionMLP` (lines 171-220)
```python
class OpenPanguVisionMLP(nn.Module):
    def __init__(
        self,
        in_features: int,
        hidden_features: int,
        bias: bool = False,
        act_fn: Callable[[torch.Tensor], torch.Tensor] = F.silu,
        vision_config=None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_act = vision_config.hidden_act
        if self.hidden_act == "silu":
            tp_size = parallel_state.get_tensor_model_parallel_world_size()
            if hidden_features % tp_size != 0:
                hidden_features = (hidden_features + tp_size - 1) // tp_size * tp_size
            self.gate_up_proj = MergedColumnParallelLinear(
                input_size=in_features,
                output_sizes=[hidden_features] * 2,
                bias=bias,
                quant_config=quant_config,
                prefix=f"{prefix}.gate_up_proj",
            )
        else:
            self.up_proj = ColumnParallelLinear(
                in_features,
                hidden_features,
                bias=bias,
                quant_config=quant_config,
                prefix=f"{prefix}.up_proj",
            )

        self.down_proj = RowParallelLinear(
            hidden_features,
            in_features,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
        )
        self.act_fn = act_fn

    def forward(self, x: torch.Tensor):
        if self.hidden_act == "silu":
            x, _ = self.gate_up_proj(x)
        else:
            x, _ = self.up_proj(x)
        x = self.act_fn(x)
        x, _ = self.down_proj(x)
        return x
```
**EN:** Defines `OpenPanguVisionMLP`, a feed-forward block that expands hidden states and projects them back. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OpenPanguVisionMLP`，它是一个扩展隐藏状态并再投影回主维度的前馈模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OpenPanguVisionBlock` (lines 223-268)
```python
class OpenPanguVisionBlock(nn.Module):
    def __init__(
        self,
        dim: int,
        num_heads: int,
        mlp_hidden_dim: int,
        act_fn: Callable[[torch.Tensor], torch.Tensor] = F.silu,
        norm_layer: Callable[[int], nn.Module] | None = None,
        vision_config=None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.norm1 = norm_layer(dim)
        self.norm2 = norm_layer(dim)
        self.attn = OpenPanguVisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
            projection_size=dim,
            quant_config=quant_config,
            prefix=f"{prefix}.attn",
        )
        self.mlp = OpenPanguVisionMLP(
            dim,
            mlp_hidden_dim,
            act_fn=act_fn,
            bias=True,
            vision_config=vision_config,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )

    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: torch.Tensor,
        cos: torch.Tensor,
        sin: torch.Tensor,
    ) -> torch.Tensor:
        hidden_states = hidden_states + self.attn(
            self.norm1(hidden_states), cu_seqlens=cu_seqlens, cos=cos, sin=sin
        )
        hidden_states = hidden_states + self.mlp(self.norm2(hidden_states))
        return hidden_states
```
**EN:** Defines `OpenPanguVisionBlock`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OpenPanguVisionBlock`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OpenPanguVisionRotaryEmbedding` (lines 271-296)
```python
class OpenPanguVisionRotaryEmbedding(nn.Module):
    def __init__(self, dim: int, theta: float = 10000.0) -> None:
        super().__init__()
        self.inv_freq = 1.0 / (
            theta ** (torch.arange(0, dim, 2, dtype=torch.float) / dim)
        )
        self._seq_len_cached = 0
        self._freqs_cached = None

    def update_freqs_cache(self, seqlen: int) -> None:
        if seqlen > self._seq_len_cached:
            seqlen *= 2
            self._seq_len_cached = seqlen
            seq = torch.arange(
                seqlen, device=self.inv_freq.device, dtype=self.inv_freq.dtype
            )
            freqs = torch.outer(seq, self.inv_freq)
            self._freqs_cached = freqs

    def forward(self, seqlen: int) -> torch.Tensor:
        self.update_freqs_cache(seqlen)
        return (
            self._freqs_cached[:seqlen]
            if self._freqs_cached is not None
            else self._freqs_cached
        )
```
**EN:** Defines `OpenPanguVisionRotaryEmbedding`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `update_freqs_cache`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OpenPanguVisionRotaryEmbedding`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `update_freqs_cache`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OpenPanguVisionPatchEmbed` (lines 299-334)
```python
class OpenPanguVisionPatchEmbed(nn.Module):
    def __init__(
        self,
        patch_size: int = 14,
        temporal_patch_size: int = 2,
        in_channels: int = 3,
        hidden_size: int = 1152,
    ) -> None:
        super().__init__()
        self.patch_size = patch_size
        self.temporal_patch_size = temporal_patch_size
        self.hidden_size = hidden_size
        self.input_size = (
            self.patch_size * self.patch_size * in_channels * self.temporal_patch_size
        )

        kernel_size = (temporal_patch_size, patch_size, patch_size)
        self.proj = nn.Conv3d(
            in_channels,
            hidden_size,
            kernel_size=kernel_size,
            stride=kernel_size,
            bias=False,
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        if x.shape[-1] != self.input_size:
            x = torch.cat(
                [
                    x.reshape(-1, self.patch_size * self.patch_size),
                    x.reshape(-1, self.patch_size * self.patch_size),
                ],
                dim=-1,
            ).reshape(-1, self.input_size)
        x = x.matmul(self.proj.weight.data.view(self.hidden_size, -1).transpose(0, 1))
        return x
```
**EN:** Defines `OpenPanguVisionPatchEmbed`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OpenPanguVisionPatchEmbed`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `OpenPanguVisionPatchMerger` (lines 337-373)
```python
class OpenPanguVisionPatchMerger(nn.Module):
    def __init__(
        self,
        d_model: int,
        context_dim: int,
        norm_layer: Callable[[int], nn.Module] | None = None,
        spatial_merge_size: int = 2,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.hidden_size = context_dim * (spatial_merge_size**2)
        self.ln_q = norm_layer(context_dim)
        self.mlp = nn.Sequential(
            ColumnParallelLinear(
                self.hidden_size,
                self.hidden_size,
                bias=True,
                quant_config=quant_config,
                prefix=f"{prefix}.mlp.0",
                return_bias=False,
            ),
            nn.GELU(),
            RowParallelLinear(
                self.hidden_size,
                d_model,
                bias=True,
                quant_config=quant_config,
                prefix=f"{prefix}.mlp.2",
                return_bias=False,
            ),
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.mlp(self.ln_q(x).view(-1, self.hidden_size))
```
**EN:** Defines `OpenPanguVisionPatchMerger`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `OpenPanguVisionPatchMerger`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Function `rescale_and_normalize` (lines 1297-1324)
```python
def rescale_and_normalize(
    images: "torch.Tensor",
    do_rescale: bool,
    rescale_factor: float,
    do_normalize: bool,
    image_mean: float | list[float],
    image_std: float | list[float],
    dtype: torch.dtype = torch.bfloat16,
) -> "torch.Tensor":
    """
    Rescale and normalize images.
    """
    image_mean, image_std, do_rescale = _fuse_mean_std_and_rescale_factor(
        do_normalize=do_normalize,
        image_mean=image_mean,
        image_std=image_std,
        do_rescale=do_rescale,
        rescale_factor=rescale_factor,
        device=images.device,
    )
    # if/elif as we use fused rescale and normalize if both are set to True
    if do_normalize:
        images = normalize(images.to(dtype=torch.float32), image_mean, image_std)
    elif do_rescale:
        images = rescale(images, rescale_factor)
    images = images.to(dtype)

    return images
```
**EN:** The function `rescale_and_normalize` helps provide a reusable helper for the surrounding model code. Its main inputs are `images`, `do_rescale`, `rescale_factor`, `do_normalize`, `image_mean`, `image_std`. Docstring hint: "Rescale and normalize images."
**CN:** 函数 `rescale_and_normalize` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `images`、`do_rescale`、`rescale_factor`、`do_normalize`、`image_mean`、`image_std`。 文档提示：“Rescale and normalize images。”

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
- **External libraries**: torch, torch.nn, torch.nn.functional, einops, torchvision.transforms, transformers.utils
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.config, vllm.distributed, vllm.model_executor.layers.activation, vllm.model_executor.layers.attention.mm_encoder_attention, vllm.model_executor.layers.layernorm, vllm.model_executor.layers.linear, vllm.model_executor.layers.quantization, vllm.model_executor.layers.quantization.auto_gptq
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .vision
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
