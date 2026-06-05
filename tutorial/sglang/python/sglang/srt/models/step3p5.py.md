# step3p5.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/step3p5.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the step3p5 model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 step3p5 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-54: Module imports
```python
from typing import Any, Dict, Iterable, Optional, Tuple, Union

import torch
import torch.nn.functional as F
from torch import nn

from sglang.srt.distributed import (
    get_moe_expert_parallel_world_size,
    get_pp_group,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.eplb.expert_location_dispatch import ExpertLocationDispatchInfo
from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.communicator import LayerCommunicator, LayerScatterModes
from sglang.srt.layers.dp_attention import (
    get_attention_tp_rank,
    get_attention_tp_size,
    is_dp_attention_enabled,
)
from sglang.srt.layers.layernorm import GemmaRMSNorm
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
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
from sglang.srt.layers.moe.topk import StandardTopKOutput, TopK
from sglang.srt.layers.moe.utils import (
    RoutingMethodType,
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、SGLang 运行时组件。

### Lines 56-56: Top-level assign
```python
Step3p5Config = None
```
**EN:** Defines or updates Step3p5Config, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 Step3p5Config，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 58-58: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 61-61: Class `Step3p5MLP` overview
```python
class Step3p5MLP(nn.Module):
```
**EN:** Defines `Step3p5MLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3p5MLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 62-96: Method `Step3p5MLP.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        swiglu_limit: Optional[float] = None,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        tp_size: Optional[int] = None,
        tp_rank: Optional[int] = None,
        reduce_results: bool = True,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
            tp_size=tp_size,
            tp_rank=tp_rank,
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
            tp_size=tp_size,
            tp_rank=tp_rank,
            reduce_results=reduce_results,
        )
        self.act_fn = SiluAndMul()
        self.limit = swiglu_limit
```
**EN:** This method implements `__init__(hidden_size: ..., intermediate_size: ..., swiglu_limit: ...=..., quant_config: ...=..., prefix: ...=..., tp_size: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., intermediate_size: ..., swiglu_limit: ...=..., quant_config: ...=..., prefix: ...=..., tp_size: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 98-110: Method `Step3p5MLP.forward`
```python
    def forward(self, x):
        if self.limit is not None:
            gate_up, _ = self.gate_up_proj(x)
            gate, up = gate_up.chunk(2, dim=-1)
            gate = F.silu(gate)
            gate = gate.clamp(min=None, max=self.limit)
            up = up.clamp(min=-self.limit, max=self.limit)
            output, _ = self.down_proj(gate * up)
        else:
            gate_up, _ = self.gate_up_proj(x)
            x = self.act_fn(gate_up)
            output, _ = self.down_proj(x)
        return output
