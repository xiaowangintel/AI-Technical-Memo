# test_pipeliner.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/unit/language/test_pipeliner.py`
- **EN:** Pytest module covering pipeliner behavior in Triton's Python tests. It contains 15 top-level definition(s) and 6 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 pipeliner 行为。 该文件包含 15 个顶层定义，以及 6 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```python
# End-to-end tests to check the correctness of the pipeliner

import pytest
import torch
import triton
import triton.language as tl

from triton._internal_testing import is_cuda, is_hopper_or_newer, is_hip_cdna, is_hip_cdna2, is_hip
```
- **EN:** Imports the modules used in this scope: `pytest`, `torch`, `triton`, `triton.language`, `triton._internal_testing`. Relevant themes: Triton language operations.
- **CN:** 导入此作用域使用的模块：`pytest`、`torch`、`triton`、`triton.language`、`triton._internal_testing`。 相关主题：Triton language 操作。

### Lines 9-11

```python


def check_capabilities():
```
- **EN:** Defines the helper function `check_capabilities`. Key calls include `is_cuda`, `torch.cuda.get_device_capability`, `pytest.skip`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `check_capabilities`。 关键调用包括 `is_cuda`、`torch.cuda.get_device_capability`、`pytest.skip`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 12-15

```python
    if is_cuda():
        cc = torch.cuda.get_device_capability()
        if cc[0] < 8:
            pytest.skip("CUDA 8.0+ required")
```
- **EN:** Invokes `is_cuda`, `torch.cuda.get_device_capability`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `is_cuda`、`torch.cuda.get_device_capability`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

### Lines 16-27

```python


@triton.jit
def matmul_kernel(  #
        a_ptr, scale_ptr, b_ptr, output_ptr,  #
        M, N, K_MXFP,  # K_MXFP is the number of mxfp vectors in a row of a. Otherwise it's just K
        stride_am, stride_ak,  #
        stride_sm, stride_sk,  #
        stride_bk, stride_bn,  #
        stride_cm, stride_cn,  #
        BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr, BLOCK_K: tl.constexpr,  #
        NUM_STAGES: tl.constexpr, a_type: tl.constexpr, b_type: tl.constexpr):
```
- **EN:** Defines the helper function `matmul_kernel`. Decorators: `triton.jit`. Parameters: `a_ptr`, `scale_ptr`, `b_ptr`, `output_ptr`, `M`, `N`, `K_MXFP`, `stride_am`, and 13 more. Key calls include `tl.program_id`, `tl.cdiv`, `tl.arange`, `tl.zeros`, `accumulator.to`, `tl.store`, and 3 more. This scope touches Triton compilation or JIT kernels, Triton language operations, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义辅助函数 `matmul_kernel`。 装饰器：`triton.jit`。 参数：`a_ptr`、`scale_ptr`、`b_ptr`、`output_ptr`、`M`、`N`、`K_MXFP`、`stride_am` 等另外 13 项。 关键调用包括 `tl.program_id`、`tl.cdiv`、`tl.arange`、`tl.zeros`、`accumulator.to`、`tl.store` 等另外 3 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 28-44

```python
    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_M)
    pid_m = pid % num_pid_m
    pid_n = pid // num_pid_m
    offs_am = (pid_m * BLOCK_M + tl.arange(0, BLOCK_M)) % M
    offs_bn = (pid_n * BLOCK_N + tl.arange(0, BLOCK_N)) % N
    IS_SCALED: tl.constexpr = a_type is not None and b_type is not None
    DIV_FACTOR: tl.constexpr = 2 if IS_SCALED and a_type == "e2m1" else 1
    # We pass K_MXFP to make explicit that KB is multiple of 32 and KA is multiple of 16 or 32
    # for the pipeliner divisibility condition
    KA = K_MXFP if not IS_SCALED else K_MXFP * (32 // DIV_FACTOR)
    KB = K_MXFP if not IS_SCALED else K_MXFP * 32
    BLOCK_AK: tl.constexpr = BLOCK_K // DIV_FACTOR
    offs_k = tl.arange(0, BLOCK_K)
    offs_ak = tl.arange(0, BLOCK_AK)
    a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_ak[None, :] * stride_ak)
    b_ptrs = b_ptr + (offs_k[:, None] * stride_bk + offs_bn[None, :] * stride_bn)
```
- **EN:** Prepares or updates state through `pid`, `num_pid_m`, `pid_m`, `pid_n`, `offs_am`, `offs_bn`, `IS_SCALED`, `DIV_FACTOR`, and 7 more. Invokes `tl.program_id`, `tl.cdiv`, `tl.arange` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid`、`num_pid_m`、`pid_m`、`pid_n`、`offs_am`、`offs_bn`、`IS_SCALED`、`DIV_FACTOR` 等另外 7 项 准备或更新状态。 调用 `tl.program_id`、`tl.cdiv`、`tl.arange` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 45-48

```python
    if IS_SCALED:
        BLOCK_SK: tl.constexpr = BLOCK_K // 32
        offs_sk = tl.arange(0, BLOCK_SK)
        scale_ptrs = scale_ptr + (offs_am[:, None] * stride_sm + offs_sk[None, :] * stride_sk)
```
- **EN:** Invokes `tl.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

#### Lines 49-49

```python
    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=tl.float32)
```
- **EN:** Prepares or updates state through `accumulator`. Invokes `tl.zeros` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `accumulator` 准备或更新状态。 调用 `tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 50-65

```python
    for k in tl.range(0, tl.cdiv(KB, BLOCK_K), num_stages=NUM_STAGES):
        mask_a = (offs_am[:, None] < M) & (offs_ak[None, :] + k * BLOCK_AK < KA)
        mask_b = ((offs_k[:, None] + k * BLOCK_K) < KB) & (offs_bn[None, :] < N)
        a = tl.load(a_ptrs, mask=mask_a, other=0)
        b = tl.load(b_ptrs, mask=mask_b, other=0)
        if IS_SCALED:
            # Adapted scale indexing and dot_scaled operation
            mask_scale = (offs_am[:, None] < M) & (offs_sk[None, :] + k * BLOCK_SK < K_MXFP)
            a_scale = tl.load(scale_ptrs, mask=mask_scale, other=0)
            accumulator = tl.dot_scaled(a, a_scale, a_type, b, None, b_type, acc=accumulator)
        else:
            accumulator = tl.dot(a, b, acc=accumulator)
        a_ptrs += BLOCK_AK * stride_ak
        b_ptrs += BLOCK_K * stride_bk
        if IS_SCALED:
            scale_ptrs += BLOCK_SK * stride_sk
```
- **EN:** Invokes `tl.cdiv`, `tl.load`, `tl.dot_scaled`, `tl.dot` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.cdiv`、`tl.load`、`tl.dot_scaled`、`tl.dot` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

#### Lines 66-72

```python
    OUT_DTYPE = tl.bfloat16 if IS_SCALED else tl.float16
    accumulator = accumulator.to(OUT_DTYPE)
    offs_cm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
    offs_cn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
    mask_c = (offs_cm[:, None] < M) & (offs_cn[None, :] < N)
    output_ptrs = output_ptr + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
    tl.store(output_ptrs, accumulator, mask=mask_c)
```
- **EN:** Prepares or updates state through `OUT_DTYPE`, `accumulator`, `offs_cm`, `offs_cn`, `mask_c`, `output_ptrs`. Invokes `accumulator.to`, `tl.arange`, `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `OUT_DTYPE`、`accumulator`、`offs_cm`、`offs_cn`、`mask_c`、`output_ptrs` 准备或更新状态。 调用 `accumulator.to`、`tl.arange`、`tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 73-80

```python


@triton.jit
def matmul_kernel_tma(  #
        a_ptr, b_ptr, output_ptr,  #
        M, N, K,  #
        BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr, BLOCK_K: tl.constexpr,  #
        NUM_STAGES: tl.constexpr):
