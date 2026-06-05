# granite_speech.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/granite_speech.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for granite_speech, including architecture wrappers and weight loading logic. / 面向推理的 granite_speech vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 27-73)
```python
import math
from collections.abc import Iterable, Mapping
from typing import Annotated

import torch
import torch.nn.functional as F
from torch import nn
from transformers import BatchFeature, PretrainedConfig

from vllm.config import CacheConfig, ModelConfig, SpeechToTextConfig, VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.config.speech_to_text import SpeechToTextParams
from vllm.inputs import MultiModalDataDict, PromptType, TokensPrompt
from vllm.model_executor.layers.linear import ColumnParallelLinear, RowParallelLinear
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import (
    AudioProcessorItems,
    MultiModalDataItems,
    MultiModalDataParser,
)
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
)
from vllm.sequence import IntermediateTensors
from vllm.tokenizers import cached_tokenizer_from_config
from vllm.transformers_utils.processor import cached_processor_from_config
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .blip2 import Blip2QFormerModel
from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
    SupportsTranscription,
)
from .utils import AutoWeightsLoader, init_vllm_registered_model, maybe_prefix
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 80-87)
```python
ISO639_1_SUPPORTED_LANGS = {
    "en": "English",
    "fr": "French",
    "de": "German",
    "ja": "Japanese",
    "pt": "Portuguese",
    "es": "Spanish",
}
```
**EN:** This block defines ISO639_1_SUPPORTED_LANGS, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 ISO639_1_SUPPORTED_LANGS，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Class `GraniteSpeechAudioInputs` (lines 91-109)
```python
class GraniteSpeechAudioInputs(TensorSchema):
    """
    Audio input features for Granite Speech model.

    Dimensions:
        - b: Batch size
        - fi: Number of input features from the Mel spectrogram.
        - fo: Number of output features, i.e. the embedding size.
        - 160: Fixed feature dimension for Mel spectrogram features
    """

    input_features: Annotated[torch.Tensor, TensorShape("b", "fi", 160)]
    """Audio input features."""

    input_features_mask: Annotated[torch.Tensor, TensorShape("b", "fo")]
    """Mask for variable length audio features."""

    audio_embed_sizes: Annotated[list[int], TensorShape("b")]
    """List of audio embedding sizes for each item in batch."""
```
**EN:** Class `GraniteSpeechAudioInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `GraniteSpeechAudioInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `GraniteSpeechMultiModalProcessingInfo` (lines 112-132)
```python
class GraniteSpeechMultiModalProcessingInfo(BaseProcessingInfo):
    def get_data_parser(self):
        feature_extractor = self.get_hf_processor().audio_processor

        return MultiModalDataParser(
            target_sr=feature_extractor.melspec_kwargs["sample_rate"],
            expected_hidden_size=self._get_expected_hidden_size(),
        )

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"audio": 1}

    # There is no limit to the maximum number of audio tokens that can be
    # encoded as features; we pick ~5000 as a number that is probably higher
    # than we would expect to encounter. The sequence of length
    # get_max_audio_len() produces get_max_audio_tokens().
    def get_max_audio_tokens(self):
        return 5001

    def get_max_audio_len(self):
        return 8000000
```
**EN:** Class `GraniteSpeechMultiModalProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_data_parser, get_supported_mm_limits, get_max_audio_tokens, get_max_audio_len.
**CN:** 类 `GraniteSpeechMultiModalProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_data_parser, get_supported_mm_limits, get_max_audio_tokens, get_max_audio_len。

### Method `GraniteSpeechMultiModalProcessingInfo.get_data_parser` (lines 113-119)
```python
    def get_data_parser(self):
        feature_extractor = self.get_hf_processor().audio_processor

        return MultiModalDataParser(
            target_sr=feature_extractor.melspec_kwargs["sample_rate"],
            expected_hidden_size=self._get_expected_hidden_size(),
        )
```
**EN:** Method `GraniteSpeechMultiModalProcessingInfo.get_data_parser` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GraniteSpeechMultiModalProcessingInfo.get_data_parser` 封装了该模块中的一段可复用核心逻辑。

