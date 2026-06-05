# examples/31_basic_syrk/basic_syrk.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/31_basic_syrk/basic_syrk.cu`
**Purpose / 用途**: Shows how to instantiate and validate a CUTLASS double-precision SYRK kernel using column-major tensors on SM80. / 展示如何在 SM80 上为列主序张量实例化并验证一个 CUTLASS 双精度 SYRK 内核。
---
## Line-by-Line Analysis / 逐行分析
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

/*
  This example demonstrates how to call a CUTLASS SYRK kernel and provides a naive reference
  matrix multiply kernel to verify its correctness.

  The CUTLASS Syrk template is instantiated in the function CutlassSsyrkNN. This is kernel computes
  the symmetric rank-k update (SYRK) using double-precision floating-point arithmetic and assumes
  all matrices have column-major layout.

  The threadblock tile size is chosen as 16x32x16 which offers good performance for large matrices.
  See the CUTLASS Parallel for All blog post for more exposition on the tunable parameters available
  in CUTLASS.

  https://devblogs.nvidia.com/cutlass-linear-algebra-cuda/

  Aside from defining and launching the SSYRK kernel, this example does not use any other components
  or utilities within CUTLASS. Such utilities are demonstrated elsewhere in other examples and are
  prevalent in the CUTLASS unit tests.

*/
```
**EN**: The standard license header is followed by an overview of the sample. It states that the example builds a SYRK kernel, uses double precision, assumes column-major matrices, and compares the CUTLASS result against a naïve CUDA implementation. The introductory text also points out that this file intentionally focuses on the core device API rather than the broader CUTLASS utility ecosystem.
**CN**: 标准许可证之后是示例总览。它说明该样例会构建一个 SYRK 内核，使用双精度，假定矩阵采用列主序，并把 CUTLASS 结果与朴素 CUDA 实现进行比较。开头说明还特别指出，这个文件有意聚焦于核心设备 API，而不是展示更广泛的 CUTLASS 工具生态。

```cpp

// Standard Library includes
#include <iostream>
#include <sstream>
#include <vector>

// Helper methods to check for errors
#include "helper.h"

//
// CUTLASS includes needed for double-precision SYRK kernel
//

// Defines cutlass::gemm::device::Syrk, the generic Syrk computation template class.
#include "cutlass/gemm/device/rank_k.h"
```
**EN**: This include block brings in standard containers and streams, a local helper for CUDA error handling, and `cutlass/gemm/device/rank_k.h`, which exposes the device-level `RankK` template used to implement SYRK. That header is the key CUTLASS dependency in this example.
**CN**: 这一组头文件引入标准容器与流、本地 CUDA 错误处理辅助工具，以及 `cutlass/gemm/device/rank_k.h`；它提供了实现 SYRK 所需的设备级 `RankK` 模板，是本示例最关键的 CUTLASS 依赖。

```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////
//
// This function defines a CUTLASS SYRK kernel instantiation, constructs its parameters object,
// and launches it on the CUDA device.
//
///////////////////////////////////////////////////////////////////////////////////////////////////