```
- **EN:** Defines the helper function `matmul_kernel_tma`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `output_ptr`, `M`, `N`, `K`, `BLOCK_M`, `BLOCK_N`, and 2 more. Key calls include `tl.program_id`, `tl.cdiv`, `tl.multiple_of`, `tl.zeros`, `accumulator.to`, `output_ptr.store`, and 3 more. This scope touches Triton compilation or JIT kernels, Triton language operations, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义辅助函数 `matmul_kernel_tma`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`output_ptr`、`M`、`N`、`K`、`BLOCK_M`、`BLOCK_N` 等另外 2 项。 关键调用包括 `tl.program_id`、`tl.cdiv`、`tl.multiple_of`、`tl.zeros`、`accumulator.to`、`output_ptr.store` 等另外 3 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 81-90

```python
    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_M)
    pid_m = pid % num_pid_m
    pid_n = pid // num_pid_m
    offs_am = (pid_m * BLOCK_M) % M
    offs_bn = (pid_n * BLOCK_N) % N
    offs_am = tl.multiple_of(offs_am, BLOCK_M)
    offs_bn = tl.multiple_of(offs_bn, BLOCK_N)
    offs_k = 0
    accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=tl.float32)
```
- **EN:** Prepares or updates state through `pid`, `num_pid_m`, `pid_m`, `pid_n`, `offs_am`, `offs_bn`, `offs_k`, `accumulator`. Invokes `tl.program_id`, `tl.cdiv`, `tl.multiple_of`, `tl.zeros` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid`、`num_pid_m`、`pid_m`、`pid_n`、`offs_am`、`offs_bn`、`offs_k`、`accumulator` 准备或更新状态。 调用 `tl.program_id`、`tl.cdiv`、`tl.multiple_of`、`tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 91-95

```python
    for _ in tl.range(0, tl.cdiv(K, BLOCK_K), num_stages=NUM_STAGES):
        a = a_ptr.load([offs_am, offs_k])
        b = b_ptr.load([offs_k, offs_bn])
        accumulator = tl.dot(a, b, acc=accumulator)
        offs_k += BLOCK_K
```
- **EN:** Invokes `tl.cdiv`, `a_ptr.load`, `b_ptr.load`, `tl.dot` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.cdiv`、`a_ptr.load`、`b_ptr.load`、`tl.dot` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

#### Lines 96-97

```python
    accumulator = accumulator.to(tl.float16)
    output_ptr.store([offs_am, offs_bn], accumulator)
```
- **EN:** Prepares or updates state through `accumulator`. Invokes `accumulator.to`, `output_ptr.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `accumulator` 准备或更新状态。 调用 `accumulator.to`、`output_ptr.store` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 98-101

```python


@triton.jit
def vecadd_kernel(a_ptr, b_ptr, output_ptr, n_elements, num_blocks, BLOCK_SIZE: tl.constexpr, NUM_STAGES: tl.constexpr):
```
- **EN:** Defines the helper function `vecadd_kernel`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `output_ptr`, `n_elements`, `num_blocks`, `BLOCK_SIZE`, `NUM_STAGES`. Key calls include `tl.program_id`, `tl.arange`, `tl.load`, `tl.store`. This scope touches Triton compilation or JIT kernels, Triton language operations, plugin or compiler extension points.
- **CN:** 定义辅助函数 `vecadd_kernel`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`output_ptr`、`n_elements`、`num_blocks`、`BLOCK_SIZE`、`NUM_STAGES`。 关键调用包括 `tl.program_id`、`tl.arange`、`tl.load`、`tl.store`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、插件或编译器扩展点。

#### Lines 102-104

```python
    pid = tl.program_id(axis=0)
    block_start = pid * BLOCK_SIZE * num_blocks
    offsets = block_start + tl.arange(0, BLOCK_SIZE)
```
- **EN:** Prepares or updates state through `pid`, `block_start`, `offsets`. Invokes `tl.program_id`, `tl.arange` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid`、`block_start`、`offsets` 准备或更新状态。 调用 `tl.program_id`、`tl.arange` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 105-111

```python
    for _ in tl.range(0, num_blocks, num_stages=NUM_STAGES):
        mask = offsets < n_elements
        x = tl.load(a_ptr + offsets, mask=mask)
        y = tl.load(b_ptr + offsets, mask=mask)
        output = x + y
        tl.store(output_ptr + offsets, output, mask=mask)
        offsets += BLOCK_SIZE
```
- **EN:** Invokes `tl.load`, `tl.store` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.load`、`tl.store` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

### Lines 112-126

```python


