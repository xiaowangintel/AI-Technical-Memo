# 05_bmg_gemm_with_topk_and_softmax.cpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `examples/05_bmg_gemm_with_epilogues/05_bmg_gemm_with_topk_and_softmax.cpp`
- **Purpose / 目的:** Demonstrates and validates the repository's softmax-related computation implementation. / 演示并验证仓库中的softmax 相关计算实现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14
````cpp
/***************************************************************************************************
 * Copyright (C) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
 * Copyright (C) 2025 - 2026 Intel Corporation, All rights reserved.
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
**EN:** This opening block carries the license banner and file-level description, framing the softmax-related computation example before the executable code begins. It corresponds to block 1 of 98 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代softmax 相关计算示例的背景。 它对应本文件顺序中的第 1/98 个代码块。

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
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Neither`, `the`, `name`, `of` showing the main symbols being prepared or consumed here. It corresponds to block 2 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Neither`、`the`、`name`、`of` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 2/98 个代码块。

### Lines 29-31
````cpp
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `OF`, `THIS`, `SOFTWARE`, `EVEN` showing the main symbols being prepared or consumed here. It corresponds to block 3 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `OF`、`THIS`、`SOFTWARE`、`EVEN` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 3/98 个代码块。

### Lines 33-34
````cpp
/*! \file
    \brief  BMG GEMM + Top-K + Softmax fusion
````
**EN:** This opening block carries the license banner and file-level description, framing the softmax-related computation example before the executable code begins. It corresponds to block 4 of 98 in the file order.
**CN:** 这一开头代码块给出许可证声明和文件级说明，在真正代码开始前先交代softmax 相关计算示例的背景。 它对应本文件顺序中的第 4/98 个代码块。

### Lines 36-37
````cpp
    This example illustrates how to use the LinCombTopKSoftmaxCol EVT node to fuse
    Top-K and Softmax into the GEMM epilogue, with certain assumptions made.
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `epilogue` showing the main symbols being prepared or consumed here. It corresponds to block 5 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `epilogue` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 5/98 个代码块。

### Lines 39-44
````cpp
    Those assumptions are as:
      1. Fusion is over the N dimension.
      2. Top-K is either 2 or 4 elements, and the value is static (meaning two kernels have to be
         compiled to support both.)
      3. The GEMM tile shape along N is greater than or equal to problem size
         along N.
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Those`, `assumptions`, `are`, `as` showing the main symbols being prepared or consumed here. It corresponds to block 6 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Those`、`assumptions`、`are`、`as` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 6/98 个代码块。

### Lines 47-48
````cpp
    The example runs the fused GEMM kernel, along with a standard unfused host reference, and
    manually performs Top-K and softmax, and compares the error between tensors.
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `softmax`, `reference` showing the main symbols being prepared or consumed here. It corresponds to block 7 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `softmax`、`reference` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 7/98 个代码块。

### Lines 50-52
````cpp
    Note that some numerical error (smaller than 1e-5) is to be expected, but this is true
    in most efficient reduction kernels, because floating point addition is not necessarily
    associative.
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Note`, `that`, `some`, `numerical` showing the main symbols being prepared or consumed here. It corresponds to block 8 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Note`、`that`、`some`、`numerical` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 8/98 个代码块。

### Lines 54-54
````cpp
    To build & run this example (from your build dir):
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `To`, `build`, `run`, `this` showing the main symbols being prepared or consumed here. It corresponds to block 9 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `To`、`build`、`run`、`this` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 9/98 个代码块。

### Lines 56-57
````cpp
      $ ninja 05_bmg_gemm_with_topk_and_softmax
      $ ./examples/sycl/05_bmg_gemm_with_epilogues/05_bmg_gemm_with_topk_and_softmax
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `softmax`, `epilogue`, `sycl`, `topk` showing the main symbols being prepared or consumed here. It corresponds to block 10 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `softmax`、`epilogue`、`sycl`、`topk` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 10/98 个代码块。

### Lines 59-67
````cpp
    Call with `--help` for information about available options
*/
#include "cutlass/epilogue/collective/default_epilogue.hpp"
#include "cutlass/epilogue/collective/xe_epilogue.hpp"
#include "cutlass/epilogue/fusion/xe_callbacks.hpp"
#include "cutlass/gemm/device/gemm_universal.h"
#include "cutlass/gemm/device/gemm_universal_adapter.h"
#include "cutlass/gemm/collective/collective_mma.hpp"
#include "cutlass/util/GPU_Clock.hpp"
````
**EN:** This block pulls in dependencies required by the file, especially `epilogue`, so the later softmax-related computation code can use the needed APIs and data structures. It corresponds to block 11 of 98 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `epilogue`，使后续softmax 相关计算代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 11/98 个代码块。

### Lines 69-70
````cpp
#include <cute/tensor.hpp>
#include <random>
````
**EN:** This block pulls in dependencies required by the file, especially `cute`, so the later softmax-related computation code can use the needed APIs and data structures. It corresponds to block 12 of 98 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `cute`，使后续softmax 相关计算代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 12/98 个代码块。

### Lines 72-85
````cpp
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/command_line.h"
#include "cutlass/util/device_memory.h"
#include "cutlass/util/packed_stride.hpp"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/reference/device/gemm_complex.h"
#include "cutlass/util/reference/device/tensor_compare.h"
#include "cutlass/util/reference/host/error_metrics.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/host/gett.hpp"
#include "helper.h"
````
**EN:** This block pulls in dependencies required by the file, especially `reference`, so the later softmax-related computation code can use the needed APIs and data structures. It corresponds to block 13 of 98 in the file order.
**CN:** 这一段引入本文件所需的依赖，尤其是 `reference`，使后续softmax 相关计算代码可以使用相应 API 和数据结构。 它对应本文件顺序中的第 13/98 个代码块。

