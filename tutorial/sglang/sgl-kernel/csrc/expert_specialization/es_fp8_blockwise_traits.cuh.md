# es_fp8_blockwise_traits.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/expert_specialization/es_fp8_blockwise_traits.cuh`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Contains low-level kernel code and support utilities for the sgl-kernel backend. This header primarily defines interfaces, templates, constants, and inline helpers shared by compilation units. / 包含 sgl-kernel 后端的底层内核代码和配套工具。 该头文件主要定义编译单元共享的接口、模板、常量和内联辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Headers and compile-time setup
```cpp
#pragma once

// Misc
#include "cute/tensor.hpp"
#include "cutlass/arch/arch.h"
#include "cutlass/arch/mma.h"
#include "cutlass/cutlass.h"
#include "cutlass/detail/blockwise_scale_layout.hpp"
#include "cutlass/epilogue/dispatch_policy.hpp"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/gemm/group_array_problem_shape.hpp"
#include "cutlass/layout/layout.h"
#include "cutlass/numeric_conversion.h"
#include "cutlass/numeric_size.h"

// Collective Builder
#include "cutlass/epilogue/collective/collective_builder.hpp"
#include "cutlass/epilogue/fusion/sm90_callbacks_tma_warpspecialized.hpp"
#include "cutlass/epilogue/thread/activation.h"
#include "cutlass/gemm/collective/collective_builder.hpp"
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 22-42: Namespace and shared declarations
```cpp
// Integration
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/kernel/gemm_universal.hpp"

namespace expert_specialization {

using namespace cute;

struct PerfConfigLowMH20 {
  // Swap A/B
  using ElementA = cutlass::float_e4m3_t;
  using MmaTileShape = Shape<_256, _32, _128>;
  using ClusterShape = Shape<_2, _1, _1>;
  using KernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise;
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecializedCooperative;
  using ScaleConfig =
      cutlass::detail::Sm90BlockwiseScaleConfig<128, 1, 128, cute::GMMA::Major::K, cute::GMMA::Major::K>;
  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());
};
```
**EN:** This section organizes shared declarations under namespace scope, keeping related symbols together and reducing naming conflicts.
**CN:** 本段把共享声明组织在命名空间作用域下，使相关符号保持聚合并减少命名冲突。

### Lines 43-66: Types and data layout
```cpp
struct PerfConfigLowMHx00 {
  // Swap A/B
  using ElementA = cutlass::float_e4m3_t;
  using MmaTileShape = Shape<_256, _32, _128>;
  using ClusterShape = Shape<_2, _1, _1>;
  using KernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise;
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecializedCooperative;
  using ScaleConfig =
      cutlass::detail::Sm90BlockwiseScaleConfig<128, 1, 128, cute::GMMA::Major::K, cute::GMMA::Major::K>;
  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());
};

struct PerfConfigMiddleMH20 {
  using ElementA = cutlass::float_e4m3_t;
  using MmaTileShape = Shape<_64, _128, _128>;
  using ClusterShape = Shape<_1, _2, _1>;
  using KernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongFP8Blockwise;
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong;
  using ScaleConfig =
      cutlass::detail::Sm90BlockwiseScaleConfig<1, 128, 128, cute::GMMA::Major::K, cute::GMMA::Major::K>;
  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());
};
```
**EN:** This section defines `PerfConfigLowMHx00`, `PerfConfigMiddleMH20`, `decltype`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`PerfConfigLowMHx00`、`PerfConfigMiddleMH20`、`decltype`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 67-90: Types and data layout
```cpp

struct PerfConfigMiddleMHx00 {
  using ElementA = cutlass::float_e4m3_t;
  using MmaTileShape = Shape<_256, _64, _128>;
  using ClusterShape = Shape<_2, _1, _1>;
  using KernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise;
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecializedCooperative;
  using ScaleConfig =
      cutlass::detail::Sm90BlockwiseScaleConfig<128, 1, 128, cute::GMMA::Major::K, cute::GMMA::Major::K>;
  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());
};

struct PerfConfigHighMH20 {
  using ElementA = cutlass::float_e4m3_t;
  using MmaTileShape = Shape<_64, _128, _128>;
  using ClusterShape = Shape<_2, _1, _1>;
  using KernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedPingpongFP8Blockwise;
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecializedPingpong;
  using ScaleConfig =
      cutlass::detail::Sm90BlockwiseScaleConfig<1, 128, 128, cute::GMMA::Major::K, cute::GMMA::Major::K>;
  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());
};
```
**EN:** This section defines `PerfConfigMiddleMHx00`, `PerfConfigHighMH20`, `decltype`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`PerfConfigMiddleMHx00`、`PerfConfigHighMH20`、`decltype`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 91-117: Types and data layout
```cpp

struct PerfConfigHighMHx00 {
  using ElementA = cutlass::float_e4m3_t;
  using MmaTileShape = Shape<_128, _128, _128>;
  using ClusterShape = Shape<_1, _2, _1>;
  using KernelSchedule = cutlass::gemm::KernelPtrArrayTmaWarpSpecializedCooperativeFP8Blockwise;
  using EpilogueSchedule = cutlass::epilogue::PtrArrayTmaWarpSpecializedCooperative;
  using ScaleConfig =
      cutlass::detail::Sm90BlockwiseScaleConfig<1, 128, 128, cute::GMMA::Major::K, cute::GMMA::Major::K>;
  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());
};

