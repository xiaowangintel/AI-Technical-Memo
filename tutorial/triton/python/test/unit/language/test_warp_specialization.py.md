# test_warp_specialization.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/unit/language/test_warp_specialization.py`
- **EN:** Pytest module covering warp specialization behavior in Triton's Python tests. It contains 20 top-level definition(s) and 8 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 warp specialization 行为。 该文件包含 20 个顶层定义，以及 8 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```python
import torch
import pytest
import pathlib
import triton
import triton.language as tl

from triton._internal_testing import is_hip, is_hopper, is_blackwell
from triton.tools.tensor_descriptor import TensorDescriptor
```
- **EN:** Imports the modules used in this scope: `torch`, `pytest`, `pathlib`, `triton`, `triton.language`, `triton._internal_testing`, `triton.tools.tensor_descriptor`. Relevant themes: Triton language operations, tensor/descriptor metadata.
- **CN:** 导入此作用域使用的模块：`torch`、`pytest`、`pathlib`、`triton`、`triton.language`、`triton._internal_testing`、`triton.tools.tensor_descriptor`。 相关主题：Triton language 操作、张量/描述符元数据。

### Lines 9-15

```python

if not is_hip() and torch.cuda.is_available() and torch.cuda.get_device_capability()[0] in [9, 10, 11]:
    from triton._C.libtriton import nvidia
    cublas_workspace = torch.empty(32 * 1024 * 1024, device="cuda", dtype=torch.uint8)
    cublas = nvidia.cublas.CublasLt(cublas_workspace)
else:
    cublas = None
```
- **EN:** Invokes `torch.cuda.is_available`, `torch.empty`, `nvidia.cublas.CublasLt`, `is_hip`, `torch.cuda.get_device_capability` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks, BLASLt integration.
- **CN:** 调用 `torch.cuda.is_available`、`torch.empty`、`nvidia.cublas.CublasLt`、`is_hip`、`torch.cuda.get_device_capability` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验、BLASLt 集成。

### Lines 16-18

```python


def is_hopper_or_blackwell():
```
- **EN:** Defines the helper function `is_hopper_or_blackwell`. Key calls include `is_hopper`, `is_blackwell`.
- **CN:** 定义辅助函数 `is_hopper_or_blackwell`。 关键调用包括 `is_hopper`、`is_blackwell`。

#### Lines 19-19

```python
    return is_hopper() or is_blackwell()
```
- **EN:** Invokes `is_hopper`, `is_blackwell` to execute the test logic.
- **CN:** 调用 `is_hopper`、`is_blackwell` 执行测试逻辑。

### Lines 20-24

```python


@pytest.mark.skipif(is_hip(), reason="warp specialization is not supported on hip devices")
@pytest.mark.skipif(not is_hopper_or_blackwell(), reason="Requires Hopper or Blackwell")
def test_warp_specialize_basic_ir(tmp_path: pathlib.Path):
```
- **EN:** Defines the test function `test_warp_specialize_basic_ir`. Decorators: `pytest.mark.skipif(is_hip(), reason='warp specialization is not supported on hip devices')`, `pytest.mark.skipif(not is_hopper_or_blackwell(), reason='Requires Hopper or Blackwell')`. Parameters: `tmp_path`. Key calls include `pytest.mark.skipif`, `temp_file.write_text`, `triton.compile`, `torch.empty`, `is_hip`, `is_hopper_or_blackwell`. This scope touches Triton compilation or JIT kernels, PyTorch tensor setup and checks.
- **CN:** 定义测试函数 `test_warp_specialize_basic_ir`。 装饰器：`pytest.mark.skipif(is_hip(), reason='warp specialization is not supported on hip devices')`、`pytest.mark.skipif(not is_hopper_or_blackwell(), reason='Requires Hopper or Blackwell')`。 参数：`tmp_path`。 关键调用包括 `pytest.mark.skipif`、`temp_file.write_text`、`triton.compile`、`torch.empty`、`is_hip`、`is_hopper_or_blackwell`。 该作用域涉及Triton 编译或 JIT kernel、PyTorch 张量准备与校验。

#### Lines 25-52

```python
    ir = """
    tt.func @kernel(%arg0: !tt.ptr<i32>) {
      %c42_i32 = arith.constant 42 : i32
      ttg.barrier local
      ttg.warp_specialize(%arg0)
      default {
        tt.store %arg0, %c42_i32 : !tt.ptr<i32>
        ttg.barrier local
        ttg.warp_yield
      }
      partition0(%arg1: !tt.ptr<i32>) num_warps(1) {
        %c5555_i32 = arith.constant 5555 : i32
        %c1_i32 = arith.constant 1 : i32
        ttg.barrier local
        %ptr = tt.addptr %arg1, %c1_i32 : !tt.ptr<i32>, i32
        tt.store %ptr, %c5555_i32 : !tt.ptr<i32>
        ttg.warp_return
      } : (!tt.ptr<i32>) -> ()
      tt.return
    }
    """

    temp_file = tmp_path / "test_warp_specialize_basic_ir.ttir"
    temp_file.write_text(ir)
    kernel = triton.compile(str(temp_file))

    input = torch.empty(2, dtype=torch.int32, device='cuda')
    kernel[(1, 1, 1)](input)
```
- **EN:** Prepares or updates state through `ir`, `temp_file`, `kernel`, `input`. Invokes `temp_file.write_text`, `triton.compile`, `torch.empty` to execute the test logic. Relevant themes: Triton compilation or JIT kernels, PyTorch tensor setup and checks.
- **CN:** 通过 `ir`、`temp_file`、`kernel`、`input` 准备或更新状态。 调用 `temp_file.write_text`、`triton.compile`、`torch.empty` 执行测试逻辑。 相关主题：Triton 编译或 JIT kernel、PyTorch 张量准备与校验。

#### Lines 53-54

```python
    assert input[0] == 42
    assert input[1] == 5555
```
- **EN:** Validates behavior with 2 assertion(s).
- **CN:** 通过 2 个断言验证行为。

### Lines 55-59

```python


@pytest.mark.skipif(is_hip(), reason="warp specialization is not supported on hip devices")
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_warp_specialize_tmem_ir(tmp_path: pathlib.Path):
```
- **EN:** Defines the test function `test_warp_specialize_tmem_ir`. Decorators: `pytest.mark.skipif(is_hip(), reason='warp specialization is not supported on hip devices')`, `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Parameters: `tmp_path`. Key calls include `pytest.mark.skipif`, `temp_file.write_text`, `triton.compile`, `torch.empty_like`, `torch.testing.assert_close`, `is_hip`, and 2 more. This scope touches Triton compilation or JIT kernels, PyTorch tensor setup and checks.
- **CN:** 定义测试函数 `test_warp_specialize_tmem_ir`。 装饰器：`pytest.mark.skipif(is_hip(), reason='warp specialization is not supported on hip devices')`、`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 参数：`tmp_path`。 关键调用包括 `pytest.mark.skipif`、`temp_file.write_text`、`triton.compile`、`torch.empty_like`、`torch.testing.assert_close`、`is_hip` 等另外 2 项。 该作用域涉及Triton 编译或 JIT kernel、PyTorch 张量准备与校验。

#### Lines 60-122

```python
    ir = """
    #blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
    #shared = #ttg.swizzled_shared<{vec=1, perPhase=1, maxPhase=1, order=[1, 0]}>
    #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>

    module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {

    tt.func @test_tmem_ws(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
      %cst = arith.constant dense<64> : tensor<128x64xi32, #blocked>
      %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
      %1 = tt.expand_dims %0 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
      %2 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
      %3 = tt.expand_dims %2 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
      %4 = tt.broadcast %1 {axis = 1 : i32} : tensor<128x1xi32, #blocked> -> tensor<128x64xi32, #blocked>
      %5 = tt.broadcast %3 {axis = 0 : i32} : tensor<1x64xi32, #blocked> -> tensor<128x64xi32, #blocked>
      %6 = arith.muli %4, %cst : tensor<128x64xi32, #blocked>
      %7 = arith.addi %6, %5 : tensor<128x64xi32, #blocked>
      %8 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<128x64x!tt.ptr<f32>, #blocked>
      %9 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<128x64x!tt.ptr<f32>, #blocked>

      %ptrs_in = tt.addptr %8, %7 : tensor<128x64x!tt.ptr<f32>, #blocked>, tensor<128x64xi32, #blocked>
      %ptrs_out = tt.addptr %9, %7 : tensor<128x64x!tt.ptr<f32>, #blocked>, tensor<128x64xi32, #blocked>

      %v_init = tt.load %ptrs_in : tensor<128x64x!tt.ptr<f32>, #blocked>

      %v_shared = ttg.local_alloc %v_init : (tensor<128x64xf32, #blocked>) -> !ttg.memdesc<128x64xf32, #shared, #ttg.shared_memory>
      %v = ttg.local_load %v_shared : !ttg.memdesc<128x64xf32, #shared, #ttg.shared_memory> -> tensor<128x64xf32, #blocked>

      %tmem_in = ttng.tmem_alloc %v : (tensor<128x64xf32, #blocked>) -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>
      %tmem_out = ttng.tmem_alloc : () -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>

      ttg.warp_specialize(%tmem_in, %tmem_out)
      default {
        ttg.warp_yield
      }
      partition0(%in: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>, %out: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>) num_warps(1) {
        ttg.warp_return
      }
      partition1(%in: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>, %out: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>) num_warps(2) {
        ttg.warp_return
      }
      partition2(%in: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>, %out: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>) num_warps(4) {
        %x = ttng.tmem_load %in : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory> -> tensor<128x64xf32, #blocked>
        %true = arith.constant true
        ttng.tmem_store %x, %out, %true : tensor<128x64xf32, #blocked> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
        ttg.warp_return
      } : (!ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>, !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>) -> ()

      %result = ttng.tmem_load %tmem_out : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x64xf32, #blocked>
      tt.store %ptrs_out, %result : tensor<128x64x!tt.ptr<f32>, #blocked>
      tt.return
    }

    }
    """

    temp_file = tmp_path / "test_warp_specialize_tmem_ir.ttgir"
    temp_file.write_text(ir)
    kernel = triton.compile(str(temp_file))
    input = torch.arange(128 * 64, dtype=torch.float32, device='cuda').reshape(128, 64)
    output = torch.empty_like(input)
    kernel[(1, 1, 1)](input, output)
    torch.testing.assert_close(input, output, atol=0, rtol=0)
```
- **EN:** Prepares or updates state through `ir`, `temp_file`, `kernel`, `input`, `output`. Invokes `temp_file.write_text`, `triton.compile`, `torch.arange`, `torch.empty_like`, `torch.testing.assert_close` to execute the test logic. Relevant themes: Triton compilation or JIT kernels, PyTorch tensor setup and checks.
- **CN:** 通过 `ir`、`temp_file`、`kernel`、`input`、`output` 准备或更新状态。 调用 `temp_file.write_text`、`triton.compile`、`torch.arange`、`torch.empty_like`、`torch.testing.assert_close` 执行测试逻辑。 相关主题：Triton 编译或 JIT kernel、PyTorch 张量准备与校验。

### Lines 123-128

```python


