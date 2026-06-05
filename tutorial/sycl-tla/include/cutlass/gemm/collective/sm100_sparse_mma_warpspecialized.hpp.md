# sm100_sparse_mma_warpspecialized.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/collective/sm100_sparse_mma_warpspecialized.hpp`
- **Purpose (EN):** Implements an architecture-specific `CollectiveMma` specialization for SM100, covering sparse metadata handling, warp-specialized scheduling.
- **用途 (CN):** 为 SM100 实现架构特化的 `CollectiveMma` 特化版本，重点覆盖 稀疏元数据处理、warp-specialized 调度。
- **Lines / 行数:** 951

## Line-by-Line Analysis / 逐行分析

### Lines 1-31

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
#pragma once
```
**EN:** This opening block combines the license banner with `#pragma once`, documenting legal terms and making the header safe to include multiple times.
**CN:** 这个开头块同时包含许可证说明和 `#pragma once`：前者给出法律信息，后者保证头文件可被重复包含而不重复定义。

### Lines 33-43

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/detail/collective.hpp"
#include "cutlass/detail/cluster.hpp"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/numeric_types.h"
#include "cutlass/pipeline/pipeline.hpp"
#include "cutlass/gemm/gemm.h"
#include "cutlass/gemm/collective/builders/sm1xx_sparse_config.inl"
#include "cutlass/trace.h"
#include "cutlass/kernel_hardware_info.hpp"
#include "cutlass/detail/sm100_tmem_helper.hpp"
```
**EN:** This include block imports cutlass.h, collective.hpp, cluster.hpp, dispatch_policy.hpp, and 7 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 cutlass.h、collective.hpp、cluster.hpp、dispatch_policy.hpp 等 11 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 45-49

```cpp
#include "cute/algorithm/functional.hpp"
#include "cute/arch/cluster_sm90.hpp"
#include "cute/atom/mma_atom.hpp"
#include "cute/algorithm/gemm.hpp"
#include "cute/numeric/arithmetic_tuple.hpp"
```
**EN:** This include block imports functional.hpp, cluster_sm90.hpp, mma_atom.hpp, gemm.hpp, and 1 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 functional.hpp、cluster_sm90.hpp、mma_atom.hpp、gemm.hpp 等 5 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

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

### Lines 56-56

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 58-104

```cpp
// WarpSpecialized Mainloop
// Both DMA Load and MMA methods of this class must be run by a single thread that's picked by elect_one
template <
  int Stages,
  int SchedulerPipelineStageCount,
  int AccumulatorPipelineStageCount,
  class ClusterShape,   // Static cluster shape or dynamic (int, int, _1)
  class TileShape_,     // (MmaAtomShapeM, MmaAtomShapeN, TileK)
  class ElementA_,
  class LayoutPairAE_,
  class ElementB_,
  class StrideB_,
  class TiledMma_,
  class GmemTiledCopyA_,
  class SmemLayoutAtomA_,
  class SmemCopyAtomA_,
  class TransformA_,
  class GmemTiledCopyB_,
  class SmemLayoutAtomB_,
  class SmemCopyAtomB_,
  class TransformB_>
