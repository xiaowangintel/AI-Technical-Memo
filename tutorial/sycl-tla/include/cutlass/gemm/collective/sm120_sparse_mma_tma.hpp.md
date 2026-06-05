# sm120_sparse_mma_tma.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/collective/sm120_sparse_mma_tma.hpp`
- **Purpose (EN):** Implements an architecture-specific `CollectiveMma` specialization for SM120, covering sparse metadata handling, TMA transfers.
- **用途 (CN):** 为 SM120 实现架构特化的 `CollectiveMma` 特化版本，重点覆盖 稀疏元数据处理、TMA 传输。
- **Lines / 行数:** 988

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

### Lines 32-32

```cpp
#pragma once
```
**EN:** `#pragma once` prevents duplicate inclusion of the header during compilation.
**CN:** `#pragma once` 用于防止编译过程中重复包含该头文件。

### Lines 34-41

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

### Lines 43-48

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

### Lines 50-50

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 52-53

```cpp
namespace cutlass::gemm::collective {
using namespace cute;
```
**EN:** This short block both opens the collective namespace and pulls CuTe symbols into scope, setting up a concise vocabulary for the rest of the file.
**CN:** 这个短代码块既打开了 collective 命名空间，也把 CuTe 符号引入作用域，为后续代码建立更简洁的表达方式。

### Lines 55-55

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 57-127

```cpp
template <
  int StagesA,
  int StagesB,
  int StagesE,
  int SchedulerPipelineStageCount,
  class ClusterShape,
  class TileShape_,
  class ElementA_,
  class LayoutPairAE_,
  class ElementB_,
  class StrideB_,
  class TiledMma_,
  class GmemTiledCopyA_,
  class SmemLayoutAtomA_,
  class SmemCopyAtomPairA_,
  class TransformA_,
  class GmemTiledCopyB_,
  class SmemLayoutAtomB_,
  class SmemCopyAtomB_,
  class TransformB_>
struct CollectiveMma<
    MainloopSm120TmaWarpSpecializedSparse<StagesA, StagesB, StagesE, SchedulerPipelineStageCount, ClusterShape>,
    TileShape_,
    ElementA_,
    LayoutPairAE_,
    ElementB_,
    StrideB_,
    TiledMma_,
    GmemTiledCopyA_,
    SmemLayoutAtomA_,
    SmemCopyAtomPairA_,
    TransformA_,
    GmemTiledCopyB_,
    SmemLayoutAtomB_,
    SmemCopyAtomB_,
    TransformB_> {
  //
  // Type Aliases
  //
  using TiledMma = TiledMma_;
  using AtomThrShapeMNK = Shape<decltype(shape<0>(typename TiledMma::ThrLayoutVMNK{})), _1, _1>;
  using DispatchPolicy = MainloopSm120TmaWarpSpecializedSparse<StagesA, StagesB, StagesE, SchedulerPipelineStageCount, ClusterShape>;
  using TileShape = TileShape_;
  using ElementA = ElementA_;
  using ElementAMma = typename TiledMma::ValTypeA;
  using ElementAMmaRaw = typename ElementAMma::raw_type;
  using LayoutPairAE = LayoutPairAE_;
  using LayoutA =  remove_cvref_t<decltype(get<0>(LayoutPairAE{}))>;
  using LayoutE =  remove_cvref_t<decltype(get<1>(LayoutPairAE{}))>;
  using StrideA =  remove_cvref_t<decltype(get<2>(LayoutPairAE{}))>;
  using ElementB = ElementB_;
  using StrideB = StrideB_;
  using ElementBMma = typename TiledMma::ValTypeB;
  using ElementEMma = typename TiledMma::ValTypeE;
  using ElementE = typename ElementEMma::raw_type;
  using CtaShape_MNK = decltype(shape_div(TileShape{}, ClusterShape{}));
  using ElementAccumulator = typename TiledMma::ValTypeC;
  using GmemTiledCopyA = GmemTiledCopyA_;
  using GmemTiledCopyB = GmemTiledCopyB_;
  using SmemLayoutAtomA = SmemLayoutAtomA_;
  using SmemLayoutAtomB = SmemLayoutAtomB_;
  using SmemCopyAtomA = remove_cvref_t<decltype(get<0>(SmemCopyAtomPairA_{}))>;
  using SmemCopyAtomE = remove_cvref_t<decltype(get<1>(SmemCopyAtomPairA_{}))>;
  using SmemCopyAtomB = SmemCopyAtomB_;
  using TransformA = TransformA_;
  using TransformB = TransformB_;
  using ArchTag = typename DispatchPolicy::ArchTag;
  using GmemTiledCopyE = GmemTiledCopyA_;
  using ArrayElementA = ElementA;
  using ArrayElementB = ElementB;
  using RegisterE = typename remove_extent<typename TiledMma::MMA_Op::ERegisters>::type;
```
**EN:** This template block declares or specializes `CollectiveMma`, the mainloop object that coordinates tile movement and matrix-multiply work.
**CN:** 这个模板块声明或特化了 `CollectiveMma`，它负责协调 tile 搬运与矩阵乘主循环。

### Lines 129-130

```cpp
  using RuntimeDataTypeA = void*;
  using RuntimeDataTypeB = void*;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 132-134

```cpp
  static constexpr int ThreadCount = size(TiledMma{});
  static constexpr int ElementAMmaSparsity = ElementAMma::sparsity;
  static constexpr int ElementEMmaSparsity = ElementEMma::sparsity;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 136-140

