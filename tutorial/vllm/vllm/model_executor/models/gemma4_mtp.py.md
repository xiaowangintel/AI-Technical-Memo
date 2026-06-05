# gemma4_mtp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/gemma4_mtp.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for gemma4_mtp, including architecture wrappers and weight loading logic. / 面向推理的 gemma4_mtp vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 22-56)
```python
from collections.abc import Iterable

import torch
from torch import nn

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import (
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_gather,
)
from vllm.logger import init_logger
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.sequence import IntermediateTensors

from .gemma4 import Gemma4MLP, _get_text_config
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    extract_layer_index,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `Gemma4MTPMaskedEmbedder` (lines 61-145)
```python
class Gemma4MTPMaskedEmbedder(nn.Module):
    """Sparse logit computation via centroid-based vocabulary masking.

    Instead of computing logits against the full vocabulary, projects
    hidden states to centroid scores, selects top-K centroids, and
    computes logits only for the ~top_k * (vocab_size / num_centroids)
    tokens belonging to those centroids.
    """

    token_ordering: torch.Tensor

    def __init__(
        self,
        hidden_size: int,
        vocab_size: int,
        num_centroids: int,
        centroid_intermediate_top_k: int,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.vocab_size = vocab_size
        self.num_centroids = num_centroids
        self.centroid_intermediate_top_k = centroid_intermediate_top_k
        self.vocab_size_per_centroid = vocab_size // num_centroids
        self.num_selected = centroid_intermediate_top_k * self.vocab_size_per_centroid
```
**EN:** Class `Gemma4MTPMaskedEmbedder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, _select_and_score, forward, get_top_tokens.
**CN:** 类 `Gemma4MTPMaskedEmbedder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, _select_and_score, forward, get_top_tokens。

### Method `Gemma4MTPMaskedEmbedder.__init__` (lines 72-91)
```python
    def __init__(
        self,
        hidden_size: int,
        vocab_size: int,
        num_centroids: int,
        centroid_intermediate_top_k: int,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.vocab_size = vocab_size
        self.num_centroids = num_centroids
        self.centroid_intermediate_top_k = centroid_intermediate_top_k
        self.vocab_size_per_centroid = vocab_size // num_centroids
        self.num_selected = centroid_intermediate_top_k * self.vocab_size_per_centroid

        self.centroids = nn.Linear(hidden_size, num_centroids, bias=False)
        self.register_buffer(
            "token_ordering",
            torch.empty(vocab_size, dtype=torch.long),
        )
```
**EN:** Method `Gemma4MTPMaskedEmbedder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4MTPMaskedEmbedder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4MTPMaskedEmbedder.forward` (lines 123-136)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        lm_head_weight: torch.Tensor,
    ) -> torch.Tensor:
        """Full-vocab logits with non-selected positions masked to -inf."""
        logits, indices = self._select_and_score(hidden_states, lm_head_weight)
        output = torch.full(
            (hidden_states.shape[0], self.vocab_size),
            fill_value=torch.finfo(hidden_states.dtype).min,
            dtype=hidden_states.dtype,
            device=hidden_states.device,
        )
        return output.scatter_(-1, indices, logits)
```
**EN:** Method `Gemma4MTPMaskedEmbedder.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Full-vocab logits with non-selected positions masked to -inf.
**CN:** Method `Gemma4MTPMaskedEmbedder.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Full-vocab logits with non-selected positions masked to -inf。

### Method `Gemma4MTPMaskedEmbedder.get_top_tokens` (lines 138-145)
```python
    def get_top_tokens(
        self,
        hidden_states: torch.Tensor,
        lm_head_weight: torch.Tensor,
    ) -> torch.Tensor:
        """Sparse argmax — returns vocab token IDs without full-vocab tensor."""
        logits, indices = self._select_and_score(hidden_states, lm_head_weight)
        return indices.gather(-1, logits.argmax(-1, keepdim=True)).squeeze(-1)
```
**EN:** Method `Gemma4MTPMaskedEmbedder.get_top_tokens` encapsulates a focused piece of reusable logic inside this module. The docstring says: Sparse argmax — returns vocab token IDs without full-vocab tensor.
**CN:** Method `Gemma4MTPMaskedEmbedder.get_top_tokens` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Sparse argmax — returns vocab token IDs without full-vocab tensor。

### Class `Gemma4MTPAttention` (lines 148-259)
```python
class Gemma4MTPAttention(nn.Module):
    """Q-only attention for Gemma4 MTP layers.

    K/V come from the target model's KV cache via
    ``kv_sharing_target_layer_name`` (set by the proposer after
    model construction).
    """

    def __init__(
        self,
        config,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        head_dim: int,
        max_position_embeddings: int,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        attn_logits_soft_cap: float | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = hidden_size
```
**EN:** Class `Gemma4MTPAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma4MTPAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma4MTPAttention.__init__` (lines 156-232)
```python
    def __init__(
        self,
        config,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        head_dim: int,
        max_position_embeddings: int,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        attn_logits_soft_cap: float | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = hidden_size

        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = head_dim
        self.q_size = self.num_heads * self.head_dim
        self.scaling = 1.0

        self.q_proj = ColumnParallelLinear(
            hidden_size,
            self.total_num_heads * self.head_dim,
            bias=config.attention_bias,
            quant_config=None,
            prefix=f"{prefix}.q_proj",
        )
        self.o_proj = RowParallelLinear(
            self.total_num_heads * self.head_dim,
            hidden_size,
            bias=config.attention_bias,
            quant_config=None,
            prefix=f"{prefix}.o_proj",
        )
        self.q_norm = RMSNorm(self.head_dim, eps=config.rms_norm_eps)

        layer_idx = extract_layer_index(prefix)
        layer_type = config.layer_types[layer_idx]
        self.is_sliding = layer_type == "sliding_attention"
        sliding_window = config.sliding_window if self.is_sliding else None

        if layer_type in config.rope_parameters:
            rope_parameters = dict(config.rope_parameters[layer_type])
        else:
            rope_parameters = dict(config.rope_parameters.copy())
            if self.is_sliding:
                rope_parameters["rope_theta"] = getattr(
                    config, "rope_local_base_freq", 10000.0
                )

        self.rotary_emb = get_rope(
            self.head_dim,
            max_position=max_position_embeddings,
            rope_parameters=rope_parameters,
# ... truncated for analysis ...
```
**EN:** Method `Gemma4MTPAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4MTPAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4MTPAttention.forward` (lines 234-259)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        **kwargs,
    ) -> torch.Tensor:
        q, _ = self.q_proj(hidden_states)

        q = q.unflatten(-1, (self.num_heads, self.head_dim))
        q = self.q_norm(q)
        q = q.flatten(-2, -1)

        q, _ = self.rotary_emb(positions, q, None)

        # Attention reads K/V from the target's cache via KV sharing;
        # these dummy tensors are never consumed but required by the API.
        num_tokens = q.shape[0]
        kv_dummy = torch.empty(
            num_tokens,
            self.num_kv_heads * self.head_dim,
            dtype=q.dtype,
            device=q.device,
        )
        attn_output = self.attn(q, kv_dummy, kv_dummy)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** Method `Gemma4MTPAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma4MTPAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma4MTPDecoderLayer` (lines 262-344)
```python
class Gemma4MTPDecoderLayer(nn.Module):
    def __init__(
        self,
        config,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size

        layer_idx = extract_layer_index(prefix)
        layer_type = config.layer_types[layer_idx]
        is_full_attention = layer_type == "full_attention"
        head_dim = (
            getattr(config, "global_head_dim", config.head_dim)
            if is_full_attention
            else config.head_dim
        )

        self.self_attn = Gemma4MTPAttention(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
```
**EN:** Class `Gemma4MTPDecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Gemma4MTPDecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Gemma4MTPDecoderLayer.__init__` (lines 263-315)
```python
    def __init__(
        self,
        config,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size

        layer_idx = extract_layer_index(prefix)
        layer_type = config.layer_types[layer_idx]
        is_full_attention = layer_type == "full_attention"
        head_dim = (
            getattr(config, "global_head_dim", config.head_dim)
            if is_full_attention
            else config.head_dim
        )

        self.self_attn = Gemma4MTPAttention(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            head_dim=head_dim,
            max_position_embeddings=config.max_position_embeddings,
            cache_config=cache_config,
            quant_config=quant_config,
            attn_logits_soft_cap=getattr(config, "attn_logit_softcapping", None),
            prefix=f"{prefix}.self_attn",
        )

        text_config = _get_text_config(config)
        self.mlp = Gemma4MLP(
            hidden_size=self.hidden_size,
            intermediate_size=text_config.intermediate_size,
            hidden_activation=text_config.hidden_activation,
            quant_config=None,
            prefix=f"{prefix}.mlp",
        )

        self.input_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.post_attention_layernorm = RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )
        self.pre_feedforward_layernorm = RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )
        self.post_feedforward_layernorm = RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )

        self.register_buffer("layer_scalar", torch.ones(1))
```
**EN:** Method `Gemma4MTPDecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4MTPDecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4MTPDecoderLayer.forward` (lines 317-344)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
        **kwargs,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        residual = hidden_states
        hidden_states = self.input_layernorm(residual)

        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
            **kwargs,
        )

        hidden_states = self.post_attention_layernorm(hidden_states)
        hidden_states = hidden_states + residual
        residual = hidden_states

        hidden_states = self.pre_feedforward_layernorm(hidden_states)
        hidden_states = self.mlp(hidden_states)

        hidden_states = self.post_feedforward_layernorm(hidden_states)
        hidden_states = hidden_states + residual

        hidden_states = hidden_states * self.layer_scalar
        return hidden_states, None
