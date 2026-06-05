# test_matmul.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/unit/language/test_matmul.py`
- **EN:** Pytest module covering matmul behavior in Triton's Python tests. It contains 23 top-level definition(s) and 8 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 matmul 行为。 该文件包含 23 个顶层定义，以及 8 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```python
import math
import pytest
import torch
import triton
import triton.language as tl
from test_mxfp import MXFP4Tensor, MXScaleTensor
import re
from triton._internal_testing import is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4, is_hip_cdna, is_hip_gfx1250
```
- **EN:** Imports the modules used in this scope: `math`, `pytest`, `torch`, `triton`, `triton.language`, `test_mxfp`, `re`, `triton._internal_testing`. Relevant themes: Triton language operations.
- **CN:** 导入此作用域使用的模块：`math`、`pytest`、`torch`、`triton`、`triton.language`、`test_mxfp`、`re`、`triton._internal_testing`。 相关主题：Triton language 操作。

### Lines 9-13

```python


def f8_to_f16(x, dtype):

    @triton.jit
```
- **EN:** Defines the helper function `f8_to_f16`. Parameters: `x`, `dtype`. Nested definitions in this scope: `kernel`. Key calls include `torch.empty`, `getattr`, `tl.program_id`, `tl.load`, `tl.store`, `triton.reinterpret`, and 4 more. This scope touches Triton compilation or JIT kernels, Triton language operations, PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `f8_to_f16`。 参数：`x`、`dtype`。 该作用域中的嵌套定义：`kernel`。 关键调用包括 `torch.empty`、`getattr`、`tl.program_id`、`tl.load`、`tl.store`、`triton.reinterpret` 等另外 4 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、PyTorch 张量准备与校验。

#### Lines 13-14

```python
    @triton.jit
    def kernel(Y, X, N, BLOCK_SIZE: tl.constexpr):
```
- **EN:** Defines the helper function `kernel`. Decorators: `triton.jit`. Parameters: `Y`, `X`, `N`, `BLOCK_SIZE`. Key calls include `tl.program_id`, `tl.load`, `tl.store`, `tl.arange`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `kernel`。 装饰器：`triton.jit`。 参数：`Y`、`X`、`N`、`BLOCK_SIZE`。 关键调用包括 `tl.program_id`、`tl.load`、`tl.store`、`tl.arange`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

##### Lines 15-19

```python
        pid = tl.program_id(0)
        offs = pid * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
        mask = offs < N
        x = tl.load(X + offs, mask=mask)
        tl.store(Y + offs, x, mask=mask)
```
- **EN:** Prepares or updates state through `pid`, `offs`, `mask`, `x`. Invokes `tl.program_id`, `tl.arange`, `tl.load`, `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid`、`offs`、`mask`、`x` 准备或更新状态。 调用 `tl.program_id`、`tl.arange`、`tl.load`、`tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 20-25

```python

    ret = torch.empty(x.shape, dtype=torch.float16, device=x.device)
    grid = lambda META: (triton.cdiv(x.numel(), META['BLOCK_SIZE']), )
    dtype = getattr(tl, dtype)
    kernel[grid](ret, triton.reinterpret(x, dtype), ret.numel(), BLOCK_SIZE=1024)
    return ret
```
- **EN:** Prepares or updates state through `ret`, `grid`, `dtype`. Invokes `torch.empty`, `triton.cdiv`, `x.numel`, `getattr`, `triton.reinterpret`, `ret.numel` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `ret`、`grid`、`dtype` 准备或更新状态。 调用 `torch.empty`、`triton.cdiv`、`x.numel`、`getattr`、`triton.reinterpret`、`ret.numel` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 26-37

```python


@triton.jit
def matmul_kernel(  #
        a_ptr, b_ptr, output_ptr,  #
        M, N, K,  #
        stride_am, stride_ak,  #
        stride_bk, stride_bn,  #
        stride_cm, stride_cn,  #
        BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr, BLOCK_K: tl.constexpr,  #
        NUM_STAGES: tl.constexpr, SCALE_A: tl.constexpr = None, PRECISION: tl.constexpr = "ieee",
        A_TRANS: tl.constexpr = False, EPILOGUE_SUBTILE: tl.constexpr = False, dummy: tl.constexpr = 0):
```
- **EN:** Defines the helper function `matmul_kernel`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `output_ptr`, `M`, `N`, `K`, `stride_am`, `stride_ak`, and 13 more. Key calls include `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.zeros`, `tl.load`, `tl.dot`, and 4 more. This scope touches Triton compilation or JIT kernels, Triton language operations, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义辅助函数 `matmul_kernel`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`output_ptr`、`M`、`N`、`K`、`stride_am`、`stride_ak` 等另外 13 项。 关键调用包括 `tl.program_id`、`tl.cdiv`、`tl.arange`、`tl.zeros`、`tl.load`、`tl.dot` 等另外 4 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 38-44

```python
    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_M)
    pid_m = pid % num_pid_m
    pid_n = pid // num_pid_m
    offs_am = (pid_m * BLOCK_M + tl.arange(0, BLOCK_M)) % M
    offs_bn = (pid_n * BLOCK_N + tl.arange(0, BLOCK_N)) % N
    offs_k = tl.arange(0, BLOCK_K)
```
- **EN:** Prepares or updates state through `pid`, `num_pid_m`, `pid_m`, `pid_n`, `offs_am`, `offs_bn`, `offs_k`. Invokes `tl.program_id`, `tl.cdiv`, `tl.arange` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid`、`num_pid_m`、`pid_m`、`pid_n`、`offs_am`、`offs_bn`、`offs_k` 准备或更新状态。 调用 `tl.program_id`、`tl.cdiv`、`tl.arange` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 45-48

```python
    if not A_TRANS:
        a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_k[None, :] * stride_ak)
    else:
        a_ptrs = a_ptr + (offs_k[:, None] * stride_ak + offs_am[None, :] * stride_am)
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 49-50

```python
    b_ptrs = b_ptr + (offs_k[:, None] * stride_bk + offs_bn[None, :] * stride_bn)
    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=output_ptr.dtype.element_ty)
```
- **EN:** Prepares or updates state through `b_ptrs`, `accumulator`. Invokes `tl.zeros` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `b_ptrs`、`accumulator` 准备或更新状态。 调用 `tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 51-60

```python
    for k in tl.range(0, tl.cdiv(K, BLOCK_K), num_stages=NUM_STAGES):
        a = tl.load(a_ptrs)
        if SCALE_A is not None:
            a = a * SCALE_A
        if A_TRANS:
            a = a.T
        b = tl.load(b_ptrs)
        accumulator = tl.dot(a, b, acc=accumulator, out_dtype=output_ptr.dtype.element_ty, input_precision=PRECISION)
        a_ptrs += BLOCK_K * stride_ak
        b_ptrs += BLOCK_K * stride_bk
```
- **EN:** Invokes `tl.cdiv`, `tl.load`, `tl.dot` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.cdiv`、`tl.load`、`tl.dot` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

#### Lines 61-75

```python
    if EPILOGUE_SUBTILE:
        acc = tl.reshape(accumulator, (BLOCK_M, 2, BLOCK_N // 2))
        acc = tl.permute(acc, (0, 2, 1))
        acc0, acc1 = tl.split(acc)
        offs_cm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
        offs_cn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N // 2)
        output_ptrs0 = output_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
        output_ptrs1 = output_ptrs0 + stride_cn * (BLOCK_N // 2)
        tl.store(output_ptrs0, acc0)
        tl.store(output_ptrs1, acc1)
    else:
        offs_cm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
        offs_cn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
        output_ptrs = output_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
        tl.store(output_ptrs, accumulator)
```
- **EN:** Invokes `tl.reshape`, `tl.permute`, `tl.split`, `tl.store`, `tl.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.reshape`、`tl.permute`、`tl.split`、`tl.store`、`tl.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

### Lines 76-78

```python


def get_src_element_ty_size(dtype_str):
```
- **EN:** Defines the helper function `get_src_element_ty_size`. Parameters: `dtype_str`. Key calls include `ValueError`.
- **CN:** 定义辅助函数 `get_src_element_ty_size`。 参数：`dtype_str`。 关键调用包括 `ValueError`。

#### Lines 79-80

```python
    if dtype_str == "float8e5":
        return 1
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 81-82

```python
    if dtype_str == "float16":
        return 2
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 83-84

```python
    if dtype_str == "float32" or dtype_str == "tensorfloat32":
        return 4
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 85-86

```python
    if dtype_str == "float64":
        return 8
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 87-87

```python
    raise ValueError(f"Unknown dtype {dtype_str}")
```
- **EN:** Invokes `ValueError` to execute the test logic.
- **CN:** 调用 `ValueError` 执行测试逻辑。

### Lines 88-100

```python


@pytest.mark.parametrize("dtype_src_str", ["float32", "tensorfloat32", "float16", "float8e5", "float64"])
@pytest.mark.parametrize("dtype_dst_str", ["float32", "float16", "float64"])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K, NUM_STAGES", [(128, 128, 16, 4), (64, 128, 32, 4), (32, 32, 32, 4),
                                                                   (256, 128, 32, 4), (64, 512, 32, 2),
                                                                   (512, 64, 32, 2), (64, 16, 64, 4)])
@pytest.mark.parametrize("NUM_CTAS", [1, 2])
@pytest.mark.parametrize("NUM_WARPS", [4, 8])
@pytest.mark.parametrize("EPILOGUE_SUBTILE", [True, False])
@pytest.mark.parametrize("LAYOUT_16x256", [True, False])
def test_simple_matmul(dtype_src_str, dtype_dst_str, BLOCK_M, BLOCK_N, BLOCK_K, NUM_STAGES, NUM_WARPS, NUM_CTAS, device,
                       EPILOGUE_SUBTILE, LAYOUT_16x256, monkeypatch):
```
- **EN:** Defines the test function `test_simple_matmul`. Decorators: `pytest.mark.parametrize('dtype_src_str', ['float32', 'tensorfloat32', 'float16', 'float8e5', 'float64'])`, `pytest.mark.parametrize('dtype_dst_str', ['float32', 'float16', 'float64'])`, `pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K, NUM_STAGES', [(128, 128, 16, 4), (64, 128, 32, 4), (32, 32, 32, 4), (256, 128, 32, 4), (64, 512, 32, 2), (512, 64, 32, 2), (64, 16, 64, 4)])`, `pytest.mark.parametrize('NUM_CTAS', [1, 2])`, `pytest.mark.parametrize('NUM_WARPS', [4, 8])`, `pytest.mark.parametrize('EPILOGUE_SUBTILE', [True, False])`, and 1 more. Parameters: `dtype_src_str`, `dtype_dst_str`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `NUM_STAGES`, `NUM_WARPS`, `NUM_CTAS`, and 4 more. Key calls include `pytest.mark.parametrize`, `torch.manual_seed`, `getattr`, `torch.empty`, `output.to`, `torch.testing.assert_close`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_simple_matmul`。 装饰器：`pytest.mark.parametrize('dtype_src_str', ['float32', 'tensorfloat32', 'float16', 'float8e5', 'float64'])`、`pytest.mark.parametrize('dtype_dst_str', ['float32', 'float16', 'float64'])`、`pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K, NUM_STAGES', [(128, 128, 16, 4), (64, 128, 32, 4), (32, 32, 32, 4), (256, 128, 32, 4), (64, 512, 32, 2), (512, 64, 32, 2), (64, 16, 64, 4)])`、`pytest.mark.parametrize('NUM_CTAS', [1, 2])`、`pytest.mark.parametrize('NUM_WARPS', [4, 8])`、`pytest.mark.parametrize('EPILOGUE_SUBTILE', [True, False])` 等另外 1 项。 参数：`dtype_src_str`、`dtype_dst_str`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`NUM_STAGES`、`NUM_WARPS`、`NUM_CTAS` 等另外 4 项。 关键调用包括 `pytest.mark.parametrize`、`torch.manual_seed`、`getattr`、`torch.empty`、`output.to`、`torch.testing.assert_close` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 101-102

```python
    if NUM_CTAS > 1 and (not is_cuda() or torch.cuda.get_device_capability()[0] < 9):
        pytest.skip("Clusters requires nvidia compute capability >= 9")
```
- **EN:** Invokes `pytest.skip`, `is_cuda`, `torch.cuda.get_device_capability` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `pytest.skip`、`is_cuda`、`torch.cuda.get_device_capability` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 103-104

```python
    shared_mem_accum = (BLOCK_K * BLOCK_M + BLOCK_K * BLOCK_N) * NUM_STAGES * get_src_element_ty_size(dtype_src_str)
    shared_mem_avail = triton.runtime.driver.active.utils.get_device_properties(0)["max_shared_mem"]
```
- **EN:** Prepares or updates state through `shared_mem_accum`, `shared_mem_avail`. Invokes `get_src_element_ty_size`, `triton.runtime.driver.active.utils.get_device_properties` to execute the test logic. Relevant themes: plugin or compiler extension points, runtime driver interaction.
- **CN:** 通过 `shared_mem_accum`、`shared_mem_avail` 准备或更新状态。 调用 `get_src_element_ty_size`、`triton.runtime.driver.active.utils.get_device_properties` 执行测试逻辑。 相关主题：插件或编译器扩展点、运行时驱动交互。

#### Lines 105-106

```python
    if shared_mem_accum > shared_mem_avail:
        pytest.skip("Skipped due to insufficient shared memory on this GPU.")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 107-108

```python
    if is_hip() and (not is_hip_cdna3()) and dtype_src_str == "tensorfloat32":
        pytest.skip("tensorfloat32 is only supported on HIP CDNA3")
```
- **EN:** Invokes `is_hip`, `pytest.skip`, `is_hip_cdna3` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip`、`pytest.skip`、`is_hip_cdna3` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 109-110

```python
    if dtype_src_str == "float8e5" and BLOCK_K == 16:
        pytest.skip("Skipping cases small K for float8")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 111-112

```python
    if dtype_src_str == "float8e5" and device == "cuda" and torch.cuda.get_device_capability()[0] < 9:
        pytest.skip("Float8 requires compute capability >= 9")
```
- **EN:** Invokes `pytest.skip`, `torch.cuda.get_device_capability` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `pytest.skip`、`torch.cuda.get_device_capability` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 113-114

```python
    if (dtype_src_str == "float64") != (dtype_dst_str == "float64"):
        pytest.skip("Skipping unsupported case")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 115-116

```python
    if "float32" in dtype_src_str and dtype_dst_str == "float16":
        pytest.skip("Skipping unsupported case")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 117-118

```python
    if "float32" == dtype_src_str and NUM_CTAS > 1:
        pytest.skip("FMA matmul not supported for multiple CTAs")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: matrix multiplication workflows.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：矩阵乘法工作流。

#### Lines 119-120

```python
    if (BLOCK_M < 64 or (BLOCK_M == 64 and BLOCK_N == 16)) and NUM_CTAS > 1:
        pytest.skip("multi-CTAs is broken for mmav2")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 121-122

```python
    if EPILOGUE_SUBTILE and (is_hip() or NUM_CTAS > 1 or BLOCK_N >= 512):
        pytest.skip("creates convert layout too big to fit in smem")
```
- **EN:** Invokes `pytest.skip`, `is_hip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `pytest.skip`、`is_hip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 123-124

```python
    if LAYOUT_16x256 and (not is_cuda() or torch.cuda.get_device_capability()[0] < 10):
        pytest.skip("skip forcing tmem layout on non blackwell targets.")
```
- **EN:** Invokes `pytest.skip`, `is_cuda`, `torch.cuda.get_device_capability` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, layout transformation reasoning.
- **CN:** 调用 `pytest.skip`、`is_cuda`、`torch.cuda.get_device_capability` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、布局变换推理。

#### Lines 125-128

```python
    M, N, K = 1024, 512, 256
    torch.manual_seed(42)
    precision = "tf32" if dtype_src_str == "tensorfloat32" else "ieee"
    dtype_src_str = "float32" if dtype_src_str == "tensorfloat32" else dtype_src_str
```
- **EN:** Prepares or updates state through `M`, `N`, `K`, `precision`, `dtype_src_str`. Invokes `torch.manual_seed` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `M`、`N`、`K`、`precision`、`dtype_src_str` 准备或更新状态。 调用 `torch.manual_seed` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 129-139

```python
    if dtype_src_str == "float8e5":
        a = torch.randint(20, 40, (M, K), dtype=torch.uint8, device=device).view(torch.float8_e5m2)
        b = torch.randint(20, 40, (K, N), dtype=torch.uint8, device=device).view(torch.float8_e5m2)
        A = f8_to_f16(a, dtype_src_str)
        B = f8_to_f16(b, dtype_src_str)
    else:
        dtype_src = getattr(torch, dtype_src_str)
        a = torch.randn(M, K, dtype=dtype_src, device=device)
        b = torch.randn(K, N, dtype=dtype_src, device=device)
        A = a
        B = b
```
- **EN:** Invokes `f8_to_f16`, `getattr`, `torch.randn`, `torch.randint` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 调用 `f8_to_f16`、`getattr`、`torch.randn`、`torch.randint` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 140-142

```python
    # pass a dummy constexpr argument to force recompilation.
    if LAYOUT_16x256:
        monkeypatch.setenv("TRITON_PREFER_TMEM_16x256_LAYOUT", "1")
```
- **EN:** Invokes `monkeypatch.setenv` to execute the test logic. Branches on runtime or test conditions. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `monkeypatch.setenv` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：布局变换推理。

#### Lines 143-151

```python
    dtype_dst = getattr(torch, dtype_dst_str)
    output = torch.empty((M, N), dtype=dtype_dst, device=device)
    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), 1)
    k = matmul_kernel[grid](a, b, output, M, N, K, a.stride(0), a.stride(1), b.stride(0), b.stride(1), output.stride(0),
                            output.stride(1), BLOCK_M, BLOCK_N, BLOCK_K, NUM_STAGES=NUM_STAGES, PRECISION=precision,
                            num_warps=NUM_WARPS, num_ctas=NUM_CTAS, EPILOGUE_SUBTILE=EPILOGUE_SUBTILE,
                            dummy=LAYOUT_16x256)
    ref_out = torch.matmul(A, B).to(torch.float32)
    output = output.to(torch.float32)
```
- **EN:** Prepares or updates state through `dtype_dst`, `output`, `grid`, `k`, `ref_out`. Invokes `getattr`, `torch.empty`, `triton.cdiv`, `a.stride`, `b.stride`, `output.stride`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points, layout transformation reasoning.
- **CN:** 通过 `dtype_dst`、`output`、`grid`、`k`、`ref_out` 准备或更新状态。 调用 `getattr`、`torch.empty`、`triton.cdiv`、`a.stride`、`b.stride`、`output.stride` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点、布局变换推理。

