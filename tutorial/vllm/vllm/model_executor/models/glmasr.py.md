# glmasr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/glmasr.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for glmasr, including encoder/decoder glue and vLLM runtime adaptation. / 面向 glmasr 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-69)
```python
from collections.abc import Iterable, Mapping, Sequence
from typing import Annotated, Any, Literal, TypeAlias

import torch
import torch.nn as nn
from transformers import BatchFeature
from transformers.models.glmasr import GlmAsrConfig, GlmAsrProcessor
from transformers.models.whisper import WhisperFeatureExtractor

from vllm.config import ModelConfig, SpeechToTextConfig, VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.config.speech_to_text import SpeechToTextParams
from vllm.distributed.parallel_state import get_tensor_model_parallel_world_size
from vllm.inputs import ModalityData, MultiModalDataDict, PromptType, TokensPrompt
from vllm.model_executor.layers.activation import get_act_fn
from vllm.model_executor.layers.attention import MMEncoderAttention
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding.common import ApplyRotaryEmb
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import (
    DictEmbeddingItems,
    ModalityDataItems,
    MultiModalDataItems,
    MultiModalDataParser,
)
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
    PromptUpdateDetails,
)
from vllm.sequence import IntermediateTensors
from vllm.tokenizers import cached_tokenizer_from_config
from vllm.transformers_utils.processor import cached_processor_from_config
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .glmasr_utils import (
    DEFAULT_CONV_PARAMS,
    DEFAULT_MAX_AUDIO_LEN_S,
    DEFAULT_MERGE_FACTOR,
    _flatten_audio_features_by_length,
    _get_audio_output_lengths_for_tower,
    _group_audio_embeddings,
    _normalize_chunk_counts,
)
from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMultiModal,
    SupportsPP,
    SupportsTranscription,
)
from .utils import AutoWeightsLoader, init_vllm_registered_model, maybe_prefix
from .whisper import ISO639_1_SUPPORTED_LANGS, _create_fake_bias_for_k_proj
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `_glmasr_field_config` (lines 623-657)
```python
def _glmasr_field_config(
    hf_inputs: Mapping[str, torch.Tensor],
) -> dict[str, MultiModalFieldConfig]:
    """
    Configure multimodal field batching strategy for GLM-ASR.

    Determines how to batch audio inputs based on whether chunking is used.
    When chunk_counts is present, features are flattened across chunks;
    otherwise, they are batched normally.

    Args:
        hf_inputs: Dictionary of preprocessed inputs from HuggingFace processor.

    Returns:
        Dictionary mapping field names to MultiModalFieldConfig objects \
            that specify batching behavior.
    """
    chunk_counts = hf_inputs.get("chunk_counts")
    if chunk_counts is not None:
        return dict(
            audio_embeds=MultiModalFieldConfig.batched("audio"),
            input_features=MultiModalFieldConfig.flat_from_sizes(
                "audio", chunk_counts, dim=0
            ),
            feature_attention_mask=MultiModalFieldConfig.flat_from_sizes(
                "audio", chunk_counts, dim=0
            ),
            chunk_counts=MultiModalFieldConfig.batched("audio"),
        )
    return dict(
        audio_embeds=MultiModalFieldConfig.batched("audio"),
        input_features=MultiModalFieldConfig.batched("audio"),
        feature_attention_mask=MultiModalFieldConfig.batched("audio"),
        chunk_counts=MultiModalFieldConfig.batched("audio"),
    )
```
**EN:** Function `_glmasr_field_config` encapsulates a focused piece of reusable logic inside this module. The docstring says: Configure multimodal field batching strategy for GLM-ASR.
**CN:** Function `_glmasr_field_config` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Configure multimodal field batching strategy for GLM-ASR。

### Class `GlmAsrEncoderRotaryEmbedding` (lines 72-128)
```python
class GlmAsrEncoderRotaryEmbedding(nn.Module):
    """
    Rotary Position Embedding for GLM-ASR encoder.

    Computes rotary position embeddings on-demand for efficiency.
    Only caches inv_freq as a buffer; cos/sin are computed during forward
    to avoid wasted computation during initialization and ensure correct
    device placement.
    """

    def __init__(self, config) -> None:
        super().__init__()

        # Compute inverse frequencies following transformers implementation
        head_dim = getattr(
            config, "head_dim", config.hidden_size // config.num_attention_heads
        )

        # Handle rope_parameters if present (for compatibility with transformers config)
        if hasattr(config, "rope_parameters") and config.rope_parameters:
            base = config.rope_parameters.get("rope_theta", 10000.0)
            partial_rotary_factor = config.rope_parameters.get(
                "partial_rotary_factor", 1.0
            )
            dim = int(head_dim * partial_rotary_factor)
