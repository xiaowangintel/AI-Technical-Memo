# mimo_v2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/mimo_v2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the mimo v2 model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 mimo v2 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 15-89: Module imports
```python
import logging
from typing import Any, Dict, Iterable, List, Optional, Tuple, Union

import torch
import torch.nn.functional as F
from torch import nn

from sglang.srt.batch_overlap.two_batch_overlap import model_forward_maybe_tbo
from sglang.srt.configs.model_config import get_mimo_v2_fused_qkv_expected_tp_size
from sglang.srt.distributed import (
    get_moe_expert_parallel_world_size,
    get_pp_group,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.eplb.expert_location import ModelConfigForExpertLocation
from sglang.srt.eplb.expert_location_dispatch import ExpertLocationDispatchInfo
from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.communicator import (
    LayerCommunicator,
    LayerScatterModes,
    ScatterMode,
    enable_moe_dense_fully_dp,
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
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.moe import (
    get_moe_a2a_backend,
    get_moe_runner_backend,
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、SGLang 运行时组件。

### Lines 91-91: Top-level assign
```python
MiMoV2Config = None
```
**EN:** Defines or updates MiMoV2Config, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 MiMoV2Config，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 93-93: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 96-125: Function `load_mimo_v2_qkv_proj_weight`
```python
def load_mimo_v2_qkv_proj_weight(
    name, param, loaded_weight, expected_fused_tp_size: Optional[int] = None
):
    if loaded_weight.shape == param.shape:
        # The checkpoint already stores this rank's qkv_proj shard.
        default_weight_loader(param, loaded_weight)
        return

    if loaded_weight.ndim != param.ndim or loaded_weight.shape[1:] != param.shape[1:]:
        raise ValueError(
            f"qkv_proj weight {name}: unexpected shape {tuple(loaded_weight.shape)}; "
            f"expected sharded {tuple(param.shape)}"
        )

    tp_size = get_attention_tp_size()
    tp_rank = get_attention_tp_rank()
    if expected_fused_tp_size is not None and tp_size != expected_fused_tp_size:
        raise ValueError(
            f"MiMoV2 fused qkv_proj checkpoint is TP={expected_fused_tp_size}-"
            f"interleaved; got attention tp_size={tp_size} while loading {name}."
        )

    fused_shape = (param.shape[0] * tp_size, *param.shape[1:])
    if tuple(loaded_weight.shape) != fused_shape:
        raise ValueError(
            f"qkv_proj weight {name}: unexpected shape {tuple(loaded_weight.shape)}; "
            f"expected fused {fused_shape} or sharded {tuple(param.shape)}"
        )

    default_weight_loader(param, loaded_weight.chunk(tp_size, dim=0)[tp_rank])
```
**EN:** This function implements `load_mimo_v2_qkv_proj_weight(name, param, loaded_weight, expected_fused_tp_size: ...=...)` and handles weight mapping, filtering, or loading for this model component.
**CN:** 这个函数实现了 `load_mimo_v2_qkv_proj_weight(name, param, loaded_weight, expected_fused_tp_size: ...=...)`，其作用是处理该模型组件的权重映射、筛选或加载逻辑。

### Lines 128-128: Class `MiMoV2MLP` overview
```python
class MiMoV2MLP(nn.Module):
```
**EN:** Defines `MiMoV2MLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoV2MLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 129-167: Method `MiMoV2MLP.__init__`
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
                f"Unsupported activation: {hidden_act}. "
                "Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()
