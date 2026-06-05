# ernie4.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/ernie4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the ernie4 architecture into SGLang's serving runtime. Inference-only Ernie4.5 model compatible with baidu/ERNIE-4.5-*-PT weights. / 该模块将 ernie4 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only Ernie4.5 model compatible with baidu/ERNIE-4.5-*-PT weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Module header and imports / 模块头与导入
```python
# Copyright 2023-2025 SGLang Team
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
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 49-65: Class: MoEGate / 类：MoEGate
```python
class MoEGate(nn.Module):
    def __init__(
        self,
        config,
        prefix: str = "",
    ):
        super().__init__()
        self.weight = nn.Parameter(
            torch.empty((config.moe_num_experts, config.hidden_size))
        )
        self.e_score_correction_bias = nn.Parameter(
            torch.empty((1, config.moe_num_experts))
        )

    def forward(self, hidden_states):
        logits = F.linear(hidden_states, self.weight, None)
        return logits
```
**EN:** This class defines Mo E Gate inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Mo E Gate，用于封装该模型组件的状态与方法。

### Lines 68-140: Class: Ernie4Moe / 类：Ernie4Moe
```python
class Ernie4Moe(nn.Module):
    def __init__(
        self,
        config: Ernie4_5_MoeConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.layer_id = layer_id
        self.tp_size = get_tensor_model_parallel_world_size()
        self.moe_num_shared_experts = getattr(config, "moe_num_shared_experts", 0)

        if config.hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {config.hidden_act}. "
                "Only silu is supported for now."
            )

        self.gate = MoEGate(config=config, prefix=add_prefix("gate", prefix))

        self.topk = TopK(
            top_k=config.moe_k,
            layer_id=layer_id,
            renormalize=True,
            use_grouped_topk=False,
            correction_bias=self.gate.e_score_correction_bias,
        )

        self.experts = get_moe_impl_class(quant_config)(
            num_experts=config.moe_num_experts,
            top_k=config.moe_k,
            hidden_size=config.hidden_size,
            intermediate_size=config.moe_intermediate_size,
            layer_id=self.layer_id,
            quant_config=quant_config,
            prefix=add_prefix("experts", prefix),
        )

        if self.moe_num_shared_experts > 0:
# ... truncated for brevity ...
```
**EN:** This class defines Ernie4 Moe inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Ernie4 Moe，用于封装该模型组件的状态与方法。

### Lines 143-236: Class: Ernie4DecoderLayer / 类：Ernie4DecoderLayer
```python
class Ernie4DecoderLayer(nn.Module):
    """A single transformer layer.

    Transformer layer takes input with size [s, b, h] and returns an
    output of the same size.
    """

    def __init__(
        self,
        config,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        is_mtp: bool = False,
    ):
        super().__init__()
        rope_theta, rope_scaling = get_rope_config(config)
        rope_is_neox_style = getattr(config, "rope_is_neox_style", False)
        # Self attention.
        self.self_attn = Ernie4Attention(
            config=config,
            hidden_size=config.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            rope_is_neox_style=rope_is_neox_style,
            max_position_embeddings=config.max_position_embeddings,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
            bias=config.use_bias,
        )
        moe_layer_start_index = getattr(
            config, "moe_layer_start_index", config.num_hidden_layers
        )
        moe_layer_end_index = getattr(
            config, "moe_layer_end_index", config.num_hidden_layers - 1
        )
        # MLP
