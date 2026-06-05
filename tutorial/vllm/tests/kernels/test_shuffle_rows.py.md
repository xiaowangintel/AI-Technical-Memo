# test_shuffle_rows.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/test_shuffle_rows.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / test_shuffle_rows, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / test_shuffle_rows 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-6)
```python
"""Tests for the shuffle_rows function

Run `pytest tests/kernels/test_shuffle_rows.py`.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 8-12)
```python
import pytest
import torch

from vllm._custom_ops import shuffle_rows
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm._custom_ops, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm._custom_ops、vllm.platforms。

### Function `test_shuffle_rows_basic` (lines 15-39)
```python
@pytest.mark.parametrize("num_tokens", [1, 16, 64, 128, 256, 512, 1024])
@pytest.mark.parametrize("hidden_size", [128, 256, 512, 1024, 2048, 4096])
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16, torch.float32])
def test_shuffle_rows_basic(num_tokens: int, hidden_size: int, dtype: torch.dtype):
    """Test basic functionality of shuffle_rows with various tensor sizes and
    dtypes."""
    if not current_platform.is_cuda():
        pytest.skip("shuffle_rows requires CUDA")

    # Create input tensor
    input_tensor = torch.randn(num_tokens, hidden_size, device="cuda", dtype=dtype)

    # Create a simple permutation map (identity mapping)
    dst2src_map = torch.arange(num_tokens, device="cuda", dtype=torch.int32)

    # Test shuffle_rows
    output = shuffle_rows(input_tensor, dst2src_map)

    # With identity mapping, output should be identical to input
    torch.testing.assert_close(output, input_tensor, atol=0, rtol=0)

    # Check output shape
    assert output.shape == (num_tokens, hidden_size)
    assert output.dtype == dtype
    assert output.device == input_tensor.device
```
**EN:** This pytest case verifies shuffle rows basic. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, dtype. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 shuffle rows basic 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、dtype 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_shuffle_rows_permutation` (lines 42-68)
```python
@pytest.mark.parametrize("num_tokens", [16, 64, 128])
@pytest.mark.parametrize("hidden_size", [128, 512, 1024])
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
def test_shuffle_rows_permutation(
    num_tokens: int, hidden_size: int, dtype: torch.dtype
):
    """Test shuffle_rows with actual permutation."""
    if not current_platform.is_cuda():
        pytest.skip("shuffle_rows requires CUDA")

    # Create input tensor
    input_tensor = torch.randn(num_tokens, hidden_size, device="cuda", dtype=dtype)

    # Create a reverse permutation map
    dst2src_map = torch.arange(num_tokens - 1, -1, -1, device="cuda", dtype=torch.int32)

    # Test shuffle_rows
    output = shuffle_rows(input_tensor, dst2src_map)

    # Check that the output is the reverse of the input
    expected_output = torch.flip(input_tensor, dims=[0])
    torch.testing.assert_close(output, expected_output, atol=1e-6, rtol=1e-5)

    # Check output shape and properties
    assert output.shape == (num_tokens, hidden_size)
    assert output.dtype == dtype
    assert output.device == input_tensor.device
```
**EN:** This pytest case verifies shuffle rows permutation. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, dtype. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 shuffle rows permutation 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、dtype 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_shuffle_rows_expansion` (lines 71-103)
```python
@pytest.mark.parametrize("num_tokens", [32, 64])
@pytest.mark.parametrize("hidden_size", [256, 512])
def test_shuffle_rows_expansion(num_tokens: int, hidden_size: int):
    """Test shuffle_rows with expansion (more output tokens than input
    tokens)."""
    if not current_platform.is_cuda():
        pytest.skip("shuffle_rows requires CUDA")

    dtype = torch.float16

    # Create input tensor
    input_tensor = torch.randn(num_tokens, hidden_size, device="cuda", dtype=dtype)

    # Create a mapping that duplicates some tokens (expansion)
    expanded_size = num_tokens * 2
    dst2src_map = torch.randint(
        0, num_tokens, (expanded_size,), device="cuda", dtype=torch.int32
    )

    # Test shuffle_rows
    output = shuffle_rows(input_tensor, dst2src_map)

    # Check output shape
    assert output.shape == (expanded_size, hidden_size)
    assert output.dtype == dtype
    assert output.device == input_tensor.device

    # Verify that each output row matches the corresponding input row
    for i in range(expanded_size):
        src_idx = dst2src_map[i].item()
        torch.testing.assert_close(
            output[i], input_tensor[src_idx], atol=1e-6, rtol=1e-5
        )
```
**EN:** This pytest case verifies shuffle rows expansion. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 shuffle rows expansion 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_shuffle_rows_random_permutation` (lines 106-137)
```python
@pytest.mark.parametrize("num_tokens", [16, 64])
@pytest.mark.parametrize("hidden_size", [128, 512])
def test_shuffle_rows_random_permutation(num_tokens: int, hidden_size: int):
    """Test shuffle_rows with random permutation."""
    if not current_platform.is_cuda():
        pytest.skip("shuffle_rows requires CUDA")

    dtype = torch.float16

    # Set seed for reproducibility
    torch.manual_seed(42)

    # Create input tensor
    input_tensor = torch.randn(num_tokens, hidden_size, device="cuda", dtype=dtype)

    # Create a random permutation map
    dst2src_map = torch.randperm(num_tokens, device="cuda", dtype=torch.int32)

    # Test shuffle_rows
    output = shuffle_rows(input_tensor, dst2src_map)

    # Check output shape and properties
    assert output.shape == (num_tokens, hidden_size)
    assert output.dtype == dtype
    assert output.device == input_tensor.device

    # Verify that each output row matches the corresponding input row
    for i in range(num_tokens):
        src_idx = dst2src_map[i].item()
        torch.testing.assert_close(
            output[i], input_tensor[src_idx], atol=1e-6, rtol=1e-5
        )
