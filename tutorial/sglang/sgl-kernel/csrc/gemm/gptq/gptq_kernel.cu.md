# gptq_kernel.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/gptq/gptq_kernel.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Headers and compile-time setup
```cpp
/*
Adapted from https://github.com/turboderp/exllamav2 and
https://github.com/qwopqwop200/GPTQ-for-LLaMa
*/

#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAGuard.h>
#include <cuda_fp16.h>
#include <cuda_runtime.h>
#include <torch/all.h>

#include <cstdint>
#include <cstdio>

#include "compat.cuh"
#include "matrix_view.cuh"
#include "qdq_2.cuh"
#include "qdq_3.cuh"
#include "qdq_4.cuh"
#include "qdq_8.cuh"

namespace sglang {
namespace gptq {
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 25-52: Local implementation details
```cpp
#define BLOCK_KN_SIZE 128
#define BLOCK_M_SIZE_MAX 8
#define MAX_GROUPS_IN_BLOCK (BLOCK_KN_SIZE / 32)
#define MAX_Q_GEMM_ROWS 50
#define MAX_Q_GEMM_ROWS_8BIT 24
#define MAX_ALT_GEMM_ROWS 8
#define THREADS_X 32
#define THREADS_Y 32
#define DIVIDE(x, size) (((x) + (size) - 1) / (size))

#if defined(USE_ROCM)
#include <hipblas/hipblas.h>
__host__ __forceinline__ hipblasStatus_t __compat_hipblasHgemm(
    hipblasHandle_t handle,
    hipblasOperation_t transA,
    hipblasOperation_t transB,
    int m,
    int n,
    int k,
    const half* alpha,
    const half* AP,
    int lda,
    const half* BP,
    int ldb,
    const half* beta,
    half* CP,
    int ldc) {
  return hipblasHgemm(
```
**EN:** This section fills in the local implementation details around `MAX_GROUPS_IN_BLOCK`, completing the behavior required by the file.
**CN:** 本段补充了`MAX_GROUPS_IN_BLOCK`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 53-82: Device helpers and synchronization
```cpp
      handle,
      transA,
      transB,
      m,
      n,
      k,
      reinterpret_cast<const hipblasHalf*>(alpha),
      reinterpret_cast<const hipblasHalf*>(AP),
      lda,
      reinterpret_cast<const hipblasHalf*>(BP),
      ldb,
      reinterpret_cast<const hipblasHalf*>(beta),
      reinterpret_cast<hipblasHalf*>(CP),
      ldc);
}
#define hipblasHgemm __compat_hipblasHgemm

// Previous version of PyTorch were converting to rocBLAS instead of hipBLAS.
#define rocblas_operation_none HIPBLAS_OP_N
#define rocblas_hgemm __compat_hipblasHgemm
#endif

__forceinline__ __device__ half2 dot22_8(half2 (&dq)[4], const half* a_ptr, const half2 g_result) {
  half2 result = {};
  const half2* a2_ptr = (const half2*)a_ptr;
#pragma unroll
  for (int i = 0; i < 4; i++)
    result = __hfma2(dq[i], *a2_ptr++, result);
  return __hadd2(result, g_result);
}
```
**EN:** This section implements `dot22_8`, `__hfma2`, `__hadd2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`dot22_8`、`__hfma2`、`__hadd2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 83-110: Device helpers and synchronization
```cpp

__forceinline__ __device__ float dot22_8_f(half2 (&dq)[4], const half* a_ptr) {
  half2 result = {};
  const half2* a2_ptr = (const half2*)a_ptr;
#pragma unroll
  for (int i = 0; i < 4; i++)
    result = __hfma2(dq[i], *a2_ptr++, result);
  return __half2float(__low2half(result)) + __half2float(__high2half(result));
}

__forceinline__ __device__ half2 dot22_8(half2 (&dq)[4], const half* a_ptr, const half2 g_result, const half qs_h) {
  half2 result = {};
  const half2* a2_ptr = (const half2*)a_ptr;
#pragma unroll
  for (int i = 0; i < 4; i++)
    result = __hfma2(dq[i], *a2_ptr++, result);
  return __hfma2(result, __halves2half2(qs_h, qs_h), g_result);
}

__forceinline__ __device__ half2 dot22_16(half2 (&dq)[8], const half* a_ptr, const half2 g_result, const half qs_h) {
  half2 result = {};
  const half2* a2_ptr = (const half2*)a_ptr;
#pragma unroll
  for (int i = 0; i < 8; i++)
    result = __hfma2(dq[i], *a2_ptr++, result);
  return __hfma2(result, __halves2half2(qs_h, qs_h), g_result);
}
```
**EN:** This section implements `dot22_8_f`, `dot22_8`, `dot22_16`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`dot22_8_f`、`dot22_8`、`dot22_16`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 111-138: Device helpers and synchronization
```cpp
__forceinline__ __device__ half2 dot22_32(half2 (&dq)[16], const half* a_ptr, const half2 g_result, const half qs_h) {
  half2 result = {};
  const half2* a2_ptr = (const half2*)a_ptr;
#pragma unroll
  for (int i = 0; i < 16; i += 1)
    result = __hfma2(dq[i], *a2_ptr++, result);
  return __hfma2(result, __halves2half2(qs_h, qs_h), g_result);
}

__forceinline__ __device__ float dot22_8_f(half2 (&dq)[4], const half* a_ptr, const float g_result, const float qs_f) {
  half2 result = {};
  const half2* a2_ptr = (const half2*)a_ptr;
#pragma unroll
  for (int i = 0; i < 4; i++)
    result = __hfma2(dq[i], *a2_ptr++, result);
  float result_f = __half2float(__low2half(result)) + __half2float(__high2half(result));
  return fma(result_f, qs_f, g_result);
}

__forceinline__ __device__ float dot22_16_f(half2 (&dq)[8], const half* a_ptr, const float g_result, const float qs_f) {
  half2 result = {};
  const half2* a2_ptr = (const half2*)a_ptr;
#pragma unroll
  for (int i = 0; i < 8; i++)
    result = __hfma2(dq[i], *a2_ptr++, result);
  float result_f = __half2float(__low2half(result)) + __half2float(__high2half(result));
  return fma(result_f, qs_f, g_result);
}
```
**EN:** This section implements `dot22_32`, `dot22_8_f`, `dot22_16_f`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`dot22_32`、`dot22_8_f`、`dot22_16_f`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 139-165: Device helpers and synchronization
```cpp

__forceinline__ __device__ float
dot22_32_f(half2 (&dq)[16], const half* a_ptr, const float g_result, const float qs_f) {
  half2 result = {};
  const half2* a2_ptr = (const half2*)a_ptr;
#pragma unroll
  for (int i = 0; i < 16; i += 1)
    result = __hfma2(dq[i], *a2_ptr++, result);
  float result_f = __half2float(__low2half(result)) + __half2float(__high2half(result));
  return fma(result_f, qs_f, g_result);
}

__forceinline__ __device__ half dot22_8_h(half2 (&dq)[4], const half* a_ptr, const half g_result, const half qs_h) {
  // Use FP32 accumulator to avoid potential overflow since unscaled weights are
  // in the range -128..127

  float result = {};
#pragma unroll
  for (int i = 0; i < 4; i++) {
    half2 w01 = dq[i];
    float w0 = __low2float(w01);
    float w1 = __high2float(w01);
    float x0 = __half2float(*a_ptr++);
    float x1 = __half2float(*a_ptr++);
    result = fma(w0, x0, result);
    result = fma(w1, x1, result);
  }
```
**EN:** This section implements `dot22_32_f`, `dot22_8_h`, `__hfma2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`dot22_32_f`、`dot22_8_h`、`__hfma2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 166-191: Device helpers and synchronization
```cpp
  float qs = __half2float(qs_h);
  result *= qs;
  half result_h = __float2half_rn(result);
  return __hadd(result_h, g_result);
}

__forceinline__ __device__ half dot22_16_h(half2 (&dq)[8], const half* a_ptr, const half g_result, const half qs_h) {
  half2 result = {};
  const half2* a2_ptr = (const half2*)a_ptr;
#pragma unroll
  for (int i = 0; i < 8; i++)
    result = __hfma2(dq[i], *a2_ptr++, result);
  half result_h = __hadd(__low2half(result), __high2half(result));
  return __hfma(result_h, qs_h, g_result);
}

__forceinline__ __device__ half dot22_32_h(half2 (&dq)[16], const half* a_ptr, const half g_result, const half qs_h) {
  half2 result = {};
  const half2* a2_ptr = (const half2*)a_ptr;
#pragma unroll
  for (int i = 0; i < 16; i += 1)
    result = __hfma2(dq[i], *a2_ptr++, result);
  half result_h = __hadd(__low2half(result), __high2half(result));
  return __hfma(result_h, qs_h, g_result);
}
```
**EN:** This section implements `dot22_16_h`, `dot22_32_h`, `__half2float`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`dot22_16_h`、`dot22_32_h`、`__half2float`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 192-220: Kernel implementation
```cpp
typedef void (*fp_gemm_half_q_half_gptq_kernel)(
    const half*,
    const uint32_t*,
    const uint32_t*,
    const half*,
    half*,
    const int,
    const int,
    const int,
    const int,
    const int*);

