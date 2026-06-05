# test_hadamard_jit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_hadamard_jit.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import math". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import math”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-139: Imports and module setup
```python
import math
import sys

import numpy as np
import pytest
import torch
import torch.nn.functional as F
from scipy.linalg import hadamard

from sglang.jit_kernel.hadamard import (
    hadamard_transform,
    hadamard_transform_12n,
    hadamard_transform_20n,
    hadamard_transform_28n,
    hadamard_transform_40n,
)
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=128, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=512, suite="nightly-kernel-1-gpu", nightly=True)

# Exact M×N Hadamard matrices (±1 entries) copied from
# python/sglang/jit_kernel/csrc/fast-hadamard-transform/code_gen.py.
# These are non-power-of-2 Hadamard matrices constructed via Paley/Williamson methods.
# "+" = +1, "-" = -1.  Used by the _12n/_20n/_28n/_40n kernel variants.

_HAD_12_STR = """
+-++++++++++
--+-+-+-+-+-
+++-++----++
+---+--+-++-
+++++-++----
+-+---+--+-+
++--+++-++--
+--++---+--+
++----+++-++
+--+-++---+-
++++----+++-
+-+--+-++---
"""

_HAD_20_STR = """
+----+----++--++-++-
-+----+---+++---+-++
--+----+---+++-+-+-+
---+----+---+++++-+-
----+----++--++-++-+
-+++++-----+--+++--+
+-+++-+---+-+--+++--
++-++--+---+-+--+++-
+++-+---+---+-+--+++
++++-----++--+-+--++
--++-+-++-+-----++++
---++-+-++-+---+-+++
+---++-+-+--+--++-++
++---++-+----+-+++-+
-++---++-+----+++++-
-+--+--++-+----+----
+-+-----++-+----+---
-+-+-+---+--+----+--
--+-+++------+----+-
+--+--++------+----+
"""

_HAD_28_STR = """
+------++----++-+--+-+--++--
-+-----+++-----+-+--+-+--++-
--+-----+++---+-+-+----+--++
---+-----+++---+-+-+-+--+--+
----+-----+++---+-+-+++--+--
-----+-----++++--+-+--++--+-
------++----++-+--+-+--++--+
--++++-+-------++--+++-+--+-
---++++-+-----+-++--+-+-+--+
+---+++--+----++-++--+-+-+--
++---++---+----++-++--+-+-+-
+++---+----+----++-++--+-+-+
++++--------+-+--++-++--+-+-
-++++--------+++--++--+--+-+
-+-++-++--++--+--------++++-
# ...
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 140-158: Function `_parse_hadamard_str`
```python
def _parse_hadamard_str(s):
    """Parse a ±1 string matrix definition into a numpy array."""
    s = s.strip().replace("+", "1").replace("-", "-1").split()
    return np.stack(
        [np.fromstring(" ".join(s[i]), dtype=np.int32, sep=" ") for i in range(len(s))]
    )


