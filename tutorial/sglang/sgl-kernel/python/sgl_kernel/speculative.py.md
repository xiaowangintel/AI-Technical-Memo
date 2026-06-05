# speculative.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/speculative.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `speculative decoding`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `speculative decoding` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Imports and module setup
````python
import torch
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 4-35: `tree_speculative_sampling_target_only` definition
````python
def tree_speculative_sampling_target_only(
    predicts: torch.Tensor,  # mutable
    accept_index: torch.Tensor,  # mutable
    accept_token_num: torch.Tensor,  # mutable
    candidates: torch.Tensor,
    retrive_index: torch.Tensor,
    retrive_next_token: torch.Tensor,
    retrive_next_sibling: torch.Tensor,
    uniform_samples: torch.Tensor,
    uniform_samples_for_final_sampling: torch.Tensor,
    target_probs: torch.Tensor,
    draft_probs: torch.Tensor,
    threshold_single: float = 1.0,
    threshold_acc: float = 1.0,
    deterministic: bool = True,
) -> None:
    torch.ops.sgl_kernel.tree_speculative_sampling_target_only.default(
        predicts,
        accept_index,
        accept_token_num,
        candidates,
        retrive_index,
        retrive_next_token,
        retrive_next_sibling,
        uniform_samples,
        uniform_samples_for_final_sampling,
        target_probs,
        draft_probs,
        threshold_single,
        threshold_acc,
        deterministic,
    )
````
**EN:** This section defines `tree_speculative_sampling_target_only` and implements the core logic associated with tree speculative decoding sampling target only. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `tree_speculative_sampling_target_only`，并实现与 tree speculative decoding sampling target only 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 38-57: `verify_tree_greedy` definition
````python
def verify_tree_greedy(
    predicts: torch.Tensor,  # mutable
    accept_index: torch.Tensor,  # mutable
    accept_token_num: torch.Tensor,  # mutable
    candidates: torch.Tensor,
    retrive_index: torch.Tensor,
    retrive_next_token: torch.Tensor,
    retrive_next_sibling: torch.Tensor,
    target_predict: torch.Tensor,
) -> None:
    torch.ops.sgl_kernel.verify_tree_greedy.default(
        predicts,
        accept_index,
        accept_token_num,
        candidates,
        retrive_index,
        retrive_next_token,
        retrive_next_sibling,
        target_predict,
    )
````
**EN:** This section defines `verify_tree_greedy`. It verifies the `tree greedy` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `verify_tree_greedy`。它负责验证模块中与 `tree greedy` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 60-87: `build_tree_kernel_efficient` definition
````python
def build_tree_kernel_efficient(
    parent_list: torch.Tensor,
    selected_index: torch.Tensor,
    verified_seq_len: torch.Tensor,
    tree_mask: torch.Tensor,
    positions: torch.Tensor,
    retrive_index: torch.Tensor,
    retrive_next_token: torch.Tensor,
    retrive_next_sibling: torch.Tensor,
    topk: int,
    depth: int,
    draft_token_num: int,
    tree_mask_mode: int,
) -> None:
    torch.ops.sgl_kernel.build_tree_kernel_efficient.default(
        parent_list,
        selected_index,
        verified_seq_len,
        tree_mask,
        positions,
        retrive_index,
        retrive_next_token,
        retrive_next_sibling,
        topk,
        depth,
        draft_token_num,
        tree_mask_mode,
    )
````
**EN:** This section defines `build_tree_kernel_efficient`. It builds the `tree kernel efficient` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `build_tree_kernel_efficient`。它负责构建模块中与 `tree kernel efficient` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 90-109: `reconstruct_indices_from_tree_mask` definition
````python
def reconstruct_indices_from_tree_mask(
    tree_mask: torch.Tensor,
    verified_seq_len: torch.Tensor,
    positions: torch.Tensor,
    retrive_index: torch.Tensor,
    retrive_next_token: torch.Tensor,
    retrive_next_sibling: torch.Tensor,
    batch_size: int,
    draft_token_num: int,
) -> None:
    torch.ops.sgl_kernel.reconstruct_indices_from_tree_mask.default(
        tree_mask,
        verified_seq_len,
        positions,
        retrive_index,
        retrive_next_token,
        retrive_next_sibling,
        batch_size,
        draft_token_num,
    )
````
**EN:** This section defines `reconstruct_indices_from_tree_mask`. It reconstructs the `indices from tree mask` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `reconstruct_indices_from_tree_mask`。它负责重建模块中与 `indices from tree mask` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 112-126: `segment_packbits` definition
````python
def segment_packbits(
    x: torch.Tensor,
    input_indptr: torch.Tensor,
    output_indptr: torch.Tensor,
    y: torch.Tensor,
    batch_size: int,
) -> None:
    torch.ops.sgl_kernel.segment_packbits.default(
        x,
        input_indptr,
        output_indptr,
        y,
        batch_size,
        torch.cuda.current_stream().cuda_stream,
    )
````
**EN:** This section defines `segment_packbits` and implements the core logic associated with segment packbits. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `segment_packbits`，并实现与 segment packbits 相关的核心逻辑。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `tree_speculative_sampling_target_only`, `verify_tree_greedy`, `build_tree_kernel_efficient`, `reconstruct_indices_from_tree_mask`, `segment_packbits`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `torch`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
