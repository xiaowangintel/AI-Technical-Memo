# test_aot.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/unit/tools/test_aot.py`
- **EN:** Pytest module covering aot behavior in Triton's Python tests. It contains 16 top-level definition(s) and 13 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 aot 行为。 该文件包含 16 个顶层定义，以及 13 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13

```python
import glob
import os
import pytest
import re
import subprocess
import sys
import tempfile

import numpy as np

import triton
from triton.backends.compiler import GPUTarget
from triton._internal_testing import is_cuda, is_hip
```
- **EN:** Imports the modules used in this scope: `glob`, `os`, `pytest`, `re`, `subprocess`, `sys`, `tempfile`, `numpy`, `triton`, `triton.backends.compiler`, and 1 more. Relevant themes: subprocess-driven validation.
- **CN:** 导入此作用域使用的模块：`glob`、`os`、`pytest`、`re`、`subprocess`、`sys`、`tempfile`、`numpy`、`triton`、`triton.backends.compiler` 等另外 1 项。 相关主题：基于子进程的验证。

### Lines 14-29

```python

if is_cuda():
    from triton.backends.nvidia.driver import include_dirs, library_dirs

    def library_names():
        return ["cuda"]

elif is_hip():
    from triton.backends.amd.driver import include_dirs, _get_path_to_hip_runtime_dylib

    def library_dirs():
        hip_runtime_dylib = _get_path_to_hip_runtime_dylib()
        return [os.path.dirname(hip_runtime_dylib)]

    def library_names():
        return ["amdhip64"]
```
- **EN:** Invokes `is_cuda`, `is_hip`, `_get_path_to_hip_runtime_dylib`, `os.path.dirname` to execute the test logic. Branches on runtime or test conditions. Relevant themes: runtime driver interaction.
- **CN:** 调用 `is_cuda`、`is_hip`、`_get_path_to_hip_runtime_dylib`、`os.path.dirname` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：运行时驱动交互。

### Lines 30-80

```python


kernel_utils_src = """
import triton

@triton.jit
def mul(x, y):
    return x * y
"""

kernel_src = """
import triton
import triton.language as tl
import kernel_utils

@triton.jit
def kernel(C, A, B, M, N, K,
          stride_cm, stride_cn,
          stride_am, stride_ak,
          stride_bk, stride_bn,
          BLOCK_M: tl.constexpr,
          BLOCK_N: tl.constexpr,
          BLOCK_K: tl.constexpr):
  pid_m = tl.program_id(0)
  pid_n = tl.program_id(1)

  offs_am = (pid_m * BLOCK_M + tl.arange(0, BLOCK_M)) % M
  offs_bn = (pid_n * BLOCK_N + tl.arange(0, BLOCK_N)) % N
  offs_k = tl.arange(0, BLOCK_K)
  a_ptrs = A + (offs_am[:, None] * stride_am + offs_k[None, :] * stride_ak)
  b_ptrs = B + (offs_k[:, None] * stride_bk + offs_bn[None, :] * stride_bn)

  accumulator = tl.zeros((BLOCK_M, BLOCK_N), dtype=tl.float32)
  for k in range(0, tl.cdiv(K, BLOCK_K)):
      # Load the next block of A and B, generate a mask by checking the K dimension.
      # If it is out of bounds, set it to 0.
      a = tl.load(a_ptrs, mask=offs_k[None, :] < K - k * BLOCK_K, other=0.0)
      b = tl.load(b_ptrs, mask=offs_k[:, None] < K - k * BLOCK_K, other=0.0)
      # We accumulate along the K dimension.
      accumulator += tl.dot(a, b)
      # Advance the ptrs to the next K block.
      a_ptrs += BLOCK_K * stride_ak
      b_ptrs += BLOCK_K * stride_bk

  c = kernel_utils.mul(accumulator, accumulator)
  # Write back the block of the output matrix C with masks.
  offs_cm = pid_m * BLOCK_M + tl.arange(0, BLOCK_M)
  offs_cn = pid_n * BLOCK_N + tl.arange(0, BLOCK_N)
  c_ptrs = C + stride_cm * offs_cm[:, None] + stride_cn * offs_cn[None, :]
  tl.store(c_ptrs, c)
"""
```
- **EN:** Prepares or updates state through `kernel_utils_src`, `kernel_src`. Relevant themes: Triton compilation or JIT kernels, Triton language operations.
- **CN:** 通过 `kernel_utils_src`、`kernel_src` 准备或更新状态。 相关主题：Triton 编译或 JIT kernel、Triton language 操作。

### Lines 81-83

```python


def get_gluon_kernel_src(threads_per_warp):
```
- **EN:** Defines the helper function `get_gluon_kernel_src`. Parameters: `threads_per_warp`. This scope touches layout transformation reasoning.
- **CN:** 定义辅助函数 `get_gluon_kernel_src`。 参数：`threads_per_warp`。 该作用域涉及布局变换推理。

#### Lines 84-102

```python
    return f"""
from triton.experimental import gluon
from triton.experimental.gluon import language as gl

@gluon.jit
def kernel(
    C, A, B, M, N, K,
    stride_cm, stride_cn,
    stride_am, stride_ak,
    stride_bk, stride_bn,
    BLOCK_M: gl.constexpr,
    BLOCK_N: gl.constexpr,
    BLOCK_K: gl.constexpr
):
    layout: gl.constexpr = gl.BlockedLayout(size_per_thread=[1], threads_per_warp=[{threads_per_warp}], warps_per_cta=[1], order=[0])
    offs = gl.arange(0, 64, layout=layout)
    a = gl.load(A + offs)
    gl.store(B + offs, a)
"""
```
- **EN:** Relevant themes: layout transformation reasoning.
- **CN:** 相关主题：布局变换推理。

### Lines 103-116

```python


if is_cuda():
    test_utils_src = """
#include <cuda.h>

// Forward declaration for backward compatibility with CUDA 12.x and 13.x
CUresult cuCtxCreate_v2(CUcontext *pctx, unsigned int flags, CUdevice dev);
"""
elif is_hip():
    test_utils_src = """
#define __HIP_PLATFORM_AMD__
#include <hip/hip_runtime.h>
"""
```
- **EN:** Invokes `is_cuda`, `is_hip` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `is_cuda`、`is_hip` 执行测试逻辑。 根据运行时或测试条件进行分支。

