# interns1_pro.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/interns1_pro.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for interns1_pro, including architecture wrappers and weight loading logic. / 面向推理的 interns1_pro vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 27-80)
```python
import functools
from collections.abc import Iterable
from typing import Any

import torch
from torch import nn
from transformers import AutoProcessor, PretrainedConfig

from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import (
    get_ep_group,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_gather,
)
from vllm.logger import init_logger
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.fused_moe import (
    FusedMoE,
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
)
from vllm.model_executor.models.utils import sequence_parallel_chunk
from vllm.multimodal import MULTIMODAL_REGISTRY

from .interfaces import MixtureOfExperts
from .qwen3_moe import (
    Qwen3MoeForCausalLM,
)
from .qwen3_vl import (
    Qwen3_VisionTransformer,
    Qwen3VLDummyInputsBuilder,
    Qwen3VLForConditionalGeneration,
    Qwen3VLMultiModalProcessor,
    Qwen3VLProcessingInfo,
)
from .qwen3_vl_moe import Qwen3MoeLLMModel
from .utils import (
    AutoWeightsLoader,
    WeightsMapper,
    extract_layer_index,
    maybe_prefix,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `InternS1ProProcessingInfo` (lines 85-90)
```python
class InternS1ProProcessingInfo(Qwen3VLProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config()

    def get_hf_processor(self, **kwargs: object) -> AutoProcessor:
        return self.ctx.get_hf_processor(**kwargs)
```
**EN:** Class `InternS1ProProcessingInfo` organizes related behavior for this model family or helper component. It inherits from Qwen3VLProcessingInfo. Key methods include get_hf_config, get_hf_processor.
**CN:** 类 `InternS1ProProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 Qwen3VLProcessingInfo。 关键方法包括 get_hf_config, get_hf_processor。

### Method `InternS1ProProcessingInfo.get_hf_config` (lines 86-87)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config()
```
**EN:** Method `InternS1ProProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternS1ProProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `InternS1ProProcessingInfo.get_hf_processor` (lines 89-90)
```python
    def get_hf_processor(self, **kwargs: object) -> AutoProcessor:
        return self.ctx.get_hf_processor(**kwargs)
```
**EN:** Method `InternS1ProProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternS1ProProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Class `InternS1ProMoeMLP` (lines 93-129)
```python
class InternS1ProMoeMLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: QuantizationConfig | None = None,
        reduce_results: bool = True,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            reduce_results=reduce_results,
            prefix=f"{prefix}.down_proj",
```
**EN:** Class `InternS1ProMoeMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `InternS1ProMoeMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `InternS1ProMoeMLP.__init__` (lines 94-123)
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: QuantizationConfig | None = None,
        reduce_results: bool = True,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            reduce_results=reduce_results,
            prefix=f"{prefix}.down_proj",
        )
        if hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()
```
**EN:** Method `InternS1ProMoeMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternS1ProMoeMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternS1ProMoeMLP.forward` (lines 125-129)
```python
    def forward(self, x):
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** Method `InternS1ProMoeMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternS1ProMoeMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternS1ProMoeSparseMoeBlock` (lines 132-262)
```python
class InternS1ProMoeSparseMoeBlock(nn.Module):
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
    ):
        super().__init__()

        config = vllm_config.model_config.hf_text_config
        parallel_config = vllm_config.parallel_config
        quant_config = vllm_config.quant_config

        self.tp_size = get_tensor_model_parallel_world_size()

        self.ep_group = get_ep_group().device_group
        self.ep_rank = get_ep_group().rank_in_group
        self.ep_size = self.ep_group.size()
        self.n_routed_experts = config.num_experts

        self.is_sequence_parallel = parallel_config.use_sequence_parallel_moe

        if self.tp_size > config.num_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {config.num_experts}."
