# test_fused_metadata_copy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_fused_metadata_copy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: """"". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“"""”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Imports and module setup
```python
"""
Comprehensive tests for JIT-compiled fused metadata copy kernels.

This test suite verifies:
1. Single-backend fused kernel (fused_metadata_copy_cuda) - all forward modes
2. Multi-backend fused kernel (fused_metadata_copy_multi_cuda) - 3 backends at once
3. Correctness against reference implementations
4. Performance benchmarks and speedup measurements
"""

import sys
import time

import pytest
import torch

from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=100, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=400, suite="nightly-kernel-1-gpu", nightly=True)

# =============================================================================
# Helper Functions
# =============================================================================
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 27-130: Function `create_test_metadata`
```python
def create_test_metadata(
    bs: int,
    max_len: int,
    max_seqlen_k: int,
    seqlens_expanded_size: int,
    has_real_page_table: bool = False,
    has_flashmla: bool = False,
    device: str = "cuda",
):
    """Create test metadata tensors matching NSA backend structure."""
    # Basic tensors (always present)
    cache_seqlens_src = torch.randint(
        1, max_len, (bs,), dtype=torch.int32, device=device
    )
    cu_seqlens_k_src = torch.zeros(bs + 1, dtype=torch.int32, device=device)
    cu_seqlens_k_src[1:] = torch.cumsum(cache_seqlens_src, dim=0)

    page_indices_src = torch.randint(
        0, 1000, (bs, max_len), dtype=torch.int32, device=device
    )
    nsa_cache_seqlens_src = torch.randint(
        1, max_len, (seqlens_expanded_size,), dtype=torch.int32, device=device
    )
    seqlens_expanded_src = torch.randint(
        1, max_seqlen_k, (seqlens_expanded_size,), dtype=torch.int32, device=device
    )
    nsa_cu_seqlens_k_src = torch.zeros(
        seqlens_expanded_size + 1, dtype=torch.int32, device=device
    )
    nsa_cu_seqlens_k_src[1:] = torch.cumsum(nsa_cache_seqlens_src, dim=0)

    # Destination tensors
    cache_seqlens_dst = torch.zeros(bs, dtype=torch.int32, device=device)
    cu_seqlens_k_dst = torch.zeros(bs + 1, dtype=torch.int32, device=device)
    page_table_1_dst = torch.zeros((bs, max_len + 16), dtype=torch.int32, device=device)
    nsa_cache_seqlens_dst = torch.zeros(
        seqlens_expanded_size, dtype=torch.int32, device=device
    )
    nsa_seqlens_expanded_dst = torch.zeros(
        seqlens_expanded_size, dtype=torch.int32, device=device
    )
    nsa_cu_seqlens_k_dst = torch.zeros(
        seqlens_expanded_size + 1, dtype=torch.int32, device=device
    )

    # Optional tensors
    real_page_table_src = None
    real_page_table_dst = None
    if has_real_page_table:
        real_page_table_cols = max_len // 2
        real_page_table_src = torch.randint(
            0, 1000, (bs, real_page_table_cols), dtype=torch.int32, device=device
        )
        real_page_table_dst = torch.zeros(
            (bs, real_page_table_cols + 8), dtype=torch.int32, device=device
        )

    flashmla_num_splits_src = None
    flashmla_num_splits_dst = None
    flashmla_metadata_src = None
    flashmla_metadata_dst = None
    if has_flashmla:
        flashmla_num_splits_src = torch.randint(
            1, 10, (seqlens_expanded_size + 1,), dtype=torch.int32, device=device
        )
        flashmla_num_splits_dst = torch.zeros(
            seqlens_expanded_size + 1, dtype=torch.int32, device=device
        )
        # FlashMLA metadata is typically (num_sm_parts, TileSchedulerMetaDataSize)
        # For testing, we use a simplified size
        flashmla_metadata_size = 128
        flashmla_metadata_src = torch.randint(
            0, 100, (flashmla_metadata_size,), dtype=torch.int32, device=device
        )
        flashmla_metadata_dst = torch.zeros(
            flashmla_metadata_size, dtype=torch.int32, device=device
        )

    return {
        "src": {
# ...
```
**EN:** This block defines `create_test_metadata`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `create_test_metadata`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 131-153: Function `reference_copy_decode`
```python
def reference_copy_decode(src, dst, max_len):
    """Reference implementation: individual .copy_() for DECODE mode."""
    bs = src["cache_seqlens"].shape[0]
    dst["cache_seqlens"].copy_(src["cache_seqlens"])
    dst["cu_seqlens_k"][1:].copy_(src["cu_seqlens_k"][1:])
    dst["page_table_1"][:, :max_len].copy_(src["page_indices"])
    dst["nsa_cache_seqlens"].copy_(src["nsa_cache_seqlens"])
    dst["nsa_cu_seqlens_k"][1 : bs + 1].copy_(src["nsa_cu_seqlens_k"][1 : bs + 1])

    if src["real_page_table"] is not None:
        rows, cols = src["real_page_table"].shape
        dst["real_page_table"][:rows, :cols].copy_(src["real_page_table"])

    if src["flashmla_num_splits"] is not None:
        flashmla_size = bs + 1
        dst["flashmla_num_splits"][:flashmla_size].copy_(
            src["flashmla_num_splits"][:flashmla_size]
        )

    if src["flashmla_metadata"] is not None:
        dst["flashmla_metadata"].copy_(src["flashmla_metadata"])
```
**EN:** This block defines `reference_copy_decode`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `reference_copy_decode`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 154-181: Function `reference_copy_target_verify`
```python
def reference_copy_target_verify(src, dst, max_seqlen_k, seqlens_expanded_size):
    """Reference implementation: individual .copy_() for TARGET_VERIFY mode."""
    bs = src["cache_seqlens"].shape[0]
    dst["cache_seqlens"].copy_(src["cache_seqlens"])
    dst["cu_seqlens_k"][1:].copy_(src["cu_seqlens_k"][1:])

    rows, cols = src["page_indices"].shape
    dst["page_table_1"][:rows, :cols].copy_(src["page_indices"])
    dst["nsa_seqlens_expanded"][:seqlens_expanded_size].copy_(src["seqlens_expanded"])
    dst["nsa_cache_seqlens"][:seqlens_expanded_size].copy_(src["nsa_cache_seqlens"])
    dst["nsa_cu_seqlens_k"][1 : seqlens_expanded_size + 1].copy_(
        src["nsa_cu_seqlens_k"][1 : seqlens_expanded_size + 1]
    )

    if src["real_page_table"] is not None:
        rows, cols = src["real_page_table"].shape
        dst["real_page_table"][:rows, :cols].copy_(src["real_page_table"])

    if src["flashmla_num_splits"] is not None:
        flashmla_size = seqlens_expanded_size + 1
        dst["flashmla_num_splits"][:flashmla_size].copy_(
            src["flashmla_num_splits"][:flashmla_size]
        )

    if src["flashmla_metadata"] is not None:
        dst["flashmla_metadata"].copy_(src["flashmla_metadata"])
```
**EN:** This block defines `reference_copy_target_verify`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `reference_copy_target_verify`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 182-214: Function `reference_copy_draft_extend`
```python
def reference_copy_draft_extend(src, dst, max_seqlen_k, seqlens_expanded_size):
    """Reference implementation: individual .copy_() for DRAFT_EXTEND mode."""
    bs = src["cache_seqlens"].shape[0]
    dst["cache_seqlens"].copy_(src["cache_seqlens"])
    dst["cu_seqlens_k"][1:].copy_(src["cu_seqlens_k"][1:])

    rows, cols = src["page_indices"].shape
    dst["page_table_1"][:rows, :cols].copy_(src["page_indices"])
    dst["nsa_seqlens_expanded"][:seqlens_expanded_size].copy_(src["seqlens_expanded"])
    dst["nsa_cache_seqlens"][:seqlens_expanded_size].copy_(src["nsa_cache_seqlens"])
    dst["nsa_cu_seqlens_k"][1 : seqlens_expanded_size + 1].copy_(
        src["nsa_cu_seqlens_k"][1 : seqlens_expanded_size + 1]
    )

    if src["real_page_table"] is not None:
        rows, cols = src["real_page_table"].shape
        dst["real_page_table"][:rows, :cols].copy_(src["real_page_table"])

    if src["flashmla_num_splits"] is not None:
        flashmla_size = seqlens_expanded_size + 1
        dst["flashmla_num_splits"][:flashmla_size].copy_(
            src["flashmla_num_splits"][:flashmla_size]
        )

    if src["flashmla_metadata"] is not None:
        dst["flashmla_metadata"].copy_(src["flashmla_metadata"])


# =============================================================================
# Single-Backend Kernel Tests
# =============================================================================
```
**EN:** This block defines `reference_copy_draft_extend`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `reference_copy_draft_extend`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 215-327: Imports and module setup
```python
def test_fused_metadata_copy_dtype_validation():
    """Test that dtype validation rejects non-int32 tensors."""
    if not torch.cuda.is_available():
        pytest.skip("CUDA not available")

    from sglang.jit_kernel.fused_metadata_copy import fused_metadata_copy_cuda

    bs = 2
    max_len = 128
    max_seqlen_k = 256
    seqlens_expanded_size = bs
    device = "cuda"

    # Create tensors with WRONG dtype (int64 instead of int32)
    cache_seqlens_src_wrong = torch.randint(
        1, max_len, (bs,), dtype=torch.int64, device=device
    )
    cu_seqlens_k_src = torch.zeros(bs + 1, dtype=torch.int32, device=device)
    page_indices_src = torch.randint(
        0, 1000, (bs, max_len), dtype=torch.int32, device=device
    )
    nsa_cache_seqlens_src = torch.randint(
        1, max_len, (seqlens_expanded_size,), dtype=torch.int32, device=device
    )
    seqlens_expanded_src = torch.randint(
        1, max_seqlen_k, (seqlens_expanded_size,), dtype=torch.int32, device=device
    )
    nsa_cu_seqlens_k_src = torch.zeros(
        seqlens_expanded_size + 1, dtype=torch.int32, device=device
    )

    # Destination tensors (correct dtype)
    cache_seqlens_dst = torch.zeros(bs, dtype=torch.int32, device=device)
    cu_seqlens_k_dst = torch.zeros(bs + 1, dtype=torch.int32, device=device)
    page_table_1_dst = torch.zeros((bs, max_len + 16), dtype=torch.int32, device=device)
    nsa_cache_seqlens_dst = torch.zeros(
        seqlens_expanded_size, dtype=torch.int32, device=device
    )
    nsa_seqlens_expanded_dst = torch.zeros(
        seqlens_expanded_size, dtype=torch.int32, device=device
    )
    nsa_cu_seqlens_k_dst = torch.zeros(
        seqlens_expanded_size + 1, dtype=torch.int32, device=device
    )

    # Test 1: Wrong dtype for source tensor should raise RuntimeError
    with pytest.raises(RuntimeError, match="must have dtype int32"):
        fused_metadata_copy_cuda(
            cache_seqlens_src_wrong,  # Wrong dtype: int64
            cu_seqlens_k_src,
            page_indices_src,
            nsa_cache_seqlens_src,
            seqlens_expanded_src,
            nsa_cu_seqlens_k_src,
            None,  # real_page_table_src
            None,  # flashmla_num_splits_src
            None,  # flashmla_metadata_src
            cache_seqlens_dst,
            cu_seqlens_k_dst,
            page_table_1_dst,
            nsa_cache_seqlens_dst,
            nsa_seqlens_expanded_dst,
            nsa_cu_seqlens_k_dst,
            None,  # real_page_table_dst
            None,  # flashmla_num_splits_dst
            None,  # flashmla_metadata_dst
            0,  # forward_mode
            bs,
            max_len,
            max_seqlen_k,
            seqlens_expanded_size,
        )

    # Test 2: Wrong dtype for destination tensor should also raise RuntimeError
    cache_seqlens_src = torch.randint(
        1, max_len, (bs,), dtype=torch.int32, device=device
    )
    cache_seqlens_dst_wrong = torch.zeros(bs, dtype=torch.int64, device=device)

    with pytest.raises(RuntimeError, match="must have dtype int32"):
# ...
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 328-428: Imports and module setup
```python
def test_fused_metadata_copy(bs, forward_mode, has_real_page_table, has_flashmla):
    """Test fused metadata copy kernel against reference implementation."""
    if not torch.cuda.is_available():
        pytest.skip("CUDA not available")

    from sglang.jit_kernel.fused_metadata_copy import fused_metadata_copy_cuda

    max_len = 128
    max_seqlen_k = 256
    seqlens_expanded_size = bs if forward_mode == 0 else bs * 2

    # Create test data
    data = create_test_metadata(
        bs=bs,
        max_len=max_len,
        max_seqlen_k=max_seqlen_k,
        seqlens_expanded_size=seqlens_expanded_size,
        has_real_page_table=has_real_page_table,
        has_flashmla=has_flashmla,
    )

    # Create separate destination tensors for reference and fused kernel
    dst_ref = {k: v.clone() if v is not None else None for k, v in data["dst"].items()}
    dst_fused = {
        k: v.clone() if v is not None else None for k, v in data["dst"].items()
    }

    # Run reference implementation
    if forward_mode == 0:  # DECODE
        reference_copy_decode(data["src"], dst_ref, max_len)
    elif forward_mode == 1:  # TARGET_VERIFY
        reference_copy_target_verify(
            data["src"], dst_ref, max_seqlen_k, seqlens_expanded_size
        )
    else:  # DRAFT_EXTEND
        reference_copy_draft_extend(
            data["src"], dst_ref, max_seqlen_k, seqlens_expanded_size
        )

    # Run fused kernel
    fused_metadata_copy_cuda(
        data["src"]["cache_seqlens"],
        data["src"]["cu_seqlens_k"],
        data["src"]["page_indices"],
        data["src"]["nsa_cache_seqlens"],
        data["src"]["seqlens_expanded"],
        data["src"]["nsa_cu_seqlens_k"],
        data["src"]["real_page_table"],
        data["src"]["flashmla_num_splits"],
        data["src"]["flashmla_metadata"],
        dst_fused["cache_seqlens"],
        dst_fused["cu_seqlens_k"],
        dst_fused["page_table_1"],
        dst_fused["nsa_cache_seqlens"],
        dst_fused["nsa_seqlens_expanded"],
        dst_fused["nsa_cu_seqlens_k"],
        dst_fused["real_page_table"],
        dst_fused["flashmla_num_splits"],
        dst_fused["flashmla_metadata"],
        forward_mode,
        bs,
        max_len,
        max_seqlen_k,
        seqlens_expanded_size,
    )

    # Compare results
    assert torch.equal(
        dst_ref["cache_seqlens"], dst_fused["cache_seqlens"]
    ), "cache_seqlens mismatch"
    assert torch.equal(
        dst_ref["cu_seqlens_k"], dst_fused["cu_seqlens_k"]
    ), "cu_seqlens_k mismatch"
    assert torch.equal(
        dst_ref["page_table_1"], dst_fused["page_table_1"]
    ), "page_table_1 mismatch"
    assert torch.equal(
        dst_ref["nsa_cache_seqlens"], dst_fused["nsa_cache_seqlens"]
    ), "nsa_cache_seqlens mismatch"
    assert torch.equal(
# ...
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 429-493: Imports and module setup
```python
def test_fused_metadata_copy_large_batch(bs):
    """Test with larger batch sizes."""
    if not torch.cuda.is_available():
        pytest.skip("CUDA not available")

    from sglang.jit_kernel.fused_metadata_copy import fused_metadata_copy_cuda

    forward_mode = 0  # DECODE
    max_len = 128
    max_seqlen_k = 256
    seqlens_expanded_size = bs

    data = create_test_metadata(
        bs=bs,
        max_len=max_len,
        max_seqlen_k=max_seqlen_k,
        seqlens_expanded_size=seqlens_expanded_size,
        has_real_page_table=True,
        has_flashmla=True,
    )

    dst_ref = {k: v.clone() if v is not None else None for k, v in data["dst"].items()}
    dst_fused = {
        k: v.clone() if v is not None else None for k, v in data["dst"].items()
    }

    reference_copy_decode(data["src"], dst_ref, max_len)

    fused_metadata_copy_cuda(
        data["src"]["cache_seqlens"],
        data["src"]["cu_seqlens_k"],
        data["src"]["page_indices"],
        data["src"]["nsa_cache_seqlens"],
        data["src"]["seqlens_expanded"],
        data["src"]["nsa_cu_seqlens_k"],
        data["src"]["real_page_table"],
        data["src"]["flashmla_num_splits"],
        data["src"]["flashmla_metadata"],
        dst_fused["cache_seqlens"],
        dst_fused["cu_seqlens_k"],
        dst_fused["page_table_1"],
        dst_fused["nsa_cache_seqlens"],
        dst_fused["nsa_seqlens_expanded"],
        dst_fused["nsa_cu_seqlens_k"],
        dst_fused["real_page_table"],
        dst_fused["flashmla_num_splits"],
        dst_fused["flashmla_metadata"],
        forward_mode,
        bs,
        max_len,
        max_seqlen_k,
        seqlens_expanded_size,
    )

    # Verify all tensors match
    for key in dst_ref:
        if dst_ref[key] is not None:
            assert torch.equal(dst_ref[key], dst_fused[key]), f"{key} mismatch"


# =============================================================================
# Multi-Backend Kernel Tests
# =============================================================================
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 494-599: Function `create_test_metadata_multi`
```python
def create_test_metadata_multi(
    bs: int,
    max_len: int,
    seqlens_expanded_size: int,
    has_real_page_table: bool = False,
    has_flashmla: bool = False,
    device: str = "cuda",
):
    """Create test metadata tensors for multi-backend testing."""
    # Source tensors (precomputed metadata)
    cache_seqlens_src = torch.randint(
        1, max_len, (bs,), dtype=torch.int32, device=device
    )
    cu_seqlens_k_src = torch.zeros(bs + 1, dtype=torch.int32, device=device)
    cu_seqlens_k_src[1:] = torch.cumsum(cache_seqlens_src, dim=0)

    page_indices_src = torch.randint(
        0, 1000, (bs, max_len), dtype=torch.int32, device=device
    )
    nsa_cache_seqlens_src = torch.randint(
        1, max_len, (seqlens_expanded_size,), dtype=torch.int32, device=device
    )
    nsa_cu_seqlens_k_src = torch.zeros(
        seqlens_expanded_size + 1, dtype=torch.int32, device=device
    )
    nsa_cu_seqlens_k_src[1:] = torch.cumsum(nsa_cache_seqlens_src, dim=0)

    # Optional tensors
    real_page_table_src = None
    if has_real_page_table:
        real_page_table_cols = max_len // 2
        real_page_table_src = torch.randint(
            0, 1000, (bs, real_page_table_cols), dtype=torch.int32, device=device
        )

    flashmla_num_splits_src = None
    flashmla_metadata_src = None
    if has_flashmla:
        flashmla_num_splits_src = torch.randint(
            1, 10, (seqlens_expanded_size + 1,), dtype=torch.int32, device=device
        )
        flashmla_metadata_size = 128
        flashmla_metadata_src = torch.randint(
            0, 100, (flashmla_metadata_size,), dtype=torch.int32, device=device
        )

    # Create destination tensors for 3 backends
    def create_dst_tensors():
        cache_seqlens_dst = torch.zeros(bs, dtype=torch.int32, device=device)
        cu_seqlens_k_dst = torch.zeros(bs + 1, dtype=torch.int32, device=device)
        page_table_1_dst = torch.zeros(
            (bs, max_len + 16), dtype=torch.int32, device=device
        )
        nsa_cache_seqlens_dst = torch.zeros(
            seqlens_expanded_size, dtype=torch.int32, device=device
        )
        nsa_cu_seqlens_k_dst = torch.zeros(
            seqlens_expanded_size + 1, dtype=torch.int32, device=device
        )

        real_page_table_dst = None
        if has_real_page_table:
            real_page_table_cols = max_len // 2
            real_page_table_dst = torch.zeros(
                (bs, real_page_table_cols + 8), dtype=torch.int32, device=device
            )

        flashmla_num_splits_dst = None
        flashmla_metadata_dst = None
        if has_flashmla:
            flashmla_num_splits_dst = torch.zeros(
                seqlens_expanded_size + 1, dtype=torch.int32, device=device
            )
            flashmla_metadata_size = 128
            flashmla_metadata_dst = torch.zeros(
                flashmla_metadata_size, dtype=torch.int32, device=device
            )

        return {
            "cache_seqlens_int32": cache_seqlens_dst,
# ...
```
**EN:** This block defines `create_test_metadata_multi`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `create_test_metadata_multi`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 600-623: Function `reference_copy_for_loop`
```python
def reference_copy_for_loop(src, dst_list, bs, max_len):
    """Reference implementation: for-loop calling copy for each backend."""
    for dst in dst_list:
        # Simulate what init_forward_metadata_replay_cuda_graph_from_precomputed does
        dst["cache_seqlens_int32"].copy_(src["cache_seqlens"])
        dst["cu_seqlens_k"][1:].copy_(src["cu_seqlens_k"][1:])
        dst["page_table_1"][:, :max_len].copy_(src["page_indices"])
        dst["nsa_cache_seqlens_int32"].copy_(src["nsa_cache_seqlens"])
        dst["nsa_cu_seqlens_k"][1 : bs + 1].copy_(src["nsa_cu_seqlens_k"][1 : bs + 1])

        if src["real_page_table"] is not None:
            rows, cols = src["real_page_table"].shape
            dst["real_page_table"][:rows, :cols].copy_(src["real_page_table"])

        if src["flashmla_num_splits"] is not None:
            flashmla_size = bs + 1
            dst["flashmla_num_splits"][:flashmla_size].copy_(
                src["flashmla_num_splits"][:flashmla_size]
            )

        if src["flashmla_metadata"] is not None:
            dst["flashmla_metadata"].copy_(src["flashmla_metadata"])
```
**EN:** This block defines `reference_copy_for_loop`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `reference_copy_for_loop`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 624-718: Imports and module setup
```python
def test_fused_metadata_copy_multi_dtype_validation():
    """Test that dtype validation rejects non-int32 tensors for multi-backend kernel."""
    if not torch.cuda.is_available():
        pytest.skip("CUDA not available")

    from sglang.jit_kernel.fused_metadata_copy import fused_metadata_copy_multi_cuda

    bs = 2
    max_len = 128
    seqlens_expanded_size = bs
    device = "cuda"

    # Create source tensors - one with WRONG dtype
    cache_seqlens_src_wrong = torch.randint(
        1, max_len, (bs,), dtype=torch.int64, device=device  # Wrong dtype!
    )
    cu_seqlens_k_src = torch.zeros(bs + 1, dtype=torch.int32, device=device)
    page_indices_src = torch.randint(
        0, 1000, (bs, max_len), dtype=torch.int32, device=device
    )
    nsa_cache_seqlens_src = torch.randint(
        1, max_len, (seqlens_expanded_size,), dtype=torch.int32, device=device
    )
    nsa_cu_seqlens_k_src = torch.zeros(
        seqlens_expanded_size + 1, dtype=torch.int32, device=device
    )

    # Create destination tensors for 3 backends (all correct dtype)
    def create_dst():
        return {
            "cache_seqlens": torch.zeros(bs, dtype=torch.int32, device=device),
            "cu_seqlens_k": torch.zeros(bs + 1, dtype=torch.int32, device=device),
            "page_table_1": torch.zeros(
                (bs, max_len + 16), dtype=torch.int32, device=device
            ),
            "nsa_cache_seqlens": torch.zeros(
                seqlens_expanded_size, dtype=torch.int32, device=device
            ),
            "nsa_cu_seqlens_k": torch.zeros(
                seqlens_expanded_size + 1, dtype=torch.int32, device=device
            ),
        }

    dst0 = create_dst()
    dst1 = create_dst()
    dst2 = create_dst()

    # Test: Wrong dtype for source tensor should raise RuntimeError
    with pytest.raises(RuntimeError, match="must have dtype int32"):
        fused_metadata_copy_multi_cuda(
            cache_seqlens_src_wrong,  # Wrong dtype: int64
            cu_seqlens_k_src,
            page_indices_src,
            nsa_cache_seqlens_src,
            nsa_cu_seqlens_k_src,
            None,  # real_page_table_src
            None,  # flashmla_num_splits_src
            None,  # flashmla_metadata_src
            # Backend 0
            dst0["cache_seqlens"],
            dst0["cu_seqlens_k"],
            dst0["page_table_1"],
            dst0["nsa_cache_seqlens"],
            dst0["nsa_cu_seqlens_k"],
            None,
            None,
            None,
            # Backend 1
            dst1["cache_seqlens"],
            dst1["cu_seqlens_k"],
            dst1["page_table_1"],
            dst1["nsa_cache_seqlens"],
            dst1["nsa_cu_seqlens_k"],
            None,
            None,
            None,
            # Backend 2
            dst2["cache_seqlens"],
            dst2["cu_seqlens_k"],
            dst2["page_table_1"],
# ...
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 719-920: Imports and module setup
```python
def test_fused_metadata_copy_multi(bs, has_real_page_table, has_flashmla):
    """Test fused multi-backend metadata copy kernel against for-loop version."""
    if not torch.cuda.is_available():
        pytest.skip("CUDA not available")

    from sglang.jit_kernel.fused_metadata_copy import fused_metadata_copy_multi_cuda

    max_len = 128
    seqlens_expanded_size = bs

    # Create test data
    data = create_test_metadata_multi(
        bs=bs,
        max_len=max_len,
        seqlens_expanded_size=seqlens_expanded_size,
        has_real_page_table=has_real_page_table,
        has_flashmla=has_flashmla,
    )

    # Create separate destination tensors for reference (for-loop) and fused kernel
    dst_ref_0 = {
        k: v.clone() if v is not None else None for k, v in data["dst0"].items()
    }
    dst_ref_1 = {
        k: v.clone() if v is not None else None for k, v in data["dst1"].items()
    }
    dst_ref_2 = {
        k: v.clone() if v is not None else None for k, v in data["dst2"].items()
    }

    dst_fused_0 = {
        k: v.clone() if v is not None else None for k, v in data["dst0"].items()
    }
    dst_fused_1 = {
        k: v.clone() if v is not None else None for k, v in data["dst1"].items()
    }
    dst_fused_2 = {
        k: v.clone() if v is not None else None for k, v in data["dst2"].items()
    }

    # Run reference implementation (for-loop)
    torch.cuda.synchronize()
    loop_start = time.perf_counter()
    reference_copy_for_loop(data["src"], [dst_ref_0, dst_ref_1, dst_ref_2], bs, max_len)
    torch.cuda.synchronize()
    loop_end = time.perf_counter()
    loop_time = loop_end - loop_start

    # Run fused kernel
    torch.cuda.synchronize()
    fused_start = time.perf_counter()
    fused_metadata_copy_multi_cuda(
        # Source tensors
        data["src"]["cache_seqlens"],
        data["src"]["cu_seqlens_k"],
        data["src"]["page_indices"],
        data["src"]["nsa_cache_seqlens"],
        data["src"]["nsa_cu_seqlens_k"],
        data["src"]["real_page_table"],
        data["src"]["flashmla_num_splits"],
        data["src"]["flashmla_metadata"],
        # Destination tensors for backend 0
        dst_fused_0["cache_seqlens_int32"],
        dst_fused_0["cu_seqlens_k"],
        dst_fused_0["page_table_1"],
        dst_fused_0["nsa_cache_seqlens_int32"],
        dst_fused_0["nsa_cu_seqlens_k"],
        dst_fused_0["real_page_table"],
        dst_fused_0["flashmla_num_splits"],
        dst_fused_0["flashmla_metadata"],
        # Destination tensors for backend 1
        dst_fused_1["cache_seqlens_int32"],
        dst_fused_1["cu_seqlens_k"],
        dst_fused_1["page_table_1"],
        dst_fused_1["nsa_cache_seqlens_int32"],
        dst_fused_1["nsa_cu_seqlens_k"],
        dst_fused_1["real_page_table"],
        dst_fused_1["flashmla_num_splits"],
        dst_fused_1["flashmla_metadata"],
        # Destination tensors for backend 2
# ...
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 921-1073: Imports and module setup
```python
def test_fused_metadata_copy_multi_large_batch(bs):
    """Test with larger batch sizes and timing comparison."""
    if not torch.cuda.is_available():
        pytest.skip("CUDA not available")

    from sglang.jit_kernel.fused_metadata_copy import fused_metadata_copy_multi_cuda

    max_len = 128
    seqlens_expanded_size = bs

    data = create_test_metadata_multi(
        bs=bs,
        max_len=max_len,
        seqlens_expanded_size=seqlens_expanded_size,
        has_real_page_table=True,
        has_flashmla=True,
    )

    dst_ref_0 = {
        k: v.clone() if v is not None else None for k, v in data["dst0"].items()
    }
    dst_ref_1 = {
        k: v.clone() if v is not None else None for k, v in data["dst1"].items()
    }
    dst_ref_2 = {
        k: v.clone() if v is not None else None for k, v in data["dst2"].items()
    }

    dst_fused_0 = {
        k: v.clone() if v is not None else None for k, v in data["dst0"].items()
    }
    dst_fused_1 = {
        k: v.clone() if v is not None else None for k, v in data["dst1"].items()
    }
    dst_fused_2 = {
        k: v.clone() if v is not None else None for k, v in data["dst2"].items()
    }

    # Warmup
    for _ in range(5):
        reference_copy_for_loop(
            data["src"], [dst_ref_0, dst_ref_1, dst_ref_2], bs, max_len
        )
        fused_metadata_copy_multi_cuda(
            data["src"]["cache_seqlens"],
            data["src"]["cu_seqlens_k"],
            data["src"]["page_indices"],
            data["src"]["nsa_cache_seqlens"],
            data["src"]["nsa_cu_seqlens_k"],
            data["src"]["real_page_table"],
            data["src"]["flashmla_num_splits"],
            data["src"]["flashmla_metadata"],
            dst_fused_0["cache_seqlens_int32"],
            dst_fused_0["cu_seqlens_k"],
            dst_fused_0["page_table_1"],
            dst_fused_0["nsa_cache_seqlens_int32"],
            dst_fused_0["nsa_cu_seqlens_k"],
            dst_fused_0["real_page_table"],
            dst_fused_0["flashmla_num_splits"],
            dst_fused_0["flashmla_metadata"],
            dst_fused_1["cache_seqlens_int32"],
            dst_fused_1["cu_seqlens_k"],
            dst_fused_1["page_table_1"],
            dst_fused_1["nsa_cache_seqlens_int32"],
            dst_fused_1["nsa_cu_seqlens_k"],
            dst_fused_1["real_page_table"],
            dst_fused_1["flashmla_num_splits"],
            dst_fused_1["flashmla_metadata"],
            dst_fused_2["cache_seqlens_int32"],
            dst_fused_2["cu_seqlens_k"],
            dst_fused_2["page_table_1"],
            dst_fused_2["nsa_cache_seqlens_int32"],
            dst_fused_2["nsa_cu_seqlens_k"],
            dst_fused_2["real_page_table"],
            dst_fused_2["flashmla_num_splits"],
            dst_fused_2["flashmla_metadata"],
            bs,
            max_len,
            seqlens_expanded_size,
        )
# ...
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `time`
- `pytest`
- `torch`
- `sglang.test.ci.ci_register -> register_cuda_ci`
- `sglang.jit_kernel.fused_metadata_copy -> fused_metadata_copy_cuda`
- `sglang.jit_kernel.fused_metadata_copy -> fused_metadata_copy_multi_cuda`
