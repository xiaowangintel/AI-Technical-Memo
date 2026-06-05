# per_token_quant_fp8.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/per_token_quant_fp8.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Kernel implementation
```cpp
#include <ATen/cuda/CUDAContext.h>

#include <cmath>
#include <flashinfer/vec_dtypes.cuh>

#include "utils.h"

static constexpr int kWarpSize = 32;
static constexpr int DEFAULT_SHARED_MEM_THRESHOLD_KB = 48;  // Default shared memory quota in KB

// ---------------------------------------------------------------------------
// 1. Warp‑local with configurable shared memory
//    • One warp handles one token.
//    • Eight tokens per 256‑thread CTA.
//    • Shared memory usage is configurable via template parameter.
// ---------------------------------------------------------------------------
template <typename T, typename DST_DTYPE, int kTokensPerCTA = 8, int kVecSize = 16, bool USE_SMEM = true>
__global__ void per_token_quant_fp8_kernel(
    const T* __restrict__ input,
    DST_DTYPE* __restrict__ output_q,
    float* __restrict__ output_s,
    const int64_t hidden_dim,
    const int64_t num_tokens) {
  const int warp_id = threadIdx.x / kWarpSize;        // 0‑7  (8 warps)
  const int lane_id = threadIdx.x & (kWarpSize - 1);  // 0‑31
  const int token_id = blockIdx.x * kTokensPerCTA + warp_id;
  if (token_id >= num_tokens) return;
```
**EN:** This section implements `per_token_quant_fp8_kernel`, `kWarpSize`, `DEFAULT_SHARED_MEM_THRESHOLD_KB`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`per_token_quant_fp8_kernel`、`kWarpSize`、`DEFAULT_SHARED_MEM_THRESHOLD_KB`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 29-50: Device helpers and synchronization
```cpp
  // Global tensors for this token
  const T* token_input = input + token_id * hidden_dim;
  DST_DTYPE* token_output = output_q + token_id * hidden_dim;
  float* token_scale = output_s + token_id;

  extern __shared__ char smem_buffer[];
  const int smem_padding = 32;  // Pad to bank boundary (32 banks * 4 bytes = 128 bytes)
  const int warp_smem_stride = (hidden_dim * sizeof(T) + smem_padding - 1) / smem_padding * smem_padding;
  const int warp_smem_offset = warp_id * warp_smem_stride;
  T* shared_input = reinterpret_cast<T*>(smem_buffer + warp_smem_offset);

  //
  // Pass-1: Load data and compute max_value
  //
  float max_value = 0.f;
  using vec_t = flashinfer::vec_t<T, kVecSize>;
  const int32_t num_vec_elems = hidden_dim / kVecSize;

  for (int32_t i = lane_id; i < num_vec_elems; i += kWarpSize) {
    vec_t input_vec;
    input_vec.cast_load(token_input + i * kVecSize);
```
**EN:** This section implements `cast_load`, `token_input`, `smem_padding`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`cast_load`、`token_input`、`smem_padding`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 51-72: Device helpers and synchronization
```cpp
    // Store to shared memory if USE_SMEM=true
    if constexpr (USE_SMEM) {
#pragma unroll
      for (uint32_t j = 0; j < kVecSize; ++j) {
        shared_input[i * kVecSize + j] = input_vec[j];
      }
    }

    // Compute max value in parallel
#pragma unroll
    for (uint32_t j = 0; j < kVecSize; ++j) {
      max_value = fmaxf(max_value, fabsf(static_cast<float>(input_vec[j])));
    }
  }

  // Ensure all threads in the warp have finished writing to shared memory
  if constexpr (USE_SMEM) {
    __syncwarp();
  }

  float warp_max = warpReduceMax(max_value);
```
**EN:** This section implements `fmaxf`, `__syncwarp`, `warpReduceMax`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`fmaxf`、`__syncwarp`、`warpReduceMax`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 73-97: Device helpers and synchronization
```cpp
  // NOTE: one CTA has multiple warps (each warp handles one token), so `scale`
  // must be per-warp/per-thread (register) instead of a single shared variable.
  const float scale = warp_max / FP8_E4M3_MAX;
  // Broadcast scale
  if (lane_id == 0) {
    token_scale[0] = scale;
  }
  const float scale_inv = (scale == 0.f) ? 0.f : 1.0f / scale;

  //
  // Pass-2: Quantize and write back
  //
  for (int i = lane_id; i < num_vec_elems; i += kWarpSize) {
    vec_t input_vec;

    if constexpr (USE_SMEM) {
      // Load from shared memory
#pragma unroll
      for (uint32_t j = 0; j < kVecSize; ++j) {
        input_vec[j] = shared_input[i * kVecSize + j];
      }
    } else {
      // Reload from global memory
      input_vec.cast_load(token_input + i * kVecSize);
    }
```
**EN:** This section implements `cast_load`, `scale`, `scale_inv`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`cast_load`、`scale`、`scale_inv`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 98-121: Runtime integration and dispatch
```cpp

    DST_DTYPE output_arr[kVecSize];
#pragma unroll
    for (uint32_t j = 0; j < kVecSize; ++j) {
      float val = static_cast<float>(input_vec[j]) * scale_inv;
      val = fmaxf(fminf(val, FP8_E4M3_MAX), -FP8_E4M3_MAX);
#if !defined(USE_ROCM) || defined(HIP_FP8_TYPE_E4M3)
      output_arr[j] = static_cast<DST_DTYPE>(val);
#else
      output_arr[j] = c10::Float8_e4m3fnuz(
          __hip_cvt_float_to_fp8(val, fp8::fp8_type::__default_saturation, fp8::fp8_type::__default_interpret),
          c10::Float8_e4m3fnuz::from_bits());
#endif
    }
    if constexpr (kVecSize == 16) {
      *(uint4*)(token_output + i * kVecSize) = *(uint4*)output_arr;
    } else {
      // Use element-wise copy for vector size 8 to ensure correctness
      for (int k = 0; k < kVecSize; ++k) {
        token_output[i * kVecSize + k] = output_arr[k];
      }
    }
  }
}
```
**EN:** This section uses `fmaxf`, `defined`, `Float8_e4m3fnuz` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fmaxf`、`defined`、`Float8_e4m3fnuz`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 122-147: Kernel implementation
```cpp