template <bool first_block, int m_count>
__global__ void gemm_half_q_half_gptq_4bit_kernel(
    const half* __restrict__ a,
    const uint32_t* __restrict__ b_q_weight,
    const uint32_t* __restrict__ b_gptq_qzeros,
    const half* __restrict__ b_gptq_scales,
    half* __restrict__ c,
    const int size_m,
    const int size_n,
    const int size_k,
    const int groups,
    const int* __restrict__ b_q_perm) {
  MatrixView_half a_(a, size_m, size_k);
  MatrixView_half_rw c_(c, size_m, size_n);
  MatrixView_q4_row b_gptq_qzeros_(b_gptq_qzeros, groups, size_n);
  MatrixView_half b_gptq_scales_(b_gptq_scales, groups, size_n);
```
**EN:** This section implements `gemm_half_q_half_gptq_4bit_kernel`, `void`, `a_`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`gemm_half_q_half_gptq_4bit_kernel`、`void`、`a_`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 221-248: Device helpers and synchronization
```cpp
  auto t = threadIdx.x;

  // Block
  auto offset_n = blockIdx.x * BLOCK_KN_SIZE * 4;
  auto offset_m = blockIdx.y * m_count;
  auto offset_k = blockIdx.z * BLOCK_KN_SIZE;

  int end_k = min(offset_k + BLOCK_KN_SIZE, size_k);

  int n = offset_n + t * 4;

  // Preload block_a
  __shared__ half block_a[m_count][BLOCK_KN_SIZE];

  if (offset_k + t < end_k) {
    for (int m = 0; m < m_count; ++m) {
      const half* a_ptr = a_.item_ptr(offset_m + m, 0);
      half* block_a_ptr = block_a[m];

      half a0;
      if (b_q_perm)
        a0 = a_ptr[b_q_perm[offset_k + t]];
      else
        a0 = a_ptr[offset_k + t];
      block_a_ptr[t] = a0;
    }
  }
```
**EN:** This section implements `min`, `item_ptr`, `a_ptr`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`min`、`item_ptr`、`a_ptr`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 249-282: Device helpers and synchronization
```cpp
  // Zero output
  if (n >= size_n) return;

  if (blockIdx.z == 0) {
    for (int m = 0; m < m_count; m++)
      *((uint64_t*)c_.item_ptr(offset_m + m, n)) = 0;
  }

  __syncthreads();

  // Find initial group
  int groupsize = size_k / groups;
  int group = offset_k / groupsize;
  int nextgroup = offset_k + groupsize;

  // a, b offset
  int qk = offset_k / (32 / 4);

  const uint32_t* b_ptr = b_q_weight + qk * size_n + n;
  const half* a_ptr = &block_a[0][0];
  int a_stride = BLOCK_KN_SIZE;

  // Initial group
  int zeros[4];
  float scales[4];
  half2 z1z16[4][2];
  half2 y1y16[4][2];
  b_gptq_qzeros_.item4(zeros, group, n);
  b_gptq_scales_.item4_f(scales, group, n);
  dequant_4bit_8_prep_zero(zeros[0] + 1, z1z16[0], y1y16[0]);
  dequant_4bit_8_prep_zero(zeros[1] + 1, z1z16[1], y1y16[1]);
  dequant_4bit_8_prep_zero(zeros[2] + 1, z1z16[2], y1y16[2]);
  dequant_4bit_8_prep_zero(zeros[3] + 1, z1z16[3], y1y16[3]);
```
**EN:** This section implements `__syncthreads`, `item4`, `item4_f`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__syncthreads`、`item4`、`item4_f`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 283-310: Control flow and branching
```cpp
  // Column result
  float block_c[m_count][4] = {};

  // Dequantize and multiply
  int k = offset_k;
  while (k < end_k) {
    if (k == nextgroup) {
      group++;
      nextgroup += groupsize;
      b_gptq_qzeros_.item4(zeros, group, n);
      b_gptq_scales_.item4_f(scales, group, n);
      dequant_4bit_8_prep_zero(zeros[0] + 1, z1z16[0], y1y16[0]);
      dequant_4bit_8_prep_zero(zeros[1] + 1, z1z16[1], y1y16[1]);
      dequant_4bit_8_prep_zero(zeros[2] + 1, z1z16[2], y1y16[2]);
      dequant_4bit_8_prep_zero(zeros[3] + 1, z1z16[3], y1y16[3]);
    }

#pragma unroll
    for (int j = 0; j < 4; j++) {
      const int4* b_ptr4 = (int4*)b_ptr;
      int4 load_int4 = *b_ptr4;

      half2 dq[4][4];
      dequant_4bit_8_gptq(load_int4.x, dq[0], z1z16[0], y1y16[0], size_n, false);
      dequant_4bit_8_gptq(load_int4.y, dq[1], z1z16[1], y1y16[1], size_n, false);
      dequant_4bit_8_gptq(load_int4.z, dq[2], z1z16[2], y1y16[2], size_n, false);
      dequant_4bit_8_gptq(load_int4.w, dq[3], z1z16[3], y1y16[3], size_n, false);
```
**EN:** This section drives `item4`, `item4_f`, `dequant_4bit_8_prep_zero` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`item4`、`item4_f`、`dequant_4bit_8_prep_zero`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 311-334: Device helpers and synchronization
```cpp
#pragma unroll
      for (int m = 0; m < m_count; m++) {
        block_c[m][0] = fma(dot22_8_f(dq[0], a_ptr + m * a_stride), scales[0], block_c[m][0]);
        block_c[m][1] = fma(dot22_8_f(dq[1], a_ptr + m * a_stride), scales[1], block_c[m][1]);
        block_c[m][2] = fma(dot22_8_f(dq[2], a_ptr + m * a_stride), scales[2], block_c[m][2]);
        block_c[m][3] = fma(dot22_8_f(dq[3], a_ptr + m * a_stride), scales[3], block_c[m][3]);
      }

      b_ptr += size_n;
      a_ptr += 8;
    }

    k += 32;
  }

  for (int m = 0; m < m_count; m++) {
    half2* out = (half2*)c_.item_ptr(offset_m + m, n);
    half2 result01 = __halves2half2(__float2half_rn(block_c[m][0]), __float2half_rn(block_c[m][1]));
    half2 result23 = __halves2half2(__float2half_rn(block_c[m][2]), __float2half_rn(block_c[m][3]));
    atomicAdd(out, result01);
    atomicAdd(out + 1, result23);
  }
}
```
**EN:** This section implements `fma`, `item_ptr`, `__halves2half2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`fma`、`item_ptr`、`__halves2half2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 335-362: Kernel implementation
```cpp
template <bool first_block, int m_count>
__global__ void gemm_half_q_half_gptq_2bit_kernel(
    const half* __restrict__ a,
    const uint32_t* __restrict__ b_q_weight,
    const uint32_t* __restrict__ b_gptq_qzeros,
    const half* __restrict__ b_gptq_scales,
    half* __restrict__ c,
    const int size_m,
    const int size_n,
    const int size_k,
    const int groups,
    const int* __restrict__ b_q_perm) {
  MatrixView_half a_(a, size_m, size_k);
  MatrixView_half_rw c_(c, size_m, size_n);
  MatrixView_q2_row b_gptq_qzeros_(b_gptq_qzeros, groups, size_n);
  MatrixView_half b_gptq_scales_(b_gptq_scales, groups, size_n);

  auto t = threadIdx.x;

  // Block
  auto offset_n = blockIdx.x * BLOCK_KN_SIZE * 4;
  auto offset_m = blockIdx.y * m_count;
  auto offset_k = blockIdx.z * BLOCK_KN_SIZE;

  int end_k = min(offset_k + BLOCK_KN_SIZE, size_k);

  int n = offset_n + t * 4;
```
**EN:** This section implements `gemm_half_q_half_gptq_2bit_kernel`, `a_`, `c_`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`gemm_half_q_half_gptq_2bit_kernel`、`a_`、`c_`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 363-389: Device helpers and synchronization
```cpp
  // Preload block_a
  __shared__ half block_a[m_count][BLOCK_KN_SIZE];

  if (offset_k + t < end_k) {
    for (int m = 0; m < m_count; ++m) {
      const half* a_ptr = a_.item_ptr(offset_m + m, 0);
      half* block_a_ptr = block_a[m];

      half a0;
      if (b_q_perm)
        a0 = a_ptr[b_q_perm[offset_k + t]];
      else
        a0 = a_ptr[offset_k + t];
      block_a_ptr[t] = a0;
    }
  }

  // Zero output
  if (n >= size_n) return;

  if (blockIdx.z == 0) {
    for (int m = 0; m < m_count; m++)
      *((uint64_t*)c_.item_ptr(offset_m + m, n)) = 0;
  }

  __syncthreads();
```
**EN:** This section implements `item_ptr`, `__syncthreads`, `a_ptr`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`item_ptr`、`__syncthreads`、`a_ptr`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 390-418: Control flow and branching
```cpp
  // Find initial group
  int groupsize = size_k / groups;
  int group = offset_k / groupsize;
  int nextgroup = offset_k + groupsize;

  // a, b offset
  int qk = offset_k / (32 / 2);

  const uint32_t* b_ptr = b_q_weight + qk * size_n + n;
  const half* a_ptr = &block_a[0][0];
  int a_stride = BLOCK_KN_SIZE;

  // Initial group
  int zeros[4];
  half scales[4];
  b_gptq_qzeros_.item4(zeros, group, n);
  b_gptq_scales_.item4(scales, group, n);
  // Column result
  half block_c[m_count][4] = {};

  // Dequantize and multiply
  int k = offset_k;
  while (k < end_k) {
    if (k == nextgroup) {
      group++;
      nextgroup += groupsize;
      b_gptq_qzeros_.item4(zeros, group, n);
      b_gptq_scales_.item4(scales, group, n);
    }
```
**EN:** This section drives `item4`, `b_ptr`, `a_ptr` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`item4`、`b_ptr`、`a_ptr`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 419-445: Control flow and branching
```cpp

#pragma unroll
    for (int j = 0; j < 1; j++) {
      const int4* b_ptr4 = (int4*)b_ptr;
      int4 load_int4 = *b_ptr4;

      half2 dq[4][8];
      dequant_2bit_16(load_int4.x, dq[0], size_n, zeros[0] + 1);
      dequant_2bit_16(load_int4.y, dq[1], size_n, zeros[1] + 1);
      dequant_2bit_16(load_int4.z, dq[2], size_n, zeros[2] + 1);
      dequant_2bit_16(load_int4.w, dq[3], size_n, zeros[3] + 1);

#pragma unroll
      for (int m = 0; m < m_count; m++) {
        block_c[m][0] = dot22_16_h(dq[0], a_ptr + m * a_stride, block_c[m][0], scales[0]);
        block_c[m][1] = dot22_16_h(dq[1], a_ptr + m * a_stride, block_c[m][1], scales[1]);
        block_c[m][2] = dot22_16_h(dq[2], a_ptr + m * a_stride, block_c[m][2], scales[2]);
        block_c[m][3] = dot22_16_h(dq[3], a_ptr + m * a_stride, block_c[m][3], scales[3]);
      }

      b_ptr += size_n;
      a_ptr += 16;
    }

    k += 16;
  }
```
**EN:** This section drives `dequant_2bit_16`, `dot22_16_h`, `b_ptr4` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`dequant_2bit_16`、`dot22_16_h`、`b_ptr4`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 446-473: Kernel implementation
```cpp
  for (int m = 0; m < m_count; m++) {
    half2* out = (half2*)c_.item_ptr(offset_m + m, n);
    half2 result01 = __halves2half2(block_c[m][0], block_c[m][1]);
    half2 result23 = __halves2half2(block_c[m][2], block_c[m][3]);
    atomicAdd(out, result01);
    atomicAdd(out + 1, result23);
  }
}

template <bool first_block, int m_count>
__global__ void gemm_half_q_half_gptq_3bit_kernel(
    const half* __restrict__ a,
    const uint32_t* __restrict__ b_q_weight,
    const uint32_t* __restrict__ b_gptq_qzeros,
    const half* __restrict__ b_gptq_scales,
    half* __restrict__ c,
    const int size_m,
    const int size_n,
    const int size_k,
    const int groups,
    const int* __restrict__ b_q_perm) {
  MatrixView_half a_(a, size_m, size_k);
  MatrixView_half_rw c_(c, size_m, size_n);
  MatrixView_q3_row b_gptq_qzeros_(b_gptq_qzeros, groups, size_n);
  MatrixView_half b_gptq_scales_(b_gptq_scales, groups, size_n);

  auto t = threadIdx.x;
```
**EN:** This section implements `gemm_half_q_half_gptq_3bit_kernel`, `item_ptr`, `__halves2half2`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`gemm_half_q_half_gptq_3bit_kernel`、`item_ptr`、`__halves2half2`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 474-502: Device helpers and synchronization
```cpp
  // Block
  auto offset_n = blockIdx.x * BLOCK_KN_SIZE * 4;
  auto offset_m = blockIdx.y * m_count;
  auto offset_k = blockIdx.z * BLOCK_KN_SIZE;

  int end_k = min(offset_k + BLOCK_KN_SIZE, size_k);

  int n = offset_n + t * 4;

  // Preload block_a
  __shared__ half block_a[m_count][BLOCK_KN_SIZE];

  if (offset_k + t < end_k) {
    for (int m = 0; m < m_count; ++m) {
      const half* a_ptr = a_.item_ptr(offset_m + m, 0);
      half* block_a_ptr = block_a[m];

      half a0;
      if (b_q_perm)
        a0 = a_ptr[b_q_perm[offset_k + t]];
      else
        a0 = a_ptr[offset_k + t];
      block_a_ptr[t] = a0;
    }
  }

  // Zero output
  if (n >= size_n) return;
```
**EN:** This section implements `min`, `item_ptr`, `a_ptr`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`min`、`item_ptr`、`a_ptr`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 503-529: Device helpers and synchronization
```cpp
  if (blockIdx.z == 0) {
    for (int m = 0; m < m_count; m++)
      *((uint64_t*)c_.item_ptr(offset_m + m, n)) = 0;
  }

  __syncthreads();

  // Find initial group
  int groupsize = size_k / groups;
  int group = offset_k / groupsize;
  int nextgroup = offset_k + groupsize;

  // a, b offset
  int qk = offset_k / 32 * 3;

  const uint32_t* b_ptr = b_q_weight + qk * size_n + n;
  const half* a_ptr = &block_a[0][0];
  int a_stride = BLOCK_KN_SIZE;

  // Initial group
  int zeros[4];
  half scales[4];
  b_gptq_qzeros_.item4(zeros, group, n);
  b_gptq_scales_.item4(scales, group, n);
  // Column result
  half block_c[m_count][4] = {};
```
**EN:** This section implements `__syncthreads`, `item4`, `b_ptr`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__syncthreads`、`item4`、`b_ptr`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 530-555: Control flow and branching
```cpp
  // Dequantize and multiply
  int k = offset_k;
  while (k < end_k) {
    if (k == nextgroup) {
      group++;
      nextgroup += groupsize;
      b_gptq_qzeros_.item4(zeros, group, n);
      b_gptq_scales_.item4(scales, group, n);
    }

#pragma unroll
    for (int j = 0; j < 1; j++) {
      int4 load_int4[3];
      load_int4[0] = *((int4*)b_ptr);
      b_ptr += size_n;
      load_int4[1] = *((int4*)b_ptr);
      b_ptr += size_n;
      load_int4[2] = *((int4*)b_ptr);
      b_ptr += size_n;

      half2 dq[4][16];
      dequant_3bit_32(load_int4[0].x, load_int4[1].x, load_int4[2].x, dq[0], size_n, zeros[0] + 1);
      dequant_3bit_32(load_int4[0].y, load_int4[1].y, load_int4[2].y, dq[1], size_n, zeros[1] + 1);
      dequant_3bit_32(load_int4[0].z, load_int4[1].z, load_int4[2].z, dq[2], size_n, zeros[2] + 1);
      dequant_3bit_32(load_int4[0].w, load_int4[1].w, load_int4[2].w, dq[3], size_n, zeros[3] + 1);
```
**EN:** This section drives `item4`, `dequant_3bit_32` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`item4`、`dequant_3bit_32`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 556-577: Device helpers and synchronization
```cpp
#pragma unroll
      for (int m = 0; m < m_count; m++) {
        block_c[m][0] = dot22_32_h(dq[0], a_ptr + m * a_stride, block_c[m][0], scales[0]);
        block_c[m][1] = dot22_32_h(dq[1], a_ptr + m * a_stride, block_c[m][1], scales[1]);
        block_c[m][2] = dot22_32_h(dq[2], a_ptr + m * a_stride, block_c[m][2], scales[2]);
        block_c[m][3] = dot22_32_h(dq[3], a_ptr + m * a_stride, block_c[m][3], scales[3]);
      }
      a_ptr += 32;
    }

    k += 32;
  }

  for (int m = 0; m < m_count; m++) {
    half2* out = (half2*)c_.item_ptr(offset_m + m, n);
    half2 result01 = __halves2half2(block_c[m][0], block_c[m][1]);
    half2 result23 = __halves2half2(block_c[m][2], block_c[m][3]);
    atomicAdd(out, result01);
    atomicAdd(out + 1, result23);
  }
}
```
**EN:** This section implements `dot22_32_h`, `item_ptr`, `__halves2half2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`dot22_32_h`、`item_ptr`、`__halves2half2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 578-605: Kernel implementation
```cpp
template <bool first_block, int m_count>
__global__ void gemm_half_q_half_gptq_8bit_kernel(
    const half* __restrict__ a,
    const uint32_t* __restrict__ b_q_weight,
    const uint32_t* __restrict__ b_gptq_qzeros,
    const half* __restrict__ b_gptq_scales,
    half* __restrict__ c,
    const int size_m,
    const int size_n,
    const int size_k,
    const int groups,
    const int* __restrict__ b_q_perm) {
  MatrixView_half a_(a, size_m, size_k);
  MatrixView_half_rw c_(c, size_m, size_n);
  MatrixView_q8_row b_gptq_qzeros_(b_gptq_qzeros, groups, size_n);
  MatrixView_half b_gptq_scales_(b_gptq_scales, groups, size_n);

  auto t = threadIdx.x;

  // Block
  auto offset_n = blockIdx.x * BLOCK_KN_SIZE * 4;
  auto offset_m = blockIdx.y * m_count;
  auto offset_k = blockIdx.z * BLOCK_KN_SIZE;

  int end_k = min(offset_k + BLOCK_KN_SIZE, size_k);

  int n = offset_n + t * 4;
```
**EN:** This section implements `gemm_half_q_half_gptq_8bit_kernel`, `a_`, `c_`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`gemm_half_q_half_gptq_8bit_kernel`、`a_`、`c_`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 606-632: Device helpers and synchronization
```cpp
  // Preload block_a
  __shared__ half block_a[m_count][BLOCK_KN_SIZE];

  if (offset_k + t < end_k) {
    for (int m = 0; m < m_count; ++m) {
      const half* a_ptr = a_.item_ptr(offset_m + m, 0);
      half* block_a_ptr = block_a[m];

      half a0;
      if (b_q_perm)
        a0 = a_ptr[b_q_perm[offset_k + t]];
      else
        a0 = a_ptr[offset_k + t];
      block_a_ptr[t] = a0;
    }
  }

  // Zero output
  if (n >= size_n) return;

  if (blockIdx.z == 0) {
    for (int m = 0; m < m_count; m++)
      *((uint64_t*)c_.item_ptr(offset_m + m, n)) = 0;
  }

  __syncthreads();
```
**EN:** This section implements `item_ptr`, `__syncthreads`, `a_ptr`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`item_ptr`、`__syncthreads`、`a_ptr`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 633-661: Control flow and branching
```cpp
  // Find initial group
  int groupsize = size_k / groups;
  int group = offset_k / groupsize;
  int nextgroup = offset_k + groupsize;

  // a, b offset
  int qk = offset_k / (32 / 8);

  const uint32_t* b_ptr = b_q_weight + qk * size_n + n;
  const half* a_ptr = &block_a[0][0];
  int a_stride = BLOCK_KN_SIZE;

  // Initial group
  int zeros[4];
  half scales[4];
  b_gptq_qzeros_.item4(zeros, group, n);
  b_gptq_scales_.item4(scales, group, n);
  // Column result
  half block_c[m_count][4] = {};

  // Dequantize and multiply
  int k = offset_k;
  while (k < end_k) {
    if (k == nextgroup) {
      group++;
      nextgroup += groupsize;
      b_gptq_qzeros_.item4(zeros, group, n);
      b_gptq_scales_.item4(scales, group, n);
    }
```
**EN:** This section drives `item4`, `b_ptr`, `a_ptr` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`item4`、`b_ptr`、`a_ptr`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 662-687: Control flow and branching
```cpp

#pragma unroll
    for (int j = 0; j < 4; j++) {
      int4 load_int4[2];
      load_int4[0] = *((int4*)b_ptr);
      b_ptr += size_n;
      load_int4[1] = *((int4*)b_ptr);
      b_ptr += size_n;

      half2 dq[4][4];
      dequant_8bit_8(load_int4[0].x, load_int4[1].x, dq[0], size_n, zeros[0] + 1);
      dequant_8bit_8(load_int4[0].y, load_int4[1].y, dq[1], size_n, zeros[1] + 1);
      dequant_8bit_8(load_int4[0].z, load_int4[1].z, dq[2], size_n, zeros[2] + 1);
      dequant_8bit_8(load_int4[0].w, load_int4[1].w, dq[3], size_n, zeros[3] + 1);

      for (int m = 0; m < m_count; m++) {
        block_c[m][0] = dot22_8_h(dq[0], a_ptr + m * a_stride, block_c[m][0], scales[0]);
        block_c[m][1] = dot22_8_h(dq[1], a_ptr + m * a_stride, block_c[m][1], scales[1]);
        block_c[m][2] = dot22_8_h(dq[2], a_ptr + m * a_stride, block_c[m][2], scales[2]);
        block_c[m][3] = dot22_8_h(dq[3], a_ptr + m * a_stride, block_c[m][3], scales[3]);
      }
      a_ptr += 8;
    }
    k += 32;
  }
```
**EN:** This section drives `dequant_8bit_8`, `dot22_8_h` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`dequant_8bit_8`、`dot22_8_h`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 688-715: Device helpers and synchronization
```cpp
  for (int m = 0; m < m_count; m++) {
    half2* out = (half2*)c_.item_ptr(offset_m + m, n);
    half2 result01 = __halves2half2(block_c[m][0], block_c[m][1]);
    half2 result23 = __halves2half2(block_c[m][2], block_c[m][3]);
    atomicAdd(out, result01);
    atomicAdd(out + 1, result23);
  }
}

