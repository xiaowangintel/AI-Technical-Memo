# deepseek_ocr.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/deepseek_ocr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Deepseek Ocr architecture into SGLang's serving runtime. Inference-only Apertus model compatible with HuggingFace weights. / 该模块将 Deepseek Ocr 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only Apertus model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19: Module header and imports / 模块头与导入
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Copyright 2025 The SwissAI Initiative
# Copyright 2023-2024 SGLang Team
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
# ==============================================================================

# Adapted from
# https://github.com/vllm-project/vllm/blob/c7f2cf2b7f67bce5842fedfdba508440fe257375/vllm/model_executor/models/llama.py#L1
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 48-48: Assignment: _is_cpu_amx_available / 赋值：_is_cpu_amx_available
```python
_is_cpu_amx_available = cpu_has_amx_support()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 49-49: Assignment: _is_cpu / 赋值：_is_cpu
```python
_is_cpu = is_cpu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 51-56: Annotated assignment: NestedTensors / 类型赋值：NestedTensors
```python
NestedTensors: TypeAlias = Union[
    list["NestedTensors"],
    list["torch.Tensor"],
    "torch.Tensor",
    tuple["torch.Tensor", ...],
]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 58-58: Annotated assignment: MultiModalEmbeddings / 类型赋值：MultiModalEmbeddings
```python
MultiModalEmbeddings: TypeAlias = list[Tensor] | Tensor | tuple[Tensor, ...]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 60-60: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 63-73: Function: _flatten_embeddings() / 函数：_flatten_embeddings()
```python
def _flatten_embeddings(embeddings: NestedTensors) -> torch.Tensor:
    """
    Recursively flattens and concatenates NestedTensors on all but the last
    dimension.
    """

    if isinstance(embeddings, torch.Tensor):
        # Flatten all but the last dimension.
        return embeddings.flatten(0, -2)

    return torch.cat(tuple(_flatten_embeddings(t) for t in embeddings))
```
**EN:** This function implements flatten embeddings for the surrounding model/runtime logic. Key parameters include embeddings.
**CN:** 该函数实现了 flatten embeddings 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 76-85: Function: _embedding_count_expression() / 函数：_embedding_count_expression()
```python
def _embedding_count_expression(embeddings: NestedTensors) -> str:
    """
    Constructs a debugging representation of the number of embeddings in the
    NestedTensors.
    """

    if isinstance(embeddings, torch.Tensor):
        return " x ".join([str(dim) for dim in embeddings.shape[:-1]])

    return " + ".join(_embedding_count_expression(inner) for inner in embeddings)
```
**EN:** This function implements embedding count expression for the surrounding model/runtime logic. Key parameters include embeddings.
**CN:** 该函数实现了 embedding count expression 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 88-127: Function: _merge_multimodal_embeddings() / 函数：_merge_multimodal_embeddings()
```python
def _merge_multimodal_embeddings(
    inputs_embeds: torch.Tensor,
    multimodal_embeddings: NestedTensors,
    is_multimodal: torch.Tensor,
) -> torch.Tensor:
    """
    Merge `multimodal_embeddings` into `inputs_embeds` by overwriting the
    positions in `inputs_embeds` corresponding to placeholder tokens in
    `input_ids`.

    Note:
        This updates `inputs_embeds` in place.
    """
    if len(multimodal_embeddings) == 0:
        return inputs_embeds

    mm_embeds_flat = _flatten_embeddings(multimodal_embeddings)
    input_dtype = inputs_embeds.dtype

    try:
        # NOTE: This can avoid D2H sync (#22105), but fails to
        # raise an error if is_multimodal.sum() < len(mm_embeds_flat)
        inputs_embeds.masked_scatter_(
            is_multimodal.unsqueeze(-1), mm_embeds_flat.to(dtype=input_dtype)
        )
    except RuntimeError as e:
        num_actual_tokens = len(mm_embeds_flat)
        num_expected_tokens = is_multimodal.sum().item()

        if num_actual_tokens != num_expected_tokens:
            expr = _embedding_count_expression(multimodal_embeddings)

            raise ValueError(
                f"Attempted to assign {expr} = {num_actual_tokens} "
                f"multimodal tokens to {num_expected_tokens} placeholders"
            ) from e

        raise ValueError("Error during masked scatter operation") from e

    return inputs_embeds
```
**EN:** This function implements merge multimodal embeddings for the surrounding model/runtime logic. Key parameters include inputs_embeds, multimodal_embeddings, is_multimodal.
**CN:** 该函数实现了 merge multimodal embeddings 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 130-139: Function: isin_list() / 函数：isin_list()
```python
def isin_list(
    elements: torch.Tensor,
    test_elements_list: list[int],
) -> torch.Tensor:
    use_pin = torch.cuda.is_available() and not getattr(torch.version, "hip", None)
    test_elements = torch.tensor(test_elements_list, pin_memory=use_pin).to(
        device=elements.device, non_blocking=use_pin
    )

    return torch.isin(elements, test_elements)
```
**EN:** This function implements isin list for the surrounding model/runtime logic. Key parameters include elements, test_elements_list.
**CN:** 该函数实现了 isin list 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 142-182: Function: merge_multimodal_embeddings() / 函数：merge_multimodal_embeddings()
```python
def merge_multimodal_embeddings(
    input_ids: torch.Tensor,
    inputs_embeds: torch.Tensor,
    multimodal_embeddings: NestedTensors,
    placeholder_token_id: int | list[int],
) -> torch.Tensor:
    """
    Merge `multimodal_embeddings` into `inputs_embeds` by overwriting the
    positions in `inputs_embeds` corresponding to placeholder tokens in
    `input_ids`.

    `placeholder_token_id` can be a list of token ids (e.g, token ids
    of img_start, img_break, and img_end tokens) when needed: This means
    the order of these tokens in the `input_ids` MUST MATCH the order of
    their embeddings in `multimodal_embeddings` since we need to
    slice-merge instead of individually scattering.

    For example, if input_ids is "TTTTTSIIIBIIIBIIIETTT", where
    - T is text token
    - S is image start token
    - I is image embedding token
    - B is image break token
    - E is image end token.

    Then the image embeddings (that correspond to I's) from vision encoder
    must be padded with embeddings of S, B, and E in the same order of
    input_ids for a correct embedding merge.

    Note:
        This updates `inputs_embeds` in place.
    """
    if isinstance(placeholder_token_id, list):
        is_multimodal = isin_list(input_ids, placeholder_token_id)
    else:
        is_multimodal = input_ids == placeholder_token_id

    return _merge_multimodal_embeddings(
        inputs_embeds,
        multimodal_embeddings=multimodal_embeddings,
        is_multimodal=is_multimodal,
# ... truncated for brevity ...
```
**EN:** This function implements merge multimodal embeddings for the surrounding model/runtime logic. Key parameters include input_ids, inputs_embeds, multimodal_embeddings, placeholder_token_id.
**CN:** 该函数实现了 merge multimodal embeddings 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 185-360: Class: MlpProjector / 类：MlpProjector
```python
class MlpProjector(nn.Module):

    def __init__(
        self,
        projector_type,
        input_dim,
        n_embed,
        depth=1,
        mlp_ratio=1,
        downsample_ratio=4,
    ):
        self.projector_type = projector_type
        self.input_dim = input_dim
        self.n_embed = n_embed
        self.depth = depth
        self.token_pooling = False
        self.conv_fusion_high_low_features = False

        super().__init__()

        if projector_type == "identity":
            modules = nn.Identity()

        elif projector_type == "linear":
            modules = nn.Linear(input_dim, n_embed)

        elif projector_type == "mlp_gelu":
            mlp_depth = depth
            modules = [nn.Linear(input_dim, n_embed)]
            for _ in range(1, mlp_depth):
                modules.append(nn.GELU())
                modules.append(nn.Linear(n_embed, n_embed))
            modules = nn.Sequential(*modules)

        elif projector_type == "normlayer_downsample_mlp_gelu":
            mlp_depth = depth
            mlp_ratio = mlp_ratio
            modules = [
                nn.LayerNorm(input_dim * downsample_ratio * downsample_ratio),
                nn.Linear(
# ... truncated for brevity ...
```
**EN:** This class defines Mlp Projector inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Mlp Projector，用于封装该模型组件的状态与方法。

