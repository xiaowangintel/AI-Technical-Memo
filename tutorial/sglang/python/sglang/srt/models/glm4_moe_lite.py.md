# glm4_moe_lite.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/glm4_moe_lite.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the glm4 Moe Lite architecture into SGLang's serving runtime. Inference-only GLM-4.7-Flash model compatible with HuggingFace weights. / 该模块将 glm4 Moe Lite 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only GLM-4.7-Flash model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Module header and imports / 模块头与导入
```python
# Copyright 2025-2026 SGLang Team
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

### Lines 80-80: Assignment: _is_cuda / 赋值：_is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 81-81: Assignment: _device_sm / 赋值：_device_sm
```python
_device_sm = get_device_sm()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 83-84: Conditional setup: _is_cuda / 条件初始化：_is_cuda
```python
if _is_cuda:
    from sgl_kernel import dsv3_router_gemm
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 86-86: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 89-164: Class: Glm4MoeLiteMLP / 类：Glm4MoeLiteMLP
```python
class Glm4MoeLiteMLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: Optional[QuantizationConfig] = None,
        reduce_results: bool = True,
        prefix: str = "",
        tp_rank: Optional[int] = None,
        tp_size: Optional[int] = None,
    ) -> None:
        super().__init__()
        self.tp_size = tp_size

        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
            tp_rank=tp_rank,
            tp_size=tp_size,
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            reduce_results=reduce_results,
            prefix=add_prefix("down_proj", prefix),
            tp_rank=tp_rank,
            tp_size=tp_size,
        )
        if hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()

# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe Lite M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Lite M L P，用于封装该模型组件的状态与方法。

### Lines 167-200: Class: Glm4MoeLiteGate / 类：Glm4MoeLiteGate
```python
class Glm4MoeLiteGate(nn.Module):
    def __init__(
        self,
        config,
        prefix: str = "",
        is_nextn: bool = False,
    ):
        super().__init__()
        self.is_nextn = is_nextn
        self.weight = nn.Parameter(
            torch.empty((config.n_routed_experts, config.hidden_size))
        )
        self.e_score_correction_bias = nn.Parameter(
            torch.empty((config.n_routed_experts), dtype=torch.float32)
        )

    def forward(self, hidden_states, gemm_output_zero_allocator: BumpAllocator = None):
        # NOTE: For some unknown reason, router_gemm seems degrade accept length.
        if (
            _is_cuda
            and not self.is_nextn
            and hidden_states.shape[0] < 4
            and hidden_states.shape[1] == 7168
            and self.weight.shape[0] == 256
            and _device_sm >= 90
        ):

            logits = dsv3_router_gemm(hidden_states, self.weight).to(
                hidden_states.dtype
            )
        else:
            logits = F.linear(hidden_states, self.weight, None)

        return logits
```
**EN:** This class defines Glm4 Moe Lite Gate inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Lite Gate，用于封装该模型组件的状态与方法。

### Lines 203-328: Class: Glm4MoeLiteSparseMoeBlock / 类：Glm4MoeLiteSparseMoeBlock
```python
class Glm4MoeLiteSparseMoeBlock(DeepseekV2MoE):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
        is_nextn: bool = False,
    ):
        nn.Module.__init__(self)
        self.tp_size = get_tensor_model_parallel_world_size()
        self.routed_scaling_factor = config.routed_scaling_factor
        self.n_shared_experts = config.n_shared_experts
        self.num_fused_shared_experts = (
            0
            if get_global_server_args().disable_shared_experts_fusion
            else config.n_shared_experts
        )
        self.config = config
        self.layer_id = layer_id
        self.alt_stream = alt_stream
        self.is_nextn = is_nextn

        if self.tp_size > config.n_routed_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {config.n_routed_experts}."
            )

        if config.hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {config.hidden_act}. "
                "Only silu is supported for now."
            )

        self.gate = Glm4MoeLiteGate(
            config=config, prefix=add_prefix("gate", prefix), is_nextn=is_nextn
        )

# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe Lite Sparse Moe Block inheriting from DeepseekV2MoE, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Lite Sparse Moe Block，用于封装该模型组件的状态与方法。

### Lines 331-419: Class: Glm4MoeLiteDecoderLayer / 类：Glm4MoeLiteDecoderLayer
```python
class Glm4MoeLiteDecoderLayer(DeepseekV2DecoderLayer):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        is_nextn: bool = False,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        nn.Module.__init__(self)
        self.hidden_size = config.hidden_size
        self.config = config
        self.nsa_enable_prefill_cp = is_nsa_enable_prefill_cp()
        rope_theta, rope_scaling = get_rope_config(config)
        max_position_embeddings = getattr(config, "max_position_embeddings", 202752)
        self.layer_id = layer_id

        self.self_attn = DeepseekV2AttentionMLA(
            config=config,
            hidden_size=config.hidden_size,
            num_heads=config.num_attention_heads,
            qk_nope_head_dim=config.qk_nope_head_dim,
            qk_rope_head_dim=config.qk_rope_head_dim,
            v_head_dim=config.v_head_dim,
            q_lora_rank=config.q_lora_rank,
            kv_lora_rank=config.kv_lora_rank,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            max_position_embeddings=max_position_embeddings,
            quant_config=quant_config,
            reduce_results=False,
            layer_id=layer_id,
            prefix=add_prefix("self_attn", prefix),
        )

        self.is_layer_sparse = self._is_layer_sparse(layer_id, is_nextn=is_nextn)
        is_previous_layer_sparse = self._is_layer_sparse(layer_id - 1, is_nextn=False)
        is_next_layer_sparse = self._is_layer_sparse(layer_id + 1, is_nextn=False)

# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe Lite Decoder Layer inheriting from DeepseekV2DecoderLayer, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Lite Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 422-468: Class: Glm4MoeLiteModel / 类：Glm4MoeLiteModel
```python
class Glm4MoeLiteModel(DeepseekV2Model):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        nn.Module.__init__(self)
        self.padding_id = config.pad_token_id
        self.vocab_size = config.vocab_size
        self.first_k_dense_replace = config.first_k_dense_replace
        self.pp_group = get_pp_group()

        # DeepseekV2Model.forward expects these attributes to exist.
        self.nsa_enable_prefill_cp = is_nsa_enable_prefill_cp()
        self.cp_size = get_attention_tp_size() if self.nsa_enable_prefill_cp else None
        self.gemm_output_zero_allocator_size = 0
        self.llama_4_scaling_config = getattr(config, "llama_4_scaling", None)

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                use_attn_tp_group=is_dp_attention_enabled(),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        self.alt_stream = torch.cuda.Stream() if _is_cuda else None
        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: Glm4MoeLiteDecoderLayer(
                config=config,
                layer_id=idx,
                quant_config=quant_config,
                prefix=prefix,
                alt_stream=self.alt_stream,
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe Lite Model inheriting from DeepseekV2Model, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Lite Model，用于封装该模型组件的状态与方法。

### Lines 471-789: Class: Glm4MoeLiteForCausalLM / 类：Glm4MoeLiteForCausalLM
```python
class Glm4MoeLiteForCausalLM(DeepseekV2ForCausalLM):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        nn.Module.__init__(self)
        config.moe_layer_freq = 1
        self.config = config
        self.tp_size = get_tensor_model_parallel_world_size()
        self.quant_config = quant_config
        self.pp_group = get_pp_group()
        self.determine_num_fused_shared_experts("Glm4MoeLiteForCausalLM")
        self.model = Glm4MoeLiteModel(
            config, quant_config, prefix=add_prefix("model", prefix)
        )
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("lm_head", prefix),
            use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
        )
        self.logits_processor = LogitsProcessor(config)

        self._routed_experts_weights_of_layer = LazyValue(
            lambda: {
                layer_id: layer.mlp.get_moe_weights()
                for layer_id, layer in enumerate(self.model.layers)
                if isinstance(layer.mlp, Glm4MoeLiteSparseMoeBlock)
            }
        )
        self.capture_aux_hidden_states = False

        self.nsa_enable_prefill_cp = is_nsa_enable_prefill_cp()
        if self.nsa_enable_prefill_cp:
            self.cp_rank = get_attention_tp_rank()
            self.cp_size = get_attention_tp_size()
        else:
# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe Lite For Causal L M inheriting from DeepseekV2ForCausalLM, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Lite For Causal L M，用于封装该模型组件的状态与方法。

### Lines 792-792: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [Glm4MoeLiteForCausalLM]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `re`
- `typing: Iterable, Optional, Tuple`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.batch_overlap.single_batch_overlap: SboFlags`
- `sglang.srt.distributed: get_moe_expert_parallel_world_size, get_pp_group, get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.attention.nsa.utils: is_nsa_enable_prefill_cp`
- `sglang.srt.layers.communicator: LayerCommunicator, LayerScatterModes, enable_moe_dense_fully_dp`
- `sglang.srt.layers.dp_attention: get_attention_tp_rank, get_attention_tp_size, is_dp_attention_enabled`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.moe: get_moe_a2a_backend, should_use_flashinfer_cutlass_moe_fp4_allgather`
- `sglang.srt.layers.moe.ep_moe.layer: get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton.layer: FusedMoE`
- `sglang.srt.layers.moe.topk: TopK, TopKOutputFormat`
