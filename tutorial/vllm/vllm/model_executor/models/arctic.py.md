# arctic.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/arctic.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for arctic, including architecture wrappers and weight loading logic. / 面向推理的 arctic vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-53)
```python
from collections.abc import Iterable
from itertools import islice

import torch
from torch import nn

from vllm.compilation.decorators import support_torch_compile
from vllm.config import CacheConfig, VllmConfig
from vllm.distributed import (
    get_pp_group,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_reduce,
)
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.fused_moe import (
    fused_experts,
    fused_topk,
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
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform
from vllm.sequence import IntermediateTensors
from vllm.transformers_utils.configs.arctic import ArcticConfig

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

### Class `ArcticMLP` (lines 56-100)
```python
class ArcticMLP(nn.Module):
    def __init__(
        self,
        config: ArcticConfig,
        expert_id: int = -1,
        is_residual_mlp: bool = False,
        quant_config: QuantizationConfig | None = None,
        reduce_results: bool = True,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.expert_id = expert_id

        self.ffn_dim = (
            config.intermediate_size if not is_residual_mlp else self.hidden_size
        )

        self.w13 = MergedColumnParallelLinear(
            self.hidden_size,
            [self.ffn_dim] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.w13",
        )
```
**EN:** Class `ArcticMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `ArcticMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `ArcticMLP.__init__` (lines 57-94)
```python
    def __init__(
        self,
        config: ArcticConfig,
        expert_id: int = -1,
        is_residual_mlp: bool = False,
        quant_config: QuantizationConfig | None = None,
        reduce_results: bool = True,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.expert_id = expert_id

        self.ffn_dim = (
            config.intermediate_size if not is_residual_mlp else self.hidden_size
        )

        self.w13 = MergedColumnParallelLinear(
            self.hidden_size,
            [self.ffn_dim] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.w13",
        )
        self.w2 = RowParallelLinear(
            self.ffn_dim,
            self.hidden_size,
            bias=False,
            reduce_results=reduce_results,
            quant_config=quant_config,
            prefix=f"{prefix}.w2",
        )
        if config.hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {config.hidden_act}. "
                "Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()
```
**EN:** Method `ArcticMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ArcticMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ArcticMLP.forward` (lines 96-100)
```python
    def forward(self, hidden_states):
        gate_up, _ = self.w13(hidden_states)
        hidden_states = self.act_fn(gate_up)
        hidden_states, _ = self.w2(hidden_states)
        return hidden_states
```
**EN:** Method `ArcticMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `ArcticMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `ArcticMoE` (lines 103-227)
```python
class ArcticMoE(nn.Module):
    """
    Model-parallel implementation of Arctic MoE Layer.
    """

    def __init__(
        self,
        config: ArcticConfig,
        tp_size: int | None = None,
        params_dtype: torch.dtype | None = None,
        quant_config: QuantizationConfig | None = None,
        reduce_results: bool = True,
        prefix: str = "",
    ):
        super().__init__()

        layer_id = extract_layer_index(prefix)
        self.tp_size = tp_size or get_tensor_model_parallel_world_size()
        self.hidden_size = config.hidden_size
        self.num_experts = config.num_local_experts
        self.layer_id = layer_id
        self.top_k = config.num_experts_per_tok
        self.intermediate_size = config.intermediate_size // self.tp_size

        self.is_moe_layer = (layer_id + 1) % config.moe_layer_frequency == 0
```
**EN:** Class `ArcticMoE` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, weight_loader, local_moe_fused, forward.
**CN:** 类 `ArcticMoE` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, weight_loader, local_moe_fused, forward。

### Method `ArcticMoE.__init__` (lines 108-179)
```python
    def __init__(
        self,
        config: ArcticConfig,
        tp_size: int | None = None,
        params_dtype: torch.dtype | None = None,
        quant_config: QuantizationConfig | None = None,
        reduce_results: bool = True,
        prefix: str = "",
    ):
        super().__init__()

        layer_id = extract_layer_index(prefix)
        self.tp_size = tp_size or get_tensor_model_parallel_world_size()
        self.hidden_size = config.hidden_size
        self.num_experts = config.num_local_experts
        self.layer_id = layer_id
        self.top_k = config.num_experts_per_tok
        self.intermediate_size = config.intermediate_size // self.tp_size

        self.is_moe_layer = (layer_id + 1) % config.moe_layer_frequency == 0
        self.reduce_results = reduce_results
        # Some other parameters
        if params_dtype is None:
            params_dtype = torch.get_default_dtype()
        self.params_dtype = params_dtype

        if not self.is_moe_layer:
            self.mlp = ArcticMLP(
                config,
                quant_config=quant_config,
                reduce_results=reduce_results,
                prefix=f"{prefix}.mlp",
            )
        else:
            self.gate = ReplicatedLinear(
                self.hidden_size,
                self.num_experts,
                bias=False,
                params_dtype=self.params_dtype,
                quant_config=quant_config,
                prefix=f"{prefix}.gate",
            )
            self.ws = nn.Parameter(
                torch.empty(
                    self.num_experts,
                    2 * self.intermediate_size,
                    self.hidden_size,
                    device=current_platform.device_type,
                    dtype=self.params_dtype,
                )
            )
            self.w2s = nn.Parameter(
                torch.empty(
                    self.num_experts,
                    self.hidden_size,
                    self.intermediate_size,
                    device=current_platform.device_type,
                    dtype=self.params_dtype,
                )
            )
# ... truncated for analysis ...
```
**EN:** Method `ArcticMoE.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ArcticMoE.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ArcticMoE.weight_loader` (lines 181-199)
```python
    def weight_loader(
        self,
        param: nn.Parameter,
        loaded_weight: torch.Tensor,
        weight_name: str,
        expert_id: int,
    ):
        tp_rank = get_tensor_model_parallel_rank()
        param_data = param.data
        shard_size = self.intermediate_size
        shard = slice(tp_rank * shard_size, (tp_rank + 1) * shard_size)
        if weight_name.endswith("w1.weight"):
            param_data[expert_id, 0:shard_size, :] = loaded_weight[shard, :]
        if weight_name.endswith("w3.weight"):
            param_data[expert_id, shard_size : 2 * shard_size, :] = loaded_weight[
                shard, :
            ]
        if weight_name.endswith("w2.weight"):
            param_data[expert_id, :, :] = loaded_weight[:, shard]
```
**EN:** Method `ArcticMoE.weight_loader` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `ArcticMoE.weight_loader` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Method `ArcticMoE.local_moe_fused` (lines 201-220)
```python
    def local_moe_fused(self, hidden_states: torch.Tensor) -> torch.Tensor:
        num_tokens, hidden_size = hidden_states.shape
        hidden_states = hidden_states.view(-1, self.hidden_size)
        # router_logits: (num_tokens, n_experts)
        router_logits, _ = self.gate(hidden_states)
        do_normalize = self.top_k > 1
        topk_weights, topk_ids, token_expert_indices = fused_topk(
            hidden_states, router_logits, self.top_k, renormalize=do_normalize
        )
        final_hidden_states = fused_experts(
            hidden_states,
            self.ws,
            self.w2s,
            topk_weights,
            topk_ids,
            inplace=True,
        )
        if self.reduce_results and self.tp_size > 1:
            final_hidden_states = tensor_model_parallel_all_reduce(final_hidden_states)
        return final_hidden_states.view(num_tokens, hidden_size)
```
**EN:** Method `ArcticMoE.local_moe_fused` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ArcticMoE.local_moe_fused` 封装了该模块中的一段可复用核心逻辑。

### Method `ArcticMoE.forward` (lines 222-227)
```python
    def forward(self, hidden_states: torch.Tensor):
        if self.is_moe_layer:
            final_hidden_states = self.local_moe_fused(hidden_states)
        else:
            final_hidden_states = self.mlp(hidden_states)
        return final_hidden_states
```
**EN:** Method `ArcticMoE.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `ArcticMoE.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `ArcticAttention` (lines 230-304)
```python
class ArcticAttention(nn.Module):
    def __init__(
        self,
        config: ArcticConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size

        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = self.hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
```
**EN:** Class `ArcticAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `ArcticAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `ArcticAttention.__init__` (lines 231-292)
```python
    def __init__(
        self,
        config: ArcticConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size

        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = self.hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim

        self.max_position_embeddings = config.max_position_embeddings
        self.scaling = self.head_dim**-0.5

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
            reduce_results=True,
            quant_config=quant_config,
            prefix=f"{prefix}.o_proj",
        )

        self.rotary_emb = get_rope(
            self.head_dim,
            max_position=self.max_position_embeddings,
            rope_parameters=config.rope_parameters,
            is_neox_style=True,
        )

        self.attn = Attention(
            self.num_heads,
            self.head_dim,
            self.scaling,
            num_kv_heads=self.num_kv_heads,
            cache_config=cache_config,
            quant_config=quant_config,
# ... truncated for analysis ...
```
**EN:** Method `ArcticAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ArcticAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ArcticAttention.forward` (lines 294-304)
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
        return output
```
**EN:** Method `ArcticAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `ArcticAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `ArcticDecoderLayer` (lines 307-376)
```python
class ArcticDecoderLayer(nn.Module):
    def __init__(
        self,
        config: ArcticConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        layer_idx = extract_layer_index(prefix)
        is_moe_layer = (layer_idx + 1) % config.moe_layer_frequency == 0
        self.use_residual = config.use_residual and is_moe_layer
        self.self_attn = ArcticAttention(
            config,
            cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.block_sparse_moe = ArcticMoE(
            config,
            quant_config=quant_config,
            reduce_results=(not self.use_residual),
            prefix=f"{prefix}.block_sparse_moe",
        )
```
**EN:** Class `ArcticDecoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `ArcticDecoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `ArcticDecoderLayer.__init__` (lines 308-347)
```python
    def __init__(
        self,
        config: ArcticConfig,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        layer_idx = extract_layer_index(prefix)
        is_moe_layer = (layer_idx + 1) % config.moe_layer_frequency == 0
        self.use_residual = config.use_residual and is_moe_layer
        self.self_attn = ArcticAttention(
            config,
            cache_config,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.block_sparse_moe = ArcticMoE(
            config,
            quant_config=quant_config,
            reduce_results=(not self.use_residual),
            prefix=f"{prefix}.block_sparse_moe",
        )

        self.input_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.post_attention_layernorm = RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )

        if self.use_residual:
            self.residual_layernorm = RMSNorm(
                config.hidden_size, eps=config.rms_norm_eps
            )
            self.residual_mlp = ArcticMLP(
                config,
                is_residual_mlp=True,
                reduce_results=False,
                prefix=f"{prefix}.residual_mlp",
            )
```
**EN:** Method `ArcticDecoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ArcticDecoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ArcticDecoderLayer.forward` (lines 349-376)
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        residual_input = hidden_states
        hidden_states = self.input_layernorm(hidden_states)
        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
        )
        hidden_states = residual_input + hidden_states

        residual_attn = hidden_states
        if self.use_residual:
            hidden_states = self.residual_layernorm(hidden_states)
            hidden_states = self.residual_mlp(hidden_states)
            residual_mlp = hidden_states
            hidden_states = self.post_attention_layernorm(residual_input)
            hidden_states = self.block_sparse_moe(hidden_states)
            hidden_states = residual_mlp + hidden_states
            hidden_states = tensor_model_parallel_all_reduce(hidden_states)
            hidden_states = residual_attn + hidden_states
        else:
            hidden_states = self.post_attention_layernorm(hidden_states)
            hidden_states = self.block_sparse_moe(hidden_states)
            hidden_states = residual_attn + hidden_states
        return hidden_states
```
**EN:** Method `ArcticDecoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `ArcticDecoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `ArcticModel` (lines 380-539)
```python
@support_torch_compile
class ArcticModel(nn.Module):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.vocab_size = config.vocab_size
        self.embed_tokens = VocabParallelEmbedding(
            self.vocab_size, config.hidden_size, org_num_embeddings=self.vocab_size
        )
        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers,
            lambda prefix: ArcticDecoderLayer(
                config, cache_config, quant_config, prefix=prefix
            ),
            prefix=f"{prefix}.layers",
        )
        self._attn_implementation = config._attn_implementation
        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.make_empty_intermediate_tensors = make_empty_intermediate_tensors_factory(
            ["hidden_states"], config.hidden_size
```
**EN:** Class `ArcticModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, embed_input_ids, forward, load_weights.
**CN:** 类 `ArcticModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, embed_input_ids, forward, load_weights。

### Method `ArcticModel.__init__` (lines 381-404)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()

        config = vllm_config.model_config.hf_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.config = config
        self.vocab_size = config.vocab_size
        self.embed_tokens = VocabParallelEmbedding(
            self.vocab_size, config.hidden_size, org_num_embeddings=self.vocab_size
        )
        self.start_layer, self.end_layer, self.layers = make_layers(
            config.num_hidden_layers,
            lambda prefix: ArcticDecoderLayer(
                config, cache_config, quant_config, prefix=prefix
            ),
            prefix=f"{prefix}.layers",
        )
        self._attn_implementation = config._attn_implementation
        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.make_empty_intermediate_tensors = make_empty_intermediate_tensors_factory(
            ["hidden_states"], config.hidden_size
        )
```
**EN:** Method `ArcticModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ArcticModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ArcticModel.embed_input_ids` (lines 406-407)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)
```
**EN:** Method `ArcticModel.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `ArcticModel.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `ArcticModel.forward` (lines 409-429)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        if get_pp_group().is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.embed_input_ids(input_ids)
        else:
            assert intermediate_tensors is not None
            hidden_states = intermediate_tensors["hidden_states"]
        for layer in islice(self.layers, self.start_layer, self.end_layer):
            hidden_states = layer(positions, hidden_states)
        if not get_pp_group().is_last_rank:
            return IntermediateTensors({"hidden_states": hidden_states})
        hidden_states = self.norm(hidden_states)
        return hidden_states
```
**EN:** Method `ArcticModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `ArcticModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `ArcticModel.load_weights` (lines 431-539)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]

        mlp_params_mapping: list[tuple[str, str, int]] = []
        expert_params_mapping: list[tuple[str, str, int]] = []

        for layer in range(self.config.num_hidden_layers):
            is_moe_layer = (layer + 1) % self.config.moe_layer_frequency == 0
            if is_moe_layer and self.config.use_residual:
                mlp_params_mapping.append(
                    (
                        f"layers.{layer}.residual_mlp.w13.weight",
                        f"layers.{layer}.residual_mlp.w1.weight",
                        0,
                    )
                )
                mlp_params_mapping.append(
                    (
                        f"layers.{layer}.residual_mlp.w13.weight",
                        f"layers.{layer}.residual_mlp.w3.weight",
                        1,
                    )
                )

            if is_moe_layer:
                for expert_id in range(self.config.num_local_experts):
                    expert_params_mapping.append(
                        ("ws", f"experts.{expert_id}.w1.weight", expert_id)
                    )
                    expert_params_mapping.append(
                        ("w2s", f"experts.{expert_id}.w2.weight", expert_id)
                    )
                    expert_params_mapping.append(
                        ("ws", f"experts.{expert_id}.w3.weight", expert_id)
                    )
            else:
                mlp_params_mapping.append(
                    (
                        f"layers.{layer}.block_sparse_moe.mlp.w13.weight",
                        f"layers.{layer}.block_sparse_moe.mlp.w1.weight",
                        0,
                    )
                )
                mlp_params_mapping.append(
                    (
                        f"layers.{layer}.block_sparse_moe.mlp.w13.weight",
                        f"layers.{layer}.block_sparse_moe.mlp.w3.weight",
                        1,
                    )
                )

        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()

        for name, loaded_weight in weights:
# ... truncated for analysis ...
```
**EN:** Method `ArcticModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `ArcticModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `ArcticForCausalLM` (lines 542-597)
```python
class ArcticForCausalLM(nn.Module, SupportsPP, SupportsQuant):
    packed_modules_mapping = {"qkv_proj": ["q_proj", "k_proj", "v_proj"]}

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.model = ArcticModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        self.vocab_size = config.vocab_size
        self.lm_head = ParallelLMHead(
            self.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "lm_head"),
        )
        if self.config.tie_word_embeddings:
            self.lm_head.weight = self.model.embed_tokens.weight
        self.num_experts = config.num_local_experts
        self.num_experts_per_tok = config.num_experts_per_tok

        self.logits_processor = LogitsProcessor(config.vocab_size)
        self.make_empty_intermediate_tensors = (
```
**EN:** Class `ArcticForCausalLM` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsPP, SupportsQuant. Key methods include __init__, embed_input_ids, forward, compute_logits, load_weights.
**CN:** 类 `ArcticForCausalLM` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsPP、SupportsQuant。 关键方法包括 __init__, embed_input_ids, forward, compute_logits, load_weights。

### Method `ArcticForCausalLM.__init__` (lines 545-568)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        self.config = config
        self.model = ArcticModel(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
        )
        self.vocab_size = config.vocab_size
        self.lm_head = ParallelLMHead(
            self.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "lm_head"),
        )
        if self.config.tie_word_embeddings:
            self.lm_head.weight = self.model.embed_tokens.weight
        self.num_experts = config.num_local_experts
        self.num_experts_per_tok = config.num_experts_per_tok

        self.logits_processor = LogitsProcessor(config.vocab_size)
        self.make_empty_intermediate_tensors = (
            self.model.make_empty_intermediate_tensors
        )
```
**EN:** Method `ArcticForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ArcticForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ArcticForCausalLM.embed_input_ids` (lines 570-571)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)
```
**EN:** Method `ArcticForCausalLM.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `ArcticForCausalLM.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `ArcticForCausalLM.forward` (lines 573-583)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor | IntermediateTensors:
        hidden_states = self.model(
            input_ids, positions, intermediate_tensors, inputs_embeds
        )
        return hidden_states
```
**EN:** Method `ArcticForCausalLM.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `ArcticForCausalLM.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `ArcticForCausalLM.compute_logits` (lines 585-590)
```python
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        logits = self.logits_processor(self.lm_head, hidden_states)
        return logits
```
**EN:** Method `ArcticForCausalLM.compute_logits` converts hidden states into prediction-ready outputs, often by applying a head, post-processing, or sampling logic.
**CN:** Method `ArcticForCausalLM.compute_logits` 将隐藏状态转成可预测输出，通常会经过输出头、后处理或采样逻辑。

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
- **Third-party / 第三方**: `import torch`, `from torch import nn`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import support_torch_compile`, `from vllm.config import CacheConfig, VllmConfig`, `from vllm.distributed import (`, `from vllm.model_executor.layers.activation import SiluAndMul`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.fused_moe import (`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.logits_processor import LogitsProcessor`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding import get_rope`, `from vllm.model_executor.layers.vocab_parallel_embedding import (`
- **Module note / 模块说明**: **EN:** Inference-only Snowflake Arctic model. **CN:** 模块文档字符串给出的原始说明是：Inference-only Snowflake Arctic model.。
