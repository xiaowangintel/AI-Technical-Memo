# w4a8_get_group_starts.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/moe/cutlass_moe/w4a8/w4a8_get_group_starts.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements mixture-of-experts routing, combination, or expert computation utilities. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 实现 MoE 的路由、聚合或专家计算辅助逻辑。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Kernel implementation
```cpp
#pragma once

#include <c10/cuda/CUDAStream.h>
#include <cuda.h>
#include <torch/all.h>

#include "cutlass/bfloat16.h"
#include "cutlass/float8.h"

template <typename ElementA, typename ElementB, typename ElementC, typename ElementAccumulator>
__global__ void int4_fp8_get_group_gemm_starts(
    int32_t* expert_offsets,
    ElementA** a_offsets,
    ElementB** b_offsets,
    ElementC** out_offsets,
    ElementAccumulator** a_scales_offsets,
    cutlass::bfloat16_t** b_scales_offsets,
    ElementA* a_base_as_int,
    ElementB* b_base_as_int,
    ElementC* out_base_as_int,
    ElementAccumulator* a_scales_base_as_int,
    cutlass::bfloat16_t* b_scales_base_as_int,
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 23-44: Kernel implementation
```cpp
    int64_t n,
    int64_t k,
    bool per_act_token,
    bool per_out_ch) {
  int expert_id = threadIdx.x;
  int32_t expert_offset = expert_offsets[expert_id];

  a_offsets[expert_id] = a_base_as_int + expert_offset * k;
  b_offsets[expert_id] = b_base_as_int + expert_id * k * n / 2;
  out_offsets[expert_id] = out_base_as_int + expert_offset * n;
  a_scales_offsets[expert_id] = a_scales_base_as_int + (per_act_token ? expert_offset : 0);
  b_scales_offsets[expert_id] = b_scales_base_as_int + (per_out_ch ? expert_id * n * k / 128 : expert_id);
}