fp_gemm_half_q_half_gptq_kernel pick_gemm_half_q_half_gptq_kernel(bool first_block, const int m_count, const int bit) {
#define SELECT_KERNEL(M_COUNT)                                             \
  if (m_count == M_COUNT) {                                                \
    if (bit == 2) return gemm_half_q_half_gptq_2bit_kernel<true, M_COUNT>; \
    if (bit == 3) return gemm_half_q_half_gptq_3bit_kernel<true, M_COUNT>; \
    if (bit == 4) return gemm_half_q_half_gptq_4bit_kernel<true, M_COUNT>; \
    if (bit == 8) return gemm_half_q_half_gptq_8bit_kernel<true, M_COUNT>; \
  }
#if BLOCK_M_SIZE_MAX >= 1
  SELECT_KERNEL(1);
#endif
#if BLOCK_M_SIZE_MAX >= 2
  SELECT_KERNEL(2);
#endif
#if BLOCK_M_SIZE_MAX >= 3
  SELECT_KERNEL(3);
#endif
#if BLOCK_M_SIZE_MAX >= 4
  SELECT_KERNEL(4);
```
**EN:** This section implements `pick_gemm_half_q_half_gptq_kernel`, `SELECT_KERNEL`, `item_ptr`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`pick_gemm_half_q_half_gptq_kernel`、`SELECT_KERNEL`、`item_ptr`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 716-743: Local implementation details
```cpp
#endif
#if BLOCK_M_SIZE_MAX >= 5
  SELECT_KERNEL(5);
#endif
#if BLOCK_M_SIZE_MAX >= 6
  SELECT_KERNEL(6);
#endif
#if BLOCK_M_SIZE_MAX >= 7
  SELECT_KERNEL(7);
#endif
#if BLOCK_M_SIZE_MAX >= 8
  SELECT_KERNEL(8);
#endif
  return NULL;
}

