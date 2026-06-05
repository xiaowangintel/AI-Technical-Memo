# fp8_blockwise_gemm_kernel.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/fp8_blockwise_gemm_kernel.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Headers and compile-time setup
```cpp
#include <ATen/cuda/CUDAContext.h>
#include <cudaTypedefs.h>
#include <cutlass/arch/arch.h>
#include <cutlass/arch/memory.h>
#include <cutlass/arch/mma.h>
#include <cutlass/array.h>
#include <cutlass/cutlass.h>
#include <cutlass/epilogue/thread/activation.h>
#include <cutlass/epilogue/thread/linear_combination.h>
#include <cutlass/epilogue/threadblock/default_thread_map_tensor_op.h>
#include <cutlass/gemm/device/gemm.h>
#include <cutlass/gemm/device/gemm_universal_adapter.h>
#include <cutlass/gemm/gemm.h>
#include <cutlass/gemm/kernel/default_gemm_universal_with_visitor.h>
#include <cutlass/gemm/thread/mma.h>
#include <cutlass/layout/matrix.h>
#include <cutlass/matrix_coord.h>
#include <cutlass/numeric_types.h>
#include <cutlass/tensor_ref.h>
#include <cutlass/util/host_tensor.h>
#include <cutlass/util/tensor_view_io.h>
#include <torch/all.h>
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 24-47: Headers and compile-time setup
```cpp
#include <cute/tensor.hpp>
#include <cutlass/epilogue/collective/collective_builder.hpp>
#include <cutlass/epilogue/collective/default_epilogue.hpp>
#include <cutlass/epilogue/threadblock/fusion/visitors.hpp>
#include <cutlass/gemm/collective/collective_builder.hpp>
#include <cutlass/gemm/dispatch_policy.hpp>
#include <cutlass/gemm/kernel/gemm_universal.hpp>
#include <cutlass/util/packed_stride.hpp>

#include "cutlass_extensions/gemm/cutlass_gemm_caller.cuh"
#include "cutlass_extensions/gemm/fp8_blockwise_gemm_sm90_dispatch.cuh"
#include "utils.h"

using namespace cute;

template <
    typename OutType,
    typename MmaTileShape,
    typename PerSmTileShape,
    typename EpilogueTileShape,
    typename ScalesPerTile,
    int TileSizeM_ = 128,
    class ClusterShape = Shape<_1, _1, _1>>
void launch_sm100_fp8_blockwise_scaled_mm(
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 48-76: Runtime integration and dispatch
```cpp
    torch::Tensor& out,
    const torch::Tensor& a,
    const torch::Tensor& b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b) {
  static constexpr int ScaleMsPerTile = size<0>(ScalesPerTile{});
  static constexpr int ScaleGranularityM = size<0>(MmaTileShape{}) / ScaleMsPerTile;
  static constexpr int ScaleGranularityN = size<1>(MmaTileShape{}) / size<1>(ScalesPerTile{});
  static constexpr int ScaleGranularityK = size<2>(MmaTileShape{}) / size<2>(ScalesPerTile{});

  using ElementAB = cutlass::float_e4m3_t;
  using ElementA = ElementAB;
  using ElementB = ElementAB;
  using ElementC = void;
  using ElementD = OutType;
  using LayoutA = cutlass::layout::RowMajor;
  using LayoutB = cutlass::layout::ColumnMajor;
  using LayoutD = cutlass::layout::RowMajor;
  using LayoutC = LayoutD;
  // This means both SFA and SFB are column-major.
  using ScaleConfig = cutlass::detail::Sm100BlockwiseScaleConfig<
      ScaleGranularityM,
      ScaleGranularityN,
      ScaleGranularityK,
      cute::UMMA::Major::MN,
      cute::UMMA::Major::K>;
  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());
```
**EN:** This section uses `decltype`, `ScaleMsPerTile`, `ScaleGranularityM` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`decltype`、`ScaleMsPerTile`、`ScaleGranularityM`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 77-103: Device helpers and synchronization
```cpp
  static constexpr int AlignmentA = 128 / cutlass::sizeof_bits<ElementA>::value;
  static constexpr int AlignmentB = 128 / cutlass::sizeof_bits<ElementB>::value;
  static constexpr int AlignmentD = 128 / cutlass::sizeof_bits<ElementD>::value;
  static constexpr int AlignmentC = AlignmentD;

  using ElementAccumulator = float;
  using ElementBlockScale = float;
  using ElementCompute = float;
  using ArchTag = cutlass::arch::Sm100;
  using OperatorClass = cutlass::arch::OpClassTensorOp;

  using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
      ArchTag,
      cutlass::arch::OpClassTensorOp,
      PerSmTileShape,
      ClusterShape,
      EpilogueTileShape,
      ElementAccumulator,
      ElementCompute,
      ElementC,
      LayoutC,
      AlignmentC,
      ElementD,
      LayoutD,
      AlignmentD,
      cutlass::epilogue::TmaWarpSpecialized1Sm>::CollectiveOp;
```
**EN:** This section implements `AlignmentA`, `AlignmentB`, `AlignmentD`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`AlignmentA`、`AlignmentB`、`AlignmentD`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 104-128: Device helpers and synchronization
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
      MmaTileShape,
      ClusterShape,
      cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
          sizeof(typename CollectiveEpilogue::SharedStorage))>,
      cutlass::gemm::KernelTmaWarpSpecializedBlockwise1SmSm100>::CollectiveOp;

  using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
      Shape<int, int, int, int>,
      CollectiveMainloop,
      CollectiveEpilogue,
      cutlass::gemm::PersistentScheduler>;
  using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;

  Gemm gemm_op;
