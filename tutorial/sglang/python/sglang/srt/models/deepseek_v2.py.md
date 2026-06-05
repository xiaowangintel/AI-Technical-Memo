# deepseek_v2.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/deepseek_v2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Deepseek v2 architecture into SGLang's serving runtime. Inference-only DeepseekV2 model. / 该模块将 Deepseek v2 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only DeepseekV2 model。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18: Module header and imports / 模块头与导入
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
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

# Adapted from:
# https://github.com/vllm-project/vllm/blob/fb6af8bc086328ca6659e72d11ffd4309ce4de22/vllm/model_executor/models/deepseek_v2.py
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 177-178: Conditional setup: _use_aiter / 条件初始化：_use_aiter
```python
if _use_aiter:
    from sglang.srt.layers.rocm_linear_utils import aiter_dsv3_router_gemm
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 180-183: Conditional setup: _use_aiter_gfx95 / 条件初始化：_use_aiter_gfx95
```python
if _use_aiter_gfx95:
    from sglang.srt.layers.rocm_linear_utils import (
        get_dsv3_gemm_output_zero_allocator_size,
    )
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 185-186: Conditional setup: _use_aiter / 条件初始化：_use_aiter
```python
if _use_aiter:
    pass
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 188-203: Conditional setup: _is_cuda / 条件初始化：_is_cuda
```python
if _is_cuda:
    from flashinfer.gemm import mm_M1_16_K7168_N256 as _raw_dsv3_router_gemm
    from sgl_kernel import dsv3_fused_a_gemm, dsv3_router_gemm
elif _is_npu:
    from sglang.srt.hardware_backend.npu.modules.deepseek_v2_attention_mla_npu import (
        forward_dsa_core_npu,
        forward_dsa_prepare_npu,
        forward_mha_core_npu,
        forward_mha_prepare_npu,
        forward_mla_core_npu,
        forward_mla_prepare_npu,
    )
elif _is_musa:
    from sgl_kernel import dsv3_fused_a_gemm, dsv3_router_gemm
else:
    pass
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 205-205: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 208-330: Class: DeepseekV2MLP / 类：DeepseekV2MLP
```python
class DeepseekV2MLP(nn.Module):
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
        swiglu_limit: Optional[float] = None,
    ) -> None:
        super().__init__()
        self.tp_size = tp_size
        self.swiglu_limit = swiglu_limit

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
        if not hasattr(self.gate_up_proj, "weight") and hasattr(
            self.gate_up_proj, "weight_packed"
        ):
            self.gate_up_proj.weight = self.gate_up_proj.weight_packed
# ... truncated for brevity ...
```
**EN:** This class defines Deepseek V2 M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek V2 M L P，用于封装该模型组件的状态与方法。

### Lines 333-430: Class: MoEGate / 类：MoEGate
```python
class MoEGate(nn.Module):
    def __init__(
        self,
        config,
        quant_config,
        prefix: str = "",
        is_nextn: bool = False,
        is_hash_moe: bool = False,
        is_deepseek_v4: bool = False,
    ):
        super().__init__()
        self.is_nextn = is_nextn
        self.is_deepseek_v4 = is_deepseek_v4
        self.weight = nn.Parameter(
            torch.empty((config.n_routed_experts, config.hidden_size))
        )

        if config.topk_method == "noaux_tc" and not is_hash_moe:
            correction_bias_dtype = torch.float32
            if quant_config is not None:
                if (
                    quant_config.get_name() == "modelopt_fp4"
                    and get_moe_runner_backend().is_flashinfer_trtllm()
                ):
                    correction_bias_dtype = torch.bfloat16
                elif _use_aiter and quant_config.get_name() in (
                    "fp8",
                    "compressed_tensors",
                    "quark",
                ):
                    correction_bias_dtype = torch.bfloat16
            self.e_score_correction_bias = nn.Parameter(
                torch.empty((config.n_routed_experts), dtype=correction_bias_dtype)
            )
        else:
            self.e_score_correction_bias = None
        if _is_cpu and _is_cpu_amx_available:
            self.quant_method = PackWeightMethod(weight_names=["weight"])
        self.nsa_enable_prefill_cp = is_nsa_enable_prefill_cp()

# ... truncated for brevity ...
```
**EN:** This class defines Mo E Gate inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Mo E Gate，用于封装该模型组件的状态与方法。

