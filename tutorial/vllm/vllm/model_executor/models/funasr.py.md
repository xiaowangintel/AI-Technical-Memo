# funasr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/funasr.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for funasr, including encoder/decoder glue and vLLM runtime adaptation. / 面向 funasr 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-67)
```python
import math
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, cast

import torch
import torch.nn.functional as F
from torch import nn
from transformers import (
    BatchFeature,
    Qwen3Config,
)

from vllm.config import ModelConfig, SpeechToTextConfig, VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.config.speech_to_text import SpeechToTextParams
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.inputs import MultiModalDataDict, PromptType
from vllm.logger import init_logger
from vllm.model_executor.layers.activation import _ACTIVATION_REGISTRY
from vllm.model_executor.layers.attention.mm_encoder_attention import (
    MMEncoderAttention,
)
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.model_executor.models.whisper_utils import (
    ISO639_1_SUPPORTED_LANGS,
)
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import MultiModalDataItems, MultiModalDataParser
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
)
from vllm.transformers_utils.processor import cached_processor_from_config
from vllm.transformers_utils.processors.funasr import FunASRFeatureExtractor
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import (
    MultiModalEmbeddings,
    SupportsMultiModal,
    SupportsTranscription,
    _require_is_multimodal,
)
from .qwen3 import Qwen3Model
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    _merge_multimodal_embeddings,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `sequence_mask` (lines 72-80)
```python
def sequence_mask(lengths, maxlen=None, dtype=torch.float32, device=None):
    if maxlen is None:
        maxlen = lengths.max()
    row_vector = torch.arange(0, maxlen, 1).to(lengths.device)
    matrix = torch.unsqueeze(lengths, dim=-1)
    mask = row_vector < matrix
    mask = mask.detach()

    return mask.type(dtype).to(device) if device is not None else mask.type(dtype)
```
**EN:** Function `sequence_mask` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `sequence_mask` 封装了该模块中的一段可复用核心逻辑。

### Class `LayerNorm` (lines 83-91)
```python
class LayerNorm(torch.nn.LayerNorm):
    def __init__(self, nout, dim=-1):
        super().__init__(nout, eps=1e-12)
        self.dim = dim

    def forward(self, x: torch.Tensor):
        if self.dim == -1:
            return super().forward(x)
        return super().forward(x.transpose(self.dim, -1)).transpose(self.dim, -1)
```
**EN:** Class `LayerNorm` organizes related behavior for this model family or helper component. It inherits from torch.nn.LayerNorm. Key methods include __init__, forward.
**CN:** 类 `LayerNorm` 用于组织该模型族或辅助组件的相关行为。 它继承自 torch.nn.LayerNorm。 关键方法包括 __init__, forward。

### Method `LayerNorm.__init__` (lines 84-86)
```python
    def __init__(self, nout, dim=-1):
        super().__init__(nout, eps=1e-12)
        self.dim = dim
```
**EN:** Method `LayerNorm.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `LayerNorm.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `LayerNorm.forward` (lines 88-91)
```python
    def forward(self, x: torch.Tensor):
        if self.dim == -1:
            return super().forward(x)
        return super().forward(x.transpose(self.dim, -1)).transpose(self.dim, -1)