@pytest.mark.skipif(is_hip(), reason="warp specialization is not supported on hip devices")
@pytest.mark.skipif(not is_hopper_or_blackwell(), reason="Requires Hopper or Blackwell")
def test_warpgroup_reduction(tmp_path: pathlib.Path):
```
- **EN:** Defines the test function `test_warpgroup_reduction`. Decorators: `pytest.mark.skipif(is_hip(), reason='warp specialization is not supported on hip devices')`, `pytest.mark.skipif(not is_hopper_or_blackwell(), reason='Requires Hopper or Blackwell')`. Parameters: `tmp_path`. Nested definitions in this scope: `template`. Key calls include `pytest.mark.skipif`, `temp_file.write_text`, `triton.compile`, `torch.arange`, `torch.empty`, `is_hip`, and 2 more. This scope touches Triton compilation or JIT kernels, PyTorch tensor setup and checks.
- **CN:** 定义测试函数 `test_warpgroup_reduction`。 装饰器：`pytest.mark.skipif(is_hip(), reason='warp specialization is not supported on hip devices')`、`pytest.mark.skipif(not is_hopper_or_blackwell(), reason='Requires Hopper or Blackwell')`。 参数：`tmp_path`。 该作用域中的嵌套定义：`template`。 关键调用包括 `pytest.mark.skipif`、`temp_file.write_text`、`triton.compile`、`torch.arange`、`torch.empty`、`is_hip` 等另外 2 项。 该作用域涉及Triton 编译或 JIT kernel、PyTorch 张量准备与校验。

#### Lines 129-129

```python
    def template(i, num_warps, in_ptr, out_ptr):
```
- **EN:** Defines the helper function `template`. Parameters: `i`, `num_warps`, `in_ptr`, `out_ptr`.
- **CN:** 定义辅助函数 `template`。 参数：`i`、`num_warps`、`in_ptr`、`out_ptr`。

##### Lines 130-143

```python
        return f"""
          %range = tt.make_range {{end = {(i+1)*256} : i32, start = {i*256} : i32}} : tensor<256xi32, #blocked{num_warps}>
          %splatted = tt.splat {in_ptr} : !tt.ptr<i32> -> tensor<256x!tt.ptr<i32>, #blocked{num_warps}>
          %ptrs = tt.addptr %splatted, %range : tensor<256x!tt.ptr<i32>, #blocked{num_warps}>, tensor<256xi32, #blocked{num_warps}>
          %input = tt.load %ptrs : tensor<256x!tt.ptr<i32>, #blocked{num_warps}>
          %result = "tt.reduce"(%input) ({{
          ^bb0(%lhs: i32, %rhs: i32):
            %result = arith.addi %lhs, %rhs : i32
            tt.reduce.return %result : i32
          }}) {{axis = 0 : i32}} : (tensor<256xi32, #blocked{num_warps}>) -> i32
          %offset = arith.constant {i} : i32
          %output = tt.addptr {out_ptr}, %offset : !tt.ptr<i32>, i32
          tt.store %output, %result : !tt.ptr<i32>
        """
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

#### Lines 144-182

```python

    ir = """
    #blocked4 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    #blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
    #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>

    module attributes {"ttg.num-warps" = 4 : i32} {

    tt.func @kernel(%arg0: !tt.ptr<i32>, %arg1: !tt.ptr<i32>) {
      ttg.warp_specialize(%arg0, %arg1)
      default {
      """ + template(0, 4, "%arg0", "%arg1") + """
        ttg.warp_yield
      }
      partition0(%arg2: !tt.ptr<i32>, %arg3: !tt.ptr<i32>) num_warps(4) {
      """ + template(1, 4, "%arg2", "%arg3") + """
        ttg.warp_return
      }
      partition1(%arg4: !tt.ptr<i32>, %arg5: !tt.ptr<i32>) num_warps(2) {
      """ + template(2, 2, "%arg4", "%arg5") + """
        ttg.warp_return
      }
      partition2(%arg6: !tt.ptr<i32>, %arg7: !tt.ptr<i32>) num_warps(1) {
      """ + template(3, 1, "%arg6", "%arg7") + """
        ttg.warp_return
      } : (!tt.ptr<i32>, !tt.ptr<i32>) -> ()
      tt.return
    }

    }
    """

    temp_file = tmp_path / "test_warpgroup_reduction.ttgir"
    temp_file.write_text(ir)
    kernel = triton.compile(str(temp_file))

    input = torch.arange(1024, dtype=torch.int32, device='cuda')
    output = torch.empty(4, dtype=torch.int32, device='cuda')
    kernel[(1, 1, 1)](input, output)
```
- **EN:** Prepares or updates state through `ir`, `temp_file`, `kernel`, `input`, `output`. Invokes `template`, `temp_file.write_text`, `triton.compile`, `torch.arange`, `torch.empty` to execute the test logic. Relevant themes: Triton compilation or JIT kernels, PyTorch tensor setup and checks.
- **CN:** 通过 `ir`、`temp_file`、`kernel`、`input`、`output` 准备或更新状态。 调用 `template`、`temp_file.write_text`、`triton.compile`、`torch.arange`、`torch.empty` 执行测试逻辑。 相关主题：Triton 编译或 JIT kernel、PyTorch 张量准备与校验。

#### Lines 183-186

```python
    assert output[0] == torch.arange(0, 256).sum()
    assert output[1] == torch.arange(256, 512).sum()
    assert output[2] == torch.arange(512, 768).sum()
    assert output[3] == torch.arange(768, 1024).sum()
```
- **EN:** Invokes `torch.arange` to execute the test logic. Validates behavior with 4 assertion(s). Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.arange` 执行测试逻辑。 通过 4 个断言验证行为。 相关主题：PyTorch 张量准备与校验。

### Lines 187-190

```python


@triton.jit
def _compute_pid(tile_id, num_pid_n, num_pid_m, GROUP_SIZE_M):
```
- **EN:** Defines the helper function `_compute_pid`. Decorators: `triton.jit`. Parameters: `tile_id`, `num_pid_n`, `num_pid_m`, `GROUP_SIZE_M`. Key calls include `min`. This scope touches Triton compilation or JIT kernels.
- **CN:** 定义辅助函数 `_compute_pid`。 装饰器：`triton.jit`。 参数：`tile_id`、`num_pid_n`、`num_pid_m`、`GROUP_SIZE_M`。 关键调用包括 `min`。 该作用域涉及Triton 编译或 JIT kernel。

#### Lines 191-197

```python
    num_pid_in_group = GROUP_SIZE_M * num_pid_n
    group_id = tile_id // num_pid_in_group
    first_pid_m = group_id * GROUP_SIZE_M
    group_size_m = min(num_pid_m - first_pid_m, GROUP_SIZE_M)
    pid_m = first_pid_m + (tile_id % group_size_m)
    pid_n = (tile_id % num_pid_in_group) // group_size_m
    return pid_m, pid_n
```
- **EN:** Prepares or updates state through `num_pid_in_group`, `group_id`, `first_pid_m`, `group_size_m`, `pid_m`, `pid_n`. Invokes `min` to execute the test logic.
- **CN:** 通过 `num_pid_in_group`、`group_id`、`first_pid_m`、`group_size_m`、`pid_m`、`pid_n` 准备或更新状态。 调用 `min` 执行测试逻辑。

### Lines 198-201

```python


@triton.jit
def _maybe_tma_load(desc, ptr, off0, off1, USE_TMA: tl.constexpr):
```
- **EN:** Defines the helper function `_maybe_tma_load`. Decorators: `triton.jit`. Parameters: `desc`, `ptr`, `off0`, `off1`, `USE_TMA`. Key calls include `desc.load`, `tl.load`, `tl.arange`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `_maybe_tma_load`。 装饰器：`triton.jit`。 参数：`desc`、`ptr`、`off0`、`off1`、`USE_TMA`。 关键调用包括 `desc.load`、`tl.load`、`tl.arange`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 202-210

```python
    if USE_TMA:
        return desc.load((off0, off1))
    else:
        offs0 = off0 + tl.arange(0, desc.block_shape[0])
        offs1 = off1 + tl.arange(0, desc.block_shape[1])
        mask0 = offs0 < desc.shape[0]
        mask1 = offs1 < desc.shape[1]
        mask = mask0[:, None] & mask1[None, :]
        return tl.load(ptr + offs0[:, None] * desc.strides[0] + offs1[None, :] * desc.strides[1], mask=mask)
```
- **EN:** Invokes `desc.load`, `tl.load`, `tl.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `desc.load`、`tl.load`、`tl.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

### Lines 211-228

```python


@triton.jit
def matmul_tma_ws_kernel(  #
        a_ptr, b_ptr, c_ptr,  #
        a_stride0, a_stride1,  #
        b_stride0, b_stride1,  #
        c_stride0, c_stride1,  #
        M, N, K,  #
        num_stages: tl.constexpr,  #
        BLOCK_SIZE_M: tl.constexpr,  #
        BLOCK_SIZE_N: tl.constexpr,  #
        BLOCK_SIZE_K: tl.constexpr,  #
        GROUP_SIZE_M: tl.constexpr,  #
        USE_FP8: tl.constexpr,  #
        A_USE_TMA: tl.constexpr,  #
        B_USE_TMA: tl.constexpr,  #
):
```
- **EN:** Defines the helper function `matmul_tma_ws_kernel`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `a_stride0`, `a_stride1`, `b_stride0`, `b_stride1`, `c_stride0`, and 12 more. Key calls include `tl.make_tensor_descriptor`, `tl.program_id`, `tl.cdiv`, `_compute_pid`, `tl.zeros`, `accumulator.to`, and 3 more. This scope touches Triton compilation or JIT kernels, Triton language operations, tensor/descriptor metadata, matrix multiplication workflows.
- **CN:** 定义辅助函数 `matmul_tma_ws_kernel`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`a_stride0`、`a_stride1`、`b_stride0`、`b_stride1`、`c_stride0` 等另外 12 项。 关键调用包括 `tl.make_tensor_descriptor`、`tl.program_id`、`tl.cdiv`、`_compute_pid`、`tl.zeros`、`accumulator.to` 等另外 3 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、张量/描述符元数据、矩阵乘法工作流。

#### Lines 229-245

```python
    a_desc = tl.make_tensor_descriptor(a_ptr, shape=[M, K], strides=[a_stride0, a_stride1],
                                       block_shape=[BLOCK_SIZE_M, BLOCK_SIZE_K])
    b_desc = tl.make_tensor_descriptor(b_ptr, shape=[N, K], strides=[b_stride0, b_stride1],
                                       block_shape=[BLOCK_SIZE_N, BLOCK_SIZE_K])
    c_desc = tl.make_tensor_descriptor(c_ptr, shape=[M, N], strides=[c_stride0, c_stride1],
                                       block_shape=[BLOCK_SIZE_M, BLOCK_SIZE_N])

    pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)
    pid_m, pid_n = _compute_pid(pid, num_pid_n, num_pid_m, GROUP_SIZE_M)

    k_tiles = tl.cdiv(K, BLOCK_SIZE_K)

    off_am = pid_m * BLOCK_SIZE_M
    off_bn = pid_n * BLOCK_SIZE_N
    accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