```
**EN:** This method implements `forward(x)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 113-113: Class `Step3p5MoEMLP` overview
```python
class Step3p5MoEMLP(nn.Module):
```
**EN:** Defines `Step3p5MoEMLP` as a reusable runtime type derived from nn.Module. The class groups 13 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3p5MoEMLP`，其继承关系为 nn.Module。这个类组织了 13 个方法，用于实现模型相关行为。

### Lines 114-181: Method `Step3p5MoEMLP.__init__`
```python
    def __init__(
        self,
        config,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.layer_id = layer_id

        self.need_fp32_gate = config.need_fp32_gate
        self.routed_scaling_factor = config.moe_router_scaling_factor
        self.use_moe_router_bias = config.use_moe_router_bias
        if self.use_moe_router_bias:
            self.router_bias = nn.Parameter(
                torch.zeros(config.moe_num_experts, dtype=torch.float32),
                requires_grad=False,
            )

        if self.tp_size > config.moe_num_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {config.moe_num_experts}."
            )

        self.limit = config.swiglu_limits[layer_id]
        self.limit = self.limit if self.limit > 0 else None

        self.topk = TopK(
            top_k=config.moe_top_k,
            renormalize=True,
            use_grouped_topk=False,
            scoring_func="sigmoid",
            correction_bias=self.router_bias,
            apply_routed_scaling_factor_on_output=False,
            layer_id=layer_id,
        )

        self.experts = get_moe_impl_class(quant_config)(
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config, layer_id: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config, layer_id: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 183-199: Method `Step3p5MoEMLP.forward`
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

### Lines 201-209: Method `Step3p5MoEMLP.get_moe_weights`
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

### Lines 211-242: Method `Step3p5MoEMLP.forward_normal`
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
        if self.need_fp32_gate:
            router_logits = torch.matmul(
                hidden_states.to(torch.float32), self.gate.weight.t().to(torch.float32)
            )
        else:
            # router_logits: (batch * sequence_length, n_experts)
            router_logits, _ = self.gate(hidden_states)
        topk_output = self.topk(hidden_states, router_logits)
        if self.routed_scaling_factor != 1.0:
            topk_output = StandardTopKOutput(
                topk_weights=topk_output.topk_weights * self.routed_scaling_factor,
                topk_ids=topk_output.topk_ids,
                router_logits=topk_output.router_logits,
            )
        final_hidden_states = self.experts(hidden_states, topk_output)
        if self.tp_size > 1 and not should_skip_post_experts_all_reduce(
            is_tp_path=True,
            use_reduce_scatter=use_reduce_scatter,
            should_allreduce_fusion=should_allreduce_fusion,
        ):
            final_hidden_states = tensor_model_parallel_all_reduce(final_hidden_states)

        return final_hidden_states.view(num_tokens, hidden_dim)
```
**EN:** This method implements `forward_normal(hidden_states: ..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_normal(hidden_states: ..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 244-264: Method `Step3p5MoEMLP.forward_deepep`
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

### Lines 266-273: Method `Step3p5MoEMLP.op_gate`
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
**EN:** This method implements `op_gate(state)` and implements a focused helper that supports the surrounding runtime flow inside `Step3p5MoEMLP`.
**CN:** 这个方法实现了 `op_gate(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3p5MoEMLP` 内部调用。

### Lines 275-291: Method `Step3p5MoEMLP.op_select_experts`
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
**EN:** This method implements `op_select_experts(state)` and implements a focused helper that supports the surrounding runtime flow inside `Step3p5MoEMLP`.
**CN:** 这个方法实现了 `op_select_experts(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3p5MoEMLP` 内部调用。

### Lines 293-299: Method `Step3p5MoEMLP.op_dispatch_a`
```python
    def op_dispatch_a(self, state):
        if self.ep_size > 1:
            self.experts.dispatcher.dispatch_a(
                hidden_states=state.pop("hidden_states_mlp_input"),
                topk_output=state.pop("topk_output"),
                tbo_subbatch_index=state.get("tbo_subbatch_index"),
            )
```
**EN:** This method implements `op_dispatch_a(state)` and implements a focused helper that supports the surrounding runtime flow inside `Step3p5MoEMLP`.
**CN:** 这个方法实现了 `op_dispatch_a(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3p5MoEMLP` 内部调用。

### Lines 301-308: Method `Step3p5MoEMLP.op_dispatch_b`
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
**EN:** This method implements `op_dispatch_b(state)` and implements a focused helper that supports the surrounding runtime flow inside `Step3p5MoEMLP`.
**CN:** 这个方法实现了 `op_dispatch_b(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3p5MoEMLP` 内部调用。

### Lines 310-313: Method `Step3p5MoEMLP.op_experts`
```python
    def op_experts(self, state):
        state.combine_input = self.experts.run_moe_core(
            dispatch_output=state.dispatch_output,
        )
```
**EN:** This method implements `op_experts(state)` and implements a focused helper that supports the surrounding runtime flow inside `Step3p5MoEMLP`.
**CN:** 这个方法实现了 `op_experts(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3p5MoEMLP` 内部调用。

### Lines 315-321: Method `Step3p5MoEMLP.op_combine_a`
```python
    def op_combine_a(self, state):
        if self.ep_size > 1:
            self.experts.dispatcher.combine_a(
                combine_input=state.pop("combine_input"),
                tbo_subbatch_index=state.get("tbo_subbatch_index"),
            )
            state.pop("dispatch_output")
```
**EN:** This method implements `op_combine_a(state)` and implements a focused helper that supports the surrounding runtime flow inside `Step3p5MoEMLP`.
**CN:** 这个方法实现了 `op_combine_a(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3p5MoEMLP` 内部调用。

### Lines 323-327: Method `Step3p5MoEMLP.op_combine_b`
```python
    def op_combine_b(self, state):
        if self.ep_size > 1:
            state.hidden_states_after_combine = self.experts.dispatcher.combine_b(
                tbo_subbatch_index=state.get("tbo_subbatch_index"),
            )
```
**EN:** This method implements `op_combine_b(state)` and implements a focused helper that supports the surrounding runtime flow inside `Step3p5MoEMLP`.
**CN:** 这个方法实现了 `op_combine_b(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3p5MoEMLP` 内部调用。

### Lines 329-330: Method `Step3p5MoEMLP.op_output`
```python
    def op_output(self, state):
        state.hidden_states_mlp_output = state.pop("hidden_states_after_combine")
```
**EN:** This method implements `op_output(state)` and implements a focused helper that supports the surrounding runtime flow inside `Step3p5MoEMLP`.
**CN:** 这个方法实现了 `op_output(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3p5MoEMLP` 内部调用。

### Lines 333-333: Class `Step3p5Attention` overview
```python
class Step3p5Attention(nn.Module):
```
**EN:** Defines `Step3p5Attention` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3p5Attention`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 334-432: Method `Step3p5Attention.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        rope_theta: float = 1000000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        head_dim: Optional[int] = None,
        max_position_embeddings: int = 32768,
        quant_config: Optional[QuantizationConfig] = None,
        rms_norm_eps: float = None,
        partial_rotary_factor: float = 1.0,
        use_head_wise_attn_gate: bool = False,
        sliding_window_size: int = -1,  # if is -1 ,normal attention,else ,window attention
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()

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
        self.kv_size = self.num_kv_heads * self.head_dim
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 434-441: Method `Step3p5Attention.forward_prepare_native`
```python
    def forward_prepare_native(self, positions, hidden_states):
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q_shape, k_shape = q.shape, k.shape
        q = self.q_norm(q.reshape(-1, self.head_dim)).reshape(q_shape)
        k = self.k_norm(k.reshape(-1, self.head_dim)).reshape(k_shape)
        q, k = self.rotary_emb(positions, q, k)
        return q, k, v
```
**EN:** This method implements `forward_prepare_native(positions, hidden_states)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_prepare_native(positions, hidden_states)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 443-468: Method `Step3p5Attention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:

        q, k, v = self.forward_prepare_native(
            positions=positions,
            hidden_states=hidden_states,
        )
        if self.use_head_wise_attn_gate:
            gate_states, _ = self.g_proj(hidden_states)
        attn_output = self.attn(q, k, v, forward_batch)
        if self.use_head_wise_attn_gate:
            output = (
                attn_output.view(
                    attn_output.shape[0],
                    self.num_heads,  # TODO: check if this is correct
                    self.head_dim,
                )
                * gate_states.unsqueeze(-1).sigmoid()
            )
            attn_output = output.view(*attn_output.shape)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 471-471: Class `Step3p5DecoderLayer` overview
```python
class Step3p5DecoderLayer(nn.Module):
```
**EN:** Defines `Step3p5DecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3p5DecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 472-593: Method `Step3p5DecoderLayer.__init__`
```python
    def __init__(
        self,
        config: Step3p5Config,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        layer_types = config.layer_types
        yarn_only_types = config.yarn_only_types
        if layer_types[layer_id] not in yarn_only_types:
            rope_scaling = None
        else:
            rope_scaling = config.rope_scaling
        rope_theta = config.rope_theta
        max_position_embeddings = config.max_position_embeddings
        head_dim = config.head_dim
        moe_layers_set = {int(x) for x in config.moe_layers_enum.split(",")}
        self.num_attention_heads = config.num_attention_heads
        self.num_key_value_heads = config.num_attention_groups
        self.is_moe_layer = layer_id in moe_layers_set
        self.is_previous_layer_sparse = (layer_id - 1) in moe_layers_set
        self.is_next_layer_sparse = (layer_id + 1) in moe_layers_set
        num_hidden_layers = config.num_hidden_layers

        if (
            config.swiglu_limits_shared
            and config.swiglu_limits_shared[layer_id] is not None
            and config.swiglu_limits_shared[layer_id] != 0
        ):
            swiglu_limit_shared = config.swiglu_limits_shared[layer_id]
        else:
            swiglu_limit_shared = None

        self.sliding_window = -1

        enable_sliding_window = layer_types[layer_id] == "sliding_attention"

# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 595-659: Method `Step3p5DecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        # Self Attention
        hidden_states, residual = self.layer_communicator.prepare_attn(
            hidden_states,
            residual,
            forward_batch,
            post_residual_addition=post_residual_addition,
        )
        if hidden_states.shape[0] != 0:
            hidden_states = self.self_attn(
                positions=positions,
                hidden_states=hidden_states,
                forward_batch=forward_batch,
            )
        # Fully Connected
        hidden_states, residual = self.layer_communicator.prepare_mlp(
            hidden_states,
            residual,
            forward_batch,
        )

        should_allreduce_fusion = (
            self.layer_communicator.should_fuse_mlp_allreduce_with_next_layer(
                forward_batch
            )
        )
        use_reduce_scatter = self.layer_communicator.should_use_reduce_scatter(
            forward_batch
        )

        if self.use_moe:
            # Both share_expert and MoE return unreduced (TP-partial) outputs.
            # Combine them first, then do a single all-reduce — saving one
# ... truncated for brevity ...
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., post_residual_addition: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., post_residual_addition: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 662-662: Class `Step3p5Model` overview
```python
class Step3p5Model(nn.Module):
```
**EN:** Defines `Step3p5Model` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3p5Model`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 663-709: Method `Step3p5Model.__init__`
```python
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.vocab_size = config.vocab_size
        self.pp_group = get_pp_group()

        alt_stream = torch.cuda.Stream() if _is_cuda else None

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                quant_config=quant_config,
                enable_tp=not is_dp_attention_enabled(),
                prefix=add_prefix("embed_tokens", prefix),
                params_dtype=(
                    torch.float32
                    if get_global_server_args().rl_on_policy_target is not None
                    else None
                ),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            # 1,
            lambda idx, prefix: Step3p5DecoderLayer(
                layer_id=idx,
                config=config,
                quant_config=quant_config,
                prefix=prefix,
                alt_stream=alt_stream,
            ),
            pp_rank=self.pp_group.rank_in_group,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config, quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config, quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 711-715: Method `Step3p5Model.get_input_embedding`
```python
    def get_input_embedding(self, input_ids: torch.Tensor) -> torch.Tensor:
        if hasattr(self.config, "scale_emb"):
            return self.get_input_embeddings()(input_ids) * self.config.scale_emb
        else:
            return self.get_input_embeddings()(input_ids)
```
**EN:** This method implements `get_input_embedding(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embedding(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 717-718: Method `Step3p5Model.get_input_embeddings`
```python
    def get_input_embeddings(self) -> nn.Embedding:
        return self.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 720-774: Method `Step3p5Model.forward`
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

        for i in range(self.start_layer, self.end_layer):
            layer = self.layers[i]
            hidden_states, residual = layer(
                positions,
                hidden_states,
                forward_batch,
                residual,
            )
            # break
        if not self.pp_group.is_last_rank:
            return PPProxyTensors(
                {
                    "hidden_states": hidden_states,
                    "residual": residual,
                }
            )
        else:
            hidden_states_before_norm = None
            if not self.pp_group.is_last_rank:
                return PPProxyTensors(
                    {
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 777-778: Class `Step3p5ForCausalLM` overview
```python
class Step3p5ForCausalLM(nn.Module):
    # BitandBytes specific attributes
```
**EN:** Defines `Step3p5ForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 8 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Step3p5ForCausalLM`，其继承关系为 nn.Module。这个类组织了 8 个方法，用于实现模型相关行为。

### Lines 779-795: Class `Step3p5ForCausalLM` attributes
```python
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
```
**EN:** Defines class-level attributes and metadata that shape how `Step3p5ForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Step3p5ForCausalLM` 在运行时的行为。

### Lines 797-844: Method `Step3p5ForCausalLM.__init__`
```python
    def __init__(
        self,
        config: Step3p5Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        self.model = Step3p5Model(
            config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )

        self.tie_word_embeddings = False
        self.num_fused_shared_experts = 0

        # handle the lm head on different pp ranks
        if self.pp_group.is_last_rank:
            if self.pp_group.world_size == 1 and self.tie_word_embeddings:
                self.lm_head = self.model.embed_tokens
            else:
                self.lm_head = ParallelLMHead(
                    config.vocab_size,
                    config.hidden_size,
                    quant_config=quant_config,
                    use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
                    prefix=add_prefix("lm_head", prefix),
                )
        else:
            # ranks other than the last rank will have a placeholder layer
            self.lm_head = PPMissingLayer()

        # perform weight tying for PP
        if self.pp_group.world_size > 1 and self.tie_word_embeddings:
            if self.pp_group.is_first_rank:
                self.pp_group.send(
                    self.model.embed_tokens.weight, dst=self.pp_group.world_size - 1
                )
            elif self.pp_group.is_last_rank:
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 846-847: Method `Step3p5ForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self) -> nn.Embedding:
        return self.model.get_input_embeddings()
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 849-875: Method `Step3p5ForCausalLM.forward`
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
        hidden_states, hidden_states_before_norm = self.model(
            input_ids,
            positions,
            forward_batch,
            input_embeds,
            pp_proxy_tensors=pp_proxy_tensors,
        )

        if self.pp_group.is_last_rank:
            return self.logits_processor(
                input_ids,
                hidden_states,
                self.lm_head,
                forward_batch,
                hidden_states_before_norm=hidden_states_before_norm,
            )
        else:
            return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 877-879: Method `Step3p5ForCausalLM.start_layer`
```python
    @property
    def start_layer(self):
        return self.model.start_layer
```
**EN:** This method implements `start_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Step3p5ForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `start_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3p5ForCausalLM` 内部调用 装饰器：property。

### Lines 881-883: Method `Step3p5ForCausalLM.end_layer`
```python
    @property
    def end_layer(self):
        return self.model.end_layer
```
**EN:** This method implements `end_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Step3p5ForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `end_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Step3p5ForCausalLM` 内部调用 装饰器：property。

### Lines 885-1023: Method `Step3p5ForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]], is_nextn=False):
        # NOTE:
        # Step3p5 HF checkpoints (e.g. MTP/nextn variants) may include an extra
        # "nextn predict layer" appended after the main decoder layers, such as:
        #   model.layers.<num_hidden_layers>.(eh_proj|enorm|hnorm|transformer.shared_head.*)
        # This implementation currently does NOT instantiate those nextn modules,
        # so we must safely skip them (or load them only when a corresponding
        # nextn model is implemented).

        def _get_layer_id_from_weight_name(weight_name: str) -> Optional[int]:
            # Expected format: "model.layers.<id>...."
            parts = weight_name.split(".")
            if len(parts) >= 3 and parts[0] == "model" and parts[1] == "layers":
                try:
                    return int(parts[2])
                except ValueError:
                    return None
            return None

        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
            (".qkv_proj", ".v_proj", "v"),
            (".gate_up_proj", ".gate_proj", 0),
            (".gate_up_proj", ".up_proj", 1),
        ]

        if self.num_fused_shared_experts > 0:
            assert self.num_fused_shared_experts == 1

        expert_params_mapping = FusedMoE.make_expert_params_mapping(
            ckpt_gate_proj_name="gate_proj",
            ckpt_down_proj_name="down_proj",
            ckpt_up_proj_name="up_proj",
            num_experts=self.config.moe_num_experts + self.num_fused_shared_experts,
        )

        params_dict = dict(self.named_parameters())
        loaded_params = set()
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ..., is_nextn=...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ..., is_nextn=...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1025-1026: Method `Step3p5ForCausalLM.get_embed_and_head`
```python
    def get_embed_and_head(self):
        return self.model.embed_tokens.weight, self.lm_head.weight
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1028-1034: Method `Step3p5ForCausalLM.set_embed_and_head`
```python
    def set_embed_and_head(self, embed, head):
        del self.model.embed_tokens.weight
        del self.lm_head.weight
        self.model.embed_tokens.weight = embed
        self.lm_head.weight = head
        torch.cuda.empty_cache()
        torch.cuda.synchronize()
```
**EN:** This method implements `set_embed_and_head(embed, head)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_embed_and_head(embed, head)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1037-1037: Top-level assign
```python
EntryClass = Step3p5ForCausalLM
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `typing.Any`
- `typing.Dict`
- `typing.Iterable`
- `typing.Optional`
- `typing.Tuple`
- `typing.Union`
- `torch`
- `torch.nn.functional`
- `torch.nn`
- `sglang.srt.distributed.get_moe_expert_parallel_world_size`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_rank`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.distributed.tensor_model_parallel_all_reduce`
- `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location_dispatch.ExpertLocationDispatchInfo`
- `sglang.srt.layers.activation.SiluAndMul`
- `sglang.srt.layers.communicator.LayerCommunicator`
- `sglang.srt.layers.communicator.LayerScatterModes`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
- `sglang.srt.layers.layernorm.GemmaRMSNorm`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.MergedColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.moe.get_moe_a2a_backend`
- `sglang.srt.layers.moe.should_skip_post_experts_all_reduce`
- `sglang.srt.layers.moe.ep_moe.layer.get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`
- `sglang.srt.layers.moe.topk.StandardTopKOutput`
- `sglang.srt.layers.moe.topk.TopK`
- `sglang.srt.layers.moe.utils.RoutingMethodType`
- `sglang.srt.layers.moe.utils.filter_moe_weight_param_global_expert`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_non_idle_and_non_empty`
- `sglang.srt.utils.make_layers`