```
**EN:** Class `InternS1ProMoeSparseMoeBlock` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, get_group_offsets, _custom_routing_function, forward.
**CN:** 类 `InternS1ProMoeSparseMoeBlock` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, get_group_offsets, _custom_routing_function, forward。

### Method `InternS1ProMoeSparseMoeBlock.__init__` (lines 133-195)
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        prefix: str = "",
    ):
        super().__init__()

        config = vllm_config.model_config.hf_text_config
        parallel_config = vllm_config.parallel_config
        quant_config = vllm_config.quant_config

        self.tp_size = get_tensor_model_parallel_world_size()

        self.ep_group = get_ep_group().device_group
        self.ep_rank = get_ep_group().rank_in_group
        self.ep_size = self.ep_group.size()
        self.n_routed_experts = config.num_experts

        self.is_sequence_parallel = parallel_config.use_sequence_parallel_moe

        if self.tp_size > config.num_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {config.num_experts}."
            )

        # Load balancing settings.
        eplb_config = vllm_config.parallel_config.eplb_config
        self.enable_eplb = parallel_config.enable_eplb

        self.n_logical_experts = self.n_routed_experts
        self.n_redundant_experts = eplb_config.num_redundant_experts
        self.n_physical_experts = self.n_logical_experts + self.n_redundant_experts
        self.n_local_physical_experts = self.n_physical_experts // self.ep_size

        self.physical_expert_start = self.ep_rank * self.n_local_physical_experts
        self.physical_expert_end = (
            self.physical_expert_start + self.n_local_physical_experts
        )

        # For custom routing function
        self.n_groups = getattr(config, "router_n_groups", -1)

        self.experts = FusedMoE(
            num_experts=self.n_routed_experts,
            top_k=config.num_experts_per_tok,
            hidden_size=config.hidden_size,
            intermediate_size=config.moe_intermediate_size,
            renormalize=config.norm_topk_prob,
            quant_config=quant_config,
            prefix=f"{prefix}.experts",
            enable_eplb=self.enable_eplb,
            num_redundant_experts=self.n_redundant_experts,
            is_sequence_parallel=self.is_sequence_parallel,
            custom_routing_function=self._custom_routing_function,
        )

        self.gate = ReplicatedLinear(
            config.hidden_size,
            config.num_experts,
# ... truncated for analysis ...
```
**EN:** Method `InternS1ProMoeSparseMoeBlock.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternS1ProMoeSparseMoeBlock.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternS1ProMoeSparseMoeBlock.get_group_offsets` (lines 199-203)
```python
    @staticmethod
    @functools.lru_cache
    def get_group_offsets(n_groups: int, group_size: int, device: str):
        group_offsets = (torch.arange(n_groups, device=device) * group_size).view(
            1, -1, 1
        )  # [1, n_groups, 1]
        return group_offsets
```
**EN:** Method `InternS1ProMoeSparseMoeBlock.get_group_offsets` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternS1ProMoeSparseMoeBlock.get_group_offsets` 封装了该模块中的一段可复用核心逻辑。

### Method `InternS1ProMoeSparseMoeBlock.forward` (lines 238-262)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        assert hidden_states.dim() <= 2, (
            "InternS1ProMoeSparseMoeBlock only supports 1D or 2D inputs"
        )
        is_input_1d = hidden_states.dim() == 1
        num_tokens, hidden_dim = hidden_states.shape
        hidden_states = hidden_states.view(-1, hidden_dim)

        if self.is_sequence_parallel:
            hidden_states = sequence_parallel_chunk(hidden_states)

        # router_logits: (num_tokens, n_experts)
        router_logits, _ = self.gate(hidden_states)
        final_hidden_states = self.experts(
            hidden_states=hidden_states, router_logits=router_logits
        )

        if self.is_sequence_parallel:
            final_hidden_states = tensor_model_parallel_all_gather(
                final_hidden_states, 0
            )
            final_hidden_states = final_hidden_states[:num_tokens]

        # return to 1d if input is 1d
        return final_hidden_states.squeeze(0) if is_input_1d else final_hidden_states
```
**EN:** Method `InternS1ProMoeSparseMoeBlock.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternS1ProMoeSparseMoeBlock.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternS1ProMoeAttention` (lines 265-367)
```python
class InternS1ProMoeAttention(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        rope_parameters: dict[str, Any],
        max_position_embeddings: int = 32768,
        head_dim: int | None = None,
        rms_norm_eps: float = 1e-06,
        qkv_bias: bool = False,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        dual_chunk_attention_config: dict[str, Any] | None = None,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
```
**EN:** Class `InternS1ProMoeAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `InternS1ProMoeAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `InternS1ProMoeAttention.__init__` (lines 266-347)
```python
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        rope_parameters: dict[str, Any],
        max_position_embeddings: int = 32768,
        head_dim: int | None = None,
        rms_norm_eps: float = 1e-06,
        qkv_bias: bool = False,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        dual_chunk_attention_config: dict[str, Any] | None = None,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
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
        self.head_dim = head_dim or (hidden_size // self.total_num_heads)
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.max_position_embeddings = max_position_embeddings
        self.dual_chunk_attention_config = dual_chunk_attention_config

        self.qkv_proj = QKVParallelLinear(
            hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=qkv_bias,
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

        rope_parameters["num_key_value_heads"] = self.num_kv_heads
        self.rotary_emb = get_rope(
            self.head_dim,
            max_position=max_position_embeddings,
# ... truncated for analysis ...
```
**EN:** Method `InternS1ProMoeAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternS1ProMoeAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternS1ProMoeAttention.forward` (lines 349-367)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        # Add qk-norm
        q_by_head = q.view(*q.shape[:-1], q.shape[-1] // self.head_dim, self.head_dim)
        q_by_head = self.q_norm(q_by_head)
        q = q_by_head.view(q.shape)

        k_by_head = k.view(*k.shape[:-1], k.shape[-1] // self.head_dim, self.head_dim)
        k_by_head = self.k_norm(k_by_head)
        k = k_by_head.view(k.shape)
        q, k = self.rotary_emb.forward_native(positions, q, k)
        attn_output = self.attn(q, k, v)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** Method `InternS1ProMoeAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternS1ProMoeAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternS1ProMoeDecoderLayer` (lines 370-457)
```python
class InternS1ProMoeDecoderLayer(nn.Module):
    def __init__(self, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()

        config = vllm_config.model_config.hf_text_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.hidden_size = config.hidden_size
        max_position_embeddings = getattr(config, "max_position_embeddings", 32768)
        dual_chunk_attention_config = getattr(
            config, "dual_chunk_attention_config", None
        )

        # update rope related parameters
        rope_scaling = config.rope_scaling
        fope_keys = {"fope_init_factor", "fope_sep_head", "num_inv_freq"}
        use_fope = any(rope_scaling.get(key) is not None for key in fope_keys)
        fope_init_factor = rope_scaling.get("fope_init_factor", None)
        fope_sep_head = rope_scaling.get("fope_sep_head", None)
        num_inv_freq = rope_scaling.get("num_inv_freq", None)

        config.rope_parameters["use_fope"] = use_fope
        config.rope_parameters["fope_init_factor"] = fope_init_factor
        config.rope_parameters["fope_sep_head"] = fope_sep_head
```
**EN:** Class `InternS1ProMoeDecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `InternS1ProMoeDecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `InternS1ProMoeDecoderLayer.__init__` (lines 371-435)
```python
    def __init__(self, vllm_config: VllmConfig, prefix: str = "") -> None:
        super().__init__()

        config = vllm_config.model_config.hf_text_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.hidden_size = config.hidden_size
        max_position_embeddings = getattr(config, "max_position_embeddings", 32768)
        dual_chunk_attention_config = getattr(
            config, "dual_chunk_attention_config", None
        )

        # update rope related parameters
        rope_scaling = config.rope_scaling
        fope_keys = {"fope_init_factor", "fope_sep_head", "num_inv_freq"}
        use_fope = any(rope_scaling.get(key) is not None for key in fope_keys)
        fope_init_factor = rope_scaling.get("fope_init_factor", None)
        fope_sep_head = rope_scaling.get("fope_sep_head", None)
        num_inv_freq = rope_scaling.get("num_inv_freq", None)

        config.rope_parameters["use_fope"] = use_fope
        config.rope_parameters["fope_init_factor"] = fope_init_factor
        config.rope_parameters["fope_sep_head"] = fope_sep_head
        config.rope_parameters["num_inv_freq"] = num_inv_freq

        assert use_fope, "should use FOPE for InternS1Pro model"
        self.self_attn = InternS1ProMoeAttention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            rope_parameters=config.rope_parameters,
            max_position_embeddings=max_position_embeddings,
            rms_norm_eps=config.rms_norm_eps,
            qkv_bias=getattr(config, "attention_bias", False),
            head_dim=getattr(config, "head_dim", None),
            cache_config=cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
            dual_chunk_attention_config=dual_chunk_attention_config,
        )

        # `mlp_only_layers` in the config.
        layer_idx = extract_layer_index(prefix)
        mlp_only_layers = (
            [] if not hasattr(config, "mlp_only_layers") else config.mlp_only_layers
        )
        if (layer_idx not in mlp_only_layers) and (
            config.num_experts > 0 and (layer_idx + 1) % config.decoder_sparse_step == 0
        ):
            self.mlp = InternS1ProMoeSparseMoeBlock(
                vllm_config=vllm_config, prefix=f"{prefix}.mlp"
            )
        else:
            self.mlp = InternS1ProMoeMLP(
                hidden_size=config.hidden_size,
                intermediate_size=config.intermediate_size,
                hidden_act=config.hidden_act,
                quant_config=quant_config,
                prefix=f"{prefix}.mlp",
# ... truncated for analysis ...
```
**EN:** Method `InternS1ProMoeDecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternS1ProMoeDecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternS1ProMoeDecoderLayer.forward` (lines 437-457)
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
            hidden_states = self.input_layernorm(hidden_states)
        else:
            hidden_states, residual = self.input_layernorm(hidden_states, residual)
        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
        )

        # Fully Connected
        hidden_states, residual = self.post_attention_layernorm(hidden_states, residual)
        hidden_states = self.mlp(hidden_states)
        return hidden_states, residual
```
**EN:** Method `InternS1ProMoeDecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternS1ProMoeDecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternS1ProMoeLLMModel` (lines 460-472)
```python
class InternS1ProMoeLLMModel(Qwen3MoeLLMModel):
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        decoder_layer_type: type[torch.nn.Module] = InternS1ProMoeDecoderLayer,
    ):
        super().__init__(
            vllm_config=vllm_config,
            prefix=prefix,
            decoder_layer_type=decoder_layer_type,
        )
