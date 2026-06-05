# batched_gemm.cu — Code Analysis / 代码分析

## Source / 源文件

`examples/05_batched_gemm/batched_gemm.cu`

## Purpose / 用途

**EN:** Demonstrates two CUTLASS single-precision batched GEMM interfaces: pointer-array batches and fixed-stride batches, then validates both against a host reference.

**CN:** 演示两种 CUTLASS 单精度批量 GEMM 接口：指针数组批量与固定步长批量，并用主机端参考实现验证结果。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40 / 第 1-40 行

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

#include <iostream>
#include <vector>

#include "cutlass/cutlass.h"
#include "cutlass/layout/matrix.h"
#include "cutlass/gemm/device/gemm_array.h"
#include "cutlass/gemm/device/gemm_batched.h"

#pragma warning( disable : 4503)
```

**EN:** License text, standard headers, CUTLASS matrix layout and batched GEMM front ends are included. The MSVC warning suppression avoids long template-name diagnostics.

**CN:** 许可证文本、标准头、CUTLASS 矩阵布局和批量 GEMM 前端在此引入。MSVC 警告抑制用于避免长模板名诊断。

### Lines 42-100 / 第 42-100 行

```cpp
/*
This example demonstrates how to use cutlass to compute a batched strided gemm in two different ways:
  1. By specifying pointers to the first matrices of the batch and the stride between the consecutive
     matrices of the batch (this is called a strided batched gemm).
  2. By copying pointers to all matrices of the batch to the device memory (this is called an array gemm).
In this example, both A and B matrix are non-transpose and column major matrix
batched_C = batched_A x batched_B
As an example, matrix C can be seen as
-----------------------------------------------------------
(0,0,0) | (0,0,1) | (0,0,2) | (1,0,0) | (1,0,1) | (1,0,2) |
-----------------------------------------------------------
(0,1,0) | (0,1,1) | (0,1,2) | (1,1,0) | (1,1,1) | (1,1,2) |
-----------------------------------------------------------
(0,2,0) | (0,2,1) | (0,2,2) | (1,2,0) | (1,2,1) | (1,2,2) |
-----------------------------------------------------------
(0,3,0) | (0,3,1) | (0,3,2) | (1,3,0) | (1,3,1) | (1,3,2) |
-----------------------------------------------------------
(0,4,0) | (0,4,1) | (0,4,2) | (1,4,0) | (1,4,1) | (1,4,2) |
-----------------------------------------------------------
(0,5,0) | (0,5,1) | (0,5,2) | (1,5,0) | (1,5,1) | (1,5,2) |
-----------------------------------------------------------
           batch 0          |           batch 1
where we denote each element with (batch_idx, row_idx, column_idx)
In this example, batch size is 2, M is 6 and N is 3
The stride (batch_stride_C) between the first element of two batches is ldc * n

matrix A can be seen as
---------------------------------------
(0,0,0) | (0,0,1) | (1,0,0) | (1,0,1) |
---------------------------------------
(0,1,0) | (0,1,1) | (1,1,0) | (1,1,1) |
---------------------------------------
(0,2,0) | (0,2,1) | (1,2,0) | (1,2,1) |
---------------------------------------
(0,3,0) | (0,3,1) | (1,3,0) | (1,3,1) |
---------------------------------------
(0,4,0) | (0,4,1) | (1,4,0) | (1,4,1) |
---------------------------------------
(0,5,0) | (0,5,1) | (1,5,0) | (1,5,1) |
---------------------------------------
     batch 0      |      batch 1
, where batch size is 2, M is 6 and K is 2
The stride (batch_stride_A) between the first element of two batches is lda * k

matrix B can be seen as
-----------------------------
(0,0,0) | (0,0,1) | (0,0,2) |
----------------------------- batch 0
(0,1,0) | (0,1,1) | (0,1,2) |
-------------------------------------
(1,0,0) | (1,0,1) | (1,0,2) |
----------------------------- batch 1
(1,1,0) | (1,1,1) | (1,1,2) |
-----------------------------
, where the batch size is 2, N is 3 and K is 2
The stride (batch_stride_B) between the first element of two batches is k


*/
```

**EN:** The introductory comment defines the computation `C_b = A_b * B_b`, contrasts strided-batched versus pointer-array batches, and illustrates the column-major physical layout and batch strides for A, B, and C.

**CN:** 开头注释定义计算 `C_b = A_b * B_b`，对比固定步长批量与指针数组批量，并用图示说明 A、B、C 的列主序物理布局和批间距。

### Lines 102-139 / 第 102-139 行

```cpp
cudaError_t cutlass_array_sgemm(
  int m,
  int n,
  int k,
  float alpha,
  float const * const *A,
  int lda,
  float const * const *B,
  int ldb,
  float * const *C,
  int ldc,
  float beta,
  int batch_count) {

  using Gemm = cutlass::gemm::device::GemmArray<
    float, cutlass::layout::ColumnMajor,
    float, cutlass::layout::ColumnMajor,
    float, cutlass::layout::ColumnMajor
  >;

  Gemm gemm_op;

  cutlass::Status status = gemm_op({
    {m, n, k},
    A, lda,
    B, ldb,
    C, ldc,
    C, ldc,
    {alpha, beta},
    batch_count
  });

  if (status != cutlass::Status::kSuccess) {
    return cudaErrorUnknown;
  }

  return cudaSuccess;
}
```

**EN:** `cutlass_array_sgemm` wraps `cutlass::gemm::device::GemmArray`. The template arguments set A, B, and C element types to `float` and layouts to column-major. The arguments pass problem size, arrays of device pointers, leading dimensions, epilogue scalars, and batch count; non-success status maps to `cudaErrorUnknown`.

**CN:** `cutlass_array_sgemm` 封装 `cutlass::gemm::device::GemmArray`。模板参数把 A、B、C 元素类型设为 `float` 且布局为列主序。实参传入问题规模、设备指针数组、leading dimension、尾声标量和批数量；非成功状态映射为 `cudaErrorUnknown`。

### Lines 141-185 / 第 141-185 行

```cpp
cudaError_t cutlass_strided_batched_sgemm(
  int m, 
  int n,
  int k,
  float alpha,
  float const *A,
  int lda,
  long long int batch_stride_A,
  float const *B,
  int ldb,
  long long int batch_stride_B,
  float *C,
  int ldc,
  long long int batch_stride_C,
  float beta,
  int batch_count) {

  using Gemm = cutlass::gemm::device::GemmBatched<
    float, cutlass::layout::ColumnMajor,
    float, cutlass::layout::ColumnMajor,
    float, cutlass::layout::ColumnMajor
  >;

  Gemm gemm_op;

  cutlass::Status status = gemm_op({
    {m, n, k},
    {A, lda}, 
    batch_stride_A,
    {B, ldb}, 
    batch_stride_B,
    {C, ldc}, 
    batch_stride_C,
    {C, ldc}, 
    batch_stride_C,
    {alpha, beta},
    batch_count
  });

  if (status != cutlass::Status::kSuccess) {
    return cudaErrorUnknown;
  }

  return cudaSuccess;
}
```

**EN:** `cutlass_strided_batched_sgemm` wraps `GemmBatched`. Unlike `GemmArray`, each tensor is described by a base pointer, leading dimension, and batch stride. C is supplied twice so the default epilogue reads old C and writes the updated C.

**CN:** `cutlass_strided_batched_sgemm` 封装 `GemmBatched`。不同于 `GemmArray`，每个张量由基指针、leading dimension 和批间距描述。C 传入两次，使默认尾声读取旧 C 并写回更新后的 C。

### Lines 187-239 / 第 187-239 行

```cpp
template<typename T> 
cudaError_t strided_batched_gemm_nn_reference(
  int m,
  int n,
  int k,
  T alpha,
  std::vector<T> const &A, 
  int lda,
  long long int batch_stride_A,
  std::vector<T> const &B, 
  int ldb,
  long long int batch_stride_B,
  std::vector<T> &C, 
  int ldc,
  long long int batch_stride_C,
  T beta,
  int batch_count) {
  /*
  strided batched gemm NN
  */
  
  cudaError_t result = cudaSuccess;

  if (A.size() < size_t(lda * k * batch_count)) {
    std::cout << "the size of A is too small" << std::endl;
    return cudaErrorInvalidValue;
  }
  if (B.size() < size_t(ldb * n)) {
    std::cout << "the size of B is too small" << std::endl;
    return cudaErrorInvalidValue;
  }
  if (C.size() < size_t(ldc * n * batch_count)) {
    std::cout << "the size of C is too small" << std::endl;
    return cudaErrorInvalidValue;
  }
  
  for (int batch_idx = 0; batch_idx < batch_count; batch_idx++) {
    for (int n_idx = 0; n_idx < n; n_idx++) {
      for (int m_idx = 0; m_idx < m; m_idx++) {
        T accum = beta * C[batch_idx * batch_stride_C + n_idx * ldc + m_idx];
        for (int k_idx = 0; k_idx < k; k_idx++) {
          accum += alpha 
            * A[batch_idx * batch_stride_A + k_idx * lda + m_idx]
            * B[batch_idx * batch_stride_B + n_idx * ldb + k_idx];
        }
        C[batch_idx * batch_stride_C + n_idx * ldc + m_idx] = accum;
      }
    }
  }

  return result;
}