@triton.jit
def mxfp_to_bf16_kernel(
    x_ptr,
    scale_ptr,
    mxfp_ptr,
    N,
    e_bits: tl.constexpr,
    m_bits: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    # x.shape ==     (N, 32) for fp8 or (N, 16) for fp4
    # scale.shape == (N,)
    # out.shape   == (N, 32)
```
- **EN:** Defines the helper function `mxfp_to_bf16_kernel`. Decorators: `triton.jit`. Parameters: `x_ptr`, `scale_ptr`, `mxfp_ptr`, `N`, `e_bits`, `m_bits`, `BLOCK_SIZE`. Key calls include `tl.load`, `tl.static_assert`, `tl.where`, `tl.store`, `tl.arange`, `tl.ravel`, and 8 more. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `mxfp_to_bf16_kernel`。 装饰器：`triton.jit`。 参数：`x_ptr`、`scale_ptr`、`mxfp_ptr`、`N`、`e_bits`、`m_bits`、`BLOCK_SIZE`。 关键调用包括 `tl.load`、`tl.static_assert`、`tl.where`、`tl.store`、`tl.arange`、`tl.ravel` 等另外 8 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 127-143

```python
    is_fp8: tl.constexpr = e_bits + m_bits == 7
    # fp8: BLOCK_SIZE -> BLOCK_SIZE // 32, 32
    # fp4: BLOCK_SIZE // 2 -> BLOCK_SIZE // 32 , 16
    PARALLEL_DIM: tl.constexpr = BLOCK_SIZE // 32
    LAST_DIM: tl.constexpr = 32 if is_fp8 else 16
    LOAD_SIZE: tl.constexpr = LAST_DIM * PARALLEL_DIM

    offsets = (tl.program_id(0) * LOAD_SIZE + tl.arange(0, PARALLEL_DIM)[:, None] * LAST_DIM +
               tl.arange(0, LAST_DIM)[None, :])
    x = tl.load(x_ptr + offsets, mask=offsets < N * LAST_DIM)

    offsets = tl.program_id(0) * PARALLEL_DIM + tl.arange(0, PARALLEL_DIM)[:, None]
    scale = tl.load(scale_ptr + offsets, mask=offsets < N)
    tl.static_assert(scale.dtype == tl.uint8)
    tl.static_assert(x.dtype == tl.uint8)

    scale_bf16 = (scale.to(tl.uint16) << 7).to(tl.bfloat16, bitcast=True)
```
- **EN:** Prepares or updates state through `is_fp8`, `PARALLEL_DIM`, `LAST_DIM`, `LOAD_SIZE`, `offsets`, `x`, `scale`, `scale_bf16`. Invokes `tl.arange`, `tl.program_id`, `tl.load`, `tl.static_assert`, `scale.to` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `is_fp8`、`PARALLEL_DIM`、`LAST_DIM`、`LOAD_SIZE`、`offsets`、`x`、`scale`、`scale_bf16` 准备或更新状态。 调用 `tl.arange`、`tl.program_id`、`tl.load`、`tl.static_assert`、`scale.to` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 144-174

```python
    if is_fp8:
        if e_bits == 5 and m_bits == 2:
            x_f8 = x.to(tl.float8e5, bitcast=True)
            x_bf16 = x_f8.to(tl.bfloat16)
            # Preserve infs and nans. FIXME Fp8E5M2_to_Bf16 doesn't preserve them!
            non_finite_mask: tl.constexpr = ((1 << e_bits) - 1) << m_bits
            non_finite_mask_bf16: tl.constexpr = ((1 << 8) - 1) << 7
            x_bf16 = tl.where(
                x & non_finite_mask == non_finite_mask,
                (x_bf16.to(tl.uint16, bitcast=True) | non_finite_mask_bf16).to(tl.bfloat16, bitcast=True),
                x_bf16,
            )
        else:
            tl.static_assert(e_bits == 4 and m_bits == 3)
            x_f8 = x.to(tl.float8e4nv, bitcast=True)
            x_bf16 = x_f8.to(tl.bfloat16)
    else:
        # e2m1
        em0 = x & 0x7
        em1 = x & 0x70
        x0 = (em0.to(tl.uint16) << 2 + 4) | ((x & 0x8).to(tl.uint16) << 8 + 4)
        x1 = (em1.to(tl.uint16) << (2)) | ((x & 0x80).to(tl.uint16) << (8))
        # Three cases:
        # 1) x is normal and non-zero: Correct bias
        x0 = tl.where((em0 & 0x6) != 0, x0 + ((127 - 1) << 7), x0)
        x1 = tl.where((em1 & 0x60) != 0, x1 + ((127 - 1) << 7), x1)
        # 2) x is subnormal (x == 0bs001 where s is the sign): Map to +-0.5 in bf16
        x0 = tl.where(em0 == 0x1, 16128 | (x0 & 0x8000), x0)
        x1 = tl.where(em1 == 0x10, 16128 | (x1 & 0x8000), x1)
        # 3) x is zero, do nothing
        x_bf16 = tl.interleave(x0, x1).to(tl.bfloat16, bitcast=True)
```
- **EN:** Invokes `tl.where`, `x.to`, `x_f8.to`, `tl.static_assert`, `em0.to`, `em1.to`, and 2 more to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.where`、`x.to`、`x_f8.to`、`tl.static_assert`、`em0.to`、`em1.to` 等另外 2 项 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

#### Lines 175-181

```python
    # Multiplication preserves infs and NaNs in x_bf16
    mxfp = x_bf16 * scale_bf16
    # If scale is NaN, we encode it as an bf16 inf, so we need to correct for that
    mxfp = tl.where(scale == 0xFF, float("nan"), mxfp)

    offsets = tl.program_id(0) * BLOCK_SIZE + tl.arange(0, BLOCK_SIZE)
    tl.store(mxfp_ptr + offsets, tl.ravel(mxfp), mask=offsets < N * 32)
```
- **EN:** Prepares or updates state through `mxfp`, `offsets`. Invokes `tl.where`, `tl.arange`, `tl.program_id`, `tl.store`, `tl.ravel` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `mxfp`、`offsets` 准备或更新状态。 调用 `tl.where`、`tl.arange`、`tl.program_id`、`tl.store`、`tl.ravel` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 182-184

```python


def dot_scale_ref(x, scale, y, type_x, type_y):
```
- **EN:** Defines the helper function `dot_scale_ref`. Parameters: `x`, `scale`, `y`, `type_x`, `type_y`. Nested definitions in this scope: `AccumulateInFp32`. Key calls include `x.contiguous`, `x.new_empty`, `x_upcast.numel`, `scale.numel`, `AccumulateInFp32`, `torch.matmul`, and 1 more. This scope touches PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 定义辅助函数 `dot_scale_ref`。 参数：`x`、`scale`、`y`、`type_x`、`type_y`。 该作用域中的嵌套定义：`AccumulateInFp32`。 关键调用包括 `x.contiguous`、`x.new_empty`、`x_upcast.numel`、`scale.numel`、`AccumulateInFp32`、`torch.matmul` 等另外 1 项。 该作用域涉及PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 185-197

```python
    e_bits, m_bits = {"e2m1": (2, 1), "e4m3": (4, 3), "e5m2": (5, 2)}[type_x]
    type_fp8_y = {"e4m3": torch.float8_e4m3fn, "e5m2": torch.float8_e5m2, "bf16": torch.bfloat16}[type_y]

    out_dtype = torch.bfloat16

    x = x.contiguous()
    x_upcast = x.new_empty(scale.shape[:-1] + (32 * scale.shape[-1], ), dtype=out_dtype)

    N = x_upcast.numel()
    BLOCK_SIZE = 512
    grid = ((N + BLOCK_SIZE - 1) // BLOCK_SIZE, )
    mxfp_to_bf16_kernel[grid](x, scale, x_upcast, scale.numel(), e_bits, m_bits, BLOCK_SIZE, num_warps=4)
    y_upcast = y if type_y == "bf16" else y.view(type_fp8_y).to(out_dtype)
```
- **EN:** Prepares or updates state through `e_bits`, `m_bits`, `type_fp8_y`, `out_dtype`, `x`, `x_upcast`, `N`, `BLOCK_SIZE`, and 2 more. Invokes `x.contiguous`, `x.new_empty`, `x_upcast.numel`, `scale.numel`, `y.view` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `e_bits`、`m_bits`、`type_fp8_y`、`out_dtype`、`x`、`x_upcast`、`N`、`BLOCK_SIZE` 等另外 2 项 准备或更新状态。 调用 `x.contiguous`、`x.new_empty`、`x_upcast.numel`、`scale.numel`、`y.view` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 198-199

```python
    assert x_upcast.dtype == out_dtype
    assert y_upcast.dtype == out_dtype
```
- **EN:** Validates behavior with 2 assertion(s).
- **CN:** 通过 2 个断言验证行为。

#### Lines 200-202

```python

    class AccumulateInFp32:
```
- **EN:** Defines class `AccumulateInFp32`. Methods: `__enter__`, `__exit__`.
- **CN:** 定义类 `AccumulateInFp32`。 方法：`__enter__`、`__exit__`。

##### Lines 203-203

```python
        def __enter__(self):
```
- **EN:** Defines the helper function `__enter__`. Parameters: `self`. This scope touches PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 定义辅助函数 `__enter__`。 参数：`self`。 该作用域涉及PyTorch 张量准备与校验、矩阵乘法工作流。

###### Lines 204-205

```python
            self.prev_value = torch.backends.cuda.matmul.allow_bf16_reduced_precision_reduction
            torch.backends.cuda.matmul.allow_bf16_reduced_precision_reduction = False
```
- **EN:** Prepares or updates state through `self`, `torch`. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 通过 `self`、`torch` 准备或更新状态。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

##### Lines 206-207

```python

        def __exit__(self, exc_type, exc_val, exc_tb):
```
- **EN:** Defines the helper function `__exit__`. Parameters: `self`, `exc_type`, `exc_val`, `exc_tb`. This scope touches PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 定义辅助函数 `__exit__`。 参数：`self`、`exc_type`、`exc_val`、`exc_tb`。 该作用域涉及PyTorch 张量准备与校验、矩阵乘法工作流。

###### Lines 208-208

```python
            torch.backends.cuda.matmul.allow_bf16_reduced_precision_reduction = self.prev_value
```
- **EN:** Prepares or updates state through `torch`. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 通过 `torch` 准备或更新状态。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 209-211

```python

    with AccumulateInFp32():
        return torch.matmul(x_upcast, y_upcast)
```
- **EN:** Invokes `AccumulateInFp32`, `torch.matmul` to execute the test logic. Uses context managers to control resources or expectations. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 调用 `AccumulateInFp32`、`torch.matmul` 执行测试逻辑。 使用上下文管理器控制资源或预期行为。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

### Lines 212-215

```python


@pytest.mark.parametrize("scale", [True, False])
def test_pipeline_matmul(scale, device):
```
- **EN:** Defines the test function `test_pipeline_matmul`. Decorators: `pytest.mark.parametrize('scale', [True, False])`. Parameters: `scale`, `device`. Key calls include `pytest.mark.parametrize`, `check_capabilities`, `torch.testing.assert_close`, `is_cuda`, `pytest.skip`, `torch.randint`, and 17 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, matrix multiplication workflows.
- **CN:** 定义测试函数 `test_pipeline_matmul`。 装饰器：`pytest.mark.parametrize('scale', [True, False])`。 参数：`scale`、`device`。 关键调用包括 `pytest.mark.parametrize`、`check_capabilities`、`torch.testing.assert_close`、`is_cuda`、`pytest.skip`、`torch.randint` 等另外 17 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、矩阵乘法工作流。

#### Lines 216-216

```python
    check_capabilities()
```
- **EN:** Invokes `check_capabilities` to execute the test logic.
- **CN:** 调用 `check_capabilities` 执行测试逻辑。

#### Lines 217-218

```python
    if scale and not (is_cuda() or is_hip_cdna()):
        pytest.skip("NYI: scale_dot just implemented in CUDA/HIP")
```
- **EN:** Invokes `pytest.skip`, `is_cuda`, `is_hip_cdna` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `pytest.skip`、`is_cuda`、`is_hip_cdna` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 219-221

```python
    M, N, K = 512, 512, 128
    BLOCK_M, BLOCK_N, BLOCK_K = 64, 64, 32
    NUM_STAGES = 4 if is_cuda() else 2
```
- **EN:** Prepares or updates state through `M`, `N`, `K`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `NUM_STAGES`. Invokes `is_cuda` to execute the test logic. Relevant themes: plugin or compiler extension points.
- **CN:** 通过 `M`、`N`、`K`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`NUM_STAGES` 准备或更新状态。 调用 `is_cuda` 执行测试逻辑。 相关主题：插件或编译器扩展点。

#### Lines 222-251

```python

    if scale:
        # Large enough tile to let our heuristics to pipeline small tensor kick in
        # for the scales
        BLOCK_M = 256
        BLOCK_K = 128
        K = BLOCK_K * NUM_STAGES
        a_type = "e2m1"
        DIV_FACTOR = 2 if a_type == "e2m1" else 1
        a = torch.randint(256, (M, K // DIV_FACTOR), device=device, dtype=torch.uint8)
        # Sample small-ish scales to avoid overflow
        scale_a = torch.randint(74, (M, K // 32), device=device, dtype=torch.uint8)
        # Use e5m2 for Ampere, as it does not support fp_to_fp conversions for fp8e4m3
        # Use bf16 for Hopper as the rhs must come from shmem
        b_type = "bf16" if is_hopper_or_newer() else "e5m2"
        if b_type == "bf16":
            b = torch.randn((K, N), device=device, dtype=torch.bfloat16)
        else:
            b = torch.randint(256, (K, N), device=device, dtype=torch.uint8)
            # e5m2 has too many non-finite values when sampled uniformly (1 / 32) and
            # Fp8E5M2_to_Bf16 doesn't preserve NaNs (fixme)
            finite = torch.arange(K * N, device=device, dtype=torch.uint8).reshape(K, N) % 0x7C
            b = torch.where(b & 0x7C == 0x7C, finite | (0x80 & b), b)
        output = torch.empty((M, N), dtype=torch.bfloat16, device=device)
    else:
        a = torch.randn(M, K, device=device, dtype=torch.float16)
        b = torch.randn(K, N, device=device, dtype=torch.float16)
        scale_a = None
        a_type, b_type = None, None
        output = torch.empty((M, N), dtype=torch.float16, device=device)
```
- **EN:** Invokes `torch.randint`, `torch.empty`, `torch.randn`, `is_hopper_or_newer`, `torch.where`, `torch.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, plugin or compiler extension points, random-data generation.
- **CN:** 调用 `torch.randint`、`torch.empty`、`torch.randn`、`is_hopper_or_newer`、`torch.where`、`torch.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、插件或编译器扩展点、随机数据生成。

#### Lines 252-253

```python
    grid = (triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N), 1)
    use_tma = not scale and is_hopper_or_newer()
```
- **EN:** Prepares or updates state through `grid`, `use_tma`. Invokes `triton.cdiv`, `is_hopper_or_newer` to execute the test logic.
- **CN:** 通过 `grid`、`use_tma` 准备或更新状态。 调用 `triton.cdiv`、`is_hopper_or_newer` 执行测试逻辑。

#### Lines 254-269

```python

    if use_tma:
        from triton.tools.tensor_descriptor import TensorDescriptor
        a_tma = TensorDescriptor.from_tensor(a, block_shape=[BLOCK_M, BLOCK_K])
        b_tma = TensorDescriptor.from_tensor(b, block_shape=[BLOCK_K, BLOCK_N])
        output_tma = TensorDescriptor.from_tensor(output, block_shape=[BLOCK_M, BLOCK_N])
        handler = matmul_kernel_tma[grid](a_tma, b_tma, output_tma, M, N, K, BLOCK_M, BLOCK_N, BLOCK_K,
                                          NUM_STAGES=NUM_STAGES)
    else:
        # Pass K_MXFP to make explicit that KB is multiple of 32 and KA is multiple of 16 or 32º
        if scale:
            K = scale_a.shape[-1]
        stride_sm, stride_sk = scale_a.stride() if scale else (0, 0)
        handler = matmul_kernel[grid](a, scale_a, b, output, M, N, K, a.stride(0), a.stride(1), stride_sm, stride_sk,
                                      b.stride(0), b.stride(1), output.stride(0), output.stride(1), BLOCK_M, BLOCK_N,
                                      BLOCK_K, NUM_STAGES=NUM_STAGES, a_type=a_type, b_type=b_type)
```
- **EN:** Invokes `TensorDescriptor.from_tensor`, `scale_a.stride`, `a.stride`, `b.stride`, `output.stride` to execute the test logic. Branches on runtime or test conditions. Relevant themes: tensor/descriptor metadata, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 调用 `TensorDescriptor.from_tensor`、`scale_a.stride`、`a.stride`、`b.stride`、`output.stride` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：张量/描述符元数据、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 270-273

```python
    if scale:
        ref_out = dot_scale_ref(a, scale_a, b, a_type, b_type)
    else:
        ref_out = torch.matmul(a, b)
```
- **EN:** Invokes `dot_scale_ref`, `torch.matmul` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 调用 `dot_scale_ref`、`torch.matmul` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 274-279

```python
    # Bigger tolerance for AMD CDNA2 devices.
    # CDNA2 devices use reduced precision fp16 and bf16 and flush input and
    # output denormal values to zero. Detailed info is at: https://pytorch.org/docs/stable/notes/numerical_accuracy.html#reduced-precision-fp16-and-bf16-gemms-and-convolutions-on-amd-instinct-mi200-devices
    atol = 1e-2 if is_hip_cdna2() or scale else None
    rtol = 1e-2 if is_hip_cdna2() or scale else None
    torch.testing.assert_close(ref_out, output, atol=atol, rtol=rtol, equal_nan=scale)
```
- **EN:** Prepares or updates state through `atol`, `rtol`. Invokes `is_hip_cdna2`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 通过 `atol`、`rtol` 准备或更新状态。 调用 `is_hip_cdna2`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 280-318

```python
    if is_cuda():
        ttgir = handler.asm["ttgir"]
        if use_tma:
            assert ttgir.count("ttng.async_tma_copy_global_to_local") != 0, "async tma copy not found"
            assert ttgir.count(f"num = {NUM_STAGES} : i32") == 0, "num_stages not match"
            assert ttgir.count("ttng.barrier_expect") != 0, "barrier_expect not found"
            assert ttgir.count("ttng.wait_barrier") != 0, "wait_barrier not found"

            if torch.cuda.get_device_capability()[0] == 9:
                # a_tma, b_tma, output_tma, barriar_tma
                assert ttgir.count("ttg.local_alloc") == 4, "alloc number not match"
                assert ttgir.count("ttng.warp_group_dot") != 0, "warp_group_dot not found"
            elif torch.cuda.get_device_capability()[0] == 10:
                # a_tma, b_tma, output_tma, barriar_tma, barriar_mma
                assert ttgir.count("ttg.local_alloc") == 5, "alloc number not match"
                assert ttgir.count("ttng.tc_gen5_mma") != 0, "warp_group_dot not found"
        else:
            # 1. check async
            assert ttgir.count("ttg.async_copy_global_to_local") != 0, "async copy not found"
            # 2. check sync point
            assert ttgir.count("num = 0 : i32") == 1, "only one sync point for the loads after the loop"
            # 3. check alloc
            if torch.cuda.get_device_capability()[0] == 10:
                if scale:
                    # A, B, scale, decomposed A shmem
                    count = 4
                else:
                    # A, B, MMA barrier
                    count = 3
                assert ttgir.count("ttg.local_alloc") == count, "alloc number not match"
            else:
                assert ttgir.count("ttg.local_alloc") == (3 if scale else 2), "alloc number not match"

            # 4. check dot
            cc = torch.cuda.get_device_capability()
            if cc[0] == 9:
                assert ttgir.count("ttng.warp_group_dot") != 0, "warp_group_dot not found"
            elif cc[0] < 9:
                assert ttgir.count("ttg.dot") != 0, "dot not found"
```
- **EN:** Invokes `is_cuda`, `torch.cuda.get_device_capability`, `ttgir.count` to execute the test logic. Validates behavior with 14 assertion(s). Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, plugin or compiler extension points.
- **CN:** 调用 `is_cuda`、`torch.cuda.get_device_capability`、`ttgir.count` 执行测试逻辑。 通过 14 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、插件或编译器扩展点。

### Lines 319-321

```python


def test_pipeline_vecadd(device):
```
- **EN:** Defines the test function `test_pipeline_vecadd`. Parameters: `device`. Key calls include `check_capabilities`, `torch.randn`, `torch.empty`, `torch.testing.assert_close`, `is_cuda`, `triton.cdiv`, and 1 more. This scope touches PyTorch tensor setup and checks, plugin or compiler extension points, random-data generation.
- **CN:** 定义测试函数 `test_pipeline_vecadd`。 参数：`device`。 关键调用包括 `check_capabilities`、`torch.randn`、`torch.empty`、`torch.testing.assert_close`、`is_cuda`、`triton.cdiv` 等另外 1 项。 该作用域涉及PyTorch 张量准备与校验、插件或编译器扩展点、随机数据生成。

#### Lines 322-333

```python
    check_capabilities()
    SIZE = 4096
    NUM_BLOCKS = 4
    BLOCK_SIZE = 256
    NUM_STAGES = 3
    a = torch.randn(SIZE, dtype=torch.float16, device=device)
    b = torch.randn(SIZE, dtype=torch.float16, device=device)
    output = torch.empty(SIZE, dtype=torch.float16, device=device)
    grid = (triton.cdiv(SIZE, NUM_BLOCKS * BLOCK_SIZE), 1)
    handler = vecadd_kernel[grid](a, b, output, SIZE, NUM_BLOCKS, BLOCK_SIZE, NUM_STAGES)
    ref_out = a + b
    torch.testing.assert_close(ref_out, output)
```
- **EN:** Prepares or updates state through `SIZE`, `NUM_BLOCKS`, `BLOCK_SIZE`, `NUM_STAGES`, `a`, `b`, `output`, `grid`, and 2 more. Invokes `check_capabilities`, `torch.randn`, `torch.empty`, `triton.cdiv`, `torch.testing.assert_close` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, plugin or compiler extension points, random-data generation.
- **CN:** 通过 `SIZE`、`NUM_BLOCKS`、`BLOCK_SIZE`、`NUM_STAGES`、`a`、`b`、`output`、`grid` 等另外 2 项 准备或更新状态。 调用 `check_capabilities`、`torch.randn`、`torch.empty`、`triton.cdiv`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、插件或编译器扩展点、随机数据生成。

#### Lines 334-339

```python
    if is_cuda():
        ttgir = handler.asm["ttgir"]
        # 1. check number of stages
        assert ttgir.count("ttg.async_copy_global_to_local") / 2 == NUM_STAGES, "num_stages not match"
        # 2. check alloc
        assert ttgir.count("ttg.local_alloc") == 2, "alloc number not match"
```
- **EN:** Invokes `is_cuda`, `ttgir.count` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `is_cuda`、`ttgir.count` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

### Lines 340-346

```python


@pytest.mark.parametrize("ROW_COUNT", [0, 1, 2, 3])
@pytest.mark.parametrize("NUM_STAGES", [1, 2, 3, 4, 5])
def test_pipeline_epilogue(ROW_COUNT, NUM_STAGES, device):

    @triton.jit
```
- **EN:** Defines the test function `test_pipeline_epilogue`. Decorators: `pytest.mark.parametrize('ROW_COUNT', [0, 1, 2, 3])`, `pytest.mark.parametrize('NUM_STAGES', [1, 2, 3, 4, 5])`. Parameters: `ROW_COUNT`, `NUM_STAGES`, `device`. Nested definitions in this scope: `kernel_up`. Key calls include `pytest.mark.parametrize`, `torch.zeros`, `torch.rand_like`, `triton.next_power_of_2`, `tl.num_programs`, `tl.arange`, and 5 more. This scope touches pytest parametrization, Triton compilation or JIT kernels, Triton language operations, PyTorch tensor setup and checks.
- **CN:** 定义测试函数 `test_pipeline_epilogue`。 装饰器：`pytest.mark.parametrize('ROW_COUNT', [0, 1, 2, 3])`、`pytest.mark.parametrize('NUM_STAGES', [1, 2, 3, 4, 5])`。 参数：`ROW_COUNT`、`NUM_STAGES`、`device`。 该作用域中的嵌套定义：`kernel_up`。 关键调用包括 `pytest.mark.parametrize`、`torch.zeros`、`torch.rand_like`、`triton.next_power_of_2`、`tl.num_programs`、`tl.arange` 等另外 5 项。 该作用域涉及pytest 参数化、Triton 编译或 JIT kernel、Triton language 操作、PyTorch 张量准备与校验。

#### Lines 346-348

```python
    @triton.jit
    def kernel_up(output_ptr, input_ptr, input_row_stride, output_row_stride, n_rows, n_cols, BLOCK_SIZE: tl.constexpr,
                  NUM_STAGES: tl.constexpr):
```
- **EN:** Defines the helper function `kernel_up`. Decorators: `triton.jit`. Parameters: `output_ptr`, `input_ptr`, `input_row_stride`, `output_row_stride`, `n_rows`, `n_cols`, `BLOCK_SIZE`, `NUM_STAGES`. Key calls include `tl.num_programs`, `tl.arange`, `tl.load`, `tl.store`. This scope touches Triton compilation or JIT kernels, Triton language operations, plugin or compiler extension points.
- **CN:** 定义辅助函数 `kernel_up`。 装饰器：`triton.jit`。 参数：`output_ptr`、`input_ptr`、`input_row_stride`、`output_row_stride`、`n_rows`、`n_cols`、`BLOCK_SIZE`、`NUM_STAGES`。 关键调用包括 `tl.num_programs`、`tl.arange`、`tl.load`、`tl.store`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、插件或编译器扩展点。

##### Lines 349-351

```python
        row_step = tl.num_programs(0)
        col_offsets = tl.arange(0, BLOCK_SIZE)
        mask = col_offsets < n_cols
```
- **EN:** Prepares or updates state through `row_step`, `col_offsets`, `mask`. Invokes `tl.num_programs`, `tl.arange` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `row_step`、`col_offsets`、`mask` 准备或更新状态。 调用 `tl.num_programs`、`tl.arange` 执行测试逻辑。 相关主题：Triton language 操作。

##### Lines 352-359

```python
        for row_idx in tl.range(0, n_rows, row_step, num_stages=NUM_STAGES):
            row_start_ptr = input_ptr + row_idx * input_row_stride
            input_ptrs = row_start_ptr + col_offsets
            val = tl.load(input_ptrs, mask=mask, other=-float('inf'))
            val += 1.0
            output_row_start_ptr = output_ptr + row_idx * output_row_stride
            output_ptrs = output_row_start_ptr + col_offsets
            tl.store(output_ptrs, val, mask=mask)
```
- **EN:** Invokes `tl.load`, `tl.store` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.load`、`tl.store` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

#### Lines 360-367

```python

    width = ROW_COUNT
    depth = 78
    x = torch.zeros(width, depth, device=device)
    y0 = torch.rand_like(x)
    n_rows, n_cols = x.shape
    BLOCK_SIZE = triton.next_power_of_2(n_cols)
    kernel_up[(1, )](y0, x, x.stride(0), y0.stride(0), n_rows, n_cols, BLOCK_SIZE, NUM_STAGES)
```
- **EN:** Prepares or updates state through `width`, `depth`, `x`, `y0`, `n_rows`, `n_cols`, `BLOCK_SIZE`. Invokes `torch.zeros`, `torch.rand_like`, `triton.next_power_of_2`, `x.stride`, `y0.stride` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, plugin or compiler extension points, random-data generation.
- **CN:** 通过 `width`、`depth`、`x`、`y0`、`n_rows`、`n_cols`、`BLOCK_SIZE` 准备或更新状态。 调用 `torch.zeros`、`torch.rand_like`、`triton.next_power_of_2`、`x.stride`、`y0.stride` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、插件或编译器扩展点、随机数据生成。

#### Lines 368-368

```python
    assert (y0 == torch.ones_like(x)).all()
```
- **EN:** Invokes `torch.ones_like` to execute the test logic. Validates behavior with 1 assertion(s). Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.ones_like` 执行测试逻辑。 通过 1 个断言验证行为。 相关主题：PyTorch 张量准备与校验。

### Lines 369-371

```python


def random_bfloat16(shape, device):
```
- **EN:** Defines the helper function `random_bfloat16`. Parameters: `shape`, `device`. Key calls include `torch.randn`, `torch.round`. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义辅助函数 `random_bfloat16`。 参数：`shape`、`device`。 关键调用包括 `torch.randn`、`torch.round`。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

#### Lines 372-382

```python
    """
    Creates a random bfloat16 tensor where every element is a multiple of 1/8.
    This should avoid floating-point errors in downstream calculations, allowing
    for exact comparisons.
    """

    X = torch.randn(shape, device=device, dtype=torch.bfloat16)
    X *= 8.0
    X = torch.round(X)
    X *= 0.125
    return X
```
- **EN:** Prepares or updates state through `X`. Invokes `torch.randn`, `torch.round` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `X` 准备或更新状态。 调用 `torch.randn`、`torch.round` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

### Lines 383-400

```python


@triton.jit
def indirect_matmul_kernel(
    Out,
    stride_out1,
    A,
    stride_a1,
    B,
    stride_b1,
    Indices,
    K,

    # output tile size:
    BLOCK_M: tl.constexpr,
    BLOCK_K: tl.constexpr,
    BLOCK_N: tl.constexpr,
):
```
- **EN:** Defines the helper function `indirect_matmul_kernel`. Decorators: `triton.jit`. Parameters: `Out`, `stride_out1`, `A`, `stride_a1`, `B`, `stride_b1`, `Indices`, `K`, and 3 more. Key calls include `tl.arange`, `tl.zeros`, `tl.store`, `tl.load`, `tl.dot`. This scope touches Triton compilation or JIT kernels, Triton language operations, matrix multiplication workflows.
- **CN:** 定义辅助函数 `indirect_matmul_kernel`。 装饰器：`triton.jit`。 参数：`Out`、`stride_out1`、`A`、`stride_a1`、`B`、`stride_b1`、`Indices`、`K` 等另外 3 项。 关键调用包括 `tl.arange`、`tl.zeros`、`tl.store`、`tl.load`、`tl.dot`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、矩阵乘法工作流。

#### Lines 401-409

```python
    index_ptrs = Indices + tl.arange(0, BLOCK_K)

    m_offs = tl.arange(0, BLOCK_M)
    n_offs = tl.arange(0, BLOCK_N)[None, :]

    A_ptrs = A + n_offs
    B_ptrs = B + m_offs

    acc = tl.zeros([BLOCK_M, BLOCK_N], tl.float32)
```
- **EN:** Prepares or updates state through `index_ptrs`, `m_offs`, `n_offs`, `A_ptrs`, `B_ptrs`, `acc`. Invokes `tl.arange`, `tl.zeros` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `index_ptrs`、`m_offs`、`n_offs`、`A_ptrs`、`B_ptrs`、`acc` 准备或更新状态。 调用 `tl.arange`、`tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 410-417

```python
    for k in range(0, K, BLOCK_K):
        idx = tl.load(index_ptrs)

        a = tl.load(A_ptrs + idx[:, None] * stride_a1)
        b = tl.load(B_ptrs + idx[:, None] * stride_b1)

        acc = tl.dot(b.T, a, acc=acc)
        index_ptrs += BLOCK_K
```
- **EN:** Invokes `tl.load`, `tl.dot` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.load`、`tl.dot` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作。

#### Lines 418-421

```python

    # now write out the accumulator:
    Out_ptrs = Out + m_offs[:, None] + n_offs * stride_out1
    tl.store(Out_ptrs, acc)
```
- **EN:** Prepares or updates state through `Out_ptrs`. Invokes `tl.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `Out_ptrs` 准备或更新状态。 调用 `tl.store` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 422-426

```python


@pytest.mark.parametrize("BLOCK_M, BLOCK_N, BLOCK_K", [(128, 128, 128), (128, 128, 64), (128, 64, 128)])
@pytest.mark.parametrize("num_stages", [1, 3, 5])
def test_indirect_matmul(BLOCK_M, BLOCK_N, BLOCK_K, num_stages, device):
```
- **EN:** Defines the test function `test_indirect_matmul`. Decorators: `pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 128), (128, 128, 64), (128, 64, 128)])`, `pytest.mark.parametrize('num_stages', [1, 3, 5])`. Parameters: `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `num_stages`, `device`. Key calls include `pytest.mark.parametrize`, `random_bfloat16`, `torch.arange`, `torch.empty`, `torch.matmul`, `torch.testing.assert_close`, and 7 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_indirect_matmul`。 装饰器：`pytest.mark.parametrize('BLOCK_M, BLOCK_N, BLOCK_K', [(128, 128, 128), (128, 128, 64), (128, 64, 128)])`、`pytest.mark.parametrize('num_stages', [1, 3, 5])`。 参数：`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`num_stages`、`device`。 关键调用包括 `pytest.mark.parametrize`、`random_bfloat16`、`torch.arange`、`torch.empty`、`torch.matmul`、`torch.testing.assert_close` 等另外 7 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 427-428

