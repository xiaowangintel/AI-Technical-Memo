# int8_gemm_kernel.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/csrc/gemm/int8_gemm_kernel.cu`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides GEMM / matrix multiplication kernels, tiling strategies, or low-level launch wrappers. This source file primarily implements kernels, host-side wrappers, bindings, or launch logic. / 提供 GEMM / 矩阵乘内核、分块策略或底层启动封装。 该源文件主要实现内核、主机侧封装、绑定或启动逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: Headers and compile-time setup
```cpp
/* Copyright 2025 SGLang Team. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
==============================================================================*/

#include <ATen/cuda/CUDAContext.h>
#include <cutlass/cutlass.h>
#include <cutlass/epilogue/thread/linear_combination.h>
#include <cutlass/epilogue/threadblock/epilogue_with_visitor.h>
#include <cutlass/gemm/device/gemm.h>
#include <cutlass/gemm/device/gemm_universal_adapter.h>
#include <cutlass/numeric_types.h>

#include <cute/atom/mma_atom.hpp>
#include <cute/tensor.hpp>
#include <cutlass/epilogue/collective/collective_builder.hpp>
#include <cutlass/gemm/collective/collective_builder.hpp>
#include <cutlass/gemm/kernel/gemm_universal.hpp>
#include <cutlass/util/packed_stride.hpp>
```
**EN:** This section prepares the file's headers, macros, and compile-time switches so later code can rely on the right platform and API definitions.
**CN:** 本段准备文件依赖的头文件、宏和编译期开关，使后续代码能够依赖正确的平台与 API 定义。

