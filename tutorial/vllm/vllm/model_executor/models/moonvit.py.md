# moonvit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/moonvit.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Moonvit multimodal model adapter used for inference in vLLM. / 实现 Moonvit 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-68)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# ruff: noqa: E501
# Adapted from https://huggingface.co/moonshotai/Kimi-VL-A3B-Instruct/blob/main/modeling_kimi_vl.py
# This file is meant to be used in kimi_vl.py only
# Copyright 2025 The Moonshot AI Team, DeepSeek-AI, and HuggingFace Inc. team. All rights reserved.
#
# The code is based on llava (llava/modeling_llava.py) and DeepSeek-V3 (DeepSeek-V3/modeling_deepseek.py), but modified for KimiVL.
#
# Licensing Information:
# - Code derived from llava (llava/modeling_llava.py) and DeepSeek-V3 (DeepSeek-V3/modeling_deepseek.py) is licensed under the Apache License, Version 2.0.
# - Other parts of the code are licensed under the MIT License.
#
# Apache License, Version 2.0:
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# ... omitted for brevity ...
from transformers.modeling_utils import PreTrainedModel

from vllm.distributed import divide, get_tensor_model_parallel_world_size
from vllm.model_executor.layers.attention import MMEncoderAttention
from vllm.model_executor.layers.conv import Conv2dLayer
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.models.utils import maybe_prefix
from vllm.model_executor.models.vision import is_vit_use_data_parallel
from vllm.platforms import current_platform
from vllm.transformers_utils.configs.moonvit import MoonViTConfig
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch, torch.nn, torch.nn.functional, transformers.activations supply framework primitives, while internal modules like vllm.distributed, vllm.model_executor.layers.attention, vllm.model_executor.layers.conv, vllm.model_executor.layers.linear connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch, torch.nn, torch.nn.functional, transformers.activations 这样的外部依赖提供基础框架能力，而 vllm.distributed, vllm.model_executor.layers.attention, vllm.model_executor.layers.conv, vllm.model_executor.layers.linear 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Function `_apply_rope_input_validation` (lines 69-73)
```python
def _apply_rope_input_validation(x, freqs_cis):
    assert x.ndim == freqs_cis.ndim + 1, (x.shape, freqs_cis.shape)
    assert x.shape[:-2] == freqs_cis.shape[:-1], (x.shape, freqs_cis.shape)
    assert x.shape[-1] == 2 * freqs_cis.shape[-1], (x.shape, freqs_cis.shape)
    assert freqs_cis.dtype == torch.complex64, freqs_cis.dtype
```
**EN:** The function `_apply_rope_input_validation` helps provide a reusable helper for the surrounding model code. Its main inputs are `x`, `freqs_cis`.
**CN:** 函数 `_apply_rope_input_validation` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `x`、`freqs_cis`。

### Function `apply_rope` (lines 76-96)
```python
def apply_rope(
    xq: torch.Tensor, xk: torch.Tensor, freqs_cis: torch.Tensor
) -> tuple[torch.Tensor, torch.Tensor]:
    """
    Args: (The leading dimensions of all inputs should be the same)
        xq: query, tensor of shape (..., num_heads, head_dim)
        xk: key, tensor of shape (..., num_heads, head_dim)
        freqs_cis: tensor of shape (..., head_dim/2), dtype=torch.complex64. It contains the precomputed cis(freqs) for each position in the 2D grid.
    Returns:
        xq_out, xk_out: tensors of shape (..., num_heads, head_dim)
    """
    _apply_rope_input_validation(xq, freqs_cis)
    _apply_rope_input_validation(xk, freqs_cis)

    freqs_cis = freqs_cis.unsqueeze(-2)  # ..., 1, head_dim/2
    # ..., num_heads, head_dim/2
    xq_ = torch.view_as_complex(xq.float().view(*xq.shape[:-1], -1, 2))
    xk_ = torch.view_as_complex(xk.float().view(*xq.shape[:-1], -1, 2))
    xq_out = torch.view_as_real(xq_ * freqs_cis).flatten(-2)  # ..., num_heads, head_dim
    xk_out = torch.view_as_real(xk_ * freqs_cis).flatten(-2)  # ..., num_heads, head_dim
    return xq_out.type_as(xq), xk_out.type_as(xk)
```
**EN:** The function `apply_rope` helps provide a reusable helper for the surrounding model code. Its main inputs are `xq`, `xk`, `freqs_cis`. Docstring hint: "Args: (The leading dimensions of all inputs should be the same) xq: query, tensor of shape (..., num_heads, head_dim) xk: key, tensor of shape (..., num_heads, head_dim) freqs_cis: tensor of shape (..., head_dim/2), dtype=torch.complex64."
**CN:** 函数 `apply_rope` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `xq`、`xk`、`freqs_cis`。 文档提示：“Args: (The leading dimensions of all inputs should be the same) xq: query, tensor of shape (..., num_heads, head_dim) xk: key, tensor of shape (..., num_heads, head_dim) freqs_cis: tensor of shape (..., head_dim/2), dtype=torch.complex64。”