```python
    if (num_stages > 3 or (num_stages >= 3 and (BLOCK_M, BLOCK_N, BLOCK_K) == (128, 128, 128))) and is_hip():
        pytest.skip("Not enough shared memory on HIP.")
```
- **EN:** Invokes `is_hip`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `is_hip`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

#### Lines 429-457

```python
    M = BLOCK_M
    N = BLOCK_N

    K = BLOCK_K * 2
    A = random_bfloat16((K, N), device=device)
    B = random_bfloat16((K, M), device=device)

    # Use arange for indices so it's numerically just a matmul
    Indices = torch.arange(K, device=device)
    Out = torch.empty((N, M), device=device, dtype=torch.float32)

    expect = torch.matmul(A.mT.to(torch.float32), B.to(torch.float32))

    indirect_matmul_kernel[(1, )](
        Out,
        Out.stride(0),
        A,
        A.stride(0),
        B,
        B.stride(0),
        Indices,
        K,
        BLOCK_M,
        BLOCK_K,
        BLOCK_N,
        num_warps=4,
        num_stages=num_stages,
    )
    torch.testing.assert_close(expect, Out)
```
- **EN:** Prepares or updates state through `M`, `N`, `K`, `A`, `B`, `Indices`, `Out`, `expect`. Invokes `random_bfloat16`, `torch.arange`, `torch.empty`, `torch.matmul`, `A.mT.to`, `B.to`, and 4 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points, random-data generation.
- **CN:** 通过 `M`、`N`、`K`、`A`、`B`、`Indices`、`Out`、`expect` 准备或更新状态。 调用 `random_bfloat16`、`torch.arange`、`torch.empty`、`torch.matmul`、`A.mT.to`、`B.to` 等另外 4 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点、随机数据生成。