```
**EN:** This section implements `CollectiveMainloop`, `GemmKernel`, `Gemm`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`CollectiveMainloop`、`GemmKernel`、`Gemm`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 129-152: Templates, aliases, and constants
```cpp
  int m = a.size(0);
  int k = a.size(1);
  int n = b.size(1);

  auto a_ptr = static_cast<ElementAB*>(a.data_ptr());
  auto b_ptr = static_cast<ElementAB*>(b.data_ptr());
  auto scales_a_ptr = static_cast<float*>(scales_a.data_ptr());
  auto scales_b_ptr = static_cast<float*>(scales_b.data_ptr());
  auto c_ptr = static_cast<ElementD*>(out.data_ptr());

  using StrideA = typename GemmKernel::StrideA;
  using StrideB = typename GemmKernel::StrideB;
  using StrideD = typename GemmKernel::StrideD;
  using StrideC = typename GemmKernel::StrideD;

  StrideA a_stride = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(m, k, 1));
  StrideB b_stride = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(n, k, 1));
  StrideC c_stride = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(m, n, 1));
  LayoutSFA layout_SFA = ScaleConfig::tile_atom_to_shape_SFA(make_shape(m, n, k, 1));
  LayoutSFB layout_SFB = ScaleConfig::tile_atom_to_shape_SFB(make_shape(m, n, k, 1));

  typename GemmKernel::MainloopArguments mainloop_args{
      a_ptr, a_stride, b_ptr, b_stride, scales_a_ptr, layout_SFA, scales_b_ptr, layout_SFB};
