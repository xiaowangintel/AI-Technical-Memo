# gemm_softmax_adapter.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `examples/nv_sycl/35_gemm_softmax/gemm_softmax_adapter.hpp`
- **Purpose / 目的:** Defines reusable declarations and helper logic for the repository's gemm + softmax fusion. / 为仓库中的GEMM 与 softmax 融合提供可复用的声明与辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14
````cpp
/***************************************************************************************************
 * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
 * Copyright (C) 2025 Intel Corporation, All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
````
**EN:** This opening block carries the license banner and file-level description, framing the gemm + softmax fusion example before the executable code begins. It corresponds to block 1 of 87 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代GEMM 与 softmax 融合示例的背景。 它对应本文件顺序中的第 1/87 个代码块。

### Lines 15-28
````cpp
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `Neither`, `the`, `name`, `of` showing the main symbols being prepared or consumed here. It corresponds to block 2 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `Neither`、`the`、`name`、`of` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 2/87 个代码块。

### Lines 29-36
````cpp
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
/*!
  \file
  \brief The universal GEMM accommodates serial reductions, parallel reductions, batched strided, and
    batched array variants.
*/
````
**EN:** This opening block carries the license banner and file-level description, framing the gemm + softmax fusion example before the executable code begins. It corresponds to block 3 of 87 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代GEMM 与 softmax 融合示例的背景。 它对应本文件顺序中的第 3/87 个代码块。

### Lines 38-38
````cpp
#pragma once
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `pragma`, `once` showing the main symbols being prepared or consumed here. It corresponds to block 4 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `pragma`、`once` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 4/87 个代码块。

### Lines 40-46
````cpp
// common
#include "cutlass/cutlass.h"
#include "cutlass/device_kernel.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/detail/layout.hpp"
#include "cutlass/detail/mma.hpp"
#include "cutlass/cuda_host_adapter.hpp"
````
**EN:** This block pulls in dependencies required by the file, especially `cuda`, so the later gemm + softmax fusion code can use the needed APIs and data structures. It corresponds to block 5 of 87 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `cuda`，使后续GEMM 与 softmax 融合代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 5/87 个代码块。

### Lines 48-51
````cpp
#if !defined(__CUDACC_RTC__)
#include "cutlass/cluster_launch.hpp"
#include "cutlass/trace.h"
#endif // !defined(__CUDACC_RTC__)
````
**EN:** This block pulls in dependencies required by the file, especially `CUDA`, so the later gemm + softmax fusion code can use the needed APIs and data structures. It corresponds to block 6 of 87 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `CUDA`，使后续GEMM 与 softmax 融合代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 6/87 个代码块。

### Lines 53-54
````cpp
// 3.x
#include "cutlass/gemm/kernel/gemm_universal.hpp"
````
**EN:** This block pulls in dependencies required by the file, especially `x`, `include`, `cutlass`, `gemm`, so the later gemm + softmax fusion code can use the needed APIs and data structures. It corresponds to block 7 of 87 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `x`、`include`、`cutlass`、`gemm`，使后续GEMM 与 softmax 融合代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 7/87 个代码块。

### Lines 56-58
````cpp
#if defined(CUTLASS_ENABLE_SYCL)
#include "cutlass/util/sycl_event_manager.hpp"
#endif
````
**EN:** This block pulls in dependencies required by the file, especially `SYCL`, `sycl`, so the later gemm + softmax fusion code can use the needed APIs and data structures. It corresponds to block 8 of 87 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `SYCL`、`sycl`，使后续GEMM 与 softmax 融合代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 8/87 个代码块。

### Lines 60-60
````cpp
#include "softmax_finalize.hpp"
````
**EN:** This block pulls in dependencies required by the file, especially `softmax`, so the later gemm + softmax fusion code can use the needed APIs and data structures. It corresponds to block 9 of 87 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `softmax`，使后续GEMM 与 softmax 融合代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 9/87 个代码块。

### Lines 62-62
````cpp
////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 10 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 10/87 个代码块。

