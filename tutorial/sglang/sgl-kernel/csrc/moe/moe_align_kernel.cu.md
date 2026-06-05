# moe_align_kernel.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/moe/moe_align_kernel.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements mixture-of-experts routing, combination, or expert computation utilities. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 实现 MoE 的路由、聚合或专家计算辅助逻辑。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Headers and compile-time setup
```cpp
/* Copyright 2025 SGLang Team. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
==============================================================================*/

#include <ATen/ATen.h>
#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAGuard.h>

#include <THC/THCAtomics.cuh>

#include "utils.h"
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 24-50: Kernel implementation
```cpp
#define VEC_SIZE 4
using Vec = int4;

template <typename scalar_t>
__global__ void count_and_sort_expert_tokens_kernel(
    const scalar_t* __restrict__ topk_ids,
    int32_t* __restrict__ sorted_token_ids,
    int32_t* __restrict__ cumsum_buffer,
    size_t numel) {
  const size_t tid = blockIdx.x * blockDim.x + threadIdx.x;
  const size_t stride = blockDim.x * gridDim.x;

  for (size_t i = tid; i < numel; i += stride) {
    int32_t expert_id = topk_ids[i] + 1;
    int32_t rank_post_pad = atomicAdd(&cumsum_buffer[expert_id], 1);
    sorted_token_ids[rank_post_pad] = i;
  }
}

#ifdef __CUDA_ARCH__
__device__ __forceinline__ int warp_exclusive_scan(int v, unsigned mask = 0xffffffffu) {
  int original = v;
#pragma unroll
  for (int offset = 1; offset < WARP_SIZE; offset <<= 1) {
    int n = __shfl_up_sync(mask, v, offset);
    if ((threadIdx.x & (WARP_SIZE - 1)) >= offset) v += n;
  }
```
**EN:** This section implements `count_and_sort_expert_tokens_kernel`, `warp_exclusive_scan`, `atomicAdd`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`count_and_sort_expert_tokens_kernel`、`warp_exclusive_scan`、`atomicAdd`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 51-77: Kernel implementation
```cpp
  return v - original;
}
#endif

template <typename scalar_t>
__global__ void moe_align_block_size_kernel(
    const scalar_t* __restrict__ topk_ids,
    int32_t* __restrict__ sorted_token_ids,
    int32_t* __restrict__ expert_ids,
    int32_t* __restrict__ total_tokens_post_pad,
    int32_t num_experts,
    int32_t block_size,
    size_t numel,
    int32_t* __restrict__ cumsum,
    bool pad_sorted_token_ids,
    const int32_t scan_size,
    int32_t max_num_tokens_padded) {
  // Use a separate thread block to populate sorted_token_ids
  if (blockIdx.x == 1) {
    if (pad_sorted_token_ids) {
      Vec fill_vec;
      fill_vec.x = fill_vec.y = fill_vec.z = fill_vec.w = numel;
      int32_t total_vecs = (max_num_tokens_padded + VEC_SIZE - 1) / VEC_SIZE;
      Vec* out_ptr = reinterpret_cast<Vec*>(sorted_token_ids);
      for (int32_t i = threadIdx.x; i < total_vecs; i += blockDim.x) {
        out_ptr[i] = fill_vec;
      }
```
**EN:** This section implements `moe_align_block_size_kernel`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`moe_align_block_size_kernel`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 78-101: Device helpers and synchronization
```cpp
    }
    return;
  }

  extern __shared__ int32_t smem[];
  int32_t* shared_counts = smem;                  // [num_experts]
  int32_t* prefix = shared_counts + num_experts;  // [num_experts + 1]
  int32_t* scan_buf = prefix + num_experts + 1;   // [scan_size]
  __shared__ int32_t s_total_tokens_post_pad;

  const size_t tid = threadIdx.x;
  const size_t stride = blockDim.x;

  if (tid < num_experts) {
    shared_counts[tid] = 0;
  }

  __syncthreads();

  for (size_t i = tid; i < numel; i += stride) {
    int expert_id = topk_ids[i] + 1;
    atomicAdd(&shared_counts[expert_id], 1);
  }
```
**EN:** This section implements `__syncthreads`, `atomicAdd`, `tid`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__syncthreads`、`atomicAdd`、`tid`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 102-127: Device helpers and synchronization
```cpp
  __syncthreads();

  int32_t padded_count = 0;
  if (tid < num_experts) {
    int32_t count = shared_counts[tid];
    padded_count = (count + block_size - 1) / block_size * block_size;
    scan_buf[tid] = padded_count;
  }

#ifndef __CUDA_ARCH__  // HIP

  if (tid >= num_experts && tid < scan_size) {
    scan_buf[tid] = 0;
  }

  __syncthreads();

  // Blelloch scan
  int offset = 1;
#pragma unroll
  for (int d = scan_size >> 1; d > 0; d >>= 1) {
    if (tid < d) {
      int ai = offset * (2 * tid + 1) - 1;
      int bi = offset * (2 * tid + 2) - 1;
      scan_buf[bi] += scan_buf[ai];
    }
```
**EN:** This section implements `__syncthreads`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__syncthreads`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 128-152: Device helpers and synchronization
```cpp
    offset <<= 1;
    __syncthreads();
  }

  // down-sweep
  if (tid == 0) {
    prefix[num_experts] = scan_buf[scan_size - 1];
    scan_buf[scan_size - 1] = 0;
  }
  __syncthreads();

