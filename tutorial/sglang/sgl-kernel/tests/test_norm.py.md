# test_norm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/tests/test_norm.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This pytest module validates `normalization` behavior, numerical correctness, and edge conditions for the sgl-kernel stack. / 该 pytest 模块验证 `normalization` 在 sgl-kernel 栈中的行为、数值正确性与边界条件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Comments and local context
````python
# Adapted from https://github.com/flashinfer-ai/flashinfer/blob/4e8eb1879f9c3ba6d75511e5893183bf8f289a62/tests/test_norm.py

````
**EN:** This block preserves surrounding comments or standalone lines that provide context for the nearby implementation.
**CN:** 该代码块保留了周边注释或独立语句，用来为附近实现提供上下文。

### Lines 3-8: Imports and module setup
````python
import sys

import pytest
import sgl_kernel
import torch
from sgl_kernel.utils import is_arch_support_pdl
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 11-18: `llama_rms_norm` definition
````python
def llama_rms_norm(x, w, eps=1e-6):
    orig_dtype = x.dtype
    x = x.float()
    variance = x.pow(2).mean(dim=-1, keepdim=True)
    x = x * torch.rsqrt(variance + eps)
    x = x * w.float()
    x = x.to(orig_dtype)
    return x
````
**EN:** This section defines `llama_rms_norm` and implements the core logic associated with llama rms normalization.
**CN:** 该部分定义 `llama_rms_norm`，并实现与 llama rms normalization 相关的核心逻辑。

### Lines 21-28: `gemma_rms_norm` definition
````python
def gemma_rms_norm(x, w, eps=1e-6):
    orig_dtype = x.dtype
    x = x.float()
    variance = x.pow(2).mean(dim=-1, keepdim=True)
    x = x * torch.rsqrt(variance + eps)
    x = x * (1.0 + w.float())
    x = x.to(orig_dtype)
    return x
````
**EN:** This section defines `gemma_rms_norm` and implements the core logic associated with gemma rms normalization.
**CN:** 该部分定义 `gemma_rms_norm`，并实现与 gemma rms normalization 相关的核心逻辑。

### Lines 31-40: `gemma_fused_add_rms_norm` definition
````python
def gemma_fused_add_rms_norm(x, residual, w, eps=1e-6):
    orig_dtype = x.dtype
    x = x + residual
    residual = x
    x = x.float()
    variance = x.pow(2).mean(dim=-1, keepdim=True)
    x = x * torch.rsqrt(variance + eps)
    x = x * (1.0 + w.float())
    x = x.to(orig_dtype)
    return x, residual
````
**EN:** This section defines `gemma_fused_add_rms_norm` and implements the core logic associated with gemma fused add rms normalization.
**CN:** 该部分定义 `gemma_fused_add_rms_norm`，并实现与 gemma fused add rms normalization 相关的核心逻辑。

### Lines 43-52: `fused_add_rms_norm` definition
````python
def fused_add_rms_norm(x, residual, weight, eps):
    orig_dtype = x.dtype
    x = x.to(torch.float32)
    x = x + residual.to(torch.float32)
    residual = x.to(orig_dtype)

    variance = x.pow(2).mean(dim=-1, keepdim=True)
    x = x * torch.rsqrt(variance + eps)
    x = (x * weight.float()).to(orig_dtype)
    return x, residual
````
**EN:** This section defines `fused_add_rms_norm` and implements the core logic associated with fused add rms normalization.
**CN:** 该部分定义 `fused_add_rms_norm`，并实现与 fused add rms normalization 相关的核心逻辑。

### Lines 55-71: `test_norm` definition
````python
@pytest.mark.parametrize("batch_size", [1, 19, 99, 989])
@pytest.mark.parametrize("hidden_size", [111, 500, 1024, 3072, 3584, 4096, 8192, 16384])
@pytest.mark.parametrize("dtype", [torch.float16])
@pytest.mark.parametrize("specify_out", [True, False])
def test_norm(batch_size, hidden_size, dtype, specify_out):
    x = torch.randn(batch_size, hidden_size).to(0).to(dtype)
    w = torch.randn(hidden_size).to(0).to(dtype)

    y_ref = llama_rms_norm(x, w)
    enable_pdl = is_arch_support_pdl()
    if specify_out:
        y = torch.empty_like(x)
        sgl_kernel.rmsnorm(x, w, out=y, enable_pdl=enable_pdl)
    else:
        y = sgl_kernel.rmsnorm(x, w, enable_pdl=enable_pdl)

    torch.testing.assert_close(y_ref, y, rtol=1e-3, atol=1e-3)