### Lines 458-468

```python


@triton.jit
def matmul_kernel_persistent_scatter(a_ptr, b_ptr, c_ptr,  #
                                     M, N, K,  #
                                     BLOCK_SIZE_M: tl.constexpr,  #
                                     BLOCK_SIZE_N: tl.constexpr,  #
                                     BLOCK_SIZE_K: tl.constexpr,  #
                                     GROUP_SIZE_M: tl.constexpr,  #
                                     NUM_SMS: tl.constexpr):  #
    # Matmul using TMA and device-side descriptor creation
```
- **EN:** Defines the helper function `matmul_kernel_persistent_scatter`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `M`, `N`, `K`, `BLOCK_SIZE_M`, `BLOCK_SIZE_N`, and 3 more. Key calls include `tl.program_id`, `tl.cdiv`, `tl.make_tensor_descriptor`, `min`, `tl.zeros`, `accumulator.to`, and 5 more. This scope touches Triton compilation or JIT kernels, Triton language operations, tensor/descriptor metadata, matrix multiplication workflows.
- **CN:** 定义辅助函数 `matmul_kernel_persistent_scatter`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`M`、`N`、`K`、`BLOCK_SIZE_M`、`BLOCK_SIZE_N` 等另外 3 项。 关键调用包括 `tl.program_id`、`tl.cdiv`、`tl.make_tensor_descriptor`、`min`、`tl.zeros`、`accumulator.to` 等另外 5 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、张量/描述符元数据、矩阵乘法工作流。

