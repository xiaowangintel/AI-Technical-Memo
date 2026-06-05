# radio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/radio.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Radio multimodal model adapter used for inference in vLLM. / 实现 Radio 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-30)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Copyright (c) 2023-2024, NVIDIA CORPORATION.  All rights reserved.
#
# NVIDIA CORPORATION and its licensors retain all intellectual property
# and proprietary rights in and to this software, related documentation
# and any modifications thereto.  Any use, reproduction, disclosure or
# distribution of this software and related documentation without an express
# license agreement from NVIDIA CORPORATION is strictly prohibited.

import math
from collections.abc import Iterable
from dataclasses import dataclass
from itertools import accumulate, repeat
from typing import TypeAlias

import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange
from transformers import PretrainedConfig

from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.model_executor.models.intern_vit import (
    InternParallelAttention,
    InternVisionEncoder,
    InternVisionEncoderLayer,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, torch.nn.functional, einops supply framework primitives, while internal modules like vllm.model_executor.layers.quantization, vllm.model_executor.model_loader.weight_utils, vllm.model_executor.models.intern_vit connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, torch.nn.functional, einops 这样的外部依赖提供基础框架能力，而 vllm.model_executor.layers.quantization, vllm.model_executor.model_loader.weight_utils, vllm.model_executor.models.intern_vit 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `input_dim_t, norm_t` (lines 31-32)
```python
input_dim_t: TypeAlias = int | tuple[int, int]
norm_t: TypeAlias = tuple[float, float, float] | torch.Tensor
```
**EN:** This assignment block centers on `input_dim_t, norm_t` and records module-level configuration, dispatch metadata, or constant defaults used later in the file.
**CN:** 这个赋值代码块以 `input_dim_t, norm_t` 为核心，记录了后续逻辑会使用的模块级配置、分发表元数据或常量默认值。

### Class `ClsToken` (lines 60-106)
```python
class ClsToken(nn.Module):
    def __init__(
        self,
        ndim: int,
        num_tokens: int = 1,
        enabled: bool = True,
        register_multiple: int | None = None,
        num_registers: int | None = None,
    ):
        super().__init__()

        self.ndim = ndim
        self.enabled = enabled
        self.num_registers = 0
        self.num_tokens = num_tokens
        if enabled:
            if num_registers:
                self.num_registers = num_registers
            elif register_multiple:
                self.num_registers = register_multiple - (
                    num_tokens % register_multiple
                )

            scale = ndim**-0.5
            self.token = nn.Parameter(
                torch.randn(num_tokens + self.num_registers, ndim) * scale
            )

        else:
            self.token = None

        self.num_patches = self.num_tokens + self.num_registers

    def forward(self, x: torch.Tensor):
        if self.token is None:
            return x

        token = self.token.unsqueeze(0).expand(x.shape[0], -1, -1)
        x = torch.cat(
            [
                token,
                x,
            ],
            dim=1,
        )

        return x
```
**EN:** Defines `ClsToken`, a supporting module used by the surrounding model implementation. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `ClsToken`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `ViTPatchGenerator` (lines 109-469)
```python
class ViTPatchGenerator(nn.Module):
    def __init__(
        self,
        #  config: PretrainedConfig,
        patch_size: int,
        embed_dim: int,
        input_dims: input_dim_t,
        abs_pos: bool = True,
        normalize_patches: bool = False,
        cls_token: bool = False,
        max_input_dims: input_dim_t | None = None,
        pos_dropout: float = 0.0,
        return_pos_enc: bool = False,
        num_cls_tokens: int = 1,
        register_multiple: int | None = None,
        num_registers: int | None = None,
        patch_bias: bool = False,
        temporal_patch_size: int = 1,
        separate_video_embedder: bool = True,
        device=None,
# ... omitted for brevity ...
    def forward(
        self, x: torch.Tensor, imgs_sizes: list[tuple[int, int]] | None = None
    ) -> torch.Tensor:
        if imgs_sizes is not None:
            patches = self.embedder(x)
            patches, pos_enc = self.apply_pos_enc_dynamic(
                patches, imgs_sizes=imgs_sizes
            )
            patches = self.cls_token_dynamic(patches, imgs_sizes=imgs_sizes)
        else:
            patches = self.embed_patches(x)
            patches, pos_enc = self.apply_pos_enc(patches, input_size=x.shape[2:])
            patches = self.cls_token(patches)
        patches = self.patch_normalizer(patches)
# ... omitted for brevity ...
        Args:
            x: [num_frames, 3, H, W] tensor of video frames

        Returns:
            Embedded patches with temporal compression applied.
        """
        assert self.temporal_patch_size > 1
        T = self.temporal_patch_size
        input_size = x.shape[2:]

        patches = self.im_to_patches(x)  # [N, num_patches, 3*P*P]
        num_frames, num_spatial, feat_dim = patches.shape

        # Pad to a multiple of T by repeating the last frame so that
# ... omitted for brevity ...
    def _load_embed(self, src_embed: torch.Tensor, targ_embed: nn.Parameter):
        if src_embed.shape != targ_embed.shape:
            src_size = int(math.sqrt(src_embed.shape[1]))

            assert src_size**2 == src_embed.shape[1], (
                "Unable to interpolate non-square embedding"
            )

            src_embed = rearrange(
                src_embed, "b (h w) c -> b c h w", h=src_size, w=src_size
            )
            src_embed = F.interpolate(
                src_embed,
                size=(self.num_rows, self.num_cols),
                mode="bicubic",
                align_corners=True,
                antialias=False,
            )
            src_embed = rearrange(src_embed, "b c h w -> b (h w) c")
```
**EN:** Defines `ViTPatchGenerator`, a supporting module used by the surrounding model implementation. It inherits from nn.Module. Key methods such as `__init__`, `forward`, `forward_video`, `apply_pos_enc_dynamic`, `cls_token_dynamic` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `ViTPatchGenerator`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module。 `__init__`, `forward`, `forward_video`, `apply_pos_enc_dynamic`, `cls_token_dynamic` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Im2Patches` (lines 472-493)
```python
class Im2Patches(nn.Module):
    def __init__(self, patch_size: int):
        super().__init__()
        self.patch_size = patch_size

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        if self.patch_size == 1:
            patches = x.flatten(2)
            patches = patches.permute(0, 2, 1)
            return patches

        py = x.shape[-2] // self.patch_size
        px = x.shape[-1] // self.patch_size
        patches = rearrange(
            x,
            "b c (py yy) (px xx) -> b (py px) (c yy xx)",
            py=py,
            yy=self.patch_size,
            px=px,
            xx=self.patch_size,
        )
        return patches
```
**EN:** Defines `Im2Patches`, a supporting module used by the surrounding model implementation. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Im2Patches`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `ViTPatchLinear` (lines 496-509)
```python
class ViTPatchLinear(nn.Linear):
    def __init__(
        self,
        patch_size: int,
        embed_dim: int,
        bias: bool = False,
        temporal_patch_size: int = 1,
        **factory,
    ):
        super().__init__(
            3 * temporal_patch_size * (patch_size**2), embed_dim, bias=bias, **factory
        )
        self.patch_size = patch_size
        self.temporal_patch_size = temporal_patch_size
```
**EN:** Defines `ViTPatchLinear`, a supporting module used by the surrounding model implementation. It inherits from nn.Linear. Key methods such as `__init__` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `ViTPatchLinear`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Linear。 `__init__` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `RadioParallelAttention` (lines 518-534)
```python
class RadioParallelAttention(InternParallelAttention):
    def forward(
        self, x: torch.Tensor, mask_meta: MaskMetadata | None = None
    ) -> torch.Tensor:
        qkv, _ = self.qkv(x)
        q, k, v = qkv.chunk(3, dim=-1)

        if self.qk_normalization:
            q, k = self._apply_qk_norm(q, k)

        cu_seqlens, max_seqlen = None, None
        if mask_meta is not None:
            cu_seqlens = mask_meta.cu_seqlens
            max_seqlen = mask_meta.max_seqlen
        out = self.attn(q, k, v, cu_seqlens=cu_seqlens, max_seqlen=max_seqlen)
        out, _ = self.proj(out)
        return out
```
**EN:** Defines `RadioParallelAttention`, a attention block that projects Q/K/V tensors and dispatches the cache-aware attention path. It inherits from InternParallelAttention. Key methods such as `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `RadioParallelAttention`，它是一个负责投影 Q/K/V 张量并分发带缓存注意力路径的注意力模块。 它继承自 InternParallelAttention。 `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `RadioVisionEncoderLayer` (lines 537-553)
```python
class RadioVisionEncoderLayer(InternVisionEncoderLayer):
    def __init__(self, *args, **kwargs) -> None:
        super().__init__(*args, attn_cls=RadioParallelAttention, **kwargs)

    def forward(
        self,
        hidden_states: torch.Tensor,
        mask_meta: MaskMetadata | None = None,
    ):
        hidden_states = (
            hidden_states
            + self.attn(self.norm1(hidden_states), mask_meta=mask_meta) * self.ls1
        )

        hidden_states = hidden_states + self.mlp(self.norm2(hidden_states)) * self.ls2

        return hidden_states
```
**EN:** Defines `RadioVisionEncoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from InternVisionEncoderLayer. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `RadioVisionEncoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 InternVisionEncoderLayer。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `RadioVisionEncoder` (lines 556-568)
```python
class RadioVisionEncoder(InternVisionEncoder):
    def __init__(self, *args, **kwargs) -> None:
        super().__init__(*args, layer_cls=RadioVisionEncoderLayer, **kwargs)

    def forward(
        self,
        inputs_embeds: torch.Tensor,
        mask_meta: MaskMetadata | None = None,
    ):
        hidden_states = inputs_embeds
        for encoder_layer in self.layers:
            hidden_states = encoder_layer(hidden_states, mask_meta=mask_meta)
        return hidden_states
```
**EN:** Defines `RadioVisionEncoder`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from InternVisionEncoder. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `RadioVisionEncoder`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 InternVisionEncoder。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `RadioInternVisionModel` (lines 571-692)
```python
class RadioInternVisionModel(nn.Module):
    packed_modules_mapping = {
        "qkv": ["qkv"],
    }

    def __init__(
        self,
        config: PretrainedConfig = None,
        quant_config: QuantizationConfig | None = None,
        *,
        num_hidden_layers_override: int | None = None,
        num_dummy_heads: int = 0,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.img_size, self.grid_size, self.num_patches = self._init_img_size(
            to_2tuple(config.patch_size), config.image_size
        )
        max_img_size = int(
            round(config.cpe_max_size / config.patch_size) * config.patch_size
        )
        self.temporal_patch_size = config.video_temporal_patch_size
# ... omitted for brevity ...
    def _init_img_size(self, patch_size, img_size: int | tuple[int, int]):
        if img_size is None:
            return None, None, None
        img_size = to_2tuple(img_size)
        grid_size = tuple([s // p for s, p in zip(img_size, patch_size)])
        num_patches = grid_size[0] * grid_size[1]
        return img_size, grid_size, num_patches
# ... omitted for brevity ...
    def get_input_embeddings(self):
        return self.embeddings
# ... omitted for brevity ...
    def forward(
        self,
        x: torch.Tensor,
        imgs_sizes: list[tuple[int, int]] | None = None,
        num_frames: int | None = None,
    ) -> torch.FloatTensor:
        T = self.temporal_patch_size

        # Build packed-sequence metadata for MMEncoderAttention when needed.
        mask_meta = None
        packed_batch_size = None  # Original batch size before packing

        if num_frames is not None and T > 1:
            # Conv3d video: all tubelets have the same sequence length.
            # Pack [num_tubelets, seq_per_tubelet, hidden] → [1, total, hidden]
            hidden_states = self.patch_generator.forward_video(x)
            packed_batch_size, seq_per_tubelet, hidden_dim = hidden_states.shape
            hidden_states = hidden_states.reshape(1, -1, hidden_dim)
            mask_meta = self._inter_image_mask_metadata_from_seq_lens(
```
**EN:** Defines `RadioInternVisionModel`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `_init_img_size`, `get_input_embeddings`, `inter_image_mask_metadata`, `_inter_image_mask_metadata_from_seq_lens` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `RadioInternVisionModel`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `_init_img_size`, `get_input_embeddings`, `inter_image_mask_metadata`, `_inter_image_mask_metadata_from_seq_lens` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `RadioModel` (lines 695-822)
```python
class RadioModel(nn.Module):
    packed_modules_mapping = {
        "qkv": ["qkv"],
    }

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        num_hidden_layers_override: int | None = None,
        num_dummy_heads: int = 0,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.model = RadioInternVisionModel(
            config=config,
            quant_config=quant_config,
            num_hidden_layers_override=num_hidden_layers_override,
            num_dummy_heads=num_dummy_heads,
            prefix=prefix,
        )
# ... omitted for brevity ...
    def forward(
        self,
        pixel_values: torch.Tensor | None = None,
        pixel_embeds: torch.Tensor | None = None,
        *,
        imgs_sizes: list[tuple[int, int]] | None = None,
        num_frames: int | None = None,
    ) -> tuple[torch.FloatTensor, torch.FloatTensor]:
        y = self.model(
            pixel_values,
# ... omitted for brevity ...
            weights_list = list(weights.items())
        else:
            weights_list = list(weights)

        for name, weight in weights_list:
            if not name.startswith("radio_model."):
                # Skip non-radio weights
                continue

            sub = name[len("radio_model.") :]  # drop "radio_model." prefix

            # Skip buffers not used in vLLM
            if sub in {"summary_idxs"}:
                continue
# ... omitted for brevity ...
    def _extract_final(
        self, y: torch.Tensor, imgs_sizes: list[tuple[int, int]] | None = None
    ) -> tuple[torch.FloatTensor, torch.FloatTensor]:
        # Remove CLS + REGISTERS tokens
        num_skip = self.model.patch_generator.num_skip
        patch_size = self.model.patch_generator.patch_size
        num_cls_tokens = self.model.patch_generator.num_cls_tokens
        if imgs_sizes is None:
            all_summary = y[:, :num_cls_tokens]
            all_feat = y[:, num_skip:]
        else:
            all_patches = []
            summaries = []
            current_pos = 0
            for num_patches in calc_seq_lens(imgs_sizes, patch_size):
                patches = y[
                    :, current_pos + num_skip : current_pos + num_skip + num_patches, :
                ]
                all_patches.append(patches)
```
**EN:** Defines `RadioModel`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from nn.Module. Key methods such as `__init__`, `forward`, `load_weights`, `_extract_final` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `RadioModel`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 nn.Module。 `__init__`, `forward`, `load_weights`, `_extract_final` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **Position encoding**
  - **EN:** Rotary or related positional logic appears in the file to preserve token order information efficiently.
  - **CN:** 文件中出现了 Rotary 等位置编码逻辑，用于高效保留 token 顺序信息。

## Dependencies / 依赖关系
- **Standard library**: math, collections.abc, dataclasses, itertools, typing
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, torch.nn.functional, einops, transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.model_executor.layers.quantization, vllm.model_executor.model_loader.weight_utils, vllm.model_executor.models.intern_vit
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