### Lines 64-64
````cpp
namespace cutlass::gemm::device {
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `namespace`, `cutlass::gemm::device` showing the main symbols being prepared or consumed here. It corresponds to block 11 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `namespace`、`cutlass::gemm::device` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 11/87 个代码块。

### Lines 66-66
````cpp
////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 12 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 12/87 个代码块。

### Lines 68-70
````cpp
////////////////////////////////////////////////////////////////////////////////
////////////////////////////// CUTLASS 3.x API /////////////////////////////////
////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `CUTLASS`, `x`, `API` showing the main symbols being prepared or consumed here. It corresponds to block 13 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `CUTLASS`、`x`、`API` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 13/87 个代码块。

### Lines 72-85
````cpp
template <class GemmKernel_>
class GemmSoftmaxAdapter
{
public:
  using GemmKernel = GemmKernel_;
  using TileShape = typename GemmKernel::TileShape;
  using ElementA = typename GemmKernel::ElementA;
  using ElementB = typename GemmKernel::ElementB;
  using ElementC = typename GemmKernel::ElementC;
  using ElementD = typename GemmKernel::ElementD;
  using ElementAccumulator = typename GemmKernel::ElementAccumulator;
  using DispatchPolicy = typename GemmKernel::DispatchPolicy;
  using CollectiveMainloop = typename GemmKernel::CollectiveMainloop;
  using CollectiveEpilogue = typename GemmKernel::CollectiveEpilogue;
````
**EN:** This block declares a type-level building block for the file, with `Epilogue`, `TileShape`, `Shape` indicating the configuration, traits, or storage policy used later. It corresponds to block 14 of 87 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `Epilogue`、`TileShape`、`Shape` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 14/87 个代码块。

### Lines 87-90
````cpp
  using SoftmaxFinalizeKernel = reduction::kernel::SoftmaxFinalize<
                                          ElementD, typename GemmKernel::StrideD,
                                          ElementAccumulator, typename GemmKernel::CollectiveEpilogue::StridePartials,
                                          ElementD, typename GemmKernel::StrideD>;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Epilogue` make the later gemm + softmax fusion code easier to assemble and read. It corresponds to block 15 of 87 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Epilogue` 这样的符号让后续GEMM 与 softmax 融合代码更容易组装和阅读。 它对应本文件顺序中的第 15/87 个代码块。

### Lines 92-97
````cpp
  // Map back to 2.x type as best as possible
  using LayoutA = gemm::detail::StrideToLayoutTagA_t<typename GemmKernel::StrideA>;
  using LayoutB = gemm::detail::StrideToLayoutTagB_t<typename GemmKernel::StrideB>;
  using LayoutC = gemm::detail::StrideToLayoutTagC_t<typename GemmKernel::StrideC>;
  using LayoutD = gemm::detail::StrideToLayoutTagC_t<typename GemmKernel::StrideD>;
  using LayoutPartials = gemm::detail::StrideToLayoutTagC_t<typename GemmKernel::StrideD>;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 16 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 16/87 个代码块。

### Lines 99-99
````cpp
  static bool const kEnableCudaHostAdapter = CUTLASS_ENABLE_CUDA_HOST_ADAPTER;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `CUDA` showing the main symbols being prepared or consumed here. It corresponds to block 17 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `CUDA` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 17/87 个代码块。

### Lines 101-104
````cpp
  static ComplexTransform const kTransformA = cute::is_same_v<typename GemmKernel::CollectiveMainloop::TransformA, cute::conjugate> ?
                                              ComplexTransform::kConjugate : ComplexTransform::kNone;
  static ComplexTransform const kTransformB = cute::is_same_v<typename GemmKernel::CollectiveMainloop::TransformB, cute::conjugate> ?
                                              ComplexTransform::kConjugate : ComplexTransform::kNone;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `cute` showing the main symbols being prepared or consumed here. It corresponds to block 18 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 18/87 个代码块。

### Lines 106-107
````cpp
  // Legacy: Assume MultiplyAdd only since we do not use this tag type in 3.0
  using MathOperator = cutlass::arch::OpMultiplyAdd;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `Legacy`, `Assume`, `MultiplyAdd`, `only` showing the main symbols being prepared or consumed here. It corresponds to block 19 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `Legacy`、`Assume`、`MultiplyAdd`、`only` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 19/87 个代码块。

### Lines 109-109
````cpp
  using OperatorClass = cutlass::detail::get_operator_class_t<typename CollectiveMainloop::TiledMma>;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `OperatorClass`, `cutlass::detail::get_operator_class_t<typename`, `CollectiveMainloop::TiledMma` make the later gemm + softmax fusion code easier to assemble and read. It corresponds to block 20 of 87 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `OperatorClass`、`cutlass::detail::get_operator_class_t<typename`、`CollectiveMainloop::TiledMma` 这样的符号让后续GEMM 与 softmax 融合代码更容易组装和阅读。 它对应本文件顺序中的第 20/87 个代码块。

### Lines 111-111
````cpp
  using ArchTag = typename GemmKernel::ArchTag;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `ArchTag`, `GemmKernel::ArchTag` make the later gemm + softmax fusion code easier to assemble and read. It corresponds to block 21 of 87 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `ArchTag`、`GemmKernel::ArchTag` 这样的符号让后续GEMM 与 softmax 融合代码更容易组装和阅读。 它对应本文件顺序中的第 21/87 个代码块。

### Lines 113-114
````cpp
  // NOTE: Assume identity swizzle for now
  using ThreadblockSwizzle = cutlass::gemm::threadblock::GemmIdentityThreadblockSwizzle<>;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `NOTE`, `Assume`, `identity`, `swizzle` showing the main symbols being prepared or consumed here. It corresponds to block 22 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `NOTE`、`Assume`、`identity`、`swizzle` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 22/87 个代码块。

### Lines 116-120
````cpp
  // Assume TiledMma's ShapeMNK is the same as 2.x's ThreadblockShape
  using ThreadblockShape = cutlass::gemm::GemmShape<
      cute::size<0>(TileShape{}),
      cute::size<1>(TileShape{}),
      cute::size<2>(TileShape{})>;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `TileShape`, `Shape`, `cute` showing the main symbols being prepared or consumed here. It corresponds to block 23 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `TileShape`、`Shape`、`cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 23/87 个代码块。

### Lines 122-125
````cpp
  using ClusterShape = cutlass::gemm::GemmShape<
      cute::size<0>(typename GemmKernel::DispatchPolicy::ClusterShape{}),
      cute::size<1>(typename GemmKernel::DispatchPolicy::ClusterShape{}),
      cute::size<2>(typename GemmKernel::DispatchPolicy::ClusterShape{})>;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `ClusterShape`, `Shape`, `cute` make the later gemm + softmax fusion code easier to assemble and read. It corresponds to block 24 of 87 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `ClusterShape`、`Shape`、`cute` 这样的符号让后续GEMM 与 softmax 融合代码更容易组装和阅读。 它对应本文件顺序中的第 24/87 个代码块。

### Lines 127-131
````cpp
  // Instruction shape is easy too, since we get that directly from our TiledMma's atom shape
  using InstructionShape = cutlass::gemm::GemmShape<
      cute::size<0>(typename CollectiveMainloop::TiledMma::AtomShape_MNK{}),
      cute::size<1>(typename CollectiveMainloop::TiledMma::AtomShape_MNK{}),
      cute::size<2>(typename CollectiveMainloop::TiledMma::AtomShape_MNK{})>;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `Shape`, `cute` showing the main symbols being prepared or consumed here. It corresponds to block 25 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `Shape`、`cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 25/87 个代码块。

### Lines 133-134
````cpp
  // Legacy: provide a correct warp count, but no reliable warp shape
  static int const kThreadCount = GemmKernel::MaxThreadsPerBlock;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `Legacy`, `provide`, `a`, `correct` showing the main symbols being prepared or consumed here. It corresponds to block 26 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `Legacy`、`provide`、`a`、`correct` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 26/87 个代码块。

### Lines 136-147
````cpp
  // Warp shape is not a primary API type in 3.x
  // But we can best approximate it by inspecting the TiledMma
  // For this, we make the assumption that we always have 4 warps along M, and rest along N, none along K
  // We also always round up the warp count to 4 if the tiled mma is smaller than 128 threads
  static constexpr int WarpsInMma = cute::max(4, CUTE_STATIC_V(cute::size(typename GemmKernel::TiledMma{})) / 32);
  static constexpr int WarpsInMmaM = 4;
  static constexpr int WarpsInMmaN = cute::ceil_div(WarpsInMma, WarpsInMmaM);
  using WarpCount = cutlass::gemm::GemmShape<WarpsInMmaM, WarpsInMmaN, 1>;
  using WarpShape = cutlass::gemm::GemmShape<
      CUTE_STATIC_V(cute::tile_size<0>(typename CollectiveMainloop::TiledMma{})) / WarpsInMmaM,
      CUTE_STATIC_V(cute::tile_size<1>(typename CollectiveMainloop::TiledMma{})) / WarpsInMmaN,
      CUTE_STATIC_V(cute::tile_size<2>(typename CollectiveMainloop::TiledMma{}))>;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `Shape`, `cute`, `CUTE` showing the main symbols being prepared or consumed here. It corresponds to block 27 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `Shape`、`cute`、`CUTE` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 27/87 个代码块。

### Lines 149-149
````cpp
  static int constexpr kStages = CollectiveMainloop::DispatchPolicy::Stages;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `static`, `constexpr`, `kStages`, `CollectiveMainloop::DispatchPolicy::Stages` showing the main symbols being prepared or consumed here. It corresponds to block 28 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `static`、`constexpr`、`kStages`、`CollectiveMainloop::DispatchPolicy::Stages` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 28/87 个代码块。

### Lines 151-159
````cpp
  // Inspect TiledCopy for A and B to compute the alignment size
  static int constexpr kAlignmentA = cutlass::detail::get_alignment_count_from_gmem_tiled_copy<
      typename CollectiveMainloop::GmemTiledCopyA, ElementA, typename CollectiveMainloop::TiledMma::ValTypeA>();
  static int constexpr kAlignmentB = cutlass::detail::get_alignment_count_from_gmem_tiled_copy<
      typename CollectiveMainloop::GmemTiledCopyB, ElementB, typename CollectiveMainloop::TiledMma::ValTypeB>();
  static int constexpr kAlignmentC = cutlass::detail::get_alignment_count_from_gmem_tiled_copy<
      typename CollectiveEpilogue::GmemTiledCopyC, ElementC>();
  static int constexpr kAlignmentD = cutlass::detail::get_alignment_count_from_gmem_tiled_copy<
      typename CollectiveEpilogue::GmemTiledCopyD, ElementD>();
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `Epilogue` showing the main symbols being prepared or consumed here. It corresponds to block 29 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `Epilogue` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 29/87 个代码块。

### Lines 161-161
````cpp
  using EpilogueOutputOp = typename CollectiveEpilogue::ThreadEpilogueOp;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Epilogue` make the later gemm + softmax fusion code easier to assemble and read. It corresponds to block 30 of 87 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Epilogue` 这样的符号让后续GEMM 与 softmax 融合代码更容易组装和阅读。 它对应本文件顺序中的第 30/87 个代码块。

### Lines 163-165
````cpp
  // Split-K preserves splits that are 128b aligned
  static int constexpr kSplitKAlignment = cute::max(
      128 / sizeof_bits<ElementA>::value, 128 / sizeof_bits<ElementB>::value);
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `cute` showing the main symbols being prepared or consumed here. It corresponds to block 31 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 31/87 个代码块。

### Lines 167-168
````cpp
  /// Argument structure: User API
  using Arguments = typename GemmKernel::Arguments;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `Argument`, `structure`, `User`, `API` showing the main symbols being prepared or consumed here. It corresponds to block 32 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `Argument`、`structure`、`User`、`API` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 32/87 个代码块。

### Lines 170-173
````cpp
  struct Params{
    typename GemmKernel::Params gemm_params;
    typename SoftmaxFinalizeKernel::Params softmax_params;
  };
````
**EN:** This block declares a type-level building block for the file, with `softmax` indicating the configuration, traits, or storage policy used later. It corresponds to block 33 of 87 in the file order.
**CN:** 这一段声明了文件中的类型级构件，其中 `softmax` 表明后续会使用的配置、traits 或存储策略。 它对应本文件顺序中的第 33/87 个代码块。

### Lines 175-175
````cpp
private:
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 34 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 34/87 个代码块。

### Lines 177-178
````cpp
  /// Kernel API parameters object
  Params params_;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `Kernel`, `API`, `parameters`, `object` showing the main symbols being prepared or consumed here. It corresponds to block 35 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `Kernel`、`API`、`parameters`、`object` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 35/87 个代码块。

### Lines 180-180
````cpp
public:
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 36 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 36/87 个代码块。

### Lines 182-185
````cpp
  /// Access the Params structure
  Params const& params() const {
    return params_;
  }
````
**EN:** This block finalizes a local computation or status path. The use of `Access`, `the`, `Params`, `structure` helps conclude the current stage cleanly before the next block. It corresponds to block 37 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `Access`、`the`、`Params`、`structure`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 37/87 个代码块。

### Lines 187-196
````cpp
  /// Determines whether the GEMM can execute the given problem.
  static Status
  can_implement(Arguments const& args) {
    if (GemmKernel::can_implement(args)) {
      return Status::kSuccess;
    }
    else {
      return Status::kInvalid;
    }
  }
````
**EN:** This block applies conditional control flow. It uses `cute` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 38 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cute` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 38/87 个代码块。

### Lines 198-204
````cpp
  /// Gets the workspace size
  static size_t
  get_workspace_size(Arguments const& args) {
    size_t workspace_bytes = 0;
    if (args.mode == GemmUniversalMode::kGemmSplitKParallel) {
      workspace_bytes += sizeof(int) * size_t(cute::size<0>(TileShape{})) * size_t(cute::size<1>(TileShape{}));
    }
````
**EN:** This block applies conditional control flow. It uses `GemmUniversal`, `TileShape`, `Shape`, `cute` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 39 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `GemmUniversal`、`TileShape`、`Shape`、`cute` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 39/87 个代码块。

### Lines 206-206
````cpp
    CUTLASS_TRACE_HOST("  workspace_bytes: " << workspace_bytes);
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `workspace` showing the main symbols being prepared or consumed here. It corresponds to block 40 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `workspace` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 40/87 个代码块。

### Lines 208-210
````cpp
    workspace_bytes += GemmKernel::get_workspace_size(args);
    return workspace_bytes;
  }
````
**EN:** This block finalizes a local computation or status path. The use of `workspace` helps conclude the current stage cleanly before the next block. It corresponds to block 41 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `workspace`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 41/87 个代码块。

### Lines 212-217
````cpp
  /// Computes the grid shape
  static dim3
  get_grid_shape(Arguments const& args, void* workspace = nullptr) {
    auto tmp_params = GemmKernel::to_underlying_arguments(args, workspace);
    return GemmKernel::get_grid_shape(tmp_params);
  }
````
**EN:** This block finalizes a local computation or status path. The use of `workspace` helps conclude the current stage cleanly before the next block. It corresponds to block 42 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `workspace`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 42/87 个代码块。

### Lines 219-223
````cpp
  /// Computes the grid shape
  static dim3
  get_grid_shape(Params const& params) {
    return GemmKernel::get_grid_shape(params.gemm_params);
  }
````
**EN:** This block finalizes a local computation or status path. The use of `Computes`, `the`, `grid`, `shape` helps conclude the current stage cleanly before the next block. It corresponds to block 43 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `Computes`、`the`、`grid`、`shape`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 43/87 个代码块。

### Lines 225-229
````cpp
  /// Computes the maximum number of active blocks per multiprocessor
  static int maximum_active_blocks(int /* smem_capacity */ = -1) {
    CUTLASS_TRACE_HOST("GemmUniversal::maximum_active_blocks()");
    int max_active_blocks = -1;
    int smem_size = GemmKernel::SharedStorageSize;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `GemmUniversal` showing the main symbols being prepared or consumed here. It corresponds to block 44 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `GemmUniversal` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 44/87 个代码块。

### Lines 231-244
````cpp
    // first, account for dynamic smem capacity if needed
    cudaError_t result;
    if (smem_size >= (48 << 10)) {
      CUTLASS_TRACE_HOST("  Setting smem size to " << smem_size);
      result = cudaFuncSetAttribute(
          device_kernel<GemmKernel>,
          cudaFuncAttributeMaxDynamicSharedMemorySize,
          smem_size);
      if (cudaSuccess != result) {
        result = cudaGetLastError(); // to clear the error bit
        CUTLASS_TRACE_HOST(
          "  cudaFuncSetAttribute() returned error: "
          << cudaGetErrorString(result));
        return -1;
````
**EN:** This block applies conditional control flow. It uses `cuda` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 45 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cuda` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 45/87 个代码块。

### Lines 245-246
````cpp
      }
    }
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 46 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 46/87 个代码块。

