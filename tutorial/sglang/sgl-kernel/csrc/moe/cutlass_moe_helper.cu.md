# cutlass_moe_helper.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/moe/cutlass_moe_helper.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements mixture-of-experts routing, combination, or expert computation utilities. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 实现 MoE 的路由、聚合或专家计算辅助逻辑。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: Kernel implementation
```cpp
#pragma once

#include <c10/cuda/CUDAStream.h>
#include <cuda.h>
#include <torch/all.h>

#include "cutlass/bfloat16.h"
#include "cutlass/float8.h"

template <
    typename ElementAB,
    typename ElementC,
    typename ElementAccumulator,
    typename LayoutSFA,
    typename LayoutSFB,
    typename ScaleConfig>
__global__ void get_group_gemm_starts(
    int32_t* expert_offsets,
    ElementAB** a_offsets,
    ElementAB** b_offsets,
    ElementC** out_offsets,
    ElementAccumulator** a_scales_offsets,
```
**EN:** This section implements the surrounding logic, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了相关逻辑等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 23-47: Device helpers and synchronization
```cpp
    ElementAccumulator** b_scales_offsets,
    ElementAB* a_base_as_int,
    ElementAB* b_base_as_int,
    ElementC* out_base_as_int,
    ElementAccumulator* a_scales_base_as_int,
    ElementAccumulator* b_scales_base_as_int,
    LayoutSFA* layout_sfa_base_as_int,
    LayoutSFB* layout_sfb_base_as_int,
    int* problem_sizes,
    int* problem_sizes_transpose,
    bool transpose = false) {
  int64_t expert_id = static_cast<int64_t>(threadIdx.x);

  if (expert_id >= gridDim.x * blockDim.x) {
    return;
  }

  int m = problem_sizes[expert_id * 3];
  int n = problem_sizes[expert_id * 3 + 1];
  int k = problem_sizes[expert_id * 3 + 2];
  if (transpose) {
    problem_sizes_transpose[expert_id * 3] = n;
    problem_sizes_transpose[expert_id * 3 + 1] = m;
    problem_sizes_transpose[expert_id * 3 + 2] = k;
  }
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 48-70: Control flow and branching
```cpp

  int64_t expert_offset = static_cast<int64_t>(expert_offsets[expert_id]);
  int64_t a_stride = 0;
  int64_t b_stride = 0;
  int64_t a_scale_stride = 0;
  int64_t b_scale_stride = 0;
  if (!transpose) {
    a_stride = expert_offset * k;
    b_stride = expert_id * k * n;
    a_scale_stride = expert_offset * k / 128;
    b_scale_stride = expert_id * k * n / 128 / 128;
  } else {
    a_stride = expert_id * k * n;
    b_stride = expert_offset * k;
    a_scale_stride = expert_id * k * n / 128 / 128;
    b_scale_stride = expert_offset * k / 128;
  }
  a_offsets[expert_id] = a_base_as_int + a_stride;
  b_offsets[expert_id] = b_base_as_int + b_stride;
  out_offsets[expert_id] = out_base_as_int + expert_offset * n;
  a_scales_offsets[expert_id] = a_scales_base_as_int + a_scale_stride;
  b_scales_offsets[expert_id] = b_scales_base_as_int + b_scale_stride;
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 71-92: Kernel implementation
```cpp
  LayoutSFA* layout_sfa_ptr = layout_sfa_base_as_int + expert_id;
  LayoutSFB* layout_sfb_ptr = layout_sfb_base_as_int + expert_id;

  if (!transpose) {
    *layout_sfa_ptr = ScaleConfig::tile_atom_to_shape_SFA(cute::make_shape(m, n, k, 1));
    *layout_sfb_ptr = ScaleConfig::tile_atom_to_shape_SFB(cute::make_shape(m, n, k, 1));
  } else {
    *layout_sfa_ptr = ScaleConfig::tile_atom_to_shape_SFA(cute::make_shape(n, m, k, 1));
    *layout_sfb_ptr = ScaleConfig::tile_atom_to_shape_SFB(cute::make_shape(n, m, k, 1));
  }
}

#define __CALL_GET_STARTS_KERNEL(TENSOR_C_TYPE, C_TYPE, LayoutSFA, LayoutSFB, ScaleConfig)         \
  else if (out_tensors.dtype() == TENSOR_C_TYPE) {                                                 \
    get_group_gemm_starts<cutlass::float_e4m3_t, C_TYPE, float, LayoutSFA, LayoutSFB, ScaleConfig> \
        <<<1, num_experts, 0, stream>>>(                                                           \
            static_cast<int32_t*>(expert_offsets.data_ptr()),                                      \
            static_cast<cutlass::float_e4m3_t**>(a_ptrs.data_ptr()),                               \
            static_cast<cutlass::float_e4m3_t**>(b_ptrs.data_ptr()),                               \
            static_cast<C_TYPE**>(out_ptrs.data_ptr()),                                            \
            static_cast<float**>(a_scales_ptrs.data_ptr()),                                        \
            static_cast<float**>(b_scales_ptrs.data_ptr()),                                        \
```
**EN:** This section implements `__CALL_GET_STARTS_KERNEL`, `tile_atom_to_shape_SFA`, `tile_atom_to_shape_SFB`, the GPU work units and launch-facing code that carry out the file's core computation.
**CN:** 本段实现了`__CALL_GET_STARTS_KERNEL`、`tile_atom_to_shape_SFA`、`tile_atom_to_shape_SFB`等 GPU 工作单元及其启动相关代码，承担该文件的核心计算。