### Lines 363-375: Class: LayerNorm2d / 类：LayerNorm2d
```python
class LayerNorm2d(nn.Module):
    def __init__(self, num_channels: int, eps: float = 1e-6) -> None:
        super().__init__()
        self.weight = nn.Parameter(torch.ones(num_channels))
        self.bias = nn.Parameter(torch.zeros(num_channels))
        self.eps = eps

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        u = x.mean(1, keepdim=True)
        s = (x - u).pow(2).mean(1, keepdim=True)
        x = (x - u) / torch.sqrt(s + self.eps)
        x = self.weight[:, None, None] * x + self.bias[:, None, None]
        return x
```
**EN:** This class defines Layer Norm2d inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Layer Norm2d，用于封装该模型组件的状态与方法。

### Lines 378-391: Class: MLPBlock / 类：MLPBlock
```python
class MLPBlock(nn.Module):
    def __init__(
        self,
        embedding_dim: int,
        mlp_dim: int,
        act: Type[nn.Module] = nn.GELU,
    ) -> None:
        super().__init__()
        self.lin1 = nn.Linear(embedding_dim, mlp_dim)
        self.lin2 = nn.Linear(mlp_dim, embedding_dim)
        self.act = act()

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.lin2(self.act(self.lin1(x)))
```
**EN:** This class defines M L P Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 M L P Block，用于封装该模型组件的状态与方法。

