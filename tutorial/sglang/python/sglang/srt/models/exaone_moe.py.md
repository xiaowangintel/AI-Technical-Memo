# exaone_moe.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/exaone_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Exaone Moe architecture into SGLang's serving runtime. Inference-only ExaoneMoE model compatible with HuggingFace weights. / 该模块将 Exaone Moe 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only ExaoneMoE model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16: Module header and imports / 模块头与导入
```python
# Copyright 2025 The LG AI Research Team
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

# Adapted from the vLLM version of EXAONE-MoE model
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 72-72: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 74-74: Assignment: _is_cuda / 赋值：_is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 77-137: Class: ExaoneMoEMLP / 类：ExaoneMoEMLP
```python
class ExaoneMoEMLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: Optional[QuantizationConfig] = None,
        reduce_results: bool = True,
        prefix: str = "",
        tp_rank: Optional[int] = None,
        tp_size: Optional[int] = None,
    ) -> None:
        super().__init__()
        gateup_quant_config = quant_config
        down_quant_config = quant_config
        if quant_config and hasattr(quant_config, "ignore") and quant_config.ignore:
            if add_prefix("gate_proj", prefix) in quant_config.ignore:
                gateup_quant_config = None
            if add_prefix("down_proj", prefix) in quant_config.ignore:
                down_quant_config = None

        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=gateup_quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
            tp_rank=tp_rank,
            tp_size=tp_size,
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=down_quant_config,
            reduce_results=reduce_results,
            prefix=add_prefix("down_proj", prefix),
            tp_rank=tp_rank,
            tp_size=tp_size,
        )
