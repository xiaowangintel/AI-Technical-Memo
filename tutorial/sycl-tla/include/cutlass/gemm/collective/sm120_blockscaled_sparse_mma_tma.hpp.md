# sm120_blockscaled_sparse_mma_tma.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/collective/sm120_blockscaled_sparse_mma_tma.hpp`
- **Purpose (EN):** Implements an architecture-specific `CollectiveMma` specialization for SM120, covering block-scaled operands, sparse metadata handling, TMA transfers.
- **用途 (CN):** 为 SM120 实现架构特化的 `CollectiveMma` 特化版本，重点覆盖 块缩放操作数、稀疏元数据处理、TMA 传输。
- **Lines / 行数:** 1320

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
/***************************************************************************************************
 * Copyright (c) 2025 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 33-33

```cpp
#pragma once
```
**EN:** `#pragma once` prevents duplicate inclusion of the header during compilation.
**CN:** `#pragma once` 用于防止编译过程中重复包含该头文件。

### Lines 35-42

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/pipeline/pipeline.hpp"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/detail/dependent_false.hpp"
#include "cutlass/trace.h"
#include "cutlass/numeric_types.h"
#include "cutlass/gemm/collective/builders/sm1xx_sparse_config.inl"
```
**EN:** This include block imports cutlass.h, gemm.h, pipeline.hpp, dispatch_policy.hpp, and 4 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 cutlass.h、gemm.h、pipeline.hpp、dispatch_policy.hpp 等 8 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 44-49

```cpp
#include "cute/arch/cluster_sm90.hpp"
#include "cute/arch/copy_sm90.hpp"
#include "cute/atom/mma_atom.hpp"
#include "cute/algorithm/functional.hpp"
#include "cute/algorithm/gemm.hpp"
#include "cute/numeric/arithmetic_tuple.hpp"
```
**EN:** This include block imports cluster_sm90.hpp, copy_sm90.hpp, mma_atom.hpp, functional.hpp, and 2 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 cluster_sm90.hpp、copy_sm90.hpp、mma_atom.hpp、functional.hpp 等 6 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 51-51

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 53-54

```cpp
namespace cutlass::gemm::collective {
using namespace cute;
```
**EN:** This short block both opens the collective namespace and pulls CuTe symbols into scope, setting up a concise vocabulary for the rest of the file.
**CN:** 这个短代码块既打开了 collective 命名空间，也把 CuTe 符号引入作用域，为后续代码建立更简洁的表达方式。

### Lines 56-57

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
/// CollectiveMma for A/B with different or same stages based on asymmetric DMA.
```
**EN:** This comment-only block labels or explains the following section:  CollectiveMma for A/B with different or same stages based on asymmetric DMA..
**CN:** 这个纯注释块用于标记或解释后续区域： CollectiveMma for A/B with different or same stages based on asymmetric DMA.。

### Lines 59-103

```cpp
template <
  int StagesA,
  int StagesB,
  int StagesE,
  int SchedulerPipelineStageCount,
  class ClusterShape,
  class TileShape_,
  class ElementPairA_,
  class LayoutPairsA_,
  class ElementPairB_,
  class StridePairB_,
  class TiledMma_,
  class GmemTiledCopyPairA_,
  class SmemLayoutAtomsA_,
  class SmemCopyAtomsA_,
  class TransformA_,
  class GmemTiledCopyPairB_,
  class SmemLayoutAtomsB_,
  class SmemCopyAtomsB_,
  class TransformB_>
struct CollectiveMma<
    MainloopSm120TmaWarpSpecializedSparseBlockScaled<StagesA, StagesB, StagesE, SchedulerPipelineStageCount, ClusterShape>,
    TileShape_,
    ElementPairA_,
    LayoutPairsA_,
    ElementPairB_,
    StridePairB_,
    TiledMma_,
    GmemTiledCopyPairA_,
    SmemLayoutAtomsA_,
    SmemCopyAtomsA_,
    TransformA_,
    GmemTiledCopyPairB_,
    SmemLayoutAtomsB_,
    SmemCopyAtomsB_,
    TransformB_> {
  //
  // Type Aliases
  //
  using ElementPairA = ElementPairA_;
  using ElementPairB = ElementPairB_;
  using LayoutPairsA = LayoutPairsA_;
  using StridePairB = StridePairB_;
  using SmemCopyAtomsA = SmemCopyAtomsA_;
  using SmemCopyAtomsB = SmemCopyAtomsB_;
```
**EN:** This template block declares or specializes `CollectiveMma`, the mainloop object that coordinates tile movement and matrix-multiply work.
**CN:** 这个模板块声明或特化了 `CollectiveMma`，它负责协调 tile 搬运与矩阵乘主循环。

### Lines 105-122

```cpp
  using TiledMma = TiledMma_;
  using AtomThrShapeMNK = Shape<decltype(shape<0>(typename TiledMma::ThrLayoutVMNK{})), _1, _1>;
  using DispatchPolicy = MainloopSm120TmaWarpSpecializedSparseBlockScaled<StagesA, StagesB, StagesE, SchedulerPipelineStageCount, ClusterShape>;
  using TileShape = TileShape_;
  using ElementA = remove_cvref_t<decltype(get<0>(ElementPairA{}))>;
  using ElementAMma = typename TiledMma::ValTypeA;
  using ElementAMmaRaw = typename ElementAMma::raw_type;
  using LayoutA =  remove_cvref_t<decltype(get<0>(LayoutPairsA{}))>;
  using LayoutE =  remove_cvref_t<decltype(get<1>(LayoutPairsA{}))>;
  using StrideA =  remove_cvref_t<decltype(get<3>(LayoutPairsA{}))>;
  using ElementB = remove_cvref_t<decltype(get<0>(ElementPairB{}))>;
  using StrideB = remove_cvref_t<decltype(get<0>(StridePairB{}))>;
  using ElementBMma = typename TiledMma::ValTypeB;
  using ElementEMma = typename TiledMma::ValTypeE;
  using ElementE = typename ElementEMma::raw_type;
  using RegisterE = typename remove_extent<typename TiledMma::MMA_Op::ERegisters>::type;
  using ArrayElementA = ElementA;
  using ArrayElementB = ElementB;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 124-151

```cpp
  // SFA, SFB and metadata config
  static_assert(cute::is_same_v<remove_cvref_t<decltype(get<1>(ElementPairA{}))>,
                                remove_cvref_t<decltype(get<1>(ElementPairB{}))>>,
                                "SFA and SFB data types should be the same");
  using ElementSF = remove_cvref_t<decltype(get<1>(ElementPairA{}))>;
  using LayoutSFA = remove_cvref_t<decltype(get<2>(LayoutPairsA{}))>;
  using LayoutSFB = remove_cvref_t<decltype(get<1>(StridePairB{}))>;
  static constexpr int SFVecSize = TiledMma::Traits::SFVecSize;
  using Sm1xxBlkScaledConfig = cutlass::detail::Sm1xxBlockScaledConfig<SFVecSize>;
  using CtaShape_MNK = decltype(shape_div(TileShape{}, ClusterShape{}));
  using ElementAccumulator = typename TiledMma::ValTypeC;
  using GmemTiledCopyA = remove_cvref_t<decltype(get<0>(GmemTiledCopyPairA_{}))>;
  using GmemTiledCopyB = remove_cvref_t<decltype(get<0>(GmemTiledCopyPairB_{}))>;;
  using SmemCopyAtomA = remove_cvref_t<decltype(get<0>(SmemCopyAtomsA{}))>;
  using SmemCopyAtomE = remove_cvref_t<decltype(get<1>(SmemCopyAtomsA{}))>;
  using SmemCopyAtomB = remove_cvref_t<decltype(get<0>(SmemCopyAtomsB{}))>;
  using SmemLayoutAtomA = remove_cvref_t<decltype(get<0>(SmemLayoutAtomsA_{}))>;
  using SmemLayoutAtomB = remove_cvref_t<decltype(get<0>(SmemLayoutAtomsB_{}))>;
  using SmemLayoutAtomSFA = remove_cvref_t<decltype(get<1>(SmemLayoutAtomsA_{}))>;
  using SmemLayoutAtomSFB = remove_cvref_t<decltype(get<1>(SmemLayoutAtomsB_{}))>;
  using SmemCopyAtomSFA = remove_cvref_t<decltype(get<2>(SmemCopyAtomsA{}))>;
  using SmemCopyAtomSFB = remove_cvref_t<decltype(get<1>(SmemCopyAtomsB{}))>;
  using GmemTiledCopySFA = remove_cvref_t<decltype(get<1>(GmemTiledCopyPairA_{}))>;
  using GmemTiledCopySFB = remove_cvref_t<decltype(get<1>(GmemTiledCopyPairB_{}))>;
  using TransformA = TransformA_;
  using TransformB = TransformB_;
  using ArchTag = typename DispatchPolicy::ArchTag;
  using GmemTiledCopyE = GmemTiledCopyA;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 153-158

```cpp
  // Asymmetric buffering
  // Tensor A/B could have different buffering, with TILEK, and STAGEs.
  //    It let AsymmetricKRatio equals TILEK_A / TILEK_B, to make sure A/B's
  //    pipeline keep same steps when produce / consume data.
  // Currently, AsymmetricKRatio = {1, 2} is the only support.
  static constexpr int AsymmetricKRatio = DispatchPolicy::StagesA != DispatchPolicy::StagesB ? 2 : 1;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 160-169

```cpp
  // Construct TileShape for SFB load from GMEM to SMEM.
  // It is required to keep consistency with BlockScaled granularity defined in Sm1xxBlkScaledConfig.
  // So that TileShape for scaling factor needs to be defined as a multiple of Blk_MN.
  using Blk_MN      = typename Sm1xxBlkScaledConfig::Blk_MN;
  using TileShapeSF = decltype(make_shape(ceil_div(size<0>(CtaShape_MNK{}), Blk_MN{}) * Blk_MN{},
                                           ceil_div(size<1>(CtaShape_MNK{}), Blk_MN{}) * Blk_MN{},
                                           shape<2>(CtaShape_MNK{})));
  using TileShapeB = decltype(make_shape(size<0>(TileShape{}),
                                         size<1>(TileShape{}),
                                         ceil_div(size<2>(TileShape{}), Int<AsymmetricKRatio>{})));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 171-174

```cpp
  static constexpr int ThreadCount = size(TiledMma{});
  static constexpr int IsCtaN64 = shape<1>(CtaShape_MNK{}) == 64;
  static constexpr int TensorAMmaSparsity = ElementAMma::sparsity;
  static constexpr int TensorEMmaSparsity = ElementEMma::sparsity;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 176-181

```cpp
  // Use two MainloopPipeline for A and B separately.
  using MainloopPipelineMK = cutlass::PipelineTmaAsync<DispatchPolicy::StagesA>;
  using MainloopPipelineNK = cutlass::PipelineTmaAsync<DispatchPolicy::StagesB>;
  using PipelineStateMK  = typename cutlass::PipelineState<DispatchPolicy::StagesA>;
  using PipelineStateNK  = typename cutlass::PipelineState<DispatchPolicy::StagesB>;
  using PipelineParams = typename MainloopPipelineMK::Params;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 183-185

```cpp
  static_assert(rank(SmemLayoutAtomA{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 187-189

```cpp
  static_assert(rank(SmemLayoutAtomB{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<1>(TileShape{}) % size<0>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 191-194

```cpp
  static_assert(not cute::is_void_v<SmemCopyAtomA>,
    "SM120 mainloop must specify a copy atom for A operand smem->rmem reads.");
  static_assert(not cute::is_void_v<SmemCopyAtomB>,
    "SM120 mainloop must specify a copy atom for B operand smem->rmem reads.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 196-216

```cpp
  // Tile along modes in a way that maximizes the TMA box size.
  // Note: SmemA, SmemSFA and SmemSFB are with same stages, while SmemB is with another stage number.
  // SmemSFB is not with same stages as SmemB, as it will not design 1.5x stages if Smem not enough.
  // These different stages setting could maximize capacity of latency hide, while keep data in SMEM.
  // Metadata may kept in SMEM, or in GMEM/L2, if under SMEM limitation.
  using SmemLayoutA = decltype(tile_to_shape(
      SmemLayoutAtomA{},
      make_shape(shape<0>(TileShape{}), shape<2>(TileShape{}), Int<DispatchPolicy::StagesA>{}),
      conditional_t< ::cutlass::gemm::detail::is_major<0,StrideA>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
  using SmemLayoutB = decltype(tile_to_shape(
      SmemLayoutAtomB{},
      make_shape(shape<1>(TileShapeB{}), shape<2>(TileShapeB{}), Int<DispatchPolicy::StagesB>{}),
      conditional_t< ::cutlass::gemm::detail::is_major<0,StrideB>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
  using SmemLayoutSFA = decltype(make_layout(
    append(shape(SmemLayoutAtomSFA{}), Int<DispatchPolicy::StagesA>{}),
    append(stride(SmemLayoutAtomSFA{}), size(filter_zeros(SmemLayoutAtomSFA{})))
  ));
  using SmemLayoutSFB = decltype(make_layout(
    append(shape(SmemLayoutAtomSFB{}), Int<DispatchPolicy::StagesA>{}),
    append(stride(SmemLayoutAtomSFB{}), size(filter_zeros(SmemLayoutAtomSFB{})))
  ));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 218-219

```cpp
  static_assert(rank(SmemLayoutA{}) == 3, "Smem layout must be rank 3.");
  static_assert(rank(SmemLayoutB{}) == 3, "Smem layout must be rank 3.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 221-229

```cpp
  static_assert(DispatchPolicy::StagesA >= 2, "Specialization requires StagesA set to value 2 or more.");
  static_assert(DispatchPolicy::StagesB >= 2, "Specialization requires StagesB set to value 2 or more.");
  static_assert(not cute::is_base_of<cute::GMMA::DescriptorIterator, typename TiledMma::FrgTypeA>::value &&
                not cute::is_base_of<cute::GMMA::DescriptorIterator, typename TiledMma::FrgTypeB>::value,
                "MMA atom must source both A and B operands from rmem for this mainloop.");
  static_assert(cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD>,
                  "GmemTiledCopy - invalid SM90 TMA copy atom specified.");
  static_assert(cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD>,
                  "GmemTiledCopy - invalid SM90 TMA copy atom specified.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 231-231

```cpp
  static constexpr bool IsF8F6F4 = detail::is_sm100_sparse_f8f6f4<TiledMma, ElementA, ElementB>();
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 233-234

```cpp
  // Is E kept in SMEM or GMEM
  static constexpr bool UseSmemE = DispatchPolicy::StagesE != 0;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 236-246

```cpp
  // For all other types, cast to size equivalent uint type to avoid any rounding by TMA.
  using TmaInternalElementA = cute::conditional_t<not IsF8F6F4,
                                                  ElementA,
                              cute::conditional_t<cute::is_same_v<ElementA, cutlass::float_e2m1_t>,
                                                  cutlass::detail::float_e2m1_unpacksmem_t,
                              cute::conditional_t<cute::is_same_v<ElementA, cutlass::float_e2m3_t>,
                                                cutlass::detail::float_e2m3_unpacksmem_t,
                              cute::conditional_t<cute::is_same_v<ElementA, cutlass::float_e3m2_t>,
                                                cutlass::detail::float_e3m2_unpacksmem_t,
                                                uint_bit_t<sizeof_bits_v<ElementA>>>>>>;
  using TmaSourceElementA = cute::conditional_t<IsF8F6F4, ElementA, uint8_t>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 248-256

```cpp
  using TmaInternalElementB = cute::conditional_t<not IsF8F6F4,
                                                  ElementB,
                              cute::conditional_t<cute::is_same_v<ElementB, cutlass::float_e2m1_t>,
                                                  cutlass::detail::float_e2m1_unpacksmem_t,
                              cute::conditional_t<cute::is_same_v<ElementB, cutlass::float_e2m3_t>,
                                                cutlass::detail::float_e2m3_unpacksmem_t,
                              cute::conditional_t<cute::is_same_v<ElementB, cutlass::float_e3m2_t>,
                                                cutlass::detail::float_e3m2_unpacksmem_t,
                                                uint_bit_t<sizeof_bits_v<ElementB>>>>>>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 258-260

```cpp
  // Set shared memory layout
  using SmemAllocTypeA = cute::conditional_t<IsF8F6F4, sparse_elem<TensorAMmaSparsity, uint8_t>, ElementAMma>;
  using SmemAllocTypeB = cute::conditional_t<IsF8F6F4, uint8_t, ElementBMma>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 262-279

```cpp
  static constexpr bool is_A_mn_major = cute::is_same_v<decltype(stride<0>(LayoutA{})), Int<TensorAMmaSparsity>>;
  using SparseConfig = cutlass::Sm1xxGemmSparseConfig<ElementAMma,
                                                      cute::conditional_t<is_A_mn_major, cutlass::layout::ColumnMajor, cutlass::layout::RowMajor>,
                                                      ElementEMma>;
  using SmemLayoutAtomE_ = typename SparseConfig::TensorEAtom;
  using SmemLayoutAtomE  = ComposedLayout<Swizzle<0,4,3>,
                                          smem_sparse_ptr_flag_bits<TensorEMmaSparsity, sizeof_bits_v<ElementE>>,
                                          SmemLayoutAtomE_>;
  using SmemLayoutE = decltype(tile_to_shape(
                  SmemLayoutAtomE{},
                  make_shape(shape<0>(TileShape{}), shape<2>(TileShape{}), Int<DispatchPolicy::StagesE>{}),
                  conditional_t< ::cutlass::gemm::detail::is_major<0,StrideA>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
  static constexpr int SmemSizeE  = UseSmemE ? cosize(SmemLayoutE{}) : 0;
  static constexpr int StageSizeE = UseSmemE ? cosize(take<0,2>(SmemLayoutE{})) : 0;
  // Check if metetata fetching needs predication
  using TensorEAtomM = typename SparseConfig::TensorEAtomM;
  using TensorEAtomK = typename SparseConfig::TensorEAtomK;
  static constexpr bool IsELoadPred = not (TensorEAtomM{} == size<0>(TileShape{}) && TensorEAtomK{} == size<2>(TileShape{}));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 281-282

```cpp
  static_assert(rank(SmemLayoutAtomE{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomE{})) == 0, "SmemLayoutAtomE must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 284-292

```cpp
  // Set the bytes transferred in this TMA transaction
  static constexpr uint32_t TmaTransactionBytesMK = static_cast<uint32_t>(
    cutlass::bits_to_bytes(cosize(take<0,2>(SmemLayoutA{})) * cute::sizeof_bits_v<ElementAMma>) +
    cutlass::bits_to_bytes(cosize(take<0,2>(SmemLayoutSFA{})) * cute::sizeof_bits_v<ElementSF>) +
    cutlass::bits_to_bytes(cosize(take<0,2>(SmemLayoutSFB{})) * cute::sizeof_bits_v<ElementSF>) +
    cutlass::bits_to_bytes(StageSizeE * cute::sizeof_bits_v<ElementEMma>));
  static constexpr uint32_t TmaTransactionBytesNK = static_cast<uint32_t>(
    cutlass::bits_to_bytes(cosize(take<0,2>(SmemLayoutB{})) * cute::sizeof_bits_v<ElementB>));
  static constexpr uint32_t TmaTransactionBytes = TmaTransactionBytesMK + TmaTransactionBytesNK;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 294-301

```cpp
  struct SharedStorage {
    struct TensorStorage : cute::aligned_struct<128> {
      alignas(1024) cute::ArrayEngine<SmemAllocTypeA, cute::cosize_v<SmemLayoutA>> smem_A;
      alignas(1024) cute::ArrayEngine<SmemAllocTypeB, cute::cosize_v<SmemLayoutB>> smem_B;
      cute::ArrayEngine<ElementSF, cute::cosize_v<SmemLayoutSFA>> smem_SFA;
      cute::ArrayEngine<ElementSF, cute::cosize_v<SmemLayoutSFB>> smem_SFB;
      cute::ArrayEngine<ElementEMma, Int<SmemSizeE>{}> smem_E;
    } tensors;
```
**EN:** This block introduces `SharedStorage` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `SharedStorage`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 303-310

```cpp
    using PipelineStorageMK = typename MainloopPipelineMK::SharedStorage;
    using PipelineStorageNK = typename MainloopPipelineNK::SharedStorage;
    alignas(16) PipelineStorageMK pipeline_storage_mk;
    alignas(16) PipelineStorageNK pipeline_storage_nk;
  };
  using TensorStorage = typename SharedStorage::TensorStorage;
  using PipelineStorageMK = typename SharedStorage::PipelineStorageMK;
  using PipelineStorageNK = typename SharedStorage::PipelineStorageNK;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 312-323

```cpp
  struct Arguments {
    ElementA const* ptr_A{nullptr};
    LayoutA layout_a{};
    ElementB const* ptr_B{nullptr};
    StrideB dB{};
    ElementE const* ptr_E{nullptr};
    LayoutE layout_e{};
    ElementSF const* ptr_SFA{nullptr};
    LayoutSFA layout_SFA{};
    ElementSF const* ptr_SFB{nullptr};
    LayoutSFB layout_SFB{};
  };
```
**EN:** This block introduces `Arguments` and groups related declarations around that symbol. `Arguments` is the host-side bundle: callers fill it with pointers, strides, and other launch-time values before parameter lowering.
**CN:** 这一段引入了 `Arguments`，并围绕该符号组织相关声明。 `Arguments` 是主机侧参数包：调用者在参数下沉前把指针、步长及其他启动期数据填入其中。

### Lines 325-372

```cpp
  // Device side kernel params
  struct Params {
    // Assumption: StrideA is congruent with Problem_MK
    using TMA_A = decltype(make_tma_copy<TmaInternalElementA>(
        GmemTiledCopyA{},
        make_tensor(recast_ptr<sparse_elem<TensorAMmaSparsity,TmaSourceElementA>>(nullptr), LayoutA{}),
        SmemLayoutA{}(_,_,0),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        _1{}));
    // Assumption: StrideB is congruent with Problem_NK
    using TMA_B = decltype(make_tma_copy(
        GmemTiledCopyB{},
        make_tensor(recast_ptr<TmaInternalElementB>(nullptr), repeat_like(StrideB{}, int32_t(0)), StrideB{}),
        SmemLayoutB{}(_,_,0),
        make_shape(shape<1>(TileShapeB{}), shape<2>(TileShapeB{})),
        _1{}));
    using TMA_E = decltype(make_tma_copy<ElementE>(
        GmemTiledCopyA{},
        make_tensor(recast_ptr<ElementEMma>(nullptr), LayoutE{}),
        SmemLayoutE{}(_,_,0),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        _1{}));
    using TMA_SFA = decltype(make_tma_copy<uint16_t>(
        GmemTiledCopySFA{},
        make_tensor(static_cast<ElementSF const*>(nullptr), LayoutSFA{}),
        SmemLayoutSFA{}(_,_,cute::Int<0>{}),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        _1{}));
    using TMA_SFB = decltype(make_tma_copy<uint16_t>(
        GmemTiledCopySFB{},
        make_tensor(static_cast<ElementSF const*>(nullptr), LayoutSFB{}),
        SmemLayoutSFB{}(_,_,cute::Int<0>{}),
        make_shape(shape<1>(TileShapeSF{}), shape<2>(TileShapeSF{})),
        _1{}));
    TMA_A tma_load_a;
    TMA_B tma_load_b;
    TMA_E tma_load_e;
    TMA_SFA tma_load_sfa;
    TMA_SFB tma_load_sfb;
    LayoutA layout_a;
    LayoutE layout_e;
    LayoutSFA layout_SFA;
    LayoutSFB layout_SFB;
    ElementE const* ptr_E{nullptr};
    uint32_t tma_transaction_bytes_mk = TmaTransactionBytesMK;
    uint32_t tma_transaction_bytes_nk = TmaTransactionBytesNK;
    uint32_t tma_transaction_bytes = TmaTransactionBytes;
  };
```
**EN:** This block introduces `Params` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `Params`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 374-376

```cpp
  //
  // Methods
  //
```
**EN:** This comment starts a method section where the collective exposes its runtime behavior.
**CN:** 这条注释表示进入方法区，collective 的运行时行为将在这里定义。

### Lines 378-381

```cpp
  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(ProblemShape const& problem_shape, Arguments const& args, void* workspace) {
    (void) workspace;
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 383-385

```cpp
    // Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK)
    auto problem_shape_MNKL = append<4>(problem_shape, 1);
    auto [M, N, K, L] = problem_shape_MNKL;
```
**EN:** The surrounding comments explain the local purpose of this block: Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK).
**CN:** 周围注释解释了这一段的局部作用：Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK)。

### Lines 387-389

```cpp
    auto ptr_A = recast_ptr<sparse_elem<TensorAMmaSparsity, TmaSourceElementA>>(args.ptr_A);
    auto ptr_B = recast_ptr<TmaInternalElementB>(args.ptr_B);
    auto ptr_E = recast_ptr<ElementEMma>(args.ptr_E);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 391-395

```cpp
    Tensor tensor_a = make_tensor(ptr_A, args.layout_a);
    Tensor tensor_b = make_tensor(ptr_B, make_layout(make_shape(N,K,L), args.dB));
    Tensor tensor_e = make_tensor(ptr_E, args.layout_e);
    Tensor tensor_sfa = make_tensor(args.ptr_SFA, args.layout_SFA);
    Tensor tensor_sfb = make_tensor(args.ptr_SFB, args.layout_SFB);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 397-439

```cpp
    typename Params::TMA_A tma_load_a = make_tma_copy<TmaInternalElementA>(
        GmemTiledCopyA{},
        tensor_a,
        SmemLayoutA{}(_,_,cute::Int<0>{}),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        _1{});
    typename Params::TMA_B tma_load_b = make_tma_copy(
        GmemTiledCopyB{},
        tensor_b,
        SmemLayoutB{}(_,_,cute::Int<0>{}),
        make_shape(shape<1>(TileShape{}), shape<2>(TileShapeB{})),
        _1{});
    typename Params::TMA_E tma_load_e = make_tma_copy<ElementE>(
        GmemTiledCopyE{},
        tensor_e,
        SmemLayoutE{}(_,_,cute::Int<0>{}),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        _1{});
    typename Params::TMA_SFA tma_load_sfa = make_tma_copy<uint16_t>(
        GmemTiledCopySFA{},
        tensor_sfa,
        SmemLayoutSFA{}(_,_,cute::Int<0>{}),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        _1{});
    typename Params::TMA_SFB tma_load_sfb = make_tma_copy<uint16_t>(
        GmemTiledCopySFB{},
        tensor_sfb,
        SmemLayoutSFB{}(_,_,cute::Int<0>{}),
        make_shape(shape<1>(TileShapeSF{}), shape<2>(TileShapeSF{})),
        _1{});
    return {
      tma_load_a,
      tma_load_b,
      tma_load_e,
      tma_load_sfa,
      tma_load_sfb,
      args.layout_a,
      args.layout_e,
      args.layout_SFA,
      args.layout_SFB,
      args.ptr_E
    };
  }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 441-447

```cpp
  template<class ProblemShape>
  CUTLASS_HOST_DEVICE static bool
  can_implement(
      ProblemShape const& problem_shape,
      [[maybe_unused]] Arguments const& args) {
    auto problem_shape_MNKL = append<4>(problem_shape, 1);
    auto [M, N, K, L] = problem_shape_MNKL;
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. This method performs runtime feasibility checks, usually validating alignment, layout assumptions, or shape constraints before launching the kernel.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这个方法执行运行时可实现性检查，通常会在启动内核前验证对齐、布局假设或形状约束。

### Lines 449-450

```cpp
    constexpr int tma_alignment_bits_A = cutlass::detail::get_input_alignment_bits<ElementA, IsF8F6F4>();
    constexpr int tma_alignment_bits_B = cutlass::detail::get_input_alignment_bits<ElementB, IsF8F6F4>();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 452-456

```cpp
    bool implementable = true;
    constexpr int min_tma_aligned_elements_A = tma_alignment_bits_A / cutlass::sizeof_bits<ElementA>::value;
    implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_A>(cute::upcast<2>(make_layout(make_shape(M, K, L), StrideA{})));
    constexpr int min_tma_aligned_elements_B = tma_alignment_bits_B / cutlass::sizeof_bits<ElementB>::value;
    implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_B>(cute::make_shape(N,K,L), StrideB{});
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 458-462

```cpp
    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem Size doesn't meet the minimum alignment requirements for TMA.\n");
    }
    return implementable;
  }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 464-474

```cpp
  /// Issue Tma Descriptor Prefetch -- ideally from a single thread for best performance
  CUTLASS_DEVICE
  static void prefetch_tma_descriptors(Params const& mainloop_params) {
    cute::prefetch_tma_descriptor(mainloop_params.tma_load_a.get_tma_descriptor());
    cute::prefetch_tma_descriptor(mainloop_params.tma_load_b.get_tma_descriptor());
    cute::prefetch_tma_descriptor(mainloop_params.tma_load_sfa.get_tma_descriptor());
    cute::prefetch_tma_descriptor(mainloop_params.tma_load_sfb.get_tma_descriptor());
    if constexpr (UseSmemE) {
      cute::prefetch_tma_descriptor(mainloop_params.tma_load_e.get_tma_descriptor());
    }
  }
```
**EN:** The code prefetches descriptor state early so later TMA or copy operations can start with lower latency. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里会提前预取描述符状态，从而让后续 TMA 或拷贝操作以更低延迟启动。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 476-481

```cpp
  /// Create fragment for metadata. The function is referred from thrfrg_A(...)
  template <class Tensor, class Atom, class TiledThr, class TiledPerm>
  CUTE_HOST_DEVICE constexpr
  auto
  thrfrg_E(Tensor&& tensor, TiledMMA<Atom, TiledThr, TiledPerm>& mma) {
    CUTE_STATIC_ASSERT_V(rank(tensor) >= Int<2>{});
```
**EN:** This block introduces `Tensor` and groups related declarations around that symbol. The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这一段引入了 `Tensor`，并围绕该符号组织相关声明。 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 483-484

```cpp
    using AtomShape_MNK  = typename Atom::Shape_MNK;
    using AtomLayoutE_TV = typename Atom::Traits::ELayout;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 486-489

```cpp
    auto t_tile = make_tile(get<0>(TiledPerm{}),
                            get<2>(TiledPerm{}));
    auto thr_layout_vmnk = mma.get_thr_layout_vmnk();
    auto t_tensor = logical_divide(tensor, t_tile);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 491-494

```cpp
    // Tile the tensor for the Atom
    auto e_tile = make_tile(make_layout(size<0>(AtomShape_MNK{})),
                            make_layout(size<2>(AtomShape_MNK{})));
    auto e_tensor = zipped_divide(t_tensor, e_tile);                                   // ((AtomM,AtomK),(RestM,RestK))
```
**EN:** The surrounding comments explain the local purpose of this block: Tile the tensor for the Atom.
**CN:** 周围注释解释了这一段的局部作用：Tile the tensor for the Atom。

### Lines 496-497

```cpp
    // Transform the Atom mode from (M,K) to (Thr,Val)
    auto tv_tensor = e_tensor.compose(AtomLayoutE_TV{},_);                               // ((ThrV,FrgV),(RestM,RestK))
```
**EN:** The surrounding comments explain the local purpose of this block: Transform the Atom mode from (M,K) to (Thr,Val).
**CN:** 周围注释解释了这一段的局部作用：Transform the Atom mode from (M,K) to (Thr,Val)。

### Lines 499-503

```cpp
    // Tile the tensor for the Thread
    auto thr_tile = make_tile(_,
                              make_tile(make_layout(size<1>(thr_layout_vmnk)),
                                        make_layout(size<3>(thr_layout_vmnk))));
    auto thr_tensor = zipped_divide(tv_tensor, thr_tile);                  // ((ThrV,(ThrM,ThrK)),(FrgV,(RestM,RestK)))
```
**EN:** The surrounding comments explain the local purpose of this block: Tile the tensor for the Thread.
**CN:** 周围注释解释了这一段的局部作用：Tile the tensor for the Thread。

### Lines 505-507

```cpp
    // Fragment layout
    return thr_tensor;
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 509-518

```cpp
  /// get metadata TV
  template<class TiledMma>
  CUTE_HOST_DEVICE constexpr
  auto
  get_layoutE_TV(TiledMma& mma)
  {
      // (M,K) -> (M,K)
      auto tile_shape_mnk = tile_shape(mma);
      auto ref_E = make_layout(make_shape(size<0>(tile_shape_mnk), size<2>(tile_shape_mnk)));
      auto thr_layout_vmnk = mma.get_thr_layout_vmnk();
```
**EN:** This block introduces `TiledMma` and groups related declarations around that symbol.
**CN:** 这一段引入了 `TiledMma`，并围绕该符号组织相关声明。

### Lines 520-524

```cpp
      // (ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK))
      auto etile = make_tile(_,
                            make_tile(make_layout(make_shape (size<1>(thr_layout_vmnk), size<2>(thr_layout_vmnk)),
                                                  make_stride(               Int<1>{} ,                Int<0>{} )),
                                      _));
```
**EN:** The surrounding comments explain the local purpose of this block: (ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK)).
**CN:** 周围注释解释了这一段的局部作用：(ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK))。

### Lines 526-530

```cpp
      // thr_idx -> (ThrV,ThrM,ThrN,ThrK)
      auto thridx_2_thrid = right_inverse(thr_layout_vmnk);
      // (thr_idx,val) -> (M,K)
      return thrfrg_E(ref_E, mma).compose(etile, _).compose(thridx_2_thrid, _);
  }
```
**EN:** The surrounding comments explain the local purpose of this block: thr_idx -> (ThrV,ThrM,ThrN,ThrK) (thr_idx,val) -> (M,K).
**CN:** 周围注释解释了这一段的局部作用：thr_idx -> (ThrV,ThrM,ThrN,ThrK) (thr_idx,val) -> (M,K)。

### Lines 532-538

```cpp
  /// Partitioning for metadata.
  template <class Tensor, class ThrMma>
  CUTE_HOST_DEVICE constexpr
  auto
  partition_fragment_E(Tensor&& tensor, ThrMma& thread_mma) {
    auto thr_tensor = make_tensor(static_cast<Tensor&&>(tensor).data(), thrfrg_E(tensor.layout(),thread_mma));
    auto thr_vmnk = thread_mma.thr_vmnk_;
```
**EN:** This block introduces `Tensor` and groups related declarations around that symbol. It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 这一段引入了 `Tensor`，并围绕该符号组织相关声明。 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 540-543

```cpp
    auto thr_vmk = make_coord(get<0>(thr_vmnk), make_coord(get<1>(thr_vmnk), get<3>(thr_vmnk)));
    auto partition = thr_tensor(thr_vmk, make_coord(_, repeat<rank<1,1>(thr_tensor)>(_)));
    return make_fragment_like<ThrMma::Atom::Traits::ValTypeE>(partition.layout());
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 545-551

```cpp
  // Temporary adhoc partitioning for scaling factors.
  template <class SFATensor, class Atom, class TiledThr, class TiledPerm>
  CUTE_HOST_DEVICE constexpr
  auto
  thrfrg_SFA(SFATensor&& sfatensor, TiledMMA<Atom, TiledThr, TiledPerm>& mma)
  {
    CUTE_STATIC_ASSERT_V(rank(sfatensor) >= Int<2>{});
```
**EN:** This block introduces `SFATensor` and groups related declarations around that symbol. The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这一段引入了 `SFATensor`，并围绕该符号组织相关声明。 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 553-554

```cpp
    using AtomShape_MNK  = typename Atom::Shape_MNK;
    using AtomLayoutSFA_TV = typename Atom::Traits::SFALayout;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 556-557

```cpp
    auto permutation_mnk = TiledPerm{};
    auto thr_layout_vmnk = mma.get_thr_layout_vmnk();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 559-562

```cpp
    // Reorder the tensor for the TiledAtom
    auto t_tile = make_tile(get<0>(permutation_mnk),
                            get<2>(permutation_mnk));
    auto t_tensor = logical_divide(sfatensor, t_tile);                                                 // (PermM,PermK)
```
**EN:** The surrounding comments explain the local purpose of this block: Reorder the tensor for the TiledAtom.
**CN:** 周围注释解释了这一段的局部作用：Reorder the tensor for the TiledAtom。

### Lines 564-567

```cpp
    // Tile the tensor for the Atom
    auto a_tile = make_tile(make_layout(size<0>(AtomShape_MNK{})),
                            make_layout(size<2>(AtomShape_MNK{})));
    auto a_tensor = zipped_divide(t_tensor, a_tile);                                   // ((AtomM,AtomK),(RestM,RestK))
```
**EN:** The surrounding comments explain the local purpose of this block: Tile the tensor for the Atom.
**CN:** 周围注释解释了这一段的局部作用：Tile the tensor for the Atom。

### Lines 569-570

```cpp
    // Transform the Atom mode from (M,K) to (Thr,Val)
    auto tv_tensor = a_tensor.compose(AtomLayoutSFA_TV{},_);                             // ((ThrV,FrgV),(RestM,RestK))
```
**EN:** The surrounding comments explain the local purpose of this block: Transform the Atom mode from (M,K) to (Thr,Val).
**CN:** 周围注释解释了这一段的局部作用：Transform the Atom mode from (M,K) to (Thr,Val)。

### Lines 572-576

```cpp
    // Tile the tensor for the Thread
    auto thr_tile = make_tile(_,
                              make_tile(make_layout(size<1>(thr_layout_vmnk)),
                                        make_layout(size<3>(thr_layout_vmnk))));
    auto thr_tensor = zipped_divide(tv_tensor, thr_tile);                  // ((ThrV,(ThrM,ThrK)),(FrgV,(RestM,RestK)))
```
**EN:** The surrounding comments explain the local purpose of this block: Tile the tensor for the Thread.
**CN:** 周围注释解释了这一段的局部作用：Tile the tensor for the Thread。

### Lines 578-579

```cpp
    return thr_tensor;
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 581-586

```cpp
  template <class SFBTensor, class Atom, class TiledThr, class TiledPerm>
  CUTE_HOST_DEVICE constexpr
  auto
  thrfrg_SFB(SFBTensor&& sfbtensor, TiledMMA<Atom, TiledThr, TiledPerm>& mma)
  {
    CUTE_STATIC_ASSERT_V(rank(sfbtensor) >= Int<2>{});
```
**EN:** This block introduces `SFBTensor` and groups related declarations around that symbol. The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这一段引入了 `SFBTensor`，并围绕该符号组织相关声明。 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 588-589

```cpp
    using AtomShape_MNK  = typename Atom::Shape_MNK;
    using AtomLayoutSFB_TV = typename Atom::Traits::SFBLayout;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 591-592

```cpp
    auto permutation_mnk = TiledPerm{};
    auto thr_layout_vmnk = mma.get_thr_layout_vmnk();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 594-597

```cpp
    // Reorder the tensor for the TiledAtom
    auto t_tile = make_tile(get<1>(permutation_mnk),
                            get<2>(permutation_mnk));
    auto t_tensor = logical_divide(sfbtensor, t_tile);                                                 // (PermN,PermK)
```
**EN:** The surrounding comments explain the local purpose of this block: Reorder the tensor for the TiledAtom.
**CN:** 周围注释解释了这一段的局部作用：Reorder the tensor for the TiledAtom。

### Lines 599-602

```cpp
    // Tile the tensor for the Atom
    auto a_tile = make_tile(make_layout(size<1>(AtomShape_MNK{})),
                            make_layout(size<2>(AtomShape_MNK{})));
    auto a_tensor = zipped_divide(t_tensor, a_tile);                                   // ((AtomN,AtomK),(RestN,RestK))
```
**EN:** The surrounding comments explain the local purpose of this block: Tile the tensor for the Atom.
**CN:** 周围注释解释了这一段的局部作用：Tile the tensor for the Atom。

### Lines 604-605

```cpp
    // Transform the Atom mode from (M,K) to (Thr,Val)
    auto tv_tensor = a_tensor.compose(AtomLayoutSFB_TV{},_);                             // ((ThrV,FrgV),(RestN,RestK))
```
**EN:** The surrounding comments explain the local purpose of this block: Transform the Atom mode from (M,K) to (Thr,Val).
**CN:** 周围注释解释了这一段的局部作用：Transform the Atom mode from (M,K) to (Thr,Val)。

### Lines 607-613

```cpp
    // Tile the tensor for the Thread
    auto thr_tile = make_tile(_,
                              make_tile(make_layout(size<2>(thr_layout_vmnk)),
                                        make_layout(size<3>(thr_layout_vmnk))));
    auto thr_tensor = zipped_divide(tv_tensor, thr_tile);                  // ((ThrV,(ThrN,ThrK)),(FrgV,(RestN,RestK)))
    return thr_tensor;
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 615-626

```cpp
  template <class SFATensor, class ThrMma>
  CUTE_HOST_DEVICE constexpr
  auto
  partition_fragment_SFA(SFATensor&& sfatensor, ThrMma& thread_mma)
  {
    using ValTypeSF = typename ThrMma::Atom::Traits::ValTypeSF;
    auto thr_tensor = make_tensor(static_cast<SFATensor&&>(sfatensor).data(), thrfrg_SFA(sfatensor.layout(),thread_mma));
    auto thr_vmnk = thread_mma.thr_vmnk_;
    auto thr_vmk = make_coord(get<0>(thr_vmnk), make_coord(get<1>(thr_vmnk), get<3>(thr_vmnk)));
    auto partition_SFA =  thr_tensor(thr_vmk, make_coord(_, repeat<rank<1,1>(thr_tensor)>(_)));
    return make_fragment_like<ValTypeSF>(partition_SFA);
  }
```
**EN:** This block introduces `SFATensor` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `SFATensor`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 628-639

```cpp
  template <class SFBTensor, class ThrMma>
  CUTE_HOST_DEVICE constexpr
  auto
  partition_fragment_SFB(SFBTensor&& sfbtensor, ThrMma& thread_mma)
  {
    using ValTypeSF = typename ThrMma::Atom::Traits::ValTypeSF;
    auto thr_tensor = make_tensor(static_cast<SFBTensor&&>(sfbtensor).data(), thrfrg_SFB(sfbtensor.layout(),thread_mma));
    auto thr_vmnk = thread_mma.thr_vmnk_;
    auto thr_vnk = make_coord(get<0>(thr_vmnk), make_coord(get<2>(thr_vmnk), get<3>(thr_vmnk)));
    auto partition_SFB =  thr_tensor(thr_vnk, make_coord(_, repeat<rank<1,1>(thr_tensor)>(_)));
    return make_fragment_like<ValTypeSF>(partition_SFB);
  }
```
**EN:** This block introduces `SFBTensor` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `SFBTensor`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 641-649

```cpp
  template<class TiledMma>
  CUTE_HOST_DEVICE constexpr
  auto
  get_layoutSFA_TV(TiledMma& mma)
  {
    // (M,K) -> (M,K)
    auto tile_shape_mnk = tile_shape(mma);
    auto ref_A = make_layout(make_shape(size<0>(tile_shape_mnk), size<2>(tile_shape_mnk)));
    auto thr_layout_vmnk = mma.get_thr_layout_vmnk();
```
**EN:** This block introduces `TiledMma` and groups related declarations around that symbol.
**CN:** 这一段引入了 `TiledMma`，并围绕该符号组织相关声明。

### Lines 651-655

```cpp
    // (ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK))
    auto atile = make_tile(_,
                          make_tile(make_layout(make_shape (size<1>(thr_layout_vmnk), size<2>(thr_layout_vmnk)),
                                                make_stride(               Int<1>{} ,                Int<0>{} )),
                                    _));
```
**EN:** The surrounding comments explain the local purpose of this block: (ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK)).
**CN:** 周围注释解释了这一段的局部作用：(ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK))。

### Lines 657-661

```cpp
    // thr_idx -> (ThrV,ThrM,ThrN,ThrK)
    auto thridx_2_thrid = right_inverse(thr_layout_vmnk);
    // (thr_idx,val) -> (M,K)
    return thrfrg_SFA(ref_A, mma).compose(atile, _).compose(thridx_2_thrid, _);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 663-671

```cpp
  template<class TiledMma>
  CUTE_HOST_DEVICE constexpr
  auto
  get_layoutSFB_TV(TiledMma& mma)
  {
    // (N,K) -> (N,K)
    auto tile_shape_mnk = tile_shape(mma);
    auto ref_B = make_layout(make_shape(size<1>(tile_shape_mnk), size<2>(tile_shape_mnk)));
    auto thr_layout_vmnk = mma.get_thr_layout_vmnk();
```
**EN:** This block introduces `TiledMma` and groups related declarations around that symbol.
**CN:** 这一段引入了 `TiledMma`，并围绕该符号组织相关声明。

### Lines 673-677

```cpp
    // (ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK))
    auto btile = make_tile(_,
                          make_tile(make_layout(make_shape (size<1>(thr_layout_vmnk), size<2>(thr_layout_vmnk)),
                                                make_stride(               Int<0>{} ,                Int<1>{} )),
                                    _));
```
**EN:** The surrounding comments explain the local purpose of this block: (ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK)).
**CN:** 周围注释解释了这一段的局部作用：(ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK))。

### Lines 679-683

```cpp
    // thr_idx -> (ThrV,ThrM,ThrN,ThrK)
    auto thridx_2_thrid = right_inverse(thr_layout_vmnk);
    // (thr_idx,val) -> (M,K)
    return thrfrg_SFB(ref_B, mma).compose(btile, _).compose(thridx_2_thrid, _);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 685-696

```cpp
  /// Set up the data needed by this collective for load and mma.
  /// Returns a tuple of tensors. The collective and the kernel layer have the contract
  /// Returned tuple must contain at least two elements, with the first two elements being:
  /// gA_mkl - The tma tensor, A after a local tile so it has shape  (BLK_M,BLK_K,m,k,l)
  /// gB_nkl - The tma tensor, B after a local tile so it has shape  (BLK_N,BLK_K,n,k,l)
  /// The rest of the tensors can be specified as needed by this collective.
  template <class ProblemShape_MNKL>
  CUTLASS_DEVICE auto
  load_init(ProblemShape_MNKL const& problem_shape_MNKL, Params const& mainloop_params) const {
    using X = Underscore;
    // Separate out problem shape for convenience
    auto [M, N, K, L] = problem_shape_MNKL;
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 698-718

```cpp
    // TMA requires special handling of strides to deal with coord codomain mapping
    // Represent the full tensors -- get these from TMA
    Tensor mA_mkl = mainloop_params.tma_load_a.get_tma_tensor(mainloop_params.layout_a.shape());             // (m,k,l)
    Tensor mB_nkl = mainloop_params.tma_load_b.get_tma_tensor(make_shape(N,K,L));                            // (n,k,l)
    Tensor mE_mkl = mainloop_params.tma_load_e.get_tma_tensor(mainloop_params.layout_e.shape());             // (m,k,l)
    Tensor mSFA_mkl = mainloop_params.tma_load_sfa.get_tma_tensor(shape(mainloop_params.layout_SFA));
    auto mSFB_nkl = [=](){
      if constexpr (IsCtaN64) {
        Tensor mSFB_tmp = mainloop_params.tma_load_sfb.get_tma_tensor(shape(mainloop_params.layout_SFB));
        auto x = stride<0,1>(mSFB_tmp);
        auto y = ceil_div(shape<0,1>(mSFB_tmp), _2{});
        auto  new_shape =  make_shape (make_shape( shape<0,0>(mSFB_tmp),
                                       make_shape( make_shape(_2{}),   y)),  shape<1>(mSFB_tmp), shape<2>(mSFB_tmp));
        auto new_stride = make_stride(make_stride(stride<0,0>(mSFB_tmp),
                                      make_stride(make_stride(_0{}),   x)), stride<1>(mSFB_tmp), stride<2>(mSFB_tmp));
        return make_tensor(mSFB_tmp.data(), make_layout(new_shape, new_stride));
      }
      else {
        return mainloop_params.tma_load_sfb.get_tma_tensor(shape(mainloop_params.layout_SFB));
      }
    }();
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 720-727

```cpp
    // Make tiled views, defer the slice
    Tensor gA_mkl = local_tile(mA_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});        // ( BLK_M, BLK_K,m,k,l)
    Tensor gB_nkl = local_tile(mB_nkl, TileShapeB{}, make_coord(_,_,_), Step< X,_1,_1>{});       // ( BLK_N, BLK_K,n,k,l)
    Tensor gE_mkl = local_tile(mE_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});        // ( BLK_N, BLK_K,n,k,l)
    Tensor gSFA_mkl = local_tile(mSFA_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});    // (TILE_M,TILE_K,m,k,l)
    Tensor gSFB_nkl = local_tile(mSFB_nkl, TileShapeSF{}, make_coord(_,_,_), Step< X,_1,_1>{});  // (TILE_N,TILE_K,n,k,l)
    return cute::make_tuple(gA_mkl, gB_nkl, gE_mkl, gSFA_mkl, gSFB_nkl);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 729-733

```cpp
  /// Perform a Producer Epilogue to prevent early exit of blocks in a Cluster
  template<class MainloopPipeline, class PipelineState>
  CUTLASS_DEVICE void
  load_tail(MainloopPipeline pipeline, PipelineState smem_pipe_write) {
    int lane_predicate = cute::elect_one_sync();
```
**EN:** This block introduces `MainloopPipeline` and groups related declarations around that symbol. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段引入了 `MainloopPipeline`，并围绕该符号组织相关声明。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 735-745

```cpp
    // Issue the epilogue waits
    if (lane_predicate) {
      /* This helps avoid early exit of blocks in Cluster
       * Waits for all stages to either be released (all
       * Consumer UNLOCKs), or if the stage was never used
       * then would just be acquired since the phase was
       * still inverted from make_producer_start_state
       */
      pipeline.producer_tail(smem_pipe_write);
    }
  }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 747-763

