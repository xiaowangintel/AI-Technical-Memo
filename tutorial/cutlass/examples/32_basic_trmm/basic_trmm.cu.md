# examples/32_basic_trmm/basic_trmm.cu — Code Analysis / 代码分析
**Source / 源文件**: `examples/32_basic_trmm/basic_trmm.cu`
**Purpose / 用途**: Shows how to instantiate and validate a CUTLASS double-precision TRMM kernel for a left-side lower-triangular matrix multiply on SM80. / 展示如何在 SM80 上为左侧下三角矩阵乘法实例化并验证一个 CUTLASS 双精度 TRMM 内核。
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
  This example demonstrates how to call a CUTLASS TRMM kernel and provides a naive reference
  matrix multiply kernel to verify its correctness.

  The CUTLASS Trmm template is instantiated in the function CutlassStrmmNN. This is kernel computes
  the triangular matrix product (TRMM) using double-precision floating-point arithmetic and assumes
  all matrices have column-major layout.

  The threadblock tile size is chosen as 64x64x16 which offers good performance for large matrices.
  See the CUTLASS Parallel for All blog post for more exposition on the tunable parameters available
  in CUTLASS.

  https://devblogs.nvidia.com/cutlass-linear-algebra-cuda/

  Aside from defining and launching the STRMM kernel, this example does not use any other components
  or utilities within CUTLASS. Such utilities are demonstrated elsewhere in other examples and are
  prevalent in the CUTLASS unit tests.

*/
```
**EN**: The file opens with the usual license block and an overview of the sample. The comment explains that the example demonstrates TRMM, uses double precision, assumes column-major layout, selects a 64x64x16 threadblock tile, and verifies correctness with a simple reference kernel.
**CN**: 文件开头是常见的许可证块和示例概述。注释说明该示例演示的是 TRMM，使用双精度，假定列主序布局，选择 64x64x16 的 threadblock 分块，并用一个简单的参考内核验证正确性。

```cpp

// Standard Library includes
#include <iostream>
#include <sstream>
#include <vector>

// Helper methods to check for errors
#include "helper.h"

//
// CUTLASS includes needed for double-precision TRMM kernel
//

// Defines cutlass::gemm::device::Trmm, the generic Trmm computation template class.
#include "cutlass/gemm/device/trmm.h"
```
**EN**: This include group mirrors the SYRK sample: standard streams and containers, the local helper, and the CUTLASS header `cutlass/gemm/device/trmm.h`. That header provides the device API used to instantiate triangular matrix multiplication.
**CN**: 这一组头文件与 SYRK 示例类似：标准流和容器、本地 helper，以及 CUTLASS 头文件 `cutlass/gemm/device/trmm.h`。该头文件提供了实例化三角矩阵乘法所需的设备 API。

```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////
//
// This function defines a CUTLASS TRMM kernel instantiation, constructs its parameters object,
// and launches it on the CUDA device.
//
///////////////////////////////////////////////////////////////////////////////////////////////////