### Lines 88-88
````cpp
using namespace cute;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `cute` make the later softmax-related computation code easier to assemble and read. It corresponds to block 14 of 98 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `cute` 这样的符号让后续softmax 相关计算代码更容易组装和阅读。 它对应本文件顺序中的第 14/98 个代码块。

### Lines 90-91
````cpp
static constexpr int TopK = 2;
static constexpr bool EnableTopKSoftmax = TopK > 1;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `static`, `constexpr`, `TopK`, `bool` showing the main symbols being prepared or consumed here. It corresponds to block 15 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `static`、`constexpr`、`TopK`、`bool` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 15/98 个代码块。

### Lines 93-95
````cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
/// GEMM kernel configurations
/////////////////////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `GEMM`, `kernel`, `configurations` showing the main symbols being prepared or consumed here. It corresponds to block 16 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `GEMM`、`kernel`、`configurations` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 16/98 个代码块。

### Lines 97-100
````cpp
// A matrix configuration
using         ElementA    = bfloat16_t;                                // Element type for A matrix operand
using         LayoutA     = cutlass::layout::RowMajor;                      // Layout type for A matrix operand
constexpr int AlignmentA  = 128 / cutlass::sizeof_bits<ElementA>::value;    // Memory access granularity/alignment of A matrix in units of elements (up to 16 bytes)
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 17 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 17/98 个代码块。

### Lines 102-105
````cpp
// B matrix configuration
using         ElementB    = bfloat16_t;                                // Element type for B matrix operand
using         LayoutB     = cutlass::layout::RowMajor;                   // Layout type for B matrix operand
constexpr int AlignmentB  = 128 / cutlass::sizeof_bits<ElementB>::value;    // Memory access granularity/alignment of B matrix in units of elements (up to 16 bytes)
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 18 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 18/98 个代码块。

### Lines 107-110
````cpp
// C matrix configuration
using         ElementC    = float; //TODO void???
using         LayoutC     = cutlass::layout::RowMajor;
constexpr int AlignmentC  = 1;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 19 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 19/98 个代码块。

### Lines 112-115
````cpp
// D matrix configuration
using         ElementD    = float;                                // Element type for C and D matrix operands
using         LayoutD     = cutlass::layout::RowMajor;                      // Layout type for output
constexpr int AlignmentD  = 128 / cutlass::sizeof_bits<ElementD>::value;    // Memory access granularity/alignment of output in units of elements (up to 16 bytes)
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Layout` showing the main symbols being prepared or consumed here. It corresponds to block 20 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Layout` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 20/98 个代码块。

### Lines 117-123
````cpp
// Core kernel configurations
using ElementAccumulator  = float;                                          // Element type for internal accumulation
using ElementCompute      = float;                                          // Element type for epilogue computation
using ArchTag             = cutlass::arch::IntelXe;                            // Tag indicating the minimum SM that supports the intended feature
using OperatorClass       = cutlass::arch::OpClassTensorOp;                 // Operator class tag
using TileShape           = Shape<_256, _256, _32>;                            // Threadblock-level tile size
using ClusterShape        = Shape<_1,_1,_1>;                                // Shape of the threadblocks in a cluster
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `epilogue`, `TileShape`, `ClusterShape`, `Shape` showing the main symbols being prepared or consumed here. It corresponds to block 21 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `epilogue`、`TileShape`、`ClusterShape`、`Shape` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 21/98 个代码块。

### Lines 125-132
````cpp
using GmemTiledCopyA = XE_2D_U16x32x32_LD_N;
using GmemTiledCopyB = XE_2D_U16x32x32_LD_V;
using TiledMma =
    typename TiledMMAHelper<MMA_Atom<XE_8x16x16_F32BF16BF16F32_TT>, Layout<TileShape>,
                                  Layout<Shape<_8, _4, _1>, Stride<_4, _1, _0>>>::TiledMMA;
constexpr int PipelineStages = 3;
using GEMMDispatchPolicy = cutlass::gemm::MainloopIntelXeXMX16<PipelineStages>;
using EpilogueDispatchPolicy = cutlass::epilogue::IntelXeXMX16;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `epilogue`, `Epilogue`, `TileShape`, `Shape` make the later softmax-related computation code easier to assemble and read. It corresponds to block 22 of 98 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `epilogue`、`Epilogue`、`TileShape`、`Shape` 这样的符号让后续softmax 相关计算代码更容易组装和阅读。 它对应本文件顺序中的第 22/98 个代码块。

### Lines 134-138
````cpp
// Top-K + Softmax fusion operation
using EpilogueFusionOperation     = std::conditional_t<EnableTopKSoftmax,
  typename cutlass::epilogue::fusion::LinCombTopKSoftmaxCol<TopK, ElementD, ElementCompute>,
  typename cutlass::epilogue::fusion::LinearCombination<ElementD, ElementCompute, ElementC, ElementCompute>
>;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `epilogue`, `Epilogue` showing the main symbols being prepared or consumed here. It corresponds to block 23 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `epilogue`、`Epilogue` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 23/98 个代码块。

### Lines 140-141
````cpp
// The fusion op only allows for epilogue tiles matching the mainloop tile.
using EpilogueTileType    = decltype(cute::take<0,2>(TileShape{}));
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `epilogue`, `Epilogue`, `mainloop`, `TileShape` showing the main symbols being prepared or consumed here. It corresponds to block 24 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `epilogue`、`Epilogue`、`mainloop`、`TileShape` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 24/98 个代码块。

### Lines 143-156
````cpp
using FusionCallBacks = cutlass::epilogue::fusion::FusionCallbacks<EpilogueDispatchPolicy, EpilogueFusionOperation, TileShape,
        decltype(tile_shape(TiledMma()))>;
