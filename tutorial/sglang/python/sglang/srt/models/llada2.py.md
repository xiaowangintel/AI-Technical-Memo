# llada2.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/llada2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the llada2 architecture into SGLang's serving runtime. SGLang LLaDA2MoeModelLM model. / 该模块将 llada2 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：SGLang LLaDA2MoeModelLM model。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19: Module header and imports / 模块头与导入
```python
# coding=utf-8
# Copyright 2023 Antgroup and The HuggingFace Inc. team. All rights reserved.
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
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 93-93: Assignment: LoraConfig / 赋值：LoraConfig
```python
LoraConfig = None
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 94-94: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 95-95: Assignment: _is_cuda / 赋值：_is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 96-96: Assignment: _is_npu / 赋值：_is_npu
```python
_is_npu = is_npu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 99-151: Class: LLaDA2MoeMLP / 类：LLaDA2MoeMLP
```python
class LLaDA2MoeMLP(nn.Module):
    def __init__(
        self,
        intermediate_size: int,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        reduce_results: Optional[bool] = True,
        prefix: str = "",
        tp_rank: Optional[int] = None,
        tp_size: Optional[int] = None,
    ) -> None:
        super().__init__()
        self.tp_size = tp_size

        self.gate_up_proj = MergedColumnParallelLinear(
            config.hidden_size,
            [intermediate_size] * 2,
            bias=config.use_bias,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
            tp_rank=tp_rank,
            tp_size=tp_size,
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            config.hidden_size,
            bias=config.use_bias,
            reduce_results=reduce_results,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
            tp_rank=tp_rank,
            tp_size=tp_size,
        )

        if config.hidden_act != "silu":
            raise ValueError("Unsupported activation. Only silu is supported for now.")
        self.act_fn = SiluAndMul()

    def forward(
        self,
# ... truncated for brevity ...
```
**EN:** This class defines L La D A2 Moe M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 L La D A2 Moe M L P，用于封装该模型组件的状态与方法。

### Lines 154-182: Class: LLaDA2MoeGate / 类：LLaDA2MoeGate
```python
class LLaDA2MoeGate(nn.Module):
    def __init__(
        self,
        config,
        params_dtype: Optional[torch.dtype] = None,
        prefix: str = "",
    ):
        super().__init__()
        if params_dtype is None:
            params_dtype = torch.get_default_dtype()
        self.params_dtype = params_dtype
        self.weight = nn.Parameter(
            torch.empty(
                (config.num_experts, config.hidden_size),
                dtype=self.params_dtype,
            ),
        )
        if getattr(config, "moe_router_enable_expert_bias", False):
            self.expert_bias = nn.Parameter(
                torch.empty((config.num_experts,), dtype=torch.float32),
            )
        else:
            self.expert_bias = None

    def forward(self, hidden_states):
        logits = F.linear(hidden_states.to(self.weight.dtype), self.weight, None).to(
            hidden_states.dtype
        )
        return logits
```
**EN:** This class defines L La D A2 Moe Gate inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 L La D A2 Moe Gate，用于封装该模型组件的状态与方法。

### Lines 185-421: Class: LLaDA2MoeSparseMoeBlock / 类：LLaDA2MoeSparseMoeBlock
```python
class LLaDA2MoeSparseMoeBlock(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        alt_stream: Optional[torch.cuda.Stream] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.layer_id = layer_id
        self.alt_stream = alt_stream
        self.tp_size = get_tensor_model_parallel_world_size()
        self.top_k = config.num_experts_per_tok
        self.norm_topk_prob = config.norm_topk_prob
        self.hidden_size = config.hidden_size
        self.num_shared_experts = config.num_shared_experts
        self.routed_scaling_factor = getattr(config, "routed_scaling_factor", 1.0)
        self.score_function = getattr(config, "score_function", None)

        # fused_topk_npu() conducting norm before scale with routed_scaling_factor by default
        # norm_topk_prob=True will renorm the routed_scaling_factor thus need to keep norm_topk_prob=False
        if _is_npu:
            self.norm_topk_prob = False

        if config.hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {config.hidden_act}. "
                "Only silu is supported for now."
            )

        # Gate always runs at half / full precision for now.
        router_dtype = getattr(config, "router_dtype", None)
        if router_dtype is None:
            self.router_dtype = None
        elif router_dtype == "fp32":
            self.router_dtype = torch.float32
        else:
            self.router_dtype = torch.bfloat16

# ... truncated for brevity ...
```
**EN:** This class defines L La D A2 Moe Sparse Moe Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 L La D A2 Moe Sparse Moe Block，用于封装该模型组件的状态与方法。

### Lines 424-563: Class: LLaDA2MoeAttention / 类：LLaDA2MoeAttention
```python
class LLaDA2MoeAttention(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        reduce_results: bool = True,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.total_num_heads = config.num_attention_heads
        self.total_kv_heads = config.num_key_value_heads
        self.dp_size = get_attention_dp_size()
        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()

        assert self.total_num_heads % attn_tp_size == 0
        if self.total_kv_heads >= attn_tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_kv_heads % attn_tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert attn_tp_size % self.total_kv_heads == 0
        assert self.total_num_heads >= self.total_kv_heads

        self.num_heads = self.total_num_heads // attn_tp_size
        self.head_dim = config.head_dim or (self.hidden_size // self.total_num_heads)
        self.q_size = self.head_dim * self.num_heads

        self.num_kv_heads = max(1, self.total_kv_heads // attn_tp_size)
        self.kv_size = max(1, self.num_kv_heads * self.head_dim)

        self.scale = self.head_dim**-0.5

        self.use_qk_norm = getattr(config, "use_qk_norm", True)

# ... truncated for brevity ...
```
**EN:** This class defines L La D A2 Moe Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 L La D A2 Moe Attention，用于封装该模型组件的状态与方法。

### Lines 566-680: Class: LLaDA2MoeBlock / 类：LLaDA2MoeBlock
```python
class LLaDA2MoeBlock(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ):
        super().__init__()
        hidden_size = config.hidden_size

        self.input_layernorm = RMSNorm(hidden_size, eps=config.rms_norm_eps)
        self.dp_size = get_attention_dp_size()
        self.attention = LLaDA2MoeAttention(
            config,
            layer_id,
            quant_config,
            reduce_results=False,
            prefix=add_prefix("attention", prefix),
            alt_stream=alt_stream,
        )
        self.layer_id = layer_id
        self.attn_tp_size = get_attention_tp_size()
        self.attn_tp_rank = get_attention_tp_rank()

        self.is_layer_sparse = self._is_layer_sparse(config, layer_id=layer_id)
        is_previous_layer_sparse = self._is_layer_sparse(config, layer_id=layer_id - 1)
        is_next_layer_sparse = self._is_layer_sparse(config, layer_id=layer_id + 1)

        self.layer_scatter_modes = LayerScatterModes.init_new(
            layer_id=layer_id,
            num_layers=config.num_hidden_layers,
            is_layer_sparse=self.is_layer_sparse,
            is_previous_layer_sparse=is_previous_layer_sparse,
            is_next_layer_sparse=is_next_layer_sparse,
        )

        self.is_last_layer = self.layer_id == config.num_hidden_layers - 1

# ... truncated for brevity ...
```
**EN:** This class defines L La D A2 Moe Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 L La D A2 Moe Block，用于封装该模型组件的状态与方法。

### Lines 683-769: Class: LLaDA2MoeModel / 类：LLaDA2MoeModel
```python
class LLaDA2MoeModel(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        alt_stream: Optional[torch.cuda.Stream] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.vocab_size = config.vocab_size
        self.embed_dim = config.hidden_size
        if self.pp_group.is_first_rank:
            self.word_embeddings = VocabParallelEmbedding(
                self.vocab_size,
                self.embed_dim,
                quant_config=quant_config,
                prefix=add_prefix("word_embeddings", prefix),
                use_attn_tp_group=is_dp_attention_enabled(),
            )
        else:
            self.word_embeddings = PPMissingLayer()

        self.embedding_dropout = torch.nn.Dropout(config.embedding_dropout)

        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: LLaDA2MoeBlock(
                layer_id=idx,
                config=config,
                quant_config=quant_config,
                prefix=prefix,
                alt_stream=alt_stream,
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix=add_prefix("layers", prefix),
        )
# ... truncated for brevity ...
```
**EN:** This class defines L La D A2 Moe Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 L La D A2 Moe Model，用于封装该模型组件的状态与方法。

### Lines 772-950: Class: LLaDA2MoeModelLM / 类：LLaDA2MoeModelLM
```python
class LLaDA2MoeModelLM(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        alt_stream = torch.cuda.Stream() if _is_cuda else None

        self.model = LLaDA2MoeModel(
            config,
            quant_config,
            alt_stream=alt_stream,
            prefix=add_prefix("model", ""),
        )

        if config.tie_word_embeddings:
            self.lm_head = self.model.word_embeddings
        else:
            # TODO something wrong with ParallelLMHead with DP attention enabled
            self.lm_head = ParallelLMHead(
                config.vocab_size,
                config.hidden_size,
                quant_config=quant_config,
                prefix=add_prefix("lm_head", prefix),
                use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
            )
        self.logits_processor = LogitsProcessor(config, return_full_logits=True)

    @property
    def start_layer(self):
        return self.model.start_layer

    @property
    def end_layer(self):
        return self.model.end_layer
# ... truncated for brevity ...
```
**EN:** This class defines L La D A2 Moe Model L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 L La D A2 Moe Model L M，用于封装该模型组件的状态与方法。

### Lines 953-953: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = LLaDA2MoeModelLM
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `typing: Iterable, Optional, Tuple, Union`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.distributed: get_pp_group, get_tensor_model_parallel_world_size, parallel_state, tensor_model_parallel_all_reduce`
- `sglang.srt.eplb.expert_distribution: get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location: ModelConfigForExpertLocation`
- `sglang.srt.eplb.expert_location_dispatch: ExpertLocationDispatchInfo`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.communicator: LayerCommunicator, LayerScatterModes, enable_moe_dense_fully_dp`
- `sglang.srt.layers.dp_attention: get_attention_dp_size, get_attention_tp_rank, get_attention_tp_size, is_dp_attention_enabled`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.moe: get_deepep_mode, get_moe_a2a_backend, should_skip_post_experts_all_reduce`
- `sglang.srt.layers.moe.ep_moe.layer: get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton.layer: FusedMoE`
- `sglang.srt.layers.moe.token_dispatcher: DeepEPDispatcher`
