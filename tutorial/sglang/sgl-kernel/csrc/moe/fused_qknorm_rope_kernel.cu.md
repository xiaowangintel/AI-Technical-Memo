# fused_qknorm_rope_kernel.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/moe/fused_qknorm_rope_kernel.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements mixture-of-experts routing, combination, or expert computation utilities. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 实现 MoE 的路由、聚合或专家计算辅助逻辑。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28: Headers and compile-time setup
```cpp
/*
 * Copyright (c) 2025, NVIDIA CORPORATION.  All rights reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
// Adapted from
// https://github.com/NVIDIA/TensorRT-LLM/blob/main/cpp/tensorrt_llm/kernels/fusedQKNormRopeKernel.cu

#include <ATen/cuda/Exceptions.h>
#include <c10/cuda/CUDAGuard.h>
#include <c10/cuda/CUDAStream.h>
#include <cuda_bf16.h>
#include <cuda_fp16.h>
#include <cuda_fp8.h>
#include <cuda_runtime.h>
#include <torch/all.h>
#include <torch/cuda.h>
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 29-51: Types and data layout
```cpp
#include <cmath>

#define CHECK_TYPE(x, st) \
  TORCH_CHECK(x.scalar_type() == st, #x " dtype is ", x.scalar_type(), ", while ", st, " is expected")
#define CHECK_TH_CUDA(x) TORCH_CHECK(x.is_cuda(), #x " must be a CUDA tensor")
#define CHECK_CONTIGUOUS(x) TORCH_CHECK(x.is_contiguous(), #x " must be contiguous")
#define CHECK_INPUT(x, st) \
  CHECK_TH_CUDA(x);        \
  CHECK_CONTIGUOUS(x);     \
  CHECK_TYPE(x, st)

#define FINAL_MASK 0xffffffff

namespace tensorrt_llm::common {
template <typename T, int num>
struct packed_as;

// Specialization for packed_as used in this kernel.
template <>
struct packed_as<uint, 1> {
  using type = uint;
};
```
**EN:** This section defines `packed_as`, `CHECK_TYPE`, `CHECK_CONTIGUOUS`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`packed_as`、`CHECK_TYPE`、`CHECK_CONTIGUOUS`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 52-75: Types and data layout
```cpp
template <>
struct packed_as<uint, 2> {
  using type = uint2;
};

template <>
struct packed_as<uint, 4> {
  using type = uint4;
};

template <typename T>
__inline__ __device__ T warpReduceSum(T val) {
#pragma unroll
  for (int mask = 16; mask > 0; mask >>= 1)
    val += __shfl_xor_sync(FINAL_MASK, val, mask,
                           32);  //__shfl_sync bf16 return float when sm < 80
  return val;
}

template <typename T>
inline __device__ __host__ T divUp(T m, T n) {
  return (m + n - 1) / n;
}
```
**EN:** This section defines `packed_as`, `warpReduceSum`, `divUp`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`packed_as`、`warpReduceSum`、`divUp`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 76-100: Namespace and shared declarations
```cpp
}  // namespace tensorrt_llm::common
namespace tensorrt_llm::kernels {

__device__ inline float compute_freq_yarn(float base, int head_dim, int half_dim, float factor, float low, float high) {
  float freq = powf(base, -2.0f * half_dim / static_cast<float>(head_dim));

  if (factor != 1.0f) {
    float inv_freq_extrapolation = freq;
    float inv_freq_interpolation = freq / factor;

    float high_adj = high;
    if (fabsf(low - high_adj) <= 1e-6f) {
      high_adj += 0.001f;
    }

    float linear_func = (static_cast<float>(half_dim) - low) / (high_adj - low);
    float ramp_func = fminf(fmaxf(linear_func, 0.0f), 1.0f);
    float inv_freq_extrapolation_factor = 1.0f - ramp_func;

    freq = inv_freq_interpolation * (1.0f - inv_freq_extrapolation_factor) +
           inv_freq_extrapolation * inv_freq_extrapolation_factor;
  }

  return freq;
}
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 101-128: Kernel implementation
```cpp

////////////////////////////////////////////////////////////////////////////////////////////////////

// Perform per-head QK Norm and RoPE in a single kernel.
// head_dim: the dimension of each head
// interleave: interleave=!is_neox.
template <int head_dim, bool interleave>
__global__ void fusedQKNormRopeKernel(
    __nv_bfloat16* qkv,             // Combined QKV tensor [num_tokens, (num_heads_q+num_heads_k+num_heads_v)*head_dim]
    int const num_heads_q,          // Number of query heads
    int const num_heads_k,          // Number of key heads
    int const num_heads_v,          // Number of value heads
    float const eps,                // Epsilon for RMS normalization
    __nv_bfloat16 const* q_weight,  // RMSNorm weights for query
    __nv_bfloat16 const* k_weight,  // RMSNorm weights for key
    float const base,               // Base for RoPE computation
    int const* position_ids,        // Position IDs for RoPE
    int const num_tokens,           // Number of tokens
    // parameters for yarn
    float factor,  // factor in rope_scaling in config.json. When it is not 1.0, it means the model is using yarn.
    float low,     // threshold for high frequency
    float high,    // threshold for low frequency
    float attention_factor,  // attention_factor applied on cos and sin
    int const rotary_dim) {
  int const warpsPerBlock = blockDim.x / 32;
  int const warpId = threadIdx.x / 32;
  int const laneId = threadIdx.x % 32;
```
**EN:** This section implements `fusedQKNormRopeKernel`, `warpsPerBlock`, `warpId`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`fusedQKNormRopeKernel`、`warpsPerBlock`、`warpId`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 129-155: Device helpers and synchronization
```cpp
  // Calculate global warp index to determine which head/token this warp processes
  int const globalWarpIdx = blockIdx.x * warpsPerBlock + warpId;

  // Total number of attention heads (Q and K)
  int const total_qk_heads = num_heads_q + num_heads_k;

  // Determine which token and head type (Q or K) this warp processes
  int const tokenIdx = globalWarpIdx / total_qk_heads;
  int const localHeadIdx = globalWarpIdx % total_qk_heads;

  // Skip if this warp is assigned beyond the number of tokens
  if (tokenIdx >= num_tokens) return;

  bool const isQ = localHeadIdx < num_heads_q;
  int const headIdx = isQ ? localHeadIdx : localHeadIdx - num_heads_q;
  int const num_heads = num_heads_q + num_heads_k + num_heads_v;
  static_assert(
      head_dim % (32 * 2) == 0,
      "head_dim must be divisible by 64 (each warp processes one head, and each thread gets even number of "
      "elements)");
  constexpr int numElemsPerThread = head_dim / 32;
  float elements[numElemsPerThread];
  constexpr int elemSizeBytes = numElemsPerThread * sizeof(__nv_bfloat16);
  static_assert(elemSizeBytes % 4 == 0, "numSizeBytes must be a multiple of 4");
  constexpr int vecSize = elemSizeBytes / 4;  // Use packed_as<uint, vecSize> to perform loading/saving.
  using vec_T = typename tensorrt_llm::common::packed_as<uint, vecSize>::type;
```
**EN:** This section implements `static_assert`, `globalWarpIdx`, `total_qk_heads`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`static_assert`、`globalWarpIdx`、`total_qk_heads`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 156-179: Device helpers and synchronization
```cpp
  int offsetWarp;  // Offset for the warp
  if (isQ) {
    // Q segment: token offset + head offset within Q segment
    offsetWarp = tokenIdx * num_heads * head_dim + headIdx * head_dim;
  } else {
    // K segment: token offset + entire Q segment + head offset within K segment
    offsetWarp = tokenIdx * num_heads * head_dim + num_heads_q * head_dim + headIdx * head_dim;
  }
  int offsetThread = offsetWarp + laneId * numElemsPerThread;

  // Sum of squares for RMSNorm
  float sumOfSquares = 0.0f;

  // Load.
  {
    vec_T vec = *reinterpret_cast<vec_T const*>(&qkv[offsetThread]);
    for (int i = 0; i < vecSize; i++) {
      float2 vals = __bfloat1622float2(*reinterpret_cast<__nv_bfloat162*>(reinterpret_cast<uint*>(&vec) + i));
      sumOfSquares += vals.x * vals.x;
      sumOfSquares += vals.y * vals.y;
      elements[2 * i] = vals.x;
      elements[2 * i + 1] = vals.y;
    }
  }
```
**EN:** This section implements `__bfloat1622float2`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`__bfloat1622float2`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 180-205: Device helpers and synchronization
```cpp

  // Reduce sum across warp using the utility function
  sumOfSquares = tensorrt_llm::common::warpReduceSum(sumOfSquares);

  // Compute RMS normalization factor
  float rms_rcp = rsqrtf(sumOfSquares / static_cast<float>(head_dim) + eps);

  // Normalize elements
  for (int i = 0; i < numElemsPerThread; i++) {
    int dim = laneId * numElemsPerThread + i;
    float weight = isQ ? __bfloat162float(q_weight[dim]) : __bfloat162float(k_weight[dim]);
    elements[i] *= rms_rcp * weight;
  }
  // Apply RoPE to normalized elements
  float elements2[numElemsPerThread];  // Additional buffer required for RoPE.
  float cos_vals[numElemsPerThread];
  float sin_vals[numElemsPerThread];
  float pos_id = static_cast<float>(position_ids[tokenIdx]);
  int const rotary_lanes = rotary_dim / numElemsPerThread;  // rotary range
  bool const applyRotary = (laneId < rotary_lanes);
  if (applyRotary) {
    if constexpr (interleave) {
      // Perform interleaving. Fill cos_vals and sin_vals.
      for (int i = 0; i < numElemsPerThread; i++) {
        elements2[i] = (i % 2 == 0) ? -elements[i + 1] : elements[i - 1];
```
**EN:** This section implements `warpReduceSum`, `rsqrtf`, `__bfloat162float`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`warpReduceSum`、`rsqrtf`、`__bfloat162float`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 206-226: Device helpers and synchronization
```cpp
        int dim_idx = laneId * numElemsPerThread + i;
        int half_dim = dim_idx / 2;
        float freq = compute_freq_yarn(base, rotary_dim, half_dim, factor, low, high);
        float theta = pos_id * freq;
        __sincosf(theta, &sin_vals[i], &cos_vals[i]);
      }

    } else {
      // Neox style
      // Before data exchange with in warp, we need to sync.
      __syncwarp();
      int const half_rotary_lanes = rotary_lanes / 2;
      unsigned int active_mask = (1u << rotary_lanes) - 1;
      // Limitation: The operation below requires half_rotary_lanes to be a power of 2.
      // because it relies on __shfl_xor_sync to exchange data within a warp.
      for (int i = 0; i < numElemsPerThread; i++) {
        elements2[i] = __shfl_xor_sync(active_mask, elements[i], half_rotary_lanes);
        if (laneId < half_rotary_lanes) {
          elements2[i] = -elements2[i];
        }
```
**EN:** This section implements `compute_freq_yarn`, `__sincosf`, `__syncwarp`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`compute_freq_yarn`、`__sincosf`、`__syncwarp`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 227-249: Device helpers and synchronization
```cpp
        int dim_idx = laneId * numElemsPerThread + i;
        dim_idx = (dim_idx * 2) % rotary_dim;
        int half_dim = dim_idx / 2;
        float freq = compute_freq_yarn(base, rotary_dim, half_dim, factor, low, high);
        float theta = pos_id * freq;
        __sincosf(theta, &sin_vals[i], &cos_vals[i]);
      }
      // __shfl_xor_sync does not provide memfence. Need to sync again.
      __syncwarp();
    }

    for (int i = 0; i < numElemsPerThread; i++) {
      elements[i] = (elements[i] * cos_vals[i] + elements2[i] * sin_vals[i]) * attention_factor;
    }
  }

  // Store.
  {
    vec_T vec;
    for (int i = 0; i < vecSize; i++) {
      __nv_bfloat162 vals = __float22bfloat162_rn(make_float2(elements[2 * i], elements[2 * i + 1]));
      reinterpret_cast<__nv_bfloat162&>(*(reinterpret_cast<uint*>(&vec) + i)) = vals;
    }
```
**EN:** This section implements `compute_freq_yarn`, `__sincosf`, `__syncwarp`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`compute_freq_yarn`、`__sincosf`、`__syncwarp`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 250-273: Control flow and branching
```cpp
    vec_T* outputPtr = reinterpret_cast<vec_T*>(&qkv[offsetThread]);
    *outputPtr = vec;
  }
}
// Borrowed from
// https://github.com/flashinfer-ai/flashinfer/blob/8125d079a43e9a0ba463a4ed1b639cefd084cec9/include/flashinfer/pos_enc.cuh#L568
#define DISPATCH_INTERLEAVE(interleave, INTERLEAVE, ...) \
  if (interleave) {                                      \
    const bool INTERLEAVE = true;                        \
    __VA_ARGS__                                          \
  } else {                                               \
    const bool INTERLEAVE = false;                       \
    __VA_ARGS__                                          \
  }
void launchFusedQKNormRope(
    void* qkv,
    int const num_tokens,
    int const num_heads_q,
    int const num_heads_k,
    int const num_heads_v,
    int const head_dim,
    float const eps,
    void const* q_weight,
    void const* k_weight,
```
**EN:** This section drives `DISPATCH_INTERLEAVE`, `INTERLEAVE` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`DISPATCH_INTERLEAVE`、`INTERLEAVE`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 274-297: Kernel implementation
```cpp
    float const base,
    bool const interleave,
    int const* position_ids,
    float factor,
    float low,
    float high,
    float attention_factor,
    int const rotary_dim,
    cudaStream_t stream) {
  constexpr int blockSize = 256;
  int const warpsPerBlock = blockSize / 32;
  int const totalQKHeads = num_heads_q + num_heads_k;
  int const totalWarps = num_tokens * totalQKHeads;
  int const gridSize = common::divUp(totalWarps, warpsPerBlock);
  dim3 gridDim(gridSize);
  dim3 blockDim(blockSize);
  // Head dimensions should be a multiple of 64
  // Add more cases as needed
  switch (head_dim) {
    case 64:
      DISPATCH_INTERLEAVE(interleave, INTERLEAVE, {
        fusedQKNormRopeKernel<64, INTERLEAVE><<<gridDim, blockDim, 0, stream>>>(
            reinterpret_cast<__nv_bfloat16*>(qkv),
            num_heads_q,
```
**EN:** This section implements `divUp`, `gridDim`, `blockDim`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`divUp`、`gridDim`、`blockDim`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 298-321: Kernel implementation
```cpp
            num_heads_k,
            num_heads_v,
            eps,
            reinterpret_cast<__nv_bfloat16 const*>(q_weight),
            reinterpret_cast<__nv_bfloat16 const*>(k_weight),
            base,
            position_ids,
            num_tokens,
            factor,
            low,
            high,
            attention_factor,
            rotary_dim);
      });
      break;
    case 128:
      DISPATCH_INTERLEAVE(interleave, INTERLEAVE, {
        fusedQKNormRopeKernel<128, INTERLEAVE><<<gridDim, blockDim, 0, stream>>>(
            reinterpret_cast<__nv_bfloat16*>(qkv),
            num_heads_q,
            num_heads_k,
            num_heads_v,
            eps,
            reinterpret_cast<__nv_bfloat16 const*>(q_weight),
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 322-345: Kernel implementation
```cpp
            reinterpret_cast<__nv_bfloat16 const*>(k_weight),
            base,
            position_ids,
            num_tokens,
            factor,
            low,
            high,
            attention_factor,
            rotary_dim);
      });
      break;
    case 256:
      DISPATCH_INTERLEAVE(interleave, INTERLEAVE, {
        fusedQKNormRopeKernel<256, INTERLEAVE><<<gridDim, blockDim, 0, stream>>>(
            reinterpret_cast<__nv_bfloat16*>(qkv),
            num_heads_q,
            num_heads_k,
            num_heads_v,
            eps,
            reinterpret_cast<__nv_bfloat16 const*>(q_weight),
            reinterpret_cast<__nv_bfloat16 const*>(k_weight),
            base,
            position_ids,
            num_tokens,
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 346-369: Runtime integration and dispatch
```cpp
            factor,
            low,
            high,
            attention_factor,
            rotary_dim);
      });
      break;
    default:
      TORCH_CHECK(false, "Unsupported head dimension for fusedQKNormRope: ", head_dim);
  }
}
}  // namespace tensorrt_llm::kernels

// Function for fused QK Norm and RoPE
// This operator applies RMS normalization and RoPE to Q and K tensors in a single CUDA kernel.
// The OP performs operations in-place on the input qkv tensor.
void fused_qk_norm_rope(
    torch::Tensor& qkv,           // Combined QKV tensor [num_tokens, (num_heads_q+num_heads_k+num_heads_v)*head_dim]
    int64_t num_heads_q,          // Number of query heads
    int64_t num_heads_k,          // Number of key heads
    int64_t num_heads_v,          // Number of value heads
    int64_t head_dim,             // Dimension per head
    double eps,                   // Epsilon for RMS normalization
    torch::Tensor& q_weight,      // RMSNorm weights for query [head_dim]
```
**EN:** This section uses `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 370-394: Runtime integration and dispatch
```cpp
    torch::Tensor& k_weight,      // RMSNorm weights for key [head_dim]
    double base,                  // Base for RoPE computation
    bool is_neox,                 // Whether RoPE is applied in Neox style
    torch::Tensor& position_ids,  // Position IDs for RoPE [num_tokens]
    // parameters for yarn
    double factor,  // factor in rope_scaling in config.json. When it is not 1.0, it means the model is using yarn.
    double low,     // threshold for high frequency
    double high,    // threshold for low frequency
    double attention_factor,  // attention_factor applied on cos and sin
    int64_t rotary_dim) {
  // Input validation
  TORCH_CHECK(qkv.dim() == 2, "QKV tensor must be 2D: [num_tokens, (num_heads_q+num_heads_k+num_heads_v)*head_dim]");
  TORCH_CHECK(position_ids.dim() == 1, "Position IDs must be 1D: [num_tokens]");
  TORCH_CHECK(q_weight.dim() == 1, "Query weights must be 1D: [head_dim]");
  TORCH_CHECK(k_weight.dim() == 1, "Key weights must be 1D: [head_dim]");
  TORCH_CHECK(q_weight.size(0) == head_dim, "Query weights size must match head dimension");
  TORCH_CHECK(k_weight.size(0) == head_dim, "Key weights size must match head dimension");
  TORCH_CHECK(rotary_dim % (head_dim / 32) == 0, "rotary_dim must be divisible by numElemsPerThread");
  if (is_neox) {
    int64_t half_rotary_lanes = rotary_dim / (head_dim / 32) / 2;
    TORCH_CHECK(
        half_rotary_lanes >= 1 && (half_rotary_lanes & (half_rotary_lanes - 1)) == 0,
        "half_rotary_lanes must be a power of 2 for neox style, got ",
        half_rotary_lanes);
  }
```
**EN:** This section uses `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 395-418: Runtime integration and dispatch
```cpp
  CHECK_INPUT(qkv, torch::kBFloat16);
  CHECK_INPUT(position_ids, torch::kInt32);
  CHECK_INPUT(q_weight, torch::kBFloat16);
  CHECK_INPUT(k_weight, torch::kBFloat16);

  int64_t num_tokens = qkv.size(0);
  TORCH_CHECK(position_ids.size(0) == num_tokens, "Number of tokens in position_ids must match QKV");

  int64_t total_heads = num_heads_q + num_heads_k + num_heads_v;
  TORCH_CHECK(
      qkv.size(1) == total_heads * head_dim, "QKV tensor size must match total number of heads and head dimension");

  auto stream = at::cuda::getCurrentCUDAStream(qkv.get_device());

  tensorrt_llm::kernels::launchFusedQKNormRope(
      reinterpret_cast<__nv_bfloat16*>(qkv.data_ptr()),
      static_cast<int>(num_tokens),
      static_cast<int>(num_heads_q),
      static_cast<int>(num_heads_k),
      static_cast<int>(num_heads_v),
      static_cast<int>(head_dim),
      static_cast<float>(eps),
      reinterpret_cast<__nv_bfloat16*>(q_weight.data_ptr()),
      reinterpret_cast<__nv_bfloat16*>(k_weight.data_ptr()),
```
**EN:** This section uses `CHECK_INPUT`, `TORCH_CHECK`, `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`CHECK_INPUT`、`TORCH_CHECK`、`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 419-428: Local implementation details
```cpp
      static_cast<float>(base),
      !is_neox,  // interleave
      reinterpret_cast<int const*>(position_ids.data_ptr()),
      static_cast<float>(factor),
      static_cast<float>(low),
      static_cast<float>(high),
      static_cast<float>(attention_factor),
      static_cast<int>(rotary_dim),
      stream);
}
```
**EN:** This section fills in the local implementation details around `data_ptr`, completing the behavior required by the file.
**CN:** 本段补充了`data_ptr`周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: None explicitly listed / 未显式列出
- **External headers / 外部头文件**: `ATen/cuda/Exceptions.h`, `c10/cuda/CUDAGuard.h`, `c10/cuda/CUDAStream.h`, `cuda_bf16.h`, `cuda_fp16.h`, `cuda_fp8.h`, `cuda_runtime.h`, `torch/all.h`, `torch/cuda.h`, `cmath`
- **Path context / 路径上下文**: moe / fused_qknorm_rope_kernel.cu