```cpp
  // Issues loads for A/E/SF only (used when DMA warp is split).
  template <
    class TensorA, class TensorB, class TensorE,
    class TensorSFA, class TensorSFB,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  load_MK(
      Params const& params,
      MainloopPipelineMK pipeline,
      PipelineStateMK smem_pipe_write,
      cute::tuple<TensorA, TensorB, TensorE, TensorSFA, TensorSFB> const& load_inputs,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
```
**EN:** This block introduces `TensorA` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TensorA`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 765-768

```cpp
    Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.begin()), SmemLayoutA{});         // (BLK_M,BLK_K,PIPE)
    Tensor sE = make_tensor(make_smem_ptr(shared_tensors.smem_E.begin()), SmemLayoutE{});         // (BLK_M,BLK_K,PIPE)
    Tensor sSFA = make_tensor(make_smem_ptr(shared_tensors.smem_SFA.begin()), SmemLayoutSFA{});   // (BLK_M,BLK_K,PIPE)
    Tensor sSFB = make_tensor(make_smem_ptr(shared_tensors.smem_SFB.begin()), SmemLayoutSFB{});   // (BLK_N,BLK_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 770-772

```cpp
    //
    // Prepare the TMA loads for A and E
    //
```
**EN:** This comment-only block labels or explains the following section:  Prepare the TMA loads for A and E .
**CN:** 这个纯注释块用于标记或解释后续区域： Prepare the TMA loads for A and E 。

