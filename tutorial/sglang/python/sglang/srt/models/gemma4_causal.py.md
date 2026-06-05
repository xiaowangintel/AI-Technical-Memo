# gemma4_causal.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/gemma4_causal.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the gemma4 Causal architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 gemma4 Causal 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Module header and imports / 模块头与导入
```python
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

### Lines 58-58: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 63-64: Function: get_attention_sliding_window_size() / 函数：get_attention_sliding_window_size()
```python
def get_attention_sliding_window_size(config):
    return config.sliding_window - 1
```
**EN:** This function implements get attention sliding window size for the surrounding model/runtime logic. Key parameters include config.
**CN:** 该函数实现了 get attention sliding window size 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 67-67: Assignment: Gemma4MLP / 赋值：Gemma4MLP
```python
Gemma4MLP = Gemma3MLP
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 68-68: Assignment: Gemma4TextScaledWordEmbedding / 赋值：Gemma4TextScaledWordEmbedding
```python
Gemma4TextScaledWordEmbedding = Gemma3TextScaledWordEmbedding
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 71-124: Class: Gemma4Router / 类：Gemma4Router
```python
class Gemma4Router(nn.Module):
    """Router for Gemma4 MoE that preprocesses input before projection.

    Applies RMSNorm (no learned weight), root_size scaling
    (hidden_size^{-0.5}), then a learned per-dimension scale before
    projecting to expert logits.

    This preprocessing is applied ONLY to the router's input, not to
    the expert MLPs' input.
    """

    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size

        # RMSNorm without learned weight — pure normalization only
        self.norm = Gemma4RMSNorm(
            self.hidden_size, eps=config.rms_norm_eps, with_scale=False
        )
        # Per-dimension learned scale, applied after norm + root_size
        self.scale = nn.Parameter(torch.ones(self.hidden_size))
        # Constant 1/sqrt(hidden_size) scaling factor
        self.register_buffer(
            "root_size",
            torch.tensor(self.hidden_size**-0.5),
            persistent=False,
        )
        # Project to expert logits; replicated across TP for consistent routing
        self.proj = ReplicatedLinear(
            self.hidden_size,
            config.num_experts,
            bias=False,
            quant_config=None,
            prefix=add_prefix("proj", prefix),
        )
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Router inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Router for Gemma4 MoE that preprocesses input before projection..
**CN:** 该类定义了 Gemma4 Router，用于封装该模型组件的状态与方法。 文档字符串摘要：Router for Gemma4 MoE that preprocesses input before projection.。

### Lines 127-204: Class: Gemma4MoE / 类：Gemma4MoE
```python
class Gemma4MoE(nn.Module):
    """Mixture of Experts for Gemma4.

    Wraps MoE implementation with custom routing. The router projection is
    external (Gemma4Router) — this class only handles expert dispatch.

    Gemma4 routing: softmax over ALL experts → top-k → renormalize.
    per_expert_scale is folded into routing weights for mathematical
    correctness with MoE's fused kernel.
    """

    def __init__(
        self,
        hidden_size: int,
        layer_id: int,
        config: Gemma4TextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.hidden_size = hidden_size
        self.num_experts = config.num_experts
        self.tp_size = get_tensor_model_parallel_world_size()

        # Per-expert output scale folded into routing weights so that
        # MoE's fused kernel computes: Σ_e (expert_e * w_e * scale_e)
        self.per_expert_scale = nn.Parameter(torch.ones(config.num_experts))

        # Capture param directly to avoid closing over self in the routing closure.
        per_expert_scale = self.per_expert_scale

        def routing_function(
            hidden_states: torch.Tensor,
            gating_output: torch.Tensor,
            topk: int,
            renormalize: bool,  # always True for Gemma4; softmax identity only holds when renormalizing
        ) -> tuple[torch.Tensor, torch.Tensor]:
            # softmax(all)[topk] / sum(softmax(all)[topk]) = softmax(topk_logits),
            # so we softmax only the top-k logits (fewer kernel launches).
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Mo E inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Mixture of Experts for Gemma4..
**CN:** 该类定义了 Gemma4 Mo E，用于封装该模型组件的状态与方法。 文档字符串摘要：Mixture of Experts for Gemma4.。

### Lines 207-425: Class: Gemma4Attention / 类：Gemma4Attention
```python
class Gemma4Attention(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: Gemma4TextConfig,
        head_dim: int,
        max_position_embeddings: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.layer_id = layer_id
        self.config = config
        tp_size = get_tensor_model_parallel_world_size()

        layer_type = config.layer_types[layer_id]
        self.sliding_window = (
            config.sliding_window if layer_type == "sliding_attention" else None
        )

        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size

        if layer_type == "sliding_attention":
            self.total_num_kv_heads = getattr(
                config, "swa_num_key_value_heads", config.num_key_value_heads
            )
        else:
            self.total_num_kv_heads = config.num_key_value_heads

        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)

        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0

        hidden_size = config.hidden_size
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Attention，用于封装该模型组件的状态与方法。