### Lines 248-253
````cpp
    // query occupancy after setting smem size
    result = cudaOccupancyMaxActiveBlocksPerMultiprocessor(
        &max_active_blocks,
        device_kernel<GemmKernel>,
        GemmKernel::MaxThreadsPerBlock,
        smem_size);
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `cuda` showing the main symbols being prepared or consumed here. It corresponds to block 47 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `cuda` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 47/87 个代码块。

### Lines 255-261
````cpp
    if (cudaSuccess != result) {
      result = cudaGetLastError(); // to clear the error bit
      CUTLASS_TRACE_HOST(
        "  cudaOccupancyMaxActiveBlocksPerMultiprocessor() returned error: "
        << cudaGetErrorString(result));
      return -1;
    }
````
**EN:** This block applies conditional control flow. It uses `cuda` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 48 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cuda` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 48/87 个代码块。

### Lines 263-265
````cpp
    CUTLASS_TRACE_HOST("  max_active_blocks: " << max_active_blocks);
    return max_active_blocks;
  }
````
**EN:** This block finalizes a local computation or status path. The use of `CUTLASS_TRACE_HOST`, `max_active_blocks` helps conclude the current stage cleanly before the next block. It corresponds to block 49 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `CUTLASS_TRACE_HOST`、`max_active_blocks`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 49/87 个代码块。

### Lines 267-279
````cpp
  void initialize_softmax_params(Arguments const& args, typename SoftmaxFinalizeKernel::Arguments& softmax_args){
    softmax_args.M = get<0>(args.problem_shape);
    softmax_args.dataN = get<1>(args.problem_shape);
    softmax_args.partialN = cute::ceil_div(get<1>(args.problem_shape), cute::shape<1>(TileShape{}));
    softmax_args.batch_count = get<3>(args.problem_shape);
    softmax_args.dInput = args.epilogue.dD;
    softmax_args.dPartial = args.epilogue.dPartials;
    softmax_args.dOutput = args.epilogue.dD;
    softmax_args.ptr_in = args.epilogue.ptr_D;
    softmax_args.ptr_partial_max = args.epilogue.ptr_max;
    softmax_args.ptr_partial_sum = args.epilogue.ptr_sum;
    softmax_args.ptr_out = args.epilogue.ptr_D;
  }
````
**EN:** This block introduces executable logic through a function or method. Here, `softmax`, `epilogue`, `TileShape`, `Shape` drive a concrete step in the file's gemm + softmax fusion flow. It corresponds to block 50 of 87 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `softmax`、`epilogue`、`TileShape`、`Shape` 推动了本文件GEMM 与 softmax 融合流程中的一个具体步骤。 它对应本文件顺序中的第 50/87 个代码块。

### Lines 281-287
````cpp
  /// Initializes GEMM state from arguments.
  Status
  initialize(
    Arguments const& args,
    void* workspace = nullptr,
    cudaStream_t stream = nullptr,
    CudaHostAdapter* cuda_adapter = nullptr) {
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `cuda`, `workspace` showing the main symbols being prepared or consumed here. It corresponds to block 51 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `cuda`、`workspace` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 51/87 个代码块。

### Lines 289-290
````cpp
    CUTLASS_TRACE_HOST("GemmUniversal::initialize() - workspace "
      << workspace << ", stream: " << (stream ? "non-null" : "null"));
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `GemmUniversal`, `workspace` showing the main symbols being prepared or consumed here. It corresponds to block 52 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `GemmUniversal`、`workspace` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 52/87 个代码块。

### Lines 292-299
````cpp
    // Initialize the workspace
    Status status = GemmKernel::initialize_workspace(args, workspace, stream, cuda_adapter);
    if (status != Status::kSuccess) {
      return status;
    }
    // Initialize the Params structure
    params_.gemm_params = GemmKernel::to_underlying_arguments(args, workspace);
    initialize_softmax_params(args, params_.softmax_params.args);
````
**EN:** This block applies conditional control flow. It uses `softmax`, `cuda`, `workspace` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 53 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `softmax`、`cuda`、`workspace` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 53/87 个代码块。

### Lines 301-310
````cpp
    // Don't set the function attributes - require the CudaHostAdapter to set it.
    if constexpr (kEnableCudaHostAdapter) {
      CUTLASS_ASSERT(cuda_adapter);
      return Status::kSuccess;
    }
    else {
      //
      // Account for dynamic smem capacity if needed
      //
      int smem_size = GemmKernel::SharedStorageSize;
````
**EN:** This block applies conditional control flow. It uses `cuda` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 54 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cuda` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 54/87 个代码块。