// ---------------------------------------------------------------------------
// 2.  Baseline kernel (1 token / CTA, CUB block reduce)
// ---------------------------------------------------------------------------
template <typename T, typename DST_DTYPE, int kVecSize = 16>
__global__ void per_token_quant_fp8_small_batch_kernel(
    const T* __restrict__ input,
    DST_DTYPE* __restrict__ output_q,
    float* __restrict__ output_s,
    const int64_t hidden_dim,
    const int64_t num_tokens) {
  const int token_idx = blockIdx.x;
  if (token_idx >= num_tokens) return;

  const int tid = threadIdx.x;
  const int block_dim = blockDim.x;

  const T* token_input = input + token_idx * hidden_dim;
  DST_DTYPE* token_output = output_q + token_idx * hidden_dim;

  float max_value = 0.0f;

  // Use template parameter for vector size
  using vec_t = flashinfer::vec_t<T, kVecSize>;
  const int32_t num_vec_elems = hidden_dim / kVecSize;
```
**EN:** This section implements `per_token_quant_fp8_small_batch_kernel`, `token_idx`, `tid`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`per_token_quant_fp8_small_batch_kernel`、`token_idx`、`tid`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 148-170: Device helpers and synchronization
```cpp
  // Find max using vectorized loads
  for (int32_t i = tid; i < num_vec_elems; i += block_dim) {
    vec_t input_vec;
    input_vec.cast_load(token_input + i * kVecSize);

#pragma unroll
    for (uint32_t j = 0; j < kVecSize; ++j) {
      float val = static_cast<float>(input_vec[j]);
      max_value = fmaxf(max_value, fabsf(val));
    }
  }

  max_value = blockReduceMax(max_value);

  __shared__ float scale;
  if (tid == 0) {
    scale = max_value / FP8_E4M3_MAX;
    output_s[token_idx] = scale;
  }
  __syncthreads();

  const float scale_inv = 1.0f / scale;
```
**EN:** This section implements `cast_load`, `fmaxf`, `blockReduceMax`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`cast_load`、`fmaxf`、`blockReduceMax`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 171-195: Runtime integration and dispatch
```cpp
  // Quantize using vectorized loads
  for (int32_t i = tid; i < num_vec_elems; i += block_dim) {
    vec_t input_vec;
    input_vec.cast_load(token_input + i * kVecSize);

    DST_DTYPE output_arr[kVecSize];
#pragma unroll
    for (uint32_t j = 0; j < kVecSize; ++j) {
      float val = fmaxf(fminf(static_cast<float>(input_vec[j]) * scale_inv, FP8_E4M3_MAX), -FP8_E4M3_MAX);
#if !defined(USE_ROCM) || defined(HIP_FP8_TYPE_E4M3)
      output_arr[j] = static_cast<DST_DTYPE>(val);
#else
      output_arr[j] = c10::Float8_e4m3fnuz(
          __hip_cvt_float_to_fp8(val, fp8::fp8_type::__default_saturation, fp8::fp8_type::__default_interpret),
          c10::Float8_e4m3fnuz::from_bits());
#endif
    }

    if constexpr (kVecSize == 16) {
      *(uint4*)(token_output + i * kVecSize) = *(uint4*)output_arr;
    } else {
      // Use element-wise copy for vector size 8 to ensure correctness
      for (int k = 0; k < kVecSize; ++k) {
        token_output[i * kVecSize + k] = output_arr[k];
      }
```
**EN:** This section uses `cast_load`, `fmaxf`, `defined` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`cast_load`、`fmaxf`、`defined`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 196-214: Device helpers and synchronization
```cpp
    }
  }
}

