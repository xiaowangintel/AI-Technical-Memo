# cohere2_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/cohere2_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Mixture-of-experts model implementation for cohere2_moe, covering expert routing, transformer blocks, and weight loading. / 面向 cohere2_moe 的 MoE 模型实现，涵盖专家路由、Transformer 模块与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-48)
```python
from collections.abc import Iterable
from itertools import islice

import torch
from torch import nn
from transformers import CohereConfig

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import (
    get_pp_group,
    get_tensor_model_parallel_world_size,
)
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.fused_moe import FusedMoE
from vllm.model_executor.layers.linear import (
    MergedColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.layers.vocab_parallel_embedding import VocabParallelEmbedding
from vllm.model_executor.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
    row_parallel_weight_loader,
)
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform
from vllm.sequence import IntermediateTensors

from .commandr import LayerNorm
from .interfaces import SupportsPP, SupportsQuant
from .utils import (
    AutoWeightsLoader,
    extract_layer_index,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `token_choice_with_bias` (lines 52-67)
```python
@torch.compile(backend=current_platform.simple_compile_backend)
def token_choice_with_bias(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
):
    """Sigmoid -> top-k (-> renormalize) custom routing for Cohere2Moe."""
    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"

    scores = gating_output.float().sigmoid()
    topk_weights, topk_ids = torch.topk(scores, k=topk, dim=-1, sorted=False)

    if renormalize:
        topk_weights = topk_weights / topk_weights.sum(dim=-1, keepdim=True)

    return topk_weights.to(torch.float32), topk_ids.to(torch.int32)
```
**EN:** Function `token_choice_with_bias` encapsulates a focused piece of reusable logic inside this module. The docstring says: Sigmoid -> top-k (-> renormalize) custom routing for Cohere2Moe.
**CN:** Function `token_choice_with_bias` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Sigmoid -> top-k (-> renormalize) custom routing for Cohere2Moe。

### Function `rms_norm_func` (lines 71-77)
```python
@torch.compile(backend=current_platform.simple_compile_backend)
def rms_norm_func(hidden_states, weight, variance_epsilon):
    input_dtype = hidden_states.dtype
    hidden_states = hidden_states.to(torch.float32)
    variance = hidden_states.pow(2).mean(-1, keepdim=True)
    hidden_states = hidden_states * torch.rsqrt(variance + variance_epsilon)
    hidden_states = weight.to(torch.float32) * hidden_states
    return hidden_states.to(input_dtype)
```
**EN:** Function `rms_norm_func` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `rms_norm_func` 封装了该模块中的一段可复用核心逻辑。

### Function `select_norm_impl` (lines 92-98)
```python
def select_norm_impl(config: CohereConfig) -> tuple[type[nn.Module], float]:
    """Returns (norm_class, eps). Uses RMSNorm when config.rms_norm_eps is set,
    otherwise falls back to LayerNorm with config.layer_norm_eps."""
    rms_eps = getattr(config, "rms_norm_eps", None)
    if rms_eps is not None:
        return RMSNorm, rms_eps
    return LayerNorm, config.layer_norm_eps
```
**EN:** Function `select_norm_impl` encapsulates a focused piece of reusable logic inside this module. The docstring says: Returns (norm_class, eps).
**CN:** Function `select_norm_impl` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Returns (norm_class, eps)。

### Class `RMSNorm` (lines 80-89)
```python
class RMSNorm(nn.Module):
    def __init__(self, param_shape=None, eps=1e-6):
        super().__init__()
        self.weight = nn.Parameter(torch.ones(param_shape))
        self.variance_epsilon = eps
        set_weight_attrs(self.weight, {"weight_loader": row_parallel_weight_loader})

    def forward(self, hidden_states, residuals=None):
        hidden_states = rms_norm_func(hidden_states, self.weight, self.variance_epsilon)
        return hidden_states, residuals
```
**EN:** Class `RMSNorm` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `RMSNorm` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `RMSNorm.__init__` (lines 81-85)
```python
    def __init__(self, param_shape=None, eps=1e-6):
        super().__init__()
        self.weight = nn.Parameter(torch.ones(param_shape))
        self.variance_epsilon = eps
        set_weight_attrs(self.weight, {"weight_loader": row_parallel_weight_loader})