### Lines 312-312
````cpp
      CUTLASS_ASSERT(cuda_adapter == nullptr);
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `cuda` showing the main symbols being prepared or consumed here. It corresponds to block 55 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `cuda` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 55/87 个代码块。

### Lines 314-327
````cpp
#if !defined(CUTLASS_ENABLE_SYCL)
      if (smem_size >= (48 << 10)) {
        CUTLASS_TRACE_HOST("  Setting smem size to " << smem_size);
        cudaError_t result = cudaFuncSetAttribute(
            device_kernel<GemmKernel>,
            cudaFuncAttributeMaxDynamicSharedMemorySize,
            smem_size);
        if (cudaSuccess != result) {
          result = cudaGetLastError(); // to clear the error bit
          CUTLASS_TRACE_HOST("  cudaFuncSetAttribute() returned error: " << cudaGetErrorString(result));
          return Status::kErrorInternal;
        }
      }
#endif
````
**EN:** This block applies conditional control flow. It uses `SYCL`, `cuda` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 56 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `SYCL`、`cuda` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 56/87 个代码块。

### Lines 328-330
````cpp
    }
    return Status::kSuccess;
  }
````
**EN:** This block finalizes a local computation or status path. The use of `Status::kSuccess` helps conclude the current stage cleanly before the next block. It corresponds to block 57 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `Status::kSuccess`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 57/87 个代码块。