```
**EN:** Method `LayerNorm.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `LayerNorm.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `EncoderLayerSANM` (lines 94-142)
```python
class EncoderLayerSANM(nn.Module):
    def __init__(
        self,
        in_size: int,
        size: int,
        self_attn: nn.Module,
        feed_forward: nn.Module,
        normalize_before=True,
    ):
        super().__init__()
        self.self_attn = self_attn
        self.feed_forward = feed_forward
        self.norm1 = LayerNorm(in_size)
        self.norm2 = LayerNorm(size)
        self.in_size = in_size
        self.size = size
        self.normalize_before = normalize_before

    def forward(
        self,
        hidden_states: torch.Tensor,
        mask: torch.Tensor | None = None,
        cache=None,
        mask_shift_chunk=None,
        mask_att_chunk_encoder=None,
```
**EN:** Class `EncoderLayerSANM` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `EncoderLayerSANM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `EncoderLayerSANM.__init__` (lines 95-110)
```python
    def __init__(
        self,
        in_size: int,
        size: int,
        self_attn: nn.Module,
        feed_forward: nn.Module,
        normalize_before=True,
    ):
        super().__init__()
        self.self_attn = self_attn
        self.feed_forward = feed_forward
        self.norm1 = LayerNorm(in_size)
        self.norm2 = LayerNorm(size)
        self.in_size = in_size
        self.size = size
        self.normalize_before = normalize_before
```
**EN:** Method `EncoderLayerSANM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `EncoderLayerSANM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `EncoderLayerSANM.forward` (lines 112-142)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        mask: torch.Tensor | None = None,
        cache=None,
        mask_shift_chunk=None,
        mask_att_chunk_encoder=None,
    ):
        residual = hidden_states
        hidden_states = self.norm1(hidden_states)

        if self.in_size == self.size:
            hidden_states = residual + self.self_attn(
                hidden_states,
                mask,
                mask_shift_chunk=mask_shift_chunk,
                mask_att_chunk_encoder=mask_att_chunk_encoder,
            )
        else:
            hidden_states = self.self_attn(
                hidden_states,
                mask,
                mask_shift_chunk=mask_shift_chunk,
                mask_att_chunk_encoder=mask_att_chunk_encoder,
            )

        residual = hidden_states
        hidden_states = self.norm2(hidden_states)
        hidden_states = residual + self.feed_forward(hidden_states)

        return hidden_states, mask, cache, mask_shift_chunk, mask_att_chunk_encoder
```
**EN:** Method `EncoderLayerSANM.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `EncoderLayerSANM.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `MultiHeadedAttentionSANM` (lines 145-253)
```python
class MultiHeadedAttentionSANM(nn.Module):
    def __init__(
        self,
        n_head: int,
        in_feat: int,
        n_feat: int,
        kernel_size: int,
        sanm_shift: int = 0,
    ):
        super().__init__()
        assert n_feat % n_head == 0
        # We assume d_v always equals d_k
        self.d_k = n_feat // n_head
        self.h = n_head
        self.out_proj = ReplicatedLinear(
            input_size=n_feat,
            output_size=n_feat,
            bias=True,
        )
        self.linear_q_k_v = ReplicatedLinear(
            input_size=in_feat,
            output_size=n_feat * 3,
            bias=True,
        )
        self.attn = None
```
**EN:** Class `MultiHeadedAttentionSANM` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward_fsmn, forward_qkv, forward_attention, forward.
**CN:** 类 `MultiHeadedAttentionSANM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward_fsmn, forward_qkv, forward_attention, forward。

### Method `MultiHeadedAttentionSANM.__init__` (lines 146-179)
```python
    def __init__(
        self,
        n_head: int,
        in_feat: int,
        n_feat: int,
        kernel_size: int,
        sanm_shift: int = 0,
    ):
        super().__init__()
        assert n_feat % n_head == 0
        # We assume d_v always equals d_k
        self.d_k = n_feat // n_head
        self.h = n_head
        self.out_proj = ReplicatedLinear(
            input_size=n_feat,
            output_size=n_feat,
            bias=True,
        )
        self.linear_q_k_v = ReplicatedLinear(
            input_size=in_feat,
            output_size=n_feat * 3,
            bias=True,
        )
        self.attn = None

        self.fsmn_block = nn.Conv1d(
            n_feat, n_feat, kernel_size, stride=1, padding=0, groups=n_feat, bias=False
        )
        # padding
        left_padding = (kernel_size - 1) // 2
        if sanm_shift > 0:
            left_padding = left_padding + sanm_shift
        right_padding = kernel_size - 1 - left_padding
        self.pad_fn = nn.ConstantPad1d((left_padding, right_padding), 0.0)
```
**EN:** Method `MultiHeadedAttentionSANM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `MultiHeadedAttentionSANM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `MultiHeadedAttentionSANM.forward_fsmn` (lines 181-201)
```python
    def forward_fsmn(
        self,
        inputs: torch.Tensor,
        mask: torch.Tensor,
        mask_shift_chunk: torch.Tensor = None,
    ):
        b, t, d = inputs.size()
        if mask is not None:
            mask = torch.reshape(mask, (b, -1, 1))
            if mask_shift_chunk is not None:
                mask = mask * mask_shift_chunk
            inputs = inputs * mask

        x = inputs.transpose(1, 2)
        x = self.pad_fn(x)
        x = self.fsmn_block(x)
        x = x.transpose(1, 2)
        x += inputs
        if mask is not None:
            x = x * mask
        return x
```
**EN:** Method `MultiHeadedAttentionSANM.forward_fsmn` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `MultiHeadedAttentionSANM.forward_fsmn` 封装了该模块中的一段可复用核心逻辑。

