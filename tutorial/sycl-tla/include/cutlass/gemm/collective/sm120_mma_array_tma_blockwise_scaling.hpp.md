# sm120_mma_array_tma_blockwise_scaling.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/gemm/collective/sm120_mma_array_tma_blockwise_scaling.hpp`
- **Purpose (EN):** Implements an architecture-specific `CollectiveMma` specialization for SM120, covering blockwise scaling rules, TMA transfers, array-based MMA tiling.
- **用途 (CN):** 为 SM120 实现架构特化的 `CollectiveMma` 特化版本，重点覆盖 块级缩放规则、TMA 传输、数组式 MMA 分块。
- **Lines / 行数:** 1001

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

### Lines 34-40

```cpp
#include "cutlass/cutlass.h"
#include "cutlass/gemm/gemm.h"
#include "cutlass/pipeline/pipeline.hpp"
#include "cutlass/gemm/dispatch_policy.hpp"
#include "cutlass/detail/dependent_false.hpp"
#include "cutlass/trace.h"
#include "cutlass/numeric_types.h"
```
**EN:** This include block imports cutlass.h, gemm.h, pipeline.hpp, dispatch_policy.hpp, and 3 more. These dependencies provide the collective templates, architecture helpers, or tensor utilities referenced later.
**CN:** 这一组 `#include` 引入了 cutlass.h、gemm.h、pipeline.hpp、dispatch_policy.hpp 等 7 项。这些依赖为后续代码提供 collective 模板、架构辅助组件或张量工具。

### Lines 42-47

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

### Lines 49-49

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 51-52

```cpp
namespace cutlass::gemm::collective {
using namespace cute;
```
**EN:** This short block both opens the collective namespace and pulls CuTe symbols into scope, setting up a concise vocabulary for the rest of the file.
**CN:** 这个短代码块既打开了 collective 命名空间，也把 CuTe 符号引入作用域，为后续代码建立更简洁的表达方式。

### Lines 54-54

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 56-100

```cpp
template <
  int Stages,
  int SchedulerPipelineStageCount,
  class ClusterShape,
  class KernelScheduleType,
  class TileShape_,
  class ElementA_,
  class StridePairA_,
  class ElementB_,
  class StridePairB_,
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
    MainloopSm120ArrayTmaWarpSpecializedBlockwiseScaling<Stages, SchedulerPipelineStageCount, ClusterShape, KernelScheduleType>,
    TileShape_,
    ElementA_,
    StridePairA_,
    ElementB_,
    StridePairB_,
    TiledMma_,
    GmemTiledCopyA_,
    SmemLayoutAtomA_,
    SmemCopyAtomA_,
    TransformA_,
    GmemTiledCopyB_,
    SmemLayoutAtomB_,
    SmemCopyAtomB_,
    TransformB_> {
  //
  // Type Aliases
  //
  using DispatchPolicy = MainloopSm120ArrayTmaWarpSpecializedBlockwiseScaling<Stages, SchedulerPipelineStageCount, ClusterShape, KernelScheduleType>;
  using TileShape = TileShape_;
  using ElementA = remove_cvref_t<ElementA_>;
  using StrideA = cute::remove_cvref_t<decltype(get<0>(StridePairA_{}))>;
  using InternalStrideA = cute::remove_pointer_t<StrideA>;
  using LayoutSFA = cute::remove_cvref_t<decltype(get<1>(StridePairA_{}))>;
  using InternalLayoutSFA = cute::remove_pointer_t<LayoutSFA>;
```
**EN:** This template block declares or specializes `CollectiveMma`, the mainloop object that coordinates tile movement and matrix-multiply work.
**CN:** 这个模板块声明或特化了 `CollectiveMma`，它负责协调 tile 搬运与矩阵乘主循环。

### Lines 102-106

```cpp
  using ElementB = remove_cvref_t<ElementB_>;
  using StrideB = cute::remove_cvref_t<decltype(get<0>(StridePairB_{}))>;
  using InternalStrideB = cute::remove_pointer_t<StrideB>;
  using LayoutSFB = cute::remove_cvref_t<decltype(get<1>(StridePairB_{}))>;
  using InternalLayoutSFB = cute::remove_pointer_t<LayoutSFB>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 108-120

```cpp
  using TiledMma = TiledMma_;
  using CtaShape_MNK = decltype(shape_div(TileShape{}, ClusterShape{}));
  using ElementAccumulator = typename TiledMma::ValTypeC;
  using ElementSF = ElementAccumulator;
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

### Lines 122-123

```cpp
  using RuntimeDataTypeA = void*;
  using RuntimeDataTypeB = void*;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 125-125

```cpp
  static constexpr int ThreadCount = size(TiledMma{});
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 127-127

```cpp
  using MainloopPipeline = cutlass::PipelineTmaAsync<DispatchPolicy::Stages>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 129-130

```cpp
  using PipelineParams = typename MainloopPipeline::Params;
  using PipelineState  = typename cutlass::PipelineState<DispatchPolicy::Stages>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 132-133

```cpp
  // One threads per CTA are producers (1 for operand tile)
  static constexpr int NumProducerThreadEvents = 33;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 135-137

```cpp
  static constexpr int ScaleGranularityM = size<0,0>(InternalLayoutSFA{});
  static constexpr int ScaleGranularityN = size<0,0>(InternalLayoutSFB{});
  static constexpr int ScaleGranularityK = size<1,0>(InternalLayoutSFB{});
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 139-144

```cpp
  static_assert(size<1, 0>(InternalLayoutSFA{}) == size<1, 0>(InternalLayoutSFB{}), "Vector size K must be equal for SFA and SFB");
  static_assert(size<0>(TileShape{}) % ScaleGranularityM == 0, "Scale Granularity M must evenly divide the tile shape M.");
  static_assert(size<1>(TileShape{}) % ScaleGranularityN == 0, "Scale Granularity N must evenly divide the tile shape N.");
  static_assert(size<2>(TileShape{}) == ScaleGranularityK    , "Scale Granularity K must be equal to the tile shape K.");
  static constexpr int ScaleMsPerTile = size<0>(TileShape{}) / ScaleGranularityM;
  static constexpr int ScaleNsPerTile = size<1>(TileShape{}) / ScaleGranularityN;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes. The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 146-150

```cpp
  using ScaleConfig = cutlass::detail::Sm120BlockwiseScaleConfig<ScaleGranularityM,
      ScaleGranularityN,
      ScaleGranularityK,
      size<0,1>(InternalLayoutSFA{}.stride()) == 1 ? UMMA::Major::MN : UMMA::Major::K,
      size<0,1>(InternalLayoutSFB{}.stride()) == 1 ? UMMA::Major::MN : UMMA::Major::K>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 152-153