### Lines 117-151

```python

test_utils_src += """
#include <stdio.h>
#include <stdint.h>
#include <string.h>
#include <assert.h>
#include "kernel.h"

static void write_buffer_to_csv(char *filename, int32_t *buffer, int size) {
    FILE *file = fopen(filename, "w");
    if (file == NULL) {
        printf("Could not open file %s\\n", filename);
        return;
    }
    for (int i = 0; i < size; i++) {
        fprintf(file, "%d", buffer[i]);
        if (i < size - 1) {
            fprintf(file, ",");
        }
    }
    fclose(file);
}

static void read_csv_to_buffer(char *filename, int16_t *buffer, int size) {
    FILE *file = fopen(filename, "r");
    if (file == NULL) {
        printf("Could not open file %s\\n", filename);
        return;
    }
    int index = 0;
    while (fscanf(file, "%hd,", &buffer[index]) != EOF && index < size) {
        index++;
    }
    fclose(file);
}"""
```
- **EN:** Prepares or updates state through `test_utils_src`.
- **CN:** 通过 `test_utils_src` 准备或更新状态。

### Lines 152-154

```python


def gen_kernel_library(dir, libname):
```
- **EN:** Defines the helper function `gen_kernel_library`. Parameters: `dir`, `libname`. Key calls include `glob.glob`, `subprocess.run`, `library_dirs`, `os.path.join`, `command.extend`. This scope touches subprocess-driven validation.
- **CN:** 定义辅助函数 `gen_kernel_library`。 参数：`dir`、`libname`。 关键调用包括 `glob.glob`、`subprocess.run`、`library_dirs`、`os.path.join`、`command.extend`。 该作用域涉及基于子进程的验证。

#### Lines 155-163

```python
    c_files = glob.glob(os.path.join(dir, "*.c"))
    subprocess.run(
        ["gcc"] + c_files + ["-I", include_dirs[0], "-c", "-fPIC"],
        check=True,
        cwd=dir,
    )
    o_files = glob.glob(os.path.join(dir, "*.o"))

    command = ["gcc", *o_files, "-shared", "-o", libname]
```
- **EN:** Prepares or updates state through `c_files`, `o_files`, `command`. Invokes `glob.glob`, `os.path.join`, `subprocess.run` to execute the test logic. Relevant themes: subprocess-driven validation.
- **CN:** 通过 `c_files`、`o_files`、`command` 准备或更新状态。 调用 `glob.glob`、`os.path.join`、`subprocess.run` 执行测试逻辑。 相关主题：基于子进程的验证。

#### Lines 164-165

```python
    for lib_dir in library_dirs():
        command.extend(["-L", lib_dir])
```
- **EN:** Invokes `library_dirs`, `command.extend` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `library_dirs`、`command.extend` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 166-166

```python
    subprocess.run(command, check=True, cwd=dir)
```
- **EN:** Invokes `subprocess.run` to execute the test logic. Relevant themes: subprocess-driven validation.
- **CN:** 调用 `subprocess.run` 执行测试逻辑。 相关主题：基于子进程的验证。

### Lines 167-169

```python


def gen_test_bin(dir, M, N, K, exe="test", algo_id=0):
```
- **EN:** Defines the helper function `gen_test_bin`. Parameters: `dir`, `M`, `N`, `K`, `exe`, `algo_id`. Key calls include `is_cuda`, `library_dirs`, `library_names`, `command.extend`, `subprocess.run`, `is_hip`, and 3 more. This scope touches subprocess-driven validation, matrix multiplication workflows, kernel launch orchestration.
- **CN:** 定义辅助函数 `gen_test_bin`。 参数：`dir`、`M`、`N`、`K`、`exe`、`algo_id`。 关键调用包括 `is_cuda`、`library_dirs`、`library_names`、`command.extend`、`subprocess.run`、`is_hip` 等另外 3 项。 该作用域涉及基于子进程的验证、矩阵乘法工作流、kernel 启动编排。

#### Lines 170-279

```python
    if is_cuda():
        test_src = f"""
int main(int argc, char **argv) {{
  int M = {M}, N = {N}, K = {K};

  // initialize CUDA handles
  CUdevice dev;
  CUcontext ctx;
  CUstream stream;
  CUdeviceptr A, B, C;
  CUresult err = 0;
  cuInit(0);
  cuDeviceGet(&dev, 0);
  cuCtxCreate_v2(&ctx, 0, dev);
  cuMemAlloc(&A, M * K * 2);
  cuMemAlloc(&B, K * N * 2);
  cuMemAlloc(&C, M * N * 4);
  cuStreamCreate(&stream, 0);
  load_matmul_fp16();

  // initialize input data
  int16_t hA[M*K];
  int16_t hB[K*N];
  memset(hA, 0, M*K*2);
  memset(hB, 0, K*N*2);
  read_csv_to_buffer(argv[1], hA, M*K);
  read_csv_to_buffer(argv[2], hB, K*N);
  cuMemcpyHtoD(A, hA, M*K*2);
  cuMemcpyHtoD(B, hB, K*N*2);

  // launch kernel
  CUresult ret;
  int algo_id = {algo_id};
  if (algo_id == 0) {{
    ret = matmul_fp16_default(stream, C, A, B, M, N, K, N, 1, K, 1, N, 1);
  }} else {{
    ret = matmul_fp16(stream, C, A, B, M, N, K, N, 1, K, 1, N, 1, {algo_id});
  }}
  if (ret != 0) fprintf(stderr, "kernel launch failed\\n");
  assert(ret == 0);

  // read data
  int32_t hC[M*N];
  memset(hC, 0, M*N*4);
  cuMemcpyDtoH(hC, C, M*N*4);
  write_buffer_to_csv(argv[3], hC, M*N);

  // free cuda handles
  unload_matmul_fp16();
  cuMemFree(A);
  cuMemFree(B);
  cuMemFree(C);
  cuCtxDestroy(ctx);
}}
"""
    elif is_hip():
        test_src = f"""
int main(int argc, char **argv) {{
  int M = {M}, N = {N}, K = {K};

  // initialize hip handles
  hipDevice_t dev;
  // hipCtx_t ctx;
  hipStream_t stream;
  hipDeviceptr_t A, B, C;
  hipError_t err = 0;
  hipInit(0);
  hipDeviceGet(&dev, 0);
  // hipCtxCreate(&ctx, 0, dev);
  hipMalloc(&A, M * K * 2);
  hipMalloc(&B, K * N * 2);
  hipMalloc(&C, M * N * 4);
  hipStreamCreateWithFlags(&stream, 0);
  load_matmul_fp16();

  // initialize input data
  int16_t hA[M*K];
  int16_t hB[K*N];
  memset(hA, 0, M*K*2);
  memset(hB, 0, K*N*2);
  read_csv_to_buffer(argv[1], hA, M*K);
  read_csv_to_buffer(argv[2], hB, K*N);
  hipMemcpyHtoD(A, hA, M*K*2);
  hipMemcpyHtoD(B, hB, K*N*2);

  // launch kernel
  hipError_t ret;
  int algo_id = {algo_id};
  if (algo_id == 0) {{
    ret = matmul_fp16_default(stream, C, A, B, M, N, K, N, 1, K, 1, N, 1);
  }} else {{
    ret = matmul_fp16(stream, C, A, B, M, N, K, N, 1, K, 1, N, 1, {algo_id});
  }}
  if (ret != 0) fprintf(stderr, "kernel launch failed\\n");
  assert(ret == 0);

  // read data
  int32_t hC[M*N];
  memset(hC, 0, M*N*4);
  hipMemcpyDtoH(hC, C, M*N*4);
  write_buffer_to_csv(argv[3], hC, M*N);

  // free hip handles
  unload_matmul_fp16();
  hipFree(A);
  hipFree(B);
  hipFree(C);
  // hipCtxDestroy(ctx);
}}
"""
```
- **EN:** Invokes `is_cuda`, `is_hip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: matrix multiplication workflows, kernel launch orchestration.
- **CN:** 调用 `is_cuda`、`is_hip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：矩阵乘法工作流、kernel 启动编排。