### Class `SinusoidalPositionEncoder` (lines 256-290)
```python
class SinusoidalPositionEncoder(torch.nn.Module):
    def __init__(self, d_model=80):
        super().__init__()

    def encode(
        self,
        positions: torch.Tensor = None,
        depth: int = None,
        dtype: torch.dtype = torch.float32,
    ):
        batch_size = positions.size(0)
        positions = positions.type(dtype)
        device = positions.device
        log_timescale_increment = torch.log(
            torch.tensor([10000], dtype=dtype, device=device)
        ) / (depth / 2 - 1)
        inv_timescales = torch.exp(
            torch.arange(depth / 2, device=device).type(dtype)
            * (-log_timescale_increment)
        )
        inv_timescales = torch.reshape(inv_timescales, [batch_size, -1])
        scaled_time = torch.reshape(positions, [1, -1, 1]) * torch.reshape(
            inv_timescales, [1, 1, -1]
        )
        encoding = torch.cat([torch.sin(scaled_time), torch.cos(scaled_time)], dim=2)
```
**EN:** Class `SinusoidalPositionEncoder` is a structural model block in the vLLM execution graph. It inherits from torch.nn.Module. Key methods include __init__, encode, forward.
**CN:** 类 `SinusoidalPositionEncoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 torch.nn.Module。 关键方法包括 __init__, encode, forward。

### Method `SinusoidalPositionEncoder.__init__` (lines 257-258)
```python
    def __init__(self, d_model=80):
        super().__init__()
```
**EN:** Method `SinusoidalPositionEncoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `SinusoidalPositionEncoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `SinusoidalPositionEncoder.encode` (lines 260-281)
```python
    def encode(
        self,
        positions: torch.Tensor = None,
        depth: int = None,
        dtype: torch.dtype = torch.float32,
    ):
        batch_size = positions.size(0)
        positions = positions.type(dtype)
        device = positions.device
        log_timescale_increment = torch.log(
            torch.tensor([10000], dtype=dtype, device=device)
        ) / (depth / 2 - 1)
        inv_timescales = torch.exp(
            torch.arange(depth / 2, device=device).type(dtype)
            * (-log_timescale_increment)
        )
        inv_timescales = torch.reshape(inv_timescales, [batch_size, -1])
        scaled_time = torch.reshape(positions, [1, -1, 1]) * torch.reshape(
            inv_timescales, [1, 1, -1]
        )
        encoding = torch.cat([torch.sin(scaled_time), torch.cos(scaled_time)], dim=2)
        return encoding.type(dtype)