### Lines 93-114: Runtime integration and dispatch
```cpp
            static_cast<cutlass::float_e4m3_t*>(a_tensors.data_ptr()),                             \
            static_cast<cutlass::float_e4m3_t*>(b_tensors.data_ptr()),                             \
            static_cast<C_TYPE*>(out_tensors.data_ptr()),                                          \
            static_cast<float*>(a_scales.data_ptr()),                                              \
            static_cast<float*>(b_scales.data_ptr()),                                              \
            reinterpret_cast<LayoutSFA*>(layout_sfa.data_ptr()),                                   \
            reinterpret_cast<LayoutSFB*>(layout_sfb.data_ptr()),                                   \
            static_cast<int*>(problem_sizes.data_ptr()),                                           \
            static_cast<int*>(problem_sizes_transpose.data_ptr()),                                 \
            transpose);                                                                            \
  }

namespace {
template <typename LayoutSFA, typename LayoutSFB, typename ScaleConfig>
void run_get_group_gemm_starts(
    torch::Tensor const& expert_offsets,
    torch::Tensor& a_ptrs,
    torch::Tensor& b_ptrs,
    torch::Tensor& out_ptrs,
    torch::Tensor& a_scales_ptrs,
    torch::Tensor& b_scales_ptrs,
    torch::Tensor const& a_tensors,
```
**EN:** This section uses `data_ptr` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`data_ptr`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 115-135: Runtime integration and dispatch
```cpp
    torch::Tensor const& b_tensors,
    torch::Tensor& out_tensors,
    torch::Tensor const& a_scales,
    torch::Tensor const& b_scales,
    torch::Tensor const& layout_sfa,
    torch::Tensor const& layout_sfb,
    torch::Tensor const& problem_sizes,
    torch::Tensor& problem_sizes_transpose,
    bool transpose = false) {
  TORCH_CHECK(a_tensors.dtype() == torch::kFloat8_e4m3fn);
  TORCH_CHECK(b_tensors.dtype() == torch::kFloat8_e4m3fn);
  TORCH_CHECK(a_scales.dtype() == torch::kFloat32);
  TORCH_CHECK(b_scales.dtype() == torch::kFloat32);
  TORCH_CHECK(out_tensors.size(1) % 128 == 0 or out_tensors.size(0) % 128 == 0);
  TORCH_CHECK(a_tensors.size(1) % 128 == 0 or a_tensors.size(0) % 128 == 0);

  int num_experts = (int)expert_offsets.size(0);
  auto stream = at::cuda::getCurrentCUDAStream(a_tensors.device().index());

  if (false) {
  }
```
**EN:** This section uses `TORCH_CHECK`, `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 136-141: Runtime integration and dispatch
```cpp
  __CALL_GET_STARTS_KERNEL(torch::kBFloat16, cutlass::bfloat16_t, LayoutSFA, LayoutSFB, ScaleConfig)
  __CALL_GET_STARTS_KERNEL(torch::kFloat16, half, LayoutSFA, LayoutSFB, ScaleConfig)
  else {
    TORCH_CHECK(false, "Invalid output type (must be float16 or bfloat16)");
  }
}
```
**EN:** This section uses `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 142-142: Local implementation details
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
- **Path context / 路径上下文**: moe / cutlass_moe_helper.cu
