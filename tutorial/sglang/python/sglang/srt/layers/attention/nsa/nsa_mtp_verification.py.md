# nsa_mtp_verification.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/nsa/nsa_mtp_verification.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages nsa mtp verification logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 nsa mtp verification 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-6: docstring
```python
"""
Verification utilities for NSA backend fused metadata copy operations.

This module contains verification code to ensure that fused metadata copy kernels
produce the same results as individual copy operations.
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 8-8: imports
```python
import torch
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 11-204: function verify_single_backend_fused_metadata_copy
```python
def verify_single_backend_fused_metadata_copy(
    metadata,
    precomputed,
    forward_mode,
    bs,
    flashmla_num_splits_src=None,
    flashmla_metadata_src=None,
    flashmla_num_splits_dst=None,
    flashmla_metadata_dst=None,
):
    """
    Verify that the fused metadata copy kernel produces the same results as individual copies.

    Args:
        metadata: The NSA metadata object containing destination tensors
        precomputed: The precomputed metadata containing source tensors
        forward_mode: The forward mode (decode, target_verify, or draft_extend)
        bs: Batch size
        flashmla_num_splits_src: Source FlashMLA num_splits tensor (optional)
        flashmla_metadata_src: Source FlashMLA metadata tensor (optional)
        flashmla_num_splits_dst: Destination FlashMLA num_splits tensor (optional)
        flashmla_metadata_dst: Destination FlashMLA metadata tensor (optional)

    Raises:
        RuntimeError: If verification fails (tensors don't match)
    """
    # Clone destination tensors to preserve fused kernel results
    fused_cache_seqlens = metadata.cache_seqlens_int32.clone()
    fused_cu_seqlens_k = metadata.cu_seqlens_k.clone()
    fused_page_table_1 = metadata.page_table_1.clone()
    fused_nsa_cache_seqlens = metadata.nsa_cache_seqlens_int32.clone()
    fused_nsa_seqlens_expanded = metadata.nsa_seqlens_expanded.clone()
# ... omitted 150 lines ...
    if precomputed.flashmla_metadata is not None:
        size = precomputed.seqlens_expanded_size
        check_tensor_equal(
            "flashmla_num_splits",
            fused_flashmla_num_splits[: size + 1],
            ref_flashmla_num_splits[: size + 1],
        )
        check_tensor_equal(
            "flashmla_metadata",
            fused_flashmla_metadata,
            ref_flashmla_metadata,
        )
```
**EN:** Implements the verify single backend fused metadata copy routine used by this attention module.
**CN:** 实现该注意力模块使用的 verify single backend fused metadata copy 例程。

### Lines 207-407: function verify_multi_backend_fused_metadata_copy
```python
def verify_multi_backend_fused_metadata_copy(
    metadata0,
    metadata1,
    metadata2,
    precomputed,
    bs,
    flashmla_num_splits_src=None,
    flashmla_metadata_src=None,
):
    """
    Verify that the multi-backend fused metadata copy kernel produces the same results
    as individual copies for all three backends.

    Args:
        metadata0: The NSA metadata object for backend 0
        metadata1: The NSA metadata object for backend 1
        metadata2: The NSA metadata object for backend 2
        precomputed: The precomputed metadata containing source tensors
        bs: Batch size
        flashmla_num_splits_src: Source FlashMLA num_splits tensor (optional)
        flashmla_metadata_src: Source FlashMLA metadata tensor (optional)

    Raises:
        RuntimeError: If verification fails (tensors don't match)
    """
    # Clone destination tensors to preserve fused kernel results
    fused_results = []
    for idx, metadata in enumerate([metadata0, metadata1, metadata2]):
        fused_cache_seqlens = metadata.cache_seqlens_int32.clone()
        fused_cu_seqlens_k = metadata.cu_seqlens_k.clone()
        fused_page_table_1 = metadata.page_table_1.clone()
        fused_nsa_cache_seqlens = metadata.nsa_cache_seqlens_int32.clone()
# ... omitted 157 lines ...
            check_tensor_equal(
                idx,
                "flashmla_num_splits",
                fused["flashmla_num_splits"][: bs + 1],
                ref["flashmla_num_splits"][: bs + 1],
            )
            check_tensor_equal(
                idx,
                "flashmla_metadata",
                fused["flashmla_metadata"],
                ref["flashmla_metadata"],
            )
```
**EN:** Implements the verify multi backend fused metadata copy routine used by this attention module.
**CN:** 实现该注意力模块使用的 verify multi backend fused metadata copy 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Flash-style fused attention kernels / **CN:** Flash 风格融合注意力内核
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Speculative decoding support / **CN:** 投机解码支持
- **EN:** Compressed or sparse attention helpers / **CN:** 压缩或稀疏注意力辅助逻辑

## Dependencies / 依赖关系
- `torch`