```
**EN:** Method `SinusoidalPositionEncoder.encode` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `SinusoidalPositionEncoder.encode` 封装了该模块中的一段可复用核心逻辑。

### Class `SenseVoiceEncoderSmall` (lines 293-409)
```python
class SenseVoiceEncoderSmall(nn.Module):
    def __init__(
        self,
        input_size: int,
        output_size: int = 256,
        attention_heads: int = 4,
        linear_units: int = 2048,
        num_blocks: int = 6,
        tp_blocks: int = 0,
        attention_dropout_rate: float = 0.0,
        normalize_before: bool = True,
        kernel_size: int = 11,
        sanm_shift: int = 0,
        **kwargs,
    ):
        super().__init__()
        self._output_size = output_size
        self.embed = SinusoidalPositionEncoder()

        self.normalize_before = normalize_before

        positionwise_layer = PositionwiseFeedForward
        positionwise_layer_args = (
            output_size,
            linear_units,
```
**EN:** Class `SenseVoiceEncoderSmall` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, output_size, forward.
**CN:** 类 `SenseVoiceEncoderSmall` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, output_size, forward。

### Method `SenseVoiceEncoderSmall.__init__` (lines 294-373)
```python
    def __init__(
        self,
        input_size: int,
        output_size: int = 256,
        attention_heads: int = 4,
        linear_units: int = 2048,
        num_blocks: int = 6,
        tp_blocks: int = 0,
        attention_dropout_rate: float = 0.0,
        normalize_before: bool = True,
        kernel_size: int = 11,
        sanm_shift: int = 0,
        **kwargs,
    ):
        super().__init__()
        self._output_size = output_size
        self.embed = SinusoidalPositionEncoder()

        self.normalize_before = normalize_before

        positionwise_layer = PositionwiseFeedForward
        positionwise_layer_args = (
            output_size,
            linear_units,
        )

        encoder_selfattn_layer = MultiHeadedAttentionSANM
        encoder_selfattn_layer_args0 = (
            attention_heads,
            input_size,
            output_size,
            kernel_size,
            sanm_shift,
        )
        encoder_selfattn_layer_args = (
            attention_heads,
            output_size,
            output_size,
            kernel_size,
            sanm_shift,
        )

        self.encoders0 = nn.ModuleList(
            [
                EncoderLayerSANM(
                    input_size,
                    output_size,
                    encoder_selfattn_layer(*encoder_selfattn_layer_args0),
                    positionwise_layer(*positionwise_layer_args),
                )
                for i in range(1)
            ]
        )
        self.encoders = nn.ModuleList(
            [
                EncoderLayerSANM(
                    output_size,
                    output_size,
                    encoder_selfattn_layer(*encoder_selfattn_layer_args),
                    positionwise_layer(*positionwise_layer_args),
# ... truncated for analysis ...
```
**EN:** Method `SenseVoiceEncoderSmall.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `SenseVoiceEncoderSmall.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `SenseVoiceEncoderSmall.output_size` (lines 375-376)
```python
    def output_size(self) -> int:
        return self._output_size
```
**EN:** Method `SenseVoiceEncoderSmall.output_size` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `SenseVoiceEncoderSmall.output_size` 封装了该模块中的一段可复用核心逻辑。

### Class `PositionwiseFeedForward` (lines 412-431)
```python
class PositionwiseFeedForward(nn.Module):
    def __init__(self, idim: int, hidden_units: int):
        super().__init__()
        self.w_1 = ColumnParallelLinear(
            input_size=idim,
            output_size=hidden_units,
            bias=True,
        )
        self.w_2 = RowParallelLinear(
            input_size=hidden_units,
            output_size=idim,
            bias=True,
        )
        self.activation = _ACTIVATION_REGISTRY["relu"]

    def forward(self, hidden_states: torch.Tensor):
        hidden_states, _ = self.w_1(hidden_states)
        hidden_states = self.activation(hidden_states)
        hidden_states, _ = self.w_2(hidden_states)
        return hidden_states
```
**EN:** Class `PositionwiseFeedForward` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `PositionwiseFeedForward` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `PositionwiseFeedForward.__init__` (lines 413-425)
```python
    def __init__(self, idim: int, hidden_units: int):
        super().__init__()
        self.w_1 = ColumnParallelLinear(
            input_size=idim,
            output_size=hidden_units,
            bias=True,
        )
        self.w_2 = RowParallelLinear(
            input_size=hidden_units,
            output_size=idim,
            bias=True,
        )
        self.activation = _ACTIVATION_REGISTRY["relu"]
```
**EN:** Method `PositionwiseFeedForward.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `PositionwiseFeedForward.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `PositionwiseFeedForward.forward` (lines 427-431)
```python
    def forward(self, hidden_states: torch.Tensor):
        hidden_states, _ = self.w_1(hidden_states)
        hidden_states = self.activation(hidden_states)
        hidden_states, _ = self.w_2(hidden_states)
        return hidden_states
```
**EN:** Method `PositionwiseFeedForward.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `PositionwiseFeedForward.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `EncoderLayer` (lines 434-455)
```python
class EncoderLayer(nn.Module):
    def __init__(
        self,
        size: int,
        self_attn: nn.Module,
        feed_forward: nn.Module,
    ):
        super().__init__()
        self.self_attn = self_attn
        self.feed_forward = feed_forward
        self.norm1 = LayerNorm(size)
        self.norm2 = LayerNorm(size)

    def forward(self, hidden_states: torch.Tensor):
        residual = hidden_states
        hidden_states = self.norm1(hidden_states)
        hidden_states = residual + self.self_attn(hidden_states, None, None)
        residual = hidden_states
        hidden_states = self.norm2(hidden_states)
        hidden_states = residual + self.feed_forward(hidden_states)

        return hidden_states
```
**EN:** Class `EncoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `EncoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `EncoderLayer.__init__` (lines 435-445)
```python
    def __init__(
        self,
        size: int,
        self_attn: nn.Module,
        feed_forward: nn.Module,
    ):
        super().__init__()
        self.self_attn = self_attn
        self.feed_forward = feed_forward
        self.norm1 = LayerNorm(size)
        self.norm2 = LayerNorm(size)
```
**EN:** Method `EncoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `EncoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `EncoderLayer.forward` (lines 447-455)
```python
    def forward(self, hidden_states: torch.Tensor):
        residual = hidden_states
        hidden_states = self.norm1(hidden_states)
        hidden_states = residual + self.self_attn(hidden_states, None, None)
        residual = hidden_states
        hidden_states = self.norm2(hidden_states)
        hidden_states = residual + self.feed_forward(hidden_states)

        return hidden_states
```
**EN:** Method `EncoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `EncoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `FunASRAudioAttention` (lines 458-525)
```python
class FunASRAudioAttention(nn.Module):
    def __init__(
        self,
        num_heads: int,
        embed_dim: int,
        prefix: str = "",
    ):
        super().__init__()
        self.embed_dim = embed_dim
        self.num_heads = num_heads
        self.head_dim = self.embed_dim // self.num_heads
        tp_size = get_tensor_model_parallel_world_size()
        self.num_local_heads = self.num_heads // tp_size

        if (self.head_dim * self.num_heads) != self.embed_dim:
            raise ValueError(
                f"embed_dim must be divisible by num_heads (got `embed_dim`: "
                f"{self.embed_dim} and `num_heads`: {self.num_heads})."
            )

        self.scaling = self.head_dim**-0.5

        self.qkv = QKVParallelLinear(
            hidden_size=self.embed_dim,
            head_size=self.head_dim,
```
**EN:** Class `FunASRAudioAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `FunASRAudioAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `FunASRAudioAttention.__init__` (lines 459-500)
```python
    def __init__(
        self,
        num_heads: int,
        embed_dim: int,
        prefix: str = "",
    ):
        super().__init__()
        self.embed_dim = embed_dim
        self.num_heads = num_heads
        self.head_dim = self.embed_dim // self.num_heads
        tp_size = get_tensor_model_parallel_world_size()
        self.num_local_heads = self.num_heads // tp_size

        if (self.head_dim * self.num_heads) != self.embed_dim:
            raise ValueError(
                f"embed_dim must be divisible by num_heads (got `embed_dim`: "
                f"{self.embed_dim} and `num_heads`: {self.num_heads})."
            )

        self.scaling = self.head_dim**-0.5

        self.qkv = QKVParallelLinear(
            hidden_size=self.embed_dim,
            head_size=self.head_dim,
            total_num_heads=self.num_heads,
            total_num_kv_heads=self.num_heads,
            bias=True,
            prefix=f"{prefix}.qkv",
        )

        self.out_proj = RowParallelLinear(
            input_size=self.embed_dim,
            output_size=self.embed_dim,
            bias=True,
            prefix=f"{prefix}.out_proj",
        )

        self.attn = MMEncoderAttention(
            num_heads=self.num_local_heads,
            head_size=self.head_dim,
            scale=self.scaling,
        )
```
**EN:** Method `FunASRAudioAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `FunASRAudioAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `FunASRAudioAttention.forward` (lines 502-525)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: torch.Tensor,
        max_seqlen: torch.Tensor | None,
    ) -> torch.Tensor:
        bs, seq_length, _ = hidden_states.size()
        qkv, _ = self.qkv(hidden_states)
        q, k, v = qkv.chunk(3, dim=-1)
        q = q.view(bs, seq_length, -1, self.head_dim)
        k = k.view(bs, seq_length, -1, self.head_dim)
        v = v.view(bs, seq_length, -1, self.head_dim)

        attn_output = self.attn(
            query=q,
            key=k,
            value=v,
            cu_seqlens=cu_seqlens,
            max_seqlen=max_seqlen,
        )

        attn_output = attn_output.view(bs, seq_length, -1)
        output, _ = self.out_proj(attn_output)
        return output
```
**EN:** Method `FunASRAudioAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `FunASRAudioAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Transformer` (lines 528-595)
```python
class Transformer(nn.Module):
    def __init__(
        self,
        downsample_rate=2,
        encoder_dim=1280,
        llm_dim=4096,
        ffn_dim: int = 2048,
        prefix: str = "",
        **kwargs,
    ):
        super().__init__()
        self.k = downsample_rate
        self.encoder_dim = encoder_dim
        self.llm_dim = llm_dim
        self.linear1 = ColumnParallelLinear(
            input_size=self.encoder_dim * self.k,
            output_size=ffn_dim,
            bias=True,
        )
        self.relu = nn.ReLU()
        self.linear2 = RowParallelLinear(
            input_size=ffn_dim,
            output_size=self.llm_dim,
            bias=True,
        )
```
**EN:** Class `Transformer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Transformer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Transformer.__init__` (lines 529-572)
```python
    def __init__(
        self,
        downsample_rate=2,
        encoder_dim=1280,
        llm_dim=4096,
        ffn_dim: int = 2048,
        prefix: str = "",
        **kwargs,
    ):
        super().__init__()
        self.k = downsample_rate
        self.encoder_dim = encoder_dim
        self.llm_dim = llm_dim
        self.linear1 = ColumnParallelLinear(
            input_size=self.encoder_dim * self.k,
            output_size=ffn_dim,
            bias=True,
        )
        self.relu = nn.ReLU()
        self.linear2 = RowParallelLinear(
            input_size=ffn_dim,
            output_size=self.llm_dim,
            bias=True,
        )

        self.blocks = None
        if kwargs.get("n_layer", 2) > 0:
            self.blocks = nn.ModuleList(
                [
                    EncoderLayer(
                        llm_dim,
                        FunASRAudioAttention(
                            kwargs.get("attention_heads", 8),
                            llm_dim,
                            prefix=f"{prefix}.self_attn",
                        ),
                        PositionwiseFeedForward(
                            llm_dim,
                            llm_dim // 4,
                        ),
                    )
                    for _ in range(kwargs.get("n_layer", 2))
                ]
            )
```
**EN:** Method `Transformer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Transformer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Transformer.forward` (lines 574-595)
```python
    def forward(self, hidden_states: torch.Tensor, ilens: int = 0):
        max_len = max(ilens)
        hidden_states = hidden_states[:, :max_len, :]
        batch_size, seq_len, dim = hidden_states.size()
        chunk_num = (seq_len - 1) // self.k + 1
        pad_num = chunk_num * self.k - seq_len
        hidden_states = F.pad(hidden_states, (0, 0, 0, pad_num, 0, 0), value=0.0)
        seq_len = hidden_states.size(1)

        hidden_states = hidden_states.contiguous()
        hidden_states = hidden_states.view(batch_size, chunk_num, dim * self.k)
        hidden_states, _ = self.linear1(hidden_states)
        hidden_states = self.relu(hidden_states)
        hidden_states, _ = self.linear2(hidden_states)

        olens = None
        olens = (ilens - 1) // self.k + 1

        if self.blocks is not None:
            for layer, block in enumerate(self.blocks):
                hidden_states = block(hidden_states)
        return hidden_states, olens
```
**EN:** Method `Transformer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Transformer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `FunASRAudioInputs` (lines 598-617)
```python
class FunASRAudioInputs(TensorSchema):
    """
    Dimensions:
        - b: Batch size
        - nmb: Number of mel bins
        - t: Time frames (M)
    """

    input_features: Annotated[
        list[torch.Tensor] | None,
        TensorShape("b", "nmb", "t"),
    ]
    speech_lengths: Annotated[
        list[torch.Tensor] | None,
        TensorShape("b"),
    ]
    fake_token_lengths: Annotated[
        list[torch.Tensor] | None,
        TensorShape("b"),
    ]
```
**EN:** Class `FunASRAudioInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `FunASRAudioInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。
- **KV cache / KV 缓存**
  - **EN:** The runtime stores key/value state so decoding can reuse prior computation efficiently.
  - **CN:** 运行时会保存 key/value 状态，以便解码阶段高效复用历史计算。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, cast`
- **Third-party / 第三方**: `import torch`, `import torch.nn.functional as F`, `from torch import nn`, `from transformers import (`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import ModelConfig, SpeechToTextConfig, VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.config.speech_to_text import SpeechToTextParams`, `from vllm.distributed import get_tensor_model_parallel_world_size`, `from vllm.inputs import MultiModalDataDict, PromptType`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.activation import _ACTIVATION_REGISTRY`, `from vllm.model_executor.layers.attention.mm_encoder_attention import (`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.vocab_parallel_embedding import ParallelLMHead`, `from vllm.model_executor.model_loader.weight_utils import default_weight_loader`
