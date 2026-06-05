# qwen3_5.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/qwen3_5.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only Qwen3.5 model and Qwen3.5 MoE model compatible with HuggingFace weights. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 15-15: Module docstring
```python
"""Inference-only Qwen3.5 model and Qwen3.5 MoE model compatible with HuggingFace weights."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 17-99: Module imports
```python
import logging
from functools import lru_cache
from typing import Iterable, Optional, Set, Tuple, Union

import torch
import torch.nn as nn
import triton

from sglang.jit_kernel.triton.gdn_fused_proj import (
    fused_qkvzba_split_reshape_cat_contiguous,
)

# Configs
from sglang.srt.configs.qwen3_5 import (
    Qwen3_5Config,
    Qwen3_5MoeConfig,
    Qwen3_5TextConfig,
)

# Distributed
from sglang.srt.distributed import get_pp_group
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.eplb.expert_location import ModelConfigForExpertLocation

# Layers - Attention
from sglang.srt.layers.attention.fla.layernorm_gated import RMSNorm as RMSNormGated
from sglang.srt.layers.attention.mamba.mamba import mamba_v2_sharded_weight_loader
from sglang.srt.layers.communicator import LayerCommunicator, LayerScatterModes
from sglang.srt.layers.dp_attention import (
    get_attention_tp_rank,
    get_attention_tp_size,
    is_dp_attention_enabled,
)

# Layers - Others
from sglang.srt.layers.layernorm import GemmaRMSNorm

