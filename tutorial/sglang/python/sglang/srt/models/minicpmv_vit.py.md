# minicpmv_vit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/minicpmv_vit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Vision Transformer for MiniCPM-V 4.6. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 8-24: Module docstring
```python
"""Vision Transformer for MiniCPM-V 4.6.

Compared to 4.5 (Idefics2VisionTransformer end-to-end + Perceiver-style
Resampler4_5), 4.6 compresses visual tokens *twice*:

    patchify -> [layer 0 .. insert_layer_id]     full-res tokens
             -> ViTWindowAttentionMerger         2x2 window attn + 2x2 fold
             -> [layer insert_layer_id+1 .. N-1] compressed tokens
             -> post_layernorm
             -> Merger (merger_times x DownsampleMLP, project to LLM dim)

With defaults (insert_layer_id=6, merger_times=1) the combined compression
is 16x. ``downsample_mode="4x"`` skips the mid-ViT merger.

Class structure mirrors the HF ref one-to-one to make weight loading and
upstream tracking easy.
"""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 26-42: Module imports
```python
from typing import List, Optional, Tuple

import torch
import torch.nn.functional as F
from torch import nn
from transformers import PretrainedConfig

from sglang.srt.layers.activation import get_act_fn
from sglang.srt.layers.attention.vision import VisionAttention
from sglang.srt.layers.linear import ColumnParallelLinear, RowParallelLinear
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.models.idefics2 import (
    Idefics2Encoder,
    Idefics2EncoderLayer,
    Idefics2VisionEmbeddings,
)
from sglang.srt.utils import add_prefix, is_npu
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 45-58: Class `MiniCPMV_ViTWindowAttentionMerger` overview
```python
class MiniCPMV_ViTWindowAttentionMerger(nn.Module):
    """Mid-ViT 2x2 window attention + 2x2 fold.

    Stage 1: reorder tokens so each 2x2 spatial window becomes 4 contiguous
    tokens; run packed self-attention with one window per cu_seqlens segment;
    un-reorder; add residual. (No length reduction yet.)

    Stage 2: fold each 2x2 window into a single token by concatenating the
    four hidden vectors along channel; pass through ``hidden*4 ->
    intermediate*4 -> hidden`` MLP; add the mean of the four window vectors
    as residual. ``target_sizes`` halves on each axis; ``cu_seqlens`` /
    ``max_seqlens`` are rebuilt for the compressed grid.
    """
```
**EN:** Defines `MiniCPMV_ViTWindowAttentionMerger` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV_ViTWindowAttentionMerger`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 59-107: Method `MiniCPMV_ViTWindowAttentionMerger.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.window_kernel_size = (2, 2)
        self.embed_dim = config.hidden_size

        # The "FFN" here is the linear_1/linear_2 pair applied after the 2x2
        # fold below (it operates on hidden*4 -> intermediate*4 -> hidden).
        # ``flatten_batch=True``: input is one packed sequence
        # ``(1, sum_windows * window_area, D)`` with cu_seqlens demarcating
        # per-window segments. The outer encoder layers use ``False`` because
        # there each batch row is one image padded to max_patches.
        self.self_attn = VisionAttention(
            embed_dim=config.hidden_size,
            num_heads=config.num_attention_heads,
            projection_size=config.hidden_size,
            use_qkv_parallel=True,
            quant_config=quant_config,
            dropout=config.attention_dropout,
            softmax_in_single_precision=True,
            flatten_batch=True,
            prefix=add_prefix("self_attn", prefix),
        )
        self.layer_norm1 = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_eps)

        window_area = self.window_kernel_size[0] * self.window_kernel_size[1]
        hidden_4x = self.embed_dim * window_area
        inter_4x = config.intermediate_size * window_area

        self.pre_norm = nn.LayerNorm(hidden_4x, eps=config.layer_norm_eps)
        self.linear_1 = ColumnParallelLinear(
            hidden_4x,
            inter_4x,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("linear_1", prefix),
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 109-151: Method `MiniCPMV_ViTWindowAttentionMerger.get_window_index`
```python
    def get_window_index(
        self, target_sizes: torch.Tensor
    ) -> Tuple[torch.Tensor, torch.Tensor, int]:
        """Return ``(permutation, per-window cu_seqlens, max_seqlens=4)``.

        Kept on CPU because mixing device-bound offsets with CPU arange trips
        strict dtype checks in PyTorch 2.10+.
        """
        window_h, window_w = self.window_kernel_size
        max_seqlens = window_h * window_w  # 4

        window_index_list: List[torch.Tensor] = []
        cu_seqlens: List[int] = [0]
        token_offset = 0

        for height, width in target_sizes:
            height, width = int(height), int(width)
            if height % window_h != 0 or width % window_w != 0:
                raise ValueError(
                    f"height={height}, width={width} must be divisible by "
                    f"window size ({window_h}, {window_w})"
                )
            index = torch.arange(height * width).reshape(height, width)
            num_windows_h = height // window_h
            num_windows_w = width // window_w
            num_windows = num_windows_h * num_windows_w

            index = index.reshape(num_windows_h, window_h, num_windows_w, window_w)
            index = index.permute(0, 2, 1, 3).reshape(num_windows, window_h * window_w)

            window_index_list.append(index.reshape(-1) + token_offset)

            cu_this = (
                torch.arange(1, num_windows + 1) * (window_h * window_w)
                + cu_seqlens[-1]
            )
            cu_seqlens.extend(cu_this.tolist())

            token_offset += height * width

# ... truncated for brevity ...
```
**EN:** This method implements `get_window_index(target_sizes: ...)` and Return ``(permutation, per-window cu_seqlens, max_seqlens=4)``.
**CN:** 这个方法实现了 `get_window_index(target_sizes: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 153-225: Method `MiniCPMV_ViTWindowAttentionMerger.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        target_sizes: torch.Tensor,
        cu_seqlens: torch.Tensor,
        max_seqlens: int,
    ) -> Tuple[torch.Tensor, torch.Tensor, torch.Tensor, int]:
        device = hidden_states.device

        # Stage 1: 2x2 window self-attention + residual.
        residual = hidden_states
        hidden_states = self.layer_norm1(hidden_states)

        window_index, window_cu_seqlens, _ = self.get_window_index(target_sizes)
        window_index = window_index.to(device)
        window_cu_seqlens = window_cu_seqlens.to(device)
        if is_npu():
            window_cu_seqlens = window_cu_seqlens.to("cpu")

        hidden_states = hidden_states[:, window_index, :]
        hidden_states = self.self_attn(hidden_states, cu_seqlens=window_cu_seqlens)
        hidden_states = hidden_states[:, torch.argsort(window_index), :]
        hidden_states = residual + hidden_states

        # Stage 2: 2x2 spatial fold + MLP + mean residual.
        if (target_sizes % 2 != 0).any():
            raise ValueError(
                f"All target_sizes must be divisible by 2, got {target_sizes}"
            )
        new_target_sizes = target_sizes // 2

        window_h, window_w = self.window_kernel_size
        batch_size = target_sizes.shape[0]
        all_pixel_values = []
        for batch_idx in range(batch_size):
            height, width = target_sizes[batch_idx]
            patch = hidden_states[
                0, cu_seqlens[batch_idx] : cu_seqlens[batch_idx + 1], :
            ].squeeze(0)