```
**EN:** This pytest case verifies shuffle rows random permutation. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 shuffle rows random permutation 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_shuffle_rows_edge_cases` (lines 140-157)
```python
def test_shuffle_rows_edge_cases():
    """Test shuffle_rows with edge cases."""
    if not current_platform.is_cuda():
        pytest.skip("shuffle_rows requires CUDA")

    dtype = torch.float16

    # Test with single token
    input_tensor = torch.randn(1, 128, device="cuda", dtype=dtype)
    dst2src_map = torch.tensor([0], device="cuda", dtype=torch.int32)
    output = shuffle_rows(input_tensor, dst2src_map)
    torch.testing.assert_close(output, input_tensor, atol=0, rtol=0)

    # Test with single feature dimension
    input_tensor = torch.randn(16, 1, device="cuda", dtype=dtype)
    dst2src_map = torch.arange(16, device="cuda", dtype=torch.int32)
    output = shuffle_rows(input_tensor, dst2src_map)
    torch.testing.assert_close(output, input_tensor, atol=0, rtol=0)
```
**EN:** This pytest case verifies shuffle rows edge cases. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 shuffle rows edge cases 的行为。 不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_shuffle_rows_moe_like_scenario` (lines 160-197)
```python
def test_shuffle_rows_moe_like_scenario():
    """Test shuffle_rows in a scenario similar to MoE usage."""
    if not current_platform.is_cuda():
        pytest.skip("shuffle_rows requires CUDA")

    dtype = torch.float16
    batch_size = 32
    hidden_size = 1024
    topk = 2

    # Simulate input tokens
    input_tensor = torch.randn(batch_size, hidden_size, device="cuda", dtype=dtype)

    # Simulate expert assignment (each token goes to topk experts)
    # This creates a mapping where tokens are duplicated for multiple experts
    total_tokens = batch_size * topk
    dst2src_map = torch.zeros(total_tokens, device="cuda", dtype=torch.int32)

    # Fill the mapping to simulate MoE token distribution
    for i in range(batch_size):
        for k in range(topk):
            dst2src_map[i * topk + k] = i

    # Test shuffle_rows
    output = shuffle_rows(input_tensor, dst2src_map)

    # Check output shape
    assert output.shape == (total_tokens, hidden_size)
    assert output.dtype == dtype
    assert output.device == input_tensor.device

    # Verify that tokens are correctly duplicated
    for i in range(batch_size):
        for k in range(topk):
            output_idx = i * topk + k
            torch.testing.assert_close(
                output[output_idx], input_tensor[i], atol=1e-6, rtol=1e-5
            )
```
**EN:** This pytest case verifies shuffle rows MoE like scenario. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 shuffle rows MoE like scenario 的行为。 不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_shuffle_rows_dtype_consistency` (lines 200-219)
```python
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16, torch.float32])
def test_shuffle_rows_dtype_consistency(dtype: torch.dtype):
    """Test that shuffle_rows preserves dtype correctly."""
    if not current_platform.is_cuda():
        pytest.skip("shuffle_rows requires CUDA")

    num_tokens = 64
    hidden_size = 512

    # Create input tensor with specific dtype
    input_tensor = torch.randn(num_tokens, hidden_size, device="cuda", dtype=dtype)
    dst2src_map = torch.arange(num_tokens, device="cuda", dtype=torch.int32)

    # Test shuffle_rows
    output = shuffle_rows(input_tensor, dst2src_map)

    # Verify dtype is preserved
    assert output.dtype == dtype
    assert output.device == input_tensor.device
    torch.testing.assert_close(output, input_tensor, atol=1e-6, rtol=1e-5)
```
**EN:** This pytest case verifies shuffle rows dtype consistency. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as dtype. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 shuffle rows dtype consistency 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 dtype 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_shuffle_rows_device_consistency` (lines 222-240)
```python
def test_shuffle_rows_device_consistency():
    """Test that shuffle_rows maintains device consistency."""
    if not current_platform.is_cuda():
        pytest.skip("shuffle_rows requires CUDA")

    num_tokens = 32
    hidden_size = 256
    dtype = torch.float16

    # Create input tensor on CUDA
    input_tensor = torch.randn(num_tokens, hidden_size, device="cuda", dtype=dtype)
    dst2src_map = torch.arange(num_tokens, device="cuda", dtype=torch.int32)

    # Test shuffle_rows
    output = shuffle_rows(input_tensor, dst2src_map)

    # Verify device is maintained
    assert output.device == input_tensor.device
    assert output.device.type == "cuda"
```
**EN:** This pytest case verifies shuffle rows device consistency. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 shuffle rows device consistency 的行为。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_shuffle_rows_contiguous_output` (lines 243-260)
```python
def test_shuffle_rows_contiguous_output():
    """Test that shuffle_rows produces contiguous output."""
    if not current_platform.is_cuda():
        pytest.skip("shuffle_rows requires CUDA")

    num_tokens = 64
    hidden_size = 512
    dtype = torch.float16

    # Create input tensor
    input_tensor = torch.randn(num_tokens, hidden_size, device="cuda", dtype=dtype)
    dst2src_map = torch.arange(num_tokens, device="cuda", dtype=torch.int32)

    # Test shuffle_rows
    output = shuffle_rows(input_tensor, dst2src_map)

    # Verify output is contiguous
    assert output.is_contiguous()
```
**EN:** This pytest case verifies shuffle rows contiguous output. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 shuffle rows contiguous output 的行为。 不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm._custom_ops -> shuffle_rows`
- `vllm.platforms -> current_platform`