### Lines 31-59: Device helpers and synchronization
```cpp
#include "cutlass_extensions/epilogue/epilogue_per_row_per_col_scale.h"
#include "cutlass_extensions/gemm/gemm_universal_base_compat.h"
#include "cutlass_extensions/gemm/gemm_with_epilogue_visitor.h"
#include "utils.h"

using namespace cute;

template <
    typename ElementOutput,
    typename ArchTag,
    typename ThreadblockShape,
    typename WarpShape,
    typename InstructionShape,
    int NumStages>
void cutlass_int8_scaled_mm(
    torch::Tensor& out,
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const c10::optional<torch::Tensor>& bias) {
  using ElementAccumulator = int32_t;
  using ElementCompute = float;
  using ElementInputA = int8_t;
  using ElementInputB = int8_t;

  using OperatorClass = cutlass::arch::OpClassTensorOp;
  using ThreadblockSwizzle = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<8>;
```
**EN:** This section implements `cutlass_int8_scaled_mm`, `cute`, `ElementAccumulator`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`cutlass_int8_scaled_mm`、`cute`、`ElementAccumulator`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 60-84: Device helpers and synchronization
```cpp
  using DefaultGemmConf = cutlass::gemm::device::
      DefaultGemmConfiguration<OperatorClass, ArchTag, ElementInputA, ElementInputB, ElementOutput, ElementCompute>;
  using EpilogueOutputOp = typename DefaultGemmConf::EpilogueOutputOp;

  using GemmKernel_ = typename cutlass::gemm::kernel::DefaultGemm<
      ElementInputA,
      cutlass::layout::RowMajor,
      DefaultGemmConf::kAlignmentA,
      ElementInputB,
      cutlass::layout::ColumnMajor,
      DefaultGemmConf::kAlignmentB,
      ElementOutput,
      cutlass::layout::RowMajor,
      ElementAccumulator,
      OperatorClass,
      ArchTag,
      ThreadblockShape,
      WarpShape,
      InstructionShape,
      EpilogueOutputOp,
      ThreadblockSwizzle,
      NumStages,
      true,
      typename DefaultGemmConf::Operator>::GemmKernel;
```
**EN:** This section implements `DefaultGemmConf`, `EpilogueOutputOp`, `GemmKernel_`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`DefaultGemmConf`、`EpilogueOutputOp`、`GemmKernel_`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 85-112: Templates, aliases, and constants
```cpp
  using AlphaColTileIterator = cutlass::epilogue::threadblock::PredicatedTileIterator<
      cutlass::epilogue::threadblock::OutputTileOptimalThreadMap<
          typename GemmKernel_::Epilogue::OutputTileIterator::ThreadMap::Shape,
          typename GemmKernel_::Epilogue::OutputTileIterator::ThreadMap::Count,
          GemmKernel_::Epilogue::OutputTileIterator::ThreadMap::kThreads,
          GemmKernel_::Epilogue::OutputTileIterator::kElementsPerAccess,
          cutlass::sizeof_bits<ElementOutput>::value>,
      ElementCompute>;

  using EpilogueVisitor = typename cutlass::epilogue::threadblock::EpilogueVisitorPerRowPerCol<
      ThreadblockShape,
      GemmKernel_::kThreadCount,
      AlphaColTileIterator,
      typename GemmKernel_::Epilogue::OutputTileIterator,
      ElementAccumulator,
      ElementCompute,
      EpilogueOutputOp>;

  using Epilogue = typename cutlass::epilogue::threadblock::
      EpilogueWithVisitorFromExistingEpilogue<EpilogueVisitor, typename GemmKernel_::Epilogue>::Epilogue;

  using GemmKernel =
      cutlass::gemm::kernel::GemmWithEpilogueVisitor<typename GemmKernel_::Mma, Epilogue, ThreadblockSwizzle>;

  using Gemm = cutlass::gemm::device::GemmUniversalBaseCompat<GemmKernel>;

  Gemm gemm_op;
```
**EN:** This section defines `AlphaColTileIterator`, `EpilogueVisitor`, `Epilogue`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`AlphaColTileIterator`、`EpilogueVisitor`、`Epilogue`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 113-139: Control flow and branching
```cpp
  int m = mat_a.size(0);
  int k = mat_a.size(1);
  int n = mat_b.size(1);

  auto a_ptr = static_cast<ElementInputA*>(mat_a.data_ptr());
  auto b_ptr = static_cast<ElementInputB*>(mat_b.data_ptr());
  auto o_ptr = static_cast<ElementOutput*>(out.data_ptr());

  auto a_s_ptr = static_cast<ElementCompute*>(scales_a.data_ptr());
  auto b_s_ptr = static_cast<ElementCompute*>(scales_b.data_ptr());

  int64_t lda = mat_a.stride(0);
  int64_t ldb = mat_b.stride(1);
  int64_t ldd = out.stride(0);

  ElementOutput* bias_ptr = nullptr;
  int64_t ldc = 0;
  if (bias) {
    bias_ptr = static_cast<ElementOutput*>(bias->data_ptr());
  }

  typename EpilogueOutputOp::Params linearScalingParams;
  typename EpilogueVisitor::Arguments visitor_args{linearScalingParams};

  typename Gemm::Arguments args{
      {m, n, k}, {a_ptr, lda}, {b_ptr, ldb}, {b_s_ptr, 0}, {a_s_ptr, 0}, {bias_ptr, ldc}, {o_ptr, ldd}, visitor_args};
```
**EN:** This section drives `data_ptr`, `stride` through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进`data_ptr`、`stride`相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 140-167: Runtime integration and dispatch
```cpp
  auto workspace = torch::empty(
      gemm_op.get_workspace_size(args), torch::TensorOptions().dtype(torch::kUInt8).device(mat_a.device()));

  auto stream = at::cuda::getCurrentCUDAStream(mat_a.get_device());

  auto can_implement = gemm_op.can_implement(args);
  TORCH_CHECK(
      can_implement == cutlass::Status::kSuccess,
      "gemm cannot implement, error: ",
      cutlassGetStatusString(can_implement));

  auto status = gemm_op(args, workspace.data_ptr(), stream);
  TORCH_CHECK(status == cutlass::Status::kSuccess, "gemm executioin failed, error: ", cutlassGetStatusString(status));
}

template <typename ElementOutput, typename ArchTag, typename InstructionShape>
void sm75_dispatch_shape(
    torch::Tensor& out,
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const c10::optional<torch::Tensor>& bias) {
  int m = mat_a.size(0);
  if (m <= 32) {
    cutlass_int8_scaled_mm<
        ElementOutput,
        ArchTag,
```
**EN:** This section uses `sm75_dispatch_shape`, `empty`, `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`sm75_dispatch_shape`、`empty`、`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 168-196: Control flow and branching
```cpp
        cutlass::gemm::GemmShape<32, 128, 64>,
        cutlass::gemm::GemmShape<32, 64, 64>,
        InstructionShape,
        2>(out, mat_a, mat_b, scales_a, scales_b, bias);
  } else if (m <= 64) {
    cutlass_int8_scaled_mm<
        ElementOutput,
        ArchTag,
        cutlass::gemm::GemmShape<64, 128, 128>,
        cutlass::gemm::GemmShape<64, 64, 64>,
        InstructionShape,
        2>(out, mat_a, mat_b, scales_a, scales_b, bias);
  } else if (m <= 256) {
    cutlass_int8_scaled_mm<
        ElementOutput,
        ArchTag,
        cutlass::gemm::GemmShape<128, 128, 128>,
        cutlass::gemm::GemmShape<64, 64, 64>,
        InstructionShape,
        2>(out, mat_a, mat_b, scales_a, scales_b, bias);
  } else {
    cutlass_int8_scaled_mm<
        ElementOutput,
        ArchTag,
        cutlass::gemm::GemmShape<128, 128, 64>,
        cutlass::gemm::GemmShape<64, 64, 64>,
        InstructionShape,
        2>(out, mat_a, mat_b, scales_a, scales_b, bias);
  }
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 197-226: Runtime integration and dispatch
```cpp
}

