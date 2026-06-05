# grammar.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/grammar.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `grammar`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `grammar` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Imports and module setup
````python
from typing import List, Optional, Union

import torch
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 6-15: `apply_token_bitmask_inplace_cuda` definition
````python
def apply_token_bitmask_inplace_cuda(
    logits: torch.Tensor,
    bitmask: torch.Tensor,
    indices: Optional[Union[List[int], torch.Tensor]] = None,
) -> None:
    if isinstance(indices, list):
        indices = torch.tensor(indices, dtype=torch.int32, device=logits.device)
    if indices is not None:
        indices = indices.to(logits.device)
    torch.ops.sgl_kernel.apply_token_bitmask_inplace_cuda(logits, bitmask, indices)
````
**EN:** This section defines `apply_token_bitmask_inplace_cuda`. It applies the `token bitmask inplace cuda` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `apply_token_bitmask_inplace_cuda`。它负责应用模块中与 `token bitmask inplace cuda` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `apply_token_bitmask_inplace_cuda`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `torch`, `typing`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