using CollectiveEpilogue = cutlass::epilogue::collective::CollectiveEpilogue<
      EpilogueDispatchPolicy,
      TileShape,
      ElementAccumulator,
      cutlass::gemm::TagToStrideC_t<LayoutC>,
      ElementD,
      cutlass::gemm::TagToStrideC_t<LayoutD>,
      FusionCallBacks,
      XE_2D_U32x8x16_LD_N,
      void, void,
      XE_2D_U32x8x16_ST_N,
      void, void>;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `epilogue`, `Epilogue`, `TileShape`, `Shape` make the later softmax-related computation code easier to assemble and read. It corresponds to block 25 of 98 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `epilogue`、`Epilogue`、`TileShape`、`Shape` 这样的符号让后续softmax 相关计算代码更容易组装和阅读。 它对应本文件顺序中的第 25/98 个代码块。

### Lines 158-168
````cpp
using CollectiveMainloop = cutlass::gemm::collective::CollectiveMma<
  GEMMDispatchPolicy,
  TileShape,
  ElementA,
  cutlass::gemm::TagToStrideA_t<LayoutA>,
  ElementB,
  cutlass::gemm::TagToStrideB_t<LayoutB>,
  TiledMma,
  GmemTiledCopyA, void, void, cute::identity,  // A
  GmemTiledCopyB, void, void, cute::identity   // B
>;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `TileShape`, `Shape`, `Layout`, `cute` make the later softmax-related computation code easier to assemble and read. It corresponds to block 26 of 98 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `TileShape`、`Shape`、`Layout`、`cute` 这样的符号让后续softmax 相关计算代码更容易组装和阅读。 它对应本文件顺序中的第 26/98 个代码块。

### Lines 170-174
````cpp
using GemmKernel = cutlass::gemm::kernel::GemmUniversal<
    Shape<int,int,int,int>, // Indicates ProblemShape
    CollectiveMainloop,
    CollectiveEpilogue
>;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `GemmUniversal`, `Epilogue`, `Shape` make the later softmax-related computation code easier to assemble and read. It corresponds to block 27 of 98 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `GemmUniversal`、`Epilogue`、`Shape` 这样的符号让后续softmax 相关计算代码更容易组装和阅读。 它对应本文件顺序中的第 27/98 个代码块。

### Lines 176-176
````cpp
using Gemm = cutlass::gemm::device::GemmUniversalAdapter<GemmKernel>;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `GemmUniversalAdapter`, `GemmUniversal` make the later softmax-related computation code easier to assemble and read. It corresponds to block 28 of 98 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `GemmUniversalAdapter`、`GemmUniversal` 这样的符号让后续softmax 相关计算代码更容易组装和阅读。 它对应本文件顺序中的第 28/98 个代码块。

### Lines 178-180
````cpp
// Extract information from Gemm kernel.
using EpilogueOutputOp  = typename Gemm::EpilogueOutputOp;
using ElementScalar     = typename EpilogueOutputOp::ElementScalar;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Epilogue` showing the main symbols being prepared or consumed here. It corresponds to block 29 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Epilogue` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 29/98 个代码块。

### Lines 182-184
````cpp
using StrideA = typename Gemm::GemmKernel::StrideA;
using StrideB = typename Gemm::GemmKernel::StrideB;
using StrideD = typename Gemm::GemmKernel::StrideD;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `StrideA`, `Gemm::GemmKernel::StrideA`, `StrideB`, `Gemm::GemmKernel::StrideB` make the later softmax-related computation code easier to assemble and read. It corresponds to block 30 of 98 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `StrideA`、`Gemm::GemmKernel::StrideA`、`StrideB`、`Gemm::GemmKernel::StrideB` 这样的符号让后续softmax 相关计算代码更容易组装和阅读。 它对应本文件顺序中的第 30/98 个代码块。

### Lines 186-190
````cpp
/// Initialization
StrideA stride_A;
StrideB stride_B;
StrideD stride_D;
uint64_t seed;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Initialization`, `StrideA`, `stride_A`, `StrideB` showing the main symbols being prepared or consumed here. It corresponds to block 31 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Initialization`、`StrideA`、`stride_A`、`StrideB` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 31/98 个代码块。

### Lines 192-192
````cpp
using LayoutScalar = cutlass::layout::PackedVectorLayout;
````
**EN:** This block defines aliases or compile-time configuration values. Symbols such as `Layout` make the later softmax-related computation code easier to assemble and read. It corresponds to block 32 of 98 in the file order.
**CN:** 这一段定义别名或编译期配置值。像 `Layout` 这样的符号让后续softmax 相关计算代码更容易组装和阅读。 它对应本文件顺序中的第 32/98 个代码块。

### Lines 194-196
````cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
/// Testbed utility types
/////////////////////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Testbed`, `utility`, `types` showing the main symbols being prepared or consumed here. It corresponds to block 33 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Testbed`、`utility`、`types` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 33/98 个代码块。