#### Lines 280-280

```python
    src = test_utils_src + test_src
```
- **EN:** Prepares or updates state through `src`.
- **CN:** 通过 `src` 准备或更新状态。

#### Lines 281-282

```python
    with open(os.path.join(dir, "test.c"), "w") as file:
        file.write(src)
```
- **EN:** Invokes `open`, `file.write`, `os.path.join` to execute the test logic. Uses context managers to control resources or expectations.
- **CN:** 调用 `open`、`file.write`、`os.path.join` 执行测试逻辑。 使用上下文管理器控制资源或预期行为。

#### Lines 283-284

```python

    command = ["gcc", "test.c"]
```
- **EN:** Prepares or updates state through `command`.
- **CN:** 通过 `command` 准备或更新状态。

#### Lines 285-286

```python
    for inc_dir in include_dirs:
        command.extend(["-I", inc_dir])
```
- **EN:** Invokes `command.extend` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `command.extend` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 287-288

```python
    for lib_dir in library_dirs():
        command.extend(["-L", lib_dir])
```
- **EN:** Invokes `library_dirs`, `command.extend` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `library_dirs`、`command.extend` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 289-290

```python
    for lib_name in library_names():
        command.extend(["-l", lib_name])
```
- **EN:** Invokes `library_names`, `command.extend` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `library_names`、`command.extend` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 291-292

```python
    command.extend(["-L", dir, "-l", "kernel", "-o", exe])
    subprocess.run(command, check=True, cwd=dir)
```
- **EN:** Invokes `command.extend`, `subprocess.run` to execute the test logic. Relevant themes: subprocess-driven validation.
- **CN:** 调用 `command.extend`、`subprocess.run` 执行测试逻辑。 相关主题：基于子进程的验证。

### Lines 293-295

```python


def write_triton_kernels(dir, src, util_src):
```
- **EN:** Defines the helper function `write_triton_kernels`. Parameters: `dir`, `src`, `util_src`. Key calls include `os.path.join`, `open`, `file.write`.
- **CN:** 定义辅助函数 `write_triton_kernels`。 参数：`dir`、`src`、`util_src`。 关键调用包括 `os.path.join`、`open`、`file.write`。

#### Lines 296-296

```python
    kernel_path = os.path.join(dir, "kernel.py")
```
- **EN:** Prepares or updates state through `kernel_path`. Invokes `os.path.join` to execute the test logic.
- **CN:** 通过 `kernel_path` 准备或更新状态。 调用 `os.path.join` 执行测试逻辑。

#### Lines 297-298

```python
    with open(kernel_path, "w") as file:
        file.write(src)
```
- **EN:** Invokes `open`, `file.write` to execute the test logic. Uses context managers to control resources or expectations.
- **CN:** 调用 `open`、`file.write` 执行测试逻辑。 使用上下文管理器控制资源或预期行为。

#### Lines 299-300

```python

    kernel_utils_path = os.path.join(dir, "kernel_utils.py")
```
- **EN:** Prepares or updates state through `kernel_utils_path`. Invokes `os.path.join` to execute the test logic.
- **CN:** 通过 `kernel_utils_path` 准备或更新状态。 调用 `os.path.join` 执行测试逻辑。

#### Lines 301-302

```python
    with open(kernel_utils_path, "w") as file:
        file.write(util_src)
```
- **EN:** Invokes `open`, `file.write` to execute the test logic. Uses context managers to control resources or expectations.
- **CN:** 调用 `open`、`file.write` 执行测试逻辑。 使用上下文管理器控制资源或预期行为。

#### Lines 303-304

```python

    return kernel_path
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 305-307

```python


def _compile_kernel(dir, signature, kernel_name, out_name, out_path, num_warps, grid, kernel_path, target=None):
```
- **EN:** Defines the helper function `_compile_kernel`. Parameters: `dir`, `signature`, `kernel_name`, `out_name`, `out_path`, `num_warps`, `grid`, `kernel_path`, and 1 more. Key calls include `os.path.join`, `cmd_args.append`, `subprocess.run`, `cmd_args.extend`. This scope touches subprocess-driven validation.
- **CN:** 定义辅助函数 `_compile_kernel`。 参数：`dir`、`signature`、`kernel_name`、`out_name`、`out_path`、`num_warps`、`grid`、`kernel_path` 等另外 1 项。 关键调用包括 `os.path.join`、`cmd_args.append`、`subprocess.run`、`cmd_args.extend`。 该作用域涉及基于子进程的验证。

#### Lines 308-313

```python
    compiler_path = os.path.join(triton.tools.__path__[0], "compile.py")
    cmd_args = [
        sys.executable, compiler_path, "-n", kernel_name, "--signature", signature, "--out-name", out_name, "-o",
        out_path, "-w",
        str(num_warps), "-g", grid
    ]