void gemm_half_q_half_cuda_part(
    const half* a,
    const uint32_t* b_q_weight,
    const uint32_t* b_gptq_qzeros,
    const half* b_gptq_scales,
    const int* b_q_perm,
    half* c,
    int size_m,
    int size_n,
    int size_k,
    int m_count,
    int groups,
```
**EN:** This section fills in the local implementation details around `SELECT_KERNEL`, completing the behavior required by the file.
**CN:** 本段补充了`SELECT_KERNEL`周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 744-772: Kernel implementation
```cpp
    int bit) {
  dim3 blockDim, gridDim;
  blockDim.x = BLOCK_KN_SIZE;
  blockDim.y = 1;
  blockDim.z = 1;
  gridDim.x = DIVIDE(size_n, BLOCK_KN_SIZE * 4);
  gridDim.y = DIVIDE(size_m, m_count);
  gridDim.z = DIVIDE(size_k, BLOCK_KN_SIZE);

  fp_gemm_half_q_half_gptq_kernel kernel = pick_gemm_half_q_half_gptq_kernel(true, m_count, bit);

  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
  kernel<<<gridDim, blockDim, 0, stream>>>(
      a, b_q_weight, b_gptq_qzeros, b_gptq_scales, c, size_m, size_n, size_k, groups, b_q_perm);
}

__global__ void reconstruct_exllama_8bit_kernel(
    const uint32_t* __restrict__ b_q_weight,
    const int* __restrict__ b_q_perm,
    const uint32_t* __restrict__ b_gptq_qzeros,
    const half* __restrict__ b_gptq_scales,
    const int size_k,
    const int size_n,
    const int groups,
    half* __restrict__ b) {
  MatrixView_half_rw b_(b, size_k, size_n);
  MatrixView_q8_row b_gptq_qzeros_(b_gptq_qzeros, groups, size_n);
  MatrixView_half b_gptq_scales_(b_gptq_scales, groups, size_n);
```
**EN:** This section implements `reconstruct_exllama_8bit_kernel`, `DIVIDE`, `pick_gemm_half_q_half_gptq_kernel`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`reconstruct_exllama_8bit_kernel`、`DIVIDE`、`pick_gemm_half_q_half_gptq_kernel`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 773-799: Device helpers and synchronization
```cpp
  auto offset_k = BLOCK_KN_SIZE * blockIdx.y;
  auto offset_n = BLOCK_KN_SIZE * blockIdx.x * 4;

  int end_k = min(offset_k + BLOCK_KN_SIZE, size_k);

  // Preload remapping table
  __shared__ int perm[BLOCK_KN_SIZE];
  auto t = threadIdx.x;

  if (b_q_perm) {
    if (offset_k + t < size_k) perm[t] = b_q_perm[offset_k + t];
  }

  // Column
  int n = offset_n + t * 4;
  if (n >= size_n) return;

  // Find initial group
  int groupsize = size_k / groups;
  int group = offset_k / groupsize;
  int nextgroup = offset_k + groupsize;

  // b offset
  int qk = offset_k / (32 / 8);

  const uint32_t* b_ptr = b_q_weight + qk * size_n + n;
```
**EN:** This section implements `min`, `b_ptr`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`min`、`b_ptr`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 800-825: Device helpers and synchronization
```cpp
  // Initial zeros/scale
  int zeros[4];
  half2 scales[4];
  b_gptq_qzeros_.item4(zeros, group, n);
  b_gptq_scales_.item4_h2(scales, group, n);

  __syncthreads();

  int k = offset_k;
  int lk = 0;

  while (k < end_k) {
    if (k == nextgroup) {
      group++;
      nextgroup += groupsize;
      b_gptq_qzeros_.item4(zeros, group, n);
      b_gptq_scales_.item4_h2(scales, group, n);
    }

    for (int p = 0; p < 4; p++) {
      int4 load_int4[2];
      load_int4[0] = *((int4*)b_ptr);
      b_ptr += size_n;
      load_int4[1] = *((int4*)b_ptr);
      b_ptr += size_n;
```
**EN:** This section implements `item4`, `item4_h2`, `__syncthreads`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`item4`、`item4_h2`、`__syncthreads`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 826-853: Control flow and branching
```cpp
      half2 dq[4][4];
      dequant_8bit_8(load_int4[0].x, load_int4[1].x, dq[0], size_n, zeros[0] + 1);
      dequant_8bit_8(load_int4[0].y, load_int4[1].y, dq[1], size_n, zeros[1] + 1);
      dequant_8bit_8(load_int4[0].z, load_int4[1].z, dq[2], size_n, zeros[2] + 1);
      dequant_8bit_8(load_int4[0].w, load_int4[1].w, dq[3], size_n, zeros[3] + 1);

      // half* dqh = (half*)dq;
      if (b_q_perm) {
        for (int j = 0; j < 4; j++) {
          for (int v = 0; v < 4; v++)
            dq[v][j] = __hmul2(scales[v], dq[v][j]);
          b_.set4(
              perm[lk++], n, __low2half(dq[0][j]), __low2half(dq[1][j]), __low2half(dq[2][j]), __low2half(dq[3][j]));
          b_.set4(
              perm[lk++],
              n,
              __high2half(dq[0][j]),
              __high2half(dq[1][j]),
              __high2half(dq[2][j]),
              __high2half(dq[3][j]));
        }
      } else {
        for (int j = 0; j < 4; j++) {
          for (int v = 0; v < 4; v++)
            dq[v][j] = __hmul2(scales[v], dq[v][j]);
          b_.set4(
              offset_k + lk++,
              n,
```
**EN:** This section drives `dequant_8bit_8`, `__hmul2`, `set4` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`dequant_8bit_8`、`__hmul2`、`set4`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 854-884: Kernel implementation
```cpp
              __low2half(dq[0][j]),
              __low2half(dq[1][j]),
              __low2half(dq[2][j]),
              __low2half(dq[3][j]));
          b_.set4(
              offset_k + lk++,
              n,
              __high2half(dq[0][j]),
              __high2half(dq[1][j]),
              __high2half(dq[2][j]),
              __high2half(dq[3][j]));
        }
      }
    }
    k += 32;
  }
}

