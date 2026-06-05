# mmvq.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/quantization/gguf/mmvq.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements quantization, dequantization, or low-bit arithmetic utilities used by inference kernels. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 实现推理内核使用的量化、反量化或低比特算术工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Kernel implementation
```cpp
// copied from
// https://github.com/vllm-project/vllm/blob/4492e3a55428e161ca8db381edc28263e5da4c8d/csrc/quantization/gguf/mmvq.cuh
// copied and adapted from https://github.com/ggerganov/llama.cpp/blob/b2899/ggml-cuda/mmvq.cu
template <typename scalar_t, int qk, int qi, typename block_q_t, int vdr, vec_dot_q_cuda_t vec_dot_q_cuda>
static __global__ void mul_mat_vec_q(
    const void* __restrict__ vx,
    const void* __restrict__ vy,
    scalar_t* __restrict__ dst,
    const int ncols,
    const int nrows,
    const int nvecs) {
  const auto row = blockIdx.x * blockDim.y + threadIdx.y;
  const auto vec = blockIdx.y;

  if (row >= nrows || vec >= nvecs) {
    return;
  }

  const int blocks_per_row = ncols / qk;
  const int blocks_per_warp = vdr * WARP_SIZE / qi;
  const int nrows_y = (ncols + 512 - 1) / 512 * 512;

  // partial sum for each thread
  float tmp = 0.0f;
```
**EN:** This section implements `mul_mat_vec_q`, `row`, `vec`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_q`、`row`、`vec`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 26-49: Device helpers and synchronization
```cpp
  const block_q_t* x = (const block_q_t*)vx;
  const block_q8_1* y = (const block_q8_1*)vy;

  for (auto i = threadIdx.x / (qi / vdr); i < blocks_per_row; i += blocks_per_warp) {
    const int ibx = row * blocks_per_row + i;  // x block index

    const int iby = vec * (nrows_y / QK8_1) + i * (qk / QK8_1);  // y block index that aligns with ibx

    const int iqs = vdr * (threadIdx.x % (qi / vdr));  // x block quant index when casting the quants to int

    tmp += vec_dot_q_cuda(&x[ibx], &y[iby], iqs);
  }

  // sum up partial sums and write back result