### Class `Learnable2DInterpPosEmb` (lines 99-130)
```python
class Learnable2DInterpPosEmb(nn.Module):
    def __init__(
        self, height: int, width: int, dim: int, interpolation_mode: str = "bicubic"
    ) -> None:
        super().__init__()
        self.height = height
        self.width = width
        self.interpolation_mode = interpolation_mode
        self.weight = nn.Parameter(torch.empty(height, width, dim))
        self.reset_parameters()

    def reset_parameters(self):
        nn.init.normal_(self.weight)

    def forward(self, x: torch.Tensor, grid_hws: torch.Tensor) -> torch.Tensor:
        pos_embs = []
        for shape in grid_hws.tolist():
            if shape == self.weight.shape[:-1]:
                pos_embs.append(self.weight.flatten(end_dim=1))
            else:
                pos_embs.append(
                    F.interpolate(
                        self.weight.permute((2, 0, 1)).unsqueeze(0),
                        size=shape,
                        mode=self.interpolation_mode,
                    )
                    .squeeze(0)
                    .permute((1, 2, 0))
                    .flatten(end_dim=1)
                )
        out = x + torch.cat(pos_embs)
        return out
```
**EN:** Defines `Learnable2DInterpPosEmb`, a supporting module used by the surrounding model implementation. It inherits from nn.Module. Key methods such as `__init__`, `reset_parameters`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Learnable2DInterpPosEmb`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module。 `__init__`, `reset_parameters`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MoonVisionPatchEmbed` (lines 133-173)
```python
class MoonVisionPatchEmbed(nn.Module):
    def __init__(
        self,
        out_dim: int,
        in_dim: int = 3,
        patch_size: int | tuple[int, int] = (14, 14),
        pos_emb_height: int = 14,
        pos_emb_width: int = 14,
    ):
        super().__init__()
        assert isinstance(patch_size, (int, Sequence)), (
            f"Invalid patch_size type: {type(patch_size)}"
        )
        if isinstance(patch_size, int):
            patch_size = (patch_size, patch_size)
        assert len(patch_size) == 2, (
            f"Expected patch_size to be a tuple of 2, got {patch_size}"
        )
        self.patch_size = patch_size

        self.proj = Conv2dLayer(
            in_dim, out_dim, kernel_size=patch_size, stride=patch_size
        )

        self.pos_emb = Learnable2DInterpPosEmb(
            height=pos_emb_height, width=pos_emb_width, dim=out_dim
        )

    def forward(self, x: torch.Tensor, grid_hw: torch.Tensor) -> torch.Tensor:
        """
        Args:
            x (L, Channels): input tensor
            grid_hw (N, 2): grid height and width

        Returns:
            (L, Cout) tensor
        """
        x = self.proj(x).view(x.size(0), -1)
        # apply positional embedding
        x = self.pos_emb(x, grid_hw)
        return x
```
**EN:** Defines `MoonVisionPatchEmbed`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MoonVisionPatchEmbed`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Rope2DPosEmb` (lines 176-295)
```python
class Rope2DPosEmb(nn.Module):
    """2D rotary position embedding with multi-resolution support.

    This class is intended to be used in the following way:
    1. Before training, create an instance of Rope2DPosEmb. This instance will hold the precomputed cis.
    2. Before each forward pass, call `get_freqs_cis_by_*` to get the `freqs_cis` tensor for this iteration.
    3. During the forward pass, pass the `freqs_cis` tensor to each attention layer, and call `apply` just before each attention operation.
        The rope is shared across all attention layers and all heads.

# ... omitted for brevity ...
    def __init__(
        self,
        dim: int,
        max_height: int,
        max_width: int,
        theta_base=10000,
        device=current_platform.device_type,
    ):
        super().__init__()
        self.dim = dim
        assert self.dim % 4 == 0, "dim must be divisible by 4"
        self.max_height = max_height
        self.max_width = max_width
        self.theta_base = theta_base
        self.device = device
# ... omitted for brevity ...
    def extra_repr(self):
        return f"dim={self.dim}, max_height={self.max_height}, max_width={self.max_width}, theta_base={self.theta_base}"
# ... omitted for brevity ...
    @cached_property
    def precomputed_freqs_cis(self) -> torch.Tensor:
        """Calculate the cis(freqs) for each position in the 2D grid.

        Return: complex tensor of shape (max_height, max_width, dim//2) and value:
            height axis: ret[h, w, 2*i] = cis(h * theta_base**(-4*i/dim))
            weight axis: ret[h, w, 2*i+1] = cis(w * theta_base**(-4*i/dim))   with (i in [0, dim//4))
            note: `cis` is a mathematical notation defined by cis x = cos x + i sin x,
        """
        N = self.max_height * self.max_width
        flat_pos = torch.arange(0, N).float().to(self.device)
        x_pos = flat_pos % self.max_width
        y_pos = flat_pos // self.max_width
        dim_range = (
            torch.arange(0, self.dim, 4)[: (self.dim // 4)].float().to(self.device)
        )  # C/4
        freqs = 1.0 / (self.theta_base ** (dim_range / self.dim))
        x_freqs = torch.outer(x_pos, freqs).float()  # N, C/4
        y_freqs = torch.outer(y_pos, freqs).float()  # N, C/4
# ... omitted for brevity ...
    def get_freqs_cis_by_seqlens(self, grid_hws: torch.Tensor) -> torch.Tensor:
        """
        Args:
            grid_hws (torch.Tensor): containing list of (height, width) or (t, height, width) tuples.
        Returns:
            freqs_cis: tensor of shape (sum(t * height * width), dim//2)
        """
        shapes = grid_hws.tolist()
        assert all(
            1 <= h <= self.max_height and 1 <= w <= self.max_width for h, w in shapes
        ), (
            shapes,
            self.max_height,
            self.max_width,
        )
        freqs_cis = torch.cat(
            [
                self.precomputed_freqs_cis[:h, :w].reshape(-1, self.dim // 2)
                for h, w in shapes
```
**EN:** Defines `Rope2DPosEmb`, a supporting module used by the surrounding model implementation. It inherits from nn.Module. Key methods such as `__init__`, `extra_repr`, `precomputed_freqs_cis`, `get_freqs_cis_by_seqlens`, `get_freqs_cis_by_idx` show where construction, forward execution, or weight adaptation happens. Docstring hint: "2D rotary position embedding with multi-resolution support."
**CN:** 定义 `Rope2DPosEmb`，它是一个被周边模型实现复用的支撑模块。 它继承自 nn.Module。 `__init__`, `extra_repr`, `precomputed_freqs_cis`, `get_freqs_cis_by_seqlens`, `get_freqs_cis_by_idx` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“2D rotary position embedding with multi-resolution support。”