__global__ void reconstruct_exllama_4bit_kernel(
    const uint32_t* __restrict__ b_q_weight,
    const int* __restrict__ b_q_perm,
    const uint32_t* __restrict__ b_gptq_qzeros,
    const half* __restrict__ b_gptq_scales,
    const int size_k,
    const int size_n,
    const int groups,
    half* __restrict__ b) {
  MatrixView_half_rw b_(b, size_k, size_n);
  MatrixView_q4_row b_gptq_qzeros_(b_gptq_qzeros, groups, size_n);
  MatrixView_half b_gptq_scales_(b_gptq_scales, groups, size_n);
```
**EN:** This section implements `reconstruct_exllama_4bit_kernel`, `__low2half`, `set4`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`reconstruct_exllama_4bit_kernel`、`__low2half`、`set4`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 885-911: Device helpers and synchronization
```cpp
  auto offset_k = BLOCK_KN_SIZE * blockIdx.y;
  auto offset_n = BLOCK_KN_SIZE * blockIdx.x * 4;

  int end_k = min(offset_k + BLOCK_KN_SIZE, size_k);

  // Preload remapping table
  __shared__ int perm[BLOCK_KN_SIZE];
  auto t = threadIdx.x;

  if (b_q_perm) {
    if (offset_k + t < size_k) perm[t] = b_q_perm[offset_k + t];
  }

  // Column
  int n = offset_n + t * 4;
  if (n >= size_n) return;

  // Find initial group
  int groupsize = size_k / groups;
  int group = offset_k / groupsize;
  int nextgroup = offset_k + groupsize;

  // b offset
  int qk = offset_k / (32 / 4);

  const uint32_t* b_ptr = b_q_weight + qk * size_n + n;
```
**EN:** This section implements `min`, `b_ptr`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`min`、`b_ptr`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 912-939: Device helpers and synchronization
```cpp
  // Initial zeros/scale
  int zeros[4];
  half2 scales[4];
  half2 z1z16[4][2];
  half2 y1y16[4][2];
  b_gptq_qzeros_.item4(zeros, group, n);
  b_gptq_scales_.item4_h2(scales, group, n);
  dequant_4bit_8_prep_zero(zeros[0] + 1, z1z16[0], y1y16[0]);
  dequant_4bit_8_prep_zero(zeros[1] + 1, z1z16[1], y1y16[1]);
  dequant_4bit_8_prep_zero(zeros[2] + 1, z1z16[2], y1y16[2]);
  dequant_4bit_8_prep_zero(zeros[3] + 1, z1z16[3], y1y16[3]);

  __syncthreads();

  int k = offset_k;
  int lk = 0;

  while (k < end_k) {
    if (k == nextgroup) {
      group++;
      nextgroup += groupsize;
      b_gptq_qzeros_.item4(zeros, group, n);
      b_gptq_scales_.item4_h2(scales, group, n);
      dequant_4bit_8_prep_zero(zeros[0] + 1, z1z16[0], y1y16[0]);
      dequant_4bit_8_prep_zero(zeros[1] + 1, z1z16[1], y1y16[1]);
      dequant_4bit_8_prep_zero(zeros[2] + 1, z1z16[2], y1y16[2]);
      dequant_4bit_8_prep_zero(zeros[3] + 1, z1z16[3], y1y16[3]);
    }
```
**EN:** This section implements `item4`, `item4_h2`, `dequant_4bit_8_prep_zero`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`item4`、`item4_h2`、`dequant_4bit_8_prep_zero`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 940-966: Control flow and branching
```cpp

    for (int p = 0; p < 4; p++) {
      half2 dq[4][4];
      const int4* b_ptr4 = (int4*)b_ptr;
      int4 load_int4 = *b_ptr4;

      dequant_4bit_8_gptq(load_int4.x, dq[0], z1z16[0], y1y16[0], size_n, false);
      dequant_4bit_8_gptq(load_int4.y, dq[1], z1z16[1], y1y16[1], size_n, false);
      dequant_4bit_8_gptq(load_int4.z, dq[2], z1z16[2], y1y16[2], size_n, false);
      dequant_4bit_8_gptq(load_int4.w, dq[3], z1z16[3], y1y16[3], size_n, false);

      b_ptr += size_n;
      // half* dqh = (half*)dq;
      if (b_q_perm) {
        for (int j = 0; j < 4; j++) {
          for (int v = 0; v < 4; v++)
            dq[v][j] = __hmul2(scales[v], dq[v][j]);
          b_.set4(
              perm[lk++], n, __low2half(dq[0][j]), __low2half(dq[1][j]), __low2half(dq[2][j]), __low2half(dq[3][j]));
          b_.set4(
              perm[lk++],
              n,
              __high2half(dq[0][j]),
              __high2half(dq[1][j]),
              __high2half(dq[2][j]),
              __high2half(dq[3][j]));
        }
```
**EN:** This section drives `dequant_4bit_8_gptq`, `__hmul2`, `set4` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`dequant_4bit_8_gptq`、`__hmul2`、`set4`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 967-991: Control flow and branching
```cpp
      } else {
        for (int j = 0; j < 4; j++) {
          for (int v = 0; v < 4; v++)
            dq[v][j] = __hmul2(scales[v], dq[v][j]);
          b_.set4(
              offset_k + lk++,
              n,
              __low2half(dq[0][j]),
              __low2half(dq[1][j]),
              __low2half(dq[2][j]),
              __low2half(dq[3][j]));
          b_.set4(
              offset_k + lk++,
              n,
              __high2half(dq[0][j]),
              __high2half(dq[1][j]),
              __high2half(dq[2][j]),
              __high2half(dq[3][j]));
        }
      }
    }
    k += 32;
  }
}
```
**EN:** This section drives `__hmul2`, `set4` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`__hmul2`、`set4`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 992-1021: Kernel implementation
```cpp
__global__ void reconstruct_exllama_3bit_kernel(
    const uint32_t* __restrict__ b_q_weight,
    const int* __restrict__ b_q_perm,
    const uint32_t* __restrict__ b_gptq_qzeros,
    const half* __restrict__ b_gptq_scales,
    const int size_k,
    const int size_n,
    const int groups,
    half* __restrict__ b) {
  MatrixView_half_rw b_(b, size_k, size_n);
  MatrixView_q3_row b_gptq_qzeros_(b_gptq_qzeros, groups, size_n);
  MatrixView_half b_gptq_scales_(b_gptq_scales, groups, size_n);

  auto offset_k = BLOCK_KN_SIZE * blockIdx.y;
  auto offset_n = BLOCK_KN_SIZE * blockIdx.x * 4;

  int end_k = min(offset_k + BLOCK_KN_SIZE, size_k);

  // Preload remapping table
  __shared__ int perm[BLOCK_KN_SIZE];
  auto t = threadIdx.x;

  if (b_q_perm) {
    if (offset_k + t < size_k) perm[t] = b_q_perm[offset_k + t];
  }

  // Column
  int n = offset_n + t * 4;
  if (n >= size_n) return;
```
**EN:** This section implements `reconstruct_exllama_3bit_kernel`, `b_`, `b_gptq_qzeros_`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`reconstruct_exllama_3bit_kernel`、`b_`、`b_gptq_qzeros_`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1022-1049: Device helpers and synchronization
```cpp
  // Find initial group
  int groupsize = size_k / groups;
  int group = offset_k / groupsize;
  int nextgroup = offset_k + groupsize;

  // b offset
  int qk = offset_k / 32 * 3;

  const uint32_t* b_ptr = b_q_weight + qk * size_n + n;

  // Initial zeros/scale
  int zeros[4];
  half2 scales[4];
  b_gptq_qzeros_.item4(zeros, group, n);
  b_gptq_scales_.item4_h2(scales, group, n);

  __syncthreads();

  int k = offset_k;
  int lk = 0;

  while (k < end_k) {
    if (k == nextgroup) {
      group++;
      nextgroup += groupsize;
      b_gptq_qzeros_.item4(zeros, group, n);
      b_gptq_scales_.item4_h2(scales, group, n);
    }
```
**EN:** This section implements `item4`, `item4_h2`, `__syncthreads`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`item4`、`item4_h2`、`__syncthreads`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1050-1079: Control flow and branching
```cpp

    for (int p = 0; p < 1; p++) {
      int4 load_int4[3];
      load_int4[0] = *((int4*)b_ptr);
      b_ptr += size_n;
      load_int4[1] = *((int4*)b_ptr);
      b_ptr += size_n;
      load_int4[2] = *((int4*)b_ptr);
      b_ptr += size_n;

      half2 dq[4][16];
      dequant_3bit_32(load_int4[0].x, load_int4[1].x, load_int4[2].x, dq[0], size_n, zeros[0] + 1);
      dequant_3bit_32(load_int4[0].y, load_int4[1].y, load_int4[2].y, dq[1], size_n, zeros[1] + 1);
      dequant_3bit_32(load_int4[0].z, load_int4[1].z, load_int4[2].z, dq[2], size_n, zeros[2] + 1);
      dequant_3bit_32(load_int4[0].w, load_int4[1].w, load_int4[2].w, dq[3], size_n, zeros[3] + 1);

      if (b_q_perm) {
        for (int j = 0; j < 16; j++) {
          for (int v = 0; v < 4; v++)
            dq[v][j] = __hmul2(scales[v], dq[v][j]);
          b_.set4(
              perm[lk++], n, __low2half(dq[0][j]), __low2half(dq[1][j]), __low2half(dq[2][j]), __low2half(dq[3][j]));
          b_.set4(
              perm[lk++],
              n,
              __high2half(dq[0][j]),
              __high2half(dq[1][j]),
              __high2half(dq[2][j]),
              __high2half(dq[3][j]));
        }
```
**EN:** This section drives `dequant_3bit_32`, `__hmul2`, `set4` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`dequant_3bit_32`、`__hmul2`、`set4`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1080-1104: Control flow and branching
```cpp
      } else {
        for (int j = 0; j < 16; j++) {
          for (int v = 0; v < 4; v++)
            dq[v][j] = __hmul2(scales[v], dq[v][j]);
          b_.set4(
              offset_k + lk++,
              n,
              __low2half(dq[0][j]),
              __low2half(dq[1][j]),
              __low2half(dq[2][j]),
              __low2half(dq[3][j]));
          b_.set4(
              offset_k + lk++,
              n,
              __high2half(dq[0][j]),
              __high2half(dq[1][j]),
              __high2half(dq[2][j]),
              __high2half(dq[3][j]));
        }
      }
    }
    k += 32;
  }
}
```
**EN:** This section drives `__hmul2`, `set4` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`__hmul2`、`set4`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1105-1134: Kernel implementation
```cpp
__global__ void reconstruct_exllama_2bit_kernel(
    const uint32_t* __restrict__ b_q_weight,
    const int* __restrict__ b_q_perm,
    const uint32_t* __restrict__ b_gptq_qzeros,
    const half* __restrict__ b_gptq_scales,
    const int size_k,
    const int size_n,
    const int groups,
    half* __restrict__ b) {
  MatrixView_half_rw b_(b, size_k, size_n);
  MatrixView_q2_row b_gptq_qzeros_(b_gptq_qzeros, groups, size_n);
  MatrixView_half b_gptq_scales_(b_gptq_scales, groups, size_n);

  auto offset_k = BLOCK_KN_SIZE * blockIdx.y;
  auto offset_n = BLOCK_KN_SIZE * blockIdx.x * 4;

  int end_k = min(offset_k + BLOCK_KN_SIZE, size_k);

  // Preload remapping table
  __shared__ int perm[BLOCK_KN_SIZE];
  auto t = threadIdx.x;

  if (b_q_perm) {
    if (offset_k + t < size_k) perm[t] = b_q_perm[offset_k + t];
  }

  // Column
  int n = offset_n + t * 4;
  if (n >= size_n) return;
```
**EN:** This section implements `reconstruct_exllama_2bit_kernel`, `b_`, `b_gptq_qzeros_`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`reconstruct_exllama_2bit_kernel`、`b_`、`b_gptq_qzeros_`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1135-1162: Device helpers and synchronization
```cpp
  // Find initial group
  int groupsize = size_k / groups;
  int group = offset_k / groupsize;
  int nextgroup = offset_k + groupsize;

  // b offset
  int qk = offset_k / (32 / 2);

  const uint32_t* b_ptr = b_q_weight + qk * size_n + n;

  // Initial zeros/scale
  int zeros[4];
  half2 scales[4];
  b_gptq_qzeros_.item4(zeros, group, n);
  b_gptq_scales_.item4_h2(scales, group, n);

  __syncthreads();

  int k = offset_k;
  int lk = 0;

  while (k < end_k) {
    if (k == nextgroup) {
      group++;
      nextgroup += groupsize;
      b_gptq_qzeros_.item4(zeros, group, n);
      b_gptq_scales_.item4_h2(scales, group, n);
    }
```
**EN:** This section implements `item4`, `item4_h2`, `__syncthreads`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`item4`、`item4_h2`、`__syncthreads`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1163-1189: Control flow and branching
```cpp

    for (int p = 0; p < 2; p++) {
      const int4* b_ptr4 = (int4*)b_ptr;
      int4 load_int4 = *b_ptr4;

      half2 dq[4][8];
      dequant_2bit_16(load_int4.x, dq[0], size_n, zeros[0] + 1);
      dequant_2bit_16(load_int4.y, dq[1], size_n, zeros[1] + 1);
      dequant_2bit_16(load_int4.z, dq[2], size_n, zeros[2] + 1);
      dequant_2bit_16(load_int4.w, dq[3], size_n, zeros[3] + 1);

      b_ptr += size_n;
      // half* dqh = (half*)dq;
      if (b_q_perm) {
        for (int j = 0; j < 8; j++) {
          for (int v = 0; v < 4; v++)
            dq[v][j] = __hmul2(scales[v], dq[v][j]);
          b_.set4(
              perm[lk++], n, __low2half(dq[0][j]), __low2half(dq[1][j]), __low2half(dq[2][j]), __low2half(dq[3][j]));
          b_.set4(
              perm[lk++],
              n,
              __high2half(dq[0][j]),
              __high2half(dq[1][j]),
              __high2half(dq[2][j]),
              __high2half(dq[3][j]));
        }
```
**EN:** This section drives `dequant_2bit_16`, `__hmul2`, `set4` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`dequant_2bit_16`、`__hmul2`、`set4`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1190-1214: Control flow and branching
```cpp
      } else {
        for (int j = 0; j < 8; j++) {
          for (int v = 0; v < 4; v++)
            dq[v][j] = __hmul2(scales[v], dq[v][j]);
          b_.set4(
              offset_k + lk++,
              n,
              __low2half(dq[0][j]),
              __low2half(dq[1][j]),
              __low2half(dq[2][j]),
              __low2half(dq[3][j]));
          b_.set4(
              offset_k + lk++,
              n,
              __high2half(dq[0][j]),
              __high2half(dq[1][j]),
              __high2half(dq[2][j]),
              __high2half(dq[3][j]));
        }
      }
    }
    k += 32;
  }
}
```
**EN:** This section drives `__hmul2`, `set4` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`__hmul2`、`set4`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1215-1243: Kernel implementation
```cpp
void reconstruct_exllama(
    const uint32_t* b_q_weight,
    const uint32_t* b_gptq_qzeros,
    const half* b_gptq_scales,
    const int* b_q_perm,
    half* out,
    int height,
    int width,
    int groups,
    int bit) {
  dim3 blockDim, gridDim;
  blockDim.x = BLOCK_KN_SIZE;
  blockDim.y = 1;
  gridDim.y = DIVIDE(height, BLOCK_KN_SIZE);
  gridDim.x = DIVIDE(width, BLOCK_KN_SIZE);

  auto reconstruct_exllama_kernel = reconstruct_exllama_4bit_kernel;
  if (bit == 2) {
    reconstruct_exllama_kernel = reconstruct_exllama_2bit_kernel;
  } else if (bit == 3) {
    reconstruct_exllama_kernel = reconstruct_exllama_3bit_kernel;
  } else if (bit == 8) {
    reconstruct_exllama_kernel = reconstruct_exllama_8bit_kernel;
  }

  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
  reconstruct_exllama_kernel<<<gridDim, blockDim, 0, stream>>>(
      b_q_weight, b_q_perm, b_gptq_qzeros, b_gptq_scales, height, width, groups, out);
}
```
**EN:** This section implements `reconstruct_exllama`, `DIVIDE`, `getCurrentCUDAStream`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`reconstruct_exllama`、`DIVIDE`、`getCurrentCUDAStream`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1244-1270: Kernel implementation
```cpp

