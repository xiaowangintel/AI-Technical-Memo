# turing_tensorop_gemm.cu — Code Analysis / 代码分析

## Source / 源文件

`examples/08_turing_tensorop_gemm/turing_tensorop_gemm.cu`

## Purpose / 用途

**EN:** Demonstrates a CUTLASS device GEMM using Turing int8 Tensor Cores with int32 accumulation and output.

**CN:** 演示使用 Turing int8 Tensor Core、int32 累加和 int32 输出的 CUTLASS device GEMM。

## Line-by-Line Analysis / 逐行分析

### Lines 1-133 / 第 1-133 行

```cpp
/***************************************************************************************************
 * Copyright (c) 2017 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/

/**
This example shows how to run matrix multiplication kernels using functions and data structures
provided by CUTLASS using tensor cores; which we run on a NVIDIA Turing GPU.

Writing a single high performance matrix multiplication kernel is hard but do-able. Whereas writing
high performance kernels at scale which works for multiple problem sizes with good abstractions is
really hard. CUTLASS solves this problem by providing simplified abstractions to compose
multiple sections of gemm kernel. When used properly, the kernels can hit peak performance of GPU
easily.

CUTLASS divides a kernel into hierarchical composable sections. Which means, at each thread, warp
and thread-block level, they compute on their own tile-size with higher level of tile sizes being
composed from lower level ones. Multiple thread-tiles (tile size each thread computes) can be used
to form warp-tiles (tile size each warp computes) and multiple warp tiles can be used to compute
threadblock-tile (tile size computed by a threadblock).

In thie example, we split variable initialization into
1. Setting up data properties : describes how matrices are laid out in the memory and how the kernel
can view them (logical to physical mapping)
2. Setting up computation properties : describes how the above set matrices will be used to compute
output of matrix multiplication.

First, we setup the data types of matrices A, B, C and D along with alpha, beta as the equation for
GEMM is D = alpha * A * B + beta * C. In CUTLASS, the kernels first compute A * B and leaves the
rest of the computation to end of the kernel as alpha * X + beta * C is a simple element-wise
operation on X (A * B) and C. We call this as epilogue of kernel. Hence, we setup data types for
alpha and beta to be equal to ElementComputeEpilogue = int32_t. As we want to use MMA instructions
on Turing and they support 8-bit signed integer (int8_t), we use data type for elements in input
matrix A and B as int8_t. Volta also supports accumulation of partial dot product to int32_t, which
can store wider range of numbers, we use it as data type of output matrix elements and accumulation.
We convey this to CUTLASS kernel by initializing template variables ElementAccumulator (int32_t),
ElementComputeEpilogue (int32_t), ElementInputA (int8_t), ElementInputB (int8_t), ElementOutput
(int32_t). Communicating just the data type is not enough. As the data is laid out linearly in
memory, we have to convey the layout of matrices. We do that by initializing template variable
LayoutInputA to column major cutlass variable, LayoutInputB to row major and LayoutOutput to row
major. Next, we setup rules to compute alpha * X + beta * C which is called epilogue of the kernel.
We initialize template variable EpilogueOp, which takes the data type of output ElementOutput
(int32_t), the number of elements per vector memory access (16), data type of accumulator (int32_t)
and data type of computation of linear combination (alpha * X + beta * C).

Now that we setup the properties of data, we have to setup properties of computation.

Second, we create template variables of tile sizes for thread-block, warp and mma-op to 128x256x64,
64x64x16, 8x8x16 (MxNxK) respectively. When passed to instantiate CUTLASS GEMM kernel, it internally
deduce the amount of threads needed per thread-block, amount of shared memory, storing data in
bank-conflict free manner, and ton of other variables required to compose, initialize and launch a
high performance GEMM kernel. This is the beauty of CUTLASS, it relieves developer from
understanding and coding complicated hardware optimizations which can easily go wrong.

CUTLASS also supports multiple MMA pipelines in a threadblock. What are MMA pipelines? MMA pipelines
constitute the whole process of loading input data from global memory to shared memory, loading data
from shared memory to registers, doing matrix multiplication, store to global memory. The below flow
sequence shows a typical mma pipeline.

matrix in global memory -> registers -> tile in shared memory -> registers -> mma -> registers ->
output to global memory

The problem with single pipeline is, each stage is synchronous which means, each stage has to wait
until the previous finished executing. There are stages in the pipeline which do not have fixed
latency, for example, the loads from global memory and shared memory. Therefore, we can add one more
pipeline with a phase shift in mma kernel to hide latency from global and shared memory loads.
Finally, the pipeline in a kernel looks like

(1) matrix in global memory -> (2) registers -> (3) tile in shared memory -> (4) registers -> (5)
mma -> (6) registers -> (7) output to global memory (1) <null> -> (2) <null> -> (3) matrix in global
memory -> (4) registers -> (5) tile in shared memory -> (6) registers -> (7) mma -> (8) registers ->
(9) output to global memory

This way, you can hide the second global memoroy load latency by doing computation on already loaded
input data.

There are few more template variables initialized such as, which threadblock tile of output matrix
is done which threadblock launched on an SM, CUDA SM architecture of GPU you want to run on.

These are all put together to create a template variable which describes CUTLASS GEMM kernel using
cutlass::gemm::device::Gemm template.

The next step is to initialize physical data, instantiate and initialize CUTLASS kernel and run it.
We use CUTLASS utilities to initialize, fill, compare matrices as they are simple and doesn't come
in the way of learning CUTLASS.

Once all the matrices are initialized and filled with data, create arguments tuple to launch CUTLASS
kernel which takes problem size (M = 5120, N = 4096 and K = 4096), matrices, alpha, beta and the
important one, split k-dimension factor. Along with that, we query CUTLASS if any scratch-space
memory required by the kernel we instantiated. If yes, we create it and pass it along with other
arguments created to initialize CUTLASS kernel then, the kernel is launched.

In this example, we later on launch a reference gemm kernel (from CUTLASS utilities) to compare if
the output from CUTLASS kernel is same as reference GEMM kernel.
*/

#include <iostream>

#include "cutlass/cutlass.h"
#include "cutlass/gemm/device/gemm.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/tensor_view_io.h"
#include "helper.h"
```

