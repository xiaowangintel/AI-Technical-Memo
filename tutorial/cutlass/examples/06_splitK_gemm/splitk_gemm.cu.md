# splitk_gemm.cu — Code Analysis / 代码分析

## Source / 源文件

`examples/06_splitK_gemm/splitk_gemm.cu`

## Purpose / 用途

**EN:** Demonstrates `GemmSplitKParallel` on Volta Tensor Cores, splitting the K dimension so multiple CTAs compute partial products and reduce them into one output.

**CN:** 演示 Volta Tensor Core 上的 `GemmSplitKParallel`，把 K 维切分给多个 CTA 计算部分乘积并归约到同一输出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-123 / 第 1-123 行

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
This example shows how to use split-k version of matrix multiplication using functions and data
structures provided by CUTLASS; which we run on a NVIDIA Volta GPU.

What is split-k?
Consider a problem size of M = 128, N = 128, K = 4096. In this case, if my thread-block tile size (a
tile can be viewed as a 2d matrix) is 128x128x4096, then we launch a singled a thread-block taking
up a single SM of 84 SMs present on V100. Hence the efficiency of computation is really low. So, how
to solve it? This is where split-k comes in. It is a way of partitioning K-dimension of matrix
multiplication and distribute across multiple SMs and get better efficiency than single SM. In the
above example, we can partition K-dimension with split-k factor of 16 i.e., thread-block tile size
will be 128x128x256 and will be launching on 16 SMs. Once each thread-block computes their partial
inner product (1/16th of output), they accumulate to single output matrix.

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

In this example, we split variable initialization into
1. Setting up data properties : describes how matrices are laid out in the memory and how the kernel
can view them (logical to physical mapping)
2. Setting up computation properties : describes how the above set matrices will be used to compute
output of matrix multiplication.

First, we setup the data types of matrices A, B, C and D along with alpha, beta as the equation for
GEMM is D = alpha * A * B + beta * C. In CUTLASS, the kernels first compute A * B and leaves the
rest of the computation to end of the kernel as alpha * X + beta * C is a simple element-wise
operation on X (A * B) and C. We call this as epilogue of kernel. Hence, we setup data types for
alpha and beta to be equal to ElementComputeEpilogue = float. As we want to MMA instructions on
Volta and they support only half-precision floating point (fp16 or half), we use data type for
elements in input matrix A and B as cutlass::half_t. Volta also supports accumulation of partial dot
product to fp32, which can store wider range of numbers, we use it as data type of output matrix
elements and accumulation. We convey this to CUTLASS kernel by initializing template variables
ElementAccumulator (float), ElementComputeEpilogue (float), ElementInputA (cutlass::half_t),
ElementInputB (cutlass::half_t), ElementOutput (float). Communicating just the data type is not
enough. As the data is laid out linearly in memory, we have to convey the layout of matrices. We do
that by initializing template variable LayoutInputA to column major cutlass variable, LayoutInputB
to row major and LayoutOutput to row major. Next, we setup rules to compute alpha * X + beta * C
which is called epilogue of the kernel. We initialize template variable EpilogueOp, which takes the
data type of output ElementOutput (float), the number of elements per vector memory access (16),
data type of accumulator (float) and data type of computation of linear combination (alpha * X +
beta * C).

Now that we setup the properties of data, we have to setup properties of computation.

Second, we create template variables of tile sizes for thread-block, warp and mma-op to 128x128x32,
64x64x4, 8x8x4 (MxNxK) respectively. When passed to instantiate CUTLASS GEMM kernel, it internally
deduce the amount of threads needed per thread-block, amount of shared memory, storing data in
bank-conflict free manner, and ton of other variables required to compose, initialize and launch a
high performance GEMM kernel. This is the beauty of CUTLASS, it relieves developer from
understanding and coding complicated hardware optimizations which can easily go wrong.

There are few more template variables initialized such as, which threadblock tile of output matrix
is done which threadblock launched on an SM, CUDA SM architecture of GPU you want to run on.

These are all put together to create a template variable which describes CUTLASS GEMM kernel using
cutlass::gemm::device::GemmSplitKParallel template.

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
#include "cutlass/gemm/device/gemm_splitk_parallel.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/tensor_view_io.h"
#include "helper.h"

