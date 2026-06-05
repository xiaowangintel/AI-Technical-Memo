# glm4_moe.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/glm4_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the glm4 Moe architecture into SGLang's serving runtime. Inference-only GLM-4.5, GLM-4.6 and GLM-4.7 model compatible with HuggingFace weights. / 该模块将 glm4 Moe 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only GLM-4.5, GLM-4.6 and GLM-4.7 model compatible with HuggingFace weights。

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

### Lines 105-105: Assignment: _is_hip / 赋值：_is_hip
```python
_is_hip = is_hip()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 106-106: Assignment: _is_cuda / 赋值：_is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 107-107: Assignment: _is_fp8_fnuz / 赋值：_is_fp8_fnuz
```python
_is_fp8_fnuz = is_fp8_fnuz()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 108-108: Assignment: _use_aiter / 赋值：_use_aiter
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 109-109: Assignment: _is_cpu_amx_available / 赋值：_is_cpu_amx_available
```python
_is_cpu_amx_available = cpu_has_amx_support()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 110-110: Assignment: _is_cpu / 赋值：_is_cpu
```python
_is_cpu = is_cpu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 111-111: Assignment: _is_npu / 赋值：_is_npu
```python
_is_npu = is_npu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 112-112: Assignment: _device_sm / 赋值：_device_sm
```python
_device_sm = get_device_sm()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 114-114: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 116-122: Conditional setup: _is_npu / 条件初始化：_is_npu
```python
if _is_npu:
    from sgl_kernel_npu.norm.split_qkv_rmsnorm_rope import split_qkv_rmsnorm_rope

    from sglang.srt.hardware_backend.npu.utils import (
        process_shared_expert,
        wait_share_stream,
    )
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 125-180: Class: Glm4MoeMLP / 类：Glm4MoeMLP
```python
class Glm4MoeMLP(nn.Module):
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
**EN:** This class defines Glm4 Moe M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe M L P，用于封装该模型组件的状态与方法。

### Lines 183-365: Class: Glm4MoeAttention / 类：Glm4MoeAttention
```python
class Glm4MoeAttention(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        rope_theta: float = 1000000,
        partial_rotary_factor: float = 0.5,
        rope_scaling: Optional[Dict[str, Any]] = None,
        max_position_embeddings: int = 8192,
        head_dim: Optional[int] = None,
        rms_norm_eps: float = 1e-05,
        attention_bias: bool = True,
        quant_config: Optional[QuantizationConfig] = None,
        use_qk_norm: bool = False,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size

        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()

        self.total_num_heads = num_heads
        assert self.total_num_heads % attn_tp_size == 0
        self.num_heads = self.total_num_heads // attn_tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= attn_tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % attn_tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert attn_tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // attn_tp_size)
        self.head_dim = head_dim or hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Attention，用于封装该模型组件的状态与方法。

### Lines 368-389: Class: Glm4MoeGate / 类：Glm4MoeGate
```python
class Glm4MoeGate(nn.Module):
    def __init__(
        self,
        config,
        prefix: str = "",
    ):
        super().__init__()
        self.weight = nn.Parameter(
            torch.empty((config.n_routed_experts, config.hidden_size))
        )
        self.e_score_correction_bias = nn.Parameter(
            torch.empty((config.n_routed_experts), dtype=torch.float32)
        )
        # GLM requires FP32 gate projection; cache to avoid per-forward cast.
        # FIXME: if gate weight is updated at runtime (e.g. expert rebalancing), _weight_fp32 must be invalidated.
        self.register_buffer("_weight_fp32", None, persistent=False)

    def forward(self, hidden_states):
        if self._weight_fp32 is None:
            self._weight_fp32 = self.weight.data.to(torch.float32)
        logits = F.linear(hidden_states.to(torch.float32), self._weight_fp32, None)
        return logits