```
**EN:** Class `GlmAsrEncoderRotaryEmbedding` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GlmAsrEncoderRotaryEmbedding` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GlmAsrEncoderRotaryEmbedding.__init__` (lines 82-110)
```python
    def __init__(self, config) -> None:
        super().__init__()

        # Compute inverse frequencies following transformers implementation
        head_dim = getattr(
            config, "head_dim", config.hidden_size // config.num_attention_heads
        )

        # Handle rope_parameters if present (for compatibility with transformers config)
        if hasattr(config, "rope_parameters") and config.rope_parameters:
            base = config.rope_parameters.get("rope_theta", 10000.0)
            partial_rotary_factor = config.rope_parameters.get(
                "partial_rotary_factor", 1.0
            )
            dim = int(head_dim * partial_rotary_factor)
            self.attention_scaling = config.rope_parameters.get(
                "attention_scaling", 1.0
            )
        else:
            base = getattr(config, "rope_theta", 10000.0)
            dim = head_dim
            self.attention_scaling = 1.0

        self.dim = dim
        self.head_dim = head_dim

        # Only cache inv_freq; cos/sin computed on-demand in correct device
        inv_freq = 1.0 / (base ** (torch.arange(0, dim, 2, dtype=torch.float) / dim))
        self.register_buffer("inv_freq", inv_freq, persistent=False)
```
**EN:** Method `GlmAsrEncoderRotaryEmbedding.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GlmAsrEncoderRotaryEmbedding.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GlmAsrEncoderRotaryEmbedding.forward` (lines 112-128)
```python
    def forward(self, seq_len: int) -> torch.Tensor:
        """
        Compute rotary position frequencies for given sequence length.

        Args:
            seq_len: The sequence length to compute embeddings for.

        Returns:
            Frequency tensor with shape [seq_len, dim/2]. Use .cos() and
            .sin() to get the rotary embedding components.
        """
        # Compute on the same device as inv_freq (automatically correct after .to())
        seq = torch.arange(
            seq_len, device=self.inv_freq.device, dtype=self.inv_freq.dtype
        )
        freqs = torch.outer(seq, self.inv_freq)
        return freqs * self.attention_scaling
```
**EN:** Method `GlmAsrEncoderRotaryEmbedding.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Compute rotary position frequencies for given sequence length.
**CN:** Method `GlmAsrEncoderRotaryEmbedding.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Compute rotary position frequencies for given sequence length。

### Class `GlmAsrEncoderAttention` (lines 131-249)
```python
class GlmAsrEncoderAttention(nn.Module):
    """
    Optimized Multi-headed Grouped Query Attention for GLM-ASR encoder.

    Uses vLLM's QKVParallelLinear for fused projections, ApplyRotaryEmb for
    rotary position embeddings, and MMEncoderAttention for hardware-optimized
    attention computation with automatic backend selection.
    """

    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.num_heads = config.num_attention_heads
        self.num_kv_heads = getattr(
            config, "num_key_value_heads", config.num_attention_heads
        )
        self.head_dim = self.hidden_size // self.num_heads

        self.tp_size = get_tensor_model_parallel_world_size()
