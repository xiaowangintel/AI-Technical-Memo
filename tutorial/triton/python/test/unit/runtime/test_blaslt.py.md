# test_blaslt.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/unit/runtime/test_blaslt.py`
- **EN:** Pytest module covering blaslt behavior in Triton's Python tests. It contains 4 top-level definition(s) and 5 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 blaslt 行为。 该文件包含 4 个顶层定义，以及 5 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4

```python
import pytest
import torch
from triton._internal_testing import is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4
from triton.tools.mxfp import MXFP4Tensor, MXScaleTensor
```
- **EN:** Imports the modules used in this scope: `pytest`, `torch`, `triton._internal_testing`, `triton.tools.mxfp`.
- **CN:** 导入此作用域使用的模块：`pytest`、`torch`、`triton._internal_testing`、`triton.tools.mxfp`。

### Lines 5-7

```python


def supports_block_scaling():
```
- **EN:** Defines the helper function `supports_block_scaling`. Key calls include `is_cuda`, `torch.cuda.get_device_capability`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `supports_block_scaling`。 关键调用包括 `is_cuda`、`torch.cuda.get_device_capability`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 8-8

```python
    return is_cuda() and torch.cuda.get_device_capability()[0] >= 10
```
- **EN:** Invokes `is_cuda`, `torch.cuda.get_device_capability` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `is_cuda`、`torch.cuda.get_device_capability` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 9-13

```python


@pytest.mark.parametrize("m, n, k", [(16, 16, 16), (32, 16, 16), (16, 32, 16), (16, 16, 32)])
@pytest.mark.parametrize("dtype_str", ["float8_e4m3fn", "float8_e4m3fnuz", "float16"])
def test_blaslt(m, n, k, dtype_str, device):
```
- **EN:** Defines the test function `test_blaslt`. Decorators: `pytest.mark.parametrize('m, n, k', [(16, 16, 16), (32, 16, 16), (16, 32, 16), (16, 16, 32)])`, `pytest.mark.parametrize('dtype_str', ['float8_e4m3fn', 'float8_e4m3fnuz', 'float16'])`. Parameters: `m`, `n`, `k`, `dtype_str`, `device`. Nested definitions in this scope: `limited_rand`. Key calls include `pytest.mark.parametrize`, `getattr`, `is_cuda`, `torch.manual_seed`, `torch.tensor`, `torch.zeros`, and 19 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 定义测试函数 `test_blaslt`。 装饰器：`pytest.mark.parametrize('m, n, k', [(16, 16, 16), (32, 16, 16), (16, 32, 16), (16, 16, 32)])`、`pytest.mark.parametrize('dtype_str', ['float8_e4m3fn', 'float8_e4m3fnuz', 'float16'])`。 参数：`m`、`n`、`k`、`dtype_str`、`device`。 该作用域中的嵌套定义：`limited_rand`。 关键调用包括 `pytest.mark.parametrize`、`getattr`、`is_cuda`、`torch.manual_seed`、`torch.tensor`、`torch.zeros` 等另外 19 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

#### Lines 14-14

```python
    dtype = getattr(torch, dtype_str)
```
- **EN:** Prepares or updates state through `dtype`. Invokes `getattr` to execute the test logic.
- **CN:** 通过 `dtype` 准备或更新状态。 调用 `getattr` 执行测试逻辑。

#### Lines 15-33

```python

    if is_cuda():
        from triton._C.libtriton import nvidia as vendor
        if dtype_str == "float8_e4m3fnuz":
            pytest.skip("float8_e4m3fnuz is not supported on CUDA")
        if dtype == torch.float8_e4m3fn and torch.cuda.get_device_capability()[0] < 9:
            pytest.skip("fp8 is only supported on CUDA with cc >= 90")
        c_dtype = dtype
        make_handle = lambda workspace: vendor.cublas.CublasLt(workspace)
    elif is_hip():
        from triton._C.libtriton import amd as vendor
        if dtype_str == "float8_e4m3fnuz" and not is_hip_cdna3():
            pytest.skip("float8_e4m3fnuz is only supported on HIP CDNA3")
        if dtype_str == "float8_e4m3fn" and not is_hip_cdna4():
            pytest.skip("float8_e4m3fn is only supported on HIP CDNA4")
        c_dtype = torch.float16 if dtype_str in ("float8_e4m3fnuz", "float8_e4m3fn") else dtype
        make_handle = lambda workspace: vendor.hipblas.HipblasLt(workspace)
    else:
        pytest.skip("test_blaslt is only supported on CUDA or HIP")
```
- **EN:** Invokes `is_cuda`, `is_hip`, `pytest.skip`, `vendor.cublas.CublasLt`, `vendor.hipblas.HipblasLt`, `torch.cuda.get_device_capability`, and 2 more to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, BLASLt integration.
- **CN:** 调用 `is_cuda`、`is_hip`、`pytest.skip`、`vendor.cublas.CublasLt`、`vendor.hipblas.HipblasLt`、`torch.cuda.get_device_capability` 等另外 2 项 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、BLASLt 集成。

