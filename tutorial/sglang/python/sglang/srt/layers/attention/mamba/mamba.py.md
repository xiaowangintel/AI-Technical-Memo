# mamba.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/mamba/mamba.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the main mamba classes used by the SGLang attention stack, together with helper routines that support runtime execution. / 该模块定义 SGLang 注意力栈使用的主要 mamba 类，并提供支撑运行时执行的辅助例程。
## Line-by-Line Analysis / 逐行分析
### Lines 1-38: imports
```python
import logging
from typing import Callable, List, Optional, Tuple

import torch
import torch.nn as nn

from sglang.srt.configs.mamba_utils import (
    Mamba2CacheParams,
    extra_groups_for_head_shards,
)
from sglang.srt.distributed import (
    divide,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from sglang.srt.layers.attention.mamba.mamba2_metadata import Mamba2Metadata
from sglang.srt.layers.attention.mamba.mixer2_rms_norm_gated import Mixer2RMSNormGated
from sglang.srt.layers.attention.mamba.ops import (
    mamba_chunk_scan_combined,
    selective_state_update,
)
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
    RowParallelLinear,
)
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.mem_cache.memory_pool import MambaPool
from sglang.srt.model_loader.weight_utils import (
    composed_weight_loader,
    sharded_weight_loader,
)
from sglang.srt.utils import (
    is_cpu,
    is_cuda,
    is_npu,
    set_weight_attrs,
)
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 40-57: conditional branch
```python
if is_cuda():
    from sglang.srt.layers.attention.mamba.causal_conv1d import (
        causal_conv1d_fn,
        causal_conv1d_update,
    )
    from sglang.srt.layers.attention.mamba.causal_conv1d_triton import (
        causal_conv1d_fn as causal_conv1d_fn_triton,
    )
    from sglang.srt.layers.attention.mamba.causal_conv1d_triton import (
        causal_conv1d_update as causal_conv1d_update_triton,
    )
elif is_npu():
    from sgl_kernel_npu.mamba.causal_conv1d import (
        causal_conv1d_fn_npu as causal_conv1d_fn,
    )
    from sgl_kernel_npu.mamba.causal_conv1d import (
        causal_conv1d_update_npu as causal_conv1d_update,
    )
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 59-61: module constants
```python
LoaderFunction = Callable[[torch.Tensor, torch.Tensor], None]

logger = logging.getLogger(__name__)
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 64-168: function mamba_v2_sharded_weight_loader
```python
def mamba_v2_sharded_weight_loader(
    shard_spec: List[Tuple[int, int, float]],
    tp_size: int,
    tp_rank: int,
) -> LoaderFunction:
    """Create a weight loader for mamba v2. This ensures that the projections
    are correctly sharded so that they can be split into x, B, C. It also
    ensures that all the groups corresponding to a head shard is placed
    together with it.
    """

    def loader(param: torch.Tensor, loaded_weight: torch.Tensor) -> None:

        # - track boundary of (sharded) param, and loaded_weight, respectively
        boundary, loaded_boundary = 0, 0

        # Calculate padding size for CPU when TP odd size
        if is_cpu():
            full_dim_sum = 0
            full_dim_list = []
            weight_full_dim_list = []
            for full_dim, _, _ in shard_spec:
                full_dim_sum = full_dim_sum + full_dim
                full_dim_list.append(full_dim)
            for full_dim in full_dim_list:
                weight_full_dim_list.append(
                    int(full_dim / full_dim_sum * loaded_weight.size(0))
                )
            assert sum(weight_full_dim_list) == loaded_weight.size(
                0
            ), f"Padding the loaded weight failed due to sizes are not divisible cleanly from {weight_full_dim_list} to {loaded_weight.size(0)}"
            if loaded_weight.size(0) < full_dim_sum and tp_rank == 0:
# ... omitted 61 lines ...
            # https://github.com/python/mypy/issues/2410
            param.data[
                boundary : (boundary + take), ...  # type: ignore[misc]
            ] = loaded_weight[
                loaded_start_idx : (loaded_start_idx + take)  # type: ignore[misc]
            ]  # type: ignore[misc]

            # move indexing boundaries
            boundary += shard_size
            loaded_boundary += full_dim - extra

    return loader
```
**EN:** Implements the mamba v2 sharded weight loader routine used by this attention module.
**CN:** 实现该注意力模块使用的 mamba v2 sharded weight loader 例程。

### Lines 171-181: class MambaMixer2
```python
class MambaMixer2(torch.nn.Module):
    """
    Compute ∆, A, B, C, and D the state space parameters and compute
    the `contextualized_states`. A, D are input independent
    (see Mamba paper [1] Section 3.5.2 "Interpretation of A"
    for why A isn't selective) ∆, B, C are input-dependent
    (this is a key difference between Mamba and the linear time
    invariant S4, and is why Mamba is called
    **selective** state spaces)
    """