```
**EN:** Method `Gemma4MTPDecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma4MTPDecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Gemma4MultiTokenPredictor` (lines 347-477)
```python
class Gemma4MultiTokenPredictor(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.speculative_config.draft_model_config.hf_config
        text_config = _get_text_config(config)
        self.config = text_config

        self.hidden_size = text_config.hidden_size
        self.backbone_hidden_size = getattr(
            config, "backbone_hidden_size", self.hidden_size
        )
        self.vocab_size = text_config.vocab_size
        self.num_mtp_layers = text_config.num_hidden_layers

        self.embed_tokens = VocabParallelEmbedding(
            self.vocab_size,
            self.hidden_size,
        )

        self.pre_projection = ColumnParallelLinear(
            2 * self.backbone_hidden_size,
            self.hidden_size,
            bias=False,
            gather_output=True,
```
**EN:** Class `Gemma4MultiTokenPredictor` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, load_weights, forward.
**CN:** 类 `Gemma4MultiTokenPredictor` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, load_weights, forward。

### Method `Gemma4MultiTokenPredictor.__init__` (lines 348-402)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.speculative_config.draft_model_config.hf_config
        text_config = _get_text_config(config)
        self.config = text_config

        self.hidden_size = text_config.hidden_size
        self.backbone_hidden_size = getattr(
            config, "backbone_hidden_size", self.hidden_size
        )
        self.vocab_size = text_config.vocab_size
        self.num_mtp_layers = text_config.num_hidden_layers

        self.embed_tokens = VocabParallelEmbedding(
            self.vocab_size,
            self.hidden_size,
        )

        self.pre_projection = ColumnParallelLinear(
            2 * self.backbone_hidden_size,
            self.hidden_size,
            bias=False,
            gather_output=True,
            prefix=f"{prefix}.pre_projection",
        )

        self.post_projection = RowParallelLinear(
            self.hidden_size,
            self.backbone_hidden_size,
            bias=False,
            input_is_parallel=False,
            prefix=f"{prefix}.post_projection",
        )

        self.layers = nn.ModuleList(
            Gemma4MTPDecoderLayer(
                text_config,
                cache_config=vllm_config.cache_config,
                quant_config=vllm_config.quant_config,
                prefix=f"{prefix}.layers.{idx}",
            )
            for idx in range(self.num_mtp_layers)
        )

        self.norm = RMSNorm(self.hidden_size, eps=text_config.rms_norm_eps)

        # After embedding sharing, embed_tokens is replaced with the
        # target model's backbone-dim embedding.  Scale by
        # sqrt(backbone_hidden_size) to match the target's convention.
        self.register_buffer(
            "normalizer",
            torch.tensor(self.backbone_hidden_size**0.5),
            persistent=False,
        )
```
**EN:** Method `Gemma4MultiTokenPredictor.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4MultiTokenPredictor.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4MultiTokenPredictor.embed_input_ids` (lines 404-405)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids) * self.normalizer
```
**EN:** Method `Gemma4MultiTokenPredictor.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Gemma4MultiTokenPredictor.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `Gemma4MultiTokenPredictor.load_weights` (lines 407-443)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        params_dict = dict(self.named_parameters())
        params_dict.update(dict(self.named_buffers()))
        loaded_params: set[str] = set()

        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue

            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
                name = name.replace(weight_name, param_name)
                if name.endswith(".bias") and name not in params_dict:
                    continue
                if name not in params_dict:
                    continue
                param = params_dict[name]
                weight_loader = param.weight_loader
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
                if name.endswith(".bias") and name not in params_dict:
                    continue
                if name not in params_dict:
                    continue
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)
            loaded_params.add(name)

        return loaded_params