```
**EN:** This class defines Glm4 Moe Gate inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Gate，用于封装该模型组件的状态与方法。

### Lines 392-783: Class: Glm4MoeSparseMoeBlock / 类：Glm4MoeSparseMoeBlock
```python
class Glm4MoeSparseMoeBlock(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ):
        nn.Module.__init__(self)
        self.top_k = config.num_experts_per_tok
        self.tp_size = get_tensor_model_parallel_world_size()
        self.moe_ep_size = get_moe_expert_parallel_world_size()
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

        self.gate = Glm4MoeGate(config=config, prefix=add_prefix("gate", prefix))

        self.experts = get_moe_impl_class(quant_config)(
# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe Sparse Moe Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Sparse Moe Block，用于封装该模型组件的状态与方法。

### Lines 786-1050: Class: Glm4MoeDecoderLayer / 类：Glm4MoeDecoderLayer
```python
class Glm4MoeDecoderLayer(nn.Module):
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
        rope_theta, rope_scaling = get_rope_config(config)
        partial_rotary_factor = (rope_scaling or {}).get("partial_rotary_factor")
        if partial_rotary_factor is None:
            partial_rotary_factor = getattr(config, "partial_rotary_factor", 0.5)
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        head_dim = getattr(
            config, "head_dim", config.hidden_size // config.num_attention_heads
        )
        rms_norm_eps = config.rms_norm_eps
        attention_bias = config.attention_bias
        self.layer_id = layer_id

        use_qk_norm = config.use_qk_norm if hasattr(config, "use_qk_norm") else False

        self.self_attn = Glm4MoeAttention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            partial_rotary_factor=partial_rotary_factor,
            max_position_embeddings=max_position_embeddings,
            head_dim=head_dim,
            rms_norm_eps=rms_norm_eps,
            attention_bias=attention_bias,
            quant_config=quant_config,
# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 1053-1170: Class: Glm4MoeModel / 类：Glm4MoeModel
```python
class Glm4MoeModel(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.vocab_size = config.vocab_size
        self.first_k_dense_replace = config.first_k_dense_replace
        self.embed_dim = config.hidden_size
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
            lambda idx, prefix: Glm4MoeDecoderLayer(
                layer_id=idx,
                config=config,
                quant_config=quant_config,
                prefix=prefix,
                alt_stream=self.alt_stream,
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix=add_prefix("layers", prefix),
        )
        if self.pp_group.is_last_rank:
            self.norm = RMSNorm(self.embed_dim, eps=config.rms_norm_eps)
        else:
            self.norm = PPMissingLayer(return_tuple=True)
# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Model，用于封装该模型组件的状态与方法。

### Lines 1173-1480: Class: Glm4MoeForCausalLM / 类：Glm4MoeForCausalLM
```python
class Glm4MoeForCausalLM(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        nn.Module.__init__(self)
        self.pp_group = get_pp_group()
        self.config = config
        self.tp_size = get_tensor_model_parallel_world_size()
        self.quant_config = quant_config
        self.num_fused_shared_experts = 0
        self.determine_num_fused_shared_experts()
        self.model = Glm4MoeModel(
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

        # For EAGLE3 support
        self.capture_aux_hidden_states = False

    def determine_num_fused_shared_experts(self):
        if get_global_server_args().disable_shared_experts_fusion:
            return

        disable_reason = None
        if (not _is_cuda or torch.cuda.get_device_capability("cuda") < (8, 0)) and (
            not _is_hip or torch.cuda.get_device_capability("cuda") < (9, 4)
        ):
            disable_reason = (
                "Only GLM-4.5 on NV-platform with capability >= 80 "
                "or AMD-platform with capability >= gfx942(MI30x) can use shared experts fusion optimization."
# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe For Causal L M，用于封装该模型组件的状态与方法。

### Lines 1483-1485: Class: GlmMoeDsaForCausalLM / 类：GlmMoeDsaForCausalLM
```python
class GlmMoeDsaForCausalLM(DeepseekV2ForCausalLM):
    def determine_num_fused_shared_experts(self):
        super().determine_num_fused_shared_experts("GlmMoeDsaForCausalLM")
```
**EN:** This class defines Glm Moe Dsa For Causal L M inheriting from DeepseekV2ForCausalLM, grouping state and methods for this model component.
**CN:** 该类定义了 Glm Moe Dsa For Causal L M，用于封装该模型组件的状态与方法。

### Lines 1488-1488: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [Glm4MoeForCausalLM, GlmMoeDsaForCausalLM]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `re`
- `typing: Any, Dict, Iterable, List, Optional, Tuple, Union`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.batch_overlap.single_batch_overlap: SboFlags`
- `sglang.srt.batch_overlap.two_batch_overlap: model_forward_maybe_tbo`
- `sglang.srt.distributed: get_moe_expert_parallel_world_size, get_pp_group, get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size, parallel_state, tensor_model_parallel_all_reduce`
- `sglang.srt.distributed.device_communicators.pynccl_allocator: use_symmetric_memory`
- `sglang.srt.environ: envs`
- `sglang.srt.eplb.expert_distribution: get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location: ModelConfigForExpertLocation`
- `sglang.srt.eplb.expert_location_dispatch: ExpertLocationDispatchInfo`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.communicator: LayerCommunicator, LayerScatterModes, enable_moe_dense_fully_dp`
- `sglang.srt.layers.dp_attention: get_attention_tp_rank, get_attention_tp_size, is_allocation_symmetric, is_dp_attention_enabled`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