# ... truncated for brevity ...
```
**EN:** This method implements `forward(hidden_states: ..., target_sizes: ..., cu_seqlens: ..., max_seqlens: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., target_sizes: ..., cu_seqlens: ..., max_seqlens: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 228-235: Class `MiniCPMV_DownsampleMLP` overview
```python
class MiniCPMV_DownsampleMLP(nn.Module):
    """One round of 2x2 spatial merge + MLP, used inside ``MiniCPMV_Merger``.

    Input channel dim is ``hidden_size * 4`` (already folded by the caller).
    Output is ``hidden_size`` for an intermediate round or ``llm_embed_dim``
    for the final round.
    """
```
**EN:** Defines `MiniCPMV_DownsampleMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV_DownsampleMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 236-262: Method `MiniCPMV_DownsampleMLP.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        llm_embed_dim: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        merged_hidden_size = hidden_size * 4

        self.pre_norm = nn.LayerNorm(merged_hidden_size, eps=1e-6)
        self.linear_1 = ColumnParallelLinear(
            merged_hidden_size,
            merged_hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("linear_1", prefix),
        )
        self.act = nn.GELU()
        self.linear_2 = RowParallelLinear(
            merged_hidden_size,
            llm_embed_dim,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("linear_2", prefix),
        )
        self.in_features = merged_hidden_size
```
**EN:** This method implements `__init__(hidden_size: ..., llm_embed_dim: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., llm_embed_dim: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 264-269: Method `MiniCPMV_DownsampleMLP.forward`
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states = self.pre_norm(hidden_states).view(-1, self.in_features)
        hidden_states, _ = self.linear_1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 272-280: Class `MiniCPMV_Merger` overview
```python
class MiniCPMV_Merger(nn.Module):
    """Iterative 2x2 fold + MLP chain between ViT and LLM.

    With ``merger_times == 1`` (the 4.6 release default) it's a single
    DownsampleMLP projecting straight into ``text_config.hidden_size``. Each
    additional round halves the grid and keeps the channel width at
    ``vision_config.hidden_size`` until the last round.
    """
```
**EN:** Defines `MiniCPMV_Merger` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV_Merger`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 281-304: Method `MiniCPMV_Merger.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.merge_kernel_size = tuple(config.merge_kernel_size)
        self.merger_times = config.merger_times
        hidden_size = config.vision_config.hidden_size
        llm_embed_dim = config.text_config.hidden_size

        self.mlp = nn.ModuleList(
            [
                MiniCPMV_DownsampleMLP(
                    hidden_size,
                    llm_embed_dim if i == self.merger_times - 1 else hidden_size,
                    quant_config=quant_config,
                    prefix=add_prefix(f"mlp.{i}", prefix),
                )
                for i in range(self.merger_times)
            ]
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 306-351: Method `MiniCPMV_Merger.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        target_sizes: torch.Tensor,
    ) -> torch.Tensor:
        merge_h, merge_w = self.merge_kernel_size

        start = 0
        processed = []
        for batch_idx in range(len(target_sizes)):
            height, width = target_sizes[batch_idx]
            num_patches = int(height * width)

            embed_dim = hidden_states.shape[-1]
            merged_h, merged_w = int(height) // merge_h, int(width) // merge_w
            hidden_state = (
                hidden_states[0, start : start + num_patches, :]
                .view(merged_h, merge_h, merged_w, merge_w, embed_dim)
                .permute(0, 2, 1, 3, 4)
                .reshape(merged_h * merged_w, merge_h * merge_w * embed_dim)
            )
            hidden_state = self.mlp[0](hidden_state)

            height, width = int(height), int(width)
            for i in range(1, self.merger_times):
                if height % merge_h != 0 or width % merge_w != 0:
                    raise ValueError(
                        f"Patch grid ({height}, {width}) must be divisible by "
                        f"merge kernel size {self.merge_kernel_size} at round {i}"
                    )
                height //= merge_h
                width //= merge_w

                inner_dim = hidden_state.shape[-1]
                merged_h, merged_w = height // merge_h, width // merge_w
                hidden_state = (
                    hidden_state.view(merged_h, merge_h, merged_w, merge_w, inner_dim)
                    .permute(0, 2, 1, 3, 4)
                    .reshape(merged_h * merged_w, merge_h * merge_w * inner_dim)
                )
# ... truncated for brevity ...
```
**EN:** This method implements `forward(hidden_states: ..., target_sizes: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., target_sizes: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 354-363: Class `MiniCPMV_VisionEncoderLayer` overview
```python
class MiniCPMV_VisionEncoderLayer(Idefics2EncoderLayer):
    """SigLip-style pre-norm encoder layer for packed NaViT input.

    Inherits Idefics2's forward and submodule layout (so HF weights map
    verbatim), then rebuilds ``self_attn`` with ``flatten_batch=True`` for
    per-image block-diagonal attention on a single packed sequence
    (Idefics2 uses padded ``(B, max_patches, D)``) and the SigLip-correct
    ``projection_size = hidden_size`` (Idefics2 sets it to ``intermediate_size``).
    """
```
**EN:** Defines `MiniCPMV_VisionEncoderLayer` as a reusable runtime type derived from Idefics2EncoderLayer. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV_VisionEncoderLayer`，其继承关系为 Idefics2EncoderLayer。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 364-381: Method `MiniCPMV_VisionEncoderLayer.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__(config, quant_config=quant_config, prefix=prefix)
        self.self_attn = VisionAttention(
            embed_dim=config.hidden_size,
            num_heads=config.num_attention_heads,
            projection_size=config.hidden_size,
            use_qkv_parallel=True,
            quant_config=quant_config,
            dropout=config.attention_dropout,
            softmax_in_single_precision=True,
            flatten_batch=True,
            prefix=add_prefix("self_attn", prefix),
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 384-391: Class `MiniCPMV_VisionEncoder` overview
```python
class MiniCPMV_VisionEncoder(Idefics2Encoder):
    """Stack of ``MiniCPMV_VisionEncoderLayer``.

    ``vit_merger`` lives one level up on ``MiniCPMV_VisionTransformer`` so the
    HF checkpoint key ``vision_tower.vit_merger.*`` lands at the matching
    sglang path.
    """
```
**EN:** Defines `MiniCPMV_VisionEncoder` as a reusable runtime type derived from Idefics2Encoder. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV_VisionEncoder`，其继承关系为 Idefics2Encoder。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 392-408: Method `MiniCPMV_VisionEncoder.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__(config, quant_config=quant_config, prefix=prefix)
        self.layers = nn.ModuleList(
            [
                MiniCPMV_VisionEncoderLayer(
                    config,
                    quant_config=quant_config,
                    prefix=add_prefix(f"layers.{i}", prefix),
                )
                for i in range(config.num_hidden_layers)
            ]
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 411-420: Class `MiniCPMV_VisionTransformer` overview
```python
class MiniCPMV_VisionTransformer(nn.Module):
    """Vision Transformer for MiniCPM-V 4.6.

    Reuses sglang's SigLIP-style ``Idefics2VisionEmbeddings`` + encoder layers,
    inserts ``MiniCPMV_ViTWindowAttentionMerger`` after layer ``insert_layer_id``,
    and applies post-encoder LayerNorm. ``forward`` returns
    ``(hidden_states, target_sizes)``; in ``"16x"`` mode ``target_sizes``
    reflects the post-merger grid, which downstream code must use.
    """
```
**EN:** Defines `MiniCPMV_VisionTransformer` as a reusable runtime type derived from nn.Module. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMV_VisionTransformer`，其继承关系为 nn.Module。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 421-453: Method `MiniCPMV_VisionTransformer.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        require_post_norm: bool = True,
        prefix: str = "",
    ) -> None:
        super().__init__()
        embed_dim = config.hidden_size
        self.config = config

        if not hasattr(config, "insert_layer_id"):
            raise ValueError(
                "MiniCPMV_VisionTransformer requires `config.insert_layer_id`"
            )

        self.insert_layer_id = config.insert_layer_id
        self.embeddings = Idefics2VisionEmbeddings(config)
        self.encoder = MiniCPMV_VisionEncoder(
            config=config,
            quant_config=quant_config,
            prefix=add_prefix("encoder", prefix),
        )
        self.post_layernorm = (
            nn.LayerNorm(embed_dim, eps=config.layer_norm_eps)
            if require_post_norm
            else nn.Identity()
        )
        self.vit_merger = MiniCPMV_ViTWindowAttentionMerger(
            config,
            quant_config=quant_config,
            prefix=add_prefix("vit_merger", prefix),
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., require_post_norm: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., require_post_norm: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 455-456: Method `MiniCPMV_VisionTransformer.get_input_embeddings`
```python
    def get_input_embeddings(self) -> nn.Module:
        return self.embeddings
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 458-469: Method `MiniCPMV_VisionTransformer.compute_cu_seqlens`
```python
    @staticmethod
    def compute_cu_seqlens(target_sizes: torch.Tensor) -> Tuple[torch.Tensor, int]:
        seqlen = (target_sizes[:, 0] * target_sizes[:, 1]).to(torch.int32)
        cu_seqlens = torch.cat(
            [
                torch.tensor([0], device=seqlen.device, dtype=torch.int32),
                torch.cumsum(seqlen, dim=0, dtype=torch.int32),
            ],
            dim=0,
        )
        max_seqlens = int(seqlen.max().item())
        return cu_seqlens, max_seqlens
```
**EN:** This method implements `compute_cu_seqlens()` and implements a focused helper that supports the surrounding runtime flow inside `MiniCPMV_VisionTransformer` Decorators: staticmethod.
**CN:** 这个方法实现了 `compute_cu_seqlens()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiniCPMV_VisionTransformer` 内部调用 装饰器：staticmethod。

### Lines 471-483: Method `MiniCPMV_VisionTransformer._pad_to_pack`
```python
    @staticmethod
    def _pad_to_pack(padded: torch.Tensor, target_sizes: torch.Tensor) -> torch.Tensor:
        """``(B, max_patches, D) -> (1, sum_patches, D)``.

        ``Idefics2VisionEmbeddings`` emits padded shape with valid tokens at
        ``[0, h_b * w_b)`` of each batch row. Strip the padding so the rest
        of the ViT runs in flat NaViT form.
        """
        seqlens = (target_sizes[:, 0] * target_sizes[:, 1]).to(torch.long)
        if padded.shape[0] == 1:
            return padded[:, : int(seqlens[0].item()), :]
        parts = [padded[b, : int(seqlens[b].item()), :] for b in range(padded.shape[0])]
        return torch.cat(parts, dim=0).unsqueeze(0)
```
**EN:** This method implements `_pad_to_pack(target_sizes: ...)` and ``(B, max_patches, D) -> (1, sum_patches, D)``. Decorators: staticmethod.
**CN:** 这个方法实现了 `_pad_to_pack(target_sizes: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：staticmethod。

### Lines 485-526: Method `MiniCPMV_VisionTransformer.forward`
```python
    def forward(
        self,
        pixel_values: torch.Tensor,
        patch_attention_mask: Optional[torch.BoolTensor] = None,
        target_sizes: Optional[torch.IntTensor] = None,
        use_vit_merger: bool = True,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        if target_sizes is None:
            raise ValueError("MiniCPMV_VisionTransformer requires `target_sizes`.")

        hidden_states = self.embeddings(
            pixel_values=pixel_values,
            patch_attention_mask=patch_attention_mask,
            tgt_sizes=target_sizes,
        )
        hidden_states = self._pad_to_pack(hidden_states, target_sizes)
        cu_seqlens, max_seqlens = self.compute_cu_seqlens(target_sizes)
        if is_npu():
            cu_seqlens = cu_seqlens.to("cpu")

        if use_vit_merger:
            # Encoder loop lives here (not inside ``MiniCPMV_VisionEncoder``)
            # so we can fire ``vit_merger`` after layer ``insert_layer_id``
            # without coupling the encoder module to it.
            for layer_index, layer in enumerate(self.encoder.layers):
                hidden_states = layer(hidden_states, cu_seqlens=cu_seqlens)
                if layer_index == self.insert_layer_id:
                    (
                        hidden_states,
                        target_sizes,
                        cu_seqlens,
                        max_seqlens,
                    ) = self.vit_merger(
                        hidden_states, target_sizes, cu_seqlens, max_seqlens
                    )
                    if is_npu():
                        cu_seqlens = cu_seqlens.to("cpu")
        else:
            hidden_states = self.encoder(hidden_states, cu_seqlens=cu_seqlens)

# ... truncated for brevity ...
```
**EN:** This method implements `forward(pixel_values: ..., patch_attention_mask: ...=..., target_sizes: ...=..., use_vit_merger: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(pixel_values: ..., patch_attention_mask: ...=..., target_sizes: ...=..., use_vit_merger: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影
- **EN:** Multimodal feature processing / **CN:** 多模态特征处理

## Dependencies / 依赖关系
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn.functional`
- `torch.nn`
- `transformers.PretrainedConfig`
- `sglang.srt.layers.activation.get_act_fn`
- `sglang.srt.layers.attention.vision.VisionAttention`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.models.idefics2.Idefics2Encoder`
- `sglang.srt.models.idefics2.Idefics2EncoderLayer`
- `sglang.srt.models.idefics2.Idefics2VisionEmbeddings`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.is_npu`
