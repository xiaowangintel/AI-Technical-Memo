# moe.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/quantization/gguf/moe.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements quantization, dequantization, or low-bit arithmetic utilities used by inference kernels. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 实现推理内核使用的量化、反量化或低比特算术工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Device helpers and synchronization
```cpp
// copied from
// https://github.com/vllm-project/vllm/blob/4492e3a55428e161ca8db381edc28263e5da4c8d/csrc/quantization/gguf/moe.cuh
#include <cstdint>

/* Adapted from ./csrc/quantization/gguf/mmq.cuh
 */
template <
    typename scalar_t,
    int qk,
    int qr,
    int qi,
    bool need_sum,
    typename block_q_t,
    int mmq_x,
    int mmq_y,
    int nwarps,
    allocate_tiles_cuda_t allocate_tiles,
    load_tiles_cuda_t load_tiles,
    int vdr,
    vec_dot_q_mul_mat_cuda_t vec_dot>
static __device__ __forceinline__ void moe_q(
    const void* __restrict__ vx,
    const void* __restrict__ vy,
    scalar_t* __restrict__ dst,
    const int* __restrict__ sorted_token_ids,
    const int* __restrict__ expert_ids,
    const int* __restrict__ num_tokens_post_padded,
    const int exp_stride,
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 29-57: Device helpers and synchronization
```cpp
    const int ncols_x,
    const int nrows_x,
    const int ncols_y,
    const int nrows_y,
    const int nrows_dst,
    const int top_k) {
  const int blocks_per_row_x = ncols_x / qk;
  const int blocks_per_col_y = nrows_y / QK8_1;
  const int blocks_per_warp = WARP_SIZE_GGUF / qi;

  const int ncols_dst = ncols_y * top_k;

  const auto row_dst_0 = blockIdx.x * mmq_y;
  const int& row_x_0 = row_dst_0;

  const auto col_dst_0 = blockIdx.y * mmq_x;

  int token_offs[mmq_x / nwarps];
  for (int i = 0; i < mmq_x; i += nwarps) {
    token_offs[i / nwarps] = sorted_token_ids[col_dst_0 + threadIdx.y + i];
  }

  const int exp_idx = expert_ids[blockIdx.y];
  if (exp_idx > 255 || exp_idx < 0) return;
  if (blockIdx.y * mmq_x > num_tokens_post_padded[0]) return;

  const block_q_t* x = (const block_q_t*)((char*)vx + exp_idx * exp_stride);
  const block_q8_1* y = (const block_q8_1*)(vy);
```
**EN:** This section implements `blocks_per_row_x`, `blocks_per_col_y`, `blocks_per_warp`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`blocks_per_row_x`、`blocks_per_col_y`、`blocks_per_warp`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 58-87: Device helpers and synchronization
```cpp
  int* tile_x_ql = nullptr;
  half2* tile_x_dm = nullptr;
  int* tile_x_qh = nullptr;
  int* tile_x_sc = nullptr;

  allocate_tiles(&tile_x_ql, &tile_x_dm, &tile_x_qh, &tile_x_sc);

  __shared__ int tile_y_qs[mmq_x * WARP_SIZE_GGUF];
  __shared__ half2 tile_y_ds[mmq_x * WARP_SIZE_GGUF / QI8_1];

  float sum[mmq_y / WARP_SIZE_GGUF][mmq_x / nwarps] = {{0.0f}};

  for (int ib0 = 0; ib0 < blocks_per_row_x; ib0 += blocks_per_warp) {
    load_tiles(
        x + row_x_0 * blocks_per_row_x + ib0,
        tile_x_ql,
        tile_x_dm,
        tile_x_qh,
        tile_x_sc,
        threadIdx.y,
        nrows_x - row_x_0 - 1,
        threadIdx.x,
        blocks_per_row_x);

    const int n_per_r = ((qk * blocks_per_warp) / qr);
#pragma unroll
    for (int ir = 0; ir < qr && ib0 * qk + ir * n_per_r < ncols_x; ++ir) {
      const auto kqs = ir * WARP_SIZE_GGUF + threadIdx.x;
      const int kbxd = kqs / QI8_1;
```
**EN:** This section implements `allocate_tiles`, `load_tiles`, `n_per_r`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`allocate_tiles`、`load_tiles`、`n_per_r`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 88-115: Device helpers and synchronization
```cpp
#pragma unroll
      for (int i = 0; i < mmq_x; i += nwarps) {
        const int col_y_eff = token_offs[i / nwarps] / top_k;
        const int block_x = ib0 * (qk / QK8_1) + kbxd;
        if (col_y_eff < ncols_y && block_x < blocks_per_col_y) {
          const block_q8_1* by0 = &y[col_y_eff * blocks_per_col_y + block_x];
          const int index_y = (threadIdx.y + i) * WARP_SIZE_GGUF + kqs % WARP_SIZE_GGUF;
          tile_y_qs[index_y] = get_int_from_int8_aligned(by0->qs, threadIdx.x % QI8_1);
        }
      }

      if (threadIdx.x < n_per_r / QK8_1) {
        const auto kby = threadIdx.x % (WARP_SIZE_GGUF / QI8_1);
        const int col_y_eff = token_offs[threadIdx.y] / top_k;
        const int block_x = ib0 * (qk / QK8_1) + ir * (WARP_SIZE_GGUF / QI8_1) + kby;

        if (col_y_eff < ncols_y && block_x < blocks_per_col_y) {
          const half2* dsi_src = &y[col_y_eff * blocks_per_col_y + block_x].ds;
          half2* dsi_dst = &tile_y_ds[threadIdx.y * (WARP_SIZE_GGUF / QI8_1) + kby];

          if (need_sum) {
            *dsi_dst = *dsi_src;
          } else {
            float* dfi_dst = (float*)dsi_dst;
            *dfi_dst = __low2float(*dsi_src);
          }
        }
      }
```
**EN:** This section implements `get_int_from_int8_aligned`, `__low2float`, `col_y_eff`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`get_int_from_int8_aligned`、`__low2float`、`col_y_eff`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 116-145: Device helpers and synchronization
```cpp
      __syncthreads();

      // #pragma unroll // unrolling this loop causes too much register pressure
      for (int k = ir * WARP_SIZE_GGUF / qr; k < (ir + 1) * WARP_SIZE_GGUF / qr; k += vdr) {
#pragma unroll
        for (int j = 0; j < mmq_x; j += nwarps) {
#pragma unroll
          for (int i = 0; i < mmq_y; i += WARP_SIZE_GGUF) {
            sum[i / WARP_SIZE_GGUF][j / nwarps] += vec_dot(
                tile_x_ql, tile_x_dm, tile_x_qh, tile_x_sc, tile_y_qs, tile_y_ds, threadIdx.x + i, threadIdx.y + j, k);
          }
        }
      }
      __syncthreads();
    }
  }

#pragma unroll
  for (int j = 0; j < mmq_x; j += nwarps) {
    const int col_dst = token_offs[j / nwarps];
    if (col_dst >= ncols_dst) {
      return;
    }

#pragma unroll
    for (int i = 0; i < mmq_y; i += WARP_SIZE_GGUF) {
      const auto row_dst = row_dst_0 + threadIdx.x + i;
      if (row_dst >= nrows_dst) {
        continue;
      }
```
**EN:** This section implements `__syncthreads`, `vec_dot`, `col_dst`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__syncthreads`、`vec_dot`、`col_dst`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 146-173: Kernel implementation
```cpp
      dst[col_dst * nrows_dst + row_dst] = sum[i / WARP_SIZE_GGUF][j / nwarps];
    }
  }
}

#if defined(USE_ROCM)
#define MOE_X_Q4_0 8
#define MOE_Y_Q4_0 128
#define NWARPS_Q4_0 8
#else
#define MOE_X_Q4_0 4
#define MOE_Y_Q4_0 32
#define NWARPS_Q4_0 4
#endif

template <typename scalar_t, bool need_check>
static __global__ void
#if defined(USE_ROCM)
__launch_bounds__(WARP_SIZE_GGUF* NWARPS_Q4_0, 2)
#endif
    moe_q4_0(
        const void* __restrict__ vx,
        const void* __restrict__ vy,
        scalar_t* __restrict__ dst,
        const int* sorted_token_ids,
        const int* expert_ids,
        const int* num_tokens_post_padded,
        const int exp_stride,
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 174-201: Device helpers and synchronization
```cpp
        const int ncols_x,
        const int nrows_x,
        const int ncols_y,
        const int nrows_y,
        const int nrows_dst,
        const int top_k) {
  const int mmq_x = MOE_X_Q4_0;
  const int mmq_y = MOE_Y_Q4_0;
  const int nwarps = NWARPS_Q4_0;

  moe_q<
      scalar_t,
      QK4_0,
      QR4_0,
      QI4_0,
      true,
      block_q4_0,
      mmq_x,
      mmq_y,
      nwarps,
      allocate_tiles_q4_0<mmq_y>,
      load_tiles_q4_0<mmq_y, nwarps, need_check>,
      VDR_Q4_0_Q8_1_MMQ,
      vec_dot_q4_0_q8_1_mul_mat>(
      vx,
      vy,
      dst,
      sorted_token_ids,
```
**EN:** This section implements `mmq_x`, `mmq_y`, `nwarps`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`mmq_x`、`mmq_y`、`nwarps`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 202-233: Device helpers and synchronization
```cpp
      expert_ids,
      num_tokens_post_padded,
      exp_stride,
      ncols_x,
      nrows_x,
      ncols_y,
      nrows_y,
      nrows_dst,
      top_k);
}

template <typename scalar_t>
static void ggml_moe_q4_0_q8_1_cuda(
    const void* inp,
    const void* w,
    scalar_t* dst,
    const int* sorted_token_ids,
    const int* expert_ids,
    const int* num_tokens_post_padded,
    const int exp_stride,
    const int ncols_x,
    const int nrows_x,
    const int ncols_y,
    const int nrows_y,
    const int nrows_dst,
    const int top_k,
    const int tokens_post_padded,
    cudaStream_t stream) {
  int mmq_x = MOE_X_Q4_0;
  int mmq_y = MOE_Y_Q4_0;
  int nwarps = NWARPS_Q4_0;
```
**EN:** This section implements `ggml_moe_q4_0_q8_1_cuda`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ggml_moe_q4_0_q8_1_cuda`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 234-261: Kernel implementation
```cpp
  const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
  const int block_num_y = (tokens_post_padded) / mmq_x;
  const dim3 block_nums(block_num_x, block_num_y, 1);
  const dim3 block_dims(WARP_SIZE_GGUF, nwarps, 1);

  if (nrows_x % mmq_y == 0) {
    constexpr bool need_check = false;
    moe_q4_0<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  } else {
    constexpr bool need_check = true;
    moe_q4_0<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
```
**EN:** This section implements `block_nums`, `block_dims`, `block_num_x`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`block_nums`、`block_dims`、`block_num_x`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 262-283: Device helpers and synchronization
```cpp
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  }
}

#if defined(USE_ROCM)
#define MOE_X_Q4_1 8
#define MOE_Y_Q4_1 128
#define NWARPS_Q4_1 8
#else
#define MOE_X_Q4_1 4
#define MOE_Y_Q4_1 32
#define NWARPS_Q4_1 4
#endif
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 284-306: Kernel implementation
```cpp
template <typename scalar_t, bool need_check>
static __global__ void
#if defined(USE_ROCM)
__launch_bounds__(WARP_SIZE_GGUF* NWARPS_Q4_1, 2)
#endif
    moe_q4_1(
        const void* __restrict__ vx,
        const void* __restrict__ vy,
        scalar_t* __restrict__ dst,
        const int* sorted_token_ids,
        const int* expert_ids,
        const int* num_tokens_post_padded,
        const int exp_stride,
        const int ncols_x,
        const int nrows_x,
        const int ncols_y,
        const int nrows_y,
        const int nrows_dst,
        const int top_k) {
  const int mmq_x = MOE_X_Q4_1;
  const int mmq_y = MOE_Y_Q4_1;
  const int nwarps = NWARPS_Q4_1;
```
**EN:** This section implements `defined`, `mmq_x`, `mmq_y`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`defined`、`mmq_x`、`mmq_y`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 307-334: Device helpers and synchronization
```cpp
  moe_q<
      scalar_t,
      QK4_1,
      QR4_1,
      QI4_1,
      true,
      block_q4_1,
      mmq_x,
      mmq_y,
      nwarps,
      allocate_tiles_q4_1<mmq_y>,
      load_tiles_q4_1<mmq_y, nwarps, need_check>,
      VDR_Q4_1_Q8_1_MMQ,
      vec_dot_q4_1_q8_1_mul_mat>(
      vx,
      vy,
      dst,
      sorted_token_ids,
      expert_ids,
      num_tokens_post_padded,
      exp_stride,
      ncols_x,
      nrows_x,
      ncols_y,
      nrows_y,
      nrows_dst,
      top_k);
}
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 335-361: Device helpers and synchronization
```cpp

template <typename scalar_t>
static void ggml_moe_q4_1_q8_1_cuda(
    const void* inp,
    const void* w,
    scalar_t* dst,
    const int* sorted_token_ids,
    const int* expert_ids,
    const int* num_tokens_post_padded,
    const int exp_stride,
    const int ncols_x,
    const int nrows_x,
    const int ncols_y,
    const int nrows_y,
    const int nrows_dst,
    const int top_k,
    const int tokens_post_padded,
    cudaStream_t stream) {
  int mmq_x = MOE_X_Q4_1;
  int mmq_y = MOE_Y_Q4_1;
  int nwarps = NWARPS_Q4_1;

  const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
  const int block_num_y = (tokens_post_padded) / mmq_x;
  const dim3 block_nums(block_num_x, block_num_y, 1);
  const dim3 block_dims(WARP_SIZE_GGUF, nwarps, 1);
```
**EN:** This section implements `ggml_moe_q4_1_q8_1_cuda`, `block_nums`, `block_dims`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ggml_moe_q4_1_q8_1_cuda`、`block_nums`、`block_dims`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 362-394: Kernel implementation
```cpp
  if (nrows_x % mmq_y == 0) {
    constexpr bool need_check = false;
    moe_q4_1<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  } else {
    constexpr bool need_check = true;
    moe_q4_1<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  }
```
**EN:** This section implements `need_check`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`need_check`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 395-422: Kernel implementation
```cpp
}

#if defined(USE_ROCM)
#define MOE_X_Q5_0 8
#define MOE_Y_Q5_0 128
#define NWARPS_Q5_0 8
#else
#define MOE_X_Q5_0 4
#define MOE_Y_Q5_0 32
#define NWARPS_Q5_0 4
#endif

template <typename scalar_t, bool need_check>
static __global__ void
#if defined(USE_ROCM)
__launch_bounds__(WARP_SIZE_GGUF* NWARPS_Q5_0, 2)
#endif
    moe_q5_0(
        const void* __restrict__ vx,
        const void* __restrict__ vy,
        scalar_t* __restrict__ dst,
        const int* sorted_token_ids,
        const int* expert_ids,
        const int* num_tokens_post_padded,
        const int exp_stride,
        const int ncols_x,
        const int nrows_x,
        const int ncols_y,
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 423-450: Device helpers and synchronization
```cpp
        const int nrows_y,
        const int nrows_dst,
        const int top_k) {
  const int mmq_x = MOE_X_Q5_0;
  const int mmq_y = MOE_Y_Q5_0;
  const int nwarps = NWARPS_Q5_0;

  moe_q<
      scalar_t,
      QK5_0,
      QR5_0,
      QI5_0,
      false,
      block_q5_0,
      mmq_x,
      mmq_y,
      nwarps,
      allocate_tiles_q5_0<mmq_y>,
      load_tiles_q5_0<mmq_y, nwarps, need_check>,
      VDR_Q5_0_Q8_1_MMQ,
      vec_dot_q5_0_q8_1_mul_mat>(
      vx,
      vy,
      dst,
      sorted_token_ids,
      expert_ids,
      num_tokens_post_padded,
      exp_stride,
```
**EN:** This section implements `mmq_x`, `mmq_y`, `nwarps`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`mmq_x`、`mmq_y`、`nwarps`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 451-479: Device helpers and synchronization
```cpp
      ncols_x,
      nrows_x,
      ncols_y,
      nrows_y,
      nrows_dst,
      top_k);
}

template <typename scalar_t>
static void ggml_moe_q5_0_q8_1_cuda(
    const void* inp,
    const void* w,
    scalar_t* dst,
    const int* sorted_token_ids,
    const int* expert_ids,
    const int* num_tokens_post_padded,
    const int exp_stride,
    const int ncols_x,
    const int nrows_x,
    const int ncols_y,
    const int nrows_y,
    const int nrows_dst,
    const int top_k,
    const int tokens_post_padded,
    cudaStream_t stream) {
  const int mmq_x = MOE_X_Q5_0;
  const int mmq_y = MOE_Y_Q5_0;
  const int nwarps = NWARPS_Q5_0;
```
**EN:** This section implements `ggml_moe_q5_0_q8_1_cuda`, `mmq_x`, `mmq_y`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ggml_moe_q5_0_q8_1_cuda`、`mmq_x`、`mmq_y`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 480-507: Kernel implementation
```cpp
  const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
  const int block_num_y = (tokens_post_padded) / mmq_x;
  const dim3 block_nums(block_num_x, block_num_y, 1);
  const dim3 block_dims(WARP_SIZE_GGUF, nwarps, 1);

  if (nrows_x % mmq_y == 0) {
    constexpr bool need_check = false;
    moe_q5_0<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  } else {
    constexpr bool need_check = true;
    moe_q5_0<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
```
**EN:** This section implements `block_nums`, `block_dims`, `block_num_x`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`block_nums`、`block_dims`、`block_num_x`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 508-529: Device helpers and synchronization
```cpp
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  }
}

#if defined(USE_ROCM)
#define MOE_X_Q5_1 8
#define MOE_Y_Q5_1 128
#define NWARPS_Q5_1 8
#else
#define MOE_X_Q5_1 4
#define MOE_Y_Q5_1 32
#define NWARPS_Q5_1 4
#endif
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 530-552: Kernel implementation
```cpp
template <typename scalar_t, bool need_check>
static __global__ void
#if defined(USE_ROCM)
__launch_bounds__(WARP_SIZE_GGUF* NWARPS_Q5_1, 2)
#endif
    moe_q5_1(
        const void* __restrict__ vx,
        const void* __restrict__ vy,
        scalar_t* __restrict__ dst,
        const int* sorted_token_ids,
        const int* expert_ids,
        const int* num_tokens_post_padded,
        const int exp_stride,
        const int ncols_x,
        const int nrows_x,
        const int ncols_y,
        const int nrows_y,
        const int nrows_dst,
        const int top_k) {
  const int mmq_x = MOE_X_Q5_1;
  const int mmq_y = MOE_Y_Q5_1;
  const int nwarps = NWARPS_Q5_1;
```
**EN:** This section implements `defined`, `mmq_x`, `mmq_y`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`defined`、`mmq_x`、`mmq_y`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 553-580: Device helpers and synchronization
```cpp
  moe_q<
      scalar_t,
      QK5_1,
      QR5_1,
      QI5_1,
      true,
      block_q5_1,
      mmq_x,
      mmq_y,
      nwarps,
      allocate_tiles_q5_1<mmq_y>,
      load_tiles_q5_1<mmq_y, nwarps, need_check>,
      VDR_Q5_1_Q8_1_MMQ,
      vec_dot_q5_1_q8_1_mul_mat>(
      vx,
      vy,
      dst,
      sorted_token_ids,
      expert_ids,
      num_tokens_post_padded,
      exp_stride,
      ncols_x,
      nrows_x,
      ncols_y,
      nrows_y,
      nrows_dst,
      top_k);
}
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 581-607: Device helpers and synchronization
```cpp

template <typename scalar_t>
static void ggml_moe_q5_1_q8_1_cuda(
    const void* inp,
    const void* w,
    scalar_t* dst,
    const int* sorted_token_ids,
    const int* expert_ids,
    const int* num_tokens_post_padded,
    const int exp_stride,
    const int ncols_x,
    const int nrows_x,
    const int ncols_y,
    const int nrows_y,
    const int nrows_dst,
    const int top_k,
    const int tokens_post_padded,
    cudaStream_t stream) {
  const int mmq_x = MOE_X_Q5_1;
  const int mmq_y = MOE_Y_Q5_1;
  const int nwarps = NWARPS_Q5_1;

  const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
  const int block_num_y = (tokens_post_padded) / mmq_x;
  const dim3 block_nums(block_num_x, block_num_y, 1);
  const dim3 block_dims(WARP_SIZE_GGUF, nwarps, 1);
```
**EN:** This section implements `ggml_moe_q5_1_q8_1_cuda`, `block_nums`, `block_dims`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ggml_moe_q5_1_q8_1_cuda`、`block_nums`、`block_dims`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 608-640: Kernel implementation
```cpp
  if (nrows_x % mmq_y == 0) {
    constexpr bool need_check = false;
    moe_q5_1<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  } else {
    constexpr bool need_check = true;
    moe_q5_1<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  }
```
**EN:** This section implements `need_check`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`need_check`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 641-668: Kernel implementation
```cpp
}

#if defined(USE_ROCM)
#define MOE_X_Q8_0 8
#define MOE_Y_Q8_0 128
#define NWARPS_Q8_0 8
#else
#define MOE_X_Q8_0 4
#define MOE_Y_Q8_0 32
#define NWARPS_Q8_0 4
#endif

template <typename scalar_t, bool need_check>
static __global__ void
#if defined(USE_ROCM)
__launch_bounds__(WARP_SIZE_GGUF* NWARPS_Q8_0, 2)
#endif
    moe_q8_0(
        const void* __restrict__ vx,
        const void* __restrict__ vy,
        scalar_t* __restrict__ dst,
        const int* sorted_token_ids,
        const int* expert_ids,
        const int* num_tokens_post_padded,
        const int exp_stride,
        const int ncols_x,
        const int nrows_x,
        const int ncols_y,
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 669-696: Device helpers and synchronization
```cpp
        const int nrows_y,
        const int nrows_dst,
        const int top_k) {
  const int mmq_x = MOE_X_Q8_0;
  const int mmq_y = MOE_Y_Q8_0;
  const int nwarps = NWARPS_Q8_0;

  moe_q<
      scalar_t,
      QK8_0,
      QR8_0,
      QI8_0,
      false,
      block_q8_0,
      mmq_x,
      mmq_y,
      nwarps,
      allocate_tiles_q8_0<mmq_y>,
      load_tiles_q8_0<mmq_y, nwarps, need_check>,
      VDR_Q8_0_Q8_1_MMQ,
      vec_dot_q8_0_q8_1_mul_mat>(
      vx,
      vy,
      dst,
      sorted_token_ids,
      expert_ids,
      num_tokens_post_padded,
      exp_stride,
```
**EN:** This section implements `mmq_x`, `mmq_y`, `nwarps`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`mmq_x`、`mmq_y`、`nwarps`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 697-725: Device helpers and synchronization
```cpp
      ncols_x,
      nrows_x,
      ncols_y,
      nrows_y,
      nrows_dst,
      top_k);
}

template <typename scalar_t>
static void ggml_moe_q8_0_q8_1_cuda(
    const void* inp,
    const void* w,
    scalar_t* dst,
    const int* sorted_token_ids,
    const int* expert_ids,
    const int* num_tokens_post_padded,
    const int exp_stride,
    const int ncols_x,
    const int nrows_x,
    const int ncols_y,
    const int nrows_y,
    const int nrows_dst,
    const int top_k,
    const int tokens_post_padded,
    cudaStream_t stream) {
  const int mmq_x = MOE_X_Q8_0;
  const int mmq_y = MOE_Y_Q8_0;
  const int nwarps = NWARPS_Q8_0;
```
**EN:** This section implements `ggml_moe_q8_0_q8_1_cuda`, `mmq_x`, `mmq_y`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ggml_moe_q8_0_q8_1_cuda`、`mmq_x`、`mmq_y`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 726-753: Kernel implementation
```cpp
  const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
  const int block_num_y = (tokens_post_padded) / mmq_x;
  const dim3 block_nums(block_num_x, block_num_y, 1);
  const dim3 block_dims(WARP_SIZE_GGUF, nwarps, 1);

  if (nrows_x % mmq_y == 0) {
    constexpr bool need_check = false;
    moe_q8_0<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  } else {
    constexpr bool need_check = true;
    moe_q8_0<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
```
**EN:** This section implements `block_nums`, `block_dims`, `block_num_x`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`block_nums`、`block_dims`、`block_num_x`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 754-775: Device helpers and synchronization
```cpp
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  }
}

#if defined(USE_ROCM)
#define MOE_X_Q2_K 8
#define MOE_Y_Q2_K 128
#define NWARPS_Q2_K 8
#else
#define MOE_X_Q2_K 4
#define MOE_Y_Q2_K 32
#define NWARPS_Q2_K 4
#endif
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 776-798: Kernel implementation
```cpp
template <typename scalar_t, bool need_check>
static __global__ void
#if defined(USE_ROCM)
__launch_bounds__(WARP_SIZE_GGUF* NWARPS_Q2_K, 2)
#endif
    moe_q2_K(
        const void* __restrict__ vx,
        const void* __restrict__ vy,
        scalar_t* __restrict__ dst,
        const int* sorted_token_ids,
        const int* expert_ids,
        const int* num_tokens_post_padded,
        const int exp_stride,
        const int ncols_x,
        const int nrows_x,
        const int ncols_y,
        const int nrows_y,
        const int nrows_dst,
        const int top_k) {
  const int mmq_x = MOE_X_Q2_K;
  const int mmq_y = MOE_Y_Q2_K;
  const int nwarps = NWARPS_Q2_K;
```
**EN:** This section implements `defined`, `mmq_x`, `mmq_y`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`defined`、`mmq_x`、`mmq_y`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 799-826: Device helpers and synchronization
```cpp
  moe_q<
      scalar_t,
      QK_K,
      QR2_K,
      QI2_K,
      false,
      block_q2_K,
      mmq_x,
      mmq_y,
      nwarps,
      allocate_tiles_q2_K<mmq_y>,
      load_tiles_q2_K<mmq_y, nwarps, need_check>,
      VDR_Q2_K_Q8_1_MMQ,
      vec_dot_q2_K_q8_1_mul_mat>(
      vx,
      vy,
      dst,
      sorted_token_ids,
      expert_ids,
      num_tokens_post_padded,
      exp_stride,
      ncols_x,
      nrows_x,
      ncols_y,
      nrows_y,
      nrows_dst,
      top_k);
}
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 827-853: Device helpers and synchronization
```cpp

template <typename scalar_t>
static void ggml_moe_q2_K_q8_1_cuda(
    const void* inp,
    const void* w,
    scalar_t* dst,
    const int* sorted_token_ids,
    const int* expert_ids,
    const int* num_tokens_post_padded,
    const int exp_stride,
    const int ncols_x,
    const int nrows_x,
    const int ncols_y,
    const int nrows_y,
    const int nrows_dst,
    const int top_k,
    const int tokens_post_padded,
    cudaStream_t stream) {
  const int mmq_x = MOE_X_Q2_K;
  const int mmq_y = MOE_Y_Q2_K;
  const int nwarps = NWARPS_Q2_K;

  const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
  const int block_num_y = (tokens_post_padded) / mmq_x;
  const dim3 block_nums(block_num_x, block_num_y, 1);
  const dim3 block_dims(WARP_SIZE_GGUF, nwarps, 1);
```
**EN:** This section implements `ggml_moe_q2_K_q8_1_cuda`, `block_nums`, `block_dims`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ggml_moe_q2_K_q8_1_cuda`、`block_nums`、`block_dims`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 854-886: Kernel implementation
```cpp
  if (nrows_x % mmq_y == 0) {
    constexpr bool need_check = false;
    moe_q2_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  } else {
    constexpr bool need_check = true;
    moe_q2_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  }
```
**EN:** This section implements `need_check`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`need_check`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 887-918: Kernel implementation
```cpp
}

#if defined(USE_ROCM)
#define MOE_X_Q3_K 8
#define MOE_Y_Q3_K 128
#define NWARPS_Q3_K 8
#else
#define MOE_X_Q3_K 4
#define MOE_Y_Q3_K 32
#define NWARPS_Q3_K 4
#endif

template <typename scalar_t, bool need_check>
static __global__ void
#if defined(USE_ROCM)
__launch_bounds__(WARP_SIZE_GGUF* NWARPS_Q3_K, 2)
#endif
    moe_q3_K(
        const void* __restrict__ vx,
        const void* __restrict__ vy,
        scalar_t* __restrict__ dst,
        const int* sorted_token_ids,
        const int* expert_ids,
        const int* num_tokens_post_padded,
        const int exp_stride,
        const int ncols_x,
        const int nrows_x,
        const int ncols_y,
        const int nrows_y,
        const int nrows_dst,
        const int top_k) {
```
**EN:** This section implements `defined`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`defined`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 919-950: Device helpers and synchronization
```cpp
  const int mmq_x = MOE_X_Q3_K;
  const int mmq_y = MOE_Y_Q3_K;
  const int nwarps = NWARPS_Q3_K;

  moe_q<
      scalar_t,
      QK_K,
      QR3_K,
      QI3_K,
      false,
      block_q3_K,
      mmq_x,
      mmq_y,
      nwarps,
      allocate_tiles_q3_K<mmq_y>,
      load_tiles_q3_K<mmq_y, nwarps, need_check>,
      VDR_Q3_K_Q8_1_MMQ,
      vec_dot_q3_K_q8_1_mul_mat>(
      vx,
      vy,
      dst,
      sorted_token_ids,
      expert_ids,
      num_tokens_post_padded,
      exp_stride,
      ncols_x,
      nrows_x,
      ncols_y,
      nrows_y,
      nrows_dst,
      top_k);
}
```
**EN:** This section implements `mmq_x`, `mmq_y`, `nwarps`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`mmq_x`、`mmq_y`、`nwarps`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 951-976: Device helpers and synchronization
```cpp
template <typename scalar_t>
static void ggml_moe_q3_K_q8_1_cuda(
    const void* inp,
    const void* w,
    scalar_t* dst,
    const int* sorted_token_ids,
    const int* expert_ids,
    const int* num_tokens_post_padded,
    const int exp_stride,
    const int ncols_x,
    const int nrows_x,
    const int ncols_y,
    const int nrows_y,
    const int nrows_dst,
    const int top_k,
    const int tokens_post_padded,
    cudaStream_t stream) {
  const int mmq_x = MOE_X_Q3_K;
  const int mmq_y = MOE_Y_Q3_K;
  const int nwarps = NWARPS_Q3_K;

  const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
  const int block_num_y = (tokens_post_padded) / mmq_x;
  const dim3 block_nums(block_num_x, block_num_y, 1);
  const dim3 block_dims(WARP_SIZE_GGUF, nwarps, 1);
```
**EN:** This section implements `ggml_moe_q3_K_q8_1_cuda`, `block_nums`, `block_dims`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ggml_moe_q3_K_q8_1_cuda`、`block_nums`、`block_dims`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 977-1009: Kernel implementation
```cpp
  if (nrows_x % mmq_y == 0) {
    constexpr bool need_check = false;
    moe_q3_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  } else {
    constexpr bool need_check = true;
    moe_q3_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  }
```
**EN:** This section implements `need_check`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`need_check`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1010-1037: Kernel implementation
```cpp
}

#if defined(USE_ROCM)
#define MOE_X_Q4_K 8
#define MOE_Y_Q4_K 128
#define NWARPS_Q4_K 8
#else
#define MOE_X_Q4_K 4
#define MOE_Y_Q4_K 32
#define NWARPS_Q4_K 4
#endif

template <typename scalar_t, bool need_check>
static __global__ void
#if defined(USE_ROCM)
__launch_bounds__(WARP_SIZE_GGUF* NWARPS_Q4_K, 2)
#endif
    moe_q4_K(
        const void* __restrict__ vx,
        const void* __restrict__ vy,
        scalar_t* __restrict__ dst,
        const int* sorted_token_ids,
        const int* expert_ids,
        const int* num_tokens_post_padded,
        const int exp_stride,
        const int ncols_x,
        const int nrows_x,
        const int ncols_y,
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1038-1065: Device helpers and synchronization
```cpp
        const int nrows_y,
        const int nrows_dst,
        const int top_k) {
  const int mmq_x = MOE_X_Q4_K;
  const int mmq_y = MOE_Y_Q4_K;
  const int nwarps = NWARPS_Q4_K;

  moe_q<
      scalar_t,
      QK_K,
      QR4_K,
      QI4_K,
      true,
      block_q4_K,
      mmq_x,
      mmq_y,
      nwarps,
      allocate_tiles_q4_K<mmq_y>,
      load_tiles_q4_K<mmq_y, nwarps, need_check>,
      VDR_Q4_K_Q8_1_MMQ,
      vec_dot_q4_K_q8_1_mul_mat>(
      vx,
      vy,
      dst,
      sorted_token_ids,
      expert_ids,
      num_tokens_post_padded,
      exp_stride,
```
**EN:** This section implements `mmq_x`, `mmq_y`, `nwarps`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`mmq_x`、`mmq_y`、`nwarps`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1066-1094: Device helpers and synchronization
```cpp
      ncols_x,
      nrows_x,
      ncols_y,
      nrows_y,
      nrows_dst,
      top_k);
}

template <typename scalar_t>
static void ggml_moe_q4_K_q8_1_cuda(
    const void* inp,
    const void* w,
    scalar_t* dst,
    const int* sorted_token_ids,
    const int* expert_ids,
    const int* num_tokens_post_padded,
    const int exp_stride,
    const int ncols_x,
    const int nrows_x,
    const int ncols_y,
    const int nrows_y,
    const int nrows_dst,
    const int top_k,
    const int tokens_post_padded,
    cudaStream_t stream) {
  const int mmq_x = MOE_X_Q4_K;
  const int mmq_y = MOE_Y_Q4_K;
  const int nwarps = NWARPS_Q4_K;
```
**EN:** This section implements `ggml_moe_q4_K_q8_1_cuda`, `mmq_x`, `mmq_y`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ggml_moe_q4_K_q8_1_cuda`、`mmq_x`、`mmq_y`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1095-1122: Kernel implementation
```cpp
  const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
  const int block_num_y = (tokens_post_padded) / mmq_x;
  const dim3 block_nums(block_num_x, block_num_y, 1);
  const dim3 block_dims(WARP_SIZE_GGUF, nwarps, 1);

  if (nrows_x % mmq_y == 0) {
    constexpr bool need_check = false;
    moe_q4_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  } else {
    constexpr bool need_check = true;
    moe_q4_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
```
**EN:** This section implements `block_nums`, `block_dims`, `block_num_x`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`block_nums`、`block_dims`、`block_num_x`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1123-1144: Device helpers and synchronization
```cpp
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  }
}

#if defined(USE_ROCM)
#define MOE_X_Q5_K 8
#define MOE_Y_Q5_K 128
#define NWARPS_Q5_K 8
#else
#define MOE_X_Q5_K 4
#define MOE_Y_Q5_K 32
#define NWARPS_Q5_K 4
#endif
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1145-1167: Kernel implementation
```cpp
template <typename scalar_t, bool need_check>
static __global__ void
#if defined(USE_ROCM)
__launch_bounds__(WARP_SIZE_GGUF* NWARPS_Q5_K, 2)
#endif
    moe_q5_K(
        const void* __restrict__ vx,
        const void* __restrict__ vy,
        scalar_t* __restrict__ dst,
        const int* sorted_token_ids,
        const int* expert_ids,
        const int* num_tokens_post_padded,
        const int exp_stride,
        const int ncols_x,
        const int nrows_x,
        const int ncols_y,
        const int nrows_y,
        const int nrows_dst,
        const int top_k) {
  const int mmq_x = MOE_X_Q5_K;
  const int mmq_y = MOE_Y_Q5_K;
  const int nwarps = NWARPS_Q5_K;
```
**EN:** This section implements `defined`, `mmq_x`, `mmq_y`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`defined`、`mmq_x`、`mmq_y`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1168-1195: Device helpers and synchronization
```cpp
  moe_q<
      scalar_t,
      QK_K,
      QR5_K,
      QI5_K,
      true,
      block_q5_K,
      mmq_x,
      mmq_y,
      nwarps,
      allocate_tiles_q5_K<mmq_y>,
      load_tiles_q5_K<mmq_y, nwarps, need_check>,
      VDR_Q5_K_Q8_1_MMQ,
      vec_dot_q5_K_q8_1_mul_mat>(
      vx,
      vy,
      dst,
      sorted_token_ids,
      expert_ids,
      num_tokens_post_padded,
      exp_stride,
      ncols_x,
      nrows_x,
      ncols_y,
      nrows_y,
      nrows_dst,
      top_k);
}
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1196-1222: Device helpers and synchronization
```cpp

template <typename scalar_t>
static void ggml_moe_q5_K_q8_1_cuda(
    const void* inp,
    const void* w,
    scalar_t* dst,
    const int* sorted_token_ids,
    const int* expert_ids,
    const int* num_tokens_post_padded,
    const int exp_stride,
    const int ncols_x,
    const int nrows_x,
    const int ncols_y,
    const int nrows_y,
    const int nrows_dst,
    const int top_k,
    const int tokens_post_padded,
    cudaStream_t stream) {
  const int mmq_x = MOE_X_Q5_K;
  const int mmq_y = MOE_Y_Q5_K;
  const int nwarps = NWARPS_Q5_K;

  const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
  const int block_num_y = (tokens_post_padded) / mmq_x;
  const dim3 block_nums(block_num_x, block_num_y, 1);
  const dim3 block_dims(WARP_SIZE_GGUF, nwarps, 1);
```
**EN:** This section implements `ggml_moe_q5_K_q8_1_cuda`, `block_nums`, `block_dims`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ggml_moe_q5_K_q8_1_cuda`、`block_nums`、`block_dims`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1223-1255: Kernel implementation
```cpp
  if (nrows_x % mmq_y == 0) {
    constexpr bool need_check = false;
    moe_q5_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  } else {
    constexpr bool need_check = true;
    moe_q5_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  }
```
**EN:** This section implements `need_check`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`need_check`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1256-1283: Kernel implementation
```cpp
}

#if defined(USE_ROCM)
#define MOE_X_Q6_K 8
#define MOE_Y_Q6_K 128
#define NWARPS_Q6_K 8
#else
#define MOE_X_Q6_K 4
#define MOE_Y_Q6_K 32
#define NWARPS_Q6_K 4
#endif

template <typename scalar_t, bool need_check>
static __global__ void
#if defined(USE_ROCM)
__launch_bounds__(WARP_SIZE_GGUF* NWARPS_Q6_K, 2)
#endif
    moe_q6_K(
        const void* __restrict__ vx,
        const void* __restrict__ vy,
        scalar_t* __restrict__ dst,
        const int* sorted_token_ids,
        const int* expert_ids,
        const int* num_tokens_post_padded,
        const int exp_stride,
        const int ncols_x,
        const int nrows_x,
        const int ncols_y,
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1284-1311: Device helpers and synchronization
```cpp
        const int nrows_y,
        const int nrows_dst,
        const int top_k) {
  const int mmq_x = MOE_X_Q6_K;
  const int mmq_y = MOE_Y_Q6_K;
  const int nwarps = NWARPS_Q6_K;

  moe_q<
      scalar_t,
      QK_K,
      QR6_K,
      QI6_K,
      false,
      block_q6_K,
      mmq_x,
      mmq_y,
      nwarps,
      allocate_tiles_q6_K<mmq_y>,
      load_tiles_q6_K<mmq_y, nwarps, need_check>,
      VDR_Q6_K_Q8_1_MMQ,
      vec_dot_q6_K_q8_1_mul_mat>(
      vx,
      vy,
      dst,
      sorted_token_ids,
      expert_ids,
      num_tokens_post_padded,
      exp_stride,
```
**EN:** This section implements `mmq_x`, `mmq_y`, `nwarps`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`mmq_x`、`mmq_y`、`nwarps`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1312-1340: Device helpers and synchronization
```cpp
      ncols_x,
      nrows_x,
      ncols_y,
      nrows_y,
      nrows_dst,
      top_k);
}

template <typename scalar_t>
static void ggml_moe_q6_K_q8_1_cuda(
    const void* inp,
    const void* w,
    scalar_t* dst,
    const int* sorted_token_ids,
    const int* expert_ids,
    const int* num_tokens_post_padded,
    const int exp_stride,
    const int ncols_x,
    const int nrows_x,
    const int ncols_y,
    const int nrows_y,
    const int nrows_dst,
    const int top_k,
    const int tokens_post_padded,
    cudaStream_t stream) {
  const int mmq_x = MOE_X_Q6_K;
  const int mmq_y = MOE_Y_Q6_K;
  const int nwarps = NWARPS_Q6_K;
```
**EN:** This section implements `ggml_moe_q6_K_q8_1_cuda`, `mmq_x`, `mmq_y`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ggml_moe_q6_K_q8_1_cuda`、`mmq_x`、`mmq_y`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 1341-1368: Kernel implementation
```cpp
  const int block_num_x = (nrows_x + mmq_y - 1) / mmq_y;
  const int block_num_y = (tokens_post_padded) / mmq_x;
  const dim3 block_nums(block_num_x, block_num_y, 1);
  const dim3 block_dims(WARP_SIZE_GGUF, nwarps, 1);

  if (nrows_x % mmq_y == 0) {
    constexpr bool need_check = false;
    moe_q6_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  } else {
    constexpr bool need_check = true;
    moe_q6_K<scalar_t, need_check><<<block_nums, block_dims, 0, stream>>>(
        w,
        inp,
        dst,
        sorted_token_ids,
```
**EN:** This section implements `block_nums`, `block_dims`, `block_num_x`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`block_nums`、`block_dims`、`block_num_x`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 1369-1379: Local implementation details
```cpp
        expert_ids,
        num_tokens_post_padded,
        exp_stride,
        ncols_x,
        nrows_x,
        ncols_y,
        nrows_y,
        nrows_dst,
        top_k);
  }
}
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `cstdint`
- **Path context / 路径上下文**: quantization / gguf / moe.cuh