### Lines 332-335
````cpp
  /// Update API is preserved in 3.0, but does not guarantee a lightweight update of params.
  Status
  update(Arguments const& args, void* workspace = nullptr) {
    CUTLASS_TRACE_HOST("GemmUniversal()::update() - workspace: " << workspace);
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `GemmUniversal`, `workspace` showing the main symbols being prepared or consumed here. It corresponds to block 58 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `GemmUniversal`、`workspace` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 58/87 个代码块。

### Lines 337-340
````cpp
    size_t workspace_bytes = get_workspace_size(args);
    if (workspace_bytes > 0 && nullptr == workspace) {
      return Status::kErrorWorkspaceNull;
    }
````
**EN:** This block introduces executable logic through a function or method. Here, `workspace` drive a concrete step in the file's gemm + softmax fusion flow. It corresponds to block 59 of 87 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `workspace` 推动了本文件GEMM 与 softmax 融合流程中的一个具体步骤。 它对应本文件顺序中的第 59/87 个代码块。

### Lines 342-345
````cpp
    params_.gemm_params = GemmKernel::to_underlying_arguments(args, workspace);
    initialize_softmax_params(args, params_.softmax_params.args);
    return Status::kSuccess;
  }
````
**EN:** This block finalizes a local computation or status path. The use of `softmax`, `workspace` helps conclude the current stage cleanly before the next block. It corresponds to block 60 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `softmax`、`workspace`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 60/87 个代码块。