/// Define a CUTLASS TRMM template and launch a TRMM kernel.
cudaError_t CutlassStrmmNN(
  int M,
  int N,
  double alpha,
  double const *A,
  int lda,
  double const *B,
  int ldb,
  double *C,
  int ldc) {
```
**EN**: These comments and the `CutlassStrmmNN` signature define a BLAS-like wrapper for TRMM. The function takes matrix sizes `M` and `N`, a scalar `alpha`, a triangular left-hand matrix `A`, a dense right-hand matrix `B`, and the destination matrix `C`.
**CN**: 这些注释和 `CutlassStrmmNN` 的函数签名定义了一个 BLAS 风格的 TRMM 包装器。函数接受矩阵尺寸 `M`、`N`，标量 `alpha`，一个位于左侧的三角矩阵 `A`，一个稠密右侧矩阵 `B`，以及输出矩阵 `C`。

```cpp

  // Define type definition for double-precision CUTLASS TRMM with column-major
  // input matrices and 64x64x16 threadblock tile size (chosen by default).
  //
  // To keep the interface manageable, several helpers are defined for plausible compositions
  // including the following example for double-precision TRMM. Typical values are used as
  // default template arguments.
  //
  // To view the full trmm device API interface, see `cutlass/gemm/device/trmm.h`

  using ColumnMajor = cutlass::layout::ColumnMajor;

  using CutlassTrmm = cutlass::gemm::device::Trmm<
    double,
    ColumnMajor,
    cutlass::SideMode::kLeft,
    cutlass::FillMode::kLower,
    cutlass::DiagType::kNonUnit,
    double,
    ColumnMajor,
    double,
    ColumnMajor,
    double,
    cutlass::arch::OpClassTensorOp,
    cutlass::arch::Sm80,
    cutlass::gemm::GemmShape<64, 64, 16>,
    cutlass::gemm::GemmShape<32, 32, 16>,
    cutlass::gemm::GemmShape<8, 8, 4>,
    cutlass::epilogue::thread::LinearCombination<
      double,
      1,
      double,
      double,
      cutlass::epilogue::thread::ScaleType::OnlyAlphaScaling 
    >,
    cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>,
    5,
    1,
    1,
    false,
    cutlass::arch::OpMultiplyAdd
  >;
```
**EN**: This alias is the heart of the sample. `Trmm<double, ColumnMajor, cutlass::SideMode::kLeft, cutlass::FillMode::kLower, cutlass::DiagType::kNonUnit, double, ColumnMajor, double, ColumnMajor, double, ...>` means: matrix `A` is a left-side lower-triangular FP64 matrix with a non-unit diagonal, `B` and `C` are also FP64 column-major matrices, and the computation stays in FP64. `OpClassTensorOp` plus `Sm80` selects Ampere Tensor Core execution. The tile hierarchy is `ThreadblockShape<64,64,16>`, `WarpShape<32,32,16>`, and `InstructionShape<8,8,4>`. The epilogue is `LinearCombination<double,1,double,double, ScaleType::OnlyAlphaScaling>`, which matches TRMM semantics because the result is just `alpha * A * B` without an extra `beta * C` term. `GemmIdentityThreadblockSwizzle<>` keeps default CTA mapping, the next `5` is stage count, the two `1`s are operand alignments, `false` disables split-K, and `OpMultiplyAdd` selects standard real multiply-add math.
**CN**: 这个类型别名是示例的核心。`Trmm<double, ColumnMajor, cutlass::SideMode::kLeft, cutlass::FillMode::kLower, cutlass::DiagType::kNonUnit, double, ColumnMajor, double, ColumnMajor, double, ...>` 的含义是：矩阵 `A` 是位于左侧、采用列主序的下三角 FP64 矩阵，且对角线不是单位值；`B` 和 `C` 也都是列主序 FP64 矩阵，整个计算保持在 FP64 精度中。`OpClassTensorOp` 配合 `Sm80` 选择 Ampere Tensor Core 执行。分块层次分别是 `ThreadblockShape<64,64,16>`、`WarpShape<32,32,16>` 和 `InstructionShape<8,8,4>`。尾处理使用 `LinearCombination<double,1,double,double, ScaleType::OnlyAlphaScaling>`，这与 TRMM 语义相符，因为结果只需要计算 `alpha * A * B`，不包含额外的 `beta * C` 项。`GemmIdentityThreadblockSwizzle<>` 保持默认 CTA 映射，后面的 `5` 是流水线 stage 数，两个 `1` 是操作数对齐要求，`false` 关闭 split-K，而 `OpMultiplyAdd` 选择标准实数乘加。

```cpp

  // Define a CUTLASS TRMM type
  CutlassTrmm trmm_operator;

  // Construct the CUTLASS TRMM arguments object.
  //
  // One of CUTLASS's design patterns is to define trmm argument objects that are constructible
  // in host code and passed to kernels by value. These may include pointers, strides, scalars,
  // and other arguments needed by Trmm and its components.
  //
  // The benefits of this pattern are (1.) a structured, composable strategy for passing host-constructible
  // arguments to kernels and (2.) minimized initialization overhead on kernel entry.
  //
  CutlassTrmm::Arguments args(cutlass::gemm::GemmUniversalMode::kGemm,
                              {M, N, M}, // Trmm Problem dimensions in Left-Side Mode
                              1, // batch_count,
                              {alpha}, // Scalars used in the Epilogue
                              reinterpret_cast<void const *>(A),
                              reinterpret_cast<void const *>(B),
                              reinterpret_cast<void *>(C), // destination matrix D (may be different memory than source C matrix)
                              (int64_t)M*M, // Batch strides
                              (int64_t)M*N,
                              (int64_t)M*N,
                              lda,
                              ldb,
                              ldc);

  //
  // Launch the CUTLASS TRMM kernel.
  //
  
  cutlass::Status status = trmm_operator(args);

  //
  // Return a cudaError_t if the CUTLASS TRMM operator returned an error code.
  //

  if (status != cutlass::Status::kSuccess) {
    return cudaErrorUnknown;
  }

  // Return success, if no errors were encountered.
  return cudaSuccess;
}
```
**EN**: After defining the alias, the code creates the operator and a host-side `Arguments` object. The problem size is passed as `{M, N, M}` in left-side mode because the triangular matrix dimension is the reduction dimension. The A, B, and D pointers plus batch strides and leading dimensions are recorded in the argument object, and launching the kernel is as simple as calling `trmm_operator(args)`. As in the SYRK sample, the CUTLASS status is translated back to a CUDA-style return code.
**CN**: 定义完类型后，代码创建操作对象和一个主机端 `Arguments` 对象。在左侧模式下，问题规模以 `{M, N, M}` 传入，因为三角矩阵的尺寸同时也是归约维度。参数对象记录了 A、B、D 指针，以及 batch 步幅和前导维；发射内核只需要调用 `trmm_operator(args)`。和 SYRK 示例一样，CUTLASS 返回的状态最终会被转换回 CUDA 风格错误码。

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
  int seed = 0,
  cutlass::FillMode fill_mode = cutlass::FillMode::kInvalid) {

  int i = threadIdx.x + blockIdx.x * blockDim.x;
  int j = threadIdx.y + blockIdx.y * blockDim.y;

  if (i < rows && j < columns) {
    if (fill_mode == cutlass::FillMode::kLower && i < j) return;
    else if (fill_mode == cutlass::FillMode::kUpper && i > j) return;
    int offset = i + j * ldm;

    // Generate arbitrary elements.
    int const k = 16807;
    int const m = 16;
    double value = double(((offset + seed) * k % m) - m / 2);

    matrix[offset] = value;
  }
}

/// Simple function to initialize a matrix to arbitrary small integers.
cudaError_t InitializeMatrix(double *matrix, int ldm, int rows, int columns, int seed = 0,
                             cutlass::FillMode fill_mode = cutlass::FillMode::kInvalid) {

  dim3 block(16, 16);
  dim3 grid(
    (rows + block.x - 1) / block.x,
    (columns + block.y - 1) / block.y
  );

  InitializeMatrix_kernel<<< grid, block >>>(matrix, ldm, rows, columns, seed, fill_mode);

  return cudaGetLastError();
}
```
**EN**: The generic CUDA helper section starts with a matrix initializer that optionally respects a triangular fill mode. Threads outside the requested lower or upper triangle return early, so the same helper can initialize dense matrices and triangular matrices. `InitializeMatrix()` wraps the kernel launch with a standard 16x16 grid.
**CN**: 通用 CUDA 辅助部分首先给出一个矩阵初始化内核，它可以选择性地遵守三角填充模式。若线程落在不需要的上三角或下三角区域，就会直接返回，因此同一个辅助内核既能初始化稠密矩阵，也能初始化三角矩阵。`InitializeMatrix()` 则用标准 16x16 网格包装该内核启动。