```
**EN:** Class `InternS1ProMoeLLMModel` is a structural model block in the vLLM execution graph. It inherits from Qwen3MoeLLMModel. Key methods include __init__.
**CN:** 类 `InternS1ProMoeLLMModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 Qwen3MoeLLMModel。 关键方法包括 __init__。

### Method `InternS1ProMoeLLMModel.__init__` (lines 461-472)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        decoder_layer_type: type[torch.nn.Module] = InternS1ProMoeDecoderLayer,
    ):
        super().__init__(
            vllm_config=vllm_config,
            prefix=prefix,
            decoder_layer_type=decoder_layer_type,
        )
```
**EN:** Method `InternS1ProMoeLLMModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternS1ProMoeLLMModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `InternS1ProMoeLLMForCausalLM` (lines 475-494)
```python
class InternS1ProMoeLLMForCausalLM(Qwen3MoeForCausalLM):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super(Qwen3MoeForCausalLM, self).__init__()
        self.config = vllm_config.model_config.hf_config.text_config
        self.quant_config = vllm_config.quant_config
        self.model = InternS1ProMoeLLMModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        self.lm_head = ParallelLMHead(
            self.config.vocab_size,
            self.config.hidden_size,
            quant_config=self.quant_config,
            prefix=maybe_prefix(prefix, "lm_head"),
        )
        if self.config.tie_word_embeddings:
            self.lm_head.weight = self.model.embed_tokens.weight
        self.logits_processor = LogitsProcessor(self.config.vocab_size)
        self.make_empty_intermediate_tensors = (
            self.model.make_empty_intermediate_tensors
        )
```
**EN:** Class `InternS1ProMoeLLMForCausalLM` organizes related behavior for this model family or helper component. It inherits from Qwen3MoeForCausalLM. Key methods include __init__.
**CN:** 类 `InternS1ProMoeLLMForCausalLM` 用于组织该模型族或辅助组件的相关行为。 它继承自 Qwen3MoeForCausalLM。 关键方法包括 __init__。

