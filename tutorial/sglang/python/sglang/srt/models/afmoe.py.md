# afmoe.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/afmoe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Afmoe architecture into SGLang's serving runtime. Inference-only AfMoE model compatible with HuggingFace weights. / 该模块将 Afmoe 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only AfMoE model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13: Module header and imports / 模块头与导入
```python
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
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 63-63: Assignment: _is_npu / 赋值：_is_npu
```python
_is_npu = is_npu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 65-68: Conditional setup: _is_npu / 条件初始化：_is_npu
```python
if _is_npu:
    from sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu import (
        fused_moe_npu as fused_moe,
    )
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 71-78: Function: get_attention_sliding_window_size() / 函数：get_attention_sliding_window_size()
```python
def get_attention_sliding_window_size(config: PretrainedConfig) -> Optional[int]:
    sliding_window = getattr(config, "sliding_window", None)
    if sliding_window is None:
        return None
    if sliding_window <= 0:
        return None
    # Align with other local attention implementations (see gpt_oss).
    return sliding_window - 1
```
**EN:** This function implements get attention sliding window size for the surrounding model/runtime logic. Key parameters include config.
**CN:** 该函数实现了 get attention sliding window size 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 81-118: Class: AfmoeMLP / 类：AfmoeMLP
```python
class AfmoeMLP(nn.Module):

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
            prefix=add_prefix("gate_up_proj", prefix),
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            reduce_results=reduce_results,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
        )
        if hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This class defines Afmoe M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Afmoe M L P，用于封装该模型组件的状态与方法。

### Lines 121-295: Class: AfmoeMoE / 类：AfmoeMoE
```python
class AfmoeMoE(nn.Module):

    @staticmethod
    def _custom_routing_function(
        hidden_states: torch.Tensor,
        gating_output: torch.Tensor,
        topk: int,
        renormalize: bool,
        *,
        score_func: str,
        expert_bias: Optional[torch.Tensor],
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        logits = gating_output.to(torch.float32)
        if score_func == "sigmoid":
            scores = torch.sigmoid(logits)
            if expert_bias is not None:
                bias = expert_bias.to(scores.device, dtype=scores.dtype)
                scores_for_choice = scores + bias
                topk_ids = torch.topk(scores_for_choice, k=topk, dim=-1)[1]
                topk_weights = scores.gather(dim=-1, index=topk_ids)
            else:
                topk_weights, topk_ids = torch.topk(scores, k=topk, dim=-1)
        else:
            if expert_bias is not None:
                logits = logits + expert_bias.to(logits.device, dtype=logits.dtype)
            probs = F.softmax(logits, dim=-1)
            topk_weights, topk_ids = torch.topk(probs, k=topk, dim=-1)

        if renormalize:
            denom = topk_weights.sum(dim=-1, keepdim=True).clamp(min=1e-20)
            topk_weights = topk_weights / denom

        return topk_weights.to(torch.float32), topk_ids.to(torch.int32)

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
# ... truncated for brevity ...
```
**EN:** This class defines Afmoe Mo E inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Afmoe Mo E，用于封装该模型组件的状态与方法。

### Lines 298-416: Class: AfmoeAttention / 类：AfmoeAttention
```python
class AfmoeAttention(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
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

        rope_theta = config.rope_parameters["rope_theta"]
        rope_scaling = config.rope_parameters
        partial_rotary_factor = getattr(config, "partial_rotary_factor", 1.0)
        self.rotary_dim = int(self.head_dim * partial_rotary_factor)
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)

        layer_types = getattr(config, "layer_types", None)
        self.is_local_attention = (
            layer_types is not None and layer_types[layer_id] == "sliding_attention"
        )
# ... truncated for brevity ...
```
**EN:** This class defines Afmoe Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Afmoe Attention，用于封装该模型组件的状态与方法。

### Lines 419-494: Class: AfmoeDecoderLayer / 类：AfmoeDecoderLayer
```python
class AfmoeDecoderLayer(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.layer_id = layer_id

        self.self_attn = AfmoeAttention(
            config=config,
            hidden_size=config.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
        )

        use_moe = False
        if hasattr(config, "num_dense_layers"):
            use_moe = layer_id >= config.num_dense_layers
        elif (
            getattr(config, "num_experts", None) is not None
            and hasattr(config, "first_k_dense_replace")
            and hasattr(config, "moe_layer_freq")
        ):
            base = config.first_k_dense_replace
            freq = config.moe_layer_freq
            use_moe = layer_id >= base and (layer_id - base) % freq == 0

        if use_moe:
            self.mlp = AfmoeMoE(
                config=config,
                quant_config=quant_config,
# ... truncated for brevity ...
```
**EN:** This class defines Afmoe Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Afmoe Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 497-550: Class: AfmoeModel / 类：AfmoeModel
```python
class AfmoeModel(nn.Module):

    fall_back_to_pt_during_load = False

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.padding_idx = config.pad_token_id
        self.vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
        )
        self.layers = nn.ModuleList(
            [
                AfmoeDecoderLayer(
                    config,
                    layer_id,
                    quant_config=quant_config,
                    prefix=add_prefix(f"layers.{layer_id}", prefix),
                )
                for layer_id in range(config.num_hidden_layers)
            ]
        )
        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)

    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if input_embeds is None:
# ... truncated for brevity ...
```
**EN:** This class defines Afmoe Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Afmoe Model，用于封装该模型组件的状态与方法。

### Lines 553-641: Class: AfmoeForCausalLM / 类：AfmoeForCausalLM
```python
class AfmoeForCausalLM(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.model = AfmoeModel(
            config, quant_config, prefix=add_prefix("model", prefix)
        )
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("lm_head", prefix),
        )
        self.logits_processor = LogitsProcessor(config)

    def get_input_embeddings(self) -> nn.Embedding:
        return self.model.embed_tokens

    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        hidden_states = self.model(input_ids, positions, forward_batch, input_embeds)
        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch
        )

    def get_attention_sliding_window_size(self) -> Optional[int]:
        return get_attention_sliding_window_size(self.config)

# ... truncated for brevity ...
```
**EN:** This class defines Afmoe For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Afmoe For Causal L M，用于封装该模型组件的状态与方法。

### Lines 644-644: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = AfmoeForCausalLM
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `__future__: annotations`
- `functools`
- `typing: Iterable, Optional, Tuple`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.distributed: get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size, tensor_model_parallel_all_reduce`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: ColumnParallelLinear, MergedColumnParallelLinear, QKVParallelLinear, ReplicatedLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.moe.moe_runner: MoeRunnerConfig`
- `sglang.srt.layers.moe.moe_runner.triton_utils: fused_moe`
- `sglang.srt.layers.moe.topk: TopK`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead, VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
