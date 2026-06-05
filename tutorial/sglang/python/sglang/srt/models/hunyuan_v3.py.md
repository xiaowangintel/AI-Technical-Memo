# hunyuan_v3.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/hunyuan_v3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Hunyuan v3 architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 Hunyuan v3 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Module header and imports / 模块头与导入
```python
# coding=utf-8
# Copyright 2026 The HunYuan team.
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
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 54-90: Class: HYV3FeedForward / 类：HYV3FeedForward
```python
class HYV3FeedForward(nn.Module):
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

    def forward(self, x):
        gate_up, _ = self.gate_up_proj(x)
        out = self.act_fn(gate_up)
        out, _ = self.down_proj(out)
        return out
```
**EN:** This class defines H Y V3 Feed Forward inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 H Y V3 Feed Forward，用于封装该模型组件的状态与方法。

### Lines 93-242: Class: HYV3MoEFused / 类：HYV3MoEFused
```python
class HYV3MoEFused(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ):
        super().__init__()
        self.tp_size = get_moe_tensor_parallel_world_size()
        self.ep_size = get_moe_expert_parallel_world_size()
        self.layer_id = layer_id
        self.alt_stream = alt_stream
        self.n_routed_experts = config.num_experts
        top_k = config.num_experts_per_tok
        intermediate_size = config.moe_intermediate_size

        self.expert_bias = nn.Parameter(
            torch.empty(config.num_experts, dtype=torch.float32)
        )
        self.expert_bias.weight_loader = HYV3MoEFused.ebias_weight_loader
        scoring_func = "sigmoid"
        self.e_score_correction_bias = self.expert_bias
        self.router_scaling_factor = getattr(config, "router_scaling_factor", 1.0)
        self.gate = ReplicatedLinear(
            config.hidden_size,
            config.num_experts,
            bias=False,
            quant_config=None,
            params_dtype=torch.float32,
            prefix=f"{prefix}.gate",
        )
        self.topk = TopK(
            top_k=config.num_experts_per_tok,
            use_grouped_topk=True,
            num_expert_group=1,
            topk_group=1,
            renormalize=config.route_norm,
            scoring_func=scoring_func,
# ... truncated for brevity ...
```
**EN:** This class defines H Y V3 Mo E Fused inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 H Y V3 Mo E Fused，用于封装该模型组件的状态与方法。

### Lines 245-336: Class: HYV3Attention / 类：HYV3Attention
```python
class HYV3Attention(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        rope_theta: float = 10000,
        rope_scaling: Optional[dict] = None,
        max_position_embeddings: int = 8192,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)

        self.head_dim = getattr(config, "head_dim", hidden_size // self.total_num_heads)
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.use_qk_norm = getattr(
            config, "use_qk_norm", getattr(config, "qk_norm", False)
        )

        self.qkv_proj = QKVParallelLinear(
            hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
# ... truncated for brevity ...
```
**EN:** This class defines H Y V3 Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 H Y V3 Attention，用于封装该模型组件的状态与方法。

### Lines 339-408: Class: HYV3DecoderLayer / 类：HYV3DecoderLayer
```python
class HYV3DecoderLayer(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.hidden_size = config.hidden_size
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        rope_theta, _ = get_rope_config(config)
        self.self_attn = HYV3Attention(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            rope_theta=rope_theta,
            max_position_embeddings=max_position_embeddings,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )
        self.input_layernorm = RMSNorm(config.hidden_size, config.rms_norm_eps)
        self.post_attention_layernorm = RMSNorm(config.hidden_size, config.rms_norm_eps)

        first_k_dense_replace = getattr(config, "first_k_dense_replace", 0)
        if layer_id < first_k_dense_replace:
            self.mlp = HYV3FeedForward(
                hidden_size=config.hidden_size,
                intermediate_size=config.intermediate_size,
                hidden_act=config.hidden_act,
                quant_config=quant_config,
                prefix=f"{prefix}.mlp",
            )
            self.block_type = "feedforward"
        else:
            self.mlp = HYV3MoEFused(
# ... truncated for brevity ...
```
**EN:** This class defines H Y V3 Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 H Y V3 Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 411-463: Class: HYV3Model / 类：HYV3Model
```python
class HYV3Model(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.quant_config = quant_config

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            prefix=f"{prefix}.embed_tokens",
        )

        self.alt_stream = torch.cuda.Stream() if is_cuda() else None

        self.layers = nn.ModuleList(
            [
                HYV3DecoderLayer(
                    config=config,
                    layer_id=i,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layers.{i}",
                    alt_stream=self.alt_stream,
                )
                for i in range(config.num_hidden_layers)
            ]
        )
        self.norm = RMSNorm(config.hidden_size, config.rms_norm_eps)

    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
# ... truncated for brevity ...
```
**EN:** This class defines H Y V3 Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 H Y V3 Model，用于封装该模型组件的状态与方法。

### Lines 466-593: Class: HYV3ForCausalLM / 类：HYV3ForCausalLM
```python
class HYV3ForCausalLM(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.quant_config = quant_config

        self.model = HYV3Model(config, quant_config, prefix=f"{prefix}.model")
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.lm_head",
        )
        if getattr(self.config, "tie_word_embeddings", False):
            self.lm_head.weight = self.model.embed_tokens.weight
        self.logits_processor = LogitsProcessor(config)

    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
    ) -> torch.Tensor:
        hidden_states = self.model(input_ids, positions, forward_batch, input_embeds)
        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch
        )

    def get_embed_and_head(self):
        return self.model.embed_tokens.weight, self.lm_head.weight

    def set_embed_and_head(self, embed, head):
        del self.model.embed_tokens.weight
# ... truncated for brevity ...
```
**EN:** This class defines H Y V3 For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 H Y V3 For Causal L M，用于封装该模型组件的状态与方法。

### Lines 596-596: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [HYV3ForCausalLM]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `typing: Iterable, Optional, Tuple`
- `torch`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.distributed: get_moe_expert_parallel_world_size, get_moe_tensor_parallel_world_size, get_tensor_model_parallel_world_size, moe_expert_parallel_all_reduce, moe_tensor_model_parallel_all_reduce`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, QKVParallelLinear, ReplicatedLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.moe: should_skip_post_experts_all_reduce`
- `sglang.srt.layers.moe.fused_moe_triton.layer: FusedMoE`
- `sglang.srt.layers.moe.topk: TopK`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead, VocabParallelEmbedding`
- `sglang.srt.managers.schedule_batch: ForwardBatch`
- `sglang.srt.model_executor.cuda_graph_runner: get_is_capture_mode`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.utils: is_cuda`
