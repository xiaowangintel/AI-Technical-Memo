# gemma3_causal.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/gemma3_causal.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the gemma3 Causal architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 gemma3 Causal 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15: Module header and imports / 模块头与导入
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Copyright 2025 SGLang Team
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

### Lines 52-52: Assignment: _is_cpu / 赋值：_is_cpu
```python
_is_cpu = is_cpu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 53-53: Assignment: _is_cpu_amx_available / 赋值：_is_cpu_amx_available
```python
_is_cpu_amx_available = cpu_has_amx_support()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 58-59: Function: get_attention_sliding_window_size() / 函数：get_attention_sliding_window_size()
```python
def get_attention_sliding_window_size(config):
    return config.sliding_window - 1
```
**EN:** This function implements get attention sliding window size for the surrounding model/runtime logic. Key parameters include config.
**CN:** 该函数实现了 get attention sliding window size 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 64-74: Function: extract_layer_index() / 函数：extract_layer_index()
```python
def extract_layer_index(prefix: str) -> int:
    """Extract the layer index from a prefix string."""
    parts = prefix.split(".")
    for part in parts:
        if part.startswith("layers."):
            layer_str = part.split(".")[-1]
            try:
                return int(layer_str)
            except ValueError:
                continue
    return -1
```
**EN:** This function implements extract layer index for the surrounding model/runtime logic. Key parameters include prefix.
**CN:** 该函数实现了 extract layer index 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 77-114: Class: Gemma3MLP / 类：Gemma3MLP
```python
class Gemma3MLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_activation: str,
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
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
        )
        if hidden_activation != "gelu_pytorch_tanh":
            raise ValueError(
                f"{self.__class__.__name__} uses `gelu_pytorch_tanh` as the hidden activation "
                "function. Please set `hidden_activation` to "
                "`gelu_pytorch_tanh`."
            )
        self.act_fn = GeluAndMul()
        self.prefix = prefix

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This class defines Gemma3 M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3 M L P，用于封装该模型组件的状态与方法。

### Lines 117-322: Class: Gemma3Attention / 类：Gemma3Attention
```python
class Gemma3Attention(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: Gemma3TextConfig,
        max_position_embeddings: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.config = config
        tp_size = get_tensor_model_parallel_world_size()

        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = config.num_key_value_heads

        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)

        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0

        hidden_size = config.hidden_size

        head_dim = getattr(
            config, "head_dim", hidden_size // config.num_attention_heads
        )
        self.head_dim = head_dim
        partial_rotary_factor = getattr(config, "partial_rotary_factor", 1)
        self.rotary_dim = int(partial_rotary_factor * self.head_dim)
        self.q_size = self.num_heads * self.head_dim

# ... truncated for brevity ...
```
**EN:** This class defines Gemma3 Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3 Attention，用于封装该模型组件的状态与方法。

### Lines 325-403: Class: Gemma3DecoderLayer / 类：Gemma3DecoderLayer
```python
class Gemma3DecoderLayer(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.self_attn = Gemma3Attention(
            layer_id=layer_id,
            config=config,
            max_position_embeddings=config.max_position_embeddings,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
        )
        self.hidden_size = config.hidden_size
        self.mlp = Gemma3MLP(
            hidden_size=self.hidden_size,
            intermediate_size=config.intermediate_size,
            hidden_activation=config.hidden_activation,
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
        )
        self.input_layernorm = Gemma3RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )
        self.post_attention_layernorm = Gemma3RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )
        self.pre_feedforward_layernorm = Gemma3RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )
        self.post_feedforward_layernorm = Gemma3RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )
        self.is_sliding = self.self_attn.is_sliding
        self.layer_id = layer_id