__global__ void gemm_half_q_half_alt_4bit_kernel(
    const half2* __restrict__ vec,
    const uint32_t* __restrict__ mat,
    half* __restrict__ mul,
    const half* __restrict__ scales,
    const uint32_t* __restrict__ zeros,
    const int* __restrict__ g_idx,
    int batch,
    int height,
    int width) {
  int zero_width = width / 8;
  int vec_height = height * 4;
  const int blockwidth2 = BLOCK_KN_SIZE / 2;
  auto b = blockIdx.y * BLOCK_M_SIZE_MAX;
  int b_end = min(BLOCK_M_SIZE_MAX, batch - b);
  auto h = BLOCK_KN_SIZE * blockIdx.z / 8;
  int h_end = min(BLOCK_KN_SIZE / 8, height - h) * 4;
  auto w = BLOCK_KN_SIZE * blockIdx.x + threadIdx.x;

  __shared__ half2 blockvec[BLOCK_M_SIZE_MAX][blockwidth2];
  if (threadIdx.x < h_end) {
    for (int m = 0; m < b_end; ++m) {
      blockvec[m][threadIdx.x] = vec[(m + b) * vec_height + blockIdx.z * BLOCK_KN_SIZE / 2 + threadIdx.x];
    }
  }
```
**EN:** This section implements `gemm_half_q_half_alt_4bit_kernel`, `min`, `blockwidth2`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`gemm_half_q_half_alt_4bit_kernel`、`min`、`blockwidth2`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1271-1298: Device helpers and synchronization
```cpp
  __shared__ half2 deq2[256][8];
  auto val = threadIdx.x / 8;
  auto off = threadIdx.x % 8;
  for (; val < 256; val += BLOCK_KN_SIZE / 8) {
    deq2[val][off] = __halves2half2(__int2half_rn(val & 0xF), __int2half_rn(val >> 4));
  }

  if (blockIdx.z == 0) {
    for (int m = 0; m < b_end; m++)
      mul[(b + m) * width + w] = __int2half_rn(0);
  }
  __syncthreads();

  int i = width * h + w;
  int g_h = h * 8;
  int k = 0;
  int z_w = w / 8;
  int z_mod = (w % 8) * 4;
  half2 res2;
  half res[BLOCK_M_SIZE_MAX] = {};

  unsigned int tmp;
  while (k < h_end) {
    tmp = mat[i];
    half2 scales_tmp[4];
    half2 zeros_tmp[4];
    for (int tmp_k = 0; tmp_k < 4; tmp_k++) {
      int g = g_idx[g_h + (k + tmp_k) * 2];
```
**EN:** This section implements `__halves2half2`, `__int2half_rn`, `__syncthreads`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__halves2half2`、`__int2half_rn`、`__syncthreads`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1299-1325: Control flow and branching
```cpp
      int g2 = g_idx[g_h + (k + tmp_k) * 2 + 1];
      half scale_f = scales[g * width + w];
      half scale_f2 = scales[g2 * width + w];
      half2 scale = __halves2half2(scale_f, scale_f2);
      half2 zero = __halves2half2(
          __hmul(scale_f, __int2half_rn(-((zeros[g * zero_width + z_w] >> z_mod) & 0xF) - 1)),
          __hmul(scale_f2, __int2half_rn(-((zeros[g2 * zero_width + z_w] >> z_mod) & 0xF) - 1)));
      scales_tmp[tmp_k] = scale;
      zeros_tmp[tmp_k] = zero;
    }
    for (int m = 0; m < b_end; m++) {
#ifndef USE_ROCM
      res2 = {};
#else
      res2.x = __half_as_ushort(__float2half(0));
      res2.y = __half_as_ushort(__float2half(0));
#endif
      res2 = __hfma2(__hfma2(deq2[(tmp >> 0) & 0xff][off], scales_tmp[0], zeros_tmp[0]), blockvec[m][k + 0], res2);
      res2 = __hfma2(__hfma2(deq2[(tmp >> 8) & 0xff][off], scales_tmp[1], zeros_tmp[1]), blockvec[m][k + 1], res2);
      res2 = __hfma2(__hfma2(deq2[(tmp >> 16) & 0xff][off], scales_tmp[2], zeros_tmp[2]), blockvec[m][k + 2], res2);
      res2 = __hfma2(__hfma2(deq2[(tmp >> 24) & 0xff][off], scales_tmp[3], zeros_tmp[3]), blockvec[m][k + 3], res2);
#ifndef USE_ROCM
      res[m] = __hadd(res[m], __hadd(res2.x, res2.y));
#else
      res[m] = __hadd(res[m], __hadd(__ushort_as_half(res2.x), __ushort_as_half(res2.y)));
#endif
    }
```
**EN:** This section drives `__halves2half2`, `__half_as_ushort`, `__hfma2` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`__halves2half2`、`__half_as_ushort`、`__hfma2`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1326-1352: Kernel implementation
```cpp
    i += width;
    k += 4;
  }
  for (int m = 0; m < b_end; m++) {
    atomicAdd(&mul[(b + m) * width + w], res[m]);
  }
}

__global__ void gemm_half_q_half_alt_8bit_kernel(
    const half2* __restrict__ vec,
    const uint32_t* __restrict__ mat,
    half* __restrict__ mul,
    const half* __restrict__ scales,
    const uint32_t* __restrict__ zeros,
    const int* __restrict__ g_idx,
    int batch,
    int height,
    int width) {
  int zero_width = width / 4;
  int vec_height = height * 2;
  const int blockwidth2 = BLOCK_KN_SIZE / 2;
  auto b = blockIdx.y * BLOCK_M_SIZE_MAX;
  int b_end = min(BLOCK_M_SIZE_MAX, batch - b);
  auto h = BLOCK_KN_SIZE * blockIdx.z / 4;
  int h_end = min(BLOCK_KN_SIZE / 4, height - h) * 2;
  auto w = BLOCK_KN_SIZE * blockIdx.x + threadIdx.x;
```
**EN:** This section implements `gemm_half_q_half_alt_8bit_kernel`, `atomicAdd`, `min`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`gemm_half_q_half_alt_8bit_kernel`、`atomicAdd`、`min`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1353-1380: Device helpers and synchronization
```cpp
  __shared__ half2 blockvec[BLOCK_M_SIZE_MAX][blockwidth2];
  if (threadIdx.x < h_end) {
    for (int m = 0; m < b_end; ++m) {
      blockvec[m][threadIdx.x] = vec[(m + b) * vec_height + blockIdx.z * BLOCK_KN_SIZE / 2 + threadIdx.x];
    }
  }

  if (blockIdx.z == 0) {
    for (int m = 0; m < b_end; m++)
      mul[(b + m) * width + w] = __int2half_rn(0);
  }
  __syncthreads();

  int i = width * h + w;
  int g_h = h * 4;
  int k = 0;
  int z_w = w / 4;
  int z_mod = (w % 4) * 8;
  half2 res2;
  half res[BLOCK_M_SIZE_MAX] = {};

  unsigned int tmp;
  while (k < h_end) {
    tmp = mat[i];
    half2 scales_tmp[2];
    half2 zeros_tmp[2];
    for (int tmp_k = 0; tmp_k < 2; tmp_k++) {
      int g = g_idx[g_h + (k + tmp_k) * 2];
```
**EN:** This section implements `__int2half_rn`, `__syncthreads`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__int2half_rn`、`__syncthreads`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1381-1407: Control flow and branching
```cpp
      int g2 = g_idx[g_h + (k + tmp_k) * 2 + 1];
      half scale_f = scales[g * width + w];
      half scale_f2 = scales[g2 * width + w];
      half2 scale = __halves2half2(scale_f, scale_f2);
      half2 zero = __halves2half2(
          __hmul(scale_f, __int2half_rn(-((zeros[g * zero_width + z_w] >> z_mod) & 0xff) - 1)),
          __hmul(scale_f2, __int2half_rn(-((zeros[g2 * zero_width + z_w] >> z_mod) & 0xff) - 1)));
      scales_tmp[tmp_k] = scale;
      zeros_tmp[tmp_k] = zero;
    }
    for (int m = 0; m < b_end; m++) {
#ifndef USE_ROCM
      res2 = {};
#else
      res2.x = __half_as_ushort(__float2half(0));
      res2.y = __half_as_ushort(__float2half(0));
#endif
      half2 v12 = __halves2half2(__int2half_rn(tmp & 0xFF), __int2half_rn((tmp >> 8) & 0xFF));
      res2 = __hfma2(__hfma2(v12, scales_tmp[0], zeros_tmp[0]), blockvec[m][k + 0], res2);
      half2 v34 = __halves2half2(__int2half_rn((tmp >> 16) & 0xFF), __int2half_rn((tmp >> 24) & 0xFF));
      res2 = __hfma2(__hfma2(v34, scales_tmp[1], zeros_tmp[1]), blockvec[m][k + 1], res2);
#ifndef USE_ROCM
      res[m] = __hadd(res[m], __hadd(res2.x, res2.y));
#else
      res[m] = __hadd(res[m], __hadd(__ushort_as_half(res2.x), __ushort_as_half(res2.y)));
#endif
    }
```
**EN:** This section drives `__halves2half2`, `__half_as_ushort`, `__hfma2` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`__halves2half2`、`__half_as_ushort`、`__hfma2`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1408-1434: Device helpers and synchronization
```cpp
    i += width;
    k += 2;
  }
  for (int m = 0; m < b_end; m++) {
    atomicAdd(&mul[(b + m) * width + w], res[m]);
  }
}

void gemm_half_q_half_alt(
    const half* a,
    const uint32_t* b_q_weight,
    const uint32_t* b_gptq_qzeros,
    const half* b_gptq_scales,
    const int* b_g_idx,
    half* c,
    int size_m,
    int size_n,
    int size_k,
    int bit) {
  dim3 blockDim, gridDim;
  blockDim.x = BLOCK_KN_SIZE;
  blockDim.y = 1;
  blockDim.z = 1;
  gridDim.x = DIVIDE(size_n, BLOCK_KN_SIZE);
  gridDim.y = DIVIDE(size_m, BLOCK_M_SIZE_MAX);
  gridDim.z = DIVIDE(size_k, BLOCK_KN_SIZE);
```
**EN:** This section implements `gemm_half_q_half_alt`, `atomicAdd`, `DIVIDE`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`gemm_half_q_half_alt`、`atomicAdd`、`DIVIDE`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1435-1462: Types and data layout
```cpp
  auto kernel = gemm_half_q_half_alt_4bit_kernel;
  if (bit == 8) {
    kernel = gemm_half_q_half_alt_8bit_kernel;
  }

  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
  kernel<<<gridDim, blockDim, 0, stream>>>(
      (const half2*)a, b_q_weight, c, b_gptq_scales, b_gptq_qzeros, b_g_idx, size_m, size_k / 32 * bit, size_n);
}