#### Lines 152-161

```python
    if dtype_src_str == "float32":
        # TF32 has lower precision than torch.float32
        atol = 0.03
        rtol = 0.03
    elif dtype_dst_str == "float16":
        atol = 0.06
        rtol = 0.06
    else:
        atol = 0.001
        rtol = 0.001
```
- **EN:** Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 162-162

```python
    torch.testing.assert_close(ref_out, output, atol=atol, rtol=rtol)
```
- **EN:** Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 163-179

```python
    # Make sure the mma is pipelined by checking if in the TTGIR we see two mmav5
    # operations. (Pipeliner will add additional mma operation by peeling the prologue.)
    # This applies only if TCv5 MMA is used (M % 64 == 0 and N % 8 == 0) and
    # when MMA arguments loads are pipelined (N > 16)
    if (device == "cuda" and torch.cuda.get_device_capability()[0] == 10 and NUM_STAGES > 1 and BLOCK_M % 64 == 0
            and BLOCK_N % 8 == 0 and BLOCK_N > 16
            and not (precision == "ieee" and (dtype_src_str == "float32" or dtype_src_str == "float64"))):
        ttgir = k.asm["ttgir"]
        count = ttgir.count("ttng.tc_gen5_mma")
        assert count == 2, "The TTGIR does not match the expected pattern."
        ptx = k.asm["ptx"]
        if LAYOUT_16x256:
            assert "16x256b" in ptx, "PTX does not contain 16x256b"
        else:
            if "32x32b" not in ptx and "16x32b" not in ptx:
                print(ptx)
            assert ("32x32b" in ptx) or ("16x32b" in ptx), "PTX does not contain 32x32b or 16x32b"
```
- **EN:** Invokes `ttgir.count`, `torch.cuda.get_device_capability` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, plugin or compiler extension points, debugging and inspection paths, layout transformation reasoning.
- **CN:** 调用 `ttgir.count`、`torch.cuda.get_device_capability` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、插件或编译器扩展点、调试与检查路径、布局变换推理。

### Lines 180-189

```python


# persistent matmul with fused loops
@triton.jit
def simple_persistent_kernel(a_ptr, b_ptr, c_ptr, M, N, K, stride_am, stride_ak,  #
                             stride_bk, stride_bn,  #
                             stride_cm, stride_cn, BLOCK_SIZE_M: tl.constexpr, BLOCK_SIZE_N: tl.constexpr,
                             BLOCK_SIZE_K: tl.constexpr,  #
                             GROUP_SIZE_M: tl.constexpr, NUM_SMS: tl.constexpr,
                             DISALLOW_ACC_MULTI_BUFFER: tl.constexpr):
```
- **EN:** Defines the helper function `simple_persistent_kernel`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `M`, `N`, `K`, `stride_am`, `stride_ak`, and 10 more. Key calls include `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.zeros`, `tl.where`, `tl.load`, and 6 more. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `simple_persistent_kernel`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`M`、`N`、`K`、`stride_am`、`stride_ak` 等另外 10 项。 关键调用包括 `tl.program_id`、`tl.cdiv`、`tl.arange`、`tl.zeros`、`tl.where`、`tl.load` 等另外 6 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 190-196

```python
    start_pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)
    k_tiles = tl.cdiv(K, BLOCK_SIZE_K)
    num_tiles = num_pid_m * num_pid_n

    tiles_per_SM = num_tiles // NUM_SMS
```
- **EN:** Prepares or updates state through `start_pid`, `num_pid_m`, `num_pid_n`, `k_tiles`, `num_tiles`, `tiles_per_SM`. Invokes `tl.program_id`, `tl.cdiv` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `start_pid`、`num_pid_m`、`num_pid_n`、`k_tiles`、`num_tiles`、`tiles_per_SM` 准备或更新状态。 调用 `tl.program_id`、`tl.cdiv` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 197-198

```python
    if start_pid < num_tiles % NUM_SMS:
        tiles_per_SM += 1
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 199-211

```python

    tile_id = start_pid - NUM_SMS
    tile_id_c = start_pid - NUM_SMS  # remat value to use in the epilogue
    ki = -1

    offs_k_for_mask = tl.arange(0, BLOCK_SIZE_K)

    num_pid_in_group = GROUP_SIZE_M * num_pid_n

    offs_am = tl.arange(0, BLOCK_SIZE_M)
    offs_bn = tl.arange(0, BLOCK_SIZE_N)

    accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
```
- **EN:** Prepares or updates state through `tile_id`, `tile_id_c`, `ki`, `offs_k_for_mask`, `num_pid_in_group`, `offs_am`, `offs_bn`, `accumulator`. Invokes `tl.arange`, `tl.zeros` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `tile_id`、`tile_id_c`、`ki`、`offs_k_for_mask`、`num_pid_in_group`、`offs_am`、`offs_bn`、`accumulator` 准备或更新状态。 调用 `tl.arange`、`tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 212-256

```python

    for _ in tl.range(0, k_tiles * tiles_per_SM, disallow_acc_multi_buffer=DISALLOW_ACC_MULTI_BUFFER):
        ki = tl.where(ki == k_tiles - 1, 0, ki + 1)
        if ki == 0:
            tile_id += NUM_SMS
            group_id = tile_id // num_pid_in_group
            first_pid_m = group_id * GROUP_SIZE_M
            group_size_m = min(num_pid_m - first_pid_m, GROUP_SIZE_M)
            pid_m = first_pid_m + (tile_id % group_size_m)
            pid_n = (tile_id % num_pid_in_group) // group_size_m

            start_m = pid_m * BLOCK_SIZE_M
            start_n = pid_n * BLOCK_SIZE_N
            offs_am = start_m + tl.arange(0, BLOCK_SIZE_M)
            offs_bn = start_n + tl.arange(0, BLOCK_SIZE_N)
            offs_am = tl.where(offs_am < M, offs_am, 0)
            offs_bn = tl.where(offs_bn < N, offs_bn, 0)
            offs_am = tl.max_contiguous(tl.multiple_of(offs_am, BLOCK_SIZE_M), BLOCK_SIZE_M)
            offs_bn = tl.max_contiguous(tl.multiple_of(offs_bn, BLOCK_SIZE_N), BLOCK_SIZE_N)
        offs_k = ki * BLOCK_SIZE_K + tl.arange(0, BLOCK_SIZE_K)
        a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_k[None, :] * stride_ak)
        b_ptrs = b_ptr + (offs_k[:, None] * stride_bk + offs_bn[None, :] * stride_bn)

        a = tl.load(a_ptrs, mask=offs_k_for_mask[None, :] < K - ki * BLOCK_SIZE_K, other=0.0)
        b = tl.load(b_ptrs, mask=offs_k_for_mask[:, None] < K - ki * BLOCK_SIZE_K, other=0.0)
        accumulator = tl.dot(a, b, accumulator)

        if ki == k_tiles - 1:
            tile_id_c += NUM_SMS
            group_id = tile_id_c // num_pid_in_group
            first_pid_m = group_id * GROUP_SIZE_M
            group_size_m = min(num_pid_m - first_pid_m, GROUP_SIZE_M)
            pid_m = first_pid_m + (tile_id_c % group_size_m)
            pid_n = (tile_id_c % num_pid_in_group) // group_size_m

            offs_cm = pid_m * BLOCK_SIZE_M + tl.arange(0, BLOCK_SIZE_M)
            offs_cn = pid_n * BLOCK_SIZE_N + tl.arange(0, BLOCK_SIZE_N)
            c_ptrs = c_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
            c_mask = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)
            if (c_ptr.dtype == tl.float8e4nv):
                c = accumulator.to(tl.float8e4nv)
            else:
                c = accumulator.to(tl.float16)
            tl.store(c_ptrs, c, mask=c_mask)
            accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
```
- **EN:** Invokes `tl.where`, `tl.load`, `tl.dot`, `min`, `tl.max_contiguous`, `tl.arange`, and 4 more to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.where`、`tl.load`、`tl.dot`、`min`、`tl.max_contiguous`、`tl.arange` 等另外 4 项 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作。

### Lines 257-263

```python


@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(128, 128, 16), (64, 128, 32), (32, 32, 32), (256, 128, 16),
                                                       (64, 512, 16), (512, 64, 16), (64, 16, 16)])
@pytest.mark.parametrize("NUM_WARPS", [4, 8])
@pytest.mark.parametrize("DISALLOW_ACC_MULTI_BUFFER", [True, False])
def test_simple_persistent_matmul(BLOCK_M, BLOCK_N, BLOCK_K, NUM_WARPS, DISALLOW_ACC_MULTI_BUFFER, device):
```
- **EN:** Defines the test function `test_simple_persistent_matmul`. Decorators: `pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 16), (64, 128, 32), (32, 32, 32), (256, 128, 16), (64, 512, 16), (512, 64, 16), (64, 16, 16)])`, `pytest.mark.parametrize('NUM_WARPS', [4, 8])`, `pytest.mark.parametrize('DISALLOW_ACC_MULTI_BUFFER', [True, False])`. Parameters: `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `NUM_WARPS`, `DISALLOW_ACC_MULTI_BUFFER`, `device`. Key calls include `pytest.mark.parametrize`, `torch.randn`, `torch.empty`, `torch.testing.assert_close`, `min`, `a.stride`, and 8 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_simple_persistent_matmul`。 装饰器：`pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 16), (64, 128, 32), (32, 32, 32), (256, 128, 16), (64, 512, 16), (512, 64, 16), (64, 16, 16)])`、`pytest.mark.parametrize('NUM_WARPS', [4, 8])`、`pytest.mark.parametrize('DISALLOW_ACC_MULTI_BUFFER', [True, False])`。 参数：`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`NUM_WARPS`、`DISALLOW_ACC_MULTI_BUFFER`、`device`。 关键调用包括 `pytest.mark.parametrize`、`torch.randn`、`torch.empty`、`torch.testing.assert_close`、`min`、`a.stride` 等另外 8 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 264-285

```python
    M, N, K = 1024, 512, 256
    NUM_STAGES = 3
    a = torch.randn(M, K, dtype=torch.float16, device=device)
    b = torch.randn(K, N, dtype=torch.float16, device=device)
    output = torch.empty((M, N), dtype=torch.float16, device=device)

    # Fake small number of SMS to test that persistent kernel works reliably
    NUM_SMS = 8

    grid = (min(NUM_SMS, triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N)), )
    k = simple_persistent_kernel[grid](
        a, b, output,  #
        M, N, K,  #
        a.stride(0), a.stride(1),  #
        b.stride(0), b.stride(1),  #
        output.stride(0), output.stride(1),  #
        BLOCK_SIZE_M=BLOCK_M, BLOCK_SIZE_N=BLOCK_N, BLOCK_SIZE_K=BLOCK_K,  #
        GROUP_SIZE_M=8, NUM_SMS=NUM_SMS, DISALLOW_ACC_MULTI_BUFFER=DISALLOW_ACC_MULTI_BUFFER, num_stages=NUM_STAGES,
        num_warps=NUM_WARPS)
    ref_out = torch.matmul(a.to(torch.float32), b.to(torch.float32)).to(torch.float16)

    torch.testing.assert_close(ref_out, output, atol=0.01, rtol=0.01)
```
- **EN:** Prepares or updates state through `M`, `N`, `K`, `NUM_STAGES`, `a`, `b`, `output`, `NUM_SMS`, and 3 more. Invokes `torch.randn`, `torch.empty`, `min`, `triton.cdiv`, `a.stride`, `b.stride`, and 5 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points, random-data generation.
- **CN:** 通过 `M`、`N`、`K`、`NUM_STAGES`、`a`、`b`、`output`、`NUM_SMS` 等另外 3 项 准备或更新状态。 调用 `torch.randn`、`torch.empty`、`min`、`triton.cdiv`、`a.stride`、`b.stride` 等另外 5 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点、随机数据生成。

#### Lines 286-294

```python

    # Make sure the mma is pipelined by checking if in the TTGIR we have peeled mmav5 ops.
    # This applies only if TCv5 MMA is used (M % 64 == 0 and N % 8 == 0) and
    # when MMA arguments loads are pipelined (N > 16)
    if (device == "cuda" and torch.cuda.get_device_capability()[0] == 10 and BLOCK_M % 64 == 0 and BLOCK_N % 8 == 0
            and BLOCK_N > 16):
        ttgir = k.asm["ttgir"]
        pattern = "ttng.tc_gen5_mma"
        assert ttgir.count(pattern) > 0, "Expect peeled mmav5 operations."
```
- **EN:** Invokes `ttgir.count`, `torch.cuda.get_device_capability` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `ttgir.count`、`torch.cuda.get_device_capability` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

### Lines 295-307

```python


@triton.jit
def mxfp_matmul(  #
        a_ptr, b_ptr, output_ptr,  #
        a_scale, b_scale,  #
        M, N, K,  #
        stride_scale: tl.constexpr,  #
        stride_am, stride_ak,  #
        stride_bk, stride_bn,  #
        stride_cm, stride_cn,  #
        BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr, BLOCK_K: tl.constexpr,  #
        NUM_STAGES: tl.constexpr):
```
- **EN:** Defines the helper function `mxfp_matmul`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `output_ptr`, `a_scale`, `b_scale`, `M`, `N`, `K`, and 11 more. Key calls include `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.zeros`, `tl.store`, `tl.load`, and 1 more. This scope touches Triton compilation or JIT kernels, Triton language operations, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义辅助函数 `mxfp_matmul`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`output_ptr`、`a_scale`、`b_scale`、`M`、`N`、`K` 等另外 11 项。 关键调用包括 `tl.program_id`、`tl.cdiv`、`tl.arange`、`tl.zeros`、`tl.store`、`tl.load` 等另外 1 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 308-320

```python
    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_M)
    pid_m = pid % num_pid_m
    pid_n = pid // num_pid_m
    offs_am = (pid_m * BLOCK_M + tl.arange(0, BLOCK_M)) % M
    offs_bn = (pid_n * BLOCK_N + tl.arange(0, BLOCK_N)) % N
    offs_k = tl.arange(0, BLOCK_K)
    offs_scale_k = tl.arange(0, BLOCK_K // 32)
    a_scale_ptr = a_scale + offs_am[:, None] * stride_scale + offs_scale_k[None, :]
    b_scale_ptr = b_scale + offs_bn[:, None] * stride_scale + offs_scale_k[None, :]
    a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_k[None, :] * stride_ak)
    b_ptrs = b_ptr + (offs_k[:, None] * stride_bk + offs_bn[None, :] * stride_bn)
    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=output_ptr.dtype.element_ty)
```
- **EN:** Prepares or updates state through `pid`, `num_pid_m`, `pid_m`, `pid_n`, `offs_am`, `offs_bn`, `offs_k`, `offs_scale_k`, and 5 more. Invokes `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.zeros` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid`、`num_pid_m`、`pid_m`、`pid_n`、`offs_am`、`offs_bn`、`offs_k`、`offs_scale_k` 等另外 5 项 准备或更新状态。 调用 `tl.program_id`、`tl.cdiv`、`tl.arange`、`tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 321-330

```python
    for k in tl.range(0, tl.cdiv(K, BLOCK_K), num_stages=NUM_STAGES):
        a = tl.load(a_ptrs)
        b = tl.load(b_ptrs)
        scale_a = tl.load(a_scale_ptr)
        scale_b = tl.load(b_scale_ptr)
        accumulator = tl.dot_scaled(a, scale_a, "e5m2", b, scale_b, "e5m2", accumulator)
        a_ptrs += BLOCK_K * stride_ak
        b_ptrs += BLOCK_K * stride_bk
        a_scale_ptr += BLOCK_K // 32
        b_scale_ptr += BLOCK_K // 32
```
- **EN:** Invokes `tl.cdiv`, `tl.load`, `tl.dot_scaled` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.cdiv`、`tl.load`、`tl.dot_scaled` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

#### Lines 331-335

```python
    offs_cm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
    offs_cn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
    output_ptrs = output_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
    c_mask = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)
    tl.store(output_ptrs, accumulator, mask=c_mask)
```
- **EN:** Prepares or updates state through `offs_cm`, `offs_cn`, `output_ptrs`, `c_mask`. Invokes `tl.arange`, `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `offs_cm`、`offs_cn`、`output_ptrs`、`c_mask` 准备或更新状态。 调用 `tl.arange`、`tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 336-338

```python


def fp8e8m0_to_float32(scale):
```
- **EN:** Defines the helper function `fp8e8m0_to_float32`. Parameters: `scale`. Key calls include `scale.view`, `scale.to`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `fp8e8m0_to_float32`。 参数：`scale`。 关键调用包括 `scale.view`、`scale.to`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 339-343

```python
    scale = scale.view(torch.uint8)
    scale = scale.to(torch.int32)
    scale = scale << 23
    scale = scale.view(torch.float32)
    return scale
```
- **EN:** Prepares or updates state through `scale`. Invokes `scale.view`, `scale.to` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `scale` 准备或更新状态。 调用 `scale.view`、`scale.to` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 344-353

```python


@pytest.mark.interpreter
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(128, 128, 128), (256, 128, 128), (128, 256, 128),
                                                       (128, 256, 256), (128, 128, 64), (128, 64, 128), (128, 16, 256),
                                                       (128, 16, 64)])
@pytest.mark.parametrize("NUM_STAGES", [1, 3])
@pytest.mark.parametrize("NUM_WARPS", [4, 8])
@pytest.mark.parametrize("nonKDim", ([0, 16, 32] if (is_hip_cdna() or is_hip_gfx1250()) else [0]))
def test_mxfp(BLOCK_M, BLOCK_N, BLOCK_K, NUM_STAGES, nonKDim, NUM_WARPS, device):
```
- **EN:** Defines the test function `test_mxfp`. Decorators: `pytest.mark.interpreter`, `pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 128), (256, 128, 128), (128, 256, 128), (128, 256, 256), (128, 128, 64), (128, 64, 128), (128, 16, 256), (128, 16, 64)])`, `pytest.mark.parametrize('NUM_STAGES', [1, 3])`, `pytest.mark.parametrize('NUM_WARPS', [4, 8])`, `pytest.mark.parametrize('nonKDim', [0, 16, 32] if is_hip_cdna() or is_hip_gfx1250() else [0])`. Parameters: `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `NUM_STAGES`, `nonKDim`, `NUM_WARPS`, `device`. Key calls include `pytest.mark.parametrize`, `torch.manual_seed`, `f8_to_f16`, `torch.randint`, `getattr`, `torch.empty`, and 20 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_mxfp`。 装饰器：`pytest.mark.interpreter`、`pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 128), (256, 128, 128), (128, 256, 128), (128, 256, 256), (128, 128, 64), (128, 64, 128), (128, 16, 256), (128, 16, 64)])`、`pytest.mark.parametrize('NUM_STAGES', [1, 3])`、`pytest.mark.parametrize('NUM_WARPS', [4, 8])`、`pytest.mark.parametrize('nonKDim', [0, 16, 32] if is_hip_cdna() or is_hip_gfx1250() else [0])`。 参数：`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`NUM_STAGES`、`nonKDim`、`NUM_WARPS`、`device`。 关键调用包括 `pytest.mark.parametrize`、`torch.manual_seed`、`f8_to_f16`、`torch.randint`、`getattr`、`torch.empty` 等另外 20 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 354-356

