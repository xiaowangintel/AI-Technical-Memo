# commandr.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/commandr.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Commandr architecture into SGLang's serving runtime. PyTorch Cohere model. / 该模块将 Commandr 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：PyTorch Cohere model。

## Line-by-Line Analysis / 逐行分析

### Lines 1-41: Module header and imports / 模块头与导入
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/main/vllm/model_executor/models/commandr.py

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
# Copyright 2024 Cohere and the HuggingFace Inc. team. All rights reserved.
#
# This code is based on EleutherAI's GPT-NeoX library and the GPT-NeoX
# and OPT implementations in this library. It has been modified from its
# original forms to accommodate minor architectural differences compared
# to GPT-NeoX and OPT used by the Meta AI team that trained the model.
#
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

# Adapted from
# https://github.com/vllm-project/vllm/blob/c7f2cf2b7f67bce5842fedfdba508440fe257375/vllm/model_executor/models/commandr.py#L1

# ... truncated for brevity ...
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 76-83: Function: layer_norm_func() / 函数：layer_norm_func()
```python
def layer_norm_func(hidden_states, weight, variance_epsilon):
    input_dtype = hidden_states.dtype
    hidden_states = hidden_states.to(torch.float32)
    mean = hidden_states.mean(-1, keepdim=True)
    variance = (hidden_states - mean).pow(2).mean(-1, keepdim=True)
    hidden_states = (hidden_states - mean) * torch.rsqrt(variance + variance_epsilon)
    hidden_states = weight.to(torch.float32) * hidden_states
    return hidden_states.to(input_dtype)
```
**EN:** This function implements layer norm func for the surrounding model/runtime logic. Key parameters include hidden_states, weight, variance_epsilon.
**CN:** 该函数实现了 layer norm func 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 86-108: Class: LayerNorm / 类：LayerNorm
```python
class LayerNorm(nn.Module):
    def __init__(self, param_shape=None, eps=1e-5):
        super().__init__()
        self.weight = nn.Parameter(torch.ones(param_shape))
        self.variance_epsilon = eps
        set_weight_attrs(self.weight, {"weight_loader": self.weight_loader})

    def forward(self, hidden_states, residuals=None):
        hidden_states = layer_norm_func(
            hidden_states, self.weight, self.variance_epsilon
        )
        return hidden_states, residuals

    def weight_loader(self, param: Parameter, loaded_weight: torch.Tensor):
        tp_rank = get_tensor_model_parallel_rank()
        shard_dim = 0 if param.dim() != 1 else None
        param_data = param.data
        if shard_dim is not None:
            shard_size = param_data.shape[shard_dim]
            start_idx = tp_rank * shard_size
            loaded_weight = loaded_weight.narrow(shard_dim, start_idx, shard_size)
        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This class defines Layer Norm inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Layer Norm，用于封装该模型组件的状态与方法。

### Lines 112-143: Class: CohereMLP / 类：CohereMLP
```python
class CohereMLP(nn.Module):
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.intermediate_size = config.intermediate_size
        self.gate_up_proj = MergedColumnParallelLinear(
            self.hidden_size,
            [self.intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
        )
        self.down_proj = RowParallelLinear(
            self.intermediate_size,
            self.hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
        )
        self.act_fn = SiluAndMul()

    def forward(self, x):
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This class defines Cohere M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Cohere M L P，用于封装该模型组件的状态与方法。

### Lines 146-258: Class: CohereAttention / 类：CohereAttention
```python
class CohereAttention(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        tp_size = get_tensor_model_parallel_world_size()
        self.config = config
        self.attention_dropout = config.attention_dropout
        self.hidden_size = config.hidden_size
        self.total_num_heads = config.num_attention_heads
        self.num_heads = self.total_num_heads // tp_size
        self.head_dim = self.hidden_size // self.total_num_heads
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.max_position_embeddings = getattr(
            config, "model_max_length", None
        ) or getattr(config, "max_position_embeddings", 8192)
        self.rope_theta = config.rope_parameters["rope_theta"]
        self.rope_scaling = config.rope_parameters
        self.use_qk_norm = getattr(config, "use_qk_norm", False)
        self.qkv_proj = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
# ... truncated for brevity ...
```
**EN:** This class defines Cohere Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Cohere Attention，用于封装该模型组件的状态与方法。

### Lines 261-307: Class: CohereDecoderLayer / 类：CohereDecoderLayer
```python
class CohereDecoderLayer(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size

        self.self_attn = CohereAttention(
            config,
            layer_id=layer_id,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
        )

        self.mlp = CohereMLP(
            config,
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
        )
        self.input_layernorm = LayerNorm(
            param_shape=(config.hidden_size), eps=config.layer_norm_eps
        )

    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        # Self Attention
        residual = hidden_states
        hidden_states, residual = self.input_layernorm(hidden_states, residual)
        hidden_states_attention = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
# ... truncated for brevity ...
```
**EN:** This class defines Cohere Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Cohere Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 310-355: Class: CohereModel / 类：CohereModel
```python
class CohereModel(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.vocab_size = config.vocab_size
        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size, config.hidden_size
        )
        self.layers = nn.ModuleList(
            [
                CohereDecoderLayer(
                    config,
                    i,
                    quant_config=quant_config,
                    prefix=add_prefix(f"layers.{i}", prefix),
                )
                for i in range(config.num_hidden_layers)
            ]
        )
        self.norm = LayerNorm(
            param_shape=(config.hidden_size), eps=config.layer_norm_eps
        )

    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        hidden_states = self.embed_tokens(input_ids)
        residual = None
        for i in range(len(self.layers)):
            layer = self.layers[i]
            hidden_states, residual = layer(
                positions,
# ... truncated for brevity ...
```
**EN:** This class defines Cohere Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Cohere Model，用于封装该模型组件的状态与方法。

### Lines 358-429: Class: CohereForCausalLM / 类：CohereForCausalLM
```python
class CohereForCausalLM(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.logit_scale = getattr(config, "logit_scale", None)
        self.logits_processor = LogitsProcessor(config, logit_scale=self.logit_scale)
        self.model = CohereModel(
            config, quant_config, prefix=add_prefix("model", prefix)
        )

    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        hidden_states = self.model(
            input_ids,
            positions,
            forward_batch,
        )
        return self.logits_processor(
            input_ids, hidden_states, self.model.embed_tokens, forward_batch
        )

    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
# ... truncated for brevity ...
```
**EN:** This class defines Cohere For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Cohere For Causal L M，用于封装该模型组件的状态与方法。

### Lines 432-433: Class: Cohere2ForCausalLM / 类：Cohere2ForCausalLM
```python
class Cohere2ForCausalLM(CohereForCausalLM):
    pass
```
**EN:** This class defines Cohere2 For Causal L M inheriting from CohereForCausalLM, grouping state and methods for this model component.
**CN:** 该类定义了 Cohere2 For Causal L M，用于封装该模型组件的状态与方法。

### Lines 436-436: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [CohereForCausalLM, Cohere2ForCausalLM]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `typing: Iterable, Optional, Tuple`
- `torch`
- `torch.utils.checkpoint`
- `torch: nn`
- `torch.nn.parameter: Parameter`
- `transformers: Cohere2Config, CohereConfig, PretrainedConfig`
- `sglang.srt.distributed: get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader, maybe_remap_kv_scale_name`
- `sglang.srt.utils: add_prefix, get_compiler_backend, set_weight_attrs`