# Parsed M×M special Hadamard matrices, keyed by M (the "multiple").
# Copied from python/sglang/jit_kernel/csrc/fast-hadamard-transform/code_gen.py
# (had_12_paley, had_20_will, had_28_will, had_40_tpal)
_SPECIAL_MATRICES = {
    12: _parse_hadamard_str(_HAD_12_STR),
    20: _parse_hadamard_str(_HAD_20_STR),
    28: _parse_hadamard_str(_HAD_28_STR),
    40: _parse_hadamard_str(_HAD_40_STR),
}
```
**EN:** This block defines `_parse_hadamard_str`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_parse_hadamard_str`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 159-177: Function `hadamard_transform_ref`
```python
def hadamard_transform_ref(x, scale=1.0):
    """Reference impl for the general (power-of-2) hadamard_transform.

    Pads dim to the next power of 2, multiplies by the full H matrix
    via F.linear, then truncates back to the original dim.
    """
    x_shape = x.shape
    dim = x.shape[-1]
    x = x.reshape(-1, dim)
    log_dim = math.ceil(math.log2(dim)) if dim > 0 else 0
    dim_padded = 2**log_dim if dim > 0 else 1
    if dim != dim_padded:
        x = F.pad(x, (0, dim_padded - dim))
    H = torch.tensor(hadamard(dim_padded, dtype=float), dtype=x.dtype, device=x.device)
    out = F.linear(x, H)
    out = out * scale
    return out[..., :dim].reshape(*x_shape)
```
**EN:** This block defines `hadamard_transform_ref`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `hadamard_transform_ref`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 178-227: Function `hadamard_transform_mn_ref`
```python
def hadamard_transform_mn_ref(x, multiple, scale=1.0):
    """Reference impl for the M×N hadamard variants (_12n, _20n, _28n, _40n).

    The kernel computes (H_M ⊗ H_N) · x via two steps:
      1) H_N (power-of-2 Hadamard) along the N dimension
      2) H_M (special ±1 matrix) along the M dimension
    where dim = M * N, M = `multiple`, N = power of 2.
    """
    x_shape = x.shape
    dim = x.shape[-1]
    x = x.reshape(-1, dim)

    # The kernel requires dim % (4*M) == 0 (for vectorized memory access).
    # See python/sglang/jit_kernel/hadamard.py: pad_multiple = 4 * 12 / 4 * 20 / etc.
    pad_multiple = 4 * multiple
    if dim % pad_multiple != 0:
        pad_size = pad_multiple - dim % pad_multiple
        x = F.pad(x, (0, pad_size))
        dim_padded = dim + pad_size
    else:
        dim_padded = dim

    # N = dim_padded / M, must be a power of 2
    n = dim_padded // multiple
    log_n = int(math.log2(n))
    assert 2**log_n == n, f"n={n} is not a power of 2"

    batch = x.shape[0]
    x = x.reshape(batch, multiple, n)  # (batch, M, N)

    # Step 1: apply H_N (standard power-of-2 Hadamard) along the N dimension
    H_n = torch.tensor(hadamard(n, dtype=float), dtype=x.dtype, device=x.device)
    x = torch.einsum("bmn,kn->bmk", x, H_n)

    # Step 2: apply H_M (special ±1 matrix) along the M dimension
    H_m = torch.tensor(
        _SPECIAL_MATRICES[multiple].astype(float), dtype=x.dtype, device=x.device
    )
    x = torch.einsum("bmn,km->bkn", x, H_m)

    x = x.reshape(batch, -1) * scale
    return x[..., : x_shape[-1]].reshape(*x_shape)


@pytest.mark.parametrize("dtype", [torch.float32, torch.float16, torch.bfloat16])
@pytest.mark.parametrize(
    "dim",
    # Power-of-2 dims from sgl-kernel/tests/test_hadamard.py (old AOT test)
    [1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096, 8192, 16384, 32768],
)
```
**EN:** This block defines `hadamard_transform_mn_ref`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `hadamard_transform_mn_ref`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 228-258: Function `test_hadamard_transform`
```python
def test_hadamard_transform(dim, dtype):
    device = "cuda"

    # Tolerances from sgl-kernel/tests/test_hadamard.py (old AOT test)
    if dtype == torch.float32:
        rtol, atol = 3e-4, 3e-3
    elif dtype == torch.bfloat16:
        rtol, atol = 1e-2, 5e-2
    else:  # float16
        rtol, atol = 3e-3, 5e-3

    torch.random.manual_seed(0)
    batch_size = 15

    x = torch.randn(batch_size, dim, device=device, dtype=dtype)
    scale = 1.0 / math.sqrt(dim)

    out = hadamard_transform(x, scale=scale)
    # Compute reference in float32 from a detached copy to avoid precision loss
    out_ref = hadamard_transform_ref(x.detach().clone().float(), scale=scale)

    torch.testing.assert_close(out.float(), out_ref, rtol=rtol, atol=atol)


@pytest.mark.parametrize("dtype", [torch.float32, torch.float16, torch.bfloat16])
@pytest.mark.parametrize(
    "dim",
    # Non-power-of-2 dims to test the padding path
    # (137 from sgl-kernel/tests/test_hadamard.py, 500/1000 added for coverage)
    [137, 500, 1000],
)
```
**EN:** This block defines `test_hadamard_transform`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_hadamard_transform`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 259-281: Function `test_hadamard_transform_non_power_of_two`
```python
def test_hadamard_transform_non_power_of_two(dim, dtype):
    device = "cuda"

    if dtype == torch.float32:
        rtol, atol = 3e-4, 3e-3
    elif dtype == torch.bfloat16:
        rtol, atol = 1e-2, 5e-2
    else:
        rtol, atol = 3e-3, 5e-3

    torch.random.manual_seed(42)
    batch_size = 15

    x = torch.randn(batch_size, dim, device=device, dtype=dtype)
    scale = 1.0 / math.sqrt(dim)

    out = hadamard_transform(x, scale=scale)
    out_ref = hadamard_transform_ref(x.detach().clone().float(), scale=scale)

    torch.testing.assert_close(out.float(), out_ref, rtol=rtol, atol=atol)