### Class `MLP2` (lines 298-335)
```python
class MLP2(nn.Module):
    """
    Args:
        dims: [in_dim, hidden_dim, out_dim]
        bias: whether to use bias in linear layer.
    """

    def __init__(
        self,
        dims: list[int],
        activation,
        bias: bool = True,
        prefix: str = "",
    ):
        super().__init__()
        assert len(dims) == 3
        self.use_data_parallel = is_vit_use_data_parallel()
        self.fc0 = ColumnParallelLinear(
            dims[0],
            dims[1],
            bias=bias,
            prefix=maybe_prefix(prefix, "fc0"),
            disable_tp=self.use_data_parallel,
        )
        self.fc1 = RowParallelLinear(
            dims[1],
            dims[2],
            bias=bias,
            prefix=maybe_prefix(prefix, "fc1"),
            disable_tp=self.use_data_parallel,
        )
        self.activation = activation

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x, _ = self.fc0(x)
        x = self.activation(x)
        x, _ = self.fc1(x)
        return x
```
**EN:** Defines `MLP2`, a feed-forward block that expands hidden states and projects them back. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Args: dims: [in_dim, hidden_dim, out_dim] bias: whether to use bias in linear layer."
**CN:** 定义 `MLP2`，它是一个扩展隐藏状态并再投影回主维度的前馈模块。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Args: dims: [in_dim, hidden_dim, out_dim] bias: whether to use bias in linear layer。”

