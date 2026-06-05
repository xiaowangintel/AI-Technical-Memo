# qwen3_next.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/qwen3_next.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the qwen3 next model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 qwen3 next 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 1-55: Module imports
```python
import enum
import logging
from typing import Any, Iterable, Optional, Set, Tuple

import torch
import triton
from torch import nn

from sglang.srt.configs.qwen3_next import Qwen3NextConfig
from sglang.srt.distributed import get_pp_group
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.eplb.expert_location import ModelConfigForExpertLocation
from sglang.srt.layers.attention.fla.layernorm_gated import RMSNorm as RMSNormGated
from sglang.srt.layers.attention.mamba.mamba import mamba_v2_sharded_weight_loader
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
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.layers.radix_linear_attention import RadixLinearAttention
from sglang.srt.layers.rotary_embedding import get_rope
from sglang.srt.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.model_executor.cuda_graph_runner import get_is_capture_mode
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import (
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、SGLang 运行时组件。

### Lines 57-57: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 59-59: Top-level importfrom
```python
from sglang.jit_kernel.triton.gdn_fused_proj import fused_qkvzba_split_reshape_cat
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 60-60: Top-level importfrom
```python
from sglang.srt.layers.attention.fla.fused_norm_gate import FusedRMSNormGated
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 62-62: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 63-63: Top-level assign
```python
_is_npu = is_npu()
```
**EN:** Defines or updates _is_npu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_npu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 64-64: Top-level assign
```python
_is_cpu = is_cpu()
```
**EN:** Defines or updates _is_cpu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cpu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 65-65: Top-level assign
```python
_is_amx_available = cpu_has_amx_support()
```
**EN:** Defines or updates _is_amx_available, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_amx_available，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 68-73: Top-level if
```python
if _is_npu:
    from sgl_kernel_npu.fla.utils import (
        fused_qkvzba_split_reshape_cat as fused_qkvzba_split_reshape_cat_npu,
    )

    fused_qkvzba_split_reshape_cat = fused_qkvzba_split_reshape_cat_npu
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 76-76: Class `Qwen3GatedDeltaNet` overview
```python
class Qwen3GatedDeltaNet(nn.Module):
```
**EN:** Defines `Qwen3GatedDeltaNet` as a reusable runtime type derived from nn.Module. The class groups 7 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3GatedDeltaNet`，其继承关系为 nn.Module。这个类组织了 7 个方法，用于实现模型相关行为。

### Lines 77-225: Method `Qwen3GatedDeltaNet.__init__`
```python
    def __init__(
        self,
        config: Qwen3NextConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        alt_stream: Optional[torch.cuda.Stream] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.attn_tp_rank = get_attention_tp_rank()
        self.attn_tp_size = get_attention_tp_size()
        self.hidden_size = config.hidden_size
        self.num_v_heads = (
            config.linear_num_value_heads
            if not _is_cpu
            else config.linear_num_value_heads_cpu
        )
        self.num_k_heads = (
            config.linear_num_key_heads
            if not _is_cpu
            else config.linear_num_key_heads_cpu
        )
        self.head_k_dim = config.linear_key_head_dim
        self.head_v_dim = config.linear_value_head_dim
        self.key_dim = self.head_k_dim * self.num_k_heads
        self.value_dim = self.head_v_dim * self.num_v_heads
        self.alt_stream = alt_stream

        self.conv_kernel_size = config.linear_conv_kernel_dim
        self.layer_id = layer_id
        self.activation = config.hidden_act
        self.layer_norm_epsilon = config.rms_norm_eps

        self.conv_dim = self.key_dim * 2 + self.value_dim
        self.conv1d = ColumnParallelLinear(
            input_size=self.conv_kernel_size,
            output_size=self.conv_dim,
            bias=False,
            quant_config=None,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., alt_stream: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., alt_stream: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 227-247: Method `Qwen3GatedDeltaNet._override_weight_loader`
```python
    @staticmethod
    def _override_weight_loader(module, new_loader):
        """Override weight_loader on a module's weight parameter.

        ModelWeightParameter exposes weight_loader as a read-only property
        backed by _weight_loader, while plain parameters store it as a
        regular attribute.  This helper handles both cases."""
        for attr_name in (
            "weight",
            "weight_scale_inv",
            "weight_scale",
            "input_scale",
            "weight_offset",
        ):
            param = getattr(module, attr_name, None)
            if param is None:
                continue
            if hasattr(param, "_weight_loader"):
                param._weight_loader = new_loader
            else:
                param.weight_loader = new_loader
```
**EN:** This method implements `_override_weight_loader(new_loader)` and Override weight_loader on a module's weight parameter. Decorators: staticmethod.
**CN:** 这个方法实现了 `_override_weight_loader(new_loader)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：staticmethod。

### Lines 249-286: Method `Qwen3GatedDeltaNet._make_packed_weight_loader`
```python
    @staticmethod
    def _make_packed_weight_loader(module):
        """Create a weight_loader that does contiguous TP slicing for fused
        (packed-format) checkpoint weights (shard_id=None), and delegates
        to the standard MergedColumnParallelLinear loader for split checkpoint
        weights (shard_id=int/tuple)."""
        original_loader = module.weight.weight_loader

        def weight_loader(param, loaded_weight, loaded_shard_id=None):
            if loaded_shard_id is None:
                # Fused checkpoint: weight is in packed (per-head-group)
                # format. Do contiguous TP slice like ColumnParallelLinear.
                output_dim = getattr(param, "output_dim", None)
                if output_dim is not None and module.tp_size > 1:
                    shard_size = param.data.shape[output_dim]
                    start_idx = module.tp_rank * shard_size
                    if (
                        _is_cpu and _is_amx_available
                    ) and start_idx + shard_size > loaded_weight.shape[output_dim]:
                        shard_size = loaded_weight.shape[output_dim] - start_idx
                    loaded_weight = loaded_weight.narrow(
                        output_dim, start_idx, shard_size
                    )
                if _is_cpu and _is_amx_available:
                    slices = tuple(slice(0, s) for s in loaded_weight.shape)
                    param.data.zero_()
                    param.data[slices].copy_(loaded_weight)
                else:
                    assert param.data.shape == loaded_weight.shape, (
                        f"Shape mismatch: param {param.data.shape} vs "
                        f"loaded {loaded_weight.shape}"
                    )
                    param.data.copy_(loaded_weight)
            else:
                # Split checkpoint (int or tuple shard_id) → standard path
                original_loader(param, loaded_weight, loaded_shard_id)

        return weight_loader
```
**EN:** This method implements `_make_packed_weight_loader()` and Create a weight_loader that does contiguous TP slicing for fused. Decorators: staticmethod.
**CN:** 这个方法实现了 `_make_packed_weight_loader()`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：staticmethod。

### Lines 288-306: Method `Qwen3GatedDeltaNet.create_qkvz_proj`
```python
    def create_qkvz_proj(
        self,
        hidden_size: int,
        key_dim: int,
        value_dim: int,
        quant_config: QuantizationConfig | None,
        prefix: str,
        tp_rank: Optional[int] = None,
        tp_size: Optional[int] = None,
    ) -> MergedColumnParallelLinear:
        return MergedColumnParallelLinear(
            input_size=hidden_size,
            output_sizes=[key_dim, key_dim, value_dim, value_dim],
            bias=False,
            quant_config=quant_config,
            prefix=prefix,
            tp_rank=tp_rank,
            tp_size=tp_size,
        )
```
**EN:** This method implements `create_qkvz_proj(hidden_size: ..., key_dim: ..., value_dim: ..., quant_config: ..., prefix: ..., tp_rank: ...=..., ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3GatedDeltaNet`.
**CN:** 这个方法实现了 `create_qkvz_proj(hidden_size: ..., key_dim: ..., value_dim: ..., quant_config: ..., prefix: ..., tp_rank: ...=..., ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3GatedDeltaNet` 内部调用。

### Lines 308-356: Method `Qwen3GatedDeltaNet.fix_query_key_value_ordering`
```python
    def fix_query_key_value_ordering(
        self,
        mixed_qkvz: torch.Tensor,
        mixed_ba: torch.Tensor,
    ):
        """
        Derives `query`, `key` and `value` tensors from `mixed_qkvzba`.
        """
        new_tensor_shape_qkvz = mixed_qkvz.size()[:-1] + (
            self.num_k_heads // self.attn_tp_size,
            (
                self.head_k_dim
                + self.head_k_dim
                + (self.head_v_dim + self.head_v_dim)
                * self.num_v_heads
                // self.num_k_heads
            ),
        )
        new_tensor_shape_ba = mixed_ba.size()[:-1] + (
            self.num_k_heads // self.attn_tp_size,
            2 * self.num_v_heads // self.num_k_heads,
        )

        mixed_qkvz = mixed_qkvz.view(*new_tensor_shape_qkvz)
        mixed_ba = mixed_ba.view(*new_tensor_shape_ba)

        split_arg_list_qkvz = [
            self.head_k_dim,
            self.head_k_dim,
            (self.num_v_heads // self.num_k_heads * self.head_v_dim),
            (self.num_v_heads // self.num_k_heads * self.head_v_dim),
        ]
        split_arg_list_ba = [
            self.num_v_heads // self.num_k_heads,
            self.num_v_heads // self.num_k_heads,
        ]

        # [b, sq, ng, (hn + hn + np/ng * hn + np/ng + np/ng)]
        # --> [b, sq, ng, hn], [b, sq, ng, hn], [b, sq, ng, np/ng * hn], [b, sq, ng, np/ng * hn], [b, sq, ng, np/ng], [b, sq, ng, np/ng]
        query, key, value, z = torch.split(mixed_qkvz, split_arg_list_qkvz, dim=2)
# ... truncated for brevity ...
```
**EN:** This method implements `fix_query_key_value_ordering(mixed_qkvz: ..., mixed_ba: ...)` and Derives `query`, `key` and `value` tensors from `mixed_qkvzba`.
**CN:** 这个方法实现了 `fix_query_key_value_ordering(mixed_qkvz: ..., mixed_ba: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 358-383: Method `Qwen3GatedDeltaNet._forward_input_proj`
```python
    def _forward_input_proj(self, hidden_states: torch.Tensor):
        if (
            _is_cpu
            or _is_npu
            or not get_global_server_args().disable_piecewise_cuda_graph
        ):
            DUAL_STREAM_TOKEN_THRESHOLD = 0
        else:
            DUAL_STREAM_TOKEN_THRESHOLD = 1024

        seq_len, _ = hidden_states.shape
        if (
            self.alt_stream is not None
            and get_is_capture_mode()
            and seq_len < DUAL_STREAM_TOKEN_THRESHOLD
        ):
            current_stream = torch.cuda.current_stream()
            self.alt_stream.wait_stream(current_stream)
            projected_states_qkvz, _ = self.in_proj_qkvz(hidden_states)
            with torch.cuda.stream(self.alt_stream):
                projected_states_ba, _ = self.in_proj_ba(hidden_states)
            current_stream.wait_stream(self.alt_stream)
        else:
            projected_states_qkvz, _ = self.in_proj_qkvz(hidden_states)
            projected_states_ba, _ = self.in_proj_ba(hidden_states)
        return projected_states_qkvz, projected_states_ba
```
**EN:** This method implements `_forward_input_proj(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `_forward_input_proj(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 385-445: Method `Qwen3GatedDeltaNet.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ):
        projected_states_qkvz, projected_states_ba = self._forward_input_proj(
            hidden_states
        )

        if self.num_v_heads // self.num_k_heads in [1, 2, 4] and not _is_cpu:
            mixed_qkv, z, b, a = fused_qkvzba_split_reshape_cat(
                projected_states_qkvz,
                projected_states_ba,
                triton.cdiv(self.num_k_heads, self.attn_tp_size),
                triton.cdiv(self.num_v_heads, self.attn_tp_size),
                self.head_k_dim,
                self.head_v_dim,
            )
        elif _is_cpu and _is_amx_available:
            mixed_qkv, z, b, a = (
                torch.ops.sgl_kernel.fused_qkvzba_split_reshape_cat_cpu(
                    projected_states_qkvz,
                    projected_states_ba,
                    self.num_k_heads // self.attn_tp_size,
                    self.num_v_heads // self.attn_tp_size,
                    self.head_k_dim,
                    self.head_v_dim,
                )
            )
        else:
            query, key, value, z, b, a = self.fix_query_key_value_ordering(
                projected_states_qkvz, projected_states_ba
            )
            query, key, value = map(
                lambda x: x.reshape(x.shape[0], -1), (query, key, value)
            )
            mixed_qkv = torch.cat((query, key, value), dim=-1)
        core_attn_out = self.attn(
            forward_batch,
            mixed_qkv=mixed_qkv,
# ... truncated for brevity ...
```
**EN:** This method implements `forward(hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 448-487: Function `_apply_qwen3_next_mlp`
```python
def _apply_qwen3_next_mlp(
    layer: nn.Module,
    hidden_states: torch.Tensor,
    residual: Optional[torch.Tensor],
    forward_batch: ForwardBatch,
) -> Tuple[torch.Tensor, Optional[torch.Tensor]]:
    hidden_states, residual = layer.layer_communicator.prepare_mlp(
        hidden_states, residual, forward_batch
    )
    use_reduce_scatter = layer.layer_communicator.should_use_reduce_scatter(
        forward_batch
    )
    should_allreduce_fusion = (
        layer.layer_communicator.should_fuse_mlp_allreduce_with_next_layer(
            forward_batch
        )
    )

    if isinstance(layer.mlp, Qwen2MoeSparseMoeBlock):
        hidden_states = layer.mlp(
            hidden_states,
            forward_batch=forward_batch,
            use_reduce_scatter=use_reduce_scatter,
            should_allreduce_fusion=should_allreduce_fusion,
        )
    else:
        hidden_states = layer.mlp(
            hidden_states,
            should_allreduce_fusion=should_allreduce_fusion,
            use_reduce_scatter=use_reduce_scatter,
        )

    if should_allreduce_fusion:
        hidden_states._sglang_needs_allreduce_fusion = True
    else:
        hidden_states, residual = layer.layer_communicator.postprocess_layer(
            hidden_states, residual, forward_batch
        )

    return hidden_states, residual
```
**EN:** This function implements `_apply_qwen3_next_mlp(layer: ..., hidden_states: ..., residual: ..., forward_batch: ...)` and applies the feed-forward transformation used inside the network block.
**CN:** 这个函数实现了 `_apply_qwen3_next_mlp(layer: ..., hidden_states: ..., residual: ..., forward_batch: ...)`，其作用是执行网络块中的前馈变换。

### Lines 490-491: Class `Qwen3HybridLinearDecoderLayer` overview
```python
class Qwen3HybridLinearDecoderLayer(nn.Module):
```
**EN:** Defines `Qwen3HybridLinearDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3HybridLinearDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 492-547: Method `Qwen3HybridLinearDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: Qwen3NextConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
        is_nextn: bool = False,
    ) -> None:
        super().__init__()
        self.config = config
        self.linear_attn = Qwen3GatedDeltaNet(
            config, layer_id, quant_config, alt_stream, prefix
        )

        # Qwen3Next all layers are sparse and have no nextn now
        self.is_layer_sparse = True
        is_previous_layer_sparse = True
        is_next_layer_sparse = True
        self.layer_id = layer_id

        self.layer_scatter_modes = LayerScatterModes.init_new(
            layer_id=layer_id,
            num_layers=config.num_hidden_layers,
            is_layer_sparse=self.is_layer_sparse,
            is_previous_layer_sparse=is_previous_layer_sparse,
            is_next_layer_sparse=is_next_layer_sparse,
        )

        if self.is_layer_sparse:
            self.mlp = Qwen2MoeSparseMoeBlock(
                layer_id=layer_id,
                config=config,
                quant_config=quant_config,
                alt_stream=alt_stream,
                prefix=add_prefix("mlp", prefix.replace(".linear_attn", "")),
                is_nextn=is_nextn,
            )
        else:
            self.mlp = Qwen2MoeMLP(
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=..., is_nextn: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=..., is_nextn: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 549-576: Method `Qwen3HybridLinearDecoderLayer.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        residual: Optional[torch.Tensor],
        captured_last_layer_outputs: Optional[list[torch.Tensor]] = None,
        **kwargs,
    ):
        forward_batch = kwargs.get("forward_batch", None)

        hidden_states, residual = (
            self.layer_communicator.prepare_attn_and_capture_last_layer_outputs(
                hidden_states,
                residual,
                forward_batch,
                captured_last_layer_outputs=captured_last_layer_outputs,
            )
        )

        if not forward_batch.forward_mode.is_idle():
            hidden_states = self.linear_attn(
                hidden_states,
                forward_batch,
            )
        hidden_states, residual = _apply_qwen3_next_mlp(
            self, hidden_states, residual, forward_batch
        )

        return hidden_states, residual
```
**EN:** This method implements `forward(hidden_states: ..., residual: ..., captured_last_layer_outputs: ...=..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., residual: ..., captured_last_layer_outputs: ...=..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 579-580: Class `Qwen3HybridAttentionDecoderLayer` overview
```python
class Qwen3HybridAttentionDecoderLayer(nn.Module):
```
**EN:** Defines `Qwen3HybridAttentionDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3HybridAttentionDecoderLayer`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 581-720: Method `Qwen3HybridAttentionDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: Qwen3NextConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
        is_nextn: bool = False,
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.attn_tp_rank = get_attention_tp_rank()
        self.attn_tp_size = get_attention_tp_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % self.attn_tp_size == 0
        self.num_heads = self.total_num_heads // self.attn_tp_size
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= self.attn_tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % self.attn_tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert self.attn_tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // self.attn_tp_size)
        self.head_dim = config.head_dim or (self.hidden_size // self.num_heads)
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.rope_theta = getattr(config, "rope_theta", 10000)
        self.max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        if "rope_parameters" in config:
            self.rope_scaling = getattr(config, "rope_parameters", None)
        else:
            self.rope_scaling = getattr(config, "rope_scaling", None)
        self.partial_rotary_factor = config.partial_rotary_factor
        self.layer_id = layer_id

# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=..., is_nextn: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=..., is_nextn: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 722-742: Method `Qwen3HybridAttentionDecoderLayer._apply_qk_norm`
```python
    def _apply_qk_norm(
        self, q: torch.Tensor, k: torch.Tensor
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        # overlap qk norm
        if self.alt_stream is not None and get_is_capture_mode():
            current_stream = torch.cuda.current_stream()
            self.alt_stream.wait_stream(current_stream)
            q_by_head = q.reshape(-1, self.head_dim)
            q_by_head = self.q_norm(q_by_head)
            with torch.cuda.stream(self.alt_stream):
                k_by_head = k.reshape(-1, self.head_dim)
                k_by_head = self.k_norm(k_by_head)
            current_stream.wait_stream(self.alt_stream)
        else:
            q_by_head = q.reshape(-1, self.head_dim)
            q_by_head = self.q_norm(q_by_head)
            k_by_head = k.reshape(-1, self.head_dim)
            k_by_head = self.k_norm(k_by_head)
        q = q_by_head.view(q.shape)
        k = k_by_head.view(k.shape)
        return q, k
```
**EN:** This method implements `_apply_qk_norm(q: ..., k: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3HybridAttentionDecoderLayer`.
**CN:** 这个方法实现了 `_apply_qk_norm(q: ..., k: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3HybridAttentionDecoderLayer` 内部调用。

### Lines 744-775: Method `Qwen3HybridAttentionDecoderLayer.self_attention`
```python
    def self_attention(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)

        if self.attn_output_gate:
            q_gate, k, v = qkv.split(
                [self.q_size * 2, self.kv_size, self.kv_size], dim=-1
            )
            orig_shape = q_gate.shape[:-1]
            q_gate = q_gate.view(*orig_shape, self.num_heads, -1)
            q, gate = torch.chunk(q_gate, 2, dim=-1)
            q = q.reshape(*orig_shape, -1)
            gate = gate.reshape(*orig_shape, -1)
        else:
            q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)

        q, k = self._apply_qk_norm(q, k)

        q, k = self.rotary_emb(positions, q, k)

        attn_output = self.attn(q, k, v, forward_batch)

        if self.attn_output_gate:
            gate = torch.sigmoid(gate)
            attn_output = attn_output * gate

        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This method implements `self_attention(positions: ..., hidden_states: ..., forward_batch: ...)` and implements attention-related tensor preparation or execution.
**CN:** 这个方法实现了 `self_attention(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是实现与注意力相关的张量准备或执行逻辑。

### Lines 777-806: Method `Qwen3HybridAttentionDecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: Optional[torch.Tensor],
        forward_batch: ForwardBatch,
        captured_last_layer_outputs: Optional[list[torch.Tensor]] = None,
        **kwargs: Any,
    ):
        hidden_states, residual = (
            self.layer_communicator.prepare_attn_and_capture_last_layer_outputs(
                hidden_states,
                residual,
                forward_batch,
                captured_last_layer_outputs=captured_last_layer_outputs,
            )
        )

        if not forward_batch.forward_mode.is_idle():
            hidden_states = self.self_attention(
                positions=positions,
                hidden_states=hidden_states,
                forward_batch=forward_batch,
            )

        hidden_states, residual = _apply_qwen3_next_mlp(
            self, hidden_states, residual, forward_batch
        )

        return hidden_states, residual
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., residual: ..., forward_batch: ..., captured_last_layer_outputs: ...=..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., residual: ..., forward_batch: ..., captured_last_layer_outputs: ...=..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 809-812: Top-level assign
```python
ALL_DECODER_LAYER_TYPES = {
    "attention": Qwen3HybridAttentionDecoderLayer,
    "linear_attention": Qwen3HybridLinearDecoderLayer,
}
```
**EN:** Defines or updates ALL_DECODER_LAYER_TYPES, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 ALL_DECODER_LAYER_TYPES，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 815-815: Class `Qwen3NextModel` overview
```python
class Qwen3NextModel(nn.Module):
```
**EN:** Defines `Qwen3NextModel` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3NextModel`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 816-858: Method `Qwen3NextModel.__init__`
```python
    def __init__(
        self,
        config: Qwen3NextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        is_nextn: bool = False,
    ) -> None:
        super().__init__()
        self.config = config

        alt_stream = torch.cuda.Stream() if _is_cuda else None

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            org_num_embeddings=config.vocab_size,
            use_attn_tp_group=is_dp_attention_enabled(),
        )

        def get_layer(idx: int, prefix: str):
            layer_class = ALL_DECODER_LAYER_TYPES[config.layers_block_type[idx]]
            if config.layers_block_type[idx] == "attention":
                prefix = add_prefix("self_attn", prefix)
            else:
                prefix = add_prefix("linear_attn", prefix)
            return layer_class(
                config,
                idx,
                quant_config=quant_config,
                prefix=prefix,
                alt_stream=alt_stream,
                is_nextn=is_nextn,
            )

        self.layers = make_layers(
            config.num_hidden_layers, get_layer, prefix=f"{prefix}.layers"
        )

        self.norm = GemmaRMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.infer_count = 0
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., is_nextn: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., is_nextn: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 860-863: Method `Qwen3NextModel.set_eagle3_layers_to_capture`
```python
    def set_eagle3_layers_to_capture(self, layers_to_capture: list[int]):
        self.layers_to_capture = layers_to_capture
        for layer_id in self.layers_to_capture:
            setattr(self.layers[layer_id], "_is_layer_to_capture", True)
```
**EN:** This method implements `set_eagle3_layers_to_capture(layers_to_capture: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3NextModel`.
**CN:** 这个方法实现了 `set_eagle3_layers_to_capture(layers_to_capture: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3NextModel` 内部调用。

### Lines 865-868: Method `Qwen3NextModel.set_dflash_layers_to_capture`
```python
    def set_dflash_layers_to_capture(self, layers_to_capture: list[int]):
        self.layers_to_capture = layers_to_capture
        for layer_id in self.layers_to_capture:
            setattr(self.layers[layer_id], "_is_layer_to_capture", True)
```
**EN:** This method implements `set_dflash_layers_to_capture(layers_to_capture: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3NextModel`.
**CN:** 这个方法实现了 `set_dflash_layers_to_capture(layers_to_capture: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3NextModel` 内部调用。

### Lines 870-914: Method `Qwen3NextModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        # mamba_cache_params: MambaCacheParams,
        inputs_embeds: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:

        # pass a sequence index tensor, that is required for
        # proper continuous batching computation including
        # chunked prefill
        if inputs_embeds is not None:
            hidden_states = inputs_embeds
        else:
            hidden_states = self.embed_tokens(input_ids)

        residual = None
        aux_hidden_states = []
        for i in range(len(self.layers)):
            layer = self.layers[i]
            with get_global_expert_distribution_recorder().with_current_layer(i):
                hidden_states, residual = layer(
                    layer_id=i,
                    positions=positions,
                    hidden_states=hidden_states,
                    residual=residual,
                    forward_batch=forward_batch,
                    captured_last_layer_outputs=(
                        aux_hidden_states
                        if getattr(layer, "_is_layer_to_capture", False)
                        else None
                    ),
                )

        if not forward_batch.forward_mode.is_idle():
            if residual is None:
                hidden_states = self.norm(hidden_states)
            else:
                hidden_states, _ = self.norm(hidden_states, residual)
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 917-917: Class `HybridLayerType` overview
```python
class HybridLayerType(enum.Enum):
```
**EN:** Defines `HybridLayerType` as a reusable runtime type derived from enum.Enum. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `HybridLayerType`，其继承关系为 enum.Enum。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 918-921: Class `HybridLayerType` attributes
```python
    full_attention = "attention"
    swa_attention = "swa_attention"
    linear_attention = "linear_attention"
    mamba2 = "mamba"
```
**EN:** Defines class-level attributes and metadata that shape how `HybridLayerType` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `HybridLayerType` 在运行时的行为。

### Lines 924-924: Class `Qwen3NextForCausalLM` overview
```python
class Qwen3NextForCausalLM(nn.Module):
```
**EN:** Defines `Qwen3NextForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 12 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3NextForCausalLM`，其继承关系为 nn.Module。这个类组织了 12 个方法，用于实现模型相关行为。

### Lines 925-934: Class `Qwen3NextForCausalLM` attributes
```python
    fall_back_to_pt_during_load = False

    # Map fused module names to their checkpoint (unfused) counterparts.
    # This is needed so the quantization exclusion logic can match
    # checkpoint-style names (e.g. "q_proj") against the fused sglang
    # module names (e.g. "qkv_proj").
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3NextForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3NextForCausalLM` 在运行时的行为。

### Lines 936-976: Method `Qwen3NextForCausalLM.__init__`
```python
    def __init__(
        self,
        config: Qwen3NextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.pp_group = get_pp_group()
        assert self.pp_group.is_first_rank and self.pp_group.is_last_rank

        # The quant config's packed_modules_mapping may be None if it wasn't
        # in the checkpoint config. The base class (QuantizationConfig) intends
        # for models to set this. We need it so is_layer_skipped can unfuse
        # "qkv_proj" into ["q_proj","k_proj","v_proj"] when checking exclusions.
        if quant_config is not None and hasattr(quant_config, "packed_modules_mapping"):
            quant_config.packed_modules_mapping = self.packed_modules_mapping

        self.quant_config = quant_config
        self.model = Qwen3NextModel(
            config, quant_config, prefix=add_prefix("model", prefix)
        )
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            org_num_embeddings=config.vocab_size,
            prefix=add_prefix("lm_head", prefix),
            use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
        )
        self.logits_processor = LogitsProcessor(config)
        # For EAGLE3 support
        self.capture_aux_hidden_states = False

        self._routed_experts_weights_of_layer = LazyValue(
            lambda: {
                layer_id: layer.mlp.get_moe_weights()
                for layer_id, layer in enumerate(self.model.layers)
                if isinstance(layer.mlp, Qwen2MoeSparseMoeBlock)
            }
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 978-980: Method `Qwen3NextForCausalLM.routed_experts_weights_of_layer`
```python
    @property
    def routed_experts_weights_of_layer(self):
        return self._routed_experts_weights_of_layer.value
```
**EN:** This method implements `routed_experts_weights_of_layer()` and handles weight mapping, filtering, or loading for this model component Decorators: property.
**CN:** 这个方法实现了 `routed_experts_weights_of_layer()`，其作用是处理该模型组件的权重映射、筛选或加载逻辑 装饰器：property。

### Lines 982-999: Method `Qwen3NextForCausalLM.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        inputs_embeds: Optional[torch.Tensor] = None,
        **kwargs,
    ):
        hidden_states = self.model(input_ids, positions, forward_batch, inputs_embeds)

        aux_hidden_states = None
        if self.capture_aux_hidden_states:
            hidden_states, aux_hidden_states = hidden_states

        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch, aux_hidden_states
        )
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., inputs_embeds: ...=..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 1001-1002: Method `Qwen3NextForCausalLM.get_embed_and_head`
```python
    def get_embed_and_head(self):
        return self.model.embed_tokens.weight, self.lm_head.weight
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1004-1005: Method `Qwen3NextForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self) -> nn.Embedding:
        return self.model.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1007-1013: Method `Qwen3NextForCausalLM.set_embed_and_head`
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

### Lines 1015-1016: Method `Qwen3NextForCausalLM.get_embed`
```python
    def get_embed(self):
        return self.model.embed_tokens.weight
```
**EN:** This method implements `get_embed()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1018-1028: Method `Qwen3NextForCausalLM.set_embed`
```python
    def set_embed(self, embed):
        # NOTE: If draft hidden size != target hidden size, the embed weight cannot be shared for EAGLE3
        if (
            hasattr(self.config, "target_hidden_size")
            and self.config.target_hidden_size != self.config.hidden_size
        ):
            return
        del self.model.embed_tokens.weight
        self.model.embed_tokens.weight = embed
        torch.cuda.empty_cache()
        torch.cuda.synchronize()
```
**EN:** This method implements `set_embed(embed)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_embed(embed)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1030-1160: Method `Qwen3NextForCausalLM.load_weights`
```python
    def load_weights(
        self, weights: Iterable[Tuple[str, torch.Tensor]], is_mtp: bool = False
    ) -> Set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            # self attention
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            # mlp
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
            # GDN
            ("in_proj_qkvz.", "in_proj_qkv.", (0, 1, 2)),
            ("in_proj_qkvz.", "in_proj_z.", 3),
            ("in_proj_ba.", "in_proj_b.", 0),
            ("in_proj_ba.", "in_proj_a.", 1),
        ]

        # Params for weights, fp8 weight scales, fp8 activation scales
        # (param_name, weight_name, expert_id, shard_id)
        expert_params_mapping = FusedMoE.make_expert_params_mapping(
            ckpt_gate_proj_name="gate_proj",
            ckpt_down_proj_name="down_proj",
            ckpt_up_proj_name="up_proj",
            num_experts=self.config.num_experts,
        )

        params_dict = dict(self.named_parameters())
        loaded_params: Set[str] = set()
        for name, loaded_weight in weights:

            if is_mtp:

                if "mtp" not in name:
                    continue

                if name in [
                    "mtp.fc.weight",
                    "mtp.pre_fc_norm_embedding.weight",
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ..., is_mtp: ...=...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ..., is_mtp: ...=...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1162-1168: Method `Qwen3NextForCausalLM.get_model_config_for_expert_location`
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

### Lines 1170-1185: Method `Qwen3NextForCausalLM.set_eagle3_layers_to_capture`
```python
    def set_eagle3_layers_to_capture(self, layer_ids: Optional[list[int]] = None):
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
**EN:** This method implements `set_eagle3_layers_to_capture(layer_ids: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3NextForCausalLM`.
**CN:** 这个方法实现了 `set_eagle3_layers_to_capture(layer_ids: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3NextForCausalLM` 内部调用。

### Lines 1187-1197: Method `Qwen3NextForCausalLM.set_dflash_layers_to_capture`
```python
    def set_dflash_layers_to_capture(self, layer_ids: list[int]):
        if not self.pp_group.is_last_rank:
            return

        if layer_ids is None:
            raise ValueError(
                "DFLASH requires explicit layer_ids for aux hidden capture."
            )

        self.capture_aux_hidden_states = True
        self.model.set_dflash_layers_to_capture([val + 1 for val in layer_ids])
```
**EN:** This method implements `set_dflash_layers_to_capture(layer_ids: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3NextForCausalLM`.
**CN:** 这个方法实现了 `set_dflash_layers_to_capture(layer_ids: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3NextForCausalLM` 内部调用。

### Lines 1200-1200: Top-level assign
```python
EntryClass = Qwen3NextForCausalLM
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `enum`
- `logging`
- `typing.Any`
- `typing.Iterable`
- `typing.Optional`
- `typing.Set`
- `typing.Tuple`
- `torch`
- `triton`
- `torch.nn`
- `sglang.srt.configs.qwen3_next.Qwen3NextConfig`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location.ModelConfigForExpertLocation`
- `sglang.srt.layers.attention.fla.layernorm_gated.RMSNorm`
- `sglang.srt.layers.attention.mamba.mamba.mamba_v2_sharded_weight_loader`
- `sglang.srt.layers.communicator.LayerCommunicator`
- `sglang.srt.layers.communicator.LayerScatterModes`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
- `sglang.srt.layers.layernorm.GemmaRMSNorm`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.MergedColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.radix_linear_attention.RadixLinearAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.cuda_graph_runner.get_is_capture_mode`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.model_loader.weight_utils.sharded_weight_loader`
- `sglang.srt.models.qwen2_moe.Qwen2MoeMLP`
- `sglang.srt.models.qwen2_moe.Qwen2MoeSparseMoeBlock`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.LazyValue`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.cpu_has_amx_support`
- `sglang.srt.utils.is_cpu`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_npu`
- `sglang.srt.utils.make_layers`
- `sglang.srt.utils.set_weight_attrs`
- `sglang.jit_kernel.triton.gdn_fused_proj.fused_qkvzba_split_reshape_cat`
- `sglang.srt.layers.attention.fla.fused_norm_gate.FusedRMSNormGated`
- `sgl_kernel_npu.fla.utils.fused_qkvzba_split_reshape_cat`