**EN:** License, tutorial documentation, and includes introduce Turing Tensor Core int8 GEMM, CUTLASS tiling hierarchy, two-stage pipelining, and helper utilities.

**CN:** 许可证、教程文档和 include 介绍 Turing Tensor Core int8 GEMM、CUTLASS tile 层级、两级流水线和辅助工具。

### Lines 135-153 / 第 135-153 行

```cpp
// The code section below describes datatype for input, output matrices and computation between
// elements in input matrices.
using ElementAccumulator = int32_t;                 // <- data type of accumulator
using ElementComputeEpilogue = ElementAccumulator;  // <- data type of epilogue operations
using ElementInputA = int8_t;                       // <- data type of elements in input matrix A
using ElementInputB = int8_t;                       // <- data type of elements in input matrix B
using ElementOutput = int32_t;                      // <- data type of elements in output matrix D

// The code section below describes matrix layout of input and output matrices. Row Major for
// Matrix A, Column Major for Matrix B and Row Major for Matrix C
using LayoutInputA = cutlass::layout::RowMajor;
using LayoutInputB = cutlass::layout::ColumnMajor;
using LayoutOutput = cutlass::layout::RowMajor;

// This code section describes whether you want to use tensor cores or regular SIMT cores on GPU SM
using MMAOp = cutlass::arch::OpClassTensorOp;

// This code section describes CUDA SM architecture number
using SmArch = cutlass::arch::Sm75;
```

**EN:** Aliases select `int8_t` inputs, `int32_t` accumulator/output/epilogue, row-major A, column-major B, row-major output, Tensor Core operator class, and Turing `Sm75` architecture.

**CN:** 别名选择 `int8_t` 输入、`int32_t` 累加器/输出/尾声、行主序 A、列主序 B、行主序输出、Tensor Core 算子类别和 Turing `Sm75` 架构。

### Lines 155-193 / 第 155-193 行

