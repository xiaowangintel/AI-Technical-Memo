# apply_token_bitmask_inplace_cuda.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/grammar/apply_token_bitmask_inplace_cuda.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Local implementation details
```cpp
// Adapted from
// https://github.com/mlc-ai/xgrammar/blob/v0.1.18/python/xgrammar/kernels/apply_token_bitmask_inplace_cuda.cu

/*
 * SPDX-FileCopyrightText: Copyright (c) 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: Apache-2.0
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 21-40: Headers and compile-time setup
```cpp
// clang-format off
#include <cuda_bf16.h>
#include <cuda_fp16.h>
#include <cuda_runtime.h>
#include <torch/all.h>
#include <ATen/cuda/CUDAContext.h>


#if !defined(USE_ROCM) && (!defined(CUDA_VERSION) || CUDA_VERSION < 12040)
void ApplyTokenBitmaskInplace(at::Tensor logits, at::Tensor bitmask, at::optional<at::Tensor> indices = at::nullopt) {
  TORCH_CHECK(false, "CUDA version must be >= 12.4 for ApplyTokenBitmaskInplace");
}
#else

#ifndef CUDART_INF_FP16
#ifndef USE_ROCM
#define CUDART_INF_FP16 __ushort_as_half((unsigned short)0x7C00U)
#endif
#endif
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 41-62: Device helpers and synchronization
```cpp
#ifndef CUDART_INF_BF16
#ifndef USE_ROCM
#define CUDART_INF_BF16 __ushort_as_bfloat16((unsigned short)0x7F80U)
#endif
#endif

constexpr int32_t BITS_PER_BLOCK = 32;
constexpr int32_t THREADS_PER_THREAD_BLOCK = 256;

template <typename T>
__device__ T NegativeInfinity() {
  return -INFINITY;
}

template <>
__device__ __half NegativeInfinity<__half>() {
#ifdef USE_ROCM
  return __float2half(-INFINITY);
#else
  return -CUDART_INF_FP16;
#endif
}
```
**EN:** This section implements `NegativeInfinity`, `__float2half`, `BITS_PER_BLOCK`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`NegativeInfinity`、`__float2half`、`BITS_PER_BLOCK`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 63-83: Device helpers and synchronization
```cpp

template <>
__device__ __nv_bfloat16 NegativeInfinity<__nv_bfloat16>() {
#ifdef USE_ROCM
  return __nv_bfloat16(-INFINITY);
#else
  return -CUDART_INF_BF16;
#endif
}

template <typename T, typename PackedT>
__device__ PackedT PackedNegativeInfinity() {
  constexpr int kAlignment = sizeof(PackedT) / sizeof(T);
  T packed[kAlignment];
#pragma unroll
  for (int i = 0; i < kAlignment; i++) {
    packed[i] = NegativeInfinity<T>();
  }
  return *reinterpret_cast<PackedT*>(packed);
}
```
**EN:** This section implements `PackedNegativeInfinity`, `__nv_bfloat16`, `kAlignment`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`PackedNegativeInfinity`、`__nv_bfloat16`、`kAlignment`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 84-108: Kernel implementation
```cpp
template <typename T, typename PackedT, int32_t kBitsPerThread>
__global__ void __launch_bounds__(THREADS_PER_THREAD_BLOCK) LogitsBitmaskKernel(
    T* __restrict__ logits,
    const int32_t* __restrict__ bitmask,
    const int32_t* __restrict__ indices,
    int32_t vocab_size,
    int32_t logits_stride,
    int32_t bitmask_stride) {
  constexpr int kAlignment = sizeof(PackedT) / sizeof(T);
  constexpr uint32_t kPackedMask = (1 << kAlignment) - 1;

  const int batch_idx = (indices == nullptr) ? blockIdx.y : indices[blockIdx.y];

  const int block_offset = blockIdx.x * THREADS_PER_THREAD_BLOCK * kBitsPerThread;
  T* logits_gmem_ptr = logits + batch_idx * logits_stride + block_offset;
  const int32_t* bitmask_gmem_ptr = bitmask + batch_idx * bitmask_stride + block_offset / BITS_PER_BLOCK;
  const int bitmask_inner_idx = threadIdx.x % (BITS_PER_BLOCK / kAlignment);
  T logits_reg[kAlignment];

#pragma unroll
  for (int offset = threadIdx.x * kAlignment; offset < THREADS_PER_THREAD_BLOCK * kBitsPerThread;
       offset += THREADS_PER_THREAD_BLOCK * kAlignment) {
    if (block_offset + offset >= vocab_size) {
      break;
    }
```
**EN:** This section implements `__launch_bounds__`, `kAlignment`, `kPackedMask`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`__launch_bounds__`、`kAlignment`、`kPackedMask`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 109-130: Control flow and branching
```cpp

    const uint32_t bitmask_val =
        (~bitmask_gmem_ptr[offset / BITS_PER_BLOCK] >> (bitmask_inner_idx * kAlignment)) & kPackedMask;

    if (bitmask_val == 0) {
      continue;
    }

    if (bitmask_val == kPackedMask) {
      *reinterpret_cast<PackedT*>(logits_gmem_ptr + offset) = PackedNegativeInfinity<T, PackedT>();
      continue;
    }

    *reinterpret_cast<PackedT*>(logits_reg) = *reinterpret_cast<PackedT*>(logits_gmem_ptr + offset);
#pragma unroll
    for (int i = 0; i < kAlignment; i++) {
      if (((bitmask_val >> i) & 1)) {
        logits_reg[i] = NegativeInfinity<T>();
      }
    }
    *reinterpret_cast<PackedT*>(logits_gmem_ptr + offset) = *reinterpret_cast<PackedT*>(logits_reg);
  }
```
**EN:** This section drives `bitmask_val` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`bitmask_val`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 131-153: Runtime integration and dispatch
```cpp
}

template <typename T, typename = std::enable_if_t<std::is_integral<T>::value>>
constexpr auto CeilDiv(T numerator, T denominator) {
  return (numerator + denominator - 1) / denominator;
}

template <typename T, typename PackedT>
void ApplyTokenBitmaskInplaceDispatchToBitsPerThread(
    T* __restrict__ logits,
    const int32_t* __restrict__ bitmask,
    const int32_t* __restrict__ indices,
    int32_t vocab_size,
    int32_t logits_stride,
    int32_t bitmask_stride,
    int32_t num_rows) {
  constexpr int kAlignment = sizeof(PackedT) / sizeof(T);
  const int32_t num_blocks_per_row = CeilDiv(2048 / THREADS_PER_THREAD_BLOCK * 128, num_rows);
  const int32_t num_bits_per_thread = CeilDiv(vocab_size, THREADS_PER_THREAD_BLOCK * num_blocks_per_row);

  const dim3 block(THREADS_PER_THREAD_BLOCK);
  cudaStream_t stream = at::cuda::getCurrentCUDAStream().stream();
```
**EN:** This section uses `CeilDiv`, `ApplyTokenBitmaskInplaceDispatchToBitsPerThread`, `block` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CeilDiv`、`ApplyTokenBitmaskInplaceDispatchToBitsPerThread`、`block`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 154-172: Kernel implementation
```cpp
  if (num_bits_per_thread <= 4 && kAlignment <= 4) {
    const dim3 grid(CeilDiv(vocab_size, THREADS_PER_THREAD_BLOCK * 4), num_rows);
    LogitsBitmaskKernel<T, PackedT, 4>
        <<<grid, block, 0, stream>>>(logits, bitmask, indices, vocab_size, logits_stride, bitmask_stride);
  } else if (num_bits_per_thread <= 8 && kAlignment <= 8) {
    const dim3 grid(CeilDiv(vocab_size, THREADS_PER_THREAD_BLOCK * 8), num_rows);
    LogitsBitmaskKernel<T, PackedT, 8>
        <<<grid, block, 0, stream>>>(logits, bitmask, indices, vocab_size, logits_stride, bitmask_stride);
  } else if (num_bits_per_thread <= 16 && kAlignment <= 16) {
    const dim3 grid(CeilDiv(vocab_size, THREADS_PER_THREAD_BLOCK * 16), num_rows);
    LogitsBitmaskKernel<T, PackedT, 16>
        <<<grid, block, 0, stream>>>(logits, bitmask, indices, vocab_size, logits_stride, bitmask_stride);
  } else {
    const dim3 grid(CeilDiv(vocab_size, THREADS_PER_THREAD_BLOCK * 32), num_rows);
    LogitsBitmaskKernel<T, PackedT, 32>
        <<<grid, block, 0, stream>>>(logits, bitmask, indices, vocab_size, logits_stride, bitmask_stride);
  }
}
```
**EN:** This section implements `grid`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`grid`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 173-198: Runtime integration and dispatch
```cpp
template <typename T>
void ApplyTokenBitmaskInplaceDispatchToPackedT(
    T* __restrict__ logits,
    const int32_t* __restrict__ bitmask,
    const int32_t* __restrict__ indices,
    int32_t vocab_size,
    int32_t logits_stride,
    int32_t bitmask_stride,
    int32_t num_rows) {
  if (logits_stride % (sizeof(float4) / sizeof(T)) == 0) {
    ApplyTokenBitmaskInplaceDispatchToBitsPerThread<T, float4>(
        logits, bitmask, indices, vocab_size, logits_stride, bitmask_stride, num_rows);
  } else {
    ApplyTokenBitmaskInplaceDispatchToBitsPerThread<T, T>(
        logits, bitmask, indices, vocab_size, logits_stride, bitmask_stride, num_rows);
  }
}