```
- **EN:** Prepares or updates state through `compiler_path`, `cmd_args`. Invokes `os.path.join` to execute the test logic.
- **CN:** 通过 `compiler_path`、`cmd_args` 准备或更新状态。 调用 `os.path.join` 执行测试逻辑。

#### Lines 314-315

```python
    if target:
        cmd_args.extend(["-t", "%s:%s:%i" % (target.backend, target.arch, target.warp_size)])
```
- **EN:** Invokes `cmd_args.extend` to execute the test logic. Branches on runtime or test conditions.
- **CN:** 调用 `cmd_args.extend` 执行测试逻辑。 根据运行时或测试条件进行分支。

#### Lines 316-317

```python
    cmd_args.append(kernel_path)
    subprocess.run(cmd_args, check=True, cwd=dir)
```
- **EN:** Invokes `cmd_args.append`, `subprocess.run` to execute the test logic. Relevant themes: subprocess-driven validation.
- **CN:** 调用 `cmd_args.append`、`subprocess.run` 执行测试逻辑。 相关主题：基于子进程的验证。

### Lines 318-322

```python


# Edge case kernel with no specialization
def compile_aot_kernel_no_specialization(dir, kernel_path, dtype, BM, BN, BK, target=None):
    # compile all desired configs
```
- **EN:** Defines the helper function `compile_aot_kernel_no_specialization`. Parameters: `dir`, `kernel_path`, `dtype`, `BM`, `BN`, `BK`, `target`. Key calls include `_compile_kernel`. This scope touches matrix multiplication workflows.
- **CN:** 定义辅助函数 `compile_aot_kernel_no_specialization`。 参数：`dir`、`kernel_path`、`dtype`、`BM`、`BN`、`BK`、`target`。 关键调用包括 `_compile_kernel`。 该作用域涉及矩阵乘法工作流。

#### Lines 323-336

```python
    sig = f"*fp32, *{dtype}, *{dtype}, i32, i32, i32, i32, i32, i32, i32, i32, i32, {BM}, {BN}, {BK}"
    name = f"matmul_{dtype}"
    grid = f"M/{BM}, N/{BN}, 1"
    _compile_kernel(
        dir=dir,
        signature=sig,
        kernel_name="kernel",
        out_name=name,
        out_path=name,
        num_warps=1,
        grid=grid,
        kernel_path=kernel_path,
        target=target,
    )
```
- **EN:** Prepares or updates state through `sig`, `name`, `grid`. Invokes `_compile_kernel` to execute the test logic. Relevant themes: matrix multiplication workflows.
- **CN:** 通过 `sig`、`name`、`grid` 准备或更新状态。 调用 `_compile_kernel` 执行测试逻辑。 相关主题：矩阵乘法工作流。

### Lines 337-340

```python


def compile_aot_kernels(dir, kernel_path, dtype, BM, BN, BK, ha_hb_hints, target=None):
    # compile all desired configs
```
- **EN:** Defines the helper function `compile_aot_kernels`. Parameters: `dir`, `kernel_path`, `dtype`, `BM`, `BN`, `BK`, `ha_hb_hints`, `target`. Key calls include `_compile_kernel`. This scope touches matrix multiplication workflows.
- **CN:** 定义辅助函数 `compile_aot_kernels`。 参数：`dir`、`kernel_path`、`dtype`、`BM`、`BN`、`BK`、`ha_hb_hints`、`target`。 关键调用包括 `_compile_kernel`。 该作用域涉及矩阵乘法工作流。

#### Lines 341-355

```python
    for ha, hb in ha_hb_hints:
        sig = f"*fp32:16, *{dtype}:16, *{dtype}:16, i32, i32, i32, i32{ha}, i32:1, i32{hb}, i32:1, i32:16, i32:1, {BM}, {BN}, {BK}"
        name = f"matmul_{dtype}"
        grid = f"M/{BM}, N/{BN}, 1"
        _compile_kernel(
            dir=dir,
            signature=sig,
            kernel_name="kernel",
            out_name=name,
            out_path=name,
            num_warps=1,
            grid=grid,
            kernel_path=kernel_path,
            target=target,
        )
```
- **EN:** Invokes `_compile_kernel` to execute the test logic. Iterates across cases or data tiles. Relevant themes: matrix multiplication workflows.
- **CN:** 调用 `_compile_kernel` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。 相关主题：矩阵乘法工作流。

### Lines 356-358

```python


def link_aot_kernels(dir):
```
- **EN:** Defines the helper function `link_aot_kernels`. Parameters: `dir`. Key calls include `os.path.join`, `glob.glob`, `subprocess.run`. This scope touches subprocess-driven validation.
- **CN:** 定义辅助函数 `link_aot_kernels`。 参数：`dir`。 关键调用包括 `os.path.join`、`glob.glob`、`subprocess.run`。 该作用域涉及基于子进程的验证。

#### Lines 359-363

```python
    linker_path = os.path.join(triton.tools.__path__[0], "link.py")

    # link all desired configs
    h_files = glob.glob(os.path.join(dir, "*.h"))
    subprocess.run([sys.executable, linker_path] + h_files + ["-o", "kernel"], check=True, cwd=dir)
```
- **EN:** Prepares or updates state through `linker_path`, `h_files`. Invokes `os.path.join`, `glob.glob`, `subprocess.run` to execute the test logic. Relevant themes: subprocess-driven validation.
- **CN:** 通过 `linker_path`、`h_files` 准备或更新状态。 调用 `os.path.join`、`glob.glob`、`subprocess.run` 执行测试逻辑。 相关主题：基于子进程的验证。

### Lines 364-366

```python


