# phimoe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/phimoe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the phimoe model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 phimoe 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31: Module imports
```python
from typing import Iterable, Optional, Tuple, Union

import torch
from torch import nn
from transformers.configuration_utils import PretrainedConfig

from sglang.srt.distributed import get_tensor_model_parallel_world_size
from sglang.srt.layers.dp_attention import get_attention_tp_rank, get_attention_tp_size
from sglang.srt.layers.linear import (
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor, LogitsProcessorOutput
from sglang.srt.layers.moe.fused_moe_triton import FusedMoE
from sglang.srt.layers.moe.topk import TopK
from sglang.srt.layers.pooler import Pooler, PoolingType
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.layers.rotary_embedding import get_rope
from sglang.srt.layers.vocab_parallel_embedding import (
    DEFAULT_VOCAB_PADDING_SIZE,
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from sglang.srt.utils import add_prefix, make_layers
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 34-35: Class `PhiMoEConfig` overview
```python
class PhiMoEConfig(PretrainedConfig):
```
**EN:** Defines `PhiMoEConfig` as a reusable runtime type derived from PretrainedConfig. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PhiMoEConfig`，其继承关系为 PretrainedConfig。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 36-36: Class `PhiMoEConfig` attributes
```python
    model_type = "phimoe"
```
**EN:** Defines class-level attributes and metadata that shape how `PhiMoEConfig` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `PhiMoEConfig` 在运行时的行为。

### Lines 38-103: Method `PhiMoEConfig.__init__`
```python
    def __init__(
        self,
        vocab_size=32000,
        hidden_size=4096,
        intermediate_size=14336,
        num_hidden_layers=32,
        num_attention_heads=32,
        num_key_value_heads=8,
        head_dim=None,
        hidden_act="silu",
        max_position_embeddings=4096 * 32,
        initializer_range=0.02,
        rms_norm_eps=1e-5,
        use_cache=True,
        pad_token_id=None,
        bos_token_id=1,
        eos_token_id=2,
        tie_word_embeddings=False,
        rope_theta=1e6,
        sliding_window=None,
        attention_dropout=0.0,
        num_experts_per_tok=2,
        num_local_experts=16,
        output_router_logits=False,
        router_aux_loss_coef=0.001,
        router_jitter_noise=0.0,
        attention_bias=False,
        lm_head_bias=False,
        **kwargs,
    ):
        self.vocab_size = vocab_size
        self.max_position_embeddings = max_position_embeddings
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads
        self.sliding_window = sliding_window
        self.attention_bias = attention_bias
        self.lm_head_bias = lm_head_bias
        # for backward compatibility
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(vocab_size=..., hidden_size=..., intermediate_size=..., num_hidden_layers=..., num_attention_heads=..., num_key_value_heads=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(vocab_size=..., hidden_size=..., intermediate_size=..., num_hidden_layers=..., num_attention_heads=..., num_key_value_heads=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 106-155: Function `sparsemixer`
```python
def sparsemixer(scores, jitter_eps=0.01):
    ################ Select first expert (topk=2) ################

    # compute mask for sparsity
    mask_logits_threshold, max_ind = scores.max(dim=-1, keepdim=True)
    factor = scores.abs().clamp(min=mask_logits_threshold)
    mask_logits_threshold = ((mask_logits_threshold - scores) / factor) > (
        2 * jitter_eps
    )

    # apply mask
    masked_gates = scores.masked_fill(mask_logits_threshold, float("-inf"))
    selected_experts = max_ind

    # compute scores for gradients
    masked_gates = torch.softmax(masked_gates, dim=-1)
    multiplier_o = masked_gates.gather(dim=-1, index=selected_experts)

    multiplier = multiplier_o

    # masked out first expert
    masked_scores = torch.scatter(
        scores,
        -1,
        selected_experts,
        float("-inf"),
    )

    ################ Select second expert (topk=2) ################
    # compute mask for sparsity
    mask_logits_threshold, max_ind = masked_scores.max(dim=-1, keepdim=True)
    factor = scores.abs().clamp(min=mask_logits_threshold)
    mask_logits_threshold = ((mask_logits_threshold - scores) / factor) > (
        2 * jitter_eps
    )

    # apply mask
    masked_gates_top2 = masked_scores.masked_fill(mask_logits_threshold, float("-inf"))
    selected_experts_top2 = max_ind
    # compute scores for gradients