#### Lines 469-494

```python
    dtype = c_ptr.dtype.element_ty
    start_pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)
    k_tiles = tl.cdiv(K, BLOCK_SIZE_K)
    num_tiles = num_pid_m * num_pid_n
    num_pid_in_group = GROUP_SIZE_M * num_pid_n

    a_desc = tl.make_tensor_descriptor(
        a_ptr,
        shape=[M, K],
        strides=[K, 1],
        block_shape=[BLOCK_SIZE_M, BLOCK_SIZE_K],
    )
    b_desc = tl.make_tensor_descriptor(
        b_ptr,
        shape=[N, K],
        strides=[K, 1],
        block_shape=[BLOCK_SIZE_N, BLOCK_SIZE_K],
    )
    c_desc = tl.make_tensor_descriptor(
        c_ptr,
        shape=[M, N],
        strides=[N, 1],
        block_shape=[1, BLOCK_SIZE_N],
    )
```
- **EN:** Prepares or updates state through `dtype`, `start_pid`, `num_pid_m`, `num_pid_n`, `k_tiles`, `num_tiles`, `num_pid_in_group`, `a_desc`, and 2 more. Invokes `tl.program_id`, `tl.cdiv`, `tl.make_tensor_descriptor` to execute the test logic. Relevant themes: Triton language operations, tensor/descriptor metadata.
- **CN:** 通过 `dtype`、`start_pid`、`num_pid_m`、`num_pid_n`、`k_tiles`、`num_tiles`、`num_pid_in_group`、`a_desc` 等另外 2 项 准备或更新状态。 调用 `tl.program_id`、`tl.cdiv`、`tl.make_tensor_descriptor` 执行测试逻辑。 相关主题：Triton language 操作、张量/描述符元数据。