@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
```
**EN:** This block defines `test_hadamard_transform_non_power_of_two`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_hadamard_transform_non_power_of_two`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 282-302: Function `test_hadamard_transform_3d_input`
```python
def test_hadamard_transform_3d_input(dtype):
    device = "cuda"

    if dtype == torch.bfloat16:
        rtol, atol = 1e-2, 5e-2
    else:
        rtol, atol = 3e-3, 5e-3

    torch.random.manual_seed(0)

    x = torch.randn(4, 8, 256, device=device, dtype=dtype)
    scale = 1.0 / math.sqrt(256)

    out = hadamard_transform(x, scale=scale)
    assert out.shape == x.shape

    out_ref = hadamard_transform_ref(x.detach().clone().float(), scale=scale)
    torch.testing.assert_close(out.float(), out_ref, rtol=rtol, atol=atol)


@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
```
**EN:** This block defines `test_hadamard_transform_3d_input`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_hadamard_transform_3d_input`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 303-332: Function `test_hadamard_transform_scale_one`
```python
def test_hadamard_transform_scale_one(dtype):
    device = "cuda"

    if dtype == torch.bfloat16:
        rtol, atol = 1e-2, 5e-2
    else:
        rtol, atol = 3e-3, 5e-3

    torch.random.manual_seed(0)

    x = torch.randn(8, 64, device=device, dtype=dtype)

    out = hadamard_transform(x, scale=1.0)
    out_ref = hadamard_transform_ref(x.detach().clone().float(), scale=1.0)

    torch.testing.assert_close(out.float(), out_ref, rtol=rtol, atol=atol)


# Test dimensions for M×N variants: dim = M * N where N = 2^k.
# M = 12/20/28/40 are the non-power-of-2 Hadamard sizes registered in
# python/sglang/jit_kernel/hadamard.py (Hadamard12NKernel, ..., Hadamard40NKernel).
# range(2,9) gives N = 4,8,...,256 so dims cover a practical range.
_12N_DIMS = [12 * (2**k) for k in range(2, 9)]  # 48, 96, ... , 3072
_20N_DIMS = [20 * (2**k) for k in range(2, 9)]  # 80, 160, ... , 5120
_28N_DIMS = [28 * (2**k) for k in range(2, 9)]  # 112, 224, ... , 7168
_40N_DIMS = [40 * (2**k) for k in range(2, 9)]  # 160, 320, ... , 10240


