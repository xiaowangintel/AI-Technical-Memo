# mamba_mixer2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/mamba/mamba_mixer2.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `Mixer2RMSNormGated`, `MambaMixer2` for Mamba/state-space layers and kernels. / 实现 `Mixer2RMSNormGated`, `MambaMixer2`，用于Mamba/状态空间模型层与内核。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 5-56)
```python
import torch
from torch import nn

from vllm.config import CacheConfig, ModelConfig, get_current_vllm_config
from vllm.distributed import (
    divide,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_gather,
    tensor_model_parallel_all_reduce,
)
from vllm.forward_context import ForwardContext, get_forward_context
from vllm.logger import init_logger
from vllm.model_executor.custom_op import CustomOp, PluggableLayer
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.mamba.abstract import MambaBase
from vllm.model_executor.layers.mamba.mamba_utils import (
    MambaStateDtypeCalculator,
    MambaStateShapeCalculator,
    is_conv_state_dim_first,
)
from vllm.model_executor.layers.mamba.ops.causal_conv1d import (
    causal_conv1d_fn,
    causal_conv1d_update,
)
from vllm.model_executor.layers.mamba.ops.layernorm_gated import rms_norm_gated
from vllm.model_executor.layers.mamba.ops.ssd_combined import (
    mamba_chunk_scan_combined_varlen,
)
from vllm.model_executor.layers.mamba.ops.ssu_dispatch import selective_state_update
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.model_loader.weight_utils import (
    LoaderFunction,
    composed_weight_loader,
    sharded_weight_loader,
)
from vllm.model_executor.parameter import BasevLLMParameter
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform
from vllm.utils.torch_utils import (
    LayerNameType,
    _encode_layer_name,
    _resolve_layer_name,
    direct_register_custom_op,
)
from vllm.v1.attention.backend import AttentionMetadata
from vllm.v1.attention.backends.mamba2_attn import Mamba2AttentionMetadata
from vllm.v1.attention.backends.registry import MambaAttentionBackendEnum
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm.config`, `vllm.distributed`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.linear`. That import mix shows the file is part of the Mamba/state-space layers and kernels stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm.config`, `vllm.distributed`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.linear`）。这些导入关系表明该文件属于Mamba/状态空间模型层与内核栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 58-58)
```python
logger = init_logger(__name__)
```
**EN:** This block defines module-level metadata or constants such as `logger`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the Mamba/state-space layers and kernels pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `logger`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在Mamba/状态空间模型层与内核流程中复用。

### Function `mamba_v2_sharded_weight_loader` (lines 171-228)
```python
def mamba_v2_sharded_weight_loader(
    shard_spec: list[tuple[int, int, float]],
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

        # - iterate over the shard specs
        for full_dim, extra, duplicate_groups in shard_spec:
            # - full dim is the model dim (before TP).
            # - extra > 0, means there is expected overall increase
            #   of dimensions. This is so because of replication.
            # - ratio is used map the tp_rank to the actual shard
            #   rank. This is useful when there is replication of
            #   groups to accompany head shards.

            # - size of the loaded shard
            shard_size = full_dim // tp_size

            # - compute the rank into the loaded shard.
            # - if there is replication, different TP shards will
            #   take from the same rank.
            # NOTE: currently we only support duplication
            # in the case where num_groups == 1
            rank = 0 if duplicate_groups else tp_rank

            # - leftmost boundary index into loaded weight.
            loaded_skip = rank * shard_size
            loaded_start_idx = loaded_boundary + loaded_skip

            # - take these many dims from the loaded weight.
            take = min(shard_size, full_dim - extra - loaded_skip)

            # - always shard on dim 0
            # - the ignore is for a mundane mypy error as it does not
            #   seem to handle slices well.
            # https://github.com/python/mypy/issues/2410
            param.data[
                boundary : (boundary + take), ...  # type: ignore[misc]
            ] = loaded_weight[
                loaded_start_idx : (
                    loaded_start_idx + take
                )  # type: ignore[misc]
            ]  # type: ignore[misc]

            # move indexing boundaries
            boundary += shard_size
            loaded_boundary += full_dim - extra

    return loader
```
**EN:** Defines function `mamba_v2_sharded_weight_loader` with signature `mamba_v2_sharded_weight_loader(shard_spec: list[tuple[int, int, float]], tp_size: int, tp_rank: int) -> LoaderFunction`. It mainly works with `shard_spec`, `tp_size`, `tp_rank`; implements one step of the Mamba/SSM execution path. The body uses branching, iteration. Key calls include `min`.
**CN:** 定义函数 `mamba_v2_sharded_weight_loader`，其签名为 `mamba_v2_sharded_weight_loader(shard_spec: list[tuple[int, int, float]], tp_size: int, tp_rank: int) -> LoaderFunction`。它主要围绕 `shard_spec`, `tp_size`, `tp_rank` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、循环处理。关键调用包括 `min`。

### Function `mamba_mixer2` (lines 1074-1082)
```python
def mamba_mixer2(
    projected_states: torch.Tensor,
    output: torch.Tensor,
    layer_name: LayerNameType,
) -> None:
    layer_name = _resolve_layer_name(layer_name)
    forward_context: ForwardContext = get_forward_context()
    self = forward_context.no_compile_layers[layer_name]
    self.conv_ssm_forward(projected_states=projected_states, output=output)
```
**EN:** Defines function `mamba_mixer2` with signature `mamba_mixer2(projected_states: torch.Tensor, output: torch.Tensor, layer_name: LayerNameType) -> None`. It mainly works with `projected_states`, `output`, `layer_name`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include `_resolve_layer_name`, `get_forward_context`, `self.conv_ssm_forward`.
**CN:** 定义函数 `mamba_mixer2`，其签名为 `mamba_mixer2(projected_states: torch.Tensor, output: torch.Tensor, layer_name: LayerNameType) -> None`。它主要围绕 `projected_states`, `output`, `layer_name` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `_resolve_layer_name`, `get_forward_context`, `self.conv_ssm_forward`。

### Function `mamba_mixer2_fake` (lines 1085-1090)
```python
def mamba_mixer2_fake(
    projected_states: torch.Tensor,
    output: torch.Tensor,
    layer_name: LayerNameType,
) -> None:
    return
```
**EN:** Defines function `mamba_mixer2_fake` with signature `mamba_mixer2_fake(projected_states: torch.Tensor, output: torch.Tensor, layer_name: LayerNameType) -> None`. It mainly works with `projected_states`, `output`, `layer_name`; implements one step of the Mamba/SSM execution path. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `mamba_mixer2_fake`，其签名为 `mamba_mixer2_fake(projected_states: torch.Tensor, output: torch.Tensor, layer_name: LayerNameType) -> None`。它主要围绕 `projected_states`, `output`, `layer_name` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Class `Mixer2RMSNormGated` overview (lines 66-168)
```python
class Mixer2RMSNormGated(CustomOp):
    # --8<-- [end:mixer2_gated_rms_norm]

    def __init__(
        self,
        full_hidden_size: int,
        full_n_groups: int,
        use_rms_norm: bool = True,
        eps: float = 1e-6,
    ):
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.full_hidden_size = full_hidden_size
        self.group_size = full_hidden_size // full_n_groups
        self.per_rank_hidden_size = full_hidden_size // self.tp_size
        self.n_groups = full_hidden_size // self.group_size

        self.variance_epsilon = eps
        self.use_rms_norm = use_rms_norm
        if self.use_rms_norm:
            # Register norm weight only if we're actually applying RMSNorm
            self.weight = nn.Parameter(torch.ones(self.per_rank_hidden_size))
            set_weight_attrs(self.weight, {"weight_loader": sharded_weight_loader(0)})
        else:
```
**EN:** Defines class `Mixer2RMSNormGated` with base classes `CustomOp` and decorators `CustomOp.register('mixer2_gated_rms_norm')`. It acts as a sequence mixer and exposes 3 direct methods, with notable entries `__init__`, `forward_native`, `forward_cuda`.
**CN:** 定义类 `Mixer2RMSNormGated`，其基类为 `CustomOp`，装饰器为 `CustomOp.register('mixer2_gated_rms_norm')`。它在整体实现中充当序列混合器，并直接暴露 3 个方法，较重要的包括 `__init__`, `forward_native`, `forward_cuda`。

### Method `Mixer2RMSNormGated.__init__` (lines 69-95)
```python
    def __init__(
        self,
        full_hidden_size: int,
        full_n_groups: int,
        use_rms_norm: bool = True,
        eps: float = 1e-6,
    ):
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()
        self.full_hidden_size = full_hidden_size
        self.group_size = full_hidden_size // full_n_groups
        self.per_rank_hidden_size = full_hidden_size // self.tp_size
        self.n_groups = full_hidden_size // self.group_size

        self.variance_epsilon = eps
        self.use_rms_norm = use_rms_norm
        if self.use_rms_norm:
            # Register norm weight only if we're actually applying RMSNorm
            self.weight = nn.Parameter(torch.ones(self.per_rank_hidden_size))
            set_weight_attrs(self.weight, {"weight_loader": sharded_weight_loader(0)})
        else:
            # Avoid checkpoint mismatch by skipping unused parameter
            self.register_parameter("weight", None)
        assert self.full_hidden_size % self.tp_size == 0, (
            "Tensor parallel world size must divide hidden size."
        )
```
**EN:** Defines function `Mixer2RMSNormGated.__init__` with signature `__init__(self, full_hidden_size: int, full_n_groups: int, use_rms_norm: bool=True, eps: float=1e-06)`. It mainly works with `full_hidden_size`, `full_n_groups`, `use_rms_norm`, `eps`; initializes the object state and cached resources. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `super.__init__`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `nn.Parameter`, `set_weight_attrs`, `self.register_parameter`.
**CN:** 定义函数 `Mixer2RMSNormGated.__init__`，其签名为 `__init__(self, full_hidden_size: int, full_n_groups: int, use_rms_norm: bool=True, eps: float=1e-06)`。它主要围绕 `full_hidden_size`, `full_n_groups`, `use_rms_norm`, `eps` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `super.__init__`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `nn.Parameter`, `set_weight_attrs`, `self.register_parameter`。

### Method `Mixer2RMSNormGated.forward_native` (lines 97-146)
```python
    def forward_native(
        self,
        x: torch.Tensor,
        gate: torch.Tensor,
    ):
        # Three tensor-parallel cases:
        #   1. n_groups is 1
        #      In this case we parallelize along the reduction dim.
        #      Each rank computes a local sum of squares followed by AllReduce
        #   2. tp_size divides n_groups
        #      Each rank only reduces within its local group(s).
        #      No collective ops necessary.
        #   3. The general case can be pretty complicated so we AllGather
        #      the input and then redundantly compute the RMSNorm.
        input_dtype = x.dtype
        x = x * nn.functional.silu(gate.to(torch.float32))
        if not self.use_rms_norm:
            return x.to(input_dtype)

        if self.n_groups == 1:
            if self.tp_size > 1:
                # Compute local sum and then reduce to obtain global sum
                local_sums = x.pow(2).sum(dim=-1, keepdim=True)
                global_sums = tensor_model_parallel_all_reduce(local_sums)
                # Calculate the variance
                count = self.tp_size * x.shape[-1]
                variance = global_sums / count

            else:
                variance = x.pow(2).mean(-1, keepdim=True)
            x = x * torch.rsqrt(variance + self.variance_epsilon)
        else:
            redundant_tp: bool = self.n_groups % self.tp_size != 0
            if redundant_tp:
                # To handle the general case, redundantly apply the variance
                x = tensor_model_parallel_all_gather(x, -1)

            *prefix_dims, hidden_dim = x.shape
            group_count = hidden_dim // self.group_size
            x_grouped = x.view(*prefix_dims, group_count, self.group_size)
            variance = x_grouped.pow(2).mean(-1, keepdim=True)
            x_grouped = x_grouped * torch.rsqrt(variance + self.variance_epsilon)
            x = x_grouped.view(*prefix_dims, hidden_dim)

            if redundant_tp:
                start = self.per_rank_hidden_size * self.tp_rank
                end = start + self.per_rank_hidden_size
                x = x[..., start:end]

        return self.weight * x.to(input_dtype)
```
**EN:** Defines function `Mixer2RMSNormGated.forward_native` with signature `forward_native(self, x: torch.Tensor, gate: torch.Tensor)`. It mainly works with `x`, `gate`; implements one step of the Mamba/SSM execution path. The body uses branching, tensor/kernel operations. Key calls include `nn.functional.silu`, `x.to`, `x.view`, `x_grouped.pow.mean`, `x_grouped.view`, `gate.to`.
**CN:** 定义函数 `Mixer2RMSNormGated.forward_native`，其签名为 `forward_native(self, x: torch.Tensor, gate: torch.Tensor)`。它主要围绕 `x`, `gate` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `nn.functional.silu`, `x.to`, `x.view`, `x_grouped.pow.mean`, `x_grouped.view`, `gate.to`。

### Method `Mixer2RMSNormGated.forward_cuda` (lines 148-168)
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        gate: torch.Tensor,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
        input_dtype = x.dtype
        if not self.use_rms_norm:
            # Keep gate in float32 for numerical stability during silu
            return x * nn.functional.silu(gate.to(torch.float32)).to(input_dtype)

        if ((self.n_groups % self.tp_size) != 0) or self.n_groups != 1:
            return self.forward_native(x, gate)

        return rms_norm_gated(
            x,
            self.weight.data,
            bias=None,
            z=gate,
            eps=self.variance_epsilon,
            norm_before_gate=False,
        )
```
**EN:** Defines function `Mixer2RMSNormGated.forward_cuda` with signature `forward_cuda(self, x: torch.Tensor, gate: torch.Tensor) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]`. It mainly works with `x`, `gate`; implements one step of the Mamba/SSM execution path. The body uses branching, tensor/kernel operations. Key calls include `rms_norm_gated`, `self.forward_native`, `nn.functional.silu.to`, `nn.functional.silu`, `gate.to`.
**CN:** 定义函数 `Mixer2RMSNormGated.forward_cuda`，其签名为 `forward_cuda(self, x: torch.Tensor, gate: torch.Tensor) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `x`, `gate` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `rms_norm_gated`, `self.forward_native`, `nn.functional.silu.to`, `nn.functional.silu`, `gate.to`。

### Class `MambaMixer2` overview (lines 234-1071)
```python
class MambaMixer2(MambaBase, PluggableLayer):
    """
    Compute ∆, A, B, C, and D the state space parameters and compute
    the `contextualized_states`. A, D are input independent
    (see Mamba paper [1] Section 3.5.2 "Interpretation of A"
    for why A isn't selective) ∆, B, C are input-dependent
    (this is a key difference between Mamba and the linear time
    invariant S4, and is why Mamba is called
    **selective** state spaces)
    """

    # --8<-- [end:mamba_mixer2]

    def __init__(
        self,
        hidden_size: int,
        ssm_state_size: int,
        conv_kernel_size: int,
        intermediate_size: int,
        use_conv_bias: bool,
        use_bias: bool,
        n_groups: int = 1,
        num_heads: int = 128,
        head_dim: int = 64,
        rms_norm_eps: float = 1e-5,
```
**EN:** Defines class `MambaMixer2` with base classes `MambaBase`, `PluggableLayer` and decorators `PluggableLayer.register('mamba_mixer2')`. It acts as a sequence mixer and exposes 7 direct methods, with notable entries `__init__`, `forward`, `_warmup_ssd_kernels`, `conv_ssm_forward`, `get_state_dtype`, `get_state_shape`. Its docstring says: Compute ∆, A, B, C, and D the state space parameters and compute the `contextualized_states`.
**CN:** 定义类 `MambaMixer2`，其基类为 `MambaBase`, `PluggableLayer`，装饰器为 `PluggableLayer.register('mamba_mixer2')`。它在整体实现中充当序列混合器，并直接暴露 7 个方法，较重要的包括 `__init__`, `forward`, `_warmup_ssd_kernels`, `conv_ssm_forward`, `get_state_dtype`, `get_state_shape`。 文档字符串进一步说明了该类的定位。

### Method `MambaMixer2.__init__` (lines 247-528)
```python
    def __init__(
        self,
        hidden_size: int,
        ssm_state_size: int,
        conv_kernel_size: int,
        intermediate_size: int,
        use_conv_bias: bool,
        use_bias: bool,
        n_groups: int = 1,
        num_heads: int = 128,
        head_dim: int = 64,
        rms_norm_eps: float = 1e-5,
        activation: str = "silu",
        use_rms_norm: bool = True,
        model_config: ModelConfig | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
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
        tp_rank = get_tensor_model_parallel_rank()

# ... truncated for analysis ...
        self.tped_dt_size = self.num_heads // self.tp_size

        self.split_hidden_states_B_C_fn = lambda hidden_states_B_C: torch.split(
            hidden_states_B_C,
            [
                self.tped_intermediate_size,
                self.groups_ssm_state_size // self.tp_size,
                self.groups_ssm_state_size // self.tp_size,
            ],
            dim=-1,
        )

        # Check if running on Blackwell (SM100+) for kernel tuning
        self.is_blackwell = current_platform.is_device_capability_family(100)
```
**EN:** Defines function `MambaMixer2.__init__` with signature `__init__(self, hidden_size: int, ssm_state_size: int, conv_kernel_size: int, intermediate_size: int, use_conv_bias: bool, use_bias: bool, n_groups: int=1, num_heads: int=128, head_dim: int=64, rms_norm_eps: float=1e-05, activation: str='silu', use_rms_norm: bool=True, model_config: ModelConfig | None=None, cache_config: CacheConfig | None=None, quant_config: QuantizationConfig | None=None, prefix: str='')`. It mainly works with `hidden_size`, `ssm_state_size`, `conv_kernel_size`, `intermediate_size`, `use_conv_bias`, `use_bias`, `n_groups`, `num_heads`; initializes the object state and cached resources. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `super.__init__`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `self.conv1d.weight.data.unsqueeze`, `self.conv1d.weight.view`, `self.register_buffer`.
**CN:** 定义函数 `MambaMixer2.__init__`，其签名为 `__init__(self, hidden_size: int, ssm_state_size: int, conv_kernel_size: int, intermediate_size: int, use_conv_bias: bool, use_bias: bool, n_groups: int=1, num_heads: int=128, head_dim: int=64, rms_norm_eps: float=1e-05, activation: str='silu', use_rms_norm: bool=True, model_config: ModelConfig | None=None, cache_config: CacheConfig | None=None, quant_config: QuantizationConfig | None=None, prefix: str='')`。它主要围绕 `hidden_size`, `ssm_state_size`, `conv_kernel_size`, `intermediate_size`, `use_conv_bias`, `use_bias`, `n_groups`, `num_heads` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `super.__init__`, `get_tensor_model_parallel_world_size`, `get_tensor_model_parallel_rank`, `self.conv1d.weight.data.unsqueeze`, `self.conv1d.weight.view`, `self.register_buffer`。

### Method `MambaMixer2.forward` (lines 530-569)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
        mup_vector: torch.Tensor | None = None,
    ):
        # 1. Gated MLP's linear projection
        projected_states, _ = self.in_proj(hidden_states)
        if mup_vector is not None:
            projected_states = projected_states * mup_vector

        # 2. Prepare inputs for conv + SSM
        ssm_output = torch.empty(
            [
                hidden_states.shape[0],
                (self.num_heads // self.tp_size) * self.head_dim,
            ],
            dtype=hidden_states.dtype,
            device=hidden_states.device,
        )

        # 3. conv + SSM
        # (split `projected_states` into hidden_states_B_C, dt in the custom op to
        # ensure it is not treated as an intermediate tensor by torch compile)
        torch.ops.vllm.mamba_mixer2(
            projected_states,
            ssm_output,
            _encode_layer_name(self.prefix),
        )

        # 4. gated MLP
        # GatedRMSNorm internally applying SiLU to the gate
        # SiLU is applied internally before normalization, unlike standard
        # norm usage
        gate = projected_states[..., : self.tped_intermediate_size]
        hidden_states = self.norm(ssm_output, gate)

        # 5. Final linear projection
        output, _ = self.out_proj(hidden_states)

        return output
```
**EN:** Defines function `MambaMixer2.forward` with signature `forward(self, hidden_states: torch.Tensor, mup_vector: torch.Tensor | None=None)`. It mainly works with `hidden_states`, `mup_vector`; runs the main forward-path computation. The body uses branching, tensor/kernel operations. Key calls include `self.in_proj`, `torch.empty`, `torch.ops.vllm.mamba_mixer2`, `self.norm`, `self.out_proj`, `_encode_layer_name`.
**CN:** 定义函数 `MambaMixer2.forward`，其签名为 `forward(self, hidden_states: torch.Tensor, mup_vector: torch.Tensor | None=None)`。它主要围绕 `hidden_states`, `mup_vector` 展开；执行主要的前向计算路径。函数体包含分支判断、张量或内核操作。关键调用包括 `self.in_proj`, `torch.empty`, `torch.ops.vllm.mamba_mixer2`, `self.norm`, `self.out_proj`, `_encode_layer_name`。

### Method `MambaMixer2.get_state_dtype` (lines 1048-1055)
```python
    def get_state_dtype(self) -> tuple[torch.dtype, torch.dtype]:
        assert self.model_config is not None
        assert self.cache_config is not None
        return MambaStateDtypeCalculator.mamba2_state_dtype(
            self.model_config.dtype,
            self.cache_config.mamba_cache_dtype,
            self.cache_config.mamba_ssm_cache_dtype,
        )
```
**EN:** Defines function `MambaMixer2.get_state_dtype` with signature `get_state_dtype(self) -> tuple[torch.dtype, torch.dtype]`. It mainly works with object context only; returns a derived property or capability check. The body uses validation/error handling. Key calls include `MambaStateDtypeCalculator.mamba2_state_dtype`.
**CN:** 定义函数 `MambaMixer2.get_state_dtype`，其签名为 `get_state_dtype(self) -> tuple[torch.dtype, torch.dtype]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含校验或报错逻辑。关键调用包括 `MambaStateDtypeCalculator.mamba2_state_dtype`。

### Method `MambaMixer2.get_state_shape` (lines 1057-1067)
```python
    def get_state_shape(self) -> tuple[tuple[int, ...], tuple[int, ...]]:
        return MambaStateShapeCalculator.mamba2_state_shape(
            intermediate_size=self.intermediate_size,
            tp_world_size=get_tensor_model_parallel_world_size(),
            n_groups=self.n_groups,
            num_heads=self.num_heads,
            head_dim=self.head_dim,
            state_size=self.ssm_state_size,
            conv_kernel=self.conv_kernel_size,
            num_spec=self.num_spec,
        )
```
**EN:** Defines function `MambaMixer2.get_state_shape` with signature `get_state_shape(self) -> tuple[tuple[int, ...], tuple[int, ...]]`. It mainly works with object context only; returns a derived property or capability check. The body uses mostly straightforward data movement and object wiring. Key calls include `MambaStateShapeCalculator.mamba2_state_shape`, `get_tensor_model_parallel_world_size`.
**CN:** 定义函数 `MambaMixer2.get_state_shape`，其签名为 `get_state_shape(self) -> tuple[tuple[int, ...], tuple[int, ...]]`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `MambaStateShapeCalculator.mamba2_state_shape`, `get_tensor_model_parallel_world_size`。

## Key Concepts / 关键概念
- **EN:** The file belongs to the Mamba/SSM path, so recurrent state updates, scan-style kernels, or short convolutions are recurring themes.
  **CN:** 该文件位于 Mamba/状态空间模型路径中，因此循环状态更新、scan 风格内核或短卷积是反复出现的主题。
- **EN:** Top-level classes include `Mixer2RMSNormGated`, `MambaMixer2`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `Mixer2RMSNormGated`, `MambaMixer2`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `mamba_v2_sharded_weight_loader`, `mamba_mixer2`, `mamba_mixer2_fake` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `mamba_v2_sharded_weight_loader`, `mamba_mixer2`, `mamba_mixer2_fake` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm.config`, `vllm.distributed`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.mamba.abstract`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.model_executor.layers.mamba.ops.causal_conv1d`, `vllm.model_executor.layers.mamba.ops.layernorm_gated`, `vllm.model_executor.layers.mamba.ops.ssd_combined`, `vllm.model_executor.layers.mamba.ops.ssu_dispatch`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.model_loader.weight_utils`, `vllm.model_executor.parameter`, `vllm.model_executor.utils`, `vllm.platforms`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.mamba2_attn`