### Lines 774-777

```cpp
    Tensor gA_mkl = get<0>(load_inputs);                                                             // (BLK_M,BLK_K,k)
    Tensor gE_mkl = get<2>(load_inputs);                                                             // (BLK_M,BLK_K,k)
    Tensor gSFA_mkl = get<3>(load_inputs);                                                           // (BLK_M,BLK_K,k)
    Tensor gSFB_nkl = get<4>(load_inputs);                                                           // (BLK_N,BLK_K,k)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 779-782

```cpp
    auto block_tma_a = params.tma_load_a.get_slice(0);
    auto block_tma_e = params.tma_load_e.get_slice(0);
    auto block_tma_sfa = params.tma_load_sfa.get_slice(0);
    auto block_tma_sfb = params.tma_load_sfb.get_slice(0);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 784-789

```cpp
    // Partition the inputs based on the current block coordinates.
    auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
    Tensor gA = gA_mkl(_,_,m_coord,_,l_coord);                                                       // (BLK_M,BLK_K,k)
    Tensor gE = gE_mkl(_,_,m_coord,_,l_coord);                                                       // (BLK_M,BLK_K,k)
    Tensor gSFA = gSFA_mkl(_,_,m_coord,_,l_coord);                                                   // (BLK_M,BLK_K,k)
    Tensor gSFB = gSFB_nkl(_,_,n_coord,_,l_coord);                                                   // (BLK_N,BLK_K,k)
```
**EN:** The surrounding comments explain the local purpose of this block: Partition the inputs based on the current block coordinates..
**CN:** 周围注释解释了这一段的局部作用：Partition the inputs based on the current block coordinates.。