```
**EN:** This section defines `data_ptr`, `make_shape`, `tile_atom_to_shape_SFA`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`data_ptr`、`make_shape`、`tile_atom_to_shape_SFA`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 153-173: Runtime integration and dispatch
```cpp
  typename GemmKernel::EpilogueArguments epilogue_args{{}, c_ptr, c_stride, c_ptr, c_stride};
  epilogue_args.thread.alpha = 1.0f;

  typename GemmKernel::Arguments args = {
      cutlass::gemm::GemmUniversalMode::kGemm, {m, n, k, 1}, mainloop_args, epilogue_args};

  auto can_implement = gemm_op.can_implement(args);
  TORCH_CHECK(can_implement == cutlass::Status::kSuccess, cutlassGetStatusString(can_implement))

  size_t workspace_size = gemm_op.get_workspace_size(args);
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

  auto init_status = gemm_op.initialize(args, workspace.get());
  TORCH_CHECK(init_status == cutlass::Status::kSuccess, cutlassGetStatusString(init_status));

  auto stream = at::cuda::getCurrentCUDAStream(a.get_device());

  auto status = gemm_op.run(stream);
  TORCH_CHECK(status == cutlass::Status::kSuccess, cutlassGetStatusString(status))
}
```
**EN:** This section uses `can_implement`, `TORCH_CHECK`, `workspace` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`can_implement`、`TORCH_CHECK`、`workspace`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 174-197: Runtime integration and dispatch
```cpp
template <typename OutType>
void sm100_fp8_blockwise_dispatch_shape(
    torch::Tensor& out,
    const torch::Tensor& a,
    const torch::Tensor& b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b) {
  if (a.size(0) <= 128) {
    using MmaTileShape = Shape<_64, _128, _128>;
    using PerSmTileShape = Shape<_64, _128, _128>;
    using EpilogueTileShape = Shape<_64, _64>;
    using ScalesPerTile = Shape<_64, _1, _1>;
    launch_sm100_fp8_blockwise_scaled_mm<OutType, MmaTileShape, PerSmTileShape, EpilogueTileShape, ScalesPerTile>(
        out, a, b, scales_a, scales_b);
  } else {
    using MmaTileShape = Shape<_128, _128, _128>;
    using PerSmTileShape = Shape<_128, _128, _128>;
    using EpilogueTileShape = Shape<_128, _64>;
    using ScalesPerTile = Shape<_128, _1, _1>;
    launch_sm100_fp8_blockwise_scaled_mm<OutType, MmaTileShape, PerSmTileShape, EpilogueTileShape, ScalesPerTile>(
        out, a, b, scales_a, scales_b);
  }
}
```
**EN:** This section uses `sm100_fp8_blockwise_dispatch_shape`, `MmaTileShape`, `PerSmTileShape` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`sm100_fp8_blockwise_dispatch_shape`、`MmaTileShape`、`PerSmTileShape`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 198-220: Types and data layout
```cpp
template <
    typename OutType,
    typename MmaTileShape,
    typename PerSmTileShape,
    typename EpilogueTileShape,
    typename ScalesPerTile,
    int TileSizeM_ = 128,
    class ClusterShape = Shape<_1, _1, _1>>
