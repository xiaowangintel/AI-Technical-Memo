# fp8_blockwise_gemm_sm90_dispatch.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/cutlass_extensions/gemm/fp8_blockwise_gemm_sm90_dispatch.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Headers and compile-time setup
```cpp
// Adapted from
// https://github.com/vllm-project/vllm/blob/main/csrc/quantization/cutlass_w8a8/c3x/scaled_mm_blockwise_sm90_fp8_dispatch.cuh
#pragma once

#include "cute/tensor.hpp"
#include "cutlass/cutlass.h"
#include "cutlass/epilogue/collective/collective_builder.hpp"
#include "cutlass/epilogue/dispatch_policy.hpp"
#include "cutlass/gemm/collective/collective_builder.hpp"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/kernel/gemm_universal.hpp"
#include "cutlass/gemm/kernel/tile_scheduler_params.h"
#include "cutlass/numeric_types.h"
#include "cutlass/tensor_ref.h"
#include "cutlass_extensions/common.hpp"
#include "cutlass_extensions/gemm/cutlass_gemm_caller.cuh"
#include "cutlass_extensions/gemm/dispatch_policy.hpp"

using namespace cute;
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 22-44: Types and data layout
```cpp
template <
    typename SchedulerType,
    typename OutType,
    int GroupSizeM_,
    int GroupSizeN_,
    int GroupSizeK_,
    int TileSizeM_ = 128,
    class ClusterShape = Shape<_1, _2, _1>>
struct cutlass_3x_gemm_fp8_blockwise {
  using GroupSizeM = Int<GroupSizeM_>;
  using GroupSizeN = Int<GroupSizeN_>;
  using GroupSizeK = Int<GroupSizeK_>;
  using TileSizeM = Int<TileSizeM_>;

  static_assert(TileSizeM_ % GroupSizeM_ == 0, "TileSizeM must be a multiple of GroupSizeM");

  using ElementAB = cutlass::float_e4m3_t;

  // A matrix configuration
  using ElementA = ElementAB;
  using LayoutA = cutlass::layout::RowMajor;
  static constexpr int AlignmentA = 128 / cutlass::sizeof_bits<ElementA>::value;
```
**EN:** This section defines `ClusterShape`, `cutlass_3x_gemm_fp8_blockwise`, `static_assert`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`ClusterShape`、`cutlass_3x_gemm_fp8_blockwise`、`static_assert`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 45-68: Templates, aliases, and constants
```cpp
  // B matrix configuration
  using ElementB = ElementAB;
  using LayoutB = cutlass::layout::ColumnMajor;
  static constexpr int AlignmentB = 128 / cutlass::sizeof_bits<ElementB>::value;

  // C/D matrix configuration
  using ElementC = void;
  using LayoutC = cutlass::layout::RowMajor;
  static constexpr int AlignmentC = 128 / cutlass::sizeof_bits<OutType>::value;

  using ElementD = OutType;
  using LayoutD = cutlass::layout::RowMajor;
  static constexpr int AlignmentD = AlignmentC;

  using ScaleTileShape = Shape<_1, _128, _128>;
  using ScaleConfig = decltype(cutlass::detail::sm90_trivial_blockwise_scale_config(ScaleTileShape{}));
  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());

  // Multiply-accumulate blocking/pipelining details
  using ElementAccumulator = float;                            // Element type for internal accumulation
  using ElementCompute = float;                                // Element type for compute
  using TileShape = Shape<TileSizeM, GroupSizeN, GroupSizeK>;  // Threadblock-level tile size
```
**EN:** This section defines `decltype`, `ElementB`, `LayoutB`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`decltype`、`ElementB`、`LayoutB`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 69-92: Device helpers and synchronization
```cpp
  using ArchTag = cutlass::arch::Sm90;
  using OperatorClass = cutlass::arch::OpClassTensorOp;
  using EpilogueSchedule = cutlass::epilogue::TmaWarpSpecializedCooperative;
  using EpilogueTileType = cutlass::epilogue::collective::EpilogueTileAuto;
  using StoreEpilogueCompute = typename cutlass::epilogue::fusion::Sm90EVT<cutlass::epilogue::fusion::Sm90AccFetch>;

  using KernelSchedule = cutlass::gemm::KernelTmaWarpSpecializedCooperativeFP8Blockwise;
  using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
      ArchTag,
      OperatorClass,
      TileShape,
      ClusterShape,
      EpilogueTileType,
      ElementAccumulator,
      ElementCompute,
      ElementC,
      LayoutC,
      AlignmentC,
      ElementD,
      LayoutD,
      AlignmentD,
      EpilogueSchedule,
      StoreEpilogueCompute>::CollectiveOp;