### Lines 791-793

```cpp
    // Partition source and destination tensors for tma copies
    Tensor tAgA = block_tma_a.partition_S(gA);                                                // (TMA,TMA_M,TMA_K,   k)
    Tensor tAsA = block_tma_a.partition_D(sA);                                                // (TMA,TMA_M,TMA_K,PIPE)
```
**EN:** The surrounding comments explain the local purpose of this block: Partition source and destination tensors for tma copies.
**CN:** 周围注释解释了这一段的局部作用：Partition source and destination tensors for tma copies。

### Lines 795-796

```cpp
    Tensor tEgE = block_tma_e.partition_S(gE);                                                // (TMA,TMA_M,TMA_K,   k)
    Tensor tEsE = block_tma_e.partition_D(sE);                                                // (TMA,TMA_M,TMA_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 798-801

```cpp
    Tensor tAgSFA = block_tma_sfa.partition_S(gSFA);                                          // (TMA,TMA_M,TMA_K,   k)
    Tensor tAsSFA = block_tma_sfa.partition_D(sSFA);                                          // (TMA,TMA_M,TMA_K,PIPE)
    Tensor tBgSFB = block_tma_sfb.partition_S(gSFB);                                          // (TMA,TMA_N,TMA_K,   k)
    Tensor tBsSFB = block_tma_sfb.partition_D(sSFB);                                          // (TMA,TMA_N,TMA_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 803-807