```
**EN:** This method implements `__init__(hidden_size: ..., intermediate_size: ..., hidden_act: ..., quant_config: ...=..., reduce_results: ...=..., prefix: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., intermediate_size: ..., hidden_act: ..., quant_config: ...=..., reduce_results: ...=..., prefix: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 169-184: Method `MiMoV2MLP.forward`
```python
    def forward(
        self,
        x,
        forward_batch: ForwardBatch = None,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
    ):
        if (self.tp_size == 1) and x.shape[0] == 0:
            return x

        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(
            x, skip_all_reduce=should_allreduce_fusion or use_reduce_scatter
        )
        return x
```
**EN:** This method implements `forward(x, forward_batch: ...=..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x, forward_batch: ...=..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 187-187: Class `MoEGate` overview
```python
class MoEGate(nn.Module):
```
**EN:** Defines `MoEGate` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MoEGate`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 188-213: Method `MoEGate.__init__`
```python
    def __init__(
        self,
        config,
        quant_config,
        prefix: str = "",
        is_nextn: bool = False,
    ):
        super().__init__()
        self.is_nextn = is_nextn
        self.dtype = torch.float32
        self.weight = nn.Parameter(
            torch.empty((config.n_routed_experts, config.hidden_size), dtype=self.dtype)
        )
        if config.topk_method == "noaux_tc":
            correction_bias_dtype = (
                torch.bfloat16
                if quant_config is not None
                and quant_config.get_name() == "modelopt_fp4"
                and get_moe_runner_backend().is_flashinfer_trtllm()
                else self.dtype
            )
            self.e_score_correction_bias = nn.Parameter(
                torch.empty((config.n_routed_experts), dtype=correction_bias_dtype)
            )
        else:
            self.e_score_correction_bias = None
```
**EN:** This method implements `__init__(config, quant_config, prefix: ...=..., is_nextn: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config, quant_config, prefix: ...=..., is_nextn: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 215-218: Method `MoEGate.forward`
```python
    def forward(self, hidden_states):
        logits = F.linear(hidden_states.to(self.dtype), self.weight, None)

        return logits
```
**EN:** This method implements `forward(hidden_states)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 221-222: Class `MiMoV2MoE` overview
```python
class MiMoV2MoE(nn.Module):
```
**EN:** Defines `MiMoV2MoE` as a reusable runtime type derived from nn.Module. The class groups 13 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoV2MoE`，其继承关系为 nn.Module。这个类组织了 13 个方法，用于实现模型相关行为。

### Lines 223-303: Method `MiMoV2MoE.__init__`
```python
    def __init__(
        self,
        config: MiMoV2Config,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        is_nextn: bool = False,
    ):
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()

        self.config = config
        self.layer_id = layer_id

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

        self.gate = MoEGate(
            config=config,
            quant_config=quant_config,
            prefix=add_prefix("gate", prefix),
            is_nextn=is_nextn,
        )

        experts_type = get_moe_impl_class(quant_config)
        self.experts = experts_type(
            num_experts=config.n_routed_experts
            + get_global_server_args().ep_num_redundant_experts,
            top_k=config.num_experts_per_tok,
            hidden_size=config.hidden_size,
            intermediate_size=config.moe_intermediate_size,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., is_nextn: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., is_nextn: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 305-310: Method `MiMoV2MoE.get_moe_weights`
```python
    def get_moe_weights(self):
        return [
            x.data
            for name, x in self.experts.named_parameters()
            if name not in ["correction_bias"]
        ]
```
**EN:** This method implements `get_moe_weights()` and handles weight mapping, filtering, or loading for this model component.
**CN:** 这个方法实现了 `get_moe_weights()`，其作用是处理该模型组件的权重映射、筛选或加载逻辑。

### Lines 312-326: Method `MiMoV2MoE.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        forward_batch: Optional[ForwardBatch] = None,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
    ) -> torch.Tensor:
        if not self._enable_a2a_moe:
            return self.forward_normal(
                hidden_states,
                should_allreduce_fusion,
                use_reduce_scatter,
            )
        else:
            return self.forward_deepep(hidden_states, forward_batch)
```
**EN:** This method implements `forward(hidden_states: ..., forward_batch: ...=..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., forward_batch: ...=..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 328-351: Method `MiMoV2MoE.forward_normal`
```python
    def forward_normal(
        self,
        hidden_states: torch.Tensor,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
    ) -> torch.Tensor:

        if hidden_states.shape[0] > 0:
            # router_logits: (num_tokens, n_experts)
            router_logits = self.gate(hidden_states)
            topk_output = self.topk(hidden_states, router_logits)
        else:
            topk_output = self.topk.empty_topk_output(hidden_states.device)

        final_hidden_states = self.experts(hidden_states, topk_output)

        if self.tp_size > 1 and not should_skip_post_experts_all_reduce(
            is_tp_path=True,
            use_reduce_scatter=use_reduce_scatter,
            should_allreduce_fusion=should_allreduce_fusion,
        ):
            final_hidden_states = tensor_model_parallel_all_reduce(final_hidden_states)

        return final_hidden_states
```
**EN:** This method implements `forward_normal(hidden_states: ..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_normal(hidden_states: ..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 353-374: Method `MiMoV2MoE.forward_deepep`
```python
    def forward_deepep(
        self, hidden_states: torch.Tensor, forward_batch: ForwardBatch
    ) -> torch.Tensor:
        if hidden_states.shape[0] > 0:
            # router_logits: (num_tokens, n_experts)
            router_logits = self.gate(hidden_states)
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
            hidden_states=hidden_states, topk_output=topk_output
        )

        return final_hidden_states
```
**EN:** This method implements `forward_deepep(hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_deepep(hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 376-383: Method `MiMoV2MoE.op_gate`
```python
    def op_gate(self, state):
        if is_non_idle_and_non_empty(
            state.forward_batch.forward_mode, state.hidden_states_mlp_input
        ):
            # router_logits: (num_tokens, n_experts)
            state.router_logits = self.gate(state.hidden_states_mlp_input)
        else:
            state.router_logits = None
```
**EN:** This method implements `op_gate(state)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2MoE`.
**CN:** 这个方法实现了 `op_gate(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2MoE` 内部调用。

### Lines 385-401: Method `MiMoV2MoE.op_select_experts`
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
**EN:** This method implements `op_select_experts(state)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2MoE`.
**CN:** 这个方法实现了 `op_select_experts(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2MoE` 内部调用。

### Lines 403-409: Method `MiMoV2MoE.op_dispatch_a`
```python
    def op_dispatch_a(self, state):
        if self.ep_size > 1:
            self.experts.dispatcher.dispatch_a(
                hidden_states=state.pop("hidden_states_mlp_input"),
                topk_output=state.pop("topk_output"),
                tbo_subbatch_index=state.get("tbo_subbatch_index"),
            )
```
**EN:** This method implements `op_dispatch_a(state)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2MoE`.
**CN:** 这个方法实现了 `op_dispatch_a(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2MoE` 内部调用。

### Lines 411-418: Method `MiMoV2MoE.op_dispatch_b`
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
**EN:** This method implements `op_dispatch_b(state)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2MoE`.
**CN:** 这个方法实现了 `op_dispatch_b(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2MoE` 内部调用。

### Lines 420-423: Method `MiMoV2MoE.op_experts`
```python
    def op_experts(self, state):
        state.combine_input = self.experts.run_moe_core(
            dispatch_output=state.dispatch_output,
        )
```
**EN:** This method implements `op_experts(state)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2MoE`.
**CN:** 这个方法实现了 `op_experts(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2MoE` 内部调用。

### Lines 425-431: Method `MiMoV2MoE.op_combine_a`
```python
    def op_combine_a(self, state):
        if self.ep_size > 1:
            self.experts.dispatcher.combine_a(
                combine_input=state.pop("combine_input"),
                tbo_subbatch_index=state.get("tbo_subbatch_index"),
            )
            state.pop("dispatch_output")
```
**EN:** This method implements `op_combine_a(state)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2MoE`.
**CN:** 这个方法实现了 `op_combine_a(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2MoE` 内部调用。

### Lines 433-437: Method `MiMoV2MoE.op_combine_b`
```python
    def op_combine_b(self, state):
        if self.ep_size > 1:
            state.hidden_states_after_combine = self.experts.dispatcher.combine_b(
                tbo_subbatch_index=state.get("tbo_subbatch_index"),
            )
```
**EN:** This method implements `op_combine_b(state)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2MoE`.
**CN:** 这个方法实现了 `op_combine_b(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2MoE` 内部调用。

### Lines 439-440: Method `MiMoV2MoE.op_output`
```python
    def op_output(self, state):
        state.hidden_states_mlp_output = state.pop("hidden_states_after_combine")
```
**EN:** This method implements `op_output(state)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2MoE`.
**CN:** 这个方法实现了 `op_output(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2MoE` 内部调用。

### Lines 443-443: Class `MiMoV2Attention` overview
```python
class MiMoV2Attention(nn.Module):
```
**EN:** Defines `MiMoV2Attention` as a reusable runtime type derived from nn.Module. The class groups 6 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoV2Attention`，其继承关系为 nn.Module。这个类组织了 6 个方法，用于实现模型相关行为。

### Lines 444-543: Method `MiMoV2Attention.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        head_dim: Optional[int] = None,
        v_head_dim: Optional[int] = None,
        v_scale: Optional[float] = None,
        sliding_window_size: int = -1,  # if is -1 ,normal attention,else ,window attention
        attention_bias: bool = False,
        attention_sink_bias: bool = False,
        layer_id: int = 0,
        rope_theta: float = 1000000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        max_position_embeddings: int = 32768,
        quant_config: Optional[QuantizationConfig] = None,
        partial_rotary_factor: float = 1.0,
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
        self.head_dim = head_dim
        self.v_head_dim = v_head_dim if v_head_dim is not None else head_dim
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., head_dim: ...=..., v_head_dim: ...=..., v_scale: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., head_dim: ...=..., v_head_dim: ...=..., v_scale: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 545-550: Method `MiMoV2Attention.op_prepare`
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

### Lines 552-555: Method `MiMoV2Attention.op_core`
```python
    def op_core(self, state):
        state.hidden_states_after_attn = self.forward_core(
            state.pop("attn_intermediate_state")
        )
```
**EN:** This method implements `op_core(state)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2Attention`.
**CN:** 这个方法实现了 `op_core(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2Attention` 内部调用。

### Lines 557-573: Method `MiMoV2Attention.forward_prepare`
```python
    def forward_prepare(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ):
        if hidden_states.shape[0] == 0:
            return hidden_states, forward_batch, None
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.k_size, self.v_size], dim=-1)

        q, k = self.rotary_emb(positions, q, k)
        if self.v_scale is not None:
            v = v * self.v_scale

        inner_state = q, k, v, forward_batch
        return None, forward_batch, inner_state
```
**EN:** This method implements `forward_prepare(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_prepare(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 575-584: Method `MiMoV2Attention.forward_core`
```python
    def forward_core(self, intermediate_state):
        hidden_states, forward_batch, inner_state = intermediate_state
        if inner_state is None:
            return hidden_states
        attn_output = self.attn(
            *inner_state,
            sinks=self.attention_sink_bias,
        )
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This method implements `forward_core(intermediate_state)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_core(intermediate_state)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 586-603: Method `MiMoV2Attention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.k_size, self.v_size], dim=-1)

        # [t, h, dr]
        q, k = self.rotary_emb(positions, q, k)
        # [t, h, d]

        if self.v_scale is not None:
            v = v * self.v_scale
        attn_output = self.attn(q, k, v, forward_batch, sinks=self.attention_sink_bias)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 606-606: Class `MiMoV2DecoderLayer` overview
```python
class MiMoV2DecoderLayer(nn.Module):
```
**EN:** Defines `MiMoV2DecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 8 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoV2DecoderLayer`，其继承关系为 nn.Module。这个类组织了 8 个方法，用于实现模型相关行为。

### Lines 607-722: Method `MiMoV2DecoderLayer.__init__`
```python
    def __init__(
        self,
        config: MiMoV2Config,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.layer_id = layer_id

        rope_theta = getattr(config, "rope_theta", 10000)
        rope_scaling = getattr(config, "rope_scaling", None)
        # In v5, rope_scaling is a property alias for rope_parameters and returns
        # a standardized dict even when there's no actual scaling.  Treat the
        # "default" (no-op) type as None so factory.py uses plain RotaryEmbedding.
        if (
            isinstance(rope_scaling, dict)
            and rope_scaling.get("rope_type") == "default"
        ):
            rope_scaling = None
        max_position_embeddings = getattr(
            config,
            "context_len",
            getattr(config, "max_position_embeddings", 32768),
        )

        if self.is_swa_layer():
            self.self_attn = MiMoV2Attention(
                hidden_size=self.hidden_size,
                num_heads=config.swa_num_attention_heads,
                num_kv_heads=config.swa_num_key_value_heads,
                head_dim=config.swa_head_dim,
                v_head_dim=getattr(config, "swa_v_head_dim", None),
                v_scale=getattr(config, "attention_value_scale", None),
                sliding_window_size=config.sliding_window_size,
                attention_bias=config.attention_bias,
                attention_sink_bias=getattr(
                    config, "add_swa_attention_sink_bias", False
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 724-769: Method `MiMoV2DecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        # Self Attention
        hidden_states, residual = self.layer_communicator.prepare_attn(
            hidden_states, residual, forward_batch
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
        )

        hidden_states = self.mlp(
            hidden_states, forward_batch, should_allreduce_fusion, use_reduce_scatter
        )

        if should_allreduce_fusion:
            hidden_states._sglang_needs_allreduce_fusion = True
# ... truncated for brevity ...
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 771-777: Method `MiMoV2DecoderLayer.is_moe_layer`
```python
    def is_moe_layer(self, layer_idx: int) -> bool:
        return (
            hasattr(self.config, "moe_layer_freq")
            and 0 <= layer_idx < len(self.config.moe_layer_freq)
            and not isinstance(self.config.moe_layer_freq, int)
            and self.config.moe_layer_freq[layer_idx]
        )
```
**EN:** This method implements `is_moe_layer(layer_idx: ...)` and coordinates expert routing or mixture-of-experts computation.
**CN:** 这个方法实现了 `is_moe_layer(layer_idx: ...)`，其作用是协调专家路由或混合专家计算。

### Lines 779-780: Method `MiMoV2DecoderLayer.is_swa_layer`
```python
    def is_swa_layer(self) -> bool:
        return self.config.hybrid_layer_pattern[self.layer_id] == 1
```
**EN:** This method implements `is_swa_layer()` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2DecoderLayer`.
**CN:** 这个方法实现了 `is_swa_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2DecoderLayer` 内部调用。

### Lines 782-800: Method `MiMoV2DecoderLayer.op_comm_prepare_attn`
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

### Lines 802-809: Method `MiMoV2DecoderLayer.op_comm_prepare_mlp`
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

### Lines 811-813: Method `MiMoV2DecoderLayer.op_mlp`
```python
    def op_mlp(self, state):
        hidden_states = state.pop("hidden_states_mlp_input")
        state.hidden_states_mlp_output = self.mlp(hidden_states, state.forward_batch)
```
**EN:** This method implements `op_mlp(state)` and applies the feed-forward transformation used inside the network block.
**CN:** 这个方法实现了 `op_mlp(state)`，其作用是执行网络块中的前馈变换。

### Lines 815-837: Method `MiMoV2DecoderLayer.op_comm_postprocess_layer`
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
**EN:** This method implements `op_comm_postprocess_layer(state)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2DecoderLayer`.
**CN:** 这个方法实现了 `op_comm_postprocess_layer(state)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2DecoderLayer` 内部调用。

### Lines 840-840: Class `MiMoV2Model` overview
```python
class MiMoV2Model(nn.Module):
```
**EN:** Defines `MiMoV2Model` as a reusable runtime type derived from nn.Module. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoV2Model`，其继承关系为 nn.Module。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 841-882: Method `MiMoV2Model.__init__`
```python
    def __init__(
        self,
        config: MiMoV2Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        decoder_layer_type: type[nn.Module] = MiMoV2DecoderLayer,
    ) -> None:
        super().__init__()
        self.config = config
        self.padding_idx = getattr(config, "pad_token_id", None)
        self.vocab_size = config.vocab_size
        self.pp_group = get_pp_group()

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                quant_config=quant_config,
                use_attn_tp_group=is_dp_attention_enabled(),
                prefix=add_prefix("embed_tokens", prefix),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        # Use the provided decoder layer type or default to MiMoV2DecoderLayer
        decoder_layer_type = decoder_layer_type or MiMoV2DecoderLayer
        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            layer_fn=lambda idx, prefix: decoder_layer_type(
                layer_id=idx,
                config=config,
                quant_config=quant_config,
                prefix=prefix,
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix=add_prefix("layers", prefix),
        )
        if self.pp_group.is_last_rank:
            self.norm = RMSNorm(config.hidden_size, eps=config.layernorm_epsilon)
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., decoder_layer_type: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., decoder_layer_type: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 884-888: Method `MiMoV2Model.get_input_embedding`
```python
    def get_input_embedding(self, input_ids: torch.Tensor) -> torch.Tensor:
        if hasattr(self.config, "scale_emb"):
            return self.get_input_embeddings()(input_ids) * self.config.scale_emb
        else:
            return self.get_input_embeddings()(input_ids)
```
**EN:** This method implements `get_input_embedding(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embedding(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 890-891: Method `MiMoV2Model.get_input_embeddings`
```python
    def get_input_embeddings(self) -> nn.Embedding:
        return self.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 893-968: Method `MiMoV2Model.forward`
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

        if forward_batch.can_run_tbo:
            tbo_start_layer = self.start_layer
            tbo_end_layer = self.end_layer

            # skip first layer for TBO when starting from layer 0
            if self.start_layer == 0:
                layer = self.layers[0]
                hidden_states, residual = layer(
                    positions, hidden_states, forward_batch, residual
                )
                tbo_start_layer = tbo_start_layer + 1

            hidden_states, residual = model_forward_maybe_tbo(
                layers=self.layers[tbo_start_layer:tbo_end_layer],
                enable_tbo=True,
                input_data_scatter_mode=(
                    ScatterMode.model_input_output()
                    if tbo_start_layer == self.start_layer
                    else self.layers[
                        tbo_start_layer - 1
                    ].layer_scatter_modes.layer_output_mode
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 973-991: Method `MiMoV2Model.load_kv_cache_scales`
```python
    def load_kv_cache_scales(self, quantization_param_path: str) -> None:
        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()
        for layer_idx, scaling_factor in kv_cache_scales_loader(
            quantization_param_path,
            attn_tp_rank,
            attn_tp_size,
            self.config.num_hidden_layers,
            self.config.__class__.model_type,
        ):
            if not isinstance(self.layers[layer_idx], nn.Identity):
                layer_self_attn = self.layers[layer_idx].self_attn
            if hasattr(layer_self_attn.attn, "k_scale"):
                layer_self_attn.attn.k_scale = scaling_factor
                layer_self_attn.attn.v_scale = scaling_factor
            else:
                raise RuntimeError(
                    "Self attention has no KV cache scaling " "factor attribute!"
                )
```
**EN:** This method implements `load_kv_cache_scales(quantization_param_path: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2Model`.
**CN:** 这个方法实现了 `load_kv_cache_scales(quantization_param_path: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2Model` 内部调用。

### Lines 994-995: Class `MiMoV2ForCausalLM` overview
```python
class MiMoV2ForCausalLM(nn.Module):
    # BitandBytes specific attributes
```
**EN:** Defines `MiMoV2ForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 18 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoV2ForCausalLM`，其继承关系为 nn.Module。这个类组织了 18 个方法，用于实现模型相关行为。

### Lines 996-1017: Class `MiMoV2ForCausalLM` attributes
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

    # Prefixes for weight routing in encoder_only/language_only modes
    _LANGUAGE_WEIGHT_PREFIXES = ("model.", "lm_head.")
    _VISION_AUDIO_WEIGHT_PREFIXES = ("visual.", "vision_model.", "audio_")
    _VISION_AUDIO_WEIGHT_SUBSTRING = "speech_embeddings"
```
**EN:** Defines class-level attributes and metadata that shape how `MiMoV2ForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `MiMoV2ForCausalLM` 在运行时的行为。

### Lines 1019-1084: Method `MiMoV2ForCausalLM.__init__`
```python
    def __init__(
        self,
        config: MiMoV2Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        self._encoder_processor = None  # lazy-created in preprocess_mm_for_encoder

        if not self.config.encoder_only:
            self.model = MiMoV2Model(
                config, quant_config=quant_config, prefix=add_prefix("model", prefix)
            )

            if self.pp_group.is_last_rank:
                self.lm_head = ParallelLMHead(
                    config.vocab_size,
                    config.hidden_size,
                    quant_config=quant_config,
                    prefix=add_prefix("lm_head", prefix),
                    use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
                )
            else:
                self.lm_head = PPMissingLayer()
        else:
            self.model = None
            self.lm_head = None

        self.logits_processor = (
            LogitsProcessor(config) if not self.config.encoder_only else None
        )

        vision_config = getattr(config, "vision_config", None)
        audio_config = getattr(config, "audio_config", None)
        self._is_multimodal = vision_config is not None and audio_config is not None
        # Always build vision/audio encoders so P can fall back to local
        # encoding when the EPD encoder is unreachable.
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1086-1088: Method `MiMoV2ForCausalLM.routed_experts_weights_of_layer`
```python
    @property
    def routed_experts_weights_of_layer(self):
        return self._routed_experts_weights_of_layer.value
```
**EN:** This method implements `routed_experts_weights_of_layer()` and handles weight mapping, filtering, or loading for this model component Decorators: property.
**CN:** 这个方法实现了 `routed_experts_weights_of_layer()`，其作用是处理该模型组件的权重映射、筛选或加载逻辑 装饰器：property。

### Lines 1090-1094: Method `MiMoV2ForCausalLM.get_input_embedding`
```python
    def get_input_embedding(self, input_ids: torch.Tensor) -> torch.Tensor:
        assert (
            self.model is not None
        ), "get_input_embedding() is not available in encoder_only mode"
        return self.model.get_input_embedding(input_ids)
```
**EN:** This method implements `get_input_embedding(input_ids: ...)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embedding(input_ids: ...)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1096-1098: Method `MiMoV2ForCausalLM.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], mm_inputs: MultimodalInputs):
        pattern = MultiModalityDataPaddingPatternMultimodalTokens()
        return pattern.pad_input_tokens(input_ids, mm_inputs)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., mm_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2ForCausalLM`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., mm_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2ForCausalLM` 内部调用。

### Lines 1100-1107: Method `MiMoV2ForCausalLM.preprocess_mm_for_encoder`
```python
    def preprocess_mm_for_encoder(self, mm_data, modality, config):
        if self._encoder_processor is None:
            from sglang.srt.multimodal.processors.mimo_v2 import MiMoProcessor

            self._encoder_processor = MiMoProcessor.from_hf_config(
                self.config, mm_config=config
            )
        return self._encoder_processor.preprocess_for_encoder(mm_data, modality)
```
**EN:** This method implements `preprocess_mm_for_encoder(mm_data, modality, config)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2ForCausalLM`.
**CN:** 这个方法实现了 `preprocess_mm_for_encoder(mm_data, modality, config)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2ForCausalLM` 内部调用。

### Lines 1109-1116: Method `MiMoV2ForCausalLM.get_image_feature`
```python
    def get_image_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        pixel_values = torch.cat([item.feature for item in items], dim=0).type(
            self.visual.dtype
        )
        image_grid_thw = torch.cat([item.image_grid_thw for item in items], dim=0)
        assert pixel_values.dim() == 2, pixel_values.dim()
        assert image_grid_thw.dim() == 2, image_grid_thw.dim()
        return self.visual(pixel_values, grid_thw=image_grid_thw)
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1118-1125: Method `MiMoV2ForCausalLM.get_video_feature`
```python
    def get_video_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        pixel_values = torch.cat([item.feature for item in items], dim=0).type(
            self.visual.dtype
        )
        video_grid_thw = torch.cat([item.video_grid_thw for item in items], dim=0)
        assert pixel_values.dim() == 2, pixel_values.dim()
        assert video_grid_thw.dim() == 2, video_grid_thw.dim()
        return self.visual(pixel_values, grid_thw=video_grid_thw)
```
**EN:** This method implements `get_video_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_video_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1127-1128: Method `MiMoV2ForCausalLM.get_audio_feature`
```python
    def get_audio_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        return self.audio_encoder.get_audio_feature(items)
```
**EN:** This method implements `get_audio_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_audio_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1130-1196: Method `MiMoV2ForCausalLM.encode_video_audio`
```python
    @torch.inference_mode()
    def encode_video_audio(self, mm_inputs: Dict) -> Optional[torch.Tensor]:
        # EPD-side hook: encode audio tracks pulled from videos and trim to the
        # interleaved per-video segments produced by MiMoProcessor (segment
        # starts / lens / per_video_num_units). Returns None if there is no
        # audio to encode. The server passes the result through to the receiver
        # under aux_data["video_audio_embedding"].
        import numpy as np

        audio_features = mm_inputs.get("video_audio_features")
        if not audio_features:
            return None

        def _as_tensor(data):
            if isinstance(data, torch.Tensor):
                return data
            if isinstance(data, np.ndarray):
                return torch.tensor(data)
            if isinstance(data, list) and data and isinstance(data[0], np.ndarray):
                return torch.tensor(np.array(data))
            if isinstance(data, list) and data and isinstance(data[0], (int, float)):
                return torch.tensor(data)
            return data

        audio_feature_lens = mm_inputs["video_audio_feature_lens"]
        audio_item = MultimodalDataItem.from_dict(
            {
                "modality": Modality.AUDIO,
                "feature": _as_tensor(audio_features),
            }
        )
        audio_item.set("audio_feature_lens", _as_tensor(audio_feature_lens))

        audio_embedding = self.get_audio_feature([audio_item]).cpu()
        if audio_embedding.ndim != 2:
            audio_embedding = audio_embedding.reshape(-1, audio_embedding.shape[-1])

        segment_lens_flat = mm_inputs["video_audio_segment_lens_flat"]
        segment_starts_flat = mm_inputs["video_audio_segment_starts_flat"]
        per_video_num_units = mm_inputs["video_audio_per_video_num_units"]
# ... truncated for brevity ...
```
**EN:** This method implements `encode_video_audio(mm_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2ForCausalLM` Decorators: torch.inference_mode().
**CN:** 这个方法实现了 `encode_video_audio(mm_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2ForCausalLM` 内部调用 装饰器：torch.inference_mode()。

### Lines 1198-1199: Method `MiMoV2ForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self) -> Optional[nn.Embedding]:
        return self.model.embed_tokens if self.model is not None else None
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1201-1241: Method `MiMoV2ForCausalLM.forward`
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
        assert (
            not self.config.encoder_only
        ), "forward() should not be called in encoder_only mode"

        if self._is_multimodal:
            hidden_states, hidden_states_before_norm = general_mm_embed_routine(
                input_ids=input_ids,
                forward_batch=forward_batch,
                language_model=self.model,
                multimodal_model=self,
                positions=positions,
                pp_proxy_tensors=pp_proxy_tensors,
            )
        else:
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
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 1243-1245: Method `MiMoV2ForCausalLM.start_layer`
```python
    @property
    def start_layer(self):
        return self.model.start_layer if self.model is not None else 0
```
**EN:** This method implements `start_layer()` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2ForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `start_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2ForCausalLM` 内部调用 装饰器：property。

### Lines 1247-1249: Method `MiMoV2ForCausalLM.end_layer`
```python
    @property
    def end_layer(self):
        return self.model.end_layer if self.model is not None else 0
```
**EN:** This method implements `end_layer()` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2ForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `end_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2ForCausalLM` 内部调用 装饰器：property。

### Lines 1251-1487: Method `MiMoV2ForCausalLM.load_weights`
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
        stacked_params_mapping_vit = [
            # (param_name, shard_name, shard_id)
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        # (param_name, weight_name, expert_id, shard_id)
        expert_params_mapping = DeepEPMoE.make_expert_params_mapping(
            ckpt_gate_proj_name="gate_proj",
            ckpt_down_proj_name="down_proj",
            ckpt_up_proj_name="up_proj",
            num_experts=self.config.n_routed_experts,
        )

        params_dict = dict(self.named_parameters())
        skipped_mtp_weights = False

        def _is_vision_audio_weight(name):
            return (
                name.startswith(self._VISION_AUDIO_WEIGHT_PREFIXES)
                or self._VISION_AUDIO_WEIGHT_SUBSTRING in name
            )

        for name, loaded_weight in weights:
            if not self._is_multimodal and _is_vision_audio_weight(name):
                continue

            if self.config.encoder_only and name.startswith(
                self._LANGUAGE_WEIGHT_PREFIXES
            ):
                continue
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1489-1493: Method `MiMoV2ForCausalLM.get_embed_and_head`
```python
    def get_embed_and_head(self):
        assert (
            self.model is not None and self.lm_head is not None
        ), "get_embed_and_head() is not available in encoder_only mode"
        return self.model.embed_tokens.weight, self.lm_head.weight
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1495-1504: Method `MiMoV2ForCausalLM.set_embed_and_head`
```python
    def set_embed_and_head(self, embed, head):
        assert (
            self.model is not None and self.lm_head is not None
        ), "set_embed_and_head() is not available in encoder_only mode"
        del self.model.embed_tokens.weight
        del self.lm_head.weight
        self.model.embed_tokens.weight = embed
        self.lm_head.weight = head
        torch.cuda.empty_cache()
        torch.cuda.synchronize()
```
**EN:** This method implements `set_embed_and_head(embed, head)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_embed_and_head(embed, head)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1506-1508: Method `MiMoV2ForCausalLM.load_kv_cache_scales`
```python
    def load_kv_cache_scales(self, quantization_param_path: str) -> None:
        if self.model is not None:
            self.model.load_kv_cache_scales(quantization_param_path)
```
**EN:** This method implements `load_kv_cache_scales(quantization_param_path: ...)` and implements a focused helper that supports the surrounding runtime flow inside `MiMoV2ForCausalLM`.
**CN:** 这个方法实现了 `load_kv_cache_scales(quantization_param_path: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `MiMoV2ForCausalLM` 内部调用。

### Lines 1510-1516: Method `MiMoV2ForCausalLM.get_model_config_for_expert_location`
```python
    @classmethod
    def get_model_config_for_expert_location(cls, config):
        return ModelConfigForExpertLocation(
            num_layers=config.num_hidden_layers,
            num_logical_experts=getattr(config, "n_routed_experts", 1),
            num_groups=getattr(config, "n_group", None),
        )
```
**EN:** This method implements `get_model_config_for_expert_location(config)` and returns a cached or derived object used elsewhere in the pipeline Decorators: classmethod.
**CN:** 这个方法实现了 `get_model_config_for_expert_location(config)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果 装饰器：classmethod。

### Lines 1520-1520: Class `MiMoV2FlashForCausalLM` overview
```python
class MiMoV2FlashForCausalLM(MiMoV2ForCausalLM):
```
**EN:** Defines `MiMoV2FlashForCausalLM` as a reusable runtime type derived from MiMoV2ForCausalLM. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `MiMoV2FlashForCausalLM`，其继承关系为 MiMoV2ForCausalLM。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 1521-1521: Class `MiMoV2FlashForCausalLM` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 1524-1524: Top-level assign
```python
EntryClass = [MiMoV2ForCausalLM, MiMoV2FlashForCausalLM]
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
- `sglang.srt.batch_overlap.two_batch_overlap.model_forward_maybe_tbo`
- `sglang.srt.configs.model_config.get_mimo_v2_fused_qkv_expected_tp_size`
- `sglang.srt.distributed.get_moe_expert_parallel_world_size`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.distributed.tensor_model_parallel_all_reduce`
- `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location.ModelConfigForExpertLocation`
- `sglang.srt.eplb.expert_location_dispatch.ExpertLocationDispatchInfo`
- `sglang.srt.layers.activation.SiluAndMul`
- `sglang.srt.layers.communicator.LayerCommunicator`
- `sglang.srt.layers.communicator.LayerScatterModes`
- `sglang.srt.layers.communicator.ScatterMode`
- `sglang.srt.layers.communicator.enable_moe_dense_fully_dp`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.MergedColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.moe.get_moe_a2a_backend`
- `sglang.srt.layers.moe.get_moe_runner_backend`
- `sglang.srt.layers.moe.should_skip_post_experts_all_reduce`
- `sglang.srt.layers.moe.ep_moe.layer.DeepEPMoE`
- `sglang.srt.layers.moe.ep_moe.layer.get_moe_impl_class`
- `sglang.srt.layers.moe.topk.TopK`
- `sglang.srt.layers.moe.topk.TopKOutputFormat`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.utils.get_layer_id`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.managers.mm_utils.MultiModalityDataPaddingPatternMultimodalTokens`
- `sglang.srt.managers.mm_utils.general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch.Modality`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.model_loader.weight_utils.kv_cache_scales_loader`
- `sglang.srt.models.mimo_audio.MiMoAudioEncoder`
- `sglang.srt.models.mimo_audio.MiMoAudioEncoderConfig`
- `sglang.srt.models.mimo_vl.MiMoVisionTransformer`
- `sglang.srt.models.mimo_vl.MiMoVLVisionConfig`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.LazyValue`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.is_non_idle_and_non_empty`
- `sglang.srt.utils.make_layers`
- `numpy`
- `sglang.srt.multimodal.processors.mimo_v2.MiMoProcessor`