### Lines 198-199
````cpp
// Command line options parsing
struct Options {
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Command`, `line`, `options`, `parsing` showing the main symbols being prepared or consumed here. It corresponds to block 34 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Command`、`line`、`options`、`parsing` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 34/98 个代码块。

### Lines 201-201
````cpp
  bool help = false;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `bool`, `help`, `false` showing the main symbols being prepared or consumed here. It corresponds to block 35 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `bool`、`help`、`false` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 35/98 个代码块。

### Lines 203-206
````cpp
  int iterations = 1000;
  int m = 16, n = 8, k = 64, l = 1;
  int verify = 1;
  double eps = 1e-5;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 36 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 36/98 个代码块。

### Lines 208-210
````cpp
  // Parses the command line
  void parse(int argc, char const **args) {
    cutlass::CommandLine cmd(argc, args);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Parses`, `the`, `command`, `line` showing the main symbols being prepared or consumed here. It corresponds to block 37 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Parses`、`the`、`command`、`line` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 37/98 个代码块。

### Lines 212-215
````cpp
    if (cmd.check_cmd_line_flag("help")) {
      help = true;
      return;
    }
````
**EN:** This block applies conditional control flow. It uses `cmd`, `check_cmd_line_flag`, `help`, `true` to select a path, validate assumptions, or handle special cases in the softmax-related computation implementation. It corresponds to block 38 of 98 in the file order.
**CN:** 这一段实现条件控制流。它借助 `cmd`、`check_cmd_line_flag`、`help`、`true` 在softmax 相关计算实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 38/98 个代码块。

### Lines 217-224
````cpp
    cmd.get_cmd_line_argument("m", m);
    cmd.get_cmd_line_argument("n", n);
    cmd.get_cmd_line_argument("k", k);
    cmd.get_cmd_line_argument("l", l);
    cmd.get_cmd_line_argument("iterations", iterations);
    cmd.get_cmd_line_argument("verify", verify, 1);
    cmd.get_cmd_line_argument("eps", eps);
  }
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `verify` showing the main symbols being prepared or consumed here. It corresponds to block 39 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `verify` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 39/98 个代码块。

### Lines 226-227
````cpp
  /// Prints the usage statement.
  std::ostream & print_usage(std::ostream &out) const {
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Prints`, `the`, `usage`, `statement` showing the main symbols being prepared or consumed here. It corresponds to block 40 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Prints`、`the`、`usage`、`statement` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 40/98 个代码块。

### Lines 229-239
````cpp
    out << "05_bmg_gemm_with_topk_and_softmax\n\n"
      << "  BMG GEMM with Top-K and softmax fusion.\n\n"
      << "Options:\n\n"
      << "  --help                      If specified, displays this usage statement\n\n"
      << "  --m=<int>                   Sets the M extent of the GEMM\n"
      << "  --n=<int>                   Sets the N extent of the GEMM\n"
      << "  --k=<int>                   Sets the K extent of the GEMM\n"
      << "  --l=<int>                   Sets the l extent (batch) of the GEMM\n"
      << "  --iterations=<int>          Number of profiling iterations to perform.\n"
      << "  --verify=<int>              Specify whether to verify. Default: 1\n"
      << "  --eps=<float>               Threshold of numerical verification. Default: 1e-5.\n\n";
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `softmax`, `verify`, `topk` showing the main symbols being prepared or consumed here. It corresponds to block 41 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `softmax`、`verify`、`topk` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 41/98 个代码块。

### Lines 241-243
````cpp
    out
      << "\n\nExamples:\n\n"
      << "$ " << "05_bmg_gemm_with_topk_and_softmax" << " --m=16 --n=8 --k=1024 \n\n";
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `softmax`, `topk` showing the main symbols being prepared or consumed here. It corresponds to block 42 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `softmax`、`topk` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 42/98 个代码块。

### Lines 245-246
````cpp
    return out;
  }
````
**EN:** This block finalizes a local computation or status path. The use of `out` helps conclude the current stage cleanly before the next block. It corresponds to block 43 of 98 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `out`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 43/98 个代码块。

### Lines 248-255
````cpp
  /// Compute performance in GFLOP/s
  double gflops(double runtime_s) const
  {
    // Two flops per multiply-add
    uint64_t flop = uint64_t(2) * m * n * k;
    double gflop = double(flop) / double(1.0e9);
    return gflop / runtime_s;
  }
````
**EN:** This block finalizes a local computation or status path. The use of `Compute`, `performance`, `in`, `GFLOP` helps conclude the current stage cleanly before the next block. It corresponds to block 44 of 98 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `Compute`、`performance`、`in`、`GFLOP`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 44/98 个代码块。

### Lines 257-260
````cpp
  float alpha() const {
    return 1.f / static_cast<float>(k);
  }
};
````
**EN:** This block introduces executable logic through a function or method. Here, `alpha`, `f`, `static_cast<float`, `k` drive a concrete step in the file's softmax-related computation flow. It corresponds to block 45 of 98 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `alpha`、`f`、`static_cast<float`、`k` 推动了本文件softmax 相关计算流程中的一个具体步骤。 它对应本文件顺序中的第 45/98 个代码块。