### Method `GraniteSpeechMultiModalProcessingInfo.get_supported_mm_limits` (lines 121-122)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"audio": 1}
```
**EN:** Method `GraniteSpeechMultiModalProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GraniteSpeechMultiModalProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Class `GraniteSpeechMultiModalProcessor` (lines 136-210)
```python
class GraniteSpeechMultiModalProcessor(
    BaseMultiModalProcessor[GraniteSpeechMultiModalProcessingInfo]
):
    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        return dict(
            input_features=MultiModalFieldConfig.batched("audio"),
            audio_embed_sizes=MultiModalFieldConfig.batched("audio"),
        )

    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> list[PromptUpdate]:
        processor = self.info.get_hf_processor(**hf_processor_mm_kwargs)
        tokenizer = self.info.get_tokenizer()
        feature_extractor = processor.audio_processor
        vocab = tokenizer.get_vocab()

        # Use getattr with default to be compatible with transformers<4.48
```
**EN:** Class `GraniteSpeechMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[GraniteSpeechMultiModalProcessingInfo]. Key methods include _get_mm_fields_config, _get_prompt_updates, _call_hf_processor.
**CN:** 类 `GraniteSpeechMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[GraniteSpeechMultiModalProcessingInfo]。 关键方法包括 _get_mm_fields_config, _get_prompt_updates, _call_hf_processor。

### Class `GraniteSpeechDummyInputsBuilder` (lines 213-237)
```python
class GraniteSpeechDummyInputsBuilder(
    BaseDummyInputsBuilder[GraniteSpeechMultiModalProcessingInfo]
):
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_audios = mm_counts.get("audio", 0)
        audio_overrides = mm_options.get("audio")

        return {
            "audio": self._get_dummy_audios(
                length=self.info.get_max_audio_len(),
                num_audios=num_audios,
                overrides=audio_overrides,
            )
        }

    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_audios = mm_counts.get("audio", 0)
        hf_processor = self.info.get_hf_processor()
        audio_token = getattr(hf_processor, "audio_token", "<|audio|>")
        return audio_token * num_audios
```
**EN:** Class `GraniteSpeechDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[GraniteSpeechMultiModalProcessingInfo]. Key methods include get_dummy_mm_data, get_dummy_text.
**CN:** 类 `GraniteSpeechDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[GraniteSpeechMultiModalProcessingInfo]。 关键方法包括 get_dummy_mm_data, get_dummy_text。

### Method `GraniteSpeechDummyInputsBuilder.get_dummy_mm_data` (lines 216-231)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        num_audios = mm_counts.get("audio", 0)
        audio_overrides = mm_options.get("audio")

        return {
            "audio": self._get_dummy_audios(
                length=self.info.get_max_audio_len(),
                num_audios=num_audios,
                overrides=audio_overrides,
            )
        }
```
**EN:** Method `GraniteSpeechDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GraniteSpeechDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Method `GraniteSpeechDummyInputsBuilder.get_dummy_text` (lines 233-237)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_audios = mm_counts.get("audio", 0)
        hf_processor = self.info.get_hf_processor()
        audio_token = getattr(hf_processor, "audio_token", "<|audio|>")
        return audio_token * num_audios
```
**EN:** Method `GraniteSpeechDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GraniteSpeechDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Class `GraniteSpeechEncoderProjector` (lines 241-291)
```python
class GraniteSpeechEncoderProjector(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        cache_config: CacheConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.projector_config.hidden_size
        self.downsample_rate = config.downsample_rate
        self.window_size = config.window_size
        self.num_queries = config.window_size // config.downsample_rate

        self.query = nn.Parameter(
            torch.zeros(1, self.num_queries, config.projector_config.hidden_size)
        )

        # NOTE - this is implemented generically in transformers,
        # but for now we create the QFormer model directly since
        # all existing models use this for the projector.
        self.qformer = Blip2QFormerModel(
            config.projector_config,
            quant_config=quant_config,
            cache_config=cache_config,
```
**EN:** Class `GraniteSpeechEncoderProjector` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GraniteSpeechEncoderProjector` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GraniteSpeechEncoderProjector.__init__` (lines 242-270)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        cache_config: CacheConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.projector_config.hidden_size
        self.downsample_rate = config.downsample_rate
        self.window_size = config.window_size
        self.num_queries = config.window_size // config.downsample_rate

        self.query = nn.Parameter(
            torch.zeros(1, self.num_queries, config.projector_config.hidden_size)
        )

        # NOTE - this is implemented generically in transformers,
        # but for now we create the QFormer model directly since
        # all existing models use this for the projector.
        self.qformer = Blip2QFormerModel(
            config.projector_config,
            quant_config=quant_config,
            cache_config=cache_config,
            prefix=f"{prefix}.qformer",
        )
        self.linear = nn.Linear(
            config.projector_config.hidden_size, config.text_config.hidden_size
        )
