# test_rms_norm_batch_invariant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/determinism/test_rms_norm_batch_invariant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Test batch-invariant RMS normalization against standard implementations. / 该文件的文档字符串表明其用途：`test batch-invariant rms normalization against standard implementations`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-8)
```python
"""
Test batch-invariant RMS normalization against standard implementations.

This test compares the Triton-based batch-invariant RMS norm implementation
with the standard CUDA-based implementation to ensure numerical accuracy.
"""
```
**EN:** Module docstring that declares the scope of the file: Test batch-invariant RMS normalization against standard implementations.
**CN:** 模块文档字符串直接说明了文件范围：`test batch-invariant rms normalization against standard implementations`。

### Imports and setup / 导入与设置 (lines 10-16)
```python
import pytest
import torch
from utils import skip_unsupported

from vllm.model_executor.layers.batch_invariant import rms_norm as triton_rms_norm
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.platforms import current_platform
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, utils`. vLLM modules under test include `vllm.model_executor.layers.batch_invariant, vllm.model_executor.layers.layernorm, vllm.platforms, vllm._custom_ops`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, utils`。 被测试的 vLLM 模块包括 `vllm.model_executor.layers.batch_invariant, vllm.model_executor.layers.layernorm, vllm.platforms, vllm._custom_ops`。

### Module state / 模块级状态 (line 18)
```python
DEVICE_TYPE = current_platform.device_type
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `DEVICE_TYPE`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`DEVICE_TYPE`。

### test_rms_norm_batch_invariant_vs_standard (lines 26-71)
```python
def test_rms_norm_batch_invariant_vs_standard(
    default_vllm_config,
    batch_size: int,
    hidden_size: int,
    dtype: torch.dtype,
    eps: float,
):
    """
    Compare batch-invariant Triton RMS norm against standard CUDA implementation.

    Tests that the Triton-based batch-invariant RMS norm produces numerically
    equivalent results to the standard CUDA implementation across various
    configurations.
    device = torch.device(DEVICE_TYPE)
    # Create test input and weight
    torch.manual_seed(42)
    # ... excerpt omitted for brevity ...
        rtol=rtol,
        atol=atol,
        msg=f"RMS norm mismatch for batch_size={batch_size}, "
        f"hidden_size={hidden_size}, "
        f"dtype={dtype}, eps={eps}",
    )
```
**EN:** Parameterized test covering `rms norm batch invariant vs standard`. Parameter axes: `batch_size, hidden_size, dtype, eps`. Inputs/fixtures: `default_vllm_config, batch_size, hidden_size, dtype, eps`. It exercises `mark.parametrize, torch.device, torch.manual_seed, torch.randn, RMSNorm.to, weight.clone`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `rms norm 批处理 invariant vs standard` 的测试用例。 参数维度：`batch_size, hidden_size, dtype, eps`。 输入或 fixture：`default_vllm_config, batch_size, hidden_size, dtype, eps`。 该测试会调用 `mark.parametrize, torch.device, torch.manual_seed, torch.randn, RMSNorm.to, weight.clone`。 主要通过 mock、回调或输出检查来完成验证。

### test_fused_add_rms_norm_batch_invariant_residual_path (lines 78-164)
```python
def test_fused_add_rms_norm_batch_invariant_residual_path(
    hidden_size: int,
    dtype: torch.dtype,
    eps: float,
):
    """
    Test the batch-invariant fused residual-add + RMSNorm helper directly.
    device = torch.device(DEVICE_TYPE)

    torch.manual_seed(42)
    x_single = torch.randn(1, hidden_size, dtype=dtype, device=device)
    residual_single = torch.randn(1, hidden_size, dtype=dtype, device=device)
    weight = torch.randn(hidden_size, dtype=dtype, device=device)
    x_batch = torch.cat(
        [
            x_single,
    # ... excerpt omitted for brevity ...
        return x, residual
        ref_out,
        rtol=rtol,
        atol=atol,
        msg="Fused add RMSNorm output should stay numerically close to the "
        "batch-invariant RMSNorm reference",
    )
```
**EN:** Parameterized test covering `fused add rms norm batch invariant residual path`. Parameter axes: `hidden_size, dtype, eps`. Inputs/fixtures: `hidden_size, dtype, eps`. It exercises `mark.parametrize, torch.device, torch.manual_seed, torch.randn, torch.cat, fused_add_rms_norm`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `fused add rms norm 批处理 invariant residual path` 的测试用例。 参数维度：`hidden_size, dtype, eps`。 输入或 fixture：`hidden_size, dtype, eps`。 该测试会调用 `mark.parametrize, torch.device, torch.manual_seed, torch.randn, torch.cat, fused_add_rms_norm`。 主要通过 mock、回调或输出检查来完成验证。

