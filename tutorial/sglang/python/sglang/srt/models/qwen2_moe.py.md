# qwen2_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/qwen2_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only Qwen2MoE model compatible with HuggingFace weights. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 19-19: Module docstring
```python
"""Inference-only Qwen2MoE model compatible with HuggingFace weights."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 21-102: Module imports
```python
import logging
from contextlib import nullcontext
from typing import Any, Dict, Iterable, List, Optional, Tuple, Union

import torch
import torch.nn.functional as F
from torch import nn
from transformers import PretrainedConfig

from sglang.srt.batch_overlap.two_batch_overlap import model_forward_maybe_tbo
from sglang.srt.distributed import (
    get_moe_data_parallel_world_size,
    get_moe_expert_parallel_world_size,
    get_pp_group,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.distributed.parallel_state import (
    get_attn_context_model_parallel_world_size,
)
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.eplb.expert_location import ModelConfigForExpertLocation
from sglang.srt.eplb.expert_location_dispatch import ExpertLocationDispatchInfo
from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.communicator import (
    LayerCommunicator,
    LayerScatterModes,
    ScatterMode,
)
from sglang.srt.layers.dp_attention import (
    get_attention_tp_rank,
    get_attention_tp_size,
    is_dp_attention_enabled,
)
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import (
    MergedColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 104-108: Top-level if
```python
if is_npu():
    from sglang.srt.hardware_backend.npu.cmo import (
        shared_expert_on_independent_stream,
        wait_share_stream,
    )
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 110-110: Top-level importfrom
```python
from sglang.srt.environ import envs
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 111-111: Top-level importfrom
```python
from sglang.srt.utils.hf_transformers_utils import get_rope_config
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 113-113: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 115-115: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 116-116: Top-level assign
```python
_is_cpu = is_cpu()
```
**EN:** Defines or updates _is_cpu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cpu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 117-117: Top-level assign
```python
_is_cpu_amx_available = cpu_has_amx_support()
```
**EN:** Defines or updates _is_cpu_amx_available, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cpu_amx_available，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 118-118: Top-level assign
```python
_is_hip = is_hip()
```
**EN:** Defines or updates _is_hip, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_hip，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 119-119: Top-level assign
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** Defines or updates _use_aiter, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _use_aiter，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 122-151: Function `can_fuse_shared_expert`
```python
def can_fuse_shared_expert(
    config: PretrainedConfig,
    quant_config: Optional[QuantizationConfig],
) -> bool:
    """Whether the shared expert may be fused as an extra MoE expert (Qwen3.5 + Aiter).

    Caller must still gate on ``support_shared_expert_fusion`` and ``_use_aiter``.
    """
    if (
        get_global_server_args().disable_shared_experts_fusion is True
        or getattr(config, "shared_expert_intermediate_size", 0) <= 0
        or config.shared_expert_intermediate_size != config.moe_intermediate_size
        or get_moe_a2a_backend().is_deepep()
    ):
        return False

    # If the shared expert is excluded from quantization (stored as FP32 in the
    # checkpoint), fusing it into the quantized MoE weight tensor requires online
    # quantization which is not supported. Disable fusion in this case.
    if quant_config is not None:
        exclude_layers = getattr(quant_config, "exclude_layers", [])
        if any(
            "shared_expert" in layer
            and "shared_expert_gate" not in layer
            and not layer.startswith("mtp.")
            for layer in exclude_layers
        ):
            return False

    return True
```
**EN:** This function implements `can_fuse_shared_expert(config: ..., quant_config: ...)` and Whether the shared expert may be fused as an extra MoE expert (Qwen3.5 + Aiter).
**CN:** 这个函数实现了 `can_fuse_shared_expert(config: ..., quant_config: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 154-154: Class `Qwen2MoeMLP` overview
```python
class Qwen2MoeMLP(nn.Module):
```
**EN:** Defines `Qwen2MoeMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen2MoeMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 155-190: Method `Qwen2MoeMLP.__init__`
```python
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
```
**EN:** This method implements `__init__(hidden_size: ..., intermediate_size: ..., hidden_act: ..., quant_config: ...=..., reduce_results: ...=..., prefix: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., intermediate_size: ..., hidden_act: ..., quant_config: ...=..., reduce_results: ...=..., prefix: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 192-203: Method `Qwen2MoeMLP.forward`
```python
    def forward(
        self,
        x,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
    ):
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(
            x, skip_all_reduce=should_allreduce_fusion or use_reduce_scatter
        )
        return x
```
**EN:** This method implements `forward(x, should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x, should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 206-206: Class `Qwen2MoeSparseMoeBlock` overview
```python
class Qwen2MoeSparseMoeBlock(nn.Module):
```
**EN:** Defines `Qwen2MoeSparseMoeBlock` as a reusable runtime type derived from nn.Module. The class groups 9 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen2MoeSparseMoeBlock`，其继承关系为 nn.Module。这个类组织了 9 个方法，用于实现模型相关行为。

### Lines 207-324: Method `Qwen2MoeSparseMoeBlock.__init__`
```python
    def __init__(
        self,
        layer_id: int,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        alt_stream: Optional[torch.cuda.Stream] = None,
        prefix: str = "",
        is_nextn: bool = False,
        support_shared_expert_fusion: bool = False,
    ):
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.layer_id = layer_id
        self.alt_stream = alt_stream
        if self.tp_size > config.num_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {config.num_experts}."
            )
        self.num_experts = config.num_experts
        self.num_shared_experts = 0
        self.num_fused_shared_experts = 0
        if hasattr(config, "n_shared_experts"):
            # config defines the number of shared experts
            self.num_shared_experts = config.n_shared_experts
        elif (
            hasattr(config, "shared_expert_intermediate_size")
            and config.shared_expert_intermediate_size > 0
        ):
            # n_shared_experts is not defined, but shared_expert_intermediate_size is defined, so we use 1 as the number of shared experts
            self.num_shared_experts = 1

        self.enable_shared_expert_fusion = False  # default to False
        if _use_aiter:
            # enable shared expert fusion when use aiter
            self.enable_shared_expert_fusion = (
                support_shared_expert_fusion
                and can_fuse_shared_expert(config, quant_config)
            )
        if self.enable_shared_expert_fusion:
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(layer_id: ..., config: ..., quant_config: ...=..., alt_stream: ...=..., prefix: ...=..., is_nextn: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(layer_id: ..., config: ..., quant_config: ...=..., alt_stream: ...=..., prefix: ...=..., is_nextn: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 326-334: Method `Qwen2MoeSparseMoeBlock.get_moe_weights`
```python
    def get_moe_weights(self):
        return [
            x.data
            for name, x in self.experts.named_parameters()
            if name not in ["correction_bias"]
            and filter_moe_weight_param_global_expert(
                name, x, self.experts.num_local_experts
            )
        ]
```
**EN:** This method implements `get_moe_weights()` and handles weight mapping, filtering, or loading for this model component.
**CN:** 这个方法实现了 `get_moe_weights()`，其作用是处理该模型组件的权重映射、筛选或加载逻辑。

### Lines 336-342: Method `Qwen2MoeSparseMoeBlock._get_shared_expert_weights`
```python
    def _get_shared_expert_weights(self, hidden_states: torch.Tensor) -> torch.Tensor:
        """Return sigmoid(shared_expert_gate) for fused shared expert weights."""
        if not self.enable_shared_expert_fusion or self.shared_expert_gate is None:
            return None
        shared_out = self.shared_expert_gate(hidden_states)
        shared_logits = shared_out[0] if isinstance(shared_out, tuple) else shared_out
        return F.sigmoid(shared_logits)
```
**EN:** This method implements `_get_shared_expert_weights(hidden_states: ...)` and Return sigmoid(shared_expert_gate) for fused shared expert weights.
**CN:** 这个方法实现了 `_get_shared_expert_weights(hidden_states: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 344-371: Method `Qwen2MoeSparseMoeBlock._append_shared_to_topk_output`
```python
    def _append_shared_to_topk_output(
        self,
        topk_output: StandardTopKOutput,
        hidden_states: torch.Tensor,
    ) -> StandardTopKOutput:
        """Append shared expert ids and weights to topk output before fused MoE."""
        if not self.enable_shared_expert_fusion:
            return topk_output
        shared_weights = self._get_shared_expert_weights(hidden_states)
        if shared_weights is None:
            return topk_output

        from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe_triton_kernels import (
            fused_append_shared_experts_with_weights,
        )

        fused_topk_ids, fused_topk_weights = fused_append_shared_experts_with_weights(
            topk_output.topk_ids,
            topk_output.topk_weights,
            shared_weights,
            self.num_fused_shared_experts,
            N=self.num_experts,
        )
        return StandardTopKOutput(
            topk_weights=fused_topk_weights,
            topk_ids=fused_topk_ids,
            router_logits=topk_output.router_logits,
        )
```
**EN:** This method implements `_append_shared_to_topk_output(topk_output: ..., hidden_states: ...)` and Append shared expert ids and weights to topk output before fused MoE.
**CN:** 这个方法实现了 `_append_shared_to_topk_output(topk_output: ..., hidden_states: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 373-392: Method `Qwen2MoeSparseMoeBlock._forward_shared_experts`
```python
    def _forward_shared_experts(self, hidden_states: torch.Tensor):
        shared_output = None
        if self.shared_expert is not None:
            shared_output = self.shared_expert(hidden_states)
            if self.shared_expert_gate is not None:
                if use_intel_amx_backend(self.shared_expert_gate):
                    shared_output = torch.ops.sgl_kernel.fused_linear_sigmoid_mul(
                        hidden_states,
                        self.shared_expert_gate.weight,
                        self.shared_expert_gate.bias,
                        True,
                        shared_output,
                    )
                else:
                    shared_output = (
                        F.sigmoid(self.shared_expert_gate(hidden_states))
                        * shared_output
                    )

        return shared_output
```
**EN:** This method implements `_forward_shared_experts(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_shared_experts(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 394-434: Method `Qwen2MoeSparseMoeBlock._forward_deepep`
```python
    def _forward_deepep(self, hidden_states: torch.Tensor, forward_batch: ForwardBatch):
        enable_dual_stream = (
            is_npu()
            and envs.SGLANG_NPU_USE_MULTI_STREAM.get()
            and forward_batch.forward_mode.is_cuda_graph()
        )
        shared_output = None
        if hidden_states.shape[0] > 0:
            # router_logits: (num_tokens, n_experts)
            router_logits, _ = self.gate(hidden_states)
            if enable_dual_stream:
                shared_output = shared_expert_on_independent_stream(
                    hidden_states.clone(), self._forward_shared_experts
                )
            else:
                shared_output = self._forward_shared_experts(hidden_states)
            topk_output = self.topk(
                hidden_states,
                router_logits,
                num_token_non_padded=forward_batch.num_token_non_padded,
                expert_location_dispatch_info=(
                    ExpertLocationDispatchInfo.init_new(
                        layer_id=self.layer_id,
                    )
                    if not self.is_nextn
                    else None
                ),
            )
        else:
            topk_output = self.topk.empty_topk_output(hidden_states.device)
        final_hidden_states = self.experts(
            hidden_states=hidden_states,
            topk_output=topk_output,
        )
        if enable_dual_stream:
            wait_share_stream()

        if shared_output is not None:
            final_hidden_states.add_(shared_output)

# ... truncated for brevity ...
```
**EN:** This method implements `_forward_deepep(hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_deepep(hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 436-444: Method `Qwen2MoeSparseMoeBlock._forward_router_experts`
```python
    def _forward_router_experts(self, hidden_states: torch.Tensor):
        # router_logits: (num_tokens, n_experts)
        router_logits, _ = self.gate(hidden_states)
        topk_output = self.topk(hidden_states, router_logits)
        if self.enable_shared_expert_fusion and TopKOutputChecker.format_is_standard(
            topk_output
        ):
            topk_output = self._append_shared_to_topk_output(topk_output, hidden_states)
        return self.experts(hidden_states, topk_output)
```
**EN:** This method implements `_forward_router_experts(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_router_experts(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 446-459: Method `Qwen2MoeSparseMoeBlock.forward_normal_dual_stream`
```python
    def forward_normal_dual_stream(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        current_stream = torch.cuda.current_stream()
        self.alt_stream.wait_stream(current_stream)
        shared_output = self._forward_shared_experts(hidden_states.clone())

        with torch.cuda.stream(self.alt_stream):
            router_output = self._forward_router_experts(hidden_states)

        current_stream.wait_stream(self.alt_stream)

        return router_output, shared_output
```
**EN:** This method implements `forward_normal_dual_stream(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_normal_dual_stream(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 461-499: Method `Qwen2MoeSparseMoeBlock.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        forward_batch: Optional[ForwardBatch] = None,
        use_reduce_scatter: bool = False,
        should_allreduce_fusion: bool = False,
    ) -> torch.Tensor:
        num_tokens, hidden_dim = hidden_states.shape
        hidden_states = hidden_states.view(-1, hidden_dim)

        if get_moe_a2a_backend().is_deepep():
            return self._forward_deepep(hidden_states, forward_batch)

        if (
            self.alt_stream is not None
            and hidden_states.shape[0] > 0
            and get_is_capture_mode()
        ):
            final_hidden_states, shared_output = self.forward_normal_dual_stream(
                hidden_states
            )
        else:
            shared_output = self._forward_shared_experts(hidden_states)
            final_hidden_states = self._forward_router_experts(hidden_states)

        if shared_output is not None:
            # In-place add is required to keep final_hidden_states in the
            # symmetric memory pool (when --enable-symm-mem is used).
            # An out-of-place add would allocate a new tensor outside symm
            # memory, breaking subsequent symmetric collective operations.
            final_hidden_states += shared_output
        if self.tp_size > 1 and not should_skip_post_experts_all_reduce(
            is_tp_path=True,
            use_reduce_scatter=use_reduce_scatter,
            should_allreduce_fusion=should_allreduce_fusion,
        ):
            final_hidden_states = tensor_model_parallel_all_reduce(final_hidden_states)

        return final_hidden_states.view(num_tokens, hidden_dim)
```
**EN:** This method implements `forward(hidden_states: ..., forward_batch: ...=..., use_reduce_scatter: ...=..., should_allreduce_fusion: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., forward_batch: ...=..., use_reduce_scatter: ...=..., should_allreduce_fusion: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 502-502: Class `Qwen2MoeAttention` overview
```python
class Qwen2MoeAttention(nn.Module):
```
**EN:** Defines `Qwen2MoeAttention` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen2MoeAttention`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 503-582: Method `Qwen2MoeAttention.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        rope_theta: float = 10000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        max_position_embeddings: int = 8192,
        qkv_bias: int = True,
        quant_config: Optional[QuantizationConfig] = None,
        dual_chunk_attention_config: Optional[dict[str, Any]] = None,
        prefix: str = "",
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
        self.head_dim = hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.rope_theta = rope_theta
        self.max_position_embeddings = max_position_embeddings

# ... truncated for brevity ...
```
**EN:** This method implements `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 584-595: Method `Qwen2MoeAttention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v, forward_batch)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 598-598: Class `Qwen2MoeDecoderLayer` overview
```python
class Qwen2MoeDecoderLayer(nn.Module):
```
**EN:** Defines `Qwen2MoeDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen2MoeDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 599-673: Method `Qwen2MoeDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        rope_theta, rope_scaling = get_rope_config(config)
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        qkv_bias = getattr(config, "qkv_bias", True)
        dual_chunk_attention_config = getattr(
            config, "dual_chunk_attention_config", None
        )
        self.self_attn = Qwen2MoeAttention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            max_position_embeddings=max_position_embeddings,
            quant_config=quant_config,
            dual_chunk_attention_config=dual_chunk_attention_config,
            qkv_bias=qkv_bias,
            prefix=add_prefix("self_attn", prefix),
        )

        self.layer_id = layer_id

        self.attn_tp_size = get_attention_tp_size()
        self.attn_tp_rank = get_attention_tp_rank()

        # Qwen2MoE all layers are sparse and have no nextn now
        self.is_layer_sparse = True
        is_previous_layer_sparse = True
        is_next_layer_sparse = True
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 675-717: Method `Qwen2MoeDecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
        captured_last_layer_outputs: Optional[List[torch.Tensor]] = None,
        **kwargs,
    ) -> Tuple[torch.Tensor, torch.Tensor]:

        hidden_states, residual = (
            self.layer_communicator.prepare_attn_and_capture_last_layer_outputs(
                hidden_states,
                residual,
                forward_batch,
                captured_last_layer_outputs=captured_last_layer_outputs,
                **kwargs,
            )
        )

        if hidden_states.shape[0] != 0:
            hidden_states = self.self_attn(
                positions=positions,
                hidden_states=hidden_states,
                forward_batch=forward_batch,
            )

        hidden_states, residual = self.layer_communicator.prepare_mlp(
            hidden_states, residual, forward_batch
        )

        # For DP with padding, reduce scatter can be used instead of all-reduce.
        use_reduce_scatter = self.layer_communicator.should_use_reduce_scatter(
            forward_batch
        )

        hidden_states = self.mlp(hidden_states, forward_batch, use_reduce_scatter)

        hidden_states, residual = self.layer_communicator.postprocess_layer(
            hidden_states, residual, forward_batch
# ... truncated for brevity ...
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., captured_last_layer_outputs: ...=..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., captured_last_layer_outputs: ...=..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 720-720: Class `Qwen2MoeModel` overview
```python
class Qwen2MoeModel(nn.Module):
```
**EN:** Defines `Qwen2MoeModel` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen2MoeModel`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 721-769: Method `Qwen2MoeModel.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        decoder_layer_type: type[nn.Module] = Qwen2MoeDecoderLayer,
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.config = config
        self.vocab_size = config.vocab_size
        self.pp_group = get_pp_group()

        self.moe_dp_size = get_moe_data_parallel_world_size()
        self.attn_cp_size = get_attn_context_model_parallel_world_size()

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                use_attn_tp_group=is_dp_attention_enabled(),
                quant_config=quant_config,
                prefix=add_prefix("embed_tokens", prefix),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        # Use the provided decoder layer type or default to Qwen2MoeDecoderLayer
        decoder_layer_type = decoder_layer_type or Qwen2MoeDecoderLayer
        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: decoder_layer_type(
                layer_id=idx,
                config=config,
                quant_config=quant_config,
                prefix=prefix,
                alt_stream=alt_stream,
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., decoder_layer_type: ...=..., alt_stream: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., decoder_layer_type: ...=..., alt_stream: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 771-774: Method `Qwen2MoeModel.set_eagle3_layers_to_capture`
```python
    def set_eagle3_layers_to_capture(self, layers_to_capture: List[int]):
        self.layers_to_capture = layers_to_capture
        for layer_id in self.layers_to_capture:
            setattr(self.layers[layer_id], "_is_layer_to_capture", True)
```
**EN:** This method implements `set_eagle3_layers_to_capture(layers_to_capture: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen2MoeModel`.
**CN:** 这个方法实现了 `set_eagle3_layers_to_capture(layers_to_capture: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen2MoeModel` 内部调用。

### Lines 776-866: Method `Qwen2MoeModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> Union[torch.Tensor, PPProxyTensors]:
        if self.pp_group.is_first_rank:
            if input_embeds is None:
                hidden_states = self.embed_tokens(input_ids)
            else:
                hidden_states = input_embeds
            residual = None
        else:
            assert pp_proxy_tensors is not None
            hidden_states = pp_proxy_tensors["hidden_states"]
            residual = pp_proxy_tensors["residual"]

        if (
            is_prefill_context_parallel_enabled()
            and forward_batch.forward_mode.is_context_parallel_extend()
            and forward_batch.attn_cp_metadata is not None
        ):
            if self.pp_group.is_first_rank:
                hidden_states = cp_split_and_rebuild_data(forward_batch, hidden_states)
            positions = cp_split_and_rebuild_position(forward_batch, positions)

        aux_hidden_states = []
        if forward_batch.can_run_tbo:
            hidden_states, residual = model_forward_maybe_tbo(
                layers=self.layers,
                enable_tbo=True,
                input_data_scatter_mode=ScatterMode.model_input_output(),
                positions=positions,
                forward_batch=forward_batch,
                hidden_states=hidden_states,
                residual=residual,
            )
        else:
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 869-869: Class `Qwen2MoeForCausalLM` overview
```python
class Qwen2MoeForCausalLM(nn.Module):
```
**EN:** Defines `Qwen2MoeForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 8 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen2MoeForCausalLM`，其继承关系为 nn.Module。这个类组织了 8 个方法，用于实现模型相关行为。

### Lines 870-870: Class `Qwen2MoeForCausalLM` attributes
```python
    fall_back_to_pt_during_load = False
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen2MoeForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen2MoeForCausalLM` 在运行时的行为。

### Lines 872-898: Method `Qwen2MoeForCausalLM.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        alt_stream = torch.cuda.Stream() if _is_cuda else None
        self.model = Qwen2MoeModel(
            config,
            quant_config,
            prefix=add_prefix("model", prefix),
            alt_stream=alt_stream,
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
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 900-924: Method `Qwen2MoeForCausalLM.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> torch.Tensor:
        hidden_states = self.model(
            input_ids,
            positions,
            forward_batch,
            input_embeds,
            pp_proxy_tensors=pp_proxy_tensors,
        )
        aux_hidden_states = None
        if self.capture_aux_hidden_states:
            hidden_states, aux_hidden_states = hidden_states
        if self.pp_group.is_last_rank:
            return self.logits_processor(
                input_ids, hidden_states, self.lm_head, forward_batch, aux_hidden_states
            )
        else:
            return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 926-967: Method `Qwen2MoeForCausalLM.forward_split_prefill`
```python
    @torch.no_grad()
    def forward_split_prefill(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        split_interval: Tuple[int, int],  # [start, end) 0-based
        input_embeds: torch.Tensor = None,
    ):
        start, end = split_interval
        # embed
        if start == 0:
            if input_embeds is None:
                forward_batch.hidden_states = self.model.embed_tokens(input_ids)
            else:
                forward_batch.hidden_states = input_embeds

        # decoder layer
        for i in range(start, end):
            with get_global_expert_distribution_recorder().with_current_layer(i):
                layer = self.model.layers[i]
                forward_batch.hidden_states, forward_batch.residual = layer(
                    positions,
                    forward_batch.hidden_states,
                    forward_batch,
                    forward_batch.residual,
                )

        if end == self.model.config.num_hidden_layers:
            # norm
            hidden_states, _ = self.model.norm(
                forward_batch.hidden_states, forward_batch.residual
            )
            forward_batch.hidden_states = hidden_states
            # logits process
            result = self.logits_processor(
                input_ids, forward_batch.hidden_states, self.lm_head, forward_batch
            )
        else:
            result = None
# ... truncated for brevity ...
```
**EN:** This method implements `forward_split_prefill(input_ids: ..., positions: ..., forward_batch: ..., split_interval: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward_split_prefill(input_ids: ..., positions: ..., forward_batch: ..., split_interval: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 969-971: Method `Qwen2MoeForCausalLM.start_layer`
```python
    @property
    def start_layer(self):
        return self.model.start_layer
```
**EN:** This method implements `start_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen2MoeForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `start_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen2MoeForCausalLM` 内部调用 装饰器：property。

### Lines 973-975: Method `Qwen2MoeForCausalLM.end_layer`
```python
    @property
    def end_layer(self):
        return self.model.end_layer
```
**EN:** This method implements `end_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen2MoeForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `end_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen2MoeForCausalLM` 内部调用 装饰器：property。

### Lines 977-1061: Method `Qwen2MoeForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        expert_params_mapping = FusedMoE.make_expert_params_mapping(
            ckpt_gate_proj_name="gate_proj",
            ckpt_down_proj_name="down_proj",
            ckpt_up_proj_name="up_proj",
            num_experts=self.config.num_experts,
        )

        params_dict = dict(self.named_parameters())
        for name, loaded_weight in weights:
            layer_id = get_layer_id(name)
            if (
                layer_id is not None
                and hasattr(self.model, "start_layer")
                and (
                    layer_id < self.model.start_layer
                    or layer_id >= self.model.end_layer
                )
            ):
                continue
            if "rotary_emb.inv_freq" in name:
                continue
            for param_name, weight_name, shard_id in stacked_params_mapping:
                # Skip non-stacked layers and experts (experts handled below).
                if weight_name not in name:
                    continue
                # We have mlp.experts[0].gate_proj in the checkpoint.
                # Since we handle the experts below in expert_params_mapping,
                # we need to skip here BEFORE we update the name, otherwise
                # name will be updated to mlp.experts[0].gate_up_proj, which
                # will then be updated below in expert_params_mapping
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1063-1069: Method `Qwen2MoeForCausalLM.get_model_config_for_expert_location`
```python
    @classmethod
    def get_model_config_for_expert_location(cls, config):
        return ModelConfigForExpertLocation(
            num_layers=config.num_hidden_layers,
            num_logical_experts=config.num_experts,
            num_groups=None,
        )
```
**EN:** This method implements `get_model_config_for_expert_location(config)` and returns a cached or derived object used elsewhere in the pipeline Decorators: classmethod.
**CN:** 这个方法实现了 `get_model_config_for_expert_location(config)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果 装饰器：classmethod。

### Lines 1071-1086: Method `Qwen2MoeForCausalLM.set_eagle3_layers_to_capture`
```python
    def set_eagle3_layers_to_capture(self, layer_ids: Optional[List[int]] = None):
        if not self.pp_group.is_last_rank:
            return

        self.capture_aux_hidden_states = True
        if layer_ids is None:
            num_layers = self.config.num_hidden_layers
            self.model.set_eagle3_layers_to_capture(
                [
                    2,
                    num_layers // 2,
                    num_layers - 3,
                ]
            )  # Specific layers for EAGLE3 support
        else:
            self.model.set_eagle3_layers_to_capture([val + 1 for val in layer_ids])
```
**EN:** This method implements `set_eagle3_layers_to_capture(layer_ids: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen2MoeForCausalLM`.
**CN:** 这个方法实现了 `set_eagle3_layers_to_capture(layer_ids: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen2MoeForCausalLM` 内部调用。

### Lines 1089-1089: Top-level assign
```python
EntryClass = Qwen2MoeForCausalLM
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `logging`
- `contextlib.nullcontext`
- `typing.Any`
- `typing.Dict`
- `typing.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `typing.Union`
- `torch`
- `torch.nn.functional`
- `torch.nn`
- `transformers.PretrainedConfig`
- `sglang.srt.batch_overlap.two_batch_overlap.model_forward_maybe_tbo`
- `sglang.srt.distributed.get_moe_data_parallel_world_size`
- `sglang.srt.distributed.get_moe_expert_parallel_world_size`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.distributed.tensor_model_parallel_all_reduce`
- `sglang.srt.distributed.parallel_state.get_attn_context_model_parallel_world_size`
- `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location.ModelConfigForExpertLocation`
- `sglang.srt.eplb.expert_location_dispatch.ExpertLocationDispatchInfo`
- `sglang.srt.layers.activation.SiluAndMul`
- `sglang.srt.layers.communicator.LayerCommunicator`
- `sglang.srt.layers.communicator.LayerScatterModes`
- `sglang.srt.layers.communicator.ScatterMode`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.MergedColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.moe.get_moe_a2a_backend`
- `sglang.srt.layers.moe.should_skip_post_experts_all_reduce`
- `sglang.srt.layers.moe.ep_moe.layer.get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton.FusedMoE`
- `sglang.srt.layers.moe.topk.StandardTopKOutput`
- `sglang.srt.layers.moe.topk.TopK`
- `sglang.srt.layers.moe.topk.TopKOutputChecker`
- `sglang.srt.layers.moe.utils.RoutingMethodType`
- `sglang.srt.layers.moe.utils.filter_moe_weight_param_global_expert`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.utils.get_layer_id`
- `sglang.srt.layers.utils.cp_utils.cp_all_gather_rerange_output`
- `sglang.srt.layers.utils.cp_utils.cp_split_and_rebuild_data`
- `sglang.srt.layers.utils.cp_utils.cp_split_and_rebuild_position`
- `sglang.srt.layers.utils.cp_utils.is_prefill_context_parallel_enabled`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.cuda_graph_runner.get_is_capture_mode`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.cpu_has_amx_support`
- `sglang.srt.utils.get_bool_env_var`
- `sglang.srt.utils.is_cpu`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_hip`
- `sglang.srt.utils.is_npu`
- `sglang.srt.utils.make_layers`
- `sglang.srt.utils.use_intel_amx_backend`
- `sglang.srt.environ.envs`
- `sglang.srt.utils.hf_transformers_utils.get_rope_config`
- `sglang.srt.hardware_backend.npu.cmo.shared_expert_on_independent_stream`
- `sglang.srt.hardware_backend.npu.cmo.wait_share_stream`
- `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe_triton_kernels.fused_append_shared_experts_with_weights`