```cpp
  static constexpr int AlignmentSFA = 1;
  static constexpr int AlignmentSFB = 1;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 155-157

```cpp
  static_assert(rank(SmemLayoutAtomA{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<0>(TileShape{}) % size<0>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomA{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 159-161

```cpp
  static_assert(rank(SmemLayoutAtomB{}) == 2, "SmemLayoutAtom must be rank 2 (M/N, K)");
  static_assert((size<1>(TileShape{}) % size<0>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
  static_assert((size<2>(TileShape{}) % size<1>(SmemLayoutAtomB{})) == 0, "SmemLayoutAtom must evenly divide tile shape.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 163-166

```cpp
  static_assert(not cute::is_void_v<SmemCopyAtomA>,
    "SM120 mainloop must specify a copy atom for A operand smem->rmem reads.");
  static_assert(not cute::is_void_v<SmemCopyAtomB>,
    "SM120 mainloop must specify a copy atom for B operand smem->rmem reads.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 168-176

```cpp
  // Tile along modes in a way that maximizes the TMA box size.
  using SmemLayoutA = decltype(tile_to_shape(
      SmemLayoutAtomA{},
      make_shape(shape<0>(TileShape{}), shape<2>(TileShape{}), Int<DispatchPolicy::Stages>{}),
      conditional_t< ::cutlass::gemm::detail::is_major<0,InternalStrideA>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
  using SmemLayoutB = decltype(tile_to_shape(
      SmemLayoutAtomB{},
      make_shape(shape<1>(TileShape{}), shape<2>(TileShape{}), Int<DispatchPolicy::Stages>{}),
      conditional_t< ::cutlass::gemm::detail::is_major<0,InternalStrideB>(), Step<_2,_1,_3>, Step<_1,_2,_3>>{}));
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 178-181

```cpp
  // Block scaling gmem-to-smem copy atom
  //  we can have partial tiles in M or N, so don't vectorize those loads
  using SmemBlockScalingCopyAtomA = Copy_Atom<SM80_CP_ASYNC_CACHEALWAYS<ElementSF>, ElementSF>;
  using SmemBlockScalingCopyAtomB = Copy_Atom<SM80_CP_ASYNC_CACHEALWAYS<ElementSF>, ElementSF>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 183-185

```cpp
  // Block scaling smem layout
  using SmemLayoutScaleA = Layout<Shape<Int<ScaleMsPerTile>, Int<DispatchPolicy::Stages>>>;
  using SmemLayoutScaleB = Layout<Shape<Int<ScaleNsPerTile>, Int<DispatchPolicy::Stages>>>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 188-189

```cpp
  static_assert(rank(SmemLayoutA{}) == 3, "Smem layout must be rank 3.");
  static_assert(rank(SmemLayoutB{}) == 3, "Smem layout must be rank 3.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 191-198

```cpp
  static_assert(DispatchPolicy::Stages >= 2, "Specialization requires Stages set to value 2 or more.");
  static_assert(not cute::is_base_of<cute::GMMA::DescriptorIterator, typename TiledMma::FrgTypeA>::value &&
                not cute::is_base_of<cute::GMMA::DescriptorIterator, typename TiledMma::FrgTypeB>::value,
                "MMA atom must source both A and B operands from rmem for this mainloop.");
  static_assert(cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD> || cute::is_same_v<GmemTiledCopyA, SM90_TMA_LOAD_MULTICAST>,
      "GmemTiledCopy - invalid SM90 TMA copy atom specified.");
  static_assert(cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD> || cute::is_same_v<GmemTiledCopyB, SM90_TMA_LOAD_MULTICAST>,
      "GmemTiledCopy - invalid SM90 TMA copy atom specified.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 200-200

```cpp
  static constexpr bool IsF8F6F4 = detail::is_sm120_f8f6f4<TiledMma, ElementA, ElementB>();
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 202-221

```cpp
  // TMA converts f32 input to tf32 when copying from GMEM to SMEM
  // For all other types, cast to size equivalent uint type to avoid any rounding by TMA.
  using TmaInternalElementA = cute::conditional_t<cute::is_same_v<ElementA, float>,
                                                  cutlass::tfloat32_t,
                              cute::conditional_t<cute::is_same_v<ElementA, cutlass::float_e2m1_t>,
                                                  cutlass::detail::float_e2m1_unpacksmem_t,
                              cute::conditional_t<cute::is_same_v<ElementA, cutlass::float_e2m3_t>,
                                                cutlass::detail::float_e2m3_unpacksmem_t,
                              cute::conditional_t<cute::is_same_v<ElementA, cutlass::float_e3m2_t>,
                                                cutlass::detail::float_e3m2_unpacksmem_t,
                                                uint_bit_t<sizeof_bits_v<ElementA>>>>>>;
  using TmaInternalElementB = cute::conditional_t<cute::is_same_v<ElementB, float>,
                                                  cutlass::tfloat32_t,
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

### Lines 223-224

```cpp
  using SmemAllocTypeA = cute::conditional_t<IsF8F6F4, uint8_t, typename TiledMma::ValTypeA>;
  using SmemAllocTypeB = cute::conditional_t<IsF8F6F4, uint8_t, typename TiledMma::ValTypeB>;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 226-231

```cpp
  // Set the bytes transferred in this TMA transaction (may involve multiple issues)
  static constexpr uint32_t TmaTransactionBytesMK = static_cast<uint32_t>(
      cutlass::bits_to_bytes(size(take<0,2>(SmemLayoutA{})) * sizeof_bits<ElementA>::value));
  static constexpr uint32_t TmaTransactionBytesNK = static_cast<uint32_t>(
      cutlass::bits_to_bytes(size(take<0,2>(SmemLayoutB{})) * sizeof_bits<ElementB>::value));
  static constexpr uint32_t TmaTransactionBytes = TmaTransactionBytesMK + TmaTransactionBytesNK;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 233-239

```cpp
  struct SharedStorage {
    struct TensorStorage : cute::aligned_struct<128, _0> {
      alignas(1024) cute::array_aligned<SmemAllocTypeA, cute::cosize_v<SmemLayoutA>> smem_A;
      alignas(1024) cute::array_aligned<SmemAllocTypeB, cute::cosize_v<SmemLayoutB>> smem_B;
      cute::array_aligned<ElementSF, cute::cosize_v<SmemLayoutScaleA>> smem_scale_A;
      cute::array_aligned<ElementSF, cute::cosize_v<SmemLayoutScaleB>> smem_scale_B;
    } tensors;
```
**EN:** This block introduces `SharedStorage` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `SharedStorage`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 241-244

```cpp
    struct TensorMapStorage : cute::aligned_struct<128, _0> {
      cute::TmaDescriptor smem_tensormap_A;
      cute::TmaDescriptor smem_tensormap_B;
    } tensormaps;
```
**EN:** This block introduces `TensorMapStorage` and groups related declarations around that symbol.
**CN:** 这一段引入了 `TensorMapStorage`，并围绕该符号组织相关声明。

### Lines 246-251

```cpp
    using PipelineStorage = typename MainloopPipeline::SharedStorage;
    alignas(16) PipelineStorage pipeline_storage;
  };
  using TensorStorage = typename SharedStorage::TensorStorage;
  using PipelineStorage = typename SharedStorage::PipelineStorage;
  using TensorMapStorage = typename SharedStorage::TensorMapStorage;
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 253-253

```cpp
  static constexpr bool IsGroupedGemmKernel = !cute::is_same_v<InternalStrideA, StrideA>;
```
**EN:** The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 255-265

```cpp
  // Host side kernel arguments
  struct Arguments {
    ElementA const** ptr_A{nullptr};
    StrideA dA{};
    ElementB const** ptr_B{nullptr};
    StrideB dB{};
    ElementAccumulator const** ptr_SFA{nullptr};
    LayoutSFA layout_SFA{};
    ElementAccumulator const** ptr_SFB{nullptr};
    LayoutSFB layout_SFB{};
  };
```
**EN:** This block introduces `Arguments` and groups related declarations around that symbol. `Arguments` is the host-side bundle: callers fill it with pointers, strides, and other launch-time values before parameter lowering.
**CN:** 这一段引入了 `Arguments`，并围绕该符号组织相关声明。 `Arguments` 是主机侧参数包：调用者在参数下沉前把指针、步长及其他启动期数据填入其中。

### Lines 267-298

```cpp
  // Device side kernel params
  struct Params {
    // Assumption: StrideA is congruent with Problem_MK
    using TMA_A = decltype(make_tma_copy(
        GmemTiledCopyA{},
        make_tensor(recast_ptr<TmaInternalElementA>(nullptr), repeat_like(InternalStrideA{}, int32_t(0)), InternalStrideA{}),
        SmemLayoutA{}(_,_,0),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        size<1>(ClusterShape{})));  // mcast along N mode for this M load, if any
    // Assumption: StrideB is congruent with Problem_NK
    using TMA_B = decltype(make_tma_copy(
        GmemTiledCopyB{},
        make_tensor(recast_ptr<TmaInternalElementB>(nullptr), repeat_like(InternalStrideB{}, int32_t(0)), InternalStrideB{}),
        SmemLayoutB{}(_,_,0),
        make_shape(shape<1>(TileShape{}), shape<2>(TileShape{})),
        size<0>(ClusterShape{}))); // mcast along M mode for this N load, if any
    TMA_A tma_load_a;
    TMA_B tma_load_b;
    uint32_t tma_transaction_bytes = TmaTransactionBytes;
    uint32_t tma_transaction_bytes_mk = TmaTransactionBytesMK;
    uint32_t tma_transaction_bytes_nk = TmaTransactionBytesNK;
    // Block scaling factors for A and B
    cute::TmaDescriptor* tensormaps;
    ElementA const** ptr_A;
    StrideA dA;
    ElementB const** ptr_B;
    StrideB dB;
    ElementSF const** ptr_SFA;
    LayoutSFA layout_SFA;
    ElementSF const** ptr_SFB;
    LayoutSFB layout_SFB;
  };
```
**EN:** This block introduces `Params` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `Params`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 300-302

```cpp
  //
  // Methods
  //
```
**EN:** This comment starts a method section where the collective exposes its runtime behavior.
**CN:** 这条注释表示进入方法区，collective 的运行时行为将在这里定义。

### Lines 304-307

```cpp
  template <class ProblemShape>
  static constexpr Params
  to_underlying_arguments(ProblemShape const& problem_shapes, Arguments const& args, void* workspace) {
    (void) workspace;
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The `static constexpr` values derive compile-time sizes, stage counts, or alignment-related constants from the dispatch policy and tile shapes.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这些 `static constexpr` 常量根据调度策略和 tile 形状推导出编译期尺寸、阶段数或对齐相关常量。

### Lines 309-318

```cpp
    auto init_shape = repeat_like(typename ProblemShape::UnderlyingProblemShape{}, int32_t(1));
    constexpr int tma_alignment_bits = 128;
    auto init_M = tma_alignment_bits;
    auto init_N = tma_alignment_bits;
    auto init_K = tma_alignment_bits;
    const uint32_t init_L = 1;
    TmaInternalElementA const* ptr_A_first_batch = nullptr;
    TmaInternalElementB const* ptr_B_first_batch = nullptr;
    InternalStrideA stride_a;
    InternalStrideB stride_b;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 320-328

```cpp
    if constexpr (IsGroupedGemmKernel) {
      stride_a = InternalStrideA{};
      stride_b = InternalStrideB{};
    }
    else {
      auto problem_shape_MNK = problem_shapes.get_host_problem_shape(0);
      init_M = get<0>(problem_shape_MNK);
      init_N = get<1>(problem_shape_MNK);
      init_K = get<2>(problem_shape_MNK);
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 330-332

```cpp
      stride_a = args.dA;
      stride_b = args.dB;
    }
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 334-335

```cpp
    Tensor tensor_a = make_tensor(ptr_A_first_batch, make_layout(make_shape(init_M, init_K, init_L), stride_a));
    Tensor tensor_b = make_tensor(ptr_B_first_batch, make_layout(make_shape(init_N, init_K, init_L), stride_b));
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 337-348

```cpp
    typename Params::TMA_A tma_load_a = make_tma_copy(
        GmemTiledCopyA{},
        tensor_a,
        SmemLayoutA{}(_,_,cute::Int<0>{}),
        make_shape(shape<0>(TileShape{}), shape<2>(TileShape{})),
        size<1>(ClusterShape{})); // mcast along N mode for this M load, if any
    typename Params::TMA_B tma_load_b = make_tma_copy(
        GmemTiledCopyB{},
        tensor_b,
        SmemLayoutB{}(_,_,cute::Int<0>{}),
        make_shape(shape<1>(TileShape{}), shape<2>(TileShape{})),
        size<0>(ClusterShape{})); // mcast along M mode for this N load, if any
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 350-366

```cpp
    return {
      tma_load_a,
      tma_load_b,
      TmaTransactionBytes,
      TmaTransactionBytesMK,
      TmaTransactionBytesNK,
      reinterpret_cast<cute::TmaDescriptor*>(workspace),
      reinterpret_cast<ElementA const**>(args.ptr_A),
      args.dA,
      reinterpret_cast<ElementB const**>(args.ptr_B),
      args.dB,
      reinterpret_cast<ElementSF const**>(args.ptr_SFA),
      args.layout_SFA,
      reinterpret_cast<ElementSF const**>(args.ptr_SFB),
      args.layout_SFB
    };
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 368-374

```cpp
  template <class ProblemShape>
  static size_t
  get_workspace_size(ProblemShape const& problem_shape, Arguments const& args, int sm_count) {
    constexpr uint32_t NumInputTmaTensors = 2;
    constexpr size_t SizeOfCuTensorMap = sizeof(cute::TmaDescriptor);
    return (NumInputTmaTensors * SizeOfCuTensorMap * sm_count);
  }
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 376-380

```cpp
  template <class ProblemShape>
  static cutlass::Status
  initialize_workspace(ProblemShape const& problem_shape, Arguments const& args, void* workspace, cudaStream_t stream, CudaHostAdapter* cuda_adapter = nullptr) {
    return cutlass::Status::kSuccess;
  }
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 382-386

```cpp
  template<class ProblemShape>
  static bool
  can_implement(
      ProblemShape problem_shapes,
      [[maybe_unused]] Arguments const& args) {
```
**EN:** This block introduces `ProblemShape` and groups related declarations around that symbol. This method performs runtime feasibility checks, usually validating alignment, layout assumptions, or shape constraints before launching the kernel.
**CN:** 这一段引入了 `ProblemShape`，并围绕该符号组织相关声明。 这个方法执行运行时可实现性检查，通常会在启动内核前验证对齐、布局假设或形状约束。

### Lines 388-391

```cpp
    constexpr int tma_alignment_bits_A = cutlass::detail::get_input_alignment_bits<ElementA, IsF8F6F4>();
    constexpr int tma_alignment_bits_B = cutlass::detail::get_input_alignment_bits<ElementB, IsF8F6F4>();
    constexpr int min_tma_aligned_elements_A = tma_alignment_bits_A / cutlass::sizeof_bits<ElementA>::value;
    constexpr int min_tma_aligned_elements_B = tma_alignment_bits_B / cutlass::sizeof_bits<ElementB>::value;
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 393-399

```cpp
    bool implementable = true;
    if (problem_shapes.is_host_problem_shape_available()) {
      for (int i = 0; i < problem_shapes.groups(); ++i) {
        auto problem_shape_MNKL = append<4>(problem_shapes.get_host_problem_shape(i), 1);
        auto [M, N, K, L] = problem_shape_MNKL;
        implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_A>(cute::make_shape(M,K,L), InternalStrideA{});
        implementable = implementable && cutlass::detail::check_alignment<min_tma_aligned_elements_B>(cute::make_shape(N,K,L), InternalStrideB{});
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 401-403

```cpp
        if (!implementable) {
          CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem Size doesn't meet the minimum alignment requirements for TMA.\n");
        }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 405-407

```cpp
        // Ensure complete scale blocks
        implementable = implementable && (M % ScaleGranularityM == 0);
        implementable = implementable && (N % ScaleGranularityN == 0);
```
**EN:** The surrounding comments explain the local purpose of this block: Ensure complete scale blocks.
**CN:** 周围注释解释了这一段的局部作用：Ensure complete scale blocks。

### Lines 409-410

```cpp
        // We expect full tiles in K
        implementable = implementable && (K % size<2>(TileShape{}) == 0);
```
**EN:** The surrounding comments explain the local purpose of this block: We expect full tiles in K.
**CN:** 周围注释解释了这一段的局部作用：We expect full tiles in K。

### Lines 412-416

```cpp
        if (!implementable) {
          CUTLASS_TRACE_HOST("  CAN IMPLEMENT: Problem Size doesn't meet the minimum alignment requirements for blockwise scaling.\n");
        }
      }
    }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 418-419

```cpp
    return implementable;
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 422-441

```cpp
  /// Set up the data needed by this collective for load and mma.
  /// Returns a tuple of tensors. The collective and the kernel layer have the contract
  /// Returned tuple must contain at least two elements, with the first two elements being:
  /// gA_mkl - The tma tensor, A after a local tile so it has shape  (BLK_M,BLK_K,m,k,l)
  /// gB_nkl - The tma tensor, B after a local tile so it has shape  (BLK_N,BLK_K,n,k,l)
  /// The rest of the tensors can be specified as needed by this collective.
  template <class ProblemShape_MNKL>
  CUTLASS_DEVICE auto
  load_init(
    ProblemShape_MNKL const& problem_shape_MNKL,
    Params const& mainloop_params,
    ElementSF const* ptr_SFA = nullptr,
    ElementSF const* ptr_SFB = nullptr,
    InternalLayoutSFA const layout_SFA = InternalLayoutSFA{},
    InternalLayoutSFB const layout_SFB = InternalLayoutSFB{}
  ) const {
    using X = Underscore;
    // Separate out problem shape for convenience
    auto [M, N, K, L] = problem_shape_MNKL;
    const int32_t init_L = 1;
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol. It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 443-446

```cpp
    // TMA requires special handling of strides to deal with coord codomain mapping
    // Represent the full tensors -- get these from TMA
    Tensor mA_mkl = mainloop_params.tma_load_a.get_tma_tensor(make_shape(M,K,init_L));                            // (m,k,l)
    Tensor mB_nkl = mainloop_params.tma_load_b.get_tma_tensor(make_shape(N,K,init_L));                            // (n,k,l)
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 448-450

```cpp
    // Make tiled views, defer the slice
    Tensor gA_mkl = local_tile(mA_mkl, TileShape{}, make_coord(_,_,_), Step<_1, X,_1>{});        // (BLK_M,BLK_K,m,k,l)
    Tensor gB_nkl = local_tile(mB_nkl, TileShape{}, make_coord(_,_,_), Step< X,_1,_1>{});        // (BLK_N,BLK_K,n,k,l)
```
**EN:** The surrounding comments explain the local purpose of this block: Make tiled views, defer the slice.
**CN:** 周围注释解释了这一段的局部作用：Make tiled views, defer the slice。

### Lines 452-453

```cpp
    Tensor mSFA_mkl = make_tensor(make_gmem_ptr(ptr_SFA), filter(layout_SFA)); // (Ms, Ks)
    Tensor mSFB_nkl = make_tensor(make_gmem_ptr(ptr_SFB), filter(layout_SFB)); // (Ns, Ks)
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 455-456

```cpp
    return cute::make_tuple(gA_mkl, gB_nkl, mSFA_mkl, mSFB_nkl);
  }
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 458-478

```cpp
  /// Perform a collective-scoped matrix multiply-accumulate
  /// Producer Perspective
  template <
    class TensorA, class TensorB,
    class TensorSFA, class TensorSFB,
    class TensorMapA, class TensorMapB,
    class KTileIterator, class BlockCoord
  >
  CUTLASS_DEVICE void
  load(
      Params const& mainloop_params,
      MainloopPipeline pipeline,
      PipelineState smem_pipe_write,
      cute::tuple<TensorA, TensorB, TensorSFA, TensorSFB> const& load_inputs,
      cute::tuple<TensorMapA, TensorMapB> const& input_tensormaps,
      BlockCoord const& blk_coord,
      KTileIterator k_tile_iter, int k_tile_count,
      int thread_idx,
      uint32_t block_rank_in_cluster,
      TensorStorage& shared_tensors) {
    int lane_predicate = cute::elect_one_sync();
```
**EN:** This block introduces `TensorA` and groups related declarations around that symbol. This part defines on-chip storage used to stage operand tiles, pipeline metadata, or accumulator state.
**CN:** 这一段引入了 `TensorA`，并围绕该符号组织相关声明。 这一部分定义了片上存储，用于暂存操作数 tile、流水线元数据或累加器状态。

### Lines 480-483

```cpp
      Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.data()), SmemLayoutA{});        // (BLK_M,BLK_K,PIPE)
      Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.data()), SmemLayoutB{});        // (BLK_N,BLK_K,PIPE)
      Tensor sSFA = make_tensor(make_smem_ptr(shared_tensors.smem_scale_A.data()), SmemLayoutScaleA{});
      Tensor sSFB = make_tensor(make_smem_ptr(shared_tensors.smem_scale_B.data()), SmemLayoutScaleB{});
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 485-487

```cpp
      //
      // Prepare the TMA loads for A and B
      //
```
**EN:** This comment-only block labels or explains the following section:  Prepare the TMA loads for A and B .
**CN:** 这个纯注释块用于标记或解释后续区域： Prepare the TMA loads for A and B 。

### Lines 489-490

```cpp
      constexpr uint32_t cluster_shape_x = get<0>(typename DispatchPolicy::ClusterShape());
      uint2 cluster_local_block_id = {block_rank_in_cluster % cluster_shape_x, block_rank_in_cluster / cluster_shape_x};
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 492-493

```cpp
      Tensor gA_mkl = get<0>(load_inputs);
      Tensor gB_nkl = get<1>(load_inputs);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 495-496

```cpp
      auto block_tma_a = mainloop_params.tma_load_a.get_slice(cluster_local_block_id.y);
      auto block_tma_b = mainloop_params.tma_load_b.get_slice(cluster_local_block_id.x);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 498-501

```cpp
      // Partition the inputs based on the current block coordinates.
      auto [m_coord, n_coord, k_coord, l_coord] = blk_coord;
      Tensor gA = gA_mkl(_,_,m_coord,_,l_coord);                                                     // (BLK_M,BLK_K,k)
      Tensor gB = gB_nkl(_,_,n_coord,_,l_coord);                                                     // (BLK_N,BLK_K,k)
```
**EN:** The surrounding comments explain the local purpose of this block: Partition the inputs based on the current block coordinates..
**CN:** 周围注释解释了这一段的局部作用：Partition the inputs based on the current block coordinates.。

### Lines 503-507

```cpp
      // Block scaling: load_scale has scaling tensors in global memory which are not tiled
      Tensor mSFA_mkl = get<2>(load_inputs);
      Tensor mSFB_nkl = get<3>(load_inputs);
      auto scales_m = get<0>(mSFA_mkl.shape());
      auto scales_n = get<0>(mSFB_nkl.shape());
```
**EN:** The surrounding comments explain the local purpose of this block: Block scaling: load_scale has scaling tensors in global memory which are not tiled.
**CN:** 周围注释解释了这一段的局部作用：Block scaling: load_scale has scaling tensors in global memory which are not tiled。

### Lines 509-522

```cpp
      Tensor cSFA_mkl = make_identity_tensor(mSFA_mkl.shape());
      Tensor cSFB_nkl = make_identity_tensor(mSFB_nkl.shape());
      Tensor gSFA = local_tile(
        mSFA_mkl, make_tile(Int<ScaleMsPerTile>{}),
        make_coord(m_coord,_,l_coord));                   // (ScaleMsPerTile,k,1)
      Tensor cSFA = local_tile(
        cSFA_mkl, make_tile(Int<ScaleMsPerTile>{}),
        make_coord(m_coord,_,l_coord));
      Tensor gSFB = local_tile(
        mSFB_nkl, make_tile(Int<ScaleNsPerTile>{}),
        make_coord(n_coord,_,l_coord));                   // (ScaleNsPerTile,k,1)
      Tensor cSFB = local_tile(
        cSFB_nkl, make_tile(Int<ScaleNsPerTile>{}),
        make_coord(n_coord,_,l_coord));
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 524-527

```cpp
      TiledCopy scale_copy_a = make_tiled_copy(SmemBlockScalingCopyAtomA{},
        Layout<Shape<_32>>{}, Layout<Shape<_1>>{});
      TiledCopy scale_copy_b = make_tiled_copy(SmemBlockScalingCopyAtomB{},
        Layout<Shape<_32>>{}, Layout<Shape<_1>>{});
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 529-530

```cpp
      ThrCopy thr_scale_copy_a = scale_copy_a.get_slice(thread_idx);
      ThrCopy thr_scale_copy_b = scale_copy_b.get_slice(thread_idx);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 532-534

```cpp
      Tensor tAgA_SFA = thr_scale_copy_a.partition_S(gSFA);
      Tensor tAcA_SFA = thr_scale_copy_a.partition_S(cSFA);
      Tensor tAsA_SFA = thr_scale_copy_a.partition_D(sSFA);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 536-538

```cpp
      Tensor tBgB_SFB = thr_scale_copy_b.partition_S(gSFB);
      Tensor tBcB_SFB = thr_scale_copy_b.partition_S(cSFB);
      Tensor tBsB_SFB = thr_scale_copy_b.partition_D(sSFB);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 540-541

```cpp
      Tensor tApA_SFA = make_tensor<bool>(shape(tAsA_SFA(_,_,0)));
      Tensor tBpB_SFB = make_tensor<bool>(shape(tBsB_SFB(_,_,0)));
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 543-544

```cpp
      auto scale_m_lim = std::min(scales_m, (m_coord + 1) * ScaleMsPerTile);
      auto scale_n_lim = std::min(scales_n, (n_coord + 1) * ScaleNsPerTile);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 546-548

```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size(tApA_SFA); ++i)
        tApA_SFA(i) = get<0>(tAcA_SFA(i)) < scale_m_lim;
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 550-552

```cpp
      CUTLASS_PRAGMA_UNROLL
      for (int i = 0; i < size(tBpB_SFB); ++i)
        tBpB_SFB(i) = get<0>(tBcB_SFB(i)) < scale_n_lim;
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 554-556

```cpp
      // Applies the mapping from block_tma_a
      Tensor tAgA = block_tma_a.partition_S(gA);                                                 // (TMA,TMA_M,TMA_K,k)
      Tensor tAsA = block_tma_a.partition_D(sA);                                              // (TMA,TMA_M,TMA_K,PIPE)
```
**EN:** The surrounding comments explain the local purpose of this block: Applies the mapping from block_tma_a.
**CN:** 周围注释解释了这一段的局部作用：Applies the mapping from block_tma_a。

### Lines 558-559

```cpp
      Tensor tBgB = block_tma_b.partition_S(gB);                                                 // (TMA,TMA_N,TMA_K,k)
      Tensor tBsB = block_tma_b.partition_D(sB);                                              // (TMA,TMA_N,TMA_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 561-563

```cpp
      // TMA Multicast Masks
      Layout cta_layout_mnk = make_layout(ClusterShape{});
      auto cta_coord_mnk = cta_layout_mnk.get_flat_coord(block_rank_in_cluster);
```
**EN:** The surrounding comments explain the local purpose of this block: TMA Multicast Masks.
**CN:** 周围注释解释了这一段的局部作用：TMA Multicast Masks。

### Lines 565-566

```cpp
      uint16_t mcast_mask_a = create_tma_multicast_mask<1>(cta_layout_mnk, cta_coord_mnk);
      uint16_t mcast_mask_b = create_tma_multicast_mask<0>(cta_layout_mnk, cta_coord_mnk);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 568-572

```cpp
      // Mainloop
      CUTLASS_PRAGMA_NO_UNROLL
      for ( ; k_tile_count > 0; --k_tile_count) {
        // LOCK smem_pipe_write for _writing_
        pipeline.producer_acquire(smem_pipe_write);
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 574-576

```cpp
        //
        // Copy gmem to smem for *k_tile_iter
        //
```
**EN:** This comment-only block labels or explains the following section:  Copy gmem to smem for *k_tile_iter .
**CN:** 这个纯注释块用于标记或解释后续区域： Copy gmem to smem for *k_tile_iter 。

### Lines 578-581

```cpp
        int write_stage = smem_pipe_write.index();
        if (lane_predicate) {
          using BarrierType = typename MainloopPipeline::ProducerBarrierType;
          BarrierType* tma_barrier = pipeline.producer_get_barrier(smem_pipe_write);
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely. Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 583-585

```cpp
          copy(mainloop_params.tma_load_a.with(get<0>(input_tensormaps), *tma_barrier, mcast_mask_a), tAgA(_,_,_,*k_tile_iter), tAsA(_,_,_,write_stage));
          copy(mainloop_params.tma_load_b.with(get<1>(input_tensormaps), *tma_barrier, mcast_mask_b), tBgB(_,_,_,*k_tile_iter), tBsB(_,_,_,write_stage));
        }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 587-591

```cpp
        // Copy scale tensors
        copy_if(scale_copy_a, tApA_SFA, tAgA_SFA(_,_,*k_tile_iter), tAsA_SFA(_,_,write_stage));
        copy_if(scale_copy_b, tBpB_SFB, tBgB_SFB(_,_,*k_tile_iter), tBsB_SFB(_,_,write_stage));
        pipeline.producer_commit(smem_pipe_write, cutlass::arch::cpasync_barrier_arrive_noinc);
        ++k_tile_iter;
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 593-597

```cpp
        // Advance smem_pipe_write
        ++smem_pipe_write;
      }
    syncwarp();
  }
```
**EN:** The surrounding comments explain the local purpose of this block: Advance smem_pipe_write.
**CN:** 周围注释解释了这一段的局部作用：Advance smem_pipe_write。

### Lines 599-602

```cpp
  /// Perform a Producer Epilogue to prevent early exit of blocks in a Cluster
  CUTLASS_DEVICE void
  load_tail(MainloopPipeline pipeline, PipelineState smem_pipe_write) {
    int lane_predicate = cute::elect_one_sync();
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 605-615

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

### Lines 617-630

```cpp
  /// Perform a collective-scoped matrix multiply-accumulate
  /// Consumer Perspective
  template <
    class FrgTensorC
  >
  CUTLASS_DEVICE void
  mma(MainloopPipeline pipeline,
      PipelineState smem_pipe_read,
      FrgTensorC& accum,
      int k_tile_count,
      int thread_idx,
      TensorStorage& shared_tensors,
      Params const& mainloop_params) {
    using namespace cute;
```
**EN:** This statement brings CuTe symbols into the local scope, which keeps the template-heavy code shorter and easier to read.
**CN:** 这条语句把 CuTe 符号引入局部作用域，从而让大量模板代码更短、更易读。

### Lines 632-632

```cpp
    static_assert(is_rmem<FrgTensorC>::value, "C tensor must be rmem resident.");
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 634-636

```cpp
    FrgTensorC tmp_accum;
    clear(accum);
    clear(tmp_accum);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 638-639

```cpp
    Tensor sA = make_tensor(make_smem_ptr(shared_tensors.smem_A.data()), SmemLayoutA{});    // (BLK_M,BLK_K,PIPE)
    Tensor sB = make_tensor(make_smem_ptr(shared_tensors.smem_B.data()), SmemLayoutB{});    // (BLK_N,BLK_K,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 641-651

```cpp
    // Block scaling
    Tensor sScaleAViewAsC = make_tensor(cute::make_smem_ptr(shared_tensors.smem_scale_A.data()),
      Layout<
        Shape<Shape<Int<ScaleGranularityM>, Int<ScaleMsPerTile>>, cute::tuple_element_t<1, TileShape>, Int<DispatchPolicy::Stages>>,
        Stride<Stride<_0, _1>, _0, Int<ScaleMsPerTile>>
      >{}); // ((ScaleGranularityM,ScaleMsPerTile),TileShape_N,stage)
    Tensor sScaleBViewAsC = make_tensor(cute::make_smem_ptr(shared_tensors.smem_scale_B.data()),
      Layout<
        Shape<cute::tuple_element_t<0, TileShape>, Shape<Int<ScaleGranularityN>, Int<ScaleNsPerTile>>, Int<DispatchPolicy::Stages>>,
        Stride<_0, Stride<_0, _1>, Int<ScaleNsPerTile>>
      >{}); // (TileShape_M,(ScaleGranularityN,ScaleNsPerTile),stage)
```
**EN:** The surrounding comments explain the local purpose of this block: Block scaling.
**CN:** 周围注释解释了这一段的局部作用：Block scaling。

### Lines 654-656

```cpp
    //
    // Define C accumulators and A/B partitioning
    //
```
**EN:** This comment-only block labels or explains the following section:  Define C accumulators and A/B partitioning .
**CN:** 这个纯注释块用于标记或解释后续区域： Define C accumulators and A/B partitioning 。

### Lines 658-659

```cpp
    TiledMma tiled_mma;
    auto thread_mma = tiled_mma.get_thread_slice(thread_idx);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 661-663

```cpp
    // Allocate fragments and descriptors
    Tensor tCrA = thread_mma.partition_fragment_A(sA(_,_,Int<0>{}));                         // (MMA,MMA_M,MMA_K)
    Tensor tCrB = thread_mma.partition_fragment_B(sB(_,_,Int<0>{}));                         // (MMA,MMA_N,MMA_K)
```
**EN:** The surrounding comments explain the local purpose of this block: Allocate fragments and descriptors.
**CN:** 周围注释解释了这一段的局部作用：Allocate fragments and descriptors。

### Lines 665-666

```cpp
    Tensor tCsScaleAViewAsC = thread_mma.partition_C(sScaleAViewAsC);                        // (MMA,MMA_M,MMA_N,PIPE)
    Tensor tCsScaleBViewAsC = thread_mma.partition_C(sScaleBViewAsC);                        // (MMA,MMA_M,MMA_N,PIPE)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 668-670

```cpp
    //
    // Copy Atom A and B retiling
    //
```
**EN:** This comment-only block labels or explains the following section:  Copy Atom A and B retiling .
**CN:** 这个纯注释块用于标记或解释后续区域： Copy Atom A and B retiling 。

### Lines 672-676

```cpp
    auto smem_tiled_copy_A = make_tiled_copy_A(SmemCopyAtomA{}, tiled_mma);
    auto smem_thr_copy_A   = smem_tiled_copy_A.get_thread_slice(thread_idx);
    Tensor tCsA            = smem_thr_copy_A.partition_S(
      as_position_independent_swizzle_tensor(sA));                                           // (CPY,CPY_M,CPY_K,PIPE)
    Tensor tCrA_copy_view  = smem_thr_copy_A.retile_D(tCrA);                                 //      (CPY,CPY_M,CPY_K)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 678-682

```cpp
    auto smem_tiled_copy_B = make_tiled_copy_B(SmemCopyAtomB{}, tiled_mma);
    auto smem_thr_copy_B   = smem_tiled_copy_B.get_thread_slice(thread_idx);
    Tensor tCsB            = smem_thr_copy_B.partition_S(
      as_position_independent_swizzle_tensor(sB));                                           // (CPY,CPY_M,CPY_K,PIPE)
    Tensor tCrB_copy_view  = smem_thr_copy_B.retile_D(tCrB);                                 //      (CPY,CPY_M,CPY_K)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 684-685

```cpp
    Tensor tCrScaleAViewAsC = make_tensor_like<ElementSF>(tCsScaleAViewAsC(_,_,_,_0{}));     // (MMA,MMA_M,MMA_N)
    Tensor tCrScaleBViewAsC = make_tensor_like<ElementSF>(tCsScaleBViewAsC(_,_,_,_0{}));     // (MMA,MMA_M,MMA_N)
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 687-694

```cpp
    CUTE_STATIC_ASSERT_V(size<1>(tCsA) == size<1>(tCrA_copy_view));
    CUTE_STATIC_ASSERT_V(size<2>(tCsA) == size<2>(tCrA_copy_view));
    CUTE_STATIC_ASSERT_V(size<1>(tCrA) == size<1>(accum));
    CUTE_STATIC_ASSERT_V(size<1>(tCrB) == size<2>(accum));
    CUTE_STATIC_ASSERT_V(size<2>(tCsA) == size<2>(tCsB));
    CUTE_STATIC_ASSERT_V(size<3>(tCsA) == size<3>(tCsB));
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::Stages>{} == size<2>(sA));
    CUTE_STATIC_ASSERT_V(Int<DispatchPolicy::Stages>{} == size<2>(sB));
```
**EN:** The compile-time assertions reject unsupported layouts, copy atoms, or tile combinations before code generation continues.
**CN:** 这些编译期断言会在代码生成继续之前，提前拒绝不受支持的布局、拷贝原语或 tile 组合。

### Lines 696-698

```cpp
    //
    // PIPELINED MAIN LOOP
    //
```
**EN:** This comment-only block labels or explains the following section:  PIPELINED MAIN LOOP .
**CN:** 这个纯注释块用于标记或解释后续区域： PIPELINED MAIN LOOP 。

### Lines 700-701

```cpp
    // Size of the register pipeline
    auto K_BLOCK_MAX = size<2>(tCrA);
```
**EN:** The surrounding comments explain the local purpose of this block: Size of the register pipeline.
**CN:** 周围注释解释了这一段的局部作用：Size of the register pipeline。

### Lines 703-705

```cpp
    int read_stage = smem_pipe_read.index();
    auto tCsA_stage   = tCsA(_,_,_,read_stage);
    auto tCsB_stage   = tCsB(_,_,_,read_stage);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 707-710

```cpp
    auto copy_kblock = [&](auto k_block) {
        // copy smem->rmem for A/B operand
      copy(smem_tiled_copy_A, tCsA_stage(_,_,k_block), tCrA_copy_view(_,_,k_block));
      copy(smem_tiled_copy_B, tCsB_stage(_,_,k_block), tCrB_copy_view(_,_,k_block));
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。

### Lines 712-716

```cpp
      // Left shift A,B for FP4
      using MMAOp = typename TiledMma::MMA_Op;
      fp4_shift_A(MMAOp{}, tCrA_copy_view(_,_,k_block));
      fp4_shift_B(MMAOp{}, tCrB_copy_view(_,_,k_block));
    };
```
**EN:** It defines local type aliases so later tensor, layout, and pipeline code can refer to long template types concisely.
**CN:** 它定义了一组本地类型别名，让后续张量、布局和流水线代码能更简洁地引用冗长模板类型。

### Lines 718-738

```cpp
    auto copy_scale_s2r = [&](auto read_stage) {
      copy(tCsScaleAViewAsC(_, _, _, read_stage), tCrScaleAViewAsC);
      copy(tCsScaleBViewAsC(_, _, _, read_stage), tCrScaleBViewAsC);
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile == 1) {
        tCrScaleAViewAsC.data()[0] = tCrScaleAViewAsC.data()[0] * tCrScaleBViewAsC.data()[0];
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile == 1) {
        ElementSF scale_b = tCrScaleBViewAsC.data()[0];
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(tCrScaleAViewAsC); i++) {
          tCrScaleAViewAsC.data()[i] = tCrScaleAViewAsC.data()[i] * scale_b;
        }
      }
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile  > 1) {
        ElementSF scale_a = tCrScaleAViewAsC.data()[0];
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(tCrScaleBViewAsC); i++) {
          tCrScaleBViewAsC.data()[i] = tCrScaleBViewAsC.data()[i] * scale_a;
        }
      }
    };
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 740-771

```cpp
    auto rescale = [&]() {
      // Block scale the accumulators with reg tensor `tCrScaleAViewAsC` and `tCrScaleBViewAsC`
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile == 1) {
        ElementSF scale_ab = tCrScaleAViewAsC.data()[0];
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(accum); ++i) {
          accum(i) += tmp_accum(i) * scale_ab;
          tmp_accum(i) = 0;
        }
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile == 1) {
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(accum); ++i) {
          accum(i) += tmp_accum(i) * tCrScaleAViewAsC(i);
          tmp_accum(i) = 0;
        }
      }
      if constexpr (ScaleMsPerTile == 1 && ScaleNsPerTile  > 1) {
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(accum); ++i) {
          accum(i) += tmp_accum(i) * tCrScaleBViewAsC(i);
          tmp_accum(i) = 0;
        }
      }
      if constexpr (ScaleMsPerTile  > 1 && ScaleNsPerTile  > 1) {
        CUTLASS_PRAGMA_UNROLL
        for (int i = 0; i < size(accum); ++i) {
          accum(i) += tmp_accum(i) * tCrScaleAViewAsC(i) * tCrScaleBViewAsC(i);
          tmp_accum(i) = 0;
        }
      }
    };
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 773-776

