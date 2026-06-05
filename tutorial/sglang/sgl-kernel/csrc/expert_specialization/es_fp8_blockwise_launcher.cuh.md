# es_fp8_blockwise_launcher.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/expert_specialization/es_fp8_blockwise_launcher.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Headers and compile-time setup
```cpp
#pragma once
#include <ATen/cuda/CUDAContext.h>
#include <c10/cuda/CUDAGuard.h>
#include <torch/all.h>

#include <cassert>
#include <iostream>
#include <string>

#include "cute/tensor.hpp"
#include "cutlass/cutlass.h"
#include "es_fp8_blockwise_functor.cuh"

namespace expert_specialization {

using namespace cute;
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 18-45: Runtime integration and dispatch
```cpp
template <typename T>
void es_sm90_fp8_blockwise_scaled_group_mm_pre_compute(
    // Output
    torch::Tensor& out_ptrs,
    torch::Tensor& a_ptrs,
    torch::Tensor& b_ptrs,
    torch::Tensor& a_scales_ptrs,
    torch::Tensor& b_scales_ptrs,
    torch::Tensor& layout_sfa,
    torch::Tensor& layout_sfb,
    torch::Tensor& lm_problem_sizes,
    torch::Tensor& mm_problem_sizes,
    torch::Tensor& hm_problem_sizes,
    // Input
    torch::Tensor& out_tensors,
    torch::Tensor const& a_tensors,
    torch::Tensor const& b_tensors,
    torch::Tensor const& a_scales,
    torch::Tensor const& b_scales,
    torch::Tensor const& problem_sizes,
    torch::Tensor const& expert_offsets,
    bool is_h20_device,
    cudaStream_t stream) {
  TORCH_CHECK(a_tensors.dtype() == torch::kFloat8_e4m3fn);
  TORCH_CHECK(b_tensors.dtype() == torch::kFloat8_e4m3fn);
  TORCH_CHECK(a_scales.dtype() == torch::kFloat32);
  TORCH_CHECK(b_scales.dtype() == torch::kFloat32);
```
**EN:** This section uses `es_sm90_fp8_blockwise_scaled_group_mm_pre_compute`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`es_sm90_fp8_blockwise_scaled_group_mm_pre_compute`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 46-67: Types and data layout
```cpp
  // Creat Scale Factor Layout Functor
  using LayoutSFA = typename PerfConfigMiddleMH20::LayoutSFA;
  using LayoutSFB = typename PerfConfigMiddleMH20::LayoutSFB;
  struct Fp8BlockwiseGroupedGemmSFLayoutFunctor<PerfConfigMiddleMH20> sf_layout(
      reinterpret_cast<LayoutSFA*>(layout_sfa.data_ptr()), reinterpret_cast<LayoutSFB*>(layout_sfb.data_ptr()));

  int num_experts = (int)expert_offsets.size(0);
  TORCH_CHECK(num_experts <= 1024, "Expert more than 1024");  // Max threads per block is 1024

  struct Fp8BlockwiseGroupedGemmOffsetFunctor<cutlass::float_e4m3_t, float, T> of(
      static_cast<int*>(expert_offsets.data_ptr()),
      static_cast<cutlass::float_e4m3_t*>(a_tensors.data_ptr()),
      static_cast<cutlass::float_e4m3_t*>(b_tensors.data_ptr()),
      static_cast<T*>(out_tensors.data_ptr()),
      static_cast<float*>(a_scales.data_ptr()),
      static_cast<float*>(b_scales.data_ptr()),
      static_cast<cutlass::float_e4m3_t**>(a_ptrs.data_ptr()),
      static_cast<cutlass::float_e4m3_t**>(b_ptrs.data_ptr()),
      static_cast<float**>(a_scales_ptrs.data_ptr()),
      static_cast<float**>(b_scales_ptrs.data_ptr()),
      static_cast<T**>(out_ptrs.data_ptr()));
  if (!is_h20_device) {
```
**EN:** This section defines `Fp8BlockwiseGroupedGemmSFLayoutFunctor`, `Fp8BlockwiseGroupedGemmOffsetFunctor`, `sf_layout`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Fp8BlockwiseGroupedGemmSFLayoutFunctor`、`Fp8BlockwiseGroupedGemmOffsetFunctor`、`sf_layout`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 68-87: Types and data layout
```cpp
    struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigLowMHx00> lm_psf(
        static_cast<int*>(lm_problem_sizes.data_ptr()));
    struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigMiddleMHx00> mm_psf(
        static_cast<int*>(mm_problem_sizes.data_ptr()));
    struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigHighMHx00> hm_psf(
        static_cast<int*>(hm_problem_sizes.data_ptr()));
    groupedGemmPreComputeKernel<<<1, num_experts, 0, stream>>>(
        static_cast<int*>(problem_sizes.data_ptr()), of, sf_layout, lm_psf, mm_psf, hm_psf);
  } else {
    struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigLowMH20> lm_psf(
        static_cast<int*>(lm_problem_sizes.data_ptr()));
    struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigMiddleMH20> mm_psf(
        static_cast<int*>(mm_problem_sizes.data_ptr()));
    struct Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor<PerfConfigHighMH20> hm_psf(
        static_cast<int*>(hm_problem_sizes.data_ptr()));
    groupedGemmPreComputeKernel<<<1, num_experts, 0, stream>>>(
        static_cast<int*>(problem_sizes.data_ptr()), of, sf_layout, lm_psf, mm_psf, hm_psf);
  }
}
```
**EN:** This section defines `Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`, `lm_psf`, `mm_psf`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`Fp8BlockwiseGroupedGemmProblemSizeFilterFunctor`、`lm_psf`、`mm_psf`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 88-109: Runtime integration and dispatch
```cpp
template <typename GemmTraits>
void launch_sm90_fp8_blockwise_scaled_group_mm(
    torch::Tensor& out_ptrs,
    const torch::Tensor& a_ptrs,
    const torch::Tensor& b_ptrs,
    const torch::Tensor& a_scales_ptrs,
    const torch::Tensor& b_scales_ptrs,
    const torch::Tensor& stride_a,
    const torch::Tensor& stride_b,
    const torch::Tensor& stride_d,
    const torch::Tensor& layout_sfa,
    const torch::Tensor& layout_sfb,
    const torch::Tensor& problem_sizes,
    const torch::Tensor& workspace,
    cudaStream_t stream,
    int sm_count) {
  using ElementA = typename GemmTraits::ElementA;
  using StrideA = typename GemmTraits::StrideA;
  using ElementB = typename GemmTraits::ElementB;
  using StrideB = typename GemmTraits::StrideB;
  using ElementAccumulator = typename GemmTraits::ElementAccumulator;
  using LayoutSFA = typename GemmTraits::LayoutSFA;
```
**EN:** This section uses `launch_sm90_fp8_blockwise_scaled_group_mm`, `ElementA`, `StrideA` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`launch_sm90_fp8_blockwise_scaled_group_mm`、`ElementA`、`StrideA`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 110-133: Runtime integration and dispatch
```cpp
  using LayoutSFB = typename GemmTraits::LayoutSFB;
  using ElementD = typename GemmTraits::ElementD;
  using StrideD = typename GemmTraits::StrideD;
  using UnderlyingProblemShape = typename GemmTraits::ProblemShape::UnderlyingProblemShape;
  using Gemm = typename GemmTraits::Gemm;
  using GemmKernel = typename GemmTraits::GemmKernel;

  int num_experts = (int)problem_sizes.size(0);
  Gemm gemm_op;

  typename GemmKernel::MainloopArguments mainloop_args{
      static_cast<const ElementA**>(a_ptrs.data_ptr()),
      static_cast<StrideA*>(stride_a.data_ptr()),
      static_cast<const ElementB**>(b_ptrs.data_ptr()),
      static_cast<StrideB*>(stride_b.data_ptr()),
      static_cast<const ElementAccumulator**>(a_scales_ptrs.data_ptr()),
      reinterpret_cast<LayoutSFA*>(layout_sfa.data_ptr()),
      static_cast<const ElementAccumulator**>(b_scales_ptrs.data_ptr()),
      reinterpret_cast<LayoutSFB*>(layout_sfb.data_ptr())};

  cutlass::KernelHardwareInfo hw_info;
  hw_info.device_id = c10::cuda::current_device();
  hw_info.sm_count = sm_count;
```
**EN:** This section uses `current_device`, `LayoutSFB`, `ElementD` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`current_device`、`LayoutSFB`、`ElementD`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 134-154: Runtime integration and dispatch
```cpp
  typename GemmKernel::EpilogueArguments epilogue_args{
      {}, nullptr, nullptr, static_cast<ElementD**>(out_ptrs.data_ptr()), static_cast<StrideD*>(stride_d.data_ptr())};

  UnderlyingProblemShape* problem_sizes_as_shapes = static_cast<UnderlyingProblemShape*>(problem_sizes.data_ptr());
  typename GemmKernel::Arguments args{
      cutlass::gemm::GemmUniversalMode::kGrouped,
      {num_experts, problem_sizes_as_shapes, nullptr},
      mainloop_args,
      epilogue_args,
      hw_info};

  auto can_implement_status = gemm_op.can_implement(args);
  TORCH_CHECK(can_implement_status == cutlass::Status::kSuccess, "Failed to implement GEMM");

  auto status = gemm_op.initialize(args, workspace.data_ptr(), stream);
  TORCH_CHECK(status == cutlass::Status::kSuccess, "Failed to initialize GEMM");

  status = gemm_op.run(stream, nullptr);
  TORCH_CHECK(status == cutlass::Status::kSuccess, "Failed to run GEMM");
}
```
**EN:** This section uses `data_ptr`, `can_implement`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`data_ptr`、`can_implement`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 155-176: Runtime integration and dispatch
```cpp
template <typename OutType>
void es_sm90_fp8_blockwise_scaled_group_mm_distpatch_out_dtype(
    torch::Tensor& out_ptrs,
    const torch::Tensor& a_ptrs,
    const torch::Tensor& b_ptrs,
    const torch::Tensor& a_scales_ptrs,
    const torch::Tensor& b_scales_ptrs,
    const torch::Tensor& stride_a,
    const torch::Tensor& stride_b,
    const torch::Tensor& stride_d,
    const torch::Tensor& layout_sfa,
    const torch::Tensor& layout_sfb,
    const torch::Tensor& lm_problem_sizes,
    const torch::Tensor& mm_problem_sizes,
    const torch::Tensor& hm_problem_sizes,
    const torch::Tensor& workspace,
    const torch::Tensor& backup_workspace_0,
    const torch::Tensor& backup_workspace_1,
    bool is_h20_device,
    cudaStream_t stream,
    cudaStream_t backup_stream_0,
    cudaStream_t backup_stream_1) {
```
**EN:** This section uses `es_sm90_fp8_blockwise_scaled_group_mm_distpatch_out_dtype` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`es_sm90_fp8_blockwise_scaled_group_mm_distpatch_out_dtype`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 177-198: Templates, aliases, and constants
```cpp
  using LowMGemmH20Traits =
      ExpertSpecializationSm90FP8BlockwiseGroupedGemmTraits<OutType, cutlass::layout::ColumnMajor, PerfConfigLowMH20>;
  using LowMGemmHx00Traits =
      ExpertSpecializationSm90FP8BlockwiseGroupedGemmTraits<OutType, cutlass::layout::ColumnMajor, PerfConfigLowMHx00>;
  using MiddleMGemmH20Traits =
      ExpertSpecializationSm90FP8BlockwiseGroupedGemmTraits<OutType, cutlass::layout::RowMajor, PerfConfigMiddleMH20>;
  using MiddleMGemmHx00Traits = ExpertSpecializationSm90FP8BlockwiseGroupedGemmTraits<
      OutType,
      cutlass::layout::ColumnMajor,
      PerfConfigMiddleMHx00>;
  using HighMGemmH20Traits =
      ExpertSpecializationSm90FP8BlockwiseGroupedGemmTraits<OutType, cutlass::layout::RowMajor, PerfConfigHighMH20>;
  using HighMGemmHx00Traits =
      ExpertSpecializationSm90FP8BlockwiseGroupedGemmTraits<OutType, cutlass::layout::RowMajor, PerfConfigHighMHx00>;

  if (!is_h20_device) {
    launch_sm90_fp8_blockwise_scaled_group_mm<HighMGemmHx00Traits>(
        out_ptrs,
        a_ptrs,
        b_ptrs,
        a_scales_ptrs,
        b_scales_ptrs,
```
**EN:** This section defines `LowMGemmH20Traits`, `LowMGemmHx00Traits`, `MiddleMGemmH20Traits`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`LowMGemmH20Traits`、`LowMGemmHx00Traits`、`MiddleMGemmH20Traits`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 199-224: Local implementation details
```cpp
        stride_a,
        stride_b,
        stride_d,
        layout_sfa,
        layout_sfb,
        hm_problem_sizes,
        workspace,
        stream,
        132);
  } else {
    launch_sm90_fp8_blockwise_scaled_group_mm<HighMGemmH20Traits>(
        out_ptrs,
        a_ptrs,
        b_ptrs,
        a_scales_ptrs,
        b_scales_ptrs,
        stride_a,
        stride_b,
        stride_d,
        layout_sfa,
        layout_sfb,
        hm_problem_sizes,
        workspace,
        stream,
        78);
  }
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 225-246: Control flow and branching
```cpp

  if (!is_h20_device) {
    launch_sm90_fp8_blockwise_scaled_group_mm<LowMGemmHx00Traits>(
        out_ptrs,
        b_ptrs,
        a_ptrs,
        b_scales_ptrs,
        a_scales_ptrs,
        stride_b,
        stride_a,
        stride_d,
        layout_sfb,
        layout_sfa,
        lm_problem_sizes,
        backup_workspace_1,
        backup_stream_1,
        132);
  } else {
    launch_sm90_fp8_blockwise_scaled_group_mm<LowMGemmH20Traits>(
        out_ptrs,
        b_ptrs,
        a_ptrs,
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 247-268: Control flow and branching
```cpp
        b_scales_ptrs,
        a_scales_ptrs,
        stride_b,
        stride_a,
        stride_d,
        layout_sfb,
        layout_sfa,
        lm_problem_sizes,
        backup_workspace_1,
        backup_stream_1,
        78);
  }

  if (!is_h20_device) {
    launch_sm90_fp8_blockwise_scaled_group_mm<MiddleMGemmHx00Traits>(
        out_ptrs,
        b_ptrs,
        a_ptrs,
        b_scales_ptrs,
        a_scales_ptrs,
        stride_b,
        stride_a,
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 269-292: Local implementation details
```cpp
        stride_d,
        layout_sfb,
        layout_sfa,
        mm_problem_sizes,
        backup_workspace_0,
        backup_stream_0,
        132);
  } else {
    launch_sm90_fp8_blockwise_scaled_group_mm<MiddleMGemmH20Traits>(
        out_ptrs,
        a_ptrs,
        b_ptrs,
        a_scales_ptrs,
        b_scales_ptrs,
        stride_a,
        stride_b,
        stride_d,
        layout_sfa,
        layout_sfb,
        mm_problem_sizes,
        backup_workspace_0,
        backup_stream_0,
        78);
  }
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 293-294: Local implementation details
```cpp
}
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

### Lines 295-295: Local implementation details
```cpp
}  // namespace expert_specialization
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cute/tensor.hpp`, `cutlass/cutlass.h`, `es_fp8_blockwise_functor.cuh`
- **External headers / 外部头文件**: `ATen/cuda/CUDAContext.h`, `c10/cuda/CUDAGuard.h`, `torch/all.h`, `cassert`, `iostream`, `string`
- **Path context / 路径上下文**: expert_specialization / es_fp8_blockwise_launcher.cuh