### Lines 262-268
````cpp
/// Result structure
struct Result {
  double avg_runtime_ms;
  double gflops;
  cutlass::Status status;
  cutlass::cudaError_t error;
  bool passed;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `cuda` showing the main symbols being prepared or consumed here. It corresponds to block 46 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `cuda` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 46/98 个代码块。

### Lines 271-274
````cpp
  cutlass::HostTensor<ElementA  , LayoutA  > tensor_A;
  cutlass::HostTensor<ElementB  , LayoutB  > tensor_B;
  cutlass::HostTensor<ElementD  , LayoutD  > tensor_D;
  cutlass::HostTensor<ElementD  , LayoutD  > tensor_ref_D;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Layout`, `Tensor` showing the main symbols being prepared or consumed here. It corresponds to block 47 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Layout`、`Tensor` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 47/98 个代码块。

### Lines 276-283
````cpp
  Result(
    double avg_runtime_ms = 0,
    double gflops = 0,
    cutlass::Status status = cutlass::Status::kSuccess,
    cutlass::cudaError_t error = cutlass::cudaSuccess)
  :
    avg_runtime_ms(avg_runtime_ms), gflops(gflops), status(status), error(error), passed(false)
  {}
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `cuda` showing the main symbols being prepared or consumed here. It corresponds to block 48 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `cuda` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 48/98 个代码块。

### Lines 285-287
````cpp
  /////////////////////////////////////////////////////////////////////////////////////////////////
  /// GEMM setup and evaluation
  /////////////////////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `GEMM`, `setup`, `and`, `evaluation` showing the main symbols being prepared or consumed here. It corresponds to block 49 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `GEMM`、`setup`、`and`、`evaluation` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 49/98 个代码块。

### Lines 289-297
````cpp
  /// Helper to initialize a block of device data
  template <typename Element, typename Layout>
  bool initialize_tensor(
      cutlass::TensorView<Element, Layout> view,
      uint64_t seed) {
    cutlass::reference::host::TensorFillRandomUniform(
      view, seed, /* max = */ 1, /* min = */ -1, /* bits = */ 2);
    return true;
  }
````
**EN:** This block finalizes a local computation or status path. The use of `Layout`, `Tensor`, `reference` helps conclude the current stage cleanly before the next block. It corresponds to block 50 of 98 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `Layout`、`Tensor`、`reference`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 50/98 个代码块。

### Lines 299-300
````cpp
  /// Initialize operands to be used in the GEMM and reference GEMM
  void initialize(const Options &options) {
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `reference` showing the main symbols being prepared or consumed here. It corresponds to block 51 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `reference` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 51/98 个代码块。

### Lines 302-304
````cpp
    stride_A = cutlass::make_cute_packed_stride(StrideA{}, cute::make_shape(options.m, options.k, options.l));
    stride_B = cutlass::make_cute_packed_stride(StrideB{}, cute::make_shape(options.n, options.k, options.l));
    stride_D = cutlass::make_cute_packed_stride(StrideD{}, cute::make_shape(options.m, options.n, options.l));
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `cute` showing the main symbols being prepared or consumed here. It corresponds to block 52 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 52/98 个代码块。

### Lines 306-308
````cpp
    auto a_coord = cutlass::make_Coord(options.m * options.l, options.k);
    auto c_coord = cutlass::make_Coord(options.m * options.l, options.n);
    auto b_coord = cutlass::make_Coord(options.k, options.n * options.l);
````
**EN:** This block introduces executable logic through a function or method. Here, `a_coord`, `cutlass::make_Coord`, `options`, `m` drive a concrete step in the file's softmax-related computation flow. It corresponds to block 53 of 98 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `a_coord`、`cutlass::make_Coord`、`options`、`m` 推动了本文件softmax 相关计算流程中的一个具体步骤。 它对应本文件顺序中的第 53/98 个代码块。

### Lines 310-313
````cpp
    tensor_A.resize(a_coord);
    tensor_B.resize(b_coord);
    tensor_D.resize(c_coord);
    tensor_ref_D.resize(c_coord);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `tensor_A`, `resize`, `a_coord`, `tensor_B` showing the main symbols being prepared or consumed here. It corresponds to block 54 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `tensor_A`、`resize`、`a_coord`、`tensor_B` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 54/98 个代码块。

### Lines 315-316
````cpp
    initialize_tensor(tensor_A.host_view(), seed + 2022);
    initialize_tensor(tensor_B.host_view(), seed + 2023);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `initialize_tensor`, `tensor_A`, `host_view`, `seed` showing the main symbols being prepared or consumed here. It corresponds to block 55 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `initialize_tensor`、`tensor_A`、`host_view`、`seed` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 55/98 个代码块。

### Lines 318-321
````cpp
    tensor_A.sync_device();
    tensor_B.sync_device();
    tensor_D.sync_device();
  }
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `tensor_A`, `sync_device`, `tensor_B`, `tensor_D` showing the main symbols being prepared or consumed here. It corresponds to block 56 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `tensor_A`、`sync_device`、`tensor_B`、`tensor_D` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 56/98 个代码块。

### Lines 323-334
````cpp
  /// Populates a Gemm::Arguments structure from the given commandline options
  typename Gemm::Arguments args_from_options(const Options &options) {
    typename Gemm::Arguments arguments{
      cutlass::gemm::GemmUniversalMode::kGemm,
      {options.m, options.n, options.k, options.l},
      {tensor_A.device_data(), stride_A, tensor_B.device_data(), stride_B},
      {
        {options.alpha(), 0.f}, // alpha, beta
        nullptr, stride_D,
        tensor_D.device_data(), stride_D
      }
    };
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `GemmUniversal` showing the main symbols being prepared or consumed here. It corresponds to block 57 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `GemmUniversal` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 57/98 个代码块。

### Lines 336-337
````cpp
    return arguments;
  }
````
**EN:** This block finalizes a local computation or status path. The use of `arguments` helps conclude the current stage cleanly before the next block. It corresponds to block 58 of 98 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `arguments`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 58/98 个代码块。

### Lines 339-342
````cpp
  bool verify(const Options &options) {
    //
    // Compute reference output
    //
````
**EN:** This block introduces executable logic through a function or method. Here, `verify`, `reference` drive a concrete step in the file's softmax-related computation flow. It corresponds to block 59 of 98 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `verify`、`reference` 推动了本文件softmax 相关计算流程中的一个具体步骤。 它对应本文件顺序中的第 59/98 个代码块。

### Lines 344-351
````cpp
    // Create instantiation for device reference gemm kernel
    auto A = cute::make_tensor(tensor_A.host_data(),
        cute::make_layout(cute::make_shape(options.m, options.k, options.l), stride_A));
    auto B = cute::make_tensor(tensor_B.host_data(),
        cute::make_layout(cute::make_shape(options.n, options.k, options.l), stride_B));
    auto D = cute::make_tensor(tensor_ref_D.host_data(),
        cute::make_layout(cute::make_shape(options.m, options.n, options.l), stride_D));
    using unused_t = decltype(D);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `cute`, `reference` showing the main symbols being prepared or consumed here. It corresponds to block 60 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `cute`、`reference` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 60/98 个代码块。

### Lines 353-353
````cpp
    cutlass::reference::host::GettMainloopParams<ElementAccumulator, decltype(A), decltype(B)> mainloop_params{A, B};
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `mainloop`, `reference` showing the main symbols being prepared or consumed here. It corresponds to block 61 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `mainloop`、`reference` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 61/98 个代码块。

### Lines 355-366
````cpp
    cutlass::reference::host::GettEpilogueParams<
        ElementScalar,
        ElementScalar,
        ElementAccumulator,
        ElementCompute,
        unused_t,
        decltype(D),
        unused_t, // bias
        unused_t, // aux
        unused_t, // valpha
        unused_t  // vbeta
    > epilogue_params;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `epilogue`, `Epilogue`, `reference`, `bias` showing the main symbols being prepared or consumed here. It corresponds to block 62 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `epilogue`、`Epilogue`、`reference`、`bias` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 62/98 个代码块。

### Lines 368-370
````cpp
    epilogue_params.D = D;
    epilogue_params.alpha = options.alpha();
    epilogue_params.beta = 0.f;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `epilogue` showing the main symbols being prepared or consumed here. It corresponds to block 63 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `epilogue` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 63/98 个代码块。

### Lines 372-373
````cpp
    // get reference result
    cutlass::reference::host::Gemm3x(mainloop_params, epilogue_params);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `epilogue`, `mainloop`, `reference` showing the main symbols being prepared or consumed here. It corresponds to block 64 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `epilogue`、`mainloop`、`reference` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 64/98 个代码块。

### Lines 375-378
````cpp
    if constexpr (EnableTopKSoftmax) {
      // top-K + softmax
      for (int k = 0; k < options.l; ++k) {
        for (int i = 0; i < options.m; ++i) {
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `softmax` advances the file toward execution, checking, or benchmarking. It corresponds to block 65 of 98 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `softmax` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 65/98 个代码块。

### Lines 380-393
````cpp
          // Find Top-K
          cutlass::Array<ElementAccumulator, TopK> top_k;
          top_k.fill(-cutlass::platform::numeric_limits<ElementCompute>::infinity());
          for (int j = 0; j < options.n; ++j) {
            auto val = static_cast<ElementAccumulator>(tensor_ref_D.host_view().ref().at({i + k * options.m, j}));
            for (int top_k_idx = 0; top_k_idx < TopK; ++top_k_idx) {
              if (val > top_k[top_k_idx]) {
                // Shift down
                for (int l = TopK - 1; l > top_k_idx; --l) {
                  top_k[l] = top_k[l - 1];
                }
                top_k[top_k_idx] = val;
                break;
              }
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `Find`, `Top`, `K`, `cutlass::Array<ElementAccumulator` advances the file toward execution, checking, or benchmarking. It corresponds to block 66 of 98 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `Find`、`Top`、`K`、`cutlass::Array<ElementAccumulator` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 66/98 个代码块。

### Lines 394-395
````cpp
            }
          }
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 67 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 67/98 个代码块。

### Lines 397-403
````cpp
          // This formulation of top-K + softmax only works when it is
          // guaranteed that none of the top-K elements are repeated!
          // If this is the case, the device kernel can also make mistakes, because
          //   A. Once the top-K values are reduced, and the operation is being applied,
          //      there is no way to tell repeated elements apart, so none are masked.
          //   B. The softmax sum of exps will be incorrect (because the repeated elements
          //      are not repeated in it.)
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `softmax` showing the main symbols being prepared or consumed here. It corresponds to block 68 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `softmax` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 68/98 个代码块。

### Lines 405-409
````cpp
          ElementAccumulator max = top_k[0];
          ElementAccumulator sum = ElementAccumulator(0.f);
          for (int top_k_idx = 0; top_k_idx < TopK; ++top_k_idx) {
            sum = sum + cutlass::fast_exp(top_k[top_k_idx] - max);
          }
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `ElementAccumulator`, `max`, `top_k`, `sum` advances the file toward execution, checking, or benchmarking. It corresponds to block 69 of 98 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `ElementAccumulator`、`max`、`top_k`、`sum` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 69/98 个代码块。

### Lines 411-423
````cpp
          for (int j=0; j < options.n; ++j) {
            auto val = tensor_ref_D.host_view().ref().at({i + k * options.m, j});
            if (val < top_k[TopK - 1]) {
              tensor_ref_D.host_view().ref().at({i + k * options.m, j}) = static_cast<ElementD>(0.f);
            } else {
              // Softmax
              auto softmax_val = cutlass::fast_exp(val - max) / sum;
              tensor_ref_D.host_view().ref().at({i + k * options.m, j}) = static_cast<ElementD>(softmax_val);
            }
          }
        }
      }
    }
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `softmax` advances the file toward execution, checking, or benchmarking. It corresponds to block 70 of 98 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `softmax` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 70/98 个代码块。

### Lines 425-426
````cpp
    // compare_reference
    tensor_D.sync_host();
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `reference` showing the main symbols being prepared or consumed here. It corresponds to block 71 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `reference` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 71/98 个代码块。

### Lines 428-431
````cpp
    double err = cutlass::reference::host::TensorRelativeErrorMetric(
      tensor_D.host_view(),
      tensor_ref_D.host_view());
    bool passed = err < options.eps;
````
**EN:** This block introduces executable logic through a function or method. Here, `Tensor`, `reference` drive a concrete step in the file's softmax-related computation flow. It corresponds to block 72 of 98 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `Tensor`、`reference` 推动了本文件softmax 相关计算流程中的一个具体步骤。 它对应本文件顺序中的第 72/98 个代码块。

### Lines 433-436
````cpp
    if (options.m <= 32 && options.n <= 32) {
      std::cout << "GEMM output:\n" << tensor_D.host_view() << "\n\n";
      std::cout << "Reference output:\n" << tensor_ref_D.host_view() << "\n\n";
    }
````
**EN:** This block applies conditional control flow. It uses `options`, `m`, `n`, `GEMM` to select a path, validate assumptions, or handle special cases in the softmax-related computation implementation. It corresponds to block 73 of 98 in the file order.
**CN:** 这一段实现条件控制流。它借助 `options`、`m`、`n`、`GEMM` 在softmax 相关计算实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 73/98 个代码块。

### Lines 438-438
````cpp
    std::cout << "  Disposition: " << (passed ? "Passed" : "Failed") << " \t Relative error: " << err << std::endl;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Disposition`, `passed`, `Passed`, `Failed` showing the main symbols being prepared or consumed here. It corresponds to block 74 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Disposition`、`passed`、`Passed`、`Failed` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 74/98 个代码块。

### Lines 440-441
````cpp
    return passed;
  }
````
**EN:** This block finalizes a local computation or status path. The use of `passed` helps conclude the current stage cleanly before the next block. It corresponds to block 75 of 98 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 `passed`，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 75/98 个代码块。

### Lines 443-443
````cpp
};
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 76 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 76/98 个代码块。

