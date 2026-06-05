# minicpmo.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/minicpmo.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only MiniCPM-o model compatible with HuggingFace weights. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 16-16: Module docstring
```python
"""Inference-only MiniCPM-o model compatible with HuggingFace weights."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 18-57: Module imports
```python
import math
from dataclasses import dataclass
from typing import Any, Iterable, List, Literal, Optional, Tuple, Union

import numpy as np
import torch
import torch.nn.functional as F
import torch.nn.utils.parametrize as P
import torch.types
from torch import nn
from torch.nn.utils import parametrizations
from tqdm import tqdm
from transformers import LlamaConfig, LlamaModel, PretrainedConfig, PreTrainedModel
from transformers.activations import ACT2FN
from transformers.cache_utils import DynamicCache, EncoderDecoderCache
from transformers.modeling_outputs import BaseModelOutputWithPast, ModelOutput
from transformers.models.whisper.modeling_whisper import (
    WhisperAttention,
    WhisperConfig,
    WhisperEncoder,
)

from sglang.srt.layers.quantization import QuantizationConfig
from sglang.srt.managers.mm_utils import (
    MultiModalityDataPaddingPatternTokenPairs,
    general_mm_embed_routine,
)
from sglang.srt.managers.schedule_batch import (
    MultimodalDataItem,
    MultimodalInputFormat,
    MultimodalInputs,
    flatten_nested_list,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.utils import set_default_torch_dtype
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.idefics2 import Idefics2VisionTransformer
from sglang.srt.models.minicpmv import MiniCPMBaseModel, Resampler2_5
from sglang.srt.models.qwen2 import Qwen2ForCausalLM
from sglang.srt.utils import logger
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 59-66: Top-level try
```python
try:
    from transformers import LogitsWarper
    from vector_quantize_pytorch import GroupedResidualFSQ

    _tts_deps = True
except:
    LogitsWarper = None
    _tts_deps = False
```
**EN:** Wraps top-level setup in a guarded block to handle optional imports, resource management, or fallback behavior safely.
**CN:** 使用受保护的顶层代码块安全处理可选导入、资源管理或回退行为。

### Lines 69-102: Function `apply_spk_emb`
```python
def apply_spk_emb(
    input_ids: torch.Tensor = None,
    spk_emb: torch.Tensor = None,
    input_embeds: torch.Tensor = None,
    spk_emb_token_id: int = 0,
    num_spk_embs: int = 1,
):
    """
    Replace consecutive `num_spk_embs` speaker embedding placeholders in input_embeds with pre-prepared speaker embeddings. This is an in-place replacement, no new tensor is created, so no value is returned.

    Args:
        input_ids (torch.Tensor): Input ID tensor, shape [batch_size, seq_len_max]
        spk_emb (torch.Tensor): Speaker embedding tensor, shape [batch_size, num_spk_emb, hidden_dim]
        input_embeds (torch.Tensor): Input embedding tensor, shape [batch_size, seq_len_max, hidden_dim]
        spk_emb_token_id (int): ID of the speaker embedding token
        num_spk_embs (int): Number of speaker embeddings

    Returns:
        None
    """

    batch_size = input_ids.shape[0]

    for idx in range(batch_size):
        input_ids_ = input_ids[idx]  # [seq_len_max]
        spk_emb_ = spk_emb[idx]  # [num_spk_emb]
        mask_ = input_ids_ == spk_emb_token_id  # [batch_size, seq_len_max]
        nonzero_position_idx = mask_.nonzero(as_tuple=False)  # [num_spk_emb, 1]
        assert nonzero_position_idx.shape[0] == num_spk_embs
        begin_idx = nonzero_position_idx.min()
        end_idx = nonzero_position_idx.max()
        input_embeds[idx, begin_idx : end_idx + 1, :] = spk_emb_

    return
```
**EN:** This function implements `apply_spk_emb(input_ids: ...=..., spk_emb: ...=..., input_embeds: ...=..., spk_emb_token_id: ...=..., num_spk_embs: ...=...)` and Replace consecutive `num_spk_embs` speaker embedding placeholders in input_embeds with pre-prepared speaker embeddings.
**CN:** 这个函数实现了 `apply_spk_emb(input_ids: ...=..., spk_emb: ...=..., input_embeds: ...=..., spk_emb_token_id: ...=..., num_spk_embs: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 105-117: Class `ConditionalChatTTSGenerationOutput` overview
```python
@dataclass
class ConditionalChatTTSGenerationOutput(ModelOutput):
    """
    Output class for ConditionalChatTTS generation.

    Args:
        new_ids (torch.LongTensor): Newly generated audio code sequence, shape (batch_size, sequence_length, num_vq).
        audio_input_ids (torch.LongTensor): Updated input IDs including condition and generated audio codes, shape (batch_size, full_sequence_length, num_vq).
        past_key_values (Tuple[Tuple[torch.FloatTensor]]): Tuple containing pre-computed keys and values used for attention mechanism. Each element has shape (batch_size, num_heads, sequence_length, embed_size_per_head).
        finished (bool): Boolean indicating whether generation is complete.

    """
```
**EN:** Defines `ConditionalChatTTSGenerationOutput` as a reusable runtime type derived from ModelOutput. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ConditionalChatTTSGenerationOutput`，其继承关系为 ModelOutput。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 118-121: Class `ConditionalChatTTSGenerationOutput` attributes
```python
    new_ids: torch.LongTensor = None
    audio_input_ids: torch.LongTensor = None
    past_key_values: Optional[Tuple[Tuple[torch.FloatTensor]]] = None
    finished: bool = None
```
**EN:** Defines class-level attributes and metadata that shape how `ConditionalChatTTSGenerationOutput` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `ConditionalChatTTSGenerationOutput` 在运行时的行为。

### Lines 124-196: Function `make_streaming_chunk_mask_generation`
```python
def make_streaming_chunk_mask_generation(
    inputs_embeds: torch.Tensor,
    past_seen_tokens: int,
    streaming_tts_text_mask: torch.Tensor,
    streaming_reserved_length: int = 300,
    streaming_audio_chunk_size: int = 50,
    streaming_text_chunk_size: int = 10,
    num_spk_emb: int = 1,
    use_spk_emb: bool = True,
) -> torch.Tensor:
    """
    In streaming audio generation, determine which `text` positions the TTS model can attend to when generating each chunk of `audio` tokens.

    This function creates a mask that allows the model to attend to a specific chunk of text
    tokens when generating each chunk of audio tokens, enabling streaming TTS generation.

    Args:
        inputs_embeds (torch.Tensor): Input embeddings tensor.
        past_seen_tokens (int): Number of tokens already seen by the model.
        streaming_tts_text_mask (torch.Tensor): Mask for the text tokens.
        streaming_reserved_length (int, optional): Number of reserved tokens for streaming. Defaults to 300.
        streaming_text_chunk_size (int, optional): Size of each text chunk. Defaults to 7.

    Returns:
        torch.Tensor: Causal mask for streaming TTS generation, shape is [batch_size=1, 1, seq_len=1, past_seen_tokens+1]

    Raises:
        AssertionError: If the batch size is not 1 (only supports batch size of 1 for inference).
    """
    assert inputs_embeds.shape[0] == 1

    dtype = inputs_embeds.dtype
    device = inputs_embeds.device
    min_dtype = torch.finfo(dtype).min

    # Add `1` to the past seen tokens to account for new `tokens` during `generate`
    causal_mask = torch.full(
        (1, past_seen_tokens + inputs_embeds.shape[1]),
        fill_value=0,
        dtype=dtype,
# ... truncated for brevity ...
```
**EN:** This function implements `make_streaming_chunk_mask_generation(inputs_embeds: ..., past_seen_tokens: ..., streaming_tts_text_mask: ..., streaming_reserved_length: ...=..., streaming_audio_chunk_size: ...=..., streaming_text_chunk_size: ...=..., ...)` and In streaming audio generation, determine which `text` positions the TTS model can attend to when generating each chunk of `audio` tokens.
**CN:** 这个函数实现了 `make_streaming_chunk_mask_generation(inputs_embeds: ..., past_seen_tokens: ..., streaming_tts_text_mask: ..., streaming_reserved_length: ...=..., streaming_audio_chunk_size: ...=..., streaming_text_chunk_size: ...=..., ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 200-200: Class `ConvNeXtBlock` overview
```python
class ConvNeXtBlock(nn.Module):
```
**EN:** Defines `ConvNeXtBlock` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ConvNeXtBlock`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 201-228: Method `ConvNeXtBlock.__init__`
```python
    def __init__(
        self,
        dim: int,
        intermediate_dim: int,
        kernel: int,
        dilation: int,
        layer_scale_init_value: float = 1e-6,
    ):
        # ConvNeXt Block copied from Vocos.
        super().__init__()
        self.dwconv = nn.Conv1d(
            dim,
            dim,
            kernel_size=kernel,
            padding=dilation * (kernel // 2),
            dilation=dilation,
            groups=dim,
        )

        self.norm = nn.LayerNorm(dim, eps=1e-6)
        self.pwconv1 = nn.Linear(dim, intermediate_dim)
        self.act = nn.GELU()
        self.pwconv2 = nn.Linear(intermediate_dim, dim)
        self.coef = (
            nn.Parameter(layer_scale_init_value * torch.ones(dim), requires_grad=True)
            if layer_scale_init_value > 0
            else None
        )
```
**EN:** This method implements `__init__(dim: ..., intermediate_dim: ..., kernel: ..., dilation: ..., layer_scale_init_value: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., intermediate_dim: ..., kernel: ..., dilation: ..., layer_scale_init_value: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 230-250: Method `ConvNeXtBlock.forward`
```python
    def forward(self, x: torch.Tensor, cond=None) -> torch.Tensor:
        residual = x

        y = self.dwconv(x)
        y.transpose_(1, 2)  # (B, C, T) -> (B, T, C)
        x = self.norm(y)
        del y
        y = self.pwconv1(x)
        del x
        x = self.act(y)
        del y
        y = self.pwconv2(x)
        del x
        if self.coef is not None:
            y *= self.coef
        y.transpose_(1, 2)  # (B, T, C) -> (B, C, T)

        x = y + residual
        del y

        return x
```
**EN:** This method implements `forward(x: ..., cond=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ..., cond=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 254-254: Class `DVAEDecoder` overview
```python
class DVAEDecoder(nn.Module):
```
**EN:** Defines `DVAEDecoder` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `DVAEDecoder`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 255-284: Method `DVAEDecoder.__init__`
```python
    def __init__(
        self,
        idim: int,
        odim: int,
        n_layer=12,
        bn_dim=64,
        hidden=256,
        kernel=7,
        dilation=2,
        up=False,
    ):
        super().__init__()
        self.up = up
        self.conv_in = nn.Sequential(
            nn.Conv1d(idim, bn_dim, 3, 1, 1),
            nn.GELU(),
            nn.Conv1d(bn_dim, hidden, 3, 1, 1),
        )
        self.decoder_block = nn.ModuleList(
            [
                ConvNeXtBlock(
                    hidden,
                    hidden * 4,
                    kernel,
                    dilation,
                )
                for _ in range(n_layer)
            ]
        )
        self.conv_out = nn.Conv1d(hidden, odim, kernel_size=1, bias=False)
```
**EN:** This method implements `__init__(idim: ..., odim: ..., n_layer=..., bn_dim=..., hidden=..., kernel=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(idim: ..., odim: ..., n_layer=..., bn_dim=..., hidden=..., kernel=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 286-295: Method `DVAEDecoder.forward`
```python
    def forward(self, x: torch.Tensor, conditioning=None) -> torch.Tensor:
        # B, C, T
        y = self.conv_in(x)
        del x
        for f in self.decoder_block:
            y = f(y, conditioning)

        x = self.conv_out(y)
        del y
        return x
```
**EN:** This method implements `forward(x: ..., conditioning=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ..., conditioning=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 299-299: Class `GFSQ` overview
```python
class GFSQ(nn.Module):
```
**EN:** Defines `GFSQ` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `GFSQ`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 300-320: Method `GFSQ.__init__`
```python
    def __init__(
        self,
        dim: int,
        levels: List[int],
        G: int,
        R: int,
        eps=1e-5,
        transpose=True,
    ):
        super(GFSQ, self).__init__()
        self.quantizer = GroupedResidualFSQ(
            dim=dim,
            levels=list(levels),
            num_quantizers=R,
            groups=G,
        )
        self.n_ind = math.prod(levels)
        self.eps = eps
        self.transpose = transpose
        self.G = G
        self.R = R
```
**EN:** This method implements `__init__(dim: ..., levels: ..., G: ..., R: ..., eps=..., transpose=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(dim: ..., levels: ..., G: ..., R: ..., eps=..., transpose=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 322-327: Method `GFSQ._embed`
```python
    def _embed(self, x: torch.Tensor):
        if self.transpose:
            x = x.transpose(1, 2)
        x = x.view(x.size(0), x.size(1), self.G, self.R).permute(2, 0, 1, 3)
        feat = self.quantizer.get_output_from_indices(x)
        return feat.transpose_(1, 2) if self.transpose else feat
```
**EN:** This method implements `_embed(x: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `_embed(x: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 329-330: Method `GFSQ.__call__`
```python
    def __call__(self, x: torch.Tensor) -> torch.Tensor:
        return super().__call__(x)
```
**EN:** This method implements `__call__(x: ...)` and implements a focused helper that supports the surrounding runtime flow inside `GFSQ`.
**CN:** 这个方法实现了 `__call__(x: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `GFSQ` 内部调用。

### Lines 332-338: Method `GFSQ.forward`
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        if self.transpose:
            x.transpose_(1, 2)
        _, ind = self.quantizer(x)
        ind = ind.permute(1, 2, 0, 3).contiguous()
        ind = ind.view(ind.size(0), ind.size(1), -1)
        return ind.transpose_(1, 2) if self.transpose else ind
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 342-342: Class `DVAE` overview
```python
class DVAE(nn.Module):
```
**EN:** Defines `DVAE` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `DVAE`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 343-381: Method `DVAE.__init__`
```python
    def __init__(
        self,
    ):
        super().__init__()

        coef = torch.rand(100)
        self.coef = nn.Parameter(coef.unsqueeze(0).unsqueeze_(2))

        self.downsample_conv = nn.Sequential(
            nn.Conv1d(100, 512, 3, 1, 1),
            nn.GELU(),
            nn.Conv1d(512, 512, 4, 2, 1),
            nn.GELU(),
        )

        self.encoder = DVAEDecoder(
            idim=512,
            odim=1024,
            hidden=256,
            n_layer=12,
            bn_dim=128,
        )

        self.decoder = DVAEDecoder(
            idim=512,
            odim=512,
            hidden=256,
            n_layer=12,
            bn_dim=128,
        )

        self.out_conv = nn.Conv1d(512, 100, 3, 1, 1, bias=False)

        self.vq_layer = GFSQ(
            dim=1024,
            levels=(5, 5, 5, 5),
            G=2,
            R=2,
        )
```
**EN:** This method implements `__init__()` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__()`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 383-419: Method `DVAE.forward`
```python
    @torch.inference_mode()
    def forward(
        self, inp: torch.Tensor, mode: Literal["encode", "decode"] = "decode"
    ) -> torch.Tensor:
        if mode == "encode" and hasattr(self, "encoder") and self.vq_layer is not None:
            mel = inp.clone()
            x: torch.Tensor = self.downsample_conv(
                torch.div(mel, self.coef.view(100, 1).expand(mel.shape), out=mel),
            ).unsqueeze_(0)
            del mel
            x = self.encoder(x)
            ind = self.vq_layer(x)
            del x
            return ind

        if self.vq_layer is not None:
            vq_feats = self.vq_layer._embed(inp)
        else:
            vq_feats = inp

        vq_feats = (
            vq_feats.view(
                (vq_feats.size(0), 2, vq_feats.size(1) // 2, vq_feats.size(2)),
            )
            .permute(0, 2, 3, 1)
            .flatten(2)
        )

        dec_out = self.out_conv(
            self.decoder(
                x=vq_feats,
            ),
        )

        del vq_feats

        return torch.mul(dec_out, self.coef, out=dec_out)
```
**EN:** This method implements `forward(inp: ..., mode: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.inference_mode().
**CN:** 这个方法实现了 `forward(inp: ..., mode: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.inference_mode()。

### Lines 423-423: Class `CustomRepetitionPenaltyLogitsProcessorRepeat` overview
```python
class CustomRepetitionPenaltyLogitsProcessorRepeat:
```
**EN:** Defines `CustomRepetitionPenaltyLogitsProcessorRepeat` as a reusable runtime type derived from no explicit base class. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `CustomRepetitionPenaltyLogitsProcessorRepeat`，其继承关系为 no explicit base class。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 424-432: Method `CustomRepetitionPenaltyLogitsProcessorRepeat.__init__`
```python
    def __init__(self, penalty: float, max_input_ids: int, past_window: int):
        if not isinstance(penalty, float) or not (penalty > 0):
            raise ValueError(
                f"`penalty` has to be a strictly positive float, but is {penalty}"
            )

        self.penalty = penalty
        self.max_input_ids = max_input_ids
        self.past_window = past_window
```
**EN:** This method implements `__init__(penalty: ..., max_input_ids: ..., past_window: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(penalty: ..., max_input_ids: ..., past_window: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 434-451: Method `CustomRepetitionPenaltyLogitsProcessorRepeat.__call__`
```python
    def __call__(
        self, input_ids: torch.LongTensor, scores: torch.FloatTensor
    ) -> torch.FloatTensor:
        if input_ids.size(1) > self.past_window:
            input_ids = input_ids.narrow(1, -self.past_window, self.past_window)
        freq = F.one_hot(input_ids, scores.size(1)).sum(1)
        if freq.size(0) > self.max_input_ids:
            freq.narrow(
                0, self.max_input_ids, freq.size(0) - self.max_input_ids
            ).zero_()
        alpha = torch.pow(self.penalty, freq)
        scores = scores.contiguous()
        inp = scores.multiply(alpha)
        oth = scores.divide(alpha)
        con = scores < 0
        out = torch.where(con, inp, oth)
        del inp, oth, scores, con, alpha
        return out
```
**EN:** This method implements `__call__(input_ids: ..., scores: ...)` and implements a focused helper that supports the surrounding runtime flow inside `CustomRepetitionPenaltyLogitsProcessorRepeat`.
**CN:** 这个方法实现了 `__call__(input_ids: ..., scores: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `CustomRepetitionPenaltyLogitsProcessorRepeat` 内部调用。

### Lines 454-545: Class `ConditionalChatTTS` overview
```python
class ConditionalChatTTS(PreTrainedModel):
    """A conditional text-to-speech model that can generate speech from text with speaker conditioning.

    This model extends PreTrainedModel to provide text-to-speech capabilities with:
    - LLM hidden state conditioning
    - Streaming generation

    The model uses a transformer architecture with LLM hidden states and can operate in both
    streaming and non-streaming modes for flexible deployment.

    The model process sequence in the following format:
    | text bos token | LLM embedding projected to tts embedding space | text tokens (fixed length, reserved for future tokens) | audio bos token | audio tokens (audio token length is not fixed)| audio eos token |

    The format is designed to support LLM-conditioned streaming audio generation.

    Usage:
    To support streaming generation, two global variables should be maintained outside of the model.
        1. `audio_input_ids`: stores *discrete* audio codes. It is a tensor with shape [1, sequence length+1, num_vq].
        2. `past_key_values`: stores the KV cache for both text tokens and audio codes. It is a list of tuples, each tuple contains two tensors with shape [1, num_attention_heads, sequence length, hidden_size // num_attention_heads]

    where `num_vq` is the number of audio codebooks, in default setting, it is `4`.

    1. Create an empty `past_key_values` with
    ```python
    initial_kv_cache_length = 1 + model.num_spk_embs + model.streaming_text_reserved_len # where `1` denotes the `bos` token
    dtype = model.emb_text.weight.dtype
    device = model.emb_text.weight.device
    past_key_values = [
        (
            torch.zeros(1, model.config.num_attention_heads, initial_kv_cache_length, model.config.hidden_size // model.config.num_attention_heads, dtype=dtype, device=device),
            torch.zeros(1, model.config.num_attention_heads, initial_kv_cache_length, model.config.hidden_size // model.config.num_attention_heads, dtype=dtype, device=device)
        )
        for _ in range(model.config.num_hidden_layers)
    ]

    2. At the same time, create an empty `audio_input_ids` with shape [1, sequence length, num_vq], `num_vq` denotes multiple layer audio codebooks. But here we also include text tokens in the sequence, but they will be zeros, and will not be used, just a placeholder.

    ```python
    initial_audio_input_ids_length = 1 + model.num_spk_embs + model.streaming_text_reserved_len + 1
    # [bos token, speaker embeddings, text tokens, audio bos token]
# ... truncated for brevity ...
```
**EN:** Defines `ConditionalChatTTS` as a reusable runtime type derived from PreTrainedModel. The class groups 6 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `ConditionalChatTTS`，其继承关系为 PreTrainedModel。这个类组织了 6 个方法，用于实现模型相关行为。

### Lines 546-547: Class `ConditionalChatTTS` attributes
```python
    config_class = PretrainedConfig
    _no_split_modules = []
```
**EN:** Defines class-level attributes and metadata that shape how `ConditionalChatTTS` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `ConditionalChatTTS` 在运行时的行为。

### Lines 549-605: Method `ConditionalChatTTS.__init__`
```python
    def __init__(self, config: PretrainedConfig):
        super().__init__(config)

        self.use_speaker_embedding = config.use_speaker_embedding
        self.use_llm_hidden_state = config.use_llm_hidden_state
        self.num_spk_embs = config.num_spk_embs
        self.spk_emb_token_id = config.spk_emb_token_id

        self.use_text = config.use_text
        self.streaming = config.streaming
        self.streaming_text_chunk_size = config.streaming_text_chunk_size
        self.streaming_audio_chunk_size = config.streaming_audio_chunk_size
        self.streaming_text_reserved_len = config.streaming_text_reserved_len
        self.audio_bos_token_id = config.audio_bos_token_id
        self.num_mel_bins = config.num_mel_bins
        self.num_vq = config.num_vq
        self.num_audio_tokens = config.num_audio_tokens

        self.top_p = config.top_p
        self.top_k = config.top_k
        self.repetition_penalty = config.repetition_penalty

        if self.config.use_mlp:
            self.projector = MultiModalProjector(config.llm_dim, config.hidden_size)
        else:
            self.projector = nn.Linear(config.llm_dim, config.hidden_size, bias=False)
        self.emb_code = nn.ModuleList(
            [
                nn.Embedding(config.num_audio_tokens, config.hidden_size)
                for _ in range(config.num_vq)
            ]
        )
        self.emb_text = nn.Embedding(config.num_text_tokens, config.hidden_size)
        self.head_code = nn.ModuleList(
            [
                parametrizations.weight_norm(
                    nn.Linear(config.hidden_size, config.num_audio_tokens, bias=False),
                    name="weight",
                )
                for _ in range(config.num_vq)
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 607-651: Method `ConditionalChatTTS.merge_inputs_embeds`
```python
    @torch.inference_mode()
    def merge_inputs_embeds(
        self,
        input_ids: torch.Tensor,
        lm_spk_emb_last_hidden_states: Optional[torch.Tensor] = None,
    ):
        """Merge `input_ids` and `lm_spk_emb_last_hidden_states` to `inputs_embeds`.

        Args:
            input_ids (torch.Tensor): Input token IDs.
            lm_spk_emb_last_hidden_states (Optional[torch.Tensor], optional): Last hidden states of speaker embeddings from the language model. Defaults to None.

        Raises:
            NotImplementedError: If speaker embedding is not used and language model hidden states are not implemented.

        Returns:
            torch.Tensor: Prepared input embeddings for the model.
        """
        assert input_ids.shape[0] == 1

        # Embed input_ids to input_embeds
        inputs_embeds = self.emb_text(input_ids)

        # Inject speaker embedding to input_embeds if it exists
        if self.use_speaker_embedding:
            spk_emb_mask = input_ids == self.spk_emb_token_id
            if spk_emb_mask.any():
                assert lm_spk_emb_last_hidden_states is not None
                # Project spk emb to tts hidden size first, [batch_size, num_spk_emb, llm_dim] -> [batch_size, num_spk_emb, self.hidden_size]
                lm_spk_emb_last_hidden_states = lm_spk_emb_last_hidden_states.to(
                    self.projector.linear1.weight.dtype
                )
                projected_spk_emb = self.projector(lm_spk_emb_last_hidden_states)
                projected_spk_emb = F.normalize(projected_spk_emb, p=2, dim=-1)
                apply_spk_emb(
                    input_ids=input_ids,
                    spk_emb=projected_spk_emb,
                    input_embeds=inputs_embeds,
                    spk_emb_token_id=self.spk_emb_token_id,
                    num_spk_embs=self.num_spk_embs,
# ... truncated for brevity ...
```
**EN:** This method implements `merge_inputs_embeds(input_ids: ..., lm_spk_emb_last_hidden_states: ...=...)` and Merge `input_ids` and `lm_spk_emb_last_hidden_states` to `inputs_embeds`. Decorators: torch.inference_mode().
**CN:** 这个方法实现了 `merge_inputs_embeds(input_ids: ..., lm_spk_emb_last_hidden_states: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：torch.inference_mode()。

### Lines 653-723: Method `ConditionalChatTTS.prefill_text`
```python
    @torch.inference_mode()
    def prefill_text(
        self,
        input_ids: torch.Tensor,
        position_ids: torch.LongTensor,
        past_key_values: List[Tuple[torch.Tensor, torch.Tensor]],
        lm_spk_emb_last_hidden_states: Optional[torch.Tensor] = None,
    ):
        """Prefill a chunk of new text tokens in streaming setting.
        Specifically speaking, update `past_key_values` using new text tokens, then the model will read the new text tokens.

        Args:
            input_ids (Tensor): Tensor of shape [batch_size, seq_len]
            position_ids (LongTensor): Tensor of shape [batch_size, seq_len]
            past_key_values (List[Tuple[Tensor]]): KV Cache of all layers, each layer is a tuple (Tensor, Tensor) denoting keys and values. Each tensor is of seq_len = `self.streaming_text_reserved_len`. `past_key_values` will be updated.
            lm_spk_emb_last_hidden_states (Tensor, optional): Tensor of shape [batch_size, num_spk_emb, llm_dim]. Defaults to None.

        Note that all `batch_size` should be `1`.
        """
        assert input_ids.shape[0] == 1
        assert past_key_values is not None

        # Merge text and LLM embeddings
        inputs_embeds = self.merge_inputs_embeds(
            input_ids=input_ids,
            lm_spk_emb_last_hidden_states=lm_spk_emb_last_hidden_states,
        )

        # Clone KV Cache
        past_key_values_for_prefill = []
        for i in range(len(past_key_values)):
            past_key_values_for_prefill.append(
                (
                    past_key_values[i][0][:, :, : position_ids[:, 0], :].clone(),
                    past_key_values[i][1][:, :, : position_ids[:, 0], :].clone(),
                )
            )

        # ModelMiniCPMVBaseModel
        outputs_prefill: BaseModelOutputWithPast = self.model(
# ... truncated for brevity ...
```
**EN:** This method implements `prefill_text(input_ids: ..., position_ids: ..., past_key_values: ..., lm_spk_emb_last_hidden_states: ...=...)` and Prefill a chunk of new text tokens in streaming setting. Decorators: torch.inference_mode().
**CN:** 这个方法实现了 `prefill_text(input_ids: ..., position_ids: ..., past_key_values: ..., lm_spk_emb_last_hidden_states: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：torch.inference_mode()。

### Lines 725-783: Method `ConditionalChatTTS.prefill_audio_ids`
```python
    @torch.inference_mode()
    def prefill_audio_ids(
        self,
        input_ids: torch.Tensor,
        past_key_values: List[Tuple[torch.Tensor, torch.Tensor]],
        streaming_tts_text_mask=None,
        add_audio_bos: bool = True,
    ):
        """Prefill a chunk of audio ids to the model. Used in sliding-window long audio generation.
        Specifically, prefill many audio ids (typically from last window) to the model in the new window.

        Args:
            input_ids (torch.Tensor): (1, seq_len, num_vq) Audio input token ids.
            past_key_values (List[Tuple[torch.Tensor, torch.Tensor]]): Past key values for attention mechanism.
        """
        assert input_ids.shape[0] == 1
        assert past_key_values is not None

        code_emb = [self.emb_code[i](input_ids[:, :, i]) for i in range(self.num_vq)]
        inputs_embeds = torch.stack(code_emb, 3).sum(3)  # [1,seq_len,768]
        input_len = input_ids.shape[1]

        if add_audio_bos:
            narrowed_input_ids = torch.tensor(
                [[self.audio_bos_token_id]], dtype=torch.long, device=self.device
            )
            bos_inputs_embeds = self.emb_text(narrowed_input_ids)
            inputs_embeds = torch.cat([bos_inputs_embeds, inputs_embeds], dim=1)
            input_len += 1

        past_key_values_length = past_key_values[0][0].shape[2]
        position_ids = torch.arange(
            past_key_values_length,
            past_key_values_length + input_len,
            dtype=torch.long,
            device=self.device,
        ).unsqueeze(0)

        cache_position = position_ids.clone()
        causal_mask = make_streaming_chunk_mask_generation(
# ... truncated for brevity ...
```
**EN:** This method implements `prefill_audio_ids(input_ids: ..., past_key_values: ..., streaming_tts_text_mask=..., add_audio_bos: ...=...)` and Prefill a chunk of audio ids to the model. Decorators: torch.inference_mode().
**CN:** 这个方法实现了 `prefill_audio_ids(input_ids: ..., past_key_values: ..., streaming_tts_text_mask=..., add_audio_bos: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：torch.inference_mode()。

### Lines 785-1051: Method `ConditionalChatTTS.generate`
```python
    @torch.inference_mode()
    def generate(
        self,
        input_ids: torch.Tensor,
        past_key_values: List[Tuple[torch.Tensor, torch.Tensor]],
        temperature: torch.Tensor,
        eos_token: Union[int, torch.Tensor],
        streaming_tts_text_mask=None,
        force_no_stop=False,
        min_new_token=10,
        max_new_token=50,
        logits_warpers: Optional[List[LogitsWarper]] = None,
        logits_processors: Optional[
            List[CustomRepetitionPenaltyLogitsProcessorRepeat]
        ] = None,
        show_tqdm=False,
    ):
        """Generate audio codes in streaming setting or non-streaming setting.
        Specifically speaking, generate audio codes when not all text tokens are prefilled.

        Always pass a valid `past_key_values` to the method. The method does not do `prefill` by itself. It relies on `prefill_text` method to provide valid `past_key_values`. Please refer to docstring of this class for more details.

        In this method, we borrowed a lot of codes from `https://github.com/2noise/ChatTTS/blob/main/ChatTTS/model/gpt.py`.

        Args:
            input_ids (torch.Tensor): Input token ids.
            past_key_values (List[Tuple[torch.Tensor, torch.Tensor]]): Past key values for attention mechanism.
            temperature (torch.Tensor): Temperature for sampling.
            eos_token (Union[int, torch.Tensor]): End of sequence token.
            streaming_tts_text_mask (Optional[torch.Tensor], optional): Mask for streaming TTS text. Defaults to None.
            max_new_token (int, optional): Maximum number of new tokens to generate. Defaults to 50.
            logits_warpers (List[LogitsWarper], optional): List of logits warpers. Defaults to [].
            logits_processors (List[CustomRepetitionPenaltyLogitsProcessorRepeat], optional): List of logits processors. Defaults to [].
            show_tqdm (bool, optional): Whether to show progress bar. Defaults to True.

        Returns:
            GenerationOutputs: Generation outputs.
        """

        # We only support batch size `1` for now
# ... truncated for brevity ...
```
**EN:** This method implements `generate(input_ids: ..., past_key_values: ..., temperature: ..., eos_token: ..., streaming_tts_text_mask=..., force_no_stop=..., ...)` and Generate audio codes in streaming setting or non-streaming setting. Decorators: torch.inference_mode().
**CN:** 这个方法实现了 `generate(input_ids: ..., past_key_values: ..., temperature: ..., eos_token: ..., streaming_tts_text_mask=..., force_no_stop=..., ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：torch.inference_mode()。

### Lines 1053-1088: Method `ConditionalChatTTS.decode_to_mel_specs`
```python
    @torch.inference_mode()
    def decode_to_mel_specs(
        self,
        result_list: List[torch.Tensor],
    ):
        """Decode discrete audio codes to mel spectrograms.

        Borrowed from `https://github.com/2noise/ChatTTS/blob/main/ChatTTS/core.py`

        Args:
            result_list (List[torch.Tensor]): Audio codes output from `generate`.

        Returns:
            torch.Tensor: Mel spectrograms.
        """

        decoder = self.dvae
        max_x_len = -1
        if len(result_list) == 0:
            return np.array([], dtype=np.float32)
        for result in result_list:
            if result.size(0) > max_x_len:
                max_x_len = result.size(0)
        batch_result = torch.zeros(
            (len(result_list), result_list[0].size(1), max_x_len),
            dtype=result_list[0].dtype,
            device=result_list[0].device,
        )
        for i in range(len(result_list)):
            src = result_list[i]
            batch_result[i].narrow(1, 0, src.size(0)).copy_(src.permute(1, 0))
            del src

        mel_specs = decoder(batch_result)
        del batch_result
        return mel_specs
```
**EN:** This method implements `decode_to_mel_specs(result_list: ...)` and Decode discrete audio codes to mel spectrograms. Decorators: torch.inference_mode().
**CN:** 这个方法实现了 `decode_to_mel_specs(result_list: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：torch.inference_mode()。

### Lines 1092-1092: Class `MiniCPMWhisperEncoderLayer` overview
```python
class MiniCPMWhisperEncoderLayer(nn.Module):
```
**EN:** Defines `MiniCPMWhisperEncoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMWhisperEncoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 1093-1109: Method `MiniCPMWhisperEncoderLayer.__init__`
```python
    def __init__(self, config: WhisperConfig, layer_idx: int = None):
        super().__init__()
        self.embed_dim = config.d_model
        self.self_attn = WhisperAttention(
            embed_dim=self.embed_dim,
            num_heads=config.encoder_attention_heads,
            dropout=config.attention_dropout,
            config=config,
            layer_idx=layer_idx,
        )
        self.self_attn_layer_norm = nn.LayerNorm(self.embed_dim)
        self.dropout = config.dropout
        self.activation_fn = ACT2FN[config.activation_function]
        self.activation_dropout = config.activation_dropout
        self.fc1 = nn.Linear(self.embed_dim, config.encoder_ffn_dim)
        self.fc2 = nn.Linear(config.encoder_ffn_dim, self.embed_dim)
        self.final_layer_norm = nn.LayerNorm(self.embed_dim)
```
**EN:** This method implements `__init__(config: ..., layer_idx: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_idx: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1111-1183: Method `MiniCPMWhisperEncoderLayer.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        attention_mask: torch.Tensor,
        layer_head_mask: torch.Tensor,
        output_attentions: bool = False,
        past_key_values: Optional[EncoderDecoderCache] = None,
        use_cache: Optional[bool] = False,
    ) -> torch.Tensor:
        r"""
        Args:
            hidden_states (`torch.FloatTensor` of shape `(batch_size, seq_len, embed_dim)`):
                Hidden states to be fed into the encoder layer.
            attention_mask (`torch.FloatTensor` of shape `(batch_size, 1, tgt_len, src_len)`):
                Attention mask where padding elements are indicated by large negative values.
            layer_head_mask (`torch.FloatTensor` of shape `(encoder_attention_heads,)`):
                Mask to nullify selected heads of the attention modules.
            output_attentions (`bool`, *optional*):
                Whether or not to return the attention weights.
            past_key_values (`EncoderDecoderCache`, *optional*):
                Past key-value pairs used for incremental decoding.
            use_cache (`bool`, *optional*):
                Whether or not to return updated `past_key_values` for caching.

        Returns:
            A tuple of shape `(hidden_states, optional(attn_weights), optional(past_key_values))`.
        """
        residual = hidden_states
        hidden_states = self.self_attn_layer_norm(hidden_states)
        # TODO (lifuhuang): confirmed with Mick that the logic for past_key_values is copied from minicpmo official code,
        # currently we are not using past_key_values at all. We need to redesign the caching logic when we support streaming
        # in the future.
        hidden_states, attn_weights = self.self_attn(
            hidden_states=hidden_states,
            attention_mask=attention_mask,
            layer_head_mask=layer_head_mask,
            output_attentions=output_attentions,
            past_key_value=past_key_values,
        )
        hidden_states = nn.functional.dropout(
# ... truncated for brevity ...
```
**EN:** This method implements `forward(hidden_states: ..., attention_mask: ..., layer_head_mask: ..., output_attentions: ...=..., past_key_values: ...=..., use_cache: ...=...)` and Args:.
**CN:** 这个方法实现了 `forward(hidden_states: ..., attention_mask: ..., layer_head_mask: ..., output_attentions: ...=..., past_key_values: ...=..., use_cache: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1187-1188: Class `MiniCPMWhisperEncoder` overview
```python
class MiniCPMWhisperEncoder(WhisperEncoder):
```
**EN:** Defines `MiniCPMWhisperEncoder` as a reusable runtime type derived from WhisperEncoder. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMWhisperEncoder`，其继承关系为 WhisperEncoder。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 1189-1196: Method `MiniCPMWhisperEncoder.__init__`
```python
    def __init__(self, config: WhisperConfig):
        super().__init__(config)
        self.layers = nn.ModuleList(
            [
                MiniCPMWhisperEncoderLayer(config, layer_idx=i)
                for i in range(config.encoder_layers)
            ]
        )
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1198-1403: Method `MiniCPMWhisperEncoder.forward`
```python
    def forward(
        self,
        input_features,
        attention_mask=None,
        head_mask=None,
        output_attentions=None,
        output_hidden_states=None,
        return_dict=None,
        past_key_values: Optional[EncoderDecoderCache] = None,
        use_cache: Optional[bool] = None,
    ):
        r"""
        Forward pass of the Whisper encoder.

        Args:
            input_features (`torch.FloatTensor` of shape `(batch_size, feature_size, sequence_length)`):
                Float values of log-mel features extracted from the raw audio waveform. Typically generated
                by a feature extractor (e.g., `WhisperFeatureExtractor`) that processes `.flac` or `.wav`
                files into padded 2D mel spectrogram frames. These features are projected via convolution layers
                (`conv1` and `conv2`) and then transformed into embeddings for the encoder.

            attention_mask (`torch.Tensor`, *optional*):
                Not used by Whisper for masking `input_features`, but included for API compatibility with
                other models. If provided, it is simply ignored within the model. By default, Whisper
                effectively ignores silence in the input log-mel spectrogram.

            head_mask (`torch.Tensor` of shape `(encoder_layers, encoder_attention_heads)`, *optional*):
                Mask to nullify selected attention heads. The elements should be either 1 or 0, where:
                - 1 indicates the head is **not masked**,
                - 0 indicates the head is **masked** (i.e., the attention head is dropped).

            output_attentions (`bool`, *optional*):
                Whether or not to return the attention tensors of all encoder layers. If set to `True`, the
                returned tuple (or `BaseModelOutputWithPast`) will contain an additional element with
                attention weights for each encoder layer.

            output_hidden_states (`bool`, *optional*):
                Whether or not to return the hidden states of all layers. If set to `True`, the returned
                tuple (or `BaseModelOutputWithPast`) will contain a tuple of hidden states, including the
                initial embedding output as well as the outputs of each layer.
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_features, attention_mask=..., head_mask=..., output_attentions=..., output_hidden_states=..., return_dict=..., ...)` and Forward pass of the Whisper encoder.
**CN:** 这个方法实现了 `forward(input_features, attention_mask=..., head_mask=..., output_attentions=..., output_hidden_states=..., return_dict=..., ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1406-1406: Class `MultiModalProjector` overview
```python
class MultiModalProjector(nn.Module):
```
**EN:** Defines `MultiModalProjector` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MultiModalProjector`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 1407-1411: Method `MultiModalProjector.__init__`
```python
    def __init__(self, in_dim, out_dim):
        super().__init__()
        self.linear1 = nn.Linear(in_features=in_dim, out_features=out_dim, bias=True)
        self.relu = nn.ReLU()
        self.linear2 = nn.Linear(in_features=out_dim, out_features=out_dim, bias=True)
```
**EN:** This method implements `__init__(in_dim, out_dim)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(in_dim, out_dim)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1413-1416: Method `MultiModalProjector.forward`
```python
    def forward(self, audio_features):
        hidden_states = self.relu(self.linear1(audio_features))
        hidden_states = self.linear2(hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(audio_features)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(audio_features)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 1419-1419: Class `MiniCPMO` overview
```python
class MiniCPMO(MiniCPMBaseModel):
```
**EN:** Defines `MiniCPMO` as a reusable runtime type derived from MiniCPMBaseModel. The class groups 16 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiniCPMO`，其继承关系为 MiniCPMBaseModel。这个类组织了 16 个方法，用于实现模型相关行为。

### Lines 1420-1460: Method `MiniCPMO.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
    ) -> None:
        super().__init__(config=config, quant_config=quant_config)

        self.llm = self.init_llm(config=config, quant_config=quant_config)

        self.embed_dim = self.llm.config.hidden_size

        # init vision module
        if self.config.init_vision:
            # print("vision-understanding enabled")
            self.vpm = self.init_vision_module(config=config, quant_config=quant_config)
            self.vision_dim = self.vpm.embed_dim
            self.resampler = self.init_resampler(self.embed_dim, self.vision_dim)

        # init audio module
        self.config.init_audio = True
        if self.config.init_audio:
            # print("audio-understanding enabled")
            self.apm = self.init_audio_module()
            audio_output_dim = int(self.apm.config.encoder_ffn_dim // 4)
            self.audio_avg_pooler = nn.AvgPool1d(
                self.config.audio_pool_step, stride=self.config.audio_pool_step
            )
            self.audio_projection_layer = MultiModalProjector(
                in_dim=audio_output_dim, out_dim=self.embed_dim
            )
            self.audio_encoder_layer = -1

        # init tts module
        self.config.init_tts = False
        logger.info("TTS is disabled for now")
        if self.config.init_tts:
            # print("tts enabled")
            assert (
                _tts_deps
            ), "please make sure vector_quantize_pytorch and vocos are installed."
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1462-1464: Method `MiniCPMO.init_tts_module`
```python
    def init_tts_module(self):
        model = ConditionalChatTTS(self.config.tts_config)
        return model
```
**EN:** This method implements `init_tts_module()` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_tts_module()`，其作用是执行后续运行时所需的初始化工作。

### Lines 1466-1468: Method `MiniCPMO.init_audio_module`
```python
    def init_audio_module(self):
        model = MiniCPMWhisperEncoder(self.config.audio_config)
        return model
```
**EN:** This method implements `init_audio_module()` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_audio_module()`，其作用是执行后续运行时所需的初始化工作。

### Lines 1470-1476: Method `MiniCPMO.init_llm`
```python
    def init_llm(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        return Qwen2ForCausalLM(config=config, quant_config=quant_config, prefix=prefix)
```
**EN:** This method implements `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_llm(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1478-1497: Method `MiniCPMO.init_vision_module`
```python
    def init_vision_module(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig],
        prefix: str = "",
    ):
        if self.config._attn_implementation == "flash_attention_2":
            self.config.vision_config._attn_implementation = "flash_attention_2"
        else:
            self.config.vision_config._attn_implementation = "eager"
        model = Idefics2VisionTransformer(
            config=config.vision_config, quant_config=quant_config, prefix=prefix
        )
        if self.config.drop_vision_last_layer:
            model.encoder.layers = model.encoder.layers[:-1]

        setattr(model, "embed_dim", model.embeddings.embed_dim)
        setattr(model, "patch_size", model.embeddings.patch_size)

        return model
```
**EN:** This method implements `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_vision_module(config: ..., quant_config: ..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1499-1517: Method `MiniCPMO.init_resampler`
```python
    def init_resampler(
        self,
        embed_dim: int,
        vision_dim: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> nn.Module:
        with set_default_torch_dtype(torch.float16):
            # The resampler in 2.6 remains consistent with the one in 2.5.
            resampler = Resampler2_5(
                num_queries=self.config.query_num,
                embed_dim=embed_dim,
                num_heads=embed_dim // 128,
                kv_dim=vision_dim,
                quant_config=quant_config,
                prefix=prefix,
            )

        return resampler.to(device="cuda", dtype=torch.get_default_dtype())
```
**EN:** This method implements `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `init_resampler(embed_dim: ..., vision_dim: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 1519-1536: Method `MiniCPMO.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], mm_input: MultimodalInputs):
        # Get all special token IDs
        im_start_id: int = mm_input.im_start_id
        im_end_id: int = mm_input.im_end_id
        slice_start_id: int = mm_input.slice_start_id
        slice_end_id: int = mm_input.slice_end_id

        data_token_pairs = [
            (im_start_id, im_end_id),
            (slice_start_id, slice_end_id),
            (mm_input.audio_start_id, mm_input.audio_end_id),
        ]
        data_start_token_ids = [im_start_id, mm_input.audio_start_id]
        pattern = MultiModalityDataPaddingPatternTokenPairs(
            data_token_pairs=data_token_pairs, data_start_token_ids=data_start_token_ids
        )

        return pattern.pad_input_tokens(input_ids, mm_input)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., mm_input: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiniCPMO`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., mm_input: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiniCPMO` 内部调用。

### Lines 1538-1548: Method `MiniCPMO._get_feat_extract_output_lengths`
```python
    def _get_feat_extract_output_lengths(self, input_lengths: torch.LongTensor):
        """
        Computes the output length of the convolutional layers and the output length of the audio encoder
        """
        input_lengths_after_cnn = (input_lengths - 1) // 2 + 1
        input_lengths_after_pooling = (
            input_lengths_after_cnn - self.config.audio_pool_step
        ) // self.config.audio_pool_step + 1
        input_lengths_after_pooling = input_lengths_after_pooling.to(dtype=torch.int32)

        return input_lengths_after_cnn, input_lengths_after_pooling
```
**EN:** This method implements `_get_feat_extract_output_lengths(input_lengths: ...)` and Computes the output length of the convolutional layers and the output length of the audio encoder.
**CN:** 这个方法实现了 `_get_feat_extract_output_lengths(input_lengths: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1550-1615: Method `MiniCPMO.get_audio_embedding_streaming`
```python
    def get_audio_embedding_streaming(self, items: List[MultimodalDataItem]):
        r"""
        Extract audio embeddings in a streaming manner using cached key-value pairs.

        This method processes incoming audio features incrementally and stores/updates `past_key_values`
        for faster inference on subsequent audio frames. It only supports batch_size=1 and is intended
        for streaming scenarios.

        Returns:
            List[List[torch.Tensor]]: audio embeddings
        """
        wavforms = flatten_nested_list([item.feature for item in items if item.feature])
        # list, [[x1, x2], [y1], [z1]]
        audio_feature_lens_raw = flatten_nested_list(
            [item.audio_feature_lens for item in items if item.audio_feature_lens]
        )

        # exist audio
        if len(wavforms) > 0:
            audio_feature_lens = torch.hstack(audio_feature_lens_raw)
            batch_size, _, max_mel_seq_len = wavforms.shape
            assert batch_size == 1
            max_seq_len = (max_mel_seq_len - 1) // 2 + 1

            if self.audio_past_key_values is not None:
                cache_length = self.audio_past_key_values[0][0].shape[2]
                apm_max_len = self.apm.embed_positions.weight.shape[0]
                if cache_length + max_seq_len >= apm_max_len:
                    logger.warning(
                        f"audio_past_key_values length {cache_length + max_seq_len} exceed {apm_max_len}, reset."
                    )
                    self.audio_past_key_values = None

            audio_outputs = self.apm(
                wavforms, past_key_values=self.audio_past_key_values, use_cache=True
            )
            audio_states = (
                audio_outputs.last_hidden_state
            )  # [:, :audio_feat_lengths, :]
            self.audio_past_key_values = audio_outputs.past_key_values
# ... truncated for brevity ...
```
**EN:** This method implements `get_audio_embedding_streaming(items: ...)` and Extract audio embeddings in a streaming manner using cached key-value pairs.
**CN:** 这个方法实现了 `get_audio_embedding_streaming(items: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1617-1648: Method `MiniCPMO.subsequent_chunk_mask`
```python
    def subsequent_chunk_mask(
        self,
        size: int,
        chunk_size: int,
        num_left_chunks: int = -1,
        device: torch.device = torch.device("cpu"),
        num_lookhead: int = 0,
    ) -> torch.Tensor:
        """Create mask for subsequent steps (size, size) with chunk size,
        this is for streaming encoder

        Args:
            size (int): size of mask
            chunk_size (int): size of chunk
            num_left_chunks (int): number of left chunks
                <0: use full chunk
                >=0: use num_left_chunks
            device (torch.device): "cpu" or "cuda" or torch.Tensor.device

        Returns:
            torch.Tensor: mask

        """
        ret = torch.zeros(size, size, device=device, dtype=torch.bool)
        for i in range(size):
            if num_left_chunks < 0:
                start = 0
            else:
                start = max((i // chunk_size - num_left_chunks) * chunk_size, 0)
            ending = min((i // chunk_size + 1) * chunk_size + num_lookhead, size)
            ret[i, start:ending] = True
        return ret
```
**EN:** This method implements `subsequent_chunk_mask(size: ..., chunk_size: ..., num_left_chunks: ...=..., device: ...=..., num_lookhead: ...=...)` and Create mask for subsequent steps (size, size) with chunk size,.
**CN:** 这个方法实现了 `subsequent_chunk_mask(size: ..., chunk_size: ..., num_left_chunks: ...=..., device: ...=..., num_lookhead: ...=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1650-1772: Method `MiniCPMO.get_audio_embedding`
```python
    def get_audio_embedding(self, items: List[MultimodalDataItem], chunk_length=-1):
        r"""
        Extract full audio embeddings with optional chunk-based attention.

        This method computes embeddings for all audio frames at once, either using full attention (when
        `chunk_length` is -1) or chunk-based attention (when `chunk_length` is a positive number). It does
        not use key-value caching and is suitable for non-streaming inference.

        Args:
            chunk_length (int, optional): Determines whether to use full attention (-1) or chunk-based
                attention (>0) during embedding computation.

        Returns:
            List[List[torch.Tensor]]: audio embeddings
        """
        # (bs, 80, frames) or [], multi audios need filled in advance
        wavforms = flatten_nested_list([item.feature for item in items if item.feature])
        # list, [[x1, x2], [y1], [z1]]
        audio_feature_lens_raw = flatten_nested_list(
            [item.audio_feature_lens for item in items if item.audio_feature_lens]
        )

        # Ensure audio_feature_lens_raw is properly formatted as [[tensor], [tensor], ...]
        if audio_feature_lens_raw:
            if isinstance(audio_feature_lens_raw[0], torch.Tensor):
                # Flat list of tensors, wrap each in a list
                audio_feature_lens_raw = [[lens] for lens in audio_feature_lens_raw]
            elif isinstance(audio_feature_lens_raw[0], list):
                # Already nested, ensure all elements are properly formatted
                # Flatten if needed
                flattened = []
                for item in audio_feature_lens_raw:
                    if isinstance(item, list):
                        flattened.extend(item)
                    else:
                        flattened.append(item)
                audio_feature_lens_raw = [
                    [item] if not isinstance(item, list) else item for item in flattened
                ]

# ... truncated for brevity ...
```
**EN:** This method implements `get_audio_embedding(items: ..., chunk_length=...)` and Extract full audio embeddings with optional chunk-based attention.
**CN:** 这个方法实现了 `get_audio_embedding(items: ..., chunk_length=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1774-1780: Method `MiniCPMO.get_audio_feature`
```python
    def get_audio_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        embedding = self.get_omni_embedding(
            items=items,
            chunk_length=self.config.audio_chunk_length,
            stream_input=False,
        )
        return embedding
```
**EN:** This method implements `get_audio_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_audio_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1782-1804: Method `MiniCPMO.get_omni_embedding`
```python
    def get_omni_embedding(
        self,
        items: List[MultimodalDataItem],
        chunk_length=-1,
        stream_input=False,
    ):
        """
        Args:
            chunk_length: whisper use full attention or chunk attention
            stream_input: use streaming audio embedding
        Returns:
            final embeddings with audio feature
        """

        if stream_input:
            audio_embeddings = self.get_audio_embedding_streaming(items)
        else:
            audio_embeddings = self.get_audio_embedding(items, chunk_length)
        bs = len(audio_embeddings)
        # batch size
        audio_embs = torch.cat(flatten_nested_list(audio_embeddings), dim=0)

        return audio_embs
```
**EN:** This method implements `get_omni_embedding(items: ..., chunk_length=..., stream_input=...)` and Args:.
**CN:** 这个方法实现了 `get_omni_embedding(items: ..., chunk_length=..., stream_input=...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 1806-1847: Method `MiniCPMO.get_image_feature`
```python
    def get_image_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        if items and items[0].format == MultimodalInputFormat.PRECOMPUTED_EMBEDDING:
            result = torch.cat([item.feature for item in items])
            return result.reshape(-1, result.shape[-1])

        # list of tensors
        pixel_values = flatten_nested_list([item.feature for item in items])
        tgt_sizes = torch.stack(
            flatten_nested_list([item.tgt_size for item in items]), dim=0
        )
        assert len(pixel_values) == tgt_sizes.shape[0]

        device = self.vpm.embeddings.position_embedding.weight.device
        dtype = self.vpm.embeddings.position_embedding.weight.dtype
        all_pixel_values_lst = [
            i.flatten(end_dim=1).permute(1, 0) for i in pixel_values
        ]

        max_patches = (tgt_sizes[:, 0] * tgt_sizes[:, 1]).max().item()
        assert isinstance(max_patches, int)
        all_pixel_values = torch.nn.utils.rnn.pad_sequence(
            all_pixel_values_lst, batch_first=True, padding_value=0.0
        )

        B, L, _ = all_pixel_values.shape
        all_pixel_values = all_pixel_values.permute(0, 2, 1).reshape(B, 3, -1, L)
        patch_attn_mask = torch.zeros(
            (B, 1, max_patches), dtype=torch.bool, device=device
        )

        tgt_sizes_tensor = tgt_sizes.clone().to(device=patch_attn_mask.device)
        mask_shapes = tgt_sizes_tensor[:, 0] * tgt_sizes_tensor[:, 1]
        patch_attn_mask[:, 0, :] = torch.arange(
            patch_attn_mask.size(2), device=patch_attn_mask.device
        ).unsqueeze(0) < mask_shapes.unsqueeze(1)

        vision_embedding = self.vpm(
            all_pixel_values.type(dtype),
            patch_attention_mask=patch_attn_mask,
            tgt_sizes=tgt_sizes,
# ... truncated for brevity ...
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1849-1864: Method `MiniCPMO.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        **kwargs: Any,
    ) -> torch.Tensor:

        hidden_states = general_mm_embed_routine(
            input_ids=input_ids,
            forward_batch=forward_batch,
            language_model=self.llm,
            multimodal_model=self,
            positions=positions,
        )
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 1866-1943: Method `MiniCPMO.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        params_dict = dict(self.named_parameters())
        for name, loaded_weight in weights:

            if "rotary_emb.inv_freq~" in name or "projector" in name:
                continue
            if "rotary_emb.cos_cached" in name or "rotary_emb.sin_cached" in name:
                # Models trained using ColossalAI may include these tensors in
                # the checkpoint. Skip them.
                continue

            # For weight_norm parametrization, handle both old and new formats
            if self.config.init_tts and "tts" in name:
                # Handle loading from older checkpoints with weight_g/weight_v format
                if ".weight_g" in name or ".weight_v" in name:
                    name = name.replace(
                        ".weight_g", ".parametrizations.weight.original0"
                    )
                    name = name.replace(
                        ".weight_v", ".parametrizations.weight.original1"
                    )
                elif ".weight" in name and name not in params_dict:
                    param_name = name.replace(
                        ".weight", ".parametrizations.weight.original0"
                    )
                    if param_name in params_dict:
                        name = param_name

            # adapt to VisionAttention
            if "vpm" in name:
                name = name.replace(r"self_attn.out_proj", r"self_attn.proj")
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1946-1946: Top-level assign
```python
EntryClass = [MiniCPMO]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `math`
- `dataclasses.dataclass`
- `typing.Any`
- `typing.Iterable`
- `typing.List`
- `typing.Literal`
- `typing.Optional`
- `typing.Tuple`
- `typing.Union`
- `numpy`
- `torch`
- `torch.nn.functional`
- `torch.nn.utils.parametrize`
- `torch.types`
- `torch.nn`
- `torch.nn.utils.parametrizations`
- `tqdm.tqdm`
- `transformers.LlamaConfig`
- `transformers.LlamaModel`
- `transformers.PretrainedConfig`
- `transformers.PreTrainedModel`
- `transformers.activations.ACT2FN`
- `transformers.cache_utils.DynamicCache`
- `transformers.cache_utils.EncoderDecoderCache`
- `transformers.modeling_outputs.BaseModelOutputWithPast`
- `transformers.modeling_outputs.ModelOutput`
- `transformers.models.whisper.modeling_whisper.WhisperAttention`
- `transformers.models.whisper.modeling_whisper.WhisperConfig`
- `transformers.models.whisper.modeling_whisper.WhisperEncoder`
- `sglang.srt.layers.quantization.QuantizationConfig`
- `sglang.srt.managers.mm_utils.MultiModalityDataPaddingPatternTokenPairs`
- `sglang.srt.managers.mm_utils.general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.managers.schedule_batch.MultimodalInputFormat`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.managers.schedule_batch.flatten_nested_list`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.utils.set_default_torch_dtype`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.idefics2.Idefics2VisionTransformer`
- `sglang.srt.models.minicpmv.MiniCPMBaseModel`
- `sglang.srt.models.minicpmv.Resampler2_5`
- `sglang.srt.models.qwen2.Qwen2ForCausalLM`
- `sglang.srt.utils.logger`
- `transformers.LogitsWarper`
- `vector_quantize_pytorch.GroupedResidualFSQ`