#pragma unroll
  for (int d = 1; d < scan_size; d <<= 1) {
    offset >>= 1;
    if (tid < d) {
      int ai = offset * (2 * tid + 1) - 1;
      int bi = offset * (2 * tid + 2) - 1;
      if (bi < scan_size) {
        int temp = scan_buf[ai];
        scan_buf[ai] = scan_buf[bi];
        scan_buf[bi] += temp;
      }
    }
    __syncthreads();
  }
```
**EN:** This section implements `__syncthreads`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__syncthreads`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 153-174: Device helpers and synchronization
```cpp

  if (tid < num_experts) {
    prefix[tid] = scan_buf[tid];
  }

  if (tid == 0) {
    s_total_tokens_post_pad = prefix[num_experts];
    *total_tokens_post_pad = s_total_tokens_post_pad;
  }
  __syncthreads();

#else  // CUDA

  // Intra warp prefix sum
  int32_t* warp_sums = scan_buf + scan_size;  // [<= 32]
  const int warp_id = tid / WARP_SIZE;
  const int lane_id = tid & (WARP_SIZE - 1);
  const int num_warps_for_scan = (scan_size + WARP_SIZE - 1) / WARP_SIZE;
  const int warp_sum = warp_exclusive_scan(padded_count) + padded_count;
  if (lane_id == WARP_SIZE - 1) warp_sums[warp_id] = warp_sum;
  __syncthreads();
```
**EN:** This section implements `__syncthreads`, `warp_id`, `lane_id`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__syncthreads`、`warp_id`、`lane_id`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 175-200: Device helpers and synchronization
```cpp
  // warp0 accumulate all the block's prefix sum
  if (tid < WARP_SIZE) {
    int val = (tid < num_warps_for_scan) ? warp_sums[tid] : 0;
    int incl = warp_exclusive_scan(val) + val;
    warp_sums[tid] = incl;
  }
  __syncthreads();

  // Every thread obtains the whole block's sum
  if (tid == 0) {
    prefix[num_experts] = warp_sums[num_warps_for_scan - 1];
    s_total_tokens_post_pad = prefix[num_experts];
    *total_tokens_post_pad = s_total_tokens_post_pad;
  }
  __syncthreads();

  // Fill 0 to scan_buf extended area (tid >= num_expert)
  if (tid >= num_experts && tid < scan_size) scan_buf[tid] = 0;
  __syncthreads();

  // Perform 2 level exclusive-prefix-sum to scan_buf
  int v = (tid < scan_size) ? scan_buf[tid] : 0;
  int pre = warp_exclusive_scan(v);
  if (lane_id == WARP_SIZE - 1) warp_sums[warp_id] = pre + v;
  __syncthreads();
