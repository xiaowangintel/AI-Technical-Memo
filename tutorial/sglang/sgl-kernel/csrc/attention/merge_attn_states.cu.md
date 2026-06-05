# merge_attn_states.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/attention/merge_attn_states.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements attention-related kernels, layouts, or dispatch helpers for high-throughput inference. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 实现面向高吞吐推理的 attention 内核、数据布局或分发辅助逻辑。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Device helpers and synchronization
```cpp
#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAGuard.h>

#include <algorithm>
#include <optional>

#include "pytorch_extension_utils.h"

// Helper functions to convert between different data types
// (float, half, bfloat16) for the merge attention states kernel.
inline __device__ float to_float(float u) {
  return u;
}
inline __device__ float to_float(half u) {
  return __half2float(u);
}
inline __device__ float to_float(__nv_bfloat16 u) {
  return __bfloat162float(u);
}
inline __device__ void from_float(float& d, float s) {
  d = s;
}
```
**EN:** This section implements `to_float`, `from_float`, `__half2float`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`to_float`、`from_float`、`__half2float`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 23-45: Kernel implementation
```cpp
inline __device__ void from_float(half& d, float s) {
  d = __float2half(s);
}
inline __device__ void from_float(__nv_bfloat16& d, float s) {
  d = __float2bfloat16(s);
}

// Implements section 2.2 of https://www.arxiv.org/pdf/2501.01005
template <typename scalar_t, const uint NUM_THREADS>
__global__ void merge_attn_states_kernel(
    scalar_t* output,
    float* output_lse,
    const scalar_t* prefix_output,
    const float* prefix_lse,
    const scalar_t* suffix_output,
    const float* suffix_lse,
    const uint num_tokens,
    const uint num_heads,
    const uint head_size) {
  using pack_128b_t = uint4;
  const uint pack_size = 16 / sizeof(scalar_t);
  const uint threads_per_head = head_size / pack_size;
```
**EN:** This section implements `from_float`, `merge_attn_states_kernel`, `__float2half`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`from_float`、`merge_attn_states_kernel`、`__float2half`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 46-70: Device helpers and synchronization
```cpp
  const uint global_idx = blockIdx.x * NUM_THREADS + threadIdx.x;
  const uint token_head_threads = num_tokens * num_heads * threads_per_head;

  if (global_idx >= token_head_threads) return;

  // global_idx -> token_idx + head_idx + pack_idx
  const uint token_head_idx = global_idx / threads_per_head;
  const uint pack_idx = global_idx % threads_per_head;

  const uint token_idx = token_head_idx / num_heads;
  const uint head_idx = token_head_idx % num_heads;

  const uint pack_offset = pack_idx * pack_size;  // (0~15)*8, etc.
  const uint head_offset = token_idx * num_heads * head_size + head_idx * head_size;
  const scalar_t* prefix_head_ptr = prefix_output + head_offset;
  const scalar_t* suffix_head_ptr = suffix_output + head_offset;
  scalar_t* output_head_ptr = output + head_offset;

  // float p_lse = prefix_lse[head_idx * num_tokens + token_idx];
  // float s_lse = suffix_lse[head_idx * num_tokens + token_idx];
  float p_lse = prefix_lse[token_idx * num_heads + head_idx];
  float s_lse = suffix_lse[token_idx * num_heads + head_idx];
  p_lse = std::isinf(p_lse) ? -std::numeric_limits<float>::infinity() : p_lse;
  s_lse = std::isinf(s_lse) ? -std::numeric_limits<float>::infinity() : s_lse;
```
**EN:** This section implements `global_idx`, `token_head_threads`, `token_head_idx`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`global_idx`、`token_head_threads`、`token_head_idx`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 71-96: Control flow and branching
```cpp
  const float max_lse = fmaxf(p_lse, s_lse);
  p_lse = p_lse - max_lse;
  s_lse = s_lse - max_lse;
  const float p_se = expf(p_lse);
  const float s_se = expf(s_lse);
  const float out_se = p_se + s_se;
  const float p_scale = p_se / out_se;
  const float s_scale = s_se / out_se;

  if (pack_offset < head_size) {
    // Pack 128b load
    pack_128b_t p_out_pack = reinterpret_cast<const pack_128b_t*>(prefix_head_ptr)[pack_offset / pack_size];
    pack_128b_t s_out_pack = reinterpret_cast<const pack_128b_t*>(suffix_head_ptr)[pack_offset / pack_size];
    pack_128b_t o_out_pack;

#pragma unroll
    for (uint i = 0; i < pack_size; ++i) {
      // Always use float for FMA to keep high precision.
      // half(uint16_t), bfloat16, float -> float.
      const float p_out_f = to_float(reinterpret_cast<const scalar_t*>(&p_out_pack)[i]);
      const float s_out_f = to_float(reinterpret_cast<const scalar_t*>(&s_out_pack)[i]);
      // fma: a * b + c = p_out_f * p_scale + (s_out_f * s_scale)
      const float o_out_f = p_out_f * p_scale + (s_out_f * s_scale);
      // float -> half(uint16_t), bfloat16, float.
      from_float(reinterpret_cast<scalar_t*>(&o_out_pack)[i], o_out_f);
    }
```
**EN:** This section drives `fmaxf`, `expf`, `to_float` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`fmaxf`、`expf`、`to_float`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 97-122: Runtime integration and dispatch
```cpp

    // Pack 128b storage
    reinterpret_cast<pack_128b_t*>(output_head_ptr)[pack_offset / pack_size] = o_out_pack;
  }
  // We only need to write to output_lse once per head.
  if (output_lse != nullptr && pack_idx == 0) {
    float out_lse = logf(out_se) + max_lse;
    output_lse[token_idx * num_heads + head_idx] = out_lse;
  }
}

// The following macro is used to dispatch the conversion function based on
// the output data type. The FN is a macro that calls a function with
// template<typename scalar_t>.
#define DISPATCH_BY_SCALAR_DTYPE(scalar_dtype, fn)                      \
  {                                                                     \
    if (scalar_dtype == at::ScalarType::Float) {                        \
      fn(float);                                                        \
    } else if (scalar_dtype == at::ScalarType::Half) {                  \
      fn(half);                                                         \
    } else if (scalar_dtype == at::ScalarType::BFloat16) {              \
      fn(__nv_bfloat16);                                                \
    } else {                                                            \
      TORCH_CHECK(false, "Unsupported data type of O: ", scalar_dtype); \
    }                                                                   \
  }
```
**EN:** This section uses `fn`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fn`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 123-144: Kernel implementation
```cpp

