# prepare_moe_input.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/moe/prepare_moe_input.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements mixture-of-experts routing, combination, or expert computation utilities. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 实现 MoE 的路由、聚合或专家计算辅助逻辑。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Kernel implementation
```cpp
#include <c10/cuda/CUDAGuard.h>
#include <cudaTypedefs.h>
#include <torch/all.h>

#include <flashinfer/vec_dtypes.cuh>
#include <iostream>

#include "cutlass/array.h"
#include "utils.h"

constexpr uint64_t THREADS_PER_EXPERT = 512;

__global__ void compute_problem_sizes(
    const int* __restrict__ topk_ids,
    int32_t* problem_sizes1,
    int32_t* problem_sizes2,
    int32_t* atomic_buffer,
    const int64_t topk_length,
    const int64_t n,
    const int64_t k) {
  int expert_id = blockIdx.x;

  int occurrences = 0;
  for (int i = threadIdx.x; i < topk_length; i += THREADS_PER_EXPERT) {
    occurrences += (topk_ids[i] == expert_id);
  }
```
**EN:** This section implements `compute_problem_sizes`, `THREADS_PER_EXPERT`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`compute_problem_sizes`、`THREADS_PER_EXPERT`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 27-52: Kernel implementation
```cpp
  atomicAdd(&atomic_buffer[expert_id], occurrences);
  __syncthreads();

  if (threadIdx.x == 0) {
    int final_occurrences = atomic_buffer[expert_id];
    problem_sizes1[expert_id * 3] = final_occurrences;
    problem_sizes1[expert_id * 3 + 1] = static_cast<int32_t>(2 * n);
    problem_sizes1[expert_id * 3 + 2] = static_cast<int32_t>(k);
    problem_sizes2[expert_id * 3] = final_occurrences;
    problem_sizes2[expert_id * 3 + 1] = static_cast<int32_t>(k);
    problem_sizes2[expert_id * 3 + 2] = static_cast<int32_t>(n);
  }
}

__global__ void compute_expert_offsets(
    const int32_t* __restrict__ problem_sizes1,
    int32_t* expert_offsets,
    int32_t* atomic_buffer,
    const int64_t num_experts) {
  int32_t tot_offset = 0;
  expert_offsets[0] = 0;
  for (int i = 0; i < num_experts; ++i) {
    atomic_buffer[i] = tot_offset;
    tot_offset += problem_sizes1[i * 3];
    expert_offsets[i + 1] = tot_offset;
  }
```
**EN:** This section implements `compute_expert_offsets`, `atomicAdd`, `__syncthreads`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`compute_expert_offsets`、`atomicAdd`、`__syncthreads`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 53-75: Kernel implementation
```cpp
}

__global__ void compute_expert_blockscale_offsets(
    const int32_t* __restrict__ problem_sizes1,
    int32_t* expert_offsets,
    int32_t* blockscale_offsets,
    int32_t* atomic_buffer,
    const int64_t num_experts) {
  int32_t tot_offset = 0;
  int32_t tot_rounded_offset = 0;
  expert_offsets[0] = 0;
  blockscale_offsets[0] = 0;
  for (int i = 0; i < num_experts; ++i) {
    atomic_buffer[i] = tot_offset;
    int num_tokens = problem_sizes1[i * 3];
    int rounded_num_tokens = (num_tokens + (128 - 1)) / 128 * 128;
    tot_offset += num_tokens;
    tot_rounded_offset += rounded_num_tokens;
    expert_offsets[i + 1] = tot_offset;
    blockscale_offsets[i + 1] = tot_rounded_offset;
  }
}
```
**EN:** This section implements `compute_expert_blockscale_offsets`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`compute_expert_blockscale_offsets`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 76-93: Kernel implementation
```cpp
__global__ void compute_arg_sorts(
    const int32_t* __restrict__ topk_ids,
    int32_t* input_permutation,
    int32_t* output_permutation,
    int32_t* atomic_buffer,
    const int64_t topk_length,
    const int64_t topk) {
  int expert_id = blockIdx.x;

  for (int i = threadIdx.x; i < topk_length; i += THREADS_PER_EXPERT) {
    if (topk_ids[i] == expert_id) {
      int start = atomicAdd(&atomic_buffer[expert_id], 1);
      input_permutation[start] = i / topk;
      output_permutation[i] = start;
    }
  }
}
```
**EN:** This section implements `compute_arg_sorts`, `atomicAdd`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`compute_arg_sorts`、`atomicAdd`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 94-111: Device helpers and synchronization
```cpp
void get_moe_prepare_input_caller(
    const torch::Tensor& topk_ids,
    torch::Tensor& expert_offsets,
    const std::optional<torch::Tensor>& blockscale_offsets,
    torch::Tensor& problem_sizes1,
    torch::Tensor& problem_sizes2,
    torch::Tensor& input_permutation,
    torch::Tensor& output_permutation,
    const int64_t num_experts,
    const int64_t n,
    const int64_t k) {
  auto stream = at::cuda::getCurrentCUDAStream(topk_ids.device().index());
  auto options_int32 = torch::TensorOptions().dtype(torch::kInt32).device(topk_ids.device());
  torch::Tensor atomic_buffer = torch::zeros(num_experts, options_int32);

  uint32_t num_threads = static_cast<uint32_t>(min(THREADS_PER_EXPERT, topk_ids.numel()));
  uint32_t num_blocks = static_cast<uint32_t>(num_experts);
```
**EN:** This section implements `get_moe_prepare_input_caller`, `getCurrentCUDAStream`, `TensorOptions`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`get_moe_prepare_input_caller`、`getCurrentCUDAStream`、`TensorOptions`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 112-133: Kernel implementation
```cpp
  compute_problem_sizes<<<num_blocks, num_threads, 0, stream>>>(
      static_cast<const int32_t*>(topk_ids.data_ptr()),
      static_cast<int32_t*>(problem_sizes1.data_ptr()),
      static_cast<int32_t*>(problem_sizes2.data_ptr()),
      static_cast<int32_t*>(atomic_buffer.data_ptr()),
      topk_ids.numel(),
      n,
      k);
  if (blockscale_offsets.has_value()) {
    compute_expert_blockscale_offsets<<<1, 1, 0, stream>>>(
        static_cast<const int32_t*>(problem_sizes1.data_ptr()),
        static_cast<int32_t*>(expert_offsets.data_ptr()),
        static_cast<int32_t*>(blockscale_offsets.value().data_ptr()),
        static_cast<int32_t*>(atomic_buffer.data_ptr()),
        num_experts);
  } else {
    compute_expert_offsets<<<1, 1, 0, stream>>>(
        static_cast<const int32_t*>(problem_sizes1.data_ptr()),
        static_cast<int32_t*>(expert_offsets.data_ptr()),
        static_cast<int32_t*>(atomic_buffer.data_ptr()),
        num_experts);
  }
```
**EN:** This section implements `data_ptr`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`data_ptr`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 134-157: Kernel implementation
```cpp
  compute_arg_sorts<<<num_blocks, num_threads, 0, stream>>>(
      static_cast<const int32_t*>(topk_ids.data_ptr()),
      static_cast<int32_t*>(input_permutation.data_ptr()),
      static_cast<int32_t*>(output_permutation.data_ptr()),
      static_cast<int32_t*>(atomic_buffer.data_ptr()),
      topk_ids.numel(),
      topk_ids.size(1));
}

void prepare_moe_input(
    const torch::Tensor& topk_ids,
    torch::Tensor& expert_offsets,
    const std::optional<torch::Tensor>& blockscale_offsets,
    torch::Tensor& problem_sizes1,
    torch::Tensor& problem_sizes2,
    torch::Tensor& input_permutation,
    torch::Tensor& output_permutation,
    const int64_t num_experts,
    const int64_t n,
    const int64_t k) {
  TORCH_CHECK(topk_ids.dtype() == torch::kInt32);
  get_moe_prepare_input_caller(
      topk_ids,
      expert_offsets,
```
**EN:** This section implements `prepare_moe_input`, `data_ptr`, `TORCH_CHECK`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`prepare_moe_input`、`data_ptr`、`TORCH_CHECK`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 158-179: Kernel implementation
```cpp
      blockscale_offsets,
      problem_sizes1,
      problem_sizes2,
      input_permutation,
      output_permutation,
      num_experts,
      n,
      k);
  return;
}

template <typename T>
__global__ void shuffleRowsKernel(
    const T* input,
    const int32_t* dst2src_map,
    T* output,
    int64_t num_src_rows,
    int64_t num_dst_rows,
    int64_t num_cols) {
  int64_t dest_row_idx = blockIdx.x;
  int64_t const source_row_idx = dst2src_map[dest_row_idx];
```
**EN:** This section implements `shuffleRowsKernel`, `source_row_idx`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`shuffleRowsKernel`、`source_row_idx`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 180-207: Kernel implementation
```cpp
  if (blockIdx.x < num_dst_rows) {
    // Load 128-bits per thread
    constexpr uint64_t ELEM_PER_THREAD = 128 / sizeof(T) / 8;
    using DataElem = cutlass::Array<T, ELEM_PER_THREAD>;

    // Duplicate and permute rows
    auto const* source_row_ptr = reinterpret_cast<DataElem const*>(input + source_row_idx * num_cols);
    auto* dest_row_ptr = reinterpret_cast<DataElem*>(output + dest_row_idx * num_cols);

    auto const start_offset = threadIdx.x;
    auto const stride = blockDim.x;
    auto const num_elems_in_col = num_cols / ELEM_PER_THREAD;

    for (auto elem_index = start_offset; elem_index < num_elems_in_col; elem_index += stride) {
      dest_row_ptr[elem_index] = source_row_ptr[elem_index];
    }
  }
}

#define DECLARE_SHUFFLE_ROWS(T)      \
  __global__ void shuffleRowsKernel( \
      const T* input,                \
      const int32_t* dst2src_map,    \
      T* output,                     \
      int64_t num_src_rows,          \
      int64_t num_dest_rows,         \
      int64_t num_cols);
```
**EN:** This section implements `DECLARE_SHUFFLE_ROWS`, `ELEM_PER_THREAD`, `DataElem`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`DECLARE_SHUFFLE_ROWS`、`ELEM_PER_THREAD`、`DataElem`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 208-227: Kernel implementation
```cpp
DECLARE_SHUFFLE_ROWS(float);
DECLARE_SHUFFLE_ROWS(half);
DECLARE_SHUFFLE_ROWS(__nv_bfloat16);
DECLARE_SHUFFLE_ROWS(__nv_fp8_e4m3);
DECLARE_SHUFFLE_ROWS(uint8_t);

#define SHUFFLE_ROWS(T)                                    \
  shuffleRowsKernel<T><<<blocks, threads, 0, stream>>>(    \
      reinterpret_cast<const T*>(input),                   \
      static_cast<const int32_t*>(dst2src_map.data_ptr()), \
      reinterpret_cast<T*>(output),                        \
      num_src_rows,                                        \
      num_dst_rows,                                        \
      num_cols)

#define DTYPE_DISPATCH_CASE(T, CUDA_T) \
  case T:                              \
    SHUFFLE_ROWS(CUDA_T);              \
    break;
```
**EN:** This section implements `DECLARE_SHUFFLE_ROWS`, `SHUFFLE_ROWS`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`DECLARE_SHUFFLE_ROWS`、`SHUFFLE_ROWS`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 228-251: Runtime integration and dispatch
```cpp
void shuffle_rows_caller(
    const torch::Tensor& input_tensor, const torch::Tensor& dst2src_map, torch::Tensor& output_tensor) {
  TORCH_CHECK(
      input_tensor.scalar_type() == output_tensor.scalar_type(),
      "Input and output tensors must have the same data type");
  auto stream = at::cuda::getCurrentCUDAStream().stream();
  uint32_t blocks = static_cast<uint32_t>(output_tensor.size(0));
  uint32_t threads = 256;
  int64_t num_dst_rows = output_tensor.size(0);
  int64_t num_src_rows = input_tensor.size(0);
  int64_t num_cols = input_tensor.size(1);
  const void* input = input_tensor.data_ptr();
  void* output = output_tensor.data_ptr();
  switch (input_tensor.scalar_type()) {
    DTYPE_DISPATCH_CASE(torch::kFloat16, half);
    DTYPE_DISPATCH_CASE(torch::kBFloat16, __nv_bfloat16);
    DTYPE_DISPATCH_CASE(torch::kFloat32, float);
    DTYPE_DISPATCH_CASE(torch::kFloat8_e4m3fn, __nv_fp8_e4m3);
    DTYPE_DISPATCH_CASE(torch::kUInt8, uint8_t);
    default:
      TORCH_CHECK(false, "[moe replicate input] data type dispatch fail!");
  }
  return;
}
```
**EN:** This section uses `shuffle_rows_caller`, `TORCH_CHECK`, `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`shuffle_rows_caller`、`TORCH_CHECK`、`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 252-278: Kernel implementation
```cpp

void shuffle_rows(const torch::Tensor& input_tensor, const torch::Tensor& dst2src_map, torch::Tensor& output_tensor) {
  shuffle_rows_caller(input_tensor, dst2src_map, output_tensor);
  return;
}

template <typename scalar_t>
__global__ void apply_shuffle_mul_sum_kernel(
    const scalar_t* __restrict__ input_tensor,  // [m * topk, k] (expert-major layout)
    scalar_t* __restrict__ output_tensor,       // [m, k] (token-major layout)
    const int32_t* __restrict__ permutation,    // [m * topk] (c_map: token-major-idx -> expert-major-idx)
    int m,
    int topk,
    int row_stride,
    const scalar_t* __restrict__ factors)  // [m * topk] (topk_weights, token-major layout)
{
  int i = blockIdx.x;
  if (i >= m) {
    return;
  }

  constexpr uint32_t vec_size = 16 / sizeof(scalar_t);
  using t = float;
  using vec_t = flashinfer::vec_t<t, vec_size>;
  int thread_idx = threadIdx.x;
  int stride = blockDim.x;
```
**EN:** This section implements `shuffle_rows`, `apply_shuffle_mul_sum_kernel`, `shuffle_rows_caller`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`shuffle_rows`、`apply_shuffle_mul_sum_kernel`、`shuffle_rows_caller`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 279-302: Control flow and branching
```cpp
  for (int d_vec_idx = thread_idx; d_vec_idx < row_stride / vec_size; d_vec_idx += stride) {
    int d = d_vec_idx * vec_size;
    vec_t sum_vec;
    sum_vec.fill(0.0f);

    for (int j = 0; j < topk; ++j) {
      int token_major_idx = i * topk + j;
      int src_row = permutation[token_major_idx];

      vec_t val_vec;
      val_vec.cast_load(input_tensor + src_row * row_stride + d);

      t factor = 1.0;
      if (factors != nullptr) {
        factor = factors[token_major_idx];
      }

#pragma unroll
      for (int k = 0; k < vec_size; ++k) {
        sum_vec[k] += factor * val_vec[k];
      }
    }
    sum_vec.cast_store(output_tensor + i * row_stride + d);
  }
```
**EN:** This section drives `fill`, `cast_load`, `cast_store` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`fill`、`cast_load`、`cast_store`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 303-322: Control flow and branching
```cpp

  //  remainder part
  int remainder_start = (row_stride / vec_size) * vec_size;
  for (int d = remainder_start + thread_idx; d < row_stride; d += stride) {
    t sum_val = 0.0;
    for (int j = 0; j < topk; ++j) {
      int token_major_idx = i * topk + j;
      int src_row = permutation[token_major_idx];
      t val = input_tensor[src_row * row_stride + d];

      t factor = 1.0;
      if (factors != nullptr) {
        factor = factors[token_major_idx];
      }
      sum_val += factor * val;
    }
    output_tensor[i * row_stride + d] = sum_val;
  }
}
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 323-346: Runtime integration and dispatch
```cpp
void get_apply_shuffle_mul_sum_caller(
    const torch::Tensor& input_tensor,                // [m * topk, row_stride], bf16/f16
    torch::Tensor& output_tensor,                     // [m, row_stride], bf16/f16
    const torch::Tensor& permutation,                 // [m * topk], int32
    const std::optional<torch::Tensor>& factors_opt)  // optional [m * topk], bf16/f16
{
  TORCH_CHECK(input_tensor.dim() == 2, "input_tensor must be 2D [m * topk, row_stride]");
  TORCH_CHECK(output_tensor.dim() == 2, "output_tensor must be 2D [m, row_stride]");
  TORCH_CHECK(permutation.dim() == 1, "permutation must be 1D [m * topk]");

  int m = output_tensor.size(0);
  int topk = int(permutation.size(0) / m);
  int row_stride = output_tensor.size(1);

  TORCH_CHECK(permutation.size(0) == m * topk, "permutation size must match m * topk");

  auto scalar_type = output_tensor.scalar_type();
  uint32_t vec_size = 16 / sizeof(scalar_type);
  auto blockDim = std::min(row_stride / vec_size, 1024U);
  dim3 block(blockDim);

  dim3 grid(m);  // blockIdx.x = j, blockIdx.y = i
  auto stream = at::cuda::getCurrentCUDAStream(input_tensor.device().index());
```
**EN:** This section uses `get_apply_shuffle_mul_sum_caller`, `TORCH_CHECK`, `int` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`get_apply_shuffle_mul_sum_caller`、`TORCH_CHECK`、`int`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 347-368: Kernel implementation
```cpp
  const int32_t* perm_ptr = permutation.data_ptr<int32_t>();

  void* factors_ptr = nullptr;
  if (factors_opt.has_value()) {
    TORCH_CHECK(factors_opt->dtype() == output_tensor.dtype(), "Factors must match output dtype");
    TORCH_CHECK(factors_opt->numel() == m * topk, "Factors must have shape [m * topk]");
    factors_ptr = factors_opt->data_ptr();
  }

  DISPATCH_PYTORCH_DTYPE_TO_CTYPE_FLOAT_FP16(output_tensor.scalar_type(), scalar_t, [&] {
    apply_shuffle_mul_sum_kernel<scalar_t><<<grid, block, 0, stream>>>(
        static_cast<const scalar_t*>(input_tensor.data_ptr()),
        static_cast<scalar_t*>(output_tensor.data_ptr()),
        perm_ptr,
        m,
        topk,
        row_stride,
        static_cast<const scalar_t*>(factors_ptr));
    return true;
  });
}
```
**EN:** This section implements `TORCH_CHECK`, `data_ptr`, `perm_ptr`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`TORCH_CHECK`、`data_ptr`、`perm_ptr`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 369-392: Runtime integration and dispatch
```cpp
/**
 * @brief Applies a permutation-based shuffle, element-wise multiplication, and reduction over the second dimension.
 *
 * This function performs the equivalent of the following PyTorch expression:
 *
 *     (c2[c_map].view(m, topk, k) * topk_weights.view(m, topk, 1).to(out_dtype)).sum(dim=1)
 *
 * Specifically:
 * - `input` is shuffled using the `permutation` tensor.
 * - The shuffled tensor is reshaped and multiplied element-wise with `factors` (e.g., top-k weights).
 * - The result is summed along dimension 1 (the top-k dimension), and stored in `output`.
 *
 * @param input        Input tensor of shape (m * topk, k), representing c2.
 * @param output       Output tensor of shape (m, k), where the final reduced results are stored.
 * @param permutation  Index tensor (e.g., c_map) that maps positions in `input` to shuffled layout.
 * @param factors      Optional scaling factors (e.g., top-k weights), shape (m * topk) or (m, topk).
 */
void apply_shuffle_mul_sum(
    const torch::Tensor& input,
    torch::Tensor& output,
    const torch::Tensor& permutation,
    const std::optional<torch::Tensor>& factors) {
  get_apply_shuffle_mul_sum_caller(input, output, permutation, factors);
}
```
**EN:** This section uses `apply_shuffle_mul_sum`, `get_apply_shuffle_mul_sum_caller` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`apply_shuffle_mul_sum`、`get_apply_shuffle_mul_sum_caller`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。
- **Synchronization / atomics / 同步与原子操作**: Uses atomics or explicit synchronization to coordinate parallel work. / 使用原子操作或显式同步协调并行工作。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cutlass/array.h`, `utils.h`
- **External headers / 外部头文件**: `c10/cuda/CUDAGuard.h`, `cudaTypedefs.h`, `torch/all.h`, `flashinfer/vec_dtypes.cuh`, `iostream`
- **Path context / 路径上下文**: moe / prepare_moe_input.cu