```cpp
    // Mainloop
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count) {
      // LOCK smem_pipe_write for _writing_
      pipeline.producer_acquire(smem_pipe_write);
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 809-813

```cpp
      //
      // Copy gmem to smem for *k_tile_iter
      //
      using BarrierType = typename MainloopPipelineMK::ProducerBarrierType;
      BarrierType* tma_barrier = pipeline.producer_get_barrier(smem_pipe_write);
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 815-823

```cpp
      int write_stage = smem_pipe_write.index();
      if (cute::elect_one_sync()) {
        copy(params.tma_load_a.with(*tma_barrier), tAgA(_,_,_,*k_tile_iter), tAsA(_,_,_,write_stage));
        copy(params.tma_load_sfa.with(*tma_barrier), tAgSFA(_,_,_,*k_tile_iter), tAsSFA(_,_,_,write_stage));
        copy(params.tma_load_sfb.with(*tma_barrier), tBgSFB(_,_,_,*k_tile_iter), tBsSFB(_,_,_,write_stage));
        if constexpr (UseSmemE) {
          copy(params.tma_load_e.with(*tma_barrier), tEgE(_,_,_,*k_tile_iter), tEsE(_,_,_,write_stage));
        }
      }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 825-829

```cpp
      if constexpr (!UseSmemE) {
        // Prefetch 1 stage of E data to L2 in advance
        auto blk_coord_mkl = make_coord(get<0>(blk_coord), *k_tile_iter, get<3>(blk_coord));         // (BLK_M,BLK_K,L)
        prefetch(make_local_E(params, blk_coord_mkl));
      }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 831-835

```cpp
      // Advance smem_pipe_write
      ++k_tile_iter;
      ++smem_pipe_write;
    }
  }
```
**EN:** The surrounding comments explain the local purpose of this block: Advance smem_pipe_write.
**CN:** 周围注释解释了这一段的局部作用：Advance smem_pipe_write。

### Lines 837-853