void launch_sm120_fp8_blockwise_scaled_mm(
    torch::Tensor& out,
    const torch::Tensor& a,
    const torch::Tensor& b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b) {
  using ElementBlockScale = float;

  // A matrix configuration
  using ElementA = cutlass::float_e4m3_t;        // Element type for A matrix operand
  using LayoutATag = cutlass::layout::RowMajor;  // Layout type for A matrix operand
  constexpr int AlignmentA =
      128 / cutlass::sizeof_bits<ElementA>::value;  // Memory access granularity/alignment of A matrix in units of
                                                    // elements (up to 16 bytes)
```
**EN:** This section defines `ClusterShape`, `launch_sm120_fp8_blockwise_scaled_mm`, `ElementBlockScale`, packaging state, metadata, or memory layout information that later kernels and wrappers consume.
**CN:** 本段定义了`ClusterShape`、`launch_sm120_fp8_blockwise_scaled_mm`、`ElementBlockScale`等内容，用于封装后续内核与封装层会使用的状态、元数据或内存布局信息。

### Lines 221-243: Templates, aliases, and constants
```cpp
  // B matrix configuration
  using ElementB = cutlass::float_e4m3_t;           // Element type for B matrix operand
  using LayoutBTag = cutlass::layout::ColumnMajor;  // Layout type for B matrix operand
  constexpr int AlignmentB =
      128 / cutlass::sizeof_bits<ElementB>::value;  // Memory access granularity/alignment of B matrix in units of
                                                    // elements (up to 16 bytes)

  // C/D matrix configuration
  using ElementD = OutType;                      // Element type for D matrix operand
  using ElementC = void;                         // Element type for C matrix operand
  using LayoutCTag = cutlass::layout::RowMajor;  // Layout type for C matrix operand
  using LayoutDTag = cutlass::layout::RowMajor;  // Layout type for D matrix operand
  constexpr int AlignmentD =
      128 / cutlass::sizeof_bits<ElementD>::value;  // Memory access granularity/alignment of C matrix in units of
                                                    // elements (up to 16 bytes)
  constexpr int AlignmentC =
      AlignmentD;  // Memory access granularity/alignment of C matrix in units of elements (up to 16 bytes)

  // Kernel functional config
  using ElementAccumulator = float;      // Element type for internal accumulation
  using ArchTag = cutlass::arch::Sm120;  // Tag indicating the minimum SM that supports the intended feature
  using OperatorClass = cutlass::arch::OpClassTensorOp;  // Operator class tag - changed from OpClassBlockScaledTensorOp
```
**EN:** This section defines `ElementB`, `LayoutBTag`, `AlignmentB`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`ElementB`、`LayoutBTag`、`AlignmentB`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 244-268: Templates, aliases, and constants
```cpp
  static constexpr int ScaleMsPerTile = size<0>(ScalesPerTile{});
  static constexpr int ScaleGranularityM = size<0>(MmaTileShape{}) / ScaleMsPerTile;
  static constexpr int ScaleGranularityN = size<1>(MmaTileShape{}) / size<1>(ScalesPerTile{});
  static constexpr int ScaleGranularityK = size<2>(MmaTileShape{}) / size<2>(ScalesPerTile{});

  using ScaleConfig = cutlass::detail::Sm120BlockwiseScaleConfig<
      ScaleGranularityM,
      ScaleGranularityN,
      ScaleGranularityK,
      cute::UMMA::Major::MN,
      cute::UMMA::Major::K>;
  // FP8 Block-wise scaling configuration
  using LayoutSFA = decltype(ScaleConfig::deduce_layoutSFA());  // Layout type for SFA matrix operand
  using LayoutSFB = decltype(ScaleConfig::deduce_layoutSFB());  // Layout type for SFB matrix operand

  constexpr bool kCanUsePingpong = (64 % ScaleGranularityM == 0);

  int m = a.size(0);
  int k = a.size(1);
  int n = b.size(1);

  auto a_ptr = static_cast<ElementA*>(a.data_ptr());
  auto b_ptr = static_cast<ElementB*>(b.data_ptr());
  auto c_ptr = static_cast<ElementD*>(out.data_ptr());
```
**EN:** This section defines `decltype`, `data_ptr`, `ScaleMsPerTile`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`decltype`、`data_ptr`、`ScaleMsPerTile`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 269-293: Templates, aliases, and constants
```cpp
  auto scales_a_ptr = static_cast<ElementBlockScale*>(scales_a.data_ptr());
  auto scales_b_ptr = static_cast<ElementBlockScale*>(scales_b.data_ptr());

  LayoutSFA layout_SFA = ScaleConfig::tile_atom_to_shape_SFA(make_shape(m, n, k, 1));
  LayoutSFB layout_SFB = ScaleConfig::tile_atom_to_shape_SFB(make_shape(m, n, k, 1));

  auto run_gemm = [&](auto tag) -> cutlass::Status {
    using GemmKernel = decltype(tag);
    using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
    Gemm gemm_op;

    using StrideA = typename GemmKernel::StrideA;
    using StrideB = typename GemmKernel::StrideB;
    using StrideC = typename GemmKernel::StrideD;

    StrideA stride_a = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(m, k, 1));
    StrideB stride_b = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(n, k, 1));
    StrideC stride_c = cutlass::make_cute_packed_stride(StrideC{}, cute::make_shape(m, n, 1));

    typename GemmKernel::MainloopArguments mainloop_args{
        a_ptr, stride_a, b_ptr, stride_b, scales_a_ptr, layout_SFA, scales_b_ptr, layout_SFB};

    typename GemmKernel::EpilogueArguments epilogue_args{{}, c_ptr, stride_c, c_ptr, stride_c};
    epilogue_args.thread.alpha = 1.0f;
```
**EN:** This section defines `data_ptr`, `tile_atom_to_shape_SFA`, `tile_atom_to_shape_SFB`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`data_ptr`、`tile_atom_to_shape_SFA`、`tile_atom_to_shape_SFB`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 294-317: Runtime integration and dispatch
```cpp
    typename Gemm::Arguments args = {
        cutlass::gemm::GemmUniversalMode::kGemm,
        {m, n, k, 1},
        mainloop_args,
        epilogue_args,
    };

    auto can_implement = gemm_op.can_implement(args);
    if (can_implement != cutlass::Status::kSuccess) {
      return can_implement;
    }

    size_t workspace_size = gemm_op.get_workspace_size(args);
    cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);

    auto init_status = gemm_op.initialize(args, workspace.get());
    if (init_status != cutlass::Status::kSuccess) {
      return init_status;
    }

    auto stream = at::cuda::getCurrentCUDAStream(a.get_device());
    return gemm_op.run(stream);
  };