```
**EN:** Method `GraniteSpeechEncoderProjector.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GraniteSpeechEncoderProjector.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GraniteSpeechEncoderProjector.forward` (lines 272-291)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        batch_size, seq_len, dim = hidden_states.size()
        nblocks = math.ceil(seq_len / self.window_size)
        pad = nblocks * self.window_size - seq_len
        hidden_states = nn.functional.pad(hidden_states, (0, 0, 0, pad), "constant", 0)
        hidden_states = hidden_states.view(batch_size * nblocks, self.window_size, dim)

        last_hidden_state = self.qformer(
            query_embeds=self.query.data,
            encoder_hidden_states=hidden_states,
        )

        query_proj = self.linear(
            last_hidden_state.view(
                batch_size,
                nblocks * self.window_size // self.downsample_rate,
                -1,
            )
        )
        return query_proj
```
**EN:** Method `GraniteSpeechEncoderProjector.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GraniteSpeechEncoderProjector.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GraniteSpeechConformerFeedForward` (lines 297-329)
```python
class GraniteSpeechConformerFeedForward(nn.Module):
    """Feedforward module for conformer encoder blocks."""

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.pre_norm = nn.LayerNorm(config.hidden_dim)

        self.up_proj = ColumnParallelLinear(
            input_size=config.hidden_dim,
            output_size=config.hidden_dim * config.feedforward_mult,
            quant_config=quant_config,
            prefix=f"{prefix}.up_proj",
        )
        self.silu = nn.SiLU()

        self.down_proj = RowParallelLinear(
            input_size=config.hidden_dim * config.feedforward_mult,
            output_size=config.hidden_dim,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
```
**EN:** Class `GraniteSpeechConformerFeedForward` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GraniteSpeechConformerFeedForward` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GraniteSpeechConformerFeedForward.__init__` (lines 300-322)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.pre_norm = nn.LayerNorm(config.hidden_dim)

        self.up_proj = ColumnParallelLinear(
            input_size=config.hidden_dim,
            output_size=config.hidden_dim * config.feedforward_mult,
            quant_config=quant_config,
            prefix=f"{prefix}.up_proj",
        )
        self.silu = nn.SiLU()

        self.down_proj = RowParallelLinear(
            input_size=config.hidden_dim * config.feedforward_mult,
            output_size=config.hidden_dim,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
        )
