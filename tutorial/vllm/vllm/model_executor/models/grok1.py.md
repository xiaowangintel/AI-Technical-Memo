# grok1.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/grok1.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for grok1, including architecture wrappers and weight loading logic. / 面向推理的 grok1 vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 26-72)
```python
import math
from collections.abc import Iterable
from itertools import islice
from typing import Any

import torch
import torch.nn.functional as F
from torch import nn

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import get_pp_group, get_tensor_model_parallel_world_size
from vllm.logger import init_logger
from vllm.model_executor.layers.activation import GeluAndMul
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.fused_moe import (
    FusedMoE,
    fused_moe_make_expert_params_mapping,
)
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    MergedColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.logits_processor import LogitsProcessor
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from vllm.model_executor.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from vllm.sequence import IntermediateTensors

from .interfaces import SupportsLoRA, SupportsPP
from .utils import (
    AutoWeightsLoader,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 75-75)
```python
DEFAULT_ATTN_OUTPUT_MULTIPLIER = 0.08838834764831845
```
**EN:** This block defines DEFAULT_ATTN_OUTPUT_MULTIPLIER, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 DEFAULT_ATTN_OUTPUT_MULTIPLIER，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 76-76)
```python
DEFAULT_OUTPUT_MULTIPLIER_SCALE = 0.5773502691896257
```
**EN:** This block defines DEFAULT_OUTPUT_MULTIPLIER_SCALE, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 DEFAULT_OUTPUT_MULTIPLIER_SCALE，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Constants / assignments (lines 77-77)
```python
DEFAULT_EMBEDDING_MULTIPLIER_SCALE = 78.38367176906169
```
**EN:** This block defines DEFAULT_EMBEDDING_MULTIPLIER_SCALE, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 DEFAULT_EMBEDDING_MULTIPLIER_SCALE，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Function `_get_num_experts` (lines 83-84)
```python
def _get_num_experts(config) -> int:
    return getattr(config, "num_experts", getattr(config, "num_local_experts", 8))
```
**EN:** Function `_get_num_experts` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_get_num_experts` 封装了该模块中的一段可复用核心逻辑。

### Function `_get_moe_intermediate_size` (lines 87-88)
```python
def _get_moe_intermediate_size(config) -> int:
    return getattr(config, "moe_intermediate_size", config.intermediate_size)
```
**EN:** Function `_get_moe_intermediate_size` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `_get_moe_intermediate_size` 封装了该模块中的一段可复用核心逻辑。

### Class `Grok1MLP` (lines 142-171)
```python
class Grok1MLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=hidden_size,
            output_sizes=[intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            input_size=intermediate_size,
            output_size=hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
        )
        self.act_fn = GeluAndMul()
```
**EN:** Class `Grok1MLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Grok1MLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Grok1MLP.__init__` (lines 143-165)
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=hidden_size,
            output_sizes=[intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            input_size=intermediate_size,
            output_size=hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.down_proj",
        )
        self.act_fn = GeluAndMul()