```cpp
  // Issues loads for B/SF only (used when DMA warp is split).
  template <
    class TensorA, class TensorB, class TensorE,
    class TensorSFA, class TensorSFB,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  load_NK(
      Params const& params,
      MainloopPipelineNK pipeline,
      PipelineStateNK smem_pipe_write,
      cute::tuple<TensorA, TensorB, TensorE, TensorSFA, TensorSFB> const& load_inputs,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
```
**EN:** This block introduces `TensorA` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TensorA`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 855-855

```cpp
    Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.begin()), SmemLayoutB{});         // (BLK_N,BLK_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 857-859

```cpp
    //
    // Prepare the TMA loads for B
    //
```
**EN:** This comment-only block labels or explains the following section:  Prepare the TMA loads for B .
**CN:** 这个纯注释块用于标记或解释后续区域： Prepare the TMA loads for B 。

### Lines 861-862

```cpp
    Tensor gB_nkl = get<1>(load_inputs);
    auto block_tma_b = params.tma_load_b.get_slice(0);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 864-866

```cpp
    // Partition the inputs based on the current block coordinates.
    auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
    Tensor gB =   gB_nkl(_,_,n_coord,_,l_coord);                                                     // (BLK_N,BLK_K,k)
```
**EN:** The surrounding comments explain the local purpose of this block: Partition the inputs based on the current block coordinates..
**CN:** 周围注释解释了这一段的局部作用：Partition the inputs based on the current block coordinates.。

### Lines 868-870

```cpp
    // Partition source and destination tensors for tma copies
    Tensor tBgB = block_tma_b.partition_S(gB);                                                // (TMA,TMA_N,TMA_K,   k)
    Tensor tBsB = block_tma_b.partition_D(sB);                                                // (TMA,TMA_N,TMA_K,PIPE)
```
**EN:** The surrounding comments explain the local purpose of this block: Partition source and destination tensors for tma copies.
**CN:** 周围注释解释了这一段的局部作用：Partition source and destination tensors for tma copies。

### Lines 872-876

```cpp
    // Mainloop
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count) {
      // LOCK smem_pipe_write for _writing_
      pipeline.producer_acquire(smem_pipe_write);
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 878-882

```cpp
      //
      // Copy gmem to smem for *k_tile_iter
      //
      using BarrierType = typename MainloopPipelineNK::ProducerBarrierType;
      BarrierType* tma_barrier = pipeline.producer_get_barrier(smem_pipe_write);
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 884-892

```cpp
      int write_stage = smem_pipe_write.index();
      if (cute::elect_one_sync()) {
        copy(params.tma_load_b.with(*tma_barrier), tBgB(_,_,_,*k_tile_iter), tBsB(_,_,_,write_stage));
      }
      // Advance smem_pipe_write
      ++k_tile_iter;
      ++smem_pipe_write;
    }
  }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 894-902

```cpp
  // Local tile E from global memory.
  template<class BlockCoord>
  CUTLASS_DEVICE auto
  make_local_E(Params const& mainloop_params,
               BlockCoord const& blk_coord) {
    // E layout
    auto layoutE = mainloop_params.layout_e;
    // E data pointer as sparse datatype
    auto ptr_E = recast_ptr<ElementEMma>(mainloop_params.ptr_E);
```
**EN:** This block introduces `BlockCoord` and groups related declarations around that symbol.
**CN:** 这一段引入了 `BlockCoord`，并围绕该符号组织相关声明。

### Lines 904-908

```cpp
    // Global gmem E
    Tensor gE = make_tensor(make_gmem_ptr(ptr_E), layoutE);                                      // (BLK_M,BLK_K,BLK_L)
    // Local tile E
    return local_tile(gE, select<0,2>(TileShape{}), blk_coord);                                        // (BLK_M,BLK_K)
  }
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 910-920

```cpp
  // Load E from global memory to registers.
  template<bool IsF8F6F4, class BlockCoord, class ProblemShape_MNKL>
  CUTLASS_DEVICE auto
  load_E(Params const& mainloop_params,
         BlockCoord const& blk_coord,
         ProblemShape_MNKL const& problem_shape_MNKL,
         int thread_idx) {
    // Workload
    auto [M, N, K, L] = problem_shape_MNKL;
    auto [m_coord, k_coord, l_coord] = blk_coord;
    auto Shape_MK = cute::make_tuple(M, K);
```
**EN:** This block introduces `BlockCoord` and groups related declarations around that symbol.
**CN:** 这一段引入了 `BlockCoord`，并围绕该符号组织相关声明。

### Lines 922-941

```cpp
    // Tiled mma and thread mma
    TiledMma tiled_mma;
    auto thread_mma = tiled_mma.get_thread_slice(thread_idx);
    // Tile shape
    auto tile_shape_mnk = tile_shape(tiled_mma);
    // Re-sue copy atom E from SmemCopyAtomE
    using GmemCopyAtomeE = SmemCopyAtomE;
    // Gmem tile copy
    auto gmem_tiled_copy_E = make_tiled_copy_impl(GmemCopyAtomeE{},
                                                  get_layoutE_TV(tiled_mma),
                                                  make_shape(size<0>(tile_shape_mnk), size<2>(tile_shape_mnk)));
    // Gmem thread copy
    auto gmem_thr_copy_E = gmem_tiled_copy_E.get_thread_slice(thread_idx);
    // Gmem local E
    auto gE_mkl = make_local_E(mainloop_params, blk_coord);
    // Tiled gmem E
    Tensor tCgE = gmem_thr_copy_E.partition_S(gE_mkl);                                             // (CPY,CPY_M,CPY_K)
    // Tiled register E and copy view
    Tensor tCrE = partition_fragment_E(gE_mkl, thread_mma);                                        // (MMA,MMA_M,MMA_K)
    Tensor tCrE_copy_view = gmem_thr_copy_E.retile_D(tCrE);                                        // (CPY,CPY_M,CPY_K)
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 943-954

```cpp
    if constexpr (IsF8F6F4) {
      auto get_copy_atom_and_common_vec = [&]() {
        using ValType = typename decltype(tCrE)::value_type;
        // Get maximum copy vector size (logically)
        auto common_layout = max_common_layout(tCgE, tCrE);
        auto vec_elem = cute::min(size(common_layout), Int<128 / sizeof_bits_v<ValType>>{});
        auto common_vec = composition(common_layout, vec_elem);
        // Compose a Copy_Atom
        using VecType = uint_bit_t<vec_elem * sizeof_bits_v<ValType>>;
        using cpy = Copy_Atom<UniversalCopy<VecType>, ValType>;
        return cute::make_tuple(cpy{}, common_vec);
      };
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 956-976

```cpp
      // Copy depends on whether predication is needed
      if constexpr (IsELoadPred) {
        // Get predication based on logical element coordinates.
        Tensor cE_mk = local_tile(
                make_identity_tensor(Shape_MK),
                make_shape(get<0>(TileShape{}), get<2>(TileShape{})),
                make_shape(m_coord, k_coord));                                                          // (BLK_M, BLK_K)
        Tensor tCcE = gmem_thr_copy_E.partition_S(cE_mk);                                            // (CPY,CPY_M,CPY_K)
        auto [atom, vec] = get_copy_atom_and_common_vec();
        // Coordinate comparison for out of bound (OOB) predication
        Tensor tZpE = cute::lazy::transform(zipped_divide(tCcE, vec), [&](auto const& c){ return cute::elem_less(c, Shape_MK); });
        // Copy
        cute::copy_if(atom, tZpE, zipped_divide(tCgE, vec), zipped_divide(tCrE_copy_view, vec));
      }
      else {
        // Copy
        cute::copy(cute::AutoVectorizingCopyWithAssumedAlignment<32>{}, tCgE, tCrE_copy_view);
      }
    }
    return tCrE;
  }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 978-999

```cpp
  /// Perform a collective-scoped matrix multiply-accumulate
  /// Consumer Perspective
  template <
    class FrgTensorC,
    class KTileIterator,
    class CtaTileCoord,
    class ProblemShape_MNKL
  >
  CUTLASS_DEVICE void
  mma(MainloopPipelineMK pipeline_mk,
      PipelineStateMK smem_pipe_read_mk,
      MainloopPipelineNK pipeline_nk,
      PipelineStateNK smem_pipe_read_nk,
      FrgTensorC& accum,
      KTileIterator k_tile_iter,
      int k_tile_count,
      int thread_idx,
      TensorStorage& shared_tensors,
      Params const& mainloop_params,
      CtaTileCoord const& cta_tile_coord,
      ProblemShape_MNKL const& problem_shape_MNKL) {
    using namespace cute;
```
**EN:** This statement brings CuTe symbols into the local scope, which keeps the template-heavy code shorter and easier to read.
**CN:** 这条语句把 CuTe 符号引入局部作用域，从而让大量模板代码更短、更易读。

### Lines 1001-1001

```cpp
    CUTE_STATIC_ASSERT(is_rmem<FrgTensorC>::value, "C tensor must be rmem resident.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 1003-1003

```cpp
    clear(accum);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1005-1023

```cpp
    Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.begin()), SmemLayoutA{});         // (BLK_M,BLK_K,PIPE)
    Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.begin()), SmemLayoutB{});         // (BLK_N,BLK_K,PIPE)
    Tensor sE = make_tensor(make_smem_ptr(shared_tensors.smem_E.begin()), SmemLayoutE{});         // (BLK_M,BLK_K,PIPE)
    Tensor sSFA = make_tensor(make_smem_ptr(shared_tensors.smem_SFA.begin()), SmemLayoutSFA{});   // (BLK_M,BLK_K,PIPE)
    auto SmemLayoutSFB_Ld = [SLayoutSFB = SmemLayoutSFB{}]() {
      if constexpr (IsCtaN64) {
        auto SLayoutSFB_tmp = SLayoutSFB;
        auto  new_shape =  make_shape (make_shape(make_shape(shape<0,0,0>(SLayoutSFB_tmp),
                                    shape<0,0,1>(SLayoutSFB_tmp) / _2{}), shape<0,1>(SLayoutSFB_tmp)),
                                    shape<1>(SLayoutSFB_tmp), shape<2>(SLayoutSFB_tmp));
        auto new_stride = stride(SLayoutSFB_tmp);
        return make_layout(new_shape, new_stride);
      }
      else {
        return SLayoutSFB;
      }
    }();
    Tensor sSFB = make_tensor(make_smem_ptr(shared_tensors.smem_SFB.begin()) +
                (IsCtaN64 && get<1>(cta_tile_coord) % 2 == 1 ? 8 : 0), SmemLayoutSFB_Ld);         // (BLK_N,BLK_K,PIPE)
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1025-1027

```cpp
    //
    // Define A/B/E partitioning
    //
```
**EN:** This comment-only block labels or explains the following section:  Define A/B/E partitioning .
**CN:** 这个纯注释块用于标记或解释后续区域： Define A/B/E partitioning 。

### Lines 1029-1030

```cpp
    TiledMma tiled_mma;
    auto thread_mma = tiled_mma.get_thread_slice(thread_idx);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1032-1037