```
**EN:** Method `Gemma4MultiTokenPredictor.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `Gemma4MultiTokenPredictor.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Method `Gemma4MultiTokenPredictor.forward` (lines 445-477)
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        spec_step_idx: int = 0,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """Returns (draft_hidden_states, backbone_hidden_states).

        draft_hidden_states: draft-dim, used by compute_logits via lm_head.
        backbone_hidden_states: backbone-dim, stored in the proposer's
            hidden-state buffer and fed back as input to the next step.
        """
        if inputs_embeds is None:
            inputs_embeds = self.embed_input_ids(input_ids)

        combined = torch.cat([inputs_embeds, hidden_states], dim=-1)
        hidden_states, _ = self.pre_projection(combined)

        residual = None
        for layer in self.layers:
            hidden_states, residual = layer(
                positions=positions,
                hidden_states=hidden_states,
                residual=residual,
            )

        draft_hidden_states = self.norm(hidden_states)

        backbone_hidden_states, _ = self.post_projection(draft_hidden_states)
        return draft_hidden_states, backbone_hidden_states
```
**EN:** Method `Gemma4MultiTokenPredictor.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Returns (draft_hidden_states, backbone_hidden_states).
**CN:** Method `Gemma4MultiTokenPredictor.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Returns (draft_hidden_states, backbone_hidden_states)。