/// Define a CUTLASS SYRK template and launch a SYRK kernel.
cudaError_t CutlassSsyrkNN(
  int N,
  int K,
  double alpha,
  double const *A,
  int lda,
  double beta,
  double *C,
  int ldc) {
```
**EN**: These comments and the `CutlassSsyrkNN` signature define a BLAS-like host wrapper around the CUTLASS kernel. The arguments describe an `N x N` symmetric output updated from an `N x K` input matrix `A` using the usual `alpha` and `beta` scalars and leading dimensions `lda`/`ldc`.
**CN**: 这些注释和 `CutlassSsyrkNN` 的函数签名定义了一个围绕 CUTLASS 内核的 BLAS 风格主机包装器。参数描述了一个 `N x N` 的对称输出矩阵，它由 `N x K` 的输入矩阵 `A` 结合常见的 `alpha`、`beta` 标量以及前导维 `lda`/`ldc` 来更新。

```cpp

  // Define type definition for double-precision CUTLASS SYRK with column-major
  // input matrices and 16x32x16 threadblock tile size (chosen by default).
  //
  // To keep the interface manageable, several helpers are defined for plausible compositions
  // including the following example for double-precision SYRK. Typical values are used as
  // default template arguments.
  //
  // To view the full syrk device API interface, see `cutlass/gemm/device/syrk.h`

  using ColumnMajor = cutlass::layout::ColumnMajor;

  using CutlassSyrk = cutlass::gemm::device::RankK<
    double,
    ColumnMajor,
    double,
    ColumnMajor,
    cutlass::FillMode::kLower,
    double,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    cutlass::gemm::GemmShape<16, 32, 16>,
    cutlass::gemm::GemmShape<16, 16, 16>,
    cutlass::gemm::GemmShape<8, 8, 4>,
    cutlass::epilogue::thread::LinearCombination<
      double,
      1,
      double,
      double
    >,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<8>,
    5,     // Stages
    1,     // AlignmentA
    false, // SplitKSerail
    cutlass::arch::OpMultiplyAdd,
    cutlass::ComplexTransform::kNone,
    cutlass::BlasMode::kSymmetric
  >;
```
**EN**: This is the core CUTLASS instantiation. `RankK<double, ColumnMajor, double, ColumnMajor, cutlass::FillMode::kLower, double, ...>` says: both operand and output elements are FP64, matrices are column-major, only the lower triangular part is considered valid, and accumulation/output are also FP64. `OpClassTensorOp` with `Sm80` selects Ampere Tensor Core execution for double precision, while `GemmShape<16,32,16>`, `GemmShape<16,16,16>`, and `GemmShape<8,8,4>` define the threadblock, warp, and instruction tile hierarchy. The epilogue is `LinearCombination<double,1,double,double>`, meaning each output element is finalized as a scalar `alpha * accum + beta * C` with vector width 1. `GemmIdentityThreadblockSwizzle<8>` controls CTA rasterization, `5` is the pipeline stage count, `1` is the A alignment, `false` disables split-K serialization, `OpMultiplyAdd` chooses standard real multiply-add math, `ComplexTransform::kNone` leaves data untransformed, and `BlasMode::kSymmetric` tells the kernel to honor SYRK symmetry semantics.
**CN**: 这里是核心的 CUTLASS 实例化。`RankK<double, ColumnMajor, double, ColumnMajor, cutlass::FillMode::kLower, double, ...>` 的含义是：操作数和输出元素都为 FP64，矩阵采用列主序，只认为下三角部分有效，累加与输出也都是 FP64。`OpClassTensorOp` 配合 `Sm80` 选择 Ampere 上的双精度 Tensor Core 执行；`GemmShape<16,32,16>`、`GemmShape<16,16,16>` 和 `GemmShape<8,8,4>` 分别定义 threadblock、warp 和指令级分块层次。尾处理使用 `LinearCombination<double,1,double,double>`，表示每个输出元素都按标量公式 `alpha * accum + beta * C` 完成收尾，向量宽度为 1。`GemmIdentityThreadblockSwizzle<8>` 控制 CTA 光栅化方式，`5` 是流水线 stage 数，`1` 是 A 的对齐要求，`false` 关闭 split-K 串行化，`OpMultiplyAdd` 选择标准实数乘加，`ComplexTransform::kNone` 不做复数变换，而 `BlasMode::kSymmetric` 则让内核遵循 SYRK 的对称矩阵语义。

```cpp

  // Define a CUTLASS SYRK type
  CutlassSyrk syrk_operator;

  // Construct the CUTLASS SYRK arguments object.
  //
  // One of CUTLASS's design patterns is to define syrk argument objects that are constructible
  // in host code and passed to kernels by value. These may include pointers, strides, scalars,
  // and other arguments needed by Syrk and its components.
  //
  // The benefits of this pattern are (1.) a structured, composable strategy for passing host-constructible
  // arguments to kernels and (2.) minimized initialization overhead on kernel entry.
  //
  CutlassSyrk::Arguments args(cutlass::gemm::GemmUniversalMode::kGemm,
                              {N, N, K}, // Syrk Problem dimensions
                              1, // batch_count,
                              {alpha, beta}, // Scalars used in the Epilogue
                              reinterpret_cast<void const *>(A),
                              const_cast<void *>(reinterpret_cast<void *>(C)),
                              reinterpret_cast<void *>(C), // destination matrix D (may be different memory than source C matrix)
                              (int64_t)N*K, // Batch strides
                              (int64_t)N*N,
                              (int64_t)N*N,
                              lda,
                              ldc,
                              ldc);

  //
  // Launch the CUTLASS SYRK kernel.
  //

  cutlass::Status status = syrk_operator(args);

  //
  // Return a cudaError_t if the CUTLASS SYRK operator returned an error code.
  //

  if (status != cutlass::Status::kSuccess) {
    return cudaErrorUnknown;
  }

  // Return success, if no errors were encountered.
  return cudaSuccess;
}
```
**EN**: Once the alias is defined, the code constructs the operator object and fills a host-side `Arguments` structure. This CUTLASS pattern keeps kernel launch state explicit: `GemmUniversalMode::kGemm` selects the standard non-batched execution path, `{N, N, K}` describes the logical problem size, and the pointer/stride fields describe A, C, and D. Here D is the same buffer as C, so the update is effectively in-place. After the call operator returns, the sample translates a non-success `cutlass::Status` into `cudaErrorUnknown` so the rest of the file can stay in CUDA-runtime style.
**CN**: 定义完类型别名后，代码构造操作对象，并填充一个主机端 `Arguments` 结构体。这体现了 CUTLASS 的典型模式：把内核启动状态显式整理出来。`GemmUniversalMode::kGemm` 选择标准的非批处理执行路径，`{N, N, K}` 描述逻辑问题规模，而指针和步幅字段则描述 A、C 与 D。这里 D 和 C 使用同一块缓冲区，因此更新本质上是原地进行的。调用操作符返回后，示例把非成功的 `cutlass::Status` 转换成 `cudaErrorUnknown`，从而让文件其余部分继续保持 CUDA Runtime 风格。

```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////
//
// The source code after this point in the file is generic CUDA using the CUDA Runtime API
// and simple CUDA kernels to initialize matrices and compute the general matrix product.
//
///////////////////////////////////////////////////////////////////////////////////////////////////