### Lines 394-427: Function: add_decomposed_rel_pos() / 函数：add_decomposed_rel_pos()
```python
def add_decomposed_rel_pos(
    q: torch.Tensor,
    rel_pos_h: torch.Tensor,
    rel_pos_w: torch.Tensor,
    q_size: Tuple[int, int],
    k_size: Tuple[int, int],
) -> torch.Tensor:
    """
    Calculate decomposed Relative Positional Embeddings from :paper:`mvitv2`.
    https://github.com/facebookresearch/mvit/blob/19786631e330df9f3622e5402b4a419a263a2c80/mvit/models/attention.py   # noqa B950
    Args:
        q (Tensor): query q in the attention layer with shape (B, q_h * q_w, C).
        rel_pos_h (Tensor): relative position embeddings (Lh, C) for height axis.
        rel_pos_w (Tensor): relative position embeddings (Lw, C) for width axis.
        q_size (Tuple): spatial sequence size of query q with (q_h, q_w).
        k_size (Tuple): spatial sequence size of key k with (k_h, k_w).
    Returns:
        attn (Tensor): attention map with added relative positional embeddings.
    """
    q_h, q_w = q_size
    k_h, k_w = k_size
    Rh = get_rel_pos(q_h, k_h, rel_pos_h)
    Rw = get_rel_pos(q_w, k_w, rel_pos_w)

    B, _, dim = q.shape
    r_q = q.reshape(B, q_h, q_w, dim)
    rel_h = torch.einsum("bhwc,hkc->bhwk", r_q, Rh)
    rel_w = torch.einsum("bhwc,wkc->bhwk", r_q, Rw)
    rel_h = rel_h.unsqueeze(-1)
    rel_w = rel_w.unsqueeze(-2)
    rel_h = rel_h.reshape(B, q_h * q_w, k_h, 1)
    rel_w = rel_w.reshape(B, q_h * q_w, 1, k_w)

    return rel_h, rel_w
```
**EN:** This function implements add decomposed rel pos for the surrounding model/runtime logic. Key parameters include q, rel_pos_h, rel_pos_w, q_size.
**CN:** 该函数实现了 add decomposed rel pos 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 430-512: Class: Attention / 类：Attention
```python
class Attention(nn.Module):
    """Multi-head Attention block with relative position embeddings."""

    def __init__(
        self,
        dim: int,
        num_heads: int = 8,
        qkv_bias: bool = True,
        use_rel_pos: bool = False,
        rel_pos_zero_init: bool = True,
        input_size: Optional[Tuple[int, int]] = None,
    ) -> None:
        """
        Args:
            dim (int): Number of input channels.
            num_heads (int): Number of attention heads.
            qkv_bias (bool):  If True, add a learnable bias to query, key, value.
            rel_pos_zero_init (bool): If True, zero initialize relative positional parameters.
            input_size (tuple(int, int) or None): Input resolution for calculating the relative
                positional parameter size.
        """
        super().__init__()
        self.num_heads = num_heads
        head_dim = dim // num_heads
        self.scale = head_dim**-0.5

        self.qkv = nn.Linear(dim, dim * 3, bias=qkv_bias)
        self.proj = nn.Linear(dim, dim)

        self.use_rel_pos = use_rel_pos
        if self.use_rel_pos:
            assert (
                input_size is not None
            ), "Input size must be provided if using relative positional encoding."
            # initialize relative positional embeddings
            self.rel_pos_h = nn.Parameter(torch.zeros(2 * input_size[0] - 1, head_dim))
            self.rel_pos_w = nn.Parameter(torch.zeros(2 * input_size[1] - 1, head_dim))

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        B, H, W, _ = x.shape
# ... truncated for brevity ...
```
**EN:** This class defines Attention inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Multi-head Attention block with relative position embeddings..
**CN:** 该类定义了 Attention，用于封装该模型组件的状态与方法。 文档字符串摘要：Multi-head Attention block with relative position embeddings.。

### Lines 515-539: Function: window_partition() / 函数：window_partition()
```python
def window_partition(
    x: torch.Tensor, window_size: int
) -> Tuple[torch.Tensor, Tuple[int, int]]:
    """
    Partition into non-overlapping windows with padding if needed.
    Args:
        x (tensor): input tokens with [B, H, W, C].
        window_size (int): window size.
    Returns:
        windows: windows after partition with [B * num_windows, window_size, window_size, C].
        (Hp, Wp): padded height and width before partition
    """
    B, H, W, C = x.shape

    pad_h = (window_size - H % window_size) % window_size
    pad_w = (window_size - W % window_size) % window_size
    if pad_h > 0 or pad_w > 0:
        x = F.pad(x, (0, 0, 0, pad_w, 0, pad_h))
    Hp, Wp = H + pad_h, W + pad_w

    x = x.view(B, Hp // window_size, window_size, Wp // window_size, window_size, C)
    windows = (
        x.permute(0, 1, 3, 2, 4, 5).contiguous().view(-1, window_size, window_size, C)
    )
    return windows, (Hp, Wp)
```
**EN:** This function implements window partition for the surrounding model/runtime logic. Key parameters include x, window_size.
**CN:** 该函数实现了 window partition 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 542-568: Function: window_unpartition() / 函数：window_unpartition()
```python
def window_unpartition(
    windows: torch.Tensor,
    window_size: int,
    pad_hw: Tuple[int, int],
    hw: Tuple[int, int],
) -> torch.Tensor:
    """
    Window unpartition into original sequences and removing padding.
    Args:
        windows (tensor): input tokens with [B * num_windows, window_size, window_size, C].
        window_size (int): window size.
        pad_hw (Tuple): padded height and width (Hp, Wp).
        hw (Tuple): original height and width (H, W) before padding.
    Returns:
        x: unpartitioned sequences with [B, H, W, C].
    """
    Hp, Wp = pad_hw
    H, W = hw
    B = windows.shape[0] // (Hp * Wp // window_size // window_size)
    x = windows.view(
        B, Hp // window_size, Wp // window_size, window_size, window_size, -1
    )
    x = x.permute(0, 1, 3, 2, 4, 5).contiguous().view(B, Hp, Wp, -1)

    if Hp > H or Wp > W:
        x = x[:, :H, :W, :].contiguous()
    return x
```
**EN:** This function implements window unpartition for the surrounding model/runtime logic. Key parameters include windows, window_size, pad_hw, hw.
**CN:** 该函数实现了 window unpartition 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 571-636: Class: Block / 类：Block
```python
class Block(nn.Module):
    """Transformer blocks with support of window attention and residual propagation blocks"""

    def __init__(
        self,
        dim: int,
        num_heads: int,
        mlp_ratio: float = 4.0,
        qkv_bias: bool = True,
        norm_layer: Type[nn.Module] = nn.LayerNorm,
        act_layer: Type[nn.Module] = nn.GELU,
        use_rel_pos: bool = False,
        rel_pos_zero_init: bool = True,
        window_size: int = 0,
        input_size: Optional[Tuple[int, int]] = None,
    ) -> None:
        """
        Args:
            dim (int): Number of input channels.
            num_heads (int): Number of attention heads in each ViT block.
            mlp_ratio (float): Ratio of mlp hidden dim to embedding dim.
            qkv_bias (bool): If True, add a learnable bias to query, key, value.
            norm_layer (nn.Module): Normalization layer.
            act_layer (nn.Module): Activation layer.
            use_rel_pos (bool): If True, add relative positional embeddings to the attention map.
            rel_pos_zero_init (bool): If True, zero initialize relative positional parameters.
            window_size (int): Window size for window attention blocks. If it equals 0, then
                use global attention.
            input_size (tuple(int, int) or None): Input resolution for calculating the relative
                positional parameter size.
        """
        super().__init__()
        self.norm1 = norm_layer(dim)
        self.attn = Attention(
            dim,
            num_heads=num_heads,
            qkv_bias=qkv_bias,
            use_rel_pos=use_rel_pos,
            rel_pos_zero_init=rel_pos_zero_init,
            input_size=input_size if window_size == 0 else (window_size, window_size),
# ... truncated for brevity ...
```
**EN:** This class defines Block inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Transformer blocks with support of window attention and residual propagation blocks.
**CN:** 该类定义了 Block，用于封装该模型组件的状态与方法。 文档字符串摘要：Transformer blocks with support of window attention and residual propagation blocks。