template <typename OutType, typename LayoutD, typename PerfConfig>
struct ExpertSpecializationSm90FP8BlockwiseGroupedGemmTraits {
  using ElementA = cutlass::float_e4m3_t;
  using ElementB = cutlass::float_e4m3_t;
  using ElementC = void;
  using ElementD = OutType;
  using ElementAccumulator = float;
  using LayoutA = cutlass::layout::RowMajor;
  using LayoutB = cutlass::layout::ColumnMajor;
  using LayoutC = LayoutD;
  using LayoutSFA = typename PerfConfig::LayoutSFA;
  using LayoutSFB = typename PerfConfig::LayoutSFB;
  using ProblemShape = cutlass::gemm::GroupProblemShape<Shape<int, int, int>>;
```
**EN:** This section defines `PerfConfigHighMHx00`, `ExpertSpecializationSm90FP8BlockwiseGroupedGemmTraits`, `decltype`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`PerfConfigHighMHx00`、`ExpertSpecializationSm90FP8BlockwiseGroupedGemmTraits`、`decltype`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 118-139: Templates, aliases, and constants
```cpp
  static constexpr int AlignmentA = 128 / cutlass::sizeof_bits<ElementA>::value;
  static constexpr int AlignmentB = 128 / cutlass::sizeof_bits<ElementB>::value;
  static constexpr int AlignmentC = 128 / cutlass::sizeof_bits<ElementD>::value;
  static constexpr int AlignmentD = 128 / cutlass::sizeof_bits<ElementD>::value;

  using ArchTag = cutlass::arch::Sm90;
  using OperatorClass = cutlass::arch::OpClassTensorOp;
  static constexpr auto RoundStyle = cutlass::FloatRoundStyle::round_to_nearest;
  using CustomEVTIdentity =  // acc
      cutlass::epilogue::fusion::Sm90EVT<
          cutlass::epilogue::fusion::
              Sm90Compute<cutlass::epilogue::thread::Identity, ElementD, ElementAccumulator, RoundStyle>,
          cutlass::epilogue::fusion::Sm90AccFetch>;

  using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
      ArchTag,
      OperatorClass,
      typename PerfConfig::MmaTileShape,
      typename PerfConfig::ClusterShape,
      cutlass::epilogue::collective::EpilogueTileAuto,
      ElementAccumulator,
      ElementAccumulator,
```
**EN:** This section defines `AlignmentA`, `AlignmentB`, `AlignmentC`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`AlignmentA`、`AlignmentB`、`AlignmentC`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 140-164: Templates, aliases, and constants
```cpp
      ElementC,  // Use void to avoid load Matrix C
      LayoutC*,
      AlignmentC,
      ElementD,
      LayoutD*,
      AlignmentD,
      typename PerfConfig::EpilogueSchedule,
      CustomEVTIdentity>::CollectiveOp;

  using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
      ArchTag,
      OperatorClass,
      ElementA,
      cute::tuple<LayoutA*, typename PerfConfig::LayoutSFA*>,
      AlignmentA,
      ElementB,
      cute::tuple<LayoutB*, typename PerfConfig::LayoutSFB*>,
      AlignmentB,
      ElementAccumulator,
      typename PerfConfig::MmaTileShape,
      typename PerfConfig::ClusterShape,
      cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
          sizeof(typename CollectiveEpilogue::SharedStorage))>,
      typename PerfConfig::KernelSchedule>::CollectiveOp;
```
**EN:** This section defines `CollectiveMainloop`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`CollectiveMainloop`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 165-173: Templates, aliases, and constants
```cpp
  using GemmKernel = cutlass::gemm::kernel::GemmUniversal<ProblemShape, CollectiveMainloop, CollectiveEpilogue, void>;
  using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
  using UnderlyingProblemShape = ProblemShape::UnderlyingProblemShape;
  using StrideA = typename Gemm::GemmKernel::InternalStrideA;
  using StrideB = typename Gemm::GemmKernel::InternalStrideB;
  using StrideC = typename Gemm::GemmKernel::InternalStrideC;
  using StrideD = typename Gemm::GemmKernel::InternalStrideD;
};
```
**EN:** This section defines `GemmKernel`, `Gemm`, `UnderlyingProblemShape`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`GemmKernel`、`Gemm`、`UnderlyingProblemShape`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 174-174: Local implementation details
```cpp
}  // namespace expert_specialization
```
**EN:** This section fills in the local implementation details around the surrounding logic, completing the behavior required by the file.
**CN:** 本段补充了相关逻辑周边的局部实现细节，以完成该文件所需的具体行为。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cute/tensor.hpp`, `cutlass/arch/arch.h`, `cutlass/arch/mma.h`, `cutlass/cutlass.h`, `cutlass/detail/blockwise_scale_layout.hpp`, `cutlass/epilogue/dispatch_policy.hpp`, `cutlass/gemm/dispatch_policy.hpp`, `cutlass/gemm/group_array_problem_shape.hpp`, `cutlass/layout/layout.h`, `cutlass/numeric_conversion.h`
- **External headers / 外部头文件**: Standard library or none / 标准库或无
- **Path context / 路径上下文**: expert_specialization / es_fp8_blockwise_traits.cuh