/// Kernel to initialize a matrix with small integers.
__global__ void InitializeMatrix_kernel(
  double *matrix,
  int ldm,
  int rows,
  int columns,
  int seed = 0) {

  int i = threadIdx.x + blockIdx.x * blockDim.x;
  int j = threadIdx.y + blockIdx.y * blockDim.y;

  if (i < rows && j < columns) {
    int offset = i + j * ldm;

    // Generate arbitrary elements.
    int const k = 16807;
    int const m = 16;
    double value = double(((offset + seed) * k % m) - m / 2);

    matrix[offset] = value;
  }
}

/// Simple function to initialize a matrix to arbitrary small integers.
cudaError_t InitializeMatrix(double *matrix, int ldm, int rows, int columns, int seed = 0) {

  dim3 block(16, 16);
  dim3 grid(
    (rows + block.x - 1) / block.x,
    (columns + block.y - 1) / block.y
  );

  InitializeMatrix_kernel<<< grid, block >>>(matrix, ldm, rows, columns, seed);

  return cudaGetLastError();
}
```
**EN**: After the CUTLASS-specific section, the file switches to ordinary CUDA helper code. `InitializeMatrix_kernel` fills a dense column-major matrix with small deterministic integers derived from the linear offset and a seed, and `InitializeMatrix()` computes a 16x16 launch shape and dispatches that kernel. The deterministic initialization makes later bitwise comparison practical.
**CN**: 在 CUTLASS 专用部分之后，文件切换回普通的 CUDA 辅助代码。`InitializeMatrix_kernel` 会根据线性偏移和种子生成确定性的小整数，填充一个稠密列主序矩阵；`InitializeMatrix()` 则计算 16x16 的启动配置并发射该内核。由于初始化是确定性的，后续做按位比较就变得可行。

```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////