### Lines 639-670: Class: PatchEmbed / 类：PatchEmbed
```python
class PatchEmbed(nn.Module):
    """
    Image to Patch Embedding.
    """

    def __init__(
        self,
        kernel_size: Tuple[int, int] = (16, 16),
        stride: Tuple[int, int] = (16, 16),
        padding: Tuple[int, int] = (0, 0),
        in_chans: int = 3,
        embed_dim: int = 768,
    ) -> None:
        """
        Args:
            kernel_size (Tuple): kernel size of the projection layer.
            stride (Tuple): stride of the projection layer.
            padding (Tuple): padding size of the projection layer.
            in_chans (int): Number of input image channels.
            embed_dim (int): Patch embedding dimension.
        """
        super().__init__()

        self.proj = nn.Conv2d(
            in_chans, embed_dim, kernel_size=kernel_size, stride=stride, padding=padding
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = self.proj(x)
        # B C H W -> B H W C
        x = x.permute(0, 2, 3, 1)
        return x
```
**EN:** This class defines Patch Embed inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Image to Patch Embedding..
**CN:** 该类定义了 Patch Embed，用于封装该模型组件的状态与方法。 文档字符串摘要：Image to Patch Embedding.。

### Lines 673-691: Function: get_abs_pos_sam() / 函数：get_abs_pos_sam()
```python
def get_abs_pos_sam(abs_pos, tgt_size):
    dtype = abs_pos.dtype

    src_size = abs_pos.size(1)

    if src_size != tgt_size:
        old_pos_embed = abs_pos.permute(0, 3, 1, 2)
        old_pos_embed = old_pos_embed.to(torch.float32)
        new_pos_embed = F.interpolate(
            old_pos_embed,
            size=(tgt_size, tgt_size),
            mode="bicubic",
            antialias=True,
            align_corners=False,
        ).to(dtype)
        new_pos_embed = new_pos_embed.permute(0, 2, 3, 1)
        return new_pos_embed
    else:
        return abs_pos
```
**EN:** This function implements get abs pos sam for the surrounding model/runtime logic. Key parameters include abs_pos, tgt_size.
**CN:** 该函数实现了 get abs pos sam 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 695-804: Class: ImageEncoderViT / 类：ImageEncoderViT
```python
class ImageEncoderViT(nn.Module):
    def __init__(
        self,
        img_size: int = 1024,
        patch_size: int = 16,
        in_chans: int = 3,
        embed_dim: int = 768,
        depth: int = 12,
        num_heads: int = 12,
        mlp_ratio: float = 4.0,
        out_chans: int = 256,
        qkv_bias: bool = True,
        norm_layer: Type[nn.Module] = nn.LayerNorm,
        act_layer: Type[nn.Module] = nn.GELU,
        use_abs_pos: bool = True,
        use_rel_pos: bool = False,
        rel_pos_zero_init: bool = True,
        window_size: int = 0,
        global_attn_indexes: Tuple[int, ...] = (),
        net_3_out_channels: int = 1024,
    ) -> None:
        """
        Args:
            img_size (int): Input image size.
            patch_size (int): Patch size.
            in_chans (int): Number of input image channels.
            embed_dim (int): Patch embedding dimension.
            depth (int): Depth of ViT.
            num_heads (int): Number of attention heads in each ViT block.
            mlp_ratio (float): Ratio of mlp hidden dim to embedding dim.
            qkv_bias (bool): If True, add a learnable bias to query, key, value.
            norm_layer (nn.Module): Normalization layer.
            act_layer (nn.Module): Activation layer.
            use_abs_pos (bool): If True, use absolute positional embeddings.
            use_rel_pos (bool): If True, add relative positional embeddings to the attention map.
            rel_pos_zero_init (bool): If True, zero initialize relative positional parameters.
            window_size (int): Window size for window attention blocks.
            global_attn_indexes (list): Indexes for blocks using global attention.
        """
        super().__init__()
# ... truncated for brevity ...
```
**EN:** This class defines Image Encoder Vi T inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Image Encoder Vi T，用于封装该模型组件的状态与方法。