```
**EN:** This section implements `__syncthreads`, `warp_exclusive_scan`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__syncthreads`、`warp_exclusive_scan`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 201-229: Device helpers and synchronization
```cpp
  if (warp_id == 0) {
    int val = (lane_id < num_warps_for_scan) ? warp_sums[lane_id] : 0;
    warp_sums[lane_id] = warp_exclusive_scan(val);
  }
  __syncthreads();

  int offset = warp_sums[warp_id];
  if (tid < scan_size) scan_buf[tid] = pre + offset;
  __syncthreads();

  // Write prefix[0..num_experts - 1] and cumsum
  if (tid < num_experts) prefix[tid] = scan_buf[tid];
#endif

  if (tid <= num_experts) {
    cumsum[tid] = prefix[tid];
  }
  // fill expert_ids
  const int32_t num_blocks = s_total_tokens_post_pad / block_size;
  for (int32_t i = tid; i < num_blocks; i += stride) {
    int32_t block_start = i * block_size;
    int left = 0, right = num_experts;
    while (left < right) {
      int mid = (left + right) >> 1;
      if (prefix[mid] <= block_start) {
        left = mid + 1;
      } else {
        right = mid;
      }
```
**EN:** This section implements `warp_exclusive_scan`, `__syncthreads`, `num_blocks`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`warp_exclusive_scan`、`__syncthreads`、`num_blocks`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 230-257: Kernel implementation
```cpp
    }
    expert_ids[i] = left - 2;
  }
}

template <typename scalar_t, int32_t fill_threads>
__global__ void moe_align_block_size_small_batch_expert_kernel(
    const scalar_t* __restrict__ topk_ids,
    int32_t* __restrict__ sorted_token_ids,
    int32_t* __restrict__ expert_ids,
    int32_t* __restrict__ total_tokens_post_pad,
    int32_t num_experts,
    int32_t block_size,
    size_t numel,
    bool pad_sorted_token_ids,
    int32_t max_num_tokens_padded) {
  // Adapted from
  // https://github.com/vllm-project/vllm/pull/29642/files#diff-5647b1413f4ae9aacba904eca8f8a8aee9079321eadff4c10101a2c6962dcc53R226
  // Use an additional group of threads to fill sorted_token_ids.
  // Since the kernel will use sorted_token_ids afterward,
  // we fill sorted_token_ids within the same threadblock to make
  // synchronization easier.
  if (threadIdx.x < fill_threads) {
    // Initialize sorted_token_ids with numel
    if (pad_sorted_token_ids) {
      for (int32_t it = threadIdx.x; it < max_num_tokens_padded; it += fill_threads) {
        sorted_token_ids[it] = numel;
      }
```
**EN:** This section implements `moe_align_block_size_small_batch_expert_kernel`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`moe_align_block_size_small_batch_expert_kernel`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 258-281: Device helpers and synchronization
```cpp
    }
    // Three __syncthreads() corresponding to the other threads
    __syncthreads();
    __syncthreads();
    __syncthreads();
    return;
  }

  const size_t tid = threadIdx.x - fill_threads;
  const size_t stride = blockDim.x - fill_threads;

  extern __shared__ int32_t shared_mem[];
  int32_t* cumsum = shared_mem;
  int32_t* tokens_cnts = (int32_t*)(shared_mem + num_experts + 1);

  for (int i = 0; i < num_experts; ++i) {
    tokens_cnts[(tid + 1) * num_experts + i] = 0;
  }

  for (size_t i = tid; i < numel; i += stride) {
    int32_t expert_id = topk_ids[i] + 1;
    ++tokens_cnts[(tid + 1) * num_experts + expert_id];
  }
```
**EN:** This section implements `__syncthreads`, `tid`, `stride`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__syncthreads`、`tid`、`stride`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 282-306: Device helpers and synchronization
```cpp
  __syncthreads();

  if (tid < num_experts) {
    tokens_cnts[tid] = 0;
    for (int i = 1; i <= stride; ++i) {
      tokens_cnts[i * num_experts + tid] += tokens_cnts[(i - 1) * num_experts + tid];
    }
  }

  __syncthreads();

  if (tid == 0) {
    cumsum[0] = 0;
    for (int i = 1; i <= num_experts; ++i) {
      cumsum[i] = cumsum[i - 1] + CEILDIV(tokens_cnts[stride * num_experts + i - 1], block_size) * block_size;
    }
    *total_tokens_post_pad = static_cast<int32_t>(cumsum[num_experts]);
  }

  __syncthreads();

  if (tid < num_experts) {
    for (int i = cumsum[tid]; i < cumsum[tid + 1]; i += block_size) {
      expert_ids[i / block_size] = tid - 1;
    }
```
**EN:** This section implements `__syncthreads`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__syncthreads`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 307-330: Device helpers and synchronization
```cpp
  }

  for (size_t i = tid; i < numel; i += stride) {
    int32_t expert_id = topk_ids[i] + 1;
    int32_t rank_post_pad = tokens_cnts[tid * num_experts + expert_id] + cumsum[expert_id];
    sorted_token_ids[rank_post_pad] = i;
    ++tokens_cnts[tid * num_experts + expert_id];
  }
}

void moe_align_block_size(
    torch::Tensor topk_ids,
    int64_t num_experts,
    int64_t block_size,
    torch::Tensor sorted_token_ids,
    torch::Tensor experts_ids,
    torch::Tensor num_tokens_post_pad,
    torch::Tensor cumsum_buffer,
    bool pad_sorted_token_ids) {
  const cudaStream_t stream = at::cuda::getCurrentCUDAStream();

  int threads = 1024;
  threads = ((threads + WARP_SIZE - 1) / WARP_SIZE) * WARP_SIZE;
```
**EN:** This section implements `moe_align_block_size`, `getCurrentCUDAStream`, `stream`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`moe_align_block_size`、`getCurrentCUDAStream`、`stream`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 331-354: Kernel implementation
```cpp
  int64_t max_num_tokens_padded = sorted_token_ids.size(0);

  DISPATCH_INTEGRAL_TYPES(topk_ids.scalar_type(), "moe_align_block_size_kernel", [&] {
    bool small_batch_expert_mode = (topk_ids.numel() < 1024) && (num_experts <= 64);

    if (small_batch_expert_mode) {
      const int32_t threads = max((int32_t)num_experts, WARP_SIZE);
      constexpr int32_t fill_threads = 256;
      const int32_t shared_mem_size = ((threads + 1) * num_experts + (num_experts + 1)) * sizeof(int32_t);

      auto small_batch_expert_kernel = moe_align_block_size_small_batch_expert_kernel<scalar_t, fill_threads>;
      small_batch_expert_kernel<<<1, fill_threads + threads, shared_mem_size, stream>>>(
          topk_ids.data_ptr<scalar_t>(),
          sorted_token_ids.data_ptr<int32_t>(),
          experts_ids.data_ptr<int32_t>(),
          num_tokens_post_pad.data_ptr<int32_t>(),
          num_experts,
          block_size,
          topk_ids.numel(),
          pad_sorted_token_ids,
          max_num_tokens_padded);
    } else {
      auto align_kernel = moe_align_block_size_kernel<scalar_t>;
```
**EN:** This section implements `numel`, `max`, `threads`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`numel`、`max`、`threads`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 355-381: Kernel implementation
```cpp
      const size_t scan_size = next_pow2(num_experts);
      const size_t shared_mem_size = (num_experts + (num_experts + 1) + scan_size + WARP_SIZE) * sizeof(int32_t);
      align_kernel<<<2, threads, shared_mem_size, stream>>>(
          topk_ids.data_ptr<scalar_t>(),
          sorted_token_ids.data_ptr<int32_t>(),
          experts_ids.data_ptr<int32_t>(),
          num_tokens_post_pad.data_ptr<int32_t>(),
          num_experts,
          block_size,
          topk_ids.numel(),
          cumsum_buffer.data_ptr<int32_t>(),
          pad_sorted_token_ids,
          scan_size,
          max_num_tokens_padded);

      const int block_threads = std::min(256, (int)threads);
      const int num_blocks = (topk_ids.numel() + block_threads - 1) / block_threads;
      const int max_blocks = 65535;
      const int actual_blocks = std::min(num_blocks, max_blocks);

      auto sort_kernel = count_and_sort_expert_tokens_kernel<scalar_t>;
      sort_kernel<<<actual_blocks, block_threads, 0, stream>>>(
          topk_ids.data_ptr<scalar_t>(),
          sorted_token_ids.data_ptr<int32_t>(),
          cumsum_buffer.data_ptr<int32_t>(),
          topk_ids.numel());
    }
```
**EN:** This section implements `next_pow2`, `numel`, `min`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`next_pow2`、`numel`、`min`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 382-383: Local implementation details
```cpp
  });
}
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **HIP runtime / HIP 运行时**: Uses HIP APIs or AMD-specific intrinsics for portability. / 使用 HIP API 或 AMD 特定 intrinsic 实现可移植性。
- **Synchronization / atomics / 同步与原子操作**: Uses atomics or explicit synchronization to coordinate parallel work. / 使用原子操作或显式同步协调并行工作。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `utils.h`
- **External headers / 外部头文件**: `ATen/ATen.h`, `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDAGuard.h`, `THC/THCAtomics.cuh`
- **Path context / 路径上下文**: moe / moe_align_kernel.cu