### Method `InternS1ProMoeLLMForCausalLM.__init__` (lines 476-494)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super(Qwen3MoeForCausalLM, self).__init__()
        self.config = vllm_config.model_config.hf_config.text_config
        self.quant_config = vllm_config.quant_config
        self.model = InternS1ProMoeLLMModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        self.lm_head = ParallelLMHead(
            self.config.vocab_size,
            self.config.hidden_size,
            quant_config=self.quant_config,
            prefix=maybe_prefix(prefix, "lm_head"),
        )
        if self.config.tie_word_embeddings:
            self.lm_head.weight = self.model.embed_tokens.weight
        self.logits_processor = LogitsProcessor(self.config.vocab_size)
        self.make_empty_intermediate_tensors = (
            self.model.make_empty_intermediate_tensors
        )
```
**EN:** Method `InternS1ProMoeLLMForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternS1ProMoeLLMForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `InternS1ProMoeMixtureOfExperts` (lines 497-538)
```python
class InternS1ProMoeMixtureOfExperts(MixtureOfExperts):
    def update_physical_experts_metadata(
        self,
        num_physical_experts: int,
        num_local_physical_experts: int,
    ) -> None:
        assert self.num_local_physical_experts == num_local_physical_experts
        self.num_physical_experts = num_physical_experts
        self.num_local_physical_experts = num_local_physical_experts
        self.num_redundant_experts = num_physical_experts - self.num_logical_experts
        for layer in self.language_model.model.layers:
            if isinstance(layer.mlp, InternS1ProMoeSparseMoeBlock):
                moe = layer.mlp
                moe.n_local_physical_experts = num_local_physical_experts
                moe.n_physical_experts = num_physical_experts
                moe.n_redundant_experts = self.num_redundant_experts
                moe.experts.update_expert_map()

    def set_moe_parameters(self):
        self.expert_weights = []

        self.moe_layers = []
        example_moe = None
        for layer in self.language_model.model.layers:
            if hasattr(layer, "mlp") and isinstance(
```
**EN:** Class `InternS1ProMoeMixtureOfExperts` organizes related behavior for this model family or helper component. It inherits from MixtureOfExperts. Key methods include update_physical_experts_metadata, set_moe_parameters.
**CN:** 类 `InternS1ProMoeMixtureOfExperts` 用于组织该模型族或辅助组件的相关行为。 它继承自 MixtureOfExperts。 关键方法包括 update_physical_experts_metadata, set_moe_parameters。

