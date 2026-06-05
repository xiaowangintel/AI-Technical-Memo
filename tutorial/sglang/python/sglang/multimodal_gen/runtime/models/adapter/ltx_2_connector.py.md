# ltx_2_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/adapter/ltx_2_connector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for ltx 2 connector within the multimodal runtime. Key symbols include `apply_interleaved_rotary_emb`, `apply_split_rotary_emb`, `_ltx2_connector_rope_freq_grid_np`. / 该模块实现多模态运行时中与 ltx 2 connector 相关的模型构件。 关键符号包括 `apply_interleaved_rotary_emb`, `apply_split_rotary_emb`, `_ltx2_connector_rope_freq_grid_np`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup / 导入与模块初始化
```python
import functools
import math
from typing import Optional, Tuple, Union

import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
from diffusers.models.attention import FeedForward

from sglang.multimodal_gen.configs.models.adapter.ltx_2_connector import (
    LTX2ConnectorConfig,
)
from sglang.multimodal_gen.runtime.layers.attention import USPAttention
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 18-24: Function `apply_interleaved_rotary_emb` / 函数 `apply_interleaved_rotary_emb`
```python
def apply_interleaved_rotary_emb(
    x: torch.Tensor, freqs: Tuple[torch.Tensor, torch.Tensor]
) -> torch.Tensor:
    cos, sin = freqs
    x_real, x_imag = x.unflatten(2, (-1, 2)).unbind(-1)  # [B, S, C // 2]
    x_rotated = torch.stack([-x_imag, x_real], dim=-1).flatten(2)
    return x * cos + x_rotated * sin
```
**EN:** This function drives `apply_interleaved_rotary_emb` with inputs such as `x`, `freqs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `apply_interleaved_rotary_emb`，主要处理 `x`, `freqs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 27-71: Function `apply_split_rotary_emb` / 函数 `apply_split_rotary_emb`
```python
def apply_split_rotary_emb(
    x: torch.Tensor, freqs: Tuple[torch.Tensor, torch.Tensor]
) -> torch.Tensor:
    cos, sin = freqs

    x_dtype = x.dtype
    needs_reshape = False
    if x.ndim != 4 and cos.ndim == 4:
        # cos is (#b, h, t, r) -> reshape x to (b, h, t, dim_per_head)
        # The cos/sin batch dim may only be broadcastable, so take batch size from x
        b = x.shape[0]
        _, h, t, _ = cos.shape
        x = x.reshape(b, t, h, -1).transpose(1, 2)
        needs_reshape = True
# ...
        out = out.transpose(1, 2).reshape(b, t, -1)

    out = out.to(dtype=x_dtype)
    return out