```
**EN:** This section uses `can_implement`, `get_workspace_size`, `workspace` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`can_implement`、`get_workspace_size`、`workspace`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 318-336: Templates, aliases, and constants
```cpp
  using CooperativeCollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
      ArchTag,
      OperatorClass,
      PerSmTileShape,
      ClusterShape,
      cutlass::epilogue::collective::EpilogueTileAuto,
      ElementAccumulator,
      ElementAccumulator,
      ElementC,
      LayoutCTag,
      AlignmentC,
      ElementD,
      LayoutDTag,
      AlignmentD,
      cutlass::epilogue::collective::EpilogueScheduleAuto>::CollectiveOp;

  using CooperativeStageCount = cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
      sizeof(typename CooperativeCollectiveEpilogue::SharedStorage))>;
```
**EN:** This section defines `CooperativeCollectiveEpilogue`, `CooperativeStageCount`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`CooperativeCollectiveEpilogue`、`CooperativeStageCount`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 337-354: Templates, aliases, and constants
```cpp
  using CooperativeCollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
      ArchTag,
      OperatorClass,
      ElementA,
      cute::tuple<LayoutATag, LayoutSFA>,
      AlignmentA,
      ElementB,
      cute::tuple<LayoutBTag, LayoutSFB>,
      AlignmentB,
      ElementAccumulator,
      MmaTileShape,
      ClusterShape,
      CooperativeStageCount,
      cutlass::gemm::KernelScheduleSm120Blockwise>::CollectiveOp;

  using CooperativeGemmKernel = cutlass::gemm::kernel::
      GemmUniversal<Shape<int, int, int, int>, CooperativeCollectiveMainloop, CooperativeCollectiveEpilogue, void>;
```
**EN:** This section defines `CooperativeCollectiveMainloop`, `CooperativeGemmKernel`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`CooperativeCollectiveMainloop`、`CooperativeGemmKernel`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 355-376: Templates, aliases, and constants
```cpp
  cutlass::Status status = cutlass::Status::kSuccess;
  if constexpr (kCanUsePingpong) {
    using PingpongMmaTileShape_MNK = Shape<_64, _128, _128>;
    using PingpongCollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
        ArchTag,
        OperatorClass,
        PerSmTileShape,
        ClusterShape,
        cutlass::epilogue::collective::EpilogueTileAuto,
        ElementAccumulator,
        ElementAccumulator,
        ElementC,
        LayoutCTag,
        AlignmentC,
        ElementD,
        LayoutDTag,
        AlignmentD,
        cutlass::epilogue::collective::EpilogueScheduleAuto>::CollectiveOp;

    using PingpongStageCount = cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
        sizeof(typename PingpongCollectiveEpilogue::SharedStorage))>;
```
**EN:** This section defines `PingpongMmaTileShape_MNK`, `PingpongCollectiveEpilogue`, `PingpongStageCount`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`PingpongMmaTileShape_MNK`、`PingpongCollectiveEpilogue`、`PingpongStageCount`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 377-399: Device helpers and synchronization
```cpp
    using PingpongCollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
        ArchTag,
        OperatorClass,
        ElementA,
        cute::tuple<LayoutATag, LayoutSFA>,
        AlignmentA,
        ElementB,
        cute::tuple<LayoutBTag, LayoutSFB>,
        AlignmentB,
        ElementAccumulator,
        PingpongMmaTileShape_MNK,
        ClusterShape,
        PingpongStageCount,
        cutlass::gemm::KernelTmaWarpSpecializedBlockwisePingpongSm120>::CollectiveOp;

    using PingpongGemmKernel = cutlass::gemm::kernel::
        GemmUniversal<Shape<int, int, int, int>, PingpongCollectiveMainloop, PingpongCollectiveEpilogue, void>;

    if (m <= 64) {
      status = run_gemm(PingpongGemmKernel{});
      if (status != cutlass::Status::kSuccess) {
        status = run_gemm(CooperativeGemmKernel{});
      }
```
**EN:** This section implements `PingpongCollectiveMainloop`, `PingpongGemmKernel`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`PingpongCollectiveMainloop`、`PingpongGemmKernel`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 400-423: Runtime integration and dispatch
```cpp
    } else {
      status = run_gemm(CooperativeGemmKernel{});
    }
  } else {
    status = run_gemm(CooperativeGemmKernel{});
  }

  TORCH_CHECK(status == cutlass::Status::kSuccess, cutlassGetStatusString(status));
}