#define LAUNCH_MERGE_ATTN_STATES(scalar_t, NUM_THREADS)                          \
  {                                                                              \
    merge_attn_states_kernel<scalar_t, NUM_THREADS><<<grid, block, 0, stream>>>( \
        reinterpret_cast<scalar_t*>(output.data_ptr()),                          \
        reinterpret_cast<float*>(output_lse.data_ptr()),                         \
        reinterpret_cast<scalar_t*>(prefix_output.data_ptr()),                   \
        reinterpret_cast<float*>(prefix_lse.data_ptr()),                         \
        reinterpret_cast<scalar_t*>(suffix_output.data_ptr()),                   \
        reinterpret_cast<float*>(suffix_lse.data_ptr()),                         \
        num_tokens,                                                              \
        num_heads,                                                               \
        head_size);                                                              \
  }

/*@brief Merges the attention states from prefix and suffix
 * into the output tensor. NUM_TOKENS: n, NUM_HEADS: h, HEAD_SIZE: d
 *
 * @param output [n,h,d] The output tensor to store the merged attention states.
 * @param output_lse [h,d] Optional tensor to store the log-sum-exp values.
 * @param prefix_output [n,h,d] The prefix attention states.
 * @param prefix_lse [n,h] The log-sum-exp values for the prefix attention
```
**EN:** This section implements `data_ptr`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`data_ptr`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 145-169: Runtime integration and dispatch
```cpp
 * states.
 * @param suffix_output [n,h,d] The suffix attention states.
 * @param suffix_lse [n,h] The log-sum-exp values for the suffix attention
 * states.
 */