### Lines 347-360
````cpp
  /// Primary run() entry point API that is static allowing users to create and manage their own params.
  /// Supplied params struct must be construct by calling GemmKernel::to_underlying_arguments()
  static Status
  run(Params& params,
      cudaStream_t stream = nullptr,
      CudaHostAdapter *cuda_adapter = nullptr,
      bool launch_with_pdl = false) {
    CUTLASS_TRACE_HOST("GemmUniversal::run()");
    dim3 const block = GemmKernel::get_block_shape();
    dim3 const grid = get_grid_shape(params);
    dim3 const block_finalize(
      NumThreadsPerWarp,
      std::min(MaxNumThreadsPerBlock / NumThreadsPerWarp, params.softmax_params.args.M),
      1
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `GemmUniversal`, `softmax`, `cuda` showing the main symbols being prepared or consumed here. It corresponds to block 61 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `GemmUniversal`、`softmax`、`cuda` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 61/87 个代码块。

### Lines 361-366
````cpp
    );
    dim3 const grid_finalize(
      cute::ceil_div(params.softmax_params.args.M, block_finalize.x),
      params.softmax_params.args.batch_count,
      1
    );
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `softmax`, `cute` showing the main symbols being prepared or consumed here. It corresponds to block 62 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `softmax`、`cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 62/87 个代码块。

### Lines 368-370
````cpp
    // configure smem size and carveout
    int smem_size = GemmKernel::SharedStorageSize;
    int smem_size_finalize = SoftmaxFinalizeKernel::SharedStorageSize;
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `configure`, `smem`, `size`, `and` showing the main symbols being prepared or consumed here. It corresponds to block 63 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `configure`、`smem`、`size`、`and` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 63/87 个代码块。

### Lines 372-383
````cpp
    Status launch_result{ Status::kSuccess };
    // Use extended launch API only for mainloops that use it
    if constexpr (GemmKernel::ArchTag::kMinComputeCapability >= 90) {
#if !defined(CUTLASS_ENABLE_SYCL)
      constexpr bool is_static_1x1x1 = cute::is_static_v<typename GemmKernel::DispatchPolicy::ClusterShape> and
                                       cute::size(typename GemmKernel::DispatchPolicy::ClusterShape{}) == 1;
      dim3 cluster(cute::size<0>(typename GemmKernel::DispatchPolicy::ClusterShape{}),
                   cute::size<1>(typename GemmKernel::DispatchPolicy::ClusterShape{}),
                   cute::size<2>(typename GemmKernel::DispatchPolicy::ClusterShape{}));
      dim3 cluster_finalize(1,1,1);
      void* kernel_params[] = {&params.gemm_params};
      void* kernel_params_finalize[] = {&params.softmax_params};
````
**EN:** This block applies conditional control flow. It uses `softmax`, `mainloop`, `ClusterShape`, `Shape` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 64 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `softmax`、`mainloop`、`ClusterShape`、`Shape` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 64/87 个代码块。

### Lines 385-390
````cpp
      if constexpr (kEnableCudaHostAdapter) {
        //
        // Use the cuda host adapter
        //
        CUTLASS_ASSERT(cuda_adapter);
        if (cuda_adapter) {
````
**EN:** This block applies conditional control flow. It uses `cuda` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 65 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cuda` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 65/87 个代码块。

### Lines 392-405
````cpp
          if (launch_with_pdl) {
            CUTLASS_TRACE_HOST(
              "GemmUniversal::run() does not support launching with PDL and a custom cuda adapter.");
            return Status::kErrorInternal;
          }
          launch_result = cuda_adapter->launch(grid,
                                               cluster,
                                               block,
                                               smem_size,
                                               stream,
                                               kernel_params,
                                               0);
          launch_result = cuda_adapter->launch(grid_finalize,
                                               cluster_finalize,
````
**EN:** This block applies conditional control flow. It uses `GemmUniversal`, `cuda` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 66 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `GemmUniversal`、`cuda` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 66/87 个代码块。

### Lines 406-419
````cpp
                                               block_finalize,
                                               smem_size_finalize,
                                               stream,
                                               kernel_params_finalize,
                                               1);
        }
        else {
          return Status::kErrorInternal;
        }
      }
      else {
        CUTLASS_ASSERT(cuda_adapter == nullptr);
        void const* kernel = (void const*) device_kernel<GemmKernel>;
        void const* kernel_finalize = (void const*) device_kernel<SoftmaxFinalizeKernel>;
````
**EN:** This block finalizes a local computation or status path. The use of `cuda` helps conclude the current stage cleanly before the next block. It corresponds to block 67 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `cuda`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 67/87 个代码块。

### Lines 420-433
````cpp
        if constexpr (GemmKernel::ArchTag::kMinComputeCapability == 90) {
          if (is_static_1x1x1 && not launch_with_pdl) {
            device_kernel<GemmKernel><<<grid, block, smem_size, stream>>>(params.gemm_params);
            device_kernel<SoftmaxFinalizeKernel><<<grid_finalize, block_finalize, smem_size_finalize, stream>>>(params.softmax_params);
          }
          else {
            launch_result = ClusterLauncher::launch(
              grid, cluster, block, smem_size, stream, kernel, kernel_params, launch_with_pdl);
            launch_result = ClusterLauncher::launch(
              grid_finalize, cluster_finalize, block_finalize, smem_size_finalize, stream, kernel_finalize, kernel_params, launch_with_pdl);
          }
        }
      }
#endif
````
**EN:** This block applies conditional control flow. It uses `softmax` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 68 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `softmax` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 68/87 个代码块。