### Lines 807-840: Function: _build_sam() / 函数：_build_sam()
```python
def _build_sam(
    encoder_embed_dim,
    encoder_depth,
    encoder_num_heads,
    encoder_global_attn_indexes,
    checkpoint=None,
    net_3_out_channels: int = 1024,
):
    prompt_embed_dim = 256
    image_size = 1024
    vit_patch_size = 16
    image_encoder = ImageEncoderViT(
        depth=encoder_depth,
        embed_dim=encoder_embed_dim,
        img_size=image_size,
        mlp_ratio=4,
        norm_layer=partial(torch.nn.LayerNorm, eps=1e-6),
        num_heads=encoder_num_heads,
        patch_size=vit_patch_size,
        qkv_bias=True,
        use_rel_pos=True,
        global_attn_indexes=encoder_global_attn_indexes,
        window_size=14,
        out_chans=prompt_embed_dim,
        net_3_out_channels=net_3_out_channels,
    )
    image_encoder.eval()
    if checkpoint is not None:
        state_dict = torch.load(checkpoint)
        image_encoder.load_state_dict(
            {k[30:]: v for k, v in state_dict.items() if "vision_tower_high" in k},
            strict=True,
        )
    return image_encoder
```
**EN:** This function implements build sam for the surrounding model/runtime logic. Key parameters include encoder_embed_dim, encoder_depth, encoder_num_heads, encoder_global_attn_indexes.
**CN:** 该函数实现了 build sam 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 843-851: Function: build_sam_vit_b() / 函数：build_sam_vit_b()
```python
def build_sam_vit_b(checkpoint=None, net_3_out_channels: int = 1024):
    return _build_sam(
        encoder_embed_dim=768,
        encoder_depth=12,
        encoder_num_heads=12,
        encoder_global_attn_indexes=[2, 5, 8, 11],
        checkpoint=checkpoint,
        net_3_out_channels=net_3_out_channels,
    )
```
**EN:** This function implements build sam vit b for the surrounding model/runtime logic. Key parameters include checkpoint, net_3_out_channels.
**CN:** 该函数实现了 build sam vit b 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 854-886: Function: get_abs_pos() / 函数：get_abs_pos()
```python
def get_abs_pos(abs_pos, tgt_size):
    # abs_pos: L, C
    # tgt_size: M
    # return: M, C
    dim = abs_pos.size(-1)
    abs_pos_new = abs_pos.squeeze(0)
    cls_token, old_pos_embed = abs_pos_new[:1], abs_pos_new[1:]

    src_size = int(math.sqrt(abs_pos_new.shape[0] - 1))
    tgt_size = int(math.sqrt(tgt_size))
    dtype = abs_pos.dtype

    if src_size != tgt_size:
        old_pos_embed = (
            old_pos_embed.view(1, src_size, src_size, dim)
            .permute(0, 3, 1, 2)
            .contiguous()
        )
        old_pos_embed = old_pos_embed.to(torch.float32)
        new_pos_embed = F.interpolate(
            old_pos_embed,
            size=(tgt_size, tgt_size),
            mode="bicubic",
            antialias=True,
            align_corners=False,
        ).to(dtype)
        new_pos_embed = new_pos_embed.permute(0, 2, 3, 1)
        new_pos_embed = new_pos_embed.view(tgt_size * tgt_size, dim)
        vision_pos_embed = torch.cat([cls_token, new_pos_embed], dim=0)
        vision_pos_embed = vision_pos_embed.view(1, tgt_size * tgt_size + 1, dim)
        return vision_pos_embed
    else:
        return abs_pos
```
**EN:** This function implements get abs pos for the surrounding model/runtime logic. Key parameters include abs_pos, tgt_size.
**CN:** 该函数实现了 get abs pos 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 889-929: Class: CLIPVisionEmbeddings / 类：CLIPVisionEmbeddings
```python
class CLIPVisionEmbeddings(nn.Module):
    def __init__(self, hidden_size=1024, image_size=224, patch_size=14, num_channels=3):
        super().__init__()
        self.embed_dim = hidden_size
        self.image_size = image_size
        self.patch_size = patch_size

        self.class_embedding = torch.nn.Parameter(torch.randn(self.embed_dim))

        self.patch_embedding = torch.nn.Conv2d(
            in_channels=num_channels,
            out_channels=self.embed_dim,
            kernel_size=self.patch_size,
            stride=self.patch_size,
            bias=False,
        )

        self.num_patches = (self.image_size // self.patch_size) ** 2
        self.num_positions = self.num_patches + 1
        self.position_embedding = torch.nn.Embedding(self.num_positions, self.embed_dim)
        self.register_buffer(
            "position_ids", torch.arange(self.num_positions).expand((1, -1))
        )

    def forward(self, pixel_values, patch_embeds):
        batch_size = pixel_values.shape[0]

        if patch_embeds is not None:
            patch_embeds = patch_embeds
        else:
            patch_embeds = self.patch_embedding(pixel_values)

        patch_embeds = patch_embeds.flatten(2).transpose(1, 2)

        class_embeds = self.class_embedding.expand(batch_size, 1, -1)
        embeddings = torch.cat([class_embeds, patch_embeds], dim=1)

        embeddings = embeddings + get_abs_pos(
            self.position_embedding(self.position_ids), embeddings.size(1)
        )
# ... truncated for brevity ...
```
**EN:** This class defines C L I P Vision Embeddings inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 C L I P Vision Embeddings，用于封装该模型组件的状态与方法。