#### Lines 495-515

```python

    for tile_id in range(start_pid, num_tiles, NUM_SMS):
        group_id = tile_id // num_pid_in_group
        first_pid_m = group_id * GROUP_SIZE_M
        group_size_m = min(num_pid_m - first_pid_m, GROUP_SIZE_M)
        pid_m = first_pid_m + (tile_id % group_size_m)
        pid_n = (tile_id % num_pid_in_group) // group_size_m

        offs_am = pid_m * BLOCK_SIZE_M
        offs_bn = pid_n * BLOCK_SIZE_N

        accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
        for ki in range(k_tiles):
            offs_k = ki * BLOCK_SIZE_K

            a = a_desc.load([offs_am, offs_k])
            b = b_desc.load([offs_bn, offs_k])
            accumulator = tl.dot(a, b.T, accumulator)

        c = accumulator.to(dtype)
        c_desc.scatter(c, offs_am + tl.arange(0, BLOCK_SIZE_M), offs_bn)
```
- **EN:** Invokes `min`, `tl.zeros`, `accumulator.to`, `c_desc.scatter`, `a_desc.load`, `b_desc.load`, and 2 more to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations.
- **CN:** 调用 `min`、`tl.zeros`、`accumulator.to`、`c_desc.scatter`、`a_desc.load`、`b_desc.load` 等另外 2 项 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作。

### Lines 516-521

```python


@pytest.mark.skipif(torch.cuda.get_device_capability()[0] != 10,
                    reason="TMA Scatter only works on cloud Blackwell Chips")
def test_scatter_pipeline(device):
```
- **EN:** Defines the test function `test_scatter_pipeline`. Decorators: `pytest.mark.skipif(torch.cuda.get_device_capability()[0] != 10, reason='TMA Scatter only works on cloud Blackwell Chips')`. Parameters: `device`. Nested definitions in this scope: `alloc_fn`. Key calls include `pytest.mark.skipif`, `triton.set_allocator`, `min`, `torch.randn`, `torch.empty`, `torch.matmul`, and 4 more. This scope touches PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 定义测试函数 `test_scatter_pipeline`。 装饰器：`pytest.mark.skipif(torch.cuda.get_device_capability()[0] != 10, reason='TMA Scatter only works on cloud Blackwell Chips')`。 参数：`device`。 该作用域中的嵌套定义：`alloc_fn`。 关键调用包括 `pytest.mark.skipif`、`triton.set_allocator`、`min`、`torch.randn`、`torch.empty`、`torch.matmul` 等另外 4 项。 该作用域涉及PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

#### Lines 522-522

```python
    def alloc_fn(size, alignment, stream):
```
- **EN:** Defines the helper function `alloc_fn`. Parameters: `size`, `alignment`, `stream`. Key calls include `torch.empty`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `alloc_fn`。 参数：`size`、`alignment`、`stream`。 关键调用包括 `torch.empty`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 523-523

```python
        return torch.empty(size, device="cuda", dtype=torch.int8)
```
- **EN:** Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 524-542