```
**EN:** Defines the mamba mixer2 type and the state it exposes to the rest of the attention stack.
**CN:** 定义 mamba mixer2 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 182-404: method MambaMixer2.__init__
```python
    def __init__(
        self,
        cache_params: Mamba2CacheParams,
        hidden_size: int,
        use_conv_bias: bool,
        use_bias: bool,
        n_groups: int = 1,
        rms_norm_eps: float = 1e-5,
        activation: str = "silu",
        use_rms_norm: bool = True,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        # For TP, the sharding plan is as follows:
        # - for the conv modules, since
        #   conv_dim = intermediate_size * 2 * n_groups * ssm_state_size,
        #   we shard intermediate_size and n_groups
        # - since intermediate_size = n_heads * head_dim, sharding on
        #   intermediate_size is achieved by sharding on n_heads.
        # - IF, world_size divides groups, then sharding
        #   (n_groups / world_size, n_heads / world_size)
        #   also maintains the invariant n_heads % n_groups == 0
        # - HOWEVER IF, world_size DOES NOT divide groups, then we need
        #   to allocate extra space in the shard, such that groups
        #   may be replicated to follow the head shard.
        # - NOTE: currently for the world size DOES NOT divide groups
        #   case, we only support the case when n_groups == 1
        self.tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()

# ... omitted 179 lines ...
            hidden_size,
            bias=use_bias,
            input_is_parallel=True,
            quant_config=quant_config,
            prefix=f"{prefix}.out_proj",
        )

        self.norm = Mixer2RMSNormGated(
            intermediate_size, n_groups, self.use_rms_norm, eps=rms_norm_eps
        )

        self.prefix = prefix
```
**EN:** Initializes the MambaMixer2 instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 MambaMixer2 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 406-709: method MambaMixer2.forward
```python
    def forward(
        self,
        *,
        hidden_states: torch.Tensor,
        output: torch.Tensor,
        layer_cache: MambaPool.State,
        metadata: Mamba2Metadata,
        mup_vector: Optional[torch.Tensor] = None,
        use_triton_causal_conv: bool = False,
    ):
        # metadata contains metadata necessary for the mamba2 triton
        # kernels to operate in continuous batching and in chunked prefill
        # modes; they are computed at top-level model forward since they
        # stay the same and reused for all mamba layers in the same iteration
        state_indices_tensor = metadata.mamba_cache_indices
        conv_state = layer_cache.conv[0]
        ssm_state = layer_cache.temporal

        query_start_loc = metadata.query_start_loc

        # 1. Gated MLP's linear projection
        projected_states, _ = self.in_proj(hidden_states)

        if mup_vector is not None:
            projected_states = projected_states * mup_vector

        gate, hidden_states_B_C, dt = torch.split(
            projected_states,
            [
                self.intermediate_size // self.tp_size,
                self.conv_dim // self.tp_size,
                self.num_heads // self.tp_size,
# ... omitted 260 lines ...
                    state_batch_indices=state_indices_tensor_d,
                    out=preallocated_ssm_out_d.view(num_decodes, -1, self.head_dim),
                )

        # 4. gated MLP
        # GatedRMSNorm internally applying SiLU to the gate
        # SiLU is applied internally before normalization, unlike standard
        # norm usage
        hidden_states = self.norm(preallocated_ssm_out, gate[:num_actual_tokens])

        # 5. Final linear projection
        output[:num_actual_tokens], _ = self.out_proj(hidden_states)
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 711-713: method MambaMixer2.mamba_type
```python
    @property
    def mamba_type(self) -> str:
        return "mamba2"
```
**EN:** Implements the mamba type routine used by this attention module.
**CN:** 实现该注意力模块使用的 mamba type 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理
- **EN:** Speculative decoding support / **CN:** 投机解码支持

## Dependencies / 依赖关系
- `logging`
- `typing.Callable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `sglang.srt.configs.mamba_utils.Mamba2CacheParams`
- `sglang.srt.configs.mamba_utils.extra_groups_for_head_shards`
- `sglang.srt.distributed.divide`
- `sglang.srt.distributed.get_tensor_model_parallel_rank`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.layers.attention.mamba.mamba2_metadata.Mamba2Metadata`
- `sglang.srt.layers.attention.mamba.mixer2_rms_norm_gated.Mixer2RMSNormGated`
- `sglang.srt.layers.attention.mamba.ops.mamba_chunk_scan_combined`
- `sglang.srt.layers.attention.mamba.ops.selective_state_update`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.MergedColumnParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.mem_cache.memory_pool.MambaPool`
- `sglang.srt.model_loader.weight_utils.composed_weight_loader`
- `sglang.srt.model_loader.weight_utils.sharded_weight_loader`
- `sglang.srt.utils.is_cpu`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_npu`
- `sglang.srt.utils.set_weight_attrs`
- `sglang.srt.layers.attention.mamba.causal_conv1d.causal_conv1d_fn`
- `sglang.srt.layers.attention.mamba.causal_conv1d.causal_conv1d_update`
- `sglang.srt.layers.attention.mamba.causal_conv1d_triton.causal_conv1d_fn`
- `sglang.srt.layers.attention.mamba.causal_conv1d_triton.causal_conv1d_update`
- `sgl_kernel_npu.mamba.causal_conv1d.causal_conv1d_fn_npu`
- `sgl_kernel_npu.mamba.causal_conv1d.causal_conv1d_update_npu`
- `copy`