/// Allocates device memory for a matrix then fills with arbitrary small integers.
cudaError_t AllocateMatrix(double **matrix, int ldm, int rows, int columns, int seed = 0) {
  cudaError_t result;

  size_t sizeof_matrix = sizeof(double) * ldm * columns;

  // Allocate device memory.
  result = cudaMalloc(reinterpret_cast<void **>(matrix), sizeof_matrix);

  if (result != cudaSuccess) {
    std::cerr << "Failed to allocate matrix: "
      << cudaGetErrorString(result) << std::endl;
    return result;
  }

  // Clear the allocation.
  result = cudaMemset(*matrix, 0, sizeof_matrix);

  if (result != cudaSuccess) {
    std::cerr << "Failed to clear matrix device memory: "
      << cudaGetErrorString(result) << std::endl;
    return result;
  }

  // Initialize matrix elements to arbitrary small integers.
  result = InitializeMatrix(*matrix, ldm, rows, columns, seed);

  if (result != cudaSuccess) {
    std::cerr << "Failed to initialize matrix: "
      << cudaGetErrorString(result) << std::endl;
    return result;
  }

  return result;
}
```
**EN**: `AllocateMatrix()` is a small utility that allocates device memory, clears it, and then initializes it with the helper kernel. Error messages are printed immediately after each CUDA API failure, so the sample can stop close to the source of any setup problem.
**CN**: `AllocateMatrix()` 是一个小型工具函数：先分配设备内存，再清零，最后调用初始化辅助内核填充值。每一步 CUDA API 失败后都会立刻打印错误信息，因此示例能够在最接近问题源头的位置停止。

```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////

/// Naive reference SYRK computation.
__global__ void ReferenceSyrk_kernel(
  int N,
  int K,
  double alpha,
  double const *A,
  int lda,
  double beta,
  double *C,
  int ldc) {

  int i = threadIdx.x + blockIdx.x * blockDim.x;
  int j = threadIdx.y + blockIdx.y * blockDim.y;

  if (i < N && j < N && i >= j ) { // Since C is in Lower Fill Mode
    double accumulator = 0;

    for (int k = 0; k < K; ++k) {
      accumulator += A[i + k * lda] * A[j + k * lda];
    }

    C[i + j * ldc] = alpha * accumulator + beta * C[i + j * ldc];
  }
}

/// Reference SYRK computation.
cudaError_t ReferenceSyrk(
  int N,
  int K,
  double alpha,
  double const *A,
  int lda,
  double beta,
  double *C,
  int ldc) {

  dim3 block(16, 16);
  dim3 grid(
    (N + block.x - 1) / block.x,
    (N + block.y - 1) / block.y
  );

  ReferenceSyrk_kernel<<< grid, block >>>(N, K, alpha, A, lda, beta, C, ldc);

  return cudaGetLastError();
}
```
**EN**: This block provides the reference SYRK implementation. The kernel visits only `i >= j`, which matches the `kLower` fill mode selected in the CUTLASS instantiation, and computes `C = alpha * A * A^T + beta * C` element by element. `ReferenceSyrk()` just builds a matching grid and launches it.
**CN**: 这一段提供参考 SYRK 实现。内核只处理 `i >= j` 的元素，这与 CUTLASS 实例化中选择的 `kLower` 填充模式一致；随后它逐元素计算 `C = alpha * A * A^T + beta * C`。`ReferenceSyrk()` 只负责构造对应网格并发射该内核。

```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////