#### Lines 34-36

```python

    torch.manual_seed(123)
    workspace_size = 32 * 1024 * 1024
```
- **EN:** Prepares or updates state through `workspace_size`. Invokes `torch.manual_seed` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `workspace_size` 准备或更新状态。 调用 `torch.manual_seed` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 37-38

```python

    def limited_rand(elements, shape):
```
- **EN:** Defines the helper function `limited_rand`. Parameters: `elements`, `shape`. Key calls include `torch.randint`, `torch.prod`, `torch.tensor`. This scope touches PyTorch tensor setup and checks, random-data generation.
- **CN:** 定义辅助函数 `limited_rand`。 参数：`elements`、`shape`。 关键调用包括 `torch.randint`、`torch.prod`、`torch.tensor`。 该作用域涉及PyTorch 张量准备与校验、随机数据生成。

##### Lines 39-41

```python
        total_elems = torch.prod(torch.tensor(shape)).item()
        indices = torch.randint(0, len(elements), (total_elems, ), device=device)
        return elements[indices].view(shape)
```
- **EN:** Prepares or updates state through `total_elems`, `indices`. Invokes `torch.prod`, `torch.tensor`, `torch.randint` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `total_elems`、`indices` 准备或更新状态。 调用 `torch.prod`、`torch.tensor`、`torch.randint` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 42-56

```python

    elements = torch.tensor([-2.0, -1.0, 0.0, 1.0, 2.0], dtype=torch.float32, device=device)
    a = limited_rand(elements, (m, k)).to(dtype)
    b = limited_rand(elements, (k, n)).to(dtype)

    c = torch.zeros((m, n), dtype=c_dtype, device=device)

    b = b.T.contiguous()

    workspace = torch.empty(workspace_size, dtype=torch.int8, device=device)
    handle = make_handle(workspace)

    handle.matmul(a, b, c)

    ref = torch.matmul(a.to(torch.float16), b.to(torch.float16).T)
```
- **EN:** Prepares or updates state through `elements`, `a`, `b`, `c`, `workspace`, `handle`, `ref`. Invokes `torch.tensor`, `limited_rand`, `torch.zeros`, `b.T.contiguous`, `torch.empty`, `make_handle`, and 4 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 通过 `elements`、`a`、`b`、`c`、`workspace`、`handle`、`ref` 准备或更新状态。 调用 `torch.tensor`、`limited_rand`、`torch.zeros`、`b.T.contiguous`、`torch.empty`、`make_handle` 等另外 4 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

#### Lines 57-58

```python

    assert torch.allclose(c.to(torch.float16), ref, atol=2.0)
```
- **EN:** Invokes `torch.allclose`, `c.to` to execute the test logic. Validates behavior with 1 assertion(s). Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.allclose`、`c.to` 执行测试逻辑。 通过 1 个断言验证行为。 相关主题：PyTorch 张量准备与校验。

### Lines 59-71

```python


