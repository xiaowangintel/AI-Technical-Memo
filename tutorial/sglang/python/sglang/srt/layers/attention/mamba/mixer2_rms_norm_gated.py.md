# mixer2_rms_norm_gated.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/mamba/mixer2_rms_norm_gated.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the main mixer2 rms norm gated classes used by the SGLang attention stack, together with helper routines that support runtime execution. / 该模块定义 SGLang 注意力栈使用的主要 mixer2 rms norm gated 类，并提供支撑运行时执行的辅助例程。
## Line-by-Line Analysis / 逐行分析
### Lines 1-16: imports
```python
from typing import Union

import torch

from sglang.srt.distributed.communication_op import (
    tensor_model_parallel_all_gather,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.distributed.parallel_state import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from sglang.srt.layers.attention.fla.layernorm_gated import rms_norm_gated
from sglang.srt.layers.utils import MultiPlatformOp
from sglang.srt.model_loader.weight_utils import sharded_weight_loader
from sglang.srt.utils.common import set_weight_attrs
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 19-19: class Mixer2RMSNormGated
```python
class Mixer2RMSNormGated(MultiPlatformOp):
```
**EN:** Defines the mixer2 rmsnorm gated type and the state it exposes to the rest of the attention stack.
**CN:** 定义 mixer2 rmsnorm gated 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 20-46: method Mixer2RMSNormGated.__init__
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
            self.weight = torch.nn.Parameter(torch.ones(self.per_rank_hidden_size))
            set_weight_attrs(self.weight, {"weight_loader": sharded_weight_loader(0)})
        else:
            # Avoid checkpoint mismatch by skipping unused parameter
            self.register_parameter("weight", None)
        assert (
            self.full_hidden_size % self.tp_size == 0
        ), "Tensor parallel world size must divide hidden size."
```
**EN:** Initializes the Mixer2RMSNormGated instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 Mixer2RMSNormGated 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 48-97: method Mixer2RMSNormGated.forward_native
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
        x = x * torch.nn.functional.silu(gate.to(torch.float32))
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
# ... omitted 6 lines ...
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
**EN:** Runs the forward-path logic for forward native, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward native 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 99-120: method Mixer2RMSNormGated.forward_cuda
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        gate: torch.Tensor,
    ) -> Union[torch.Tensor, tuple[torch.Tensor, torch.Tensor]]:
        input_dtype = x.dtype
        if not self.use_rms_norm:
            # Keep gate in float32 for numerical stability during silu
            return x * torch.nn.functional.silu(gate.to(torch.float32)).to(input_dtype)

        if ((self.n_groups % self.tp_size) != 0) or self.n_groups != 1:
            return self.forward_native(x, gate)

        return rms_norm_gated(
            x=x,
            weight=self.weight.data,
            bias=None,
            z=gate,
            eps=self.variance_epsilon,
            norm_before_gate=False,
            is_rms_norm=True,
        )
```
**EN:** Runs the forward-path logic for forward cuda, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward cuda 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调

## Dependencies / 依赖关系
- `typing.Union`
- `torch`
- `sglang.srt.distributed.communication_op.tensor_model_parallel_all_gather`
- `sglang.srt.distributed.communication_op.tensor_model_parallel_all_reduce`
- `sglang.srt.distributed.parallel_state.get_tensor_model_parallel_rank`
- `sglang.srt.distributed.parallel_state.get_tensor_model_parallel_world_size`
- `sglang.srt.layers.attention.fla.layernorm_gated.rms_norm_gated`
- `sglang.srt.layers.utils.MultiPlatformOp`
- `sglang.srt.model_loader.weight_utils.sharded_weight_loader`
- `sglang.srt.utils.common.set_weight_attrs`