```
**EN:** Method `Grok1MLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Grok1MLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Grok1MLP.forward` (lines 167-171)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x, _ = self.gate_up_proj(x)
        x = self.act_fn(x)
        x, _ = self.down_proj(x)
        return x
```
**EN:** Method `Grok1MLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Grok1MLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Grok1MoE` (lines 174-234)
```python
class Grok1MoE(nn.Module):
    """A tensor-parallel MoE implementation for Grok1 that shards each expert
    across all ranks.

    Each expert's weights are sharded across all ranks and a fused MoE
    kernel is used for the forward pass, and finally we reduce the outputs
    across ranks.
    """

    def __init__(
        self,
        num_experts: int,
        top_k: int,
        hidden_size: int,
        intermediate_size: int,
        router_logit_soft_cap: float = 0.0,
        params_dtype: torch.dtype | None = None,
        quant_config: QuantizationConfig | None = None,
        tp_size: int | None = None,
        renormalize: bool = False,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = hidden_size
```
**EN:** Class `Grok1MoE` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Grok1MoE` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Grok1MoE.__init__` (lines 183-221)
```python
    def __init__(
        self,
        num_experts: int,
        top_k: int,
        hidden_size: int,
        intermediate_size: int,
        router_logit_soft_cap: float = 0.0,
        params_dtype: torch.dtype | None = None,
        quant_config: QuantizationConfig | None = None,
        tp_size: int | None = None,
        renormalize: bool = False,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = hidden_size

        # Gate always runs at half / full precision for now.
        self.gate = ReplicatedLinear(
            hidden_size,
            num_experts,
            bias=False,
            params_dtype=params_dtype,
            quant_config=None,
            prefix=f"{prefix}.gate",
        )

        self.experts = FusedMoE(
            num_experts=num_experts,
            top_k=top_k,
            hidden_size=hidden_size,
            intermediate_size=intermediate_size,
            params_dtype=params_dtype,
            renormalize=renormalize,
            quant_config=quant_config,
            tp_size=tp_size,
            activation="gelu",
            prefix=f"{prefix}.experts",
        )
        self.router_logit_soft_cap = router_logit_soft_cap
```
**EN:** Method `Grok1MoE.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Grok1MoE.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Grok1MoE.forward` (lines 223-234)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        # NOTE: hidden_states can have either 1D or 2D shape.
        orig_shape = hidden_states.shape
        hidden_states = hidden_states.view(-1, self.hidden_size)
        # router_logits: (num_tokens, n_experts)
        router_logits, _ = self.gate(hidden_states)
        if self.router_logit_soft_cap > 0:
            router_logits = self.router_logit_soft_cap * F.tanh(
                router_logits / self.router_logit_soft_cap
            )
        final_hidden_states = self.experts(hidden_states, router_logits)
        return final_hidden_states.view(orig_shape)