def generate_matmul_test_data(dir, M, N, K):
```
- **EN:** Defines the helper function `generate_matmul_test_data`. Parameters: `dir`, `M`, `N`, `K`. Key calls include `os.path.join`, `np.random.randn`, `x.view`. This scope touches matrix multiplication workflows, random-data generation.
- **CN:** 定义辅助函数 `generate_matmul_test_data`。 参数：`dir`、`M`、`N`、`K`。 关键调用包括 `os.path.join`、`np.random.randn`、`x.view`。 该作用域涉及矩阵乘法工作流、随机数据生成。

#### Lines 367-371

```python
    a = np.random.randn(M * K).astype(np.float16).reshape((M, K))
    b = np.random.randn(N * K).astype(np.float16).reshape((K, N))
    a_path = os.path.join(dir, "a.csv")
    b_path = os.path.join(dir, "b.csv")
    c_path = os.path.join(dir, "c.csv")
```
- **EN:** Prepares or updates state through `a`, `b`, `a_path`, `b_path`, `c_path`. Invokes `np.random.randn`, `os.path.join` to execute the test logic. Relevant themes: random-data generation.
- **CN:** 通过 `a`、`b`、`a_path`、`b_path`、`c_path` 准备或更新状态。 调用 `np.random.randn`、`os.path.join` 执行测试逻辑。 相关主题：随机数据生成。

#### Lines 372-373

```python
    for x, path in [(a, a_path), (b, b_path)]:
        x.view(np.int16).ravel().tofile(path, sep=",")
```
- **EN:** Invokes `x.view` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `x.view` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 374-374

```python
    return a, b, a_path, b_path, c_path
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 375-378

```python


def check_hasco_binary_str(tmp_dir: str, dtype: str):
    # Linking is not yet enabled on HIP backend so just check compilation for now.
```
- **EN:** Defines the helper function `check_hasco_binary_str`. Parameters: `tmp_dir`, `dtype`. Key calls include `glob.glob`, `re.compile`, `open`, `c_file.read`, `pattern.findall`, `os.path.join`. This scope touches matrix multiplication workflows.
- **CN:** 定义辅助函数 `check_hasco_binary_str`。 参数：`tmp_dir`、`dtype`。 关键调用包括 `glob.glob`、`re.compile`、`open`、`c_file.read`、`pattern.findall`、`os.path.join`。 该作用域涉及矩阵乘法工作流。

#### Lines 379-380

```python
    h_files = glob.glob(f"matmul_{dtype}.*.h", root_dir=tmp_dir)
    c_files = glob.glob(f"matmul_{dtype}.*.c", root_dir=tmp_dir)
```
- **EN:** Prepares or updates state through `h_files`, `c_files`. Invokes `glob.glob` to execute the test logic. Relevant themes: matrix multiplication workflows.
- **CN:** 通过 `h_files`、`c_files` 准备或更新状态。 调用 `glob.glob` 执行测试逻辑。 相关主题：矩阵乘法工作流。

#### Lines 381-382

```python
    assert len(h_files) == 1, "Expected one .h file"
    assert len(c_files) == 1, "Expected one .c file"
```
- **EN:** Validates behavior with 2 assertion(s).
- **CN:** 通过 2 个断言验证行为。

#### Lines 383-383

```python
    pattern = re.compile(r'HSACO_NAME\[(\d+)\]')
```
- **EN:** Prepares or updates state through `pattern`. Invokes `re.compile` to execute the test logic.
- **CN:** 通过 `pattern` 准备或更新状态。 调用 `re.compile` 执行测试逻辑。

#### Lines 384-388

```python
    with open(os.path.join(tmp_dir, c_files[0]), "r") as c_file:
        content = c_file.read()
        matches = pattern.findall(content)
        assert len(matches) == 1, "Expected one HSACO_NAME definition"
        assert int(matches[0]) > 16, "Expected valid HSACO object binary string"
```
- **EN:** Invokes `open`, `c_file.read`, `pattern.findall`, `os.path.join` to execute the test logic. Validates behavior with 2 assertion(s). Uses context managers to control resources or expectations.
- **CN:** 调用 `open`、`c_file.read`、`pattern.findall`、`os.path.join` 执行测试逻辑。 通过 2 个断言验证行为。 使用上下文管理器控制资源或预期行为。

### Lines 389-392

```python


# Test edge case where the provided kernel signature has no specializations
def test_compile_link_matmul_no_specialization():
```
- **EN:** Defines the test function `test_compile_link_matmul_no_specialization`. Key calls include `np.random.seed`, `tempfile.TemporaryDirectory`, `write_triton_kernels`, `compile_aot_kernel_no_specialization`, `is_hip`, `link_aot_kernels`, and 12 more. This scope touches subprocess-driven validation, matrix multiplication workflows, random-data generation.
- **CN:** 定义测试函数 `test_compile_link_matmul_no_specialization`。 关键调用包括 `np.random.seed`、`tempfile.TemporaryDirectory`、`write_triton_kernels`、`compile_aot_kernel_no_specialization`、`is_hip`、`link_aot_kernels` 等另外 12 项。 该作用域涉及基于子进程的验证、矩阵乘法工作流、随机数据生成。

#### Lines 393-393

```python
    np.random.seed(3)
```
- **EN:** Invokes `np.random.seed` to execute the test logic. Relevant themes: random-data generation.
- **CN:** 调用 `np.random.seed` 执行测试逻辑。 相关主题：随机数据生成。

#### Lines 394-423