template <class T, int bit>
__global__ void reconstruct_gptq_kernel(
    const uint32_t* __restrict__ w,
    const half* __restrict__ w_scales,
    const uint32_t* __restrict__ w_zeros,
    const int* __restrict__ g_idx,
    const int height,
    const int width,
    const int group,
    half* __restrict__ out) {
  // Start of block

  auto column = BLOCK_KN_SIZE * blockIdx.x + threadIdx.x;
  auto row = blockIdx.y * 32 / bit;
  if (column >= width) return;

  // Views
```
**EN:** This section defines `T`, `reconstruct_gptq_kernel`, `getCurrentCUDAStream`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`T`、`reconstruct_gptq_kernel`、`getCurrentCUDAStream`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 1463-1494: Kernel implementation
```cpp
  MatrixView_half_rw out_(out, height, width);
  MatrixView_half w_scales_(w_scales, group, width);
  T w_zeros_(w_zeros, group, width);

  uint32_t w_read = w[blockIdx.y * width + column];
  half* out_ptr = out_.item_ptr(row, column);

#pragma unroll
  for (int s = 0; s < 32; s += bit) {
    int group = g_idx[row + s / bit];
    half w_scale = w_scales_.item(group, column);
    uint32_t w_zero = w_zeros_.item(group, column) + 1;
    half w_item = __hmul(__int2half_rn((int)((w_read >> s) & ((1 << bit) - 1)) - w_zero), w_scale);
    *out_ptr = w_item;
    out_ptr += out_.width;
  }
}

__global__ void reconstruct_gptq_3bit_kernel(
    const uint32_t* __restrict__ w,
    const half* __restrict__ w_scales,
    const uint32_t* __restrict__ w_zeros,
    const int* __restrict__ g_idx,
    const int height,
    const int width,
    const int group,
    half* __restrict__ out) {
  // Start of block
  auto column = BLOCK_KN_SIZE * blockIdx.x + threadIdx.x;
  auto row = blockIdx.y * 32;
  if (column >= width) return;
```
**EN:** This section implements `reconstruct_gptq_3bit_kernel`, `out_`, `w_scales_`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`reconstruct_gptq_3bit_kernel`、`out_`、`w_scales_`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1495-1522: Device helpers and synchronization
```cpp
  // Views

  MatrixView_half_rw out_(out, height, width);
  MatrixView_half w_scales_(w_scales, group, width);
  MatrixView_q3_row w_zeros_(w_zeros, group, width);

  uint32_t w1 = w[(blockIdx.y * 3) * width + column];
  uint32_t w2 = w[(blockIdx.y * 3 + 1) * width + column];
  uint32_t w3 = w[(blockIdx.y * 3 + 2) * width + column];
  half* out_ptr = out_.item_ptr(row, column);

#pragma unroll
  for (int i = 0; i < 32; i += 1) {
    int group = g_idx[row + i];
    half w_scale = w_scales_.item(group, column);
    uint32_t w_zero = w_zeros_.item(group, column) + 1;
    int w_item;
    if (i == 10) {
      w_item = (w1 >> 30) | ((w2 << 2) & 0x4);
    } else if (i == 21) {
      w_item = (w2 >> 31) | ((w3 << 1) & 0x6);
    } else if (i < 10) {
      w_item = ((w1 >> (i * 3)) & 0x7);
    } else if (i < 21) {
      w_item = ((w2 >> (i * 3 - 32)) & 0x7);
    } else {
      w_item = ((w3 >> (i * 3 - 64)) & 0x7);
    }
```
**EN:** This section implements `out_`, `w_scales_`, `w_zeros_`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`out_`、`w_scales_`、`w_zeros_`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1523-1552: Control flow and branching
```cpp
    *out_ptr = __hmul(__int2half_rn(w_item - w_zero), w_scale);
    out_ptr += out_.width;
  }
}

void reconstruct_gptq(
    const uint32_t* b_q_weight,
    const uint32_t* b_gptq_qzeros,
    const half* b_gptq_scales,
    const int* b_g_idx,
    half* out,
    int height,
    int width,
    int groups,
    int bit) {
  dim3 blockDim, gridDim;
  blockDim.x = BLOCK_KN_SIZE;
  blockDim.y = 1;
  gridDim.y = DIVIDE(height, 32 / bit);
  gridDim.x = DIVIDE(width, BLOCK_KN_SIZE);

  auto kernel = reconstruct_gptq_kernel<MatrixView_q4_row, 4>;
  if (bit == 2) {
    kernel = reconstruct_gptq_kernel<MatrixView_q2_row, 2>;
  } else if (bit == 8) {
    kernel = reconstruct_gptq_kernel<MatrixView_q8_row, 8>;
  } else if (bit == 3) {
    kernel = reconstruct_gptq_3bit_kernel;
    gridDim.y = DIVIDE(height, 32);
  }
```
**EN:** This section drives `reconstruct_gptq`, `__hmul`, `DIVIDE` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`reconstruct_gptq`、`__hmul`、`DIVIDE`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1553-1581: Types and data layout
```cpp

  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
  kernel<<<gridDim, blockDim, 0, stream>>>(
      b_q_weight, b_gptq_scales, b_gptq_qzeros, b_g_idx, height, width, groups, out);
}

void gemm_half_q_half_cuda(
    cublasHandle_t cublas_handle,
    const half* a,
    const uint32_t* b_q_weight,
    const uint32_t* b_gptq_qzeros,
    const half* b_gptq_scales,
    const int* b_g_idx,
    half* c,
    half* temp_dq,
    int size_m,
    int size_n,
    int size_k,
    int groups,
    bool use_shuffle,
    int bit) {
  bool use_reconstruct;
  if (use_shuffle) {
    use_reconstruct = ((bit == 8 && size_m > MAX_Q_GEMM_ROWS_8BIT) || (bit != 8 && size_m > MAX_Q_GEMM_ROWS));
  } else {
    // The 2/3-bit kernels are somehow slower than dequant + gemm baseline, so
    // we disabled them for now.
    use_reconstruct = (bit < 4 || size_m > MAX_ALT_GEMM_ROWS);
  }
```
**EN:** This section defines `gemm_half_q_half_cuda`, `getCurrentCUDAStream`, `stream`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`gemm_half_q_half_cuda`、`getCurrentCUDAStream`、`stream`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 1582-1612: Control flow and branching
```cpp
  if (use_reconstruct) {
    // Reconstruct FP16 matrix, then cuBLAS
    if (use_shuffle) {
      reconstruct_exllama(b_q_weight, b_gptq_qzeros, b_gptq_scales, b_g_idx, temp_dq, size_k, size_n, groups, bit);
    } else {
      reconstruct_gptq(b_q_weight, b_gptq_qzeros, b_gptq_scales, b_g_idx, temp_dq, size_k, size_n, groups, bit);
    }

    const half alpha = __float2half(1.0f);
    const half beta = __float2half(0.0f);
    cublasHgemm(
        cublas_handle,
        CUBLAS_OP_N,
        CUBLAS_OP_N,
        size_n,
        size_m,
        size_k,
        &alpha,
        temp_dq,
        size_n,
        a,
        size_k,
        &beta,
        c,
        size_n);
  } else if (use_shuffle) {
    // Quantized matmul
    int max_chunks = size_m / BLOCK_M_SIZE_MAX;
    int last_chunk = max_chunks * BLOCK_M_SIZE_MAX;
    int last_chunk_size = size_m - last_chunk;
```
**EN:** This section drives `reconstruct_exllama`, `reconstruct_gptq`, `__float2half` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`reconstruct_exllama`、`reconstruct_gptq`、`__float2half`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1613-1643: Control flow and branching
```cpp
    if (max_chunks) {
      gemm_half_q_half_cuda_part(
          a,
          b_q_weight,
          b_gptq_qzeros,
          b_gptq_scales,
          b_g_idx,
          c,
          last_chunk,
          size_n,
          size_k,
          BLOCK_M_SIZE_MAX,
          groups,
          bit);
    }

    if (last_chunk_size) {
      gemm_half_q_half_cuda_part(
          a + last_chunk * size_k,
          b_q_weight,
          b_gptq_qzeros,
          b_gptq_scales,
          b_g_idx,
          c + last_chunk * size_n,
          last_chunk_size,
          size_n,
          size_k,
          last_chunk_size,
          groups,
          bit);
    }
```
**EN:** This section drives `gemm_half_q_half_cuda_part` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`gemm_half_q_half_cuda_part`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1644-1671: Kernel implementation
```cpp
  } else {
    gemm_half_q_half_alt(a, b_q_weight, b_gptq_qzeros, b_gptq_scales, b_g_idx, c, size_m, size_n, size_k, bit);
  }
}

__global__ void shuffle_4bit_kernel(uint32_t* __restrict__ b_q_weight, const int size_k, const int size_n) {
  auto n = blockIdx.x * THREADS_X + threadIdx.x;
  if (n >= size_n) return;
  int k = 0;
  uint32_t* b_ptr = b_q_weight + n;
  while (k < size_k) {
    shuffle_4bit_8(b_ptr, size_n);
    b_ptr += 1 * size_n;
    k += 8;
  }
}

__global__ void shuffle_8bit_kernel(uint32_t* __restrict__ b_q_weight, const int size_k, const int size_n) {
  auto n = blockIdx.x * THREADS_X + threadIdx.x;
  if (n >= size_n) return;
  int k = 0;
  uint32_t* b_ptr = b_q_weight + n;
  while (k < size_k) {
    shuffle_8bit_4(b_ptr, size_n);
    b_ptr += 1 * size_n;
    k += 4;
  }
}
```
**EN:** This section implements `shuffle_4bit_kernel`, `shuffle_8bit_kernel`, `gemm_half_q_half_alt`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`shuffle_4bit_kernel`、`shuffle_8bit_kernel`、`gemm_half_q_half_alt`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1672-1696: Kernel implementation
```cpp

__global__ void shuffle_2bit_kernel(uint32_t* __restrict__ b_q_weight, const int size_k, const int size_n) {
  auto n = blockIdx.x * THREADS_X + threadIdx.x;
  if (n >= size_n) return;
  int k = 0;
  uint32_t* b_ptr = b_q_weight + n;
  while (k < size_k) {
    shuffle_2bit_16(b_ptr, size_n);
    b_ptr += 1 * size_n;
    k += 16;
  }
}

__global__ void shuffle_3bit_kernel(uint32_t* __restrict__ b_q_weight, const int size_k, const int size_n) {
  auto n = blockIdx.x * THREADS_X + threadIdx.x;
  if (n >= size_n) return;
  int k = 0;
  uint32_t* b_ptr = b_q_weight + n;
  while (k < size_k) {
    shuffle_3bit_32(b_ptr, size_n);
    b_ptr += 3 * size_n;
    k += 32;
  }
}
```
**EN:** This section implements `shuffle_2bit_kernel`, `shuffle_3bit_kernel`, `shuffle_2bit_16`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`shuffle_2bit_kernel`、`shuffle_3bit_kernel`、`shuffle_2bit_16`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1697-1724: Kernel implementation
```cpp
__global__ void make_sequential_4bit_kernel(
    const uint32_t* __restrict__ w, uint32_t* __restrict__ w_new, const int* __restrict__ q_perm, const int w_width) {
  const uint64_t* w2 = (uint64_t*)w;
  uint64_t* w_new2 = (uint64_t*)w_new;
  int w2_stride = w_width >> 1;
  auto w2_column = THREADS_X * blockIdx.x + threadIdx.x;
  if (w2_column >= w2_stride) return;
  auto w_new2_row = blockIdx.y;
  int q_perm_idx = w_new2_row << 3;
  uint64_t dst = 0;

#pragma unroll
  for (int i = 0; i < 8; i++) {
    int source_row = q_perm[q_perm_idx++];

    int w2_row = source_row >> 3;
    int w2_subrow = source_row & 0x07;
    int w2_row_shift = w2_subrow << 2;
    int wnew2_row_shift = i << 2;

    uint64_t src = w2[w2_row * w2_stride + w2_column];
    src >>= w2_row_shift;
    src &= 0x0000000f0000000f;
    src <<= wnew2_row_shift;
    dst |= src;
  }
  w_new2[w_new2_row * w2_stride + w2_column] = dst;
}
```
**EN:** This section implements `make_sequential_4bit_kernel`, `w2`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`make_sequential_4bit_kernel`、`w2`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1725-1753: Kernel implementation
```cpp

