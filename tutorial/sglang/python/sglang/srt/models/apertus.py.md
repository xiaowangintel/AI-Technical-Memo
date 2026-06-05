# apertus.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/apertus.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Apertus architecture into SGLang's serving runtime. Inference-only Apertus model compatible with HuggingFace weights. / 该模块将 Apertus 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only Apertus model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19: Module header and imports / 模块头与导入
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Copyright 2025 The SwissAI Initiative
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

# Adapted from
# https://github.com/vllm-project/vllm/blob/c7f2cf2b7f67bce5842fedfdba508440fe257375/vllm/model_executor/models/llama.py#L1
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 60-60: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 63-110: Class: ApertusMLP / 类：ApertusMLP
```python
class ApertusMLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: Optional[QuantizationConfig] = None,
        bias: bool = False,
        prefix: str = "",
        reduce_results: bool = True,
    ) -> None:
        super().__init__()
        self.up_proj = ColumnParallelLinear(
            hidden_size,
            intermediate_size,
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("up_proj", prefix),
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
            reduce_results=reduce_results,
        )
        if hidden_act != "xielu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. "
                "Only xIELU is supported for now."
            )
        self.act_fn = XIELU()

    def forward(
        self,
        x,
        forward_batch=None,
        use_reduce_scatter: bool = False,
    ):
# ... truncated for brevity ...
```
**EN:** This class defines Apertus M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Apertus M L P，用于封装该模型组件的状态与方法。

### Lines 113-209: Class: ApertusAttention / 类：ApertusAttention
```python
class ApertusAttention(nn.Module):
    def __init__(
        self,
        config: ApertusConfig,
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
        bias_o_proj: bool = False,
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
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
        partial_rotary_factor = getattr(config, "partial_rotary_factor", 1)
        self.rotary_dim = int(partial_rotary_factor * self.head_dim)
# ... truncated for brevity ...
```
**EN:** This class defines Apertus Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Apertus Attention，用于封装该模型组件的状态与方法。

### Lines 212-291: Class: ApertusDecoderLayer / 类：ApertusDecoderLayer
```python
class ApertusDecoderLayer(nn.Module):
    def __init__(
        self,
        config: ApertusConfig,
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
        # Support llamafy/Qwen-Qwen2.5-7B-Instruct-llamafied with attention_bias
        # Support internlm/internlm-7b with bias
        attention_bias = getattr(config, "attention_bias", False) or getattr(
            config, "bias", False
        )
        bias_o_proj = attention_bias
        # support internlm/internlm3-8b with qkv_bias
        if hasattr(config, "qkv_bias"):
            attention_bias = config.qkv_bias
        self.self_attn = ApertusAttention(
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
**EN:** This class defines Apertus Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Apertus Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 294-404: Class: ApertusModel / 类：ApertusModel
```python
class ApertusModel(nn.Module):
    def __init__(
        self,
        config: ApertusConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.quant_config = quant_config
        self.config = config
        self.padding_idx = config.pad_token_id
        self.vocab_size = config.vocab_size
        self.org_vocab_size = config.vocab_size
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
            lambda idx, prefix: ApertusDecoderLayer(
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

# ... truncated for brevity ...
```
**EN:** This class defines Apertus Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Apertus Model，用于封装该模型组件的状态与方法。

### Lines 407-684: Class: ApertusForCausalLM / 类：ApertusForCausalLM
```python
class ApertusForCausalLM(nn.Module):
    # LoRA specific attributes
    embedding_modules = {
        "embed_tokens": "input_embeddings",
        "lm_head": "output_embeddings",
    }
    embedding_padding_modules = ["lm_head"]
    # BitandBytes specific attributes
    default_bitsandbytes_target_modules = [
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
        ".q_proj": (".qkv_proj", 0),
        ".k_proj": (".qkv_proj", 1),
        ".v_proj": (".qkv_proj", 2),
    }

    def __init__(
        self,
        config: ApertusConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        self.model = self._init_model(config, quant_config, add_prefix("model", prefix))
        if self.config.tie_word_embeddings:
            self.lm_head = self.model.embed_tokens
        else:
            self.lm_head = ParallelLMHead(
# ... truncated for brevity ...
```
**EN:** This class defines Apertus For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Apertus For Causal L M，用于封装该模型组件的状态与方法。

### Lines 687-687: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [ApertusForCausalLM]
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
- `transformers: ApertusConfig`
- `sglang.srt.distributed: get_pp_group, get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation: XIELU`
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