```

**EN:** The templated CPU reference checks vector sizes and performs the NN GEMM loops in batch, column, row, and K order. Index formulas match the column-major, strided-batched layout used by the device kernels.

**CN:** 模板化 CPU 参考实现先检查向量大小，再按批、列、行、K 的顺序执行 NN GEMM 循环。索引公式与设备内核使用的列主序固定步长批量布局一致。

### Lines 241-299 / 第 241-299 行

```cpp
cudaError_t run_batched_gemm(bool use_array) {

  const char* gemm_desc = use_array ? "array" : "strided batched";
  std::cout << "Running " << gemm_desc << " gemm" << std::endl;

  // Arbitrary matrix shape
  int const m = 520;
  int const n = 219;
  int const k = 129;

  int const batch_count = 17;

  // A, B are non-transpose, column major
  int const lda = m;
  int const ldb = k * batch_count;
  int const ldc = m;

  int const count_A = batch_count * lda * k;
  int const count_B = ldb * n;
  int const count_C = batch_count * ldc * n;

  // the memory is batched along K dimension
  long long int batch_stride_A = static_cast<long long int>(lda) * static_cast<long long int>(k);
  long long int batch_stride_B = static_cast<long long int>(k);
  long long int batch_stride_C = static_cast<long long int>(ldc) * static_cast<long long int>(n);

  // alpha and beta
  float alpha = 1.0f;
  float beta = 2.0f;

  cudaError_t result = cudaSuccess;

  // allocate the host memory
  std::vector<float> host_A(count_A);
  std::vector<float> host_B(count_B);
  std::vector<float> host_C(count_C);
  std::vector<float> result_C(count_C);

  // allocate the device memory
  float *A;
  float *B;
  float *C;

  result = cudaMalloc(&A, count_A * sizeof(float));
  if (result != cudaSuccess) {
    std::cerr << "cudaMalloc result = " << result << std::endl;
    return result;
  }
  result = cudaMalloc(&B, count_B * sizeof(float));
  if (result != cudaSuccess) {
    std::cerr << "cudaMalloc result = " << result << std::endl;
    return result;
  }
  result = cudaMalloc(&C, count_C * sizeof(float));
  if (result != cudaSuccess) {
    std::cerr << "cudaMalloc result = " << result << std::endl;
    return result;
  }

```

**EN:** `run_batched_gemm` chooses the interface, defines an arbitrary M/N/K and batch count, computes leading dimensions, allocation sizes, and batch strides, sets `alpha`/`beta`, allocates host vectors and CUDA buffers.

**CN:** `run_batched_gemm` 选择接口，定义任意 M/N/K 与批数量，计算 leading dimension、分配大小和批间距，设置 `alpha`/`beta`，并分配主机向量和 CUDA 缓冲区。

### Lines 300-347 / 第 300-347 行

```cpp
  // Limit range to avoid floating-point errors
  int const kRange = 8;

  // fill A
  for (int b_idx = 0; b_idx < batch_count; b_idx++) {
    for (int col_idx = 0; col_idx < k; col_idx++) {
      for (int row_idx = 0; row_idx < m; row_idx++) {
        host_A[row_idx + col_idx * lda + b_idx * lda * k] = static_cast<float>((row_idx + col_idx * lda + b_idx * lda * k) % kRange);
      }
    }
  }
  // fill B
  for (int b_idx = 0; b_idx < batch_count; b_idx++) {
    for (int col_idx = 0; col_idx < n; col_idx++) {
      for (int row_idx = 0; row_idx < k; row_idx++) {
        host_B[row_idx + col_idx * ldb + b_idx * k] = static_cast<float>(((n + k * ldb + batch_count * k) - (row_idx + col_idx * ldb + b_idx * k)) % kRange);
      }
    }
  }
  // fill C
  for (int b_idx = 0; b_idx < batch_count; b_idx++) {
    for (int col_idx = 0; col_idx < n; col_idx++) {
      for (int row_idx = 0; row_idx < m; row_idx++) {
        host_C[row_idx + col_idx * ldc + b_idx * ldc * n] = 1.f;
      }
    }
  }

  // ref memory
  std::vector<float> ref_A(host_A);
  std::vector<float> ref_B(host_B);
  std::vector<float> ref_C(host_C);
  // copy host memory to device
  result = cudaMemcpy(A, host_A.data(), count_A * sizeof(float), cudaMemcpyHostToDevice);
  if (result != cudaSuccess) {
    std::cerr << "cudaMemcpy result = " << result << std::endl;
    return result;
  }
  result = cudaMemcpy(B, host_B.data(), count_B * sizeof(float), cudaMemcpyHostToDevice);
  if (result != cudaSuccess) {
    std::cerr << "cudaMemcpy result = " << result << std::endl;
    return result;
  }
  result = cudaMemcpy(C, host_C.data(), count_C * sizeof(float), cudaMemcpyHostToDevice);
  if (result != cudaSuccess) {
    std::cerr << "cudaMemcpy result = " << result << std::endl;
    return result;
  }
```

**EN:** Host A, B, and C are initialized with small integer-like float values to reduce roundoff differences. Copies move all operands to the device before launching CUTLASS.

**CN:** 主机端 A、B、C 用较小的类整数浮点值初始化，以降低舍入误差。随后把所有操作数拷贝到设备端以便启动 CUTLASS。

### Lines 349-412 / 第 349-412 行

```cpp
  // run cutlass
  if (use_array) {
    // allocate the host memory for the pointers to the matrices of the batch
    std::vector<float*> host_ptr_A(batch_count);
    std::vector<float*> host_ptr_B(batch_count);
    std::vector<float*> host_ptr_C(batch_count);

    // permute the batch elements to emphasize that GemmArray does not depend on matrices being separated by a fixed stride
    std::vector<size_t> permutation = {14, 11, 3, 10, 1, 13, 9, 4, 6, 16, 8, 15, 7, 12, 0, 2, 5};
    for (size_t b_idx = 0; b_idx < batch_count; b_idx++) {
      host_ptr_A[b_idx] = A + permutation[b_idx] * batch_stride_A;
      host_ptr_B[b_idx] = B + permutation[b_idx] * batch_stride_B;
      host_ptr_C[b_idx] = C + permutation[b_idx] * batch_stride_C;
    }

    // allocate the corresponding device memory
    float const **ptr_A;
    float const **ptr_B;
    float **ptr_C;

    result = cudaMalloc(&ptr_A, batch_count * sizeof(float*));
    if (result != cudaSuccess) {
      std::cerr << "cudaMalloc result = " << result << std::endl;
      return result;
    }
    result = cudaMalloc(&ptr_B, batch_count * sizeof(float*));
    if (result != cudaSuccess) {
      std::cerr << "cudaMalloc result = " << result << std::endl;
      return result;
    }
    result = cudaMalloc(&ptr_C, batch_count * sizeof(float*));
    if (result != cudaSuccess) {
      std::cerr << "cudaMalloc result = " << result << std::endl;
      return result;
    }

    // copy the matrix pointers to the device
    result = cudaMemcpy(ptr_A, host_ptr_A.data(), batch_count * sizeof(float*), cudaMemcpyHostToDevice);
    if (result != cudaSuccess) {
      std::cerr << "cudaMemcpy result = " << result << std::endl;
      return result;
    }
    result = cudaMemcpy(ptr_B, host_ptr_B.data(), batch_count * sizeof(float*), cudaMemcpyHostToDevice);
    if (result != cudaSuccess) {
      std::cerr << "cudaMemcpy result = " << result << std::endl;
      return result;
    }
    result = cudaMemcpy(ptr_C, host_ptr_C.data(), batch_count * sizeof(float*), cudaMemcpyHostToDevice);
    if (result != cudaSuccess) {
      std::cerr << "cudaMemcpy result = " << result << std::endl;
      return result;
    }

    result = cutlass_array_sgemm(m, n, k, alpha, ptr_A, lda, ptr_B, ldb, ptr_C, ldc, beta, batch_count);

    if (result != cudaSuccess)
      return result;
  } else {
    result = cutlass_strided_batched_sgemm(
      m, n, k, alpha, A, lda, batch_stride_A, B, ldb, batch_stride_B, C, ldc, batch_stride_C,
      beta, batch_count);
    if (result != cudaSuccess)
      return result;
  }
```

**EN:** For array GEMM, host arrays of device pointers are built using a permutation to prove fixed strides are not required, then copied to device pointer arrays. For strided GEMM, the base pointers and strides are passed directly.

**CN:** 对于数组 GEMM，代码按一个置换构造设备指针的主机数组，以证明不要求固定步长，然后拷贝到设备端指针数组。对于固定步长 GEMM，则直接传入基指针和步长。

### Lines 414-451 / 第 414-451 行

```cpp
  // copy device memory to host
  result = cudaMemcpy(result_C.data(), C, count_C * sizeof(float), cudaMemcpyDeviceToHost);
  if (result != cudaSuccess) {
    std::cerr << "cudaMemcpy result = " << result << std::endl;
    return result;
  }

  //compare with reference code
  result = strided_batched_gemm_nn_reference(m, n, k, alpha, ref_A, lda, batch_stride_A, ref_B, ldb, batch_stride_B, ref_C, ldc, batch_stride_C,
    beta, batch_count);
  if (result != 0)
    return result;

  // Expect bit-level accuracy for this simple example
  if (ref_C != result_C) {
    std::cout << "CUTLASS " << gemm_desc << " gemm does not run correctly" << std::endl;
    return cudaErrorUnknown;
  }

  // free memory
  result = cudaFree(A);
  if (result != cudaSuccess) {
    std::cerr << "cudaFree result = " << result << std::endl;
    return result;
  }
  result = cudaFree(B);
  if (result != cudaSuccess) {
    std::cerr << "cudaFree result = " << result << std::endl;
    return result;
  }
  result = cudaFree(C);
  if (result != cudaSuccess) {
    std::cerr << "cudaFree result = " << result << std::endl;
    return result;
  }

  return result;
}
```

**EN:** The result tensor is copied back, a host reference is computed, and exact vector equality is required for this simple deterministic data. Device allocations are freed before returning.

**CN:** 结果张量拷回主机，计算主机参考结果，并对这种简单确定性数据要求逐元素精确相等。返回前释放设备分配。

### Lines 453-467 / 第 453-467 行

```cpp
int main() {

  cudaError_t result = cudaSuccess;
  for (bool use_array : {false, true}) {
    result = run_batched_gemm(use_array);
    if (result == cudaSuccess) {
      std::cout << "Passed." << std::endl;
    } else {
      break;
    }
  }

  // Exit.
  return result == cudaSuccess ? 0 : -1;
}
```

**EN:** `main` runs both strided-batched and array variants, prints `Passed.` for each success, and returns process success only if all runs succeeded.

**CN:** `main` 依次运行固定步长批量和数组批量两种变体，每次成功打印 `Passed.`，只有全部成功时进程才返回成功。

## Key Concepts / 关键概念

- **EN:** `GemmArray` receives device arrays of per-batch A/B/C pointers, so batches may be non-contiguous or permuted.
  **CN:** `GemmArray` 接收设备端的每批 A/B/C 指针数组，因此各批矩阵可以不连续，也可以重排。

- **EN:** `GemmBatched` receives one base pointer plus a batch stride for each operand, which is efficient when batches are regularly spaced.
  **CN:** `GemmBatched` 对每个操作数接收一个基指针和批间距，适合批数据按固定间隔存放的场景。

- **EN:** All matrices use `cutlass::layout::ColumnMajor`; leading dimensions are physical row strides, and the sample packs B batches along the K dimension with `ldb = k * batch_count`.
  **CN:** 所有矩阵使用 `cutlass::layout::ColumnMajor`；leading dimension 是物理行跨度，示例中 B 用 `ldb = k * batch_count` 沿 K 维交错存放批次。

- **EN:** No architecture tag or tile shape is specified here; the device-level convenience templates select a default SGEMM kernel for the build target.
  **CN:** 本例没有显式架构标签或 tile 形状；device 级便捷模板会为编译目标选择默认 SGEMM 内核。

- **EN:** The epilogue is the default linear combination `D = alpha * accumulator + beta * C`, with C used in-place as both source and destination.
  **CN:** 尾声阶段使用默认线性组合 `D = alpha * accumulator + beta * C`，其中 C 同时作为源矩阵和目标矩阵原地更新。

## Dependencies / 依赖项

- **EN:** CUDA runtime allocation/copy/free APIs and CUTLASS core headers.
  **CN:** CUDA 运行时分配、拷贝、释放 API，以及 CUTLASS 核心头文件。

- **EN:** `cutlass/gemm/device/gemm_array.h` and `cutlass/gemm/device/gemm_batched.h` provide the two GEMM front ends.
  **CN:** `cutlass/gemm/device/gemm_array.h` 与 `cutlass/gemm/device/gemm_batched.h` 提供两种 GEMM 前端。

- **EN:** The C++ standard library is used for I/O and host vectors.
  **CN:** C++ 标准库用于输入输出和主机端向量。