```
**EN:** This function drives `apply_split_rotary_emb` with inputs such as `x`, `freqs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `apply_split_rotary_emb`，主要处理 `x`, `freqs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 74-84: Function `_ltx2_connector_rope_freq_grid_np` / 函数 `_ltx2_connector_rope_freq_grid_np`
```python
@functools.lru_cache(maxsize=5)
def _ltx2_connector_rope_freq_grid_np(
    theta: float, num_pos_dims: int, dim: int
) -> torch.Tensor:
    # Official LTX uses NumPy float64 for double-precision RoPE frequencies.
    n_elem = 2 * num_pos_dims
    pow_indices = np.power(
        theta,
        np.linspace(0.0, 1.0, dim // n_elem, dtype=np.float64),
    )
    return torch.tensor(pow_indices * math.pi / 2.0, dtype=torch.float32)
```
**EN:** This function drives `_ltx2_connector_rope_freq_grid_np` with inputs such as `theta`, `num_pos_dims`, `dim`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_ltx2_connector_rope_freq_grid_np`，主要处理 `theta`, `num_pos_dims`, `dim` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 87-233: Class `LTX2Attention` / 类 `LTX2Attention`
```python
class LTX2Attention(torch.nn.Module):
    r"""
    Attention class for all LTX-2.0 attention layers. Compared to LTX-1.0, this supports specifying the query and key
    RoPE embeddings separately for audio-to-video (a2v) and video-to-audio (v2a) cross-attention.
    """

    def __init__(
        self,
        query_dim: int,
        heads: int = 8,
        kv_heads: int = 8,
        dim_head: int = 64,
        dropout: float = 0.0,
        bias: bool = True,
# ...

        hidden_states = self.to_out[0](hidden_states)
        hidden_states = self.to_out[1](hidden_states)
        return hidden_states
```
**EN:** This class models `LTX2Attention` as a specialization of `torch.nn.Module`. Attention class for all LTX-2.0 attention layers. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2Attention`，并继承/扩展 `torch.nn.Module`。 文档字符串指出：Attention class for all LTX-2.0 attention layers. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 236-341: Class `LTX2RotaryPosEmbed1d` / 类 `LTX2RotaryPosEmbed1d`
```python
class LTX2RotaryPosEmbed1d(nn.Module):
    """
    1D rotary positional embeddings (RoPE) for the LTX 2.0 text encoder connectors.
    """

    def __init__(
        self,
        dim: int,
        base_seq_len: int = 4096,
        theta: float = 10000.0,
        double_precision: bool = True,
        rope_type: str = "interleaved",
        num_attention_heads: int = 32,
    ):
# ...
        if dtype is not None:
            cos_freqs = cos_freqs.to(dtype)
            sin_freqs = sin_freqs.to(dtype)
        return cos_freqs, sin_freqs
```
**EN:** This class models `LTX2RotaryPosEmbed1d` as a specialization of `nn.Module`. 1D rotary positional embeddings (RoPE) for the LTX 2.0 text encoder connectors. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2RotaryPosEmbed1d`，并继承/扩展 `nn.Module`。 文档字符串指出：1D rotary positional embeddings (RoPE) for the LTX 2.0 text encoder connectors. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 344-388: Class `LTX2TransformerBlock1d` / 类 `LTX2TransformerBlock1d`
```python
class LTX2TransformerBlock1d(nn.Module):
    def __init__(
        self,
        dim: int,
        num_attention_heads: int,
        attention_head_dim: int,
        activation_fn: str = "gelu-approximate",
        eps: float = 1e-6,
        rope_type: str = "interleaved",
        apply_gated_attention: bool = False,
    ):
        super().__init__()

        self.norm1 = torch.nn.RMSNorm(dim, eps=eps, elementwise_affine=False)
# ...
        ff_hidden_states = self.ff(norm_hidden_states)
        hidden_states = hidden_states + ff_hidden_states

        return hidden_states
```
**EN:** This class models `LTX2TransformerBlock1d` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2TransformerBlock1d`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 391-528: Class `LTX2ConnectorTransformer1d` / 类 `LTX2ConnectorTransformer1d`
```python
class LTX2ConnectorTransformer1d(nn.Module):
    """
    A 1D sequence transformer for modalities such as text.
    In LTX 2.0, this is used to process the text encoder hidden states for each of the video and audio streams.
    """

    _supports_gradient_checkpointing = True

    def __init__(
        self,
        num_attention_heads: int = 30,
        attention_head_dim: int = 128,
        num_layers: int = 2,
        num_learnable_registers: int | None = 128,
# ...

        hidden_states = self.norm_out(hidden_states)

        return hidden_states, attention_mask
```
**EN:** This class models `LTX2ConnectorTransformer1d` as a specialization of `nn.Module`. A 1D sequence transformer for modalities such as text. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2ConnectorTransformer1d`，并继承/扩展 `nn.Module`。 文档字符串指出：A 1D sequence transformer for modalities such as text. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 531-705: Class `LTX2TextConnectors` / 类 `LTX2TextConnectors`
```python
class LTX2TextConnectors(nn.Module):
    """
    Text connector stack used by LTX 2.0 to process the packed text encoder hidden states for both the video and audio
    streams.
    """

    def __init__(
        self,
        config: LTX2ConnectorConfig,
    ):
        super().__init__()
        caption_channels = config.caption_channels
        self.caption_channels = caption_channels
        text_proj_in_factor = config.text_proj_in_factor
# ...
            audio_hidden_states, attention_mask
        )

        return video_text_embedding, audio_text_embedding, new_attn_mask
```
**EN:** This class models `LTX2TextConnectors` as a specialization of `nn.Module`. Text connector stack used by LTX 2.0 to process the packed text encoder hidden states for both the video and audio Important methods include `__init__`, `_rescale_v2_features`, `forward`.
**CN:** 该类实现 `LTX2TextConnectors`，并继承/扩展 `nn.Module`。 文档字符串指出：Text connector stack used by LTX 2.0 to process the packed text encoder hidden states for both the video and audio 其中较重要的方法包括 `__init__`, `_rescale_v2_features`, `forward`。

### Lines 706-708: Top-level configuration / 顶层配置
```python


EntryClass = LTX2TextConnectors
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Caching strategy / 缓存策略
- Video generation flow / 视频生成流程
- Audio generation flow / 音频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.adapter.ltx_2_connector`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.platforms`
- **External / 外部**: `numpy`, `torch`, `torch.nn`, `torch.nn.functional`, `diffusers.models.attention`
- **Stdlib / 标准库**: `functools`, `math`, `typing`