### Class `Gemma4MTP` (lines 481-603)
```python
@support_torch_compile
class Gemma4MTP(nn.Module):
    """Gemma4 Multi-Token Prediction model for speculative decoding.

    forward() returns (draft_hidden_states, backbone_hidden_states).
    The proposer uses draft_hidden_states for compute_logits (via
    the draft-dim lm_head) and backbone_hidden_states for the
    hidden-state feedback buffer.
    """

    has_own_lm_head = True

    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            "pre_projection.": "model.pre_projection.",
            "post_projection.": "model.post_projection.",
        },
    )

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.speculative_config.draft_model_config.hf_config
        text_config = _get_text_config(config)
        self.config = config
```
**EN:** Class `Gemma4MTP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, forward, _get_full_lm_head_weight, compute_logits, get_top_tokens.
**CN:** 类 `Gemma4MTP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, forward, _get_full_lm_head_weight, compute_logits, get_top_tokens。

### Method `Gemma4MTP.__init__` (lines 499-545)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.speculative_config.draft_model_config.hf_config
        text_config = _get_text_config(config)
        self.config = config

        self.model = Gemma4MultiTokenPredictor(
            vllm_config=vllm_config,
            prefix=maybe_prefix(prefix, "draft_model"),
        )

        # lm_head operates in draft-dim.  Tied to embed_tokens at init
        # so load_weights populates both from a single checkpoint entry.
        # After embedding sharing, lm_head.weight still references the
        # original draft-dim tensor.
        self.lm_head = ParallelLMHead(
            text_config.vocab_size,
            text_config.hidden_size,
            prefix=maybe_prefix(prefix, "lm_head"),
        )
        if getattr(config, "tie_word_embeddings", True):
            self.lm_head.weight = self.model.embed_tokens.weight

        self.logits_processor = LogitsProcessor(
            text_config.vocab_size,
            soft_cap=getattr(text_config, "final_logit_softcapping", None),
        )

        if getattr(config, "use_ordered_embeddings", False):
            num_centroids = getattr(config, "num_centroids", 2048)
            top_k = getattr(config, "centroid_intermediate_top_k", 32)
            self.masked_embedding = Gemma4MTPMaskedEmbedder(
                hidden_size=text_config.hidden_size,
                vocab_size=text_config.vocab_size,
                num_centroids=num_centroids,
                centroid_intermediate_top_k=top_k,
            )
            logger.info(
                "Gemma4 MTP: centroids masking enabled "
                "(num_centroids=%d, top_k=%d, active_tokens=%d/%d).",
                num_centroids,
                top_k,
                top_k * (text_config.vocab_size // num_centroids),
                text_config.vocab_size,
            )
        else:
            self.masked_embedding = None
```
**EN:** Method `Gemma4MTP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Gemma4MTP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Gemma4MTP.embed_input_ids` (lines 547-548)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)
```
**EN:** Method `Gemma4MTP.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Gemma4MTP.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `Gemma4MTP.forward` (lines 550-567)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        spec_step_idx: int = 0,
        **kwargs: object,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        return self.model(
            input_ids,
            positions,
            hidden_states,
            intermediate_tensors,
            inputs_embeds,
            spec_step_idx,
        )
```
**EN:** Method `Gemma4MTP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Gemma4MTP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `Gemma4MTP.compute_logits` (lines 579-589)
```python
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
        spec_step_idx: int = 0,
    ) -> torch.Tensor | None:
        if self.masked_embedding is not None:
            return self.masked_embedding(
                hidden_states,
                self._get_full_lm_head_weight(),
            )
        return self.logits_processor(self.lm_head, hidden_states)