```python
    M = 1024
    N = 512
    K = 2048
```
- **EN:** Prepares or updates state through `M`, `N`, `K`.
- **CN:** 通过 `M`、`N`、`K` 准备或更新状态。

#### Lines 357-358

```python
    if K % BLOCK_K != 0:
        pytest.skip("Kernel requires shapes aligned by K dimension")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 359-367

```python
    if is_cuda() and torch.cuda.get_device_capability()[0] < 10:
        pytest.skip("Requires compute capability >= 10")
    elif is_hip():
        if not (is_hip_cdna4() or is_hip_gfx1250()):
            pytest.skip("Scaled mxfp8 matmul is only natively supported on CDNA4 or above")
        if (nonKDim == 16 and BLOCK_K < 128) or (nonKDim == 32 and BLOCK_K < 64):
            pytest.skip(f"CDNA4 does not support {BLOCK_K=} for scaled mfma {nonKDim=} variants")
        if (BLOCK_M == 256 or BLOCK_N == 256) and BLOCK_K == 256:
            pytest.skip("Config requires too much shared memory")
```
- **EN:** Invokes `is_cuda`, `pytest.skip`, `is_hip`, `torch.cuda.get_device_capability`, `is_hip_cdna4`, `is_hip_gfx1250` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 调用 `is_cuda`、`pytest.skip`、`is_hip`、`torch.cuda.get_device_capability`、`is_hip_cdna4`、`is_hip_gfx1250` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 368-370

```python

    if BLOCK_N == 256 and BLOCK_K == 256:
        NUM_STAGES = min(NUM_STAGES, 2)
