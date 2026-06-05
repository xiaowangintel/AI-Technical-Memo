# sarvam_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/sarvam_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only Sarvam MoE models for SGLang. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4: Module docstring
```python
"""Inference-only Sarvam MoE models for SGLang.
- SarvamMLAForCausalLM (105B)
- SarvamMoEForCausalLM (30B)
"""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 6-71: Module imports
```python
import math
from enum import IntEnum, auto
from typing import Any, Dict, Iterable, Optional, Tuple

import torch
import torch.nn.functional as F
from torch import nn
from transformers import PretrainedConfig

from sglang.srt.distributed import (
    get_pp_group,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.eplb.expert_location import ModelConfigForExpertLocation
from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.attention.utils import concat_and_cast_mha_k_triton
from sglang.srt.layers.communicator import (
    LayerCommunicator,
    LayerScatterModes,
    enable_moe_dense_fully_dp,
)
from sglang.srt.layers.dp_attention import (
    get_attention_tp_rank,
    get_attention_tp_size,
    is_dp_attention_enabled,
)
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor, LogitsProcessorOutput
from sglang.srt.layers.moe import should_skip_post_experts_all_reduce
from sglang.srt.layers.moe.ep_moe.layer import get_moe_impl_class
from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE
from sglang.srt.layers.moe.topk import TopK
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 73-73: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 74-74: Top-level assign
```python
_is_cublas_ge_129 = is_nvidia_cublas_version_ge_12_9()
```
**EN:** Defines or updates _is_cublas_ge_129, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cublas_ge_129，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 76-97: Top-level if
```python
if _is_cuda:
    try:
        from sgl_kernel import bmm_fp8, concat_mla_k, merge_state_v2

        from sglang.srt.layers.quantization.fp8_kernel import per_tensor_quant_mla_fp8

        _has_fp8_support = True
        _has_concat_mla_k = True
    except ImportError:
        _has_fp8_support = False
        _has_concat_mla_k = False
        bmm_fp8 = None
        concat_mla_k = None
        merge_state_v2 = None
        per_tensor_quant_mla_fp8 = None
else:
    _has_fp8_support = False
    _has_concat_mla_k = False
    bmm_fp8 = None
    concat_mla_k = None
    merge_state_v2 = None
    per_tensor_quant_mla_fp8 = None
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 100-100: Class `AttnForwardMethod` overview
```python
class AttnForwardMethod(IntEnum):
```
**EN:** Defines `AttnForwardMethod` as a reusable runtime type derived from IntEnum. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `AttnForwardMethod`，其继承关系为 IntEnum。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 101-103: Class `AttnForwardMethod` attributes
```python
    MLA_SEPARATE_ROPE = auto()
    MLA_CONCAT_ROPE = auto()
    MHA_PREFILL = auto()
```
**EN:** Defines class-level attributes and metadata that shape how `AttnForwardMethod` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `AttnForwardMethod` 在运行时的行为。

### Lines 106-108: Top-level assign
```python
SEPARATE_ROPE_BACKENDS = frozenset(
    ["fa3", "flashinfer", "nsa", "cutlass_mla", "trtllm_mla"]
)
```
**EN:** Defines or updates SEPARATE_ROPE_BACKENDS, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 SEPARATE_ROPE_BACKENDS，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 109-109: Top-level assign
```python
CONCAT_ROPE_BACKENDS = frozenset(["flashmla", "triton"])
```
**EN:** Defines or updates CONCAT_ROPE_BACKENDS, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 CONCAT_ROPE_BACKENDS，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 112-112: Class `AttentionBackendRegistry` overview
```python
class AttentionBackendRegistry:
```
**EN:** Defines `AttentionBackendRegistry` as a reusable runtime type derived from no explicit base class. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `AttentionBackendRegistry`，其继承关系为 no explicit base class。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 113-113: Class `AttentionBackendRegistry` attributes
```python
    _handlers = {}
```
**EN:** Defines class-level attributes and metadata that shape how `AttentionBackendRegistry` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `AttentionBackendRegistry` 在运行时的行为。

### Lines 115-117: Method `AttentionBackendRegistry.register`
```python
    @classmethod
    def register(cls, backend_name: str, handler_func):
        cls._handlers[backend_name] = handler_func
```
**EN:** This method implements `register(backend_name: ..., handler_func)` and registers metadata or implementation classes into a global lookup table Decorators: classmethod.
**CN:** 这个方法实现了 `register(backend_name: ..., handler_func)`，其作用是把元数据或实现类注册到全局查找表中 装饰器：classmethod。

### Lines 119-121: Method `AttentionBackendRegistry.get_handler`
```python
    @classmethod
    def get_handler(cls, backend_name: str):
        return cls._handlers.get(backend_name, cls._default_handler)
```
**EN:** This method implements `get_handler(backend_name: ...)` and returns a cached or derived object used elsewhere in the pipeline Decorators: classmethod.
**CN:** 这个方法实现了 `get_handler(backend_name: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果 装饰器：classmethod。

### Lines 123-125: Method `AttentionBackendRegistry._default_handler`
```python
    @classmethod
    def _default_handler(cls, attn, forward_batch) -> AttnForwardMethod:
        return AttnForwardMethod.MLA_CONCAT_ROPE
```
**EN:** This method implements `_default_handler(attn, forward_batch)` and implements a focused helper that supports the surrounding runtime flow inside `AttentionBackendRegistry` Decorators: classmethod.
**CN:** 这个方法实现了 `_default_handler(attn, forward_batch)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `AttentionBackendRegistry` 内部调用 装饰器：classmethod。

### Lines 127-132: Method `AttentionBackendRegistry.get_forward_method`
```python
    @classmethod
    def get_forward_method(
        cls, backend_name: str, attn, forward_batch
    ) -> AttnForwardMethod:
        handler = cls.get_handler(backend_name)
        return handler(attn, forward_batch)
```
**EN:** This method implements `get_forward_method(backend_name: ..., attn, forward_batch)` and runs the main forward computation and returns tensors for the next inference stage Decorators: classmethod.
**CN:** 这个方法实现了 `get_forward_method(backend_name: ..., attn, forward_batch)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：classmethod。

### Lines 135-136: Function `_handle_separate_rope_backend`
```python
def _handle_separate_rope_backend(attn, forward_batch) -> AttnForwardMethod:
    return AttnForwardMethod.MLA_SEPARATE_ROPE
```
**EN:** This function implements `_handle_separate_rope_backend(attn, forward_batch)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `_handle_separate_rope_backend(attn, forward_batch)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 139-140: Function `_handle_concat_rope_backend`
```python
def _handle_concat_rope_backend(attn, forward_batch) -> AttnForwardMethod:
    return AttnForwardMethod.MLA_CONCAT_ROPE
```
**EN:** This function implements `_handle_concat_rope_backend(attn, forward_batch)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `_handle_concat_rope_backend(attn, forward_batch)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 143-144: Top-level for
```python
for backend in SEPARATE_ROPE_BACKENDS:
    AttentionBackendRegistry.register(backend, _handle_separate_rope_backend)
```
**EN:** Executes module-level control flow that prepares derived tables or registration data before the main classes are used.
**CN:** 执行模块级控制流，在主类被使用前准备派生表或注册数据。

### Lines 145-146: Top-level for
```python
for backend in CONCAT_ROPE_BACKENDS:
    AttentionBackendRegistry.register(backend, _handle_concat_rope_backend)
```
**EN:** Executes module-level control flow that prepares derived tables or registration data before the main classes are used.
**CN:** 执行模块级控制流，在主类被使用前准备派生表或注册数据。

### Lines 149-160: Function `get_attn_forward_method`
```python
def get_attn_forward_method(server_args, forward_batch) -> AttnForwardMethod:
    is_decode = forward_batch.forward_mode.is_decode_or_idle()
    if is_decode:
        backend = server_args.decode_attention_backend or server_args.attention_backend
    else:
        backend = server_args.prefill_attention_backend or server_args.attention_backend
        if (
            forward_batch.forward_mode.is_extend_without_speculative()
            and backend == "fa3"
        ):
            return AttnForwardMethod.MHA_PREFILL
    return AttentionBackendRegistry.get_forward_method(backend, None, forward_batch)
```
**EN:** This function implements `get_attn_forward_method(server_args, forward_batch)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个函数实现了 `get_attn_forward_method(server_args, forward_batch)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 163-163: Class `SarvamMoEMLP` overview
```python
class SarvamMoEMLP(nn.Module):
```
**EN:** Defines `SarvamMoEMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SarvamMoEMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 164-199: Method `SarvamMoEMLP.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        reduce_results: bool = True,
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
            prefix=add_prefix("down_proj", prefix),
            reduce_results=reduce_results,
            tp_rank=tp_rank,
            tp_size=tp_size,
        )
        if hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. Only silu is supported."
            )
        self.act_fn = SiluAndMul()