```

**EN:** The file begins with license text, a long tutorial comment explaining split-K motivation, CUTLASS hierarchy, data/computation setup, epilogue, and includes for GEMM, tensors, reference kernels, and helpers.

**CN:** 文件以许可证和长教程注释开始，说明 split-K 动机、CUTLASS 层级、数据/计算配置、尾声，并包含 GEMM、张量、参考内核和辅助头。

### Lines 124-142 / 第 124-142 行

```cpp
// The code section below describes datatype for input, output matrices and computation between
// elements in input matrices.
using ElementAccumulator = float;                   // <- data type of accumulator
using ElementComputeEpilogue = ElementAccumulator;  // <- data type of epilogue operations
using ElementInputA = cutlass::half_t;              // <- data type of elements in input matrix A
using ElementInputB = cutlass::half_t;              // <- data type of elements in input matrix B
using ElementOutput = float;                        // <- data type of elements in output matrix D

// The code section below describes matrix layout of input and output matrices. Column Major for
// Matrix A, Row Major for Matrix B and Row Major for Matrix C
using LayoutInputA = cutlass::layout::ColumnMajor;
using LayoutInputB = cutlass::layout::RowMajor;
using LayoutOutput = cutlass::layout::RowMajor;

// This code section describes whether you want to use tensor cores or regular SIMT cores on GPU SM
using MMAOp = cutlass::arch::OpClassTensorOp;

// This code section describes CUDA SM architecture number
using SmArch = cutlass::arch::Sm70;
```

**EN:** Type aliases define FP32 accumulation/epilogue, FP16 inputs, FP32 output, column-major A, row-major B/output, Tensor Core operator class, and Volta `Sm70` architecture tag.

**CN:** 类型别名定义 FP32 累加/尾声、FP16 输入、FP32 输出、列主序 A、行主序 B/输出、Tensor Core 算子类别以及 Volta `Sm70` 架构标签。

### Lines 144-176 / 第 144-176 行

```cpp
// This code section describes the tile size a thread block will compute
using ShapeMMAThreadBlock =
    cutlass::gemm::GemmShape<128, 128, 32>;  // <- threadblock tile M = 128, N = 128, K = 32
// This code section describes tile size a warp will compute
using ShapeMMAWarp = cutlass::gemm::GemmShape<64, 64, 32>;  // <- warp tile M = 64, N = 64, K = 32
// This code section describes the size of MMA op
using ShapeMMAOp = cutlass::gemm::GemmShape<8, 8, 4>;  // <- MMA Op tile M = 8, N = 8, K = 4

// This code section describes ?
using EpilogueOp = cutlass::epilogue::thread::LinearCombination<
    ElementOutput,                                     // <- data type of output matrix
    128 / cutlass::sizeof_bits<ElementOutput>::value,  // <- This is the number of elements per
                                                       // vectorized memory access. For half
                                                       // precision, it's 8 elements. This becomes
                                                       // the vector width of math instructions in
                                                       // epilogue too
    ElementAccumulator,                                // <- data type of accumulator
    ElementComputeEpilogue>;  // <- data type for alpha/beta in linear combination function

// Put all the created template variables to create GemmSplitKParallel template variable
using Gemm = cutlass::gemm::device::GemmSplitKParallel<ElementInputA,
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
                                                       EpilogueOp>;
```

**EN:** The GEMM hierarchy uses a `128x128x32` threadblock, `64x64x32` warp, and `8x8x4` MMA instruction. `LinearCombination` is the epilogue, and all aliases instantiate `GemmSplitKParallel`.

**CN:** GEMM 层级使用 `128x128x32` threadblock、`64x64x32` warp 和 `8x8x4` MMA 指令。`LinearCombination` 是尾声，所有别名共同实例化 `GemmSplitKParallel`。

### Lines 178-220 / 第 178-220 行

```cpp
int run() {

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;
    return -1;
  }

  if (props.major != 7) {
    std::cerr << "Volta Tensor Ops must be run on a machine with compute capability of 70, 72, or 75."
              << std::endl;

    // Return 0 so tests pass if run on unsupported architectures or CUDA Toolkits.
    return 0;
  }

  //
  // Define problem size
  //

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