```
**EN:** Class `GlmAsrEncoderAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GlmAsrEncoderAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GlmAsrEncoderAttention.__init__` (lines 140-198)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.num_heads = config.num_attention_heads
        self.num_kv_heads = getattr(
            config, "num_key_value_heads", config.num_attention_heads
        )
        self.head_dim = self.hidden_size // self.num_heads

        self.tp_size = get_tensor_model_parallel_world_size()
        self.num_heads_per_rank = self.num_heads // self.tp_size
        self.num_kv_heads_per_rank = max(1, self.num_kv_heads // self.tp_size)

        # Use QKVParallelLinear for fused QKV projection
        # Note: GLM-ASR uses bias on Q and V, but not K
        # For simplicity with QKVParallelLinear, we use bias=True for all
        self.qkv_proj = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
            self.num_heads,
            self.num_kv_heads,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )

        self.o_proj = RowParallelLinear(
            self.hidden_size,
            self.hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.o_proj",
        )

        # Use vLLM's ApplyRotaryEmb CustomOp
        # enforce_enable=True ensures the op is always enabled (important for ViT)
        rope_params = getattr(config, "rope_parameters", None)
        if rope_params:
            partial_rotary_factor = rope_params.get("partial_rotary_factor", 0.5)
        else:
            partial_rotary_factor = getattr(config, "partial_rotary_factor", 0.5)
        self.rotary_dim = int(self.head_dim * partial_rotary_factor)
        self.apply_rotary_emb = ApplyRotaryEmb(enforce_enable=True)

        # Use vLLM's MMEncoderAttention for hardware-optimized attention
        # Automatically selects Flash Attention, SDPA, or Pallas based on device
        self.attn = MMEncoderAttention(
            num_heads=self.num_heads_per_rank,
            head_size=self.head_dim,
            scale=self.head_dim**-0.5,
            num_kv_heads=self.num_kv_heads_per_rank,
            prefix=f"{prefix}.attn",
        )
```
**EN:** Method `GlmAsrEncoderAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GlmAsrEncoderAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GlmAsrEncoderAttention.forward` (lines 200-249)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        rotary_pos_emb_cos: torch.Tensor,
        rotary_pos_emb_sin: torch.Tensor,
    ) -> torch.Tensor:
        """
        Args:
            hidden_states: [batch_size, seq_len, hidden_size]
            rotary_pos_emb_cos: [seq_len, rotary_dim/2] - cosine of rotary embeddings
            rotary_pos_emb_sin: [seq_len, rotary_dim/2] - sine of rotary embeddings

        Returns:
            [batch_size, seq_len, hidden_size]
        """
        batch_size, seq_len, _ = hidden_states.shape

        # QKV projection - fused for efficiency
        qkv, _ = self.qkv_proj(hidden_states)

        # Split into q, k, v
        q_size = self.num_heads_per_rank * self.head_dim
        kv_size = self.num_kv_heads_per_rank * self.head_dim
        q, k, v = qkv.split([q_size, kv_size, kv_size], dim=-1)

        # Reshape to [batch, seq, num_heads, head_dim] for ApplyRotaryEmb
        q = q.view(batch_size, seq_len, self.num_heads_per_rank, self.head_dim)
        k = k.view(batch_size, seq_len, self.num_kv_heads_per_rank, self.head_dim)
        v = v.view(batch_size, seq_len, self.num_kv_heads_per_rank, self.head_dim)

        # Apply rotary position embeddings using vLLM's ApplyRotaryEmb
        # ApplyRotaryEmb expects x: [batch, seq, heads, head_dim]
        # cos/sin: [seq_len, rotary_dim/2]
        q[..., : self.rotary_dim] = self.apply_rotary_emb(
            q[..., : self.rotary_dim], rotary_pos_emb_cos, rotary_pos_emb_sin
        )
        k[..., : self.rotary_dim] = self.apply_rotary_emb(
            k[..., : self.rotary_dim], rotary_pos_emb_cos, rotary_pos_emb_sin
        )

        # MMEncoderAttention expects [batch, seq, num_heads, head_dim]
        # It handles GQA internally via repeat_interleave
        attn_output = self.attn(q, k, v)

        # Reshape back to [batch, seq, hidden_size]
        attn_output = attn_output.view(batch_size, seq_len, -1)

        # Output projection
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** Method `GlmAsrEncoderAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Args: hidden_states: [batch_size, seq_len, hidden_size] rotary_pos_emb_cos: [seq_len, rotary_dim/2] - cosine of rotary embeddings rotary_pos_emb_sin: [seq_len, rotary_dim/2] - sine of rotary embeddings  Returns: [batch_size, seq_len, hidden_size].
**CN:** Method `GlmAsrEncoderAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Args: hidden_states: [batch_size, seq_len, hidden_size] rotary_pos_emb_cos: [seq_len, rotary_dim/2] - cosine of rotary embeddings rotary_pos_emb_sin: [seq_len, rotary_dim/2] - sine of rotary embeddings  Returns: [batch_size, seq_len, hidden_size]。

### Class `GlmAsrEncoderMLP` (lines 252-291)
```python
class GlmAsrEncoderMLP(nn.Module):
    """
    Optimized MLP for GLM-ASR encoder.
    Uses vLLM's parallel linear layers for better performance.
    """

    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.intermediate_size = config.intermediate_size

        self.fc1 = ColumnParallelLinear(
            self.hidden_size,
            self.intermediate_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.fc1",
        )
```
**EN:** Class `GlmAsrEncoderMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GlmAsrEncoderMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GlmAsrEncoderMLP.__init__` (lines 258-285)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.intermediate_size = config.intermediate_size

        self.fc1 = ColumnParallelLinear(
            self.hidden_size,
            self.intermediate_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.fc1",
        )

        self.act_fn = get_act_fn(config.hidden_act)

        self.fc2 = RowParallelLinear(
            self.intermediate_size,
            self.hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.fc2",
        )
