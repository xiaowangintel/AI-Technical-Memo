# test_pack_unpack_triton.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/attention/test_pack_unpack_triton.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / attention / test_pack_unpack_triton, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / attention / test_pack_unpack_triton 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-7)
```python
import torch
from torch.testing import assert_close

from vllm.v1.attention.ops.common import pack_seq_triton, unpack_seq_triton
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as torch, torch.testing; and vLLM components like vllm.v1.attention.ops.common.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 torch、torch.testing；vLLM 内部组件，例如 vllm.v1.attention.ops.common。

### Function `test_pack_seq_basic_fp8` (lines 10-45)
```python
def test_pack_seq_basic_fp8():
    """Test basic functionality of pack_seq_triton with fp8 and 3D tensors."""
    device = "cuda"
    dtype = torch.float8_e4m3fn

    # Test cases with 3D tensors (N, H, D)
    test_cases = [
        (6, 8, 4, 2, [3, 3]),  # (6, 8, 4) -> (2, 3, 8, 4)
        (10, 4, 8, 3, [2, 4, 4]),  # (10, 4, 8) -> (3, 4, 4, 8)
        (20, 16, 32, 4, [5, 5, 5, 5]),  # (20, 16, 32) -> (4, 5, 16, 32)
    ]

    for N, H, D, B, lengths_list in test_cases:
        # Create input tensor with small values for fp8
        x = torch.randn(N, H, D, dtype=torch.float32, device=device) * 0.1
        x = x.to(dtype=dtype)
        lengths = torch.tensor(lengths_list, device=device)

        # Pack the data
        packed = pack_seq_triton(x, lengths)

        # Check output shape and properties
        expected_shape = (B, max(lengths_list), H, D)
        assert packed.shape == expected_shape
        assert packed.dtype == dtype
        assert packed.device == x.device

        # Check that valid data is preserved (within fp8 precision)
        for b in range(B):
            start_idx = sum(lengths_list[:b])
            seq_len = lengths_list[b]

            expected_data = x[start_idx : start_idx + seq_len].to(torch.float32)
            actual_data = packed[b, :seq_len].to(torch.float32)

            assert_close(actual_data, expected_data, rtol=1e-1, atol=1e-2)
```
**EN:** This pytest case verifies pack seq basic FP8. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 pack seq basic FP8 的行为。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_pack_seq_custom_padding_fp8` (lines 48-76)
```python
def test_pack_seq_custom_padding_fp8():
    """Test pack_seq_triton with custom padding values for fp8."""
    device = "cuda"
    dtype = torch.float8_e4m3fn
    N, H, D, B = 20, 8, 16, 2
    lengths = torch.tensor([10, 10], device=device)

    x = torch.randn(N, H, D, dtype=torch.float32, device=device) * 0.1
    x = x.to(dtype=dtype)

    # Test with different padding values
    for pad_value in [-100.0, -10.0, 0.0, 10.0, 100.0]:
        result = pack_seq_triton(x, lengths, pad_value=pad_value)

        # Check valid data
        for b in range(B):
            start_idx = b * 10
            expected_data = x[start_idx : start_idx + 10].to(torch.float32)
            actual_data = result[b, :10].to(torch.float32)
            assert_close(actual_data, expected_data, rtol=1e-1, atol=1e-2)

        # Check padding (fp8 has limited range, so check for large values)
        padded_data = result[:, 10:].to(torch.float32)
        if pad_value < 0:
            assert torch.all(padded_data < -50)  # Large negative values
        elif pad_value > 0:
            assert torch.all(padded_data > 50)  # Large positive values
        else:
            assert torch.allclose(padded_data, torch.zeros_like(padded_data), atol=1e-2)
```
**EN:** This pytest case verifies pack seq custom padding FP8. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 pack seq custom padding FP8 的行为。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_pack_seq_default_negative_inf_padding_fp8` (lines 79-95)
```python
def test_pack_seq_default_negative_inf_padding_fp8():
    """Test that pack_seq_triton uses -inf padding by default for fp8."""
    device = "cuda"
    dtype = torch.float8_e4m3fn
    # B = 2
    N, H, D = 20, 8, 16
    lengths = torch.tensor([10, 10], device=device)

    x = torch.randn(N, H, D, dtype=torch.float32, device=device) * 0.1
    x = x.to(dtype=dtype)
    result = pack_seq_triton(x, lengths)

    # Check that padding is large negative values (fp8 representation of -inf)
    padded_data = result[:, 10:].to(torch.float32)
    assert torch.all(
        padded_data < -100
    )  # fp8 -inf is represented as large negative number
