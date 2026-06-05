# gemm_configuration_sycl.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/gemm/legacy/gemm_configuration_sycl.hpp`
- **EN:** Legacy Intel Xe SYCL GEMM configuration layer that supports both standard FP32-output GEMM and mixed-precision GEMM with scale/zero metadata.
- **CN:** 旧版 Intel Xe SYCL GEMM 配置层：同时支持标准 FP32 输出 GEMM 和带 scale/zero 元数据的混合精度 GEMM。

## Line-by-Line Analysis / 逐行分析

### Lines 1-53 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
   3 |  * SPDX-License-Identifier: BSD-3-Clause
   4 |  *
   5 |  * Redistribution and use in source and binary forms, with or without
   6 |  * modification, are permitted provided that the following conditions are met:
   7 |  *
   8 |  * 1. Redistributions of source code must retain the above copyright notice, this
   9 |  * list of conditions and the following disclaimer.
  10 |  *
  11 |  * 2. Redistributions in binary form must reproduce the above copyright notice,
  12 |  * this list of conditions and the following disclaimer in the documentation
  13 |  * and/or other materials provided with the distribution.
  14 |  *
  15 |  * 3. Neither the name of the copyright holder nor the names of its
  16 |  * contributors may be used to endorse or promote products derived from
  17 |  * this software without specific prior written permission.
  18 |  *
  19 |  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  20 |  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  21 |  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  22 |  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  23 |  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  24 |  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  25 |  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  26 |  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  27 |  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  28 |  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  29 |  *
  30 |  **************************************************************************************************/
  31 | 
  32 | #pragma once
  33 | 
  34 | #include "cute/atom/mma_atom.hpp"
  35 | #include "cute/atom/copy_atom.hpp"
  36 | 
  37 | #include "cutlass/cutlass.h"
  38 | #include "cutlass/gemm/gemm.h"
  39 | #include "cutlass/arch/arch.h"
  40 | #include "cutlass/arch/mma.h"
  41 | #include "cutlass/layout/layout.h"
  42 | #include "cutlass/gemm/dispatch_policy.hpp"
  43 | #include "cutlass/gemm/collective/collective_mma.hpp"
  44 | #include "cutlass/gemm/collective/collective_builder.hpp"
  45 | #include "cutlass/epilogue/collective/collective_builder.hpp"
  46 | 
  47 | #include "cutlass/epilogue/collective/default_epilogue.hpp"
  48 | #include "cutlass/epilogue/thread/linear_combination.h"
  49 | 
  50 | using namespace cute;
  51 | 
  52 | namespace cutlass::gemm::device {
  53 | 
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 54-80 — Scheduler and primary templates
```cpp
  54 | enum class Scheduler { Gemm, GemmSplitK, GemmStreamK };
  55 | 
  56 | template<
  57 |   class ArchTag,
  58 |   class ElementA, class LayoutA,
  59 |   class ElementB, class LayoutB,
  60 |   class ElementC, class LayoutC,
  61 |   class ElementAccumulator,
  62 |   class TileShape, Scheduler TileScheduler, class TiledMma = void,
  63 |   class GmemTiledCopyA = void, class GmemTiledCopyB = void,
  64 |   class EpilogueOp = epilogue::fusion::LinearCombination<float, float, float, float, FloatRoundStyle::round_to_nearest>>
  65 | struct GemmConfiguration {
  66 |   static_assert(sizeof(ElementA) == 0, "No valid GemmConfiguration configuration exists.");
  67 | };
  68 | 
  69 | template<
  70 |   class ArchTag,
  71 |   class ElementA, class LayoutA,
  72 |   class ElementB, class LayoutB, class ElementC, typename LayoutC,
  73 |   class ElementScale, typename StrideS,
  74 |   class ElementZero, typename StrideZ,
  75 |   class TileShape, Scheduler TileScheduler,
  76 |   class TiledMma, class GmemTiledCopyA, class GmemTiledCopyB,
  77 |   class GmemTiledCopyC,   class EpilogueOp, int Stages = 3>
  78 | struct MixedPrecisionGemmConfiguration{
  79 |   static_assert(sizeof(ElementA) == 0, "No valid MixedPrecisionGemmConfiguration configuration exists.");
  80 | };
```
**EN:** Defines the scheduler enum plus fail-fast primary templates for both regular GEMM and mixed-precision GEMM configurations.
**CN:** 定义调度器枚举，并为常规 GEMM 与混合精度 GEMM 配置提供失败即停止的主模板。

### Lines 84-173 — Standard Intel Xe GEMM specialization
```cpp
  84 | // bfloat16
  85 | 
  86 | template<class ElementA, class LayoutA,
  87 |   class ElementB, class LayoutB, typename LayoutC,
  88 |   class TileShape, Scheduler TileScheduler,
  89 |   class TiledMma, class GmemTiledCopyA, class GmemTiledCopyB,  class EpilogueOp>
  90 | struct GemmConfiguration<
  91 |       arch::IntelXe,
  92 |       ElementA, LayoutA,
  93 |       ElementB, LayoutB,
  94 |       float, LayoutC,
  95 |       float,
  96 |       TileShape, TileScheduler, TiledMma,
  97 |       GmemTiledCopyA, GmemTiledCopyB, EpilogueOp>
  98 | {
  99 |   using KernelScheduleType = std::conditional_t<TileScheduler == Scheduler::Gemm,
 100 |     cutlass::gemm::KernelXe, cutlass::gemm::KernelXeCooperative>;
 101 |   using DispatchPolicy = MainloopIntelXeXMX16<3, KernelScheduleType>;
 102 | 
 103 |   // Configurations in benchmarks.hpp can pass either a layout tag (e.g. RowMajor) or a Stride directly
 104 |   using StrideA = std::conditional_t<cute::is_tuple_v<LayoutA>, LayoutA, TagToStrideA_t<LayoutA>>;
 105 |   using StrideB = std::conditional_t<cute::is_tuple_v<LayoutB>, LayoutB, TagToStrideB_t<LayoutB>>;
 106 |   using StrideC = std::conditional_t<cute::is_tuple_v<LayoutC>, LayoutC, TagToStrideC_t<LayoutC>>;
 107 | 
 108 |   using ClusterShape = Shape<_1, _1, _1>;
 109 |   static constexpr bool use_collective_mma_builder = std::is_void_v<TiledMma>;
 110 |   static_assert(
 111 |     use_collective_mma_builder == std::is_void_v<GmemTiledCopyA> and
 112 |     use_collective_mma_builder == std::is_void_v<GmemTiledCopyB>,
 113 |     "TiledMma, GmemTileCopyA, and GmemTileCopyB must be all void or none of them may be void."
 114 |   );
 115 |   // Mainloop
 116 |   using CollectiveMainloop =
 117 |     std::conditional_t<use_collective_mma_builder,
 118 |       typename cutlass::gemm::collective::CollectiveBuilder<
 119 |         cutlass::arch::IntelXe, cutlass::arch::OpClassTensorOp,
 120 |         ElementA, LayoutA, sizeof(ElementA),
 121 |         ElementB, LayoutB, sizeof(ElementB),
 122 |         float,
 123 |         TileShape, ClusterShape,
 124 |         cutlass::gemm::collective::StageCountAuto,
 125 |         KernelScheduleType
 126 |       >::CollectiveOp,
 127 |       collective::CollectiveMma<
 128 |         DispatchPolicy, TileShape,
 129 |         ElementA, StrideA,
 130 |         ElementB, StrideB,
 131 |         TiledMma,
 132 |         GmemTiledCopyA, void, void, identity, // A
 133 |         GmemTiledCopyB, void, void, identity // B
 134 |   >>;
 135 | 
 136 |   // Epilogue
 137 |   using CollectiveEpilogue = typename cutlass::epilogue::collective::CollectiveBuilder<
 138 |     cutlass::arch::IntelXe, cutlass::arch::OpClassTensorOp,
 139 |     TileShape, ClusterShape,
 140 |     cutlass::epilogue::collective::EpilogueTileAuto, float,
 141 |     float,
 142 |     float, LayoutC, sizeof(float),
 143 |     float, LayoutC, sizeof(float),
 144 |     cutlass::epilogue::collective::EpilogueScheduleAuto,
 145 |     EpilogueOp
 146 |   >::CollectiveOp;
 147 | 
 148 |   using GemmKernel = kernel::GemmUniversal<
 149 |     Shape<int, int, int, int>,
 150 |     CollectiveMainloop,
 151 |     CollectiveEpilogue,
 152 |     std::conditional_t<TileScheduler == Scheduler::Gemm, void, cutlass::gemm::StreamKScheduler>
 153 |   >;
 154 | 
 155 |   using Gemm = GemmUniversalAdapter<GemmKernel>;
 156 | 
 157 |   constexpr static typename GemmKernel::Arguments defaultArguments() {
 158 |     using StreamKMode =
 159 |       cutlass::gemm::kernel::detail::PersistentTileSchedulerXeStreamKParams::DecompositionMode;
 160 |     if constexpr (TileScheduler == Scheduler::Gemm) {
 161 |       return {};
 162 |     } else if constexpr (TileScheduler == Scheduler::GemmStreamK) {
 163 |       typename GemmKernel::Arguments arguments{};
 164 |       arguments.scheduler = {1, StreamKMode::StreamK};
 165 |       return arguments;
 166 |     } else {
 167 |       static_assert(TileScheduler == Scheduler::GemmSplitK);
 168 |       typename GemmKernel::Arguments arguments{};
 169 |       arguments.scheduler = {2, StreamKMode::SplitK};
 170 |       return arguments;
 171 |     }
 172 |   }
 173 | };
```
**EN:** Assembles the normal Intel Xe GEMM path, including stride normalization, optional CollectiveBuilder usage, epilogue construction, and scheduler defaults.
**CN:** 组装常规 Intel Xe GEMM 路径，包括步长规范化、可选的 CollectiveBuilder 用法、epilogue 构建和调度器默认值。

### Lines 175-249 — Mixed-precision specialization
```cpp
 175 | template<class ElementA, class LayoutA,
 176 |   class ElementB, class LayoutB,
 177 |   class ElementC, typename LayoutC,
 178 |   class ElementScale, typename StrideS,
 179 |   class ElementZero, typename StrideZ,
 180 |   class TileShape, Scheduler TileScheduler,
 181 |   class TiledMma, class GmemTiledCopyA, class GmemTiledCopyB,
 182 |   class GmemTiledCopyC, class EpilogueOp, int Stages>
 183 | struct MixedPrecisionGemmConfiguration<
 184 |       arch::IntelXe,
 185 |       ElementA, LayoutA,
 186 |       ElementB, LayoutB,
 187 |       ElementC, LayoutC,
 188 |       ElementScale, StrideS,
 189 |       ElementZero, StrideZ,
 190 |       TileShape, TileScheduler, TiledMma,
 191 |       GmemTiledCopyA, GmemTiledCopyB,
 192 |       GmemTiledCopyC, EpilogueOp, Stages>
 193 | {
 194 |   using LayoutD = LayoutC;
 195 | 
 196 |   using GEMMDispatchPolicy = cutlass::gemm::MainloopIntelXeXMX16MixedPrecision<Stages>;
 197 |   using EpilogueDispatchPolicy = cutlass::epilogue::IntelXeXMX16;
 198 | 
 199 |   using ElementAccumulator = typename TiledMma::ValTypeD;
 200 | 
 201 |   using FusionCallBacks = cutlass::epilogue::fusion::FusionCallbacks<EpilogueDispatchPolicy, EpilogueOp, TileShape,
 202 |           decltype(tile_shape(TiledMma()))>;
 203 |   using CollectiveEpilogue = cutlass::epilogue::collective::CollectiveEpilogue<
 204 |           EpilogueDispatchPolicy,
 205 |           TileShape,
 206 |           ElementAccumulator,
 207 |           cutlass::gemm::TagToStrideC_t<LayoutC>,
 208 |           ElementC,
 209 |           cutlass::gemm::TagToStrideC_t<LayoutD>,
 210 |           FusionCallBacks,
 211 |           XE_2D_U32x8x16_LD_N,
 212 |           void, void,
 213 |           GmemTiledCopyC,
 214 |           void, void>;
 215 | 
 216 |   static constexpr bool IsAQuant = cutlass::platform::numeric_limits<ElementA>::is_integer
 217 |                                     ^ cutlass::platform::numeric_limits<ElementAccumulator>::is_integer;
 218 |   static constexpr bool IsBQuant = cutlass::platform::numeric_limits<ElementB>::is_integer
 219 |                                     ^ cutlass::platform::numeric_limits<ElementAccumulator>::is_integer;
 220 | 
 221 |   using CollectiveMainloop = collective::CollectiveMma<GEMMDispatchPolicy, TileShape,
 222 |                                                        cute::conditional_t<IsAQuant, cute::tuple<ElementA, ElementScale, StrideS, ElementZero, StrideZ>, ElementA>,
 223 |                                                        cutlass::gemm::TagToStrideA_t<LayoutA>,
 224 |                                                        cute::conditional_t<IsBQuant, cute::tuple<ElementB, ElementScale, StrideS, ElementZero, StrideZ>, ElementB>,
 225 |                                                        cutlass::gemm::TagToStrideB_t<LayoutB>, TiledMma,
 226 |                                                        GmemTiledCopyA, void, void, cute::identity, GmemTiledCopyB, void, void,
 227 |                                                        cute::identity>;
 228 | 
 229 |   using GemmKernel = kernel::GemmUniversal<Shape<int, int, int, int>, CollectiveMainloop, CollectiveEpilogue>;
 230 | 
 231 |   using Gemm = device::GemmUniversalAdapter<GemmKernel>;
 232 | 
 233 |   constexpr static typename GemmKernel::Arguments defaultArguments() {
 234 |     using StreamKMode =
 235 |       cutlass::gemm::kernel::detail::PersistentTileSchedulerXeStreamKParams::DecompositionMode;
 236 |     if constexpr (TileScheduler == Scheduler::Gemm) {
 237 |       return {};
 238 |     } else if constexpr (TileScheduler == Scheduler::GemmStreamK) {
 239 |       typename GemmKernel::Arguments arguments{};
 240 |       arguments.scheduler = {1, StreamKMode::StreamK};
 241 |       return arguments;
 242 |     } else {
 243 |       static_assert(TileScheduler == Scheduler::GemmSplitK);
 244 |       typename GemmKernel::Arguments arguments{};
 245 |       arguments.scheduler = {2, StreamKMode::SplitK};
 246 |       return arguments;
 247 |     }
 248 |   }
 249 | };
```
**EN:** Builds the mixed-precision Intel Xe GEMM path, threading scale and zero-point metadata through the mainloop and fused epilogue.
**CN:** 构建混合精度的 Intel Xe GEMM 路径，把 scale 与 zero-point 元数据贯穿到 mainloop 和融合 epilogue 中。

### Lines 250-251 — File epilogue
```cpp
 250 | 
 251 | } // namespace cutlass::gemm::device
```
**EN:** This trailing block closes scopes or keeps small glue code that finalizes the file structure.
**CN:** 这一尾部代码块用于关闭作用域，或保留收尾用的小型胶水代码，从而完成文件结构。

## Key Concepts / 关键概念

- Scheduler enum / 调度器枚举
- Standard GEMM specialization / 标准 GEMM 特化
- Mixed-precision specialization / 混合精度特化

## Dependencies / 依赖关系

- `CollectiveBuilder` and `CollectiveMma` — mainloop construction / mainloop 构建
- `FusionCallbacks` — fused epilogue support / 融合 epilogue 支持
- `MainloopIntelXeXMX16MixedPrecision` — mixed-precision path / 混合精度路径