```
**EN:** Method `GlmAsrEncoderMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GlmAsrEncoderMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GlmAsrEncoderMLP.forward` (lines 287-291)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.fc1(hidden_states)
        hidden_states = self.act_fn(hidden_states)
        hidden_states, _ = self.fc2(hidden_states)
        return hidden_states
```
**EN:** Method `GlmAsrEncoderMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GlmAsrEncoderMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GlmAsrEncoderLayer` (lines 294-358)
```python
class GlmAsrEncoderLayer(nn.Module):
    """
    Optimized Transformer encoder layer for GLM-ASR.
    Combines attention and MLP with residual connections and layer norms.
    """

    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size

        self.self_attn = GlmAsrEncoderAttention(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )

        self.mlp = GlmAsrEncoderMLP(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
```
**EN:** Class `GlmAsrEncoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GlmAsrEncoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GlmAsrEncoderLayer.__init__` (lines 300-325)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size

        self.self_attn = GlmAsrEncoderAttention(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )

        self.mlp = GlmAsrEncoderMLP(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )

        layer_norm_eps = getattr(config, "layer_norm_eps", 1e-5)
        self.input_layernorm = nn.LayerNorm(self.hidden_size, eps=layer_norm_eps)
        self.post_attention_layernorm = nn.LayerNorm(
            self.hidden_size, eps=layer_norm_eps
        )
```
**EN:** Method `GlmAsrEncoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GlmAsrEncoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GlmAsrEncoderLayer.forward` (lines 327-358)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        rotary_pos_emb_cos: torch.Tensor,
        rotary_pos_emb_sin: torch.Tensor,
    ) -> torch.Tensor:
        """
        Args:
            hidden_states: [batch_size, seq_len, hidden_size]
            rotary_pos_emb_cos: [seq_len, rotary_dim/2] - cosine of rotary embeddings
            rotary_pos_emb_sin: [seq_len, rotary_dim/2] - sine of rotary embeddings

        Returns:
            [batch_size, seq_len, hidden_size]
        """
        # Self-attention with residual
        residual = hidden_states
        hidden_states = self.input_layernorm(hidden_states)
        hidden_states = self.self_attn(
            hidden_states=hidden_states,
            rotary_pos_emb_cos=rotary_pos_emb_cos,
            rotary_pos_emb_sin=rotary_pos_emb_sin,
        )
        hidden_states = residual + hidden_states

        # MLP with residual
        residual = hidden_states
        hidden_states = self.post_attention_layernorm(hidden_states)
        hidden_states = self.mlp(hidden_states)
        hidden_states = residual + hidden_states

        return hidden_states
```
**EN:** Method `GlmAsrEncoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Args: hidden_states: [batch_size, seq_len, hidden_size] rotary_pos_emb_cos: [seq_len, rotary_dim/2] - cosine of rotary embeddings rotary_pos_emb_sin: [seq_len, rotary_dim/2] - sine of rotary embeddings  Returns: [batch_size, seq_len, hidden_size].
**CN:** Method `GlmAsrEncoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Args: hidden_states: [batch_size, seq_len, hidden_size] rotary_pos_emb_cos: [seq_len, rotary_dim/2] - cosine of rotary embeddings rotary_pos_emb_sin: [seq_len, rotary_dim/2] - sine of rotary embeddings  Returns: [batch_size, seq_len, hidden_size]。

### Class `_GlmAsrEncoderOutput` (lines 361-377)
```python
class _GlmAsrEncoderOutput:
    """
    Simple output container compatible with transformers' BaseModelOutput.

    This lightweight container holds the encoder output and is compatible
    with the transformers library's output format while being more efficient
    than a full dataclass.

    Attributes:
        last_hidden_state: Final layer hidden states from the encoder.
            Shape: [batch_size, seq_len, hidden_size]
    """

    __slots__ = ("last_hidden_state",)

    def __init__(self, last_hidden_state: torch.Tensor):
        self.last_hidden_state = last_hidden_state
```
**EN:** Class `_GlmAsrEncoderOutput` organizes related behavior for this model family or helper component. Key methods include __init__.
**CN:** 类 `_GlmAsrEncoderOutput` 用于组织该模型族或辅助组件的相关行为。 关键方法包括 __init__。