### Lines 932-990: Class: NoTPAttention / 类：NoTPAttention
```python
class NoTPAttention(torch.nn.Module):
    def __init__(self, cfg):
        super().__init__()
        self.num_heads = cfg["num_attention_heads"]
        self.n_local_heads = cfg["num_attention_heads"]
        self.head_dim = cfg["hidden_size"] // cfg["num_attention_heads"]
        self.max_seq_len = cfg["seq_length"]
        self.use_flash_attention = cfg["use_flash_attn"]

        self.qkv_proj = torch.nn.Linear(
            cfg["hidden_size"], cfg["hidden_size"] * 3, bias=True
        )
        self.out_proj = torch.nn.Linear(
            cfg["hidden_size"], cfg["hidden_size"], bias=True
        )

        # self.core_attention = CoreAttention(cfg, AttnType.self_attn)

        self.attn_drop = cfg["attention_dropout"]

    def forward(
        self,
        x: torch.Tensor,
    ):
        bsz, seqlen, _ = x.shape
        xqkv = self.qkv_proj(x)
        xqkv = xqkv.view(bsz, seqlen, 3, self.num_heads, self.head_dim)

        if self.use_flash_attention:

            xq, xk, xv = torch.split(xqkv, 1, dim=2)
            xq = xq.squeeze(2)
            xk = xk.squeeze(2)
            xv = xv.squeeze(2)
            # xq, xk, xv = xqkv[:, :, 0, ...], xqkv[:, :, 1, ...], xqkv[:, :, 2, ...]

            # （B, num_head, S, head_size)
            xq = xq.permute(0, 2, 1, 3)
            xk = xk.permute(0, 2, 1, 3)
            xv = xv.permute(0, 2, 1, 3)
# ... truncated for brevity ...
```
**EN:** This class defines No T P Attention inheriting from torch.nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 No T P Attention，用于封装该模型组件的状态与方法。

### Lines 994-995: Function: quick_gelu() / 函数：quick_gelu()
```python
def quick_gelu(x):
    return x * torch.sigmoid(1.702 * x)
```
**EN:** This function implements quick gelu for the surrounding model/runtime logic. Key parameters include x.
**CN:** 该函数实现了 quick gelu 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 998-1012: Class: NoTPFeedForward / 类：NoTPFeedForward
```python
class NoTPFeedForward(nn.Module):
    def __init__(
        self,
        cfg,
        dim: int,
        hidden_dim: int,
    ):
        super().__init__()

        self.fc1 = torch.nn.Linear(dim, hidden_dim, bias=True)
        self.fc2 = torch.nn.Linear(hidden_dim, dim, bias=True)

    def forward(self, x):
        output = self.fc2(quick_gelu(self.fc1(x)))
        return output
```
**EN:** This class defines No T P Feed Forward inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 No T P Feed Forward，用于封装该模型组件的状态与方法。

### Lines 1015-1021: Class: LayerNormfp32 / 类：LayerNormfp32
```python
class LayerNormfp32(torch.nn.LayerNorm):
    """Subclass torch's LayerNorm to handle fp16."""

    def forward(self, x: torch.Tensor):
        orig_type = x.dtype
        ret = super().forward(x.type(torch.float32))
        return ret.type(orig_type)
```
**EN:** This class defines Layer Normfp32 inheriting from torch.nn.LayerNorm, grouping state and methods for this model component. Docstring summary: Subclass torch's LayerNorm to handle fp16..
**CN:** 该类定义了 Layer Normfp32，用于封装该模型组件的状态与方法。 文档字符串摘要：Subclass torch's LayerNorm to handle fp16.。

### Lines 1024-1047: Class: NoTPTransformerBlock / 类：NoTPTransformerBlock
```python
class NoTPTransformerBlock(nn.Module):
    def __init__(self, cfg, layer_id: int, multiple_of=256):
        super().__init__()

        self.n_heads = cfg["num_attention_heads"]
        self.dim = cfg["hidden_size"]
        self.head_dim = cfg["hidden_size"] // cfg["num_attention_heads"]
        self.self_attn = NoTPAttention(cfg)
        self.mlp = NoTPFeedForward(
            cfg, dim=cfg["hidden_size"], hidden_dim=cfg["ffn_hidden_size"]
        )
        self.layer_id = layer_id
        self.layer_norm1 = torch.nn.LayerNorm(
            cfg["hidden_size"], eps=cfg["layernorm_epsilon"]
        )
        self.layer_norm2 = torch.nn.LayerNorm(
            cfg["hidden_size"], eps=cfg["layernorm_epsilon"]
        )

    def forward(self, x: torch.Tensor):
        residual = self.self_attn.forward(self.layer_norm1(x))
        h = x + residual
        out = h + self.mlp.forward(self.layer_norm2(h))
        return out
```
**EN:** This class defines No T P Transformer Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 No T P Transformer Block，用于封装该模型组件的状态与方法。