```
**EN:** Method `RMSNorm.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `RMSNorm.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `RMSNorm.forward` (lines 87-89)
```python
    def forward(self, hidden_states, residuals=None):
        hidden_states = rms_norm_func(hidden_states, self.weight, self.variance_epsilon)
        return hidden_states, residuals
```
**EN:** Method `RMSNorm.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `RMSNorm.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Cohere2MoeMLP` (lines 101-141)
```python
class Cohere2MoeMLP(nn.Module):
    """Cohere MLP used as shared experts in the MoE block."""

    def __init__(
        self,
        config: CohereConfig,
        intermediate_size: int | None = None,
        quant_config: QuantizationConfig | None = None,
        reduce_results: bool = False,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.intermediate_size = (
            intermediate_size
            if intermediate_size is not None
            else config.intermediate_size
        )
        self.gate_up_proj = MergedColumnParallelLinear(
            self.hidden_size,
            [self.intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
```
**EN:** Class `Cohere2MoeMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Cohere2MoeMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Cohere2MoeMLP.__init__` (lines 104-135)
```python
    def __init__(
        self,
        config: CohereConfig,
        intermediate_size: int | None = None,
        quant_config: QuantizationConfig | None = None,
        reduce_results: bool = False,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.intermediate_size = (
            intermediate_size
            if intermediate_size is not None
            else config.intermediate_size
        )
        self.gate_up_proj = MergedColumnParallelLinear(
            self.hidden_size,
            [self.intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            self.intermediate_size,
            self.hidden_size,
            bias=False,
            quant_config=quant_config,
            reduce_results=reduce_results,
            prefix=f"{prefix}.down_proj",
        )
        self.act_fn = SiluAndMul()
```
**EN:** Method `Cohere2MoeMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Cohere2MoeMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Cohere2MoeMLP.forward` (lines 137-141)
```python
    def forward(self, x):
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** Method `Cohere2MoeMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Cohere2MoeMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Cohere2MoeAttention` (lines 144-242)
```python
class Cohere2MoeAttention(nn.Module):
    """Cohere MoE attention with sliding-window interleave."""

    def __init__(
        self,
        config: CohereConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        tp_size = get_tensor_model_parallel_world_size()
        self.config = config
        self.layer_idx = extract_layer_index(prefix)
        self.hidden_size = config.hidden_size
        self.total_num_heads = config.num_attention_heads
        self.num_heads = self.total_num_heads // tp_size
        self.head_dim = getattr(
            config, "head_dim", self.hidden_size // self.total_num_heads
        )
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0
```
**EN:** Class `Cohere2MoeAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Cohere2MoeAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Cohere2MoeAttention.__init__` (lines 147-229)
```python
    def __init__(
        self,
        config: CohereConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        tp_size = get_tensor_model_parallel_world_size()
        self.config = config
        self.layer_idx = extract_layer_index(prefix)
        self.hidden_size = config.hidden_size
        self.total_num_heads = config.num_attention_heads
        self.num_heads = self.total_num_heads // tp_size
        self.head_dim = getattr(
            config, "head_dim", self.hidden_size // self.total_num_heads
        )
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.max_position_embeddings = getattr(
            config, "model_max_length", None
        ) or getattr(config, "max_position_embeddings", 8192)
        self.qkv_proj = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )
        self.o_proj = RowParallelLinear(
            self.total_num_heads * self.head_dim,
            self.hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.o_proj",
        )
        self.rotary_emb = get_rope(
            self.head_dim,
            max_position=self.max_position_embeddings,
            rope_parameters=config.rope_parameters,
            is_neox_style=False,
        )

        self.sliding_window = None
        layer_types = getattr(config, "layer_types", None)
        if (
            layer_types is not None
            and layer_types[self.layer_idx] == "sliding_attention"
        ):
            self.sliding_window = config.sliding_window

# ... truncated for analysis ...
```
**EN:** Method `Cohere2MoeAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Cohere2MoeAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Cohere2MoeAttention.forward` (lines 231-242)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        if self.sliding_window or self.force_rope:
            q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** Method `Cohere2MoeAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Cohere2MoeAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Cohere2Moe` (lines 245-323)
```python
class Cohere2Moe(nn.Module):
    """Tensor-parallel MoE block for Cohere2Moe with shared experts."""

    def __init__(
        self,
        config: CohereConfig,
        params_dtype: torch.dtype | None = None,
        quant_config: QuantizationConfig | None = None,
        tp_size: int | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.tp_size = get_tensor_model_parallel_world_size()

        if self.tp_size > config.num_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {config.num_experts}."
            )

        if (
            hasattr(config, "expert_selection_fn")
            and config.expert_selection_fn == "sigmoid"
        ):
```
**EN:** Class `Cohere2Moe` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Cohere2Moe` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Cohere2Moe.__init__` (lines 248-312)
```python
    def __init__(
        self,
        config: CohereConfig,
        params_dtype: torch.dtype | None = None,
        quant_config: QuantizationConfig | None = None,
        tp_size: int | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.tp_size = get_tensor_model_parallel_world_size()

        if self.tp_size > config.num_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {config.num_experts}."
            )

        if (
            hasattr(config, "expert_selection_fn")
            and config.expert_selection_fn == "sigmoid"
        ):
            self.custom_routing_function = token_choice_with_bias
        else:
            self.custom_routing_function = None

        self.gate = ReplicatedLinear(
            config.hidden_size,
            config.num_experts,
            bias=False,
            params_dtype=params_dtype,
            quant_config=None,
            prefix=f"{prefix}.gate",
        )

        if hasattr(config, "num_shared_experts") and config.num_shared_experts > 0:
            self.shared_experts = Cohere2MoeMLP(
                config=config,
                intermediate_size=config.intermediate_size * config.num_shared_experts,
                quant_config=quant_config,
                prefix=f"{prefix}.shared_experts",
            )
            self.shared_expert_combination_strategy = getattr(
                config, "shared_expert_combination_strategy", "sum"
            )
            assert self.shared_expert_combination_strategy in ("average", "sum"), (
                "shared_expert_combination_strategy must be one of ['average', 'sum']"
            )
        else:
            self.shared_experts = None
            self.shared_expert_combination_strategy = None

        self.experts = FusedMoE(
            num_experts=config.num_experts,
            top_k=config.num_experts_per_tok,
            hidden_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            params_dtype=params_dtype,
            renormalize=getattr(config, "norm_topk_prob", True),
            quant_config=quant_config,
# ... truncated for analysis ...
```
**EN:** Method `Cohere2Moe.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Cohere2Moe.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Cohere2Moe.forward` (lines 314-323)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        orig_shape = hidden_states.shape
        hidden_states = hidden_states.view(-1, self.hidden_size)
        router_logits, _ = self.gate(hidden_states)
        # FusedMoE handles shared expert overlap internally and returns
        # shared_output + routed_output when shared_experts is set.
        final_hidden_states = self.experts(hidden_states, router_logits)
        if self.shared_expert_combination_strategy == "average":
            final_hidden_states = final_hidden_states / 2
        return final_hidden_states.view(orig_shape)
```
**EN:** Method `Cohere2Moe.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Cohere2Moe.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Cohere2MoeDecoderLayer` (lines 326-381)
```python
class Cohere2MoeDecoderLayer(nn.Module):
    def __init__(
        self,
        config: CohereConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.layer_idx = extract_layer_index(prefix)

        self.self_attn = Cohere2MoeAttention(
            config,
            cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )

        # Layers before first_k_dense_replace use a dense MLP instead of MoE.
        first_k_dense_replace = getattr(config, "first_k_dense_replace", 0)
        if self.layer_idx < first_k_dense_replace:
            self.mlp = Cohere2MoeMLP(
                config=config,
```
**EN:** Class `Cohere2MoeDecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Cohere2MoeDecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Cohere2MoeDecoderLayer.__init__` (lines 327-364)
```python
    def __init__(
        self,
        config: CohereConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.layer_idx = extract_layer_index(prefix)

        self.self_attn = Cohere2MoeAttention(
            config,
            cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )

        # Layers before first_k_dense_replace use a dense MLP instead of MoE.
        first_k_dense_replace = getattr(config, "first_k_dense_replace", 0)
        if self.layer_idx < first_k_dense_replace:
            self.mlp = Cohere2MoeMLP(
                config=config,
                intermediate_size=getattr(
                    config, "prefix_dense_intermediate_size", config.intermediate_size
                ),
                quant_config=quant_config,
                reduce_results=True,
                prefix=f"{prefix}.mlp",
            )
        else:
            self.mlp = Cohere2Moe(
                config=config, quant_config=quant_config, prefix=f"{prefix}.mlp"
            )

        norm_cls, norm_eps = select_norm_impl(config)
        self.input_layernorm = norm_cls(param_shape=(config.hidden_size,), eps=norm_eps)
```
**EN:** Method `Cohere2MoeDecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Cohere2MoeDecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Cohere2MoeDecoderLayer.forward` (lines 366-381)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        residual = hidden_states
        hidden_states, residual = self.input_layernorm(hidden_states, residual)
        hidden_states_attention = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
        )
        hidden_states_mlp = self.mlp(hidden_states)

        hidden_states = residual + hidden_states_attention + hidden_states_mlp
        return hidden_states, residual
```
**EN:** Method `Cohere2MoeDecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Cohere2MoeDecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Cohere2MoeModel` (lines 385-532)
```python
@support_torch_compile
class Cohere2MoeModel(nn.Module):
    """Transformer decoder for Cohere2Moe."""

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.quant_config = quant_config
        self.vocab_size = config.vocab_size
        self.org_vocab_size = config.vocab_size
        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size, config.hidden_size
        )
        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers,
            lambda prefix: Cohere2MoeDecoderLayer(
                config, cache_config, quant_config, prefix=prefix
            ),
            prefix=f"{prefix}.layers",
        )