template <bool USE_SMEM, typename scalar_t, int TOKENS_PER_CTA>
static inline void launch_per_token_quant_fp8_warp_kernel(
    const dim3& grid,
    const dim3& block,
    size_t dynamicSmemSz,
    cudaStream_t stream,
    bool use_vec16,
    bool use_vec8,
    torch::Tensor input,
    torch::Tensor output_q,
    torch::Tensor output_s,
    const int64_t hidden_dim,
    const int64_t num_tokens) {
  const size_t smem_size = USE_SMEM ? dynamicSmemSz : 0;
```
**EN:** This section implements `launch_per_token_quant_fp8_warp_kernel`, `smem_size`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`launch_per_token_quant_fp8_warp_kernel`、`smem_size`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 215-239: Kernel implementation
```cpp
  if (use_vec16) {
    per_token_quant_fp8_kernel<scalar_t, __nv_fp8_e4m3, TOKENS_PER_CTA, 16, USE_SMEM>
        <<<grid, block, smem_size, stream>>>(
            static_cast<const scalar_t*>(input.data_ptr()),
            static_cast<__nv_fp8_e4m3*>(output_q.data_ptr()),
            static_cast<float*>(output_s.data_ptr()),
            hidden_dim,
            num_tokens);
  } else if (use_vec8) {
    per_token_quant_fp8_kernel<scalar_t, __nv_fp8_e4m3, TOKENS_PER_CTA, 8, USE_SMEM>
        <<<grid, block, smem_size, stream>>>(
            static_cast<const scalar_t*>(input.data_ptr()),
            static_cast<__nv_fp8_e4m3*>(output_q.data_ptr()),
            static_cast<float*>(output_s.data_ptr()),
            hidden_dim,
            num_tokens);
  } else {
    per_token_quant_fp8_kernel<scalar_t, __nv_fp8_e4m3, TOKENS_PER_CTA, 4, USE_SMEM>
        <<<grid, block, smem_size, stream>>>(
            static_cast<const scalar_t*>(input.data_ptr()),
            static_cast<__nv_fp8_e4m3*>(output_q.data_ptr()),
            static_cast<float*>(output_s.data_ptr()),
            hidden_dim,
            num_tokens);
  }
```
**EN:** This section implements `data_ptr`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`data_ptr`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 240-264: Device helpers and synchronization
```cpp
}