### Lines 428-661: Class: Gemma4DecoderLayer / 类：Gemma4DecoderLayer
```python
class Gemma4DecoderLayer(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.hidden_size_per_layer_input = (
            getattr(config, "hidden_size_per_layer_input", None) or 0
        )

        self.layer_id = layer_id

        # Gemma 4 uses different head dimensions for sliding vs full attention
        layer_type = config.layer_types[layer_id]
        self.is_full_attention = layer_type == "full_attention"
        if self.is_full_attention:
            head_dim = config.head_dim  # following sglang naming
        else:
            head_dim = getattr(config, "swa_head_dim", config.head_dim)

        self.self_attn = Gemma4Attention(
            layer_id=layer_id,
            config=config,
            max_position_embeddings=config.max_position_embeddings,
            head_dim=head_dim,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
        )

        first_kv_shared_layer_idx = config.num_hidden_layers - getattr(
            config, "num_kv_shared_layers", 0
        )
        is_kv_shared_layer = self.layer_id >= first_kv_shared_layer_idx > 0
        use_double_wide_mlp = (
            getattr(config, "use_double_wide_mlp", False) and is_kv_shared_layer
        )
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 664-869: Class: Gemma4TextModel / 类：Gemma4TextModel
```python
class Gemma4TextModel(PreTrainedModel):
    def __init__(
        self,
        config: Gemma4TextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__(config=config)
        self.config = config
        self.quant_config = quant_config
        self.vocab_size = config.vocab_size
        self.padding_idx = getattr(config, "pad_token_id", None)

        self.embed_tokens = Gemma4TextScaledWordEmbedding(
            config.vocab_size,
            config.hidden_size,
            self.padding_idx,
            embed_scale=self.config.hidden_size**0.5,  # embedded normalizer
        )

        # Per-layer input embeddings
        self.hidden_size = config.hidden_size
        self.hidden_size_per_layer_input = (
            getattr(config, "hidden_size_per_layer_input", None) or 0
        )
        self.vocab_size_per_layer_input = (
            getattr(config, "vocab_size_per_layer_input", None) or config.vocab_size
        )

        if self.hidden_size_per_layer_input and self.hidden_size_per_layer_input > 0:
            self.embed_tokens_per_layer = Gemma4TextScaledWordEmbedding(
                self.vocab_size_per_layer_input,
                config.num_hidden_layers * self.hidden_size_per_layer_input,
                self.padding_idx,
                embed_scale=self.hidden_size_per_layer_input**0.5,
            )

            self.per_layer_model_projection = ReplicatedLinear(
                self.hidden_size,
                config.num_hidden_layers * self.hidden_size_per_layer_input,
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Text Model inheriting from PreTrainedModel, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Text Model，用于封装该模型组件的状态与方法。

### Lines 872-1136: Class: Gemma4ForCausalLM / 类：Gemma4ForCausalLM
```python
class Gemma4ForCausalLM(PreTrainedModel):
    config_class = Gemma4TextConfig
    base_model_prefix = "language_model"
    _tied_weights_keys = {"lm_head.weight": "model.embed_tokens.weight"}
    _tp_plan = {"lm_head": "colwise_rep"}

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

    # Gemma does not apply LoRA to the embedding layer.
    embedding_modules = {}
    embedding_padding_modules = []
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 For Causal L M inheriting from PreTrainedModel, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 For Causal L M，用于封装该模型组件的状态与方法。

### Lines 1139-1139: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = Gemma4ForCausalLM
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `re`
- `typing: Iterable, List, Optional, Set, Tuple`
- `torch`
- `torch: nn`
- `transformers: Gemma4TextConfig, PretrainedConfig, PreTrainedModel`
- `sglang.srt.distributed: get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size`
- `sglang.srt.layers.gemma4_fused_ops: gemma_dual_rmsnorm_residual_scalar, gemma_qkv_rmsnorm, gemma_rmsnorm_residual_scalar`
- `sglang.srt.layers.layernorm: Gemma4RMSNorm, RMSNorm`
- `sglang.srt.layers.linear: QKVParallelLinear, ReplicatedLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.moe.ep_moe.layer: get_moe_impl_class`
- `sglang.srt.layers.moe.topk: TopK`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader, maybe_remap_kv_scale_name`
- `sglang.srt.models.gemma3_causal: Gemma3MLP, Gemma3TextScaledWordEmbedding`