**EN:** `run` queries device properties, skips unsupported non-Volta GPUs, defines a large `5120x4096x4096` GEMM, and allocates CUTLASS `HostTensor` objects for A, B, C, D, and reference D.

**CN:** `run` 查询设备属性，跳过非 Volta GPU，定义大的 `5120x4096x4096` GEMM，并为 A、B、C、D 和参考 D 分配 CUTLASS `HostTensor`。

### Lines 221-267 / 第 221-267 行

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

  // Split K dimension into 16 partitions
  int split_k_slices = 16;

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

**EN:** The tensors are randomly filled or zeroed, synchronized to the device, and scalar `alpha=1`, `beta=0` are set. `split_k_slices=16` is encoded in `Gemm::Arguments` together with tensor device references.

**CN:** 张量被随机填充或清零并同步到设备；设置 `alpha=1`、`beta=0`。`split_k_slices=16` 连同张量设备引用一起写入 `Gemm::Arguments`。

### Lines 269-284 / 第 269-284 行

```cpp
  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = Gemm::get_workspace_size(arguments);

  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm_op;

  // Initialize CUTLASS kernel with arguments and workspace pointer
  cutlass::Status status = gemm_op.initialize(arguments, workspace.get());
  CUTLASS_CHECK(status);

  // Launch initialized CUTLASS kernel
  status = gemm_op();
  CUTLASS_CHECK(status);
```

**EN:** The kernel reports required workspace for split-K reduction. Workspace is allocated, the operation is initialized, and the CUTLASS kernel is launched through `operator()`.

**CN:** 内核报告 split-K 归约所需工作空间。代码分配工作空间、初始化操作，并通过 `operator()` 启动 CUTLASS 内核。

### Lines 286-320 / 第 286-320 行

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

**EN:** A CUTLASS device reference GEMM computes the expected result. After synchronization and host copies, `TensorEquals` compares CUTLASS output against the reference and returns pass/fail.

**CN:** CUTLASS 设备端参考 GEMM 计算期望结果。同步并拷回主机后，`TensorEquals` 比较 CUTLASS 输出与参考输出并返回通过/失败。

### Lines 323-339 / 第 323-339 行

```cpp
int main() {

  //
  // Volta Tensor Core operations exposed with mma.sync are first available in CUDA 10.1.
  //
  // CUTLASS must be compiled with CUDA 10.1 Toolkit to run these examples.
  //
  if (!(__CUDACC_VER_MAJOR__ > 10 || (__CUDACC_VER_MAJOR__ == 10 && __CUDACC_VER_MINOR__ >= 1))) {
    std::cerr << "Volta Tensor Core operations must be compiled with CUDA 10.1 Toolkit or later." << std::endl;

    // Returning zero, so this test passes when built with older CUDA Toolkits. Its action are no-op.
    return 0;
  }
  else {
    return run();
  }
}
```

**EN:** `main` enforces CUDA 10.1+ because Volta `mma.sync` support starts there, returning success as a no-op on older toolkits; otherwise it calls `run`.

**CN:** `main` 要求 CUDA 10.1+，因为 Volta `mma.sync` 从该版本开始支持；旧工具链上以 no-op 形式返回成功，否则调用 `run`。

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

- **EN:** `GemmSplitKParallel` adds workspace and a split count because partial sums from multiple K slices must be accumulated.
  **CN:** `GemmSplitKParallel` 需要工作空间和切分数量，因为多个 K 切片产生的部分和必须被累加。

- **EN:** Volta half Tensor Cores use `cutlass::half_t` inputs, `float` accumulation, and an `8x8x4` instruction shape.
  **CN:** Volta half Tensor Core 使用 `cutlass::half_t` 输入、`float` 累加和 `8x8x4` 指令形状。

## Dependencies / 依赖项

- **EN:** CUTLASS core, GEMM device, `HostTensor`, tensor fill/copy/compare, device reference GEMM, and `helper.h` for error checking.
  **CN:** 依赖 CUTLASS 核心、GEMM device、`HostTensor`、张量填充/拷贝/比较、设备参考 GEMM，以及用于错误检查的 `helper.h`。

- **EN:** CUDA runtime and a GPU/toolkit matching the architecture checks.
  **CN:** 需要 CUDA 运行时，以及满足架构检查的 GPU/工具链。