```
- **EN:** Prepares or updates state through `a_desc`, `b_desc`, `c_desc`, `pid`, `num_pid_m`, `num_pid_n`, `pid_m`, `pid_n`, and 4 more. Invokes `tl.make_tensor_descriptor`, `tl.program_id`, `tl.cdiv`, `_compute_pid`, `tl.zeros` to execute the test logic. Relevant themes: Triton language operations, tensor/descriptor metadata.
- **CN:** 通过 `a_desc`、`b_desc`、`c_desc`、`pid`、`num_pid_m`、`num_pid_n`、`pid_m`、`pid_n` 等另外 4 项 准备或更新状态。 调用 `tl.make_tensor_descriptor`、`tl.program_id`、`tl.cdiv`、`_compute_pid`、`tl.zeros` 执行测试逻辑。 相关主题：Triton language 操作、张量/描述符元数据。

#### Lines 246-250

```python
    for k in tl.range(k_tiles, warp_specialize=True, num_stages=num_stages):
        off_k = k * BLOCK_SIZE_K
        a = _maybe_tma_load(a_desc, a_ptr, off_am, off_k, A_USE_TMA)
        b = _maybe_tma_load(b_desc, b_ptr, off_bn, off_k, B_USE_TMA)
        accumulator = tl.dot(a, b.T, accumulator)
```
- **EN:** Invokes `_maybe_tma_load`, `tl.dot` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `_maybe_tma_load`、`tl.dot` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

#### Lines 251-253

```python

    c = accumulator.to(tl.float8e4nv if USE_FP8 else tl.float16)
    c_desc.store((off_am, off_bn), c)
```
- **EN:** Prepares or updates state through `c`. Invokes `accumulator.to`, `c_desc.store` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `c` 准备或更新状态。 调用 `accumulator.to`、`c_desc.store` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 254-256

```python


def exceeds_smem_capacity(num_stages, BLOCK_M, BLOCK_N, BLOCK_K, use_fp8):
```
- **EN:** Defines the helper function `exceeds_smem_capacity`. Parameters: `num_stages`, `BLOCK_M`, `BLOCK_N`, `BLOCK_K`, `use_fp8`. This scope touches plugin or compiler extension points.
- **CN:** 定义辅助函数 `exceeds_smem_capacity`。 参数：`num_stages`、`BLOCK_M`、`BLOCK_N`、`BLOCK_K`、`use_fp8`。 该作用域涉及插件或编译器扩展点。

#### Lines 257-257

```python
    return (num_stages * BLOCK_K * (BLOCK_M + BLOCK_N) + BLOCK_M * BLOCK_N) * (1 if use_fp8 else 2) > 228 * 1024
```
- **EN:** Relevant themes: plugin or compiler extension points.
- **CN:** 相关主题：插件或编译器扩展点。

### Lines 258-272

```python


@pytest.mark.parametrize("M, N, K", [(32, 32, 32), (2048, 2048, 512)])
@pytest.mark.parametrize("BLOCK_SIZE_M", [128])
@pytest.mark.parametrize("BLOCK_SIZE_N", [128, 256])
@pytest.mark.parametrize("BLOCK_SIZE_K", [64, 128])
@pytest.mark.parametrize("num_stages", [0, 2, 3])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.parametrize("use_fp8", [False, True])
@pytest.mark.parametrize("a_use_tma", [False, True])
@pytest.mark.parametrize("b_use_tma", [False, True])
@pytest.mark.skipif(is_hip(), reason="warp specialization is not supported on hip devices")
@pytest.mark.skipif(not is_hopper_or_blackwell(), reason="Requires Hopper or Blackwell")
def test_warp_specialize_tma_matmul(M, N, K, BLOCK_SIZE_M, BLOCK_SIZE_N, BLOCK_SIZE_K, num_stages, num_warps, use_fp8,
                                    a_use_tma, b_use_tma):
```
- **EN:** Defines the test function `test_warp_specialize_tma_matmul`. Decorators: `pytest.mark.parametrize('M, N, K', [(32, 32, 32), (2048, 2048, 512)])`, `pytest.mark.parametrize('BLOCK_SIZE_M', [128])`, `pytest.mark.parametrize('BLOCK_SIZE_N', [128, 256])`, `pytest.mark.parametrize('BLOCK_SIZE_K', [64, 128])`, `pytest.mark.parametrize('num_stages', [0, 2, 3])`, `pytest.mark.parametrize('num_warps', [4, 8])`, and 5 more. Parameters: `M`, `N`, `K`, `BLOCK_SIZE_M`, `BLOCK_SIZE_N`, `BLOCK_SIZE_K`, `num_stages`, `num_warps`, and 3 more. Nested definitions in this scope: `alloc_fn`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `exceeds_smem_capacity`, `torch.manual_seed`, `triton.set_allocator`, `torch.empty`, and 14 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_warp_specialize_tma_matmul`。 装饰器：`pytest.mark.parametrize('M, N, K', [(32, 32, 32), (2048, 2048, 512)])`、`pytest.mark.parametrize('BLOCK_SIZE_M', [128])`、`pytest.mark.parametrize('BLOCK_SIZE_N', [128, 256])`、`pytest.mark.parametrize('BLOCK_SIZE_K', [64, 128])`、`pytest.mark.parametrize('num_stages', [0, 2, 3])`、`pytest.mark.parametrize('num_warps', [4, 8])` 等另外 5 项。 参数：`M`、`N`、`K`、`BLOCK_SIZE_M`、`BLOCK_SIZE_N`、`BLOCK_SIZE_K`、`num_stages`、`num_warps` 等另外 3 项。 该作用域中的嵌套定义：`alloc_fn`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`exceeds_smem_capacity`、`torch.manual_seed`、`triton.set_allocator`、`torch.empty` 等另外 14 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 273-274

```python
    if is_hopper() and not (a_use_tma and b_use_tma):
        pytest.skip("Hopper warp specialization requires all TMA loads")
```
- **EN:** Invokes `is_hopper`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hopper`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 275-276

```python
    if is_blackwell() and not a_use_tma and not b_use_tma:
        pytest.skip("Blackwell warp specialization requires at least one TMA load")
```
- **EN:** Invokes `is_blackwell`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_blackwell`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 277-278

```python
    if exceeds_smem_capacity(num_stages, BLOCK_SIZE_M, BLOCK_SIZE_N, BLOCK_SIZE_K, use_fp8=use_fp8):
        pytest.skip("uses too much shared memory")
```
- **EN:** Invokes `exceeds_smem_capacity`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `exceeds_smem_capacity`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

#### Lines 279-280

```python
    if num_stages == 0 and a_use_tma and b_use_tma and not use_fp8 and (BLOCK_SIZE_N, BLOCK_SIZE_K) == (256, 128):
        pytest.skip("uses too much shared memory")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

#### Lines 281-289

```python
    dtype = torch.float8_e4m3fn if use_fp8 else torch.float16

    GROUP_SIZE_M = 8

    device = "cuda"
    torch.manual_seed(42)
    A = torch.randn((M, K), dtype=torch.float16, device=device).to(dtype)
    B = torch.randn((N, K), dtype=torch.float16, device=device).to(dtype)
    C = torch.randn((M, N), dtype=torch.float16, device=device).to(dtype)
```
- **EN:** Prepares or updates state through `dtype`, `GROUP_SIZE_M`, `device`, `A`, `B`, `C`. Invokes `torch.manual_seed`, `torch.randn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `dtype`、`GROUP_SIZE_M`、`device`、`A`、`B`、`C` 准备或更新状态。 调用 `torch.manual_seed`、`torch.randn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 290-291

```python

    def alloc_fn(size, align, stream):
```
- **EN:** Defines the helper function `alloc_fn`. Parameters: `size`, `align`, `stream`. Key calls include `torch.empty`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `alloc_fn`。 参数：`size`、`align`、`stream`。 关键调用包括 `torch.empty`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 292-292

```python
        return torch.empty(size, dtype=torch.int8, device="cuda")
```
- **EN:** Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 293-305

```python

    triton.set_allocator(alloc_fn)

    grid = (triton.cdiv(M, BLOCK_SIZE_M) * triton.cdiv(N, BLOCK_SIZE_N), )
    kernel = matmul_tma_ws_kernel[grid](A, B, C, *A.stride(), *B.stride(), *C.stride(), M, N, K, num_stages,
                                        BLOCK_SIZE_M, BLOCK_SIZE_N, BLOCK_SIZE_K, GROUP_SIZE_M, num_warps=num_warps,
                                        USE_FP8=use_fp8, A_USE_TMA=a_use_tma, B_USE_TMA=b_use_tma)

    ref_out = torch.empty((M, N), dtype=dtype, device=device)
    cublas.matmul(A, B, ref_out)
    torch.testing.assert_close(ref_out.to(torch.float16), C.to(torch.float16), atol=0.03, rtol=0.03)

    ttgir = kernel.asm["ttgir"]
```
- **EN:** Prepares or updates state through `grid`, `kernel`, `ref_out`, `ttgir`. Invokes `triton.set_allocator`, `triton.cdiv`, `A.stride`, `B.stride`, `C.stride`, `torch.empty`, and 4 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 通过 `grid`、`kernel`、`ref_out`、`ttgir` 准备或更新状态。 调用 `triton.set_allocator`、`triton.cdiv`、`A.stride`、`B.stride`、`C.stride`、`torch.empty` 等另外 4 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 306-309

```python
    if is_blackwell():
        assert "ttng.tc_gen5_mma" in ttgir
    else:
        assert "ttng.warp_group_dot" in ttgir
```
- **EN:** Invokes `is_blackwell` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions.
- **CN:** 调用 `is_blackwell` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 310-311

```python
    if a_use_tma or b_use_tma:
        assert "ttng.async_tma_copy_global_to_local" in ttgir
```
- **EN:** Validates behavior with 1 assertion(s). Branches on runtime or test conditions.
- **CN:** 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 312-313

```python
    if (not a_use_tma or not b_use_tma) and num_stages > 1:
        assert "ttg.async_copy_global_to_local" in ttgir
```
- **EN:** Validates behavior with 1 assertion(s). Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

#### Lines 314-317

```python
    if is_hopper() and (num_warps == 8 or num_stages <= 1):
        assert "ttg.warp_specialize" not in ttgir
    else:
        assert "ttg.warp_specialize" in ttgir
```
- **EN:** Invokes `is_hopper` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `is_hopper` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

### Lines 318-325

```python


