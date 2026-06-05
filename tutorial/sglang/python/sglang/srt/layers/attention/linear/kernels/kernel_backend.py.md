# kernel_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/linear/kernels/kernel_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the kernel backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 kernel backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-3: imports
```python
from abc import ABC, abstractmethod

import torch
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 6-12: class LinearAttnKernelBase
```python
class LinearAttnKernelBase(ABC):
    """Abstract base class for linear attention kernel implementations.

    Each concrete implementation wraps a specific kernel (Triton, CuTe DSL, etc.)
    and provides decode/extend/target_verify methods with a unified interface.
    """
```
**EN:** Defines the linear attn kernel base type and the state it exposes to the rest of the attention stack.
**CN:** 定义 linear attn kernel base 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 13-28: method LinearAttnKernelBase.decode
```python
    @abstractmethod
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
    ) -> torch.Tensor: ...
```
**EN:** Implements the decode routine used by this attention module.
**CN:** 实现该注意力模块使用的 decode 例程。

### Lines 30-43: method LinearAttnKernelBase.extend
```python
    @abstractmethod
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
        **kwargs,
    ) -> tuple: ...
```
**EN:** Implements the extend routine used by this attention module.
**CN:** 实现该注意力模块使用的 extend 例程。

### Lines 45-62: method LinearAttnKernelBase.target_verify
```python
    def target_verify(
        self,
        A_log: torch.Tensor,
        dt_bias: torch.Tensor,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        a: torch.Tensor,
        b: torch.Tensor,
        *,
        ssm_states: torch.Tensor,
        cache_indices: torch.Tensor,
        query_start_loc: torch.Tensor,
        **kwargs,
    ) -> torch.Tensor:
        raise NotImplementedError(
            f"{self.__class__.__name__} does not support target_verify"
        )
```
**EN:** Implements the target verify routine used by this attention module.
**CN:** 实现该注意力模块使用的 target verify 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** Speculative decoding support / **CN:** 投机解码支持

## Dependencies / 依赖关系
- `abc.ABC`
- `abc.abstractmethod`
- `torch`