```cpp
// This code section describes the tile size a thread block will compute
using ShapeMMAThreadBlock =
    cutlass::gemm::GemmShape<128, 256, 64>;  // <- threadblock tile M = 128, N = 256, K = 64
// This code section describes tile size a warp will compute
using ShapeMMAWarp = cutlass::gemm::GemmShape<64, 64, 64>;  // <- warp tile M = 64, N = 64, K = 64 
// This code section describes the size of MMA op
using ShapeMMAOp = cutlass::gemm::GemmShape<8, 8, 16>;  // <- MMA Op tile M = 8, N = 8, K = 16

// This code section describes how threadblocks are scheduled on GPU
using SwizzleThreadBlock = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>;

// This code section describes the epilogue part of the kernel
using EpilogueOp = cutlass::epilogue::thread::LinearCombination<
    ElementOutput,                                     // <- data type of output matrix
    128 / cutlass::sizeof_bits<ElementOutput>::value,  // <- the number of elements per vectorized
                                                       // memory access. For a byte, it's 16
                                                       // elements. This becomes the vector width of
                                                       // math instructions in the epilogue too
    ElementAccumulator,                                // <- data type of accumulator
    ElementComputeEpilogue>;  // <- data type for alpha/beta in linear combination function

// Number of pipelines you want to use
constexpr int NumStages = 2;

using Gemm = cutlass::gemm::device::Gemm<ElementInputA,
                                         LayoutInputA,
                                         ElementInputB,
                                         LayoutInputB,
                                         ElementOutput,
                                         LayoutOutput,
                                         ElementAccumulator,
                                         MMAOp,
                                         SmArch,
                                         ShapeMMAThreadBlock,
                                         ShapeMMAWarp,
                                         ShapeMMAOp,
                                         EpilogueOp,
                                         SwizzleThreadBlock,
                                         NumStages>;
```

**EN:** The template shapes are `128x256x64` threadblock, `64x64x64` warp, and `8x8x16` MMA instruction. `LinearCombination` writes vectorized int32 outputs, identity swizzle schedules CTAs, and two pipeline stages are requested.

**CN:** 模板形状为 `128x256x64` threadblock、`64x64x64` warp 和 `8x8x16` MMA 指令。`LinearCombination` 向量化写出 int32，identity swizzle 调度 CTA，并请求两级流水。

### Lines 195-217 / 第 195-217 行

```cpp
int run() {

  const int length_m = 5120;
  const int length_n = 4096;
  const int length_k = 4096;

  // Create a tuple of problem size for matrix multiplication
  cutlass::gemm::GemmCoord problem_size(length_m, length_n, length_k);

  // Initialize tensors using CUTLASS helper functions
  cutlass::HostTensor<ElementInputA, LayoutInputA> tensor_a(
      problem_size.mk());  // <- Create matrix A with dimensions M x K
  cutlass::HostTensor<ElementInputB, LayoutInputB> tensor_b(
      problem_size.kn());  // <- Create matrix B with dimensions K x N
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_c(
      problem_size.mn());  // <- Create matrix C with dimensions M x N
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_d(
      problem_size.mn());  // <- Create matrix D with dimensions M x N used to store output from
                           // CUTLASS kernel
  cutlass::HostTensor<ElementOutput, LayoutOutput> tensor_ref_d(
      problem_size.mn());  // <- Create matrix D with dimensions M x N used to store output from
                           // reference kernel

```

**EN:** `run` defines a `5120x4096x4096` GEMM and allocates CUTLASS `HostTensor` storage for A, B, C, D, and reference D using the chosen layouts.

**CN:** `run` 定义 `5120x4096x4096` GEMM，并按照所选布局为 A、B、C、D 和参考 D 分配 CUTLASS `HostTensor`。

### Lines 218-264 / 第 218-264 行