@pytest.mark.parametrize("M, N, K", [(512, 512, 512)])
@pytest.mark.parametrize("num_stages", [0, 3])
@pytest.mark.parametrize("a_use_tma", [False, True])
@pytest.mark.parametrize("b_use_tma", [False, True])
@pytest.mark.skipif(not is_hopper_or_blackwell(), reason="Requires Hopper or Blackwell")
def test_warp_specialize_tma_matmul_consan(M, N, K, num_stages, a_use_tma, b_use_tma, fresh_knobs):
```
- **EN:** Defines the test function `test_warp_specialize_tma_matmul_consan`. Decorators: `pytest.mark.parametrize('M, N, K', [(512, 512, 512)])`, `pytest.mark.parametrize('num_stages', [0, 3])`, `pytest.mark.parametrize('a_use_tma', [False, True])`, `pytest.mark.parametrize('b_use_tma', [False, True])`, `pytest.mark.skipif(not is_hopper_or_blackwell(), reason='Requires Hopper or Blackwell')`. Parameters: `M`, `N`, `K`, `num_stages`, `a_use_tma`, `b_use_tma`, `fresh_knobs`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `is_hopper`, `test_warp_specialize_tma_matmul`, `is_hopper_or_blackwell`. This scope touches pytest parametrization, matrix multiplication workflows, plugin or compiler extension points, debugging and inspection paths.
- **CN:** 定义测试函数 `test_warp_specialize_tma_matmul_consan`。 装饰器：`pytest.mark.parametrize('M, N, K', [(512, 512, 512)])`、`pytest.mark.parametrize('num_stages', [0, 3])`、`pytest.mark.parametrize('a_use_tma', [False, True])`、`pytest.mark.parametrize('b_use_tma', [False, True])`、`pytest.mark.skipif(not is_hopper_or_blackwell(), reason='Requires Hopper or Blackwell')`。 参数：`M`、`N`、`K`、`num_stages`、`a_use_tma`、`b_use_tma`、`fresh_knobs`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`is_hopper`、`test_warp_specialize_tma_matmul`、`is_hopper_or_blackwell`。 该作用域涉及pytest 参数化、矩阵乘法工作流、插件或编译器扩展点、调试与检查路径。

#### Lines 326-328

```python
    if is_hopper():
        # FIXME: Hopper warp specialization generates incorrect debug info.
        triton.knobs.compilation.disable_line_info = True
```
- **EN:** Invokes `is_hopper` to execute the test logic. Branches on runtime or test conditions. Relevant themes: debugging and inspection paths.
- **CN:** 调用 `is_hopper` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：调试与检查路径。

#### Lines 329-331

```python
    triton.knobs.compilation.instrumentation_mode = "consan"
    test_warp_specialize_tma_matmul(M, N, K, BLOCK_SIZE_M=128, BLOCK_SIZE_N=128, BLOCK_SIZE_K=64, num_stages=num_stages,
                                    num_warps=4, use_fp8=False, a_use_tma=a_use_tma, b_use_tma=b_use_tma)
```
- **EN:** Prepares or updates state through `triton`. Invokes `test_warp_specialize_tma_matmul` to execute the test logic. Relevant themes: matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 通过 `triton` 准备或更新状态。 调用 `test_warp_specialize_tma_matmul` 执行测试逻辑。 相关主题：矩阵乘法工作流、插件或编译器扩展点。

### Lines 332-351

```python


@triton.jit
def matmul_tma_persistent_ws_kernel(  #
        a_ptr, b_ptr, c_ptr,  #
        a_stride0, a_stride1,  #
        b_stride0, b_stride1,  #
        c_stride0, c_stride1,  #
        M, N, K,  #
        num_stages: tl.constexpr,  #
        BLOCK_SIZE_M: tl.constexpr,  #
        BLOCK_SIZE_N: tl.constexpr,  #
        BLOCK_SIZE_K: tl.constexpr,  #
        GROUP_SIZE_M: tl.constexpr,  #
        NUM_SMS: tl.constexpr,  #
        USE_FP8: tl.constexpr,  #
        FLATTEN: tl.constexpr,  #
        A_USE_TMA: tl.constexpr,  #
        B_USE_TMA: tl.constexpr,  #
):
```
- **EN:** Defines the helper function `matmul_tma_persistent_ws_kernel`. Decorators: `triton.jit`. Parameters: `a_ptr`, `b_ptr`, `c_ptr`, `a_stride0`, `a_stride1`, `b_stride0`, `b_stride1`, `c_stride0`, and 14 more. Key calls include `tl.make_tensor_descriptor`, `tl.program_id`, `tl.cdiv`, `_compute_pid`, `tl.zeros`, `accumulator.to`, and 3 more. This scope touches Triton compilation or JIT kernels, Triton language operations, tensor/descriptor metadata, matrix multiplication workflows.
- **CN:** 定义辅助函数 `matmul_tma_persistent_ws_kernel`。 装饰器：`triton.jit`。 参数：`a_ptr`、`b_ptr`、`c_ptr`、`a_stride0`、`a_stride1`、`b_stride0`、`b_stride1`、`c_stride0` 等另外 14 项。 关键调用包括 `tl.make_tensor_descriptor`、`tl.program_id`、`tl.cdiv`、`_compute_pid`、`tl.zeros`、`accumulator.to` 等另外 3 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、张量/描述符元数据、矩阵乘法工作流。

#### Lines 352-363

```python
    a_desc = tl.make_tensor_descriptor(a_ptr, shape=[M, K], strides=[a_stride0, a_stride1],
                                       block_shape=[BLOCK_SIZE_M, BLOCK_SIZE_K])
    b_desc = tl.make_tensor_descriptor(b_ptr, shape=[N, K], strides=[b_stride0, b_stride1],
                                       block_shape=[BLOCK_SIZE_N, BLOCK_SIZE_K])
    c_desc = tl.make_tensor_descriptor(c_ptr, shape=[M, N], strides=[c_stride0, c_stride1],
                                       block_shape=[BLOCK_SIZE_M, BLOCK_SIZE_N])

    start_pid = tl.program_id(axis=0)
    num_pid_m = tl.cdiv(M, BLOCK_SIZE_M)
    num_pid_n = tl.cdiv(N, BLOCK_SIZE_N)
    k_tiles = tl.cdiv(K, BLOCK_SIZE_K)
    num_tiles = num_pid_m * num_pid_n
```
- **EN:** Prepares or updates state through `a_desc`, `b_desc`, `c_desc`, `start_pid`, `num_pid_m`, `num_pid_n`, `k_tiles`, `num_tiles`. Invokes `tl.make_tensor_descriptor`, `tl.program_id`, `tl.cdiv` to execute the test logic. Relevant themes: Triton language operations, tensor/descriptor metadata.
- **CN:** 通过 `a_desc`、`b_desc`、`c_desc`、`start_pid`、`num_pid_m`、`num_pid_n`、`k_tiles`、`num_tiles` 准备或更新状态。 调用 `tl.make_tensor_descriptor`、`tl.program_id`、`tl.cdiv` 执行测试逻辑。 相关主题：Triton language 操作、张量/描述符元数据。

#### Lines 364-379

```python

    for tile_id in tl.range(start_pid, num_tiles, NUM_SMS, flatten=FLATTEN, warp_specialize=True,
                            num_stages=num_stages):
        pid_m, pid_n = _compute_pid(tile_id, num_pid_n, num_pid_m, GROUP_SIZE_M)

        off_am = pid_m * BLOCK_SIZE_M
        off_bn = pid_n * BLOCK_SIZE_N
        accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
        for ki in range(k_tiles):
            off_k = ki * BLOCK_SIZE_K
            a = _maybe_tma_load(a_desc, a_ptr, off_am, off_k, A_USE_TMA)
            b = _maybe_tma_load(b_desc, b_ptr, off_bn, off_k, B_USE_TMA)
            accumulator = tl.dot(a, b.T, accumulator)

        c = accumulator.to(tl.float8e4nv if USE_FP8 else tl.float16)
        c_desc.store((off_am, off_bn), c)
```
- **EN:** Invokes `_compute_pid`, `tl.zeros`, `accumulator.to`, `c_desc.store`, `_maybe_tma_load`, `tl.dot` to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `_compute_pid`、`tl.zeros`、`accumulator.to`、`c_desc.store`、`_maybe_tma_load`、`tl.dot` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

### Lines 380-395

```python


@pytest.mark.parametrize("M, N, K", [(32, 32, 32), (8192, 8192, 512)])
@pytest.mark.parametrize("BLOCK_SIZE_M", [128])
@pytest.mark.parametrize("BLOCK_SIZE_N", [128, 256])
@pytest.mark.parametrize("BLOCK_SIZE_K", [64, 128])
@pytest.mark.parametrize("num_stages", [2, 3])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.parametrize("use_fp8", [False, True])
@pytest.mark.parametrize("flatten", [False, True] if is_blackwell() else [True])
@pytest.mark.parametrize("a_use_tma", [False, True])
@pytest.mark.parametrize("b_use_tma", [False, True])
@pytest.mark.skipif(is_hip(), reason="warp specialization is not supported on hip devices")
@pytest.mark.skipif(not is_hopper_or_blackwell(), reason="Requires Hopper or Blackwell")
def test_warp_specialize_tma_matmul_persistent(M, N, K, BLOCK_SIZE_M, BLOCK_SIZE_N, BLOCK_SIZE_K, num_stages, num_warps,
                                               use_fp8, flatten, a_use_tma, b_use_tma):
```
- **EN:** Defines the test function `test_warp_specialize_tma_matmul_persistent`. Decorators: `pytest.mark.parametrize('M, N, K', [(32, 32, 32), (8192, 8192, 512)])`, `pytest.mark.parametrize('BLOCK_SIZE_M', [128])`, `pytest.mark.parametrize('BLOCK_SIZE_N', [128, 256])`, `pytest.mark.parametrize('BLOCK_SIZE_K', [64, 128])`, `pytest.mark.parametrize('num_stages', [2, 3])`, `pytest.mark.parametrize('num_warps', [4, 8])`, and 6 more. Parameters: `M`, `N`, `K`, `BLOCK_SIZE_M`, `BLOCK_SIZE_N`, `BLOCK_SIZE_K`, `num_stages`, `num_warps`, and 4 more. Nested definitions in this scope: `alloc_fn`, `grid`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `exceeds_smem_capacity`, `torch.manual_seed`, `triton.set_allocator`, `is_blackwell`, and 16 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_warp_specialize_tma_matmul_persistent`。 装饰器：`pytest.mark.parametrize('M, N, K', [(32, 32, 32), (8192, 8192, 512)])`、`pytest.mark.parametrize('BLOCK_SIZE_M', [128])`、`pytest.mark.parametrize('BLOCK_SIZE_N', [128, 256])`、`pytest.mark.parametrize('BLOCK_SIZE_K', [64, 128])`、`pytest.mark.parametrize('num_stages', [2, 3])`、`pytest.mark.parametrize('num_warps', [4, 8])` 等另外 6 项。 参数：`M`、`N`、`K`、`BLOCK_SIZE_M`、`BLOCK_SIZE_N`、`BLOCK_SIZE_K`、`num_stages`、`num_warps` 等另外 4 项。 该作用域中的嵌套定义：`alloc_fn`、`grid`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`exceeds_smem_capacity`、`torch.manual_seed`、`triton.set_allocator`、`is_blackwell` 等另外 16 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 396-397

```python
    if is_hopper() and not (a_use_tma and b_use_tma):
        pytest.skip("Hopper warp specialization requires all TMA loads")
```
- **EN:** Invokes `is_hopper`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_hopper`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 398-399

```python
    if exceeds_smem_capacity(num_stages, BLOCK_SIZE_M, BLOCK_SIZE_N, BLOCK_SIZE_K, use_fp8):
        pytest.skip("uses too much shared memory")
