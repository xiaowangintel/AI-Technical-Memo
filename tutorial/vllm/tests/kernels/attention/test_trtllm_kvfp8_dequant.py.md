# test_trtllm_kvfp8_dequant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_trtllm_kvfp8_dequant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_trtllm_kvfp8_dequant, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_trtllm_kvfp8_dequant 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-8)
```python
"""
Standalone unit tests for trtllm_prefill_attn_kvfp8_dequant.

Tests both contiguous and non-contiguous (cross-layer unified) KV cache
layouts against a pure-PyTorch reference implementation.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 10-13)
```python
import pytest
import torch

from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm.platforms。

### Top-level block starting at line 15 (lines 15-19)
```python
if current_platform.is_rocm():
    pytest.skip(
        "trtllm kvfp8 dequant is not supported on ROCm.",
        allow_module_level=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 21-23)
```python
FP8_DTYPE = current_platform.fp8_dtype()

NUM_BLOCKS = 128
```
**EN:** This block centralizes shared constants and parameter grids, including FP8_DTYPE, NUM_BLOCKS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 FP8_DTYPE、NUM_BLOCKS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `to_float8` (lines 26-34)
```python
def to_float8(x, dtype=None):
    if dtype is None:
        dtype = FP8_DTYPE
    finfo = torch.finfo(dtype)
    min_val, max_val = x.aminmax()
    amax = torch.maximum(min_val.abs(), max_val.abs()).clamp(min=1e-12)
    scale = finfo.max / amax * 0.1
    x_scl_sat = (x * scale).clamp(min=finfo.min, max=finfo.max)
    return x_scl_sat.to(dtype), scale.float().reciprocal()
```
**EN:** This helper function implements the shared logic for to float8. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 to float8 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_contiguous_kv_cache` (lines 37-49)
```python
def make_contiguous_kv_cache(num_blocks, num_kv_heads, block_size, head_size):
    """Create a standard contiguous fp8 KV cache (HND layout)."""
    raw = torch.randn(
        num_blocks,
        2,
        num_kv_heads,
        block_size,
        head_size,
        dtype=torch.bfloat16,
        device="cuda",
    )
    kv_cache, scale = to_float8(raw)
    return kv_cache, scale
```
**EN:** This helper function implements the shared logic for contiguous KV cache. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 contiguous KV cache 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_cross_layer_kv_cache` (lines 52-81)
```python
def make_cross_layer_kv_cache(
    num_blocks,
    num_kv_heads,
    block_size,
    head_size,
    num_layers=4,
):
    """
    Create a non-contiguous per-layer view mimicking cross-layer allocation.

    Physical layout: (num_blocks, 2, num_kv_heads, num_layers, block_size, head_size)
    Returned view:   (num_blocks, 2, num_kv_heads, block_size, head_size)
    with non-contiguous strides on dims 0, 1, 2 (they skip over num_layers).
    """
    raw = torch.randn(
        num_blocks,
        2,
        num_kv_heads,
        num_layers,
        block_size,
        head_size,
        dtype=torch.bfloat16,
        device="cuda",
    )
    fp8_full, scale = to_float8(raw)
    layer_view = fp8_full[:, :, :, 0, :, :]
    assert not layer_view.is_contiguous(), (
        f"Expected non-contiguous view, got strides {layer_view.stride()}"
    )
    return layer_view, scale
```
**EN:** This helper function implements the shared logic for cross layer KV cache. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 cross layer KV cache 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `ref_dequant` (lines 84-109)
```python
def ref_dequant(kv_cache, block_tables, k_scale, v_scale, dequant_dtype):
    """Pure PyTorch reference: gather pages and dequantize fp8 -> dequant_dtype."""
    batch_size, num_pages_per_seq = block_tables.shape
    s = kv_cache.shape
    out = torch.zeros(
        batch_size * num_pages_per_seq + 1,
        s[1],
        s[2],
        s[3],
        s[4],
        dtype=dequant_dtype,
        device=kv_cache.device,
    )
    for b in range(batch_size):
        for p in range(num_pages_per_seq):
            page_idx = block_tables[b, p].item()
            if page_idx <= 0:
                continue
            mock_idx = b * num_pages_per_seq + p + 1
            out[mock_idx, 0] = (kv_cache[page_idx, 0].float() * k_scale.item()).to(
                dequant_dtype
            )
            out[mock_idx, 1] = (kv_cache[page_idx, 1].float() * v_scale.item()).to(
                dequant_dtype
            )
    return out