@pytest.mark.parametrize("dtype", [torch.float32, torch.float16, torch.bfloat16])
@pytest.mark.parametrize("dim", _12N_DIMS)
```
**EN:** This block defines `test_hadamard_transform_scale_one`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_hadamard_transform_scale_one`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 333-356: Function `test_hadamard_transform_12n`
```python
def test_hadamard_transform_12n(dim, dtype):
    device = "cuda"

    if dtype == torch.float32:
        rtol, atol = 3e-4, 3e-3
    elif dtype == torch.bfloat16:
        rtol, atol = 1e-2, 5e-2
    else:
        rtol, atol = 3e-3, 5e-3

    torch.random.manual_seed(0)
    batch_size = 15

    x = torch.randn(batch_size, dim, device=device, dtype=dtype)
    scale = 1.0 / math.sqrt(dim)

    out = hadamard_transform_12n(x, scale=scale)
    out_ref = hadamard_transform_mn_ref(x.detach().clone().float(), 12, scale=scale)

    torch.testing.assert_close(out.float(), out_ref, rtol=rtol, atol=atol)


@pytest.mark.parametrize("dtype", [torch.float32, torch.float16, torch.bfloat16])
@pytest.mark.parametrize("dim", _20N_DIMS)
```
**EN:** This block defines `test_hadamard_transform_12n`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_hadamard_transform_12n`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 357-380: Function `test_hadamard_transform_20n`
```python
def test_hadamard_transform_20n(dim, dtype):
    device = "cuda"

    if dtype == torch.float32:
        rtol, atol = 3e-4, 3e-3
    elif dtype == torch.bfloat16:
        rtol, atol = 1e-2, 5e-2
    else:
        rtol, atol = 3e-3, 5e-3

    torch.random.manual_seed(0)
    batch_size = 15

    x = torch.randn(batch_size, dim, device=device, dtype=dtype)
    scale = 1.0 / math.sqrt(dim)

    out = hadamard_transform_20n(x, scale=scale)
    out_ref = hadamard_transform_mn_ref(x.detach().clone().float(), 20, scale=scale)

    torch.testing.assert_close(out.float(), out_ref, rtol=rtol, atol=atol)


@pytest.mark.parametrize("dtype", [torch.float32, torch.float16, torch.bfloat16])
@pytest.mark.parametrize("dim", _28N_DIMS)
```
**EN:** This block defines `test_hadamard_transform_20n`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_hadamard_transform_20n`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 381-404: Function `test_hadamard_transform_28n`
```python
def test_hadamard_transform_28n(dim, dtype):
    device = "cuda"

    if dtype == torch.float32:
        rtol, atol = 3e-4, 3e-3
    elif dtype == torch.bfloat16:
        rtol, atol = 1e-2, 5e-2
    else:
        rtol, atol = 3e-3, 5e-3

    torch.random.manual_seed(0)
    batch_size = 15

    x = torch.randn(batch_size, dim, device=device, dtype=dtype)
    scale = 1.0 / math.sqrt(dim)

    out = hadamard_transform_28n(x, scale=scale)
    out_ref = hadamard_transform_mn_ref(x.detach().clone().float(), 28, scale=scale)

    torch.testing.assert_close(out.float(), out_ref, rtol=rtol, atol=atol)


@pytest.mark.parametrize("dtype", [torch.float32, torch.float16, torch.bfloat16])
@pytest.mark.parametrize("dim", _40N_DIMS)
```
**EN:** This block defines `test_hadamard_transform_28n`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_hadamard_transform_28n`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 405-428: Function `test_hadamard_transform_40n`
```python
def test_hadamard_transform_40n(dim, dtype):
    device = "cuda"

    if dtype == torch.float32:
        rtol, atol = 3e-4, 3e-3
    elif dtype == torch.bfloat16:
        rtol, atol = 1e-2, 5e-2
    else:
        rtol, atol = 3e-3, 5e-3

    torch.random.manual_seed(0)
    batch_size = 15

    x = torch.randn(batch_size, dim, device=device, dtype=dtype)
    scale = 1.0 / math.sqrt(dim)

    out = hadamard_transform_40n(x, scale=scale)
    out_ref = hadamard_transform_mn_ref(x.detach().clone().float(), 40, scale=scale)

    torch.testing.assert_close(out.float(), out_ref, rtol=rtol, atol=atol)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
```
**EN:** This block defines `test_hadamard_transform_40n`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_hadamard_transform_40n`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `math`
- `sys`
- `numpy as np`
- `pytest`
- `torch`
- `torch.nn.functional as F`
- `scipy.linalg -> hadamard`
- `sglang.jit_kernel.hadamard -> (`
- `sglang.test.ci.ci_register -> register_cuda_ci`
