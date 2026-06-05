# weak_ref_tensor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/compilation/weak_ref_tensor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the compilation and graph lowering part of the SRT runtime and implements logic centered on `weak_ref_tensor`. It exposes primary entry points such as `weak_ref_tensors`. / 该模块属于 SRT 运行时的编译与图优化部分，主要实现围绕 `weak_ref_tensor` 的逻辑。 它对外提供的主要入口包括 `weak_ref_tensors`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Module imports, constants, and setup
```python
from typing import Any, Union

import torch

from sglang.srt.utils.common import is_cuda, is_hip, is_musa, is_npu

if is_cuda() or is_hip() or is_musa():
    from sgl_kernel import weak_ref_tensor
elif is_npu():
    from torch_npu._C import _weak_ref_tensor as weak_ref_tensor
else:
    raise NotImplementedError("weak_ref_tensor is implemented only for CUDA and NPU.")


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 15-28: Function weak_ref_tensors
```python
def weak_ref_tensors(
    tensors: Union[torch.Tensor, list[torch.Tensor], tuple[torch.Tensor]],
) -> Union[torch.Tensor, list[Any], tuple[Any], Any]:
    """
    Convenience function to create weak references to tensors,
    for single tensor, list of tensors or tuple of tensors.
    """
    if isinstance(tensors, torch.Tensor):
        return weak_ref_tensor(tensors)
    if isinstance(tensors, list):
        return [weak_ref_tensor(t) for t in tensors]
    if isinstance(tensors, tuple):
        return tuple(weak_ref_tensor(t) for t in tensors)
    raise ValueError("Invalid type for tensors")
```
**EN:** This callable implements `weak_ref_tensors`. It takes `tensors` and mainly implements weak ref tensors. The docstring states: "Convenience function to create weak references to tensors, for single tensor, list of tensors or tuple of tensors." In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `weak_ref_tensors`。它接收 `tensors`，主要用于实现 weak ref tensors 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

## Key Concepts / 关键概念
- `weak_ref_tensors`: implements weak ref tensors / 实现 weak ref tensors 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: `torch`, `sgl_kernel`, `torch_npu._C`
- **Internal modules / 内部模块**: `sglang.srt.utils.common`