### Lines 445-449
````cpp
/// Execute a given example GEMM computation
template <typename Gemm>
int run(Options &options) {
  // Instantiate CUTLASS kernel depending on templates
  Gemm gemm;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `cute` showing the main symbols being prepared or consumed here. It corresponds to block 77 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `cute` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 77/98 个代码块。

### Lines 451-452
````cpp
  Result result;
  result.initialize(options);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Result`, `result`, `initialize`, `options` showing the main symbols being prepared or consumed here. It corresponds to block 78 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Result`、`result`、`initialize`、`options` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 78/98 个代码块。

### Lines 454-455
````cpp
  // Create a structure of gemm kernel arguments suitable for invoking an instance of Gemm
  auto arguments = result.args_from_options(options);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Create`, `a`, `structure`, `of` showing the main symbols being prepared or consumed here. It corresponds to block 79 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Create`、`a`、`structure`、`of` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 79/98 个代码块。

### Lines 457-458
````cpp
  // Using the arguments, query for extra workspace required for matrix multiplication computation
  size_t workspace_size = Gemm::get_workspace_size(arguments);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `workspace` showing the main symbols being prepared or consumed here. It corresponds to block 80 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `workspace` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 80/98 个代码块。

### Lines 460-461
````cpp
  // Allocate workspace memory
  cutlass::device_memory::allocation<uint8_t> workspace(workspace_size);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `workspace` showing the main symbols being prepared or consumed here. It corresponds to block 81 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `workspace` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 81/98 个代码块。

### Lines 463-464
````cpp
  // Check if the problem size is supported or not
  CUTLASS_CHECK(gemm.can_implement(arguments));
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Check`, `the`, `problem`, `size` showing the main symbols being prepared or consumed here. It corresponds to block 82 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Check`、`the`、`problem`、`size` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 82/98 个代码块。

### Lines 466-467
````cpp
  // Initialize CUTLASS kernel with arguments and workspace pointer
  CUTLASS_CHECK(gemm.initialize(arguments, workspace.get()));
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `workspace` showing the main symbols being prepared or consumed here. It corresponds to block 83 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `workspace` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 83/98 个代码块。

### Lines 469-470
````cpp
  // Correctness / Warmup iteration
  CUTLASS_CHECK(gemm.run());
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Correctness`, `Warmup`, `iteration`, `CUTLASS_CHECK` showing the main symbols being prepared or consumed here. It corresponds to block 84 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Correctness`、`Warmup`、`iteration`、`CUTLASS_CHECK` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 84/98 个代码块。

### Lines 472-475
````cpp
  if (options.verify != 0) {
    // Check if output from CUTLASS kernel and reference kernel are equal or not
    result.passed = result.verify(options);
    std::cout << "Disposition: " << (result.passed ? "Passed" : "Failed") << std::endl;
````
**EN:** This block applies conditional control flow. It uses `verify`, `reference` to select a path, validate assumptions, or handle special cases in the softmax-related computation implementation. It corresponds to block 85 of 98 in the file order.
**CN:** 这一段实现条件控制流。它借助 `verify`、`reference` 在softmax 相关计算实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 85/98 个代码块。

### Lines 477-482
````cpp
    if (!result.passed) {
      return -1;
    }
  } else {
    std::cout << "Disposition is skipped." << std::endl;
  }
````
**EN:** This block applies conditional control flow. It uses `result`, `passed`, `else`, `Disposition` to select a path, validate assumptions, or handle special cases in the softmax-related computation implementation. It corresponds to block 86 of 98 in the file order.
**CN:** 这一段实现条件控制流。它借助 `result`、`passed`、`else`、`Disposition` 在softmax 相关计算实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 86/98 个代码块。

### Lines 484-491
````cpp
  // Run profiling loop
  if (options.iterations > 0) {
    GpuTimer timer;
    timer.start();
    for (int iter = 0; iter < options.iterations; ++iter) {
      CUTLASS_CHECK(gemm.run());
    }
    timer.stop();
````
**EN:** This block iterates over problem instances or data movement steps. The loop around `Run`, `profiling`, `loop`, `options` advances the file toward execution, checking, or benchmarking. It corresponds to block 87 of 98 in the file order.
**CN:** 这一段围绕问题规模或数据移动步骤进行迭代。围绕 `Run`、`profiling`、`loop`、`options` 的循环把文件推进到执行、校验或基准测试阶段。 它对应本文件顺序中的第 87/98 个代码块。

### Lines 493-496
````cpp
    // Compute average runtime and GFLOPs.
    float elapsed_ms = timer.elapsed_millis();
    result.avg_runtime_ms = double(elapsed_ms) / double(options.iterations);
    result.gflops = options.gflops(result.avg_runtime_ms / 1000.0);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Compute`, `average`, `runtime`, `and` showing the main symbols being prepared or consumed here. It corresponds to block 88 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Compute`、`average`、`runtime`、`and` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 88/98 个代码块。

### Lines 498-501
````cpp
    std::cout << "  Problem Size: " << options.m << 'x' << options.n << 'x' << options.k << 'x' << options.l << std::endl;
    std::cout << "  Avg runtime: " << result.avg_runtime_ms << " ms" << std::endl;
    std::cout << "  GFLOPS: " << result.gflops << std::endl;
  }
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Problem`, `Size`, `options`, `m` showing the main symbols being prepared or consumed here. It corresponds to block 89 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Problem`、`Size`、`options`、`m` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 89/98 个代码块。

### Lines 503-504
````cpp
  return 0;
}
````
**EN:** This block finalizes a local computation or status path. The use of the surrounding symbols helps conclude the current stage cleanly before the next block. It corresponds to block 90 of 98 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 周围的符号，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 90/98 个代码块。

