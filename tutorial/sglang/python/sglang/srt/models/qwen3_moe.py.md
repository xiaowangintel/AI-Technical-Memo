# qwen3_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/qwen3_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only Qwen3MoE model compatible with HuggingFace weights. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 18-18: Module docstring
```python
"""Inference-only Qwen3MoE model compatible with HuggingFace weights."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 20-90: Module imports
```python
import logging
import math
from typing import Any, Dict, Iterable, List, Optional, Tuple, TypeVar

import torch
from torch import nn
from transformers import PretrainedConfig

from sglang.srt.distributed import (
    get_attn_context_model_parallel_rank,
    get_attn_context_model_parallel_world_size,
    get_moe_data_parallel_world_size,
    get_moe_expert_parallel_world_size,
    get_moe_tensor_parallel_world_size,
    get_pp_group,
    get_tensor_model_parallel_rank,
    moe_expert_parallel_all_reduce,
    moe_tensor_model_parallel_all_reduce,
)
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.eplb.expert_location import ModelConfigForExpertLocation
from sglang.srt.eplb.expert_location_dispatch import ExpertLocationDispatchInfo
from sglang.srt.layers.communicator import LayerCommunicator, LayerScatterModes
from sglang.srt.layers.dp_attention import get_attention_tp_rank, get_attention_tp_size
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import (
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.moe import (
    get_moe_a2a_backend,
    should_skip_post_experts_all_reduce,
)
from sglang.srt.layers.moe.ep_moe.layer import get_moe_impl_class
from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE
from sglang.srt.layers.moe.topk import TopK
from sglang.srt.layers.moe.utils import (
    RoutingMethodType,
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 92-92: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 94-98: Top-level if
```python
if _is_cuda:
    from sglang.jit_kernel.fused_qknorm_rope import (
        can_use_fused_qk_norm_rope,
        fused_qk_norm_rope,
    )
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 100-100: Top-level assign
```python
TConfig = TypeVar("TConfig", bound=PretrainedConfig)
```
**EN:** Defines or updates TConfig, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 TConfig，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 102-102: Top-level assign
```python
Qwen3MoeConfig = None
```
**EN:** Defines or updates Qwen3MoeConfig, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 Qwen3MoeConfig，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 104-104: Top-level assign
```python
_is_flashinfer_available = is_flashinfer_available()
```
**EN:** Defines or updates _is_flashinfer_available, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_flashinfer_available，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 106-106: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 107-107: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 108-108: Top-level assign
```python
_is_npu = is_npu()
```
**EN:** Defines or updates _is_npu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_npu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 110-111: Top-level if
```python
if _is_npu:
    from sgl_kernel_npu.norm.split_qkv_rmsnorm_rope import split_qkv_rmsnorm_rope
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 114-230: Function `compute_yarn_parameters`
```python
def compute_yarn_parameters(
    config: PretrainedConfig,
) -> tuple[float, float, float, float]:
    """
    Refer to https://github.com/huggingface/transformers/blob/main/src/transformers/modeling_rope_utils.py#L197C1-L288C1
    Computes the inverse frequencies with NTK scaling. Please refer to the
    [original paper](https://huggingface.co/papers/2309.00071)
    Args:
        config ([`~transformers.PretrainedConfig`]):
            The model configuration.
    Returns:
        factor: float, the scaling factor for the RoPE embeddings
        low: float, the lower bound of the dimension range
        high: float, the upper bound of the dimension range
        attention_factor: float, the post-processing scaling factor applied to the computed cos/sin
    """

    # The config does not contain rope_scaling, which means the model is not using yarn.
    # In transformers v5, rope_parameters is never None (even for default rope), so also
    # check rope_type to distinguish actual yarn configs from plain rotary embeddings.
    rope_scaling = getattr(config, "rope_parameters", None)
    if rope_scaling is None:
        rope_scaling = getattr(config, "rope_scaling", None)
    if rope_scaling is None:
        return 1.0, 0, 0, 1.0
    rope_type = rope_scaling.get("rope_type") or rope_scaling.get("type") or "default"
    if rope_type == "default":
        return 1.0, 0, 0, 1.0

    base = rope_scaling.get("rope_theta") or getattr(config, "rope_theta", 10000)
    partial_rotary_factor = (
        config.partial_rotary_factor
        if hasattr(config, "partial_rotary_factor")
        else 1.0
    )
    head_dim = getattr(
        config, "head_dim", config.hidden_size // config.num_attention_heads
    )
    dim = int(head_dim * partial_rotary_factor)
    factor = rope_scaling.get("factor", 1.0)
# ... truncated for brevity ...
```
**EN:** This function implements `compute_yarn_parameters(config: ...)` and Refer to https://github.com/huggingface/transformers/blob/main/src/transformers/modeling_rope_utils.py#L197C1-L288C1.
**CN:** 这个函数实现了 `compute_yarn_parameters(config: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 233-233: Class `Qwen3MoeSparseMoeBlock` overview
```python
class Qwen3MoeSparseMoeBlock(nn.Module):
```
**EN:** Defines `Qwen3MoeSparseMoeBlock` as a reusable runtime type derived from nn.Module. The class groups 13 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3MoeSparseMoeBlock`，其继承关系为 nn.Module。这个类组织了 13 个方法，用于实现模型相关行为。

### Lines 234-290: Method `Qwen3MoeSparseMoeBlock.__init__`
```python
    def __init__(
        self,
        layer_id: int,
        config: Qwen3MoeConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.tp_size = get_moe_tensor_parallel_world_size()
        self.ep_size = get_moe_expert_parallel_world_size()
        self.layer_id = layer_id
        if self.tp_size > config.num_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {config.num_experts}."
            )

        from sglang.srt.layers.quantization.gguf import GGUFConfig

        norm_topk_prob = getattr(config, "norm_topk_prob", True)
        if isinstance(quant_config, GGUFConfig):
            norm_topk_prob = False

        self.topk = TopK(
            top_k=config.num_experts_per_tok,
            renormalize=norm_topk_prob,
            use_grouped_topk=False,
            layer_id=layer_id,
        )

        self.experts = get_moe_impl_class(quant_config)(
            num_experts=config.num_experts
            + get_global_server_args().ep_num_redundant_experts,
            top_k=config.num_experts_per_tok,
            layer_id=layer_id,
            hidden_size=config.hidden_size,
            intermediate_size=config.moe_intermediate_size,
            quant_config=quant_config,
            prefix=add_prefix("experts", prefix),
            routing_method_type=RoutingMethodType.Renormalize,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(layer_id: ..., config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(layer_id: ..., config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 292-308: Method `Qwen3MoeSparseMoeBlock.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        forward_batch: Optional[ForwardBatch] = None,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
    ) -> torch.Tensor:

        if (
            not get_moe_a2a_backend().is_deepep()
            and not get_moe_a2a_backend().is_ascend_fuseep()
        ):
            return self.forward_normal(
                hidden_states, should_allreduce_fusion, use_reduce_scatter
            )
        else:
            return self.forward_deepep(hidden_states, forward_batch)
```
**EN:** This method implements `forward(hidden_states: ..., forward_batch: ...=..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., forward_batch: ...=..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 310-318: Method `Qwen3MoeSparseMoeBlock.get_moe_weights`
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

### Lines 320-350: Method `Qwen3MoeSparseMoeBlock.forward_normal`
```python
    def forward_normal(
        self,
        hidden_states: torch.Tensor,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
    ) -> torch.Tensor:
        num_tokens, hidden_dim = hidden_states.shape
        hidden_states = hidden_states.view(-1, hidden_dim)

        # router_logits: (num_tokens, n_experts)
        router_logits, _ = self.gate(hidden_states)
        topk_output = self.topk(hidden_states, router_logits)
        final_hidden_states = self.experts(hidden_states, topk_output)

        if self.ep_size > 1 and not should_skip_post_experts_all_reduce(
            is_tp_path=False,
            use_reduce_scatter=use_reduce_scatter,
            should_allreduce_fusion=should_allreduce_fusion,
        ):
            final_hidden_states = moe_expert_parallel_all_reduce(final_hidden_states)

        if self.tp_size > 1 and not should_skip_post_experts_all_reduce(
            is_tp_path=True,
            use_reduce_scatter=use_reduce_scatter,
            should_allreduce_fusion=should_allreduce_fusion,
        ):
            final_hidden_states = moe_tensor_model_parallel_all_reduce(
                final_hidden_states
            )

        return final_hidden_states.view(num_tokens, hidden_dim)
```
**EN:** This method implements `forward_normal(hidden_states: ..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_normal(hidden_states: ..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 352-372: Method `Qwen3MoeSparseMoeBlock.forward_deepep`
```python
    def forward_deepep(
        self, hidden_states: torch.Tensor, forward_batch: ForwardBatch
    ) -> torch.Tensor:
        if hidden_states.shape[0] > 0:
            # router_logits: (num_tokens, n_experts)
            router_logits, _ = self.gate(hidden_states)
            topk_output = self.topk(
                hidden_states,
                router_logits,
                num_token_non_padded=forward_batch.num_token_non_padded,
                expert_location_dispatch_info=ExpertLocationDispatchInfo.init_new(
                    layer_id=self.layer_id,
                ),
            )
        else:
            topk_output = self.topk.empty_topk_output(hidden_states.device)
        final_hidden_states = self.experts(
            hidden_states=hidden_states,
            topk_output=topk_output,
        )
        return final_hidden_states
```
**EN:** This method implements `forward_deepep(hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_deepep(hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 374-381: Method `Qwen3MoeSparseMoeBlock.op_gate`
```python
    def op_gate(self, state):
        if is_non_idle_and_non_empty(
            state.forward_batch.forward_mode, state.hidden_states_mlp_input
        ):
            # router_logits: (num_tokens, n_experts)
            state.router_logits, _ = self.gate(state.hidden_states_mlp_input)
        else:
            state.router_logits = None
```
**EN:** This method implements `op_gate(state)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeSparseMoeBlock`.
**CN:** 这个方法实现了 `op_gate(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeSparseMoeBlock` 内部调用。

### Lines 383-399: Method `Qwen3MoeSparseMoeBlock.op_select_experts`
```python
    def op_select_experts(self, state):
        router_logits = state.pop("router_logits")
        hidden_states = state.hidden_states_mlp_input
        if router_logits is not None:
            with get_global_expert_distribution_recorder().with_current_layer(
                self.layer_id
            ):
                state.topk_output = self.topk(
                    hidden_states=hidden_states,
                    router_logits=router_logits,
                    num_token_non_padded=state.forward_batch.num_token_non_padded,
                    expert_location_dispatch_info=ExpertLocationDispatchInfo.init_new(
                        layer_id=self.layer_id,
                    ),
                )
        else:
            state.topk_output = self.topk.empty_topk_output(hidden_states.device)
```
**EN:** This method implements `op_select_experts(state)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeSparseMoeBlock`.
**CN:** 这个方法实现了 `op_select_experts(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeSparseMoeBlock` 内部调用。

### Lines 401-407: Method `Qwen3MoeSparseMoeBlock.op_dispatch_a`
```python
    def op_dispatch_a(self, state):
        if self.ep_size > 1:
            self.experts.dispatcher.dispatch_a(
                hidden_states=state.pop("hidden_states_mlp_input"),
                topk_output=state.pop("topk_output"),
                tbo_subbatch_index=state.get("tbo_subbatch_index"),
            )
```
**EN:** This method implements `op_dispatch_a(state)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeSparseMoeBlock`.
**CN:** 这个方法实现了 `op_dispatch_a(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeSparseMoeBlock` 内部调用。

### Lines 409-416: Method `Qwen3MoeSparseMoeBlock.op_dispatch_b`
```python
    def op_dispatch_b(self, state):
        if self.ep_size > 1:
            with get_global_expert_distribution_recorder().with_current_layer(
                self.layer_id
            ):
                state.dispatch_output = self.experts.dispatcher.dispatch_b(
                    tbo_subbatch_index=state.get("tbo_subbatch_index"),
                )
```
**EN:** This method implements `op_dispatch_b(state)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeSparseMoeBlock`.
**CN:** 这个方法实现了 `op_dispatch_b(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeSparseMoeBlock` 内部调用。

### Lines 418-421: Method `Qwen3MoeSparseMoeBlock.op_experts`
```python
    def op_experts(self, state):
        state.combine_input = self.experts.run_moe_core(
            dispatch_output=state.dispatch_output,
        )
```
**EN:** This method implements `op_experts(state)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeSparseMoeBlock`.
**CN:** 这个方法实现了 `op_experts(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeSparseMoeBlock` 内部调用。

### Lines 423-429: Method `Qwen3MoeSparseMoeBlock.op_combine_a`
```python
    def op_combine_a(self, state):
        if self.ep_size > 1:
            self.experts.dispatcher.combine_a(
                combine_input=state.pop("combine_input"),
                tbo_subbatch_index=state.get("tbo_subbatch_index"),
            )
            state.pop("dispatch_output")
```
**EN:** This method implements `op_combine_a(state)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeSparseMoeBlock`.
**CN:** 这个方法实现了 `op_combine_a(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeSparseMoeBlock` 内部调用。

### Lines 431-435: Method `Qwen3MoeSparseMoeBlock.op_combine_b`
```python
    def op_combine_b(self, state):
        if self.ep_size > 1:
            state.hidden_states_after_combine = self.experts.dispatcher.combine_b(
                tbo_subbatch_index=state.get("tbo_subbatch_index"),
            )
```
**EN:** This method implements `op_combine_b(state)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeSparseMoeBlock`.
**CN:** 这个方法实现了 `op_combine_b(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeSparseMoeBlock` 内部调用。

### Lines 437-438: Method `Qwen3MoeSparseMoeBlock.op_output`
```python
    def op_output(self, state):
        state.hidden_states_mlp_output = state.pop("hidden_states_after_combine")
```
**EN:** This method implements `op_output(state)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeSparseMoeBlock`.
**CN:** 这个方法实现了 `op_output(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeSparseMoeBlock` 内部调用。

### Lines 441-441: Class `Qwen3MoeAttention` overview
```python
class Qwen3MoeAttention(nn.Module):
```
**EN:** Defines `Qwen3MoeAttention` as a reusable runtime type derived from nn.Module. The class groups 9 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3MoeAttention`，其继承关系为 nn.Module。这个类组织了 9 个方法，用于实现模型相关行为。

### Lines 442-550: Method `Qwen3MoeAttention.__init__`
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
        head_dim: Optional[int] = None,
        rms_norm_eps: float = 1e-06,
        attention_bias: bool = False,
        config: Optional[TConfig] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        dual_chunk_attention_config: Optional[dict[str, Any]] = None,
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size

        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()

        self.config = config
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
**EN:** This method implements `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 552-557: Method `Qwen3MoeAttention.op_prepare`
```python
    def op_prepare(self, state):
        state.attn_intermediate_state = self.forward_prepare(
            positions=state.positions,
            hidden_states=state.pop("hidden_states_after_comm_pre_attn"),
            forward_batch=state.forward_batch,
        )
```
**EN:** This method implements `op_prepare(state)` and prepares intermediate tensors, masks, or metadata before the main compute path.
**CN:** 这个方法实现了 `op_prepare(state)`，其作用是在主计算路径前准备中间张量、掩码或元数据。

### Lines 559-562: Method `Qwen3MoeAttention.op_core`
```python
    def op_core(self, state):
        state.hidden_states_after_attn = self.forward_core(
            state.pop("attn_intermediate_state")
        )
```
**EN:** This method implements `op_core(state)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeAttention`.
**CN:** 这个方法实现了 `op_core(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeAttention` 内部调用。

### Lines 564-588: Method `Qwen3MoeAttention.forward_prepare_npu`
```python
    def forward_prepare_npu(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ):
        qkv, _ = self.qkv_proj(hidden_states)
        if self.attn.layer_id == forward_batch.token_to_kv_pool.start_layer:
            self.rotary_emb.get_cos_sin_with_position(positions)
        q, k, v = split_qkv_rmsnorm_rope(
            qkv,
            self.rotary_emb.position_sin,
            self.rotary_emb.position_cos,
            self.q_size,
            self.kv_size,
            self.head_dim,
            eps=self.q_norm.variance_epsilon,
            q_weight=self.q_norm.weight,
            k_weight=self.k_norm.weight,
            q_bias=getattr(self.q_norm, "bias", None),
            k_bias=getattr(self.k_norm, "bias", None),
        )

        inner_state = q, k, v, forward_batch
        return None, forward_batch, inner_state
```
**EN:** This method implements `forward_prepare_npu(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_prepare_npu(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 590-601: Method `Qwen3MoeAttention.forward_prepare_native`
```python
    def forward_prepare_native(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ):
        qkv, _ = self.qkv_proj(hidden_states)

        q, k, v = self.apply_qk_norm_rope(qkv, positions, forward_batch)

        inner_state = q, k, v, forward_batch
        return None, forward_batch, inner_state
```
**EN:** This method implements `forward_prepare_native(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_prepare_native(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 603-657: Method `Qwen3MoeAttention.apply_qk_norm_rope`
```python
    def apply_qk_norm_rope(self, qkv, positions, forward_batch):
        use_fused = self.use_fused_qk_norm_rope and qkv.dtype == torch.bfloat16
        if use_fused:
            theta = self.rope_theta
            positions = (
                positions.view(-1).to(dtype=torch.int32, device=qkv.device).contiguous()
            )
            factor, low, high, attention_factor = compute_yarn_parameters(self.config)
            fused_qk_norm_rope(
                qkv,
                self.num_heads,
                self.num_kv_heads,
                self.num_kv_heads,
                self.head_dim,
                self.q_norm.variance_epsilon,
                self.q_norm.weight,
                self.k_norm.weight,
                theta,
                self.rotary_emb.is_neox_style,
                positions,
                factor,
                low,
                high,
                attention_factor,
            )
            q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
            self._used_fused_qk_norm_rope_last_call = True
        else:
            # Fallback to non-fused QK Norm & RoPE implementation
            q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
            q, k = apply_qk_norm(
                q=q,
                k=k,
                q_norm=self.q_norm,
                k_norm=self.k_norm,
                head_dim=self.head_dim,
                alt_stream=self.alt_stream,
            )
            q, k = self.rotary_emb(
                positions,
# ... truncated for brevity ...
```
**EN:** This method implements `apply_qk_norm_rope(qkv, positions, forward_batch)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeAttention`.
**CN:** 这个方法实现了 `apply_qk_norm_rope(qkv, positions, forward_batch)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeAttention` 内部调用。

### Lines 659-681: Method `Qwen3MoeAttention.forward_prepare`
```python
    def forward_prepare(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ):
        if hidden_states.shape[0] == 0:
            return hidden_states, forward_batch, None
        if (
            not _is_npu
            or forward_batch.forward_mode.is_extend_or_draft_extend_or_mixed()
        ):
            return self.forward_prepare_native(
                positions=positions,
                hidden_states=hidden_states,
                forward_batch=forward_batch,
            )
        else:
            return self.forward_prepare_npu(
                positions=positions,
                hidden_states=hidden_states,
                forward_batch=forward_batch,
            )
```
**EN:** This method implements `forward_prepare(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_prepare(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 683-703: Method `Qwen3MoeAttention.forward_core`
```python
    def forward_core(self, intermediate_state):
        hidden_states, forward_batch, inner_state = intermediate_state
        if inner_state is None:
            return hidden_states

        q, k, v, fb = inner_state

        must_save_kv = self._used_fused_qk_norm_rope_last_call
        save_kv_cache = must_save_kv or not (
            enable_fused_set_kv_buffer(forward_batch)
            and self.compatible_with_fused_kv_buffer
        )
        attn_output = self.attn(
            q,
            k,
            v,
            fb,
            save_kv_cache=save_kv_cache,
        )
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This method implements `forward_core(intermediate_state)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_core(intermediate_state)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 705-716: Method `Qwen3MoeAttention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        s = self.forward_prepare(
            positions=positions,
            hidden_states=hidden_states,
            forward_batch=forward_batch,
        )
        return self.forward_core(s)
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 719-719: Class `Qwen3MoeDecoderLayer` overview
```python
class Qwen3MoeDecoderLayer(nn.Module):
```
**EN:** Defines `Qwen3MoeDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 6 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3MoeDecoderLayer`，其继承关系为 nn.Module。这个类组织了 6 个方法，用于实现模型相关行为。

### Lines 720-804: Method `Qwen3MoeDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: Qwen3MoeConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        rope_theta, rope_scaling = get_rope_config(config)
        self.rope_theta = rope_theta
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        head_dim = getattr(
            config, "head_dim", config.hidden_size // config.num_attention_heads
        )
        rms_norm_eps = config.rms_norm_eps
        attention_bias = config.attention_bias
        dual_chunk_attention_config = getattr(
            config, "dual_chunk_attention_config", None
        )
        self.self_attn = Qwen3MoeAttention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            max_position_embeddings=max_position_embeddings,
            head_dim=head_dim,
            rms_norm_eps=rms_norm_eps,
            attention_bias=attention_bias,
            config=config,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
            dual_chunk_attention_config=dual_chunk_attention_config,
            alt_stream=alt_stream,
        )

# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 806-859: Method `Qwen3MoeDecoderLayer.forward`
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

        should_allreduce_fusion = (
            self.layer_communicator.should_fuse_mlp_allreduce_with_next_layer(
                forward_batch
            )
        )

        # For DP with padding, reduce scatter can be used instead of all-reduce.
        use_reduce_scatter = self.layer_communicator.should_use_reduce_scatter(
            forward_batch
# ... truncated for brevity ...
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., captured_last_layer_outputs: ...=..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., captured_last_layer_outputs: ...=..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 861-879: Method `Qwen3MoeDecoderLayer.op_comm_prepare_attn`
```python
    def op_comm_prepare_attn(
        self,
        state,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
        tbo_subbatch_index: Optional[int] = None,
    ):
        state.hidden_states_after_comm_pre_attn, state.residual_after_input_ln = (
            self.layer_communicator.prepare_attn(hidden_states, residual, forward_batch)
        )
        state.update(
            dict(
                forward_batch=forward_batch,
                positions=positions,
                tbo_subbatch_index=tbo_subbatch_index,
            )
        )
```
**EN:** This method implements `op_comm_prepare_attn(state, positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., tbo_subbatch_index: ...=...)` and implements attention-related tensor preparation or execution.
**CN:** 这个方法实现了 `op_comm_prepare_attn(state, positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., tbo_subbatch_index: ...=...)`，其作用是实现与注意力相关的张量准备或执行逻辑。

### Lines 881-888: Method `Qwen3MoeDecoderLayer.op_comm_prepare_mlp`
```python
    def op_comm_prepare_mlp(self, state):
        state.hidden_states_mlp_input, state.residual_after_comm_pre_mlp = (
            self.layer_communicator.prepare_mlp(
                state.pop("hidden_states_after_attn"),
                state.pop("residual_after_input_ln"),
                state.forward_batch,
            )
        )
```
**EN:** This method implements `op_comm_prepare_mlp(state)` and applies the feed-forward transformation used inside the network block.
**CN:** 这个方法实现了 `op_comm_prepare_mlp(state)`，其作用是执行网络块中的前馈变换。

### Lines 890-892: Method `Qwen3MoeDecoderLayer.op_mlp`
```python
    def op_mlp(self, state):
        hidden_states = state.pop("hidden_states_mlp_input")
        state.hidden_states_mlp_output = self.mlp(hidden_states, state.forward_batch)
```
**EN:** This method implements `op_mlp(state)` and applies the feed-forward transformation used inside the network block.
**CN:** 这个方法实现了 `op_mlp(state)`，其作用是执行网络块中的前馈变换。

### Lines 894-916: Method `Qwen3MoeDecoderLayer.op_comm_postprocess_layer`
```python
    def op_comm_postprocess_layer(self, state):
        hidden_states, residual = self.layer_communicator.postprocess_layer(
            state.pop("hidden_states_mlp_output"),
            state.pop("residual_after_comm_pre_mlp"),
            state.forward_batch,
        )

        output = dict(
            positions=state.positions,
            hidden_states=hidden_states,
            residual=residual,
            forward_batch=state.forward_batch,
            tbo_subbatch_index=state.tbo_subbatch_index,
        )

        state.clear(
            expect_keys={
                "positions",
                "forward_batch",
                "tbo_subbatch_index",
            }
        )
        return output
```
**EN:** This method implements `op_comm_postprocess_layer(state)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeDecoderLayer`.
**CN:** 这个方法实现了 `op_comm_postprocess_layer(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeDecoderLayer` 内部调用。

### Lines 919-919: Class `Qwen3MoeModel` overview
```python
class Qwen3MoeModel(Qwen2MoeModel):
```
**EN:** Defines `Qwen3MoeModel` as a reusable runtime type derived from Qwen2MoeModel. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3MoeModel`，其继承关系为 Qwen2MoeModel。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 920-934: Method `Qwen3MoeModel.__init__`
```python
    def __init__(
        self,
        config: Qwen3MoeConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        decoder_layer_type=Qwen3MoeDecoderLayer,
    ) -> None:
        alt_stream = torch.cuda.Stream() if _is_cuda else None
        super().__init__(
            config=config,
            quant_config=quant_config,
            prefix=prefix,
            decoder_layer_type=decoder_layer_type,
            alt_stream=alt_stream,
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., decoder_layer_type=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., decoder_layer_type=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 936-939: Method `Qwen3MoeModel.set_dflash_layers_to_capture`
```python
    def set_dflash_layers_to_capture(self, layers_to_capture: List[int]):
        self.layers_to_capture = layers_to_capture
        for layer_id in self.layers_to_capture:
            setattr(self.layers[layer_id], "_is_layer_to_capture", True)
```
**EN:** This method implements `set_dflash_layers_to_capture(layers_to_capture: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeModel`.
**CN:** 这个方法实现了 `set_dflash_layers_to_capture(layers_to_capture: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeModel` 内部调用。

### Lines 942-942: Class `Qwen3MoeForCausalLM` overview
```python
class Qwen3MoeForCausalLM(nn.Module):
```
**EN:** Defines `Qwen3MoeForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 11 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3MoeForCausalLM`，其继承关系为 nn.Module。这个类组织了 11 个方法，用于实现模型相关行为。

### Lines 943-951: Class `Qwen3MoeForCausalLM` attributes
```python
    fall_back_to_pt_during_load = False

    # Mapping from fused module names to their component weight names.
    # Required for quantization configs (e.g., ModelOpt FP4) to correctly identify
    # which layers should be skipped based on the exclude_modules/ignore list.
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3MoeForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3MoeForCausalLM` 在运行时的行为。

### Lines 953-983: Method `Qwen3MoeForCausalLM.__init__`
```python
    def __init__(
        self,
        config: Qwen3MoeConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        self.model = Qwen3MoeModel(
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
        self.capture_aux_hidden_states = False

        self.attn_cp_size = get_attn_context_model_parallel_world_size()
        self.attn_cp_rank = get_attn_context_model_parallel_rank()
        self.moe_dp_size = get_moe_data_parallel_world_size()

        assert self.attn_cp_size % self.moe_dp_size == 0, (
            f"attn_cp_size ({self.attn_cp_size}) must be divisible by "
            f"moe_dp_size ({self.moe_dp_size})"
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 985-986: Method `Qwen3MoeForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self) -> nn.Embedding:
        return self.model.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 988-1024: Method `Qwen3MoeForCausalLM.forward`
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
        if is_prefill_context_parallel_enabled():
            if can_cp_split(len(input_ids), self.attn_cp_size, forward_batch):
                forward_batch.attn_cp_metadata = prepare_context_parallel_metadata(
                    len(input_ids),
                    self.attn_cp_rank,
                    self.attn_cp_size,
                    forward_batch.seq_lens_cpu.tolist(),
                )

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
            logits_output = self.logits_processor(
                input_ids, hidden_states, self.lm_head, forward_batch, aux_hidden_states
            )
            return logits_output
        else:
            return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 1026-1067: Method `Qwen3MoeForCausalLM.forward_split_prefill`
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

### Lines 1069-1071: Method `Qwen3MoeForCausalLM.start_layer`
```python
    @property
    def start_layer(self):
        return self.model.start_layer
```
**EN:** This method implements `start_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `start_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeForCausalLM` 内部调用 装饰器：property。

### Lines 1073-1075: Method `Qwen3MoeForCausalLM.end_layer`
```python
    @property
    def end_layer(self):
        return self.model.end_layer
```
**EN:** This method implements `end_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `end_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeForCausalLM` 内部调用 装饰器：property。

### Lines 1077-1078: Method `Qwen3MoeForCausalLM.get_embed_and_head`
```python
    def get_embed_and_head(self):
        return self.model.embed_tokens.weight, self.lm_head.weight
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1080-1095: Method `Qwen3MoeForCausalLM.set_eagle3_layers_to_capture`
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
**EN:** This method implements `set_eagle3_layers_to_capture(layer_ids: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeForCausalLM`.
**CN:** 这个方法实现了 `set_eagle3_layers_to_capture(layer_ids: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeForCausalLM` 内部调用。

### Lines 1097-1107: Method `Qwen3MoeForCausalLM.set_dflash_layers_to_capture`
```python
    def set_dflash_layers_to_capture(self, layer_ids: List[int]):
        if not self.pp_group.is_last_rank:
            return

        if layer_ids is None:
            raise ValueError(
                "DFLASH requires explicit layer_ids for aux hidden capture."
            )

        self.capture_aux_hidden_states = True
        self.model.set_dflash_layers_to_capture([val + 1 for val in layer_ids])
```
**EN:** This method implements `set_dflash_layers_to_capture(layer_ids: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3MoeForCausalLM`.
**CN:** 这个方法实现了 `set_dflash_layers_to_capture(layer_ids: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3MoeForCausalLM` 内部调用。

### Lines 1109-1222: Method `Qwen3MoeForCausalLM.load_weights`
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

        # Pre-define `params_dict` to avoid repeated expensive traversal of model parameters.
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
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1224-1230: Method `Qwen3MoeForCausalLM.get_model_config_for_expert_location`
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

### Lines 1233-1233: Top-level assign
```python
EntryClass = Qwen3MoeForCausalLM
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
- `math`
- `typing.Any`
- `typing.Dict`
- `typing.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `typing.TypeVar`
- `torch`
- `torch.nn`
- `transformers.PretrainedConfig`
- `sglang.srt.distributed.get_attn_context_model_parallel_rank`
- `sglang.srt.distributed.get_attn_context_model_parallel_world_size`
- `sglang.srt.distributed.get_moe_data_parallel_world_size`
- `sglang.srt.distributed.get_moe_expert_parallel_world_size`
- `sglang.srt.distributed.get_moe_tensor_parallel_world_size`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_rank`
- `sglang.srt.distributed.moe_expert_parallel_all_reduce`
- `sglang.srt.distributed.moe_tensor_model_parallel_all_reduce`
- `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location.ModelConfigForExpertLocation`
- `sglang.srt.eplb.expert_location_dispatch.ExpertLocationDispatchInfo`
- `sglang.srt.layers.communicator.LayerCommunicator`
- `sglang.srt.layers.communicator.LayerScatterModes`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.moe.get_moe_a2a_backend`
- `sglang.srt.layers.moe.should_skip_post_experts_all_reduce`
- `sglang.srt.layers.moe.ep_moe.layer.get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`
- `sglang.srt.layers.moe.topk.TopK`
- `sglang.srt.layers.moe.utils.RoutingMethodType`
- `sglang.srt.layers.moe.utils.filter_moe_weight_param_global_expert`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.MRotaryEmbedding`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.utils.get_layer_id`
- `sglang.srt.layers.utils.cp_utils.can_cp_split`
- `sglang.srt.layers.utils.cp_utils.is_prefill_context_parallel_enabled`
- `sglang.srt.layers.utils.cp_utils.prepare_context_parallel_metadata`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.qwen2_moe.Qwen2MoeMLP`
- `sglang.srt.models.qwen2_moe.Qwen2MoeModel`
- `sglang.srt.models.utils.apply_qk_norm`
- `sglang.srt.models.utils.create_fused_set_kv_buffer_arg`
- `sglang.srt.models.utils.enable_fused_set_kv_buffer`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.LazyValue`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_flashinfer_available`
- `sglang.srt.utils.is_non_idle_and_non_empty`
- `sglang.srt.utils.is_npu`
- `sglang.srt.utils.hf_transformers_utils.get_rope_config`
- `sglang.jit_kernel.fused_qknorm_rope.can_use_fused_qk_norm_rope`
- `sglang.jit_kernel.fused_qknorm_rope.fused_qk_norm_rope`
- `sgl_kernel_npu.norm.split_qkv_rmsnorm_rope.split_qkv_rmsnorm_rope`
- `sglang.srt.layers.quantization.gguf.GGUFConfig`