```cpp
    auto gemm_kblock = [&](auto k_block) {
      // (V,M) x (V,N) => (V,M,N)
      cute::gemm(tiled_mma, tCrA(_,_,k_block), tCrB(_,_,k_block), tmp_accum);
    };
```
**EN:** This block contains the actual multiply-accumulate step where prepared fragments are consumed by the MMA engine.
**CN:** 这一段包含真正的乘加步骤，已经准备好的片段会在这里被 MMA 引擎消费。

### Lines 778-786

```cpp
    pipeline.consumer_wait(smem_pipe_read);
    copy_scale_s2r(read_stage);
    copy_kblock(_0{});
    CUTLASS_PRAGMA_NO_UNROLL
    for ( ; k_tile_count > 1; --k_tile_count) {
      //
      // Compute on k_tile
      //
      for_each(make_int_sequence<K_BLOCK_MAX>{}, [&] (auto k_block) {
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 788-788

```cpp
        auto k_block_next = ((k_block + 1) == K_BLOCK_MAX) ? 0 : (k_block + 1);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 790-800

```cpp
        if (k_block == K_BLOCK_MAX - 1) {
          cutlass::arch::NamedBarrier::sync(
          thr_size(tiled_mma), cutlass::arch::ReservedNamedBarriers::Sm120MainloopBarrier);
          // UNLOCK smem_pipe_read, done _computing_ on it
          pipeline.consumer_release(smem_pipe_read);
          ++smem_pipe_read;
          read_stage = smem_pipe_read.index();
          tCsA_stage   = tCsA(_,_,_,read_stage);
          tCsB_stage   = tCsB(_,_,_,read_stage);
          pipeline.consumer_wait(smem_pipe_read);
        }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 802-803

```cpp
        copy_kblock(k_block_next);
        gemm_kblock(k_block);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 805-808

