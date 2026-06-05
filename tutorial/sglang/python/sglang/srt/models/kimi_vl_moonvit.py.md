# kimi_vl_moonvit.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/kimi_vl_moonvit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Kimi Vl Moonvit architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 Kimi Vl Moonvit 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-43: Module header and imports / 模块头与导入
```python
# SPDX-License-Identifier: Apache-2.0
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
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# MIT License:
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# ... truncated for brevity ...
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 57-60: Error handling block: optional dependency guard / 异常处理块：optional dependency guard
```python
try:
    from flash_attn.flash_attn_interface import flash_attn_varlen_func
except ImportError:
    flash_attn_varlen_func = None
```
**EN:** This block guards optional functionality and keeps the module resilient when dependencies are unavailable.
**CN:** 该代码块通过异常保护可选功能，以便在依赖缺失时保持模块可用。

### Lines 71-120: Function: multihead_attention() / 函数：multihead_attention()
```python
def multihead_attention(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    q_cu_seqlens: Optional[torch.Tensor] = None,
    k_cu_seqlens: Optional[torch.Tensor] = None,
):
    """Multi-head attention using flash attention 2.
    This function is used to handle the case where the query, key, and value are packed.
    Args:
        q, k, v: tensor of shape (tot_seqlens, num_heads, head_dim).
        q_cu_seqlens (torch.Tensor): cumulative sequence lengths of q.
            The first element should be 0 and the last element should be q.shape[0].
        k_cu_seqlens (torch.Tensor): cumulative sequence lengths of k.
            The first element should be 0 and the last element should be k.shape[0].

    Returns:
        output: shape (batch_size, seqlen, dim) or (tot_seqlens, dim) if packing,
            where dim = num_heads * head_dim
    """
    if flash_attn_varlen_func is None:
        raise ImportError(
            "flash_attn is not installed, this function needs flash_attn_varlen_func from flash_attn"
        )
    # Unified format legal check
    assert q.dim() == k.dim() == v.dim() == 3, "q, k, v must have 3 dims"
    assert q_cu_seqlens[-1] == q.shape[0], "q_cu_seqlens must sum to q.shape[0]"
    assert (
        k_cu_seqlens[-1] == k.shape[0] == v.shape[0]
    ), "k_cu_seqlens must sum to k.shape[0]"
    assert q.dtype in [
        torch.bfloat16,
        torch.float16,
    ], f"unsupported dtype {q.dtype} for multihead attn"

    max_seqlen_q = (q_cu_seqlens[1:] - q_cu_seqlens[:-1]).max().item()
    max_seqlen_k = (k_cu_seqlens[1:] - k_cu_seqlens[:-1]).max().item()
    attn_out = flash_attn_varlen_func(
        q,
        k,
# ... truncated for brevity ...
```
**EN:** This function implements multihead attention for the surrounding model/runtime logic. Key parameters include q, k, v, q_cu_seqlens.
**CN:** 该函数实现了 multihead attention 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 123-162: Function: sdpa_attention() / 函数：sdpa_attention()
```python
def sdpa_attention(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    q_cu_seqlens: Optional[torch.Tensor] = None,
    k_cu_seqlens: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    """Multi-head attention using torch scaled dot product attention.
    This function is used to handle the case where the query, key, and value are packed.
    Args:
        q, k, v: tensor of shape (tot_seqlens, num_heads, head_dim).
        q_cu_seqlens (torch.Tensor): cumulative sequence lengths of q.
            The first element should be 0 and the last element should be q.shape[0].
        k_cu_seqlens (torch.Tensor): cumulative sequence lengths of k.
            The first element should be 0 and the last element should be k.shape[0].

    Returns:
        output: shape (batch_size, seqlen, dim) or (tot_seqlens, dim) if packing,
            where dim = num_heads * head_dim
    """
    # Unified format legal check
    assert q.dim() == k.dim() == v.dim() == 3, "q, k, v must have 3 dims"
    assert q_cu_seqlens[-1] == q.shape[0], "q_cu_seqlens must sum to q.shape[0]"
    seq_length = q.shape[0]
    attention_mask = torch.zeros(
        [1, seq_length, seq_length], device=q.device, dtype=torch.bool
    )
    for i in range(1, len(q_cu_seqlens)):
        attention_mask[
            ...,
            q_cu_seqlens[i - 1] : q_cu_seqlens[i],
            q_cu_seqlens[i - 1] : q_cu_seqlens[i],
        ] = True
    q = q.transpose(0, 1)
    k = k.transpose(0, 1)
    v = v.transpose(0, 1)
    attn_output = F.scaled_dot_product_attention(q, k, v, attention_mask, dropout_p=0.0)
    attn_output = attn_output.transpose(0, 1)
    attn_output = attn_output.reshape(seq_length, -1)
    return attn_output
```
**EN:** This function implements sdpa attention for the surrounding model/runtime logic. Key parameters include q, k, v, q_cu_seqlens.
**CN:** 该函数实现了 sdpa attention 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 165-168: Assignment: VL_VISION_ATTENTION_FUNCTIONS / 赋值：VL_VISION_ATTENTION_FUNCTIONS
```python
VL_VISION_ATTENTION_FUNCTIONS = {
    "flash_attention_2": multihead_attention,
    "sdpa": sdpa_attention,
}
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 171-175: Function: _apply_rope_input_validation() / 函数：_apply_rope_input_validation()
```python
def _apply_rope_input_validation(x, freqs_cis):
    assert x.ndim == freqs_cis.ndim + 1, (x.shape, freqs_cis.shape)
    assert x.shape[:-2] == freqs_cis.shape[:-1], (x.shape, freqs_cis.shape)
    assert x.shape[-1] == 2 * freqs_cis.shape[-1], (x.shape, freqs_cis.shape)
    assert freqs_cis.dtype == torch.complex64, freqs_cis.dtype
```
**EN:** This function implements apply rope input validation for the surrounding model/runtime logic. Key parameters include x, freqs_cis.
**CN:** 该函数实现了 apply rope input validation 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 178-198: Function: apply_rope() / 函数：apply_rope()
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
**EN:** This function implements apply rope for the surrounding model/runtime logic. Key parameters include xq, xk, freqs_cis.
**CN:** 该函数实现了 apply rope 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 201-233: Class: Learnable2DInterpPosEmb / 类：Learnable2DInterpPosEmb
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
**EN:** This class defines Learnable2 D Interp Pos Emb inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Learnable2 D Interp Pos Emb，用于封装该模型组件的状态与方法。

### Lines 236-277: Class: MoonVisionPatchEmbed / 类：MoonVisionPatchEmbed
```python
class MoonVisionPatchEmbed(nn.Module):

    def __init__(
        self,
        out_dim: int,
        in_dim: int = 3,
        patch_size: Union[int, Tuple[int, int]] = (14, 14),
        pos_emb_height: int = 14,
        pos_emb_width: int = 14,
    ):
        super().__init__()
        assert isinstance(
            patch_size, (int, Sequence)
        ), f"Invalid patch_size type: {type(patch_size)}"
        if isinstance(patch_size, int):
            patch_size = (patch_size, patch_size)
        assert (
            len(patch_size) == 2
        ), f"Expected patch_size to be a tuple of 2, got {patch_size}"
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
# ... truncated for brevity ...
```
**EN:** This class defines Moon Vision Patch Embed inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Moon Vision Patch Embed，用于封装该模型组件的状态与方法。

### Lines 280-394: Class: Rope2DPosEmb / 类：Rope2DPosEmb
```python
class Rope2DPosEmb(nn.Module):
    """2D rotary position embedding with multi-resolution support.

    This class is intended to be used in the following way:
    1. Before training, create an instance of Rope2DPosEmb. This instance will hold the precomputed cis.
    2. Before each forward pass, call `get_freqs_cis_by_*` to get the `freqs_cis` tensor for this iteration.
    3. During the forward pass, pass the `freqs_cis` tensor to each attention layer, and call `apply` just before each attention operation.
        The rope is shared across all attention layers and all heads.

    Refs:
    - RoFormer: https://arxiv.org/abs/2104.09864
    - VisionLLaMA: https://arxiv.org/abs/2403.00522
    - https://github.com/Meituan-AutoML/VisionLLaMA/blob/main/dit/models.py

    Args:
        dim (int): usually the multi-head attention dimension, should be divisible by 4 (TODO: relax this constraint if needed)
        max_height (int): the maximum height of the 2D grid
        max_width (int): the maximum width of the 2D grid
        theta_base (float): the base of the theta
        device (str): the device to store the precomputed cis
    """

    def __init__(
        self, dim: int, max_height: int, max_width: int, theta_base=10000, device="cuda"
    ):
        super().__init__()
        self.dim = dim
        assert self.dim % 4 == 0, "dim must be divisible by 4"
        self.max_height = max_height
        self.max_width = max_width
        self.theta_base = theta_base
        self.device = device

    def extra_repr(self):
        return f"dim={self.dim}, max_height={self.max_height}, max_width={self.max_width}, theta_base={self.theta_base}"

    @cached_property
    def precomputed_freqs_cis(self) -> torch.Tensor:
        """Calculate the cis(freqs) for each position in the 2D grid.

# ... truncated for brevity ...
```
**EN:** This class defines Rope2 D Pos Emb inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: 2D rotary position embedding with multi-resolution support..
**CN:** 该类定义了 Rope2 D Pos Emb，用于封装该模型组件的状态与方法。 文档字符串摘要：2D rotary position embedding with multi-resolution support.。

### Lines 397-450: Class: MLP2 / 类：MLP2
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
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        assert len(dims) == 3

        self.quant_config = quant_config
        if isinstance(self.quant_config, ModelSlimConfig):
            self.fc0 = ReplicatedLinear(
                dims[0],
                dims[1],
                bias=bias,
                quant_config=quant_config,
                prefix=add_prefix("fc0", prefix),
            )
            self.fc1 = ReplicatedLinear(
                dims[1],
                dims[2],
                bias=bias,
                quant_config=quant_config,
                prefix=add_prefix("fc1", prefix),
            )
        else:
            self.fc0 = nn.Linear(dims[0], dims[1], bias=bias)
            self.fc1 = nn.Linear(dims[1], dims[2], bias=bias)
            for m in [self.fc0, self.fc1]:
                nn.init.trunc_normal_(m.weight, std=math.sqrt(2 / m.in_features))
                if m.bias is not None:
# ... truncated for brevity ...
```
**EN:** This class defines M L P2 inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Args:.
**CN:** 该类定义了 M L P2，用于封装该模型组件的状态与方法。 文档字符串摘要：Args:。

### Lines 453-532: Class: MoonVitEncoderLayer / 类：MoonVitEncoderLayer
```python
class MoonVitEncoderLayer(nn.Module):

    def __init__(
        self,
        num_heads: int,
        hidden_dim: int,
        mlp_dim: int,
        *,
        attn_implementation: str = "flash_attention_2",  # use fa2 in sglang by default
        activation=F.gelu,
        attn_bias: bool = False,
    ):
        super().__init__()
        self.num_heads = num_heads
        self.hidden_dim = hidden_dim
        self.hidden_size_per_attention_head = self.hidden_dim // self.num_heads
        self.attn_implementation = attn_implementation

        self.norm0 = nn.LayerNorm(hidden_dim)
        self.norm1 = nn.LayerNorm(hidden_dim)
        self.mlp = MLP2([hidden_dim, mlp_dim, hidden_dim], activation)
        self.wqkv = nn.Linear(hidden_dim, hidden_dim * 3, bias=attn_bias)
        self.wo = nn.Linear(hidden_dim, hidden_dim, bias=attn_bias)

    def attention_qkvpacked(
        self,
        x: torch.Tensor,
        cu_seqlens: torch.Tensor,
        rope_freqs_cis: Optional[torch.Tensor] = None,
    ):
        """
        Args:
            x (torch.Tensor): (batch_size, seqlen, hidden_dim)
            cu_seqlens (torch.Tensor):
        """
        xqkv = self.wqkv(x)

        qkv_shape = xqkv.size()[:-1] + (
            3,
            self.num_heads,
# ... truncated for brevity ...
```
**EN:** This class defines Moon Vit Encoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Moon Vit Encoder Layer，用于封装该模型组件的状态与方法。

### Lines 535-573: Class: MoonVitEncoder / 类：MoonVitEncoder
```python
class MoonVitEncoder(nn.Module):

    def __init__(
        self,
        hidden_dim: int,
        num_layers: int,
        block_cfg: dict,
    ) -> None:
        super().__init__()

        self.rope_2d = Rope2DPosEmb(
            block_cfg["hidden_dim"] // block_cfg["num_heads"], 512, 512
        )
        self.blocks = nn.ModuleList(
            [MoonVitEncoderLayer(**block_cfg) for _ in range(num_layers)]
        )
        self.final_layernorm = nn.LayerNorm(hidden_dim)

    def forward(
        self, hidden_states: torch.Tensor, grid_hw: torch.Tensor
    ) -> torch.Tensor:
        rope_freqs_cis = self.rope_2d.get_freqs_cis_by_seqlens(grid_hws=grid_hw)

        lengths = torch.cat(
            (
                torch.zeros(1, device=hidden_states.device, dtype=grid_hw.dtype),
                grid_hw[:, 0] * grid_hw[:, 1],
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
**EN:** This class defines Moon Vit Encoder inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Moon Vit Encoder，用于封装该模型组件的状态与方法。

### Lines 576-602: Function: patch_merger() / 函数：patch_merger()
```python
def patch_merger(
    x: torch.Tensor,
    grid_hw: torch.Tensor,
    merge_kernel_size: list[int, int] = (2, 2),
) -> List[torch.Tensor]:
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
**EN:** This function implements patch merger for the surrounding model/runtime logic. Key parameters include x, grid_hw, merge_kernel_size.
**CN:** 该函数实现了 patch merger 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 605-628: Class: MoonVitVLProjector / 类：MoonVitVLProjector
```python
class MoonVitVLProjector(nn.Module):

    def __init__(
        self,
        in_channels: int,
        merge_kernel_size: list[int, int],
        hidden_act: str = "gelu",
        ln_eps: float = 1e-5,
        out_dim: int = 4096,
    ):
        super().__init__()
        self.hidden_size = in_channels * merge_kernel_size[0] * merge_kernel_size[1]

        self.pre_norm = nn.nn.LayerNorm(in_channels, eps=ln_eps)
        self.linear_1 = nn.Linear(self.hidden_size, self.hidden_size, bias=True)
        self.act = ACT2FN[hidden_act]
        self.linear_2 = nn.Linear(self.hidden_size, out_dim, bias=True)

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states = self.pre_norm(hidden_states).view(-1, self.hidden_size)
        hidden_states = self.linear_1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** This class defines Moon Vit V L Projector inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Moon Vit V L Projector，用于封装该模型组件的状态与方法。

### Lines 631-681: Class: MoonVitPretrainedModel / 类：MoonVitPretrainedModel
```python
class MoonVitPretrainedModel(PreTrainedModel):
    config_class = MoonViTConfig
    model_type = "moonvit"
    _no_split_modules = ["PackingTransformer"]
    _supports_flash_attn_2 = True
    _supports_sdpa = True

    def __init__(self, config: MoonViTConfig, *inputs, **kwargs):
        from transformers.activations import GELUTanh

        super().__init__(config, *inputs, **kwargs)
        config = deepcopy(config)
        self.merge_kernel_size = config.merge_kernel_size
        self.patch_size = config.patch_size
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
                "activation": GELUTanh(),
                "attn_bias": True,
                "attn_implementation": config._attn_implementation,
            },
        )

    def forward(
        self, pixel_values: torch.Tensor, grid_hw: torch.Tensor
    ) -> torch.Tensor:
        """
        Args:
            pixel_values (torch.Tensor): The input pixel values.
# ... truncated for brevity ...
```
**EN:** This class defines Moon Vit Pretrained Model inheriting from PreTrainedModel, grouping state and methods for this model component.
**CN:** 该类定义了 Moon Vit Pretrained Model，用于封装该模型组件的状态与方法。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `math`
- `copy: deepcopy`
- `functools: cached_property`
- `typing: List, Optional, Sequence, Tuple, Union`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `transformers.activations: ACT2FN`
- `transformers.modeling_utils: PreTrainedModel`
- `sglang.kernel_api_logging: debug_kernel_api`
- `sglang.srt.configs: MoonViTConfig`
- `sglang.srt.layers.conv: Conv2dLayer`
- `sglang.srt.layers.linear: ReplicatedLinear`
- `sglang.srt.layers.quantization: QuantizationConfig`
- `sglang.srt.layers.quantization.modelslim.modelslim: ModelSlimConfig`
- `sglang.srt.utils: add_prefix`