```cpp
  // Fill input and output matrices on host using CUTLASS helper functions
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_a.host_view(),
      1,
      ElementInputA(4),
      ElementInputA(-4),
      0);  // <- Fill matrix A on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_b.host_view(),
      1,
      ElementInputB(4),
      ElementInputB(-4),
      0);  // <- Fill matrix B on host with uniform-distribution random data
  cutlass::reference::host::TensorFillRandomUniform(
      tensor_c.host_view(),
      1,
      ElementOutput(4),
      ElementOutput(-4),
      0);  // <- Fill matrix C on host with uniform-distribution random data
  cutlass::reference::host::TensorFill(
      tensor_d.host_view());  // <- fill matrix D on host with zeros
  cutlass::reference::host::TensorFill(
      tensor_ref_d.host_view());  // <- fill matrix D for reference on host with zeros

  // Copy data from host to GPU
  tensor_a.sync_device();
  tensor_b.sync_device();
  tensor_c.sync_device();
  tensor_d.sync_device();
  tensor_ref_d.sync_device();

  // Initialize alpha and beta for dot product computation
  ElementComputeEpilogue alpha = ElementComputeEpilogue(1);
  ElementComputeEpilogue beta = ElementComputeEpilogue(0);

  // Split K dimension into 1 partitions
  int split_k_slices = 1;

  // Create a tuple of gemm kernel arguments. This is later passed as arguments to launch
  // instantiated CUTLASS kernel
  typename Gemm::Arguments arguments{problem_size,  // <- problem size of matrix multiplication
                                     tensor_a.device_ref(),  // <- reference to matrix A on device
                                     tensor_b.device_ref(),  // <- reference to matrix B on device
                                     tensor_c.device_ref(),  // <- reference to matrix C on device
                                     tensor_d.device_ref(),  // <- reference to matrix D on device
                                     {alpha, beta},          // <- tuple of alpha and beta
                                     split_k_slices};        // <- k-dimension split factor
```

**EN:** The tensors are initialized with small random integers or zeros, synchronized to the GPU, and `Gemm::Arguments` is built with `alpha=1`, `beta=0`, tensor references, and no split-K (`1` slice).

**CN:** 张量使用小随机整数或零初始化并同步到 GPU；随后以 `alpha=1`、`beta=0`、张量引用和无 split-K（`1` 个切片）构造 `Gemm::Arguments`。

### Lines 266-285 / 第 266-285 行

```cpp
  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = Gemm::get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm_op;

  // Check the problem size is supported or not 
  cutlass::Status status = gemm_op.can_implement(arguments);
  CUTLASS_CHECK(status);

  // Initialize CUTLASS kernel with arguments and workspace pointer
  status = gemm_op.initialize(arguments, workspace.get());
  CUTLASS_CHECK(status);

  // Launch initialized CUTLASS kernel
  status = gemm_op();
  CUTLASS_CHECK(status);
```

**EN:** Workspace is queried and allocated. `can_implement` catches unsupported alignments/shapes, then `initialize` binds arguments and workspace and `operator()` launches the GEMM.

**CN:** 查询并分配工作空间。`can_implement` 捕获不支持的对齐/形状，然后 `initialize` 绑定参数和工作空间，`operator()` 启动 GEMM。

### Lines 287-321 / 第 287-321 行

```cpp
  // Create instantiation for device reference gemm kernel
  cutlass::reference::device::Gemm<ElementInputA,
                                   LayoutInputA,
                                   ElementInputB,
                                   LayoutInputB,
                                   ElementOutput,
                                   LayoutOutput,
                                   ElementComputeEpilogue,
                                   ElementComputeEpilogue>
      gemm_device;

  // Launch device reference gemm kernel
  gemm_device(problem_size,
              alpha,
              tensor_a.device_ref(),
              tensor_b.device_ref(),
              beta,
              tensor_c.device_ref(),
              tensor_ref_d.device_ref());

  // Wait for kernels to finish
  cudaDeviceSynchronize();

  // Copy output data from CUTLASS and reference kernel to host for comparison
  tensor_d.sync_host();
  tensor_ref_d.sync_host();

  // Check if output from CUTLASS kernel and reference kernel are equal or not
  bool passed = cutlass::reference::host::TensorEquals(
    tensor_d.host_view(),
    tensor_ref_d.host_view());

  std::cout << (passed ? "Passed" : "Failed") << std::endl;

  return (passed ? 0  : -1);
```

**EN:** The reference device GEMM uses the same types/layouts to compute expected D. Synchronization, host copies, and `TensorEquals` determine pass/fail.