### Method `InternS1ProMoeMixtureOfExperts.update_physical_experts_metadata` (lines 498-513)
```python
    def update_physical_experts_metadata(
        self,
        num_physical_experts: int,
        num_local_physical_experts: int,
    ) -> None:
        assert self.num_local_physical_experts == num_local_physical_experts
        self.num_physical_experts = num_physical_experts
        self.num_local_physical_experts = num_local_physical_experts
        self.num_redundant_experts = num_physical_experts - self.num_logical_experts
        for layer in self.language_model.model.layers:
            if isinstance(layer.mlp, InternS1ProMoeSparseMoeBlock):
                moe = layer.mlp
                moe.n_local_physical_experts = num_local_physical_experts
                moe.n_physical_experts = num_physical_experts
                moe.n_redundant_experts = self.num_redundant_experts
                moe.experts.update_expert_map()
```
**EN:** Method `InternS1ProMoeMixtureOfExperts.update_physical_experts_metadata` verifies runtime/model configuration and applies compatibility fixes before the rest of the stack executes.
**CN:** Method `InternS1ProMoeMixtureOfExperts.update_physical_experts_metadata` 会在其余栈执行之前校验运行时/模型配置，并应用兼容性修补。

### Method `InternS1ProMoeMixtureOfExperts.set_moe_parameters` (lines 515-538)
```python
    def set_moe_parameters(self):
        self.expert_weights = []

        self.moe_layers = []
        example_moe = None
        for layer in self.language_model.model.layers:
            if hasattr(layer, "mlp") and isinstance(
                layer.mlp, InternS1ProMoeSparseMoeBlock
            ):
                example_moe = layer.mlp
                self.moe_layers.append(layer.mlp.experts)

        if example_moe is None:
            raise RuntimeError("No InternS1ProMoe layer found in the language_model.")

        # Set MoE hyperparameters
        self.num_moe_layers = len(self.moe_layers)
        self.num_expert_groups = 1
        self.num_shared_experts = 0
        self.num_logical_experts = example_moe.n_logical_experts
        self.num_physical_experts = example_moe.n_physical_experts
        self.num_local_physical_experts = example_moe.n_local_physical_experts
        self.num_routed_experts = example_moe.n_routed_experts
        self.num_redundant_experts = example_moe.n_redundant_experts
```
**EN:** Method `InternS1ProMoeMixtureOfExperts.set_moe_parameters` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternS1ProMoeMixtureOfExperts.set_moe_parameters` 封装了该模块中的一段可复用核心逻辑。

### Class `InternS1ProForConditionalGeneration` (lines 546-643)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Qwen3VLMultiModalProcessor,
    info=InternS1ProProcessingInfo,
    dummy_inputs=Qwen3VLDummyInputsBuilder,
)
class InternS1ProForConditionalGeneration(
    Qwen3VLForConditionalGeneration, InternS1ProMoeMixtureOfExperts
):
    is_3d_moe_weight: bool = True
    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
    }

    # To ensure correct weight loading and mapping.
    hf_to_vllm_mapper = WeightsMapper(
        orig_to_new_prefix={
            "model.visual.": "visual.",
            "lm_head.": "language_model.lm_head.",
            "model.language_model.": "language_model.model.",
        },
    )
```
**EN:** Class `InternS1ProForConditionalGeneration` organizes related behavior for this model family or helper component. It inherits from Qwen3VLForConditionalGeneration, InternS1ProMoeMixtureOfExperts. Key methods include __init__, get_frope_params_map, load_weights.
**CN:** 类 `InternS1ProForConditionalGeneration` 用于组织该模型族或辅助组件的相关行为。 它继承自 Qwen3VLForConditionalGeneration、InternS1ProMoeMixtureOfExperts。 关键方法包括 __init__, get_frope_params_map, load_weights。