# ... truncated for brevity ...
```
**EN:** This class defines Exaone Mo E M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Exaone Mo E M L P，用于封装该模型组件的状态与方法。

### Lines 140-312: Class: ExaoneMoESparseMoEBlock / 类：ExaoneMoESparseMoEBlock
```python
class ExaoneMoESparseMoEBlock(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        alt_stream: Optional[torch.cuda.Stream] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.moe_ep_size = get_moe_expert_parallel_world_size()
        self.layer_id = layer_id
        self.routed_scaling_factor = config.routed_scaling_factor
        self.alt_stream = alt_stream

        self.n_routed_experts = config.num_experts

        if self.tp_size > config.num_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {config.num_experts}."
            )

        self.gate = ReplicatedLinear(
            config.hidden_size,
            config.num_experts,
            bias=False,
            quant_config=None,
            prefix=add_prefix("gate", prefix),
        )

        self.e_score_correction_bias = nn.Parameter(
            torch.empty(config.num_experts, dtype=torch.float32)
        )

        self.experts = get_moe_impl_class(quant_config)(
            num_experts=config.num_experts
            + get_global_server_args().ep_num_redundant_experts,
            top_k=config.num_experts_per_tok,
# ... truncated for brevity ...
```
**EN:** This class defines Exaone Mo E Sparse Mo E Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Exaone Mo E Sparse Mo E Block，用于封装该模型组件的状态与方法。

### Lines 315-443: Class: ExaoneMoEAttention / 类：ExaoneMoEAttention
```python
class ExaoneMoEAttention(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        rope_theta: float = 1000000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        rope_is_neox_style: bool = True,
        max_position_embeddings: int = 8192,
        quant_config: Optional[QuantizationConfig] = None,
        bias: bool = False,
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
        # MistralConfig has an optional head_dim introduced by Mistral-Nemo
        self.head_dim = getattr(
            config, "head_dim", self.hidden_size // self.total_num_heads
        )
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
# ... truncated for brevity ...
```
**EN:** This class defines Exaone Mo E Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Exaone Mo E Attention，用于封装该模型组件的状态与方法。

### Lines 446-535: Class: ExaoneMoEDecoderLayer / 类：ExaoneMoEDecoderLayer
```python
class ExaoneMoEDecoderLayer(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.config = config
        rope_theta = getattr(config, "rope_theta", 1000000)
        rope_scaling = getattr(config, "rope_scaling", None)
        if rope_scaling is not None and getattr(
            config, "original_max_position_embeddings", None
        ):
            rope_scaling["original_max_position_embeddings"] = (
                config.original_max_position_embeddings
            )
        rope_is_neox_style = getattr(config, "rope_is_neox_style", True)
        max_position_embeddings = getattr(config, "max_position_embeddings", 131072)

        attention_bias = getattr(config, "attention_bias", False) or getattr(
            config, "bias", False
        )
        self.attn_tp_size = get_attention_tp_size()
        self.attn_tp_rank = get_attention_tp_rank()

        self.self_attn = ExaoneMoEAttention(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            rope_is_neox_style=rope_is_neox_style,
            max_position_embeddings=max_position_embeddings,
            quant_config=quant_config,
# ... truncated for brevity ...
```
**EN:** This class defines Exaone Mo E Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Exaone Mo E Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 538-629: Class: ExaoneMoEModel / 类：ExaoneMoEModel
```python
class ExaoneMoEModel(nn.Module):
    fall_back_to_pt_during_load = False

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.config = config
        self.padding_idx = config.pad_token_id
        self.vocab_size = config.vocab_size
        self.pp_group = get_pp_group()

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                enable_tp=not is_dp_attention_enabled(),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: ExaoneMoEDecoderLayer(
                layer_id=idx,
                config=config,
                quant_config=quant_config,
                prefix=prefix,
                alt_stream=alt_stream,
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix=add_prefix("layers", prefix),
        )

        if self.pp_group.is_last_rank:
# ... truncated for brevity ...
```
**EN:** This class defines Exaone Mo E Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Exaone Mo E Model，用于封装该模型组件的状态与方法。

### Lines 632-884: Class: ExaoneMoEForCausalLM / 类：ExaoneMoEForCausalLM
```python
class ExaoneMoEForCausalLM(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        alt_stream = torch.cuda.Stream() if _is_cuda else None
        self.model = ExaoneMoEModel(
            config,
            quant_config=quant_config,
            prefix=add_prefix("model", prefix),
            alt_stream=alt_stream,
        )
        if self.config.tie_word_embeddings:
            self.lm_head = self.model.embed_tokens
        else:
            self.lm_head = ParallelLMHead(
                config.vocab_size,
                config.hidden_size,
                quant_config=quant_config,
                prefix=add_prefix("lm_head", prefix),
                use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
            )
        self.logits_processor = LogitsProcessor(config)
        # For EAGLE3 support
        self.capture_aux_hidden_states = False

        self._routed_experts_weights_of_layer = LazyValue(
            lambda: {
                layer_id: self.model.layers[layer_id].mlp.get_moe_weights()
                for layer_id in range(self.start_layer, self.end_layer)
                if isinstance(self.model.layers[layer_id].mlp, ExaoneMoESparseMoEBlock)
            }
        )

# ... truncated for brevity ...
```
**EN:** This class defines Exaone Mo E For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Exaone Mo E For Causal L M，用于封装该模型组件的状态与方法。

### Lines 887-887: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = ExaoneMoEForCausalLM
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `collections.abc: Iterable`
- `typing: Any, Dict, Optional, Tuple, Union`
- `torch`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.distributed: get_moe_expert_parallel_world_size, get_pp_group, get_tensor_model_parallel_world_size, tensor_model_parallel_all_reduce`
- `sglang.srt.eplb.expert_distribution: get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location: ModelConfigForExpertLocation`
- `sglang.srt.eplb.expert_location_dispatch: ExpertLocationDispatchInfo`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.dp_attention: get_attention_tp_rank, get_attention_tp_size, is_dp_attention_enabled`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, QKVParallelLinear, ReplicatedLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor, LogitsProcessorOutput`
- `sglang.srt.layers.moe: get_moe_a2a_backend, should_skip_post_experts_all_reduce`
- `sglang.srt.layers.moe.ep_moe.layer: get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton: FusedMoE`
- `sglang.srt.layers.moe.topk: TopK`
- `sglang.srt.layers.moe.utils: RoutingMethodType`