**CN:** 参考设备 GEMM 使用相同类型/布局计算期望 D。同步、主机拷贝和 `TensorEquals` 决定通过/失败。

### Lines 324-357 / 第 324-357 行

```cpp
int main() {
  bool notSupported = false;

  // Turing Tensor Core operations exposed with mma.sync and ldmatrix are first available
  // in CUDA 10.2. 
  //
  // CUTLASS must be compiled with CUDA 10.2 Toolkit to run these examples.
  if (!(__CUDACC_VER_MAJOR__ > 10 || (__CUDACC_VER_MAJOR__ == 10 && __CUDACC_VER_MINOR__ >= 2))) {
    std::cerr << "Turing Tensor Core operations must be compiled with CUDA 10.2 Toolkit or later." << std::endl;
    notSupported = true;
  }

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }

  if (!((props.major * 10 + props.minor) >= 75)) {
    std::cerr << "Turing Tensor Core operations must be run on a machine with compute capability at least 75."
              << std::endl;

    notSupported = true;
  }

  if (notSupported) {
    // Returning zero so this test passes on older Toolkits. Its actions are no-op.
    return 0;
  }

  return run();
}
```

**EN:** `main` requires CUDA 10.2+ and compute capability at least 7.5 for Turing `mma.sync`/`ldmatrix`; unsupported environments return success without running the sample.

**CN:** `main` 要求 CUDA 10.2+ 且计算能力至少 7.5，以支持 Turing `mma.sync`/`ldmatrix`；不支持的环境不运行示例并返回成功。

## Key Concepts / 关键概念

- **EN:** `cutlass::gemm::device::Gemm`-style templates compose data types, layouts, accumulator type, operator class, architecture, tile shapes, epilogue, swizzle, and pipeline stages into one device GEMM.
  **CN:** `cutlass::gemm::device::Gemm` 类模板把数据类型、布局、累加器类型、算子类别、架构、tile 形状、尾声、调度 swizzle 和流水级数合成为一个设备 GEMM。

- **EN:** Architecture tags such as `Sm70` and `Sm75` select kernels and instructions legal for Volta or Turing Tensor Cores.
  **CN:** `Sm70`、`Sm75` 等架构标签选择在 Volta 或 Turing Tensor Core 上合法的内核与指令。

- **EN:** Threadblock, warp, and instruction shapes form a hierarchy: CTA tiles are decomposed into warp tiles, which are decomposed into hardware MMA instruction tiles.
  **CN:** Threadblock、warp、instruction 形状构成层级：CTA tile 分解为 warp tile，warp tile 再分解为硬件 MMA 指令 tile。

- **EN:** The epilogue applies `D = alpha * accumulator + beta * C` and controls vectorized output width.
  **CN:** 尾声执行 `D = alpha * accumulator + beta * C`，并控制向量化输出宽度。

- **EN:** Layouts map logical matrix coordinates to memory. Column-major A and row-major B are common for Tensor Core-friendly access patterns.
  **CN:** 布局把逻辑矩阵坐标映射到内存。列主序 A 与行主序 B 是适合 Tensor Core 访问模式的常见组合。

- **EN:** Turing `Sm75` supports integer Tensor Core MMA; this sample uses `int8_t` A/B and `int32_t` accumulation.
  **CN:** Turing `Sm75` 支持整数 Tensor Core MMA；本示例使用 `int8_t` A/B 和 `int32_t` 累加。

- **EN:** The `8x8x16` instruction shape reflects the wider K dimension of int8 Tensor Core instructions.
  **CN:** `8x8x16` 指令形状反映 int8 Tensor Core 指令更宽的 K 维。

## Dependencies / 依赖项

- **EN:** CUTLASS core, GEMM device, `HostTensor`, tensor fill/copy/compare, device reference GEMM, and `helper.h` for error checking.
  **CN:** 依赖 CUTLASS 核心、GEMM device、`HostTensor`、张量填充/拷贝/比较、设备参考 GEMM，以及用于错误检查的 `helper.h`。

- **EN:** CUDA runtime and a GPU/toolkit matching the architecture checks.
  **CN:** 需要 CUDA 运行时，以及满足架构检查的 GPU/工具链。