__global__ void make_sequential_2bit_kernel(
    const uint32_t* __restrict__ w, uint32_t* __restrict__ w_new, const int* __restrict__ q_perm, const int w_width) {
  const uint64_t* w2 = (uint64_t*)w;
  uint64_t* w_new2 = (uint64_t*)w_new;
  int w2_stride = w_width >> 1;
  auto w2_column = THREADS_X * blockIdx.x + threadIdx.x;
  if (w2_column >= w2_stride) return;
  auto w_new2_row = blockIdx.y;
  int q_perm_idx = w_new2_row << 4;
  uint64_t dst = 0;

#pragma unroll
  for (int i = 0; i < 16; i++) {
    int source_row = q_perm[q_perm_idx++];

    int w2_row = source_row >> 4;
    int w2_subrow = source_row & 0x0f;
    int w2_row_shift = w2_subrow << 1;
    int wnew2_row_shift = i << 1;

    uint64_t src = w2[w2_row * w2_stride + w2_column];
    src >>= w2_row_shift;
    src &= 0x0000000300000003;
    src <<= wnew2_row_shift;
    dst |= src;
  }
  w_new2[w_new2_row * w2_stride + w2_column] = dst;
}
```
**EN:** This section implements `make_sequential_2bit_kernel`, `w2`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`make_sequential_2bit_kernel`、`w2`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1754-1783: Kernel implementation
```cpp

__global__ void make_sequential_3bit_kernel(
    const uint32_t* __restrict__ w, uint32_t* __restrict__ w_new, const int* __restrict__ q_perm, const int w_width) {
  auto w_column = THREADS_X * blockIdx.x + threadIdx.x;
  if (w_column >= w_width) return;
  auto w_new_row = blockIdx.y * 3;
  auto q_perm_idx = blockIdx.y << 5;
  uint32_t dst[3] = {0, 0, 0};

#pragma unroll
  for (int i = 0; i < 32; i++) {
    int source_row = q_perm[q_perm_idx++];
    int z_w = (source_row / 32) * 3;
    int z_mod = source_row % 32;
    int z_bit;

    if (z_mod != 10) {
      if (z_mod != 21) {
        z_bit = z_mod;
        if (z_bit > 21) {
          z_bit *= 3;
          z_bit -= 64;
          z_w += 2;
        } else if (z_bit > 10) {
          z_bit *= 3;
          z_bit -= 32;
          z_w += 1;
        } else {
          z_bit *= 3;
        }
```
**EN:** This section implements `make_sequential_3bit_kernel`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`make_sequential_3bit_kernel`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1784-1814: Control flow and branching
```cpp
      } else {
        z_w += 1;
      }
    }

    uint64_t src;
    if (z_mod == 10) {
      src = (w[z_w * w_width + w_column] >> 30) | ((w[(z_w + 1) * w_width + w_column] << 2) & 0x4);
    } else if (z_mod == 21) {
      src = (w[z_w * w_width + w_column] >> 31) | ((w[(z_w + 1) * w_width + w_column] << 1) & 0x6);
    } else {
      src = w[z_w * w_width + w_column];
      src >>= z_bit;
      src &= 0x07;
    }

    z_w = 0;
    if (i != 10) {
      if (i != 21) {
        z_bit = i;
        if (z_bit > 21) {
          z_bit *= 3;
          z_bit -= 64;
          z_w += 2;
        } else if (z_bit > 10) {
          z_bit *= 3;
          z_bit -= 32;
          z_w += 1;
        } else {
          z_bit *= 3;
        }
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1815-1844: Kernel implementation
```cpp
      } else {
        z_w += 1;
      }
    }
    if (i == 10) {
      dst[z_w] |= (src & 0x03) << 30;
      dst[z_w + 1] |= ((src & 0x4) >> 2);
    } else if (i == 21) {
      dst[z_w] |= (src & 0x01) << 31;
      dst[z_w + 1] |= ((src & 0x6) >> 1);
    } else {
      dst[z_w] |= (src << z_bit);
    }
  }
  w_new[w_new_row * w_width + w_column] = dst[0];
  w_new[(w_new_row + 1) * w_width + w_column] = dst[1];
  w_new[(w_new_row + 2) * w_width + w_column] = dst[2];
}

__global__ void make_sequential_8bit_kernel(
    const uint32_t* __restrict__ w, uint32_t* __restrict__ w_new, const int* __restrict__ q_perm, const int w_width) {
  const uint64_t* w2 = (uint64_t*)w;
  uint64_t* w_new2 = (uint64_t*)w_new;
  int w2_stride = w_width >> 1;
  auto w2_column = THREADS_X * blockIdx.x + threadIdx.x;
  if (w2_column >= w2_stride) return;
  auto w_new2_row = blockIdx.y;
  int q_perm_idx = w_new2_row << 2;
  uint64_t dst = 0;
```
**EN:** This section implements `make_sequential_8bit_kernel`, `w2`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`make_sequential_8bit_kernel`、`w2`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1845-1873: Control flow and branching
```cpp
#pragma unroll
  for (int i = 0; i < 4; i++) {
    int source_row = q_perm[q_perm_idx++];

    int w2_row = source_row >> 2;
    int w2_subrow = source_row & 0x03;
    int w2_row_shift = w2_subrow << 3;
    int wnew2_row_shift = i << 3;

    uint64_t src = w2[w2_row * w2_stride + w2_column];
    src >>= w2_row_shift;
    src &= 0x000000ff000000ff;
    src <<= wnew2_row_shift;
    dst |= src;
  }
  w_new2[w_new2_row * w2_stride + w2_column] = dst;
}

void shuffle_exllama_weight(uint32_t* q_weight, int* q_perm, int height, int width, int bit) {
  if (q_perm) {
    uint32_t* new_qweight = NULL;
    cudaMalloc(&new_qweight, height / 32 * bit * width * sizeof(uint32_t));

    dim3 blockDim, gridDim;
    blockDim.x = THREADS_X;
    blockDim.y = 1;
    gridDim.x = DIVIDE(width, THREADS_X);
    gridDim.y = height / 32 * bit;
```
**EN:** This section drives `shuffle_exllama_weight`, `cudaMalloc`, `DIVIDE` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`shuffle_exllama_weight`、`cudaMalloc`、`DIVIDE`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 1874-1903: Kernel implementation
```cpp
    auto kernel = make_sequential_4bit_kernel;
    if (bit == 2) {
      kernel = make_sequential_2bit_kernel;
    } else if (bit == 3) {
      kernel = make_sequential_3bit_kernel;
      gridDim.y = height / 32;
    } else if (bit == 8) {
      kernel = make_sequential_8bit_kernel;
    }
    const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
    kernel<<<gridDim, blockDim, 0, stream>>>(q_weight, new_qweight, q_perm, width);
    // Replace qweights
    cudaMemcpyAsync(q_weight, new_qweight, height / 32 * bit * width * sizeof(uint32_t), cudaMemcpyDeviceToDevice);
    // Cleanup
    cudaDeviceSynchronize();
    cudaFree(new_qweight);
  }
  dim3 blockDim, gridDim;
  blockDim.x = THREADS_X;
  blockDim.y = 1;
  gridDim.x = DIVIDE(width, THREADS_X);
  gridDim.y = 1;
  auto shuffle_kernel = shuffle_4bit_kernel;
  if (bit == 2) {
    shuffle_kernel = shuffle_2bit_kernel;
  } else if (bit == 3) {
    shuffle_kernel = shuffle_3bit_kernel;
  } else if (bit == 8) {
    shuffle_kernel = shuffle_8bit_kernel;
  }
```
**EN:** This section implements `getCurrentCUDAStream`, `cudaMemcpyAsync`, `cudaDeviceSynchronize`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`getCurrentCUDAStream`、`cudaMemcpyAsync`、`cudaDeviceSynchronize`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1904-1931: Kernel implementation
```cpp
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();
  shuffle_kernel<<<gridDim, blockDim, 0, stream>>>(q_weight, height, width);
}

}  // namespace gptq
}  // namespace sglang

torch::Tensor gptq_gemm(
    torch::Tensor a,
    torch::Tensor b_q_weight,
    torch::Tensor b_gptq_qzeros,
    torch::Tensor b_gptq_scales,
    torch::Tensor b_g_idx,
    bool use_shuffle,
    int64_t bit) {
  const at::cuda::OptionalCUDAGuard device_guard(device_of(a));
  auto options = torch::TensorOptions().dtype(a.dtype()).device(a.device());
  at::Tensor c = torch::empty({a.size(0), b_q_weight.size(1)}, options);
  at::Tensor temp_dq = torch::empty({b_q_weight.size(0) * 32 / bit, b_q_weight.size(1)}, options);

  sglang::gptq::gemm_half_q_half_cuda(
      at::cuda::getCurrentCUDABlasHandle(),
      (const half*)a.data_ptr(),
      (const uint32_t*)b_q_weight.data_ptr(),
      (const uint32_t*)b_gptq_qzeros.data_ptr(),
      (const half*)b_gptq_scales.data_ptr(),
      b_g_idx.device().is_meta() ? NULL : (const int*)b_g_idx.data_ptr(),
      (half*)c.data_ptr(),
```
**EN:** This section implements `gptq_gemm`, `getCurrentCUDAStream`, `device_guard`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`gptq_gemm`、`getCurrentCUDAStream`、`device_guard`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1932-1950: Runtime integration and dispatch
```cpp
      (half*)temp_dq.data_ptr(),
      c.size(0),              // m
      c.size(1),              // n
      a.size(1),              // k
      b_gptq_qzeros.size(0),  // group number
      use_shuffle,
      bit);
  return c;
}

void gptq_shuffle(torch::Tensor q_weight, torch::Tensor q_perm, int64_t bit) {
  const at::cuda::OptionalCUDAGuard device_guard(device_of(q_weight));
  sglang::gptq::shuffle_exllama_weight(
      (uint32_t*)q_weight.data_ptr(),
      q_perm.device().is_meta() || q_perm.numel() == 0 ? NULL : (int*)q_perm.data_ptr(),
      q_weight.size(0) * 32 / bit,
      q_weight.size(1),
      bit);
}
```
**EN:** This section uses `gptq_shuffle`, `data_ptr`, `device_guard` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`gptq_shuffle`、`data_ptr`、`device_guard`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `compat.cuh`, `matrix_view.cuh`, `qdq_2.cuh`, `qdq_3.cuh`, `qdq_4.cuh`, `qdq_8.cuh`
- **External headers / 外部头文件**: `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDAGuard.h`, `cuda_fp16.h`, `cuda_runtime.h`, `torch/all.h`, `cstdint`, `cstdio`, `hipblas/hipblas.h`
- **Path context / 路径上下文**: gemm / gptq / gptq_kernel.cu
