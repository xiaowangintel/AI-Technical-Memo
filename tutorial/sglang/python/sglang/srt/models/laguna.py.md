# laguna.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/laguna.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Laguna architecture into SGLang's serving runtime. Inference-only Laguna (poolside/Laguna-XS.2) model. / 该模块将 Laguna 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only Laguna (poolside/Laguna-XS.2) model。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6: Module header and imports / 模块头与导入
```python
# Copyright 2023-2026 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 61-61: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 64-111: Class: LagunaMLP / 类：LagunaMLP
```python
class LagunaMLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: Optional[QuantizationConfig] = None,
        reduce_results: bool = True,
        prefix: str = "",
    ) -> None:
        super().__init__()
        if hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. Only silu is supported."
            )
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            reduce_results=reduce_results,
            prefix=add_prefix("down_proj", prefix),
        )
        self.act_fn = SiluAndMul()

    def forward(
        self,
        x: torch.Tensor,
        forward_batch: Optional[ForwardBatch] = None,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
    ) -> torch.Tensor:
        gate_up, _ = self.gate_up_proj(x)
# ... truncated for brevity ...
```
**EN:** This class defines Laguna M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Laguna M L P，用于封装该模型组件的状态与方法。

### Lines 114-133: Class: LagunaMoEGate / 类：LagunaMoEGate
```python
class LagunaMoEGate(nn.Module):
    def __init__(
        self,
        config: LagunaConfig,
        prefix: str = "",
    ):
        super().__init__()
        self.weight = nn.Parameter(
            torch.empty(config.num_experts, config.hidden_size, dtype=torch.float32)
        )
        # Released checkpoint stores this under `mlp.experts.e_score_correction_bias`
        # (load_weights remaps it) but every value is 0.0; zero-init keeps us
        # correct if a future checkpoint omits the tensor entirely.
        self.e_score_correction_bias = nn.Parameter(
            torch.zeros(config.num_experts, dtype=torch.float32),
            requires_grad=False,
        )

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        return F.linear(hidden_states.to(torch.float32), self.weight, None)
```
**EN:** This class defines Laguna Mo E Gate inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Laguna Mo E Gate，用于封装该模型组件的状态与方法。

### Lines 136-225: Class: LagunaMoE / 类：LagunaMoE
```python
class LagunaMoE(nn.Module):
    def __init__(
        self,
        config: LagunaConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.routed_scaling_factor = config.moe_routed_scaling_factor
        self.router_logit_softcapping = getattr(
            config, "moe_router_logit_softcapping", 0.0
        )

        if self.tp_size > config.num_experts:
            raise ValueError(
                f"TP size {self.tp_size} > num_experts {config.num_experts}."
            )

        self.gate = LagunaMoEGate(config, prefix=add_prefix("gate", prefix))

        self.experts = get_moe_impl_class(quant_config)(
            num_experts=config.num_experts
            + get_global_server_args().ep_num_redundant_experts,
            top_k=config.num_experts_per_tok,
            layer_id=layer_id,
            hidden_size=config.hidden_size,
            intermediate_size=config.moe_intermediate_size,
            quant_config=quant_config,
            reduce_results=False,
            apply_router_weight_on_input=bool(config.moe_apply_router_weight_on_input),
            prefix=add_prefix("experts", prefix),
        )

        self.topk = TopK(
            top_k=config.num_experts_per_tok,
            layer_id=layer_id,
            renormalize=True,
            use_grouped_topk=False,
# ... truncated for brevity ...
```
**EN:** This class defines Laguna Mo E inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Laguna Mo E，用于封装该模型组件的状态与方法。

### Lines 228-358: Class: LagunaAttention / 类：LagunaAttention
```python
class LagunaAttention(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        head_dim: int,
        layer_id: int,
        rms_norm_eps: float,
        rope_theta: float,
        rope_scaling: Optional[Dict[str, Any]],
        partial_rotary_factor: float,
        max_position_embeddings: int,
        attention_bias: bool,
        sliding_window_size: int,
        layer_type: str,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.head_dim = head_dim
        self.layer_id = layer_id

        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()

        self.total_num_heads = num_heads
        assert self.total_num_heads % attn_tp_size == 0
        self.num_heads = self.total_num_heads // attn_tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= attn_tp_size:
            assert self.total_num_kv_heads % attn_tp_size == 0
        else:
            assert attn_tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // attn_tp_size)
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5

# ... truncated for brevity ...
```
**EN:** This class defines Laguna Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Laguna Attention，用于封装该模型组件的状态与方法。

### Lines 361-495: Class: LagunaDecoderLayer / 类：LagunaDecoderLayer
```python
class LagunaDecoderLayer(nn.Module):
    def __init__(
        self,
        config: LagunaConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.layer_id = layer_id
        self.hidden_size = config.hidden_size

        layer_types = config.layer_types
        layer_type = layer_types[layer_id]
        is_swa = layer_type == "sliding_attention"

        layer_num_heads = config.num_attention_heads_per_layer[layer_id]

        if is_swa:
            rope_theta = config.swa_rope_theta
            rope_scaling = config.swa_rope_scaling
            partial_rotary_factor = config.swa_partial_rotary_factor
        else:
            rope_theta = config.rope_theta
            rope_scaling = config.full_rope_scaling
            partial_rotary_factor = config.partial_rotary_factor

        self.self_attn = LagunaAttention(
            hidden_size=self.hidden_size,
            num_heads=layer_num_heads,
            num_kv_heads=config.num_key_value_heads,
            head_dim=config.head_dim,
            layer_id=layer_id,
            rms_norm_eps=config.rms_norm_eps,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            partial_rotary_factor=partial_rotary_factor,
            max_position_embeddings=config.max_position_embeddings,
            attention_bias=config.attention_bias,
# ... truncated for brevity ...
```
**EN:** This class defines Laguna Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Laguna Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 498-578: Class: LagunaModel / 类：LagunaModel
```python
class LagunaModel(nn.Module):
    def __init__(
        self,
        config: LagunaConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        decoder_layer_type: type = LagunaDecoderLayer,
    ) -> None:
        super().__init__()
        self.config = config
        self.padding_idx = getattr(config, "pad_token_id", None)
        self.vocab_size = config.vocab_size
        self.pp_group = get_pp_group()

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                use_attn_tp_group=is_dp_attention_enabled(),
                prefix=add_prefix("embed_tokens", prefix),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        decoder_layer_type = decoder_layer_type or LagunaDecoderLayer
        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: decoder_layer_type(
                layer_id=idx,
                config=config,
                quant_config=quant_config,
                prefix=prefix,
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix=add_prefix("layers", prefix),
        )
        if self.pp_group.is_last_rank:
            self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        else:
# ... truncated for brevity ...
```
**EN:** This class defines Laguna Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Laguna Model，用于封装该模型组件的状态与方法。

### Lines 581-784: Class: LagunaForCausalLM / 类：LagunaForCausalLM
```python
class LagunaForCausalLM(nn.Module):
    fall_back_to_pt_during_load = False
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }

    def __init__(
        self,
        config: LagunaConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.model = LagunaModel(
            config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )
        if self.pp_group.is_last_rank:
            self.lm_head = ParallelLMHead(
                config.vocab_size,
                config.hidden_size,
                quant_config=quant_config,
                prefix=add_prefix("lm_head", prefix),
                use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
            )
        else:
            self.lm_head = PPMissingLayer()
        self.logits_processor = LogitsProcessor(config)

        # Only walk this rank's local layers — out-of-range entries can be PPMissingLayer.
        self._routed_experts_weights_of_layer = LazyValue(
            lambda: {
                layer_id: self.model.layers[layer_id].mlp.get_moe_weights()
                for layer_id in range(self.start_layer, self.end_layer)
                if isinstance(self.model.layers[layer_id].mlp, LagunaMoE)
            }
        )

# ... truncated for brevity ...
```
**EN:** This class defines Laguna For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Laguna For Causal L M，用于封装该模型组件的状态与方法。

### Lines 787-787: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = LagunaForCausalLM
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `__future__: annotations`
- `logging`
- `collections.abc: Iterable`
- `typing: Any, Dict, Optional, Tuple, Union`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `sglang.srt.configs.laguna: LagunaConfig`
- `sglang.srt.distributed: get_pp_group, get_tensor_model_parallel_world_size, tensor_model_parallel_all_reduce`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.communicator: LayerCommunicator, LayerScatterModes`
- `sglang.srt.layers.dp_attention: get_attention_tp_rank, get_attention_tp_size, is_dp_attention_enabled`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: ColumnParallelLinear, MergedColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.moe: should_skip_post_experts_all_reduce`
- `sglang.srt.layers.moe.ep_moe.layer: get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton.layer: FusedMoE`
- `sglang.srt.layers.moe.topk: TopK`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