```
**EN:** Method `Grok1MoE.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Grok1MoE.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Grok1Attention` (lines 237-331)
```python
class Grok1Attention(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        max_position: int = 4096 * 32,
        rope_parameters: dict[str, Any] | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        config=None,  # Added config parameter
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.config = config  # Store config reference
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
```
**EN:** Class `Grok1Attention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Grok1Attention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Grok1Attention.__init__` (lines 238-318)
```python
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        max_position: int = 4096 * 32,
        rope_parameters: dict[str, Any] | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        config=None,  # Added config parameter
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.config = config  # Store config reference
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5

        self.qkv_proj = QKVParallelLinear(
            hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )
        self.o_proj = RowParallelLinear(
            self.total_num_heads * self.head_dim,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.o_proj",
        )
        self.rotary_emb = get_rope(
            self.head_dim,
            max_position=max_position,
            rope_parameters=rope_parameters,
            is_neox_style=True,
        )

        attn_logits_soft_cap = max(getattr(config, "attn_logit_softcapping", 30.0), 0.0)
        attn_logit_softcapping_method = getattr(
            config, "attn_logit_softcapping_method", None
# ... truncated for analysis ...
```
**EN:** Method `Grok1Attention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Grok1Attention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Grok1Attention.forward` (lines 320-331)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v)
        output, _ = self.o_proj(attn_output)
        output *= self.attn_multiplier
        return output
```
**EN:** Method `Grok1Attention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Grok1Attention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Grok1DecoderLayer` (lines 334-433)
```python
class Grok1DecoderLayer(nn.Module):
    def __init__(
        self,
        config,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        # Check for fp8 quantization
        self.use_fp8 = False
        if quant_config is not None:
            self.use_fp8 = getattr(quant_config, "is_fp8_w8a8", lambda: False)()
            if not self.use_fp8 and hasattr(quant_config, "is_fp8"):
                self.use_fp8 = quant_config.is_fp8

        self.attn = Grok1Attention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            max_position=config.max_position_embeddings,
            num_kv_heads=config.num_key_value_heads,
            rope_parameters=_get_rope_parameters(config),
            cache_config=cache_config,
            quant_config=quant_config,
```
**EN:** Class `Grok1DecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `Grok1DecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `Grok1DecoderLayer.__init__` (lines 335-399)
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
        # Check for fp8 quantization
        self.use_fp8 = False
        if quant_config is not None:
            self.use_fp8 = getattr(quant_config, "is_fp8_w8a8", lambda: False)()
            if not self.use_fp8 and hasattr(quant_config, "is_fp8"):
                self.use_fp8 = quant_config.is_fp8

        self.attn = Grok1Attention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            max_position=config.max_position_embeddings,
            num_kv_heads=config.num_key_value_heads,
            rope_parameters=_get_rope_parameters(config),
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.attn",
            config=config,
        )  # Pass config to Grok1Attention

        num_experts = _get_num_experts(config)
        num_experts_per_tok = getattr(config, "num_experts_per_tok", 2)
        moe_intermediate_size = _get_moe_intermediate_size(config)
        moe_renormalize = _get_moe_renormalize(config)

        self.moe_block = Grok1MoE(
            num_experts=num_experts,
            top_k=num_experts_per_tok,
            hidden_size=config.hidden_size,
            intermediate_size=moe_intermediate_size,
            router_logit_soft_cap=max(
                getattr(
                    config,
                    "router_logit_softcapping",
                    DEFAULT_ROUTER_LOGIT_SOFTCAP,
                ),
                0.0,
            ),
            quant_config=quant_config,
            renormalize=moe_renormalize,
            prefix=f"{prefix}.moe_block",
        )
        self.residual_moe = getattr(config, "residual_moe", False)
        self.residual_moe_scale = 1.0 / math.sqrt(2.0)

        self.pre_attn_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.post_attn_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.pre_moe_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.post_moe_norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.mlp = None
        if self.residual_moe:
            self.mlp = Grok1MLP(
# ... truncated for analysis ...
```
**EN:** Method `Grok1DecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Grok1DecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Grok1DecoderLayer.forward` (lines 401-433)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: torch.Tensor | None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        # Self Attention
        if residual is None:
            residual = hidden_states
            hidden_states = self.pre_attn_norm(hidden_states)
        else:
            hidden_states, residual = self.pre_attn_norm(hidden_states, residual)

        hidden_states = self.attn(
            positions=positions,
            hidden_states=hidden_states,
        )

        # Post attention normalization
        hidden_states = self.post_attn_norm(hidden_states)

        # MoE block with normalization
        hidden_states, residual = self.pre_moe_norm(hidden_states, residual)
        if self.residual_moe:
            assert self.mlp is not None
            hidden_states = (
                self.moe_block(hidden_states) + self.mlp(hidden_states)
            ) * self.residual_moe_scale
        else:
            hidden_states = self.moe_block(hidden_states)
        hidden_states = self.post_moe_norm(hidden_states)

        return hidden_states, residual
```
**EN:** Method `Grok1DecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `Grok1DecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `Grok1Model` (lines 437-637)
```python
@support_torch_compile
class Grok1Model(nn.Module):
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        ckpt_gate_proj_name: str = "linear",
        ckpt_down_proj_name: str = "linear_1",
        ckpt_up_proj_name: str = "linear_v",
        weight_name_remapping: dict[str, str] | None = None,
    ):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.quant_config = quant_config

        # Store expert naming for weight loading
        self.ckpt_gate_proj_name = ckpt_gate_proj_name
        self.ckpt_down_proj_name = ckpt_down_proj_name
        self.ckpt_up_proj_name = ckpt_up_proj_name
```
**EN:** Class `Grok1Model` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, forward, get_expert_mapping, load_weights.
**CN:** 类 `Grok1Model` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, forward, get_expert_mapping, load_weights。

### Method `Grok1Model.__init__` (lines 438-486)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        ckpt_gate_proj_name: str = "linear",
        ckpt_down_proj_name: str = "linear_1",
        ckpt_up_proj_name: str = "linear_v",
        weight_name_remapping: dict[str, str] | None = None,
    ):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.quant_config = quant_config

        # Store expert naming for weight loading
        self.ckpt_gate_proj_name = ckpt_gate_proj_name
        self.ckpt_down_proj_name = ckpt_down_proj_name
        self.ckpt_up_proj_name = ckpt_up_proj_name
        self.weight_name_remapping = weight_name_remapping or {}

        self.vocab_size = config.vocab_size

        self.embedding_multiplier_scale = getattr(
            config, "embedding_multiplier_scale", DEFAULT_EMBEDDING_MULTIPLIER_SCALE
        )

        self.embed_tokens = VocabParallelEmbedding(
            self.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
        )

        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers,
            lambda prefix: Grok1DecoderLayer(
                config, cache_config, quant_config=quant_config, prefix=prefix
            ),
            prefix=f"{prefix}.layers",
        )

        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.make_empty_intermediate_tensors = make_empty_intermediate_tensors_factory(
            ["hidden_states", "residual"], config.hidden_size
        )
