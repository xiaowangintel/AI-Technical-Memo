# test_matmul_batch_invariant.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/determinism/test_matmul_batch_invariant.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Test batch-invariant matmul against torch.matmul for various shape combinations. / 该文件的文档字符串表明其用途：`test batch-invariant matmul against torch.matmul for various shape combinations`。

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
Test batch-invariant matmul against torch.matmul for various shape combinations.

Tests correctness (matches torch.matmul) and batch invariance (result for one
item doesn't change based on other items in the batch).
"""
```
**EN:** Module docstring that declares the scope of the file: Test batch-invariant matmul against torch.matmul for various shape combinations.
**CN:** 模块文档字符串直接说明了文件范围：`test batch-invariant matmul against torch.matmul for various shape combinations`。

### Imports and setup / 导入与设置 (lines 10-15)
```python
import pytest
import torch
from utils import skip_unsupported

from vllm.model_executor.layers.batch_invariant import matmul_batch_invariant
from vllm.platforms import current_platform
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, utils`. vLLM modules under test include `vllm.model_executor.layers.batch_invariant, vllm.platforms`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, utils`。 被测试的 vLLM 模块包括 `vllm.model_executor.layers.batch_invariant, vllm.platforms`。

### Module state / 模块级状态 (line 17)
```python
DEVICE_TYPE = current_platform.device_type
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `DEVICE_TYPE`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`DEVICE_TYPE`。

### test_matmul_correctness (lines 51-80)
```python
def test_matmul_correctness(a_shape, b_shape, dtype):
    """
    Compare matmul_batch_invariant against torch.matmul for various shapes.
    """
    device = torch.device(DEVICE_TYPE)

    torch.manual_seed(42)
    a = torch.rand(a_shape, dtype=dtype, device=device)
    b = torch.rand(b_shape, dtype=dtype, device=device)

    # Standard implementation (CUDA ops)
    standard_output = torch.matmul(a, b)

    # Batch-invariant implementation (Triton)
    triton_output = matmul_batch_invariant(a, b)

    # Compare outputs
    # Use looser tolerance for bfloat16 due to its lower precision
    if dtype == torch.bfloat16:
        rtol, atol = 1e-1, 1e-1  # 10% relative tolerance for bfloat16
    else:
        rtol, atol = 1e-2, 1e-2  # 1% for float16/float32

    torch.testing.assert_close(
        triton_output,
        standard_output,
        rtol=rtol,
        atol=atol,
        msg=f"matmul mismatch for a ndim={a.ndim}, b ndim={b.ndim},",
    )
```
**EN:** Parameterized test covering `matmul correctness`. Parameter axes: `a_shape, b_shape, dtype`. Inputs/fixtures: `a_shape, b_shape, dtype`. It exercises `mark.parametrize, torch.device, torch.manual_seed, torch.rand, torch.matmul, matmul_batch_invariant`. Validation happens through mocks, callbacks, or output inspection.
**CN:** 该代码块是覆盖 `matmul correctness` 的测试用例。 参数维度：`a_shape, b_shape, dtype`。 输入或 fixture：`a_shape, b_shape, dtype`。 该测试会调用 `mark.parametrize, torch.device, torch.manual_seed, torch.rand, torch.matmul, matmul_batch_invariant`。 主要通过 mock、回调或输出检查来完成验证。

### test_matmul_batch_invariance (lines 85-105)
```python
def test_matmul_batch_invariance(dtype):
    """
    Verify that the result for one item is bitwise identical regardless
    of what other items are in the batch.
    """

    device = torch.device(DEVICE_TYPE)

    torch.manual_seed(42)
    a_single = torch.rand((1, 64, 32), dtype=dtype, device=device)
    b = torch.rand((32, 128), dtype=dtype, device=device)

    standard_output = matmul_batch_invariant(a_single, b)

    a_batch = torch.rand((8, 64, 32), dtype=dtype, device=device)
    a_batch[3] = a_single[0]

    batch_output = matmul_batch_invariant(a_batch, b)
    batch_output_a = batch_output[3]

    assert torch.equal(standard_output[0], batch_output_a)
```
**EN:** Parameterized test covering `matmul batch invariance`. Parameter axes: `dtype`. Inputs/fixtures: `dtype`. It exercises `mark.parametrize, torch.device, torch.manual_seed, torch.rand, matmul_batch_invariant, torch.equal`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `matmul 批处理 invariance` 的测试用例。 参数维度：`dtype`。 输入或 fixture：`dtype`。 该测试会调用 `mark.parametrize, torch.device, torch.manual_seed, torch.rand, matmul_batch_invariant, torch.equal`。 代码主体包含 1 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch, utils`.
- **CN:** 外部库：`pytest, torch, utils`。
- **EN:** vLLM modules under test: `vllm.model_executor.layers.batch_invariant, vllm.platforms`.
- **CN:** 被测试的 vLLM 模块：`vllm.model_executor.layers.batch_invariant, vllm.platforms`。