```python

    with tempfile.TemporaryDirectory() as tmp_dir:
        dtype = "fp16"
        BM, BN, BK = 16, 16, 16

        kernel_path = write_triton_kernels(tmp_dir, kernel_src, kernel_utils_src)
        compile_aot_kernel_no_specialization(tmp_dir, kernel_path, dtype, BM, BN, BK)
        if is_hip():
            check_hasco_binary_str(tmp_dir, dtype)

        link_aot_kernels(tmp_dir)

        # compile test case
        M, N, K = 16, 16, 16
        gen_kernel_library(tmp_dir, "libkernel.so")
        gen_test_bin(tmp_dir, M, N, K)

        # initialize test data
        a, b, a_path, b_path, c_path = generate_matmul_test_data(tmp_dir, M, N, K)

        # run test case
        env = os.environ.copy()
        env["LD_LIBRARY_PATH"] = tmp_dir
        subprocess.run(["./test", a_path, b_path, c_path], env=env, check=True, cwd=tmp_dir)

        # read data and compare against reference
        c = np.genfromtxt(c_path, delimiter=",", dtype=np.int32)
        c_tri = c.reshape((M, N)).view(np.float32)
        c_ref = np.matmul(a.astype(np.float32), b.astype(np.float32))
        np.testing.assert_allclose(c_tri, c_ref * c_ref, atol=1e-4, rtol=0.0)
```
- **EN:** Invokes `tempfile.TemporaryDirectory`, `write_triton_kernels`, `compile_aot_kernel_no_specialization`, `is_hip`, `link_aot_kernels`, `gen_kernel_library`, and 11 more to execute the test logic. Uses context managers to control resources or expectations. Relevant themes: subprocess-driven validation, matrix multiplication workflows.
- **CN:** 调用 `tempfile.TemporaryDirectory`、`write_triton_kernels`、`compile_aot_kernel_no_specialization`、`is_hip`、`link_aot_kernels`、`gen_kernel_library` 等另外 11 项 执行测试逻辑。 使用上下文管理器控制资源或预期行为。 相关主题：基于子进程的验证、矩阵乘法工作流。

### Lines 424-426

```python


def test_compile_link_matmul():
```
- **EN:** Defines the test function `test_compile_link_matmul`. Key calls include `np.random.seed`, `tempfile.TemporaryDirectory`, `write_triton_kernels`, `compile_aot_kernels`, `is_hip`, `link_aot_kernels`, and 12 more. This scope touches subprocess-driven validation, matrix multiplication workflows, random-data generation.
- **CN:** 定义测试函数 `test_compile_link_matmul`。 关键调用包括 `np.random.seed`、`tempfile.TemporaryDirectory`、`write_triton_kernels`、`compile_aot_kernels`、`is_hip`、`link_aot_kernels` 等另外 12 项。 该作用域涉及基于子进程的验证、矩阵乘法工作流、随机数据生成。

#### Lines 427-427

```python
    np.random.seed(3)
```
- **EN:** Invokes `np.random.seed` to execute the test logic. Relevant themes: random-data generation.
- **CN:** 调用 `np.random.seed` 执行测试逻辑。 相关主题：随机数据生成。

#### Lines 428-456

```python

    with tempfile.TemporaryDirectory() as tmp_dir:
        dtype = "fp16"
        BM, BN, BK = 16, 16, 16

        kernel_path = write_triton_kernels(tmp_dir, kernel_src, kernel_utils_src)
        compile_aot_kernels(tmp_dir, kernel_path, dtype, BM, BN, BK, ha_hb_hints=[(":16", ":16")])
        if is_hip():
            check_hasco_binary_str(tmp_dir, dtype)
        link_aot_kernels(tmp_dir)

        # compile test case
        M, N, K = 16, 16, 16
        gen_kernel_library(tmp_dir, "libkernel.so")
        gen_test_bin(tmp_dir, M, N, K)

        # initialize test data
        a, b, a_path, b_path, c_path = generate_matmul_test_data(tmp_dir, M, N, K)

        # run test case
        env = os.environ.copy()
        env["LD_LIBRARY_PATH"] = tmp_dir
        subprocess.run(["./test", a_path, b_path, c_path], env=env, check=True, cwd=tmp_dir)

        # read data and compare against reference
        c = np.genfromtxt(c_path, delimiter=",", dtype=np.int32)
        c_tri = c.reshape((M, N)).view(np.float32)
        c_ref = np.matmul(a.astype(np.float32), b.astype(np.float32))
        np.testing.assert_allclose(c_tri, c_ref * c_ref, atol=1e-4, rtol=0.0)
```
- **EN:** Invokes `tempfile.TemporaryDirectory`, `write_triton_kernels`, `compile_aot_kernels`, `is_hip`, `link_aot_kernels`, `gen_kernel_library`, and 11 more to execute the test logic. Uses context managers to control resources or expectations. Relevant themes: subprocess-driven validation, matrix multiplication workflows.
- **CN:** 调用 `tempfile.TemporaryDirectory`、`write_triton_kernels`、`compile_aot_kernels`、`is_hip`、`link_aot_kernels`、`gen_kernel_library` 等另外 11 项 执行测试逻辑。 使用上下文管理器控制资源或预期行为。 相关主题：基于子进程的验证、矩阵乘法工作流。

### Lines 457-459

```python


def test_launcher_has_no_available_kernel():
```
- **EN:** Defines the test function `test_launcher_has_no_available_kernel`. Key calls include `np.random.seed`, `tempfile.TemporaryDirectory`, `write_triton_kernels`, `compile_aot_kernels`, `is_hip`, `link_aot_kernels`, and 6 more. This scope touches subprocess-driven validation, matrix multiplication workflows, kernel launch orchestration, random-data generation.
- **CN:** 定义测试函数 `test_launcher_has_no_available_kernel`。 关键调用包括 `np.random.seed`、`tempfile.TemporaryDirectory`、`write_triton_kernels`、`compile_aot_kernels`、`is_hip`、`link_aot_kernels` 等另外 6 项。 该作用域涉及基于子进程的验证、矩阵乘法工作流、kernel 启动编排、随机数据生成。

#### Lines 460-460

```python
    np.random.seed(3)
```
- **EN:** Invokes `np.random.seed` to execute the test logic. Relevant themes: random-data generation.
- **CN:** 调用 `np.random.seed` 执行测试逻辑。 相关主题：随机数据生成。

#### Lines 461-494