```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////

/// Allocates device memory for a matrix then fills with arbitrary small integers.
cudaError_t AllocateMatrix(double **matrix, int ldm, int rows, int columns, int seed = 0,
                           cutlass::FillMode fill_mode = cutlass::FillMode::kInvalid) {
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
  result = InitializeMatrix(*matrix, ldm, rows, columns, seed, fill_mode);

  if (result != cudaSuccess) {
    std::cerr << "Failed to initialize matrix: "
      << cudaGetErrorString(result) << std::endl;
    return result;
  }

  return result;
}
```
**EN**: `AllocateMatrix()` reuses the initialization logic while exposing the fill-mode parameter. This is important for TRMM because matrix `A` must be allocated as a lower-triangular matrix, while `B`, `C_cutlass`, and `C_reference` are allocated as ordinary dense matrices.
**CN**: `AllocateMatrix()` 在封装分配逻辑的同时保留了 fill-mode 参数。这对 TRMM 很重要，因为矩阵 `A` 必须按下三角矩阵方式分配，而 `B`、`C_cutlass` 和 `C_reference` 则是普通稠密矩阵。

```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////

/// Naive reference TRMM computation.
__global__ void ReferenceTrmm_kernel(
  int M,
  int N,
  double alpha,
  double const *A,
  int lda,
  double const *B,
  int ldb,
  double *C,
  int ldc) {

  int i = threadIdx.x + blockIdx.x * blockDim.x;
  int j = threadIdx.y + blockIdx.y * blockDim.y;

  if (i < M && j < N) {
    double accumulator = 0;

    for (int k = 0; k < M; ++k) {
      accumulator += A[i + k * lda] * B[k + j * ldb]; // Since A is in Left-Side Mode
    }

    C[i + j * ldc] = alpha * accumulator;
  }
}

/// Reference TRMM computation.
cudaError_t ReferenceTrmm(
  int M,
  int N,
  double alpha,
  double const *A,
  int lda,
  double const *B,
  int ldb,
  double *C,
  int ldc) {

  dim3 block(16, 16);
  dim3 grid(
    (M + block.x - 1) / block.x,
    (N + block.y - 1) / block.y
  );

  ReferenceTrmm_kernel<<< grid, block >>>(M, N, alpha, A, lda, B, ldb, C, ldc);

  return cudaGetLastError();
}
```
**EN**: This block implements the reference TRMM. The kernel computes `C = alpha * A * B` for every output element, and because the initializer already zeroed the unused part of the triangular matrix `A`, the reference loop can simply iterate `k` from `0` to `M - 1` without special-case bounds logic. `ReferenceTrmm()` only handles the launch geometry.
**CN**: 这一段实现参考 TRMM。内核对每个输出元素计算 `C = alpha * A * B`；由于初始化阶段已经把三角矩阵 `A` 中未使用的区域清零，因此参考循环可以直接让 `k` 从 `0` 迭代到 `M - 1`，而不需要额外的边界分支。`ReferenceTrmm()` 只负责构造启动网格。