### Lines 1050-1074: Class: NoTPTransformer / 类：NoTPTransformer
```python
class NoTPTransformer(nn.Module):
    def __init__(self, cfg):
        super().__init__()

        self.cfg = cfg
        self.num_layers = cfg["num_layers"]

        self.layers = torch.nn.ModuleList()
        for layer_id in range(self.num_layers):
            self.layers.append(
                NoTPTransformerBlock(
                    cfg,
                    layer_id + 1,
                )
            )

    def forward(
        self,
        hidden_states,
    ):

        for layer in self.layers:
            hidden_states = layer(hidden_states)

        return hidden_states
```
**EN:** This class defines No T P Transformer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 No T P Transformer，用于封装该模型组件的状态与方法。

### Lines 1077-1130: Class: VitModel / 类：VitModel
```python
class VitModel(nn.Module):
    def __init__(self, cfg, freeze_embed=False, freeze_pre_norm=False) -> None:
        super().__init__()

        self.embeddings = CLIPVisionEmbeddings(
            hidden_size=cfg["hidden_size"],
            image_size=cfg["image_size"],
            patch_size=cfg["patch_size"],
        )

        if freeze_embed:
            for _, param in self.embeddings.named_parameters():
                param.requires_grad = False

        self.transformer = NoTPTransformer(cfg=cfg)

        if cfg.get("fp32norm", False):
            logger.info("Load fp32 layernorm for ViT.")
            self.pre_layrnorm = LayerNormfp32(
                cfg["hidden_size"],
                eps=cfg.get("pre_layernorm_epsilon", 1e-5),
            )
        else:
            self.pre_layrnorm = torch.nn.LayerNorm(
                cfg["hidden_size"],
                eps=cfg.get("pre_layernorm_epsilon", 1e-5),
            )

        if freeze_pre_norm:
            for _, param in self.pre_layrnorm.named_parameters():
                param.requires_grad = False

        for p in self.parameters():
            p.micro_dp = True

    @property
    def dtype(self):
        return next(self.parameters()).dtype

    def set_input_tensor(self, input_tensor):
# ... truncated for brevity ...
```
**EN:** This class defines Vit Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Vit Model，用于封装该模型组件的状态与方法。

### Lines 1133-1151: Assignment: vit_model_cfg / 赋值：vit_model_cfg
```python
vit_model_cfg = dict(
    num_layers=24,
    hidden_size=1024,
    num_heads=16,
    num_attention_heads=16,
    ffn_hidden_size=4096,
    seq_length=256,
    max_position_embeddings=256,
    use_flash_attn=False,
    understand_projector_stride=2,
    hidden_dropout=0.0,
    attention_dropout=0.0,
    no_persist_layer_norm=False,
    layernorm_epsilon=1e-5,
    pre_layernorm_epsilon=1e-5,
    image_size=224,
    patch_size=14,
    recompute_list=[],
)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 1154-1159: Function: build_clip_l() / 函数：build_clip_l()
```python
def build_clip_l():
    return VitModel(
        cfg=vit_model_cfg,
        freeze_embed=False,
        freeze_pre_norm=False,
    )