```
- **EN:** Invokes `exceeds_smem_capacity`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `exceeds_smem_capacity`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

#### Lines 400-409

```python
    dtype = torch.float8_e4m3fn if use_fp8 else torch.float16

    GROUP_SIZE_M = 8
    NUM_SMS = torch.cuda.get_device_properties("cuda").multi_processor_count

    device = "cuda"
    torch.manual_seed(42)
    A = torch.randn((M, K), dtype=torch.float16, device=device).to(dtype)
    B = torch.randn((N, K), dtype=torch.float16, device=device).to(dtype)
    C = torch.randn((M, N), dtype=torch.float16, device=device).to(dtype)
```
- **EN:** Prepares or updates state through `dtype`, `GROUP_SIZE_M`, `NUM_SMS`, `device`, `A`, `B`, `C`. Invokes `torch.cuda.get_device_properties`, `torch.manual_seed`, `torch.randn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `dtype`、`GROUP_SIZE_M`、`NUM_SMS`、`device`、`A`、`B`、`C` 准备或更新状态。 调用 `torch.cuda.get_device_properties`、`torch.manual_seed`、`torch.randn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 410-411

```python

    def alloc_fn(size, align, stream):
```
- **EN:** Defines the helper function `alloc_fn`. Parameters: `size`, `align`, `stream`. Key calls include `torch.empty`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `alloc_fn`。 参数：`size`、`align`、`stream`。 关键调用包括 `torch.empty`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 412-412

```python
        return torch.empty(size, dtype=torch.int8, device="cuda")
```
- **EN:** Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 413-414

```python

    triton.set_allocator(alloc_fn)
```
- **EN:** Invokes `triton.set_allocator` to execute the test logic.
- **CN:** 调用 `triton.set_allocator` 执行测试逻辑。

#### Lines 415-416

```python

    def grid(META):
```
- **EN:** Defines the helper function `grid`. Parameters: `META`. Key calls include `min`, `triton.cdiv`.
- **CN:** 定义辅助函数 `grid`。 参数：`META`。 关键调用包括 `min`、`triton.cdiv`。

##### Lines 417-420

```python
        return (min(
            NUM_SMS,
            triton.cdiv(M, META["BLOCK_SIZE_M"]) * triton.cdiv(N, META["BLOCK_SIZE_N"]),
        ), )
```
- **EN:** Invokes `min`, `triton.cdiv` to execute the test logic.
- **CN:** 调用 `min`、`triton.cdiv` 执行测试逻辑。

#### Lines 421-426

```python

    kernel = matmul_tma_persistent_ws_kernel[grid](A, B, C, *A.stride(), *B.stride(), *C.stride(), M, N, K, num_stages,
                                                   BLOCK_SIZE_M, BLOCK_SIZE_N, BLOCK_SIZE_K, GROUP_SIZE_M, NUM_SMS,
                                                   num_warps=num_warps, USE_FP8=use_fp8, FLATTEN=flatten
                                                   and is_blackwell(), A_USE_TMA=a_use_tma, B_USE_TMA=b_use_tma)
    ttgir = kernel.asm["ttgir"]
```
- **EN:** Prepares or updates state through `kernel`, `ttgir`. Invokes `A.stride`, `B.stride`, `C.stride`, `is_blackwell` to execute the test logic. Relevant themes: matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 通过 `kernel`、`ttgir` 准备或更新状态。 调用 `A.stride`、`B.stride`、`C.stride`、`is_blackwell` 执行测试逻辑。 相关主题：矩阵乘法工作流、插件或编译器扩展点。

#### Lines 427-430

```python
    if is_blackwell():
        assert "ttng.tc_gen5_mma" in ttgir
    else:
        assert "ttng.warp_group_dot" in ttgir
```
- **EN:** Invokes `is_blackwell` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions.
- **CN:** 调用 `is_blackwell` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 431-432

```python
    if a_use_tma or b_use_tma:
        assert "ttng.async_tma_copy_global_to_local" in ttgir
```
- **EN:** Validates behavior with 1 assertion(s). Branches on runtime or test conditions.
- **CN:** 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 433-434

```python
    if not a_use_tma or not b_use_tma:
        assert "ttg.async_copy_global_to_local" in ttgir
```
- **EN:** Validates behavior with 1 assertion(s). Branches on runtime or test conditions.
- **CN:** 通过 1 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 435-438

```python
    if is_hopper() and num_warps == 8:
        assert "ttg.warp_specialize" not in ttgir
    else:
        assert "ttg.warp_specialize" in ttgir
```
- **EN:** Invokes `is_hopper` to execute the test logic. Validates behavior with 2 assertion(s). Branches on runtime or test conditions.
- **CN:** 调用 `is_hopper` 执行测试逻辑。 通过 2 个断言验证行为。 根据运行时或测试条件进行分支。

#### Lines 439-442

```python

    ref_out = torch.empty((M, N), dtype=dtype, device=device)
    cublas.matmul(A, B, ref_out)
    torch.testing.assert_close(ref_out.to(torch.float16), C.to(torch.float16), atol=0.03, rtol=0.03)
```
- **EN:** Prepares or updates state through `ref_out`. Invokes `torch.empty`, `cublas.matmul`, `torch.testing.assert_close`, `ref_out.to`, `C.to` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 通过 `ref_out` 准备或更新状态。 调用 `torch.empty`、`cublas.matmul`、`torch.testing.assert_close`、`ref_out.to`、`C.to` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

### Lines 443-450

```python


@pytest.mark.parametrize("M, N, K", [(512, 512, 512)])
@pytest.mark.parametrize("a_use_tma", [False, True])
@pytest.mark.parametrize("b_use_tma", [False, True])
@pytest.mark.parametrize("flatten", [False, True] if is_blackwell() else [True])
@pytest.mark.skipif(not is_hopper_or_blackwell(), reason="Requires Hopper or Blackwell")
def test_warp_specialize_tma_matmul_persistent_consan(M, N, K, a_use_tma, b_use_tma, flatten, fresh_knobs):
```
- **EN:** Defines the test function `test_warp_specialize_tma_matmul_persistent_consan`. Decorators: `pytest.mark.parametrize('M, N, K', [(512, 512, 512)])`, `pytest.mark.parametrize('a_use_tma', [False, True])`, `pytest.mark.parametrize('b_use_tma', [False, True])`, `pytest.mark.parametrize('flatten', [False, True] if is_blackwell() else [True])`, `pytest.mark.skipif(not is_hopper_or_blackwell(), reason='Requires Hopper or Blackwell')`. Parameters: `M`, `N`, `K`, `a_use_tma`, `b_use_tma`, `flatten`, `fresh_knobs`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `is_hopper`, `test_warp_specialize_tma_matmul_persistent`, `is_blackwell`, `is_hopper_or_blackwell`. This scope touches pytest parametrization, matrix multiplication workflows, plugin or compiler extension points, debugging and inspection paths.
- **CN:** 定义测试函数 `test_warp_specialize_tma_matmul_persistent_consan`。 装饰器：`pytest.mark.parametrize('M, N, K', [(512, 512, 512)])`、`pytest.mark.parametrize('a_use_tma', [False, True])`、`pytest.mark.parametrize('b_use_tma', [False, True])`、`pytest.mark.parametrize('flatten', [False, True] if is_blackwell() else [True])`、`pytest.mark.skipif(not is_hopper_or_blackwell(), reason='Requires Hopper or Blackwell')`。 参数：`M`、`N`、`K`、`a_use_tma`、`b_use_tma`、`flatten`、`fresh_knobs`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`is_hopper`、`test_warp_specialize_tma_matmul_persistent`、`is_blackwell`、`is_hopper_or_blackwell`。 该作用域涉及pytest 参数化、矩阵乘法工作流、插件或编译器扩展点、调试与检查路径。

#### Lines 451-453

```python
    if is_hopper():
        # FIXME: Hopper warp specialization generates incorrect debug info.
        triton.knobs.compilation.disable_line_info = True
```
- **EN:** Invokes `is_hopper` to execute the test logic. Branches on runtime or test conditions. Relevant themes: debugging and inspection paths.
- **CN:** 调用 `is_hopper` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：调试与检查路径。

#### Lines 454-457

```python
    triton.knobs.compilation.instrumentation_mode = "consan"
    test_warp_specialize_tma_matmul_persistent(M, N, K, BLOCK_SIZE_M=128, BLOCK_SIZE_N=128, BLOCK_SIZE_K=64,
                                               num_stages=3, num_warps=4, use_fp8=False, flatten=flatten,
                                               a_use_tma=a_use_tma, b_use_tma=b_use_tma)
```
- **EN:** Prepares or updates state through `triton`. Invokes `test_warp_specialize_tma_matmul_persistent` to execute the test logic. Relevant themes: matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 通过 `triton` 准备或更新状态。 调用 `test_warp_specialize_tma_matmul_persistent` 执行测试逻辑。 相关主题：矩阵乘法工作流、插件或编译器扩展点。

### Lines 458-468

```python


@triton.jit
def attention_inner_loop_kernel(  #
        desc_q, desc_k, desc_v,  #
        desc_acc, l_i_ptr, m_i_ptr,  #
        M, N, qk_scale,  #
        BLOCK_M: tl.constexpr,  #
        HEAD_DIM: tl.constexpr,  #
        warp_specialize: tl.constexpr  #
):
```
- **EN:** Defines the helper function `attention_inner_loop_kernel`. Decorators: `triton.jit`. Parameters: `desc_q`, `desc_k`, `desc_v`, `desc_acc`, `l_i_ptr`, `m_i_ptr`, `M`, `N`, and 4 more. Key calls include `tl.zeros`, `desc_q.load`, `desc_acc.store`, `tl.store`, `tl.program_id`, `tl.multiple_of`, and 10 more. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `attention_inner_loop_kernel`。 装饰器：`triton.jit`。 参数：`desc_q`、`desc_k`、`desc_v`、`desc_acc`、`l_i_ptr`、`m_i_ptr`、`M`、`N` 等另外 4 项。 关键调用包括 `tl.zeros`、`desc_q.load`、`desc_acc.store`、`tl.store`、`tl.program_id`、`tl.multiple_of` 等另外 10 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 469-474

```python
    m_i = tl.zeros([BLOCK_M], dtype=tl.float32) - float("inf")
    l_i = tl.zeros([BLOCK_M], dtype=tl.float32) + 1.0
    acc = tl.zeros([BLOCK_M, HEAD_DIM], dtype=tl.float32)

    off_m = tl.program_id(0) * BLOCK_M
    q = desc_q.load([off_m, 0])
```
- **EN:** Prepares or updates state through `m_i`, `l_i`, `acc`, `off_m`, `q`. Invokes `tl.zeros`, `tl.program_id`, `desc_q.load` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `m_i`、`l_i`、`acc`、`off_m`、`q` 准备或更新状态。 调用 `tl.zeros`、`tl.program_id`、`desc_q.load` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 475-494

```python

    for start_n in tl.range(0, N, HEAD_DIM, warp_specialize=warp_specialize):
        start_n = tl.multiple_of(start_n, HEAD_DIM)
        k = desc_k.load([start_n, 0]).T

        qk = tl.dot(q, k)

        m_ij = tl.maximum(m_i, tl.max(qk, 1) * qk_scale)
        qk = qk * qk_scale - m_ij[:, None]
        p = tl.math.exp2(qk)
        alpha = tl.math.exp2(m_i - m_ij)
        l_ij = tl.sum(p, 1)
        acc = acc * alpha[:, None]

        v = desc_v.load([start_n, 0])
        p = p.to(v.dtype)
        acc = tl.dot(p, v, acc)

        l_i = l_i * alpha + l_ij
        m_i = m_ij
