# top_k.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/top_k.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `top-k selection`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `top-k selection` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Imports and module setup
````python
from typing import Optional

import torch
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 6-13: `fast_topk` definition
````python
def fast_topk(values, topk, dim):
    if topk == 1:
        # Use max along the specified dimension to get both value and index
        return torch.max(values, dim=dim, keepdim=True)
    else:
        # Use topk for efficiency with larger k values
        # TODO: implement faster cuda kernels for large vocab sizes
        return torch.topk(values, topk, dim=dim)
````
**EN:** This section defines `fast_topk` and implements the core logic associated with fast topk.
**CN:** 该部分定义 `fast_topk`，并实现与 fast topk 相关的核心逻辑。

### Lines 16-42: `fast_topk_v2` definition
````python
def fast_topk_v2(
    score: torch.Tensor,
    lengths: torch.Tensor,
    topk: int,
    row_starts: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    """
    Get the topk indices of the score tensor.
    Args:
        score: The score tensor of shape (B, L). The score tensor is the logits
            between the query and the key whose layout is either ragged or paged.
            row_starts is only required when the key is ragged.
        lengths: The lengths tensor of shape (B)
        topk: The number of topk indices to get
        row_starts: The start index of each row in the score tensor of shape (B).
            For each row i, topk only applies to section [row_starts[i], row_starts[i] + lengths[i]]
            of the score tensor.
    Returns:
        The topk indices tensor of shape (B, topk)
    """
    assert (
        topk == 2048
    ), "fast_topk_v2 is only optimized for deepseek v3.2 model, where topk=2048"
    assert score.dim() == 2
    topk_indices = score.new_empty((score.size(0), topk), dtype=torch.int32)
    torch.ops.sgl_kernel.fast_topk(score, topk_indices, lengths, row_starts)
    return topk_indices
````
**EN:** This section defines `fast_topk_v2` and implements the core logic associated with fast topk v2. Docstring summary: Get the topk indices of the score tensor. It also performs explicit assertions or shape checks before continuing. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `fast_topk_v2`，并实现与 fast topk v2 相关的核心逻辑。 文档字符串摘要：Get the topk indices of the score tensor. 它还会在继续执行前进行显式断言或形状检查。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 45-80: `fast_topk_transform_fused` definition
````python
def fast_topk_transform_fused(
    score: torch.Tensor,
    lengths: torch.Tensor,
    page_table_size_1: torch.Tensor,  # NOTE: page size should be 1
    cu_seqlens_q: torch.Tensor,
    topk: int,
    row_starts: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    """
    Get the topk indices of the score tensor and then transform the topk indices
    to indices to the page table (page_size = 1)
    Args:
        score: The score tensor of shape (B, L). The score tensor is the logits
            between the query and the key whose layout is either ragged or paged.
            row_starts is only required when the key is ragged.
        lengths: The lengths tensor of shape (B)
        page_table_size_1: The page table tensor of shape (Batch, topk)
        cu_seqlens_q: The cumulative sequence lengths tensor of shape (Batch + 1)
        topk: The number of topk indices to get
        row_starts: The start index of each row in the score tensor of shape (B).
            For each row i, topk only applies to section [row_starts[i], row_starts[i] + lengths[i]]
            of the score tensor. It's only used for cases where the key is
            ragged, i.e. during extend and draft extend.
    Returns:
        The topk indices tensor of shape (B, topk)
    """
    assert (
        topk == 2048
    ), "fast_topk_transform_fused is only optimized for deepseek v3.2 model, where topk=2048"
    assert score.dim() == 2
    src_page_table = page_table_size_1
    dst_page_table = score.new_empty((score.shape[0], topk), dtype=torch.int32)
    torch.ops.sgl_kernel.fast_topk_transform_fused(
        score, lengths, dst_page_table, src_page_table, cu_seqlens_q, row_starts
    )
    return dst_page_table
````
**EN:** This section defines `fast_topk_transform_fused` and implements the core logic associated with fast topk transform fused. Docstring summary: Get the topk indices of the score tensor and then transform the topk indices to indices to the page table (page_size = 1) Args: score: The score tensor of shape (B, L). It also performs explicit assertions or shape checks before continuing. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `fast_topk_transform_fused`，并实现与 fast topk transform fused 相关的核心逻辑。 文档字符串摘要：Get the topk indices of the score tensor and then transform the topk indices to indices to the page table (page_size = 1) Args: score: The score tensor of shape (B, L). 它还会在继续执行前进行显式断言或形状检查。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 83-117: `fast_topk_transform_ragged_fused` definition
````python
def fast_topk_transform_ragged_fused(
    score: torch.Tensor,
    lengths: torch.Tensor,
    topk_indices_offset: torch.Tensor,  # ragged kv
    topk: int,
    row_starts: Optional[torch.Tensor] = None,
) -> torch.Tensor:
    """
    Get the topk indices of the score tensor and then transform the topk indices to
    indices to ragged kv (non-paged). This function is only used for extend,
    not including draft extend.
    Args:
        score: The score tensor of shape (B, L). The score tensor is the logits
            between the query and the key which can be ragged or paged.
            row_starts is only required when the key is ragged.
        lengths: The lengths tensor of shape (B)
        topk_indices_offset: The offset of topk indices in ragged kv of shape (B)
        topk: The number of topk indices to get
        row_starts: The start index of each row in the score tensor of shape (B).
            For each row i, topk only applies to section [row_starts[i], row_starts[i] + lengths[i]]
            of the score tensor. It can be None if only the fast path is triggered,
            in the case of all values in lengths <= topk (not checked in the kernel,
            guaranteed by the caller).
    Returns:
        The topk indices tensor of shape (B, topk)
    """
    assert (
        topk == 2048
    ), "fast_topk_transform_ragged_fused is only optimized for deepseek v3.2 model, where topk=2048"
    assert score.dim() == 2
    topk_indices_ragged = score.new_empty((score.shape[0], topk), dtype=torch.int32)
    torch.ops.sgl_kernel.fast_topk_transform_ragged_fused(
        score, lengths, topk_indices_ragged, topk_indices_offset, row_starts
    )
    return topk_indices_ragged
````
**EN:** This section defines `fast_topk_transform_ragged_fused` and implements the core logic associated with fast topk transform ragged fused. Docstring summary: Get the topk indices of the score tensor and then transform the topk indices to indices to ragged kv (non-paged). It also performs explicit assertions or shape checks before continuing. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `fast_topk_transform_ragged_fused`，并实现与 fast topk transform ragged fused 相关的核心逻辑。 文档字符串摘要：Get the topk indices of the score tensor and then transform the topk indices to indices to ragged kv (non-paged). 它还会在继续执行前进行显式断言或形状检查。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `fast_topk`, `fast_topk_v2`, `fast_topk_transform_fused`, `fast_topk_transform_ragged_fused`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `torch`, `typing`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