void sgl_per_token_quant_fp8(torch::Tensor input, torch::Tensor output_q, torch::Tensor output_s) {
  CHECK_INPUT(input);
  CHECK_INPUT(output_q);
  CHECK_INPUT(output_s);
  const auto input_sizes = input.sizes();
  const int64_t num_tokens = input_sizes[0];
  const int64_t hidden_dim = input_sizes[1];
  TORCH_CHECK(hidden_dim % 4 == 0, "Hidden dimension must be divisible by 4, but got ", hidden_dim);

  cudaStream_t stream = at::cuda::getCurrentCUDAStream();
  const int sm_count = at::cuda::getCurrentDeviceProperties()->multiProcessorCount;
  const int TOKENS_PER_CTA = 8;
  const bool use_warp_kernel = (num_tokens >= sm_count * 2 * TOKENS_PER_CTA);
  const bool use_vec16 = (hidden_dim % 16 == 0);
  const bool use_vec8 = (hidden_dim % 8 == 0);

  const int sizeof_T = input.scalar_type() == torch::kFloat16 ? 2 : (input.scalar_type() == torch::kBFloat16 ? 2 : 4);
  const int smem_padding = 32;  // Pad to bank boundary to avoid conflicts
  const int warp_smem_stride = (hidden_dim * sizeof_T + smem_padding - 1) / smem_padding * smem_padding;
  const size_t dynamicSmemSz = warp_smem_stride * TOKENS_PER_CTA;

  bool use_smem = (hidden_dim < 2048);
```
**EN:** This section implements `sgl_per_token_quant_fp8`, `CHECK_INPUT`, `sizes`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`sgl_per_token_quant_fp8`、`CHECK_INPUT`、`sizes`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 265-288: Device helpers and synchronization
```cpp
  if (dynamicSmemSz >= DEFAULT_SHARED_MEM_THRESHOLD_KB) {
    use_smem = false;  // Disable shared memory if >= 48KB to avoid allocation failures
  }

  DISPATCH_PYTORCH_DTYPE_TO_CTYPE_FLOAT_FP16(input.scalar_type(), scalar_t, [&] {
    if (use_warp_kernel) {
      // -------- warp‑local ---------------------------------------------------
      constexpr int THREADS = TOKENS_PER_CTA * kWarpSize;
      dim3 grid((num_tokens + TOKENS_PER_CTA - 1) / TOKENS_PER_CTA);
      dim3 block(THREADS);

      if (use_smem) {
        launch_per_token_quant_fp8_warp_kernel</*USE_SMEM=*/true, scalar_t, TOKENS_PER_CTA>(
            grid, block, dynamicSmemSz, stream, use_vec16, use_vec8, input, output_q, output_s, hidden_dim, num_tokens);
      } else {
        launch_per_token_quant_fp8_warp_kernel</*USE_SMEM=*/false, scalar_t, TOKENS_PER_CTA>(
            grid, block, dynamicSmemSz, stream, use_vec16, use_vec8, input, output_q, output_s, hidden_dim, num_tokens);
      }
    } else {
      // -------- baseline -----------------------------------------------------
      constexpr int THREADS = 256;
      dim3 grid(num_tokens);
      dim3 block(THREADS);
```
**EN:** This section implements `grid`, `block`, `THREADS`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`grid`、`block`、`THREADS`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 289-311: Kernel implementation
```cpp
      if (use_vec16) {
        per_token_quant_fp8_small_batch_kernel<scalar_t, __nv_fp8_e4m3, 16><<<grid, block, 0, stream>>>(
            static_cast<const scalar_t*>(input.data_ptr()),
            static_cast<__nv_fp8_e4m3*>(output_q.data_ptr()),
            static_cast<float*>(output_s.data_ptr()),
            hidden_dim,
            num_tokens);
      } else if (use_vec8) {
        per_token_quant_fp8_small_batch_kernel<scalar_t, __nv_fp8_e4m3, 8><<<grid, block, 0, stream>>>(
            static_cast<const scalar_t*>(input.data_ptr()),
            static_cast<__nv_fp8_e4m3*>(output_q.data_ptr()),
            static_cast<float*>(output_s.data_ptr()),
            hidden_dim,
            num_tokens);
      } else {
        per_token_quant_fp8_small_batch_kernel<scalar_t, __nv_fp8_e4m3, 4><<<grid, block, 0, stream>>>(
            static_cast<const scalar_t*>(input.data_ptr()),
            static_cast<__nv_fp8_e4m3*>(output_q.data_ptr()),
            static_cast<float*>(output_s.data_ptr()),
            hidden_dim,
            num_tokens);
      }
    }
```
**EN:** This section implements `data_ptr`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`data_ptr`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 312-314: Local implementation details
```cpp
    return true;
  });
}
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `utils.h`
- **External headers / 外部头文件**: `ATen/cuda/CUDAContext.h`, `cmath`, `flashinfer/vec_dtypes.cuh`
- **Path context / 路径上下文**: gemm / per_token_quant_fp8.cu