```
**EN:** Method `GraniteSpeechConformerFeedForward.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GraniteSpeechConformerFeedForward.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GraniteSpeechConformerFeedForward.forward` (lines 324-329)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states = self.pre_norm(hidden_states)
        hidden_states, _ = self.up_proj(hidden_states)
        hidden_states = self.silu(hidden_states)
        hidden_states, _ = self.down_proj(hidden_states)
        return hidden_states
```
**EN:** Method `GraniteSpeechConformerFeedForward.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GraniteSpeechConformerFeedForward.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GraniteSpeechConformerAttention` (lines 332-419)
```python
class GraniteSpeechConformerAttention(nn.Module):
    """Attention for conformer blocks using Shaw's relative positional
    embeddings. See the following [paper](https://arxiv.org/pdf/1803.02155)
    for more details.
    """

    def __init__(self, config: PretrainedConfig, prefix: str = ""):
        super().__init__()

        inner_dim = config.dim_head * config.num_heads
        self.max_pos_emb = config.max_pos_emb
        self.context_size = config.context_size
        self.num_heads = config.num_heads
        self.dim_head = config.dim_head
        self.scale = self.dim_head**-0.5
        self.pre_norm = nn.LayerNorm(config.hidden_dim)
        self.to_q = nn.Linear(config.hidden_dim, inner_dim, bias=False)
        self.to_kv = nn.Linear(config.hidden_dim, inner_dim * 2, bias=False)
        self.to_out = nn.Linear(inner_dim, config.hidden_dim)
        self.rel_pos_emb = nn.Embedding(2 * self.max_pos_emb + 1, self.dim_head)

        if self.context_size <= 0 or self.context_size > self.max_pos_emb:
            raise ValueError(
                f"Context size should be > 0 and "
                f"<= max_pos_emb ({self.max_pos_emb}), "
```
**EN:** Class `GraniteSpeechConformerAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GraniteSpeechConformerAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GraniteSpeechConformerAttention.__init__` (lines 338-358)
```python
    def __init__(self, config: PretrainedConfig, prefix: str = ""):
        super().__init__()

        inner_dim = config.dim_head * config.num_heads
        self.max_pos_emb = config.max_pos_emb
        self.context_size = config.context_size
        self.num_heads = config.num_heads
        self.dim_head = config.dim_head
        self.scale = self.dim_head**-0.5
        self.pre_norm = nn.LayerNorm(config.hidden_dim)
        self.to_q = nn.Linear(config.hidden_dim, inner_dim, bias=False)
        self.to_kv = nn.Linear(config.hidden_dim, inner_dim * 2, bias=False)
        self.to_out = nn.Linear(inner_dim, config.hidden_dim)
        self.rel_pos_emb = nn.Embedding(2 * self.max_pos_emb + 1, self.dim_head)

        if self.context_size <= 0 or self.context_size > self.max_pos_emb:
            raise ValueError(
                f"Context size should be > 0 and "
                f"<= max_pos_emb ({self.max_pos_emb}), "
                f"got {self.context_size}."
            )
```
**EN:** Method `GraniteSpeechConformerAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GraniteSpeechConformerAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GraniteSpeechConformerAttention.forward` (lines 360-419)
```python
    def forward(
        self, hidden_states: torch.Tensor, attention_dists: torch.Tensor
    ) -> torch.Tensor:
        hidden_states = self.pre_norm(hidden_states)
        bsz, num_features, _ = hidden_states.shape

        num_blocks = math.ceil(num_features / self.context_size)
        remainder = num_features % self.context_size
        if remainder > 0:
            # right padding to reach block size
            hidden_states = torch.nn.functional.pad(
                hidden_states, (0, 0, 0, self.context_size - remainder)
            )

        # NOTE: would be nice to try to use qkvparallellinear
        # here for this block attention implementation if possible
        query_states = self.to_q(hidden_states)
        key_states, value_states = self.to_kv(hidden_states).chunk(2, dim=-1)

        query_states = query_states.reshape(
            bsz, num_blocks, self.context_size, self.num_heads, -1
        ).transpose(2, 3)
        key_states = key_states.reshape(
            bsz, num_blocks, self.context_size, self.num_heads, -1
        ).transpose(2, 3)
        value_states = value_states.reshape(
            bsz, num_blocks, self.context_size, self.num_heads, -1
        ).transpose(2, 3)

        # shaw's relative positional embedding
        dist = attention_dists.to(hidden_states.device)
        rel_pos_emb = self.rel_pos_emb(dist)
        rel_pos_emb_expanded = rel_pos_emb.view([1, 1, 1] + list(rel_pos_emb.shape))
        pos_attn = (
            torch.sum(query_states.unsqueeze(-2) * rel_pos_emb_expanded, dim=-1)
            * self.scale
        )

        if remainder > 0:
            # masked attention in the extended block
            mask = torch.ones(
                self.context_size,
                self.context_size,
                dtype=bool,
                device=hidden_states.device,
            )
            mask[:remainder, :remainder] = 0
            mask_value = -torch.finfo(pos_attn.dtype).max
            pos_attn[:, -1, :].masked_fill_(mask, mask_value)

        with torch.nn.attention.sdpa_kernel(torch.nn.attention.SDPBackend.MATH):
            out = F.scaled_dot_product_attention(
                query_states,
                key_states,
                value_states,
                attn_mask=pos_attn,
                scale=self.scale,
            )
        out = out.transpose(2, 3).reshape(bsz, hidden_states.shape[1], -1)
        return self.to_out(out[:, :num_features, :])
```
**EN:** Method `GraniteSpeechConformerAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GraniteSpeechConformerAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GraniteSpeechConformerDepthWiseConv1d` (lines 422-438)
```python
class GraniteSpeechConformerDepthWiseConv1d(nn.Module):
    """Wrapper for padded 1D pointwise convolution."""

    def __init__(self, chan_in: int, chan_out: int, kernel_size: int, prefix: str = ""):
        super().__init__()
        # Padding for the 1D conv is symmetric or close (i.e., offset by one).
        pad = kernel_size // 2
        pad_offset = (kernel_size + 1) % 2
        self.padding = (pad, pad - pad_offset)

        self.conv = nn.Conv1d(
            chan_in, chan_out, kernel_size, groups=chan_in, bias=False
        )

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states = F.pad(hidden_states, self.padding)
        return self.conv(hidden_states)
```
**EN:** Class `GraniteSpeechConformerDepthWiseConv1d` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GraniteSpeechConformerDepthWiseConv1d` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GraniteSpeechConformerDepthWiseConv1d.__init__` (lines 425-434)
```python
    def __init__(self, chan_in: int, chan_out: int, kernel_size: int, prefix: str = ""):
        super().__init__()
        # Padding for the 1D conv is symmetric or close (i.e., offset by one).
        pad = kernel_size // 2
        pad_offset = (kernel_size + 1) % 2
        self.padding = (pad, pad - pad_offset)

        self.conv = nn.Conv1d(
            chan_in, chan_out, kernel_size, groups=chan_in, bias=False
        )
```
**EN:** Method `GraniteSpeechConformerDepthWiseConv1d.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GraniteSpeechConformerDepthWiseConv1d.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GraniteSpeechConformerDepthWiseConv1d.forward` (lines 436-438)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states = F.pad(hidden_states, self.padding)
        return self.conv(hidden_states)