### test_rms_norm_3d_input (lines 171-208)
```python
def test_rms_norm_3d_input(
    default_vllm_config, batch_size: int, seq_len: int, hidden_size: int
):
    """
    Test RMS norm with 3D input tensors (batch, seq_len, hidden_size).

    Ensures that the batch-invariant RMS norm correctly handles multi-dimensional
    inputs that are common in transformer models.
    device = torch.device(DEVICE_TYPE)
    dtype = torch.bfloat16
    eps = 1e-6
    torch.manual_seed(42)
    input_tensor = torch.randn(
        batch_size, seq_len, hidden_size, dtype=dtype, device=device
    )
    weight = torch.randn(hidden_size, dtype=dtype, device=device)
    # ... excerpt omitted for brevity ...
        standard_output,
        rtol=rtol,
        atol=atol,
        msg=f"RMS norm mismatch for 3D input with batch_size={batch_size}, "
        f"seq_len={seq_len}, hidden_size={hidden_size}",
```
**EN:** Parameterized test covering `rms norm 3d input`. Parameter axes: `batch_size, seq_len, hidden_size`. Inputs/fixtures: `default_vllm_config, batch_size, seq_len, hidden_size`. It exercises `mark.parametrize, torch.device, torch.manual_seed, torch.randn, RMSNorm.to, weight.clone`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `rms norm 3d input` 的测试用例。 参数维度：`batch_size, seq_len, hidden_size`。 输入或 fixture：`default_vllm_config, batch_size, seq_len, hidden_size`。 该测试会调用 `mark.parametrize, torch.device, torch.manual_seed, torch.randn, RMSNorm.to, weight.clone`。 主要通过 mock、回调或输出检查来完成验证。

### test_rms_norm_numerical_stability (lines 212-268)
```python
def test_rms_norm_numerical_stability(default_vllm_config):
    """
    Test RMS norm numerical stability with extreme values.

    Ensures that both implementations handle edge cases like very small or large
    values without producing NaN or Inf.
    device = torch.device(DEVICE_TYPE)
    dtype = torch.float16
    eps = 1e-6
    hidden_size = 2048
    # Test cases with extreme values
    test_cases = [
        # Very small values
        torch.ones(4, hidden_size, dtype=dtype, device=device) * 1e-5,
        # Very large values
        torch.ones(4, hidden_size, dtype=dtype, device=device) * 1e4,
    # ... excerpt omitted for brevity ...
        assert not torch.isnan(standard_output).any(), (
        assert not torch.isinf(standard_output).any(), (
        assert not torch.isnan(triton_output).any(), (
        assert not torch.isinf(triton_output).any(), (
            triton_output,
            standard_output,
            rtol=2e-1,  # 20% tolerance for extreme values
            atol=2e-1,
            msg=f"RMS norm mismatch for extreme value test case {idx}",
        )
```
**EN:** Test case covering `rms norm numerical stability`. Inputs/fixtures: `default_vllm_config`. It exercises `torch.device, torch.ones, enumerate, RMSNorm.to, weight.clone, rms_norm_layer.forward_cuda`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `rms norm numerical stability` 的测试用例。 输入或 fixture：`default_vllm_config`。 该测试会调用 `torch.device, torch.ones, enumerate, RMSNorm.to, weight.clone, rms_norm_layer.forward_cuda`。 代码主体包含 4 个显式断言。

### test_rms_norm_formula (lines 272-301)
```python
def test_rms_norm_formula(default_vllm_config):
    """
    Test that RMS norm follows the correct mathematical formula.

    Verifies: output = input / sqrt(mean(input^2) + eps) * weight
    """
    device = torch.device(DEVICE_TYPE)
    dtype = torch.float32  # Use float32 for higher precision in formula check
    eps = 1e-6
    hidden_size = 1024

    torch.manual_seed(42)
    input_tensor = torch.randn(8, hidden_size, dtype=dtype, device=device)
    weight = torch.randn(hidden_size, dtype=dtype, device=device)

    # Compute expected output using the formula
    variance = (input_tensor.pow(2).mean(dim=-1, keepdim=True)).to(dtype)
    expected_output = input_tensor * torch.rsqrt(variance + eps) * weight

    # Batch-invariant implementation
    triton_output = triton_rms_norm(input_tensor, weight, eps=eps)

    # Compare against formula
    torch.testing.assert_close(
        triton_output,
        expected_output,
        rtol=1e-4,
        atol=1e-4,
        msg="Triton RMS norm doesn't match expected formula",
    )
```
**EN:** Test case covering `rms norm formula`. Inputs/fixtures: `default_vllm_config`. It exercises `torch.device, torch.manual_seed, torch.randn, mean.to, triton_rms_norm, testing.assert_close`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `rms norm formula` 的测试用例。 输入或 fixture：`default_vllm_config`。 该测试会调用 `torch.device, torch.manual_seed, torch.randn, mean.to, triton_rms_norm, testing.assert_close`。 主要通过 mock、回调或输出检查来完成验证。

