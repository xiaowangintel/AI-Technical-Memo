# nemotron_h.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/nemotron_h.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only NemotronH model. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 18-18: Module docstring
```python
"""Inference-only NemotronH model."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 20-87: Module imports
```python
from collections.abc import Iterable
from typing import Optional, Union

import torch
from torch import nn

from sglang.srt.compilation.compilation_config import register_split_op
from sglang.srt.compilation.piecewise_context_manager import (
    get_forward_context,
    is_in_piecewise_cuda_graph,
)
from sglang.srt.configs import NemotronHConfig
from sglang.srt.configs.nemotron_h import ATTENTION, MAMBA, MLP, MOE
from sglang.srt.distributed import (
    get_moe_ep_group,
    get_pp_group,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.layers.activation import ReLU2
from sglang.srt.layers.attention.hybrid_linear_attn_backend import (
    HybridLinearAttnBackend,
    Mamba2AttnBackend,
)
from sglang.srt.layers.attention.mamba.mamba import MambaMixer2
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.moe.ep_moe.layer import get_moe_impl_class
from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE
from sglang.srt.layers.moe.topk import TopK
from sglang.srt.layers.moe.utils import RoutingMethodType
from sglang.srt.layers.quantization import QuantizationConfig
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.layers.utils import PPMissingLayer, get_layer_id
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、SGLang 运行时组件。

### Lines 89-89: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 92-92: Class `NemotronHMLP` overview
```python
class NemotronHMLP(nn.Module):
```
**EN:** Defines `NemotronHMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `NemotronHMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 93-119: Method `NemotronHMLP.__init__`
```python
    def __init__(
        self,
        config: NemotronHConfig,
        intermediate_size: int,
        quant_config: Optional[QuantizationConfig] = None,
        bias: bool = False,
        reduce_results: bool = True,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.up_proj = ColumnParallelLinear(
            input_size=config.hidden_size,
            output_size=intermediate_size,
            bias=bias,
            quant_config=quant_config,
            prefix=f"{prefix}.up_proj",
        )
        self.down_proj = RowParallelLinear(
            input_size=intermediate_size,
            output_size=config.hidden_size,
            bias=bias,
            quant_config=quant_config,
            reduce_results=reduce_results,
            prefix=f"{prefix}.down_proj",
        )
        self.act_fn = ReLU2()
```
**EN:** This method implements `__init__(config: ..., intermediate_size: ..., quant_config: ...=..., bias: ...=..., reduce_results: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., intermediate_size: ..., quant_config: ...=..., bias: ...=..., reduce_results: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 121-125: Method `NemotronHMLP.forward`
```python
    def forward(self, x: torch.Tensor):
        x, _ = self.up_proj(x)
        x = self.act_fn(x)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This method implements `forward(x: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 128-128: Top-level assign
```python
_alt_stream = None
```
**EN:** Defines or updates _alt_stream, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _alt_stream，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 131-135: Function `_get_or_create_alt_stream`
```python
def _get_or_create_alt_stream(device_module):
    global _alt_stream
    if _alt_stream is None:
        _alt_stream = device_module.Stream()
    return _alt_stream
```
**EN:** This function implements `_get_or_create_alt_stream(device_module)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个函数实现了 `_get_or_create_alt_stream(device_module)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 138-138: Class `NemotronHMoE` overview
```python
class NemotronHMoE(nn.Module):
```
**EN:** Defines `NemotronHMoE` as a reusable runtime type derived from nn.Module. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `NemotronHMoE`，其继承关系为 nn.Module。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 139-227: Method `NemotronHMoE.__init__`
```python
    def __init__(
        self,
        config: NemotronHConfig,
        layer_idx: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.tp_size = get_tensor_model_parallel_world_size()
        self.routed_scaling_factor = config.routed_scaling_factor
        self.device_module = torch.get_device_module()

        self.ep_group = get_moe_ep_group().device_group
        self.ep_rank = self.ep_group.rank()
        self.ep_size = self.ep_group.size()
        self.n_routed_experts = config.n_routed_experts
        self.n_shared_experts = config.n_shared_experts
        self.use_latent_moe = getattr(config, "moe_latent_size", None) is not None
        self.moe_hidden_size = (
            config.moe_latent_size if self.use_latent_moe else config.hidden_size
        )

        self.gate = ReplicatedLinear(
            config.hidden_size,
            config.n_routed_experts,
            bias=False,
            params_dtype=torch.float32,
            quant_config=None,
            prefix=f"{prefix}.gate",
        )
        self.gate.e_score_correction_bias = nn.Parameter(
            torch.empty(config.n_routed_experts, dtype=torch.float32)
        )

        self.topk = TopK(
            top_k=config.num_experts_per_tok,
            use_grouped_topk=True,
            topk_group=config.topk_group,
            num_expert_group=config.n_group,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 229-238: Method `NemotronHMoE._forward_core`
```python
    def _forward_core(
        self,
        hidden_states: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        # torch.compile cannot trace CUDA streams, so use the non-overlapping
        # path when inside piecewise CUDA graph compilation.
        if _is_cuda and not is_in_piecewise_cuda_graph():
            return self._forward_core_shared_routed_overlap(hidden_states)
        else:
            return self._forward_core_normal(hidden_states)
```
**EN:** This method implements `_forward_core(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_core(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 240-254: Method `NemotronHMoE._forward_core_normal`
```python
    def _forward_core_normal(
        self,
        hidden_states: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        # router_scores: [num_tokens, num_experts]
        router_logits, _ = self.gate(hidden_states.to(dtype=torch.float32))
        if self.shared_experts is not None:
            shared_output = self.shared_experts(hidden_states)
        else:
            shared_output = None
        topk_output = self.topk(hidden_states, router_logits)
        if self.use_latent_moe:
            hidden_states, _ = self.fc1_latent_proj(hidden_states)
        final_hidden_states = self.experts(hidden_states, topk_output)
        return final_hidden_states, shared_output
```
**EN:** This method implements `_forward_core_normal(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_core_normal(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 256-278: Method `NemotronHMoE._forward_core_shared_routed_overlap`
```python
    def _forward_core_shared_routed_overlap(
        self,
        hidden_states: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        alt_stream = _get_or_create_alt_stream(self.device_module)

        alt_stream.wait_stream(get_current_device_stream_fast())

        if self.shared_experts is not None:
            shared_output = self.shared_experts(hidden_states)
        else:
            shared_output = None

        with self.device_module.stream(alt_stream):
            # router_scores: [num_tokens, num_experts]
            router_logits, _ = self.gate(hidden_states.to(dtype=torch.float32))
            topk_output = self.topk(hidden_states, router_logits)
            if self.use_latent_moe:
                hidden_states, _ = self.fc1_latent_proj(hidden_states)
            final_hidden_states = self.experts(hidden_states, topk_output)
        get_current_device_stream_fast().wait_stream(alt_stream)

        return final_hidden_states, shared_output
```
**EN:** This method implements `_forward_core_shared_routed_overlap(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_core_shared_routed_overlap(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 280-300: Method `NemotronHMoE.forward`
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        num_tokens, hidden_dim = hidden_states.shape
        final_hidden_states, shared_output = self._forward_core(hidden_states)

        # Fix FP16 overflow
        if hidden_states.dtype != torch.float16:
            final_hidden_states *= self.routed_scaling_factor
        elif self.shared_experts is not None:
            assert shared_output is not None
            shared_output *= 1.0 / self.routed_scaling_factor

        if self.use_latent_moe:
            final_hidden_states, _ = self.fc2_latent_proj(final_hidden_states)

        if shared_output is not None:
            final_hidden_states += shared_output

        if self.tp_size > 1:
            final_hidden_states = tensor_model_parallel_all_reduce(final_hidden_states)

        return final_hidden_states.view(num_tokens, hidden_dim)
```
**EN:** This method implements `forward(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 303-303: Class `NemotronHMLPDecoderLayer` overview
```python
class NemotronHMLPDecoderLayer(nn.Module):
```
**EN:** Defines `NemotronHMLPDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `NemotronHMLPDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 304-332: Method `NemotronHMLPDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: NemotronHConfig,
        layer_idx: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config

        hybrid_override_pattern = config.hybrid_override_pattern
        mlp_index = hybrid_override_pattern[: layer_idx + 1].count("-") - 1
        if isinstance(config.intermediate_size, list):
            if len(config.intermediate_size) == 1:
                intermediate_size = config.intermediate_size[0]
            else:
                intermediate_size = config.intermediate_size[mlp_index]
        else:
            intermediate_size = config.intermediate_size

        self.mixer = NemotronHMLP(
            config,
            intermediate_size=intermediate_size,
            quant_config=quant_config,
            bias=config.mlp_bias,
            prefix=f"{prefix}.mixer",
        )

        self.norm = RMSNorm(config.hidden_size, eps=config.layer_norm_epsilon)
```
**EN:** This method implements `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 334-348: Method `NemotronHMLPDecoderLayer.forward`
```python
    def forward(
        self,
        *,
        hidden_states: torch.Tensor,
        residual: Optional[torch.Tensor],
        forward_batch: ForwardBatch,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        if residual is None:
            residual = hidden_states
            hidden_states = self.norm(hidden_states)
        else:
            hidden_states, residual = self.norm(hidden_states, residual)

        hidden_states = self.mixer.forward(hidden_states)
        return hidden_states, residual
```
**EN:** This method implements `forward(*, hidden_states: ..., residual: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(*, hidden_states: ..., residual: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 351-351: Class `NemotronHMoEDecoderLayer` overview
```python
class NemotronHMoEDecoderLayer(nn.Module):
```
**EN:** Defines `NemotronHMoEDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `NemotronHMoEDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 352-368: Method `NemotronHMoEDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: NemotronHConfig,
        layer_idx: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.mixer = NemotronHMoE(
            config,
            layer_idx=layer_idx,
            quant_config=quant_config,
            prefix=f"{prefix}.mixer",
        )

        self.norm = RMSNorm(config.hidden_size, eps=config.layer_norm_epsilon)
```
**EN:** This method implements `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 370-384: Method `NemotronHMoEDecoderLayer.forward`
```python
    def forward(
        self,
        *,
        hidden_states: torch.Tensor,
        residual: Optional[torch.Tensor],
        forward_batch: ForwardBatch,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        if residual is None:
            residual = hidden_states
            hidden_states = self.norm(hidden_states)
        else:
            hidden_states, residual = self.norm(hidden_states, residual)

        hidden_states = self.mixer.forward(hidden_states)
        return hidden_states, residual
```
**EN:** This method implements `forward(*, hidden_states: ..., residual: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(*, hidden_states: ..., residual: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 387-387: Class `NemotronHMambaDecoderLayer` overview
```python
class NemotronHMambaDecoderLayer(nn.Module):
```
**EN:** Defines `NemotronHMambaDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `NemotronHMambaDecoderLayer`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 388-410: Method `NemotronHMambaDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: NemotronHConfig,
        layer_idx: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.layer_id = layer_idx
        self.mixer = MambaMixer2(
            cache_params=config.mamba2_cache_params,
            hidden_size=config.hidden_size,
            use_conv_bias=config.use_conv_bias,
            use_bias=config.use_bias,
            n_groups=config.mamba_n_groups,
            rms_norm_eps=config.layer_norm_epsilon,
            activation=config.mamba_hidden_act,
            quant_config=quant_config,
            prefix=f"{prefix}.mixer",
        )

        self.norm = RMSNorm(config.hidden_size, eps=config.layer_norm_epsilon)
```
**EN:** This method implements `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 412-427: Method `NemotronHMambaDecoderLayer._forward_mamba`
```python
    def _forward_mamba(
        self, hidden_states: torch.Tensor, forward_batch: ForwardBatch
    ) -> torch.Tensor:
        """Core Mamba forward logic, called directly or via split op."""
        output = torch.empty_like(hidden_states)
        attn_backend = forward_batch.attn_backend
        assert isinstance(attn_backend, HybridLinearAttnBackend)
        assert isinstance(attn_backend.linear_attn_backend, Mamba2AttnBackend)
        attn_backend.linear_attn_backend.forward(
            mixer=self.mixer,
            layer_id=self.layer_id,
            hidden_states=hidden_states,
            output=output,
            use_triton_causal_conv=True,
        )
        return output
```
**EN:** This method implements `_forward_mamba(hidden_states: ..., forward_batch: ...)` and Core Mamba forward logic, called directly or via split op.
**CN:** 这个方法实现了 `_forward_mamba(hidden_states: ..., forward_batch: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 429-453: Method `NemotronHMambaDecoderLayer.forward`
```python
    def forward(
        self,
        *,
        hidden_states: torch.Tensor,
        residual: Optional[torch.Tensor],
        forward_batch: ForwardBatch,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        if residual is None:
            residual = hidden_states
            hidden_states = self.norm(hidden_states)
        else:
            hidden_states, residual = self.norm(hidden_states, residual)

        if is_in_breakable_cuda_graph():
            output = torch.empty_like(hidden_states)
            breakable_nemotron_mamba2_with_output(hidden_states, output, self.layer_id)
            return output, residual

        if is_in_piecewise_cuda_graph():
            output = torch.empty_like(hidden_states)
            nemotron_mamba2_with_output(hidden_states, output, self.layer_id)
            return output, residual
        else:
            output = self._forward_mamba(hidden_states, forward_batch)
            return output, residual
```
**EN:** This method implements `forward(*, hidden_states: ..., residual: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(*, hidden_states: ..., residual: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 456-456: Class `NemotronHAttention` overview
```python
class NemotronHAttention(nn.Module):
```
**EN:** Defines `NemotronHAttention` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `NemotronHAttention`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 457-513: Method `NemotronHAttention.__init__`
```python
    def __init__(
        self,
        config: NemotronHConfig,
        layer_idx: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        if hasattr(config, "head_dim") and config.head_dim is not None:
            self.head_dim = config.head_dim
        else:
            self.head_dim = config.hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5

        self.qkv_proj = QKVParallelLinear(
            config.hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 515-522: Method `NemotronHAttention.forward`
```python
    def forward(
        self, hidden_states: torch.Tensor, forward_batch: ForwardBatch
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        attn_output = self.attn.forward(q, k, v, forward_batch)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This method implements `forward(hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 525-525: Class `NemotronHAttentionDecoderLayer` overview
```python
class NemotronHAttentionDecoderLayer(nn.Module):
```
**EN:** Defines `NemotronHAttentionDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `NemotronHAttentionDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 526-542: Method `NemotronHAttentionDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: NemotronHConfig,
        layer_idx: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.mixer = NemotronHAttention(
            config,
            layer_idx,
            quant_config,
            prefix=f"{prefix}.mixer",
        )

        self.norm = RMSNorm(config.hidden_size, eps=config.layer_norm_epsilon)
```
**EN:** This method implements `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_idx: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 544-560: Method `NemotronHAttentionDecoderLayer.forward`
```python
    def forward(
        self,
        *,
        hidden_states: torch.Tensor,
        residual: Optional[torch.Tensor],
        forward_batch: ForwardBatch,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        if residual is None:
            residual = hidden_states
            hidden_states = self.norm(hidden_states)
        else:
            hidden_states, residual = self.norm(hidden_states, residual)

        hidden_states = self.mixer.forward(
            hidden_states=hidden_states, forward_batch=forward_batch
        )
        return hidden_states, residual
```
**EN:** This method implements `forward(*, hidden_states: ..., residual: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(*, hidden_states: ..., residual: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 563-568: Top-level assign
```python
Layers = (
    NemotronHAttentionDecoderLayer,
    NemotronHMLPDecoderLayer,
    NemotronHMambaDecoderLayer,
    NemotronHMoEDecoderLayer,
)
```
**EN:** Defines or updates Layers, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 Layers，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 569-574: Top-level annassign
```python
ALL_DECODER_LAYER_TYPES: dict[str, type] = {
    ATTENTION: NemotronHAttentionDecoderLayer,
    MLP: NemotronHMLPDecoderLayer,
    MAMBA: NemotronHMambaDecoderLayer,
    MOE: NemotronHMoEDecoderLayer,
}
```
**EN:** Defines or updates ALL_DECODER_LAYER_TYPES, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 ALL_DECODER_LAYER_TYPES，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 577-577: Class `NemotronHModel` overview
```python
class NemotronHModel(nn.Module):
```
**EN:** Defines `NemotronHModel` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `NemotronHModel`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 578-621: Method `NemotronHModel.__init__`
```python
    def __init__(
        self,
        *,
        config: NemotronHConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        lora_config = None
        self.config = config
        lora_vocab = (
            (lora_config.lora_extra_vocab_size * (lora_config.max_loras or 1))
            if lora_config
            else 0
        )
        self.vocab_size = config.vocab_size + lora_vocab
        self.org_vocab_size = config.vocab_size
        self.pp_group = get_pp_group()

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                self.vocab_size,
                config.hidden_size,
                org_num_embeddings=config.vocab_size,
            )
        else:
            self.embed_tokens = PPMissingLayer()

        def get_layer(idx: int, prefix: str):
            layer_class = ALL_DECODER_LAYER_TYPES[config.hybrid_override_pattern[idx]]
            return layer_class(config, idx, quant_config=quant_config, prefix=prefix)

        self.layers, self.start_layer, self.end_layer = make_layers(
            len(config.hybrid_override_pattern),
            get_layer,
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix=f"{prefix}.layers",
        )
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 623-657: Method `NemotronHModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
        inputs_embeds: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, PPProxyTensors]:
        if self.pp_group.is_first_rank:
            if inputs_embeds is not None:
                hidden_states = inputs_embeds
            else:
                hidden_states = self.embed_tokens(input_ids)
            residual = None
        else:
            assert pp_proxy_tensors is not None
            hidden_states = pp_proxy_tensors["hidden_states"]
            residual = pp_proxy_tensors["residual"]

        for i in range(self.start_layer, self.end_layer):
            layer = self.layers[i]
            if not isinstance(layer, Layers):
                raise ValueError(f"Unknown layer type: {type(layer)}")
            hidden_states, residual = layer.forward(
                hidden_states=hidden_states,
                residual=residual,
                forward_batch=forward_batch,
            )

        if not self.pp_group.is_last_rank:
            return PPProxyTensors(
                {"hidden_states": hidden_states, "residual": residual}
            )
        hidden_states, _ = self.norm_f(hidden_states, residual)
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., pp_proxy_tensors: ...=..., inputs_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., pp_proxy_tensors: ...=..., inputs_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 660-660: Class `NemotronHForCausalLM` overview
```python
class NemotronHForCausalLM(nn.Module):
```
**EN:** Defines `NemotronHForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 11 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `NemotronHForCausalLM`，其继承关系为 nn.Module。这个类组织了 11 个方法，用于实现模型相关行为。

### Lines 661-694: Class `NemotronHForCausalLM` attributes
```python
    stacked_params_mapping = [
        # (param_name, shard_name, shard_id)
        ("qkv_proj", "q_proj", "q"),
        ("qkv_proj", "k_proj", "k"),
        ("qkv_proj", "v_proj", "v"),
    ]
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
    }
    supported_lora_modules = [
        "qkv_proj",
        "o_proj",
        "out_proj",
        "in_proj",
        "up_proj",
        "gate_up_proj",
        "down_proj",
        "fc1_latent_proj",
        "fc2_latent_proj",
    ]

    remap_prefix = {"backbone": "model"}
    remap_substr = {
        "A_log": "A",
        "embeddings": "embed_tokens",
        "k_proj.k_scale": "attn.k_scale",
        "v_proj.v_scale": "attn.v_scale",
    }

    hf_to_sglang_mapper = WeightsMapper(
        orig_to_new_prefix={
            "backbone.": "model.",
        }
    )
```
**EN:** Defines class-level attributes and metadata that shape how `NemotronHForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `NemotronHForCausalLM` 在运行时的行为。

### Lines 696-749: Method `NemotronHForCausalLM.__init__`
```python
    def __init__(
        self,
        *,
        config: NemotronHConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        lora_config = None
        self.config = config
        self.quant_config = quant_config
        self.model = self._init_model(
            config=config, quant_config=quant_config, prefix=prefix
        )
        self.pp_group = get_pp_group()

        if self.pp_group.is_last_rank:
            if self.pp_group.world_size == 1 and self.config.tie_word_embeddings:
                self.lm_head = self.model.embed_tokens
            else:
                self.unpadded_vocab_size = config.vocab_size
                if lora_config:
                    self.unpadded_vocab_size += lora_config.lora_extra_vocab_size
                self.lm_head = ParallelLMHead(
                    self.unpadded_vocab_size,
                    config.hidden_size,
                    org_num_embeddings=config.vocab_size,
                    padding_size=(
                        DEFAULT_VOCAB_PADDING_SIZE
                        # We need bigger padding if using lora for kernel
                        # compatibility
                        if not lora_config
                        else lora_config.lora_vocab_padding_size
                    ),
                    quant_config=quant_config,
                    prefix=add_prefix("lm_head", prefix),
                )
        else:
            self.lm_head = PPMissingLayer()

# ... truncated for brevity ...
```
**EN:** This method implements `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 751-759: Method `NemotronHForCausalLM._init_model`
```python
    def _init_model(
        self,
        config: NemotronHConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        return NemotronHModel(
            config=config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )
```
**EN:** This method implements `_init_model(config: ..., quant_config: ...=..., prefix: ...=...)` and performs setup work for later runtime execution.
**CN:** 这个方法实现了 `_init_model(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是执行后续运行时所需的初始化工作。

### Lines 761-762: Method `NemotronHForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self) -> VocabParallelEmbedding:
        return self.model.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 764-771: Method `NemotronHForCausalLM.get_stacked_multiply`
```python
    def get_stacked_multiply(self, module_name):
        """Non-gated MoE uses stacked_multiply=1 for gate_up_proj_moe."""
        if module_name == "gate_up_proj_moe":
            return 1  # Non-gated: only w1, no w3
        # Fall back to defaults for everything else
        from sglang.srt.lora.utils import get_stacked_multiply

        return get_stacked_multiply(module_name)
```
**EN:** This method implements `get_stacked_multiply(module_name)` and Non-gated MoE uses stacked_multiply=1 for gate_up_proj_moe.
**CN:** 这个方法实现了 `get_stacked_multiply(module_name)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 773-856: Method `NemotronHForCausalLM.get_hidden_dim`
```python
    def get_hidden_dim(self, module_name, layer_idx):
        """Return (input_dim, output_dim) for LoRA buffers, per layer type."""
        config = self.config
        layer_type = config.layers_block_type[layer_idx]
        hidden_size = config.hidden_size
        head_dim = getattr(
            config, "head_dim", hidden_size // config.num_attention_heads
        )

        if module_name == "qkv_proj":
            return (
                hidden_size,
                head_dim
                * (config.num_attention_heads + config.num_key_value_heads * 2),
            )
        elif module_name == "o_proj":
            return (
                head_dim * config.num_attention_heads,
                hidden_size,
            )
        elif module_name == "out_proj":
            # Mamba out_proj: RowParallelLinear from mamba_intermediate to hidden_size
            mamba_intermediate = config.mamba_num_heads * config.mamba_head_dim
            return mamba_intermediate, hidden_size
        elif module_name == "gate_up_proj":
            if layer_type == "mamba":
                # Mamba in_proj gate component: output = mamba_num_heads * mamba_head_dim
                mamba_intermediate = config.mamba_num_heads * config.mamba_head_dim
                return hidden_size, mamba_intermediate * 2
            elif layer_type == "moe":
                # Shared expert: only has up_proj (no gate), but gets stacked
                shared_inter = (
                    config.moe_shared_expert_intermediate_size * config.n_shared_experts
                )
                return hidden_size, shared_inter * 2
            else:
                # MLP layer
                return hidden_size, config.intermediate_size * 2
        elif module_name == "up_proj":
            if layer_type == "moe":
# ... truncated for brevity ...
```
**EN:** This method implements `get_hidden_dim(module_name, layer_idx)` and Return (input_dim, output_dim) for LoRA buffers, per layer type.
**CN:** 这个方法实现了 `get_hidden_dim(module_name, layer_idx)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 858-875: Method `NemotronHForCausalLM.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: Optional[torch.Tensor] = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ):
        hidden_states = self.model.forward(
            input_ids, positions, forward_batch, pp_proxy_tensors, input_embeds
        )
        if self.pp_group.is_last_rank:
            return self.logits_processor(
                input_ids, hidden_states, self.lm_head, forward_batch
            )
        else:
            return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 877-878: Method `NemotronHForCausalLM.copy_inputs_before_cuda_graphs`
```python
    def copy_inputs_before_cuda_graphs(self, input_buffers, **kwargs):
        return self.mamba_cache.copy_inputs_before_cuda_graphs(input_buffers, **kwargs)
```
**EN:** This method implements `copy_inputs_before_cuda_graphs(input_buffers, **kwargs)` and implements a focused helper that supports the surrounding runtime flow inside `NemotronHForCausalLM`.
**CN:** 这个方法实现了 `copy_inputs_before_cuda_graphs(input_buffers, **kwargs)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `NemotronHForCausalLM` 内部调用。

### Lines 880-881: Method `NemotronHForCausalLM.get_seqlen_agnostic_capture_inputs`
```python
    def get_seqlen_agnostic_capture_inputs(self, batch_size: int):
        return self.mamba_cache.get_seqlen_agnostic_capture_inputs(batch_size)
```
**EN:** This method implements `get_seqlen_agnostic_capture_inputs(batch_size: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_seqlen_agnostic_capture_inputs(batch_size: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 883-884: Method `NemotronHForCausalLM.get_embed_and_head`
```python
    def get_embed_and_head(self):
        return self.model.embed_tokens.weight, self.lm_head.weight
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 886-892: Method `NemotronHForCausalLM.set_embed_and_head`
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

### Lines 894-1002: Method `NemotronHForCausalLM.load_weights`
```python
    def load_weights(
        self, weights: Iterable[tuple[str, torch.Tensor]], is_mtp: bool = False
    ) -> None:
        # - FusedMoe.w1 (aka gate_proj) should be up_proj since that's
        #   what the activation is applied to
        # - FusedMoe.w3 (aka up_proj) should be ignored since we're
        #   using non-gated MoE
        expert_params_mapping = FusedMoE.make_expert_params_mapping(
            ckpt_gate_proj_name="up_proj",
            ckpt_down_proj_name="down_proj",
            ckpt_up_proj_name="",
            num_experts=self.config.n_routed_experts,
        )

        params_dict = dict(self.named_parameters())

        # Stream weights directly from the generator to avoid buffering
        # the entire checkpoint (~75 GB) into a Python list. On unified-
        # memory systems (e.g. DGX Spark, 119 GB) the old buffered path
        # caused OOM: skeleton 81.6 GB + buffer 75 GB = 157 GB peak.
        for name, loaded_weight in weights:
            name = replace_prefix(name, self.remap_prefix)
            name = replace_substrings(name, self.remap_substr)
            if is_mtp:
                if "mtp" not in name:
                    continue

                name = name.replace("mtp.layers.", "model.layers.")

                if "embeddings" in name:
                    name = name.replace("embeddings", "model.embed_tokens")
                    if name.startswith("backbone."):
                        name = name.replace("backbone.", "")

            if not is_mtp and "mtp" in name:
                continue

            if "scale" in name:
                if name not in params_dict:
                    name = maybe_remap_kv_scale_name(name, params_dict)
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ..., is_mtp: ...=...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ..., is_mtp: ...=...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1005-1005: Top-level assign
```python
EntryClass = [NemotronHForCausalLM]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

### Lines 1008-1038: Function `nemotron_mamba2_with_output`
```python
@register_custom_op(mutates_args=["output"])
@register_split_op()
def nemotron_mamba2_with_output(
    hidden_states: torch.Tensor,
    output: torch.Tensor,
    layer_id: int,
) -> None:
    """Split op for Mamba2 forward in piecewise CUDA graph mode."""
    context = get_forward_context()
    forward_batch = context.forward_batch
    attention_layers = context.attention_layers
    mamba_layer = attention_layers[layer_id]

    # In piecewise CUDA graph mode, hidden_states may be padded to the
    # captured graph size. Slice to actual token count for Mamba forward.
    attn_backend = forward_batch.attn_backend
    metadata = attn_backend.linear_attn_backend.forward_metadata
    num_actual_tokens = metadata.num_prefill_tokens + (
        metadata.num_decodes * metadata.draft_token_num
        if metadata.is_target_verify
        else metadata.num_decodes
    )
    if hidden_states.shape[0] != num_actual_tokens:
        hidden_states = hidden_states[:num_actual_tokens]

    ret = mamba_layer._forward_mamba(hidden_states, forward_batch)

    # Copy result back; output may be larger (padded) so only fill actual tokens
    output[:num_actual_tokens].view(ret.shape).copy_(ret)
    if output.shape[0] != num_actual_tokens:
        output[num_actual_tokens:].zero_()
```
**EN:** This function implements `nemotron_mamba2_with_output(hidden_states: ..., output: ..., layer_id: ...)` and Split op for Mamba2 forward in piecewise CUDA graph mode. Decorators: register_custom_op(mutates_args=['output']), register_split_op().
**CN:** 这个函数实现了 `nemotron_mamba2_with_output(hidden_states: ..., output: ..., layer_id: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：register_custom_op(mutates_args=['output']), register_split_op()。

### Lines 1041-1043: Top-level assign
```python
breakable_nemotron_mamba2_with_output = eager_on_graph(True)(
    nemotron_mamba2_with_output
)
```
**EN:** Defines or updates breakable_nemotron_mamba2_with_output, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 breakable_nemotron_mamba2_with_output，为后续代码提供模块级配置、元数据或缓存对象。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影
- **EN:** Mixture-of-Experts routing / **CN:** 混合专家路由

## Dependencies / 依赖关系
- `collections.abc.Iterable`
- `typing.Optional`
- `typing.Union`
- `torch`
- `torch.nn`
- `sglang.srt.compilation.compilation_config.register_split_op`
- `sglang.srt.compilation.piecewise_context_manager.get_forward_context`
- `sglang.srt.compilation.piecewise_context_manager.is_in_piecewise_cuda_graph`
- `sglang.srt.configs.NemotronHConfig`
- `sglang.srt.configs.nemotron_h.ATTENTION`
- `sglang.srt.configs.nemotron_h.MAMBA`
- `sglang.srt.configs.nemotron_h.MLP`
- `sglang.srt.configs.nemotron_h.MOE`
- `sglang.srt.distributed.get_moe_ep_group`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.distributed.tensor_model_parallel_all_reduce`
- `sglang.srt.layers.activation.ReLU2`
- `sglang.srt.layers.attention.hybrid_linear_attn_backend.HybridLinearAttnBackend`
- `sglang.srt.layers.attention.hybrid_linear_attn_backend.Mamba2AttnBackend`
- `sglang.srt.layers.attention.mamba.mamba.MambaMixer2`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.moe.ep_moe.layer.get_moe_impl_class`
- `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`
- `sglang.srt.layers.moe.topk.TopK`
- `sglang.srt.layers.moe.utils.RoutingMethodType`
- `sglang.srt.layers.quantization.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.utils.get_layer_id`
- `sglang.srt.layers.vocab_parallel_embedding.DEFAULT_VOCAB_PADDING_SIZE`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.breakable_cuda_graph.breakable_cuda_graph.eager_on_graph`
- `sglang.srt.model_executor.breakable_cuda_graph.context.is_in_breakable_cuda_graph`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.model_loader.weight_utils.maybe_remap_kv_scale_name`
- `sglang.srt.model_loader.weight_utils.replace_prefix`
- `sglang.srt.model_loader.weight_utils.replace_substrings`
- `sglang.srt.models.utils.WeightsMapper`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.get_current_device_stream_fast`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.make_layers`
- `sglang.srt.utils.custom_op.register_custom_op`
- `sglang.utils.logger`
- `sglang.srt.lora.utils.get_stacked_multiply`