/// Allocate several matrices in GPU device memory and call a double-precision
/// CUTLASS SYRK kernel.
cudaError_t TestCutlassSyrk(int N, int K, double alpha, double beta) {
  cudaError_t result;

  //
  // Define several matrices to be used as operands to SYRK kernels.
  //

  // Compute leading dimensions for each matrix.
  int lda = N;
  int ldc = N;

  // Compute size in bytes of the C matrix.
  size_t sizeof_C = sizeof(double) * ldc * N;

  // Define pointers to matrices in GPU device memory.
  double *A;
  double *C_cutlass;
  double *C_reference;

  //
  // Allocate matrices in GPU device memory with arbitrary seeds.
  //

  result = AllocateMatrix(&A, lda, N, K, 0);

  if (result !=  cudaSuccess) {
    return result;
  }

  result = AllocateMatrix(&C_cutlass, ldc, N, N, 101);

  if (result != cudaSuccess) {
    cudaFree(A);
    return result;
  }

  result = AllocateMatrix(&C_reference, ldc, N, N, 101);

  if (result != cudaSuccess) {
    cudaFree(A);
    cudaFree(C_cutlass);
    return result;
  }
```
**EN**: `TestCutlassSyrk()` starts by deriving leading dimensions, matrix sizes, and device pointers. It allocates one input matrix `A` and two copies of `C`: one for CUTLASS and one for the reference path. The two C buffers are initialized with the same seed so both paths start from identical state.
**CN**: `TestCutlassSyrk()` 首先推导前导维、矩阵字节数和设备指针。它分配一个输入矩阵 `A`，以及两份 `C`：一份给 CUTLASS，另一份给参考路径。两份 C 使用相同的种子初始化，因此二者从完全一致的初始状态开始。

```cpp

  result = cudaMemcpy(C_reference, C_cutlass, sizeof_C, cudaMemcpyDeviceToDevice);

  if (result != cudaSuccess) {
    std::cerr << "Failed to copy C_cutlass matrix to C_reference: "
      << cudaGetErrorString(result) << std::endl;

    cudaFree(C_reference);
    cudaFree(C_cutlass);
    cudaFree(A);

    return result;
  }

  //
  // Launch CUTLASS SYRK.
  //

  result = CutlassSsyrkNN(N, K, alpha, A, lda, beta, C_cutlass, ldc);

  if (result != cudaSuccess) {
    std::cerr << "CUTLASS SYRK kernel failed: "
      << cudaGetErrorString(result) << std::endl;

    cudaFree(C_reference);
    cudaFree(C_cutlass);
    cudaFree(A);

    return result;
  }
```
**EN**: Before running the kernels, the code copies `C_cutlass` into `C_reference` with a device-to-device memcpy so the starting values are guaranteed identical. It then calls `CutlassSsyrkNN()` and performs immediate cleanup on failure. This keeps the comparison fair and the control flow simple.
**CN**: 在运行内核之前，代码通过一次设备到设备的 memcpy 把 `C_cutlass` 复制到 `C_reference`，从而保证初值完全一致。随后它调用 `CutlassSsyrkNN()`，若失败则立刻清理资源并返回。这样既保证了比较公平，也让控制流保持简单。

```cpp

  //
  // Verify.
  //

  // Launch reference SYRK
  result = ReferenceSyrk(N, K, alpha, A, lda, beta, C_reference, ldc);

  if (result != cudaSuccess) {
    std::cerr << "Reference SYRK kernel failed: "
      << cudaGetErrorString(result) << std::endl;

    cudaFree(C_reference);
    cudaFree(C_cutlass);
    cudaFree(A);

    return result;
  }

  // Copy to host and verify equivalence.
  std::vector<double> host_cutlass(ldc * N, 0);
  std::vector<double> host_reference(ldc * N, 0);

  result = cudaMemcpy(host_cutlass.data(), C_cutlass, sizeof_C, cudaMemcpyDeviceToHost);

  if (result != cudaSuccess) {
    std::cerr << "Failed to copy CUTLASS SYRK results: "
      << cudaGetErrorString(result) << std::endl;

    cudaFree(C_reference);
    cudaFree(C_cutlass);
    cudaFree(A);

    return result;
  }

  result = cudaMemcpy(host_reference.data(), C_reference, sizeof_C, cudaMemcpyDeviceToHost);

  if (result != cudaSuccess) {
    std::cerr << "Failed to copy Reference SYRK results: "
      << cudaGetErrorString(result) << std::endl;

    cudaFree(C_reference);
    cudaFree(C_cutlass);
    cudaFree(A);

    return result;
  }

  //
  // Free device memory allocations.
  //

  cudaFree(C_reference);
  cudaFree(C_cutlass);
  cudaFree(A);

  //
  // Test for bit equivalence of results.
  //

  if (host_cutlass != host_reference) {
    std::cerr << "CUTLASS results incorrect." << std::endl;

    return cudaErrorUnknown;
  }

  return cudaSuccess;
```
**EN**: The verification path launches the reference kernel, copies both outputs back to host vectors, frees the device allocations, and finally compares the two host buffers for exact equality. Because the sample uses deterministic initialization and a matching reference formula, an exact vector comparison is enough here.
**CN**: 校验路径会发射参考内核，把两份输出都复制回主机向量，释放设备内存，最后对两个主机缓冲区做精确相等比较。由于示例使用确定性初始化，并且参考公式与目标公式一致，这里用精确的向量比较就足够了。

```cpp
}

///////////////////////////////////////////////////////////////////////////////////////////////////