#pragma unroll
  for (int mask = WARP_SIZE / 2; mask > 0; mask >>= 1) {
    tmp += SGLANG_SHFL_XOR_SYNC(uint32_t(-1), tmp, mask);
  }

  if (threadIdx.x == 0) {
    dst[vec * nrows + row] = tmp;
  }
}
```
**EN:** This section implements `vec_dot_q_cuda`, `SGLANG_SHFL_XOR_SYNC`, `x`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`vec_dot_q_cuda`、`SGLANG_SHFL_XOR_SYNC`、`x`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 50-73: Kernel implementation
```cpp
template <typename scalar_t>
static void mul_mat_vec_q4_0_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK4_0, QI4_0, block_q4_0, VDR_Q4_0_Q8_1_MMVQ, vec_dot_q4_0_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_q4_1_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
```
**EN:** This section implements `mul_mat_vec_q4_0_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_q4_0_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 74-97: Kernel implementation
```cpp
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK4_0, QI4_1, block_q4_1, VDR_Q4_1_Q8_1_MMVQ, vec_dot_q4_1_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_q5_0_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK5_0, QI5_0, block_q5_0, VDR_Q5_0_Q8_1_MMVQ, vec_dot_q5_0_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}
```
**EN:** This section implements `mul_mat_vec_q5_0_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_q5_0_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 98-121: Kernel implementation
```cpp
template <typename scalar_t>
static void mul_mat_vec_q5_1_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK5_1, QI5_1, block_q5_1, VDR_Q5_1_Q8_1_MMVQ, vec_dot_q5_1_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_q8_0_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
```
**EN:** This section implements `mul_mat_vec_q5_1_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_q5_1_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 122-145: Kernel implementation
```cpp
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK8_0, QI8_0, block_q8_0, VDR_Q8_0_Q8_1_MMVQ, vec_dot_q8_0_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_q2_K_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI2_K, block_q2_K, VDR_Q2_K_Q8_1_MMVQ, vec_dot_q2_K_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}
```
**EN:** This section implements `mul_mat_vec_q2_K_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_q2_K_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 146-169: Kernel implementation
```cpp
template <typename scalar_t>
static void mul_mat_vec_q3_K_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI3_K, block_q3_K, VDR_Q3_K_Q8_1_MMVQ, vec_dot_q3_K_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_q4_K_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
```
**EN:** This section implements `mul_mat_vec_q3_K_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_q3_K_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 170-193: Kernel implementation
```cpp
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI4_K, block_q4_K, VDR_Q4_K_Q8_1_MMVQ, vec_dot_q4_K_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_q5_K_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI5_K, block_q5_K, VDR_Q5_K_Q8_1_MMVQ, vec_dot_q5_K_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}
```
**EN:** This section implements `mul_mat_vec_q5_K_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_q5_K_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 194-217: Kernel implementation
```cpp
template <typename scalar_t>
static void mul_mat_vec_q6_K_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI6_K, block_q6_K, VDR_Q6_K_Q8_1_MMVQ, vec_dot_q6_K_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_iq2_xxs_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
```
**EN:** This section implements `mul_mat_vec_q6_K_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_q6_K_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 218-241: Kernel implementation
```cpp
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI2_XXS, block_iq2_xxs, 1, vec_dot_iq2_xxs_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_iq2_xs_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI2_XS, block_iq2_xs, 1, vec_dot_iq2_xs_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}
```
**EN:** This section implements `mul_mat_vec_iq2_xs_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_iq2_xs_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 242-265: Kernel implementation
```cpp
template <typename scalar_t>
static void mul_mat_vec_iq2_s_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI2_S, block_iq2_s, 1, vec_dot_iq2_s_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_iq3_xxs_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
```
**EN:** This section implements `mul_mat_vec_iq2_s_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_iq2_s_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 266-289: Kernel implementation
```cpp
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI3_XXS, block_iq3_xxs, 1, vec_dot_iq3_xxs_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_iq1_s_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI1_S, block_iq1_s, 1, vec_dot_iq1_s_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}
```
**EN:** This section implements `mul_mat_vec_iq1_s_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_iq1_s_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 290-313: Kernel implementation
```cpp
template <typename scalar_t>
static void mul_mat_vec_iq1_m_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI1_M, block_iq1_m, 1, vec_dot_iq1_m_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_iq4_nl_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
```
**EN:** This section implements `mul_mat_vec_iq1_m_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_iq1_m_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 314-337: Kernel implementation
```cpp
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK4_NL, QI4_NL, block_iq4_nl, VDR_Q4_0_Q8_1_MMVQ, vec_dot_iq4_nl_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}

template <typename scalar_t>
static void mul_mat_vec_iq4_xs_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI4_XS, block_iq4_xs, 1, vec_dot_iq4_xs_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}
```
**EN:** This section implements `mul_mat_vec_iq4_xs_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_iq4_xs_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 338-352: Kernel implementation
```cpp
template <typename scalar_t>
static void mul_mat_vec_iq3_s_q8_1_cuda(
    const void* vx,
    const void* vy,
    scalar_t* dst,
    const int ncols,
    const int nrows,
    const int nvecs,
    cudaStream_t stream) {
  const int block_num_y = (nrows + GGML_CUDA_MMV_Y - 1) / GGML_CUDA_MMV_Y;
  const dim3 block_nums(block_num_y, nvecs, 1);
  const dim3 block_dims(WARP_SIZE, GGML_CUDA_MMV_Y, 1);
  mul_mat_vec_q<scalar_t, QK_K, QI3_XS, block_iq3_s, 1, vec_dot_iq3_s_q8_1>
      <<<block_nums, block_dims, 0, stream>>>(vx, vy, dst, ncols, nrows, nvecs);
}
```
**EN:** This section implements `mul_mat_vec_iq3_s_q8_1_cuda`, `block_nums`, `block_dims`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`mul_mat_vec_iq3_s_q8_1_cuda`、`block_nums`、`block_dims`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: quantization / gguf / mmvq.cuh