```
**EN:** This function implements build clip l for the surrounding model/runtime logic.
**CN:** 该函数实现了 build clip l 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 1162-1332: Class: CustomQwen2Decoder / 类：CustomQwen2Decoder
```python
class CustomQwen2Decoder(nn.Module):
    """Qwen2 decoder with mixed causal masking for OCR2 vision encoder."""

    def __init__(
        self,
        decoder_layer: int = 24,
        max_position_embeddings: int = 131072,
        hidden_dimension: int = 896,
        num_attention_heads: int = 14,
        num_key_value_heads: int = 2,
        intermediate_size: int = 4864,
        vocab_size: int = 151936,
        attn_implementation: str = "sdpa",
        rms_norm_eps: float = 1e-6,
        rope_theta: float = 1000000.0,
        attention_dropout: float = 0.0,
        hidden_act: str = "silu",
        initializer_range: float = 0.02,
    ):
        super().__init__()
        if attn_implementation == "flash_attention_2":
            raise ValueError(
                "CustomQwen2Decoder does not support flash_attention_2; "
                "use sdpa or eager."
            )

        Qwen2Model = getattr(transformers.models.qwen2.modeling_qwen2, "Qwen2Model")
        Qwen2Config = getattr(transformers, "Qwen2Config")

        config = Qwen2Config(
            hidden_size=hidden_dimension,
            num_hidden_layers=decoder_layer,
            num_attention_heads=num_attention_heads,
            num_key_value_heads=num_key_value_heads,
            intermediate_size=intermediate_size,
            max_position_embeddings=max_position_embeddings,
            vocab_size=vocab_size,
            rms_norm_eps=rms_norm_eps,
            rope_theta=rope_theta,
            attention_dropout=attention_dropout,
# ... truncated for brevity ...
```
**EN:** This class defines Custom Qwen2 Decoder inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Qwen2 decoder with mixed causal masking for OCR2 vision encoder..
**CN:** 该类定义了 Custom Qwen2 Decoder，用于封装该模型组件的状态与方法。 文档字符串摘要：Qwen2 decoder with mixed causal masking for OCR2 vision encoder.。

### Lines 1335-1396: Class: Qwen2Decoder2Encoder / 类：Qwen2Decoder2Encoder
```python
class Qwen2Decoder2Encoder(nn.Module):
    """Decoder-as-encoder for OCR2 vision tokens."""

    def __init__(
        self,
        decoder_layer: int,
        hidden_dimension: int,
        num_attention_heads: int,
        num_key_value_heads: int,
        intermediate_size: int,
        max_query: int,
    ):
        super().__init__()
        self.model = CustomQwen2Decoder(
            decoder_layer=decoder_layer,
            hidden_dimension=hidden_dimension,
            num_attention_heads=num_attention_heads,
            num_key_value_heads=num_key_value_heads,
            intermediate_size=intermediate_size,
            attn_implementation="sdpa",
        )

        self.query_768 = nn.Embedding(144, hidden_dimension)
        self.query_1024 = nn.Embedding(256, hidden_dimension)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = x.flatten(2).transpose(1, 2)
        bs, n_query, _ = x.shape

        if n_query == 144:
            param_img = self.query_768.weight
        elif n_query == 256:
            param_img = self.query_1024.weight
        else:
            base = (
                self.query_1024.weight
                if n_query > self.query_768.num_embeddings
                else self.query_768.weight
            )
            param_img = (
# ... truncated for brevity ...
```
**EN:** This class defines Qwen2 Decoder2 Encoder inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Decoder-as-encoder for OCR2 vision tokens..
**CN:** 该类定义了 Qwen2 Decoder2 Encoder，用于封装该模型组件的状态与方法。 文档字符串摘要：Decoder-as-encoder for OCR2 vision tokens.。

### Lines 1399-1419: Function: build_qwen2_decoder_as_encoder() / 函数：build_qwen2_decoder_as_encoder()
```python
def build_qwen2_decoder_as_encoder(
    decoder_layer: int = 24,
    hidden_dimension: int = 896,
    num_attention_heads: int = 14,
    num_key_value_heads: int = 2,
    intermediate_size: int = 4864,
    max_query: int = 400,
    checkpoint=None,
):
    decoder_as_encoder = Qwen2Decoder2Encoder(
        decoder_layer=decoder_layer,
        hidden_dimension=hidden_dimension,
        num_attention_heads=num_attention_heads,
        num_key_value_heads=num_key_value_heads,
        intermediate_size=intermediate_size,
        max_query=max_query,
    )
    if checkpoint is not None:
        state_dict = torch.load(checkpoint)
        decoder_as_encoder.load_state_dict(state_dict, strict=True)
    return decoder_as_encoder
```
**EN:** This function implements build qwen2 decoder as encoder for the surrounding model/runtime logic. Key parameters include decoder_layer, hidden_dimension, num_attention_heads, num_key_value_heads.
**CN:** 该函数实现了 build qwen2 decoder as encoder 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 1422-1889: Class: DeepseekOCRForCausalLM / 类：DeepseekOCRForCausalLM
```python
class DeepseekOCRForCausalLM(nn.Module):
    def __init__(
        self,
        *,
        config: DeepseekVLV2Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.config = config

        self.vision_config = config.vision_config
        self.projector_config = config.projector_config
        self.text_config = config.text_config
        self.is_ocr2 = (
            str(getattr(self.vision_config, "model_name", "")).lower()
            == "deepencoderv2"
            or getattr(self.projector_config, "input_dim", None) == 896
        )
        n_embed = getattr(self.projector_config, "n_embed", 1280)

        self.tile_tag = config.tile_tag
        self.global_view_pos = config.global_view_pos

        # special token for image token sequence format
        embed_std = 1 / torch.sqrt(torch.tensor(n_embed, dtype=torch.float32))
        if self.tile_tag == "2D":
            # <|view_separator|>, <|\n|>
            self.view_seperator = nn.Parameter(torch.randn(n_embed) * embed_std)
            if not self.is_ocr2:
                self.image_newline = nn.Parameter(torch.randn(n_embed) * embed_std)
        else:
            raise ValueError(
                f"Only 2D tile_tag is supported currently, got: {self.tile_tag}"
            )

        if not self.is_ocr2:
            if self.text_config.topk_method == "noaux_tc":
                self.model = DeepseekV3ForCausalLM(
# ... truncated for brevity ...
```
**EN:** This class defines Deepseek O C R For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek O C R For Causal L M，用于封装该模型组件的状态与方法。

### Lines 1892-1892: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [DeepseekOCRForCausalLM]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `copy`
- `logging`
- `math`
- `functools: partial`
- `typing: Iterable, List, Optional, Set, Tuple, Type, TypeAlias, Union`
- `torch`
- `torch.nn.functional`
- `transformers`
- `torch: Tensor, nn`
- `transformers.models.vitdet.modeling_vitdet: get_rel_pos`
- `sglang.srt.configs.deepseek_ocr: DeepseekVLV2Config`
- `sglang.srt.layers.quantization: QuantizationConfig`
- `sglang.srt.managers.mm_utils: MultiModalityDataPaddingPatternMultimodalTokens, general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch: MultimodalDataItem, MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.models.deepseek: DeepseekForCausalLM`
- `sglang.srt.models.deepseek_v2: DeepseekV2ForCausalLM, DeepseekV3ForCausalLM`
- `sglang.srt.models.transformers: maybe_prefix`
- `sglang.srt.utils: cpu_has_amx_support, is_cpu`