```
**EN:** This helper acts as a reference implementation for dequant. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 dequant 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_trtllm_kvfp8_dequant` (lines 112-177)
```python
@pytest.mark.parametrize("num_kv_heads", [1, 8])
@pytest.mark.parametrize("head_size", [64, 128])
@pytest.mark.parametrize("block_size", [16, 32])
@pytest.mark.parametrize("batch_size", [1, 4])
@pytest.mark.parametrize("num_pages_per_seq", [3, 8])
@pytest.mark.parametrize("contiguous", [True, False])
@torch.inference_mode()
def test_trtllm_kvfp8_dequant(
    num_kv_heads: int,
    head_size: int,
    block_size: int,
    batch_size: int,
    num_pages_per_seq: int,
    contiguous: bool,
):
    from vllm.v1.attention.backends.flashinfer import (
        trtllm_prefill_attn_kvfp8_dequant,
    )

    torch.set_default_device("cuda")

    if contiguous:
        kv_cache, scale = make_contiguous_kv_cache(
            NUM_BLOCKS,
            num_kv_heads,
            block_size,
            head_size,
        )
    else:
        kv_cache, scale = make_cross_layer_kv_cache(
            NUM_BLOCKS,
            num_kv_heads,
            block_size,
            head_size,
        )

    k_scale = scale.clone()
    v_scale = scale.clone()

    block_tables = torch.randint(
        1,
        NUM_BLOCKS,
        (batch_size, num_pages_per_seq),
        dtype=torch.int32,
    )

    mock_kv_cache, mock_block_table = trtllm_prefill_attn_kvfp8_dequant(
        kv_cache,
        block_tables,
        k_scale,
        v_scale,
        torch.bfloat16,
    )

    ref = ref_dequant(kv_cache, block_tables, k_scale, v_scale, torch.bfloat16)

    expected_bt = torch.arange(
        1,
        batch_size * num_pages_per_seq + 1,
        dtype=torch.int32,
        device="cuda",
    ).reshape(batch_size, num_pages_per_seq)
    torch.testing.assert_close(mock_block_table, expected_bt)

    # Page 0 is padding (never written), compare only pages 1+
    torch.testing.assert_close(mock_kv_cache[1:], ref[1:], atol=1e-3, rtol=1e-3)
```
**EN:** This pytest case verifies trtllm kvfp8 dequant. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_kv_heads, head_size, block_size, batch_size. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 trtllm kvfp8 dequant 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 num_kv_heads、head_size、block_size、batch_size 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较。