# Layers - Linear
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 101-101: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 102-102: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 103-103: Top-level assign
```python
_is_npu = is_npu()
```
**EN:** Defines or updates _is_npu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_npu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 104-104: Top-level assign
```python
_is_cpu = is_cpu()
```
**EN:** Defines or updates _is_cpu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cpu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 105-105: Top-level assign
```python
_is_gfx95 = is_gfx95_supported()
```
**EN:** Defines or updates _is_gfx95, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_gfx95，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 106-106: Top-level assign
```python
_is_hip = is_hip()
```
**EN:** Defines or updates _is_hip, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_hip，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 107-107: Top-level assign
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** Defines or updates _use_aiter, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _use_aiter，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 108-108: Top-level assign
```python
_is_amx_available = cpu_has_amx_support()
```
**EN:** Defines or updates _is_amx_available, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_amx_available，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 110-110: Top-level assign
```python
cached_get_processor = lru_cache(get_processor)
```
**EN:** Defines or updates cached_get_processor, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 cached_get_processor，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 113-113: Class `Qwen3_5GatedDeltaNet` overview
```python
class Qwen3_5GatedDeltaNet(nn.Module):
```
**EN:** Defines `Qwen3_5GatedDeltaNet` as a reusable runtime type derived from nn.Module. The class groups 10 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3_5GatedDeltaNet`，其继承关系为 nn.Module。这个类组织了 10 个方法，用于实现模型相关行为。

### Lines 114-252: Method `Qwen3_5GatedDeltaNet.__init__`
```python
    def __init__(
        self,
        config: Qwen3_5TextConfig,
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

        # Conv1d layer
        self.conv_dim = self.key_dim * 2 + self.value_dim
        self.conv1d = ColumnParallelLinear(
            input_size=self.conv_kernel_size,
            output_size=self.conv_dim,
            bias=False,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., alt_stream: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., alt_stream: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 254-274: Method `Qwen3_5GatedDeltaNet._override_weight_loader`
```python
    @staticmethod
    def _override_weight_loader(param, loader):
        """Robustly override loader for:
        1) BasevLLMParameter subclasses: real storage is `_weight_loader`
        2) regular Parameters that already have mutable `weight_loader`
        3) regular Parameters without `weight_loader` yet
        """
        if hasattr(param, "_weight_loader"):
            # FP8 / quantized BasevLLMParameter path
            param._weight_loader = loader
            return

        if hasattr(param, "weight_loader"):
            # Regular parameter/tensor that already has a mutable attr.
            # Do NOT call set_weight_attrs here, because it asserts when
            # overwriting an existing attribute.
            param.weight_loader = loader
            return

        # Fresh attribute on a normal tensor/Parameter
        set_weight_attrs(param, {"weight_loader": loader})
```
**EN:** This method implements `_override_weight_loader(loader)` and Robustly override loader for:. Decorators: staticmethod.
**CN:** 这个方法实现了 `_override_weight_loader(loader)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：staticmethod。

### Lines 276-286: Method `Qwen3_5GatedDeltaNet._bind_packed_weight_loaders`
```python
    def _bind_packed_weight_loaders(self, module):
        """Bind packed-checkpoint-aware loaders to all relevant params of a merged module."""
        for attr_name in ("weight", "weight_scale_inv", "weight_scale", "input_scale"):
            param = getattr(module, attr_name, None)
            if param is None:
                continue
            original_loader = getattr(param, "weight_loader", None)
            if original_loader is None:
                continue
            wrapped_loader = self._make_packed_weight_loader(module, original_loader)
            self._override_weight_loader(param, wrapped_loader)
```
**EN:** This method implements `_bind_packed_weight_loaders(module)` and Bind packed-checkpoint-aware loaders to all relevant params of a merged module.
**CN:** 这个方法实现了 `_bind_packed_weight_loaders(module)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 288-305: Method `Qwen3_5GatedDeltaNet._get_split_sizes_for_param`
```python
    @staticmethod
    def _get_split_sizes_for_param(module, param, loaded_shard_id):
        """Return checkpoint-side split sizes for this param type."""
        if isinstance(param, BlockQuantScaleParameter):
            # Split by output blocks, not raw output sizes.
            block_n, _ = module.quant_method.quant_config.weight_block_size
            block_n = 1 if getattr(param, "format_ue8m0", False) else block_n
            return [
                (module.output_sizes[idx] + block_n - 1) // block_n
                for idx in loaded_shard_id
            ]

        if isinstance(param, PerTensorScaleParameter):
            # One logical scale per logical shard.
            return [1 for _ in loaded_shard_id]

        # Normal weight / non-block quant tensor
        return [module.output_sizes[idx] for idx in loaded_shard_id]
```
**EN:** This method implements `_get_split_sizes_for_param(param, loaded_shard_id)` and Return checkpoint-side split sizes for this param type. Decorators: staticmethod.
**CN:** 这个方法实现了 `_get_split_sizes_for_param(param, loaded_shard_id)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：staticmethod。

### Lines 307-356: Method `Qwen3_5GatedDeltaNet._make_packed_weight_loader`
```python
    @classmethod
    def _make_packed_weight_loader(cls, module, original_weight_loader):
        """Wrap the param's original loader so split checkpoints:
          - in_proj_qkv + in_proj_z -> merged in_proj_qkvz
          - in_proj_b + in_proj_a   -> merged in_proj_ba
        can load correctly for both normal and FP8 params.
        """

        def weight_loader(param, loaded_weight, loaded_shard_id=None):
            # Only intercept split-checkpoint tuple shards.
            # int shard_id and None should preserve original behavior.
            if isinstance(loaded_shard_id, tuple):
                split_sizes = cls._get_split_sizes_for_param(
                    module, param, loaded_shard_id
                )

                if loaded_weight.numel() == 1:
                    # Single-element tensor (scalar or [1]):
                    # broadcast to each logical shard.
                    chunks = [loaded_weight.view(-1)] * len(loaded_shard_id)
                else:
                    split_dim = getattr(param, "output_dim", 0)
                    if _is_cpu:
                        cpu_split_sizes = []
                        split_size_sum = sum(split_sizes)
                        target_size_sim = loaded_weight.size(split_dim)
                        for i in range(len(split_sizes)):
                            cpu_split_sizes.append(
                                int(target_size_sim * split_sizes[i] / split_size_sum)
                            )
                        assert (
                            sum(cpu_split_sizes) == target_size_sim
                        ), f"Padding the loaded weight failed due to sizes are not divisible cleanly from {cpu_split_sizes} to {target_size_sim}"
                        chunks = loaded_weight.split(cpu_split_sizes, dim=split_dim)
                    else:
                        chunks = loaded_weight.split(split_sizes, dim=split_dim)

                assert len(chunks) == len(loaded_shard_id), (
                    f"Chunk/shard mismatch: {len(chunks)=}, "
                    f"{len(loaded_shard_id)=}, {split_sizes=}"
# ... truncated for brevity ...
```
**EN:** This method implements `_make_packed_weight_loader(module, original_weight_loader)` and Wrap the param's original loader so split checkpoints:. Decorators: classmethod.
**CN:** 这个方法实现了 `_make_packed_weight_loader(module, original_weight_loader)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：classmethod。

### Lines 358-376: Method `Qwen3_5GatedDeltaNet.create_qkvz_proj`
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
**EN:** This method implements `create_qkvz_proj(hidden_size: ..., key_dim: ..., value_dim: ..., quant_config: ..., prefix: ..., tp_rank: ...=..., ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3_5GatedDeltaNet`.
**CN:** 这个方法实现了 `create_qkvz_proj(hidden_size: ..., key_dim: ..., value_dim: ..., quant_config: ..., prefix: ..., tp_rank: ...=..., ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3_5GatedDeltaNet` 内部调用。

### Lines 378-398: Method `Qwen3_5GatedDeltaNet.create_ba_proj`
```python
    def create_ba_proj(
        self,
        hidden_size: int,
        num_v_heads: int,
        quant_config: QuantizationConfig | None,
        prefix: str,
        tp_rank: Optional[int] = None,
        tp_size: Optional[int] = None,
    ) -> MergedColumnParallelLinear:
        # Qwen3.5 has separate in_proj_b and in_proj_a weights in the
        # checkpoint, which are loaded into the fused in_proj_ba parameter
        # via stacked_params_mapping with shard_id 0 and 1 respectively.
        return MergedColumnParallelLinear(
            input_size=hidden_size,
            output_sizes=[num_v_heads, num_v_heads],
            bias=False,
            quant_config=quant_config,
            prefix=prefix,
            tp_rank=tp_rank,
            tp_size=tp_size,
        )
```
**EN:** This method implements `create_ba_proj(hidden_size: ..., num_v_heads: ..., quant_config: ..., prefix: ..., tp_rank: ...=..., tp_size: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3_5GatedDeltaNet`.
**CN:** 这个方法实现了 `create_ba_proj(hidden_size: ..., num_v_heads: ..., quant_config: ..., prefix: ..., tp_rank: ...=..., tp_size: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3_5GatedDeltaNet` 内部调用。

### Lines 400-420: Method `Qwen3_5GatedDeltaNet.fix_query_key_value_ordering`
```python
    def fix_query_key_value_ordering(
        self,
        mixed_qkvz: torch.Tensor,
        mixed_ba: torch.Tensor,
    ):
        """
        Derives `query`, `key` and `value` tensors from `mixed_qkvzba`.
        """
        k_tp = self.key_dim // self.attn_tp_size
        v_tp = self.value_dim // self.attn_tp_size
        nv_tp = self.num_v_heads // self.attn_tp_size

        # Directly split, no head group reshape
        query, key, value, z = mixed_qkvz.split([k_tp, k_tp, v_tp, v_tp], dim=-1)
        b, a = mixed_ba.split([nv_tp, nv_tp], dim=-1)

        # value / z reshape to (seq, num_v_heads/tp, head_v_dim)
        value = value.reshape(value.size(0), -1, self.head_v_dim)
        z = z.reshape(z.size(0), -1, self.head_v_dim)

        return query, key, value, z, b, a
```
**EN:** This method implements `fix_query_key_value_ordering(mixed_qkvz: ..., mixed_ba: ...)` and Derives `query`, `key` and `value` tensors from `mixed_qkvzba`.
**CN:** 这个方法实现了 `fix_query_key_value_ordering(mixed_qkvz: ..., mixed_ba: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 422-447: Method `Qwen3_5GatedDeltaNet._forward_input_proj`
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

### Lines 449-523: Method `Qwen3_5GatedDeltaNet.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ):
        """
        Forward pass with three parts:
        1. Input projection
        2. Core attention (custom op)
        3. Output projection
        """
        projected_states_qkvz, projected_states_ba = self._forward_input_proj(
            hidden_states
        )

        if (
            self.num_v_heads // self.num_k_heads in [1, 2, 4]
            and not _is_cpu
            and not _is_npu
        ):
            mixed_qkv, z, b, a = fused_qkvzba_split_reshape_cat_contiguous(
                projected_states_qkvz,
                projected_states_ba,
                triton.cdiv(self.num_k_heads, self.attn_tp_size),
                triton.cdiv(self.num_v_heads, self.attn_tp_size),
                self.head_k_dim,
                self.head_v_dim,
            )
        elif _is_cpu and _is_amx_available:
            mixed_qkv, z, b, a = (
                torch.ops.sgl_kernel.fused_qkvzba_split_reshape_cat_contiguous_cpu(
                    projected_states_qkvz,
                    projected_states_ba,
                    self.num_k_heads // self.attn_tp_size,
                    self.num_v_heads // self.attn_tp_size,
                    self.head_k_dim,
                    self.head_v_dim,
                )
            )
        else:
# ... truncated for brevity ...
```
**EN:** This method implements `forward(hidden_states: ..., forward_batch: ...)` and Forward pass with three parts:.
**CN:** 这个方法实现了 `forward(hidden_states: ..., forward_batch: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 526-528: Class `Qwen3_5LinearDecoderLayer` overview
```python
class Qwen3_5LinearDecoderLayer(nn.Module):
    """Qwen3.5 Decoder Layer with Linear Attention (GatedDeltaNet)."""
```
**EN:** Defines `Qwen3_5LinearDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3_5LinearDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 529-598: Method `Qwen3_5LinearDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: Qwen3_5TextConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
        is_nextn: bool = False,
    ) -> None:
        super().__init__()
        self.config = config
        self.layer_id = layer_id

        linear_attn_quant_config = (
            None
            if quant_config and quant_config.get_name() == "modelopt_fp4"
            else quant_config
        )
        self.linear_attn = Qwen3_5GatedDeltaNet(
            config, layer_id, linear_attn_quant_config, alt_stream, prefix
        )

        # NOTE: Determine the MLP type based on the model type
        # Qwen3.5 use all layers for MLP / Qwen3.5-MoE use sparse MoE blocks
        if config.model_type == "qwen3_5_moe_text":
            self.mlp = Qwen2MoeSparseMoeBlock(
                layer_id=layer_id,
                config=config,
                quant_config=quant_config,
                alt_stream=alt_stream,
                prefix=add_prefix("mlp", prefix.replace(".linear_attn", "")),
                is_nextn=is_nextn,
                support_shared_expert_fusion=True,
            )
            is_layer_sparse = True
            is_previous_layer_sparse = True
            is_next_layer_sparse = True
        elif config.model_type == "qwen3_5_text":
            self.mlp = Qwen2MoeMLP(
                hidden_size=config.hidden_size,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=..., is_nextn: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=..., is_nextn: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 600-657: Method `Qwen3_5LinearDecoderLayer.forward`
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        residual: Optional[torch.Tensor],
        **kwargs,
    ):
        forward_batch = kwargs.get("forward_batch", None)

        hidden_states, residual = (
            self.layer_communicator.prepare_attn_and_capture_last_layer_outputs(
                hidden_states,
                residual,
                forward_batch,
                captured_last_layer_outputs=kwargs.get(
                    "captured_last_layer_outputs", None
                ),
            )
        )

        if not forward_batch.forward_mode.is_idle():
            hidden_states = self.linear_attn(
                hidden_states,
                forward_batch,
            )

        # Fully Connected
        hidden_states, residual = self.layer_communicator.prepare_mlp(
            hidden_states, residual, forward_batch
        )

        use_reduce_scatter = self.layer_communicator.should_use_reduce_scatter(
            forward_batch
        )

        should_allreduce_fusion = (
            self.layer_communicator.should_fuse_mlp_allreduce_with_next_layer(
                forward_batch
            )
        )
        if isinstance(self.mlp, Qwen2MoeSparseMoeBlock):
# ... truncated for brevity ...
```
**EN:** This method implements `forward(hidden_states: ..., residual: ..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., residual: ..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 660-662: Class `Qwen3_5AttentionDecoderLayer` overview
```python
class Qwen3_5AttentionDecoderLayer(nn.Module):
    """Qwen3.5 Decoder Layer with Full Attention."""
```
**EN:** Defines `Qwen3_5AttentionDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3_5AttentionDecoderLayer`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 663-805: Method `Qwen3_5AttentionDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: Qwen3_5TextConfig,
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
            assert self.total_num_kv_heads % self.attn_tp_size == 0
        else:
            assert self.attn_tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // self.attn_tp_size)
        self.head_dim = config.head_dim or (self.hidden_size // self.num_heads)
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.max_position_embeddings = getattr(config, "max_position_embeddings", 8192)

        self.rope_theta, rope_scaling = get_rope_config(config)
        self.partial_rotary_factor = getattr(config, "partial_rotary_factor", 1.0)
        self.layer_id = layer_id

        # If rope_scaling doesn't specify a scaling type, treat as no scaling
        if rope_scaling and not ("rope_type" in rope_scaling or "type" in rope_scaling):
            rope_scaling = None

        self.attn_output_gate = getattr(config, "attn_output_gate", True)
        if self.attn_output_gate:
            logger.warning_once("using attn output gate!")
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=..., is_nextn: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=..., is_nextn: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 807-836: Method `Qwen3_5AttentionDecoderLayer._apply_qk_norm`
```python
    def _apply_qk_norm(
        self, q: torch.Tensor, k: torch.Tensor
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        """Apply Q/K normalization with optional alt_stream overlap."""
        if self.alt_stream is not None and get_is_capture_mode():
            current_stream = torch.cuda.current_stream()
            self.alt_stream.wait_stream(current_stream)
            q_by_head = q.reshape(-1, self.head_dim)
            q_by_head = self.q_norm(q_by_head)
            with torch.cuda.stream(self.alt_stream):
                k_by_head = k.reshape(-1, self.head_dim)
                k_by_head = self.k_norm(k_by_head)
            current_stream.wait_stream(self.alt_stream)
        elif _is_hip:
            q_by_head, k_by_head = fused_qk_gemma_rmsnorm(
                q,
                k,
                self.q_norm.weight.data,
                self.k_norm.weight.data,
                self.q_norm.variance_epsilon,
                self.head_dim,
            )
        else:
            q_by_head = q.reshape(-1, self.head_dim)
            q_by_head = self.q_norm(q_by_head)
            k_by_head = k.reshape(-1, self.head_dim)
            k_by_head = self.k_norm(k_by_head)
        q = q_by_head.view(q.shape)
        k = k_by_head.view(k.shape)
        return q, k
```
**EN:** This method implements `_apply_qk_norm(q: ..., k: ...)` and Apply Q/K normalization with optional alt_stream overlap.
**CN:** 这个方法实现了 `_apply_qk_norm(q: ..., k: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 838-868: Method `Qwen3_5AttentionDecoderLayer.self_attention`
```python
    def self_attention(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        """Full attention forward pass."""
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
**EN:** This method implements `self_attention(positions: ..., hidden_states: ..., forward_batch: ...)` and Full attention forward pass.
**CN:** 这个方法实现了 `self_attention(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 870-926: Method `Qwen3_5AttentionDecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        residual: Optional[torch.Tensor],
        forward_batch: ForwardBatch,
        captured_last_layer_outputs: Optional[list[torch.Tensor]] = None,
        **kwargs,
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

        # Fully Connected
        hidden_states, residual = self.layer_communicator.prepare_mlp(
            hidden_states, residual, forward_batch
        )
        use_reduce_scatter = self.layer_communicator.should_use_reduce_scatter(
            forward_batch
        )

        should_allreduce_fusion = (
            self.layer_communicator.should_fuse_mlp_allreduce_with_next_layer(
                forward_batch
            )
        )
        if isinstance(self.mlp, Qwen2MoeSparseMoeBlock):
            hidden_states = self.mlp(
# ... truncated for brevity ...
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., residual: ..., forward_batch: ..., captured_last_layer_outputs: ...=..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., residual: ..., forward_batch: ..., captured_last_layer_outputs: ...=..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 929-932: Top-level assign
```python
ALL_DECODER_LAYER_TYPES = {
    "attention": Qwen3_5AttentionDecoderLayer,
    "linear_attention": Qwen3_5LinearDecoderLayer,
}
```
**EN:** Defines or updates ALL_DECODER_LAYER_TYPES, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 ALL_DECODER_LAYER_TYPES，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 935-937: Class `Qwen3_5ForCausalLM` overview
```python
class Qwen3_5ForCausalLM(nn.Module):
    """Qwen3.5 Model with support for dense variant."""
```
**EN:** Defines `Qwen3_5ForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 9 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3_5ForCausalLM`，其继承关系为 nn.Module。这个类组织了 9 个方法，用于实现模型相关行为。

### Lines 938-953: Class `Qwen3_5ForCausalLM` attributes
```python
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
        "in_proj_qkvz": ["in_proj_qkv", "in_proj_z"],
        "in_proj_ba": ["in_proj_b", "in_proj_a"],
    }

    supported_lora_modules = [
        "qkv_proj",
        "o_proj",
        "out_proj",
        "in_proj_qkvz",
        "gate_up_proj",
        "down_proj",
        "lm_head",
    ]
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3_5ForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3_5ForCausalLM` 在运行时的行为。

### Lines 955-1002: Method `Qwen3_5ForCausalLM.get_hidden_dim`
```python
    def get_hidden_dim(self, module_name: str, layer_idx: int):
        config = self.config
        head_dim = config.head_dim or (config.hidden_size // config.num_attention_heads)

        if module_name == "qkv_proj":
            attn_output_gate = getattr(config, "attn_output_gate", True)
            q_heads = config.num_attention_heads * (2 if attn_output_gate else 1)
            return (
                config.hidden_size,
                head_dim * (q_heads + config.num_key_value_heads * 2),
            )
        elif module_name == "o_proj":
            return config.num_attention_heads * head_dim, config.hidden_size
        elif module_name == "out_proj":
            value_dim = config.linear_value_head_dim * config.linear_num_value_heads
            return value_dim, config.hidden_size
        elif module_name == "in_proj_qkvz":
            key_dim = config.linear_key_head_dim * config.linear_num_key_heads
            value_dim = config.linear_value_head_dim * config.linear_num_value_heads
            return config.hidden_size, key_dim * 2 + value_dim * 2
        elif module_name == "gate_up_proj":
            # MoE: shared expert uses shared_expert_intermediate_size
            # Dense: regular MLP uses intermediate_size
            is_moe = "moe" in getattr(config, "model_type", "")
            if is_moe:
                inter = config.shared_expert_intermediate_size
            else:
                inter = config.intermediate_size
            return config.hidden_size, inter * 2
        elif module_name == "down_proj":
            is_moe = "moe" in getattr(config, "model_type", "")
            if is_moe:
                inter = config.shared_expert_intermediate_size
            else:
                inter = config.intermediate_size
            return inter, config.hidden_size
        elif module_name == "gate_up_proj_moe":
            return config.hidden_size, config.moe_intermediate_size * 2
        elif module_name == "down_proj_moe":
            return config.moe_intermediate_size, config.hidden_size
# ... truncated for brevity ...
```
**EN:** This method implements `get_hidden_dim(module_name: ..., layer_idx: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_hidden_dim(module_name: ..., layer_idx: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1004-1060: Method `Qwen3_5ForCausalLM.__init__`
```python
    def __init__(
        self,
        config: Qwen3_5TextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        is_nextn: bool = False,
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.pp_group = get_pp_group()

        alt_stream = torch.cuda.Stream() if _is_cuda else None

        # Embedding layer
        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                org_num_embeddings=config.vocab_size,
                enable_tp=not is_dp_attention_enabled(),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        # Decoder layers
        def get_layer(idx: int, prefix: str):
            layer_type = config.layers_block_type[idx]
            layer_class = ALL_DECODER_LAYER_TYPES[layer_type]
            if layer_type == "attention":
                prefix = add_prefix("self_attn", prefix)
            else:
                prefix = add_prefix("linear_attn", prefix)
            return layer_class(
                config=config,
                layer_id=idx,
                quant_config=quant_config,
                prefix=prefix,
                alt_stream=alt_stream,
                is_nextn=is_nextn,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., is_nextn: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., is_nextn: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1062-1063: Method `Qwen3_5ForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self):
        return self.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1065-1068: Method `Qwen3_5ForCausalLM.set_dflash_layers_to_capture`
```python
    def set_dflash_layers_to_capture(self, layers_to_capture: list[int]):
        self.layers_to_capture = layers_to_capture
        for layer_id in self.layers_to_capture:
            setattr(self.layers[layer_id], "_is_layer_to_capture", True)
```
**EN:** This method implements `set_dflash_layers_to_capture(layers_to_capture: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3_5ForCausalLM`.
**CN:** 这个方法实现了 `set_dflash_layers_to_capture(layers_to_capture: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3_5ForCausalLM` 内部调用。

### Lines 1070-1072: Method `Qwen3_5ForCausalLM.start_layer`
```python
    @property
    def start_layer(self) -> int:
        return self._start_layer
```
**EN:** This method implements `start_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3_5ForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `start_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3_5ForCausalLM` 内部调用 装饰器：property。

### Lines 1074-1076: Method `Qwen3_5ForCausalLM.end_layer`
```python
    @property
    def end_layer(self) -> int:
        return self._end_layer
```
**EN:** This method implements `end_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3_5ForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `end_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3_5ForCausalLM` 内部调用 装饰器：property。

### Lines 1078-1149: Method `Qwen3_5ForCausalLM.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: Optional[torch.Tensor] = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
        input_deepstack_embeds: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, PPProxyTensors]:
        # Initialize hidden states
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

        aux_hidden_states = []
        # Pass through decoder layers
        for layer_idx in range(self.start_layer, self.end_layer):
            layer = self.layers[layer_idx]
            with get_global_expert_distribution_recorder().with_current_layer(
                layer_idx
            ):
                hidden_states, residual = layer(
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
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=..., input_deepstack_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=..., input_deepstack_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 1151-1219: Method `Qwen3_5ForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
            # GDN
            ("in_proj_qkvz.", "in_proj_qkv.", (0, 1, 2)),
            ("in_proj_qkvz.", "in_proj_z.", 3),
            ("in_proj_ba.", "in_proj_b.", 0),
            ("in_proj_ba.", "in_proj_a.", 1),
        ]

        loaded_params: Set[str] = set()
        params_dict = dict(self.named_parameters(remove_duplicate=False))
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            if "mtp" in name:
                continue
            if "visual" in name:
                continue
            if "language_model" in name:
                name = name.replace(r"model.language_model.", r"model.")
            if ".self_attn." in name:
                name = name.replace(".self_attn", "")
            layer_id = get_layer_id(name)
            if (
                layer_id is not None
                and hasattr(self, "start_layer")
                and (layer_id < self.start_layer or layer_id >= self.end_layer)
            ):
                continue

            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue

# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1221-1227: Method `Qwen3_5ForCausalLM.get_model_config_for_expert_location`
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

### Lines 1230-1230: Class `Qwen3_5MoeForCausalLM` overview
```python
class Qwen3_5MoeForCausalLM(Qwen3_5ForCausalLM):
```
**EN:** Defines `Qwen3_5MoeForCausalLM` as a reusable runtime type derived from Qwen3_5ForCausalLM. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3_5MoeForCausalLM`，其继承关系为 Qwen3_5ForCausalLM。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 1231-1237: Method `Qwen3_5MoeForCausalLM.__init__`
```python
    def __init__(
        self,
        config: Qwen3_5TextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__(config=config, quant_config=quant_config, prefix=prefix)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1239-1437: Method `Qwen3_5MoeForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
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

        # Skip loading extra parameters for GPTQ/modelopt models.
        ignore_suffixes = (
            ".bias",
            "_bias",
            ".k_scale",
            "_k_scale",
            ".v_scale",
            "_v_scale",
            ".weight_scale",
            "_weight_scale",
            ".input_scale",
            "_input_scale",
        )

        is_fused_expert = False
        fused_expert_params_mapping = [
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1440-1441: Class `Qwen3_5ForConditionalGeneration` overview
```python
class Qwen3_5ForConditionalGeneration(Qwen3VLForConditionalGeneration):
```
**EN:** Defines `Qwen3_5ForConditionalGeneration` as a reusable runtime type derived from Qwen3VLForConditionalGeneration. The class groups 8 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3_5ForConditionalGeneration`，其继承关系为 Qwen3VLForConditionalGeneration。这个类组织了 8 个方法，用于实现模型相关行为。

### Lines 1442-1445: Class `Qwen3_5ForConditionalGeneration` attributes
```python
    packed_modules_mapping = Qwen3_5ForCausalLM.packed_modules_mapping
    hf_to_sglang_mapper = None

    supported_lora_modules = Qwen3_5ForCausalLM.supported_lora_modules
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3_5ForConditionalGeneration` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3_5ForConditionalGeneration` 在运行时的行为。

### Lines 1447-1461: Method `Qwen3_5ForConditionalGeneration.__init__`
```python
    def __init__(
        self,
        config: Qwen3_5Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        language_model_cls=Qwen3_5ForCausalLM,
    ):
        super().__init__(config, quant_config, prefix, language_model_cls)

        rope_config = getattr(self.config, "rope_parameters", None) or getattr(
            self.config, "rope_scaling", {}
        )
        self.is_mrope_enabled = "mrope_section" in rope_config

        self.deepstack_visual_indexes = self.visual.deepstack_visual_indexes
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., language_model_cls=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., language_model_cls=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1463-1464: Method `Qwen3_5ForConditionalGeneration.get_hidden_dim`
```python
    def get_hidden_dim(self, module_name: str, layer_idx: int):
        return self.model.get_hidden_dim(module_name, layer_idx)
```
**EN:** This method implements `get_hidden_dim(module_name: ..., layer_idx: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_hidden_dim(module_name: ..., layer_idx: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1466-1467: Method `Qwen3_5ForConditionalGeneration.should_apply_lora`
```python
    def should_apply_lora(self, module_name: str) -> bool:
        return module_name.startswith("model.layers.")
```
**EN:** This method implements `should_apply_lora(module_name: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3_5ForConditionalGeneration`.
**CN:** 这个方法实现了 `should_apply_lora(module_name: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3_5ForConditionalGeneration` 内部调用。

### Lines 1469-1471: Method `Qwen3_5ForConditionalGeneration.start_layer`
```python
    @property
    def start_layer(self) -> int:
        return getattr(getattr(self, "model", None), "start_layer", 0)
```
**EN:** This method implements `start_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3_5ForConditionalGeneration` Decorators: property.
**CN:** 这个方法实现了 `start_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3_5ForConditionalGeneration` 内部调用 装饰器：property。

### Lines 1473-1480: Method `Qwen3_5ForConditionalGeneration.end_layer`
```python
    @property
    def end_layer(self) -> int:
        model = getattr(self, "model", None)
        end_layer = getattr(model, "end_layer", None)
        if end_layer is not None:
            return end_layer
        cfg = getattr(model, "config", None)
        return int(getattr(cfg, "num_hidden_layers", 0))
```
**EN:** This method implements `end_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3_5ForConditionalGeneration` Decorators: property.
**CN:** 这个方法实现了 `end_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3_5ForConditionalGeneration` 内部调用 装饰器：property。

### Lines 1482-1485: Method `Qwen3_5ForConditionalGeneration.get_embed_and_head`
```python
    def get_embed_and_head(self):
        embed = self.model.embed_tokens.weight if self.pp_group.is_first_rank else None
        head = self.lm_head.weight if self.pp_group.is_last_rank else None
        return embed, head
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1487-1495: Method `Qwen3_5ForConditionalGeneration.set_embed_and_head`
```python
    def set_embed_and_head(self, embed, head):
        if self.pp_group.is_first_rank and embed is not None:
            del self.model.embed_tokens.weight
            self.model.embed_tokens.weight = embed
        if self.pp_group.is_last_rank and head is not None:
            del self.lm_head.weight
            self.lm_head.weight = head
        torch.cuda.empty_cache()
        torch.cuda.synchronize()
```
**EN:** This method implements `set_embed_and_head(embed, head)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_embed_and_head(embed, head)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1497-1590: Method `Qwen3_5ForConditionalGeneration.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
            # GDN fused projections
            ("in_proj_qkvz.", "in_proj_qkv.", (0, 1, 2)),
            ("in_proj_qkvz.", "in_proj_z.", 3),
            ("in_proj_ba.", "in_proj_b.", 0),
            ("in_proj_ba.", "in_proj_a.", 1),
        ]

        loaded_params: Set[str] = set()
        params_dict = dict(self.named_parameters(remove_duplicate=False))
        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            if "mtp" in name:
                continue
            if "language_model" in name:
                name = name.replace(r"model.language_model.", r"model.")
            if ".self_attn." in name:
                name = name.replace(".self_attn", "")
            if (
                self.config.tie_word_embeddings
                and self.pp_group.is_last_rank
                and "model.embed_tokens.weight" in name
            ):
                if "lm_head.weight" in params_dict:
                    lm_head_param = params_dict["lm_head.weight"]
                    weight_loader = getattr(
                        lm_head_param, "weight_loader", default_weight_loader
                    )
                    weight_loader(lm_head_param, loaded_weight)
            layer_id = get_layer_id(name)
            if (
                layer_id is not None
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1593-1595: Class `Qwen3_5MoeForConditionalGeneration` overview
```python
class Qwen3_5MoeForConditionalGeneration(Qwen3VLForConditionalGeneration):
    """Qwen3.5 MoE Vision-Language Model."""
```
**EN:** Defines `Qwen3_5MoeForConditionalGeneration` as a reusable runtime type derived from Qwen3VLForConditionalGeneration. The class groups 9 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3_5MoeForConditionalGeneration`，其继承关系为 Qwen3VLForConditionalGeneration。这个类组织了 9 个方法，用于实现模型相关行为。

### Lines 1596-1599: Class `Qwen3_5MoeForConditionalGeneration` attributes
```python
    packed_modules_mapping = Qwen3_5ForCausalLM.packed_modules_mapping
    hf_to_sglang_mapper = None

    supported_lora_modules = Qwen3_5ForCausalLM.supported_lora_modules
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3_5MoeForConditionalGeneration` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3_5MoeForConditionalGeneration` 在运行时的行为。

### Lines 1601-1619: Method `Qwen3_5MoeForConditionalGeneration.__init__`
```python
    def __init__(
        self,
        config: Qwen3_5MoeConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        language_model_cls=Qwen3_5MoeForCausalLM,
    ) -> None:
        super().__init__(config, quant_config, prefix, language_model_cls)
        rope_config = getattr(self.config, "rope_parameters", None) or getattr(
            self.config, "rope_scaling", {}
        )
        self.is_mrope_enabled = "mrope_section" in rope_config

        self.deepstack_visual_indexes = self.visual.deepstack_visual_indexes
        self.num_fused_shared_experts = 0
        if _use_aiter:
            self.num_fused_shared_experts = self._get_num_fused_shared_experts()

        self.enable_shared_expert_fusion = self.num_fused_shared_experts > 0
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., language_model_cls=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., language_model_cls=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 1621-1622: Method `Qwen3_5MoeForConditionalGeneration.get_hidden_dim`
```python
    def get_hidden_dim(self, module_name: str, layer_idx: int):
        return self.model.get_hidden_dim(module_name, layer_idx)
```
**EN:** This method implements `get_hidden_dim(module_name: ..., layer_idx: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_hidden_dim(module_name: ..., layer_idx: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1624-1626: Method `Qwen3_5MoeForConditionalGeneration.should_apply_lora`
```python
    def should_apply_lora(self, module_name: str) -> bool:
        # Accept all language model layer modules (attention, linear_attn, mlp).
        return module_name.startswith("model.layers.")
```
**EN:** This method implements `should_apply_lora(module_name: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3_5MoeForConditionalGeneration`.
**CN:** 这个方法实现了 `should_apply_lora(module_name: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3_5MoeForConditionalGeneration` 内部调用。

### Lines 1628-1635: Method `Qwen3_5MoeForConditionalGeneration._get_num_fused_shared_experts`
```python
    def _get_num_fused_shared_experts(self):
        if not (
            hasattr(self.model, "layers")
            and len(self.model.layers) > 0
            and hasattr(self.model.layers[0].mlp, "num_fused_shared_experts")
        ):
            return 0
        return self.model.layers[0].mlp.num_fused_shared_experts
```
**EN:** This method implements `_get_num_fused_shared_experts()` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `_get_num_fused_shared_experts()`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 1637-1640: Method `Qwen3_5MoeForConditionalGeneration.get_embed_and_head`
```python
    def get_embed_and_head(self):
        embed = self.model.embed_tokens.weight if self.pp_group.is_first_rank else None
        head = self.lm_head.weight if self.pp_group.is_last_rank else None
        return embed, head
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1642-1650: Method `Qwen3_5MoeForConditionalGeneration.set_embed_and_head`
```python
    def set_embed_and_head(self, embed, head):
        if self.pp_group.is_first_rank and embed is not None:
            del self.model.embed_tokens.weight
            self.model.embed_tokens.weight = embed
        if self.pp_group.is_last_rank and head is not None:
            del self.lm_head.weight
            self.lm_head.weight = head
        torch.cuda.empty_cache()
        torch.cuda.synchronize()
```
**EN:** This method implements `set_embed_and_head(embed, head)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_embed_and_head(embed, head)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 1652-1963: Method `Qwen3_5MoeForConditionalGeneration.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
            # GDN fused projections
            ("in_proj_qkvz.", "in_proj_qkv.", (0, 1, 2)),
            ("in_proj_qkvz.", "in_proj_z.", 3),
            ("in_proj_ba.", "in_proj_b.", 0),
            ("in_proj_ba.", "in_proj_a.", 1),
        ]

        num_experts = self.config.num_experts

        # Params for weights, fp8 weight scales, fp8 activation scales
        # (param_name, weight_name, expert_id, shard_id)
        expert_params_mapping = FusedMoE.make_expert_params_mapping(
            ckpt_gate_proj_name="gate_proj",
            ckpt_down_proj_name="down_proj",
            ckpt_up_proj_name="up_proj",
            num_experts=(
                num_experts
                if not self.enable_shared_expert_fusion
                else num_experts + self.num_fused_shared_experts
            ),
        )

        # Skip loading extra parameters for GPTQ/modelopt models.
        ignore_suffixes = (
            ".bias",
            "_bias",
            ".k_scale",
            "_k_scale",
            ".v_scale",
            "_v_scale",
            "_weight_scale",
            "_input_scale",
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 1965-1967: Method `Qwen3_5MoeForConditionalGeneration.routed_experts_weights_of_layer`
```python
    @property
    def routed_experts_weights_of_layer(self):
        return self._routed_experts_weights_of_layer.value
```
**EN:** This method implements `routed_experts_weights_of_layer()` and handles weight mapping, filtering, or loading for this model component Decorators: property.
**CN:** 这个方法实现了 `routed_experts_weights_of_layer()`，其作用是处理该模型组件的权重映射、筛选或加载逻辑 装饰器：property。

### Lines 1969-1976: Method `Qwen3_5MoeForConditionalGeneration.get_model_config_for_expert_location`
```python
    @classmethod
    def get_model_config_for_expert_location(cls, config):
        text_config = getattr(config, "text_config", config)
        return ModelConfigForExpertLocation(
            num_layers=text_config.num_hidden_layers,
            num_logical_experts=text_config.num_experts,
            num_groups=None,
        )
```
**EN:** This method implements `get_model_config_for_expert_location(config)` and returns a cached or derived object used elsewhere in the pipeline Decorators: classmethod.
**CN:** 这个方法实现了 `get_model_config_for_expert_location(config)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果 装饰器：classmethod。

### Lines 1979-1979: Top-level assign
```python
EntryClass = [Qwen3_5MoeForConditionalGeneration, Qwen3_5ForConditionalGeneration]
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
- `functools.lru_cache`
- `typing.Iterable`
- `typing.Optional`
- `typing.Set`
- `typing.Tuple`
- `typing.Union`
- `torch`
- `torch.nn`
- `triton`
- `sglang.jit_kernel.triton.gdn_fused_proj.fused_qkvzba_split_reshape_cat_contiguous`
- `sglang.srt.configs.qwen3_5.Qwen3_5Config`
- `sglang.srt.configs.qwen3_5.Qwen3_5MoeConfig`
- `sglang.srt.configs.qwen3_5.Qwen3_5TextConfig`
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
- `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`
- `sglang.srt.layers.parameter.BlockQuantScaleParameter`
- `sglang.srt.layers.parameter.PerTensorScaleParameter`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.radix_linear_attention.RadixLinearAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.utils.get_layer_id`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.cuda_graph_runner.get_is_capture_mode`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.model_loader.weight_utils.sharded_weight_loader`
- `sglang.srt.models.qwen2_moe.Qwen2MoeMLP`
- `sglang.srt.models.qwen2_moe.Qwen2MoeSparseMoeBlock`
- `sglang.srt.models.qwen3_vl.Qwen3VLForConditionalGeneration`
- `sglang.srt.models.utils.fused_qk_gemma_rmsnorm`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.LazyValue`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.cpu_has_amx_support`
- `sglang.srt.utils.get_bool_env_var`
- `sglang.srt.utils.is_cpu`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_gfx95_supported`
- `sglang.srt.utils.is_hip`
- `sglang.srt.utils.is_npu`
- `sglang.srt.utils.make_layers`
- `sglang.srt.utils.set_weight_attrs`
- `sglang.srt.utils.hf_transformers_utils.get_processor`
- `sglang.srt.utils.hf_transformers_utils.get_rope_config`