### Lines 433-1306: Class: DeepseekV2MoE / 类：DeepseekV2MoE
```python
class DeepseekV2MoE(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
        is_nextn: bool = False,
        is_deepseek_v4: bool = False,
    ):
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.moe_ep_size = get_moe_expert_parallel_world_size()
        self.routed_scaling_factor = config.routed_scaling_factor
        self.n_shared_experts = config.n_shared_experts

        n_shared_experts = (
            0 if config.n_shared_experts is None else int(config.n_shared_experts)
        )
        _fusion_disabled = get_global_server_args().disable_shared_experts_fusion

        # num_fused_shared_experts drives weight remapping in deepseek_weight_loader:
        # mlp.shared_experts → mlp.experts.256 when > 0.
        self.num_fused_shared_experts = 0 if _fusion_disabled else n_shared_experts

        # DeepEP shared expert fusion: shared expert is fused into the same MoE kernel
        # as a local expert at the home EP rank. Expert layout is expanded from 256
        # routed to 256+EP_size (e.g. 272 for EP=16). TopK handles interleaving.
        _is_deepep_fusion = (
            is_deepep_class_backend() and self.num_fused_shared_experts > 0
        )

        if _is_deepep_fusion:
            # 256 routed + EP_size shared slots = 272 experts total (for EP=16)
            num_experts_for_moe = config.n_routed_experts + self.moe_ep_size
            top_k_for_moe = config.num_experts_per_tok + 1  # 8 routed + 1 shared
            # Interleaving for DeepEP dispatch is handled by TopK internally.
        else:
# ... truncated for brevity ...
```
**EN:** This class defines Deepseek V2 Mo E inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek V2 Mo E，用于封装该模型组件的状态与方法。

### Lines 1309-1776: Class: DeepseekV2AttentionMLA / 类：DeepseekV2AttentionMLA
```python
class DeepseekV2AttentionMLA(
    nn.Module,
    DeepseekMHAForwardMixin,
    DeepseekMLAForwardMixin,
    DeepseekMLARocmForwardMixin,
    DeepseekMLACpuForwardMixin,
):

    def __init__(
        self,
        config: PretrainedConfig,
        hidden_size: int,
        num_heads: int,
        qk_nope_head_dim: int,
        qk_rope_head_dim: int,
        v_head_dim: int,
        q_lora_rank: int,
        kv_lora_rank: int,
        rope_theta: float = 10000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        max_position_embeddings: int = 8192,
        quant_config: Optional[QuantizationConfig] = None,
        reduce_results: bool = True,
        layer_id: int = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
        skip_rope: bool = False,
        is_nextn: bool = False,
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.hidden_size = hidden_size
        self.qk_nope_head_dim = qk_nope_head_dim
        self.qk_rope_head_dim = qk_rope_head_dim
        self.qk_head_dim = qk_nope_head_dim + qk_rope_head_dim
        self.v_head_dim = v_head_dim
        self.q_lora_rank = q_lora_rank
        self.kv_lora_rank = kv_lora_rank
        self.quant_config = quant_config
        attn_tp_rank = get_attention_tp_rank()
# ... truncated for brevity ...
```
**EN:** This class defines Deepseek V2 Attention M L A inheriting from nn.Module, DeepseekMHAForwardMixin, DeepseekMLAForwardMixin, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek V2 Attention M L A，用于封装该模型组件的状态与方法。

### Lines 1779-2074: Class: DeepseekV2DecoderLayer / 类：DeepseekV2DecoderLayer
```python
class DeepseekV2DecoderLayer(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        moe_quant_config_override: Optional[QuantizationConfig] = None,
        is_nextn: bool = False,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.config = config
        if hasattr(config, "rope_parameters"):
            rope_theta = config.rope_parameters["rope_theta"]
            assert rope_theta is not None, f"rope_theta not found in config: {config}"
            rope_type = config.rope_parameters.get("rope_type")
            rope_scaling = config.rope_parameters if rope_type != "default" else None
        else:
            rope_theta = config.rope_theta
            rope_scaling = config.rope_scaling
        max_position_embeddings = config.max_position_embeddings
        self.speculative_algorithm = SpeculativeAlgorithm.from_string(
            get_global_server_args().speculative_algorithm
        )
        self.nsa_enable_prefill_cp = is_nsa_enable_prefill_cp()
        self.layer_id = layer_id
        self.is_nextn = is_nextn
        self.self_attn = DeepseekV2AttentionMLA(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            qk_nope_head_dim=config.qk_nope_head_dim,
            qk_rope_head_dim=config.qk_rope_head_dim,
            v_head_dim=config.v_head_dim,
            q_lora_rank=(
                config.q_lora_rank if hasattr(config, "q_lora_rank") else None
            ),
# ... truncated for brevity ...
```
**EN:** This class defines Deepseek V2 Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek V2 Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 2077-2346: Class: DeepseekV2Model / 类：DeepseekV2Model
```python
class DeepseekV2Model(nn.Module):
    fall_back_to_pt_during_load = False

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.padding_id = config.pad_token_id
        self.vocab_size = config.vocab_size
        self.first_k_dense_replace = config.first_k_dense_replace
        self.pp_group = get_pp_group()
        self.nsa_enable_prefill_cp = is_nsa_enable_prefill_cp()
        if self.nsa_enable_prefill_cp:
            self.cp_size = get_attention_cp_size()
        else:
            self.cp_size = None

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                use_attn_tp_group=is_dp_attention_enabled(),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        self.alt_stream = (
            torch.cuda.Stream()
            if (
                _is_cuda
                or _is_musa
                or envs.SGLANG_NPU_USE_MULTI_STREAM.get()
                or envs.SGLANG_ROCM_USE_MULTI_STREAM.get()
            )
            else None
        )

# ... truncated for brevity ...
```
**EN:** This class defines Deepseek V2 Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek V2 Model，用于封装该模型组件的状态与方法。