```cpp
  // Asymmetric buffering
  // Tensor A/B could have different buffering, with TILEK, and STAGEs.
  //    It let AsymmetricKRatio equals TILEK_A / TILEK_B, to make sure A/B's
  //    pipeline keep same steps when produce / consume data.
  static constexpr int AsymmetricKRatio = DispatchPolicy::StagesA != DispatchPolicy::StagesB ? 2 : 1;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 142-144

```cpp
  using TileShapeB = decltype(make_shape(size<0>(TileShape{}),
                                         size<1>(TileShape{}),
                                         ceil_div(size<2>(TileShape{}), Int<AsymmetricKRatio>{})));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 146-148

```cpp
  // Use two MainloopPipeline for A and B separately.
  using MainloopPipelineMK = cutlass::PipelineTmaAsync<DispatchPolicy::StagesA>;
  using MainloopPipelineNK = cutlass::PipelineTmaAsync<DispatchPolicy::StagesB>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 150-152

```cpp
  using PipelineParams = typename MainloopPipelineMK::Params;
  using PipelineStateMK  = typename cutlass::PipelineState<DispatchPolicy::StagesA>;
  using PipelineStateNK  = typename cutlass::PipelineState<DispatchPolicy::StagesB>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 154-156

```cpp
  static_assert(rank(SmemLayoutAtomA{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 158-160

```cpp
  static_assert(rank(SmemLayoutAtomB{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<1>(TileShape{}) % size<0>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 162-165

```cpp
  static_assert(not cute::is_void_v<SmemCopyAtomA>,
    "SM120 mainloop must specify a copy atom for A operand smem->rmem reads.");
  static_assert(not cute::is_void_v<SmemCopyAtomB>,
    "SM120 mainloop must specify a copy atom for B operand smem->rmem reads.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 167-168

```cpp
  static_assert(DispatchPolicy::StagesA >= 2, "Specialization requires Stages set to value 2 or more.");
  static_assert(DispatchPolicy::StagesB >= 2, "Specialization requires Stages set to value 2 or more.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 170-178

```cpp
  // Tile along modes in a way that maximizes the TMA box size.
  using SmemLayoutA = decltype(tile_to_shape(
      SmemLayoutAtomA{},
      make_shape(shape<0>(TileShape{}), shape<2>(TileShape{}), Int<DispatchPolicy::StagesA>{}),
      conditional_t< ::cutlass::gemm::detail::is_major<0,StrideA>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
  using SmemLayoutB = decltype(tile_to_shape(
      SmemLayoutAtomB{},
      make_shape(shape<1>(TileShapeB{}), shape<2>(TileShapeB{}), Int<DispatchPolicy::StagesB>{}),
      conditional_t< ::cutlass::gemm::detail::is_major<0,StrideB>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 180-181

```cpp
  static_assert(rank(SmemLayoutA{}) == 3, "Smem layout must be rank 3.");
  static_assert(rank(SmemLayoutB{}) == 3, "Smem layout must be rank 3.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 183-189

```cpp
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

### Lines 191-191

```cpp
  static constexpr bool IsF8F6F4 = detail::is_sm100_sparse_f8f6f4<TiledMma, ElementA, ElementB>();
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 193-194

```cpp
  // Is E kept in SMEM or GMEM
  static constexpr bool UseSmemE = DispatchPolicy::StagesE != 0;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 196-205

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
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 207-215

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

### Lines 217-219

```cpp
  // Set shared memory layout
  using SmemAllocTypeA = cute::conditional_t<IsF8F6F4, sparse_elem<ElementAMmaSparsity, uint8_t>, ElementAMma>;
  using SmemAllocTypeB = cute::conditional_t<IsF8F6F4, uint8_t, ElementBMma>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 221-239

```cpp
  static constexpr bool is_A_mn_major = cute::is_same_v<decltype(stride<0>(LayoutA{})), Int<ElementAMmaSparsity>>;
  using SparseConfig = cutlass::Sm1xxGemmSparseConfig<
                                    ElementAMma,
                                    cute::conditional_t<is_A_mn_major, cutlass::layout::ColumnMajor, cutlass::layout::RowMajor>,
                                    ElementEMma>;
  using SmemLayoutAtomE_ = typename SparseConfig::TensorEAtom;
  using SmemLayoutAtomE  = ComposedLayout<Swizzle<0,4,3>,
                                          smem_sparse_ptr_flag_bits<ElementEMmaSparsity, sizeof_bits_v<ElementE>>,
                                          SmemLayoutAtomE_>;
  using SmemLayoutE = decltype(tile_to_shape(
                  SmemLayoutAtomE{},
                  make_shape(shape<0>(TileShape{}), shape<2>(TileShape{}), Int<DispatchPolicy::StagesE>{}),
                  conditional_t< ::cutlass::gemm::detail::is_major<0,StrideA>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
  static constexpr int SmemSizeE  = UseSmemE ? cosize(SmemLayoutE{}) : 0;
  static constexpr int StageSizeE = UseSmemE ? cosize(take<0,2>(SmemLayoutE{})) : 0;
  // Check if metetata fetching needs predicator
  using TensorEAtomM = typename SparseConfig::TensorEAtomM;
  using TensorEAtomK = typename SparseConfig::TensorEAtomK;
  static constexpr bool IsELoadPred = not (TensorEAtomM{} == size<0>(TileShape{}) && TensorEAtomK{} == size<2>(TileShape{}));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 241-242

```cpp
  static_assert(rank(SmemLayoutAtomE{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomE{})) == 0, "SmemLayoutAtomE must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 244-250

```cpp
  // Set the bytes transferred in this TMA transaction
  static constexpr uint32_t TmaTransactionBytesMK = static_cast<uint32_t>(
    cutlass::bits_to_bytes(cosize(take<0,2>(SmemLayoutA{})) * cute::sizeof_bits_v<ElementAMma>) +
    cutlass::bits_to_bytes(StageSizeE * cute::sizeof_bits_v<ElementEMma>));
  static constexpr uint32_t TmaTransactionBytesNK = static_cast<uint32_t>(
    cutlass::bits_to_bytes(cosize(take<0,2>(SmemLayoutB{})) * cute::sizeof_bits_v<ElementB>));
  static constexpr uint32_t TmaTransactionBytes = TmaTransactionBytesMK + TmaTransactionBytesNK;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 252-257

```cpp
  struct SharedStorage {
    struct TensorStorage : cute::aligned_struct<128, _0> {
      alignas(1024) cute::ArrayEngine<SmemAllocTypeA, cute::cosize_v<SmemLayoutA>> smem_A;
      alignas(1024) cute::ArrayEngine<SmemAllocTypeB, cute::cosize_v<SmemLayoutB>> smem_B;
      cute::ArrayEngine<ElementEMma, Int<SmemSizeE>{}> smem_E;
    } tensors;
```
**EN:** This block introduces `SharedStorage` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `SharedStorage`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 259-266

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

### Lines 268-275

```cpp
  struct Arguments {
    ElementA const* ptr_A{nullptr};
    LayoutA layout_a{};
    ElementB const* ptr_B{nullptr};
    StrideB dB{};
    ElementE const* ptr_E{nullptr};
    LayoutE layout_e{};
  };
```
**EN:** This block introduces `Arguments` and groups related declarations around that symbol. `Arguments` is the host-side bundle: callers fill it with pointers, strides, and other launch-time values before parameter lowering.
**CN:** 这一段引入了 `Arguments`，并围绕该符号组织相关声明。 `Arguments` 是主机侧参数包：调用者在参数下沉前把指针、步长及其他启动期数据填入其中。

### Lines 277-285

```cpp
  // Device side kernel params
  struct Params {
    // Assumption: StrideA is congruent with Problem_MK
    using TMA_A = decltype(make_tma_copy<TmaInternalElementA>(
        GmemTiledCopyA{},
        make_tensor(recast_ptr<sparse_elem<ElementAMmaSparsity,ElementA>>(nullptr), LayoutA{}),
        SmemLayoutA{}(_,_,0),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        _1{}));
```
**EN:** This block introduces `Params` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `Params`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 287-309

```cpp
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
    TMA_A tma_load_a;
    TMA_B tma_load_b;
    TMA_E tma_load_e;
    LayoutA layout_a;
    LayoutE layout_e;
    ElementE const* ptr_E{nullptr};
    uint32_t tma_transaction_bytes_mk = TmaTransactionBytesMK;
    uint32_t tma_transaction_bytes_nk = TmaTransactionBytesNK;
    uint32_t tma_transaction_bytes = TmaTransactionBytes;
  };
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 311-313

```cpp
  //
  // Methods
  //
```
**EN:** This comment starts a method section where the collective exposes its runtime behavior.
**CN:** 这条注释表示进入方法区，collective 的运行时行为将在这里定义。

### Lines 315-318

```cpp
  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(ProblemShape const& problem_shape, Arguments const& args, void* workspace) {
    (void) workspace;
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 320-322

```cpp
    // Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK)
    auto problem_shape_MNKL = append<4>(problem_shape, 1);
    auto [M, N, K, L] = problem_shape_MNKL;
```
**EN:** The surrounding comments explain the local purpose of this block: Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK).
**CN:** 周围注释解释了这一段的局部作用：Optionally append 1s until problem shape is rank-4 (MNKL), in case it is only rank-3 (MNK)。

### Lines 324-326

```cpp
    auto ptr_A = recast_ptr<sparse_elem<ElementAMmaSparsity, ElementA>>(args.ptr_A);
    auto ptr_B = recast_ptr<TmaInternalElementB>(args.ptr_B);
    auto ptr_E = recast_ptr<ElementEMma>(args.ptr_E);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 328-357

```cpp
    Tensor tensor_a = make_tensor(ptr_A, args.layout_a);
    Tensor tensor_b = make_tensor(ptr_B, make_layout(make_shape(N,K,L), args.dB));
    Tensor tensor_e = make_tensor(ptr_E, args.layout_e);
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
        make_shape(shape<1>(TileShapeB{}), shape<2>(TileShapeB{})),
        _1{});
    typename Params::TMA_E tma_load_e = make_tma_copy<ElementE>(
        GmemTiledCopyE{},
        tensor_e,
        SmemLayoutE{}(_,_,cute::Int<0>{}),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        _1{});
    return {
      tma_load_a,
      tma_load_b,
      tma_load_e,
      args.layout_a,
      args.layout_e,
      args.ptr_E
    };
  }
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 359-365

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

### Lines 367-368

```cpp
    constexpr int tma_alignment_bits_A = cutlass::detail::get_input_alignment_bits<ElementA, IsF8F6F4>();
    constexpr int tma_alignment_bits_B = cutlass::detail::get_input_alignment_bits<ElementB, IsF8F6F4>();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 370-374

```cpp
    bool implementable = true;
    constexpr int min_tma_aligned_elements_A = tma_alignment_bits_A / cutlass::sizeof_bits<ElementA>::value;
    implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_A>(cute::upcast<2>(make_layout(make_shape(M, K, L), StrideA{})));
    constexpr int min_tma_aligned_elements_B = tma_alignment_bits_B / cutlass::sizeof_bits<ElementB>::value;
    implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_B>(cute::make_shape(N,K,L), StrideB{});
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 376-380

```cpp
    if (!implementable) {
      CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem Size doesn't meet the minimum alignment requirements for TMA.\n");
    }
    return implementable;
  }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 382-390

```cpp
  /// Issue Tma Descriptor Prefetch -- ideally from a single thread for best performance
  CUTLASS_DEVICE
  static void prefetch_tma_descriptors(Params const& mainloop_params) {
    cute::prefetch_tma_descriptor(mainloop_params.tma_load_a.get_tma_descriptor());
    cute::prefetch_tma_descriptor(mainloop_params.tma_load_b.get_tma_descriptor());
    if constexpr (UseSmemE) {
      cute::prefetch_tma_descriptor(mainloop_params.tma_load_e.get_tma_descriptor());
    }
  }
```
**EN:** The code prefetches descriptor state early so later TMA or copy operations can start with lower latency. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这里会提前预取描述符状态，从而让后续 TMA 或拷贝操作以更低延迟启动。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 392-397

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

### Lines 399-400

```cpp
    using AtomShape_MNK  = typename Atom::Shape_MNK;
    using AtomLayoutE_TV = typename Atom::Traits::ELayout;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 402-405

```cpp
    auto t_tile = make_tile(get<0>(TiledPerm{}),
                            get<2>(TiledPerm{}));
    auto thr_layout_vmnk = mma.get_thr_layout_vmnk();
    auto t_tensor = logical_divide(tensor, t_tile);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 407-410

```cpp
    // Tile the tensor for the Atom
    auto e_tile = make_tile(make_layout(size<0>(AtomShape_MNK{})),
                            make_layout(size<2>(AtomShape_MNK{})));
    auto e_tensor = zipped_divide(t_tensor, e_tile);                                   // ((AtomM,AtomK),(RestM,RestK))
```
**EN:** The surrounding comments explain the local purpose of this block: Tile the tensor for the Atom.
**CN:** 周围注释解释了这一段的局部作用：Tile the tensor for the Atom。

### Lines 412-413

```cpp
    // Transform the Atom mode from (M,K) to (Thr,Val)
    auto tv_tensor = e_tensor.compose(AtomLayoutE_TV{},_);                               // ((ThrV,FrgV),(RestM,RestK))
```
**EN:** The surrounding comments explain the local purpose of this block: Transform the Atom mode from (M,K) to (Thr,Val).
**CN:** 周围注释解释了这一段的局部作用：Transform the Atom mode from (M,K) to (Thr,Val)。

### Lines 415-419

```cpp
    // Tile the tensor for the Thread
    auto thr_tile = make_tile(_,
                              make_tile(make_layout(size<1>(thr_layout_vmnk)),
                                        make_layout(size<3>(thr_layout_vmnk))));
    auto thr_tensor = zipped_divide(tv_tensor, thr_tile);                  // ((ThrV,(ThrM,ThrK)),(FrgV,(RestM,RestK)))
```
**EN:** The surrounding comments explain the local purpose of this block: Tile the tensor for the Thread.
**CN:** 周围注释解释了这一段的局部作用：Tile the tensor for the Thread。

### Lines 421-423

```cpp
    // Fragment layout
    return thr_tensor;
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 425-434

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

### Lines 436-440

```cpp
      // (ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK))
      auto etile = make_tile(_,
                            make_tile(make_layout(make_shape (size<1>(thr_layout_vmnk), size<2>(thr_layout_vmnk)),
                                                  make_stride(               Int<1>{} ,                Int<0>{} )),
                                      _));
```
**EN:** The surrounding comments explain the local purpose of this block: (ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK)).
**CN:** 周围注释解释了这一段的局部作用：(ThrV,(ThrM,ThrK)) -> (ThrV,(ThrM,ThrN,ThrK))。

### Lines 442-446

```cpp
      // thr_idx -> (ThrV,ThrM,ThrN,ThrK)
      auto thridx_2_thrid = right_inverse(thr_layout_vmnk);
      // (thr_idx,val) -> (M,K)
      return thrfrg_E(ref_E, mma).compose(etile, _).compose(thridx_2_thrid, _);
  }
```
**EN:** The surrounding comments explain the local purpose of this block: thr_idx -> (ThrV,ThrM,ThrN,ThrK) (thr_idx,val) -> (M,K).
**CN:** 周围注释解释了这一段的局部作用：thr_idx -> (ThrV,ThrM,ThrN,ThrK) (thr_idx,val) -> (M,K)。

### Lines 448-454

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

### Lines 456-459

```cpp
    auto thr_vmk = make_coord(get<0>(thr_vmnk), make_coord(get<1>(thr_vmnk), get<3>(thr_vmnk)));
    auto partition = thr_tensor(thr_vmk, make_coord(_, repeat<rank<1,1>(thr_tensor)>(_)));
    return make_fragment_like<ThrMma::Atom::Traits::ValTypeE>(partition.layout());
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 461-472

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

### Lines 474-478

```cpp
    // TMA requires special handling of strides to deal with coord codomain mapping
    // Represent the full tensors -- get these from TMA
    Tensor mA_mkl = mainloop_params.tma_load_a.get_tma_tensor(mainloop_params.layout_a.shape());             // (m,k,l)
    Tensor mB_nkl = mainloop_params.tma_load_b.get_tma_tensor(make_shape(N,K,L));                            // (n,k,l)
    Tensor mE_mkl = mainloop_params.tma_load_e.get_tma_tensor(mainloop_params.layout_e.shape());             // (m,k,l)
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 480-485

```cpp
    // Make tiled views, defer the slice
    Tensor gA_mkl = local_tile(mA_mkl, TileShape{},  make_coord(_,_,_), Step<_1, X,_1>{});       // (BLK_M,BLK_K,m,k,l)
    Tensor gB_nkl = local_tile(mB_nkl, TileShapeB{}, make_coord(_,_,_), Step< X,_1,_1>{});       // (BLK_N,BLK_K,n,k,l)
    Tensor gE_mkl = local_tile(mE_mkl, TileShape{},  make_coord(_,_,_), Step<_1, X,_1>{});       // (BLK_N,BLK_K,n,k,l)
    return cute::make_tuple(gA_mkl, gB_nkl, gE_mkl);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 487-502

```cpp
  /// Issues loads for A/E only (used when DMA warp is split).
  template <
    class TensorA, class TensorB, class TensorE,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  load_MK(
      Params const& mainloop_params,
      MainloopPipelineMK pipeline,
      PipelineStateMK smem_pipe_write,
      cute::tuple<TensorA, TensorB, TensorE> const& load_inputs,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
```
**EN:** This block introduces `TensorA` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TensorA`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 504-505

```cpp
    Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.begin()), SmemLayoutA{});         // (BLK_M,BLK_K,PIPE)
    Tensor sE = make_tensor(make_smem_ptr(shared_tensors.smem_E.begin()), SmemLayoutE{});         // (BLK_M,BLK_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 507-511

```cpp
    // Prepare the TMA loads for A and B
    Tensor gA_mkl = get<0>(load_inputs);
    Tensor gE_mkl = get<2>(load_inputs);
    auto block_tma_a = mainloop_params.tma_load_a.get_slice(0);
    auto block_tma_e = mainloop_params.tma_load_e.get_slice(0);
```
**EN:** The surrounding comments explain the local purpose of this block: Prepare the TMA loads for A and B.
**CN:** 周围注释解释了这一段的局部作用：Prepare the TMA loads for A and B。

### Lines 513-516

```cpp
    // Partition the inputs based on the current block coordinates.
    auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
    Tensor gA = gA_mkl(_,_,m_coord,_,l_coord);                                                     // (BLK_M,BLK_K,  k)
    Tensor gE = gE_mkl(_,_,m_coord,_,l_coord);                                                     // (BLK_M,BLK_K,  k)
```
**EN:** The surrounding comments explain the local purpose of this block: Partition the inputs based on the current block coordinates..
**CN:** 周围注释解释了这一段的局部作用：Partition the inputs based on the current block coordinates.。

### Lines 518-522

```cpp
    // Applies the mapping from block_tma_a
    Tensor tAgA = block_tma_a.partition_S(gA);                                                // (TMA,TMA_M,TMA_K,   k)
    Tensor tAsA = block_tma_a.partition_D(sA);                                                // (TMA,TMA_M,TMA_K,PIPE)
    Tensor tEgE = block_tma_e.partition_S(gE);                                                // (TMA,TMA_M,TMA_K,   k)
    Tensor tEsE = block_tma_e.partition_D(sE);                                                // (TMA,TMA_M,TMA_K,PIPE)
```
**EN:** The surrounding comments explain the local purpose of this block: Applies the mapping from block_tma_a.
**CN:** 周围注释解释了这一段的局部作用：Applies the mapping from block_tma_a。

### Lines 524-528

```cpp
    // Mainloop
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count) {
      // LOCK smem_pipe_write for _writing_
      pipeline.producer_acquire(smem_pipe_write);
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 530-532

```cpp
      //
      // Copy gmem to smem for *k_tile_iter
      //
```
**EN:** This comment-only block labels or explains the following section:  Copy gmem to smem for *k_tile_iter .
**CN:** 这个纯注释块用于标记或解释后续区域： Copy gmem to smem for *k_tile_iter 。

### Lines 534-535

```cpp
      using BarrierType = typename MainloopPipelineMK::ProducerBarrierType;
      BarrierType* tma_barrier = pipeline.producer_get_barrier(smem_pipe_write);
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 537-537

```cpp
      int write_stage = smem_pipe_write.index();
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 539-544

```cpp
      if (cute::elect_one_sync()) {
        copy(mainloop_params.tma_load_a.with(*tma_barrier), tAgA(_,_,_,*k_tile_iter), tAsA(_,_,_,write_stage));
        if constexpr (UseSmemE) {
          copy(mainloop_params.tma_load_e.with(*tma_barrier), tEgE(_,_,_,*k_tile_iter), tEsE(_,_,_,write_stage));
        }
      }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 546-549

```cpp
      if constexpr (!UseSmemE) {
        auto blk_coord_mkl = make_coord(get<0>(blk_coord), *k_tile_iter, get<3>(blk_coord));         // (BLK_M,BLK_K,L)
        prefetch(make_local_E(mainloop_params, blk_coord_mkl));
      }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 551-555

```cpp
      // Advance smem_pipe_write
      ++k_tile_iter;
      ++smem_pipe_write;
    }
  }
```
**EN:** The surrounding comments explain the local purpose of this block: Advance smem_pipe_write.
**CN:** 周围注释解释了这一段的局部作用：Advance smem_pipe_write。

### Lines 557-572

```cpp
  /// Issues loads for B only (used when DMA warp is split).
  template <
    class TensorA, class TensorB, class TensorE,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  load_NK(
      Params const& mainloop_params,
      MainloopPipelineNK pipeline,
      PipelineStateNK smem_pipe_write,
      cute::tuple<TensorA, TensorB, TensorE> const& load_inputs,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
```
**EN:** This block introduces `TensorA` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TensorA`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 574-574

```cpp
    Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.begin()), SmemLayoutB{});     //     (BLK_N,BLK_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 576-578

```cpp
    // Prepare the TMA loads for A and B
    Tensor gB_nkl = get<1>(load_inputs);
    auto block_tma_b = mainloop_params.tma_load_b.get_slice(0);
```
**EN:** The surrounding comments explain the local purpose of this block: Prepare the TMA loads for A and B.
**CN:** 周围注释解释了这一段的局部作用：Prepare the TMA loads for A and B。

### Lines 580-582

```cpp
    // Partition the inputs based on the current block coordinates.
    auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
    Tensor gB = gB_nkl(_,_,n_coord,_,l_coord);                                                //     (BLK_N,BLK_K,   k)
```
**EN:** The surrounding comments explain the local purpose of this block: Partition the inputs based on the current block coordinates..
**CN:** 周围注释解释了这一段的局部作用：Partition the inputs based on the current block coordinates.。

### Lines 584-586

```cpp
    // Applies the mapping from block_tma_a
    Tensor tBgB = block_tma_b.partition_S(gB);                                                // (TMA,TMA_N,TMA_K,   k)
    Tensor tBsB = block_tma_b.partition_D(sB);                                                // (TMA,TMA_N,TMA_K,PIPE)
```
**EN:** The surrounding comments explain the local purpose of this block: Applies the mapping from block_tma_a.
**CN:** 周围注释解释了这一段的局部作用：Applies the mapping from block_tma_a。

### Lines 588-592

```cpp
    // Mainloop
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count) {
      // LOCK smem_pipe_write for _writing_
      pipeline.producer_acquire(smem_pipe_write);
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 594-596

```cpp
      //
      // Copy gmem to smem for *k_tile_iter
      //
```
**EN:** This comment-only block labels or explains the following section:  Copy gmem to smem for *k_tile_iter .
**CN:** 这个纯注释块用于标记或解释后续区域： Copy gmem to smem for *k_tile_iter 。

### Lines 598-599

```cpp
      using BarrierType = typename MainloopPipelineNK::ProducerBarrierType;
      BarrierType* tma_barrier = pipeline.producer_get_barrier(smem_pipe_write);
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 601-604

```cpp
      int write_stage = smem_pipe_write.index();
      if (cute::elect_one_sync()) {
        copy(mainloop_params.tma_load_b.with(*tma_barrier), tBgB(_,_,_,*k_tile_iter), tBsB(_,_,_,write_stage));
      }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 606-610

```cpp
      // Advance smem_pipe_write
      ++k_tile_iter;
      ++smem_pipe_write;
    }
  }
```
**EN:** The surrounding comments explain the local purpose of this block: Advance smem_pipe_write.
**CN:** 周围注释解释了这一段的局部作用：Advance smem_pipe_write。

### Lines 612-616

```cpp
  /// Perform a Producer Epilogue to prevent early exit of blocks in a Cluster
  template<class MainloopPipeline, class PipelineState>
  CUTLASS_DEVICE void
  load_tail(MainloopPipeline pipeline, PipelineState smem_pipe_write) {
    int lane_predicate = cute::elect_one_sync();
```
**EN:** This block introduces `MainloopPipeline` and groups related declarations around that symbol. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这一段引入了 `MainloopPipeline`，并围绕该符号组织相关声明。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 618-628

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

### Lines 630-638

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

### Lines 640-644

```cpp
    // Global gmem E
    Tensor gE = make_tensor(make_gmem_ptr(ptr_E), layoutE);                                      // (BLK_M,BLK_K,BLK_L)
    // Local tile E
    return local_tile(gE, select<0,2>(TileShape{}), blk_coord);                                        // (BLK_M,BLK_K)
  }
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 646-656

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

### Lines 658-677

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

### Lines 679-690

```cpp
    if constexpr (IsF8F6F4) {
      auto get_copy_atom_and_common_vec = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
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

### Lines 692-712

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

### Lines 714-735

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

### Lines 737-737

```cpp
    static_assert(is_rmem<FrgTensorC>::value, "C tensor must be rmem resident.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 739-739

```cpp
    clear(accum);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 741-743

```cpp
    Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.begin()), SmemLayoutA{});         // (BLK_M,BLK_K,PIPE)
    Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.begin()), SmemLayoutB{});         // (BLK_N,BLK_K,PIPE)
    Tensor sE = make_tensor(make_smem_ptr(shared_tensors.smem_E.begin()), SmemLayoutE{});         // (BLK_M,BLK_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 745-747

```cpp
    //
    // Define A/B/E partitioning
    //
```
**EN:** This comment-only block labels or explains the following section:  Define A/B/E partitioning .
**CN:** 这个纯注释块用于标记或解释后续区域： Define A/B/E partitioning 。

### Lines 749-750

```cpp
    TiledMma tiled_mma;
    auto thread_mma = tiled_mma.get_thread_slice(thread_idx);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 752-755

```cpp
    // Allocate fragments and descriptors
    Tensor tCrA = thread_mma.partition_fragment_A(sA(_,_,Int<0>{}));                               // (MMA,MMA_M,MMA_K)
    Tensor tCrB = thread_mma.partition_fragment_B(sB(_,_,Int<0>{}));                               // (MMA,MMA_N,MMA_K)
    Tensor tCrE = partition_fragment_E(sE(_,_,Int<0>{}), thread_mma);                              // (MMA,MMA_M,MMA_K)
```
**EN:** The surrounding comments explain the local purpose of this block: Allocate fragments and descriptors.
**CN:** 周围注释解释了这一段的局部作用：Allocate fragments and descriptors。

### Lines 757-764

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

### Lines 766-770

```cpp
    auto smem_tiled_copy_B = make_tiled_copy_B(SmemCopyAtomB{}, tiled_mma);
    auto smem_thr_copy_B   = smem_tiled_copy_B.get_thread_slice(thread_idx);
    Tensor tCsB            = smem_thr_copy_B.partition_S(
         as_position_independent_swizzle_tensor(sB));                                         // (CPY,CPY_N,CPY_K,PIPE)
    Tensor tCrB_copy_view  = smem_thr_copy_B.retile_D(tCrB);                                  //      (CPY,CPY_N,CPY_K)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 772-779

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

### Lines 781-793

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
    if constexpr (UseSmemE) {
      CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::StagesA>{} == size<2>(sE));
    }
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 795-797

```cpp
    //
    // DEFINE FUNCTIONS FOR PIPELINED MAIN LOOP
    //
```
**EN:** This comment-only block labels or explains the following section:  DEFINE FUNCTIONS FOR PIPELINED MAIN LOOP .
**CN:** 这个纯注释块用于标记或解释后续区域： DEFINE FUNCTIONS FOR PIPELINED MAIN LOOP 。

### Lines 799-801

```cpp
    // We release buffers to producer warps(dma load) with some mmas in flight
    PipelineStateMK smem_pipe_release_mk = smem_pipe_read_mk;
    PipelineStateNK smem_pipe_release_nk = smem_pipe_read_nk;
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 803-807

```cpp
    // Wait consumer barrier MK
    auto wait_barrier_mk = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      auto barrier_token_mk = pipeline_mk.consumer_try_wait(smem_pipe_read_mk);
      pipeline_mk.consumer_wait(smem_pipe_read_mk, barrier_token_mk);
    };
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 809-813

```cpp
    // Wait consumer barrier NK
    auto wait_barrier_nk = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      auto barrier_token_nk = pipeline_nk.consumer_try_wait(smem_pipe_read_nk);
      pipeline_nk.consumer_wait(smem_pipe_read_nk, barrier_token_nk);
    };
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 815-820

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

### Lines 822-827

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

### Lines 829-836

```cpp
    // Copy A from SMEM to register, and do transform if needed
    auto copy_transform_A = [&](auto m_block, auto k_block) CUTLASS_LAMBDA_FUNC_INLINE {
      // copy smem->rmem for A operand
      copy(smem_tiled_copy_A, tCsA(_,m_block,k_block,smem_pipe_read_mk.index()), tCrA_copy_view(_,m_block,k_block));
      // Perform transform if needed.
      using MMAOp = typename TiledMma::MMA_Op;
      fp4_shift_A(MMAOp{}, tCrA(_,m_block,k_block));
    };
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 838-845

```cpp
    // Copy B from SMEM to register, and do transform if needed
    auto copy_transform_B = [&](auto n_block, auto k_block) CUTLASS_LAMBDA_FUNC_INLINE {
      // copy smem->rmem for B operand
      copy(smem_tiled_copy_B, tCsB(_,n_block,k_block,smem_pipe_read_nk.index()), tCrB_copy_view(_,n_block,k_block));
      // Perform transform if needed.
      using MMAOp = typename TiledMma::MMA_Op;
      fp4_shift_B(MMAOp{}, tCrB(_,n_block,k_block));
    };
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 847-852

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

### Lines 854-858

```cpp
    // TILE M/N/K for one TILE block
    constexpr auto M_BLOCK_MAX = size<1>(tCrA);
    constexpr auto N_BLOCK_MAX = size<1>(tCrB);
    constexpr auto K_BLOCK_MAX = size<2>(tCrA);
    constexpr auto K_BLOCK_STEP = K_BLOCK_MAX / Int<AsymmetricKRatio>{};
```
**EN:** The surrounding comments explain the local purpose of this block: TILE M/N/K for one TILE block.
**CN:** 周围注释解释了这一段的局部作用：TILE M/N/K for one TILE block。

### Lines 860-864

```cpp
    // Perform mainloop gemm, when E is in SMEM.
    auto gemm_loop_with_SmemE = [&]() CUTLASS_LAMBDA_FUNC_INLINE {
      // WAIT on smem_pipe_read until data is available
      wait_barrier_mk();
      wait_barrier_nk();
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 866-870

```cpp
      // Load A/B/E, then do gemm.
      for_each(make_int_sequence<K_BLOCK_MAX>{}, [&] (auto k_block) {
        for_each(make_int_sequence<N_BLOCK_MAX>{}, [&] (auto n_block) {
          // Copy smem->rmem for B operand
          copy_transform_B(n_block, k_block);
```
**EN:** The surrounding comments explain the local purpose of this block: Load A/B/E, then do gemm. Copy smem->rmem for B operand.
**CN:** 周围注释解释了这一段的局部作用：Load A/B/E, then do gemm. Copy smem->rmem for B operand。

### Lines 872-875

```cpp
          for_each(make_int_sequence<M_BLOCK_MAX>{}, [&] (auto m_block) {
            // Copy smem->rmem for A operand
            copy_transform_A(m_block, k_block);
            copy_E(m_block, k_block);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for A operand.
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for A operand。

### Lines 877-884

```cpp
            // Gemm
            cute::gemm(tiled_mma,
                      make_zip_tensor(tCrA(_,m_block,k_block), tCrE(_,m_block,k_block)),
                      tCrB(_,n_block,k_block),
                      accum(_,m_block,n_block));
          });
        });
      });
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。

### Lines 886-887

```cpp
      cutlass::arch::NamedBarrier::sync(
        thr_size(tiled_mma), cutlass::arch::ReservedNamedBarriers::Sm120MainloopBarrier);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 889-892

```cpp
      // Advance consumer pipeline mk/nk
      release_advance_mk();
      release_advance_nk();
    };
```
**EN:** The surrounding comments explain the local purpose of this block: Advance consumer pipeline mk/nk.
**CN:** 周围注释解释了这一段的局部作用：Advance consumer pipeline mk/nk。

### Lines 894-899

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

### Lines 901-903

```cpp
      // WAIT on smem_pipe_read until data is available
      wait_barrier_mk();
      wait_barrier_nk();
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 905-908

```cpp
      for_each(make_int_sequence<K_BLOCK_STEP>{}, [&] (auto k_block) {
        for_each(make_int_sequence<N_BLOCK_MAX>{}, [&] (auto n_block) {
          // Copy smem->rmem for B operand
          copy_transform_B(n_block, k_block);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for B operand.
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for B operand。

### Lines 910-912

```cpp
          for_each(make_int_sequence<M_BLOCK_MAX>{}, [&] (auto m_block) {
            // Copy smem->rmem for A operand
            copy_transform_A(m_block, k_block);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for A operand.
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for A operand。

### Lines 914-921

```cpp
            // Gemm
            cute::gemm(tiled_mma,
                      make_zip_tensor(tCrA(_,m_block,k_block), tCrE(_,m_block,k_block)),
                      tCrB(_,n_block,k_block),
                      accum(_,m_block,n_block));
          });
        });
      });
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。

### Lines 923-924

```cpp
      cutlass::arch::NamedBarrier::sync(
        thr_size(tiled_mma), cutlass::arch::ReservedNamedBarriers::Sm120MainloopBarrier);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 926-929

```cpp
      // Advance consumer pipeline_nk
      release_advance_nk();
      // Wait next buffer
      wait_barrier_nk();
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 931-935

```cpp
      for_each(make_int_sequence<K_BLOCK_STEP>{}, [&] (auto k_block) {
        auto k_block_a = k_block + K_BLOCK_STEP;
        for_each(make_int_sequence<N_BLOCK_MAX>{}, [&] (auto n_block) {
          // Copy smem->rmem for B operand
          copy_transform_B(n_block, k_block);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for B operand.
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for B operand。

### Lines 937-939

```cpp
          for_each(make_int_sequence<M_BLOCK_MAX>{}, [&] (auto m_block) {
            // Copy smem->rmem for A operand
            copy_transform_A(m_block, k_block_a);
```
**EN:** The surrounding comments explain the local purpose of this block: Copy smem->rmem for A operand.
**CN:** 周围注释解释了这一段的局部作用：Copy smem->rmem for A operand。

### Lines 941-948

```cpp
            // Gemm
            cute::gemm(tiled_mma,
                      make_zip_tensor(tCrA(_,m_block,k_block_a), tCrE(_,m_block,k_block_a)),
                      tCrB(_,n_block,k_block),
                      accum(_,m_block,n_block));
          });
        });
      });
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。

### Lines 950-951

```cpp
      cutlass::arch::NamedBarrier::sync(
        thr_size(tiled_mma), cutlass::arch::ReservedNamedBarriers::Sm120MainloopBarrier);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 953-956

```cpp
      // Advance consumer pipeline mk/nk
      release_advance_mk();
      release_advance_nk();
    };
```
**EN:** The surrounding comments explain the local purpose of this block: Advance consumer pipeline mk/nk.
**CN:** 周围注释解释了这一段的局部作用：Advance consumer pipeline mk/nk。

### Lines 959-961

```cpp
    //
    // PIPELINED MAIN LOOP
    //
```
**EN:** This comment-only block labels or explains the following section:  PIPELINED MAIN LOOP .
**CN:** 这个纯注释块用于标记或解释后续区域： PIPELINED MAIN LOOP 。

### Lines 963-964

```cpp
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 0; --k_tile_count) {
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 966-973

```cpp
      // Case when A/B with same stages, and keep E in SMEM.
      if constexpr (UseSmemE) {
        gemm_loop_with_SmemE();
      }
      // Case when A/B with different stages, and keep E in GMEM.
      else {
        gemm_loop_with_GmemE();
      } // end if
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 975-976

```cpp
    } // end loop k_tile_count
  }
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 978-982

```cpp
  /// Perform a Consumer Epilogue to release all buffers
  CUTLASS_DEVICE void
  mma_tail(MainloopPipelineMK, PipelineStateMK, MainloopPipelineNK, PipelineStateNK, int) {
  }
};
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 984-984

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 986-986

```cpp
} // namespace cutlass::gemm::collective
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 988-988

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
  - `ElementA_`
  - `LayoutPairAE_`
  - `ElementB_`
  - `StrideB_`
  - `TiledMma_`
  - `GmemTiledCopyA_`
  - `SmemLayoutAtomA_`
  - `SmemCopyAtomPairA_`
  - `TransformA_`
  - `GmemTiledCopyB_`
  - `SmemLayoutAtomB_`
  - `SmemCopyAtomB_`
  - `TransformB_`
  - `CollectiveMma`
- **Shared abstractions / 共享抽象:** `CollectiveMma` / `CollectiveBuilder`, dispatch-policy tags, CuTe tensor/layout utilities, and CUTLASS pipeline helpers. / `CollectiveMma` / `CollectiveBuilder`、dispatch-policy 标签、CuTe 张量/布局工具以及 CUTLASS 流水线辅助组件。