```
**EN:** Method `Grok1Model.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `Grok1Model.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `Grok1Model.embed_input_ids` (lines 488-491)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        hidden_states = self.embed_tokens(input_ids)
        hidden_states = hidden_states * self.embedding_multiplier_scale
        return hidden_states
```
**EN:** Method `Grok1Model.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `Grok1Model.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Class `GrokBaseForCausalLM` (lines 640-737)
```python
class GrokBaseForCausalLM(nn.Module, SupportsLoRA, SupportsPP):
    """Base class for Grok models with shared logic."""

    fall_back_to_pt_during_load = False

    # Subclasses should override these
    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
    }

    # Expert weight naming - subclasses override these
    ckpt_gate_proj_name: str = "linear"
    ckpt_down_proj_name: str = "linear_1"
    ckpt_up_proj_name: str = "linear_v"

    def get_weight_name_remapping(self) -> dict[str, str]:
        """Return weight name remapping for this version. Override in subclasses."""
        return {}

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
```
**EN:** Class `GrokBaseForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsLoRA, SupportsPP. Key methods include get_weight_name_remapping, __init__, embed_input_ids, forward, compute_logits, load_weights.
**CN:** 类 `GrokBaseForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsLoRA、SupportsPP。 关键方法包括 get_weight_name_remapping, __init__, embed_input_ids, forward, compute_logits, load_weights。

### Method `GrokBaseForCausalLM.get_weight_name_remapping` (lines 659-661)
```python
    def get_weight_name_remapping(self) -> dict[str, str]:
        """Return weight name remapping for this version. Override in subclasses."""
        return {}
```
**EN:** Method `GrokBaseForCausalLM.get_weight_name_remapping` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders. The docstring says: Return weight name remapping for this version.
**CN:** Method `GrokBaseForCausalLM.get_weight_name_remapping` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。 文档字符串表达的核心意思是：Return weight name remapping for this version。

### Method `GrokBaseForCausalLM.__init__` (lines 663-702)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.quant_config = quant_config

        self.model = Grok1Model(
            vllm_config=vllm_config,
            prefix=maybe_prefix(prefix, "model"),
            ckpt_gate_proj_name=self.ckpt_gate_proj_name,
            ckpt_down_proj_name=self.ckpt_down_proj_name,
            ckpt_up_proj_name=self.ckpt_up_proj_name,
            weight_name_remapping=self.get_weight_name_remapping(),
        )

        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "lm_head"),
        )

        if self.config.tie_word_embeddings:
            self.lm_head.weight = self.model.embed_tokens.weight

        self.output_multiplier_scale = getattr(
            config, "output_multiplier_scale", DEFAULT_OUTPUT_MULTIPLIER_SCALE
        )
        self.logits_processor = LogitsProcessor(
            config.vocab_size,
            scale=self.output_multiplier_scale,
            soft_cap=getattr(config, "final_logit_softcapping", None),
        )

        self.make_empty_intermediate_tensors = (
            self.model.make_empty_intermediate_tensors
        )