```
**EN:** This section implements `ArchTag`, `OperatorClass`, `EpilogueSchedule`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ArchTag`、`OperatorClass`、`EpilogueSchedule`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 93-114: Templates, aliases, and constants
```cpp
  using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
      ArchTag,
      OperatorClass,
      ElementA,
      cute::tuple<LayoutA, LayoutSFA>,
      AlignmentA,
      ElementB,
      cute::tuple<LayoutB, LayoutSFB>,
      AlignmentB,
      ElementAccumulator,
      TileShape,
      ClusterShape,
      cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
          sizeof(typename CollectiveEpilogue::SharedStorage))>,
      KernelSchedule>::CollectiveOp;

  using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
      Shape<int, int, int, int>,  // Indicates ProblemShape
      CollectiveMainloop,
      CollectiveEpilogue,
      SchedulerType>;
};
```
**EN:** This section defines `CollectiveMainloop`, `GemmKernel`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`CollectiveMainloop`、`GemmKernel`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 115-138: Runtime integration and dispatch
```cpp

template <typename Gemm>
void cutlass_gemm_caller_blockwise(
    torch::Tensor& out,
    torch::Tensor const& a,
    torch::Tensor const& b,
    torch::Tensor const& a_scales,
    torch::Tensor const& b_scales) {
  using GemmKernel = typename Gemm::GemmKernel;
  using ElementAB = typename Gemm::ElementAB;
  using ElementA = ElementAB;
  using ElementB = ElementAB;
  using ElementD = typename Gemm::ElementD;
  using ElementBlockScale = float;

  using ScaleTileShape = Shape<_1, _128, _128>;
  using ScaleConfig = decltype(cutlass::detail::sm90_trivial_blockwise_scale_config(ScaleTileShape{}));
  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());

  int m = a.size(0);
  int k = a.size(1);
  int n = b.size(1);
```
**EN:** This section uses `cutlass_gemm_caller_blockwise`, `decltype`, `GemmKernel` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`cutlass_gemm_caller_blockwise`、`decltype`、`GemmKernel`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 139-160: Templates, aliases, and constants
```cpp
  auto a_ptr = static_cast<ElementA*>(a.data_ptr());
  auto b_ptr = static_cast<ElementB*>(b.data_ptr());

  auto a_s_ptr = static_cast<ElementBlockScale*>(a_scales.data_ptr());
  auto b_s_ptr = static_cast<ElementBlockScale*>(b_scales.data_ptr());

  using StrideA = typename GemmKernel::StrideA;
  using StrideB = typename GemmKernel::StrideB;
  using StrideD = typename GemmKernel::StrideD;
  using StrideC = typename GemmKernel::StrideC;

  StrideA a_stride = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(m, k, 1));
  StrideB b_stride = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(n, k, 1));
  StrideC c_stride = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(m, n, 1));
  LayoutSFA layout_sfa = ScaleConfig::tile_atom_to_shape_SFA(make_shape(m, n, k, 1));
  LayoutSFB layout_sfb = ScaleConfig::tile_atom_to_shape_SFB(make_shape(m, n, k, 1));

  typename GemmKernel::MainloopArguments mainloop_args{
      a_ptr, a_stride, b_ptr, b_stride, a_s_ptr, layout_sfa, b_s_ptr, layout_sfb};
  auto c_ptr = static_cast<ElementD*>(out.data_ptr());
  typename GemmKernel::EpilogueArguments epilogue_args{{}, c_ptr, c_stride, c_ptr, c_stride};
```
**EN:** This section defines `data_ptr`, `make_shape`, `tile_atom_to_shape_SFA`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`data_ptr`、`make_shape`、`tile_atom_to_shape_SFA`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 161-178: Templates, aliases, and constants
```cpp
  typename GemmKernel::TileSchedulerArguments scheduler;

  static constexpr bool UsesStreamKScheduler =
      cute::is_same_v<typename GemmKernel::TileSchedulerTag, cutlass::gemm::StreamKScheduler>;

  if constexpr (UsesStreamKScheduler) {
    using DecompositionMode =
        typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90StreamKParams::DecompositionMode;
    using ReductionMode =
        typename cutlass::gemm::kernel::detail::PersistentTileSchedulerSm90StreamKParams::ReductionMode;

    scheduler.decomposition_mode = DecompositionMode::StreamK;
    scheduler.reduction_mode = ReductionMode::Nondeterministic;
  }

  cutlass_gemm_caller<GemmKernel>(a.device(), {m, n, k, 1}, mainloop_args, epilogue_args, scheduler);
}
```
**EN:** This section defines `UsesStreamKScheduler`, `DecompositionMode`, `ReductionMode`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`UsesStreamKScheduler`、`DecompositionMode`、`ReductionMode`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 179-197: Runtime integration and dispatch
```cpp
template <typename OutType>
void cutlass_gemm_blockwise_sm90_fp8_dispatch(
    torch::Tensor& out,
    torch::Tensor const& a,
    torch::Tensor const& b,
    torch::Tensor const& a_scales,
    torch::Tensor const& b_scales) {
  auto k = a.size(1);
  auto n = b.size(1);

  if (k > 3 * n) {
    cutlass_gemm_caller_blockwise<cutlass_3x_gemm_fp8_blockwise<cutlass::gemm::StreamKScheduler, OutType, 1, 128, 128>>(
        out, a, b, a_scales, b_scales);
  } else {
    cutlass_gemm_caller_blockwise<
        cutlass_3x_gemm_fp8_blockwise<cutlass::gemm::PersistentScheduler, OutType, 1, 128, 128>>(
        out, a, b, a_scales, b_scales);
  }
}
```
**EN:** This section uses `cutlass_gemm_blockwise_sm90_fp8_dispatch` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`cutlass_gemm_blockwise_sm90_fp8_dispatch`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cute/tensor.hpp`, `cutlass/cutlass.h`, `cutlass/epilogue/collective/collective_builder.hpp`, `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/gemm/collective/collective_builder.hpp`, `cutlass/gemm/device/gemm_universal_adapter.h`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/gemm/kernel/gemm_universal.hpp`, `cutlass/gemm/kernel/tile_scheduler_params.h`, `cutlass/numeric_types.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: cutlass_extensions / gemm / fp8_blockwise_gemm_sm90_dispatch.cuh
