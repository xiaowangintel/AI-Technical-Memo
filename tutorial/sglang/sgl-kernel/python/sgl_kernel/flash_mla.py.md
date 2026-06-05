# flash_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/flash_mla.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `FlashMLA`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `FlashMLA` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Imports and module setup
````python
from typing import Optional, Tuple

import torch
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 5-10: Optional dependency handling
````python
try:
    from sgl_kernel import flashmla_ops  # triggers TORCH extension registration
except Exception as _e:
    _flashmla_import_error = _e
else:
    _flashmla_import_error = None
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 12-14: Constants and configuration
````python
_IMPORT_ERROR = ImportError(
    "Failed to load sgl_kernel.flashmla_ops extension. Ensure CUDA Driver >= 12.4"
)
````
**EN:** This block defines shared constants or configuration values such as `_IMPORT_ERROR`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_IMPORT_ERROR`），供后续函数或控制流程复用。

### Lines 17-54: `get_mla_metadata` definition
````python
def get_mla_metadata(
    cache_seqlens: torch.Tensor,
    num_q_tokens_per_head_k: int,
    num_heads_k: int,
    num_heads_q: Optional[int] = None,
    is_fp8_kvcache: bool = False,
    topk: Optional[int] = None,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """
    Arguments:
        cache_seqlens: (batch_size), dtype torch.int32.
        num_q_tokens_per_head_k: Equals to num_q_tokens_per_q_seq * num_heads_q // num_heads_k.
        num_heads_k: The number of k heads.
        num_heads_q: The number of q heads. This argument is optional when sparse attention is not enabled
        is_fp8_kvcache: Whether the k_cache and v_cache are in fp8 format.
        topk: If not None, sparse attention will be enabled, and only tokens in the `indices` array passed to `flash_mla_with_kvcache_sm90` will be attended to.

    Returns:
        tile_scheduler_metadata: (num_sm_parts, TileSchedulerMetaDataSize), dtype torch.int32.
        num_splits: (batch_size + 1), dtype torch.int32.
    """
    if _flashmla_import_error is not None:
        raise _IMPORT_ERROR from _flashmla_import_error

    if is_fp8_kvcache and topk is None:
        return torch.ops.sgl_kernel.get_mla_decoding_metadata_dense_fp8.default(
            cache_seqlens,
            num_q_tokens_per_head_k,
            num_heads_k,
        )
    return torch.ops.sgl_kernel.get_mla_decoding_metadata.default(
        cache_seqlens,
        num_q_tokens_per_head_k,
        num_heads_k,
        num_heads_q,
        is_fp8_kvcache,
        topk,
    )
````
**EN:** This section defines `get_mla_metadata`. It retrieves or computes the `MLA metadata` path used by the module. Docstring summary: Arguments: cache_seqlens: (batch_size), dtype torch.int32. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `get_mla_metadata`。它负责获取或计算模块中与 `MLA metadata` 相关的处理路径。 文档字符串摘要：Arguments: cache_seqlens: (batch_size), dtype torch.int32. 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 57-131: `flash_mla_with_kvcache` definition
````python
def flash_mla_with_kvcache(
    q: torch.Tensor,
    k_cache: torch.Tensor,
    block_table: torch.Tensor,
    cache_seqlens: torch.Tensor,
    head_dim_v: int,
    tile_scheduler_metadata: torch.Tensor,
    num_splits: torch.Tensor,
    softmax_scale: Optional[float] = None,
    causal: bool = False,
    descale_q: torch.Tensor | None = None,
    descale_k: torch.Tensor | None = None,
    is_fp8_kvcache: bool = False,
    indices: Optional[torch.Tensor] = None,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """
    Arguments:
        q: (batch_size, seq_len_q, num_heads_q, head_dim).
        k_cache: (num_blocks, page_block_size, num_heads_k, head_dim).
        block_table: (batch_size, max_num_blocks_per_seq), torch.int32.
        cache_seqlens: (batch_size), torch.int32.
        head_dim_v: Head dimension of v.
        tile_scheduler_metadata: (num_sm_parts, TileSchedulerMetaDataSize), torch.int32, returned by get_mla_metadata.
        num_splits: (batch_size + 1), torch.int32, returned by get_mla_metadata.
        softmax_scale: float. The scale of QK^T before applying softmax. Default to 1 / sqrt(head_dim).
        causal: bool. Whether to apply causal attention mask.
        descale_q: (batch_size), torch.float32. Descaling factors for Q, used for fp8 quantization.
        descale_k: (batch_size), torch.float32. Descaling factors for K, used for fp8 quantization.
        is_fp8_kvcache: bool. Whether the k_cache and v_cache are in fp8 format. For the format of FP8 KV cache, please refer to README.md
        indices: (batch_size, seq_len_q, topk), torch.int32. If not None, sparse attention will be enabled, and only tokens in the `indices` array will be attended to. Invalid indices should be set to -1 or numbers >= total_seq_len_kv. For details about how to set up `indices`, please refer to README.md.

    Returns:
        out: (batch_size, seq_len_q, num_heads_q, head_dim_v).
        softmax_lse: (batch_size, num_heads_q, seq_len_q), torch.float32.
    """
    if _flashmla_import_error is not None:
        raise _IMPORT_ERROR from _flashmla_import_error

    if softmax_scale is None:
        softmax_scale = q.shape[-1] ** (-0.5)
    if indices is not None:
        assert causal == False, "causal must be `false` if sparse attention is enabled."
    assert (descale_q is None) == (
        descale_k is None
    ), "descale_q and descale_k should be both None or both not None"

    if indices is None and q.element_size() == 1:
        out, softmax_lse = torch.ops.sgl_kernel.fwd_kvcache_mla_fp8.default(
            q,
            k_cache,
            head_dim_v,
            cache_seqlens,
            block_table,
            softmax_scale,
            causal,
            tile_scheduler_metadata,
            num_splits,
            descale_q,
            descale_k,
        )
    else:
        out, softmax_lse = torch.ops.sgl_kernel.fwd_kvcache_mla.default(
            q,
            k_cache,
            head_dim_v,
            cache_seqlens,
            block_table,
            softmax_scale,
            causal,
            tile_scheduler_metadata,
            num_splits,
            is_fp8_kvcache,
            indices,
        )
    return out, softmax_lse
````
**EN:** This section defines `flash_mla_with_kvcache` and implements the core logic associated with flash MLA with kvcache. Docstring summary: Arguments: q: (batch_size, seq_len_q, num_heads_q, head_dim). It also performs explicit assertions or shape checks before continuing. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `flash_mla_with_kvcache`，并实现与 flash MLA with kvcache 相关的核心逻辑。 文档字符串摘要：Arguments: q: (batch_size, seq_len_q, num_heads_q, head_dim). 它还会在继续执行前进行显式断言或形状检查。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 134-164: `flash_mla_sparse_fwd` definition
````python
def flash_mla_sparse_fwd(
    q: torch.Tensor,
    kv: torch.Tensor,
    indices: torch.Tensor,
    sm_scale: float,
    d_v: int = 512,
) -> Tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    Sparse attention prefill kernel

    Args:
        q: [s_q, h_q, d_qk], bfloat16
        kv: [s_kv, h_kv, d_qk], bfloat16
        indices: [s_q, h_kv, topk], int32. Invalid indices should be set to -1 or numbers >= s_kv
        sm_scale: float
        d_v: The dimension of value vectors. Can only be 512

    Returns:
        (output, max_logits, lse)
        About the definition of output, max_logits and lse, please refer to README.md
        - output: [s_q, h_q, d_v], bfloat16
        - max_logits:  [s_q, h_q], float
        - lse: [s_q, h_q], float, 2-based log-sum-exp
    """
    if _flashmla_import_error is not None:
        raise _IMPORT_ERROR from _flashmla_import_error

    results = torch.ops.sgl_kernel.sparse_prefill_fwd.default(
        q, kv, indices, sm_scale, d_v
    )
    return results
````
**EN:** This section defines `flash_mla_sparse_fwd` and implements the core logic associated with flash MLA sparse fwd. Docstring summary: Sparse attention prefill kernel Args: q: [s_q, h_q, d_qk], bfloat16 kv: [s_kv, h_kv, d_qk], bfloat16 indices: [s_q, h_kv, topk], int32. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `flash_mla_sparse_fwd`，并实现与 flash MLA sparse fwd 相关的核心逻辑。 文档字符串摘要：Sparse attention prefill kernel Args: q: [s_q, h_q, d_qk], bfloat16 kv: [s_kv, h_kv, d_qk], bfloat16 indices: [s_q, h_kv, topk], int32. 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `get_mla_metadata`, `flash_mla_with_kvcache`, `flash_mla_sparse_fwd`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`
- **External / 外部**: `torch`, `typing`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