void ApplyTokenBitmaskInplace(at::Tensor logits, at::Tensor bitmask, at::optional<at::Tensor> indices = at::nullopt) {
  TORCH_CHECK(logits.is_cuda(), "logits must be a CUDA tensor.");
  TORCH_CHECK(logits.is_contiguous(), "logits must be contiguous.");
  TORCH_CHECK(logits.dim() == 1 || logits.dim() == 2, "logits must be a 1D or 2D tensor.");
  std::pair<int32_t, int32_t> logits_shape =
      logits.dim() == 2 ? std::make_pair(static_cast<int32_t>(logits.size(0)), static_cast<int32_t>(logits.size(1)))
                        : std::make_pair(1, static_cast<int32_t>(logits.size(0)));
```
**EN:** This section uses `ApplyTokenBitmaskInplaceDispatchToPackedT`, `ApplyTokenBitmaskInplace`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`ApplyTokenBitmaskInplaceDispatchToPackedT`、`ApplyTokenBitmaskInplace`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 199-217: Runtime integration and dispatch
```cpp
  TORCH_CHECK(bitmask.is_cuda(), "bitmask must be a CUDA tensor.");
  TORCH_CHECK(bitmask.is_contiguous(), "bitmask must be contiguous.");
  TORCH_CHECK(bitmask.dim() == 1 || bitmask.dim() == 2, "bitmask must be a 1D or 2D tensor.");
  std::pair<int32_t, int32_t> bitmask_shape =
      bitmask.dim() == 2 ? std::make_pair(static_cast<int32_t>(bitmask.size(0)), static_cast<int32_t>(bitmask.size(1)))
                         : std::make_pair(1, static_cast<int32_t>(bitmask.size(0)));

  TORCH_CHECK(bitmask.dtype() == torch::kInt32, "bitmask must be of type int32.");

  TORCH_CHECK(
      (logits_shape.second + BITS_PER_BLOCK - 1) / BITS_PER_BLOCK >= bitmask_shape.second,
      "The provided logits's vocab size should be no less than the bitmask's vocab size "
      "(converted from bitmask size). But got vocab size ",
      logits_shape.second,
      " vs bitmask size ",
      bitmask_shape.second);

  int vocab_size = std::min(logits_shape.second, bitmask_shape.second * BITS_PER_BLOCK);
```
**EN:** This section uses `TORCH_CHECK`, `dim`, `min` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`dim`、`min`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 218-242: Runtime integration and dispatch
```cpp
  int32_t num_rows = logits_shape.first;
  int32_t* indices_ptr = nullptr;
  if (indices) {
    TORCH_CHECK(indices->is_cuda(), "indices must be a CUDA tensor.");
    TORCH_CHECK(indices->is_contiguous(), "indices must be contiguous.");
    TORCH_CHECK(indices->dim() == 1, "indices must be a 1D tensor.");
    TORCH_CHECK(indices->dtype() == torch::kInt32, "indices must be of type int32.");
    num_rows = indices->size(0);
    indices_ptr = indices->data_ptr<int32_t>();
  } else {
    TORCH_CHECK(logits_shape.first == bitmask_shape.first, "logits and bitmask must have the same batch size.");
  }

  switch (logits.scalar_type()) {
    case torch::kFloat32: {
      ApplyTokenBitmaskInplaceDispatchToPackedT(
          logits.data_ptr<float>(),
          bitmask.data_ptr<int32_t>(),
          indices_ptr,
          vocab_size,
          logits_shape.second,
          bitmask_shape.second,
          num_rows);
      break;
    }
```
**EN:** This section uses `TORCH_CHECK`, `ApplyTokenBitmaskInplaceDispatchToPackedT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`ApplyTokenBitmaskInplaceDispatchToPackedT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 243-264: Runtime integration and dispatch
```cpp
    case torch::kFloat16: {
      ApplyTokenBitmaskInplaceDispatchToPackedT(
          reinterpret_cast<__half*>(logits.data_ptr<torch::Half>()),
          bitmask.data_ptr<int32_t>(),
          indices_ptr,
          vocab_size,
          logits_shape.second,
          bitmask_shape.second,
          num_rows);
      break;
    }
    case torch::kBFloat16: {
      ApplyTokenBitmaskInplaceDispatchToPackedT(
          reinterpret_cast<__nv_bfloat16*>(logits.data_ptr<torch::BFloat16>()),
          bitmask.data_ptr<int32_t>(),
          indices_ptr,
          vocab_size,
          logits_shape.second,
          bitmask_shape.second,
          num_rows);
      break;
    }
```
**EN:** This section uses `ApplyTokenBitmaskInplaceDispatchToPackedT` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`ApplyTokenBitmaskInplaceDispatchToPackedT`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 265-269: Runtime integration and dispatch
```cpp
    default:
      TORCH_CHECK(false, "logits dtype must be float, half or bfloat16.");
      break;
  }
}
```
**EN:** This section uses `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 270-271: Local implementation details
```cpp
#endif
// clang-format on
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `cuda_bf16.h`, `cuda_fp16.h`, `cuda_runtime.h`, `torch/all.h`, `ATen/cuda/CUDAContext.h`
- **Path context / 路径上下文**: grammar / apply_token_bitmask_inplace_cuda.cu