### Method `_GlmAsrEncoderOutput.__init__` (lines 376-377)
```python
    def __init__(self, last_hidden_state: torch.Tensor):
        self.last_hidden_state = last_hidden_state
```
**EN:** Method `_GlmAsrEncoderOutput.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `_GlmAsrEncoderOutput.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `GlmAsrEncoder` (lines 380-536)
```python
class GlmAsrEncoder(nn.Module):
    """
    Optimized GLM-ASR Audio Encoder with vLLM native implementation.

    This encoder processes audio features through convolutional layers
    followed by transformer layers with rotary position embeddings.
    Optimized for performance with:
    - QKVParallelLinear for fused attention projections
    - Tensor parallelism support via ColumnParallelLinear/RowParallelLinear
    - Quantization support
    - Flash Attention (SDPA)
    """

    # Mapping for weight loading: transformers uses separate q/k/v, we use fused qkv
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
    }

    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
```
**EN:** Class `GlmAsrEncoder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, _get_feat_extract_output_lengths, forward, load_weights.
**CN:** 类 `GlmAsrEncoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, _get_feat_extract_output_lengths, forward, load_weights。

### Method `GlmAsrEncoder.__init__` (lines 398-439)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config

        # Convolutional feature extraction layers
        self.conv1 = nn.Conv1d(
            config.num_mel_bins,
            config.hidden_size,
            kernel_size=3,
            padding=1,
        )
        self.conv2 = nn.Conv1d(
            config.hidden_size,
            config.hidden_size,
            kernel_size=3,
            stride=2,
            padding=1,
        )

        # Transformer encoder layers
        self.layers = nn.ModuleList(
            [
                GlmAsrEncoderLayer(
                    config,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layers.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )

        # Final layer norm
        layer_norm_eps = getattr(config, "layer_norm_eps", 1e-5)
        self.norm = nn.LayerNorm(config.hidden_size, eps=layer_norm_eps)

        # Rotary position embeddings
        self.rotary_emb = GlmAsrEncoderRotaryEmbedding(config)
```
**EN:** Method `GlmAsrEncoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GlmAsrEncoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GlmAsrEncoder.forward` (lines 461-496)
```python
    def forward(self, input_features: torch.Tensor) -> _GlmAsrEncoderOutput:
        """
        Forward pass through the encoder.

        Args:
            input_features: [batch_size, num_mel_bins, seq_len]

        Returns:
            _GlmAsrEncoderOutput: Object with .last_hidden_state attribute \
                containing [batch_size, seq_len', hidden_size] where seq_len' \
                is the sequence length after convolutions
        """
        # Apply convolutional layers with GELU activation
        hidden_states = torch.nn.functional.gelu(self.conv1(input_features))
        hidden_states = torch.nn.functional.gelu(self.conv2(hidden_states))

        # Transpose to [batch_size, seq_len, hidden_size]
        hidden_states = hidden_states.transpose(1, 2)
        output_seq_len = hidden_states.shape[1]

        # Compute rotary position embeddings on-demand
        rotary_pos_emb = self.rotary_emb(output_seq_len)
        rotary_pos_emb_cos = rotary_pos_emb.cos().to(dtype=hidden_states.dtype)
        rotary_pos_emb_sin = rotary_pos_emb.sin().to(dtype=hidden_states.dtype)

        # Apply transformer layers
        for encoder_layer in self.layers:
            hidden_states = encoder_layer(
                hidden_states, rotary_pos_emb_cos, rotary_pos_emb_sin
            )

        # Final layer norm
        hidden_states = self.norm(hidden_states)

        # Return in a format compatible with transformers' BaseModelOutput
        return _GlmAsrEncoderOutput(last_hidden_state=hidden_states)
```
**EN:** Method `GlmAsrEncoder.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Forward pass through the encoder.
**CN:** Method `GlmAsrEncoder.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Forward pass through the encoder。

### Class `GlmAsrFeatureInputs` (lines 539-559)
```python
class GlmAsrFeatureInputs(TensorSchema):
    """
    Dimensions:
        - num_chunks: Number of audio chunks (flattened)
        - nmb: Number of mel bins
        - num_audios: Number of original audio files
    """

    type: Literal["audio_features"]
    input_features: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("num_chunks", "nmb", "chunk_length", dynamic_dims={"chunk_length"}),
    ]
    feature_attention_mask: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("num_chunks", "chunk_length", dynamic_dims={"chunk_length"}),
    ]
    chunk_counts: Annotated[
        torch.Tensor | list[torch.Tensor],
        TensorShape("num_audios"),
    ]
