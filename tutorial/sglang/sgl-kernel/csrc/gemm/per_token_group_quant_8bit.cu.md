# per_token_group_quant_8bit.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/per_token_group_quant_8bit.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Device helpers and synchronization
```cpp
#include <ATen/cuda/CUDAContext.h>
#include <cuda_fp8.h>

#include <cmath>
#include <flashinfer/vec_dtypes.cuh>

#include "utils.h"

__device__ __forceinline__ float GroupReduceMax(float val, const int tid) {
  unsigned mask = threadIdx.x % 32 >= 16 ? 0xffff0000 : 0x0000ffff;

  val = fmaxf(val, __shfl_xor_sync(mask, val, 8));
  val = fmaxf(val, __shfl_xor_sync(mask, val, 4));
  val = fmaxf(val, __shfl_xor_sync(mask, val, 2));
  val = fmaxf(val, __shfl_xor_sync(mask, val, 1));
  return val;
}
```
**EN:** This section implements `GroupReduceMax`, `fmaxf`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`GroupReduceMax`、`fmaxf`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 19-40: Kernel implementation
```cpp
template <
    typename T,
    typename DST_DTYPE,
    bool IS_COLUMN_MAJOR = false,
    bool SCALE_UE8M0 = false,
    typename scale_packed_t = std::conditional_t<SCALE_UE8M0, uint32_t, float>>
__global__ void per_token_group_quant_8bit_kernel(
    const T* __restrict__ input,
    void* __restrict__ output_q,
    scale_packed_t* __restrict__ output_s,
    const int group_size,
    const int num_groups,
    const int groups_per_block,
    const float eps,
    const float min_8bit,
    const float max_8bit,
    const int num_groups_per_row = 0,
    const int scale_stride = 0) {
  const int threads_per_group = 16;
  const int64_t local_group_id = threadIdx.x / threads_per_group;
  const int lane_id = threadIdx.x % threads_per_group;
```
**EN:** This section implements `per_token_group_quant_8bit_kernel`, `num_groups_per_row`, `scale_stride`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`per_token_group_quant_8bit_kernel`、`num_groups_per_row`、`scale_stride`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 41-65: Device helpers and synchronization
```cpp
  const int64_t block_group_id = blockIdx.x * groups_per_block;
  const int64_t global_group_id = block_group_id + local_group_id;
  const int64_t block_group_offset = global_group_id * group_size;

  float local_absmax = eps;

  using scale_element_t = std::conditional_t<SCALE_UE8M0, uint8_t, float>;
  static_assert(sizeof(scale_packed_t) % sizeof(scale_element_t) == 0);

  const T* group_input = input + block_group_offset;
  DST_DTYPE* group_output = static_cast<DST_DTYPE*>(output_q) + block_group_offset;
  scale_element_t* scale_output;

  if constexpr (IS_COLUMN_MAJOR) {
    const int num_elems_per_pack = static_cast<int>(sizeof(scale_packed_t) / sizeof(scale_element_t));
    const int row_idx = global_group_id / num_groups_per_row;
    const int col_idx_unpacked = global_group_id % num_groups_per_row;
    const int col_idx = col_idx_unpacked / num_elems_per_pack;
    const int pack_idx = col_idx_unpacked % num_elems_per_pack;
    scale_output = reinterpret_cast<scale_element_t*>(output_s) +
                   (col_idx * scale_stride * num_elems_per_pack + row_idx * num_elems_per_pack + pack_idx);
  } else {
    static_assert(!SCALE_UE8M0);
    scale_output = output_s + global_group_id;
  }
```
**EN:** This section implements `static_assert`, `block_group_id`, `global_group_id`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`static_assert`、`block_group_id`、`global_group_id`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 66-89: Templates, aliases, and constants
```cpp

  constexpr uint32_t vec_size = 16 / sizeof(T);
  using vec_t = flashinfer::vec_t<T, vec_size>;

  const int32_t num_vec_elems = group_size / vec_size;

  for (int32_t i = lane_id; i < num_vec_elems; i += 16) {
    vec_t input_vec;
    input_vec.cast_load(group_input + i * vec_size);

#pragma unroll
    for (uint32_t j = 0; j < vec_size; ++j) {
      float val = static_cast<float>(input_vec[j]);
      float abs_val = fabsf(val);
      local_absmax = fmaxf(local_absmax, abs_val);
    }
  }

  local_absmax = GroupReduceMax(local_absmax, lane_id);

  float y_s = local_absmax / max_8bit;
  if constexpr (SCALE_UE8M0) {
    y_s = exp2f(ceilf(log2f(fmaxf(y_s, 1e-10f))));
  }
```
**EN:** This section defines `cast_load`, `fabsf`, `fmaxf`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`cast_load`、`fabsf`、`fmaxf`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 90-112: Templates, aliases, and constants
```cpp

  // TODO can optimize
  scale_element_t y_s_quant;
  if constexpr (SCALE_UE8M0) {
    y_s_quant = (uint8_t)(((int)log2f(y_s)) + 127);
  } else {
    y_s_quant = y_s;
  }

  if (lane_id == 0) {
    *scale_output = y_s_quant;
  }

  for (int32_t i = lane_id; i < num_vec_elems; i += 16) {
    vec_t input_vec;
    input_vec.cast_load(group_input + i * vec_size);

#pragma unroll
    for (uint32_t j = 0; j < vec_size; ++j) {
      float val = static_cast<float>(input_vec[j]);
      float q_val = fminf(fmaxf(val / y_s, min_8bit), max_8bit);
      group_output[i * vec_size + j] = DST_DTYPE(q_val);
    }
```
**EN:** This section defines `log2f`, `cast_load`, `fminf`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`log2f`、`cast_load`、`fminf`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 113-134: Runtime integration and dispatch
```cpp
  }
}

void sgl_per_token_group_quant_8bit(
    torch::Tensor input,
    torch::Tensor output_q,
    torch::Tensor output_s,
    int64_t group_size,
    double eps,
    double min_8bit,
    double max_8bit,
    bool scale_ue8m0) {
  CHECK_INPUT(input);
  CHECK_INPUT(output_q);

  const int num_groups = input.numel() / group_size;

  CHECK_EQ(input.numel() % group_size, 0);
  CHECK_EQ(output_s.dim(), 2);

  cudaStream_t stream = at::cuda::getCurrentCUDAStream();
```
**EN:** This section uses `sgl_per_token_group_quant_8bit`, `CHECK_INPUT`, `CHECK_EQ` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`sgl_per_token_group_quant_8bit`、`CHECK_INPUT`、`CHECK_EQ`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 135-157: Templates, aliases, and constants
```cpp
  constexpr int THREADS_PER_GROUP = 16;

  int groups_per_block = 1;

  if (num_groups % 16 == 0) {
    groups_per_block = 16;
  } else if (num_groups % 8 == 0) {
    groups_per_block = 8;
  } else if (num_groups % 4 == 0) {
    groups_per_block = 4;
  } else if (num_groups % 2 == 0) {
    groups_per_block = 2;
  }

  auto dst_type = output_q.scalar_type();
  const int num_blocks = num_groups / groups_per_block;
  const int num_threads = groups_per_block * THREADS_PER_GROUP;

  const bool is_column_major = output_s.stride(0) < output_s.stride(1);
  const int hidden_dim = input.size(input.dim() - 1);
  const int num_groups_per_row = hidden_dim / group_size;
  const int scale_stride = output_s.stride(1);
```
**EN:** This section defines `scalar_type`, `stride`, `THREADS_PER_GROUP`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`scalar_type`、`stride`、`THREADS_PER_GROUP`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 158-179: Kernel implementation
```cpp
#define LAUNCH_KERNEL(T, DST_DTYPE)                                                               \
  do {                                                                                            \
    dim3 grid(num_blocks);                                                                        \
    dim3 block(num_threads);                                                                      \
    if (is_column_major) {                                                                        \
      if (scale_ue8m0) {                                                                          \
        per_token_group_quant_8bit_kernel<T, DST_DTYPE, true, true><<<grid, block, 0, stream>>>(  \
            static_cast<T*>(input.data_ptr()),                                                    \
            output_q.data_ptr(),                                                                  \
            static_cast<uint32_t*>(output_s.data_ptr()),                                          \
            group_size,                                                                           \
            num_groups,                                                                           \
            groups_per_block,                                                                     \
            (float)eps,                                                                           \
            (float)min_8bit,                                                                      \
            (float)max_8bit,                                                                      \
            num_groups_per_row,                                                                   \
            scale_stride);                                                                        \
      } else {                                                                                    \
        per_token_group_quant_8bit_kernel<T, DST_DTYPE, true, false><<<grid, block, 0, stream>>>( \
            static_cast<T*>(input.data_ptr()),                                                    \
            output_q.data_ptr(),                                                                  \
```
**EN:** This section implements `grid`, `block`, `data_ptr`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`grid`、`block`、`data_ptr`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 180-204: Kernel implementation
```cpp
            static_cast<float*>(output_s.data_ptr()),                                             \
            group_size,                                                                           \
            num_groups,                                                                           \
            groups_per_block,                                                                     \
            (float)eps,                                                                           \
            (float)min_8bit,                                                                      \
            (float)max_8bit,                                                                      \
            num_groups_per_row,                                                                   \
            scale_stride);                                                                        \
      }                                                                                           \
    } else {                                                                                      \
      assert(!scale_ue8m0);                                                                       \
      per_token_group_quant_8bit_kernel<T, DST_DTYPE, false><<<grid, block, 0, stream>>>(         \
          static_cast<T*>(input.data_ptr()),                                                      \
          output_q.data_ptr(),                                                                    \
          static_cast<float*>(output_s.data_ptr()),                                               \
          group_size,                                                                             \
          num_groups,                                                                             \
          groups_per_block,                                                                       \
          (float)eps,                                                                             \
          (float)min_8bit,                                                                        \
          (float)max_8bit);                                                                       \
    }                                                                                             \
  } while (0)
```
**EN:** This section implements `data_ptr`, `assert`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`data_ptr`、`assert`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 205-217: Runtime integration and dispatch
```cpp
  DISPATCH_PYTORCH_DTYPE_TO_CTYPE_FLOAT_FP16(input.scalar_type(), scalar_t, [&] {
    if (dst_type == at::ScalarType::Char) {
      LAUNCH_KERNEL(scalar_t, int8_t);
      return true;
    } else if (dst_type == at::ScalarType::Float8_e4m3fn) {
      LAUNCH_KERNEL(scalar_t, __nv_fp8_e4m3);
      return true;
    }
    return false;
  });

#undef LAUNCH_KERNEL
}
```
**EN:** This section uses `LAUNCH_KERNEL` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`LAUNCH_KERNEL`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Device helpers / 设备侧辅助函数**: Uses inline device routines to structure low-level operations. / 使用内联设备函数组织底层操作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `utils.h`
- **External headers / 外部头文件**: `ATen/cuda/CUDAContext.h`, `cuda_fp8.h`, `cmath`, `flashinfer/vec_dtypes.cuh`
- **Path context / 路径上下文**: gemm / per_token_group_quant_8bit.cu