### Lines 434-441
````cpp
    }
    else {
      launch_result = Status::kSuccess;
      if constexpr (kEnableCudaHostAdapter) {
        CUTLASS_ASSERT(cuda_adapter);
        if (cuda_adapter) {
          void* kernel_params[] = {&params.gemm_params};
          void* kernel_params_finalize[] = {&params.softmax_params};
````
**EN:** This block applies conditional control flow. It uses `softmax`, `cuda` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 69 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `softmax`、`cuda` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 69/87 个代码块。

### Lines 443-448
````cpp
          launch_result = cuda_adapter->launch(
            grid, block, smem_size, stream, kernel_params, 0
          );
          launch_result = cuda_adapter->launch(
            grid_finalize, block_finalize, smem_size_finalize, stream, kernel_params_finalize, 1
          );
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `cuda` showing the main symbols being prepared or consumed here. It corresponds to block 70 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `cuda` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 70/87 个代码块。

### Lines 450-463
````cpp
        }
        else {
          return Status::kErrorInternal;
        }
      }
      else {
        CUTLASS_ASSERT(cuda_adapter == nullptr);
#if defined(CUTLASS_ENABLE_SYCL)
        const compat::dim3 sycl_grid(grid.x, grid.y, grid.z);
        const compat::dim3 sycl_block(block.x, block.y, block.z);
#if defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)
        sycl::ext::oneapi::experimental::properties smem_prop{
          sycl::ext::oneapi::experimental::work_group_scratch_size(smem_size)
        };
````
**EN:** This block finalizes a local computation or status path. The use of `SYCL`, `sycl`, `cuda` helps conclude the current stage cleanly before the next block. It corresponds to block 71 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `SYCL`、`sycl`、`cuda`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 71/87 个代码块。

### Lines 464-477
````cpp
        compat::experimental::launch_properties launch_props{smem_prop};
#if defined(CUTLASS_SYCL_PROFILING_ENABLED)
        auto event = compat::experimental::launch<device_kernel<GemmKernel>, GemmKernel>(compat::experimental::launch_policy{
          sycl_grid,
          sycl_block,
          launch_props
#if defined(SYCL_INTEL_TARGET)
          , compat::experimental::kernel_properties{sycl_exp::sub_group_size<DispatchPolicy::SubgroupSize>}
#endif // defined(SYCL_INTEL_TARGET)
        }, params.gemm_params);
#else
        compat::experimental::launch<device_kernel<GemmKernel>, GemmKernel, false>(compat::experimental::launch_policy{
          sycl_grid,
          sycl_block,
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `SYCL`, `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 72 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `SYCL`、`sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 72/87 个代码块。

### Lines 478-483
````cpp
          launch_props
#if defined(SYCL_INTEL_TARGET)
          , compat::experimental::kernel_properties{sycl_exp::sub_group_size<DispatchPolicy::SubgroupSize>}
#endif // defined(SYCL_INTEL_TARGET)
        }, params.gemm_params);
#endif
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `SYCL`, `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 73 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `SYCL`、`sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 73/87 个代码块。

### Lines 485-498
````cpp
        compat::experimental::launch_properties kernel_launch_props_finalize{
          sycl::ext::oneapi::experimental::work_group_scratch_size(smem_size_finalize)
        };
        const compat::dim3 sycl_grid_finalize(grid_finalize.x, grid_finalize.y, grid_finalize.z);
        const compat::dim3 sycl_block_finalize(block_finalize.x, block_finalize.y, block_finalize.z);
#if defined(CUTLASS_SYCL_PROFILING_ENABLED)
        auto event_finalize = compat::experimental::launch<device_kernel<SoftmaxFinalizeKernel>, SoftmaxFinalizeKernel>(compat::experimental::launch_policy{
            sycl_grid_finalize,
            sycl_block_finalize,
            kernel_launch_props_finalize,
          }, params.softmax_params);
        EventManager::getInstance().addEvent(event_finalize);
#else
        compat::experimental::launch<device_kernel<SoftmaxFinalizeKernel>, SoftmaxFinalizeKernel, false>(compat::experimental::launch_policy{
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `softmax`, `SYCL`, `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 74 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `softmax`、`SYCL`、`sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 74/87 个代码块。

### Lines 499-512
````cpp
            sycl_grid_finalize,
            sycl_block_finalize,
            kernel_launch_props_finalize,
          }, params.softmax_params);
#endif
#else
        using namespace compat::experimental;
#if defined(CUTLASS_SYCL_PROFILING_ENABLED)
        auto event = launch<device_kernel<GemmKernel>>(launch_policy{
          sycl_grid, sycl_block, local_mem_size{static_cast<std::size_t>(smem_size)}
#if defined (SYCL_INTEL_TARGET)
          , kernel_properties{sycl_exp::sub_group_size<DispatchPolicy::SubgroupSize>}
#endif
        }, params.gemm_params);
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `softmax`, `SYCL`, `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 75 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `softmax`、`SYCL`、`sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 75/87 个代码块。

### Lines 513-526
````cpp
#else
        launch<device_kernel<GemmKernel>, sycl::detail::auto_name, false>(launch_policy{
          sycl_grid, sycl_block, local_mem_size{static_cast<std::size_t>(smem_size)}
#if defined (SYCL_INTEL_TARGET)
          , kernel_properties{sycl_exp::sub_group_size<DispatchPolicy::SubgroupSize>}
#endif
        }, params.gemm_params);
#endif
        const auto sycl_block_finalize = compat::dim3(block_finalize.x, block_finalize.y, block_finalize.z);
        const auto sycl_grid_finalize = compat::dim3(grid_finalize.x, grid_finalize.y, grid_finalize.z);
#if defined(CUTLASS_SYCL_PROFILING_ENABLED)
        auto event2 = launch<device_kernel<SoftmaxFinalizeKernel>>(launch_policy{
          sycl_grid_finalize, sycl_block_finalize, local_mem_size{static_cast<std::size_t>(smem_size_finalize)}},
          params.softmax_params);
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `softmax`, `SYCL`, `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 76 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `softmax`、`SYCL`、`sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 76/87 个代码块。

### Lines 527-539
````cpp
        EventManager::getInstance().addEvent(event2);
#else
        launch<device_kernel<SoftmaxFinalizeKernel>, sycl::detail::auto_name, false>(launch_policy{
          sycl_grid_finalize, sycl_block_finalize, local_mem_size{static_cast<std::size_t>(smem_size_finalize)}},
          params.softmax_params);
#endif
#endif // defined(SYCL_EXT_ONEAPI_WORK_GROUP_SCRATCH_MEMORY)
#else
        device_kernel<GemmKernel><<<grid, block, smem_size, stream>>>(params.gemm_params);
        device_kernel<SoftmaxFinalizeKernel><<<grid_finalize, block_finalize, smem_size_finalize, stream>>>(params.softmax_params);
#endif
      }
    }
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `softmax`, `SYCL`, `sycl` showing the main symbols being prepared or consumed here. It corresponds to block 77 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `softmax`、`SYCL`、`sycl` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 77/87 个代码块。

