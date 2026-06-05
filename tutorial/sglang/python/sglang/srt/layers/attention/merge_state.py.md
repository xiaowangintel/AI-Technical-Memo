# merge_state.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/merge_state.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the merge state structures that carry execution metadata through the attention pipeline. / 该模块定义 merge state 结构，用于在注意力流水线中传递执行元数据。
## Line-by-Line Analysis / 逐行分析
### Lines 1-7: imports
```python
from typing import Optional, Tuple

import torch
from sgl_kernel import merge_state_v2

from sglang.srt.layers.attention.triton_ops.merge_state import merge_state_triton
from sglang.srt.utils import is_cuda
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 9-9: module constants
```python
_is_cuda = is_cuda()
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 15-16: function _supported_dtypes
```python
def _supported_dtypes(o: torch.Tensor) -> bool:
    return o.dtype in [torch.float32, torch.half, torch.bfloat16]
```
**EN:** Implements the supported dtypes routine used by this attention module.
**CN:** 实现该注意力模块使用的 supported dtypes 例程。

### Lines 19-23: function _supported_headdim
```python
def _supported_headdim(o: torch.Tensor) -> bool:
    headdim = o.shape[2]  # [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    if o.dtype == torch.float32:
        return headdim % 4 == 0
    return headdim % 8 == 0
```
**EN:** Implements the supported headdim routine used by this attention module.
**CN:** 实现该注意力模块使用的 supported headdim 例程。

### Lines 26-46: function merge_state
```python
def merge_state(
    prefix_output: torch.Tensor,
    prefix_lse: torch.Tensor,
    suffix_output: torch.Tensor,
    suffix_lse: torch.Tensor,
    output: Optional[torch.Tensor] = None,
    output_lse: Optional[torch.Tensor] = None,
) -> Tuple[torch.Tensor, Optional[torch.Tensor]]:
    if (
        _is_cuda
        and _supported_dtypes(prefix_output)
        and _supported_headdim(prefix_output)
    ):
        return merge_state_v2(
            prefix_output, prefix_lse, suffix_output, suffix_lse, output, output_lse
        )
    else:
        # Fallback to Triton kernel
        return merge_state_triton(
            prefix_output, prefix_lse, suffix_output, suffix_lse, output, output_lse
        )
```
**EN:** Implements the merge state routine used by this attention module.
**CN:** 实现该注意力模块使用的 merge state 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `sgl_kernel.merge_state_v2`
- `sglang.srt.layers.attention.triton_ops.merge_state.merge_state_triton`
- `sglang.srt.utils.is_cuda`