@pytest.mark.parametrize(
    "m, n, k",
    [
        (256, 256, 512),
        (256, 512, 512),
        (512, 256, 512),
        (512, 512, 512),
        (1024, 1024, 1024),
    ],
)
def test_block_scaled_matmul_mxfp8(m, n, k, device):
```
- **EN:** Defines the test function `test_block_scaled_matmul_mxfp8`. Decorators: `pytest.mark.parametrize('m, n, k', [(256, 256, 512), (256, 512, 512), (512, 256, 512), (512, 512, 512), (1024, 1024, 1024)])`. Parameters: `m`, `n`, `k`, `device`. Nested definitions in this scope: `unpack_scale`. Key calls include `pytest.mark.parametrize`, `torch.manual_seed`, `torch.empty`, `nvidia.cublas.CublasLt`, `torch.randn`, `a_fp32.to`, and 21 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 定义测试函数 `test_block_scaled_matmul_mxfp8`。 装饰器：`pytest.mark.parametrize('m, n, k', [(256, 256, 512), (256, 512, 512), (512, 256, 512), (512, 512, 512), (1024, 1024, 1024)])`。 参数：`m`、`n`、`k`、`device`。 该作用域中的嵌套定义：`unpack_scale`。 关键调用包括 `pytest.mark.parametrize`、`torch.manual_seed`、`torch.empty`、`nvidia.cublas.CublasLt`、`torch.randn`、`a_fp32.to` 等另外 21 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

#### Lines 72-72

```python
    """Test block-scaled matmul with MXFP8 format (FP8 E4M3 inputs, E8M0 scales)."""
```
- **EN:** Relevant themes: matrix multiplication workflows.
- **CN:** 相关主题：矩阵乘法工作流。

#### Lines 73-74

```python
    if not is_cuda():
        pytest.skip("block_scaled_matmul is only supported on CUDA")
```
- **EN:** Invokes `is_cuda`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: matrix multiplication workflows.
- **CN:** 调用 `is_cuda`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：矩阵乘法工作流。

#### Lines 75-76

```python
    if not supports_block_scaling():
        pytest.skip("block_scaled_matmul requires compute capability 10.0 (Blackwell)")
```
- **EN:** Invokes `supports_block_scaling`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: matrix multiplication workflows.
- **CN:** 调用 `supports_block_scaling`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：矩阵乘法工作流。

#### Lines 77-78

```python

    from triton._C.libtriton import nvidia