```cpp

///////////////////////////////////////////////////////////////////////////////////////////////////

/// Allocate several matrices in GPU device memory and call a double-precision
/// CUTLASS TRMM kernel.
cudaError_t TestCutlassTrmm(int M, int N, double alpha) {
  cudaError_t result;

  //
  // Define several matrices to be used as operands to TRMM kernels.
  //

  // Compute leading dimensions for each matrix.
  int lda = M;
  int ldb = M;
  int ldc = M;

  // Compute size in bytes of the C matrix.
  size_t sizeof_C = sizeof(double) * ldc * N;

  // Define pointers to matrices in GPU device memory.
  double *A;
  double *B;
  double *C_cutlass;
  double *C_reference;

  //
  // Allocate matrices in GPU device memory with arbitrary seeds.
  //

  result = AllocateMatrix(&A, lda, M, M, 0, cutlass::FillMode::kLower);

  if (result !=  cudaSuccess) {
    return result;
  }

  result = AllocateMatrix(&B, ldb, M, N, 17);

  if (result !=  cudaSuccess) {
    cudaFree(A);
    return result;
  }

  result = AllocateMatrix(&C_cutlass, ldc, M, N, 101);

  if (result != cudaSuccess) {
    cudaFree(A);
    cudaFree(B);
    return result;
  }

  result = AllocateMatrix(&C_reference, ldc, M, N, 101);

  if (result != cudaSuccess) {
    cudaFree(A);
    cudaFree(B);
    cudaFree(C_cutlass);
    return result;
  }
```
**EN**: `TestCutlassTrmm()` derives leading dimensions, computes output size, and allocates four device matrices. The important detail is the first allocation: `A` is created with `cutlass::FillMode::kLower`, which makes the test data match the template instantiation's structural assumptions. The reference and CUTLASS output matrices are initialized identically so later comparison is meaningful.
**CN**: `TestCutlassTrmm()` 会推导前导维、计算输出大小，并分配四个设备矩阵。最重要的细节是第一步：`A` 使用 `cutlass::FillMode::kLower` 创建，这使测试数据与模板实例化的结构假设保持一致。CUTLASS 输出矩阵和参考输出矩阵采用相同初始化，因此后续比较才有意义。