```cpp
    // Allocate fragments and descriptors
    Tensor tCrA = thread_mma.partition_fragment_A(sA(_,_,Int<0>{}));                               // (MMA,MMA_M,MMA_K)
    Tensor tCrB = thread_mma.partition_fragment_B(sB(_,_,Int<0>{}));                               // (MMA,MMA_N,MMA_K)
    Tensor tCrE = partition_fragment_E(sE(_,_,Int<0>{}), thread_mma);                              // (MMA,MMA_M,MMA_K)
    Tensor tCrSFA = partition_fragment_SFA(sSFA(_,_,Int<0>{}), thread_mma);                        // (MMA,MMA_M,MMA_K)
    Tensor tCrSFB = partition_fragment_SFB(sSFB(_,_,Int<0>{}), thread_mma);                        // (MMA,MMA_N,MMA_K)
```
**EN:** The surrounding comments explain the local purpose of this block: Allocate fragments and descriptors.
**CN:** 周围注释解释了这一段的局部作用：Allocate fragments and descriptors。

### Lines 1039-1046

```cpp
    //
    // Copy Atom A, B and E retiling
    //
    auto smem_tiled_copy_A = make_tiled_copy_A(SmemCopyAtomA{}, tiled_mma);
    auto smem_thr_copy_A   = smem_tiled_copy_A.get_thread_slice(thread_idx);
    Tensor tCsA            = smem_thr_copy_A.partition_S(
          as_position_independent_swizzle_tensor(sA));                                        // (CPY,CPY_M,CPY_K,PIPE)
    Tensor tCrA_copy_view  = smem_thr_copy_A.retile_D(tCrA);                                  //      (CPY,CPY_M,CPY_K)
```
**EN:** The surrounding comments explain the local purpose of this block:  Copy Atom A, B and E retiling .
**CN:** 周围注释解释了这一段的局部作用： Copy Atom A, B and E retiling 。

### Lines 1048-1052

```cpp
    auto smem_tiled_copy_B = make_tiled_copy_B(SmemCopyAtomB{}, tiled_mma);
    auto smem_thr_copy_B   = smem_tiled_copy_B.get_thread_slice(thread_idx);
    Tensor tCsB            = smem_thr_copy_B.partition_S(
         as_position_independent_swizzle_tensor(sB));                                         // (CPY,CPY_N,CPY_K,PIPE)
    Tensor tCrB_copy_view  = smem_thr_copy_B.retile_D(tCrB);                                  //      (CPY,CPY_N,CPY_K)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1054-1061

```cpp
    auto tile_shape_mnk    = tile_shape(tiled_mma);
    auto smem_tiled_copy_E = make_tiled_copy_impl(SmemCopyAtomE{},
                                                  get_layoutE_TV(tiled_mma),
                                                  make_shape(size<0>(tile_shape_mnk), size<2>(tile_shape_mnk)));
    auto smem_thr_copy_E   = smem_tiled_copy_E.get_thread_slice(thread_idx);
    Tensor tCsE            = smem_thr_copy_E.partition_S(
                                  as_position_independent_swizzle_tensor(sE));                // (CPY,CPY_M,CPY_K,PIPE)
    Tensor tCrE_copy_view  = smem_thr_copy_E.retile_D(tCrE);                                  //      (CPY,CPY_M,CPY_K)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1063-1071

```cpp
    // SFA
    auto smem_tiled_copy_SFA = make_tiled_copy_impl(SmemCopyAtomSFA{},
                                                    get_layoutSFA_TV(tiled_mma),
                                                    make_shape(size<0>(tile_shape_mnk), size<2>(tile_shape_mnk))
                                                  );
    auto smem_thr_copy_SFA   = smem_tiled_copy_SFA.get_thread_slice(thread_idx);
    Tensor tCsSFA            = smem_thr_copy_SFA.partition_S(
        as_position_independent_swizzle_tensor(sSFA));                                        // (CPY,CPY_M,CPY_K,PIPE)
    Tensor tCrSFA_copy_view  = smem_thr_copy_SFA.retile_D(tCrSFA);                            //      (CPY,CPY_M,CPY_K)
```
**EN:** The surrounding comments explain the local purpose of this block: SFA.
**CN:** 周围注释解释了这一段的局部作用：SFA。

### Lines 1073-1081

```cpp
    // SFB
    auto smem_tiled_copy_SFB = make_tiled_copy_impl(SmemCopyAtomSFB{},
                                                    get_layoutSFB_TV(tiled_mma),
                                                    make_shape(size<1>(tile_shape_mnk), size<2>(tile_shape_mnk))
                                                  );
    auto smem_thr_copy_SFB   = smem_tiled_copy_SFB.get_thread_slice(thread_idx);
    Tensor tCsSFB            = smem_thr_copy_SFB.partition_S(
      as_position_independent_swizzle_tensor(sSFB));                                          // (CPY,CPY_N,CPY_K,PIPE)
    Tensor tCrSFB_copy_view  = smem_thr_copy_SFB.retile_D(tCrSFB);                            //      (CPY,CPY_N,CPY_K)
```
**EN:** The surrounding comments explain the local purpose of this block: SFB.
**CN:** 周围注释解释了这一段的局部作用：SFB。

### Lines 1083-1092

```cpp
    CUTE_STATIC_ASSERT_V(size<1>(tCsA) == size<1>(tCrA_copy_view));
    CUTE_STATIC_ASSERT_V(size<2>(tCsA) == size<2>(tCrA_copy_view));
    CUTE_STATIC_ASSERT_V(size<1>(tCsE) == size<1>(tCrE_copy_view));
    CUTE_STATIC_ASSERT_V(size<1>(tCrA) == size<1>(accum));
    CUTE_STATIC_ASSERT_V(size<1>(tCrB) == size<2>(accum));
    CUTE_STATIC_ASSERT_V(size<2>(tCsA) == size<2>(tCsB) * Int<AsymmetricKRatio>{});
    CUTE_STATIC_ASSERT_V(size<3>(tCsA) == Int<DispatchPolicy::StagesA>{});
    CUTE_STATIC_ASSERT_V(size<3>(tCsB) == Int<DispatchPolicy::StagesB>{});
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::StagesA>{} == size<2>(sA));
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::StagesB>{} == size<2>(sB));
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 1094-1102

```cpp
    CUTE_STATIC_ASSERT_V(size<1>(tCsSFA) == size<1>(tCrSFA_copy_view));                       // CPY_M
    CUTE_STATIC_ASSERT_V(size<2>(tCsSFA) == size<2>(tCrSFA_copy_view));                       // CPY_K
    CUTE_STATIC_ASSERT_V(size<1>(tCrSFA) == size<1>(accum));                                  // MMA_M
    CUTE_STATIC_ASSERT_V(size<1>(tCrSFB) == size<2>(accum));                                  // MMA_N
    CUTE_STATIC_ASSERT_V(size<2>(tCsSFA) == size<2>(tCsSFB));                                 // CPY_K
    CUTE_STATIC_ASSERT_V(size<3>(tCsSFA) == size<3>(tCsSFB));                                 // PIPE
    CUTE_STATIC_ASSERT_V(size<2>(sA)     == size<2>(sSFA));                                   // PIPE
    CUTE_STATIC_ASSERT_V(size<2>(sSFB)   == Int<DispatchPolicy::StagesA>{});                  // PIPE
    CUTE_STATIC_ASSERT_V(size<2>(sB)     == Int<DispatchPolicy::StagesB>{});                  // PIPE
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 1104-1106

```cpp
    if constexpr (UseSmemE) {
      CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::StagesA>{} == size<2>(sE));
    }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1108-1110

```cpp
    //
    // DEFINE FUNCTIONS FOR PIPELINED MAIN LOOP
    //
```
**EN:** This comment-only block labels or explains the following section:  DEFINE FUNCTIONS FOR PIPELINED MAIN LOOP .
**CN:** 这个纯注释块用于标记或解释后续区域： DEFINE FUNCTIONS FOR PIPELINED MAIN LOOP 。

### Lines 1112-1114

```cpp
    // We release buffers to producer warps(dma load) with some mmas in flight
    PipelineStateMK smem_pipe_release_mk = smem_pipe_read_mk;
    PipelineStateNK smem_pipe_release_nk = smem_pipe_read_nk;
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1116-1120

```cpp
    // Wait consumer barrier MK
    auto wait_barrier_mk = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      auto barrier_token_mk = pipeline_mk.consumer_try_wait(smem_pipe_read_mk);
      pipeline_mk.consumer_wait(smem_pipe_read_mk, barrier_token_mk);
    };
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1122-1126

```cpp
    // Wait consumer barrier NK
    auto wait_barrier_nk = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      auto barrier_token_nk = pipeline_nk.consumer_try_wait(smem_pipe_read_nk);
      pipeline_nk.consumer_wait(smem_pipe_read_nk, barrier_token_nk);
    };
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1128-1133

```cpp
    // Release consumer barrier MK, and move forward
    auto release_advance_mk = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      pipeline_mk.consumer_release(smem_pipe_release_mk);
      ++smem_pipe_read_mk;
      ++smem_pipe_release_mk;
    };
```
**EN:** The surrounding comments explain the local purpose of this block: Release consumer barrier MK, and move forward.
**CN:** 周围注释解释了这一段的局部作用：Release consumer barrier MK, and move forward。

### Lines 1135-1140

```cpp
    // Release consumer barrier NK, and move forward
    auto release_advance_nk = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      pipeline_nk.consumer_release(smem_pipe_release_nk);
      ++smem_pipe_read_nk;
      ++smem_pipe_release_nk;
    };
```
**EN:** The surrounding comments explain the local purpose of this block: Release consumer barrier NK, and move forward.
**CN:** 周围注释解释了这一段的局部作用：Release consumer barrier NK, and move forward。

### Lines 1142-1149

```cpp
    // Copy A from SMEM to register, and do transform if needed
    auto copy_transform_A = [&](auto m_block, auto k_block) CUTLASS_LAMBDA_FUNC_INLINE {
      // copy smem->rmem for A operand
      copy(smem_tiled_copy_A, tCsA(_,m_block,k_block,smem_pipe_read_mk.index()), tCrA_copy_view(_,m_block,k_block));
      // Perform transform if needed.
      using MMAOp = typename TiledMma::MMA_Op;
      fp4_shift_A(MMAOp{}, tCrA_copy_view(_,m_block,k_block));
    };
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1151-1158

```cpp
    // Copy B from SMEM to register, and do transform if needed
    auto copy_transform_B = [&](auto n_block, auto k_block) CUTLASS_LAMBDA_FUNC_INLINE {
      // copy smem->rmem for B operand
      copy(smem_tiled_copy_B, tCsB(_,n_block,k_block,smem_pipe_read_nk.index()), tCrB_copy_view(_,n_block,k_block));
      // Perform transform if needed.
      using MMAOp = typename TiledMma::MMA_Op;
      fp4_shift_B(MMAOp{}, tCrB_copy_view(_,n_block,k_block));
    };
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1160-1164

```cpp
    // Copy SFA from SMEM to register
    auto copy_SFA = [&](auto m_block, auto k_block) CUTLASS_LAMBDA_FUNC_INLINE {
      // Copy smem->rmem for SFA operand
      copy(tCsSFA(_,m_block,k_block,smem_pipe_read_mk.index()), tCrSFA_copy_view(_,m_block,k_block));
    };
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1166-1170

```cpp
    // Copy SFB of all Ns from SMEM to register
    auto copy_SFBs = [&](auto k_block) CUTLASS_LAMBDA_FUNC_INLINE {
      // Copy smem->rmem for SFB operand
      copy(tCsSFB(_,_,k_block,smem_pipe_read_mk.index()), tCrSFB_copy_view(_,_,k_block));
    };
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1172-1177

