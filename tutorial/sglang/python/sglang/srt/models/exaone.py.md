# exaone.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/exaone.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Exaone architecture into SGLang's serving runtime. Inference-only Exaone model compatible with HuggingFace weights. / 该模块将 Exaone 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only Exaone model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16: Module header and imports / 模块头与导入
```python
# Copyright 2024 The LGcns AI Engineering Team
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

# Adapted from llama2.py
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 46-81: Class: ExaoneGatedMLP / 类：ExaoneGatedMLP
```python
class ExaoneGatedMLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: Optional[QuantizationConfig] = None,
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
        self.c_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("c_proj", prefix),
        )
        if hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. "
                "Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()

    def forward(self, x):
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.c_proj(x)
        return x
```
**EN:** This class defines Exaone Gated M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Exaone Gated M L P，用于封装该模型组件的状态与方法。

### Lines 84-173: Class: ExaoneAttention / 类：ExaoneAttention
```python
class ExaoneAttention(nn.Module):
    def __init__(
        self,
        config,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        rope_theta: float = 500000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        rope_is_neox_style: bool = True,
        max_position_embeddings: int = 4096,
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
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        # MistralConfig has an optional head_dim introduced by Mistral-Nemo
        self.head_dim = getattr(
            config, "head_dim", self.hidden_size // self.total_num_heads
        )
        self.rotary_dim = int(
            self.head_dim * getattr(config, "partial_rotary_factor", 1)
        )
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
# ... truncated for brevity ...
```
**EN:** This class defines Exaone Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Exaone Attention，用于封装该模型组件的状态与方法。

### Lines 176-241: Class: ExaoneDecoderLayer / 类：ExaoneDecoderLayer
```python
class ExaoneDecoderLayer(nn.Module):
    def __init__(
        self,
        config,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        rope_theta, rope_scaling = get_rope_config(config)
        if rope_scaling is not None and getattr(
            config, "original_max_position_embeddings", None
        ):
            rope_scaling["original_max_position_embeddings"] = (
                config.original_max_position_embeddings
            )
        rope_is_neox_style = getattr(config, "rope_is_neox_style", True)
        max_position_embeddings = getattr(config, "max_position_embeddings", 4096)
        self.self_attn = ExaoneAttention(
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
            prefix=add_prefix("self_attn", prefix),
        )
        self.mlp = ExaoneGatedMLP(
            hidden_size=self.hidden_size,
            intermediate_size=config.intermediate_size,
            hidden_act=config.activation_function,
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
        )
        rms_norm_eps = config.layer_norm_epsilon
# ... truncated for brevity ...
```
**EN:** This class defines Exaone Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Exaone Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 244-294: Class: ExaoneModel / 类：ExaoneModel
```python
class ExaoneModel(nn.Module):
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.padding_idx = config.pad_token_id
        self.vocab_size = config.vocab_size
        self.wte = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
        )
        self.h = nn.ModuleList(
            [
                ExaoneDecoderLayer(
                    config,
                    i,
                    quant_config=quant_config,
                    prefix=add_prefix(f"h.{i}", prefix),
                )
                for i in range(config.num_hidden_layers)
            ]
        )
        rms_norm_eps = config.layer_norm_epsilon
        self.ln_f = RMSNorm(config.hidden_size, eps=rms_norm_eps)

    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
    ) -> torch.Tensor:
        if input_embeds is None:
            hidden_states = self.wte(input_ids)
        else:
            hidden_states = input_embeds
# ... truncated for brevity ...
```
**EN:** This class defines Exaone Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Exaone Model，用于封装该模型组件的状态与方法。

### Lines 297-374: Class: ExaoneForCausalLM / 类：ExaoneForCausalLM
```python
class ExaoneForCausalLM(nn.Module):
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.transformer = ExaoneModel(
            config, quant_config=quant_config, prefix=add_prefix("transformer", prefix)
        )
        if self.config.tie_word_embeddings:
            self.lm_head = self.transformer.wte
        else:
            self.lm_head = ParallelLMHead(
                config.vocab_size,
                config.hidden_size,
                prefix=add_prefix("lm_head", prefix),
            )
        self.logits_processor = LogitsProcessor(config)

    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
    ) -> LogitsProcessorOutput:
        hidden_states = self.transformer(
            input_ids, positions, forward_batch, input_embeds
        )
        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch
        )

    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
# ... truncated for brevity ...
```
**EN:** This class defines Exaone For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Exaone For Causal L M，用于封装该模型组件的状态与方法。

### Lines 377-377: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = ExaoneForCausalLM
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `typing: Any, Dict, Iterable, Optional, Tuple`
- `torch`
- `torch: nn`
- `sglang.srt.distributed: get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor, LogitsProcessorOutput`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead, VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.utils: add_prefix`
- `sglang.srt.utils.hf_transformers_utils: get_rope_config`