```
**EN:** Method `GraniteSpeechConformerDepthWiseConv1d.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GraniteSpeechConformerDepthWiseConv1d.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GraniteSpeechConformerConvModule` (lines 441-470)
```python
class GraniteSpeechConformerConvModule(nn.Module):
    """Conformer conv module consisting of several 1D/depthwise 1D
    convolutional layers.
    """

    def __init__(self, config: PretrainedConfig, prefix: str = ""):
        super().__init__()
        inner_dim = config.hidden_dim * config.conv_expansion_factor

        self.norm = nn.LayerNorm(config.hidden_dim)
        self.up_conv = nn.Conv1d(config.hidden_dim, inner_dim * 2, 1)
        self.glu = nn.GLU(dim=1)
        self.depth_conv = GraniteSpeechConformerDepthWiseConv1d(
            inner_dim,
            inner_dim,
            kernel_size=config.conv_kernel_size,
            prefix=f"{prefix}.depth_conv",
        )
        self.silu = nn.SiLU()
        self.batch_norm = nn.BatchNorm1d(inner_dim)
        self.down_conv = nn.Conv1d(inner_dim, config.hidden_dim, 1)

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states = self.norm(hidden_states)
        hidden_states = self.up_conv(hidden_states.permute(0, 2, 1))
```
**EN:** Class `GraniteSpeechConformerConvModule` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GraniteSpeechConformerConvModule` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GraniteSpeechConformerConvModule.__init__` (lines 446-461)
```python
    def __init__(self, config: PretrainedConfig, prefix: str = ""):
        super().__init__()
        inner_dim = config.hidden_dim * config.conv_expansion_factor

        self.norm = nn.LayerNorm(config.hidden_dim)
        self.up_conv = nn.Conv1d(config.hidden_dim, inner_dim * 2, 1)
        self.glu = nn.GLU(dim=1)
        self.depth_conv = GraniteSpeechConformerDepthWiseConv1d(
            inner_dim,
            inner_dim,
            kernel_size=config.conv_kernel_size,
            prefix=f"{prefix}.depth_conv",
        )
        self.silu = nn.SiLU()
        self.batch_norm = nn.BatchNorm1d(inner_dim)
        self.down_conv = nn.Conv1d(inner_dim, config.hidden_dim, 1)