```python

    with tempfile.TemporaryDirectory() as tmp_dir:
        dtype = "fp16"
        BM, BN, BK = 16, 16, 16

        kernel_path = write_triton_kernels(tmp_dir, kernel_src, kernel_utils_src)
        compile_aot_kernels(tmp_dir, kernel_path, dtype, BM, BN, BK, ha_hb_hints=[(":1", ":1")])
        if is_hip():
            check_hasco_binary_str(tmp_dir, dtype)

        link_aot_kernels(tmp_dir)

        # compile test case
        M, N, K = 16, 16, 16
        gen_kernel_library(tmp_dir, "libkernel.so")
        gen_test_bin(tmp_dir, M, N, K)

        # initialize test data
        a, b, a_path, b_path, c_path = generate_matmul_test_data(tmp_dir, M, N, K)

        # run test case
        env = os.environ.copy()
        env["LD_LIBRARY_PATH"] = tmp_dir
        result = subprocess.run(
            ["./test", a_path, b_path, c_path],
            env=env,
            cwd=tmp_dir,
            capture_output=True,
            text=True,
        )

        # It should fail since the launcher requires all the strides be 1 while they are not.
        assert result.returncode == -6
        assert "kernel launch failed" in result.stderr
```
- **EN:** Invokes `tempfile.TemporaryDirectory`, `write_triton_kernels`, `compile_aot_kernels`, `is_hip`, `link_aot_kernels`, `gen_kernel_library`, and 5 more to execute the test logic. Validates behavior with 2 assertion(s). Uses context managers to control resources or expectations. Relevant themes: subprocess-driven validation, matrix multiplication workflows, kernel launch orchestration.
- **CN:** 调用 `tempfile.TemporaryDirectory`、`write_triton_kernels`、`compile_aot_kernels`、`is_hip`、`link_aot_kernels`、`gen_kernel_library` 等另外 5 项 执行测试逻辑。 通过 2 个断言验证行为。 使用上下文管理器控制资源或预期行为。 相关主题：基于子进程的验证、矩阵乘法工作流、kernel 启动编排。

### Lines 495-497

```python


def test_compile_link_autotune_matmul():
```
- **EN:** Defines the test function `test_compile_link_autotune_matmul`. Key calls include `np.random.seed`, `tempfile.TemporaryDirectory`, `write_triton_kernels`, `link_aot_kernels`, `gen_kernel_library`, `generate_matmul_test_data`, and 10 more. This scope touches subprocess-driven validation, autotuning logic, matrix multiplication workflows, random-data generation.
- **CN:** 定义测试函数 `test_compile_link_autotune_matmul`。 关键调用包括 `np.random.seed`、`tempfile.TemporaryDirectory`、`write_triton_kernels`、`link_aot_kernels`、`gen_kernel_library`、`generate_matmul_test_data` 等另外 10 项。 该作用域涉及基于子进程的验证、自动调优逻辑、矩阵乘法工作流、随机数据生成。

#### Lines 498-498

```python
    np.random.seed(3)
```
- **EN:** Invokes `np.random.seed` to execute the test logic. Relevant themes: random-data generation.
- **CN:** 调用 `np.random.seed` 执行测试逻辑。 相关主题：随机数据生成。

#### Lines 499-542

```python

    with tempfile.TemporaryDirectory() as tmp_dir:
        dtype = "fp16"

        kernel_path = write_triton_kernels(tmp_dir, kernel_src, kernel_utils_src)

        tile_sizes = [
            [16, 16, 16],
            [64, 64, 32],
        ]

        for ts in tile_sizes:
            BM, BN, BK = ts[0], ts[1], ts[2]
            compile_aot_kernels(tmp_dir, kernel_path, dtype, BM, BN, BK, ha_hb_hints=[(":16", ":16"), (":16", ""),
                                                                                      ("", ":16")])

        link_aot_kernels(tmp_dir)

        gen_kernel_library(tmp_dir, "libkernel.so")

        # compile test case
        M, N, K = 64, 64, 64
        # initialize test data
        a, b, a_path, b_path, c_path = generate_matmul_test_data(tmp_dir, M, N, K)
        c_ref = np.matmul(a.astype(np.float32), b.astype(np.float32))

        for algo_id in range(len(tile_sizes)):
            # generate and run test case
            test_name = f"test_{algo_id}"
            gen_test_bin(tmp_dir, M, N, K, exe=test_name, algo_id=algo_id)

            env = os.environ.copy()
            env["LD_LIBRARY_PATH"] = tmp_dir
            subprocess.run(
                [f"./{test_name}", a_path, b_path, c_path],
                check=True,
                cwd=tmp_dir,
                env=env,
            )

            # read data and compare against reference
            c = np.genfromtxt(c_path, delimiter=",", dtype=np.int32)
            c_tri = c.reshape((M, N)).view(np.float32)
            np.testing.assert_allclose(c_tri, c_ref * c_ref, atol=1e-4, rtol=1e-4)
```
- **EN:** Invokes `tempfile.TemporaryDirectory`, `write_triton_kernels`, `link_aot_kernels`, `gen_kernel_library`, `generate_matmul_test_data`, `np.matmul`, and 9 more to execute the test logic. Uses context managers to control resources or expectations. Relevant themes: subprocess-driven validation, matrix multiplication workflows.
- **CN:** 调用 `tempfile.TemporaryDirectory`、`write_triton_kernels`、`link_aot_kernels`、`gen_kernel_library`、`generate_matmul_test_data`、`np.matmul` 等另外 9 项 执行测试逻辑。 使用上下文管理器控制资源或预期行为。 相关主题：基于子进程的验证、矩阵乘法工作流。

### Lines 543-545

```python


def test_ttgir_to_asm():
```
- **EN:** Defines the test function `test_ttgir_to_asm`. Key calls include `is_hip`, `GPUTarget`, `tempfile.TemporaryDirectory`, `os.path.join`, `triton.compile`, `is_cuda`, and 3 more. This scope touches Triton compilation or JIT kernels.
- **CN:** 定义测试函数 `test_ttgir_to_asm`。 关键调用包括 `is_hip`、`GPUTarget`、`tempfile.TemporaryDirectory`、`os.path.join`、`triton.compile`、`is_cuda` 等另外 3 项。 该作用域涉及Triton 编译或 JIT kernel。

#### Lines 546-553

```python
    src = """
module attributes {{"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = {warp_size} : i32, "ttg.num-ctas" = 1 : i32}} {{
  tt.func public @sum_kernel_0d1d(%arg0: !tt.ptr<i32>, %arg1: !tt.ptr<i32>) {{
    tt.return
  }}
}}
"""
    target = GPUTarget("hip", "gfx942", 64) if is_hip() else GPUTarget("cuda", 80, 32)
```
- **EN:** Prepares or updates state through `src`, `target`. Invokes `is_hip`, `GPUTarget` to execute the test logic.
- **CN:** 通过 `src`、`target` 准备或更新状态。 调用 `is_hip`、`GPUTarget` 执行测试逻辑。

#### Lines 554-566