### Class `MoonVitEncoderLayer` (lines 338-454)
```python
class MoonVitEncoderLayer(nn.Module):
    def __init__(
        self,
        num_heads: int,
        hidden_dim: int,
        mlp_dim: int,
        prefix: str = "",
        *,
        activation=F.gelu,
        attn_bias: bool = False,
    ):
        super().__init__()
        self.use_data_parallel = is_vit_use_data_parallel()

        self.num_heads = num_heads
        self.hidden_dim = hidden_dim
        self.hidden_size_per_attention_head = self.hidden_dim // self.num_heads
        self.tp_size = (
            1 if self.use_data_parallel else get_tensor_model_parallel_world_size()
        )
# ... omitted for brevity ...
    def attention_qkvpacked(
        self,
        x: torch.Tensor,
        cu_seqlens: torch.Tensor,
        rope_freqs_cis: torch.Tensor | None = None,
    ):
        """
        Args:
            x (torch.Tensor): (seqlen, hidden_dim)
            cu_seqlens (torch.Tensor):
        """
        seq_length = x.size(0)
        xqkv, _ = self.wqkv(x)

        qkv_shape = xqkv.size()[:-1] + (
            3,
            self.num_attention_heads_per_partition,
            self.hidden_size_per_attention_head,
        )
# ... omitted for brevity ...
    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: torch.Tensor,
        rope_freqs_cis: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """
        Args:
            hidden_states: non-packed (B, N, D) or packed (L, D). if non-packed, seqlens should be None, if packed, seqlens should be set

        Returns:
            output: same shape of input, non-packed (B, N, D) for non-packed input, (L, D) for packed input
        """
        residual = hidden_states
        hidden_states = self.norm0(hidden_states)
        attn_out = self.attention_qkvpacked(
            hidden_states, cu_seqlens, rope_freqs_cis=rope_freqs_cis
        )
        hidden_states = residual + attn_out
```
**EN:** Defines `MoonVitEncoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `attention_qkvpacked`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MoonVitEncoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `attention_qkvpacked`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `MoonVitEncoder` (lines 457-501)
```python
class MoonVitEncoder(nn.Module):
    def __init__(
        self,
        hidden_dim: int,
        num_layers: int,
        block_cfg: dict,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.rope_2d = Rope2DPosEmb(
            block_cfg["hidden_dim"] // block_cfg["num_heads"], 512, 512
        )
        self.blocks = nn.ModuleList(
            [
                MoonVitEncoderLayer(
                    prefix=f"{prefix}.blocks.{layer_idx}",
                    **block_cfg,
                )
                for layer_idx in range(num_layers)
            ]
        )
        self.final_layernorm = nn.LayerNorm(hidden_dim)

    def forward(
        self, hidden_states: torch.Tensor, grid_hw: torch.Tensor
    ) -> torch.Tensor:
        rope_freqs_cis = self.rope_2d.get_freqs_cis_by_seqlens(grid_hws=grid_hw)

        lengths = torch.cat(
            (
                torch.zeros(1, device=hidden_states.device, dtype=grid_hw.dtype),
                (grid_hw[:, 0] * grid_hw[:, 1]).to(hidden_states.device),
            )
        )
        cu_seqlens = lengths.cumsum(dim=0, dtype=torch.int32)

        for _, block in enumerate(self.blocks):
            hidden_states = block(
                hidden_states, cu_seqlens, rope_freqs_cis=rope_freqs_cis
            )

        hidden_states = self.final_layernorm(hidden_states)

        return hidden_states
```
**EN:** Defines `MoonVitEncoder`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from nn.Module. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MoonVitEncoder`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 nn.Module。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Function `patch_merger` (lines 504-530)
```python
def patch_merger(
    x: torch.Tensor,
    grid_hw: torch.Tensor,
    merge_kernel_size: list[int, int] = (2, 2),
) -> list[torch.Tensor]:
    d_model = x.size(-1)

    outputs = []
    pre_sum = 0
    for x_shape in grid_hw.tolist():
        height, width = x_shape[0], x_shape[1]
        # Get the current sequence
        seq = x[pre_sum : pre_sum + height * width]
        # Reshape along self.merge_kernel_size and concat to the last dimension
        kernel_height, kernel_width = merge_kernel_size
        new_height, new_width = height // kernel_height, width // kernel_width
        reshaped_seq = seq.view(
            new_height, kernel_height, new_width, kernel_width, d_model
        )
        reshaped_seq = reshaped_seq.permute(0, 2, 1, 3, 4).contiguous()
        padded_seq = reshaped_seq.view(
            new_height * new_width, kernel_height * kernel_width, -1
        )
        outputs.append(padded_seq)
        pre_sum += height * width

    return outputs
```
**EN:** The function `patch_merger` helps provide a reusable helper for the surrounding model code. Its main inputs are `x`, `grid_hw`, `merge_kernel_size`.
**CN:** 函数 `patch_merger` 用于为周边模型代码提供可复用辅助函数。 它的主要输入包括 `x`、`grid_hw`、`merge_kernel_size`。

### Class `MoonVitPretrainedModel` (lines 533-589)
```python
class MoonVitPretrainedModel(PreTrainedModel):
    config_class = MoonViTConfig
    model_type = "moonvit"
    _no_split_modules = ["PackingTransformer"]
    _supports_flash_attn_2 = True
    _supports_sdpa = True

    def __init__(
        self,
        config: MoonViTConfig,
        prefix: str = "",
        *inputs,
        **kwargs,
    ):
        super().__init__(config, *inputs, **kwargs)
        config = deepcopy(config)
        self.merge_kernel_size = config.merge_kernel_size
        self.hidden_size = config.hidden_size
        self.patch_size = config.patch_size
        self.vit_processing_type = "rope_2d"
        self.patch_embed = MoonVisionPatchEmbed(
            out_dim=config.hidden_size,
            patch_size=config.patch_size,
            pos_emb_height=config.init_pos_emb_height,
            pos_emb_width=config.init_pos_emb_width,
        )

        self.encoder = MoonVitEncoder(
            hidden_dim=config.hidden_size,
            num_layers=config.num_hidden_layers,
            block_cfg={
                "num_heads": config.num_attention_heads,
                "hidden_dim": config.hidden_size,
                "mlp_dim": config.intermediate_size,
                "activation": ACT2FN["gelu_pytorch_tanh"],
                "attn_bias": True,
            },
            prefix=f"{prefix}.encoder",
        )

    def forward(
        self, pixel_values: torch.Tensor, grid_hw: torch.Tensor
    ) -> torch.Tensor:
        """
        Args:
            pixel_values (torch.Tensor): The input pixel values.
            grid_hw (torch.Tensor): The grid height and width.

        Returns:
            torch.Tensor: The output tokens.
        """
        hidden_states = self.patch_embed(pixel_values, grid_hw)
        hidden_states = self.encoder(hidden_states, grid_hw)
        hidden_states = patch_merger(
            hidden_states, grid_hw, merge_kernel_size=self.merge_kernel_size
        )
        return hidden_states
```
**EN:** Defines `MoonVitPretrainedModel`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from PreTrainedModel. Key methods such as `__init__`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `MoonVitPretrainedModel`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 PreTrainedModel。 `__init__`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

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
- **Position encoding**
  - **EN:** Rotary or related positional logic appears in the file to preserve token order information efficiently.
  - **CN:** 文件中出现了 Rotary 等位置编码逻辑，用于高效保留 token 顺序信息。

## Dependencies / 依赖关系
- **Standard library**: collections.abc, copy, functools
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch, torch.nn, torch.nn.functional, transformers.activations, transformers.modeling_utils
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.distributed, vllm.model_executor.layers.attention, vllm.model_executor.layers.conv, vllm.model_executor.layers.linear, vllm.model_executor.models.utils, vllm.model_executor.models.vision, vllm.platforms, vllm.transformers_utils.configs.moonvit
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