```cpp

  result = cudaMemcpy(C_reference, C_cutlass, sizeof_C, cudaMemcpyDeviceToDevice);

  if (result != cudaSuccess) {
    std::cerr << "Failed to copy C_cutlass matrix to C_reference: "
      << cudaGetErrorString(result) << std::endl;

    cudaFree(C_reference);
    cudaFree(C_cutlass);
    cudaFree(B);
    cudaFree(A);

    return result;
  }

  //
  // Launch CUTLASS TRMM.
  //

  result = CutlassStrmmNN(M, N, alpha, A, lda, B, ldb, C_cutlass, ldc);

  if (result != cudaSuccess) {
    std::cerr << "CUTLASS TRMM kernel failed: "
      << cudaGetErrorString(result) << std::endl;

    cudaFree(C_reference);
    cudaFree(C_cutlass);
    cudaFree(B);
    cudaFree(A);

    return result;
  }
```
**EN**: The test harness copies the initial CUTLASS output buffer into the reference buffer, then launches `CutlassStrmmNN()`. Although the TRMM epilogue ignores `beta`, keeping both paths initialized from the same state makes the structure parallel to the other examples and simplifies debugging.
**CN**: 测试框架会先把 CUTLASS 输出缓冲区的初始内容复制到参考缓冲区，然后发射 `CutlassStrmmNN()`。尽管 TRMM 的 epilogue 不使用 `beta`，让两条路径从相同初始状态出发仍然能保持与其他示例一致的结构，也更便于调试。

```cpp

  //
  // Verify.
  //

  // Launch reference TRMM
  result = ReferenceTrmm(M, N, alpha, A, lda, B, ldb, C_reference, ldc);

  if (result != cudaSuccess) {
    std::cerr << "Reference TRMM kernel failed: "
      << cudaGetErrorString(result) << std::endl;

    cudaFree(C_reference);
    cudaFree(C_cutlass);
    cudaFree(B);
    cudaFree(A);

    return result;
  }

  // Copy to host and verify equivalence.
  std::vector<double> host_cutlass(ldc * N, 0);
  std::vector<double> host_reference(ldc * N, 0);

  result = cudaMemcpy(host_cutlass.data(), C_cutlass, sizeof_C, cudaMemcpyDeviceToHost);

  if (result != cudaSuccess) {
    std::cerr << "Failed to copy CUTLASS TRMM results: "
      << cudaGetErrorString(result) << std::endl;

    cudaFree(C_reference);
    cudaFree(C_cutlass);
    cudaFree(B);
    cudaFree(A);

    return result;
  }

  result = cudaMemcpy(host_reference.data(), C_reference, sizeof_C, cudaMemcpyDeviceToHost);

  if (result != cudaSuccess) {
    std::cerr << "Failed to copy Reference TRMM results: "
      << cudaGetErrorString(result) << std::endl;

    cudaFree(C_reference);
    cudaFree(C_cutlass);
    cudaFree(B);
    cudaFree(A);

    return result;
  }

  //
  // Free device memory allocations.
  //

  cudaFree(C_reference);
  cudaFree(C_cutlass);
  cudaFree(B);
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
**EN**: This block runs the reference kernel, copies both outputs to host memory, frees all device allocations, and compares the results exactly. The cleanup is deliberately explicit, so each failure path releases only the resources that have already been allocated.
**CN**: 这一段会运行参考内核，把两份输出复制回主机内存，释放所有设备分配，并精确比较结果。代码中的清理流程刻意写得很显式，因此每条失败路径只会释放已经成功分配的资源。

```cpp
}