```
- **EN:** Imports the modules used in this scope: `triton._C.libtriton`.
- **CN:** 导入此作用域使用的模块：`triton._C.libtriton`。

#### Lines 79-123

```python

    torch.manual_seed(42)

    # Constants for MXFP8
    VEC_SIZE = 32  # 32-element groups for E8M0 scales

    # Create workspace and cuBLAS handle
    workspace_size = 32 * 1024 * 1024
    workspace = torch.empty(workspace_size, dtype=torch.uint8, device=device)
    handle = nvidia.cublas.CublasLt(workspace)

    # Generate random FP8 inputs
    a_fp32 = torch.randn(m, k, device=device, dtype=torch.float32)
    b_fp32 = torch.randn(n, k, device=device, dtype=torch.float32)

    # Convert to FP8 E4M3
    a = a_fp32.to(torch.float8_e4m3fn)
    b = b_fp32.to(torch.float8_e4m3fn)

    # Generate scales in the expected 4D layout, then reshape to 5D and flatten
    # Scale shape: [M // 128, K // VEC_SIZE // 4, 32, 16]
    a_scale_shape = [m // 128, k // VEC_SIZE // 4, 32, 16]
    b_scale_shape = [n // 128, k // VEC_SIZE // 4, 32, 16]

    epsilon = 1e-8
    a_scale_raw = torch.rand(a_scale_shape, device=device) + epsilon
    b_scale_raw = torch.rand(b_scale_shape, device=device) + epsilon

    # Convert to MXScaleTensor (E8M0 format)
    a_scale_mx = MXScaleTensor(a_scale_raw)
    b_scale_mx = MXScaleTensor(b_scale_raw)
    a_scale = a_scale_mx.data
    b_scale = b_scale_mx.data

    # Reshape to 5D for TMA and flatten for cuBLAS
    a_scale_5d = a_scale.reshape(1, a_scale_shape[0], a_scale.shape[1], 2, 256)
    b_scale_5d = b_scale.reshape(1, b_scale_shape[0], b_scale.shape[1], 2, 256)
    a_scale_cublas = a_scale_5d.contiguous().flatten()
    b_scale_cublas = b_scale_5d.contiguous().flatten()

    # Prepare output tensor
    output = torch.empty((m, n), dtype=torch.float16, device=device)

    # Call cuBLAS block-scaled matmul
    handle.block_scaled_matmul_mxfp8(a, b, output, a_scale_cublas, b_scale_cublas)
```
- **EN:** Prepares or updates state through `VEC_SIZE`, `workspace_size`, `workspace`, `handle`, `a_fp32`, `b_fp32`, `a`, `b`, and 14 more. Invokes `torch.manual_seed`, `torch.empty`, `nvidia.cublas.CublasLt`, `torch.randn`, `a_fp32.to`, `b_fp32.to`, and 7 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, layout transformation reasoning, random-data generation.
- **CN:** 通过 `VEC_SIZE`、`workspace_size`、`workspace`、`handle`、`a_fp32`、`b_fp32`、`a`、`b` 等另外 14 项 准备或更新状态。 调用 `torch.manual_seed`、`torch.empty`、`nvidia.cublas.CublasLt`、`torch.randn`、`a_fp32.to`、`b_fp32.to` 等另外 7 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、布局变换推理、随机数据生成。

#### Lines 124-126

```python

    # Compute reference using PyTorch
    def unpack_scale(packed):
```
- **EN:** Defines the helper function `unpack_scale`. Parameters: `packed`. Key calls include `packed.reshape`, `packed.permute`.
- **CN:** 定义辅助函数 `unpack_scale`。 参数：`packed`。 关键调用包括 `packed.reshape`、`packed.permute`。

##### Lines 127-129

```python
        packed = packed.reshape(*packed.shape[:-2], 32, 4, 4)
        num_chunk_m, num_chunk_k, _, _, _ = packed.shape
        return packed.permute(0, 3, 2, 1, 4).reshape(num_chunk_m * 128, num_chunk_k * 4).contiguous()
```
- **EN:** Prepares or updates state through `packed`, `num_chunk_m`, `num_chunk_k`, `_`. Invokes `packed.reshape`, `packed.permute` to execute the test logic.
- **CN:** 通过 `packed`、`num_chunk_m`、`num_chunk_k`、`_` 准备或更新状态。 调用 `packed.reshape`、`packed.permute` 执行测试逻辑。

#### Lines 130-138

```python

    a_scale_ref = a_scale_mx.to(torch.float32)
    b_scale_ref = b_scale_mx.to(torch.float32)
    a_scale_ref = unpack_scale(a_scale_ref).repeat_interleave(VEC_SIZE, dim=1)[:m, :k]
    b_scale_ref = unpack_scale(b_scale_ref).repeat_interleave(VEC_SIZE, dim=1).T.contiguous()[:k, :n]

    ref = torch.matmul(a.to(torch.float32) * a_scale_ref, b.to(torch.float32).T * b_scale_ref)

    torch.testing.assert_close(output.to(torch.float32), ref, atol=1e-1, rtol=1e-1)
```
- **EN:** Prepares or updates state through `a_scale_ref`, `b_scale_ref`, `ref`. Invokes `a_scale_mx.to`, `b_scale_mx.to`, `unpack_scale`, `torch.matmul`, `a.to`, `b.to`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 通过 `a_scale_ref`、`b_scale_ref`、`ref` 准备或更新状态。 调用 `a_scale_mx.to`、`b_scale_mx.to`、`unpack_scale`、`torch.matmul`、`a.to`、`b.to` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

### Lines 139-142

```python


@pytest.mark.parametrize("m, n, k", [(256, 256, 512), (512, 512, 512), (1024, 1024, 1024)])
def test_block_scaled_matmul_nvfp4(m, n, k, device):
```
- **EN:** Defines the test function `test_block_scaled_matmul_nvfp4`. Decorators: `pytest.mark.parametrize('m, n, k', [(256, 256, 512), (512, 512, 512), (1024, 1024, 1024)])`. Parameters: `m`, `n`, `k`, `device`. Nested definitions in this scope: `unpack_scale`. Key calls include `pytest.mark.parametrize`, `torch.manual_seed`, `torch.empty`, `nvidia.cublas.CublasLt`, `a_ref.to_packed_tensor`, `b_ref.to_packed_tensor`, and 20 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 定义测试函数 `test_block_scaled_matmul_nvfp4`。 装饰器：`pytest.mark.parametrize('m, n, k', [(256, 256, 512), (512, 512, 512), (1024, 1024, 1024)])`。 参数：`m`、`n`、`k`、`device`。 该作用域中的嵌套定义：`unpack_scale`。 关键调用包括 `pytest.mark.parametrize`、`torch.manual_seed`、`torch.empty`、`nvidia.cublas.CublasLt`、`a_ref.to_packed_tensor`、`b_ref.to_packed_tensor` 等另外 20 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

#### Lines 143-143

```python
    """Test block-scaled matmul with NVFP4 format (packed FP4 inputs, FP8 E4M3 scales)."""
```
- **EN:** Relevant themes: matrix multiplication workflows.
- **CN:** 相关主题：矩阵乘法工作流。

#### Lines 144-145

```python
    if not is_cuda():
        pytest.skip("block_scaled_matmul is only supported on CUDA")
```
- **EN:** Invokes `is_cuda`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: matrix multiplication workflows.
- **CN:** 调用 `is_cuda`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：矩阵乘法工作流。

#### Lines 146-147

```python
    if not supports_block_scaling():
        pytest.skip("block_scaled_matmul requires compute capability 10.0 (Blackwell)")
```
- **EN:** Invokes `supports_block_scaling`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: matrix multiplication workflows.
- **CN:** 调用 `supports_block_scaling`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：矩阵乘法工作流。

#### Lines 148-149

```python

    from triton._C.libtriton import nvidia
```
- **EN:** Imports the modules used in this scope: `triton._C.libtriton`.
- **CN:** 导入此作用域使用的模块：`triton._C.libtriton`。

#### Lines 150-190

```python

    torch.manual_seed(42)

    # Constants for NVFP4
    VEC_SIZE = 16  # 16-element groups for FP8 E4M3 scales

    # Create workspace and cuBLAS handle
    workspace_size = 32 * 1024 * 1024
    workspace = torch.empty(workspace_size, dtype=torch.uint8, device=device)
    handle = nvidia.cublas.CublasLt(workspace)

    # Generate random MXFP4 tensors
    a_ref = MXFP4Tensor(size=(m, k), device=device).random()
    b_ref = MXFP4Tensor(size=(n, k), device=device).random()

    # Pack two FP4 elements per byte along K dimension
    a = a_ref.to_packed_tensor(dim=1)  # (M, K//2) in uint8
    b = b_ref.to_packed_tensor(dim=1)  # (N, K//2) in uint8

    # Generate scales in the expected 4D layout
    # Scale shape: [M // 128, K // VEC_SIZE // 4, 32, 16]
    a_scale_shape = [m // 128, k // VEC_SIZE // 4, 32, 16]
    b_scale_shape = [n // 128, k // VEC_SIZE // 4, 32, 16]

    epsilon = 1e-8
    a_scale_raw = torch.rand(a_scale_shape, device=device) + epsilon
    b_scale_raw = torch.rand(b_scale_shape, device=device) + epsilon

    # For NVFP4, scales are FP8 E4M3
    a_scale = a_scale_raw.to(torch.float8_e4m3fn)
    b_scale = b_scale_raw.to(torch.float8_e4m3fn)

    # Flatten for cuBLAS (use original 4D layout, not 5D reshaped)
    a_scale_cublas = a_scale.contiguous().flatten()
    b_scale_cublas = b_scale.contiguous().flatten()

    # Prepare output tensor
    output = torch.empty((m, n), dtype=torch.float16, device=device)

    # Call cuBLAS block-scaled matmul
    handle.block_scaled_matmul_nvfp4(a, b, output, a_scale_cublas, b_scale_cublas)
```
- **EN:** Prepares or updates state through `VEC_SIZE`, `workspace_size`, `workspace`, `handle`, `a_ref`, `b_ref`, `a`, `b`, and 10 more. Invokes `torch.manual_seed`, `torch.empty`, `nvidia.cublas.CublasLt`, `MXFP4Tensor`, `a_ref.to_packed_tensor`, `b_ref.to_packed_tensor`, and 6 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, layout transformation reasoning, random-data generation.
- **CN:** 通过 `VEC_SIZE`、`workspace_size`、`workspace`、`handle`、`a_ref`、`b_ref`、`a`、`b` 等另外 10 项 准备或更新状态。 调用 `torch.manual_seed`、`torch.empty`、`nvidia.cublas.CublasLt`、`MXFP4Tensor`、`a_ref.to_packed_tensor`、`b_ref.to_packed_tensor` 等另外 6 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、布局变换推理、随机数据生成。

#### Lines 191-193

```python

    # Compute reference using PyTorch
    def unpack_scale(packed):
```
- **EN:** Defines the helper function `unpack_scale`. Parameters: `packed`. Key calls include `packed.reshape`, `packed.permute`.
- **CN:** 定义辅助函数 `unpack_scale`。 参数：`packed`。 关键调用包括 `packed.reshape`、`packed.permute`。

##### Lines 194-196

```python
        packed = packed.reshape(*packed.shape[:-2], 32, 4, 4)
        num_chunk_m, num_chunk_k, _, _, _ = packed.shape
        return packed.permute(0, 3, 2, 1, 4).reshape(num_chunk_m * 128, num_chunk_k * 4).contiguous()
```
- **EN:** Prepares or updates state through `packed`, `num_chunk_m`, `num_chunk_k`, `_`. Invokes `packed.reshape`, `packed.permute` to execute the test logic.
- **CN:** 通过 `packed`、`num_chunk_m`、`num_chunk_k`、`_` 准备或更新状态。 调用 `packed.reshape`、`packed.permute` 执行测试逻辑。

#### Lines 197-205

```python

    a_scale_ref = a_scale.to(torch.float32)
    b_scale_ref = b_scale.to(torch.float32)
    a_scale_ref = unpack_scale(a_scale_ref).repeat_interleave(VEC_SIZE, dim=1)[:m, :k]
    b_scale_ref = unpack_scale(b_scale_ref).repeat_interleave(VEC_SIZE, dim=1).T.contiguous()[:k, :n]

    ref = torch.matmul(a_ref.to(torch.float32) * a_scale_ref, b_ref.to(torch.float32).T * b_scale_ref)

    torch.testing.assert_close(output.to(torch.float32), ref, atol=1e-1, rtol=1e-1)
```
- **EN:** Prepares or updates state through `a_scale_ref`, `b_scale_ref`, `ref`. Invokes `a_scale.to`, `b_scale.to`, `unpack_scale`, `torch.matmul`, `a_ref.to`, `b_ref.to`, and 2 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 通过 `a_scale_ref`、`b_scale_ref`、`ref` 准备或更新状态。 调用 `a_scale.to`、`b_scale.to`、`unpack_scale`、`torch.matmul`、`a_ref.to`、`b_ref.to` 等另外 2 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `supports_block_scaling`, `test_blaslt`, `test_block_scaled_matmul_mxfp8`, `test_block_scaled_matmul_nvfp4`
  **CN:** 顶层作用域，例如 `supports_block_scaling`、`test_blaslt`、`test_block_scaled_matmul_mxfp8`、`test_block_scaled_matmul_nvfp4`
- **EN:** pytest parametrization
  **CN:** pytest 参数化
- **EN:** PyTorch tensor setup and checks
  **CN:** PyTorch 张量准备与校验
- **EN:** matrix multiplication workflows
  **CN:** 矩阵乘法工作流
- **EN:** layout transformation reasoning
  **CN:** 布局变换推理
- **EN:** random-data generation
  **CN:** 随机数据生成
- **EN:** BLASLt integration
  **CN:** BLASLt 集成

## Dependencies / 依赖关系

- **EN:** External or absolute imports include `pytest`, `torch`, `triton._internal_testing`, `triton.tools.mxfp`, `triton._C.libtriton`.
  **CN:** 外部或绝对导入包括 `pytest`、`torch`、`triton._internal_testing`、`triton.tools.mxfp`、`triton._C.libtriton`。
- **EN:** Execution centers on top-level definitions such as `supports_block_scaling`, `test_blaslt`, `test_block_scaled_matmul_mxfp8`, `test_block_scaled_matmul_nvfp4`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `supports_block_scaling`、`test_blaslt`、`test_block_scaled_matmul_mxfp8`、`test_block_scaled_matmul_nvfp4`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