# ... truncated for brevity ...
```
**EN:** This class defines Ernie4 Decoder Layer inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: A single transformer layer..
**CN:** 该类定义了 Ernie4 Decoder Layer，用于封装该模型组件的状态与方法。 文档字符串摘要：A single transformer layer.。

### Lines 239-286: Class: Ernie4Model / 类：Ernie4Model
```python
class Ernie4Model(nn.Module):
    def __init__(
        self,
        config: Ernie4_5_MoeConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("embed_tokens", prefix),
        )
        self.layers = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: Ernie4DecoderLayer(
                config=config, layer_id=idx, quant_config=quant_config, prefix=prefix
            ),
            prefix="model.layers",
        )

        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)

    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, List[torch.Tensor]]]:
        if input_embeds is None:
            hidden_states = self.embed_tokens(input_ids)
        else:
            hidden_states = input_embeds
        residual = None
        for layer in self.layers:
            hidden_states, residual = layer(
# ... truncated for brevity ...
```
**EN:** This class defines Ernie4 Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Ernie4 Model，用于封装该模型组件的状态与方法。

### Lines 289-360: Class: Ernie4_5_ForCausalLM / 类：Ernie4_5_ForCausalLM
```python
class Ernie4_5_ForCausalLM(nn.Module):
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }
    stacked_params_mapping = [
        # (param_name, weight_name, shard_id)
        (".qkv_proj", ".q_proj", "q"),
        (".qkv_proj", ".k_proj", "k"),
        (".qkv_proj", ".v_proj", "v"),
        (".gate_up_proj", ".gate_proj", 0),
        (".gate_up_proj", ".up_proj", 1),
    ]

    def __init__(
        self,
        config: Ernie4_5_MoeConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config: Ernie4_5_MoeConfig = config
        self.quant_config = quant_config
        self.model = Ernie4Model(config, quant_config, add_prefix("model", prefix))
        if config.tie_word_embeddings:
            self.lm_head = self.model.embed_tokens
        else:
            self.lm_head = ParallelLMHead(
                config.vocab_size,
                config.hidden_size,
                quant_config=quant_config,
                prefix="lm_head",
            )
        self.logits_processor = LogitsProcessor(config)

    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
# ... truncated for brevity ...
```
**EN:** This class defines Ernie4 5 For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Ernie4 5 For Causal L M，用于封装该模型组件的状态与方法。

### Lines 363-424: Class: Ernie4_5_MoeForCausalLM / 类：Ernie4_5_MoeForCausalLM
```python
class Ernie4_5_MoeForCausalLM(Ernie4_5_ForCausalLM):
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        expert_params_mapping = FusedMoE.make_expert_params_mapping(
            ckpt_gate_proj_name="gate_proj",
            ckpt_down_proj_name="down_proj",
            ckpt_up_proj_name="up_proj",
            num_experts=self.config.moe_num_experts,
        )
        params_dict = dict(self.named_parameters())
        for name, loaded_weight in weights:
            if self.config.tie_word_embeddings and "lm_head.weight" in name:
                continue
            if name.startswith("model.mtp_"):
                continue
            if "moe_statics.e_score_correction_bias" in name:
                name = name.replace("moe_statics", "gate")
            for param_name, weight_name, shard_id in self.stacked_params_mapping:
                if weight_name not in name:
                    continue
                # We have mlp.experts[0].gate_proj in the checkpoint.
                # Since we handle the experts below in expert_params_mapping,
                # we need to skip here BEFORE we update the name, otherwise
                # name will be updated to mlp.experts[0].gate_up_proj, which
                # will then be updated below in expert_params_mapping
                # for mlp.experts[0].gate_gate_up_proj, which breaks load.
                if ("mlp.experts." in name) and name not in params_dict:
                    continue
                name = name.replace(weight_name, param_name)
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
                    if name in params_dict.keys():
                        param = params_dict[name]
# ... truncated for brevity ...
```
**EN:** This class defines Ernie4 5 Moe For Causal L M inheriting from Ernie4_5_ForCausalLM, grouping state and methods for this model component.
**CN:** 该类定义了 Ernie4 5 Moe For Causal L M，用于封装该模型组件的状态与方法。

### Lines 427-427: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [Ernie4_5_MoeForCausalLM, Ernie4_5_ForCausalLM]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `typing: Iterable, List, Optional, Tuple, Union`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `transformers.models.ernie4_5_moe.configuration_ernie4_5_moe: Ernie4_5_MoeConfig`
- `sglang.srt.distributed: get_tensor_model_parallel_world_size, tensor_model_parallel_all_reduce`
- `sglang.srt.layers.communicator: enable_moe_dense_fully_dp`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.moe.ep_moe.layer: get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton.layer: FusedMoE`
- `sglang.srt.layers.moe.topk: TopK`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead, VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.models.deepseek_v2: DeepseekV2MLP`
- `sglang.srt.models.llama: LlamaAttention`
- `sglang.srt.utils: add_prefix, make_layers`
- `sglang.srt.utils.hf_transformers_utils: get_rope_config`