### Lines 2349-2576: Class: DeepseekV2ForCausalLM / 类：DeepseekV2ForCausalLM
```python
class DeepseekV2ForCausalLM(nn.Module, DeepseekV2WeightLoaderMixin):
    # for quark model load
    packed_modules_mapping = {}

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        # for quark model load
        # Fuse q_a_proj and kv_a_proj_with_mqa along output dimension when q_lora_rank is not None
        self.fuse_qkv_a_proj = (
            hasattr(config, "q_lora_rank") and config.q_lora_rank is not None
        )
        if self.fuse_qkv_a_proj:
            self.packed_modules_mapping["fused_qkv_a_proj_with_mqa"] = [
                "q_a_proj",
                "kv_a_proj_with_mqa",
            ]

        # Quant configs like Quark may rely on the model to provide fused-module
        # mappings so exclusion checks can unfuse derived names back to the
        # checkpoint's source layer names.
        if quant_config is not None and hasattr(quant_config, "packed_modules_mapping"):
            quant_config.packed_modules_mapping = self.packed_modules_mapping

        self.pp_group = get_pp_group()
        self.config = config
        self.tp_size = get_tensor_model_parallel_world_size()
        self.quant_config = quant_config
        self.determine_num_fused_shared_experts()
        self.use_nsa = is_deepseek_nsa(config)
        self.model = DeepseekV2Model(
            config, quant_config, prefix=add_prefix("model", prefix)
        )

        if self.pp_group.is_last_rank:
# ... truncated for brevity ...
```
**EN:** This class defines Deepseek V2 For Causal L M inheriting from nn.Module, DeepseekV2WeightLoaderMixin, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek V2 For Causal L M，用于封装该模型组件的状态与方法。

### Lines 2579-2580: Class: DeepseekV3ForCausalLM / 类：DeepseekV3ForCausalLM
```python
class DeepseekV3ForCausalLM(DeepseekV2ForCausalLM):
    pass
```
**EN:** This class defines Deepseek V3 For Causal L M inheriting from DeepseekV2ForCausalLM, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek V3 For Causal L M，用于封装该模型组件的状态与方法。

### Lines 2583-2584: Class: DeepseekV32ForCausalLM / 类：DeepseekV32ForCausalLM
```python
class DeepseekV32ForCausalLM(DeepseekV2ForCausalLM):
    pass
```
**EN:** This class defines Deepseek V32 For Causal L M inheriting from DeepseekV2ForCausalLM, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek V32 For Causal L M，用于封装该模型组件的状态与方法。

### Lines 2592-2602: Function: flashinfer_dsv3_router_gemm() / 函数：flashinfer_dsv3_router_gemm()
```python
def flashinfer_dsv3_router_gemm(
    logits: torch.Tensor,
    hidden_states: torch.Tensor,
    weight: torch.Tensor,
) -> None:
    _raw_dsv3_router_gemm(
        hidden_states,
        weight.t(),
        logits,
        launch_with_pdl=True,
    )
```
**EN:** This function implements flashinfer dsv3 router gemm for the surrounding model/runtime logic. Key parameters include logits, hidden_states, weight.
**CN:** 该函数实现了 flashinfer dsv3 router gemm 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 2605-2605: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [DeepseekV2ForCausalLM, DeepseekV3ForCausalLM, DeepseekV32ForCausalLM]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `__future__: annotations`
- `logging`
- `contextlib: nullcontext`
- `typing: Any, Dict, Iterable, List, Optional, Tuple, Union`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.jit_kernel.deepseek_v4: silu_and_mul_clamp, silu_and_mul_contig_post_quant`
- `sglang.srt.batch_overlap.single_batch_overlap: SboFlags, compute_overlap_args`
- `sglang.srt.batch_overlap.two_batch_overlap: MaybeTboDeepEPDispatcher, model_forward_maybe_tbo`
- `sglang.srt.configs.model_config: compute_mla_mscale_scaling, get_nsa_index_head_dim, get_nsa_index_n_heads, get_nsa_index_topk, is_deepseek_nsa`
- `sglang.srt.distributed: divide, get_moe_expert_parallel_world_size, get_pp_group, get_tensor_model_parallel_world_size, tensor_model_parallel_all_reduce`
- `sglang.srt.environ: envs`
- `sglang.srt.eplb.expert_distribution: get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location: ModelConfigForExpertLocation`
- `sglang.srt.eplb.expert_location_dispatch: ExpertLocationDispatchInfo`
- `sglang.srt.layers: deep_gemm_wrapper`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.amx_utils: PackWeightMethod`