# ... truncated for brevity ...
```
**EN:** This class defines Gemma3 Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3 Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 406-520: Class: Gemma3RotaryEmbedding / 类：Gemma3RotaryEmbedding
```python
class Gemma3RotaryEmbedding(nn.Module):
    def __init__(self, config: Gemma3TextConfig, device=None):
        super().__init__()
        # BC: "rope_type" was originally "type"
        rope_scaling = config.rope_parameters
        if rope_scaling is not None:
            self.rope_type = rope_scaling.get(
                "rope_type", rope_scaling.get("type", "default")
            )

        else:
            self.rope_type = "default"

        if self.rope_type is None:
            self.rope_type = "default"

        self.max_seq_len_cached = config.max_position_embeddings
        self.original_max_seq_len = config.max_position_embeddings

        self.config = config

        if self.rope_type == "default":
            self.rope_init_fn = self.compute_default_rope_parameters
        else:
            self.rope_init_fn = ROPE_INIT_FUNCTIONS[self.rope_type]

        inv_freq, self.attention_scaling = self.rope_init_fn(self.config, device)
        self.register_buffer("inv_freq", inv_freq, persistent=False)
        self.original_inv_freq = self.inv_freq

    def _dynamic_frequency_update(self, position_ids, device):
        """
        dynamic RoPE layers should recompute `inv_freq` in the following situations:
        1 - growing beyond the cached sequence length (allow scaling)
        2 - the current sequence length is in the original scale (avoid losing precision with small sequences)
        """
        seq_len = torch.max(position_ids) + 1
        if seq_len > self.max_seq_len_cached:  # growth
            inv_freq, self.attention_scaling = self.rope_init_fn(
                self.config, device, seq_len=seq_len
# ... truncated for brevity ...
```
**EN:** This class defines Gemma3 Rotary Embedding inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3 Rotary Embedding，用于封装该模型组件的状态与方法。

### Lines 523-539: Class: Gemma3TextScaledWordEmbedding / 类：Gemma3TextScaledWordEmbedding
```python
class Gemma3TextScaledWordEmbedding(nn.Embedding):
    """
    This module overrides nn.Embeddings' forward by multiplying with embeddings scale.
    """

    def __init__(
        self,
        num_embeddings: int,
        embedding_dim: int,
        padding_idx: int,
        embed_scale: Optional[float] = 1.0,
    ):
        super().__init__(num_embeddings, embedding_dim, padding_idx)
        self.embed_scale = embed_scale

    def forward(self, input_ids: torch.Tensor):
        return super().forward(input_ids) * self.embed_scale
```
**EN:** This class defines Gemma3 Text Scaled Word Embedding inheriting from nn.Embedding, grouping state and methods for this model component. Docstring summary: This module overrides nn.Embeddings' forward by multiplying with embeddings scale..
**CN:** 该类定义了 Gemma3 Text Scaled Word Embedding，用于封装该模型组件的状态与方法。 文档字符串摘要：This module overrides nn.Embeddings' forward by multiplying with embeddings scale.。

### Lines 542-670: Class: Gemma3TextModel / 类：Gemma3TextModel
```python
class Gemma3TextModel(PreTrainedModel):
    def __init__(
        self,
        config: Gemma3TextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__(config=config)
        self.config = config
        self.quant_config = quant_config

        self.padding_idx = config.pad_token_id
        self.vocab_size = config.vocab_size

        # Gemma3 downcasts the below to float16, causing sqrt(3072)=55.4256 to become 55.5. See https://github.com/huggingface/transformers/pull/29402
        self.embed_tokens = Gemma3TextScaledWordEmbedding(
            config.vocab_size,
            config.hidden_size,
            self.padding_idx,
            embed_scale=self.config.hidden_size**0.5,
        )

        self.norm = Gemma3RMSNorm(config.hidden_size, eps=config.rms_norm_eps)

        # In transformers v5, rope_parameters is nested per layer type:
        #   {"sliding_attention": {"rope_type": ..., "rope_theta": 10000},
        #    "full_attention":    {"rope_type": ..., "rope_theta": 1000000}}
        # Flatten into the format Gemma3RotaryEmbedding expects.
        rope_params = config.rope_parameters
        if isinstance(rope_params, dict) and "full_attention" in rope_params:
            global_theta = rope_params["full_attention"].get("rope_theta", 1000000.0)
            local_theta = rope_params["sliding_attention"].get("rope_theta", 10000.0)
        else:
            # v4 flat format fallback
            global_theta = (
                rope_params.get("rope_theta", 10000.0) if rope_params else 10000.0
            )
            local_theta = getattr(config, "rope_local_base_freq", 10000.0)

        global_config = copy.deepcopy(config)
# ... truncated for brevity ...
```
**EN:** This class defines Gemma3 Text Model inheriting from PreTrainedModel, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3 Text Model，用于封装该模型组件的状态与方法。

### Lines 673-928: Class: Gemma3ForCausalLM / 类：Gemma3ForCausalLM
```python
class Gemma3ForCausalLM(PreTrainedModel):
    config_class = Gemma3TextConfig

    _tied_weights_keys = {"lm_head.weight": "model.embed_tokens.weight"}
    _tp_plan = {"lm_head": "colwise_rep"}
    _pp_plan = {"lm_head": (["hidden_states"], ["logits"])}
    config_class = Gemma3TextConfig
    base_model_prefix = "language_model"

    # BitandBytes specific attributes
    default_bitsandbytes_target_modules = [
        ".gate_proj.",
        ".down_proj.",
        ".up_proj.",
        ".q_proj.",
        ".k_proj.",
        ".v_proj.",
        ".o_proj.",
    ]
    bitsandbytes_stacked_params_mapping = {
        # shard_name, weight_name, index
        "q_proj": ("qkv_proj", 0),
        "k_proj": ("qkv_proj", 1),
        "v_proj": ("qkv_proj", 2),
        "gate_proj": ("gate_up_proj", 0),
        "up_proj": ("gate_up_proj", 1),
    }

    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
        "gate_up_proj": [
            "gate_proj",
            "up_proj",
        ],
    }

# ... truncated for brevity ...
```
**EN:** This class defines Gemma3 For Causal L M inheriting from PreTrainedModel, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3 For Causal L M，用于封装该模型组件的状态与方法。

### Lines 931-931: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = Gemma3ForCausalLM
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `copy`
- `typing: Iterable, List, Optional, Set, Tuple`
- `einops`
- `torch`
- `torch: nn`
- `transformers: ROPE_INIT_FUNCTIONS, Gemma3TextConfig, PretrainedConfig, PreTrainedModel`
- `sglang.srt.distributed: get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation: GeluAndMul`
- `sglang.srt.layers.layernorm: Gemma3RMSNorm`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: AttentionType, RadixAttention`
- `sglang.srt.layers.rotary_embedding: apply_rotary_pos_emb, get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader, maybe_remap_kv_scale_name`
- `sglang.srt.utils: add_prefix, cpu_has_amx_support, is_cpu, make_layers`