struct CollectiveMma<
    MainloopSm100TmaUmmaWarpSpecializedSparse<
      Stages,
      SchedulerPipelineStageCount,
      AccumulatorPipelineStageCount,
      ClusterShape>,
    TileShape_,
    ElementA_,
    LayoutPairAE_,
    ElementB_,
    StrideB_,
    TiledMma_,
    GmemTiledCopyA_,
    SmemLayoutAtomA_,
    SmemCopyAtomA_,
    TransformA_,
    GmemTiledCopyB_,
    SmemLayoutAtomB_,
    SmemCopyAtomB_,
    TransformB_>
{
  //
  // Type Aliases
  //
  using TiledMma = TiledMma_;
  using AtomThrShapeMNK = Shape<decltype(shape<0>(typename TiledMma::ThrLayoutVMNK{})), _1, _1>;
```
**EN:** This template block declares or specializes `CollectiveMma`, the mainloop object that coordinates tile movement and matrix-multiply work.
**CN:** 这个模板块声明或特化了 `CollectiveMma`，它负责协调 tile 搬运与矩阵乘主循环。

### Lines 106-111

```cpp
  using DispatchPolicy = MainloopSm100TmaUmmaWarpSpecializedSparse<
                          Stages,
                          SchedulerPipelineStageCount,
                          AccumulatorPipelineStageCount,
                          ClusterShape>;
  using TileShape = TileShape_;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 113-114

```cpp
  static constexpr bool IsDynamicCluster = not cute::is_static_v<ClusterShape>;
  static constexpr bool IsOverlappingAccum = DispatchPolicy::IsOverlappingAccum;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 116-117

```cpp
  CUTE_STATIC_ASSERT_V(evenly_divides(TileShape{}, tile_shape(TiledMma{})),
                       "Static cluster shape used: TileShape should be evenly divided by TiledMma");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 119-119

```cpp
  using CtaShape_MNK = decltype(shape_div(TileShape{}, AtomThrShapeMNK{}));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 121-126

```cpp
  // Define A and B block shapes for reduced size TMA_LOADs
  using MmaShapeA_MK = decltype(partition_shape_A(TiledMma{}, make_shape(size<0>(TileShape{}), size<2>(TileShape{}))));
  using MmaShapeB_NK = decltype(partition_shape_B(TiledMma{}, make_shape(size<1>(TileShape{}), size<2>(TileShape{}))));
  static_assert(get<0,0>(MmaShapeA_MK{}) == 128 &&
                (get<2>(MmaShapeA_MK{}) == 2 || get<2>(MmaShapeA_MK{}) == 4),
                "This kernel only support MmaShape=128 and 2/4 kphase.");
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 128-134

```cpp
  using ElementA = ElementA_;
  using ElementAMma = typename TiledMma::ValTypeA;
  using ElementAMmaRaw = typename ElementAMma::raw_type;
  using LayoutPairAE = LayoutPairAE_;
  using LayoutA =  remove_cvref_t<decltype(get<0>(LayoutPairAE{}))>;
  static constexpr int ElementAMmaSparsity = ElementAMma::sparsity;
  static constexpr bool IsRuntimeDataTypeA = cutlass::gemm::collective::detail::is_sm10x_runtime_f8f6f4<ElementA>();
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 136-139

```cpp
  using ElementB = ElementB_;
  using ElementBMma = typename TiledMma::ValTypeB;
  using StrideB = StrideB_;
  static constexpr bool IsRuntimeDataTypeB = cutlass::gemm::collective::detail::is_sm10x_runtime_f8f6f4<ElementB>();
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 141-141

```cpp
  static constexpr bool IsRuntimeDataType = IsRuntimeDataTypeA && IsRuntimeDataTypeB;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 143-146

```cpp
  using ElementEMma = typename TiledMma::ValTypeE;
  using ElementE = typename ElementEMma::raw_type;
  using LayoutE =  remove_cvref_t<decltype(get<1>(LayoutPairAE{}))>;
  static constexpr int ElementEMmaSparsity = ElementEMma::sparsity;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 148-157

```cpp
  using ElementAccumulator = typename TiledMma::ValTypeC;
  using GmemTiledCopyA = GmemTiledCopyA_;
  using GmemTiledCopyB = GmemTiledCopyB_;
  using SmemLayoutAtomA = SmemLayoutAtomA_;
  using SmemLayoutAtomB = SmemLayoutAtomB_;
  using SmemCopyAtomA = SmemCopyAtomA_;
  using SmemCopyAtomB = SmemCopyAtomB_;
  using TransformA = TransformA_;
  using TransformB = TransformB_;
  using ArchTag = typename DispatchPolicy::ArchTag;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 159-162

```cpp
  static_assert(is_sparse<ElementAMma>::value, "ElementAMma is sparse");
  static_assert(!is_sparse<ElementA>::value, "ElementA is not sparse");
  static_assert((IsRuntimeDataTypeA && IsRuntimeDataTypeB) || (!IsRuntimeDataTypeA && !IsRuntimeDataTypeB),
                "ElementA and ElementB should be both runtime or both static.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 164-165

```cpp
  // LayoutA is nested in the stride due to the sparsity.
  static constexpr bool is_A_mn_major = cute::is_same_v<decltype(stride<0>(LayoutA{})), Int<ElementAMmaSparsity>>;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 167-169

```cpp
  using SparseConfig = cutlass::Sm1xxGemmSparseConfig<ElementAMma,
                                                      cute::conditional_t<is_A_mn_major, cutlass::layout::ColumnMajor, cutlass::layout::RowMajor>,
                                                      ElementEMma>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 171-175

```cpp
  // The offline permutation for the metadata.
  using SmemLayoutAtomE_ = typename SparseConfig::TensorEAtom;
  using SmemLayoutAtomE  = ComposedLayout<Swizzle<0,4,3>,
                                          smem_sparse_ptr_flag_bits<ElementEMmaSparsity, sizeof_bits_v<ElementE>>,
                                          SmemLayoutAtomE_>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 177-178

```cpp
  // Metadata pathways
  using GmemCopyAtomE = GmemTiledCopyA;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 180-184

```cpp
  using MainloopPipeline = cutlass::PipelineTmaSparseUmmaAsync<
                             DispatchPolicy::Stages,
                             ClusterShape,
                             AtomThrShapeMNK>;
  using MainloopPipelineState = typename MainloopPipeline::PipelineState;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 186-191

```cpp
  static constexpr int UtccpReuseCnt = ((size<2>(TileShape{}) / typename SparseConfig::TensorEAtomK{}) == 0) ?
                                        typename SparseConfig::TensorEAtomK{} / size<2>(TileShape{}) : 1;
  static_assert(UtccpReuseCnt == 1 || UtccpReuseCnt == 2, "UTCCP reuse count can only be either one or two");
  // (TileM, TileN, TileK) TileK is adjusted according to the reuse.
  using TileShapeE = decltype(replace<2>(TileShape{}, cute::lcm(size<2>(TileShape{}), typename SparseConfig::TensorEAtomK{})));
  using MmaShapeE_MK = decltype(partition_shape_A(TiledMma{}, make_shape(size<0>(TileShapeE{}), size<2>(TileShapeE{}))));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 193-197

```cpp
  static_assert(rank(SmemLayoutAtomA{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtomA must evenly divide the tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtomA must evenly divide the tile shape.");
  static_assert(cute::is_void_v<SmemCopyAtomA>,
      "SM100 UMMA cannot have a non-void copy atom for smem sourced instructions.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 199-203

```cpp
  static_assert(rank(SmemLayoutAtomB{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<1>(TileShape{}) % size<0>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtomB must evenly divide the tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtomB must evenly divide the tile shape.");
  static_assert(cute::is_void_v<SmemCopyAtomB>,
      "SM100 UMMA cannot have a non-void copy atom for smem sourced instructions.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 205-206

```cpp
  static_assert(rank(SmemLayoutAtomE{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomE{})) == 0, "SmemLayoutAtomE must evenly divide the tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 208-223

```cpp
  // Tile along K mode first before tiling over MN. PIPE mode last as usual.
  // This maximizes TMA boxes due to better smem-K vectorization, reducing total issued TMAs.
  // (MMA_TILE_M,MMA_TILE_K),MMA_M,MMA_K,PIPE)
  using SmemLayoutA = decltype(UMMA::tile_to_mma_shape(
      SmemLayoutAtomA{},
      append(MmaShapeA_MK{}, Int<DispatchPolicy::Stages>{}),
      cute::conditional_t<is_A_mn_major, Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
  // (MMA_TILE_M,MMA_TILE_K),MMA_M,MMA_K,PIPE) that one UTCCP instruction can provide
  using SmemLayoutE = decltype(UMMA::tile_to_mma_shape(
      SmemLayoutAtomE{},
      append(MmaShapeE_MK{}, Int<DispatchPolicy::Stages>{})));
  // (MMA_TILE_N,MMA_TILE_K),MMA_N,MMA_K,PIPE)
  using SmemLayoutB = decltype(UMMA::tile_to_mma_shape(
      SmemLayoutAtomB{},
      append(MmaShapeB_NK{}, Int<DispatchPolicy::Stages>{}),
      cute::conditional_t<cutlass::gemm::detail::is_mn_major<StrideB>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 225-239

```cpp
  static_assert(cute::is_base_of<cute::UMMA::DescriptorIterator, typename TiledMma::FrgTypeA>::value &&
                cute::is_base_of<cute::UMMA::DescriptorIterator, typename TiledMma::FrgTypeB>::value,
                "MMA atom must source both A and B operand from smem_desc for this mainloop.");
  static_assert(
      (size(AtomThrShapeMNK{}) == 1 &&
        (cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD> || cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD_MULTICAST>)) ||
      (size(AtomThrShapeMNK{}) == 2 &&
        (cute::is_same_v<GmemTiledCopyA, SM100_TMA_2SM_LOAD> || cute::is_same_v<GmemTiledCopyA, SM100_TMA_2SM_LOAD_MULTICAST>)),
      "GmemTiledCopy - invalid TMA copy atom specified.");
  static_assert(
      (size(AtomThrShapeMNK{}) == 1 &&
        (cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD> || cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD_MULTICAST>)) ||
      (size(AtomThrShapeMNK{}) == 2 &&
        (cute::is_same_v<GmemTiledCopyB, SM100_TMA_2SM_LOAD> || cute::is_same_v<GmemTiledCopyB, SM100_TMA_2SM_LOAD_MULTICAST>)),
      "GmemTiledCopy -  invalid TMA copy atom specified.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 241-242

```cpp
  static_assert(rank(SmemLayoutAtomE{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomE{})) == 0, "SmemLayoutAtomE must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 244-244

```cpp
  static constexpr bool IsF8F6F4 = detail::is_sm100_sparse_f8f6f4<TiledMma, ElementA, ElementB>();
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 246-250

```cpp
  using TmaInternalElementA = cute::sparse_elem<ElementAMmaSparsity,
                                                cute::conditional_t<cute::is_same_v<ElementA, float>,
                                                                    cutlass::tfloat32_t,
                                                                    ElementAMmaRaw>>;
  using TmaInternalElementB = cute::conditional_t<cute::is_same_v<ElementB, float>, cutlass::tfloat32_t, ElementBMma>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 252-256

```cpp
  using SmemAllocTypeA = cute::sparse_elem<ElementAMmaSparsity,
                                           cute::conditional_t<IsF8F6F4 && cute::sizeof_bits_v<ElementAMmaRaw> < 8,
                                                               uint8_t,
                                                               ElementAMmaRaw>>;
  using SmemAllocTypeB = cute::conditional_t<IsF8F6F4 && cute::sizeof_bits_v<ElementBMma> < 8, uint8_t, ElementBMma>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 258-264

```cpp
  // Kernel Input Data Type that consider runtime dtype
  using ArrayElementA = cute::conditional_t<IsRuntimeDataTypeA,
                                            cute::uint_bit_t<cute::sizeof_bits_v<ElementA>>,
                                            ElementA>;
  using ArrayElementB = cute::conditional_t<IsRuntimeDataTypeB,
                                            cute::uint_bit_t<cute::sizeof_bits_v<ElementB>>,
                                            ElementB>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 266-267

```cpp
  using RuntimeDataTypeA = cute::conditional_t<IsRuntimeDataTypeA, cute::UMMA::MXF8F6F4Format, void*>;
  using RuntimeDataTypeB = cute::conditional_t<IsRuntimeDataTypeB, cute::UMMA::MXF8F6F4Format, void*>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 269-274

```cpp
  struct SharedStorage {
    struct TensorStorage : cute::aligned_struct<128, _0> {
      cute::ArrayEngine<SmemAllocTypeA, cute::cosize_v<SmemLayoutA>> smem_A;
      cute::ArrayEngine<SmemAllocTypeB, cute::cosize_v<SmemLayoutB>> smem_B;
      cute::ArrayEngine<ElementEMma, cute::cosize_v<SmemLayoutE>> smem_E;
    } tensors;
```
**EN:** This block introduces `SharedStorage` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `SharedStorage`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 276-278

```cpp
    using PipelineStorage = typename MainloopPipeline::SharedStorage;
    PipelineStorage pipeline;
  };
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 280-282

```cpp
  // Expose shared storage for tensors/pipelines separately to allow kernel layer to reorder them.
  using TensorStorage = typename SharedStorage::TensorStorage;
  using PipelineStorage = typename SharedStorage::PipelineStorage;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 284-290

```cpp
  // Only one thread issues the TMA and updates the barriers in a 2SM MMA, adjust bytes accordingly
  static constexpr uint32_t ABTmaTransactionBytes =
    cutlass::bits_to_bytes(size(AtomThrShapeMNK{}) * cosize(take<0,3>(SmemLayoutA{})) * cute::sizeof_bits_v<TmaInternalElementA>) +
    cutlass::bits_to_bytes(size(AtomThrShapeMNK{}) * cosize(take<0,3>(SmemLayoutB{})) * cute::sizeof_bits_v<TmaInternalElementB>);
  static constexpr uint32_t MetadataTmaTransactionBytes =
    cutlass::bits_to_bytes(size(AtomThrShapeMNK{}) * cosize(take<0,3>(SmemLayoutE{})) * cute::sizeof_bits_v<ElementEMma>);
  static constexpr uint32_t MainLoadTmaTransactionBytes = ABTmaTransactionBytes;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 292-296

```cpp
  template <class AccTensor, class ETensor>
  struct TmemStorage {
    AccTensor accumulators;
    ETensor tCtE;
  };
```
**EN:** This block introduces `AccTensor` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `AccTensor`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 298-317

```cpp
  template <
    class KTileCount, class KTileMetadataCount,
    class GTensorPartitionedA, class GTensorPartitionedB, class GTensorPartitionedE,
    class STensorA, class STensorB, class STensorE
  >
  struct LoadParams {
    // for scheduler
    KTileCount k_tiles;
    KTileMetadataCount k_tiles_metadata;
    // for input tensor values
    GTensorPartitionedA tAgA_mkl;
    GTensorPartitionedB tBgB_nkl;
    GTensorPartitionedE tEgE_nkl;
    STensorA tAsA;
    STensorB tBsB;
    STensorE tEsE;
    // the TMA multicast masks
    uint16_t mcast_mask_a;
    uint16_t mcast_mask_b;
    uint16_t mcast_mask_e;
```
**EN:** This block introduces `KTileCount` and groups related declarations around that symbol. This helper structure packages the values needed by a later load or MMA phase so the call sites stay organized.
**CN:** 这一段引入了 `KTileCount`，并围绕该符号组织相关声明。 这个辅助结构把后续 load 或 MMA 阶段需要的值打包起来，使调用点保持清晰。

### Lines 319-329

```cpp
    CUTLASS_DEVICE
    LoadParams (
        KTileCount k_tiles_, KTileMetadataCount k_tiles_metadata_,
        GTensorPartitionedA tAgA_mkl_, GTensorPartitionedB tBgB_nkl_, GTensorPartitionedE tEgE_nkl_,
        STensorA tAsA_, STensorB tBsB_, STensorE tEsE_,
        uint16_t mcast_mask_a_, uint16_t mcast_mask_b_, uint16_t mcast_mask_e_)
    : k_tiles(k_tiles_), k_tiles_metadata(k_tiles_metadata_)
    , tAgA_mkl(tAgA_mkl_), tBgB_nkl(tBgB_nkl_), tEgE_nkl(tEgE_nkl_)
    , tAsA(tAsA_), tBsB(tBsB_), tEsE(tEsE_)
    , mcast_mask_a(mcast_mask_a_), mcast_mask_b(mcast_mask_b_), mcast_mask_e(mcast_mask_e_) {}
  };
```
**EN:** This helper structure packages the values needed by a later load or MMA phase so the call sites stay organized.
**CN:** 这个辅助结构把后续 load 或 MMA 阶段需要的值打包起来，使调用点保持清晰。

### Lines 331-346

```cpp
  template <
    class TiledMma,
    class FragmentA, class FragmentB,
    class FragmentE, class ETiledCopy, class SmemFrgE, class TmemFrgE
  >
  struct MmaParams {
    TiledMma tiled_mma;
    // A
    FragmentA tCrA;
    // B
    FragmentB tCrB;
    // E
    FragmentE tCtE;
    ETiledCopy tiled_copy_s2t_E;
    SmemFrgE thr_tCsE_s2t;
    TmemFrgE thr_tCtE_s2t;
```
**EN:** This block introduces `TiledMma` and groups related declarations around that symbol. This helper structure packages the values needed by a later load or MMA phase so the call sites stay organized.
**CN:** 这一段引入了 `TiledMma`，并围绕该符号组织相关声明。 这个辅助结构把后续 load 或 MMA 阶段需要的值打包起来，使调用点保持清晰。

### Lines 348-358

```cpp
    CUTLASS_DEVICE
    MmaParams (
        TiledMma tiled_mma_,
        FragmentA tCrA_, FragmentB tCrB_,
        FragmentE tCtE_, ETiledCopy tiled_copy_s2t_E_,
        SmemFrgE thr_tCsE_s2t_, TmemFrgE thr_tCtE_s2t_)
    : tiled_mma(tiled_mma_)
    , tCrA(tCrA_), tCrB(tCrB_)
    , tCtE(tCtE_), tiled_copy_s2t_E(tiled_copy_s2t_E_)
    , thr_tCsE_s2t(thr_tCsE_s2t_), thr_tCtE_s2t(thr_tCtE_s2t_) {}
  };
```
**EN:** This helper structure packages the values needed by a later load or MMA phase so the call sites stay organized.
**CN:** 这个辅助结构把后续 load 或 MMA 阶段需要的值打包起来，使调用点保持清晰。

### Lines 360-371

```cpp
  // Host side kernel arguments
  struct Arguments {
    // A is A Compressed, not raw tensorA
    ArrayElementA const* ptr_A{nullptr};
    LayoutA layout_a{};
    ArrayElementB const* ptr_B{nullptr};
    StrideB dB{};
    ElementE const* ptr_E{nullptr};
    LayoutE layout_e{};
    RuntimeDataTypeA runtime_data_type_a{};
    RuntimeDataTypeB runtime_data_type_b{};
  };
```
**EN:** This block introduces `Arguments` and groups related declarations around that symbol. `Arguments` is the host-side bundle: callers fill it with pointers, strides, and other launch-time values before parameter lowering.
**CN:** 这一段引入了 `Arguments`，并围绕该符号组织相关声明。 `Arguments` 是主机侧参数包：调用者在参数下沉前把指针、步长及其他启动期数据填入其中。

### Lines 373-377

```cpp
  // Device side kernel params
  struct Params {
    using ClusterLayout_VMNK =
      decltype(tiled_divide(make_layout(conditional_return<IsDynamicCluster>(make_shape(uint32_t(0), uint32_t(0), Int<1>{}),
                                                                              ClusterShape{})), make_tile(typename TiledMma::AtomThrID{})));
```
**EN:** This block introduces `Params` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `Params`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 379-386

```cpp
    using TMA_A = decltype(make_tma_atom_A_sm100<typename TmaInternalElementA::raw_type>(
        GmemTiledCopyA{},
        make_tensor(recast_ptr<TmaInternalElementA>(nullptr), LayoutA{}),
        SmemLayoutA{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        ClusterLayout_VMNK{})
      );
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 388-395

```cpp
    using TMA_E = decltype(make_tma_atom_A_sm100<uint64_t>( // use uint64_t to get the largest loading box.
        GmemCopyAtomE{},
        make_tensor(recast_ptr<ElementEMma>(nullptr), LayoutE{}),
        SmemLayoutE{}(_,_,_,cute::Int<0>{}),
        TileShapeE{},
        TiledMma{},
        ClusterLayout_VMNK{})
      );
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 397-404

```cpp
    using TMA_B = decltype(make_tma_atom_B_sm100<TmaInternalElementB>(
        GmemTiledCopyB{},
        make_tensor(recast_ptr<TmaInternalElementB>(nullptr), repeat_like(StrideB{}, int32_t(0)), StrideB{}),
        SmemLayoutB{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        ClusterLayout_VMNK{})
      );
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 406-417

```cpp
    TMA_A tma_load_a;
    TMA_E tma_load_e;
    TMA_B tma_load_b;
    TMA_A tma_load_a_fallback;
    TMA_E tma_load_e_fallback;
    TMA_B tma_load_b_fallback;
    LayoutA layout_a;
    LayoutE layout_e;
    dim3 cluster_shape_fallback;
    RuntimeDataTypeA runtime_data_type_a;
    RuntimeDataTypeB runtime_data_type_b;
  };
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 419-439

```cpp
  CUTLASS_DEVICE
  CollectiveMma(Params const& params, ClusterShape cluster_shape, uint32_t block_rank_in_cluster)
    : cluster_shape_(cluster_shape)
    , block_rank_in_cluster_(block_rank_in_cluster)
    , layout_a_(params.layout_a)
    , layout_e_(params.layout_e)
    , runtime_data_type_a_(params.runtime_data_type_a)
    , runtime_data_type_b_(params.runtime_data_type_b) {
    if constexpr (IsDynamicCluster) {
      const bool is_fallback_cluster = (cute::size<0>(cluster_shape_) == params.cluster_shape_fallback.x &&
                                        cute::size<1>(cluster_shape_) == params.cluster_shape_fallback.y);
      observed_tma_load_a_ = is_fallback_cluster ? &params.tma_load_a_fallback : &params.tma_load_a;
      observed_tma_load_e_ = is_fallback_cluster ? &params.tma_load_e_fallback : &params.tma_load_e;
      observed_tma_load_b_ = is_fallback_cluster ? &params.tma_load_b_fallback : &params.tma_load_b;
    }
    else {
      observed_tma_load_a_ = &params.tma_load_a;
      observed_tma_load_e_ = &params.tma_load_e;
      observed_tma_load_b_ = &params.tma_load_b;
    }
  }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 441-447

```cpp
  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(
    ProblemShape const& problem_shape,
    Arguments const& args,
    [[maybe_unused]] void* workspace,
    cutlass::KernelHardwareInfo const& hw_info = cutlass::KernelHardwareInfo{}) {
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 449-451

```cpp
    // Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK)
    auto problem_shape_MNKL = append<4>(problem_shape, 1);
    auto [M,N,K,L] = problem_shape_MNKL;
```
**EN:** The surrounding comments explain the local purpose of this block: Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK).
**CN:** 周围注释解释了这一段的局部作用：Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK)。

### Lines 453-455

```cpp
    auto ptr_A = recast_ptr<TmaInternalElementA>(args.ptr_A);
    auto ptr_B = recast_ptr<TmaInternalElementB>(args.ptr_B);
    auto ptr_E = recast_ptr<ElementEMma>(args.ptr_E);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 457-459

```cpp
    Tensor tensor_a = make_tensor(ptr_A, args.layout_a);
    Tensor tensor_b = make_tensor(ptr_B, make_layout(make_shape(N,K,L), args.dB));
    Tensor tensor_e = make_tensor(ptr_E, args.layout_e);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 461-461

```cpp
    auto cluster_shape = cutlass::detail::select_cluster_shape(ClusterShape{}, hw_info.cluster_shape);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 463-473

```cpp
    // Cluster layout for TMA construction
    auto cluster_layout_vmnk = tiled_divide(make_layout(cluster_shape), make_tile(typename TiledMma::AtomThrID{}));
    auto cluster_shape_fallback = cutlass::detail::select_cluster_shape(ClusterShape{}, hw_info.cluster_shape_fallback);
    auto cluster_layout_vmnk_fallback = tiled_divide(make_layout(cluster_shape_fallback), make_tile(typename TiledMma::AtomThrID{}));
    typename Params::TMA_A tma_load_a = make_tma_atom_A_sm100<typename TmaInternalElementA::raw_type>(
        GmemTiledCopyA{},
        tensor_a,
        SmemLayoutA{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        cluster_layout_vmnk);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 475-481

```cpp
    typename Params::TMA_E tma_load_e = make_tma_atom_A_sm100<uint64_t>( // use uint64_t to get the largest loading box.
        GmemCopyAtomE{},
        tensor_e,
        SmemLayoutE{}(_,_,_,cute::Int<0>{}),
        TileShapeE{},
        TiledMma{},
        cluster_layout_vmnk);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 483-489

```cpp
    typename Params::TMA_B tma_load_b = make_tma_atom_B_sm100<TmaInternalElementB>(
        GmemTiledCopyB{},
        tensor_b,
        SmemLayoutB{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        cluster_layout_vmnk);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 491-497

```cpp
    typename Params::TMA_A tma_load_a_fallback = make_tma_atom_A_sm100<typename TmaInternalElementA::raw_type>(
        GmemTiledCopyA{},
        tensor_a,
        SmemLayoutA{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        cluster_layout_vmnk_fallback);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 499-505

```cpp
    typename Params::TMA_E tma_load_e_fallback = make_tma_atom_A_sm100<uint64_t>( // use uint64_t to get the largest loading box.
        GmemCopyAtomE{},
        tensor_e,
        SmemLayoutE{}(_,_,_,cute::Int<0>{}),
        TileShapeE{},
        TiledMma{},
        cluster_layout_vmnk_fallback);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 507-513

```cpp
    typename Params::TMA_B tma_load_b_fallback = make_tma_atom_B_sm100<TmaInternalElementB>(
        GmemTiledCopyB{},
        tensor_b,
        SmemLayoutB{}(_,_,_,cute::Int<0>{}),
        TileShape{},
        TiledMma{},
        cluster_layout_vmnk_fallback);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 515-528

```cpp
    return {
      tma_load_a,
      tma_load_e,
      tma_load_b,
      tma_load_a_fallback,
      tma_load_e_fallback,
      tma_load_b_fallback,
      args.layout_a,
      args.layout_e,
      hw_info.cluster_shape_fallback,
      args.runtime_data_type_a,
      args.runtime_data_type_b
    };
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 530-534

```cpp
  template <class ProblemShape>
  static bool
  can_implement(
      ProblemShape const& problem_shape,
      [[maybe_unused]] Arguments const& args) {
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. This method performs runtime feasibility checks, usually validating alignment, layout assumptions, or shape constraints before launching the kernel.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这个方法执行运行时可实现性检查，通常会在启动内核前验证对齐、布局假设或形状约束。

### Lines 536-537

```cpp
    auto problem_shape_MNKL = append<4>(problem_shape, 1);
    auto [M,N,K,L] = problem_shape_MNKL;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 539-541

```cpp
    constexpr int tma_alignment_bits_A = cutlass::detail::get_input_alignment_bits<ElementA, IsF8F6F4>();
    constexpr int tma_alignment_bits_B = cutlass::detail::get_input_alignment_bits<ElementB, IsF8F6F4>();
    constexpr int min_tma_aligned_elements_A = tma_alignment_bits_A / cute::sizeof_bits_v<ElementA>;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 543-555

```cpp
    bool implementable = true;
    // Check Alignment A
    if constexpr (is_A_mn_major) {
      implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_A>(cute::make_shape(M,     K/2, L),
                                                                                                    cute::make_stride(_1{}, M,   M*K/2));
    }
    else { // If A is K-major
      implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_A>(cute::make_shape(M,    K/2,  L),
                                                                                                    cute::make_stride(K/2, _1{}, M*K/2));
    }
    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem Size doesn't meet the minimum alignment requirements for TMA on tensorA\n");
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 557-562

```cpp
    // Check Alignment B
    constexpr int min_tma_aligned_elements_B = tma_alignment_bits_B / cute::sizeof_bits_v<ElementB>;
    implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_B>(cute::make_shape(N,K,L), StrideB{});
    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem Size doesn't meet the minimum alignment requirements for TMA on tensorB\n");
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 564-570

```cpp
    // Check for AB layout requirement
    const auto layout_a_ref = SparseConfig::fill_layoutA(problem_shape_MNKL);
    const auto layout_e_ref = SparseConfig::fill_layoutE(problem_shape_MNKL);
    implementable = implementable && (layout_a_ref == args.layout_a);
    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: layout_a mismatch\n");
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 572-575

```cpp
    implementable = implementable && (layout_e_ref == args.layout_e);
    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: layout_e mismatch\n");
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 577-581

```cpp
    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem Size doesn't meet the minimum alignment requirements for TMA.\n");
    }
    return implementable;
  }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 583-589

```cpp
  /// Issue Tma Descriptor Prefetch -- ideally from a single thread for best performance
  CUTLASS_DEVICE void
  prefetch_tma_descriptors() {
    cute::prefetch_tma_descriptor(observed_tma_load_a_->get_tma_descriptor());
    cute::prefetch_tma_descriptor(observed_tma_load_b_->get_tma_descriptor());
    cute::prefetch_tma_descriptor(observed_tma_load_e_->get_tma_descriptor());
  }
```
**EN:** The code prefetches descriptor state early so later TMA or copy operations can start with lower latency.
**CN:** 这里会提前预取描述符状态，从而让后续 TMA 或拷贝操作以更低延迟启动。

### Lines 591-595

```cpp
  /// Construct A Single Stage's Accumulator Shape
  CUTLASS_DEVICE static
  auto
  partition_accumulator_shape() {
    auto acc_shape = partition_shape_C(TiledMma{}, take<0,2>(TileShape{}));  // ((MMA_TILE_M,MMA_TILE_N),MMA_M,MMA_N)
```
**EN:** This logic sets up accumulator fragments or TMEM-backed storage so the mainloop and epilogue agree on where partial sums live.
**CN:** 这些逻辑用于设置累加器片段或基于 TMEM 的存储，使主循环与 epilogue 对部分和的存放位置达成一致。

### Lines 597-598

```cpp
    return acc_shape;
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 600-605

```cpp
  template <class TmemStorage>
  CUTLASS_DEVICE static
  auto
  slice_accumulator(TmemStorage tmem_storage, int stage) {
    return cute::make_tuple(tmem_storage.accumulators(_,_,_,stage));
  }
```
**EN:** This block introduces `TmemStorage` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TmemStorage`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 607-616

```cpp
  template <class EpilogueTile, bool IsOverlappingAccum = false>
  CUTLASS_DEVICE static
  auto
  init_tmem_tensors(EpilogueTile epi_tile) {
    TiledMma tiled_mma;
    auto acc_shape = partition_accumulator_shape();
    // ((MMA_TILE_M,MMA_TILE_N),MMA_M,MMA_N,ACC_PIPE) where ACC_PIPE=2 so we can double buffer our accumulators for mainloop and epilogue.
    Tensor accumulators = cutlass::detail::make_sm100_accumulator<AccumulatorPipelineStageCount, IsOverlappingAccum>(
        tiled_mma, acc_shape, EpilogueTile{});
    Tensor tCtE   = make_tensor<typename TiledMma::FrgTypeE>(take<0,3>(shape(SmemLayoutE{})));
```
**EN:** This block introduces `EpilogueTile` and groups related declarations around that symbol. This logic sets up accumulator fragments or TMEM-backed storage so the mainloop and epilogue agree on where partial sums live.
**CN:** 这一段引入了 `EpilogueTile`，并围绕该符号组织相关声明。 这些逻辑用于设置累加器片段或基于 TMEM 的存储，使主循环与 epilogue 对部分和的存放位置达成一致。

### Lines 618-620

```cpp
    TmemStorage<decltype(accumulators), decltype(tCtE)> tmem_storage;
    tmem_storage.accumulators = accumulators;
    tmem_storage.tCtE = tCtE;
```
**EN:** This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 622-623

```cpp
    return tmem_storage;
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 625-631

```cpp
  template <class TmemStorage>
  CUTLASS_DEVICE static
  void
  set_tmem_offsets(TmemStorage& tmem_storage, uint32_t tmem_base_addr) {
    tmem_storage.accumulators.data() = tmem_base_addr;
    tmem_storage.tCtE.data()         = tmem_base_addr + cutlass::detail::find_tmem_tensor_col_offset(tmem_storage.accumulators);
  }
```
**EN:** This block introduces `TmemStorage` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TmemStorage`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 633-646

```cpp
  /// Set up the data needed by this collective for load.
  /// Return tuple element contain
  /// gA_mkl - The tiled tma tensor for input A
  /// gB_nkl - The tiled tma tensor for input B
  /// tAsA - partitioned smem tensor for A
  /// tBsB - partitioned smem tensor for B
  /// mcast_mask_a - tma multicast mask for A
  /// mcast_mask_b - tma multicast mask for B
  template <class ProblemShape_MNKL>
  CUTLASS_DEVICE auto
  load_init(
      ProblemShape_MNKL const& problem_shape_MNKL,
      TensorStorage& shared_tensors) const {
    using X = Underscore;
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 648-649

```cpp
    // Separate out problem shape for convenience
    auto [M,N,K,L] = problem_shape_MNKL;
```
**EN:** The surrounding comments explain the local purpose of this block: Separate out problem shape for convenience.
**CN:** 周围注释解释了这一段的局部作用：Separate out problem shape for convenience。

### Lines 651-654

```cpp
    // Represent the full tensors -- get these from TMA
    Tensor mA_mkl = observed_tma_load_a_->get_tma_tensor(layout_a_.shape());
    Tensor mB_nkl = observed_tma_load_b_->get_tma_tensor(make_shape(N,K,L));
    Tensor mE_mkl = observed_tma_load_e_->get_tma_tensor(layout_e_.shape());
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 656-659

```cpp
    // Tile the tensors and defer the slice
    Tensor gA_mkl = local_tile(mA_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});    // (BLK_M, BLK_K, m, k, l)
    Tensor gB_nkl = local_tile(mB_nkl, TileShape{}, make_coord(_,_,_), Step< X,_1,_1>{});    // (BLK_N, BLK_K, n, k, l)
    Tensor gE_mkl = local_tile(mE_mkl, TileShapeE{}, make_coord(_,_,_), Step<_1, X,_1>{});    // (BLK_M, BLK_K, m, k, l)
```
**EN:** The surrounding comments explain the local purpose of this block: Tile the tensors and defer the slice.
**CN:** 周围注释解释了这一段的局部作用：Tile the tensors and defer the slice。

### Lines 661-662

```cpp
    // Partition for this CTA
    ThrMMA cta_mma = TiledMma{}.get_slice(BlockIdxX() % size(typename TiledMma::AtomThrID{}));
```
**EN:** The surrounding comments explain the local purpose of this block: Partition for this CTA.
**CN:** 周围注释解释了这一段的局部作用：Partition for this CTA。

### Lines 664-666

```cpp
    Tensor tCgA_mkl = cta_mma.partition_A(gA_mkl);          // (MMA, MMA_M, MMA_K, m, k, l)
    Tensor tCgB_nkl = cta_mma.partition_B(gB_nkl);          // (MMA, MMA_N, MMA_K, n, k, l)
    Tensor tCgE_mkl = cta_mma.partition_A(gE_mkl);          // (MMA, MMA_M, MMA_K, m, k, l)
```
**EN:** This block prepares the load path by slicing global tensors, partitioning work for the current CTA/subgroup, and binding shared-memory destinations.
**CN:** 这一段为加载路径做准备：切分全局张量、为当前 CTA/子组划分工作，并绑定共享内存目标。

### Lines 668-670

```cpp
    Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.begin()), SmemLayoutA{});  // (MMA,MMA_M,MMA_K,PIPE)
    Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.begin()), SmemLayoutB{});  // (MMA,MMA_N,MMA_K,PIPE)
    Tensor sE = make_tensor(make_smem_ptr(shared_tensors.smem_E.begin()), SmemLayoutE{});  // (MMA,MMA_M,MMA_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 672-675

```cpp
    // Define the CTA-in-cluster Layout and Coord
    Layout cta_layout_mnk  = make_layout(cluster_shape_);
    Layout cta_layout_vmnk = tiled_divide(cta_layout_mnk, make_tile(typename TiledMma::AtomThrID{}));
    auto cta_coord_vmnk  = cta_layout_vmnk.get_flat_coord(block_rank_in_cluster_);
```
**EN:** The surrounding comments explain the local purpose of this block: Define the CTA-in-cluster Layout and Coord.
**CN:** 周围注释解释了这一段的局部作用：Define the CTA-in-cluster Layout and Coord。

### Lines 677-680

```cpp
    // Project the cta_layout for tma_a along the n-modes
    auto [tAgA_mkl, tAsA] = tma_partition(*observed_tma_load_a_,
                                      get<2>(cta_coord_vmnk), make_layout(size<2>(cta_layout_vmnk)),
                                      group_modes<0,3>(sA), group_modes<0,3>(tCgA_mkl));
```
**EN:** The surrounding comments explain the local purpose of this block: Project the cta_layout for tma_a along the n-modes.
**CN:** 周围注释解释了这一段的局部作用：Project the cta_layout for tma_a along the n-modes。

### Lines 682-685

```cpp
    // Project the cta_layout for tma_b along the m-modes
    auto [tBgB_nkl, tBsB] = tma_partition(*observed_tma_load_b_,
                                      get<1>(cta_coord_vmnk), make_layout(size<1>(cta_layout_vmnk)),
                                      group_modes<0,3>(sB), group_modes<0,3>(tCgB_nkl));
```
**EN:** The surrounding comments explain the local purpose of this block: Project the cta_layout for tma_b along the m-modes.
**CN:** 周围注释解释了这一段的局部作用：Project the cta_layout for tma_b along the m-modes。

### Lines 687-690

```cpp
    // Project the cta_layout for tma_a along the n-modes
    auto [tEgE_mkl, tEsE] = tma_partition(*observed_tma_load_e_,
                                      get<2>(cta_coord_vmnk), make_layout(size<2>(cta_layout_vmnk)),
                                      group_modes<0,3>(sE), group_modes<0,3>(tCgE_mkl));
```
**EN:** The surrounding comments explain the local purpose of this block: Project the cta_layout for tma_a along the n-modes.
**CN:** 周围注释解释了这一段的局部作用：Project the cta_layout for tma_a along the n-modes。

### Lines 692-695

```cpp
    // TMA Multicast Masks
    uint16_t mcast_mask_a = create_tma_multicast_mask<2>(cta_layout_vmnk, cta_coord_vmnk);
    uint16_t mcast_mask_b = create_tma_multicast_mask<1>(cta_layout_vmnk, cta_coord_vmnk);
    uint16_t mcast_mask_e = create_tma_multicast_mask<2>(cta_layout_vmnk, cta_coord_vmnk);
```
**EN:** The surrounding comments explain the local purpose of this block: TMA Multicast Masks.
**CN:** 周围注释解释了这一段的局部作用：TMA Multicast Masks。

### Lines 697-701

```cpp
    return LoadParams{
      size<3>(gA_mkl), size<3>(gE_mkl),               // for scheduler
      tAgA_mkl, tBgB_nkl, tEgE_mkl, tAsA, tBsB, tEsE, // for input tensor values
      mcast_mask_a, mcast_mask_b, mcast_mask_e};      // multicast masks
  }
```
**EN:** This helper structure packages the values needed by a later load or MMA phase so the call sites stay organized. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这个辅助结构把后续 load 或 MMA 阶段需要的值打包起来，使调用点保持清晰。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 703-708

```cpp
  /// Set up the data needed by this collective for mma compute.
  template <class TmemStorage>
  CUTLASS_DEVICE auto
  mma_init(
    TmemStorage tmem_storage,
    TensorStorage& shared_tensors) const {
```
**EN:** This block introduces `TmemStorage` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TmemStorage`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 710-713

```cpp
    // Allocate "fragments/descriptors" for A B E matrices
    Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.begin()), SmemLayoutA{});  // (BLK_M,BLK_K,PIPE)
    Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.begin()), SmemLayoutB{});  // (BLK_N,BLK_K,PIPE)
    Tensor sE = make_tensor(make_smem_ptr(shared_tensors.smem_E.begin()), SmemLayoutE{});  // (MMA,MMA_M,MMA_K,PIPE) that one UTCCP can provide
```
**EN:** The surrounding comments explain the local purpose of this block: Allocate "fragments/descriptors" for A B E matrices.
**CN:** 周围注释解释了这一段的局部作用：Allocate "fragments/descriptors" for A B E matrices。

### Lines 715-717

```cpp
    // Allocate "fragments/descriptors" for A and B matrices
    Tensor tCrA = TiledMma::make_fragment_A(sA);                                           // (MMA,MMA_M,MMA_K,PIPE)
    Tensor tCrB = TiledMma::make_fragment_B(sB);                                           // (MMA,MMA_N,MMA_K,PIPE)
```
**EN:** The surrounding comments explain the local purpose of this block: Allocate "fragments/descriptors" for A and B matrices.
**CN:** 周围注释解释了这一段的局部作用：Allocate "fragments/descriptors" for A and B matrices。

### Lines 719-721

```cpp
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::Stages>{} == size<3>(sA));                                     // PIPE
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::Stages>{} == size<3>(sB));                                     // PIPE
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::Stages>{} == size<3>(sE));                                     // PIPE
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 723-727

```cpp
    Tensor tCtE = tmem_storage.tCtE;
    using AtomThrID = typename TiledMma::AtomThrID;
    using UtccpEOp = cute::conditional_t<(decltype(cute::size(AtomThrID{}) == Int<2>{})::value),
      cute::SM100_UTCCP_128dp128bit_2cta, cute::SM100_UTCCP_128dp128bit_1cta>;
    auto tiled_copy_s2t_E = make_utccp_copy(UtccpEOp{}, recast<ElementE>(tCtE));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 729-733

```cpp
    auto thr_copy_s2t_E = tiled_copy_s2t_E.get_slice(0);
    Tensor thr_tCsE_s2t_ = thr_copy_s2t_E.partition_S(recast<ElementE>(sE));
    // SMEM to TMEM copy operation requires source SMEM operand to be an SMEM descriptor
    Tensor thr_tCsE_s2t = get_utccp_smem_desc_tensor<UtccpEOp>(thr_tCsE_s2t_);
    Tensor thr_tCtE_s2t = thr_copy_s2t_E.partition_D(recast<ElementE>(tCtE));
```
**EN:** The surrounding comments explain the local purpose of this block: SMEM to TMEM copy operation requires source SMEM operand to be an SMEM descriptor.
**CN:** 周围注释解释了这一段的局部作用：SMEM to TMEM copy operation requires source SMEM operand to be an SMEM descriptor。

### Lines 735-735

```cpp
    TiledMma tiled_mma;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 737-742

```cpp
    if constexpr (IsRuntimeDataType) {
      // Update instruction descriptor according to runtime argument.
      // Applying bitmask (0b111) to help compiler deduce that the conversion and assignment are safe.
      tiled_mma.idesc_.a_format_ = uint8_t(runtime_data_type_a_) & 0b111;
      tiled_mma.idesc_.b_format_ = uint8_t(runtime_data_type_b_) & 0b111;
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 744-748

```cpp
    return MmaParams{
      tiled_mma,
      tCrA, tCrB,
      tCtE, tiled_copy_s2t_E, thr_tCsE_s2t, thr_tCtE_s2t};
  }
```
**EN:** This helper structure packages the values needed by a later load or MMA phase so the call sites stay organized. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这个辅助结构把后续 load 或 MMA 阶段需要的值打包起来，使调用点保持清晰。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 750-763

```cpp
  /// Perform a collective-scoped matrix multiply-accumulate
  /// Producer Perspective
  template <
    class LoadParams,
    class TileCoordMNKL,
    class KTileIterator
  >
  CUTLASS_DEVICE auto
  load(
    MainloopPipeline mainloop_pipeline,
    MainloopPipelineState mainloop_pipe_producer_state,
    LoadParams const& load_inputs,
    TileCoordMNKL const& cta_coord_mnkl,
    KTileIterator k_tile_iter, int k_tile_count) {
```
**EN:** This block introduces `LoadParams` and groups related declarations around that symbol. This helper structure packages the values needed by a later load or MMA phase so the call sites stay organized.
**CN:** 这一段引入了 `LoadParams`，并围绕该符号组织相关声明。 这个辅助结构把后续 load 或 MMA 阶段需要的值打包起来，使调用点保持清晰。

### Lines 765-767

```cpp
    auto [k_tiles, k_tiles_metadata,
          tAgA_mkl, tBgB_nkl, tEgE_mkl, tAsA, tBsB, tEsE,
          mcast_mask_a, mcast_mask_b, mcast_mask_e] = load_inputs;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 769-772

```cpp
    // slice out the work coord from partitioned tensors
    Tensor tAgA = tAgA_mkl(_, get<0>(cta_coord_mnkl) / size(typename TiledMma::AtomThrID{}), _, get<3>(cta_coord_mnkl));
    Tensor tEgE = tEgE_mkl(_, get<0>(cta_coord_mnkl) / size(typename TiledMma::AtomThrID{}), _, get<3>(cta_coord_mnkl));
    Tensor tBgB = tBgB_nkl(_, get<1>(cta_coord_mnkl), _, get<3>(cta_coord_mnkl));
```
**EN:** The surrounding comments explain the local purpose of this block: slice out the work coord from partitioned tensors.
**CN:** 周围注释解释了这一段的局部作用：slice out the work coord from partitioned tensors。

### Lines 774-775

```cpp
    auto barrier_token = mainloop_pipeline.producer_try_acquire(mainloop_pipe_producer_state);
    uint32_t iter = 0;
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 777-779

```cpp
    // K_tile_iter for E
    auto k_tile_start = cute::crd2idx(k_tile_iter.coord, k_tiles);
    auto k_utccp_tile_iter = cute::make_coord_iterator(idx2crd(k_tile_start / UtccpReuseCnt, k_tiles_metadata), k_tiles_metadata);
```
**EN:** The surrounding comments explain the local purpose of this block: K_tile_iter for E.
**CN:** 周围注释解释了这一段的局部作用：K_tile_iter for E。

### Lines 781-784

```cpp
    // Issue the Mainloop loads
    CUTLASS_PRAGMA_NO_UNROLL
    while (k_tile_count > 0) {
      bool load_e = iter % UtccpReuseCnt == 0;
```
**EN:** This comment marks the mainloop logic that repeatedly loads tiles and issues MMA operations.
**CN:** 这条注释标记了主循环逻辑：重复加载 tile 并发起 MMA 运算。

### Lines 786-789

```cpp
      // LOCK mainloop_pipe_producer_state for _writing_
      mainloop_pipeline.producer_acquire(mainloop_pipe_producer_state, load_e, barrier_token);
      // Note: We don't synchronize the sf_pipeline for "Buffer_Empty". We use mainloop pipeline
      // to do the synchronization at once.
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 791-792

```cpp
      using BarrierType = typename MainloopPipeline::ProducerBarrierType;
      BarrierType* tma_barrier = mainloop_pipeline.producer_get_barrier(mainloop_pipe_producer_state);
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 794-796

```cpp
      int write_stage = mainloop_pipe_producer_state.index();
      ++mainloop_pipe_producer_state;
      barrier_token = mainloop_pipeline.producer_try_acquire(mainloop_pipe_producer_state);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 798-801

```cpp
      if (cute::elect_one_sync()) {
        copy(observed_tma_load_a_->with(*tma_barrier, mcast_mask_a), tAgA(_,*k_tile_iter), tAsA(_,write_stage));
        copy(observed_tma_load_b_->with(*tma_barrier, mcast_mask_b), tBgB(_,*k_tile_iter), tBsB(_,write_stage));
      }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 803-808

```cpp
      if (load_e) {
        if (cute::elect_one_sync()) {
          copy(observed_tma_load_e_->with(*tma_barrier, mcast_mask_e), tEgE(_,*k_utccp_tile_iter), tEsE(_,write_stage));
        }
        ++k_utccp_tile_iter;
      }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 810-813

```cpp
      ++k_tile_iter;
      --k_tile_count;
      iter++;
    }
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 815-816

```cpp
    return cute::make_tuple(mainloop_pipe_producer_state, k_tile_iter);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 818-828

```cpp
  /// Perform a Producer Epilogue to prevent early exit of ctas in a Cluster
  CUTLASS_DEVICE void
  load_tail(MainloopPipeline mainloop_pipeline, MainloopPipelineState mainloop_pipe_producer_state) {
    // Issue the epilogue waits
    // This helps avoid early exit of ctas in Cluster
    // Waits for all stages to either be released (all
    // Consumer UNLOCKs), or if the stage was never used
    // then would just be acquired since the phase was
    // still inverted from make_producer_start_state
    mainloop_pipeline.producer_tail(mainloop_pipe_producer_state);
  }
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 830-849

```cpp
  /// Perform a collective-scoped matrix multiply-accumulate
  /// Consumer Perspective
  template <
    class AccumulatorPipeline,
    class FrgEngine, class FrgLayout,
    class MmaParams,
    class CtaTileCoord
  >
  CUTLASS_DEVICE auto
  mma(cute::tuple<MainloopPipeline,
                  AccumulatorPipeline> pipelines,
      cute::tuple<MainloopPipelineState,
                  typename AccumulatorPipeline::PipelineState> pipeline_states,
      cute::tuple<cute::Tensor<FrgEngine, FrgLayout>> const& accumulators_pair,
      MmaParams const& mma_inputs,
      CtaTileCoord cta_tile_coord,
      int k_tile_count
  ) {
    static_assert(is_tmem<FrgEngine>::value, "Accumulator must be tmem resident.");
    static_assert(rank(FrgLayout{}) == 3, "Accumulator must be MMA-partitioned: (MMA, MMA_M, MMA_N)");
```
**EN:** This block introduces `AccumulatorPipeline` and groups related declarations around that symbol. The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这一段引入了 `AccumulatorPipeline`，并围绕该符号组织相关声明。 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 851-854

```cpp
    auto accumulators = get<0>(accumulators_pair);
    auto [tiled_mma,
          tCrA, tCrB,
          tCtE, tiled_copy_s2t_E, thr_tCsE_s2t, thr_tCtE_s2t ] = mma_inputs;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 856-857

```cpp
    auto [mainloop_pipeline, accumulator_pipeline] = pipelines;
    auto [mainloop_pipe_consumer_state, accumulator_pipe_producer_state] = pipeline_states;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 859-862

```cpp
    uint32_t skip_wait = k_tile_count <= 0;
    auto barrier_token = mainloop_pipeline.consumer_try_wait(mainloop_pipe_consumer_state, skip_wait);
    uint32_t math_mma_e_stage_idx = 0;
    uint32_t iter = 0;
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 864-871

```cpp
    //
    // PIPELINED MAIN LOOP
    //
    tiled_mma.accumulate_ = UMMA::ScaleOut::Zero;
    if constexpr (not IsOverlappingAccum) {
      // Wait for tmem accumulator buffer to become empty with a flipped phase
      accumulator_pipeline.producer_acquire(accumulator_pipe_producer_state);
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 873-877

```cpp
    CUTLASS_PRAGMA_NO_UNROLL
    while (k_tile_count > 0) {
      // WAIT on mainloop_pipe_consumer_state until its data are available
      // (phase bit flips from mainloop_pipe_consumer_state.phase() value)
      mainloop_pipeline.consumer_wait(mainloop_pipe_consumer_state, barrier_token);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 879-882

```cpp
      // Compute on k_tile
      int read_stage = mainloop_pipe_consumer_state.index();
      // Save current mainlop pipeline read state
      auto curr_mainloop_pipe_consumer_state = mainloop_pipe_consumer_state;
```
**EN:** The surrounding comments explain the local purpose of this block: Compute on k_tile Save current mainlop pipeline read state.
**CN:** 周围注释解释了这一段的局部作用：Compute on k_tile Save current mainlop pipeline read state。

### Lines 884-889

```cpp
      // Advance mainloop_pipe
      ++mainloop_pipe_consumer_state;
      --k_tile_count;
      skip_wait = k_tile_count <= 0;
      // Peek at next iteration
      barrier_token = mainloop_pipeline.consumer_try_wait(mainloop_pipe_consumer_state, skip_wait);
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 891-902

```cpp
      if constexpr (UtccpReuseCnt == 1) {
        if (cute::elect_one_sync()) {
          copy(tiled_copy_s2t_E, thr_tCsE_s2t(_,_,_,_,read_stage), thr_tCtE_s2t);
        }
      }
      else {
        if (not (iter & 1)) {
          if (cute::elect_one_sync()) {
            copy(tiled_copy_s2t_E, thr_tCsE_s2t(_,_,_,_,read_stage), thr_tCtE_s2t);
          }
        }
      }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 904-908

```cpp
      if constexpr (IsOverlappingAccum) {
        if (iter == 0) {
          accumulator_pipeline.producer_acquire(accumulator_pipe_producer_state);
        }
      }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 910-919

```cpp
      // Unroll the K mode manually so we can set scale C to 1
      CUTLASS_PRAGMA_UNROLL
      for (int k_block = 0; k_block < size<2>(tCrA); ++k_block) {
        // (V,M) x (V,N) => (V,M,N)
        cute::gemm(tiled_mma.with(tCtE(_,_,math_mma_e_stage_idx * UtccpReuseCnt + k_block)),
            tCrA(_,_,k_block,read_stage),
            tCrB(_,_,k_block,read_stage),
            accumulators);
        tiled_mma.accumulate_ = UMMA::ScaleOut::One;
      }
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine. Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 921-924

```cpp
      if constexpr (UtccpReuseCnt != 1) {
        // Each E Smem Stage contain two CtaK's Metadata when UtccpReuse
        math_mma_e_stage_idx ^= 1;
      }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 926-928

```cpp
      mainloop_pipeline.consumer_release(curr_mainloop_pipe_consumer_state);
      ++iter;
    }
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 930-931

```cpp
    return mainloop_pipe_consumer_state;
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 933-933

```cpp
protected:
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 935-941

```cpp
  typename Params::TMA_A const* observed_tma_load_a_{nullptr};
  typename Params::TMA_E const* observed_tma_load_e_{nullptr};
  typename Params::TMA_B const* observed_tma_load_b_{nullptr};
  LayoutA layout_a_;
  LayoutE layout_e_;
  RuntimeDataTypeA runtime_data_type_a_{};
  RuntimeDataTypeB runtime_data_type_b_{};
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 943-945

```cpp
  ClusterShape cluster_shape_;
  uint32_t block_rank_in_cluster_;
};
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 947-947

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 949-949

```cpp
} // namespace cutlass::gemm::collective
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 951-951

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

## Key Concepts / 关键概念

- **EN:** TMA data movement  
  **CN:** TMA 数据搬运
- **EN:** UMMA tensor-core instructions  
  **CN:** UMMA 张量核指令
- **EN:** warp-specialized scheduling  
  **CN:** warp-specialized 调度
- **EN:** sparse MMA support  
  **CN:** 稀疏 MMA 支持
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
  - `cutlass/detail/collective.hpp`
  - `cutlass/detail/cluster.hpp`
  - `cutlass/gemm/dispatch_policy.hpp`
  - `cutlass/numeric_types.h`
  - `cutlass/pipeline/pipeline.hpp`
  - `cutlass/gemm/gemm.h`
  - `cutlass/gemm/collective/builders/sm1xx_sparse_config.inl`
  - `cutlass/trace.h`
  - `cutlass/kernel_hardware_info.hpp`
  - `cutlass/detail/sm100_tmem_helper.hpp`
  - `cute/algorithm/functional.hpp`
  - `cute/arch/cluster_sm90.hpp`
  - `cute/atom/mma_atom.hpp`
  - `cute/algorithm/gemm.hpp`
  - `cute/numeric/arithmetic_tuple.hpp`
- **Primary symbols / 主要符号:**
  - `ClusterShape`
  - `TileShape_`
  - `ElementA_`
  - `LayoutPairAE_`
  - `ElementB_`
  - `StrideB_`
  - `TiledMma_`
  - `GmemTiledCopyA_`
  - `SmemLayoutAtomA_`
  - `SmemCopyAtomA_`
  - `TransformA_`
  - `GmemTiledCopyB_`
  - `SmemLayoutAtomB_`
  - `SmemCopyAtomB_`
  - `TransformB_`
  - `CollectiveMma`
- **Shared abstractions / 共享抽象:** `CollectiveMma` / `CollectiveBuilder`, dispatch-policy tags, CuTe tensor/layout utilities, and CUTLASS pipeline helpers. / `CollectiveMma` / `CollectiveBuilder`、dispatch-policy 标签、CuTe 张量/布局工具以及 CUTLASS 流水线辅助组件。