# ... truncated for brevity ...
```
**EN:** This function implements `sparsemixer(scores, jitter_eps=...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `sparsemixer(scores, jitter_eps=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 158-169: Function `phimoe_routing_function`
```python
def phimoe_routing_function(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
):
    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"
    assert topk == 2, "Only top-2 routing is supported"
    assert renormalize is False, "Renormalization is not supported"

    topk_weights, topk_ids = sparsemixer(gating_output)
    return topk_weights, topk_ids
```
**EN:** This function implements `phimoe_routing_function(hidden_states: ..., gating_output: ..., topk: ..., renormalize: ...)` and coordinates expert routing or mixture-of-experts computation.
**CN:** 这个函数实现了 `phimoe_routing_function(hidden_states: ..., gating_output: ..., topk: ..., renormalize: ...)`，其作用是协调专家路由或混合专家计算。

### Lines 172-180: Class `PhiMoE` overview
```python
class PhiMoE(nn.Module):
    """A tensor-parallel MoE implementation for PhiMoE that shards each expert
    across all ranks.

    Each expert's weights are sharded across all ranks and a fused MoE
    kernel is used for the forward pass, and finally we reduce the outputs
    across ranks.
    """
```
**EN:** Defines `PhiMoE` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PhiMoE`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 181-218: Method `PhiMoE.__init__`
```python
    def __init__(
        self,
        num_experts: int,
        top_k: int,
        hidden_size: int,
        intermediate_size: int,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = hidden_size
        self.tp_size = get_tensor_model_parallel_world_size()

        # Gate always runs at half / full precision for now.
        self.gate = ReplicatedLinear(
            hidden_size,
            num_experts,
            bias=False,
            quant_config=None,
        )

        self.topk = TopK(
            top_k=top_k,
            renormalize=False,
            custom_routing_function=phimoe_routing_function,
        )

        self.experts = FusedMoE(
            num_experts=num_experts,
            top_k=top_k,
            layer_id=layer_id,
            hidden_size=hidden_size,
            intermediate_size=intermediate_size,
            reduce_results=True,
            quant_config=quant_config,
            prefix=add_prefix("experts", prefix),
        )
```
**EN:** This method implements `__init__(num_experts: ..., top_k: ..., hidden_size: ..., intermediate_size: ..., layer_id: ..., quant_config: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(num_experts: ..., top_k: ..., hidden_size: ..., intermediate_size: ..., layer_id: ..., quant_config: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 220-229: Method `PhiMoE.forward`
```python
    def forward(
        self, hidden_states: torch.Tensor, forward_batch: Optional[ForwardBatch] = None
    ) -> torch.Tensor:
        # NOTE: hidden_states can have either 1D or 2D shape.
        orig_shape = hidden_states.shape
        hidden_states = hidden_states.view(-1, self.hidden_size)
        router_logits, _ = self.gate(hidden_states)
        topk_output = self.topk(hidden_states, router_logits)
        final_hidden_states = self.experts(hidden_states, topk_output)
        return final_hidden_states.view(orig_shape)
```
**EN:** This method implements `forward(hidden_states: ..., forward_batch: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., forward_batch: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 232-233: Class `PhiMoEAttention` overview
```python
class PhiMoEAttention(nn.Module):
```
**EN:** Defines `PhiMoEAttention` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PhiMoEAttention`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 234-312: Method `PhiMoEAttention.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        head_dim: Optional[int] = None,
        max_position: int = 4096 * 32,
        rope_theta: float = 10000,
        layer_id: int = 0,
        attention_bias: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        rope_scaling: Optional[dict] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size

        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()

        self.total_num_heads = num_heads
        assert self.total_num_heads % attn_tp_size == 0
        self.num_heads = self.total_num_heads // attn_tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= attn_tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % attn_tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert attn_tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // attn_tp_size)
        if head_dim is None:
            head_dim = hidden_size // num_heads
        self.head_dim = head_dim

        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., head_dim: ...=..., max_position: ...=..., rope_theta: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., head_dim: ...=..., max_position: ...=..., rope_theta: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 314-325: Method `PhiMoEAttention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v, forward_batch)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 328-329: Class `PhiMoEDecoderLayer` overview
```python
class PhiMoEDecoderLayer(nn.Module):
```
**EN:** Defines `PhiMoEDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PhiMoEDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 330-369: Method `PhiMoEDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: PhiMoEConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        rope_theta = config.rope_parameters["rope_theta"]
        self.self_attn = PhiMoEAttention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            max_position=config.max_position_embeddings,
            num_kv_heads=config.num_key_value_heads,
            head_dim=getattr(
                config, "head_dim", self.hidden_size // config.num_attention_heads
            ),
            rope_theta=rope_theta,
            layer_id=layer_id,
            attention_bias=config.attention_bias,
            quant_config=quant_config,
            rope_scaling=config.rope_parameters,
            prefix=add_prefix("self_attn", prefix),
        )
        self.block_sparse_moe = PhiMoE(
            num_experts=config.num_local_experts,
            top_k=config.num_experts_per_tok,
            hidden_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            layer_id=layer_id,
            quant_config=quant_config,
            prefix=add_prefix("block_sparse_moe", prefix),
        )
        self.input_layernorm = nn.LayerNorm(
            config.hidden_size, eps=config.rms_norm_eps, elementwise_affine=True
        )
        self.post_attention_layernorm = nn.LayerNorm(
            config.hidden_size, eps=config.rms_norm_eps, elementwise_affine=True
        )
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 371-396: Method `PhiMoEDecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: Optional[torch.Tensor],
        forward_batch: ForwardBatch,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        residual = hidden_states

        hidden_states = self.input_layernorm(hidden_states)

        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
            forward_batch=forward_batch,
        )
        hidden_states = hidden_states + residual

        residual = hidden_states
        hidden_states = self.post_attention_layernorm(hidden_states)
        hidden_states = self.block_sparse_moe(
            hidden_states, forward_batch=forward_batch
        )

        hidden_states = hidden_states + residual
        return hidden_states, residual
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., residual: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., residual: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 399-400: Class `PhiMoEModel` overview
```python
class PhiMoEModel(nn.Module):
```
**EN:** Defines `PhiMoEModel` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PhiMoEModel`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 401-428: Method `PhiMoEModel.__init__`
```python
    def __init__(
        self,
        config: PhiMoEConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.config = config
        self.quant_config = quant_config
        self.vocab_size = config.vocab_size
        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("embed_tokens", prefix),
        )

        self.layers = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: PhiMoEDecoderLayer(
                config, int(prefix.split(".")[-1]), quant_config, prefix=prefix
            ),
            prefix=add_prefix("layers", prefix),
        )
        self.norm = nn.LayerNorm(
            config.hidden_size, eps=config.rms_norm_eps, elementwise_affine=True
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 430-449: Method `PhiMoEModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor]:
        if input_embeds is None:
            hidden_states = self.embed_tokens(input_ids)
        else:
            hidden_states = input_embeds
        residual = None

        for layer in self.layers:
            hidden_states, residual = layer(
                positions, hidden_states, residual, forward_batch=forward_batch
            )

        hidden_states = self.norm(hidden_states)
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 452-453: Class `PhiMoEForCausalLM` overview
```python
class PhiMoEForCausalLM(nn.Module):
```
**EN:** Defines `PhiMoEForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `PhiMoEForCausalLM`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 454-480: Method `PhiMoEForCausalLM.__init__`
```python
    def __init__(
        self,
        config: PhiMoEConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):

        super().__init__()
        self.config = config
        self.quant_config = quant_config

        self.model = PhiMoEModel(
            config=config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            org_num_embeddings=config.vocab_size,
            padding_size=DEFAULT_VOCAB_PADDING_SIZE,
            quant_config=quant_config,
            bias=True,
            prefix=add_prefix("lm_head", prefix),
        )
        if self.config.tie_word_embeddings:
            self.lm_head.weight = self.model.embed_tokens.weight
        self.logits_processor = LogitsProcessor(config)
        self.pooler = Pooler(pooling_type=PoolingType.LAST, normalize=True)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 482-499: Method `PhiMoEForCausalLM.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        inputs_embeds: Optional[torch.Tensor] = None,
        get_embedding: bool = False,
    ) -> LogitsProcessorOutput:
        hidden_states = self.model(input_ids, positions, forward_batch, inputs_embeds)

        if not get_embedding:
            return self.logits_processor(
                input_ids, hidden_states, self.lm_head, forward_batch
            )

        else:
            return self.pooler(hidden_states, forward_batch)
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=..., get_embedding: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=..., get_embedding: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 501-556: Method `PhiMoEForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]

        expert_params_mapping = FusedMoE.make_expert_params_mapping(
            ckpt_gate_proj_name="w1",
            ckpt_down_proj_name="w2",
            ckpt_up_proj_name="w3",
            num_experts=self.config.num_local_experts,
        )

        params_dict = dict(self.named_parameters())
        for name, loaded_weight in weights:
            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
                name = name.replace(weight_name, param_name)
                if name.endswith(".bias") and name not in params_dict:
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
                    param = params_dict[name]
                    weight_loader = param.weight_loader
                    weight_loader(
                        param,
                        loaded_weight,
                        name,
                        shard_id=shard_id,
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 559-559: Top-level assign
```python
EntryClass = PhiMoEForCausalLM
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `typing.Iterable`
- `typing.Optional`
- `typing.Tuple`
- `typing.Union`
- `torch`
- `torch.nn`
- `transformers.configuration_utils.PretrainedConfig`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.logits_processor.LogitsProcessorOutput`
- `sglang.srt.layers.moe.fused_moe_triton.FusedMoE`
- `sglang.srt.layers.moe.topk.TopK`
- `sglang.srt.layers.pooler.Pooler`
- `sglang.srt.layers.pooler.PoolingType`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.vocab_parallel_embedding.DEFAULT_VOCAB_PADDING_SIZE`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.model_loader.weight_utils.maybe_remap_kv_scale_name`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.make_layers`