```
**EN:** Method `GraniteSpeechConformerConvModule.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GraniteSpeechConformerConvModule.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GraniteSpeechConformerConvModule.forward` (lines 463-470)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states = self.norm(hidden_states)
        hidden_states = self.up_conv(hidden_states.permute(0, 2, 1))
        hidden_states = self.glu(hidden_states)
        hidden_states = self.depth_conv(hidden_states)
        hidden_states = self.silu(self.batch_norm(hidden_states))
        hidden_states = self.down_conv(hidden_states).permute(0, 2, 1)
        return hidden_states
```
**EN:** Method `GraniteSpeechConformerConvModule.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GraniteSpeechConformerConvModule.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GraniteSpeechConformerBlock` (lines 473-495)
```python
class GraniteSpeechConformerBlock(nn.Module):
    """Conformer block, consisting largely of linear layers,
    attention, and convolutional layers."""

    def __init__(self, config: PretrainedConfig, prefix: str = ""):
        super().__init__()
        self.ff1 = GraniteSpeechConformerFeedForward(config, prefix=f"{prefix}.ff1")
        self.attn = GraniteSpeechConformerAttention(config, prefix=f"{prefix}.attn")
        self.conv = GraniteSpeechConformerConvModule(config, prefix=f"{prefix}.conv")
        self.ff2 = GraniteSpeechConformerFeedForward(config, prefix=f"{prefix}.ff2")
        self.post_norm = nn.LayerNorm(config.hidden_dim)

    def forward(
        self, hidden_states: torch.Tensor, attention_dists: torch.Tensor
    ) -> torch.Tensor:
        hidden_states = 0.5 * self.ff1(hidden_states) + hidden_states
        hidden_states = (
            self.attn(hidden_states, attention_dists=attention_dists) + hidden_states
        )
        hidden_states = self.conv(hidden_states) + hidden_states
        hidden_states = 0.5 * self.ff2(hidden_states) + hidden_states
        hidden_states = self.post_norm(hidden_states)
        return hidden_states
```
**EN:** Class `GraniteSpeechConformerBlock` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GraniteSpeechConformerBlock` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GraniteSpeechConformerBlock.__init__` (lines 477-483)
```python
    def __init__(self, config: PretrainedConfig, prefix: str = ""):
        super().__init__()
        self.ff1 = GraniteSpeechConformerFeedForward(config, prefix=f"{prefix}.ff1")
        self.attn = GraniteSpeechConformerAttention(config, prefix=f"{prefix}.attn")
        self.conv = GraniteSpeechConformerConvModule(config, prefix=f"{prefix}.conv")
        self.ff2 = GraniteSpeechConformerFeedForward(config, prefix=f"{prefix}.ff2")
        self.post_norm = nn.LayerNorm(config.hidden_dim)
```
**EN:** Method `GraniteSpeechConformerBlock.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GraniteSpeechConformerBlock.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GraniteSpeechConformerBlock.forward` (lines 485-495)
```python
    def forward(
        self, hidden_states: torch.Tensor, attention_dists: torch.Tensor
    ) -> torch.Tensor:
        hidden_states = 0.5 * self.ff1(hidden_states) + hidden_states
        hidden_states = (
            self.attn(hidden_states, attention_dists=attention_dists) + hidden_states
        )
        hidden_states = self.conv(hidden_states) + hidden_states
        hidden_states = 0.5 * self.ff2(hidden_states) + hidden_states
        hidden_states = self.post_norm(hidden_states)
        return hidden_states
```
**EN:** Method `GraniteSpeechConformerBlock.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GraniteSpeechConformerBlock.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **LoRA support / LoRA 支持**
  - **EN:** The file is aware of low-rank adaptation interfaces or compatible parameter layouts.
  - **CN:** 该文件考虑了 LoRA 接口或兼容的低秩参数布局。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable, Mapping`, `from typing import Annotated`
- **Third-party / 第三方**: `import torch`, `import torch.nn.functional as F`, `from torch import nn`, `from transformers import BatchFeature, PretrainedConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import CacheConfig, ModelConfig, SpeechToTextConfig, VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.config.speech_to_text import SpeechToTextParams`, `from vllm.inputs import MultiModalDataDict, PromptType, TokensPrompt`, `from vllm.model_executor.layers.linear import ColumnParallelLinear, RowParallelLinear`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import (`, `from vllm.multimodal.processing import (`, `from vllm.sequence import IntermediateTensors`
- **Module note / 模块说明**: **EN:** Inference-only IBM Granite speech model. **CN:** 模块文档字符串给出的原始说明是：Inference-only IBM Granite speech model.。