````
**EN:** This section defines the test `test_norm`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_norm`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 74-96: `test_fused_add_rmsnorm` definition
````python
@pytest.mark.parametrize("batch_size", [1, 19, 99, 989])
@pytest.mark.parametrize("hidden_size", [111, 500, 1024, 3072, 3584, 4096, 8192, 16384])
@pytest.mark.parametrize("dtype", [torch.float16, torch.float32])
def test_fused_add_rmsnorm(batch_size, hidden_size, dtype):
    eps = 1e-6

    x = torch.randn(batch_size, hidden_size, dtype=dtype, device="cuda")
    residual = torch.randn_like(x)
    weight = torch.randn(hidden_size, dtype=dtype, device="cuda")

    x_native, residual_native = fused_add_rms_norm(
        x.clone(), residual.clone(), weight, eps
    )

    x_fused = x.clone()
    residual_fused = residual.clone()
    enable_pdl = is_arch_support_pdl()
    sgl_kernel.fused_add_rmsnorm(
        x_fused, residual_fused, weight, eps, enable_pdl=enable_pdl
    )

    torch.testing.assert_close(x_fused, x_native, rtol=1e-3, atol=1e-3)
    torch.testing.assert_close(residual_fused, residual_native, rtol=1e-3, atol=1e-3)
````
**EN:** This section defines the test `test_fused_add_rmsnorm`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_fused_add_rmsnorm`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 99-115: `test_gemma_norm` definition
````python
@pytest.mark.parametrize("batch_size", [1, 19, 99, 989])
@pytest.mark.parametrize("hidden_size", [111, 500, 1024, 3072, 3584, 4096, 8192, 16384])
@pytest.mark.parametrize("dtype", [torch.float16])
@pytest.mark.parametrize("specify_out", [True, False])
def test_gemma_norm(batch_size, hidden_size, dtype, specify_out):
    x = torch.randn(batch_size, hidden_size).to(0).to(dtype)
    w = torch.randn(hidden_size).to(0).to(dtype)

    y_ref = gemma_rms_norm(x, w)
    enable_pdl = is_arch_support_pdl()
    if specify_out:
        y = torch.empty_like(x)
        sgl_kernel.gemma_rmsnorm(x, w, out=y, enable_pdl=enable_pdl)
    else:
        y = sgl_kernel.gemma_rmsnorm(x, w, enable_pdl=enable_pdl)

    torch.testing.assert_close(y_ref, y, rtol=1e-3, atol=1e-3)
````
**EN:** This section defines the test `test_gemma_norm`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_gemma_norm`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 118-140: `test_gemma_fused_add_rmsnorm` definition
````python
@pytest.mark.parametrize("batch_size", [1, 19, 99, 989])
@pytest.mark.parametrize("hidden_size", [111, 500, 1024, 3072, 3584, 4096, 8192, 16384])
@pytest.mark.parametrize("dtype", [torch.float16])
def test_gemma_fused_add_rmsnorm(batch_size, hidden_size, dtype):
    eps = 1e-6

    x = torch.randn(batch_size, hidden_size, dtype=dtype, device="cuda")
    residual = torch.randn_like(x)
    weight = torch.randn(hidden_size, dtype=dtype, device="cuda")

    x_native, residual_native = gemma_fused_add_rms_norm(
        x.clone(), residual.clone(), weight, eps
    )

    x_fused = x.clone()
    residual_fused = residual.clone()
    enable_pdl = is_arch_support_pdl()
    sgl_kernel.gemma_fused_add_rmsnorm(
        x_fused, residual_fused, weight, eps, enable_pdl=enable_pdl
    )

    torch.testing.assert_close(x_fused, x_native, rtol=1e-3, atol=1e-3)
    torch.testing.assert_close(residual_fused, residual_native, rtol=1e-3, atol=1e-3)
````
**EN:** This section defines the test `test_gemma_fused_add_rmsnorm`. It sets up inputs, runs the target path, and checks the expected result. Parameterized decorators expand the coverage across multiple input cases.
**CN:** 该部分定义测试 `test_gemma_fused_add_rmsnorm`。它会准备输入、执行目标路径，并检查预期结果。 参数化装饰器会把覆盖范围扩展到多组输入场景。

### Lines 143-144: Command-line entry point
````python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
````
**EN:** This block is the script entry point. It triggers the top-level execution path when the file is run directly.
**CN:** 该部分是脚本入口；当文件被直接执行时，它会触发顶层执行路径。

## Key Concepts / 关键概念
- **Role / 角色**: Validation module / 验证模块
- **Primary symbols / 主要符号**: `llama_rms_norm`, `gemma_rms_norm`, `gemma_fused_add_rms_norm`, `fused_add_rms_norm`, `test_norm`, `test_fused_add_rmsnorm`, `test_gemma_norm`, `test_gemma_fused_add_rmsnorm`
- **Core technologies / 核心技术**: PyTorch tensors / PyTorch 张量, pytest parameterization / pytest 参数化
- **Execution style / 执行方式**: Runnable script as well as importable module / 既可直接运行也可作为可导入模块

## Dependencies / 依赖关系
- **Internal / 内部**: `sgl_kernel`, `sgl_kernel.utils`
- **External / 外部**: `pytest`, `sys`, `torch`
- **Runtime hooks / 运行时钩子**: direct script execution path / 直接脚本执行路径