### Method `InternS1ProForConditionalGeneration.__init__` (lines 567-613)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super(Qwen3VLForConditionalGeneration, self).__init__()
        config: PretrainedConfig = vllm_config.model_config.hf_config
        multimodal_config = vllm_config.model_config.multimodal_config

        self.config = config
        self.multimodal_config = multimodal_config
        self.use_data_parallel = multimodal_config.mm_encoder_tp_mode == "data"
        self.video_pruning_rate = multimodal_config.video_pruning_rate
        self.is_multimodal_pruning_enabled = (
            multimodal_config.is_multimodal_pruning_enabled()
        )

        with self._mark_tower_model(vllm_config, {"image", "video"}):
            self.visual = Qwen3_VisionTransformer(
                config.vision_config,
                norm_eps=getattr(config, "rms_norm_eps", 1e-6),
                prefix=maybe_prefix(prefix, "visual"),
            )

        with self._mark_language_model(vllm_config):
            self.language_model = InternS1ProMoeLLMForCausalLM(
                vllm_config=vllm_config,
                prefix=maybe_prefix(prefix, "language_model"),
            )

        # Whether to include the gate_up_proj mapping is determined by
        # the language model.
        self.packed_modules_mapping = (
            self.packed_modules_mapping | self.language_model.packed_modules_mapping
        )

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )

        self.use_deepstack = hasattr(config.vision_config, "deepstack_visual_indexes")
        self.deepstack_num_level = (
            len(config.vision_config.deepstack_visual_indexes)
            if self.use_deepstack
            else 0
        )
        self.visual_dim = config.vision_config.out_hidden_size
        self.multiscale_dim = self.visual_dim * self.deepstack_num_level

        # Set MoE hyperparameters
        self.set_moe_parameters()
```
**EN:** Method `InternS1ProForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternS1ProForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternS1ProForConditionalGeneration.get_frope_params_map` (lines 615-626)
```python
    def get_frope_params_map(self) -> str:
        mapper = {}
        for name, params in self.language_model.model.named_parameters():
            if "rotary_emb.sin_coef" in name:
                mapper["language_model.model.rotary_emb.sin_coef"] = (
                    f"language_model.model.{name}"
                )
            if "rotary_emb.cos_coef" in name:
                mapper["language_model.model.rotary_emb.cos_coef"] = (
                    f"language_model.model.{name}"
                )
        return mapper
```
**EN:** Method `InternS1ProForConditionalGeneration.get_frope_params_map` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `InternS1ProForConditionalGeneration.get_frope_params_map` 封装了该模块中的一段可复用核心逻辑。

### Method `InternS1ProForConditionalGeneration.load_weights` (lines 628-643)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        """load weights"""
        skip_prefixes = ["model.time_series."]
        if self.visual is None:
            skip_prefixes.append("visual.")
        # FIXME(Isotr0py): See if we can avoid tighing FoPE to PP layers
        weights_mapper = WeightsMapper(
            orig_to_new_prefix={
                "model.visual.": "visual.",
                "lm_head.": "language_model.lm_head.",
                "model.language_model.": "language_model.model.",
            },
            orig_to_new_suffix=self.get_frope_params_map(),
        )
        loader = AutoWeightsLoader(self, skip_prefixes=skip_prefixes)
        return loader.load_weights(weights, mapper=weights_mapper)
```
**EN:** Method `InternS1ProForConditionalGeneration.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders. The docstring says: load weights.
**CN:** Method `InternS1ProForConditionalGeneration.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。 文档字符串表达的核心意思是：load weights。

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
- **Standard library / 标准库**: `import functools`, `from collections.abc import Iterable`, `from typing import Any`
- **Third-party / 第三方**: `import torch`, `from torch import nn`, `from transformers import AutoProcessor, PretrainedConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import (`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.activation import SiluAndMul`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.fused_moe import (`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding import get_rope`, `from vllm.model_executor.layers.vocab_parallel_embedding import (`
- **Module note / 模块说明**: **EN:** Inference-only InternS1Pro model compatible with HuggingFace weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only InternS1Pro model compatible with HuggingFace weights.。