### Function `test_block_tables_with_zero_pages` (lines 180-224)
```python
@torch.inference_mode()
def test_block_tables_with_zero_pages():
    """Pages with index <= 0 must be skipped (early return in kernel)."""
    from vllm.v1.attention.backends.flashinfer import (
        trtllm_prefill_attn_kvfp8_dequant,
    )

    torch.set_default_device("cuda")
    num_kv_heads, block_size, head_size = 8, 16, 64

    kv_cache, scale = make_contiguous_kv_cache(
        NUM_BLOCKS,
        num_kv_heads,
        block_size,
        head_size,
    )
    k_scale = v_scale = scale.clone()

    # Mix of valid pages and zeros (padding)
    block_tables = torch.tensor(
        [[5, 0, 10], [0, 0, 0], [3, 7, 0]],
        dtype=torch.int32,
        device="cuda",
    )

    mock_kv_cache, _ = trtllm_prefill_attn_kvfp8_dequant(
        kv_cache,
        block_tables,
        k_scale,
        v_scale,
        torch.bfloat16,
    )
    ref = ref_dequant(kv_cache, block_tables, k_scale, v_scale, torch.bfloat16)

    # Only compare pages that were actually written (non-zero page indices)
    for b in range(block_tables.shape[0]):
        for p in range(block_tables.shape[1]):
            if block_tables[b, p].item() > 0:
                idx = b * block_tables.shape[1] + p + 1
                torch.testing.assert_close(
                    mock_kv_cache[idx],
                    ref[idx],
                    atol=1e-3,
                    rtol=1e-3,
                )
```
**EN:** This pytest case verifies block tables with zero pages. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 block tables with zero pages 的行为。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_all_zero_block_tables` (lines 227-256)
```python
@torch.inference_mode()
def test_all_zero_block_tables():
    """All-zero block_tables: kernel should write nothing."""
    from vllm.v1.attention.backends.flashinfer import (
        trtllm_prefill_attn_kvfp8_dequant,
    )

    torch.set_default_device("cuda")
    num_kv_heads, block_size, head_size = 4, 16, 64

    kv_cache, scale = make_contiguous_kv_cache(
        NUM_BLOCKS,
        num_kv_heads,
        block_size,
        head_size,
    )
    k_scale = v_scale = scale.clone()

    block_tables = torch.zeros(2, 4, dtype=torch.int32, device="cuda")

    # Should not crash even though no pages are valid
    mock_kv_cache, mock_block_table = trtllm_prefill_attn_kvfp8_dequant(
        kv_cache,
        block_tables,
        k_scale,
        v_scale,
        torch.bfloat16,
    )
    assert mock_kv_cache.shape[0] == 2 * 4 + 1
    assert mock_block_table.shape == (2, 4)
```
**EN:** This pytest case verifies all zero block tables. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 all zero block tables 的行为。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_different_k_v_scales` (lines 259-289)
```python
@torch.inference_mode()
def test_different_k_v_scales():
    """Verify K and V are dequantized with independent scales."""
    from vllm.v1.attention.backends.flashinfer import (
        trtllm_prefill_attn_kvfp8_dequant,
    )

    torch.set_default_device("cuda")
    num_kv_heads, block_size, head_size = 8, 16, 64

    kv_cache, _ = make_contiguous_kv_cache(
        NUM_BLOCKS,
        num_kv_heads,
        block_size,
        head_size,
    )
    k_scale = torch.tensor([0.5], dtype=torch.float32, device="cuda")
    v_scale = torch.tensor([2.0], dtype=torch.float32, device="cuda")

    block_tables = torch.tensor([[1, 2]], dtype=torch.int32, device="cuda")

    mock_kv_cache, _ = trtllm_prefill_attn_kvfp8_dequant(
        kv_cache,
        block_tables,
        k_scale,
        v_scale,
        torch.bfloat16,
    )
    ref = ref_dequant(kv_cache, block_tables, k_scale, v_scale, torch.bfloat16)

    torch.testing.assert_close(mock_kv_cache[1:], ref[1:], atol=1e-3, rtol=1e-3)
```
**EN:** This pytest case verifies different k v scales. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 different k v scales 的行为。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_single_page_per_seq` (lines 292-321)
```python
@torch.inference_mode()
def test_single_page_per_seq():
    """Minimum grid dim 1 = 1 page per sequence."""
    from vllm.v1.attention.backends.flashinfer import (
        trtllm_prefill_attn_kvfp8_dequant,
    )

    torch.set_default_device("cuda")
    num_kv_heads, block_size, head_size = 8, 16, 128

    kv_cache, scale = make_contiguous_kv_cache(
        NUM_BLOCKS,
        num_kv_heads,
        block_size,
        head_size,
    )
    k_scale = v_scale = scale.clone()

    block_tables = torch.tensor([[5], [10], [20]], dtype=torch.int32, device="cuda")

    mock_kv_cache, _ = trtllm_prefill_attn_kvfp8_dequant(
        kv_cache,
        block_tables,
        k_scale,
        v_scale,
        torch.bfloat16,
    )
    ref = ref_dequant(kv_cache, block_tables, k_scale, v_scale, torch.bfloat16)

    torch.testing.assert_close(mock_kv_cache[1:], ref[1:], atol=1e-3, rtol=1e-3)