### Lines 506-506
````cpp
///////////////////////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 91 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 91/98 个代码块。

### Lines 508-511
````cpp
int main(int argc, char const **args) {
  //
  // Parse options
  //
````
**EN:** This block introduces executable logic through a function or method. Here, `main`, `argc`, `char`, `args` drive a concrete step in the file's softmax-related computation flow. It corresponds to block 92 of 98 in the file order.
**CN:** 这一段通过函数或方法引入可执行逻辑，其中 `main`、`argc`、`char`、`args` 推动了本文件softmax 相关计算流程中的一个具体步骤。 它对应本文件顺序中的第 92/98 个代码块。

### Lines 513-513
````cpp
  Options options;
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Options`, `options` showing the main symbols being prepared or consumed here. It corresponds to block 93 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Options`、`options` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 93/98 个代码块。

### Lines 515-515
````cpp
  options.parse(argc, args);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `options`, `parse`, `argc`, `args` showing the main symbols being prepared or consumed here. It corresponds to block 94 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `options`、`parse`、`argc`、`args` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 94/98 个代码块。

### Lines 517-520
````cpp
  if (options.help) {
    options.print_usage(std::cout) << std::endl;
    return 0;
  }
````
**EN:** This block applies conditional control flow. It uses `options`, `help`, `print_usage` to select a path, validate assumptions, or handle special cases in the softmax-related computation implementation. It corresponds to block 95 of 98 in the file order.
**CN:** 这一段实现条件控制流。它借助 `options`、`help`、`print_usage` 在softmax 相关计算实现中选择路径、验证假设或处理特殊情况。 它对应本文件顺序中的第 95/98 个代码块。

### Lines 522-525
````cpp
  //
  // Evaluate CUTLASS kernels
  //
  run<Gemm>(options);
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with `Evaluate`, `CUTLASS`, `kernels`, `run<Gemm` showing the main symbols being prepared or consumed here. It corresponds to block 96 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 `Evaluate`、`CUTLASS`、`kernels`、`run<Gemm` 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 96/98 个代码块。

### Lines 527-528
````cpp
  return 0;
}
````
**EN:** This block finalizes a local computation or status path. The use of the surrounding symbols helps conclude the current stage cleanly before the next block. It corresponds to block 97 of 98 in the file order.
**CN:** 这一段结束局部计算或状态路径。借助 周围的符号，当前阶段会被清晰地收束，再进入下一段。 它对应本文件顺序中的第 97/98 个代码块。