///////////////////////////////////////////////////////////////////////////////////////////////////

/// Entry point to basic_trmm example.
//
// usage:
//
//   00_basic_trmm <M> <N> <alpha> 
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
**EN**: `main()` begins with the same environment guards as the SYRK sample: CUDA 11+ and compute capability 8.0+ are required for this FP64 Tensor Core configuration. Unsupported builds or GPUs exit early with an explanatory message.
**CN**: `main()` 以与 SYRK 示例相同的环境检查开头：该 FP64 Tensor Core 配置需要 CUDA 11 及以上，以及至少 8.0 的计算能力。不满足条件的构建或 GPU 会在提示信息后提前退出。

```cpp

  //
  // Parse the command line to obtain TRMM dimensions and scalar values.
  //

  // TRMM problem dimensions.
  int problem[2] = { 128, 128 };

  for (int i = 1; i < argc && i < 3; ++i) {
    std::stringstream ss(arg[i]);
    ss >> problem[i - 1];
  }

  // Scalars used for linear scaling the result of the matrix product.
  double scalars[1] = { 1 };

  for (int i = 3; i < argc && i < 4; ++i) {
    std::stringstream ss(arg[i]);
    ss >> scalars[i - 3];
  }

  //
  // Run the CUTLASS TRMM test.
  //

  cudaError_t result = TestCutlassTrmm(
    problem[0],     // TRMM M dimension
    problem[1],     // TRMM N dimension
    scalars[0]     // alpha
  );

  if (result == cudaSuccess) {
    std::cout << "Passed." << std::endl;
  }

  // Exit.
  return result == cudaSuccess ? 0 : -1;
}

///////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN**: The final block parses optional `M`, `N`, and `alpha` values, defaults to a 128x128 problem with `alpha = 1`, runs the test harness, prints `Passed.` on success, and converts the CUDA-style result into a normal program exit status.
**CN**: 最后这段代码解析可选的 `M`、`N` 和 `alpha` 参数，默认为 128x128 且 `alpha = 1`，然后运行测试框架；成功时打印 `Passed.`，并把 CUDA 风格返回值转换为普通程序退出码。

---
## Key Concepts / 关键概念
- CUTLASS `Trmm` as a triangular-matrix device kernel / 使用 CUTLASS `Trmm` 表示三角矩阵设备内核
- Left-side lower-triangular non-unit-diagonal semantics / 左侧下三角非单位对角语义
- `OnlyAlphaScaling` epilogue pattern for TRMM / 适用于 TRMM 的 `OnlyAlphaScaling` 尾处理模式
- Column-major FP64 Tensor Core tiling on SM80 / SM80 上的列主序 FP64 Tensor Core 分块
- Reference-based validation with structured test data / 通过结构化测试数据进行参考校验
## Dependencies / 依赖项
- `iostream` — console logging / 控制台日志输出
- `sstream` — command-line parsing / 命令行解析
- `vector` — host-side result storage / 主机端结果存储
- `helper.h` — local CUDA error helper macros and utilities / 本地 CUDA 错误检查宏与工具
- `cutlass/gemm/device/trmm.h` — device-level CUTLASS TRMM kernel API / 设备级 CUTLASS TRMM 内核 API