```
**EN:** This pytest case verifies single page per seq. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 single page per seq 的行为。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_large_page_indices` (lines 324-359)
```python
@torch.inference_mode()
def test_large_page_indices():
    """Page indices near the top of the buffer stress offset arithmetic."""
    from vllm.v1.attention.backends.flashinfer import (
        trtllm_prefill_attn_kvfp8_dequant,
    )

    torch.set_default_device("cuda")
    num_kv_heads, block_size, head_size = 8, 16, 128
    large_num_blocks = 32768

    kv_cache, scale = make_contiguous_kv_cache(
        large_num_blocks,
        num_kv_heads,
        block_size,
        head_size,
    )
    k_scale = v_scale = scale.clone()

    # Use page indices near the top of the buffer
    block_tables = torch.tensor(
        [[large_num_blocks - 1, large_num_blocks - 2, 1]],
        dtype=torch.int32,
        device="cuda",
    )

    mock_kv_cache, _ = trtllm_prefill_attn_kvfp8_dequant(
        kv_cache,
        block_tables,
        k_scale,
        v_scale,
        torch.bfloat16,
    )
    ref = ref_dequant(kv_cache, block_tables, k_scale, v_scale, torch.bfloat16)

    torch.testing.assert_close(mock_kv_cache[1:], ref[1:], atol=1e-3, rtol=1e-3)
```
**EN:** This pytest case verifies large page indices. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 large page indices 的行为。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_large_block_size` (lines 362-397)
```python
@torch.inference_mode()
def test_large_block_size():
    """block_size=64 -> HEAD_STRIDE=8192, large tl.arange per thread block."""
    from vllm.v1.attention.backends.flashinfer import (
        trtllm_prefill_attn_kvfp8_dequant,
    )

    torch.set_default_device("cuda")
    num_kv_heads, block_size, head_size = 4, 64, 128

    kv_cache, scale = make_contiguous_kv_cache(
        NUM_BLOCKS,
        num_kv_heads,
        block_size,
        head_size,
    )
    k_scale = v_scale = scale.clone()

    block_tables = torch.randint(
        1,
        NUM_BLOCKS,
        (2, 4),
        dtype=torch.int32,
        device="cuda",
    )

    mock_kv_cache, _ = trtllm_prefill_attn_kvfp8_dequant(
        kv_cache,
        block_tables,
        k_scale,
        v_scale,
        torch.bfloat16,
    )
    ref = ref_dequant(kv_cache, block_tables, k_scale, v_scale, torch.bfloat16)

    torch.testing.assert_close(mock_kv_cache[1:], ref[1:], atol=1e-3, rtol=1e-3)
```
**EN:** This pytest case verifies large block size. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 large block size 的行为。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_cross_layer_many_layers` (lines 400-440)
```python
@torch.inference_mode()
def test_cross_layer_many_layers():
    """
    Non-contiguous with 36 layers -- matches real gpt-oss-120b.
    Strides are far from contiguous (factor of 36 in the gaps).
    """
    from vllm.v1.attention.backends.flashinfer import (
        trtllm_prefill_attn_kvfp8_dequant,
    )

    torch.set_default_device("cuda")
    num_kv_heads, block_size, head_size = 8, 16, 64
    num_layers = 36

    kv_cache, scale = make_cross_layer_kv_cache(
        NUM_BLOCKS,
        num_kv_heads,
        block_size,
        head_size,
        num_layers=num_layers,
    )
    k_scale = v_scale = scale.clone()

    block_tables = torch.randint(
        1,
        NUM_BLOCKS,
        (4, 6),
        dtype=torch.int32,
        device="cuda",
    )

    mock_kv_cache, _ = trtllm_prefill_attn_kvfp8_dequant(
        kv_cache,
        block_tables,
        k_scale,
        v_scale,
        torch.bfloat16,
    )
    ref = ref_dequant(kv_cache, block_tables, k_scale, v_scale, torch.bfloat16)

    torch.testing.assert_close(mock_kv_cache[1:], ref[1:], atol=1e-3, rtol=1e-3)
```
**EN:** This pytest case verifies cross layer many layers. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 cross layer many layers 的行为。 数值结果会在显式容差下与参考结果进行比较。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.platforms -> current_platform`