```python
    with tempfile.TemporaryDirectory() as tmp_dir:
        kernel_path = os.path.join(tmp_dir, "empty_kernel.ttgir")
        with open(kernel_path, "w") as fp:
            fp.write(src.format(warp_size=target.warp_size))
        k = triton.compile(kernel_path, target=target)
        if is_cuda():
            ptx = k.asm["ptx"]
            assert ".target sm_80" in ptx
            assert ".address_size 64" in ptx
        elif is_hip():
            amdgcn = k.asm["amdgcn"]
            assert '.amdgcn_target "amdgcn-amd-amdhsa--gfx942"' in amdgcn
            assert '.wavefront_size: 64' in amdgcn
```
- **EN:** Invokes `tempfile.TemporaryDirectory`, `os.path.join`, `triton.compile`, `is_cuda`, `open`, `fp.write`, and 2 more to execute the test logic. Validates behavior with 4 assertion(s). Uses context managers to control resources or expectations. Relevant themes: Triton compilation or JIT kernels.
- **CN:** 调用 `tempfile.TemporaryDirectory`、`os.path.join`、`triton.compile`、`is_cuda`、`open`、`fp.write` 等另外 2 项 执行测试逻辑。 通过 4 个断言验证行为。 使用上下文管理器控制资源或预期行为。 相关主题：Triton 编译或 JIT kernel。

### Lines 567-571

```python


@pytest.mark.parametrize("target", [GPUTarget("hip", "gfx942", 64), GPUTarget("hip", "gfx1250", 32)])
@pytest.mark.skipif(not is_hip(), reason="Requires HIP")
def test_gluon_kernel(target):
```
- **EN:** Defines the test function `test_gluon_kernel`. Decorators: `pytest.mark.parametrize('target', [GPUTarget('hip', 'gfx942', 64), GPUTarget('hip', 'gfx1250', 32)])`, `pytest.mark.skipif(not is_hip(), reason='Requires HIP')`. Parameters: `target`. Key calls include `pytest.mark.parametrize`, `pytest.mark.skipif`, `tempfile.TemporaryDirectory`, `get_gluon_kernel_src`, `write_triton_kernels`, `compile_aot_kernel_no_specialization`, and 3 more. This scope touches pytest parametrization.
- **CN:** 定义测试函数 `test_gluon_kernel`。 装饰器：`pytest.mark.parametrize('target', [GPUTarget('hip', 'gfx942', 64), GPUTarget('hip', 'gfx1250', 32)])`、`pytest.mark.skipif(not is_hip(), reason='Requires HIP')`。 参数：`target`。 关键调用包括 `pytest.mark.parametrize`、`pytest.mark.skipif`、`tempfile.TemporaryDirectory`、`get_gluon_kernel_src`、`write_triton_kernels`、`compile_aot_kernel_no_specialization` 等另外 3 项。 该作用域涉及pytest 参数化。

#### Lines 572-578

```python
    with tempfile.TemporaryDirectory() as tmp_dir:
        dtype = "fp16"
        BM, BN, BK = 16, 16, 16
        gluon_kernel_src = get_gluon_kernel_src(target.warp_size)
        kernel_path = write_triton_kernels(tmp_dir, gluon_kernel_src, kernel_utils_src)
        compile_aot_kernel_no_specialization(tmp_dir, kernel_path, dtype, BM, BN, BK, target=target)
        check_hasco_binary_str(tmp_dir, dtype)
```
- **EN:** Invokes `tempfile.TemporaryDirectory`, `get_gluon_kernel_src`, `write_triton_kernels`, `compile_aot_kernel_no_specialization`, `check_hasco_binary_str` to execute the test logic. Uses context managers to control resources or expectations.
- **CN:** 调用 `tempfile.TemporaryDirectory`、`get_gluon_kernel_src`、`write_triton_kernels`、`compile_aot_kernel_no_specialization`、`check_hasco_binary_str` 执行测试逻辑。 使用上下文管理器控制资源或预期行为。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `get_gluon_kernel_src`, `gen_kernel_library`, `gen_test_bin`, `write_triton_kernels`, `_compile_kernel`, `compile_aot_kernel_no_specialization`, `compile_aot_kernels`, `link_aot_kernels`
  **CN:** 顶层作用域，例如 `get_gluon_kernel_src`、`gen_kernel_library`、`gen_test_bin`、`write_triton_kernels`、`_compile_kernel`、`compile_aot_kernel_no_specialization`、`compile_aot_kernels`、`link_aot_kernels`
- **EN:** pytest parametrization
  **CN:** pytest 参数化
- **EN:** Triton compilation or JIT kernels
  **CN:** Triton 编译或 JIT kernel
- **EN:** Triton language operations
  **CN:** Triton language 操作
- **EN:** subprocess-driven validation
  **CN:** 基于子进程的验证
- **EN:** autotuning logic
  **CN:** 自动调优逻辑
- **EN:** matrix multiplication workflows
  **CN:** 矩阵乘法工作流
- **EN:** runtime driver interaction
  **CN:** 运行时驱动交互

## Dependencies / 依赖关系

- **EN:** External or absolute imports include `glob`, `os`, `pytest`, `re`, `subprocess`, `sys`, `tempfile`, `numpy`, `triton`, `triton.backends.compiler`, `triton._internal_testing`, `triton.backends.nvidia.driver`, and 1 more.
  **CN:** 外部或绝对导入包括 `glob`、`os`、`pytest`、`re`、`subprocess`、`sys`、`tempfile`、`numpy`、`triton`、`triton.backends.compiler`、`triton._internal_testing`、`triton.backends.nvidia.driver` 等另外 1 项。
- **EN:** Execution centers on top-level definitions such as `get_gluon_kernel_src`, `gen_kernel_library`, `gen_test_bin`, `write_triton_kernels`, `_compile_kernel`, `compile_aot_kernel_no_specialization`, `compile_aot_kernels`, `link_aot_kernels`, `generate_matmul_test_data`, `check_hasco_binary_str`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `get_gluon_kernel_src`、`gen_kernel_library`、`gen_test_bin`、`write_triton_kernels`、`_compile_kernel`、`compile_aot_kernel_no_specialization`、`compile_aot_kernels`、`link_aot_kernels`、`generate_matmul_test_data`、`check_hasco_binary_str`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