```
**EN:** Class `Cohere2MoeModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, get_input_embeddings, forward, load_weights.
**CN:** 类 `Cohere2MoeModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, get_input_embeddings, forward, load_weights。

### Method `Cohere2MoeModel.__init__` (lines 388-413)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.quant_config = quant_config
        self.vocab_size = config.vocab_size
        self.org_vocab_size = config.vocab_size
        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size, config.hidden_size
        )
        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers,
            lambda prefix: Cohere2MoeDecoderLayer(
                config, cache_config, quant_config, prefix=prefix
            ),
            prefix=f"{prefix}.layers",
        )
        norm_cls, norm_eps = select_norm_impl(config)
        self.norm = norm_cls(param_shape=(config.hidden_size,), eps=norm_eps)
        self.make_empty_intermediate_tensors = make_empty_intermediate_tensors_factory(
            ["hidden_states", "residual"], config.hidden_size
        )
```
**EN:** Method `Cohere2MoeModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Cohere2MoeModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Cohere2MoeModel.get_input_embeddings` (lines 415-416)
```python
    def get_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
```
**EN:** Method `Cohere2MoeModel.get_input_embeddings` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Cohere2MoeModel.get_input_embeddings` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `Cohere2MoeModel.forward` (lines 418-442)
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        if get_pp_group().is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.get_input_embeddings(input_ids)
            residual = None
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]
            residual = intermediate_tensors["residual"]
        for layer in islice(self.layers, self.start_layer, self.end_layer):
            hidden_states, residual = layer(positions, hidden_states, residual)
        if not get_pp_group().is_last_rank:
            return IntermediateTensors(
                {"hidden_states": hidden_states, "residual": residual}
            )
        hidden_states, _ = self.norm(hidden_states, residual)
        return hidden_states
```
**EN:** Method `Cohere2MoeModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Cohere2MoeModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `Cohere2MoeModel.load_weights` (lines 444-532)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        expert_params_mapping = FusedMoE.make_expert_params_mapping(
            self,
            ckpt_gate_proj_name="gate_proj",
            ckpt_down_proj_name="down_proj",
            ckpt_up_proj_name="up_proj",
            num_experts=self.config.num_experts,
        )

        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue

            if self.quant_config is not None and (
                scale_name := self.quant_config.get_cache_scale(name)
            ):
                param = params_dict[scale_name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                loaded_weight = (
                    loaded_weight if loaded_weight.dim() == 0 else loaded_weight[0]
                )
                weight_loader(param, loaded_weight)
                loaded_params.add(scale_name)
                continue

            for param_name, shard_name, shard_id in stacked_params_mapping:
                if shard_name not in name:
                    continue
                if "mlp.experts" in name:
                    continue
                name = name.replace(shard_name, param_name)
                if name.endswith(".bias") and name not in params_dict:
                    continue
                if is_pp_missing_parameter(name, self):
                    continue
                param = params_dict[name]
                weight_loader = param.weight_loader
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
                for mapping in expert_params_mapping:
                    param_name, weight_name, expert_id, shard_id = mapping
                    if weight_name not in name:
                        continue
                    name = name.replace(weight_name, param_name)
                    if is_pp_missing_parameter(name, self):
                        continue
                    if (
                        name.endswith(".bias") or name.endswith("_bias")
                    ) and name not in params_dict:
# ... truncated for analysis ...
```
**EN:** Method `Cohere2MoeModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `Cohere2MoeModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `Cohere2MoeForCausalLM` (lines 535-593)
```python
class Cohere2MoeForCausalLM(nn.Module, SupportsPP, SupportsQuant):
    is_text_generation_model = True

    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
        "gate_up_proj": [
            "gate_proj",
            "up_proj",
        ],
    }

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        assert getattr(config, "tie_word_embeddings", True)
        self.unpadded_vocab_size = config.vocab_size
        self.quant_config = quant_config
        self.logits_scale = config.logit_scale
        self.logits_processor = LogitsProcessor(
```
**EN:** Class `Cohere2MoeForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsPP, SupportsQuant. Key methods include __init__, embed_input_ids, get_input_embeddings, forward, compute_logits, load_weights.
**CN:** 类 `Cohere2MoeForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsPP、SupportsQuant。 关键方法包括 __init__, embed_input_ids, get_input_embeddings, forward, compute_logits, load_weights。