```
- **EN:** Invokes `min` to execute the test logic. Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `min` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

#### Lines 371-384

```python
    torch.manual_seed(42)
    dtype_src_str = "float8e5"
    dtype_dst_str = "float32"
    a = torch.randint(20, 40, (M, K), dtype=torch.uint8, device=device).view(torch.float8_e5m2)
    a_f16 = f8_to_f16(a, dtype_src_str)
    b = torch.randint(20, 40, (K, N), dtype=torch.uint8, device=device).view(torch.float8_e5m2)
    b_f16 = f8_to_f16(b, dtype_src_str)
    a_scale = torch.randint(64, 130, (M, K // 32), dtype=torch.uint8, device=device)
    b_scale = torch.randint(64, 130, (N, K // 32), dtype=torch.uint8, device=device)

    dtype_dst = getattr(torch, dtype_dst_str)
    output = torch.empty((M, N), dtype=dtype_dst, device=device)
    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), 1)
    kernel_kwargs = {}
```
- **EN:** Prepares or updates state through `dtype_src_str`, `dtype_dst_str`, `a`, `a_f16`, `b`, `b_f16`, `a_scale`, `b_scale`, and 4 more. Invokes `torch.manual_seed`, `torch.randint`, `f8_to_f16`, `getattr`, `torch.empty`, `triton.cdiv` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `dtype_src_str`、`dtype_dst_str`、`a`、`a_f16`、`b`、`b_f16`、`a_scale`、`b_scale` 等另外 4 项 准备或更新状态。 调用 `torch.manual_seed`、`torch.randint`、`f8_to_f16`、`getattr`、`torch.empty`、`triton.cdiv` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 385-386

```python
    if is_hip():
        kernel_kwargs["matrix_instr_nonkdim"] = nonKDim
```
- **EN:** Invokes `is_hip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 387-404

```python

    out = mxfp_matmul[grid](a, b, output, a_scale, b_scale, M, N, K, a_scale.stride(0), a.stride(0), a.stride(1),
                            b.stride(0), b.stride(1), output.stride(0), output.stride(1), BLOCK_M, BLOCK_N, BLOCK_K,
                            NUM_STAGES=NUM_STAGES, **kernel_kwargs, num_warps=NUM_WARPS)
    a_scale_f32 = fp8e8m0_to_float32(a_scale)
    b_scale_f32 = fp8e8m0_to_float32(b_scale)
    a_scale_f32 = a_scale_f32.repeat_interleave(32, dim=1)
    b_scale_f32 = b_scale_f32.repeat_interleave(32, dim=1)

    # b_scales are always col major
    b_scale_f32 = b_scale_f32.T.contiguous()

    a = a_f16 * a_scale_f32
    b = b_f16 * b_scale_f32
    ref_out = torch.matmul(a, b).to(torch.float32)
    output = output.to(torch.float32)
    atol = 0.0001
    torch.testing.assert_close(ref_out, output, atol=atol, rtol=0)
```
- **EN:** Prepares or updates state through `out`, `a_scale_f32`, `b_scale_f32`, `a`, `b`, `ref_out`, `output`, `atol`. Invokes `a_scale.stride`, `a.stride`, `b.stride`, `output.stride`, `fp8e8m0_to_float32`, `a_scale_f32.repeat_interleave`, and 5 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 通过 `out`、`a_scale_f32`、`b_scale_f32`、`a`、`b`、`ref_out`、`output`、`atol` 准备或更新状态。 调用 `a_scale.stride`、`a.stride`、`b.stride`、`output.stride`、`fp8e8m0_to_float32`、`a_scale_f32.repeat_interleave` 等另外 5 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 405-408

```python

    if is_cuda() and torch.cuda.get_device_capability()[0] == 12:
        ptx = out.asm["ptx"]
        assert "mma.sync.aligned.m16n8k32.row.col.kind::mxf8f6f4.block_scale.scale_vec::1X" in ptx
```
- **EN:** Invokes `is_cuda`, `torch.cuda.get_device_capability` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `is_cuda`、`torch.cuda.get_device_capability` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

### Lines 409-413

```python


def _knob_promote_lhs_to_tmem(monkeypatch):
    # Promoting the LHS to TMEM should be patched because it will otherwise
    # unintentionally be enabled for all consecutive tests if using os.environ
```
- **EN:** Defines the helper function `_knob_promote_lhs_to_tmem`. Parameters: `monkeypatch`. Key calls include `monkeypatch.setenv`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `_knob_promote_lhs_to_tmem`。 参数：`monkeypatch`。 关键调用包括 `monkeypatch.setenv`。 该作用域涉及布局变换推理。

#### Lines 414-414

```python
    monkeypatch.setenv("ALLOW_LHS_TMEM_LAYOUT_CONVERSION", "1")
```
- **EN:** Invokes `monkeypatch.setenv` to execute the test logic. Relevant themes: layout transformation reasoning.
- **CN:** 调用 `monkeypatch.setenv` 执行测试逻辑。 相关主题：布局变换推理。

### Lines 415-430

```python


@triton.jit
def block_scale_mxfp_matmul(  #
        a_ptr, b_ptr, output_ptr,  #
        a_scale, b_scale,  #
        M, N, K,  #
        stride_sk, stride_sb, stride_sc, stride_sd: tl.constexpr,  # Need tl.constexpr to pipeline scale load. Why?
        stride_am, stride_ak,  #
        stride_bk, stride_bn,  #
        stride_cm, stride_cn,  #
        BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr, BLOCK_K: tl.constexpr,  #
        NUM_STAGES: tl.constexpr, USE_2D_SCALE_LOAD: tl.constexpr):
    # This kernel assumes a_scale and b_scale are coming in with shapes
    # [BLOCK_M(or N) // 128, BLOCK_K // 128, 32, 4, 4] for optimial performance
    # on nvidia sm100+ HW
```
- **EN:** Defines the helper function `block_scale_mxfp_matmul`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `output_ptr`, `a_scale`, `b_scale`, `M`, `N`, `K`, and 15 more. Key calls include `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.zeros`, `tl.store`, `tl.load`, and 5 more. This scope touches Triton compilation or JIT kernels, Triton language operations, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义辅助函数 `block_scale_mxfp_matmul`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`output_ptr`、`a_scale`、`b_scale`、`M`、`N`、`K` 等另外 15 项。 关键调用包括 `tl.program_id`、`tl.cdiv`、`tl.arange`、`tl.zeros`、`tl.store`、`tl.load` 等另外 5 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 431-440

```python
    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_M)
    pid_m = pid % num_pid_m
    pid_n = pid // num_pid_m
    offs_am = (pid_m * BLOCK_M + tl.arange(0, BLOCK_M)) % M
    offs_bn = (pid_n * BLOCK_N + tl.arange(0, BLOCK_N)) % N
    offs_k = tl.arange(0, BLOCK_K)

    offs_sm = (pid_m * (BLOCK_M // 128) + tl.arange(0, BLOCK_M // 128))
    offs_sn = (pid_n * (BLOCK_N // 128) + tl.arange(0, BLOCK_N // 128))
```
- **EN:** Prepares or updates state through `pid`, `num_pid_m`, `pid_m`, `pid_n`, `offs_am`, `offs_bn`, `offs_k`, `offs_sm`, and 1 more. Invokes `tl.program_id`, `tl.cdiv`, `tl.arange` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid`、`num_pid_m`、`pid_m`、`pid_n`、`offs_am`、`offs_bn`、`offs_k`、`offs_sm` 等另外 1 项 准备或更新状态。 调用 `tl.program_id`、`tl.cdiv`、`tl.arange` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 441-455

```python

    if USE_2D_SCALE_LOAD:
        offs_inner = tl.arange(0, (BLOCK_K // 128) * 32 * 4 * 4)
        a_scale_ptr = a_scale + offs_sm[:, None] * stride_sk + offs_inner[None, :]
        b_scale_ptr = b_scale + offs_sn[:, None] * stride_sk + offs_inner[None, :]
    else:
        offs_sk = tl.arange(0, (BLOCK_K // 128))
        offs_sc = tl.arange(0, 32)
        offs_sd = tl.arange(0, 4)
        a_scale_ptr = a_scale + (offs_sm[:, None, None, None, None] * stride_sk + offs_sk[None, :, None, None, None] *
                                 stride_sb + offs_sc[None, None, :, None, None] * stride_sc +
                                 offs_sd[None, None, None, :, None] * stride_sd + offs_sd[None, None, None, None, :])
        b_scale_ptr = b_scale + (offs_sn[:, None, None, None, None] * stride_sk + offs_sk[None, :, None, None, None] *
                                 stride_sb + offs_sc[None, None, :, None, None] * stride_sc +
                                 offs_sd[None, None, None, :, None] * stride_sd + offs_sd[None, None, None, None, :])
```
- **EN:** Invokes `tl.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

#### Lines 456-459

```python

    a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_k[None, :] * stride_ak)
    b_ptrs = b_ptr + (offs_k[:, None] * stride_bk + offs_bn[None, :] * stride_bn)
    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=output_ptr.dtype.element_ty)
```
- **EN:** Prepares or updates state through `a_ptrs`, `b_ptrs`, `accumulator`. Invokes `tl.zeros` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `a_ptrs`、`b_ptrs`、`accumulator` 准备或更新状态。 调用 `tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 460-480

```python
    for k in tl.range(0, tl.cdiv(K, BLOCK_K), num_stages=NUM_STAGES):
        a = tl.load(a_ptrs)
        b = tl.load(b_ptrs)
        scale_a = tl.load(a_scale_ptr)
        scale_b = tl.load(b_scale_ptr)

        if USE_2D_SCALE_LOAD:
            scale_a = scale_a.reshape(BLOCK_M // 128, BLOCK_K // 128, 32, 4, 4)
            scale_b = scale_b.reshape(BLOCK_N // 128, BLOCK_K // 128, 32, 4, 4)

        # Scales are coming in for optimial performance, but we reshape here for
        # the canonical inputs to dot_scaled
        # These reshapes and transposes will be optimized away during lowering
        scale_a = scale_a.trans(0, 3, 2, 1, 4).reshape(BLOCK_M, BLOCK_K // 32)
        scale_b = scale_b.trans(0, 3, 2, 1, 4).reshape(BLOCK_N, BLOCK_K // 32)
        accumulator = tl.dot_scaled(a, scale_a, "e5m2", b, scale_b, "e5m2", accumulator)

        a_ptrs += BLOCK_K * stride_ak
        b_ptrs += BLOCK_K * stride_bk
        a_scale_ptr += BLOCK_K // 128 * stride_sb
        b_scale_ptr += BLOCK_K // 128 * stride_sb
```
- **EN:** Invokes `tl.cdiv`, `tl.load`, `tl.dot_scaled`, `scale_a.reshape`, `scale_b.reshape`, `scale_a.trans`, and 1 more to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.cdiv`、`tl.load`、`tl.dot_scaled`、`scale_a.reshape`、`scale_b.reshape`、`scale_a.trans` 等另外 1 项 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

#### Lines 481-485

```python
    offs_cm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
    offs_cn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
    output_ptrs = output_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
    c_mask = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)
    tl.store(output_ptrs, accumulator, mask=c_mask)
```
- **EN:** Prepares or updates state through `offs_cm`, `offs_cn`, `output_ptrs`, `c_mask`. Invokes `tl.arange`, `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `offs_cm`、`offs_cn`、`output_ptrs`、`c_mask` 准备或更新状态。 调用 `tl.arange`、`tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 486-495

```python


@triton.jit
def _gemm_kernel_preshuffled_scales_cdna4(a_ptr, b_ptr, c_ptr, a_scales_ptr, b_scales_ptr, M, N, K, stride_am,
                                          stride_ak, stride_bk, stride_bn, stride_cm, stride_cn, stride_asm, stride_ask,
                                          stride_bsn, stride_bsk,
                                          # Meta-parameters
                                          DTYPE_A: tl.constexpr, DTYPE_B: tl.constexpr, BLOCK_M: tl.constexpr,
                                          BLOCK_N: tl.constexpr, BLOCK_K: tl.constexpr, mfma_nonkdim: tl.constexpr,
                                          preshuffle: tl.constexpr, fast_math: tl.constexpr):
```
- **EN:** Defines the helper function `_gemm_kernel_preshuffled_scales_cdna4`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `a_scales_ptr`, `b_scales_ptr`, `M`, `N`, `K`, and 18 more. Key calls include `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.zeros`, `accumulator.to`, `tl.store`, and 2 more. This scope touches Triton compilation or JIT kernels, Triton language operations, cache management behavior, matrix multiplication workflows.
- **CN:** 定义辅助函数 `_gemm_kernel_preshuffled_scales_cdna4`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`a_scales_ptr`、`b_scales_ptr`、`M`、`N`、`K` 等另外 18 项。 关键调用包括 `tl.program_id`、`tl.cdiv`、`tl.arange`、`tl.zeros`、`accumulator.to`、`tl.store` 等另外 2 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、缓存管理行为、矩阵乘法工作流。

#### Lines 496-512

```python
    """Kernel for computing the matmul C = A x B.
    A_scales and B_scales are in e8m0 format.
    A has shape (M, K), B has shape (K, N) and C has shape (M, N)
    """

    PACK_FACTOR_A: tl.constexpr = 2 if DTYPE_A == "e2m1" else 1
    PACK_FACTOR_B: tl.constexpr = 2 if DTYPE_B == "e2m1" else 1

    pid = tl.program_id(axis=0)

    num_pid_n = tl.cdiv(N, BLOCK_N)
    pid_m = pid // num_pid_n
    pid_n = pid % num_pid_n

    # We assume 32 elements along K share the same scale.
    SCALE_GROUP_SIZE: tl.constexpr = 32
    MX_SCALE_BLOCK_K: tl.constexpr = BLOCK_K // SCALE_GROUP_SIZE
```
- **EN:** Prepares or updates state through `PACK_FACTOR_A`, `PACK_FACTOR_B`, `pid`, `num_pid_n`, `pid_m`, `pid_n`, `SCALE_GROUP_SIZE`, `MX_SCALE_BLOCK_K`. Invokes `tl.program_id`, `tl.cdiv` to execute the test logic. Relevant themes: Triton language operations, matrix multiplication workflows.
- **CN:** 通过 `PACK_FACTOR_A`、`PACK_FACTOR_B`、`pid`、`num_pid_n`、`pid_m`、`pid_n`、`SCALE_GROUP_SIZE`、`MX_SCALE_BLOCK_K` 准备或更新状态。 调用 `tl.program_id`、`tl.cdiv` 执行测试逻辑。 相关主题：Triton language 操作、矩阵乘法工作流。

#### Lines 513-517

```python

    if preshuffle:
        NON_K_PRESHUFFLE_BLOCK_SIZE: tl.constexpr = 32
    else:
        NON_K_PRESHUFFLE_BLOCK_SIZE: tl.constexpr = 1
```
- **EN:** Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

#### Lines 518-529

```python

    # Create pointers for first block of A and B input matrices
    # The BLOCK sizes are of the elements and in fp4 we pack 2 per uint8 container.
    offs_ak = tl.arange(0, BLOCK_K // PACK_FACTOR_A)
    offs_bk = tl.arange(0, BLOCK_K // PACK_FACTOR_B)
    offs_am = (pid_m * BLOCK_M + tl.arange(0, BLOCK_M)) % M
    offs_bn = (pid_n * BLOCK_N + tl.arange(0, BLOCK_N)) % N
    a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_ak[None, :] * stride_ak)
    b_ptrs = b_ptr + (offs_bk[:, None] * stride_bk + offs_bn[None, :] * stride_bn)

    # Create pointers for the first block of A and B scales
    offs_ks = tl.arange(0, MX_SCALE_BLOCK_K * NON_K_PRESHUFFLE_BLOCK_SIZE)
```
- **EN:** Prepares or updates state through `offs_ak`, `offs_bk`, `offs_am`, `offs_bn`, `a_ptrs`, `b_ptrs`, `offs_ks`. Invokes `tl.arange` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `offs_ak`、`offs_bk`、`offs_am`、`offs_bn`、`a_ptrs`、`b_ptrs`、`offs_ks` 准备或更新状态。 调用 `tl.arange` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 530-536

```python

    # B scales are N x K even though B operand is K x N.
    if a_scales_ptr is not None:
        offs_asm = (pid_m *
                    (BLOCK_M // NON_K_PRESHUFFLE_BLOCK_SIZE) + tl.arange(0,
                                                                         (BLOCK_M // NON_K_PRESHUFFLE_BLOCK_SIZE))) % M
        a_scale_ptrs = (a_scales_ptr + offs_asm[:, None] * stride_asm + offs_ks[None, :] * stride_ask)
```
- **EN:** Invokes `tl.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations, random-data generation.
- **CN:** 调用 `tl.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作、随机数据生成。

#### Lines 537-541

```python
    if b_scales_ptr is not None:
        offs_asn = (pid_n *
                    (BLOCK_N // NON_K_PRESHUFFLE_BLOCK_SIZE) + tl.arange(0,
                                                                         (BLOCK_N // NON_K_PRESHUFFLE_BLOCK_SIZE))) % N
        b_scale_ptrs = (b_scales_ptr + offs_asn[:, None] * stride_bsn + offs_ks[None, :] * stride_bsk)
```
- **EN:** Invokes `tl.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

#### Lines 542-542

```python
    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=tl.float32)
```
- **EN:** Prepares or updates state through `accumulator`. Invokes `tl.zeros` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `accumulator` 准备或更新状态。 调用 `tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 543-604

```python

    for k in range(0, tl.cdiv(K, BLOCK_K)):
        if preshuffle:
            # Here we "undo" the shuffle done in global memory (shuffle_scales_cdna4 function).
            if mfma_nonkdim == 32:
                if a_scales_ptr is not None:
                    a_scales = tl.load(a_scale_ptrs).reshape(BLOCK_M // NON_K_PRESHUFFLE_BLOCK_SIZE,
                                                             MX_SCALE_BLOCK_K // 8, 2, 32, 4,
                                                             1).permute(0, 3, 1, 4, 2,
                                                                        5).reshape(BLOCK_M, MX_SCALE_BLOCK_K)
                else:
                    a_scales = None
                if b_scales_ptr is not None:
                    b_scales = tl.load(b_scale_ptrs).reshape(BLOCK_N // NON_K_PRESHUFFLE_BLOCK_SIZE,
                                                             MX_SCALE_BLOCK_K // 8, 2, 32, 4,
                                                             1).permute(0, 3, 1, 4, 2,
                                                                        5).reshape(BLOCK_N, MX_SCALE_BLOCK_K)
                else:
                    b_scales = None
            elif mfma_nonkdim == 16:
                if a_scales_ptr is not None:
                    a_scales = tl.load(a_scale_ptrs).reshape(BLOCK_M // NON_K_PRESHUFFLE_BLOCK_SIZE,
                                                             MX_SCALE_BLOCK_K // 8, 4, 16, 2, 2,
                                                             1).permute(0, 5, 3, 1, 4, 2,
                                                                        6).reshape(BLOCK_M, MX_SCALE_BLOCK_K)
                else:
                    a_scales = None
                if b_scales_ptr is not None:
                    b_scales = tl.load(b_scale_ptrs).reshape(BLOCK_N // NON_K_PRESHUFFLE_BLOCK_SIZE,
                                                             MX_SCALE_BLOCK_K // 8, 4, 16, 2, 2,
                                                             1).permute(0, 5, 3, 1, 4, 2,
                                                                        6).reshape(BLOCK_N, MX_SCALE_BLOCK_K)
                else:
                    b_scales = None
        else:
            if a_scales_ptr is not None:
                a_scales = tl.load(a_scale_ptrs)
            else:
                a_scales = None
            if b_scales_ptr is not None:
                b_scales = tl.load(b_scale_ptrs)
            else:
                b_scales = None

        a = tl.load(a_ptrs)
        b = tl.load(b_ptrs, cache_modifier=None)

        accumulator += tl.dot_scaled(a, a_scales, DTYPE_A, b, b_scales, DTYPE_B, fast_math=fast_math)

        # Advance the ptrs to the next K block.
        a_ptrs += (BLOCK_K // PACK_FACTOR_A) * stride_ak
        b_ptrs += (BLOCK_K // PACK_FACTOR_B) * stride_bk
        if preshuffle:
            if a_scales_ptr is not None:
                a_scale_ptrs += BLOCK_K * stride_ask
            if b_scales_ptr is not None:
                b_scale_ptrs += BLOCK_K * stride_bsk
        else:
            if a_scales_ptr is not None:
                a_scale_ptrs += MX_SCALE_BLOCK_K * stride_ask
            if b_scales_ptr is not None:
                b_scale_ptrs += MX_SCALE_BLOCK_K * stride_bsk
```
- **EN:** Invokes `tl.cdiv`, `tl.load`, `tl.dot_scaled` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, cache management behavior.
- **CN:** 调用 `tl.cdiv`、`tl.load`、`tl.dot_scaled` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、缓存管理行为。

#### Lines 605-614

```python

    c = accumulator.to(c_ptr.type.element_ty)

    # Write back the block of the output matrix C with masks.
    offs_cm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M).to(tl.int64)
    offs_cn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N).to(tl.int64)
    c_ptrs = (c_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :])
    c_mask = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)

    tl.store(c_ptrs, c, mask=c_mask, cache_modifier=".wt")
```
- **EN:** Prepares or updates state through `c`, `offs_cm`, `offs_cn`, `c_ptrs`, `c_mask`. Invokes `accumulator.to`, `tl.arange`, `tl.store` to execute the test logic. Relevant themes: Triton language operations, cache management behavior.
- **CN:** 通过 `c`、`offs_cm`、`offs_cn`、`c_ptrs`、`c_mask` 准备或更新状态。 调用 `accumulator.to`、`tl.arange`、`tl.store` 执行测试逻辑。 相关主题：Triton language 操作、缓存管理行为。

### Lines 615-630

```python


@pytest.mark.interpreter
@pytest.mark.parametrize("M, N, K", [(1024, 1024, 1024)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(128, 128, 256), (64, 64, 512), [32, 32, 64]])
@pytest.mark.parametrize("DTYPE_A, DTYPE_B, FAST_MATH", [("mxfp4", "mxfp4", False), ("fp16", "mxfp8e5", False),
                                                         ("mxfp8e4", "bf16", False), ("bf16", "mxfp4", True)])
@pytest.mark.parametrize("mfma_nonkdim", [16, 32])
@pytest.mark.parametrize("preshuffle", [True, False])
@pytest.mark.skipif(is_cuda() and torch.cuda.get_device_capability()[0] in [10, 11],
                    reason="Compilation bug for GB200.")
@pytest.mark.skipif(is_hip() and not (is_hip_cdna4() or is_hip_gfx1250()),
                    reason="Scaled dot is not emulated on other archs yet.")
def test_preshuffle_scale_mxfp_cdna4(M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, DTYPE_A, DTYPE_B, FAST_MATH, mfma_nonkdim,
                                     preshuffle, device):
    # For details about scale shuffling on AMD GPUs please take a look at documentation in 10-block-scaled-matmu.py.
```
- **EN:** Defines the test function `test_preshuffle_scale_mxfp_cdna4`. Decorators: `pytest.mark.interpreter`, `pytest.mark.parametrize('M, N, K', [(1024, 1024, 1024)])`, `pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 256), (64, 64, 512), [32, 32, 64]])`, `pytest.mark.parametrize('DTYPE_A, DTYPE_B, FAST_MATH', [('mxfp4', 'mxfp4', False), ('fp16', 'mxfp8e5', False), ('mxfp8e4', 'bf16', False), ('bf16', 'mxfp4', True)])`, `pytest.mark.parametrize('mfma_nonkdim', [16, 32])`, `pytest.mark.parametrize('preshuffle', [True, False])`, and 2 more. Parameters: `M`, `N`, `K`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `DTYPE_A`, `DTYPE_B`, and 4 more. Nested definitions in this scope: `shuffle_scales_cdna4`, `e8m0_to_f32`, `run_torch`, `generate_gemm_input`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `generate_gemm_input`, `run_torch`, `torch.empty`, `is_hip`, and 33 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_preshuffle_scale_mxfp_cdna4`。 装饰器：`pytest.mark.interpreter`、`pytest.mark.parametrize('M, N, K', [(1024, 1024, 1024)])`、`pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 256), (64, 64, 512), [32, 32, 64]])`、`pytest.mark.parametrize('DTYPE_A, DTYPE_B, FAST_MATH', [('mxfp4', 'mxfp4', False), ('fp16', 'mxfp8e5', False), ('mxfp8e4', 'bf16', False), ('bf16', 'mxfp4', True)])`、`pytest.mark.parametrize('mfma_nonkdim', [16, 32])`、`pytest.mark.parametrize('preshuffle', [True, False])` 等另外 2 项。 参数：`M`、`N`、`K`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`DTYPE_A`、`DTYPE_B` 等另外 4 项。 该作用域中的嵌套定义：`shuffle_scales_cdna4`、`e8m0_to_f32`、`run_torch`、`generate_gemm_input`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`generate_gemm_input`、`run_torch`、`torch.empty`、`is_hip` 等另外 33 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 631-632

```python
    if preshuffle and (BLOCK_M < 32 or BLOCK_N < 32 or BLOCK_K < 256):
        pytest.skip("Minimal tile size for preshuffling is 32x32x256")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 633-635

```python

    if not (DTYPE_A.startswith("mx") or DTYPE_B.startswith("mx")):
        pytest.skip("Requires at least 1 microscaling operand")
```
- **EN:** Invokes `pytest.skip`, `DTYPE_A.startswith`, `DTYPE_B.startswith` to execute the test logic. Branches on runtime or test conditions. Relevant themes: random-data generation.
- **CN:** 调用 `pytest.skip`、`DTYPE_A.startswith`、`DTYPE_B.startswith` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：随机数据生成。

#### Lines 636-638

```python

    if is_cuda() and (DTYPE_A == "mxfp8e4" or DTYPE_B == "mxfp8e4"):
        pytest.skip("Skip fp8e4 on NV backend")
```
- **EN:** Invokes `is_cuda`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_cuda`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 639-640

```python

    def shuffle_scales_cdna4(scales: torch.Tensor):
```
- **EN:** Defines the helper function `shuffle_scales_cdna4`. Parameters: `scales`. Key calls include `scales.clone`, `scales_shuffled.view`, `scales_shuffled.permute`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `shuffle_scales_cdna4`。 参数：`scales`。 关键调用包括 `scales.clone`、`scales_shuffled.view`、`scales_shuffled.permute`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 641-642

```python
        if not preshuffle:
            return scales
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

##### Lines 643-646

```python

        scales_shuffled = scales.clone()

        sm, sn = scales_shuffled.shape
```
- **EN:** Prepares or updates state through `scales_shuffled`, `sm`, `sn`. Invokes `scales.clone` to execute the test logic.
- **CN:** 通过 `scales_shuffled`、`sm`、`sn` 准备或更新状态。 调用 `scales.clone` 执行测试逻辑。

##### Lines 647-652

```python
        if mfma_nonkdim == 32:
            scales_shuffled = scales_shuffled.view(sm // 32, 32, sn // 8, 4, 2, 1)
            scales_shuffled = scales_shuffled.permute(0, 2, 4, 1, 3, 5).contiguous()
        elif mfma_nonkdim == 16:
            scales_shuffled = scales_shuffled.view(sm // 32, 2, 16, sn // 8, 2, 4, 1)
            scales_shuffled = scales_shuffled.permute(0, 3, 5, 2, 4, 1, 6).contiguous()
```
- **EN:** Invokes `scales_shuffled.view`, `scales_shuffled.permute` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `scales_shuffled.view`、`scales_shuffled.permute` 执行测试逻辑。 根据运行时或测试条件进行分支。

##### Lines 653-655

```python

        scales_shuffled = scales_shuffled.view(sm // 32, sn * 32)
        return scales_shuffled
```
- **EN:** Prepares or updates state through `scales_shuffled`. Invokes `scales_shuffled.view` to execute the test logic.
- **CN:** 通过 `scales_shuffled` 准备或更新状态。 调用 `scales_shuffled.view` 执行测试逻辑。

#### Lines 656-657

```python

    def e8m0_to_f32(x):
```
- **EN:** Defines the helper function `e8m0_to_f32`. Parameters: `x`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `e8m0_to_f32`。 参数：`x`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 658-660

```python
        x_f32 = 2**((x - 127).to(torch.float32))
        x_f32[x_f32 == 128] = float("nan")
        return x_f32
```
- **EN:** Prepares or updates state through `x_f32`. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `x_f32` 准备或更新状态。 相关主题：PyTorch 张量准备与校验。

#### Lines 661-663

```python

    def run_torch(x, w, x_scales, w_scales, dtype):
        # First convert the x and w inputs to f32.
```
- **EN:** Defines the helper function `run_torch`. Parameters: `x`, `w`, `x_scales`, `w_scales`, `dtype`. Key calls include `x.to`, `w.to`, `e8m0_to_f32`, `torch.mm`, `x_scales.repeat_interleave`, `w_scales.repeat_interleave`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `run_torch`。 参数：`x`、`w`、`x_scales`、`w_scales`、`dtype`。 关键调用包括 `x.to`、`w.to`、`e8m0_to_f32`、`torch.mm`、`x_scales.repeat_interleave`、`w_scales.repeat_interleave`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 664-666

```python
        SCALE_GROUP_SIZE = 32
        x_f32 = x.to(torch.float32)
        w_f32 = w.to(torch.float32)
```
- **EN:** Prepares or updates state through `SCALE_GROUP_SIZE`, `x_f32`, `w_f32`. Invokes `x.to`, `w.to` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `SCALE_GROUP_SIZE`、`x_f32`、`w_f32` 准备或更新状态。 调用 `x.to`、`w.to` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

##### Lines 667-671

```python
        # Next convert the e8m0 scales to f32.
        if x_scales is not None:
            x_scales = x_scales.repeat_interleave(SCALE_GROUP_SIZE, dim=1).to(torch.float32)
            x_scales_f32 = e8m0_to_f32(x_scales)
            x_f32 = x_f32 * x_scales_f32
```
- **EN:** Invokes `e8m0_to_f32`, `x_scales.repeat_interleave` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `e8m0_to_f32`、`x_scales.repeat_interleave` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

##### Lines 672-675

```python
        if w_scales is not None:
            w_scales = w_scales.repeat_interleave(SCALE_GROUP_SIZE, dim=1).to(torch.float32)
            w_scales_f32 = e8m0_to_f32(w_scales)
            w_f32 = w_f32 * w_scales_f32
```
- **EN:** Invokes `e8m0_to_f32`, `w_scales.repeat_interleave` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `e8m0_to_f32`、`w_scales.repeat_interleave` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

##### Lines 676-676

```python
        return torch.mm(x_f32, w_f32.T).to(dtype)
```
- **EN:** Invokes `torch.mm` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.mm` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 677-682

```python

    dtype_to_torch_type = {
        "fp16": torch.half, "bf16": torch.bfloat16, "mxfp8e5": torch.float8_e5m2, "mxfp8e4": torch.float8_e4m3fn
    }

    dtype_to_triton_type = {"fp16": "fp16", "bf16": "bf16", "mxfp8e5": "e5m2", "mxfp8e4": "e4m3", "mxfp4": "e2m1"}
```
- **EN:** Prepares or updates state through `dtype_to_torch_type`, `dtype_to_triton_type`. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `dtype_to_torch_type`、`dtype_to_triton_type` 准备或更新状态。 相关主题：PyTorch 张量准备与校验。

#### Lines 683-684

```python

    def generate_gemm_input(dim0, dim1, dtype):
```
- **EN:** Defines the helper function `generate_gemm_input`. Parameters: `dim0`, `dim1`, `dtype`. Key calls include `torch.manual_seed`, `dtype.startswith`, `torch.randint`, `shuffle_scales_cdna4`, `MXFP4Tensor`, `torch.randn`, and 1 more. This scope touches PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 定义辅助函数 `generate_gemm_input`。 参数：`dim0`、`dim1`、`dtype`。 关键调用包括 `torch.manual_seed`、`dtype.startswith`、`torch.randint`、`shuffle_scales_cdna4`、`MXFP4Tensor`、`torch.randn` 等另外 1 项。 该作用域涉及PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

##### Lines 685-686

```python
        torch.manual_seed(5)
        SCALE_GROUP_SIZE = 32
```
- **EN:** Prepares or updates state through `SCALE_GROUP_SIZE`. Invokes `torch.manual_seed` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `SCALE_GROUP_SIZE` 准备或更新状态。 调用 `torch.manual_seed` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

##### Lines 687-697

```python

        if dtype == "mxfp4":
            v = MXFP4Tensor(size=(dim0, dim1), device="cuda").random()
        elif dtype == "mxfp8e5":
            v = torch.randint(20, 40, (dim0, dim1), dtype=torch.uint8).view(torch.float8_e5m2).to(device)
        elif dtype == "mxfp8e4":
            v = torch.randint(20, 40, (dim0, dim1), dtype=torch.uint8).view(torch.float8_e4m3fn).to(device)
        elif dtype in ("fp16", "bf16"):
            v = torch.randn((dim0, dim1), device=device, dtype=dtype_to_torch_type[dtype])
        else:
            raise ValueError(f"Unsupported data type: {dtype}")
```
- **EN:** Invokes `MXFP4Tensor`, `torch.randn`, `ValueError`, `torch.randint` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 调用 `MXFP4Tensor`、`torch.randn`、`ValueError`、`torch.randint` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、随机数据生成。

##### Lines 698-704

```python

        if dtype.startswith("mx"):
            scales = torch.randint(124, 128, (dim0, dim1 // SCALE_GROUP_SIZE), dtype=torch.uint8, device=device)
            scales_shuffled = shuffle_scales_cdna4(scales)
        else:
            scales = None
            scales_shuffled = None
```
- **EN:** Invokes `dtype.startswith`, `torch.randint`, `shuffle_scales_cdna4` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 调用 `dtype.startswith`、`torch.randint`、`shuffle_scales_cdna4` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、随机数据生成。

##### Lines 705-706

```python

        return (v, scales, scales_shuffled)
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

#### Lines 707-711

```python

    x, x_scales, x_scales_triton = generate_gemm_input(M, K, DTYPE_A)
    w, w_scales, w_scales_triton = generate_gemm_input(N, K, DTYPE_B)

    torch_out = run_torch(x, w, x_scales, w_scales, torch.float32)
```
- **EN:** Prepares or updates state through `x`, `x_scales`, `x_scales_triton`, `w`, `w_scales`, `w_scales_triton`, `torch_out`. Invokes `generate_gemm_input`, `run_torch` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 通过 `x`、`x_scales`、`x_scales_triton`、`w`、`w_scales`、`w_scales_triton`、`torch_out` 准备或更新状态。 调用 `generate_gemm_input`、`run_torch` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 712-714

```python

    if DTYPE_A == "mxfp4":
        x = x.to_packed_tensor(dim=1)
```
- **EN:** Invokes `x.to_packed_tensor` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `x.to_packed_tensor` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 715-717

```python

    if DTYPE_B == "mxfp4":
        w = w.to_packed_tensor(dim=1)
```
- **EN:** Invokes `w.to_packed_tensor` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `w.to_packed_tensor` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 718-725

```python

    w = w.T
    triton_out = torch.empty((M, N), device=x.device)

    x_scales_strides = x_scales_triton.stride() if x_scales is not None else (None, None)
    w_scales_strides = w_scales_triton.stride() if w_scales is not None else (None, None)

    kernel_kwargs = {}
```
- **EN:** Prepares or updates state through `w`, `triton_out`, `x_scales_strides`, `w_scales_strides`, `kernel_kwargs`. Invokes `torch.empty`, `x_scales_triton.stride`, `w_scales_triton.stride` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `w`、`triton_out`、`x_scales_strides`、`w_scales_strides`、`kernel_kwargs` 准备或更新状态。 调用 `torch.empty`、`x_scales_triton.stride`、`w_scales_triton.stride` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 726-727

```python
    if is_hip():
        kernel_kwargs["matrix_instr_nonkdim"] = mfma_nonkdim
```
- **EN:** Invokes `is_hip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 728-738

```python

    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), 1)
    k = _gemm_kernel_preshuffled_scales_cdna4[grid](x, w, triton_out, x_scales_triton, w_scales_triton, M, N, K,
                                                    x.stride(0), x.stride(1), w.stride(0), w.stride(1),
                                                    triton_out.stride(0), triton_out.stride(1), *x_scales_strides,
                                                    *w_scales_strides, dtype_to_triton_type[DTYPE_A],
                                                    dtype_to_triton_type[DTYPE_B], BLOCK_M, BLOCK_N, BLOCK_K,
                                                    mfma_nonkdim, preshuffle, fast_math=FAST_MATH, num_warps=8,
                                                    num_stages=1, **kernel_kwargs)
    triton_out = triton_out.to(torch.float32)
    torch.testing.assert_close(torch_out, triton_out, atol=2e-5, rtol=1e-4)
```
- **EN:** Prepares or updates state through `grid`, `k`, `triton_out`. Invokes `triton.cdiv`, `x.stride`, `w.stride`, `triton_out.stride`, `triton_out.to`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 通过 `grid`、`k`、`triton_out` 准备或更新状态。 调用 `triton.cdiv`、`x.stride`、`w.stride`、`triton_out.stride`、`triton_out.to`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 739-744

```python
    if is_hip_cdna4() and preshuffle:
        assert "ds_read_u8" not in k.asm["amdgcn"]
        if mfma_nonkdim == 16:
            assert "tilesPerWarp = [2, 2]" in k.asm["ttgir"]
        elif mfma_nonkdim == 32:  # default tilesPerWarp = [1, 1]
            assert "tilesPerWarp" not in k.asm["ttgir"]
```
- **EN:** Invokes `is_hip_cdna4` to execute the test logic. Validates behavior with 3 assertion(s). Branches on runtime or test conditions.
- **CN:** 调用 `is_hip_cdna4` 执行测试逻辑。 通过 3 个断言验证行为。 根据运行时或测试条件进行分支。

### Lines 745-754

```python


@pytest.mark.interpreter
@pytest.mark.parametrize("M, N, K", [(1024, 512, 512), (998, 111, 512), (63, 128, 512)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(128, 128, 128), (256, 128, 128), (128, 256, 128),
                                                       (128, 128, 256), (128, 256, 256)])
@pytest.mark.parametrize("NUM_STAGES", [1, 2, 4])
@pytest.mark.parametrize("USE_2D_SCALE_LOAD", [False, True])
@pytest.mark.skipif(is_hip() or torch.cuda.get_device_capability()[0] != 10, reason="Requires compute capability == 10")
def test_blocked_scale_mxfp(M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, NUM_STAGES, USE_2D_SCALE_LOAD, device):
```
- **EN:** Defines the test function `test_blocked_scale_mxfp`. Decorators: `pytest.mark.interpreter`, `pytest.mark.parametrize('M, N, K', [(1024, 512, 512), (998, 111, 512), (63, 128, 512)])`, `pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 128), (256, 128, 128), (128, 256, 128), (128, 128, 256), (128, 256, 256)])`, `pytest.mark.parametrize('NUM_STAGES', [1, 2, 4])`, `pytest.mark.parametrize('USE_2D_SCALE_LOAD', [False, True])`, `pytest.mark.skipif(is_hip() or torch.cuda.get_device_capability()[0] != 10, reason='Requires compute capability == 10')`. Parameters: `M`, `N`, `K`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `NUM_STAGES`, `USE_2D_SCALE_LOAD`, and 1 more. Nested definitions in this scope: `flatten_scale`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `torch.manual_seed`, `f8_to_f16`, `getattr`, `torch.empty`, and 21 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_blocked_scale_mxfp`。 装饰器：`pytest.mark.interpreter`、`pytest.mark.parametrize('M, N, K', [(1024, 512, 512), (998, 111, 512), (63, 128, 512)])`、`pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 128), (256, 128, 128), (128, 256, 128), (128, 128, 256), (128, 256, 256)])`、`pytest.mark.parametrize('NUM_STAGES', [1, 2, 4])`、`pytest.mark.parametrize('USE_2D_SCALE_LOAD', [False, True])`、`pytest.mark.skipif(is_hip() or torch.cuda.get_device_capability()[0] != 10, reason='Requires compute capability == 10')`。 参数：`M`、`N`、`K`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`NUM_STAGES`、`USE_2D_SCALE_LOAD` 等另外 1 项。 该作用域中的嵌套定义：`flatten_scale`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`torch.manual_seed`、`f8_to_f16`、`getattr`、`torch.empty` 等另外 21 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 755-758

```python
    if BLOCK_N == 256 and BLOCK_K == 256:
        NUM_STAGES = min(NUM_STAGES, 2)
    elif BLOCK_K == 256:
        NUM_STAGES = min(NUM_STAGES, 3)
```
- **EN:** Invokes `min` to execute the test logic. Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `min` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

#### Lines 759-780

```python
    # since the block size are big we use num_warps = 8 to avoid pressure problems.
    num_warps = 8
    torch.manual_seed(42)
    dtype_src_str = "float8e5"
    dtype_dst_str = "float32"
    a = torch.randint(20, 40, (M, K), dtype=torch.uint8, device=device).view(torch.float8_e5m2)
    A = f8_to_f16(a, dtype_src_str)
    b = torch.randint(20, 40, (K, N), dtype=torch.uint8, device=device).view(torch.float8_e5m2)
    B = f8_to_f16(b, dtype_src_str)
    ceildiv = lambda a, b: math.ceil(a / b)
    a_scale = torch.randint(130, (ceildiv(M, 128), ceildiv(K, 128), 32, 4, 4), dtype=torch.uint8).to(device)
    b_scale = torch.randint(130, (ceildiv(N, 128), ceildiv(K, 128), 32, 4, 4), dtype=torch.uint8).to(device)

    dtype_dst = getattr(torch, dtype_dst_str)
    output = torch.empty((M, N), dtype=dtype_dst, device=device)
    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), 1)
    out = block_scale_mxfp_matmul[grid](a, b, output, a_scale, b_scale, M, N, K, a_scale.stride(0), a_scale.stride(1),
                                        a_scale.stride(2), a_scale.stride(3), a.stride(0), a.stride(1), b.stride(0),
                                        b.stride(1), output.stride(0), output.stride(1), BLOCK_M, BLOCK_N, BLOCK_K,
                                        NUM_STAGES=NUM_STAGES, USE_2D_SCALE_LOAD=USE_2D_SCALE_LOAD, num_warps=num_warps)
    ttgir = out.asm["ttgir"]
    ptx = out.asm["ptx"]
```
- **EN:** Prepares or updates state through `num_warps`, `dtype_src_str`, `dtype_dst_str`, `a`, `A`, `b`, `B`, `ceildiv`, and 8 more. Invokes `torch.manual_seed`, `torch.randint`, `f8_to_f16`, `math.ceil`, `ceildiv`, `getattr`, and 6 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points, random-data generation.
- **CN:** 通过 `num_warps`、`dtype_src_str`、`dtype_dst_str`、`a`、`A`、`b`、`B`、`ceildiv` 等另外 8 项 准备或更新状态。 调用 `torch.manual_seed`、`torch.randint`、`f8_to_f16`、`math.ceil`、`ceildiv`、`getattr` 等另外 6 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点、随机数据生成。

#### Lines 781-782

```python

    def flatten_scale(scale):
```
- **EN:** Defines the helper function `flatten_scale`. Parameters: `scale`. Key calls include `scale.permute`.
- **CN:** 定义辅助函数 `flatten_scale`。 参数：`scale`。 关键调用包括 `scale.permute`。

##### Lines 783-784

```python
        num_chunk_m, num_chunk_k, _, _, _ = scale.shape
        return scale.permute(0, 3, 2, 1, 4).reshape(num_chunk_m * 128, num_chunk_k * 4).contiguous()
```
- **EN:** Prepares or updates state through `num_chunk_m`, `num_chunk_k`, `_`. Invokes `scale.permute` to execute the test logic.
- **CN:** 通过 `num_chunk_m`、`num_chunk_k`、`_` 准备或更新状态。 调用 `scale.permute` 执行测试逻辑。

#### Lines 785-800

```python

    a_scale_f32 = flatten_scale(fp8e8m0_to_float32(a_scale))[:M]
    b_scale_f32 = flatten_scale(fp8e8m0_to_float32(b_scale))[:N]
    a_scale_f32 = a_scale_f32.repeat_interleave(32, dim=1)
    b_scale_f32 = b_scale_f32.repeat_interleave(32, dim=1)

    # b_scales are always col major
    b_scale_f32 = b_scale_f32.T.contiguous()

    a = A * a_scale_f32
    b = B * b_scale_f32
    ref_out = torch.matmul(a, b).to(torch.float32)
    output = output.to(torch.float32)
    atol = 0.0001
    rtol = 0.0001
    torch.testing.assert_close(ref_out, output, atol=atol, rtol=rtol)
```
- **EN:** Prepares or updates state through `a_scale_f32`, `b_scale_f32`, `a`, `b`, `ref_out`, `output`, `atol`, `rtol`. Invokes `flatten_scale`, `fp8e8m0_to_float32`, `a_scale_f32.repeat_interleave`, `b_scale_f32.repeat_interleave`, `b_scale_f32.T.contiguous`, `torch.matmul`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 通过 `a_scale_f32`、`b_scale_f32`、`a`、`b`、`ref_out`、`output`、`atol`、`rtol` 准备或更新状态。 调用 `flatten_scale`、`fp8e8m0_to_float32`、`a_scale_f32.repeat_interleave`、`b_scale_f32.repeat_interleave`、`b_scale_f32.T.contiguous`、`torch.matmul` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 801-805

```python

    if USE_2D_SCALE_LOAD:
        # Due to an issue in the coalescing pass, tmem_copy can not be generated for the 5D load.
        # The issue is fixed using the patch from https://github.com/triton-lang/triton/pull/4914
        assert "tcgen05.cp" in ptx
```
- **EN:** Validates behavior with 1 assertion(s). Branches on runtime or test conditions.
- **CN:** 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 806-820

```python
    if NUM_STAGES > 1:
        if BLOCK_M == BLOCK_K and BLOCK_N == BLOCK_K:
            load_pipelined = ttgir.count(f"ttg.local_alloc : () -> !ttg.memdesc<{NUM_STAGES}x{BLOCK_M}x{BLOCK_K}") == 2
        else:
            load_pipelined = (ttgir.count(f"ttg.local_alloc : () -> !ttg.memdesc<{NUM_STAGES}x{BLOCK_M}x{BLOCK_K}")
                              and ttgir.count(f"ttg.local_alloc : () -> !ttg.memdesc<{NUM_STAGES}x{BLOCK_K}x{BLOCK_N}"))

        if load_pipelined and USE_2D_SCALE_LOAD:
            # If load is pipelined and tmem_copy is used,  MMA pipelining should also kick in
            assert "ttng.wait_barrier" in ttgir
        elif not load_pipelined:
            # The behavior of load pipelining seems to depend on the size of input tensors.
            # In this test, it fails to pipeline the RHS tensor when N is not a multiple of 128. Pipelining of the LHS tensor
            # does not seem to be affected by the value of M, though.
            print(f"SWP failed for M = {M}, N = {N}")
```
- **EN:** Invokes `ttgir.count` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points, debugging and inspection paths.
- **CN:** 调用 `ttgir.count` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点、调试与检查路径。

### Lines 821-828

```python


@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(128, 128, 64), (128, 64, 128), (64, 128, 32), (128, 256, 32),
                                                       (256, 64, 32)])
@pytest.mark.parametrize("a_trans", [False, True])
@pytest.mark.parametrize("dtype_src_str", ["float32", "float16", "float8e5"])
@pytest.mark.skipif(is_hip() or torch.cuda.get_device_capability()[0] != 10, reason="Requires compute capability == 10")
def test_lhs_in_tmem(BLOCK_M, BLOCK_N, BLOCK_K, a_trans, dtype_src_str, device, monkeypatch):
```
- **EN:** Defines the test function `test_lhs_in_tmem`. Decorators: `pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 64), (128, 64, 128), (64, 128, 32), (128, 256, 32), (256, 64, 32)])`, `pytest.mark.parametrize('a_trans', [False, True])`, `pytest.mark.parametrize('dtype_src_str', ['float32', 'float16', 'float8e5'])`, `pytest.mark.skipif(is_hip() or torch.cuda.get_device_capability()[0] != 10, reason='Requires compute capability == 10')`. Parameters: `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `a_trans`, `dtype_src_str`, `device`, `monkeypatch`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `_knob_promote_lhs_to_tmem`, `torch.manual_seed`, `torch.empty`, `torch.testing.assert_close`, and 13 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_lhs_in_tmem`。 装饰器：`pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 64), (128, 64, 128), (64, 128, 32), (128, 256, 32), (256, 64, 32)])`、`pytest.mark.parametrize('a_trans', [False, True])`、`pytest.mark.parametrize('dtype_src_str', ['float32', 'float16', 'float8e5'])`、`pytest.mark.skipif(is_hip() or torch.cuda.get_device_capability()[0] != 10, reason='Requires compute capability == 10')`。 参数：`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`a_trans`、`dtype_src_str`、`device`、`monkeypatch`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`_knob_promote_lhs_to_tmem`、`torch.manual_seed`、`torch.empty`、`torch.testing.assert_close` 等另外 13 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 829-833

```python
    M = 1024
    N = 512
    K = 256
    _knob_promote_lhs_to_tmem(monkeypatch)
    torch.manual_seed(42)
```
- **EN:** Prepares or updates state through `M`, `N`, `K`. Invokes `_knob_promote_lhs_to_tmem`, `torch.manual_seed` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `M`、`N`、`K` 准备或更新状态。 调用 `_knob_promote_lhs_to_tmem`、`torch.manual_seed` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 834-848

```python
    if dtype_src_str == "float8e5":
        a = torch.randint(20, 40, (M, K), dtype=torch.int8, device=device).view(torch.float8_e5m2)
        b = torch.randint(20, 40, (K, N), dtype=torch.int8, device=device).view(torch.float8_e5m2)
        if a_trans:
            a = a.T.contiguous().T
        A = f8_to_f16(a, dtype_src_str)
        B = f8_to_f16(b, dtype_src_str)
    else:
        dtype_src = getattr(torch, dtype_src_str)
        a = torch.randn(M, K, dtype=dtype_src, device=device)
        b = torch.randn(K, N, dtype=dtype_src, device=device)
        if a_trans:
            a = a.T.contiguous().T
        A = a
        B = b
```
- **EN:** Invokes `f8_to_f16`, `getattr`, `torch.randn`, `torch.randint`, `a.T.contiguous` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 调用 `f8_to_f16`、`getattr`、`torch.randn`、`torch.randint`、`a.T.contiguous` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 849-859

```python
    output = torch.empty((M, N), dtype=torch.float32, device=device)
    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), 1)
    k = matmul_kernel[grid](a, b, output, M, N, K, a.stride(0), a.stride(1), b.stride(0), b.stride(1), output.stride(0),
                            output.stride(1), BLOCK_M, BLOCK_N, BLOCK_K, NUM_STAGES=1, SCALE_A=None, PRECISION="tf32",
                            A_TRANS=a_trans)
    ref_out = torch.matmul(A, B).to(torch.float32)
    atol = 0.03
    rtol = 0.03
    torch.testing.assert_close(ref_out, output, atol=atol, rtol=rtol)
    pattern = r"%\w+\s*=\s*ttng\.tmem_alloc[\s\S]*?tng\.tc_gen5_mma\s+%\w+,"
    ttgir = k.asm["ttgir"]
```
- **EN:** Prepares or updates state through `output`, `grid`, `k`, `ref_out`, `atol`, `rtol`, `pattern`, `ttgir`. Invokes `torch.empty`, `triton.cdiv`, `a.stride`, `b.stride`, `output.stride`, `torch.matmul`, and 1 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 通过 `output`、`grid`、`k`、`ref_out`、`atol`、`rtol`、`pattern`、`ttgir` 准备或更新状态。 调用 `torch.empty`、`triton.cdiv`、`a.stride`、`b.stride`、`output.stride`、`torch.matmul` 等另外 1 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 860-860

```python
    assert re.search(pattern, ttgir)
```
- **EN:** Invokes `re.search` to execute the test logic. Validates behavior with 1 assertion(s).
- **CN:** 调用 `re.search` 执行测试逻辑。 通过 1 个断言验证行为。

### Lines 861-871

```python


@triton.jit
def lhs_in_tmem_kernel_mxfp(  #
        a_ptr, b_ptr, output_ptr,  #
        a_scale, b_scale,  #
        stride_scale,  #
        stride_am, stride_ak,  #
        stride_bk, stride_bn,  #
        stride_cm, stride_cn,  #
        M: tl.constexpr, N: tl.constexpr, K: tl.constexpr):
```
- **EN:** Defines the helper function `lhs_in_tmem_kernel_mxfp`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `output_ptr`, `a_scale`, `b_scale`, `stride_scale`, `stride_am`, `stride_ak`, and 7 more. Key calls include `tl.arange`, `tl.load`, `tl.dot_scaled`, `tl.store`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `lhs_in_tmem_kernel_mxfp`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`output_ptr`、`a_scale`、`b_scale`、`stride_scale`、`stride_am`、`stride_ak` 等另外 7 项。 关键调用包括 `tl.arange`、`tl.load`、`tl.dot_scaled`、`tl.store`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 872-888

```python
    offs_am = tl.arange(0, M)
    offs_bn = tl.arange(0, N)
    offs_k = tl.arange(0, K)
    offs_scale_k = tl.arange(0, K // 32)
    a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_k[None, :] * stride_ak)
    b_ptrs = b_ptr + (offs_k[:, None] * stride_bk + offs_bn[None, :] * stride_bn)
    a_scale_ptr = a_scale + offs_am[:, None] * stride_scale + offs_scale_k[None, :]
    b_scale_ptr = b_scale + offs_bn[:, None] * stride_scale + offs_scale_k[None, :]
    a = tl.load(a_ptrs)
    b = tl.load(b_ptrs)
    scale_a = tl.load(a_scale_ptr)
    scale_b = tl.load(b_scale_ptr)
    accumulator = tl.dot_scaled(a, scale_a, "e5m2", b, scale_b, "e5m2")
    offs_cm = tl.arange(0, M)
    offs_cn = tl.arange(0, N)
    output_ptrs = output_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
    tl.store(output_ptrs, accumulator)
```
- **EN:** Prepares or updates state through `offs_am`, `offs_bn`, `offs_k`, `offs_scale_k`, `a_ptrs`, `b_ptrs`, `a_scale_ptr`, `b_scale_ptr`, and 8 more. Invokes `tl.arange`, `tl.load`, `tl.dot_scaled`, `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `offs_am`、`offs_bn`、`offs_k`、`offs_scale_k`、`a_ptrs`、`b_ptrs`、`a_scale_ptr`、`b_scale_ptr` 等另外 8 项 准备或更新状态。 调用 `tl.arange`、`tl.load`、`tl.dot_scaled`、`tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 889-893

```python


@pytest.mark.interpreter
@pytest.mark.skipif(is_hip() or torch.cuda.get_device_capability()[0] != 10, reason="Requires compute capability == 10")
def test_lhs_in_tmem_mxfp(device, monkeypatch):
```
- **EN:** Defines the test function `test_lhs_in_tmem_mxfp`. Decorators: `pytest.mark.interpreter`, `pytest.mark.skipif(is_hip() or torch.cuda.get_device_capability()[0] != 10, reason='Requires compute capability == 10')`. Parameters: `device`, `monkeypatch`. Key calls include `pytest.mark.skipif`, `_knob_promote_lhs_to_tmem`, `torch.manual_seed`, `torch.randint`, `f8_to_f16`, `torch.empty`, and 12 more. This scope touches PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 定义测试函数 `test_lhs_in_tmem_mxfp`。 装饰器：`pytest.mark.interpreter`、`pytest.mark.skipif(is_hip() or torch.cuda.get_device_capability()[0] != 10, reason='Requires compute capability == 10')`。 参数：`device`、`monkeypatch`。 关键调用包括 `pytest.mark.skipif`、`_knob_promote_lhs_to_tmem`、`torch.manual_seed`、`torch.randint`、`f8_to_f16`、`torch.empty` 等另外 12 项。 该作用域涉及PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

#### Lines 894-920

```python
    _knob_promote_lhs_to_tmem(monkeypatch)
    M, N, K = 128, 64, 32
    torch.manual_seed(42)
    a = torch.randint(20, 40, (M, K), dtype=torch.uint8, device=device)
    b = torch.randint(20, 40, (K, N), dtype=torch.uint8, device=device)
    A = f8_to_f16(a, "float8e5")
    B = f8_to_f16(b, "float8e5")
    a_scale = torch.randint(124, 130, (M, K // 32), dtype=torch.uint8, device=device)
    b_scale = torch.randint(124, 130, (N, K // 32), dtype=torch.uint8, device=device)
    output = torch.empty((M, N), dtype=torch.float16, device=device)
    grid = (1, 1)
    lhs_in_tmem_kernel_mxfp[grid](a, b, output, a_scale, b_scale, a_scale.stride(0), a.stride(0), a.stride(1),
                                  b.stride(0), b.stride(1), output.stride(0), output.stride(1), M, N, K)
    a_scale_f32 = fp8e8m0_to_float32(a_scale)
    b_scale_f32 = fp8e8m0_to_float32(b_scale)
    a_scale_f32 = a_scale_f32.repeat_interleave(32, dim=1)
    b_scale_f32 = b_scale_f32.repeat_interleave(32, dim=1)

    # b_scales are always col major
    b_scale_f32 = b_scale_f32.T.contiguous()

    a = A * a_scale_f32
    b = B * b_scale_f32
    ref_out = torch.matmul(a, b).to(torch.float16)
    atol = 0.003
    rtol = 0.003
    torch.testing.assert_close(ref_out, output, atol=atol, rtol=rtol)
```
- **EN:** Prepares or updates state through `M`, `N`, `K`, `a`, `b`, `A`, `B`, `a_scale`, and 8 more. Invokes `_knob_promote_lhs_to_tmem`, `torch.manual_seed`, `torch.randint`, `f8_to_f16`, `torch.empty`, `a_scale.stride`, and 9 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 通过 `M`、`N`、`K`、`a`、`b`、`A`、`B`、`a_scale` 等另外 8 项 准备或更新状态。 调用 `_knob_promote_lhs_to_tmem`、`torch.manual_seed`、`torch.randint`、`f8_to_f16`、`torch.empty`、`a_scale.stride` 等另外 9 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

### Lines 921-936

```python


@triton.jit
def block_scale_fp4_matmul(  #
        a_ptr, b_ptr, output_ptr,  #
        a_scale, b_scale,  #
        M, N, K,  #
        stride_scale,  #
        stride_am, stride_ak,  #
        stride_bk, stride_bn,  #
        stride_cm, stride_cn,  #
        VEC_SIZE: tl.constexpr,  #
        BLOCK_M: tl.constexpr,  #
        BLOCK_N: tl.constexpr,  #
        BLOCK_K: tl.constexpr,  #
        NUM_STAGES: tl.constexpr, PACK_ALONG_K: tl.constexpr):  #
```
- **EN:** Defines the helper function `block_scale_fp4_matmul`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `output_ptr`, `a_scale`, `b_scale`, `M`, `N`, `K`, and 13 more. Key calls include `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.zeros`, `tl.store`, `tl.load`, and 1 more. This scope touches Triton compilation or JIT kernels, Triton language operations, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义辅助函数 `block_scale_fp4_matmul`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`output_ptr`、`a_scale`、`b_scale`、`M`、`N`、`K` 等另外 13 项。 关键调用包括 `tl.program_id`、`tl.cdiv`、`tl.arange`、`tl.zeros`、`tl.store`、`tl.load` 等另外 1 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 937-950

```python
    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_M)
    pid_m = pid % num_pid_m
    pid_n = pid // num_pid_m
    offs_am = (pid_m * BLOCK_M + tl.arange(0, BLOCK_M))
    offs_bn = (pid_n * BLOCK_N + tl.arange(0, BLOCK_N))
    PACKING_ALONG_M_N: tl.constexpr = 1 if PACK_ALONG_K else 2
    offs_am_packed = (pid_m * (BLOCK_M // PACKING_ALONG_M_N) + tl.arange(0, BLOCK_M // PACKING_ALONG_M_N))
    offs_bn_packed = (pid_n * (BLOCK_N // PACKING_ALONG_M_N) + tl.arange(0, BLOCK_N // PACKING_ALONG_M_N))
    BLOCK_K_PACKED: tl.constexpr = BLOCK_K // 2 if PACK_ALONG_K else BLOCK_K

    # Two e2m1 values per K
    offs_k = tl.arange(0, BLOCK_K_PACKED)
    offs_scale_k = tl.arange(0, BLOCK_K // VEC_SIZE)
```
- **EN:** Prepares or updates state through `pid`, `num_pid_m`, `pid_m`, `pid_n`, `offs_am`, `offs_bn`, `PACKING_ALONG_M_N`, `offs_am_packed`, and 4 more. Invokes `tl.program_id`, `tl.cdiv`, `tl.arange` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid`、`num_pid_m`、`pid_m`、`pid_n`、`offs_am`、`offs_bn`、`PACKING_ALONG_M_N`、`offs_am_packed` 等另外 4 项 准备或更新状态。 调用 `tl.program_id`、`tl.cdiv`、`tl.arange` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 951-952

```python
    if a_scale is not None:
        a_scale_ptr = a_scale + offs_am[:, None] * stride_scale + offs_scale_k[None, :]
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 953-954

```python
    if b_scale is not None:
        b_scale_ptr = b_scale + offs_bn[:, None] * stride_scale + offs_scale_k[None, :]
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 955-957

```python
    a_ptrs = a_ptr + (offs_am_packed[:, None] * stride_am + offs_k[None, :] * stride_ak)
    b_ptrs = b_ptr + (offs_k[:, None] * stride_bk + offs_bn_packed[None, :] * stride_bn)
    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=output_ptr.dtype.element_ty)
```
- **EN:** Prepares or updates state through `a_ptrs`, `b_ptrs`, `accumulator`. Invokes `tl.zeros` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `a_ptrs`、`b_ptrs`、`accumulator` 准备或更新状态。 调用 `tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 958-976

```python
    for k in tl.range(0, tl.cdiv(K, BLOCK_K), num_stages=NUM_STAGES):
        a = tl.load(a_ptrs)
        b = tl.load(b_ptrs)
        if a_scale is not None:
            scale_a = tl.load(a_scale_ptr)
        else:
            scale_a = None
        if b_scale is not None:
            scale_b = tl.load(b_scale_ptr)
        else:
            scale_b = None
        accumulator = tl.dot_scaled(a, scale_a, "e2m1", b, scale_b, "e2m1", accumulator, lhs_k_pack=PACK_ALONG_K,
                                    rhs_k_pack=PACK_ALONG_K)
        a_ptrs += (BLOCK_K_PACKED) * stride_ak
        b_ptrs += (BLOCK_K_PACKED) * stride_bk
        if a_scale is not None:
            a_scale_ptr += BLOCK_K // VEC_SIZE
        if b_scale is not None:
            b_scale_ptr += BLOCK_K // VEC_SIZE
```
- **EN:** Invokes `tl.cdiv`, `tl.load`, `tl.dot_scaled` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.cdiv`、`tl.load`、`tl.dot_scaled` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

#### Lines 977-981

```python
    offs_cm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
    offs_cn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
    output_ptrs = output_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
    c_mask = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)
    tl.store(output_ptrs, accumulator, mask=c_mask)
```
- **EN:** Prepares or updates state through `offs_cm`, `offs_cn`, `output_ptrs`, `c_mask`. Invokes `tl.arange`, `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `offs_cm`、`offs_cn`、`output_ptrs`、`c_mask` 准备或更新状态。 调用 `tl.arange`、`tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 982-996

```python


@pytest.mark.interpreter
@pytest.mark.parametrize("M, N, K", [(1024, 512, 256)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(128, 128, 128), (256, 128, 128), (128, 256, 128),
                                                       (128, 256, 256), (128, 128, 64), (128, 64, 128), (16, 256, 256),
                                                       (32, 256, 256), (64, 256, 256)])
@pytest.mark.parametrize("with_a_scale", [True, False])
@pytest.mark.parametrize("with_b_scale", [True, False])
@pytest.mark.parametrize("pack_along_k", [True, False])
@pytest.mark.parametrize(("scale_type", "VEC_SIZE"), [("float8_e8m0fnu", 32), ("float8_e4m3fn", 16)],
                         ids=["mxfp4", "nvfp4"])
@pytest.mark.parametrize("nonKDim", ([0, 16, 32] if (is_hip_cdna() or is_hip_gfx1250()) else [0]))
def test_block_scale_fp4(M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, VEC_SIZE, with_a_scale, with_b_scale, pack_along_k,
                         scale_type, nonKDim, device):
```
- **EN:** Defines the test function `test_block_scale_fp4`. Decorators: `pytest.mark.interpreter`, `pytest.mark.parametrize('M, N, K', [(1024, 512, 256)])`, `pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 128), (256, 128, 128), (128, 256, 128), (128, 256, 256), (128, 128, 64), (128, 64, 128), (16, 256, 256), (32, 256, 256), (64, 256, 256)])`, `pytest.mark.parametrize('with_a_scale', [True, False])`, `pytest.mark.parametrize('with_b_scale', [True, False])`, `pytest.mark.parametrize('pack_along_k', [True, False])`, and 2 more. Parameters: `M`, `N`, `K`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `VEC_SIZE`, `with_a_scale`, and 5 more. Key calls include `pytest.mark.parametrize`, `is_cuda`, `torch.manual_seed`, `a_mxfp4.to_packed_tensor`, `torch.rand`, `a_scale.stride`, and 22 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_block_scale_fp4`。 装饰器：`pytest.mark.interpreter`、`pytest.mark.parametrize('M, N, K', [(1024, 512, 256)])`、`pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 128), (256, 128, 128), (128, 256, 128), (128, 256, 256), (128, 128, 64), (128, 64, 128), (16, 256, 256), (32, 256, 256), (64, 256, 256)])`、`pytest.mark.parametrize('with_a_scale', [True, False])`、`pytest.mark.parametrize('with_b_scale', [True, False])`、`pytest.mark.parametrize('pack_along_k', [True, False])` 等另外 2 项。 参数：`M`、`N`、`K`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`VEC_SIZE`、`with_a_scale` 等另外 5 项。 关键调用包括 `pytest.mark.parametrize`、`is_cuda`、`torch.manual_seed`、`a_mxfp4.to_packed_tensor`、`torch.rand`、`a_scale.stride` 等另外 22 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 997-999

```python
    assert M % BLOCK_M == 0
    assert N % BLOCK_N == 0
    assert K % BLOCK_K == 0
```
- **EN:** Validates behavior with 3 assertion(s).
- **CN:** 通过 3 个断言验证行为。

#### Lines 1000-1020

```python

    if is_cuda():
        if BLOCK_M < 128 and not pack_along_k:
            pytest.skip("Packing along M/N with BLOCK_M < 128 is not supported on CUDA")
        if scale_type == "float8_e4m3fn" and not pack_along_k:
            pytest.skip("Packing along K is required for float8_e4m3fn")
        if scale_type == "float8_e4m3fn" and torch.cuda.get_device_capability()[0] < 9:
            pytest.skip("fp8e4nv not supported on Ampere or older")
        if BLOCK_N == 256 and BLOCK_K == 256 and torch.cuda.get_device_capability()[0] < 9:
            pytest.skip("Insufficient SMEM Ampere or older")
        if not (with_a_scale and with_b_scale):
            pytest.skip("None aScale/bScale is only tested on AMD backend for now")
    elif is_hip():
        if not (is_hip_cdna4() or is_hip_gfx1250()):
            pytest.skip("Scaled fp4 matmul is only natively supported on CDNA4")
        if is_hip_gfx1250() and scale_type == "float8_e8m0fnu" and not pack_along_k:
            pytest.skip("fp4 matmul packed along M/N unsupported on gfx1250")
        if scale_type != 'float8_e8m0fnu':
            pytest.skip("CDNA4 only supports E8M0 scale")
        if (nonKDim == 16 and BLOCK_K < 128) or (nonKDim == 32 and BLOCK_K < 64):
            pytest.skip(f"CDNA4 does not support {BLOCK_K=} for scaled mfma {nonKDim=} variants")
```
- **EN:** Invokes `is_cuda`, `is_hip`, `pytest.skip`, `is_hip_gfx1250`, `torch.cuda.get_device_capability`, `is_hip_cdna4` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 调用 `is_cuda`、`is_hip`、`pytest.skip`、`is_hip_gfx1250`、`torch.cuda.get_device_capability`、`is_hip_cdna4` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 1021-1036

```python

    NUM_STAGES = 1
    torch.manual_seed(42)
    packing_dim = 1 if pack_along_k else 0
    a_mxfp4 = MXFP4Tensor(size=(M, K), device=device).random()
    a = a_mxfp4.to_packed_tensor(dim=packing_dim)
    # Generate b with k-major layout, pack two e2m1 along k or n, then logical transpose to K, N
    b_mxfp4 = MXFP4Tensor(size=(N, K), device=device).random()
    b = b_mxfp4.to_packed_tensor(dim=packing_dim).T
    # No need to pack along K since we convert each e2m1 to f32 directly for the reference matmul
    b_ref = b_mxfp4.to(torch.float32).T

    a_size = (M, (K + VEC_SIZE - 1) // VEC_SIZE)
    b_size = (N, (K + VEC_SIZE - 1) // VEC_SIZE)
    a_scale = torch.rand(a_size, device=device)
    b_scale = torch.rand(b_size, device=device)
```
- **EN:** Prepares or updates state through `NUM_STAGES`, `packing_dim`, `a_mxfp4`, `a`, `b_mxfp4`, `b`, `b_ref`, `a_size`, and 3 more. Invokes `torch.manual_seed`, `MXFP4Tensor`, `a_mxfp4.to_packed_tensor`, `b_mxfp4.to_packed_tensor`, `b_mxfp4.to`, `torch.rand` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points, layout transformation reasoning.
- **CN:** 通过 `NUM_STAGES`、`packing_dim`、`a_mxfp4`、`a`、`b_mxfp4`、`b`、`b_ref`、`a_size` 等另外 3 项 准备或更新状态。 调用 `torch.manual_seed`、`MXFP4Tensor`、`a_mxfp4.to_packed_tensor`、`b_mxfp4.to_packed_tensor`、`b_mxfp4.to`、`torch.rand` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点、布局变换推理。

#### Lines 1037-1046

```python
    if scale_type == "float8_e8m0fnu":
        a_scale_ref = MXScaleTensor(a_scale)
        b_scale_ref = MXScaleTensor(b_scale)
        a_scale = a_scale_ref.data
        b_scale = b_scale_ref.data
    elif scale_type == "float8_e4m3fn":
        a_scale = a_scale.to(torch.float8_e4m3fn)
        b_scale = b_scale.to(torch.float8_e4m3fn)
        a_scale_ref = a_scale
        b_scale_ref = b_scale
```
- **EN:** Invokes `MXScaleTensor`, `a_scale.to`, `b_scale.to` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `MXScaleTensor`、`a_scale.to`、`b_scale.to` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 1047-1050

```python

    a_scale_ref = a_scale_ref.to(torch.float32).repeat_interleave(VEC_SIZE, dim=1)[:M, :K]
    b_scale_ref = b_scale_ref.to(torch.float32).repeat_interleave(VEC_SIZE, dim=1).T.contiguous()[:K, :N]
    stride_scale = a_scale.stride(0)
```
- **EN:** Prepares or updates state through `a_scale_ref`, `b_scale_ref`, `stride_scale`. Invokes `a_scale_ref.to`, `b_scale_ref.to`, `a_scale.stride` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `a_scale_ref`、`b_scale_ref`、`stride_scale` 准备或更新状态。 调用 `a_scale_ref.to`、`b_scale_ref.to`、`a_scale.stride` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 1051-1053

```python
    if not with_a_scale:
        a_scale = None
        a_scale_ref = 1.0
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 1054-1056

```python
    if not with_b_scale:
        b_scale = None
        b_scale_ref = 1.0
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 1057-1061

```python
    ref_out = torch.matmul(a_mxfp4.to(torch.float32) * a_scale_ref, b_ref * b_scale_ref)

    output = a.new_empty((M, N), dtype=torch.float32)
    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), 1)
    kernel_kwargs = {}
```
- **EN:** Prepares or updates state through `ref_out`, `output`, `grid`, `kernel_kwargs`. Invokes `torch.matmul`, `a_mxfp4.to`, `a.new_empty`, `triton.cdiv` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 通过 `ref_out`、`output`、`grid`、`kernel_kwargs` 准备或更新状态。 调用 `torch.matmul`、`a_mxfp4.to`、`a.new_empty`、`triton.cdiv` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 1062-1063

```python
    if is_hip():
        kernel_kwargs["matrix_instr_nonkdim"] = nonKDim
```
- **EN:** Invokes `is_hip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1064-1069

```python
    k = block_scale_fp4_matmul[grid](a, b, output, a_scale, b_scale, M, N, K, stride_scale, a.stride(0), a.stride(1),
                                     b.stride(0), b.stride(1), output.stride(0), output.stride(1), VEC_SIZE, BLOCK_M,
                                     BLOCK_N, BLOCK_K, NUM_STAGES=NUM_STAGES, PACK_ALONG_K=pack_along_k,
                                     **kernel_kwargs)
    torch.testing.assert_close(ref_out, output, atol=1e-3, rtol=1e-3)
    nvfp4_fallback = BLOCK_M < 128
```
- **EN:** Prepares or updates state through `k`, `nvfp4_fallback`. Invokes `a.stride`, `b.stride`, `output.stride`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 通过 `k`、`nvfp4_fallback` 准备或更新状态。 调用 `a.stride`、`b.stride`、`output.stride`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 1070-1075

```python
    if is_cuda() and torch.cuda.get_device_capability()[0] in (10, 12) and not nvfp4_fallback:
        ptx = k.asm["ptx"]
        if pack_along_k:
            assert "kind::mxf4" in ptx
        else:
            assert "kind::mxf8f6f4" in ptx
```
- **EN:** Invokes `is_cuda`, `torch.cuda.get_device_capability` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `is_cuda`、`torch.cuda.get_device_capability` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

### Lines 1076-1094

```python


@triton.jit
def mxfp8_mxfp4_matmul(  #
        a_ptr, b_ptr, output_ptr,  #
        a_scale, b_scale,  #
        M, N, K,  #
        stride_scale,  #
        stride_am, stride_ak,  #
        stride_bk, stride_bn,  #
        stride_cm, stride_cn,  #
        tensor_scale: tl.constexpr,  #
        DTYPE_A: tl.constexpr,  #
        DTYPE_B: tl.constexpr,  #
        BLOCK_M: tl.constexpr,  #
        BLOCK_N: tl.constexpr,  #
        BLOCK_K: tl.constexpr,  #
        NUM_STAGES: tl.constexpr,  #
        PACK_B_ALONG_K: tl.constexpr = True):  #
```
- **EN:** Defines the helper function `mxfp8_mxfp4_matmul`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `output_ptr`, `a_scale`, `b_scale`, `M`, `N`, `K`, and 15 more. Key calls include `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.zeros`, `tl.store`, `tl.load`, and 3 more. This scope touches Triton compilation or JIT kernels, Triton language operations, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义辅助函数 `mxfp8_mxfp4_matmul`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`output_ptr`、`a_scale`、`b_scale`、`M`、`N`、`K` 等另外 15 项。 关键调用包括 `tl.program_id`、`tl.cdiv`、`tl.arange`、`tl.zeros`、`tl.store`、`tl.load` 等另外 3 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 1095-1108

```python
    DIV_FACTOR_A: tl.constexpr = 2 if DTYPE_A == "e2m1" else 1
    DIV_FACTOR_B: tl.constexpr = 2 if DTYPE_B == "e2m1" else 1
    DIV_FACTOR_B_K: tl.constexpr = DIV_FACTOR_B if PACK_B_ALONG_K else 1
    DIV_FACTOR_B_N: tl.constexpr = 1 if PACK_B_ALONG_K else DIV_FACTOR_B
    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_M)
    pid_m = pid % num_pid_m
    pid_n = pid // num_pid_m
    offs_am = (pid_m * BLOCK_M + tl.arange(0, BLOCK_M))
    offs_bn = (pid_n * BLOCK_N // DIV_FACTOR_B_N + tl.arange(0, BLOCK_N // DIV_FACTOR_B_N))
    offs_bn_scale = (pid_n * BLOCK_N + tl.arange(0, BLOCK_N)) % N
    offs_ak = tl.arange(0, BLOCK_K // DIV_FACTOR_A)
    offs_bk = tl.arange(0, BLOCK_K // DIV_FACTOR_B_K)
    offs_scale_k = tl.arange(0, BLOCK_K // 32)
```
- **EN:** Prepares or updates state through `DIV_FACTOR_A`, `DIV_FACTOR_B`, `DIV_FACTOR_B_K`, `DIV_FACTOR_B_N`, `pid`, `num_pid_m`, `pid_m`, `pid_n`, and 6 more. Invokes `tl.program_id`, `tl.cdiv`, `tl.arange` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `DIV_FACTOR_A`、`DIV_FACTOR_B`、`DIV_FACTOR_B_K`、`DIV_FACTOR_B_N`、`pid`、`num_pid_m`、`pid_m`、`pid_n` 等另外 6 项 准备或更新状态。 调用 `tl.program_id`、`tl.cdiv`、`tl.arange` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 1109-1111

```python

    if a_scale is not None:
        a_scale_ptr = a_scale + offs_am[:, None] * stride_scale + offs_scale_k[None, :]
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 1112-1113

```python
    if b_scale is not None:
        b_scale_ptr = b_scale + offs_bn_scale[:, None] * stride_scale + offs_scale_k[None, :]
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 1114-1117

```python
    a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_ak[None, :] * stride_ak)
    b_ptrs = b_ptr + (offs_bk[:, None] * stride_bk + offs_bn[None, :] * stride_bn)

    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=output_ptr.dtype.element_ty)
```
- **EN:** Prepares or updates state through `a_ptrs`, `b_ptrs`, `accumulator`. Invokes `tl.zeros` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `a_ptrs`、`b_ptrs`、`accumulator` 准备或更新状态。 调用 `tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 1118-1139

```python

    for k in tl.range(0, tl.cdiv(K, BLOCK_K), num_stages=NUM_STAGES):
        a = tl.load(a_ptrs)
        b = tl.load(b_ptrs)
        if a_scale is not None:
            if tensor_scale:
                scale_a = tl.load(a_scale_ptr)
            else:
                scale_a = tl.full(a_scale_ptr.shape, a_scale.to(tl.int8), dtype=tl.int8)
        else:
            scale_a = None
        if b_scale is not None:
            scale_b = tl.load(b_scale_ptr)
        else:
            scale_b = None
        accumulator = tl.dot_scaled(a, scale_a, DTYPE_A, b, scale_b, DTYPE_B, accumulator, rhs_k_pack=PACK_B_ALONG_K)
        a_ptrs += (BLOCK_K // DIV_FACTOR_A) * stride_ak
        b_ptrs += (BLOCK_K // DIV_FACTOR_B_K) * stride_bk
        if a_scale is not None:
            a_scale_ptr += BLOCK_K // 32
        if b_scale is not None:
            b_scale_ptr += BLOCK_K // 32
```
- **EN:** Invokes `tl.cdiv`, `tl.load`, `tl.dot_scaled`, `tl.full`, `a_scale.to` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.cdiv`、`tl.load`、`tl.dot_scaled`、`tl.full`、`a_scale.to` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

#### Lines 1140-1145

```python

    offs_cm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
    offs_cn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
    output_ptrs = output_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
    c_mask = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)
    tl.store(output_ptrs, accumulator, mask=c_mask)
```
- **EN:** Prepares or updates state through `offs_cm`, `offs_cn`, `output_ptrs`, `c_mask`. Invokes `tl.arange`, `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `offs_cm`、`offs_cn`、`output_ptrs`、`c_mask` 准备或更新状态。 调用 `tl.arange`、`tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 1146-1162

```python


@pytest.mark.interpreter
@pytest.mark.parametrize("M, N, K", [(1024, 512, 512)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(128, 128, 128), (256, 128, 128), (128, 256, 128),
                                                       (128, 256, 256), (128, 128, 64), (128, 64, 128)])
@pytest.mark.parametrize("NUM_STAGES", [1, 3])
@pytest.mark.parametrize("B_TRANS", [True, False])
@pytest.mark.parametrize("PACK_B_ALONG_K", [True, False])
@pytest.mark.parametrize("CONST_SCALE", [True, False])
@pytest.mark.parametrize("A_DATA_TYPE", ["float8e5", "float8e4nv", "float4"])
@pytest.mark.parametrize("B_DATA_TYPE", ["float8e5", "float8e4nv", "float4"])
@pytest.mark.parametrize("WITH_A_SCALE", [True, False])
@pytest.mark.parametrize("WITH_B_SCALE", [True, False])
@pytest.mark.parametrize("nonKDim", ([0, 16, 32] if (is_hip_cdna() or is_hip_gfx1250()) else [0]))
def test_mxfp8_mxfp4_matmul(M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, NUM_STAGES, B_TRANS, PACK_B_ALONG_K, CONST_SCALE,
                            A_DATA_TYPE, B_DATA_TYPE, WITH_A_SCALE, WITH_B_SCALE, nonKDim, device):
```
- **EN:** Defines the test function `test_mxfp8_mxfp4_matmul`. Decorators: `pytest.mark.interpreter`, `pytest.mark.parametrize('M, N, K', [(1024, 512, 512)])`, `pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 128), (256, 128, 128), (128, 256, 128), (128, 256, 256), (128, 128, 64), (128, 64, 128)])`, `pytest.mark.parametrize('NUM_STAGES', [1, 3])`, `pytest.mark.parametrize('B_TRANS', [True, False])`, `pytest.mark.parametrize('PACK_B_ALONG_K', [True, False])`, and 6 more. Parameters: `M`, `N`, `K`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `NUM_STAGES`, `B_TRANS`, and 8 more. Nested definitions in this scope: `create_operand`. Key calls include `pytest.mark.parametrize`, `is_cuda`, `torch.manual_seed`, `create_operand`, `b_scale.stride`, `torch.matmul`, and 22 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_mxfp8_mxfp4_matmul`。 装饰器：`pytest.mark.interpreter`、`pytest.mark.parametrize('M, N, K', [(1024, 512, 512)])`、`pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 128), (256, 128, 128), (128, 256, 128), (128, 256, 256), (128, 128, 64), (128, 64, 128)])`、`pytest.mark.parametrize('NUM_STAGES', [1, 3])`、`pytest.mark.parametrize('B_TRANS', [True, False])`、`pytest.mark.parametrize('PACK_B_ALONG_K', [True, False])` 等另外 6 项。 参数：`M`、`N`、`K`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`NUM_STAGES`、`B_TRANS` 等另外 8 项。 该作用域中的嵌套定义：`create_operand`。 关键调用包括 `pytest.mark.parametrize`、`is_cuda`、`torch.manual_seed`、`create_operand`、`b_scale.stride`、`torch.matmul` 等另外 22 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 1163-1180

```python
    if is_cuda():
        if torch.cuda.get_device_capability()[0] != 10:
            pytest.skip("Requires compute capability == 10")
        if not (WITH_A_SCALE and WITH_B_SCALE):
            pytest.skip("None scale has not been tested on NV backend")
        if not (A_DATA_TYPE == "float8e5" and B_DATA_TYPE == "float4"):
            pytest.skip(f"(A: {A_DATA_TYPE}, B: {B_DATA_TYPE}) has not been tested on NV backend")
    elif is_hip():
        if not (is_hip_cdna4() or is_hip_gfx1250()):
            pytest.skip("Scaled mxfp4 & mxfp8 matmul is only natively supported on CDNA4 and above")
        if (nonKDim == 16 and BLOCK_K < 128) or (nonKDim == 32 and BLOCK_K < 64):
            pytest.skip(f"CDNA4 does not support {BLOCK_K=} for scaled mfma {nonKDim=} variants")
        if (A_DATA_TYPE == 'float4' and not WITH_A_SCALE) or (B_DATA_TYPE == 'float4' and not WITH_B_SCALE):
            pytest.skip("Float4 without scale is tested in test_block_scale_fp4")
        if (is_hip_gfx1250() and B_DATA_TYPE == 'float4' and not PACK_B_ALONG_K):
            pytest.skip("Float4 matmul packed along M/N unsupported on gfx1250")
        if (BLOCK_M == 256 or BLOCK_N == 256) and BLOCK_K == 256:
            pytest.skip("Config requires too much shared memory")
```
- **EN:** Invokes `is_cuda`, `is_hip`, `pytest.skip`, `torch.cuda.get_device_capability`, `is_hip_gfx1250`, `is_hip_cdna4` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 调用 `is_cuda`、`is_hip`、`pytest.skip`、`torch.cuda.get_device_capability`、`is_hip_gfx1250`、`is_hip_cdna4` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 1181-1182

```python
    if not PACK_B_ALONG_K and B_DATA_TYPE != "float4":
        pytest.skip("Pack along K can only be False for float4")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1183-1184

```python
    if BLOCK_N == 256 and BLOCK_K == 256:
        NUM_STAGES = 2
```
- **EN:** Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

#### Lines 1185-1186

```python

    torch.manual_seed(42)
```
- **EN:** Invokes `torch.manual_seed` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 调用 `torch.manual_seed` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1187-1189

```python

    def create_operand(dtype: str, size0: int, size1: int, k_dim: int, transpose: bool = True,
                       pack_along_k: bool = True):
```
- **EN:** Defines the helper function `create_operand`. Parameters: `dtype`, `size0`, `size1`, `k_dim`, `transpose`, `pack_along_k`. Key calls include `v_mxfp4.to_packed_tensor`, `v_mxfp4.to`, `f8_to_f16`, `v.view`, `MXFP4Tensor`, `torch.randint`. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义辅助函数 `create_operand`。 参数：`dtype`、`size0`、`size1`、`k_dim`、`transpose`、`pack_along_k`。 关键调用包括 `v_mxfp4.to_packed_tensor`、`v_mxfp4.to`、`f8_to_f16`、`v.view`、`MXFP4Tensor`、`torch.randint`。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

##### Lines 1190-1217

```python
        if dtype == "float8e5":
            if transpose:
                v = torch.randint(20, 40, (size0, size1), dtype=torch.uint8).view(torch.float8_e5m2).to(device)
                v_ref = f8_to_f16(v.view(torch.float8_e5m2), dtype).to(torch.float32)
            else:
                v = torch.randint(20, 40, (size1, size0), dtype=torch.uint8).view(torch.float8_e5m2).to(device).T
                v_ref = f8_to_f16(v.view(torch.float8_e5m2).T, dtype).to(torch.float32).T
        elif dtype == "float8e4nv":
            if transpose:
                v = torch.randint(20, 40, (size0, size1), dtype=torch.uint8).view(torch.float8_e4m3fn).to(device)
                v_ref = f8_to_f16(v.view(torch.float8_e4m3fn), dtype).to(torch.float32)
            else:
                v = torch.randint(20, 40, (size1, size0), dtype=torch.uint8).view(torch.float8_e4m3fn).to(device).T
                v_ref = f8_to_f16(v.view(torch.float8_e4m3fn).T, dtype).to(torch.float32).T
        else:
            # float4
            if pack_along_k:
                pack_dim = k_dim
            else:
                pack_dim = (k_dim + 1) % 2
            if transpose:
                v_mxfp4 = MXFP4Tensor(size=(size0, size1), device=device).random()
                v = v_mxfp4.to_packed_tensor(dim=pack_dim)
                v_ref = v_mxfp4.to(torch.float32)
            else:
                v_mxfp4 = MXFP4Tensor(size=(size1, size0), device=device).random()
                v = v_mxfp4.to_packed_tensor(dim=(pack_dim + 1) % 2).T
                v_ref = v_mxfp4.to(torch.float32).T
```
- **EN:** Invokes `v_mxfp4.to_packed_tensor`, `v_mxfp4.to`, `f8_to_f16`, `v.view`, `MXFP4Tensor`, `torch.randint` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 调用 `v_mxfp4.to_packed_tensor`、`v_mxfp4.to`、`f8_to_f16`、`v.view`、`MXFP4Tensor`、`torch.randint` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、随机数据生成。

##### Lines 1218-1218

```python
        return v, v_ref
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

#### Lines 1219-1230

```python

    dtype_converter = {'float8e5': 'e5m2', 'float8e4nv': 'e4m3', 'float4': 'e2m1'}

    a, a_ref = create_operand(A_DATA_TYPE, M, K, 1)
    b, b_ref = create_operand(B_DATA_TYPE, K, N, 0, B_TRANS, PACK_B_ALONG_K)

    a_scale_mxfp4 = MXScaleTensor(size=(M, (K + 32 - 1) // 32), device=device).random(high=32.0)
    b_scale_mxfp4 = MXScaleTensor(size=(N, (K + 32 - 1) // 32), device=device).random(high=32.0)
    a_scale = a_scale_mxfp4.data
    b_scale = b_scale_mxfp4.data

    a_scale_ref = a_scale_mxfp4.to(torch.float32).repeat_interleave(32, dim=1)[:M, :K]
```
- **EN:** Prepares or updates state through `dtype_converter`, `a`, `a_ref`, `b`, `b_ref`, `a_scale_mxfp4`, `b_scale_mxfp4`, `a_scale`, and 2 more. Invokes `create_operand`, `MXScaleTensor`, `a_scale_mxfp4.to` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `dtype_converter`、`a`、`a_ref`、`b`、`b_ref`、`a_scale_mxfp4`、`b_scale_mxfp4`、`a_scale` 等另外 2 项 准备或更新状态。 调用 `create_operand`、`MXScaleTensor`、`a_scale_mxfp4.to` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1231-1233

```python
    if CONST_SCALE:
        a_scale_ref = torch.full_like(a_scale_ref, 2.0)
        a_scale = 128  # 2.0 in e8m0
```
- **EN:** Invokes `torch.full_like` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.full_like` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 1234-1235

```python
    b_scale_ref = b_scale_mxfp4.to(torch.float32).repeat_interleave(32, dim=1).T.contiguous()[:K, :N]
    stride_scale = b_scale.stride(0)
```
- **EN:** Prepares or updates state through `b_scale_ref`, `stride_scale`. Invokes `b_scale_mxfp4.to`, `b_scale.stride` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `b_scale_ref`、`stride_scale` 准备或更新状态。 调用 `b_scale_mxfp4.to`、`b_scale.stride` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 1236-1238

```python
    if not WITH_A_SCALE:
        a_scale = None
        a_scale_ref = 1.0
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 1239-1241

```python
    if not WITH_B_SCALE:
        b_scale = None
        b_scale_ref = 1.0
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 1242-1247

```python

    ref_out = torch.matmul(a_ref * a_scale_ref, b_ref * b_scale_ref)

    output = a.new_empty((M, N), dtype=torch.float32)
    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), 1)
    kernel_kwargs = {}
```
- **EN:** Prepares or updates state through `ref_out`, `output`, `grid`, `kernel_kwargs`. Invokes `torch.matmul`, `a.new_empty`, `triton.cdiv` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 通过 `ref_out`、`output`、`grid`、`kernel_kwargs` 准备或更新状态。 调用 `torch.matmul`、`a.new_empty`、`triton.cdiv` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 1248-1249

```python
    if is_hip():
        kernel_kwargs["matrix_instr_nonkdim"] = nonKDim
```
- **EN:** Invokes `is_hip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1250-1253

```python
    out = mxfp8_mxfp4_matmul[grid](a, b, output, a_scale, b_scale, M, N, K, stride_scale, a.stride(0), a.stride(1),
                                   b.stride(0), b.stride(1), output.stride(0), output.stride(1), not CONST_SCALE,
                                   dtype_converter[A_DATA_TYPE], dtype_converter[B_DATA_TYPE], BLOCK_M, BLOCK_N,
                                   BLOCK_K, PACK_B_ALONG_K=PACK_B_ALONG_K, NUM_STAGES=NUM_STAGES, **kernel_kwargs)
```
- **EN:** Prepares or updates state through `out`. Invokes `a.stride`, `b.stride`, `output.stride` to execute the test logic. Relevant themes: matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 通过 `out` 准备或更新状态。 调用 `a.stride`、`b.stride`、`output.stride` 执行测试逻辑。 相关主题：矩阵乘法工作流、插件或编译器扩展点。

#### Lines 1254-1256

```python
    if is_cuda():
        ttgir = out.asm["ttgir"]
        assert "fp4Padded = true" in ttgir
```
- **EN:** Invokes `is_cuda` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions.
- **CN:** 调用 `is_cuda` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 1257-1258

```python

    torch.testing.assert_close(ref_out, output, atol=1e-3, rtol=1e-3)
```
- **EN:** Invokes `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 1259-1272

```python


@triton.jit
def batched_mxfp_matmul(  #
        a_ptr, b_ptr, output_ptr,  #
        a_scale, b_scale,  #
        M, N, K,  #
        stride_sfa_bs: tl.constexpr, stride_sfa_m: tl.constexpr, stride_sfb_bs: tl.constexpr,
        stride_sfb_n: tl.constexpr, stride_ab, stride_am, stride_ak,  #
        stride_bb, stride_bk, stride_bn,  #
        stride_cb, stride_cm, stride_cn,  #
        BATCH_SIZE, BLOCK_BATCH_SIZE: tl.constexpr,  #
        BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr, BLOCK_K: tl.constexpr,  #
        NUM_STAGES: tl.constexpr):
```
- **EN:** Defines the helper function `batched_mxfp_matmul`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `output_ptr`, `a_scale`, `b_scale`, `M`, `N`, `K`, and 19 more. Key calls include `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.zeros`, `tl.store`, `tl.load`, and 1 more. This scope touches Triton compilation or JIT kernels, Triton language operations, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义辅助函数 `batched_mxfp_matmul`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`output_ptr`、`a_scale`、`b_scale`、`M`、`N`、`K` 等另外 19 项。 关键调用包括 `tl.program_id`、`tl.cdiv`、`tl.arange`、`tl.zeros`、`tl.store`、`tl.load` 等另外 1 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 1273-1296

```python
    pid = tl.program_id(axis=0)
    batch_id = tl.program_id(axis=1)

    num_pid_m = tl.cdiv(M, BLOCK_M)
    pid_m = pid % num_pid_m
    pid_n = pid // num_pid_m

    offs_batch = (batch_id * BLOCK_BATCH_SIZE + tl.arange(0, BLOCK_BATCH_SIZE)) % BATCH_SIZE
    offs_am = (pid_m * BLOCK_M + tl.arange(0, BLOCK_M)) % M
    offs_bn = (pid_n * BLOCK_N + tl.arange(0, BLOCK_N)) % N
    offs_k = tl.arange(0, BLOCK_K)
    offs_scale_k = tl.arange(0, BLOCK_K // 32)

    a_scale_ptr = (a_scale + offs_batch[:, None, None] * stride_sfa_bs + offs_am[None, :, None] * stride_sfa_m +
                   offs_scale_k[None, None, :])
    b_scale_ptr = (b_scale + offs_batch[:, None, None] * stride_sfb_bs + offs_bn[None, :, None] * stride_sfb_n +
                   offs_scale_k[None, None, :])

    a_ptrs = (a_ptr + offs_batch[:, None, None] * stride_ab + offs_am[None, :, None] * stride_am +
              offs_k[None, None, :] * stride_ak)
    b_ptrs = (b_ptr + offs_batch[:, None, None] * stride_bb + offs_k[None, :, None] * stride_bk +
              offs_bn[None, None, :] * stride_bn)

    accumulator = tl.zeros((BLOCK_BATCH_SIZE, BLOCK_M, BLOCK_N), dtype=output_ptr.dtype.element_ty)
```
- **EN:** Prepares or updates state through `pid`, `batch_id`, `num_pid_m`, `pid_m`, `pid_n`, `offs_batch`, `offs_am`, `offs_bn`, and 7 more. Invokes `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.zeros` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid`、`batch_id`、`num_pid_m`、`pid_m`、`pid_n`、`offs_batch`、`offs_am`、`offs_bn` 等另外 7 项 准备或更新状态。 调用 `tl.program_id`、`tl.cdiv`、`tl.arange`、`tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 1297-1308

```python

    for k in tl.range(0, tl.cdiv(K, BLOCK_K), num_stages=NUM_STAGES):
        a = tl.load(a_ptrs)
        b = tl.load(b_ptrs)
        scale_a = tl.load(a_scale_ptr)
        scale_b = tl.load(b_scale_ptr)
        accumulator = tl.dot_scaled(a, scale_a, "e5m2", b, scale_b, "e5m2", accumulator)

        a_ptrs += BLOCK_K * stride_ak
        b_ptrs += BLOCK_K * stride_bk
        a_scale_ptr += BLOCK_K // 32
        b_scale_ptr += BLOCK_K // 32
```
- **EN:** Invokes `tl.cdiv`, `tl.load`, `tl.dot_scaled` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.cdiv`、`tl.load`、`tl.dot_scaled` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

#### Lines 1309-1315

```python

    offs_cm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
    offs_cn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
    output_ptrs = (output_ptr + stride_cb * offs_batch[:, None, None] + stride_cm * offs_cm[None, :, None] +
                   stride_cn * offs_cn[None, None, :])
    c_mask = ((offs_batch[:, None, None] < BATCH_SIZE) & (offs_cm[None, :, None] < M) & (offs_cn[None, None, :] < N))
    tl.store(output_ptrs, accumulator, mask=c_mask)
```
- **EN:** Prepares or updates state through `offs_cm`, `offs_cn`, `output_ptrs`, `c_mask`. Invokes `tl.arange`, `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `offs_cm`、`offs_cn`、`output_ptrs`、`c_mask` 准备或更新状态。 调用 `tl.arange`、`tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 1316-1324

```python


@pytest.mark.interpreter
@pytest.mark.parametrize("BATCH_SIZE, BLOCK_BATCH_SIZE", [(1, 1), (16, 1), (16, 4)])
@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(128, 128, 64), (128, 64, 128), (64, 64, 128)])
@pytest.mark.parametrize("NUM_STAGES", [1, 2 if is_hip() else 3])
@pytest.mark.parametrize("NUM_WARPS", [4, 8])
@pytest.mark.parametrize("nonKDim", ([0, 16, 32] if (is_hip_cdna() or is_hip_gfx1250()) else [0]))
def test_batched_mxfp(BATCH_SIZE, BLOCK_BATCH_SIZE, BLOCK_M, BLOCK_N, BLOCK_K, NUM_STAGES, nonKDim, NUM_WARPS, device):
```
- **EN:** Defines the test function `test_batched_mxfp`. Decorators: `pytest.mark.interpreter`, `pytest.mark.parametrize('BATCH_SIZE, BLOCK_BATCH_SIZE', [(1, 1), (16, 1), (16, 4)])`, `pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 64), (128, 64, 128), (64, 64, 128)])`, `pytest.mark.parametrize('NUM_STAGES', [1, 2 if is_hip() else 3])`, `pytest.mark.parametrize('NUM_WARPS', [4, 8])`, `pytest.mark.parametrize('nonKDim', [0, 16, 32] if is_hip_cdna() or is_hip_gfx1250() else [0])`. Parameters: `BATCH_SIZE`, `BLOCK_BATCH_SIZE`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `NUM_STAGES`, `nonKDim`, `NUM_WARPS`, and 1 more. Key calls include `pytest.mark.parametrize`, `torch.manual_seed`, `f8_to_f16`, `torch.randint`, `getattr`, `torch.empty`, and 19 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_batched_mxfp`。 装饰器：`pytest.mark.interpreter`、`pytest.mark.parametrize('BATCH_SIZE, BLOCK_BATCH_SIZE', [(1, 1), (16, 1), (16, 4)])`、`pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 64), (128, 64, 128), (64, 64, 128)])`、`pytest.mark.parametrize('NUM_STAGES', [1, 2 if is_hip() else 3])`、`pytest.mark.parametrize('NUM_WARPS', [4, 8])`、`pytest.mark.parametrize('nonKDim', [0, 16, 32] if is_hip_cdna() or is_hip_gfx1250() else [0])`。 参数：`BATCH_SIZE`、`BLOCK_BATCH_SIZE`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`NUM_STAGES`、`nonKDim`、`NUM_WARPS` 等另外 1 项。 关键调用包括 `pytest.mark.parametrize`、`torch.manual_seed`、`f8_to_f16`、`torch.randint`、`getattr`、`torch.empty` 等另外 19 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 1325-1325

```python
    M, N, K = 1024, 512, 2048
```
- **EN:** Prepares or updates state through `M`, `N`, `K`.
- **CN:** 通过 `M`、`N`、`K` 准备或更新状态。

#### Lines 1326-1328

```python

    if K % BLOCK_K != 0:
        pytest.skip("Kernel requires shapes aligned by K dimension")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1329-1337

```python
    if is_cuda() and torch.cuda.get_device_capability()[0] < 10:
        pytest.skip("Requires compute capability >= 10")
    elif is_hip():
        if not (is_hip_cdna4() or is_hip_gfx1250()):
            pytest.skip("Scaled mxfp8 matmul is only natively supported on CDNA4 and above")
        if (nonKDim == 16 and BLOCK_K < 128) or (nonKDim == 32 and BLOCK_K < 64):
            pytest.skip(f"CDNA4 does not support {BLOCK_K=} for scaled mfma {nonKDim=} variants")
        if is_hip_cdna4() and NUM_STAGES > 1 and max(BLOCK_M, BLOCK_N) > 64:
            pytest.skip("Config requires too much shared memory")
```
- **EN:** Invokes `is_cuda`, `pytest.skip`, `is_hip`, `torch.cuda.get_device_capability`, `is_hip_cdna4`, `is_hip_gfx1250`, and 1 more to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 调用 `is_cuda`、`pytest.skip`、`is_hip`、`torch.cuda.get_device_capability`、`is_hip_cdna4`、`is_hip_gfx1250` 等另外 1 项 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 1338-1355

```python

    torch.manual_seed(42)
    dtype_src_str = "float8e5"
    dtype_dst_str = "float32"

    a = torch.randint(20, 40, (BATCH_SIZE, M, K), dtype=torch.uint8, device=device).view(torch.float8_e5m2)
    b = torch.randint(20, 40, (BATCH_SIZE, K, N), dtype=torch.uint8, device=device).view(torch.float8_e5m2)
    a_f16 = f8_to_f16(a, dtype_src_str)
    b_f16 = f8_to_f16(b, dtype_src_str)

    a_scale = torch.randint(64, 130, (BATCH_SIZE, M, K // 32), dtype=torch.uint8, device=device)
    b_scale = torch.randint(64, 130, (BATCH_SIZE, N, K // 32), dtype=torch.uint8, device=device)

    dtype_dst = getattr(torch, dtype_dst_str)
    output = torch.empty((BATCH_SIZE, M, N), dtype=dtype_dst, device=device)
    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), BATCH_SIZE // BLOCK_BATCH_SIZE)

    kernel_kwargs = {}
```
- **EN:** Prepares or updates state through `dtype_src_str`, `dtype_dst_str`, `a`, `b`, `a_f16`, `b_f16`, `a_scale`, `b_scale`, and 4 more. Invokes `torch.manual_seed`, `torch.randint`, `f8_to_f16`, `getattr`, `torch.empty`, `triton.cdiv` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `dtype_src_str`、`dtype_dst_str`、`a`、`b`、`a_f16`、`b_f16`、`a_scale`、`b_scale` 等另外 4 项 准备或更新状态。 调用 `torch.manual_seed`、`torch.randint`、`f8_to_f16`、`getattr`、`torch.empty`、`triton.cdiv` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 1356-1357

```python
    if is_hip():
        kernel_kwargs["matrix_instr_nonkdim"] = nonKDim
```
- **EN:** Invokes `is_hip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 1358-1397

```python

    out = batched_mxfp_matmul[grid](
        a,
        b,
        output,
        a_scale,
        b_scale,
        M,
        N,
        K,
        a_scale.stride(0),
        a_scale.stride(1),
        b_scale.stride(0),
        b_scale.stride(1),
        a.stride(0),
        a.stride(1),
        a.stride(2),
        b.stride(0),
        b.stride(1),
        b.stride(2),
        output.stride(0),
        output.stride(1),
        output.stride(2),
        BATCH_SIZE,
        BLOCK_BATCH_SIZE,
        BLOCK_M,
        BLOCK_N,
        BLOCK_K,
        NUM_STAGES=NUM_STAGES,
        num_warps=NUM_WARPS,
        **kernel_kwargs,
    )

    a_scale_f32 = fp8e8m0_to_float32(a_scale).repeat_interleave(32, dim=2)
    b_scale_f32 = fp8e8m0_to_float32(b_scale).repeat_interleave(32, dim=2)
    b_scale_f32 = b_scale_f32.permute(0, 2, 1).contiguous()  # b_scales are always col major

    ref_out = torch.matmul(a_f16 * a_scale_f32, b_f16 * b_scale_f32).to(torch.float32)

    torch.testing.assert_close(ref_out, output.to(torch.float32), atol=0.02, rtol=0)
```
- **EN:** Prepares or updates state through `out`, `a_scale_f32`, `b_scale_f32`, `ref_out`. Invokes `a_scale.stride`, `b_scale.stride`, `a.stride`, `b.stride`, `output.stride`, `fp8e8m0_to_float32`, and 4 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 通过 `out`、`a_scale_f32`、`b_scale_f32`、`ref_out` 准备或更新状态。 调用 `a_scale.stride`、`b_scale.stride`、`a.stride`、`b.stride`、`output.stride`、`fp8e8m0_to_float32` 等另外 4 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 1398-1401

```python

    if is_cuda() and torch.cuda.get_device_capability()[0] == 12:
        ptx = out.asm["ptx"]
        assert "mma.sync.aligned.m16n8k32.row.col.kind::mxf8f6f4.block_scale.scale_vec::1X" in ptx
```
- **EN:** Invokes `is_cuda`, `torch.cuda.get_device_capability` to execute the test logic. Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `is_cuda`、`torch.cuda.get_device_capability` 执行测试逻辑。 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `f8_to_f16`, `matmul_kernel`, `get_src_element_ty_size`, `test_simple_matmul`, `simple_persistent_kernel`, `test_simple_persistent_matmul`, `mxfp_matmul`, `fp8e8m0_to_float32`
  **CN:** 顶层作用域，例如 `f8_to_f16`、`matmul_kernel`、`get_src_element_ty_size`、`test_simple_matmul`、`simple_persistent_kernel`、`test_simple_persistent_matmul`、`mxfp_matmul`、`fp8e8m0_to_float32`
- **EN:** pytest parametrization
  **CN:** pytest 参数化
- **EN:** Triton compilation or JIT kernels
  **CN:** Triton 编译或 JIT kernel
- **EN:** Triton language operations
  **CN:** Triton language 操作
- **EN:** PyTorch tensor setup and checks
  **CN:** PyTorch 张量准备与校验
- **EN:** cache management behavior
  **CN:** 缓存管理行为
- **EN:** matrix multiplication workflows
  **CN:** 矩阵乘法工作流
- **EN:** plugin or compiler extension points
  **CN:** 插件或编译器扩展点

## Dependencies / 依赖关系

- **EN:** External or absolute imports include `math`, `pytest`, `torch`, `triton`, `triton.language`, `test_mxfp`, `re`, `triton._internal_testing`.
  **CN:** 外部或绝对导入包括 `math`、`pytest`、`torch`、`triton`、`triton.language`、`test_mxfp`、`re`、`triton._internal_testing`。
- **EN:** Execution centers on top-level definitions such as `f8_to_f16`, `matmul_kernel`, `get_src_element_ty_size`, `test_simple_matmul`, `simple_persistent_kernel`, `test_simple_persistent_matmul`, `mxfp_matmul`, `fp8e8m0_to_float32`, `test_mxfp`, `_knob_promote_lhs_to_tmem`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `f8_to_f16`、`matmul_kernel`、`get_src_element_ty_size`、`test_simple_matmul`、`simple_persistent_kernel`、`test_simple_persistent_matmul`、`mxfp_matmul`、`fp8e8m0_to_float32`、`test_mxfp`、`_knob_promote_lhs_to_tmem`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