template <typename scalar_t>
void merge_attn_states_launcher(
    const at::Tensor& prefix_output,  // [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    const at::Tensor& prefix_lse,     // [NUM_TOKENS, NUM_HEADS]
    const at::Tensor& suffix_output,  // [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    const at::Tensor& suffix_lse,     // [NUM_TOKENS, NUM_HEADS]
    at::Tensor& output,               // [NUM_TOKENS, NUM_HEADS, HEAD_SIZE]
    at::Tensor& output_lse            // [NUM_TOKENS, NUM_HEADS]
) {
  constexpr uint NUM_THREADS = 128;
  const uint num_tokens = output.size(0);
  const uint num_heads = output.size(1);
  const uint head_size = output.size(2);
  const uint pack_size = 16 / sizeof(scalar_t);
  TORCH_CHECK(head_size % pack_size == 0, "headsize must be multiple of pack_size:", pack_size);
  // Process one pack elements per thread. for float, the
  // pack_size is 4 for half/bf16, the pack_size is 8.
  const uint threads_per_head = head_size / pack_size;
  const uint total_threads = num_tokens * num_heads * threads_per_head;
```
**EN:** This section uses `merge_attn_states_launcher`, `TORCH_CHECK`, `NUM_THREADS` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`merge_attn_states_launcher`、`TORCH_CHECK`、`NUM_THREADS`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 170-191: Runtime integration and dispatch
```cpp
  dim3 block(NUM_THREADS);
  dim3 grid((total_threads + NUM_THREADS - 1) / NUM_THREADS);

  const c10::cuda::OptionalCUDAGuard device_guard(prefix_output.device());
  auto stream = at::cuda::getCurrentCUDAStream();

  LAUNCH_MERGE_ATTN_STATES(scalar_t, NUM_THREADS);
}

#define CALL_MERGE_ATTN_STATES_LAUNCHER(scalar_t)                                 \
  {                                                                               \
    merge_attn_states_launcher<scalar_t>(v_a, s_a, v_b, s_b, v_merged, s_merged); \
  }

void merge_state_v2(
    at::Tensor v_a, at::Tensor s_a, at::Tensor v_b, at::Tensor s_b, at::Tensor v_merged, at::Tensor s_merged) {
  // Input tensors must be contiguous
  CHECK_INPUT(v_a);  // v_a prefix_output (seq_len, num_heads, head_dim)
  CHECK_INPUT(s_a);  // s_a prefix_lse (seq_len, num_heads)
  CHECK_INPUT(v_b);  // v_b suffix_output (seq_len, num_heads, head_dim)
  CHECK_INPUT(s_b);  // s_b suffix_lse (seq_len, num_heads)
  // v_merged output (seq_len, num_heads, head_dim)
```
**EN:** This section uses `merge_state_v2`, `block`, `grid` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`merge_state_v2`、`block`、`grid`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 192-206: Local implementation details
```cpp
  // s_merged output_lse (seq_len, num_heads)
  auto device = v_a.device();
  CHECK_EQ(s_a.device(), device);
  CHECK_EQ(v_b.device(), device);
  CHECK_EQ(s_b.device(), device);
  CHECK_DIM(3, v_a);
  CHECK_DIM(2, s_a);
  CHECK_DIM(3, v_b);
  CHECK_DIM(2, s_b);
  CHECK_SHAPE(v_a, v_b);
  CHECK_SHAPE(s_a, s_b);
  CHECK_EQ(v_a.size(0), s_a.size(0));
  CHECK_EQ(v_a.size(1), s_b.size(1));
  DISPATCH_BY_SCALAR_DTYPE(v_merged.dtype(), CALL_MERGE_ATTN_STATES_LAUNCHER);
}
```
**EN:** This section fills in the local implementation details around `device`, `CHECK_EQ`, `CHECK_DIM`, completing the behavior required by the file.
**CN:** 本段补充了`device`、`CHECK_EQ`、`CHECK_DIM`周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `pytorch_extension_utils.h`
- **External headers / 外部头文件**: `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDAGuard.h`, `algorithm`, `optional`
- **Path context / 路径上下文**: attention / merge_attn_states.cu