```
- **EN:** Invokes `tl.multiple_of`, `tl.dot`, `tl.maximum`, `tl.math.exp2`, `tl.sum`, `desc_v.load`, and 3 more to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.multiple_of`、`tl.dot`、`tl.maximum`、`tl.math.exp2`、`tl.sum`、`desc_v.load` 等另外 3 项 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作。

#### Lines 495-498

```python

    desc_acc.store([off_m, 0], acc.to(q.dtype))
    tl.store(l_i_ptr + off_m + tl.arange(0, BLOCK_M), l_i)
    tl.store(m_i_ptr + off_m + tl.arange(0, BLOCK_M), m_i)
```
- **EN:** Invokes `desc_acc.store`, `acc.to`, `tl.store`, `tl.arange` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `desc_acc.store`、`acc.to`、`tl.store`、`tl.arange` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 499-511

```python


@pytest.mark.parametrize("M, N", [(8192, 8192), (1024, 1024)])
@pytest.mark.parametrize("BLOCK_M", [64, 128])
@pytest.mark.parametrize("HEAD_DIM", [64, 128])
@pytest.mark.parametrize("num_stages", [2, 3])
@pytest.mark.parametrize("disable_acc_multibuf", [False, True])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.parametrize("use_fp8", [False, True])
@pytest.mark.skipif(is_hip(), reason="warp specialization is not supported on hip devices")
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_warp_specialize_attention_forward(M, N, BLOCK_M, HEAD_DIM, num_stages, disable_acc_multibuf, num_warps,
                                           use_fp8):
```
- **EN:** Defines the test function `test_warp_specialize_attention_forward`. Decorators: `pytest.mark.parametrize('M, N', [(8192, 8192), (1024, 1024)])`, `pytest.mark.parametrize('BLOCK_M', [64, 128])`, `pytest.mark.parametrize('HEAD_DIM', [64, 128])`, `pytest.mark.parametrize('num_stages', [2, 3])`, `pytest.mark.parametrize('disable_acc_multibuf', [False, True])`, `pytest.mark.parametrize('num_warps', [4, 8])`, and 3 more. Parameters: `M`, `N`, `BLOCK_M`, `HEAD_DIM`, `num_stages`, `disable_acc_multibuf`, `num_warps`, `use_fp8`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `torch.manual_seed`, `torch.empty`, `TensorDescriptor`, `torch.testing.assert_close`, and 10 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_warp_specialize_attention_forward`。 装饰器：`pytest.mark.parametrize('M, N', [(8192, 8192), (1024, 1024)])`、`pytest.mark.parametrize('BLOCK_M', [64, 128])`、`pytest.mark.parametrize('HEAD_DIM', [64, 128])`、`pytest.mark.parametrize('num_stages', [2, 3])`、`pytest.mark.parametrize('disable_acc_multibuf', [False, True])`、`pytest.mark.parametrize('num_warps', [4, 8])` 等另外 3 项。 参数：`M`、`N`、`BLOCK_M`、`HEAD_DIM`、`num_stages`、`disable_acc_multibuf`、`num_warps`、`use_fp8`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`torch.manual_seed`、`torch.empty`、`TensorDescriptor`、`torch.testing.assert_close` 等另外 10 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、插件或编译器扩展点。

#### Lines 512-515

```python
    if BLOCK_M == 128 and HEAD_DIM == 128 and not use_fp8:
        # These configurations currently use too much shared memory.
        if (num_warps, num_stages) in [(4, 4), (8, 4), (8, 3)]:
            pytest.skip("uses too much shared memory")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

#### Lines 516-545

```python

    dtype = torch.float8_e4m3fn if use_fp8 else torch.float16

    torch.manual_seed(42)
    q = torch.randn((M, HEAD_DIM), device="cuda").to(dtype)
    k = torch.randn((N, HEAD_DIM), device="cuda").to(dtype)
    v = torch.randn((N, HEAD_DIM), device="cuda").to(dtype)

    acc_ref = torch.empty((M, HEAD_DIM), dtype=dtype, device="cuda")
    l_i_ref = torch.empty((M, ), dtype=dtype, device="cuda")
    m_i_ref = torch.empty((M, ), dtype=dtype, device="cuda")
    acc = torch.empty((M, HEAD_DIM), dtype=dtype, device="cuda")
    l_i = torch.empty((M, ), dtype=dtype, device="cuda")
    m_i = torch.empty((M, ), dtype=dtype, device="cuda")

    desc_q = TensorDescriptor(q, shape=[M, HEAD_DIM], strides=[HEAD_DIM, 1], block_shape=[BLOCK_M, HEAD_DIM])
    desc_k = TensorDescriptor(k, shape=[N, HEAD_DIM], strides=[HEAD_DIM, 1], block_shape=[BLOCK_M, HEAD_DIM])
    desc_v = TensorDescriptor(v, shape=[N, HEAD_DIM], strides=[HEAD_DIM, 1], block_shape=[BLOCK_M, HEAD_DIM])
    desc_acc_ref = TensorDescriptor(acc_ref, shape=[M, HEAD_DIM], strides=[HEAD_DIM, 1],
                                    block_shape=[BLOCK_M, HEAD_DIM])
    desc_acc = TensorDescriptor(acc, shape=[M, HEAD_DIM], strides=[HEAD_DIM, 1], block_shape=[BLOCK_M, HEAD_DIM])

    attention_inner_loop_kernel[(M // BLOCK_M, )](desc_q, desc_k, desc_v, desc_acc_ref, l_i_ref, m_i_ref, M, N, 0.5,
                                                  BLOCK_M, HEAD_DIM, False, num_stages=num_stages, num_warps=num_warps)
    attention_inner_loop_kernel[(M // BLOCK_M, )](desc_q, desc_k, desc_v, desc_acc, l_i, m_i, M, N, 0.5, BLOCK_M,
                                                  HEAD_DIM, True, num_stages=num_stages, num_warps=num_warps)

    torch.testing.assert_close(acc.to(torch.float32), acc_ref.to(torch.float32), atol=0, rtol=0)
    torch.testing.assert_close(l_i.to(torch.float32), l_i_ref.to(torch.float32), atol=0, rtol=0)
    torch.testing.assert_close(m_i.to(torch.float32), m_i_ref.to(torch.float32), atol=0, rtol=0)
```
- **EN:** Prepares or updates state through `dtype`, `q`, `k`, `v`, `acc_ref`, `l_i_ref`, `m_i_ref`, `acc`, and 7 more. Invokes `torch.manual_seed`, `torch.randn`, `torch.empty`, `TensorDescriptor`, `torch.testing.assert_close`, `acc.to`, and 5 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, plugin or compiler extension points, random-data generation.
- **CN:** 通过 `dtype`、`q`、`k`、`v`、`acc_ref`、`l_i_ref`、`m_i_ref`、`acc` 等另外 7 项 准备或更新状态。 调用 `torch.manual_seed`、`torch.randn`、`torch.empty`、`TensorDescriptor`、`torch.testing.assert_close`、`acc.to` 等另外 5 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、插件或编译器扩展点、随机数据生成。

### Lines 546-556

```python


@triton.jit
def attention_persistent_inner_loop_kernel(  #
        desc_q, desc_k, desc_v,  #
        desc_acc, l_i_ptr, m_i_ptr,  #
        M, N, qk_scale,  #
        BLOCK_M: tl.constexpr,  #
        HEAD_DIM: tl.constexpr,  #
        warp_specialize: tl.constexpr,  #
        num_stages: tl.constexpr):
```
- **EN:** Defines the helper function `attention_persistent_inner_loop_kernel`. Decorators: `triton.jit`. Parameters: `desc_q`, `desc_k`, `desc_v`, `desc_acc`, `l_i_ptr`, `m_i_ptr`, `M`, `N`, and 5 more. Key calls include `tl.program_id`, `tl.num_programs`, `tl.cdiv`, `tl.zeros`, `desc_q.load`, `desc_acc.store`, and 12 more. This scope touches Triton compilation or JIT kernels, Triton language operations, plugin or compiler extension points.
- **CN:** 定义辅助函数 `attention_persistent_inner_loop_kernel`。 装饰器：`triton.jit`。 参数：`desc_q`、`desc_k`、`desc_v`、`desc_acc`、`l_i_ptr`、`m_i_ptr`、`M`、`N` 等另外 5 项。 关键调用包括 `tl.program_id`、`tl.num_programs`、`tl.cdiv`、`tl.zeros`、`desc_q.load`、`desc_acc.store` 等另外 12 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、插件或编译器扩展点。

#### Lines 557-561

```python
    prog_id = tl.program_id(0)
    num_sm = tl.num_programs(0)
    num_tiles = tl.cdiv(M, BLOCK_M)

    tiles_per_sm = num_tiles // num_sm
```
- **EN:** Prepares or updates state through `prog_id`, `num_sm`, `num_tiles`, `tiles_per_sm`. Invokes `tl.program_id`, `tl.num_programs`, `tl.cdiv` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `prog_id`、`num_sm`、`num_tiles`、`tiles_per_sm` 准备或更新状态。 调用 `tl.program_id`、`tl.num_programs`、`tl.cdiv` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 562-563

```python
    if prog_id < num_tiles % num_sm:
        tiles_per_sm += 1
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 564-565

```python

    tile_idx = prog_id
```
- **EN:** Prepares or updates state through `tile_idx`.
- **CN:** 通过 `tile_idx` 准备或更新状态。

#### Lines 566-598

```python
    for _ in tl.range(0, tiles_per_sm, warp_specialize=warp_specialize, num_stages=num_stages):
        m_i = tl.zeros([BLOCK_M], dtype=tl.float32) - float("inf")
        l_i = tl.zeros([BLOCK_M], dtype=tl.float32) + 1.0
        acc = tl.zeros([BLOCK_M, HEAD_DIM], dtype=tl.float32)

        off_m = tile_idx * BLOCK_M
        q = desc_q.load([off_m, 0])

        for start_n in tl.range(0, N, HEAD_DIM):
            start_n = tl.multiple_of(start_n, HEAD_DIM)
            k = desc_k.load([start_n, 0]).T

            qk = tl.dot(q, k)

            m_ij = tl.maximum(m_i, tl.max(qk, 1) * qk_scale)
            qk = qk * qk_scale - m_ij[:, None]
            p = tl.math.exp2(qk)
            alpha = tl.math.exp2(m_i - m_ij)
            l_ij = tl.sum(p, 1)
            acc = acc * alpha[:, None]

            v = desc_v.load([start_n, 0])
            p = p.to(v.dtype)
            acc = tl.dot(p, v, acc)

            l_i = l_i * alpha + l_ij
            m_i = m_ij

        desc_acc.store([off_m, 0], acc.to(q.dtype))
        tl.store(l_i_ptr + off_m + tl.arange(0, BLOCK_M), l_i)
        tl.store(m_i_ptr + off_m + tl.arange(0, BLOCK_M), m_i)

        tile_idx += num_sm