```python

    triton.set_allocator(alloc_fn)

    M, N, K, = 1024, 1024, 1024
    BLOCK_M, BLOCK_N, BLOCK_K = 64, 64, 32
    GROUP_SIZE_M = 4

    NUM_SMS = torch.cuda.get_device_properties("cuda").multi_processor_count
    grid_x = min(NUM_SMS, triton.cdiv(M, BLOCK_M) * triton.cdiv(N, BLOCK_N))

    a = torch.randn(M, K, device=device, dtype=torch.float16)
    b = torch.randn(N, K, device=device, dtype=torch.float16)
    c = torch.empty((M, N), device=device, dtype=torch.float16)

    kernel = matmul_kernel_persistent_scatter[(grid_x, )](a, b, c, M, N, K, BLOCK_M, BLOCK_N, BLOCK_K, GROUP_SIZE_M,
                                                          NUM_SMS)

    ref = torch.matmul(a, b.T)
    torch.testing.assert_close(c, ref)
```
- **EN:** Prepares or updates state through `M`, `N`, `K`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `GROUP_SIZE_M`, `NUM_SMS`, and 6 more. Invokes `triton.set_allocator`, `torch.cuda.get_device_properties`, `min`, `triton.cdiv`, `torch.randn`, `torch.empty`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 通过 `M`、`N`、`K`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`GROUP_SIZE_M`、`NUM_SMS` 等另外 6 项 准备或更新状态。 调用 `triton.set_allocator`、`torch.cuda.get_device_properties`、`min`、`triton.cdiv`、`torch.randn`、`torch.empty` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

#### Lines 543-544

```python

    assert kernel.asm["ttgir"].count("tma_store_wait") == 2, "expected pipelined TMA scatter"
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 545-548

```python


@pytest.mark.parametrize("num_stages", [1, 2, 3])
def test_conditional_store_pipeline(num_stages, device):
```
- **EN:** Defines the test function `test_conditional_store_pipeline`. Decorators: `pytest.mark.parametrize('num_stages', [1, 2, 3])`. Parameters: `num_stages`, `device`. Nested definitions in this scope: `conditional_store_kernel`. Key calls include `pytest.mark.parametrize`, `check_capabilities`, `torch.arange`, `torch.zeros`, `torch.equal`, `tl.load`, and 2 more. This scope touches pytest parametrization, Triton compilation or JIT kernels, Triton language operations, PyTorch tensor setup and checks.
- **CN:** 定义测试函数 `test_conditional_store_pipeline`。 装饰器：`pytest.mark.parametrize('num_stages', [1, 2, 3])`。 参数：`num_stages`、`device`。 该作用域中的嵌套定义：`conditional_store_kernel`。 关键调用包括 `pytest.mark.parametrize`、`check_capabilities`、`torch.arange`、`torch.zeros`、`torch.equal`、`tl.load` 等另外 2 项。 该作用域涉及pytest 参数化、Triton 编译或 JIT kernel、Triton language 操作、PyTorch 张量准备与校验。

#### Lines 549-554

```python
    """
    Test for the conditional store pipelining bugfix.
    This reproduces the race condition where conditional code gets moved to epilogue cluster,
    causing users of loads to be scheduled in later clusters than the loads themselves.
    """
    check_capabilities()
```
- **EN:** Invokes `check_capabilities` to execute the test logic.
- **CN:** 调用 `check_capabilities` 执行测试逻辑。

#### Lines 555-563

```python

    @triton.jit
    def conditional_store_kernel(
        arange_ptr,
        output_ptr,
        loop_stages: tl.constexpr,
        N: tl.constexpr,
        always_true_but_not_constexpr,
    ):
```
- **EN:** Defines the helper function `conditional_store_kernel`. Decorators: `triton.jit`. Parameters: `arange_ptr`, `output_ptr`, `loop_stages`, `N`, `always_true_but_not_constexpr`. Key calls include `tl.load`, `tl.store`, `tl.arange`. This scope touches Triton compilation or JIT kernels, Triton language operations, plugin or compiler extension points.
- **CN:** 定义辅助函数 `conditional_store_kernel`。 装饰器：`triton.jit`。 参数：`arange_ptr`、`output_ptr`、`loop_stages`、`N`、`always_true_but_not_constexpr`。 关键调用包括 `tl.load`、`tl.store`、`tl.arange`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、插件或编译器扩展点。

##### Lines 564-567

```python
        for i in tl.range(0, N, num_stages=loop_stages):
            out_idx = tl.load(arange_ptr + i + tl.arange(0, 1))
            if always_true_but_not_constexpr:
                tl.store(output_ptr + out_idx, i + 1)
```
- **EN:** Invokes `tl.load`, `tl.store`, `tl.arange` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.load`、`tl.store`、`tl.arange` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

#### Lines 568-576

```python

    N = 17
    arange = torch.arange(N, dtype=torch.int32, device=device)
    output = torch.zeros((N, ), dtype=torch.int32, device=device)

    conditional_store_kernel[(1, )](arange, output, num_stages, N, True)

    # Expected output: [1, 2, 3, 4, ..., N]
    expected = torch.arange(1, N + 1, dtype=torch.int32, device=device)
```
- **EN:** Prepares or updates state through `N`, `arange`, `output`, `expected`. Invokes `torch.arange`, `torch.zeros` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, plugin or compiler extension points.
- **CN:** 通过 `N`、`arange`、`output`、`expected` 准备或更新状态。 调用 `torch.arange`、`torch.zeros` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、插件或编译器扩展点。

#### Lines 577-577

```python
    assert torch.equal(output, expected)
```
- **EN:** Invokes `torch.equal` to execute the test logic. Validates behavior with 1 assertion(s). Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.equal` 执行测试逻辑。 通过 1 个断言验证行为。 相关主题：PyTorch 张量准备与校验。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `check_capabilities`, `matmul_kernel`, `matmul_kernel_tma`, `vecadd_kernel`, `mxfp_to_bf16_kernel`, `dot_scale_ref`, `test_pipeline_matmul`, `test_pipeline_vecadd`
  **CN:** 顶层作用域，例如 `check_capabilities`、`matmul_kernel`、`matmul_kernel_tma`、`vecadd_kernel`、`mxfp_to_bf16_kernel`、`dot_scale_ref`、`test_pipeline_matmul`、`test_pipeline_vecadd`
- **EN:** pytest parametrization
  **CN:** pytest 参数化
- **EN:** Triton compilation or JIT kernels
  **CN:** Triton 编译或 JIT kernel
- **EN:** Triton language operations
  **CN:** Triton language 操作
- **EN:** PyTorch tensor setup and checks
  **CN:** PyTorch 张量准备与校验
- **EN:** tensor/descriptor metadata
  **CN:** 张量/描述符元数据
- **EN:** matrix multiplication workflows
  **CN:** 矩阵乘法工作流
- **EN:** plugin or compiler extension points
  **CN:** 插件或编译器扩展点

## Dependencies / 依赖关系

- **EN:** External or absolute imports include `pytest`, `torch`, `triton`, `triton.language`, `triton._internal_testing`, `triton.tools.tensor_descriptor`.
  **CN:** 外部或绝对导入包括 `pytest`、`torch`、`triton`、`triton.language`、`triton._internal_testing`、`triton.tools.tensor_descriptor`。
- **EN:** Execution centers on top-level definitions such as `check_capabilities`, `matmul_kernel`, `matmul_kernel_tma`, `vecadd_kernel`, `mxfp_to_bf16_kernel`, `dot_scale_ref`, `test_pipeline_matmul`, `test_pipeline_vecadd`, `test_pipeline_epilogue`, `random_bfloat16`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `check_capabilities`、`matmul_kernel`、`matmul_kernel_tma`、`vecadd_kernel`、`mxfp_to_bf16_kernel`、`dot_scale_ref`、`test_pipeline_matmul`、`test_pipeline_vecadd`、`test_pipeline_epilogue`、`random_bfloat16`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