### Lines 530-530
````cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
````
**EN:** This block continues the file's softmax-related computation setup or compute path, with the surrounding symbols showing the main symbols being prepared or consumed here. It corresponds to block 98 of 98 in the file order.
**CN:** 这一段继续推进本文件的softmax 相关计算初始化或计算流程，其中 周围的符号 展示了此处正在准备或使用的主要符号。 它对应本文件顺序中的第 98/98 个代码块。

## Key Concepts / 关键概念
- **EN:** Template-based GEMM composition with CUTLASS collectives, adapters, or visitors.
  **CN:** 使用 CUTLASS collective、adapter 或 visitor 进行模板化 GEMM 组合。
- **EN:** Post-processing is fused into the compute path through epilogue logic.
  **CN:** 后处理逻辑通过 epilogue 融合进主计算路径。
- **EN:** The file mixes compile-time configuration with runtime problem sizes or arguments.
  **CN:** 该文件把编译期配置与运行期问题规模或参数结合起来。
- **EN:** Validation, benchmarking, or reporting code is interleaved with kernel setup.
  **CN:** 校验、基准测试或结果报告代码与内核配置过程交织在一起。

## Dependencies / 依赖关系
- **Direct dependencies / 直接依赖:** `cutlass/epilogue/collective/default_epilogue.hpp`, `cutlass/epilogue/collective/xe_epilogue.hpp`, `cutlass/epilogue/fusion/xe_callbacks.hpp`, `cutlass/gemm/device/gemm_universal.h`, `cutlass/gemm/device/gemm_universal_adapter.h`, `cutlass/gemm/collective/collective_mma.hpp`, `cutlass/util/GPU_Clock.hpp`, `cute/tensor.hpp`, `random`, `cutlass/util/host_tensor.h`, `cutlass/util/command_line.h`, `cutlass/util/device_memory.h`, ...
- **Runtime expectations / 运行时依赖:** CUTLASS/CUTE templates / CUTLASS/CUTE 模板
- **Build/context note / 构建说明:** The file relies on surrounding repository infrastructure, compile flags, and example utilities. / 该文件依赖仓库周边基础设施、编译选项以及示例辅助代码。