### Method `Cohere2MoeForCausalLM.__init__` (lines 550-567)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        assert getattr(config, "tie_word_embeddings", True)
        self.unpadded_vocab_size = config.vocab_size
        self.quant_config = quant_config
        self.logits_scale = config.logit_scale
        self.logits_processor = LogitsProcessor(
            self.unpadded_vocab_size, config.vocab_size, scale=self.logits_scale
        )
        self.model = Cohere2MoeModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        self.make_empty_intermediate_tensors = (
            self.model.make_empty_intermediate_tensors
        )
```
**EN:** Method `Cohere2MoeForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Cohere2MoeForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Cohere2MoeForCausalLM.embed_input_ids` (lines 569-570)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.get_input_embeddings(input_ids)
```
**EN:** Method `Cohere2MoeForCausalLM.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Cohere2MoeForCausalLM.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `Cohere2MoeForCausalLM.get_input_embeddings` (lines 572-573)
```python
    def get_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.get_input_embeddings(input_ids)
```
**EN:** Method `Cohere2MoeForCausalLM.get_input_embeddings` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Cohere2MoeForCausalLM.get_input_embeddings` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `Cohere2MoeForCausalLM.forward` (lines 576-583)
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        return self.model(input_ids, positions, intermediate_tensors, inputs_embeds)
```
**EN:** Method `Cohere2MoeForCausalLM.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Cohere2MoeForCausalLM.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

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
- **Standard library / 标准库**: `from collections.abc import Iterable`, `from itertools import islice`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import CohereConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import (`, `from vllm.model_executor.layers.activation import SiluAndMul`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.fused_moe import FusedMoE`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding import get_rope`, `from vllm.model_executor.layers.vocab_parallel_embedding import VocabParallelEmbedding`, `from vllm.model_executor.model_loader.weight_utils import (`