template <typename OutType>
void sm120_fp8_blockwise_dispatch_shape(
    torch::Tensor& out,
    const torch::Tensor& a,
    const torch::Tensor& b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b) {
  using MmaTileShape = Shape<_128, _128, _128>;
  using PerSmTileShape = Shape<_128, _128, _128>;
  using EpilogueTileShape = Shape<_128, _64>;
  using ScalesPerTile = Shape<_128, _1, _1>;
  launch_sm120_fp8_blockwise_scaled_mm<OutType, MmaTileShape, PerSmTileShape, EpilogueTileShape, ScalesPerTile>(
      out, a, b, scales_a, scales_b);
}
```
**EN:** This section uses `sm120_fp8_blockwise_dispatch_shape`, `TORCH_CHECK`, `MmaTileShape` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`sm120_fp8_blockwise_dispatch_shape`、`TORCH_CHECK`、`MmaTileShape`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 424-446: Runtime integration and dispatch
```cpp

torch::Tensor fp8_blockwise_scaled_mm(
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const torch::Dtype& out_dtype) {
  TORCH_CHECK(mat_a.is_cuda(), "mat_a must be a CUDA tensor");
  TORCH_CHECK(mat_b.is_cuda(), "mat_b must be a CUDA tensor");
  TORCH_CHECK(mat_a.dim() == 2, "mat_a must be a 2D tensor");
  TORCH_CHECK(mat_b.dim() == 2, "mat_b must be a 2D tensor");
  TORCH_CHECK(mat_a.stride(1) == 1, "mat_a must be a row major tensor");
  TORCH_CHECK(mat_b.stride(0) == 1, "mat_b must be a column major tensor");
  TORCH_CHECK(mat_a.size(1) == mat_b.size(0), "mat_a and mat_b shapes cannot be multiplied");

  TORCH_CHECK(
      (mat_a.size(1) * mat_a.element_size()) % 16 == 0, "mat_a must be multiple of 16 bytes for memory alignment");
  TORCH_CHECK(
      (mat_b.size(0) * mat_b.element_size()) % 16 == 0, "mat_b must be multiple of 16 bytes for memory alignment");
  TORCH_CHECK(mat_a.scalar_type() == torch::kFloat8_e4m3fn, "mat_a must be Float8_e4m3fn");
  TORCH_CHECK(mat_b.scalar_type() == torch::kFloat8_e4m3fn, "mat_b must be Float8_e4m3fn");
  TORCH_CHECK(out_dtype == torch::kHalf || out_dtype == torch::kBFloat16, "out_dtype must be Half or BFloat16");
```
**EN:** This section uses `fp8_blockwise_scaled_mm`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`fp8_blockwise_scaled_mm`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 447-471: Runtime integration and dispatch
```cpp
  auto is_contiguous_vector = [](const torch::Tensor& t) {
    auto t_sizes = t.sizes();
    return t.is_contiguous() &&
           (t.dim() == 1 || (t.dim() == 2 && *std::min_element(t_sizes.begin(), t_sizes.end()) == 1));
  };

  TORCH_CHECK(mat_a.size(0) == scales_a.size(0), "size of scales_a is not matched");
  TORCH_CHECK(mat_a.size(1) / 128 == scales_a.size(1), "size of scales_a is not matched");
  TORCH_CHECK(scales_a.stride(0) == 1 || is_contiguous_vector(scales_a), "scales_a must be M major");
  TORCH_CHECK(mat_b.size(0) / 128 == scales_b.size(0), "size of scales_b is not matched");
  TORCH_CHECK(mat_b.size(1) / 128 == scales_b.size(1), "size of scales_b is not matched");
  TORCH_CHECK(scales_b.stride(0) == 1 || is_contiguous_vector(scales_b), "scales_b must be K major");
  TORCH_CHECK(scales_a.scalar_type() == torch::kFloat32, "scales_a must be Float32");
  TORCH_CHECK(scales_b.scalar_type() == torch::kFloat32, "scales_b must be Float32");

  torch::Tensor out = torch::empty({mat_a.size(0), mat_b.size(1)}, mat_a.options().dtype(out_dtype));
  TORCH_CHECK((out.size(1) * out.element_size()) % 16 == 0, "out must be multiple of 16 bytes for memory alignment");

  auto sm_version = getSMVersion();

  int64_t original_rows = mat_a.size(0);
  torch::Tensor mat_a_padded = pad_tensor(mat_a, /*alignment=*/4);
  torch::Tensor scales_a_padded = pad_tensor(scales_a, /*alignment=*/4, /*col_major=*/true);
  torch::Tensor out_padded = torch::empty({mat_a_padded.size(0), mat_b.size(1)}, out.options());
```
**EN:** This section uses `sizes`, `is_contiguous`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`sizes`、`is_contiguous`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 472-500: Runtime integration and dispatch
```cpp
#if defined(CUTLASS_ARCH_MMA_SM90_SUPPORTED)
#if defined CUDA_VERSION && CUDA_VERSION >= 12000
  if (sm_version == 90) {
    torch::Tensor scales_b_contiguous = scales_b.contiguous();
    if (out_dtype == torch::kBFloat16) {
      cutlass_gemm_blockwise_sm90_fp8_dispatch<cutlass::bfloat16_t>(
          out_padded, mat_a_padded, mat_b, scales_a_padded, scales_b_contiguous);
    } else {
      cutlass_gemm_blockwise_sm90_fp8_dispatch<cutlass::half_t>(
          out_padded, mat_a_padded, mat_b, scales_a_padded, scales_b_contiguous);
    }
    return out_padded.slice(0, 0, original_rows);
  }
#endif
#endif

#if defined(CUTLASS_ARCH_MMA_SM100A_SUPPORTED) || defined(CUTLASS_ARCH_MMA_SM100_SUPPORTED)
#if defined CUDA_VERSION && CUDA_VERSION >= 12080
  if (sm_version == 100
#if CUDA_VERSION >= 12090
      || sm_version == 103
#endif
  ) {
    if (out_dtype == torch::kBFloat16) {
      sm100_fp8_blockwise_dispatch_shape<cutlass::bfloat16_t>(
          out_padded, mat_a_padded, mat_b, scales_a_padded, scales_b);
    } else {
      sm100_fp8_blockwise_dispatch_shape<cutlass::half_t>(out_padded, mat_a_padded, mat_b, scales_a_padded, scales_b);
    }
```
**EN:** This section uses `defined`, `contiguous`, `slice` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`defined`、`contiguous`、`slice`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 501-522: Runtime integration and dispatch
```cpp
    return out_padded.slice(0, 0, original_rows);
  }
#endif
#endif

#if defined(CUTLASS_ARCH_MMA_SM120A_SUPPORTED) || defined(CUTLASS_ARCH_MMA_SM120_SUPPORTED)
#if defined(CUDA_VERSION) && CUDA_VERSION >= 12080
  if (sm_version >= 120) {
    if (out_dtype == torch::kBFloat16) {
      sm120_fp8_blockwise_dispatch_shape<cutlass::bfloat16_t>(
          out_padded, mat_a_padded, mat_b, scales_a_padded, scales_b);
    } else {
      sm120_fp8_blockwise_dispatch_shape<cutlass::half_t>(out_padded, mat_a_padded, mat_b, scales_a_padded, scales_b);
    }
    return out_padded.slice(0, 0, original_rows);
  }
#endif
#endif

  TORCH_CHECK_NOT_IMPLEMENTED(
      false, "No implemented fp8_blockwise_scaled_mm for current compute capability: ", sm_version);
}
```
**EN:** This section uses `defined`, `slice`, `TORCH_CHECK_NOT_IMPLEMENTED` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`defined`、`slice`、`TORCH_CHECK_NOT_IMPLEMENTED`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **PyTorch extension / PyTorch 扩展**: Bridges low-level kernels with PyTorch tensor/runtime abstractions. / 把底层内核与 PyTorch 张量/运行时抽象连接起来。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cutlass_extensions/gemm/cutlass_gemm_caller.cuh`, `cutlass_extensions/gemm/fp8_blockwise_gemm_sm90_dispatch.cuh`, `utils.h`
- **External headers / 外部头文件**: `ATen/cuda/CUDAContext.h`, `cudaTypedefs.h`, `cutlass/arch/arch.h`, `cutlass/arch/memory.h`, `cutlass/arch/mma.h`, `cutlass/array.h`, `cutlass/cutlass.h`, `cutlass/epilogue/thread/activation.h`, `cutlass/epilogue/thread/linear_combination.h`, `cutlass/epilogue/threadblock/default_thread_map_tensor_op.h`, `cutlass/gemm/device/gemm.h`, `cutlass/gemm/device/gemm_universal_adapter.h`
- **Path context / 路径上下文**: gemm / fp8_blockwise_gemm_kernel.cu