```cpp
        if (k_block == K_BLOCK_MAX - 1) {
          rescale();
          copy_scale_s2r(read_stage);
        }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 810-810

```cpp
      });
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 812-812

```cpp
    } // k_tile_count
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 814-817

```cpp
    //
    // Hoist out last k_tile
    //
    for_each(make_int_sequence<K_BLOCK_MAX>{}, [&] (auto k_block) {
```
**EN:** The surrounding comments explain the local purpose of this block:  Hoist out last k_tile .
**CN:** 周围注释解释了这一段的局部作用： Hoist out last k_tile 。

### Lines 819-819

```cpp
      auto k_block_next = ((k_block + 1) == K_BLOCK_MAX) ? 0 : (k_block + 1);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 821-827

```cpp
      if (k_block == K_BLOCK_MAX - 1) {
        cutlass::arch::NamedBarrier::sync(
        thr_size(tiled_mma), cutlass::arch::ReservedNamedBarriers::Sm120MainloopBarrier);
        // UNLOCK smem_pipe_read, done _computing_ on it
        pipeline.consumer_release(smem_pipe_read);
        ++smem_pipe_read;
      }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 829-832

```cpp
      if (k_block_next > 0) {
        copy_kblock(k_block_next);
      }
      gemm_kblock(k_block);
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 834-836

```cpp
    });
    rescale();
  }
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 838-841

```cpp
  /// Perform a Consumer Epilogue to release all buffers
  CUTLASS_DEVICE void
  mma_tail(MainloopPipeline, PipelineState, int) {
  }
```
**EN:** Synchronization and pipeline bookkeeping appear here so producer/consumer phases stay correctly ordered across threads or warpgroups.
**CN:** 这里出现了同步与流水线记账逻辑，以保证不同线程或 warpgroup 之间的生产者/消费者阶段顺序正确。

### Lines 844-846

```cpp
  //
  // Methods to perform different parts of TMA/Tensormap modifications
  //
```
**EN:** This comment starts a method section where the collective exposes its runtime behavior.
**CN:** 这条注释表示进入方法区，collective 的运行时行为将在这里定义。

### Lines 848-854

```cpp
  CUTLASS_DEVICE auto
  tensormaps_init(
      Params const& mainloop_params,
      TensorMapStorage& shared_tensormaps,
      int32_t sm_count,
      int32_t sm_idx) {
    cute::TmaDescriptor* gmem_tensormap = reinterpret_cast<cute::TmaDescriptor*>(mainloop_params.tensormaps);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 856-857

```cpp
    cute::TmaDescriptor* tma_desc_a = &gmem_tensormap[sm_idx];
    cute::TmaDescriptor* tma_desc_b = &gmem_tensormap[sm_idx + sm_count];
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 859-864

```cpp
    if (cute::elect_one_sync()) {
      // Bringing tensormaps from params to smem for modification later
      Tensor pA_tensormap = make_tensor(mainloop_params.tma_load_a.get_tma_descriptor(), Int<1>{}, Int<1>{});
      Tensor sA_tensormap = make_tensor(make_smem_ptr(&shared_tensormaps.smem_tensormap_A), Int<1>{}, Int<1>{});
      Tensor pB_tensormap = make_tensor(mainloop_params.tma_load_b.get_tma_descriptor(), Int<1>{}, Int<1>{});
      Tensor sB_tensormap = make_tensor(make_smem_ptr(&shared_tensormaps.smem_tensormap_B), Int<1>{}, Int<1>{});
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 866-871

```cpp
      copy(recast<uint128_t>(pA_tensormap), recast<uint128_t>(sA_tensormap));
      copy(recast<uint128_t>(pB_tensormap), recast<uint128_t>(sB_tensormap));
    }
    syncwarp();
    return cute::make_tuple(tma_desc_a, tma_desc_b);
  }
```
**EN:** It also issues explicit data-movement operations such as copies or prefetches to keep the compute pipeline fed. The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 它还会发出显式数据搬运操作，例如 copy 或 prefetch，以持续为计算流水线供数。 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 873-884

```cpp
  // Replace address for the global tensor (to be done by single thread)
  CUTLASS_DEVICE void
  tensormaps_replace_global_address(
      TensorMapStorage& shared_tensormaps,
      Params const& mainloop_params,
      int32_t next_batch) {
    // Replacing global_address for the next batch
    cute::tma_descriptor_replace_addr_in_shared_mem(shared_tensormaps.smem_tensormap_A,
                                                    mainloop_params.ptr_A[next_batch]);
    cute::tma_descriptor_replace_addr_in_shared_mem(shared_tensormaps.smem_tensormap_B,
                                                    mainloop_params.ptr_B[next_batch]);
  }
```
**EN:** The surrounding comments explain the local purpose of this block: Replace address for the global tensor (to be done by single thread) Replacing global_address for the next batch.
**CN:** 周围注释解释了这一段的局部作用：Replace address for the global tensor (to be done by single thread) Replacing global_address for the next batch。

### Lines 886-901

```cpp
  template <class ProblemShape_MNKL>
  CUTLASS_DEVICE void
  tensormaps_replace_global_tensor_properties(
      TensorMapStorage& shared_tensormaps,
      Params const& mainloop_params,
      int32_t next_group,
      ProblemShape_MNKL problem_shape_mnkl) {
    const uint32_t M = get<0>(problem_shape_mnkl);
    const uint32_t N = get<1>(problem_shape_mnkl);
    const uint32_t K = get<2>(problem_shape_mnkl);
    // Replace all dims for consistency
    constexpr int MaxTensorRank = 5;
    cute::array<uint32_t, MaxTensorRank> prob_shape_A  = {1,1,1,1,1};
    cute::array<uint64_t, MaxTensorRank> prob_stride_A = {0,0,0,0,0};
    cute::array<uint32_t, MaxTensorRank> prob_shape_B  = {1,1,1,1,1};
    cute::array<uint64_t, MaxTensorRank> prob_stride_B = {0,0,0,0,0};
```
**EN:** This block introduces `ProblemShape_MNKL` and groups related declarations around that symbol.
**CN:** 这一段引入了 `ProblemShape_MNKL`，并围绕该符号组织相关声明。

### Lines 903-904

```cpp
    TmaInternalElementA const* ptr_A = nullptr;
    Tensor tensor_a = make_tensor(ptr_A, make_shape(M,K,Int<1>{}), mainloop_params.dA[next_group]);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 907-908

```cpp
    TmaInternalElementB const* ptr_B = nullptr;
    Tensor tensor_b = make_tensor(ptr_B, make_shape(N,K,Int<1>{}), mainloop_params.dB[next_group]);
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 910-920

```cpp
    cute::detail::fill_tma_gmem_shape_stride(mainloop_params.tma_load_a, tensor_a,
                                            prob_shape_A, prob_stride_A);
    cute::detail::fill_tma_gmem_shape_stride(mainloop_params.tma_load_b, tensor_b,
                                            prob_shape_B, prob_stride_B);
    // Convert strides to byte strides
    for (uint64_t& stride : prob_stride_A) {
      stride = (stride * sizeof_bits_v<TmaInternalElementA>) / 8;
    }
    for (uint64_t& stride : prob_stride_B) {
      stride = (stride * sizeof_bits_v<TmaInternalElementB>) / 8;
    }
```
**EN:** Loop structure in this block advances over stages, K-tiles, or fragment elements while giving the compiler room to unroll hot paths.
**CN:** 这一段中的循环会推进各个 stage、K-tile 或片段元素，同时为编译器展开热点路径创造条件。

### Lines 922-928

```cpp
    cute::tma_descriptor_replace_dims_strides_in_shared_mem(shared_tensormaps.smem_tensormap_A,
                                                            prob_shape_A,
                                                            prob_stride_A);
    cute::tma_descriptor_replace_dims_strides_in_shared_mem(shared_tensormaps.smem_tensormap_B,
                                                            prob_shape_B,
                                                            prob_stride_B);
  }
```
**EN:** This block continues the file’s template-driven collective GEMM implementation with local declarations or control flow.
**CN:** 这一段继续推进该文件基于模板的 collective GEMM 实现，补充局部声明或控制流程。

### Lines 930-941

```cpp
  // The entire warp must call this function collectively (that is, the instructions are aligned)
  template <class TensorMapA, class TensorMapB, class ProblemShape_MNKL>
  CUTLASS_DEVICE void
  tensormaps_perform_update(
      TensorMapStorage& shared_tensormaps,
      Params const& mainloop_params,
      cute::tuple<TensorMapA, TensorMapB> const& input_tensormaps,
      ProblemShape_MNKL problem_shape_mnkl,
      int32_t next_batch) {
    if (cute::elect_one_sync()) {
      // Replacing global_address for the next batch
      tensormaps_replace_global_address(shared_tensormaps, mainloop_params, next_batch);
```
**EN:** This block introduces `TensorMapA` and groups related declarations around that symbol. Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这一段引入了 `TensorMapA`，并围绕该符号组织相关声明。 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 943-949

```cpp
      if constexpr (IsGroupedGemmKernel) {
        // Replacing global dims and strides for the next batch
        tensormaps_replace_global_tensor_properties(shared_tensormaps,
          mainloop_params, next_batch, problem_shape_mnkl);
      }
    }
  }
```
**EN:** Conditional branches select fallback behavior, runtime datatype handling, or optional fast paths based on compile-time/runtime state.
**CN:** 这些条件分支会依据编译期或运行时状态选择回退行为、运行时数据类型处理路径或可选快速路径。

### Lines 951-959

```cpp
  template <class TensorMapA, class TensorMapB>
  CUTLASS_DEVICE void
  tensormaps_cp_fence_release (
      TensorMapStorage& shared_tensormaps,
      cute::tuple<TensorMapA, TensorMapB> const& input_tensormaps) {
    // Entire warp must do this (i.e. it's aligned)
    tma_descriptor_cp_fence_release(get<0>(input_tensormaps), shared_tensormaps.smem_tensormap_A);
    tma_descriptor_cp_fence_release(get<1>(input_tensormaps), shared_tensormaps.smem_tensormap_B);
  }
```
**EN:** This block introduces `TensorMapA` and groups related declarations around that symbol.
**CN:** 这一段引入了 `TensorMapA`，并围绕该符号组织相关声明。

### Lines 961-967

```cpp
  // The entire warp must call this function collectively (that is, the instructions are aligned)
  template <class TensorMapA, class TensorMapB>
  CUTLASS_DEVICE void
  tensormaps_fence_acquire(cute::tuple<TensorMapA, TensorMapB> const& input_tensormaps) {
    cute::tma_descriptor_fence_acquire(get<0>(input_tensormaps));
    cute::tma_descriptor_fence_acquire(get<1>(input_tensormaps));
  }
```
**EN:** This block introduces `TensorMapA` and groups related declarations around that symbol.
**CN:** 这一段引入了 `TensorMapA`，并围绕该符号组织相关声明。

### Lines 969-987

```cpp
  template <class InputTensors, class ProblemShape_MNKL>
  CUTLASS_DEVICE InputTensors
  tensors_perform_update(
      InputTensors const& input_tensors,
      Params const& mainloop_params,
      ProblemShape_MNKL problem_shape_mnkl,
      int32_t next_batch) {
    if constexpr (IsGroupedGemmKernel) {
      return load_init(
        problem_shape_mnkl,
        mainloop_params,
        mainloop_params.ptr_SFA[next_batch],
        mainloop_params.ptr_SFB[next_batch],
        mainloop_params.layout_SFA[next_batch],
        mainloop_params.layout_SFB[next_batch]
      );
    }
    else {
      auto [gA_mkl, gB_nkl, mSFA_mkl, mSFB_nkl] = input_tensors;
```
**EN:** This block introduces `InputTensors` and groups related declarations around that symbol. This block prepares the load path by slicing global tensors, partitioning work for the current CTA/subgroup, and binding shared-memory destinations.
**CN:** 这一段引入了 `InputTensors`，并围绕该符号组织相关声明。 这一段为加载路径做准备：切分全局张量、为当前 CTA/子组划分工作，并绑定共享内存目标。

### Lines 989-990

```cpp
      mSFA_mkl = make_tensor(make_gmem_ptr(mainloop_params.ptr_SFA[next_batch]), mainloop_params.layout_SFA[next_batch]);
      mSFB_nkl = make_tensor(make_gmem_ptr(mainloop_params.ptr_SFB[next_batch]), mainloop_params.layout_SFB[next_batch]);
```
**EN:** It constructs tensor views or copy descriptors that map global-memory operands onto the collective’s tiled execution model.
**CN:** 它构造张量视图或拷贝描述符，把全局内存中的操作数映射到 collective 的分块执行模型上。

### Lines 992-995

```cpp
      return cute::make_tuple(gA_mkl, gB_nkl, mSFA_mkl, mSFB_nkl);
    }
  }
};
```
**EN:** The return statement hands the assembled descriptor, tensor view, boolean result, or helper object back to the caller.
**CN:** 这里的返回语句把组装好的描述符、张量视图、布尔结果或辅助对象交还给调用方。

### Lines 997-997

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
```
**EN:** This comment-only block labels or explains the following section: .
**CN:** 这个纯注释块用于标记或解释后续区域：。

### Lines 999-999

```cpp
} // namespace cutlass::gemm::collective
```
**EN:** This closing code ends the current scope, specialization, or helper definition.
**CN:** 这段结尾代码用于关闭当前作用域、特化或辅助定义。

### Lines 1001-1001

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
- **EN:** UMMA tensor-core instructions  
  **CN:** UMMA 张量核指令
- **EN:** warp-specialized scheduling  
  **CN:** warp-specialized 调度
- **EN:** blockwise scaling  
  **CN:** 块级缩放
- **EN:** cp.async copy stages  
  **CN:** cp.async 拷贝阶段
- **EN:** array-style MMA tiling  
  **CN:** 数组式 MMA 分块
- **EN:** Intel Xe subgroup execution  
  **CN:** Intel Xe 子组执行
- **EN:** software pipelines  
  **CN:** 软件流水线
- **EN:** compile-time validation  
  **CN:** 编译期校验

## Dependencies / 依赖关系

- **Direct includes / 直接包含:**
  - `cutlass/cutlass.h`
  - `cutlass/gemm/gemm.h`
  - `cutlass/pipeline/pipeline.hpp`
  - `cutlass/gemm/dispatch_policy.hpp`
  - `cutlass/detail/dependent_false.hpp`
  - `cutlass/trace.h`
  - `cutlass/numeric_types.h`
  - `cute/arch/cluster_sm90.hpp`
  - `cute/arch/copy_sm90.hpp`
  - `cute/atom/mma_atom.hpp`
  - `cute/algorithm/functional.hpp`
  - `cute/algorithm/gemm.hpp`
  - `cute/numeric/arithmetic_tuple.hpp`
- **Primary symbols / 主要符号:**
  - `ClusterShape`
  - `KernelScheduleType`
  - `TileShape_`
  - `ElementA_`
  - `StridePairA_`
  - `ElementB_`
  - `StridePairB_`
  - `TiledMma_`
  - `GmemTiledCopyA_`
  - `SmemLayoutAtomA_`
  - `SmemCopyAtomA_`
  - `TransformA_`
  - `GmemTiledCopyB_`
  - `SmemLayoutAtomB_`
  - `SmemCopyAtomB_`
  - `TransformB_`
- **Shared abstractions / 共享抽象:** `CollectiveMma` / `CollectiveBuilder`, dispatch-policy tags, CuTe tensor/layout utilities, and CUTLASS pipeline helpers. / `CollectiveMma` / `CollectiveBuilder`、dispatch-policy 标签、CuTe 张量/布局工具以及 CUTLASS 流水线辅助组件。