```
**EN:** This pytest case verifies pack seq default negative inf padding FP8. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 pack seq default negative inf padding FP8 的行为。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_pack_seq_edge_cases_fp8` (lines 98-122)
```python
def test_pack_seq_edge_cases_fp8():
    """Test pack_seq_triton with edge cases for fp8."""
    device = "cuda"
    dtype = torch.float8_e4m3fn

    # Test with single batch element
    x = torch.randn(10, 8, 16, dtype=torch.float32, device=device) * 0.1
    x = x.to(dtype=dtype)
    lengths = torch.tensor([10], device=device)
    result = pack_seq_triton(x, lengths)
    assert result.shape == (1, 10, 8, 16)

    # Test with very short sequences
    x = torch.randn(20, 4, 8, dtype=torch.float32, device=device) * 0.1
    x = x.to(dtype=dtype)
    lengths = torch.tensor([1, 1, 1], device=device)
    result = pack_seq_triton(x, lengths)
    assert result.shape == (3, 1, 4, 8)

    # Test with different sequence lengths
    x = torch.randn(15, 8, 16, dtype=torch.float32, device=device) * 0.1
    x = x.to(dtype=dtype)
    lengths = torch.tensor([5, 7, 3], device=device)
    result = pack_seq_triton(x, lengths)
    assert result.shape == (3, 7, 8, 16)
```
**EN:** This pytest case verifies pack seq edge cases FP8. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 pack seq edge cases FP8 的行为。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_pack_seq_different_block_sizes_fp8` (lines 125-146)
```python
def test_pack_seq_different_block_sizes_fp8():
    """Test pack_seq_triton with different block sizes for fp8."""
    device = "cuda"
    dtype = torch.float8_e4m3fn
    N, H, D, B = 100, 16, 32, 4
    lengths = torch.tensor([25, 25, 25, 25], device=device)

    x = torch.randn(N, H, D, dtype=torch.float32, device=device) * 0.1
    x = x.to(dtype=dtype)

    # Test different block sizes
    for block_t, block_d in [(32, 32), (64, 64), (128, 128)]:
        result = pack_seq_triton(x, lengths, block_t=block_t, block_d=block_d)

        assert result.shape == (B, 25, H, D)

        # Check that valid data is preserved (within fp8 precision)
        for b in range(B):
            start_idx = b * 25
            expected_data = x[start_idx : start_idx + 25].to(torch.float32)
            actual_data = result[b, :25].to(torch.float32)
            assert_close(actual_data, expected_data, rtol=1e-1, atol=1e-2)