```
- **EN:** Invokes `tl.zeros`, `desc_q.load`, `desc_acc.store`, `tl.store`, `tl.multiple_of`, `tl.dot`, and 9 more to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, plugin or compiler extension points.
- **CN:** 调用 `tl.zeros`、`desc_q.load`、`desc_acc.store`、`tl.store`、`tl.multiple_of`、`tl.dot` 等另外 9 项 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、插件或编译器扩展点。

### Lines 599-610

```python


@pytest.mark.parametrize("M, N", [(8192, 8192), (1024, 1024)])
@pytest.mark.parametrize("BLOCK_M", [64, 128])
@pytest.mark.parametrize("HEAD_DIM", [64, 128])
@pytest.mark.parametrize("num_stages", [2, 3])
@pytest.mark.parametrize("disable_acc_multibuf", [False, True])
@pytest.mark.parametrize("num_warps", [4, 8])
@pytest.mark.parametrize("use_fp8", [False, True])
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_warp_specialize_attention_persistent_forward(M, N, BLOCK_M, HEAD_DIM, num_stages, disable_acc_multibuf,
                                                      num_warps, use_fp8):
```
- **EN:** Defines the test function `test_warp_specialize_attention_persistent_forward`. Decorators: `pytest.mark.parametrize('M, N', [(8192, 8192), (1024, 1024)])`, `pytest.mark.parametrize('BLOCK_M', [64, 128])`, `pytest.mark.parametrize('HEAD_DIM', [64, 128])`, `pytest.mark.parametrize('num_stages', [2, 3])`, `pytest.mark.parametrize('disable_acc_multibuf', [False, True])`, `pytest.mark.parametrize('num_warps', [4, 8])`, and 2 more. Parameters: `M`, `N`, `BLOCK_M`, `HEAD_DIM`, `num_stages`, `disable_acc_multibuf`, `num_warps`, `use_fp8`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `torch.manual_seed`, `torch.empty`, `TensorDescriptor`, `torch.testing.assert_close`, and 9 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, tensor/descriptor metadata, plugin or compiler extension points.
- **CN:** 定义测试函数 `test_warp_specialize_attention_persistent_forward`。 装饰器：`pytest.mark.parametrize('M, N', [(8192, 8192), (1024, 1024)])`、`pytest.mark.parametrize('BLOCK_M', [64, 128])`、`pytest.mark.parametrize('HEAD_DIM', [64, 128])`、`pytest.mark.parametrize('num_stages', [2, 3])`、`pytest.mark.parametrize('disable_acc_multibuf', [False, True])`、`pytest.mark.parametrize('num_warps', [4, 8])` 等另外 2 项。 参数：`M`、`N`、`BLOCK_M`、`HEAD_DIM`、`num_stages`、`disable_acc_multibuf`、`num_warps`、`use_fp8`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`torch.manual_seed`、`torch.empty`、`TensorDescriptor`、`torch.testing.assert_close` 等另外 9 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、张量/描述符元数据、插件或编译器扩展点。

#### Lines 611-614

```python
    if BLOCK_M == 128 and HEAD_DIM == 128 and not use_fp8:
        # These configurations currently use too much shared memory.
        if (num_warps, num_stages) in [(4, 4), (8, 4), (8, 3), (4, 3)]:
            pytest.skip("uses too much shared memory")