```
**EN:** Method `GrokBaseForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GrokBaseForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `Grok1ForCausalLM` (lines 740-750)
```python
class Grok1ForCausalLM(GrokBaseForCausalLM):
    """Grok1-specific implementation."""

    # Grok1 expert weight naming
    ckpt_gate_proj_name = "linear"
    ckpt_down_proj_name = "linear_1"
    ckpt_up_proj_name = "linear_v"

    def get_weight_name_remapping(self) -> dict[str, str]:
        # Grok1 uses standard naming, no remapping needed
        return {}
```
**EN:** Class `Grok1ForCausalLM` organizes related behavior for this model family or helper component. It inherits from GrokBaseForCausalLM. Key methods include get_weight_name_remapping.
**CN:** 类 `Grok1ForCausalLM` 用于组织该模型族或辅助组件的相关行为。 它继承自 GrokBaseForCausalLM。 关键方法包括 get_weight_name_remapping。

### Method `Grok1ForCausalLM.get_weight_name_remapping` (lines 748-750)
```python
    def get_weight_name_remapping(self) -> dict[str, str]:
        # Grok1 uses standard naming, no remapping needed
        return {}
```
**EN:** Method `Grok1ForCausalLM.get_weight_name_remapping` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `Grok1ForCausalLM.get_weight_name_remapping` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `Grok2ForCausalLM` (lines 753-779)
```python
class Grok2ForCausalLM(GrokBaseForCausalLM):
    """Grok2-specific implementation."""

    # Grok2 has additional packed modules for MLP
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

    # Grok2 expert weight naming
    ckpt_gate_proj_name = "w1"
    ckpt_down_proj_name = "w2"
    ckpt_up_proj_name = "w3"

    def get_weight_name_remapping(self) -> dict[str, str]:
        # Grok2 checkpoint uses different naming conventions
        return {
            ".self_attn.": ".attn.",
```
**EN:** Class `Grok2ForCausalLM` organizes related behavior for this model family or helper component. It inherits from GrokBaseForCausalLM. Key methods include get_weight_name_remapping.
**CN:** 类 `Grok2ForCausalLM` 用于组织该模型族或辅助组件的相关行为。 它继承自 GrokBaseForCausalLM。 关键方法包括 get_weight_name_remapping。

### Method `Grok2ForCausalLM.get_weight_name_remapping` (lines 774-779)
```python
    def get_weight_name_remapping(self) -> dict[str, str]:
        # Grok2 checkpoint uses different naming conventions
        return {
            ".self_attn.": ".attn.",
            ".block_sparse_moe.": ".moe_block.",
        }
```
**EN:** Method `Grok2ForCausalLM.get_weight_name_remapping` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `Grok2ForCausalLM.get_weight_name_remapping` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `GrokForCausalLM` (lines 789-804)
```python
class GrokForCausalLM(GrokBaseForCausalLM):
    """Factory class that dispatches to version-specific implementation."""

    def __new__(cls, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_config
        version = _get_grok_version(config)

        instance_cls = _GROK_VERSIONS.get(version)
        if instance_cls is None:
            raise ValueError(f"Unsupported Grok version: {version}")

        # Merge class attributes for LoRA/quantization compatibility
        cls.packed_modules_mapping = dict(cls.packed_modules_mapping)
        cls.packed_modules_mapping.update(instance_cls.packed_modules_mapping)

        return instance_cls(vllm_config=vllm_config, prefix=prefix)
```
**EN:** Class `GrokForCausalLM` organizes related behavior for this model family or helper component. It inherits from GrokBaseForCausalLM. Key methods include __new__.
**CN:** 类 `GrokForCausalLM` 用于组织该模型族或辅助组件的相关行为。 它继承自 GrokBaseForCausalLM。 关键方法包括 __new__。

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
- **Standard library / 标准库**: `import math`, `from collections.abc import Iterable`, `from itertools import islice`, `from typing import Any`
- **Third-party / 第三方**: `import torch`, `import torch.nn.functional as F`, `from torch import nn`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import get_pp_group, get_tensor_model_parallel_world_size`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.activation import GeluAndMul`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.fused_moe import (`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding import get_rope`
- **Module note / 模块说明**: **EN:** Inference-only Grok (Grok1/Grok2) model. **CN:** 模块文档字符串给出的原始说明是：Inference-only Grok (Grok1/Grok2) model.。