template <typename ElementA, typename ElementB, typename ElementC, typename ElementAccumulator>
__global__ void int4_fp8_get_group_gemm_starts_3d(
    ElementA** a_offsets,
    ElementB** b_offsets,
    ElementC** out_offsets,
    ElementAccumulator** a_scales_offsets,
    cutlass::bfloat16_t** b_scales_offsets,
    ElementA* a_base_as_int,
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 45-69: Device helpers and synchronization
```cpp
    ElementB* b_base_as_int,
    ElementC* out_base_as_int,
    ElementAccumulator* a_scales_base_as_int,
    cutlass::bfloat16_t* b_scales_base_as_int,
    int64_t n,
    int64_t m,
    int64_t k,
    bool per_act_token,
    bool per_out_ch,
    int num_experts) {
  int expert_id = blockIdx.x * blockDim.x + threadIdx.x;
  if (expert_id >= num_experts) return;

  int64_t a_offset = expert_id * m * k;
  int64_t b_offset = expert_id * k * n / 2;
  int64_t out_offset = expert_id * m * n;
  int64_t a_scales_offset = 0;
  int64_t b_scales_offset = per_out_ch ? expert_id * n * 4 * k / 512 : expert_id;

  a_offsets[expert_id] = a_base_as_int + a_offset;
  b_offsets[expert_id] = b_base_as_int + b_offset;
  out_offsets[expert_id] = out_base_as_int + out_offset;
  a_scales_offsets[expert_id] = a_scales_base_as_int + a_scales_offset;
  b_scales_offsets[expert_id] = b_scales_base_as_int + b_scales_offset;
}
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 70-91: Kernel implementation
```cpp

#define __CALL_W4A8_GET_STARTS_KERNEL(TENSOR_C_TYPE, C_TYPE)                              \
  else if (out_tensors.dtype() == TENSOR_C_TYPE) {                                        \
    int4_fp8_get_group_gemm_starts<cutlass::float_e4m3_t, cutlass::int8_t, C_TYPE, float> \
        <<<1, num_experts, 0, stream>>>(                                                  \
            static_cast<int32_t*>(expert_offsets.data_ptr()),                             \
            static_cast<cutlass::float_e4m3_t**>(a_ptrs.data_ptr()),                      \
            static_cast<cutlass::int8_t**>(b_ptrs.data_ptr()),                            \
            static_cast<C_TYPE**>(out_ptrs.data_ptr()),                                   \
            static_cast<float**>(a_scales_ptrs.data_ptr()),                               \
            static_cast<cutlass::bfloat16_t**>(b_scales_ptrs.data_ptr()),                 \
            static_cast<cutlass::float_e4m3_t*>(a_tensors.data_ptr()),                    \
            static_cast<cutlass::int8_t*>(b_tensors.data_ptr()),                          \
            static_cast<C_TYPE*>(out_tensors.data_ptr()),                                 \
            static_cast<float*>(a_scales.data_ptr()),                                     \
            static_cast<cutlass::bfloat16_t*>(b_scales.data_ptr()),                       \
            out_tensors.size(1),                                                          \
            a_tensors.size(1),                                                            \
            per_act_token,                                                                \
            per_out_ch);                                                                  \
  }
```
**EN:** This section implements `__CALL_W4A8_GET_STARTS_KERNEL`, `data_ptr`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`__CALL_W4A8_GET_STARTS_KERNEL`、`data_ptr`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 92-113: Kernel implementation
```cpp
#define __CALL_W4A8_GET_STARTS_KERNEL_3D(TENSOR_C_TYPE, C_TYPE)                              \
  else if (out_tensors.dtype() == TENSOR_C_TYPE) {                                           \
    int4_fp8_get_group_gemm_starts_3d<cutlass::float_e4m3_t, cutlass::int8_t, C_TYPE, float> \
        <<<1, num_experts, 0, stream>>>(                                                     \
            static_cast<cutlass::float_e4m3_t**>(a_ptrs.data_ptr()),                         \
            static_cast<cutlass::int8_t**>(b_ptrs.data_ptr()),                               \
            static_cast<C_TYPE**>(out_ptrs.data_ptr()),                                      \
            static_cast<float**>(a_scales_ptrs.data_ptr()),                                  \
            static_cast<cutlass::bfloat16_t**>(b_scales_ptrs.data_ptr()),                    \
            static_cast<cutlass::float_e4m3_t*>(a_tensors.data_ptr()),                       \
            static_cast<cutlass::int8_t*>(b_tensors.data_ptr()),                             \
            static_cast<C_TYPE*>(out_tensors.data_ptr()),                                    \
            static_cast<float*>(a_scales.data_ptr()),                                        \
            static_cast<cutlass::bfloat16_t*>(b_scales.data_ptr()),                          \
            out_tensors.size(2),                                                             \
            a_tensors.size(1),                                                               \
            a_tensors.size(2),                                                               \
            per_act_token,                                                                   \
            per_out_ch,                                                                      \
            num_experts);                                                                    \
  }
```
**EN:** This section implements `__CALL_W4A8_GET_STARTS_KERNEL_3D`, `data_ptr`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`__CALL_W4A8_GET_STARTS_KERNEL_3D`、`data_ptr`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 114-136: Namespace and shared declarations
```cpp
namespace {

void run_int4_fp8_get_group_gemm_starts(
    torch::Tensor const& expert_offsets,
    torch::Tensor& a_ptrs,
    torch::Tensor& b_ptrs,
    torch::Tensor& out_ptrs,
    torch::Tensor& a_scales_ptrs,
    torch::Tensor& b_scales_ptrs,
    torch::Tensor const& a_tensors,
    torch::Tensor const& b_tensors,
    torch::Tensor& out_tensors,
    torch::Tensor const& a_scales,
    torch::Tensor const& b_scales) {
  TORCH_CHECK(a_tensors.dtype() == torch::kFloat8_e4m3fn);
  TORCH_CHECK(b_tensors.dtype() == torch::kInt8);
  TORCH_CHECK(a_scales.dtype() == torch::kFloat32);
  TORCH_CHECK(b_scales.dtype() == torch::kBFloat16);

  int num_experts = static_cast<int>(expert_offsets.size(0));
  bool per_act_token = a_scales.numel() != 1;
  bool per_out_ch = b_scales.numel() != num_experts;
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 137-157: Runtime integration and dispatch
```cpp
  auto stream = at::cuda::getCurrentCUDAStream(expert_offsets.device().index());

  if (a_tensors.dim() == 3) {
    if (false) {
    }
    __CALL_W4A8_GET_STARTS_KERNEL_3D(torch::kBFloat16, cutlass::bfloat16_t)
    __CALL_W4A8_GET_STARTS_KERNEL_3D(torch::kFloat16, half)
    else {
      TORCH_CHECK(false, "Invalid output type (must be float16 or bfloat16)");
    }
  } else {
    if (false) {
    }
    __CALL_W4A8_GET_STARTS_KERNEL(torch::kBFloat16, cutlass::bfloat16_t)
    __CALL_W4A8_GET_STARTS_KERNEL(torch::kFloat16, half)
    else {
      TORCH_CHECK(false, "Invalid output type (must be float16 or bfloat16)");
    }
  }
}
```
**EN:** This section uses `getCurrentCUDAStream`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`getCurrentCUDAStream`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 158-158: Local implementation details
```cpp
}  // namespace
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **GPU kernel programming / GPU 内核编程**: Defines executable device-side work launched from the host. / 定义由主机启动、在设备侧执行的工作。
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cutlass/bfloat16.h`, `cutlass/float8.h`
- **External headers / 外部头文件**: `c10/cuda/CUDAStream.h`, `cuda.h`, `torch/all.h`
- **Path context / 路径上下文**: moe / cutlass_moe / w4a8 / w4a8_get_group_starts.cuh
