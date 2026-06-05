# kda_triton.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/linear/kernels/kda_triton.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module wraps low-level kda triton compute routines used by the attention stack. It focuses on tensor layout, launch preparation, and accelerated execution details. / 该模块封装注意力栈使用的底层 kda triton 计算例程，重点处理张量布局、启动准备和加速执行细节。
## Line-by-Line Analysis / 逐行分析
### Lines 1-8: imports
```python
from typing import Optional

import torch

from sglang.srt.layers.attention.linear.kernels.kernel_backend import (
    LinearAttnKernelBase,
)
from sglang.srt.utils import is_cpu
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 10-14: conditional branch
```python
if not is_cpu():
    from sglang.srt.layers.attention.fla.fused_sigmoid_gating_recurrent import (
        fused_sigmoid_gating_delta_rule_update,
    )
    from sglang.srt.layers.attention.fla.kda import chunk_kda
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 17-19: class TritonKDAKernel
```python
class TritonKDAKernel(LinearAttnKernelBase):
    """Triton-based kernel for KDA (Kimi Delta Attention) linear attention."""
```
**EN:** Defines the triton kdakernel type and the state it exposes to the rest of the attention stack.
**CN:** 定义 triton kdakernel 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 20-50: method TritonKDAKernel.decode
```python
    def decode(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        a: torch.Tensor,
        b: torch.Tensor,
        *,
        A_log: torch.Tensor,
        dt_bias: torch.Tensor,
        ssm_states: torch.Tensor,
        cache_indices: torch.Tensor,
        query_start_loc: torch.Tensor,
        **kwargs,
    ) -> torch.Tensor:
        return fused_sigmoid_gating_delta_rule_update(
            A_log=A_log,
            dt_bias=dt_bias,
            q=q,
            k=k,
            v=v,
            a=a,
            b=b,
            initial_state_source=ssm_states,
            initial_state_indices=cache_indices,
            cu_seqlens=query_start_loc,
            use_qk_l2norm_in_kernel=True,
            softplus_beta=1.0,
            softplus_threshold=20.0,
            is_kda=True,
        )
```
**EN:** Implements the decode routine used by this attention module.
**CN:** 实现该注意力模块使用的 decode 例程。

### Lines 52-81: method TritonKDAKernel.extend
```python
    def extend(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        g: torch.Tensor,
        beta: torch.Tensor,
        *,
        ssm_states: torch.Tensor,
        cache_indices: torch.Tensor,
        query_start_loc: torch.Tensor,
        A_log: Optional[torch.Tensor] = None,
        dt_bias: Optional[torch.Tensor] = None,
        lower_bound: Optional[float] = None,
        **kwargs,
    ) -> torch.Tensor:
        return chunk_kda(
            q=q,
            k=k,
            v=v,
            g=g,
            beta=beta,
            initial_state=ssm_states,
            initial_state_indices=cache_indices,
            use_qk_l2norm_in_kernel=True,
            cu_seqlens=query_start_loc,
            A_log=A_log,
            dt_bias=dt_bias,
            lower_bound=lower_bound,
        )
```
**EN:** Implements the extend routine used by this attention module.
**CN:** 实现该注意力模块使用的 extend 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成

## Dependencies / 依赖关系
- `typing.Optional`
- `torch`
- `sglang.srt.layers.attention.linear.kernels.kernel_backend.LinearAttnKernelBase`
- `sglang.srt.utils.is_cpu`
- `sglang.srt.layers.attention.fla.fused_sigmoid_gating_recurrent.fused_sigmoid_gating_delta_rule_update`
- `sglang.srt.layers.attention.fla.kda.chunk_kda`
