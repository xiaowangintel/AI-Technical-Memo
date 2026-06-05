# ngram_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/ngram_embedding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "from __future__ import annotations". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“from __future__ import annotations”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and module setup
```python
from __future__ import annotations

from typing import TYPE_CHECKING

from sglang.jit_kernel.utils import cache_once, load_jit
from sglang.kernel_api_logging import debug_kernel_api

if TYPE_CHECKING:
    import torch
    from tvm_ffi.module import Module


@cache_once
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 14-25: Function `_jit_ngram_embedding_module`
```python
def _jit_ngram_embedding_module() -> Module:
    return load_jit(
        "ngram_embedding",
        cuda_files=["ngram_embedding.cuh"],
        cuda_wrappers=[
            ("compute_n_gram_ids", "&NgramEmbeddingKernel::compute_n_gram_ids"),
            ("update_token_table", "&NgramEmbeddingKernel::update_token_table"),
        ],
    )


@debug_kernel_api
```
**EN:** This block defines `_jit_ngram_embedding_module`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_jit_ngram_embedding_module`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 26-71: Function `compute_n_gram_ids`
```python
def compute_n_gram_ids(
    ne_n: int,
    ne_k: int,
    ne_weights: torch.Tensor,
    ne_mods: torch.Tensor,
    exclusive_ne_embedder_size_sums: torch.Tensor,
    tokens: torch.Tensor,
    exclusive_req_len_sums: torch.Tensor,
    ne_token_table: torch.Tensor,
    row_indices: torch.Tensor,
    column_starts: torch.Tensor,
    n_gram_ids: torch.Tensor,
) -> None:
    """
    Compute n-gram IDs for embedding.

    Args:
        ne_n: n value for n-gram
        ne_k: k value for n-gram configurations
        ne_weights: weights tensor with shape [ne_n-1, ne_k, ne_n]
        ne_mods: mods tensor with shape [ne_n-1, ne_k]
        exclusive_ne_embedder_size_sums: exclusive sum of embedder sizes
        tokens: input token ids
        exclusive_req_len_sums: exclusive sum of request lengths
        ne_token_table: token table for all requests
        row_indices: row indices for each request
        column_starts: column start positions for each request
        n_gram_ids: output tensor for n-gram ids
    """
    module = _jit_ngram_embedding_module()
    module.compute_n_gram_ids(
        ne_n,
        ne_k,
        ne_weights,
        ne_mods,
        exclusive_ne_embedder_size_sums,
        tokens,
        exclusive_req_len_sums,
        ne_token_table,
        row_indices,
        column_starts,
        n_gram_ids,
    )


@debug_kernel_api
```
**EN:** This block defines `compute_n_gram_ids`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `compute_n_gram_ids`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 72-102: Function `update_token_table`
```python
def update_token_table(
    tokens: torch.Tensor,
    ne_token_table: torch.Tensor,
    row_indices: torch.Tensor,
    column_starts: torch.Tensor,
    req_lens: torch.Tensor,
    ignore_tokens: torch.Tensor | None = None,
) -> None:
    """
    Update the token table with new tokens.

    Args:
        tokens: input token ids
        ne_token_table: token table for all requests
        row_indices: row indices for each request
        column_starts: column start positions for each request
        req_lens: request lengths
        ignore_tokens: tokens to be ignored (marked as negative in table)
    """
    module = _jit_ngram_embedding_module()
    if ignore_tokens is None:
        # Create an empty tensor for ignore_tokens
        ignore_tokens = tokens.new_empty(0, dtype=tokens.dtype)
    module.update_token_table(
        tokens,
        ne_token_table,
        row_indices,
        column_starts,
        req_lens,
        ignore_tokens,
    )
```
**EN:** This block defines `update_token_table`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `update_token_table`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `__future__ -> annotations`
- `typing -> TYPE_CHECKING`
- `sglang.jit_kernel.utils -> cache_once`
- `sglang.kernel_api_logging -> debug_kernel_api`
- `torch`
- `tvm_ffi.module -> Module`