/// Entry point to basic_syrk example.
//
// usage:
//
//   00_basic_syrk <N> <K> <alpha> <beta>
//
int main(int argc, const char *arg[]) {

  bool notSupported = false;

  // CUTLASS must be compiled with CUDA 11 Toolkit to run these examples.
  if (!(__CUDACC_VER_MAJOR__ >= 11)) {
    std::cerr << "NVIDIA  Ampere Tensor Core operations must be compiled with CUDA 11.0 Toolkit or later." << std::endl;
    notSupported = true;
  }

  cudaDeviceProp props;

  cudaError_t error = cudaGetDeviceProperties(&props, 0);
  if (error != cudaSuccess) {
    std::cerr << "cudaGetDeviceProperties() returned an error: " << cudaGetErrorString(error) << std::endl;

    return -1;
  }

  if (!((props.major * 10 + props.minor) >= 80)) {

    std::cerr << "This example requires compute capability at least 80."
              << std::endl;
    notSupported = true;
  }

  if (notSupported) {
    return 0;
  }
```
**EN**: `main()` first checks whether the build and the GPU support this kernel configuration. The code requires CUDA 11+ and compute capability 8.0 or newer because the instantiation targets Ampere Tensor Core execution for FP64. Unsupported environments return early instead of attempting to launch an invalid kernel.
**CN**: `main()` 首先检查当前构建环境和 GPU 是否支持该内核配置。由于实例化目标是 Ampere 上的 FP64 Tensor Core 执行，因此代码要求 CUDA 11 及以上，以及至少 8.0 的计算能力。不满足条件时会提前返回，而不会尝试发射无效内核。

```cpp

  //
  // Parse the command line to obtain SYRK dimensions and scalar values.
  //

  // SYRK problem dimensions.
  int problem[2] = { 128, 128 };

  for (int i = 1; i < argc && i < 3; ++i) {
    std::stringstream ss(arg[i]);
    ss >> problem[i - 1];
  }

  // Scalars used for linear scaling the result of the matrix product.
  double scalars[2] = { 1, 0 };

  for (int i = 3; i < argc && i < 5; ++i) {
    std::stringstream ss(arg[i]);
    ss >> scalars[i - 3];
  }

  //
  // Run the CUTLASS SYRK test.
  //

  cudaError_t result = TestCutlassSyrk(
    problem[0],     // SYRK N dimension
    problem[1],     // SYRK K dimension
    scalars[0],     // alpha
    scalars[1]      // beta
  );

  if (result == cudaSuccess) {
    std::cout << "Passed." << std::endl;
  }

  // Exit.
  return result == cudaSuccess ? 0 : -1;
}

///////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The final block parses up to four command-line values, falling back to `N = 128`, `K = 128`, `alpha = 1`, and `beta = 0`. It then runs the test harness, prints `Passed.` on success, and converts the CUDA-style result code into a conventional process exit status.
**CN**: 最后这段代码最多解析四个命令行参数；如果未提供，则默认使用 `N = 128`、`K = 128`、`alpha = 1` 和 `beta = 0`。之后它运行测试框架，成功时打印 `Passed.`，并把 CUDA 风格的返回值转换成常规进程退出码。

---
## Key Concepts / 关键概念
- CUTLASS `RankK` as the device-level SYRK abstraction / 使用 CUTLASS `RankK` 作为设备级 SYRK 抽象
- Column-major lower-triangular symmetric update / 列主序下三角对称更新
- FP64 Tensor Core MMA on SM80 / SM80 上的 FP64 Tensor Core MMA
- LinearCombination epilogue with `alpha` and `beta` / 带 `alpha` 与 `beta` 的 LinearCombination 尾处理
- Reference-kernel validation for exact correctness / 通过参考内核做精确正确性验证
## Dependencies / 依赖项
- `iostream` — console logging / 控制台日志输出
- `sstream` — command-line string parsing / 命令行字符串解析
- `vector` — host-side result buffers / 主机端结果缓冲区
- `helper.h` — local CUDA error helper macros and utilities / 本地 CUDA 错误检查宏与工具
- `cutlass/gemm/device/rank_k.h` — device-level CUTLASS SYRK/RankK kernel API / 设备级 CUTLASS SYRK/RankK 内核 API