```
**EN:** This method implements `__init__(hidden_size: ..., intermediate_size: ..., hidden_act: ..., quant_config: ...=..., prefix: ...=..., reduce_results: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., intermediate_size: ..., hidden_act: ..., quant_config: ...=..., prefix: ...=..., reduce_results: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 201-215: Method `SarvamMoEMLP.forward`
```python
    def forward(
        self,
        x,
        forward_batch: ForwardBatch = None,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
    ):
        if x.shape[0] == 0:
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

### Lines 218-218: Class `SarvamMoESparseMoeBlock` overview
```python
class SarvamMoESparseMoeBlock(nn.Module):
```
**EN:** Defines `SarvamMoESparseMoeBlock` as a reusable runtime type derived from nn.Module. The class groups 7 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SarvamMoESparseMoeBlock`，其继承关系为 nn.Module。这个类组织了 7 个方法，用于实现模型相关行为。

### Lines 219-310: Method `SarvamMoESparseMoeBlock.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ):
        super().__init__()
        self.config = config
        self.layer_id = layer_id
        self.tp_size = get_tensor_model_parallel_world_size()
        self.routed_scaling_factor = getattr(config, "routed_scaling_factor", 2.5)
        self.score_function = getattr(config, "score_function", "sigmoid")
        self.n_group = getattr(config, "n_group", None)
        self.topk_group = getattr(config, "topk_group", None)
        self.alt_stream = alt_stream

        dtype_map = {
            "fp32": torch.float32,
            "bf16": torch.bfloat16,
            "bfloat16": torch.bfloat16,
        }
        router_dtype_cfg = getattr(config, "router_dtype", "fp32")
        self.router_dtype = dtype_map.get(router_dtype_cfg, None)

        if self.tp_size > config.num_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {config.num_experts}."
            )

        self.e_score_correction_bias = nn.Parameter(
            torch.zeros(config.num_experts, dtype=torch.float32),
            requires_grad=False,
        )

        self.topk = TopK(
            top_k=config.num_experts_per_tok,
            use_grouped_topk=self.n_group is not None and self.topk_group is not None,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 312-334: Method `SarvamMoESparseMoeBlock.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        forward_batch: Optional[ForwardBatch] = None,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
        gemm_output_zero_allocator: Optional[BumpAllocator] = None,
    ) -> torch.Tensor:
        del gemm_output_zero_allocator

        if (
            self.shared_experts is not None
            and self.alt_stream is not None
            and hidden_states.shape[0] > 0
            and get_is_capture_mode()
        ):
            return self.forward_normal_dual_stream(
                hidden_states, should_allreduce_fusion, use_reduce_scatter
            )
        else:
            return self.forward_normal(
                hidden_states, should_allreduce_fusion, use_reduce_scatter
            )
```
**EN:** This method implements `forward(hidden_states: ..., forward_batch: ...=..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=..., gemm_output_zero_allocator: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., forward_batch: ...=..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=..., gemm_output_zero_allocator: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 336-341: Method `SarvamMoESparseMoeBlock.get_moe_weights`
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

### Lines 343-344: Method `SarvamMoESparseMoeBlock._forward_shared_experts`
```python
    def _forward_shared_experts(self, hidden_states: torch.Tensor) -> torch.Tensor:
        return self.shared_experts(hidden_states)
```
**EN:** This method implements `_forward_shared_experts(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_shared_experts(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 346-355: Method `SarvamMoESparseMoeBlock._forward_router_experts`
```python
    def _forward_router_experts(self, hidden_states: torch.Tensor) -> torch.Tensor:
        if self.router_dtype is not None:
            router_logits = F.linear(
                hidden_states.to(self.router_dtype),
                self.gate.weight.to(self.router_dtype),
            )
        else:
            router_logits, _ = self.gate(hidden_states)
        topk_output = self.topk(hidden_states, router_logits)
        return self.experts(hidden_states, topk_output)
```
**EN:** This method implements `_forward_router_experts(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_router_experts(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 357-379: Method `SarvamMoESparseMoeBlock.forward_normal_dual_stream`
```python
    def forward_normal_dual_stream(
        self,
        hidden_states: torch.Tensor,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
    ) -> torch.Tensor:
        num_tokens, hidden_dim = hidden_states.shape
        current_stream = torch.cuda.current_stream()
        self.alt_stream.wait_stream(current_stream)
        shared_out = self._forward_shared_experts(hidden_states)
        with torch.cuda.stream(self.alt_stream):
            final_hidden_states = self._forward_router_experts(hidden_states)
            if self.routed_scaling_factor != 1.0:
                final_hidden_states = final_hidden_states * self.routed_scaling_factor
        current_stream.wait_stream(self.alt_stream)
        final_hidden_states = final_hidden_states + shared_out
        if self.tp_size > 1 and not should_skip_post_experts_all_reduce(
            is_tp_path=True,
            use_reduce_scatter=use_reduce_scatter,
            should_allreduce_fusion=should_allreduce_fusion,
        ):
            final_hidden_states = tensor_model_parallel_all_reduce(final_hidden_states)
        return final_hidden_states.view(num_tokens, hidden_dim)
```
**EN:** This method implements `forward_normal_dual_stream(hidden_states: ..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_normal_dual_stream(hidden_states: ..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 381-422: Method `SarvamMoESparseMoeBlock.forward_normal`
```python
    def forward_normal(
        self,
        hidden_states: torch.Tensor,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
    ) -> torch.Tensor:
        if hidden_states.shape[0] == 0:
            return hidden_states

        num_tokens, hidden_dim = hidden_states.shape
        identity = (
            hidden_states.clone() if self.shared_experts is not None else hidden_states
        )

        if self.router_dtype is not None:
            router_logits = F.linear(
                hidden_states.to(self.router_dtype),
                self.gate.weight.to(self.router_dtype),
            )
        else:
            router_logits, _ = self.gate(hidden_states)
        topk_output = self.topk(hidden_states, router_logits)
        final_hidden_states = self.experts(hidden_states, topk_output)

        if self.shared_experts is not None:
            shared_out = self.shared_experts(identity)
            if self.routed_scaling_factor != 1.0:
                shared_out.add_(final_hidden_states, alpha=self.routed_scaling_factor)
            else:
                shared_out.add_(final_hidden_states)
            final_hidden_states = shared_out
        elif self.routed_scaling_factor != 1.0:
            final_hidden_states = final_hidden_states * self.routed_scaling_factor

        if self.tp_size > 1 and not should_skip_post_experts_all_reduce(
            is_tp_path=True,
            use_reduce_scatter=use_reduce_scatter,
            should_allreduce_fusion=should_allreduce_fusion,
        ):
            final_hidden_states = tensor_model_parallel_all_reduce(final_hidden_states)
# ... truncated for brevity ...
```
**EN:** This method implements `forward_normal(hidden_states: ..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_normal(hidden_states: ..., should_allreduce_fusion: ...=..., use_reduce_scatter: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 425-425: Class `SarvamMoEMLAAttention` overview
```python
class SarvamMoEMLAAttention(nn.Module):
```
**EN:** Defines `SarvamMoEMLAAttention` as a reusable runtime type derived from nn.Module. The class groups 12 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SarvamMoEMLAAttention`，其继承关系为 nn.Module。这个类组织了 12 个方法，用于实现模型相关行为。

### Lines 426-571: Method `SarvamMoEMLAAttention.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        hidden_size: int,
        num_heads: int,
        layer_id: int = 0,
        rope_theta: float = 10000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        max_position_embeddings: int = 8192,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = hidden_size
        self.layer_id = layer_id
        self.alt_stream = alt_stream
        self.quant_config = quant_config

        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()

        self.qk_nope_head_dim = config.qk_nope_head_dim
        self.qk_rope_head_dim = config.qk_rope_head_dim
        self.qk_head_dim = config.qk_nope_head_dim + config.qk_rope_head_dim
        self.v_head_dim = config.v_head_dim
        self.q_lora_rank = getattr(config, "q_lora_rank", None)
        self.kv_lora_rank = config.kv_lora_rank

        self.num_heads = num_heads
        assert num_heads % attn_tp_size == 0
        self.num_local_heads = num_heads // attn_tp_size

        self.scaling = self.qk_head_dim**-0.5
        self.rope_theta = rope_theta
        self.max_position_embeddings = max_position_embeddings
        self.kv_cache_dtype = get_global_server_args().kv_cache_dtype

        self._server_args = None
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., hidden_size: ..., num_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., hidden_size: ..., num_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 573-576: Method `SarvamMoEMLAAttention.yarn_get_mscale`
```python
    def yarn_get_mscale(self, scale: float = 1, mscale: float = 1) -> float:
        if scale <= 1:
            return 1.0
        return 0.1 * mscale * math.log(scale) + 1.0
```
**EN:** This method implements `yarn_get_mscale(scale: ...=..., mscale: ...=...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `yarn_get_mscale(scale: ...=..., mscale: ...=...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 578-617: Method `SarvamMoEMLAAttention._concat_and_cast_mha_k`
```python
    def _concat_and_cast_mha_k(
        self,
        k_nope: torch.Tensor,
        k_pe: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        k_shape = (k_nope.shape[0], self.num_local_heads, self.qk_head_dim)

        if (
            _is_cuda
            and _has_concat_mla_k
            and (self.num_local_heads == 128)
            and (self.qk_nope_head_dim == 128)
            and (self.qk_rope_head_dim == 64)
        ):
            k = k_nope.new_empty(*k_shape)
            concat_mla_k(k=k, k_nope=k_nope, k_rope=k_pe)
            return k

        if (
            _is_cuda
            and next_power_of_2(self.num_local_heads) == self.num_local_heads
            and next_power_of_2(self.qk_nope_head_dim) == self.qk_nope_head_dim
            and next_power_of_2(self.qk_rope_head_dim) == self.qk_rope_head_dim
        ):
            if (
                self.current_attention_backend == "fa3"
                and self.kv_cache_dtype != "auto"
            ):
                attn_dtype = forward_batch.token_to_kv_pool.dtype
            else:
                attn_dtype = k_nope.dtype
            k = k_nope.new_empty(*k_shape, dtype=attn_dtype)
            concat_and_cast_mha_k_triton(k, k_nope, k_pe)
            return k

        k = k_nope.new_empty(*k_shape)
        k[..., : self.qk_nope_head_dim] = k_nope
        k[..., self.qk_nope_head_dim :] = k_pe
        return k
```
**EN:** This method implements `_concat_and_cast_mha_k(k_nope: ..., k_pe: ..., forward_batch: ...)` and implements a focused helper that supports the surrounding runtime flow inside `SarvamMoEMLAAttention`.
**CN:** 这个方法实现了 `_concat_and_cast_mha_k(k_nope: ..., k_pe: ..., forward_batch: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SarvamMoEMLAAttention` 内部调用。

### Lines 619-631: Method `SarvamMoEMLAAttention._set_current_attention_backend`
```python
    def _set_current_attention_backend(self, forward_batch: ForwardBatch) -> None:
        if self._server_args is None:
            self._server_args = get_global_server_args()
        if forward_batch.forward_mode.is_decode_or_idle():
            self.current_attention_backend = (
                self._server_args.decode_attention_backend
                or self._server_args.attention_backend
            )
        else:
            self.current_attention_backend = (
                self._server_args.prefill_attention_backend
                or self._server_args.attention_backend
            )
```
**EN:** This method implements `_set_current_attention_backend(forward_batch: ...)` and implements attention-related tensor preparation or execution.
**CN:** 这个方法实现了 `_set_current_attention_backend(forward_batch: ...)`，其作用是实现与注意力相关的张量准备或执行逻辑。

### Lines 633-659: Method `SarvamMoEMLAAttention._maybe_fp8_bmm`
```python
    def _maybe_fp8_bmm(
        self,
        x_bmk: torch.Tensor,
        w_bkn: torch.Tensor,
        zero_allocator: Optional[BumpAllocator] = None,
    ) -> torch.Tensor:
        if (
            _has_fp8_support
            and w_bkn is not None
            and w_bkn.dtype == torch.float8_e4m3fn
        ):
            x_val, x_scale = per_tensor_quant_mla_fp8(
                x_bmk,
                (
                    torch.zeros((1,), dtype=torch.float32, device=x_bmk.device)
                    if _is_cublas_ge_129
                    else (
                        zero_allocator.allocate(1)
                        if zero_allocator
                        else torch.zeros((1,), dtype=torch.float32, device=x_bmk.device)
                    )
                ),
            )
            w_scale = self.w_scale if self.w_scale is not None else 1.0
            return bmm_fp8(x_val, w_bkn, x_scale, w_scale, torch.bfloat16)

        return torch.bmm(x_bmk, w_bkn)
```
**EN:** This method implements `_maybe_fp8_bmm(x_bmk: ..., w_bkn: ..., zero_allocator: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `SarvamMoEMLAAttention`.
**CN:** 这个方法实现了 `_maybe_fp8_bmm(x_bmk: ..., w_bkn: ..., zero_allocator: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SarvamMoEMLAAttention` 内部调用。

### Lines 661-725: Method `SarvamMoEMLAAttention._run_mha_prefill`
```python
    def _run_mha_prefill(
        self,
        positions: torch.Tensor,
        q: torch.Tensor,
        q_pe: torch.Tensor,
        k_nope: torch.Tensor,
        k_pe: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:

        q_pe, k_pe = self.rotary_emb(positions, q_pe, k_pe)
        q[..., self.qk_nope_head_dim :] = q_pe

        forward_batch.token_to_kv_pool.set_mla_kv_buffer(
            self.attn_mha,
            forward_batch.out_cache_loc,
            k_nope,
            k_pe,
        )

        kv_a = k_nope.squeeze(1)
        kv_expanded, _ = self.kv_b_proj(kv_a)
        kv_expanded = kv_expanded.view(
            -1, self.num_local_heads, self.qk_nope_head_dim + self.v_head_dim
        )
        k_nope_expanded = kv_expanded[..., : self.qk_nope_head_dim]
        v = kv_expanded[..., self.qk_nope_head_dim :]

        k = self._concat_and_cast_mha_k(k_nope_expanded, k_pe, forward_batch)

        has_extend_prefix = forward_batch.extend_prefix_lens_cpu is not None and any(
            forward_batch.extend_prefix_lens_cpu
        )

        self._set_current_attention_backend(forward_batch)
        can_use_prefix_cache = not self._server_args.disable_radix_cache
        do_prefix_merge = has_extend_prefix and can_use_prefix_cache

        if do_prefix_merge and forward_batch.num_prefix_chunks is None:
            if hasattr(forward_batch, "prepare_chunked_prefix_cache_info"):
# ... truncated for brevity ...
```
**EN:** This method implements `_run_mha_prefill(positions: ..., q: ..., q_pe: ..., k_nope: ..., k_pe: ..., forward_batch: ...)` and implements a focused helper that supports the surrounding runtime flow inside `SarvamMoEMLAAttention`.
**CN:** 这个方法实现了 `_run_mha_prefill(positions: ..., q: ..., q_pe: ..., k_nope: ..., k_pe: ..., forward_batch: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SarvamMoEMLAAttention` 内部调用。

### Lines 727-736: Method `SarvamMoEMLAAttention._chunked_prefix_attn_mha`
```python
    def _chunked_prefix_attn_mha(
        self,
        q: torch.Tensor,
        accum_output: torch.Tensor,
        accum_lse: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        return DeepseekMHAForwardMixin._chunked_prefix_attn_mha(
            self, q, accum_output, accum_lse, forward_batch
        )
```
**EN:** This method implements `_chunked_prefix_attn_mha(q: ..., accum_output: ..., accum_lse: ..., forward_batch: ...)` and implements attention-related tensor preparation or execution.
**CN:** 这个方法实现了 `_chunked_prefix_attn_mha(q: ..., accum_output: ..., accum_lse: ..., forward_batch: ...)`，其作用是实现与注意力相关的张量准备或执行逻辑。

### Lines 738-746: Method `SarvamMoEMLAAttention._get_mla_kv_buffer`
```python
    def _get_mla_kv_buffer(
        self,
        kv_indices: torch.Tensor,
        dst_dtype: torch.dtype,
        forward_batch: ForwardBatch,
    ):
        return DeepseekMHAForwardMixin._get_mla_kv_buffer(
            self, kv_indices, dst_dtype, forward_batch
        )
```
**EN:** This method implements `_get_mla_kv_buffer(kv_indices: ..., dst_dtype: ..., forward_batch: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `_get_mla_kv_buffer(kv_indices: ..., dst_dtype: ..., forward_batch: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 748-842: Method `SarvamMoEMLAAttention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        zero_allocator: Optional[BumpAllocator] = None,
        llama_4_scaling: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        del llama_4_scaling
        if hidden_states.shape[0] == 0:
            return hidden_states

        if self.q_lora_rank is None:
            q, _ = self.q_proj(hidden_states)
            latent_cache, _ = self.kv_a_proj_with_mqa(hidden_states)
            k_nope = latent_cache[..., : self.kv_lora_rank]
            k_nope = self.kv_a_layernorm(k_nope).unsqueeze(1)
        else:
            q_a, _ = self.q_a_proj(hidden_states)
            q_a = self.q_a_layernorm(q_a)
            q, _ = self.q_b_proj(q_a)
            latent_cache, _ = self.kv_a_proj_with_mqa(hidden_states)
            k_nope = latent_cache[..., : self.kv_lora_rank]
            k_nope = self.kv_a_layernorm(k_nope).unsqueeze(1)

        q = q.view(-1, self.num_local_heads, self.qk_head_dim)
        q_nope, q_pe = q.split([self.qk_nope_head_dim, self.qk_rope_head_dim], dim=-1)
        k_pe = latent_cache[..., self.kv_lora_rank :].unsqueeze(1)

        if self._server_args is None:
            self._server_args = get_global_server_args()
        self._set_current_attention_backend(forward_batch)

        forward_method = get_attn_forward_method(self._server_args, forward_batch)

        if forward_method == AttnForwardMethod.MHA_PREFILL:
            return self._run_mha_prefill(
                positions=positions,
                q=q,
                q_pe=q_pe,
# ... truncated for brevity ...
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., zero_allocator: ...=..., llama_4_scaling: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., zero_allocator: ...=..., llama_4_scaling: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 844-934: Method `SarvamMoEMLAAttention.forward_prepare`
```python
    def forward_prepare(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        zero_allocator: Optional[BumpAllocator] = None,
        llama_4_scaling: Optional[torch.Tensor] = None,
    ) -> Tuple[Optional[torch.Tensor], ForwardBatch, Optional[Tuple]]:
        del llama_4_scaling
        if hidden_states.shape[0] == 0:
            return hidden_states, forward_batch, None

        if self.q_lora_rank is None:
            # Dual-stream parallel Q and KV projections
            if self.alt_stream is not None and get_is_capture_mode():
                current_stream = torch.cuda.current_stream()
                self.alt_stream.wait_stream(current_stream)
                with torch.cuda.stream(self.alt_stream):
                    latent_cache, _ = self.kv_a_proj_with_mqa(hidden_states)
                q, _ = self.q_proj(hidden_states)
                current_stream.wait_stream(self.alt_stream)
            else:
                q, _ = self.q_proj(hidden_states)
                latent_cache, _ = self.kv_a_proj_with_mqa(hidden_states)
            k_nope = latent_cache[..., : self.kv_lora_rank]
            k_nope = self.kv_a_layernorm(k_nope).unsqueeze(1)
        else:
            # For q_lora_rank path, overlap q_a_proj with kv_a_proj
            if self.alt_stream is not None and get_is_capture_mode():
                current_stream = torch.cuda.current_stream()
                self.alt_stream.wait_stream(current_stream)
                with torch.cuda.stream(self.alt_stream):
                    latent_cache, _ = self.kv_a_proj_with_mqa(hidden_states)
                q_a, _ = self.q_a_proj(hidden_states)
                current_stream.wait_stream(self.alt_stream)
            else:
                q_a, _ = self.q_a_proj(hidden_states)
                latent_cache, _ = self.kv_a_proj_with_mqa(hidden_states)
            q_a = self.q_a_layernorm(q_a)
            q, _ = self.q_b_proj(q_a)
# ... truncated for brevity ...
```
**EN:** This method implements `forward_prepare(positions: ..., hidden_states: ..., forward_batch: ..., zero_allocator: ...=..., llama_4_scaling: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_prepare(positions: ..., hidden_states: ..., forward_batch: ..., zero_allocator: ...=..., llama_4_scaling: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 936-981: Method `SarvamMoEMLAAttention.forward_core`
```python
    def forward_core(
        self,
        intermediate_state: Tuple[
            Optional[torch.Tensor], ForwardBatch, Optional[Tuple]
        ],
    ) -> torch.Tensor:
        hidden_states, forward_batch, inner_state = intermediate_state

        if inner_state is None:
            return hidden_states

        q_nope_out, k_nope, q_pe, k_pe, forward_batch, zero_allocator = inner_state

        if self._server_args is None:
            self._server_args = get_global_server_args()
        self._set_current_attention_backend(forward_batch)

        forward_method = get_attn_forward_method(self._server_args, forward_batch)

        if forward_method == AttnForwardMethod.MLA_SEPARATE_ROPE:
            attn_output = self.attn_mqa(
                q_nope_out,
                k_nope,
                k_nope,
                forward_batch,
                q_rope=q_pe,
                k_rope=k_pe,
            )
        else:
            q = torch.cat([q_nope_out, q_pe], dim=-1)
            k = torch.cat([k_nope, k_pe], dim=-1)
            attn_output = self.attn_mqa(
                q,
                k,
                k_nope,
                forward_batch,
            )
        attn_output = attn_output.view(-1, self.num_local_heads, self.kv_lora_rank)

        attn_bmm_output = self._maybe_fp8_bmm(
# ... truncated for brevity ...
```
**EN:** This method implements `forward_core(intermediate_state: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_core(intermediate_state: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 983-988: Method `SarvamMoEMLAAttention.prepare_qkv_latent`
```python
    def prepare_qkv_latent(
        self, hidden_states: torch.Tensor, forward_batch: ForwardBatch
    ) -> torch.Tensor:
        del forward_batch
        latent_cache, _ = self.kv_a_proj_with_mqa(hidden_states)
        return latent_cache
```
**EN:** This method implements `prepare_qkv_latent(hidden_states: ..., forward_batch: ...)` and prepares intermediate tensors, masks, or metadata before the main compute path.
**CN:** 这个方法实现了 `prepare_qkv_latent(hidden_states: ..., forward_batch: ...)`，其作用是在主计算路径前准备中间张量、掩码或元数据。

### Lines 991-992: Class `SarvamMoEMLADecoderLayer` overview
```python
class SarvamMoEMLADecoderLayer(nn.Module):
```
**EN:** Defines `SarvamMoEMLADecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SarvamMoEMLADecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 993-1093: Method `SarvamMoEMLADecoderLayer.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.config = config
        self.layer_id = layer_id

        if hasattr(config, "rope_parameters"):
            rope_theta = config.rope_parameters.get("rope_theta")
            rope_type = config.rope_parameters.get("rope_type")
            rope_scaling = config.rope_parameters if rope_type != "default" else None
        else:
            rope_theta = getattr(config, "rope_theta", 10000)
            rope_scaling = getattr(config, "rope_scaling", None)
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)

        self.self_attn = SarvamMoEMLAAttention(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            layer_id=layer_id,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            max_position_embeddings=max_position_embeddings,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
            alt_stream=alt_stream,
        )

        first_k_dense = getattr(config, "first_k_dense_replace", 1)
        moe_layer_freq = getattr(config, "moe_layer_freq", 1)
        has_moe = getattr(config, "num_experts", None) is not None
        self.is_layer_sparse = (
            has_moe
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1095-1138: Method `SarvamMoEMLADecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
    ) -> Tuple[torch.Tensor, torch.Tensor]:
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
        use_reduce_scatter = self.layer_communicator.should_use_reduce_scatter(
            forward_batch
        )
        hidden_states = self.mlp(
            hidden_states, forward_batch, should_allreduce_fusion, use_reduce_scatter
        )
        if (
            not self.is_layer_sparse
            and self.attn_tp_size > 1
            and not use_reduce_scatter
            and not should_allreduce_fusion
        ):
            hidden_states = tensor_model_parallel_all_reduce(hidden_states)
        if should_allreduce_fusion:
            hidden_states._sglang_needs_allreduce_fusion = True
        else:
# ... truncated for brevity ...
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 1141-1142: Class `SarvamMLAModel` overview
```python
class SarvamMLAModel(nn.Module):
```
**EN:** Defines `SarvamMLAModel` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SarvamMLAModel`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 1143-1184: Method `SarvamMLAModel.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.padding_idx = config.pad_token_id
        self.vocab_size = config.vocab_size
        self.pp_group = get_pp_group()
        self.alt_stream = torch.cuda.Stream() if _is_cuda else None

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                quant_config=quant_config,
                prefix=add_prefix("embed_tokens", prefix),
                enable_tp=not is_dp_attention_enabled(),
            )
        else:
            self.embed_tokens = nn.Identity()

        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: SarvamMoEMLADecoderLayer(
                config=config,
                quant_config=quant_config,
                layer_id=idx,
                prefix=prefix,
                alt_stream=self.alt_stream,
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix="model.layers",
        )

        if self.pp_group.is_last_rank:
            self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1186-1222: Method `SarvamMLAModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> torch.Tensor:
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
                positions, hidden_states, forward_batch, residual
            )

        if not self.pp_group.is_last_rank:
            return PPProxyTensors(
                {"hidden_states": hidden_states, "residual": residual}
            )

        if hidden_states.shape[0] != 0:
            if residual is None:
                hidden_states = self.norm(hidden_states)
            else:
                hidden_states, _ = self.norm(hidden_states, residual)

        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 1225-1226: Class `SarvamMLAForCausalLM` overview
```python
class SarvamMLAForCausalLM(nn.Module):
```
**EN:** Defines `SarvamMLAForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 10 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SarvamMLAForCausalLM`，其继承关系为 nn.Module。这个类组织了 10 个方法，用于实现模型相关行为。

### Lines 1227-1246: Method `SarvamMLAForCausalLM.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self._remap_config(config)
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        self.model = SarvamMLAModel(config, quant_config, add_prefix("model", prefix))
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("lm_head", prefix),
            use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
        )
        self.logits_processor = LogitsProcessor(config)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1248-1265: Method `SarvamMLAForCausalLM._remap_config`
```python
    @staticmethod
    def _remap_config(config: PretrainedConfig) -> None:
        defaults = {
            "first_k_dense_replace": 1,
            "moe_layer_freq": 1,
            "hidden_act": "silu",
            "tie_word_embeddings": False,
            "n_group": 1,
            "topk_group": 1,
            "router_dtype": "fp32",
            "routed_scaling_factor": 2.5,
            "score_function": "sigmoid",
            "norm_topk_prob": True,
            "topk_method": "noaux_tc",
        }
        for attr, default in defaults.items():
            if not hasattr(config, attr):
                setattr(config, attr, default)
```
**EN:** This method implements `_remap_config()` and implements a focused helper that supports the surrounding runtime flow inside `SarvamMLAForCausalLM` Decorators: staticmethod.
**CN:** 这个方法实现了 `_remap_config()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SarvamMLAForCausalLM` 内部调用 装饰器：staticmethod。

### Lines 1267-1269: Method `SarvamMLAForCausalLM.start_layer`
```python
    @property
    def start_layer(self):
        return self.model.start_layer
```
**EN:** This method implements `start_layer()` and implements a focused helper that supports the surrounding runtime flow inside `SarvamMLAForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `start_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SarvamMLAForCausalLM` 内部调用 装饰器：property。

### Lines 1271-1273: Method `SarvamMLAForCausalLM.end_layer`
```python
    @property
    def end_layer(self):
        return self.model.end_layer
```
**EN:** This method implements `end_layer()` and implements a focused helper that supports the surrounding runtime flow inside `SarvamMLAForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `end_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SarvamMLAForCausalLM` 内部调用 装饰器：property。

### Lines 1275-1276: Method `SarvamMLAForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self) -> nn.Embedding:
        return self.model.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1278-1294: Method `SarvamMLAForCausalLM.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> LogitsProcessorOutput:
        hidden_states = self.model(
            input_ids, positions, forward_batch, input_embeds, pp_proxy_tensors
        )
        if self.pp_group.is_last_rank:
            return self.logits_processor(
                input_ids, hidden_states, self.lm_head, forward_batch
            )
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 1296-1334: Method `SarvamMLAForCausalLM.forward_split_prefill`
```python
    @torch.no_grad()
    def forward_split_prefill(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        split_interval: Tuple[int, int],
        input_embeds: torch.Tensor = None,
    ) -> Optional[LogitsProcessorOutput]:
        start, end = split_interval
        if start == 0:
            if input_embeds is None:
                forward_batch.hidden_states = self.model.embed_tokens(input_ids)
            else:
                forward_batch.hidden_states = input_embeds
            forward_batch.residual = None

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
            if forward_batch.residual is None:
                hidden_states = self.model.norm(forward_batch.hidden_states)
            else:
                hidden_states, _ = self.model.norm(
                    forward_batch.hidden_states, forward_batch.residual
                )
            forward_batch.hidden_states = hidden_states
            return self.logits_processor(
                input_ids, forward_batch.hidden_states, self.lm_head, forward_batch
            )
        return None
```
**EN:** This method implements `forward_split_prefill(input_ids: ..., positions: ..., forward_batch: ..., split_interval: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward_split_prefill(input_ids: ..., positions: ..., forward_batch: ..., split_interval: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 1336-1342: Method `SarvamMLAForCausalLM.get_model_config_for_expert_location`
```python
    @classmethod
    def get_model_config_for_expert_location(cls, config):
        return ModelConfigForExpertLocation(
            num_layers=config.num_hidden_layers,
            num_logical_experts=config.num_experts,
            num_groups=getattr(config, "n_group", None),
        )
```
**EN:** This method implements `get_model_config_for_expert_location(config)` and returns a cached or derived object used elsewhere in the pipeline Decorators: classmethod.
**CN:** 这个方法实现了 `get_model_config_for_expert_location(config)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果 装饰器：classmethod。

### Lines 1344-1429: Method `SarvamMLAForCausalLM.load_weights`
```python
    def load_weights(
        self,
        weights: Iterable[Tuple[str, torch.Tensor]],
        is_nextn: bool = False,
    ) -> None:
        del is_nextn
        stacked_params_mapping = [
            (".gate_up_proj", ".gate_proj", 0),
            (".gate_up_proj", ".up_proj", 1),
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
            if layer_id is not None and (
                layer_id < self.start_layer or layer_id >= self.end_layer
            ):
                continue

            if "rotary_emb.inv_freq" in name:
                continue

            if ".mlp.gate.e_score_correction_bias" in name:
                name = name.replace(
                    ".mlp.gate.e_score_correction_bias", ".mlp.e_score_correction_bias"
                )

            is_stacked = False
            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name or "mlp.experts" in name:
                    continue
                mapped_name = name.replace(weight_name, param_name)
                if mapped_name.endswith(".bias") and mapped_name not in params_dict:
                    continue
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ..., is_nextn: ...=...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ..., is_nextn: ...=...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1431-1474: Method `SarvamMLAForCausalLM._set_mla_wkc_wvc`
```python
    def _set_mla_wkc_wvc(self) -> None:
        for layer_id in range(self.start_layer, self.end_layer):
            layer = self.model.layers[layer_id]
            self_attn = layer.self_attn
            if not hasattr(self_attn, "kv_b_proj") or self_attn.kv_b_proj is None:
                continue

            w = self_attn.kv_b_proj.weight.data
            weight_scale = None
            if w.dtype in (torch.float8_e4m3fn, torch.float8_e4m3fnuz):
                if (
                    hasattr(self_attn.kv_b_proj, "weight_scale")
                    and self_attn.kv_b_proj.weight_scale is not None
                ):
                    weight_scale = self_attn.kv_b_proj.weight_scale
                elif (
                    hasattr(self_attn.kv_b_proj, "weight_scale_inv")
                    and self_attn.kv_b_proj.weight_scale_inv is not None
                ):
                    weight_scale = self_attn.kv_b_proj.weight_scale_inv
                elif (
                    hasattr(self_attn.kv_b_proj, "scale")
                    and self_attn.kv_b_proj.scale is not None
                ):
                    weight_scale = self_attn.kv_b_proj.scale

            w_reshaped = w.unflatten(
                0,
                (
                    self_attn.num_local_heads,
                    self_attn.qk_nope_head_dim + self_attn.v_head_dim,
                ),
            )
            w_kc, w_vc = w_reshaped.split(
                [self_attn.qk_nope_head_dim, self_attn.v_head_dim], dim=1
            )
            self_attn.w_kc = bind_or_assign(
                self_attn.w_kc, w_kc.transpose(1, 2).contiguous().transpose(1, 2)
            )
            self_attn.w_vc = bind_or_assign(
# ... truncated for brevity ...
```
**EN:** This method implements `_set_mla_wkc_wvc()` and implements a focused helper that supports the surrounding runtime flow inside `SarvamMLAForCausalLM`.
**CN:** 这个方法实现了 `_set_mla_wkc_wvc()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SarvamMLAForCausalLM` 内部调用。

### Lines 1477-1479: Class `SarvamMoEForCausalLM` overview
```python
class SarvamMoEForCausalLM(BailingMoEForCausalLM):

    @torch.no_grad()
```
**EN:** Defines `SarvamMoEForCausalLM` as a reusable runtime type derived from BailingMoEForCausalLM. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SarvamMoEForCausalLM`，其继承关系为 BailingMoEForCausalLM。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 1479-1520: Method `SarvamMoEForCausalLM.forward_split_prefill`
```python
    @torch.no_grad()
    def forward_split_prefill(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        split_interval: Tuple[int, int],
        input_embeds: torch.Tensor = None,
    ) -> Optional[LogitsProcessorOutput]:
        start, end = split_interval

        if start == 0:
            if input_embeds is None:
                forward_batch.hidden_states = self.model.word_embeddings(input_ids)
            else:
                forward_batch.hidden_states = input_embeds
            forward_batch.residual = None

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
            if forward_batch.residual is None:
                hidden_states = self.model.norm(forward_batch.hidden_states)
            else:
                hidden_states, _ = self.model.norm(
                    forward_batch.hidden_states, forward_batch.residual
                )
            forward_batch.hidden_states = hidden_states

            return self.logits_processor(
                input_ids, forward_batch.hidden_states, self.lm_head, forward_batch
            )
# ... truncated for brevity ...
```
**EN:** This method implements `forward_split_prefill(input_ids: ..., positions: ..., forward_batch: ..., split_interval: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward_split_prefill(input_ids: ..., positions: ..., forward_batch: ..., split_interval: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 1523-1523: Top-level assign
```python
EntryClass = [SarvamMLAForCausalLM, SarvamMoEForCausalLM]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `math`
- `enum.IntEnum`
- `enum.auto`
- `typing.Any`
- `typing.Dict`
- `typing.Iterable`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn.functional`
- `torch.nn`
- `transformers.PretrainedConfig`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.distributed.tensor_model_parallel_all_reduce`
- `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location.ModelConfigForExpertLocation`
- `sglang.srt.layers.activation.SiluAndMul`
- `sglang.srt.layers.attention.utils.concat_and_cast_mha_k_triton`
- `sglang.srt.layers.communicator.LayerCommunicator`
- `sglang.srt.layers.communicator.LayerScatterModes`
- `sglang.srt.layers.communicator.enable_moe_dense_fully_dp`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.MergedColumnParallelLinear`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.logits_processor.LogitsProcessorOutput`
- `sglang.srt.layers.moe.should_skip_post_experts_all_reduce`
- `sglang.srt.layers.moe.ep_moe.layer.get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`
- `sglang.srt.layers.moe.topk.TopK`
- `sglang.srt.layers.moe.utils.RoutingMethodType`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.utils.get_layer_id`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.cuda_graph_runner.get_is_capture_mode`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.bailing_moe.BailingMoEForCausalLM`
- `sglang.srt.models.deepseek_common.attention_forward_methods.forward_mha.DeepseekMHAForwardMixin`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.BumpAllocator`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.bind_or_assign`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_nvidia_cublas_version_ge_12_9`
- `sglang.srt.utils.make_layers`
- `sglang.srt.utils.next_power_of_2`
- `sgl_kernel.bmm_fp8`
- `sgl_kernel.concat_mla_k`
- `sgl_kernel.merge_state_v2`
- `sglang.srt.layers.quantization.fp8_kernel.per_tensor_quant_mla_fp8`