```cpp
    // Copy E from SMEM to register
    auto copy_E = [&](auto m_block, auto k_block) CUTLASS_LAMBDA_FUNC_INLINE {
      // copy smem->rmem for E operand
      copy( recast<RegisterE>(tCsE(_,m_block,k_block,smem_pipe_read_mk.index())),
            recast<RegisterE>(tCrE_copy_view(_,m_block,k_block)));
    };
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 1179-1182

```cpp
    constexpr auto M_BLOCK_MAX = size<1>(tCrA);
    constexpr auto N_BLOCK_MAX = size<1>(tCrB);
    constexpr auto K_BLOCK_MAX = size<2>(tCrA);
    constexpr auto K_BLOCK_STEP = K_BLOCK_MAX / Int<AsymmetricKRatio>{};
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1184-1188

```cpp
    // Perform mainloop gemm, when E is in SMEM.
    auto gemm_loop_with_SmemE = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      // WAIT on smem_pipe_read until data is available
      wait_barrier_mk();
      wait_barrier_nk();
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1190-1195

```cpp
      // Load A/B/E/SFA/SFB, then do gemm.
      for_each(make_int_sequence<K_BLOCK_MAX>{}, [&] (auto k_block) {
        // Copy smem->rmem for A/B/E operand
        copy_transform_A(_, k_block);
        copy_transform_B(_, k_block);
        copy_E(_, k_block);
```
**EN:** The surrounding comments explain the local purpose of this block: Load A/B/E/SFA/SFB, then do gemm. Copy smem->rmem for A/B/E operand.
**CN:** 周围注释解释了这一段的局部作用：Load A/B/E/SFA/SFB, then do gemm. Copy smem->rmem for A/B/E operand。

### Lines 1197-1199

```cpp
        // Copy smem->rmem for SFA/SFB operand
        copy_SFA(_, k_block);
        copy_SFBs(k_block);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for SFA/SFB operand.
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for SFA/SFB operand。

### Lines 1201-1205

```cpp
        // Gemm
        cute::gemm(tiled_mma,
                  make_zip_tensor(tCrA(_,_,k_block), tCrSFA(_,_,k_block), tCrE(_,_,k_block)),
                  make_zip_tensor(tCrB(_,_,k_block), tCrSFB(_,_,k_block)),
                  accum);
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。

### Lines 1207-1207

```cpp
      });
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 1209-1210

```cpp
      cutlass::arch::NamedBarrier::sync(
        thr_size(tiled_mma), cutlass::arch::ReservedNamedBarriers::Sm120MainloopBarrier);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1212-1215

```cpp
      // Advance consumer pipeline mk/nk
      release_advance_mk();
      release_advance_nk();
    };
```
**EN:** The surrounding comments explain the local purpose of this block: Advance consumer pipeline mk/nk.
**CN:** 周围注释解释了这一段的局部作用：Advance consumer pipeline mk/nk。

### Lines 1217-1222

```cpp
    // Perform mainloop gemm, when E is in GMEM.
    auto gemm_loop_with_GmemE = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      // Copy gmem->rmem for E operand
      auto blk_coord = make_coord(get<0>(cta_tile_coord), *k_tile_iter, get<3>(cta_tile_coord));     // (BLK_M,BLK_K,L)
      Tensor tCrE = load_E<IsF8F6F4>(mainloop_params, blk_coord, problem_shape_MNKL, thread_idx);
      ++k_tile_iter;
```
**EN:** This comment marks the mainloop logic that repeatedly loads tiles and issues MMA operations.
**CN:** 这条注释标记了主循环逻辑：重复加载 tile 并发起 MMA 运算。

### Lines 1224-1226

```cpp
      // WAIT on smem_pipe_read until data is available
      wait_barrier_mk();
      wait_barrier_nk();
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1228-1231

```cpp
      for_each(make_int_sequence<K_BLOCK_STEP>{}, [&] (auto k_block) {
        // Copy smem->rmem for SFB operand. SFB needs to be copied with all N_BLOCK_MAX,
        //   as each LDS loads several groups of data needed by one MMA instruction.
        copy_SFBs(k_block);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for SFB operand. SFB needs to be copied with all N_BLOCK_MAX, as each LDS loads several groups of data needed by one MMA instruction..
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for SFB operand. SFB needs to be copied with all N_BLOCK_MAX, as each LDS loads several groups of data needed by one MMA instruction.。

### Lines 1233-1235

```cpp
        for_each(make_int_sequence<N_BLOCK_MAX>{}, [&] (auto n_block) {
          // Copy smem->rmem for B operand
          copy_transform_B(n_block, k_block);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for B operand.
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for B operand。

### Lines 1237-1240

```cpp
          for_each(make_int_sequence<M_BLOCK_MAX>{}, [&] (auto m_block) {
            // Copy smem->rmem for A operand
            copy_transform_A(m_block, k_block);
            copy_SFA(m_block, k_block);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for A operand.
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for A operand。

### Lines 1242-1249

```cpp
            // Gemm
            cute::gemm(tiled_mma,
                      make_zip_tensor(tCrA(_,m_block,k_block), tCrSFA(_,m_block,k_block), tCrE(_,m_block,k_block)),
                      make_zip_tensor(tCrB(_,n_block,k_block), tCrSFB(_,n_block,k_block)),
                      accum(_,m_block,n_block));
          });
        });
      });
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。

### Lines 1251-1252

```cpp
      cutlass::arch::NamedBarrier::sync(
        thr_size(tiled_mma), cutlass::arch::ReservedNamedBarriers::Sm120MainloopBarrier);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1254-1257

```cpp
      // Advance consumer pipeline_nk
      release_advance_nk();
      // Wait next buffer
      wait_barrier_nk();
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1259-1260

```cpp
      for_each(make_int_sequence<K_BLOCK_STEP>{}, [&] (auto k_block) {
        auto k_block_a = k_block + K_BLOCK_STEP;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1262-1264

```cpp
        // Copy smem->rmem for SFB operand. SFB needs to be copied with all N_BLOCK_MAX,
        //   as each LDS loads several groups of data needed by one MMA instruction.
        copy_SFBs(k_block_a);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for SFB operand. SFB needs to be copied with all N_BLOCK_MAX, as each LDS loads several groups of data needed by one MMA instruction..
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for SFB operand. SFB needs to be copied with all N_BLOCK_MAX, as each LDS loads several groups of data needed by one MMA instruction.。

### Lines 1266-1268

```cpp
        for_each(make_int_sequence<N_BLOCK_MAX>{}, [&] (auto n_block) {
          // Copy smem->rmem for B operand
          copy_transform_B(n_block, k_block);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for B operand.
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for B operand。

### Lines 1270-1273

```cpp
          for_each(make_int_sequence<M_BLOCK_MAX>{}, [&] (auto m_block) {
            // Copy smem->rmem for A operand
            copy_transform_A(m_block, k_block_a);
            copy_SFA(m_block, k_block_a);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for A operand.
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for A operand。

### Lines 1275-1282

```cpp
            // Gemm
            cute::gemm(tiled_mma,
                      make_zip_tensor(tCrA(_,m_block,k_block_a), tCrSFA(_,m_block,k_block_a), tCrE(_,m_block,k_block_a)),
                      make_zip_tensor(tCrB(_,n_block,k_block), tCrSFB(_,n_block,k_block_a)),
                      accum(_,m_block,n_block));
          });
        });
      });
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。

### Lines 1284-1285

```cpp
      cutlass::arch::NamedBarrier::sync(
        thr_size(tiled_mma), cutlass::arch::ReservedNamedBarriers::Sm120MainloopBarrier);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 1287-1290

```cpp
      // Advance consumer pipeline mk/nk
      release_advance_mk();
      release_advance_nk();
    };
```
**EN:** The surrounding comments explain the local purpose of this block: Advance consumer pipeline mk/nk.
**CN:** 周围注释解释了这一段的局部作用：Advance consumer pipeline mk/nk。

### Lines 1292-1294

```cpp
    //
    // PIPELINED MAIN LOOP
    //
```
**EN:** This comment-only block labels or explains the following section:  PIPELINED MAIN LOOP .
**CN:** 这个纯注释块用于标记或解释后续区域： PIPELINED MAIN LOOP 。

### Lines 1296-1305

```cpp
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count) {
      // Case when A/B with same stages, and keep E in SMEM.
      if constexpr (UseSmemE) {
        gemm_loop_with_SmemE();
      }
      // Case when A/B with different stages, and keep E in GMEM.
      else {
        gemm_loop_with_GmemE();
      } // end if
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 1307-1308

```cpp
    }
  }
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 1310-1314

```cpp
  /// Perform a Consumer Epilogue to release all buffers
  CUTLASS_DEVICE void
  mma_tail(MainloopPipelineMK, PipelineStateMK, MainloopPipelineNK, PipelineStateNK, int) {
  }
};
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 1316-1316

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 1318-1318

```cpp
} // namespace cutlass::gemm::collective
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 1320-1320

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

## Key Concepts / 关键概念

- **EN:** TMA data movement  
  **CN:** TMA 数据搬运
- **EN:** GMMA tensor-core instructions  
  **CN:** GMMA 张量核指令
- **EN:** warp-specialized scheduling  
  **CN:** warp-specialized 调度
- **EN:** sparse MMA support  
  **CN:** 稀疏 MMA 支持
- **EN:** block-scaled numeric formats  
  **CN:** 块缩放数值格式
- **EN:** array-style MMA tiling  
  **CN:** 数组式 MMA 分块
- **EN:** Intel Xe subgroup execution  
  **CN:** Intel Xe 子组执行
- **EN:** software pipelines  
  **CN:** 软件流水线
- **EN:** compile-time validation  
  **CN:** 编译期校验
- **EN:** collective GEMM mainloop specialization  
  **CN:** collective GEMM 主循环特化

## Dependencies / 依赖关系

- **Direct includes / 直接包含:**
  - `cutlass/cutlass.h`
  - `cutlass/gemm/gemm.h`
  - `cutlass/pipeline/pipeline.hpp`
  - `cutlass/gemm/dispatch_policy.hpp`
  - `cutlass/detail/dependent_false.hpp`
  - `cutlass/trace.h`
  - `cutlass/numeric_types.h`
  - `cutlass/gemm/collective/builders/sm1xx_sparse_config.inl`
  - `cute/arch/cluster_sm90.hpp`
  - `cute/arch/copy_sm90.hpp`
  - `cute/atom/mma_atom.hpp`
  - `cute/algorithm/functional.hpp`
  - `cute/algorithm/gemm.hpp`
  - `cute/numeric/arithmetic_tuple.hpp`
- **Primary symbols / 主要符号:**
  - `ClusterShape`
  - `TileShape_`
  - `ElementPairA_`
  - `LayoutPairsA_`
  - `ElementPairB_`
  - `StridePairB_`
  - `TiledMma_`
  - `GmemTiledCopyPairA_`
  - `SmemLayoutAtomsA_`
  - `SmemCopyAtomsA_`
  - `TransformA_`
  - `GmemTiledCopyPairB_`
  - `SmemLayoutAtomsB_`
  - `SmemCopyAtomsB_`
  - `TransformB_`
  - `CollectiveMma`
- **Shared abstractions / 共享抽象:** `CollectiveMma` / `CollectiveBuilder`, dispatch-policy tags, CuTe tensor/layout utilities, and CUTLASS pipeline helpers. / `CollectiveMma` / `CollectiveBuilder`、dispatch-policy 标签、CuTe 张量/布局工具以及 CUTLASS 流水线辅助组件。