```
- **EN:** Invokes `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: plugin or compiler extension points.
- **CN:** 调用 `pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：插件或编译器扩展点。

#### Lines 615-646

```python

    dtype = torch.float8_e4m3fn if use_fp8 else torch.float16

    torch.manual_seed(42)

    q = torch.randn((M, HEAD_DIM), device="cuda").to(dtype)
    k = torch.randn((N, HEAD_DIM), device="cuda").to(dtype)
    v = torch.randn((N, HEAD_DIM), device="cuda").to(dtype)

    acc_ref = torch.empty((M, HEAD_DIM), dtype=dtype, device="cuda")
    l_i_ref = torch.empty((M, ), dtype=dtype, device="cuda")
    m_i_ref = torch.empty((M, ), dtype=dtype, device="cuda")
    acc = torch.empty((M, HEAD_DIM), dtype=dtype, device="cuda")
    l_i = torch.empty((M, ), dtype=dtype, device="cuda")
    m_i = torch.empty((M, ), dtype=dtype, device="cuda")

    desc_q = TensorDescriptor(q, shape=[M, HEAD_DIM], strides=[HEAD_DIM, 1], block_shape=[BLOCK_M, HEAD_DIM])
    desc_k = TensorDescriptor(k, shape=[N, HEAD_DIM], strides=[HEAD_DIM, 1], block_shape=[BLOCK_M, HEAD_DIM])
    desc_v = TensorDescriptor(v, shape=[N, HEAD_DIM], strides=[HEAD_DIM, 1], block_shape=[BLOCK_M, HEAD_DIM])
    desc_acc_ref = TensorDescriptor(acc_ref, shape=[M, HEAD_DIM], strides=[HEAD_DIM, 1],
                                    block_shape=[BLOCK_M, HEAD_DIM])
    desc_acc = TensorDescriptor(acc, shape=[M, HEAD_DIM], strides=[HEAD_DIM, 1], block_shape=[BLOCK_M, HEAD_DIM])

    NUM_SM = 4
    attention_persistent_inner_loop_kernel[(NUM_SM, )](desc_q, desc_k, desc_v, desc_acc, l_i, m_i, M, N, 0.5, BLOCK_M,
                                                       HEAD_DIM, True, num_stages=num_stages, num_warps=num_warps)
    attention_inner_loop_kernel[(M // BLOCK_M, )](desc_q, desc_k, desc_v, desc_acc_ref, l_i_ref, m_i_ref, M, N, 0.5,
                                                  BLOCK_M, HEAD_DIM, False, num_stages=num_stages, num_warps=num_warps)

    torch.testing.assert_close(acc.to(torch.float32), acc_ref.to(torch.float32), atol=0, rtol=0)
    torch.testing.assert_close(l_i.to(torch.float32), l_i_ref.to(torch.float32), atol=0, rtol=0)
    torch.testing.assert_close(m_i.to(torch.float32), m_i_ref.to(torch.float32), atol=0, rtol=0)
```
- **EN:** Prepares or updates state through `dtype`, `q`, `k`, `v`, `acc_ref`, `l_i_ref`, `m_i_ref`, `acc`, and 8 more. Invokes `torch.manual_seed`, `torch.randn`, `torch.empty`, `TensorDescriptor`, `torch.testing.assert_close`, `acc.to`, and 5 more to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata, plugin or compiler extension points, random-data generation.
- **CN:** 通过 `dtype`、`q`、`k`、`v`、`acc_ref`、`l_i_ref`、`m_i_ref`、`acc` 等另外 8 项 准备或更新状态。 调用 `torch.manual_seed`、`torch.randn`、`torch.empty`、`TensorDescriptor`、`torch.testing.assert_close`、`acc.to` 等另外 5 项 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据、插件或编译器扩展点、随机数据生成。

### Lines 647-663

```python


@triton.jit
def grouped_matmul_tma_kernel(
    group_a_ptrs,
    group_b_ptrs,
    group_c_ptrs,
    gm,
    gn,
    gk,
    g_lds,
    group_size,
    NUM_SM: tl.constexpr,
    BLOCK_SIZE_M: tl.constexpr,
    BLOCK_SIZE_N: tl.constexpr,
    BLOCK_SIZE_K: tl.constexpr,
):
```
- **EN:** Defines the helper function `grouped_matmul_tma_kernel`. Decorators: `triton.jit`. Parameters: `group_a_ptrs`, `group_b_ptrs`, `group_c_ptrs`, `gm`, `gn`, `gk`, `g_lds`, `group_size`, and 4 more. Key calls include `tl.cdiv`, `tl.program_id`, `tl.load`, `tl.make_tensor_descriptor`, `tl.pointer_type`, `tl.zeros`, and 5 more. This scope touches Triton compilation or JIT kernels, Triton language operations, tensor/descriptor metadata, matrix multiplication workflows.
- **CN:** 定义辅助函数 `grouped_matmul_tma_kernel`。 装饰器：`triton.jit`。 参数：`group_a_ptrs`、`group_b_ptrs`、`group_c_ptrs`、`gm`、`gn`、`gk`、`g_lds`、`group_size` 等另外 4 项。 关键调用包括 `tl.cdiv`、`tl.program_id`、`tl.load`、`tl.make_tensor_descriptor`、`tl.pointer_type`、`tl.zeros` 等另外 5 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、张量/描述符元数据、矩阵乘法工作流。

#### Lines 664-668

```python
    dtype = tl.float16
    num_m_tiles = tl.cdiv(gm, BLOCK_SIZE_M)
    num_n_tiles = tl.cdiv(gn, BLOCK_SIZE_N)
    num_tiles = num_m_tiles * num_n_tiles
    start_pid = tl.program_id(axis=0)
```
- **EN:** Prepares or updates state through `dtype`, `num_m_tiles`, `num_n_tiles`, `num_tiles`, `start_pid`. Invokes `tl.cdiv`, `tl.program_id` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `dtype`、`num_m_tiles`、`num_n_tiles`、`num_tiles`、`start_pid` 准备或更新状态。 调用 `tl.cdiv`、`tl.program_id` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 669-715

```python

    for g in tl.range(group_size, warp_specialize=True):
        lda = tl.load(g_lds + g * 3)
        ldb = tl.load(g_lds + g * 3 + 1)
        ldc = tl.load(g_lds + g * 3 + 2)

        a_ptr = tl.load(group_a_ptrs + g).to(tl.pointer_type(dtype))
        b_ptr = tl.load(group_b_ptrs + g).to(tl.pointer_type(dtype))
        c_ptr = tl.load(group_c_ptrs + g).to(tl.pointer_type(dtype))

        a_desc = tl.make_tensor_descriptor(
            a_ptr,
            shape=[gm, gk],
            strides=[lda, 1],
            block_shape=[BLOCK_SIZE_M, BLOCK_SIZE_K],
        )

        b_desc = tl.make_tensor_descriptor(
            b_ptr,
            shape=[gn, gk],
            strides=[ldb, 1],
            block_shape=[BLOCK_SIZE_N, BLOCK_SIZE_K],
        )
        c_desc = tl.make_tensor_descriptor(
            c_ptr,
            shape=[gm, gn],
            strides=[ldc, 1],
            block_shape=[BLOCK_SIZE_M, BLOCK_SIZE_N],
        )

        for tile_idx in tl.range(start_pid, num_tiles, NUM_SM):
            tile_m_idx = tile_idx // num_n_tiles
            tile_n_idx = tile_idx % num_n_tiles
            offs_am = tile_m_idx * BLOCK_SIZE_M
            offs_bn = tile_n_idx * BLOCK_SIZE_N

            accumulator = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
            for kk in range(0, tl.cdiv(gk, BLOCK_SIZE_K)):
                a = a_desc.load([offs_am, kk * BLOCK_SIZE_K])
                b = b_desc.load([offs_bn, kk * BLOCK_SIZE_K])
                accumulator += tl.dot(a, b.T)

            offs_cm = tile_m_idx * BLOCK_SIZE_M
            offs_cn = tile_n_idx * BLOCK_SIZE_N

            c = accumulator.to(dtype)
            c_desc.store([offs_cm, offs_cn], c)
```
- **EN:** Invokes `tl.load`, `tl.make_tensor_descriptor`, `tl.pointer_type`, `tl.zeros`, `accumulator.to`, `c_desc.store`, and 4 more to execute the test logic. Iterates across cases or data tiles. Relevant themes: Triton language operations, tensor/descriptor metadata.
- **CN:** 调用 `tl.load`、`tl.make_tensor_descriptor`、`tl.pointer_type`、`tl.zeros`、`accumulator.to`、`c_desc.store` 等另外 4 项 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：Triton language 操作、张量/描述符元数据。

### Lines 716-718

```python


def group_gemm_tma_fn(group_A, group_B):
```
- **EN:** Defines the helper function `group_gemm_tma_fn`. Parameters: `group_A`, `group_B`. Nested definitions in this scope: `alloc_fn`. Key calls include `torch.tensor`, `triton.set_allocator`, `torch.empty`, `group_C.append`, `A_addrs.append`, `B_addrs.append`, and 7 more. This scope touches PyTorch tensor setup and checks, matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 定义辅助函数 `group_gemm_tma_fn`。 参数：`group_A`、`group_B`。 该作用域中的嵌套定义：`alloc_fn`。 关键调用包括 `torch.tensor`、`triton.set_allocator`、`torch.empty`、`group_C.append`、`A_addrs.append`、`B_addrs.append` 等另外 7 项。 该作用域涉及PyTorch 张量准备与校验、矩阵乘法工作流、插件或编译器扩展点。

#### Lines 719-719

```python
    assert len(group_A) == len(group_B)
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 720-728

```python
    group_size = len(group_A)

    A_addrs = []
    B_addrs = []
    C_addrs = []
    g_lds = []
    group_C = []
    M, K = group_A[0].shape
    N, _ = group_B[0].shape
```
- **EN:** Prepares or updates state through `group_size`, `A_addrs`, `B_addrs`, `C_addrs`, `g_lds`, `group_C`, `M`, `K`, and 2 more.
- **CN:** 通过 `group_size`、`A_addrs`、`B_addrs`、`C_addrs`、`g_lds`、`group_C`、`M`、`K` 等另外 2 项 准备或更新状态。

#### Lines 729-738

```python

    for i in range(group_size):
        A = group_A[i]
        B = group_B[i]
        C = torch.empty((M, N), device="cuda", dtype=A.dtype)
        group_C.append(C)
        A_addrs.append(A.data_ptr())
        B_addrs.append(B.data_ptr())
        C_addrs.append(C.data_ptr())
        g_lds += [A.stride(0), B.stride(0), C.stride(0)]
```
- **EN:** Invokes `torch.empty`, `group_C.append`, `A_addrs.append`, `B_addrs.append`, `C_addrs.append`, `A.data_ptr`, and 5 more to execute the test logic. Iterates across cases or data tiles. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.empty`、`group_C.append`、`A_addrs.append`、`B_addrs.append`、`C_addrs.append`、`A.data_ptr` 等另外 5 项 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：PyTorch 张量准备与校验。

#### Lines 739-743

```python

    d_a_ptrs = torch.tensor(A_addrs, device="cuda")
    d_b_ptrs = torch.tensor(B_addrs, device="cuda")
    d_c_ptrs = torch.tensor(C_addrs, device="cuda")
    d_g_lds = torch.tensor(g_lds, dtype=torch.int32, device="cuda")
```
- **EN:** Prepares or updates state through `d_a_ptrs`, `d_b_ptrs`, `d_c_ptrs`, `d_g_lds`. Invokes `torch.tensor` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `d_a_ptrs`、`d_b_ptrs`、`d_c_ptrs`、`d_g_lds` 准备或更新状态。 调用 `torch.tensor` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 744-745

```python

    def alloc_fn(size: int, _, __):
```
- **EN:** Defines the helper function `alloc_fn`. Parameters: `size`, `_`, `__`. Key calls include `torch.empty`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `alloc_fn`。 参数：`size`、`_`、`__`。 关键调用包括 `torch.empty`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 746-746

```python
        return torch.empty(size, device="cuda", dtype=torch.int8)
```
- **EN:** Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

#### Lines 747-752

```python

    triton.set_allocator(alloc_fn)

    grid = lambda META: (META['NUM_SM'], )
    out = grouped_matmul_tma_kernel[grid](d_a_ptrs, d_b_ptrs, d_c_ptrs, M, N, K, d_g_lds, group_size, BLOCK_SIZE_M=128,
                                          BLOCK_SIZE_N=128, BLOCK_SIZE_K=64, NUM_SM=4, num_stages=3)
```
- **EN:** Prepares or updates state through `grid`, `out`. Invokes `triton.set_allocator` to execute the test logic. Relevant themes: matrix multiplication workflows, plugin or compiler extension points.
- **CN:** 通过 `grid`、`out` 准备或更新状态。 调用 `triton.set_allocator` 执行测试逻辑。 相关主题：矩阵乘法工作流、插件或编译器扩展点。

#### Lines 753-753

```python
    assert "ttg.warp_specialize" in out.asm["ttgir"]
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

#### Lines 754-754

```python
    return group_C
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 755-762

```python


@pytest.mark.parametrize("M", [128, 256, 512, 1024, 2048, 4096, 8192])
@pytest.mark.parametrize("N", [256, 512, 1024, 2048, 4096, 8192])
@pytest.mark.parametrize("K", [128, 512, 1024, 2048, 4096])
@pytest.mark.parametrize("group_size", [4, 8, 16])
@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_grouped_gemm(M, N, K, group_size):
```
- **EN:** Defines the test function `test_grouped_gemm`. Decorators: `pytest.mark.parametrize('M', [128, 256, 512, 1024, 2048, 4096, 8192])`, `pytest.mark.parametrize('N', [256, 512, 1024, 2048, 4096, 8192])`, `pytest.mark.parametrize('K', [128, 512, 1024, 2048, 4096])`, `pytest.mark.parametrize('group_size', [4, 8, 16])`, `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Parameters: `M`, `N`, `K`, `group_size`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `torch.manual_seed`, `group_gemm_tma_fn`, `torch.rand`, `B.T.contiguous`, and 7 more. This scope touches pytest parametrization, PyTorch tensor setup and checks, matrix multiplication workflows, random-data generation.
- **CN:** 定义测试函数 `test_grouped_gemm`。 装饰器：`pytest.mark.parametrize('M', [128, 256, 512, 1024, 2048, 4096, 8192])`、`pytest.mark.parametrize('N', [256, 512, 1024, 2048, 4096, 8192])`、`pytest.mark.parametrize('K', [128, 512, 1024, 2048, 4096])`、`pytest.mark.parametrize('group_size', [4, 8, 16])`、`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 参数：`M`、`N`、`K`、`group_size`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`torch.manual_seed`、`group_gemm_tma_fn`、`torch.rand`、`B.T.contiguous` 等另外 7 项。 该作用域涉及pytest 参数化、PyTorch 张量准备与校验、矩阵乘法工作流、随机数据生成。

#### Lines 763-766

```python
    torch.manual_seed(42)
    group_A = []
    group_B = []
    group_B_T = []
```
- **EN:** Prepares or updates state through `group_A`, `group_B`, `group_B_T`. Invokes `torch.manual_seed` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 通过 `group_A`、`group_B`、`group_B_T` 准备或更新状态。 调用 `torch.manual_seed` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 767-774

```python

    for i in range(group_size):
        A = torch.rand((M, K), device="cuda", dtype=torch.float16)
        B = torch.rand((K, N), device="cuda", dtype=torch.float16)
        B_T = B.T.contiguous()
        group_A.append(A)
        group_B.append(B)
        group_B_T.append(B_T)
```
- **EN:** Invokes `torch.rand`, `B.T.contiguous`, `group_A.append`, `group_B.append`, `group_B_T.append` to execute the test logic. Iterates across cases or data tiles. Relevant themes: PyTorch tensor setup and checks, random-data generation.
- **CN:** 调用 `torch.rand`、`B.T.contiguous`、`group_A.append`、`group_B.append`、`group_B_T.append` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：PyTorch 张量准备与校验、随机数据生成。

#### Lines 775-778

```python

    ref_out = [torch.matmul(a, b) for a, b in zip(group_A, group_B)]

    tri_tma_out = group_gemm_tma_fn(group_A, group_B_T)
```
- **EN:** Prepares or updates state through `ref_out`, `tri_tma_out`. Invokes `torch.matmul`, `zip`, `group_gemm_tma_fn` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, matrix multiplication workflows.
- **CN:** 通过 `ref_out`、`tri_tma_out` 准备或更新状态。 调用 `torch.matmul`、`zip`、`group_gemm_tma_fn` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、矩阵乘法工作流。

#### Lines 779-780

```python
    for i in range(group_size):
        assert torch.allclose(ref_out[i], tri_tma_out[i], atol=1e-2, rtol=1e-2)
```
- **EN:** Invokes `torch.allclose` to execute the test logic. Validates behavior with 1 assertion(s). Iterates across cases or data tiles. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.allclose` 执行测试逻辑。 通过 1 个断言验证行为。 通过循环覆盖多个用例或数据分块。 相关主题：PyTorch 张量准备与校验。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `is_hopper_or_blackwell`, `test_warp_specialize_basic_ir`, `test_warp_specialize_tmem_ir`, `test_warpgroup_reduction`, `_compute_pid`, `_maybe_tma_load`, `matmul_tma_ws_kernel`, `exceeds_smem_capacity`
  **CN:** 顶层作用域，例如 `is_hopper_or_blackwell`、`test_warp_specialize_basic_ir`、`test_warp_specialize_tmem_ir`、`test_warpgroup_reduction`、`_compute_pid`、`_maybe_tma_load`、`matmul_tma_ws_kernel`、`exceeds_smem_capacity`
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

- **EN:** External or absolute imports include `torch`, `pytest`, `pathlib`, `triton`, `triton.language`, `triton._internal_testing`, `triton.tools.tensor_descriptor`, `triton._C.libtriton`.
  **CN:** 外部或绝对导入包括 `torch`、`pytest`、`pathlib`、`triton`、`triton.language`、`triton._internal_testing`、`triton.tools.tensor_descriptor`、`triton._C.libtriton`。
- **EN:** Execution centers on top-level definitions such as `is_hopper_or_blackwell`, `test_warp_specialize_basic_ir`, `test_warp_specialize_tmem_ir`, `test_warpgroup_reduction`, `_compute_pid`, `_maybe_tma_load`, `matmul_tma_ws_kernel`, `exceeds_smem_capacity`, `test_warp_specialize_tma_matmul`, `test_warp_specialize_tma_matmul_consan`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `is_hopper_or_blackwell`、`test_warp_specialize_basic_ir`、`test_warp_specialize_tmem_ir`、`test_warpgroup_reduction`、`_compute_pid`、`_maybe_tma_load`、`matmul_tma_ws_kernel`、`exceeds_smem_capacity`、`test_warp_specialize_tma_matmul`、`test_warp_specialize_tma_matmul_consan`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