template <typename ElementOutput, typename ArchTag, typename InstructionShape>
void sm80_dispatch_shape(
    torch::Tensor& out,
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const c10::optional<torch::Tensor>& bias) {
  int m = mat_a.size(0);
  int n = mat_b.size(1);
  if (m <= 16) {
    if (n <= 4096) {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<16, 64, 128>,
          cutlass::gemm::GemmShape<16, 64, 64>,
          InstructionShape,
          6>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<16, 64, 128>,
          cutlass::gemm::GemmShape<16, 64, 64>,
          InstructionShape,
          5>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
```
**EN:** This section uses `sm80_dispatch_shape` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`sm80_dispatch_shape`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 227-254: Control flow and branching
```cpp
  } else if (m <= 32) {
    if (n <= 4096) {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<32, 64, 128>,
          cutlass::gemm::GemmShape<32, 64, 64>,
          InstructionShape,
          6>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<32, 64, 128>,
          cutlass::gemm::GemmShape<32, 64, 64>,
          InstructionShape,
          5>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
  } else if (m <= 64) {
    if (n <= 4096) {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<64, 64, 128>,
          cutlass::gemm::GemmShape<32, 64, 64>,
          InstructionShape,
          5>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 255-281: Control flow and branching
```cpp
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<64, 128, 128>,
          cutlass::gemm::GemmShape<64, 64, 64>,
          InstructionShape,
          5>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
  } else if (m <= 128 && n < 8192) {
    cutlass_int8_scaled_mm<
        ElementOutput,
        ArchTag,
        cutlass::gemm::GemmShape<64, 128, 128>,
        cutlass::gemm::GemmShape<64, 64, 64>,
        InstructionShape,
        5>(out, mat_a, mat_b, scales_a, scales_b, bias);
  } else {
    cutlass_int8_scaled_mm<
        ElementOutput,
        ArchTag,
        cutlass::gemm::GemmShape<128, 128, 64>,
        cutlass::gemm::GemmShape<64, 64, 64>,
        InstructionShape,
        5>(out, mat_a, mat_b, scales_a, scales_b, bias);
  }
}
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 282-311: Runtime integration and dispatch
```cpp
// Dispatch shape for sm89 (L40S, L20, RTX 4090), according to:
// https://github.com/vllm-project/vllm/blob/main/csrc/quantization/cutlass_w8a8/scaled_mm_c2x_sm89_int8_dispatch.cuh
template <typename ElementOutput, typename ArchTag, typename InstructionShape>
void sm89_dispatch_shape(
    torch::Tensor& out,
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const c10::optional<torch::Tensor>& bias) {
  int m = mat_a.size(0);
  int n = mat_b.size(1);
  if (m <= 16) {
    if (n <= 8192) {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<16, 64, 128>,
          cutlass::gemm::GemmShape<16, 64, 64>,
          InstructionShape,
          5>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<16, 128, 128>,
          cutlass::gemm::GemmShape<16, 64, 64>,
          InstructionShape,
          4>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
```
**EN:** This section uses `sm89_dispatch_shape` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`sm89_dispatch_shape`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 312-339: Control flow and branching
```cpp
  } else if (m <= 32) {
    if (n <= 8192) {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<32, 64, 128>,
          cutlass::gemm::GemmShape<16, 64, 64>,
          InstructionShape,
          5>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<32, 128, 128>,
          cutlass::gemm::GemmShape<32, 64, 64>,
          InstructionShape,
          4>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
  } else if (m <= 64) {
    if (n <= 8192) {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<64, 64, 128>,
          cutlass::gemm::GemmShape<32, 64, 64>,
          InstructionShape,
          5>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 340-373: Control flow and branching
```cpp
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<64, 128, 128>,
          cutlass::gemm::GemmShape<64, 64, 64>,
          InstructionShape,
          3>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
  } else if (m <= 128) {
    if (n <= 8192) {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<64, 128, 128>,
          cutlass::gemm::GemmShape<32, 64, 64>,
          InstructionShape,
          3>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else if (n <= 16384) {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<128, 128, 64>,
          cutlass::gemm::GemmShape<64, 64, 64>,
          InstructionShape,
          5>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<64, 64, 128>,
          cutlass::gemm::GemmShape<32, 64, 64>,
          InstructionShape,
          5>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 374-407: Control flow and branching
```cpp
  } else if (m <= 256) {
    if (n <= 4096) {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<64, 128, 128>,
          cutlass::gemm::GemmShape<64, 64, 64>,
          InstructionShape,
          3>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else if (n <= 8192) {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<128, 128, 64>,
          cutlass::gemm::GemmShape<64, 64, 64>,
          InstructionShape,
          5>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else if (n <= 16384) {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<256, 128, 64>,
          cutlass::gemm::GemmShape<64, 64, 64>,
          InstructionShape,
          3>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
      cutlass_int8_scaled_mm<
          ElementOutput,
          ArchTag,
          cutlass::gemm::GemmShape<128, 128, 64>,
          cutlass::gemm::GemmShape<64, 64, 64>,
          InstructionShape,
          5>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
```
**EN:** This section drives the surrounding logic through branches and loops, deciding which execution path or data-handling strategy is used.
**CN:** 本段通过分支和循环推进相关逻辑相关逻辑，决定采用哪条执行路径或数据处理策略。

### Lines 408-433: Runtime integration and dispatch
```cpp
  } else {
    cutlass_int8_scaled_mm<
        ElementOutput,
        ArchTag,
        cutlass::gemm::GemmShape<128, 128, 64>,
        cutlass::gemm::GemmShape<64, 64, 64>,
        InstructionShape,
        5>(out, mat_a, mat_b, scales_a, scales_b, bias);
  }
}

template <
    typename ElementOutput,
    typename TileShape,
    typename ClusterShape,
    typename MainloopScheduleType,
    bool WithBias>
void cutlass_int8_scaled_mm_sm90(
    torch::Tensor& out,
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const c10::optional<torch::Tensor>& bias) {
  using ArchTag = cutlass::arch::Sm90;
```
**EN:** This section uses `cutlass_int8_scaled_mm_sm90`, `ArchTag` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`cutlass_int8_scaled_mm_sm90`、`ArchTag`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 434-463: Device helpers and synchronization
```cpp
  using ElementAccumulator = int32_t;
  using ElementCompute = float;
  using ElementInputA = int8_t;
  using ElementInputB = int8_t;

  static constexpr int AlignmentA = 128 / cutlass::sizeof_bits<ElementInputA>::value;
  static constexpr int AlignmentB = 128 / cutlass::sizeof_bits<ElementInputB>::value;
  static constexpr int AlignmentC = 128 / cutlass::sizeof_bits<ElementOutput>::value;
  static constexpr int AlignmentOutput = 128 / cutlass::sizeof_bits<ElementOutput>::value;

  using OperatorClass = cutlass::arch::OpClassTensorOp;

  using EpilogueScheduleType = cutlass::epilogue::TmaWarpSpecialized;
  using TileSchedulerType = cutlass::gemm::PersistentScheduler;

  using XScale = cutlass::epilogue::fusion::
      Sm90ColBroadcast<0, TileShape, ElementCompute, ElementCompute, Stride<Int<1>, Int<0>, Int<0>>>;

  using WScale = cutlass::epilogue::fusion::
      Sm90RowBroadcast<0, TileShape, ElementCompute, ElementCompute, Stride<Int<0>, Int<1>, Int<0>>>;

  using Bias = cutlass::epilogue::fusion::
      Sm90RowBroadcast<0, TileShape, ElementOutput, ElementOutput, Stride<Int<0>, Int<1>, Int<0>>>;

  using Accum = cutlass::epilogue::fusion::Sm90AccFetch;

  // Scale
  using Compute0 = cutlass::epilogue::fusion::
      Sm90Compute<cutlass::multiplies, ElementCompute, ElementCompute, cutlass::FloatRoundStyle::round_to_nearest>;
```
**EN:** This section implements `ElementAccumulator`, `ElementCompute`, `ElementInputA`, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了`ElementAccumulator`、`ElementCompute`、`ElementInputA`等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 464-494: Templates, aliases, and constants
```cpp
  using EVTCompute0 = cutlass::epilogue::fusion::Sm90EVT<Compute0, WScale, Accum>;

  using Compute1 = cutlass::epilogue::fusion::
      Sm90Compute<cutlass::multiplies, ElementOutput, ElementCompute, cutlass::FloatRoundStyle::round_to_nearest>;

  using EVTCompute1 = cutlass::epilogue::fusion::Sm90EVT<Compute1, XScale, EVTCompute0>;

  // With bias
  using ComputeWithBias = cutlass::epilogue::fusion::
      Sm90Compute<cutlass::multiply_add, ElementOutput, ElementCompute, cutlass::FloatRoundStyle::round_to_nearest>;
  using EVTComputeWithBias = cutlass::epilogue::fusion::Sm90EVT<ComputeWithBias, XScale, EVTCompute0, Bias>;

  using EpilogueEVT = typename cutlass::platform::conditional<WithBias, EVTComputeWithBias, EVTCompute1>::type;

  using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
      ArchTag,
      OperatorClass,
      TileShape,
      ClusterShape,
      cutlass::epilogue::collective::EpilogueTileAuto,
      ElementAccumulator,
      ElementCompute,
      ElementOutput,
      cutlass::layout::RowMajor,
      AlignmentC,
      ElementOutput,
      cutlass::layout::RowMajor,
      AlignmentOutput,
      EpilogueScheduleType,
      EpilogueEVT>::CollectiveOp;
```
**EN:** This section defines `EVTCompute0`, `Compute1`, `EVTCompute1`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`EVTCompute0`、`Compute1`、`EVTCompute1`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 495-522: Templates, aliases, and constants
```cpp
  using Stages = cutlass::gemm::collective::StageCountAutoCarveout<static_cast<int>(
      sizeof(typename CollectiveEpilogue::SharedStorage))>;

  using CollectiveMainloop = typename cutlass::gemm::collective::CollectiveBuilder<
      ArchTag,
      OperatorClass,
      ElementInputA,
      cutlass::layout::RowMajor,
      AlignmentA,
      ElementInputB,
      cutlass::layout::ColumnMajor,
      AlignmentB,
      ElementAccumulator,
      TileShape,
      ClusterShape,
      Stages,
      MainloopScheduleType>::CollectiveOp;

  using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
      Shape<int, int, int, int>,  // Indicates ProblemShape
      CollectiveMainloop,
      CollectiveEpilogue,
      TileSchedulerType>;

  using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;

  Gemm gemm_op;
```
**EN:** This section defines `Stages`, `CollectiveMainloop`, `GemmKernel`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`Stages`、`CollectiveMainloop`、`GemmKernel`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 523-553: Templates, aliases, and constants
```cpp
  int m = mat_a.size(0);
  int k = mat_a.size(1);
  int n = mat_b.size(1);

  auto a_ptr = static_cast<ElementInputA*>(mat_a.data_ptr());
  auto b_ptr = static_cast<ElementInputB*>(mat_b.data_ptr());
  auto o_ptr = static_cast<ElementOutput*>(out.data_ptr());

  auto a_s_ptr = static_cast<ElementCompute*>(scales_a.data_ptr());
  auto b_s_ptr = static_cast<ElementCompute*>(scales_b.data_ptr());

  using StrideA = typename Gemm::GemmKernel::StrideA;
  using StrideB = typename Gemm::GemmKernel::StrideB;
  using StrideC = typename Gemm::GemmKernel::StrideC;
  using StrideD = typename Gemm::GemmKernel::StrideD;

  StrideA stride_a = cutlass::make_cute_packed_stride(StrideA{}, make_shape(m, k, 1));
  StrideB stride_b = cutlass::make_cute_packed_stride(StrideB{}, make_shape(n, k, 1));
  StrideC stride_c;
  StrideD stride_d = cutlass::make_cute_packed_stride(StrideD{}, make_shape(m, n, 1));

  typename Gemm::Arguments args = {
      cutlass::gemm::GemmUniversalMode::kGemm,
      {m, n, k, 1},
      {a_ptr, stride_a, b_ptr, stride_b},
      {{},  // epilogue.thread
       nullptr,
       stride_c,
       o_ptr,
       stride_d}};
```
**EN:** This section defines `data_ptr`, `make_shape`, `StrideA`, giving the file reusable compile-time abstractions, aliases, and specialization points.
**CN:** 本段定义了`data_ptr`、`make_shape`、`StrideA`等内容，为文件提供可复用的编译期抽象、类型别名和特化入口。

### Lines 554-580: Runtime integration and dispatch
```cpp
  if constexpr (WithBias) {
    ElementOutput* bias_ptr = static_cast<ElementOutput*>(bias->data_ptr());
    args.epilogue.thread = {
        {a_s_ptr},
        {{b_s_ptr}, {}, {}},
        {bias_ptr},
        {},
    };
  } else {
    args.epilogue.thread = {
        {a_s_ptr},
        {{b_s_ptr}, {}, {}},
        {},
    };
  }

  auto workspace = torch::empty(
      gemm_op.get_workspace_size(args), torch::TensorOptions().dtype(torch::kUInt8).device(mat_a.device()));

  auto stream = at::cuda::getCurrentCUDAStream(mat_a.get_device());

  auto can_implement = gemm_op.can_implement(args);
  TORCH_CHECK(
      can_implement == cutlass::Status::kSuccess,
      "gemm cannot implement, error: ",
      cutlassGetStatusString(can_implement));
```
**EN:** This section uses `data_ptr`, `empty`, `getCurrentCUDAStream` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`data_ptr`、`empty`、`getCurrentCUDAStream`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 581-608: Runtime integration and dispatch
```cpp
  auto status = gemm_op(args, workspace.data_ptr(), stream);
  TORCH_CHECK(status == cutlass::Status::kSuccess, "gemm executioin failed, error: ", cutlassGetStatusString(status));
}

template <typename ElementOutput, typename TileShape, typename ClusterShape, typename MainloopScheduleType>
void sm90_dispatch_bias(
    torch::Tensor& out,
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const c10::optional<torch::Tensor>& bias) {
  if (bias) {
    cutlass_int8_scaled_mm_sm90<ElementOutput, TileShape, ClusterShape, MainloopScheduleType, true>(
        out, mat_a, mat_b, scales_a, scales_b, bias);
  } else {
    cutlass_int8_scaled_mm_sm90<ElementOutput, TileShape, ClusterShape, MainloopScheduleType, false>(
        out, mat_a, mat_b, scales_a, scales_b, bias);
  }
}

template <typename ElementOutput>
void sm90_dispatch_shape(
    torch::Tensor& out,
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
```
**EN:** This section uses `sm90_dispatch_bias`, `gemm_op`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`sm90_dispatch_bias`、`gemm_op`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 609-639: Device helpers and synchronization
```cpp
    const c10::optional<torch::Tensor>& bias) {
  int m = mat_a.size(0);
  int n = mat_b.size(1);
  if (m <= 32) {
    if (n < 8192) {
      return sm90_dispatch_bias<
          ElementOutput,
          Shape<_64, _64, _128>,
          Shape<_1, _8, _1>,
          cutlass::gemm::KernelTmaWarpSpecialized>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
      return sm90_dispatch_bias<
          ElementOutput,
          Shape<_64, _128, _128>,
          Shape<_1, _8, _1>,
          cutlass::gemm::KernelTmaWarpSpecialized>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
  } else if (m <= 64) {
    if (n < 8192) {
      return sm90_dispatch_bias<
          ElementOutput,
          Shape<_64, _64, _128>,
          Shape<_1, _4, _1>,
          cutlass::gemm::KernelTmaWarpSpecialized>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
      return sm90_dispatch_bias<
          ElementOutput,
          Shape<_64, _64, _256>,
          Shape<_1, _1, _1>,
          cutlass::gemm::KernelTmaWarpSpecialized>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 640-662: Device helpers and synchronization
```cpp
  } else if (m <= 128) {
    if (n <= 4096) {
      return sm90_dispatch_bias<
          ElementOutput,
          Shape<_64, _64, _128>,
          Shape<_2, _1, _1>,
          cutlass::gemm::KernelTmaWarpSpecialized>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
      return sm90_dispatch_bias<
          ElementOutput,
          Shape<_64, _128, _128>,
          Shape<_2, _1, _1>,
          cutlass::gemm::KernelTmaWarpSpecialized>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
  } else {
    return sm90_dispatch_bias<
        ElementOutput,
        Shape<_128, _128, _128>,
        Shape<_2, _1, _1>,
        cutlass::gemm::KernelTmaWarpSpecializedPingpong>(out, mat_a, mat_b, scales_a, scales_b, bias);
  }
}
```
**EN:** This section implements the surrounding logic, low-level device-side helpers for arithmetic, synchronization, communication, or memory movement.
**CN:** 本段实现了相关逻辑等底层设备侧辅助逻辑，用于算术处理、同步、通信或内存搬运。

### Lines 663-690: Runtime integration and dispatch
```cpp
torch::Tensor int8_scaled_mm(
    const torch::Tensor& mat_a,
    const torch::Tensor& mat_b,
    const torch::Tensor& scales_a,
    const torch::Tensor& scales_b,
    const torch::Dtype& out_dtype,
    const c10::optional<torch::Tensor>& bias) {
  TORCH_CHECK(mat_a.is_cuda(), "mat_a must be a CUDA tensor");
  TORCH_CHECK(mat_b.is_cuda(), "mat_b must be a CUDA tensor");
  TORCH_CHECK(mat_a.dim() == 2, "mat_a must be a 2D tensor");
  TORCH_CHECK(mat_b.dim() == 2, "mat_b must be a 2D tensor");
  TORCH_CHECK(mat_a.stride(1) == 1, "mat_a must be a row major tensor");
  TORCH_CHECK(mat_b.stride(0) == 1, "mat_b must be a column major tensor");
  TORCH_CHECK(mat_a.size(1) == mat_b.size(0), "mat_a and mat_b shapes cannot be multiplied");
  TORCH_CHECK(mat_a.size(1) % 16 == 0, "mat_a.size(1) must be multiple of 16 for memory alignment");
  TORCH_CHECK(mat_b.size(0) % 16 == 0, "mat_b.size(0) must be multiple of 16 for memory alignment");
  TORCH_CHECK(mat_b.size(1) % 8 == 0, "mat_b.size(1) must be multiple of 8 for memory alignment");  // out.stride(0)
  TORCH_CHECK(mat_a.scalar_type() == torch::kInt8, "mat_a must be Int8");
  TORCH_CHECK(mat_b.scalar_type() == torch::kInt8, "mat_b must be Int8");
  TORCH_CHECK(out_dtype == torch::kHalf || out_dtype == torch::kBFloat16, "out_dtype must be Half or BFloat16");

  TORCH_CHECK(scales_a.numel() == mat_a.size(0), "size of scales_a is not matched");
  TORCH_CHECK(scales_b.numel() == mat_b.size(1), "size of scales_b is not matched");
  TORCH_CHECK(scales_a.is_contiguous(), "scales_a must be contiguous");
  TORCH_CHECK(scales_b.is_contiguous(), "scales_b msut be contiguous");
  TORCH_CHECK(scales_a.scalar_type() == torch::kFloat32, "scales_a must be Float32");
  TORCH_CHECK(scales_b.scalar_type() == torch::kFloat32, "scales_b must be Float32");
```
**EN:** This section uses `int8_scaled_mm`, `TORCH_CHECK` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`int8_scaled_mm`、`TORCH_CHECK`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 691-722: Runtime integration and dispatch
```cpp
  if (bias) {
    TORCH_CHECK(bias->numel() == mat_b.size(1), "size of bias is not matched");
    TORCH_CHECK(bias->is_contiguous(), "bias must be contiguous");
    TORCH_CHECK(bias->dtype() == out_dtype, "bias dtype must match output dtype");
  }

  torch::Tensor out = torch::empty({mat_a.size(0), mat_b.size(1)}, mat_a.options().dtype(out_dtype));

  auto sm_version = getSMVersion();

  if (sm_version >= 75 && sm_version < 80) {
    TORCH_CHECK(out_dtype == torch::kHalf, "out_dtype must be Half for SM75");
    sm75_dispatch_shape<cutlass::half_t, cutlass::arch::Sm75, cutlass::gemm::GemmShape<8, 8, 16>>(
        out, mat_a, mat_b, scales_a, scales_b, bias);
  } else if (sm_version >= 80 && sm_version < 90) {
    // sm86/sm89 has a much smaller shared memory size (100K) than sm80 (160K)
    if (sm_version == 86 || sm_version == 89) {
      if (out_dtype == torch::kBFloat16) {
        sm89_dispatch_shape<cutlass::bfloat16_t, cutlass::arch::Sm80, cutlass::gemm::GemmShape<16, 8, 32>>(
            out, mat_a, mat_b, scales_a, scales_b, bias);
      } else {
        sm89_dispatch_shape<cutlass::half_t, cutlass::arch::Sm80, cutlass::gemm::GemmShape<16, 8, 32>>(
            out, mat_a, mat_b, scales_a, scales_b, bias);
      }
    } else {
      if (out_dtype == torch::kBFloat16) {
        sm80_dispatch_shape<cutlass::bfloat16_t, cutlass::arch::Sm80, cutlass::gemm::GemmShape<16, 8, 32>>(
            out, mat_a, mat_b, scales_a, scales_b, bias);
      } else {
        sm80_dispatch_shape<cutlass::half_t, cutlass::arch::Sm80, cutlass::gemm::GemmShape<16, 8, 32>>(
            out, mat_a, mat_b, scales_a, scales_b, bias);
      }
```
**EN:** This section uses `TORCH_CHECK`, `options`, `getSMVersion` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK`、`options`、`getSMVersion`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

### Lines 723-747: Runtime integration and dispatch
```cpp
    }
  } else if (sm_version == 90) {
#if defined CUDA_VERSION && CUDA_VERSION >= 12000
    // cutlass 3.x
    if (out_dtype == torch::kBFloat16) {
      sm90_dispatch_shape<cutlass::bfloat16_t>(out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
      sm90_dispatch_shape<cutlass::half_t>(out, mat_a, mat_b, scales_a, scales_b, bias);
    }
#else
    // fallback to cutlass 2.x
    if (out_dtype == torch::kBFloat16) {
      sm80_dispatch_shape<cutlass::bfloat16_t, cutlass::arch::Sm80, cutlass::gemm::GemmShape<16, 8, 32>>(
          out, mat_a, mat_b, scales_a, scales_b, bias);
    } else {
      sm80_dispatch_shape<cutlass::half_t, cutlass::arch::Sm80, cutlass::gemm::GemmShape<16, 8, 32>>(
          out, mat_a, mat_b, scales_a, scales_b, bias);
    }
#endif
  } else {
    TORCH_CHECK_NOT_IMPLEMENTED(false, "No implemented int8_scaled_mm for current compute capability.");
  }

  return out;
}
```
**EN:** This section uses `TORCH_CHECK_NOT_IMPLEMENTED` to validate inputs, manage pointers/streams, and bridge low-level kernels to framework or runtime APIs.
**CN:** 本段通过`TORCH_CHECK_NOT_IMPLEMENTED`等逻辑完成输入校验、指针/stream 管理，并把底层内核接入框架或运行时 API。

## Key Concepts / 关键概念
- **Template specialization / 模板特化**: Relies on C++ templates to specialize code paths by type or shape. / 依赖 C++ 模板按类型或形状特化代码路径。
- **CUDA runtime / CUDA 运行时**: Uses CUDA APIs, streams, or intrinsics for NVIDIA GPUs. / 使用 CUDA API、stream 或 intrinsic 支持 NVIDIA GPU。
- **CUTLASS integration / CUTLASS 集成**: Builds on CUTLASS/CUTE abstractions for optimized kernel generation. / 基于 CUTLASS/CUTE 抽象构建优化内核。
- **Quantization / 量化**: Handles low-precision representations and conversion logic. / 处理低精度表示及其转换逻辑。

## Dependencies / 依赖关系
- **Internal headers / 内部头文件**: `cutlass_extensions/epilogue/epilogue_per_row_per_col_scale.h`, `cutlass_extensions/gemm/gemm_universal_base_compat.h`, `cutlass_extensions/gemm/gemm_with_epilogue_visitor.h`, `utils.h`
- **External headers / 外部头文件**: `ATen/cuda/CUDAContext.h`, `cutlass/cutlass.h`, `cutlass/epilogue/thread/linear_combination.h`, `cutlass/epilogue/threadblock/epilogue_with_visitor.h`, `cutlass/gemm/device/gemm.h`, `cutlass/gemm/device/gemm_universal_adapter.h`, `cutlass/numeric_types.h`, `cute/atom/mma_atom.hpp`, `cute/tensor.hpp`, `cutlass/epilogue/collective/collective_builder.hpp`, `cutlass/gemm/collective/collective_builder.hpp`, `cutlass/gemm/kernel/gemm_universal.hpp`
- **Path context / 路径上下文**: gemm / int8_gemm_kernel.cu