```
**EN:** Method `Gemma4MTP.compute_logits` converts hidden states into prediction-ready outputs, often by applying a head, post-processing, or sampling logic.
**CN:** Method `Gemma4MTP.compute_logits` 将隐藏状态转成可预测输出，通常会经过输出头、后处理或采样逻辑。

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
- **Standard library / 标准库**: `from collections.abc import Iterable`
- **Third-party / 第三方**: `import torch`, `from torch import nn`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import (`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding import get_rope`, `from vllm.model_executor.layers.vocab_parallel_embedding import (`, `from vllm.model_executor.model_loader.weight_utils import default_weight_loader`
- **Module note / 模块说明**: **EN:** Inference-only Gemma4 MTP (Multi-Token Prediction) model.  The Gemma4 assistant model is a lightweight decoder that shares KV cache with the target (backbone) model.  All assistant decoder layers are KV-shared: they only have Q projections (no K/V projections or norms), and read K/V from the target model's cache at runtime.  Checkpoint layout (``gemma4_assistant``)::  model.embed_tokens.*          -- token embeddings model.layers.{i}.*            -- decoder layers (Q-only attention + MLP) model.norm.*                  -- final RMSNorm pre_projection.*              -- Linear(2 * backbone_hidden_size, hidden_size) post_projection.*             -- Linear(hidden_size, backbone_hidden_size) lm_head.*                     -- language model head (tied to embed_tokens) masked_embedding.centroids.*  -- centroid projection (when use_ordered_embeddings) masked_embedding.token_ordering -- token-to-centroid mapping buffer **CN:** 模块文档字符串给出的原始说明是：Inference-only Gemma4 MTP (Multi-Token Prediction) model.  The Gemma4 assistant model is a lightweight decoder that shares KV cache with the target (backbone) model.  All assistant decoder layers are KV-shared: they only have Q projections (no K/V projections or norms), and read K/V from the target model's cache at runtime.  Checkpoint layout (``gemma4_assistant``)::  model.embed_tokens.*          -- token embeddings model.layers.{i}.*            -- decoder layers (Q-only attention + MLP) model.norm.*                  -- final RMSNorm pre_projection.*              -- Linear(2 * backbone_hidden_size, hidden_size) post_projection.*             -- Linear(hidden_size, backbone_hidden_size) lm_head.*                     -- language model head (tied to embed_tokens) masked_embedding.centroids.*  -- centroid projection (when use_ordered_embeddings) masked_embedding.token_ordering -- token-to-centroid mapping buffer。