```
**EN:** This pytest case verifies pack seq different block sizes FP8. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 pack seq different block sizes FP8 的行为。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_pack_seq_shape_consistency` (lines 149-164)
```python
def test_pack_seq_shape_consistency():
    """Test that pack_seq_triton maintains shape consistency."""
    device = "cuda"
    dtype = torch.float8_e4m3fn
    N, H, D, B = 20, 8, 16, 2
    lengths = torch.tensor([10, 10], device=device)

    x = torch.randn(N, H, D, dtype=torch.float32, device=device) * 0.1
    x = x.to(dtype=dtype)

    result = pack_seq_triton(x, lengths)

    # Check shape consistency
    assert result.shape[0] == B  # Batch dimension
    assert result.shape[1] == lengths.max().item()  # Max sequence length
    assert result.shape[2:] == x.shape[1:]  # Feature dimensions preserved
```
**EN:** This pytest case verifies pack seq shape consistency. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 pack seq shape consistency 的行为。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_pack_unpack_roundtrip_fp8` (lines 167-200)
```python
def test_pack_unpack_roundtrip_fp8():
    """Test that pack -> unpack gives us back the original data for fp8."""
    device = "cuda"
    dtype = torch.float8_e4m3fn

    # Test cases with 3D tensors
    test_cases = [
        (6, 8, 4, 2, [3, 3]),
        (10, 4, 8, 3, [2, 4, 4]),
        (20, 16, 32, 4, [5, 5, 5, 5]),
        (15, 8, 16, 3, [7, 5, 3]),
    ]

    for N, H, D, B, lengths_list in test_cases:
        # Create input tensor with small values for fp8
        x = torch.randn(N, H, D, dtype=torch.float32, device=device) * 0.1
        x = x.to(dtype=dtype)
        lengths = torch.tensor(lengths_list, device=device)

        # Pack the data
        packed = pack_seq_triton(x, lengths)

        # Unpack the data
        unpacked = unpack_seq_triton(packed, lengths)

        # Check that we get back the original data (within fp8 precision)
        assert unpacked.shape == x.shape
        x_f32 = x.to(torch.float32)
        unpacked_f32 = unpacked.to(torch.float32)
        assert_close(x_f32, unpacked_f32, rtol=1e-3, atol=1e-3)

        # Unpack without explicit start locations (computed in kernel)
        unpacked_with_loc = unpack_seq_triton(packed, lengths)
        assert_close(x_f32, unpacked_with_loc.to(torch.float32), rtol=1e-3, atol=1e-2)
```
**EN:** This pytest case verifies pack unpack roundtrip FP8. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 pack unpack roundtrip FP8 的行为。 数值结果会在显式容差下与参考结果进行比较。

### Function `test_unpack_seq_triton_edge_cases_fp8` (lines 203-234)
```python
def test_unpack_seq_triton_edge_cases_fp8():
    """Test unpack function with edge cases for fp8."""
    device = "cuda"
    dtype = torch.float8_e4m3fn

    # Test with single batch element
    x = torch.randn(10, 8, 16, dtype=torch.float32, device=device) * 0.1
    x = x.to(dtype=dtype)
    lengths = torch.tensor([10], device=device)
    packed = pack_seq_triton(x, lengths)
    unpacked = unpack_seq_triton(packed, lengths)
    assert unpacked.shape == x.shape
    assert_close(x.to(torch.float32), unpacked.to(torch.float32), rtol=1e-1, atol=1e-2)

    # Test with very short sequences
    x = torch.randn(20, 4, 8, dtype=torch.float32, device=device) * 0.1
    x = x.to(dtype=dtype)
    lengths = torch.tensor([1, 1, 1], device=device)
    packed = pack_seq_triton(x, lengths)
    unpacked = unpack_seq_triton(packed, lengths)
    # Only compare the first 3 elements that were actually packed
    assert_close(
        x[:3].to(torch.float32), unpacked.to(torch.float32), rtol=1e-1, atol=1e-2
    )

    x = torch.randn(15, 8, 16, dtype=torch.float32, device=device) * 0.1
    x = x.to(dtype=dtype)
    lengths = torch.tensor([5, 7, 3], device=device)
    packed = pack_seq_triton(x, lengths)
    unpacked = unpack_seq_triton(packed, lengths)
    assert unpacked.shape == x.shape
    assert_close(x.to(torch.float32), unpacked.to(torch.float32), rtol=1e-1, atol=1e-2)
```
**EN:** This pytest case verifies unpack seq triton edge cases FP8. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 unpack seq triton edge cases FP8 的行为。 数值结果会在显式容差下与参考结果进行比较。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `torch`
- `torch.testing -> assert_close`
- `vllm.v1.attention.ops.common -> pack_seq_triton, unpack_seq_triton`
