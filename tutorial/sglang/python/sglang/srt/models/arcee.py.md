# arcee.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/arcee.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Arcee architecture into SGLang's serving runtime. Inference-only Arcee Foundational Model (AFM) compatible with HuggingFace weights. / 该模块将 Arcee 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only Arcee Foundational Model (AFM) compatible with HuggingFace weights。

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

### Lines 54-54: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 57-101: Class: ArceeMLP / 类：ArceeMLP
```python
class ArceeMLP(nn.Module):
    """
    MLP block for the Arcee model, using a ReLU-squared activation function.
    This differs from the Llama SwiGLU activation.
    """

    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        reduce_results: bool = True,
    ) -> None:
        super().__init__()
        # Arcee uses a single up-projection, not a merged gate/up projection.
        self.up_proj = ColumnParallelLinear(
            hidden_size,
            intermediate_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("up_proj", prefix),
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
            reduce_results=reduce_results,
        )
        if hidden_act != "relu2":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. "
                "Arcee model in SGLang only supports 'relu2'."
            )
        # The activation function is relu(x)^2
        self.act_fn = get_act_fn("relu2")

# ... truncated for brevity ...
```
**EN:** This class defines Arcee M L P inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: MLP block for the Arcee model, using a ReLU-squared activation function..
**CN:** 该类定义了 Arcee M L P，用于封装该模型组件的状态与方法。 文档字符串摘要：MLP block for the Arcee model, using a ReLU-squared activation function.。

### Lines 104-189: Class: ArceeAttention / 类：ArceeAttention
```python
class ArceeAttention(nn.Module):
    def __init__(
        self,
        config: LlamaConfig,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        rope_theta: float = 10000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        rope_is_neox_style: bool = True,
        max_position_embeddings: int = 8192,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        bias: bool = False,
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
        self.head_dim = getattr(config, "head_dim", None)
        if self.head_dim is None:
            self.head_dim = self.hidden_size // self.total_num_heads
        self.partial_rotary_factor = getattr(config, "partial_rotary_factor", 1)
        self.rotary_dim = int(self.partial_rotary_factor * self.head_dim)
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.rope_theta = rope_theta
        self.max_position_embeddings = max_position_embeddings

        self.qkv_proj = QKVParallelLinear(
# ... truncated for brevity ...
```
**EN:** This class defines Arcee Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Arcee Attention，用于封装该模型组件的状态与方法。

### Lines 192-263: Class: ArceeDecoderLayer / 类：ArceeDecoderLayer
```python
class ArceeDecoderLayer(nn.Module):
    def __init__(
        self,
        config: LlamaConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        rope_theta = config.rope_parameters["rope_theta"]
        rope_scaling = config.rope_parameters
        if rope_scaling is not None and getattr(
            config, "original_max_position_embeddings", None
        ):
            rope_scaling["original_max_position_embeddings"] = (
                config.original_max_position_embeddings
            )
        rope_is_neox_style = getattr(config, "rope_is_neox_style", True)
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        attention_bias = getattr(config, "attention_bias", False) or getattr(
            config, "bias", False
        )
        self.self_attn = ArceeAttention(
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
            bias=attention_bias,
        )
        self.mlp = ArceeMLP(
            hidden_size=self.hidden_size,
            intermediate_size=config.intermediate_size,
# ... truncated for brevity ...
```
**EN:** This class defines Arcee Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Arcee Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 266-369: Class: ArceeModel / 类：ArceeModel
```python
class ArceeModel(nn.Module):
    def __init__(
        self,
        config: LlamaConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
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
                quant_config=quant_config,
                prefix=add_prefix("embed_tokens", prefix),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: ArceeDecoderLayer(
                config=config, quant_config=quant_config, layer_id=idx, prefix=prefix
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix="model.layers",
        )

        if self.pp_group.is_last_rank:
            self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        else:
            self.norm = PPMissingLayer(return_tuple=True)
        self.layers_to_capture = []

    def forward(
        self,
# ... truncated for brevity ...
```
**EN:** This class defines Arcee Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Arcee Model，用于封装该模型组件的状态与方法。

### Lines 372-529: Class: ArceeForCausalLM / 类：ArceeForCausalLM
```python
class ArceeForCausalLM(nn.Module):
    # BitandBytes specific attributes
    default_bitsandbytes_target_modules = [
        # Note: gate_proj is removed compared to Llama
        ".down_proj.",
        ".up_proj.",
        ".q_proj.",
        ".k_proj.",
        ".v_proj.",
        ".o_proj.",
    ]
    # in TP, these weights are partitioned along the column dimension (dim=-1)
    column_parallel_weights_modules = [".down_proj.", ".o_proj."]
    bitsandbytes_stacked_params_mapping = {
        # shard_name, weight_name, index
        # Note: gate_proj and up_proj are removed as they are not stacked in ArceeMLP
        ".q_proj": (".qkv_proj", 0),
        ".k_proj": (".qkv_proj", 1),
        ".v_proj": (".qkv_proj", 2),
    }

    def __init__(
        self,
        config: LlamaConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        self.model = self._init_model(config, quant_config, add_prefix("model", prefix))
        # Arcee does not tie word embeddings
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("lm_head", prefix),
            use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
        )
# ... truncated for brevity ...
```
**EN:** This class defines Arcee For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Arcee For Causal L M，用于封装该模型组件的状态与方法。

### Lines 532-532: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [ArceeForCausalLM]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `typing: Any, Dict, Iterable, List, Optional, Tuple, Union`
- `torch`
- `torch: nn`
- `transformers: LlamaConfig`
- `sglang.srt.distributed: get_pp_group, get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation: get_act_fn`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: ColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor, LogitsProcessorOutput`
- `sglang.srt.layers.pooler: Pooler, PoolingType`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.utils: PPMissingLayer, get_layer_id`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead, VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch, PPProxyTensors`
- `sglang.srt.model_loader.weight_utils: default_weight_loader, kv_cache_scales_loader, maybe_remap_kv_scale_name`
- `sglang.srt.server_args: get_global_server_args`
- `sglang.srt.utils: add_prefix, make_layers`