```
**EN:** Class `GlmAsrFeatureInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `GlmAsrFeatureInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `GlmAsrEmbeddingInputs` (lines 562-575)
```python
class GlmAsrEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - bn: Batch size
        - naf: Number of audio features
        - hs: Hidden size (must match the hidden size of language model
          backbone)
    """

    type: Literal["audio_embeds"] = "audio_embeds"
    audio_embeds: Annotated[
        list[torch.Tensor],
        TensorShape("bn", "naf", "hs", dynamic_dims={"naf"}),
    ]
```
**EN:** Class `GlmAsrEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `GlmAsrEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `GlmAsrMultiModalProjector` (lines 581-620)
```python
class GlmAsrMultiModalProjector(nn.Module):
    """
    Projects audio encoder outputs to language model hidden space.

    This projector uses a two-layer MLP to map audio features from the
    encoder's intermediate size to the language model's hidden size.
    Uses vLLM's parallel linear layers for tensor parallelism support.

    Architecture:
        - Linear layer: intermediate_size -> hidden_size * 2
        - Activation function (e.g., GELU)
        - Linear layer: hidden_size * 2 -> hidden_size
    """

    def __init__(
        self,
        config: GlmAsrConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.linear_1 = ColumnParallelLinear(
            input_size=config.audio_config.intermediate_size,
            output_size=config.text_config.hidden_size * 2,
            quant_config=quant_config,
```
**EN:** Class `GlmAsrMultiModalProjector` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `GlmAsrMultiModalProjector` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `GlmAsrMultiModalProjector.__init__` (lines 595-614)
```python
    def __init__(
        self,
        config: GlmAsrConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.linear_1 = ColumnParallelLinear(
            input_size=config.audio_config.intermediate_size,
            output_size=config.text_config.hidden_size * 2,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_1",
        )
        self.act = get_act_fn(config.projector_hidden_act)
        self.linear_2 = RowParallelLinear(
            input_size=config.text_config.hidden_size * 2,
            output_size=config.text_config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_2",
        )
```
**EN:** Method `GlmAsrMultiModalProjector.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GlmAsrMultiModalProjector.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GlmAsrMultiModalProjector.forward` (lines 616-620)
```python
    def forward(self, audio_features: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.linear_1(audio_features)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** Method `GlmAsrMultiModalProjector.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GlmAsrMultiModalProjector.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GlmAsrMultiModalDataParser` (lines 660-679)
```python
class GlmAsrMultiModalDataParser(MultiModalDataParser):
    """
    Custom parser for GLM-ASR multimodal data.

    Extends the base parser to handle GLM-ASR specific audio data formats,
    including both pre-computed audio embeddings and raw audio features.
    """

    def _parse_audio_data(
        self,
        data: dict[str, torch.Tensor] | ModalityData[Any],
    ) -> ModalityDataItems[Any, Any] | None:
        if isinstance(data, dict):
            return DictEmbeddingItems(
                data,
                modality="audio",
                required_fields={"audio_embeds"},
                fields_factory=_glmasr_field_config,
            )
        return super()._parse_audio_data(data)
```
**EN:** Class `GlmAsrMultiModalDataParser` organizes related behavior for this model family or helper component. It inherits from MultiModalDataParser. Key methods include _parse_audio_data.
**CN:** 类 `GlmAsrMultiModalDataParser` 用于组织该模型族或辅助组件的相关行为。 它继承自 MultiModalDataParser。 关键方法包括 _parse_audio_data。

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
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping, Sequence`, `from typing import Annotated, Any, Literal, TypeAlias`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `from transformers import BatchFeature`, `from transformers.models.glmasr import GlmAsrConfig, GlmAsrProcessor`, `from transformers.models.whisper import WhisperFeatureExtractor`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import ModelConfig, SpeechToTextConfig, VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.config.speech_to_text import SpeechToTextParams`, `from vllm.distributed.parallel_state import get_tensor_model_parallel_world_size`, `from vllm.inputs import ModalityData, MultiModalDataDict, PromptType, TokensPrompt`, `from vllm.model_executor.layers.activation import get_act_fn`, `from vllm.model_executor.layers.attention import MMEncoderAttention`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding.common import ApplyRotaryEmb`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`, `from vllm.multimodal import MULTIMODAL_REGISTRY`