### test_rms_norm_different_hidden_sizes (lines 306-339)
```python
def test_rms_norm_different_hidden_sizes(default_vllm_config, hidden_size: int):
    """
    Test RMS norm with various hidden sizes to ensure block size handling.

    The Triton kernel uses a fixed BLOCK_SIZE=1024, so this tests that it
    correctly handles hidden sizes both smaller and larger than the block size.
    device = torch.device(DEVICE_TYPE)
    dtype = torch.bfloat16
    eps = 1e-6
    batch_size = 16
    torch.manual_seed(42)
    input_tensor = torch.randn(batch_size, hidden_size, dtype=dtype, device=device)
    weight = torch.randn(hidden_size, dtype=dtype, device=device)
    # Standard implementation
    rms_norm_layer = RMSNorm(hidden_size, eps=eps, dtype=dtype).to(device)
    # ... excerpt omitted for brevity ...
        triton_output,
        standard_output,
        rtol=rtol,
        atol=atol,
        msg=f"RMS norm mismatch for hidden_size={hidden_size}",
    )
```
**EN:** Parameterized test covering `rms norm different hidden sizes`. Parameter axes: `hidden_size`. Inputs/fixtures: `default_vllm_config, hidden_size`. It exercises `mark.parametrize, torch.device, torch.manual_seed, torch.randn, RMSNorm.to, weight.clone`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `rms norm different hidden sizes` 的测试用例。 参数维度：`hidden_size`。 输入或 fixture：`default_vllm_config, hidden_size`。 该测试会调用 `mark.parametrize, torch.device, torch.manual_seed, torch.randn, RMSNorm.to, weight.clone`。 主要通过 mock、回调或输出检查来完成验证。

### test_rms_norm_determinism (lines 343-375)
```python
def test_rms_norm_determinism(default_vllm_config):
    """
    Test that batch-invariant RMS norm produces deterministic results.

    Runs the same input through the kernel multiple times and verifies
    identical outputs.
    device = torch.device(DEVICE_TYPE)
    dtype = torch.bfloat16
    eps = 1e-6
    hidden_size = 4096
    batch_size = 32
    torch.manual_seed(42)
    input_tensor = torch.randn(batch_size, hidden_size, dtype=dtype, device=device)
    weight = torch.randn(hidden_size, dtype=dtype, device=device)
    # Run multiple times
    # ... excerpt omitted for brevity ...
            output,
            reference,
            rtol=0.0,
            atol=0.0,
            msg=f"RMS norm not deterministic: run {idx} differs from reference",
        )
```
**EN:** Test case covering `rms norm determinism`. Inputs/fixtures: `default_vllm_config`. It exercises `torch.device, torch.manual_seed, torch.randn, range, enumerate, triton_rms_norm`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `rms norm determinism` 的测试用例。 输入或 fixture：`default_vllm_config`。 该测试会调用 `torch.device, torch.manual_seed, torch.randn, range, enumerate, triton_rms_norm`。 主要通过 mock、回调或输出检查来完成验证。

### Module state / 模块级状态 (lines 378-412)
```python
if __name__ == "__main__":
    # Run a quick smoke test
    print("Running quick smoke test of RMS norm implementations...")

    device = torch.device(DEVICE_TYPE)
    batch_size = 8
    hidden_size = 4096
    dtype = torch.bfloat16
    eps = 1e-6
    torch.manual_seed(42)
    input_tensor = torch.randn(batch_size, hidden_size, dtype=dtype, device=device)
    weight = torch.randn(hidden_size, dtype=dtype, device=device)
    # Standard implementation
    rms_norm_layer = RMSNorm(hidden_size, eps=eps, dtype=dtype).to(device)
    rms_norm_layer.weight.data = weight.clone()
    standard_output = rms_norm_layer.forward_cuda(input_tensor)
    # ... excerpt omitted for brevity ...
    print(f"Triton output sample: {triton_output[0, :5].tolist()}")
    if max_diff < 1e-3:
        print("✓ Smoke test passed!")
    else:
        print("✗ Smoke test failed - differences too large")
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `device, batch_size, hidden_size, dtype, eps, input_tensor, ...`. Conditional logic is used to adapt the suite to optional dependencies or platform differences. Shared setup calls include `print, torch.device, torch.manual_seed, torch.randn, RMSNorm.to, ...`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`device, batch_size, hidden_size, dtype, eps, input_tensor, ...`。 条件逻辑用于适配可选依赖或平台差异。 共享初始化调用包括 `print, torch.device, torch.manual_seed, torch.randn, RMSNorm.to, ...`。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch, utils`.
- **CN:** 外部库：`pytest, torch, utils`。
- **EN:** vLLM modules under test: `vllm.model_executor.layers.batch_invariant, vllm.model_executor.layers.layernorm, vllm.platforms, vllm._custom_ops`.
- **CN:** 被测试的 vLLM 模块：`vllm.model_executor.layers.batch_invariant, vllm.model_executor.layers.layernorm, vllm.platforms, vllm._custom_ops`。