### Lines 541-549
````cpp
    cudaError_t result = cudaGetLastError();
    if (cudaSuccess == result && Status::kSuccess == launch_result) {
      return Status::kSuccess;
    }
    else {
      CUTLASS_TRACE_HOST("  Kernel launch failed. Reason: " << result);
      return Status::kErrorInternal;
    }
  }
````
**EN:** This block applies conditional control flow. It uses `cuda` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 78 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cuda` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 78/87 个代码块。

### Lines 551-553
````cpp
  //
  // Non-static launch overloads that first create and set the internal params struct of this kernel handle.
  //
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `Non`, `static`, `launch`, `overloads` showing the main symbols being prepared or consumed here. It corresponds to block 79 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `Non`、`static`、`launch`、`overloads` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 79/87 个代码块。

### Lines 555-564
````cpp
  /// Launches the kernel after first constructing Params internal state from supplied arguments.
  Status
  run(
    Arguments const& args,
    void* workspace = nullptr,
    cudaStream_t stream = nullptr,
    CudaHostAdapter *cuda_adapter = nullptr,
    bool launch_with_pdl = false
  ) {
    Status status = initialize(args, workspace, stream, cuda_adapter);
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `cuda`, `workspace` showing the main symbols being prepared or consumed here. It corresponds to block 80 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `cuda`、`workspace` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 80/87 个代码块。

### Lines 566-570
````cpp
    if (Status::kSuccess == status) {
      status = run(params_, stream, cuda_adapter, launch_with_pdl);
    }
    return status;
  }
````
**EN:** This block applies conditional control flow. It uses `cuda` to select a path, validate assumptions, or handle special cases in the gemm + softmax fusion implementation. It corresponds to block 81 of 87 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cuda` 在GEMM 与 softmax 融合实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 81/87 个代码块。

### Lines 572-581
````cpp
  /// Launches the kernel after first constructing Params internal state from supplied arguments.
  Status
  operator()(
    Arguments const& args,
    void* workspace = nullptr,
    cudaStream_t stream = nullptr,
    CudaHostAdapter *cuda_adapter = nullptr,
    bool launch_with_pdl = false) {
    return run(args, workspace, stream, cuda_adapter, launch_with_pdl);
  }
````
**EN:** This block finalizes a local computation or status path. The use of `cuda`, `workspace` helps conclude the current stage cleanly before the next block. It corresponds to block 82 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `cuda`、`workspace`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 82/87 个代码块。

### Lines 583-590
````cpp
  /// Overload that allows a user to re-launch the same kernel without updating internal params struct.
  Status
  run(
    cudaStream_t stream = nullptr,
    CudaHostAdapter *cuda_adapter = nullptr,
    bool launch_with_pdl = false) {
    return run(params_, stream, cuda_adapter, launch_with_pdl);
  }
````
**EN:** This block finalizes a local computation or status path. The use of `cuda` helps conclude the current stage cleanly before the next block. It corresponds to block 83 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `cuda`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 83/87 个代码块。

### Lines 592-597
````cpp
  /// Overload that allows a user to re-launch the same kernel without updating internal params struct.
  Status
  operator()(cudaStream_t stream = nullptr, CudaHostAdapter *cuda_adapter = nullptr, bool launch_with_pdl = false) {
    return run(params_, stream, cuda_adapter, launch_with_pdl);
  }
};
````
**EN:** This block finalizes a local computation or status path. The use of `cuda` helps conclude the current stage cleanly before the next block. It corresponds to block 84 of 87 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `cuda`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 84/87 个代码块。

### Lines 599-599
````cpp
////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 85 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 85/87 个代码块。

### Lines 601-601
````cpp
} // namespace cutlass::gemm::device
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with `namespace`, `cutlass::gemm::device` showing the main symbols being prepared or consumed here. It corresponds to block 86 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 `namespace`、`cutlass::gemm::device` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 86/87 个代码块。

### Lines 603-603
````cpp
////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's gemm + softmax fusion setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 87 of 87 in the file order.
**CN:** 这一段继续推进本文件的GEMM 与 softmax 融合初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 87/87 个代码块。

## Key Concepts / 关键概念
- **EN:** Template-based GEMM composition with CUTLASS collectives, adapters, or visitors.
  **CN:** 使用 CUTLASS collective、adapter 或 visitor 进行模板化 GEMM 组合。
- **EN:** Post-processing is fused into the compute path through epilogue logic.
  **CN:** 后处理逻辑通过 epilogue 融合进主计算路径。
- **EN:** SYCL objects and queues act as the execution interface for heterogeneous kernels.
  **CN:** SYCL 对象与队列构成异构内核的执行接口。
- **EN:** Header-only abstractions centralize reusable declarations and launch helpers.
  **CN:** 头文件抽象把可复用声明与启动辅助逻辑集中起来。
- **EN:** The file mixes compile-time configuration with runtime problem sizes or arguments.
  **CN:** 该文件把编译期配置与运行期问题规模或参数结合起来。

## Dependencies / 依赖关系
- **Direct dependencies / 直接依赖:** `cutlass/cutlass.h`, `cutlass/device_kernel.h`, `cutlass/gemm/gemm.h`, `cutlass/detail/layout.hpp`, `cutlass/detail/mma.hpp`, `cutlass/cuda_host_adapter.hpp`, `cutlass/cluster_launch.hpp`, `cutlass/trace.h`, `cutlass/gemm/kernel/gemm_universal.hpp`, `cutlass/util/sycl_event_manager.hpp`, `softmax_finalize.hpp`
- **Runtime expectations / 运行时依赖:** SYCL runtime / SYCL 运行时; CUDA runtime / CUDA 运行时; CUTLASS/CUTE templates / CUTLASS/CUTE 模板
- **Build/context note / 构建说明:** The file relies on surrounding repository infrastructure, compile flags, and example utilities. / 该文件依赖仓库周边基础设施、编译选项以及示例辅助代码。
